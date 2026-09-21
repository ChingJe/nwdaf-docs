# Slice 3 — 直接重掛 Leaf 與混合深度訓練實作計畫

日期：2026-09-21

狀態：本地實作、驗證與提交已完成；正式 testbed 驗收待執行。Go／PyMTLF 的候選欄位改名、直接重掛 Leaf 與混合深度聚合，以及本地 runner 的設定與紀錄格式更新均已提交。本地真實程序已分別通過 smoke、E0、E1、E2a、E2b；正式 testbed 的五個配對 seeds、故障注入與論文統計尚未執行，不能據本地短程測試宣稱 testbed 驗收完成。

本 Slice 對應 [實驗能力設計的第 3 項](../Hierarchical%20FL%20Experiment%20Capability%20Design.md)及 [E2a／E2b 實驗情境](../Hierarchical%20FL%20E0-E2b%20Experiments%20and%20Testbed%20Context.md)。本文件將已確認的行為轉成實作邊界、資料流與驗收項目。沿用原本的遞迴拓樸語意，但候選 payload 欄位統一改為 `flTopology`／`flTopologyReport`；不採用論文附件 B 的其他欄位。

### 候選 schema 命名同步

本次只移除專案新增 JSON payload 欄位的 `x-` 前綴，不更動欄位型別、語意、`suppFeats` 協商或 3GPP 原有欄位。候選 OpenAPI 已改名；Go／PyMTLF 的工作樹修改同步切換發送、接收、驗證與實驗紀錄，本地真實程序亦已完成跨程序交換驗證。

| 既有 payload 欄位 | 新 payload 欄位 | 訊息位置與方向 |
| --- | --- | --- |
| `x-flTopology` | `flTopology` | Create／PUT／PATCH：上層 NWDAF → 直接子節點 NWDAF |
| `x-retainedResultReq` | `retainedResultReq` | Create／PUT／PATCH：上層 NWDAF → 直接子節點 NWDAF；本 Slice 不啟用 retained-result 復用 |
| `x-flTopologyReport` | `flTopologyReport` | Notify／`immReport`：直接子節點 NWDAF → 上層 NWDAF |
| `x-retainedResultStatus` | `retainedResultStatus` | Notify／`immReport`：直接子節點 NWDAF → 上層 NWDAF；本 Slice 不啟用 retained-result 復用 |

Go NWDAF 須同步修改 Model Training 的 request／patch／notification JSON 映射、原始訊息解析與驗證錯誤路徑；PyMTLF 須同步修改 wire aliases、發送／接收處理、實驗事件摘錄與相關測試／範例。Root、Branch、Leaf 各自的 Go↔PyMTLF 私有路徑與跨 NF SBI 都要使用同一組新名稱；不得只更改其中一端。現階段快速迭代不保留舊名稱的雙格式相容讀取。`x-stage3-rules` 等 OpenAPI 文件擴充不是 JSON payload 欄位，既有 Go↔PyMTLF 訂閱 ID header 也不在此改名範圍。合約同步與 E0／E1 回歸通過後，才以新欄位驗證 E2a／E2b。

## 1. 共同起點

原始拓樸為 Root→A／B／C，每個 Branch 各管理兩個 Leaves。訓練進行中，A 永久失效，且本情境**不由替代 Branch 接手**。A1／A2 是否仍可用，由各實驗情境決定。Root 在修復期間可繼續使用未受影響的 B／C 訓練，但能否接受該輪仍須符合當時的訓練條件。

A 失效後，Root 依拓樸設定最外層的 `on_branch_failure` 選擇由另一個 Branch 接手，或由 Root 直接接手原 Leaves；E2a／E2b 採後者。這是 Root 的內部決策，不新增 Model Training 的協定欄位。各 Branch group 原有的候選、Leaf 清單、policy 與 strategy 仍維持各自用途，不承載此修復選項。

