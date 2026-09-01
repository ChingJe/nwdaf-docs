# Hierarchical NWDAF Federated Learning 協定設計

日期：2026-09-01

最後更新：2026-09-02

狀態：核心設計決策已確認；OpenAPI／schema refinement pending

相關文件：

- [Topology、policy 與 strategy 細節設計](./topology_policy_design.md)
- [NWDAF Federated Learning Release 18 規格解讀](../../specification-guides/NWDAF%20Federated%20Learning%20Release%2018%20規格解讀.md)
- [Hierarchical NWDAF Federated Learning Implementation Plan](../../plans/hierarchical-federated-learning/Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)
- [既有 protocol/schema feasibility proposal](../../proposals/nwdaf/hierarchical-federated-learning/protocol_schema_feasibility.md)

---

## 1. 文件定位

本文是 hierarchical NWDAF Federated Learning protocol／schema 設計的主要
文件，負責固定設計目標、標準邊界、核心原則與各議題狀態。各議題的
procedure、資訊模型與候選欄位拆至個別細節文件，避免主要目標被 schema
細節淹沒。

新設計以目前確認過的 3GPP mechanism、現行 implementation facts 與會議後
提出的需求重新建立；既有 proposal 保留為歷史輸入，不直接視為本文件的
既定方案。

目前要解決的核心問題是：

> 如何在不固定 Root／Branch／Leaf NF role 與 topology 深度的前提下，使用
> 多個 `Nnwdaf_MLModelTraining` local FL processes 組成 hierarchical FL，
> 並讓上層能傳遞 topology instruction、下放 decision、取得逐級結果及關聯
> 各層執行狀態。

---

## 2. 設計目標與範圍

### 2.1 主要設計目標

1. 使用 role-neutral recursive structure 表示任意深度的 parent／child
   relationships。
2. 同時支援上層明確指定 participants，以及 Intermediate 依 policy 自行
   選擇 direct children。
3. 分開定義 FL Server 的 selection／participant management policy，以及
   training／aggregation strategy；兩者都需要 minimum structured semantics，
   而不是完全任意的 organization-specific object。
4. 讓 topology instruction、selection result 與 lifecycle status 能逐級
   傳遞及回報。
5. 優先重用既有 3GPP training task、model、deadline、notification 與
   correlation fields，只擴充 hierarchical orchestration 真正缺少的資訊。
6. 區分整個 hierarchical procedure、各層 local FL process、subscription
   resource 與 local round 的 correlation responsibility。

### 2.2 本階段不處理

- Branch failure recovery 的完整 protocol。
- retained result replay、re-parent authorization、fencing 或 ownership
  token。
- runtime self-healing implementation。
- NRF 保存 application-specific topology ownership。
- Visualizer、OAM／MANO implementation 或 dynamic topology optimizer。
- FL algorithm 本身的創新。

上述項目不代表不重要；目前先讓 topology、policy、strategy 與 correlation
semantics 收斂，再決定是否建立其他細節設計文件。

### 2.3 用語

本文不單獨使用 `HFL` 指稱 hierarchical FL。3GPP Release 19 的
TS 23.288 已使用 `HFL` 表示 Horizontal Federated Learning，直接沿用會造成
術語衝突。

`Root`、`Intermediate` 與 `Leaf` 只描述某次 hierarchy 中的相對位置，不是
新的 NF type。任一 Intermediate 對上層是 FL Client，對自己的 direct
children 則是 FL Server。

---

## 3. 已確認的標準基線

### 3.1 Release 18 已提供的能力

TS 23.288 §6.2C.2.1 至 §6.2C.2.3 已提供 flat FL 所需的主要 mechanism：

- FL Server NWDAF 可透過 NRF discovery 尋找並選擇 FL Client NWDAFs。
- discovery／selection 可考慮 Analytics ID、FL capability、service area、
  data-source NF types、time availability 與 model interoperability。
- preparation procedure 讓候選 Client 判斷是否符合 training requirement
  並決定是否參與。
- training execution 支援 deadline、delay notification、skip、等待全部或
  聚合已收到的結果。
- FL Server 可重新選擇、新增或移除 Clients；Client 也可離開。

因此新設計不把「能選 Client」、「能處理 Client 離開」或「能重新選擇
participant」本身當成 hierarchical extension。真正缺少的是這些 mechanisms
如何被組成多層 local FL processes，以及 decision 與執行結果如何跨層傳遞。

