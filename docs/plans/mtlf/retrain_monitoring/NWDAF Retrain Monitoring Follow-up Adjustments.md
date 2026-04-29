# NWDAF Retrain / Monitoring — Follow-up Adjustments

> 對應主文件：`NWDAF Retrain Monitoring Implementation Plan.md`
>
> 本文件記錄 Checkpoint B/C 落地後，根據 `tools/retrain_replay` 實驗與報表觀察所整理出的後續調整方向。
> 它的定位是 **補充設計說明**，用來承接已合併設計之外的新發現與未定案問題。

---

## 1. 目的與範圍

本文件處理三類 follow-up 調整：

- `minSamples` 的語意修正
- degradation path 的 low-traffic 保護
- frozen reference baseline 的設計問題釐清
- metric history buffer 的資料結構調整
- low-traffic overprediction 類型的補充 decision path

這些項目都會影響 retrain decision semantics，但目前不應直接覆蓋既有 Checkpoint B 設計說明而不留下決策脈絡，因此以補充文件管理。

---

## 2. 已收斂的調整方向

### 2.1 `minSamples` 改為 per-scope gate

**調整原因**

- 現行 round-total gate 只保證「整個 monitor round 有足夠成熟 pair」，不保證單一 scope 的 metric 足夠穩定。
- 在 per-scope policy 下，真正參與 retrain decision 的單位是 scope；因此 `minSamples` 應直接約束該 scope 的 `SampleCount`。
- replay 觀察到單一 scope 只有 `sampleCount=1` 時，仍可能因其他 scope 補足 round-total 而進入 policy evaluation，這會放大 WAPE 等 normalized metric 的噪音。

**新語意**

- `minSamples` 代表：**單一 monitoring scope 在單一 monitor round 中，至少要有多少 matched pairs，才允許該 scope 進入 accuracy-policy evaluation**。
- 若某 scope `SampleCount < minSamples`：
  - 該 scope 的 raw monitor result 仍可記錄於 observability
  - 但不得進入 MTLF retrain decision
  - 也不得寫入該 scope 的 degradation/chronic decision history

**影響**

- AnLF / replay tool 都需以 per-scope `SampleCount` 套用 `minSamples`
- `minBufferSamples` 仍維持為「baseline 歷史樣本數門檻」，兩者不能混淆

### 2.2 degradation path 補 `minTrafficScale` eligibility guard

**調整原因**

- WAPE 在 near-zero actual traffic 時不穩定；當 traffic 太小時，即使 error 不大也可能產生高 WAPE 或高 z-score。
- 既有 chronic path 已有 `minTrafficScale` guard，但 degradation path 尚未對 normalized metric 的低流量不可靠區域做保護。

**新語意**

- degradation path 新增獨立設定 `degradationPolicy.minDecisionTrafficScale`
- 此 guard 用來判斷「當輪 primary metric 是否足夠可信，能參與 degradation decision」
- chronic path 的對應命名也應一併對齊為 `chronicPolicy.minDecisionTrafficScale`
- 兩者都屬於 decision eligibility，但仍分別服務不同 path，不能混成單一共用欄位

**建議輸入值**

- degradation path 應以 **當輪 `AccuracyReport.TrafficScale`** 判斷 eligibility
- chronic path 維持使用 recent traffic-scale mean，因為 chronic 判斷的是持續性 operating region，而不是單輪 metric 可不可信

**低於門檻時的合理行為**

- 保留 observability：`current`、`trafficScale`、必要時 `mean/std/zscore` 仍可記錄
- 不得把該輪視為 degradation hit
- 不得把該輪視為 degradation miss
- 不得推進 degradation decision window
- log / report 語意上應視為 `skipped because traffic is too low`，而不是「模型正常」

這裡的核心原則是：**低流量代表證據不足，不代表沒有退化。**

### 2.3 traffic-scale naming 應做出 path 對稱性

目前 chronic path 的 traffic-scale guard 已是 policy namespace 下的欄位；若 degradation path 新增對應 guard，命名應與 chronic path 對稱。

**建議命名**

- `degradationPolicy.minDecisionTrafficScale`
- `chronicPolicy.minDecisionTrafficScale`

**原因**

- 兩者都是 decision eligibility guard，不是一般 metric metadata
- 後續若再加入其他 path，例如 low-traffic overprediction path，也能延續相同命名風格
- 與單純 `degradationMinTrafficScale` 相比，policy namespace 更能避免欄位散落

---

## 3. degradation reference buffer：比 frozen baseline 更準確的設計方向

