# Slice 0 Baseline Audit and Contract Freeze Detailed Plan

日期：2026-08-18

狀態：Completed；唯讀baseline audit與contract freeze已完成，並已作為Slice 1–7的
implementation baseline。2026-08-21依後續production evidence完成狀態一致性回填

上層計畫：

- [Hierarchical NWDAF Federated Learning Implementation Plan](./Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)

相關文件：

- [Hierarchical Federated Learning Proposal Draft](../../proposals/nwdaf/hierarchical-federated-learning/proposal_draft.md)
- [Phase 0 Release 18 Contract Foundation Detailed Plan](../federated-learning/Phase%200%20Release%2018%20Contract%20Foundation%20Detailed%20Plan.md)
- [Distributed NWDAF Federated Learning Implementation Plan](../federated-learning/Distributed%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)
- [Phase 3 And 4 Federated Training Execution Detailed Plan](../federated-learning/Phase%203%20And%204%20Federated%20Training%20Execution%20Detailed%20Plan.md)
- [NWDAF Development Policy](../../development_policy.md)

---

## 1. 目的與完成邊界

Slice 0 先固定 hierarchical FL 第一版跨 repository 共用的 contract，避免後續在
`NWDAF/`、`PyMTLF/` 與 integration resources 各自推測 hierarchy semantics。

本 Slice 必須完成：

1. 稽核既有 distributed FL 可重用的標準 SBI、process state、artifact 與 trigger
   boundary；
2. 明確區分 Release 18 標準欄位與 same-vendor hierarchy metadata；
3. 固定 assignment／preparation-result bundle 的 versioned schema 方向；
4. 固定第一版 strategy、static topology、runtime capability、trigger、single-active、
   terminal failure 與 restart contracts；
5. 提供 Slice 1–7 可直接引用的 migration map、repository ownership 與 acceptance
   checklist。

本 Slice 是 documentation-only contract freeze：

- 不修改 `NWDAF/`、`PyMTLF/`、`PyAnLF/` 或 `nwdaf-resources/` production code；
- 不新增 placeholder endpoint 或未被 runtime owner 使用的 schema；
- 不執行 HFL experiment；
- 不把 proposal 中尚未採用的 partial admission、automatic re-planning 或其他 Daisy
  strategies 帶入第一版。

完成本文件並通過 review 後，production implementation 才從 Slice 1 開始。

---

## 2. Baseline 與證據邊界

### 2.1 Repository baseline

| Repository | Branch | Baseline revision | Slice 0 disposition |
| --- | --- | --- | --- |
| `NWDAF/` | `feat/r18-federated-learning` | `c53f05804c6533ec4c5fa7e230e90048fb219162` | read-only audit |
| `PyMTLF/` | `feat/r18-federated-learning` | `7e8ab7f23bf5d6398eb1cd5f053dd8bda9439a87` | read-only audit |
| `PyAnLF/` | `feat/r18-federated-learning` | `08798f15c3693027e00bc60dd53f74ebaa26c3a1` | read-only audit |
| `nwdaf-resources/` | `feat/r18-federated-learning` | `8a7619ec9c745f71a8fea42134cefd550ad2c180` | read-only audit |
| `nwdaf-docs/` | `main` | `2e0cf51ab6adcba4eb28ceabc479746798a69e5f` | 本文件 owner |

稽核開始時各 repository worktree 均為 clean。

後續 production slice 若既有 distributed FL baseline 尚未合併，依上層計畫從各
repository 的 `feat/r18-federated-learning` 建立
`feat/r18-hierarchical-federated-learning`；文件仍直接更新 `nwdaf-docs/main`。

### 2.2 Evidence order

本 Slice 依序使用：

1. 以上 baseline 的實作與 tests；
2. 上層計畫與 proposal 中已確認的 decisions；
3. local Release 18 TS 29.520／TS 23.288 與 OpenAPI；
4. specifically applicable free5GC development guidance；
5. local generated OpenAPI 與 free5GC reference implementation。

Release 18 OpenAPI／TS 決定標準 SBI path、method、body、response、callback 與
conditional field semantics。Hierarchy assignment、cross-tier mapping、strategy、static
topology 與 result bundle 都是 project-private contract。

本次特別核對：

- `TS29520_Nnwdaf_MLModelTraining.yaml` 的 `NwdafMLModelTrainNotif`；
- TS 29.520 §5.5.6.2.8 的 notification conditional rules；
- TS 23.288 §6.2C.2.1 steps 7–10 的 FL preparation／join decision；
- TS 23.288 §6.2F 的 Training Subscribe／Notify lifecycle。

Release 19 TS 28.105 只提供 management-trigger semantics 參考；本專案的 private API
不宣稱是 TS 28.105 或 TS 28.532 compliant implementation。

---

## 3. 現況稽核

### 3.1 Go NWDAF standard Training boundary

既有實作已具備完整 distributed FL transport skeleton：

| Concern | Current owner／evidence | 結論 |
| --- | --- | --- |
| Public Training SBI | `internal/sbi/api_ml_model_training.go` | 已有標準 POST／PUT／PATCH／DELETE subscription routes |
| Training callback | `internal/sbi/api_ml_model_callback.go` | 已有 per-route callback correlation |
| Public／backend processing | `internal/sbi/processor/ml_model_training.go` | 已有 raw-body validation、route mapping、response forwarding 與 cleanup |
| Peer Training consumer | `internal/sbi/consumer/ml_model_peer_service.go` | 已有對 remote NWDAF 的標準 create／replace／patch／delete |
| Go–PyMTLF private gateway | `internal/mtlf/api_ml_model_gateway.go` | 已有 internal subscription CRUD 與 notification ingress |
| Route state | `internal/context/ml_model_training_routes.go` | 已保存 standard resource、peer、backend、notification 與 expected-round correlation |
| NRF discovery gateway | `internal/backend/nrf_discovery.go`, `internal/mtlf/api_nf_discovery.go` | 已支援 exact NF instance、service name 與 ML analytics capability query |
| Release 18 FL capability | `internal/compat/nrf/models.go` | 已支援 `FL_SERVER`、`FL_CLIENT`、`FL_SERVER_AND_CLIENT` |
| Backend generation reset | `internal/backend/availability_monitor.go`, `internal/sbi/processor/ml_model_backend_reset.go` | PyMTLF generation 改變時清除相依 routes，對 consumer-owned peer resource做一次 best-effort DELETE，不建立 recovery state |

