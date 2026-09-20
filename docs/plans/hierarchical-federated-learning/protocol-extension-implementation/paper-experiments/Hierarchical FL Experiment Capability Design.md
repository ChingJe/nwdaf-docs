# Hierarchical FL 論文實驗能力設計討論

日期：2026-09-20

狀態：Slice 1 訂閱識別的本地實作與審查已確認；第 2–6 項依 E0–E2b 的工作拆分已確認，詳細計畫與實作仍待進行；五 seed 實驗尚未執行。

本文承接 [E0–E2b 實驗情境與 Testbed 對照](./Hierarchical%20FL%20E0-E2b%20Experiments%20and%20Testbed%20Context.md)及[後續實驗能力與證據盤點](./Hierarchical%20FL%20Experiment%20Capability%20and%20Evidence%20Inventory.md)，集中維護接下來的設計討論。前兩份文件分別記錄實驗需求與現況缺口；後續對以下議題形成的設計決定，優先更新本文，不把尚未決定的方案寫成既有實作或已完成實驗。

## 後續協定基準與論文附件界線

依後續會議決定，第 2 項起沿用[既有候選 OpenAPI](../../../../design/hierarchical-federated-learning/candidate_openapi.yaml)與[欄位語意](../../../../design/hierarchical-federated-learning/candidate_openapi_schema.md)作為實作基準：subscription／PATCH 使用 `x-flTopology`，notification 使用 `x-flTopologyReport`，並保留 recursive `children`、node-level `policy`、`strategy`、`reportAfter` 及 status 語意。這是專案候選 extension，不是已採納的 3GPP 欄位；後續新增能力也須逐項確認，不因沿用 schema 就宣稱已實作。

論文附錄 B 的 `flTopology`／`flTopologyReport`、`topologyVersion`、`candidates[].childInstruction`、`reparentInstruction` 與 `directEdges[].edgeState` 保留為[兩版對照](./Hierarchical%20FL%20E1%20Wire%20Schema%20Flow%20Comparison.md)及論文修訂參考，**不列為本批 wire migration 目標**。若論文仍要求附件特有的欄位或證據，須另行協調論文敘述或明確決策新增機制；不能把兩版欄位視為等價，亦不預先增加相容雙格式。

## 1. 訂閱資源識別與生命週期

本節保存 Slice 1 開工前的 **ML Model Training 訂閱**設計與當時的資料流描述；目前本地實作結果見[Slice 1 詳細計畫](./slices/Slice%201%20ML%20Model%20Training%20Subscription%20Resource%20Identity%20and%20Lifecycle%20Detailed%20Plan.md)，不把下文的「現行程式」當成最新狀態。以下以 `callbackRouteId` 指發起端 Go 產生的本地 callback 路由識別碼，以 `subscriptionResourceId` 指接收端 Go 在 SBI `Location` 公布的訂閱資源識別碼；兩者是本文的概念名稱，不是新增的 wire 欄位。原本接收端 Go 與其 PyMTLF 各產生一個 UUID，發起端 Go 另以 `callbackRouteId` 作為路由主鍵並回給發起端 PyMTLF；Slice 1 改以 `subscriptionResourceId` 作為同一訂閱的正式資源 ID。`subscriptionResourceId` 只要求在**接收端 NF 的 ML Model Training 服務內**唯一；本專案接收端可繼續產生 UUIDv4，但不能據此假設不同 NF 回傳的 ID 必然唯一。跨 NF 辨識一條邊時使用「接收端 `nfInstanceId`、服務、`subscriptionResourceId`」；`mlCorreId` 是整個 FL procedure 的關聯 ID，`notifCorreId` 是通知關聯 ID，兩者都不取代訂閱資源 ID。

### 1.1 接收端建立資源