replay 結果顯示，單純 sliding baseline 可能在 concept drift 發生後快速吸收異常值，進而壓低後續 z-score。這個問題值得處理，但目前更合理的方向不是「整個 baseline freeze」，而是把 degradation path 使用的 reference history 單獨隔離出來。

### 3.1 為什麼不直接叫 frozen baseline

若設計是「單輪 hit 或 signal 的 observation 不進入 degradation baseline」，那它的語意其實更接近：

- 一條獨立的 degradation reference history
- 一條有 admission rule 的 healthy-only buffer

而不是傳統意義上「填滿後完全不再更新」的 frozen baseline。

因此文件上建議使用：

- `degradation reference buffer`
- 或 `degradation buffer`

而不要先把名稱定成 frozen baseline。

### 3.2 建議 admission rule

若後續導入 degradation reference buffer，較合理的寫入條件是：

- `SampleCount >= minSamples`
- `TrafficScale >= degradationPolicy.minDecisionTrafficScale`
- 該輪 observation 可被視為 valid monitor round
- **若該輪已命中 degradation signal，則不寫入 degradation reference buffer**

這個設計的核心是：**不讓已被判成異常的 observation 反過來重定義 degradation baseline。**

### 3.3 與 recent buffer 的分工

若採此方向，兩種 buffer 應明確分工：

- `recentObservationBuffer`
  - 盡量保留完整 monitor history
  - 服務 chronic path、observability、low-traffic 特例路徑
- `degradationReferenceBuffer`
  - 只服務 degradation path
  - admission rule 較嚴
  - 不吸收 degradation-signal round

---

## 4. metric history 應改為 observation buffer，而不是 metric-to-slice

目前 Go runtime 與 replay tool 都採用 `metric name -> ring buffer` 結構。這對單純算 mean/std 雖然可行，但當 policy 開始需要：

- 依據同一輪的 `SampleCount`
- 同時檢查 actual / predicted traffic scale
- 某些 round 只排除在特定 path 的 reference 外
- 同一輪要作為 chronic / degradation / low-traffic path 的共同輸入

時，`metric -> []float64` 會讓時間對齊與 admission rule 變得過於破碎。

### 4.1 建議方向

改成 per-round observation buffer。每個時間點存一筆 observation，而不是每個 metric 各自一條 slice。

例如：

```go
type ScopeObservation struct {
    Timestamp            time.Time
    SampleCount          int
    TrafficScaleActual   float64
    TrafficScalePred     float64
    Metrics              map[string]float64
}
```

其中：

- `TrafficScaleActual` 對應目前 `AccuracyReport.TrafficScale`
- `TrafficScalePred` 是本次新增建議欄位，用來支撐 low-traffic overprediction 類型的判斷
- `Metrics` 可以先維持 `map[string]float64`，不一定要一次把 metric schema 寫死

### 4.2 直接影響

若 observation buffer 成立，則：

- `recentObservationBuffer` 與 `degradationReferenceBuffer` 可以共用同一種 struct
- path 的差異只反映在 admission rule，不反映在資料格式
- 同一輪的 metric / traffic / sampleCount 不會失去時間對齊

---

## 5. predicted traffic scale：需要正式進入 report / observation contract

若後續要處理 low-traffic 區域的 false negative / false positive，僅有 actual-side 的 `TrafficScale` 不夠。

目前至少需要把 predicted side 的量級也顯式帶進來。

### 5.1 需要新增的資料

建議在 `AccuracyReport` 或等價 observation contract 中新增：

- `PredictedTrafficScale`

若後續有需要，也可再細分成：

- `PredictedTrafficScaleUL`
- `PredictedTrafficScaleDL`

但第一版先有總量級即可。

### 5.2 作用

- 支撐 low-traffic overprediction path
- 區分「actual 很小所以 normalized metric 不可信」與「模型明顯高估流量」
- 讓 chronic / degradation 之外的新 path 不必硬靠 MAE 或 WAPE 代理判斷

---

## 6. chronic low-traffic 補充 path：可考慮 low-traffic overprediction path

現有 chronic path 在 `traffic scale` 太低時會因 eligibility guard 而無法使用。這在 near-zero 區域可避免 normalized metric 誤判，但也會漏掉一類真實問題：

- 實際流量幾乎沒有
- 模型卻持續預測出明顯偏高的流量

這類情況不適合硬塞回原本 chronic signal，比較合理的是獨立成另一條 path。

### 6.1 建議方向

新增一條候選 path，暫稱：

- `lowTrafficOverprediction`

### 6.2 高層規則

這條 path 的 hit 可建立在下列條件之上：

- `actualTrafficScale < <path>.maxActualTrafficScale`
- `predictedTrafficScale > <path>.minPredictedTrafficScale`
- `predictedTrafficScale > <path>.predictionOvershootRatio * max(actualTrafficScale, epsilonBase)`

