# MTLF Backend Transition Plan

Date: 2026-07-17

Status: High-level architecture and Phase 1 detailed plan confirmed; later phase plans pending

Related plans:

- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 1 PyMTLF Foundation And Backend Boundary.md`
- `nwdaf-docs/docs/plans/anlf-backend-transition/AnLF Backend Transition Plan.md`
- `nwdaf-docs/docs/plans/anlf-backend-transition/Phase 4 Accuracy Workflow Migration.md`
- `nwdaf-docs/docs/plans/anlf-backend-transition/Behavioral Parity Remediation Plan.md`
- `nwdaf-docs/docs/plans/daisy/general_improvement/nwdaf-daisy-improvement-plan.md`

---

## 1. Purpose

這份文件定義 `MTLF Backend Transition` 這條新的主題型工作線。

它的目的，是把目前由 `NWDAF/` Go runtime 承載的 MTLF domain logic 遷移到新的
`PyMTLF` backend，同時保留 Go 作為 NWDAF 與 5GC-facing layer 的責任。

這條線的核心目標是：

1. 將 accuracy decision policy、retrain decision、training job lifecycle、dataset
   preprocessing、local training 與 model artifact generation 移到 `PyMTLF`
2. 讓 Go 保留所有標準 service communication，包括 ADRF、Nnwdaf ML Model Provision、
   NRF-facing procedure 與 callback coordination
3. 讓 `PyMTLF` 只透過 NWDAF internal backend contract 要求資料與回報訓練結果，不直接
   成為 ADRF consumer、MongoDB client 或標準 NF
4. 將現有 ADRF retrieval 與歷史 MongoDB traffic storage 收斂成 Go-owned training data
   provider boundary
5. 以簡單、可驗證的 Python local trainer 取代目前 training framework dependency，並為
   後續 multiple-NWDAF federated learning 保留可替換的 trainer seam
6. 在工作線完成時，`NWDAF/` 與 `PyMTLF/` 不再保留 Daisy-specific code、config、API、
   callback、dependency 或命名
7. 讓 PyMTLF 成為 model generation 與 immutable artifact URL 的 owner；Go 只協調
   ModelReady、model apply 與 apply result，不代理 model bytes

這份文件是這條工作線的唯一主入口。各 phase 被正式啟動後，應在同資料夾建立細部文件，
並由本文件連結。

---

## 2. Scope

### 2.1 Included

這份 plan 涵蓋：

1. `NWDAF/`、`PyMTLF/` 與 `PyAnLF/` 的最終責任邊界
2. Go-to-MTLF-backend internal contract
3. accuracy report routing 與 MTLF policy ownership transition
4. retrain job、model identity、generation allocation、activation 與 restart reconciliation
5. ADRF 與 MongoDB training data provider architecture
6. Go-owned ADRF retrieval subscribe、notify、fetch 與 unsubscribe lifecycle
7. NWDAF-owned MongoDB local training data store 的正式化
8. dataset chunk delivery、completion metadata、cancellation 與 failure semantics
9. Python local training baseline 與可替換 trainer boundary
10. ModelReady 經 Go 交給 PyAnLF，以及 ModelApplyResult 經 Go 回到 PyMTLF 的完整流程
11. 現有 Go MTLF domain runtime 與 Daisy-specific integration 的移除順序
12. cross-repository verification 與 behavioral parity closure
13. 各 phase 為完成 AnLF/MTLF interaction 所需的 `PyAnLF/`、`NWDAF/internal/anlf/` 與
    shared Go wiring/contract 調整
14. MTLF-backend-owned private artifact repository、immutable download endpoint 與 retention

### 2.2 Explicitly Excluded

這份 plan 不包含：

1. 將 `PyMTLF` 或 MTLF 設計成獨立標準 NF
2. 讓 `PyMTLF` 註冊 NRF、實作標準 SBI server，或直接持有 5GC credentials
3. 讓 `PyMTLF` 直接呼叫 ADRF fetch address、使用 Fetch Correlation ID 或管理 ADRF
   retrieval subscription
4. 讓 `PyMTLF` 或 `PyAnLF` 直接查詢 NWDAF MongoDB
5. 讓 `PyAnLF` 與 `PyMTLF` 建立繞過 Go 的 production control/event path；經 Go provision
   的 private artifact URL download 是唯一明列的 data-path 例外
6. 在本階段實作 multiple-NWDAF federated learning framework
7. 繼續整合 centralized FL training framework
8. 把 internal backend API 宣稱為 3GPP standard API
9. 因 backend transition 重寫現有標準 Nnwdaf service semantics
10. 未核對正式 TS 29.575 local corpus 就臆造完整 ADRF OpenAPI contract
11. 在本 transition 內補齊 ADRF NRF discovery、OAuth2、307/308 redirect、完整
    ProblemDetails 或 feature negotiation
12. 在本 transition 內導入 `Nadrf_MLModelManagement` 作為 model artifact repository
13. 與該 phase 的 accuracy、model provision、runtime loading 或 cross-backend contract
    無關的 AnLF refactor
14. 在本 transition 內新增標準 `Nnwdaf_MLModelProvision` provider server
15. 讓 Go 儲存、代理或提供 MTLF backend 產生的 model artifact bytes

---

## 3. Planning Basis

本文件基於以下本地來源與已確認討論結果整理：

1. workspace root `AGENTS.md`
2. `nwdaf-docs/docs/development_policy.md`
3. `free5gc-dev-skill/SKILL.md`
4. `NWDAF/` current tree on 2026-07-17
5. `PyAnLF/` current tree on 2026-07-17
6. `AnLF Backend Transition Plan.md` 與 Phase 4 accuracy ownership 結果
7. `NWDAF/internal/mtlf/`
8. `NWDAF/internal/sbi/consumer/adrf_service.go`
9. `NWDAF/internal/sbi/consumer/mtlf_service.go`
10. `NWDAF/internal/sbi/processor/upf_notify.go`
11. `NWDAF/internal/context/db_models.go`
12. `NWDAF/internal/context/db_query.go`
13. `NWDAF/pkg/factory/config.go`
14. `NWDAF/config/nwdafcfg.yaml`
15. `nwdaf-docs/specs/TS 23.288/10 ADRF Services.md`
16. `nwdaf-docs/specs/TS 29.575/README.md`
17. `nwdaf-docs/specs/openapi/TS29575_Nadrf_DataManagement.yaml`
18. `nwdaf-docs/specs/openapi/TS29575_Nadrf_MLModelManagement.yaml`
19. local Release 18 TS 29.520 text and OpenAPI material
20. read-only free5GC reference tree under `resources/references/free5gc-main/`
21. `nwdaf-daisy-improvement-plan.md` 中已驗證的 URL-based model bundle/cache 經驗
22. `PyAnLF/src/py_anlf/core/model_manager.py` current artifact download/cache/load behavior

目前 canonical local spec corpus 已收錄完整 TS 29.575 與隨附的
`TS29575_Nadrf_DataManagement.yaml`、`TS29575_Nadrf_MLModelManagement.yaml`。
本計畫已依這些來源確認目前 ADRF subscribe、notify、fetch 與 record response 的 transition
baseline。TS 29.575 對 `FetchInstruction` 的 exact schema 仍外部引用未收錄的 TS 29.576；
此缺口及完整 OAuth、discovery、redirect、feature negotiation 與 error contract 不阻擋本次
backend transition，而是列入後續 standard-alignment 工作。

free5GC exemplar 在本計畫中的使用方式如下：

1. `UDR` 是 persistence ownership 的主要 exemplar
2. 其他 NF 的 MongoDB usage 用於確認各 NF 擁有自己的 collection/schema，而不是建立
   跨 NF shared database contract
3. free5GC SBI guidance 用於維持 Go handler、processor、consumer、context/store 與
   lifecycle boundary
4. free5GC reference tree 沒有可直接複製的 PyMTLF backend 或 NWDAF traffic training
   data exemplar，因此 backend 與 time-series design 是基於本專案需求的明確推論，不是
   upstream direct precedent

既有 Daisy improvement plan 只作為 model bundle、URL download、atomic load 與 cache 的
歷史實作證據。本計畫不繼承其中的 Daisy task、callback、endpoint 或 framework ownership。

---

## 4. Agreed Decisions

以下方向已確認，不應在實作時無聲改變：

1. 所有 accuracy policy 與 retrain decision logic 移到 `PyMTLF`
2. dataset preprocessing、training execution 與 model artifact generation 移到 `PyMTLF`
3. 所有標準 communication 保留在 Go，包括 ADRF 與 Nnwdaf ML Model Provision
4. `PyMTLF` 與 MTLF 不會成為獨立標準 NF
5. `PyMTLF` 不註冊 NRF，也不直接呼叫標準 NF service
6. `PyAnLF` 繼續擁有 prediction bookkeeping、ground truth、accuracy measurement 與
   accuracy information generation
7. PyAnLF accuracy report 先到 Go，再由 Go 交給 `PyMTLF`
8. `PyAnLF` 與 `PyMTLF` 不建立 production direct control/event path；model artifact GET 依
   decision 40 處理
9. `PyMTLF` 需要 training data 時，透過 internal API 向 Go 提出 dataset request
10. 若 Go 最終使用 ADRF，Go 必須自行完成 RetrievalSubscribe、RetrievalNotify、
    RetrievalRequest、RetrievalUnsubscribe 與相關 retry、request timeout、watchdog 及 cancellation
11. Go 不把 ADRF fetch address、Fetch Correlation IDs、subscription ID 或
    credentials 交給 `PyMTLF`
12. Go 從 ADRF 取回資料後，以 bounded chunks 傳給 `PyMTLF`，不先把整份 dataset
    無限制 materialize 後再一次傳送
13. MongoDB 是 NWDAF-owned local training data store，不是標準 ADRF，也不是 Python
    backend 的 shared database
14. 收到有效 UPF notification 時，允許持續 dual-write：raw standard-oriented record
    進 ADRF、normalized observation 進 MongoDB
15. training data provider 原則上 ADRF 優先、MongoDB fallback；fallback 必須由 config
    明確允許並留下原因，不得無聲切換
16. ADRF 與 MongoDB 必須輸出同一份 internal training observation contract
17. Go 只做 transport-level parsing、source resolution 與 normalization，不承接 feature
    engineering 或 model-specific preprocessing
18. 現階段使用簡單 Python local trainer
19. trainer 必須有窄且可替換的 interface，讓後續 multiple-NWDAF FL framework 可在不改
    data provider 與 policy contract 的前提下接手
20. 最終 `NWDAF/` 與 `PyMTLF/` 不保留 Daisy 字樣或相依性
21. `PyMTLF/` 建立為獨立 repository，service/package discipline 原則上參考 `PyAnLF/`，
    但兩者不共用 production runtime state，也不建立 direct control API
22. dataset delivery 採 Go 主動 push bounded chunks 到 PyMTLF，並以 acknowledgement 或
    equivalent flow control 實作 backpressure
23. 第一版 PyMTLF accuracy policy window 可採 process-local memory；但 generation allocation、
    pending provision 與 terminal job state 必須使用最小 durable journal 保存，初始建議採
    embedded SQLite。process restart 時不恢復半完成 training，必須先與 Go/PyAnLF reconcile
    active/pending generation 後才能接受新的 decision
24. model provision 採 URL-based artifact retrieval；不把 model binary 直接嵌入 provision
    notification 或 internal completion event
25. 過渡階段由 PyMTLF 維護 persistent local artifact repository 與 private immutable download
    endpoint；PyMTLF 將 URL、digest、identity 與 generation 交給 Go，Go 驗證並轉送 metadata，
    不保存或代理 model bytes
26. 長期 external model provider 的主體是「含 Go standard layer 與 PyMTLF backend 的另一個
    NWDAF」，不是 PyMTLF 自身
27. 移除現有 Daisy-oriented startup automatic training；bootstrap model 由 config/artifact
    提供，後續 training 只由 accuracy policy 或明確 manual operation 觸發
28. 保留現有標準 `Nnwdaf_MLModelProvision` consumer capability，不隨 legacy training
    framework cleanup 一起移除；本 transition 只保持未來 provider 可映射的 URL semantic，
    不新增 standard-facing provider server
29. 命名邊界比照既有 `AnlfBackend`/`PyAnLF` 原則：`PyMTLF` 只表示 Python repository、
    package 或 process；`NWDAF/` Go code、config、API seam、client、logs、tests 與 route
    naming 一律使用 `MTLF backend` 語意，不得出現 `PyMTLF`、`PyMtlf`、`pyMtlf` 或
    `pymtlf` implementation naming
30. ADRF transition baseline 沿用 current Go client flow：使用已設定的 ADRF endpoint、
    `consTrigNotif=true`、由 callback 取得 Fetch Correlation IDs，再由 Go 分批執行
    `RetrievalRequest`
31. Go 不使用 callback 的 `fetchUri` 作為動態 routing authority；標準條文指出該 URI 在
    此流程實際上不需要使用，因此 Go 繼續以自己的 ADRF endpoint configuration 為準
32. 本 transition 可沿用 current free5GC OpenAPI dependency 中既有的
    `models.FetchInstruction`；缺少 TS 29.576 exact attachment 不阻擋 owner migration
33. 本 transition 不以補齊 NRF discovery、OAuth2、307/308 redirect、完整 ProblemDetails、
    feature negotiation 或 ADRF ML model repository 為完成條件
34. 目前 ADRF path 應描述為支援 transition 所需的相容子集，不宣稱已達成完整
    TS 29.575 conformance；deferred standard alignment 不得被誤列為 PyMTLF owner migration
    的 blocker
35. MTLF transition 不是 MTLF-only code scope；當 phase 目標涉及 accuracy report、model
    provision、bundle compatibility、runtime activation 或其他 AnLF interaction 時，允許並
    應完整修改 `PyAnLF/`、`NWDAF/internal/anlf/` 及必要的 shared Go wiring/tests。實際修改
    範圍由各 phase plan 明列，不將此決策解讀為無限制的 cross-repository refactor 授權
36. model generation 由 PyMTLF 依 `(providerId, modelUniqueId)` 配置為單調遞增正整數；
    candidate 通過評估且 content-addressed artifact 完成 immutable publish 後才配置 target
    generation，generation 與 artifact digest 的 binding 保存於 durable journal/ModelReady，
    已配置的 generation 即使 apply 失敗也不得重用
37. ModelReady 必須帶 `baseGeneration`、`targetGeneration`、artifact URL 與 digest；PyAnLF
    不自行配置 generation，而是以 current generation 對 base generation 做 compare-and-swap，
    完整載入成功後才原子切換到指定 target generation
38. PyAnLF 經 Go 回傳 `APPLIED`、`FAILED`、`STALE`、`NO_MATCH` 或 `CONFLICT` apply result；
    只有 `APPLIED` 會讓 PyMTLF 將 target generation 標記為 active/confirmed，`NO_MATCH`
    只表示 artifact 已發布但目前沒有 matching runtime
39. rollback 不降低或重用 generation；若重新啟用舊 artifact content，PyMTLF 仍配置新的
    target generation，並把該 generation 綁定到舊 artifact digest
40. PyAnLF 與 MTLF backend 不建立 direct control/event API；但 PyAnLF 可使用經 Go 驗證與
    provision 的 immutable URL，直接向 MTLF backend private artifact endpoint 下載 model
41. ADRF-to-Mongo fallback 採 dataset-request-level all-or-nothing semantics；provider attempt
    中途失敗時，Go abort 該 attempt，PyMTLF 丟棄其 chunks，再以新的 attempt ID 從 MongoDB
    重新取得整份 dataset，不建立 mixed-provider training dataset

---

## 5. Current State

### 5.1 Repository State

截至 2026-07-17：

1. `NWDAF/`、`nwdaf-docs/` 與 `PyAnLF/` 是獨立 repositories；`NWDAF/` 與 `PyAnLF/` clean，
   `nwdaf-docs/` 目前有本計畫目錄的 untracked content
2. workspace 尚未存在 `PyMTLF/` repository
3. 現有 MTLF domain runtime 全部位於 `NWDAF/` Go code
4. `PyAnLF/` 已完成 AnLF backend transition 與後續 behavioral parity remediation，並已
   成為 accuracy measurement owner

### 5.2 Current Go-side MTLF Responsibilities

目前 `NWDAF/internal/mtlf/` 同時承載：

1. 接收 PyAnLF model accuracy report
2. report deduplication 與 model generation gate
3. per-model/per-scope accuracy decision state
4. degradation policy evaluation
5. retrain in-flight state
6. ADRF-assisted retrain job lifecycle
7. ADRF fetch instruction queue 與 record retrieval
8. historical data upload to current training framework
9. training task submission 與 completion callback
10. successful training result 到 PyAnLF model provision event 的轉接
11. startup-triggered training
12. shutdown-owned goroutine coordination

這表示 MTLF policy、training、standard data communication 與 framework-specific transport
目前仍混在同一個 Go package boundary。

### 5.3 Current Standard Communication

Go 已經實作：

1. `Nadrf_DataManagement_StorageRequest`
2. `Nadrf_DataManagement_RetrievalSubscribe`
3. `Nadrf_DataManagement_RetrievalNotify` callback handling
4. 依 Fetch Correlation IDs 執行 `RetrievalRequest`
5. `RetrievalUnsubscribe`
6. `Nnwdaf_MLModelProvision` generated client-based subscription operations

這些標準 responsibility 應保留在 Go，不因 backend transition 下沉到 Python。

### 5.4 Current ADRF Data Flow

目前有效 UPF notification 會逐 item 序列化成獨立 notification payload，再依 ADRF buffer
threshold 立即送出或將同一 correlation 的多個 payload 合併進一個 storage record。retrain 時，Go：

1. 依 retrain context 找出需要的 targets
2. 建立 ADRF retrieval subscriptions 與 time window
3. 接收 callback 中的 Fetch Correlation IDs
4. 依設定 batch size 取回 records
5. 將 records 上傳給目前的 training framework
6. 收斂 termination/watchdog 後清理 subscriptions

這條流程已具備 standard procedure skeleton，但 record delivery destination、job ownership、
failure handling 與 package boundary 都需要為 `PyMTLF` 重整。

### 5.5 Current MongoDB Data Flow

MongoDB traffic storage 是 ADRF 尚未導入前建立的舊路徑，後續沒有持續維護；但目前
production handler 仍保留寫入行為。

當 MongoDB config 存在且 startup ping 成功時：

1. NWDAF 建立 `nwdaf.upfTrafficData` time-series collection
2. 有效 UPF measurement 會被轉成 normalized `UpfTrafficRecord`
3. record 保存 correlation ID、SUPI、group ID、DNN、IP、measurement timestamp、volume、
   packet count 與 throughput
4. Mongo write failure 只記錄錯誤，不阻止後續 ADRF forwarding

現有 query code 支援 correlation ID、多 correlation IDs 與 `[from, to)` time range，但
目前沒有找到正式 production caller，且缺少完整的 retention、secondary indexes、snapshot、
deduplication、pagination 與 runtime health recovery。

因此現況是未正式定義的 dual-write，而不是已完成設計的 fallback architecture。

### 5.6 Current Training Dependency

目前 Go config、client、callback、tests、task payload 與 data upload 都直接使用 Daisy-specific
語意。團隊已另有人繼續研究 centralized FL usage；該工作不屬於本 NWDAF/PyMTLF transition。

這條工作線不保留 Daisy compatibility layer。最終 production runtime、config、tests 與
dependency 應完整移除相關語意。

### 5.7 Current Model Apply And Generation Flow

目前 Go 會把 training completion 正規化成 model provision event 並同步呼叫 PyAnLF；PyAnLF
依自己的 current generation 配置 `next_generation`，完成 atomic apply 後回傳 generation。
Go 現有 callback 只使用 error，沒有把完整 apply response 回送目前的 Go MTLF state。

target architecture 將改為由 PyMTLF 配置 base/target generation，PyAnLF 只負責驗證與原子
apply，並由 Go 將完整 ModelApplyResult 回送 PyMTLF。這是 owner migration 的明確行為變更，
不是對 current code 的現況描述。

---

## 6. Target Architecture

本工作線最主要的 target，是同一個 NWDAF 內由 PyMTLF 完成 retraining 後，將新模型提供給
PyAnLF 載入。完整主流程如下：

```text
1. Accuracy and retrain decision

   PyAnLF -- accuracy report --> NWDAF Go -- internal report --> PyMTLF

