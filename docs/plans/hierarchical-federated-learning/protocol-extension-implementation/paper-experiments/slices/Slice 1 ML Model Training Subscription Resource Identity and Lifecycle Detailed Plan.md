# Slice 1 — ML Model Training 訂閱資源識別與生命週期詳細計畫

日期：2026-09-20

狀態：本地實作與審查已確認；正式多節點 testbed 整合驗證仍待執行。

本文件實作範圍以[論文實驗能力設計討論 §1](../Hierarchical%20FL%20Experiment%20Capability%20Design.md)已確認的訂閱識別方向為準；[能力與證據盤點](../Hierarchical%20FL%20Experiment%20Capability%20and%20Evidence%20Inventory.md)說明其對 E0／E1 的用途。這是本批**論文實驗能力**的 Slice 1，與先前 protocol extension 的 Slice 1 無關。

## 1. 交付結果與邊界

本 slice 只調整 `Nnwdaf_MLModelTraining` 訂閱的資源識別與生命週期，不改訓練演算法或 topology message。完成後，一條已建立的訂閱應同時滿足：

1. 接收端 Go 對外 SBI `Location` 末段的 `subscriptionResourceId`，等於其 PyMTLF 私有資源的 ID；不再另有 backend-local 訂閱 ID。
2. 發起端 Go 收到接收端 `Location` 後，以「接收端 `nfInstanceId`、ML Model Training 服務、`subscriptionResourceId`」識別正式 outbound route。發起端 PyMTLF 收到自己的 Go 私有 URL，其路徑亦含該 `subscriptionResourceId`，後續 PUT／PATCH／DELETE 仍交給自己的 Go 轉送。
3. 建立前產生的 `callbackRouteId` 只負責通知 URI 與建立中關聯；建立後保留在正式 route 供 callback 查找，但不再冒充訂閱資源 ID。
4. 建立、更新、刪除、通知、失敗清理及 backend reset／pending cleanup 均遵守以上區分。兩個不同接收端即使回覆相同資源 ID，也不會操作或清除彼此的訂閱。

本 slice 不是逐節點事件紀錄的實作；它先提供後續紀錄可使用的真實訂閱資源身分。也不實作 E2a／E2b 重掛、五 seed 排程或 testbed 收集腳本。第 2 項起沿用既有 `x-flTopology`／`x-flTopologyReport` 候選 schema，不因本 slice 改動拓樸欄位。

## 2. 現況、權威來源與契約

盤點基準：`NWDAF` `be3fa57`、`PyMTLF` `bdbd2a9`、本文件所在 `nwdaf-docs` `4492e20`。開始實作前重新核對 HEAD 與各儲存庫工作樹；本段不是對未來程式狀態的保證。

| 資訊 | 權威產生端與傳遞方式 | 現行狀態與目標 |
| --- | --- | --- |
| `subscriptionResourceId` | 接收端 Go 產生本專案使用的 UUIDv4，經私有 Create header 交給自己的 PyMTLF；接收端 SBI `201 Location` 再交給發起端 Go | 現在接收端 Go 與 PyMTLF 各自產生 ID；改為同一值。從外部 peer 收到的資源 ID 不額外要求 UUIDv4，只驗證是可安全識別的非空資源路徑片段。 |
| 接收端身分 | 發起端 Go 已有選定目標的 `nfInstanceId`；正式 outbound route 與私有 URL 同時保存它 | 資源 ID 不假設跨 NF 唯一，不能只以末段字串查找 outbound route。 |
| `callbackRouteId` | 發起端 Go 在 peer Create 前產生，寫入自己對外提供的 callback URI | 現在兼任 outbound route 主鍵與回給 PyMTLF 的資源路徑；改為只做建立中 token 和存續期 callback 查找。 |
| `notifCorreId`／`mlCorreId` | 現有訂閱／通知內容 | 維持既有通知和程序關聯語意，不取代資源 ID。 |

