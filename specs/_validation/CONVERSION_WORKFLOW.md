# 3GPP Word 規格轉換為 Agent-readable Markdown 的可複現流程

**版本：2.0**  
**更新日期：2026-08-03**

> 適用案例：將 3GPP `.docx`、legacy `.doc` 與其官方 OpenAPI／ABNF／圖資附件，轉換成可由 coding agent 分層查閱的 `specs/` corpus。  
> 核心要求是保留規格文字精確度、結構語意、附件原貌與完整可追溯性，而不是讓 LLM 重寫規格。
>
> 本流程已在 21 份 Release 18 規格、原生 DOCX、legacy DOC、數千個 clause、數百張 EMF／WMF／Visio 圖與數十份 OpenAPI YAML 的多輪增量整合中實際使用與修正。

---

## 0. 核心結論

整個流程可濃縮為：

```text
官方 ZIP
    ↓
來源盤點、版本確認、SHA-256
    ↓
依來源格式選擇 DOCX / legacy DOC adapter
    ↓
Pandoc 結構抽取 + 獨立文字抽取
    ↓
deterministic correction
    ↓
heading tree + reassembly hash
    ↓
progressive-disclosure 拆檔
    ↓
原始附件、rendered preview、embedded objects
    ↓
README、manifest、OpenAPI dependency inventory
    ↓
文字、結構、連結、附件與視覺驗證
    ↓
clean-build ZIP + SHA-256
```

最重要的邊界：

```text
規格正文
    只能由 deterministic extraction 產生

OpenAPI / ABNF / XML 等官方附件
    必須逐位元組保存

LLM
    只能協助檢查、導航與異常定位
    不得改寫、補寫或解釋後重新輸出 normative text
```

---

# 1. 目標、非目標與信任邊界

## 1.1 目標

輸出 corpus 應滿足：

1. 規格正文不摘要、不翻譯、不潤飾。
2. `shall`、`shall not`、`should`、`may`、條件、否定與例外不得被改寫。
3. 表格 cell、NOTE、EXAMPLE、figure caption 與 clause 編號不得被語意重組。
4. 每個檔案可追溯到來源 ZIP、來源 Word、版本與 SHA-256。
5. 大型規格透過 progressive disclosure 分層，不要求 agent 一次載入全文件。
6. OpenAPI、ABNF 與其他官方附件原封不動保存。
7. EMF、WMF、OLE、Visio 等原始物件保留，同時盡量產生 PNG preview。
8. 產生 machine-readable manifest、dependency inventory 與 validation report。
9. 每次交付可由 clean build 重現。
10. 不對 GitHub repository 做任何寫入，除非使用者另行明確要求。

## 1.2 非目標

本流程不保證：

- Markdown 與 Word 的排版視覺完全一致。
- 保留 Word 的字型、頁邊距、分頁、頁首頁尾與浮動 anchor。
- 自動解釋規格意義。
- 自動補齊整個 5GC 的所有跨規格依賴。
- 將所有複雜表格轉成 pipe table。
- 每份大型規格都能順利全文 export PDF。
- 未經原始來源確認，自動修正疑似錯字或文法。

## 1.3 信任順序

重要規範判定時，證據優先順序應為：

```text
原始 3GPP Word / 官方出版物
    ↓
同一 package 的官方 OpenAPI / ABNF / XML 附件
    ↓
本流程產生的 Markdown
    ↓
repository-generated README / manifest / topic metadata
    ↓
LLM 產生的解釋
```

Markdown 是檢索與閱讀介面，不是 3GPP 官方出版格式的替代品。

---

# 2. 輸出 corpus 架構

推薦結構：

```text
specs/
├── README.md
├── manifest.yaml
├── TS 23.288/
│   ├── README.md
│   ├── manifest.yaml
│   ├── clause files/directories
│   └── assets/
│       ├── original/
│       ├── rendered/
│       └── embedded/
├── TS 29.520/
│   ├── README.md
│   ├── manifest.yaml
│   ├── clause files/directories
│   └── assets/
├── ...
├── openapi/
│   ├── README.md
│   ├── manifest.yaml
│   └── official YAML files
├── attachments/
│   └── optional shared official attachments
└── _validation/
    ├── README.md
    ├── build.json
    ├── environment.json
    ├── source-inventory.json
    ├── commands.log
    ├── SHA256SUMS.txt
    ├── report.json
    ├── final-checks.json
    ├── text-cross-check.json
    ├── visual-sampling.json
    ├── dependency-report.json
    ├── deterministic-corrections.yaml
    └── CONVERSION_WORKFLOW.md
```

導覽模式：

```text
specs/README.md
    ↓
TS xx.xxx/README.md
    ↓
大型 clause/README.md
    ↓
atomic clause Markdown
```

各層責任：

| 層級 | 責任 |
|---|---|
| `specs/README.md` | 整體規格地圖、常見開發路徑、信任邊界 |
| `TS xx.xxx/README.md` | 該規格用途、版本、章節地圖、相關規格 |
| clause directory `README.md` | immediate children 導航 |
| clause Markdown | 規格原文 |
| `manifest.yaml` | 精確機器定位 |
| `_validation/` | 來源、工具、修正、驗證與限制 |

