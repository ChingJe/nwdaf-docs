# Model Bundle Metadata to Protocol Schema Mapping

日期：2026-09-02

狀態：Ready for User Review／欄位與 production cutover owner 已完成初步盤點

相關文件：

- [Protocol Extension Implementation Plan](./Hierarchical%20NWDAF%20FL%20Protocol%20Extension%20Implementation%20Plan.md)
- [Protocol Implementation Current-State Inventory](./Protocol%20Implementation%20Current-State%20Inventory.md)
- [Candidate OpenAPI artifact](../../../design/hierarchical-federated-learning/candidate_openapi.yaml)

---

## 1. 目的與邊界

本文件確認現有 HFL implementation 放在 model bundle 中的 hierarchy control
metadata，應如何遷移到新的 `Nnwdaf_MLModelTraining` candidate schema。

盤點範圍包括：

- static topology 產生的 Root→Branch／Branch→Leaf assignment；
- `HIERARCHY_ASSIGNMENT.hierarchy_metadata`；
- `HIERARCHY_PREPARATION_RESULT.hierarchy_metadata`；
- 目前與 hierarchy instruction 混在 round artifact 中的 local-work instruction。

Model bytes、preprocessing artifact、digest、sample count、dataset evidence、aggregation
provenance 與 validation evidence 不屬於 topology protocol instruction，仍由 model／
result artifact 負責。本次不把所有 artifact metadata 搬入 SBI message。

---

## 2. 遷移原則

每個舊欄位只採以下一種處理方式：

1. **搬到 protocol**：跨 NWDAF 的 topology、policy、strategy 或 reporting
   instruction／result。
2. **保留在 artifact**：模型內容、完整性、訓練結果或可驗證 evidence。
3. **改為 local state**：subscription resource、route、workspace 或 execution
   lifecycle 所需，但不需要跨 peer 傳遞的 bookkeeping。
4. **移除**：可由 operation、subscription edge 或其他欄位推導，或只是舊格式為了
   multiplex 多種 bundle message 而存在的重複資訊。

完成 migration 後，protocol message 是 orchestration instruction 的 authoritative
source。Artifact 中即使保留 correlation／round provenance，也只能用來驗證 artifact
與 message 是否一致，不能成為第二套 topology 或 policy control source。

---

## 3. Assignment metadata 對照

### 3.1 Common metadata

| 舊欄位 | 現有用途 | 新位置／處理 | 結論 |
| --- | --- | --- | --- |
| `bundle_schema_version` | 驗證 model bundle container 格式 | 繼續由 artifact contract 使用 | 保留在 artifact，不搬到 protocol |
| `file_digests` | 驗證 model／preprocessing files 完整性 | 繼續由 artifact contract 使用 | 保留在 artifact，不搬到 protocol |
| `artifact_role: HIERARCHY_ASSIGNMENT` | 讓下載端判斷 bundle 是 hierarchy assignment | Create／PUT／PATCH 中出現 `x-flTopology` 已能表示這是 topology instruction | 移除 assignment-specific artifact role；model bundle 回到一般模型 artifact |
| `contract_version` | 驗證舊 `hierarchy_metadata` 格式 | Candidate API schema／negotiated `HierarchicalFLOrch` 定義 wire contract | 不搬入 topology node；local parser version 若仍需要，只留在 implementation |
| `message_type` | 區分 `BRANCH_ASSIGNMENT` 與 `LEAF_ASSIGNMENT` | 每段 subscription 都使用相同的 role-neutral `x-flTopology`；接收者由是否有 `children`、policy 與自身 capability 決定 local responsibilities | 移除，不在 protocol 固定 Root／Branch／Leaf role |
| `plan_id` | 舊 hierarchy workspace、artifact 與 execution correlation | Root 產生 UUID 字串作為 hierarchy-wide `mlCorreId`；reservation／workspace key 可繼續作為 local state | 不新增 `planId` extension，也不把 internal reservation ID 上線 |
| `publisher_nf_instance_id` | 驗證 assignment 發布者 | Direct subscription edge、authenticated peer／route context 已識別 sender；Notify report 另以 wrapper `nfInstanceId` 識別 reporting node | 不在 request topology 重複傳遞 |
| `intended_recipient_nf_instance_id` | 驗證 bundle 接收者 | `x-flTopology.nfInstanceId` 必須等於當段 subscription 的接收 NWDAF | 搬到 topology root identity |

