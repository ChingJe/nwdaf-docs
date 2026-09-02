# Hierarchical NWDAF FL Candidate OpenAPI Schema

日期：2026-09-02

狀態：候選設計，待使用者審查

上層文件：

- [Hierarchical NWDAF Federated Learning 協定設計](./protocol_design.md)
- [Topology、policy 與 strategy 細節設計](./topology_policy_design.md)
- [標準欄位與 Extension 邊界](./standard_field_extension_boundary.md)

---

## 1. 文件目的

本文將已確認的 hierarchical NWDAF FL topology、participant policy、training
strategy、node-local execution setting、逐級 status report 與 retained-result
lookup 語意，映射成可供審查的 OpenAPI 3.0 candidate schema。

本文採用 Stage 3 形式：先說明 service operation、資料型別、條件與 procedure
語意，再提供 candidate OpenAPI YAML fragments 與實際 HTTP message examples。
它不是直接修改 3GPP Release 18 OpenAPI attachment，也不是已完成 code
generation 的獨立 OpenAPI file。設計確認後，才需要將 fragments 整合成可由
validator 與 generator 處理的完整 YAML。

主要 baseline 為 TS 29.520 Release 18 V18.14.0、
`Nnwdaf_MLModelTraining` API 1.0.5。Release 19／20 的差異已在標準邊界文件
查核；本 candidate 不混入較新 release 的 `/unsubscribe-info`
resource 或其他新增欄位。

---

## 2. 服務操作映射

本設計不增加新的 resource path 或 service operation，只擴充既有 message
types：

| 方向 | 既有 operation／representation | 候選 properties | Producer → Consumer |
| --- | --- | --- | --- |
| Create | `POST /subscriptions`，`NwdafMLModelTrainSubsc` | `x-flTopology`、`x-retainedResultReq` | Parent FL Server → direct FL Client |
| Full update | `PUT /subscriptions/{subscriptionId}`，`NwdafMLModelTrainSubsc` | 同 Create | Parent FL Server → direct FL Client |
| Partial update | `PATCH /subscriptions/{subscriptionId}`，`NwdafMLModelTrainSubscPatch` | `x-flTopology`、`x-retainedResultReq` | Parent FL Server → direct FL Client |
| Disabled-child removal | `DELETE /subscriptions/{subscriptionId}` | 無新增 request property；由 parent 收到 child `enabled: false` 後觸發 | Parent／Intermediate FL Server → disabled direct FL Client |
| Immediate report | Create／update response 的 `immReport` | 經由 `NwdafMLModelTrainNotif` 自動取得 report properties | Direct FL Client → parent FL Server |
| Notify | 對既有 `notifUri` 執行 POST，`NwdafMLModelTrainNotif` | `x-flTopologyReport`、`x-retainedResultStatus` | Direct FL Client → parent FL Server |

`x-flTopology` 與 `x-flTopologyReport` 是放在 message schema
`properties` 內的 JSON payload property。它們不是直接放在 OpenAPI Schema
Object 上、只供 tooling 解讀的 OpenAPI Specification Extension。

`mlCorreId`、`notifCorreId`、`roundInd`、`mLModelInfos`、`mLPreFlag`、
`mLModelTrainInfos`、`mLTrainRepInfo` 與 `eventReq` 繼續使用既有 3GPP
properties，不在 custom objects 中複製。

`suppFeats` 也直接重用既有 `SupportedFeatures` property。本文為整組
hierarchical orchestration semantics 定義一個候選 optional feature；不另增
capability property，也不為每個 extension property 個別配置 feature bit。

---

## 3. 候選資料模型

### 3.1 請求方向的 topology node

`FlTopologyNode` 是 role-neutral recursive node。它描述接收者的 subtree
instruction；`nfInstanceId` 是唯一無條件必填欄位，`priority` 另有跨欄位
條件：

| Property | P | Type | 語意 |
| --- | --- | --- | --- |
| `nfInstanceId` | M | `NfInstanceId` | 此 node 所代表的 NWDAF instance |
| `enabled` | O | boolean，default `true` | 此 node 作為 candidate 時是否允許 establishment 與 training selection |
| `priority` | C | integer, `>= 0` | 此 node 作為 parent candidate 時的建立與 selection 優先順位；`selectionMethod: priority` 時，所有仍啟用的 explicit children 必須提供 |
| `policy` | O | `FlPolicy` | 此 node 作為 local FL Server 時管理 direct children 的 policy |
| `strategy` | O | `FlStrategy` | 此 node 與 direct children 使用的 training／aggregation contract |
| `reportAfter` | O | `FlReportAfter` | 此 node 完成多少 local work 後向 direct parent 回報 |
| `retainedResultReq` | O | boolean | Parent 在當次對此 node 建立或更新 subscription 時，是否要求一次 retained-result lookup |
| `children` | O | array(`FlTopologyNode`) | 下一層 candidate／instruction nodes |

`enabled`、`priority` 與 `retainedResultReq` 是 parent-to-child edge
instruction。即使它們放在 child node object 中，也不描述 child 對自己的
children 所採用的值，且不自動向 descendants 繼承。`enabled` 只在 node
出現在 parent 的 `children` 時有意義；最外層 node 應省略。

### 3.2 參與者 policy

`FlPolicy` 的 individual properties 保持 optional，讓上層只指定自己要控制的
decision。具有明確 default 的 `allowAdditionalCandidates` 與
`additionalCandidatePriority` 依 protocol default 解讀；其餘未指定部分才由
接收 node 的 local configuration 決定。Request 未指定的實際值可在
`x-flTopologyReport` 中用同名 properties 回報。

| Property | Type／range | 語意 |
| --- | --- | --- |
| `allowAdditionalCandidates` | boolean，default `false` | 是否授權接收 node 加入上層 `children` 未列出的 candidates |
| `additionalCandidatePriority` | integer, `>= 0`，default `0` | 接收 node 自行加入之 candidates 的預設 priority |
| `selectionMethod` | `priority`／`random` | 從 eligible candidate pool 選擇的方式 |
| `minAvailableNodes` | integer, `>= 1` | local FL process 可開始 training 所需的最少 `ACTIVE` direct children |
| `fractionTrain` | number, `0 < value <= 1` | 每一輪預計從 active pool 選取的比例 |
| `minTrainNodes` | integer, `>= 1` | 每一輪至少選取的 Clients 數量 |
| `acceptFailures` | boolean | 被選 Clients 部分失敗時是否仍允許聚合 |
| `minCompletionRate` | number, `0 < value <= 1` | `acceptFailures: true` 時允許聚合的最低成功比例 |