Go route state目前不含：

- `planId`；
- Branch upper／lower process mapping；
- assigned／prepared participant snapshot；
- hierarchy bundle schema；
- strategy 或 topology admission state。

這些內容不應整體移入 Go。Go 的責任維持在 standardized SBI transport、NRF access、
backend generation ownership 與最小 routing／correlation state；hierarchy business state
由 PyMTLF 擁有。

### 3.2 Go capability and service-registration baseline

`pkg/factory/config.go` 已驗證 `flCapabilityType`，且只有具備 `FL_CLIENT` 或
`FL_SERVER_AND_CLIENT` 的 NWDAF 才需要暴露 `nnwdaf-mlmodeltraining` service 供其他
FL Server 建立 subscription。

因此第一版不新增 `ROOT`、`BRANCH`、`LEAF` NF profile 欄位：

- Root eligibility：`FL_SERVER` 或 `FL_SERVER_AND_CLIENT`；
- Branch eligibility：`FL_SERVER_AND_CLIENT`；
- Leaf eligibility：`FL_CLIENT` 或 `FL_SERVER_AND_CLIENT`；
- 實際 topology position 只由有效 assignment 與 active `planId` 決定。

目前 Go containing-NWDAF context response 只回傳 `nfInstanceId`、public API root 與
internal API root，尚未讓 PyMTLF 驗證 Go 所廣告的 per-analytics FL capability。Slice 2
只擴充這個既有 private context，不建立第二套 capability config或新的 public SBI。

### 3.3 PyMTLF runtime and process baseline

#### Runtime gating

`src/py_mtlf/config.py` 的 `runtime.mode` 目前只有：

- `local`；
- `fl_server`；
- `fl_client`。

Settings validator 明確禁止同時配置 `federated_learning.server` 與
`federated_learning.client`。`src/py_mtlf/api/ml_model_training.py` 也以互斥 mode
分別 gate subscription CRUD 與 notification ingress。

雖然 `src/py_mtlf/app.py` 會建立 `FLClientService` 與 `FLServerOrchestrator` objects，
其 routes、lifecycle 與業務入口仍受互斥 mode 限制。因此 Branch 所需的同一 runtime
Server + Client behavior 尚不可用。

#### FL Server

`src/py_mtlf/core/fl_server.py` 已具備：

- degradation policy intent ingress；
- participant discovery／assignment；
- asynchronous preparation；
- synchronous round dispatch／waiting；
- result validation與 aggregation；
- final validation／publication／cutover；
- terminal failure 與 participant cleanup；
- notification correlation 與 duplicate-content checks。

`FLServerSettings.max_active_processes` 預設為 `1`，但 schema 允許 `1..32`；它不是
HFL instance-wide single-active contract。現有 `_processes`、`_correlations` 與 executor
也只屬於 FL Server engine，無法約束同一 Branch 的 upper Client 與 lower Server
resources 必須屬於同一 experiment。

現有 `AccuracyPolicy` 的 `_in_flight` 只做 per-family suppression；FL process terminal後
`complete_retrain()` 會移除該標記，後續 degradation reports可以再次產生 intent。這不等於
已確認的「失敗後等待 operator明確重新啟動」語意，因此 HFL coordinator另需
process-memory failure latch。

#### FL Client

`src/py_mtlf/core/fl_client.py` 已具備：

- standard Training subscription resource CRUD；
- asynchronous preparation；
- local round與 final validation jobs；
- bounded worker／callback outbox；
- duplicate callback protection；
- terminal failure states。

Resources 保存在 process memory。`restore_after_restart()` 與
`discard_restored_routes()` 雖存在，但目前沒有 active call site；不能把這些 helpers
視為可用的 restart recovery protocol。

現有 Go／PyMTLF notification validator 把 `statusReport` 單獨存在視為有效，FL Client
preparation success也只送 `statusReport`。但 TS 29.520 §5.5.6.2.8 NOTE 1 要求至少提供
`delayEventNotif`、`mLModelInfos` 或 `termTrainReq` 其中之一；`statusReport` 不在這個最低
集合中。這是既有 distributed FL baseline 的標準條件差距，HFL 不得延續。

這不是單一 implementation 漏項，而是既有計畫的 contract regression。Phase 0 contract
foundation 已記錄 `statusReport` 不能獨自滿足上述條件；後續 distributed FL 主計畫與
Phase 3／4 execution plan 卻改為接受 `statusReport`-only preparation callback，並明確要求
validator 不得拒絕它。現有 sender／validator／receiver 是依後期計畫實作，因此 Slice 4
必須把文件與實作視為同一項 contract correction，而不是只修 HFL-specific code。

固定 `statusReport.trainInDataInfo.samplRatio: 100` 只是既有實作用來表示 preparation
completed 的 marker；`samplRatio` 表達的是 sampling ratio，不是 preparation outcome、精確
sample count 或 model result。修正後不得再把此值當作 completed latch。`statusReport` 可
保留為 optional supplemental status，但成功與否由符合下述 notification profile 的
`mLModelInfos` artifact、active preparation stage 與 correlation 共同判定。

### 3.4 PyMTLF algorithm baseline

`src/py_mtlf/core/federated_trainer.py` 目前：

- local objective 只有 Huber task loss；
- optimizer 為 Adam；
- 沒有保存本輪 global-reference parameters 供 proximal term 使用；
- server aggregation 已是依 `training_sample_count` 計算的 sample-weighted FedAvg；
- tensor shape、dtype 與 non-floating state 已有 compatibility validation。

因此第一版 FedProx 的實作差距只在 local objective 與 typed configuration propagation；
Server aggregation 可以重用既有 sample-weighted primitive，但必須新增 Branch effective
sample count propagation 與至少兩個 Branch updates 的 upper-tier acceptance test。

### 3.5 PyMTLF bundle and artifact baseline

