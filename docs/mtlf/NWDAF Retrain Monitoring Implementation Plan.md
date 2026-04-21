# NWDAF Retrain / Monitoring — Implementation Plan

> 對應策略文件：`NWDAF Retrain Monitoring Strategy.md`
>
> 本文件以「實作模組」為主軸描述要做的事；每個模組都包含目的、設計原因、實作要點、注意事項與任務清單。看完這份可直接進入 coding 階段。

---

## 1. 背景與目標

### 1.1 問題

現行 retrain 觸發策略高度依賴單一 metric（sMAPE）＋ 固定 threshold，遇到以下情境表現不穩：

- 低流量 / near-zero 時 sMAPE 震盪大，容易誤判
- Threshold 用固定值，跨不同 path 的 traffic pattern 調不起來
- 所有 path 的樣本被混成單一 model-level aggregate，某條退化的 path 會被其他正常 path 稀釋
- Metric 本身既負責描述誤差、又負責判 drift、又負責觸發 retrain，三件事混在一層

### 1.2 設計核心原則

- **Metric 先不定死**：MAE / MSE / WAPE / NRMSE / sMAPE 都列候選，全部同步計算；primary 先用 **MAE** 驅動 decision，測試期間再據 CSV 調整
- **Path-level 與 model-level 分離**：metric 先在 path（先用 groupId）分開算；model-level decision 只看「任一 path 退化」
- **Drift 建立在 recent baseline**：每條 path 保留 recent metric ring buffer，以 mean / std / z-score 當 drift statistic
- **Retrain 需通過兩層 gate**：
  - 第一層（absolute）：error 本身夠高
  - 第二層（relative）：相對於 recent baseline 明顯偏離
  - 兩層 AND 才算該 path 本輪觸發；再連續 N 輪觸發才真正 retrain
- **可觀測 / 可除錯**：每輪把統計值與 raw pred/actual 寫入 CSV，方便離線畫圖、選 metric、調 threshold

### 1.3 本文件範圍

- In-scope：path-level 分桶、recent baseline、兩層 gate、cold start 保護、觀測 CSV（暫時性）、config 擴充
- Out-of-scope：訂閱 filter / AoI 作為 path key（保留至未來版本）
- CSV 是**過渡期觀察工具**：待 threshold / primary metric 調穩後可移除；不規劃升級到 Prometheus / Mongo

---

## 2. 現況基準

| 項目 | 位置 | 現狀 |
|------|------|------|
| Metric | `internal/anlf/monitor.go: computeSMAPE` | 只有 sMAPE，單一值 |
| Aggregate 層級 | `checkModelAccuracy` | 所有 mature pairs 混成一個 model-level 值 |
| Store | `internal/context/accuracy_store.go: ModelAccuracyStore` | per-model；無 per-path 狀態，無歷史 buffer |
| Decision | `internal/mtlf/trigger.go` | `consecutive` / `ema` 兩種單層絕對門檻策略 |
| Cold start | `AccuracyMonitorConfig.WarmupDuration` | 僅 model-level 一次性 warmup；無 per-path 保護 |
| Config | `pkg/factory/config.go: AccuracyMonitorConfig` | `DeviationThreshold / MinSamples / WarmupDuration / EmaAlpha / ConsecutiveBreaches` |
| Path key 候選 | `PredictionRecord.NwdafSubId` 已保存 | 尚未作為 grouping key 使用 |
| Retrain 重入保護 | `store.IsRetraining` | 在 flight 時 skip；hot-swap 完會走新 monitor 的 WarmupDuration（見 §6.4） |

---

## 3. 整體架構

### 3.1 資料流

