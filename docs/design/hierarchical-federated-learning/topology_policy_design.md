# Hierarchical NWDAF FL Topology、Policy 與 Strategy 細節設計

日期：2026-09-01

狀態：候選設計，供後續討論與 schema refinement 使用

上層文件：

- [Hierarchical NWDAF Federated Learning 協定設計](./protocol_design.md)

---

## 1. 文件目的

本文聚焦 hierarchical NWDAF FL 中 topology instruction、Client selection、
node-level policy、training／aggregation strategy、candidate priority 與逐級
status report 的語意。本文先固定 producer、consumer 與 lifecycle
responsibility。Participant policy 先採用 Flower 中已有對應概念的 candidate
field names；最終欄位掛載位置與 OpenAPI mapping 仍待後續 schema 階段確認。

本文不處理 `mlCorreId` 共用方式、Branch replacement 的完整 recovery
procedure 或 retained result replay。這些議題與 topology 有關，但需要另外
建立完整 directional flow，避免混入本次 selection design。

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
└── status information
```

`children`、node policy 與 node strategy 都是 optional。Policy 與 strategy
直接作為 node 的 sibling fields，不增加實際的 `localProcess` wrapper。

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

目前候選語意是：

> 一個 node 上的 policy，描述該 node 作為 local FL Server 時，如何選擇及
> 管理 direct children。

Policy 不自動套用到所有 descendants。若未來需要 subtree inheritance，必須
另外定義 propagation、override 與 conflict semantics，不能只依 JSON nesting
推測。

Policy 的內容聚焦 participant criteria、candidate priority 的使用方式、
目標／最低人數、部分失敗容忍與 dropout／replacement behavior。實際
priority value 仍屬於各 candidate 的 metadata；FedProx、aggregation
weighting 與 training hyperparameters 不屬於這個 policy。

### 2.4 Strategy 的責任與作用範圍

Strategy 描述 local FL process 的 training 與 aggregation 行為，包括：

- local training method 與必要參數，例如 FedProx。
- aggregation algorithm 與 weighting rule，例如 sample-weighted。
- lower-tier local execution 與 upper-tier update cadence。

Policy 回答「找誰、多少人、失敗怎麼辦」；strategy 回答「找到人後如何訓練
與聚合」。兩者直接放在同一個 recursive node，並維持不同 object 與
semantics；它們共同作用於該 node 作為 FL Server 時，和 direct children
組成的 local FL process。

這裡的 local FL process 是作用範圍，不是額外的 schema wrapper。其中 local
training instructions 必須到達實際執行 training 的 selected FL Clients，
aggregation instructions 則由負責該 local FL process 的 FL Server 消費。
後續 schema 不能只因兩者位於同一 node，就假設由同一 component 執行全部
內容。

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

以下只說明資訊關係，並非最終 wire format；policy property names 是目前採用的
candidate names，topology 與 strategy names 仍是概念表示：

```yaml
node:
  nfInstanceId: branch-a
  children:
    - nfInstanceId: client-a
      priority: 100
    - nfInstanceId: client-b
      priority: 80
    - nfInstanceId: client-c
      priority: 60
    - nfInstanceId: client-d
      priority: 40
    - nfInstanceId: client-e
      priority: 20
  policy:
    minAvailableNodes: 3
    fractionTrain: 0.6
    minTrainNodes: 3
    acceptFailures: true
    minCompletionRate: 0.6
  strategy:
    localTrainingMethod: fedProx
    aggregation: sampleWeighted
```

正式自定義 object 會使用 `x-` prefix；object 內的 policy properties 沿用
Flower 概念名稱，並依 JSON／3GPP naming style 使用 lower camel case。

### 3.3 三階段 participant policy

Participant policy 分成候選池就緒、每輪選取及聚合判定三個階段，不能以
同一個模糊的 participant count 同時表示三者。

#### 3.3.1 候選池就緒：`minAvailableNodes`

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

#### 3.3.2 每輪選取：`fractionTrain` 與 `minTrainNodes`

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

#### 3.3.3 聚合判定：`acceptFailures` 與 `minCompletionRate`

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

### 3.4 Criteria 與既有標準資訊

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
subtree instruction，以及需要傳遞的 node policy／strategy。Intermediate
依序處理上層提供的候選者，必要時透過 NRF discovery 補充自己的 candidate
pool，再對實際選中的 direct children 建立 Model Training subscriptions，
直到滿足 `minAvailableNodes`，並把 Client 需要消費的 strategy instructions
傳到下游。達標後可以先向上回報 realized topology 已可運作，也可以繼續
確認其餘 candidates。

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

目前候選 status 如下：

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

Status report 不限定於 preparation。Topology establishment 完成後，一般
training round 只更新各 local FL process 的 `roundInd`、model reference、
deadline 與結果，不需要重新建立 tree。

若 Active Client 因資源不足、取消 subscription 或其他原因離開，負責該
direct-child relationship 的 FL Server 將其狀態更新為 `INACTIVE`，並逐級
回報。是否嘗試尚未確認的候選者、重新 discovery 或接受較少 participants，
由該 node policy 決定。

---

## 6. 完整 selection 與 training 範例

假設 Root 對 Branch-A 提供五個有 priority 的 candidates，並提供以下 policy：

```yaml
x-policy:
  minAvailableNodes: 3
  fractionTrain: 0.6
  minTrainNodes: 3
  acceptFailures: true
  minCompletionRate: 0.6
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
6. 假設目前有三個 `ACTIVE` Clients，本輪依 `fractionTrain` 與
   `minTrainNodes` 選出三個 Clients 參與 training。
