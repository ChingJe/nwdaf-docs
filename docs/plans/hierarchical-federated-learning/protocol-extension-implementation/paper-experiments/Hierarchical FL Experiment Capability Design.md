# Hierarchical FL 論文實驗能力設計討論

日期：2026-09-20

狀態：Slice 1 訂閱識別的本地實作與審查已確認；第 2–4 項工作邊界及 E2a／E2b 的盡力重掛判斷已確認；其餘詳細設計與實作仍待進行，五 seed 實驗尚未執行。

本文承接 [E0–E2b 實驗情境與 Testbed 對照](./Hierarchical%20FL%20E0-E2b%20Experiments%20and%20Testbed%20Context.md)及[後續實驗能力與證據盤點](./Hierarchical%20FL%20Experiment%20Capability%20and%20Evidence%20Inventory.md)，集中維護接下來的設計討論。前兩份文件分別記錄實驗需求與現況缺口；後續對以下議題形成的設計決定，優先更新本文，不把尚未決定的方案寫成既有實作或已完成實驗。

## 後續協定基準與論文附件界線

依後續會議決定，第 2 項起沿用[候選 OpenAPI](../../../../design/hierarchical-federated-learning/candidate_openapi.yaml)與[欄位語意](../../../../design/hierarchical-federated-learning/candidate_openapi_schema.md)作為實作基準：subscription／PATCH 使用 `flTopology`，notification 使用 `flTopologyReport`，並保留 recursive `children`、node-level `policy`、`strategy`、`reportAfter` 及 status 語意。候選欄位已移除 `x-` 前綴，但 Go／PyMTLF 尚未同步；這是專案 extension，不是已採納的 3GPP 欄位。後續新增能力仍須逐項確認，不因更新 schema 就宣稱已實作。

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

## 第 2–4 項的實作拆分

Slice 1 已處理訂閱資源識別；下列各項只描述下一步的實作範圍與驗收重點，**尚未**宣稱詳細資料流或測試計畫已定案。共同基準是候選 `flTopology`／`flTopologyReport` 的遞迴語意、目前可運作的 Root–Branch–Leaf 訓練與 A→A* replacement；Go／PyMTLF 同步改名仍待實作。不遷移至論文附錄 B 的其他設計，也不新增 `topologyVersion`。`requested`、`realized`、`accepted` 是不同時點的拓樸語意，不拆成獨立功能。第 2 項補原始證據；第 3 項以同一實作項目完成 E2a／E2b 的盡力直接重掛與混合深度訓練；第 4 項處理配對輸入與實驗後分析交接。

| 項目 | 主要支持的實驗 | 責任與直接產出 |
| --- | --- | --- |
| 2. 逐節點協定與拓樸證據 | E0–E2b | PyMTLF 記錄訂閱操作、形成結果與接受決定，不在執行時維護呼叫次數 |
| 3. E2a／E2b 直接重掛與持續訓練 | E2a、E2b | Root PyMTLF 盡力建立 Root→Leaf 新關係、依 Root 訓練門檻判斷能否繼續，並讓已接回的直接 Leaf 與 B／C 在同輪正確訓練與聚合 |
| 4. 配對實驗輸入與原始資料交接 | E0–E2b | 確認 PyMTLF 所需的設定與輸出；testbed 負責五 seed 排程、故障注入與原始資料收集，交接後續離線分析 |

## 2. 逐節點協定與拓樸證據

**現況。** 各節點可寫入以 `mlCorreId` 分目錄的 `observations.jsonl`；目前有 Root 的模型評估、round outcome、Branch failure／replacement ready 與 final model，Branch／Leaf 的模型評估則取決於本地 validation 設定。現有結構化紀錄尚不足以逐條證明訂閱建立、拓樸形成和修復。Slice 1 已讓發起端 PyMTLF 取得由接收端 Go 公布的正式訂閱資源 ID，但尚未把它寫成完整實驗事件。

**要實作。** 擴充各節點 PyMTLF 的原始事件紀錄，按實際行為記錄：向哪個 NF 發起／收到訂閱 Create、更新、刪除及結果；收到的 topology instruction、回報的 topology report；preparation／參與確認後形成或失去的 direct edge；Root 依實際 report 作出的接受／拒絕決定；每輪選入、成功、失敗的 direct participants 與必要的下層結果。事件需有 UTC 時間、本節點 `nfInstanceId`、可確認的對端 `nfInstanceId`、方向、`mlCorreId`、可用時的 `notifCorreId`、操作結果；已取得正式 ID 的訂閱以接收端 NF 加 `subscriptionResourceId` 識別。保存足以解釋訓練任務與決策的訂閱摘要，包括適用的 `mLEventSubscs`／`modelInterInfo`、`mLModelTrainInfos` 與 `flTopology`／`flTopologyReport` 的 children、policy、status；不複製模型、憑證或整份 payload。發出 Create、收到成功回覆、確認 edge、Root 接受拓樸必須是可區分的事實；失敗嘗試不可記成 confirmed edge。Root 原有 validation／round 紀錄保留，新增事件與它們使用同一 `mlCorreId`。本項只保存逐筆事實，不新增執行時呼叫計數器；第 3 項的各個決策點沿用本項的紀錄方式。

