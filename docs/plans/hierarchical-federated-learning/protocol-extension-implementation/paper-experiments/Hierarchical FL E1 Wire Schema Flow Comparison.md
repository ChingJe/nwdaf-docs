# Hierarchical FL E1 傳遞 Schema 範例：既有候選契約與論文附錄對照

狀態：兩版對照保留作差異參考；後續實作沿用版本一的既有候選 schema，本文不修改任一候選 schema。

本文用同一個 E1 情境，逐步展示**既有候選契約**與**新版論文附錄契約**在 NWDAF 間傳遞的訊息：Root 原本透過 A 管理 A1／A2；A 在訓練中失效；Root 選擇預部署的 A*；A* 對原 Leaves 建立新訂閱，回報拓樸後參與後續訓練。B／C 與其 Leaves 持續使用原有訂閱。版本一依據[既有候選 OpenAPI](../../../../design/hierarchical-federated-learning/candidate_openapi.yaml)與[欄位語意](../../../../design/hierarchical-federated-learning/candidate_openapi_schema.md)，是後續實作基準；版本二依據論文 *Resilient Hierarchical Federated Learning for NWDAF: Protocol Extensions and a free5GC-Based Realization* 第 4 節及附錄 B，僅保留作歷史比較與論文修訂參考。兩者都不是已採納的 3GPP extension；版本二不是現有程式已實作的 wire format，也不是本批遷移目標。

本文只列出 **NWDAF ↔ NWDAF 的 `Nnwdaf_MLModelTraining` 訊息類型**，涵蓋建立、preparation 回報、正常 round、失效後的新訂閱與恢復貢獻。同一 schema 在 A1／A2 或 B／C 重複時，列出兩條 edge 的實際識別值，不複製相同的 JSON。HTTP 範例保留本情境需要的標準欄位；未展示認證、完整模型下載協定及模型二進位內容。`X_IMAGE_CLASSIFICATION` 和 `modelInterInfo` 值是本專案的實驗工作負載約定，並非 3GPP 新增的標準事件。

## 共同情境與訊息順序

| 節點 | `nfInstanceId` |
| --- | --- |
| A | `10000000-0000-4000-8000-000000000101` |
| A* | `10000000-0000-4000-8000-000000000111` |
| A1 | `10000000-0000-4000-8000-000000001101` |
| A2 | `10000000-0000-4000-8000-000000001102` |

所有 edge 共用 `mlCorreId: hfl-example-001`，但每條 edge 有自己的 `notifCorreId` 與接收端建立的訂閱資源。下列 UUID、URL 與時間均為示意值，不是既有 testbed run 的原始紀錄：

| Edge | `notifCorreId` | 示意 Location 末段 |
| --- | --- | --- |
| Root → A | `root-a` | `20000000-0000-4000-8000-000000000101` |
| A → A1／A2 | `a-a1`／`a-a2` | `20000000-0000-4000-8000-000000001101`／`20000000-0000-4000-8000-000000001102` |
| Root → A* | `root-a-star` | `20000000-0000-4000-8000-000000000111` |
| A* → A1／A2 | `a-star-a1`／`a-star-a2` | `30000000-0000-4000-8000-000000001101`／`30000000-0000-4000-8000-000000001102` |

兩個版本的傳遞順序相同，差別在 request／report 的拓樸欄位：