選擇直接接手時，Root 對 A1、A2 **並行發起**各自的新訂閱與 preparation，不等 A1 完成才向 A2 發起，也不因已達 Root 的 `minAvailableNodes` 就停止嘗試其他舊 Leaf。各 Leaf 成功、失敗或逾時的結果分別處理：成功確認的關係更新 realized topology，但只能從下一個安全選人邊界進入訓練；未確認者不加入。先完成的 Leaf 不必等待另一個 Leaf 逾時才可參與後續 round。E2a／E2b 不另設「至少接回一個 Leaf」的修復門檻，也不新增修復成功／失敗標記；接回幾個由實際訂閱、確認關係與拓樸紀錄事後判讀。

Root 能否繼續訓練仍依本層既有的 `minAvailableNodes`、`minTrainNodes` 與當輪完成條件判斷。即使 A1、A2 都未接回，只要 B／C 仍達 Root 門檻，就可繼續降級訓練，但不得把未形成的 Root→Leaf 關係當成已接回。現行 A→A* 流程在替代候選用盡時也是依剩餘 active Branch groups 的 readiness 決定能否繼續；Slice 3 須將 Root 的計數與 round 執行調整為能正確處理直接 Leaf 與 Branch 並存，而非增加獨立的 Area A 修復下限。

新確認的 Root→Leaf 關係沿用現行 A* 接回的參與時點：Root 在每次 round 開始時選擇當時可用的直接子節點。若 Leaf 在某輪已發起後才完成重掛，它只能從下一次選人時成為候選，不插入進行中的 round；是否實際選入仍依 Root 的既有 round policy。

在本實驗的 A 無回報情況下，Root 於當輪等待回報期限到期、判定 A 未能貢獻後才啟動修復。E2b 的 A2 不必與 A 精確同時停止；只要在 Root 嘗試對 A2 建立新訂閱前不可用即可。此時 A2 的失敗會在建立訂閱／preparation 階段被觀察到，應依通訊失敗或逾時的處理原則標記為不可用，不把未確認的 Root→A2 算作已形成關係。這與 A 在既有訂閱的訓練回報階段失聯，發生於不同時點；若 A2 已先完成 Root 的新訂閱才停止，則屬於重掛後再次失效，不是此處的 E2b 情境。

修復只針對 Area A：同一次訓練的 `mlCorreId` 應延續；Root→B／C 及 B／C→Leaves 的既有關係不因 A 失效而重建。新建的 Root→Leaf 關係使用各自的新訂閱資源。此情境不要求取得 A 故障前尚未送達的 Leaf 計算結果，也不以成功清理 A 的舊訂閱作為建立新關係的前提。

### Root 本地設定決策

`on_branch_failure` 與拓樸 YAML 的 `policy`、`strategy`、`branch_groups` 同層，不放進任何一個 Branch group，也不下發到 `flTopology`。此設定在同一次實驗中適用於 Root 管理的所有 Branch groups，不另設逐組覆寫。每次實驗的 Root 設定明確選擇：E1 使用 `replace_branch`，沿用同組替代 Branch 候選；E2a／E2b 使用 `reparent_leaves_to_root`，在 A 失效後由 Root 嘗試接回 A 失效前已確認的直接下層 Leaves。兩種值控制修復路徑，不改變既有 `policy` 的參與及訓練門檻。

現有拓樸 YAML 的 `admission.mode: complete_required` 僅被設定解析器固定要求，沒有被 Root 用來選擇接受行為。本 Slice 實作時一併移除該本地欄位、對應的 `StaticTopologyFile`／`TopologyAssignment` 欄位與相關設定測試／範例；既有初始拓樸建立條件維持不變，不藉此加入 partial admission 模式。記錄實際已確認關係的 `RootAdmissionSnapshot` 是另一項執行狀態，應保留。

### 混合深度訓練與聚合決策

