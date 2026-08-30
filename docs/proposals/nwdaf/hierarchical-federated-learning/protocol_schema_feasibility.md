# Hierarchical NWDAF FL Protocol／Schema Feasibility

日期：2026-08-30

狀態：討論草稿；持續更新中

## 1. 文件目的

本文件整理 Hierarchical NWDAF FL 的 protocol／schema extension 可行性。內容以
Release 18 `Nnwdaf_MLModelTraining` 為現有基礎，記錄已完成討論的候選schema、
procedure與其適用邊界。

本文件記錄已完成的schema討論，並將Branch replacement與殘留計算結果的handoff
拆成基礎替換流程與尚待定義的replay／re-binding語意。以下欄位與schema名稱皆為
candidate extension，不是現行Release 18定義。

本文以YAML呈現OpenAPI schema與候選資料結構，僅供閱讀與schema討論；實際
`Nnwdaf_MLModelTraining` service operation透過HTTP傳送JSON body。

## 2. FL topology 逐級下發

### 2.1 設計目標

FL Server 與 FL Client 是單一 FL process 中的相對角色，不應在 protocol schema 中固定
Root、Branch 或 Leaf，也不應將 topology 限制成三層。

候選表示方式是一棵可遞迴的 subtree。每個 node 只保存自己的 `nfInstanceId` 與下一層
`children`。收到 subtree 的 NWDAF確認根節點是自己，對每個直接 child建立
`Nnwdaf_MLModelTraining` subscription，並把該 child對應的subtree繼續往下傳遞。
沒有`children`的node不再建立下一層subscription。

### 2.2 YAML 結構示意（非傳輸範例）

以下YAML只用來說明subtree資料形狀，不代表實際wire format。範例表示`branch-1`所對應的
subtree；實際`nfInstanceId`值使用標準UUID格式。

```yaml
flTopology:
  nfInstanceId: branch-1
  children:
    - nfInstanceId: branch-2
      children:
        - nfInstanceId: client-a
        - nfInstanceId: client-b

    - nfInstanceId: branch-3
      children:
        - nfInstanceId: client-c
        - nfInstanceId: client-d
```

`branch-1`依此對`branch-2`與`branch-3`建立subscriptions，並分別將以該child為根的
subtree繼續下發。Node是否為中間聚合者，只由它是否具有`children`以及它在當次process
中的相對位置決定。

### 2.3 OpenAPI 遞迴 schema

OpenAPI 3.0可使用self-referencing `$ref`表達上述遞迴結構。候選欄位可掛在
`NwdafMLModelTrainSubsc`，型別則由`FlTopologyNode`遞迴定義：

```yaml
components:
  schemas:
    NwdafMLModelTrainSubsc:
      type: object
      properties:
        # Existing Release 18 fields are omitted here.
        flTopology:
          $ref: '#/components/schemas/FlTopologyNode'

    NwdafMLModelTrainSubscPatch:
      type: object
      properties:
        # Existing Release 18 fields are omitted here.
        flTopology:
          $ref: '#/components/schemas/FlTopologyNode'

    FlTopologyNode:
      type: object
      required:
        - nfInstanceId
      properties:
        nfInstanceId:
          $ref: 'TS29571_CommonData.yaml#/components/schemas/NfInstanceId'
        children:
          type: array
          minItems: 1
          items:
            $ref: '#/components/schemas/FlTopologyNode'
```

`children`不是required property，因此末端node只需要`nfInstanceId`。OpenAPI schema可驗證
遞迴資料形狀，但「subtree根節點必須等於request接收者」屬於procedure／runtime validation，
無法只靠schema表達。

### 2.4 Training lifecycle 中的 topology 更新

`flTopology`不限定於preparation。建立subscription時可用它下發初始subtree；training lifecycle
期間也可在`NwdafMLModelTrainSubscPatch`中傳送新的`flTopology`，調整接收者目前管理的
downstream topology。

候選更新語意採用完整subtree replacement：PATCH body中只要出現`flTopology`，該值就是以
request接收者為根的最新完整subtree，而不是對`children`逐項合併。接收者比較新舊subtree後，
為新增的direct child建立subscription、刪除已移除child的subscription，並將仍存在但subtree
內容改變的child透過PATCH繼續逐級更新。這種語意也符合JSON Merge Patch對array採整體替換的
行為，避免新增、刪除與重新指派node時產生歧義。

以下範例表示training期間將原本的第一個downstream node替換成新的participant；實際欄位名稱與
更新procedure仍屬candidate extension：