---

# 3. Build model：每次都要 clean build

## 3.1 禁止不可追蹤的就地累積修改

多輪增量加入規格時，最安全方式是：

```text
上一版已驗證 ZIP
    ↓ 解壓成唯讀 parent corpus
建立新的空 output directory
    ↓
複製 parent corpus
    ↓
加入新規格
    ↓
重新生成所有受影響的全域檔案
    ↓
重新跑全 corpus 驗證
    ↓
產生新的 ZIP
```

不應直接在上一版 output directory 中持續手動修補，否則容易產生：

- 重複 correction。
- 舊 manifest 殘留。
- orphan files。
- dependency inventory 不同步。
- validation report 混用不同 build 的結果。
- 已刪規格仍留在全域索引。

## 3.2 Build identity

每次 build 應產生唯一身份：

```yaml
build:
  schema_version: 1
  build_id: r18-corpus-20260803-001
  created_at: "2026-08-03T00:00:00+08:00"
  parent_archive:
    file: previous-corpus.zip
    sha256: ...
  output_archive:
    file: new-corpus.zip
    sha256: ...
```

## 3.3 Source provenance

```yaml
source_archives:
  - file: 23288-id0.zip
    sha256: ...
    spec: TS 23.288
    version: 18.13.0
    release: 18
    source_document: 23288-id0.docx
    source_format: docx

  - file: 29520-ie0.zip
    sha256: ...
    spec: TS 29.520
    version: 18.14.0
    release: 18
    source_document: 29520-ie0.doc
    source_format: doc
```

每個 build 都要記錄實際加入的所有 ZIP，不只列最新一批。

---

# 4. 工具鏈與環境固定

## 4.1 主要工具

| 工具 | 用途 |
|---|---|
| Python | tree parsing、拆檔、manifest、驗證、封裝 |
| Pandoc | DOCX → Markdown、media extraction |
| LibreOffice | legacy DOC → DOCX/TXT/PDF |
| `docx2txt` | DOCX 獨立全文抽取 |
| `antiword` | legacy DOC 獨立全文抽取 |
| Inkscape | EMF／WMF → PNG |
| ImageMagick | 圖片轉換 fallback |
| Poppler `pdftoppm` | PDF page → PNG |
| PyYAML | YAML parse 與 manifest |
| `zip` / `unzip` | 解包與交付 |
| SHA-256 | 來源與輸出完整性 |

## 4.2 必須記錄實際版本

```json
{
  "python": "3.13.5",
  "pandoc": "3.1.11.1",
  "libreoffice": "25.2.3.2",
  "inkscape": "1.4",
  "imagemagick": "7.1.2-1",
  "poppler": "25.06.0",
  "pyyaml": "6.0.3"
}
```

工具升級可能改變：

- heading style mapping。
- HTML table 輸出。
- 圖片 relationship。
- DOC → DOCX 結果。
- soft hyphen 與 spacing。
- 圖片轉換成功率。

## 4.3 建議容器化

正式化後應以 Dockerfile 或 devcontainer 固定版本。至少提供：

```text
Dockerfile
requirements.txt
tool-versions.txt
```

---

# 5. 工作目錄

建議：

```text
work/
├── parent/
├── source/
│   ├── TS_23.288/
│   └── TS_29.520/
├── intermediate/
│   ├── docx/
│   ├── markdown/
│   ├── media/
│   └── plain-text/
├── rendered-source/
├── qa/
├── logs/
├── build/
│   └── specs/
└── scripts/
```

輸出只能寫入新的 `work/build/specs/`。

來源與 parent corpus 應視為唯讀。

---

# 6. Phase 0：來源盤點

## 6.1 ZIP inventory

```bash
unzip -l SPEC.zip
sha256sum SPEC.zip
```

檢查：

- Word 是 `.docx` 還是 `.doc`。
- 是否有 YAML、ABNF、XML、JSON 或其他附件。
- 是否有重複 package。
- 文件頁首版本是否與 ZIP filename code 一致。
- 附件的 `externalDocs` 版本是否較早。
- ZIP 是否另含多層 ZIP。

## 6.2 版本確認順序

外部 archive index 可能落後。以以下順序判斷：

1. 使用者提供的 ZIP。
2. Word 文件內頁首／封面。
3. Word properties。
4. 附件內 `externalDocs`。
5. 3GPP／ETSI archive 頁面。

package version 與附件 declared version 要分開記錄，不可強制統一。

## 6.3 Source inventory

產生：

```json
{
  "archive": "29520-ie0.zip",
  "archive_sha256": "...",
  "members": [
    {
      "path": "29520-ie0.doc",
      "size": 12345678,
      "sha256": "..."
    },
    {
      "path": "TS29520_Nnwdaf_EventsSubscription.yaml",
      "size": 123456,
      "sha256": "..."
    }
  ]
}
```

---

# 7. Phase 1：依來源格式選擇 adapter

原生 DOCX 與 legacy DOC 必須視為兩種 pipeline。

