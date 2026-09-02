# Hierarchical NWDAF FL 標準欄位與 Extension 邊界

日期：2026-09-02

狀態：Release 18 欄位與 procedure mapping 已完成第一輪查核；Release 19／20
差異已確認至 `Nnwdaf_MLModelTraining` OpenAPI 與 feature table；candidate
schema 待使用者審查

上層文件：

- [Hierarchical NWDAF Federated Learning 協定設計](./protocol_design.md)

相關細節：

- [Topology、policy 與 strategy 細節設計](./topology_policy_design.md)
- [Candidate OpenAPI Schema](./candidate_openapi_schema.md)

---

## 1. 文件目的

本文逐項對照 hierarchical NWDAF FL 的資訊需求與既有 3GPP
`Nnwdaf_MLModelTraining` contract，區分：

1. 已由 3GPP 定義、可直接重用的資訊。
2. 既有 mechanism 能執行，但 message 沒有承載跨層指令的資訊。
3. Hierarchical orchestration 確實缺少、需要 proposed extension 的資訊。
4. 應保留在 implementation／operator policy，而不應加入 protocol 的資訊。

這份文件固定 extension boundary，不重複展開完整 OpenAPI component schemas；
對應 mapping 見 [Candidate OpenAPI Schema](./candidate_openapi_schema.md)。
Candidate schema 不得把既有 task、data、model、deadline 或 correlation
資訊重新包進 generic custom object。

---

## 2. 查核基準與判定方式

主要基準為：

- TS 23.288 Release 18 V18.13.0 §6.2C.2.1 至 §6.2C.2.3，以及 §6.2F.2。
- TS 29.520 Release 18 V18.14.0 §4.6.2.2.2、§4.6.2.4.2、§5.5.6.2、
  §5.5.7.3 與 §5.5.8。
- TS 29.500 Release 18 §5.2.7.2 與 §6.6.2。
- `TS29520_Nnwdaf_MLModelTraining.yaml` Release 18 API 1.0.5、Release 19
  API 1.1.1、Release 20 API 1.2.0-alpha.1。

判定時分開檢查：

- 欄位是否存在於 OpenAPI schema。
- TS procedure 是否定義 producer、consumer、條件與行為。
- 現有語意是否真的等同於 hierarchical requirement，而不是只有名稱相近。
- 資訊是否能從 authoritative producer 傳到實際 decision owner。

Release 18 是目前主要 service-side baseline；Release 19／20 用來避免把較新
release 已新增的能力誤判為 gap。

---

## 3. Subscription 中可直接重用的標準資訊

### 3.1 Training task、filter 與 model interoperability

| 需求 | 既有欄位 | 已確認語意 | 邊界判定 |
| --- | --- | --- | --- |
| Analytics／training task | `mLEventSubscs[].mLEvent` | 識別要訓練模型的 Analytics ID／event | 直接重用，不新增 `jobDescription` |
| Training filter | `mLEventSubscs[].mLEventFilter` | 表達該 Analytics 的 filter，例如 S-NSSAI、Area of Interest 等 event-specific 條件 | 直接重用，不在 policy 內複製 filter |
| Model interoperability | `mLEventSubscs[].modelInterInfo` | TS 29.520 Model Training subscription 要求每個 event 提供 interoperability information；其值的格式由 vendors 協議 | 直接重用；不能把它誤當作 FL method／aggregation contract |
| Target UE | `tgtRepUe` | 指定 training data 所對應的 UE 或 UE group | 直接重用 |
| Model use context | `mLModelInfos[].useCaseCxt` | 可隨 ML Model Information 表達 model context；其值格式不由 3GPP 標準化 | 視 model information 使用，不新增 hierarchy-specific job context |

`MLEventSubscription.useCaseCxt` 雖存在於共用 type，但 TS 29.520
§5.5.6.2.2 明定它不適用於 Model Training subscription。若需要 use case
context，應使用該 procedure 允許的 `mLModelInfos[].useCaseCxt`，不能只因
OpenAPI component 中看得到 property 就直接放進 `mLEventSubscs`。

### 3.2 Data 與 time requirement

`mLModelTrainInfos[]` 中的 `MLModelTrainInfo` 已提供 preparation 所需的
training requirements：

