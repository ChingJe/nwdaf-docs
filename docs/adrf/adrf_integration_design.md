# NWDAF × ADRF × Daisy 整合設計

---

## 0. 修訂紀錄

| 日期 | 版本 | 說明 |
|------|------|------|
| 2026-03-25 | v1.0 | 初版（Phase E2 完成時確立） |
| 2026-03-26 | v1.1 | Phase E3 設計修訂：對齊 ADRF 方 adrf1/adrf2 設計稿；調整 fetch 策略、收斂條件、watchdog、endpoint 路徑、HTTP codes；詳見各節 `[v1.1 變更]` 標記 |
| 2026-03-27 | v1.2 | Phase E3 實作完成：函式重命名（`TriggerRetraining`→`startRetrainWorkflow`、`triggerTrainingAsync`→`submitDaisyTask`）；UPF notification 拆分為逐 item ADRF record；新增 retrieval log 計數；記錄 2026-03-26 實測觀察 |
| 2026-03-27 | v1.3 | 更新 E2E 驗證狀態：UploadData 與 `terminationReq=true` 均確認正常（第二次實測）；Daisy `model.npy` 為已知 bug，NWDAF/ADRF 流程全數通過 |

---

## 1. 概述

記錄 NWDAF 整合 ADRF 的設計決策與 E2E 流程。
整合目的：**在 retrain 時，由 ADRF 提供歷史 UPF 流量資料作為 Daisy 的訓練集。**

規格參照：3GPP TS 29.575（Nadrf_DataManagement Service）。

---

## 2. 規格重點摘要（TS 29.575）

### 2.1 Nadrf_DataManagement 服務操作

| 操作 | 方向 | 說明 |
|------|------|------|
| `StorageRequest` | NWDAF → ADRF | 將資料存入 ADRF，回 201 + storeTransId |
| `RetrievalSubscribe` | NWDAF → ADRF | 訂閱歷史資料，ADRF 主動 push 通知（含 fetch correlation IDs） |
| `RetrievalNotify` | ADRF → NWDAF | 通知內容可以是直接帶資料，或帶 fetchInstruct（ID list） |
| `RetrievalRequest` | NWDAF → ADRF | 用 fetch-correlation-ids GET 實際資料 |
| `RetrievalUnsubscribe` | NWDAF → ADRF | 取消 retrieval 訂閱 |

### 2.2 關鍵 Schema

**`NadrfDataStoreRecord`**（存入與取出的資料格式）：
```
oneOf:
  - anaSub + anaNotifications  （analytics 路徑）
  - dataSub + dataNotif        （原始資料路徑）
```

**`DataSubscription`**（dataSub 的型別，每個物件 oneOf）：
```
oneOf: amfDataSub | smfDataSub | upfDataSub | udmDataSub | ...
```
`dataSub` 是 **array**，允許一筆 dataNotif 對應多個訂閱描述。

**`DataNotification`**（dataNotif 的型別，oneOf）：
```
oneOf: amfEventNotifs | smfEventNotifs | upfEventNotifs | ...
```

---

## 3. smfDataSub + upfEventNotifs 設計決策

### 3.1 問題背景

NWDAF 取 UPF 資料的路徑：
1. NWDAF 向 **SMF** 訂閱，指定 event = `UPF_EVENT`
2. SMF 在內部向 UPF 建立訂閱（`UpfEventSubscription`）—— NWDAF 不可見、無法取得
3. UPF **直接**將資料（`NotificationData`）送至 NWDAF，不經過 SMF

因此 NWDAF 手上有：
- `smfDataSub`（`NsmfEventExposure`）：自己向 SMF 建立的訂閱物件
- `upfEventNotifs`（`NotificationData` array）：UPF 直接送來的資料
- **沒有** `upfDataSub`（`UpfEventSubscription`）：SMF 內部的，NWDAF 無從取得

### 3.2 合規分析

**線索 1 — Schema 無跨欄位約束**：`dataSub` 的 NF 選擇與 `dataNotif` 的 NF 選擇是兩個獨立的 `oneOf`，schema 層沒有要求兩者選同一個 NF 來源。

