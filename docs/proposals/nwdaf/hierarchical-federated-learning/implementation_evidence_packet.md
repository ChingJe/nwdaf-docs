# Hierarchical NWDAF Federated Learning 實作證據包

日期：2026-08-27

狀態：草稿；持續查證中；testbed E2E 執行結果待補

## 1. 文件目的

本文件集中整理论文／實驗提案所需的實作與整合證據，回答三組問題：

1. 現有 Flat／Hierarchical FL 實作已完成並驗證到哪個層級；
2. 論文指標能否從現有component與event boundary可靠取得；
3. HFL 與 3GPP NWDAF 機制整合時，哪些現象屬於 standard boundary、
   free5GC-specific issue、our design choice，或 misunderstanding／non-issue。

本文件是 [Proposal 初稿](proposal_draft.md) 的supporting evidence，不取代canonical
implementation plan、testbed repository的raw run record或正式experiment specification。

## 2. 證據政策與狀態標記

### 2.1 證據層級

| 證據類型 | 本文件中的用途 |
| --- | --- |
| TS 條文 | 判斷procedure intent、角色、責任與允許的組合方式 |
| Release 18 OpenAPI YAML | 判斷 path、method、status、header、field 與 schema constraint |
| 現行實作 | 說明NWDAF／PyMTLF實際production behavior |
| Deterministic test／本機 E2E | 證明本機contract、process composition與multi-process behavior |
| Testbed 證據 | 證明指定revisions在real NRF／NF／VM environment的執行結果 |
| 設計推論 | 由規格與實作證據推導，但不是3GPP直接定義的結論 |

### 2.2 狀態標記

| 狀態 | 定義 |
| --- | --- |
| `Confirmed` | 已有直接來源或執行證據，且目前未發現反證 |
| `Reported` | 由團隊或執行者回報，尚未取得可重查的紀錄 |
| `Pending verification` | 已有候選結論，但仍需完成指定查證 |
| `Open` | 尚未形成足夠結論 |
| `Not applicable` | 經查證後不適用於目前範圍 |

同一事項可以在本機為 `Confirmed`，但在testbed仍為
`Pending verification`。本文件不得以 unit／mock／local multi-process evidence 取代
real NRF、跨 VM、SMF／UPF、UE 或 data-plane evidence。

## 3. 實作事實與 E2E 狀態

### 3.1 目前結論

| 項目 | 本機證據 | Testbed 證據 | 狀態／待補資料 |
| --- | --- | --- | --- |
| 同一套實作支援 Flat／HFL | Explicit orchestration config與本機real-process scenarios已完成 | 待確認部署版本與實際config | 本機 `Confirmed`；testbed `Pending verification` |
| Branch upper-client／lower-server dual role | 本機HFL實作、tests與multi-process E2E已完成 | 待取得runtime topology與process evidence | 本機 `Confirmed`；testbed `Pending verification` |
| Same global-round semantics | 本機流程已驗證每次lower-tier aggregation後形成upper-tier update | 待取得round-level runtime record | 本機 `Confirmed`；testbed `Pending verification` |
| HFL testbed E2E | 不適用 | 團隊回報目前正在執行 | `Reported`；結果、commands與record待補 |
| Flat／HFL正式實驗就緒度 | 本機isolated scenarios已存在，但不是正式論文實驗結果 | 待確認testbed topology、dataset與instrumentation | `Pending verification` |

本機的canonical status仍見
[Hierarchical NWDAF Federated Learning Implementation Plan](../../../plans/hierarchical-federated-learning/Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)；
testbed摘要見
[NWDAF Testbed 整合進度摘要](../../../progress/testbed_integration_status.md)。

### 3.2 Testbed 證據收集欄位

Testbed E2E 完成後，本節至少補齊下列資料：

- testbed repository branch、HEAD與working-tree狀態；
- 所有 affected component revisions與submodule gitlinks；
- VM、Root／Branch／Leaf、NRF、ADRF、SMF／UPF與資料來源 placement；
- 實際使用的 topology、capability、trigger、round、strategy與dataset設定；
- 執行 commands、開始／結束時間、Run ID及log／artifact位置；
- preparation、round、aggregation、validation、publication與cleanup結果；
- failure、timeout、retry、restart或skipped checks；
- runtime remediation，以及 remediation 後是否使用相同或更新後的 revisions 重跑；
- confirmed claims、remaining gaps與明確不可宣稱的範圍。