`mlCorreId` 不取代 subscription resource ID 或 `notifCorreId`。前者關聯整個
hierarchical FL procedure，後兩者繼續識別各 direct parent–child subscription 與
callback lifecycle。

### 3.2 Branch assignment

| 舊欄位 | 新位置／處理 | 結論 |
| --- | --- | --- |
| `assigned_leaf_nf_instance_ids[]` | `x-flTopology.children[].nfInstanceId`，並可在 child node 加入 `enabled`、`priority`、`reportAfter` 或更深的 subtree | 搬到 recursive topology，且不再限定下一層一定是 Leaf |
| `admission.mode: complete_required` | 收斂到 node `policy` 的 topology-ready、selection 與 aggregation rules | 不保留另一個 `admission` object |
| `strategy.algorithm.name: fedprox` | `strategy.method: fedProx` | 搬到 typed strategy |
| `strategy.algorithm.proximal_mu` | `strategy.methodParameters.proximalMu` | 搬到 typed method parameters |
| `strategy.aggregation: sample_weighted` | `strategy.aggregation: sampleWeighted` | 搬到 typed strategy |
| `strategy.participant_selection: all` | `policy.fractionTrain: 1`，並以 `minTrainNodes` 固定最低選取數 | 從 algorithm strategy 移到 FL Server participant policy |
| `strategy.waiting_policy: all` | `policy.acceptFailures: false` | 從 algorithm strategy 移到 aggregation acceptance policy |

舊 `complete_required` profile 對具有 `N` 個明確 children 的等價設定為：

```yaml
policy:
  allowAdditionalCandidates: false
  minAvailableNodes: N
  fractionTrain: 1
  minTrainNodes: N
  acceptFailures: false
```

`minCompletionRate` 在 `acceptFailures: false` 時不參與判定，因此不需要為了重現舊
行為而附上。舊實作沒有 candidate priority 或 delegated discovery，故 migration
baseline 也不應憑空產生 priority；selection order 維持 local implementation
decision。

### 3.3 Leaf assignment

| 舊欄位 | 新位置／處理 | 結論 |
| --- | --- | --- |
| `parent_branch_nf_instance_id` | 由建立該 subscription 的 direct parent 與 local route／resource binding 表示 | 從 wire metadata 移除 |
| `strategy` | 放在該 Leaf 所收到的 `x-flTopology` root node | 逐級傳遞同一 method／aggregation contract |

現有 `republish_leaf_assignment` 會複製同一份 model bytes，主要目的是加入 Leaf
專屬 assignment metadata，並使新 URL 由 Branch 的 plan workspace 管理。Topology
instruction 移入 protocol message 後，不再需要為產生 `LEAF_ASSIGNMENT` 而複製或
重新封裝 model bytes。

新 protocol path 依 preparation 與 model direction 分開處理。Preparation subscription
只以 `x-flTopology` 傳遞 hierarchy contract，設定 `mLPreFlag: true` 並省略
`mLModelInfos`；接收端仍驗證 `modelInterInfo` 與 training requirements，但不下載或驗證
model artifact。Root 收到滿足 readiness 的 realized topology report 後，才將
initial／global model 存入 ADRF，並在第一輪 PUT／PATCH 以標準
`mLModelInfos[].mLModelAdrf` 將同一份 reference 逐級傳遞。Branch 與 Leaves 直接向
ADRF 取得 model，不由 Intermediate 下載後重新發布。由下往上的 Leaf local result 與
Branch domain aggregate 則維持由產生節點透過暫存 `mLFileAddr` 提供，direct parent
直接取得，不存入 ADRF。若 Branch 在下一次 upper-tier update 前，以該 domain
aggregate 啟動後續 lower-tier round，同一 artifact 也由 Branch 以自己的暫存
`mLFileAddr` 下發給 selected children，不轉存 ADRF。

`allowConsumerList` 是 optional schema property；TS 29.575 §4.3.2.2.2 允許 storage
request 省略此欄位，但 §4.3.2.3.2 仍要求 retrieval consumer 位於 allowed consumer
list，或本身就是存入模型的 NWDAF。因此，標準語意下省略清單不等於 public access，
`AllowedConsumer` 也沒有 wildcard。現有 testbed 使用 plain HTTP，ADRF 尚未取得
authenticated caller NF identity，因此目前實作不會在 server side 強制執行該
list。Protocol mode 不再省略 `allowConsumerList`：Root 依 realized topology 為每輪
record 列出可能取得 global model 的 Branch／Leaf `nfInstanceId`，或在能完整且不過度
授權時使用 `nfSetId`。Topology update 若加入新的 Root global-model consumer，Root
必須在下發 reference 前透過 ADRF PUT 更新清單，或等下一輪建立包含該 identity 的新
record。現有 testbed
尚未執行 caller authorization，因此只能驗證清單的產生、傳輸與保存，不能把成功
retrieval 當成 access-control enforcement 的證據。

