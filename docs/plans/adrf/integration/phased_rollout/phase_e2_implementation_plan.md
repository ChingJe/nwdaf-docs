# Phase E2 實作報告：NWDAF ADRF 存入端

**狀態**：✅ 完成（commit `5dbbb13`）
**日期**：2026-03-25

---

## 概述

Phase E2 實作 NWDAF 將 UPF 資料存入 ADRF 的功能（Nadrf_DataManagement StorageRequest，TS 29.575）。

每次收到 UPF notification，processor 內的 ADRF buffer 依 correlationId 分別累積，達到 threshold 後呼叫一次 `StorageRequest` 送入 ADRF。預設 `threshold=1`（逐筆立即送出）。

---

## 資料流

```
UPF notification
  → POST /collector/upf-notify
  → Processor.HandleUpfNotification()
  → ctx.GetAdrfSmfInfo(correlationId)        ← AdrfSmfInfo（訂閱時存入）
  → json.Marshal(notif) → notifJSON
  → p.adrfBuffer.add(info, notifJSON)
    → pending[correlationId] 累積至 threshold
    → flushOne(correlationId, info)
      → consumer.AdrfClient.StorageRequest(info, notifJSONs)
        → POST /nadrf-datamanagement/v1/data-store-records → ADRF
        → 回 201 + Location header → storeTransId
```

---

## 異動檔案（14 個，988 行）

| 檔案 | 動作 | +/- |
|------|------|-----|
| `pkg/factory/config.go` | 新增 `AdrfConfig` struct，加入 `Configuration` | +29 |
| `config/nwdafcfg.yaml` | 補 `adrf:` 區塊（`configuration:` 內） | +6 |
| `internal/context/context.go` | `NWDAFContext` 加 `adrfSmfInfos sync.Map` | +4 |
| `internal/context/traffic_data.go` | 新增 `AdrfSmfInfo` + 3 個 context methods | +35 |
| `internal/sbi/processor/data_collection.go` | SMF 訂閱成功後存入 `AdrfSmfInfo` | +10 |
| `internal/sbi/consumer/adrf_service.go` | 新建：型別定義 + `AdrfClient` + `StorageRequest` | +115 |
| `internal/sbi/consumer/consumer.go` | `Consumer` 加 `*AdrfClient`；`NewConsumer` 條件初始化 | +7 |
| `internal/sbi/processor/adrf_buffer.go` | 新建：`adrfBuffer`（per-correlationId） | +60 |
| `internal/sbi/processor/processor.go` | `Processor` 加 `adrfBuffer`；`NewProcessor` 初始化 | +17 |
| `internal/sbi/processor/upf_notify.go` | `HandleUpfNotification` 結尾呼叫 buffer | +12 |
| `internal/sbi/processor/eventssubscription.go` | `cleanupDataCollection` 呼叫 `DeleteAdrfSmfInfo` | +3 |
| `test/fake_adrf/fake_adrf_server.py` | 新建 fake ADRF server（FastAPI） | +409 |
| `test/fake_adrf/pyproject.toml` | fake_adrf 依賴設定 | +10 |
| `test/fake_adrf/uv.lock` | lockfile | +276 |

---

## 各檔案實際實作

### 1. `pkg/factory/config.go`

新增獨立 `AdrfConfig` struct（與 `SmfConfig`、`MlServiceConfig`、`MtlfConfig` 同層級）：

```go
type AdrfConfig struct {
    Url              string `yaml:"url,omitempty"`
    StorageThreshold int    `yaml:"storageThreshold,omitempty"` // default: 1
    FetchBatchSize   int    `yaml:"fetchBatchSize,omitempty"`   // default: 30 (Phase E3)
}

func (a *AdrfConfig) AdrfEnabled() bool {
    return a != nil && a.Url != ""
}

func (a *AdrfConfig) StorageThresholdOrDefault() int {
    if a == nil || a.StorageThreshold <= 0 {
        return 1
    }
    return a.StorageThreshold
}

func (a *AdrfConfig) FetchBatchSizeOrDefault() int {
    if a == nil || a.FetchBatchSize <= 0 {
        return 30
    }
    return a.FetchBatchSize
}
```

`Configuration` struct 加欄位：

```go
Adrf *AdrfConfig `yaml:"adrf,omitempty"`
```

---