`src/py_mtlf/core/fl_workspace.py` 與 `src/py_mtlf/core/fl_artifacts.py` 已提供：

- fixed bundle file set：`config.json`、`model.py`、`model.npy`、`scaler.pkl`；
- archive size、entry、path、origin 與 digest validation；
- `ROUND_LOCAL`、`ROUND_GLOBAL`、`FINAL_MODEL` typed roles；
- publisher-owned temporary artifact URL；
- URL path內的 process、participant、round、role 與 archive digest；
- workspace TTL cleanup。

現有 gaps：

- 沒有 hierarchy assignment／preparation-result artifact roles；
- 沒有 `planId`、publisher、intended recipient 或 assigned subtree metadata；
- 沒有 normalized strategy／admission contract；
- 沒有 prepared／failed／timed-out result schema；
- 沒有 Branch download／process／republish provenance；
- workspace cleanup 只在 open 時清除過期目錄，沒有明確的 per-plan release primitive；
- active process、mapping 與 participant snapshot 不持久化。

Hierarchy metadata 應加入既有 `config.json`，不得增加只為控制 plane 使用的新 archive
entry，也不得破壞有效 model artifact 的四檔案 contract。

### 3.6 PyMTLF trigger and persistence baseline

目前唯一自動入口是 model monitor degradation：

```text
ML Model Monitor notification
-> AccuracyPolicy creates RetrainIntent
-> ml_model_monitor dispatch
-> FLServerOrchestrator.accept_policy_intents()
```

目前沒有直接由 PyMTLF 擁有的 manual training-request API。

Current FL server／client process state、callback correlation、policy in-flight set 與
temporary workspace ownership主要在 memory。Completed model catalog／publication state
有獨立 durable semantics，但它不是 active FL experiment recovery state。

### 3.7 PyAnLF boundary

`PyAnLF/` 不接收 `Nnwdaf_MLModelTraining`、不執行 local FL training，也不解析 hierarchy
assignment／result bundle。它目前透過標準 ML Model Provision 取得完成模型 URL，驗證並
載入既有四檔案 bundle，再切換 analytics runtime。

第一版 HFL 不改變 final model Provision contract，因此 PyAnLF disposition 是：

- Slice 0–7 無 production change；
- Slice 8 只執行既有 final model download／activation regression；
- 只有最終 bundle 或 Provision contract 真正改變時，才建立 PyAnLF HFL branch。

### 3.8 nwdaf-resources boundary

`nwdaf-resources/` 不擁有 runtime contract。Slice 1–7 只有在需要 cross-repository fixture
或可重用 validation sample 時才修改；Slice 8 再加入 topology deployment profiles 與
E2E scripts。

---

## 4. Reuse and gap matrix

| Capability | Reuse as-is | Extend | New owner |
| --- | --- | --- | --- |
| Release 18 Training SBI | Go handler／processor／consumer skeleton | hierarchy-aware route tests only where required | `NWDAF/` |
| NRF exact-instance discovery | existing Go private gateway | configured Branch／Leaf eligibility queries | `NWDAF/` + `PyMTLF/` |
| Per-tier standard resource | existing subscription／callback lifecycle | explicit upper／lower mapping in PyMTLF | `PyMTLF/` |
| Bundle transport | existing PyMTLF direct serving／download | hierarchy roles, metadata and per-plan cleanup | `PyMTLF/` |
| Local training | current deterministic trainer | FedProx proximal objective | `PyMTLF/` |
| Aggregation | current sample-weighted tensor aggregation | Branch effective weight and upper tier | `PyMTLF/` |
| Degradation trigger | current AccuracyPolicy intent | shared HFL request coordinator／guard | `PyMTLF/` |
| Manual trigger | none | config-gated async private resource | `PyMTLF/` |
| Restart handling | Go backend generation reset; new PyMTLF process memory | HFL stale interaction tests and per-plan cleanup | `NWDAF/` + `PyMTLF/` |
| Final model activation | existing Provision path | regression only | `PyAnLF/` |

---

## 5. Contract freeze

### 5.1 Identity model

第一版使用四種不同 identity，不得互相代用：

| Identity | Owner | Lifetime／meaning |
| --- | --- | --- |
| `requestId` | trigger producer／private API resource | 一次 initiation request 的 idempotency identity |
| `planId` | Root PyMTLF | 一次完整 topology attempt；retry 必須建立新 UUIDv4 |
| `mlCorreId` | 每個標準 FL Server process | 只屬於一個 Root–Branch 或 Branch–Leaf tier process |
| subscription／notification correlation | Go／PyMTLF per standard resource | 只在所屬 tier resource 內有效 |

Rules：

- `planId` 是 vendor hierarchy identity，不取代標準 `mlCorreId`；
- Root 的 upper process 與每個 Branch 的 lower process 使用不同 `mlCorreId`；
- Branch business state以 `planId` 明確連結一個 upper Client resource、lower Server
  process、assigned Leaves 與 temporary artifacts；
- 不得以相同 `roundInd`、subscription ID、URL path 或 NF identity 推測 cross-tier
  relationship；
- completed／failed plan 的 identity 不得在同一 process lifetime 被重新啟用；
- operator 重新開始實驗時，即使 topology 與 model family相同，也產生新的
  `requestId`／`planId`。

### 5.2 Capability-oriented runtime migration

Target configuration 不新增永久 hierarchy role。既有 mode migration 固定為：

```yaml
runtime:
  mode: federated

federated_learning:
  server:
    # FL Server engine settings
  client:
    # FL Client engine settings
```

Rules：

- `runtime.mode` 第一版收斂為 `local` 或 `federated`；
- `local` 維持目前 local-training path，且不得配置 FL Server／Client；
- `federated` 至少配置 `server` 或 `client` 其中一項；
- `server` section 存在即啟用 FL Server engine；
- `client` section 存在即啟用 FL Client engine；
- 兩者可同時存在，這是 Branch deployment capability，不是 `branch` role config；
- 不接受 `root`、`branch`、`leaf`、`fl_branch` 等 runtime mode；
- API route與 lifecycle gate 依 enabled engine，不再依 `mode == fl_server` 或
  `mode == fl_client`；