2. Training data acquisition

   PyMTLF -- dataset request --> NWDAF Go
                                      |-- ADRFProvider --> ADRF
                                      `-- MongoProvider --> MongoDB

   PyMTLF <-- normalized data chunks -- NWDAF Go

3. Training and model bundle production

   PyMTLF
      |-- preprocessing
      |-- local training
      |-- candidate evaluation
      |-- finalize and checksum bundle
      |-- publish content-addressed bundle -> MTLF Backend Artifact Repository
      |                                          |
      |                                          `-- private immutable URL
      `-- allocate target generation and bind artifact digest

4. Model provision to AnLF

   PyMTLF -- ModelReady(identity, base/target generation, URL, digest) --> NWDAF Go

   NWDAF Go
      |-- validates MTLF backend URL and metadata
      `-- sends ModelProvisionEvent(URL, digest, generations) -------> PyAnLF

   PyAnLF -- GET immutable URL --> MTLF Backend Artifact Endpoint

   PyAnLF -- verify -> cache -> load target generation -> atomic switch

   PyAnLF -- ModelApplyResult(status, active generation) --> NWDAF Go
   PyMTLF <-- ModelApplyResult ------------------------------ NWDAF Go
```

這條主流程中：

1. 模型內容與 bundle 由 PyMTLF 產生
2. PyMTLF 先由自己的 private artifact endpoint 發布 content-addressed immutable URL，再配置
   target generation 並持久化 generation-to-digest binding
3. Go 不訓練、不組合、不保存也不代理 model bundle；Go 驗證 URL metadata、協調 model
   provision，並把 apply result 回送 PyMTLF
4. PyAnLF 透過經 Go provision 的 URL 直接取得 PyMTLF training output，驗證與完整載入成功
   後才切換到指定 target generation
5. PyMTLF 只有在收到 `APPLIED` 後才把 target generation 標記為 active/confirmed
6. future external NWDAF 是這條本地流程的延伸，不是本 transition 的首要 use case

### 6.1 NWDAF Go Responsibilities

目標上，Go 保留：

1. standard SBI ingress、callback 與 consumer communication
2. ADRF StorageRequest 與完整 retrieval procedure ownership
3. MongoDB local store connection、schema、query 與 retention ownership
4. training data provider selection
5. dataset request validation、scope resolution 與 time-window enforcement
6. raw source record 到 common internal observation 的 transport-level normalization
7. bounded chunk delivery、backpressure、retry、cancellation 與 completion metadata
8. PyAnLF accuracy report 到 PyMTLF 的 routing
9. PyMTLF ModelReady 到 PyAnLF 與 PyAnLF ModelApplyResult 回到 PyMTLF 的雙向 routing
10. standard ML Model Provision subscription/correlation state
11. MTLF backend artifact URL scheme/host allowlist、metadata validation 與 correlation
12. app lifecycle、shutdown 與 cross-component coordination

Go 不再擁有：

1. degradation threshold policy
2. baseline/window/z-score/retrain gate
3. model-specific dataset preprocessing
4. training algorithm
5. training framework task payload
6. MTLF training job business state beyond transport correlation
7. model bundle content、preprocessing metadata 或 model weights 的產生
8. model generation allocation
9. model artifact storage、download endpoint 或 bytes proxy

### 6.2 PyMTLF Responsibilities

目標上，`PyMTLF` 擁有：

1. model identity/generation aware accuracy report consumption
2. report deduplication 與 stale generation handling
3. per-model/per-scope decision state
4. degradation 與 retrain policy
5. retrain concurrency、cooldown 與 job lifecycle
6. training dataset requirement definition
7. dataset completeness validation
8. preprocessing 與 feature construction
9. local training execution
10. model evaluation 與 acceptance decision
11. 完整 model bundle、weights、scaler、runtime config 與 integrity metadata 的產生
12. trainer interface 與未來 training implementation replacement seam
13. 將完成驗證的 bundle 原子發布到 local artifact repository
14. 維護 private immutable artifact download endpoint 與 retention
15. 依 model identity 配置 base/target generation，並保存最小 durable generation journal
16. 將 ModelReady URL、identity、base/target generation 與 digest 回報 Go
17. 消費 ModelApplyResult，reconcile pending/active generation 與 terminal job state

`PyMTLF` 不擁有：

1. NRF registration
2. standard Nnwdaf server/client
3. ADRF subscription、fetch 或 credentials
4. MongoDB connection
5. PyAnLF runtime mapping
6. external consumer notification
7. standard-facing model download endpoint；PyMTLF 的 artifact endpoint 是 private internal
   service，不是標準 SBI

### 6.3 PyAnLF Responsibilities

`PyAnLF` 繼續擁有：

1. model runtime 與 model usage mapping
2. inference 與 analytics generation
3. prediction bookkeeping
4. ground-truth matching
5. accuracy metric calculation
6. model/per-scope accuracy information generation
7. model replacement、runtime fallback 與 active generation transition，但不負責 generation allocation
8. model URL download、bundle validation、local cache 與 candidate atomic activation
9. 驗證 base generation、套用 PyMTLF 指定的 target generation，並回傳 apply result

PyAnLF 不擁有 retrain policy 或 generation allocation，也不直接與 PyMTLF 建立 control/event
通訊。

### 6.4 Boundary Rule

最終責任線為：

1. Go 擁有 standard procedure 與 data access capability
2. PyAnLF 擁有 analytics/accuracy execution
3. PyMTLF 擁有 policy/retrain/training execution
4. backend 間的 cross-domain coordination 一律經過 Go
5. Python backend 只接觸 internal domain contract，不接觸 standard fetch capability
6. PyAnLF 只依 Go provision 的 model URL 下載 bundle，不直接呼叫 PyMTLF control API；
   model bytes 可直接由 MTLF backend artifact endpoint 傳給 PyAnLF
7. PyMTLF 是 generation allocator、artifact producer 與 private URL publisher；Go 是
   validation/routing coordinator；PyAnLF 是 artifact consumer/runtime activation owner
8. 責任邊界不等於 code freeze；為實作上述 interaction，phase plan 可修改 PyAnLF 與 Go
   AnLF package 的 contract、routing、loading、state transition 及 tests

### 6.5 Future Extension: External Model Provider Shape

以下不是目前主要 implementation target，而是上述本地 model provision flow 未來可延伸的
方向。規格中的 provider 是「含 MTLF 的 NWDAF」，consumer 是另一個「含 AnLF 或 MTLF
的 NWDAF」。因此 future external use case 可形成：

```text
NWDAF A (model provider)
  Go standard Nnwdaf_MLModelProvision service
  PyMTLF internal backend
  MTLF-backend-owned immutable model artifact URL
                |
                | Nnwdaf_MLModelProvision_Notify
                |   mLFileAddr.mLModelUrl
                v
