# Phase 1 PyMTLF Foundation And Backend Boundary

Date: 2026-07-17

Status: Phase 1 implementation decisions confirmed; ready for implementation

Parent plan:

- `nwdaf-docs/docs/plans/mtlf-backend-transition/MTLF Backend Transition Plan.md`

Expected repositories:

1. `PyMTLF/` — new repository created by this phase
2. `NWDAF/`
3. `PyAnLF/` — consumer-oracle tests and only the compatibility changes proven necessary by those tests
4. `nwdaf-docs/`

---

## 1. Purpose

這份文件定義 `MTLF Backend Transition` 的 `Phase 1` 細部實作計畫。

本 phase 的目的不是立即把 production accuracy policy 或 training execution 從 Go 切換到
Python，而是先建立一條可被後續 phases 安全使用的 `MTLF backend` boundary：

1. 建立獨立、可測試、可啟停的 `PyMTLF/` service repository
2. 在 Go 建立不含 Python implementation naming 的 `MtlfBackend` contract/client seam
3. 固定 accuracy、dataset、model-ready、apply-result 與 reconciliation contract semantics
4. 建立 content-addressed artifact repository 與 current PyAnLF loader 的 bundle oracle
5. 建立 PyMTLF-owned generation journal 與 restart reconciliation domain seam
6. 在 production owner 切換前建立 current Go MTLF policy 的 compatibility oracle

Phase 1 完成後，系統仍由現有 Go MTLF path 執行 live accuracy decision、ADRF-assisted retrieval、
Daisy training 與 model provision。新 backend 在這個階段只提供 foundation、artifact fixture、
durable-state primitives 與 contract verification；不得形成第二套 live policy owner。

---

## 2. Goals

Phase 1 必須達成：

1. `PyMTLF/` 可獨立安裝、啟動、測試與停止
2. Python package使用 implementation kickoff核准的名稱（recommendation：`py_mtlf`），並與 Go
   domain naming明確分離
3. Go config、interface、client、logs、routes 與 tests 只使用 `mtlfBackend`/`MtlfBackend`
4. internal contract v1 的方向、欄位、identity、idempotency、failure 與 version policy 被固定
5. Go/PyMTLF兩端對 MTLF backend contract examples可完成 encode/decode/validation；target
   Go/PyAnLF model-apply schema則先固定 examples，留待 Phase 3 isolated contract cutover驗證
6. PyMTLF artifact repository 可原子發布並以 immutable URL 提供 deterministic bundle fixture
7. PyAnLF 可直接下載、驗證並載入 Phase 1 bundle fixture，不需要 Go 代理 bytes
8. generation allocation、artifact binding、pending apply 與 terminal result 可由 embedded SQLite
   transactional journal 表達並測試
9. startup reconciliation 有明確 state machine、typed outcomes 與 mockable Go state-query client
10. current Go policy 的 historical/current expected behavior 具有 language-neutral fixtures 與 provenance
11. lifecycle、timeout、shutdown、logging、config validation 與 tests 在 foundation 階段已形成穩定基礎
12. Phase 2 至 Phase 4 可以擴充同一 boundary，不必另建平行 backend API

---

## 3. Scope

### 3.1 Included

1. 新建 `PyMTLF/` repository 與 Python `src/` packaging
2. PyMTLF config loading、validation、logging、FastAPI app、health、startup 與 shutdown
3. PyMTLF core interfaces：artifact repository、generation journal、reconciliation client seam
4. private content-addressed artifact GET endpoint
5. bundle v1 contract 與 deterministic consumer fixture
6. Go `MtlfBackendConfig`、neutral backend interface、HTTP client 與 config validation
7. internal contract v1 model definitions與 examples
8. future route inventory與 capability activation phase
9. compatibility oracle fixture schema、provenance與 current Go baseline validation
10. PyAnLF loader/cache/archive validation的 consumer-oracle tests
11. PyAnLF導入 Ruff baseline，並完成必要、行為不變的 lint remediation
12. repository-local unit/API/lint tests與最小 cross-process smoke tests
13. documentation、implementation record與 repository-separated commit boundaries

### 3.2 Explicitly Excluded

1. live accuracy report routing切換到 PyMTLF
2. PyMTLF accuracy policy、retrain decision 或 training algorithm 的 production execution
3. Go-owned ADRF/Mongo provider implementation與 dataset chunk delivery
4. local trainer、candidate evaluation與正式 model generation
5. production ModelReady -> PyAnLF activation cutover
6. 將 model generation allocator 從 current PyAnLF production path 切換到 PyMTLF
7. 移除或停用 current Go MTLF policy、Daisy client、callback、task config 或 scheduler
8. 修改 standard `Nnwdaf_MLModelProvision` provider/consumer semantics
9. 新增 standard provider server、NRF advertisement 或 external artifact proxy
10. 讓 PyMTLF 直接呼叫 ADRF、MongoDB、NRF 或其他標準 NF service
11. 讓 PyAnLF 與 PyMTLF 建立 direct control/event API
12. automatic artifact garbage collection；Phase 1 先採 conservative retention
13. bundle signing、ONNX/TorchScript migration 或 untrusted external producer support
14. multiple-NWDAF federated learning

---

## 4. Planning Basis

### 4.1 Repository Baselines

本文件以 2026-07-17 local state 為基準：

1. `NWDAF@58a69a0`
2. `PyAnLF@d75be4c`
3. `nwdaf-docs@1a0e527`
4. workspace 尚未建立 `PyMTLF/`

在 Phase 1 實作開始前必須重新記錄實際 baseline；若上述 repositories 已前進，應先確認
current changes 是否改變 contract、file path 或 owner assumptions。

### 4.2 Required Local Sources

1. workspace `AGENTS.md`
2. `nwdaf-docs/docs/development_policy.md`
3. `free5gc-dev-skill/SKILL.md`
4. `free5gc-dev-skill/references/source-orientation.md`
5. `free5gc-dev-skill/references/exemplar-alignment.md`
6. `free5gc-dev-skill/references/nf-architecture.md`
7. `free5gc-dev-skill/references/sbi-development.md`
8. `free5gc-dev-skill/references/openapi-contract.md`
9. `free5gc-dev-skill/references/config-and-lifecycle.md`
10. `free5gc-dev-skill/references/concurrency-lifecycle.md`
11. `free5gc-dev-skill/references/testing.md`
12. parent `MTLF Backend Transition Plan.md`
13. AnLF transition `R0 Compatibility Oracle Framework.md`
14. AnLF transition `R3 Model Identity Generation And Provision Correctness.md`
15. `nwdaf-daisy-improvement-plan.md` bundle/cache implementation record
16. current `NWDAF/internal/mtlf/`、`NWDAF/internal/anlf/`、factory與service wiring
17. current `PyAnLF/src/py_anlf/`、tests、`docs/api.md`與 bundle loader

### 4.3 Free5GC Exemplar Use

Phase 1 的 Go-side alignment 使用 local reference tree：

1. primary exemplar：`NFs/nrf`
   - `pkg/app/app.go`
   - `pkg/service/init.go`
   - `pkg/factory/config.go`
   - 用於 app/config/service/lifecycle responsibility shape
2. secondary exemplar：`NFs/udm`
   - `pkg/service/init.go`
   - `internal/sbi/consumer/consumer.go`
   - `internal/sbi/consumer/nf_management_test.go`
   - 用於 injected client、context cancellation與 outbound HTTP test discipline

直接證據只支持 Go-side config、service wiring、client/test 與 shutdown shape。free5GC reference
沒有 PyMTLF、NWDAF internal Python backend 或 generation journal exemplar；Python package、SQLite、
artifact與 internal contract設計是依 current PyAnLF、NWDAF需求與 parent decisions所做的 project-specific
設計，不宣稱為 free5GC upstream pattern。

### 4.4 Standard Contract Boundary

Phase 1 API 是 Go NWDAF 與 internal MTLF backend 間的 private contract，不是 3GPP SBI。

因此：

1. private endpoints 不使用 `Nnwdaf_*` service name
2. private payload可使用清楚的 language-neutral domain models
3. 不為 private API 偽造 generated 3GPP types
4. standard-facing code仍使用 current generated OpenAPI models/clients
5. `mLModelUrl` 等欄位只在需要保持標準 semantic mapping時沿用 exact spelling

---

## 5. Current State And Constraints