- 純 Server／純 Client profiles 必須可由舊設定 deterministic migration，且保留既有
  distributed FL regression behavior。

Go containing-NWDAF private context 在 Slice 2 增加由實際 NRF profile config投影出的
`mlAnalyticsCapabilities`：

```json
{
  "nfInstanceId": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb",
  "apiRoot": "http://nwdaf.example",
  "internalApiRoot": "http://nwdaf.internal",
  "mlAnalyticsCapabilities": [
    {
      "mlAnalyticsIds": ["UE_COMMUNICATION"],
      "flCapabilityType": "FL_SERVER_AND_CLIENT"
    }
  ]
}
```

Slice 7 後此 private response 另包含每次 Go process startup 產生的
`processInstanceId`，供 PyMTLF 偵測 containing Go generation 改變。它是後續 lifecycle
extension，不改變本 Slice 凍結的 capability contract，也不代表 hierarchy role。

Projection只包含 PyMTLF engine consistency所需的 standard capability fields；entries與
analytics IDs canonical sort並去重。Go仍以自己的 validated `nwdafInfo.mlAnalyticsList`
作為唯一 advertisement source，PyMTLF不得再配置第二份 advertised capability。

PyMTLF 在進入 ready state前必須確認：

- enabled Server engine 對應 `FL_SERVER` 或 `FL_SERVER_AND_CLIENT`；
- enabled Client engine 對應 `FL_CLIENT` 或 `FL_SERVER_AND_CLIENT`；
- HFL Branch 所屬 analytics capability 明確為 `FL_SERVER_AND_CLIENT`；
- Go 廣告某 engine capability但 PyMTLF 未啟用對應 engine，或 PyMTLF 啟用未被 Go
  廣告的 engine，均不得進入 ready state。

這項 cross-process validation 不在 FastAPI lifespan中做一次性 blocking fetch。PyMTLF
process可先啟動，但 dynamic readiness在 containing Go internal listener尚不可用、
capability projection malformed或 capability mismatch時維持 `503 not_ready`；後續 probe
可重新驗證。Go目前先完成 NRF registration與 owned listener startup，再啟動 backend
availability monitor，因此不形成 Go 等 PyMTLF ready、PyMTLF 又等 Go listener的啟動
循環。

Assignment 仍要在每次 attempt 透過 NRF 驗證 target 的 current profile；self-capability
readiness check 不取代 peer discovery。

### 5.3 Standard field and vendor metadata ownership

| Information | Carrier | Ownership |
| --- | --- | --- |
| Analytics ID／filter／target | standard Training request | Release 18 SBI |
| training requirements／data availability | standard Training request | Release 18 SBI |
| model interoperability | standard Training request | Release 18 SBI |
| preparation flag／round indicator | standard Training request | Release 18 SBI |
| `mlCorreId`／notification correlation | standard request／notification | Release 18 SBI |
| notification URI／resource location | HTTP／standard resource | Release 18 SBI |
| model download URL | `mLModelUrl` | standard field carrying vendor bundle URL |
| `planId`／intended recipient | bundle `hierarchy_metadata` | same-vendor contract |
| assigned subtree／admission | bundle `hierarchy_metadata` | same-vendor contract |
| normalized strategy | bundle `hierarchy_metadata` | same-vendor contract |
| prepared／failed／timed-out Leaves | result bundle | same-vendor contract |
| upper／lower process mapping | Branch memory state | private runtime state |
| static topology template | Root PyMTLF config file | deployment config |

Bundle 不重複或覆寫標準 Analytics ID、training requirement、`mlCorreId`、notification URI
或 round semantics。若標準 request 與 bundle 中可驗證的 model contract 發生衝突，接收者
必須拒絕，不得選擇其中一方繼續。

### 5.4 Hierarchy bundle envelope

沿用既有四檔案 model bundle。`config.json` 擴充 typed hierarchy projection：

```json
{
  "bundle_schema_version": "1.0",
  "file_digests": {
    "model.py": "<sha256>",
    "model.npy": "<sha256>",
    "scaler.pkl": "<sha256>"
  },
  "artifact_role": "HIERARCHY_ASSIGNMENT",
  "hierarchy_metadata": {
    "contract_version": "1.0",
    "message_type": "BRANCH_ASSIGNMENT",
    "plan_id": "11111111-1111-4111-8111-111111111111",
    "publisher_nf_instance_id": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
    "intended_recipient_nf_instance_id": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb"
  }
}
```

First-version artifact roles：

- `HIERARCHY_ASSIGNMENT`；
- `HIERARCHY_PREPARATION_RESULT`；
- 既有 `ROUND_LOCAL`、`ROUND_GLOBAL`、`FINAL_MODEL`。

Common validation：

- typed hierarchy projection內的 unknown `artifact_role`、`message_type`、field 或
  incompatible version fail closed；既有 model／inference manifest fields不受影響；
- identity fields 是 normalized UUID；`plan_id` 必須是 UUIDv4；
- publisher 必須等於 current standard resource所預期的 peer NWDAF，且 URL origin必須通過
  allowed-origin／resolved-target checks；第一版沒有 bundle signature，這是 logical peer
  binding，不宣稱 cryptographic publisher attestation；
- intended recipient 必須等於 containing NWDAF；
- archive URL origin、size、entry set、component digest、model／preprocessing contract 與
  weights digest 沿用既有 validation；
- hierarchy artifact URL的 final path segment必須是 lowercase SHA-256；downloader計算完整
  archive digest並要求與 URL digest及 `X-Artifact-SHA256` response header一致；現有
  `FLWorkspace.download()` 只計算 local key而未做這項 transport binding，Slice 1補齊；
- list ordering 使用 normalized NF instance ID lexical order；不得重複；
- bundle 下載成功不等於 participant admission；接收者仍執行 NRF capability、model、
  dataset 與 timing checks。

第一版沿用現有 FL workspace coordinate：hierarchy artifacts以 `planId` 作 `process_id`、
counterparty NF instance ID作 `participant_id`、preparation stage固定使用 storage coordinate
`round_indicator = 0`，並以不同 `artifact_role` 隔離 assignment／result。這個 URL path中的
`0` 只是 private storage coordinate，不是標準 `roundInd`，也不得寫入 preparation
notification的 standard `roundInd`。