---

## 4. Preparation result metadata 對照

| 舊欄位 | 新位置／處理 | 結論 |
| --- | --- | --- |
| `artifact_role: HIERARCHY_PREPARATION_RESULT` | Notify 的 `x-flTopologyReport` 已直接承載 realized topology | 若沒有新的 model／result artifact，就不再為回報 topology status 建立整包 model bundle |
| `message_type: PREPARATION_RESULT` | 由 Notify field `x-flTopologyReport` 表示 | 移除 discriminator |
| `plan_id` | Notify 的 `mlCorreId` | 不新增 report-local plan ID |
| `publisher_nf_instance_id` | `x-flTopologyReport.nfInstanceId` | 搬到 reporting-node wrapper |
| `intended_recipient_nf_instance_id` | 由 callback resource、`notifCorreId` 與 direct parent route 決定 | 不在 report 重複傳遞 |
| `assigned_client_nf_instance_ids[]` | `x-flTopologyReport.children[]` 的 candidate snapshot | 不另外維護 assigned list |
| `prepared_clients[]` | 對應 child `status: ACTIVE` 與 `statusTimestamp` | 搬到 per-node status |
| `failed_clients[]` | 對應 child `status: FAILED`、`statusTimestamp` 與 `statusCause` | 搬到 per-node status |
| `timed_out_client_nf_instance_ids[]` | 對應 child `status: FAILED`、`statusCause: RESPONSE_TIMEOUT` | 合併到統一 status model |
| `outcome: READY／FAILED` | 由 realized child statuses 與 node 實際採用的 `policy` 計算 | 移除重複的整體結果欄位 |

新 report 可以同時回報 `UNCONFIRMED`、`DEPLOYING`、`ACTIVE`、`FAILED` 與
`INACTIVE`，不再要求每次 preparation 結果把所有 assigned Clients 強制切成
prepared／failed／timed-out 三個完整 partition。這也避免現有 pre-dispatch failure
發生時，將尚未實際嘗試的其他 Clients 人工標成 failure。

### 4.1 舊 failure cause 的處理

舊 cause 不應機械式一對一保留：

| 舊 cause | 新處理 |
| --- | --- |
| `NOT_AVAILABLE_ML_TRAIN` | `statusCause: NOT_AVAILABLE_ML_TRAIN` |
| timeout partition | `statusCause: RESPONSE_TIMEOUT` |
| downstream subscription 明確拒絕 | `statusCause: SUBSCRIPTION_REJECTED` |
| transport／peer communication failure | `statusCause: COMMUNICATION_FAILURE` |
| FL capability 不符合 | 優先使用 `NOT_AVAILABLE_ML_TRAIN`；只有 optional feature negotiation 失敗才使用 `FEATURE_NOT_SUPPORTED` |
| `INVALID_ASSIGNMENT` | 上游 request 尚未接受時使用既有 `400` error；不要先建立 resource 再轉成 topology status |
| `INVALID_BUNDLE` | Hierarchy assignment 移出 bundle 後失去原用途；一般 model artifact 無效仍走既有 model／training error path |
| `REQUIREMENTS_NOT_MET` | 若 downstream operation 被拒絕，relationship 回報 `SUBSCRIPTION_REJECTED`；完整原因保留在 HTTP／local evidence |
| `INTERNAL_ERROR` 或無法安全分類的 discovery error | `statusCause: OTHER`；只有確認為通信失敗才使用 `COMMUNICATION_FAILURE` |

---

## 5. Round artifact 中的 instruction 與 evidence

舊 model bundle 不只含 `hierarchy_metadata`。以下欄位需要按語意分開：

