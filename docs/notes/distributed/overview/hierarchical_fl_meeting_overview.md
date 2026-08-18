# Hierarchical NWDAF Federated Learning 設計導覽

> **核心構想**
>
> 由 Root NWDAF 透過 NRF 感知可用的 NWDAFs、規劃 hierarchical topology，並將
> topology plan 放入 model bundle，透過標準 ML Model Training 介面逐層完成
> preparation 與 training。

---

## 1. 5GC／NWDAF 環境中的核心問題

一般 FL framework 必須先解決三件事：

- FL Server 如何知道有哪些 FL Clients？
- Server 如何理解各 Client 的能力與可用狀態？
- Server 如何向 Client 傳送模型與訓練要求，再取得 model updates？

在 NWDAF 環境中，這些問題不能只靠 framework 內部的 client registry 或私有連線。
NWDAF 間的 discovery 與 communication 必須放回 5GC 架構理解：

~~~mermaid
flowchart LR
    S[FL Server NWDAF]
    NRF[NRF]
    C[Candidate NWDAFs]

    S -->|Nnrf_NFDiscovery_Request| NRF
    NRF -->|NF profiles and capabilities| S
    S <-->|Standard NWDAF SBI| C
~~~

NRF 可以回答「有哪些候選 NWDAFs」以及「它們宣告哪些能力」，但 hierarchical FL
還需要進一步回答：

~~~text
候選NWDAFs
    ↓
誰作為Branch？
    ↓
每個Branch管理哪些Leaf？
    ↓
如何把這項topology決策傳給Branch？
~~~

因此，本設計的第一個問題不是 aggregation algorithm，而是：

> FL Server 如何透過 5GC mechanism 感知環境、建立 candidate inventory，並形成
> 可執行的 hierarchical FL topology？

---

## 2. 3GPP 提供的基礎與尚未定義的部分

### 2.1 Registration 與 discovery

TS 23.288 §6.2C.2.1 說明 NWDAF containing MTLF 可將 FL 相關能力註冊到 NRF：

> “NWDAF containing MTLF as FL Server NWDAF or FL Client NWDAF registers to NRF
> with its NF profile, which includes ... Analytics ID(s), Address information of
> NWDAF, Service Area, FL capability type information ...”

FL Server 可再透過 NRF 發現符合條件的 Clients，條件可包含：

- Analytics ID
- FL capability type
- Service Area
- Time Period of Interest
- ML Model Interoperability Indicator
- Client 可收集資料的 data-source NF type

TS 29.510 定義三種 FL capability：

~~~text
FL_SERVER
FL_CLIENT
FL_SERVER_AND_CLIENT
~~~

因此，同時宣告 FL Server 與 FL Client 能力的 NWDAF，可以成為 Branch candidate：

~~~text
在upper-tier process：FL Client
在lower-tier process：FL Server
~~~

### 2.2 標準尚未直接提供 hierarchy

3GPP 已定義：

- NWDAF FL capability registration 與 discovery
- FL preparation
- FL training rounds
- model notification
- delay、termination 與 participant maintenance

但尚未定義：

- Root／Branch／Leaf topology
- Root 如何指派 Branch 與 Leaf
- Branch 如何把下層 FL process 對應到上層 FL process
- 多個標準 FL processes 如何組成 hierarchical FL

本設計不是新增另一套 FL protocol，而是：

> 使用多個標準 NWDAF FL processes，並補上 topology planning、assignment 與
> cross-process coordination。

規格來源：

- [TS 23.288 §6.2C](../../../../specs/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2C%20Federated%20Learning%20among%20Multiple%20NWDAFs.md)
- [TS 29.510 FlCapabilityType](../../../../specs/TS%2029.510/6%20API%20Definitions/6.1%20Nnrf_NFManagement%20Service%20API/6.1.6%20Data%20Model/6.1.6.3%20Simple%20data%20types%20and%20enumerations/6.1.6.3.19%20Enumeration%20FlCapabilityType.md)

---

## 3. 設計前提與管理權

### 3.1 Root 持有主要 FL 管理權

在一棵 hierarchical FL topology 中，主要管理權由 Root 持有：

- 建立並維護 candidate inventory
- 規劃 Root／Branch／Leaf topology
- 指派 Branch 與 Leaf
- 決定 topology admission policy
- 接收 preparation results
- 接受、補選或重建 topology
- 啟動、推進與結束整體 training

