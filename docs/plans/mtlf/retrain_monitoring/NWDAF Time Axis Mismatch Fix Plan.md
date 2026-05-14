# NWDAF Time-Axis Mismatch Fix Plan

> 對應 issue：`.agent/docs/issues/architecture/nwdaf-time-axis-mismatch-issue-2026-05-14.md`
>
> 本文件整理 `UE_COMMUNICATION` prediction 在 NWDAF 內部與對外通知之間的時間軸錯位，並提出修正步驟。
>
> 本次 commit 目標是一次完成時間軸與 pairing 相關修正；不再拆成「最小修正」與後續 follow-up 兩階段。
> `commDur` 先視為既有設計，不納入本次 issue。

---

## 1. 問題摘要

目前專案同時存在兩套時間體系：

- `startTime` 體系：UPF measurement slot 的開始時間
- `now` / `snappedNow` 體系：NWDAF 做 inference 與發 callback 的牆鐘時間

這導致同一筆 prediction 同時有三個不同時間：

- 歷史輸入最後一格的 measurement slot time
- accuracy monitor 使用的 `PredictionRecord.TargetTime`
- callback 對外送出的 `ueComm.ts`

一旦 `latest aggregated slot` 與 `now` 之間有 gap，prediction/actual pairing 與 consumer 看到的時間都會錯位。

本次 commit 的完成定義是：

- callback `ueComm.ts` 修正到 `startTime` 體系
- internal `PredictionRecord.TargetTime` 修正到同一條時間軸
- pred/actual pairing 不再依賴 `TargetTime ± samplingInterval + nearest`
- store / monitor 不再依賴現有單次 mature consume 模型
- 單元測試與必要文件同步完成

### 1.1 Current Status

截至目前，本計畫對應的核心實作已完成，狀態如下：

- `analytics` 已改為從最後一個 historical slot 推導 `baseTargetTime`
- callback `ueComm.ts` 已改為使用 `baseTargetTime`
- internal `PredictionRecord.TargetTime` 已改為使用 `baseTargetTime + i * samplingInterval`
- `PredictionRecord` 已新增 `TargetSlotTime`、`MissCount` 與 store-assigned `ID`
- accuracy store 已從 `ConsumeMaturePredictions()` 改為 snapshot + resolve 模型
- accuracy monitor 已從 `TargetTime ± si + nearest` 改為 slot-equality pairing
- pending prediction lifecycle 已改為 full-scan + retry/discard
- 對應單元測試已更新
- 驗證已完成：`go test ./...`、`make build`、`make lint`

目前仍明確不在本次修改範圍內的項目：

- `commDur` 語意重設
- 對外 `ueComms[]` 改為 per-step 表達
- ML service `predicted_data[].ts` 改為可直接信任的絕對時間

---

## 2. 已確認的語意

### 2.1 `ts` 應回到 `startTime` 體系

依 TS 29.520 `UeCommunication` 語意，`ts` 應表示該 communication interval 的 start time。

對本專案目前的 traffic prediction prototype，最接近且可落地的映射是：

- `ueComm.ts` = 這次 prediction 實際回答的第一個 future slot start time
- `PredictionRecord.TargetTime(step 0)` = 同一個時間點

### 2.2 Step index 從 0 開始

本次統一採用：

```text
latestHistoricalSlotTime = 這次實際送進模型的歷史資料中，最後一個有效 slot 時間
baseTargetTime           = latestHistoricalSlotTime + samplingInterval
TargetTime(step i)       = baseTargetTime + i * samplingInterval
```

其中：

- `i = 0` 對應第一個 future slot
- `i = 1` 對應第二個 future slot

狀態：

- 已落地實作
- `predictionTargetTime()` 目前直接以 `baseTargetTime` 為 step 0

### 2.3 `commDur` 暫不列入本次 issue

本次先接受目前聚合語意：

- 一筆 `UeCommunication` 代表從 `ts` 開始的一段 prediction window summary
- `commDur = outputWindow * samplingInterval`
- `trafChar.ulVol/dlVol` 代表該 window 內聚合後的總流量

若未來需要將單一 `UeCommunication` 改為更嚴格的 per-step 或 per-episode 表達，再另開 follow-up。

### 2.4 Confirmed Decisions

本次討論後已確認並採納的決策如下：