現行路徑證據：Go 的 `internal/sbi/processor/ml_model_training.go` 在 local Create 先產生 route ID，呼叫 PyMTLF 後另解析 backend `Location`；remote Create 則以 callback route ID 回給發起端 PyMTLF。`internal/context/ml_model_training_routes.go` 目前用單一字串主鍵保存全部 training routes；`internal/mtlf/client/ml_model.go` 的 Create 尚不能附加所需私有 header。PyMTLF 的 `src/py_mtlf/api/ml_model_training.py` 呼叫 `fl_client.create(payload)`；`src/py_mtlf/core/fl_client.py` 在 Create 內自行產生 UUID。`src/py_mtlf/core/fl_server.py` 保存 Go 私有 `Location`，並從末段取得 ID 用於既有關聯。

### 2.1 現有訂閱實際存放方式

以下是現行資料結構的重點摘錄，非本計畫要新增的型別。Go 的 `NWDAFContext` 在程序記憶體建立一張共用的 `mlModelTrainingRoutes`；`Add／Get／Update／Delete` 都只使用 `route.SubscriptionID` 這個字串當 key，`Find...ByBackendResourceID` 與 `Find...ByCorrelation` 則掃描同一張表。其他 ML Model 服務各有自己的 route map，但 training 的 inbound／outbound 混放在這張表。

```go
mlModelTrainingRoutes  map[string]MLModelTrainingSubscriptionRoute
mlModelDeletionRecords map[MLModelResourceKind]map[string]MLModelDeletionRecord

type MLModelTrainingSubscriptionRoute struct {
    SubscriptionID string
    PeerRoute      MLModelPeerRoute // Direction, SelectedTarget, PeerLocation,
                                    // BackendLocation, BackendResourceID, LifecycleState,
                                    // OperationRevision, ProcessGeneration, cleanup state
    // AcceptedRepresentation, BackendRepresentation, notifCorreId,
    // mlCorreId, expected round, feature state, participant NF identity
}
```

| 現行 route | `mlModelTrainingRoutes` key／`SubscriptionID` | 其他識別資訊 | 對外呈現 |
| --- | --- | --- | --- |
| 接收端 Go 的 inbound route | Go 建立時的 `localRouteID` | PyMTLF 自產的另一個 ID 存在 `PeerRoute.BackendResourceID`，私有完整位置在 `BackendLocation` | SBI `Location` 末段是 `localRouteID`。 |
| 發起端 Go 的 outbound route | Go 發給 peer 前的 `localRouteID`，實際用在 callback URI；本文稱 `callbackRouteId` | peer 真正的訂閱位置只存在 `PeerRoute.PeerLocation`，接收端身分存在 `SelectedTarget` | 回給發起端 PyMTLF 的 Go 私有 `Location` 末段仍是 `callbackRouteId`。 |

因此現行 `SubscriptionID` 在兩種方向的意義不同；它不是可靠的「同一份訂閱資源 ID」。若直接把 outbound `SubscriptionID` 換成 peer ID，兩個 peer 回相同 ID、或 inbound／outbound 撞號時便會互相覆寫／誤查；pending cleanup 也必須能找到正確 route。現有 `mlModelDeletionRecords[kind][resourceID]` 用於本 NF inbound 資源在 backend reset 後的晚到 DELETE 確認，不需為 outbound peer 另建一套 deletion ledger。

接收端 PyMTLF 另有程序記憶體中的 `_resources: dict[str, FLClientResource]`。目前 `fl_client.create` 自行 `uuid4()`，以此值建立 `_resources` key、`FLClientResource.subscription_id`、experiment reservation，並讓私有 API 的 `Location` 以該值結尾；`_deleting`、callback outbox key、timer 等也依此 ID 管理。發起端 PyMTLF 的 `FLParticipant.resource_location` 則保存自己的 Go 回傳的完整私有 URL，並將末段當作 subscription identity。這兩側都不是持久化訂閱儲存；Go 或 PyMTLF 程序重啟後不能假設這些記憶體索引仍在。

### 2.2 標準與實作結構參考

標準邊界：本地 Release 18 `specs/Rel-18/openapi/TS29520_Nnwdaf_MLModelTraining.yaml` 的 `/subscriptions` POST 要求 `201` representation 與 `Location`，後續 PUT／PATCH／DELETE 使用 `/subscriptions/{subscriptionId}`；`subscriptionId` 是 string。**不修改**公開 SBI 路徑、body、標準欄位或回應語意。`X-NWDAF-Subscription-Id` 僅是本專案接收端 Go→自身 PyMTLF 的私有 Create header，不送給其他 NWDAF。私有 outbound URL 是發起端 Go 的代理路徑，不是 peer `Location`。

