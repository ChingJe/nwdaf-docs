# Hierarchical NWDAF FL Protocol Resource Lifecycle and Wire Integration Mapping

日期：2026-09-02

狀態：Ready for User Review／lifecycle trace、type integration 與 migration authority
已完成第一輪盤點

相關文件：

- [Protocol Extension Implementation Plan](./Hierarchical%20NWDAF%20FL%20Protocol%20Extension%20Implementation%20Plan.md)
- [Protocol Implementation Current-State Inventory](./Protocol%20Implementation%20Current-State%20Inventory.md)
- [Model Bundle Metadata to Protocol Schema Mapping](./Model%20Bundle%20Metadata%20to%20Protocol%20Schema%20Mapping.md)
- [Candidate OpenAPI artifact](../../../design/hierarchical-federated-learning/candidate_openapi.yaml)
- [Protocol Conformance Matrix](../../../design/hierarchical-federated-learning/protocol_conformance_matrix.md)

---

## 1. 目的

本文件確認 candidate fields 在既有 Create／PUT／PATCH／DELETE／Notify flow 中會經過
哪些 component、在哪裡保存，以及現有 typed reconstruction 會造成什麼差距。

本文件只盤點 wire integration 與 resource lifecycle，不開始 production code 修改。
盤點結論已在第 7.1 節固定 Go type integration 方式，供後續 detailed slice 使用。

---

## 2. 現有 route ownership

### 2.1 Go NWDAF

Go NWDAF 是 external／peer SBI 與 local PyMTLF 之間的 route owner，負責：

- 解析與驗證 request／Notify；
- 將 public callback URI 改寫成 containing NWDAF 的 local callback；
- 呼叫 local PyMTLF backend 或 peer NWDAF consumer；
- 保存 public accepted representation、backend representation、destination、
  `notifCorreId`、`mlCorreId` 與 expected `roundInd`；
- 以 operation revision 保護 Create／PUT／PATCH／DELETE transition；
- 依 callback route 將 Notify 原始 body 送到 PyMTLF 或外部 consumer。

Go NWDAF 不應執行 topology selection、policy、strategy、local training 或 aggregation。

### 2.2 PyMTLF

PyMTLF FL Client resource 是接收端 subscription execution owner，負責：

- 保存已接受的 subscription representation；
- 將 preparation、round、validation 或 idle contract 映射到 local state；
- 執行 Branch lower-tier dispatch 或 Leaf local training；
- 產生 callback result。

PyMTLF FL Server process 是發送端 participant-resource owner，負責：

- 建立每段 downstream subscription；
- 保存 participant resource URL 與 `notifCorreId` mapping；
- 發送 PUT／PATCH／DELETE；
- 收集與驗證 Notify；
- 維護該層 local round、selection 與 aggregation state。

---

## 3. 現有 field preservation 結論

PyMTLF 的 standard-shaped Pydantic models 使用 `extra="allow"`，因此 unknown JSON
property 通常能留在 resource representation。這只代表 bytes 有機會被保存，不代表
field 已經被驗證或執行。

Go compatibility models 只定義現有 Release 18 fields。現有 parse→marshal、
PATCH apply 與 backend／peer response reconstruction 都會以 typed struct 重建 JSON，
因此 unknown `x-*` fields 會遺失。Request URI rewrite 本身使用 raw JSON object，會
保留其他 properties；真正的遺失點是在 destination response 與 effective resource
state 重建。

所以「request 能到達 PyMTLF」不能視為 protocol integration 完成。合法 candidate
fields 必須同時通過：

1. inbound parse 與 cross-field validation；
2. callback URI rewrite 與 peer forwarding；
3. destination accepted response；
4. Go accepted／backend resource representation；
5. 後續 PUT／PATCH effective-state calculation；
6. PyMTLF persistent contract 與 execution consumer；
7. Notify validation、raw transport 與 FL Server consumer。

---

## 4. 各 operation 對照

### 4.1 Create

現有 local Create 流程為：