### 3.2 既有 Model Training 資訊

Release 18 TS 29.520 的 `NwdafMLModelTrainSubsc` 已能表達：

| 資訊 | 既有欄位 |
| --- | --- |
| Analytics／training task 與 filter | `mLEventSubscs` |
| callback 與 subscription correlation | `notifUri`、`notifCorreId` |
| FL procedure correlation | `mlCorreId` |
| preparation／execution | `mLPreFlag` |
| global／initial model reference | `mLModelInfos` |
| data 與 time availability requirement | `mLModelTrainInfos` |
| response deadline | `mLTrainRepInfo.maxResTime` |
| iteration | `roundInd` |
| training target | `tgtRepUe` |
| skip current round | `skipFlInd` |
| reporting mode／expiry | `eventReq` |

`NwdafMLModelTrainNotif` 已能回傳 `mlCorreId`、`roundInd`、local／interim
model information、model accuracy、training data information、delay 與
termination request。

因此 extension 不應重新建立 `jobDescription`，也不應把標準已能表達的
Analytics、data availability、time availability、model reference 或 deadline
再包進 generic policy object。

### 3.3 Hierarchical orchestration 的缺口

在已查核的 Release 18、Release 19 與 Release 20
`Nnwdaf_MLModelTraining` schema 中，目前未看到下列直接表示方式：

- recursive parent／child FL topology。
- Intermediate 對上為 Client、對下為 Server 的 cross-process binding。
- 上層對下層的 decision delegation 與其作用範圍。
- node-specific policy／strategy，或不同 subtrees 使用不同設定。
- hierarchical procedure 與各 local FL processes 的 correlation model。
- 每個 node／local process 的 progress visibility。

這些項目才是本設計需要處理的 extension boundary。

### 3.4 NRF 的責任邊界

本設計只重用 NRF 已有的 NWDAF registration、capability、availability 與
discovery 資訊，協助 FL Server 找到符合條件的候選 NWDAFs。Hierarchical
topology、policy、parent／child relationship 與 topology ownership 不保存於
NRF，也不由 NRF 負責 orchestration。

NRF 狀態變化可以成為 FL Server 重新評估 participant 的輸入，但如何形成或
修改 hierarchy 仍由 FL Server 及其 policy 決定。NRF schema 或
implementation extension 不在本次設計範圍內。

---

## 4. 核心設計原則

### 4.1 Recursive node 同時承載 instruction、policy、strategy 與 execution setting

Topology 以 role-neutral recursive nodes 表示，不增加固定的
Root／Branch／Leaf enum。每個 node 可以帶有上層提供的 `children`，也可以
帶有該 node 建立 direct-child local FL process 時應採用的 policy 與
strategy，以及 optional、用來指定該 node 完成多少 local work 後回報的
`reportAfter`。

不另外定義 explicit、delegated 或 hybrid mode。上層已列出的 children、
node policy 授權的自行選擇能力，以及實際逐級形成的結果共同決定 topology。
詳細語意見[Topology、policy 與 strategy 細節設計](./topology_policy_design.md)。

### 4.2 Policy 描述 Server-side orchestration

Policy 描述 node 作為 FL Server 時如何選擇與管理 direct children，包括
participant criteria、candidate priority 的使用方式、目標與最低人數、部分
失敗容忍、dropout／replacement behavior，以及 Intermediate 可以自行決定的
範圍。

`allowAdditionalCandidates` 明確表示 node 能否加入上層 `children` 未列出的
候選者；`additionalCandidatePriority` 提供這類候選者的預設 priority；
`selectionMethod` 則只決定如何從 eligible candidate pool 中選擇。這三項
分別處理 selection authority、candidate ordering 與 selection algorithm，
不能以某個 object 是否存在來暗示授權。

第一版 participant policy 參考 Flower 已實際使用的 selection／aggregation
概念，分成三個階段：以 `minAvailableNodes` 判斷可用 direct-child pool 是否
就緒；以 `fractionTrain` 與 `minTrainNodes` 決定每輪實際參與者；最後以
`acceptFailures` 與 `minCompletionRate` 判斷收到部分失敗時是否仍可聚合。
本設計沿用這些概念名稱，但把 `minAvailableNodes` 的可用條件收緊為已成功
建立 Model Training subscription 並加入 topology 的 direct children，而非
僅表示 NRF discovery result 或目前在線的 NWDAF。

