# 3GPP Word 規格轉換為 Agent-readable Markdown 的可複現流程

> 適用案例：將 3GPP TS 23.288、TS 29.520 等 Word 規格轉成可逐層載入的 `specs/` 文件庫，同時維持規格文字精確度、保留 OpenAPI 與原始圖資，並產生可追溯的驗證紀錄。  
> 本文件記錄 2026 年 7 月實際處理下列輸入時採用的流程：
>
> - TS 23.288 V18.13.0：`23288-id0.zip`
> - TS 29.520 V18.14.0：`29520-ie0.zip`
>
> 本流程不直接修改 GitHub repository；所有輸出先在獨立工作目錄生成，驗證後再交付 ZIP。

---

## 1. 目標與非目標

### 1.1 目標

此流程的目標不是單純將 Word「另存為 Markdown」，而是建立一套適合 coding agent 查閱的規格 corpus：

```text
specs/README.md
    ↓
TS xx.xxx/README.md
    ↓
大型 clause/README.md
    ↓
最小可用的 clause Markdown
```

輸出需同時滿足：

1. 規格正文不經摘要、翻譯或改寫。
2. `shall`、`should`、`may`、否定條件、例外條件及表格欄位不得被語意重寫。
3. 大型 clause 採 progressive disclosure，避免 agent 一次讀取整份規格。
4. 每個輸出檔可追溯到來源 ZIP、來源文件、版本與 SHA-256。
5. 原始 OpenAPI YAML 原封不動保存。
6. 圖片提供可閱讀的 PNG preview，同時保留 EMF、WMF、Visio、OLE 等原始物件。
7. 產生 machine-readable manifest 與 validation report。
8. 不在未確認版本相容性的情況下，自動補入其他 Release 的 schema dependency。

### 1.2 非目標

本流程不保證：

- Markdown 與 Word 的頁面外觀完全一致。
- 保留 Word 分頁、浮動 anchor、字型、頁首頁尾與所有視覺排版。
- 自動解釋規格意義。
- 自動補齊所有跨 TS 引用的規格。
- 以 LLM 猜測或修正文句。

此流程優先保證的是：

```text
文字忠實度
→ 結構忠實度
→ 導航效率
→ 視覺近似
```

---

## 2. 本次輸入與輸出基準

### 2.1 來源檔案

| 規格 | 來源 ZIP | 來源文件 | Release / Version |
|---|---|---|---|
| TS 23.288 | `23288-id0.zip` | `23288-id0.docx` | Release 18 / V18.13.0 |
| TS 29.520 | `29520-ie0.zip` | `29520-ie0.doc` | Release 18 / V18.14.0 |

TS 29.520 ZIP 另附 8 份 OpenAPI YAML：

```text
TS29520_Nnwdaf_AnalyticsInfo.yaml
TS29520_Nnwdaf_DataManagement.yaml
TS29520_Nnwdaf_EventsSubscription.yaml
TS29520_Nnwdaf_MLModelMonitor.yaml
TS29520_Nnwdaf_MLModelProvision.yaml
TS29520_Nnwdaf_MLModelTraining.yaml
TS29520_Nnwdaf_RoamingAnalytics.yaml
TS29520_Nnwdaf_RoamingData.yaml
```

### 2.2 SHA-256

```text
7116a1f4123dcf949818ab56feb6b242eda8819877346ab045654e1d0d8c36aa  23288-id0.zip
4937d4ffcdc8760cac3dd990b10d5c0beb722f9eab26d94ce4b1f296287057c1  29520-ie0.zip
```

本次最終交付 ZIP：

```text
78aa204449f5571d18fe9f939e42722c99c5232a1f0ccdeec28394546ddfecc0
nwdaf-specs-r18-ts23288-v18.13.0-ts29520-v18.14.0.zip
```

### 2.3 最終輸出結構

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
│       ├── original/
│       ├── rendered/
│       └── embedded/
├── openapi/
│   ├── README.md
│   ├── manifest.yaml
│   └── official YAML files
└── _validation/
    ├── README.md
    ├── SHA256SUMS.txt
    ├── report.json
    ├── final-checks.json
    ├── text-cross-check.json
    ├── visual-sampling.json
    └── deterministic-corrections.yaml