- `ts` 應回到 `startTime` 體系，不再使用 callback 發送當下的 `now`
- `step 0` 的語意是「這次輸入歷史最後一格之下一個 slot」
- `baseTargetTime = latestHistoricalSlotTime + samplingInterval`
- `TargetTime` 保留 prediction 語意用途
- pairing 用途需拆到獨立 slot 欄位，不讓 `TargetTime` 同時承擔語意與配對兩種角色
- pred/actual pairing 不再使用 `TargetTime ± samplingInterval + nearest`
- pred/actual 應映射到同一個 slot identity，再以 slot equality 配對
- `globalIndex` 的思想可沿用，但必須從 `snappedNow` 解耦
- monitor/store 不再依賴單次 mature consume
- 改採 pending full-scan + retry/discard
- miss/discard 的設計仍保留現有 `2 * samplingInterval` 直覺，作為等待 actual 的容忍時間參考

---

## 3. 目前實作確認

### 3.1 歷史輸入資料屬於 `startTime` 體系

UPF ingest 目前以 `startTime` 作為 measurement timestamp，只有缺值才 fallback 到 `timeStamp`。

因此：

- in-memory `UpfDataPoint.Timestamp`
- MongoDB `UpfTrafficRecord.Timestamp`
- accuracy monitor ground truth lookup 的 actual timestamp

本質上都已經是 `startTime` 體系。

### 3.2 Prediction output 仍使用 `now` / `snappedNow`

目前：

- `ueComm.ts = now`
- `PredictionRecord.TargetTime = predictionTargetTime(snappedNow, samplingInterval, i)`

這是本次主要修正點。

狀態：

- 已修正
- callback `ts` 與 internal prediction target 均已改為從 historical slot 推導

### 3.3 ML service 回傳的 `predicted_data[].ts` 目前沒有參考價值

`.agent/NWDAF-ML-Service` 目前的 predictor 直接回傳：

```text
ts="predicted_t+1"
```

這是 placeholder，不是真正可用的絕對時間，也不能作為 NWDAF anchor 的真相來源。

因此本次 `ts` / `TargetTime` 必須由 NWDAF 根據歷史資料自行推導。

---

## 4. 設計決策

### 4.1 Anchor 應來自歷史資料最後一格，而不是 `snappedNow`

本次統一規則：

- 不再以 `snappedNow` 定義第 0 步 prediction 的 target
- 改以「這次實際送進模型的最後一個有效 historical slot」作為基準

也就是：

```text
last historical slot = T
first predicted slot = T + samplingInterval
```

### 4.2 `ueComm.ts` 與 `PredictionRecord.TargetTime` 必須共用同一個 base

不能只修 callback `ts` 或只修 accuracy monitor。

本次要求：

- callback `ueComm.ts`
- internal `PredictionRecord.TargetTime`
- monitor 的 mature / ground truth pairing

都必須建立在相同的 `baseTargetTime` 上。

狀態：

- 已完成

### 4.3 `TargetTime` 不應再同時承擔配對格線角色

目前 `TargetTime` 在 monitor 中同時被當成：

- prediction 語意上指向的 target slot time
- actual lookup 的查詢中心
- mature timing 的時間依據
- accuracy report window 的時間依據

這讓 `TargetTime` 一旦定錯，整條 monitor 鏈都會一起錯。

後續設計應將兩種角色拆開：

- `TargetTime`：prediction 真正回答的 future slot start time
- `TargetSlotTime` / `TargetSlotKey`：拿來做 pred/actual slot-level pairing 的格線時間或格線識別

本次 commit 直接將 pairing 用途拆到獨立欄位，不再留待 follow-up。

命名上暫不定案：

- `SnappedTargetTime`
- `TargetSlotTime`
- `TargetSlotKey`

其中較建議優先考慮 `TargetSlotTime` / `TargetSlotKey`，避免與現有 `snappedNow` 的 wall-clock 語意混淆。

狀態：

- 目前已採用 `TargetSlotTime`
- 尚未引入獨立的 `TargetSlotKey` 欄位；目前 pairing 先以 `TargetSlotTime` 經 slot 映射後完成

### 4.4 `PredictedAt` 保持不變

`PredictedAt` 仍表示「什麼時候做出 prediction」。

因此：

- `PredictedAt = now` 保留
- 不將 `PredictedAt` 與 slot target time 混用

### 4.5 `latestHistoricalSlotTime` 的定義需明文化

目前 `alignAndZipInMemory()` 輸出的 `TrafficObservation.Ts` 是代表性 slot time：

- 有真實資料的 slot：使用 contributing IP centers 的平均值
- 零補 slot：使用從對齊格點推導出的時間

本次先採以下實作定義：

- `latestHistoricalSlotTime = historicalData[len(historicalData)-1].Ts`

