# UE Communication Prediction `TargetTime` / Notification Time Semantics Mismatch

本文件記錄在 `UE_COMMUNICATION` 預測路徑中觀察到的時間語意問題。

本 issue 與先前的 `UPF startTime` 錯誤不同。即使 `UPF startTime` 已修正，
目前 `NWDAF` 內部 prediction record 的 `TargetTime`，以及對外送出的
`UE_COMMUNICATION` notify，仍存在一組彼此不一致的時間語意。

---

## 結論摘要

目前行為有兩個核心問題：

1. `TargetTime` 很可能存在 **off-by-one**。
   對 1-step prediction 而言，程式目前把第一個預測點對到
   `snappedNow + samplingInterval`，這會使「當前完整 interval」被略過，
   實際上等於從「下一個完整 interval」才開始預測。

2. 對外 `UE_COMMUNICATION` notify 的時間語意與內部 prediction 語意不一致。
   內部 prediction record 是 per-step、per-slot；
   對外卻送成單一 `ueComm`，且 `ts=now`、`commDur=整個 horizon`，
   這會讓 consumer 以為預測涵蓋 `now ~ now+commDur`，
   但內部實際對應的卻是某個 future slot 或多個 future slots。

其中最重要、最直接的實際風險是：

3. `accuracy monitor` 的 prediction/actual **pairing 會對錯 slot**。
   一旦 `TargetTime` 被往後推一格，monitor 就會拿本應屬於 slot `N`
   的 prediction，去和 slot `N+1` 的 actual 做比對；
   即使模型本身預測正確，pair 仍可能被錯配，導致 metrics 與 retrain
   trigger 判斷一起失真。

最短結論：

- 這不是只影響 `accuracy monitor`
- 它同時影響：
  - prediction record 的意義
  - ground truth matching 與 prediction/actual pairing
  - NWDAF 對外 analytics notify 的時間語意

---

## 問題背景

目前 `NWDAF` 以 `samplingInterval` 為粒度，從近期歷史流量建立模型輸入，
再透過 ML service 取得 `predictedData`。

相關程式位置：

- `internal/anlf/analytics.go`
- `internal/anlf/monitor.go`
- `internal/notifier/handler.go`
- `internal/notifier/notifier.go`
- `internal/context/accuracy_store.go`

目前專案實際使用情境以 **1-step prediction** 為主，因此第 1 個預測點的時間定義
是否正確，會直接決定整體語意是否成立。

---

## 目前實作

### 1. 內部 `TargetTime` 的設定

`internal/anlf/analytics.go`

```go
TargetTime: snappedNow.Add(time.Duration((i+1)*samplingInterval) * time.Second),
```

這代表：

- step 0 -> `snappedNow + 1*samplingInterval`
- step 1 -> `snappedNow + 2*samplingInterval`
- ...

若目前只做 1-step prediction，則唯一一筆 prediction 的 `TargetTime`
會被設成「下一個完整 interval 的起點」。

### 2. Accuracy monitor 的 ground truth matching

`internal/anlf/monitor.go`

ground truth lookup 直接用 `pred.TargetTime` 去找對應實際資料。

因此只要 `TargetTime` 被往後推一格，
accuracy monitor 就會對到「下一格」而不是「當前完整格」。

### 3. 對外 `UE_COMMUNICATION` notify 的生成

`internal/anlf/analytics.go`

目前會把所有 `predictedData` 聚合成一筆：

```go
return models.UeCommunication{
    CommDur: commDur,
    Ts:      &now,
    TrafChar: &models.TrafficCharacterization{
        Dnn:   dnn,
        UlVol: totalUl,
        DlVol: totalDl,
    },
    Confidence: avgConfidence,
}, nil
```

這表示目前對外送出的時間語意是：

- `ts = now`
- `commDur = outputWindow * samplingInterval`
- `ulVol/dlVol = 所有 prediction steps 的總和`

也就是 consumer 看到後，會自然理解成：

`這是一段從 ts 開始、持續 commDur 的預測`

---

## 為什麼這是問題

### 問題一：第 1 個預測點被往後推一格

如果目前的模型語意是：

- 使用最近 `inputWindow` 個完整 historical periods
- 預測「下一個要發生的 period」

那麼對 1-step prediction 而言，第一個預測點更合理的起點應該是：