## 7.1 Native DOCX adapter

```text
DOCX
├── Pandoc → raw Markdown
├── docx2txt → independent plain text
├── OOXML direct inspection
├── word/media extraction
└── word/embeddings extraction
```

DOCX 是 ZIP-based OOXML，可直接檢查：

```text
word/document.xml
word/styles.xml
word/numbering.xml
word/_rels/document.xml.rels
word/media/*
word/embeddings/*
```

## 7.2 Legacy DOC adapter

```text
原始 DOC
├── antiword → independent text evidence
├── LibreOffice TXT → second text evidence
├── LibreOffice DOCX → structural intermediate
├── Pandoc intermediate DOCX → Markdown
└── 原始 DOC → PDF，若可行
```

命令：

```bash
libreoffice \
  --headless \
  --convert-to docx \
  --outdir work/intermediate/docx \
  source.doc
```

重要：

```text
intermediate DOCX 不是 source of truth。
```

原始 DOC 必須保留，因為 DOC → DOCX 可能改變：

- heading styles。
- text box 順序。
- merged-cell 表示。
- floating objects。
- list numbering。
- 字詞的 layout-driven 分割。

---

# 8. Phase 2：Pandoc 結構抽取

## 8.1 建議命令

```bash
pandoc \
  source.docx \
  --from=docx \
  --to=gfm \
  --wrap=none \
  --extract-media=work/intermediate/media/SPEC \
  --output=work/intermediate/markdown/SPEC.md
```

legacy DOC 使用中介 DOCX。

## 8.2 保存命令紀錄

每個 subprocess 都應寫入 `commands.log`：

```json
{
  "command": ["pandoc", "..."],
  "started_at": "...",
  "ended_at": "...",
  "exit_code": 0,
  "timeout_seconds": 1800,
  "stdout_log": "...",
  "stderr_log": "...",
  "inputs": {"source.docx": "sha256"},
  "outputs": {"SPEC.md": "sha256"}
}
```

不要只在最終文件中寫「曾經執行 Pandoc」，要保留 exact command。

## 8.3 複雜表格

GFM 無法完整表示：

- rowspan。
- colspan。
- nested table。
- cell 內多段落。
- cell 內 list 或圖片。

原則：

```text
簡單表格
→ Markdown pipe table

複雜／合併表格
→ 保留 Pandoc HTML table
```

不可為了「純 Markdown」強制重寫 HTML table。

---

# 9. Phase 3：獨立文字抽取

## 9.1 DOCX

```bash
docx2txt source.docx > independent.txt
pandoc raw.md --to=plain --output=pandoc.txt
```

也可直接解析 `word/document.xml` 作第三路徑。

## 9.2 Legacy DOC

```bash
antiword source.doc > antiword.txt

libreoffice \
  --headless \
  --convert-to txt:Text \
  --outdir work/intermediate/plain-text \
  source.doc
```

## 9.3 Normalization

不同 extractor 會出現：

- soft hyphen。
- non-breaking space。
- header/footer。
- table serialization。
- TOC。
- hidden fields。
- `Cardinality` → `Cardinal ity`。

建議 normalization：

```python
text = text.replace("\u00ad", "")
text = text.replace("\u00a0", " ")
text = unicodedata.normalize("NFKC", text)
tokens = re.findall(
    r"[A-Za-z0-9]+(?:[-'][A-Za-z0-9]+)*",
    text.lower(),
)
```

## 9.4 比對指標

至少計算：

1. normalized token count。
2. token multiset overlap。
3. 12-token shingle overlap。
4. normative marker count。
5. NOTE／EXAMPLE count。
6. 章節標題集合。
7. 表格／圖片／OLE 數量。

Normative markers：

```text
shall
shall not
should
should not
may
may not
must
must not
NOTE
EXAMPLE
```

---

# 10. Legacy DOC 差異分類

legacy DOC 不可只用「是否完全相同」判斷成功。

差異需分類：

| 類型 | 處理 |
|---|---|
| 正文 token 缺失或新增 | Fail，必須回查來源 |
| normative marker 差異 | 高風險，逐 block 回查 |
| 表格 cell serialization 差異 | 回查來源表格，可接受但需記錄 |
| header/footer 差異 | 通常可接受 |
| soft hyphen / spacing | 正規化後可接受 |
| list numbering 變化 | 回查結構 |
| text box 順序差異 | 高風險，視覺或 OOXML 檢查 |
| TOC 重複／缺失 | 通常不影響正文 |

範例：

```yaml
legacy_doc_differences:
  - block: table-87
    category: table-serialization
    antiword_missing_marker: shall
    source_page_reviewed: true
    accepted: true
    note: antiword omitted one merged cell; Pandoc output matches Word table
```

不能因 `antiword` 少一個 `shall` 就直接宣稱 Pandoc 錯誤，也不能直接忽略。

---

# 11. Phase 4：Deterministic correction

## 11.1 只修結構，不修句子

允許：

- Annex heading level correction。
- 空白 heading 移除。
- 遺失 heading style 的恢復。
- TOC link 轉成 visible text。
- figure path normalization。
- 切檔後的 local navigation link 更新。