Raw logs、generated configs、run artifacts及完整操作紀錄由
`5G_NWDAF_Infrastructure` 保存；本文件只保存可供 proposal 引用的摘要與精確來源。
目前 workspace 中的唯讀 reference 為該 repository 舊版 `main@7d0a36c`，只用於理解
既有 Flat testbed topology、config renderer、component lock、experiments與observability
結構，不作為本次 HFL testbed 執行證據。

## 4. 指標與 instrumentation 可行性

### 4.1 初始可行性矩陣

下表先記錄候選measurement boundary；正式結論須在trace production path後更新。

| 指標 | 論文語意 | 候選measurement boundary | 目前狀態 | 待確認事項 |
| --- | --- | --- | --- | --- |
| Root-facing bytes | Flat：Client ↔ Root；HFL：Branch ↔ Root | Upper-tier training requests、callbacks與artifact transfers | `Pending verification` | producer／consumer哪一端作為authoritative counter；retry如何計入 |
| Lower-tier bytes | HFL：Leaf ↔ Branch | Lower-tier training requests、callbacks與artifact transfers | `Pending verification` | tier label與duplicate／failed transfer規則 |
| Total FL bytes | 全run的FL application payload與model／update artifact bytes | Root-facing與lower-tier raw events離線加總 | `Pending verification` | 避免同一transfer在兩端重複計數 |
| Message count | 成功與失敗的FL-related application messages | 共用event schema | `Pending verification` | request、response、callback、retry的message type分類 |
| E2E training time | Root接受trigger至最後global round aggregation完成 | Root top-level request lifecycle | `Pending verification` | 精確start／end event與clock source |
| Per-global-round latency | 同一global round的Root dispatch至Root aggregate | Root與Branch round lifecycle | `Pending verification` | upper／lower correlation與round ID mapping |
| Root／Branch aggregation time | 單次aggregation owner的執行時間 | PyMTLF aggregation boundary | `Pending verification` | 是否包含artifact load／publish |
| Final model performance | 同一held-out test set上的final model result | Existing final validation／offline evaluator | `Pending verification` | primary learning metric與dataset identity |
| Process CPU／peak RSS | 個別Root／Branch process resource usage | External process-level sampler | `Open／Optional` | shared host干擾與採樣成本 |

### 4.2 紀錄結構契約

Flat／HFL應使用同一套raw event schema。每筆FL／SBI event至少需要：

- timestamp與clock source；
- sender、receiver與containing NWDAF identity；
- topology mode與tier；
- experiment／Run ID、FL process／correlation ID與round ID；
- message type、direction與success／failure；
- application payload bytes與model artifact bytes；
- retry／attempt identity，以及是否納入paper metric。

本節只固定觀測資料需求，不預先決定必須使用application log、structured event、HTTP
middleware、artifact metadata、runner observation或外部process sampler。選擇實作方式前，
必須先確認每個值的authoritative producer、transport path與double-counting規則。

## 5. HFL 整合觀察稽核

### 5.1 統一查核格式

每項觀察使用以下欄位：

1. **Requirement**：hierarchical execution需要解決的精確問題；
2. **Spec evidence**：TS text與OpenAPI分開列出；
3. **Current implementation**：production owner、data flow、state與test evidence；
4. **Consequence**：若沒有額外處理，實際procedure或interoperability會發生什麼；
5. **Our treatment**：目前實作如何處理，以及是否為第一版限制；
6. **Classification**：`standard boundary`、`free5GC-specific issue`、
   `our design choice`或`misunderstanding／non-issue`；
7. **Status**：`Confirmed`、`Pending verification`、`Open`或`Not applicable`。

### 5.2 P1：Branch dual role／upper-lower FL process composition

- **Requirement**：確認Branch如何對Root作為FL Client、對Leaves作為FL Server，並維持兩個
  process的identity、round、callback與lifecycle mapping。
