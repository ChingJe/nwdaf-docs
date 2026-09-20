# Slice 2 — 逐節點協定與拓樸證據實作計畫

日期：2026-09-20

狀態：計畫已確認，待程式實作與跨節點驗證；E0–E2b 證據、三層 JSON 欄位、事件名稱、寫入邊界及驗收測試已規劃。

本計畫從 [E0–E2b 實驗要求](../Hierarchical%20FL%20E0-E2b%20Experiments%20and%20Testbed%20Context.md)出發，先確認每個節點**送出、收到、處理及回報**哪些資訊，再據此設計 JSON 紀錄。不能只保存 Root 或父節點聲稱已送出的內容；接收端實際收到的 subscription、自己的處理結果，也都是證明協定執行的原始事實。本次將重構整套實驗事件紀錄；現有 recorder 僅供盤點，不預設沿用其事件名稱、欄位或格式。本文件不修改既有 `x-flTopology`／`x-flTopologyReport` 協定，也不納入 E3。

## 1. 實驗需要回答什麼

| 情境 | 從原始紀錄應可確認的事 |
| --- | --- |
| E0：無故障對照 | 初始 Root–A/B/C–Leaves 訂閱是否由各接收端收到並完成準備；Root 是否接受形成的拓樸；正常訓練期間是否維持原有關係；每輪參與者與 Root validation 軌跡為何。 |
| E1：A→A* | A 停止與 Root 偵測的時點；A* 實際收到什麼接手訂閱，A1/A2 又實際收到 A* 的哪些新訂閱；新舊 `subscriptionId`、`mlCorreId` 延續、B/C 關係不變、降級 rounds 與 A* 首次 accepted contribution。 |
| E2a：A1/A2 直掛 Root | A1/A2 是否各自收到並接受 Root 的新訂閱；Root 是否形成並接受深度不一致的拓樸；B/C 子樹是否保留；直接 Leaf 和 Branch 是否都能參與後續 accepted rounds。 |
| E2b：僅 A1 直掛 Root | Root 原本要求什麼、A1 實際收到並接受什麼、A2 未形成新關係；Root 是否依當時條件接受部分修復；資料覆蓋與模型結果相對 E0 如何變化。 |

各工作負載／情境使用五個配對 seeds 是實驗條件，不是已完成的結果。失敗、未完成或未恢復的 run 必須保留已產生的紀錄，不能只收成功樣本。

## 2. 按實際傳遞方向盤點原始事實

Root 對自己的直接子節點是發起端；Branch **對上接收 Root 的 subscription，對下發起給 Leaves 的 subscriptions**；Leaf 接收直接父節點的 subscription。E2a／E2b 修復後，Root 也可能直接對 Leaf 發起訂閱；E2b 不預設已不可用的 A2 會收到新訂閱。每個節點只記自己實際看見或作出的事，不把別人的判定抄成自己的觀測。

本文件的 `subscriptionId` 指接收訂閱的 NWDAF 建立並回傳的正式訂閱 ID；跨節點比較時連同接收端 `nfInstanceId` 辨識。它不是 `mlCorreId`、`notifCorreId` 或 callback 用的本地識別碼；本計畫不新增紀錄專用 ID。

| 行為／事實 | 直接觀察者 | 為何要留下 |
| --- | --- | --- |
| 本次 run 的情境、工作負載、seed、資料分割、初始模型、訓練設定與故障安排 | 實驗執行端的 run metadata | 確認同 seed 的 E0–E2b 能配對；記清 E2a／E2b 若調整 Root 接受條件，不用在每筆節點 log 重複整份設定。 |
| 向哪個直接子節點發起 Create、更新或刪除；實際下發的訓練與拓樸內容、操作結果，以及成功建立後取得的 `subscriptionId` | 發起端 Root 或 Branch | 證明上層**要求**什麼、何時要求，並核對操作失敗或結果不明時沒有被誤算為成立的關係；操作紀錄供事後統計發起次數。 |
| 收到哪個 Create、更新或刪除；接收時的訂閱內容、當時已知的 `subscriptionId`、本地接受／拒絕／處理失敗的結果 | 接收端 Branch 或 Leaf；E2 修復時也包括直接接收 Root 訂閱的 A1/A2 | 證明指令真的抵達並被接收端如何處理。Root 發出 A* 接手指令，不能代替 A* 收到該指令的證據；A* 發出新訂閱，也不能代替 A1/A2 實際收到的證據。 |
| 訂閱內容中影響本次訓練的實際值：標準訓練任務／資料與模型要求、`x-flTopology` 候選與 node policy／strategy、更新時的 `roundInd` 及模型參照等 | 發起端保留送出內容；接收端保留收到內容 | 核對傳遞前後的關鍵要求，並解釋各節點為何選擇、訓練、回報或拒絕；不把 model artifact、令牌或無關 payload 複製進事件。欄位與摘錄規則見第 5 節。 |
| 接收端完成 preparation、確認參與，或因要求不符而拒絕；後續訂閱更新／終止造成的本地狀態變化 | 接收該訂閱的 Branch 或 Leaf | 區分「收到了訂閱」、「回覆建立成功」與「可參與訓練」；E2b 的 A2 若未完成，不能被當作已形成的邊。若結果已在相關回覆或 notification 表達，就隨該筆通訊記錄，不預設另寫一筆。 |
| 子節點實際產生並送出的 topology report／訓練結果，以及直接父節點實際收到和採用的內容 | 報告的發出端與接收端各記自己那一側 | 讓 Leaf→Branch→Root 的逐級回報有證據；發出或排送不等於父節點已收到，也不等於該輪已被接受。 |
| 某條直接父子關係何時經必要確認而成立、失效或被替換，對應的接收端與 `subscriptionId` | 管理該直接關係的父節點，對照接收端的處理紀錄 | 從已成立的關係重建 realized topology；以修復前後的關係與 ID 比較 A／A*、Root→A1/A2 及未受影響的 B/C，不額外寫「B/C 沒變」布林值。 |
| Root 收到的逐級 topology report、實際形成的拓樸、接受或拒絕它時使用的條件與結果 | Root | 將 initial intention、realized topology、accepted realized topology 分開；E2b 尤其不能把只形成 A1 的結果寫成 A1/A2 都已接回。 |

例如 E1 要能循序對照：Root 發出給 A* 的訂閱 → A* 收到並處理 → A* 發出給 A1/A2 的訂閱 → A1/A2 各自收到並處理 → A* 收到下層回報並向 Root 回報 → Root 判斷是否接受修復。E2a 由 Root 直接對 A1/A2 發起；E2b 只記 Root 實際嘗試的對象與接收端確實收到的內容，不替 A2 虛構接收事件。這是證據路徑，**不是**要求每一步各發明一種事件或一個新 ID。

## 3. 故障、訓練與模型結果