Root 的 `minAvailableNodes`、`minTrainNodes`、`fractionTrain` 與當輪完成條件均以當時已確認的**直接子節點**為單位，不以 Branch group 或所有後代 Leaves 計數。A 失效且尚未接回 Leaves 時，Root 可用的直接子節點為 B／C；E2a 接回後為 A1／A2／B／C；E2b 接回後為 A1／B／C。每輪仍由 Root 依本層 policy 從當時可用的直接子節點選人，僅以該輪實際成功貢獻者聚合。

Root 對改掛 Leaf 建立的是新訂閱；該 Leaf 改掛後的訓練內容以 **Root 在新訂閱下發的指令**為準，不自動繼承舊 Branch 訂閱的 strategy 或 `reportAfter`。實驗可刻意下發相同參數以控制變因，但相同參數不是修復流程的固定語意。

同一 Root round 可以同時收到直接 Leaf 的 `TRAINING` 結果及 Branch 的 `HIERARCHY_AGGREGATE` 結果。Root 應按每個直接參與者的實際角色驗證結果型別與模型／round／procedure identity；Branch 結果另核對其已確認的下層參與者，直接 Leaf 不套用 Branch 的下層名單驗證。通過驗證後，沿用 `sampleWeighted`，以每份結果代表的訓練樣本數加權：直接 Leaf 用自己的樣本數，Branch 用其聚合結果記載的下層樣本總數；不再把失效的 A 另算一次。若六個 Leaves 各有 8,000 筆且都成功貢獻，E2a 的 Root 權重分別為 A1 8,000、A2 8,000、B 16,000、C 16,000；E2b 缺少 A2 時，總樣本數為 40,000。這些數字只是該實驗配置的例子，程式仍以實際結果的樣本數為準。

## 2. E2a：A1／A2 都直接改掛 Root

A1、A2 仍可用。Root 分別與兩者建立直接關係，修復後希望形成的拓樸是：

```text
Root
├── A1
├── A2
├── B ── B1, B2
└── C ── C1, C2
```

應能分別看見 Root 的修復指示、A1／A2 實際確認的新關係、逐級回報形成的 realized topology，以及 Root 是否接受該拓樸。接受後，直接 Leaf 的訓練結果與 B／C 的區域聚合結果應能在後續 Root 訓練中共同參與。六個 Leaves 的資料仍在，但階層深度和聚合路徑已改變；不預設模型曲線與無故障對照完全相同。

## 3. E2b：只有 A1 直接改掛 Root

A1 仍可用，A2 不可用。Root 仍嘗試接回 A1／A2，但只有實際確認的 Root→A1 會出現在 realized topology；不得將 A2 算作已形成的關係。Root 依本層既有訓練門檻判斷能否繼續，後續可用的直接參與者為 A1、B、C。「部分接回」是從實際形成的關係判讀的實驗結果，不是另設一套要求至少接回一個 Leaf 的線上接受條件。

此情境與 E2a 不只差在拓樸：A2 的訓練資料也永久缺席。因此後續模型表現的差異，必須與訂閱／拓樸修復是否成功分開判讀。

## 4. 既有流程與本 Slice 的處置

實作前，Root 的 `FLRootCoordinator` 以 `branch_groups` 建立 Root→Branch 訂閱，`FLServerEngine` 處理 preparation／round，失效時 `_retire_failed_branch` 移除 A 並以 `_replace_branch_group` 準備 A*。當時 Root round 的選人、ready 門檻、模型取用名單與結果型別都只按 active Branch 計算；`_realized_topology` 也只呈現 active Branch reports。本 Slice 延伸這條既有主流程，沒有建立另一條獨立的 FL loop。

