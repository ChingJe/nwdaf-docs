# NWDAF Retrain / Monitoring — Implementation Plan

> 對應策略文件：`NWDAF Retrain Monitoring Strategy.md`
>
> 本文件以「實作模組」為主軸描述要做的事；每個模組都包含目的、設計原因、實作要點、注意事項與任務清單。看完這份可直接進入 coding 階段。
>
> 本階段實作需遵循 NWDAF 專案開發規範：`.agent/docs/development_policy.md`
>
> 進度、branch、merge 與 blocker 維護改由 `NWDAF Retrain Monitoring Workflow and Progress.md` 負責。
> 本文件中的任務條列只保留設計拆項，不再作為進度勾選來源。

---

## 1. 背景與目標

### 1.1 問題

現行 retrain 觸發策略高度依賴單一 metric（sMAPE）＋ 固定 threshold，遇到以下情境表現不穩：

- 低流量 / near-zero 時 sMAPE 震盪大，容易誤判
- Threshold 用固定值，跨不同 monitoring scope 的 traffic pattern 調不起來
- 所有 scope 的樣本被混成單一 model-level aggregate，某個退化 scope 會被其他正常 scope 稀釋
- Metric 計算、degradation policy、retrain execution 目前耦合太緊，不利於和 TS 29.520 / TS 23.288 的角色分工對齊

### 1.2 設計核心原則

- **Metric 先不定死**：MAE / MSE / WAPE / NRMSE / sMAPE 都列候選，全部同步計算；Checkpoint A 僅觀測，Checkpoint B 再切 primary metric
- **Scope-level 與 model-level 分離**：metric 先在 monitoring scope 分開算；model-level retrain decision 只看「任一 scope 退化」
- **Monitoring scope 以 subscription / Target UE 語意定義**：scope key 來自 analytics consumer 的 `TargetUe`，兼容 group 與 direct SUPI；不是由 `corrId` 或事後 traffic grouping 反推
- **AnLF / MTLF 角色切開**：AnLF 負責 accuracy monitoring 與 metric 計算；MTLF 負責 baseline、degradation policy、retrain 決策與執行
- **Drift 建立在 recent baseline**：每條 scope 保留 recent metric ring buffer，以 mean / std / z-score 當 drift statistic
- **Retrain 需通過兩層 gate**：
  - 第一層（absolute）：error 本身夠高
  - 第二層（relative）：相對於 recent baseline 明顯偏離
  - 兩層 AND 才算該 scope 本輪觸發；再連續 N 輪觸發才真正 retrain
- **可觀測 / 可除錯**：每輪把統計值與 raw pred/actual 寫入 CSV，方便離線畫圖、選 metric、調 threshold

### 1.3 本文件範圍

- In-scope：scope-level 分桶、AnLF→MTLF accuracy report、recent baseline、兩層 gate、cold start 保護、觀測 CSV（暫時性）、config 擴充
- Out-of-scope：訂閱 filter / AoI 細粒度拆分為額外 scope key（保留至未來版本）
- CSV 是**過渡期觀察工具**：待 threshold / primary metric 調穩後可移除；不規劃升級到 Prometheus / Mongo

---

## 2. 現況基準

| 項目 | 位置 | 現狀 |
|------|------|------|
| Metric | `internal/anlf/monitor.go: computeSMAPE` | 只有 sMAPE，單一值 |
| Aggregate 層級 | `checkModelAccuracy` | 所有 mature pairs 混成一個 model-level 值 |
| Store | `internal/context/accuracy_store.go: ModelAccuracyStore` | per-model；無 per-scope 狀態，無歷史 buffer |
| Decision | `internal/mtlf/trigger.go` | `consecutive` / `ema` 兩種單層絕對門檻策略 |
| Cold start | `AccuracyMonitorConfig.WarmupDuration` | 僅 model-level 一次性 warmup；無 per-scope 保護 |
| Config | `pkg/factory/config.go: AccuracyMonitorConfig` | `DeviationThreshold / MinSamples / WarmupDuration / EmaAlpha / ConsecutiveBreaches` |
| Scope 候選資料 | `PredictionRecord.NwdafSubId`、`NwdafSubResource.OriginalGroupId` | 已有 subscription 與 group 關聯資訊，但尚未 materialize 成 immutable scope key |
| Retrain 重入保護 | `store.IsRetraining` | 在 flight 時 skip；hot-swap 完會走新 monitor 的 WarmupDuration（見 §6.4） |

