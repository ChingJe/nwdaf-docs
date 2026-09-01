# Hierarchical NWDAF FL Topology、Policy 與 Strategy 細節設計

日期：2026-09-01

最後更新：2026-09-02

狀態：核心語意與 participant policy 欄位已確認；OpenAPI／schema refinement pending

上層文件：

- [Hierarchical NWDAF Federated Learning 協定設計](./protocol_design.md)

---

## 1. 文件目的

本文聚焦 hierarchical NWDAF FL 中 topology instruction、Client selection、
node-level policy、training／aggregation strategy、node-local execution
instruction、candidate priority、逐級 status report 與 retained-result lookup
的語意。本文先固定 producer、consumer 與 lifecycle responsibility。
Participant policy 採用 Flower 中已有對應概念的 field names；request 使用
`x-flTopology`，Notify 使用 `x-flTopologyReport`，最終 OpenAPI type、
validation 與 procedure conditional mapping 仍待後續 schema 階段確認。

本文不重新討論 `mlCorreId` 共用方式，也不處理 Branch replacement 的完整
recovery procedure；只定義新 FL Server 重新建立 direct-client subscriptions
時需要的 retained-result request／report behavior。Result retention、接受、
去重、ownership 與 fencing 仍不在本文範圍。

---

## 2. 核心模型

### 2.1 Role-neutral recursive node

Topology 以可遞迴的 node 表示。Node 不帶固定的 Root、Branch 或 Leaf role；
角色由它在當次 hierarchy 中的位置與 local FL process direction 決定。

概念關係如下：

```text
node
├── nfInstanceId
├── children[]
│   └── child node
├── node policy
├── node strategy
├── node reportAfter
├── retainedResultReq
└── status information
```

`children`、node policy、node strategy、node `reportAfter` 與
`retainedResultReq` 都是 optional。Policy、strategy、`reportAfter` 與
`retainedResultReq` 直接作為 node 的 sibling fields，不增加實際的
`localProcess` wrapper。

沒有 children 不代表該 node 永遠是 Leaf；如果 policy 授權它自行選擇
Clients，它仍可發現候選者並建立下一層 local FL process。若 node 沒有
direct children，也沒有建立下一層 process 的需求，則可以省略 policy 與
strategy。

### 2.2 不增加 selection mode

本設計不增加 `EXPLICIT`、`DELEGATED` 或 `HYBRID` mode。三種行為由 node
內容自然形成：

- 上層提供 children 時，這些 nodes 是已知且優先考慮的候選者。
- node policy 允許自行選擇時，接收者可依 criteria 補充其他候選者。
- 同一棵 tree 的不同 nodes 可以採取不同組合。

因此 Root 可以完整指定某一個 subtree，同時只指定另一個 Intermediate，再
由該 Intermediate 依 policy 選擇 direct children。即使同一個 node 已帶有
children，policy 仍可允許接收者補足人數，不需要再切換模式。

### 2.3 Policy 的直接作用範圍

已確認語意是：

> 一個 node 上的 policy，描述該 node 作為 local FL Server 時，如何選擇及
> 管理 direct children。

Policy 不自動套用到所有 descendants。若未來需要 subtree inheritance，必須
另外定義 propagation、override 與 conflict semantics，不能只依 JSON nesting
推測。

Policy 的內容聚焦 participant criteria、candidate priority 的使用方式、
additional candidate authority、selection method、目標／最低人數、部分失敗
容忍與 dropout／replacement behavior。實際 priority value 仍屬於各
candidate 的 metadata；FedProx、aggregation weighting、training
hyperparameters 與 `reportAfter` 不屬於這個 policy。

### 2.4 Strategy 的責任與作用範圍

Strategy 描述 local FL process 的共同 training 與 aggregation contract：

- `method` 表示 FL method，例如 `fedProx`。
- `aggregation` 表示結果的 aggregation／weighting rule，例如
  `sampleWeighted`。
- `methodParameters` 承載由 `method` 決定的 typed parameters。

Policy 回答「找誰、多少人、失敗怎麼辦」；strategy 回答「找到人後如何訓練
與聚合」。兩者直接放在同一個 recursive node，並維持不同 object 與
semantics；它們共同作用於該 node 作為 FL Server 時，和 direct children
組成的 local FL process。

不再把同一份 FedProx contract 拆成 `localTraining.method: fedProx` 與
`aggregation.method: fedAvg`。當上層把 `method: fedProx` 與
`aggregation: sampleWeighted` 指定給某個 node，該 node 向下建立 Model
Training subscriptions 時必須維持相同 contract 並逐級傳遞。

Method-specific parameters 不直接散落在 strategy 共通層，而是放入
`methodParameters`：

```yaml
x-flTopology:
  nfInstanceId: branch-a
  strategy:
    method: fedProx
    aggregation: sampleWeighted
    methodParameters:
      proximalMu: 0.01
```