第一版不另外增加 dropout 或 replacement mode。Training 中 participant 退出後
使用 `INACTIVE` 回報；是否補入下一位 candidate 由 active count、candidate
priority、`minAvailableNodes` 與 additional-candidate authority 共同決定。

### 3.3 訓練 strategy

`FlStrategy` 使用 discriminator 將 `method` 綁定到 typed parameters。第一版
只有 `fedProx`：

| Property | P | Type／value | 語意 |
| --- | --- | --- | --- |
| `method` | M | `fedProx` | 此 local FL process 的共同 FL method |
| `aggregation` | M | `sampleWeighted` | FL Server 使用 sample count 執行 weighted aggregation |
| `methodParameters` | M | `FedProxParameters` | `fedProx` 專用 parameters |
| `methodParameters.proximalMu` | M | number, `>= 0` | FedProx proximal term coefficient；沒有隱含 default |

未來增加其他 method 時，新增另一個 strategy subtype 及其 typed parameter
schema；不把 `methodParameters` 改回 arbitrary object。

### 3.4 Node-local 回報指令

`FlReportAfter.count` 是 positive integer；`unit` 為 `epoch` 或 `round`。
`epoch` 用於 Client local training，`round` 用於 Intermediate 完成 lower-tier
rounds 後再向 parent 回報。此 object 不自動向 descendants 繼承。

### 3.5 Topology 回報

`FlTopologyReport` 是通知者本身的 report wrapper，承載通知者的
`nfInstanceId`、實際採用的 `policy`／`strategy`／`reportAfter`，以及它對
direct children 的 recursive reports。Wrapper 不帶自己的 `status`：通知者
與 direct parent 之間的 relationship 由 parent 管理，不能要求通知者替自己
產生 parent-owned status timestamp。

Wrapper 下的 `FlTopologyReportNode` 只回報：

- 上層明確提供的 candidates；以及
- selection owner 已納入自己 candidate pool、準備嘗試或已嘗試的 additional
  candidates。

它不回報 NRF discovery 回應中的所有 NWDAFs。每個 child report node 必須帶有
`nfInstanceId`、`status` 與 `statusTimestamp`；可用同一組 `policy`、
`strategy` 與 `reportAfter` schemas 回報實際採用值。當 `status` 為
`FAILED` 或 `INACTIVE` 時，還必須帶有 `statusCause`，讓上層區分應重試、
更換 candidate 或接受 participant 離開。

| Status | 語意 |
| --- | --- |
| `UNCONFIRMED` | 已知或已列入 candidate pool，但尚未取得參與確認 |
| `DEPLOYING` | 已開始建立 subscription／執行 preparation，尚無最終結果 |
| `ACTIVE` | 已加入目前 realized topology |
| `FAILED` | 已嘗試，但被拒絕、timeout 或建立失敗 |
| `INACTIVE` | 原本已加入，後續退出、取消 subscription 或被移除 |

`statusTimestamp` 與 `statusCause` 由管理該 direct-child relationship 的
parent FL Server 產生；逐級包裝 report 時不得改寫 descendant values。
`statusCause` 只描述 topology relationship 狀態，不取代 individual event
failure、training delay、termination 或 HTTP rejection 使用的既有 3GPP
fields。若 direct child 以既有 `termTrainReq` 回報 `NWDAF_OVERLOAD` 或
`NOT_AVAILABLE_ML_TRAIN`，parent 在 topology report 中保留相同 cause value。
`termTrainReq: OTHERS` 映射為 `statusCause: OTHER`；成功建立 resource 但未
協商到必要 feature 3 時，使用 `FEATURE_NOT_SUPPORTED`。

Status、cause 與其他不決定 typed object shape 的 candidate enums 採用 3GPP
常見的 forward-compatible pattern。未知 status 不得被舊 consumer 推測為
`ACTIVE`；未知 cause 可以保留與轉送，但不能觸發已知 cause 的特定處理。
`FlStrategy.method` 例外維持 closed discriminator，因為它決定
`methodParameters` 的具體 schema。

### 3.6 Feature negotiation

本 candidate 在 TS 29.520 §5.5.8 的 Supported Features table 增加一個
optional feature：

| Feature number | Feature name | Description |
| --- | --- | --- |
| 3 | `HierarchicalFLOrch` | 支援本文定義的 recursive topology instruction／report、node-scoped policy／strategy／`reportAfter`，以及 retained-result lookup procedure |

選擇 feature number `3` 是為了避免與 Release 19 的 feature 1
`UnsubscribeWithInfo` 及 Release 20 的 feature 2 `EnAccuracyInformation`
衝突。依 `SupportedFeatures` bitmask 編碼，僅宣告 feature 3 時，`suppFeats`
值為 `"4"`。

每一個 parent-to-child subscription resource 都要分別協商此 feature。
Root 與 Intermediate 協商成功，不代表 Intermediate 與其 direct children
也已協商成功；Intermediate 建立下游 subscription 時同樣必須帶
`suppFeats: "4"`，並檢查成功 response 是否仍包含 feature 3。

TS 29.500 §6.6.2 允許 consumer 在 resource 的 supported features 尚未確定前
先於初始 POST 傳送 feature-specific information，因此 initial Create 可以同時
帶 `suppFeats` 與 `x-*` properties。若成功 response 的 `suppFeats` 未包含
feature 3，consumer 不得把該 resource 視為 hierarchical orchestration
resource，也不得在後續 PUT／PATCH 或 Notify 套用本文 procedure；若本次
subscription 的目的必須依賴 hierarchy，consumer 應刪除該 resource，並將
該 candidate 回報為 `FAILED`／`FEATURE_NOT_SUPPORTED`。

---

## 4. 候選 OpenAPI YAML

### 4.1 既有 message type 擴充

以下 fragments 應合併至 Release 18 相同名稱 schema 的 `properties`。Notify
另需加入 §4.3 的 detailed-information 與 retained-result conditions。

```yaml
components:
  schemas:
    NwdafMLModelTrainSubsc:
      properties:
        x-flTopology:
          $ref: '#/components/schemas/FlTopologyNode'
        x-retainedResultReq:
          type: boolean
          writeOnly: true
          description: >
            Indicates whether the recipient shall perform one retained-result lookup
            for this operation and report the latest completed result retained for
            the ML procedure identified by mlCorreId.

    NwdafMLModelTrainSubscPatch:
      properties:
        x-flTopology:
          $ref: '#/components/schemas/FlTopologyNode'
        x-retainedResultReq:
          type: boolean
          writeOnly: true
          description: >
            Indicates whether the recipient shall perform one retained-result lookup
            for this partial-update operation.

    NwdafMLModelTrainNotif:
      properties:
        x-flTopologyReport:
          $ref: '#/components/schemas/FlTopologyReport'
        x-retainedResultStatus:
          $ref: '#/components/schemas/RetainedResultStatus'
```