| 時點 | 方向與操作 | 傳遞內容／結果 |
| --- | --- | --- |
| 建立 | Root → A：Create；A → A1、A2：Create | 標準 training requirement、`mlCorreId`、拓樸 instruction；各接收端回 `201 Created`、`Location` 與 accepted subscription representation |
| 確認 | A1、A2 → A：Notify；A → Root：Notify | Leaf preparation outcome；A 彙整自己的 direct edges 後向 Root 回報；Root **內部判斷**是否接受 realized topology |
| 訓練 | Root → A、A → A1／A2：PATCH；A1／A2 → A、A → Root：Notify | 沿用 `roundInd`、`mLModelInfos` 與原訂閱；B／C 的對應訊息同樣持續，沒有重新建立其訂閱 |
| 故障 | A 停止回應；Root 等候逾時 | 故障注入、deadline／timeout 與 Root 的偵測決策**不是** `flTopologyReport`／`x-flTopologyReport`；已失效的 A 不可能再發一份可靠的故障回報 |
| 接手 | Root → A*：Create；A* → A1、A2：Create | Root 把接手指令發給**新 A***，不是發給已失效的 A；舊 A→Leaf 資源不必先成功清除，新的 A*→Leaf 資源仍有新的 ID |
| 恢復 | A1、A2 → A*：Notify；A* → Root：Notify；之後雙向 round 訊息 | Root 接受修復後的 realized topology，再提供當前模型與訓練指令；A* 第一次成功向 Root 貢獻，才算恢復參與 |

各 Create 成功時的 response schema 仍是 `NwdafMLModelTrainSubsc`，並以 `Location` 指向**接收端建立**的 individual subscription resource；拓樸 instruction 會位於同一個 subscription representation。各 Notify 成功時回 `204 No Content`。A 已不可達時，對其舊訂閱執行 `DELETE` 不是建立 A*→Leaf 新 edge 的前置條件；本流程沒有假造 A 發出的 `termTrainReq` 或成功的舊資源刪除回覆。

### 兩版共用的標準 round 訊息

Preparation Create 不攜帶 `mLModelInfos`；模型在拓樸確認後的訓練更新中傳遞。下例的 `PATCH` 是 Root 傳給 A；Root 傳給 A*、以及 Branch 傳給 Leaf 時沿用同一標準 `NwdafMLModelTrainSubscPatch` schema，改用該 edge 的訂閱 URI 與相應模型位置。Root global model 可由 ADRF 提供；Branch 的區域聚合模型可由 Branch 自己暫存。這些 artifact 位置不屬於兩種拓樸欄位的差異。

```http
PATCH /nnwdaf-mlmodeltraining/v1/subscriptions/20000000-0000-4000-8000-000000000101 HTTP/1.1
Host: a.example.org
Content-Type: application/merge-patch+json

{
  "mLPreFlag": false,
  "roundInd": 0,
  "mLModelInfos": [
    {
      "event": "X_IMAGE_CLASSIFICATION",
      "mLModelAdrf": {
        "adrfId": "10000000-0000-4000-8000-000000009001",
        "storTransId": "global-model-round-0"
      }
    }
  ]
}
```

每條被 PATCH 的資源分別回標準的 `200 OK`（更新後的 subscription representation）或 `204 No Content`。Leaf 向自己的 parent、Branch 向 Root 報告本輪模型時，使用標準 `NwdafMLModelTrainNotif`；下例是 A1 → A，A2、B／C 子樹以及接手後 A1 → A* 採同一個 schema，使用各自的 `notifCorreId`、模型位置與 `roundInd`：

```http
POST /callbacks/ml-model-training HTTP/1.1
Host: a.example.org
Content-Type: application/json

{
  "notifCorreId": "a-a1",
  "mlCorreId": "hfl-example-001",
  "roundInd": 0,
  "mLModelInfos": [
    {
      "event": "X_IMAGE_CLASSIFICATION",
      "mLFileAddr": {
        "mLModelUrl": "https://a1.example.org/models/update-round-0"
      }
    }
  ]
}
```

上列只示範一種合法的模型位置表示；實際 artifact 取得與成功的模型內容，不能由 URL 字串本身推定。Root 對修復後的 A* 送出當前模型時，`roundInd` 取當時 round，**不因 A* 是新訂閱就重新開始整個 FL procedure**。論文與既有設計都沒有在拓樸 object 內重新定義 `roundInd`。

修復後 A* → Root 的**首次模型貢獻**也使用同一標準 Notify schema；下面的 `roundInd: 14` 只是假設當時正處理這一輪，並非由拓樸版本 `1` 換算而來：