---

## 3. 整體架構

### 3.1 資料流

```
ML Service prediction
        │
        ▼
AddPrediction(PredictionRecord: ScopeKey, NwdafSubId, TargetTime, PredUl, PredDl)
        │
        ▼  （每 checkInterval 秒）
ConsumeMaturePredictions → lookupGroundTruth
        │
        ▼
                    ┌── group by ScopeKey (= canonical TargetUe scope) ──┐
                    │                                    │
                    ▼                                    ▼
      compute metrics for scope A         compute metrics for scope B
      {sMAPE, MAE, MSE, WAPE, NRMSE}       {sMAPE, MAE, MSE, WAPE, NRMSE}
                    │                                    │
                    ▼                                    ▼
          build AccuracyReport                 build AccuracyReport
          (current metrics only)               (current metrics only)
                    │                                    │
                    └────────── notify AnLF → MTLF ──────┘
                                  │
                                  ▼
                      MTLF PathState.RecordMetric
                      (ring buffer + mean/std + breach)
                                  │
                                  ▼
                    two-layer gate on primary per scope
                                  │
                    └─────────── OR ─────────────────────┘
                                  │
                                  ▼
                consecutive N rounds any-scope triggered
                                  │
                                  ▼
                          startRetrainWorkflow
```

### 3.2 新增 / 改動的組件

| 組件 | 所在 package | 新/改 |
|------|--------------|------|
| `computeMAE / MSE / WAPE / NRMSE` | `internal/anlf` | 新 |
| `MetricResult`（或 `map[string]float64`） | `internal/anlf` | 新 |
| `ScopeKey` materialization（subscription `TargetUe` → canonical scope） | `internal/anlf` / `internal/context` | 新 |
| `AccuracyReport`（AnLF → MTLF） | `internal/anlf` / `internal/mtlf` | 新 |
| `PathState` / `ScopeState`（ring buffer + stats） | `internal/mtlf` | 新 |
| `MonitorStateStore` | `internal/mtlf` | 新 |
| 兩層 gate decision | `internal/mtlf/trigger.go` | 改寫 |
| Cold start 保護（minBufferSamples、minStd） | `internal/mtlf/trigger.go` | 新邏輯 |
| CSV dumper（metrics.csv + pairs.csv） | `internal/anlf` 或獨立 pkg | 新 |
| Config 欄位 | `pkg/factory/config.go` | 新增/移除 |

---

## 4. 實作模組

### 4.1 Metric 候選集合

**目的**：產出同一批樣本的多個 metric 值，讓觀察期可以離線比較各 metric 的穩定性與可解釋性，再決定 primary。

**為什麼這樣設計**
- sMAPE 在低流量 near-zero 不穩；MAE 單位直覺但量綱依流量而異；WAPE / NRMSE 各有 trade-off
- 過早 lock 單一 metric 會讓後續調 threshold 變成猜測遊戲
- 同一批樣本同時算多個 metric 幾乎沒成本，但離線分析價值高

**實作要點**
- 候選集合：`sMAPE`、`MAE`、`MSE`、`WAPE`、`NRMSE`
- 單一入口 `computeAll(pairs []matchedPair) map[string]float64`，回傳每個 metric 的值
- 保留現有 `computeSMAPE` 作為其中一支，簽章一致
- 全部 metric 都算、都進 CSV、都進 buffer
- Primary metric **預計在 Checkpoint B 先用 `MAE`** 驅動 decision（理由：單位直覺、低流量不像 sMAPE 那樣爆）；其餘 metric 只當觀察用
- 日後若 CSV 分析顯示其他 metric 更適合，只需改 config `primaryMetric`