```
ML Service prediction
        │
        ▼
AddPrediction(PredictionRecord: NwdafSubId, TargetTime, PredUl, PredDl)
        │
        ▼  （每 checkInterval 秒）
ConsumeMaturePredictions → lookupGroundTruth
        │
        ▼
                    ┌── group by PathKey (= groupId) ──┐
                    │                                    │
                    ▼                                    ▼
      compute metrics for path A          compute metrics for path B
      {sMAPE, MAE, MSE, WAPE, NRMSE}       {sMAPE, MAE, MSE, WAPE, NRMSE}
                    │                                    │
                    ▼                                    ▼
          PathState.RecordMetric               PathState.RecordMetric
          (ring buffer + mean/std)             (ring buffer + mean/std)
                    │                                    │
                    ▼                                    ▼
        two-layer gate on primary             two-layer gate on primary
        (absolute AND z-score)                (absolute AND z-score)
                    │                                    │
                    └─────────── OR ─────────────────────┘
                                  │
                                  ▼
                 consecutive N rounds any-path triggered
                                  │
                                  ▼
                          startRetrainWorkflow
```

### 3.2 新增 / 改動的組件

| 組件 | 所在 package | 新/改 |
|------|--------------|------|
| `computeMAE / MSE / WAPE / NRMSE` | `internal/anlf` | 新 |
| `MetricResult`（或 `map[string]float64`） | `internal/anlf` | 新 |
| `PathKey` 解析（NwdafSubId → groupId） | `internal/context` | 新（借用 GroupResolver） |
| `PathState`（ring buffer + stats） | `internal/context` | 新 |
| `ModelAccuracyStore.paths` | `internal/context` | 改（加 map） |
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
- Primary metric **先用 `MAE`** 驅動 decision（理由：單位直覺、低流量不像 sMAPE 那樣爆）；其餘 metric 只當觀察用
- 日後若 CSV 分析顯示其他 metric 更適合，只需改 config `primaryMetric`

**注意事項**
- `MSE` 量綱是 bytes²，很容易跨 path 尺度爆炸 → CSV 照寫，但不建議當 primary
- `WAPE` 分母是 `sum(|actual|)`，若整段都 0 要能安全回傳 0 而非 NaN
- `NRMSE` 分母選用 range 或 mean 要固定，單元測試時也要鎖定
- 所有 metric 在 `actual=0 AND pred=0` 的 sample 都視為 0 貢獻，不計入 baseline（見 §4.5）

**任務**
- [ ] 實作 `computeMAE / computeMSE / computeWAPE / computeNRMSE`
- [ ] `computeAll` 統一入口
- [ ] Perfect / zero / large-error / mixed 的單元測試（每 metric 至少 4 case）

---

### 4.2 Path-level 分桶

**目的**：把同一 model 下不同 group 的樣本分開計算 metric，避免退化 path 被稀釋。

**為什麼這樣設計**
- Monitor 掛在 model 上（一個 model 一個 store），但同一 model 可能服務多個 group，實際退化通常是「某條 path」而不是整體
- 先分桶、再在 model-level 做 OR 邏輯（任一 path 退化就 retrain），比「全體平均」更貼近真實需求

**決策結論**
- **Path key = `groupId`**（起步版）
- 最終版：訂閱 filter 中的 **AoI（Area of Interest）**，屬於未來工作（§6.1）
- `NwdafSubId` / `corrId` 不採用

**實作要點**
- `PredictionRecord` 加 `GroupIds []string`（或在 consume 時再解析；看哪個 race 風險低）
- 解析路徑：`NwdafSubId → Subscription.TargetOfAnalytics → groupIds`（呼叫現有 `GroupResolver` / context API；若 subscription 直接帶 supi list 而非 group，先不支援，記 log skip）
- 一個訂閱可能對應多 group，每個 group 都算一條獨立 path
- `checkModelAccuracy`：
  1. consume mature predictions
  2. 對每個 pred lookup ground truth
  3. **依 groupId 分桶**（一個 pred 若屬多 group，分別入桶）
  4. 每桶分別呼叫 `computeAll`
- Model-level aggregate 保留作為 debug log，不再進 decision

**注意事項**
- Path key 抽象成 string，呼叫 `resolvePathKey(pred) []string`，未來換 AoI 只改這個 function
- 單 prediction 屬多 group 時會在多桶各算一份，會放大 sample 數；log 時標清楚
- 若 `resolvePathKey` 回傳空（group 解析失敗），該 pred 歸到特殊桶 `__unresolved__` 或直接跳過（建議跳過並 log）

