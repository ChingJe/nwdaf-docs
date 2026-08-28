# Branch NWDAF 的 FL 能力、TAI 與 NRF Discovery

## 1. 文件目的

本筆記整理 hierarchical NWDAF Federated Learning 討論中，Branch NWDAF 應如何
透過 NRF profile 表達 FL 能力與適用區域。重點包括：

- `FL_SERVER`、`FL_CLIENT` 與 `FL_SERVER_AND_CLIENT` 的程序語意；
- `Nnwdaf_MLModelTraining_Subscribe` 的發起方與接收方；
- 將 Server／Client 能力分成不同 `MlAnalyticsInfo` entries 是否可行；
- `NwdafInfo.taiList` 與 `MlAnalyticsInfo.trackingAreaList` 的語意差異；
- Root 如何查詢同時具備 Server 與 Client 能力的 Branch candidates。

本文件不把 Root／Branch／Leaf topology 寫入 NRF。NRF 負責保存與回傳候選
NWDAF 的標準能力；Root 取得 discovery result 後，才自行選擇 Branch、Leaf 並建立
topology。

---

## 2. FL Server 與 FL Client 的程序方向

### 2.1 角色是針對一個 FL process 定義

在一個標準 FL process 中：

| FL 角色 | `Nnwdaf_MLModelTraining` 程序責任 | SBI 位置 |
| --- | --- | --- |
| FL Server NWDAF | 選擇 FL Clients、提供初始或 aggregated model、發起 preparation／training request、接收 local model information、進行 aggregation | `Nnwdaf_MLModelTraining` service consumer |
| FL Client NWDAF | 接收 preparation／training request、使用 local data 訓練、回傳 local model information 或 training status | `Nnwdaf_MLModelTraining` service producer |

TS 23.288 §6.2C.2.1 step 7 將方向定義為 FL Server NWDAF 向 FL Client
NWDAF 發送 preparation request；可使用
`Nnwdaf_MLModelTraining_Subscribe` 或
`Nnwdaf_MLModelTrainingInfo_Request`。§6.2C.2.2 step 2 對正式 training 也使用相同
方向：FL Server 向選定的 FL Clients 發出 request，要求它們執行 local training。

因此，若只討論 `Nnwdaf_MLModelTraining_Subscribe`：

```text
FL Server NWDAF
    -- POST /nnwdaf-mlmodeltraining/v1/subscriptions -->
FL Client NWDAF
```

- 「發起 Model Training subscription」對應 FL Server 的程序責任；
- 「接受並處理 Model Training subscription」對應 FL Client 的程序責任；
- FL Client 應提供可供呼叫的 `nnwdaf-mlmodeltraining` service endpoint；
- FL Server 在這次呼叫中是 NF service consumer，不是因為名稱為 Server 就成為
  `Nnwdaf_MLModelTraining` 的 HTTP service producer。

TS 29.520 §4.6.2 對 API boundary 的描述也一致：
`Nnwdaf_MLModelTraining_Subscribe` 由 NF service consumer 發起，向含有 MTLF 的
NWDAF 建立 ML Model Training subscription。

規格依據：

- [TS 23.288 §6.2C.2.1 與 §6.2C.2.2](../../../../specs/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2C%20Federated%20Learning%20among%20Multiple%20NWDAFs.md)
- [TS 29.520 §4.6 Nnwdaf_MLModelTraining Service](../../../../specs/TS%2029.520/4%20Services%20offered%20by%20the%20NWDAF/4.6%20Nnwdaf_MLModelTraining%20Service.md)

### 2.2 Hierarchical topology 中的 Branch

Branch 同時參與兩個相連、但彼此不同的 FL processes：

```text
Upper-tier process
Root (FL Server) --> Branch (FL Client)

Lower-tier process
Branch (FL Server) --> Leaf (FL Client)
```

所以 `FL_SERVER_AND_CLIENT` 表示 Branch 具有兩種 FL 能力；它不表示 Branch 必須
在同一個 FL process 中同時充當 Server 與 Client，也不表示 NRF 已經知道它是
哪個 Root 的 Client 或哪些 Leafs 的 Server。

---

## 3. NRF profile 中的 FL capability

TS 23.288 §6.2C.2.1 說明，含有 MTLF 的 NWDAF 可在 NRF profile 中註冊：

- Analytics ID；
- NWDAF address information；
- Service Area；
- FL capability type；
- 支援 FL 的 time interval。

TS 29.510 將 `FlCapabilityType` 定義為三個不同值：

```text
FL_SERVER
FL_CLIENT
FL_SERVER_AND_CLIENT
```

這些值是 capability 宣告。實際某次 FL process 中的角色，仍由 discovery、selection
及後續 training procedure 決定。

