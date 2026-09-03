# Hierarchical NWDAF Federated Learning 協定設計

日期：2026-09-01

最後更新：2026-09-02

狀態：核心設計決策已確認；candidate OpenAPI schema／artifact 待使用者審查

相關文件：

- [Topology、policy 與 strategy 細節設計](./topology_policy_design.md)
- [標準欄位與 Extension 邊界](./standard_field_extension_boundary.md)
- [Candidate OpenAPI Schema](./candidate_openapi_schema.md)
- [Candidate OpenAPI artifact](./candidate_openapi.yaml)
- [Protocol Conformance Matrix](./protocol_conformance_matrix.md)
- [NWDAF Federated Learning Release 18 規格解讀](../../specification-guides/NWDAF%20Federated%20Learning%20Release%2018%20規格解讀.md)
- [Hierarchical NWDAF Federated Learning Implementation Plan](../../plans/hierarchical-federated-learning/hierarchical-fl-model-bundle-edition/Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)
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
- retained result 的保存期限、接受／去重、re-parent authorization、fencing
  或 ownership token；本階段只定義要求回報最新保留結果的 trigger。
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

每段 subscription 傳遞的 subtree root 必須是實際接收者，回報 wrapper 也必須
綁定該 subscription 的 direct Client identity。單一 subtree 不允許重複
`nfInstanceId` 或 ancestor cycle；payload identity 不取代既有 subscription／
callback context 的 peer binding。

不另外定義 explicit、delegated 或 hybrid mode。上層已列出的 children、
node policy 授權的自行選擇能力，以及實際逐級形成的結果共同決定 topology。
詳細語意見 [Topology、policy 與 strategy 細節設計](./topology_policy_design.md)。

### 4.2 Policy 描述 Server-side orchestration

Policy 描述 node 作為 FL Server 時如何選擇與管理 direct children，包括
candidate ordering、selection authority、可用與每輪參與門檻，以及部分失敗
容忍。Participant eligibility 優先使用既有 training requirements 與 NRF
information；policy 則界定 Intermediate 可以自行決定的範圍。

Policy 不承載 FedProx、sample-weighted aggregation 等 training／aggregation
設定，避免把「找誰、多少人、失敗怎麼辦」和「找到人後如何訓練與聚合」混為
同一種 semantics。欄位、Flower 對應與逐步 selection behavior 見
[Topology、policy 與 strategy 細節設計](./topology_policy_design.md)。

### 4.3 Strategy 描述 training 與 aggregation

Strategy 保留 local FL process 的 training／aggregation responsibility，涵蓋
共同 FL method、aggregation／weighting rule 與 method-specific parameters。
它與 participant policy 分離，但和 policy 一樣作用於 node 作為 FL Server 時
所建立的 direct-child local FL process。

`policy` 與 `strategy` 直接作為 recursive node 的 sibling fields，不再增加
實際的 `localProcess` wrapper。完整 method contract、typed parameters、
producer／consumer 與逐級傳遞規則見
[Topology、policy 與 strategy 細節設計](./topology_policy_design.md)。

### 4.4 `reportAfter` 描述 node-local 回報週期

`reportAfter` 是 optional node-local execution instruction，不屬於 participant
policy。直接上層可以指定，省略時由 node 依 local capability 或 configuration
決定；它不自動向 descendants 繼承。`epoch`／`round` 語意與 override 規則見
[Topology、policy 與 strategy 細節設計](./topology_policy_design.md)。

### 4.5 Instruction tree 與 realized topology 必須區分

上層提供的 children 可以是優先候選者，不代表全部都已成為 active
participants。Intermediate 也可依 policy 補充自己發現的候選者，逐一建立
direct-child subscriptions，直到達成 policy 所需條件。

因此 forward instruction 表達候選者與 decision requirement；backward
report 才表達實際嘗試結果與目前形成的 topology。未經確認、正在建立、成功、
失敗及後續退出必須能被區分。Request 使用 `x-flTopology`，Notify 使用
`x-flTopologyReport`；完整 status vocabulary、resolved contract report 與
directional mapping 見
[Topology、policy 與 strategy 細節設計](./topology_policy_design.md)。

上層若要明確剔除某個 child，使用該 child node 的 `enabled: false`；降低
priority 或從 upstream-assigned `children` 省略該 identity 都不等同於禁止
Intermediate 透過 local discovery 再次選用它。