| 需求 | 既有欄位 | 已確認語意 |
| --- | --- | --- |
| Training input events | `dataAvReq.inpEvents[]` | 要為 local model training 收集的 event IDs |
| Dataset properties | `dataAvReq.dataStatProps[]` | Dataset statistical properties |
| Minimum samples | `dataAvReq.minNumSamples` | 可接受的最少 training samples |
| Sample time windows | `dataAvReq.timeWindows[]` | Training samples 的時間區間 |
| FL availability time | `timeAvReq` | Client 可供此次 model training 使用的時間要求 |

TS 29.520 §5.5.6.2.5 要求 `mLPreFlag: true` 時提供 `dataAvReq` 與
`timeAvReq`。TS 23.288 §6.2C.2.1 也明確把這些資訊用於 FL Client
preparation／selection。

因此 policy 中若需要「具有何種資料的 Client」，應引用或使用上述標準
training requirements 形成 eligible candidate pool，不重新定義
`requiredEvents`、`minimumSamples` 或另一組 time requirement。

Resource／load requirement 不同：TS 23.288 允許 Client 依 computation、
communication capability 或 operator policy 判斷是否參與，也允許 Server 在
reselection 時考慮 load；但目前 `NwdafMLModelTrainSubsc` 沒有通用的
resource threshold object。此類資訊的 authoritative source 可能是 NRF
profile、analytics、local configuration 或 Client admission decision，不能在
尚未確認來源與傳遞路徑前直接新增 generic `resourceCriteria`。

### 3.3 Model、training lifecycle 與 reporting

| 需求 | 既有欄位／procedure | 邊界判定 |
| --- | --- | --- |
| Preparation／execution | `mLPreFlag` | 直接重用 |
| Initial／global／local model | `mLModelInfos[]`，其中使用 `mlFile`、`mLFileAddr` 或 `mLModelAdrf` | 直接重用；model artifact 不放入 topology extension |
| Model accuracy check | `mLAccChkFlg` | 直接重用 |
| Maximum response time | `mLTrainRepInfo.maxResTime` | 直接重用，表示等待 training result 的 deadline |
| Current local process round | `roundInd` | 直接重用；不以 topology report 複製 descendants 的 rounds |
| Skip current round | `skipFlInd` | 直接重用 |
| Reporting mode／expiry | `eventReq` | 直接重用 |
| Optional feature negotiation | `suppFeats` | 直接重用 `SupportedFeatures`；不新增 hierarchy-specific capability property |
| FL procedure correlation | `mlCorreId` | 直接重用；hierarchy-wide reuse 是本設計補充的 correlation semantics |
| Callback correlation | `notifUri`、`notifCorreId` | 每個 local subscription 直接重用 |

`reportAfter` 仍是獨立 extension requirement。`roundInd` 表示目前 training
round，`maxResTime` 表示等待結果的 deadline；兩者都沒有表達「此 node 完成
多少 local epochs 或 lower-tier rounds 後才向 parent 回報一次」。

### 3.4 `mlCorreId` 與 local process correlation

TS 23.288 與 TS 29.520 Release 18 至 Release 20 對 `mlCorreId` 的核心定義
一致：它識別訓練 ML Model 的 Federated Learning procedure。TS 29.520 的
`NwdafMLModelTrainSubsc` 與 `NwdafMLModelTrainNotif` 都包含此欄位；
structured data type 將它定義為 `string`、cardinality `0..1`，並要求 service
用於 Federated Learning 時提供。

| Release | 查核版本 | Subscription | Notify | 定義變化 |
| --- | --- | --- | --- | --- |
| Release 18 | TS 29.520 V18.14.0 | `mlCorreId` | `mlCorreId` | 無 |
| Release 19 | TS 29.520 V19.7.0 | `mlCorreId` | `mlCorreId` | 無 |
| Release 20 | TS 29.520 V20.0.0 | `mlCorreId` | `mlCorreId` | 無 |

三個 release 都沒有替 `mlCorreId` 定義全域唯一性、namespace，或限制它只能
對應一個 Model Training subscription resource；也沒有明文定義多個
subscriptions 共用同一值的 hierarchical binding 行為。因此既有規格沒有
否定以同一 `mlCorreId` 關聯多個 local FL processes，但 hierarchy-wide reuse
仍是本設計補充的 correlation semantics，不能宣稱為 3GPP 已定義的 procedure。