這個定義雖然不是「全域絕對 grid start」，但已經明顯比 `snappedNow` 更接近 measurement-slot 語意，足以修正本 issue 的主體錯位。

若未來需要更嚴格的 canonical slot start，再考慮讓 aligner 額外回傳專用 anchor。

---

### 4.6 現有 `globalIndex` 思路可延伸成正式 slot identity

目前專案尚未正式定義 `slotKey`，但 historical aggregation 已經有隱含的 slot identity：

- per-IP `centerUnix`
- cross-IP `globalIndex`

這代表本次不一定要重新發明一套新概念；可以沿用既有思路，將：

```text
globalIndex = round((center - snappedNow) / si)
```

抽象成更一般化的 measurement-slot-based slot identity。

重要限制：

- 不能直接沿用目前相對 `snappedNow` 的 `globalIndex`
- 必須先把基準從 wall-clock snapped grid 改為 `startTime` 體系下的共同基準線

也就是後續應收斂成：

- pred 有 `TargetSlotKey`
- actual 有 `ActualSlotKey`
- pairing 只接受 `pred.slotKey == actual.slotKey`

狀態：

- 已採納其核心思路
- 目前實作沒有額外 materialize `TargetSlotKey` / `ActualSlotKey` 欄位
- 目前做法是用 `TargetSlotTime` 加 `slotKeyForTime(...)` 在 lookup 時映射到同一個 slot identity

### 4.7 pairing 不應停留在 `TargetTime ± samplingInterval + nearest`

目前 monitor 的配對方式是容錯式最近鄰：

- 以 `TargetTime` 為中心
- 查 `±samplingInterval`
- 選最近 actual

這雖然能容忍 jitter，但會帶來誤命中風險：

- `TargetTime` 若本身偏移，會整批配到錯 slot
- 窗口內若同時存在前後鄰格 actual，可能被 nearest 選錯
- group / 多 corrId 情況下，不同 corrId 甚至可能各自命中不同 slot 後再被錯誤聚合

因此本次 commit 直接納入：

- 將 pred / actual 都先映射到同一個 slot identity
- 再用 slot equality 配對
- 不再將 `±si` nearest 作為最終配對機制

狀態：

- 已完成

## 5. 具體修改範圍

### 5.1 `internal/anlf/analytics.go`

目標：

- 由 `historicalData` 算出 `latestHistoricalSlotTime`
- 產生 `baseTargetTime`
- 讓對外 `ueComm.ts` 與 internal `TargetTime` 使用同一組時間軸

預計修改：

1. 取得 `last := historicalData[len(historicalData)-1]`
2. parse `last.Ts`
3. `baseTargetTime = lastTs + samplingInterval`
4. 新的 helper：

```go
func predictionTargetTime(baseTargetTime time.Time, samplingInterval, step int) time.Time {
    return baseTargetTime.Add(time.Duration(step*samplingInterval) * time.Second)
}
```

5. `PredictionRecord.TargetTime` 改為使用 `baseTargetTime`
6. aggregated `models.UeCommunication.Ts` 改為 `baseTargetTime`

注意：

- 若 `last.Ts` parse 失敗，應明確記 log 並 fallback；fallback 策略須保守，避免 silent wrong time
- 不建議再把 helper 命名或註解寫成 `snappedNow` 語意

狀態：

- 已完成
- 目前 fallback 是 parse 失敗時記 warning 並退回既有 `snappedNow`

### 5.2 `internal/anlf/monitor.go`

本次討論已確認，現有 monitor 邏輯有兩個深層問題：

1. `TargetTime ± samplingInterval + nearest` 存在誤命中風險
2. `mature` / consume 機制過度依賴目前的時間定義

因此本次 commit 直接納入：

- 新增 `TargetSlotTime` / `TargetSlotKey`
- actual 端也 materialize 對應 slot identity
- pred/actual 改用同 slot 配對
- 重新設計 pending prediction 的生命週期管理

狀態：

- 已完成
- pairing 已改為 slot-equality
- in-memory 與 Mongo 路徑都已同步

### 5.3 `internal/context/accuracy_store.go`

目前 store 的 pending prediction 管理模型是：

- 加入 prediction
- 直到 `TargetTime + graceAfterTarget < now`
- 被 `ConsumeMaturePredictions()` 取出
- 本輪若沒配到 actual，就直接被消耗掉

這個模型的問題是：

- 內部隱含了 `TargetTime` 與 wall-clock 的耦合
- prediction 一旦在 consume 當輪 miss，就不再保留待後續重試
- 無法良好支援以 slot identity 為基礎的 repeated matching

本次 commit 採用的方向是：

