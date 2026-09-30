# Development policy 與 AGENTS.md 漸進式揭露重構詳細計畫

日期：2026-09-30

狀態：計畫審查已同意；計畫文件提交已批准；尚未執行 policy 或 AGENTS.md 重構。

## 1. 目的與本次交付邊界

將 workspace 指令整理為「精簡常駐指引、短 policy 入口、按需求載入的規則模組」。
目標是減少無關內容的重複載入與重複維護，同時保留既有授權、範圍、架構、
review 與驗證要求。採用 skills 的漸進式揭露方式，不建立新的 skill 安裝或執行機制。

依使用者追加要求，本計畫同時調整執行方式：規則依任務載入，重讀與重驗依
需求變更或證據缺口觸發。已完成的 review 與驗證可以支持後續交付；批准是
操作授權，不是重新開始整套流程。規則以正向適用條件描述，保留授權與驗收深度。

本次授權只涵蓋規劃與撰寫本文件及分類索引；不代表已授權執行本計畫中的重構，
也不授權 stage、commit 或 push。未來執行仍需使用者指示，交付與 Git 操作仍走各自閘門。

相關來源：

- workspace 根目錄 `AGENTS.md`：repository 邊界、授權、讀取與執行規則。
- [NWDAF Development Policy](../../development_policy.md)：現行開發與 review 規範。
- [文件分類慣例](../../README.md)：採文件類型優先的分類方式。
- `free5gc-dev-skill/SKILL.md`：既有 free5GC 技術工作路由；本計畫不修改其內容。

本計畫放在 `docs/plans/development-workflow/`，因為修改對象是 workspace 開發流程，
不是 NWDAF、AnLF、MTLF 或其他 NF 的功能。

## 2. 現況與已確認問題

### 2.1 文件現況

- `development_policy.md` 目前有 825 行、12 大節，包含規劃、架構、契約、語言、
  remediation、decision gate、finding admission、review、build、證據與交付流程。
- `AGENTS.md` 同時包含 repository map、task routing 與大量詳細 policy 規則。
- 多份 plans 已引用 `development_policy.md`，部分仍在使用的計畫明確要求完整重讀。
- workspace 根目錄不是 Git repository；`AGENTS.md` 不會自動包含在 `nwdaf-docs` commit。
- `nwdaf-docs` 在本次規劃開始時工作樹乾淨；執行重構時須重新確認，不能沿用此狀態。

### 2.2 重複與範圍落差

| 現況 | 問題 |
| --- | --- |
| AGENTS.md 的 Repository Map 與 Task Routing | 重複列出相同 repository 與責任 |
| 兩份文件的 User Review／Commit Gate | 共同規則有多個維護位置 |
| 兩份文件的 Documentation Language Gate | 完整選擇與檢查流程重複 |
| AGENTS.md 的 End-to-End Feasibility 與 policy §2.2 | 跨邊界資料流要求高度重疊 |
| 兩份文件的 Reference Order 與證據要求 | 來源順序與推論限制重複 |
| AGENTS.md 與 policy 的 fresh-read 規則 | 拆檔若未調整路由，仍可能要求載入全部模組 |
| policy 開頭只列 NWDAF、PyAnLF、PyMTLF | 與 AGENTS.md 已要求其他 NF repository 讀取 policy 的範圍不一致 |
| policy §12 重述前面各節流程 | 容易形成另一份需同步維護的詳細規範 |
| 每個 follow-up、action 與 commit checkpoint 都要求 fresh-read | 對話與授權切換容易觸發同一組規則反覆載入 |
| 完成核對、proposal 與提交執行沒有明確區分證據責任 | 容易重跑驗證、重建對照表與反覆輸出完整 status／diff |

這些同時是文件組織與執行成本問題。依本次要求調整無條件重複流程，同時保留
驗收內容、證據品質、例外與授權邊界；同名條文仍需核對獨有細節。