1. 接收端 Go 收到對本 NF 的 Create（外部 SBI 或本地私有入口）後，先產生 `subscriptionResourceId` 並暫存建立中狀態。標準訂閱 body 仍維持原樣；不新增承載此 ID 的 3GPP 欄位。
2. 接收端 Go 呼叫**自己的** PyMTLF 私有 Create 時，在私有 HTTP header `X-NWDAF-Subscription-Id` 傳入 `subscriptionResourceId`。此 header 不轉送給其他 NWDAF。PyMTLF 必須檢查 header 存在、符合本專案使用的 UUIDv4 格式且未占用，再以該值建立及索引訂閱；不再自行產生另一個 backend-local ID。
3. PyMTLF 的私有 `201 Created` 保留訂閱 representation，`Location` 的資源路徑以相同 `subscriptionResourceId` 結尾。Go 檢查狀態、representation 及私有 `Location` 是否指向預期 backend 的該筆訂閱，確認後才啟用路由，並以同一個 `subscriptionResourceId` 作為對外 SBI `Location` 的末段。若不一致，不得把該次 Create 回報為成功，也不能依錯誤 `Location` 的 ID 誤刪其他資源。
4. 後續外部 PUT／PATCH／DELETE 使用對外路徑的 `subscriptionResourceId`；接收端 Go 用同一個 ID 找到本地路由，並將操作送至自己 PyMTLF 的對應資源。接收端不再維護「對外訂閱 ID → backend-local ID」映射。

這是 **Go→PyMTLF 私有契約**的變更，會涉及 Go 的 backend client、接收端 processor／route，以及 PyMTLF 的私有 API 與資源儲存；不是新的公開 SBI 欄位或 resource path。

### 1.2 發起端建立與操作資源

1. 發起端 Go 在向子節點發送 Create 前仍產生暫時的 `callbackRouteId`，用它建立通知 callback URI，並暫存尚未完成的 Create。`callbackRouteId` 不是訂閱資源 ID。
2. 子節點 Go 的 SBI `201 Created` 回傳 `Location`；發起端 Go 保留完整 peer `Location` 作為後續對外操作目標，並從資源路徑取得 `subscriptionResourceId`。此處的 peer 訂閱資源 ID 是對方提供的非空資源字串，不額外要求一定是 UUIDv4；建構私有 URL 時須正確編碼該路徑片段。
3. 發起端 Go 以「目標 `nfInstanceId`、ML Model Training 服務、`subscriptionResourceId`」保存正式 outbound route，並回給自己的 PyMTLF 一個**發起端 Go 的私有 URL**，例如 `/internal/v1/ml-model-training/targets/{targetNfInstanceId}/subscriptions/{subscriptionResourceId}`。這個 URL 同時攜帶目標 NF scope，最後一段是接收端的訂閱資源 ID；它不是子節點的 SBI URL。發起端 PyMTLF 保存該私有 `Location`，後續 PUT／PATCH／DELETE 原樣交給自己的 Go，再由 Go 使用保存的 peer `Location` 轉送。
4. `callbackRouteId →（目標 NF、服務、subscriptionResourceId）` 的 callback 索引在 Create 成功後仍保留，直到該訂閱終止，因為先前下發的通知 URI 仍含 `callbackRouteId`。只有建立中的其他暫存資料可在綁定訂閱資源 ID 後移除。callback 先以 `callbackRouteId` 找到 route，再用原有的 `notifCorreId`、`mlCorreId` 等內容驗證；不得把 `callbackRouteId` 寫成對外 subscription ID。同一個 `subscriptionResourceId` 若由不同目標 NF 回傳，也不得覆蓋彼此的 route。

目前 Go 的私有 PUT／PATCH／DELETE 路由只以單一 `subscriptionId` 查找，因此目標 NF scope 必須與上述私有 URL 及路由儲存一起調整；不能只改 Create 回應的 `Location` 字串。發起端 PyMTLF 已保存並使用該 URL，但其從 URL 末段擷取 ID 的邏輯也需要對照新路徑驗證。

### 1.3 失敗、通知與紀錄界線