**任務**
- [ ] `resolvePathKey(pred PredictionRecord) []string` 實作
- [ ] `checkModelAccuracy` 改寫為 per-path compute
- [ ] 單元測試：兩個 group 分別退化，各自計算正確、互不干擾

---

### 4.3 Per-path recent buffer & baseline 統計

**目的**：替每條 path 維護近期 metric 歷史，供 drift 判斷用。

**為什麼這樣設計**
- 策略核心：「看這次 error 相對於 recent baseline 是否明顯偏離」，所以必須保留歷史
- Mean / std 是最容易解釋、最容易畫圖、最容易 debug 的 summary statistics（優於直接套論文公式）

**實作要點**
- 新型別 `PathState`（`internal/context`）：
  ```go
  type PathState struct {
      pathKey      string
      buffers      map[string]*ringBuffer // metric name → ring buffer of float64
      validSamples map[string]int          // metric name → 有效樣本數（排除 both-zero）
      lastUpdate   time.Time
      breach       int                     // 連續觸發計數（§4.4）
      // mutex
  }
  ```
- Ring buffer 容量由 config `recentBufferSize` 控制（預設 20）
- 方法：
  - `RecordMetric(metric string, value float64, isValid bool)`
  - `Mean(metric) / Std(metric) / ZScore(metric, current)`
  - `SampleCount(metric) int`
- `ModelAccuracyStore` 加 `paths map[string]*PathState`（以 pathKey 為 key），並處理 concurrency（path map RW lock + 各 PathState 自己的 lock）

**生命週期（決策）**
- **建立**：某 path 首次 `RecordMetric` 時 lazy create
- **清除**：被動 GC — `lastUpdate > TTL`（預設 10 min）由 monitor loop 定期掃描清掉
- **不採**訂閱取消 callback 路線（避免侵入 subscription lifecycle）
- Hot-swap 時整個 `ModelAccuracyStore` 重建，不用特別處理 path 清理

**注意事項**
- 多 goroutine 可能同時 `AddPrediction`、`RecordMetric`、GC 清理 → concurrency 是重點
- Ring buffer 用 slice 實作，`append` + modulo 即可，不必引外部套件
- 統計值（mean/std）每次 on-demand 算，不快取（buffer 小、不值得）

**任務**
- [ ] `PathState` 型別 + ring buffer
- [ ] `ModelAccuracyStore.GetOrCreatePath(pathKey)`
- [ ] 定期 GC（在 `runModelAccuracyLoop` 中順便跑，不開新 goroutine）
- [ ] Concurrency 測試（`go test -race`）

---

### 4.4 兩層 retrain decision

**目的**：只有「error 夠高」且「相對 baseline 明顯偏離」同時成立時才視為觸發，再加連續 N 輪 dampen 偶發波動。

**為什麼這樣設計**
- 單層絕對門檻：對不同 path 流量差異無能為力
- 單層 z-score：低 error 下的 outlier 也會觸發，造成 baseline 低且穩定的 path 被誤判
- 兩層 AND + 連續 N 輪：能擋掉低量 false positive，也能擋掉偶發 spike

**判斷邏輯**
```
absGate     = current > max(fixedFloor, mean + dynamicFloorK * std)
zScore      = (current - mean) / max(std, minStd)
relGate     = zScore > zScoreThreshold
pathTriggered = absGate AND relGate

if pathTriggered:
    path.breach += 1
else:
    path.breach = 0

if any path.breach >= consecutiveBreaches:
    startRetrainWorkflow()
```

**決策結論**
- 兩層保留、AND 觸發
- Threshold 只作用在 z-score（scale-invariant），不做 per-metric 配置
- Primary metric：**MAE**（測試期間可能調整）
- `fixedFloor` 預設值先暫定（見 §4.7），測試階段用 CSV 觀察後再回填
- `zScoreThreshold` 先用預設 `3.0`，測試階段調整
- `consecutiveBreaches` 沿用現有欄位（預設 3）
- EMA 策略整個移除
- 沿用 `store.IsRetraining` in-flight guard，不變

