# Phase 3 Analytics Runtime Migration

Date: 2026-07-10

Status: Planned

Parent plan:

- `AnLF Backend Transition Plan.md`

Previous phase:

- `Phase 2 Model Lifecycle Migration.md`

---

## 1. Purpose

本 phase 的目標，是把目前仍由 `NWDAF/` Go 端主導的 analytics runtime
遷移到 `PyAnLF/`。

這裡的 analytics runtime 不只代表執行模型推論，而是包含：

1. 接收並保存 analytics 所需的 observation
2. 依 subscription runtime 選取 observation sources
3. 執行時間窗對齊、聚合與模型輸入 shaping
4. 根據 reporting parameters 決定何時產生 analytics report
5. 執行 inference 並建立 analytics-domain output
6. 主動把完成的 report callback 給 Go

Phase 3 完成後，Go 不應再以 scheduler 定期呼叫低階 `Predict`。Go 應保留
5GC-facing coordination、資料收集 subscription、外部 notification delivery
以及 3GPP OpenAPI mapping；PyAnLF 則成為 analytics generation 的 owner。

---

## 2. Scope

### 2.1 Included

本 phase 包含：

1. 將 observation source 與 subscription runtime 的 binding contract 定型
2. 讓 Go 把正規化 observation 傳送至 PyAnLF
3. 保留目前 SMF/UPF data collection 的共享與 reference-counting 行為
4. 將 input window、output window、sampling interval、alignment、aggregation
   與 prediction result shaping 移到 PyAnLF
5. 將 periodic report trigger ownership 移到 PyAnLF
6. 新增 PyAnLF 到 Go 的 analytics report callback
7. 將 Go notifier 收斂為 report mapping 與外部 delivery
8. 移除 production `Predict` contract
9. 移除不再有 consumer 的 legacy PyAnLF model/predict endpoints
10. 搬移 analytics config，並補上 observation delivery 所需的 transport config
11. 補齊 Go、Python、API、concurrency 與 cross-repo interaction tests

### 2.2 Explicitly Excluded

本 phase 不包含：

1. `Nnwdaf_AnalyticsInfo_Request`
2. 一次性 synchronous analytics generation API
3. `THRESHOLD` notification method 的首次實作
4. 非 `UE_COMMUNICATION` analytics type 的 runtime 遷移
5. prediction bookkeeping、ground truth matching、accuracy evaluation 與 retrain
   decision migration
6. PyAnLF 直接與 SMF、UPF、NRF 或其他 5GC NF 溝通
7. PyAnLF 直接向外部 NF consumer 發送 3GPP notification
8. persistent message broker 或跨 process restart 的 durable delivery
9. 完整 `Nnwdaf_EventsSubscription` specification coverage 擴充

accuracy workflow 明確保留給 Phase 4。本 phase 只應提供 Phase 4 未來可使用的
乾淨 analytics runtime 與 observation state，不提前搬移 accuracy policy。

---

## 3. Planning Basis

本計畫依據以下來源：

1. `NWDAF/internal/anlf/analytics.go`
2. `NWDAF/internal/anlf/backend.go`
3. `NWDAF/internal/anlf/client/backend.go`
4. `NWDAF/internal/notifier/`
5. `NWDAF/internal/sbi/processor/data_collection.go`
6. `NWDAF/internal/sbi/processor/upf_notify.go`
7. `NWDAF/internal/context/traffic_data.go`
8. `PyAnLF/src/py_anlf/core/runtime_manager.py`
9. `PyAnLF/src/py_anlf/core/predictor.py`
10. `PyAnLF/src/py_anlf/sbi/routers/analytics.py`
11. `nwdaf-docs/specs/TS 23.288/6.1 Procedures for analytics exposure.md`
12. `nwdaf-docs/specs/TS 23.288/6.1.3 Contents of Analytics Exposure.md`
13. `nwdaf-docs/specs/TS 23.288/7.2 Nnwdaf_AnalyticsSubscription Service.md`
14. `nwdaf-docs/specs/yaml/TS29520_Nnwdaf_EventsSubscription.yaml`
15. `nwdaf-docs/specs/yaml/TS29523_Npcf_EventExposure.yaml`
16. `resources/references/free5gc-main/NFs/pcf/internal/sbi/server.go`
17. `resources/references/free5gc-main/NFs/pcf/internal/sbi/api_httpcallback.go`
18. `free5gc-dev-skill/` 的 SBI、config、lifecycle、concurrency 與 testing guidance

PCF 只作為 callback route、handler 到 processor、以及 outbound notification
分層的 free5GC shape exemplar。Go 與 PyAnLF 之間的 API 不是標準 3GPP SBI，
因此 payload 不應假裝是另一個標準 NF service，也不應直接套用不相符的 generated
operation。

---

## 4. Agreed Decisions

本 phase 已確認以下方向：

1. production flow 不再保留 Go 到 PyAnLF 的 `Predict`
2. PyAnLF 主動產生 analytics report，並 callback Go
3. Go 保留對外 `Nnwdaf_EventsSubscription` 與 NF notification delivery
4. `Nnwdaf_AnalyticsInfo_Request` 目前尚未實作，本 phase 不預留 speculative
   request/response contract
