# Protocol Extension Slice 詳細計畫

日期：2026-09-03

## 文件定位

本分類收錄 hierarchical NWDAF FL protocol extension 各 implementation slice 的
獨立詳細計畫。

上層 [Protocol Extension Implementation Slice Map](../Protocol%20Extension%20Implementation%20Slice%20Map.md)
負責維護整體拆分、slice 邊界、依賴與執行順序；本分類內的文件則分別記錄單一 slice
的 exact scope、affected repositories、contract、implementation steps、acceptance tests、
verification 與 deferred work。

## Slice 計畫

- [Slice 1 — Wire Contract and Resource Lifecycle Foundation](./Slice%201%20Wire%20Contract%20and%20Resource%20Lifecycle%20Foundation%20Detailed%20Plan.md)：
  建立 Go／PyMTLF typed candidate contract、receiver validation、persistent／
  operation-scoped state 分離、CRUD atomicity 與 per-resource feature state。

## 文件慣例

- 每個 slice 使用一份獨立 detailed plan，不把所有實作細節塞回上層主計畫或 slice
  map。
- 一份計畫只涵蓋一個可獨立 review、驗證與交付的 work unit。
- Slice plan 必須沿用上層 slice map 已確認的邊界；若需要改變整體拆分，先更新上層
  slice map，再同步調整個別計畫。
- Review、commit 與驗證仍依 repository boundary 分開處理。
