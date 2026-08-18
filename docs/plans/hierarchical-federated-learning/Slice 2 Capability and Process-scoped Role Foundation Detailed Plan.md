# Slice 2：Capability 與 Process-scoped Role Foundation 詳細計畫

日期：2026-08-18

狀態：Completed；production implementation、review 與 verification 已完成

上層計畫：

- [Hierarchical NWDAF Federated Learning Implementation Plan](./Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)

前置 contracts：

- [Slice 0 Baseline Audit and Contract Freeze Detailed Plan](./Slice%200%20Baseline%20Audit%20and%20Contract%20Freeze%20Detailed%20Plan.md)
- [Slice 1 Hierarchy Bundle Contract and Artifact Primitives Detailed Plan](./Slice%201%20Hierarchy%20Bundle%20Contract%20and%20Artifact%20Primitives%20Detailed%20Plan.md)

相關文件：

- [Hierarchical NWDAF Federated Learning Proposal Draft](../../proposals/nwdaf/hierarchical-federated-learning/proposal_draft.md)
- [NWDAF Development Policy](../../development_policy.md)

---

## 1. 目的與完成邊界

Slice 2 要移除 PyMTLF 目前將 FL Server 與 FL Client 視為互斥部署角色的限制，改成由
同一個標準 NWDAF 所廣告的 FL capability 決定可啟用的 engines，並建立 hierarchy
experiment 的 process-memory ownership foundation。

完成後，同一份 NWDAF／PyMTLF implementation 必須能形成三種 capability profiles：

| Deployment capability | Enabled PyMTLF engines | 可被後續 assignment 指派的位置 |
| --- | --- | --- |
| `FL_SERVER` | Server | Root |
| `FL_CLIENT` | Client | Leaf |
| `FL_SERVER_AND_CLIENT` | Server + Client | Root、Branch 或 Leaf |

表中的位置不是 deployment role。實際 Root、Branch、Leaf 仍須由後續 Slice 3／4 的
Root assignment、有效 hierarchy bundle 與 active `planId` 決定。

本 Slice 必須完成：

1. 將 PyMTLF `runtime.mode` 由互斥 `local`／`fl_server`／`fl_client` 收斂成
   `local`／`federated`；
2. 以 `federated_learning.server`／`client` section 是否存在決定對應 engine 是否啟用；
3. 讓 Server 與 Client engines 可在同一 PyMTLF process 內共同啟動、接受各自的 private
   Training routes，並共同停止；
4. 由 Go containing-NWDAF private context 投影實際 NRF profile 中的 FL capabilities，
   讓 PyMTLF readiness 驗證 engine 與 advertisement 一致；
5. 建立 instance-level active experiment registry、provisional Client reservation 與
   Branch upper／lower process mapping primitives；
6. 固定 single-active、identity retirement、fresh-state restart、bounded concurrency 與
   cleanup ownership；
7. 保留既有純 Server、純 Client 與 local profiles 的 regression behavior，並增加
   Server + Client combined profile tests。

本 Slice 不會建立 topology、主動產生 `planId`、解析 assignment 後建立 lower-tier
subscriptions，或實作完整 Root／Branch／Leaf orchestration。那些行為分別屬於 Slice 3
與 Slice 4。本 Slice 也不修改 Release 18 OpenAPI。

---

## 2. 已確認且不得在本 Slice 重新決策的事項

本 Slice 直接繼承下列已確認 decisions：

- 所有節點都是標準 NWDAF，不增加新的 NF type 或 NRF hierarchy role；
- capability 決定 eligibility，Root assignment 才決定某個 `planId` 中的實際位置；
- 不接受永久 `root`、`branch`、`leaf` 或 `fl_branch` runtime mode／config；
- Root 至少需要 FL Server capability；Branch 必須具備
  `FL_SERVER_AND_CLIENT`；Leaf 至少需要 FL Client capability；
- `planId` 是 vendor hierarchy attempt identity，不取代每層標準 `mlCorreId`；
- Root upper Server process、Branch upper Client process 與 Branch lower Server process
  保有各自的標準 resource／correlation identity；
- Branch 只以 process-memory state 連結 upper／lower processes；Go 不需要理解
  hierarchy mapping；
- 每個 PyMTLF instance 同時間只允許一個 top-level training experiment；Branch 的
  upper Client resource group 與 lower Server process 合計仍算同一個 experiment；
- 所有 Go／PyMTLF restart 都從全新 state 開始，不 persistence、resume、reconcile 或
  reconstruct 舊 hierarchy process；
- 任一必要 participant failure 終止 experiment，且不自動 retry；failure latch 與明確
  operator restart 的 Root initiation integration 屬於 Slice 3；
- Go NWDAF 是唯一標準 NF／SBI boundary；PyMTLF private API 不因此變成標準 SBI；
- Go 已廣告的 FL capability 是唯一 deployment capability source；PyMTLF 不配置第二份
  advertised capability；
- 第一版不新增 caller identity、expected Root identity 或 hierarchy security header。

若實作需要改變以上任一點，必須先更新 canonical plan，不得以新增 mode、複製 capability
config 或把 `planId` 塞入 Go route state 的方式繞過 decision。

