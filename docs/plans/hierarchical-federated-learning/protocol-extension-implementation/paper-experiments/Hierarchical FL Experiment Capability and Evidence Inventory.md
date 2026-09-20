# Hierarchical FL 後續實驗能力與證據盤點

日期：2026-09-20

狀態：盤點內容已確認；尚未開始本批功能實作或 E0–E3 五 seed 實驗。

本文承接 [E0–E3 實驗情境與 Testbed 對照](./Hierarchical%20FL%20E0-E3%20Experiments%20and%20Testbed%20Context.md)，盤點現有實作能否承載各情境，以及缺少哪些功能與原始證據。此處不重新定義實驗的統計判準，也不直接修改 testbed。既有 MNIST／CIFAR-10 單次配對是歷史觀測，不是接下來五 seed 實驗的完成證據。

## 1. 盤點基準與識別語意

在目前 testbed 設定下，Root PyMTLF 已記錄初始與逐個 accepted round 的模型驗證、Root round outcome、Branch 故障偵測與替代就緒，以及 final model。紀錄位於每個 `mlCorreId` 的 `observations.jsonl`。Branch／Leaf 雖可設定 node-local validation，但現有結構化紀錄沒有涵蓋其訂閱建立、變更、失效與拓樸決定；若未啟用本地驗證，也不能由現有紀錄推論它們已有逐節點事件。現行 testbed 另外從 PyMTLF 日誌擷取 `resourceLocation`，但此值是發起端 Go 的私有代理位置，不是接收端產生的 SBI 訂閱資源 ID。故障情境的收集器目前只挑 Root→A／A* 及 A／A*→A1／A2 的資源紀錄，不完整收集 Root→B／C 等未受影響邊。

本次現況判斷對照 `NWDAF/internal/sbi/processor/ml_model_training.go`、`NWDAF/internal/context/ml_model_training_routes.go`、`PyMTLF/src/py_mtlf/core/fl_server.py`、`fl_root.py`、`fl_branch.py`、`fl_topology.py`、`experiment_recording.py`，以及唯讀 testbed 參考中的 `fl_experiment.py`、`fl-experiment-run.py`；後者不是本次修改目標。

本文件用以下名稱區分識別碼，避免將它們都稱為 subscription ID：

| 名稱 | 產生與用途 | 是否可作為論文中的訂閱資源 ID |
| --- | --- | --- |
| `L` | 發起端 Go 在送出 Create 前建立的本地 callback 路由 token；目前也被用作發起端 route 主鍵及回給 PyMTLF 的私有資源路徑 | 否 |
| `R` | 接收端 Go 建立 SBI 訂閱後，在 `Location` 回傳的對外訂閱資源 ID | 是；須連同接收端身分或資源位置辨識所屬訂閱 |
| `mlCorreId` | 同一次 hierarchical FL procedure 的關聯識別碼 | 否；它可跨多條訂閱關係共用 |
| `notifCorreId` | Notification 關聯識別碼 | 否；不取代訂閱資源 ID |

現有資料流為：父節點 PyMTLF 請自己的 Go 建立訂閱；父節點 Go 先以 `L` 預留路由及 callback URI，向子節點 Go 建立訂閱；子節點 Go 以 `Location` 回傳 `R`；父節點 Go 將 `R` 存於 `L` 主鍵的路由中，卻回給父節點 PyMTLF 以 `L` 為路徑的私有 `Location`。因此 PyMTLF 日誌與 testbed 擷取到的是 `L`，無法直接支持「舊、新 SBI 訂閱 ID」的論文主張。

已確認的修改方向是：建立期間只讓 `L` 擔任 callback token／暫存關聯；確認收到 `R` 後，正式訂閱管理改以 `R` 為資源識別，並清掉不再需要的建立中暫存狀態。父節點 PyMTLF 收到的是**自己 Go 的私有 URL，但其資源路徑使用 `R`**；後續 PUT／PATCH／DELETE 仍交由該 Go 轉送，不能直接呼叫子節點的對外 URL。Go 保留必要的 `L → R` callback 查找，讓既有 notification URI 在訂閱存續期間可用。這是目標行為，**不是現行實作**。詳細計畫仍須處理 Create 尚未完成就收到 callback、Create 失敗或 peer 已建立但回覆／清理失敗，以及不同接收端可能有相同資源 ID 時的路由命名空間；不得只改 PyMTLF 日誌欄位就宣稱完成。

## 2. 共通必備能力與紀錄