Branch 是 Root 指派的下層協調者：

- 對 Root 是 FL Client
- 對 Leaf 是 FL Server
- 執行 lower-tier preparation 與 training
- 聚合 Leaf results 並向 Root 回報
- 不在 Root 未知情況下自行改變整體 topology

Leaf 負責檢查參與條件，並使用自己的 local dataset 執行 training。

### 3.2 Topology 位置與標準角色分開

| Topology 位置 | Upper-tier role | Lower-tier role |
| --- | --- | --- |
| Root NWDAF | FL Server | — |
| Branch NWDAF | FL Client | FL Server |
| Leaf NWDAF | — | FL Client |

Root、Branch 與 Leaf 不是新的 NF types，也不是 NWDAF 永久固定的身分。它們描述
某次 topology assignment 中的位置；FL Server／FL Client 才是個別 FL process
中的標準角色。

### 3.3 同 vendor 前提

所有 NWDAFs 使用相同 vendor implementation，能共同理解本設計加入的 model bundle
格式。NWDAF 間仍使用 NRF discovery 與標準 SBI；同 vendor 前提只用來定義標準
沒有涵蓋的 bundle metadata 與 hierarchy coordination。

---

## 4. Preparation：Model Training 介面的準備模式

### 4.1 Preparation 使用相同的 Model Training 介面

Preparation 不是另一套獨立 SBI service。它使用：

~~~text
Nnwdaf_MLModelTraining_Subscribe
~~~

並將 <code>NwdafMLModelTrainSubsc.mLPreFlag</code> 設為 <code>true</code>。

TS 29.520 §5.5.6.2.2 的原文是：

> “Indicates whether the subscription is for preparation of ML Model training.
> Set to "true" if it is for ML training preparation, otherwise set to "false".”

並明確要求：

> “It shall be present when the service is for preparation of Federated Learning.”

簡化的標準 request schema：

~~~text
POST {apiRoot}/nnwdaf-mlmodeltraining/v1/subscriptions
~~~

~~~json
{
  "mLEventSubscs": [
    {
      "mLEvent": "NF_LOAD",
      "mLEventFilter": {},
      "modelInterInfo": "vendor-model-format-v1"
    }
  ],
  "notifUri": "https://root.example.com/ml-training/callback",
  "notifCorreId": "prep-root-branch-1",
  "mlCorreId": "fl-upper-001",
  "mLPreFlag": true,
  "mLModelInfos": [
    {
      "event": "NF_LOAD",
      "mLFileAddr": {
        "mLModelUrl": "https://models.example.com/bundles/plan-001"
      }
    }
  ]
}
~~~

此範例只呈現必要結構、FL correlation、preparation flag 與 model URL；實際 request
可再透過 <code>mLModelTrainInfos</code> 表達 data／time availability requirements。

### 4.2 Preparation 與 Training 的差別

| 階段 | 主要目的 | Client 行為 |
| --- | --- | --- |
| Preparation | 確認能否參與本次 FL | 檢查 requirements、model interoperability、資料／時間能力與模型下載 |
| Training | 實際產生 model update | 使用 local data 訓練模型並回傳 interim local model |

TS 23.288 §6.2C.2.1 step 8 對 preparation 的描述是：

> “FL Client NWDAF(s) checks if it can meet the ML Model training requirement
> and/or can successfully download the model ... and decides whether to join the
> Federated Learning process ...”

TS 23.288 §6.2C.2.2 step 4 對 training 的描述是：

> “Each FL Client NWDAF further trains the ML Model provided by the FL Server NWDAF
> based on its local data and reports the interim local ML Model information ...”

兩個階段使用相同的 ML Model Training service，但 <code>mLPreFlag</code> 與 Client
應執行的行為不同。

規格來源：

- [TS 23.288 §6.2C](../../../../specs/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2C%20Federated%20Learning%20among%20Multiple%20NWDAFs.md)
- [TS 29.520 §5.5.6 Data Model](../../../../specs/TS%2029.520/5%20API%20Definitions/5.5%20Nnwdaf_MLModelTraining%20Service%20API/5.5.6%20Data%20Model.md)
- [Release 18 OpenAPI NwdafMLModelTrainSubsc](../../../../specs/openapi/TS29520_Nnwdaf_MLModelTraining.yaml)