```http
POST /callbacks/ml-model-training HTTP/1.1
Host: root.example.org
Content-Type: application/json

{
  "notifCorreId": "root-a-star",
  "mlCorreId": "hfl-example-001",
  "roundInd": 14,
  "mLModelInfos": [
    {
      "event": "X_IMAGE_CLASSIFICATION",
      "mLFileAddr": {
        "mLModelUrl": "https://a-star.example.org/models/aggregate-round-14"
      }
    }
  ]
}
```

上述 Notify 成功時，接收方回 `204 No Content`。Root 是否將這次貢獻納入 accepted round 仍取決於該輪的聚合結果，不能僅由 callback 回覆推定。

## 版本一：現行 `x-flTopology`／`x-flTopologyReport` 契約

### 1. Root 建立 A 的上層訂閱

下列是 Root → A 的 preparation Create。`children` 是 A 的候選 direct clients；Root 對 A 自己的候選順位屬於 Root 的上層選擇，不會寫在發給 A 的最外層 node 上。

```http
POST /nnwdaf-mlmodeltraining/v1/subscriptions HTTP/1.1
Host: a.example.org
Content-Type: application/json

{
  "mLEventSubscs": [{
    "mLEvent": "X_IMAGE_CLASSIFICATION",
    "mLEventFilter": {},
    "modelInterInfo": "pymtlf-image-classification-mnist"
  }],
  "notifUri": "https://root.example.org/callbacks/ml-model-training",
  "notifCorreId": "root-a",
  "mlCorreId": "hfl-example-001",
  "mLPreFlag": true,
  "suppFeats": "4",
  "x-flTopology": {
    "nfInstanceId": "10000000-0000-4000-8000-000000000101",
    "policy": {
      "allowAdditionalCandidates": false,
      "selectionMethod": "priority",
      "minAvailableNodes": 2,
      "minTrainNodes": 2
    },
    "strategy": {
      "method": "fedProx",
      "aggregation": "sampleWeighted",
      "methodParameters": { "proximalMu": 0.01 }
    },
    "children": [
      { "nfInstanceId": "10000000-0000-4000-8000-000000001101", "priority": 100 },
      { "nfInstanceId": "10000000-0000-4000-8000-000000001102", "priority": 90 }
    ]
  }
}
```

A 成功建立後回 `201 Created`，以 `Location: https://a.example.org/nnwdaf-mlmodeltraining/v1/subscriptions/20000000-0000-4000-8000-000000000101` 指定資源，response body 是接收的 `NwdafMLModelTrainSubsc` representation；`suppFeats` response 必須確認雙方確實支援此候選功能。**這個 response 只證明 Root→A 訂閱資源存在，不證明 A1／A2 已加入。**

### 2. A 逐一建立下層訂閱並向 Root 回報

A → A1 的 preparation Create 如下；A → A2 使用完全相同的 `NwdafMLModelTrainSubsc` schema，但接收者為 `a2.example.org`、`notifCorreId` 為 `a-a2`，最外層 `nfInstanceId` 為 A2。兩個接收端分別回 `201 Created`，`Location` 末段為 `20000000-0000-4000-8000-000000001101`／`20000000-0000-4000-8000-000000001102`。

```http
POST /nnwdaf-mlmodeltraining/v1/subscriptions HTTP/1.1
Host: a1.example.org
Content-Type: application/json

{
  "mLEventSubscs": [{
    "mLEvent": "X_IMAGE_CLASSIFICATION",
    "mLEventFilter": {},
    "modelInterInfo": "pymtlf-image-classification-mnist"
  }],
  "notifUri": "https://a.example.org/callbacks/ml-model-training",
  "notifCorreId": "a-a1",
  "mlCorreId": "hfl-example-001",
  "mLPreFlag": true,
  "suppFeats": "4",
  "x-flTopology": {
    "nfInstanceId": "10000000-0000-4000-8000-000000001101",
    "strategy": {
      "method": "fedProx",
      "aggregation": "sampleWeighted",
      "methodParameters": { "proximalMu": 0.01 }
    },
    "reportAfter": { "count": 4, "unit": "epoch" }
  }
}
```