free5GC 結構對照只用本地唯讀鏡像：BSF `internal/sbi/processor/subscriptions.go` 顯示資源 ID 與 `201 Location` 的公開資源關係；PCF `internal/sbi/api_httpcallback.go` 顯示 callback route 與處理器分界。具體路由狀態、私有後端契約仍以本專案程式和上述 OpenAPI 為準，不從鏡像推論相同的生命週期。

## 3. 基準流程的處置

| 既有階段 | 本 slice 處置 |
| --- | --- |
| 由發起端 PyMTLF 觸發、Go 選定 peer | 沿用；不改 NRF 選擇或候選 policy。 |
| 接收端 preparation／PyMTLF 建立資源 | 調整 ID 的產生者與私有傳遞；其餘 participation／feature 驗證維持。 |
| Create 回應與資源 handoff | 調整接收端 Go／PyMTLF 的同 ID 檢查，以及發起端的 scoped route 綁定與私有 `Location`。 |
| 正常訓練、round 驗證與 model transport | 沿用；不得以更改 ID 為由改動訓練語意。 |
| PUT／PATCH／DELETE | 沿用原有 body、status 與操作規則；改為以正式資源 key 尋址，保留既有 revision fencing 與 failure rollback。 |
| Notify／callback | 沿用 `notifCorreId`／`mlCorreId` 驗證與 Go→PyMTLF 轉送；只將 callback 查找與正式資源 key 拆開。 |
| 建立拒絕、timeout、早到通知、部分失敗 | 調整暫存與正式索引的清理；保留早到通知 `503` 與通知 outbox 重試。無 peer `Location` 時不宣稱已建立。 |
| backend reset、pending cleanup、終止 | 調整 training route 的遍歷與清理目標；inbound 晚到 DELETE 繼續沿用既有 tombstone，避免以 callback token 或不含 peer 身分的 ID 誤查 outbound route。 |
| 跨 Go 程序重啟的殘留資源恢復 | 本次不新增持久化／重啟後對帳；依主設計列為範圍外。 |

## 4. 實作契約與資料流

### 4.1 接收端：Go → 自身 PyMTLF → 對外 SBI

1. 接收端 Go 在外部 SBI Create 或本地私有入口收到合法訂閱後，產生 UUIDv4 `subscriptionResourceId`，先保留建立中 route。此 ID 不寫入標準訂閱 body。
2. Go 對自己的 PyMTLF 私有 POST `/internal/v1/ml-model-training/subscriptions` 附上 `X-NWDAF-Subscription-Id`。在現有 `internal/mtlf/client` 路徑加入最小 header 傳遞能力；不讓 peer consumer 共用或轉送此 header。
3. PyMTLF 私有 API 要求 header 存在、值為本專案使用的 UUIDv4 且尚未占用；`fl_client.create` 使用它作為 `_resources` key、實驗 reservation 關聯及回應 `Location` 末段，不再自產另一個 resource ID。缺少／格式錯誤拒絕；已占用不得覆蓋或誤清既有資源。
4. Go 檢查 PyMTLF 的 `201`、representation、`Location` 的私有路徑及末段 ID 均符合本次建立。成功才把 route 轉為 active，並以同一 ID 回傳外部 SBI `Location`。後續外部 PUT／PATCH／DELETE 直接用此 ID 找到 local route，轉送自身 PyMTLF 同 ID 資源。

沿用下節 Root 訂閱 Branch 的例子：Branch Go 收到 SBI Create 後，向**自己的** Branch PyMTLF 發送私有 Create。以下只節錄與 ID 相關的 HTTP 起始行及 header；請求 body 仍是訂閱 JSON（其中 `notifUri` 已由 Branch Go 改為自己的 callback URI），不把資源 ID 塞進 body。

```http
POST /internal/v1/ml-model-training/subscriptions HTTP/1.1
Host: branch-pymtlf.internal
Content-Type: application/json
X-NWDAF-Subscription-Id: 550e8400-e29b-41d4-a716-446655440000
```