- `TargetTime = snappedNow`

而不是：

- `TargetTime = snappedNow + samplingInterval`

也就是說，現行 `(i+1)` 很像是一個 **off-by-one**。

白話講：

- 現在的實作會略過 `snappedNow ~ snappedNow+samplingInterval` 這個完整區間
- 預測直接從下一個完整區間開始

如果這不是刻意設計，而只是 indexing 選錯，則目前語意是偏掉的。

對 `accuracy monitor` 而言，這不只是「預測區間被略過」而已，
而是更嚴重的：

- stored prediction record 的 `TargetTime` 會指向錯的 actual slot
- pairing 會從一開始就對錯
- 後續算出的 MAE / sMAPE / MSE / WAPE / NRMSE 都會建立在錯配 pair 上

也就是說，這會直接污染 monitor 的輸入資料，而不是只影響報表解讀。

### 問題二：內外時間語意不同步

內部 record 的語意比較像：

- 一筆 prediction = 一個 future slot
- slot start = `TargetTime`
- slot duration = `samplingInterval`

但外部 notify 的語意卻是：

- 一筆 `ueComm`
- `ts = now`
- `commDur = 整個 horizon`

因此兩者形成衝突：

- 內部：per-slot prediction
- 外部：one-shot aggregated interval prediction

對 consumer 而言，這會造成誤解：

- consumer 看到 `ts=now, commDur=25s`
- 會理解成：`now ~ now+25s` 的預測
- 但內部實際第一個 prediction 可能對的是 `snappedNow` 或 `snappedNow+5s`

這是明確的時間語意落差。

---

## Spec 對照

### 1. `UeCommunication` 本身是 time-slot oriented

`.agent/specs/TS 23.288/6.7 UE related analytics.md`

`UE Communication Statistics` 寫的是：

- `UE communications (1..max)`: list of communication time slots
- `Start time / Duration`

`UE Communication Predictions` 寫的是：

- `UE communications (1..max)`: list of predicted communication time slots

雖然 prediction 那張表沒有再次逐欄列出 `Start time / Duration`，
但整體結構仍然是「一串 predicted communication time slots」。

### 2. `TS 29.520` / OpenAPI 對 `UeCommunication` 的欄位

`.agent/specs/yaml/TS29520_Nnwdaf_EventsSubscription.yaml`

`UeCommunication` 需要：

- `commDur`
- `trafChar`
- `ts` 或 `recurringTime`

這很強地暗示：

- `ts` 是該筆 `UeCommunication` 的時間定位
- `commDur` 是該筆 communication 的 duration

因此較自然的解讀就是：

- consumer 會把一筆 `ueComm` 理解為 `ts ~ ts+commDur`

### 3. Event-level `start` / `expiry`

`EventNotification` 層級另有：

- `start`: start time for stats/predictions
- `expiry`: expiration time for stats/predictions
- `timeStampGen`: timestamp of analytics generation

spec note 還寫：

- `expiry` 不可早於 `start`
- validity period 是 target period 的子集

因此較合理的分工應是：

- `EventNotification.start/expiry`: 這整份 analytics 的有效期間
- `EventNotification.timeStampGen`: analytics 生成時間
- `ueComms[i].ts/commDur`: 每個 predicted slot 的開始時間與長度

目前實作沒有善用這層分工。

---

## 具體時間例子

假設：

- `samplingInterval = 5s`
- `inputWindow = 30`
- 現在時間落在某個已對齊的推論時刻
- 只預測 1 步

### 目前實作

- internal `TargetTime = snappedNow + 5s`
- notify `ts = now`
- notify `commDur = 5s`

這代表：

- internal 認為預測對的是下一個完整 period
- external 看起來卻像從現在開始預測 5 秒

### 較一致的做法

若模型第 1 步本來就是預測當前完整 period：

- internal `TargetTime = snappedNow`
- notify `ueComms[0].ts = snappedNow`
- notify `ueComms[0].commDur = 5s`
- `EventNotification.timeStampGen = now`

若模型刻意預測下一個完整 period：

- internal `TargetTime = snappedNow + 5s`
- notify `ueComms[0].ts = snappedNow + 5s`
- notify `ueComms[0].commDur = 5s`
- `EventNotification.timeStampGen = now`