### 4.2 新增 component schemas

所有 candidate structured objects 使用 `additionalProperties: false`，讓接收者
只接受本版本明確定義的 semantics。新增 policy property、FL method 或
method-specific parameter 時，必須同步擴充對應 component schema，不能以
未定義的 arbitrary property 取代設計。

```yaml
components:
  schemas:
    FlTopologyNode:
      description: >
        Represents a role-neutral node in a recursive hierarchical FL topology
        instruction.
      type: object
      additionalProperties: false
      properties:
        nfInstanceId:
          $ref: 'TS29571_CommonData.yaml#/components/schemas/NfInstanceId'
        enabled:
          type: boolean
          default: true
          description: >
            Indicates whether this node is enabled for candidate establishment and
            training selection by its direct parent.
        priority:
          type: integer
          minimum: 0
          description: >
            Candidate priority assigned by the direct parent for candidate
            establishment and priority-based training selection. A greater value
            has higher priority.
        policy:
          $ref: '#/components/schemas/FlPolicy'
        strategy:
          $ref: '#/components/schemas/FlStrategy'
        reportAfter:
          $ref: '#/components/schemas/FlReportAfter'
        retainedResultReq:
          type: boolean
          writeOnly: true
          description: >
            Indicates whether the parent shall request one retained-result lookup in
            the operation that establishes or updates the subscription towards this
            node.
        children:
          type: array
          items:
            $ref: '#/components/schemas/FlTopologyNode'
          minItems: 1
      required:
        - nfInstanceId
      not:
        allOf:
          - properties:
              enabled:
                type: boolean
                enum: [false]
            required: [enabled]
          - properties:
              retainedResultReq:
                type: boolean
                enum: [true]
            required: [retainedResultReq]

    FlPolicy:
      description: >
        Represents participant selection and result acceptance decisions applied by
        a node acting as the FL Server for its direct children.
      type: object
      additionalProperties: false
      properties:
        allowAdditionalCandidates:
          type: boolean
          default: false
        additionalCandidatePriority:
          type: integer
          minimum: 0
          default: 0
        selectionMethod:
          $ref: '#/components/schemas/FlSelectionMethod'
        minAvailableNodes:
          type: integer
          minimum: 1
        fractionTrain:
          type: number
          format: float
          minimum: 0
          exclusiveMinimum: true
          maximum: 1
        minTrainNodes:
          type: integer
          minimum: 1
        acceptFailures:
          type: boolean
        minCompletionRate:
          type: number
          format: float
          minimum: 0
          exclusiveMinimum: true
          maximum: 1

    FlSelectionMethod:
      description: Represents the method used to select eligible FL Clients.
      anyOf:
        - type: string
          enum:
            - priority
            - random
        - type: string
          description: >
            This string provides forward-compatibility with future extensions to
            the enumeration but is not used to encode content defined in this
            version of the API.

    FlStrategy:
      description: Represents the training and aggregation contract of a local FL process.
      oneOf:
        - $ref: '#/components/schemas/FedProxStrategy'
      discriminator:
        propertyName: method
        mapping:
          fedProx: '#/components/schemas/FedProxStrategy'

    FedProxStrategy:
      type: object
      additionalProperties: false
      properties:
        method:
          type: string
          enum:
            - fedProx
        aggregation:
          $ref: '#/components/schemas/FlAggregation'
        methodParameters:
          $ref: '#/components/schemas/FedProxParameters'
      required:
        - method
        - aggregation
        - methodParameters

    FlAggregation:
      description: Represents the aggregation rule used by the FL Server.
      anyOf:
        - type: string
          enum:
            - sampleWeighted
        - type: string
          description: >
            This string provides forward-compatibility with future extensions to
            the enumeration but is not used to encode content defined in this
            version of the API.

    FedProxParameters:
      description: Represents parameters required by the FedProx method.
      type: object
      additionalProperties: false
      properties:
        proximalMu:
          type: number
          format: float
          minimum: 0
      required:
        - proximalMu

    FlReportAfter:
      description: >
        Represents the amount of node-local work to complete before reporting to
        the direct parent.
      type: object
      additionalProperties: false
      properties:
        count:
          type: integer
          minimum: 1
        unit:
          $ref: '#/components/schemas/FlReportUnit'
      required:
        - count
        - unit

    FlReportUnit:
      description: Represents the unit used by a report-after instruction.
      anyOf:
        - type: string
          enum:
            - epoch
            - round
        - type: string
          description: >
            This string provides forward-compatibility with future extensions to
            the enumeration but is not used to encode content defined in this
            version of the API.

    FlTopologyReport:
      description: >
        Represents the reporting node and the recursive states of the candidates
        managed by that node.
      type: object
      additionalProperties: false
      properties:
        nfInstanceId:
          $ref: 'TS29571_CommonData.yaml#/components/schemas/NfInstanceId'
        policy:
          $ref: '#/components/schemas/FlPolicy'
        strategy:
          $ref: '#/components/schemas/FlStrategy'
        reportAfter:
          $ref: '#/components/schemas/FlReportAfter'
        children:
          type: array
          items:
            $ref: '#/components/schemas/FlTopologyReportNode'
          minItems: 1
      required:
        - nfInstanceId

    FlTopologyReportNode:
      description: >
        Represents a node in a recursive report of a realized hierarchical FL
        topology and its candidate states.
      type: object
      additionalProperties: false
      properties:
        nfInstanceId:
          $ref: 'TS29571_CommonData.yaml#/components/schemas/NfInstanceId'
        status:
          $ref: '#/components/schemas/FlTopologyStatus'
        statusTimestamp:
          $ref: 'TS29571_CommonData.yaml#/components/schemas/DateTime'
        statusCause:
          $ref: '#/components/schemas/FlTopologyStatusCause'
        policy:
          $ref: '#/components/schemas/FlPolicy'
        strategy:
          $ref: '#/components/schemas/FlStrategy'
        reportAfter:
          $ref: '#/components/schemas/FlReportAfter'
        children:
          type: array
          items:
            $ref: '#/components/schemas/FlTopologyReportNode'
          minItems: 1
      required:
        - nfInstanceId
        - status
        - statusTimestamp
      oneOf:
        - properties:
            status:
              type: string
              enum:
                - FAILED
                - INACTIVE
          required:
            - statusCause
        - properties:
            status:
              type: string
              enum:
                - UNCONFIRMED
                - DEPLOYING
                - ACTIVE
          not:
            required:
              - statusCause
        - properties:
            status:
              type: string
              not:
                enum:
                  - UNCONFIRMED
                  - DEPLOYING
                  - ACTIVE
                  - FAILED
                  - INACTIVE

    FlTopologyStatus:
      description: Represents the lifecycle state of a hierarchical FL candidate.
      anyOf:
        - type: string
          enum:
            - UNCONFIRMED
            - DEPLOYING
            - ACTIVE
            - FAILED
            - INACTIVE
        - type: string
          description: >
            This string provides forward-compatibility with future extensions to
            the enumeration but is not used to encode content defined in this
            version of the API.

    FlTopologyStatusCause:
      description: |
        Represents the reason why a hierarchical FL candidate relationship
        entered the FAILED or INACTIVE state.
        Possible values are:
        - SUBSCRIPTION_REJECTED: Establishment of the downstream subscription was rejected.
        - RESPONSE_TIMEOUT: The required response was not received before the applicable deadline.
        - COMMUNICATION_FAILURE: An SBI transport or connection failure prevented the exchange.
        - NWDAF_OVERLOAD: The Client reported the existing NWDAF_OVERLOAD termination cause.
        - NOT_AVAILABLE_ML_TRAIN: The Client reported that ML model training was not available.
        - PARTICIPANT_WITHDRAWN: The participant withdrew from the local FL process.
        - REMOVED_BY_POLICY: The parent removed the participant by instruction or local policy.
        - FEATURE_NOT_SUPPORTED: The required hierarchical orchestration feature was not negotiated.
        - OTHER: The reason cannot be classified by another value.
      anyOf:
        - type: string
          enum:
            - SUBSCRIPTION_REJECTED
            - RESPONSE_TIMEOUT
            - COMMUNICATION_FAILURE
            - NWDAF_OVERLOAD
            - NOT_AVAILABLE_ML_TRAIN
            - PARTICIPANT_WITHDRAWN
            - REMOVED_BY_POLICY
            - FEATURE_NOT_SUPPORTED
            - OTHER
        - type: string
          description: >
            This string provides forward-compatibility with future extensions to
            the enumeration but is not used to encode content defined in this
            version of the API.

    RetainedResultStatus:
      description: |
        Represents the outcome of a retained-result lookup.
        Possible values are:
        - FOUND: A latest completed result was found.
        - NOT_FOUND: The lookup completed and no retained result was found.
        - FAILED: The accepted lookup could not be completed.
      anyOf:
        - type: string
          enum:
            - FOUND
            - NOT_FOUND
            - FAILED
        - type: string
          description: >
            This string provides forward-compatibility with future extensions to
            the enumeration but is not used to encode content defined in this
            version of the API.
```