Leaf 回報自己的 preparation outcome 時可送 `NwdafMLModelTrainNotif`；以下是 A1 → A，A2 → A 改用 `a-a2` 與 A2 identity。Leaf 的 wrapper **不替自己的 upstream edge 填 status**；A 才是該 edge 狀態的 owner。

```http
POST /callbacks/ml-model-training HTTP/1.1
Host: a.example.org
Content-Type: application/json

{
  "notifCorreId": "a-a1",
  "mlCorreId": "hfl-example-001",
  "x-flTopologyReport": {
    "nfInstanceId": "10000000-0000-4000-8000-000000001101"
  }
}
```

A 確認兩條 direct edges 可參與後，向 Root 發送下列 Notify。Root 依此與自己的其他子樹資料判定是否接受 realized topology；這個接受決策沒有另定一個訊息欄位。

```http
POST /callbacks/ml-model-training HTTP/1.1
Host: root.example.org
Content-Type: application/json

{
  "notifCorreId": "root-a",
  "mlCorreId": "hfl-example-001",
  "x-flTopologyReport": {
    "nfInstanceId": "10000000-0000-4000-8000-000000000101",
    "children": [
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000001101",
        "status": "ACTIVE",
        "statusTimestamp": "2026-09-20T08:00:00Z"
      },
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000001102",
        "status": "ACTIVE",
        "statusTimestamp": "2026-09-20T08:00:01Z"
      }
    ]
  }
}
```

### 3. A 失效後，Root 對 A* 建立新訂閱

原設計**沒有** `reparentInstruction`。Root 是以發給 A* 的 `x-flTopology.children` 指定它需要嘗試的 A1／A2，接手原因由 Root 的選擇狀態及這次新訂閱的上下文表達；此 payload 本身沒有宣告「failed parent 是 A」。

```http
POST /nnwdaf-mlmodeltraining/v1/subscriptions HTTP/1.1
Host: a-star.example.org
Content-Type: application/json

{
  "mLEventSubscs": [{
    "mLEvent": "X_IMAGE_CLASSIFICATION",
    "mLEventFilter": {},
    "modelInterInfo": "pymtlf-image-classification-mnist"
  }],
  "notifUri": "https://root.example.org/callbacks/ml-model-training",
  "notifCorreId": "root-a-star",
  "mlCorreId": "hfl-example-001",
  "mLPreFlag": true,
  "suppFeats": "4",
  "x-flTopology": {
    "nfInstanceId": "10000000-0000-4000-8000-000000000111",
    "policy": {
      "allowAdditionalCandidates": false,
      "selectionMethod": "priority",
      "minAvailableNodes": 2,
      "minTrainNodes": 2
    },
    "strategy": {
      "method": "fedProx",
      "aggregation": "sampleWeighted",
      "methodParameters": { "proximalMu": 0.01 }
    },
    "children": [
      { "nfInstanceId": "10000000-0000-4000-8000-000000001101", "priority": 100 },
      { "nfInstanceId": "10000000-0000-4000-8000-000000001102", "priority": 90 }
    ]
  }
}
```

A* 回 `201 Created`，`Location` 末段為 `20000000-0000-4000-8000-000000000111`。接著 A* → A1／A2 重用本版第 2 步的 Create schema，`notifUri` 改為 `https://a-star.example.org/callbacks/ml-model-training`，`notifCorreId` 分別改為 `a-star-a1`／`a-star-a2`；A1／A2 分別建立 **新**資源 `30000000-0000-4000-8000-000000001101`／`30000000-0000-4000-8000-000000001102`。原本由 A 建立的兩筆資源不會被改名或重用。

A1／A2 → A* 重用本版第 2 步的 Leaf Notify schema，使用新的 notification correlation。A* 確認兩條 direct edges 後向 Root 送出實際修復結果：

```http
POST /callbacks/ml-model-training HTTP/1.1
Host: root.example.org
Content-Type: application/json

{
  "notifCorreId": "root-a-star",
  "mlCorreId": "hfl-example-001",
  "x-flTopologyReport": {
    "nfInstanceId": "10000000-0000-4000-8000-000000000111",
    "children": [
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000001101",
        "status": "ACTIVE",
        "statusTimestamp": "2026-09-20T08:10:00Z"
      },
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000001102",
        "status": "ACTIVE",
        "statusTimestamp": "2026-09-20T08:10:01Z"
      }
    ]
  }
}
```