禁止：

- 文法修正。
- 拼字修正。
- 句子重寫。
- 表格內容猜測。
- normative wording 修改。

## 11.2 Annex 通用規則

不能只硬編碼 Annex A。

通用規則：

```text
Annex [A-Z] 主標題
→ root-level specification child

[A-Z].1
→ Annex 的第一層子節

[A-Z].1.1
→ 下一層
```

需考慮：

- Annex A～K 或更多。
- Annex heading 被轉成 H8。
- Annex 子節被轉成 H1。
- change history 沒有一般 clause 子節。
- 某一個 Annex heading style 遺失。

## 11.3 空白 heading

只在確認 source 沒有 visible text 時移除。

不得單憑：

```markdown
#
```

就刪除前後內容。

## 11.4 Per-spec correction registry

```yaml
corrections:
  - spec: TS 29.520
    source_line: 40525
    type: restore-heading
    original: A.9 Nnwdaf_RoamingAnalytics API
    replacement: "## A.9 Nnwdaf_RoamingAnalytics API"
    evidence:
      source_page: 432
      source_style: lost-during-doc-conversion
```

每次 clean build 都從 registry 重新套用，不直接在輸出檔手改。

---

# 12. Phase 5：建立 heading tree

## 12.1 Node 結構

```python
@dataclass
class Node:
    level: int
    title: str
    clause: str | None
    content: list[str]
    children: list["Node"]
    parent: "Node | None"
    word_count: int
    subtree_words: int
    source_start_line: int
    source_end_line: int
```

Clause regex 需支援：

```text
4
4.2
6.2A
6.2C.2.1
A.1
Annex A
```

範例：

```python
r"^((?:\d+[A-Z]?|[A-Z])(?:\.\d+[A-Z]?)*)(?:\s+|$)"
```

## 12.2 Reassembly hash

tree 建立後，必須重新組回原 Markdown：

```text
preprocessed Markdown SHA-256
tree-reassembled Markdown SHA-256
```

兩者必須相同。

這證明：

- 沒漏段落。
- 沒改順序。
- 沒因 parser tree 丟失表格或圖片。
- 拆檔前資料完整。

若 hash 不同，不可繼續拆檔。

---

# 13. Phase 6：Progressive-disclosure 拆檔

## 13.1 預設門檻

```python
MAX_WORDS = 4200
```

這不是絕對限制，而是導航門檻。

## 13.2 Leaf file

subtree 小於門檻：

```text
6.2A Procedure for ML Model Provisioning.md
```

可內含較小的子 clause。

## 13.3 Directory

subtree 超過門檻且有子節：

```text
6 Procedures/
├── README.md
├── 6.1 ...md
└── 6.2 .../
    ├── README.md
    └── ...
```

Directory README 只列 immediate children，不列所有 descendants。

## 13.4 父 clause 的前置正文

若父 clause 在第一個子節前有正文：

```markdown
## Specification text before child clauses
```

此 heading 是 generated navigation，必須在 front matter 標記。

## 13.5 Oversized atomic clause

若沒有安全語意邊界，不可任意切：

- 單一大型 data type table。
- 單一 service operation。
- 複雜 merged table。
- 大型 API schema table。

保留完整並列入：

```json
{
  "oversized_atomic_files": [
    {
      "spec": "TS 29.xxx",
      "clause": "6.1.6.3",
      "words": 8200,
      "reason": "single complex normative table"
    }
  ]
}
```

## 13.6 Change history

Change history 常是單一大型 HTML table。

可依年度拆分：

```text
Annex Change history/
├── README.md
├── 2022.md
├── 2023.md
└── ...
```

規則：

- table header 可重複。
- row 內容不改寫。
- continuation row 不拆錯年度。
- README 說明這是 loading optimization，不是新增規範章節。

---

# 14. Front matter 與 manifest

## 14.1 Clause front matter

```yaml
---
spec: TS 23.288
version: 18.13.0
release: "18"
clause: 6.2A
title: Procedure for ML Model Provisioning
source_archive: 23288-id0.zip
source_document: 23288-id0.docx
source_archive_sha256: ...
source_document_sha256: ...
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
build_id: r18-corpus-20260803-001
---
```

Generated README：

```yaml
content_origin: generated-navigation-and-3gpp-source
```

## 14.2 Per-spec manifest

每個 heading 都要登錄，即使沒有獨立檔案：

```yaml
clauses:
  - clause: "6.2A"
    title: Procedure for ML Model Provisioning
    path: "6 Procedures/.../6.2A Procedure for ML Model Provisioning.md"
    kind: source-file
    source_words: 2340

  - clause: "6.2A.2"
    title: Procedure
    path: "6 Procedures/.../6.2A Procedure for ML Model Provisioning.md"
    kind: embedded-clause
    source_words: 1810
```

## 14.3 全域 manifest

```yaml
specifications:
  - spec: TS 23.288
    version: 18.13.0
    path: TS 23.288/
    complete: true
    source_archive: 23288-id0.zip

  - spec: TS 23.502
    version: 18.14.0
    path: TS 23.502/
    complete: true
```

