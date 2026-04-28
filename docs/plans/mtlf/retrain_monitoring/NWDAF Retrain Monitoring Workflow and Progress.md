# NWDAF Retrain / Monitoring — Workflow And Progress

> 對應主文件：`NWDAF Retrain Monitoring Implementation Plan.md`
>
> 本文件是此階段的實作管理文件，用來記錄 branch 策略、merge 時機、開發注意事項與進度。
> 進度更新以本文件為主，不再回頭維護主計畫文件中的 `- [ ]` 勾選。
>
> 本階段實作需遵循 NWDAF 專案開發規範：`.agent/docs/development_policy.md`

---

## 1. 用途與更新原則

### 1.1 本文件用途

- 記錄此階段的 branch 策略與 merge 節點
- 維護 checkpoint / work item 進度
- 記錄實作中的注意事項、阻塞點與已定決策
- 作為對話中回報進度的主要落點

### 1.2 更新原則

- **主計畫文件**負責設計內容與實作範圍，不拿來追日常進度
- **本文件**負責進度、branch、merge 與執行中注意事項
- 每次開始一個新 work item、完成一個 work item、或遇到 blocker 時，優先更新本文件
- 若設計本身被推翻或擴充，再回頭改 `Implementation Plan`

### 1.3 狀態標記

使用下列狀態欄位：

- `todo`: 尚未開始
- `in_progress`: 進行中
- `blocked`: 有明確阻塞
- `review`: 已完成實作，等待檢查或合併
- `done`: 已合併完成

---

## 2. Branch 策略

### 2.1 原則

- 此階段以 **checkpoint 為唯一 branch 單位**
- 每個 branch 應能對應一個明確、可驗證、可合併的實作增量
- 同一 checkpoint 內的 work item 直接在同一 branch 內推進
- 不為更細的 feature 額外拆 branch
- branch 名稱應反映交付內容，不直接使用文件內部的 `checkpoint-a/b/c` 命名

### 2.2 建議命名

建議以階段目標命名，並在進度表中記錄它對應哪個 checkpoint：

```text
feat/retrain-monitoring-observability
feat/retrain-monitoring-degradation-policy
feat/retrain-monitoring-tuning-cleanup
```

修補性 branch：

```text
fix/retrain-monitoring-<topic>
docs/retrain-monitoring-<topic>
test/retrain-monitoring-<topic>
```

### 2.3 何時開新 branch

- 新 checkpoint 開始時

### 2.4 何時不要新開 branch

- 單純更新進度表或補一小段說明時
- 同一 checkpoint 內的功能細項推進時
- 同一 work item 的連續修正仍屬同一邏輯增量時

---

## 3. Merge 策略

### 3.1 Merge 回主線的時機

以下條件都滿足時才 merge：

1. 該 branch 對應的 work item 範圍已收斂
2. 需要的單元測試已補上
3. 至少完成 `make build` 與 `make lint`
4. 若是程式碼變更，原則上應再跑相關 `go test`
5. 文件、log、config 行為與 branch 目的相符

### 3.2 Checkpoint merge 原則

- **Checkpoint A**：
  - 直接在對應的 observability branch 上逐步完成
  - 但只有在「不改變既有 retrain decision」前提下才可合併
- **Checkpoint B**：
  - 直接在對應的 degradation-policy branch 上完成
  - 因為 state store / gate / cold start 彼此耦合較高，不再拆 feature branch
- **Checkpoint C**：
  - 直接在對應的 tuning-cleanup branch 上完成調參、清理、移除觀測性程式碼
  - 應在 A/B 穩定後再進行

### 3.3 Merge 順序建議

建議順序：

1. `feat/retrain-monitoring-observability`
2. `feat/retrain-monitoring-degradation-policy`
3. `feat/retrain-monitoring-tuning-cleanup`

---

## 4. 階段注意事項

### 4.1 Checkpoint A 注意事項

- 這一階段**不能改變既有 sMAPE retrain decision**
- 新增 `ScopeKey`、多 metric、`AccuracyReport`、CSV 時，要先確認舊流程仍然能跑
- 若實作方式會讓舊 decision 語意變動，該工作必須移到 Checkpoint B

### 4.2 Checkpoint B 注意事項

- MTLF 才是 degradation policy 與 retrain decision 的 owner
- baseline、breach counter、retrain guard 相關狀態應集中在 MTLF 端理解
- `ScopeKey`、`AccuracyReport` 契約若在 A 已落地，B 階段盡量不要再改 schema；若必改，需回補相容性說明