- 建立訂閱時，接收端 PyMTLF 明確拒絕私有 Create、回覆內容不符契約，或接收端 Go 無法啟用對應路由，均不得對外回報建立成功。接收端 Go 已知 `subscriptionResourceId`，若 PyMTLF 可能已建立資源，便以此 ID 嘗試清理。發起端若收到明確的 peer Create 拒絕，移除未綁定的 `callbackRouteId`；若已取得有效 peer `Location` 卻無法完成本地綁定，則對該 peer URL 嘗試清理。這些失敗都不能記成 confirmed edge，也不能覆寫其他已建立的 route。
- 建立期間的 callback URI 已含 `callbackRouteId`，但正式 `subscriptionResourceId` 可能尚未回到發起端 Go。現行 Go 路由在 `Creating` 狀態會對早到通知回覆 `503`；接收端 PyMTLF 的通知 outbox 遇到 `5xx` 會重試。待發起端 Go 綁定訂閱資源 ID 並啟用路由後，重試的通知才可被接收。這沿用既有 callback／重試流程，不新增通知緩衝或 schema 欄位；實作驗證須涵蓋「通知早於 Create 回覆，先收到 `503`、後收到 `204`」的雙端時序。
- 已建立訂閱的 PUT／PATCH／DELETE 仍指向同一個 `subscriptionResourceId`；操作失敗時不得記成更新或終止成功，也不得因一次失敗清掉仍需使用的正式 route。E1 的 A 中途失效是另一個階段的事件：Root 依既有等待期限判斷 A 未貢獻，由 B／C 支持降級訓練；Root 建立與 A* 的新訂閱，A* 再向原 Leaves 建立新訂閱。舊 A 關係的清理不作為新 edge 建立的前置條件；B／C 的訂閱不重建。
- 逐節點事件以接收端身分加 `subscriptionResourceId` 識別訂閱，分開記錄 Create 嘗試、建立確認、通知、更新與終止。`callbackRouteId` 只供 callback 路由及排錯，不作為實驗中的 subscription ID。E0／E1 應能以真實訂閱資源 ID 比對 A 與 A* 的新舊父子訂閱，並確認 B／C 的關係未變。

若 Create 回覆遺失而無法取得 peer `subscriptionResourceId`，不得憑 `callbackRouteId` 猜測資源 ID 或宣稱 edge 已建立；保留失敗／結果不明的紀錄即可。跨程序重啟後的殘留資源清理不列為本次 E0–E2b 實驗能力的完成條件。

## 第 2–6 項的實作拆分

Slice 1 已處理訂閱資源識別；下列各項只描述下一步的實作範圍與驗收重點，**尚未**宣稱詳細資料流或測試計畫已定案。共同基準是現有 `x-flTopology`／`x-flTopologyReport`、目前可運作的 Root–Branch–Leaf 訓練與 A→A* replacement；不遷移至論文附錄 B，也不新增 `topologyVersion`。`requested`、`realized`、`accepted` 是不同時點的拓樸語意，不拆成獨立功能。

| 項目 | 主要支持的實驗 | 責任與直接產出 |
| --- | --- | --- |
| 2. 逐節點協定與拓樸證據 | E0–E2b | PyMTLF 記錄操作、形成結果與接受決定；必要的 Go SBI 計數另補最小邊界紀錄 |
| 3. Root 修復時直接接回 Leaf | E2a、E2b | Root PyMTLF 以既有候選 schema 建立 Root→A1／A2 關係，保留 B／C |
| 4. 不等深度的訓練 round | E2a、E2b | Root PyMTLF 同輪處理直接 Leaf 與 Branch aggregate，並正確下發模型 |
| 5. 部分修復與明示的接受條件 | E2b | Root PyMTLF 在 A2 不可用時依設定判斷是否接受 A1，並繼續訓練 |
| 6. 配對實驗輸入與原始資料交接 | E0–E2b | 確認 PyMTLF 所需的設定與輸出；testbed 負責五 seed 排程、故障注入與彙整 |

## 2. 逐節點協定與拓樸證據

**現況。** 各節點可寫入以 `mlCorreId` 分目錄的 `observations.jsonl`；目前有 Root 的模型評估、round outcome、Branch failure／replacement ready 與 final model，Branch／Leaf 的模型評估則取決於本地 validation 設定。現有結構化紀錄尚不足以逐條證明訂閱建立、拓樸形成和修復。Slice 1 已讓發起端 PyMTLF 取得由接收端 Go 公布的正式訂閱資源 ID，但尚未把它寫成完整實驗事件。

**要實作。** 擴充各節點 PyMTLF 的原始事件紀錄，按實際行為記錄：向哪個 NF 發起／收到訂閱 Create、更新、刪除及結果；收到的 topology instruction、回報的 topology report；preparation／參與確認後形成或失去的 direct edge；Root 依實際 report 作出的接受／拒絕決定；每輪選入、成功、失敗的 direct participants 與必要的下層結果。事件需有 UTC 時間、本節點與對端 `nfInstanceId`、方向、`mlCorreId`、可用時的 `notifCorreId`、操作結果；已取得正式 ID 的訂閱以接收端 NF 加 `subscriptionResourceId` 識別。保存足以解釋訓練任務與決策的訂閱摘要，包括適用的 `mLEventSubscs`／`modelInterInfo`、`mLModelTrainInfos` 與 `x-flTopology`／`x-flTopologyReport` 的 children、policy、status；不複製模型、憑證或整份 payload。發出 Create、收到成功回覆、確認 edge、Root 接受拓樸必須是可區分的事實；失敗嘗試不可記成 confirmed edge。Root 原有 validation／round 紀錄保留，新增事件與它們使用同一 `mlCorreId`。