| 原始事實 | 直接觀察者 | 為何要留下 |
| --- | --- | --- |
| 故障注入的實際對象與時間；Root 判定某節點失效的時間 | 注入由實驗控制端記；判定由 Root 記 | 計算 failure→detection；節點 log 不猜測外部控制器何時停止 A 或 A2。 |
| 修復指令送出與接收、新關係分別準備完成、Root 接受修復、修復後節點首次列入 accepted Root contribution 的時間 | 各時點由親自送出、收到或判定的節點記 | 拆開 detection→instruction、instruction→ready、ready→first accepted contribution；關係就緒不等於已貢獻。 |
| 每個 round 向哪些直接子節點下達訓練要求；各接收端收到哪個 `roundInd` 與模型參照、完成或無法完成本地工作、回報了什麼 | Root／Branch 記各自下發與收回；Branch／Leaf 記各自接收與執行 | 核對正常、降級與 mixed-depth 訓練確實走過各節點；跨層 `roundInd` 不預設相同，也不把某節點收到指令等同於成功貢獻。 |
| 每個 Root／Branch 本地 round 選入、成功、失敗的直接子節點及該輪是否被接受 | 各自執行聚合的 Root 或 Branch | 證明 E0 的完整參與、E1 故障時 B/C 仍可支援 accepted rounds、E2 修復後直接 Leaf 與 Branch 的實際貢獻。 |
| Root 初始模型及每個 accepted Root round 在固定 validation set 上的 accuracy、loss、對應 round 與量測時間 | Root | 形成主要學習曲線，供五 seed 比較及恢復判定；初始評估和 accepted round 須能區分。Branch／Leaf 本地 validation 若有啟用可留作診斷，不取代 Root 曲線。 |
| 完成後的 final model 與獨立 official test set 結果；各 Leaf 資料量與類別分布 | final model 由 Root；test 評估與資料分割資訊由實驗端 | 比較 endpoint；E2b 要能分清 A2 資料缺席和協定修復失敗。Final test 不混入逐輪 validation。 |

## 4. 節點紀錄的三個大類

下表逐項列出 E0–E2b 所需的實際事件。分類用來安排證據，不是固定 `recordType`，也不要求每列各寫一筆獨立 log；同一次操作的要求、回覆與結果可合理地保存在同一筆紀錄。實驗控制端的 run metadata 與故障注入另存，不算節點紀錄。

| 分類 | 實際事件 | 直接紀錄者 | 實驗用途 |
| --- | --- | --- | --- |
| Model Training 通訊 | 發起訂閱建立並取得操作結果 | 發起端 Root／Branch | 留下送出的訓練要求與 `x-flTopology`、目標節點及成功後取得的 `subscriptionId`；追查 E1／E2 的新關係。 |
| Model Training 通訊 | 收到訂閱建立並作出回覆 | 接收端 Branch／Leaf | 證明 A*、A1／A2 等節點實際收到的內容，以及本端接受、拒絕或處理失敗的結果。 |
| Model Training 通訊 | 發起訂閱更新並取得操作結果 | 發起端 Root／Branch | 留下每輪下發的 `roundInd`／模型參照，或修復時更新的拓樸指示。 |
| Model Training 通訊 | 收到訂閱更新並作出回覆 | 接收端 Branch／Leaf | 證明訓練或修復指示實際抵達，以及接收端如何處理。 |
| Model Training 通訊 | 發起訂閱刪除並取得操作結果 | 發起端 Root／Branch | 追查受影響關係的清理；送出刪除不等於對端已刪除。 |
| Model Training 通訊 | 收到訂閱刪除並作出回覆 | 接收端 Branch／Leaf；僅在要求實際抵達時 | 證明哪份接收端訂閱實際終止；失聯節點不會憑空產生接收紀錄。 |
| Model Training 通訊 | 發出 preparation／拓樸狀態 notification | 發出端 Branch／Leaf | 留下參與結果與 `x-flTopologyReport`，包括已確認、失敗或未形成的下層關係。 |
| Model Training 通訊 | 收到 preparation／拓樸狀態 notification | 直接父節點 | 證明逐級回報抵達，供 Root 重建 realized topology。 |
| Model Training 通訊 | 發出本輪模型結果 notification | 發出端 Branch／Leaf | 證明結果實際上報，而非只完成本地計算。 |
| Model Training 通訊 | 收到本輪模型結果 notification | 直接父節點 | 區分結果已抵達與結果已納入 accepted round。 |
| 節點內部決策 | 選定本次要嘗試的直接子節點候選 | 擔任父節點的 Root／Branch | 解釋 A* 或 A1／A2 的選擇；同一選擇結果可交代未嘗試候選，不為「未嘗試」另造事件。 |
| 節點內部決策 | 確認直接父子關係已成立 | 管理該關係的父節點 | 將 subscription 回覆成功與 preparation 後確認參與分開，據此重建 realized topology。 |
| 節點內部決策 | 判定直接子節點失效、關係不再可用 | 管理該關係的父節點 | 留下 Root 偵測 A 失效的時點，以及失去的是哪條關係。 |
| 節點內部決策 | 決定修復方式及對象 | Root | 區分 E1 選 A*、E2a 直掛 A1／A2、E2b 將 A1／A2 列為修復候選的意圖；實際形成哪些關係由後續確認紀錄呈現。 |
| 節點內部決策 | 接受或拒絕 realized topology | Root | 證明 E2b 的部分修復是否滿足當時 policy；收到 report 不等於接受拓樸。 |
| 節點內部決策 | 完成本地一輪聚合判定 | 各自聚合的 Root／Branch | 留下 selected／successful／failed 直接子節點及該輪是否 accepted；辨認 degraded rounds 與 A* 首次有效貢獻。 |
| 模型量測與產物 | 評估初始模型 | Root | 提供圖中的 round 0 基準；它不是 accepted training round。 |
| 模型量測與產物 | 評估每個 accepted Root round 的模型 | Root | 留下 validation accuracy／loss、輪次及時間，供 E0 配對曲線與事後 recovery 分析。 |
| 模型量測與產物 | 保存完成模型 | Root | 留下最後模型的產物身分與對應輪次；使用第 5 節定義的事件與欄位。 |
| 模型量測與產物 | 評估完成模型的 official test set | 實際執行測試的節點或實驗端 | 留下 endpoint test 結果，不與逐輪 validation 混用。 |

若 Branch／Leaf 有啟用本地 validation，可另外保存其量測作診斷，但不取代 Root 的主要曲線。`x-flTopology`／`x-flTopologyReport` 仍是本專案候選擴充，不稱為 3GPP 已定義欄位。例如 A* 收到並接受 Root 的訂閱，是通訊結果；A* 選擇 A1／A2，是內部決策；向 A1／A2 建立訂閱，又是通訊。preparation 的接受／拒絕若已在回覆或 notification 表達，不再重寫一筆內部事件。