- 不再把 pending prediction 分成單次 consume 的 mature / immature
- 改成每輪掃描整個 pending list 嘗試配對
- 配到就移除
- 沒配到就保留
- 連續 miss 多次才丟棄

也就是從：

```text
pending -> mature -> consume once
```

改成：

```text
pending -> try match every round -> matched or discarded
```

狀態：

- 已完成
- 目前 store 已改為：
  - `SnapshotPredictions()`
  - `ResolvePredictions(...)`
  - `DiscardAllPredictions()`

### 5.4 pending prediction retry / discard 機制

根據目前討論，本次 commit 採用的方向是：

- 每輪 monitor 都對所有 pending prediction 嘗試配對 actual
- prediction 不再依賴單一 mature 時間點才被檢查
- 每筆 prediction 維護 miss counter
- 若連續多輪都沒命中 actual，才將其丟棄以釋放空間

這個設計的理由：

- 可以避免現行「consume 當輪 miss 就永久消失」
- 若 actual 因延遲稍後才進入 DB / memory，後續仍有機會補命中
- 不需讓 prediction semantics 與 wall-clock `deadline` 過度糾纏

需要注意：

- miss counter 不能是脫離時間尺度的裸常數
- `checkInterval` 不同，代表每次 miss 的等待時間不同
- 因此 counter 門檻仍須由 `samplingInterval` / `checkInterval` 推導

狀態：

- 已完成
- 目前 `maxMissCount` 由 `samplingInterval` 與 `checkInterval` 推導

### 5.5 現行 `2 * samplingInterval` 思路的保留價值

目前 mature grace 設為 `2 * samplingInterval` 的直觀語意是：

- `1 * si`：先等 prediction 指向的 target slot 本身完整過去
- `1 * si`：再給 UPF notify 最壞延遲一格時間

雖然本次 commit 傾向取消現有 mature consume 機制，
但這個設計直覺仍保留，作為 miss counter / discard policy 的設計參考。

例如後續可由此推導：

- prediction 至少應允許跨越多少次 monitor retry
- 多久還沒命中 actual 才算合理丟棄

狀態：

- 已採用
- 目前 retry/discard 門檻就是以這個直覺為推導基礎

### 5.6 `internal/anlf/analytics_test.go`

需更新或新增測試，至少覆蓋：

- `step 0` 不再等於 `snappedNow`
- `predictionTargetTime()` 改以 `baseTargetTime` 為 anchor
- `baseTargetTime = last historical slot + samplingInterval`
- 若後續新增 `TargetSlotKey` / `TargetSlotTime`，其產生規則需有獨立測試

現有將 `step 0 == snappedNow` 視為正確的測試必須改寫。

狀態：

- 已完成

### 5.7 `internal/anlf/monitor_test.go` / `internal/context/accuracy_store_test.go`

重點不是全面改邏輯，而是補足：

- mature timing 仍按 `TargetTime` 判斷
- ground truth pairing 對新的 target slot 仍成立
- 若改成 pending full-scan + retry/discard，需重寫 store / monitor 測試基線
- 若引入 `TargetSlotKey` / `ActualSlotKey`，需新增 slot-equality pairing 測試

狀態：

- 已完成
- 已新增「鄰格不應誤命中」與「late ground truth 可後續補命中」測試

### 5.8 文件更新

至少同步以下內容：

- issue 對應 plan link
- 若有對外測試/示例文件明示 `Timestamp` 語意，需更新為「prediction window start time」，避免仍被誤解為 callback 發送時間
- 若 pairing 從 nearest 改成 slot key，相關設計文件需同步記錄 slot identity contract

---

## 6. 建議實作步驟

### Step 1. 在 analytics 路徑引入 `baseTargetTime`

- 從 `historicalData` 最後一格取得 `lastTs`
- 明確建立 `baseTargetTime`
- 將 helper 與內部命名改為以 `baseTargetTime` 為中心

### Step 2. 統一 callback `ts` 與 internal `TargetTime`

- `ueComm.ts = baseTargetTime`
- per-step `PredictionRecord.TargetTime = baseTargetTime + i*si`

### Step 3. 定義 slot identity 並落地

- 盤點現有 `centerUnix` / `globalIndex` 可重用之處
- 決定 `TargetSlotTime` / `TargetSlotKey` 命名與資料模型
- 定義 pred / actual 共用的 measurement-slot 基準線
- 讓 pred 與 actual 都能 materialize 到同一個 slot identity

### Step 4. 重設 monitor pending lifecycle