**注意事項**
- `MSE` 量綱是 bytes²，很容易跨 scope 尺度爆炸 → CSV 照寫，但不建議當 primary
- `WAPE` 分母是 `sum(|actual|)`，若整段都 0 要能安全回傳 0 而非 NaN
- `NRMSE` 分母選用 range 或 mean 要固定，單元測試時也要鎖定
- 所有 metric 在 `actual=0 AND pred=0` 的 sample 都視為 0 貢獻，並保留在 recent baseline / buffer 中（見 §4.6）

**任務**
- 實作 `computeMAE / computeMSE / computeWAPE / computeNRMSE`
- `computeAll` 統一入口
- Perfect / zero / large-error / mixed 的單元測試（每 metric 至少 4 case）

---

### 4.2 Monitoring Scope 定義

**目的**：把同一 model 下不同 subscription target scope 的樣本分開計算 metric，避免退化 scope 被稀釋。

**為什麼這樣設計**
- Monitor 掛在 model 上（一個 model 一個 store），但同一 model 可能服務多個不同 consumer target，實際退化通常是「某個 target scope」而不是整體
- scope 直接對齊 subscription `TargetUe` 語意，兼容 group 與 SUPI，且與 TS 29.520 `Nnwdaf_MLModelMonitor_Subscribe/Notify` 的 `tgtUe` 欄位一致
- 先分 scope、再在 model-level 做 OR 邏輯（任一 scope 退化就 retrain），比「全體平均」更貼近真實需求

**決策結論**
- 文件中的 `path` 一詞統一指 **monitoring scope**，不是 `corrId`、不是 network path
- **Scope key 由 subscription `TargetUe` canonicalize 而來**：
  - 單一 group：`group:<groupId>`
  - 多個 group：`groups:<sorted-groupIds>`
  - 單一 SUPI：`supi:<supi>`
  - 多個 SUPI：`supis:<sorted-supis>`
- `NwdafSubId` 與 `corrId` 不作為最終 scope key；它們只作關聯與 ground truth lookup
- AoI / analytics filter 細粒度拆分保留到未來版本（§6.1）

**實作要點**
- 新增 `resolveMonitoringScope(...)`，以 subscription 原始 target 語意產生 canonical scope key
- `PredictionRecord` 在建立當下就 materialize `ScopeKey`；**不要**在 monitor 階段再從 live subscription 反查，避免 subscription 更新/刪除造成同一筆 prediction scope 漂移
- 若現階段無法直接從原始 subscription object 取得 canonical target，允許用現有 context 資料組裝等價 scope，但產生後仍要 snapshot 到 `PredictionRecord`
- `checkModelAccuracy`：
  1. consume mature predictions
  2. 對每個 pred lookup ground truth
  3. **依 ScopeKey 分桶**
  4. 每桶分別呼叫 `computeAll`
- Model-level aggregate 保留作為 debug log，不再進 decision

**注意事項**
- Scope key 抽象成 string；未來若加 AoI 或 filter subset，擴充 canonicalization 規則即可
- direct SUPI subscription **必須支援**；不能因為沒有 groupId 就跳過
- 若 target 組合當前無法 canonicalize，該 prediction 建議 `skip + log`，不要退回 model-level 混算

**任務**
- `PredictionRecord` 新增 `ScopeKey string`
- `resolveMonitoringScope(...) string` 實作
- `checkModelAccuracy` 改寫為 per-scope compute
- 單元測試：group 與 SUPI scope 各自計算正確、互不干擾

---

### 4.3 AnLF → MTLF Accuracy Report 介面

**目的**：把 AnLF 的「accuracy computation」和 MTLF 的「degradation policy」切開，對齊 TS 29.520 / TS 23.288 的責任分工。

**為什麼這樣設計**
- TS 29.520 `Nnwdaf_MLModelMonitor` / TS 23.288 6.2E 的理想流程是：AnLF 監測並通知 accuracy information；MTLF 根據通知決定 model 是否 degraded、是否 retrain
- 現況 `onDeviationReport(modelUrl, deviation, store)` 只傳單一 float，不足以承載 scope-level 多 metric 資訊
- 明確定義內部 `AccuracyReport` 後，後續若要對接正式 `Nnwdaf_MLModelMonitor_Notify`，資料模型也比較平順