| 項目 | 現況及缺口 | 接下來需要的結果與主要擁有者 |
| --- | --- | --- |
| 真實訂閱識別與生命週期 | Go 知道 peer `Location`，PyMTLF／testbed 只取得以 `L` 為主鍵的私有位置；文字日誌只涵蓋成功建立的部分資源 | **NWDAF Go 與 PyMTLF** 完成上述 `R` 資源管理及雙向契約調整；對每條父子關係可重建 Create 嘗試、成功／失敗、更新、終止與新舊 `R`，保留 peer 身分及 `mlCorreId`。接收端仍需保留 Go 對其 backend-local 資源的映射；不把後者誤當 SBI ID。 |
| 逐節點訂閱與拓樸事件 | 現有 `observations.jsonl` 缺少 Branch／Leaf 的訂閱事件；Root round 與 replacement 事件不足以證明各 edge 何時建立、保留或消失 | **各節點 PyMTLF** 擴充 node-local `observations.jsonl`，記錄自己的訂閱決策與結果、父子 NF、方向、`R`、關聯 ID、時間及重要 instruction／report 摘要。Go 應透過自己的私有回應交付從 peer `Location` 得到的 `R` 與操作結果，不讓 PyMTLF 直接使用 peer 對外 URL；不預設 Go 另外寫實驗 JSONL。PyMTLF 看不到的內部重試或 wire 結果若也是證據需求，才另定最小 Go telemetry 與收集方式。 |
| 三種拓樸狀態 | 協定可傳 `x-flTopology`／`x-flTopologyReport`，但現有實驗紀錄未留下完整的 Root 建立／修復意圖、已確認邊、Root 接受或拒絕的前後快照 | **Root／各直接父節點 PyMTLF** 紀錄 initial／revised topology instruction、realized topology 的 confirmed edges 與未成功候選者、Root acceptance decision 和 accepted realized topology。每次修復至少有可排序的變更標記或本地版本；不先假定需增加標準或 extension wire `topologyVersion` 欄位。逐輪成功貢獻者另記，不等同拓樸成員。 |
| 訂閱重要內容 | 現行資源日誌只列 `process_id`、對端 NF、`notifCorreId` 與代理 `Location` | **各節點 PyMTLF** 記錄足以解釋決策的 request／accepted representation 摘要，例如 `mLEventSubscs`／`mLModelTrainInfos` 的訓練要求、`x-flTopology` 的接收節點與候選 children／priority、各節點 policy／strategy／report-after、`x-flTopologyReport` 的回報狀態，以及更新前後的差異；連同操作結果和原因。Go 仍須正確傳回操作回應及 `R`，但不需為同一內容再寫一份實驗檔。避免把完整 model artifact、權杖或無關 payload 重複寫入紀錄。候選、成功訂閱及 confirmed edge 必須分開。 |
| 逐輪與模型證據 | Root 已記錄 selected／successful／failed participant IDs、accepted 與 validation；Branch／Leaf 只在啟用本地驗證時記模型評估 | **PyMTLF** 保留 Root 每次 round attempt／accepted outcome 與初始、逐 accepted round 的 validation；視情境補上 Branch 下層 selected／successful／failed set、Leaf 本地訓練或回報完成及失敗原因。Root 模型結果須能對回相同 `mlCorreId`、`roundInd`；圖表用 one-based accepted round、初始評估為 round 0。各節點本地 validation 可選，不應被誤認為所有情境的必要前置。 |
| 時間線及控制面次數 | Controller 有故障注入時間；Root 有故障偵測與 replacement ready；尚缺明確的 reparent instruction、新 edge 分別完成的時間；文字日誌不足以可靠計數 API 呼叫 | **Controller** 提供故障注入時間；**PyMTLF** 記錄 detection、instruction、各新 edge ready、Root 接受修復及首次 accepted contribution。精確的 SBI API-call 數、Go 內部重試與未傳回 PyMTLF 的失敗，須由 **Go SBI 邊界**提供可收集的最小 telemetry，或先證明既有紀錄足夠；不預設 Go 寫 `observations.jsonl`。重試、通知與雙端觀測的計數口徑需固定，跨程序時間須可對齊或明示誤差。 |
| 五 seed 配對與失敗保留 | 既有論文配對每條件只有一次；testbed run metadata 可保存 scenario，但尚未證明五 seed E0–E3 全套可追蹤 | **Testbed 執行／收集端**保存 workload、scenario、seed、資料分割、初始模型、訓練設定、觸發 round、程式版本與 run ID；所有失敗或未恢復 run 仍保留原始事件與模型曲線。PyMTLF 只需產生可靠原始紀錄，不實作 CI、AUC 或 recovery 判定。 |

逐節點實驗檔以 PyMTLF 為主要寫入者；Go 是真實 SBI 資源 ID 與部分 wire 事實的權威來源，**不是預設的第二套實驗檔寫入者**。控制面精確計數或 Go 內部失敗若無法經 PyMTLF 現有回應觀察，才需要 Go 提供額外最小 telemetry。Testbed 收集與離線關聯另屬其擁有者；不把收集腳本的修改算作 PyMTLF 已完成的能力。

## 3. 各實驗額外需要的能力