5. Go 繼續負責向 SMF/UPF 建立與清理 data collection subscription
6. Go 將 5GC callback payload 正規化後再交給 PyAnLF
7. observation ingestion 採 source-scoped，而不是 subscription-scoped
8. subscription runtime 透過 observation source IDs 引用共享資料來源
9. collection profile 完全相同時才允許複用 source
10. collection requirements 由 PyAnLF 決定並回給 Go
11. source binding 使用獨立 state-sync operation，不重新濫用 model lifecycle apply
12. observation source ID 直接重用現有 correlation ID，但在 contract 中保持 opaque
13. Go 到 PyAnLF observation delivery 使用 bounded in-memory queue、有限重試與
    batch idempotency
14. PyAnLF 到 Go report callback 使用同步 delivery acknowledgement、report ID
    idempotency 與有限重試
15. 第一份 report 只在 reporting information 要求 immediate reporting 時立即觸發；
    否則等待第一個 reporting period
16. 資料或模型不足時，Phase 3 先維持 confidence 0 的既有外部行為
17. Phase 3 只遷移 `UE_COMMUNICATION`
18. 新 contract 完成切換後，移除 legacy model/predict HTTP endpoints
19. PyAnLF 是 Phase 3 後 `UE_COMMUNICATION` 的必要 runtime；若 backend disabled
    或 initial apply 無法完成，不保留 Go-side fallback scheduler

---

## 5. Current State

### 5.1 Current Analytics Generation Flow

目前 periodic scheduler 位於 Go：

```text
Go notification scheduler tick
    -> GenerateUeCommunicationAnalytics
    -> fetchHistoricalData
    -> alignAndZipInMemory
    -> PyAnLF Predict(subscriptionId, historicalData)
    -> Go aggregates predicted steps
    -> Go builds models.UeCommunication
    -> Go sends external notification
```

這代表 PyAnLF 雖然已在 Phase 2 接手 model lifecycle，但仍只被當成低階
inference executor。

### 5.2 Current Go-owned Analytics Logic

目前 Go 仍負責：

1. 根據 subscription 找出 correlation IDs
2. 從 in-memory traffic buckets 取出資料
3. 以 sampling interval 建立時間格
4. 處理不同 UE session anchor 與 late joiner alignment
5. 聚合多個 IP/session 的 observation
6. 套用 input window
7. 建立 backend `historical_data`
8. 計算 prediction target timestamps
9. 彙整多步 UL/DL volume
10. 計算平均 confidence、communication duration 與輸出時間範圍
11. 在模型或資料不足時建立 confidence 0 fallback
12. 保存 prediction records 供 accuracy monitor 使用

其中第 1 至 11 項屬於 Phase 3 遷移範圍；第 12 項留給 Phase 4。

### 5.3 Current Data Collection Reuse

目前 Go 不會為每個 NWDAF subscription 無條件建立新的 SMF subscription。

現有 data collection reuse key 是：

```text
SUPI + SMF endpoint
```

相同 target 與 endpoint 會共用：

1. correlation ID
2. SMF subscription
3. UPF notification stream
4. traffic bucket

`SmfSubscription` 以 `RefCount` 與 `NwdafSubIds` 追蹤使用者；只有最後一個
NWDAF subscription 離開時才向 SMF unsubscribe。

因此 Phase 3 不可把同一份 observation 依 subscription 重複傳送並各自保存，否則會
破壞既有 reuse 的主要價值。

### 5.4 Current Reuse Limitation

現有 reuse key 尚未包含：

1. sampling interval
2. requested measurements
3. source-side filters
4. notification profile

目前 UE Communication 使用 global Go config，所以不同 subscription 通常得到相同
profile。Phase 3 改由 PyAnLF 提供 collection requirements 後，reuse key 必須包含
canonical collection profile，避免不同需求錯誤共用同一條 SMF subscription。

### 5.5 Current PyAnLF State

PyAnLF 目前：

1. 擁有 subscription model runtime
2. 擁有 model sharing、replacement、fallback 與 release
3. 只在 HTTP `Predict` 收到完整 historical data 後執行 inference
4. 不保存 observation source
5. 不擁有 reporting timer
6. 不主動 callback analytics report

---

## 6. Target Architecture

### 6.1 End-to-end Flow

```text
NF consumer
    -> NWDAF Go creates analytics subscription
    -> Go applies PyAnLF subscription runtime
    -> PyAnLF returns collection requirements and runtime revision

Go
    -> creates or reuses compatible SMF/UPF data collection
    -> binds observation source IDs to PyAnLF subscription runtime

SMF/UPF notification
    -> Go validates and normalizes 5GC payload
    -> Go queues one source-scoped observation batch
    -> PyAnLF stores the batch once per observation source

PyAnLF reporting runtime
    -> selects sources bound to subscription
    -> aligns and aggregates observations
    -> runs inference
    -> builds analytics-domain report
    -> callbacks Go anlfServer

Go anlfServer
    -> API handler
    -> AnLF processor
    -> report dispatcher
    -> maps to generated 3GPP models
    -> sends Nnwdaf_EventsSubscription notification to NF consumer
```

### 6.2 NWDAF Go Ownership

Phase 3 後 Go 保留：

1. external `Nnwdaf_EventsSubscription` create/update/delete
2. subscription ID、notification URI 與 notification correlation state
3. SMF/UPF discovery、subscription、callback 與 unsubscribe
4. 5GC payload validation 與 transport-specific normalization
5. observation source creation、compatibility key 與 reference counting
6. bounded observation delivery queue
7. internal PyAnLF report callback server
8. report revision/idempotency validation
9. internal report 到 generated OpenAPI model 的 mapping
10. external NF notification delivery
11. Phase 4 尚未遷移的 ground truth 與 accuracy state