**實作要點**
- 定義內部 `AccuracyReport`（命名可調整）：
  ```go
  type AccuracyReport struct {
      ModelURL      string
      ScopeKey      string
      NwdafSubID    string
      Metrics       map[string]float64
      SampleCount   int
      InferenceNum  int
      WindowStart   time.Time
      WindowEnd     time.Time
      AccuMeetInd   *bool // optional, for future TS 29.520 alignment
  }
  ```
- AnLF 在 `checkModelAccuracy` 完成 per-scope metric 後產生一批 reports，透過 callback 通知 MTLF
- MTLF 只依 report 做 policy evaluation，不再依賴 AnLF 內部 state

**生命週期（決策）**
- `AccuracyReport` 是單輪快照，不保留歷史
- 歷史 baseline 與 breach counter 由 MTLF 自己維護（見 §4.4）
- Hot-swap 後 AnLF 繼續送 report；MTLF 端是否清 state 取決於 modelUrl 是否更換

**注意事項**
- 內部 callback 雖然不是標準 SBI，但欄位語意應盡量對齊 `MLModelMonitorNotify` / `MLModelAccuracyInfo`：model、metric、deviation/accuracy snapshot、`tgtUe` 對應的 scope context、`inferenceNum`
- 若後續要導入正式 monitor subscription，MTLF 既有 decision 邏輯應可直接吃同一份資料結構或薄轉換層

**任務**
- 設計 `AccuracyReport`
- 改寫 `onDeviationReport` callback 為 report-based 介面
- 測試 AnLF 正確送出 per-scope reports

---

### 4.4 Per-scope recent buffer & baseline 統計（MTLF 擁有）

**目的**：由 MTLF 替每條 monitoring scope 維護近期 metric 歷史，供 degradation policy 與 retrain 決策使用。

**為什麼這樣設計**
- 根據規格，AnLF 應負責監測與通知 accuracy information；MTLF 應根據多次通知與本地 policy 決定 degradation / retraining
- baseline、breach counter、global decision 都是 retrain policy 的一部分，邏輯上應放在 MTLF

**實作要點**
- 新型別 `ScopeState`（建議放 `internal/mtlf`）：
  ```go
  type ScopeState struct {
      scopeKey     string
      buffers      map[string]*ringBuffer
      reportCounts  map[string]int
      lastUpdate   time.Time
      breach       int
      // mutex
  }
  ```
- 另有 per-model 的 `MonitorStateStore`：
  - key: `modelUrl`
  - value: `map[scopeKey]*ScopeState`
- Ring buffer 容量由 config `recentBufferSize` 控制（預設 20）
- 方法：
  - `RecordMetric(metric string, value float64, isValid bool)`
  - `Mean(metric) / Std(metric) / ZScore(metric, current)`
  - `SampleCount(metric) int`

**生命週期（決策）**
- **建立**：某 scope 首次收到 `AccuracyReport` 時 lazy create
- **清除**：被動 GC — `lastUpdate > TTL`（預設 10 min）由 MTLF decision loop / report handling 順便掃描
- Hot-swap 時以 `modelUrl` 為邊界，新 modelUrl 對應新 state；舊 modelUrl state 可直接刪除

**注意事項**
- 多 goroutine 可能同時收到多份 report、跑 retrain callback、做 GC → concurrency 是重點
- Ring buffer 用 slice 實作，`append` + modulo 即可，不必引外部套件
- 統計值（mean/std）每次 on-demand 算，不快取（buffer 小、不值得）

**任務**
- `ScopeState` 型別 + ring buffer
- `MonitorStateStore.GetOrCreateScope(modelUrl, scopeKey)`
- 定期 GC
- Concurrency 測試（`go test -race`）

---

### 4.5 兩層 retrain decision

**目的**：只有「error 夠高」且「相對 baseline 明顯偏離」同時成立時才視為某個 scope 觸發，再加連續 N 輪 dampen 偶發波動。