Candidate enums 採 3GPP 常見的 forward-compatible enumeration pattern，讓
舊版 schema 可以解析未來新增的 vocabulary。未知 report-side value 不得被
推測成既有狀態；未知 instruction-side value 若接收者無法履行，仍應拒絕整個
operation。由於 `strategy.method` 會決定 typed parameters，它維持 closed
discriminator，新增 method 時一併增加對應 subtype。

### 4.6 Topology 與 training lifecycle 分離

Topology establishment 完成後，一般 training round 不重新建立 hierarchy。
各 tier 保留既有 Model Training subscription resources，並更新自己的
`roundInd`、`mLModelInfos`、deadline 與其他 round-specific information。

一般 round-specific update 不需要重送 `x-flTopology`。只有 participant
membership、parent／child relationship、policy、strategy 或 node-local
instruction 等 orchestration contract 改變時，才更新 topology extension。
`x-flTopologyReport` 可以在 preparation 與 training lifecycle 持續更新，不
限定於初始建立階段。

### 4.7 Correlation 不等同於 round synchronization

`nfInstanceId`、subscription resource、`notifCorreId`、`mlCorreId` 與
`roundInd` 各自具有不同責任。本設計以 hierarchy-wide `mlCorreId` 關聯同一
hierarchical training procedure，各 local subscriptions 維持自己的 lifecycle
identity 與 local `roundInd`；上下層 rounds 不要求同步。Release 18 至
Release 20 的正式欄位與相容性查核見
[標準欄位與 Extension 邊界](./standard_field_extension_boundary.md)。

### 4.8 Retained result retrieval 使用明確的 subscription trigger

Retained-result lookup 與新一輪 training 是不同動作。新 FL Server 可以在
建立 direct-client subscription 時，以 `x-retainedResultReq` 要求查找同一
`mlCorreId` 的最新已完成 local result；Client 以
`x-retainedResultStatus` 明確回報 `FOUND`、`NOT_FOUND` 或 `FAILED`。結果可
搭配既有 immediate report 或後續 Notify 傳遞；找不到結果或已接受的 lookup
後續執行失敗，都不改變 subscription 已成功建立的事實。這是
operation-scoped 的一次性 request；當次 Create、PUT 或
PATCH 帶 `true` 時只查詢一次，不會成為持續的 subscription state。
每個 local subscription 同時最多只能有一個 outstanding lookup；Server
收到本次 lookup outcome 後才能再次要求，因此不新增 request ID。等待 outcome
timeout 時不得在同一 subscription 另起新的 outstanding lookup。本版本使用
`FOUND`／`NOT_FOUND`／`FAILED`；未知的 forward-compatible outcome 只結束
該次 lookup，不視為可用結果。
完整 procedure、回報內容與 HTTP examples 見
[Topology、policy 與 strategy 細節設計](./topology_policy_design.md)。

### 4.9 Extension capability 沿每個 subscription edge 協商

本設計重用 `NwdafMLModelTrainSubsc.suppFeats`，不新增 capability 欄位。
整組 hierarchical orchestration semantics 使用一個 candidate optional
feature `HierarchicalFLOrch`。為避開 Release 19 feature 1 與 Release 20
feature 2，候選 feature number 為 3，單獨宣告時的 bitmask 值為 `"4"`。

Feature negotiation 以 local subscription resource 為 scope。Root 與
Intermediate 協商成功，不代表 Intermediate 與 Leaves 已協商成功；每個
Intermediate 建立 direct-child subscription 時都要重新協商並檢查 response。
若 initial Create response 未回傳 feature 3，consumer 不得把該 resource
當作 hierarchical resource，也不得在後續 operation 套用 extension procedure。
若 hierarchy 是本次 subscription 的必要條件，consumer 刪除該 resource，並將
該 candidate 回報為 `FAILED`／`FEATURE_NOT_SUPPORTED`。

