# UE Communication `TargetTime` Semantics Fix Plan

> 對應 issue：`.agent/docs/issues/architecture/ue_communication_prediction_target_time_semantics.md`
>
> 本文件記錄 `UE_COMMUNICATION` prediction time semantics 的程式碼確認結果、修正範圍、風險評估與建議落地方向。
>
> 本次目標是先修正 **internal prediction record / accuracy monitoring path** 的時間對位，不在同一批修改中直接改寫對外 `UE_COMMUNICATION` notify contract。

---

## 1. 問題摘要

目前 `UE_COMMUNICATION` ML prediction 路徑有兩套不一致的時間語意：

- internal `PredictionRecord.TargetTime` 目前以 `snappedNow + (i+1)*samplingInterval` 建立
- external `UeCommunication` notify 目前以 `ts=now`、`commDur=整個 horizon` 表達

其中本次最優先處理的是 internal accuracy path：

- `TargetTime` 會直接影響 mature timing
- `TargetTime` 會直接影響 ground truth lookup
- `TargetTime` 一旦偏一格，prediction/actual pairing 會整批錯 slot

---

## 2. 相關程式碼確認

### 2.1 Prediction record 產生點

`internal/anlf/analytics.go`

- `PredictionRecord` 的建立點只有一處
- 現況：

```go
TargetTime: snappedNow.Add(time.Duration((i+1)*samplingInterval) * time.Second),
```

這是目前最主要的修正點。

### 2.2 Accuracy monitor 配對邏輯

`internal/anlf/monitor.go`

- `lookupGroundTruth()` 直接以 `pred.TargetTime` 查詢 `TargetTime ± samplingInterval`
- nearest match 也是以 `pred.TargetTime` 為中心

因此：

- 若 `TargetTime` 偏移
- ground truth pairing 就會跟著偏移
- monitor metric 與 retrain trigger 判斷都會失真

### 2.3 Prediction maturity 判斷

`internal/context/accuracy_store.go`

- mature 條件是 `TargetTime + graceAfterTarget < now`

因此 internal `TargetTime` 修正後：

- prediction 會更早一個 slot 進入 monitor 比對流程
- 這是預期效果，但需要驗證實際資料到達延遲是否足以支撐

### 2.4 對外通知產生點

`internal/anlf/analytics.go`
`internal/notifier/handler.go`

目前對外固定回傳：

- 一筆 aggregated `UeCommunication`
- `Ts = now`
- `CommDur = outputWindow * samplingInterval`

這部分與 internal per-step prediction semantics 不一致，但本次先不處理。

補充：

- 這是 **既有設計問題**，不是本次 internal `TargetTime` 修正才會引入的新問題
- 目前主設定 `outputWindow=1`，因此在現行主要使用情境下，這個「多個 predicted slots 壓成一筆」的資訊損失實際上很有限
- 但只要未來 `outputWindow > 1`，或 ML service 恢復多步 prediction，這個 notify semantics 問題就會立刻放大

### 2.5 ML response 既有 `ts` 欄位未使用

`internal/sbi/consumer/ml_service.go`

`UeCommunicationPrediction` 已定義：

```go
type UeCommunicationPrediction struct {
    Ts         string                  `json:"ts"`
    TrafChar   TrafficCharacterization `json:"traf_char"`
    Confidence int32                   `json:"confidence"`
}
```

但目前 `generateMlBasedUeCommunication()` 沒有使用 `predicted_data[].ts`：

- 沒拿來決定 internal `TargetTime`
- 沒拿來校驗 NWDAF 本地推導的 target slot
- 沒反映到 external notify

這代表目前的時間語意完全由 NWDAF 本地推導承擔。

---

## 3. 本次建議修正範圍

### 3.1 In-scope

本次只處理 internal accuracy path 的時間對位一致性：

- 修正 `PredictionRecord.TargetTime`
- 驗證 mature timing 是否仍合理
- 驗證 ground truth pairing 是否對到正確 slot
- 補足對應單元測試

