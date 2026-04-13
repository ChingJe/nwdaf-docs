# Phase E3 實作計畫：NWDAF ADRF 取資料 + Retrain 前置整合

**狀態**：✅ 完成
**前置條件**：E1（Daisy /upload_data endpoint）、E2（ADRF StorageRequest，已完成）
**完成日期**：2026-03-27

---

## 概述

E3 實作 NWDAF 在觸發 Daisy retrain 之前，先從 ADRF 取回歷史 UPF 資料並上傳至 Daisy 的完整流程：

```
accuracy 下降 → startRetrainWorkflow
  → ADRF-enabled: 逐 SUPI 發 RetrievalSubscribe
  → ADRF callback (/collector/retrieval-notify): accumulate fetchCorrIds
  → 逐 ID RetrievalRequest → UploadData 推 Daisy
  → 收斂（AND 條件 or watchdog）→ RetrievalUnsubscribe
  → submitDaisyTask（帶 TID）
```

若 ADRF 未啟用（`adrf.url` 未設），維持原有流程（直接呼叫 `submitDaisyTask`）。

---

## 異動總表

| 檔案 | 動作 | 關鍵內容 |
|------|------|---------|
| `pkg/factory/config.go` | 修改 | `AdrfConfig` 新增 `RetrainWindow`、`WatchdogTimeout`；`FetchBatchSize` fallback 改為 1；加 constraint 說明 |
| `config/nwdafcfg.yaml` | 修改 | `adrf:` 補 `retrainWindow`、`watchdogTimeout`；`fetchBatchSize` 改為 1 加說明 |
| `internal/context/ml_model.go` | 修改 | `SharedModelInfo` 新增 `GetSubscriberIDs() []string` |
| `internal/sbi/consumer/adrf_service.go` | 修改 | 新增 `AdrfTimePeriod`、`NadrfDataRetrievalSubscription` 型別 + `RetrievalSubscribe`、`RetrievalRequest`、`RetrievalUnsubscribe` 方法 |
| `internal/sbi/consumer/daisy_service.go` | 修改 | 新增 `UploadData()`；`TriggerTrainingAsync` 加 `tidOverride` 參數 |
| `internal/sbi/consumer/daisy_service_test.go` | 修改 | 更新 3 個 `TriggerTrainingAsync` call site（補 `""` 參數） |
| `internal/sbi/api_adrf_notify.go` | 新增 | `POST /collector/retrieval-notify` handler |
| `internal/sbi/api_collector.go` | 修改 | `getCollectorRoutes()` 加入 `/retrieval-notify` route |
| `internal/sbi/processor/processor.go` | 修改 | 新增 `HandleAdrfRetrievalNotify` delegate；修正 nil guard |
| `internal/mtlf/adrf_retrieval.go` | 新增 | `retrainJob` struct + 三個核心函式 |
| `internal/mtlf/mtlf.go` | 修改 | `MtlfService` 加 `activeJobs sync.Map` |
| `internal/mtlf/training.go` | 修改 | `startRetrainWorkflow`（原 `TriggerRetraining`）加 ADRF 前置路徑；提取 `buildNwdafURL`；`submitDaisyTask`（原 `triggerTrainingAsync`）加 `tid` 參數 |
| `internal/mtlf/trigger.go` | 修改 | call site 更新為 `startRetrainWorkflow` |
| `internal/sbi/processor/upf_notify.go` | 修改 | ADRF 存入前逐 `NotificationItem` 拆分（每筆一個 ADRF record） |

---

## 1. `pkg/factory/config.go`

### 修改 `AdrfConfig`

```go
type AdrfConfig struct {
    Url              string `yaml:"url,omitempty"`
    StorageThreshold int    `yaml:"storageThreshold,omitempty"` // default: 1
    // default: 1; ADRF V0 accepts exactly 1 fetch-correlation-id per GET — do not set above 1
    FetchBatchSize  int `yaml:"fetchBatchSize,omitempty"`
    RetrainWindow   int `yaml:"retrainWindow,omitempty"`   // default: 1800 (seconds of history to fetch)
    WatchdogTimeout int `yaml:"watchdogTimeout,omitempty"` // default: 120 (seconds after last callback)
}
```