### 2. `config/nwdafcfg.yaml`

在 `configuration:` block 內（2-space indent，與 `smf:`、`mtlf:` 同層）新增：

```yaml
  # ADRF (Analytics Data Repository Function) — TS 29.575
  adrf:
    url: http://127.0.0.1:9888
    storageThreshold: 1   # per-UE: flush every N notifications from the same UE; 1 = immediate
    fetchBatchSize: 30    # max fetchCorrIds per retrieval request
```

**陷阱**：`adrf:` 必須在 `configuration:` 內縮排。若誤置於頂層，`factory.NwdafConfig.Configuration.Adrf` 永遠為 nil，ADRF client 不會初始化。

---

### 3. `internal/context/context.go`

`NWDAFContext` struct 加欄位：

```go
adrfSmfInfos sync.Map // correlationId → *AdrfSmfInfo
```

---

### 4. `internal/context/traffic_data.go`

新增 `AdrfSmfInfo` struct：

```go
// AdrfSmfInfo records the SMF subscription parameters needed to reconstruct
// smfDataSub when storing UPF data to ADRF (TS 29.575 NadrfDataStoreRecord).
// Populated at SMF subscription time; keyed by correlationId.
//
// SmfSubscription serves routing/lifecycle purposes only.
// Storing here captures the actual parameters used at subscription time,
// rather than re-deriving from config which may change at runtime.
type AdrfSmfInfo struct {
    Supi        string // Target SUPI
    NotifId     string // correlationId (SMF notifId)
    NotifUri    string // SMF notification URI
    UpfNotifUri string // UPF bundled notification URI (for reconstructing eventSubs)
    NotifMethod string // e.g. "PERIODIC"
    RepPeriod   int32  // report period in seconds
}
```

三個 context methods：

```go
func (c *NWDAFContext) StoreAdrfSmfInfo(correlationId string, info *AdrfSmfInfo)
func (c *NWDAFContext) GetAdrfSmfInfo(correlationId string) *AdrfSmfInfo  // nil if not found
func (c *NWDAFContext) DeleteAdrfSmfInfo(correlationId string)
```

`DeleteAdrfSmfInfo` 由 `cleanupDataCollection` 在訂閱釋放時呼叫，防止 memory leak。

---

### 5. `internal/sbi/processor/data_collection.go`

在 `triggerTargetDataCollection` 中，`sub.Unlock()` 之後加入：

```go
ctx.StoreAdrfSmfInfo(correlationId, &nwdaf_context.AdrfSmfInfo{
    Supi:        target.Supi,
    NotifId:     correlationId,
    NotifUri:    smfNotifUri,
    UpfNotifUri: upfNotifUri,
    NotifMethod: "PERIODIC",
    RepPeriod:   smfRepPeriod,
})
```

---

### 6. `internal/sbi/consumer/adrf_service.go`（新建）

#### 型別定義（TS 29.575 NadrfDataStoreRecord，dataSub + dataNotif 路徑）

```go
// AdrfDataSubscription wraps a single DataSubscription element (TS 29.575).
// Using smfDataSub path: ExtendedNsmfEventExposure maps to NsmfEventExposure (TS 29.508).
type AdrfDataSubscription struct {
    SmfDataSub *ExtendedNsmfEventExposure `json:"smfDataSub,omitempty"`
}

// AdrfDataNotification holds the upfEventNotifs array (TS 29.575 DataNotification).
// Each element is a serialized NotificationData (TS 29.564) passed as raw JSON
// to avoid importing processor types.
type AdrfDataNotification struct {
    UpfEventNotifs []json.RawMessage `json:"upfEventNotifs"`
}

// NadrfDataStoreRecord is the request body for StorageRequest (TS 29.575).
// Uses the dataSub + dataNotif path for raw UPF data.
type NadrfDataStoreRecord struct {
    DataSub   []AdrfDataSubscription `json:"dataSub"`
    DataNotif *AdrfDataNotification  `json:"dataNotif,omitempty"`
}
```

**設計決策**：`smfDataSub` 使用既有的 `ExtendedNsmfEventExposure`（`consumer/models.go`），其 JSON tags 已完整對應 TS 29.508 `NsmfEventExposure`（`supi`、`notifId`、`notifUri`、`eventSubs`、`notifMethod`、`repPeriod`），不另建新 struct。