### 4.3 Notify 條件

TS 29.520 Release 18 §4.6.2.4.2 與 §5.5.6.2.8 目前要求 Notify 至少包含
`delayEventNotif`、`mLModelInfos` 或 `termTrainReq` 之一。只把 custom property
加入 OpenAPI `properties`，不能讓 topology-only report、`NOT_FOUND` 或
`FAILED` report 自動符合這項 procedure 條件。

Candidate procedure 必須把 `x-flTopologyReport` 與
`x-retainedResultStatus` 加入 notification detailed information 的合法集合。
對應的 OpenAPI condition 可表示為：

```yaml
NwdafMLModelTrainNotif:
  allOf:
    - anyOf:
        - required: [delayEventNotif]
        - required: [mLModelInfos]
        - required: [termTrainReq]
        - required: [x-flTopologyReport]
        - required: [x-retainedResultStatus]
    - oneOf:
        - not:
            required: [x-retainedResultStatus]
        - allOf:
            - properties:
                x-retainedResultStatus:
                  type: string
                  enum: [FOUND]
              required:
                - x-retainedResultStatus
                - roundInd
                - mLModelInfos
        - allOf:
            - properties:
                x-retainedResultStatus:
                  type: string
                  enum: [NOT_FOUND, FAILED]
              required:
                - x-retainedResultStatus
            - not:
                anyOf:
                  - required: [roundInd]
                  - required: [mLModelInfos]
        - properties:
            x-retainedResultStatus:
              type: string
              not:
                enum: [FOUND, NOT_FOUND, FAILED]
          required:
            - x-retainedResultStatus
    - oneOf:
        - not:
            anyOf:
              - required: [x-flTopologyReport]
              - required: [x-retainedResultStatus]
        - allOf:
            - anyOf:
                - required: [x-flTopologyReport]
                - required: [x-retainedResultStatus]
            - required: [mlCorreId]
```

這個 condition 保留既有 detailed information，並要求任何 extension report
同時帶有 `mlCorreId`。本版本定義三種 retained-result report：

- `FOUND` 必須同時提供既有 `roundInd` 與 `mLModelInfos`。
- `NOT_FOUND` 不攜帶 `roundInd` 或空／不存在的 model result。
- `FAILED` 表示已接受的 lookup 後續執行失敗，同樣不攜帶 `roundInd` 或
  model result。

Forward-compatible 的未知 outcome 可以通過資料型別驗證，但舊版 consumer
不得將它解讀成 `FOUND`、`NOT_FOUND` 或 `FAILED`；它應結束目前 outstanding
lookup、不使用其中的 model result，並依 local unsupported-outcome handling
決定是否重試或停止 recovery。

既有 `delayEventNotif` 與 `mLModelInfos`、`delayEventNotif` 與
`termTrainReq` 的 mutual-exclusion procedure rules 不變。

---

## 5. Schema 以外的驗證規則

OpenAPI 3.0 可以表示 property type、static range、required fields 與部分
conditional composition，但不能完整表達跨遞迴 node、既有 resource state 或
兩個 numeric properties 間的比較。實作與 Stage 3 procedure text 仍需檢查：

1. Create／PUT 中若出現 `x-flTopology` 或
   `x-retainedResultReq: true`，同一 `NwdafMLModelTrainSubsc` 必須提供
   `mlCorreId`。
2. PATCH 不提供新的 `mlCorreId`；只有既有 resource 已屬於 FL procedure 時，
   才能接受 `x-flTopology` 或 `x-retainedResultReq: true`。
3. Notify 若帶有 `x-flTopologyReport` 或 `x-retainedResultStatus`，同一
   `NwdafMLModelTrainNotif` 必須提供 `mlCorreId`；`notifCorreId` 仍負責 local
   subscription callback correlation。