接收者不得靜默改寫已協商後的 instruction。Schema 或 validation 錯誤使用
既有 `400 Bad Request`；message 合法但不能履行 node-wide `policy`、
`strategy` 或 `reportAfter` contract 時，使用
`403 ML_MODEL_TRAINING_REQS_NOT_MET`。完整 mapping 見
[Candidate OpenAPI Schema](./candidate_openapi_schema.md)。

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
| Candidate expansion、priority、selection 與數量／失敗門檻 | 語意與欄位名稱已確認；candidate OpenAPI mapping 待審查 | [Topology、policy 與 strategy 細節設計](./topology_policy_design.md)；[Candidate OpenAPI Schema](./candidate_openapi_schema.md) |
| Training／aggregation strategy | `method`／`aggregation`、typed `methodParameters` 與逐級傳遞語意已確認；candidate OpenAPI mapping 待審查 | [Topology、policy 與 strategy 細節設計](./topology_policy_design.md)；[Candidate OpenAPI Schema](./candidate_openapi_schema.md) |
| Node-local `reportAfter` | `epoch`／`round` 語意、parent override／local decision 與 local scope 已確認；candidate OpenAPI mapping 待審查 | [Topology、policy 與 strategy 細節設計](./topology_policy_design.md)；[Candidate OpenAPI Schema](./candidate_openapi_schema.md) |
| Topology status 與逐級回報 | `x-flTopologyReport`、status vocabulary、`FAILED`／`INACTIVE` 的 `statusCause`，以及以同名 `policy`／`strategy`／`reportAfter` 回報實際採用值的語意已確認；candidate OpenAPI mapping 待審查 | [Topology、policy 與 strategy 細節設計](./topology_policy_design.md)；[Candidate OpenAPI Schema](./candidate_openapi_schema.md) |
| `mlCorreId` 與 local process correlation | 已確認 Release 18 至 Release 20 schema／procedure 未限制 hierarchy-wide reuse；共用 ID 是本設計的 hierarchical semantics，subscription lifecycle 與 `roundInd` 維持 local scope | 本文件 §4.7；[標準欄位與 Extension 邊界](./standard_field_extension_boundary.md) |
| 標準欄位與 extension boundary | 已完成 Release 18 request／Notify mapping，並確認 Release 19 unsubscribe-info 與 Release 20 status report 差異；task、data、model、deadline 與 local lifecycle 優先重用既有欄位 | [標準欄位與 Extension 邊界](./standard_field_extension_boundary.md) |
| Branch replacement 與 retained result | 完整 replacement／ownership recovery 延後；`x-retainedResultReq` lookup trigger、`x-retainedResultStatus` outcome 與既有 immediate／Notify 回報方式已確認 | 本文件 §4.8；[Topology、policy 與 strategy 細節設計](./topology_policy_design.md)；[標準欄位與 Extension 邊界](./standard_field_extension_boundary.md) |
| Extension feature negotiation 與 rejection | 已確認重用 `suppFeats`、candidate feature 3，並以既有 `400`／`403` error semantics 拒絕無效或無法履行的 instruction | 本文件 §4.9；[Candidate OpenAPI Schema](./candidate_openapi_schema.md) |

---

## 7. 下一步

1. 審查 [Candidate OpenAPI Schema](./candidate_openapi_schema.md) 與
   [Candidate OpenAPI artifact](./candidate_openapi.yaml) 的 message mapping、
   component types、validation rules、HTTP examples，以及
   [Protocol Conformance Matrix](./protocol_conformance_matrix.md) 的預期結果。
2. 設計確認後，將 artifact 映射到 `NWDAF` 現有 Model Training wire model、
   request／Notify validation boundary 與 procedure state owner，再切分實作。

---

## 8. 證據來源