```text
External／Root PyMTLF
  -> Go Parse + Validate
  -> raw JSON notifUri rewrite
  -> local PyMTLF Create
  -> PyMTLF resource representation response
  -> Go typed parse + marshal
  -> accepted representation / 201 response
```

現況差距：

- Raw request 中的 unknown fields 可以到達 local PyMTLF。
- PyMTLF 會把 unknown fields 保留在自身 resource representation。
- Go 將 PyMTLF response parse 成現有 typed model 後重新 marshal，導致
  `x-flTopology` 從 public accepted representation 與 `201` response 消失。
- Local route 的 backend representation 目前保存 callback-rewritten request，可能仍
  含 unknown fields，但後續 PATCH 是從 accepted representation 計算，兩份 state
  因而會分歧。

Remote Create 還會多經過 peer consumer 與對端 Go route；request 仍可能到達對端
PyMTLF，但任一 Go typed response reconstruction 都會讓 sender 看不到 extension。

目標 invariant：

- 若 `HierarchicalFLOrch` negotiation 成功，accepted response、Go route state 與
  recipient PyMTLF resource 必須保存同一份 persistent topology contract。
- Preparation Create 設定 `mLPreFlag: true` 且省略 `mLModelInfos`；接收端驗證
  `modelInterInfo`、training requirements 與 topology contract，但不得要求 model
  reference、下載或 artifact validation。
- `x-retainedResultReq` 與 topology node 中的 `retainedResultReq` 只觸發本次 lookup，
  不得出現在 accepted persistent representation。
- Create failure 不得留下可接受 callback 的半成品 route；沿用現有 creating revision
  與 compensation behavior。

### 4.2 PUT

現有 PUT 會先驗證 resource identity，再轉送完整 representation。Destination 回
`200` 時，Go 會 parse／marshal response；回 `204` 時，Go 使用原 request 與
callback-rewritten request 更新兩份 representation。

因此目前 extension preservation 會依 response status 改變：

- `200`：unknown fields 會在 typed response reconstruction 遺失；
- `204`：raw request 可能被保存，但沒有 candidate validation 或 operation-scoped
  instruction stripping。

目標 invariant：

- `200` 與 `204` 必須產生相同的 effective persistent contract。
- `mlCorreId` 與 `notifCorreId` 維持現有 immutable resource identity。
- 完整 replacement 的 `x-flTopology` 取代舊 persistent topology；省略時代表新的
  representation 沒有 persistent topology，而不是沿用 PATCH semantics。
- Lookup trigger 在本次 PUT 完成後即被消費，不進入新 representation。

### 4.3 PATCH

現有 Go PATCH 先把 accepted representation parse 成 typed subscription，再套用 typed
patch。Raw patch 會送到 destination，但 Go 自己計算的 effective representation 不會
包含 unknown fields。即使 destination 正確接受 `x-flTopology`，Go route state 仍會
與 destination resource 分歧。

PyMTLF 現有 PATCH 以 top-level object update 建立 effective representation。若只依
`extra="allow"`，`x-flTopology` 會被當成完整 top-level value 取代，這與 candidate
example 的完整 subtree replacement 相容；但它不會執行 recursive identity、policy
或 disabled-child validation。此外，unknown `x-retainedResultReq` 也會被 update 進
persistent representation，違反一次性 semantics。

目標 invariant：

- `x-flTopology` 在 PATCH 中是完整 subtree replacement，不做 array merge，也不與
  local candidate pool 做 JSON-level merge。
- Recipient coordinator 收到新 persistent topology 後，才依 `enabled`、priority 與
  policy 調整 local candidate／subscription state。
- `x-retainedResultReq` 與 child `retainedResultReq` 必須在 validation 後抽出為
  operation command，從 effective persistent representation 移除。
- 同一 local subscription 同時最多一個 outstanding lookup；前一個 outcome 回覆後
  才能接受下一個 lookup。
- Downstream mutation 失敗時，Go accepted representation 與 PyMTLF persistent
  contract 都不得部分更新；沿用現有 operation revision 與 PyMTLF replace rollback
  邊界。

### 4.4 DELETE