隨後 Root → A*、A* → Leaves 重用前述 round PATCH／Notify schema 送出當前模型並取得新貢獻；沒有 `x-retainedResultReq`，本例不取用舊計算結果。

## 版本二：論文附錄 B 的 `flTopology`／`flTopologyReport` 比較契約

論文把 topology instruction 改成 `FLTopologyInstruction`：`addressedNodeId` 指接收者，`candidates[].childInstruction` 可再逐級下發，且每份 instruction 都要求 `topologyVersion`。論文附錄沒有給 `suppFeats` feature number，也未給完整 presence／cardinality 規則；因此以下是依附錄 B 組成的**可比較訊息示意**，不是已可直接部署的完整 Stage-3 API。

### 1. Root 建立 A 的上層訂閱

同一個標準 `NwdafMLModelTrainSubsc` envelope，替換的是 topology property 及其內部結構。初次形成的示意 `topologyVersion` 為 `0`；論文沒有規定所有 implementation 必須以零開始。

```http
POST /nnwdaf-mlmodeltraining/v1/subscriptions HTTP/1.1
Host: a.example.org
Content-Type: application/json

{
  "mLEventSubscs": [{
    "mLEvent": "X_IMAGE_CLASSIFICATION",
    "mLEventFilter": {},
    "modelInterInfo": "pymtlf-image-classification-mnist"
  }],
  "notifUri": "https://root.example.org/callbacks/ml-model-training",
  "notifCorreId": "root-a",
  "mlCorreId": "hfl-example-001",
  "mLPreFlag": true,
  "flTopology": {
    "topologyVersion": 0,
    "addressedNodeId": "10000000-0000-4000-8000-000000000101",
    "allowNrfDiscovery": false,
    "minDirectChildren": 2,
    "candidates": [
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000001101",
        "priority": 100,
        "childInstruction": {
          "topologyVersion": 0,
          "addressedNodeId": "10000000-0000-4000-8000-000000001101"
        }
      },
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000001102",
        "priority": 90,
        "childInstruction": {
          "topologyVersion": 0,
          "addressedNodeId": "10000000-0000-4000-8000-000000001102"
        }
      }
    ]
  }
}
```

A 回 `201 Created` 與 `Location: https://a.example.org/nnwdaf-mlmodeltraining/v1/subscriptions/20000000-0000-4000-8000-000000000101`。論文沿用 subscription resource，但沒有另外定義「`minDirectChildren` 對應到原設計哪組 `minAvailableNodes`／每輪 `minTrainNodes`」的完整轉換，兩者不能直接視為重新命名。

### 2. A 下發 child instruction，Leaf／A 逐級回報

A → A1 用 `childInstruction` 形成新的 direct-child Create；A → A2 的 schema 相同，`notifCorreId` 與 `addressedNodeId` 換成 `a-a2` 與 A2。各 Leaf 回 `201 Created`，`Location` 末段分別為 `20000000-0000-4000-8000-000000001101`／`20000000-0000-4000-8000-000000001102`。

```http
POST /nnwdaf-mlmodeltraining/v1/subscriptions HTTP/1.1
Host: a1.example.org
Content-Type: application/json

{
  "mLEventSubscs": [{
    "mLEvent": "X_IMAGE_CLASSIFICATION",
    "mLEventFilter": {},
    "modelInterInfo": "pymtlf-image-classification-mnist"
  }],
  "notifUri": "https://a.example.org/callbacks/ml-model-training",
  "notifCorreId": "a-a1",
  "mlCorreId": "hfl-example-001",
  "mLPreFlag": true,
  "flTopology": {
    "topologyVersion": 0,
    "addressedNodeId": "10000000-0000-4000-8000-000000001101"
  }
}
```