```

---

## 3. 核心設計原則

## 3.1 正文只由 deterministic parser 產生

正文不交給 LLM 重新輸出。處理方式是：

```text
Word source
    ↓
Pandoc / OOXML / legacy DOC conversion
    ↓
Markdown 中間結果
    ↓
規則式 heading tree
    ↓
規則式拆檔
    ↓
Markdown corpus
```

LLM 僅可用於：

- 視覺抽查時指出疑似排版錯位。
- 協助人工理解異常。
- 未來產生額外的 topic/navigation metadata。
- 檢查 README 是否足以讓 agent 找到相關 clause。

LLM 不可用於：

- 改寫 normative text。
- 自行修正文法。
- 猜測遺失文字。
- 改寫表格內容。
- 改動 modal verbs。
- 依「合理意思」補完原文。

## 3.2 OpenAPI 不由 Annex Markdown 反向重建

TS 29.520 Annex A 內也包含 API definition，但 ZIP 已提供官方 YAML。處理原則：

```text
ZIP 中的 YAML = API source of truth
```

因此：

- YAML 使用 byte-for-byte copy。
- 不重新格式化。
- 不改版號。
- 不將所有 YAML 都強制標記為 package 的最新 point release。
- Annex A 的 Markdown 只保留導覽與一般說明，API body 連到官方 YAML。
- 解析 `$ref`，但不自動混入其他 Release 的 dependency。

## 3.3 先建樹，再拆檔

不可直接以固定 heading level 切檔。正確順序：

1. 將完整 Markdown 解析成 heading tree。
2. 計算每個 node 本身與 subtree 的字數。
3. 依語意 clause 與大小決定是：
   - 單一 Markdown；
   - 目錄加 `README.md`；
   - 特殊大表格拆分。
4. 任何子 clause 都必須保留在 manifest 中，即使它沒有獨立檔案。

本次大小門檻：

```python
MAX_WORDS = 4200
```

規則：

```text
subtree <= 4200 words
    → 一個 Markdown，內含其子 clause

subtree > 4200 words 且存在子 clause
    → 建立資料夾 README，只列 immediate children

沒有合理子 clause 可拆，但仍超過 4200
    → 保持完整 atomic clause，不在任意段落位置強拆
```

---

## 4. 使用工具與版本

本次實際環境：

| 工具 | 用途 | 版本 |
|---|---|---|
| Python | tree parsing、拆檔、manifest、驗證 | 3.13.5 |
| PyYAML | YAML parse 與 manifest 輸出 | 6.0.3 |
| Pandoc | DOCX → Markdown，保留 heading/table/media | 3.1.11.1 |
| LibreOffice | legacy DOC → intermediate DOCX/TXT/PDF | 25.2.3.2 |
| `docx2txt` | TS 23.288 獨立文字抽取 | Python CLI |
| `antiword` | TS 29.520 legacy DOC 獨立文字抽取 | 系統工具 |
| Inkscape | EMF/WMF → PNG preview | 1.4 |
| ImageMagick | 圖片轉換 fallback | 7.1.2-1 |
| Poppler `pdftoppm` | PDF page → image，供視覺抽查 | 25.06.0 |
| `zip` / `unzip` | 來源解包與交付封裝 | 系統工具 |
| SHA-256 | 來源與輸出完整性 | `sha256sum` / Python `hashlib` |

建議將版本寫入 validation report，因為 Pandoc 或 LibreOffice 升級後可能改變：

- heading style mapping；
- HTML table 產生方式；
- 圖片 anchor；
- legacy DOC 轉換結果。

---

## 5. 工作目錄配置

推薦使用以下工作區，不直接在 repository 內執行：

```text
work/
├── source/
│   ├── 23288/
│   └── 29520/
├── intermediate/
│   └── 29520_docx/
├── pandoc/
│   ├── 23288.md
│   ├── 23288_media/
│   ├── 29520.md
│   └── 29520_media/
├── validation_raw/
├── rendered_source/
├── page_samples/
├── build_specs.py
└── final_validate.py