規格依據：

- [TS 23.288 §6.2C.2.1 Registration and Discovery](../../../../specs/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2C%20Federated%20Learning%20among%20Multiple%20NWDAFs.md)
- [TS 29.510 FlCapabilityType](../../../../specs/TS%2029.510/6%20API%20Definitions/6.1%20Nnrf_NFManagement%20Service%20API/6.1.6%20Data%20Model/6.1.6.3%20Simple%20data%20types%20and%20enumerations/6.1.6.3.19%20Enumeration%20FlCapabilityType.md)

---

## 4. 兩個 TAI 欄位的語意

TAI 在 NWDAF profile 中出現在不同層級，不能視為完全相同的欄位。

| Profile 位置 | 欄位 | 規格語意 |
| --- | --- | --- |
| `NwdafInfo` 層級 | `taiList`／`taiRangeList` | 此 NWDAF instance 整體可以服務的 TAIs；兩者皆省略時，表示可針對 serving network 中任何 TAI 被選擇 |
| `MlAnalyticsInfo` entry | `trackingAreaList` | 該 ML model／Analytics capability 的 Area of Interest；省略時，該模型能力可適用任何 TAI |

例如：

```json
{
  "nwdafInfo": {
    "taiList": [
      {
        "plmnId": { "mcc": "001", "mnc": "01" },
        "tac": "000001"
      },
      {
        "plmnId": { "mcc": "001", "mnc": "01" },
        "tac": "000002"
      }
    ],
    "mlAnalyticsList": [
      {
        "mlAnalyticsIds": ["NF_LOAD"],
        "trackingAreaList": [
          {
            "plmnId": { "mcc": "001", "mnc": "01" },
            "tac": "000002"
          }
        ],
        "flCapabilityType": "FL_SERVER_AND_CLIENT"
      }
    ]
  }
}
```

這份 profile 表示：

1. NWDAF instance 整體可服務 `000001` 與 `000002`；
2. `NF_LOAD` 的這筆 FL 能力只宣告適用於 `000002`；
3. 對 `000002` 的 `NF_LOAD` model，該 NWDAF 同時具備 FL Server 與 FL Client
   capability。

因此，`NwdafInfo.taiList` 適合回答「這個 NF 整體可服務哪裡」，而
`MlAnalyticsInfo.trackingAreaList` 適合回答「這一組 Analytics／model／FL
capability 適用哪裡」。

規格依據：

- [TS 29.510 NwdafInfo](../../../../specs/TS%2029.510/6%20API%20Definitions/6.1%20Nnrf_NFManagement%20Service%20API/6.1.6%20Data%20Model/6.1.6.2%20Structured%20data%20types/6.1.6.2.45%20Type%20NwdafInfo.md)
- [TS 29.510 MlAnalyticsInfo](../../../../specs/TS%2029.510/6%20API%20Definitions/6.1%20Nnrf_NFManagement%20Service%20API/6.1.6%20Data%20Model/6.1.6.2%20Structured%20data%20types/6.1.6.2.84%20Type%20MlAnalyticsInfo.md)

---

## 5. Server 與 Client 能力依 TAI 分開宣告

### 5.1 分成不同 `MlAnalyticsInfo` entries 是可行的

如果同一個 NWDAF 的能力確實隨 Area 改變，可以分成不同 entries：

```json
{
  "nwdafInfo": {
    "mlAnalyticsList": [
      {
        "mlAnalyticsIds": ["NF_LOAD"],
        "trackingAreaList": [
          {
            "plmnId": { "mcc": "001", "mnc": "01" },
            "tac": "000001"
          }
        ],
        "flCapabilityType": "FL_CLIENT"
      },
      {
        "mlAnalyticsIds": ["NF_LOAD"],
        "trackingAreaList": [
          {
            "plmnId": { "mcc": "001", "mnc": "01" },
            "tac": "000002"
          }
        ],
        "flCapabilityType": "FL_SERVER"
      }
    ]
  }
}
```

其語意為：

```text
NF_LOAD + TAI 000001 -> FL Client capability
NF_LOAD + TAI 000002 -> FL Server capability
```

這種表示不需要建立兩個 `nfInstanceId`，也不需要把同一個實體 NWDAF 註冊兩次；
同一份 `NwdafInfo.mlAnalyticsList` 就能承載不同的能力組合。

`MlAnalyticsInfo` 的規格也要求：若 Analytics IDs 的其他屬性值不同，應拆到不同
`MlAnalyticsInfo` instances。因此，使用不同 entries 表達 area-dependent
capability，符合 schema 的分組方式。

### 5.2 同一個 Branch 管轄區域可宣告雙能力

另一種情況是：Branch 管理 `000002`，並且對同一區域的模型同時執行：