第一版只定義 `FedProxParameters`，其已知 property 為 `proximalMu`。
`FedProxParameters` 必須使用 `additionalProperties: false`，不能成為任意
object；`methodParameters` 必須依 `method` 使用對應的 typed schema。其他
方法只有在需求確定後才新增自己的 parameter schema。

當 `method: fedProx` 時，`methodParameters` 與其中的 `proximalMu` 都是條件
必填，不提供隱含 default，避免不同 tiers 採用不同的 μ。其他 `method` 不得
攜帶 `FedProxParameters`，而必須使用該方法對應的 typed schema。

這裡的 local FL process 是作用範圍，不是額外的 schema wrapper。其中
selected FL Clients 消費 method 所需的 training instructions，FL Server
執行 aggregation contract。後續 schema 仍須分別驗證各方向的 producer、
consumer 與 transport mapping。

### 2.5 Node-local `reportAfter`

`reportAfter` 是 optional node-local execution instruction，不屬於 policy，
也不跟隨共同 strategy 原樣傳給 descendants。直接上層可以明確指定；若
省略，對應的 Client 或 Intermediate 可依自己的 local capability 或
configuration 決定：

```yaml
x-flTopology:
  nfInstanceId: client-a
  reportAfter:
    count: 5
    unit: epoch
```

- `unit: epoch` 表示 node 完成指定 local epochs 後，向 parent 回傳一次 model
  update。
- `unit: round` 表示 Intermediate 完成指定次數的 direct-child FL rounds 後，
  向 parent 回傳一次 aggregated update。

`count` 必須是 positive integer。上層若為 child node 提供 `reportAfter`，
Intermediate 便使用該值建立下一層 subscription；若省略，Intermediate
可以作為直接上層自行指定，也可以不指定並讓 child 自行決定。Intermediate
自行加入額外候選者時也採相同規則。無論如何，Intermediate 都不能把自己的
`reportAfter` 直接複製給 children。

下層若無法執行直接上層明確指定的值，應在 preparation 拒絕參與或回報失敗，
不能自行更改。只有上層未指定時，Client 或 Intermediate 才能使用自己的
local decision。

---

## 3. Candidate selection

### 3.1 候選者來源

一個 Intermediate 的 candidate pool 可以來自：

- 上層在 children 中明確提供的候選者。
- Intermediate 依標準 training requirement 與 node policy，透過 NRF
  discovery 找到的其他候選者。

NRF 只提供 registration、capability、availability 與 discovery 資訊，不保存
hierarchical topology，也不執行 selection policy。Intermediate 是自己
direct-child local FL process 的 selection owner。

上層不需要揭露自己如何產生候選順序。例如 Root 可以依內部保存的穩定性、
效率或過往合作經驗計算偏好；protocol 只需傳遞結果，不標準化 reputation
model。

### 3.2 上層候選者與 priority

上層提供的 children 預設是優先候選者，而不是保證全部加入的 active
topology。每個 child 可以帶有 optional priority，讓接收者先嘗試偏好較高的
候選者，失敗時再依序遞補。

若沒有 priority，候選者視為同等順位，Intermediate 可依自己的 policy 選擇。
Priority 只決定嘗試順序，不表示該 Client 必然符合 training requirement，
也不取代 preparation 中的接受、拒絕與 availability check。

以下範例使用 `x-flTopology` 作為直接加入既有 3GPP subscription 的 extension
entry。進入這個自定義 object 後，內部 properties 不再重複使用 `x-`：

```yaml
x-flTopology:
  nfInstanceId: branch-a
  children:
    - nfInstanceId: client-a
      priority: 100
      reportAfter:
        count: 5
        unit: epoch
    - nfInstanceId: client-b
      priority: 80
      reportAfter:
        count: 5
        unit: epoch
    - nfInstanceId: client-c
      priority: 60
      reportAfter:
        count: 5
        unit: epoch
    - nfInstanceId: client-d
      priority: 40
      reportAfter:
        count: 5
        unit: epoch
    - nfInstanceId: client-e
      priority: 20
      reportAfter:
        count: 5
        unit: epoch
  policy:
    allowAdditionalCandidates: true
    additionalCandidatePriority: 0
    selectionMethod: priority
    minAvailableNodes: 3
    fractionTrain: 0.6
    minTrainNodes: 3
    acceptFailures: true
    minCompletionRate: 0.6
  strategy:
    method: fedProx
    aggregation: sampleWeighted
    methodParameters:
      proximalMu: 0.01
  reportAfter:
    count: 3
    unit: round
```

