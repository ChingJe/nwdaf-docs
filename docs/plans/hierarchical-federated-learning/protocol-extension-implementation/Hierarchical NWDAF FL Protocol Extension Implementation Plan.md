# Hierarchical NWDAF FL Protocol Extension Implementation Plan

日期：2026-09-07

狀態：Slice 1、2、4A、4、5、6 Committed；Slice 3 Branch replacement detailed
plan Review Confirmed／Commit Approval Pending；retained-result runtime暫緩

索引：

- [Protocol Extension Implementation Plans](./README.md)
- [Protocol Implementation Current-State Inventory](./Protocol%20Implementation%20Current-State%20Inventory.md)
- [Model Bundle Metadata to Protocol Schema Mapping](./Model%20Bundle%20Metadata%20to%20Protocol%20Schema%20Mapping.md)
- [Protocol Extension Implementation Slice Map](./Protocol%20Extension%20Implementation%20Slice%20Map.md)
- [Protocol Extension Implementation Review Ledger](./Protocol%20Extension%20Implementation%20Review%20Ledger.md)
- [Slice 3 Detailed Plan](./slices/Slice%203%20Branch%20Replacement%20without%20Retained-result%20Recovery%20Detailed%20Plan.md)

設計輸入：

- [Hierarchical NWDAF Federated Learning 協定設計](../../../design/hierarchical-federated-learning/protocol_design.md)
- [Topology、Policy 與 Strategy 細節設計](../../../design/hierarchical-federated-learning/topology_policy_design.md)
- [標準欄位與 Extension 邊界](../../../design/hierarchical-federated-learning/standard_field_extension_boundary.md)
- [Candidate OpenAPI Schema](../../../design/hierarchical-federated-learning/candidate_openapi_schema.md)
- [Candidate OpenAPI artifact](../../../design/hierarchical-federated-learning/candidate_openapi.yaml)
- [Protocol Conformance Matrix](../../../design/hierarchical-federated-learning/protocol_conformance_matrix.md)
- [NWDAF Development Policy](../../../development_policy.md)

既有 implementation baseline：

- [Hierarchical NWDAF Federated Learning Implementation Plan](../hierarchical-fl-model-bundle-edition/Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)
- [Explicit Flat 與 Hierarchical Orchestration 設定詳細計畫](../explicit-flat-hierarchical-orchestration/Explicit%20Flat%20and%20Hierarchical%20Orchestration%20Configuration%20Detailed%20Plan.md)

---

## 1. 背景

現有 Root–Branch–Leaf HFL 已在 `NWDAF`／`PyMTLF` multi-process environment
跑通，但 topology assignment、strategy 與 preparation result 主要透過 model
bundle metadata 傳遞。這證明 hierarchical execution 可行，卻沒有讓這些
orchestration semantics 成為 `Nnwdaf_MLModelTraining` subscription resource
的一部分。

新的 candidate protocol 已定義 recursive topology、node-scoped policy／strategy、
realized-topology report、feature negotiation 與 retained-result fields。本
workstream 的目的，是在保留既有 HFL 執行能力的前提下，先將 topology、policy、
strategy 與 report semantics 接入正式 Model Training message flow。Retained-result
fields 保留在 wire contract，但本階段不建立其執行狀態與 recovery behavior。

---

## 2. 目標

### 2.1 Protocol 目標

- Subscription／PUT／PATCH 可逐級傳遞 recursive `x-flTopology`。
- Node 可同時承載 direct-child `policy`、local-process `strategy` 與對 direct
  parent 的 `reportAfter` instruction。
- Notify 可用 `x-flTopologyReport` 回報 realized topology、node status 與實際
  採用的 contract。
- 每條 parent-to-child subscription edge 可透過 `suppFeats` 協商
  `HierarchicalFLOrch`。
- 保留 `x-retainedResultReq` 與 `x-retainedResultStatus` 的 schema、解析與驗證能力；
  本階段不執行 lookup，也不以 retained result 接續替代 node 的訓練。
- Root 以 UUID 字串產生每個 hierarchical FL procedure 的 `mlCorreId`，整棵
  hierarchy 逐級共用同一值。此識別用途不代表 node 必須保存可依該 ID 查詢的歷史
  result artifact。
- Training途中一個direct Branch發生可歸因的availability failure時，Root可從該
  Branch group的direct-child candidates選出新Branch、重建該group的Leaf candidate
  contract，並從上一個成功的Root global model繼續後續training。

### 2.2 Execution 目標

- Protocol fields 必須到達真正的 instruction producer、state owner 與 execution
  consumer，不能只完成 schema parsing。
- Intermediate 可同時執行上層明確指定的 children 與獲授權的 local candidate
  selection。
- FL Server 可依 participant policy 決定 topology readiness、per-round selection
  與 aggregation completion。