## 3. 修改範圍與非目標

### 3.1 預計修改位置

| 位置 | 預計責任 |
| --- | --- |
| workspace `AGENTS.md` | 精簡常駐指引、合併 repository 路由、導向 policy |
| `nwdaf-docs/docs/development_policy.md` | 保留既有入口路徑，改成共同原則與情境路由 |
| `nwdaf-docs/docs/development-policy/` | 新增六份規則模組 |
| `nwdaf-docs/README.md` | 調整正式規範入口說明，不複製路由表 |
| `nwdaf-docs/docs/plans/` 中受影響的現行指令 | 引用、舊章節定位、讀取與重複驗證要求的必要修正 |
| 本計畫與分類 README | 記錄重構交付狀態，不建立額外追蹤系統 |

其他 repository 不在預計修改範圍。若發現其他 repository 有實際阻擋新路由的
指令，先回報具體文件與衝突，再取得擴大範圍的決定。

### 3.2 非目標

- 不修改 production code、OpenAPI、TS 正文、runtime 契約或測試行為。
- 不重構 `free5gc-dev-skill/`，也不為純文件工作啟動完整 free5GC 開發流程。
- 不新增 helper、validator、hash、manifest、build ID、版本登錄或 CI gate。
- 不安裝 skill、不新增 `SKILL.md`、不設計自動載入器。
- 不把每一條規則拆成單獨文件，不替每個 repository 複製一組 policy。
- 不重寫歷史實作紀錄、review ledger 或 archive 以追求格式一致。
- 保留原有接受標準、Git 授權流程與實作範圍；本次調整的是載入時機與證據沿用方式。

## 4. 目標結構與規則所有權

### 4.1 文件配置

```text
AGENTS.md
nwdaf-docs/
└── docs/
    ├── development_policy.md
    └── development-policy/
        ├── planning.md
        ├── architecture.md
        ├── implementation.md
        ├── review.md
        ├── documentation.md
        └── delivery.md
```

`development_policy.md` 是唯一完整路由表；不另外建立模組 README 或重複索引。
模組開頭直接寫明適用條件、責任與必要的條件式補讀連結。

### 4.2 AGENTS.md 的常駐內容

保留以下內容，因為不應依賴 agent 已先載入某個模組才知道：

1. 合併後的 repository 地圖：位置、主要用途、唯讀 reference 邊界。
2. 各 repository 獨立；在進入修改或 Git 操作時確認工作樹邊界，保留使用者變更。
3. 討論、診斷與 review 不自動授權修改；實作授權不等於 Git 操作授權。
4. 使用者 review 前保持 intended changes unstaged／uncommitted 與計畫開放。
5. 必須先提出 commit proposal 並獲明確批准；push 與歷史改寫另需授權。
6. 所有 script／code execution 與 network operation 的現行提權要求。
7. 回覆精簡但須有證據、區分事實／推論／設計，不假設跨邊界資料存在。
8. code 與 API 文字的語言底線；文件須遵循既有文件或系列語言。
9. 何時進入 development policy、何時觸發 free5GC skill 的簡短條件與排除條件。
10. 適用指引缺失或衝突時回報，不把「沒讀到」當作沒有規則。

詳細 proposal 欄位、語言檢查步驟、端到端資料流清單與 review 流程移出。
授權摘要是常駐安全提醒，不是第二份完整規則；必須連到正式細節。

### 4.3 Policy 入口的責任

- 說明適用範圍，與現行 AGENTS.md 已列出的 implementation repository 對齊。
  共通規則可適用多個 repository，但 NWDAF 專用命令與 commit 格式範圍不得擅自擴大。
- 保存範圍不擴張、避免過度工程、禁止無依據完成宣稱等共通原則。
- 保存一份精簡的證據順序及證據使用方式，涵蓋純分析，不強迫讀 implementation 模組。
- 指明目前實作以 Release 18 為基準；Release 19／20 只有在任務明確需要時使用，
  不混用 release。OpenAPI、TS 與 generated code 的責任不同。