### 5.1 Current Go MTLF Is The Production Owner

`NWDAF/internal/mtlf/` 目前同時擁有：

1. accuracy report dedup與 generation gate
2. per-model/per-scope policy state
3. degradation/chronic/low-traffic decision logic
4. retrain in-flight state
5. ADRF-assisted retrieval jobs
6. Daisy upload、task submit與 completion callback
7. model provision event dispatch
8. startup training scheduler

Phase 1 不修改上述 live control flow。任何新 backend client都不得被接到
`HandleModelAccuracyReport()`、`dispatchRetrain()` 或 current training completion path。

### 5.2 Current PyAnLF Owns Process-local Generation

current PyAnLF `ModelProvisionEvent`只帶 event、identity與 artifact，PyAnLF runtime manager在成功
apply時配置下一個 process-local generation。這是 AnLF R3 已修正且正在 production 使用的行為。

parent MTLF plan已決定未來改成：

1. PyMTLF配置 `base_generation`與 `target_generation`
2. PyAnLF只做 exact target compare-and-swap
3.完整 ModelApplyResult經 Go 回到 PyMTLF
4. generation allocation與 pending apply state持久化

Phase 1 只定義並測試這個 v1 target contract，不切換 current PyAnLF endpoint。真正 runtime contract
cutover屬於 Phase 3 isolated verification與 Phase 4 production activation。

### 5.3 Current Bundle Path Is Usable But Trusted-local

current PyAnLF loader已支援 HTTP URL、tar.gz、`config.json`、dynamic `model.py`、`model.npy`與
`scaler.pkl`。Phase 1沿用這條 consumer-compatible path，但必須補上：

1. content-addressed full-reference artifact key
2. archive與 per-file integrity metadata
3. immutable publish semantics
4. size、timeout、redirect與 safe extraction limits
5. generation不寫死在 bundle bytes；binding由 ModelReady/journal表達

`model.py`、pickle scaler與 NumPy pickle仍只允許 trusted deployment boundary。

### 5.4 Existing Ports And Servers

套用本次 Phase 1 decision 後的 local defaults：

1. PyAnLF backend：`127.0.0.1:9093`
2. NWDAF AnLF callback server：`127.0.0.1:9091`
3. NWDAF MTLF auxiliary callback server：`127.0.0.1:8091`
4. Daisy endpoint：`127.0.0.1:9887`

`9090`保留給 local Prometheus 的常見 default，PyMTLF使用 `127.0.0.1:9092`，PyAnLF改用
`127.0.0.1:9093`；NWDAF config與 PyAnLF runtime fallback必須同步，不能只修改其中一側。

### 5.5 Existing Go Client Package Collision

`NWDAF/internal/mtlf/client/` 目前已由 Daisy `Client`/`NewClient` 使用。Phase 1 不應在同一 package
加入第二個含糊的 `Client`。新的 neutral backend client建議放在：

```text
NWDAF/internal/mtlf/backend/
```

package可使用 `backend`，公開 type使用 `Client`；其完整 Go path與 config/log semantics仍是
`mtlf backend`，不包含 Python implementation name。

---

## 6. Inherited Confirmed Decisions

以下來自 parent plan，Phase 1 不重新選擇：

1. PyMTLF不會成為獨立標準 NF
2. Go保留 ADRF與所有 standard communication
3. Go naming不得包含 `PyMTLF`/`pyMtlf`/`pymtlf`
4. PyMTLF不直接存取 ADRF或 MongoDB
5. PyAnLF/PyMTLF control與 event coordination都經 Go
6. artifact bytes可由 PyMTLF private endpoint直接送到 PyAnLF
7. Go不下載、保存或代理 artifact bytes
8. artifact採 URL reference，不嵌入 control event
9. PyMTLF是 future generation allocator與 durable journal owner
10. generation以 `(provider_id, model_unique_id)` 單調遞增且不重用
11. candidate通過且 content-addressed artifact完成 immutable publish後才配置 generation
12. rollback以新 generation綁定舊 artifact digest
13.只有 `APPLIED`可更新 active/confirmed generation
14.第一版 accuracy window可 process-local，但 generation/provision terminal state必須 durable
15. Phase 1不切換 live owner，也不新增 standard ML Model Provision provider server
16. Phase 1若 consumer oracle證明必要，可修改 PyAnLF或 Go AnLF package，但不得提前改 owner

---

## 7. Phase 1 Confirmed Exact Decisions

以下 baseline 已於 2026-07-17 確認，可直接作為 implementation kickoff input。除非實作證據觸發
section 26 blocker/replan condition，實作時不得無聲替換。

### 7.1 Repository And Runtime

1. repository：`PyMTLF/`
2. distribution/package：`py_mtlf`
3. Python：`>=3.12`，與 current PyAnLF一致
4. package manager/build：`uv` + `hatchling`
5. HTTP framework：FastAPI + Pydantic v2 + Uvicorn
6. config：YAML輸入，啟動時轉成 typed immutable settings
7. SQLite：Python standard library `sqlite3`
8. default bind：`127.0.0.1:9092`
9. timestamps：timezone-aware UTC RFC 3339
10. JSON fields：private contract一律 `snake_case`；只對標準映射欄位保留 exact 3GPP spelling

### 7.2 Internal API Versioning

1. private API prefix：`/internal/v1`
2. payload帶 `contract_version: "1.0"`
3. minor-compatible additive fields必須 optional
4. required field、enum meaning或 identity semantic變更需要新的 major route/version
5. FastAPI generated OpenAPI是已註冊 PyMTLF inbound routes的 machine-readable source；
   contract-only future routes在啟用前以 Pydantic models/examples固定
6. Go inbound route與 Go type是 PyMTLF-to-Go operations的 source of truth
7.雙方以 cross-process tests而不是 shared runtime package防止 drift

### 7.3 Authentication And Deployment Boundary

1. Phase 1不實作 application-level auth或 native TLS
2. default只 bind loopback，視為 trusted internal deployment
3. config必須分開 `binding_host`與 `artifact_public_base_url`
4. non-loopback/container deployment必須明確配置 allowed origins與 network policy
5.不把 no-auth baseline描述成 external NWDAF可接受的安全模型
6. future auth/TLS不得改變 domain payload或 generation identity

### 7.4 Conservative Artifact Retention

1. Phase 1不執行 automatic GC
2.所有成功 publish的 immutable artifacts保留
3.提供 repository-level protected-delete primitive與 metadata，但不掛 scheduler
4. Phase 3/4有 ModelApplyResult與 rollback retention後才啟用 cleanup policy

---

## 8. Phase 1 Target Shape

```text
                         production path remains unchanged

PyAnLF -- accuracy report --> NWDAF Go --> current Go MTLF policy/Daisy

                         Phase 1 foundation path

NWDAF Go
  |-- MtlfBackendConfig (disabled by default)
  |-- MtlfBackendAPI contract
  |-- neutral HTTP client + contract tests
  `-- future callback contract types (no live routing)
                    |
                    | private /internal/v1 contract
                    v
PyMTLF
  |-- typed config and app lifecycle
  |-- health/readiness
  |-- contract models
  |-- SQLite generation journal
  |-- reconciliation engine with mocked Go state client
  `-- content-addressed artifact repository
                    |
                    | GET immutable artifact URL
                    v
PyAnLF consumer oracle
  `-- download -> verify -> safe extract -> load fixture
```

Important invariants：

1. Phase 1沒有 live report dual delivery
2. Phase 1沒有 second policy state
3. Phase 1沒有 training dispatch
4. Phase 1 journal不會從 production event配置 generation
5. Go config預設 `mtlfBackend.enabled: false`
6. artifact smoke path不代表 model activation production cutover

---

## 9. PyMTLF Repository Layout

推薦初始 layout：

```text
PyMTLF/
├── README.md
├── .gitignore
├── pyproject.toml
├── uv.lock
├── run.py
├── config/
│   └── config.yaml
├── docs/
│   └── api.md
├── src/
│   └── py_mtlf/
│       ├── __init__.py
│       ├── app.py
│       ├── config.py
│       ├── models.py
│       ├── core/
│       │   ├── __init__.py
│       │   ├── artifacts.py
│       │   ├── generation_journal.py
│       │   └── reconciliation.py
│       ├── client/
│       │   ├── __init__.py
│       │   └── nwdaf.py
│       └── api/
│           ├── __init__.py
│           ├── health.py
│           └── artifacts.py
└── tests/
    ├── fixtures/
    │   ├── backend_contract_v1/
    │   ├── policy_compatibility/
    │   └── model_bundle_v1/
    ├── test_config.py
    ├── test_health_api.py
    ├── test_contract_models.py
    ├── test_artifact_repository.py
    ├── test_artifact_api.py
    ├── test_generation_journal.py
    └── test_reconciliation.py
