# Protocol Extension Slice 詳細計畫

日期：2026-09-07

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
- [Slice 2 — Candidate Pool, Policy and Local Contract Execution](./Slice%202%20Candidate%20Pool%20Policy%20and%20Local%20Contract%20Execution%20Detailed%20Plan.md)：
  建立 PyMTLF candidate pool、delegated discovery、policy／strategy／`reportAfter`
  local execution、selected-set aggregation gate 與 realized topology snapshot。
- [Slice 3 — Branch Replacement without Retained-result Recovery](./Slice%203%20Branch%20Replacement%20without%20Retained-result%20Recovery%20Detailed%20Plan.md)：
  以`branch_groups -> branches／leaves`保存不重複的static candidate assignment，並讓
  Root選Branch與Branch選Leaf共用direct-child priority／policy semantics；主要驗證單一
  Branch於training途中失效後的replacement，不取回舊結果。
- [Slice 4A — Digest Simplification and Contract Cleanup](./Slice%204A%20Digest%20Simplification%20and%20Contract%20Cleanup%20Detailed%20Plan.md)：
  只保留完整artifact的content-addressed SHA-256 key，移除其他bundle、model、training
  evidence、Notify、topology與collection content digests，並以explicit identity及state
  維持必要流程。
- [Slice 4 — Controlled Local Training Workload](./Slice%204%20Controlled%20Local%20Training%20Workload%20Detailed%20Plan.md)：
  建立known workload／data-source boundary、MNIST／CIFAR-10 local loader、classification
  training、profile-aware artifacts與held-out evaluation；大量部署使用共同config與
  per-Client mount，同時保留標準`UE_COMMUNICATION + consumer_subscription`流程。
- [Slice 5 — Protocol-driven Hierarchy Integration](./Slice%205%20Protocol-driven%20Hierarchy%20Integration%20Detailed%20Plan.md)：
  將candidate contract接上真實Root→Branch→Leaf subscription／Notify flow，完成
  model-free preparation、逐edge feature negotiation、Root global-model ADRF lifecycle
  與Branch local-round artifact transport，並重用Slice 4 controlled workload。
- [Slice 6 — Migration and Regression Closure](./Slice%206%20Migration%20and%20Regression%20Closure%20Detailed%20Plan.md)：
  將hierarchical runtime收斂為單一protocol authority，移除assignment／preparation-result
  bundle schema、producer、consumer與legacy deployment scenario，同時保留flat／distributed
  FL及hierarchical model／result artifact regression。

Retained-result runtime仍維持暫緩；Slice 3編號已重新用於不使用retained result的
Branch replacement。Slice 1、2、3、4A、4、5與6均已完成審查、驗證與commit；
正式multi-host testbed尚未進入。

## 文件慣例

- 每個 slice 使用一份獨立 detailed plan，不把所有實作細節塞回上層主計畫或 slice
  map。
- 一份計畫只涵蓋一個可獨立 review、驗證與交付的 work unit。
- Slice plan 必須沿用上層 slice map 已確認的邊界；若需要改變整體拆分，先更新上層
  slice map，再同步調整個別計畫。
- Review、commit 與驗證仍依 repository boundary 分開處理。