- 保存唯一任務／動作路由表，以及情境補讀、證據沿用與必要重驗的條件。
- 保存 decision gate 的共通條件與回報要求，讓任何模組都不需要複製一份。
- 區分工作單位續行、情境補讀與真正改變 scope；不要求重新讀所有無關技術資料。

讀取方向由 AGENTS 指向 policy，再由 policy 指向適用模組。入口承接已載入的
workspace 指令；模組按當前問題連到必要細節，維持單向、條件式路由。

### 4.4 六個模組的責任

| 模組 | 唯一主要責任 |
| --- | --- |
| `planning.md` | slice 定義、既有流程 baseline、語意延伸、非目標與延後項目分類 |
| `architecture.md` | owner、Go package、跨邊界 producer／transport／consumer、契約層級、experimental schema |
| `implementation.md` | code quality、行為安全、test-first remediation、hash 預設限制、repository-native verification |
| `review.md` | finding admission、severity、initial／targeted review、plan conformance、完成前 reconciliation |
| `documentation.md` | 文件語言選擇與完整檢查、canonical plan、review ledger 與文件責任分工 |
| `delivery.md` | 使用者 review handoff、commit proposal、批准範圍、Git 操作限制、commit message |

共通 workflow 只留下短步驟與連結。模組不複製別人的完整規則；例如 review
確認需 remediation 時才補讀 implementation，而不是每次 review 一律要求執行修正。

相同情境下已載入且仍適用的規則可沿用；路由表描述首次需要的資訊，而不是
每次說話都重新執行的檢查清單。

## 5. 現行規則搬移對照

### 5.1 Development policy

| 原章節 | 目標位置與處理 |
| --- | --- |
| 前言與適用範圍 | policy 入口，修正 repository 範圍描述落差 |
| §1、§1.1–1.4 | `planning.md`；最小完整 slice 原則在入口保留短提醒 |
| §2、§2.1、§2.2 | `architecture.md`，合併 AGENTS 的資料流要求 |
| §3.1–3.4 | `architecture.md`；保留 standard 與 private／experimental 的區別 |
| §4 code quality 部分 | `implementation.md` |
| §4 文件語言部分 | `documentation.md`；常駐語言底線留 AGENTS |
| §5、§5.1、§5.2 | `implementation.md`，保留 in-scope remediation 與擴大範圍時的停止條件 |
| §5.3 | `implementation.md`；入口有避免非必要 hash 的短提醒 |
| §6、§6.1 | policy 入口的共通 decision gate |
| §7、§7.1 | `review.md`；引用 planning 的延後分類，不重述分類全文 |
| §8.1–8.3 | `review.md`；§8.2.1 改為依當前要求與有效證據的最終核對，載入條件由入口持有 |
| §8.4 | `documentation.md`；review 模組按需連結 |
| §9 | `implementation.md`；文件專用 diff check 在 documentation 直接列明 |
| §10、§10.1 | policy 入口的精簡證據原則，保留反證、準確引用與事實／推論區分 |
| §11.1–11.3 | `delivery.md`；完整保留既有批准語意與原 commit 格式適用範圍 |
| §12.1 | 入口保留短 workflow；細節由各模組持有，區分驗證、proposal 與提交執行 |
| §12.2 | `documentation.md` 與 `delivery.md` 按責任拆分，不保留第二份完整交付流程 |

### 5.2 AGENTS.md