---

## 5. 核心機制：Model Bundle Contract

上一章說明 preparation request 可透過標準 `mLModelUrl` 提供模型。NRF discovery
能建立 candidate inventory，但 Root 仍需要一種方式把 topology plan 交給 Branch。
本設計因此擴充 `mLModelUrl` 所指向的 model file packaging，以 model bundle 連接
這兩個階段：

~~~text
NRF discovery
    ↓
Root建立candidate inventory
    ↓
Root規劃hierarchical topology
    ↓
Root將topology plan放入model bundle
    ↓
以標準mLModelUrl傳給Branch
    ↓
Branch解析assignment並建立lower-tier process
~~~

### 5.1 標準欄位與額外用途

TS 29.520 對 <code>mLModelUrl</code> 的定義是：

> “The URL of the ML Model file.”

同一份 data model 對直接承載的 model file 說明：

> “The format of its value is out of 3GPP scope.”

本設計讓 <code>mLModelUrl</code> 指向仍包含有效 model artifact 的 model bundle，
並在 bundle 中加入 hierarchy manifest：

~~~json
{
  "bundleVersion": "1.0",
  "model": {
    "artifact": "model.onnx",
    "interoperability": "vendor-model-format-v1"
  },
  "hierarchyPlan": {
    "planId": "plan-001",
    "assignedClients": [
      {"nfInstanceId": "leaf-1"},
      {"nfInstanceId": "leaf-2"}
    ],
    "admissionPolicy": {
      "mode": "PARTIAL_ALLOWED",
      "minimumClients": 1
    }
  }
}
~~~

這項用法必須明確區分：

- <code>mLModelUrl</code> 與 ML Model Training SBI 是標準欄位與標準介面。
- Hierarchy manifest、assignment 與 result manifest 是本設計加入的語意。
- Model file 內部格式未被 3GPP 規範，也沒有明文禁止附加同 vendor metadata。
- 規格沒有定義 hierarchy manifest 的內容；它是附加於有效 model bundle 的同 vendor
  contract，而不是 3GPP-defined topology property。

因此，正確定位是：

> 使用標準 SBI 欄位傳遞 model bundle URL，並以 vendor-specific contract 定義
> bundle 內的 hierarchy metadata；不能宣稱 3GPP 已定義這項 hierarchy procedure。

### 5.2 各位置新增的責任

| 位置 | Model bundle responsibility |
| --- | --- |
| Root | 建立模型與 topology plan、發布 bundle、維持 URL lifecycle |
| Branch | 下載並驗證 bundle、解析 assignment、透過 NRF 重新確認 Leaf |
| Leaf | 接收該層所需的模型與 training contract，不需理解完整 topology |

### 5.3 Preparation Notify 的成功載體與擴充界線

TS 23.288 §6.2C.2.1 step 9 明確允許 FL Client 使用 Notify 回報是否參與 FL：

> “FL Client NWDAF(s) invokes Nnwdaf_MLModelTraining_Notify ... to indicate to the
> FL Server NWDAF whether it will join the FL procedure ...”

因此，Branch 等待下層 Leafs 完成檢查後，再以非同步 Notify 回報 Root，本身符合
preparation procedure。

TS 29.520 §5.5.6.2.8 NOTE 1 同時要求
<code>NwdafMLModelTrainNotif</code> 至少包含
<code>delayEventNotif</code>、<code>mLModelInfos</code> 或
<code>termTrainReq</code> 其中之一；<code>statusReport</code> 雖可附加，但不能
單獨滿足這項條件。對已成功完成、沒有要求延後或終止的 preparation outcome 而言，
<code>delayEventNotif</code> 與 <code>termTrainReq</code> 都不符合該結果，因此剩下的
標準結果欄位就是 <code>mLModelInfos</code>。

規格沒有逐字寫成「preparation success shall include
<code>mLModelInfos</code>」；這是結合 Stage 2 join decision 與 Stage 3 conditional
fields 所得的規格推論。因此 Client 指回已下載並驗證的有效 input model bundle，
可視為 preparation 成功確認，並不需要先將這種用法判定為不自然。它也不表示 Client
在 preparation 階段產生了新的 interim local model。