`notifCorreId` 與 subscription resource ID 繼續識別各 local subscription
及其 callback，避免讓共用的 `mlCorreId` 取代 subscription lifecycle
identity。三個 release 的 `NwdafMLModelTrainSubscPatch` 都沒有
`mlCorreId`，因此 partial update 不以此欄位改變既有 FL procedure
correlation。`roundInd` 表示 local process round，規格沒有定義其與
`mlCorreId` 的一對一關係，也不要求 hierarchical tiers 的 rounds 同步。

---

## 4. Notify 與 lifecycle 中可直接重用的標準資訊

### 4.1 Training result、delay 與 termination

`NwdafMLModelTrainNotif` 已能承載：

- `mLModelInfos[]`：local／interim model information 或 artifact reference。
- `mlCorreId` 與 `roundInd`：procedure 與該 local process round 的關聯。
- `statusReport`：model accuracy 與 training input data information。
- `delayEventNotif`：無法在 `maxResTime` 內完成、原因與預估完成時間。
- `termTrainReq`：FL Client NWDAF 以 Notify 向 FL Server NWDAF 要求終止
  subscription 及其原因。

這些欄位處理單一 FL Server／Client subscription 的 training output 與
lifecycle event，不應在 `x-flTopologyReport` 中建立另一份 model、round、
delay 或 termination payload。

TS 29.520 Release 18 §5.5.6.2.8 另有重要條件：通知至少要包含
`delayEventNotif`、`mLModelInfos` 或 `termTrainReq` 之一。現有
`statusReport` 單獨存在也不能滿足這個條件。因此後續若允許只回報 topology
establishment result，必須在正式 extension procedure 中定義
`x-flTopologyReport` 如何成為合法的 notification detailed information；不能
假設加入 vendor property 後就自然取代既有條件。

### 4.2 既有 status 不等同於 topology status

`statusReport` 的名稱看似接近 topology status，但其內容是
`mlModelAcc`／training input data information；Release 20 另加入
`mlModelLoss` 與 `modelMetric`。它不表示 candidate 是否已被嘗試、
subscription 是否建立成功，或 node 是否仍屬於 realized topology。

同樣地：

- `failEventReports` 是 subscription request 中 individual events 未成功及其
  原因，不是 recursive nodes 的 establishment result。
- `delayEventNotif` 表示已參與 Client 的 training delay，不是
  `UNCONFIRMED`／`DEPLOYING`／`ACTIVE` topology lifecycle。
- `termTrainReq` 表示 Client 主動要求終止 local subscription，不會自動形成
  Root 可見的整棵 realized topology。

因此逐級回報 node status、status owner、timestamp，以及 `FAILED`／`INACTIVE`
relationship 的分類原因仍需要 `x-flTopologyReport`。Direct Client 的 local
training 結果與 termination request 繼續使用標準欄位；parent 將
`termTrainReq` 轉換成 recursive topology report 時，`statusCause` 保留
`NWDAF_OVERLOAD` 或 `NOT_AVAILABLE_ML_TRAIN` 的既有 value，讓更上層能理解
relationship 為何轉為 `INACTIVE`，但不在 report 中複製 termination payload。
既有 `termTrainReq: OTHERS` 則映射為 candidate
`statusCause: OTHER`。

### 4.3 Request rejection 可重用的標準行為

TS 29.500 §5.2.7.2 已定義 schema-invalid JSON IE 的拒絕方式。若 extension
property 的 type、closed discriminator、range、required condition 或跨欄位
規則不合法，使用 `400 Bad Request` 與適合的既有 cause，並在
`ProblemDetails.invalidParams` 指出 extension path，不需要新增 hierarchical
error object。