其中需要同時有：

- actual-side low-traffic 條件
- predicted-side absolute floor
- relative overshoot 條件

避免 actual 接近 0 時只因 ratio 爆炸就誤判。

### 6.3 暫時結論

- 這不是既有 chronic path 的簡單延伸，而是更像一條新 path
- 是否要實作，可列為 follow-up branch 中的候選設計
- 若不打算實作這條 path，則 chronic path 在 low-traffic 區域的 blind spot 必須被文件明確承認

---

## 7. 對主設計文件的同步要求

本文件落地後，主計畫與 workflow 至少要同步下列變更：

- 主 plan：
  - `minSamples` 改為 per-scope gate
  - degradation path eligibility 補入 `degradationPolicy.minDecisionTrafficScale`
  - chronic path 的對應欄位命名也同步調整
  - 若後續採 observation buffer，主 plan 中的 state-store 設計也要一併更新
  - `PredictedTrafficScale` 納入 report / observation contract
  - degradation reference buffer 改列為 follow-up 設計方向，不直接當成已定案 frozen baseline
- workflow / decision history：
  - 記錄 `minSamples` 語意從 round-total 改為 per-scope
  - 記錄 degradation path 新增 low-traffic guard
  - 記錄 traffic-scale naming 對齊 policy namespace
  - 記錄 observation buffer / predicted traffic scale / low-traffic overprediction path 為 follow-up 設計方向

---

## 8. 後續實作拆項

建議後續工作至少拆成三條：

1. 更新文件與 decision history，使 runtime / replay / analysis 的語意先一致
2. 先實作 `minSamples` per-scope 與 degradation `minDecisionTrafficScale` guard
3. 設計並評估 `degradation reference buffer` / observation buffer / predicted traffic scale
4. 視需要另外引入 low-traffic overprediction path，不與前述基礎調整混做

### 8.1 建議的實作切分

若後續要進入 follow-up branch，建議再拆成下列具體 work items：

1. `AccuracyReport` / observation contract 擴充
- 加入 `PredictedTrafficScale`
- 明確 actual / predicted traffic scale 的計算與傳遞
- 補 AnLF 與 replay 對應欄位

2. per-scope gate 調整
- `minSamples` 改成 per-scope `SampleCount` gate
- runtime 與 replay 一起改
- 補對應測試與 log 驗證

3. observation buffer 重構
- `metric -> ring buffer` 改成 per-round `ScopeObservation`
- 先做 `recentObservationBuffer`
- 讓 chronic / observability 先吃這個結構

4. degradation reference buffer
- 新增 `degradationReferenceBuffer`
- 實作 admission rule：
  - `SampleCount >= minSamples`
  - `actualTrafficScale >= degradationPolicy.minDecisionTrafficScale`
  - degradation signal round 不寫入
- 把 degradation z-score 改成吃這個 buffer

5. policy config 命名收斂
- `degradationPolicy.minDecisionTrafficScale`
- `chronicPolicy.minDecisionTrafficScale`
- 補 config fallback、文件、測試

6. chronic path 遷移
- chronic path 改吃 observation buffer
- eligibility 改用 `chronicPolicy.minDecisionTrafficScale`
- 確認 chronic aggregate 邏輯不變

7. low-traffic overprediction path
- 新增 path 與 config
- 使用 `actualTrafficScale + PredictedTrafficScale`
- 補 absolute floor / overshoot ratio 規則
- 補 hit reason、decision window、測試

8. replay / analysis 對齊
- replay tool mirror runtime 語意
- analysis/report 工具補新欄位與新 path 顯示
- 確認 log / trace schema 一致

目前 runtime / replay / analysis 需至少對齊下列 policy observability 欄位：
- `degradationBaselineReady`
- `recentBaselineReady`
- `actualTrafficScale`
- `recentTrafficScaleMean`
- `PredictedTrafficScale`
- `lowTrafficEligible`
- `lowTrafficSignal`
- `lowTrafficOvershootRatio`
- `degradationReferenceSamples`
- `recentSamples`

其中：
- `actualTrafficScale` 應代表當輪 decision 使用的 actual-side traffic scale
- `recentTrafficScaleMean` 應代表 chronic path 使用的 recent observation mean
- analysis/report 若要畫 low-traffic overprediction detail，必須優先使用 `actualTrafficScale` 而不是 recent mean

### 8.2 建議順序

較保守的順序可為：

`1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 8 -> 7`

也就是先把資料契約、gate、state store 與 degradation/chronic 主路徑收斂，再處理 low-traffic overprediction path。