- FL Client／Intermediate 可依 strategy 與 `reportAfter` 執行 local work 並向
  parent 回報。
- Subscription state 與 local process state 必須有清楚且互不混淆的 lifecycle owner。

### 2.3 Migration 目標

現有 model-bundle metadata 是可回歸的 production baseline，不在新路徑尚未成立
前移除。完成 protocol path 後，應使同一項 orchestration information 只有一個
authoritative source，避免 bundle 與 message field 同時控制行為。

---

## 3. 主要情境

### 3.1 Hierarchy establishment

Root 建立第一段 Model Training subscription，將接收 node 的 subtree、policy、
strategy 與 reporting instruction 放入 `x-flTopology`。Intermediate 只取出自己
負責的 direct-child contract，選擇或補充 candidates，再逐級建立下一層
subscriptions。

Preparation subscription 設定 `mLPreFlag: true`，但不提供 `mLModelInfos`。接收端
只驗證request實際提供的標準欄位、hierarchy contract、自身可用性與local workload
readiness，不下載、載入或驗證實際 model artifact。TS 29.520 Release 18
§4.6.2.2.2／§5.5.6.2.2 將 `mLModelInfos` 定義為 optional，因此 topology
establishment 不需要先發布 global model。

每個 node 透過 `x-flTopologyReport` 向 direct parent 回報實際建立結果與採用的
contract；Root 最終可取得整體 realized topology，但不需要直接建立所有
lower-tier resources。

### 3.2 Normal training and topology update

一般 training round 沿用既有 `roundInd` 與 model／result fields，不重送整棵
topology。只有 membership、candidate instruction 或 local process contract 改變
時，parent 才透過 PUT／PATCH 更新相關 subtree。

Root 等待逐級回傳的 `x-flTopologyReport` 滿足 topology readiness，取得實際建立的
participant identities 後，才為第一輪建立 ADRF global-model record。Store request 的
`allowConsumerList` 明確列出該輪可能取得模型的 realized Branch／Leaf NWDAFs；若能以
既有 NF Set 完整且不過度授權地涵蓋相同集合，也可使用 `nfSetId`。Root 隨後以第一輪
PUT／PATCH 將 `mLPreFlag` 改為 `false`，並下發 `roundInd` 與標準
`mLModelInfos[].mLModelAdrf` reference，正式啟動 training loop。

Branch 以新收到的 Root global model 啟動 lower-tier round 時，對 selected children
原樣傳遞同一 ADRF reference；Branch 與 Leaves 直接向 ADRF 取得該 global model，不由
Intermediate 重新發布。若 Branch 在下一次 upstream update 前，以自身完成的 domain
aggregate 繼續執行 lower-tier round，則由 Branch 將該 aggregate 發布在自己的暫存
workspace，並以 `mLFileAddr` 下發給 selected children，不存入 ADRF。由下往上的 Leaf
local result 與 Branch domain aggregate 也維持由各產生節點以暫存 `mLFileAddr`
提供。Topology update 若加入新的 Root global-model consumer，Root 必須先更新該
record 的 `allowConsumerList`，或在下一輪建立的新 record 納入該 identity，之後才能向
該 node 下發 ADRF reference。

目前 testbed ADRF 尚未從 authenticated request context 強制執行
`allowConsumerList`；protocol path 仍必須產生並傳送正確清單，不能再依賴省略欄位形成
實質上的開放存取。Real-process evidence 可驗證 store request 與 record 中的清單，但在
caller authentication 實作前，不能宣稱已驗證 ADRF authorization enforcement。

Explicit children 與 delegated selection 可以共存；不另外新增 orchestration
mode。上層提供的 children 是 explicit candidate set，node 是否能自行加入其他
candidates 由該 node 的 policy 決定。

### 3.3 Branch replacement support

當 active Branch 失效時，Root 可依內部 selection mechanism 選擇替代 Branch，
再使用既有 subscription operation 將新的 subtree contract 下發。替代 Branch 重新
建立 downstream subscriptions 後，依 Root 後續下發的 model／round instruction
繼續執行。Root將單一direct Branch的round dispatch failure、termination Notify或
response deadline timeout分類為可恢復的availability failure；candidate使用前必須經
NRF fresh exact-ID resolve。

Root與各Branch的policy均放在local topology對應層級，並實際控制該Server對direct
children的readiness、per-round selection與completion。Branch失效時，Root先依
`acceptFailures`與`minCompletionRate`判斷當輪；policy接受便只聚合成功Branches，拒絕才
丟棄partial results。只要剩餘Branches仍符合Root policy，training在replacement
preparation期間繼續；replacement只從下一個尚未dispatch的round加入。新Branch與Leaves
沿用同一hierarchy-wide `mlCorreId`，但使用新的per-edge resource與`notifCorreId`。