output/
└── specs/
```

此次環境使用：

```text
/mnt/data/spec_work/
/mnt/data/NWDAF_3GPP_R18_specs/
```

腳本不應硬編碼這些路徑；正式重做時應改為 CLI arguments 或 config YAML。

---

## 6. 完整轉換流程

# Phase 0：來源確認與唯讀盤點

## 6.1 檢查 ZIP

```bash
unzip -l 23288-id0.zip
unzip -l 29520-ie0.zip
sha256sum 23288-id0.zip 29520-ie0.zip
```

確認：

- 檔名版本碼。
- Word 格式是 `.docx` 還是 `.doc`。
- 是否包含 YAML、圖片或其他附件。
- 是否有意外的多層 ZIP。
- 檔案日期與規格文件內頁首版本是否一致。

注意：3GPP 網頁 archive 索引可能晚於實際收到的檔案。應以：

1. 上傳 ZIP；
2. ZIP 中 Word 文件頁首；
3. 文件 properties；
4. 附件內容；

共同判斷，不應只信外部列表。

## 6.2 解壓

```bash
mkdir -p work/source/23288 work/source/29520

unzip -q 23288-id0.zip -d work/source/23288
unzip -q 29520-ie0.zip -d work/source/29520
```

不要修改 source 目錄中的檔案。

---

# Phase 1：建立可解析的 Word 來源

## 6.3 TS 23.288：直接使用 DOCX

來源：

```text
work/source/23288/23288-id0.docx
```

DOCX 是 ZIP-based OOXML，可直接：

- 由 Pandoc 解析；
- 由 `docx2txt` 抽取；
- 解開 `word/media/`；
- 解開 `word/embeddings/`；
- 讀取 styles、numbering、relationships。

## 6.4 TS 29.520：legacy DOC 先產生 intermediate DOCX

來源是 binary `.doc`：

```text
work/source/29520/29520-ie0.doc
```

Pandoc 無法可靠直接處理所有舊式 Word 結構，因此先用 LibreOffice 轉成中間 DOCX：

```bash
mkdir -p work/intermediate/29520_docx

libreoffice \
  --headless \
  --convert-to docx \
  --outdir work/intermediate/29520_docx \
  work/source/29520/29520-ie0.doc
```

重要原則：

```text
intermediate DOCX 是結構解析介面，不是 source of truth。
原始 DOC 仍須保留並作獨立文字比對。
```

原因：

- DOC → DOCX 可能改變 heading styles。
- text box 可能改變順序。
- merged cells 可能被重新表示。
- floating objects / OLE anchor 可能變動。
- 字詞可能因版面斷行而被拆開。

---

# Phase 2：Pandoc 結構抽取

## 6.5 DOCX → Markdown

本次保留的工作結果顯示 Pandoc 使用了獨立 media extraction 目錄。可用下列命令重現同類結果：

```bash
mkdir -p work/pandoc/23288_media work/pandoc/29520_media

pandoc \
  work/source/23288/23288-id0.docx \
  --from=docx \
  --to=gfm \
  --wrap=none \
  --extract-media=work/pandoc/23288_media \
  --output=work/pandoc/23288.md

pandoc \
  work/intermediate/29520_docx/29520-ie0.docx \
  --from=docx \
  --to=gfm \
  --wrap=none \
  --extract-media=work/pandoc/29520_media \
  --output=work/pandoc/29520.md
```

> 可重現性註記：本次原始 shell command 沒有被獨立保存，但輸出路徑、Pandoc 版本、media 目錄及產生結果均保留。上列命令是與本次結果一致的重建方式。後續正式 pipeline 應將所有命令寫入 Makefile、shell script 或 CI log。

## 6.6 為何使用 GFM，但保留 HTML table

GitHub-Flavored Markdown 無法完整表達：

- rowspan；
- colspan；
- nested table；
- 複雜 cell block；
- 多段落 cell。

Pandoc 在這些情況會輸出 HTML `<table>`。不要強制將所有 HTML table 改寫成 pipe table，否則容易破壞語意。

原則：

```text
簡單表格
→ Markdown table 可接受

合併儲存格或複雜內容
→ 保留 Pandoc HTML table
```

---

# Phase 3：獨立文字抽取

單一路徑轉換不能證明文字正確。至少保留第二條 extractor。

## 6.7 TS 23.288

```bash
docx2txt \
  work/source/23288/23288-id0.docx \
  work/validation_raw/23288_docx2txt.txt
```

另外可直接由 OOXML 讀取 `word/document.xml`，作第三條路徑。

Pandoc 版本輸出另轉為 plain text：

```bash
pandoc work/pandoc/23288.md --to=plain \
  --output=work/validation_raw/23288_pandoc.txt