```http
:method: PATCH
:scheme: https
:authority: branch-1.example.org
:path: /nnwdaf-mlmodeltraining/v1/subscriptions/upper-training-subscription
content-type: application/merge-patch+json
accept: application/json

{
  "flTopology": {
    "nfInstanceId": "11111111-1111-4111-8111-111111111111",
    "children": [
      {
        "nfInstanceId": "44444444-4444-4444-8444-444444444444",
        "children": [
          {
            "nfInstanceId": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa"
          },
          {
            "nfInstanceId": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb"
          }
        ]
      },
      {
        "nfInstanceId": "33333333-3333-4333-8333-333333333333",
        "children": [
          {
            "nfInstanceId": "cccccccc-cccc-4ccc-8ccc-cccccccccccc"
          },
          {
            "nfInstanceId": "dddddddd-dddd-4ddd-8ddd-dddddddddddd"
          }
        ]
      }
    ]
  }
}
```

若既有intermediate node已完全失效而無法接收PATCH，上游不能只靠修改該失效subscription完成
替換；它仍須對替代node建立新的subscription。替代node如何接續原training process、原下游
nodes如何重新binding，留待Branch replacement部分討論。

### 2.5 目前結論

- 以遞迴subtree逐級下發topology在OpenAPI 3.0中可以直接表達。
- Schema不需要定義Root、Branch或Leaf role，也不限制topology深度。
- 每一層使用相同的Model Training subscription與subtree forwarding規則。
- `flTopology`可在subscription建立與後續PATCH中使用；PATCH中的值代表接收者最新的完整
  subtree。
- 本節只確認schema representation feasibility；field naming與完整procedure wording尚未定案。

## 3. 組織自定義 strategy

### 3.1 使用範圍

`strategy`作為optional candidate extension，由部署組織定義實際內容。Protocol只提供承載與
逐級轉送方式，不固定FL method、aggregation方式或其他參數，也不將它限定在preparation階段。

- 建立subscription與後續PATCH都可以帶入`strategy`。
- 收到`strategy`的中間node須將它逐級傳給其direct children。
- `strategy`的生效時點、沿用、替換或合併規則，不由本candidate extension固定，而由
  `strategyId`所識別的組織自定義contract決定。

Participant admission policy目前不納入candidate extension。Root可依逐級回傳的結果決定是否
保留或刪除相關subscriptions，因此本文件先不定義`admission.mode`。

### 3.2 YAML 結構示意（非傳輸範例）

以下YAML只用來說明`strategy`資料形狀；實際service request使用JSON body。

```yaml
strategy:
  strategyId: example.org/federated-training-v1
  parameters:
    method: fedProx
    aggregation: sampleWeighted
```

`strategyId`用來識別組織定義的strategy contract；`parameters`內容保持開放。上述`method`與
`aggregation`只是容易理解的使用範例，不是protocol固定enum。

### 3.3 OpenAPI schema

Release 18 `Nnwdaf_MLModelTraining`使用OpenAPI 3.0.0；可用
`additionalProperties: true`表達組織自定義的parameters：

```yaml
components:
  schemas:
    Strategy:
      type: object
      required:
        - strategyId
      properties:
        strategyId:
          type: string
          description: Organization-defined strategy identifier.
          example: example.org/federated-training-v1
        parameters:
          type: object
          additionalProperties: true
          description: Organization-defined strategy parameters.
          example:
            method: fedProx
            aggregation: sampleWeighted
```

這種表示方式確認schema可行性，但不同組織之間若要互通，仍須對`strategyId`及對應parameters
另有共同約定。

### 3.4 帶有實際值的 HTTP subscription request 範例

以下是Root向第一層participant建立preparation subscription時送出的HTTP request示意。為了
清楚表示5G SBI使用的HTTP/2語意，範例列出pseudo-headers；實際framing由HTTP stack處理。
POST request的media type依Release 18 OpenAPI為`application/json`。既有Release 18欄位負責
event、callback、correlation、model address、training requirement與reporting deadline；
`flTopology`及`strategy`是目前討論的candidate extensions：