### 6.3 PyAnLF Ownership

Phase 3 後 PyAnLF 接手：

1. collection requirements policy
2. observation source registry 與 source buffer
3. source-to-subscription binding state
4. input/output window config
5. observation alignment、aggregation 與 feature shaping
6. model inference invocation
7. prediction horizon 與 analytics-domain result shaping
8. immediate/periodic report trigger
9. report ID、report sequence 與 callback retry
10. confidence 0 fallback generation

### 6.4 Boundary Rule

PyAnLF 產生的是 internal analytics-domain report，不是直接對 NF consumer 發送的
`NnwdafEventsSubscriptionNotification`。

Go 必須從自己的 subscription context 取得：

1. external `notificationURI`
2. `notifCorrId`
3. active subscription status
4. external serialization requirements

PyAnLF 不得使用 consumer-provided external notification URI 直接繞過 Go。

---

## 7. Internal Contract Design

### 7.1 Contract Summary

Phase 3 最終 contract：

```text
Go -> PyAnLF
PUT    /subscriptions/{subscriptionId}/runtime
PUT    /subscriptions/{subscriptionId}/observation-bindings
DELETE /subscriptions/{subscriptionId}/runtime
POST   /observation-sources/{sourceId}/observations

PyAnLF -> Go
POST   /subscriptions/{subscriptionId}/analytics-reports
```

以下 endpoint 應退出 production contract：

```text
POST /subscriptions/{subscriptionId}/predict
POST /predict
POST /model/load
POST /model/unload
```

PyAnLF 內部仍可保留 Python predictor function，但它不再是跨 process API。

### 7.2 Apply Subscription Runtime Extension

既有 apply request 繼續攜帶完整 subscription context，並新增 Go internal report
callback URI：

```json
{
  "subscription": {
    "subscription_id": "sub-123",
    "notif_corr_id": "consumer-corr-1",
    "evt_req": {
      "notifMethod": "PERIODIC",
      "repPeriod": 30,
      "maxReportNbr": 10,
      "immRep": false
    },
    "event_subscriptions": []
  },
  "report_callback_uri": "http://127.0.0.1:8090/subscriptions/sub-123/analytics-reports"
}
```

external consumer notification URI 不需要交給 PyAnLF。PyAnLF 只需要 Go internal
callback URI。

Apply response 新增：

```json
{
  "subscription_id": "sub-123",
  "runtime_state": "PENDING",
  "result": "PENDING_PROVISION",
  "runtime_revision": 1,
  "collection_requirements": {
    "sampling_interval_seconds": 30,
    "required_measurements": [
      "TOTAL_VOLUME",
      "UL_VOLUME",
      "DL_VOLUME",
      "TOTAL_PACKET_COUNT",
      "UL_PACKET_COUNT",
      "DL_PACKET_COUNT",
      "UL_THROUGHPUT",
      "DL_THROUGHPUT",
      "UL_PACKET_THROUGHPUT",
      "DL_PACKET_THROUGHPUT"
    ]
  }
}
```

`runtime_revision` 由 PyAnLF 產生並擁有。Go 保存目前 active revision，供 callback
驗證。model provision、subscription update 或其他會改變 active runtime 的 apply
可產生新 revision。

若 model replacement 不改變 collection requirements，PyAnLF 應保留既有 source
bindings，不要求 Go 無意義地重新建立 data collection。

### 7.3 Collection Profile Compatibility

Phase 3 採 exact-match reuse。

Go-side canonical compatibility key 至少包含：

```text
target identity
+ SMF endpoint
+ sampling interval
+ normalized required measurement set
+ source-side filter fingerprint, if any
+ notification method/profile required by the source subscription
```

required measurement set 在產生 key 前必須排序與 canonicalize，不能因 array 順序
不同而錯誤建立兩條 source。

本 phase 不實作：

1. 高頻 source 對低頻 subscription 的 downsampling reuse
2. measurement superset/subset compatibility
3. runtime 動態 renegotiation

### 7.4 Observation Source Identifier

Go 繼續使用現有 correlation ID 作為 source identity，但對 PyAnLF 只暴露為：

```text
observation_source_id
```

PyAnLF 必須將其視為 opaque string，不解析 SMF 或 free5GC 語意。

### 7.5 Sync Observation Bindings

在 Go 建立或複用 collection source 後，呼叫：

```http
PUT /subscriptions/{subscriptionId}/observation-bindings
```

request：

```json
{
  "runtime_revision": 1,
  "bindings": [
    {
      "observation_source_id": "corr-17",
      "source": {
        "source_type": "SMF_UPF",
        "supi": "imsi-123456789012345"
      },
      "subscription_scope": {
        "original_group_id": "group-test-001"
      },
      "collection_profile": {
        "sampling_interval_seconds": 30,
        "required_measurements": [
          "TOTAL_VOLUME",
          "UL_VOLUME",
          "DL_VOLUME",
          "TOTAL_PACKET_COUNT",
          "UL_PACKET_COUNT",
          "DL_PACKET_COUNT",
          "UL_THROUGHPUT",
          "DL_THROUGHPUT",
          "UL_PACKET_THROUGHPUT",
          "DL_PACKET_THROUGHPUT"
        ]
      }
    }
  ]
}
```