Publisher retention rules：

- assignment／result artifacts至少保留到所屬 plan terminal cleanup開始；
- callback收到 `204 No Content` 不代表 parent已完成非同步下載，publisher不得因此立即
  刪除 result bundle；
- normal flow由 subscription deletion／plan terminal cleanup釋放 per-plan artifacts；
- Slice 1 初版以 workspace TTL處理 crash leftovers；此 cleanup mechanism已由Slice 7取代為
  startup在admission前無條件清空每個PyMTLF獨占的FL scratch workspace，不建立跨restart
  artifact reconciliation；
- content-addressed URL在保留期間 immutable，同一 URL不得換內容。

### 5.5 Branch assignment schema

Root 對每個 Branch 發布一份 recipient-specific bundle：

```json
{
  "artifact_role": "HIERARCHY_ASSIGNMENT",
  "hierarchy_metadata": {
    "contract_version": "1.0",
    "message_type": "BRANCH_ASSIGNMENT",
    "plan_id": "11111111-1111-4111-8111-111111111111",
    "publisher_nf_instance_id": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
    "intended_recipient_nf_instance_id": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb",
    "assigned_leaf_nf_instance_ids": [
      "cccccccc-cccc-4ccc-8ccc-cccccccccccc",
      "dddddddd-dddd-4ddd-8ddd-dddddddddddd"
    ],
    "admission": {
      "mode": "complete_required"
    },
    "strategy": {
      "algorithm": {
        "name": "fedprox",
        "proximal_mu": 0.01
      },
      "participant_selection": "all",
      "waiting_policy": "all",
      "aggregation": "sample_weighted"
    }
  }
}
```

Rules：

- assigned Leaf list 至少一項；
- Root、recipient Branch 與 assigned Leaves 必須互不相同；
- bundle 只包含該 Branch subtree，不洩漏其他 Branch assignment；
- Branch 先下載、驗證與解析此 bundle，再經 NRF exact-instance query 驗證 Leaves；
- Branch 不得自行新增、替換或移除 assigned Leaf；任何不合格 Leaf 進入 result failure
  list，交由 Root依 `complete_required` reject attempt。

### 5.6 Leaf assignment schema and republishing

Branch 對每個 Leaf 重新發布 recipient-specific bundle：

```json
{
  "artifact_role": "HIERARCHY_ASSIGNMENT",
  "hierarchy_metadata": {
    "contract_version": "1.0",
    "message_type": "LEAF_ASSIGNMENT",
    "plan_id": "11111111-1111-4111-8111-111111111111",
    "publisher_nf_instance_id": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb",
    "intended_recipient_nf_instance_id": "cccccccc-cccc-4ccc-8ccc-cccccccccccc",
    "parent_branch_nf_instance_id": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb",
    "strategy": {
      "algorithm": {
        "name": "fedprox",
        "proximal_mu": 0.01
      },
      "participant_selection": "all",
      "waiting_policy": "all",
      "aggregation": "sample_weighted"
    }
  }
}
```

Rules：

- publisher 與 `parent_branch_nf_instance_id` 必須相同；
- intended recipient 必須是 Root assignment 中該 Branch 的 assigned Leaf；
- Leaf 不取得其他 Leaves 或完整 topology；
- model／preprocessing contract與 input weights 必須承接 Root bundle；
- URL 必須由 Branch PyMTLF serving；直接把 Root URL 傳給 Leaf 是 contract violation；
- republish 是 download／validate／process／package／serve，不是 transparent proxy。

### 5.7 Preparation result schema

Branch 在 lower-tier preparation 完成後發布：

```json
{
  "artifact_role": "HIERARCHY_PREPARATION_RESULT",
  "hierarchy_metadata": {
    "contract_version": "1.0",
    "message_type": "PREPARATION_RESULT",
    "plan_id": "11111111-1111-4111-8111-111111111111",
    "publisher_nf_instance_id": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb",
    "intended_recipient_nf_instance_id": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
    "outcome": "FAILED",
    "assigned_client_nf_instance_ids": [
      "cccccccc-cccc-4ccc-8ccc-cccccccccccc",
      "dddddddd-dddd-4ddd-8ddd-dddddddddddd"
    ],
    "prepared_clients": [
      {
        "nf_instance_id": "cccccccc-cccc-4ccc-8ccc-cccccccccccc"
      }
    ],
    "failed_clients": [
      {
        "nf_instance_id": "dddddddd-dddd-4ddd-8ddd-dddddddddddd",
        "cause": "REQUIREMENTS_NOT_MET"
      }
    ],
    "timed_out_client_nf_instance_ids": []
  }
}
```

First-version causes：

- `DISCOVERY_FAILED`；
- `CAPABILITY_MISMATCH`；
- `INVALID_ASSIGNMENT`；
- `INVALID_BUNDLE`；
- `REQUIREMENTS_NOT_MET`；
- `NOT_AVAILABLE_ML_TRAIN`；
- `INTERNAL_ERROR`。

Result invariants：

- prepared、failed 與 timed-out identities互斥；
- 三者 union 必須剛好等於 assigned client set；
- 每個 assigned client 只出現一次；
- `READY` 只允許全部 assigned clients prepared；
- 其他組合的 `outcome` 必須為 `FAILED`；
- failure cause 是 bounded machine-readable enum，不放 exception stack 或任意 log text；
- result bundle 仍含有效 model artifact，且 model／preprocessing／input weights contract
  必須與 Branch 收到的 assignment bundle一致；
- Branch 無法產生有效 result bundle 時，以 standard termination notification 結束；Root
  將該 Branch 視為 failed，不猜測 subordinate outcome。

Branch 透過 upper-tier notification 的 `mLModelInfos.mLFileAddr.mLModelUrl` 提供 result
bundle URL。Root 只接受 current upper subscription、matching notification correlation、
matching `planId`、expected Branch publisher 與 intended Root 的 result。

#### Preparation notification profile

為符合 TS 29.520 §5.5.6.2.8 NOTE 1，第一版固定：

