# Hierarchical NWDAF FL Protocol Conformance Matrix

日期：2026-09-02

狀態：候選測試契約，待使用者審查

相關文件：

- [Hierarchical NWDAF Federated Learning 協定設計](./protocol_design.md)
- [Candidate OpenAPI Schema](./candidate_openapi_schema.md)
- [Candidate OpenAPI artifact](./candidate_openapi.yaml)

---

## 1. 文件目的

本文把既有 candidate schema 與 `x-stage3-rules` 轉成實作時可直接採用的
conformance cases。它不增加欄位或 procedure semantics；若本文與
[Candidate OpenAPI Schema](./candidate_openapi_schema.md) 不一致，以後者的
完整語意為準。

測試分成三層：

- **Schema／decoder**：JSON shape、required property、range 與 closed
  discriminator。
- **Receiver validation**：需要 subscription context、receiver identity 或
  跨欄位判斷的 request／Notify validation。
- **Procedure／state**：feature negotiation、PATCH merge、candidate
  selection、subscription cleanup 與 retained-result serialization。

`400` cases 應使用既有 `ProblemDetails` 與適合的 3GPP cause，並在
`invalidParams` 指出 extension path。Message 在 schema 上合法、但接收者無法
履行明確指定的 contract 時，使用
`403 ML_MODEL_TRAINING_REQS_NOT_MET`。

---

## 2. 基準 fixtures

實作測試可從 [`candidate_openapi.yaml`](./candidate_openapi.yaml) 的 examples
建立下列 fixtures，再只修改各 case 指定的欄位：

| Fixture | OpenAPI example | 用途 |
| --- | --- | --- |
| `SUB_VALID` | `HierarchicalTopologySubscription` | 合法 Create／PUT request |
| `PATCH_REMOVE` | `CandidateRemovalPatch` | 合法 topology replacement PATCH |
| `NOTIFY_TOPOLOGY` | `TopologyReportNotification` | 合法 topology report |
| `NOTIFY_FOUND` | `RetainedResultFoundNotification` | 合法 retained result |
| `NOTIFY_NOT_FOUND` | `RetainedResultNotFoundNotification` | 合法 lookup miss |
| `NOTIFY_FAILED` | `RetainedResultFailedNotification` | 合法 lookup failure |

OpenAPI examples 只提供最小 payload；涉及既有 subscription、negotiated
features、local candidate pool 或 outstanding lookup 的 cases，必須另外建立
表格所列的 state fixture。

---

## 3. Request 與 topology validation

| ID | Layer | Input／precondition | Expected result |
| --- | --- | --- | --- |
| `REQ-01` | Schema／decoder | `SUB_VALID` | 接受 payload，保留既有 Release 18 fields 與全部 `x-*` properties |
| `REQ-02` | Receiver validation | Create／PUT 帶 `x-flTopology` 或 `x-retainedResultReq: true`，但省略 `mlCorreId` | `400 Bad Request`；`invalidParams` 指向 `mlCorreId` |
| `REQ-03` | Receiver validation | PATCH 帶 `x-flTopology`，但既有 resource 沒有 FL procedure correlation | `400 Bad Request`；既有 resource 不得部分更新 |
| `TOP-01` | Receiver validation | `x-flTopology.nfInstanceId` 等於 request receiver，subtree identities 唯一且無 cycle | 接受 topology instruction |
| `TOP-02` | Receiver validation | Topology root identity 不等於 request receiver | `400 Bad Request`；`invalidParams` 指向 `x-flTopology.nfInstanceId` |
| `TOP-03` | Receiver validation | 同一 subtree 重複 `nfInstanceId`，包含 sibling duplicate 或 ancestor cycle | `400 Bad Request`；不得建立任何 downstream subscription |
| `TOP-04` | Receiver validation | `selectionMethod: priority`，但 enabled explicit child 沒有 `priority` | `400 Bad Request`；指向缺少 priority 的 child path |
| `TOP-05` | Receiver validation | `minAvailableNodes < minTrainNodes` | `400 Bad Request`；既有 resource 不得部分更新 |
| `TOP-06` | Receiver validation | 同一 child 同時為 `enabled: false` 與 `retainedResultReq: true` | `400 Bad Request`；不得執行 DELETE 或 lookup |
| `TOP-07` | Schema／decoder | `strategy.method: fedProx` 缺少 `methodParameters.proximalMu`，或值小於 `0` | `400 Bad Request` |
| `TOP-08` | Schema／decoder | `strategy.method` 是本版本未定義的 value | `400 Bad Request`；closed discriminator 不接受未知 method |
| `TOP-09` | Receiver validation | Instruction enum 可由 forward-compatible schema 解析，但 receiver 無法履行 | `403 Forbidden`／`ML_MODEL_TRAINING_REQS_NOT_MET`；不得靜默改寫 |