`upfEventNotifs` 使用 `[]json.RawMessage`：processor 的 `json.Marshal(UpfNotificationData)` 產生的 JSON 已符合 TS 29.564 `NotificationData` 格式，不需在 consumer 重複定義型別，也避免 consumer → processor 的反向依賴。

#### `AdrfClient`

```go
const (
    AdrfDataStoreRecordsPath = "/nadrf-datamanagement/v1/data-store-records"
    adrfStorageTimeout       = 10 * time.Second  // unexported
)

type AdrfClient struct {
    endpoint   string
    httpClient *http.Client
}

func NewAdrfClient(endpoint string) *AdrfClient {
    return &AdrfClient{
        endpoint: endpoint,
        httpClient: &http.Client{
            Timeout: adrfStorageTimeout,
            Transport: &http.Transport{
                MaxIdleConns:        50,
                MaxIdleConnsPerHost: 10,
                IdleConnTimeout:     90 * time.Second,
            },
        },
    }
}
```

#### `StorageRequest` 實作

```go
func (c *AdrfClient) StorageRequest(
    info *nwdaf_context.AdrfSmfInfo,
    upfNotifJSONs []json.RawMessage,
) (string, error) {
    smfDataSub := &ExtendedNsmfEventExposure{
        Supi:        info.Supi,
        NotifId:     info.NotifId,
        NotifUri:    info.NotifUri,
        NotifMethod: info.NotifMethod,
        RepPeriod:   info.RepPeriod,
        EventSubs:   BuildUpfEventSubs(info.UpfNotifUri, true, true),
    }

    record := NadrfDataStoreRecord{
        DataSub:   []AdrfDataSubscription{{SmfDataSub: smfDataSub}},
        DataNotif: &AdrfDataNotification{UpfEventNotifs: upfNotifJSONs},
    }

    // POST /nadrf-datamanagement/v1/data-store-records
    // 201 → storeTransId = path.Base(Location header)
}
```

**lint 修正**：
- `resp.Body.Close()` 使用 `if closeErr := ...; closeErr != nil` pattern（匹配現有 consumer 風格）
- HTTP request 使用 `http.NewRequestWithContext(context.Background(), ...)` + `httpClient.Do(req)`（避免 `noctx` lint 錯誤）

---

### 7. `internal/sbi/consumer/consumer.go`

`Consumer` struct 加欄位：

```go
type Consumer struct {
    *NsmfService
    *NmtlfService
    Adrf *AdrfClient // nil if ADRF not configured
}
```

`NewConsumer` 條件初始化（`adrf.url` 有設才建立 client）：

```go
if factory.NwdafConfig.Configuration.Adrf.AdrfEnabled() {
    c.Adrf = NewAdrfClient(factory.NwdafConfig.Configuration.Adrf.Url)
    consumerLog.Infof("ADRF client initialized: url=%s",
        factory.NwdafConfig.Configuration.Adrf.Url)
}
```

---

### 8. `internal/sbi/processor/adrf_buffer.go`（新建）

Per-correlationId sub-buffer，threshold 達到時合併送出：

```go
type adrfBuffer struct {
    mu        sync.Mutex
    pending   map[string][]json.RawMessage // correlationId → accumulated notifJSONs
    threshold int
    client    *consumer.AdrfClient
}
```

**設計決策**：無 `infos` map。計畫中曾設計 `infos map[string]*AdrfSmfInfo` 儲存對應資訊，但發現此 map 的 key 永遠不刪除（`flushOne` 只刪 `pending`，不刪 `infos`），形成 memory leak。解決方案：`info` 在 `add()` 呼叫時直接傳入，並由 `add` 轉傳給 `flushOne`，不存在 buffer 內。

```go
func (b *adrfBuffer) add(info *nwdaf_context.AdrfSmfInfo, notifJSON json.RawMessage) {
    b.mu.Lock()
    defer b.mu.Unlock()

    correlationId := info.NotifId
    b.pending[correlationId] = append(b.pending[correlationId], notifJSON)

    if len(b.pending[correlationId]) >= b.threshold {
        b.flushOne(correlationId, info)
    }
}

func (b *adrfBuffer) flushOne(correlationId string, info *nwdaf_context.AdrfSmfInfo) {
    // Must be called under b.mu
    notifJSONs := b.pending[correlationId]

    storeTransId, err := b.client.StorageRequest(info, notifJSONs)
    if err != nil {
        logger.ProcLog.Warnf("ADRF StorageRequest failed for correlationId=%s: %v", correlationId, err)
    } else {
        logger.ProcLog.Infof("ADRF stored: storeTransId=%s supi=%s count=%d",
            storeTransId, info.Supi, len(notifJSONs))
    }

    delete(b.pending, correlationId)
}
```