Branch PyMTLF 用 header 中的 ID 建立 `_resources` 項目，並向 Branch Go 回覆同一個 ID；以下同樣省略訂閱 representation body：

```http
HTTP/1.1 201 Created
Location: http://branch-pymtlf.internal/internal/v1/ml-model-training/subscriptions/550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json
```

Branch Go 驗證私有回覆後，才回傳下節所示的 SBI `201 Location` 給 Root Go。這個私有 header 不會出現在 Branch Go 的 SBI 回覆，也不是 Root PyMTLF 發起訂閱時送給 Root Go 的 header。

### 4.2 發起端：自身 PyMTLF → Go → peer Go

1. 發起端 PyMTLF 仍以現有私有 Create 及選定目標資訊請自己的 Go 建立訂閱。Go 配置 `callbackRouteId`、預留 `Creating` route，並把它寫入發給 peer 的 `notifUri`。
2. peer `201` 後，發起端 Go 驗證並保留完整、已解析的 peer `Location`；從預期的 ML Model Training subscription 資源路徑提取非空 `subscriptionResourceId`。不要用現有 `ResourceIDFromLocation` 對 peer 強制 UUIDv4；它目前供本地 backend `Location` 使用，不能全域放寬而影響其他服務。對私有 URL 路徑片段做正確編碼／解碼。
3. Go 將建立中 route 原子綁定到正式 key（目標 `nfInstanceId`、服務、`subscriptionResourceId`），並回覆自己的 PyMTLF 例如 `/internal/v1/ml-model-training/targets/{targetNfInstanceId}/subscriptions/{subscriptionResourceId}` 的私有 `Location`。它不把 peer 對外 URL 交給 PyMTLF。PyMTLF 繼續保存完整私有 URL，後續 PUT／PATCH／DELETE 原樣交還自己的 Go。
4. Go 的私有更新／刪除 route 解析目標身分和 ID，查到保存的 peer `Location` 後再由 peer consumer 發出操作；不可從私有 URL 重新猜 peer API root。PyMTLF 從 `Location` 末段取得 ID 的既有用法須依新路徑檢查，避免把編碼形式或目標 scope 誤認成資源 ID。

以 Root PyMTLF 訂閱 Branch NWDAF 為例，發起方向是 `Root PyMTLF → Root Go → SBI → Branch Go → Branch PyMTLF`。建立完成後，Branch Go 先在 SBI 回覆 Root Go：

```http
HTTP/1.1 201 Created
Location: https://branch.example/nnwdaf-mlmodeltraining/v1/subscriptions/550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json
```

Root Go 以 Branch 的 `nfInstanceId` 與這個 `Location` 末段的 `subscriptionResourceId` 綁定正式路由，再回覆 Root PyMTLF：

```http
HTTP/1.1 201 Created
Location: http://root-go.internal/internal/v1/ml-model-training/targets/10000000-0000-4000-8000-000000000101/subscriptions/550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json
```

兩個 `201` 都保留訂閱 representation 作為回應 body；ID 由各自的 `Location` 表達。Root PyMTLF 保存的是**自己的 Go 私有 URL**，不是 Branch 的 SBI URL；改動後其末段才是 Branch 真正建立的訂閱 ID。現行實作回給 Root PyMTLF 的私有 URL 末段則是 Root Go 在發送 Create 前產生的 `callbackRouteId`。該暫時 ID 仍留在發給 Branch 的 callback URI 與 Root Go 的 callback 查找欄位中，但不再充當訂閱資源 ID。接收端 Go → 自身 PyMTLF 的私有 ID header 只用於 Branch 端建立本地資源，不是 Root PyMTLF 發起 Create 時附帶的 header。

### 4.3 路由索引與 callback

目標不是把現有 `map[string]route` 的 key 直接換成另一個裸字串。下列結構是本 slice 的**邏輯儲存契約草圖**，型別名稱可依 Go 現有命名調整，但不同索引的用途及不變條件須保留：

