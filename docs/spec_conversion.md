# 3GPP 規格轉換指南

本文件說明如何將 3GPP Word 規格加入 `specs/`。目標是提供容易搜尋與
分段讀取的 Markdown，不建立額外的稽核或發布系統。

## 基本原則

- 每個 release 使用獨立目錄，例如 `Rel-18/`、`Rel-19/`、`Rel-20/`。
- 規格正文保留原文，不摘要、不翻譯，也不由 LLM 改寫。
- 官方 OpenAPI、ABNF、XML 與其他附件直接複製，不重新格式化。
- Git 負責內容差異與歷史；repository 不保存額外 checksum 或 build report。
- 原始 Word 或 3GPP 官方出版物仍是高風險解讀的最終依據。

## 目錄

```text
specs/
├── README.md
├── manifest.yaml
├── Rel-18/
│   ├── README.md
│   ├── manifest.yaml
│   ├── TS 23.288/
│   ├── TS 29.520/
│   └── openapi/
├── Rel-19/
└── Rel-20/
```

不同 release 的正文與 schema 不得混在同一目錄。跨 release 比較必須明確
標示兩邊版本，不能把另一個 release 的同名 schema 當成本地相依。

## 轉換流程

1. 從文件封面或頁首確認 TS 編號、版本與 release。
2. 記錄來源 ZIP 與 Word 檔名。
3. 原生 DOCX 直接使用 Pandoc；legacy DOC 先以 LibreOffice 轉成 DOCX。
4. 使用 Pandoc 轉成 GFM，並抽出圖片與 embedded objects。
5. 只修正可由來源確認的 heading、圖片路徑與導覽結構。
6. 依 clause hierarchy 拆檔，建立必要的 README 導覽。
7. 將官方附件直接複製到同一 release 的 `openapi/` 或規格附件目錄。
8. 更新 release README、release manifest 與 corpus release index。
9. 執行基本驗證並人工抽查高風險內容。

建議的 Pandoc 命令：

```bash
pandoc source.docx \
  --from=docx \
  --to=gfm \
  --wrap=none \
  --extract-media=media \
  --output=spec.md
```

## 內容處理

允許的調整：

- 修正明確錯誤的 heading level。
- 移除沒有可見文字的空 heading。
- 正規化本地圖片與導覽連結。
- 為大型 clause 建立目錄與 README。

不得進行：

- 改寫句子、拼字或文法。
- 修改 `shall`、`should`、`may`、否定、條件或例外。
- 猜測表格 cell 或缺失文字。
- 由 Markdown 反向產生官方 OpenAPI YAML。

簡單表格可使用 Markdown table。包含 rowspan、colspan、多段落或巢狀內容的
表格保留 HTML，不為格式一致而重寫。

拆檔以 clause 邊界為主。單一大型表格或不可安全拆分的 clause 可以保留成
較大的檔案，不需要為了固定字數強制切割。

## 圖片與附件

- 保留原始圖片與 embedded OLE／Visio 檔案。
- 能可靠產生 PNG preview 時可以加入；失敗不阻擋規格收錄。
- 不建立空白 placeholder 假裝轉換成功。
- 官方 YAML、ABNF 與 XML 不排序、不重排縮排、不統一版本字串。
- OpenAPI 外部 `$ref` 只需在 `openapi/README.md` 說明；不要求補齊整個 5GC。

## Manifest

Corpus manifest 只索引 release。每個 release manifest 只保存規格識別資訊：

```yaml
release: '20'
specifications:
- spec: TS 23.288
  version: 20.x.0
  path: TS 23.288/
  source_archive: 23288-xxx.zip
```

不保存 checksum、build ID、工具版本、逐 clause word count 或重複的 clause
索引。Clause 導覽由規格目錄中的 README 提供。

## 驗證

提交前只需確認：

- release manifest 中的規格目錄都存在。
- README、Markdown 圖片與本地連結沒有斷鏈。
- 收錄的 YAML 可以解析。
- OpenAPI `$ref` 不會被錯誤地跨 release 解析。
- 抽查複雜表格、Annex、圖片、text box 與曾修正的 heading。
- 自行產生或編輯的 Markdown、索引與 manifest 通過 `git diff --check`。

格式檢查須排除依原樣保留要求收錄的官方附件。官方 YAML、ABNF 或 XML 原有的
CRLF、行尾空白與縮排不視為轉換缺陷，不為消除格式診斷而修改。附件仍須完成
適用的解析及來源內容保留檢查。

檢查輸出只列出新增、可處理的問題；已確認的來源格式特性或已記錄的限制以簡短
說明帶過，避免重複列出大量已知診斷。沿用仍適用的驗證結果，重跑條件依
[Development Policy](development_policy.md#evidence-applicability)。

不產生永久的 validation JSON、command log、environment snapshot、文字相似度
報告、視覺分級或 hash 清單。若某份規格仍有實際閱讀限制，直接記在該 release
或規格 README 中。

## 完成條件

新規格可以透過 release README 找到，正文與附件位於正確 release，基本連結與
YAML 檢查通過，而且高風險內容已抽查，即可交付 review。