### 4.3 Checkpoint C 注意事項

- 以收斂與清理為主，不應再引入新的核心設計
- 若要移除 CSV，必須先確認調參已完成，且已有替代的觀測方式或明確結論

### 4.4 共通注意事項

- `PredictionRecord.ScopeKey` 必須是 snapshot，不能在 maturity 後重新反查 live subscription
- direct SUPI scope 必須視為一等公民，不能只優先照顧 group flow
- 主專案中的 code comment、log、API-facing string 與一般文件不應出現 `checkpoint A/B/C` 等內部階段性字眼；若需要描述當前狀態，請改用中性、對外可理解的行為描述
- 若出現設計更動，先在本文件記錄「決策變更」，再視需要同步主計畫

---

## 5. 進度表

> 更新方式：每次至少改 `Status`、`Owner/Branch`、`Last Update` 三欄；合併後補 `Commit`。
>
> Checkpoint A 已完成既定實作、驗證與主專案合併；A1-A5 統一改為 `done`，merge commit 為 `5da65cb`。
> Checkpoint B 已完成既定實作、驗證與主專案合併；B1-B7 統一改為 `done`，merge commit 為 `a1fed3d`。

| ID | Checkpoint | Work Item | Status | Owner/Branch | Commit | Last Update | Notes |
|----|------------|-----------|--------|--------------|--------|-------------|-------|
| A1 | A | Scope key materialization (`PredictionRecord.ScopeKey`) | done | master | 5da65cb | 2026-04-22 | canonicalize subscription `TargetUe`; unresolved scope keeps legacy prediction path |
| A2 | A | Metric candidates (`MAE/MSE/WAPE/NRMSE`) | done | master | 5da65cb | 2026-04-22 | multi-metric computation added for observability; legacy decision still uses model-level sMAPE |
| A3 | A | `AccuracyReport` contract | done | master | 5da65cb | 2026-04-22 | internal report callback added alongside legacy deviation callback |
| A4 | A | Accuracy observability output | done | master | 5da65cb | 2026-04-22 | Checkpoint A 先以 process-level CSV observability 交付；後續已於 2026-04-28 改為 log-only 並移除 CSV dump |
| A5 | A | Checkpoint A compatibility tests | done | master | 5da65cb | 2026-04-22 | legacy model-level sMAPE path verified against scope/report additions; `go test ./internal/...`, `make build`, `make lint` |
| B1 | B | MTLF per-scope state store | done | master | a1fed3d | 2026-04-23 | `MonitorStateStore` / `ScopeState` / ring buffer / TTL GC merged to master via Checkpoint B merge |
| B2 | B | Degradation path | done | master | a1fed3d | 2026-04-23 | processor cut over to `AccuracyReport`; degradation path merged as eligibility guard + decision signal |
| B3 | B | Cold start protection | done | master | a1fed3d | 2026-04-23 | `minBufferSamples` 作為 baseline 建立期 gate；baseline 未就緒前只累積 recent history、不做 retrain decision；both-zero rounds retained in recent buffer |
| B4 | B | Chronic poor-quality path | done | master | a1fed3d | 2026-04-23 | normalized chronic metric path merged with `mean | percentile` aggregation and `minTrafficScale` eligibility guard |
| B5 | B | Flexible `M-of-N` breach policy | done | master | a1fed3d | 2026-04-23 | strict consecutive replaced by configurable decision window; `M=N` remains the strict-behavior degenerate case |
| B6 | B | Config migration | done | master | a1fed3d | 2026-04-23 | `decisionWindowSize` / `requiredHitsInWindow` / `chronicPolicy.*` merged; YAML and defaults updated; legacy decision fields no longer drive retrain policy |
| B7 | B | Retrain lifecycle tests | done | master | a1fed3d | 2026-04-23 | merged after branch verification with `go test ./...`, `go test -race ./internal/mtlf/...`, `make build`, and `make lint` |
| C1 | C | Offline retrain analysis report tool | done | feat/retrain-analysis-report | 2e9c0dc | 2026-04-28 | HTML report tool implemented under `tools/retrain_analysis`; input source unified to log + config |
| C2 | C | Threshold tuning | todo | - | - | - | based on observed log-derived analysis report output |
| C3 | C | Cleanup and code removal | todo | - | - | - | remove temporary observation code when stable |
| C4 | C | Final documentation sync | todo | - | - | - | sync outcomes back to main docs if needed |

---

## 6. 決策變更紀錄

