# nwdaf-docs

此倉庫用於保存 NWDAF 開發過程中的文件、3GPP 規格資料、設計筆記與進度紀錄。

主要內容：

- `docs/`：開發筆記、設計規劃、議題紀錄與進度整理
- `specs/`：NWDAF 實作相關的完整 3GPP Release 18 規格 corpus
  - `specs/README.md`：規格版本、收錄範圍與建議閱讀順序
  - `specs/TS .../`：十份完整 TS 的語意章節、文字與圖表
  - `specs/openapi/`：隨收錄套件提供的原始 OpenAPI YAML 附件
  - `specs/_validation/`：轉換流程、checksum、驗證結果與已知限制
- `archive/`：歷史討論、舊版計畫與已歸檔文件

`specs/` 是目前使用中的 NWDAF 實作相關 Release 18 規格來源，但不是所有
3GPP 外部相依 schema 的完整集合。使用 OpenAPI YAML 前，應先檢查
[`specs/openapi/README.md`](./specs/openapi/README.md) 列出的缺少相依項目；尚未
收錄的規格於實際需要時再補入相同 Release 的來源。

長期開發規範可參考：

- [`docs/development_policy.md`](./docs/development_policy.md)