**注意事項**
- Cold start：buffer 填充未達 `minBufferSamples` 時只走 `absGate`，跳過 `relGate`（見 §4.5）
- 多 path 之中只要任一觸發就 retrain，不做「多 path 同時觸發才算」的 AND
- 觸發之後立刻 `store.SetRetraining(true)` 並呼叫 `startRetrainWorkflow`；所有 path 的 breach 同時歸零
- Log 格式建議標註每層結果：`path=X metric=Y cur=... mean=... std=... z=... absGate=T zGate=F breach=2/3`

**任務**
- [ ] 移除 `TriggerStrategy / EmaAlpha / checkEMATrigger`
- [ ] 改寫 `HandleDeviationReport` 為 per-path two-layer gate
- [ ] Model-level OR + consecutive counter
- [ ] 單元測試：
  - 低 error + 高 z：不觸發（absGate 擋）
  - 高 error + 低 z：不觸發（zGate 擋）
  - 兩層皆過 + 連續 3 輪：觸發
  - 中間一輪低於 threshold：breach 歸零

---

### 4.5 Cold start 保護（策略文件未寫）

**目的**：避免 buffer 還沒填滿、std 還沒穩定時出現爆 z-score 的 false positive。

**為什麼要補**
- 策略文件假設 baseline 已建立；實際上：
  - 每個新 path 第一次收到樣本時 buffer 是空的
  - 前幾筆樣本進來時 std 極小，z-score 會炸
  - 低流量 path 可能很久才累積到有效樣本
- Model-level 已有 `WarmupDuration` 處理整個 monitor 啟動時的冷啟動，但 path-level 的 buffer 冷啟動要獨立處理

**保護機制**

| 機制 | 作用 |
|------|------|
| `minBufferSamples`（預設 8） | 有效樣本數未達門檻前，decision 只走 `absGate`，跳過 `relGate` |
| `minStd`（預設 0.01） | 算 z-score 時以 `max(std, minStd)` 防除 0 / 爆值 |
| both-zero 排除 | `actual=0 AND pred=0` 的 sample 不計入 baseline（`validSamples` 不增），但 metric 值仍寫 CSV |
| no-ground-truth 跳過 | `lookupGroundTruth` 返回 nil 時完全跳過（現行為），不入 buffer |

**注意事項**
- `minBufferSamples` 與 config `minSamples`（每輪最少 matched pairs 才評估）是不同層概念，不要混淆
- Post hot-swap 靠現有 `WarmupDuration` 處理 model 層級冷啟動（見 §6.4），path-level 仍靠上述機制
- Cold start 期間的 CSV 資料照常寫（方便觀察 warmup 行為）

**任務**
- [ ] `minBufferSamples` gate 加進 §4.4 decision 邏輯
- [ ] `Std` 計算內建 `max(std, minStd)`（或在 `ZScore` 內套）
- [ ] `both-zero` 樣本 `isValid=false` 傳入 `RecordMetric`
- [ ] 單元測試：buffer=0~7 時只判 absGate；buffer=8 起啟用 z-score

---

### 4.6 觀測資料落地（CSV）

**目的**：觀察期離線分析用 — 選 metric、調 threshold、畫 pred vs actual 時序圖。

**定位**
- **過渡期工具**：目的是幫助測試期間選 metric、調 threshold；功能穩定後可移除
- 不規劃升級到 Prometheus / Mongo

**為什麼用 CSV**
- 最小侵入（不動現有 logger / DB）
- 直接 pandas 讀取，轉 plot 容易
- 移除時只要砍 writer、拿掉兩個 config 欄位

**兩個檔案**

**(a) `metrics_<ts>.csv`** — 每輪 × 每 path × 每 metric 一列