> 僅記錄會影響實作分工、資料模型、branch 拆分方式的變更。

| Date | Decision | Impact |
|------|----------|--------|
| 2026-04-21 | AnLF 只負責 accuracy monitoring / metric computation；MTLF 負責 degradation policy 與 retrain decision | state store 與 retrain policy flow（eligibility guard / decision signal / decision window）移到 MTLF |
| 2026-04-21 | Monitoring scope 以 subscription `TargetUe` 語意定義，不再以 group-only path 描述 | `ScopeKey` 必須兼容 group 與 SUPI |
| 2026-04-21 | 進度不再維護於主計畫文件勾選，改由本文件統一維護 | 後續進度更新只改本文件 |
| 2026-04-22 | Degradation path 採 `fixedFloor` eligibility guard + `z-score` decision signal；不再使用 `mean + k*std` 作為 eligibility 條件 | `fixedFloor` 只處理「error 本身太小」，`z-score` 只處理 relative anomaly，`std` 過小由 `minStd` 處理 |
| 2026-04-22 | Checkpoint B 的 both-zero round 視為真實觀測，保留在 recent buffer / baseline 中 | 不另外排除 both-zero；retrain 抑制交由 path eligibility、decision signal 與 decision window 處理 |
| 2026-04-22 | Checkpoint B 不新增 `BaselineEligible` 類型欄位 | `AccuracyReport` 維持較小契約，MTLF 直接以 scope report 寫入 recent buffer |
| 2026-04-22 | `baselineReady=false` 改為純 baseline 建立期，不再允許 eligibility-only trigger，也不累 breach | cold-start 期間仍保留真實樣本進 recent buffer，但 retrain decision 必須等 baseline ready 後才開始 |
| 2026-04-22 | `warmupDuration` 改為 startup-only monitor warmup，不再於 hot-swap 後重跑 | post-swap 直接依 fresh model state 重新監測；若未來需要 swap 後 grace，必須另立設計，不重用 startup warmup |
| 2026-04-22 | Checkpoint B 補入 `chronic poor-quality path` 設計，用來抓「模型一開始就差且持續差」的情況 | retrain policy 不再只有 degradation path；實作採 shared window config、per-path internal bookkeeping |
| 2026-04-22 | chronic path 第一版不做依流量量級切換 metric；改採固定 metric + `minTrafficScale` eligibility guard | 降低 regime 邊界複雜度；高低流量切換不透過切 metric 處理 |
| 2026-04-22 | chronic path 的 aggregator 核心只保留 `mean | percentile`；`median` 以 `percentile=50` 表示 | config 與實作維持一致，不另外引入 `median` 作為獨立核心模式 |
| 2026-04-22 | breach policy 改為通用 `M-of-N` decision window；`M=N` 可退化成現行 strict consecutive | 對 noisy runtime 更有容錯性；實作上 degradation/chronic 共用 window config、各自維護 hit bookkeeping |
| 2026-04-28 | cold-start 期間 `zscore` / `chronicValue` 持續計算與記錄，但 `degradationSignal` / `chronicSignal` 統一標為 `skipped` | 觀測值與決策值分離；baseline 建立期資訊保留完整，同時避免誤觸發 retrain |
| 2026-04-28 | accuracy CSV dump 移除，Checkpoint C 分析流程統一改為 log-only | 主專案移除 CSV writer / config；離線報表工具改由 log + config 重建 metric、policy、traffic 與 lifecycle timeline |

---

## 7. Blocker Log

> 有阻塞時新增條目；解除後更新狀態與處置結果。

| ID | Status | Date | Related Item | Blocker | Resolution / Next Action |
|----|--------|------|--------------|---------|--------------------------|
| BL-001 | open | - | - | - | - |

---

## 8. 使用方式

### 8.1 開始新工作前

1. 確認 work item 是否已存在於進度表
2. 若不存在，先新增一列再開始
3. 決定是否需要新 branch

### 8.2 工作進行中

- 將 `Status` 改為 `in_progress`
- 在 `Owner/Branch` 記錄目前 branch 名稱
- 若遇阻塞，新增到 `Blocker Log`

### 8.3 準備合併時

- 將 `Status` 改為 `review`
- 在 `Notes` 記錄已完成的驗證，例如 `make build`, `make lint`, `go test ./internal/mtlf/...`

### 8.4 合併完成後

- 將 `Status` 改為 `done`
- 清空暫時性的 blocker
- 若設計有正式變更，再回頭同步主計畫文件