---

## 3. Repository、branch 與責任歸屬

### 3.1 受影響的 repositories

Production repositories：

- `NWDAF/`
- `PyMTLF/`

Documentation repository：

- `nwdaf-docs/`

實作 branch：

- `NWDAF/feat/r18-hierarchical-federated-learning`
- `PyMTLF/feat/r18-hierarchical-federated-learning`

撰寫本計畫時的 baseline：

| Repository | Branch | Revision | 狀態 |
| --- | --- | --- | --- |
| `NWDAF/` | `feat/r18-federated-learning` | `c53f058` | clean，已同步 remote |
| `PyMTLF/` | `feat/r18-hierarchical-federated-learning` | `6cc1df0` | clean，Slice 1 已完成 |
| `nwdaf-docs/` | `main` | `dc228f7` | clean，已同步 remote |

開始 production implementation 前，`NWDAF/` 應從目前已 review 的 distributed-FL
baseline 建立與 PyMTLF 一致命名的 hierarchy branch。不得在 `nwdaf-docs/` commit 中混入
production repository changes，也不得把兩個 production repositories 混成同一 commit。

本 Slice 不修改：

- `PyAnLF/`；
- `nrf/`；
- `nwdaf-resources/`；
- `resources/` reference tree；
- Release 18 TS／OpenAPI corpus。

### 3.2 Responsibility boundary

| State／behavior | Owner |
| --- | --- |
| NRF profile config 與 registered FL capability | Go NWDAF |
| containing-NWDAF capability projection | Go private MTLF boundary |
| public Training subscription／callback routes | Go NWDAF |
| standard subscription ID、peer/backend location、generation | Go NWDAF context |
| FL Server／Client engine enablement | PyMTLF config／app lifecycle |
| capability consistency readiness | PyMTLF，資料來源為 Go projection |
| `planId`、assigned role、upper／lower mapping | PyMTLF process memory |
| single-active experiment arbitration | PyMTLF shared registry |
| standard Client resource state | PyMTLF FL Client engine |
| standard Server process／participant state | PyMTLF FL Server engine |
| restart 時移除 Go backend-dependent routes | 既有 Go backend generation reset |

---

## 4. 已確認的實作基線

### 4.1 PyMTLF config 與 lifecycle

`src/py_mtlf/config.py` 目前：

- `runtime.mode` 只接受 `local`、`fl_server`、`fl_client`；
- `Settings.validate_runtime_configuration()` 明確禁止 FL Server 與 Client sections 同時
  存在；
- `FLServerSettings.max_active_processes` 預設為 `1`，但 schema 允許 `1..32`；
- `FLClientSettings.max_concurrent_jobs` 是工作 executor capacity，不是 experiment count。

`src/py_mtlf/app.py` 目前雖然總是建立 `FLClientService` 與
`FLServerOrchestrator`，但 route、worker 與 model service lifecycle 仍以互斥 mode gate。
Disabled engine 也會以 default settings 被建構，object existence 因此不能代表 capability。

`src/py_mtlf/api/ml_model_training.py` 目前再做一次 mode gate：

- subscription CRUD 只允許 `runtime.mode == "fl_client"`；
- notification ingress 只允許 `runtime.mode == "fl_server"`。

因此，目前同一 process 無法同時處理 Branch 的 upper Client resource 與 lower Server
callbacks。

### 4.2 PyMTLF Client／Server state

`FLClientService` 目前擁有：

- `_resources`，key 為 backend subscription ID；
- standard `mlCorreId`／`notifCorreId`／round identity；
- bounded executor、callback outbox、delay timers 與 close lifecycle；
- preparation、round、validation、result 與 terminal states。

`FLServerOrchestrator` 目前擁有：

- `_processes`，key 為 server-generated process UUID；
- `_correlations`，將 notification correlation 對應到 Server process；
- participant subscription locations 與 per-process condition；
- executor capacity 與 close lifecycle。

兩個 engine 的 locks、maps 與 capacities彼此獨立。現有結構沒有 instance-level owner 可判斷
某個 upper Client group 與 lower Server process 是否屬於同一 hierarchy experiment。

`FLClientService.restore_after_restart()` 與
`FLServerOrchestrator.discard_restored_routes()` 目前沒有 production caller。它們不是 recovery
protocol，本 Slice 不得接上這些 helpers。除非它們造成實際 current-slice contradiction，
移除 dead helpers 只列為 legacy cleanup，不擴大本 Slice。

### 4.3 Go standard route state 已可重用

`internal/context/ml_model_training_routes.go` 的
`MLModelTrainingSubscriptionRoute` 已保存：

- public／private subscription ID；
- standard `mlCorreId`、`notifCorreId` 與 expected round；
- accepted／backend representations；
- peer/backend locations；
- inbound／outbound direction；
- lifecycle revision 與 backend process generation。

`internal/sbi/processor/ml_model_training.go` 已有兩條需要的 route：

1. peer／external request進入 Go，再建立本地 PyMTLF Client resource；
2. PyMTLF Server透過 private Go boundary選定 peer，再建立 outbound peer Training resource。

