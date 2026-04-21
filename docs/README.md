# NWDAF Project Documentation

本目錄 (`.agent/docs`) 包含了 NWDAF 專案的所有文件、計畫、問題追蹤與知識庫。

## 目錄結構策略 (Directory Structure)

專案文件採用 **「Type-First (文件類型為主) -> Component (系統元件) -> Feature (功能特性)」** 的三層式結構。

### 主要分類 (Main Categories)

* **`plans/` (計畫與設計)**
  * 存放即將實作的計畫、系統設計圖、API 整合設計，以及對外部模組（如 Daisy, ADRF 等）的實作要求。
  * **用途**：尋找「接下來要做什麼」、「架構該怎麼設計」。

* **`issues/` (問題記錄與重構追蹤)**
  * 用來記錄複雜的 Bug 調查（Root cause analysis）、跨 NF 整合發生的問題（如 UPF 時間同步），以及針對技術債的重構報告。
  * **用途**：尋找「為什麼發生這個問題」、「過去是怎麼修復某個 Bug 的」。

* **`notes/` (知識沉澱與筆記)**
  * 存放已完成機制的說明、全域架構概觀、合規性報告（Compliance Reports）、整體策略，以及開發技術的學習筆記。
  * **用途**：作為專案的知識庫、策略參考與機制原理說明。

* **`progress/` (進度報告)**
  * 存放專案整體的進度報告、里程碑或階段性總結（例如：`project_progress.md`）。
  * **用途**：快速掌握專案目前開發到哪個階段。

---

## 檔案歸檔慣例 (File Hierarchy Convention)

新增文件時，請依照以下的資料夾命名慣例放置檔案：
`[文件類型 (Type)] / [模組元件 (Component)] / [功能特性 (Feature)] / 文件名稱.md`

**範例：**
如果您要新增一篇關於 SMF 模組中資料收集功能的筆記，請將其放在：
`.agent/docs/notes/smf/data_collection/smf_data_collection_mechanisms.md`

*(註：官方的 3GPP 規格書或外部標準參考資料，統一存放於與 `.agent/docs/` 同層級的 `specs/` 目錄下。)*