| 原區塊 | 目標處理 |
| --- | --- |
| Repository Map、Task Routing | 合併為一份地圖；具體 repo 責任與 Python backend 定位不遺失 |
| Repository Boundaries | 保留常駐底線，刪除重複 repo 列表；工作樹確認配合修改與 Git 操作時機 |
| User Review Gate | 保留必要摘要；使用者提出提交目前成果時視為 review 已確認，詳細交付與狀態流程移到 `delivery.md` |
| Commit Approval Gate | 保留明確授權底線；完整 proposal、重批與歷史操作限制移到 `delivery.md` |
| Continuous Work Units | 定義移到 policy 入口；一般續行沿用規則，壓縮後按任務重讀規則與計畫，其他變更或缺口按需補讀 |
| Final Completion Re-read Gate | 改為最終符合性核對；依已建立的要求與證據核對，遇到變更或缺口時補讀與補驗 |
| free5GC Skill Scope | 留簡短觸發與排除條件；選技術 reference 仍由既有 skill 負責 |
| Reference Order | 移到 policy 入口，AGENTS 只指向它 |
| Reasoning And Response Discipline | 留短回覆與證據底線；詳細證據要求移到入口 |
| End-to-End Feasibility Discipline | 併入 `architecture.md`，保留雙向流程、角色限定與未知輸入回報要求 |
| Execution And Safety | 保留資料安全、提權與語言底線；移除重複流程 |
| Documentation Language Gate | 完整內容移到 `documentation.md`；修改後完整檢查的結果可支持後續 proposal 與提交 |

每次合併要逐條核對來源，特別是某一份文件獨有的例外。搬移對照是人工 review
工具，不新增永久 machine-readable registry 或另一本完整規則副本。

## 6. 漸進式讀取規則

### 6.1 基本流程

1. 進入 workspace 任務時，從適用指令確認任務、repository、操作權限與目前動作。
2. 需要開發規範時使用 policy 入口定位相關規則，涵蓋規劃、review、文件修改與技術分析。
3. 首次觸發某模組的條件時，讀取該模組全文；後續在相同情境沿用已掌握的內容。
4. 任務同時涉及多個層面時，載入其適用模組的聯集；補讀連結以當前需求為條件。
5. 跨入新契約、owner 或操作階段時，補足尚未掌握的規則再行動。
6. technical reference 依當前結論需要直接查閱；既有證據與新問題的適用性一併判斷。

入口用簡短的適用條件說明「何時讀、讀哪裡」。流程保持正向描述，讓需求本身
決定讀取範圍；關鍵授權與資料安全限制仍保留明確文字。

### 6.2 任務路由驗收情境

以下是各情境首次需要的模組。AGENTS、入口與 active plan 中已掌握且有效的內容
可沿用；模組按情境補足。free5GC skill 仍按標準／Go NF 邊界觸發。

| 情境 | 適用模組 | 補充條件與交付責任 |
| --- | --- | --- |
| 只問本地規格或 code 現況 | 使用入口證據原則 | 依問題直接讀來源；以說明交付 |
| 撰寫 substantial implementation plan | planning、documentation | 架構／跨邊界決策另讀 architecture |
| 小型、已批准的內部 code 修改 | implementation | slice 或 owner／契約改變時補讀 planning／architecture |
| 延伸既有 production flow | planning、implementation | baseline 語意與 owner／契約變更另讀 architecture |
| 新 package、跨 repo／process 或契約修改 | architecture | 規劃加 planning，實作加 implementation；skill 按技術邊界觸發 |
| 使用者只要求 code review | review | 架構議題加 architecture；以 finding 與證據交付 |
| 已授權實作中的 finding 修正 | review、implementation | 改契約／owner 時加 architecture 並走 decision gate |
| 純文件修改與文件 review | documentation | findings／完成核對加 review；驗證以文件為對象 |
| 準備完成與使用者 review handoff | review、delivery | 核對當前要求與有效證據，整理一次結果與缺口 |
| commit proposal、獲批 stage／commit、push | delivery | 沿用驗證證據，分別確認 proposal、提交範圍與操作授權 |
| 上下文壓縮後恢復工作 | 依當前任務與階段重新讀取對應規則 | 重讀 active plan 的相關要求、驗收、決策與進度，恢復後再續行 |