只有 `x-flTopology` 這個 3GPP schema extension entry 使用 `x-` prefix。
`children`、`policy`、`strategy`、`reportAfter`、`retainedResultReq` 與更
內層的 properties 都位於自定義 type 的 namespace 內，因此依 JSON／3GPP
naming style 使用 lower camel case，不再逐層加上 `x-`。

### 3.3 Additional candidates 與 selection method

`allowAdditionalCandidates` 與 `selectionMethod` 必須分開：前者是上層授予
node 加入未列於 `children` 的額外候選者之權限，後者只決定如何從 eligible
candidate pool 中選擇。

- `allowAdditionalCandidates: false`：node 只能使用上層列出的 children。
  Node 仍可透過 NRF resolve／確認這些指定 identities；禁止的是加入未指定
  candidate，不是禁止 NRF access。
- `allowAdditionalCandidates: true`：node 可依既有 training requirements 與
  NRF information 發現及加入其他 eligible NWDAFs。
- `additionalCandidatePriority`：所有自行加入的 candidates 使用的預設
  priority。上層明確指定的 child 若帶有自己的 `priority`，仍採該個別值。

`selectionMethod` 第一版具有兩個已知值：

- `priority`：priority 數值越大越優先；同順位使用 random selection。
- `random`：從 eligible candidate pool 均勻隨機選擇。

Criteria 先用來過濾 eligibility，`selectionMethod` 再決定從符合條件的
candidates 中選誰。因此不另外增加 `criteriaBased` method。要讓全部 active
Clients 參與某輪 training，使用 `fractionTrain: 1.0`，不增加 `all` method。

### 3.4 三階段 participant policy

Participant policy 分成候選池就緒、每輪選取及聚合判定三個階段，不能以
同一個模糊的 participant count 同時表示三者。

#### 3.4.1 候選池就緒：`minAvailableNodes`

`minAvailableNodes` 表示一個 node 的 direct-child local FL process 至少需要
多少個可用 Clients，才可視為已具備開始 training 的條件。這裡的 available
不是只被 NRF discovery 找到或目前可連線，而是已成功建立 Model Training
subscription、通過 preparation 並成為 `ACTIVE` 的 direct child。

Intermediate 達到此門檻後即可向上回報目前 realized topology 已可運作；它
也可以繼續確認其餘 candidates，擴大後續每輪可選擇的 pool。此欄位是最低
門檻，不是 active participants 的上限或必須停止建立 subscriptions 的目標值。

Candidate priority 只決定先嘗試誰及失敗後由誰遞補。兩者搭配時，
Intermediate 依 priority 建立 subscriptions，直到至少有
`minAvailableNodes` 個 `ACTIVE` children。

#### 3.4.2 每輪選取：`fractionTrain` 與 `minTrainNodes`

Topology 可用後，每個 local training round 由該 node 從目前 `ACTIVE` 的
direct children 中選出本輪 participants。候選選取數量採用與 Flower
`FedAvg` 相同的基本語意：

```text
selected count = max(floor(active count * fractionTrain), minTrainNodes)
```

`fractionTrain` 表示本輪選取 active pool 的比例；`minTrainNodes` 表示本輪
至少要選取多少 Clients。`minTrainNodes` 是被要求參與 training 的最低數量，
不是成功回覆或允許聚合的最低數量。

第一版要求 `0 < fractionTrain <= 1`，且
`minAvailableNodes >= minTrainNodes >= 1`。若執行時 active count 低於
`minTrainNodes`，該 node 不應開始新一輪 training，並依既有 deadline、
participant update 或後續 policy 處理。

#### 3.4.3 聚合判定：`acceptFailures` 與 `minCompletionRate`

`acceptFailures` 表示本輪被選中的 Clients 發生失敗時，是否仍允許使用成功
結果聚合。當它是 `false`，所有被選中的 Clients 都必須成功，
`minCompletionRate` 不再影響判定。

當 `acceptFailures` 是 `true`，以 `minCompletionRate` 表示允許聚合所需的最低
成功比例：

```text
completion rate = successful replies / selected participants
```

只有 `completion rate >= minCompletionRate` 時才進行聚合。第一版要求
`0 < minCompletionRate <= 1`。它描述的是本輪結果接受條件，不決定候選建立
順序，也不等同於 `minTrainNodes`。

上述欄位名稱與基本拆分參考 Flower，但並非宣稱 Flower 已定義 NWDAF
topology。Flower 的 `minAvailableNodes` 只要求有足夠 nodes 可供抽樣；本設計
還需要完成 Model Training subscription／preparation，才能將 NWDAF 計入
available pool。Flower 現行 `FedAvg` 提供前三個 selection fields；
`acceptFailures` 與 completion-rate threshold 則來自其 legacy
`FedAvg`／`FaultTolerantFedAvg` strategy，作為已實作過的 failure-policy
語意參考。

### 3.5 Criteria 與既有標準資訊

