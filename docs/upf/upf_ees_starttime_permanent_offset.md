# UPF EES startTime 永久性偏差問題 — Report to Developer

本文件記錄在第二次整合測試中觀察到的新 UPF EES 問題：
ip=10.10.0.1 的 EES 通知其 `startTime` 欄位相較於 wall clock **永久性落後約 76–77 秒**，
導致 NWDAF accuracy monitor 無法取得正確的 ground truth，
**accuracy 永遠顯示為 0%（deviation=2.0000）**，且屬 UPF EES 側問題，與 NWDAF 無關。

---

## 情境說明

測試環境包含兩條訂閱路徑：

| group | correlationId | IP | 路徑說明 |
|-------|---------------|----|----------|
| group-test-001 | corr-2 | 10.10.0.1 (192.168.127.10) | 有 pseudo driver |
| group-test-002 | corr-1 | 10.100.0.2 (192.168.127.11) | 無 pseudo driver |

兩個 group 共用同一個 ML 模型（`model_new.npy`），predictions 交錯寫入同一個 accuracy store。

---

## 問題：ip=10.10.0.1（corr-2）startTime 永久落後 wall clock 約 76–77 秒

### 觀察到的 log

```
# 以下為實驗進行約 9 分鐘後的資料（非啟動初期）

# corr-2 (ip=10.10.0.1) 通知，每 5s 到達一筆：
到達 08:23:07  startTime=2026-03-17T08:21:50Z   ← 落後 77s
到達 08:23:12  startTime=2026-03-17T08:21:57Z   ← 落後 75s
到達 08:23:17  startTime=2026-03-17T08:22:01Z   ← 落後 76s
到達 08:23:22  startTime=2026-03-17T08:22:05Z   ← 落後 77s
到達 08:23:27  startTime=2026-03-17T08:22:12Z   ← 落後 75s
到達 08:23:32  startTime=2026-03-17T08:22:16Z   ← 落後 76s

# corr-1 (ip=10.100.0.2) 通知（對照組，正常）：
到達 08:23:08  startTime=2026-03-17T08:23:02Z   ← 落後 6s（正常 EES off-by-one）
到達 08:23:13  startTime=2026-03-17T08:23:07Z   ← 落後 6s
到達 08:23:18  startTime=2026-03-17T08:23:12Z   ← 落後 6s
```

### 關鍵觀察

- **偏差不會縮小**：實驗從 08:14 持續到 08:23，9 分鐘後偏差仍為 ~76s
- **前進速率正常**：通知到達間隔 5s，startTime 也每筆推進約 5s，表示不是 pseudo driver 卡住重播
- **是 live data，但 startTime 的基準錯誤**：時間軸在正確地前進，只是起點偏移了 ~76s

```
wall clock: 08:23:07 ──────→ 08:23:12 ──────→ 08:23:17 ──────→ ...
              ↕ 77s              ↕ 75s              ↕ 76s
  startTime: 08:21:50 ──────→ 08:21:57 ──────→ 08:22:01 ──────→ ...（平行前進，永遠落後）
```

---

## 對 NWDAF 的影響

### Accuracy Monitor 永遠無法取得 corr-2 的 ground truth

NWDAF accuracy monitor 的 ground truth 查找窗口為 `TargetTime ±5s`：

```
TargetTime（由 wall clock 推算）≈ 08:23:10
查找窗口：[08:23:05, 08:23:15]

corr-2 ring buffer 中最新的 startTime：08:22:16（落後 54s）
→ 永遠在窗口外 → ground truth NOT FOUND
```

### Accuracy 結果永遠顯示 1,0,1,0 交替模式

兩個 group 的 predictions 交錯存入同一個 store（每 5s 各一筆），按時間順序排列如下：