**紀錄邊界。** 發起端 PyMTLF 已能從自己的 Go 私有回應取得成功建立的正式訂閱資源 ID；本項在此記錄其發起的操作及所見結果。接收端 PyMTLF 的私有 Create 目前沒有直接提供可驗證的發起端 `nfInstanceId`；不憑接收端單筆紀錄推定對端身分，可由發起端紀錄與訂閱資源關係重建。此項不新增 Go 實驗紀錄、精確 wire-level SBI 計數或內部重試統計，也不把 PyMTLF 的操作紀錄宣稱為逐筆 SBI HTTP 傳輸證據。

**驗收。** E0 可重建初次意圖、已確認的 A／B／C 與六條下層邊、Root 接受時的 realized topology；E1 可對照 A→A1/A2 與 A*→A1/A2 的新舊資源 ID、保留 Root→B/C 原 ID，並區分故障偵測、修復指令、新 edge ready 與首次 accepted A* contribution。E2a／E2b 使用同一紀錄方式，A2 未確認或失敗不算已形成 edge；其實際修復與訓練行為由第 3 項實作。拓樸前後比較由原始事件和訂閱關係重建，不假造 wire `topologyVersion`。此項的詳細計畫須以實際 producer／consumer 路徑確認每個紀錄點及可取得的欄位，不擴成 Go SBI 計數工作。

## 3. E2a／E2b 直接重掛、拓樸判斷與持續訓練

**現況。** Root 的靜態拓樸是 `branch_groups`；A 失效後，既有流程只會在同一 group 中尋找替代 Branch。雖然 group 已列出 A1／A2，這是原 Branch 的下層候選，不等於 Root 已獲授權直接訂閱它們。現有 Root 的 active group／readiness 也只計算 active Branch；每輪對所有直接參與者要求 `HIERARCHY_AGGREGATE`，以 Branch report 的下層名單驗證結果，Branch→Leaf round 則要求 `TRAINING`。因此只建立 Root→Leaf 訂閱仍不足以讓 E2a／E2b 繼續訓練。Root 初始 admission 僅接受 `complete_required`；Branch group 的 policy 是 Branch 管理下層 Leaves 的條件，不能直接拿來判斷 Root 的修復結果。

**關係形成。** 在 Root 的本地拓樸設定明示 A 失效且沒有 A* 時可將哪些存活 Leaves 接回 Root；不能單靠看見 `group.leaves` 就自行 reparent。Root 依既有 Go→peer Model Training Create 路徑，對 A1／A2 下發直接 Leaf 的 `flTopology` 指令，沿用同一 `mlCorreId`，取得各自的新 `subscriptionResourceId` 並完成 preparation／參與確認；再由 `flTopologyReport` 和實際邊更新 realized topology。原 A 的訂閱清理不是建立新邊的前置條件；Root→B/C 與其下層訂閱保持原狀。這是 Root PyMTLF 的選擇、訂閱與狀態調整，若 Go 既有私有 Create／peer SBI 路徑足以傳遞同一契約，不新增對外操作。

**盡力重掛與訓練門檻。** Root 對 A1／A2 都嘗試建立直接關係，不因本層 `minAvailableNodes` 已滿足就停止嘗試；各自的成功、失敗或逾時結果決定哪些 direct edges 進入 realized topology。不另設要求至少接回一個 Leaf 的修復門檻，也不新增修復成功／失敗標記；兩個、一個或零個 Leaf 接回，從訂閱、確認關係與拓樸紀錄事後判讀。Root 能否繼續訓練仍看本層既有 `minAvailableNodes`、`minTrainNodes` 與當輪完成條件；A1／A2 都未接回時，若 B／C 達標仍可降級訓練，但不能把未形成的關係記成已接回。B／C 的 Branch-local policy 不變；現有 Root 門檻只計 active Branch groups，Slice 3 須使它正確作用於混合深度的直接參與者。

