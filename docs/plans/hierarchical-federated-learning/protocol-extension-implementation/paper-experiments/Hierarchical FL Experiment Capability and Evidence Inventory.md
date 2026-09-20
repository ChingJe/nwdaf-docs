# Hierarchical FL 後續實驗能力與證據盤點

日期：2026-09-20

狀態：Slice 1 訂閱識別的本地實作與審查已確認；第 2 項起沿用既有候選 schema 的盤點、第 2–4 項工作邊界及 E2a／E2b 的盡力重掛判斷已確認；五 seed 實驗尚未執行。

本文承接 [E0–E2b 實驗情境與 Testbed 對照](./Hierarchical%20FL%20E0-E2b%20Experiments%20and%20Testbed%20Context.md)，盤點現有實作能否承載各情境，以及缺少哪些功能與原始證據。後續沿用既有遞迴拓樸語意；候選 payload 欄位已改名為 `flTopology`／`flTopologyReport`，但 Go／PyMTLF 實作尚待同步。它們是專案 extension，不是已採納的 3GPP 欄位。此處不重新定義實驗的統計判準，也不直接修改 testbed。既有 MNIST／CIFAR-10 單次配對是歷史觀測，不是接下來五 seed 實驗的完成證據。

## 既有候選契約與論文附件的邊界

目前 NWDAF／PyMTLF 仍以 `x-flTopology`、`x-flTopologyReport` 和 recursive node/status 交換拓樸資訊；候選 schema 已將前兩者改名為 `flTopology`、`flTopologyReport`，程式同步屬後續實作前置工作。新版論文附錄 B 除同名欄位外，另提出 `topologyVersion`、`reparentInstruction` 與 direct-edge report 等不同設計；這些不是本批要遷移的 wire format，也不能把兩者視為只有命名差異。附件仍可供論文修訂時比較，但不能直接當作實作驗收條件。

既有 node 的 `policy`、`strategy`、`reportAfter` 與 participant／round 門檻繼續按原語意使用，不因附件 B 未列出就移除。後續若需要新增欄位，須先確認現有標準欄位、既有候選 schema 與本地配置能否承載；不能把附件 B 的 `minDirectChildren` 逕視為 `minAvailableNodes` 或 `minTrainNodes` 的改名。

## 1. 盤點基準與識別語意

在既有 testbed 執行設定下，Root PyMTLF 已記錄初始與逐個 accepted round 的模型驗證、Root round outcome、Branch 故障偵測與替代就緒，以及 final model。紀錄位於每個 `mlCorreId` 的 `observations.jsonl`。Branch／Leaf 雖可設定 node-local validation，但現有結構化紀錄沒有涵蓋其訂閱建立、變更、失效與拓樸決定；若未啟用本地驗證，也不能由現有紀錄推論它們已有逐節點事件。當時 testbed 從 PyMTLF 日誌擷取的 `resourceLocation` 是發起端 Go 的私有代理位置，不是接收端產生的 SBI 訂閱資源 ID；Slice 1 更新後須重新驗證該擷取結果。故障情境的收集器目前只挑 Root→A／A* 及 A／A*→A1／A2 的資源紀錄，不完整收集 Root→B／C 等未受影響邊。

本次現況判斷對照 `NWDAF/internal/sbi/processor/ml_model_training.go`、`NWDAF/internal/context/ml_model_training_routes.go`、`PyMTLF/src/py_mtlf/core/fl_server.py`、`fl_root.py`、`fl_branch.py`、`fl_topology.py`、`experiment_recording.py`，以及唯讀 testbed 參考中的 `fl_experiment.py`、`fl-experiment-run.py`；後者不是本次修改目標。

以下識別碼盤點以 Slice 1 開工前的程式為基準；目前本地實作已完成該 slice，結果見[詳細計畫](./slices/Slice%201%20ML%20Model%20Training%20Subscription%20Resource%20Identity%20and%20Lifecycle%20Detailed%20Plan.md)。本文件用以下概念名稱區分識別碼，避免將它們都稱為 subscription ID；`callbackRouteId` 與 `subscriptionResourceId` 並非新增的 wire 欄位：