| 既有階段 | 處置與本 Slice 的必要差異 |
| --- | --- |
| 觸發與初始設定 | 沿用手動或既有 Root trigger、相同的 `mlCorreId`；靜態拓樸解析新增 Root 本地 `on_branch_failure`，並刪除無效的 `admission.mode`。初始 Root→Branch→Leaf formation 語意不變。 |
| Preparation 與關係確認 | 以現有 Root→A* 的 preparation 契約為基礎，但 A1／A2 的 Create 與等待結果須並行，不能直接用單一修復 worker 串行呼叫阻塞式 `prepare_protocol_replacement_target`。送出各自 Leaf 形態的 `flTopology`；僅有成功回報並確認的 Root→Leaf 關係進入 realized topology。 |
| 訓練執行 | 原有同一個 Root round loop 改用已確認的直接子節點集合；下一輪才選入新 Leaf。模型仍在 round PATCH 下發，不把 preparation 改為模型驗證。 |
| 結果驗證與聚合 | 原本一輪只有一個 `expected_result_type`；改為按直接子節點核對 `TRAINING` 或 `HIERARCHY_AGGREGATE`，保留共通 identity／model 驗證，再按各結果的樣本數聚合。 |
| 模型發布與回報 | Root 的 global round model 仍先存 ADRF，再讓選入的直接子節點及必要的下層從該參照取用；Branch 的區域模型與各節點向上回報不改為 ADRF。沿用既有 Root validation、最終模型保存與 Slice 2 事件紀錄。 |
| 故障、逾時與清理 | 沿用 A 回報逾時後退役的時機、遠端刪除失敗不阻擋新關係、Root 達門檻可降級前進的規則。直掛 Leaf 的個別訂閱失敗／逾時不算已確認；正常完成或 Root 失敗時仍由既有 FL Server process 清理其所管理的訂閱。 |
| 重新啟動 | 現有 Root 訓練與修復狀態是程序內記憶體狀態，本 Slice 不宣稱 Root 程序重啟後恢復同一次訓練；故障注入只針對 A／A2。關閉或 generation 變更時須阻止在途重掛結果重新寫入舊 run，並沿用既有清理路徑。 |

## 5. 修改位置、狀態與資料流

### 5.1 本地設定與 Root 狀態

- 在 `PyMTLF/src/py_mtlf/core/fl_topology.py` 的 `StaticTopologyFile`、`TopologyAssignment` 與 `StaticTopologyPlanner` 加入必填的 `on_branch_failure`，只接受 `replace_branch`／`reparent_leaves_to_root`。同步修改拓樸 YAML、相關 fixtures／tests，移除 `CompleteRequiredTopologyAdmission`、`admission_mode` 和無效的 `admission.mode` 輸入；不保留舊版相容讀取。`RootAdmissionSnapshot` 繼續記錄現有已確認 Branch 關係，不因同名欄位被移除。
- 在 Root request 的記憶體狀態中，另保留**已確認的直連 Leaf 關係**：至少包含 Leaf `nfInstanceId`、所屬原 Branch group、準備回報與正式訂閱位置。這筆 Root 本地狀態不取代接收端產生的正式訂閱 ID，也不以 Branch group 的 active 狀態代替。Root 的直接子節點快照由 active Branch 與 confirmed direct Leaf 合併產生，供 readiness、選人、拓樸呈現與模型授權共用；原 Branch group 退役後不得以其舊 report 再計入 active 關係。
- Root 應在清除 A 的 `active_report` **之前**取出 A 失效前已確認的直接子 Leaf ID，與該 group 的原始 Leaf 設定核對；不要把未確認候選或 B／C 的後代誤列為要重掛的對象。現有單一修復 worker 可負責協調本次修復，但須讓 A1／A2 的訂閱與 preparation 各自作為並行的等待工作執行，不讓其中一個目標的網路等待或逾時阻塞另一個。每個結果獨立回填；即使 Root 已達 B／C 的訓練門檻，也要發起對兩者的嘗試。每次候選與訂閱結果保留在 Slice 2 的現有紀錄，不新增修復專用旗標。

### 5.2 Root→Leaf 訂閱的端到端路徑