現有 Go DELETE 先鎖定 route operation，再刪除 peer 或 local PyMTLF resource；成功、
destination `404` 或已知 reset deletion 都可收斂為 local route removal。PyMTLF 對舊
hierarchy-bound resource 會取消 Branch lower process、清除 workspace 與留下短期
tombstone。

目標 invariant：

- 刪除 Intermediate 的上層 subscription 時，由該 Intermediate PyMTLF coordinator
  清除它持有的 direct-child subscriptions 與 local execution state；Go 只管理本段
  route。
- Cleanup binding 必須改由 protocol topology／local process mapping 判定，不再依
  `hierarchy_assignment` bundle 是否存在。
- 尚未完成的 retained-result lookup 隨 resource termination 結束；late outcome
  不得重新建立已刪除的 route 或 execution。

### 4.5 Notify

現有 Go Notify 先 parse typed notification、依 route 驗證 `notifCorreId`、
`mlCorreId` 與 expected `roundInd`，之後轉送原始 body。因此只要 validator 接受，
unknown properties 可以原樣到達 PyMTLF。

目前有兩個 blocker：

- 只有 `x-flTopologyReport` 或 `x-retainedResultStatus` 的合法 candidate Notify，會被
  現有 shape validator 視為缺少 `delayEventNotif`、`mLModelInfos` 或
  `termTrainReq` 而拒絕。
- PyMTLF FL Server 雖可保留 unknown properties，卻不會驗證 report root identity、
  status／cause、retained-result outcome 或把 realized topology 交給 coordinator。

目標 invariant：

- Candidate detailed information 應加入既有 Notify minimum-result 判定，但不改變
  `delayEventNotif` 與 standard result fields 的互斥規則。
- `x-flTopologyReport.nfInstanceId` 必須符合 callback resource 綁定的 direct
  participant identity。
- Topology report 是 notification evidence，不覆寫接收端 subscription contract；
  上層 coordinator 可依 report 決定後續 PUT／PATCH。
- `FOUND` retained result 必須帶 standard `roundInd` 與 `mLModelInfos`；
  `NOT_FOUND`／`FAILED` 不得帶兩者。
- 一般 training-result Notify 維持既有 expected `roundInd` equality validation。
  `x-retainedResultStatus: FOUND` 是 lookup outcome，其 `roundInd` 表示已保存結果的
  local round；route 只驗證 non-negative value、`mlCorreId`、artifact fields 與
  outstanding lookup，不與 replacement subscription 的 expected round 比較。
- Notify transport 保留 raw candidate fields，但 validation 與 consumer 必須使用同一
  typed contract，不能依 unknown-object passthrough 運作。

---

## 5. Persistent、local 與 operation-scoped state

| 資訊 | Go route | PyMTLF subscription resource | PyMTLF local process |
| --- | --- | --- | --- |
| Negotiated `HierarchicalFLOrch` | 每個 resource 保存並 gate 後續 operations | 保存 accepted feature view | 只讀取 capability，不自行重新 negotiation |
| `x-flTopology` persistent contract | Accepted／backend representation 必須一致 | 保存接收者目前有效 subtree | Coordinator 建立 direct-child execution mapping |
| `policy`／`strategy`／`reportAfter` | 作為 topology contract 保存，不執行 | 保存 resolved input／回報值 | Selection、training、aggregation 與 reporting consumer |
| `x-retainedResultReq` | 只在當次 operation 驗證與 gate，不進 accepted representation | 觸發一次 lookup，不進 representation | 保存單一 outstanding operation 直到 outcome |
| Child `retainedResultReq` | 隨 request 驗證，但不進 persistent subtree | Intermediate 抽出後對指定 child 發送一次 request | 保存 per-child outstanding mapping |
| `x-flTopologyReport` | 驗證 callback identity 後 raw 轉送 | 不屬於 Client subscription state | Parent callback collector／coordinator 保存 realized view |
| `x-retainedResultStatus` | 驗證後 raw轉送；`FOUND` 不套用 normal-result expected-round equality | 不屬於 Client subscription state | Parent lookup waiter 保存 terminal outcome |
| Local round／artifact／workspace | 只保存 expected wire `roundInd` | 保存 Client job binding | FL Server／Client process 擁有實際 state 與 cleanup |