| 舊資訊 | 新處理 |
| --- | --- |
| `analytics_event` | Training requirement 以 `mLEventSubscs[].mLEvent` 為 authoritative source；artifact 可保留 model compatibility descriptor 並與 message 交叉驗證 |
| `model_interoperability` | Training requirement 以既有 `mLEventSubscs[].modelInterInfo` 為 authoritative source；artifact 可保留 descriptor 驗證可載入性 |
| `fl_metadata.ml_corre_id` | SBI instruction／result 以標準 `mlCorreId` 為 authoritative source；artifact 可保留副本作 provenance，且必須與 message 相符 |
| `fl_metadata.round_ind` | SBI 使用標準 `roundInd`；artifact 可保留副本驗證該 result 所屬 local round，不放入 topology report |
| `client_training.epochs` | 對 Leaf 的持續 local-work contract 可表達為 `reportAfter: {count: ..., unit: epoch}`；若每輪變更則透過既有 subscription update 更新 contract |
| Branch 每次 upper update 前執行的 lower rounds | 可表達為 Branch node 的 `reportAfter: {count: ..., unit: round}`；現行一次 lower round 對一次 upper update 的 baseline 為 count `1` |
| model／preprocessing digests、weights digest | 保留在 artifact，作完整性與 lineage evidence |
| training sample count、participant contribution、dataset evidence | 保留在 result artifact，供 aggregation 與實驗 evidence 使用 |
| subordinate aggregation／validation summaries | 保留在 result／final artifact；它們是 learning-result provenance，不是 topology lifecycle status |

`reportAfter` 描述 local node 完成多少工作後向 direct parent 回報，不取代
`roundInd`，也不要求上下 tiers 的 local rounds 同步。

---

## 6. 預期可刪除的舊 hierarchy-only payload

當 protocol-driven path 完整成立後，可從 runtime control path 移除：

- `BranchAssignmentMetadata` 與 `LeafAssignmentMetadata` 的 wire transport；
- `PreparationResultMetadata` 的 bundle transport；
- `HierarchyMessageType` 及 assignment／preparation-result discriminator；
- 為了攜帶 topology status 而建立的 `HIERARCHY_PREPARATION_RESULT` model bundle；
- `assigned_*`、prepared／failed／timed-out partition 與整體 `outcome` 的重複表示；
- bundle 中作為 orchestration source 的 admission、participant selection、waiting
  policy 與 strategy。

以下內容不能因 migration 一併刪除：

- 實際 model／preprocessing files、Root global model 的 ADRF storage／retrieval
  boundary，以及 Leaf result 與 Branch domain aggregate 的暫存 URL serving
  boundary；後者同時涵蓋 Branch 對 direct parent 的上行回報與對 selected children
  的後續 lower-tier 下發；
- round input、local result、global result 與 final model artifact；
- digest、sample count、aggregation provenance、dataset evidence 與 validation
  evidence；
- subscription、callback、workspace、reservation 與 cleanup 所需的 local state。

---

## 7. Production owner 與 cutover

### 7.1 Root→Intermediate instruction

現有 Root PyMTLF coordinator 同時持有 static topology planner、strategy config、
model catalog 與 hierarchy execution lifecycle，因此它是第一段 `x-flTopology` 的
authoritative producer。Go NWDAF route 不具備 topology decision 所需的輸入，不應
在 gateway 內自行組合 tree、policy 或 strategy。

Preparation 資料流為：

```text
Root PyMTLF coordinator
  -> FL Server preparation subscription builder
  -> standard requirements + mLPreFlag=true + x-flTopology; no mLModelInfos
  -> Root Go NWDAF route / peer transport
  -> Intermediate Go NWDAF resource
  -> Intermediate PyMTLF FL Client resource
  -> lower-tier preparation subscriptions; no mLModelInfos
  -> x-flTopologyReport rolls up to Root
```

第一輪與後續 round 的 global-model 資料流為：

```text
Root PyMTLF coordinator
  -> derive allowed consumers from realized topology
  -> publish temporary global-model URL
  -> containing Root Go NWDAF ADRF ML Model Management proxy
  -> ADRF POST /mlmodel-store-records with allowConsumerList
  -> validate 201 + Location + NadrfMLModelStoreRecord
  -> build mLModelAdrf reference
  -> FL Server PUT/PATCH builder with roundInd
  -> Root Go NWDAF route / peer transport
  -> Intermediate Go NWDAF resource
  -> Intermediate PyMTLF FL Client resource
  -> containing Intermediate Go NWDAF ADRF retrieval proxy
  -> ADRF GET /mlmodel-store-records?store-trans-id=...
  -> validate record and download its mLFileAddr
  -> Intermediate coordinator
```

Branch domain aggregate 作為後續 lower-tier round 輸入時，資料流為：

```text
Branch aggregator
  -> publish aggregate in Branch temporary workspace
  -> lower-tier PUT/PATCH with mLFileAddr
  -> selected child Go NWDAF / PyMTLF resource
  -> download from Branch temporary URL
  -> local trainer
```

