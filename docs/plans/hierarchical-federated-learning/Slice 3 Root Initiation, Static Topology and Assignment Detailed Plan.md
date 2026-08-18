# Slice 3：Root Initiation、Static Topology 與 Assignment 詳細計畫

日期：2026-08-18

狀態：Ready for implementation

上層計畫：

- [Hierarchical NWDAF Federated Learning Implementation Plan](./Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)

前置 contracts：

- [Slice 0 Baseline Audit and Contract Freeze Detailed Plan](./Slice%200%20Baseline%20Audit%20and%20Contract%20Freeze%20Detailed%20Plan.md)
- [Slice 1 Hierarchy Bundle Contract and Artifact Primitives Detailed Plan](./Slice%201%20Hierarchy%20Bundle%20Contract%20and%20Artifact%20Primitives%20Detailed%20Plan.md)
- [Slice 2 Capability and Process-scoped Role Foundation Detailed Plan](./Slice%202%20Capability%20and%20Process-scoped%20Role%20Foundation%20Detailed%20Plan.md)

相關文件：

- [Hierarchical NWDAF Federated Learning Proposal Draft](../../proposals/nwdaf/hierarchical-federated-learning/proposal_draft.md)
- [NWDAF Development Policy](../../development_policy.md)

---

## 1. 目的與完成邊界

Slice 3 要讓具備 FL Server capability 的 NWDAF，在 Root PyMTLF 中建立一次新的
hierarchical FL attempt，載入 Root-only static topology，確認被指定節點仍可透過 NRF
解析且具備正確 capability，發布每個 Branch 專屬的 assignment bundle，最後透過既有
standard Training service 向 Branch 建立 upper-tier preparation resources。

本 Slice 完成後，系統必須支援以下兩個 initiation entrypoints：

1. 既有 model accuracy degradation policy 觸發；
2. config 明確啟用時，由 Root PyMTLF private API 接受 operator request。

兩者必須匯入同一個 Root coordinator、共用同一個 single-active guard、產生新的
`planId`，並遵守相同 topology、NRF、artifact、dispatch 與 failure rules。

本 Slice 的成功終點是：

```text
Root request accepted
  -> topology snapshot validated
  -> configured Branch／Leaf identities resolved through NRF
  -> one recipient-specific assignment bundle published per Branch
  -> one standard upper-tier preparation resource created per Branch
  -> Root attempt enters PREPARATION_WAITING
```

本 Slice 不會：

- 實作 Branch 下載 assignment 後向 Leaf 展開 lower-tier preparation；
- 解讀 Branch 回傳的 subordinate preparation result；
- 修正 preparation notification 的 `statusReport`／`mLModelInfos` contract；
- 進行 complete-required admission decision；
- 啟動 federated training rounds 或 aggregation；
- 增加 topology hot reload、automatic retry、resume 或 persistent recovery；
- 將 private initiation API 經過 Go NWDAF；
- 宣稱 private initiation API 是 3GPP OAM API；
- 修改 Release 18 OpenAPI。

以上 deferred behavior 分別屬於 Slice 4 之後。Slice 3 的測試不得藉由模擬成功的 Branch
preparation outcome，偷偷把流程推進到 training。

---

## 2. 已確認且不得在本 Slice 重新決策的事項

本 Slice 直接繼承下列 decisions：

- 所有 Root、Branch、Leaf 都是標準 NWDAF；不存在新的 NF type 或永久 deployment role；
- 是否能被指派為 Root、Branch 或 Leaf，由 NWDAF 已廣告的 FL capability 決定；
- 實際角色只對當次有效 `planId` 成立，由 Root assignment 決定；
- Root 必須有 FL Server；Branch 必須有 `FL_SERVER_AND_CLIENT`；Leaf 至少有 FL Client；
- topology 只配置在 Root，不在 Branch／Leaf 增加角色或 parent config；
- 第一版 topology strategy 只有 `static`，但保留 strategy config field；
- topology file 與 PyMTLF main config 分離，由 main config 引用；
- topology file path 相對於 main config 所在目錄，而不是 process current working directory；
- 第一版 admission 只有 `complete_required`；任何必要 participant 失敗即終止 attempt；
- 失敗後不自動 retry、補選、恢復或重建 state；operator 排除問題後自行開始新 attempt；
- restart 後是全新 process-memory state；不 persistence、resume 或 reconcile；
- 每個 PyMTLF instance 同時間只允許一個 active top-level training experiment；
- `planId` 是 hierarchy attempt identity，不取代各層標準 `mlCorreId`、subscription ID 或
  notification correlation；
- Branch-specific assignment 必須先由 Root 發布成可下載 bundle，再把該 URL 放入
  standard preparation request；
- Branch 後續必須下載、處理並重新發布 Leaf-facing bundle，不可要求 Leaf 全部回 Root
  下載；該行為屬於 Slice 4；
- private initiation API 直接由 PyMTLF 提供，預設關閉，且是否掛載 route 是 config 選項；
- standard `Nnwdaf_MLModelTraining` 是訓練流程中的 service，不是訓練前的人工作業觸發 API；
- 第一版不增加 caller identity、expected Root identity、hierarchy security header 或專用
  trust handshake；