NWDAF B (model consumer)
  Go standard consumer
  PyAnLF or another PyMTLF backend
```

`PyMTLF` 可成為另一個 NWDAF 的 MTLF backend，但不會自己變成 external NF。當前同一
NWDAF 內的 transition 應先使用相同的 URL-based provision semantic，讓未來拆成兩個 NWDAF
時不必重設 model artifact contract。真正啟用 external provider 前，必須另行確認該 URL 對
consumer 可達，或由未來的 Go/gateway proxy、mirror 到 external-reachable storage；這些不在
本 transition 內實作。

### 6.6 Naming Boundary

architecture document 可以使用 `PyMTLF` 指稱實際 Python repository/process，正如 AnLF
文件可使用 `PyAnLF` 指稱 Python backend implementation；但這個 implementation name 不得
穿透 Go boundary。

Go-side naming 應使用：

1. interface/type：`MtlfBackendAPI`、`MtlfBackendClient` 或同等 `MtlfBackend` 語意
2. config：`mtlfBackend`
3. field/constructor parameter：`mtlfBackend`
4. logs/errors：`MTLF backend`
5. internal HTTP route/service metadata：`mtlf-backend` 或 domain-oriented operation name
6. tests/mocks：`MtlfBackend` naming

Go-side naming 不得使用：

1. `PyMTLF`
2. `PyMtlf`
3. `pyMtlf`
4. `pymtlf`
5. 任何把 Python runtime framework 寫入 domain contract 的名稱

wire payload fields 也應描述 accuracy report、dataset request、training job、artifact 與
model provision 等 domain semantics，不使用 producer language 或 repository name。這能讓
Go 的 backend abstraction 未來替換 implementation，而不必重命名 NWDAF code/config/API。

---

## 7. Internal Contract Families

本主文件先固定 contract semantics，不預先鎖死 HTTP path 或 wire format 細節。Phase 1
必須定義至少以下 contract families。

### 7.1 Accuracy Report Delivery

方向：

```text
PyAnLF -> Go -> PyMTLF
```

最低語意包括：

1. stable report ID
2. model provider ID 與 model unique ID
3. model generation
4. analytics event 與 canonical monitoring scope
5. sample/inference count
6. measured metrics
7. retrain context 所需的 observation source references
8. report timestamp

Go 只驗證 transport 與 correlation shape，不重新執行 policy。

### 7.2 Training Data Request

方向：

```text
PyMTLF -> Go
```

最低語意包括：

1. retrain job ID
2. dataset request ID
3. model identity/generation
4. analytics event
5. target/scope
6. observation source references
7. requested time window
8. required source policy or provider preference
9. schema version

PyMTLF 描述「需要什麼資料」，但不指定 ADRF fetch keys 或 MongoDB query implementation。

### 7.3 Dataset Chunk Delivery

方向：

```text
Go -> PyMTLF
```

每個 chunk 至少包含：

1. dataset request ID
2. provider attempt ID
3. chunk index
4. source provider
5. normalized observations
6. ordering/snapshot information
7. idempotency information

Go 應在 ADRF fetch 或 Mongo cursor 前進時逐批傳送，並以 bounded queue 或 acknowledgement
形成 backpressure。

### 7.4 Dataset Completion Or Failure

方向：

```text
Go -> PyMTLF
```

completion metadata 至少包含：

1. actual provider
2. completed provider attempt ID
3. fallback reason
4. effective time window
5. snapshot/cutoff
6. record/chunk count
7. schema version
8. completeness indication
9. duplicate/drop summary

failure 必須區分：

1. provider unavailable
2. empty result
3. partial result
4. invalid scope/request
5. deadline exceeded
6. cancellation/shutdown
7. delivery failure

### 7.5 Training Completion And ModelReady

方向：

```text
PyMTLF -> Go
```

training failure 與 accepted candidate 必須明確區分。只有 candidate 通過評估、artifact 完成
immutable publish 且 target generation 已配置後，PyMTLF 才送出 ModelReady。最低語意包括：

1. idempotent event ID
2. training job ID
3. model identity
4. base generation 與 target generation
5. model update indication
6. MTLF-backend-owned immutable artifact URL
7. artifact digest、size、bundle schema 與其他 integrity metadata
8. analytics event
9. candidate evaluation summary
10. publication time 與 optional validity metadata

training failure 可用同一 job family 的 terminal result 回報，但不得偽造成 ModelReady，也不得
配置或 provision 可供 activation 的 artifact generation。

### 7.6 Model Apply And Result

方向：

```text
PyMTLF -> Go -> PyAnLF -> Go -> PyMTLF
```

Go 驗證 ModelReady correlation、URL scheme/host allowlist、identity、generations 與 integrity
metadata 後，將 internal ModelProvisionEvent 送給 PyAnLF。PyAnLF：

1. 解析完整 matching runtime set；沒有 matching runtime 時回傳 `NO_MATCH`
2. 驗證所有 matching runtime 的 current generation 都等於 base generation；任一不符時回傳
   `STALE`，且不得切換任何 runtime
3. 若同一 event/target generation/digest 已成功套用，回傳 idempotent `APPLIED`
4. 若 target generation 相同但 digest 不同，回傳 `CONFLICT`
5. 下載、驗證、cache 並完整載入 target generation candidate
6. 在整個 matching runtime set 與其 generation 未改變的前提下，以 all-or-nothing 方式原子切換
   到 target generation；任一 runtime activation 失敗時全部保留舊 generation
7. 回傳 status、active generation、affected runtime count 與 failure detail

Go 必須把完整 ModelApplyResult 回送 PyMTLF，不得只保留 error。PyMTLF 只有在 `APPLIED`
時將 target generation 標記為 active/confirmed；`FAILED`、`STALE`、`NO_MATCH` 與 `CONFLICT`
都保留既有 active generation，並依 policy 決定 reconcile、retry 或終止 job。

### 7.7 Standard Mapping

本地 Release 18 contract 的標準對應為 `MLEventNotif.mLFileAddr.mLModelUrl`，或在適用時
提供 `mLModelAdrf`。`mlFile` 雖出現在 reused data type 中，但 TS 29.520 明確標示不適用於
Nnwdaf_MLModelProvision API，因此 transition 不應把 model binary 直接塞進 notification。

過渡階段的 expected flow 為：

1. PyMTLF 完成並原子化發布 model bundle 到自己的 persistent artifact repository
2. PyMTLF 維護 private immutable download URL，並經 ModelReady 交給 Go
3. Go 驗證 URL metadata，再以 internal model provision event 將 URL 交給 PyAnLF
4. PyAnLF 直接向 MTLF backend endpoint 下載，驗證並完整載入後才切換 generation
5. PyAnLF apply result 經 Go 回到 PyMTLF
6. 未來另一個 NWDAF 訂閱時，Go 可把同一種 URL semantic 映射進標準
   `Nnwdaf_MLModelProvision_Notify`

本 transition 不新增 standard-facing `Nnwdaf_MLModelProvision` provider server；上述第 6 項
只固定 future-compatible mapping semantic。真正 external provision 還需處理 URL reachability、
trust 與可能的 proxy/mirror。

---

## 8. Training Data Architecture

### 8.1 Provider Boundary

Go 應建立單一 training data service，底層至少提供：

1. `ADRFProvider`
2. `MongoProvider`

provider interface 應以 dataset request、chunk sink 與 completion result 為中心，不應把
ADRF-specific `FetchInstruction` 或 Mongo-specific cursor/filter 暴露到上層。

### 8.2 ADRFProvider

ADRF path 由 Go 完整擁有：

1. 建立 RetrievalSubscribe
2. 保存 subscription/job correlation
3. 接收 RetrievalNotify
4. 接收並驗證 Fetch Correlation IDs；`fetchUri` 可解析或記錄，但不作為動態 endpoint
   selection authority
5. 使用 Go 已設定的 ADRF endpoint 與 current batching policy 執行 RetrievalRequest
6. 沿用 current request timeout 與 watchdog 作為 operation lifecycle control，處理 timeout、
   late callback 與 cancellation；本階段不新增 exact `FetchInstruction.expiry` enforcement
7. 將 records 轉成 common observation chunks
8. 處理 termination request 與 unsubscribe
9. 將 provider completion/failure 回報 PyMTLF

`PyMTLF` 永遠不接收原始 fetch capability。

transition contract 的精確語意為：`fetch-correlation-ids` 可在單次 GET 中攜帶 `1..N` 個
identifier，成功回應為一個 `NadrfDataStoreRecord` envelope。TS 29.575 沒有為此 GET 定義
cursor/paging；Go 的 chunk size、queue 與 backpressure 都是 internal delivery policy，
不是 ADRF paging contract。Retrieval subscription/query 使用 `dataSetId` 字串；
`DataSetTag` 則是 stored record metadata，不應混為同一個 provider request type。

本 transition 不要求 ADRFProvider 同時新增 NRF discovery、OAuth2 access token、307/308
redirect、完整 ProblemDetails、feature negotiation 或 exact TS 29.576 model regeneration。
這些能力保留在後續 standard-alignment scope，且不得穿透到 MTLF backend API。

### 8.3 MongoProvider

MongoDB path 是 NWDAF private implementation：

1. 依 model/scope 解析 correlation IDs、SUPIs、group IDs 與 DNN
2. 使用明確的 `[from, to)` time window
3. 使用 ingest snapshot/cutoff 保證同一 dataset request 可重現
4. 依穩定排序與 pagination/chunk cursor 傳送 observations
5. 區分 empty、unavailable 與 partial result

MongoDB 不取代 ADRF standard API，也不對其他 NF 或 Python backend 暴露 collection。

### 8.4 Source Selection And Fallback

預期 config shape 應表達以下概念，但 exact key 由 phase plan 決定：

```yaml
trainingData:
  localStore:
    enabled: true
    type: mongodb
  provider:
    preferred: adrf
    fallback: mongodb