```

## 6.8 TS 29.520

原始 DOC 使用兩條額外路徑：

```bash
antiword work/source/29520/29520-ie0.doc \
  > work/validation_raw/29520_antiword.txt
```

以及 LibreOffice text export：

```bash
mkdir -p work/validation_raw/lo_txt

libreoffice \
  --headless \
  --convert-to txt:Text \
  --outdir work/validation_raw/lo_txt \
  work/source/29520/29520-ie0.doc
```

Pandoc 中間結果再輸出 plain text：

```bash
pandoc work/pandoc/29520.md --to=plain \
  --output=work/validation_raw/29520_pandoc.txt
```

### 6.9 為何不能要求全文 byte-equal

不同 extractor 會有合理差異：

- soft hyphen；
- non-breaking space；
- layout-driven line break；
- header/footer；
- table serialization；
- `Cardinality` 被拆成 `Cardinal ity`；
- TOC 重複文字；
- hidden Word fields。

因此比對前需正規化：

```python
text = text.replace("\u00ad", "")
text = text.replace("\u00a0", " ")
tokens = re.findall(r"[A-Za-z0-9]+(?:[-'][A-Za-z0-9]+)*", text.lower())
```

採用：

1. token count；
2. 12-token shingles；
3. token multiset overlap；
4. modal verb 與 marker 精確計數。

高敏感詞：

```text
shall
shall not
should
should not
may
may not
must
NOTE
EXAMPLE
```

---

# Phase 4：Markdown 預處理與 deterministic correction

## 6.10 只校正結構，不校正文句

本次發現的 deterministic corrections：

### TS 23.288

- Pandoc 將 Annex A、Annex B 映射成 H8。
- Annex 子節被誤成 H1。

修正：

```text
######## Annex A ...
→ # Annex A ...

# A.1 ...
→ ## A.1 ...
```

### TS 29.520

- 出現一個空白 H1。
- Annex A / Annex B 被映射成 H8。
- A.2～A.8 被映射成錯誤層級。
- `A.9 Nnwdaf_RoamingAnalytics API` 在 DOC → DOCX 時失去 heading style。

修正：

```text
# [blank]
→ 移除 formatting artifact

A.9 Nnwdaf_RoamingAnalytics API
→ ## A.9 Nnwdaf_RoamingAnalytics API
```

每個 correction 必須寫入：

```text
specs/_validation/deterministic-corrections.yaml
```

格式：

```yaml
corrections:
  - spec: TS 29.520
    line: 40525
    type: restore-heading
    original: A.9 Nnwdaf_RoamingAnalytics API
    replacement: "## A.9 Nnwdaf_RoamingAnalytics API"
```

不可在沒有 source-page 證據時猜測 heading。

---

# Phase 5：建立 heading tree

## 6.11 Node 資料結構

使用 Python 將 `#`～`######` 解析成樹：

```python
@dataclass
class Node:
    level: int
    title: str
    content: list[str]
    children: list["Node"]
    parent: "Node | None"
    word_count: int
    subtree_words: int
```

Clause number 從 title 提取：

```text
6
6.2
6.2A
6.2C.2.1
Annex A
```

推薦 regex：

```python
r"^((?:\d+[A-Z]?|[A-Z])(?:\.\d+[A-Z]?)*)(?:\s+|$)"
```

## 6.12 完整性 hash

建立 tree 後，將 tree 重新組回 Markdown，對 preprocessed Markdown 算 SHA-256：

```text
preprocessed Markdown hash
tree reassembled Markdown hash
```

必須相同。

這能證明：

- parse tree 沒有漏掉 paragraph；
- 沒有改變順序；
- 拆檔前資料完整存在。

---

# Phase 6：Progressive disclosure 拆檔

## 6.13 一般規則

```python
MAX_WORDS = 4200
```

### Leaf file

當 subtree 不超過門檻，輸出：

```text
6.2A Procedure for ML Model Provisioning.md
```

該檔可包含較小的子 clause。

### Directory

當 subtree 超過門檻且有子節：

```text
6 Procedures to Support Network Data Analytics/
├── README.md
├── 6.1 ...md
├── 6.2 .../
│   ├── README.md
│   └── ...
└── ...
```

Directory README 只列 immediate children，避免一次載入全部 descendants。

