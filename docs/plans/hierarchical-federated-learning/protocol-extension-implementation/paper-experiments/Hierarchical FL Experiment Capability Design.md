# Hierarchical FL 論文實驗能力設計討論

日期：2026-09-20

狀態：新版論文附錄 B 已選為後續實作目標；本文件修訂已確認，其他細節仍待逐項確認；尚未開始本批功能實作或 E0–E2b 五 seed 實驗。

本文承接 [E0–E2b 實驗情境與 Testbed 對照](./Hierarchical%20FL%20E0-E2b%20Experiments%20and%20Testbed%20Context.md)及[後續實驗能力與證據盤點](./Hierarchical%20FL%20Experiment%20Capability%20and%20Evidence%20Inventory.md)，集中維護接下來的設計討論。前兩份文件分別記錄實驗需求與現況缺口；後續對以下議題形成的設計決定，優先更新本文，不把尚未決定的方案寫成既有實作或已完成實驗。

## 已選定的協定目標與遷移界線

本批工作以新版論文附錄 B 的候選 Stage-3 設計為目標：subscription／PATCH 傳遞 `flTopology`，notification 傳遞 `flTopologyReport`；instruction 與 report 都帶 `topologyVersion`，以 `candidates[].childInstruction` 逐級下發，以 `reparentInstruction` 指示替代節點接手，以 `directEdges[].edgeState` 回報直接子關係。這是**待實作的論文提案**，不是現行 3GPP schema 或已部署的 wire format。現行 `x-flTopology`／`x-flTopologyReport` 對照留在 [E1 兩版範例](./Hierarchical%20FL%20E1%20Wire%20Schema%20Flow%20Comparison.md)，不作為新功能的目標契約。

已選定目標不等於附錄 B 所有細節都已定案。附錄尚未完整定義原有 node `policy`、`strategy`、`reportAfter` 與 per-round 門檻的去向，也未分配正式 `suppFeats` feature number 或完成 presence／cardinality 規則。下階段須逐條核對現行資料產生端、Go／PyMTLF 私有邊界、對外 SBI、儲存狀態、通知與修復路徑；避免只換 JSON property 名稱，或把現有訓練設定無聲移除。實驗性舊欄位不預設永久相容讀取；若有仍在使用的接收端，再以實際需求判斷。

## 1. 訂閱資源識別與生命週期

本節細化的是 **ML Model Training 訂閱**的目標設計，尚非現行實作。以下以 `callbackRouteId` 指發起端 Go 產生的本地 callback 路由識別碼，以 `subscriptionResourceId` 指接收端 Go 在 SBI `Location` 公布的訂閱資源識別碼；兩者是本文的概念名稱，不是新增的 wire 欄位。現行程式讓接收端 Go 與其 PyMTLF 各產生一個 UUID，發起端 Go 另以 `callbackRouteId` 作為路由主鍵並回給發起端 PyMTLF；改動後，以 `subscriptionResourceId` 作為同一訂閱的正式資源 ID。`subscriptionResourceId` 只要求在**接收端 NF 的 ML Model Training 服務內**唯一；本專案接收端可繼續產生 UUIDv4，但不能據此假設不同 NF 回傳的 ID 必然唯一。跨 NF 辨識一條邊時使用「接收端 `nfInstanceId`、服務、`subscriptionResourceId`」；`mlCorreId` 是整個 FL procedure 的關聯 ID，`notifCorreId` 是通知關聯 ID，兩者都不取代訂閱資源 ID。

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

## 2. 逐節點原始事件紀錄

已確認的方向：各節點 PyMTLF 的本地 `observations.jsonl` 是實驗事件的主要紀錄；Go 提供真實 SBI 資源與操作結果。只有 PyMTLF 無法觀察、且論文確實需要的 Go 內部呼叫或失敗，才另討論最小 Go telemetry。不重複保存完整模型、權杖或無關 payload。

待討論：

- 哪些訂閱、訓練與故障事件必須分開記錄；特別區分「發出 Create」、「收到成功回覆」、「確認參與及形成 edge」和「Root 接受拓樸」。
- 每種事件的必要欄位與產生點：本節點及對端 `nfInstanceId`、方向、`subscriptionResourceId` 及其 owner、`mlCorreId`、時間、操作結果、原因，以及目標 `flTopology`／`flTopologyReport` 中足以解釋決策的版本、candidate、reparent instruction 與 direct-edge 摘要。
- 重試、雙端觀測、跨程序時鐘與 run 資料夾的關聯方式；哪些數據只需保存原始事件，交由離線分析計算。

## 3. 拓樸狀態與接受決定

已確認的語意：Root 下發的 topology intention／instruction 是建立意圖；realized topology 只包含已確認的 parent–child 關係；Root 接受後才稱 accepted realized topology。候選者、成功訂閱、confirmed edge 與單輪成功貢獻者不可混為同一集合。

待討論：

- 初次形成及局部修復時，由哪個節點、在哪個時點記錄 instruction、realized report、接受或拒絕結果與前後快照。
- 如何讓新版論文候選契約中的 `topologyVersion` 貫穿 Root instruction、各層 report 與 Root 接受決定，並處理過期或亂序資料。版本不同於 `roundInd`；Root 的接受決定可作為內部狀態與實驗證據，附錄 B 尚未定義專用接受通知。
- 如何證明未受影響的 Root→B／C 等關係保持不變，以及未確認或失敗候選如何與 realized edges 一起呈現而不被誤算。

## 4. E2a／E2b：不同深度與部分修復

現況缺口：新版論文的 recursive instruction／report 可描述不等深度的樹，但目前 Root PyMTLF 的 static `branch_groups` 與 round 執行仍以直接 Branch 和 `HIERARCHY_AGGREGATE` 為主，不能據此宣稱 E2a／E2b 已可執行。

待討論：

- A 消失後，Root 如何向 A1／A2 建立 direct subscriptions，同時保留 B／C 與其下層關係。
- Root 如何在同一輪接收直接 Leaf 的 `TRAINING` 結果和 Branch 的 aggregate，並依各結果實際代表的樣本數做 aggregation。
- 新的直接 Leaf 如何取得下一輪 global model；ADRF 的 consumer 清單與修復時序如何對齊。
- E2b 只接回 A1 時，Root 如何根據明示的 direct-child policy 接受或拒絕部分 realized topology，並保存 A2 不可用及資料覆蓋變動的證據。

## 5. 實驗設定與證據交接

已確認的界線：PyMTLF 提供可信的逐節點原始事件及模型觀測；故障注入與五 seed 排程由 testbed 執行端處理；95% CI、AUC、recovery 判定及圖表由離線分析處理。既有單次 MNIST／CIFAR-10 配對不得充當新的五 seed 結果。

待討論：

- 每個 scenario 的 workload、seed、資料分割、初始模型、policy、觸發輪次及程式版本，如何與各節點的 `mlCorreId` 紀錄可靠關聯。
- `failure→detection→instruction→new edges ready→first accepted contribution` 各時間點由誰產生及如何對齊；控制面 API-call 數量的計數邊界如何固定。
- 失敗或未恢復 run 的原始檔如何完整保留；在 E2a／E2b 調整接受條件時，如何明示與 E0／E1 的設定差異。

以上是後續逐項討論的設計範圍，不表示各項已具備實作方案。E0／E1 的主要訓練與 Branch replacement 流程已有既有執行基礎，但使用舊契約；新增工作重點是對齊附錄 B，並取得能支持論文主張的識別、拓樸及事件證據。E2a／E2b 另需解決本文列出的執行語意。