不論採哪一種，都不應該再送成：

- `ueComms[0].ts = now`
- `ueComms[0].commDur = horizon`
- `trafChar = 所有 future steps 加總`

因為這會混淆 slot-level 與 report-level 語意。

---

## 影響範圍

這個問題不只影響 `accuracy monitor`。

### 1. Prediction record / accuracy path

受影響檔案：

- `internal/anlf/analytics.go`
- `internal/context/accuracy_store.go`
- `internal/anlf/monitor.go`

影響：

- `TargetTime` 可能對錯 slot
- mature timing 與 ground truth matching 會一起偏移
- prediction/actual pairing 會直接錯配
- metric 計算與 retrain trigger 判斷會因此失真

### 2. 對外 notify 語意

受影響檔案：

- `internal/anlf/analytics.go`
- `internal/notifier/handler.go`
- `internal/notifier/notifier.go`

影響：

- consumer 會收到與內部實際 prediction slot 不一致的時間描述
- `ts` / `commDur` 容易被誤解為 `now ~ now+commDur`

### 3. 下游 consumer 對 analytics 的解讀

若有 NF / AF / test harness 根據 `ts` / `commDur` 解讀預測時間窗，
則目前結果可能在時間上向前或向後錯位。

---

## 建議修正方向

### 本次修正範圍決議

本次只打算依照 **修正方向 A** 進行：

- 修正 internal `TargetTime`

以下項目 **本次不處理**，先保留為後續議題：

- 修正方向 B：對外 `ueComms` 改為 per-step 表達
- 修正方向 C：補齊 / 重新定義 `EventNotification.start/expiry/timeStampGen`

也就是說，本次修正目標是先讓：

- internal prediction record
- ground truth matching / prediction-actual pairing
- accuracy monitor

在時間對位上恢復一致。

而對外 `UE_COMMUNICATION` notify 的時間語意落差，先作為已知限制保留。

### 修正方向 A：修正 internal `TargetTime`

若目前 1-step prediction 的正確語意是「預測當前完整 interval」，
則建議將：

```go
snappedNow + (i+1)*samplingInterval
```

改成：

```go
snappedNow + i*samplingInterval
```

對目前只做 1-step prediction 的情況，相當於：

- step 0 -> `TargetTime = snappedNow`

### 修正方向 B：對外 `ueComms` 改為 per-step 表達

本方向目前 **不納入本次修正**。

比 spec 語意更自然的做法應是：

- 一個 prediction step 對應一筆 `ueComm`
- `ueComms[i].ts = 該 step 的 TargetTime`
- `ueComms[i].commDur = samplingInterval`
- `ueComms[i].trafChar = 該 step 的 UL/DL`
- `ueComms[i].confidence = 該 step 的 confidence`

而不是把多步 prediction 壓扁成一筆 aggregated communication。

### 修正方向 C：report-level 時間另用 `EventNotification`

本方向目前 **不納入本次修正**。

若需要表達整份 analytics 的有效期，可使用：

- `eventNotification.start`
- `eventNotification.expiry`
- `eventNotification.timeStampGen`

建議分工：

- `timeStampGen = now`
- `start = 第一個 predicted slot 的開始時間`
- `expiry = 最後一個 predicted slot 的結束時間`

但 per-slot 時間仍應由 `ueComms[].ts/commDur` 承擔。

---

## 建議驗證項目

修正後至少應驗證：

1. 1-step prediction 的 `TargetTime` 是否符合模型實際預測語意
2. accuracy monitor 是否對到正確 ground truth slot
3. 對外 notify 的 `ts` / `commDur` 是否能讓 consumer 唯一解讀
4. 若未來恢復 multi-step prediction，`ueComms` 是否能逐 step 正確表示時間

---

## 建議後續決策

建議先明確回答以下問題，再動手修：

1. 目前 ML model 的第 1 個輸出 step，語意上到底是：
   - 當前完整 interval
   - 還是下一個完整 interval

2. `UE_COMMUNICATION` 對外到底要表達：
   - 一串 predicted communication slots
   - 還是一段 aggregated horizon summary

若答案是前者，則目前實作很可能需要修正。  
若答案是後者，則至少要重新定義 `ts` / `commDur` 與 internal `TargetTime` 的對應，
否則時間語意仍然不一致。