```go
type TrainingResourceKey struct {
    Direction         MLModelRouteDirection // inbound or outbound
    OwnerNFInstanceID string                // local NF for inbound; receiver NF for outbound
    SubscriptionID    string                // {subscriptionId} from SBI Location
}

type MLModelTrainingSubscriptionRoute struct {
    SubscriptionID    string // the actual {subscriptionId} when known
    CallbackRouteID   string // outbound callback routing only
    PeerRoute         MLModelPeerRoute
    // Existing representation, correlation, round, feature, and revision fields.
}

routesByResourceKey       map[TrainingResourceKey]MLModelTrainingSubscriptionRoute
pendingByCallbackRouteID map[string]MLModelTrainingSubscriptionRoute
```

`TrainingResourceKey` 的 Go store 專供 ML Model Training，因此服務名由 store 本身固定；跨服務或離線對照時，完整資源身分仍是「owner `nfInstanceId`、服務、`subscriptionId`」。`Direction` 是同一 Go 程序內 inbound／outbound route 的區隔，不是新增的 SBI 識別欄位。`routesByResourceKey` 可以存放已知資源 ID、但仍處於 `CREATING` 的 inbound route；名稱不代表全部已 active。`MLModelPeerRoute.SelectedTarget` 和 `PeerLocation` 繼續承載 outbound 的目標與實際操作位置。共用 `MLModelPeerRoute.BackendResourceID` 欄位可因其他 ML Model 服務保留，但 training 不再用它表示第二個 ID 或作為尋址權威；接收端 training 的私有 CRUD 直接使用 `SubscriptionID`。現有 training 專用的 `Find...ByBackendResourceID` 若已無 caller，應移除，不為舊雙 ID 語意保留別名。`BackendLocation` 如保留，只作回應檢查或診斷，不能成為另一個路由主鍵。晚到 inbound DELETE 繼續使用現有 deletion ledger 與本 NF 的資源 ID，不新增 training 專用帳本。

| 時點或操作 | 應存放／查找的結構 | 狀態變化 |
| --- | --- | --- |
| 接收端 Go 開始建立 | `routesByResourceKey[(inbound, 本 NF, Go 配置的 ID)]` | route 為 `CREATING`；同一 ID 經私有 header 交給 PyMTLF。私有 Create 及檢查成功後改 `ACTIVE`，失敗則移除本次 route。 |
| 發起端 Go 向 peer 發 Create 前 | `pendingByCallbackRouteID[callbackRouteId]` | route 為 `CREATING`；此時真正的 peer `subscriptionId` 尚不存在於本地已知狀態。callback 可查到 pending route 並回 `503`。 |
| 發起端 Go 收到有效 peer `201 Location` | `routesByResourceKey[(outbound, peer NF, peer ID)]` | 保存完整 `PeerLocation` 與原 `CallbackRouteID`，在同一次受鎖保護的轉換中啟用 route 並移除 pending 項目；之後才回自己的 PyMTLF 正式私有 `Location`。 |
| peer 已建立，但正式綁定失敗 | 留在 `pendingByCallbackRouteID`，沿用現有失敗清理狀態與已知 `PeerLocation` | 不得對 PyMTLF 宣稱 active 訂閱；清理失敗依現有機制處理，該實驗 run 不視為成功。 |
| 發起端 PyMTLF 做 PUT／PATCH／DELETE | 從私有 URL 解析 peer NF + ID，查 `routesByResourceKey[(outbound, peer NF, ID)]` | 由 Go 使用保存的 `PeerLocation` 操作 peer；操作失敗時按現有 revision／rollback 處理，不移除仍 active 的索引。 |
| 接收端收到公開 PUT／PATCH／DELETE | 用本 NF 身分與公開 path ID 查 `routesByResourceKey[(inbound, 本 NF, ID)]` | 對自身 PyMTLF 的同 ID 資源執行；成功終止才移除 route。 |
| 通知抵達發起端 Go | 先查 `pendingByCallbackRouteID`，再依 `CallbackRouteID` 查找正式 route | pending 的早到通知仍回 `503`；active 才進入原有 correlation／identity 驗證與轉送。正式 route 數量很少，可沿用現有掃描查找方式，不新增第三張索引表。 |
| reset、pending cleanup 或晚到 DELETE | 迭代 pending 與 active 並保留每筆 route 的完整 key | 維持現有 inbound deletion ledger；cleanup 操作使用 route 保存的實際目標，不以裸 ID 誤查 outbound peer。 |