```http
:method: POST
:scheme: https
:authority: branch-1.example.org
:path: /nnwdaf-mlmodeltraining/v1/subscriptions
content-type: application/json
accept: application/json

{
  "mLEventSubscs": [
    {
      "mLEvent": "UE_COMMUNICATION",
      "mLEventFilter": {},
      "modelInterInfo": "pymtlf-model-bundle-v1"
    }
  ],
  "notifUri": "https://root.example.org/nnwdaf-mlmodeltraining/v1/notifications",
  "notifCorreId": "root-to-branch-1-training-subscription",
  "mlCorreId": "hfl-training-2026-001",
  "mLPreFlag": true,
  "mLModelInfos": [
    {
      "event": "UE_COMMUNICATION",
      "mLFileAddr": {
        "mLModelUrl": "https://root.example.org/artifacts/base-model.tar.gz"
      }
    }
  ],
  "eventReq": {
    "notifMethod": "ON_EVENT_DETECTION"
  },
  "mLModelTrainInfos": [
    {
      "dataAvReq": {
        "inpEvents": [
          {
            "upfEvent": "USER_DATA_USAGE_TRENDS"
          }
        ],
        "minNumSamples": 1,
        "timeWindows": [
          {
            "startTime": "2026-08-30T09:00:00Z",
            "stopTime": "2026-08-30T10:00:00Z"
          }
        ]
      },
      "timeAvReq": "PT300S"
    }
  ],
  "mLTrainRepInfo": {
    "maxResTime": 300
  },
  "flTopology": {
    "nfInstanceId": "11111111-1111-4111-8111-111111111111",
    "children": [
      {
        "nfInstanceId": "22222222-2222-4222-8222-222222222222",
        "children": [
          {
            "nfInstanceId": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa"
          },
          {
            "nfInstanceId": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb"
          }
        ]
      },
      {
        "nfInstanceId": "33333333-3333-4333-8333-333333333333",
        "children": [
          {
            "nfInstanceId": "cccccccc-cccc-4ccc-8ccc-cccccccccccc"
          },
          {
            "nfInstanceId": "dddddddd-dddd-4ddd-8ddd-dddddddddddd"
          }
        ]
      }
    ]
  },
  "strategy": {
    "strategyId": "example.org/federated-training-v1",
    "parameters": {
      "method": "fedProx",
      "aggregation": "sampleWeighted"
    }
  }
}
```

此例中`flTopology`的根節點是本次request的接收者。接收者依兩個direct children建立下一層
subscriptions，並分別下發對應subtree。若training lifecycle後續透過PATCH傳送`strategy`，
request body仍使用JSON，media type則依Release 18 OpenAPI為`application/merge-patch+json`；
生效與更新語意由`strategyId`所識別的組織contract決定。

## 4. Topology status 逐級回報

### 4.1 設計目標

目前實作把preparation結果放在`mLModelUrl`指向的model bundle config中，以assigned、prepared、
failed與timed-out client清單表示。Candidate extension改由`NwdafMLModelTrainNotif`直接攜帶
遞迴的`flTopologyStatus`；tree中的每個node保存自己的`nfInstanceId`與目前狀態，中間node收集
direct children的status subtree後，再組回自己的subtree逐級向上通知。

`flTopologyStatus`不限定於preparation。Training期間若某個downstream participant失敗、退出或
其subscription被刪除，中間node也可使用同一欄位通知上游。Model或model update本身仍由既有
`mLModelInfos`與model URL傳遞；topology status不取代artifact transport。

### 4.2 YAML 結構示意（非傳輸範例）

以下YAML只用來說明status subtree的資料形狀，不代表實際wire format。範例表示一個Leaf的
subscription被取消，其餘nodes仍可參與training：

```yaml
flTopologyStatus:
  nfInstanceId: branch-1
  status: ACTIVE
  children:
    - nfInstanceId: branch-2
      status: ACTIVE
      children:
        - nfInstanceId: client-a
          status: ACTIVE
        - nfInstanceId: client-b
          status: INACTIVE
          cause: SUBSCRIPTION_CANCELLED
```

實際notification透過HTTP傳送JSON body，並使用標準UUID格式的`nfInstanceId`。

### 4.3 最小 status 集合

`status`只表示node目前是否仍可參與該training topology，不承擔round progression或完整training
lifecycle的所有狀態：

- `ACTIVE`：node已就緒且仍可參與。
- `INACTIVE`：node因正常或受控原因停止參與，例如subscription被取消。
- `FAILED`：node因異常狀況無法參與。

更詳細的原因放在optional `cause`中。`TIMEOUT`等狀況是`FAILED`的cause，而不是持續擴充
`status` enum。

### 4.4 OpenAPI 遞迴 schema