| 情境 | 目前可沿用 | 必須補足或確認 |
| --- | --- | --- |
| E0：無故障 baseline（必要） | 固定三個 Branch group 的訓練、Root 每輪 outcome／validation、final model | 五 seed 配對紀錄；初始 instruction、realized／accepted topology 與每條真實 `R` 的建立證據；確認未出現非預期重建。 |
| E1：A 故障、A* 接手（必要） | 已有單一 Branch replacement、degraded round、A* 建新下層關係以及 Root failure／ready 事件；不取回舊 Leaf 計算結果 | 以真實 `R` 證明 A→A1/A2 轉成 A*→A1/A2 且 Root→B/C 與 B/C 子樹不變；記錄 reparent instruction、各新 edge 完成、首次 accepted A* contribution、修復前後拓樸與五 seed 成敗。不能把「ready」當作首次貢獻。 |
| E2a：A 消失，A1／A2 直掛 Root（核心、強烈建議） | 候選協定的 recursive tree 可表達深度不一致的拓樸 | **現行 PyMTLF Root 尚非此情境的可執行實作**：static topology config 以 `branch_groups` 組織；Root 直接子節點選擇與結果驗證按 Branch／`HIERARCHY_AGGREGATE` 處理。需補 Root 對直接 Leaf 的 preparation、`TRAINING` 結果及 Branch aggregate 混合接納，驗證混合貢獻的樣本數權重、ADRF 模型取用名單、partial subtree repair 與接受紀錄；同時確認 Go／wire 既有契約足以承載。 |
| E2b：只有 A1 直掛 Root（建議消融實驗） | 可沿用 E2a 的 mixed-depth 路徑 | 再補 partial repair 與 policy 決定：instruction 可含 A1／A2，但 realized 只含 A1，Root 必須依明示的 direct-child 條件接受或拒絕；不能沿用目前 `complete_required` 及三 Branch cohort 的門檻卻宣稱自然接受。記錄 A2 不可用和資料覆蓋變動。 |
| E3：Leaf A1 消失、A 留下（可選） | Branch candidate pool 已有 direct-child policy、狀態及 topology report 表示法 | 既有 testbed Branch policy 要求兩個 Leaves；在此設定下，A 只收到 A2 不足以接受該次下層 round。需先設定允許僅 A2 的 formation／round completion 條件，並確認與實作 A1 edge 失效、A 的 realized subtree 更新及向 Root 逐級通知／接受，而非只記錄 A2 成功。Root→A/B/C 應保持。這是 protocol boundary test，不另擴張為收斂研究。 |

E2a／E2b 的 Root direct cohort 與 E3 的 Branch 存活者數量改變，會使 `minAvailableNodes`、`minTrainNodes`、`minCompletionRate` 等設定的分母和對象不同於 E0／E1。實驗前要逐情境固定設定並保留配置快照；不能把修改 policy 後的結果描述成僅改故障類型。E2／E3 的故障觸發輪次與選用 workload 也待實驗規格確定。上述「需補」是能力盤點，不是已經通過 testbed 的宣告。

## 4. 原始紀錄與離線分析邊界

逐節點事件至少要能以 `recordedAt`、`mlCorreId`、本節點及對端 `nfInstanceId`、事件／操作類型、方向、結果及原因串起同一條父子關係；正式訂閱事件使用接收端的 `R` 與其 owner，`L` 如需保留僅供 callback 排錯，名稱不得叫 `subscriptionId`。訂閱建立嘗試、收到 Create 成功回覆、完成 preparation／形成 confirmed edge、Root 接受拓樸是四種不同事件。更新與刪除亦保留嘗試及結果，不能把發出命令視為成功。

原始紀錄交給離線分析計算五 seed mean／95% CI、固定 post-failure window accuracy AUC、paired E0 endpoint 差值、rounds-to-recovery、各時間分段與每個情境的成功數。Recovery 採實驗文件所定義的「修復後 accuracy 進入 E0 對應 round 的 95% CI，且連續維持兩個 accepted rounds」；這不是 PyMTLF 的運行期判斷。未恢復的 run 明記 `not recovered`，不從分母或圖表中刪除。Final official test 與逐輪 Root validation 分開保存。

## 5. 進入詳細實作計畫前的確認

1. 以 Go Create／callback／PUT／PATCH／DELETE 的完整生命週期確定 `R` 主鍵、peer scope 與 `L` callback 索引的資料結構及失敗清理；對應更新 PyMTLF private API、測試與 testbed 收集器的識別碼解讀。
2. 確定 Go 的私有回應如何交付 `R`、操作結果與必要的訂閱資訊，讓各節點 PyMTLF 的事件紀錄足以支持資源與拓樸主張；只對 PyMTLF 看不到、但論文確實需要的 SBI 內部呼叫或失敗補最小 Go telemetry。明確定義事件名稱、必要欄位、產生點及重試去重方式，不靠解析人類可讀日誌當唯一證據。
3. 驗證 E2a／E2b mixed-depth 根據目前 Go／PyMTLF 契約所需的全部變更，特別是直接 Leaf 結果型別、aggregation 權重、ADRF 讀取權限與 Root policy；此項目前不能標為實作就緒。
4. 驗證 E3 下層 edge 故障的偵測、狀態轉換及向 Root 的 report／acceptance 路徑；先固定只剩 A2 時的 Branch policy，再決定最小實作範圍。
5. 確定 testbed 的五 seed 排程、故障事件與跨程序時鐘、API-call 計數及 run artifact 收集界線。參考 testbed 目前是唯讀來源；此文件不授權修改其部署或執行實驗。

本盤點的下一步是依上述確認拆出可獨立審查的功能與紀錄詳細計畫；不在此文件預設實作 slice 數量或宣告 E0–E3 已完成。