4. `x-flTopology` 最外層 `nfInstanceId` 必須等於接收 subscription 的 NWDAF
   instance；`x-flTopologyReport` 最外層 identity 必須等於該
   subscription／`notifCorreId` 所綁定的 direct Client NWDAF。Identity
   mismatch 以 `400 Bad Request` 拒絕。
5. 單一 instruction 或 report subtree 中，同一 `nfInstanceId` 不得重複出現，
   也不得在自己的 descendants 再次出現；相同 parent 的 `children` 不得包含
   重複 identities。這項 local validation 不解決 delegated selection 在不同
   Branches 間產生的 ownership conflict。
6. `allowAdditionalCandidates` 省略時為 `false`，
   `additionalCandidatePriority` 省略時為 `0`。當上層指定
   `selectionMethod: priority` 時，每個 `enabled` 未設為 `false` 的 explicit
   child 都必須提供 `priority`；locally discovered candidates 使用
   `additionalCandidatePriority`。
7. `minAvailableNodes >= minTrainNodes`。只有 `ACTIVE` direct children 數量
   達到 `minAvailableNodes`，local FL process 才達到 topology-ready 條件；
   active count 低於 `minTrainNodes` 時不得開始新一輪。
8. 每輪選取數量為
   `min(active count, max(floor(active count * fractionTrain), minTrainNodes))`。
9. `additionalCandidatePriority` 只用於
   `allowAdditionalCandidates: true` 時由 node 自行加入的 candidates。
10. Candidate establishment 與 training selection 必須分開：
    `selectionMethod: priority` 先嘗試較高 priority 的 eligible
    `UNCONFIRMED` candidates，使其進入 `DEPLOYING`；每輪 training 則只從
    `ACTIVE` pool 依較大 priority 優先選取，同順位使用 random。
    `selectionMethod: random` 從各階段的 eligible candidate pool 均勻隨機
    選取。
11. `enabled: false` 優先於 priority 與 candidate source，且不得與同一 node
    的 `retainedResultReq: true` 同時出現。接收者立即停止嘗試或選取該 node；
    若 direct-child subscription 已存在，使用既有 DELETE 移除它。DELETE
    回覆 `204`／`404` 時回報 `INACTIVE`／`REMOVED_BY_POLICY`；timeout 或
    communication failure 時仍維持 local `INACTIVE`，分別使用
    `RESPONSE_TIMEOUT` 或 `COMMUNICATION_FAILURE`，並由 implementation
    cleanup retry。同一 node 不得因 local discovery 被重新加入，且其
    descendants 不再繼續建立。
12. `minCompletionRate` 只在 `acceptFailures: true` 時參與 aggregation
    decision，完成比例為 successful replies 除以 selected participants；
    `acceptFailures: false` 要求所有 selected participants 成功。
13. `enabled`、`priority` 與 `retainedResultReq` 只由 direct parent 消費，不向
    subtree 自動繼承。`retainedResultReq: true` 只映射到當次對該
    direct child 發出的 Create、PUT 或 PATCH operation。
14. `policy` 與 `strategy` 作用於 node 作為 FL Server 時建立的 direct-child
    local FL process。Intermediate 建立下游 subscriptions 時必須維持上層明確
    指定的 `method`、`aggregation` 與 `methodParameters` contract。
    `reportAfter` 只作用於該 node 對 direct parent 的回報。
15. `x-flTopologyReport` wrapper 不帶通知者自己的 relationship status。
    `children[].statusTimestamp` 保留管理該 relationship 的 parent 所產生之
    時間；轉送 Intermediate 不得重寫 descendant timestamps。
16. `children[].statusCause` 在 `FAILED` 與 `INACTIVE` 時必填，在
    `UNCONFIRMED`、`DEPLOYING` 與 `ACTIVE` 時不得提供。既有
    `termTrainReq: OTHERS` 映射為 `statusCause: OTHER`。未知 status 不得被
    推測為 `ACTIVE`；未知 cause 可保留與轉送，但不得觸發已知 cause 的特定
    recovery action。
17. 每個被接受的 Create、PUT 或 PATCH 只有在當次攜帶
    `x-retainedResultReq: true` 時觸發一次 lookup。接收者必須回報
    `FOUND`、`NOT_FOUND` 或 `FAILED`；回報後動作完成，不保留為 subscription
    state。若 request 在被接受前即失敗，使用該 Create／PUT／PATCH operation
    的既有 HTTP error response，不另外產生 `FAILED` outcome。
    未知 forward-compatible outcome 會結束 outstanding lookup，但不得被當成
    `FOUND`、`NOT_FOUND` 或 `FAILED`，也不得使用其中的 model result。每個
    local subscription 同時最多一個 outstanding lookup；只有收到前一次
    outcome 後才能開始下一次，因此不增加 request ID。等待 outcome timeout
    時，該 lookup 仍視為 outstanding；Server 可繼續等待或終止目前
    subscription，但不得另起新的 outstanding lookup。
18. `FlSelectionMethod`、`FlAggregation` 與 `FlReportUnit` 的未知值可以被
    schema 解析，但接收者若無法履行該 instruction，必須依 §5.1 拒絕整個
    operation。`FlStrategy.method` 仍維持 closed discriminator；增加新的 FL
    method 時必須新增對應 subtype 與 typed parameter schema。
19. `HierarchicalFLOrch` 必須在每個 local subscription resource 分別協商。
    Initial POST 可同時攜帶 `suppFeats: "4"` 與本文 extension properties；
    後續 PUT／PATCH 與 Notify 只有在該 resource 的 negotiated
    `suppFeats` 包含 feature 3 時才能使用這些 properties 與 procedure。

### 5.1 Request rejection 與既有 subscription state

接收者已協商 `HierarchicalFLOrch` 後，若 request 內的 extension field 不符合
candidate schema，例如 closed discriminator、range、required／conditional
field 或跨欄位 validation 錯誤，應依 TS 29.500 §5.2.7.2 以
`400 Bad Request` 拒絕，使用最
適合的既有 cause（例如 `INVALID_MSG_FORMAT`、`MANDATORY_IE_INCORRECT` 或
`OPTIONAL_IE_INCORRECT`），並以 `ProblemDetails.invalidParams` 指出錯誤的
extension path。

若 message 在 schema 上合法，但接收者不能履行上層明確指定的 node-wide
`policy`、`strategy` 或 `reportAfter` contract，則重用 TS 29.520 §5.5.7.3
的 `403 Forbidden`／`ML_MODEL_TRAINING_REQS_NOT_MET`，並以
`invalidParams` 指出未滿足的 extension property。這是 candidate Stage 3
procedure 對既有 application error 適用範圍的擴充，不主張 Release 18 已把
custom fields 納入 `mLModelTrainInfos`。`failEventReports` 只描述部分
`mLEventSubscs` 未成功，不用來表示整個 node 無法接受 hierarchical contract。