**線索 2 — "corresponding" 的語義**：spec 說 dataSub 是 "the subscription information of the corresponding data notification"。「corresponding」可解讀為「導致此筆資料被收到的那條訂閱」——即 NWDAF 向 SMF 建立的 `NsmfEventExposure`，而非 SMF 內部對 UPF 的訂閱。

**線索 3 — UPF event 只能透過 SMF 訂閱**：TS 29.508 明確說明 `UPF_EVENT: UPF event subscribed via SMF`。若要求 `upfDataSub` 與 `upfEventNotifs` 同源配對，在此情境下 `upfDataSub` 永遠無法填，形成無解死角，不符合規格設計意圖。

**結論**：`smfDataSub + upfEventNotifs` **不違反 TS 29.575 規格**，是此情境下唯一合理可行的做法。

---

## 4. E2E 整合流程

### 4.1 Phase 1：NWDAF 即時存入 ADRF

```
UPF → (NotificationData) → NWDAF
                              ↓
                    [buffer 累積至閾值]
                              ↓
              NWDAF → POST /data-store-records → ADRF
              body: NadrfDataStoreRecord {
                dataSub: [{ smfDataSub: NsmfEventExposure }],
                dataNotif: { upfEventNotifs: [...] }
              }
              → 回 201 + storeTransId
```

**存入觸發條件**：buffer 累積筆數達到設定閾值（config：`adrf.storageThreshold`）後 flush。預設值為 **1**（逐筆存入），日後視需求調高以批次存入。

### 4.2 Phase 2：MTLF 觸發 Retrain 前的資料準備

**步驟 1 — 按 SUPI 向 ADRF 訂閱歷史資料**

NWDAF 維護 SUPI → Group ID 的 map（因為向 SMF 訂閱時無法指定 group，只能逐 SUPI 訂閱）。
MTLF 對每個 SUPI 個別發起 `RetrievalSubscribe`：

```
MTLF → POST /data-retrieval-subscriptions → ADRF
body: NadrfDataRetrievalSubscription {
  notifCorrId: "<TID>",
  notificationURI: "http://nwdaf/collector/retrieval-notify",
  timePeriod: { startTime: ..., stopTime: ... },
  dataSub: { smfDataSub: { supi: "imsi-001", ... } },
  consTrigNotif: true
}
→ 回 201 + subscriptionId
```

> **[v1.1 變更]** `notificationURI` 路徑由 `/adrf-notify` 改為 `/collector/retrieval-notify`。理由：callback 屬於資料收集域，與 `/collector/upf-notify` 同組；`/adrf` 或 `/anlf` 前綴均不能清楚表達接收方職責，`/collector` 在現有 server 路由中已代表「外部資料推入 NWDAF」的語意群組。

`consTrigNotif: true` 讓 ADRF 進入 fetch 模式：callback 只送 `fetchInstruct`（ID list），不直接帶資料本體。`notifCorrId` 與本次 retrain 的 TID 一對一綁定。

**多 SUPI 訂閱的 counter 設計**：一次 retrain 可能涉及多個 SUPI，每個 SUPI 各發一個 `RetrievalSubscribe`，但所有訂閱共用同一個 `notifCorrId = TID`（目的是讓所有 callback 都與同一次 retrain job 關聯）。`dataSub.smfDataSub.supi` 已指定該訂閱的目標 UE，ADRF 依此過濾；fetch 回來的每筆 record 也可從 `dataSub[0].smfDataSub.supi` 自行取得 SUPI，不需在 `notifCorrId` 中編碼。MTLF 需追蹤「本次 retrain 發出了 N 個 `RetrievalSubscribe`」，收到 N 個 `terminationReq=true` 後才代表全部訂閱完成。

**步驟 2 — 接收 fetch correlation ID list**

ADRF 主動 POST 到 `notificationURI`，body 為 `NadrfDataRetrievalNotification`，帶 `fetchInstruct`（fetch correlation IDs），不直接帶資料本體。

MTLF 收到後記錄所有 IDs，等待所有 SUPI 的通知收齊。