本設計還希望在這次 Notify 的 <code>mLModelInfos.mLFileAddr.mLModelUrl</code>
提供一個 result bundle URL。為了維持 model file 的基本解讀，Bundle 仍包含 Root
原先提供的有效 model artifact，但 Branch 不會在 preparation 階段宣稱它是新訓練的
模型；Bundle 另外加入：

~~~json
{
  "preparationResult": {
    "planId": "plan-001",
    "preparedClients": ["leaf-1"],
    "failedClients": [
      {
        "nfInstanceId": "leaf-2",
        "cause": "INSUFFICIENT_DATA"
      }
    ]
  }
}
~~~

這裡需要區分標準相容用法與語意延伸：

- 以 <code>mLModelInfos</code> 回報成功 preparation outcome，有上述 conditional
  fields 的規格推論支撐。
- URL 指回已下載並驗證的 input model bundle，不代表 preparation 產生新的 model
  update。
- 規格將 <code>mLModelInfos</code> 描述為 “Represents the ML Model
  information.”，而 <code>mLModelUrl</code> 是 ML Model file 的 URL。
- 規格沒有定義 bundle 內的 <code>preparationResult</code> schema，也沒有將它標準化為
  topology preparation result channel。
- Model file 內容格式未標準化，也沒有明文禁止在有效 model bundle 中附加這些
  metadata。

因此，這個 URL 在 schema 與標準 Notify 流程中可以被傳遞，但
<code>preparationResult</code> 的內容與解析方式仍是 vendor-specific contract。
它是本方法用來讓 Root 取得 subordinate preparation details 的核心設計。需要明確
揭露的界線是：<code>mLModelInfos</code> 作為成功 outcome 載體可由規格條件推導；
<code>preparationResult</code> 的內容及其位於 model bundle 內的解析方式，則是
vendor-specific contract。

規格來源：

- [TS 23.288 §6.2C.2.1](../../../../specs/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2C%20Federated%20Learning%20among%20Multiple%20NWDAFs.md)
- [TS 29.520 §5.5.6.2.8 NwdafMLModelTrainNotif](../../../../specs/TS%2029.520/5%20API%20Definitions/5.5%20Nnwdaf_MLModelTraining%20Service%20API/5.5.6%20Data%20Model.md)
- [TS 29.520 §5.4.6 Data Model](../../../../specs/TS%2029.520/5%20API%20Definitions/5.4%20Nnwdaf_MLModelProvision%20Service%20API/5.4.6%20Data%20Model.md)
- [Release 18 OpenAPI MLModelAddr](../../../../specs/openapi/TS29520_Nnwdaf_MLModelProvision.yaml)

---

## 6. Hierarchical Topology 與建立流程

### 6.1 可擴充的 topology

在一棵 topology 中只有一個 Root；Branch 數量以及每個 Branch 所管理的 Leaf 數量
都可以擴充：

~~~mermaid
flowchart TB
    ROOT[Root NWDAF]

    B1[Branch 1]
    BDOTS[...]
    BM[Branch M]

    L11[Leaf 1.1]
    L1DOTS[...]
    L1N[Leaf 1.N]

    LM1[Leaf M.1]
    LMDOTS[...]
    LMK[Leaf M.K]

    ROOT --> B1
    ROOT --> BDOTS
    ROOT --> BM

    B1 --> L11
    B1 --> L1DOTS
    B1 --> L1N

    BM --> LM1
    BM --> LMDOTS
    BM --> LMK
~~~

- 圖中的 M、N、K 不是固定數量。
- 不同 Branch 可以管理不同數量的 Leafs。
- 本設計描述可擴充的三層 topology，不預先宣稱支援任意深度的 recursive hierarchy。

Hierarchy 由多個標準 FL processes 組成：

~~~text
Upper-tier process
Root：FL Server
└─ Branches：FL Clients

Lower-tier process
Branch：FL Server
└─ Leaves：FL Clients
~~~

每個 process 各自維護 subscription、<code>mlCorreId</code>、notification correlation
與 round state，不要求整棵 topology 共用同一組 process identifier。

### 6.2 Topology establishment 流程