- 第一版 strategy 仍只有 FedProx，且 `proximal_mu` 位於 strategy-specific config，不是
  hierarchy-global 欄位。

若實作需要改變以上任一點，必須先更新 canonical plan。不得以新增 `root` mode、把 endpoint
寫進 topology file、在 Go route state 保存 `planId`，或把 Slice 4 行為提前塞入 Root
coordinator 的方式繞過 decision。

---

## 3. Repository、branch 與責任歸屬

### 3.1 受影響的 repositories

預期 production repository：

- `PyMTLF/`

Contract verification repository：

- `NWDAF/`

Documentation repository：

- `nwdaf-docs/`

撰寫本計畫時的 baseline：

| Repository | Branch | Revision | 狀態 |
| --- | --- | --- | --- |
| `PyMTLF/` | `feat/r18-hierarchical-federated-learning` | `c688113` | clean，Slice 2 與 engine naming alignment 已完成 |
| `NWDAF/` | `feat/r18-hierarchical-federated-learning` | `b045423` | clean |
| `nwdaf-docs/` | `main` | `31b445a` | clean，local commits 尚未 push |

本 Slice 預期不需修改 Go production code。現有 Go private NRF discovery boundary 已能接受
standard-shaped exact-instance query，且 outbound Training resource creation 已由 Slice 0
確認可重用。只有在 focused contract test 證明既有 boundary 無法表達本計畫要求時，才可
擴大 `NWDAF/` production scope，並必須先記錄實際 gap。

本 Slice 不修改：

- `PyAnLF/`；
- `nrf/`；
- `nwdaf-resources/`；
- `resources/` reference tree；
- Release 18 TS／OpenAPI corpus。

### 3.2 Responsibility boundary

| State／behavior | Owner |
| --- | --- |
| registered NWDAF profile 與 FL capability | Go NWDAF／NRF |
| exact-instance NRF query transport | 既有 Go private MTLF boundary |
| discovery response interpretation與eligibility decision | Root PyMTLF hierarchy resolver |
| static topology config loading／canonical snapshot | Root PyMTLF static topology planner |
| request ID、plan ID、attempt state 與 failure latch | Root PyMTLF coordinator |
| model-family selection與degradation intent adaptation | Root PyMTLF monitor integration |
| assignment bundle schema與publication | 既有 PyMTLF hierarchy artifact service |
| standard Server process、participant resource與correlation | PyMTLF FL Server engine |
| public Training subscription／callback forwarding | Go NWDAF |
| private initiation request／status resource | Root PyMTLF private API |
| instance-level single-active arbitration | 既有 PyMTLF shared experiment registry |

Root coordinator 負責「這次要訓練誰、目前走到哪一步」；FL Server engine 負責「如何建立並
追蹤 standard Server process resources」。兩者不得各自建立第二份 participant、correlation
或 cleanup ownership。

---

## 4. 已確認的實作基線與 gap

### 4.1 Config loading

`src/py_mtlf/config.py` 目前以 strict Pydantic settings 解析 YAML，但 `load_settings(path)`
只把 YAML 內容交給 `Settings.model_validate()`，沒有保留 main config directory。

因此 topology path resolution 必須在 `load_settings()` boundary 完成：

1. 將 caller 傳入的 main config path 正規化；
2. 以 main config parent 解析 relative `topology.config_file`；
3. 將解析後的 absolute path 交給 runtime settings；
4. 不在 coordinator 或 planner 使用 CWD 猜測來源。

若測試直接建構 `Settings`，必須使用 absolute topology path；不得在 model validator 偷渡
環境相依的 CWD 行為。

### 4.2 Existing hierarchy artifacts

Slice 1 已提供 strict、immutable hierarchy models 與 `HierarchyArtifactService`：

- canonical UUIDv4 `planId`；
- Branch／Leaf assignment metadata；
- FedProx-only strategy contract；
- `complete_required` admission；
- digest-pinned bundle publication；
- recipient-specific Branch assignment publication；
- later Branch republish／preparation-result primitives。

Slice 3 必須直接呼叫既有 publication API，不得自行拼 JSON manifest、另外定義 plan schema
或繞過 workspace lifecycle。

### 4.3 Existing experiment registry

Slice 2 shared registry 已提供：

- `reserve_root(plan_id)`；
- `attach_server(reservation_id, plan_id, process_id)`；
- single-active arbitration；
- terminal、cleanup、release 與 retired identity rules；
- shutdown fence 與 fresh-process state。

Root coordinator 必須先 reserve Root attempt，再建立獨立的 Server process identity 並 attach。
`planId` 不可直接重用為 `process_id` 或 `mlCorreId`。

### 4.4 Existing FL Server engine

目前 `FLServerEngine` 已擁有 standard Server process、participant subscriptions、notification
correlations、cleanup 與 outbound request behavior。但既有 flat-FL `_run` 會自行從 active
scopes 搜尋 Clients、執行 preparation 並繼續 training rounds，不適合直接當成 hierarchy
Root orchestration。

本 Slice 必須增加一個 narrowly-scoped hierarchy preparation seam，讓 coordinator 提供已
驗證的 Branch targets 與各自 assignment URL，由同一個 engine 建立 process 和 standard
resources，並停在 `PREPARATION_WAITING`。不得複製 engine 內部 maps 或再做第二個 Server
orchestrator。