| 名稱 | 產生與用途 | 是否可作為論文中的訂閱資源 ID |
| --- | --- | --- |
| `callbackRouteId` | 發起端 Go 在送出 Create 前建立的本地 callback 路由 token；Slice 1 前也被用作發起端 route 主鍵及回給 PyMTLF 的私有資源路徑 | 否 |
| `subscriptionResourceId` | 接收端 Go 建立 SBI 訂閱後，在 `Location` 回傳的對外訂閱資源 ID | 是；須連同接收端身分或資源位置辨識所屬訂閱 |
| `mlCorreId` | 同一次 hierarchical FL procedure 的關聯識別碼 | 否；它可跨多條訂閱關係共用 |
| `notifCorreId` | Notification 關聯識別碼 | 否；不取代訂閱資源 ID |

Slice 1 開工前的資料流為：父節點 PyMTLF 請自己的 Go 建立訂閱；父節點 Go 先以 `callbackRouteId` 預留路由及 callback URI，向子節點 Go 建立訂閱；子節點 Go 以 `Location` 回傳 `subscriptionResourceId`；父節點 Go 將訂閱資源 ID 存於以 `callbackRouteId` 為主鍵的路由中，卻回給父節點 PyMTLF 以 callback 路由識別碼為路徑的私有 `Location`。因此當時 PyMTLF 日誌與 testbed 擷取到的是 `callbackRouteId`，無法直接支持「舊、新 SBI 訂閱 ID」的論文主張。

Slice 1 已在本地實作確認：建立期間讓 `callbackRouteId` 擔任 callback token／暫存關聯；確認收到 `subscriptionResourceId` 後，以接收端 NF 身分與資源 ID 識別正式訂閱。父節點 PyMTLF 收到的是**自己 Go 的私有 URL，但其資源路徑使用接收端的訂閱資源 ID**；後續 PUT／PATCH／DELETE 仍交由該 Go 轉送，不能直接呼叫子節點的對外 URL。Go 保留必要的 callback 查找，讓既有 notification URI 在訂閱存續期間可用。Create、早到 callback、失敗清理與不同接收端相同 ID 的本地測試已完成；正式多節點 testbed 整合仍待驗證。

## 2. 共通必備能力與紀錄

| 項目 | 現況及缺口 | 接下來需要的結果與主要擁有者 |
| --- | --- | --- |
| 真實訂閱識別與生命週期 | Slice 1 已讓接收端 Go／PyMTLF 共用同一資源 ID，發起端以接收端 NF 身分與資源 ID 管理正式路由；本地測試已通過，正式多節點驗證仍待執行 | **各節點事件紀錄**須使用已取得的真實 `subscriptionResourceId`，並保留可確認的 peer 身分及 `mlCorreId`；後續逐筆 Create／更新／終止證據屬第 2 項，不把 Slice 1 的資料結構調整誤當成事件紀錄已完成。 |
| 逐節點訂閱與拓樸事件 | 現有 `observations.jsonl` 缺少 Branch／Leaf 的訂閱事件；Root round 與 replacement 事件不足以證明各 edge 何時建立、保留或消失 | **各節點 PyMTLF，第 2 項**擴充 node-local `observations.jsonl`，記錄自己發起或收到的訂閱操作與結果、可確認的父子 NF、方向、正式 `subscriptionResourceId`、關聯 ID、時間及重要 instruction／report 摘要。發起端已可透過自己的 Go 私有回應取得 peer `Location` 對應的資源 ID；不讓 PyMTLF 直接使用 peer 對外 URL，也不為此新增 Go 實驗紀錄。 |
| 三種拓樸狀態 | 既有 `flTopology`／`flTopologyReport` 可傳遞建立意圖與逐級 status，但現有實驗紀錄未完整留下意圖、已確認關係及 Root 接受／拒絕快照；舊 schema 沒有 protocol `topologyVersion` | **Root／各直接父節點 PyMTLF** 依既有 instruction／report 與實際訂閱關係記錄 requested、realized、accepted 的不同時點及 Root 決定。逐輪成功貢獻者另記；若論文仍要求版本證據，須另行決策，不預設新增 wire 欄位。 |
| 訂閱重要內容 | 現有資源日誌尚未完整記錄訂閱操作與拓樸決策 | **各節點 PyMTLF** 記錄足以解釋決策的 request／accepted representation 摘要，例如標準 `mLEventSubscs`／`mLModelTrainInfos`、`flTopology` 的接收節點、`children`／priority、`policy`、`strategy`、`reportAfter`，以及 `flTopologyReport` 的 status／cause；連同操作結果和原因。Go 仍須正確傳回操作回應及 `subscriptionResourceId`，但不需為同一內容再寫一份實驗檔。避免重複寫入完整 model artifact、權杖或無關 payload；候選、成功訂閱及 confirmed edge 必須分開。 |
| 逐輪與模型證據 | Root 已記錄 selected／successful／failed participant IDs、accepted 與 validation；Branch／Leaf 只在啟用本地驗證時記模型評估 | **PyMTLF，第 2 項**保留 Root 每次 round attempt／accepted outcome 與初始、逐 accepted round 的 validation；視情境補上 Branch 下層 selected／successful／failed set、Leaf 本地訓練或回報完成及失敗原因。Root 模型結果須能對回相同 `mlCorreId`、`roundInd`；圖表用 one-based accepted round、初始評估為 round 0。各節點本地 validation 可選，不應被誤認為所有情境的必要前置。 |
| 修復時間線與事後操作統計 | Controller 有故障注入時間；Root 有故障偵測與 replacement ready；尚缺明確記錄 `flTopology` 修復指令與各新 edge 分別完成的時間 | **Controller** 提供故障注入時間；**PyMTLF，第 2 項**留下 detection、instruction、各新 edge ready、Root 對當時 realized topology 的接受決定及首次 accepted contribution 等原始時間點及逐筆訂閱操作，不另記修復成功／失敗標記。**第 4 項**只確認原始資料可交接；正式實驗後才由離線分析統計訂閱資源操作發起次數與時間分段。不在 PyMTLF 維護線上計數器，也不在本批增設 Go wire-level 計數或內部重試紀錄。跨程序時間須可對齊或明示誤差。 |
| 五 seed 配對與失敗保留 | 既有論文配對每條件只有一次；testbed run metadata 可保存 scenario，但尚未證明五 seed E0–E2b 全套可追蹤 | **Testbed 執行／收集端**保存 workload、scenario、seed、資料分割、初始模型、訓練設定、觸發 round、程式版本與 run ID；所有失敗或未恢復 run 仍保留原始事件與模型曲線。PyMTLF 只需產生可靠原始紀錄，不實作 CI、AUC 或 recovery 判定。 |

