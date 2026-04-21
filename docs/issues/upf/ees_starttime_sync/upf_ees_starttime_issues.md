# UPF EES startTime Issues — Report to Developer

本文件記錄在整合測試中觀察到的兩個 UPF EES 通知問題，供 UPF EES 開發者修正參考。

測試環境：NWDAF AnLF 以 `samplingInterval=5s` 對 UPF 流量進行 anchor-based 對齊聚合，
因此 UPF notification 的 `startTime` 欄位必須準確反映每個 reporting period 的起始時間。

---

## 情境說明

實驗室測試設置包含兩條訂閱路徑：

| 訂閱 | correlationId | IP | 說明 |
|------|---------------|----|------|
| `f34b24cd` | corr-2 | 10.10.0.3 | **有 pseudo driver**：啟動時由 pseudo driver 一次性塞入 30 筆歷史資料，之後接續真實 UE 流量 |
| `a43ede69` | corr-1 | 10.100.0.3 | **無 pseudo driver**：完全從即時 UE 流量開始累積 |

pseudo driver 被掛載在 UPF EES 上，負責在訂閱建立時預先注入歷史流量資料，
讓 NWDAF 在啟動後能立即進行推論，而不需等待 `inputWindow × samplingInterval`（= 30 × 5s = 150s）的暖機時間。

---

## 問題一：pseudo driver 最後一筆 startTime 與 live 第一筆重複（corr-2 / 10.10.0.3）

### 觀察到的 log

```
# pseudo driver 最後三筆（14:22:29 一次性批量送達）
UPF VOLUME: ip=10.10.0.3, startTime=2026-03-10T14:22:12Z, ...
UPF VOLUME: ip=10.10.0.3, startTime=2026-03-10T14:22:17Z, ...
UPF VOLUME: ip=10.10.0.3, startTime=2026-03-10T14:22:22Z, ...   ← 最後一筆

# 後續真實 UE 流量的第一筆（14:22:34 到達）
UPF VOLUME: ip=10.10.0.3, startTime=2026-03-10T14:22:22Z, ...   ← 與上方重複！

# NWDAF 每個推論 cycle 都發出 Warn
[WARN] dedup collision ip=10.10.0.3: t=1773152542 and previous both snap to center=1773152542
       (anchor=1773152302 si=5), keeping later
```

`t=1773152542` = `2026-03-10T14:22:22Z`，兩筆 startTime 完全相同。

### 根本原因

pseudo driver 最後一筆 reporting period 的 startTime 為 `14:22:22Z`，
而 UPF EES 在切換至即時流量後，第一筆 live notification 的 startTime **仍然是 `14:22:22Z`**，
代表同一個 reporting period 被通知了兩次。

```
pseudo driver: ... → 14:22:12Z → 14:22:17Z → 14:22:22Z ┐
                                                          ├─ 重複！
live UE:                                      14:22:22Z ┘ → 14:22:27Z → ...
```

正確行為應為 live 第一筆 startTime = `14:22:27Z`（pseudo driver 最後一筆 + 5s）。

### 對 NWDAF 的影響

- NWDAF 使用 last-wins dedup，保留後到的那筆（live 覆蓋 pseudo driver），功能上不影響推論結果
- 但每個推論 cycle 都會輸出 WARN，且同一個 reporting period 的量測值被計算兩次（pseudo driver 的歷史值被 live 覆蓋）

---

## 問題二：live 第一筆與第二筆 startTime 相差 10s（corr-1 / 10.100.0.3）

### 觀察到的 log

```
# corr-1 建立，第一筆 live 到達（14:22:31）
UPF VOLUME: ip=10.100.0.3, startTime=2026-03-10T14:22:19Z, ...   ← 第 1 筆

# 第二筆 live（14:22:36）
UPF VOLUME: ip=10.100.0.3, startTime=2026-03-10T14:22:29Z, ...   ← 第 2 筆（差 10s！）

# NWDAF 每個推論 cycle 都有 gap
[DEBU] Aggregated 3 global slots ipCount/slot:[1,0,1]
[DEBU] Aggregated 4 global slots ipCount/slot:[1,0,1,1]
[DEBU] Aggregated 5 global slots ipCount/slot:[1,0,1,1,1]
```