- **Spec evidence**：待查。
- **Current implementation**：待以Branch preparation、round、validation與cleanup production
  path補齊。
- **Consequence**：待查。
- **Our treatment**：待查。
- **Classification**：待定。
- **Status**：`Open`。

### 5.3 P2：NRF capability discovery與hierarchy／topology establishment

- **Requirement**：區分NRF能證明的candidate capability／endpoint資訊，與Root／operator建立
  parent-child assignment所需的額外資訊。
- **Spec evidence**：待查。
- **Current implementation**：待以Root exact-instance discovery、static topology與Branch Leaf
  re-resolution production path補齊。
- **Consequence**：待查。
- **Our treatment**：待查。
- **Classification**：待定。
- **Status**：`Open`。

### 5.4 P3：`mLModelUrl`／model bundle與cross-process mapping

- **Requirement**：釐清`mLModelUrl`在preparation／training中的標準語意，以及hierarchy
  metadata與cross-process mapping是否屬於標準內容。
- **Spec evidence**：待查。
- **Current implementation**：待以artifact publication、download、validation、Branch republish
  與typed hierarchy metadata production path補齊。
- **Consequence**：待查。
- **Our treatment**：待查。
- **Classification**：待定。
- **Status**：`Open`。

### 5.5 P4：Preparation accept／reject／status expressiveness

- **Requirement**：確認ML Model Training preparation的response、notification與status能否表達
  hierarchical subordinate preparation結果。
- **Spec evidence**：待查。
- **Current implementation**：待以Leaf outcome、Branch result bundle與Root admission production
  path補齊。
- **Consequence**：待查。
- **Our treatment**：待查。
- **Classification**：待定。
- **Status**：`Open`。

### 5.6 P5：Server-driven selection與Client-initiated participation

- **Requirement**：確認Release 18 NWDAF FL procedure由誰選擇／邀請participants，以及是否存在
  FL Client主動加入既有Server process的標準mechanism。
- **Spec evidence**：待查。
- **Current implementation**：待以Root topology、NRF resolution與Training Subscribe initiation
  production path補齊。
- **Consequence**：待查。
- **Our treatment**：待查。
- **Classification**：待定。
- **Status**：`Open`。

## 6. 主張與證據交接

| 候選主張 | 必要證據 | 目前狀態 | Proposal處理 |
| --- | --- | --- | --- |
| 多個既有NWDAF FL processes可組成Root–Branch–Leaf execution | Spec composition analysis、production trace、local與testbed E2E | Local部分已確認；testbed待補 | 維持candidate claim |
| Branch可安全維持upper-client／lower-server dual role | Process mapping、correlation isolation、round／failure／cleanup evidence | Local部分已確認；audit待補 | 不先擴張為標準已定義行為 |
| HFL降低Root-facing communication | Frozen byte semantics與正式Flat／HFL experiment | 尚無正式結果 | 不提前寫成已證實結論 |
| HFL引入lower-tier communication、Branch aggregation與latency cost | Frozen measurement semantics與正式實驗 | 尚無正式結果 | 以research question／hypothesis表述 |
| 3GPP management／FL models未直接表達hierarchical orchestration | Release-specific TS／OpenAPI audit | 待完成P1–P5與OAM evidence reconciliation | 暫列practical implication候選 |

## 7. 待辦事項

- [ ] 取得本次testbed run的repository HEAD、component revisions與working-tree狀態。
- [ ] 取得testbed topology、commands、Run ID、logs與result summary。
- [ ] Trace完整Flat／HFL message與artifact transport path。
- [ ] 固定bytes counting、retry、failure與double-counting規則。
- [ ] 確認E2E與per-round的authoritative start／end events。
- [ ] 確認final learning-outcome primary metric與held-out dataset identity。
- [ ] 完成P1–P5 spec、implementation與classification audit。
- [ ] 將可成立的claim與limitation回填Proposal初稿。

## 8. 更新紀錄

| 日期 | 更新 |
| --- | --- |
| 2026-08-27 | 建立初稿；固定證據層級、testbed收集欄位、instrumentation矩陣、P1–P5模板與主張交接 |