1. Root PyMTLF 以已保存的 A1／A2 身分及本地訓練設定建立各自的 Leaf `FlTopologyNode`：沒有 `children` 或下層選擇授權，明確帶上此次要給 Leaf 的 `strategy` 與 epoch 單位 `reportAfter`。設定來源是 Root 在本次新訂閱採用的拓樸配置，**不是**複製舊 A→Leaf 訂閱的執行狀態。Root 以 `HierarchyNodeRole.LEAF` 解析兩個目標，在同一 `FLServerEngine` process 上並行發起各自的 Create／preparation；`mlCorreId` 保持不變，`notifCorreId` 與訂閱資源各自新建。現有 `prepare_protocol_replacement_target` 會等待單一目標完成，實作須調整呼叫安排或拆開發起與等待，並確認共用 process 的 participant／callback 狀態在並行時安全，不在等待網路回覆期間持有阻塞另一目標的鎖。
2. Root PyMTLF 透過現有私有 Model Training 發起路徑交給 Root Go NWDAF；Root Go 將 Create 送往目標 Leaf Go NWDAF 的現有 `Nnwdaf_MLModelTraining` SBI，Leaf Go 再交給 Leaf PyMTLF。Root Go 回傳的正式訂閱位置由 Root PyMTLF 保存。`mLPreFlag` 為 true，preparation 不帶模型；沿用既有任務欄位、`suppFeats` 與 `flTopology` 的驗證／協商，只同步改名既有候選欄位，不新增 Go route、其他 SBI 欄位或新的 subscription ID 類型。
3. Leaf PyMTLF 依新訂閱的 `flTopology` 判定自己為 Leaf，確認本地資料集、strategy、epoch 與 capability，透過原 notification 路徑回傳本 Leaf 的 `flTopologyReport`。Root PyMTLF 只在收到符合新訂閱身分且確認可參與的回報後，把該 Root→Leaf 邊納入 realized topology；僅 Create 成功或目標在 NRF 可發現都不足以算 confirmed。Leaf 現有同一 `mlCorreId` 的新 preparation 回報獲得確認後，已有停用舊父節點訂閱的流程；此處須以跨程序測試核對 A 失聯時該流程不阻擋新關係。因新舊兩份訂閱在交接期間短暫並存，部署的 Leaf `max_concurrent_jobs` 至少須容納兩份；現有常用 Leaf 設定值為 2，不新增為此情境特製的容量繞路。
4. A1／A2 各自完成、失敗、被拒絕或 preparation 逾時時，分別記錄該嘗試的實際結果；單一目標的逾時從自身操作計算，不延後另一目標的發起或已取得的成功結果。確認成功的 Leaf 由 Root round loop 在下一個安全選人邊界納入，無須等待全部嘗試結束；若已建立但未確認，安排清理該次 Root-side 目標。`remove_protocol_participant` 只容許準備評估或 ready 狀態；並行工作把須移除的目標交給 Root round loop，由後者在 round 結束、下一輪選人前的 ready 邊界清理，不能在 active round 中修改 participant 集合。若 run 先結束，交由整個 process 的既有關閉路徑清理。預期內的 A2 失敗只更新該次嘗試結果，不寫成讓 Root `_wait_for_root_readiness` 拋錯的 `replacement_error`；Root 持續按當時已確認的直接子節點數量評估，不因 A2 的等待或失敗將 B／C 或已接回的 A1 一起退役。若 Root generation 已失效，各個在途回覆均不得重新啟用舊 run；已取得的新資源走既有清理路徑。

### 5.3 下一輪訓練、模型與聚合