建立後可丟掉的只有**已成功轉換的 pending 項目**，不是 `callbackRouteId` 本身：通知 URI 已發給 peer，故正式 route 的 `CallbackRouteID` 須保留到訂閱終止。`notifCorreId` 在 pending 與 active 的聯集中仍維持既有唯一性；接收端私有通知沒有 callback token 時，仍能依它找到正確 inbound route。`Get／Update／Delete／GetAll` 及清理迴圈應傳遞完整 route key 或 pending 參照，不可再以 `route.SubscriptionID` 單獨定位 outbound route。

PyMTLF 的目標儲存仍是一張 `_resources`，但 key 改由 Go 私有 header 指定：`_resources[subscriptionId]`、`FLClientResource.subscription_id`、experiment reservation、`_deleting`、outbox 與 timer 均使用同一 ID，不新增 Go ID→backend ID 映射。沿用現有資源鎖檢查已存在的 ID，拒絕覆寫；Go 配置 UUIDv4，不為未觀察到的並行同 ID Create 新增預留集合或復原流程。發起端 PyMTLF 仍保存完整 Go 私有 `resource_location`；其末段是編碼後的 peer ID，若要當 identity 比較須正確解碼，而 PUT／PATCH／DELETE 應使用完整 URL。

callback URI 在資源存續期仍含原 `callbackRouteId`。通知先依此 token 找到 active route，再沿用既有 correlation／participant／round 驗證；`Creating` 階段回 `503`，讓接收端 PyMTLF outbox 重試，active 後回原有成功狀態。對於沒有 callback token 的本地後端通知，保留既有 `notifCorreId` 查找及驗證。成功 DELETE 或確定終止後移除正式 route；操作失敗不可提前清除。

調整 `Get／Update／Delete／GetAll`、backend reset、pending cleanup、termination tombstone 及 revision fencing 的 training-route 呼叫點，令其使用正確 route key；不能只把 Create 回應換成新的 URL。其他 ML Model Provision／Monitor route 不在修改範圍。

### 4.4 失敗處理範圍

Create 明確失敗或未取得可用的 `Location`，便不宣稱訂閱建立成功；已知本次資源時沿用現有清理／重試機制，不能以 `callbackRouteId` 猜測 peer 資源 ID。通知早於 Create 回覆時，保留現有 `Creating` 回 `503`、outbox 重試後 active 回 `204` 的路徑。已建立訂閱的 PUT／PATCH／DELETE 若失敗，沿用現有 revision／rollback 規則，不把失敗操作記成成功，也不提前移除仍需使用的 route。

接收端私有 header 缺失、格式不合法或 ID 已占用，直接拒絕 Create。接收端 PyMTLF 回覆與本次已知 ID 不符、peer 回覆異常，或正式 route 無法綁定時，不對上游回報成功；沿用既有失敗清理，該實驗 run 視為失敗並重跑。不為受控 testbed 中未觀察到的同一接收端 ID 撞號、異常 peer `Location` 或回覆遺失建立額外補償狀態機、持久化或專項復原測試。兩個**不同** peer 回相同 ID 則是正常的命名空間情況，仍須由 `nfInstanceId` 隔離。

## 5. 儲存庫工作與順序

| 順序 | 儲存庫／主要擁有者 | 工作 |
| --- | --- | --- |
| 1 | `NWDAF/internal/context/ml_model_training_routes.go`、相關 processor | 將正式 route 改以 scoped resource key 儲存；建立中的 outbound route 以 callback token 暫存，完成時轉入正式 route；更新所有 training route 查找／清理呼叫點。 |
| 2 | `NWDAF/internal/mtlf/client/ml_model.go`、`internal/sbi/processor/ml_model_training.go` | 接收端預配置 ID，透過專屬私有 header 傳給 PyMTLF，確認私有 `Location` 與本次 ID 相同，調整 local CRUD；失敗時沿用現有清理。 |
| 3 | `PyMTLF/src/py_mtlf/api/ml_model_training.py`、`core/fl_client.py` | 接受／驗證私有 header，使用 Go 給的同一 ID 建立資源與回應；更新直接呼叫 Create 的測試，不保留舊版自產 ID 的相容分支。 |
| 4 | `NWDAF/internal/mtlf/api_ml_model_gateway.go`、`internal/sbi/processor/ml_model_training.go`、`internal/context` | 發起端 scoped 私有 URL 與正式 route；調整 callback 查找、PUT／PATCH／DELETE、reset／pending cleanup。既有 inbound tombstone 依原機制運作；外部 SBI public route 不增新欄位或 endpoint。 |
| 5 | `PyMTLF/src/py_mtlf/core/fl_server.py` 及使用它的 Root／Branch 路徑 | 以回傳的完整 Go 私有 URL 執行後續操作；檢查從末段取得 ID 的地方只用於對應訂閱關聯，且能正確處理編碼。 |