A1 → A 的 Notify 使用論文的 `FLTopologyReport`；A2 同樣以自己的 identity 及 `a-a2` 回報。`nodeState` 是該 report node 的 readiness，**不是** A 對 A1／A2 的 direct-edge 決定。

```http
POST /callbacks/ml-model-training HTTP/1.1
Host: a.example.org
Content-Type: application/json

{
  "notifCorreId": "a-a1",
  "mlCorreId": "hfl-example-001",
  "flTopologyReport": {
    "topologyVersion": 0,
    "reportingNodeId": "10000000-0000-4000-8000-000000001101",
    "nodeState": "READY"
  }
}
```

A 才在自己的 direct-edge report 填入 `CONFIRMED`，並把 Leaf report 放入 `childReport`：

```http
POST /callbacks/ml-model-training HTTP/1.1
Host: root.example.org
Content-Type: application/json

{
  "notifCorreId": "root-a",
  "mlCorreId": "hfl-example-001",
  "flTopologyReport": {
    "topologyVersion": 0,
    "reportingNodeId": "10000000-0000-4000-8000-000000000101",
    "nodeState": "READY",
    "directEdges": [
      {
        "childNfId": "10000000-0000-4000-8000-000000001101",
        "edgeState": "CONFIRMED",
        "childReport": {
          "topologyVersion": 0,
          "reportingNodeId": "10000000-0000-4000-8000-000000001101",
          "nodeState": "READY"
        }
      },
      {
        "childNfId": "10000000-0000-4000-8000-000000001102",
        "edgeState": "CONFIRMED",
        "childReport": {
          "topologyVersion": 0,
          "reportingNodeId": "10000000-0000-4000-8000-000000001102",
          "nodeState": "READY"
        }
      }
    ]
  }
}
```

Root 內部接受此 realized topology 後，正常 round 的 PATCH／Notify 沿用前述標準 `roundInd`、`mLModelInfos`，不必每輪重送 `flTopology` 或建立新訂閱。

### 3. A 失效後，Root 明確指示 A* 接手

相較版本一，這份新 Create 除列出 A1／A2，還透過 `reparentInstruction` 明確告知**接收者 A***：原 parent 是 A、獲授權接手的 children 是 A1／A2。示意版本前進到 `1`；這是拓樸版本，不是新的 `mlCorreId` 或訓練 `roundInd`。

```http
POST /nnwdaf-mlmodeltraining/v1/subscriptions HTTP/1.1
Host: a-star.example.org
Content-Type: application/json

{
  "mLEventSubscs": [{
    "mLEvent": "X_IMAGE_CLASSIFICATION",
    "mLEventFilter": {},
    "modelInterInfo": "pymtlf-image-classification-mnist"
  }],
  "notifUri": "https://root.example.org/callbacks/ml-model-training",
  "notifCorreId": "root-a-star",
  "mlCorreId": "hfl-example-001",
  "mLPreFlag": true,
  "flTopology": {
    "topologyVersion": 1,
    "addressedNodeId": "10000000-0000-4000-8000-000000000111",
    "allowNrfDiscovery": false,
    "minDirectChildren": 2,
    "candidates": [
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000001101",
        "priority": 100,
        "childInstruction": {
          "topologyVersion": 1,
          "addressedNodeId": "10000000-0000-4000-8000-000000001101"
        }
      },
      {
        "nfInstanceId": "10000000-0000-4000-8000-000000001102",
        "priority": 90,
        "childInstruction": {
          "topologyVersion": 1,
          "addressedNodeId": "10000000-0000-4000-8000-000000001102"
        }
      }
    ],
    "reparentInstruction": {
      "failedParentNfId": "10000000-0000-4000-8000-000000000101",
      "adoptChildNfIds": [
        "10000000-0000-4000-8000-000000001101",
        "10000000-0000-4000-8000-000000001102"
      ]
    }
  }
}
```