```

Layout rules：

1. `api/`只負責 HTTP parsing、response與 error mapping
2. `core/`擁有 domain state與 transaction
3. `client/`只負責 outbound Go internal API
4. Pydantic request/response models不承載 policy algorithm
5. SQLite connection與 filesystem path不放進 global module state
6. app construction使用 dependency injection，tests可注入 temporary DB、artifact root與 fake client
7.不因參考 PyAnLF而複製其 model runtime、analytics或 accuracy implementation
8. `.gitignore`至少排除 `data/`、Python cache、build output與 local virtual environment；SQLite、
   published artifacts與其他 runtime state不得進入 git

---

## 10. Internal Contract Ownership And Conventions

### 10.1 Contract Ownership

| Direction | Canonical implementation source | Mirror/verification |
| --- | --- | --- |
| Go -> PyMTLF | Phase 1為 PyMTLF Pydantic model/examples；route啟用後由 FastAPI OpenAPI固定 | Go contract types/client tests + live test |
| PyMTLF -> Go | Go contract type + route/processor interface | PyMTLF typed client tests + live test |
| Go -> PyAnLF model apply | Phase 1 target examples + Go contract；Phase 3啟用時由 PyAnLF Pydantic model固定 | current client/API tests + Phase 3 live test |
| PyMTLF artifact -> PyAnLF | PyMTLF artifact metadata/HTTP behavior | PyAnLF consumer-oracle tests |
| Standard Nnwdaf/ADRF | local generated OpenAPI package | existing Go SBI tests/spec audit |

兩個 repositories不得在 runtime互相 import，也不得依賴 sibling filesystem。Cross-workspace tests可由
explicit environment variable啟用，但 repository-local tests必須可獨立執行。

### 10.2 Shared Python API Template And Envelope Rules

PyMTLF參考 current PyAnLF的 service shape，但「統一模板」只表示兩個獨立 repositories遵循同一套
可觀察 contract慣例，不建立 shared runtime package：

1. FastAPI + Pydantic v2 + Uvicorn，並提供 generated OpenAPI
2. request/response採 typed Pydantic models，HTTP router不承載 domain policy
3. internal fields使用 `snake_case`；只有直接映射 3GPP model provision的欄位保留
   `mLModelUrl`、`modelUniqueId`等 exact spelling
4.時間使用帶 timezone的 UTC RFC 3339
5. state-changing message帶 stable ID、contract version與 typed error
6. config、app construction與 dependency injection可由 tests覆寫，不依賴 sibling repository import
7. repository-local tests使用 pytest；PyMTLF與 PyAnLF都使用 Ruff作為 lint baseline

current PyAnLF routes沒有 version prefix，而且已被 Go呼叫。Phase 1不為了形式統一而破壞性搬移這些
routes；PyMTLF新 private API使用 `/internal/v1`，Go inbound API使用 `/mtlf-backend/v1`。未來若要
version PyAnLF routes，必須先加相容 alias並另行安排 caller migration。

其中 route path的 `v1`表示 HTTP contract major version；payload的
`contract_version: "1.0"`讓 persisted event、fixture與 log即使離開原始 URL仍可辨識 exact schema。
兩者不是 Python package version，也不是 3GPP API version。

所有 state-changing private messages至少遵守：

1. `contract_version`
2. stable operation/event/request ID
3. explicit model identity where applicable
4. timezone-aware timestamp
5. idempotency semantic
6. typed terminal status或 error code
7. unknown additive fields可忽略，但 unknown enum不得靜默接受
8. request body有明確 size limit
9. log correlation使用 ID，不記錄完整 observations、model bytes或 secrets

### 10.3 Private Error Shape

建議 private error response：

```json
{
  "code": "STALE_GENERATION",
  "message": "active generation does not match base generation",
  "retryable": false,
  "correlation_id": "event-or-request-id"
}
```

Phase 1應固定：

1. malformed JSON：`400`
2. Pydantic/typed validation：`422`
3. unknown resource：`404`
4. idempotency/state conflict：`409`
5. body/artifact too large：`413`
6. dependency/reconciliation unavailable：`503`
7. unexpected internal failure：`500`，response不得洩漏 stack或 local path

這是 private API error，不使用 `models.ProblemDetails`。標準 SBI仍維持既有 ProblemDetails handling。

### 10.4 Concrete Request Shapes

以下 examples用來說明 caller、server與 URL ownership；Phase 1只固定 schema，仍依 section 12的
activation phase決定何時註冊 live route。

Go將 PyAnLF accuracy report正規化後送給 PyMTLF：

```http
POST http://127.0.0.1:9092/internal/v1/accuracy-reports
Content-Type: application/json

{
  "contract_version": "1.0",
  "report_id": "acc-7f9e",
  "report_sequence": 18,
  "generated_at": "2026-07-17T09:30:00Z",
  "model_identity": {
    "provider_id": "local",
    "model_unique_id": 1
  },
  "generation": 4,
  "monitoring_context": {
    "analytics_event": "UE_COMMUNICATION",
    "target_ue": {},
    "event_filter": {},
    "scope_id": "scope-a"
  },
  "accuracy_information": {
    "metrics": {"WAPE": 0.1},
    "deviation": 0.1,
    "sample_count": 2,
    "inference_count": 2,
    "actual_traffic_scale": 10240.0,
    "predicted_traffic_scale": 11264.0,
    "window_start": "2026-07-17T09:00:00Z",
    "window_end": "2026-07-17T09:30:00Z"
  },
  "retrain_context": {
    "subscription_ids": ["subscription-a"],
    "observation_source_ids": ["source-a"]
  }
}
```

PyMTLF需要訓練資料時呼叫 Go-owned MTLF auxiliary server：

```http
POST http://127.0.0.1:8091/mtlf-backend/v1/dataset-requests
Content-Type: application/json

{
  "contract_version": "1.0",
  "dataset_request_id": "dataset-2c41",
  "training_job_id": "job-123",
  "model_identity": {
    "provider_id": "local",
    "model_unique_id": 1
  },
  "base_generation": 4,
  "analytics_event": "UE_COMMUNICATION",
  "scope": {},
  "observation_source_ids": [],
  "time_window": {
    "from": "2026-07-16T00:00:00Z",
    "to": "2026-07-17T00:00:00Z"
  },
  "provider_preference": "ADRF",
  "allow_fallback": true,
  "observation_schema_version": "1.0"
}
```

Go收到 ModelReady後，仍呼叫 current PyAnLF endpoint family；PyAnLF的新 default base URL是
`http://127.0.0.1:9093`：

```http
POST http://127.0.0.1:9093/model-provision-events
```

request body依 section 11.7帶 exact `base_generation`、`target_generation`與 artifact metadata。
PyAnLF再直接對 PyMTLF的
`GET http://127.0.0.1:9092/internal/v1/artifacts/{sha256}`下載 bytes；Go不代理下載。

---

## 11. Contract Families

Phase 1 必須把以下 types定義成 exact JSON examples與雙邊 validation tests。除 health與 artifact GET
以外，routes依各 phase啟用，不因 schema已存在就提早接 production flow。

### 11.1 Model Identity

```json
{
  "provider_id": "local",
  "model_unique_id": 1
}
```

Rules：

1. `provider_id`為 trim後非空字串
2. `model_unique_id`為 non-negative int64，維持 current standard-facing identity range
3. generation key固定是這兩欄，不包含 URL、scope或 process ID

### 11.2 Accuracy Report

v1沿用 current `ModelAccuracyReport` domain meaning，至少包含：

1. `contract_version`
2. `report_id`
3. `report_sequence`
4. `generated_at`
5. `model_identity`
6. `generation`
7. `monitoring_context`
8. `accuracy_information`
9. `retrain_context`

Phase 1 oracle必須證明 Go mirror沒有丟失 current PyAnLF fields。Phase 3才把 live report由 Go轉給
backend；Phase 1不得同時交給 current Go policy與 backend，也不得在 current PyAnLF -> Go ingress
payload強制新增 `contract_version`。Version envelope由 future Go -> MTLF backend conversion產生。

