# Hierarchical NWDAF Federated Learning 設計文件

本目錄整理 hierarchical NWDAF Federated Learning 的 protocol／schema
設計。新設計以已確認的 3GPP mechanism、現行 implementation facts 與會議後
提出的問題為基礎；既有 proposal 只作為歷史輸入。

## 主要文件

- [Hierarchical NWDAF Federated Learning 協定設計](./protocol_design.md)：
  本系列的主要設計文件，涵蓋設計目標、標準邊界、核心原則與各議題狀態。

## 細節設計

- [Topology、policy 與 strategy 細節設計](./topology_policy_design.md)：
  整理 recursive topology、node-level policy、training／aggregation strategy、
  node-level execution instruction、Client selection、candidate priority、
  participant threshold、status lifecycle 與 retained-result lookup behavior。
- [標準欄位與 Extension 邊界](./standard_field_extension_boundary.md)：
  對照 Release 18 至 Release 20 的既有 Model Training fields，區分可直接重用
  的資訊、hierarchical orchestration gap 與 proposed extension 的 minimum
  scope；不承載完整 procedure 或 HTTP examples。
- [Candidate OpenAPI Schema](./candidate_openapi_schema.md)：
  將已確認的 topology、policy、strategy、status 與 retained-result 語意映射
  至 Subscription、PATCH 與 Notify，並提供 candidate OpenAPI YAML、跨欄位
  validation rules、`suppFeats` negotiation、error mapping 與 HTTP message
  examples。

## 設計情境

- [Branch 故障替換情境](./branch_replacement_scenario.md)：
  記錄三區域 deployment 中 active Branch 失效、Root 選擇替代 Branch、重新
  建立 Leaf subscriptions，以及取回 Leaves 已保留結果的流程與責任邊界。

## 相關資料

- [NWDAF Federated Learning Release 18 規格解讀](../../specification-guides/NWDAF%20Federated%20Learning%20Release%2018%20規格解讀.md)
- [Hierarchical NWDAF Federated Learning Implementation Plan](../../plans/hierarchical-federated-learning/Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)
- [既有 protocol/schema feasibility proposal](../../proposals/nwdaf/hierarchical-federated-learning/protocol_schema_feasibility.md)：
  歷史 proposal，並非新設計的主要依據。