PUT／PATCH 只有在成功處理及接受後才修改並保存 subscription。因此上述錯誤
response 不得留下部分更新；既有 subscription representation 與 negotiated
features 維持原狀。

### 5.2 PATCH 合併語意

Release 18 PATCH 使用 `application/merge-patch+json`。因此：

- 未出現在 PATCH body 的持續 extension property 維持原值。
- `x-retainedResultReq` 是一次性 operation instruction，不參與 resource state
  merge；當次為 `true` 時查詢一次，省略或為 `false` 時不查詢。
- `x-flTopology.children[].retainedResultReq` 同樣是一次性 edge
  instruction。Receiver 執行當次對應的 downstream operation 後，不將它
  保存為 upstream-assigned topology 的持續狀態。
- Object members 依 JSON Merge Patch 合併。
- `children` 是 array；PATCH 一旦提供某個 node 的 `children`，該 array 取代
  resource 中同位置的整個 upstream-assigned candidate array，不做以
  `nfInstanceId` 為 key 的逐項 merge。

`children` replacement 只更新上層在 subscription resource 中明確指定的
candidate set。接收者依 `allowAdditionalCandidates: true` 自行 discovery 並
保存的 local candidates 不在 request array 中，因此不能因這次 PATCH 被清除。
接收者必須在 local state 中區分 candidate source；同一 `nfInstanceId` 同時由
兩種來源取得時，以上層明確指定的 edge instructions 為準。

`priority: 0` 只表示最低順位，不表示刪除或停用。若上層需要明確禁止一個
candidate 繼續留在 topology 或被 local discovery 重新加入，必須在完整
replacement `children` array 保留該 node 並指定 `enabled: false`。省略該 node
會同時移除 upstream assignment 與既有 explicit prohibition，但不形成新的
上層指派；只有 `allowAdditionalCandidates: true` 時，Branch 才可透過 local
selection 再次加入它。若 array 保留該 node 並設為 `enabled: true` 或省略
`enabled`，則表示上層明確重新啟用及指派它。

一般 training round 只更新既有 `roundInd`、`mLModelInfos`、deadline 等標準
properties，不需要重送 `x-flTopology`。只有 membership、parent／child
relationship、policy、strategy 或 node-local instruction 改變時才更新
topology extension。

---

## 6. HTTP 訊息範例

以下 examples 使用同一組 Branch／Leaf identities，並只放入說明本設計所需的
既有與 extension properties。

### 6.1 Root 建立 Branch subscription

Root 指定五個 candidates，同時授權 Branch 補充其他 candidates。這是 explicit
children 與 delegated selection 自然共存的例子：

```http
POST /nnwdaf-mlmodeltraining/v1/subscriptions HTTP/1.1
Host: branch-a.example.org
Content-Type: application/json
Accept: application/json

{
  "mLEventSubscs": [
    {
      "mLEvent": "UE_COMMUNICATION",
      "mLEventFilter": {},
      "modelInterInfo": "pymtlf-model-bundle-v1"
    }
  ],
  "notifUri": "https://root.example.org/callbacks/ml-model-training",
  "notifCorreId": "root-branch-a-subscription",
  "suppFeats": "4",
  "mlCorreId": "hierarchical-fl-001",
  "mLPreFlag": true,
  "mLModelTrainInfos": [
    {
      "dataAvReq": {
        "inpEvents": [
          { "nwdafEvent": "UE_COMMUNICATION" }
        ]
      },
      "timeAvReq": "2026-09-02T14:00:00+08:00/2026-09-02T18:00:00+08:00"
    }
  ],
  "eventReq": {
    "notifMethod": "ON_EVENT_DETECTION",
    "immRep": false
  },
  "x-flTopology": {
    "nfInstanceId": "10000000-0000-4000-8000-000000000001",
    "children": [
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000000101",
        "priority": 100,
        "reportAfter": { "count": 5, "unit": "epoch" }
      },
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000000102",
        "priority": 80,
        "reportAfter": { "count": 5, "unit": "epoch" }
      },
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000000103",
        "priority": 60,
        "reportAfter": { "count": 5, "unit": "epoch" }
      },
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000000104",
        "priority": 40,
        "reportAfter": { "count": 5, "unit": "epoch" }
      },
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000000105",
        "priority": 20,
        "reportAfter": { "count": 5, "unit": "epoch" }
      }
    ],
    "policy": {
      "allowAdditionalCandidates": true,
      "additionalCandidatePriority": 0,
      "selectionMethod": "priority",
      "minAvailableNodes": 3,
      "fractionTrain": 0.6,
      "minTrainNodes": 3,
      "acceptFailures": true,
      "minCompletionRate": 0.6
    },
    "strategy": {
      "method": "fedProx",
      "aggregation": "sampleWeighted",
      "methodParameters": {
        "proximalMu": 0.01
      }
    },
    "reportAfter": {
      "count": 3,
      "unit": "round"
    }
  }
}
```

### 6.2 Branch 回報 preparation 結果

Branch 已建立三個 active direct children；一個失敗，最後一個尚未確認。這份
Notify 可單獨以 `x-flTopologyReport` 作為 detailed information：

```http
POST /callbacks/ml-model-training HTTP/1.1
Host: root.example.org
Content-Type: application/json

{
  "notifCorreId": "root-branch-a-subscription",
  "mlCorreId": "hierarchical-fl-001",
  "x-flTopologyReport": {
    "nfInstanceId": "10000000-0000-4000-8000-000000000001",
    "policy": {
      "allowAdditionalCandidates": true,
      "additionalCandidatePriority": 0,
      "selectionMethod": "priority",
      "minAvailableNodes": 3,
      "fractionTrain": 0.6,
      "minTrainNodes": 3,
      "acceptFailures": true,
      "minCompletionRate": 0.6
    },
    "strategy": {
      "method": "fedProx",
      "aggregation": "sampleWeighted",
      "methodParameters": {
        "proximalMu": 0.01
      }
    },
    "reportAfter": {
      "count": 3,
      "unit": "round"
    },
    "children": [
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000000101",
        "status": "ACTIVE",
        "statusTimestamp": "2026-09-02T14:29:10+08:00",
        "reportAfter": { "count": 5, "unit": "epoch" }
      },
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000000102",
        "status": "FAILED",
        "statusTimestamp": "2026-09-02T14:29:40+08:00",
        "statusCause": "RESPONSE_TIMEOUT",
        "reportAfter": { "count": 5, "unit": "epoch" }
      },
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000000103",
        "status": "ACTIVE",
        "statusTimestamp": "2026-09-02T14:29:32+08:00",
        "reportAfter": { "count": 5, "unit": "epoch" }
      },
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000000104",
        "status": "ACTIVE",
        "statusTimestamp": "2026-09-02T14:30:05+08:00",
        "reportAfter": { "count": 5, "unit": "epoch" }
      },
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000000105",
        "status": "UNCONFIRMED",
        "statusTimestamp": "2026-09-02T14:30:05+08:00",
        "reportAfter": { "count": 5, "unit": "epoch" }
      }
    ]
  }
}
```