### 11.3 Dataset Request

PyMTLF -> Go：

1. `contract_version`
2. `dataset_request_id`
3. `training_job_id`
4. `model_identity`
5. `base_generation`
6. `analytics_event`
7. normalized `scope`
8. `observation_source_ids`
9. `[from, to)` requested time window
10. provider preference/fallback permission
11. observation schema version

Request只描述資料需求，不包含 ADRF fetch URI、Fetch Correlation ID、subscription ID、Mongo query或
credentials。Live route由 Phase 2實作。

### 11.4 Dataset Attempt And Chunk

Go -> PyMTLF：

1. `dataset_request_id`
2. `provider_attempt_id`
3. `provider`：`ADRF`或 `MONGODB`
4. `chunk_index`，從 0單調遞增
5. `snapshot_id`/cutoff metadata
6. normalized observations
7. chunk idempotency key
8. schema version

Ack至少表達 `ACCEPTED`、`DUPLICATE`或 `REJECTED`，並可提供 `next_expected_chunk` 作 bounded
delivery flow control。Exact retry/resume window仍是 Phase 2 implementation detail；aborted attempt
永不跨 attempt resume。

### 11.5 Dataset Completion, Abort And Failure

Completion至少包含：

1. completed attempt ID
2. actual provider
3. record/chunk count
4. effective time window與 snapshot
5. completeness metadata
6. duplicate/drop summary
7. fallback reason

Abort/failure至少包含 typed reason、retryable、occurred_at與已送 chunks count。PyMTLF收到 abort後
必須丟棄整個 attempt。Live route與 all-or-nothing behavior由 Phase 2完成。

### 11.6 ModelReady

PyMTLF -> Go：

```json
{
  "contract_version": "1.0",
  "event_id": "opaque-uuid",
  "training_job_id": "job-123",
  "model_identity": {
    "provider_id": "local",
    "model_unique_id": 1
  },
  "base_generation": 3,
  "target_generation": 4,
  "artifact": {
    "url": "http://127.0.0.1:9092/internal/v1/artifacts/<sha256>",
    "digest_algorithm": "sha-256",
    "digest": "<lowercase-hex>",
    "size_bytes": 12345,
    "media_type": "application/gzip",
    "bundle_schema_version": "1.0"
  },
  "analytics_event": "UE_COMMUNICATION",
  "published_at": "2026-07-17T00:00:00Z"
}
```

Rules：

1. event ID為 opaque stable UUID，不由 URL或 generation臨時計算
2. `target_generation > base_generation`
3.正常 candidate target是 last allocated + 1
4. rollback仍配置新 target，但 artifact digest可指向舊 content
5. URL只允許 configured MTLF backend artifact origin
6. Go只驗證/轉送 metadata，不 GET bytes
7. Phase 1只定義 contract與 journal tests；live callback由 Phase 3 isolated path建立

### 11.7 ModelProvisionEvent To PyAnLF

Go -> PyAnLF。Target event沿用 current private endpoint family，但增加 PyMTLF已配置的 generation與
artifact integrity metadata：

1. `contract_version`
2. `event_id`，原樣沿用 ModelReady ID
3. `source`
4. `model_identity`
5. `base_generation`
6. `target_generation`
7. `model_update_ind`
8. artifact `mLModelUrl`、digest、size、media type與 bundle schema version
9. `analytics_event`
10. notification correlation
11. `training_job_id`

Rules：

1. Go不得重新配置或增加 generation
2. Go不得把 URL改成自己的 proxy URL
3. PyAnLF驗證所有 matching runtimes都位於 base generation
4.任一 runtime不符合時整批回 `STALE`，不得部分切換
5. candidate完整載入後才 all-or-nothing切到 exact target generation
6. response直接回 ModelApplyResult給 Go；PyAnLF不 callback PyMTLF
7. Phase 1只固定 target schema，current production `/model-provision-events` behavior不變

### 11.8 ModelApplyResult

Go -> PyMTLF：

1. `contract_version`
2. `event_id`
3. `model_identity`
4. `target_generation`
5. `status`：`APPLIED`、`FAILED`、`STALE`、`NO_MATCH`、`CONFLICT`
6. `active_generation`
7. `active_artifact_digest` where available
8. `affected_runtime_count`
9. typed failure code/detail
10. `completed_at`

只有 `APPLIED`更新 confirmed active generation。相同 event/result必須 idempotent；相同 event搭配
不同 terminal result必須 conflict並保留 evidence。

### 11.9 Active Model State For Reconciliation

PyMTLF經 Go查詢，Go再讀取 PyAnLF authoritative runtime state。Response至少包含：

1. model identity
2. status：`CONSISTENT`、`NO_MATCH`、`DIVERGED`、`UNAVAILABLE`
3. active generation/digest when consistent
4. matching runtime count
5. observed_at
6. divergence summary，不能只回 arbitrary first runtime

若 matching runtimes不在同一 generation/digest，Go必須回 `DIVERGED`，PyMTLF不得自行選 winner。

---

## 12. Route Inventory And Activation Phase

### 12.1 PyMTLF Inbound Routes

| Method/path | Purpose | Defined | Live implementation |
| --- | --- | --- | --- |
| `GET /health/live` | process liveness | Phase 1 | Phase 1 |
| `GET /health/ready` | DB/artifact/reconciliation readiness | Phase 1 | Phase 1 |
| `GET /internal/v1/artifacts/{artifact_key}` | immutable bundle bytes | Phase 1 | Phase 1 |
| `POST /internal/v1/accuracy-reports` | accuracy delivery | Phase 1 | Phase 3 |
| `PUT /internal/v1/dataset-attempts/{attempt_id}/chunks/{index}` | chunk delivery | Phase 1 | Phase 2 |
| `POST /internal/v1/dataset-attempts/{attempt_id}/complete` | commit dataset | Phase 1 | Phase 2 |
| `POST /internal/v1/dataset-attempts/{attempt_id}/abort` | discard attempt | Phase 1 | Phase 2 |
| `POST /internal/v1/model-apply-results` | consume activation result | Phase 1 | Phase 3 isolated / Phase 4 production |

Contract-only routes不得以 dummy success handler出現在 production app。只有對應 owner已實作時才註冊
route；這可防止 caller把 `200/202`誤認為資料或 policy已被真正處理。

### 12.2 Go MTLF Backend Inbound Routes

建議使用 current MTLF auxiliary server並加入 neutral prefix：

| Method/path | Purpose | Defined | Live implementation |
| --- | --- | --- | --- |
| `POST /mtlf-backend/v1/dataset-requests` | request Go-owned data | Phase 1 | Phase 2 |
| `POST /mtlf-backend/v1/model-ready` | provision target artifact | Phase 1 | Phase 3 isolated / Phase 4 production |
| `GET /mtlf-backend/v1/models/{provider_id}/{model_unique_id}/active-state` | reconciliation state | Phase 1 | Phase 3 |

Phase 1不註冊上述 routes，避免尚未具備 processor semantics時暴露 false capability。Go tests先以
`httptest` fake server驗證 outbound client，Python client tests則以 mock transport驗證 future requests。

### 12.3 PyAnLF Target Routes

| Method/path | Purpose | Phase 1 behavior | Target activation |
| --- | --- | --- | --- |
| `POST /model-provision-events` | exact base/target generation apply | current contract不變；只固定 v1 target examples | Phase 3 isolated / Phase 4 production |
| `GET /model-identities/{provider_id}/{model_unique_id}/active-state` | authoritative reconciliation state | contract only，不註冊 route | Phase 3 |

Go收到 synchronous ModelApplyResult後，再呼叫 MTLF backend的
`POST /internal/v1/model-apply-results`。這條路徑不建立 PyAnLF -> PyMTLF direct callback。

---

## 13. PyMTLF App Lifecycle

### 13.1 Startup Order

1. load YAML
2. validate typed settings
3. configure logging without secrets或完整 filesystem dump
4.建立 artifact root與 state directory
5. open SQLite，設定 foreign keys、busy timeout與 schema version
6.執行 forward-only schema migration
7.驗證 journal invariants
8.掃描 pending provision events
9.若無 pending event，mark ready
10.若有 pending event，啟動 reconciliation；未收斂前 readiness為 false
11.建立 FastAPI routes與 dependencies
12.啟動 Uvicorn