## 5. 從論文證據到三層欄位設計

論文第 6 節將協定結果列為主要證據：requested／realized／accepted topology、修復是否成功、`mlCorreId` 延續、舊／新訂閱與未受影響的關係、逐輪貢獻者及修復時間。模型 accuracy／loss 是輔助證據。下表先區分**必須留下的原始事實**與可以事後計算的結果；不把每個論文指標都變成 PyMTLF 事件。

| 論文要確認的事 | 原始紀錄的最小來源 | 不必另設的欄位／事件 |
| --- | --- | --- |
| E0–E2b 同 seed 可配對 | 執行端 run metadata；節點紀錄的 `mlCorreId` | 不在每筆節點事件複製 seed、分割及全部訓練設定。 |
| 拓樸意圖、形成與接受不同 | 實際送出／收到的 `x-flTopology`、`x-flTopologyReport`，父節點確認的直接關係，以及 Root 對 realized topology 的接受決定 | 不虛構 wire `topologyVersion`；不把候選或訂閱建立成功直接當作 confirmed edge。 |
| 同一程序局部修復、B／C 未重建 | 每條關係的接收端 `nfInstanceId`＋真實 `subscriptionId`，以及全程訂閱操作；各節點的 `mlCorreId` | 不另記 `unaffected=true` 或第二種訂閱 ID。 |
| 故障到首次重新貢獻的各階段 | 執行端注入時間、Root 偵測、修復 instruction 發起、各新關係確認、Root accepted round 的原始時間 | 不在線上計算 latency 或 `recovered`。 |
| 降級與修復後仍有 accepted rounds | Root／Branch 各自的 selected／successful／failed direct-child set、`roundInd` 與 accepted 判定 | 不把 replacement ready 當作首次有效貢獻；不假設跨層 `roundInd` 相同。 |
| 學習曲線、endpoint、E2b 資料缺席 | Root 初始與逐 accepted round 的 validation；完成模型；獨立 test 結果；執行端的 Leaf 分割與故障紀錄 | 五 seed CI、AUC、paired 差值、class-wise effects、recovery rounds 均事後計算。 |
| 控制面交互數量 | PyMTLF 各發起端實際留下的訂閱與 notification 操作 | 事後按發起端紀錄計數；不將發出與收到兩側相加，也不宣稱是精確 SBI HTTP／Go retry 數。 |

以下三層描述的是**欄位適用範圍**，不是三種新的檔案或額外識別碼。節點各自寫入該 `mlCorreId` 目錄的 JSONL；實驗控制端的 run metadata、故障注入與離線 test 結果維持獨立來源。欄位只在本節點確實知道其值時出現，不填猜測值或無意義的 `null`。`recordType` 指具體發生的事；其所屬三大類由定義表判讀，不另存一個重複的類別欄位。下列名稱是**重新設計的實驗紀錄**，不預設沿用現有 recorder 的事件 taxonomy。

### 5.1 第一層：所有逐節點事件共用

| 欄位 | 含義與紀錄目的 |
| --- | --- |
| `recordedAt` | 本節點觀察到該結果的 UTC 時間；供節點內排序及與其他來源對時。跨節點時間差仍須檢查校時。 |
| `nfInstanceId` | 寫入這筆紀錄的 NWDAF 身分；不另設 `actor`／`observer`。 |
| `mlCorreId` | 本筆所屬的 hierarchical FL procedure；用同一值串起 Root、Branch、Leaf，而非識別單條訂閱。不能關聯至有效 procedure 的請求留在一般錯誤日誌，不編入某次實驗 JSONL。 |
| `recordType` | 這筆紀錄具體是哪種通訊、內部決定或模型結果；本文件 §5.5 固定目前實驗會使用的名稱。 |

不新增 `runId`、operation ID、獨立 `recordCategory` 或 topology version 作為逐筆必填欄位。Run／seed 配對由執行端 metadata 管理；同一 `mlCorreId` 的資料夾是節點本地的整理方式，不取代逐筆 `mlCorreId`。

### 5.2 第二層：三大類內可重用的欄位

「類別共用」指相近事件沿用同名、同義欄位，**不表示該類每筆紀錄都要填齊**。例如建立訂閱的發起端在收到成功回覆前還沒有正式 `subscriptionId`，接收端也不一定能驗證發起者身分。

| 類別 | 可重用欄位 | 用途與出現條件 |
| --- | --- | --- |
| Model Training 通訊 | `operation`、`direction` | 區分 `CREATE`／`PUT`／`PATCH`／`DELETE`／`NOTIFY` 與 `SENT`／`RECEIVED`；事後只計發起端操作，不將兩側重複相加。 |
| Model Training 通訊 | `startedAt`、`subscriptionId`、`targetNfInstanceId`、`sourceNfInstanceId`、`outcome`、`cause` | `startedAt` 是發起端 PyMTLF 交給本機 Go 要求的時間，或接收端 PyMTLF 從本機 Go 收到要求的時間；第一層 `recordedAt` 是各自取得本地操作結果的時間，包含失敗或逾時。若只留下完成後一筆紀錄，仍可拆出兩個本地時間點；它們不是對端 NWDAF 送達或處理時間的直接量測。`subscriptionId` 只在正式資源 ID 已知時記，並以接收端身分界定其範圍。`targetNfInstanceId` 用於發起端已選定對象時；`sourceNfInstanceId` 僅在接收端能確認來源時使用。`outcome`／`cause` 只反映本端實際取得的操作結果。 |
| 節點內部決策 | `roundInd`、`childNfInstanceId`、`subscriptionId`、`accepted`、`cause` | `roundInd` 是作決定者自己的 local round；直接關係決定記其 child 及正式資源 ID；拓樸或聚合決定才使用 `accepted`。原因有實際值才記，不編造 A2 未參與的理由。 |
| 模型量測與產物 | `roundInd`、`dataset`、`sampleCount` | 有對應 local round 時才記 `roundInd`；初始模型評估沒有 accepted round。`dataset`／`sampleCount` 用於實際評估；保存完成模型不需要假裝做過評估。 |

`subscriptionId` 由接收端建立的資源回覆提供，跨接收端須與該端 `nfInstanceId` 一起比較；`notifCorreId` 保存於 `message` 內，用於訂閱及通知對照，不是訂閱主鍵。通訊結果若要表示 HTTP status，僅能標示 PyMTLF 實際收到的 Go 私有回覆；不能把它冒稱為已量到的跨 NWDAF SBI 封包結果。

### 5.3 第三層：按事件需要附加的內容

下表中的欄位可以由相近事件共用，但不為了格式整齊而複製不適用的資料。`message` 是**實際送出或收到的 Model Training JSON 的必要欄位摘錄**，不是新增的 protocol 欄位；摘錄內保留原 wire 欄位名稱與實際值，不另造抽象 `payloadHash` 或只記「有拓樸」布林值。

