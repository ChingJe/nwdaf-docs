# Hierarchical NWDAF FL Protocol Implementation Current-State Inventory

日期：2026-09-02

狀態：Ready for User Review／production owner、resource lifecycle 與 slice boundary
已完成第一輪盤點

相關文件：

- [Protocol Extension Implementation Plan](./Hierarchical%20NWDAF%20FL%20Protocol%20Extension%20Implementation%20Plan.md)
- [Model Bundle Metadata to Protocol Schema Mapping](./Model%20Bundle%20Metadata%20to%20Protocol%20Schema%20Mapping.md)
- [Protocol Resource Lifecycle and Wire Integration Mapping](./Protocol%20Resource%20Lifecycle%20and%20Wire%20Integration%20Mapping.md)
- [Protocol Conformance Case Ownership Mapping](./Protocol%20Conformance%20Case%20Ownership%20Mapping.md)
- [Protocol Extension Implementation Slice Map](./Protocol%20Extension%20Implementation%20Slice%20Map.md)
- [Protocol Conformance Matrix](../../../design/hierarchical-federated-learning/protocol_conformance_matrix.md)
- [Candidate OpenAPI artifact](../../../design/hierarchical-federated-learning/candidate_openapi.yaml)

---

## 1. 文件目的

本文件承載實作前的詳細盤點，包括 repository revision、現有 production flow、
component owner、目標差距與尚待確認的資料流。主計畫只保留背景、目標、情境、
範圍與高階工作拆分。

本次盤點已完成第一輪 implementation slice boundary；若後續 production evidence
改變 owner 或 dependency，必須先更新本文件與 slice map，再調整實作範圍。

---

## 2. 盤點基準

| Repository | Revision | 盤點時工作樹 | 與目標的關係 |
| --- | --- | --- | --- |
| `NWDAF/` | `6aed268d6528f8be6c729cbd45b59d067e5e80dc` | Clean | External／peer SBI、Go→PyMTLF gateway、route state 與 validation |
| `PyMTLF/` | `747962971b63f0a53031d52a1eb7e047ae776998` | Clean | Hierarchical orchestration、local FL process、artifact 與 training state owner |
| `nwdaf-resources/` | `83473770e958623f052097349173f56af1e25953` | Clean | Real-process scenarios、protocol evidence 與 regression harness |
| `nwdaf-docs/` | `e38a6faa9a6a2706c6214b817134d0bc5a7d303a` | 本 workstream 文件尚未提交 | Canonical design、plan 與 conformance evidence |

目前不預期修改 `nrf/`、`smf-nwdaf-ext/`、`udm/`、`udr/`、`adrf/`、`PyAnLF/`
與 `resources/`。`adrf/` 雖不是預期修改對象，仍是 Slice 4 global-model
distribution 的 runtime dependency 與 real-process 受測 component；evidence harness
仍由 `nwdaf-resources/` 擁有。NRF 維持既有
registration／discovery owner，不保存
hierarchical topology 或 policy。這是依目前資料流得到的範圍假設；若後續
production trace 證明 required value 無法在上述 owner 間產生或傳遞，需先更新
盤點與主計畫，不能直接把 receiver-local 欄位視為完整實作。

---

## 3. 現有實作

### 3.1 `NWDAF` Model Training path

目前已具備：

- Release 18 compatibility models：`NwdafMLModelTrainSubsc`、
  `NwdafMLModelTrainSubscPatch` 與 `NwdafMLModelTrainNotif`。
- Public SBI、peer NWDAF 與 local PyMTLF 之間的 Create、PUT、PATCH、DELETE、
  Notify flow。
- Route state 保存 public／backend representation、peer resource、callback URI、
  `notifCorreId`、`mlCorreId` 與 expected `roundInd`。
- FL preparation、resource identity、PATCH 與 Notify correlation validation。
- Mutation revision、backend generation、delete cleanup 與 callback fencing。

與新 protocol 的直接差距：

- Go compatibility models 尚未定義 candidate `x-*` fields，也沒有 hierarchical
  cross-field validation。