逐節點實驗檔以 PyMTLF 為寫入者；Go 仍負責傳遞真實 SBI 資源 ID 與操作結果，**不是本批實驗紀錄的第二個寫入者**。發起端的逐筆操作紀錄可供實驗後統計；這個數字是 PyMTLF 可觀察的訂閱資源交互次數，不等同精確的跨 NF HTTP 傳輸次數。Testbed 收集與離線關聯另屬其擁有者；不把收集腳本的修改算作 PyMTLF 已完成的能力。

## 3. 各實驗額外需要的能力

| 情境 | 目前可沿用 | 必須補足或確認 |
| --- | --- | --- |
| E0：無故障 baseline（必要） | 固定三個 Branch group 的訓練、Root 每輪 outcome／validation、final model | 五 seed 配對紀錄；以既有 `flTopology`／`flTopologyReport` 記錄初始建立、Root accepted topology、每條真實 `subscriptionResourceId` 的建立證據；確認未出現非預期重建。 |
| E1：A 故障、A* 接手（必要） | 既有 testbed 已觀察單一 Branch replacement、degraded round、A* 建新下層關係以及 Root failure／ready 事件；不取回舊 Leaf 計算結果 | 以發給 A* 的 `flTopology.children`、逐級 `flTopologyReport` status 和真實 `subscriptionResourceId` 證明 A→A1/A2 轉成 A*→A1/A2，且 Root→B/C 與 B/C 子樹不變；記錄各新 edge 完成、首次 accepted A* contribution、修復前後拓樸與五 seed 成敗。不能把「ready」當作首次貢獻。 |
| E2a：A 消失，A1／A2 直掛 Root（後續實驗） | 既有 recursive instruction／report 可描述深度不一致的拓樸；現行程式未能跑通此情境 | **現行 PyMTLF Root 尚非此情境的可執行實作**：static topology config 以 `branch_groups` 組織；Root 直接子節點選擇與結果驗證按 Branch／`HIERARCHY_AGGREGATE` 處理。**第 3 項**補足 Root 盡力嘗試 A1／A2 的訂閱與關係形成，以及直接 Leaf 與 Branch aggregate 同輪參與所需的樣本數權重及模型取用。兩條新邊均已確認的結果由第 2 項原始紀錄判讀；不預設改用附件 B wire。 |
| E2b：只有 A1 直掛 Root（新版論文主要實驗） | 可沿用 E2a 的第 3 項直接重掛與混合深度訓練路徑 | **第 3 項**仍嘗試 A1／A2，A2 不可用時 realized topology 只含 A1。Root 沿用本層訓練門檻判斷能否繼續，不另設「至少接回一個 Leaf」的修復門檻；接回數由實際關係判讀。第 2 項記錄 A2 不可用與拓樸變化，資料覆蓋變動由實際參與者和資料分割對照。 |