語意：

1. 這是 replace-style state sync，不是 append
2. request 代表該 revision 的完整 source binding set
3. 相同 source 可被多個 subscription runtime 引用
4. source-level metadata 放在 `source`
5. group membership 等 subscription-specific 意義放在 `subscription_scope`
6. stale revision 回 `409 Conflict`
7. 成功回 `204 No Content`

PyAnLF 應以 binding reference count 管理 source buffer。release 一個 subscription
只解除該 subscription binding；最後一個 binding 消失時才釋放 source buffer。

### 7.6 Observation Ingestion

Go 呼叫：

```http
POST /observation-sources/{sourceId}/observations
```

request：

```json
{
  "batch_id": "5f876e51-c92d-4c70-b74c-91689661ce18",
  "observations": [
    {
      "observed_at": "2026-07-10T12:00:00Z",
      "ipv4_address": "10.60.0.1",
      "supi": "imsi-123456789012345",
      "dnn": "internet",
      "snssai": {
        "sst": 1,
        "sd": "010203"
      },
      "total_volume": 3000,
      "uplink_volume": 1000,
      "downlink_volume": 2000,
      "total_packet_count": 30,
      "uplink_packet_count": 10,
      "downlink_packet_count": 20,
      "uplink_throughput": 8000,
      "downlink_throughput": 16000,
      "uplink_packet_throughput": 8,
      "downlink_packet_throughput": 16
    }
  ]
}
```

Go 的 normalization boundary：

1. 解析 SMF/UPF callback schema
2. 驗證 correlation 與基本 measurement shape
3. 轉換單位到 contract 定義的 base units
4. 補上已知 SUPI、DNN、S-NSSAI 與 session scope
5. 不執行 analytics time-slot alignment
6. 不套用 model input window
7. 不進行跨 session aggregation

PyAnLF 的 ingestion boundary：

1. 驗證 source 已存在且有 active binding
2. 以 `batch_id` 做 bounded idempotency
3. 保存一次 source-scoped data
4. 保留 source 內 per-session identity
5. 根據 backend config 執行 retention

成功保存或已處理過相同 batch 時回 `204 No Content`。

### 7.7 Analytics Report Callback

PyAnLF 呼叫 Go `anlfServer`：

```http
POST /subscriptions/{subscriptionId}/analytics-reports
```

request：

```json
{
  "report_id": "report-00042",
  "report_sequence": 42,
  "runtime_revision": 3,
  "generated_at": "2026-07-10T12:00:00Z",
  "event_notifications": [
    {
      "event": "UE_COMMUNICATION",
      "ue_communications": [
        {
          "communication_duration": 30,
          "timestamp": "2026-07-10T12:00:30Z",
          "traffic_characterization": {
            "dnn": "internet",
            "uplink_volume": 1000,
            "downlink_volume": 2000
          },
          "confidence": 80
        }
      ]
    }
  ]
}
```

internal report schema 應保持 analytics-domain 語意，並可明確 mapping 到 local
OpenAPI YAML 的 `EventNotification`、`UeCommunication` 與
`TrafficCharacterization`。它不攜帶 external notification URI 或 `notifCorrId`。

Go processor 必須：

1. 查詢 path 中的 subscription ID
2. 確認 subscription 存在且 active
3. 驗證 `runtime_revision`
4. 驗證 report event 是 subscription 要求的 event
5. 以 `report_id` 做 in-flight 與 completed deduplication
6. mapping 成 generated `models.NnwdafEventsSubscriptionNotification`
7. 從 Go context 補上 `subscriptionId` 與 `notifCorrId`
8. 使用既有 external `notificationURI` 發送 array-shaped notification body
9. 只有 consumer 回傳成功後才把 report 標記為 delivered

### 7.8 Report Callback Status Semantics

建議 status：

1. `204 No Content`
   - external consumer 已接受 report
   - 或相同 `report_id` 已成功 delivery
2. `400 Bad Request`
   - malformed internal payload
   - unsupported event shape
3. `404 Not Found`
   - subscription 已不存在
4. `409 Conflict`
   - stale runtime revision
   - report sequence 與 active state 衝突
5. `502 Bad Gateway`
   - external consumer 回傳非成功結果
6. `503 Service Unavailable`
   - Go 暫時無法執行 external delivery

PyAnLF 只對 network error、timeout、`502` 與 `503` 進行有限重試。`400`、`404`
與 `409` 不重試同一份 report。

### 7.9 Callback Idempotency And Ordering

每個 runtime revision 的 report 必須有：

1. stable `report_id`
2. monotonic `report_sequence`
3. `runtime_revision`

Go 端需以 per-subscription serialization 或等價機制避免兩個相同 report 同時穿透
到 external consumer。

Go completed report cache 必須 bounded，並具 TTL 或固定窗口。它不是 durable queue；
Go process restart 後不保證保留 deduplication history。這項限制必須在驗證紀錄中
明確說明。

### 7.10 No AnalyticsInfo Contract In Phase 3

目前 `NWDAF/` 沒有 `Nnwdaf_AnalyticsInfo` route、processor 或 service wiring。

因此 Phase 3：