| 事件群 | 需要補的欄位 | 證據用途與來源 |
| --- | --- | --- |
| 訂閱 Create／PUT／PATCH 的送出及接收 | `message`：有提供時保留 `mLEventSubscs` 的訓練任務／model interoperability、`mLModelTrainInfos`、`mLPreFlag`、`roundInd`、模型參照，以及完整 `x-flTopology`；建立／更新的結果及正式 `subscriptionId` 由第二層欄位表示 | 發起端記自己真正交給 Go 的要求；接收端記自己真正收到的表示。這可核對候選、priority、policy、strategy、report-after，以及 E1／E2 的接手或直掛指示。Model URL／ADRF 參照可記，模型檔內容與權杖不進 JSONL。 |
| 訂閱 DELETE 的送出及接收 | 第二層的接收端身分、`subscriptionId`、時間與操作結果即可；有實際 `cause` 時才補 | 比對關係是否被清理；發起端送出不證明失聯的接收端已刪除。 |
| Preparation／拓樸 notification 的送出及接收 | `message.notifCorreId`、`message.x-flTopologyReport`；有實際 status／cause 時保留在原 report 結構中 | 逐級核對 confirmed／未形成的 descendant，並區分發出報告和上層真正收到。 |
| 模型結果 notification 的送出及接收 | `message.notifCorreId`、`message.roundInd` 及實際上報的模型資訊／模型參照與資料量 | 說明哪個 local round 的結果離開子節點、抵達直接父節點；父節點是否採用仍看自己的聚合判定。 |
| 候選選擇、故障偵測與修復決定 | 選擇時用 `candidateNfInstanceIds`、`selectedNfInstanceIds`；偵測直接子節點故障時使用第二層的 `childNfInstanceId`，若屬某個本地 round 再附 `roundInd` | 解釋 Root 選 A* 或 Root 直掛 A1／A2；正式拓樸指令仍以相應訂閱紀錄中的 `x-flTopology` 為準，不複製成第二份 requested tree。 |
| 直接關係確認或失效 | `childNfInstanceId`、`subscriptionId`；失效原因可確認時附 `cause` | 父節點只記自己管理的直接邊，據以重建 realized topology 及比較修復前後資源；用 `EDGE_CONFIRMED`／`EDGE_UNAVAILABLE` 區分結果，不另複製 `edgeState`。 |
| Root 接受或拒絕拓樸 | `realizedTopology`、`accepted`；拒絕原因可確認時附 `cause` | `realizedTopology` 是當時由已確認關係形成的快照，`accepted=true` 才代表 accepted realized topology；不另複製一份 `acceptedTopology`。當次適用的靜態 policy 由 run metadata 對照，不在事件重複整份設定。E2b 可比較 instruction 中的 A1／A2 與快照中真正形成的 A1。 |
| Root／Branch 本地聚合結果 | `roundInd`、`selectedNfInstanceIds`、`successfulNfInstanceIds`、`failedNfInstanceIds`、`accepted` | 看出 B／C 支援的 degraded rounds、A* 首次出現在 accepted Root outcome，以及 E2a 中直接 Leaf 與 Branch 是否同輪參與。各層只記自己的 local round。 |
| 初始／逐輪／可選本地模型評估 | `evaluationStage`、`dataSplit`、`accuracy`、`loss`，並使用第二層的 `dataset`、`sampleCount`；有對應 round 才附 `roundInd` | `dataSplit` 區分 `VALIDATION` 和 `TEST`，`evaluationStage` 區分初始、Root accepted round 與可選的 Branch／Leaf 本地評估。只有實際執行評估者寫入，不將 official test 偽裝成每輪 Root validation。 |
| 完成模型保存 | `artifactFile` 與其所屬 `roundInd` | 識別供後續 official test 使用的完成模型；不要求新增 digest／hash 驗證或沿用舊 recorder 的事件名稱。 |

若 official test 由 testbed 離線執行，它的結果與使用的完成模型在執行端 artifact 中關聯，不強迫未執行該測試的 PyMTLF 寫一筆評估。`message` 的摘錄範圍、逐節點範例與預計寫入邊界見 §5.5–5.8；實作時仍需以真實呼叫結果核對。

### 5.4 每個事件實際套用的欄位表

第 4 節列出的**每一筆逐節點事件**都先使用 §5.1 的 `recordedAt`、`nfInstanceId`、`mlCorreId`、`recordType`。下表再列它使用 §5.2 哪些類別共用欄位，以及 §5.3 哪些事件內容；「可確認時」才填的欄位不因列在表中就變成必填。通訊事件的 `recordedAt` 均是**該紀錄節點的本地結果時間**，不是推定的對端完成時間。實驗控制端的故障注入與離線 official test 若不由 PyMTLF 執行，不套用節點事件三層表。