**為什麼這樣設計**
- 只看絕對門檻：對不同 scope 流量差異無能為力
- 只看 z-score：當 baseline 很低且很穩時，小幅變差也可能被放大成高 z-score
- 兩層 AND + 連續 N 輪：能同時要求「業務上夠大」與「統計上夠異常」，也能擋掉偶發 spike

**判斷邏輯**
```
absGate     = current > fixedFloor
zScore      = (current - mean) / max(std, minStd)
relGate     = zScore > zScoreThreshold
scopeTriggered = absGate AND relGate

if scopeTriggered:
    scope.breach += 1
else:
    scope.breach = 0

if any scope.breach >= consecutiveBreaches:
    startRetrainWorkflow()
```

**決策結論**
- 兩層保留、AND 觸發
- 第一層只檢查「當前 error 本身是否夠大」；第二層只檢查「相對 baseline 是否明顯異常」
- Primary metric：**MAE**（測試期間可能調整）
- `fixedFloor` 預設值先暫定（見 §4.8），測試階段用 CSV 觀察後再回填
- `zScoreThreshold` 先用預設 `3.0`，測試階段調整
- `consecutiveBreaches` 沿用現有欄位（預設 3）
- EMA 策略整個移除
- 沿用 model-level retrain in-flight guard，不變

**注意事項**
- Cold start：buffer 填充未達 `minBufferSamples` 時只走 `absGate`，跳過 `relGate`（見 §4.6）
- `mean` 不做額外保護；若 baseline 很低導致 relative 判斷過敏，優先透過 `fixedFloor` 擋下
- 多 scope 之中只要任一觸發就 retrain，不做「多 scope 同時觸發才算」的 AND
- 觸發之後立刻 `store.SetRetraining(true)` 並呼叫 `startRetrainWorkflow`；同 model 下所有 scope 的 breach 同時歸零
- Log 格式建議標註每層結果：`scope=X metric=Y cur=... mean=... std=... z=... absGate=T zGate=F breach=2/3`

**任務**
- 移除 `TriggerStrategy / EmaAlpha / checkEMATrigger`
- 改寫 `HandleDeviationReport` 為 report-based per-scope two-layer gate
- Model-level OR + consecutive counter
- 單元測試：
  - 低 error + 高 z：不觸發（absGate 擋）
  - 高 error + 低 z：不觸發（zGate 擋）
  - 兩層皆過 + 連續 3 輪：觸發
  - 中間一輪低於 threshold：breach 歸零

---

### 4.6 Cold start 保護（策略文件未寫）

**目的**：避免 buffer 還沒填滿、std 還沒穩定時出現爆 z-score 的 false positive。

**為什麼要補**
- 策略文件假設 baseline 已建立；實際上：
  - 每個新 scope 第一次收到樣本時 buffer 是空的
  - 前幾筆樣本進來時 std 極小，z-score 會炸
  - 低流量 scope 可能很久才累積到有效樣本
- Model-level 已有 `WarmupDuration` 處理 AnLF monitor 啟動時的冷啟動，但 MTLF 的 per-scope baseline 冷啟動仍要獨立處理

**保護機制**

| 機制 | 作用 |
|------|------|
| `minBufferSamples`（預設 8） | recent buffer 樣本數未達門檻前，decision 只走 `absGate`，跳過 `relGate` |
| `minStd`（預設 0.01） | 算 z-score 時以 `max(std, minStd)` 防止 std 太小把小波動放大成高 z-score |
| both-zero 保留 | `actual=0 AND pred=0` 的 sample 視為真實觀測，保留在 recent buffer 與 baseline 中；是否 retrain 交由 `fixedFloor + z-score + consecutiveBreaches` 決定 |
| no-ground-truth 跳過 | `lookupGroundTruth` 返回 nil 時完全跳過（現行為），不入 buffer |

**注意事項**
- `minBufferSamples` 與 config `minSamples`（每輪最少 matched pairs 才評估）是不同層概念，不要混淆
- Post hot-swap 靠現有 `WarmupDuration` 處理 model 層級冷啟動（見 §6.4），scope-level 仍靠上述機制
- Cold start 期間的 CSV 資料照常寫（方便觀察 warmup 行為）