因此 Branch 可以在同一 Go process 中同時存在 upper-tier inbound route 與 lower-tier
outbound routes。Go route state 不需要新增 `planId`、Root identity 或 hierarchy role；它只
維持各標準 resource 的 routing／correlation。

### 4.4 Go restart boundary 已可重用

`pkg/service/init.go` 以 backend readiness 的 `processInstanceId` 監看 PyMTLF generation。
generation 改變時，`ResetMLModelBackendGeneration()` 會移除依賴舊 backend 的 Training
routes，並對 consumer-owned remote resource做一次 best-effort DELETE。

這已符合 fresh-state restart 的 Go boundary。本 Slice 不增加 durable route replay、舊
subscription restore 或 crash reconciliation。

### 4.5 containing-NWDAF context gap

Go 的 `/internal/v1/nwdaf-context` 目前只回傳：

- `nfInstanceId`；
- `apiRoot`；
- `internalApiRoot`。

PyMTLF `NwdafContextClient` 也只解析並 cache 這三項。Go runtime context 已保有實際
registration用 `NFProfile` snapshot，其中包含 Release 18 `mlAnalyticsList` 與
`flCapabilityType`；缺口只是尚未投影到 private response。

### 4.6 free5GC implementation-shape evidence

本 Slice 的 Go lifecycle／config shape 以 workspace local mirror 的 free5GC `nrf` 作為主要
exemplar：

- `pkg/factory/config.go`：validated config 留在 factory boundary；
- `pkg/service/init.go`：context 在 processor／consumer／server 前建立，server 與 goroutine
  由 app lifecycle owner 啟停；
- `internal/context`：runtime profile state由 context owner保存。

這只支持 Go package placement 與 lifecycle ownership。free5GC 沒有 exact hierarchical
FL experiment registry exemplar；PyMTLF shared registry 是依本計畫 process ownership 所做的
project-specific design，不宣稱是 free5GC-defined pattern。

---

## 5. Target capability model

### 5.1 PyMTLF config migration

Target config：

```yaml
runtime:
  mode: federated

federated_learning:
  server:
    max_active_processes: 1
    # existing server settings
  client:
    # existing client settings
```

Validation rules：

- `runtime.mode` 只接受 `local` 或 `federated`；
- `local` 不得配置 `federated_learning.server` 或 `client`，並保留既有 local trainer；
- `federated` 至少配置 `server` 或 `client`；
- `server` 存在即 `server_enabled == true`；
- `client` 存在即 `client_enabled == true`；
- 兩個 sections 可同時存在；
- federated Server第一版 deterministic要求 `max_active_processes == 1`；
- `FLClientSettings.max_concurrent_jobs` 繼續限制同一 experiment 內的工作量，不改作
  experiment guard；
- 舊 `fl_server`、`fl_client` values 不保留為 hidden aliases，repository sample configs、
  tests 與文件要在同一 checkpoint 完成 deterministic migration；
- unknown hierarchy role values一律 validation failure。

需要維護三個 federated sample profiles：

- Server-only：既有 `config/fl-server.yaml` 遷移為 `mode: federated`；
- Client-only：既有 `config/fl-client.yaml` 遷移為 `mode: federated`；
- Server + Client：新增 combined capability sample，用於 Branch engine foundation；檔名不得
  將 Branch 當成永久角色，建議 `config/fl-server-client.yaml`。

### 5.2 Engine construction and availability

`create_app()` 不再以 default settings 建構 disabled engine：

- `server` section不存在時，`app.state.fl_server` 明確為 `None`；
- `client` section不存在時，`app.state.fl_client` 明確為 `None`；
- enabled engine只建構一次，並由 FastAPI lifespan owner關閉；
- combined profile建立兩個 engines，兩者共用同一 `NwdafContextClient`、workspace 與 shared
  experiment registry；
- local profile不建構 FL engines。

Route rules：

| Route group | Availability condition |
| --- | --- |
| Provision／Monitor | `local` 或 Server engine enabled |
| Training subscription CRUD | Client engine enabled |
| Training notification ingress | Server engine enabled |
| Artifact／health／supporting private routes | 保留既有 availability |

Training router可在任一 FL engine enabled時掛載，但每個 handler必須以對應 engine
existence／capability判斷，不再檢查 `mode == fl_client` 或 `mode == fl_server`。Combined profile
的 OpenAPI paths與實際 handler behavior都必須同時可用。

### 5.3 Go capability projection contract

Go private response增加：

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

Contract rules：

- authoritative source是 Go 已 validated、準備註冊至 NRF 的 `NFProfileSnapshot()`；
- 只投影 `mlAnalyticsIds` 與 `flCapabilityType`；
- 不回傳 topology role、`planId`、endpoint assignment或 duplicated PyMTLF config；
- entries、analytics IDs canonical sort並去重，使相同 profile得到 deterministic response；
- malformed／unsupported capability不得由 handler猜測或修正；
- 已正確初始化但沒有 `nwdafInfo`／FL capability的 local profile回傳空 projection；
- MTLF private context handler無法取得有效 snapshot時回 service unavailable，不回 partial
  identity response；
