# Hierarchical NWDAF FL Protocol Conformance Case Ownership Mapping

日期：2026-09-02

狀態：Ready for User Review／case owner、test seam 與 retained-result owner 已完成對照

相關文件：

- [Protocol Implementation Current-State Inventory](./Protocol%20Implementation%20Current-State%20Inventory.md)
- [Protocol Resource Lifecycle and Wire Integration Mapping](./Protocol%20Resource%20Lifecycle%20and%20Wire%20Integration%20Mapping.md)
- [Protocol Conformance Matrix](../../../design/hierarchical-federated-learning/protocol_conformance_matrix.md)

---

## 1. 目的

本文件不重述 conformance case 的輸入與預期結果，只確認每組 case 的 primary
production owner、需要的 mirror validation、可重用 behavior 與驗證位置。這份對照
用來避免把 schema parsing、route lifecycle 或 PyMTLF execution 混成單一大型
implementation slice。

---

## 2. Owner 原則

- **Go wire／receiver validation**：負責 public／peer SBI message shape、cross-field
  rule、containing-NWDAF identity、callback route identity 與 `ProblemDetails`。
- **Go route state**：負責 accepted／backend representation、feature negotiation、
  operation revision、callback routing 與 CRUD atomicity；不執行 FL policy。
- **PyMTLF wire mirror**：private backend boundary 重複必要 contract validation，避免
  internal request 或 future caller 繞過 public SBI 後產生不同語意。
- **PyMTLF FL Client resource**：負責 persistent subscription contract、operation
  dispatch、一次性 retained-result lookup與 resource cleanup。
- **PyMTLF FL Server／coordinator**：負責 participant resource、candidate pool、policy
  execution、Notify consumption、outstanding lookup 與逐級 topology report。
- **`nwdaf-resources`**：只驗證跨 real Go／PyMTLF／peer process 的 observable
  behavior，不取代 repository unit tests。

---

## 3. Request／topology cases

| Cases | 主要 owner | Mirror／consumer | 驗證重點 |
| --- | --- | --- | --- |
| `REQ-01`, `TOP-07`, `TOP-08` | Go `internal/compat/mlmodeltraining` candidate types／decoder | PyMTLF wire models | Go model tests＋PyMTLF wire tests；確認 exact JSON round trip |
| `REQ-02`, `REQ-03` | Go request／PATCH validation，並讀取 existing route identity | PyMTLF subscription validator | Go validation與 processor mutation tests；invalid request不得進 backend |
| `TOP-01`, `TOP-02`, `TOP-03` | Go receiver validation＋containing NWDAF identity | PyMTLF wire／resource admission | Root identity、duplicate與 cycle unit tests；至少一個 peer-process rejection case |
| `TOP-04`, `TOP-05`, `TOP-06` | Go cross-field validator | PyMTLF resource admission | Path-specific `invalidParams`；確認失敗不建立、DELETE 或 lookup |
| `TOP-09` | Go receiver capability／known-value validation | PyMTLF execution capability | Unknown forward-compatible value decode pass；無法執行時回 `403`，不得 local fallback |

直接可重用：現有 `RequirementsError`→`ProblemDetails.invalidParams` mapping、
Create reservation cleanup、PUT／PATCH operation revision 與 PyMTLF replace rollback。

確定 gap：Go／PyMTLF 尚無 candidate types、recursive validation、request receiver
identity validation或 unknown-but-unsupported execution gate。

---

## 4. Policy、scope 與 PATCH cases

| Cases | 主要 owner | 支援 owner | 驗證重點 |
| --- | --- | --- | --- |
| `POL-01`, `POL-02` | PyMTLF local FL Server readiness state | Intermediate coordinator candidate status | FL Server state-machine unit tests；未達門檻不得 dispatch round |
| `POL-03` | PyMTLF FL Server participant selector | Coordinator active pool | Selection-count table tests，包含 fraction floor、minimum 與 active cap |
| `POL-04`, `POL-05` | PyMTLF FL Server aggregation completion policy | Notify/result collector | Success、failure、timeout組合；未達 acceptance不得產生 aggregate |
| `PATCH-01` | PyMTLF FL Client resource＋Intermediate coordinator | Go effective representation | Upstream assigned array完整取代、local discovered pool保留的 state test |
| `PATCH-02` | Intermediate coordinator | FL Server downstream DELETE；Go mutation atomicity | Active child立即排除、DELETE outcome與 `INACTIVE` report |
| `PATCH-03` | Intermediate coordinator assigned／local candidate provenance | Local discovery owner | 省略、prohibition與再次 discovery 的 provenance test |
| `SCOPE-01` | PyMTLF coordinator／FL Server／FL Client execution binding | Wire contract | 每個 node 的 policy、strategy、`reportAfter` consumer test；不得自動繼承 |

直接可重用：現有 FL Server participant collection、round wait、timeout、aggregation、
peer resource PATCH／DELETE 與 PyMTLF resource operation gate。

確定 gap：現有 state machine 固定 complete-required／all participants；candidate pool
沒有 upstream-assigned 與 locally discovered provenance，也沒有 policy-driven
selection、partial completion 或 disabled-child transition。