### 根本原因

第 1 筆 startTime = `14:22:19Z`，第 2 筆 startTime = `14:22:29Z`，相差 **10s = 2 個 sampling interval**。
`14:22:24Z` 那個 reporting period 的通知從未發送。

```
應有: 14:22:19Z → 14:22:24Z → 14:22:29Z → ...
實際: 14:22:19Z →    (缺)   → 14:22:29Z → ...
```

NWDAF 以第 1 筆 startTime 為 anchor 進行 anchor-based round：

```
anchor = 14:22:19Z

14:22:19Z → n = round((19-19)/5) = 0 → center = 14:22:19Z  ✓
14:22:29Z → n = round((29-19)/5) = 2 → center = 14:22:29Z  ✓
14:22:24Z → n = 1 → center = 14:22:24Z  ← 此格永遠空缺
```

### 對 NWDAF 的影響

- `14:22:24Z` 那個 slot 在 ring buffer 的整個生命週期內都是空格
- NWDAF 以 zero-padding 填入 0 值，ML 模型會看到一個持續性的異常零值點
- 此 gap **不會隨時間消失**（不同於 historical/live 邊界的 gap 會隨視窗左移淡出）
- 即使後續 UE 流量正常，推論輸入序列中永遠有一個 0 值污染 ML 輸入

---

## 共同規律觀察

兩條路徑的「大間隔」現象高度一致：

| 路徑 | 第 N 筆 startTime | 第 N+1 筆 startTime | 間隔 | 後續間隔 |
|------|-------------------|---------------------|------|----------|
| corr-2（pseudo driver 切換後） | 14:22:22Z（pseudo driver 最後一筆） | 14:22:32Z（live 第一筆新資料） | **10s** | 恢復 5s |
| corr-1（無 pseudo driver） | 14:22:19Z（第 1 筆） | 14:22:29Z（第 2 筆） | **10s** | 恢復 5s |

兩條路徑都是「某個特定轉折點的兩筆之間差 10s，之後恢復正常」，不像是資料量或網路問題，規律性太強。

**進一步觀察：通知送出節奏正常，問題在 `startTime` 欄位的值**

比對 corr-1 的到達時間（`timeStamp`）與 `startTime`：

```
到達 14:22:31  startTime=14:22:19Z
到達 14:22:36  startTime=14:22:29Z  ← 到達間隔 5s（正常），startTime 卻跳 10s
到達 14:22:41  startTime=14:22:34Z
到達 14:22:46  startTime=14:22:39Z
```

UPF EES 每 5s 準時送出一筆通知，**送出節奏完全正常**。問題不在「什麼時候送」，而在「通知裡 `startTime` 填的是哪個 period 的起始時間」：第二筆通知的 `startTime` 應為 `14:22:24Z`，實際填入 `14:22:29Z`，**多跳了一個 5s period**。

**推測根本原因**：UPF EES 內部計算 `startTime` 時，session 建立後第一個 reporting period 的起始時間被錯誤地跳過，導致後續每筆通知的 `startTime` 都比實際量測起點晚了 5s。`timeStamp`（通知送出時間）不受影響。

```
UPF EES 行為（推測）:
  T+0   → 建立 session，開始計時
  T+5   → 第一個 period 結束，送出通知（timeStamp=T+5），startTime 應=T+0，實際填=T+5  ← 多跳一格
  T+10  → 第二個 period 結束，送出通知（timeStamp=T+10），startTime 應=T+5，實際填=T+10
  ...
```

無論具體原因為何，修正方向應為確保 **`startTime` 欄位填入該筆通知所涵蓋 reporting period 的實際起始時間**，而非結束時間或下一個 period 的起始時間。

---

## 修正建議

### 問題一修正

pseudo driver 送完最後一筆後，UPF EES 切換到即時流量時，**live 的第一筆 startTime 必須是 pseudo driver 最後一筆 startTime + `samplingInterval`**。