Policy 不承載 FedProx、sample-weighted aggregation 等 training／aggregation
設定，避免把「找誰、多少人、失敗怎麼辦」和「找到人後如何訓練與聚合」混為
同一種 semantics。

### 4.3 Strategy 描述 training 與 aggregation

Strategy 保留 training／aggregation responsibility。`method` 表示該 local
FL process 使用的共同方法，例如 `fedProx`；`aggregation` 明確補充結果的
聚合／加權方式，例如 `sampleWeighted`。不再把同一份 FedProx contract 拆成
`localTraining.method: fedProx` 與 `aggregation.method: fedAvg`，避免看起來像
上下游使用兩種不同方法。

Method-specific parameters 放在獨立的 `methodParameters` object，不與共通
欄位混在同一層。第一版只定義目前需求可證明的 `FedProxParameters`，其中
包含 `proximalMu`；不先臆測其他 FL methods 的參數，也不使用
任意 properties。`FedProxParameters` 在 OpenAPI 中必須使用
`additionalProperties: false`：

```yaml
x-flTopology:
  nfInstanceId: branch-a
  strategy:
    method: fedProx
    aggregation: sampleWeighted
    methodParameters:
      proximalMu: 0.01
```

`methodParameters` 必須依 `method` 選用對應的 typed schema。未來確定需要
其他方法時，再新增其 parameter schema，而不是把未知欄位塞進 generic
object。當 `method: fedProx` 時，`methodParameters` 與其中的 `proximalMu`
皆為條件必填；不提供隱含 default。其他 `method` 不得攜帶
`FedProxParameters`。

`policy` 與 `strategy` 直接作為 recursive node 的 sibling fields，不再增加
實際的 `localProcess` wrapper。兩者共同描述該 node 作為 FL Server 時，和
direct children 組成的 local FL process；沒有 direct children 的 node 可以
省略兩者。

Strategy 中的不同資訊仍可能由不同角色消費：selected FL Clients 使用 local
training instructions，FL Server 使用 aggregation instructions。最終 schema
仍須明確定義各欄位的 producer、consumer 與逐級傳遞方式。

當上層在某個 node 指定 `method`、`aggregation` 與 `methodParameters`，該
node 對下建立 Model Training subscriptions 時必須維持相同 contract 並逐級
傳遞。Node-specific `reportAfter` 不隨 strategy 原樣向 descendants 繼承。

第一版先定義目前已知且足夠使用的 semantics；真正未知或
organization-specific 的 model parameters 再透過受控 extension point
處理。只有直接加入既有 3GPP message 的 extension entry 使用 `x-` prefix；
本設計的 subscription entry 為 `x-flTopology`。進入自定義 topology object
後，`children`、`policy`、`strategy`、`reportAfter` 及其內部 properties
不再重複加上 `x-`。

### 4.4 `reportAfter` 描述 node-local 回報週期

`reportAfter` 是 optional node-local execution instruction，不屬於 participant
policy。直接上層提供時，表示明確指定該 node 的回報週期；未提供時，該 node
可依自己的 local capability 或 configuration 決定。它使用明確的 `count` 與
`unit`，不能只依 node 當下看似 Leaf 或 Intermediate 的角色推測語意：

- `unit: epoch`：node 完成指定 local epochs 後，向 parent 回傳一次 model
  update。
- `unit: round`：Intermediate 完成指定次數的 direct-child FL rounds 後，向
  parent 回傳一次 aggregated update。

`count` 必須是 positive integer。若直接上層已提供 `reportAfter`，下層無法
執行時應在 preparation 拒絕參與或回報失敗，不能自行改寫。若直接上層未
提供，Client 或 Intermediate 可以自行決定自己的回報週期。

`reportAfter` 只作用於所在 node。Intermediate 向下建立 subscription 時，若
subtree 已包含各 child 的 `reportAfter`，便依指定值下發；未包含時，
Intermediate 可以作為這些 children 的直接上層自行指定，也可以不指定並讓
各 child 自行決定。Intermediate 不能把自己的值直接複製給 children。

### 4.5 Instruction tree 與 realized topology 必須區分