### 13.2 Readiness

`/health/live`只表示 process event loop可回應。

`/health/ready`必須同時要求：

1. config valid
2. SQLite可讀寫且 schema current
3. artifact root可讀寫
4.沒有 unresolved journal corruption
5. startup reconciliation已完成或沒有 pending events

Phase 1尚未接 production routing，因此 Go startup不依賴 PyMTLF readiness；smoke/E2E tests才主動檢查。

### 13.3 Shutdown

1.停止接收新 state-changing request
2.取消 reconciliation/outbound HTTP contexts
3.等待 owned tasks在 bounded timeout內停止
4. rollback open transactions
5. close SQLite connections
6.不刪除 artifacts、journal或 pending evidence
7. shutdown不得等待 Go/PyAnLF network call無限期完成

---

## 14. Generation Journal

### 14.1 Persistence Boundary

Phase 1使用 embedded SQLite，建議 default：

```text
data/mtlf-state.sqlite3
```

DB path必須可由 config覆寫。Tests一律使用 temporary directory，不共用 developer state。

### 14.2 Minimum Tables

`model_generation_state`：

1. `provider_id`
2. `model_unique_id`
3. `last_allocated_generation`
4. `active_generation`
5. `active_artifact_digest`
6. `updated_at`
7. composite primary key `(provider_id, model_unique_id)`

`provision_events`：

1. `event_id` primary key
2. model identity
3. `training_job_id`
4. `base_generation`
5. `target_generation`
6. artifact key、URL、digest、size
7. state
8. apply status/code/detail
9. created/updated/completed timestamps
10. canonical payload digest for idempotency conflict detection

`training_job_terminal_state`：

1. `training_job_id` primary key
2. model identity
3. terminal status/reason
4. candidate artifact digest where applicable
5. completed_at

Phase 1不建立 policy window、dataset chunks或 half-trained model checkpoint tables。

### 14.3 Provision Event States

最低 state machine：

```text
artifact published
      |
      v
PENDING_DELIVERY -- ModelReady accepted by Go --> PENDING_APPLY
      |                                             |
      |                                             +--> APPLIED
      |                                             +--> FAILED
      |                                             +--> STALE
      |                                             +--> NO_MATCH
      |                                             `--> CONFLICT
      `-- delivery terminal failure -------------> FAILED
```

`APPLIED`、`FAILED`、`STALE`、`NO_MATCH`與 `CONFLICT`都是 durable terminal evidence；policy是否
建立新 job由後續 phase決定。Transport timeout本身不直接寫 terminal，因為 remote可能已處理。

### 14.4 Allocation Transaction

1. candidate acceptance完成
2. bundle在 temporary path完成、驗證與 checksum
3. artifact repository以 atomic rename完成 content-addressed publish
4. SQLite `BEGIN IMMEDIATE`
5.讀取 identity row與 last allocated/active generation
6.配置新的 target generation
7.建立 generation-to-digest binding與 `PENDING_DELIVERY` event
8.更新 last allocated generation
9. commit

若 filesystem publish成功但 DB transaction失敗，留下未綁定 orphan artifact；generation不得視為已配置。
Phase 1記錄 orphan並保留，不以不安全 cleanup掩蓋失敗。

### 14.5 Non-reuse And Rollback

1.任何已 commit的 target generation不得重用
2. apply failure後下一次 allocation仍使用更大值
3. rollback選擇舊 digest，但配置新 target generation
4. active generation只由 `APPLIED` transition更新
5. journal不得依 process-local counter或 filesystem listing推算下一 generation

### 14.6 SQLite Concurrency

1.所有 write由 narrow repository methods擁有
2.使用 transaction序列化同 identity allocation
3.設定 bounded busy timeout
4.不得跨 network call持有 transaction
5.不得在 DB lock內下載或產生 artifact
6.duplicate event/payload回原 row；same event/different payload為 conflict
7.tests必須覆蓋 concurrent allocation、process reopen與 failed transaction

---

## 15. Startup Reconciliation

### 15.1 Reconciliation Inputs

1. durable pending event
2. journal active/last allocated generation
3. target artifact digest
4.經 Go取得的 PyAnLF active state
5. stable event ID

### 15.2 Outcomes

| Observed state | Journal action |
| --- | --- |
| target generation + digest active | mark `APPLIED` and confirm active |
| base generation remains active | keep pending for bounded redelivery/retry policy |
| no matching runtime | record `NO_MATCH` only after explicit apply result；state query alone不偽造 terminal result |
| active generation greater than target | mark reconciliation conflict，停止新 decision |
| same target generation but different digest | mark `CONFLICT`，停止新 decision |
| runtimes diverged | keep not-ready and require operator/plan decision |
| Go/PyAnLF unavailable | keep not-ready，bounded retry，preserve journal |

### 15.3 Phase 1 Implementation Boundary

Phase 1實作 reconciliation engine、typed state client interface與 fake-client tests。因 Go active-state
route尚未 live，normal Phase 1 DB不會由 production產生 pending event。Live route與 real startup
reconciliation在 Phase 3接通；若 Phase 1 service發現手工/測試留下 pending row而無 state client，
readiness必須維持 false，不能假設失敗或自動配置新 generation。

---

## 16. Artifact Repository And Bundle V1

### 16.1 Content Addressing

1. artifact key為 archive SHA-256 lowercase hex
2. canonical URL：`/internal/v1/artifacts/{sha256_hex}`
3. URL不包含 `latest`或 mutable alias
4. URL不必包含 generation；ModelReady/journal建立 generation binding
5.同 content rollback重用 URL/digest
6. artifact內嵌的 model identity必須與 ModelReady一致；不得為了跨 identity reuse而忽略 bundle identity

### 16.2 Publish

1.只接受 application產生的 local candidate path，不提供任意 upload route
2.在 artifact root內建立 temporary file
3.計算 digest與 size
4.驗證 archive/bundle contract
5. flush/fsync where supported
6. atomic rename到 digest path
7.同 digest已存在時驗證 size/content後 idempotent reuse
8.不得把 local path回傳給 Go

### 16.3 HTTP Retrieval

Phase 1 endpoint至少提供：

1. `GET` immutable bytes
2. `Content-Type: application/gzip`
3. exact `Content-Length`
4. strong ETag或 equivalent digest header
5. `404` unknown digest
6. path traversal rejection
7. no directory listing
8. bounded file open/read errors
9. cache header可標示 immutable content

Phase 1不要求 Range request。PyAnLF仍必須以 ModelReady提供的 digest做 end-to-end verification，不能只
信任 URL或 ETag。

### 16.4 Bundle V1

初始 bundle維持 current PyAnLF-compatible四件套：

| File | Phase 1 requirement |
| --- | --- |
| `config.json` | schema version、model params、feature order、window、outputs、component filenames |
| `model.py` | trusted-local fixed `Model` entry point |
| `model.npy` | model weights |
| `scaler.pkl` | fitted preprocessing scaler |

manifest/config還必須表達：

1. bundle schema version
2. model identity
3. analytics event/use case
4. feature names/order/units
5. sequence length與 output fields
6. runtime/framework compatibility
7. per-file digests
8. creation time與 producer metadata

generation不寫進 immutable bundle identity。target generation只存在 ModelReady、journal與 PyAnLF active
state，避免 rollback到相同 content時重打包。

### 16.5 Consumer Security Oracle

PyAnLF tests至少驗證：

1. full URL + identity + target generation + digest cache separation
2. archive digest mismatch
3. oversized archive
4. decompression expansion limit
5. `..` path traversal
6. absolute path
7. symlink/hardlink abuse
8. missing/duplicate required file
9. invalid config/component filename
10. load failure保留舊 runtime

若 current PyAnLF缺少上述安全檢查，Phase 1允許補上 loader/cache behavior與 tests；但不得同時切換
generation allocator或 model apply control contract。

### 16.6 Initial Resource And Download Limits

Phase 1確認以下 conservative、configurable defaults：

1. compressed archive最大 `256 MiB`
2.解壓後全部 regular files合計最大 `1 GiB`
3.單一解壓檔案最大 `512 MiB`
4. archive entries最多 `32`
5. PyAnLF artifact download overall timeout預設 `300 seconds`
6. HTTP redirect預設拒絕；未來若允許，只能在 configured allowed origin內 bounded follow