**Go 邊界。** Go 是實際 SBI 操作與 `Location` 的權威來源；PyMTLF 先記錄其私有呼叫能觀察到的結果。若論文需要精確的跨 NF API-call／message 數量或 Go 內部重試次數，須在 Go 的實際 SBI 收送邊界補可蒐集的最小計數紀錄，包含操作、方向、對端、結果及時間，再由離線分析以發送端的實際跨 NF 呼叫計數，避免收送兩端重複計入；不能把 PyMTLF 發起一次私有請求直接算成一次成功的 SBI 訊息，也不預設 Go 寫第二份 `observations.jsonl`。

**驗收。** E0 可重建初次意圖、已確認的 A／B／C 與六條下層邊、Root 接受時的 realized topology；E1 可對照 A→A1/A2 與 A*→A1/A2 的新舊資源 ID、保留 Root→B/C 原 ID，並區分故障偵測、修復指令、新 edge ready 與首次 accepted A* contribution。E2a／E2b 使用同一紀錄方式，A2 未確認或失敗不算已形成 edge。拓樸前後比較由原始事件和訂閱關係重建，不假造 wire `topologyVersion`。此項的詳細計畫須以實際 producer／consumer 路徑確認每個事件的產生點及 Go 是否必須增加欄位或紀錄。

## 3. Root 修復時直接接回 Leaf

**現況。** Root 的靜態拓樸是 `branch_groups`；A 失效後，既有流程只會在同一 group 中尋找替代 Branch。雖然 group 已列出 A1／A2，這是原 Branch 的下層候選，不等於 Root 已獲授權直接訂閱它們。現有 Root 的 active group／readiness 也只計算 active Branch。

**要實作。** 在 Root 的本地拓樸設定明示 A 失效且沒有 A* 時可將哪些存活 Leaves 接回 Root，以及該修復所適用的 Root direct-child 條件；不能單靠看見 `group.leaves` 就自行 reparent。Root 依既有 Go→peer Model Training Create 路徑，對 A1／A2 下發直接 Leaf 的 `x-flTopology` 指令，沿用同一 `mlCorreId`，取得各自的新 `subscriptionResourceId` 並完成 preparation／參與確認；再由 `x-flTopologyReport` 和實際邊更新 realized topology。原 A 的訂閱清理不是建立新邊的前置條件；Root→B/C 與其下層訂閱保持原狀。這是 Root PyMTLF 的選擇、訂閱與狀態調整，若 Go 既有私有 Create／peer SBI 路徑足以傳遞同一契約，不新增對外操作。

**驗收。** E2a 中 A 消失且沒有替代 Branch 時，Root 分別與 A1、A2 建立新訂閱，能回報 `Root→A1/A2`、`Root→B→B1/B2`、`Root→C→C1/C2` 的 realized topology；故障前後 `mlCorreId` 不變，B/C 的既有訂閱 ID 不變。E2b 中 A2 不可用時，建立失敗不應被當成 confirmed edge；是否接受只剩 A1 的拓樸由第 5 項處理。正式設定欄位及失敗後何時啟動此路徑，需在本項詳細計畫中對照現有 Root group／candidate pool 定案。

## 4. 不等深度的訓練 round

**現況。** Root 每輪只選 active Branch，對所有直接參與者要求 `HIERARCHY_AGGREGATE`，並以 Branch report 的下層名單驗證結果；Branch→Leaf round 則要求 `TRAINING`。因此只建立 Root→Leaf 訂閱，還不足以讓 E2a／E2b 繼續訓練。

**要實作。** Root 的 round 選擇、等待、結果驗證與 aggregation 按每個 direct child 的實際角色處理：B/C 回 `HIERARCHY_AGGREGATE`，直接接回的 A1／A2 回 `TRAINING`；兩種 artifact 保持原本各自的身分、`mlCorreId`、`roundInd`、training scope 驗證，並依其實際 `training_sample_count` 混合加權。Root 發布的 global model 仍由 ADRF 提供，該輪實際選入的 B/C 及 A1／A2 必須能依既有權限取得模型；Branch 對自己的 Leaves 的模型下發路徑不變。Root 在 round outcome 記錄真實 direct participant set，不能把直接 Leaf 偽裝成 Branch 或把它的結果算兩次。