---

## 6. Feature negotiation 邊界

`HierarchicalFLOrch` 必須逐段 negotiation。Root→Intermediate 成功不代表
Intermediate→下一層自動成功。

現有 `suppFeats` 只有 subscription wire field，Go route 沒有保存「request offered
feature」與「destination accepted feature」的 resolved state。實作時至少需要：

1. Create request 記錄 sender offered feature；
2. destination accepted representation 回傳 negotiated `suppFeats`；
3. Go route 保存該 subscription 的 negotiated feature；
4. 後續 PUT／PATCH、candidate Notify 與 retained lookup 由該 route gate；
5. Intermediate 建立下一段 subscription 時重新提出 feature，不複製上段的 accepted
   result 當成下段能力。

若對端未接受 feature，該段不得送 candidate fields。是否退回舊 model-bundle mode
只能由 execution-level compatibility policy 決定，不能由 Go route 靜默切換。

---

## 7. Type integration 必須滿足的條件

Go 與 PyMTLF 無論採手寫 model 或 code generation，都必須共用以下 observable
behavior：

- request、response、stored representation 與 Notify 使用相同 JSON names；
- recursive topology 有 depth、node count、identity uniqueness 與 cycle validation；
- status、cause、policy、strategy 與 retained-result cross-field rules 一致；
- forward-compatible enum 接受未知 string，但本版本 implementation 不得執行未知
  method／selection semantics；
- PATCH 明確區分 persistent replacement 與 operation-scoped command；
- invalid field path 可回到 `ProblemDetails.invalidParams`；
- Go accepted／backend representation 與 PyMTLF representation 不因 `200`／`204` 或
  local／peer route 而產生差異。

### 7.1 Go integration choice

第一版沿用 `NWDAF/internal/compat/mlmodeltraining`，手寫 candidate types、parse、
validation 與 PATCH application，不另外產生一套 OpenAPI Go module。理由是：

- 現有 Release 18 ML Model Training 本來就由此 compatibility package 補足 pinned
  free5GC OpenAPI 的缺口；
- handler、processor、peer consumer、private gateway 與 tests 都已依賴同一 package；
- repository 沒有針對 TS 29.520 ML Model Training 的 generation workflow；
- 另外生成 candidate module 會同時引入 shared 3GPP type轉換與 dual-model boundary，
  但不會消除 cross-field／resource-state validation。

Candidate OpenAPI artifact仍是 JSON contract與 test fixture來源。Go 手寫 type不應
複製整份 3GPP model，只擴充現有 compatibility structs與必要 nested candidate
types。PyMTLF 則在現有 `wire/ml_model_training.py` 建立對應 typed Pydantic fields，
移除依賴 unknown-property passthrough 的行為。

### 7.2 Migration authority choice

Legacy bundle與 protocol contract的切換由 Root PyMTLF orchestration明確選擇，
不由 Go route或接收端根據缺少欄位靜默猜測：

- **Legacy execution**：只產生舊 `HIERARCHY_ASSIGNMENT` bundle，不送
  `x-flTopology`。
- **Protocol execution**：Preparation 送出標準必要／training requirement fields 與
  `x-flTopology`，但不帶 model reference；Root 收到滿足 readiness 的 realized
  topology 後，第一輪／後續 round update 才以
  `mLModelAdrf` 下發 initial／global model。由下往上的 local／aggregate result 使用
  產生節點的暫存 `mLFileAddr`；Branch aggregate 若成為後續 lower-tier round 的
  輸入，也由 Branch 以同一類暫存 `mLFileAddr` 下發。Model artifact 不得再包含
  hierarchy assignment metadata。
- 同一 request若同時帶 `x-flTopology`與 hierarchy assignment artifact，接收端應
  拒絕為 ambiguous contract，不能合併或任選其中之一。
- Compatibility period先保留 legacy regression；protocol-driven E2E完成後，再在
  migration closure移除 assignment／preparation-result bundle runtime path。