- Create／PUT／PATCH 的 backend response 會經 typed parse 再 marshal；PATCH
  effective representation 也由 typed model 重建。因此新欄位若只被當成未知
  JSON，不能保證沿完整 resource lifecycle 保留。
- Notify 目前會以 typed model 驗證後轉送原始 body，可保留未知 JSON bytes，
  但現行 validator、route state 與 consumer 都不理解 topology report 或
  retained-result outcome。
- `suppFeats` 目前只有 wire field，尚未形成每個 training resource 的
  `HierarchicalFLOrch` negotiation 與 operation gate。

### 3.2 `PyMTLF` Model Training resource

目前已具備：

- 與 Go gateway 對應的 Create、PUT、PATCH、DELETE 與 Notify private routes。
- `FLClientEngine` 保存 subscription representation、local state、revision、
  prepared dataset、hierarchy assignment 與 Branch lower-process binding。
- Pydantic standard-shaped models 允許未知 properties，resource response 會從
  representation 重新輸出，因此單純欄位保存能力比 Go typed reconstruction
  寬鬆。
- Preparation、round training、final validation、callback retry／delay 與
  hierarchy-bound cleanup 已有 production owner。
- `MLEventNotification` wire model 已能解析 `mLModelAdrf`，但 FL preparation 目前仍
  要求 `mLFileAddr`，normal-round execution 也只接受 `mLFileAddr`。目標 preparation
  必須能在省略 `mLModelInfos` 時只建立 topology；normal-round execution 則需依 model
  producer 支援兩種標準來源：Root global model 使用 `mLModelAdrf` 並查詢 ADRF record，
  Branch domain aggregate 使用產生 Branch 的暫存 `mLFileAddr`。兩者不得被合併成一律
  republish 或一律存入 ADRF 的單一路徑。

與新 protocol 的直接差距：

- Wire models 尚未把 `x-flTopology`、`x-flTopologyReport` 與 retained-result
  fields 建成有語意的 typed properties。
- 現有 engine 不會根據這些 extension 執行 topology、policy、strategy 或
  report contract；未知 property 即使被保留，也不等於 protocol behavior 已實作。
- 現有 PATCH 使用 top-level update 建立 effective representation；recursive
  topology array replacement、disabled-child cleanup 與 operation-scoped lookup
  尚未映射到 resource transition。

### 3.3 現有 hierarchical orchestration

目前 hierarchy 已能實際執行，但控制資訊走的是舊路徑：

1. Root 從 static topology YAML 建立明確的 Branch／Leaf assignment。
2. Root 透過 NRF 解析已指定的 Branch 與 Leaf capability。
3. Root 將 Branch assignment、完整 Leaf list、FedProx strategy 與
   `complete_required` admission 放入 model bundle artifact。
4. Branch 從 preparation request 的 model URL 下載 assignment，解析後再為每個
   Leaf 發布 Leaf assignment artifact，並建立 lower-tier subscriptions。
5. Branch 將 preparation partition、lower-tier aggregation 與 validation evidence
   再包入 artifact，透過既有 Model Training result path 回到 Root。

現有策略與限制為：

- Hierarchical participant source 固定為 `static`。
- Root 明確提供所有 children；沒有 delegated candidate expansion。
- Participant selection、waiting policy 固定為 `all`，admission 固定為
  `complete_required`。
- Training method 固定為 FedProx，aggregation 固定為 `sample_weighted`。
- Client epochs、Root round count 與 deadline 來自 local config／round artifact，
  尚未使用 node-level `reportAfter` contract。
- Branch 顯式保存 upper `mlCorreId` 與獨立 lower process ID／round mapping；尚未
  改為整個 hierarchy 共用 `mlCorreId` 的新 protocol semantics。

### 3.4 `nwdaf-resources` real-process baseline

現有 harness 已有：

- `smoke`：一個 Root、一個 Branch、兩個 Leaves。
- `aggregation`：一個 Root、兩個 Branch、四個 Leaves。
- `manual-success`、`capability-mismatch`、`preparation-failure`、
  `round-timeout`、`restart-generations`、`degradation-success` 等情境。