TS 29.520 §5.5.7.3 已定義 `403 Forbidden`／
`ML_MODEL_TRAINING_REQS_NOT_MET`，並要求 `invalidParams` 指出未滿足的
training requirement。因此 message 合法但 Client 無法履行上層指定的
node-wide `policy`、`strategy` 或 `reportAfter` contract 時，可重用此
application error；candidate Stage 3 procedure 必須明訂其也適用於這些
extension requirements，不能宣稱 Release 18 已自動涵蓋 custom fields。
`failEventReports` 只適合部分 `mLEventSubscs` 未成功，不能替代整個 node
的 contract rejection。

PUT／PATCH 只有在成功處理及接受後才修改、保存 subscription；error response
不形成部分 hierarchical update，既有 subscription state 維持不變。

---

## 5. Procedure 已提供、但缺少跨層指令的能力

TS 23.288 Release 18 已讓 FL Server NWDAF：

- 透過 NRF discovery／selection 找到 FL Client NWDAFs。
- 以 preparation request 檢查 Client 是否符合 data、time、model
  interoperability 等 requirement。
- 在 execution phase reselect、add 或 remove Clients。
- 依 deadline、Client delay 或 local configuration 決定等待、skip 或聚合已
  收到的結果。

這表示「Server 可以選 Client」、「Client 可以離開」以及「Server 可以接受
部分已收到的結果」本身不是新能力。真正缺少的是上層如何把 decision
responsibility 與 participant policy 傳給 Intermediate，以及 Intermediate
如何把 realized result 逐級回報。

| Hierarchical requirement | 既有 mechanism | 為何仍需 extension |
| --- | --- | --- |
| Intermediate 自行加入未明列的 candidates | Server 本來可以自行 discovery／selection | 上層沒有欄位授權或限制 subordinate Server 的 selection scope |
| Priority／random selection | Selection algorithm 可由 implementation／operator policy 決定 | 上層沒有跨層傳遞指定 algorithm 或 candidate ordering 的欄位 |
| `minAvailableNodes` | Preparation 可逐一確認 Client | 沒有 local FL process ready threshold |
| `fractionTrain`／`minTrainNodes` | Server 可自行決定每輪 participants | 沒有上層傳給 Intermediate 的 structured round-selection contract |
| `acceptFailures`／`minCompletionRate` | Server 可等待全部或聚合已收到結果 | 既有 procedure 將 decision 留給 Server local configuration，沒有跨層傳遞 threshold |

這些欄位標準化的是 hierarchical delegation contract，不是重新創造 FL
Server 原本就有的 selection／aggregation 能力。

---

## 6. Candidate extension 的 minimum boundary

### 6.1 確實需要的新資訊

| Proposed information | 新增理由 | 可重用的標準型別／欄位 |
| --- | --- | --- |
| `x-flTopology` | `NwdafMLModelTrainSubsc` 沒有 recursive parent／child instruction | Node identity 重用 `NfInstanceId` |
| `children[]` | 沒有逐級傳遞 candidate subtree 的結構 | 每個 child 的 identity 重用 `NfInstanceId` |
| `enabled` | 只省略 child 或降低 priority 無法明確禁止 Intermediate 使用及重新發現該 candidate | `false` 時重用既有 Unsubscribe HTTP DELETE 移除已存在的 direct-child subscription |
| `priority` | 沒有跨層 candidate ordering | 新語意 |
| `policy` | 沒有 subordinate Server 的 selection authority、participant thresholds 與 failure acceptance contract | Data／time eligibility 繼續使用 `mLModelTrainInfos` |
| `strategy` | 沒有 FL method、aggregation rule 與 typed method parameters | Model interoperability 不能替代此 contract |
| `reportAfter` | 沒有 node-local work-to-upstream-report cadence | `roundInd`／`maxResTime` 不等同此語意 |
| `x-flTopologyReport` | 沒有 recursive realized topology、逐級 node lifecycle report，以及 `FAILED`／`INACTIVE` relationship 的分類原因 | Node identity 重用 `NfInstanceId`；timestamp 重用 `DateTime`；`statusCause` 不取代 direct-child `termTrainReq` 或 HTTP `ProblemDetails.cause` |
| `retainedResultReq` | 上層沒有欄位可要求 Intermediate 對指定 direct child 執行 retained-result lookup | 作為 `x-flTopology` child node 的一次性 instruction，由 parent 映射成下游 operation 的 `x-retainedResultReq` |
| `x-retainedResultReq` | 新 FL Server 建立或更新 subscription 時，沒有標準欄位可明確要求 Client 查找同一 FL procedure 的最新保留結果 | Operation-scoped 一次性 trigger；結果重用既有 immediate report 或 Notify 中的 `roundInd` 與 `mLModelInfos` |
| `x-retainedResultStatus` | 標準 notification 沒有明確區分 retained result 已找到、已完成查找但不存在，或已接受的 lookup 後續執行失敗 | `FOUND` 時重用 `roundInd` 與 `mLModelInfos`；`NOT_FOUND`／`FAILED` 不建立空 model payload |

