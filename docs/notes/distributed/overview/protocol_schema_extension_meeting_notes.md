# Hierarchical NWDAF FL Protocol／Schema Extension 會議說明筆記

日期：2026-08-31

## 1. 說明目的

這次主要說明兩個問題：

1. 現在放在model bundle config中的topology指示與participant結果，如果改由
   `Nnwdaf_MLModelTraining` protocol message承載，可以如何表示。
2. Training期間若某個intermediate node失效，Root如何用新node取代它，以及原本的
   downstream clients能否把尚未成功回報的計算結果交給新node繼續處理。

以下欄位與procedure都是candidate extensions，不是現行Release 18定義。完整schema、HTTP
範例與規格邊界記錄於
[Protocol／Schema Feasibility 討論稿](../../../proposals/nwdaf/hierarchical-federated-learning/protocol_schema_feasibility.md)。

## 2. Topology指示與結果移入protocol message

### 2.1 為什麼要移動

目前實作透過model bundle config夾帶Root的downstream topology指示，也把Branch收集到的
participant結果放回bundle。這種方式可以支援同一套實作，但topology control與training
artifact綁在同一個vendor-specific檔案格式中，protocol message本身看不出完整的階層關係與
各participant狀態。

候選方向是讓model bundle繼續承載model與model update，將topology control及status搬到
subscription、PATCH與notification message。

| 現在由bundle config承載的資訊 | 候選protocol欄位 |
| --- | --- |
| 下一層participants及其階層關係 | `flTopology` |
| FL method、aggregation等部署設定 | `strategy` |
| assigned、prepared、failed、timed-out participants | `flTopologyStatus` |
| Model或model update artifact | 維持使用既有`mLModelInfos`與model URL |

### 2.2 `flTopology`逐級下發

`flTopology`使用遞迴subtree，不在schema中固定Root、Branch或Leaf，也不限制topology只能有
三層。每個接收者只需要知道自己的subtree：它對direct children建立subscriptions，再把每個
child對應的subtree繼續下發。

```text
Root
  └─ subscription: subtree(Branch A)
       Branch A
         └─ subscription: subtree(Branch B)
              Branch B
                ├─ subscription: Leaf A
                └─ subscription: Leaf B
```

Subscription建立時可用`flTopology`下發初始subtree；training期間也可以透過PATCH替換接收者
目前管理的完整subtree。因此，上游在participant失效或重新分配時，仍可使用相同欄位更新
topology。

以下只保留辨識同一次preparation及說明candidate extensions所需的欄位；其餘既有Release 18
subscription fields省略：

```http
:method: POST
:scheme: https
:authority: branch-a.example.org
:path: /nnwdaf-mlmodeltraining/v1/subscriptions
content-type: application/json

{
  "mlCorreId": "hfl-preparation-2026-001",
  "mLPreFlag": true,
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

`strategy`保留為組織自定義欄位。Protocol只負責承載與逐級傳送，不把FL method、aggregation
方式或生效語意固定成通用enum。

### 2.3 `flTopologyStatus`逐級回報

回報方向同樣使用遞迴tree。每個node記錄自己的`nfInstanceId`與最小狀態，中間node收集
direct children的status subtree後，再把自己的subtree逐級通知上游。

最小狀態只保留：

- `ACTIVE`：已就緒且仍可參與。
- `INACTIVE`：因正常或受控原因停止參與。
- `FAILED`：因異常狀況無法參與。

更詳細的原因放在optional `cause`。以下先以preparation結果為例：Branch已完成preparation，
一個Leaf準備成功，另一個Leaf在期限內未完成。Preparation尚未進入training round，因此範例
不帶`roundInd`。為了和前一個subscription request直接對照，notification同樣只保留
`mlCorreId`與candidate `flTopologyStatus`；其餘既有notification fields省略。

```http
:method: POST
:scheme: https
:authority: root.example.org
:path: /nnwdaf-mlmodeltraining/v1/notifications
content-type: application/json