**訓練執行。** Root 的 round 選擇、等待、結果驗證與 aggregation 按每個 direct child 的實際角色處理：B/C 回 `HIERARCHY_AGGREGATE`，直接接回的 A1／A2 回 `TRAINING`；兩種 artifact 保持原本各自的身分、`mlCorreId`、`roundInd`、training scope 驗證，並依其實際 `training_sample_count` 混合加權。Root 發布的 global model 仍由 ADRF 提供，該輪實際選入的 B/C 及 A1／A2 必須能依既有權限取得模型；Branch 對自己的 Leaves 的模型下發路徑不變。Root 在 round outcome 記錄真實 direct participant set，不能把直接 Leaf 偽裝成 Branch 或把它的結果算兩次。

**驗收。** E2a 中 A 消失且沒有替代 Branch 時，Root 分別與 A1、A2 建立新訂閱，能回報 `Root→A1/A2`、`Root→B→B1/B2`、`Root→C→C1/C2` 的 realized topology；故障前後 `mlCorreId` 不變，B/C 的既有訂閱 ID 不變。兩條 direct edges 確認後，Root 記錄實際形成的拓樸，並可在同一 accepted Root round 聚合 A1、A2、B、C 的有效結果；不另記完整修復的接受標記。

E2b 的 instruction 可仍列 A1／A2，但 A2 不可用或建立失敗時不算 confirmed edge；realized topology 只含 A1。Root 依原有訓練門檻判斷能否繼續，後續 round 可聚合 A1、B、C；「部分接回」由實際形成的關係判讀，不另設線上接受條件。`minAvailableNodes` 檢查當時可用的 Root 直接子節點；`minTrainNodes`、`fractionTrain` 決定該輪選入數，`minCompletionRate` 仍依選入者計算完成比例。A2 的資料不再被算入該輪樣本權重。兩種情境皆須保留結果型別、身分、樣本數不合法時的既有失敗處理，且 B/C 路徑不得退化。

本項的詳細計畫須對照現有 Root group／candidate pool，定案 direct-repair 的本地設定與啟動時點；逐段檢查 Root round dispatch、ADRF 儲存／允許清單、Go 模型取用契約、Leaf 模型取得、artifact 驗證與 round cleanup。這是 Root 設定及狀態的擴充，不預設新的 wire 欄位，也不能只修改 aggregation 函式。

## 4. 配對實驗輸入與原始資料交接

**PyMTLF 所需能力。** 確認 E0–E2b 的本地設定可逐 run 指定相同 seed 對應的資料分割、初始模型 artifact、local epochs、FedProx 與 aggregation 設定、總 accepted rounds，並在每個節點原始資料中對回 `mlCorreId`。Root 既有 initial／逐 accepted round validation、final model 和 round outcome 繼續使用；第 2 項補足 protocol 時間點，失敗或未恢復的 run 也應保留截至停止時的原始事件。若現有設定與輸出已足夠，這一項只需驗證及文件化，不為五 seed 增加新的 runtime 排程器。

**實驗執行端的界線。** Testbed controller 負責五 seed 配對、MNIST 第 12 輪／CIFAR-10 第 20 輪後的故障注入、A2 是否停止、各節點檔案收集與 run metadata；離線分析負責 95% CI、AUC、paired E0 差值、recovery 判定、失敗 run 統計，以及依第 2 項原始紀錄事後計算訂閱資源 Create／PUT／PATCH／DELETE 的發起次數。若分析通知，須與訂閱資源操作分開統計；發起與回覆、發送與接收不得重複計數。這是 PyMTLF 可觀察的操作次數，不宣稱精確的 SBI wire-level HTTP 呼叫數。這些都不是 PyMTLF 的線上決策或本項新增的計數功能。Controller 的故障注入時間與第 2 項的各節點 UTC 事件對齊後，才能計算 failure→detection→instruction→new edges ready→first accepted contribution；跨機器時鐘同步／誤差是 testbed 交接條件，不應由 PyMTLF 虛構時間點。

**驗收。** 對同一 workload／seed 的四種情境，可以以 run metadata 和 `mlCorreId` 對齊各節點原始事件與 Root 模型曲線；設定快照保留 Root 訓練門檻與本地重掛策略，實際接回數由各 run 的關係紀錄確認，不能把 E2a／E2b 寫成只差故障名稱。未恢復的 run 仍可供離線分析；舊的單次 MNIST／CIFAR-10 配對不計入新的五 seed 結果。第 2、3 項通過本地測試後，仍需另由 testbed 驗證實際跨 NF 行為，不能以本地測試替代正式實驗。