Exact config key留給 detailed implementation slice，但 selector owner與互斥語意已
固定。Go NWDAF 只按收到的 message執行該段 contract，不進行 execution-level
fallback。

### 7.3 ADRF global-model lifecycle

Protocol execution 的 global-model path 重用既有 ADRF ML Model Management proxy，
但不重用 final-model catalog commit semantics：

1. Preparation subscriptions 不帶 `mLModelInfos`，也不建立 ADRF record。Root 等待
   逐級 `x-flTopologyReport` 滿足 topology readiness，取得實際建立的 participant
   identities。
2. Root PyMTLF global-round owner 發布 model-payload immutable temporary artifact，
   依 realized topology 產生該輪 `allowConsumerList`，並透過 containing Root Go
   NWDAF 向已解析的 ADRF 建立 store record。
3. Root 驗證 `201 Created`、`Location` 與 returned
   `NadrfMLModelStoreRecord`，保存該 round 對 `adrfId`、`storTransId`、
   `modelUniqueId` 的 mapping。下發的 `mLModelInfos[]` entry 同時攜帶
   `modelUniqueId`，以及包含 `adrfId`／`storTransId` 的 `mLModelAdrf`；本 profile
   不接受缺少 `storTransId` 的 global-model reference。
4. Root 以第一輪 PUT／PATCH 下發 `mLPreFlag: false`、`roundInd` 與上述
   `mLModelInfos[]`，只有成功接受 update 的 resource 才能進入 training execution。
5. Intermediate／Leaf 收到 reference 後，依 `adrfId` 解析指定 ADRF，透過自己的
   containing Go NWDAF 執行 collection GET，再驗證 record identity、model identity
   與 storage size，最後下載 record 中的 `mLFileAddr`。
6. Intermediate 以新收到的 Root global model 啟動 lower-tier round 時，對 selected
   children 的 PUT／PATCH 原樣傳遞同一 `mLModelAdrf`，不下載後 republish global
   model bytes。
7. Intermediate 完成 domain aggregation 後，若在下一次 upstream update 前繼續
   lower-tier round，則由 Intermediate 將 aggregate 發布在自己的暫存 workspace，
   以 `mLFileAddr` 下發給 selected children。該 aggregate 不建立 ADRF record，並可由
   同一暫存 reference 向 direct parent 回報。
8. Store 失敗時保留已建立的 preparation resources，但不得發出該輪 update；下游
   retrieval 或
   artifact validation 失敗時不得開始 local training，並沿既有 training delay／
   failure path回報 direct parent。
9. Topology update 若加入新的 Root global-model consumer，Root 必須先以 ADRF PUT 更新
   `allowConsumerList`，或等下一輪新 record 納入該 identity，再向該 node 下發
   reference。ACL metadata 可更新，但同一 record 的 model payload 不得改成另一輪
   model。
10. Root 保留 record，直到該輪 selected subtree 全部進入 terminal outcome且沒有
   in-flight retry，或整個 procedure terminal，再透過 ADRF DELETE 清理。Restart
   無法恢復 round-to-record mapping 時，procedure 明確失效，不以 `roundInd`
   猜測另一筆 record。

現有 testbed ADRF 沒有 authenticated caller identity，也未在 server side 強制
`allowConsumerList`。本 workstream 仍要求 Root 產生並送出正確清單；real-process
tests 驗證 request／stored record 的 ACL，而不把 retrieval success 當成 caller
authorization 已被 enforcement 的證據。

---

## 8. 後續銜接

[Protocol Conformance Case Ownership Mapping](./Protocol%20Conformance%20Case%20Ownership%20Mapping.md)
已將 candidate cases 對應到 Go wire／route、PyMTLF resource／execution consumer 與
unit／boundary／real-process test seam。依該 ownership 與 dependency 形成的工作單位
記錄於
[Protocol Extension Implementation Slice Map](./Protocol%20Extension%20Implementation%20Slice%20Map.md)。
下一步是在修改 production code 前，建立 Slice 1 detailed plan，列出 exact files、
test matrix、feature-disabled behavior 與 repository-specific verification boundary。