### 4.5 Existing degradation trigger

`AccuracyPolicy.observe()` 目前產生 `RetrainIntent`，`_dispatch_retrain_intents()` 在 federated
Server profile 下直接交給 `FLServerEngine.accept_policy_intents()`。

整合後：

- 有有效 static topology coordinator 時，degradation intent 交給 Root coordinator；
- 沒有 Root topology config 的既有 flat-FL Server，維持原本 dispatch behavior；
- 不得要求所有 federated Server deployment 都配置 hierarchy topology；
- degradation 重複發生於 active attempt 或 failed latch 時，不建立新 plan。

### 4.6 Existing Go boundaries

現有 Go private endpoint `GET /internal/v1/nrf/nf-instances` 已把 query 傳給 generated NRF
discovery consumer，並可表達：

- target NF type `NWDAF`；
- exact target NF instance ID；
- service name `nnwdaf-mlmodeltraining`；
- analytics information；
- FL capability type。

現有 outbound Training private boundary 也已能讓 PyMTLF 以 selected target 建立 peer resource。
因此 Slice 3 的預設工作是補 PyMTLF resolver 與 focused regression tests，不新增 hierarchy-
specific Go endpoint，也不把 peer API root 寫死在 topology config。

`nrf/internal/sbi/processor/nf_discovery_extended.go` 的既有 matching 已明確把
`FL_SERVER_AND_CLIENT` 視為可符合 `FL_CLIENT` query，且已有 combined-capability test。
因此 Leaf query 可以保留 FL Client capability constraint，不需要為 combined profile 移除
constraint 或新增 fallback query。

### 4.7 Known preparation result gap

目前 Server callback path 對 preparation outcome 的 `statusReport`／`mLModelInfos` 解讀仍是
Slice 4 要修正的 contract。Slice 3 只驗證 outbound resources 已建立且 correlations 已註冊；
不得把目前 callback behavior 誤當成完成整個 hierarchy preparation 的證據。

---

## 5. Target configuration 與 topology contract

### 5.1 Main config

Root-capable deployment 可選擇配置：

```yaml
federated_learning:
  server:
    # existing FL Server settings
  strategy:
    algorithm:
      name: fedprox
      proximal_mu: 0.01
    participant_selection: all
    waiting_policy: all
    aggregation: sample_weighted
  topology:
    strategy: static
    config_file: "./topology/hierarchical-topology.yaml"
  training_trigger:
    private_api:
      enabled: false
```

Rules：

- `topology` 整個 section 是 optional；未配置時保留 flat-FL behavior；
- 配置 `topology` 時必須明確配置 `strategy`，避免 Root 靜默選擇未確認的
  `proximal_mu`；
- 第一版 strategy 只接受 `fedprox`、positive finite `proximal_mu`、`all` participant
  selection、`all` waiting 與 `sample_weighted` aggregation；
- top-level `proximal_mu`、FedAvg、generic parameters map 與 unknown strategy values 拒絕；
- 第一版 `strategy` 只接受 `static`；欄位保留供未來增加其他 planner；
- 配置 `topology` 時必須同時啟用 FL Server engine；
- HFL-enabled Server 必須維持既有 `max_active_processes == 1` validation；
- `config_file` 必須存在、可讀並在 application startup 完整驗證；
- `private_api.enabled` 預設 `false`；
- private API 只在 FL Server 與有效 Root topology 同時存在時才可啟用；
- invalid combination 應在 startup fail fast，不建立半可用 application；
- config 不增加 `role: root`、Branch URL、Leaf URL 或 caller identity。

### 5.2 Static topology file

第一版 contract：

```yaml
version: 1
admission:
  mode: complete_required
branches:
  - nf_instance_id: "11111111-1111-4111-8111-111111111111"
    leaves:
      - nf_instance_id: "22222222-2222-4222-8222-222222222222"
      - nf_instance_id: "33333333-3333-4333-8333-333333333333"
```

Topology file 只描述 identity、tree shape 與 admission，不可包含：

- Root／Branch／Leaf endpoint；
- `planId`、`mlCorreId`、subscription ID 或 callback URI；
- model family、model URL 或 artifact digest；
- training strategy；
- permanent node role；
- runtime state。

Validation rules：

- `version` 必須精確為 `1`；
- `admission.mode` 必須精確為 `complete_required`；
- 至少一個 Branch；每個 Branch 至少一個 Leaf；
- 所有 IDs 必須是 canonical UUIDv4；
- Branch IDs 不可重複；Leaf IDs 在整棵 tree 中不可重複；
- 同一 ID 不可同時是 Branch 與 Leaf；
- containing Root NF instance ID 不可出現在 assignment tree；
- unknown fields 一律拒絕；
- parser 將 branches 與 leaves 依 canonical ID 排序後形成 immutable snapshot，使 bundle 與
  測試結果不受 YAML ordering 影響。

檔案本身可在 startup 前修改，但 process 啟動後不 hot reload。新內容只在下一次 process
start 生效。

### 5.3 Planner interface

建立 implementation-neutral planner boundary，概念介面為：

```python
class TopologyPlanner(Protocol):
    def build(self, *, root_nf_instance_id: UUID) -> TopologyAssignment: ...
```

