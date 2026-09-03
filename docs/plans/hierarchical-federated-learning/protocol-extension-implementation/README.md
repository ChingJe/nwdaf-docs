# Hierarchical NWDAF FL Protocol Extension Implementation Plans

日期：2026-09-02

狀態：Ready for User Review；第一輪實作盤點與 slice map 已完成，model-free
preparation、topology-ready 後的 ADRF distribution boundary 與 retained-result
validation 已補入實作要求，production implementation 尚未開始

## 文件定位

本分類管理 hierarchical NWDAF FL protocol extension 的主計畫、現有實作盤點、
後續詳細計畫、review ledger 與驗證紀錄。

它和上一版 [Hierarchical NWDAF Federated Learning Implementation Plan](../hierarchical-fl-model-bundle-edition/Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)
分開維護。舊計畫記錄目前已跑通的 model-bundle／static orchestration
implementation；本分類則負責將已確認的 topology、policy、strategy、Notify、
feature negotiation 與 retained-result semantics 實作到正式
`Nnwdaf_MLModelTraining` message flow。

## 主計畫

- [Hierarchical NWDAF FL Protocol Extension Implementation Plan](./Hierarchical%20NWDAF%20FL%20Protocol%20Extension%20Implementation%20Plan.md)

## 實作盤點

- [Protocol Implementation Current-State Inventory](./Protocol%20Implementation%20Current-State%20Inventory.md)：
  記錄 repository revisions、現有 production flow、component ownership、目標差距
  與待確認事項。
- [Model Bundle Metadata to Protocol Schema Mapping](./Model%20Bundle%20Metadata%20to%20Protocol%20Schema%20Mapping.md)：
  逐欄位確認舊 hierarchy bundle metadata 應搬到 protocol、保留在 artifact、改為
  local state 或移除，並記錄 producer、transport、state owner、consumer 與
  runtime cutover boundary。
- [Protocol Resource Lifecycle and Wire Integration Mapping](./Protocol%20Resource%20Lifecycle%20and%20Wire%20Integration%20Mapping.md)：
  逐段確認 Create／PUT／PATCH／DELETE／Notify 的 field preservation、resource state、
  feature negotiation 與 operation-scoped command boundary。
- [Protocol Conformance Case Ownership Mapping](./Protocol%20Conformance%20Case%20Ownership%20Mapping.md)：
  將 candidate conformance cases 指派到 Go validation／route、PyMTLF resource／
  coordinator 與 real-process test seam。

## Slice map

- [Protocol Extension Implementation Slice Map](./Protocol%20Extension%20Implementation%20Slice%20Map.md)：
  依 wire、resource、execution owner與可獨立驗證邊界拆分 production work units。
- [Slice 詳細計畫](./slices/)：
  收錄接下來各 slice 的獨立實作計畫；每份計畫只處理一個可 review、驗證與交付的
  work unit。

## 設計輸入

- [Hierarchical NWDAF Federated Learning 協定設計](../../../design/hierarchical-federated-learning/protocol_design.md)
- [Topology、Policy 與 Strategy 細節設計](../../../design/hierarchical-federated-learning/topology_policy_design.md)
- [標準欄位與 Extension 邊界](../../../design/hierarchical-federated-learning/standard_field_extension_boundary.md)
- [Candidate OpenAPI Schema](../../../design/hierarchical-federated-learning/candidate_openapi_schema.md)
- [Candidate OpenAPI artifact](../../../design/hierarchical-federated-learning/candidate_openapi.yaml)
- [Protocol Conformance Matrix](../../../design/hierarchical-federated-learning/protocol_conformance_matrix.md)

## 後續文件慣例

- 主計畫只維護背景、目標、主要情境、範圍、高階工作拆分與整體狀態。
- Repository revision、code-level production flow、owner／transport mapping 與
  gap analysis 放在獨立盤點文件。
- 在 end-to-end data flow 與 owner 確認前，不預先固定 implementation slice。
- 後續若形成可獨立實作與審查的工作單位，在 `slices/` 建立對應 detailed plan。
- Implementation review 維持單一 phase ledger；不為每次修正建立新的完整 review
  文件。
- 各 repository 分開 review、commit 與驗證，不因其中一個 repository 通過就宣稱
  end-to-end 完成。