`300 seconds`只用於 model artifact byte download，不套用到 accuracy、dataset request、ModelReady、
ModelApplyResult或 health等 control API。所有 limits都可由 typed config調整，但必須為正值、滿足
single-file不大於 total extracted limit，且不能停用 digest、origin或 safe-extraction validation。
若 deterministic consumer fixture或實際 model bundle證明預設值不足，可依測量結果調高並記錄在
implementation record；不得以移除 upper bound作為調整方式。

---

## 17. Go MTLF Backend Boundary

### 17.1 Config

在 `pkg/factory/config.go`新增獨立 config，不重用含 Daisy semantics的 current `MtlfConfig`：

```yaml
mtlfBackend:
  enabled: false
  endpoint: http://127.0.0.1:9092
  requestTimeout: 5
  artifactAllowedOrigins:
    - http://127.0.0.1:9092
```

Validation至少包含：

1. enabled時 endpoint required且為 HTTP/HTTPS absolute URL
2. request timeout positive
3. allowed origins只允許 scheme + host + optional port，不允許 path/query/userinfo
4. endpoint與 allowed origin normalization一致且可比較
5. optional config省略時 current production behavior不變

### 17.2 Interface

`internal/mtlf/backend.go`定義 narrow `MtlfBackendAPI`，語意方法可包含：

1. `Health`
2. `DeliverAccuracyReport`
3. `DeliverDatasetChunk`
4. `CompleteDatasetAttempt`
5. `AbortDatasetAttempt`
6. `DeliverModelApplyResult`

Phase 1 client可實作 future methods與 tests，但 production wiring只建立 optional dependency，不呼叫
state-changing methods。若 unused interface造成無意義 dead code，可將 methods依 Phase 2/3逐步加入；
contract types與 route table仍須在本文件固定。

### 17.3 Client

`internal/mtlf/backend/client.go`：

1.持有 normalized endpoint與 injected `http.Client`
2.每次 request接受 parent context
3.使用 bounded timeout
4. response body有 size limit
5. error保留 operation、status與 backend code
6.不在 client內執行 policy retry或 fallback
7.不產生/增加 generation
8.不下載 artifact
9. logs使用 `MTLF backend`，不含 implementation name

Retry policy在 operation semantics確定前不做 generic automatic retry。只有明確 idempotent operation在
後續 phase定義 stable ID後才加 bounded retry。

### 17.4 Service Wiring

`pkg/service/init.go`可在 `mtlfBackend.enabled`時建立 client並注入 `MtlfService`或 future coordinator
seam，但 Phase 1必須保證：

1. current `DaisyAPI`仍被 current training path使用
2. accuracy processor仍只呼叫 current `MtlfService.HandleModelAccuracyReport`
3. current model provision callback不經 backend round trip
4. backend不可用不影響 default NWDAF startup，因 config預設 disabled
5.不啟動 shadow/dual decision

若注入後完全沒有 safe read-only use，Phase 1可只完成 config/client constructor tests而不保存 unused
field；Phase 2/3再於第一個實際 operation wiring。

---

## 18. PyAnLF Consumer Oracle

### 18.1 Required Verification

Phase 1以 current PyAnLF loader為 consumer oracle，建立由 PyMTLF artifact repository提供的 deterministic
bundle fixture，驗證：

1. HTTP GET成功
2. archive digest/size符合 metadata
3. current bundle四件套可載入
4. feature order與 output fields保持 current inference semantic
5. repeated same content可 cache reuse
6. corrupt/incomplete/unsafe archive拒絕
7. load failure不替換 active model

### 18.2 Allowed PyAnLF Changes

只允許 consumer oracle直接要求的：

1. digest與 size validation
2. URL allowlist/redirect/timeout/size settings
3. content-addressed cache key
4. safe extraction
5. tests與 API documentation

Phase 1不得：

1.將 current `ModelProvisionEvent` production route切到 base/target generation
2.移除 current process-local generation
3.加入 PyAnLF -> PyMTLF control callback
4.改變 runtime matching、accuracy或 scheduler owner

---

## 19. Compatibility Oracle

### 19.1 Purpose

Phase 3搬移 policy前，expected behavior必須先由 current Go production logic固定；不能等 Python policy
寫完後用 Python output反向建立 golden values。

### 19.2 Fixture Location

canonical fixtures建議放在：

```text
PyMTLF/tests/fixtures/policy_compatibility/
```

它們是 target policy的長期 regression assets，但 expected values必須引用 NWDAF provenance。Repository-
local tests不得依賴 sibling path；Go baseline verification可用對應 table-driven tests與文件記錄證明。

### 19.3 Required Metadata

每個 fixture至少包含：

1. unique fixture ID
2. classification：`historical_invariant`或 `approved_change`
3. repository/commit/source path/test path/test name provenance
4. logical clock
5. effective policy config
6. ordered report inputs
7. expected per-step state/result
8. retrain trigger expectation
9. expected dedup/stale behavior

### 19.4 Initial Fixture Families

1. cold start只建立 baseline，不 retrain
2. minimum sample gate
3. fixed-floor ineligible report
4. z-score degradation hit/window threshold
5. chronic mean/percentile path
6. low-traffic overprediction path
7. traffic-scale eligibility
8. per-scope isolation
9. stale scope GC with frozen clock
10. concurrent retrain suppression
11. report ID duplicate
12. stale/new generation transition
13. missing primary metric
14. retrain success/failure後 policy state behavior

Phase 1只要求 fixtures、schema、provenance與 current Go baseline proof。PyMTLF policy尚未存在，因此不把
fixture未能對 Python policy執行視為 Phase 1缺陷；Phase 3必須以同一 fixtures完成 parity。

---

## 20. Planned Repository Changes

### 20.1 PyMTLF

| Area | Planned change |
| --- | --- |
| project files | `pyproject.toml`、lockfile、README、run entrypoint |
| `.gitignore` | 排除 `data/`、cache、build output與 local environment |
| config | typed server/storage/artifact/Go-client settings與 validation |
| app | FastAPI construction、dependencies、startup/shutdown |
| models | contract v1 Pydantic models/enums/examples |
| artifact core | content addressing、atomic publish、metadata、safe retrieval |
| generation core | SQLite schema、transactions、state transitions |
| reconciliation | active-state client seam與 startup engine |
| API | live/ready health與 artifact GET |
| docs | private API/version/lifecycle/failure documentation |
| tests | config、models、journal、artifact、reconciliation、API與 fixtures |

### 20.2 NWDAF

| File/area | Planned change |
| --- | --- |
| `pkg/factory/config.go` | `MtlfBackendConfig`、validation與 getters |
| `pkg/factory/config_test.go` | valid/disabled/missing/invalid endpoint、origin與 timeout matrix |
| `config/nwdafcfg.yaml` | disabled-by-default `mtlfBackend` sample |
| `internal/mtlf/backend.go` | neutral backend interface |
| `internal/mtlf/contract/` | versioned backend contract types，不改 standard generated models |
| `internal/mtlf/backend/client.go` | private HTTP client |
| `internal/mtlf/backend/client_test.go` | request/response/error/timeout/size tests |
| `pkg/service/init.go` | optional client construction only when it has a safe injection seam |
| `pkg/service/init_test.go` | disabled behavior與 optional wiring |
| current MTLF tests | compatibility oracle provenance/baseline assertions |

Go source、config、logs、routes與 tests不得出現 `PyMTLF` implementation naming。

### 20.3 PyAnLF

| Area | Planned change |
| --- | --- |
| model manager tests | PyMTLF-produced bundle consumer oracle |
| loader/cache | only fixes proven necessary for digest/size/cache/safe extraction |
| config | artifact download trust/limit settings if missing；download timeout default `300 seconds` |
| `pyproject.toml`/lockfile | 加入 Ruff dev dependency與最小明確設定 |
| source/tests | 只做 Ruff要求且不改變行為的 lint remediation；禁止順帶 refactor |
| docs/API | document direct artifact GET as provisioned data path |

若 current code已滿足 oracle，PyAnLF除 Ruff tooling與必要 lint remediation外，只新增 tests/fixtures，
不做 speculative refactor。Ruff導入前後都必須通過 full PyAnLF test suite。

### 20.4 nwdaf-docs

1.更新本文件 implementation record
2. parent plan加入 Phase 1 link/status
3.記錄 decision gates與任何 approved deviation
4.各 repository commit分開記錄