```yaml
components:
  schemas:
    NwdafMLModelTrainNotif:
      type: object
      properties:
        # Existing Release 18 fields are omitted here.
        flTopologyStatus:
          $ref: '#/components/schemas/FlTopologyStatusNode'

    FlTopologyStatusNode:
      type: object
      required:
        - nfInstanceId
        - status
      properties:
        nfInstanceId:
          $ref: 'TS29571_CommonData.yaml#/components/schemas/NfInstanceId'
        status:
          type: string
          enum:
            - ACTIVE
            - INACTIVE
            - FAILED
        cause:
          type: string
        children:
          type: array
          minItems: 1
          items:
            $ref: '#/components/schemas/FlTopologyStatusNode'
```

### 4.5 帶有實際值的 HTTP notification 範例

以下範例表示training期間一個Leaf的subscription被取消後，containing intermediate node把更新後的
status subtree通知上游。`roundInd`與其他既有欄位仍依原training procedure使用；本例只凸顯新增的
`flTopologyStatus`：

```http
:method: POST
:scheme: https
:authority: root.example.org
:path: /nnwdaf-mlmodeltraining/v1/notifications
content-type: application/json

{
  "notifCorreId": "root-to-branch-1-training-subscription",
  "mlCorreId": "hfl-training-2026-001",
  "roundInd": 3,
  "flTopologyStatus": {
    "nfInstanceId": "11111111-1111-4111-8111-111111111111",
    "status": "ACTIVE",
    "children": [
      {
        "nfInstanceId": "22222222-2222-4222-8222-222222222222",
        "status": "ACTIVE",
        "children": [
          {
            "nfInstanceId": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
            "status": "ACTIVE"
          },
          {
            "nfInstanceId": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb",
            "status": "INACTIVE",
            "cause": "SUBSCRIPTION_CANCELLED"
          }
        ]
      }
    ]
  }
}
```

在preparation結果中，tree本身可表達原本的assigned client集合；`ACTIVE` nodes對應prepared
clients，`FAILED`加`cause`對應failed或timed-out clients。因此不需要另外維護互相重複的
assigned、prepared、failed與timed-out清單。

目前先確認遞迴status tree與最小status集合可覆蓋已知需求。

## 5. Branch replacement與殘留結果handoff

### 5.1 問題情境

本節探討的不是一般性的dynamic topology reconfiguration，而是一個較具體的failure
recovery問題：某個intermediate node在training期間失效後，Root如何以新node取代它，
原本的downstream Leaves又能否將已經計算完成、但尚未成功回報的結果交給新node繼續聚合。

這個問題需要區分兩個階段：

1. **Topology replacement**：偵測與舊Branch之間的training subscriptions已無法繼續，改由
   新Branch與原Leaves建立subscriptions。
2. **Retained-result handoff**：將Leaves在舊subscriptions下已完成或保留的結果，重新
   binding至新Branch所建立的subscriptions。

### 5.2 基礎失效偵測與topology replacement流程

Release 18已提供這個流程需要的部分基礎機制：notification送往subscription中的
`notifUri`，並且notification的`notifCorreId`必須與對應subscription相同；
`mLTrainRepInfo.maxResTime`則表示等待notifications的最長時間。但規格沒有直接定義
hierarchical topology中的Branch replacement procedure。

因此，目前的candidate failure-handling flow為：

1. Leaf將結果回報給舊Branch時，若notification在組織設定的retry或timeout policy後仍無法
   送達，Leaf將該callback path視為unavailable。是否立即終止或刪除本地subscription，屬於
   implementation／organization policy，不是前述Release 18欄位已定義的行為。
2. Root在原本設定的`maxResTime`內沒有收到舊Branch的notification時，可依組織政策將舊
   Branch視為本次training中已unavailable，並將原upper-tier subscription視為無法繼續。
   這個判定無法單憑timeout區分process crash、network partition或單純運算過慢。
3. Root選擇一個新Branch，向它建立新的Model Training subscription，並將原本由舊Branch
   管理的subtree改由新Branch接手。因為舊Branch已不可達，這不能只靠PATCH舊Branch的
   subscription完成。
4. 新Branch依收到的`flTopology`對原Leaves建立新subscriptions。Leaves後續回報時使用
   新subscriptions對應的`notifUri`與`notifCorreId`，至此完成topology replacement。

這個流程不需要從舊Branch取回狀態就能重建topology；但它也不會自然保留Leaves在舊
subscriptions下已計算的結果。

### 5.3 舊計算結果不會自然轉移