Intermediate 自行選擇時，training task、Analytics／data requirement、time
availability 與 model interoperability 應優先重用
`mLEventSubscs`、`mLModelTrainInfos`、`mLModelInfos` 等既有 Model Training
欄位及 NRF capability information。

Policy 只補充 hierarchical selection 真正缺少的 decision semantics，例如
需要補足的人數、候選排序與是否允許部分失敗。Resource criteria 的權威來源與
可觀測方式尚待確認，不先假設 Intermediate 自然擁有相關資訊。

---

## 4. Instruction tree 與 realized topology

### 4.1 Forward instruction

Root 對 direct Intermediate 傳送既有 training task context、recursive
subtree instruction，以及需要傳遞的 node policy／strategy／`reportAfter`／
`retainedResultReq`。Intermediate 依序處理上層提供的候選者，必要時透過 NRF
discovery 補充自己的 candidate pool，再對實際選中的 direct children 建立
Model Training subscriptions，直到滿足 `minAvailableNodes`，並把 Client
需要消費的 strategy instructions、已指定的 child `reportAfter`，以及需要
映射到下游 subscription 的 child `retainedResultReq` 送到下游。若 child
沒有指定 `reportAfter`，Intermediate 可自行補上或讓 child 自行決定。只有
`allowAdditionalCandidates: true` 時才能加入 children 未列出的候選者。達標
後可以先向上回報 realized topology 已可運作，也可以繼續確認其餘
candidates。

Request 中出現的 children 不表示 subscriptions 已經全部建立，也不表示它們
已經接受參與。因此 forward tree 是 candidate／instruction view；目前真正
形成的 topology 必須依 backward report 判斷。

### 4.2 Backward confirmation

每個 Intermediate 保存自己 direct children 的最新 selection 與 subscription
狀態，將已考慮的 candidate pool 及其狀態組成 subtree report，再逐級向上
彙整。Root 最後取得的是目前已知的 realized topology 與尚未確認的候選資訊，
而不是假設 forward instruction 已完整實現。

一段 direct-child relationship 的 status 由實際建立及維護該 subscription
的 parent FL Server 記錄。逐級回報必須保留各層 status owner 提供的 status
與 status time；Intermediate 可以包裝 child reports，但不能把 descendant
status 的發生時間改成自己向上通知的時間。

Notify 以 `x-flTopologyReport` 承載 recursive report。除了 status result，
report node 可重用 subscription node 已定義的 `policy`、`strategy` 與
`reportAfter` 欄位，表示該 node 實際採用的 contract。兩個方向使用相同欄位
名稱及 component schemas，不另外建立 `effective*` 欄位：subscription 中是
上層下發的 instruction，Notify 中則是 resolved／applied result。

若上層已指定值，回報相同值表示 node 已採用該 contract；若上層省略並允許
node 自行決定，回報值則揭露 local decision。例如上層沒有指定 Client 每次
回報前執行多少 local epochs，Client 可以在 preparation result 中回報：

```json
{
  "nfInstanceId": "client-a",
  "status": "ACTIVE",
  "statusTimestamp": "2026-09-02T14:30:25+08:00",
  "reportAfter": {
    "count": 5,
    "unit": "epoch"
  }
}
```

這只是 `reportAfter` resolved value 的使用範例；同一原則也適用於 node
實際採用的 `policy` 與 `strategy`。上層可依回報結果接受目前設定、透過既有
subscription update／PATCH 再調整，或終止該 subscription。

`x-flTopologyReport` 不用來同步 descendants 的 local round。通知者仍以既有
頂層 `NwdafMLModelTrainNotif.roundInd` 表示自己和 parent 之間 local FL
process 的 round；Intermediate 內部維護 lower-tier progress 與 upper-tier
update 的關係。

### 4.3 Partial／unbalanced topology

假設 Root 要求 Branch-A 管理 Client-A1、Client-A2，Branch-B 管理
Client-B1、Client-B2，但 Client-B2 拒絕參與或 timeout。Branch-B 回報自己
只建立 Client-B1 後，Root 可以依 policy 接受 unbalanced topology、要求
Branch-B 補足替代者、移除失敗部分，或終止 establishment。

本範例只說明 backward confirmation 必須提供足夠資訊讓 decision owner
執行 policy，不表示第一版必須支援所有 correction action。

---

## 5. Status lifecycle

### 5.1 Candidate status vocabulary

已確認的 status vocabulary 如下：

| Status | 語意 |
| --- | --- |
| `UNCONFIRMED` | 已知或已列入候選，但目前尚未取得其參與確認。 |
| `DEPLOYING` | 已開始建立 subscription／執行 preparation，尚未取得最終結果。 |
| `ACTIVE` | 已成功加入目前 topology。 |
| `FAILED` | 已嘗試加入，但拒絕、timeout 或建立失敗。 |
| `INACTIVE` | 原本已加入，後續退出、取消 subscription 或被移除。 |