Capability negotiation 不在這張表新增 property。`NwdafMLModelTrainSubsc`
已包含 `suppFeats`，candidate 直接用它協商整組 hierarchical orchestration
semantics。Feature number 採 `3`，以避開 Release 19 feature 1
`UnsubscribeWithInfo` 與 Release 20 feature 2 `EnAccuracyInformation`；只宣告
feature 3 時的 bitmask 為 `"4"`。每一個 parent-to-child subscription resource
都必須獨立協商，不能把 upstream negotiation 結果視為 downstream 已支援。
若成功建立的 resource 沒有協商到 feature 3，consumer 不得使用 extension
procedure；hierarchy 是必要條件時，刪除該 resource 並以
`FAILED`／`FEATURE_NOT_SUPPORTED` 回報 candidate establishment result。

`x-flTopology` 最外層 identity 必須等於 request receiver，
`x-flTopologyReport` wrapper identity 必須等於 subscription callback context
中的 direct Client。這項 binding 與 subtree identity uniqueness 屬於
hierarchical procedure validation，不由 `NfInstanceId` 型別本身保證。

Candidate vocabulary 採用 Release 18 enumeration data types 相同的
forward-compatible `anyOf` pattern。這只允許舊 schema 解析未來值，不表示
舊 receiver 必須執行未知 instruction；無法履行的 selection、aggregation 或
report unit 仍以既有 `403 ML_MODEL_TRAINING_REQS_NOT_MET` 拒絕。
`strategy.method` 因為綁定 typed parameters，不採 generic future string。

### 6.2 `x-retainedResultReq` 與既有回報方式

在 Model Training message level，本設計新增 request-side
`x-retainedResultReq` 與 report-side `x-retainedResultStatus`。上層若要指示
Intermediate 對特定 direct child 使用此 trigger，則在該 `x-flTopology`
child node 使用 `retainedResultReq`。查詢 key 重用 subscription 的 `mlCorreId`；
`FOUND` outcome 重用 `roundInd` 與 `mLModelInfos` 承載結果，`NOT_FOUND`／
`FAILED` 不建立空 model payload。`FAILED` 只表示 request 已被接受後，lookup
本身執行失敗；request 接受前的錯誤仍由 Create／PUT／PATCH 的 HTTP response
表達。結果可以透過既有 `immReport` 或 Notify 傳遞，不需要增加 service
operation。

兩個 request-side properties 都是 operation-scoped 的一次性 instruction。
Create、PUT 或 PATCH 在當次攜帶 `true` 時只觸發一次 lookup；省略或
設為 `false` 不觸發。這個值不作為持續 subscription state，若日後需要
重新查詢，Server 必須在新的 operation 再次攜帶 `true`。
每個 local subscription 同時最多只有一個 outstanding lookup；前一次
lookup outcome 尚未透過 immediate report 或 Notify 回報時，Server 不得再對
同一 subscription 發出新的 lookup request。只有收到前一次
`FOUND`／`NOT_FOUND`／`FAILED` 或未知 outcome 後，才能開始下一次。等待
outcome timeout 時，該 lookup 仍視為 outstanding；Server 可繼續等待或終止
目前 subscription，但不能另起新的 outstanding lookup。本版本只使用
`FOUND`／`NOT_FOUND`／`FAILED`；未知的
forward-compatible outcome 結束該次 lookup，但不形成可用結果。因為 requests
不並行，可繼續以既有 subscription resource 與 `notifCorreId` 關聯結果，不
增加另一個 request ID。