---

## 21. Implementation Sequence

### Step 1: Record Baselines And Confirmed Decisions

1.重新記錄 repository commits/status
2.將 section 28的 confirmed defaults寫入 implementation config與 tests
3.記錄 artifact root、DB path與 bundle v1 limits
4.記錄 current PyAnLF loader oracle scope
5.若 current code與本文件衝突，先 replan

### Step 2: Establish Contract Examples And Oracle Inventory

1.建立 language-neutral contract examples
2.固定 field names、enums、timestamps、IDs與 error shape
3.建立 policy fixture schema/provenance
4.以 current Go code/tests證明 expected behavior
5.不實作 Python policy

### Step 3: Create PyMTLF Repository Foundation

1.初始化獨立 git repository
2.建立 src packaging、config、logging、app與 tests
3.建立 liveness/readiness
4.驗證 clean startup/shutdown與 invalid config failure

### Step 4: Implement Artifact Foundation

1.建立 content-addressed repository
2.實作 atomic publish與 immutable GET
3.建立 deterministic bundle fixture
4.補 PyAnLF consumer-oracle tests與必要安全修正
5.驗證 Go不接觸 bytes

### Step 5: Implement Generation Journal

1.建立 schema/version migration
2.實作 allocation/binding/terminal transition
3.實作 idempotency/conflict
4.實作 reopen/concurrency/failure tests
5.不從 production path配置 generation

### Step 6: Implement Reconciliation Domain Seam

1.定義 active-state request/response
2.實作 typed Go client seam in PyMTLF
3.使用 fake client覆蓋 outcomes
4.將 pending startup與 readiness連動
5.不註冊尚未有 processor semantics的 Go route

### Step 7: Add Go Neutral Backend Boundary

1.新增 config與 validation
2.新增 contract/interface/client
3.以 `httptest`驗證 HTTP behavior
4.確認 default disabled不改 startup或 live MTLF
5.搜尋並禁止 Go-side Python naming

### Step 8: Cross-process Smoke And Review

1.啟動 PyMTLF
2.驗證 live/ready
3.由 PyAnLF下載 artifact fixture
4.驗證 contract examples雙邊 encode/decode
5.驗證 shutdown/restart與 journal persistence
6.執行三個 implementation repository的 focused/full checks
7.做 ownership、security與 no-dual-owner review

### Step 9: Record Completion

1.分 repository commit
2.更新本文件 implementation record
3.更新 parent plan Phase 1 status
4.記錄未執行的 environment validation

---

## 22. Test Plan

### 22.1 PyMTLF Config And Lifecycle

1. valid minimal config
2. invalid/missing bind host、port、DB path、artifact root/base URL
3. default port `9092`
4. startup creates schema/directories
5. second startup reuses schema without data loss
6. unsupported schema version fails closed
7. readiness false during pending reconciliation
8. cancellation stops owned tasks
9. repeated startup/shutdown does not leak connection/task

### 22.2 Contract Models

1. valid examples round-trip
2. missing/blank IDs
3. timezone-less timestamps
4. invalid generation ordering
5. unknown enum
6. negative size/count/index
7. URL/digest mismatch shape
8. additive optional field compatibility
9. major contract version rejection

### 22.3 Artifact Repository/API

1. deterministic digest/key
2. duplicate publish idempotency
3. same key/different bytes conflict
4. atomic publish leaves no partial visible artifact
5. unknown key `404`
6. path traversal rejected
7. correct length/type/cache/digest headers
8. process reopen retains artifact
9. read/write failure mapped without local path leak
10. no automatic GC in Phase 1

### 22.4 Generation Journal

1. first allocation target 1/base 0
2. allocation after active generation N
3. concurrent same identity allocations serialize
4. different identities independent
5. failed transaction does not consume generation
6. committed failed apply consumes generation
7. duplicate event/same payload idempotent
8. duplicate event/different payload conflict
9. only APPLIED updates active
10. rollback old digest/new generation
11. reopen preserves last/active/pending
12. terminal job state persists

### 22.5 Reconciliation

1. target generation/digest active -> APPLIED
2. base still active -> pending
3. target generation/different digest -> conflict/not-ready
4. active generation ahead -> conflict/not-ready
5. diverged runtimes -> not-ready
6. unavailable Go/PyAnLF -> bounded retry/not-ready
7. cancellation stops retry
8. state query alone does not invent NO_MATCH terminal result

### 22.6 NWDAF

1. optional `mtlfBackend` omitted/disabled preserves current startup
2. enabled config validation
3. endpoint/origin normalization
4. client request method/path/body/content type
5. parent context cancellation與 timeout
6. response body limit
7. typed backend error preservation
8. client does not mutate IDs/generation
9. current accuracy report仍只進 Go MTLF
10. current Daisy client/callback仍工作
11. Go naming audit無 Python implementation name

### 22.7 PyAnLF Consumer Oracle

1. valid bundle download/load
2. digest/size mismatch
3. unsafe archive entries
4. missing/invalid components
5. same content reuse
6. same URL/different target generation cache separation where target contract is test-injected
7. candidate failure keeps current runtime
8. no direct control/event call to PyMTLF

### 22.8 Cross-process Smoke

1. NWDAF test client -> PyMTLF health
2. PyAnLF -> PyMTLF artifact GET
3. artifact bytes不經 Go
4. graceful PyMTLF shutdown
5. restart retains artifact/journal
6. pending reconciliation blocks readiness

Phase 1不把 health/artifact smoke描述成 complete training E2E。

---

## 23. Verification Commands

實作時依實際檔名調整，但最低要求：

### 23.1 PyMTLF

```bash
cd PyMTLF
uv sync --dev
uv run pytest -q
uv run ruff check .
```

Ruff已是 confirmed baseline。若環境或 dependency問題使 Ruff無法使用，必須依 development policy
提出 blocker/replan，不得無聲略過 lint或自行替換工具。

### 23.2 NWDAF

```bash
cd NWDAF
go test ./internal/mtlf/... ./pkg/factory ./pkg/service
go test -race ./internal/mtlf/... ./pkg/factory ./pkg/service
make test
make build
make lint
```

### 23.3 PyAnLF

```bash
cd PyAnLF
uv run pytest -q tests/test_model_manager.py tests/test_lifecycle_api.py
uv run pytest -q
uv run ruff check .
```

PyAnLF current repository只有 pytest dev dependency，沒有 Ruff、Flake8、Pylint、Mypy或 Pyright
設定。Phase 1將 Ruff加入 dev dependency與 lockfile，採最小明確設定，先修正 Ruff實際報告且可證明
不改變行為的問題。若某條規則需要 architecture refactor或 behavior change，應 narrow-ignore並留下理由，
不得為了 lint score擴張本 phase。

### 23.4 Cross-process

Cross-process command/entrypoint在實作時固定，最低需要：

1. live readiness check
2. real HTTP artifact download
3. real PyAnLF consumer load
4. restart persistence check

依 workspace policy，tests、scripts與 local services使用 elevated permission執行。若環境無法啟動
PyAnLF/PyMTLF，必須提出 blocker並保持 Phase 1未完成；不能用 FastAPI unit test取代。只有使用者依
blocker report明確核准降低或延期這項驗證後，才能更新 completion standard。

### 23.5 Repository Hygiene

每個 repository分別執行：

```bash
git diff --check
git status --short
```

不得跨 repositories建立單一 commit。

---

## 24. Observability And Security

### 24.1 Required Logs

使用 structured fields或一致 key/value wording記錄：

1. app startup/shutdown與 readiness transition
2. schema migration version
3. artifact publish key、size與 digest，不記 local temp path
4. generation allocation identity/base/target/event
5. apply-result terminal transition
6. reconciliation start/outcome/retry reason
7. backend HTTP operation、status與 correlation ID

不得記錄 model bytes、完整 observations、auth material或 stack到 client response。

### 24.2 Initial Metrics Boundary

Phase 1不要求新增 metrics server。若新增 metrics，至少需回答：

1. unresolved pending events
2. reconciliation failures
3. artifact publish/download failures
4. journal transaction conflicts

metrics不得為了「看起來完整」而擴張 lifecycle；logs與 health足以完成 Phase 1時可 deferred。

### 24.3 SSRF And Artifact Trust