| 第 4 節事件 | §5.2 類別共用欄位 | §5.3 事件需要的欄位 |
| --- | --- | --- |
| 發起訂閱建立並取得操作結果 | `operation=CREATE`、`direction=SENT`、`startedAt`、`targetNfInstanceId`、`outcome`；成功取得資源後 `subscriptionId`，失敗原因可確認時 `cause` | `message` 中本端實際送出的訓練欄位與 `x-flTopology`。 |
| 收到訂閱建立並作出回覆 | `operation=CREATE`、`direction=RECEIVED`、`startedAt`、`outcome`；可確認時 `sourceNfInstanceId`、`subscriptionId`、`cause` | `message` 中本端實際收到的訓練欄位與 `x-flTopology`。 |
| 發起訂閱更新並取得操作結果 | `operation=PUT` 或 `PATCH`、`direction=SENT`、`startedAt`、`targetNfInstanceId`、`subscriptionId`、`outcome`；可確認時 `cause` | `message` 中本端實際送出的 `roundInd`／模型參照或修復用 `x-flTopology`。 |
| 收到訂閱更新並作出回覆 | `operation=PUT` 或 `PATCH`、`direction=RECEIVED`、`startedAt`、`subscriptionId`、`outcome`；可確認時 `sourceNfInstanceId`、`cause` | `message` 中本端實際收到的更新欄位。 |
| 發起訂閱刪除並取得操作結果 | `operation=DELETE`、`direction=SENT`、`startedAt`、`targetNfInstanceId`、`subscriptionId`、`outcome`；可確認時 `cause` | 無；正式資源身分與操作結果已足夠。 |
| 收到訂閱刪除並作出回覆 | `operation=DELETE`、`direction=RECEIVED`、`startedAt`、`subscriptionId`、`outcome`；可確認時 `sourceNfInstanceId`、`cause` | 無；只記實際抵達本端的刪除。 |
| 發出 preparation／拓樸狀態 notification | `operation=NOTIFY`、`direction=SENT`、`startedAt`、`outcome`；已知時 `targetNfInstanceId`、`subscriptionId`、`cause` | `message.notifCorreId`、`message.x-flTopologyReport`。 |
| 收到 preparation／拓樸狀態 notification | `operation=NOTIFY`、`direction=RECEIVED`、`startedAt`、`outcome`；已知時 `sourceNfInstanceId`、`subscriptionId`、`cause` | `message.notifCorreId`、`message.x-flTopologyReport`。 |
| 發出本輪模型結果 notification | `operation=NOTIFY`、`direction=SENT`、`startedAt`、`outcome`；已知時 `targetNfInstanceId`、`subscriptionId`、`cause` | `message.notifCorreId`、`message.roundInd` 與實際上報的模型資訊／參照、資料量。 |
| 收到本輪模型結果 notification | `operation=NOTIFY`、`direction=RECEIVED`、`startedAt`、`outcome`；已知時 `sourceNfInstanceId`、`subscriptionId`、`cause` | `message.notifCorreId`、`message.roundInd` 與實際收到的模型資訊／參照、資料量。 |
| 選定要嘗試的直接子節點候選 | 無固定的第二層欄位；若決定屬某個本地 round，使用 `roundInd` | `candidateNfInstanceIds`、`selectedNfInstanceIds`。 |
| 確認直接父子關係已成立 | `childNfInstanceId`、`subscriptionId` | 使用 `EDGE_CONFIRMED`；以本筆 `recordedAt` 作確認時間，不另存同義的 `edgeState`。 |
| 判定直接子節點失效、關係不再可用 | `childNfInstanceId`、已知的 `subscriptionId`；相關時 `roundInd`、可確認時 `cause` | 使用 `EDGE_UNAVAILABLE`；不重複記另一個 failed-child ID。 |
| 決定修復方式及對象 | 可確認時 `childNfInstanceId`、`cause` | `candidateNfInstanceIds`、`selectedNfInstanceIds`；後續實際下發的 tree 仍看發起訂閱事件。 |
| 接受或拒絕 realized topology | `accepted`；拒絕原因可確認時 `cause` | `realizedTopology` 快照；它與 `accepted=true` 一起界定 accepted realized topology。 |
| 完成本地一輪聚合判定 | `roundInd`、`accepted`；失敗原因可確認時 `cause` | `selectedNfInstanceIds`、`successfulNfInstanceIds`、`failedNfInstanceIds`，均只指本地 direct children。 |
| 評估初始模型 | `dataset`、`sampleCount`；不填 `roundInd` | `evaluationStage` 表示初始模型、`dataSplit=VALIDATION`、`accuracy`、`loss`。 |
| 評估每個 accepted Root round 的模型 | `roundInd`、`dataset`、`sampleCount` | `evaluationStage` 表示 Root global model、`dataSplit=VALIDATION`、`accuracy`、`loss`。 |
| 保存完成模型 | `roundInd`；不填評估資料集或分數 | `artifactFile`。 |
| 評估完成模型的 official test set | 若由節點執行：`dataset`、`sampleCount`，有對應 round 時 `roundInd` | 若由節點執行：`evaluationStage` 表示 final test、`dataSplit=TEST`、`accuracy`、`loss`；若由實驗端離線執行，改由該端保存，不產生節點事件。 |

Branch／Leaf 啟用本地 validation 時，沿用「模型評估」一列的欄位，只將 `evaluationStage` 改為本地階段，`roundInd` 指該節點自己的 round；不另造一套量測欄位。

### 5.5 固定事件名稱與 `message` 摘錄

第 5.4 節同類事件共用以下 `recordType`，再以既有 `operation`、`direction` 或事件欄位區分實際行為；不為每種 HTTP 方法、發送／接收方向各造一種事件名稱。這些是**實驗 JSONL 名稱**，不是 protocol 新欄位，也不要求沿用舊 recorder 的事件 taxonomy。

| `recordType` | 對應 §5.4 的行為 |
| --- | --- |
| `MODEL_TRAINING_OPERATION` | 訂閱 CREATE／PUT／PATCH／DELETE、preparation／拓樸及模型結果 NOTIFY 的本端發起或接收結果。 |
| `CANDIDATE_SELECTION` | 父節點選擇這次要嘗試的直接子節點。 |
| `EDGE_CONFIRMED` | 管理直接關係的父節點完成必要確認，該邊進入 realized topology。 |
| `EDGE_UNAVAILABLE` | 父節點判定自己的直接子節點失效、該邊不再可用。 |
| `REPAIR_SELECTION` | Root 決定本次修復對象及方式；後續訂閱才是實際下發的證據。 |
| `TOPOLOGY_ACCEPTANCE` | Root 接受或拒絕當時的 realized topology。 |
| `ROUND_AGGREGATION` | Root／Branch 對自己的 local round 作聚合及 accepted 判定。 |
| `MODEL_EVALUATION` | 初始模型、每個 accepted Root round，或已啟用的本地／final test 評估。 |
| `MODEL_ARTIFACT_SAVED` | Root 完成模型產物保存。 |

`MODEL_TRAINING_OPERATION.message` 只保存**本節點實際送出或收到**的 wire 欄位，名稱與巢狀結構維持原狀。對 CREATE／PUT／PATCH，保留有出現的 `notifCorreId`、`suppFeats`、`mLEventSubscs`、`mLModelTrainInfos`、`mLPreFlag`、`mLTrainRepInfo`、`roundInd`、`mLModelInfos`、`x-flTopology`、`skipFlInd`、`mLAccChkFlg`；其中 `x-flTopology` 包含完整遞迴 children、priority、policy、strategy、reportAfter，不只記 node ID。對 NOTIFY，保留有出現的 `notifCorreId`、`roundInd`、`x-flTopologyReport`、`mLModelInfos`、`statusReport`、`termTrainReq`、`delayEventNotif`；`x-flTopologyReport` 保留各節點原本回報的 status／statusTimestamp／statusCause。DELETE 沒有 message body。`mlCorreId` 放在共用欄位；`mLModelInfos` 只保留 event、modelUniqueId 與實際模型參照 `mLFileAddr`／`mLModelAdrf`，不複製模型檔、`mlFile` 內容、權杖或完整回應 body。`notifUri` 會由 Go 代理改寫，不能拿兩端 URI 是否相同當作傳遞正確性的證據，故不放入摘錄。

同一個 NOTIFY 可以同時包含拓樸與模型資訊，只寫一筆實際操作紀錄，保留兩種有出現的欄位。`message` 是實驗保存的摘錄，不宣稱發送端與接收端的整個 HTTP body 逐位元相同；回應狀態及可確認的原因放在本筆 `outcome`／`cause`，不假裝成 request/notification 欄位。