第一版只有 `StaticTopologyPlanner`。它持有 startup 時解析完成的 immutable config，並在
`build()` 時完成需要 Root identity 才能判斷的 collision check。

Planner 不負責 NRF discovery、artifact publication、request state 或 Server process creation。

---

## 6. Root initiation 與 state model

### 6.1 Typed initiation inputs

Root coordinator 接受兩種 adapter 轉成的 typed input：

| Source | Required input |
| --- | --- |
| degradation | internally generated request ID、affected model family、active scope snapshot |
| private API | caller-provided request ID、selected model family |

Private trigger 不應偽造 `RetrainIntent`。Coordinator 可使用共同的 `RootInitiation` model，
degradation adapter 再附帶 policy context。Manual request 只需要選定目前 model catalog 中已存在
且可做 HFL 的 family；active monitor scope 可為空。

開始前必須確認：

- model family 存在；
- current base model 可取得；
- analytics event 是第一版已支援的 `UE_COMMUNICATION`；
- interoperability 資訊存在；
- Root containing-NWDAF identity 與 FL Server capability readiness 已取得。

### 6.2 Identity rules

- `requestId`：operator／trigger request identity，用於 idempotency；
- `planId`：每次真正建立的新 hierarchy attempt UUIDv4；
- Server `processId`／`mlCorreId`：由 FL Server engine 另行產生；
- per-Branch `notifCorreId` 與 subscription ID：沿用 standard Server engine behavior。

不得讓上述 identities 互相替代。

Coordinator 產生的 `planId` 若與 active 或 retired registry identity 重複，必須拒絕該次
建立且不得沿用既有 plan state。這是 identity collision，不是 idempotent request replay。

### 6.3 Request states

第一版 private status resource 至少暴露：

```text
ACCEPTED
VALIDATING
DISPATCHING
PREPARATION_WAITING
FAILED
```

狀態含義：

- `ACCEPTED`：single-active reservation 已成功，尚未宣稱 topology 有效；
- `VALIDATING`：建立 topology snapshot、取得模型並執行 exact-instance discovery；
- `DISPATCHING`：發布 assignments 並建立 standard upper-tier resources；
- `PREPARATION_WAITING`：所有必要 Branch resource 已建立，等待 Slice 4 處理 outcomes；
- `FAILED`：terminal failure，保留 request status 供查詢，但已完成可清理 resources 的 cleanup。

Slice 4 可在不改變既有語意下增加後續 states。不得把 HTTP `202` 解讀成 topology 已完成。

### 6.4 Single-active、idempotency 與 failure latch

- 同一 `requestId` 搭配相同 request content 重送：回傳既有 resource，不建立新 `planId`；
- 同一 `requestId` 搭配不同 model family：`409 Conflict`；
- 不同 request 在 active attempt 存在時：`409 Conflict`；
- attempt `FAILED` 後，operator 使用新的 `requestId` 可明確開始全新 attempt；
- degradation trigger 在 active attempt 或 failed latch 存在時不建立新 attempt；
- failure latch 只防止 policy 自動重複啟動，不是 persistent recovery state；
- 新的 operator request 成功取得 reservation 時，開始新 attempt 並取代先前 failure latch；
- restart 後 memory state 自然清空。

Coordinator 必須在做 discovery、publication 或 outbound network action 前先取得 registry
reservation，避免兩個 concurrent initiations 都產生外部副作用。

---

## 7. Exact-instance NRF resolution 與 eligibility

### 7.1 Query contract

對 static topology 中每一個 configured node，Root 都必須進行 exact-instance NRF query，而
不是把 ID 直接拼成 peer URL。

Branch query 至少包含：

- target NF type：`NWDAF`；
- exact target NF instance ID；
- service：`nnwdaf-mlmodeltraining`；
- analytics：`UE_COMMUNICATION`；
- capability：`FL_SERVER_AND_CLIENT`。

Leaf query 至少使用 exact target ID、Training service、analytics 與 FL Client capability
constraint；response interpretation 必須接受 `FL_CLIENT` 或 `FL_SERVER_AND_CLIENT`，因為具備
combined capability 的 NWDAF 仍可被指派為 Leaf。

現有 NRF matching 已有 `FL_CLIENT` query 接受 combined profile 的 test；Slice 3 需從 PyMTLF
到 Go private boundary 補上 focused contract verification，避免 query serialization 導致這個
語意在跨 repository boundary 遺失。

### 7.2 Response validation

Resolver 不可只因 HTTP success 就接受 target。它必須確認：

- response 中只有／可唯一選出 configured exact NF instance ID；
- NF status 是 `REGISTERED`；
- profile 具有所需 FL capability；
- ML analytics list 支援本次 analytics ID；
- `nnwdaf-mlmodeltraining` service 存在且 service status 可用；
- API root／service endpoint 可由既有 discovery result 唯一決定；
- malformed、duplicate 或 contradictory profile 一律失敗，不猜測 endpoint。

### 7.3 Discovery snapshot lifecycle

Root 在一次 attempt 的 validation phase 驗證整棵 configured tree，包括所有 Branch 與 Leaf。
這裡對 Leaf 的驗證是 Root admission 前檢查；Slice 4 中 Branch 仍必須在實際 lower-tier
dispatch 前重新解析被指派的 Leaf。