A* 回 `201 Created` 與新的 `Location`；接著 A* → A1／A2 重用本版第 2 步的 Create schema，`topologyVersion` 改為 `1`，`notifUri` 改為 A*，`notifCorreId` 分別為 `a-star-a1`／`a-star-a2`。**`reparentInstruction` 不會原封不動轉送給 Leafs**：A* 在各自的 direct-child 訂閱中下發該 Leaf 的 `childInstruction`。A1／A2 回應後形成 `30000000-0000-4000-8000-000000001101`／`30000000-0000-4000-8000-000000001102` 新資源，再用本版第 2 步的 Leaf Notify schema 回報 `READY`；新通知改用版本 `1` 及新的 correlation。

A* → Root 的修復回報仍使用同一 `flTopologyReport` schema，但與第一次形成時不同，它回報新版拓樸及修復後的 A* subtree。以下以附錄 B 列出的 `RESTORED` 作示意；附錄沒有完整定義何時必須從 `READY` 轉成 `RESTORED`，不能據此範例宣稱已定案的狀態轉移規則：

```http
POST /callbacks/ml-model-training HTTP/1.1
Host: root.example.org
Content-Type: application/json

{
  "notifCorreId": "root-a-star",
  "mlCorreId": "hfl-example-001",
  "flTopologyReport": {
    "topologyVersion": 1,
    "reportingNodeId": "10000000-0000-4000-8000-000000000111",
    "nodeState": "RESTORED",
    "directEdges": [
      {
        "childNfId": "10000000-0000-4000-8000-000000001101",
        "edgeState": "CONFIRMED",
        "childReport": {
          "topologyVersion": 1,
          "reportingNodeId": "10000000-0000-4000-8000-000000001101",
          "nodeState": "READY"
        }
      },
      {
        "childNfId": "10000000-0000-4000-8000-000000001102",
        "edgeState": "CONFIRMED",
        "childReport": {
          "topologyVersion": 1,
          "reportingNodeId": "10000000-0000-4000-8000-000000001102",
          "nodeState": "READY"
        }
      }
    ]
  }
}
```

是否把整個 repaired topology 接受為新版本，由 Root 依 policy **內部判斷**；論文沒有另定一個「接受成功」的回覆訊息。之後 Root → A*、A* → A1／A2 使用前述標準 round PATCH 傳當前模型；A1／A2 → A*、A* → Root 使用標準模型 Notify。A* 第一次成功向 Root 報告模型後，Root 的 accepted round 才能將 A* 列為成功貢獻者。

## 對照時不能跳過的界線

| 問題 | 版本一 | 版本二 |
| --- | --- | --- |
| 拓樸欄位 | `x-flTopology`／`x-flTopologyReport` | `flTopology`／`flTopologyReport` |
| Root → A* 的接手語意 | 透過 A* 的 `children` 指派 A1／A2；未在 payload 中宣告失效 parent | `reparentInstruction.failedParentNfId` 與 `adoptChildNfIds` 明確表達 |
| 遞迴與回報 | `children` tree；child `status`／`statusTimestamp`／`statusCause` | `candidates[].childInstruction`；`nodeState` 與 `directEdges[].edgeState`／`failureCause` |
| 版號 | 無 protocol `topologyVersion` | instruction／report 必帶 `topologyVersion` |
| 訓練與 participant 設定 | node 的 `policy`、`strategy`、`reportAfter` 有候選 schema | 附錄 B 只列 `allowNrfDiscovery`、`minDirectChildren`、`maxDepthBelow` 等；未說明如何等價承載原本的訓練與每輪門檻 |
| 功能協商 | 候選 feature 3，示意 `suppFeats: "4"` | 論文要求協商但未分配 feature number；**無法產出已定案的數值** |
| retained result | 原設計有一次性 lookup 擴充；本 E1 範例刻意未啟用 | 附錄 B 未列此擴充；不可據此推論已正式刪除 |

最重要的是：**兩版都把接手指令發給新 A***，A 不會收到修復請求；舊 A→A1／A2 的資源也不會被直接改名為 A*→A1／A2。`mlCorreId`、標準模型／round 欄位以及未受影響的 Root→B／C 訂閱在兩版中維持原用途。後續以版本一作為實作基準；這份比較不把附錄 B 特有欄位當作版本一已具備的能力，也不宣稱版本二已完成實作。