由於 TS 29.520 Release 18 要求 Notify 至少包含 `delayEventNotif`、
`mLModelInfos` 或 `termTrainReq` 之一，正式 extension 必須將
`x-retainedResultStatus` 納入合法 notification detailed information，才能
單獨表達 `NOT_FOUND` 或 `FAILED`。完整 lookup procedure 與 HTTP examples 見
[Topology、policy 與 strategy 細節設計](./topology_policy_design.md)。
任何攜帶 `x-flTopologyReport` 或 `x-retainedResultStatus` 的 Notify 亦必須
提供 `mlCorreId`，讓 extension report 明確關聯到 hierarchy-wide procedure；
`notifCorreId` 仍負責 local callback／subscription correlation。

### 6.3 不新增或不放進 extension 的資訊

第一版不新增：

- `jobDescription`、Analytics ID、training filter、data／time requirement。
- model bytes、model URL、ADRF reference 或 model identifier。
- 另一組 global／local `roundInd`。
- 另一組 procedure ID、callback correlation ID 或 subscription resource ID。
- training delay、expected completion time 或另一份 local termination cause
  payload。
- 未知來源的 generic resource criteria。
- Root 內部如何計算 candidate reputation／priority 的方法。

這些資訊不是已有標準欄位，就是屬於 implementation／operator policy，或尚未
具備完整 producer-to-consumer flow。

---

## 7. Release 18 至 Release 20 差異

| Release | `Nnwdaf_MLModelTraining` 相關變化 | 對本設計的影響 |
| --- | --- | --- |
| Release 18 | 既有 subscription、PATCH、Notify、data／time requirement、round、delay、termination request 與 `suppFeats`；feature table 尚無 optional feature | 目前主要 service-side baseline |
| Release 19 | 新增 `/subscriptions/{subscriptionId}/unsubscribe-info` 與 feature 1 `UnsubscribeWithInfo` | Server unselect、suspend 或 finish local Client subscription 不需自行建立新的 termination payload；仍沒有 hierarchical topology report |
| Release 20 | `StatusReportInfo` 新增 `mlModelLoss` 與 `modelMetric`；新增 feature 2 `EnAccuracyInformation` | 擴充 training outcome report，不提供 recursive node lifecycle、policy 或 strategy |

Release 19／20 的 `NwdafMLModelTrainSubsc`、`NwdafMLModelTrainSubscPatch`
與 `NwdafMLModelTrainNotif` 仍沒有 recursive topology、node-scoped policy、
strategy 或 `reportAfter`。因此目前 candidate extension 的核心 gap 沒有被較新
release 取代。

若 implementation 必須嚴格維持 Release 18 contract，Server 只能使用該
release 已有的 unsubscribe／DELETE 行為，不能直接把 Release 19
`TrainingUnsubscribeInfo` 當成 Release 18 欄位。本文記錄 Release 19 能力是為了
界定版本差異；candidate schema 維持 Release 18 message baseline，並為避免與
後續 release 已配置的 feature bits 衝突，將 `HierarchicalFLOrch` 配置為候選
feature number 3。

---

## 8. 下一步

1. 以本文件作為 [Candidate OpenAPI Schema](./candidate_openapi_schema.md) 的
   review boundary，確認沒有重複建立既有標準資訊。
2. 審查 POST／PUT、PATCH 與 Notify direction 的 producer、consumer、required
   condition、validation，以及 topology-only Notify 的 procedure extension。
3. 設計確認後，視需要將 candidate fragments 整合成獨立 OpenAPI YAML。

---

## 9. 證據來源