**任務**
- `minBufferSamples` gate 加進 §4.5 decision 邏輯
- `Std` 計算內建 `max(std, minStd)`（或在 `ZScore` 內套）
- `both-zero` 樣本照常傳入 `RecordMetric`
- 單元測試：buffer=0~7 時只判 absGate；buffer=8 起啟用 z-score

---

### 4.7 觀測資料落地（CSV）

**目的**：觀察期離線分析用 — 選 metric、調 threshold、畫 pred vs actual 時序圖。

**定位**
- **過渡期工具**：目的是幫助測試期間選 metric、調 threshold；功能穩定後可移除
- 不規劃升級到 Prometheus / Mongo

**為什麼用 CSV**
- 最小侵入（不動現有 logger / DB）
- 直接 pandas 讀取，轉 plot 容易
- 移除時只要砍 writer、拿掉兩個 config 欄位

**兩個檔案**

**(a) `metrics_<ts>.csv`** — 每輪 × 每 scope × 每 metric 一列

欄位：
```
timestamp, model, scope, metric, current, mean, std, min, max,
cv, zeroInflation, slope, absDev, zscore, sampleCount
```

衍生欄位說明：
- `cv = std / mean`（變異係數，scale-invariant）
- `zeroInflation`：`both-zero` sample 比例（看 metric 對低流量的暴露）
- `slope`：recent buffer 的簡單線性回歸斜率（漸進劣化 vs 突變）
- `absDev = |current - mean|`
- `zscore = (current - mean) / max(std, minStd)`

**(b) `pairs_<ts>.csv`** — 每個 matched pair 一列

欄位：
```
checkTime, model, scope, nwdafSubId, predictedAt, targetTime,
predUl, actualUl, predDl, actualDl
```

用途：
- 直接畫 UL / DL 的 pred vs actual 時序圖
- 對照 metrics.csv 的異常時間點查 raw 行為
- 若 `lookupGroundTruth` 沒對到，actualUl / actualDl 留空仍寫入一列（便於看 match rate）

**檔案策略**
- 每次 NWDAF 啟動建新檔（時間戳後綴），不覆寫、不 rotate
- 路徑：`<csvDumpDir>/metrics_<YYYYMMDD_HHMMSS>.csv` / `<csvDumpDir>/pairs_<ts>.csv`
- `csvDumpEnabled=false` 時完全不寫
- Flush 策略：每輪 `checkInterval` 跑完後 flush 一次（頻率低，不必緩衝）

**注意事項**
- 正式環境若 disk I/O 有顧慮可透過 `csvDumpEnabled` 關掉
- Scope 名稱若含特殊字元（`,`、`"`），寫入前要逃脫（用 `encoding/csv` 即可）
- Shutdown 時確保 flush + close，避免尾端資料遺失（hook 到 `CancelContext` 的 Done）

**任務**
- CSV writer（共用 `encoding/csv`），封裝開檔 / append / close
- Metrics writer：在 `checkModelAccuracy` 計算完後寫一批
- Pairs writer：在 match 結束後寫一批
- Shutdown flush（掛 monitor loop 的 ctx.Done）

---

### 4.8 Config 擴充

**目標**：反映新的 decision 結構與觀測能力；同時移除 EMA 相關殘留。

**`AccuracyMonitorConfig` 變更清單**

**沿用**
- `enabled`、`checkInterval`、`minSamples`、`warmupDuration`、`consecutiveBreaches`

**移除**
- `triggerStrategy`（新版為唯一策略，不再切換）
- `emaAlpha`（EMA 整個拿掉）
- `deviationThreshold`（改用 `fixedFloor + z-score`）

**新增**