本次 Root discovery results 形成 immutable attempt snapshot：

- 不在同一 attempt 中自動改用其他 instance；
- 不因 NRF 後續變化 hot-swap endpoint；
- async discovery result 必須攜帶／對應目前 request 與 plan；若 attempt 已 terminal、被清理或
  identity 不符，該結果視為 stale 並丟棄，不得繼續 publication／dispatch；
- 若 dispatch 前已知 target discovery data 無效或建立 resource 失敗，整個 attempt 失敗；
- 新 attempt 重新查詢 NRF。

---

## 8. Assignment publication 與 upper-tier preparation

### 8.1 End-to-end ownership flow

```text
degradation／private request
  -> RootCoordinator acquires single-active reservation
  -> StaticTopologyPlanner creates canonical tree snapshot
  -> HierarchyNodeResolver validates every configured node via NRF
  -> RootCoordinator loads selected base model／training contract
  -> HierarchyArtifactService publishes one assignment bundle per Branch
  -> FLServerEngine creates one hierarchy Server process
  -> registry attaches that process to the Root reservation
  -> FLServerEngine creates one standard preparation resource per Branch
  -> Root request enters PREPARATION_WAITING
```

### 8.2 Recipient-specific bundle

每個 Branch 取得不同 bundle URL。Bundle 必須包含：

- 相同 `planId`；
- publisher：Root NF instance ID；
- recipient：該 Branch NF instance ID；
- 僅屬於該 Branch 的 assigned Leaf IDs；
- `complete_required` admission；
- Slice 1 已固定的 FedProx strategy contract；
- valid base model artifact 與 digest metadata。

Root 不發布一個包含整棵 topology、讓所有 Branch 共用的 bundle。Topology file 也不直接
暴露為下載 artifact。

### 8.3 FL Server engine seam

在既有 `FLServerEngine` 增加 focused method，概念輸入為：

- plan／registry reservation identities；
- selected model family與base model descriptor；
- canonical list of resolved Branch targets；
- per-Branch assignment bundle URL；
- active scope snapshot（可為空）。

Engine 負責：

- 產生獨立 Server process UUID／standard `mlCorreId`；
- 將 process attach 到既有 Root reservation；
- 為每個 Branch 產生獨立 notification correlation；
- 透過既有 Go private outbound Training boundary 建立 standard preparation subscription；
- 保存 returned locations、participant state 與 cleanup ownership；
- 全數建立完成後停在 preparation waiting state。

Coordinator 不得直接操作 engine private maps。Engine 也不得自行重新規劃 topology 或從
active scopes 做 flat discovery。

### 8.4 Standard preparation payload

對每個 Branch 的 request 沿用現有 standard Training contract：

- `mLPreFlag: true` 表達 preparation；
- 本次 model analytics event、filter、target 與 interoperability requirements；
- shared upper-tier standard `mlCorreId`；
- per-Branch `notifCorreId`；
- `mLModelUrl` 指向該 Branch 的 assignment bundle；
- selected target 由 exact NRF result 交給 Go private boundary。

不在 standard request 增加 vendor `planId` header；Branch 後續從 bundle 取得 `planId`。

### 8.5 Partial dispatch failure

若第 N 個 Branch resource 建立失敗：

1. FL Server engine 對已建立的前 N-1 個 outbound resources 執行 bounded best-effort cleanup；
2. 移除／terminalize本地correlations與Server process；
3. coordinator 將 request 標為 `FAILED`；
4. registry 依 terminal -> cleaning -> released lifecycle 結束 active reservation；
5. release此次published artifacts；
6. 設定 policy failure latch，不自動 retry。

Cleanup failure 必須被記錄，但不把 attempt 重新標為 active，也不無限阻塞 shutdown。

---

## 9. PyMTLF private initiation API

### 9.1 Route gating

第一版採用：

```text
POST /internal/v1/hierarchical-fl/training-requests
GET  /internal/v1/hierarchical-fl/training-requests/{requestId}
```

Router 只在 `training_trigger.private_api.enabled: true` 時掛載。關閉時 route 應為 `404`，而
不是掛載後回 `403`。Route 不經過 Go NWDAF，也不列入 standard SBI advertisement。

本 Slice 不新增認證機制；部署層仍須把 PyMTLF internal surface 限制在既有 private network
boundary。這是 current private API convention，不是對外安全保證。

### 9.2 Create request

Request body：

```json
{
  "requestId": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
  "modelFamilyId": "ue-communication-default"
}
```

Rules：

- `requestId` 必須是 canonical UUIDv4，由 caller 提供以支援安全重送；
- `modelFamilyId` 必須精確匹配目前 catalog family key；
- unknown fields 拒絕；
- request 不接受 topology path、node list、algorithm override 或 arbitrary model URL。

成功接受回傳 `202 Accepted`、`Location` header 與 status representation：

```json
{
  "requestId": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
  "planId": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb",
  "modelFamilyId": "ue-communication-default",
  "state": "ACCEPTED"
}
```

同一 request idempotent replay 固定回 `202 Accepted`，並回同一 `Location`、`planId` 與
current representation。

### 9.3 Status request