本流程不送出`x-retainedResultReq`，也不恢復或沿用失敗路徑尚未送達的local result。
完整multi-vendor re-parent authorization、old Branch恢復後的ownership handback、Root
restart recovery與多個Branches同時失效仍不在本階段處理。

---

## 4. 實作範圍

預期涉及：

- `NWDAF/`：External／peer SBI wire contract、validation、resource state、callback
  routing、Go→PyMTLF transport，以及Leaf rebind時由backend主動淘汰舊inbound route的
  internal lifecycle operation。
- `PyMTLF/`：Topology／policy／strategy execution、local process state 與 realized
  report owner，以及controlled local training workload。
- `nwdaf-resources/`：Real-process request／Notify evidence、negative cases 與
  regression scenarios；dataset產生與切分由experiment工作另行提供。
- `nwdaf-docs/`：Canonical plan、盤點、conformance mapping 與 review evidence。

目前不預期修改 NRF schema 或讓 NRF 保存 hierarchy-specific topology／policy。
`adrf/` 是 global-model distribution 的 runtime dependency 與 real-process evidence
component，但依目前 production trace 不預期修改其 repository。
其他 repository 只有在完整資料流證明存在 current-work blocker 時才納入。

---

## 5. 高階工作拆分

以下是 workstream 的主要工作面向，不預設它們等同 implementation slice，也不先
固定 commit 或執行順序。

### 5.1 Wire contract 與驗證

- 將 candidate request、PATCH 與 Notify fields 接入 Go／Python wire models。
- 保證合法 extension fields 在 parse、marshal、callback rewrite、accepted
  response 與 PATCH lifecycle 中不遺失。
- 實作 schema、cross-field、identity 與 error mapping rules。

### 5.2 Resource state 與 feature negotiation

- 建立每個 subscription resource 的 `HierarchicalFLOrch` negotiation 與後續
  operation gate。
- 區分 persistent subscription contract與local process state；retained-result
  instruction 不進 persistent representation，也不在本階段建立 lookup state。
- 維持 Create／PUT／PATCH atomicity、DELETE cleanup 與 restart boundary。

### 5.3 Controlled local training workload

- 以known workload與data-source config區分既有`UE_COMMUNICATION` forecasting及
  controlled image classification；MNIST／CIFAR-10只驗證FL與systems behavior，不表示
  它們是標準NWDAF analytics資料。
- Dataset／experiment工作一次產生所有Client shards；部署時使用共同config template，
  將不同shard以read-only方式掛載到各PyMTLF相同的容器內路徑。每個instance不執行
  dataset preparation，也不維護manifest或hash設定。
- `workload.profile`選擇task／training semantics，data-source設定選擇既有
  `consumer_subscription`、`private_api`或controlled `local`；標準
  `UE_COMMUNICATION + consumer_subscription`行為保持不變。
- `image_classification`再由local config選擇MNIST或CIFAR-10的preprocessing contract，
  並與received model bundle交叉驗證，同時重用既有FedProx與sample-weighted
  aggregation owner。
- Final global model在training結束後，以獨立held-out test set計算accuracy；不把traffic
  WAPE acceptance gate套用到classification workload。

### 5.4 Topology orchestration 遷移

- 將 recursive subtree instruction 與 realized report 接入既有 Root／Intermediate／
  Client flow。
- 支援 explicit／delegated hybrid selection、priority、node status 與 topology
  updates。
- 將 preparation 與 model distribution 分離：preparation 不帶 model；Root 在
  realized topology ready 後，才建立含 `allowConsumerList` 的 ADRF record，並於
  training round 逐級下發同一 `mLModelAdrf` reference。Branch 自身完成的 domain
  aggregate 若作為後續 lower-tier round 輸入，則由 Branch 以暫存 `mLFileAddr`
  下發；上行 local／aggregate result 也維持由產生節點以暫存 `mLFileAddr` 提供。
- 在 protocol path 成立後，移除相同 orchestration information 對 model-bundle
  metadata 的 runtime dependency。

### 5.5 Digest contract簡化

- 完整壓縮artifact bytes的SHA-256保留為repository key與URL identity；下載者只比較
  實際bytes與URL key。
- 移除component、model／weights、scope／dataset／tensor、Notify body、topology與
  collection content digest；不以新的config fingerprint或sidecar取代。
- 以typed model／workload identity、process／round／resource state、explicit topology
  與state-dict compatibility保留真正必要的語意檢查。
- 此cleanup在protocol E2E integration前獨立完成，避免新路徑繼續依賴即將移除的
  project-private digest contract。

### 5.6 Policy、strategy 與 reporting execution

- 讓 participant policy 實際控制 topology readiness、round selection 與
  aggregation completion。
