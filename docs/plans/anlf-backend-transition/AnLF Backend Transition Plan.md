# AnLF Backend Transition Plan

Date: 2026-07-07

Status: Preparation, Phase 1, Phase 2, and Phase 3.5 completed; Phase 3 and Phase 4 responsibility migration implemented, behavioral parity validation incomplete

Related source line:

- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 14 HTTP Edge Shape And Flow Alignment Plan.md`
- `Behavioral Parity Audit.md`
- `Behavioral Parity Remediation Plan.md`

Audit note:

- 2026-07-13 的遷移等價性稽核確認 Phase 3 與 Phase 4 已完成責任邊界切換，但
  PyAnLF 尚未完整保留搬移前 Go analytics shaping、accuracy measurement、report cadence、
  scope identity 與 model generation 語意。完整證據、風險與已確認決策以
  `Behavioral Parity Audit.md` 為來源；R0至R5實作順序、contract、tests與completion criteria
  由`Behavioral Parity Remediation Plan.md`承接。
- 2026-07-13 已完成R0 Analytics slice與R1，恢復source-aware two-step alignment、internal
  zero-padding、mean timestamp及Go-compatible rounding；accuracy與後續lifecycle remediation仍未完成。

---

## 1. Purpose

這份文件定義 `AnLF Backend Transition` 這條新的主題型工作線。

它的目的不是再做一次 free5GC-aligned HTTP shape 調整，而是把目前
`NWDAF/` 內由 Go 端承載的 `AnLF` 業務邏輯，逐步遷移到獨立的
`PyAnLF/` backend。

這條線的核心目標是：

1. 讓 `NWDAF/` 保留 5GC-facing 的 transport、subscription、
   context、consumer 與流程協調責任
2. 讓 `PyAnLF/` 逐步承接 `AnLF` domain runtime，包括模型生命週期、
   analytics runtime、以及後續 accuracy workflow
3. 讓 Go 端對下游實作的依賴收斂成穩定的 backend contract，而不是把
   Python 本身寫死成主要架構語意

這份文件是這條工作線的唯一主入口。

---

## 2. Scope

這份 plan 涵蓋：

1. `NWDAF/` 與 `PyAnLF/` 之間的邊界定義
2. `AnLF` 相關業務邏輯的遷移順序
3. 每個 phase 的責任切分原則
4. 與既有 `free5gc-alignment` 工作線的關係切分

這份 plan 不直接處理：

1. `internal/sbi`、`internal/anlf`、`internal/mtlf` 的既有 HTTP edge
   shape 對齊收尾
2. free5GC exemplar alignment 本身的完成度判定
3. `PyAnLF/` 內部每個 phase 的實作細節與逐檔案步驟

後者應在各 phase 被正式啟動時，再由對應 phase 文件承接。

---

## 3. Planning Basis

本文件基於以下本地來源整理：

1. `AGENTS.md`
2. `nwdaf-docs/docs/development_policy.md`
3. `NWDAF/` current tree on 2026-07-07
4. `PyAnLF/` current tree on 2026-07-07
5. `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 14 HTTP Edge Shape And Flow Alignment Plan.md`
6. local 3GPP specs under `nwdaf-docs/specs/`

這條工作線不是 free5GC alignment 任務本身，因此它不以 free5GC
reference NF 作為主要 shape driver。

但實作時仍應遵守本 workspace 的一般開發紀律，並以：

1. `NWDAF/` 現有流程與 state ownership
2. local 3GPP spec semantics
3. `PyAnLF/` 實際承接能力

作為主要設計依據。

---

## 4. Current State

### 4.1 Current Go-side AnLF Responsibilities

Phase 3.5 完成後，`NWDAF/` 已不再直接執行模型載入、替換、回退、卸載、analytics
generation 或 reporting lifecycle。

目前仍主要留在 Go 端的責任包括：

1. 5GC subscription、callback ingress 與 SMF/UPF data collection
2. subscription runtime request、observation binding 與 backend transport coordination
3. 將 backend analytics report 映射成標準 Nnwdaf notification 並送至 external consumer
4. prediction bookkeeping、ground truth matching 與 accuracy monitor loop
5. accuracy reports 對 `MTLF` degradation/retrain policy 的輸入材料
6. 為尚未遷移的 accuracy/retrain flow 暫留的 model-reference correlation

Go-side AnLF code 已拆成 HTTP edge、contract、client、processor、coordinator 與
transitional accuracy packages；頂層 notifier 也已移入 `internal/sbi/notifier`。
Go 端不再持有 backend model ID，也不再執行 load、unload、swap、prediction shaping 或
analytics scheduling。但 accuracy workflow 與 `SharedModelInfo` correlation 尚未遷移，
因此目前仍是可工作的過渡架構，而不是最終的薄 transport shell。

### 4.2 Current PyAnLF Responsibilities

`PyAnLF/` 目前是獨立的 Python service。

截至 Phase 3 完成後，它已經完成：

1. repo naming 收斂到 `PyAnLF`
2. Python package layout 收斂到 `src/py_anlf`
3. service metadata、imports、runtime entrypoint 命名同步更新
4. subscription-scoped runtime apply、release 與 observation binding API
5. model acquisition、shared usage、replacement、fallback 與 release ownership
6. shared observation store、window shaping、inference 與 analytics generation
7. periodic reporting lifecycle 與 Go callback delivery
8. backend-local default model selection
9. domain、API、concurrency 與 cross-repo contract test coverage

Preparation 命名整理已在 `PyAnLF/` commit `987e98e` 完成；Phase 2 lifecycle
遷移則完成於 commit `7a2ebc4`。

功能上，`PyAnLF` 現在已承接 subscription runtime、model lifecycle 與 analytics runtime，
但尚未承接：

1. prediction bookkeeping 與 ground truth matching
2. accuracy evaluation 與 accuracy information generation
3. retrain input preparation 所需的 model/subscription correlation

也就是說，它已不再只是 inference-oriented service，但仍未成為完整的
`AnLF backend`。

### 4.3 Relationship To Free5GC Alignment

這條線是從既有 `free5gc-alignment` future work 延伸出來的，但不再屬於同一條
主工作線。

原因是：

1. `free5gc-alignment` 的核心問題是 Go-side shape、lifecycle、以及 NF-style
   boundary consistency
2. 這條線的核心問題則是跨 repo 的 backend ownership transition
3. 後者不應再混入前者的完成條件判定

---

## 5. Target Architecture

目標架構不是讓 `NWDAF/` 消失，而是讓它收斂成較薄的 5GC-facing layer。

### 5.1 NWDAF Responsibilities

目標上，`NWDAF/` 應保留：

1. SBI / auxiliary HTTP ingress
2. standards-facing consumers
3. subscription 與 correlation state
4. NWDAF-local context ownership
5. cross-domain orchestration
6. 與 `PyAnLF` 的 backend contract invocation

### 5.2 PyAnLF Responsibilities

目標上，`PyAnLF/` 應逐步承接：

1. model lifecycle runtime
2. analytics runtime
3. prediction bookkeeping
4. accuracy evaluation logic
5. 後續更完整的 `AnLF` domain procedure ownership

### 5.3 Boundary Rule

最終希望形成的邊界是：

1. `NWDAF/` 負責 NWDAF-facing 與 5GC-facing coordination
2. `PyAnLF/` 負責 AnLF backend execution
3. Go 端不把下游實作語言寫死成主要抽象語意

---

## 6. Migration Principles

這條線遵守以下原則：

1. 不一次大搬家；先定邊界，再搬邏輯
2. 每個 phase 都應保留可工作的中間狀態
3. 不為了 backend 遷移而破壞 `NWDAF/` 既有 free5GC-aligned transport shape
4. `NWDAF/` 與 `PyAnLF/` 的責任線必須能清楚說明 ownership
5. phase 的完成標準應以責任轉移是否明確為主，而不是以檔案數量或 package
   漂亮程度為主

---

## 7. Preparation

`Preparation` 已完成，且不納入正式 phase 編號。

### 7.1 Objective

在不碰 `NWDAF/` Go-side 命名與邊界的前提下，先把 Python repo 的身份與命名收斂。

### 7.2 Completed Work

已完成事項：

1. `NWDAF-ML-Service` 更名為 `PyAnLF`
2. repo 轉為 workspace root 下的獨立主目標 repo
3. Python package 改為 `src/py_anlf`
4. `pyproject.toml`、README、imports、runtime entrypoint 一起收斂

### 7.3 Non-goals

Preparation 明確不處理：

1. `NWDAF/` 內的 `inferenceEngine` 命名重整
2. Go-side backend abstraction 定型
3. domain logic ownership 實質搬移

---

## 8. Migration Phases

### 8.1 Phase 1: Backend Boundary Alignment

目標：

1. 在 `NWDAF/` 內把目前的 `inference engine` seam 重整成較中性的
   `AnlfBackend` 邊界
2. 明確定義 `NWDAF/` 呼叫 `PyAnLF/` 的 contract
3. 先完成 naming、config、client seam、injection point 的收斂

這個 phase 主要是定型邊界，不是大量搬移邏輯。

### 8.2 Phase 2: Model Lifecycle Migration

Status: Completed

Implementation commits:

1. `NWDAF/`: `62e2f9f` (`refactor(anlf): delegate model lifecycle to backend`)
2. `PyAnLF/`: `7a2ebc4` (`feat(runtime): manage subscription model lifecycle`)

目標：

1. 把 model load / unload / swap 的主要 runtime 責任逐步移到 `PyAnLF/`
2. 讓 `MTLF` provision callback 後的 activation path 更偏向 backend-owned
3. 降低 Go 端對 model lifecycle 細節的直接承載

完成結果：

1. backend contract 收斂成 subscription-scoped apply、release 與 predict
2. `MTLF` provision callback 與 Daisy retrain completion 收斂到同一條 apply path
3. Go 端移除 model ID、load、unload 與 swap lifecycle ownership
4. `PyAnLF` 接手模型共享、替換失敗回退、預設模型選擇與 release
5. 保留 Go-side model reference 僅供尚未遷移的 accuracy/retrain correlation 使用

完整 contract、實作與驗證紀錄見
`Phase 2 Model Lifecycle Migration.md`。

### 8.3 Phase 3: Analytics Runtime Migration

Status: Responsibility migration implemented; behavioral parity validation incomplete

目標：

1. 逐步把 analytics request shaping 與 prediction runtime 搬到 `PyAnLF/`
2. 讓 Go 端更多扮演 transport / orchestration shell
3. 收斂 Go-side `AnLF` 對 prediction 細節的本地實作

已確認的方向：

1. 不再保留 production Go-to-PyAnLF `Predict` contract
2. Go 將正規化 observation 依共享 observation source 傳給 `PyAnLF`
3. `PyAnLF` 擁有 observation buffer、analytics generation 與 reporting trigger
4. `PyAnLF` 主動 callback analytics report 給 Go `anlfServer`
5. Go 保留 external 3GPP mapping 與 NF consumer notification delivery
6. 目前未支援的 `Nnwdaf_AnalyticsInfo_Request` 不在本 phase 預留 speculative API
7. Phase 3 只遷移 `UE_COMMUNICATION`，accuracy workflow 保留給 Phase 4
8. PyAnLF 成為 `UE_COMMUNICATION` 必要 runtime；initial backend unavailable 時，
   整份拒絕使用 `503 ProblemDetails`，部分 event failure 使用
   `failEventReports/OTHER`，不保留 Go fallback scheduler

完成結果：

以下項目只表示 ownership、transport 與 contract 已完成切換。2026-07-13 behavioral
parity audit 已確認 historical input shaping、timestamp alignment 與 zero-padding 尚未完整
移植，不能把此處的 repository-level 驗證解讀為 Phase 3 行為等價完成。

1. Go production flow 已移除低階 `Predict`、local analytics scheduler 與 prediction shaping
2. PyAnLF 已接手 shared observation store、alignment、window shaping、inference 與
   periodic reporting lifecycle
3. source reuse 已改為 collection-profile exact match，observation delivery 具 bounded queue、
   stable batch ID 與 finite retry
4. PyAnLF report 會經 Go `anlfServer` processor/dispatcher mapping 後送給 external consumer
5. runtime revision、binding activation、stale callback 與 report dedup 已納入 contract
6. Go analytics business config 已搬移到 PyAnLF，Phase 4 accuracy 所需 retention 明確保留
7. Go、Python、race、lint、build 與 local cross-repo live contract 已驗證

完整 contract、lifecycle、failure handling 與 verification plan 見
`Phase 3 Analytics Runtime Migration.md`。

### 8.4 Phase 3.5: Go Package Boundary Consolidation

Status: Completed

Implementation commit:

- `NWDAF/`: `a7d0693` (`refactor(anlf): consolidate Go package boundaries`)

目標：

1. 在不改變 Phase 3 observable behavior 的前提下，整理 Go-side AnLF package boundary
2. 讓 `internal/anlf` 根 package 收斂成 HTTP server 與 API adapter，route declaration
   沿用現有 SBI 慣例放在對應 `api_*.go`
3. 將 backend contract、cross-domain coordinator、observation delivery 與 transitional
   accuracy logic 移到責任明確的子 package
4. 保留 free5GC-style notifier abstraction，並把目前頂層 `internal/notifier`
   收斂到 `internal/sbi/notifier`
5. 移除 `pkg/service` 經由 SBI processor 間接控制 AnLF-owned worker 的 lifecycle passthrough

本 phase 是 behavior-preserving refactor，不得改動 HTTP path、JSON contract、status code、
subscription lifecycle、retry/dedup、config schema 或 Phase 3 failure semantics。

完成結果：

1. `internal/anlf` 根 package 只保留 server 與 API adapter
2. backend wire DTO、HTTP transport、callback processing、coordination 與 accuracy 分別移入
   `contract`、`client`、`processor`、`coordinator` 與 `accuracy`
3. analytics notifier 移入 `internal/sbi/notifier`
4. `pkg/service` 直接持有並控制 AnLF coordinator/observation worker lifecycle
5. broad `AnlfService` 與 SBI processor lifecycle passthrough 已移除
6. full Go tests、lint、build、targeted race 與真實 PyAnLF live contract 均通過

完整 package layout、dependency direction、搬移順序與 verification plan 見
`Phase 3.5 Go Package Boundary Consolidation.md`。

### 8.5 Phase 4: Accuracy Workflow Migration

Status: Responsibility migration implemented; behavioral parity validation incomplete

目標：

1. 把 prediction bookkeeping、ground truth matching、AnLF-side Analytics/ML Model
   Accuracy Information generation 的主要 runtime 搬到 `PyAnLF/`
2. 重新定義 `MTLF` 需要從 backend 拿到哪些 retrain input materials
3. 讓 Go 端只保留必要的 subscription / cross-component coordination
4. 保持 `MTLF` 對 model degradation、retrain/reprovision 的判斷責任，不把
   MTLF decision policy 誤搬進 AnLF Backend
5. 將 MTLF model provision event 轉交 AnLF Backend，由 backend 根據自身 runtime state
   決定受影響的 subscriptions 與 replacement/fallback 行為
6. 替換 accuracy/retrain 對 Go model-to-subscription registry 的依賴後，移除
   `SharedModelInfo`、`sharedModelRegistry` 與 coordinator 的 model fan-out

已確認的方向：

1. 每個 stable model identity 擁有一個 monitor，monitor 內依 Analytics ID、target、
   filter 等 monitoring context 維護多個 scopes
2. `modelUniqueId` 與 model URL、Daisy TID、loaded-instance UUID 分離
3. PyAnLF 從既有 observation stream 形成 ground truth，並擁有 prediction bookkeeping、
   matching、metrics 與 report trigger
4. PyAnLF 先將 accuracy information callback Go，再由 Go 交給目前的 MTLF
5. MTLF degradation/retrain decision policy 保留在 Go；未來 PyMTLF transition 另行規劃
6. 缺少 stable model identity 的模型可繼續 inference，但不啟動 accuracy monitoring
7. 同 identity retrain 沿用 model ID，成功後切換 generation並重置 active monitor/policy
   state；失敗時保留舊 generation
8. Daisy task ID只作 training correlation，不作 model identity
9. 完整 model provision event交給 PyAnLF，由 backend決定 affected runtimes與 fallback
10. 本 phase 不實作完整標準 `Nnwdaf_MLModelMonitor` service lifecycle，但 internal
    contract欄位應可對應標準語意
11. external MTLF subscription建立前，Go先將 notification/provision correlation同步給
    PyAnLF，確保 callback可由 backend自行 resolve runtimes

完整 identity、contract、config、migration sequence、failure semantics與 verification plan
見 `Phase 4 Accuracy Workflow Migration.md`。

完成結果：

以下項目只表示 accuracy ownership 與 Go/PyAnLF contract 已完成切換。2026-07-13
behavioral parity audit 已確認 matching、metric/cadence、scope identity、model
identity/generation 與 completion lifecycle 尚有不等價或缺口，不能把此處的測試通過
解讀為 Phase 4 行為等價完成。

1. PyAnLF已接手prediction、ground truth、per-model/per-scope accuracy monitor與report delivery
2. Go MTLF保留decision/retrain policy，並以model identity、generation與scope接收report
3. provision binding在external MTLF callback前同步，Daisy與external MTLF completion收斂成
   backend-owned model provision event
4. Go-side accuracy package、model accuracy store、shared model registry與coordinator fan-out
   已移除
5. Go/Python config ownership已拆分，startup model使用固定bootstrap identity
6. NWDAF commit為`64fdf0a`，PyAnLF commit為`60994c3`
7. Python tests、Go tests/lint/build、targeted race與真實cross-repo HTTP contract均通過
8. 完整5GC、external MTLF、ADRF與Daisy共同運行的環境級實驗尚未執行

---

## 9. Verification Strategy

這條工作線不應只規劃責任遷移，也必須同時規劃對應的驗證出口。

主文件層級至少先要求以下三類測試在後續 phase 規劃中被明確承接：

1. `PyAnLF/` 內部業務邏輯的 unit tests，用於驗證 backend 接手後的 domain
   behavior
2. `PyAnLF/` API-level tests，用於驗證 service endpoint、request/response
   contract 與 error handling
3. `NWDAF/` 與 `PyAnLF/` 之間的實際交互測試，用於驗證 Go-side caller 與
   Python backend 的 contract alignment

各 phase 的細部文件應再進一步定義：

1. 該 phase 最低必須補齊哪些測試
2. 哪些屬於 unit-level 驗證
3. 哪些屬於 API-level 或 integration-level 驗證
4. 在當前環境下哪些只能做到局部驗證，哪些需要保留為後續執行驗證

---

## 10. Confirmed Final Boundary

Phase 4 planning 已解決原本保留的 ownership questions：

1. subscription runtime、model usage、model-to-subscription mapping、prediction、ground truth
   與 accuracy monitor 由 PyAnLF 擁有
2. Go 保留標準 analytics subscription、MTLF subscription、notification correlation、
   retrain task 與 MTLF decision state
3. PyAnLF 與目前 MTLF 的互動先經過 Go；未來若引入 PyMTLF，仍由 Go 維持兩個 backend
   boundary 之間的 coordination
4. model provision callback 與 Daisy completion 由 Go 正規化後整體轉交 PyAnLF，不再由 Go
   決定 per-subscription model fan-out
5. callback 所需的標準 procedure correlation 會預先同步給 PyAnLF，不以 Go model registry
   代替 backend runtime resolution
6. 完整 `Nnwdaf_MLModelMonitor` service 與 persistent model catalog 保留為 future work

---

## 11. Relation To Other Plans

### 11.1 Relation To Free5GC Alignment

`free5gc-alignment` 既有文件中的 auxiliary future work，現在只保留為來源與引用。

這份 plan 應視為：

1. 接手 Python-side migration 相關 future work 的新主文件
2. 與 `free5gc-alignment` 平行存在的新主題工作線

### 11.2 Phase Documents

各 phase 的細部文件位於同資料夾：

1. `Phase 1 Backend Boundary Alignment.md`
2. `Phase 2 Model Lifecycle Migration.md`
3. `Phase 3 Analytics Runtime Migration.md`
4. `Phase 3.5 Go Package Boundary Consolidation.md`
5. `Phase 4 Accuracy Workflow Migration.md`
6. `Behavioral Parity Audit.md`