1. 不保留 `Predict` 作為 speculative one-shot API
2. 不先決定未來 `GenerateAnalytics` URL 或 schema
3. 不以尚未支援的外部 service 阻礙目前 subscription-driven architecture 收斂

未來實作 `Nnwdaf_AnalyticsInfo_Request` 時，應新增高階 analytics generation
contract，重用當時的 observation store 與 analytics runtime，而不是恢復低階
historical-data `Predict`。

### 7.11 External Subscription Rejection Semantics

3GPP 不知道 NWDAF 內部的 Go/PyAnLF 拆分，因此沒有
`PYANLF_BACKEND_DISABLED` 之類的標準 application error。

但 `Nnwdaf_EventsSubscription` create operation 支援 generic
`503 Service Unavailable` 與 `ProblemDetails`。Phase 3 採以下 policy：

1. PyAnLF disabled、endpoint 缺失或 initial runtime apply 無法連線時，
   `UE_COMMUNICATION` 視為目前不可服務
2. 若 request 中所有可接受 events 都依賴 PyAnLF，Go 回
   `503 Service Unavailable`，不建立 subscription
3. `ProblemDetails` 至少包含 `status`、`title` 與不暴露內部敏感細節的 `detail`
4. `cause` 在此情況不是 TS 29.520 強制欄位，不應偽造不存在的 3GPP standard cause
5. 若同一 request 還有其他可接受 events，Go 可建立 subscription，並在
   `failEventReports` 對 `UE_COMMUNICATION` 使用 `NwdafFailureCode_OTHER`
6. 不使用 `UNAVAILABLE_DATA`，因為該 code 的標準語意是過去 statistics 所需資料
   不存在，不是 backend service unavailable

整份拒絕範例：

```http
HTTP/1.1 503 Service Unavailable
Content-Type: application/problem+json
```

```json
{
  "status": 503,
  "title": "Service Unavailable",
  "detail": "UE_COMMUNICATION analytics runtime is unavailable"
}
```

部分 event failure 範例：

```json
{
  "failEventReports": [
    {
      "event": "UE_COMMUNICATION",
      "failureCode": "OTHER"
    }
  ]
}
```

若未來 NWDAF 完成 NRF analytics capability advertisement，PyAnLF disabled 時也不應
宣告支援 `UE_COMMUNICATION`，讓 consumer 優先在 discovery 階段避開不可服務的
instance。這項 NRF profile alignment 不要求在 Phase 3 內首次完成；若目前尚無對應
registration capability，仍以 create-time validation 為必要保證。

---

## 8. Lifecycle Semantics

### 8.1 Subscription Create

建議順序：

1. Go 驗證 external subscription request
2. Go 確認需要 PyAnLF 的 event 目前可服務
3. Go initial apply PyAnLF runtime
4. PyAnLF 回 collection requirements 與 revision
5. initial apply 失敗時，依 7.11 回 `503` 或 event-level failure，不保存失敗 event
6. Go 保存已接受的 external subscription
7. Go 建立或複用 compatible sources
8. Go sync complete observation bindings
9. Go 開始把 source observation batches 送給 PyAnLF
10. PyAnLF 啟動 reporting runtime

若 `immRep=true`，PyAnLF 可在 runtime 與 binding ready 後立即產生第一份 report。
若資料或模型不足，依已確認方向產生 confidence 0 fallback。

Phase 3 不將第一份 report inline 放入 subscribe HTTP response；它仍透過 report
callback 與 external notification path 提供。完整 immediate subscribe response
alignment 不在本 phase 範圍。

Go 不得先對 consumer 回 `201 Created`，再因 initial PyAnLF apply 失敗而靜默留下
一個永遠不會產生 report 的 subscription。若在 initial apply 成功後、保存 external
subscription 前發生本地失敗，Go 必須 best-effort release 已建立的 backend runtime。

### 8.2 Subscription Update

update 必須採 reconcile，而不是先破壞所有舊狀態再盲目重建：

1. Go apply 新 subscription runtime
2. PyAnLF 產生新 revision
3. Go 比較舊、新 collection profile 與 target set
4. 相容 source 保留並重用
5. 新 source 建立 reference
6. Go sync 新 revision 的完整 binding set
7. PyAnLF 原子切換 reporting runtime
8. Go 才釋放不再使用的舊 source references

舊 revision 產生中的 callback 必須被取消；已抵達 Go 的舊 report 回 `409`，不得向
external consumer 發送。

### 8.3 Subscription Release

release 順序：

1. Go 將 external subscription 標記為不再接受 report
2. Go 呼叫 `ReleaseSubscriptionRuntime`
3. PyAnLF 停止 timer、callback retry 與 in-flight generation
4. PyAnLF 解除 source bindings
5. Go 釋放 SMF source references
6. 最後一個 reference 離開時，Go 向 SMF unsubscribe

release 應保持 idempotent。

### 8.4 Model Replacement

Phase 2 model replacement semantics 繼續有效：

1. candidate 成功才切換
2. candidate 失敗保留舊 model
3. reporting runtime 不因 replacement failure 被無條件停止
4. active revision 改變時，callback 必須使用新 revision
5. source bindings 在 collection requirements 相容時延續

### 8.5 Reporting Limits

PyAnLF 擁有：

1. `repPeriod`
2. `immRep`
3. `maxReportNbr`
4. `monDur`
5. report sequence

