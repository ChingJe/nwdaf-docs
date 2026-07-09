# AnLF Backend Transition Plan

Date: 2026-07-07

Status: Preparation completed; Phase 1 completed; Phase 2 not started

Related source line:

- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 14 HTTP Edge Shape And Flow Alignment Plan.md`

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

目前 `NWDAF/` 的 Go 端承載的 `AnLF` 邏輯，不只是一個單純的推論 client。

目前仍主要留在 Go 端的責任包括：

1. `MTLF` model provision callback 後的 activation orchestration
2. model load / unload / swap 生命週期管理
3. per-subscription `MlModelInfo` 與 per-model `SharedModelInfo`
4. UE Communication historical data 對齊、整理與 prediction request shaping
5. analytics result 轉換成 `models.UeCommunication`
6. prediction bookkeeping、ground truth matching、accuracy monitor loop
7. accuracy reports 對 `MTLF` retrain policy 的輸入材料

因此，目前的 Go 端實際上仍然是 `AnLF` 主體，而不是只扮演 transport shell。

### 4.2 Current PyAnLF Responsibilities

`PyAnLF/` 目前是獨立的 Python service。

截至 Preparation 完成後，它已經完成：

1. repo naming 收斂到 `PyAnLF`
2. Python package layout 收斂到 `src/py_anlf`
3. service metadata、imports、runtime entrypoint 命名同步更新

這部分已在 `PyAnLF/` commit `987e98e` 完成。

但功能上，`PyAnLF` 目前仍主要只承接：

1. model load
2. model unload
3. predict

也就是說，它目前仍比較接近 inference-oriented service，而不是完整
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

目標：

1. 把 model load / unload / swap 的主要 runtime 責任逐步移到 `PyAnLF/`
2. 讓 `MTLF` provision callback 後的 activation path 更偏向 backend-owned
3. 降低 Go 端對 model lifecycle 細節的直接承載

### 8.3 Phase 3: Analytics Runtime Migration

目標：

1. 逐步把 analytics request shaping 與 prediction runtime 搬到 `PyAnLF/`
2. 讓 Go 端更多扮演 transport / orchestration shell
3. 收斂 Go-side `AnLF` 對 prediction 細節的本地實作

### 8.4 Phase 4: Accuracy Workflow Migration

目標：

1. 把 prediction bookkeeping、ground truth matching、accuracy evaluation
   的主要 runtime 搬到 `PyAnLF/`
2. 重新定義 `MTLF` 需要從 backend 拿到哪些 retrain input materials
3. 讓 Go 端只保留必要的 subscription / cross-component coordination

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

## 10. Open Questions

以下問題目前仍屬後續 phase 需要逐步決策的範圍：

1. 哪些 state 必須繼續留在 `NWDAF/` context，而不能下沉到 `PyAnLF/`
2. model provisioning callback 完成後，誰是 activation state machine 的最終 owner
3. ground truth 資料與 accuracy monitor 資料的 owner 要如何切分
4. `MTLF` 與 `PyAnLF` 的互動是否要始終經過 `NWDAF/`，還是某些資料可由
   `PyAnLF` 直接承接

這些問題不應在未經 phase 選擇的情況下提前一次解完。

---

## 11. Relation To Other Plans

### 11.1 Relation To Free5GC Alignment

`free5gc-alignment` 既有文件中的 auxiliary future work，現在只保留為來源與引用。

這份 plan 應視為：

1. 接手 Python-side migration 相關 future work 的新主文件
2. 與 `free5gc-alignment` 平行存在的新主題工作線

### 11.2 Future Phase Documents

當某個 phase 被正式選定後，應在同資料夾下建立對應的細部文件，例如：

1. `Phase 1 Backend Boundary Alignment.md`
2. `Phase 2 Model Lifecycle Migration.md`
3. `Phase 3 Analytics Runtime Migration.md`
4. `Phase 4 Accuracy Workflow Migration.md`

在那之前，這份主文件維持高層規劃即可。