`UNCONFIRMED` 比 `NOT_ATTEMPTED` 更適合 recursive report。上層可能沒有直接
接觸 descendant，但不能據此宣稱所有下層都未嘗試；它只能表示目前尚未取得
確認結果。一旦 Intermediate 回報 child 已成功或失敗，Root 就應採用該狀態，
不能因自己沒有直接接觸該 Client 而繼續標為 `UNCONFIRMED`。

### 5.2 `DEPLOYING` 的使用時點

`DEPLOYING` 是執行狀態，不是 Root 預先下發給整棵 tree 的命令。只有實際
開始對某個 direct child 建立 subscription／執行 preparation 的 node，才能
把該 child 更新為 `DEPLOYING`。

Forward instruction 可以不帶 status。若共用的表示形式要求填寫尚未確認的
descendants，應使用 `UNCONFIRMED`；不能因 Root 即將把 instruction 交給
Intermediate，就把所有更下層 Clients 預先標為 `DEPLOYING`。

典型狀態轉換如下：

```text
UNCONFIRMED -> DEPLOYING -> ACTIVE
                         -> FAILED
ACTIVE -> INACTIVE
```

實作不一定要回報每個短暫的 `DEPLOYING` transition，但只要回報該狀態，就
必須符合上述語意。

### 5.3 Status timestamp

每個 node status 應帶有 status timestamp，表示負責該 direct-child
relationship 的 parent FL Server 確認或最後更新 status 的時間，而不是 Root
收到 notification 的時間。逐級轉送時保留 status owner 產生的值，不由更
上層 Intermediate 重寫。

概念範例如下：

```json
{
  "nfInstanceId": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
  "status": "ACTIVE",
  "statusTimestamp": "2026-09-01T14:30:25+08:00"
}
```

正式 schema 可重用 TS 29.571 CommonData 的 `DateTime` 型別。實際 property
name 仍待 schema 階段決定。

### 5.4 Training lifecycle 中的狀態更新

`x-flTopologyReport` 不限定於 preparation。Topology establishment 完成後，
一般 training round 只更新各 local FL process 的 `roundInd`、model reference、
deadline 與結果，不需要重新建立 tree。

若 Active Client 因資源不足、取消 subscription 或其他原因離開，負責該
direct-child relationship 的 FL Server 將其狀態更新為 `INACTIVE`，並逐級
回報。是否嘗試尚未確認的候選者、重新 discovery 或接受較少 participants，
由該 node policy 決定。

---

## 6. Retained-result lookup

### 6.1 Request 與 correlation

接收 node 不能只依新的 subscription 或相同 `mlCorreId` 推測自己是替代
Intermediate，也不能自行推測是否應向 direct children 取回既有結果。上層
若要求這項動作，必須在 subtree instruction 的對應 child nodes 明確加入
optional `retainedResultReq: true`：

```json
{
  "x-flTopology": {
    "nfInstanceId": "branch-a2",
    "children": [
      {
        "nfInstanceId": "leaf-a1",
        "retainedResultReq": true
      },
      {
        "nfInstanceId": "leaf-a2",
        "retainedResultReq": true
      }
    ]
  }
}
```

`retainedResultReq` 是 parent 建立對該 child 的 subscription 時所消費的
edge-level instruction。Parent 對該 child 建立 `NwdafMLModelTrainSubsc`
時，將它映射成 message-level optional `x-retainedResultReq: true`，要求
接收者查找並回報本地針對同一 `mlCorreId` 保留的最新已完成結果。位於
自定義 topology node 內的 property 不重複加 `x-`；直接加入既有 3GPP
message 的 entry 才使用 `x-retainedResultReq`。

這項 instruction 不自動向 descendants 繼承。若上層希望一個 Intermediate
向多個 direct children 查詢，就在各 child node 分別指定；省略時，接收
Intermediate 不得只因自己是新選出的 participant 而自行啟動 lookup。因為本
設計讓整棵 hierarchy 共用 `mlCorreId`，extension 不重複攜帶另一個 process
ID。

Retained-result lookup 與新一輪 training 是兩個不同動作。
`x-retainedResultReq: true` 時，本次 request 只執行查找，不開始 local
training。Server 收到查詢結果後，如需繼續訓練，必須另外更新 subscription，
清除 `x-retainedResultReq` 或設為 `false`，再以正常的 model／round
instruction 啟動 training。

找不到 retained result 不使 subscription 建立失敗。新 Branch 仍需要透過
成功建立的 subscription 維持後續 notification 與 training lifecycle，不能
因為 Client 沒有舊結果而拒絕建立關係。

### 6.2 Result status