新增 helper methods：

```go
func (a *AdrfConfig) FetchBatchSizeOrDefault() int  // default 1
func (a *AdrfConfig) RetrainWindowOrDefault() int   // default 1800
func (a *AdrfConfig) WatchdogTimeoutOrDefault() int // default 120
```

---

## 2. `config/nwdafcfg.yaml`

```yaml
  adrf:
    url: http://127.0.0.1:9888
    storageThreshold: 1   # per-UE: flush every N notifications; 1 = immediate
    fetchBatchSize: 1     # IDs per RetrievalRequest GET; ADRF V0 enforces exactly 1 — do not change
    retrainWindow: 1800   # seconds of history to fetch for retrain (30 min)
    watchdogTimeout: 120  # seconds after last callback before watchdog fires
```

---

## 3. `internal/context/ml_model.go`

新增 `GetSubscriberIDs()` 以避免 `runAdrfRetrainWorkflow` 直接存取 `SharedModelInfo.mu`：

```go
func (s *SharedModelInfo) GetSubscriberIDs() []string {
    s.RLock()
    defer s.RUnlock()
    ids := make([]string, 0, len(s.Subscribers))
    for id := range s.Subscribers {
        ids = append(ids, id)
    }
    return ids
}
```

---

## 4. `internal/sbi/consumer/adrf_service.go`

### 新增型別

```go
type AdrfTimePeriod struct {
    StartTime string `json:"startTime"` // RFC 3339
    StopTime  string `json:"stopTime"`  // RFC 3339
}

// dataSub 是單一物件（非 array），與 NadrfDataStoreRecord.dataSub（array）不同
type NadrfDataRetrievalSubscription struct {
    NotifCorrId     string               `json:"notifCorrId"`
    NotificationURI string               `json:"notificationURI"`
    TimePeriod      AdrfTimePeriod       `json:"timePeriod"`
    DataSub         AdrfDataSubscription `json:"dataSub"`
    ConsTrigNotif   bool                 `json:"consTrigNotif,omitempty"`
}
```

### 新增常數

```go
AdrfDataRetrievalSubscriptionsPath = "/nadrf-datamanagement/v1/data-retrieval-subscriptions"
adrfRetrievalTimeout               = 15 * time.Second
adrfFetchTimeout                   = 10 * time.Second
adrfUnsubscribeMaxRetry            = 3
adrfUnsubscribeRetryBackoff        = 500 * time.Millisecond
```

### `RetrievalSubscribe`

- POST `AdrfDataRetrievalSubscriptionsPath`
- 從 `Location` header 取 `subscriptionId = path.Base(location)`
- 期望 201；使用 `adrfRetrievalTimeout`

### `RetrievalRequest`

- GET `AdrfDataStoreRecordsPath?fetch-correlation-ids=id1`
- 200 → decode → return record, nil
- 204 → return nil, nil（此 ID 無資料，非錯誤）
- 其他 → return nil, error
- ADRF V0 嚴格限制每次只接受 **1 個** ID（`maxFetchCorrelationIDPerQ=1`）

### `RetrievalUnsubscribe`

- DELETE `AdrfDataRetrievalSubscriptionsPath/{subscriptionId}`
- 204 → nil；404 → nil（視為已清理）
- 5xx → backoff 重試最多 3 次

---

## 5. `internal/sbi/consumer/daisy_service.go`

### `UploadData`

```go
type DaisyUploadDataRequest struct {
    TID            string            `json:"TID"`
    GroupId        string            `json:"group_id"`
    UpfEventNotifs []json.RawMessage `json:"upfEventNotifs"`
}

func (c *DaisyClient) UploadData(tid, groupId string, upfEventNotifs []json.RawMessage) error
```

- POST `/upload_data`，timeout 10s

### `TriggerTrainingAsync` 簽名變更

```go
func (c *DaisyClient) TriggerTrainingAsync(
    task map[string]any, callbackURL string, tidOverride string,
) (string, error)
```