```
pseudo driver last: startTime = T
live first:         startTime = T + samplingInterval   ← 正確
live first:         startTime = T                      ← 錯誤（現狀）
```

### 問題二修正

UPF EES 在開始發送即時流量通知時，必須確保 **每個 sampling interval 都有對應的 notification**，不得跳過任何一個 reporting period。若某個 period 的流量為零，仍需發送 `volume=0` 的通知。

```
正確: startTime=T → T+5 → T+10 → T+15 → ...（連續，無跳躍）
錯誤: startTime=T →     → T+10 → T+15 → ...（跳過 T+5）
```

---

## 附錄：實測 startTime 原始序列

以下為本次測試 log 中兩條訂閱各自收到的通知序列，按到達順序排列。

### corr-2（ip=10.10.0.3，有 pseudo driver）

| 到達時間（UTC） | startTime（UTC） | ul / dl (bytes) | 來源 |
|----------------|-----------------|-----------------|------|
| 14:22:29.235 ～ | （log 截取起點前，bulk 共 30 筆，最早約 14:17:37Z） | — | pseudo driver |
| 14:22:29.236 | **2026-03-10T14:22:12Z** | ul=4,845 / dl=8,238 | pseudo driver |
| 14:22:29.237 | **2026-03-10T14:22:17Z** | ul=16,571 / dl=96,172 | pseudo driver |
| 14:22:29.238 | **2026-03-10T14:22:22Z** | ul=1,741 / dl=61,113 | pseudo driver（最後一筆） |
| 14:22:34.672 | **2026-03-10T14:22:22Z** | ul=0 / dl=0 | **live（與上方重複！）** |
| 14:22:39.703 | **2026-03-10T14:22:32Z** | ul=0 / dl=0 | live（距上一筆 10s，跳過 14:22:27Z） |
| 14:22:44.672 | **2026-03-10T14:22:37Z** | ul=0 / dl=0 | live |
| 14:22:49.673 | **2026-03-10T14:22:42Z** | ul=0 / dl=0 | live |

**問題點**：
1. `14:22:22Z` 重複出現（pseudo driver 最後一筆 ＝ live 第一筆）
2. live 實質上第一筆新資料為 `14:22:32Z`，距 pseudo driver 最後一筆 `14:22:22Z` 相差 **10s**，跳過了 `14:22:27Z`

---

### corr-1（ip=10.100.0.3，無 pseudo driver）

| 到達時間（UTC） | startTime（UTC） | ul / dl (bytes) | 備註 |
|----------------|-----------------|-----------------|------|
| 14:22:31.963 | **2026-03-10T14:22:19Z** | ul=0 / dl=0 | 第 1 筆（bucket 此時建立） |
| 14:22:36.961 | **2026-03-10T14:22:29Z** | ul=0 / dl=0 | 第 2 筆（距第 1 筆 **10s**，跳過 14:22:24Z） |
| 14:22:41.961 | **2026-03-10T14:22:34Z** | ul=0 / dl=0 | 第 3 筆 |
| 14:22:46.962 | **2026-03-10T14:22:39Z** | ul=0 / dl=0 | 第 4 筆 |
| 14:22:51.961 | **2026-03-10T14:22:44Z** | ul=0 / dl=0 | 第 5 筆 |

**問題點**：
- 第 1 筆 `14:22:19Z` → 第 2 筆 `14:22:29Z`，相差 **10s**，跳過了 `14:22:24Z`
- 從第 2 筆起，間隔恢復正常（5s）

---

## 附：NWDAF anchor-based 對齊原理（供參考）

NWDAF 使用每個 IP session 的**第一筆 startTime 作為 anchor**，後續每筆通知的 reporting period 以以下公式對齊至最近的 slot：

```
n = round( (startTime - anchor) / samplingInterval )
center = anchor + n × samplingInterval
```

容許誤差為 ±`samplingInterval/2`（= ±2.5s）。只要相鄰兩筆 startTime 之差不超過 `1.5 × samplingInterval`（= 7.5s），就能被正確對齊至相鄰 slot。若差距 ≥ `1.5 × samplingInterval`，則會出現空格，NWDAF 以 zero-padding 填充。