E2a／E2b 形成混合深度拓樸後，現行只計 active Branch groups 的 Root 可用數與選入數判斷須改為正確作用於當時的直接子節點；`minCompletionRate` 仍依該輪選入者計算，不因此新增 direct-repair cohort 門檻。設定快照須保留實際採用的 Root policy，若變更 policy，不能把結果描述成僅改故障類型。E0–E2b 的兩種工作負載與故障邊界仍依實驗矩陣。上述「需補」是能力盤點，不是已經通過 testbed 的宣告。

## 4. 原始紀錄與離線分析邊界

逐節點事件至少要能以 `recordedAt`、`mlCorreId`、本節點及可確認的對端 `nfInstanceId`、事件／操作類型、方向、結果及原因串起同一條父子關係；正式訂閱事件使用接收端的 `subscriptionResourceId` 與其 owner，`callbackRouteId` 如需保留僅供 callback 排錯，名稱不得叫 `subscriptionId`。接收端 PyMTLF 的私有 Create 目前未直接收到可驗證的發起端 `nfInstanceId`，不得在該端單筆事件中推定。訂閱建立嘗試、收到 Create 成功回覆、完成 preparation／形成 confirmed edge、Root 接受拓樸是四種不同事實。更新與刪除亦保留嘗試及結果，不能把發出命令視為成功。

原始紀錄交給離線分析計算五 seed mean／95% CI、固定 post-failure window accuracy AUC、paired E0 endpoint 差值、rounds-to-recovery、各時間分段、各情境成功數，以及訂閱資源 Create／PUT／PATCH／DELETE 的發起次數。對同一操作的發起與回覆不重複計數；notification 如需統計，另列類別，不混入訂閱資源操作。Recovery 採實驗文件所定義的「修復後 accuracy 進入 E0 對應 round 的 95% CI，且連續維持兩個 accepted rounds」；這不是 PyMTLF 的運行期判斷。未恢復的 run 明記 `not recovered`，不從分母或圖表中刪除。Final official test 與逐輪 Root validation 分開保存。

## 5. 進入詳細實作計畫前的確認

1. 以現有遞迴拓樸契約的 producer、consumer、儲存狀態及測試為基準，先同步改名為候選 `flTopology`／`flTopologyReport`，再確認第 2 項事件紀錄與 E2a／E2b 能力是否缺少實際資訊。原有 policy／strategy／report-after 和各種訓練門檻維持其已定語意；若確需擴充，與 3GPP 既有欄位分開標示，不預設附件 B 其他欄位或相容雙格式。
2. Slice 1 已完成 Go／PyMTLF 的 `subscriptionResourceId` 主鍵、peer scope 與 `callbackRouteId` callback 索引本地實作及測試；後續須在正式多節點環境驗證，並調整 testbed 收集器對識別碼的解讀。
3. 以 Slice 1 已有的 Go 私有回應為基準，確認 PyMTLF 能取得的 `subscriptionResourceId`、操作結果及必要訂閱摘要；明確定義逐筆紀錄的欄位與產生點，使發起與回覆可對回同一次操作，事後統計時不重複計數。不把 Go 內部呼叫、重試或精確 wire-level 計數列為本項工作，也不靠解析人類可讀日誌當唯一證據。
4. 驗證 E2a／E2b mixed-depth 根據目前 Go／PyMTLF 契約所需的全部變更，特別是直接 Leaf 結果型別、aggregation 權重、ADRF 讀取權限與 Root policy；此項目前不能標為實作就緒。
5. 確定 testbed 的五 seed 排程、故障事件與跨程序時鐘、run artifact 收集，以及由 PyMTLF 原始操作紀錄離線統計訂閱資源交互次數的界線。參考 testbed 目前是唯讀來源；此文件不授權修改其部署或執行實驗。

本盤點的下一步是依上述確認拆出可獨立審查的功能與紀錄詳細計畫；不在此文件預設實作 slice 數量或宣告 E0–E2b 已完成。
