# Collector 模組實作報告

**日期**: 2026-01-13  
**目的**: 從 SMF 蒐集 UE 通訊資料，供 NWDAF 分析使用

---

## 1. 模組概覽

```
internal/
├── collector/                     # 新增目錄
│   ├── collector.go               # Context 與資料儲存
│   ├── smf_client.go              # SMF 訂閱客戶端
│   ├── notification_handler.go    # 通知處理
│   └── collector_test.go          # 單元測試
└── sbi/
    └── api_collector.go           # HTTP 路由處理 (新增)
```

---

## 2. 新增檔案詳解

### 2.1 collector.go (134 行)

**核心資料結構**:

```go
type CollectorContext struct {
    SmfSubscriptions sync.Map  // map[subscriptionId]*SmfSubscription
    UeDataStore      sync.Map  // map[supi]*UeCommunicationData
}

type UeCommunicationData struct {
    Supi, Dnn, Snssai, PduSessId
    StartTime, LastUpdate time.Time
    Events []SmfEventExposureEventNotification  // 原始事件
    TotalUlVolume, TotalDlVolume, SessionCount  // 聚合指標
}
```

**主要函式**:

| 函式 | 用途 |
|------|------|
| `GetSelf()` | 取得 singleton context |
| `StoreSubscription/GetSubscription/DeleteSubscription` | SMF 訂閱管理 |
| `StoreUeData/GetUeData/GetOrCreateUeData` | UE 資料存取 |
| `AppendEvent` | 新增事件並更新元資料 |
| `ClearUeData/ClearSubscriptions` | 測試用清除函式 |

---

### 2.2 smf_client.go (149 行)

**功能**: 實作 `Nsmf_EventExposure` 服務客戶端

| 函式 | HTTP | 用途 |
|------|------|------|
| `SubscribeToSmf` | POST | 建立 SMF 事件訂閱 |
| `UnsubscribeFromSmf` | DELETE | 取消 SMF 訂閱 |
| `SubscribeForUeCommunication` | - | 便利函式，訂閱 PDU_SES_EST/REL + QOS_MON |

**訂閱流程**:
```
1. 產生 notifId (UUID)
2. 建立 NsmfEventExposure 請求
3. POST 到 SMF /nsmf-event-exposure/v1/subscriptions
4. 解析 Location header 或 response body 取得 subscriptionId
5. 儲存到 SmfSubscriptions
```

---

### 2.3 notification_handler.go (71 行)

**功能**: 處理 SMF 發送的事件通知

| 函式 | 用途 |
|------|------|
| `HandleNotification` | 處理 `NsmfEventExposureNotification` |
| `processEvent` | 處理單一事件，呼叫 AppendEvent |
| `handlePduSessionEstablished` | 處理 PDU_SES_EST，增加 SessionCount |
| `handlePduSessionReleased` | 處理 PDU_SES_REL |
| `handleQosMonitoring` | 處理 QOS_MON |

---

### 2.4 api_collector.go (54 行)

**功能**: 提供 HTTP 端點接收 SMF 通知

```go
// 路由註冊
POST /collector/notify → HandleCollectorNotify

// 處理流程
1. 解析 JSON → NsmfEventExposureNotification
2. 呼叫 collector.HandleNotification()
3. 回傳 204 No Content
```

---

### 2.5 collector_test.go (160 行)

| 測試 | 驗證內容 |
|------|---------|
| `TestStoreAndRetrieveSubscription` | 訂閱 CRUD |
| `TestStoreAndRetrieveUeData` | UE 資料 CRUD |
| `TestGetOrCreateUeData` | 自動建立功能 |
| `TestAppendEvent` | 事件追加與元資料更新 |
| `TestHandleNotification` | 完整通知處理流程 |

---

## 3. 修改的現有檔案

| 檔案 | 修改內容 |
|------|---------|
| `internal/logger/logger.go` | 新增 `CollectorLog` |
| `internal/sbi/server.go` | 新增 `/collector` 路由群組 |
| `internal/notifier/analytics.go` | 整合 collector，使用蒐集資料 |
| `pkg/factory/config.go` | 新增 `DataCollection` 配置結構 |

---

## 4. 資料流程圖

```
┌────────────┐  1. 訂閱   ┌───────┐
│  Consumer  │ ─────────►│ NWDAF │
└────────────┘           │       │
                         │   2. 呼叫 SubscribeToSmf
                         │       │
                         ▼       ▼
                      ┌───────────────┐
                      │      SMF      │
                      └───────────────┘
                              │
                    3. POST /collector/notify
                              │
                              ▼
                      ┌───────────────┐
                      │   collector/  │
                      │ HandleNotify  │
                      └───────────────┘
                              │
                    4. 儲存到 UeDataStore
                              │
                              ▼
                      ┌───────────────┐
                      │   notifier/   │
                      │  analytics.go │◄── 5. 查詢蒐集資料
                      └───────────────┘
                              │
                    6. 產生分析報告
                              │
                              ▼
                      ┌────────────┐
                      │  Consumer  │
                      └────────────┘
```

---

## 5. 配置支援

```yaml
# config/nwdafcfg.yaml
dataCollection:
  smf:
    enabled: true
    endpoints:
      - "http://127.0.0.1:29502"
    subscriptionDuration: 3600
```

---

## 6. 測試結果

```
=== RUN   TestHandleNotification
INFO Received SMF notification, notifId: notif-001, events: 2
INFO PDU Session Established: supi=imsi-111111111111111
PASS
ok  github.com/free5gc/nwdaf/internal/collector  0.002s
```

**總計**: 5 個 collector 測試 + 24 個其他測試 = 29 通過