### 3.2 Out-of-scope

以下項目不納入同一批修改：

- 對外 `ueComms[]` 改為 per-step 表達
- `EventNotification.start/expiry/timeStampGen` 的正式分工
- 重新定義 aggregated horizon summary 的 API 語意
- 全面以 ML service `predicted_data[].ts` 取代本地 slot 推導

上述項目建議另開 follow-up。

---

## 4. 建議修正方向

### 4.1 Direction A: 修正 internal `TargetTime`

若第 1 個 prediction step 的語意是「當前完整 interval」，
則建議將：

```go
snappedNow + (i+1)*samplingInterval
```

改成：

```go
snappedNow + i*samplingInterval
```

對目前 `outputWindow=1` 的主要使用情境來說，效果就是：

- step 0 → `TargetTime = snappedNow`

### 4.2 Direction A 的前提

這個修正成立的前提是：

- ML model 的第 1 個輸出 step 語意確實對應 `snappedNow`
- 而不是 `snappedNow + samplingInterval`

依目前 `alignAndZipInMemory()` 的資料排列方式來看，較合理的推論是：

- historical input 的最後一個完整 slot 是 `snappedNow` 之前一格
- 下一個尚未觀測到的 slot 應落在 `snappedNow`

但這仍屬於 **從 NWDAF 端推論出的語意**，不是目前程式內明文保證的 contract。

### 4.3 Direction B: 加入 ML `ts` 校驗

本次不一定要直接讓 ML `predicted_data[].ts` 成為唯一真相來源，
但建議至少加入下列 guard：

- 若 `pred.Ts` 可 parse
- 則與 NWDAF 本地推導的 target slot 做比對
- 若不一致則記 warning log

這能避免未來模型端與 NWDAF 端各自演化後，時間語意靜默分叉。

---

## 5. 風險評估

### 5.1 最大風險：改錯模型 step semantics

若 ML model 的 step 0 實際代表「下一個完整 interval」，
則把 `(i+1)` 改成 `i` 會把 pairing 反而往前錯一格。

因此本次修改前，至少要確認：

- 訓練資料標籤的定義
- ML service `predicted_data[].ts` 是否可作為旁證

### 5.2 資料到達延遲風險

修正後 prediction 會比現在更早成熟一個 slot。

可能的影響：

- 若 UPF report 到達有額外延遲
- 現有 `2 * samplingInterval` grace 可能剛好夠，也可能過緊

此風險不一定要在本次一併改 config，但至少要在修正後驗證。

### 5.3 外部相容性風險

若只修 internal `TargetTime`：

- 外部 API schema 不變
- consumer contract 風險低

若同時修改對外 `ts/commDur` semantics：

- 會變成 notification contract change
- 風險顯著提高
- 而且需要明確區分：這是在處理 **既有 aggregated notify semantics 問題**，不是在修補本次 internal fix 造成的新回歸

因此不建議綁在同一批修改。

### 5.4 測試缺口風險

目前測試主要驗：

- monitor/store 如何使用 `TargetTime`
- 但沒有直接驗 `TargetTime` 如何產生
- 也沒有直接驗 `UE_COMMUNICATION` notify 的時間語意

因此這次修正若不補測試，未來很容易再回歸。

---

## 6. 修改難度評估

### 6.1 只修 internal `TargetTime`

- 難度：低到中
- code 改動量：小
- 主要成本：釐清 model step semantics、補測試、做實際驗證

### 6.2 加上 ML `ts` 校驗

- 難度：中
- 需要處理 timestamp parse、格式錯誤、fallback 行為

### 6.3 連對外 notify 一起重構

- 難度：中到高
- 牽涉通知 contract、consumer 相容性、測試面與文件面同步

---

## 7. 建議落地步驟

### Step 1. 確認 step 0 語意

至少確認以下其中之一：

- 訓練/推論資料文件能證明第 1 個輸出 step 對應 `snappedNow`
- 或 ML `predicted_data[].ts` 與 `snappedNow` 對得上