```
t=T+0s  → group-test-002（corr-1）的 prediction → ground truth FOUND   → 1
t=T+5s  → group-test-001（corr-2）的 prediction → ground truth MISSING → 0
t=T+10s → group-test-002（corr-1）的 prediction → ground truth FOUND   → 1
t=T+5s  → group-test-001（corr-2）的 prediction → ground truth MISSING → 0
...
```

實測 accuracy log：

```
[DEBU] Ground truth [12 mature → 6 matched]: 1,0,1,0,1,0,1,0,1,0,1,0
[INFO] Accuracy [model_new.npy]: deviation=2.0000, accuracy=0%, samples=6, inferences=12
[WARN] Threshold breach: deviation=2.0000 > 1.00 (3/3)
```

由於 matched samples 中有效的只有 6 筆（corr-1），而全部預測值都不等於 ground truth（模型尚未收斂或兩個 group 流量特性不同），sMAPE 維持在最大值 2.0，導致持續觸發不必要的 retraining。

### 此問題不在 NWDAF 側

- NWDAF 對 corr-1 的 ground truth 查找完全正常（startTime 落後僅 6s，在 ±5s 容許範圍的邊緣）
- NWDAF 的 ring buffer 確實儲存了 corr-2 的資料，資料本身存取正確
- 問題來源是 ip=10.10.0.1 的 EES 提供了錯誤的 startTime

---

## 推測根本原因

ip=10.10.0.1 的 EES 在計算 startTime 時，**使用了某個固定的時間基準偏差**，
而非當下 reporting period 的實際起始時間。

76–77 秒的偏差大小符合以下推測：pseudo driver 注入了約 15 筆歷史資料（15 × 5s = 75s），
EES 在 pseudo driver 結束後，live 資料的 startTime 從 pseudo driver 開始的時間點繼續累算，
但 **wall clock 已經過了 75s**，造成永久性落後。

即：EES 的 startTime 使用的是「pseudo driver 起始時間 + 已傳送 period 數 × si」，
而非「current_time - si」。

---

## 修正建議

**live 資料的 `startTime` 應始終反映當下 reporting period 的實際起始時間**：

```
正確計算：startTime = current_time - samplingInterval
錯誤計算：startTime = pseudo_driver_start + n × samplingInterval  ← 現狀（推測）
```

驗證方法：在 pseudo driver 結束後，檢查 live 第一筆通知的 startTime 是否接近 wall clock（差距應在 1 個 samplingInterval 以內）。

---

## 附錄：實測 startTime 原始序列（ip=10.10.0.1，實驗進行約 9 分鐘後）

| 通知到達時間（UTC） | startTime（UTC） | 落後量 | ul / dl (bytes) |
|---------------------|-----------------|--------|-----------------|
| 08:23:07.556 | 2026-03-17T08:21:50Z | 77s | ul=1,024 / dl=960 |
| 08:23:12.556 | 2026-03-17T08:21:57Z | 75s | ul=0 / dl=0 |
| 08:23:17.557 | 2026-03-17T08:22:01Z | 76s | ul=0 / dl=0 |
| 08:23:22.557 | 2026-03-17T08:22:05Z | 77s | ul=960 / dl=1,024 |
| 08:23:27.556 | 2026-03-17T08:22:12Z | 75s | ul=0 / dl=0 |
| 08:23:32.556 | 2026-03-17T08:22:16Z | 76s | ul=0 / dl=0 |

對照組 ip=10.100.0.2（正常）：

| 通知到達時間（UTC） | startTime（UTC） | 落後量 |
|---------------------|-----------------|--------|
| 08:23:08.213 | 2026-03-17T08:23:02Z | 6s |
| 08:23:13.213 | 2026-03-17T08:23:07Z | 6s |
| 08:23:18.213 | 2026-03-17T08:23:12Z | 6s |
| 08:23:23.213 | 2026-03-17T08:23:17Z | 6s |
| 08:23:28.213 | 2026-03-17T08:23:22Z | 6s |
| 08:23:33.213 | 2026-03-17T08:23:27Z | 6s |
