# Hierarchical NWDAF FL Protocol Extension Implementation Plan

日期：2026-09-02

狀態：Ready for User Review／第一輪實作盤點與 slice map 已完成；production
implementation 尚未開始

索引：

- [Protocol Extension Implementation Plans](./README.md)
- [Protocol Implementation Current-State Inventory](./Protocol%20Implementation%20Current-State%20Inventory.md)
- [Model Bundle Metadata to Protocol Schema Mapping](./Model%20Bundle%20Metadata%20to%20Protocol%20Schema%20Mapping.md)
- [Protocol Extension Implementation Slice Map](./Protocol%20Extension%20Implementation%20Slice%20Map.md)

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
realized-topology report、feature negotiation 與 retained-result lookup。本
workstream 的目的，是在保留既有 HFL 執行能力的前提下，將這些 semantics 接入
正式 Model Training message flow。

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
- 替代 node 可用一次性 `x-retainedResultReq` 查詢相同 `mlCorreId` 的最新已完成
  結果，並透過 `x-retainedResultStatus` 回報 outcome。
- Root 以 UUID 字串產生每個 hierarchical FL procedure 的 `mlCorreId`，整棵
  hierarchy 逐級共用同一值；receiver 在本地 retention window 內不得接受與另一個
  active procedure 衝突的 UUID。

### 2.2 Execution 目標

- Protocol fields 必須到達真正的 instruction producer、state owner 與 execution
  consumer，不能只完成 schema parsing。
- Intermediate 可同時執行上層明確指定的 children 與獲授權的 local candidate
  selection。
- FL Server 可依 participant policy 決定 topology readiness、per-round selection
  與 aggregation completion。
- FL Client／Intermediate 可依 strategy 與 `reportAfter` 執行 local work 並向
  parent 回報。
- Subscription state、local process state 與 operation-scoped lookup state 必須有
  清楚且互不混淆的 lifecycle owner。

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
只驗證 training requirements、`modelInterInfo`、hierarchy contract 與自身可用性，
不下載、載入或驗證實際 model artifact。TS 29.520 Release 18
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
再使用既有 subscription operation 將新的 subtree contract 下發。若需要接續既有
計算結果，Root 可要求替代 Branch 對相關 Clients 執行一次 retained-result lookup。

本 workstream 只提供 topology update 與 lookup 所需的 protocol information；完整
failure detection、replacement selection、fencing 與 result acceptance 仍由內部
機制或後續工作負責。

---

## 4. 實作範圍

預期涉及：

- `NWDAF/`：External／peer SBI wire contract、validation、resource state、callback
  routing 與 Go→PyMTLF transport。
- `PyMTLF/`：Topology／policy／strategy execution、local process state、realized
  report 與 retained-result owner。
- `nwdaf-resources/`：Real-process request／Notify evidence、negative cases 與
  regression scenarios。
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
- 區分 persistent subscription contract、local process state 與一次性 lookup
  state。
- 維持 Create／PUT／PATCH atomicity、DELETE cleanup 與 restart boundary。

### 5.3 Topology orchestration 遷移

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

### 5.4 Policy、strategy 與 reporting execution

- 讓 participant policy 實際控制 topology readiness、round selection 與
  aggregation completion。
- 讓 strategy 與 `reportAfter` 到達 local training／aggregation owner。
- 讓 node 回報實際採用的 policy、strategy 與 reporting contract。

### 5.5 Retained-result lookup

- 實作一次性 lookup request 與 `FOUND`／`NOT_FOUND`／`FAILED` outcome。
- 明確定義 latest-completed result owner、同一 resource 的 operation serialization
  與 timeout／termination behavior。
- 一般 training result 繼續比對 route 的 expected `roundInd`；
  `x-retainedResultStatus: FOUND` 的 `roundInd` 則表示被查到的 local result round，
  只驗證格式、artifact 與 procedure correlation，不與 replacement subscription 的
  expected round 比較。
- 不在此工作面向擴張為完整 Branch recovery protocol。

### 5.6 驗證與 migration closure

- 將 Protocol Conformance Matrix 映射到 unit、boundary 與 real-process tests。
- 保留既有 static／model-bundle baseline，直到 protocol-driven E2E 可以回歸。
- 分別驗證 explicit topology、delegated／hybrid topology、normal rounds、topology
  update、feature rejection 與 retained-result lookup。

---

## 6. 明確非目標

- 新 FL algorithm 或 learning-quality claim。
- Branch failure replacement 的完整 orchestration protocol。
- Re-parent authorization、fencing、ownership token、retained-result freshness、
  deduplication 或 aggregation acceptance。
- NRF schema extension、application-specific topology storage 或 ranking algorithm。
- OAM／MANO、runtime topology optimizer 或完整 self-healing。
- Flat／hierarchical experiment metrics instrumentation。

---

## 7. 目前狀態與下一步

- Candidate protocol、OpenAPI artifact 與 conformance cases 已有設計輸入。
- 既有 static／model-bundle HFL 已有 local 與 real-process baseline。
- Production implementation 尚未開始；current-state、owner 與主要 data-flow 已完成
  第一輪盤點。ADRF global-model distribution 的 exact method placement、record
  lifecycle 與驗證 evidence 已列入 Slice 4 closure，等待 user review。

已依
[Model Bundle Metadata to Protocol Schema Mapping](./Model%20Bundle%20Metadata%20to%20Protocol%20Schema%20Mapping.md)
確認舊 model-bundle control metadata 的 migration boundary，並以
[Protocol Implementation Current-State Inventory](./Protocol%20Implementation%20Current-State%20Inventory.md)
完成各欄位的 production trace 與 gap mapping。Implementation work units 已整理於
[Protocol Extension Implementation Slice Map](./Protocol%20Extension%20Implementation%20Slice%20Map.md)；
下一個 production work unit 是 Slice 1，但開始修改 production code 前需先完成 Slice 1
detailed plan 與 user review。