| 欄位 | 型別 | 說明 | 預設（暫定） |
|------|------|------|------|
| `metricsToRecord` | `[]string` | 候選 metric，CSV 都會寫 | `[sMAPE, MAE, MSE, WAPE, NRMSE]` |
| `primaryMetric` | `string` | Checkpoint B 起進 decision 的 metric | `MAE` |
| `recentBufferSize` | `int` | per-scope ring buffer 長度 | 20 |
| `minBufferSamples` | `int` | 啟用 z-score 前的最少有效樣本數 | 8 |
| `minStd` | `float64` | std floor，避免 /0 | 0.01 |
| `fixedFloor` | `float64` | 第一層絕對下限（MAE 單位：bytes） | 先暫定（測試後調整） |
| `zScoreThreshold` | `float64` | 第二層 z-score 門檻 | 3.0 |
| `scopeStateTTL` | `int`（秒） | scope state 被動 GC 時間 | 600 |
| `csvDumpDir` | `string` | 觀察 CSV 輸出目錄（過渡期） | `log/accuracy/` |
| `csvDumpEnabled` | `bool` | 是否寫 CSV（過渡期功能，穩定後可移除） | `true` |

**注意事項**
- `fixedFloor` 依 primary metric 單位（MAE: bytes）先給一個暫定值不卡住實作；測試期間看 CSV 再調
- Checkpoint A 保留舊 config 的 `deviationThreshold / emaAlpha / triggerStrategy`；Checkpoint B 才在 `config/nwdafcfg.yaml` 同步清掉

**任務**
- 修改 `pkg/factory/config.go: AccuracyMonitorConfig`
- 同步更新 `config/nwdafcfg.yaml`
- 預設值驗證：missing 欄位 fallback 合理

---

### 4.9 測試策略

**單元測試**

| 層級 | 內容 |
|------|------|
| Metric | 每個 metric 的 perfect / zero / large / mixed case |
| Scope canonicalization | 單一 SUPI、多 SUPI、單一 group、多 group、排序正規化、unsupported target 組合 |
| Scope snapshot | `PredictionRecord.ScopeKey` 在 prediction 建立後不可漂移；subscription 後續修改/刪除不影響既有 prediction |
| Scope 分桶 | group 與 SUPI scope 分別退化，只有退化者被計入 |
| AccuracyReport | AnLF 產出報告欄位正確、可被 MTLF 消費；空 metrics / 缺 primary metric / `SampleCount=0` 的錯誤案例明確處理 |
| Checkpoint A 相容性 | 新增 scope / metrics / CSV / report 後，既有 sMAPE threshold decision 不回歸 |
| Baseline | Ring buffer 溢位後舊值被捨棄、mean/std 正確 |
| No ground truth / both-zero | no-ground-truth 不入 baseline；both-zero 進 recent buffer，且 report / CSV 行為符合設計 |
| Cold start | Buffer 0~7 只走 absGate；8 起啟用 z；std=0 時不爆 |
| Two-layer gate | `fixedFloor` 能擋下「error 很小但 z 很高」；`minStd` 能擋下「std 太小造成 z 爆高」；連續 N 輪邏輯正確 |
| Retrain lifecycle | in-flight retrain 時新 report 被忽略；failure 會清 retrain guard；觸發後 breach 歸零；post-swap 舊 model state 清理 |
| Concurrency | `RecordMetric` / `HandleDeviationReport` / GC 並發不 race |
| GC | scope 被動清理 `lastUpdate > TTL` |
| CSV | 欄位齊全、shutdown flush |

**整合驗證（在 Milestone 1 / 2 分段做）**
- Milestone 1：跑一段時間後離線分析 CSV（metric 相關係數、scope 分布差異）
- Milestone 2：Staging 跑一段時間，對比 Milestone 1 的 log 確認 retrain 觸發時機合理
- Milestone 2：驗證 hot-swap 後 monitor / baseline / retrain guard 狀態符合預期，不會立即重複觸發 retrain

---

## 5. 實作順序

**原則**：不以 milestone 嚴格劃分「完整測試後才進下一階段」；每個 checkpoint 可獨立合入，threshold / primary metric 等「測試才能決定的值」先用暫定預設值（見 §4.8），跑起來之後再按 CSV 調整。

### Checkpoint A — Metric 候選 + Scope 分桶 + CSV 觀察

**目標**：先把「能看到東西」的管線打通；舊 decision 仍在運作。