- successful Leaf preparation notification包含 `mLModelInfos`，其 URL指回該 Leaf已下載
  並驗證的 Branch-published assignment bundle；可另外包含 `statusReport`；
- successful Branch preparation notification包含 `mLModelInfos`，其 URL指向
  Branch-published preparation-result bundle；可另外包含 `statusReport`；
- failed Branch若仍能形成完整 result bundle，notification同時包含 `mLModelInfos` 與
  standard `termTrainReq: NOT_AVAILABLE_ML_TRAIN`；TS允許這兩者共存；
- participant在產生有效 result bundle前失敗時，只送 standard `termTrainReq`；
- `delayEventNotif` 不得與 `mLModelInfos` 或 `termTrainReq` 共存；
- preparation notification不帶 standard `roundInd`。

TS conditional rule結合 Stage 2 join decision可推導出：成功且不延後、不終止的
preparation outcome應使用 `mLModelInfos`。因此成功時回報已下載並驗證的 input assignment
model URL可作為成功 acknowledgement；它保持 URL指向有效 model artifact，也不宣稱產生
新 trained model。真正的擴充語意是 bundle內的 hierarchy metadata。接收端依 active
preparation stage解讀 assignment acknowledgement或 Branch result。Slice 4需同步修正
Go／PyMTLF wire validation、sender與 stage-aware receiver tests；不能只改 HFL orchestrator
而保留 status-only validator。對同時含 result bundle與
`termTrainReq` 的 Branch failure，receiver必須先驗證並記錄 result partition，再將 process
標記 terminal；現有 `FLServerOrchestrator.receive_notification()` 的 mutually exclusive
`elif` handling不能直接沿用。

上述修正也必須覆蓋 non-hierarchical distributed FL regression：既有 preparation sender
改用相同的 `mLModelInfos` 成功 contract，receiver 不要求 `statusReport`，而 subscription
resource、round notification 與 final-result behavior 維持相容。歷史 Phase 3／4 的
`statusReport`-only 決策由本計畫 supersede，不作為相容性要求保留。

### 5.8 First-version strategy contract

Root main config：

```yaml
federated_learning:
  strategy:
    algorithm:
      name: fedprox
      proximal_mu: 0.01
    participant_selection: all
    waiting_policy: all
    aggregation: sample_weighted
```

Typed rules：

- `algorithm` 是 discriminated settings block；
- 第一版 `algorithm.name` 唯一合法值為 `fedprox`；
- `proximal_mu` 必須 finite 且 `> 0`；
- `participant_selection` 唯一合法值為 `all`；
- `waiting_policy` 唯一合法值為 `all`；
- `aggregation` 唯一合法值為 `sample_weighted`；
- top-level `proximal_mu`、generic `parameters` dictionary、Python module／class name、
  `fedavg`、`fixed_count`、`minimum_results` 與 unknown fields全部在 config validation
  階段拒絕；
- Root 將 normalized strategy 放入 Branch assignment；Branch 原樣驗證並放入 Leaf
  assignment；不得由各 tier 以 local default重新推測；
- 第一版不支援 per-tier override。

FedProx local objective：

```text
local_loss = task_loss
             + (proximal_mu / 2) * sum(||w - w_global||^2)
```

`w_global` 是本輪開始時下載並驗證的 global weights immutable snapshot。Server-side
aggregation 維持 sample-weighted averaging。Branch 回報 Root 的 effective sample count
是該輪所有 accepted Leaf training sample counts總和。

### 5.9 Static topology config contract

Root main config：

```yaml
federated_learning:
  topology:
    strategy: static
    config_file: "./topology/hierarchical-topology.yaml"
  training_trigger:
    private_api:
      enabled: false
```

Referenced file：

```yaml
version: 1

admission:
  mode: complete_required

branches:
  - nf_instance_id: "22222222-2222-4222-8222-222222222222"
    leaves:
      - nf_instance_id: "33333333-3333-4333-8333-333333333333"
      - nf_instance_id: "44444444-4444-4444-8444-444444444444"
```

Validation：

- `config_file` 相對 main config所在目錄解析；
- `version` 第一版只接受 integer `1`；
- `strategy` 第一版只接受 `static`；
- `admission.mode` 第一版只接受 `complete_required`；
- 至少一個 Branch，每個 Branch 至少一個 Leaf；
- all identities 是 UUIDv4 且全樹唯一；
- containing Root identity 不得出現在 Branch／Leaf list；
- topology file 不保存 endpoint、URL、callback、subscription、`requestId`、`planId` 或
  `mlCorreId`；
- file missing、unreadable、unknown field、invalid version、empty branch 或 identity
  conflict皆使 Root config startup fail；
- 第一版無 hot reload；修改後重新啟動 Root；
- peer endpoint與 current registered capability 在每次新 attempt 透過 NRF解析，不寫回
  config file。

Branch／Leaf deployment不需要此檔案，也不配置 hierarchy role。只有可主動建立 plan 的
Root deployment配置 topology reference；這不改變其標準 NF identity。

Root配置 topology時，degradation intent進入本文件的 HFL coordinator；未配置 topology的
既有 FL Server仍可保留 flat distributed FL behavior。Private HFL API永遠要求 topology，
不得用 API request body臨時提供 topology。

### 5.10 Training initiation and private API contract

兩個 initiation producers 使用同一個 Root-owned request coordinator：

1. degradation producer；
2. config-enabled private management request API。

Private API exact first-version contract：

```text
POST /internal/v1/hierarchical-federated-learning/requests
GET  /internal/v1/hierarchical-federated-learning/requests/{request_id}
```

POST body：

```json
{
  "requestId": "99999999-9999-4999-8999-999999999999",
  "modelFamilyId": "ue-communication-default"
}
```

First acceptance：

```text
202 Accepted
Location: /internal/v1/hierarchical-federated-learning/requests/{request_id}
```

```json
{
  "requestId": "99999999-9999-4999-8999-999999999999",
  "planId": "11111111-1111-4111-8111-111111111111",
  "state": "ACCEPTED"
}
```

Status resource fields：