需雙向驗證：

```text
manifest 中的規格
↔ 實體規格目錄
```

---

# 15. 圖片、Visio 與 OLE

## 15.1 資產分類

```text
assets/
├── original/
├── rendered/
└── embedded/
```

定義：

```text
original
    source of truth

embedded
    可編輯／原生 OLE、Visio、embedded Word

rendered
    agent-friendly preview，不是來源真相
```

## 15.2 Embedded extraction

```python
with zipfile.ZipFile(source_docx) as z:
    for name in z.namelist():
        if name.startswith("word/embeddings/"):
            ...
```

保留：

- `.vsd`
- `.vsdx`
- `.doc`
- `.docx`
- `oleObject*.bin`

## 15.3 Preview rendering

優先順序：

1. PNG/JPEG 直接 copy。
2. Inkscape 處理 EMF／WMF。
3. 將可疑 WMF 暫時複製為 `.emf` 再試。
4. ImageMagick fallback。
5. 失敗時不建立假的空白 PNG。

```bash
inkscape source.emf \
  --export-type=png \
  --export-filename=target.png \
  --export-dpi=150
```

Fallback：

```bash
magick -density 180 source.emf -trim +repage target.png
```

## 15.4 必備穩定性措施

```text
per-image timeout
bounded concurrency
resume existing non-empty output
temporary extension override
failure manifest
```

推薦：

```text
Inkscape timeout：20 秒
ImageMagick timeout：30 秒
workers：最多 4
```

## 15.5 Preview failure policy

PNG preview 失敗不一定使 corpus fail，只要：

- 原始 vector 存在。
- 引用指向實際存在的原始或 rendered asset。
- 失敗清單記錄。
- 沒有 placeholder 假裝成功。

---

# 16. 官方 OpenAPI、ABNF 與其他附件

## 16.1 Byte-for-byte copy

```python
shutil.copy2(source, destination)
assert sha256(source) == sha256(destination)
```

禁止：

- 自動格式化。
- 排序 key。
- 改縮排。
- 統一版本字串。
- 修正 schema。
- 由 Annex Markdown 反向生成 YAML。

## 16.2 Package version 與附件 declared version

```yaml
package:
  spec: TS 29.518
  version: 18.14.0

attachments:
  - file: TS29518_Namf_EventExposure.yaml
    declared_spec_version: 18.13.0
    sha256: ...
    byte_identical_to_source: true
```

附件較早版本通常代表自前一 point release 後未修改，不是錯誤。

## 16.3 Annex API handling

若 Word Annex 重複提供完整 API transcription，而 ZIP 有官方 YAML：

- 正文可保留 Annex 一般說明。
- API body 不需再建第二份手動維護的 YAML。
- Markdown 應導向 exact official attachment。

---

# 17. OpenAPI dependency policy

## 17.1 不追求無限制歸零

OpenAPI dependency 會跨：

```text
NWDAF
→ DCCF / MFAF / ADRF
→ AMF / UDM / SMF / PCF / NEF / LMF / NSSF
→ policy / charging / location / security schemas
```

若以「任何 unresolved `$ref` 都要補」為目標，corpus 會無限擴張。

## 17.2 依賴分級

```yaml
dependencies:
  required_for_current_project:
    - TS29510_Nnrf_NFManagement.yaml

  required_for_included_primary_apis:
    - TS29571_CommonData.yaml

  optional_feature_dependencies:
    - TS29572_Nlmf_Location.yaml

  unresolved_external_dependencies:
    - TS32291_Nchf_ConvergedCharging.yaml
```

## 17.3 Closure 驗收標準

問題不是：

> 是否完全沒有 unresolved `$ref`？

而是：

> 目前專案會實際 bundle／generate／呼叫的 API 是否閉合？

例如：

```text
NWDAF → NRF → SMF / UPF / ADRF
    應閉合

尚未實作的 LMF / NEF / NSSF feature
    可保留 unresolved
```

## 17.4 Dependency scan

```python
pattern = r"\$ref:\s*['\"]?([^'\"\s]+)"
```

分類：

- internal `#/...`
- local external file
- missing external file
- remote URL
- circular reference

產生：

```text
openapi/manifest.yaml
_validation/dependency-report.json
```

---

# 18. README 與 agent navigation

## 18.1 根 README

應包含：

- 規格列表。
- 版本與 Release。
- Stage 2／Stage 3／OpenAPI 關係。
- 常見開發路徑。
- dependency 與 source integrity 說明。
- 如何引用 clause。

範例：

```text
NWDAF registration
TS 23.288 → TS 23.502 → TS 29.510 → OpenAPI

SMF event exposure
TS 23.502 → TS 29.508 → OpenAPI

ADRF
TS 23.288 / TS 23.501 → TS 29.575 → OpenAPI
```

## 18.2 Per-spec README

應包含：

- 規格定位。
- 何時讀取。
- clause map。
- 相關規格。
- 完整／scoped 狀態。
- OpenAPI 附件入口。