上層提供的 children 可以是優先候選者，不代表全部都已成為 active
participants。Intermediate 也可依 policy 補充自己發現的候選者，逐一建立
direct-child subscriptions，直到達成 policy 所需條件。

因此 forward instruction 表達候選者與 decision requirement；backward
report 才表達實際嘗試結果與目前形成的 topology。未經確認、正在建立、成功、
失敗及後續退出必須能被區分。

### 4.6 Topology 與 training lifecycle 分離

Topology establishment 完成後，一般 training round 不重新建立 hierarchy。
各 tier 保留既有 Model Training subscription resources，並更新自己的
`roundInd`、`mLModelInfos`、deadline 與其他 round-specific information。

只有 participant membership 或 parent／child relationship 改變時，才更新
topology information。Status report 可以在 preparation 與 training lifecycle
持續更新，不限定於初始建立階段。

### 4.7 Correlation 不等同於 round synchronization

`nfInstanceId`、subscription resource、`notifCorreId`、`mlCorreId` 與
`roundInd` 各自具有不同責任。一個 Intermediate 可以在 lower tier 完成多次
local rounds，才向 upper tier 提交一次 aggregated result，因此上下層
`roundInd` 不必同步。

本設計讓整棵 hierarchy 共用一個 `mlCorreId`，表示同一次 hierarchical
training procedure；各 tier 的 subscription resource 繼續作為對應 local FL
process 的 lifecycle identity。第一版不另外增加 `localProcessId`。

`roundInd` 維持 local FL process scope，不要求上下層同步。每個 node 可在
hierarchical report 中回報自己的 local `roundInd`，由 recursive tree position、
`nfInstanceId` 與 subscription context 判斷其所屬 process。多個 subscriptions
共用同一 `mlCorreId` 是否完全符合既有 3GPP scope 仍須正式規格查核；這是
compatibility validation，不再是 correlation model 的設計選擇。

---

## 5. 現行實作事實

現行 PyMTLF implementation 是驗證需求的參考實作，不直接決定未來
protocol semantics：

- Root 從 static topology file 取得完整 Branch／Leaf identities。
- Root 明確指定每個 Branch 管理哪些 Leaves；目前沒有 delegated
  participant selection。
- Branch 收到 assignment 後，透過 NRF resolve 指定 Leaves，並建立
  lower-tier Model Training subscriptions。
- Branch 同時維護 upper-client 與 lower-server state，完成 lower-tier
  aggregation 後再向 Root 回傳結果。
- participant management policy 固定為選擇全部 participants 並等待全部
  結果。
- training／aggregation strategy 固定使用 FedProx 與 `sample_weighted`
  aggregation。
- upper-tier 與 lower-tier 目前使用不同 process／correlation state，Branch
  顯式保存兩者 mapping。
- topology、assignment 與 preparation result metadata 目前透過 model
  bundle artifact 傳遞，而不是 Model Training message 的正式欄位。

因此現況證明 Root–Intermediate–Leaf composition 可以執行，但尚不能證明
目前 artifact contract 就是適合標準化的 protocol design。

---

## 6. 設計議題狀態

| 設計議題 | 狀態 | 細節位置 |
| --- | --- | --- |
| Recursive topology 與 node-scoped policy | 已確認 | [Topology、policy 與 strategy 細節設計](./topology_policy_design.md) |
| Explicit／delegated selection 共存 | 已確認不使用額外 mode；由 children 與 policy 自然組合 | [Topology、policy 與 strategy 細節設計](./topology_policy_design.md) |
| Candidate expansion、priority、selection 與數量／失敗門檻 | 語意與欄位名稱已確認；OpenAPI mapping 待定 | [Topology、policy 與 strategy 細節設計](./topology_policy_design.md) |
| Training／aggregation strategy | `method`／`aggregation`、typed `methodParameters` 與逐級傳遞語意已確認；OpenAPI mapping 待定 | [Topology、policy 與 strategy 細節設計](./topology_policy_design.md) |
| Node-local `reportAfter` | `epoch`／`round` 語意、parent override／local decision 與 local scope 已確認；OpenAPI mapping 待定 | [Topology、policy 與 strategy 細節設計](./topology_policy_design.md) |
| Topology status 與逐級回報 | Status vocabulary 與 lifecycle 已確認；notification mapping 待定 | [Topology、policy 與 strategy 細節設計](./topology_policy_design.md) |
| `mlCorreId` 與 local process correlation | 共用 `mlCorreId`、subscription-local lifecycle 與 local `roundInd` 已確認；規格相容性待查核 | 本文件 §4.7 |
| Branch replacement 與 retained result | 本階段延後 | 不在目前設計範圍 |