~~~mermaid
sequenceDiagram
    participant ROOT as Root NWDAF
    participant NRF
    participant BRANCH as Branch NWDAF
    participant A as Leaf A
    participant B as Leaf B

    ROOT->>NRF: Nnrf_NFDiscovery_Request<br/>FL_SERVER_AND_CLIENT / FL_CLIENT
    NRF-->>ROOT: Branch and Leaf candidates
    ROOT->>ROOT: Build candidate inventory<br/>and topology plan

    ROOT->>BRANCH: Nnwdaf_MLModelTraining_Subscribe<br/>mLPreFlag=true + model bundle URL
    BRANCH->>NRF: Resolve assigned Leaf NWDAFs
    NRF-->>BRANCH: Current profiles and endpoints

    BRANCH->>A: Nnwdaf_MLModelTraining_Subscribe<br/>mLPreFlag=true
    BRANCH->>B: Nnwdaf_MLModelTraining_Subscribe<br/>mLPreFlag=true

    A-->>BRANCH: Nnwdaf_MLModelTraining_Notify<br/>preparation outcome
    B-->>BRANCH: Nnwdaf_MLModelTraining_Notify<br/>preparation outcome

    BRANCH->>BRANCH: Collect subordinate results
    BRANCH-->>ROOT: Nnwdaf_MLModelTraining_Notify<br/>outcome + result bundle URL
    ROOT->>ROOT: Accept or rebuild topology
~~~

文字流程：

1. Root 透過 NRF 查詢 dual-role Branch candidates 與 Leaf FL Client candidates。
2. Root 建立 candidate inventory，規劃 topology 與 admission policy。
3. Root 透過 preparation request 將 model bundle 與 topology assignment 傳給 Branch。
4. Branch 解析 bundle，再透過 NRF 確認被指派的 Leafs。
5. Branch 建立獨立 lower-tier process，對 Leafs 發起 preparation。
6. Leafs 分別檢查能否參與，並以 Notify 回報 outcome。
7. Branch 彙整實際結果，透過 upper-tier Notify 回報 Root。
8. Root 根據 policy 接受目前 topology，或補選、重新分組與再次 preparation。

完整成功與部分成功都可以成為 policy。無論採哪種 policy，整體 topology 的接受與
重建決策仍由 Root 持有。

規格來源：

- [TS 23.288 §6.2C.2.1](../../../../specs/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2C%20Federated%20Learning%20among%20Multiple%20NWDAFs.md)
- [TS 29.520 §5.5.6 Data Model](../../../../specs/TS%2029.520/5%20API%20Definitions/5.5%20Nnwdaf_MLModelTraining%20Service%20API/5.5.6%20Data%20Model.md)
- [Release 18 OpenAPI NwdafMLModelTrainSubsc](../../../../specs/openapi/TS29520_Nnwdaf_MLModelTraining.yaml)

---

## 7. Hierarchical Training Flow

TS 23.288 §6.2C.2.2 step 2 說明 FL Server 下送訓練要求：

> “FL Server NWDAF sends a Nnwdaf_MLModelTraining_Subscribe ... to the selected
> NWDAF(s) containing MTLF ... to perform the local model training and determine
> the interim local ML Model information ...”

Step 4 說明 FL Client 的訓練與回報：

> “Each FL Client NWDAF further trains the ML Model provided by the FL Server NWDAF
> based on its local data and reports the interim local ML Model information to the
> FL Server NWDAF in Nnwdaf_MLModelTraining_Notify ...”

Step 5 定義 FL Server aggregation：

> “The FL Server NWDAF aggregates all the local ML Model information retrieved at
> step 4, to update the global ML Model.”

Training 不需要為每一輪重新建立 subscription。TS 23.288 §6.2F 說明：

> “When NWDAF service consumer determine to further update the ML Model, NWDAF
> service consumer modifies the subscription by invoking
> Nnwdaf_MLModelTraining_Subscribe ...”

TS 29.520 §4.6.2.2.3 再將既有 subscription 的完整更新映射為：

> “The NF service consumer shall send an HTTP PUT request with ...
> /subscriptions/{subscriptionId} ... to update an Individual NWDAF ML Model
> Training Subscription.”

因此，本設計建立一次 subscription，後續 rounds 沿用同一個
<code>subscriptionId</code>，更新 model information、<code>roundInd</code> 與該輪
training parameters。TS 29.520 也允許 PATCH 進行 partial modification；下圖以更新
既有 subscription 表達，不把 PUT 視為唯一選項。