架構可行性諮詢即使不改檔，涉及跨邊界設計時仍須讀 architecture。
讀到 remediation 流程不新增修改權限；review-only 與已授權實作的 follow-up 必須區分。

### 6.3 情境補讀與證據沿用

讀取與重讀依以下需求觸發：首次進入相關工作、經歷上下文壓縮、任務／repository／
技術邊界改變、收到規則或 plan 已修改的資訊，或現有上下文不足以可靠決策。補讀範圍對應
受影響的規則與來源；單純續行、詢問進度或批准既有 proposal，沿用同一工作單位的內容。

完成前核對涵蓋 narrative requirements、驗收項目、required commands、baseline
disposition 與 approved deferral。已建立的對照表與驗證結果是這次核對的基礎；
要求有變動或證據不足時，回到相關原文補查，再補齊缺口。

測試、完整文件語言檢查與 review 結果，適用於當時檢查的內容及條件。相關內容、
依賴、工具／環境、驗收要求有實質變更，或無法確認既有證據對當前狀態仍適用時，
依影響範圍重新檢查。重驗依需要涵蓋 focused 或 required full verification；
例如只更新 plan 的交付狀態時，核對該文件即可，production test 證據可繼續使用。

證據直接保留在既有工作上下文、必要的 plan 紀錄與交付摘要中。新 agent 或
上下文不足的續行，從必要來源恢復要求與結果；高優先級 runtime／skill 的
明確讀取要求仍依其規定處理。

#### 壓縮後的任務恢復

上下文壓縮後，先以最新使用者目標與摘要定位正在進行的任務、repository、slice
及工作階段，再從原文重新建立這個任務的執行依據：

1. 重新讀取適用的 workspace 指令、policy 入口及當前任務對應的規則模組。
2. 重新讀取 active plan 中相關的目標、範圍、既有決策、驗收條件、required commands、
   進度與未完成項目；涉及 parent plan 的承諾時連同對應內容查閱。
3. 將摘要中的待辦與已完成工作對照原文，確認下一步、證據缺口及實際授權狀態後續行。

摘要用來導航，規則與計畫原文用來確認要求；只有在恢復所需資訊時才擴大讀取範圍。
這個步驟恢復的是任務依據，已完成的測試與 review 仍依本節的證據適用條件沿用。
若當前任務沒有 active plan，依使用者要求與適用規則恢復，不另造計畫或恢復紀錄。

### 6.4 驗證、批准與提交的責任分離

| 階段 | 應完成的工作 | 可沿用的內容 |
| --- | --- | --- |
| 實作與 review 收斂 | 對最終內容完成要求的驗證、符合性核對與語言檢查，整理結果與缺口 | 已覆蓋同一最終內容與條件的 focused／full 結果 |
| 使用者 review 與 commit proposal | 使用者提出提交目前成果時確認 review，更新相關文件狀態，再說明範圍、完整 commit message、驗證與排除項目 | 已完成的 review、驗證與符合性對照 |
| 批准後 stage／commit | 確認當前工作樹及批准範圍，stage 指定內容、核對實際 staged changes、commit | 仍適用的驗證證據與已批准 proposal |
| 獲准 push | 確認目標 branch／remote 與待推送 commit，執行推送並回報結果 | 已建立 commit 的驗證與批准資訊 |

提交檢查保留 repository 邊界與實際內容核對；status、diff 的查閱服務於這個目的。
一般交付以短摘要呈現，詳讀差異依變動與風險範圍決定。若批准後出現實質內容或
scope／message 變動，針對差異補 review／驗證，並依原授權閘門重新取得必要批准。

工作流程條文一併檢查載入、最終核對、語言檢查、full verification、status／diff
與實作紀錄的觸發時機。以「條件成立時執行、證據仍有效時沿用」表達，讓每個階段
只承擔自己新增的責任。