- 評估是否保留現有 `ConsumeMaturePredictions()`
- 若改為 pending full-scan + retry/discard，定義：
  - 每輪如何掃描
  - miss counter 怎麼累計
  - discard 門檻如何由 `samplingInterval` / `checkInterval` 推導

### Step 5. 將 pairing 改為 slot equality

- monitor 不再以 `TargetTime ± si` nearest 作為最終配對
- 以 pred / actual 的 slot identity 做一對一配對
- group / 多 corrId 情況下先落格再聚合，避免混格

### Step 6. 修正單元測試

- 更新 `analytics_test.go`
- 驗證 monitor 與 store 不再依賴舊 `snappedNow` 假設

### Step 7. 做實際 log 驗證

驗證案例：

- `latest aggregated slot = 03:40:40`
- `samplingInterval = 30s`
- 預期 callback `ts = 03:41:10`
- 預期 `TargetTime(step 0) = 03:41:10`

且不應再出現：

- callback `ts = inference now`
- `TargetTime = snappedNow`

### 6.1 Progress Snapshot

目前實際完成度可整理為：

- Step 1：完成
- Step 2：完成
- Step 3：完成
- Step 4：完成
- Step 5：完成
- Step 6：完成
- Step 7：程式層與單元測試層驗證完成；若需要再補 deployment log 驗證，可作為額外觀察項

---

## 7. 風險與注意事項

### 7.1 `last.Ts` 是 representative slot time，不是嚴格 canonical grid

這是本次方案有意接受的近似。

判斷理由：

- 它仍屬於 measurement-slot 體系
- 比 wall-clock `snappedNow` 更接近 prediction 實際回答的時間
- 能以較小改動修正主要錯位

### 7.2 slot identity 設計若定義不穩，整條 pairing 仍可能整批錯位

本次會將 nearest-time pairing 改為 slot-equality pairing；
因此風險中心會從「窗口誤命中」轉成「slot identity 是否定義正確」。

需要特別驗證：

- pred 與 actual 使用的是同一套 measurement-slot 基準線
- multi-corrId 落格後不會跨 slot 混合
- zero-padded / representative Ts 不會污染 target slot identity

目前狀態：

- 已以單元測試覆蓋主要路徑
- production-like replay / log 驗證仍值得後續觀察

### 7.3 Parse failure 的 fallback 要保守

若 `historicalData` 最後一格 `Ts` 無法 parse：

- 不能靜默回退成 `now`
- 應至少打 warning/error log
- fallback 行為需明確標註，避免難以追查

建議 fallback 優先序：

1. 嘗試 parse `last.Ts`
2. 若失敗，記錄 log 並回退到既有 `snappedNow` 路徑
3. 後續再觀察是否需要將 parse failure 視為 hard error

### 7.4 pending full-scan + miss counter 仍會受 monitor 週期影響

若後續改成：

- 每輪嘗試配對全部 pending pred
- miss 多次才丟棄

則丟棄策略會受 `checkInterval` 影響。

因此：

- `maxMissCount` 不應脫離 monitor 週期單獨設常數
- 同樣的 miss 次數，在不同 `checkInterval` 下代表不同等待時間

### 7.5 ML service `ts` 目前不可作為依據

除非 `.agent/NWDAF-ML-Service` 後續也同步改成輸出真正的絕對時間，
否則 NWDAF 不應依賴 `predicted_data[].ts` 做本次修正。

---

## 8. Out of Scope

以下不納入本次修改：

- 重新定義 `commDur`
- 將 aggregated `UeCommunication` 改為 per-step `ueComms[]`
- 要求 ML service 輸出可直接信任的絕對 `ts`
- 為 aligner 額外設計 canonical slot-start 專用結構

這些項目若後續需要，另開 follow-up。

---

## 9. 預期完成後的狀態

修正完成後，應達成：

- prediction history、callback `ueComm.ts`、accuracy monitor `TargetTime` 三者回到同一條 measurement-slot 時間軸
- `step 0` 明確表示「這次輸入歷史最後一格之下一個 slot」
- prediction/actual pairing 不再受 `latest aggregated slot` 與 `now` 的 gap 影響而整批錯位
- pred/actual pairing 改為 slot-identity equality，不再依賴 `±si` nearest
- 現有 mature consume 機制改為 pending full-scan + retry/discard

本次修正不追求一次解決所有 `UE_COMMUNICATION` 語意問題，但應先把最核心的時間軸錯位消除。

### 9.1 Verification Status

目前已完成的驗證：

- `go test ./...`
- `make build`
- `make lint`

結果：

- build 通過
- lint 通過
- 全量 Go 測試通過