**步驟 3 — GET 實際資料（依 `fetchBatchSize` 控制每次帶幾個 ID）**

```
MTLF → GET /data-store-records?fetch-correlation-ids=<id1>[,id2,...] → ADRF
→ 回 200 + NadrfDataStoreRecord
```

> **[v1.1 變更]** fetch 策略由「固定分批（每批 ≤30 筆）」改為由 `adrf.fetchBatchSize` config 控制（**預設 1**，即每次 GET 僅帶 1 個 ID），對齊 ADRF 方 adrf1 設計稿的逐筆設計。
>
> 規格（TS 29.575 YAML）的 `fetch-correlation-ids` 為 `style: form, explode: false`，schema 允許多個 ID 逗號分隔（`type: array, minItems: 1`），批次 GET 規格合規；`fetchBatchSize` 保留在 `AdrfConfig` 作為未來可調整的選項，目前以 1 為預設值確保與 ADRF 端行為一致。

**`RetrievalRequest` HTTP response 處理**：

| HTTP code | 規格說明 | NWDAF 處理 |
|-----------|---------|-----------|
| `200 OK` | "Data store records are returned"，body 為 `NadrfDataStoreRecord` | 正常處理，推入 retrain staging |
| `204 No Content` | "No matching ADRF data were found"（此 fetchCorrId 在 ADRF 查無記錄） | 視為「此 ID 無資料」，不重試，繼續下一個 ID |
| `400` / `401` / `403` | 請求或授權錯誤 | 記錄錯誤原因，進入異常收斂（仍需做 unsubscribe） |
| `404 Not Found` | ADRF endpoint 本身找不到（配置或程式問題）；**注意：此與 `204` 不同，`204` 才是「資料不存在」** | 視為配置錯誤，進入異常收斂 |
| `406 Not Acceptable` | ADRF 無法以要求格式回應 | 進入異常收斂 |
| `429 Too Many Requests` | Rate limit | backoff 後重試；超過上限進異常收斂 |
| `5xx` / timeout | ADRF 內部錯誤或連線逾時 | 有限次 backoff 重試；超過上限進異常收斂 |

GET response 結構（`NadrfDataStoreRecord`）：
```json
{
  "dataSub": [{ "smfDataSub": { "supi": "imsi-001", ... } }],
  "dataNotif": {
    "upfEventNotifs": [
      { "notificationItems": [...], "correlationId": "corr-session-001" }
    ]
  }
}
```

**步驟 4 — 標記資料並推入 Daisy**

每筆 GET 回來的 `NadrfDataStoreRecord`，MTLF：
1. 從 `dataSub[0].smfDataSub.supi` 取出 SUPI
2. 查 SUPI → Group ID map 得到 `group_id`
3. 附上 `TID` 和 `group_id`，將 `dataNotif.upfEventNotifs` 原封推入 Daisy

```json
POST /upload_data

{
  "TID": "abc123",
  "group_id": "ue-comm-group1",
  "upfEventNotifs": [
    { "notificationItems": [...], "correlationId": "corr-session-001" }
  ]
}
```

**步驟 5 — 清理訂閱**

> **[v1.1 變更]** 收斂條件由「OR」改為「**AND**」：必須同時滿足以下兩個條件才送 DELETE，原因見下方說明。

**正常收斂條件（AND）**：
1. **pending queue 清空**：所有已收到的 `fetchCorrIds` 均已 GET 完畢
2. **`terminationReq=true` 已收到**：ADRF 明確告知不再發送新 callback

兩者缺一不可。原因：NWDAF 在收到 `terminationReq=true` 之前無法得知 ADRF 是否還有更多 callback 未送達；若只以「pending queue 清空」為條件，可能在 ADRF 仍有後續批次時提前結束。

```
DELETE /data-retrieval-subscriptions/{subscriptionId}
```

**Watchdog（防止永久等待）**：當 pending queue 清空但 `terminationReq=true` 遲遲未到時，以「最後一次成功處理 callback 的時間點」為基準，若超過閾值（例如 120 秒）仍無新 callback，watchdog 觸發：視同資料已全部收取，進入正常收斂流程（送 DELETE → 觸發訓練），並記錄 warning。