回報方向在 `NwdafMLModelTrainNotif` 增加 optional
`x-retainedResultStatus`，只定義兩個 lookup outcomes：

- `FOUND`：已找到最新完成結果；同一 report 使用既有 `roundInd` 與
  `mLModelInfos` 回傳該結果。
- `NOT_FOUND`：已完成查找，但本地沒有同一 `mlCorreId` 的已完成保留結果；
  不建立空的 model payload。

當 `x-retainedResultReq: true` 時，接收者必須透過 immediate report 或後續
Notify 明確回報其中一個 outcome。Server 不能只依 `mLModelInfos` 缺席或
callback timeout 推測結果不存在。`NOT_FOUND` 不是 subscription 或 API
resource failure，因此不使用 HTTP `404`、`ProblemDetails`、
`failEventReports` 或 `termTrainReq`。

`x-retainedResultReq` 不要求 `eventReq.immRep` 必須為 `true`。若兩者一起
使用且 lookup 已完成，接收者可以在 subscription response 的 `immReport`
回傳；否則在 subscription 建立後透過既有 Notify procedure 回報。

### 6.3 HTTP examples

以下 request 要求 Leaf 查找同一 hierarchical FL procedure 的最新保留
結果，並選擇搭配 immediate reporting：

```http
POST /nnwdaf-mlmodeltraining/v1/subscriptions HTTP/1.1
Host: leaf-a.example.org
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
  "notifUri": "https://branch-b.example.org/nnwdaf-mlmodeltraining/v1/notifications",
  "notifCorreId": "branch-b-leaf-a-subscription",
  "mlCorreId": "hierarchical-fl-001",
  "eventReq": {
    "notifMethod": "ON_EVENT_DETECTION",
    "immRep": true
  },
  "x-retainedResultReq": true
}
```

若 Leaf 保留的最新已完成 local result 為 round 5，POST response 使用既有
`immReport`：

```http
HTTP/1.1 201 Created
Location: https://leaf-a.example.org/nnwdaf-mlmodeltraining/v1/subscriptions/sub-002
Content-Type: application/json

{
  "mLEventSubscs": [
    {
      "mLEvent": "UE_COMMUNICATION",
      "mLEventFilter": {},
      "modelInterInfo": "pymtlf-model-bundle-v1"
    }
  ],
  "notifUri": "https://branch-b.example.org/nnwdaf-mlmodeltraining/v1/notifications",
  "notifCorreId": "branch-b-leaf-a-subscription",
  "mlCorreId": "hierarchical-fl-001",
  "eventReq": {
    "notifMethod": "ON_EVENT_DETECTION",
    "immRep": true
  },
  "x-retainedResultReq": true,
  "immReport": {
    "notifCorreId": "branch-b-leaf-a-subscription",
    "mlCorreId": "hierarchical-fl-001",
    "x-retainedResultStatus": "FOUND",
    "roundInd": 5,
    "mLModelInfos": [
      {
        "event": "UE_COMMUNICATION",
        "mLFileAddr": {
          "mLModelUrl": "https://leaf-a.example.org/results/hierarchical-fl-001/round-5"
        }
      }
    ]
  }
}
```

若未要求 immediate report，且 Leaf 查找後沒有保留結果，則以 Notify 明確
回報 `NOT_FOUND`：

```http
POST /nnwdaf-mlmodeltraining/v1/notifications HTTP/1.1
Host: branch-b.example.org
Content-Type: application/json

{
  "notifCorreId": "branch-b-leaf-a-subscription",
  "mlCorreId": "hierarchical-fl-001",
  "x-retainedResultStatus": "NOT_FOUND"
}
```

`roundInd` 表示該 Client local process 的最新已完成 round，不代表 Root 或
其他 tier 的 round。Result retention、cleanup、接受、去重與 aggregation
handling 均由 implementation／strategy 決定，不由 request boolean 或
outcome enum 定義。

---

## 7. 完整 selection 與 training 範例

假設 Root 對 Branch-A 提供五個有 priority 的 candidates，並提供以下 policy：

```yaml
x-flTopology:
  nfInstanceId: branch-a
  children:
    - nfInstanceId: client-a
      priority: 100
      reportAfter:
        count: 5
        unit: epoch
    - nfInstanceId: client-b
      priority: 80
      reportAfter:
        count: 5
        unit: epoch
    - nfInstanceId: client-c
      priority: 60
      reportAfter:
        count: 5
        unit: epoch
    - nfInstanceId: client-d
      priority: 40
      reportAfter:
        count: 5
        unit: epoch
    - nfInstanceId: client-e
      priority: 20
      reportAfter:
        count: 5
        unit: epoch
  policy:
    allowAdditionalCandidates: true
    additionalCandidatePriority: 0
    selectionMethod: priority
    minAvailableNodes: 3
    fractionTrain: 0.6
    minTrainNodes: 3
    acceptFailures: true
    minCompletionRate: 0.6
  strategy:
    method: fedProx
    aggregation: sampleWeighted
    methodParameters:
      proximalMu: 0.01
  reportAfter:
    count: 3
    unit: round
```

