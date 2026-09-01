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
  Client selection、candidate priority、participant threshold 與 status
  lifecycle。

## 相關資料

- [NWDAF Federated Learning Release 18 規格解讀](../../specification-guides/NWDAF%20Federated%20Learning%20Release%2018%20規格解讀.md)
- [Hierarchical NWDAF Federated Learning Implementation Plan](../../plans/hierarchical-federated-learning/Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)
- [既有 protocol/schema feasibility proposal](../../proposals/nwdaf/hierarchical-federated-learning/protocol_schema_feasibility.md)：
  歷史 proposal，並非新設計的主要依據。