---

## 4. Policy 與 PATCH procedure

| ID | Layer | Input／precondition | Expected result |
| --- | --- | --- | --- |
| `POL-01` | Procedure／state | `activeCount >= minAvailableNodes` 且 `activeCount >= minTrainNodes` | Local FL process 可以進入本輪 selection |
| `POL-02` | Procedure／state | `activeCount < minAvailableNodes` 或 `activeCount < minTrainNodes` | 不得開始新一輪；可繼續建立候選 subscription |
| `POL-03` | Procedure／state | `fractionTrain`、`minTrainNodes` 與 active pool 已知 | 選取數量為 `min(activeCount, max(floor(activeCount * fractionTrain), minTrainNodes))` |
| `POL-04` | Procedure／state | `acceptFailures: false` 且任一 selected participant 失敗 | 不得聚合本輪結果 |
| `POL-05` | Procedure／state | `acceptFailures: true` | 只有 successful completion rate 達到 `minCompletionRate` 才可聚合 |
| `PATCH-01` | Procedure／state | `PATCH_REMOVE`，Branch 另有 locally discovered candidates | PATCH 中的 `children` 取代完整 upstream-assigned array；locally discovered pool 保留 |
| `PATCH-02` | Procedure／state | Active child 被設為 `enabled: false` | 立即排除 selection，對既有 direct-child resource 執行 DELETE，並依結果回報 `INACTIVE` cause |
| `PATCH-03` | Procedure／state | PATCH 省略某 child，但沒有留下 `enabled: false` prohibition | 移除 upstream assignment；只有 policy 允許 additional candidates 時，該 identity 才可由 local discovery 再加入 |
| `SCOPE-01` | Procedure／state | Node 同時帶 `policy`、`strategy` 與 `reportAfter` | `policy`／`strategy` 只作用於其 direct-child local FL process；`reportAfter` 只作用於該 node 對 direct parent 的回報，三者都不自動向 descendants 繼承 |

---

## 5. Notify 與 topology report validation

| ID | Layer | Input／precondition | Expected result |
| --- | --- | --- | --- |
| `NOT-01` | Receiver validation | `NOTIFY_TOPOLOGY`，callback identity、report root 與 `mlCorreId` 均匹配 | 接受並回覆 `204 No Content` |
| `NOT-02` | Receiver validation | Notification 沒有既有 detailed information，也沒有 `x-flTopologyReport` 或 `x-retainedResultStatus` | `400 Bad Request` |
| `NOT-03` | Receiver validation | Notification 帶 extension report，但缺少 `mlCorreId` | `400 Bad Request`；指向 `mlCorreId` |
| `NOT-04` | Receiver validation | `x-flTopologyReport.nfInstanceId` 不等於 callback subscription 綁定的 direct Client | `400 Bad Request` |
| `NOT-05` | Receiver validation | Report subtree 有 duplicate identity 或 ancestor cycle | `400 Bad Request`；不得部分套用 status |
| `NOT-06` | Receiver validation | `FAILED`／`INACTIVE` node 缺少 `statusCause` | `400 Bad Request` |
| `NOT-07` | Receiver validation | `UNCONFIRMED`／`DEPLOYING`／`ACTIVE` node 帶有 `statusCause` | `400 Bad Request` |
| `NOT-08` | Procedure／state | Report 使用未知 forward-compatible status／cause | 可以解析與轉送；不得把未知 status 推測為 `ACTIVE`，也不得觸發已知 cause 的特定 recovery action |
| `NOT-09` | Receiver validation | `delayEventNotif` 與 `mLModelInfos` 或 `termTrainReq` 同時出現 | `400 Bad Request`；既有 Release 18 mutual-exclusion rule 不因 extension 放寬 |