- [TS 23.288 Release 18 §6.2C Federated Learning among Multiple NWDAFs](../../../specs/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2C%20Federated%20Learning%20among%20Multiple%20NWDAFs.md)
- [TS 23.288 Release 18 §6.2F Procedure for ML Model Training](../../../specs/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2F%20Procedure%20for%20ML%20Model%20Training.md)
- [TS 29.520 Release 18 Nnwdaf_MLModelTraining OpenAPI](../../../specs/openapi/TS29520_Nnwdaf_MLModelTraining.yaml)
- [TS 29.520 Release 18 Nnwdaf_MLModelProvision OpenAPI](../../../specs/openapi/TS29520_Nnwdaf_MLModelProvision.yaml)
- [TS 29.500 Release 18 §6.6 Extensibility Mechanisms](../../../specs/TS%2029.500/6%20General%20Functionalities%20in%20Service%20Based%20Architecture/6.6%20Extensibility%20Mechanisms.md)
- [3GPP official Release 18 `TS29520_Nnwdaf_MLModelTraining.yaml`](https://forge.3gpp.org/rep/all/5G_APIs/-/blob/REL-18/TS29520_Nnwdaf_MLModelTraining.yaml)
- [3GPP official Release 19 `TS29520_Nnwdaf_MLModelTraining.yaml`](https://forge.3gpp.org/rep/all/5G_APIs/-/blob/REL-19/TS29520_Nnwdaf_MLModelTraining.yaml)
- [3GPP official Release 20 `TS29520_Nnwdaf_MLModelTraining.yaml`](https://forge.3gpp.org/rep/all/5G_APIs/-/blob/REL-20/TS29520_Nnwdaf_MLModelTraining.yaml)
- [TS 23.288 Release 19 V19.7.0](https://www.etsi.org/deliver/etsi_ts/123200_123299/123288/19.07.00_60/ts_123288v190700p.pdf)
- [TS 23.288 Release 20 V20.0.0 source](https://www.3gpp.org/ftp/Specs/archive/23_series/23.288/23288-k00.zip)
- [TS 29.520 Release 19 V19.7.0](https://www.etsi.org/deliver/etsi_ts/129500_129599/129520/19.07.00_60/ts_129520v190700p.pdf)
- [TS 29.520 Release 20 V20.0.0 source](https://www.3gpp.org/ftp/Specs/archive/29_series/29.520/29520-k00.zip)
- [Flower `FedAvg` ServerApp strategy（revision `492a31b`）](https://github.com/adap/flower/blob/492a31baf6e6dafbfddc4ad12dcb04ec279ac4be/framework/py/flwr/serverapp/strategy/fedavg.py)
- [Flower legacy `FaultTolerantFedAvg`（revision `492a31b`）](https://github.com/adap/flower/blob/492a31baf6e6dafbfddc4ad12dcb04ec279ac4be/framework/py/flwr/server/strategy/fault_tolerant_fedavg.py)
- 現行實作：`PyMTLF/src/py_mtlf/core/fl_topology.py`,
  `fl_hierarchy.py`、`fl_branch.py`

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
| 2026-09-02 | 確認 Notify 使用 `x-flTopologyReport` 回報 realized topology，並重用同名 `policy`、`strategy` 與 `reportAfter` 表示實際採用值；topology report 不跨層攜帶 descendants 的 `roundInd`。 |
| 2026-09-02 | 完成 TS 23.288／TS 29.520 Release 18 至 Release 20 `mlCorreId` 查核；確認 hierarchy-wide reuse 不受既有 schema／procedure 排除，但屬於本設計新增的 hierarchical correlation semantics。 |
| 2026-09-02 | 建立標準欄位與 extension boundary 細節文件；確認 task、data／time requirement、model、deadline、round 與 local lifecycle 應重用既有欄位，candidate extension 僅補 hierarchical orchestration semantics。 |
| 2026-09-02 | 加入 retained-result lookup：`x-retainedResultReq` 觸發查詢，`x-retainedResultStatus` 明確回報 `FOUND`／`NOT_FOUND`／`FAILED`，並可選擇搭配既有 immediate reporting；查詢不觸發新訓練。 |
| 2026-09-02 | 確認 topology report 的 `FAILED`／`INACTIVE` node 必須帶 `statusCause`；其 vocabulary 與既有 HTTP、training termination 及 retained-result cause／outcome 分開處理。 |
| 2026-09-02 | 確認整組 extension 重用 per-resource `suppFeats` negotiation，使用 candidate feature 3，並完成 schema-invalid 與無法履行 contract 的既有 `400`／`403` error mapping。 |
| 2026-09-02 | 補齊 subtree identity binding／uniqueness、explicit 與 local candidate priority、Notify `mlCorreId`、disabled-child cleanup 與 unsupported feature failure semantics；candidate enums 採 3GPP forward-compatible pattern，`strategy.method` 維持 typed closed discriminator。 |
| 2026-09-02 | 確認同一 subscription 的 retained-result lookup 必須序列化；只有收到前一次 outcome 後才能開始下一次，不增加 request ID。另統一 replacement array 中 explicit prohibition 的解除語意。 |