1. Root 傳送 Branch-A 的 subtree instruction；五個 candidates 尚未取得
   結果，因此為 `UNCONFIRMED`。
2. Branch-A 先嘗試前三順位，將它們更新為 `DEPLOYING`。
3. 第一與第三順位成功，成為 `ACTIVE`；第二順位 timeout，成為 `FAILED`。
4. Branch-A 嘗試第四順位。第四順位成功後，已達三個 active participants，
   滿足 `minAvailableNodes`；第五順位可以維持 `UNCONFIRMED`，Branch-A 也可
   繼續確認它以擴大 available pool。
5. Branch-A 將五個 candidates 的最新 status 與各自 timestamp 組成 subtree
   report，回傳 Root，表示目前已有足夠 Clients 可開始 training。
6. Branch-A 向被選中的 Leaves 下發相同的 `method: fedProx`、
   `aggregation: sampleWeighted` 與 `proximalMu: 0.01` contract，並依各 Leaf
   node 的 `reportAfter` 指定 local epochs；Branch-A 自己則依 `unit: round`
   完成三個 lower-tier rounds 後才向 Root 回報一次 aggregated update。
7. 假設目前有三個 `ACTIVE` Clients，本輪依 `fractionTrain` 與
   `minTrainNodes` 選出三個 Clients 參與 training。
8. 本輪若兩個 Clients 成功回覆、一個失敗，completion rate 為 `2/3`，高於
   `minCompletionRate: 0.6`，因此 Branch-A 可以聚合兩份有效結果。
9. Training 期間若第一順位 Client 取消 subscription，Branch-A 將其更新為
   `INACTIVE`；若 active count 低於 `minAvailableNodes`，則嘗試第五順位或
   其他 discovery candidates 補足後再開始下一輪。自行 discovery 的候選者
   使用 `additionalCandidatePriority: 0`。

這個流程同時涵蓋上層明確候選、Intermediate 執行 selection、priority
fallback、部分失敗容忍與 training 期間狀態變更，不需要額外的 selection
mode。

---

## 8. 初步資訊模型

以下固定 request-side `x-flTopology`、Notify-side `x-flTopologyReport`、資訊
關係與 participant policy field names；正式 OpenAPI type、required condition
與 procedure mapping 留到 schema 階段完成：

```text
ML Model Training subscription
├── existing 3GPP task, model, timing and correlation fields
└── x-flTopology
    └── topology node
        ├── nfInstanceId
        ├── optional candidate priority
        ├── optional direct-child policy
        │   ├── selection criteria
        │   ├── allowAdditionalCandidates
        │   ├── additionalCandidatePriority
        │   ├── selectionMethod
        │   ├── minAvailableNodes
        │   ├── fractionTrain
        │   ├── minTrainNodes
        │   ├── acceptFailures
        │   ├── minCompletionRate
        │   ├── dropout behavior
        │   └── replacement behavior
        ├── optional direct-child strategy
        │   ├── method
        │   ├── aggregation
        │   └── conditionally required typed methodParameters
        │       └── FedProxParameters
        │           └── proximalMu
        ├── optional node-local reportAfter
        │   ├── count
        │   └── unit: epoch／round
        ├── optional retainedResultReq
        │   └── parent 建立對此 node 的 subscription 時要求查找保留結果
        └── candidate child nodes[]

Hierarchical topology report
└── node result
    ├── nfInstanceId
    ├── status
    ├── status timestamp
    ├── failure cause（optional）
    ├── policy（optional，實際採用值，重用 FlPolicy）
    ├── strategy（optional，實際採用值，重用 FlStrategy）
    ├── reportAfter（optional，實際採用值，重用 FlReportAfter）
    └── child results[]

Retained-result lookup
├── x-flTopology child node.retainedResultReq
│   └── parent maps to NwdafMLModelTrainSubsc.x-retainedResultReq
└── NwdafMLModelTrainNotif.x-retainedResultStatus
    ├── FOUND -> roundInd + mLModelInfos
    └── NOT_FOUND
```

「實際採用」是欄位在 Notify direction 的語意，不是 property name 的
prefix。Request node 與 report node 可分別定義各自特有欄位，但共同引用
同一組 `FlPolicy`、`FlStrategy` 與 `FlReportAfter` component schemas。
Request node 可帶 candidate `priority` 與 `retainedResultReq`；report node 則帶
`status`、`statusTimestamp` 與 optional `cause`。