Go 保留 subscription 中的 reporting information，作為 external correlation、callback
validation 與防禦性檢查，不再用自己的 ticker 產生 report。

`THRESHOLD` 目前仍維持既有 `501 Not Implemented` 行為，不在本 phase 偷渡實作。

### 8.6 Backend Availability After Acceptance

7.11 定義的是 create-time availability。subscription 已接受後，PyAnLF 暫時失聯時：

1. Go 不恢復舊 scheduler 或本地 analytics generation
2. observation delivery 依 9 節執行 bounded retry
3. PyAnLF 恢復後繼續依仍有效的 runtime/reporting state 工作
4. 若 runtime 已無法恢復，應透過明確 termination/error procedure 收斂，而不是
   永久靜默；該完整 external notification policy若超出目前 supported flow，需在
   實作前依現有 spec model 與 NWDAF 能力確認，不得臨時發明 callback payload

Phase 3 的最低完成要求是 initial create 不接受明知不可用的 runtime，以及 runtime
失聯期間不重新啟用 Go analytics fallback。完整跨 process durable recovery 不在本 phase。

---

## 9. Observation Delivery Reliability

### 9.1 Go-owned Queue

UPF callback handler 不應直接同步等待 PyAnLF network call。

Go 應建立 app-owned bounded queue/worker：

1. enqueue normalized source batch
2. 保持同 source 的 delivery ordering
3. 使用 app cancel context
4. HTTP call 有 timeout
5. 失敗執行有限次 retry
6. shutdown 停止接收新 batch 並在 bounded time 內結束 worker
7. queue 滿時記錄 source ID、drop count 與原因，但不記錄敏感完整 payload

### 9.2 Delivery Guarantee

Phase 3 的保證是：

```text
at-least-once while the Go process and bounded queue remain alive
```

不保證：

1. Go process restart 後恢復尚未送出的 batch
2. disk durability
3. exactly-once transport

PyAnLF 以 `batch_id` 把 at-least-once transport 收斂成 idempotent storage。

### 9.3 Queue Configuration

Go config 建議在 `anlfBackend` 下新增 transport-oriented 設定：

```yaml
anlfBackend:
  enabled: true
  endpoint: http://127.0.0.1:9090
  observationDelivery:
    queueCapacity: 1024
    requestTimeout: 5
    maxRetries: 3
    retryInterval: 1
```

這些欄位屬於 Go 到 backend 的 transport policy，不是 analytics business config。

PyAnLF config 應有對應 source retention 與 callback transport 設定，但不得重複保存
Go queue policy。

---

## 10. Config Migration

### 10.1 Move To PyAnLF

目前 Go `analytics.ueCommunication` 中屬於 analytics runtime 的欄位應移到
PyAnLF，例如：

1. `samplingInterval`
2. `inputWindow`
3. `outputWindow`
4. alignment/lookback policy
5. PyAnLF source retention/window capacity

PyAnLF 應從這些設定產生 `collection_requirements`。

### 10.2 Remain In Go

Go 保留：

1. SMF endpoints、subscription duration 與 notification URIs
2. AnLF backend endpoint
3. anlfServer binding/register address
4. observation delivery queue、timeout 與 retry config
5. Phase 4 尚未遷移前，accuracy/ground-truth 所需的 transitional retention config

Go 不應再保留一份獨立的 inference input/output window policy。

### 10.3 Transitional Accuracy Constraint

目前 Go accuracy monitor 仍使用 sampling interval 與本地 traffic data。Phase 3 不得
直接刪除這條依賴後讓 accuracy workflow 失效。

建議做法：

1. Go 保存 PyAnLF 回傳的 active collection profile
2. accuracy monitor 從 active source/subscription runtime state 取得 sampling interval
3. Go-side traffic retention 明確標記為 Phase 4 transitional ground-truth support
4. 不再從 Go analytics business config 推導 inference parameters

---

## 11. Planned Implementation Areas

### 11.1 NWDAF Repository

預計影響：

1. `internal/anlf/backend.go`
   - 移除 `Predict`
   - 新增 collection requirements、runtime revision、bindings、observation 與 report types
2. `internal/anlf/client/backend.go`
   - 新增 sync bindings 與 observation ingestion calls
   - 移除 subscription predict call
3. `internal/anlf/server.go`
   - 註冊 analytics report callback route
   - 擴充 narrow processor interface
4. `internal/anlf/api_analytics_report.go`
   - path/body parsing、HTTP-level validation 與 response mapping
5. `internal/anlf/processor/analytics_report.go`
   - subscription/revision/event/idempotency validation
   - 呼叫 injected report dispatcher
6. `internal/notifier/`
   - 將 scheduler-owned generation 拆除
   - 保留並抽出 external report dispatcher/sender
7. `internal/sbi/processor/eventssubscription.go`
   - 移除 Go periodic analytics scheduler ownership
   - create/update/release 改為 runtime、collection、binding reconcile
8. `internal/sbi/processor/data_collection.go`
   - 使用 collection requirements
   - 擴充 exact-match collection profile key
9. `internal/sbi/processor/upf_notify.go`
   - 建立 normalized observation batch 並 enqueue
   - 保留 Phase 4 ground truth 與 ADRF 既有路徑
10. `internal/context/`
    - 保存 active runtime revision、collection profile 與 source bindings
    - bounded report dedup state
11. new observation delivery worker package
    - queue、retry、ordering、shutdown