---

## 6. Retained-result lookup

| ID | Layer | Input／precondition | Expected result |
| --- | --- | --- | --- |
| `RET-01` | Receiver validation | `NOTIFY_FOUND` | 接受；`roundInd` 與 `mLModelInfos` 共同表示最新已完成 local result |
| `RET-02` | Receiver validation | `x-retainedResultStatus: FOUND` 缺少 `roundInd` 或 `mLModelInfos` | `400 Bad Request` |
| `RET-03` | Receiver validation | `NOTIFY_NOT_FOUND` 或 `NOTIFY_FAILED` | 接受；不得推動新一輪 training |
| `RET-04` | Receiver validation | `NOT_FOUND`／`FAILED` 同時帶 `roundInd` 或 `mLModelInfos` | `400 Bad Request` |
| `RET-05` | Procedure／state | Create／PUT／PATCH 帶 `x-retainedResultReq: true`，且沒有 outstanding lookup | 只啟動一次 lookup；instruction 不保存為 resource state |
| `RET-06` | Procedure／state | Operation 省略 `x-retainedResultReq` 或設為 `false` | 不啟動 lookup |
| `RET-07` | Procedure／state | 前一個 lookup 尚未收到 outcome | Consumer 不得發出下一個 lookup operation；timeout 不解除 outstanding state |
| `RET-08` | Procedure／state | 收到 `FOUND`、`NOT_FOUND`、`FAILED` 或未知 forward-compatible outcome | 結束 outstanding lookup；只有 `FOUND` 可使用 model result，之後才可再次要求 lookup |

---

## 7. Feature negotiation

| ID | Layer | Input／precondition | Expected result |
| --- | --- | --- | --- |
| `FEAT-01` | Schema／procedure | Initial POST 同時帶 `suppFeats: "4"` 與 extension properties | Request 可進入正常 validation，不因 feature 尚未確認而先拒絕 |
| `FEAT-02` | Consumer procedure | 成功 response 的 negotiated `suppFeats` 包含 feature 3 | 此 local resource 後續可使用 hierarchy PUT／PATCH／Notify semantics |
| `FEAT-03` | Consumer procedure | 成功 response 未包含 feature 3，而 hierarchy 是此次 subscription 必要條件 | 不得把 resource 視為 hierarchical resource；刪除 resource，並回報 `FAILED`／`FEATURE_NOT_SUPPORTED` |
| `FEAT-04` | Procedure／state | Root–Intermediate edge 已協商成功，但 Intermediate–Client edge 尚未協商 | 不得繼承上游結果；每一條 parent-to-child subscription edge 必須獨立協商 |

---

## 8. 實作採用方式

進入 `NWDAF` implementation mapping 後，應把每個 case 指派到確切 owner：

- JSON model／decoder tests；
- public SBI request validation；
- Notify consumer validation；
- subscription resource／negotiated-feature state；
- participant selection 與 retained-result procedure state。

同一 case 若跨越 receiver 與 consumer boundary，必須拆成各方向的 focused
test；mocked handler test 不代表已完成跨 NWDAF integration validation。