**實作**
1. Metric 候選集合（§4.1）— 此階段僅觀測，不改 decision
2. Monitoring scope 定義與 `PredictionRecord.ScopeKey`（§4.2）
3. `AccuracyReport` 介面（§4.3）
4. CSV dump：`metrics_<ts>.csv` + `pairs_<ts>.csv`（§4.7）
5. 既有 `consecutive` + `ema` decision 保持原樣（此階段不碰 retrain policy）

**驗收**
- 系統跑起來後 CSV 有資料
- log 能看到每個 scope 的多 metric 值
- 舊 decision 仍以既有 sMAPE threshold 運作
- `ScopeKey` canonicalization 與 snapshot 測試全過
- `AccuracyReport` 能被 MTLF 消費，但不改變既有 retrain 行為

---

### Checkpoint B — Per-scope baseline + 兩層 gate + cold start

**目標**：切換到新 decision 邏輯；不等 A 的 CSV 分析完，先用暫定 threshold。

**實作**
1. MTLF per-scope state + recent buffer（§4.4）
2. 兩層 gate（§4.5）
3. Cold start 保護（§4.6）
4. Config 擴充、移除 EMA 與舊 threshold（§4.8）
5. scope state 被動 GC（§4.4）
6. 單元測試（§4.9）

**暫定值**（可在 A 的 CSV 累積後再回頭調）
- `primaryMetric = MAE`
- `zScoreThreshold = 3.0`
- `fixedFloor` 依實測 MAE 量級先估（例如觀察幾輪後取大致 median 當參考）
- `consecutiveBreaches = 3`、`recentBufferSize = 20`、`minBufferSamples = 8`

**驗收**
- 單元測試全過（含 `-race`）
- retrain lifecycle 測試全過（in-flight skip / failure reset / post-swap cleanup）
- 實跑不誤觸（誤觸就調 CSV 再調參）

---

### Checkpoint C — 調參 / 清理

**目標**：根據 A + B 的運作資料微調 threshold 與 primary metric；穩定後清掉觀察性質的程式碼。

**內容**
- 按 CSV 調 `primaryMetric` / `fixedFloor` / `zScoreThreshold`
- 確認穩定後移除 CSV dump（§4.6 定位為過渡期工具）
- 舊程式碼 / 註解清理 / 文件化

---

## 6. 未決 / 待補

### 6.1 AoI / analytics filter 作為更細粒度 scope key
目前 scope 先以 subscription `TargetUe` 為主。TS 23.288 / TS 29.520 的 analytics filter、AoI 等更細粒度語意保留至未來版本。Milestone 2 完成後的下一個題目。

### 6.2 策略文件是否同步補 cold start 段
策略文件假設 baseline 已建立，沒寫 cold start；決定是否回補到該文件，或只在本 plan 保留。

### 6.3 CSV flush 細節
目前預設每輪 flush；若 scope 數 × metric 數過多導致 I/O 壓力，再改成緩衝 N 筆或每 T 秒。

### 6.4 Post-swap 額外 grace
Model-level 已靠現有 `WarmupDuration` 處理（`swapModelAfterRetrain` → `onModelSwapped` → `StartAccuracyMonitorForModel` → `runModelAccuracyLoop` 起始的 warmup，`internal/anlf/monitor.go:81-92`、`internal/sbi/processor/processor.go:54,100`、`internal/mtlf/training.go:236`）。目前判斷不必新增 `postSwapGraceDuration`；若 Checkpoint B 觀察到 retrain 完成後立刻又被觸發的情形，再補。

### 6.5 `pairs.csv` 是否寫無 ground truth 的列
初版先寫（看 match rate），量太大再移除。

### 6.6 `fixedFloor` 的具體數字（MAE 單位）
先給一個暫定值不卡住實作；跑起來後看 CSV 調整。Checkpoint C 正式定案。

### 6.7 CSV 移除時機
定位為過渡期工具。當 Checkpoint C 確認 primary metric 與 threshold 穩定、線上運作無異常後即可移除（砍 writer + 兩個 config 欄位）。