- shared response type可保留 `omitempty` 供其他 backend context consumers相容，但 MTLF
  response在有效 federated advertisement下必須明確提供 projection。

預計 Go 變更邊界：

- `internal/backend/contract.go`：private response models；
- `internal/context/context.go` 或同 package focused helper：thread-safe capability projection；
- `internal/mtlf/api_nwdaf_context.go`：MTLF response assembly；
- 對應 context／handler tests。

不修改 public SBI、不修改 generated models、不修改 NRF profile schema。

### 5.4 PyMTLF readiness consistency

PyMTLF將 Go projection解析成 per-analytics typed entries，再計算 advertisement的 global
engine bits：

- `advertised_server = any(FL_SERVER or FL_SERVER_AND_CLIENT)`；
- `advertised_client = any(FL_CLIENT or FL_SERVER_AND_CLIENT)`。

Ready condition要求：

```text
(configured_server, configured_client)
==
(advertised_server, advertised_client)
```

這是 exact engine consistency，不是 peer assignment eligibility。某個 Analytics ID 是否
真的具備 combined capability，仍由 Slice 3／4 對該 analytics做 per-entry validation。

Readiness rules：

- FastAPI lifespan不因 Go listener尚未啟動而永久 blocking；
- `/health/ready` 透過 refreshable checker取得 containing-NWDAF context；
- Go unavailable、response malformed或 capability mismatch時回 `503 not_ready`；
- 後續 probe可重新取得並在一致後轉為 ready；
- successful context可以 cache，但 mismatch／unavailable不得永久 cache成不可恢復 state；
- readiness payload保留 `processInstanceId` 與 `runtimeMode`，並增加可判斷 enabled engines與
  capability verification state的簡短欄位；
- readiness failure不建立或恢復 experiment state；
- normal shutdown仍關閉 context HTTP client。

測試需使用 injectable／fake `NwdafContextClient` 或 checker，不能依賴 running Go process。

---

## 6. Shared active experiment registry

### 6.1 Registry purpose

新增 process-local shared registry，作為 FL Client 與 FL Server engines之上的 ownership
arbiter。它不取代兩個 engines自己的 resource maps，也不處理 standard wire validation。

Registry只回答：

1. 這個 PyMTLF instance目前是否已有 active top-level experiment；
2. 新的 Client resource是否屬於同一 provisional upper process group；
3. 驗證 hierarchy bundle後，該 group綁定哪個 `planId` 與 assigned role；
4. Branch lower Server process是否可附加至同一 plan；
5. terminal cleanup後何時可以釋放 active slot；
6. 某個 completed／failed `planId` 是否已在本 process lifetime被 retired。

建議放在新的 focused module，例如 `src/py_mtlf/core/fl_experiment.py`。這是 module，不建立
新的 package layer。

### 6.2 State model

建議 typed state至少包含：

```text
ExperimentRecord
├─ reservation_id                 process-local UUID
├─ plan_id                        optional，bundle validation前未知
├─ assigned_role                  ROOT | BRANCH | LEAF | none
├─ lifecycle                      PROVISIONAL | ACTIVE | TERMINAL | CLEANING
├─ upper_client_ml_correlation_id optional
├─ upper_client_subscription_ids  set
├─ server_process_id              optional
├─ terminal_outcome               optional
└─ cleanup_pending                boolean
```

`assigned_role` 是 active process state，不是 config。欄位名稱可在實作 review時依 local
naming微調，但 invariants不得改變。

Registry另保留 current process lifetime的 `retired_plan_ids`。Retired identity只用來拒絕
same／stale `planId`，不保存 topology、subscription representation或 round state。

### 6.3 Provisional Client reservation

Inbound Client create在 background下載 hierarchy bundle前不知道 `planId`，因此 admission
順序固定為：

1. 完成 standard request基本 validation；
2. 產生 backend subscription ID；
3. 以 `subscription_id` + standard `mlCorreId` 向 shared registry取得 provisional
   reservation；
4. reservation成功後才把 Client resource加入 engine map並開始 background operation；
5. create失敗時同步移除 Client resource並回滾 reservation；
6. Slice 4驗證 bundle後，以 `planId` 與 assigned role bind reservation。

同一 upper standard Server process可能建立一個以上的 Client resources，因此相同非空
`mlCorreId` 可加入同一 provisional group；不同 `mlCorreId` 在 active slot occupied時必須
拒絕。這避免把同一 experiment的 resource group誤判為多個 experiments，也避免兩個
concurrent creates在 background worker才同時發現衝突。

### 6.4 Server process attachment

Server engine建立 process時必須向同一 registry登記：

- Root：Slice 3先建立／bind plan，再 attach upper Server process；
- Branch：Slice 4以已 bind 的同一 `planId` attach lower Server process；
- unrelated process：active slot occupied時拒絕；
- 同一 record不得 attach第二個 conflicting Server process；
- existing non-hierarchical Server flow使用 process-local reservation，不假造 hierarchy
  `planId`，並維持 single-active behavior。

本 Slice只建立並測試 attach primitives及既有 Server flow的 guard integration；真正由
assignment驅動 Branch lower process建立屬於 Slice 4。

### 6.5 Identity and lookup rules