---

## 7. 下一步

1. 依已確認的 topology／policy／strategy 語意形成 candidate OpenAPI schema
   與 HTTP examples。
2. 逐項對照既有 Model Training／NRF fields，確認 standardized、missing 與
   implementation-specific boundary。
3. 查核多個 hierarchical subscriptions 共用 `mlCorreId` 的 3GPP 規格相容性。
4. 視討論成熟度建立 correlation 或其他獨立細節設計文件。

---

## 8. 證據來源

- [TS 23.288 Release 18 §6.2C Federated Learning among Multiple NWDAFs](../../../specs/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2C%20Federated%20Learning%20among%20Multiple%20NWDAFs.md)
- [TS 23.288 Release 18 §6.2F Procedure for ML Model Training](../../../specs/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2F%20Procedure%20for%20ML%20Model%20Training.md)
- [TS 29.520 Release 18 Nnwdaf_MLModelTraining OpenAPI](../../../specs/openapi/TS29520_Nnwdaf_MLModelTraining.yaml)
- [TS 29.520 Release 18 Nnwdaf_MLModelProvision OpenAPI](../../../specs/openapi/TS29520_Nnwdaf_MLModelProvision.yaml)
- 3GPP official `REL-19` and `REL-20` `TS29520_Nnwdaf_MLModelTraining.yaml`
- [Flower `FedAvg` ServerApp strategy（revision `492a31b`）](https://github.com/adap/flower/blob/492a31baf6e6dafbfddc4ad12dcb04ec279ac4be/framework/py/flwr/serverapp/strategy/fedavg.py)
- [Flower legacy `FaultTolerantFedAvg`（revision `492a31b`）](https://github.com/adap/flower/blob/492a31baf6e6dafbfddc4ad12dcb04ec279ac4be/framework/py/flwr/server/strategy/fault_tolerant_fedavg.py)
- Current implementation: `PyMTLF/src/py_mtlf/core/fl_topology.py`,
  `fl_hierarchy.py` and `fl_branch.py`

---

## 9. 變更紀錄

| 日期 | 內容 |
| --- | --- |
| 2026-09-01 | 建立主要設計文件；整理標準基線、現行實作與主要設計空間。 |
| 2026-09-01 | 將 topology／policy procedure 與資訊模型拆至獨立細節文件，主文件保留目標、邊界、核心原則與議題狀態。 |
| 2026-09-01 | 區分 participant management policy 與 training／aggregation strategy 的責任。 |
| 2026-09-01 | 確認 policy 與 strategy 直接放在 recursive node，作用於該 node 的 direct-child local FL process，不增加 `localProcess` wrapper。 |
| 2026-09-01 | 參考 Flower 將 participant policy 收斂為候選池就緒、每輪選取與聚合門檻三個階段。 |
| 2026-09-01 | 確認 recursive topology、hybrid selection、candidate priority、participant policy、strategy responsibility 與 status lifecycle 為設計決策；後續只保留 schema／mapping 細節。 |
| 2026-09-02 | 分開 additional-candidate authority、default priority 與 selection method；確認 `priority`／`random` selection semantics。 |
| 2026-09-02 | Strategy 收斂為共同 `method` 與 `aggregation` contract，並加入上層指定、node-local 的 `reportAfter` execution instruction。 |
| 2026-09-02 | 新增 typed `methodParameters`；`method: fedProx` 時 `methodParameters.proximalMu` 條件必填、不提供隱含 default，並以 `additionalProperties: false` 排除任意 properties。 |
| 2026-09-02 | 確認 `reportAfter` 為 optional；直接上層可明確指定，省略時由接收 node 自行決定。 |
| 2026-09-02 | 確認只有直接加入既有 3GPP message 的 `x-flTopology` extension entry 使用 `x-`；自定義 topology object 的內部 properties 不重複加 prefix。 |
| 2026-09-02 | 確認 hierarchy-wide `mlCorreId`、subscription-local lifecycle 與 local `roundInd` correlation model；保留正式規格相容性查核。 |