**`RetrievalUnsubscribe` HTTP response 處理**：

| HTTP code | 規格說明 | NWDAF 處理 |
|-----------|---------|-----------|
| `204 No Content` | "The subscription was deleted" | 成功 |
| `404 Not Found` | Subscription 不存在（可能已被 ADRF 自行清理） | V0：視為已達清理目的，當作成功 |
| `400` / `401` / `403` | 請求或授權錯誤 | 記錄錯誤，不重試 |
| `429 Too Many Requests` | Rate limit | backoff 後重試 |
| `5xx` / timeout | 暫時性錯誤 | 有限次 backoff 重試 |

**異常收斂**（retrain job 取消、超時、GET 失敗達上限）：也應主動送 DELETE 清理資源，再依情況決定是否仍觸發訓練。

**步驟 6 — 觸發 Retrain**

訂閱清理完畢後，MTLF 發起 Daisy retrain，附上 `TID`。
Daisy 以 `TID` 查出對應的 dataset，執行訓練。

### 4.3 整體序列圖（文字描述）

```
UPF  →  NWDAF(AnLF)  →  [buffer]  →  ADRF (持續存入)
                                         ↑
                                    [歷史資料庫]
                                         ↓
MTLF 偵測到 accuracy 下降
  → 按每個 SUPI 發 RetrievalSubscribe 給 ADRF
  → ADRF push fetchInstruct (ID list，可多批 callback)
  → MTLF 逐 ID GET 實際資料（每次帶 fetchBatchSize 個，預設 1）
  → 每筆資料標記 group_id + TID，推入 Daisy
  → pending queue 清空 AND 收到 terminationReq=true（或 watchdog 觸發）
  → DELETE /data-retrieval-subscriptions/{subscriptionId}
  → MTLF 發 submitDaisyTask（帶 TID）給 Daisy
  → Daisy 以 TID 查 dataset → 執行 FL 訓練
  → Daisy callback NWDAF（帶 model_url）
  → NWDAF 換版流程（現有 Phase A/B 流程）
```

---

## 5. 實作細節

### 5.1 NWDAF 側（Go）

| 項目 | 說明 |
|------|------|
| Processor buffer（E2 已實作） | `internal/sbi/processor/adrf_buffer.go`：收到 UPF notification 時累積，達 `storageThreshold` 後 flush；無獨立 `infos` map，`AdrfSmfInfo` 在 `add()` 時直接傳入 `flushOne()` |
| Per-item ADRF storage（E3 新增） | `upf_notify.go` 在 `adrfBuffer.add()` 前，將每個 `NotificationItem` 拆分為獨立的 `UpfNotificationData`（各含單一 item），確保 ADRF 時間窗查詢以 per-item `startTime` 精確匹配；若打包整份 notification，ADRF 時間窗命中任一 item 就整包回傳，精度不足 |
| ADRF consumer（E2/E3 已實作） | `internal/sbi/consumer/adrf_service.go`：`AdrfClient` + `StorageRequest`（E2）、`RetrievalSubscribe`/`RetrievalRequest`/`RetrievalUnsubscribe`（E3）；含 connection-pool Transport |
| `smfDataSub` 型別（E2 已實作） | 重用 `ExtendedNsmfEventExposure`（已符合 TS29508 `NsmfEventExposure`），不另建新 struct |
| ADRF notify endpoint（E3 已實作） | `POST /collector/retrieval-notify`：接收 ADRF 的 fetchInstruct 通知；掛在 `/collector` group（`api_collector.go`），與 `/collector/upf-notify` 同組，語意為「外部資料推入 NWDAF」 |
| MTLF retrain 前置（E3 已實作） | `startRetrainWorkflow`（原 `TriggerRetraining`）：ADRF 啟用時走 `runAdrfRetrainWorkflow`；關閉或 fallback 時直接呼叫 `submitDaisyTask`（原 `triggerTrainingAsync`） |
| ADRF retrieval job（E3 已實作） | `internal/mtlf/adrf_retrieval.go`：`retrainJob` struct（`sync.Map activeJobs`、`fetchCh chan []string`、`closeOnce`、watchdog timer）+ `runFetchLoop`（drain fetchCh，每批 `fetchBatchSize` 個 ID）+ `HandleAdrfRetrievalNotify`（路由 callback 到 job，印 `ids/terminationReq/termCount` log） |
| Config（E2/E3 已實作） | `pkg/factory/config.go`：`AdrfConfig`（`Url`、`StorageThreshold`、`FetchBatchSize`[default 1，ADRF V0 limit]、`RetrainWindow`[default 1800s]、`WatchdogTimeout`[default 120s]） |
| Context lifecycle（E2 已實作） | `AdrfSmfInfo` keyed by `correlationId`；SMF 訂閱建立時 `StoreAdrfSmfInfo`；訂閱釋放時 `DeleteAdrfSmfInfo`（`cleanupDataCollection`） |