- `reservation_id` 只在 PyMTLF memory中使用，不進標準 request／bundle；
- `planId` 只從 Root initiation或已驗證 hierarchy metadata取得；
- `mlCorreId` 仍由每個 standard Server process擁有；
- subscription ID與 notification correlation仍由對應 engine／Go route擁有；
- registry不得用相同 round、NF identity、URL或 callback URI推測 cross-tier relationship；
- Go不需要保存 reservation ID或 `planId`；
- callback先由現有 standard correlation map找到 engine process，再由 process mapping檢查
  active ownership；callback本身不得建立 registry record。

### 6.6 Terminal and release rules

- 成功、失敗或明確 termination先將 record設為 `TERMINAL`；
- remote subscription、timer、future、artifact reference等必要 cleanup進行時為
  `CLEANING`；
- cleanup完成或達到既定 bounded best-effort終點後才 release active slot；
- 已知 `planId` 在 release時加入 `retired_plan_ids`；
- same／stale `planId` 不得重新 bind；
- release不重啟 degradation intent，也不自動建立新 experiment；
- Root failure latch與 private operator restart integration延至 Slice 3，但 registry需提供
  terminal／release hook，不得迫使後續繞過 guard。

### 6.7 Locking and lifecycle

- registry使用單一明確 lock保護 record、indexes與 retired identities；
- 不在持有 registry lock時做 HTTP、bundle I/O、等待 condition或 executor shutdown；
- engine lock與 registry lock不得形成反向 nested acquisition；實作前要在 methods docstring
  或 focused comment寫明 acquisition rule；
- reservation／bind／attach／terminal／release methods回傳 immutable snapshot或 token，
  不把 mutable internal record洩漏給 engines；
- shutdown先阻止新 admission，再停止 timers／workers／HTTP clients；
- restart建立新的空 registry，不讀 temporary workspace重建 record。

---

## 7. Go route state 與 PyMTLF business state 切分

### 7.1 Branch實際 flow中的 state placement

```text
Root Go public Training request
  -> Branch Go inbound Training route
       stores standard route/callback/backend generation
  -> Branch PyMTLF Client resource
       reserves shared experiment slot
       later binds validated planId as BRANCH
  -> Branch PyMTLF Server process (Slice 4)
       attaches lower process to same planId
  -> Branch Go outbound Training routes to Leaves
       each stores its own standard peer route/correlation
```

Go只知道獨立的 standard resources。PyMTLF才知道這些 resources在 business semantics上屬於
同一個 hierarchy experiment。

### 7.2 Explicit non-changes

本 Slice不得：

- 在 Go `MLModelTrainingSubscriptionRoute` 增加 `planId`；
- 增加 expected Root ID、actual peer caller ID或 hierarchy role header；
- 讓 Go解析 hierarchy bundle；
- 讓 PyMTLF直接呼叫 public peer SBI而繞過 Go；
- 以 Go route存在推導 PyMTLF experiment已恢復；
- 為 restart persistence Go／PyMTLF mapping。

### 7.3 Restart example

若 Branch PyMTLF restart：

1. 新 PyMTLF產生新的 `processInstanceId` 與空 registry；
2. Go availability monitor偵測 generation改變；
3. Go移除依賴舊 backend generation的 inbound／outbound Training routes，並做既有
   best-effort cleanup；
4. 舊 callback因 route或 correlation不存在而被拒絕；
5. PyMTLF不從 workspace、舊 URL或 callback重建 plan；
6. 只有 Root後續以全新 `planId` 發出的 valid assignment能開始新 attempt。

---

## 8. 預計 production 變更範圍

### 8.1 `NWDAF/`

預計修改：

- `internal/backend/contract.go`
  - 增加 capability projection private models；
- `internal/context/context.go` 或 focused同 package helper
  - 從 locked NF profile snapshot建立 canonical projection；
- `internal/mtlf/api_nwdaf_context.go`
  - 回傳 projection並處理 snapshot failure；
- `internal/mtlf/api_nwdaf_context_test.go`
  - Server-only、Client-only、combined、sorting／dedupe與 failure cases；
- 必要的 context focused tests。

只有在 test證明 current factory validation無法接受合法
`FL_SERVER_AND_CLIENT` profile時，才修改 `pkg/factory`；不得先建立第二套 capability config。

### 8.2 `PyMTLF/`

預計修改：

- `src/py_mtlf/config.py`
  - `local|federated` migration與 engine-presence validation；
- `src/py_mtlf/app.py`
  - optional engine construction、shared registry與 lifecycle wiring；
- `src/py_mtlf/api/ml_model_training.py`
  - engine-based handler availability；
- `src/py_mtlf/api/health.py`
  - dynamic capability consistency readiness；
- `src/py_mtlf/core/nwdaf_context.py`
  - typed capability projection parsing與refresh behavior；
- `src/py_mtlf/core/fl_experiment.py`（建議名稱）
  - shared registry與mapping primitives；
- `src/py_mtlf/core/fl_client.py`
  - provisional reservation／rollback／release integration；
- `src/py_mtlf/core/fl_server.py`
  - Server process guard／attach integration；