1. Go驗證 ModelReady artifact origin
2. PyAnLF獨立驗證 allowed origin，不只信任 Go
3. redirect預設拒絕或限制在同一 allowed origin
4. DNS rebinding/external trust不宣稱在 loopback Phase 1完整解決
5. external deployment前必須加入 authentication、TLS/trust與 signed/safe bundle decision

---

## 25. Risks And Controls

### 25.1 Premature Dual Ownership

Risk：Go把 live report送到新 backend，同時仍由 current policy決策。

Control：config default disabled；Phase 1沒有 live accuracy route；tests斷言 current handler只進 Go path。

### 25.2 Contract Types Become Dead Parallel Models

Risk：Phase 1建立大量 types，後續 implementation另開一套。

Control：route inventory、ownership與 exact examples固定；Phase 2/3必須擴充同一 types，deviation需 replan。

### 25.3 SQLite And Filesystem Are Not One Transaction

Risk：artifact publish成功但 DB allocation失敗。

Control：publish-before-allocation；generation只由 committed journal決定；orphan保留並可觀測。

### 25.4 Current PyAnLF Contract Conflicts With Target Generation

Risk：Phase 1為了 live smoke提前切換 production apply semantics。

Control：Phase 1只做 bundle consumer oracle與 target contract models；production cutover留給 Phase 4。

### 25.5 Trusted Python/Pickle Bundle

Risk：dynamic code/pickle可執行不受信任內容。

Control：loopback trusted boundary、origin/digest/archive validation；external provider前必須 replan format/signing。

### 25.6 Artifact Disk Growth

Risk：Phase 1 conservative retention持續占用 disk。

Control：只用 fixture/人工 publish；記錄 size與 inventory；production training前完成 retention policy。

### 25.7 Optional Client Changes Startup Behavior

Risk：新增 backend config後 NWDAF無 backend就啟動失敗。

Control：section optional且 disabled by default；不做 startup hard dependency。

### 25.8 Oracle Captures Current Bug

Risk：直接 snapshot current output把未知 bug當 invariant。

Control：每個 fixture需 source/test provenance與 classification；矛盾先 decision gate，不自動接受 output。

---

## 26. Blocker And Replan Conditions

遇到以下情況必須停止實作並依 development policy提出 blocker report：

1. current PyAnLF loader無法在不切換 generation owner下驗證 target bundle
2. current model bundle缺少必要 inference semantic，需重新設計 format而非 additive metadata
3. Go current accuracy report contract與 PyAnLF實際 payload不一致
4. current Go MTLF tests無法提供 policy oracle且需要行為推測
5. SQLite無法符合 deployment filesystem或並行需求
6. artifact URL必須由 Go proxy才能讓 current PyAnLF到達
7. Go package只能透過加入 implementation-specific naming才能接 client
8. Phase 1若不註冊 future routes就無法做任何必要 contract verification
9. standard generated model與 local TS/OpenAPI對 identity或 URL semantic有影響本 phase的衝突
10. safe extraction/digest修正會改變 current production bundle compatibility
11. tests需要未提供的 model fixture、dependency、port或 network authority
12. implementation需要修改 Phase 2/3/4 ownership才能保持系統可運作

Blocker report必須包含原假設、實際證據、可行選項、建議、tradeoff與是否需先更新 parent/phase plan。

---

## 27. Completion Criteria

Phase 1只有在以下全部成立時才能標記 complete：

1. `PyMTLF/`是獨立、clean、可安裝與可運行的 repository
2. package/runtime/config使用 `py_mtlf`與 approved defaults，且 `.gitignore`排除 `data/`
3. PyMTLF config、health、startup、shutdown與 error mapping有 tests
4. Go具有 neutral `MtlfBackend` config/contract/client boundary
5. Go source/config/logs/routes/tests不含 Python implementation naming
6. backend disabled時 current NWDAF startup/live MTLF behavior不變
7. contract v1 families、examples、version與 error semantics已固定
8. contract-only routes未以 dummy success暴露
9. SQLite journal可保存 last/active/pending generation、event與 terminal state
10. generation allocation monotonic、non-reuse且 rollback使用新 generation
11. restart/reopen不遺失 generation/provision state
12. reconciliation engine覆蓋 consistent、pending、conflict、diverged與 unavailable
13. unresolved pending state會阻擋 PyMTLF readiness
14. artifact repository可 atomic、content-addressed、immutable publish與 GET
15. artifact URL不含 mutable `latest`，generation不寫死在 bytes
16. PyAnLF可直接下載並載入 valid fixture
17. corrupt、oversized、unsafe或 incompatible bundle不替換 current model
18. Go未下載、保存或代理 model artifact bytes
19. compatibility oracle fixtures有 provenance、classification與 current Go baseline proof
20.沒有 live accuracy dual routing、second policy owner或 production training
21. PyMTLF與 PyAnLF都已配置 Ruff，`uv run ruff check .`通過
22. PyMTLF/NWDAF/PyAnLF focused與 full repository checks依 section 23完成
23. cross-process health/artifact smoke完成
24.各 repository diff/commit分離且無 unrelated changes
25.本文件與 parent plan更新 implementation commits、verification與 remaining gaps

---

## 28. Confirmed Decisions Before Implementation

以下十項已確認：

1. Python runtime/template/ports
   - PyMTLF使用 Python `>=3.12`、FastAPI、Pydantic v2、Uvicorn、`uv`、hatchling與 package
     `py_mtlf`
   - PyMTLF default為 `127.0.0.1:9092`
   - PyAnLF default由 `9090`移到 `127.0.0.1:9093`；`9090`不再作為 PyAnLF fallback
2. private API template/version
   - PyMTLF route prefix為 `/internal/v1`，payload帶 `contract_version: "1.0"`
   - Go inbound route prefix為 `/mtlf-backend/v1`
   - PyMTLF與 PyAnLF統一 FastAPI/Pydantic、JSON、timestamp、error與 test conventions，但不共用
     runtime package，也不在 Phase 1破壞性搬移 current PyAnLF routes
3. Phase 1 authentication
   - loopback trusted boundary，no application auth/native TLS；external deployment必須 replan trust
4. Go config key與 client path
   - `mtlfBackend`、`internal/mtlf/backend/`，所有 Go naming不含 Python implementation name
5. PyMTLF state/artifact defaults
   - SQLite default為 `data/mtlf-state.sqlite3`，artifact使用 content-addressed root
   - repository `.gitignore`必須排除整個 `data/`
6. artifact identity/integrity
   - SHA-256 lowercase hex為 artifact key，ModelReady digest與 size是 end-to-end authoritative metadata
   - immutable GET提供 exact length、content type、digest/ETag與 immutable cache semantics
7. artifact resource/download limits
   - compressed `256 MiB`、extracted total `1 GiB`、single file `512 MiB`、最多 `32` entries
   - artifact download timeout為 `300 seconds`；control API timeout不隨之放寬
   - redirects預設拒絕；預設值可依 fixture/實測調整，但 upper bounds不可移除
8. automatic artifact GC
   - Phase 1 disabled，只提供 protected-delete primitive與 inventory metadata
9. oracle baseline與 PyAnLF hardening scope
   - policy oracle以 implementation kickoff時的 current NWDAF commit為 baseline，每個 fixture記錄更精確
     historical/current provenance
   - PyAnLF只做 consumer oracle證明必要的 digest、size、origin、cache、timeout與 safe-extraction changes
10. Python lint
    - 新 PyMTLF與 current PyAnLF都在 Phase 1使用 Ruff
    - PyAnLF只接受必要、行為不變的 lint remediation；需要 refactor的規則採有理由的 narrow ignore

這些 choices只決定 Phase 1 implementation details，不重新打開 parent plan已定的 ownership boundary。

---

## 29. Implementation Record Template

Phase 1實作後在此新增：

### 29.1 Confirmed Decisions

1. exact runtime/package/port
2. exact paths/auth/config
3. exact DB schema version與 artifact limits
4. approved deviations

### 29.2 Repository Commits

1. `PyMTLF@...`
2. `NWDAF@...`
3. `PyAnLF@...` if changed
4. `nwdaf-docs@...`

### 29.3 Verification Results

1. commands
2. pass/fail counts
3. unit/module/build/lint/cross-process level
4. skipped checks與原因

### 29.4 Remaining Gaps

1. Phase 2 data service work
2. Phase 3 policy/training/isolation work
3. Phase 4 production cutover work
4. environment-only or accepted risks