12. `pkg/factory/config.go` 與 `config/nwdafcfg.yaml`
    - analytics config migration
    - transport config 與 validation
13. `pkg/service/init.go`
    - constructor injection、worker lifecycle 與 shutdown wiring

具體 package 名稱應在實作前依現有 dependency direction 決定。不得為了方便讓
`anlf` API handler 直接依賴 broad SBI processor，也不得建立 package import cycle。

### 11.2 PyAnLF Repository

預計影響：

1. `src/py_anlf/models.py`
   - collection requirements、binding、observation batch 與 report schemas
2. `src/py_anlf/core/runtime_manager.py`
   - runtime revision 與 reporting lifecycle coordination
3. new observation source/store module
   - shared source buffers、binding reference、batch dedup 與 retention
4. new analytics runtime module
   - alignment、aggregation、window selection、inference 與 fallback
5. new reporting scheduler module
   - immediate/periodic trigger、limits、cancellation 與 sequence
6. new Go callback client
   - report delivery、timeout、retry 與 status handling
7. `src/py_anlf/sbi/routers/analytics.py`
   - bindings 與 observations routes
   - 移除 production predict route
8. `src/py_anlf/sbi/server.py`
   - 以 `app.state` 或現有 DI 方式組裝 runtime dependencies
9. `config/config.yaml`
   - analytics runtime、source retention 與 callback client settings
10. README 與 manual interaction client
    - 改成 apply、bind、ingest、report callback flow

### 11.3 Free5GC Shape Alignment

Go inbound report callback 應維持：

```text
anlfServer route
    -> api handler
    -> anlf processor
    -> injected dispatcher
    -> external notifier/consumer boundary
```

PCF callback files提供 handler/processor boundary 的 shape evidence；真正 internal
payload、revision、idempotency 與 source binding 是本 NWDAF/PyAnLF contract 的
domain-specific design。

---

## 12. Implementation Order

建議順序：

1. 在 PyAnLF 新增 schema、source store、bindings 與 collection requirements
2. 在 PyAnLF 新增 analytics runtime 與 report callback client
3. 保留舊 predict route，先完成 Python tests
4. 在 Go 新增 contract types、bindings client 與 observation delivery worker
5. 擴充 Go collection profile reuse key
6. 在 Go anlfServer 新增 report callback API、processor 與 dispatcher
7. 切換 subscription create/update/delete 與 UPF notification flow
8. 停止建立 Go analytics scheduler
9. 移除 Go `Predict` interface、client、analytics shaping 與相關 tests
10. 搬移 config 並修正 Phase 4 transitional accuracy dependencies
11. 執行 cross-repo interaction tests
12. 確認無 consumer 後移除 PyAnLF legacy HTTP endpoints
13. 更新 README、plan implementation record 與 verification result

這個順序只用於降低開發期間的跨 repo breakage。Phase 3 final state 不保留長期雙軌
production flow。

---

## 13. Verification Plan

### 13.1 NWDAF Unit Tests

至少覆蓋：

1. 同 target、endpoint、profile 只建立一次 SMF subscription
2. 不同 sampling interval 建立不同 observation source
3. required measurements 不同時不誤用既有 source
4. source reference add/release 與 last-reference unsubscribe
5. normalized observation 的 timestamps、units、scope 與 measurement fields
6. queue ordering、timeout、retry、overflow 與 shutdown
7. batch ID 在 retry 時保持不變
8. binding client path、payload 與 status handling
9. report callback malformed body
10. unknown/inactive subscription
11. stale runtime revision
12. duplicate/in-flight report 不重複 external delivery
13. report 到 generated OpenAPI model 的完整 mapping
14. zero-value volume/confidence 的 serialization
15. external consumer `204`、非成功 response 與 network failure
16. create/update/release 不再啟動 Go analytics scheduler
17. backend disabled、endpoint missing 與 initial apply failure 回 `503`
18. mixed-event request 使用 `failEventReports/OTHER`，且不使用 `UNAVAILABLE_DATA`
19. initial apply 成功後本地 create 失敗會 best-effort release backend runtime
20. config valid、missing、invalid 與 defaults

### 13.2 PyAnLF Unit Tests

至少覆蓋：

1. collection requirements 由 analytics config 產生
2. source binding replace semantics
3. 多個 subscription 共用同一 source buffer
4. release 一個 subscription 不刪除仍在使用的 source
5. last binding release 清除 source
6. duplicate batch ID 不重複保存
7. source retention bound
8. per-session alignment、late joiner 與 aggregation
9. input window truncation/padding policy
10. output window 與 prediction timestamps
11. model ready inference path
12. no model/no data confidence 0 fallback
13. `immRep`、first periodic tick、`maxReportNbr` 與 `monDur`
14. update/release 取消舊 timer 與 in-flight callback
15. report ID、sequence 與 runtime revision
16. callback retryable/non-retryable status

### 13.3 API Tests

PyAnLF API tests：

1. apply response 包含 revision 與 collection requirements
2. binding sync success/stale revision/unknown subscription
3. observation ingestion success/duplicate/unknown source/invalid payload
4. production predict route 已移除

Go API tests：

1. analytics report callback success
2. invalid report body
3. stale revision
4. duplicate report
5. external delivery failure mapping

### 13.4 Cross-repo Interaction Tests

至少建立以下 local interaction：