以上是端到端施工順序，不要求跨儲存庫混成一個 commit。任何需要改動其他標準欄位、其他服務的共用 route 契約或額外持久化的情況，先回報設計差異，不默默擴大本 slice。

## 6. 驗收與驗證

| 驗收案例 | 直接驗證點 |
| --- | --- |
| Local Create／後續 CRUD | 接收端 Go 產生的 UUIDv4 經私有 header 到 PyMTLF；私有與 SBI `Location`、PyMTLF 儲存 key 相同；同一 ID 完成 PUT／PATCH／DELETE。 |
| Remote Create／後續 CRUD | peer 回傳資源 ID 後，發起端 PyMTLF 的私有 `Location` 含 peer 身分與該 ID；Go 對已保存的 peer `Location` 轉送操作，不誤用 callback token。 |
| 命名空間隔離 | 兩個 peer 回同一非 UUID 資源 ID，仍能分別更新、刪除、接收通知；不碰另一條 route。 |
| 儲存結構與索引轉換 | 檢查 inbound `CREATING`→`ACTIVE`、outbound pending→active；callback token 在 active route 中仍能查到，`GetAll` 與 pending cleanup 可處理兩張表中的 route。 |
| 建立中通知 | 以可控時序重現 callback 先到 `503`、綁定後重試 `204`，並驗證 correlation；不能只 mock 掉處理 callback 的 Go 或通知 outbox。 |
| 基本失敗路徑 | 私有 header 無效、peer Create 明確拒絕、更新／刪除失敗時，不回報成功或誤刪既有 route；沿用現有清理。罕見協定異常或同一 peer ID 撞號不新增專項復原測試。 |
| 現有生命週期回歸 | backend reset、pending cleanup、inbound 晚到 DELETE、termination notification 及既有 training subscription 不因重設索引失效。 |

Go 與 PyMTLF 在對應的現有測試中驗證實際 header、`Location`、路由 key、callback 與後續操作目標；不要求逐一覆蓋所有罕見錯誤排列。若現有本地多程序環境可用，再以兩組 NWDAF／PyMTLF 走一次 Create→Notify→PATCH／DELETE；若未執行，明列為整合驗證缺口，不冒稱 testbed 通過。

正式驗證在各自儲存庫執行：`NWDAF` focused Go tests、`make test`、`make lint`、`make build`；`PyMTLF` focused pytest、全套 pytest 與既有 lint。實際指令與結果依實作時倉庫入口決定。

## 7. 明確延後與完成門檻

- `future-phase handoff`：逐節點 `observations.jsonl` 訂閱操作原始事件屬後續證據工作；訂閱資源交互次數由實驗後離線統計，testbed 收集器另須對新資源 ID 正確解析。本 slice 不把人類可讀日誌當作完成證據。
- `future-phase handoff`：以既有 `x-flTopology`／`x-flTopologyReport` 候選 schema 規劃逐節點證據、E2a／E2b mixed-depth execution 與五 seed 實驗；不預設附錄 B wire migration 或 `topologyVersion`／`reparentInstruction` 欄位。
- `optional hardening`：跨 Go 程序重啟後的 orphan 訂閱掃描／對帳，不列入此處的資源識別驗收。
- `integration verification gap`：正式 multi-host testbed、控制器故障注入與新實驗資料收集需在後續階段完成；本 slice 的單元／本地流程測試不替代它們。

實作時對照本計畫與主設計 §1，將每個驗收點映射到 production 路徑和測試。使用者已確認 `NWDAF` 與 `PyMTLF` 的本地實作及審查結果；正式多節點 testbed 整合驗證不在本次本地測試結果之內。