**驗收。** E2a 在同一 accepted Root round 可以同時聚合 A1、A2、B、C 的有效結果；E2b 可聚合 A1、B、C。結果型別錯誤、身分或樣本數欄位不合法時仍沿既有失敗處理，B/C 的原路徑不得退化。詳細計畫必須逐段對照 Root 的 round dispatch、ADRF 儲存／允許清單、Go 模型取用契約、Leaf 模型取得、artifact 驗證與 round cleanup，不能只改 aggregation 函式。

## 5. 部分修復與明示的接受條件

**現況。** Root 靜態設定的初始 admission 僅接受 `complete_required`，且 Root readiness 以 Branch group 數量判斷；Branch group 的 policy 是 Branch 管理下層 Leaves 的條件。E2b 所需的「Root 嘗試 A1／A2，但只確認 A1 仍接受修復」不能誤用原 Branch 的兩個 Leaves 門檻，也不能把 A2 的消失視為無條件成功。

**要實作。** 在 Root 本地設定明確表達 direct-repair cohort 的最低可用數及 Root 這一層每輪的選擇／完成條件，與 B/C 原有 Branch-local policy 分開。Root 嘗試被授權的 A1／A2，保留 A2 失敗或不可用的結果，再以實際 confirmed direct edges 和對應 policy 判斷接受或拒絕；只有接受後才把修復結果作為新的 accepted realized topology。修復期間未受影響的 B/C 仍可依既有 policy 支持降級 round。E2a 的兩 Leaf 完整接回與 E2b 的一 Leaf 部分接回須使用明示、可記錄的條件；原 E0／E1 的三 Branch 設定不默默改寫。

**驗收。** E2b 的 instruction 可以仍列 A1／A2，realized topology 只含 A1，Root 能記下所用 policy 和接受／拒絕結果；接受時後續訓練由 A1、B、C 繼續，拒絕時不宣稱恢復。Root 的 `minAvailableNodes`、`minTrainNodes`、`fractionTrain`、`minCompletionRate` 分母都以當時的 direct cohort 說明並測試；A2 的訓練資料不再被算入該輪樣本權重。詳細計畫須決定既有本地設定如何表達 direct-repair 條件及其啟用時點，這是設定／Root 狀態的擴充，不預設新的 wire 欄位。

## 6. 配對實驗輸入與原始資料交接

**PyMTLF 所需能力。** 確認 E0–E2b 的本地設定可逐 run 指定相同 seed 對應的資料分割、初始模型 artifact、local epochs、FedProx 與 aggregation 設定、總 accepted rounds，並在每個節點原始資料中對回 `mlCorreId`。Root 既有 initial／逐 accepted round validation、final model 和 round outcome 繼續使用；第 2 項補足 protocol 時間點，失敗或未恢復的 run 也應保留截至停止時的原始事件。若現有設定與輸出已足夠，這一項只需驗證及文件化，不為五 seed 增加新的 runtime 排程器。

**實驗執行端的界線。** Testbed controller 負責五 seed 配對、MNIST 第 12 輪／CIFAR-10 第 20 輪後的故障注入、A2 是否停止、各節點檔案收集與 run metadata；離線分析負責 95% CI、AUC、paired E0 差值、recovery 判定及失敗 run 的統計。這些不是 PyMTLF 的線上決策。Controller 的故障注入時間與第 2 項的各節點 UTC 事件對齊後，才能計算 failure→detection→instruction→new edges ready→first accepted contribution；跨機器時鐘同步／誤差是 testbed 交接條件，不應由 PyMTLF 虛構時間點。

**驗收。** 對同一 workload／seed 的四種情境，可以以 run metadata 和 `mlCorreId` 對齊各節點原始事件與 Root 模型曲線；設定快照明示 E2a／E2b 若採不同接受條件，不能把它寫成與 E0／E1 僅差故障類型。未恢復的 run 仍可供離線分析；舊的單次 MNIST／CIFAR-10 配對不計入新的五 seed 結果。第 2–5 項通過本地測試後，仍需另由 testbed 驗證實際跨 NF 行為，不能以本地測試替代正式實驗。