- [TS 23.288 Release 18 §6.2C Federated Learning among Multiple NWDAFs](../../../specs/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2C%20Federated%20Learning%20among%20Multiple%20NWDAFs.md)
- [TS 23.288 Release 18 §6.2F ML Model Training](../../../specs/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2F%20Procedure%20for%20ML%20Model%20Training.md)
- [TS 29.520 Release 18 §4.6 Nnwdaf_MLModelTraining Service](../../../specs/TS%2029.520/4%20Services%20offered%20by%20the%20NWDAF/4.6%20Nnwdaf_MLModelTraining%20Service.md)
- [TS 29.520 Release 18 §5.5.6 Data Model](../../../specs/TS%2029.520/5%20API%20Definitions/5.5%20Nnwdaf_MLModelTraining%20Service%20API/5.5.6%20Data%20Model.md)
- [TS 29.520 Release 18 §5.5.7 Error handling](../../../specs/TS%2029.520/5%20API%20Definitions/5.5%20Nnwdaf_MLModelTraining%20Service%20API/5.5.7%20Error%20handling.md)
- [TS 29.520 Release 18 §5.5.8 Feature negotiation](../../../specs/TS%2029.520/5%20API%20Definitions/5.5%20Nnwdaf_MLModelTraining%20Service%20API/5.5.8%20Feature%20negotiation.md)
- [TS 29.500 Release 18 §6.6 Extensibility Mechanisms](../../../specs/TS%2029.500/6%20General%20Functionalities%20in%20Service%20Based%20Architecture/6.6%20Extensibility%20Mechanisms.md)
- [TS 29.500 Release 18 §5.2.7.2 NF as HTTP Server](../../../specs/TS%2029.500/5%20Protocols%20Over%20Service%20Based%20Interfaces/5.2%20HTTP%20and%202%20Protocol/5.2.7%20HTTP%20status%20codes/5.2.7.2%20NF%20as%20HTTP%20Server.md)
- [TS 29.520 Release 18 Nnwdaf_MLModelTraining OpenAPI](../../../specs/openapi/TS29520_Nnwdaf_MLModelTraining.yaml)
- [TS 29.520 Release 18 Nnwdaf_MLModelProvision OpenAPI](../../../specs/openapi/TS29520_Nnwdaf_MLModelProvision.yaml)
- [3GPP official Release 18 `TS29520_Nnwdaf_MLModelTraining.yaml`](https://forge.3gpp.org/rep/all/5G_APIs/-/blob/REL-18/TS29520_Nnwdaf_MLModelTraining.yaml)
- [3GPP official Release 19 `TS29520_Nnwdaf_MLModelTraining.yaml`](https://forge.3gpp.org/rep/all/5G_APIs/-/blob/REL-19/TS29520_Nnwdaf_MLModelTraining.yaml)
- [3GPP official Release 20 `TS29520_Nnwdaf_MLModelTraining.yaml`](https://forge.3gpp.org/rep/all/5G_APIs/-/blob/REL-20/TS29520_Nnwdaf_MLModelTraining.yaml)
- [TS 29.520 Release 19 V19.7.0](https://www.etsi.org/deliver/etsi_ts/129500_129599/129520/19.07.00_60/ts_129520v190700p.pdf)
- [TS 29.520 Release 20 V20.0.0 source](https://www.3gpp.org/ftp/Specs/archive/29_series/29.520/29520-k00.zip)

---

## 10. 變更紀錄

| 日期 | 內容 |
| --- | --- |
| 2026-09-02 | 建立標準欄位與 proposed extension 邊界；完成 Release 18 request／Notify mapping，並納入 Release 19 unsubscribe-info 與 Release 20 status report 差異。 |
| 2026-09-02 | 加入 retained-result extension boundary：上層以 topology child node 的 `retainedResultReq` 指示 Intermediate，downstream request 使用 `x-retainedResultReq`，report 使用 `x-retainedResultStatus`，並標示既有 `mlCorreId`、`roundInd`、`mLModelInfos`、`immReport` 與 Notify 的重用位置。 |
| 2026-09-02 | 確認重用 `suppFeats` 並使用 candidate feature 3；補入 Release 18 `400`／`403` rejection mapping，以及 Release 19／20 feature numbering。 |
| 2026-09-02 | 補充 `statusCause` 的 extension boundary：只分類 `FAILED`／`INACTIVE` topology relationship，direct-child termination 與 HTTP rejection 仍分別使用既有 `termTrainReq` 與 `ProblemDetails.cause`。 |
| 2026-09-02 | 補充 identity binding、Notify `mlCorreId`、unsupported feature cause、`termTrainReq: OTHERS` mapping，以及 Release 18-style forward-compatible enum boundary。 |
| 2026-09-02 | Retained-result outcome 加入 `FAILED`，並確認同一 subscription 的 lookup 必須序列化；只有收到前一次 outcome 後才能開始下一次，不增加 request ID。 |