## 18.3 Navigation 與正文隔離

README 可包含 generated 說明，但必須明確標記：

```text
Generated navigation is not 3GPP normative text.
```

---

# 19. Markdown-aware validation

後續經驗證明，簡單 regex 會誤判：

- 規格中的 regex。
- inline code。
- code fence。
- 範例路徑。
- validation 文件中的 H8 範例。
- `(foo/bar)` 類型文字。

## 19.1 不要用單一 regex 掃全部

優先使用：

- Markdown AST。
- HTML parser。
- fenced-code-aware scanner。

至少要先排除：

```text
fenced code blocks
inline code
HTML comments
literal examples
```

## 19.2 驗證範圍分類

不同檔案類型使用不同規則：

| 檔案 | 驗證 |
|---|---|
| clause Markdown | heading、link、image、source text |
| README | link、navigation、generated metadata |
| validation docs | 不以範例文字觸發 corpus failure |
| YAML | YAML parse、`$ref` scan、byte identity |
| ABNF | byte identity |
| code fence | 不做 Markdown link／heading 判斷 |

## 19.3 Absolute path 檢查

應檢查輸出是否意外包含：

```text
/mnt/data/
/tmp/
C:\Users\
```

但要排除流程文件中刻意展示的 code examples。

---

# 20. 視覺驗證分級

大型文件不一定能成功全文 PDF export。不可把完整 PDF export 當硬性必要條件。

## 20.1 Level A：完整視覺驗證

- 原始 Word → PDF 成功。
- 逐頁或廣泛 contact sheet。
- 抽查所有高風險頁面。

## 20.2 Level B：Risk-based visual sampling

全文 export 過慢或失敗時：

- 選複雜表格頁。
- 選 sequence diagram。
- 選 Annex。
- 選 text box。
- 選 heading correction 頁。
- 選 OpenAPI resource figure。

## 20.3 Level C：結構與文字最低保證

至少完成：

- tree reassembly hash。
- independent text extraction。
- heading count。
- table count。
- image/OLE count。
- manifest verification。
- source asset preservation。

## 20.4 報告必須明確標示層級

```json
{
  "visual_validation": {
    "level": "B",
    "full_pdf_export": false,
    "sampled_pages": [30, 31, 102, 103],
    "reason": "LibreOffice full export exceeded timeout"
  }
}
```

不可籠統寫「視覺驗證已通過」而不說明範圍。

---

# 21. 自動驗證

## 21.1 必查項目

- local links。
- image links。
- HTML image refs。
- manifest paths。
- global manifest ↔ directory。
- blank headings。
- H7+ headings。
- absolute temp paths。
- YAML parse。
- attachment byte identity。
- `$ref` inventory。
- source/output asset counts。
- oversized atomic files。
- orphan files。
- duplicate paths。
- duplicate clause IDs。

## 21.2 驗證器失敗時

```python
if failures:
    raise SystemExit("Final validation has failures")
```

不能在已知 fail 的情況下封裝並宣稱完成。

## 21.3 驗證 false positive

若 validator 攔下內容：

1. 先判斷是 corpus error 還是 validator bug。
2. 修 validator。
3. 從 clean parent corpus 重建。
4. 不在現有 output 上連續手工修補。
5. 將 validator 變更寫入 build log。

---

# 22. 失敗分類與接受政策

## 22.1 Hard failure

以下不得交付：

- tree reassembly hash 不同。
- clause 正文遺失。
- normative wording 被改動。
- broken manifest path。
- broken local links。
- OpenAPI YAML 被改寫。
- source attachment 缺失。
- ZIP integrity failure。
- 無法解釋的正文 token 大量差異。
- correction 未記錄。

## 22.2 Soft limitation

可交付但需明確列出：

- 部分 PNG preview 失敗。
- 大型文件未完成全文 PDF export。
- 未使用功能的 external `$ref` 缺失。
- oversized atomic clause。
- legacy DOC table serialization mismatch。
- 部分 OLE 無法直接預覽。

## 22.3 Review required

- normative marker count 差異。
- heading style 遺失。
- text box 順序差異。
- merged table cell 差異。
- figure 與 caption 不相鄰。
- Annex tree 不合理。

---

# 23. LLM 使用邊界

## 23.1 允許

- 比較 source page 與 Markdown render。
- 標記疑似 misplaced paragraph。
- 建議高風險頁面。
- 協助解讀 parser error。
- 產生非規範性 README 導航。
- 產生 topic tags，但必須與正文隔離。
- 幫助使用者查找 clause。

## 23.2 禁止

- 讓 LLM 讀整章後重新輸出正文。
- 讓 LLM「修順」英文。
- 用 LLM 補 parser 漏字。
- 猜表格 cell。
- 修改 modal verbs。
- 自行建立官方 YAML。
- 在沒有原始來源證據時恢復 heading。
- 將 LLM summary 混入 clause source file。

正確流程：

```text
LLM / validator 發現疑點
    ↓
回查 source Word / PDF / OOXML
    ↓
確認 exact text 與結構
    ↓
新增 deterministic rule
    ↓
clean rebuild
    ↓
重新驗證
```