- upper-tier：接受 Root 發出的 training subscription，作為 FL Client；
- lower-tier：向 Leafs 發出 training subscriptions，作為 FL Server；
- 聚合 Leafs 的 local model information，再向 Root 回報 upper-tier result。

如果這兩種能力都適用 `000002`，可以使用同一筆 entry：

```json
{
  "mlAnalyticsIds": ["NF_LOAD"],
  "trackingAreaList": [
    {
      "plmnId": { "mcc": "001", "mnc": "01" },
      "tac": "000002"
    }
  ],
  "flCapabilityType": "FL_SERVER_AND_CLIENT"
}
```

這項表示的設計理由不是「標準要求所有 Branch 都必須宣告雙能力」，而是：在目前
提出的 hierarchical topology 中，Branch 對自己負責的 model AoI 確實需要同時
提供兩種能力。若部署中的能力真的依不同 Area 分離，上一節的兩-entry 表示仍然是
有效選項。

---

## 6. Root 查詢 Branch candidates

### 6.1 `ml-analytics-info-list` matching 語意

以下討論採用未帶 `complex-query` 的一般 discovery request。TS 29.510 對
`ml-analytics-info-list` 定義以下 matching 規則：

1. query list 中有多個 `MlAnalyticsInfo` objects 時，NWDAF profile 支援其中至少一筆
   就可被回傳，亦即 list entries 之間是 OR；
2. 同一個 query `MlAnalyticsInfo` object 內若包含多個 attributes，profile 中必須
   有同一筆 `MlAnalyticsInfo` 同時符合全部 attributes，亦即同一 entry 內是 AND；
3. `mlAnalyticsIds`、`trackingAreaList` 等 array attributes，只要 query 與 profile
   至少有一個共同元素即可匹配。

所以，把以下兩筆放進同一次 query：

```json
[
  {
    "trackingAreaList": [
      {
        "plmnId": { "mcc": "001", "mnc": "01" },
        "tac": "000001"
      }
    ],
    "flCapabilityType": "FL_CLIENT"
  },
  {
    "trackingAreaList": [
      {
        "plmnId": { "mcc": "001", "mnc": "01" },
        "tac": "000002"
      }
    ],
    "flCapabilityType": "FL_SERVER"
  }
]
```

只能表示「符合其中一筆即可」，不能保證回傳的同一個 NWDAF 同時具備兩筆能力。
若 Root 要求候選 Branch 對同一 model AoI 直接宣告兩種能力，查詢
`FL_SERVER_AND_CLIENT` 會更直接，也不需要在收到 profile 後再合併檢查兩筆 entries。

規格依據：

- [TS 29.510 NF Discovery GET：ml-analytics-info-list](../../../../specs/TS%2029.510/6%20API%20Definitions/6.2%20Nnrf_NFDiscovery%20Service%20API/6.2.3%20Resources/6.2.3.2%20Resource%20nf-instances%20(Store)/6.2.3.2.3.1%20GET.md)

### 6.2 Branch candidate query 範例

以下是標準 Nnrf NF Discovery endpoint 的概念範例。Root 不帶
`target-nf-instance-id`，因為此時目標是取得所有符合條件的 Branch candidates：

```bash
curl --get \
  'http://nrf.example.net/nnrf-disc/v1/nf-instances' \
  --data-urlencode 'target-nf-type=NWDAF' \
  --data-urlencode 'requester-nf-type=NWDAF' \
  --data-urlencode 'service-names=nnwdaf-mlmodeltraining' \
  --data-urlencode 'tai-list=[{
    "plmnId":{"mcc":"001","mnc":"01"},
    "tac":"000002"
  }]' \
  --data-urlencode 'ml-analytics-info-list=[{
    "mlAnalyticsIds":["NF_LOAD"],
    "trackingAreaList":[{
      "plmnId":{"mcc":"001","mnc":"01"},
      "tac":"000002"
    }],
    "flCapabilityType":"FL_SERVER_AND_CLIENT"
  }]'
```

這個 query 同時要求：

- target 是 NWDAF；
- profile 提供 `nnwdaf-mlmodeltraining`，因此 Root 能取得可接受 training
  subscription 的 service endpoint；
- NWDAF instance 整體支援 TAI `000002`；
- 同一筆 `MlAnalyticsInfo` 對 `NF_LOAD`、TAI `000002` 宣告
  `FL_SERVER_AND_CLIENT`。

`tai-list` query parameter 的標準語意是：NRF 應回傳支援清單中所有 TAIs 的 NFs。
相對地，`ml-analytics-info-list[].trackingAreaList` 按 ML analytics matching 規則採
array overlap。兩者同時出現在一般 discovery request 時，都是候選 profile 必須通過
的 top-level query conditions。