- `FLRootCoordinator` 的 readiness、priority／fraction 選人、當輪 `selected`／`successful`／`failed` 名單及拓樸接受判定，統一讀取**目前已確認的 Root 直接子節點**。Branch 仍以原 group 的候選優先級排序；直連 Leaf 使用其原本配置的 Leaf 優先級。計數規則不展開 B／C 的子樹；Root 的 `minimum_completion_rate` 仍以當輪選入的直接子節點為分母。每輪開始時固定本輪 selected IDs 與各自角色；若 B／C 已達門檻，A1／A2 準備中的結果不插入目前執行中的選人集合。若 A1 已確認而 A2 仍在等待，A1 可在下一次選人時納入，無須等待 A2 的最終結果。Root 選人、可用數計算及失敗名單處理都必須共用此直接子節點定義，不能只改聚合器。
- Root 發布每輪 `ROUND_INPUT`／ADRF global model 時，授權集合包含該輪選入的 direct Leaf、本輪選入的 Branch，以及這些 Branch 已確認且需要取用 global model 的下層；不能因 A 的舊 report 把已失效的 A 或尚未確認的 A2 算入。Root 對 A1／A2 使用與其他直接子節點相同的 round PATCH；Leaf 依**新訂閱**收到的指令與模型產出 `TRAINING` 結果。Branch 繼續處理自己的 Leaves 並回報 `HIERARCHY_AGGREGATE`。
- `FLServerEngine.execute_hierarchy_round` 與 `_aggregate_round` 改由 Root 提供每個**已選入直接子節點**的預期結果型別，而不是為整輪固定一種型別。聚合前核對所選者有且只有其角色允許的 artifact；所有結果仍須符合 `mlCorreId`、`roundInd`、participant ID、training scope 與模型相容性。只對 Branch 的 `HIERARCHY_AGGREGATE` 核對其已確認的 subordinate IDs；對 direct Leaf 的 `TRAINING` 不要求 subordinate 清單。Branch 內部呼叫這個共用引擎時仍全部預期 `TRAINING`，避免破壞現有下層聚合語意。
- 對該輪實際成功且通過驗證的 direct results，沿用 `FederatedTrainer.aggregate` 的 `training_sample_count` 加權。Branch 的 count 已代表其下層貢獻總量，direct Leaf 的 count 代表自身資料量；Root 不再額外展開 Branch 子樹重複聚合。當輪未達接受條件時維持現有不更新 global model 的行為；達標才記錄 accepted round、Root validation 和後續 final model。
- 當輪失敗名單按**直接子節點角色**分流：失敗 Branch 走既有退役及 `on_branch_failure` 修復選項；已直掛 Root 的 Leaf 若在後續 round 失效，就移除該直連關係並依剩餘直接子節點及既有 policy 判定能否前進，不把它當成 Branch group 或重新觸發 A→A*。本 Slice 的 E2b A2 是重掛 preparation 未確認，並非已直掛 Leaf 在 round 中失效。

### 5.4 關係呈現、紀錄與清理

- `_realized_topology` 由 active Branch reports 與已確認 Root→Leaf 邊組成；A 未修復前只呈現 B／C，E2a 呈現 A1／A2／B／C，E2b 呈現 A1／B／C。候選、Create 成功但未完成 preparation 的 A2 不得混入已形成拓樸。Root 每次形成新關係或失去關係後，按當時 Root policy 對**當時 realized topology**記錄接受結果；不能只因某 Leaf preparation 成功就固定寫 `accepted=true`。沿用 Slice 2 的直接關係確認、失效、修復選擇、拓樸接受、訂閱與 round 原始紀錄；不新增 wire `topologyVersion` 或可由現有紀錄事後推導的 `recovered` 欄位。
- `RootAdmissionSnapshot` 保留既有 Branch 清單的語意；修復後完整 Root 直接子節點由 realized topology 與直接關係紀錄呈現，不把 Leaf 偽裝成 admitted Branch。B／C 及其下層既有訂閱 ID 必須保留；舊 A 的刪除失敗只留下清理結果，不阻斷新 Leaf 訂閱。
- 正常完成時，`close_hierarchy_training` 清理 Root 管理的現存直接訂閱並保存 final model；失敗／關閉時，沿用既有取消、workspace 與 reservation 清理。重掛中的 future 須與 Root generation 綁定，避免 run 結束後晚到的 preparation 結果改寫狀態。跨節點正式訂閱 ID 仍以接收端產生並回傳的值記錄，不用 callback ID 代替。