### 6.3 Training 中明確剔除 topology candidate

若 Root 要求 Branch 以新的 `...106` 取代目前 `ACTIVE` 的 `...101`，且不允許
Branch 再透過 local discovery 選回 `...101`，PATCH 必須提供完整的 replacement
upstream-assigned `children` array，並將舊 node 設為 `enabled: false`。其他
subscription properties 保持不變：

```http
PATCH /nnwdaf-mlmodeltraining/v1/subscriptions/sub-branch-a HTTP/1.1
Host: branch-a.example.org
Content-Type: application/merge-patch+json

{
  "x-flTopology": {
    "nfInstanceId": "10000000-0000-4000-8000-000000000001",
    "children": [
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000000106",
        "priority": 120,
        "reportAfter": { "count": 5, "unit": "epoch" }
      },
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000000101",
        "enabled": false
      },
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000000102",
        "priority": 80,
        "reportAfter": { "count": 5, "unit": "epoch" }
      },
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000000103",
        "priority": 60,
        "reportAfter": { "count": 5, "unit": "epoch" }
      },
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000000104",
        "priority": 40,
        "reportAfter": { "count": 5, "unit": "epoch" }
      },
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000000105",
        "priority": 20,
        "reportAfter": { "count": 5, "unit": "epoch" }
      }
    ]
  }
}
```

Branch 收到這份 PATCH 後，不再把 `...101` 納入 eligible pool。由於它已有
active downstream subscription，Branch 使用既有 Unsubscribe operation：

```http
DELETE /nnwdaf-mlmodeltraining/v1/subscriptions/sub-leaf-101 HTTP/1.1
Host: leaf-101.example.org
```

Leaf 成功刪除 subscription 後回覆：

```http
HTTP/1.1 204 No Content
```

Branch 將 `...101` 更新為 `INACTIVE`，以
`statusCause: REMOVED_BY_POLICY` 說明是上層明確排除，並透過後續
`x-flTopologyReport` 回報。
若 Leaf 回覆 `404`，Branch 同樣視為 subscription 已不存在；若 DELETE
timeout 或發生 communication failure，Branch 仍立即將 `...101` 排除於
eligible pool，但分別以 `RESPONSE_TIMEOUT` 或 `COMMUNICATION_FAILURE`
回報 `INACTIVE`，並在 implementation 內繼續 cleanup retry、忽略該舊
subscription 之後抵達的 result。
新的 `...106` 先由 `UNCONFIRMED` 進入 `DEPLOYING`；只有成功成為 `ACTIVE`
後才能參與 training。只要 `enabled: false` instruction 仍存在，Branch 不得因
NRF 再次找到 `...101` 而把它加入 locally discovered set。

### 6.4 替代 Branch 要求 Leaf 回傳 retained result

替代 Branch 對 Leaf 建立新 subscription，使用同一 `mlCorreId` 並明確要求
lookup。這項 request 不要求 `eventReq.immRep` 必須為 `true`：

```http
POST /nnwdaf-mlmodeltraining/v1/subscriptions HTTP/1.1
Host: leaf-a1.example.org
Content-Type: application/json
Accept: application/json

{
  "mLEventSubscs": [
    {
      "mLEvent": "UE_COMMUNICATION",
      "mLEventFilter": {},
      "modelInterInfo": "pymtlf-model-bundle-v1"
    }
  ],
  "notifUri": "https://branch-a2.example.org/callbacks/ml-model-training",
  "notifCorreId": "branch-a2-leaf-a1-subscription",
  "suppFeats": "4",
  "mlCorreId": "hierarchical-fl-001",
  "eventReq": {
    "notifMethod": "ON_EVENT_DETECTION",
    "immRep": false
  },
  "x-retainedResultReq": true
}
```

當次 Create 只觸發一次 lookup。後續其他 subscription update 不會因
此自動再查；若 Server 需要再次查詢，必須在新的 operation 再次
攜帶 `x-retainedResultReq: true`。在當次 outcome 尚未回報前，不得對這個
subscription 發出另一個 lookup request；若收到未知的
forward-compatible outcome，該次 lookup 同樣結束，但本版本不得使用其中的
model result。只有收到前一次 `FOUND`、`NOT_FOUND`、`FAILED` 或未知 outcome
後，Server 才能在同一 subscription 發出下一次 lookup；若等待 timeout，只能
繼續等待或終止目前 subscription，不能另起新的 outstanding lookup。

若 Leaf 沒有同一 procedure 的已完成結果，subscription 仍可建立，之後透過
Notify 明確回報：

```http
POST /callbacks/ml-model-training HTTP/1.1
Host: branch-a2.example.org
Content-Type: application/json

{
  "notifCorreId": "branch-a2-leaf-a1-subscription",
  "mlCorreId": "hierarchical-fl-001",
  "x-retainedResultStatus": "NOT_FOUND"
}
```

若 lookup request 已被接受，但 Leaf 後續無法完成本地查詢，則回報：

```http
POST /callbacks/ml-model-training HTTP/1.1
Host: branch-a2.example.org
Content-Type: application/json

{
  "notifCorreId": "branch-a2-leaf-a1-subscription",
  "mlCorreId": "hierarchical-fl-001",
  "x-retainedResultStatus": "FAILED"
}
```

若找到結果，則回報該 Leaf local process 的 round 與既有 model reference：

```http
POST /callbacks/ml-model-training HTTP/1.1
Host: branch-a2.example.org
Content-Type: application/json

{
  "notifCorreId": "branch-a2-leaf-a1-subscription",
  "mlCorreId": "hierarchical-fl-001",
  "x-retainedResultStatus": "FOUND",
  "roundInd": 5,
  "mLModelInfos": [
    {
      "event": "UE_COMMUNICATION",
      "mLFileAddr": {
        "mLModelUrl": "https://leaf-a1.example.org/results/hierarchical-fl-001/round-5"
      }
    }
  ]
}
```

### 6.5 無法履行 strategy contract