{
  "mlCorreId": "hfl-preparation-2026-001",
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
            "status": "FAILED",
            "cause": "PREPARATION_TIMEOUT"
          }
        ]
      }
    ]
  }
}
```

這樣即可由同一棵tree表達原本的assigned participants與每個participant的最新狀態，不必再
另外維護互相重複的prepared、failed與timed-out清單。同一個`flTopologyStatus`也能在training
期間回報participant退出、subscription取消或failure；training notification再依既有procedure
攜帶相應的`roundInd`。

## 3. Branch失效、替換與殘留結果handoff

### 3.1 問題發生的順序

假設某個Branch在training途中失效：

1. Leaves已完成本地計算，但依舊subscription中的`notifUri`向舊Branch回報時持續失敗。
2. Root在原先設定的`mLTrainRepInfo.maxResTime`內沒有收到舊Branch的notification，因此依部署
   policy將它視為unavailable。
3. Root選擇新Branch，向新Branch建立subscription，並把原subtree改由新Branch接手。
4. 新Branch依`flTopology`向原Leaves建立新subscriptions，讓後續結果改送到新的callback。

到這一步，topology已經重建；但Leaves在舊subscriptions下已完成的結果不會因此自然轉移到
新Branch。舊結果仍綁定舊subscription的callback與correlation context。

### 3.2 要求Leaf回傳最新保留結果

候選方式是在新Branch建立Leaf subscription時加入`retainedResultReq`，明確指定要查詢哪一個
舊`mlCorreId`的最新已完成結果。Leaf若仍保留該結果，就使用新subscription的callback context
回傳；結果本身仍透過既有model URL傳遞。

```text
Root偵測舊Branch失效
  → Root對新Branch建立subscription並下發原subtree
  → 新Branch對Leaves建立subscriptions，帶入retainedResultReq
  → Leaves回傳舊process的最新完成round與model update
  → 新Branch依strategy判斷接受、去重或捨棄結果
```

```http
:method: POST
:scheme: https
:authority: leaf-a.example.org
:path: /nnwdaf-mlmodeltraining/v1/subscriptions
content-type: application/json

{
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

若Leaf保留的最新完成結果是round 5，candidate procedure可重用create response中的既有
`immReport`回傳舊process context：

```http
:status: 201
location: https://leaf-a.example.org/nnwdaf-mlmodeltraining/v1/subscriptions/replacement-subscription
content-type: application/json

{
  "notifCorreId": "new-branch-to-leaf-a-subscription",
  "mlCorreId": "replacement-lower-process-001",
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

### 3.3 `mlCorreId`與`roundInd`的管理邊界

Candidate extension不預先規定整棵hierarchy必須共用同一個`mlCorreId`。組織可以把整棵tree
視為同一個FL procedure，也可以讓每個local FL process使用不同ID；後者需要由中間node維護
upper-tier與lower-tier process IDs之間的mapping。

即使共用`mlCorreId`，upper-tier與lower-tier的`roundInd`也不一定相同。Lower tier可能完成
多次local aggregation後才向上一層回報一次，因此Root只能掌握最近一次收到的last-known
snapshot，不能從自己的`roundInd`推導每個Leaf的即時進度。

新Branch接手後，應以Leaves實際保留的process ID、完成round與result status為準，再依
`strategy`決定resume point以及哪些舊結果仍可納入聚合。

## 4. 說明收束

第一個部分處理正常training lifecycle中的topology control與status visibility：model artifact仍由
既有model URL傳輸，但topology指示與participant結果改由明確的protocol fields逐級傳遞。

第二個部分處理Branch失效後的恢復：既有subscription機制可作為重新建立topology的基礎，但
舊計算結果不會自然轉移；`retainedResultReq`用來顯式要求Leaf replay最新保留結果，而結果是否
可接受仍由部署組織的training strategy決定。

這份說明目前聚焦schema與procedure可行性，不擴大成完整standard proposal，也不要求在本階段
修改現行實作。