本設計將同一組標準行為分別套用在 Root–Branch 與 Branch–Leaf process：

~~~mermaid
sequenceDiagram
    participant ROOT as Root NWDAF
    participant BRANCH as Branch NWDAF
    participant A as Leaf A
    participant B as Leaf B

    loop Training rounds
        ROOT->>BRANCH: Nnwdaf_MLModelTraining_Subscribe<br/>update existing subscription

        BRANCH->>A: Nnwdaf_MLModelTraining_Subscribe<br/>update existing subscription
        BRANCH->>B: Nnwdaf_MLModelTraining_Subscribe<br/>update existing subscription

        A->>A: Local training
        B->>B: Local training

        A-->>BRANCH: Nnwdaf_MLModelTraining_Notify<br/>local model info + roundInd + status report
        B-->>BRANCH: Nnwdaf_MLModelTraining_Notify<br/>local model info + roundInd + status report

        BRANCH->>BRANCH: Aggregate Leaf model results
        BRANCH-->>ROOT: Nnwdaf_MLModelTraining_Notify<br/>aggregated model info + roundInd + status report

        ROOT->>ROOT: Aggregate Branch model results
    end
~~~

文字流程：

1. Root 更新既有 upper-tier subscription，將本輪模型、round information 與
   maximum response time 傳給 Branch。
2. Branch 對下作為 FL Server，更新既有 lower-tier subscriptions，將模型傳給
   所屬 Leafs。
3. Leafs 使用 local datasets 訓練並回傳 local model information。
4. Branch 聚合 Leaf results。
5. Branch 在 upper-tier process 中，將 aggregated model 視為自己的 interim local
   ML Model information並回報 Root。
6. Root 聚合各 Branch results，形成下一輪模型並重複流程。

3GPP 定義每一個 FL process 內的 Server／Client training procedure；Branch 將
lower-tier aggregated result 作為 upper-tier Client result，是本設計對兩個標準
processes 的 hierarchical composition。

Upper-tier 與 lower-tier processes 各自維護 correlation 與 round state，圖中的
<code>roundInd</code> 不表示兩層必須使用相同值。

規格來源：

- [TS 23.288 §6.2C.2.2](../../../../specs/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2C%20Federated%20Learning%20among%20Multiple%20NWDAFs.md)
- [TS 23.288 §6.2F](../../../../specs/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2F%20Procedure%20for%20ML%20Model%20Training.md)
- [TS 29.520 §4.6 Nnwdaf_MLModelTraining](../../../../specs/TS%2029.520/4%20Services%20offered%20by%20the%20NWDAF/4.6%20Nnwdaf_MLModelTraining%20Service.md)

---

## 8. Daisy 作為實作參考

Daisy 的 Master–Zone–Client hierarchical control 可用來類比
Root–Branch–Leaf：

| Daisy | 本設計 | 可參考的責任 |
| --- | --- | --- |
| Master | Root NWDAF | 協調上層 rounds，聚合 Branch results |
| Zone | Branch NWDAF | 協調下層 training，形成一份上行 result |
| Client | Leaf NWDAF | 使用 local dataset 執行 training |

Daisy 可提供的 high-level 參考：

- **Strategy abstraction**：將 selection、wait 與 aggregation 從固定流程抽離。
- **Participant selection**：區分 process candidate inventory 與每輪 client selection。
- **Aggregation**：沿用 sample-count-weighted FedAvg，或替換其他 strategy。
- **Waiting policy**：等待全部 clients，或達到 minimum-results／quorum 後聚合。
- **Asynchronous option**：以 FedAsync 與 staleness weighting 處理 late updates。
- **Hierarchical round control**：Branch 先完成下層聚合，再向 Root 回覆一份 result。
- **Per-tier configuration**：upper-tier 與 lower-tier 可選擇不同 policies。

~~~text
Daisy可重用：
FL engine concepts、strategy、selection、wait、aggregation、round control

NWDAF仍負責：
NRF discovery、topology assignment、model bundle contract、standard SBI lifecycle
~~~

不直接搬用 Daisy 的 gRPC networking、private discovery、parent address 或 component
identity。Daisy 的價值是提供內部 FL engine 與 policy 組合方式，而不是取代
NWDAF-to-NWDAF 標準 communication。