- `config/fl-server.yaml`、`config/fl-client.yaml`
  - deterministic mode migration；
- `config/fl-server-client.yaml`
  - combined engine sample；
- `README.md`、`docs/api.md`
  - capability-oriented runtime與route matrix；
- focused tests與既有 regression tests。

若實作發現需要新增超過上述 focused module的 package或新的 Go private endpoint，屬於
ownership decision gate，必須先更新本計畫。

---

## 9. 實作順序與 checkpoints

### Checkpoint 1：Go capability projection

1. 先新增 failing handler／context tests；
2. 建立 typed private projection；
3. 從 actual `NFProfileSnapshot()` canonical投影；
4. 更新 MTLF context response；
5. 驗證既有 context與registration tests沒有 regression。

Checkpoint完成條件：PyMTLF有單一、可驗證且與 NRF advertisement同源的 capability資料。

### Checkpoint 2：PyMTLF config and engine lifecycle

1. 先將 config matrix寫成 table-driven／parameterized tests；
2. 實作 `local|federated` validator；
3. 只建構 enabled engines；
4. 將 route與retrain dispatch改成 engine-based gating；
5. 遷移 sample configs與文件；
6. 驗證 local、Server-only、Client-only、combined profiles。

Checkpoint完成條件：combined process同時能接受 Client CRUD與 Server notification handler，
disabled engine沒有 worker／client／resolver lifecycle。

### Checkpoint 3：Capability readiness

1. 先建立 projection parse與mismatch tests；
2. 擴充 `NwdafContext` typed model；
3. 實作 refreshable consistency checker；
4. 將 checker納入 health readiness；
5. 驗證 Go late-start、temporary unavailable、malformed與mismatch後可恢復情境。

Checkpoint完成條件：engine／advertisement不一致時不 ready，一致且 artifact healthy時 ready。

### Checkpoint 4：Shared registry primitives

1. 先以純 unit tests固定 state invariants；
2. 實作 provisional Client group reservation；
3. 實作 `planId` bind、role assignment、Server attach；
4. 實作 terminal、cleanup、release與retired identity；
5. 加入 shutdown admission fence與lock-order tests。

Checkpoint完成條件：不啟動 Go／NRF也能完整驗證 single-active與Branch pairing semantics。

### Checkpoint 5：Engine integration and regression

1. Client create接入 reservation／rollback；
2. Server process建立接入 shared guard；
3. 驗證同一 `mlCorreId` resource group與conflicting process；
4. 驗證 combined profile的 upper Client + lower Server test seam；
5. 驗證 fresh registry on restart／new app construction；
6. 跑 repository full verification。

Checkpoint完成條件：Slice 3／4可以直接使用 registry APIs，不需要再改寫 capability model或
engine lifecycle。

### Checkpoint 6：不中斷 code review and remediation

依 development policy，implementation與full verification完成後不中斷：

1. review兩個 production repository的完整 Slice diff；
2. 檢查 package placement、data flow、lock order、shutdown與skipped verification；
3. 對已確認且不改變本計畫的 findings直接 test-first修復；
4. 執行 targeted verification與follow-up review；
5. 最後重跑被修復影響的 full verification；
6. 分 repository準備 commit checkpoints。

---

## 10. 驗收測試矩陣

### 10.1 Go capability projection

- Server-only profile投影 `FL_SERVER`；
- Client-only profile投影 `FL_CLIENT`；
- combined profile投影 `FL_SERVER_AND_CLIENT`；
- 多 entries與 analytics IDs deterministic sort／dedupe；
- projection不包含 topology role／plan state；
- valid local profile沒有FL capability時回傳空 projection且不 panic；
- existing containing-context identity／URI fields維持不變；
- existing NRF profile registration serialization維持不變。

### 10.2 PyMTLF config matrix

| Mode | Server section | Client section | Expected |
| --- | --- | --- | --- |
| local | absent | absent | valid |
| local | present | absent／present | invalid |
| federated | absent | absent | invalid |
| federated | present | absent | valid Server-only |
| federated | absent | present | valid Client-only |
| federated | present | present | valid combined |
| fl_server／fl_client | any | any | invalid migrated value |
| root／branch／leaf | any | any | invalid role config |

另驗證 federated Server的 `max_active_processes != 1` 被 deterministic拒絕。

### 10.3 Route and lifecycle

- local：Provision／Monitor有，local trainer有，Training routes無；
- Server-only：Provision／Monitor與notification ingress有，Client CRUD不可用；
- Client-only：Client CRUD有，Provision／Monitor與Server notification不可用；
- combined：Client CRUD與Server notification同時可用；
- disabled engine沒有被default settings偷偷建構；
- combined shutdown兩個 engines都停止，timer／executor／HTTP client不殘留；
- retrain intent只送到enabled Server engine；
- local retrain仍送到local coordinator。

### 10.4 Readiness

- configured Server對advertised Server -> ready；
- configured Client對advertised Client -> ready；
- configured combined對advertised combined -> ready；
- configured／advertised任一方向缺 engine -> 503；
- malformed或empty federated projection -> 503；
- containing Go unavailable -> 503，不阻止process listener啟動；
- Go後續可用且一致 -> 下一次probe恢復200；
- artifact probe failure仍維持既有503；
- readiness response有stable process generation與explicit enabled-engine資訊。