Delegated candidate discovery 不需要修改 NRF schema。Go internal NRF proxy 已允許
NWDAF query 省略 `target-nf-instance-id`，並使用 `nnwdaf-mlmodeltraining` service、
FL capability、analytics event、model interoperability 與 tracking area 等既有條件
取得多個 profiles。PyMTLF 現有 hierarchy resolver 只提供 explicit identity 的唯一
解析，因此需在同一 resolver boundary增加 list-discovery operation。

Discovery filter 的 authoritative input 是既有 Model Training request、該 node 的
local area／operator config與 advertised NF profile，不新增 policy criteria field。
若 node沒有足夠 local criteria履行 `allowAdditionalCandidates`，它必須拒絕該明確
contract，不能無條件選取所有 NWDAFs。取得 profiles後，Intermediate coordinator
排除自身、重複 explicit identities與不符合 capability 的 profiles，並以
`upstream-assigned`／`locally-discovered` provenance存入 local candidate pool。

---

## 5. Notify／topology report cases

| Cases | 主要 owner | Mirror／consumer | 驗證重點 |
| --- | --- | --- | --- |
| `NOT-01`, `NOT-03`, `NOT-04` | Parent Go callback route validation | Parent PyMTLF FL Server callback collector | `notifCorreId`、`mlCorreId`、selected peer identity 與 report root組合 |
| `NOT-02`, `NOT-06`, `NOT-07`, `NOT-09` | Go notification shape／cross-field validator | PyMTLF Notify model | Candidate detailed-information minimum、status cause 與既有 mutual exclusion |
| `NOT-05` | Go recursive report validator | PyMTLF report consumer | Duplicate／cycle rejection；coordinator realized view不得部分更新 |
| `NOT-08` | PyMTLF coordinator report consumer | Go transparent typed transport | Unknown status／cause可保存與轉送，但不觸發已知 action |

直接可重用：Go callback route lookup、standard correlation、一般 training result 的
expected round validation、raw body forwarding，以及 PyMTLF callback duplicate digest
handling。Retained `FOUND` 使用被查到結果的 local `roundInd`，不得沿用 normal-result
expected-round equality。

確定 gap：Go 現行 Notify shape拒絕 topology-only report；route identity尚未比較
`x-flTopologyReport.nfInstanceId`；PyMTLF 不會建立 realized topology view。

---

## 6. Retained-result cases

| Cases | 主要 owner | 支援 owner | 驗證重點 |
| --- | --- | --- | --- |
| `RET-01`–`RET-04` | Go Notify cross-field validator | PyMTLF wire validator／FL Server consumer | `FOUND`、`NOT_FOUND`、`FAILED` 與 standard model／round fields 的組合表；`FOUND.roundInd` 不與 replacement resource expected round 比較 |
| `RET-05`, `RET-06` | Recipient PyMTLF FL Client resource | Go operation gate＋accepted representation | Create／PUT／PATCH各觸發一次；false／省略不觸發；trigger不持久化 |
| `RET-07` | Requesting PyMTLF FL Server participant state | Go route serialization | Outstanding lookup存在時拒絕第二次 request；timeout不自動解除 |
| `RET-08` | Requesting PyMTLF FL Server callback consumer | Parent coordinator | 任一 terminal outcome解除 outstanding；只有 `FOUND` 交付 artifact |

目前沒有可直接重用的 latest-completed retained-result index。可以重用的是既有
participant callback correlation、artifact validation、operation revision 與 resource
cleanup。現有 local result 在 callback delivery 前已完成 artifact publication，但
`FLClientResource` 沒有保存 result reference；callback payload 只存在 outbox work item，
不能供另一個 subscription 查詢。

---

## 7. Feature negotiation cases

| Cases | 主要 owner | 支援 owner | 驗證重點 |
| --- | --- | --- | --- |
| `FEAT-01` | Go request decoder／validator | PyMTLF recipient capability | Offered feature與 candidate fields 可進正常 validation |
| `FEAT-02` | Sender Go route negotiated-feature state | Sender PyMTLF FL Server participant resource | `201` accepted representation與 local route保存相同 negotiated bit |
| `FEAT-03` | Sender PyMTLF FL Server cleanup／status producer | Go peer DELETE route | Required feature未接受時刪除已建立 resource，回報 `FEATURE_NOT_SUPPORTED` |
| `FEAT-04` | 每一段 sender FL Server／Go route | Intermediate coordinator | Root→Intermediate與 Intermediate→Client 使用獨立 response fixture |

確定 gap：現有 `suppFeats` 只是被 request／response model攜帶，沒有 offered／accepted
intersection、route-level negotiated state 或後續 candidate operation gate。

---

## 8. Real-process minimum evidence

Repository unit tests完成後，`nwdaf-resources` 至少需要下列跨程序證據：

1. Create 經 Root Go→peer Go→Intermediate PyMTLF 後，`201` response、兩端 route 與
   recipient resource 都保留相同 `x-flTopology`。
2. PATCH 完整取代 upstream-assigned children，同時保留 Intermediate local candidate
   pool，並對 disabled active child 執行 DELETE。