- Real NWDAF／PyMTLF multi-process preparation、two-tier aggregation、final
  validation、publication、cutover 與 cleanup evidence。

目前這些 scenarios 驗證的是 static topology 與 model-bundle assignment，不是
candidate `x-*` message contract。後續可重用 process topology 與 failure
injection，但 request／Notify evidence 與 assertions 需要改成 protocol field。

---

## 4. Baseline flow 與待補 connection

| Baseline stage | 現有 production behavior | 本 workstream 需要確認／補足 |
| --- | --- | --- |
| Root initiation | Private API／degradation trigger 讀取 static topology 與 config | 已確認由 Root PyMTLF coordinator 產生首段 `x-flTopology`；Go route 只保存、驗證與轉送 contract |
| Public／backend Create | Go 驗證標準欄位、改寫 callback、轉送 PyMTLF 或 peer | 新欄位 preservation、validation、feature negotiation 與 accepted representation |
| Intermediate preparation | Branch 從 model URL 下載 assignment artifact | 改由 model-free subscription resource 取得 subtree／policy／strategy；`mLPreFlag: true` 且不帶 `mLModelInfos`，接收端不下載或驗證 model artifact |
| Lower-tier dispatch | Branch 依 artifact 中完整 Leaf list discovery 並建立 subscriptions | Explicit／delegated candidates、priority、threshold 與逐級 subtree transport |
| Preparation report | Branch 產生 preparation-result artifact | Branch coordinator／local FL Server state 產生 realized status，由 FL Client Notify 以 `x-flTopologyReport` 逐級回報 |
| Training round | 標準 `roundInd`／model URL；Branch 另維護 lower process 與 round | Root 在 realized topology ready 後建立含 `allowConsumerList` 的 ADRF record，再以第一輪 PUT／PATCH 下發 `roundInd` 與 `mLModelAdrf`；各接收 node 經 containing Go NWDAF 查詢 record 並下載其中 `mLFileAddr`。Branch 轉送 Root global model 時沿用同一 ADRF reference；Branch aggregate 若作為後續 lower-tier round 輸入，則由 Branch 以自己的暫存 `mLFileAddr` 下發。Leaf local result 與 Branch aggregate 也以各產生節點的暫存 `mLFileAddr` 上傳 |
| PUT／PATCH | Go／PyMTLF 重建 effective resource 並執行 replace／patch | Persistent subtree 完整 replacement、operation-scoped command 抽離、reselection、disabled-child cleanup 與原子性 |
| Notify | Go 依 local resource identity 驗證並轉送 raw body | Topology report、resolved policy／strategy 與 retained-result status 的 producer／consumer |
| DELETE／failure | 刪除 resource、取消 hierarchy workspace、保存短期 tombstone | Protocol-owned child cleanup 與 operation-scoped lookup 的 timeout／late outcome |
| Restart | Go generation fencing 與 PyMTLF workspace cleanup 已有情境 | Negotiated feature、persistent contract 與 outstanding lookup 是否需要恢復或明確失效 |

---

## 5. 盤點結論與實作時需落地事項

以下整理已確認的 implementation boundary，以及 detailed slice 開始時仍需落實的
repository-local contract；不代表設計語意尚未決策：

1. [Model Bundle Metadata to Protocol Schema Mapping](./Model%20Bundle%20Metadata%20to%20Protocol%20Schema%20Mapping.md)
   已完成第一輪欄位、producer、transport、state owner 與 consumer 對照；後續若
   production trace 改變任何 owner，必須先更新該文件。
2. Go wire model 已決定延伸現有 `internal/compat/mlmodeltraining`；candidate
   OpenAPI artifact 作為 contract／fixture，不另建 generated module。PyMTLF 對應
   extension 改為 explicit typed properties，不依賴 unknown-field passthrough。