### 5.6 同一個接手情境的逐節點 JSON 範例

下例是 E1 中 Root 要求 A* 接手 A1／A2 的**節點本地紀錄**；每個 code block 屬於不同 NWDAF 的 `observations.jsonl`，不是合併後的一份檔案。UUID、時間及 `mlCorreId` 僅作範例。第一筆 Root 發送與第二筆 A* 接收指向同一份已建立訂閱；`recordedAt` 分別是兩端各自知道結果的時間。實際寫檔時每個 JSON object 佔 JSONL 一行。

Root 節點：

```json
{
  "recordedAt": "2026-09-21T10:00:02Z",
  "nfInstanceId": "10000000-0000-4000-8000-000000000001",
  "mlCorreId": "11111111-1111-4111-8111-111111111111",
  "recordType": "MODEL_TRAINING_OPERATION",
  "operation": "CREATE",
  "direction": "SENT",
  "startedAt": "2026-09-21T10:00:00Z",
  "targetNfInstanceId": "10000000-0000-4000-8000-000000000111",
  "subscriptionId": "22222222-2222-4222-8222-222222222222",
  "outcome": "SUCCESS",
  "message": {
    "notifCorreId": "root-to-a-star",
    "suppFeats": "4",
    "mLEventSubscs": [{"mLEvent": "UE_COMMUNICATION", "mLEventFilter": {}, "modelInterInfo": "image-classification-pytorch"}],
    "mLModelTrainInfos": [{"dataAvReq": {"inpEvents": [{"nwdafEvent": "UE_COMMUNICATION"}], "minNumSamples": 1, "timeWindows": [{"startTime": "2026-09-21T09:55:00Z", "stopTime": "2026-09-21T10:00:00Z"}]}, "timeAvReq": "PT300S"}],
    "mLPreFlag": true,
    "mLTrainRepInfo": {"maxResTime": 300},
    "x-flTopology": {
      "nfInstanceId": "10000000-0000-4000-8000-000000000111",
      "enabled": true,
      "priority": 50,
      "policy": {"allowAdditionalCandidates": false, "selectionMethod": "priority", "minAvailableNodes": 2, "fractionTrain": 1, "minTrainNodes": 2, "acceptFailures": false, "minCompletionRate": 1},
      "strategy": {"method": "fedProx", "aggregation": "sampleWeighted", "methodParameters": {"proximalMu": 0.01}},
      "reportAfter": {"count": 1, "unit": "round"},
      "children": [
        {"nfInstanceId": "10000000-0000-4000-8000-000000001101", "enabled": true, "priority": 100, "strategy": {"method": "fedProx", "aggregation": "sampleWeighted", "methodParameters": {"proximalMu": 0.01}}, "reportAfter": {"count": 4, "unit": "epoch"}},
        {"nfInstanceId": "10000000-0000-4000-8000-000000001102", "enabled": true, "priority": 100, "strategy": {"method": "fedProx", "aggregation": "sampleWeighted", "methodParameters": {"proximalMu": 0.01}}, "reportAfter": {"count": 4, "unit": "epoch"}}
      ]
    }
  }
}
```

A* 節點：

```json
{
  "recordedAt": "2026-09-21T10:00:01Z",
  "nfInstanceId": "10000000-0000-4000-8000-000000000111",
  "mlCorreId": "11111111-1111-4111-8111-111111111111",
  "recordType": "MODEL_TRAINING_OPERATION",
  "operation": "CREATE",
  "direction": "RECEIVED",
  "startedAt": "2026-09-21T10:00:01Z",
  "subscriptionId": "22222222-2222-4222-8222-222222222222",
  "outcome": "SUCCESS",
  "message": {
    "notifCorreId": "root-to-a-star",
    "suppFeats": "4",
    "mLEventSubscs": [{"mLEvent": "UE_COMMUNICATION", "mLEventFilter": {}, "modelInterInfo": "image-classification-pytorch"}],
    "mLModelTrainInfos": [{"dataAvReq": {"inpEvents": [{"nwdafEvent": "UE_COMMUNICATION"}], "minNumSamples": 1, "timeWindows": [{"startTime": "2026-09-21T09:55:00Z", "stopTime": "2026-09-21T10:00:00Z"}]}, "timeAvReq": "PT300S"}],
    "mLPreFlag": true,
    "mLTrainRepInfo": {"maxResTime": 300},
    "x-flTopology": {
      "nfInstanceId": "10000000-0000-4000-8000-000000000111",
      "enabled": true,
      "priority": 50,
      "policy": {"allowAdditionalCandidates": false, "selectionMethod": "priority", "minAvailableNodes": 2, "fractionTrain": 1, "minTrainNodes": 2, "acceptFailures": false, "minCompletionRate": 1},
      "strategy": {"method": "fedProx", "aggregation": "sampleWeighted", "methodParameters": {"proximalMu": 0.01}},
      "reportAfter": {"count": 1, "unit": "round"},
      "children": [
        {"nfInstanceId": "10000000-0000-4000-8000-000000001101", "enabled": true, "priority": 100, "strategy": {"method": "fedProx", "aggregation": "sampleWeighted", "methodParameters": {"proximalMu": 0.01}}, "reportAfter": {"count": 4, "unit": "epoch"}},
        {"nfInstanceId": "10000000-0000-4000-8000-000000001102", "enabled": true, "priority": 100, "strategy": {"method": "fedProx", "aggregation": "sampleWeighted", "methodParameters": {"proximalMu": 0.01}}, "reportAfter": {"count": 4, "unit": "epoch"}}
      ]
    }
  }
}
```

Leaf A1 節點（A* 逐級下發，這是另一份訂閱資源）：

```json
{
  "recordedAt": "2026-09-21T10:00:04Z",
  "nfInstanceId": "10000000-0000-4000-8000-000000001101",
  "mlCorreId": "11111111-1111-4111-8111-111111111111",
  "recordType": "MODEL_TRAINING_OPERATION",
  "operation": "CREATE",
  "direction": "RECEIVED",
  "startedAt": "2026-09-21T10:00:03Z",
  "subscriptionId": "33333333-3333-4333-8333-333333333333",
  "outcome": "SUCCESS",
  "message": {
    "notifCorreId": "a-star-to-a1",
    "suppFeats": "4",
    "mLEventSubscs": [{"mLEvent": "UE_COMMUNICATION", "mLEventFilter": {}, "modelInterInfo": "image-classification-pytorch"}],
    "mLModelTrainInfos": [{"dataAvReq": {"inpEvents": [{"nwdafEvent": "UE_COMMUNICATION"}], "minNumSamples": 1, "timeWindows": [{"startTime": "2026-09-21T09:55:03Z", "stopTime": "2026-09-21T10:00:03Z"}]}, "timeAvReq": "PT300S"}],
    "mLPreFlag": true,
    "mLTrainRepInfo": {"maxResTime": 300},
    "x-flTopology": {"nfInstanceId": "10000000-0000-4000-8000-000000001101", "enabled": true, "priority": 100, "strategy": {"method": "fedProx", "aggregation": "sampleWeighted", "methodParameters": {"proximalMu": 0.01}}, "reportAfter": {"count": 4, "unit": "epoch"}}
  }
}
```