3. Topology-only Notify 通過兩個 Go routes 到達 parent PyMTLF，錯誤 root identity
   在 parent callback boundary 被拒絕。
4. Feature 3 在 Root→Intermediate成功、Intermediate→Client失敗時不被跨 edge繼承，
   並完成失敗 resource cleanup。
5. Retained-result request不進 persistent representation，連續第二次 request在前一個
   outcome 前被拒絕，terminal outcome後才可再次送出。
6. 不含 candidate fields的既有 flat／distributed FL Create、round、Notify與 DELETE
   regression仍維持原行為。
7. Root／Intermediate 的 preparation Create 均設定 `mLPreFlag: true` 且不帶
   `mLModelInfos`；接收端不發出 ADRF GET 或 model artifact validation，Root 只在收到
   滿足 readiness 的 realized topology report 後才進入 model distribution。
8. Root 依 realized topology 建立含正確 `allowConsumerList` 的 ADRF record，再以
   第一輪 PUT／PATCH 下發 `roundInd` 與 `mLModelAdrf`；兩個不同 containing NWDAFs
   使用同一 reference 完成 store-record retrieval 與 artifact download，Intermediate
   不 republish global model bytes。
9. Branch 完成 domain aggregation 後，以自己的暫存 `mLFileAddr` 將 aggregate 下發給
   selected children，啟動下一個 lower-tier round；該路徑不建立 ADRF record，且同一
   artifact 可透過暫存 reference 向 direct parent 回報。
10. ADRF store failure保留 preparation resources但阻止該輪 update；retrieval／artifact
   validation失敗不開始 local training；procedure terminal後清除該輪 store record。
   Testbed evidence 驗證 allow list 的產生與保存，並明確記錄目前尚未執行 caller
   authorization enforcement。

這些情境只證明 wire 與 process integration；policy selection、threshold arithmetic與
recursive validation仍由 deterministic unit tests負責完整覆蓋。

---

## 9. Retained-result local owner 結論

### 9.1 直接證據

- Leaf local training 與 Branch lower-tier aggregate 最後都由同一 containing NWDAF 的
  `FLClientEngine` 組成上行 Notify。
- Result artifact 在 callback delivery 前已經發布完成；transport failure只會讓
  resource停在 `RESULT_PENDING` 並持續 retry。
- `FLExperimentRegistry` 已允許同一 active experiment 下，多個 Client subscription
  使用相同 `mlCorreId`；因此 replacement Branch 建立的新 subscription 可以與舊
  subscription 共存。
- 現有 workspace 可依 process／participant／round／digest提供 artifact，但沒有
  `mlCorreId -> latest completed result` 查詢，也不能從 callback outbox穩定取得結果。

### 9.2 Implementation owner

Latest-completed index 應由接收 training command並產生上行 result 的
`FLClientEngine` 擁有，key 使用 Root 產生的 hierarchy-wide UUID `mlCorreId`，而不是
舊 Branch 的 subscription ID。Receiver 在本地 active／retention window 內拒絕將同一
UUID綁到另一個 procedure scope。Leaf result與 Branch aggregate沿用同一抽象，不需要
由 FL Server process各自建立第二套 retained store。

每筆 entry 至少保存：

- `mlCorreId`；
- 最新完成的 local `roundInd`；
- 可重建 `mLModelInfos` 的 artifact reference與 event；
- artifact retention handle／owner；
- completion timestamp，供 local lifecycle與 debug使用。

Entry 應在 model／aggregate artifact成功發布後、第一次 callback enqueue 前原子更新。
因此舊 Branch的 callback即使送不出去，新 subscription仍能要求同一 NWDAF回傳結果。
Slice 3需加入 UUID scope test：replacement subscription可重用同一 procedure UUID；
已被另一個 active procedure使用的 UUID不得綁定、讀取或覆寫其 retained entry。

### 9.3 Lifecycle constraint

- 較高 `roundInd` 的完成結果取代較低 round；未完成工作不得覆寫 index。
- 單一舊 subscription DELETE 不得直接刪除同一 `mlCorreId` 的 retained entry，若另一個
  replacement subscription或 active experiment仍引用該 procedure。
- Procedure terminal cleanup、backend generation reset或 retention expiry時，index與
  artifact reference必須一起失效。
- `mlCorreId` 已採 UUID 格式，能通過現有 workspace key 的 syntax requirement；但
  retained-result 與 hierarchy plan 仍是不同 lifecycle owner，不能因格式相同就直接
  把 `mlCorreId` 交給既有 `release_plan` API。Implementation slice 必須另行定義
  retention handle或調整 workspace owner abstraction。
- 查到 entry但 artifact已被清除或 integrity validation失敗時，不得回 `FOUND`；依
  candidate contract回 `FAILED`。沒有 entry則回 `NOT_FOUND`。

這是 local implementation state，不新增 protocol field，也不要求 Root知道 Leaf
workspace或舊 subscription ID。至此 candidate cases 已能分配到現有 repository與
component boundary，可依 dependency與驗證邊界拆分 implementation slices。