`GET` 回傳目前 state；失敗時額外包含 stable machine-readable cause，不暴露 traceback、完整
peer response 或 secrets。未知 `requestId` 回 `404` ProblemDetails。

第一版沒有 list、DELETE、cancel、retry 或 reset endpoint。失敗後以新的 create request 啟動
新的 attempt。

### 9.4 Error mapping

| Condition | HTTP／result |
| --- | --- |
| invalid UUID／unknown field／empty family | `400` ProblemDetails |
| unknown model family | `404` ProblemDetails |
| same request ID with conflicting body | `409` ProblemDetails |
| different request while active | `409` ProblemDetails |
| Root capability／topology runtime unavailable | `503` ProblemDetails |
| async validation／discovery／dispatch failure | create remains accepted；status becomes `FAILED` |

FastAPI validation handler 必須將 `/internal/v1/hierarchical-fl/` 加入既有 private
ProblemDetails normalization，而不是對此 route 洩漏預設 `422` shape。

---

## 10. Failure、retry 與 restart rules

| Failure point | Required behavior |
| --- | --- |
| invalid／missing topology at startup | application startup fails；不掛載 hierarchy route |
| Root identity／capability mismatch | request fails terminally；不 dispatch |
| any configured Branch／Leaf discovery failure | whole attempt fails；不接受 partial tree |
| model family／base artifact unavailable | whole attempt fails；不 dispatch |
| assignment publication failure | release已發布artifacts；whole attempt fails |
| any upper resource creation failure | cleanup已建立resources與artifacts；whole attempt fails |
| repeated degradation while active／latched | no new plan；record observable reason |
| operator new request after terminal failure | create fresh request／plan after old cleanup |
| PyMTLF restart | fresh registry、requests、latch與coordinator state |

第一版不做 background retry、replacement participant、backoff scheduler 或 durable job queue。
Failure cause 應在 logs 與 private status 中足以定位哪一個 phase／configured NF identity 失敗，
但不得把整份 NRF profile 或 artifact content 塞進 status response。

---

## 11. 預計 production 變更範圍

### 11.1 `PyMTLF/`

預期新增／修改：

- `src/py_mtlf/config.py`
  - typed HFL strategy、topology／training-trigger strict settings；
  - main-config-relative path resolution；
  - invalid combination startup validation。
- `src/py_mtlf/core/fl_topology.py`（new）
  - strict topology file models；
  - immutable assignment snapshot；
  - `TopologyPlanner`／`StaticTopologyPlanner`。
- `src/py_mtlf/core/fl_hierarchy_discovery.py`（new）
  - exact-instance Branch／Leaf resolver；
  - profile／capability／service validation。
- `src/py_mtlf/core/fl_root.py`（new）
  - initiation request records；
  - Root coordinator state machine；
  - idempotency、failure latch與cleanup sequencing。
- `src/py_mtlf/core/fl_server.py`
  - hierarchy preparation seam；
  - process attach／participant dispatch／bounded rollback。
- `src/py_mtlf/api/hierarchical_fl.py`（new）
  - config-gated create／status routes。
- `src/py_mtlf/api/ml_model_monitor.py`
  - degradation intent routing to Root coordinator；
  - flat-FL fallback preservation。
- `src/py_mtlf/app.py`
  - startup planner load；
  - coordinator construction／state exposure；
  - conditional router mount；
  - close ordering。
- sample config／topology files
  - provide one Root-capable deployment example；
  - do not turn topology into required config for all federated deployments。
- focused unit／integration tests。

檔名可依實際 module cohesion 微調，但責任分層不得退化成單一巨大 coordinator file。

### 11.2 `NWDAF/`

預設只有 verification：

- exact target NF instance query 正確傳遞；
- Branch combined capability query可表達；
- selected target 的 outbound Training resource path 沿用既有 behavior。

若沒有 contract gap，不建立 empty production commit。若只需補 regression test，應以獨立 Go
repository commit 提交。

---

## 12. 實作順序與 checkpoints

### Checkpoint 1：Config 與 static topology contract

1. 先寫 topology/config negative tests；
2. 實作 main-config-relative path resolution；
3. 實作 strict file models 與 canonical ordering；
4. 實作 planner interface／static implementation；
5. 驗證 flat-FL config 完全不受影響。

完成條件：startup validation 與 immutable topology snapshot 可獨立測試，不需要 network。

### Checkpoint 2：Exact-instance hierarchy resolver

1. Characterize existing Go private discovery response；
2. 實作 Branch／Leaf query construction與response validation；
3. 覆蓋 combined-capability Leaf matching behavior；
4. 固定 deterministic endpoint selection與malformed-response failures。

完成條件：給定每種 profile fixture，都能明確判斷 eligible target 或 stable failure cause。

### Checkpoint 3：Root request state 與 private API

1. 建立 typed initiation與request record；
2. 先 reserve registry，再非同步執行 validation；
3. 實作 idempotent replay、conflict、failure latch；
4. conditional mount create／status router；
5. 統一 ProblemDetails validation behavior。

完成條件：尚未接 outbound dispatch 時，API concurrency 與state transitions已可完整測試。

### Checkpoint 4：Assignment publication 與 hierarchy preparation seam