Leaf原本要傳給舊Branch的notification對應舊subscription，其`notifUri`與`notifCorreId`
也都屬於舊Branch。新Branch重建subscription後會產生新的callback與correlation context。
Release 18目前提供的Subscribe／Notify機制沒有直接說明：在舊subscription下已計算、但
回報失敗的結果，可以在新subscription成立後當作新notification重新送出。

若要避免Leaves全部重新計算，candidate extension需要定義retained-result replay／re-binding
contract：

1. Leaf在向舊Branch回報失敗後，保留model update artifact與其process／round context，而不是在
   舊subscription失效後立即刪除。
2. 新Branch向Leaf建立subscription後，Leaf可回報自己保留的進度與結果狀態。
3. Leaf使用新subscription的`notifCorreId`傳送notification，同時攜帶足以識別舊結果所屬
   FL procedure與round的context；實際artifact仍可以透過既有`mLModelInfos`與model URL傳遞。
4. 新Branch依組織定義的`strategy`決定接受、忽略或去除重複結果，以及從哪個進度繼續。

這四點是candidate procedure，不是單純加入一個schema field就會自然成立的Release 18行為。

### 5.4 `retainedResultReq` candidate extension

為了顯式觸發前述retained-result replay，可以在`NwdafMLModelTrainSubsc`加入optional
`retainedResultReq`。這個欄位指定要查詢的舊`mlCorreId`；收到request的NWDAF需要查找
該`mlCorreId`在本地最新、已完成且仍保留的結果。

不將這個候選欄位定義為boolean，是因為新subscription的`mlCorreId`與要取回的舊process
ID不一定相同。若組織讓整棵hierarchy共用同一個`mlCorreId`，兩者可以使用相同值；
若組織為不同local processes分配不同IDs，`retainedResultReq.mlCorreId`則明確指向舊process。

Candidate OpenAPI schema如下：

```yaml
components:
  schemas:
    NwdafMLModelTrainSubsc:
      type: object
      properties:
        # Existing Release 18 fields are omitted here.
        retainedResultReq:
          $ref: '#/components/schemas/RetainedResultRequest'

    RetainedResultRequest:
      type: object
      required:
        - mlCorreId
      properties:
        mlCorreId:
          type: string
          description: Identifies the FL procedure whose latest retained result is requested.
```

回傳不需要新增service operation。Release 18已允許consumer在`eventReq.immRep`設為`true`時，
由subscription create response的`immReport`回傳可用report。因此candidate procedure規定：

- `retainedResultReq`出現時，`eventReq.immRep`必須為`true`；這是cross-field procedure validation，
  不只是上述OpenAPI資料形狀的驗證。
- 若找到retained result，`immReport.notifCorreId`使用新subscription的correlation ID，
  `immReport.mlCorreId`表示該retained result原本所屬的FL procedure。
- `immReport.roundInd`回傳本地保留結果中最高的已完成round；尚在計算的round不屬於
  「最新已完成結果」。
- `immReport.mLModelInfos`攜帶model update information，實際artifact仍透過model URL取得。

以下範例表示新Branch建立Leaf subscription時，同時要求舊FL process的最新retained
result。新subscription與舊process在範例中使用不同的`mlCorreId`，以顯示這兩個scope不必
相同：

```http
:method: POST
:scheme: https
:authority: leaf-a.example.org
:path: /nnwdaf-mlmodeltraining/v1/subscriptions
content-type: application/json
accept: application/json

{
  "mLEventSubscs": [
    {
      "mLEvent": "UE_COMMUNICATION",
      "mLEventFilter": {},
      "modelInterInfo": "pymtlf-model-bundle-v1"
    }
  ],
  "notifUri": "https://new-branch.example.org/nnwdaf-mlmodeltraining/v1/notifications",
  "notifCorreId": "new-branch-to-leaf-a-subscription",
  "mlCorreId": "replacement-lower-process-001",
  "eventReq": {
    "notifMethod": "ON_EVENT_DETECTION",
    "immRep": true
  },
  "retainedResultReq": {
    "mlCorreId": "old-lower-process-001"
  }
}
```

若Leaf保留的最新已完成結果為round 5，create response可以使用既有`immReport`形狀
回傳：