`tidOverride` 非空時重用（ADRF path，TID 必須與 UploadData 一致）；空時自動產生新 UUID。

---

## 6. `internal/sbi/api_adrf_notify.go`（新增）

```go
type NadrfDataRetrievalNotification struct {
    NotifCorrId    string            `json:"notifCorrId"`
    TimeStamp      string            `json:"timeStamp"`
    FetchInstruct  *FetchInstruction `json:"fetchInstruct,omitempty"`
    TerminationReq bool              `json:"terminationReq,omitempty"`
}

type FetchInstruction struct {
    FetchUri     string   `json:"fetchUri"`
    FetchCorrIds []string `json:"fetchCorrIds"`
}
```

Handler `HandleAdrfRetrievalNotify`：解析 callback → 委派 `s.Processor().HandleAdrfRetrievalNotify(...)` → 回 `204 No Content`。

---

## 7. `internal/sbi/api_collector.go`

在 `getCollectorRoutes()` 加入：

```go
{Name: "AdrfRetrievalNotify", Method: "POST", Pattern: "/retrieval-notify", APIFunc: s.HandleAdrfRetrievalNotify}
```

最終路徑：`POST /collector/retrieval-notify`。

---

## 8. `internal/sbi/processor/processor.go`

```go
func (p *Processor) HandleAdrfRetrievalNotify(notifCorrId string, fetchCorrIds []string, terminationReq bool) {
    p.mtlf.HandleAdrfRetrievalNotify(notifCorrId, fetchCorrIds, terminationReq)
}
```

ADRF buffer 初始化也加入 nil guard：`if c := p.nwdaf.Consumer(); c != nil { if c.Adrf != nil { ... } }`。

---

## 9. `internal/mtlf/adrf_retrieval.go`（新增）

### `retrainJob` struct

```go
type retrainJob struct {
    tid         string
    oldModelUrl string
    store       *nwdaf_context.ModelAccuracyStore

    totalSubs        int
    watchdogDuration time.Duration
    supiToGroup      map[string]string // SUPI → groupId，建立時一次性計算

    mu              sync.Mutex
    subscriptionIds []string
    termCount       int
    closed          bool // guards against late sends after fetchCh close

    fetchCh   chan []string
    closeOnce sync.Once
    watchdog  *time.Timer
}
```

### `runAdrfRetrainWorkflow`（原 `startAdrfRetrieval`）

1. 從 `SharedModelInfo.GetSubscriberIDs()` → `GetNwdafSubResources()` → `GetAdrfSmfInfo(resource.CorrelationId)` 收集 SUPI
2. 一次性建立 `supiToGroup` reverse map（供 `runFetchLoop` O(1) 查詢）
3. 建立 `retrainJob`，register 到 `activeJobs`，啟動 watchdog
4. 逐 SUPI 發 `RetrievalSubscribe`（失敗時 decrement `totalSubs`）
5. 若全部失敗 → fallback `go submitDaisyTask(...)`
6. 正常 → `go runFetchLoop(...)`

fallback 路徑（三種）：無 SharedModel、無 ADRF-tracked SUPI、所有 subscribe 失敗，均直接 `go submitDaisyTask`。

### `runFetchLoop`

- `defer`：panic recover、`watchdog.Stop()`、`activeJobs.Delete()`
- 逐 ID 呼叫 `RetrievalRequest`（chunk 大小由 `fetchBatchSize` 控制，預設 1）
- 從 `record.DataSub[0].SmfDataSub.Supi` + `supiToGroup` 查 groupId
- 呼叫 `daisyClient.UploadData(job.tid, groupId, ...)`
- 迴圈結束後印 summary log：`ids=N fetched=M uploaded=K`
- 呼叫 `cleanupSubscriptions` 後 `go submitDaisyTask(mtlfCfg, job.tid, ...)`

### `HandleAdrfRetrievalNotify`

- 從 `activeJobs` 路由到 job
- lock 內：reset watchdog、increment `termCount`（若 `terminationReq`）、讀取收斂狀態
- 印 log：`ids=N terminationReq=T termCount=M/total`
- `closed` flag guard：`if len(fetchCorrIds) > 0 && !isClosed { job.fetchCh <- fetchCorrIds }`
- 收斂（`termCount >= totalSubs`）→ `closeOnce.Do(close(fetchCh))`