欄位：
```
timestamp, model, path, metric, current, mean, std, min, max,
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
checkTime, model, path, nwdafSubId, predictedAt, targetTime,
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
- Path 名稱若含特殊字元（`,`、`"`），寫入前要逃脫（用 `encoding/csv` 即可）
- Shutdown 時確保 flush + close，避免尾端資料遺失（hook 到 `CancelContext` 的 Done）

**任務**
- [ ] CSV writer（共用 `encoding/csv`），封裝開檔 / append / close
- [ ] Metrics writer：在 `checkModelAccuracy` 計算完後寫一批
- [ ] Pairs writer：在 match 結束後寫一批
- [ ] Shutdown flush（掛 monitor loop 的 ctx.Done）

---

### 4.7 Config 擴充

**目標**：反映新的 decision 結構與觀測能力；同時移除 EMA 相關殘留。

**`AccuracyMonitorConfig` 變更清單**

**沿用**
- `enabled`、`checkInterval`、`minSamples`、`warmupDuration`、`consecutiveBreaches`

**移除**
- `triggerStrategy`（新版為唯一策略，不再切換）
- `emaAlpha`（EMA 整個拿掉）
- `deviationThreshold`（改用動態 floor + z-score）

**新增**

| 欄位 | 型別 | 說明 | 預設（暫定） |
|------|------|------|------|
| `metricsToRecord` | `[]string` | 候選 metric，CSV 都會寫 | `[sMAPE, MAE, MSE, WAPE, NRMSE]` |
| `primaryMetric` | `string` | 進 decision 的 metric | `MAE` |
| `recentBufferSize` | `int` | per-path ring buffer 長度 | 20 |
| `minBufferSamples` | `int` | 啟用 z-score 前的最少有效樣本數 | 8 |
| `minStd` | `float64` | std floor，避免 /0 | 0.01 |
| `fixedFloor` | `float64` | 第一層絕對下限（MAE 單位：bytes） | 先暫定（測試後調整） |
| `dynamicFloorK` | `float64` | 動態 floor `mean + k*std` 的 k | 2.0 |
| `zScoreThreshold` | `float64` | 第二層 z-score 門檻 | 3.0 |
| `pathStateTTL` | `int`（秒） | path state 被動 GC 時間 | 600 |
| `csvDumpDir` | `string` | 觀察 CSV 輸出目錄（過渡期） | `logs/accuracy/` |
| `csvDumpEnabled` | `bool` | 是否寫 CSV（過渡期功能，穩定後可移除） | `true` |

**注意事項**
- `fixedFloor` 依 primary metric 單位（MAE: bytes）先給一個暫定值不卡住實作；測試期間看 CSV 再調
- 舊 config 的 `deviationThreshold / emaAlpha / triggerStrategy` 要在 `config/nwdafcfg.yaml` 同步清掉

**任務**
- [ ] 修改 `pkg/factory/config.go: AccuracyMonitorConfig`
- [ ] 同步更新 `config/nwdafcfg.yaml`
- [ ] 預設值驗證：missing 欄位 fallback 合理

---

### 4.8 測試策略

**單元測試**

| 層級 | 內容 |
|------|------|
| Metric | 每個 metric 的 perfect / zero / large / mixed case |
| Path 分桶 | 兩個 group 分別退化，只有退化者被計入 |
| Baseline | Ring buffer 溢位後舊值被捨棄、mean/std 正確 |
| Cold start | Buffer 0~7 只走 absGate；8 起啟用 z；std=0 時不爆 |
| Two-layer gate | 四象限（abs T/F × z T/F）各一案、連續 N 輪正確 |
| Concurrency | `RecordMetric` / `HandleDeviationReport` / GC 並發不 race |
| GC | path 被動清理 `lastUpdate > TTL` |
| CSV | 欄位齊全、shutdown flush |

**整合驗證（在 Milestone 1 / 2 分段做）**
- Milestone 1：跑一段時間後離線分析 CSV（metric 相關係數、path 分布差異）
- Milestone 2：Staging 跑一段時間，對比 Milestone 1 的 log 確認 retrain 觸發時機合理

---