1. 啟動真實 PyAnLF 與 Go callback test server
2. apply 一個 UE Communication subscription
3. 取得 collection requirements
4. bind source
5. ingest observations
6. 等待 PyAnLF 主動 callback report
7. 驗證 Go mapping 後的 external notification body
8. 驗證相同 report retry 不重複 delivery

共享 source case：

1. 建立兩個 subscription
2. 綁定相同 observation source
3. 只 ingest 一份 batch
4. 兩個 subscription 各自依 reporting state 產生 report
5. release 第一個 subscription 後 source 仍存在
6. release最後一個 subscription 後 source 被清除

### 13.5 Repository Verification

`NWDAF/`：

```bash
make test
make build
make lint
```

`PyAnLF/`：

```bash
uv run --group dev pytest -q
```

若環境允許，再執行 local live API interaction。完整 SMF、UPF、MTLF 與 external
NF consumer 的 5GC end-to-end test 若無法執行，必須明確記為未驗證，不能以
unit/API tests 代替宣稱。

---

## 14. Completion Criteria

Phase 3 只有在以下條件滿足時才算完成：

1. Go production flow 不再呼叫 `Predict`
2. PyAnLF 擁有 observation buffer、alignment、window 與 inference orchestration
3. PyAnLF 擁有 supported periodic reporting trigger
4. PyAnLF report 會 callback Go anlfServer
5. Go 會驗證 revision/idempotency 並轉送 external consumer
6. data collection reuse 已擴充為 profile-compatible source reuse
7. 同一 source observation 不會因多個 subscription 重複傳送與保存
8. observation delivery 有 bounded queue、retry 與 batch dedup
9. analytics business config 已移到 PyAnLF
10. Phase 4 accuracy workflow 沒有因 config/state migration 失效
11. create/update/release lifecycle 可取消 timer、retry 與 source references
12. legacy cross-process predict/model endpoints 已移除或有明確、經確認的 external
    consumer 阻擋證據
13. backend disabled/unavailable 時不建立無法服務的 `UE_COMMUNICATION` subscription
14. Go、Python 與 cross-repo tests 通過
15. 文件記錄實際完成內容、偏差與未執行驗證

---

## 15. Risks And Mitigations

### 15.1 Collection Requirement Bootstrap

風險：Go 必須先問 PyAnLF requirements，才能建立 source；source 建立後又必須回頭
sync bindings。

處理：明確採 apply、collection reconcile、binding sync 三段式 lifecycle，不把隱含
side effect 藏進單一 API。

### 15.2 Shared Source Compatibility

風險：不同 sampling profile 錯誤共用同一 SMF subscription。

處理：Phase 3 只允許 canonical profile exact match，並增加不同 profile tests。

### 15.3 Duplicate Observation

風險：Go retry 造成相同 measurement 重複進入模型窗口。

處理：stable `batch_id`、PyAnLF bounded dedup 與 retry tests。

### 15.4 Duplicate External Report

風險：external consumer 已收到 report，但 Go 到 PyAnLF 的 response 丟失，造成 PyAnLF
重試並重複通知 consumer。

處理：Go 在成功 external delivery 後記錄 `report_id`，相同 report 直接回 `204`。

### 15.5 Stale Report After Update

風險：舊 timer 或 in-flight inference 在 subscription update 後送出過期結果。

處理：runtime revision、PyAnLF cancellation、Go stale revision rejection。

### 15.6 Queue Data Loss

風險：bounded queue overflow 或 process restart 會遺失 observation。

處理：明確 log/drop observability、有限 retry、文件揭露保證；durable broker/ADRF
recovery 不在本 phase 偷渡。

### 15.7 Accuracy Regression

風險：搬走 Go analytics config 或 input shaping 後，Phase 4 尚未遷移的 ground truth
matching 失去 sampling context。

處理：保存 active collection profile，讓 accuracy monitor 讀 runtime state，而不是直接
刪除必要資訊。

### 15.8 Package Dependency Pollution

風險：anlf callback processor 為了送 external notification 直接依賴 broad SBI
processor，造成 import cycle 或責任污染。

處理：注入 narrow report dispatcher interface；API 只處理 HTTP，processor 做 procedure，
dispatcher 做 external mapping/delivery。

### 15.9 Accepted Subscription Without Runtime

風險：Go 先回 `201 Created`，後續 asynchronous apply 才發現 PyAnLF 不可用，留下
無法產生 report 的 subscription。

處理：create path 在 commit external subscription 前同步取得 initial apply 結果；
全數依賴不可用 backend 時回標準 `503 ProblemDetails`，mixed-event request 則使用
`failEventReports/OTHER`。不以 Go fallback scheduler 掩蓋 backend availability 問題。

---

## 16. Relation To Phase 4

Phase 3 完成後仍留在 Go 的主要 AnLF 業務邏輯應只剩：

1. prediction record correlation
2. ground truth lookup
3. accuracy metric calculation
4. accuracy monitor scheduling
5. retrain input/report preparation
6. MTLF accuracy workflow coordination

Phase 4 應基於 Phase 3 的 observation source、analytics report 與 runtime revision
設計，重新決定哪些 prediction/ground-truth state 下沉到 PyAnLF。

Phase 3 不應因為預期 Phase 4 會再搬移，而留下兩套 analytics generation path 或
繼續保留低階 `Predict` workaround。