若無法確認，先不要直接改 code。

### Step 2. 最小修正 internal `TargetTime`

修改 `internal/anlf/analytics.go`：

- 將 `(i+1)` 改為 `i`
- 保留其他 monitor / store 邏輯不變

### Step 3. 補測試

至少補下列測試：

1. `generateMlBasedUeCommunication()` 在 1-step prediction 下建立的 `TargetTime`
   - 應對到 `snappedNow`

2. accuracy monitor pairing 測試
   - 驗證修正後 `TargetTime` 能對到正確 ground truth slot

3. maturity timing 測試
   - 驗證修正後 prediction 不會過早消耗、也不會永遠 pending

4. ML `predicted_data[].ts` 校驗測試
   - 若本次實作了 warning guard，需補 parse success / mismatch case

### Step 4. 實測驗證

修正後至少要觀察：

- mature prediction 的時間點是否合理
- matched pair 數量是否下降
- ground truth hit rate 是否正常
- metric 是否出現不合理跳動

---

## 8. 建議後續 follow-up

本次 internal fix 完成後，建議另開第二份計畫處理 external semantics：

- `ueComms[]` 是否改為 per-step list
- `ueComms[i].ts/commDur` 是否改為 slot-level 表達
- `EventNotification.start/expiry/timeStampGen` 的 report-level 分工
- aggregated horizon summary 是否仍需要保留，以及保留形式

補充判斷：

- 若系統長期維持 `outputWindow=1`，則這個 follow-up 的急迫性相對低
- 若 roadmap 上可能恢復 multi-step prediction，則這個 follow-up 應提前，否則 external notify 會持續無法完整表達多個 predicted slots

這些問題屬於 API / spec interpretation 層級，不應與本次 internal pairing fix 混在同一個 patch。

---

## 9. 建議結論

建議採取兩階段策略：

### Phase 1

- 先修 internal `TargetTime`
- 補測試
- 視需要加入 ML `predicted_data[].ts` warning guard

### Phase 2

- 再處理對外 `UE_COMMUNICATION` notify 的時間語意重整

這樣可以先把最直接影響 accuracy monitor 與 retrain semantics 的問題收斂，
同時避免把同一批修改擴成外部通知 contract 變更。

---

## 10. Branch Landing Strategy

目前本地 branch 狀態：

- feature branch: `feat/retrain-monitoring-followups`
- mainline branch: `master`

本 patch 預期需要部署到兩個 branch，因此建議採用 **master-first** 的方式落地，避免把一個小型 bugfix 綁死在進行中的 feature branch 上。

### 建議方式

1. 從 `master` 開一個短命修正 branch
2. 在該 branch 完成實作與驗證
3. 先合回 `master`
4. 再將同一份修正 commit `cherry-pick` 到 `feat/retrain-monitoring-followups`

### 原因

- 這個 patch 本質上是獨立 bugfix，不依賴 `feat/retrain-monitoring-followups` 專屬功能
- 先進 `master`，可讓主線更早獲得修正，也避免未來 feature branch 延長壽命時造成主線遺漏
- 再以 `cherry-pick` 帶回 feature branch，可以只同步這個 patch，不必整包引入 `master` 的其他變更
- commit 脈絡會比「先在 feature branch 做，再回灌主線」更清楚

### 注意事項

- 將修正整理成單一 commit 或一小組相鄰 commits，後續 `cherry-pick` 會最省事
- 若 patch 只涉及 `internal/anlf`、`internal/context`、測試與文件，理論上雙分支衝突風險低
- 若 feature branch 之後本來就要同步主線，仍可在 `cherry-pick` 之後視情況再 `merge master` 或 rebase；但這不應取代本 patch 的獨立同步
- 若未來 external notify semantics 也要修，應視為獨立 patch，再同樣同步到兩個 branch，避免把 internal fix 與 API semantics change 混成單一佈署單位