#### Commit 請求與文件狀態同步

使用者明確要求提交目前交付成果，例如「進行 commit」或「將計畫進行 commit」，
代表已審查並同意該成果，可進入 commit proposal 階段。這與對已提出 proposal 的
批准是兩個不同事件；Git 操作仍以獲批的 repository、scope 與完整 message 為準。
詢問 commit 方法、假設情境或討論未來提交，不視為目前成果的 review 確認。

收到此類請求後，先同步本次成果涉及的 active plan、實作／review 紀錄與相關索引
中的當前狀態，例如 `User Review Confirmed`、`Commit Pending Approval`，再提出 proposal。
狀態修改納入同一提交範圍；更新目前狀態，保留歷史 review 與驗證事實。

文件狀態分別反映計畫審查、實作、驗證、外部驗收與提交進度。例如計畫本身已同意
但重構尚未開始，應記錄這兩個事實；仍有必要外部驗證或後續工作時保留開放項目。
提出 commit 請求只確認當前成果的 review，不表示所有階段已完成或已取得 push 授權。

## 7. 引用與歷史文件的遷移策略

1. 保留 `docs/development_policy.md` 路徑，既有只連入口的文件不需大量改寫。
2. 用 `rg` 盤點入口引用、舊章節／anchor、fresh-read、完整重讀與階段性重複驗證措辭；命中只是線索，
   需直接讀上下文，不以 regex 結果判定文件是否仍在使用。
3. 分成「目前有效指令」、「已完成工作的歷史描述」、「已歸檔資料」三類。
   文件 status 不是唯一判準：已完成計畫中的後續工作指令仍可能有效。
4. 目前有效指令若引用已移動的特定規則，改成模組或穩定 heading 連結。
5. 目前有效的無條件重讀、從零重建對照表與每個 checkpoint 重跑同一驗證要求，
   依本次方向改成情境／變更驅動；保留 plan-conformance 與 required verification 的內容。
6. 計畫若有具體驗收理由要求特定時機重新驗證，保留該條件。需要變更驗收本身
   或與更高優先級指令衝突時，列出影響並取得決定。
7. 歷史紀錄如「當時已完整重讀」保持原文；必要導覽修正不得重寫過去驗證結果。
8. 不預設新增舊 section-number alias。若確有有效 anchor 依賴，優先修正 caller；
   不能安全處理的剩餘依賴列為開放項目，不留一整份舊 policy 當第二主來源。
9. `nwdaf-docs/README.md` 只連正式入口；workspace 路徑在 AGENTS 使用相對 workspace
   的位置，policy 與模組採 repository 內相對連結，不寫 `/home/...` 絕對部署路徑。

## 8. 執行步驟與檢查點

### 8.1 檢查點 A：範圍與逐條盤點

1. 進入重構時掌握現行 AGENTS、development policy 與本計畫的適用要求，確認使用者授權。
2. 查 `nwdaf-docs` status，列出 unrelated changes；再次確認 workspace 根目錄的 Git 邊界。
3. 依第 5 節逐條比較，補足獨有規則、現行 caller 與真正衝突。
4. 建立工作中的舊規則到新位置對照，不新增 helper 或獨立追蹤檔。
5. 依本計畫調整載入與證據沿用方式；遇到其他規則語意變更、模組數量擴張或
   其他 repository 修改時，先回報決策需求。

### 8.2 檢查點 B：先建模組，再切換入口

1. 先新增六份英文 policy 模組，保持既有 policy 系列的語言。
2. 將完整條件、例外與禁止事項搬到指定 owner，合併重複規則。
3. 將現有 policy 改為英文短入口，加入需求驅動的路由、證據沿用與共通 decision gate。
4. 精簡英文 AGENTS，合併 repository 路由並連到已存在的正式規則。
5. 同一工作切片內完成入口切換與去重；不得先刪除規則，留下不存在的必讀目標。
6. 主指令調整期間仍遵循現行有效 user／runtime 要求，不能藉編輯授權條文擴權。