7. 本輪若兩個 Clients 成功回覆、一個失敗，completion rate 為 `2/3`，高於
   `minCompletionRate: 0.6`，因此 Branch-A 可以聚合兩份有效結果。
8. Training 期間若第一順位 Client 取消 subscription，Branch-A 將其更新為
   `INACTIVE`；若 active count 低於 `minAvailableNodes`，則嘗試第五順位或
   其他 discovery candidates 補足後再開始下一輪。

這個流程同時涵蓋上層明確候選、Intermediate 執行 selection、priority
fallback、部分失敗容忍與 training 期間狀態變更，不需要額外的 selection
mode。

---

## 7. 初步資訊模型

以下固定目前的資訊關係與 participant policy candidate names；custom object
的掛載位置，以及 topology、strategy、status 的最終 OpenAPI property names
仍待 schema 階段確認：

```text
ML Model Training subscription
├── existing 3GPP task, model, timing and correlation fields
└── proposed hierarchical instruction
    └── node
        ├── nfInstanceId
        ├── optional candidate priority
        ├── optional direct-child policy
        │   ├── selection criteria
        │   ├── minAvailableNodes
        │   ├── fractionTrain
        │   ├── minTrainNodes
        │   ├── acceptFailures
        │   ├── minCompletionRate
        │   ├── dropout behavior
        │   └── replacement behavior
        ├── optional direct-child strategy
        │   ├── local training method／parameters
        │   ├── aggregation algorithm／weighting
        │   └── local execution／upper update cadence
        └── candidate child nodes[]

Hierarchical topology report
└── node result
    ├── nfInstanceId
    ├── status
    ├── status timestamp
    ├── failure cause（optional）
    └── child results[]
```

Request、PATCH 與 notification 的 directional flow 必須分開驗證。若
topology、policy 或 strategy 可以在既有 resource 上調整，candidate extension
必須同時定義 create representation、PUT／PATCH 與 notification 的適用欄位，
不能只修改 `NwdafMLModelTrainSubsc`。

---

## 8. 尚待確認

- priority 的資料型別、排序方向與同順位處理。
- candidate pool 要回報到什麼範圍，避免把所有 NRF discovery results 都
  當成 topology nodes。
- 各欄位 default value 與 invalid combination 的最終 OpenAPI validation。
- policy 是否需要 inheritance／override；目前只定義 direct-child scope。
- node strategy 中 training 與 aggregation instructions 如何投影至實際
  subscription／notification message，以及各方向是否需要避免重複傳遞。
- topology／strategy／status 的 exact custom property names、`x-` prefix
  placement 與 status enum spelling。
- status report 掛入 subscription notification、獨立 report object 或其他
  既有 message 的最終方式。

---

## 9. 變更紀錄

| 日期 | 內容 |
| --- | --- |
| 2026-09-01 | 從主要設計文件拆出 topology、policy、Client selection、priority、participant threshold 與 status lifecycle 細節。 |
| 2026-09-01 | 將 FL Server participant policy 與 training／aggregation strategy 拆為不同責任，並保留 node-level strategy 作為候選 scope。 |
| 2026-09-01 | 確認 policy 與 strategy 為 recursive node 的 sibling fields，兩者作用於 node 的 direct-child local FL process，不增加 `localProcess` wrapper。 |
| 2026-09-01 | 參考 Flower 將 policy 拆為 `minAvailableNodes`、`fractionTrain`、`minTrainNodes`、`acceptFailures` 與 `minCompletionRate`，分別處理候選池就緒、每輪選取及聚合判定。 |
