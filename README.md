# nwdaf-docs

此倉庫用於保存 NWDAF 開發過程中的文件、3GPP 規格資料、設計筆記與進度紀錄。

主要內容：

- `docs/`：開發筆記、設計規劃、議題紀錄與進度整理
- `specs/`：依 3GPP release 隔離的 NWDAF 實作相關規格 corpus
  - `specs/README.md`：release 索引、收錄政策與目錄規則
  - `specs/Rel-18/`：22 份完整 Release 18 TS 與同 release OpenAPI 附件
  - `specs/Rel-19/`：Release 19 補充研究規格與同 release 附件
  - `specs/Rel-20/`：4 份 Release 20 TS／TR 參考規格
- `archive/`：歷史討論、舊版計畫與已歸檔文件

`specs/Rel-18/` 是目前使用中的 NWDAF 實作規格來源；其他 release 不應
視為 Release 18 實作需求。各 release 都不保證包含所有 3GPP 外部相依
schema。使用 OpenAPI YAML 前，應先檢查該 release 的 `openapi/README.md`；
尚未收錄的規格於實際需要時再補入相同 release 的目錄。

長期開發規範可參考：

- [`docs/development_policy.md`](./docs/development_policy.md)：開發規範入口，依目前任務導向適用的規則模組，沿用仍有效的驗證與審查結果。
- [`docs/spec_conversion.md`](./docs/spec_conversion.md)