---

# 24. 增量加入新規格的完整流程

假設已存在一個已驗證 corpus ZIP，需要加入新規格。

## 24.1 Input

```text
parent-corpus.zip
new-spec-1.zip
new-spec-2.zip
```

## 24.2 Steps

1. 驗證 parent corpus ZIP。
2. 記錄 parent SHA-256。
3. 解壓 parent 到唯讀目錄。
4. 建立新的空 build directory。
5. 複製 parent corpus。
6. 對每個新 ZIP 做 inventory。
7. 依 DOCX／DOC adapter 轉換。
8. 建立 independent text evidence。
9. 套用 correction registry。
10. 建 heading tree。
11. 驗證 reassembly hash。
12. 拆檔。
13. 產生 per-spec README／manifest。
14. 複製官方附件。
15. 抽取 original／rendered／embedded assets。
16. 更新全域 README。
17. 重建全域 manifest。
18. 重算所有 OpenAPI dependencies。
19. 更新 validation records。
20. 跑全 corpus validation。
21. 產生 ZIP。
22. `unzip -t`。
23. 產生 SHA-256。
24. 交付 ZIP 與驗證摘要。
25. 不修改 repository。

## 24.3 防止 orphan file

新 build 完成後檢查：

```text
所有 TS 目錄都在 global manifest
所有 global manifest 規格都有目錄
所有附件都在 attachment inventory
所有 clause file 都至少被一個 manifest entry 引用
```

---

# 25. Command logging

真正可重現不能只靠流程文字。

每次 build 至少產生：

```text
_validation/commands.log
_validation/environment.json
_validation/source-inventory.json
_validation/build.json
_validation/tool-versions.txt
```

建議 command wrapper：

```python
def run_logged(
    command: list[str],
    *,
    timeout: int,
    cwd: Path,
    log_dir: Path,
) -> subprocess.CompletedProcess:
    ...
```

記錄：

- exact argv。
- cwd。
- environment overrides。
- start/end。
- duration。
- timeout。
- exit code。
- stdout/stderr。
- input/output hashes。

---

# 26. Suggested config format

```yaml
build:
  build_id: r18-corpus-20260803-001
  parent_archive: previous-corpus.zip
  output_archive: new-corpus.zip

split:
  max_words: 4200

render:
  dpi: 150
  inkscape_timeout: 20
  imagemagick_timeout: 30
  workers: 4

specs:
  - spec: TS 23.288
    version: 18.13.0
    archive: 23288-id0.zip
    document: 23288-id0.docx
    source_format: docx

  - spec: TS 29.520
    version: 18.14.0
    archive: 29520-ie0.zip
    document: 29520-ie0.doc
    source_format: doc
    exact_attachments:
      - "*.yaml"

validation:
  require_tree_hash_match: true
  require_zero_broken_links: true
  require_attachment_byte_identity: true
  dependency_policy: project-required
```

---

# 27. 建議的正式 CLI

```bash
python convert_3gpp_corpus.py build \
  --config corpus.yaml \
  --workdir work \
  --output output/specs
```

子命令：

```text
inventory
convert
validate-text
render-assets
build-tree
split
build-manifests
scan-dependencies
validate
package
```

每個 phase 可重跑，但最終 build 必須從乾淨狀態開始。

---

# 28. Validation report 建議格式

```json
{
  "build_id": "r18-corpus-20260803-001",
  "specifications": 21,
  "files": 6745,
  "markdown_files": 3587,
  "manifest_entries": 9897,
  "local_links": {
    "checked": 3716,
    "broken": 0
  },
  "image_references": {
    "checked": 1087,
    "broken": 0
  },
  "openapi": {
    "yaml_files": 60,
    "parse_success": 60,
    "byte_identical": 60
  },
  "official_non_yaml_attachments": {
    "files": 2,
    "byte_identical": 2
  },
  "visual_validation": {
    "level": "B"
  },
  "limitations": []
}
```

數字應由實際 build 自動產生，不可手寫。

---

# 29. 更新後的驗收清單

## 29.1 Source and build

- [ ] 每個 source ZIP 已記錄 SHA-256。
- [ ] 文件內版本已確認。
- [ ] package version 與 attachment declared version 分開記錄。
- [ ] parent corpus ZIP 已記錄 SHA-256。
- [ ] build 有唯一 build ID。
- [ ] exact tool versions 已記錄。
- [ ] exact commands 已記錄。

## 29.2 Text and structure

- [ ] tree reassembly hash 完全一致。
- [ ] independent text extraction 已完成。
- [ ] normative markers 已比對。
- [ ] NOTE／EXAMPLE 已比對。
- [ ] Annex tree 合理。
- [ ] blank heading 為 0。
- [ ] H7+ unintended headings 為 0。
- [ ] 所有 correction 均有 registry 紀錄。
- [ ] legacy DOC 差異已分類。

## 29.3 Navigation