3. Root instruction 的 authoritative producer 已確認為 Root PyMTLF coordinator。
   Preparation builder 下發標準必要／training requirement fields 與
   `x-flTopology`，但不建立 ADRF record，也不附 `mLModelInfos`。Root 收到滿足
   readiness 的 realized topology 後，才重用既有
   PyMTLF ADRF discovery 與 containing Go NWDAF ML Model Management proxy：Root
   PyMTLF 發布暫存 model URL、以 realized identities 產生 `allowConsumerList`、要求
   ADRF 建立 record，驗證 `201`／`Location`／response 後取得 `adrfId`、
   `storTransId` 與 `modelUniqueId`，再於第一輪 PUT／PATCH 將 `mLModelAdrf` 與
   `roundInd` 交給既有 FL Server participant resources。接收端依 reference 透過自己
   的 Go NWDAF 查詢同一 ADRF record，再由 trainer 下載 record 中的 `mLFileAddr`。
   Branch 只有在轉送 Root global model 時沿用該 ADRF reference；Branch domain
   aggregate 作為後續 lower-tier round 輸入時，由 Branch 的暫存 workspace 提供
   `mLFileAddr`，不建立 ADRF record。
   Exact helper placement、每輪 model-payload immutable `modelUniqueId` allocator、ACL
   update ordering 與 record cleanup 由 Slice 4 detailed plan 固定。
4. Subscription contract 中哪些資料由 Go resource state 保存，哪些只由 PyMTLF
   execution state 保存；restart 時各自採恢復或失效語意。
5. `suppFeats` 的 request、accepted response、peer edge 與後續 PUT／PATCH gate
   如何落入現有 route state。
6. Delegated selection 沿用 Go internal NRF proxy，在 PyMTLF hierarchy resolver
   新增省略 exact identity 的 list-discovery operation。Criteria 來自標準 training
   fields、local area／operator config 與既有 NF profile；不新增 NRF schema 或
   custom criteria field。Coordinator 保存 explicit／local provenance 並產生 realized
   report。
7. 現有 complete-required／all-participants server state machine 如何泛化為
   preparation minimum、per-round selection 與 aggregation completion threshold，
   同時保留 timeout 與 failure semantics。
8. `strategy`、`reportAfter` 與 local config 的優先關係，以及 protocol 省略欄位時
   node-local decision 的回報位置。
9. Root 以 UUID 字串產生 hierarchy-wide `mlCorreId`；同一 hierarchy 逐級共用，
   receiver 在本地 retention window 內拒絕與另一 active procedure 衝突的 UUID。
   Retained result 的 local owner 已確認為 PyMTLF `FLClientEngine`；需新增以該 UUID
   索引的 latest-completed record，並另行處理 artifact retention。UUID 格式相容既有
   workspace key 要求，但 retained lifecycle 仍不得直接借用另一個 owner 的
   `release_plan` 語意。
10. Model-bundle metadata 的 migration authority 已確認由 Root PyMTLF
    orchestration 明確選擇 legacy bundle 或 protocol mode；兩者互斥，Go route 不做
    fallback。Protocol E2E 完成前保留 legacy regression，closure 再移除舊 runtime
    path。
11. Protocol Conformance Matrix 的 case owner、test seam 與 real-process evidence
    已完成第一輪對照；retained-result latest-completed state owner 亦已收斂至
    PyMTLF `FLClientEngine`。

---

## 6. 盤點完成條件

- [x] 完成舊 model-bundle control metadata 到 protocol／artifact／local state／移除
  四類的 producer／consumer 對照。
- [x] 逐段確認 Create／PUT／PATCH／DELETE／Notify 的雙向資料流與 current owner。
- [x] 確認每個 extension value 的 producer、transport、stored owner、consumer 與
  failure behavior。
- [ ] 在 Slice 4 detailed plan 固定 model-free preparation gate、ADRF per-round
  `modelUniqueId` allocator、realized-topology-to-`allowConsumerList` mapping、ACL update
  ordering、record cleanup／restart 與 store／retrieval failure mapping，並將其轉成
  boundary／real-process tests。
- [x] 確認 Go／Python model integration 方式與合法欄位 preservation。
- [x] 對照 Protocol Conformance Matrix，標記現有可重用 behavior、確定 gap 與
  test owner。
- [x] 確認 model-bundle metadata migration boundary 與可回歸的 baseline mode。
- [x] 提出可獨立實作、審查與驗證的工作單位，回填主計畫的後續執行順序。