1. 以 Slice 1 service 發布 recipient-specific bundles；
2. 在 FL Server engine 增加 focused hierarchy entrypoint；
3. 建立獨立 Server process並attach registry；
4. 對所有 Branch 建立 standard resources；
5. 實作 partial-dispatch rollback與artifact release；
6. 成功後停在 `PREPARATION_WAITING`。

完成條件：多 Branch flow 可證明 bundle isolation、standard correlation ownership 與 rollback。

### Checkpoint 5：Degradation integration 與 lifecycle regression

1. hierarchy-enabled Server 將 degradation intent送入Root coordinator；
2. hierarchy-disabled Server維持既有flat behavior；
3. 驗證 active／failed latch不會重複建立plan；
4. 驗證 close ordering、bounded cleanup與fresh restart state。

### Checkpoint 6：不中斷 code review、修復與針對性重驗

實作完成後不中斷地執行 code review。若 finding 不改變已確認架構、不跨 repository boundary、
不影響上層 plan，直接修復並針對 finding 重驗；只有 decision 變更或重大 scope expansion 才
停止並回報。

Review 至少檢查：

- config path 是否仍可能受 CWD 影響；
- coordinator／engine 是否出現雙重 state owner；
- network side effect 是否發生在 reservation 前；
- failure path 是否遺漏 artifact／subscription cleanup；
- exact-instance response 是否仍可能選錯 profile；
- Slice 4 callback semantics 是否被意外提前或誤判完成。

---

## 13. 驗收測試矩陣

### 13.1 Config／topology

- topology omitted preserves flat-FL config；
- topology requires an explicit valid first-version strategy；
- invalid／zero／non-finite `proximal_mu` and unsupported strategy options fail；
- relative path is resolved from main config directory under different CWD；
- absolute path remains stable；
- missing／unreadable file fails startup；
- unknown strategy、version、admission、fields fail；
- empty branches／leaves fail；
- invalid UUID、duplicate Branch、duplicate global Leaf、cross-role collision fail；
- containing Root ID collision fails；
- input ordering produces same canonical snapshot；
- topology without Server engine fails；
- HFL-enabled Server with `max_active_processes != 1` fails；
- private API enabled without valid topology fails；
- private API default is disabled。

### 13.2 Discovery

- exact Branch with combined capability resolves；
- exact Leaf with client capability resolves；
- combined-capability node may resolve as Leaf；
- wrong returned NF ID fails；
- deregistered／suspended target fails；
- missing analytics、capability、service或API root fails；
- duplicate／malformed matching profiles fail deterministically；
- one failed configured Leaf fails the whole complete-required attempt；
- Root validates all configured nodes before publication／dispatch。

### 13.3 Private API and state

- route absent when disabled；
- valid create returns `202`、Location、requestId與planId；
- status returns current state；
- same request／same body returns same plan；
- same request／different body returns `409`；
- different concurrent request returns `409`；
- malformed input returns local `400` ProblemDetails；
- unknown family returns `404`；
- async failure becomes queryable `FAILED`；
- new request after terminal cleanup creates new plan；
- generated duplicate／retired plan identity is rejected without reusing old state；
- no list／delete／cancel routes exist。

### 13.4 Assignment and dispatch

- one unique Branch bundle per Branch；
- each bundle contains only that Branch’s leaves；
- bundles share planId but have correct publisher／recipient；
- strategy remains Slice 1 FedProx contract；
- standard `mLModelUrl` is the Branch-specific bundle URL；
- one Server process uses distinct process ID from planId；
- process is attached to registry before participant callbacks matter；
- one independent correlation／resource per Branch；
- successful dispatch ends at `PREPARATION_WAITING`；
- no training round starts；
- Nth Branch failure cleans first N-1 resources and all attempt artifacts；
- cleanup failure remains bounded and attempt remains terminal。

### 13.5 Trigger and lifecycle

- degradation in hierarchy-enabled deployment creates one Root attempt；
- repeated degradation while active creates none；
- degradation after failure latch creates none；
- operator new request after failure is allowed；
- hierarchy-disabled federated Server retains flat-FL dispatch；
- app close stops accepting requests before engine cleanup；
- new process starts with no request、plan、latch或registry state。

### 13.6 Regression commands

PyMTLF implementation完成後至少執行：

```bash
.venv/bin/ruff check .
.venv/bin/pytest -q
```

若 `NWDAF/` 有任何變更，依 repository policy 執行 focused tests後，再執行：

```bash
gofmt -w <changed-go-files>
go test ./...
make lint
make build
```

若 repository 的既有 `make test` 是 canonical full suite，則同時執行並記錄結果。測試或腳本
執行依 workspace policy 使用 elevated permission。

---

## 14. 風險與控制

### 14.1 Relative path 在測試通過、deployment 失敗

控制：只在 `load_settings(main_config_path)` 解析相對路徑；測試刻意切換 CWD。

### 14.2 Static assignment 被誤當成 discovery result

控制：topology 只提供 exact identity；endpoint與eligibility每個 attempt都必須由NRF驗證。

### 14.3 Root coordinator 複製 FL Server state

控制：coordinator只保存request／plan／snapshot；process、participants與correlations仍由engine
唯一擁有。

### 14.4 Single-active check 太晚

控制：在任何 artifact 或 network side effect 前先 reserve registry，並以 concurrent tests
驗證只有一個 winner。