這條路徑不建立 ADRF record；Branch 可用同一份暫存 aggregate 向 direct parent 回報，
並在本地 lifecycle 結束後依既有 workspace owner 清理。

現有 `HierarchyPreparationTarget.assignment_url` 同時代表 model transport 與
hierarchy instruction。Cutover 後必須拆成：

- Preparation request body 只以 `x-flTopology` 表示接收者 subtree，並省略
  `mLModelInfos`；
- Root 收到 realized topology 後建立含 `allowConsumerList` 的 ADRF record；
- 第一輪與後續 round update 才在標準 `mLModelInfos[]` 放入 initial／global model
  identity 與 reference；本 profile 要求同一 entry 同時提供 `modelUniqueId`，以及包含
  `adrfId` 與 `storTransId` 的 `mLModelAdrf`。

每一份 initial／global model 使用 model payload immutable 的 ADRF store record。Root
PyMTLF 是該 record 的 distribution lifecycle owner，負責配置不重複的
`modelUniqueId`、依 realized topology 產生 `allowConsumerList`、保存
`adrfId`／`storTransId` 與該 round 的關聯，並在所有該輪 selected subtree outcomes
結束且沒有 in-flight retry，或 procedure terminal 時要求刪除 record。若 Root
restart 後無法恢復這組 mapping，該 procedure 必須明確失效，不能僅靠
`roundInd` 猜測 ADRF record。Exact durable allocator、restart representation 與
cleanup helper 由 Slice 4 detailed plan 固定；新增 consumer 時可在下發 reference 前
更新 ACL metadata，但不得改寫同一 record 的 model payload 來服務不同 round，避免
late consumer 下載到後一輪 model。

Go NWDAF 負責 wire validation、callback rewrite、peer routing 與 accepted resource
representation；它保存 contract，但不執行 participant selection 或 aggregation
policy。接收端 PyMTLF FL Client resource 保存接收到的 subtree，Intermediate
coordinator 才是 direct-child selection 與 lower-tier dispatch 的 consumer。

### 7.2 Intermediate→下一層 instruction

Intermediate coordinator 以收到的 subtree、node policy 與本地 selection 結果產生
下一段 contract。它只把 direct child 對應的 node 當成下一段
`x-flTopology` root，不能把上層整棵 tree 原樣轉送，也不能由 Go route 推測下一層
assignment。

下一段仍沿用同一條 FL Server→Go NWDAF→peer NWDAF→FL Client resource 路徑。
接收者若沒有 children，local training owner 消費 strategy 與 `reportAfter`；接收者
若還要建立下一層，則由其 coordinator 繼續逐級處理。

### 7.3 Realized topology report

Direct parent 的 FL Server／Intermediate coordinator 最清楚哪些 child subscription
已建立、失敗、逾時或離開，因此它是 direct-child relationship status 的
authoritative producer。PyMTLF FL Client resource 將自身採用的 contract 與該
coordinator 產生的 child snapshot 包成 `x-flTopologyReport`，再透過既有 Notify
路徑逐級送回 parent。

目標資料流為：

```text
Intermediate coordinator / local FL Server state
  -> Intermediate FL Client Notify producer
  -> Intermediate Go NWDAF callback route
  -> Parent Go NWDAF route
  -> Parent PyMTLF FL Server callback collector
  -> Parent coordinator
```

因此現有 `HIERARCHY_PREPARATION_RESULT` bundle、Root 端 result-bundle download 與
assignment-specific validation 可移除。HTTP／subscription failure 的完整證據仍保留
在 local execution record；跨層只回報 candidate schema 定義的 status、timestamp
與 cause。

### 7.4 Correlation 與 local state

Root coordinator 為每個 hierarchical FL procedure 產生 UUID 字串，並逐級傳遞為
hierarchy-wide `mlCorreId`。Receiver 在本地 active／retention window 內不得把同一
UUID 綁到另一個 procedure scope。現有 FL Server
`process_id`、subscription resource ID、`notifCorreId`、workspace key 與 experiment
reservation ID 繼續由各 local owner 管理，不因 wire correlation 共用而合併。

現有 FL Server 在建立每段 preparation subscription 時直接把 local
`process.process_id` 寫入 `mlCorreId`；這是 cutover 時必須拆開的 coupling。每段
local process 需要保存：

- hierarchy-wide `mlCorreId`，用於 wire correlation；
- local process／resource identifier，用於本地 lifecycle、round 與 cleanup；
- parent／child resource mapping，用於 Notify、PUT／PATCH 與 DELETE。