若該父 clause 在第一個子 clause 前已有規格正文，README 中另放：

```markdown
## Specification text before child clauses
```

並明確標記導航文字不是 3GPP 原文。

## 6.14 Change history 特例

Change history 是大型單一表格，通常沒有可用的 clause children。任意切 paragraph 會破壞 table。

本次按 calendar year 拆分：

```text
Annex B Change history/
├── README.md
├── 2022.md
├── 2023.md
├── 2024.md
├── 2025.md
└── 2026.md
```

規則：

- table header 複製到每個年度檔。
- row 本身不改寫。
- continuation row 與前一年度維持在一起。
- README 說明這是 progressive loading，不是原始規格的新章節。

## 6.15 不強制拆開 atomic clause

有些 clause 超過 4,200 words，但沒有安全的語意子節，例如：

- 單一大型 data type table；
- 單一 service operation；
- 合併儲存格密集的表格。

這類檔案保留完整，並在 `final-checks.json` 中列為 oversized exception。

---

# Phase 7：Front matter 與 manifest

## 6.16 Clause file front matter

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
---
```

對 navigation README：

```yaml
content_origin: generated-navigation-and-3gpp-source
```

## 6.17 Manifest

每份規格有：

```text
TS xx.xxx/manifest.yaml
```

每個 heading 都登錄，即使它只是某個大檔案內的 embedded clause：

```yaml
clauses:
  - clause: "6.2A"
    title: Procedure for ML Model Provisioning
    path: "6 Procedures/.../6.2A Procedure for ML Model Provisioning.md"
    source_words: 2340
    kind: source-file

  - clause: "6.2A.2"
    title: Procedure
    path: "6 Procedures/.../6.2A Procedure for ML Model Provisioning.md"
    source_words: 1810
    kind: embedded-clause
```

如此 agent 可透過 manifest 直接定位，不必掃描所有檔案。

---

# Phase 8：圖片、Visio 與 OLE

## 6.18 資產分類

每份規格：

```text
assets/
├── original/   # Pandoc 抽出的 EMF/WMF/PNG/JPG
├── rendered/   # PNG reading preview
└── embedded/   # word/embeddings 中的 Visio/OLE/Word object
```

## 6.19 原始物件抽取

DOCX 可直接當 ZIP 開啟：

```python
with zipfile.ZipFile(source_docx) as z:
    for name in z.namelist():
        if name.startswith("word/embeddings/"):
            ...
```

TS 29.520 使用 LibreOffice 生成的 intermediate DOCX 抽取 embeddings，但仍保留原始 `.doc` 作來源。

本次結果：

| 規格 | media | embedded |
|---|---:|---:|
| TS 23.288 | 94 | 94 |
| TS 29.520 | 54 | 20 |

TS 23.288 embeddings 包含：

- `.vsd`
- `.vsdx`
- embedded `.doc`
- embedded `.docx`
- `oleObject*.bin`

## 6.20 PNG preview 轉換

優先順序：

1. PNG 直接 copy。
2. EMF/WMF 嘗試 Inkscape。
3. 失敗時 ImageMagick fallback。
4. 每張設 timeout，避免單張 vector 卡死整批。
5. 已存在且非空的 PNG 直接 skip。
6. 原始 vector 永遠保留，即使 preview 失敗。

核心邏輯：

```python
inkscape source.emf \
  --export-type=png \
  --export-filename=target.png \
  --export-dpi=150