- [ ] per-spec README 存在。
- [ ] per-spec manifest 存在。
- [ ] global manifest 與目錄一致。
- [ ] embedded clauses 也有 manifest entry。
- [ ] common entry paths 可定位主要功能。
- [ ] generated navigation 與 normative text 有明確區隔。

## 29.4 Attachments and assets

- [ ] 官方 YAML／ABNF／XML 與來源 byte-identical。
- [ ] YAML parse success。
- [ ] original vector assets 完整。
- [ ] embedded OLE／Visio 完整。
- [ ] preview failures 已列出。
- [ ] 沒有空白 placeholder 假裝轉換成功。

## 29.5 Dependency

- [ ] 所有 `$ref` 已掃描。
- [ ] project-required dependencies 已閉合。
- [ ] optional dependencies 已分類。
- [ ] unresolved dependencies 有來源檔與引用者。
- [ ] 不為追求零依賴而無限加入規格。

## 29.6 Global validation

- [ ] broken local links 為 0。
- [ ] broken image refs 為 0。
- [ ] missing manifest paths 為 0。
- [ ] 沒有 orphan files。
- [ ] 沒有不應存在的 absolute work paths。
- [ ] validator 排除 code fence／inline code。
- [ ] 視覺驗證層級已標示。
- [ ] ZIP `unzip -t` 通過。
- [ ] 最終 ZIP SHA-256 已提供。

---

# 30. 可直接交給下一個對話的指示

```text
請依照附上的
「3GPP Word 規格轉換為 Agent-readable Markdown 的可複現流程 v2」
處理我提供的 3GPP ZIP。

要求：

1. 不修改任何 GitHub repository。
2. 以上一版已驗證 corpus ZIP 為 parent，執行 clean build。
3. 記錄 parent ZIP、每個 source ZIP 與最終 ZIP 的 SHA-256。
4. 產生唯一 build ID，記錄所有工具版本與 exact command log。
5. 先盤點 ZIP、文件版本、Word 格式、附件與 declared attachment versions。
6. 規格正文必須 deterministic extraction，不可用 LLM 重寫。
7. DOCX 使用 Pandoc、docx2txt 與 OOXML；legacy DOC 使用 antiword、LibreOffice TXT、LibreOffice DOCX 與 Pandoc。
8. legacy DOC 的差異要分類，不可只用全文相等判斷。
9. 建立 heading tree，reassembly hash 一致後才可拆檔。
10. 預設以約 4,200 words 與 clause hierarchy 做 progressive disclosure。
11. 不可任意拆開單一大型 normative table 或 atomic clause。
12. 複雜 merged table 保留 HTML table。
13. 所有 heading correction 必須 deterministic 並寫入 correction registry。
14. 官方 OpenAPI／ABNF／XML 必須 byte-for-byte 保存。
15. package version 與附件 declared version 分開記錄。
16. 保留 original vector、rendered preview 與 embedded OLE／Visio。
17. preview 失敗可接受，但原始圖資不得遺失，且需列入報告。
18. OpenAPI dependency 依 current-project、primary、optional、unresolved 分級。
19. 不為追求所有 `$ref` 歸零而無限加入其他規格。
20. 使用 Markdown-aware validator，排除 code fence、inline code 與範例文字。
21. 驗證 local links、image links、manifest paths、YAML、附件、absolute paths、orphan files 與 ZIP 完整性。
22. 對高風險表格、圖、text box、Annex 與 heading correction 做視覺抽查。
23. 清楚標示視覺驗證 Level A、B 或 C，不得誇大。
24. 全 corpus 驗證失敗時不得封裝交付。
25. 最終提供頂層為 specs/ 的完整 ZIP、SHA-256 與驗證摘要。
```

---

# 31. 最重要的精確度原則

最終 corpus 應遵守：

```text
正文精確度
    優先於格式美觀

結構可追溯性
    優先於檔案數量最少

官方附件原貌
    優先於本地格式一致

可重建性
    優先於手動快速修補

project-required dependency closure
    優先於整個 5GC dependency 歸零

原始圖資
    優先於 PNG preview 成功率
```

若有任何衝突，回到原始 3GPP 文件確認，不得由 LLM 或 parser 猜測。

---

# 32. 版本 2.0 相較 1.0 的主要更新

1. 新增 clean-build 與 parent corpus 策略。
2. 新增 build ID、provenance 與 exact command logging。
3. 將 DOCX 與 legacy DOC 正式拆成兩種 adapter。
4. 新增 legacy DOC 差異分類與接受政策。
5. 泛化 Annex heading correction。
6. 明確定義原始圖資、embedded object 與 preview 的信任層級。
7. 新增 preview timeout、resume 與 failure manifest。
8. 將視覺驗證分成 Level A／B／C。
9. 新增 Markdown-aware validation，避免 regex false positive。
10. 新增 dependency 分級，不追求無限制 `$ref` 歸零。
11. 新增 package version 與 attachment declared version 分離。
12. 新增 orphan file 與 manifest 雙向一致性檢查。
13. 新增 hard failure、soft limitation 與 review-required 分類。
14. 增量整合改為每輪完整重建與全 corpus 驗證。
15. 驗收標準擴充為可直接工程化的 checklist。