```

要求：

1. fallback 必須顯式啟用
2. 每個 dataset 必須記錄 actual provider
3. ADRF failure 不得被偽裝成正常 Mongo result
4. Mongo fallback dataset 必須具備與 ADRF path 相同的 internal schema
5. 若兩個 provider 都不可用，retrain job 應明確失敗，舊 generation 繼續服務
6. 一個成功完成的 dataset attempt 只能使用一個 provider，不允許 ADRF/Mongo mixed dataset

第一版 fallback 以整個 dataset request 為單位：

1. 每次 provider execution 使用不同的 attempt ID
2. ADRF 在任何 chunks 送出後發生 partial failure，Go 必須送出 attempt-aborted terminal event
3. PyMTLF 收到 abort 後必須丟棄該 attempt 已收到的所有 chunks，不得拿去 training
4. Go 以新 attempt ID 從 MongoDB 重新查詢完整 target 與 time window
5. 只有收到 successful completion 的 attempt 能被 PyMTLF commit 成 training dataset

預設 fallback classification：

1. ADRF unavailable、transport failure、request timeout、watchdog convergence failure 或
   partial retrieval failure：若 config 允許則 fallback
2. valid empty result：預設視為 empty dataset，不自動 fallback；如需 fallback 必須另有明確 config
3. invalid request、invalid scope、authorization failure、cancellation 或 shutdown：不 fallback
4. Mongo fallback 再失敗：整個 dataset request 失敗，不回頭混用已 abort 的 ADRF chunks

### 8.5 Dual-write Policy

若 MongoDB 被設定為 fallback source，NWDAF 必須在正常運行時持續保存 local copy；不能等
ADRF 發生故障後才開始寫入，否則 fallback 沒有歷史訓練資料。

dual-write 的語意為：

1. ADRF 保存 standard-oriented/raw record
2. MongoDB 保存 normalized local observation
3. 任一路徑寫入失敗不應自動阻斷另一條路徑
4. write failures 必須可觀測並能反映 local dataset completeness
5. exact-once 不作為無證據的預設保證；需要明確 deduplication key 與 policy

### 8.6 Common Training Observation

兩個 provider 最終應輸出相同的 language-neutral observation contract，至少涵蓋：

1. observed time
2. ingested time
3. source/correlation identity
4. SUPI/group/DNN/S-NSSAI/IP 等適用 scope metadata
5. volume、packet count、throughput 與明確 units
6. source provider 與 provenance
7. schema version

Go 可做 standard payload parsing、unit normalization 與 common schema mapping；feature ordering、
window shaping、scaling、imputation 與 model-specific transforms 屬於 PyMTLF。

### 8.7 MongoDB Formalization Requirements

現有 Mongo code 被正式採用前至少需要：

1. 從 stale context helper 收斂成明確 repository/store boundary
2. 定義 collection ownership 與 schema migration strategy
3. 增加 `ingestedAt` 或 equivalent snapshot boundary
4. 設定 retention/TTL，且 retention 大於最大 retrain window
5. 建立 correlation ID/SUPI 與 timestamp 相關 indexes
6. 定義 duplicate/idempotency key
7. 支援 cursor/pagination/chunking
8. 修正 global limit 可能偏向高流量 source 的問題
9. 建立 startup、runtime health、reconnect 與 unavailable state semantics
10. 增加實際 Mongo integration tests

---

## 9. Training Architecture

### 9.1 Local Training Baseline

本階段使用簡單 Python local trainer，以完成 MTLF ownership transition 與可驗證的 end-to-end
workflow 為優先。

local trainer 應：

1. 由 PyMTLF process 執行
2. 接收已完成驗證的 dataset abstraction
3. 將 preprocessing 與 training config 明確版本化
4. 產生可由 PyAnLF 載入的 artifact
5. 回報 metrics、artifact metadata 與 failure reason
6. 支援 deterministic seed 與 reproducible test fixture
7. 不暴露 training implementation 細節到 Go

### 9.2 Trainer Boundary

PyMTLF 內應定義窄的 trainer interface，例如語意上包含：

1. validate dataset
2. train
3. evaluate candidate
4. publish or expose artifact
5. cancel

本計畫不預先規定 class/module naming。未來 multiple-NWDAF FL framework 應替換 trainer
implementation，而不改 accuracy policy、dataset request、Go data providers 或 model provision
contract。

### 9.3 Model Bundle And MTLF-Backend-Owned URL Provision

既有 improvement plan 與 current PyAnLF 已驗證一條可延續的 bundle path：producer 將模型
套組打包成 `tar.gz`，PyAnLF 依 URL 下載、解壓、cache，全部載入成功後才原子切換。

transition 的初始 bundle v1 應以 current PyAnLF compatibility 為基礎，至少包含：

| File | Responsibility |
|------|----------------|
| `config.json` | bundle schema、model constructor parameters、inference feature order、window、output fields 與 component filenames |
| `model.py` | current trusted-local Python model definition，提供固定 `Model` entry point |
| `model.npy` | trained model weights |
| `scaler.pkl` | fitted preprocessing scaler |

原 improvement plan 曾提議 `config.yaml`，但已完成的實作與 current PyAnLF 實際使用
`config.json`。PyMTLF 第一版應產生 current consumer 能載入的實際格式，不回到未落地的
舊提案。

bundle contract 還應新增或明確化：

1. bundle schema version
2. model identity；generation 不寫死在 artifact content，改由 durable journal 與 ModelReady
   將 target generation 綁定到 artifact digest
3. analytics event/use case
4. feature order、units、sequence length 與 output fields
5. preprocessing implementation/version
6. framework/runtime compatibility
7. per-file digest 與 archive SHA-256
8. creation time、training job ID 與 producer metadata
9. immutable artifact key

publish/load requirements：

1. PyMTLF 只在 bundle 完整產生、驗證與 checksum 完成後才發布
2. URL 必須對應 immutable artifact key/content digest，不使用內容可被覆寫的 `latest` URL；
   同一 artifact content 因 rollback 綁定到新 generation 時不需重打包或改寫 URL
3. PyMTLF private artifact endpoint 不得暴露 local filesystem path，也不得接受任意 path traversal
4. artifact storage 必須在 PyMTLF process restart 後保持有效，不得只存在 temporary directory
5. PyMTLF 回報 URL、digest、size、identity、base generation 與 target generation
6. Go 驗證 URL scheme/host allowlist 與 metadata，但不下載、保存或代理 model bytes
7. PyAnLF cache key 必須包含完整 URL、model identity、target generation 與 digest
8. PyAnLF download 必須限制 size、timeout、redirect、accepted scheme 與 allowed host
9. archive extraction 必須防止 path traversal、link abuse 與 unbounded expansion
10. bundle validation/load 失敗時不得替換 active model
11. PyMTLF artifact cleanup 不得刪除 active generation 或仍在 rollback retention 內的 artifact
12. cleanup eligibility 必須由經 Go 回傳的 ModelApplyResult 與 retention policy 決定，不能只
    依 training completion 時間推測

MTLF backend artifact endpoint 是 internal private service，不是 `Nnwdaf_MLModelProvision`
標準 API，也不使 PyMTLF 成為獨立 NF。PyAnLF 對該 endpoint 的 GET 是經 Go provision 的
artifact data path；所有 accuracy、training、apply 與 lifecycle control event 仍必須經過 Go。

current bundle 的 `model.py` dynamic import、`joblib`/pickle scaler 與 NumPy pickle loading
具有執行不受信任內容的風險。第一版只允許在同一 trusted deployment boundary 內使用；在
external NWDAF model provision 啟用前，必須另行決定：

1. bundle signing 與 producer trust validation；以及
2. 是否改成 ONNX/TorchScript 等不攜帶任意 Python source 的模型格式，並以 JSON/NPZ 等
   safer representation 取代 pickle scaler。

TS 23.288 將 ML Model Interoperability Information 定義為 vendor-specific/out-of-scope，
因此 bundle v1 是本專案 internal interoperability contract，不應宣稱為 3GPP standard
model file format。

### 9.4 No Daisy Compatibility Layer

本 transition 不建立 neutral API 外包一層再暗中保留 Daisy。完成時必須移除：

1. Daisy client
2. upload-data operation
3. publish-task operation
4. Daisy callback server/route/payload
5. Daisy task config
6. Daisy-specific task ID/event source
7. Daisy dependency/import
8. Daisy-specific tests、logs 與 comments

若團隊未來需要 centralized FL，應由獨立 framework integration work line 實作，不回填本
工作線的 runtime。

---

## 10. Lifecycle And Failure Principles

### 10.1 Job Identity

以下 identity 必須分開：

1. model identity
2. model generation
3. accuracy report ID
4. retrain job ID
5. dataset request ID
6. provider attempt ID
7. provider subscription ID
8. Fetch Correlation ID
9. ModelReady/model provision event ID
10. backend process instance/epoch ID

任何 provider key、artifact URL 或 process-local UUID 都不得被當成 stable model identity。

### 10.2 Generation Allocation And Activation

PyMTLF 是 model generation allocator。每個 model identity 至少保存：

1. last allocated generation
2. active/confirmed generation
3. pending target generation
4. retrain/training job ID 與 terminal job state
5. ModelReady event ID
6. artifact URL 與 digest
7. apply status 與 terminal reason

第一版使用 embedded SQLite 或同等 transactional local store 保存這份最小 durable journal。
accuracy policy window 與半完成 training working state 不要求恢復；generation allocation 與已發布
artifact/provision state 則不得因 restart 遺失。

allocation 與 activation 規則：

1. 無既有 model 時以 base generation `0`、target generation `1` 開始
2. candidate 未通過評估或 content-addressed artifact 未完成 immutable publish 時不配置
   generation；artifact publish 完成後，target generation allocation 與 digest binding 必須在
   同一 durable transaction 中完成
3. 同一 ModelReady retry 使用同一 event ID、target generation、URL 與 digest
4. 已配置 generation 不重用；apply failure 後下一次 candidate 使用更大的 generation
5. PyAnLF 不自行執行 `current + 1`，只套用 PyMTLF 指定的 target generation
6. rollback 以新 generation 指向舊 artifact content，不降低 active generation
7. PyMTLF 只有收到 `APPLIED` 才更新 active/confirmed generation

PyMTLF startup 必須先進入 reconciliation：讀取 durable journal，經 Go 取得 PyAnLF 目前 active
identity/generation/digest，解析 pending event 為 applied、failed 或 unknown。在 reconciliation 完成前，
PyMTLF 不得接受新的 retrain/provision decision。無法確認的半完成 training job 視為失敗；
已發布 pending artifact 則依 event ID 與 PyAnLF active state 處理，不得重用 generation。

### 10.3 Idempotency And Stale Events

1. duplicate accuracy reports 不得重複觸發同一 decision
2. stale generation reports 不得影響 active generation
3. duplicate dataset chunks 必須可辨識
4. duplicate completion/provision events 不得重複切換 model
5. late ADRF callbacks 在 job completion/cancellation 後不得重新開啟 job
6. aborted provider attempt 的 chunks 即使晚到也不得進入後續 successful attempt
7. 相同 target generation 若對應不同 artifact digest 必須視為 conflict

### 10.4 Failure Behavior

1. data retrieval failure 不得切換 model
2. training failure 不得切換 model
3. candidate evaluation failure 不得切換 model
4. PyAnLF apply failure 必須保留舊 generation，並經 Go 將完整 failure result 回報 PyMTLF
5. fallback 使用必須留下 provider 與 reason
6. partial dataset 是否允許 training 必須由 PyMTLF policy 明確決定，不能由 Go 猜測
7. process shutdown 必須取消 owned retrieval、delivery 與 training work
8. 第一版 PyMTLF restart 後不恢復 process-local policy window 或半完成 retrain job
9. restart 前未完成的 training job 必須被視為失敗；舊 model generation 繼續服務
10. published/pending generation 依 durable journal 與 PyAnLF active state reconcile，不得只因
    process restart 自動配置新的 generation
11. duplicate/late completion 即使在 restart 後到達，也不得誤切換 active model

### 10.5 Intermediate Migration State

每個 phase 必須維持可工作的中間狀態，但不應長期保留兩套 active policy 或 training
runtime。

若需要 temporary feature flag：

1. 必須明確指定 owner 與 removal phase
2. 不允許同一 report 同時由 Go 與 PyMTLF 做 decision
3. 不允許同一 retrain job 同時送往兩個 trainer
4. completion criteria 必須包含 transitional path removal

---

## 11. Migration Principles

這條工作線遵守：

1. 先定 backend contract 與 identity，再搬 policy
2. 先建立可單獨驗證的 data pipeline，再切換 production retrain owner
3. standard communication 不因 Python 化而離開 Go consumer/processor boundary
4. 不把 Python implementation language 寫死成 Go-side architecture semantic
5. 不直接複製現有 framework-specific task model
6. 不因 MongoDB 已有舊程式碼就假設其 production readiness
7. 不因 ADRF path 已能運作就跳過本地 TS 29.575/OpenAPI contract audit
8. 每個 phase 都要定義 rollback/failure behavior
9. responsibility transfer 以 owner 是否唯一且可驗證為完成標準
10. behavioral parity、cross-process contract 與 runtime lifecycle 必須一起驗證
11. compatibility oracle/golden fixtures 必須在 production cutover 前建立並通過，不留到
    closure phase 才第一次定義
12. model artifact bytes 由 MTLF backend endpoint 直接提供給 PyAnLF；Go 只傳遞並驗證
    control metadata 與 apply result

---

## 12. Migration Phases

每個 phase 的細部計畫必須依實際 end-to-end 目標決定 repository/package scope，而不是預設
只修改 `PyMTLF/` 或 `NWDAF/internal/mtlf/`。若該 phase 需要調整 accuracy report、model
provision、bundle contract、runtime loading、generation/fallback state 或 cross-backend
routing，可直接把 `PyAnLF/`、`NWDAF/internal/anlf/` 與必要的 shared Go wiring/tests 納入
該 phase。細部計畫至少應列出：

1. 要修改的 repositories 與 packages
2. 變更的 internal contract 與 ownership impact
3. 需要保留的既有 AnLF behavior
4. repository-local tests 與 cross-process verification
5. 各 repository 分開的 commit/rollback boundary

這項授權只涵蓋完成 phase 目標所需的 interaction change；無關的 AnLF cleanup 或 architecture
refactor 仍不屬於本工作線。

### 12.1 Phase 1: PyMTLF Foundation And Backend Boundary

Detailed plan：`Phase 1 PyMTLF Foundation And Backend Boundary.md`

Planning status：implementation decisions confirmed；ready for implementation

目標：

1. 建立獨立 `PyMTLF/` repository 與 Python `src/` package layout
2. 定義 PyMTLF app、config、domain、API、trainer 與 test 基礎結構
3. 在 Go 建立 neutral `MtlfBackend` contract/client seam
4. 將 Go config、interface、client、field、logs、routes 與 tests 全部固定在
   `MtlfBackend`/`mtlfBackend` 語意，禁止 Python implementation naming 穿透
5. 定義 accuracy report、dataset request/attempt/chunk/completion、ModelReady、
   ModelProvisionEvent 與 ModelApplyResult contracts
6. 定義 Go-push chunk acknowledgement、attempt abort 與 backpressure contract
7. 在 owner migration 前由現有 Go MTLF behavior 建立 compatibility oracle/golden fixtures
8. 以 current PyAnLF loader 為 consumer oracle，定義 URL-based model bundle v1
9. 定義 PyMTLF persistent artifact repository、private immutable download endpoint、URL
   allowlist 與 retention boundary
10. 定義 PyMTLF generation journal、allocation、rollback 與 startup reconciliation protocol
11. 建立 health/startup/shutdown 與 cross-process contract smoke tests
12. 在 PyMTLF與 PyAnLF建立 Ruff lint baseline；PyAnLF只做必要、行為不變的 remediation
13. 保留現有 Go policy/training production path，Phase 1 不切換 owner

本 phase 不應：

1. 開始第二套 production decision policy
2. 讓 PyMTLF 直接接 ADRF/Mongo
3. 把現有 Daisy payload 當作 backend contract
4. 在 Go 新增任何 `PyMTLF`/`pyMtlf` implementation-specific identifier

### 12.2 Phase 2: Training Data Service And Local Store Formalization

目標：

1. 從現有 Go MTLF retrain workflow 抽出 Go-owned training data service
2. 建立 `ADRFProvider` 與 `MongoProvider`
3. 將 ADRF callback/fetch lifecycle 收斂到 provider-owned job
4. 實作 Go-to-MTLF-backend bounded chunk delivery、provider attempt abort 與 completion/failure
5. 正式化 MongoDB local training store、schema、snapshot、retention、indexes 與 query
6. 將 UPF dual-write 改成明確 config 與 observable policy
7. 驗證 ADRF 與 Mongo provider 的 common observation parity
8. 實作 dataset-request-level ADRF-to-Mongo all-or-nothing fallback，驗證 aborted attempt chunks
   不會進入 successful Mongo attempt

本 phase 可以用人工或測試 dataset request 驅動，不要求立即切換 accuracy policy owner。
ADRF provider 以 section 8.2 的 transition baseline 為驗收範圍；不因未完成 NRF discovery、
OAuth2、redirect、完整 error contract 或 TS 29.576 model regeneration 而阻擋本 phase。

### 12.3 Phase 3: Accuracy Policy And Retrain Migration

目標：

1. 完成 PyAnLF accuracy report 經 Go 到 PyMTLF 的 routing，但 production cutover 前只以
   replay/golden fixtures 或 isolated test flow 驗證，不讓同一 live report 同時由兩個 owner 決策
2. 遷移 report dedup、generation gate、per-scope state、degradation policy 與 retrain decision
3. 讓 PyMTLF 建立 dataset request 並消費 Phase 2 data pipeline
4. 實作 preprocessing、local trainer、candidate evaluation 與 training job lifecycle
5. 產生、驗證並原子發布 model bundle v1 到 PyMTLF artifact repository
6. 配置 base/target generation，持久化 generation journal 並產生 ModelReady
7. 在 isolated/non-production PyAnLF runtime 驗證 direct artifact GET、compare-and-swap apply
   與 ModelApplyResult round trip
8. 實作 PyMTLF restart reconciliation、pending generation recovery 與 stale event rejection
9. 以 Phase 1 oracle 完成 policy、training 與 generation behavioral parity
10. Phase 3 結束時保持現有 Go production owner；正式切換與 legacy removal 由 Phase 4 完成

### 12.4 Phase 4: Model Provision Integration And Legacy Removal

目標：

1. 原子切換 live accuracy report routing，使 PyMTLF 成為唯一 production decision/retrain owner
2. 啟用 production ModelReady -> Go -> PyAnLF -> Go -> PyMTLF apply/result flow
3. 由 PyAnLF 直接向 MTLF backend private endpoint 下載 immutable bundle，Go 不代理 bytes
4. 將 internal model URL semantic 對齊標準
   `MLEventNotif.mLFileAddr.mLModelUrl`
5. 驗證 replacement、apply failure、stale/conflict、NO_MATCH、rollback-as-new-generation 與
   late completion behavior
6. 移除或停用 Go-side accuracy policy execution，禁止 dual decision
7. 移除 Daisy-oriented startup automatic training，bootstrap 改由 config/artifact 提供
8. 移除 Go-side training scheduler、framework task state 與 legacy callback path
9. 移除 NWDAF config、client、server、tests、logs、comments 與 dependencies 中的 Daisy 語意
10. 確認 `PyMTLF/` 從建立起即不包含 Daisy 語意
11. 保留 Go standard Nnwdaf/ADRF consumers 與必要 coordination，不誤刪 standard MTLF
   procedure code
12. 不新增 standard `Nnwdaf_MLModelProvision` provider server，只驗證 future-compatible
    `mLModelUrl` mapping

### 12.5 Phase 5: Behavioral Parity And Closure

目標：

1. 重跑 Phase 1 已建立的 compatibility oracle/golden fixtures，不在 cutover 後才首次定義
2. 驗證 PyMTLF decision、retrain state、data completeness、generation journal 與 reconciliation
3. 執行 NWDAF/PyAnLF/PyMTLF cross-process live contract tests
4. 分別驗證 ADRF primary 與 Mongo fallback end-to-end flow
5. 執行 Go tests、race、lint、build，以及 Python unit/API tests與 Ruff
6. 在可用環境執行完整 5GC E2E；若環境不足，明確記錄未驗證層級
7. 稽核 ownership、config、dead code 與 Daisy removal
8. 稽核 MTLF backend artifact URL、retention、apply-result feedback 與 absence of Go bytes proxy
9. 更新主計畫狀態與各 repository 實作 commits

---

## 13. Verification Strategy

### 13.1 PyMTLF Unit Tests

至少覆蓋：

1. accuracy report validation/deduplication
2. stale/new generation transition
3. per-model/per-scope policy state
4. baseline、threshold、cooldown 與 retrain gate parity
5. concurrent retrain suppression
6. dataset completeness validation
7. preprocessing determinism
8. local trainer success/failure/cancellation
9. candidate acceptance/rejection
10. ModelReady event idempotency
11. bundle manifest/config generation 與 deterministic packaging
12. bundle checksum 與 immutable artifact identity
13. per-identity monotonic generation allocation 與 non-reuse
14. apply failure 後 generation gap、rollback-as-new-generation 與 active generation transition
15. durable generation journal transaction、restart reconciliation 與 unresolved pending event handling
16. ModelApplyResult 的 `APPLIED`、`FAILED`、`STALE`、`NO_MATCH`、`CONFLICT` state transition
17. artifact publish atomicity、retention 與 pending/confirmed cleanup protection

### 13.2 PyMTLF API Tests

至少覆蓋：

1. request parsing 與 validation
2. duplicate request handling
3. chunk ordering、duplicate、missing chunk 與 backpressure
4. provider attempt abort、late chunk rejection 與 dataset completion/failure
5. training cancellation
6. ModelReady validation、retry 與 delivery
7. ModelApplyResult validation、duplicate handling 與 state update
8. private immutable artifact endpoint、digest mismatch、not-found 與 download limits
9. health、startup、reconciliation gate 與 shutdown

### 13.3 PyAnLF Tests

至少覆蓋：

1. ModelProvisionEvent identity、base/target generation、URL 與 digest validation
2. zero、single 與 multiple matching runtime resolution
3. `NO_MATCH`、`STALE`、`CONFLICT` 與 idempotent `APPLIED` behavior
4. MTLF backend artifact URL allowlist、timeout、redirect、size 與 digest enforcement
5. corrupt/incomplete bundle、unsafe archive 與 incompatible runtime rejection
6. candidate load failure 時舊 generation 不變
7. multiple matching runtimes 的 all-or-nothing atomic activation
8. ModelApplyResult 的 active generation、affected runtime count 與 failure detail

### 13.4 NWDAF Go Tests

至少覆蓋：

1. backend client contract
2. accuracy report routing
3. ADRF RetrievalSubscribe/Notify/Request/Unsubscribe lifecycle
4. ADRF request timeout、watchdog、late callback 與 cancellation
5. Mongo query scope、time range、pagination 與 snapshot
6. dataset-request-level source selection、attempt abort 與 all-or-nothing fallback semantics
7. chunk retry/backpressure/idempotency，以及 aborted attempt chunk rejection
8. ModelReady 與 ModelApplyResult bidirectional routing
9. shutdown goroutine ownership
10. absence of Go-side duplicate policy execution
11. artifact URL scheme/host/identity metadata validation，以及 Go 不儲存或代理 artifact bytes
12. standard `mLFileAddr.mLModelUrl` mapping
13. Go config/API/client/log/test naming 只使用 `MtlfBackend`/`mtlfBackend` semantics
14. Go 不配置、增加或重用 model generation

### 13.5 Integration Tests

至少規劃：

1. NWDAF <-> PyMTLF real HTTP contract
2. PyAnLF -> NWDAF -> PyMTLF accuracy report flow
3. mock ADRF -> NWDAF -> PyMTLF dataset flow
4. MongoDB -> NWDAF -> PyMTLF dataset flow
5. PyMTLF ModelReady -> NWDAF -> PyAnLF -> NWDAF -> PyMTLF ModelApplyResult flow
6. ADRF partial/timeout -> attempt abort -> complete Mongo retry，且 successful dataset 不混合 provider
7. both providers unavailable -> retrain failure with old model retained
8. full local training and model replacement
9. PyMTLF artifact endpoint -> PyAnLF direct download、digest verification 與 atomic load
10. corrupt/incomplete/oversized bundle -> old generation retained
11. PyMTLF restart during retrain -> job failed without model switch
12. PyMTLF restart with pending ModelReady -> reconcile exact generation without duplicate allocation
13. `FAILED`/`STALE`/`NO_MATCH`/`CONFLICT` apply result -> old generation retained and pending state resolved
14. rollback request -> old artifact content applied under a newly allocated generation
15. multiple matching PyAnLF runtimes -> all switch to exact target generation or none switch

### 13.6 Completion Audit

完成前至少確認：

1. `NWDAF/` 不含 Daisy code/config/API/dependency/test semantic
2. `PyMTLF/` 不含 Daisy code/config/API/dependency/test semantic
3. Go 不再執行 accuracy decision policy 或 training algorithm
4. PyMTLF 不含 ADRF/Mongo direct client
5. PyAnLF 不直接呼叫 PyMTLF control/event API；唯一例外是使用 Go 已驗證並轉送的 URL，向
   MTLF backend artifact endpoint 執行 model bundle GET
6. standard SBI code 仍使用 generated OpenAPI models/clients where available
7. local Mongo provider 被清楚標示為 private implementation，不宣稱標準 ADRF
8. model provision event 只傳 URL/reference，不嵌入 model binary
9. future external-facing standard provider 是 NWDAF Go standard layer，不是 PyMTLF service；
   本 transition 未新增該 provider server
10. `NWDAF/` Go source、config、routes、logs 與 test fixtures 搜尋不到 `PyMTLF`、
    `PyMtlf`、`pyMtlf` 或 `pymtlf` implementation naming
11. PyMTLF 是 generation allocation 與 durable journal 的唯一 owner，Go 與 PyAnLF 均不自行
    增加 generation
12. Go 不儲存、下載或代理 model artifact bytes
13. 每個 successful training dataset 只包含單一 completed provider attempt 的 observations

---

## 14. Decision Gates And Open Questions

high-level ownership、generation、artifact handoff、apply-result 與 provider fallback semantics 已由
section 4 定案。Phase 1 detailed plan又確認了 Python/package/port、private API version/auth、SQLite、
artifact identity/resource limits、retention與 lint baseline。以下只剩會影響後續 phase實作 contract或
deployment的 exact choices，不重新打開已確認的 architecture boundary。

以下項目尚未定案，必須在對應 phase plan 開始前由使用者確認，不得由實作者自行選擇：

1. dataset chunk acknowledgement、retry 與同一 live attempt 內的 resume protocol；aborted
   attempt 與 process restart 不跨 attempt resume
2. common `TrainingObservation` exact schema 與 schema version policy
3. MongoDB database/collection naming、retention duration、index set 與 migration policy
4. ADRF request timeout/retry/watchdog 的 exact 數值，以及 valid empty result 是否可由明確
   `fallbackOnEmpty` config 覆寫預設的 no-fallback behavior
5. partial dataset 是否可訓練，以及最低 completeness threshold
6. local training algorithm、fixture dataset 與 candidate evaluation criteria
7. bundle v1 supported runtime compatibility matrix；manifest fields、archive limits與 validation
   baseline已在 Phase 1 plan固定
8. trusted-local `model.py`/pickle bundle 何時及如何升級成可接受 external NWDAF artifact 的
    signed or safer format
9. manual training operation 是否需要，以及其 owner/authorization semantic
10. 現有 `externalMtlf` config/test naming 如何收斂成「另一個含 MTLF 的 NWDAF」語意；
    standard consumer capability 不得與 legacy framework cleanup 一起誤刪
11. full 5GC E2E 所需 ADRF/Mongo/PyAnLF/PyMTLF environment 與驗證時間點

以下項目已明確 deferred，不是上述 transition decision gate：

1. 以 generated client/model 取代 current raw/ad hoc ADRF contract handling
2. NRF-based ADRF discovery 與 OAuth2 client credentials flow
3. 307/308 SBI redirect 與完整 ProblemDetails/error handling
4. supported feature negotiation 與 `EnhDataMgmt` extension coverage
5. 取得 TS 29.576 exact contract 後重新核對 `FetchInstruction` 與 expiry semantics
6. 導入 `Nadrf_MLModelManagement`，以及處理 TS 29.575 V18.11 text 與目前 V18.7
   attachment 間的 schema/cardinality 差異
7. 實作並透過 NRF advertisement 暴露 standard `Nnwdaf_MLModelProvision` provider server
8. 當 MTLF backend 不可被另一個 NWDAF 直接到達時，由 Go 提供 external artifact
   proxy/mirror、授權轉換或一次性標準下載 URL

若實作過程發現 current code、spec 與本計畫衝突，必須依 development policy 提出 blocker
report，說明原假設、矛盾、選項、建議與是否需要 replan。

---

## 15. Completion Criteria

這條工作線只有在以下條件全部成立時才可標記完成：

1. `PyMTLF/` 是可獨立測試與運行的 internal backend repository
2. PyMTLF 是 accuracy policy、retrain decision、training execution、generation allocation 與
   generation journal 的唯一 owner
3. Go 是 ADRF 與其他 standard communication 的唯一 owner
4. PyMTLF 不作為獨立 NF，也不直接存取 ADRF 或 MongoDB
5. PyAnLF accuracy report 可經 Go 穩定送達 PyMTLF
6. ADRFProvider 可由 Go 依 transition baseline 完成 subscription/fetch/cleanup 並串流 dataset
7. MongoProvider 可提供可重現、可分批、具 retention 與 completeness metadata 的 dataset
8. provider fallback 可設定、可觀測、以完整 provider attempt 為單位，且 successful dataset
   不會混合 ADRF 與 Mongo observations
9. local trainer 可完成 dataset-to-artifact workflow
10. PyMTLF 可產生 versioned model bundle，並由自己的 persistent private immutable URL 提供下載
11. PyAnLF 可依 Go 驗證並轉送的 model URL 直接下載、驗證，並以 base/target generation
    compare-and-swap 對完整 matching runtime set 執行 all-or-nothing 原子載入
12. ModelReady 可經 Go 交給 PyAnLF，完整 ModelApplyResult 可經 Go 回到 PyMTLF，且
    `APPLIED`/failure/stale/no-match/conflict behavior 正確
13. 同一 artifact semantic 可由 Go 映射到標準 `mLFileAddr.mLModelUrl`
14. Go 不再保留 duplicated accuracy policy、retrain business state 或 training algorithm
15. Daisy-oriented startup training 已移除；現有 standard ML Model Provision consumer capability
    仍保留，而 standard provider server 明確維持 deferred
16. `NWDAF/` 與 `PyMTLF/` 不再包含 Daisy-specific implementation semantic
17. `NWDAF/` 的 config、code、routes、logs 與 tests 只使用 `MTLF backend` naming，不含
    `PyMTLF` implementation name
18. repository-level tests、cross-process contract 與可執行的 integration verification 完成
19. 未執行的 full-environment validation 被明確記錄，不以 unit tests 代替 E2E 結論
20. PyMTLF restart 可由 durable journal 與 PyAnLF active state reconcile pending generation，且不
    重用已配置 generation
21. rollback 以新的 generation 指向既有 artifact content，不降低 active generation
22. Go 不儲存或代理 model artifact bytes，也不自行配置 generation

---

## 16. Relation To Other Plans

### 16.1 Relation To AnLF Backend Transition

這條工作線直接承接 AnLF Phase 4 已確認的最終邊界：

1. PyAnLF 產生 accuracy information
2. Go 保持 cross-domain coordination
3. 目前 Go MTLF 擁有 decision/retrain policy
4. 本計畫將第 3 項 ownership 遷移到 PyMTLF

AnLF transition plan 不再作為 MTLF implementation 的 canonical planning entry；後續 MTLF
decisions 與 progress 以本資料夾為準。這不表示 `PyAnLF/` 或 `NWDAF/internal/anlf/` code
freeze。當 MTLF phase 的 end-to-end interaction 需要 AnLF-side change 時，該 phase plan
應直接納入並驗證這些修改；只有在改變長期 AnLF responsibility boundary 時，才需要同步
更新 AnLF architecture/transition 文件。

### 16.2 Relation To Free5GC Alignment

這不是新的 free5GC NF，也不是把 Python backend 做成標準 SBI component。

free5GC alignment 在本計畫中只負責：

1. 維持 Go standard consumer、processor、context/store 與 lifecycle shape
2. 維持 generated OpenAPI contract 使用方式
3. 參考 NF-private MongoDB persistence ownership

PyMTLF internal domain architecture 以本專案需求、TS 23.288 MTLF/AnLF responsibility 與
跨 repo contract clarity 為主要依據。

### 16.3 Future Multiple-NWDAF FL Work

multiple-NWDAF federated learning 是後續獨立工作線。它應接在 PyMTLF trainer boundary，
不改變：

1. PyAnLF -> Go -> PyMTLF accuracy flow
2. PyMTLF -> Go training data request
3. Go ADRF/Mongo provider ownership
4. PyMTLF -> Go -> PyAnLF -> Go -> PyMTLF model ready/apply-result control flow
5. PyAnLF 依 Go provision 的 URL，直接向 MTLF backend artifact endpoint 取得 model bytes

這能確保目前 local training transition 不會再次把特定 training framework 寫死進 NWDAF。