### `cleanupSubscriptions`

拷貝 `subscriptionIds` 後逐一 `RetrievalUnsubscribe`。

---

## 10. `internal/mtlf/training.go` + `trigger.go`

### 函式重命名

| 舊名 | 新名 | 說明 |
|------|------|------|
| `TriggerRetraining` | `startRetrainWorkflow` | 工作流派發層：選 ADRF path 或 direct path |
| `startAdrfRetrieval` | `runAdrfRetrainWorkflow` | 完整 ADRF retrain 流程，非單純「取資料」 |
| `triggerTrainingAsync` | `submitDaisyTask` | 最底層 Daisy HTTP 送任務 |

### `buildNwdafURL` 提取

`buildCallbackURL()` 和 `runAdrfRetrainWorkflow` 中的 `buildNwdafURL("/collector/retrieval-notify")` 共用同一個 helper。

---

## 11. `internal/sbi/processor/upf_notify.go`

ADRF 存入前逐 `NotificationItem` 拆分：

```go
for i := range notif.NotificationItems {
    singleNotif := UpfNotificationData{
        CorrelationId:     notif.CorrelationId,
        EventNotifyUri:    notif.EventNotifyUri,
        NotificationItems: []UpfNotificationItem{notif.NotificationItems[i]},
    }
    // marshal → adrfBuffer.add(info, notifJSON)
}
```

**理由**：每個 `NotificationItem` 各有獨立的 `startTime`。若多筆 item 打包成同一個 ADRF record，ADRF 時間窗查詢（`notificationItems[*].startTime`）命中任一筆就整包回傳，精度不足。拆分後每筆 item 對應一個 `storeTransId`，與 MongoDB per-item 儲存邏輯一致。

---

## 實測紀錄

### 第一次實測（2026-03-26 午後，master.log）

| 項目 | 狀態 | 說明 |
|------|------|------|
| ADRF StorageRequest | ✅ 正常 | 兩個 SUPI 持續寫入 |
| RetrievalSubscribe | ✅ 正常 | 201 + subscriptionId |
| /collector/retrieval-notify callback | ✅ 正常 | 兩筆 callback 204 |
| RetrievalRequest（從 ADRF 取資料） | ✅ 正常 | UploadData 有被呼叫可驗證 |
| UploadData 推 Daisy | ❌ 失敗 | Daisy 那台 MongoDB 未啟動，回 500 |
| `terminationReq=true` 送達 | ❌ 未收到 | watchdog 在 120s 後觸發；ADRF 端 `terminationReq` 實作可能尚未完成 |

### 第二次實測（2026-03-26，nwdaf_0326.log）

| 項目 | 狀態 | 說明 |
|------|------|------|
| ADRF StorageRequest | ✅ 正常 | 兩個 SUPI 持續寫入，全程無錯 |
| RetrievalSubscribe | ✅ 正常 | 4 次 retrain 均回 201 + subscriptionId |
| /collector/retrieval-notify callback | ✅ 正常 | 每次 retrain 收到 7 個 callback，回 204 |
| `terminationReq=true` 送達 | ✅ 正常 | 每次均收到 2/2，ADRF 端正常 |
| RetrievalRequest（從 ADRF 取資料） | ✅ 正常 | 591/603/615/627 筆，fetched=ids |
| UploadData 推 Daisy | ✅ 正常 | uploaded=fetched，全部成功 |
| Daisy 訓練 → callback → 換版 | ❌ 失敗 | `model/model.npy` not found（Daisy 已知 bug，修復中） |

---

## E2E 驗證

- [x] 確認 `terminationReq=true` 正常送達
- [x] 確認 UploadData 成功（uploaded=fetched）
- [x] 確認 `submitDaisyTask` 被呼叫並帶正確 TID
- [ ] 確認 Daisy 以 TID 查到 dataset 並執行訓練（待 Daisy bug 修復）
- [ ] 確認 callback 回 NWDAF 並觸發換版流程