threshold=1（預設）：每次 `add` 立即 `flushOne`，退化為逐筆存入。

---

### 9. `internal/sbi/processor/upf_notify.go`

`HandleUpfNotification` 的 `return nil` 前加入：

```go
if p.adrfBuffer != nil {
    if info := ctx.GetAdrfSmfInfo(notif.CorrelationId); info != nil {
        if notifJSON, err := json.Marshal(notif); err == nil {
            p.adrfBuffer.add(info, notifJSON)
        } else {
            logger.ProcLog.Warnf("Failed to marshal UPF notification for ADRF: %v", err)
        }
    }
}
```

`info == nil` 時靜默略過（此 correlationId 尚無 AdrfSmfInfo，非錯誤）。

---

### 10. `internal/sbi/processor/processor.go`

`Processor` struct 加欄位：

```go
adrfBuffer *adrfBuffer
```

`NewProcessor` 中，Wire 1/2/3 之後加入（local var 避免 `StorageThresholdOrDefault()` 被呼叫兩次）：

```go
if adrf := p.nwdaf.Consumer().Adrf; adrf != nil {
    threshold := factory.NwdafConfig.Configuration.Adrf.StorageThresholdOrDefault()
    p.adrfBuffer = newAdrfBuffer(threshold, adrf)
    logger.ProcLog.Infof("ADRF buffer initialized: threshold=%d", threshold)
}
```

---

### 11. `internal/sbi/processor/eventssubscription.go`

`cleanupDataCollection` 加入 cleanup（與 `DeleteTrafficBucket` 並列）：

```go
ctx.DeleteAdrfSmfInfo(res.CorrelationId)
```

---

### 12. `test/fake_adrf/fake_adrf_server.py`（新建）

FastAPI server 實作 Nadrf_DataManagement（TS 29.575）：

| endpoint | 說明 |
|----------|------|
| `POST /nadrf-datamanagement/v1/data-store-records` | StorageRequest：存入 record，回 201 + Location header（storeTransId）|
| `GET /nadrf-datamanagement/v1/data-store-records` | RetrievalRequest：按 `fetch-correlation-ids` 查詢，merge 後回 200 |
| `POST /nadrf-datamanagement/v1/data-retrieval-subscriptions` | RetrievalSubscribe：建訂閱，非同步送 fetchInstruct callback（含 storeTransIds）|
| `DELETE /nadrf-datamanagement/v1/data-retrieval-subscriptions/{id}` | RetrievalUnsubscribe：刪訂閱，回 204 |
| `GET /health` | 健康檢查 |
| `GET /debug/records` | 回傳所有 storeTransId list（測試驗證用）|
| `GET /debug/subscriptions` | 回傳所有 retrieval subscription（測試驗證用）|

---

## 關鍵設計決策

| 決策 | 理由 |
|------|------|
| `AdrfConfig` 獨立 struct | ADRF 是外部 NF，與 `SmfConfig`/`MlServiceConfig`/`MtlfConfig` 同層級 |
| `smfDataSub` 重用 `ExtendedNsmfEventExposure` | 其 JSON tags 已符合 TS 29.508 `NsmfEventExposure`；不另建 `AdrfSmfDataSub` |
| `upfEventNotifs` 用 `[]json.RawMessage` | 避免 consumer 反向依賴 processor 型別；`json.Marshal(UpfNotificationData)` 已符合格式 |
| `AdrfClient` 在 `Consumer` | Consumer 是「所有外部 NF client」集合，語意一致 |
| `AdrfClient` 條件初始化 | `adrf.url` 有設才建立；與 `adrfBuffer` 的 nil 檢查一致 |
| `AdrfSmfInfo` 在 context（非擴充 `SmfSubscription`）| `SmfSubscription` 職責為路由與生命週期；ADRF 需要的是訂閱規格封存 |
| 填入 `AdrfSmfInfo` 時機：SMF 訂閱成功當下 | 捕捉「當時實際發出的參數」，不依賴 config 事後仍一致 |
| `DeleteAdrfSmfInfo` 在 `cleanupDataCollection` | 訂閱釋放時一併清除，防止 memory leak |
| `adrfBuffer` 無 `infos` map；`info` 直接傳入 | 有 `infos` map 時 `flushOne` 只刪 `pending` 不刪 `infos`，會 leak；直接傳參數更簡單 |
| `adrfBuffer` 在 `processor` package | Consumer = 純 HTTP client（無狀態）；buffer 是 application-level 協調邏輯 |
| Per-correlationId sub-buffer | threshold 語意清楚（同一 UE 累積 N 筆）；不同 UE 各自獨立 |