```http
:status: 201
location: https://leaf-a.example.org/nnwdaf-mlmodeltraining/v1/subscriptions/replacement-subscription
content-type: application/json

{
  "mLEventSubscs": [
    {
      "mLEvent": "UE_COMMUNICATION",
      "mLEventFilter": {},
      "modelInterInfo": "pymtlf-model-bundle-v1"
    }
  ],
  "notifUri": "https://new-branch.example.org/nnwdaf-mlmodeltraining/v1/notifications",
  "notifCorreId": "new-branch-to-leaf-a-subscription",
  "mlCorreId": "replacement-lower-process-001",
  "eventReq": {
    "notifMethod": "ON_EVENT_DETECTION",
    "immRep": true
  },
  "retainedResultReq": {
    "mlCorreId": "old-lower-process-001"
  },
  "immReport": {
    "notifCorreId": "new-branch-to-leaf-a-subscription",
    "mlCorreId": "old-lower-process-001",
    "roundInd": 5,
    "mLModelInfos": [
      {
        "event": "UE_COMMUNICATION",
        "mLFileAddr": {
          "mLModelUrl": "https://leaf-a.example.org/artifacts/old-lower-process-001/round-5.tar.gz"
        }
      }
    ]
  }
}
```

若沒有找到對應的retained result，只在`201 Created`中省略`immReport`，無法說明是本地
從未產生matching result、結果尚未完成，或結果已被cleanup。因此negative result的精確
HTTP／schema representation仍需另行定義；本節先確認成功路徑可以重用既有`immReport`。

`retainedResultReq`只負責觸發與識別retained-result replay。新Branch是否接受、去重或捨棄
該結果，仍由組織定義的`strategy`決定。

### 5.5 `mlCorreId`的scope不能預先寫死

Hierarchical FL中`mlCorreId`的scope取決於組織如何管理FL processes。組織可以讓整棵
hierarchy tree共用同一個`mlCorreId`，將它視為同一個hierarchical FL procedure；也可以讓
每個local FL process使用不同的`mlCorreId`。Candidate extension不應固定其中一種。

共用`mlCorreId`可以幫助新Branch識別retained result仍屬於同一個hierarchical FL procedure，
但`mlCorreId`本身不能說明該結果的local round、是否已被聚合，也不能單獨決定該結果
是否還可接受。若每個local process使用不同的`mlCorreId`，則intermediate node還需要知道自己
與downstream processes之間的identifier mapping。這項mapping可以放入前述遞迴status tree，再由
各層逐級向上回報。

### 5.6 `roundInd`與Root的state visibility

即使整棵hierarchy共用同一個`mlCorreId`，`roundInd`是否共用仍然是獨立的管理決策。
Upper-tier與lower-tier可能使用不同的aggregation cadence，lower-tier也可能進行多次round
才進入一次global aggregation。同一local process中的Clients也可能因計算時間不同，而保留不同
的進度；例如某個Client計算太久，已被Server在當次aggregation跳過。

候選方式是讓status tree中的每個node可帶有其`mlCorreId`、`roundInd`與結果狀態。但Root
看到的只會是最近一次逐級回報的last-known snapshot。Lower-tier不是每一次local round都會
進入global aggregation，也不一定每次都向Root發送notification，因此Root無法單憑自己保留的
`roundInd`推導所有Leaves的即時進度。

新Branch接手後，應從Leaves重新收集它們實際保留的process ID、round進度與結果狀態，
再依`strategy`決定resume point與可接受的retained results。若組織要求Root掌握更即時的lower-tier
state，就需要增加status notification頻率、periodic checkpoint或shared persistent state，並承擔
額外Root-facing communication與state management cost。

## 6. 更新紀錄

| 日期 | 更新 |
| --- | --- |
| 2026-08-30 | 建立討論草稿；確認role-neutral recursive subtree及OpenAPI self-referencing schema可行 |
| 2026-08-30 | 加入可由subscription create與PATCH承載的組織自定義strategy candidate extension；暫不討論admission mode |
| 2026-08-30 | 加入Model Training subscription整合`flTopology`與`strategy`的HTTP／JSON request範例；strategy更新語意改由組織contract定義 |
| 2026-08-30 | 將`flTopology`擴展至training期間PATCH，並以完整subtree replacement定義topology更新語意 |
| 2026-08-30 | 加入可供preparation與training lifecycle共用的遞迴`flTopologyStatus`、最小status集合及HTTP notification範例 |
| 2026-08-31 | 以Branch失效、topology replacement與retained-result handoff重新整理recovery問題，並區分現有機制與candidate procedure |
| 2026-08-31 | 加入`retainedResultReq`候選欄位，以既有`immRep`／`immReport`觸發與回傳指定`mlCorreId`的最新已完成結果 |