### 5.2 Daisy 側（新需求）

| 項目 | 說明 |
|------|------|
| 資料接收 endpoint | `POST /upload_data`：接收帶有 `TID`、`group_id`、`upfEventNotifs` 的訓練資料 |
| 資料庫 | 持久化儲存各 TID 的 dataset |
| 訓練查詢 | `publish_task` 帶 `TID`，Daisy 以此查出 dataset |

### 5.3 ADRF 相依

ADRF 需已部署且可連線。NWDAF config 新增 `adrf.url`（`AdrfConfig.Url`）。

---

## 6. 分階段實作計畫

### Phase E1 — Daisy 資料接收端
**負責方**：Daisy
**前置條件**：無
**狀態**：部分完成（`/upload_data` endpoint 存在但 MongoDB 未啟動，回 500）

- [ ] 新增 `POST /upload_data` endpoint：接收 `{ TID, group_id, upfEventNotifs }` 並持久化
- [ ] 建立以 TID 為 key 的資料庫，支援持續累積同一 TID 的多筆資料
- [ ] `publish_task` 收到 TID 後，能從 DB 查出對應 dataset 執行訓練

---

### Phase E2 — NWDAF ADRF 存入端
**負責方**：NWDAF
**前置條件**：無（可與 E1 並行）
**狀態**：✅ 完成

- [x] `pkg/factory/config.go`：新增獨立 `AdrfConfig` struct（`Url`、`StorageThreshold` 預設 1、`FetchBatchSize` 預設 30），掛在 `Configuration.Adrf`
- [x] `config/nwdafcfg.yaml`：補上 `adrf:` block（`configuration:` 內，2-space indent）
- [x] `internal/sbi/consumer/adrf_service.go`（新增）：`AdrfClient` + `StorageRequest`（POST `/data-store-records`），含 connection-pool Transport
- [x] `internal/sbi/processor/adrf_buffer.go`（新增）：per-correlationId buffer；達 `storageThreshold` 時 flush（`AdrfSmfInfo` 直接傳入，無額外 map）
- [x] `internal/sbi/processor/upf_notify.go`：收到 UPF notification 後 marshal → 丟入 `adrfBuffer.add()`
- [x] `internal/context/traffic_data.go`：`AdrfSmfInfo` struct + `StoreAdrfSmfInfo` / `GetAdrfSmfInfo` / `DeleteAdrfSmfInfo`
- [x] `internal/sbi/processor/data_collection.go`：SMF 訂閱成功後呼叫 `StoreAdrfSmfInfo`
- [x] `internal/sbi/processor/eventssubscription.go`：`cleanupDataCollection` 加上 `DeleteAdrfSmfInfo`
- [x] 整合測試：fake_smf_upf + fake_adrf 驗證 `NadrfDataStoreRecord` 結構正確（`dataSub[0].smfDataSub` + `dataNotif.upfEventNotifs`）

---

### Phase E3 — NWDAF ADRF 取資料 + retrain 前置整合
**負責方**：NWDAF
**前置條件**：E1、E2
**狀態**：✅ 完成（2026-03-27）