---

## 整合測試結果

**測試日期**：2026-03-25
**測試環境**：fake_smf_upf（port 8081）+ NWDAF（port 8080）+ fake_adrf（port 9888）

**測試步驟**：
1. 啟動 fake_adrf、fake_smf_upf、NWDAF
2. POST subscription（`UE_COMMUNICATION`，SUPI=`imsi-123456789012345`，`repPeriod=5`）
3. 等待 UPF notifications（每 5 秒一筆，stable pattern）
4. `GET http://127.0.0.1:9888/debug/records` 確認記錄數
5. `GET /nadrf-datamanagement/v1/data-store-records?fetch-correlation-ids=<id>` 驗證結構

**NWDAF 啟動 log**：
```
[INFO][NWDAF][Consumer] ADRF client initialized: url=http://127.0.0.1:9888
[INFO][NWDAF][Proc]     ADRF buffer initialized: threshold=1
```

**訂閱建立 log**：
```
[INFO][NWDAF][Consumer] Subscribing to SMF: endpoint=http://127.0.0.1:8081,
                         supi=imsi-123456789012345, notifId=corr-1
[INFO][NWDAF][Consumer] SMF subscription created: id=fake-smf-sub-12ad89ca
[INFO][NWDAF][Proc]     SMF subscription created: supi=imsi-123456789012345,
                         subId=fake-smf-sub-12ad89ca, corrId=corr-1
```

**每筆 UPF notification → ADRF 存入 log**：
```
[INFO][NWDAF][Proc] ADRF stored: storeTransId=057809df-... supi=imsi-123456789012345 count=1
[INFO][NWDAF][Proc] ADRF stored: storeTransId=48d87e46-... supi=imsi-123456789012345 count=1
[INFO][NWDAF][Proc] ADRF stored: storeTransId=75903024-... supi=imsi-123456789012345 count=1
```

**GET 驗證的 `NadrfDataStoreRecord` 結構**：
```json
{
  "dataSub": [{
    "smfDataSub": {
      "supi": "imsi-123456789012345",
      "notifUri": "http://127.0.0.1:8080/collector/notify",
      "notifId": "corr-1",
      "eventSubs": [{
        "event": "UPF_EVENT",
        "upfEvents": [{ "type": "USER_DATA_USAGE_MEASURES",
                        "measurementTypes": ["VOLUME_MEASUREMENT","THROUGHPUT_MEASUREMENT"],
                        "granularityOfMeasurement": "PER_SESSION" }],
        "bundlingAllowed": true,
        "bundledEventNotifyUri": "http://127.0.0.1:8080/collector/upf-notify"
      }],
      "notifMethod": "PERIODIC",
      "repPeriod": 5
    }
  }],
  "dataNotif": {
    "upfEventNotifs": [{
      "correlationId": "corr-1",
      "notificationItems": [{
        "eventType": "USER_DATA_USAGE_MEASURES",
        "userDataUsageMeasurements": [{
          "volumeMeasurement":    { "totalVolume": 4013, "ulVolume": 1941, "dlVolume": 2072, ... },
          "throughputMeasurement":{ "ulThroughput": "15 Kbps", "dlThroughput": "16 Kbps", ... }
        }]
      }]
    }]
  }
}
```

**結論**：結構完全符合 TS 29.575 `NadrfDataStoreRecord`（`dataSub[smfDataSub] + dataNotif[upfEventNotifs]`）。8 秒內 ADRF 收到 3 筆記錄，每筆均包含完整的 SMF 訂閱參數與 UPF 流量數據。