## 6. 修改範圍與驗證

**修改範圍**：`NWDAF` 與 `PyMTLF` 同步上述四個候選 payload 欄位名稱、解析／驗證與相關測試；`PyMTLF` 另調整靜態拓樸模型與 YAML、Root coordinator／Root 本地狀態、A1／A2 的並行 preparation 協調、共用 FL Server round 聚合及相關單元／流程測試。並行範圍限本次需重掛的 direct Leaves；E1 同一 Branch group 內依 priority 依序嘗試替代 Branch 候選的語意不變。為執行必要的本地真實程序驗證，`nwdaf-resources` 的 runner 也已更新：從目前的 NWDAF factory 欄位產生節點設定，改用現行 topology／紀錄欄位，並加入 E0、E2a、E2b 的短程 profile；不修改實驗室 testbed。Leaf PyMTLF 的新訂閱與舊關係停用路徑、Go NWDAF 的 private gateway／SBI Create／PATCH／notification 流程，以及 ADRF model distribution 仍使用既有路徑。Go 邊界的依據是 `NWDAF/internal/sbi/processor/ml_model_training.go` 現有遠端 Create 路由與 PyMTLF `FLServerEngine` 的 private training 操作；本 Slice 不主張新標準欄位或 free5GC 原生支援混合深度 FL。

| 驗證層級 | 要直接證明的結果 |
| --- | --- |
| 設定與拓樸單元測試 | 缺少或填錯 `on_branch_failure` 被拒；E1/E2 模式各自選對路徑；舊 `admission` 不再作為合法本地欄位；初始 policy 門檻與 branch group 映射不變。 |
| 候選 schema 合約測試 | 沿用現有合約測試確認 Go、PyMTLF 的 Create／PUT／PATCH／Notify／`immReport` 使用無前綴欄位，並以程式碼檢查確認沒有舊欄位相容讀取；不為每個過時欄位另設拒絕測試。E0／E1 跨程序回歸仍須證明新格式可交換。 |
| Root coordinator 決定性測試 | 確認 A 的原 confirmed Leaf 名單在 report 清除前保存；A1／A2 的 preparation 可並行，A1 成功時不受 A2 等待或失敗阻擋，且從下一輪才可被選入；B／C 繼續貢獻。使用可控制的回覆／同步點證明主要流程，不以每個狀態排列組合各新增測試。失效 generation 與在途資源清理沿用既有機制並在審查時核對。 |
| FL Server／artifact 測試 | 同一 Root round 同時接受 Leaf `TRAINING` 與 Branch `HIERARCHY_AGGREGATE`；錯型別、錯 `mlCorreId`／round／participant、錯 Branch subordinate set 會被拒；Leaf 不被要求 Branch subordinate set；按實際樣本數加權且不雙重計數；原 Branch→Leaf 純 `TRAINING` 路徑不退步。 |
| 訂閱、模型與紀錄流程測試 | Root→A1／A2 的 Create／preparation 可重疊執行，且各自的 notify／round PATCH 經現有 Go-facing 路徑到達正確 Leaf；Root 的 ADRF allowlist 包含已選入的直接 Leaf／必要後代，未確認 A2 不列入；同一 `mlCorreId`、各自的新 Root→Leaf 訂閱 ID、B／C 未重建；realized／accepted topology 與事件和實際形成關係一致。需在實際 Leaf 容量設定下覆蓋原 A 訂閱尚殘留時的新訂閱、preparation 回報與舊關係停用結果。 |
| 本地 real-process 與 testbed | 先跑無故障 E0、既有 A→A* E1 回歸，再跑 E2a、A2 於重掛前停止的 E2b；檢查 accepted Root rounds 的直接參與者、模型保存及逐節點紀錄。實驗端的五個 seeds、故障注入時點與論文統計屬後續 testbed 驗收，未跑前不可宣稱 E2a／E2b 已在 testbed 完成。 |