- [x] `internal/sbi/consumer/adrf_service.go`：`RetrievalSubscribe`（POST `/data-retrieval-subscriptions`，`consTrigNotif: true`，`notifCorrId=TID`）、`RetrievalRequest`（GET `/data-store-records?fetch-correlation-ids=id`，ADRF V0 嚴格限制每次 1 個 ID）、`RetrievalUnsubscribe`（DELETE，404 視為已清理，5xx backoff 重試 3 次）
- [x] `internal/sbi/api_adrf_notify.go`（新增）：`POST /collector/retrieval-notify`，回 `204 No Content`，印 `ids/terminationReq` log
- [x] `internal/sbi/api_collector.go`：在 `getCollectorRoutes()` 加入 `/retrieval-notify` route
- [x] `internal/sbi/processor/processor.go`：`HandleAdrfRetrievalNotify` 委派至 MTLF；加 nil guard
- [x] `internal/mtlf/adrf_retrieval.go`（新增）：`retrainJob` + `runAdrfRetrainWorkflow` + `runFetchLoop`（印 `ids/fetched/uploaded` summary）+ `HandleAdrfRetrievalNotify`（印 `termCount/totalSubs` log）+ `cleanupSubscriptions`
- [x] `internal/mtlf/training.go`：`startRetrainWorkflow`（原 `TriggerRetraining`）+ `submitDaisyTask`（原 `triggerTrainingAsync`，加 `tid` 參數）；提取 `buildNwdafURL`
- [x] `internal/sbi/consumer/daisy_service.go`：`UploadData()`；`TriggerTrainingAsync` 加 `tidOverride` 參數
- [x] `internal/sbi/processor/upf_notify.go`：ADRF 存入前逐 `NotificationItem` 拆分（每筆獨立 ADRF record）
- [x] `pkg/factory/config.go`：`AdrfConfig` 新增 `RetrainWindow`（default 1800s）、`WatchdogTimeout`（default 120s）；`FetchBatchSize` 加 ADRF V0 constraint 說明

**E2E 驗證狀態（2026-03-26 第二次實測，nwdaf_0326.log）**：

| 項目 | 狀態 | 說明 |
|------|------|------|
| ADRF StorageRequest | ✅ | 兩個 SUPI 持續寫入，全程無錯 |
| RetrievalSubscribe | ✅ | 4 次 retrain 均回 201 + subscriptionId |
| `/collector/retrieval-notify` callback | ✅ | 每次 retrain 收到 7 個 callback，回 204 |
| `terminationReq=true` 送達 | ✅ | 每次均收到 2/2（兩 SUPI 各送一次），ADRF 端正常 |
| RetrievalRequest（從 ADRF 取資料） | ✅ | 591/603/615/627 筆（隨歷史資料增長），fetched=ids |
| UploadData 推 Daisy | ✅ | uploaded=fetched，全部成功（Daisy `/upload_data` 接收正常） |
| Daisy 訓練 → callback → 換版 | ❌ | Daisy 回 failure：`model/model.npy` not found（已知 bug，Daisy 側修復中） |

**Callback 收斂模式**（以兩個 SUPI 為例）：
```
ids=100  terminationReq=false  termCount=0/2  ← SUPI 1 batch 1
ids=38   terminationReq=true   termCount=1/2  ← SUPI 1 終止
ids=100  terminationReq=false  termCount=1/2
ids=100  terminationReq=false  termCount=1/2  ← SUPI 2 中間批次
ids=100  terminationReq=false  termCount=1/2
ids=100  terminationReq=false  termCount=1/2
ids=65   terminationReq=true   termCount=2/2  ← SUPI 2 終止，fetchCh close
```
兩 SUPI 資料量差異大（138 vs 465）屬正常，取決於各自的訂閱時間與流量。整個 StorageRequest → UploadData 流程約 3 秒完成。

待完成：Daisy `model.npy` bug 修復後，驗證完整 E2E（Daisy 訓練成功 → callback → NWDAF 換版流程）。

---

### 彙整

| Phase | 負責方 | 前置條件 | 狀態 |
|-------|--------|---------|------|
| E1 | Daisy | 無 | 待實作 |
| E2 | NWDAF | 無 | ✅ 完成 |
| E3 | NWDAF | E1、E2 | ✅ 完成 |