上例的 Create 回覆成功不等於邊已確認；後續 A1 的 preparation notification、A* 的下層確認與回報、Root 的 `TOPOLOGY_ACCEPTANCE` 才能證明接手完成。Leaf 端若無可靠來源 NF 身分，就不填 `sourceNfInstanceId`，但其 `subscriptionId`、`notifCorreId` 和實際收到的內容仍可與 A* 發起端對照。

### 5.7 欄位來源、傳遞邊界與預計寫入點

以下是根據目前程式的**實作對照**，不是已完成的新紀錄功能。Root、Branch 作為父節點時，PyMTLF 先向自己的 Go 私有 gateway 發起操作；Go 代為呼叫子節點 Go 的 SBI；子節點 Go 將建立／更新／刪除要求交給自己的 PyMTLF。通知則由子節點 PyMTLF 經自己的 Go 送往父節點 Go，最後交給父節點 PyMTLF。每一端記自己的觀測，不以單邊紀錄替代另一端。

| 紀錄者與操作 | 目前可取得的值及寫入位置 | 不可直接推定的值／處理 |
| --- | --- | --- |
| 父節點 PyMTLF 發起 Create | `FLServer._create_protocol_preparation()` 已有目標 NF、實際送出的 `NwdafMLModelTrainSubsc`、`mlCorreId` 與 Go 私有回覆 `Location`；在交給 Go 前取 `startedAt`，收到本地回覆或例外後寫一筆。Slice 1 的成功 `Location` 路徑使用接收端正式 `subscriptionId`。 | 不能把本機 Go 的 callback token 或私有 URL 整串當訂閱 ID；Go／peer 在中途拒絕時，發起端只記自己拿到的結果。 |
| 接收端 PyMTLF 處理 Create | `api/ml_model_training.py:create_training_subscription()` 已取得經 Go 轉交的 body 與 `X-NWDAF-Subscription-Id`；進入處理時取 `startedAt`，本地 `FLClient.create()` 成功或 handler 決定拒絕時寫一筆。 | 此 API 沒有可直接採信的發起端 `nfInstanceId`，故不填猜測的 `sourceNfInstanceId`。Go／FastAPI 在送入此 handler 前就拒絕的無效內容，若不能連回有效 `mlCorreId`，只留一般錯誤紀錄。 |
| 父節點 PyMTLF 發起 PUT／PATCH／DELETE | `FLServer` 對既有 `participant.resource_location` 的操作已有目標 participant、正式資源 ID 及 PUT／PATCH 的實際 body；在呼叫本機 Go 前取 `startedAt`，取得回覆或例外後寫一筆。 | 私有 `Location` 只是後續交給本機 Go 的位址，事件仍以正式 `subscriptionId` 加接收端身分描述該資源。 |
| 接收端 PyMTLF 處理 PUT／PATCH／DELETE | `api/ml_model_training.py` 的 route 已有 path `subscription_id`、更新 body 與本地結果；在呼叫 `FLClient` 前取得原資源的 `mlCorreId`，完成或拒絕後寫一筆。 | DELETE 完成後資源可能已移除，不能到寫檔時才反查 procedure；查無資源且無法關聯 procedure 的要求不硬塞入實驗 JSONL。 |
| 子節點 PyMTLF 發出 NOTIFY | `FLClient._enqueue_delivery()` 已有通知 body、訂閱資源 ID 與 `mlCorreId`；`_deliver_until_ack()` 知道本次邏輯交付是否獲得 204／明確拒絕。在開始該邏輯交付時取 `startedAt`，獲得終局結果時寫一筆。 | 不以 Go 或 HTTP 內部重試各寫一筆來冒充新的 Model Training 操作；失聯節點尚未取得交付結果時，也不能先記 `SUCCESS`。 |
| 父節點 PyMTLF 收到 NOTIFY | `api/ml_model_training.py:receive_training_notification()` 有實際 body；`FLServer.receive_notification()` 以 `notifCorreId` 找到本地 participant、`mlCorreId`、目標 NF 與其資源位置。接收處理開始取 `startedAt`，驗證／處理完成或拒絕後寫一筆。 | 通知 body 本身未必帶 `mlCorreId` 或正式 `subscriptionId`；應使用已存在的本地關聯，不能從 `notifUri` 猜來源。Go 在交給此 API 前拒絕的要求不虛構「PyMTLF 收到」事件。 |
| Root／Branch 的選擇、邊狀態、拓樸與 round 判定 | Root 的 `FLRootCoordinator` 與 Branch 的 `FLBranchPreparationCoordinator`／`CandidatePool` 具有候選、準備結果與本地 policy；`FLServerEngine.execute_hierarchy_round()` 回傳本輪 selected／successful／failed set 與 accepted 結果。對應決定成立當下寫 node-local event。 | Branch 的 `roundInd` 是自己的 lower-tier round；Root 不假設它等於 upper-tier round。Root 只記自己已收報告、已確認的邊與自己接受的拓樸，不替未接觸的 Leaf 宣告成功。 |
| 模型驗證及完成模型 | 現有 `ExperimentRecorder` 已在 Root 初始／逐輪驗證及完成模型保存路徑被呼叫；本次沿原始數值與產物來源改寫事件格式。 | Official test 若由實驗端離線執行，不為了格式統一而在 PyMTLF 偽造評估事件。 |

`recordedAt` 在本節各通訊列都是**本節點取得本地操作終局結果**的時點；`startedAt` 是本節點將要求交給本機 Go 或從本機 Go 收到要求的時點。兩者不是跨節點網路封包的送達時間。`outcome` 使用 `SUCCESS`（本端操作成功）、`REJECTED`（收到明確拒絕）、`FAILED`（本端失敗或逾時）；`cause` 只寫已取得的 ProblemDetails cause 或本地失敗原因。若同一通知還在重試，不應把其中一次失敗當作整個邏輯通知的終局結果。

接收端 Create 的正式 ID 由接收端 Go 產生，再交給接收端 PyMTLF；發起端 PyMTLF 於建立成功後從自己 Go 回覆的私有 `Location` 取得相同資源 ID。這是 Slice 1 已建立的路徑，無須為紀錄再造 ID 或新增 Go 計數器。`nfInstanceId` 由各 PyMTLF 現有 NWDAF context 提供。若某個實際呼叫路徑尚無可用的欄位，先在該節點既有 PyMTLF 所有者處補紀錄／傳參；只有確認 Go 私有契約真的未傳必要值時，才另列合約變更，不預設本 Slice 需要修改 Go。