### 8.3 檢查點 C：Caller、情境與規則 review

1. 只修第 7 節判定需要改的 caller 與 repo README 說明。
2. 用第 6.2 節每個情境人工演練讀取路徑與操作權限，確認不漏必要模組。
3. 反向核對全部舊規則都有新 owner 或明確的去重說明，且短摘要沒有取代獨有細節。
4. 確認路由有清楚的適用條件，續行、批准與交付可沿用有效證據；檢查類似無條件
   重複流程是否已依第 6.4 節統一調整。
5. 對已確認的文件缺陷做最小修正與 targeted follow-up review；不擴張到技術實作。

### 8.4 檢查點 D：最終核對與使用者交付

1. 依第 6.3 節沿用與補足證據，完成第 9 節所有驗收條件的核對。
2. 在最終文件內容完成獨立語言一致性檢查，結果支持接下來的 review handoff 與提交。
3. 確認最終 diff check 的覆蓋範圍與結果，清楚區分 workspace AGENTS 與 docs repository。
4. 在本計畫簡短記錄實際修改、驗證與未關閉項目，不建立額外 review ledger。
5. 保持狀態為 `Ready for User Review` 或同等開放狀態，changes unstaged／uncommitted。
6. 使用者確認 review 或明確要求提交目前成果後，依第 6.4 節更新相關文件狀態並
   沿用結果準備 commit proposal；獲批再 stage／commit，push 另行批准。
7. workspace AGENTS 需獨立報告 diff 與保存狀態，不能聲稱已包含於 docs commit。

## 9. 驗收與驗證矩陣

### 9.1 重構驗收條件

| ID | 必須成立的結果 | 直接驗證方式 |
| --- | --- | --- |
| A1 | AGENTS 地圖與 task routing 合一，repo／reference 邊界不遺失 | 對照原 repo 列表與角色說明 |
| A2 | 既有 policy 入口可用，六份模組存在且有明確適用條件 | 直接讀入口與六份模組、檢查本地連結 |
| A3 | 原 12 節及 AGENTS 詳細規則均有 owner；獨有例外未遺失 | 第 5 節對照與逐條反向 review |
| A4 | 精簡沒有弱化 review、commit、push、歷史操作與提權限制 | 授權負面情境逐一核對 |
| A5 | 第 6.2 節全部情境可導向最小但完整的適用集合 | 逐情境人工追蹤，列出實際必讀文件 |
| A6 | 一般續行沿用內容；壓縮後按任務重讀規則與計畫；變更或缺口按需補讀 | 演練一般續行、壓縮恢復、新邊界與規則變更，核對兩種原文的恢復範圍 |
| A7 | 有效舊引用／完整重讀要求已處理；歷史敘述未被改寫 | `rg` 搜尋後逐命中分類與上下文 review |
| A8 | policy 適用範圍對齊，repo 專用規則與 free5GC 排除條件保留 | 對照 AGENTS、入口與模組範圍 |
| A9 | 無 helper、hash、manifest、安裝機制或新依賴 | 最終新增檔案與完整 diff review |
| A10 | changed documents 語言一致、diff check 通過、改動未暫存 | 完整文件語言檢查、Git 狀態與 diff check |
| A11 | commit 請求確認 review 並同步文件狀態；proposal 與提交沿用有效驗證 | 演練 commit 請求、狀態更新、proposal 與批准後提交，確認各階段責任 |
| A12 | 實質變更會補足相應 review／驗證，批准範圍仍正確 | 演練 code／依賴／環境／驗收改變與無關狀態更新 |
| A13 | 路由、完成核對、語言與 full verification 等類似指令採一致觸發方式 | 完整 diff review，確認條件式正向措辭與證據沿用 |