### 本地短程驗證設定

以下是 `nwdaf-resources/deployments/hierarchical_fl` runner 的實際設定，**不是**正式論文實驗的資料量、故障輪次或訓練時長。

| 項目 | 本地實際值 |
| --- | --- |
| 節點與拓樸 | 1 Root、3 個現役 Branch、每個 Branch 2 個 Leaves；E1 另預先部署 A*，E0／E2a／E2b 不部署 A*。 |
| 故障處理 | E1 使用 Root 本地 `on_branch_failure: replace_branch`，A／A* 優先級分別為 100／50；E2a／E2b 使用 `reparent_leaves_to_root`。 |
| Root policy | `minAvailableNodes=2`、`minTrainNodes=2`、`fractionTrain=1.0`、`acceptFailures=true`、`minCompletionRate=0.66`；因此 A 失效後，B／C 的兩份成功結果仍可使該輪被接受。 |
| Branch policy | 每個 Branch 要求兩個直接 Leaves 均可用且參與：`minAvailableNodes=2`、`minTrainNodes=2`、`fractionTrain=1.0`、`acceptFailures=false`、`minCompletionRate=1.0`。 |
| 訓練與模型 | Root 執行 4 個 rounds；`fedProx` 的 `proximalMu=0.01`，以 `sampleWeighted` 聚合。Branch 的 `reportAfter` 為 1 round，Leaf 為 2 epochs；Leaf 的 2 epochs 由新訂閱的 `reportAfter` 決定，不採用 round bundle 中預設的 `client_training.epochs=1`。Leaf 使用 batch size 16、learning rate 0.001，在 CPU 執行。 |
| 資料與量測 | MNIST；seed 42 從官方訓練集隨機分配每個 Leaf 64 筆本地 shard。Root validation 128 筆、獨立 held-out 128 筆，兩者均從官方測試集抽取；不是正式論文實驗的資料分割。 |
| 逾時與故障注入 | Preparation 與 round timeout 各為 60 秒。第 1 個 Root round 被接受、下一輪進入等待回報後，runner 停止 A 的 Go NWDAF 與 PyMTLF 程序；E2b 同時停止 A2 的兩個程序。 |

本地真實程序結果：smoke 通過單組 Branch／Leaf 協定流程；E0 四輪均由 A／B／C 貢獻；E1 在 A 失效後由 A* 建立新關係並重新貢獻；E2a 由 Root 直接接回 A1／A2；E2b 同時嘗試接回 A1／A2，但 A2 停止服務，只有 A1 成功確認並參與後續 round。E1／E2a／E2b 均有 B／C 的降級 round 與最終模型保存證據。這些各為單次、四輪的本地流程測試；正式 testbed 的多 seed 結果仍待驗收。

四次本地 run 的最終 held-out accuracy 均為 11／128（約 8.59%）。此資料量與輪數只用於驗證訂閱、故障處理、混合深度聚合及模型保存能否執行，不能用來主張學習品質或故障後 accuracy recovery。

執行階段依 `PyMTLF` 既有 lint 與 pytest 工作流，先跑上述 focused tests，再跑 full suite；Go 欄位改名須跑對應的合約／SBI 測試與建置。此處的本地實作及驗證結果已供使用者確認並完成提交，**不代表**正式 testbed 驗收完成。

## 7. 範圍外

五個配對 seeds 的排程、故障注入、跨節點資料收集、95% CI／AUC／恢復判定與離線模型分析由實驗執行端處理，不納入本 Slice 的 PyMTLF 功能。亦不做 retained-result 復用、Root 重啟後接續訓練、新的 topology version wire 欄位、跨多個同時失效 Branch 的一般化修復、或把 E2b 的資料缺席解讀成協定失敗。Slice 2 的逐節點紀錄提供原始證據；本 Slice 只補足 E2a／E2b 的實際執行能力。