- `requestId`；
- `planId`；
- `modelFamilyId`；
- `state`: `ACCEPTED`、`RUNNING`、`SUCCEEDED` 或 `FAILED`；
- terminal machine-readable `failureCause` when failed。

`ACCEPTED` 表示 request與 guard已被接受但尚未建立第一個 standard subscription；建立後
進入 `RUNNING`。`SUCCEEDED` 只在 training、final validation、publication與必要 cutover
全部完成後設定；任何不可恢復錯誤設定 `FAILED`。第一版不提供額外 phase enum。

API rules：

- `training_trigger.private_api.enabled` 預設 `false`；disabled 時 route 不掛載並回 404；
- enabled 需要 FL Server engine與有效 static topology config，否則 startup fail；
- API 直接由 PyMTLF處理，不經過 Go NWDAF；
- body不允許 topology、strategy、participant 或 endpoint override；
- `requestId` 是 caller-provided UUIDv4 idempotency key；
- 相同 `requestId` + 相同 canonical body 重送，回傳既有 request／plan；
- 首次建立回 `202 Accepted`；相同 request的 replay回 `200 OK`，兩者都提供相同
  `Location`；
- 相同 `requestId` + 不同 body 回 `409 Conflict`；
- 已有另一個 active experiment 時，新 `requestId` 回 `409 Conflict`；
- malformed request回 `400 Bad Request`；unknown family回 `404 Not Found`；沒有 current
  model、沒有可用 active training scope或 single-active conflict回 `409 Conflict`；
  所有 rejection都發生在 plan dispatch前，不建立半套 standard subscriptions；
- GET unknown `requestId` 回 `404 Not Found`；
- idempotency registry只屬於 current PyMTLF process generation；restart後不恢復 request
  records，符合 fresh-state decision；
- errors 使用既有 private `application/problem+json` convention；
- 新 path目前不在 global `RequestValidationError` private-prefix handling內，Slice 3必須
  明確納入，確保 malformed body依本契約回 400而不是 FastAPI default 422；
- 第一版不新增 application-level authentication，沿用現有 internal API deployment／network
  access boundary；後續若要跨 trust domain暴露，必須另開 security slice。

Degradation producer以 server-generated `requestId` 進入相同 coordinator。兩種 producer
都在建立任何 `Nnwdaf_MLModelTraining` resource之前先取得 instance-level guard；Training
SBI 不作 initiation API。

### 5.11 Single-active experiment contract

每個 PyMTLF instance 同時間只允許一個 active top-level training experiment：

- Root：一個 `planId` + upper Server process；
- Branch：同一 `planId` 下的一個 upper Client resource group + 一個 lower Server
  process，合計仍是一個 experiment；
- Leaf：一個 lower Client resource group。

Guard 不只是 `FLServerSettings.max_active_processes`：

- HFL-enabled config deterministic要求 `max_active_processes: 1`；
- shared registry跨 FL Server／Client engines仲裁 active `planId`；
- matching Branch upper／lower resources加入同一 plan，不被誤判為兩個 experiments；
- unrelated inbound assignment、degradation 或 private request在 guard occupied 時拒絕；
- duplicate standard request只有在 current resource identity與 canonical content相符時才
  idempotent；conflicting duplicate fail closed。

Inbound Client create在非同步下載 bundle前尚未知道 `planId`，因此必須先取得一個
instance-level provisional reservation；bundle驗證成功後再把 reservation bind到
`planId`。驗證失敗即 terminal並釋放 reservation。這避免兩個 concurrent creates在
background worker才同時發現衝突。

`FAILED`、`SUCCEEDED` 與完成必要 cleanup後才釋放 guard。Release guard 不復活原
degradation intent；後續實驗必須由新 trigger明確建立。

Failure latch rules：

- Root experiment terminal failure先設定 `manual_restart_required`，再完成 bounded cleanup
  並釋放 active guard；
- latch存在時，後續 degradation intents只記錄／抑制，不建立新 `planId`；
- enabled private API提交新的 `requestId` 是 Root第一版的 explicit operator restart；它
  成功取得 guard並建立新 plan後才清除 latch；
- private API disabled時，Root第一版只能透過 process restart清除 latch並重新允許
  degradation initiation；latch不持久化；
- Branch／Leaf收到由 Root operator重新建立、具有全新 `planId` 的 valid assignment，視為
  explicit new attempt，可以在舊 attempt已 terminal並完成 cleanup後取得 guard；
- same／stale `planId` 不得清除 latch或重建 resource；
- successful experiment不設定 failure latch，之後新的 degradation evidence仍可觸發新的
  experiment。

### 5.12 Failure and retry contract

第一版只有 `complete_required` + `all` + `all`：

- 任一 assigned Branch／Leaf preparation failure或 timeout -> 整個 attempt failed；
- 任一 selected participant round failure或 timeout -> 整個 experiment failed；
- invalid／stale／conflicting callback或 bundle -> current experiment failed；
- 不補選、不重新分組、不保留跨 attempt成功 participants；
- Branch 不自行替換 Leaf；
- Root 不自動重新建立 `planId`。

允許的 retry 僅限不改變業務 attempt 的 bounded transport行為：

- identical notification delivery retry；
- idempotent HTTP replay；
- best-effort cleanup DELETE retry；
- temporary download transport retry within the same deadline。

不得把 transport retry擴大成 preparation、round、participant selection、topology 或完整
experiment retry。

### 5.13 Fresh-state restart contract

Root、Branch、Leaf 的 Go 或 PyMTLF process重啟後：

- 產生新的 backend `processInstanceId`；
- active hierarchy registry、mapping、participant snapshot與 round state皆為空；
- 不讀取 temporary workspace來重建 process；
- 不恢復舊 `planId` 或 standard subscriptions；
- 不與 peers執行 crash reconciliation。

Go 已有 backend generation reset，可在 PyMTLF generation變更時移除相依 Training routes並
對 consumer-owned remote resource做一次 best-effort cleanup。HFL沿用此 boundary，不新增
durable replay log。

Restart前的 callback、result bundle或 round artifact只可能：

- 因 route不存在回 404；
- 因 generation／correlation／`planId` 不符被拒絕；
- 因 publisher artifact expiry下載失敗。