Request、PATCH 與 notification 的 directional flow 必須分開驗證。若
topology、policy 或 strategy 可以在既有 resource 上調整，candidate extension
必須同時定義 create representation、PUT／PATCH 與 notification 的適用欄位，
不能只修改 `NwdafMLModelTrainSubsc`。

---

## 9. 後續 schema refinement 與查核

以下項目不重新打開前述設計語意，只在形成正式 OpenAPI 時完成：

- 設定 priority、`additionalCandidatePriority`、`proximalMu`、`reportAfter`
  等 properties 的資料型別、數值範圍與 invalid-combination validation。
- 以 OpenAPI 表達 `method` 與 typed `methodParameters` 的條件 binding。
- 將 topology、strategy、`reportAfter` 與 status report 分別映射到 create、
  update／PATCH 與 notification direction；直接加入既有 3GPP message 的
  entry 一律使用 `x-` prefix。Notify entry 使用 `x-flTopologyReport`，其
  `policy`、`strategy` 與 `reportAfter` 重用 request-side component schemas。
- 確認只帶 `x-flTopologyReport` 的 Notify 如何滿足 TS 29.520 既有
  detailed-information 條件；在 procedure 正式擴充前，不假設 vendor field
  可取代既有 `mLModelInfos`、`delayEventNotif` 或 `termTrainReq`。
- 將 `x-retainedResultReq` 映射至 subscription create／update，並將
  `x-retainedResultStatus` 納入 immediate report／Notify 的合法 detailed
  information；`FOUND` 時要求 `roundInd` 與 `mLModelInfos`，`NOT_FOUND` 時
  不要求 model payload。
- 將 topology node 的 `retainedResultReq` 定義為 boolean，並驗證 parent
  只把它映射至該 node 對應的 direct-client subscription，不將其視為 subtree
  inheritance rule。
- 在 notification schema 階段界定 candidate pool 的回報範圍，避免把所有
  NRF discovery results 都當成 topology nodes。
- 查核無法接受 `reportAfter` 時可重用的 preparation failure 表達方式。

---

## 10. 變更紀錄

| 日期 | 內容 |
| --- | --- |
| 2026-09-01 | 從主要設計文件拆出 topology、policy、Client selection、priority、participant threshold 與 status lifecycle 細節。 |
| 2026-09-01 | 將 FL Server participant policy 與 training／aggregation strategy 拆為不同責任，並保留 node-level strategy 作為候選 scope。 |
| 2026-09-01 | 確認 policy 與 strategy 為 recursive node 的 sibling fields，兩者作用於 node 的 direct-child local FL process，不增加 `localProcess` wrapper。 |
| 2026-09-01 | 參考 Flower 將 policy 拆為 `minAvailableNodes`、`fractionTrain`、`minTrainNodes`、`acceptFailures` 與 `minCompletionRate`，分別處理候選池就緒、每輪選取及聚合判定。 |
| 2026-09-01 | 確認本文的 topology、selection、priority、participant policy、strategy responsibility 與 status lifecycle 語意；後續只保留 schema／mapping 細節。 |
| 2026-09-02 | 確認 `allowAdditionalCandidates`、`additionalCandidatePriority` 與 `selectionMethod` 分別表示額外候選授權、預設順位與 selection algorithm；第一版 method 為 `priority`／`random`。 |
| 2026-09-02 | Strategy 收斂為逐級傳遞的共同 `method`／`aggregation` contract；新增 node-local 且不向 descendants 原樣繼承的 `reportAfter`。 |
| 2026-09-02 | 新增 typed `methodParameters`；`method: fedProx` 時 `methodParameters.proximalMu` 條件必填、不提供隱含 default，並以 `additionalProperties: false` 排除任意 properties。 |
| 2026-09-02 | 確認 `reportAfter` 為 optional；直接上層可明確指定，省略時由接收 node 自行決定。 |
| 2026-09-02 | 確認只有直接加入既有 3GPP message 的 `x-flTopology` extension entry 使用 `x-`；自定義 topology object 的內部 properties 不重複加 prefix。 |
| 2026-09-02 | 確認 Notify 使用 `x-flTopologyReport`；report node 重用同名 `policy`、`strategy` 與 `reportAfter` 表示實際採用值，不增加 `effective*` 欄位，且不跨層回報 descendants 的 `roundInd`。 |
| 2026-09-02 | 加入 retained-result lookup：`x-retainedResultReq` 只觸發查詢，`x-retainedResultStatus` 明確回報 `FOUND`／`NOT_FOUND`，並可使用 immediate report 或後續 Notify；完整 Branch recovery 維持不在本文範圍。 |
| 2026-09-02 | 新增 topology node `retainedResultReq` instruction；上層以 child node 明確要求 parent 在對該 child 建立 subscription 時加入 `x-retainedResultReq`，不以新 subscription 或 `mlCorreId` 暗示 replacement behavior。 |