### 5.8 修復里程碑與驗收測試

為了讓各 run 的時間分解一致，`instruction` 採 Root 對新 parent／直掛 Leaf 發出第一筆修復訂閱要求的 `startedAt`；`new subscriptions ready` 採 Root **收到所需下層確認並接受修復後 realized topology** 的 `TOPOLOGY_ACCEPTANCE.recordedAt`。E1 現行 Root 在 `_apply_group_preparation()` 驗證 A* 的 topology report、將 group 設為 active 後記錄 replacement ready，可作為此里程碑的實作入口；E2a／E2b 後續由第 3 項提供對應的 Root 接受點。成功 Create、A* 自行報告 ready 或單一新 edge confirmed，都不足以單獨標為整體 ready。`first accepted contribution` 是其後第一個 `ROUND_AGGREGATION`：`accepted=true` 且 successful direct-child set 含修復後的參與者。若某里程碑未達，不填虛構時間。

一個 E1／E2a／E2b run 算作「完成重配置」，至少要在同一 `mlCorreId` 下看到：Root 接受修復後 realized topology、應成立的新邊具有正式 `subscriptionId` 且舊受影響關係不再被算作有效、B／C 等未受影響關係未重建，以及至少一個後續 accepted Root round 確實包含修復後路徑的貢獻。E1 的修復後參與者是 A*；E2a 為直接 A1／A2 且兩條新邊都須確認；E2b 為直接 A1，A2 未形成的事實須照實呈現。這是**離線判讀條件**，不是 PyMTLF 新增的運行期成功開關。模型 accuracy 是否達到 recovery 門檻另行判定，兩者不可混用。

| 驗收測試 | 必須核對的證據 |
| --- | --- |
| Root→Branch→Leaf 的 preparation | 發起端與接收端各自有一筆 CREATE 結果；相同接收端 `nfInstanceId`＋正式 `subscriptionId` 可對照；`x-flTopology` 逐級不同，`mlCorreId` 相同；Create 成功不提前產生 `EDGE_CONFIRMED`。 |
| 更新、刪除及通知 | PUT／PATCH 記實際 `roundInd`／模型參照或修復 tree；DELETE 記正式 ID；NOTIFY 的發出／接收各保留 `notifCorreId`、實際 report／模型欄位與本端結果；同一邏輯通知的內部重試不重複計數。 |
| 拒絕、逾時與識別資訊不足 | 有效 procedure 的拒絕／失敗留下真實 `outcome`／可取得的 `cause`；建立未成功時不編造 `subscriptionId`；接收端未知來源時不填 `sourceNfInstanceId`；無法歸屬 procedure 的早期解析錯誤只留一般日誌。 |
| E0／E1 的拓樸與 round | E0 完成初始拓樸後沒有重建事件；E1 有 A→A* 的新邊、原 Root→B／C ID 延續、Root 接受修復及其後首次含 A* 的 accepted round；Root、Branch 各記自己的 local round，不跨層混用。 |
| 模型結果與失敗 run | 初始及每個 accepted Root round 的 validation accuracy／loss 可對回 `roundInd`；final model 事件能找到產物；未恢復 run 已有的 JSONL 不因缺 final model 而被刪除；模擬紀錄寫入失敗時，該 run 不被視為證據完整。 |
| 後續 E2a／E2b 整合 | 待第 3 項功能完成後，驗證 Root→Leaf 收／發證據、mixed-depth round、E2b A2 未形成而 Root 依 policy 接受；未完成前不宣稱本 Slice 已證明這兩種執行能力。 |

本 Slice 的程式驗證優先用 PyMTLF recorder、API 與協調器的 focused tests；涉及接收端正式 ID 及跨 Go 私有路徑的部分，沿用 Slice 1 既有合約測試並補一條端到端的 Root→Branch→Leaf 紀錄核對。正式五 seed、故障注入與跨 VM 收集仍由實驗執行端處理，不是本 Slice 的單元測試通過條件。

## 6. 紀錄與分析的界線

同一次訂閱可同時留下發起端與接收端的紀錄，因為它們證明不同的事；計算操作次數時只採發起端，不能把兩端加總。接收端若沒有可確認的發起者 `nfInstanceId`，就記自己確實收到的內容和本端 `subscriptionId`，不從通知 URI 猜填對方身分。建立失敗且未取得正式 `subscriptionId` 時，保留各端已知的嘗試及結果；本計畫不為配對失敗嘗試另造 ID。

每個 run 的執行完成或失敗由實驗執行端保存，不能因缺少 final model 或 final test 紀錄就把未完成的 run 排除；這不是要在 PyMTLF 新增 run 結束事件。啟用實驗紀錄後若 JSONL 寫入失敗，該 run 不得視為證據完整的成功實驗；已寫入的紀錄仍應保留供追查。是否達到論文定義的 recovery 則在事後對照 E0 判定。E2b 的 A2 不可用條件由實驗設定與故障注入紀錄交代，節點紀錄只呈現實際嘗試、回報和已形成的關係。E2a 的 mixed-depth 參與由逐輪結果與聚合判定核對；E2b 的 class-wise model effect 若要呈現，可於訓練後以完成模型離線評估，不另增訓練期間事件。

PyMTLF 可觀察到的訂閱操作次數，可在實驗後從原始紀錄計算；它不等同精確的跨 NWDAF SBI HTTP 次數或 Go 內部重試次數。若論文要後兩者，須另定量測來源。老師提及的 topology version 也不是目前 `x-flTopology`／`x-flTopologyReport` 候選 schema 的欄位；目前先保存不同時點的意圖、回報、已形成關係與 Root 接受決定，不虛構 version。

實驗後再依同 seed 的 E0 配對計算五 seed 平均與 95% CI、accuracy AUC、endpoint 差值、各段修復時間及 rounds-to-recovery。Recovery 依實驗文件所定義的 E0 95% CI 且連續兩個 accepted rounds 判定；它不是訓練期間的狀態欄位。未恢復的 run 不從分析中刪除。跨節點延遲要由實驗環境校時或交代時鐘誤差，不能只靠各節點時間戳便宣稱精確。

## 7. 後續在同一計畫補齊

本文件已固定本批事件名稱、`message` 摘錄、三種角色的例子、本地寫入邊界及驗收測試。進入程式修改時仍須逐一核對每個實際呼叫路徑的可取得值，特別是拒絕／逾時、接收端通知關聯與 Root 的 topology acceptance 寫入點；不因表格列了欄位就假定程式已傳到該點。第 3 項尚未實作的 E2a／E2b Root direct-Leaf 接受與 mixed-depth 訓練不是本 Slice 的完成條件。此刻不宣稱 Slice 2 已完成或通過跨節點驗證。