若兩個 Branch instances 都符合，NRF 會在 `SearchResult.nfInstances` 回傳兩份
`NFProfile`。以下只節錄本次討論需要的能力與 endpoint 欄位：

```json
{
  "validityPeriod": 100,
  "nfInstances": [
    {
      "nfInstanceId": "branch-nwdaf-01",
      "nfType": "NWDAF",
      "nwdafInfo": {
        "mlAnalyticsList": [
          {
            "mlAnalyticsIds": ["NF_LOAD"],
            "trackingAreaList": [
              {
                "plmnId": { "mcc": "001", "mnc": "01" },
                "tac": "000002"
              }
            ],
            "flCapabilityType": "FL_SERVER_AND_CLIENT"
          }
        ]
      },
      "nfServiceList": {
        "training-service-01": {
          "serviceInstanceId": "training-service-01",
          "serviceName": "nnwdaf-mlmodeltraining",
          "versions": [
            {
              "apiVersionInUri": "v1",
              "apiFullVersion": "1.0.0"
            }
          ],
          "scheme": "http",
          "nfServiceStatus": "REGISTERED",
          "apiPrefix": "http://10.10.0.21:8000"
        }
      }
    },
    {
      "nfInstanceId": "branch-nwdaf-02",
      "nfType": "NWDAF",
      "nwdafInfo": {
        "mlAnalyticsList": [
          {
            "mlAnalyticsIds": ["NF_LOAD"],
            "trackingAreaList": [
              {
                "plmnId": { "mcc": "001", "mnc": "01" },
                "tac": "000002"
              }
            ],
            "flCapabilityType": "FL_SERVER_AND_CLIENT"
          }
        ]
      },
      "nfServiceList": {
        "training-service-02": {
          "serviceInstanceId": "training-service-02",
          "serviceName": "nnwdaf-mlmodeltraining",
          "versions": [
            {
              "apiVersionInUri": "v1",
              "apiFullVersion": "1.0.0"
            }
          ],
          "scheme": "http",
          "nfServiceStatus": "REGISTERED",
          "apiPrefix": "http://10.10.0.22:8000"
        }
      }
    }
  ]
}
```

NRF 回傳的是符合條件的 NF profiles，不是把 query attributes 原樣投影成 response。
因此 Root 除了能力資料，也能取得已註冊的 address、service version 與 endpoint，並在
discovery 後自行選擇 Branch 和建立 topology。

規格依據：

- [TS 29.510 NF Discovery GET](../../../../specs/TS%2029.510/6%20API%20Definitions/6.2%20Nnrf_NFDiscovery%20Service%20API/6.2.3%20Resources/6.2.3.2%20Resource%20nf-instances%20(Store)/6.2.3.2.3.1%20GET.md)
- [TS 29.510 SearchResult](../../../../specs/TS%2029.510/6%20API%20Definitions/6.2%20Nnrf_NFDiscovery%20Service%20API/6.2.6%20Data%20Model/6.2.6.2%20Structured%20data%20types/6.2.6.2.2%20Type%20SearchResult.md)
- [Release 18 Nnrf_NFDiscovery OpenAPI](../../../../specs/openapi/TS29510_Nnrf_NFDiscovery.yaml)

---

## 7. 討論整理

本次討論可收斂為以下幾點：

1. Server／Client capability 與 Root／Branch／Leaf topology 是不同層次的資訊；NRF
   提供前者，Root 在 discovery 後建立後者。
2. 將 `FL_CLIENT` 與 `FL_SERVER` 分別放在不同 `MlAnalyticsInfo` entries，並配上
   不同 `trackingAreaList`，在 schema 上是可行的，適合能力確實依 model AoI 分離
   的 NWDAF。
3. 若 Branch 對自己負責的同一 model AoI，對上需要接受 Root 的 training
   subscription、對下需要向 Leafs 發起 training subscriptions，則
   `FL_SERVER_AND_CLIENT` 能直接表達該區域的雙能力。
4. 對 Root 而言，使用單一 `MlAnalyticsInfo` query 同時指定 Analytics ID、
   `trackingAreaList` 與 `FL_SERVER_AND_CLIENT`，比從多筆 OR-matched entries 推導
   「同一個 instance 具有兩種能力」更直接。
5. `NwdafInfo.taiList` 是 instance-level serving area；
   `MlAnalyticsInfo.trackingAreaList` 是 model／analytics capability 的 AoI。即使值
   相同，也不應把兩者解讀成同一個欄位。

以上說明支持的是目前 topology 下較容易 discovery 的 profile 表示，不排除未來在
能力確實依 Area 分離時，改用多筆 `MlAnalyticsInfo` entries。