### 10.5 Registry invariants

- 第一個 Client create取得provisional slot；
- 相同 `mlCorreId`可加入同一 resource group；
- 不同 `mlCorreId` concurrent create被拒絕；
- create中途失敗會rollback reservation；
- provisional group可bind一次有效 UUIDv4 `planId`；
- conflicting rebind／different role被拒絕；
- Branch record可attach一個lower Server process；
- unrelated Server process被single-active guard拒絕；
- Root／Leaf不接受不符合其state shape的attach；
- terminal但cleanup未完成時slot仍occupied；
- release後新 plan可取得slot；
- retired `planId`不可在相同 process lifetime重用；
- new app／registry instance沒有舊 active或retired state；
- callback lookup miss不建立record；
- registry methods不在lock內執行外部I/O。

### 10.6 Regression

- NWDAF full Go test suite；
- PyMTLF full pytest suite；
- Go formatting／lint依repository policy；
- PyMTLF Ruff；
- existing distributed Server／Client FL tests；
- Slice 1 hierarchy artifact contract tests；
- backend generation reset tests；
- standard inbound／outbound Training route tests。

Documentation-only plan checkpoint可只執行 diff check；production completion不可用doc check取代
上述 repository verification。

---

## 11. Failure mapping

| Failure | Boundary behavior | State effect |
| --- | --- | --- |
| Go capability context unavailable | readiness 503 | 不建立experiment |
| capability projection malformed | readiness 503 | 不建立experiment |
| configured／advertised mismatch | readiness 503 | 不建立experiment |
| disabled engine route被呼叫 | private service unavailable／route unavailable | 不建立resource |
| active slot conflict | existing private overload／conflict mapping | 不建立第二個experiment |
| provisional create rollback | 原create error | reservation同步釋放 |
| stale／conflicting `planId` bind | fail closed | current record不變 |
| PyMTLF restart | new process generation | registry為空；Go清舊routes |
| Go restart | new Go route state | 不要求PyMTLF重建Go routes |
| cleanup error | bounded best effort並記錄 | 到既定終點後terminal release |

本 Slice不新增新的 public `ProblemDetails` cause。Private error exact mapping優先沿用現有
`OVERLOAD`、`SERVICE_NOT_AVAILABLE` 或等價 local convention；若 registry integration需要
新增 externally observable private error contract，必須在實作前於本節補充。

---

## 12. 明確延後的行為

以下全部不屬於 Slice 2：

- Root-only static topology config；
- topology file loading／validation；
- degradation trigger與private management request API；
- Root `planId` creation與manual-restart-required latch integration；
- NRF exact-instance peer discovery／assignment eligibility；
- Branch下載 assignment後解析並建立 lower-tier process；
- Leaf assignment handling；
- Branch artifact republish orchestration；
- preparation result notification與`complete_required` admission；
- `statusReport`／`mLModelInfos` preparation contract correction；
- FedProx local objective與hierarchical rounds；
- upper／lower aggregation與effective sample count；
- terminal artifact cleanup closure與multi-process E2E；
- restart recovery／state persistence／reconciliation；
- arbitrary parallel top-level experiments；
- role config、dynamic topology、partial admission或automatic retry。

Slice 2只保留後續需要的 typed hooks，不提供可配置但尚未實作的假選項。

---

## 13. 風險與控制

### 13.1 把 capability 再做成 PyMTLF config

風險：Go與PyMTLF可配置出互相矛盾的 advertisement。

控制：PyMTLF sections只啟用 engines；standard capability唯一來源是Go profile projection，
並以readiness exact consistency檢查。

### 13.2 將 Branch 做成永久 mode

風險：同一 combined NWDAF無法在不同 plan被指派成不同位置。

控制：config只表達 Server／Client engine presence；role只存在registry record。

### 13.3 把 Go route state 變成 hierarchy database

風險：duplicated state、restart semantics與callback ownership混亂。

控制：Go只維持standard route；`planId`與upper／lower mapping只在PyMTLF memory。

### 13.4 Single-active guard誤擋同一 Branch的兩層工作

風險：Branch attach lower Server時被視為第二個experiment。

控制：先bind `planId`，再以同plan attach Server process；resource group與process IDs仍獨立。

### 13.5 Single-active guard放得太晚

風險：兩個concurrent creates都已啟動background下載後才發現衝突。

控制：Client resource進engine map與排入executor前先取得provisional reservation。

### 13.6 Readiness形成startup cycle

風險：Go等PyMTLF ready，PyMTLF又在lifespan blocking等待Go listener。

控制：PyMTLF listener可啟動；capability只在dynamic readiness probe驗證。Go既有順序是先
完成own listeners，再由availability monitor probe backend。

### 13.7 Lock inversion or shutdown hang

風險：registry lock、Client lock、Server lock與network／executor wait形成deadlock。

控制：registry methods不做I/O、不等待condition；engine integration採短暫claim／snapshot，
shutdown有明確admission fence並沿用app-owned lifecycle。

---