授權負面情境至少包含：`continue` 不算 commit 批准、proposal 前的批准不算
目前 proposal 批准、變更 commit scope／message 必須重新批准、commit 不授權 push，
review-only 不授權修正、缺少規範不能跳過、必須提權的命令不能換工具繞過。
同時驗證明確的 commit 請求可確認 review，而詢問／假設情境維持原 review 狀態；
文件狀態更新後仍保留未完成工作，stage／commit 等待目前 proposal 的批准。

不設定硬性行數配額；只要讀取量降低是靠去重與按需路由，而不是刪除必要規則，
即可接受。保留所有模組的總文字量不必最小，但常駐入口不得重塞完整細節。

### 9.2 驗證方法

- 使用直接讀檔、`rg` 搜尋與人工規則／情境對照；不撰寫驗證 helper。
- 對最終修改執行 `git diff --check`，review 時掌握完整 intended diff；工作樹與
  staged 範圍在對應操作時確認，後續交付沿用結果並補查新差異。
- 對 workspace AGENTS 使用 `git diff --no-index` 比較原始內容與修改結果，
  以直接保存的暫存原始副本為比較來源，不新增持久備份或完整 policy 副本。
- Git 的一般 diff 不涵蓋 untracked 新文件；新文件要直接完整閱讀，並用
  `git diff --no-index --check /dev/null <new-file>` 檢查。退出碼須結合診斷判讀，
  不能把「有差異」當作 whitespace failure。
- 本地連結以實際目標與 heading 核對；預定文件路徑不偽裝為已存在的可用連結。
- 壓縮恢復演練包含已有 active plan 與無 active plan 的任務；確認能從相關原文恢復
  目標、驗收、進度與下一步，並區分恢復閱讀與驗證證據的適用性判斷。
- 純文件重構不執行 Go／Python code tests，也不宣稱驗證 agent 每次都一定遵從路由。
  人工演練證明文件可導覽且規則完整，未來實際 agent 載入行為不是本切片的已驗證結果。

### 9.3 計畫撰寫與未來重構分開驗收

本次計畫撰寫的交付只需確認分類位置、範圍、搬移對照、路由、遷移策略、
執行順序與驗收矩陣齊全，文件語言與 diff check 通過。A1–A13 是未來重構的
接受條件，不得因本文件已寫好就標示已滿足。

## 10. 風險與停止條件

- **規則遺失**：同名規則可能有不同例外；逐條合併，不以刪掉重複 heading 當完成。
- **漏讀交付規則**：常駐授權底線與首次進入交付時的 delivery 路由同時存在。
- **模組互相載入**：跨模組只使用情境式連結，不新增「全部必讀」依賴鏈。
- **計畫指令衝突**：區分無條件重複流程與有驗收理由的要求；本次調整前者，後者保留或另行決策。
- **兩層摘要漂移**：AGENTS 只保留最小安全提醒，詳細規則只有一份；一起 review 摘要與 owner。
- **根目錄不受版本控管**：AGENTS 修改是本地交付，不自動新增 workspace Git repository。
- **證據失效**：實際 staged 內容與已驗證變更需一致；變動、環境差異與未知來源要依第 6.3 節補查。
- **錯把結構變更當規則變更授權**：如需降低驗證、改批准方式或更換技術規格基準，
  回報原規則、衝突、選項與影響，取得決定後才繼續。

## 11. 目前交付狀態

- 本次文件範圍為本詳細計畫與分類 README；計畫已納入需求驅動載入、壓縮後規則與
  計畫重讀、證據沿用、commit 請求的 review 確認與文件狀態同步。
- AGENTS、development policy、既有 plans 與 production repository 尚未依本計畫修改。
- 使用者已明確要求提交本計畫，計畫 review 已確認；本次文件提交 proposal 已批准。
- 正式規範重構尚未開始，重構驗收 A1–A13 尚未執行。
- 已批准的提交範圍僅包含本詳細計畫與分類 README；push 尚未授權。