它們不得隱式建立 resource或恢復 experiment。Workspace仍需 TTL garbage collection與
normal terminal per-plan cleanup，但 crash前可能遺留的 remote resource不構成 recovery
需求。

---

## 6. Slice ownership and implementation handoff

### Slice 1: bundle／artifact primitives

主要 repository：`PyMTLF/`。

依本文件建立：

- typed strategy models；
- hierarchy assignment／result discriminated unions；
- artifact role validation；
- Root publish、Branch download／republish、result publish primitives；
- per-plan artifact release與 TTL tests；
- positive／negative schema fixtures。

Slice 1 不建立完整 Root／Branch orchestration。

### Slice 2: capability and process foundation

主要 repositories：`NWDAF/`、`PyMTLF/`。

- runtime `local|federated` migration；
- server／client engine-presence gating；
- Go advertised capability projection；
- PyMTLF readiness consistency validation；
- shared active experiment registry；
- Branch upper／lower mapping model；
- pure Server／Client與 combined capability regression。

### Slice 3: Root initiation／topology／assignment

主要 repository：`PyMTLF/`，必要時只擴充既有 Go NRF gateway contract。

- static topology loading；
- private request／status API；
- degradation producer integration；
- Root plan creation與 guard；
- configured target NRF resolution；
- Branch assignment publish與 upper preparation dispatch。

### Slice 4–7

- Slice 4：Root–Branch–Leaf asynchronous preparation與 `complete_required` admission；
- Slice 4同時修正 standard preparation notification最低欄位集合、success sender與
  preparation-stage `mLModelInfos` receiver；
- Slice 5：FedProx local objective、Branch lower aggregation、effective weight與 Root upper
  aggregation；
- Slice 6：hierarchical final validation、evidence gate與publication；
- Slice 7：terminal lifecycle、cleanup、fresh-state restart與stale interaction hardening。

### Slice 8

`nwdaf-resources/` 建立：

- smoke：1 Root + 1 Branch + 2 Leaves；
- aggregation acceptance：至少 2 Branches，且能驗證不同 subtree sample counts；
- degradation與 enabled private API triggers；
- private API disabled profile；
- restart／stale callback scenarios；
- PyAnLF final Provision／activation regression。

---

## 7. Slice 0 review checklist

### 7.1 Architecture consistency

- [x] 沒有新增 Root／Branch／Leaf NF type或 permanent runtime role。
- [x] Branch eligibility明確要求 `FL_SERVER_AND_CLIENT`。
- [x] per-tier標準 `mlCorreId` 與 vendor `planId` 未混用。
- [x] Go只擁有標準 transport、NRF gateway與最小 route state。
- [x] PyMTLF擁有 topology、strategy、bundle、admission與 aggregation。
- [x] PyAnLF維持 final Provision consumer boundary。

### 7.2 Contract completeness

- [x] Branch assignment、Leaf assignment與 preparation result皆有 version、publisher、
  intended recipient與 `planId`。
- [x] result partitions完整覆蓋 assigned client set。
- [x] bundle始終包含有效 model artifact。
- [x] preparation success不再只送 `statusReport`，且不誤帶 standard `roundInd`。
- [x] Go／PyMTLF validator 不再把 `statusReport` 單獨存在視為有效最低結果集合。
- [x] preparation-stage receiver 以 `mLModelInfos`、active stage 與 correlation 判定成功，
  不要求 `statusReport`，也不把固定 `samplRatio: 100` 當 completed latch。
- [x] non-hierarchical distributed FL preparation regression tests 已納入 Slice 4 範圍。
- [x] Branch-to-Leaf URL由 Branch發布，而非 Root URL passthrough。
- [x] strategy只接受 FedProx／all／all／sample-weighted。
- [x] `proximal_mu` 只在 typed algorithm block且 `> 0`。
- [x] topology config只含 static identities、tree與 admission。

### 7.3 Lifecycle completeness

- [x] private API預設關閉、由 PyMTLF直接處理且不經 Go。
- [x] degradation與 private API共用 single-active guard。
- [x] Branch upper／lower processes算同一 active experiment。
- [x] preparation與 round必要 participant failure皆 terminal。
- [x] transport retry沒有變成 experiment retry。
- [x] restart後不恢復、reconcile或隱式重建舊 experiment。

### 7.4 Repository readiness

- [x] production branches依既有命名與 baseline建立。
- [x] 每個 production slice有自己的 detailed plan、focused tests、full regression與 commit。
- [x] 不跨 repositories混合 commit。
- [x] 不修改 `resources/` reference tree。

---

## 8. Deferred behavior

第一版明確不做：

- FedAvg／`proximal_mu: 0` baseline；
- `fixed_count` participant selection；
- `minimum_results` waiting／quorum；
- non-sample-weighted aggregation；
- per-tier strategy override；
- dynamic topology或 hot reload；
- partial admission／minimum clients；
- automatic participant replacement／re-grouping／re-plan；
- arbitrary-depth recursive hierarchy；
- Branch artifact transparent proxy mode；
- active experiment persistence／restart recovery／reconciliation；
- Go-owned private trigger endpoint；
- full OAM MnS／TS 28.532 implementation；
- private API cross-trust-domain authentication；
- PyAnLF理解 hierarchy metadata。

上述欄位的 schema extension points可以保留，但不得提供看似可配置、實際未實作的合法
值。

---

## 9. Slice 0 completion criteria

Slice 0 在下列條件全部成立後完成：

1. 本文件通過 team review，且不再有會改變 Slice 1 schema ownership的 open decision；
2. baseline evidence map可對應到實際 files與 current tests；
3. assignment／result／strategy／topology／runtime／trigger contracts互相一致；
4. standard與 vendor ownership明確，不需要修改 Release 18 OpenAPI；
5. restart、single-active與 failure semantics沒有隱含 recovery或 automatic retry；
6. Slice 1可以只依本文件建立 typed bundle／artifact primitives，不必重新決定 schema。

Review通過後，下一個 production action是建立 Slice 1 detailed plan，再於
`PyMTLF/feat/r18-hierarchical-federated-learning` 實作 hierarchy bundle contract and
artifact primitives。