## 14. Decision gates

目前沒有需要重新選擇architecture的open decision。實作只在下列證據出現時停下：

1. actual Go profile snapshot無法在不複製config的前提下產生deterministic capability
   projection；
2. existing standard FL flow證明同一 Client上的不同 `mlCorreId` 必須同時代表同一
   top-level experiment，且目前contract沒有可驗證的grouping identity；
3. Branch lower Server process在建立前沒有任何可信 `planId` binding point，導致只能靠
   NF identity／round／URL推測關係；
4. registry整合必須改變public SBI或Release 18 OpenAPI；
5. lifecycle只能靠持久化或restart recovery才能正確收斂。

遇到上述情況需記錄原假設、直接證據、可行方案與建議，先更新canonical plan再繼續。
單純檔名、private class名稱或test fixture placement不是decision gate。

---

## 15. Review checklist

### 15.1 Capability and config

- [x] 沒有新增永久Root／Branch／Leaf mode；
- [x] Go capability仍是唯一advertisement source；
- [x] combined engine config可用；
- [x] old sample configs與docs同checkpoint遷移；
- [x] invalid matrix完整fail closed。

### 15.2 Boundary ownership

- [x] Go route沒有新增`planId`或hierarchy role；
- [x] PyMTLF沒有直接處理public peer SBI；
- [x] capability projection來自runtime NF profile snapshot；
- [x] registry不取代Client／Server engine standard maps；
- [x] free5GC exemplar claim只限實際比較的lifecycle／package boundary。

### 15.3 Concurrency and lifecycle

- [x] guard在background work前取得；
- [x] Branch pair算一個experiment；
- [x] terminal cleanup前不提早release；
- [x] every timer／executor／HTTP client有owner與stop path；
- [x] registry lock內沒有I/O或blocking wait；
- [x] restart後state為空且不呼叫restore helpers。

### 15.4 Tests and scope

- [x] local／pure Server／pure Client／combined regression都存在；
- [x] readiness unavailable／mismatch／recovery有測試；
- [x] registry正反向invariants有isolated tests；
- [x] Go backend generation reset仍通過；
- [x] Slice 3／4／5 behavior沒有提前混入；
- [x] full verification與skipped checks明確記錄。

### 15.5 Implementation、review 與 verification record

Production commits：

- `NWDAF/`：`b045423 feat(context): expose federated learning capabilities`；
- `PyMTLF/`：
  - `4b7bc00 feat(mtlf): enable capability-based FL engines`；
  - `6b8ff10 feat(mtlf): verify advertised FL capabilities`；
  - `530180b feat(mtlf): add process-scoped FL experiment registry`；
  - `ecd6dee feat(mtlf): integrate FL experiment ownership`；
  - `c688113 refactor(mtlf): align FL engine class names`。

完成後的內部runtime class名稱統一為`FLClientEngine`與`FLServerEngine`。本計畫前段保留的
`FLClientService`／`FLServerOrchestrator`名稱是Slice開始時的baseline記錄；rename未改變
private API、config、`app.state.fl_client`／`fl_server`或engine責任邊界。

Mandatory review檢查完整Slice diff後，直接修復下列in-scope findings：

- shutdown fence擴及既有provisional record的plan bind與Server attach；
- bound Client resource在尚未進入experiment cleanup時拒絕DELETE且保留engine state；
- Client mapping只在provisional rollback或`CLEANING`階段解除；
- 非同步Client create測試改為等待可觀察完成條件，避免依賴executor排程時機。

Follow-up targeted review未發現新的in-scope finding。最終verification：

- `NWDAF/`：`make test`通過；`make build`通過；`make lint`通過，0 issues；
- `PyMTLF/`：`.venv/bin/pytest -q`通過，298 passed、2 skipped；兩項skip皆因本機沒有
  CUDA runtime；
- `PyMTLF/`：`.venv/bin/ruff check .`通過。

---

## 16. 完成條件

Slice 2只有在下列條件全部成立後才算完成：

1. PyMTLF只使用`local|federated` runtime profile，且Server／Client engines由sections存在性
   決定；
2. Server-only、Client-only與Server + Client profiles都通過config、route與lifecycle
   tests；
3. combined process能同時處理Client CRUD與Server notification ingress；
4. Go private context從實際NF profile提供canonical FL capability projection；
5. PyMTLF engine與advertised capability不一致時維持not ready，後續一致時可恢復；
6. shared registry可在Client bundle解析前provisional reserve，並在後續bind `planId`；
7. Branch upper Client resource group與lower Server process可綁定同一experiment；
8. conflicting experiment、stale plan與duplicate attach會fail closed；
9. terminal cleanup與slot release順序明確，且不自動retry／restart；
10. new process不恢復active或retired hierarchy state；
11. Go standard route、backend generation reset與existing distributed FL沒有regression；
12. `NWDAF/`與`PyMTLF/`各自完成full verification、mandatory code review與in-scope
    remediation；
13. production changes按repository分開commit，並記錄exact commands與results。

完成後，下一個production slice是Slice 3：Root initiation、single-active Root coordinator、
static topology loading、NRF exact-instance validation與Branch assignment dispatch。
