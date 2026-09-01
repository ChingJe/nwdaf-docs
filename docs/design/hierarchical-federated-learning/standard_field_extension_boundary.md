# Hierarchical NWDAF FL 標準欄位與 Extension 邊界

日期：2026-09-02

狀態：Release 18 欄位與 procedure mapping 已完成第一輪查核；Release 19／20
差異已確認至 `Nnwdaf_MLModelTraining` OpenAPI

上層文件：

- [Hierarchical NWDAF Federated Learning 協定設計](./protocol_design.md)

相關細節：

- [Topology、policy 與 strategy 細節設計](./topology_policy_design.md)

---

## 1. 文件目的

本文逐項對照 hierarchical NWDAF FL 的資訊需求與既有 3GPP
`Nnwdaf_MLModelTraining` contract，區分：

1. 已由 3GPP 定義、可直接重用的資訊。
2. 既有 mechanism 能執行，但 message 沒有承載跨層指令的資訊。
3. Hierarchical orchestration 確實缺少、需要 proposed extension 的資訊。
4. 應保留在 implementation／operator policy，而不應加入 protocol 的資訊。

這份文件先固定 extension boundary，不在此階段完成正式 OpenAPI component
schemas。後續 schema 不得把既有 task、data、model、deadline 或 correlation
資訊重新包進 generic custom object。

---

## 2. 查核基準與判定方式

主要基準為：

- TS 23.288 Release 18 V18.13.0 §6.2C.2.1 至 §6.2C.2.3，以及 §6.2F.2。
- TS 29.520 Release 18 V18.14.0 §4.6.2.2.2、§4.6.2.4.2 與
  §5.5.6.2。
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

因此逐級回報 node status、status owner 與 timestamp 仍需要
`x-flTopologyReport`；但 local training 結果與 termination cause 應繼續重用
標準欄位。

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
| `priority` | 沒有跨層 candidate ordering | 新語意 |
| `policy` | 沒有 subordinate Server 的 selection authority、participant thresholds 與 failure acceptance contract | Data／time eligibility 繼續使用 `mLModelTrainInfos` |
| `strategy` | 沒有 FL method、aggregation rule 與 typed method parameters | Model interoperability 不能替代此 contract |
| `reportAfter` | 沒有 node-local work-to-upstream-report cadence | `roundInd`／`maxResTime` 不等同此語意 |
| `x-flTopologyReport` | 沒有 recursive realized topology 與逐級 node lifecycle report | Node identity 重用 `NfInstanceId`；timestamp 重用 `DateTime` |
| `retainedResultReq` | 上層沒有欄位可要求 Intermediate 對指定 direct child 執行 retained-result lookup | 作為 `x-flTopology` child node instruction，由 parent 映射成下游 subscription 的 `x-retainedResultReq` |
| `x-retainedResultReq` | 新 FL Server 建立 subscription 時，沒有標準欄位可明確要求 Client 查找同一 FL procedure 的最新保留結果 | Trigger 為新欄位；結果重用既有 immediate report 或 Notify 中的 `roundInd` 與 `mLModelInfos` |
| `x-retainedResultStatus` | 標準 notification 沒有明確區分 retained result 已找到或已完成查找但不存在 | `FOUND` 時重用 `roundInd` 與 `mLModelInfos`；`NOT_FOUND` 不建立空 model payload |

### 6.2 `x-retainedResultReq` 與既有回報方式

在 Model Training message level，本設計新增 request-side
`x-retainedResultReq` 與 report-side `x-retainedResultStatus`。上層若要指示
Intermediate 對特定 direct child 使用此 trigger，則在該 `x-flTopology`
child node 使用 `retainedResultReq`。查詢 key 重用 subscription 的 `mlCorreId`；
`FOUND` outcome 重用 `roundInd` 與 `mLModelInfos` 承載結果，`NOT_FOUND`
不建立空 model payload。結果可以透過既有 `immReport` 或 Notify 傳遞，
不需要增加 service operation。

由於 TS 29.520 Release 18 要求 Notify 至少包含 `delayEventNotif`、
`mLModelInfos` 或 `termTrainReq` 之一，正式 extension 必須將
`x-retainedResultStatus` 納入合法 notification detailed information，才能
單獨表達 `NOT_FOUND`。完整 lookup procedure 與 HTTP examples 見
[Topology、policy 與 strategy 細節設計](./topology_policy_design.md)。

### 6.3 不新增或不放進 extension 的資訊

第一版不新增：

- `jobDescription`、Analytics ID、training filter、data／time requirement。
- model bytes、model URL、ADRF reference 或 model identifier。
- 另一組 global／local `roundInd`。
- 另一組 procedure ID、callback correlation ID 或 subscription resource ID。
- training delay、expected completion time 或 local termination cause。
- 未知來源的 generic resource criteria。
- Root 內部如何計算 candidate reputation／priority 的方法。

這些資訊不是已有標準欄位，就是屬於 implementation／operator policy，或尚未
具備完整 producer-to-consumer flow。