```

Fallback：

```bash
magick -density 180 source.emf -trim +repage target.png
```

實際問題：

- TS 29.520 某些 `.wmf` 內容實際可由 EMF parser 處理。
- 直接以 `.wmf` extension 呼叫工具可能卡住。
- 解法是複製到 temporary `.emf` 再交給 Inkscape。
- 設定 20 秒 Inkscape timeout、30 秒 ImageMagick timeout。
- 使用最多 4 worker，避免向量轉檔過度併發。

Markdown 只連到 `assets/rendered/*.png`，但 `assets/original/` 是追溯來源。

---

# Phase 9：TS 29.520 Annex A 與 OpenAPI

## 6.21 Byte-for-byte copy

```python
shutil.copy2(source_yaml, output_yaml)
```

驗證：

```text
source SHA-256 == output SHA-256
```

本次 8/8 byte-identical，8/8 YAML parse success。

## 6.22 `$ref` dependency inventory

掃描：

```python
r"\$ref:\s*['\"]?([^'\"\s]+)"
```

分成：

- internal ref：`#/components/...`
- external file ref：`TS29571_CommonData.yaml#/...`

若 external file 不在本次 ZIP：

- 寫入 `openapi/manifest.yaml`；
- 寫入 `openapi/README.md`；
- 不自動下載其他版本。

本次缺少 17 個 shared schema 檔，例如：

```text
TS29510_Nnrf_NFManagement.yaml
TS29571_CommonData.yaml
TS29575_Nadrf_DataManagement.yaml
...
```

## 6.23 Package version 與 file-declared version 分開

TS 29.520 package 是 V18.14.0，但 8 份 YAML 中只有 MLModelTraining 宣告 V18.14.0，其餘宣告 V18.13.0。

不可批次改寫。應保留：

```yaml
package_version: 18.14.0
external_docs_description: 3GPP TS 29.520 V18.13.0 ...
```

---

# Phase 10：視覺驗證

## 6.24 原始 Word 渲染

用 LibreOffice 將原始 Word 轉 PDF：

```bash
libreoffice \
  --headless \
  --convert-to pdf \
  --outdir work/rendered_source \
  source.docx
```

legacy DOC 同理直接由原始 `.doc` 轉 PDF，不要只渲染 intermediate DOCX。

## 6.25 抽取高風險頁面

```bash
pdftoppm \
  -f START_PAGE \
  -l END_PAGE \
  -png \
  source.pdf \
  work/page_samples/sample
```

本次抽查：

| 規格 | 頁面 | 風險 |
|---|---|---|
| TS 23.288 | 30–35 | clause hierarchy、numbered procedure、sequence diagram |
| TS 23.288 | 200–203 | complex analytics table、Table 6.8.3-1、notes |
| TS 29.520 | 26–35 | nested list、NOTE numbering、clause boundaries |
| TS 29.520 | 100–104 | API resource/data-model tables |
| TS 29.520 | 343–345 | MLModelMonitor URI figure、method table |

抽查重點：

- paragraph 是否落在正確 clause。
- list numbering 是否正確。
- NOTE 是否包含完整範圍。
- table cell 是否錯位。
- caption 與 figure 是否相鄰。
- 圖片 preview 是否和 source 一致。
- heading style correction 是否有 source-page 證據。

視覺抽查結果寫入：

```text
_validation/visual-sampling.json
```

此步驟是 risk-based sampling，不是逐頁認證。

---

# Phase 11：自動驗證

## 6.26 文字交叉驗證

### TS 23.288

本次結果：

- Pandoc normalized tokens：152,389
- `docx2txt` normalized tokens：152,392
- 12-token sequence overlap：約 99.34%
- token multiset overlap：>99.98%
- modal/NOTE/EXAMPLE count 完全一致

### TS 29.520

只比較 normative main body：

- Pandoc normalized tokens：113,481
- LibreOffice text normalized tokens：113,470
- modal/NOTE/EXAMPLE count 完全一致
- `antiword` 作第三路徑，主要差異是 layout-induced word split

## 6.27 Link 與檔案檢查

檢查：

- Markdown local links。
- Markdown image links。
- HTML `<img src>`。
- manifest path。
- H7 以上 heading。
- blank heading。
- `/mnt/data/`、`/tmp/` 等工作路徑是否洩漏。
- YAML parse。
- YAML byte identity。
- oversized files。

驗證失敗時腳本應 non-zero exit，不可仍然封裝：

```python
if failures:
    raise SystemExit("Final validation has failures")
```

本次結果：

```text
files: 803
Markdown files: 375
local links checked: 390
broken local links: 0
image references checked: 157
broken image references: 0
manifest entries: 389 + 746
missing manifest paths: 0
H7-or-deeper headings: 0
blank headings: 0
absolute work paths: 0
OpenAPI byte-identical: 8/8
OpenAPI YAML valid: 8/8
```

---

# Phase 12：封裝

驗證通過後才封裝：

```bash
cd output-parent
zip -r \
  nwdaf-specs-r18-ts23288-v18.13.0-ts29520-v18.14.0.zip \
  specs/
```

測試 ZIP：

```bash
unzip -t \
  nwdaf-specs-r18-ts23288-v18.13.0-ts29520-v18.14.0.zip
```

產生 SHA-256：

```bash
sha256sum \
  nwdaf-specs-r18-ts23288-v18.13.0-ts29520-v18.14.0.zip
```

在使用者明確要求前，不：

- push；
- commit；
- create branch；
- create PR；
- 刪除或覆寫 repository 的 `specs/`。

---

## 7. 本次遇到的主要問題與處理方式

| 問題 | 風險 | 處理方式 |
|---|---|---|
| 外部 3GPP archive 索引落後 | 選到較舊版本 | 以使用者 ZIP 與文件內版本為準 |
| TS 29.520 是 legacy DOC | heading/table/text box 轉換失真 | LibreOffice intermediate DOCX + original DOC independent checks |
| Annex headings 被映射成 H8 | tree 層級錯誤 | deterministic heading promotion |
| TS 29.520 A.9 heading style 遺失 | clause 被併入前一節 | 依原始頁面恢復 heading，記錄 correction |
| 空白 heading | 產生無名 clause | 移除 formatting artifact，記錄 correction |
| TOC anchor 在拆檔後失效 | 大量 broken links | 將原 TOC internal links轉成 visible text |
| merged-cell table | pipe table 破壞語意 | 保留 HTML table |
| Change history 過大 | agent 一次讀太多 | 依年度拆 row，文字不改寫 |
| 某些 atomic clause 仍過大 | 強拆會破壞語意 | 列為 oversized exception |
| WMF/EMF 轉 PNG 卡住 | build 無限等待 | per-image timeout、temporary `.emf`、fallback |
| OLE/Visio 無法完整呈現在 Markdown | 圖資遺失 | 原始 embedded object 與 preview 同時保留 |
| Annex A 與 YAML 重複 | 兩份 API source 可能漂移 | Markdown 不重抄 API body，改連 exact YAML |
| OpenAPI 缺 shared schemas | schema 無法完全 resolve | 列出 17 個 dependency，不跨版本自動補入 |
| YAML package/file version 不一致 | 錯誤統一版本 | 原樣保留 individual declaration |
| 單一路徑文字看似正常但可能漏字 | 隱性 normative error | independent extractors + modal count + shingle comparison |

---

## 8. LLM 在流程中的允許範圍

### 8.1 可以使用

- 觀看 source page 與 Markdown render，標記可疑位置。
- 解釋 parser 為何可能誤判。
- 建議需要人工抽查的高風險頁面。
- 產生 corpus 根 README 的一般使用說明。
- 在不混入正文的前提下，未來建立 `Read when` 或 topic index。

### 8.2 不可使用

- 讓模型讀一整章後重新輸出 Markdown 正文。
- 讓模型「修順」英文。
- 用模型補回 parser 沒讀到的句子。
- 依語意推測表格 cell。
- 修改 `shall`、`may` 等 normative wording。
- 直接以模型輸出取代原始 YAML。
- 在沒有原始頁面證據時更正 clause boundary。

正確校正流程：

```text
LLM / validator 發現疑點
    ↓
回查原始 Word / PDF 頁面
    ↓
確認 exact source text 與結構
    ↓
修改 deterministic rule
    ↓
重新 build
    ↓
重新驗證
```

---

## 9. 驗收標準

新的規格在交付前至少要達成：

### 必須通過

- [ ] 來源 ZIP 與文件 SHA-256 已記錄。
- [ ] 規格版本已由文件本身確認。
- [ ] parser tree reassembly hash 完全一致。
- [ ] 沒有 broken local link。
- [ ] 沒有 broken image reference。
- [ ] manifest path 全部存在。
- [ ] OpenAPI YAML 與來源 byte-identical。
- [ ] YAML 全部 parse success。
- [ ] `shall/should/may` 等關鍵詞獨立抽取計數一致。
- [ ] NOTE 與 EXAMPLE 計數一致。
- [ ] 沒有空 heading 或 H7+ 殘留。
- [ ] 沒有 temporary absolute path。
- [ ] 複雜表格與圖至少完成 risk-based visual sampling。
- [ ] 所有 deterministic correction 都有紀錄。
- [ ] ZIP `unzip -t` 通過。

### 需列入限制說明

- [ ] oversized atomic files。
- [ ] missing external OpenAPI schemas。
- [ ] failed image previews。
- [ ] legacy DOC text box / floating object 風險。
- [ ] 未逐頁人工認證的事實。

---

## 10. 重跑清單

下一次對其他 3GPP 規格執行時：

1. 收到官方 ZIP。
2. 不修改 repo。
3. `unzip -l` 與 SHA-256。
4. 確認 `.doc` / `.docx`。
5. 盤點附帶 YAML、XML、圖片及其他附件。
6. legacy DOC 轉 intermediate DOCX。
7. Pandoc 產生 raw Markdown 與 media。
8. 產生至少一條獨立文字抽取。
9. 建立 preprocessor correction list。
10. 建立 heading tree。
11. reassembly hash check。
12. 依 4,200 words 與 clause hierarchy 拆檔。
13. 產生 front matter 與 manifest。
14. 抽出 original/rendered/embedded assets。
15. OpenAPI byte copy 與 dependency scan。
16. text cross-check。
17. link、manifest、heading、path 檢查。
18. risk-based visual sampling。
19. 產生 `_validation/`。
20. 所有檢查通過後封裝 ZIP。
21. 將 ZIP 與 SHA-256 交付。
22. 除非使用者另行要求，不做任何 GitHub write action。

---

## 11. 建議未來改進

本次流程已可重現，但仍可工程化：

### 11.1 將臨時腳本改成正式 CLI

```bash
python convert_3gpp_spec.py \
  --archive 23288-id0.zip \
  --spec TS-23.288 \
  --version 18.13.0 \
  --output output/specs
```

### 11.2 使用 config YAML

```yaml
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

split:
  max_words: 4200
```

### 11.3 容器化

固定：

- Pandoc；
- LibreOffice；
- Inkscape；
- ImageMagick；
- Poppler；
- Python dependencies。

否則工具升級可能使輸出產生非語意差異。

### 11.4 產生 diff-friendly stable IDs

未來可為每個 source block 建立：

```yaml
source_block_id: paragraph-1842
source_xml_path: word/document.xml
```

如此新版規格可做 clause-level diff。

### 11.5 新增 automated page risk scoring

高風險頁面可依下列訊號自動選出：

- table 數量；
- merged cell；
- OLE；
- text box；
- floating image；
- 多層 numbering；
- Annex；
- code/YAML；
- heading style mismatch。

---

## 12. 可直接交給下一個對話的工作指示

```text
請依照附上的
「3GPP Word 規格轉換為 Agent-readable Markdown 的可複現流程」
處理我提供的 3GPP ZIP。

要求：

1. 不修改任何 GitHub repository。
2. 先盤點 ZIP、版本、Word 格式、附件與 SHA-256。
3. 規格正文必須 deterministic extraction，不可用 LLM 重寫。
4. legacy DOC 需建立 intermediate DOCX，但原始 DOC 必須作獨立文字驗證。
5. 使用 Pandoc 保留 heading、table、list、caption 與 media。
6. 建立 heading tree，先驗證 reassembly hash，再拆檔。
7. 採 progressive disclosure；預設約 4,200 words 為拆分門檻。
8. 複雜 merged tables 保留 HTML table。
9. OpenAPI/config 附件必須 byte-for-byte 保存。
10. 不自動加入其他 Release 的 schema dependency。
11. 保留 original vector、rendered PNG 與 embedded OLE/Visio。
12. 使用至少兩條獨立文字抽取路徑，檢查 modal verbs、NOTE、EXAMPLE 與 token overlap。
13. 對高風險表格、圖、text box 與 heading correction 做 source-page 視覺抽查。
14. 所有修正必須是 deterministic correction，並寫入 validation report。
15. 驗證 local links、image links、manifest paths、YAML、absolute paths 及 ZIP 完整性。
16. 最後提供頂層為 specs/ 的 ZIP 與 SHA-256。
```

---

## 13. 最重要的精確度界線

此 corpus 是方便 agent 檢索的衍生格式，不是 3GPP 官方出版格式。

遇到高風險標準解讀時，引用順序應為：

```text
原始 3GPP Word / 官方出版物
    ↓
同版本官方 OpenAPI attachment
    ↓
本流程產生的 Markdown
    ↓
repository-generated navigation / summaries
```

Markdown 的價值是：

- 快速定位；
- progressive loading；
- repository search；
- agent context control；
- clause-level version control。

它不能取代在重要規範判定時回查原始版本。