- 讓 strategy 與 `reportAfter` 到達 local training／aggregation owner。
- 讓 node 回報實際採用的 policy、strategy 與 reporting contract。

### 5.7 Retained-result boundary

- 保留 Slice 1 已完成的 `x-retainedResultReq`／`x-retainedResultStatus` wire models、
  non-persistence 與 cross-field validation。
- Production execution 不建立 `mlCorreId -> latest completed result` index，不接受
  lookup 為可執行 operation，也不為此新增 artifact retention timer／cleanup owner。
- 未來若重新採用 retained result，必須另行決定保存期限、artifact ownership、
  replacement subscription correlation、timeout 與 cleanup，再建立獨立實作計畫。

### 5.8 Branch replacement execution

- Static topology以多個Branch groups表示並行assignment；每組將有priority的Branch
  candidates與只宣告一次的Leaf candidates分開保存，不以重複Leaf subtree推導群組。
- Root選Branch與Branch選Leaf共用既有direct-child candidate pool、priority與policy
  semantics；initial preparation每組只選出一個active Branch，replacement使用前經NRF
  fresh exact-ID resolve。本階段production recovery與E2E只驗證Branch replacement。
- Root policy與各group的Branch-to-Leaf policy直接放在static topology對應層級；runtime
  必須使用configured minimum、fraction與completion threshold，不保留hard-coded
  all-required policy。
- 將direct Branch availability failure從validation／aggregation／ADRF／Root internal
  failure中明確分類；只有前者能進入bounded replacement loop。
- Replacement以same `mlCorreId`與same Leaf subtree建立新resources；Leaf成功接受新edge
  後淘汰舊edge的pending work與callback correlation，並要求containing Go NWDAF依
  backend resource identity／generation退休對應inbound route。
- Root policy接受的degraded round只聚合successful Branch results；拒絕的attempt才丟棄
  partial results。剩餘cohort符合readiness時不等待replacement，replacement ready後只
  加入下一個尚未dispatch的round。
- Candidate exhaustion後依Root readiness policy決定維持degraded training或terminal
  cleanup；同時多Branch失效與non-recoverable errors仍terminal。Leaf replacement的
  production transition與E2E evidence維持延後。

### 5.9 驗證與 migration closure

- 將 Protocol Conformance Matrix 映射到 unit、boundary 與 real-process tests。
- Slice 5已完成protocol-driven local real-process E2E；Slice 6改碼前只保留一次
  static／model-bundle migration checkpoint，cutover後移除舊runtime path。
- 分別驗證 explicit topology、delegated／hybrid topology、normal rounds、topology
  update 與 feature rejection。

---

## 6. 明確非目標

- 新 FL algorithm 或 learning-quality claim。
- Retained-result persistence、lookup、artifact retention policy與舊計算結果接續。
- 通用re-parent authorization、跨節點ownership／fencing token、retained-result
  freshness、deduplication 或其aggregation acceptance；本slice仍需完成本地resource
  revision與late callback fencing。
- Leaf replacement、多個Branches同時失效、任意NRF list discovery、Root restart
  recovery與old Branch ownership handback。
- NRF schema extension、application-specific topology storage 或 ranking algorithm。
- OAM／MANO、runtime topology optimizer 或一般化self-healing。
- Flat／hierarchical experiment metrics instrumentation。

---

## 7. 目前狀態與下一步

- Candidate protocol、OpenAPI artifact 與 conformance cases 已有設計輸入。
- Slice 1、2、4A、4、5與6均已完成審查、驗證及commit；protocol-driven hierarchy、
  protocol-only migration與standard flat／distributed FL已有local real-process evidence。
- Slice 3已重新界定為「不使用retained result的Branch replacement」；完整實作計畫已
  通過user review並等待commit approval，尚未進入production implementation。
- Retained-result runtime維持暫緩；正式multi-host testbed仍是integration verification
  gap。

已依
[Model Bundle Metadata to Protocol Schema Mapping](./Model%20Bundle%20Metadata%20to%20Protocol%20Schema%20Mapping.md)
確認舊 model-bundle control metadata 的 migration boundary，並以
[Protocol Implementation Current-State Inventory](./Protocol%20Implementation%20Current-State%20Inventory.md)
完成各欄位的 production trace 與 gap mapping。Implementation work units 已整理於
[Protocol Extension Implementation Slice Map](./Protocol%20Extension%20Implementation%20Slice%20Map.md)；
各slice的實作與驗證證據記錄於
[Protocol Extension Implementation Review Ledger](./Protocol%20Extension%20Implementation%20Review%20Ledger.md)；
Slice 3的現況盤點、精確owner、round semantics、實作順序與驗收條件見
[Slice 3 Detailed Plan](./slices/Slice%203%20Branch%20Replacement%20without%20Retained-result%20Recovery%20Detailed%20Plan.md)。