### 14.5 Dispatch 部分成功造成 orphan subscriptions

控制：engine 回傳結構化 dispatch progress，failure path 由同一 owner 執行 bounded rollback。

### 14.6 Private API 被誤稱 OAM／standard API

控制：路徑維持 `/internal/v1`、default disabled、不經 Go、不寫入 NRF service profile，文件只
稱 operator-facing private trigger。

### 14.7 提前依賴錯誤的 preparation callback 語意

控制：本 Slice 停在 waiting，callback outcome/admission tests 明確留給 Slice 4。

### 14.8 Combined capability contract 在跨 repository boundary 遺失

控制：保留既有 NRF combined-capability unit test，並在 PyMTLF／Go private boundary 增加
focused verification；不得用移除所有 capability checks或hardcoded endpoint繞過。

---

## 15. Decision gates

本計畫沒有待確認的 architecture gate。以下 implementation details 先固定，以避免實作期間
產生多套 contract：

| Item | Slice 3 decision |
| --- | --- |
| private create path | `/internal/v1/hierarchical-fl/training-requests` |
| private status path | `/internal/v1/hierarchical-fl/training-requests/{requestId}` |
| manual selector | existing `modelFamilyId` |
| idempotency key | caller-provided canonical UUIDv4 `requestId` |
| cancellation | V1 不提供 |
| retry | new request ID creates a fresh attempt after terminal cleanup |
| topology ordering | loader canonicalizes；不要求 YAML author 手動排序 |
| Go changes | verify first；只在 focused test 證明 gap 時修改 |
| Slice end state | `PREPARATION_WAITING` |

若 review 發現以下情況，必須停止實作並回到 plan decision：

- 現有 standard Training payload無法攜帶Branch-specific bundle URL；
- exact-instance NRF boundary無法在不改public SBI contract下取得所需profile；
- existing registry無法在不破壞single-active invariant下attach Root Server process；
- complete-required failure需要新的跨process durable state；
- implementation被迫新增永久Root／Branch／Leaf role config。

一般檔名調整、internal method signature、status cause enum或test fixture shape不構成decision gate。

---

## 16. Review checklist

### 16.1 Contract and configuration

- [ ] topology path relative to main config, not CWD
- [ ] no permanent role config
- [ ] no endpoints or runtime IDs in topology
- [ ] strategy is typed and `proximal_mu` only exists inside FedProx algorithm block
- [ ] private route absent by default
- [ ] flat-FL config remains valid
- [ ] all strict config models reject unknown fields

### 16.2 Identity and ownership

- [ ] requestId、planId、processId、mlCorreId與notifCorreId保持分離
- [ ] registry reserved before external side effects
- [ ] Root coordinator does not own Server correlation maps
- [ ] FL Server engine does not re-plan static topology
- [ ] Branch/Leaf role exists only inside active plan snapshot

### 16.3 Discovery and dispatch

- [ ] every configured identity is exact-resolved
- [ ] profile identity/status/capability/analytics/service are validated
- [ ] no hardcoded peer endpoint
- [ ] every Branch receives isolated assignment bundle
- [ ] all required Branch resources exist before waiting state
- [ ] partial dispatch rolls back already-created resources

### 16.4 Failure and lifecycle

- [ ] no automatic retry after required participant failure
- [ ] degradation respects active and failed latch
- [ ] explicit new request can start after terminal cleanup
- [ ] restart produces fresh memory state
- [ ] shutdown cleanup is bounded
- [ ] status errors do not expose sensitive payloads

### 16.5 Slice boundary

- [ ] no lower-tier Branch orchestration
- [ ] no preparation result interpretation
- [ ] no admission or round start
- [ ] no Release 18 OpenAPI change
- [ ] no unnecessary Go production change

### 16.6 Implementation record

完成後在本文件補記：

- production commits per repository；
- focused與full verification commands／results；
- code review findings與remediation commits；
- actual changed-file scope；
- 任何已核准的plan deviation。

---

## 17. 完成條件

Slice 3 只有在以下條件全部成立後才能標為 `Completed`：

1. Root topology config 可從 main-config-relative file strict load，且 startup validation 完整；
2. first-version typed strategy config 被驗證並直接形成 Slice 1 bundle strategy contract；
3. static planner 產生 deterministic immutable assignment snapshot；
4. 所有 configured Branch／Leaf identities 都經 exact-instance NRF resolution 與 eligibility
   validation；
5. degradation 與 enabled private API 共用同一 Root coordinator；
6. private request具備single-active、idempotent replay、conflict與queryable terminal failure；
7. 每個 Branch 都取得recipient-specific assignment bundle；
8. 同一 FL Server engine建立並擁有所有upper-tier standard preparation resources；
9. process identity 與 plan identity 分離且正確 attach shared registry；
10. 成功流程停在 `PREPARATION_WAITING`，沒有誤啟動 round；
11. 任一 required validation／publication／dispatch failure 都清理本次資源並終止 attempt；
12. hierarchy-disabled flat-FL behavior、combined Server／Client lifecycle與既有測試均無 regression；
13. implementation完成後已不中斷執行code review、必要修復與針對性重驗；
14. 各 repository changes 分別 commit，且本文件補齊實作與驗證紀錄。