---

## 7. Release 18 至 Release 20 差異

| Release | `Nnwdaf_MLModelTraining` 相關變化 | 對本設計的影響 |
| --- | --- | --- |
| Release 18 | 既有 subscription、PATCH、Notify、data／time requirement、round、delay 與 termination request | 目前主要 service-side baseline |
| Release 19 | 新增 `/subscriptions/{subscriptionId}/unsubscribe-info`，`TrainingUnsubscribeInfo` 可由 FL Server NWDAF 向 FL Client NWDAF 攜帶 termination cause 與 optional model information | Server unselect、suspend 或 finish local Client subscription 不需自行建立新的 termination payload；仍沒有 hierarchical topology report |
| Release 20 | `StatusReportInfo` 新增 `mlModelLoss` 與 `modelMetric` | 擴充 training outcome report，不提供 recursive node lifecycle、policy 或 strategy |

Release 19／20 的 `NwdafMLModelTrainSubsc`、`NwdafMLModelTrainSubscPatch`
與 `NwdafMLModelTrainNotif` 仍沒有 recursive topology、node-scoped policy、
strategy 或 `reportAfter`。因此目前 candidate extension 的核心 gap 沒有被較新
release 取代。

若 implementation 必須嚴格維持 Release 18 contract，Server 只能使用該
release 已有的 unsubscribe／DELETE 行為，不能直接把 Release 19
`TrainingUnsubscribeInfo` 當成 Release 18 欄位。本文記錄 Release 19 能力是為了
界定版本差異，後續 schema 必須先決定 target release，不能把不同 release 的
欄位混成同一份 baseline。

---

## 8. 下一步

1. 依本文件邊界建立 `x-flTopology`、`x-flTopologyReport`、
   `x-retainedResultReq` 與 `x-retainedResultStatus` 的 candidate OpenAPI
   mapping。
2. 分別處理 POST／PUT、PATCH 與 Notify direction，確認每個 extension entry
   的 producer、consumer、required condition 與 validation。
3. 解決 topology-only Notify 與既有 detailed-information condition 的
   procedure mapping。
4. 以本文件作為 schema review boundary；完整 procedure 與 HTTP examples
   維護於 [Topology、policy 與 strategy 細節設計](./topology_policy_design.md)。

---

## 9. 證據來源

- [TS 23.288 Release 18 §6.2C Federated Learning among Multiple NWDAFs](../../../specs/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2C%20Federated%20Learning%20among%20Multiple%20NWDAFs.md)
- [TS 23.288 Release 18 §6.2F ML Model Training](../../../specs/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2F%20Procedure%20for%20ML%20Model%20Training.md)
- [TS 29.520 Release 18 §4.6 Nnwdaf_MLModelTraining Service](../../../specs/TS%2029.520/4%20Services%20offered%20by%20the%20NWDAF/4.6%20Nnwdaf_MLModelTraining%20Service.md)
- [TS 29.520 Release 18 §5.5.6 Data Model](../../../specs/TS%2029.520/5%20API%20Definitions/5.5%20Nnwdaf_MLModelTraining%20Service%20API/5.5.6%20Data%20Model.md)
- [TS 29.520 Release 18 Nnwdaf_MLModelTraining OpenAPI](../../../specs/openapi/TS29520_Nnwdaf_MLModelTraining.yaml)
- [TS 29.520 Release 18 Nnwdaf_MLModelProvision OpenAPI](../../../specs/openapi/TS29520_Nnwdaf_MLModelProvision.yaml)
- [3GPP official Release 18 `TS29520_Nnwdaf_MLModelTraining.yaml`](https://forge.3gpp.org/rep/all/5G_APIs/-/blob/REL-18/TS29520_Nnwdaf_MLModelTraining.yaml)
- [3GPP official Release 19 `TS29520_Nnwdaf_MLModelTraining.yaml`](https://forge.3gpp.org/rep/all/5G_APIs/-/blob/REL-19/TS29520_Nnwdaf_MLModelTraining.yaml)
- [3GPP official Release 20 `TS29520_Nnwdaf_MLModelTraining.yaml`](https://forge.3gpp.org/rep/all/5G_APIs/-/blob/REL-20/TS29520_Nnwdaf_MLModelTraining.yaml)

---

## 10. 變更紀錄

| 日期 | 內容 |
| --- | --- |
| 2026-09-02 | 建立標準欄位與 proposed extension 邊界；完成 Release 18 request／Notify mapping，並納入 Release 19 unsubscribe-info 與 Release 20 status report 差異。 |
| 2026-09-02 | 加入 retained-result extension boundary：上層以 topology child node 的 `retainedResultReq` 指示 Intermediate，downstream request 使用 `x-retainedResultReq`，report 使用 `x-retainedResultStatus`，並標示既有 `mlCorreId`、`roundInd`、`mLModelInfos`、`immReport` 與 Notify 的重用位置。 |