## 5. 實作順序

**原則**：不以 milestone 嚴格劃分「完整測試後才進下一階段」；每個 checkpoint 可獨立合入，threshold / primary metric 等「測試才能決定的值」先用暫定預設值（見 §4.7），跑起來之後再按 CSV 調整。

### Checkpoint A — Metric 候選 + Path 分桶 + CSV 觀察

**目標**：先把「能看到東西」的管線打通；舊 decision 仍在運作。

**實作**
1. Metric 候選集合（§4.1）— primary 先設 `MAE`
2. Path-level 分桶（§4.2）
3. CSV dump：`metrics_<ts>.csv` + `pairs_<ts>.csv`（§4.6）
4. 既有 `consecutive` + `ema` decision 保持原樣（此階段不碰 decision）

**驗收**
- 系統跑起來後 CSV 有資料
- log 能看到每個 path 的多 metric 值

---

### Checkpoint B — Per-path baseline + 兩層 gate + cold start

**目標**：切換到新 decision 邏輯；不等 A 的 CSV 分析完，先用暫定 threshold。

**實作**
1. Per-path state + recent buffer（§4.3）
2. 兩層 gate（§4.4）
3. Cold start 保護（§4.5）
4. Config 擴充、移除 EMA 與舊 threshold（§4.7）
5. Path state 被動 GC（§4.3）
6. 單元測試（§4.8）

**暫定值**（可在 A 的 CSV 累積後再回頭調）
- `primaryMetric = MAE`
- `zScoreThreshold = 3.0`
- `fixedFloor` 依實測 MAE 量級先估（例如觀察幾輪後取大致 median 當參考）
- `dynamicFloorK = 2.0`、`consecutiveBreaches = 3`、`recentBufferSize = 20`、`minBufferSamples = 8`

**驗收**
- 單元測試全過（含 `-race`）
- 實跑不誤觸（誤觸就調 CSV 再調參）

---

### Checkpoint C — 調參 / 清理

**目標**：根據 A + B 的運作資料微調 threshold 與 primary metric；穩定後清掉觀察性質的程式碼。

**內容**
- 按 CSV 調 `primaryMetric` / `fixedFloor` / `zScoreThreshold` / `dynamicFloorK`
- 確認穩定後移除 CSV dump（§4.6 定位為過渡期工具）
- 舊程式碼 / 註解清理 / 文件化

---

## 6. 未決 / 待補

### 6.1 AoI 作為 final path key
TS 23.288 訂閱 filter 解析、`GroupResolver` 改造；保留至未來版本。Milestone 2 完成後的下一個題目。

### 6.2 策略文件是否同步補 cold start 段
策略文件假設 baseline 已建立，沒寫 cold start；決定是否回補到該文件，或只在本 plan 保留。

### 6.3 CSV flush 細節
目前預設每輪 flush；若 path 數 × metric 數過多導致 I/O 壓力，再改成緩衝 N 筆或每 T 秒。

### 6.4 Post-swap 額外 grace
Model-level 已靠現有 `WarmupDuration` 處理（`swapModelAfterRetrain` → `onModelSwapped` → `StartAccuracyMonitorForModel` → `runModelAccuracyLoop` 起始的 warmup，`internal/anlf/monitor.go:81-92`、`internal/sbi/processor/processor.go:54,100`、`internal/mtlf/training.go:236`）。目前判斷不必新增 `postSwapGraceDuration`；若 Checkpoint B 觀察到 retrain 完成後立刻又被觸發的情形，再補。

### 6.5 `pairs.csv` 是否寫無 ground truth 的列
初版先寫（看 match rate），量太大再移除。

### 6.6 `fixedFloor` 的具體數字（MAE 單位）
先給一個暫定值不卡住實作；跑起來後看 CSV 調整。Checkpoint C 正式定案。

### 6.7 CSV 移除時機
定位為過渡期工具。當 Checkpoint C 確認 primary metric 與 threshold 穩定、線上運作無異常後即可移除（砍 writer + 兩個 config 欄位）。