若接收者已支援 `HierarchicalFLOrch`，request 也符合 schema，但接收者無法
執行上層指定的 `proximalMu`，則拒絕整個 Create／PUT／PATCH，不以
`failEventReports` 表示：

```http
HTTP/1.1 403 Forbidden
Content-Type: application/problem+json

{
  "status": 403,
  "cause": "ML_MODEL_TRAINING_REQS_NOT_MET",
  "invalidParams": [
    {
      "param": "x-flTopology.strategy.methodParameters.proximalMu",
      "reason": "The requested FedProx parameter cannot be supported."
    }
  ]
}
```

若問題是 schema-invalid value，例如 `proximalMu` 低於零，則改回覆
`400 Bad Request` 與適合的既有 400 cause，並在 `invalidParams` 指向相同
property path。

---

## 7. Schema 必須搭配的 procedure 語意

獨立 OpenAPI properties 不能完整定義 hierarchical behavior。若後續整理成正式
Stage 3 proposal，至少需要同步補充下列 procedure rules：

1. `x-flTopology` 的 node policy 只管理 direct children，沒有 subtree
   inheritance。
2. Forward `children` 是 candidate／instruction view，不等於全部已加入的
   realized topology。
3. PATCH `children` replacement 只更新 upstream-assigned candidate set，不能
   清除 locally discovered set；candidate-set ownership 與 replacement
   語意依 §5.2。
4. Child `enabled: false` 是 idempotent exclusion state；接收者須依 §5 與
   §6.3 停止 establishment／selection，必要時使用既有 Unsubscribe operation
   移除下游 subscription，並依 DELETE 結果回報 `INACTIVE`；不得同時要求
   該 child 執行 retained-result lookup。
5. Request／report wrapper identity 必須綁定 direct subscription peer，單一
   subtree 內的 identities 必須符合 tree uniqueness 規則。
6. `x-flTopologyReport` 可在 preparation 或 training lifecycle 回報，且成為
   Notify 合法的 detailed information。
7. `x-flTopologyReport` 的 candidate scope 與 timestamp ownership 依 §3.5。
8. `FAILED`／`INACTIVE` child report 必須提供 `statusCause`，其他已知 status
   不得提供；cause vocabulary 與既有 error／termination fields 的邊界依
   §3.5 與 §5。
9. Create／PUT、PATCH 與 Notify 的 `mlCorreId` 條件依 §4.3 與 §5；每個
   local subscription 仍使用自己的 resource ID 與 `notifCorreId`。
10. `x-retainedResultReq` 是當次 operation 的一次性 instruction，查詢同一
   `mlCorreId` 的最新已完成 local result；`FOUND`／`NOT_FOUND`／`FAILED` 條件依
   §4.3，lookup 不自行開始新一輪 training，也不保留為 subscription
   state。每個 local subscription 同時最多一個 outstanding lookup，只有收到
   前一次 outcome 後才能開始下一次，因此不增加 request ID。
11. 本版本已知 enums 以 3GPP forward-compatible pattern 表達；未知的
    report-side value 不得被誤判為已知狀態，未知的 instruction-side value 若
    無法執行則拒絕整個 operation。`FlStrategy.method` 維持 typed、closed
    discriminator。
12. `HierarchicalFLOrch` 使用既有 `suppFeats` 在每個 local subscription
   resource 分別協商；feature 3 未協商成功時，不得在該 resource 後續使用
   本文 extension procedure，並以 `FEATURE_NOT_SUPPORTED` 回報 candidate
   establishment failure。
13. Receiver 不得靜默改寫上層明確指定的 `policy`、`strategy` 或
    `reportAfter`。Schema／validation 錯誤依 §5.1 回覆 `400`；合法但無法
    履行的 contract 依 §5.1 回覆 `403 ML_MODEL_TRAINING_REQS_NOT_MET`。

---

## 8. 範圍邊界

本 candidate schema 已涵蓋目前已確認的 protocol information flow，但不定義：

- Root 如何決定 topology、priority 或 Branch replacement。
- NRF 如何 ranking candidates；NRF schema 不在本次範圍。
- Retained result 的保存期限、storage、freshness、deduplication、aggregation
  acceptance、ownership 或 fencing。
- Dynamic recovery state machine 或 seamless resume guarantee。
- 另一組 global round、hierarchical process ID 或 topology version。
- 未確認 authoritative source 的 generic resource criteria。

---

## 9. 規格依據

- TS 29.520 Release 18 V18.14.0 §4.6.2.2.2：Subscription creation。
- TS 29.520 Release 18 V18.14.0 §4.6.2.2.3：Full update with HTTP PUT。
- TS 29.520 Release 18 V18.14.0 §4.6.2.2.4：Partial update with HTTP PATCH。
- TS 29.520 Release 18 V18.14.0 §4.6.2.3：Unsubscribe with HTTP DELETE。
- TS 29.520 Release 18 V18.14.0 §4.6.2.4.2：Notify procedure 與 detailed
  information condition。
- TS 29.520 Release 18 V18.14.0 §5.5.3.3.3.3：Individual subscription 的
  DELETE representation 與 `204 No Content` response。
- TS 29.520 Release 18 V18.14.0 §5.5.6.2.2、§5.5.6.2.3、§5.5.6.2.8：
  `NwdafMLModelTrainSubsc`、`NwdafMLModelTrainSubscPatch` 與
  `NwdafMLModelTrainNotif` data types。
- TS 29.520 Release 18 V18.14.0 §5.5.7.3：
  `ML_MODEL_TRAINING_REQS_NOT_MET` 與 `ProblemDetails.invalidParams`。
- TS 29.520 Release 18 V18.14.0 §5.5.8，以及 TS 29.500 Release 18
  §6.6.2：`suppFeats` negotiation、resource scope 與 feature-specific
  information 使用條件。
- TS 29.500 Release 18 §5.2.7.2：schema-invalid IE 的 HTTP error mapping。
- TS 29.520 Release 19 V19.7.0 §5.5.8 與 Release 20 V20.0.0 §5.5.8：
  feature 1 `UnsubscribeWithInfo` 與 feature 2 `EnAccuracyInformation` 的既有
  numbering。
- TS 29.571 Release 18 `NfInstanceId`、`DateTime` 與 `Uinteger` common data
  types。
- [Release 18 Nnwdaf_MLModelTraining OpenAPI](../../../specs/openapi/TS29520_Nnwdaf_MLModelTraining.yaml)
- [Release 18 Nnwdaf_MLModelProvision OpenAPI](../../../specs/openapi/TS29520_Nnwdaf_MLModelProvision.yaml)
- [標準欄位與 Extension 邊界](./standard_field_extension_boundary.md)