Go route 與 PyMTLF subscription resource 保存的是已接受的 persistent wire
contract；round state、selection state、training workspace 與 aggregation evidence
仍由 PyMTLF local process 保存。

### 7.5 欄位 owner 摘要

| 資訊 | 資訊產生 owner | Wire 傳遞 | 保存 owner | 執行 consumer |
| --- | --- | --- | --- | --- |
| 初始 subtree | Root PyMTLF coordinator | `x-flTopology` | Go route representation＋接收端 PyMTLF resource | Intermediate coordinator／local Client |
| 下一層 subtree | Direct-parent Intermediate coordinator | 下一段 `x-flTopology` | 該段 Go route＋接收端 PyMTLF resource | 下一層 Intermediate／Client |
| `policy` | 該 node 的 direct parent；省略時由 node local decision 補足 | node-local `policy` | 接收端 resource＋local FL Server process | Candidate selection、readiness、round selection、aggregation completion |
| `strategy` | 上層 coordinator；依 subtree 逐級明確傳遞 | node-local `strategy` | 接收端 resource＋local process | Local training／aggregation implementation |
| `reportAfter` | Direct parent；省略時由接收 node 決定 | node-local `reportAfter` | 接收端 resource＋local process | 該 node 對 direct parent 的回報 cadence |
| Realized topology | Direct parent 的 coordinator／FL Server state | Notify `x-flTopologyReport` | 各層 callback collector／coordinator | 上層 topology view 與後續 update decision |
| Root initial／global model bytes | Root model catalog／aggregator；Root PyMTLF 在 realized topology ready 後擁有 ADRF distribution lifecycle | 第一輪／後續 round update 的標準 `mLModelInfos[].mLModelAdrf`；preparation 不帶 model | ADRF record＋Root procedure-to-record／allowed-consumer mapping | Branch／Leaf 透過 containing Go NWDAF retrieval proxy 取得 record 後交給 trainer |
| Leaf local result bytes | Leaf trainer | 標準 `mLModelInfos[].mLFileAddr` | Leaf 的暫存 workspace | Direct-parent aggregator |
| Branch domain aggregate bytes | Branch aggregator | 向 direct parent 回報及作為後續 lower-tier round 輸入時，皆使用標準 `mLModelInfos[].mLFileAddr` | Branch 的暫存 workspace | Direct-parent aggregator／selected-child trainer |
| Hierarchy correlation | Root coordinator | 標準 `mlCorreId` | 每段 Go route 與 PyMTLF resource 另存一份 | Callback validation 與跨層 procedure correlation |
| Local round／process state | 各層 local FL Server／Client process | 標準 `roundInd` 與 local state | PyMTLF execution owner | 該層 training／aggregation；不跨層同步 |

---

## 8. 實作階段仍需落實的細節

Go candidate types 已決定延伸既有 `internal/compat/mlmodeltraining`，PyMTLF 則在
既有 wire model 建立 explicit typed properties。Legacy／protocol 切換也已決定由
Root PyMTLF orchestration 明確選擇，且同一 execution 不得合併兩種 authority。

Detailed implementation slice 仍需落實：

1. Create／PUT／PATCH 中 candidate fields 的 exact helper／method placement 與
   representation update path。
2. Legacy／protocol selector 的 exact config key，以及 ambiguous contract 的錯誤
   mapping。
3. Slice 4 detailed plan 需固定 preparation model-free gate、每輪 model-payload
   immutable ADRF record 的 exact
   `modelUniqueId` allocator、Root procedure-to-record persistence、store／retrieval
   helper placement、realized-topology-to-`allowConsumerList` mapping、ACL update ordering、
   terminal cleanup 與 failure mapping；現有 normal-round validator／consumer 需同時
   支援兩種標準 model source：Root global model 的 `mLModelAdrf` 必須查詢 standard
   store record，再下載 record 中的 `mLFileAddr`；Branch domain aggregate 的
   `mLFileAddr` 則維持直接下載。Testbed 尚未執行 caller authorization 的限制需保留
   在 evidence，不以 successful retrieval 宣稱 access-control enforcement。
4. `reportAfter` 開始由 protocol 控制後，現有 local config 與 round artifact
   `client_training.epochs` 的 fallback／consistency validation 規則。
5. Preparation result bundle 移除後，哪些 local HTTP／timeout evidence 需要保留在
   execution record 供 debug 與測試斷言，但不再跨層放入 model artifact。
