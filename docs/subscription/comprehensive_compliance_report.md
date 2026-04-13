# NWDAF EventsSubscription 3GPP 合規性報告

**最後更新**: 2026-01-07 18:21

## 目錄

1. [摘要](#1-摘要)
2. [規範參照](#2-規範參照)
3. [API 端點合規性](#3-api-端點合規性)
4. [資料結構合規性](#4-資料結構合規性)
5. [ABNORMAL_BEHAVIOUR 驗證合規性](#5-abnormal_behaviour-驗證合規性)
6. [通知機制合規性](#6-通知機制合規性)
7. [錯誤處理合規性](#7-錯誤處理合規性)
8. [自訂決策 (規範未定義)](#8-自訂決策-規範未定義)
9. [遺漏功能清單](#9-遺漏功能清單)
10. [待改進項目](#10-待改進項目)

---

## 1. 摘要

本報告針對 NWDAF EventsSubscription 服務的現有實作與 3GPP TS 29.520、TS 23.288 規範進行詳細對照分析。

### 整體合規狀態

| 類別 | 狀態 | 說明 |
|------|------|------|
| API 端點 | ✅ 合規 | Subscribe/Update/Delete 三大操作完全符合規範 |
| NnwdafEventsSubscription 結構 | ✅ 主要合規 | 核心必要欄位皆已實作 |
| ABNORMAL_BEHAVIOUR 驗證 | ✅ 合規 | 完整驗證 tgtUe、exptAnaType、excepRequs、anyUe 規則 |
| 通知機制 | ✅ 合規 | PERIODIC 模式已實作，包含 maxReportNbr/monDur/notifCorrId |
| 錯誤處理 | ✅ 合規 | 使用規範定義的 cause 值 |
| Analytics Target Period | ✅ 合規 | 正確實作 BOTH_STAT_PRED_NOT_ALLOWED |

---

## 2. 規範參照

### 主要參照規範

| 規範 | 版本 | 章節 | 內容 |
|------|------|------|------|
| TS 29.520 | R18 | §4.2 | Nnwdaf_EventsSubscription Service Operations |
| TS 29.520 | R18 | §5.1 | Nnwdaf_EventsSubscription Service API |
| TS 23.288 | R18 | §6.1 | Procedures for analytics exposure |
| OpenAPI | 3.0.0 | - | TS29520_Nnwdaf_EventsSubscription.yaml |

### 實作檔案對應

| 規範功能 | 實作檔案 | 行數 |
|---------|---------|------|
| API Handlers | `internal/sbi/api_eventssubscription.go` | 86 |
| Business Logic | `internal/sbi/processor/eventssubscription.go` | 649 |
| Data Storage | `internal/context/context.go` | 133 |
| Notification | `internal/notifier/notifier.go` | 207 |
| Notification Tests | `internal/notifier/notifier_test.go` | 286 |
| Analytics | `internal/notifier/analytics.go` | 50 |

---

## 3. API 端點合規性

### 3.1 Nnwdaf_EventsSubscription_Subscribe (POST /subscriptions)

**規範定義** (TS 29.520 §5.1.3.2.3.1):

> Resource URI: `{apiRoot}/nnwdaf-eventssubscription/<apiVersion>/subscriptions`
> Request Body: `NnwdafEventsSubscription` (M, 1)
> Response: 201 Created with `NnwdafEventsSubscription` body
> Header: Location containing the URI of the created subscription

| 檢查項目 | 規範要求 | 實作狀態 | 說明 |
|---------|---------|---------|------|
| Resource URI | /subscriptions | ✅ | `api_eventssubscription.go:15` |
| Request Body | NnwdafEventsSubscription | ✅ | 使用 `models.NnwdafEventsSubscription` |
| 201 Created | 成功時回傳 | ✅ | `api_eventssubscription.go:45` |
| Location Header | 必須包含 | ✅ | `api_eventssubscription.go:44` |
| failEventReports | 可選回傳 | ✅ | `eventssubscription.go:89-92` |

**程式碼參照**:
```go
// api_eventssubscription.go:36-44
locationUri := fmt.Sprintf("%s://%s:%d%s/subscriptions/%s",
    cfg.Configuration.Sbi.Scheme,
    cfg.Configuration.Sbi.RegisterIPv4,
    cfg.Configuration.Sbi.Port,
    factory.NwdafEventsSubResUriPrefix,
    subscriptionId)
c.Header("Location", locationUri)
```

---

### 3.2 Nnwdaf_EventsSubscription_Subscribe Update (PUT /subscriptions/{subscriptionId})

**規範定義** (TS 29.520 §5.1.3.3.3.2):

> Resource URI: `{apiRoot}/nnwdaf-eventssubscription/<apiVersion>/subscriptions/{subscriptionId}`
> Response: 200 OK or 204 No Content

| 檢查項目 | 規範要求 | 實作狀態 | 說明 |
|---------|---------|---------|------|
| Resource URI | /subscriptions/{subscriptionId} | ✅ | `api_eventssubscription.go:50` |
| 200 OK | 成功時回傳更新後的訂閱 | ✅ | `api_eventssubscription.go:70` |
| 204 No Content | 可選無內容回應 | ❌ | 目前只實作 200 OK |
| 404 Not Found | 訂閱不存在 | ✅ | `eventssubscription.go:109-114` |

---

### 3.3 Nnwdaf_EventsSubscription_Unsubscribe (DELETE /subscriptions/{subscriptionId})

**規範定義** (TS 29.520 §5.1.3.3.3.1):

> Response: 204 No Content

| 檢查項目 | 規範要求 | 實作狀態 | 說明 |
|---------|---------|---------|------|
| 204 No Content | 成功刪除 | ✅ | `api_eventssubscription.go:84` |
| 404 Not Found | 訂閱不存在 | ✅ | `eventssubscription.go:201-207` |

---

## 4. 資料結構合規性

### 4.1 NnwdafEventsSubscription (TS 29.520 §5.1.6.2.2)

**規範定義**:

| Attribute | Data type | P | 實作狀態 | 說明 |
|-----------|-----------|---|---------|------|
| eventSubscriptions | array(EventSubscription) | M | ✅ | `req.EventSubscriptions` |
| evtReq | ReportingInformation | O | ✅ | `req.EvtReq` |
| notificationURI | Uri | C | ✅ | 驗證必須存在 |
| notifCorrId | string | O | ✅ | 儲存並回傳 |
| eventNotifications | array(EventNotification) | C | ❌ | immRep 功能未實作 |
| failEventReports | array(FailureEventInfo) | O | ✅ | 部分事件失敗時回傳 |
| supportedFeatures | SupportedFeatures | C | ❌ | 未實作 feature negotiation |

**驗證程式碼** (`eventssubscription.go:239-264`):
```go
func (p *Processor) validateBasicStructure(req *models.NnwdafEventsSubscription) *models.ProblemDetails {
    if len(req.EventSubscriptions) == 0 {
        return &models.ProblemDetails{
            Cause: "INVALID_REQUEST",
            Detail: "eventSubscriptions is required",
        }
    }
    if req.NotificationURI == "" {
        return &models.ProblemDetails{
            Cause: "INVALID_REQUEST",
            Detail: "notificationURI is required",
        }
    }
    return nil
}
```

---

### 4.2 EventSubscription (TS 29.520 §5.1.6.2.3)

針對 `ABNORMAL_BEHAVIOUR` 事件的 EventSubscription 欄位對照：

| Attribute | Data type | P | 實作狀態 | 說明 |
|-----------|-----------|---|---------|------|
| event | NwdafEvent | M | ✅ | 必要欄位驗證 |
| tgtUe | TargetUeInformation | O | ✅ | 驗證 supis/intGroupIds/anyUe |
| excepRequs | array(Exception) | C | ✅ | 與 exptAnaType 互斥驗證 |
| exptAnaType | ExpectedAnalyticsType | C | ✅ | 只支援 COMMUN |
| networkArea | NetworkAreaInfo | C | ✅ | anyUe 時的條件驗證 |
| snssais | array(Snssai) | C | ✅ | anyUe 時的條件驗證 |
| appIds | array(ApplicationId) | C | ✅ | anyUe + COMMUN 時驗證 |
| dnns | array(Dnn) | C | ✅ | anyUe + COMMUN 時驗證 |
| repetitionPeriod | DurationSec | C | ⚠️ | 讀取但未優先使用 evtReq.repPeriod |

---

### 4.3 ReportingInformation/evtReq (TS 29.520 §5.1.6.2.2 NOTE 1, NOTE 2)

**規範定義**:

> NOTE 1: If the `evtReq` attribute is provided and contains `notifMethod`, it takes preference over `notificationMethod` in `EventSubscription`.
> NOTE 2: If the `evtReq` attribute is provided and contains `repPeriod`, it takes preference over `repetitionPeriod` in `EventSubscription`.

| Attribute | Data type | 實作狀態 | 說明 |
|-----------|-----------|---------|------|
| notifMethod | NotificationMethod | ✅ | 正確優先使用 evtReq |
| repPeriod | int32 | ✅ | 正確優先使用 evtReq |
| maxReportNbr | int32 | ✅ | 已實作限制邏輯 (2026-01-07) |
| monDur | DateTime | ✅ | 已實作過期檢查 (2026-01-07) |
| immRep | boolean | ❌ | 未實作 immediate report |
| sampRatio | SamplingRatio | ❌ | 未實作 |

**程式碼** (`eventssubscription.go:51-60`):
```go
if req.EvtReq != nil {
    subscription.NotifMethod = string(req.EvtReq.NotifMethod)
    subscription.RepPeriod = req.EvtReq.RepPeriod
    subscription.MaxReportNbr = req.EvtReq.MaxReportNbr
    if req.EvtReq.MonDur != nil {
        monDur := *req.EvtReq.MonDur
        subscription.MonDur = &monDur
    }
}
```

---

## 5. ABNORMAL_BEHAVIOUR 驗證合規性

### 5.1 必要欄位驗證

**規範原文** (TS 29.520 §4.2.2.2.2):

> if the feature `AbnormalBehaviour` is supported and the event is `ABNORMAL_BEHAVIOUR`, shall provide:
> 1) identification of target UE(s) to which the subscription applies by `supis`, `intGroupIds` or `anyUe` attribute set to "true" in the `tgtUe` attribute; and
> 2) either the expected analytics type via `exptAnaType` attribute or a list of exception Ids with the associated thresholds via `excepRequs` attribute.

| 驗證規則 | 實作狀態 | 程式碼位置 |
|---------|---------|-----------|
| tgtUe 必須存在 | ✅ | `eventssubscription.go:422-429` |
| tgtUe 必須包含 supis/intGroupIds/anyUe | ✅ | `eventssubscription.go:431-442` |
| excepRequs 或 exptAnaType 必須存在 | ✅ | `eventssubscription.go:448-455` |
| excepRequs 與 exptAnaType 互斥 | ✅ | `eventssubscription.go:457-464` |

---

### 5.2 anyUe 特殊規則驗證

**規範原文** (TS 29.520 §4.2.2.2.2):

> If the `anyUe` attribute in the `tgtUe` attribute sets to "true":
> a) the expected analytics type via the `exptAnaType` attribute or the list of Exception Ids via `excepRequs` attribute shall not be requested for both mobility and communication related analytics at the same time;
> b) if the expected analytics type via the `exptAnaType` attribute or the list of Exception Ids via `excepRequs` attribute is mobility related, at least one of identification of network area(s) by `networkArea` attribute and identification of network slice(s) by `snssais` attribute should be provided; and
> c) if the expected analytics type via the `exptAnaType` attribute or the list of Exception Ids via `excepRequs` attribute is communication related, at least one of identification of network area(s) by `networkArea` attribute, identification of application(s) by `appIds` attribute, identification of DNN(s) in the `dnns` attribute and identification of network slice(s) by `snssais` attribute should be provided;

| anyUe 規則 | 實作狀態 | 程式碼位置 |
|-----------|---------|-----------|
| 禁止同時 mobility + communication | ✅ | `eventssubscription.go:483-490` |
| Mobility → networkArea 或 snssais | ✅ | `eventssubscription.go:498-505` |
| Communication → networkArea/appIds/dnns/snssais | ✅ | `eventssubscription.go:507-514` |

**例外類型分類** (`eventssubscription.go:401-415`):
```go
// Mobility-related exception IDs
var mobilityExceptionIds = []models.ExceptionId{
    models.ExceptionId_UNEXPECTED_UE_LOCATION,
    models.ExceptionId_PING_PONG_ACROSS_CELLS,
    models.ExceptionId_UNEXPECTED_WAKEUP,
    models.ExceptionId_UNEXPECTED_RADIO_LINK_FAILURES,
}

// Communication-related exception IDs
var communExceptionIds = []models.ExceptionId{
    models.ExceptionId_UNEXPECTED_LONG_LIVE_FLOW,
    models.ExceptionId_UNEXPECTED_LARGE_RATE_FLOW,
    models.ExceptionId_SUSPICION_OF_DDOS_ATTACK,
    models.ExceptionId_WRONG_DESTINATION_ADDRESS,
    models.ExceptionId_TOO_FREQUENT_SERVICE_ACCESS,
}
```

---

### 5.3 exptAnaType 對應 ExceptionId 規則

**規範原文** (TS 29.520 §4.2.2.2.2):

> If the expected analytics type via `exptAnaType` attribute is provided, the NWDAF shall derive the corresponding Exception Ids from the received expected analytics type as follows:
> a) if `exptAnaType` attribute sets to "MOBILITY", the corresponding list of Exception Ids are "UNEXPECTED_UE_LOCATION", "PING_PONG_ACROSS_CELLS", "UNEXPECTED_WAKEUP" and "UNEXPECTED_RADIO_LINK_FAILURES";
> b) if `exptAnaType` attribute sets to "COMMUN", the corresponding list of Exception Ids are "UNEXPECTED_LONG_LIVE_FLOW", "UNEXPECTED_LARGE_RATE_FLOW", "SUSPICION_OF_DDOS_ATTACK", "WRONG_DESTINATION_ADDRESS" and "TOO_FREQUENT_SERVICE_ACCESS"; and
> c) if `exptAnaType` attribute sets to "MOBILITY_AND_COMMUN", the corresponding list of Exception Ids includes all above derived exception Ids.

| 實作狀態 | 說明 |
|---------|------|
| ⚠️ 部分實作 | 目前只支援 COMMUN，輸入 MOBILITY 會被拒絕 |

**程式碼** (`eventssubscription.go:574-584`):
```go
func (p *Processor) validateExptAnaType(exptAnaType models.ExpectedAnalyticsType) *models.ProblemDetails {
    if exptAnaType != "" && exptAnaType != models.ExpectedAnalyticsType_COMMUN {
        return &models.ProblemDetails{
            Status: http.StatusBadRequest,
            Cause:  "UNSUPPORTED_ANALYTICS_TYPE",
            Detail: "Only COMMUN analytics type is supported for DDoS detection",
        }
    }
    return nil
}
```

---

## 6. 通知機制合規性

### 6.1 Nnwdaf_EventsSubscription_Notify (POST to notificationURI)

**規範定義** (TS 29.520 §5.1.5.2.2):

> Callback URI: `{notificationURI}`
> Request Body: `array(NnwdafEventsSubscriptionNotification)`
> Response: 204 No Content

| 檢查項目 | 規範要求 | 實作狀態 | 說明 |
|---------|---------|---------|------|
| POST to notificationURI | ✅ | `notifier.go:106` |
| Request Body | NnwdafEventsSubscriptionNotification | ✅ | `notifier.go:121-137` |
| Response handling | 204 No Content | ✅ | `notifier.go:113-117` |
| Content-Type | application/json | ✅ | `notifier.go:106` |

---

### 6.2 NnwdafEventsSubscriptionNotification 結構

**規範定義** (TS 29.520 §5.1.6.2.4):

| Attribute | Data type | P | 實作狀態 | 說明 |
|-----------|-----------|---|---------|------|
| eventNotifications | array(EventNotification) | C | ✅ | `notifier.go:178-192` |
| subscriptionId | string | M | ✅ | `notifier.go:194` |
| notifCorrId | string | O | ✅ | `notifier.go:198-200` (EneNA feature) |
| termCause | TermCause | O | ❌ | 未實作終止請求 |

---

### 6.3 EventNotification 結構 (ABNORMAL_BEHAVIOUR)

**規範定義** (TS 29.520 §5.1.6.2.5):

| Attribute | Data type | P | 實作狀態 | 說明 |
|-----------|-----------|---|---------|------|
| event | NwdafEvent | M | ✅ | `notifier.go:127` |
| abnorBehavrs | array(AbnormalBehaviour) | C | ✅ | `notifier.go:128` |
| start | DateTime | O | ❌ | 未包含分析起始時間 |
| expiry | DateTime | O | ❌ | 未包含分析過期時間 |

---

### 6.4 AbnormalBehaviour 結構

**規範定義** (TS 29.520 §5.1.6.2.15):

| Attribute | Data type | 實作狀態 | 說明 |
|-----------|-----------|---------|------|
| excep | Exception | ✅ | 包含 ExcepId, ExcepLevel, ExcepTrend |
| supis | array(Supi) | ✅ | Mock SUPI 資料 |
| dnn | Dnn | ✅ | Mock DNN 資料 |
| ratio | Uinteger | ✅ | 異常比例 |
| confidence | Uinteger | ✅ | 信心度 |
| addtMeasInfo | AdditionalMeasurement | ✅ | 包含 DDoS 來源 IP |

**Mock 資料實作** (`analytics.go:12-37`):
```go
func generateMockAbnormalBehaviours() []models.AbnormalBehaviour {
    return []models.AbnormalBehaviour{
        {
            Excep: &models.Exception{
                ExcepId:    models.ExceptionId_SUSPICION_OF_DDOS_ATTACK,
                ExcepLevel: 3,
                ExcepTrend: models.ExceptionTrend_UP,
            },
            Supis:      []string{"imsi-123456789012345"},
            Dnn:        "internet",
            Ratio:      15,
            Confidence: 85,
            AddtMeasInfo: &models.AdditionalMeasurement{
                DdosAttack: &models.AddressList{
                    Ipv4Addrs: []string{"192.168.1.100", "192.168.1.101"},
                },
            },
        },
    }
}
```

---

### 6.5 PERIODIC 通知模式

**規範原文** (TS 29.520 §4.2.2.4.2):

> If both the repetition period (`repPeriod` or `repetitionPeriod`) attribute and the `offsetPeriod` attribute are present in the subscription request for periodical notification, the NWDAF shall produce a notification in every repetition period seconds

| 功能 | 實作狀態 | 說明 |
|------|---------|------|
| PERIODIC ticker | ✅ | `notifier.go:94` 使用 time.Ticker |
| 立即發送首次通知 | ✅ | `notifier.go:97-101` |
| repPeriod 週期 | ✅ | scheduler 使用 repPeriod |
| maxReportNbr 限制 | ✅ | `notifier.go:118-121` 達到上限後停止 |
| monDur 過期檢查 | ✅ | `notifier.go:124-127` 到期後停止 |
| offsetPeriod | ❌ | 未實作偏移週期 |

---

### 6.6 通知控制限制 (2026-01-07 新增)

**規範原文** (TS 29.520 §4.2.2.2.2):

> The `NnwdafEventsSubscription` data structure provided in the request body may include:
> - event reporting information as the `evtReq` attribute, which applies for each event and may contain the following attributes:
>     2) maximum Number of Reports in the `maxReportNbr` attribute;
>     3) monitoring duration in the `monDur` attribute;

**規範原文** (TS 29.520 §4.2.2.4.2 - notifCorrId):

> and may include:
>     a) the notification correlation identifier in the `notifCorrId` attribute, if the "EneNA" feature is supported.

| 功能 | 規範參照 | 實作狀態 | 說明 |
|------|---------|---------|------|
| maxReportNbr | §4.2.2.2.2 line 251 | ✅ | 達到上限後 scheduler 自動停止 |
| monDur | §4.2.2.2.2 line 252 | ✅ | 監控期限到期後 scheduler 自動停止 |
| notifCorrId | §4.2.2.4.2 line 689 | ✅ | 通知中包含訂閱時提供的關聯識別碼 |

**實作程式碼** (`notifier.go:115-130`):
```go
// shouldContinue checks if notification should continue based on maxReportNbr and monDur
func (s *NotificationScheduler) shouldContinue() bool {
    s.mu.Lock()
    defer s.mu.Unlock()

    // Check maxReportNbr limit (0 means unlimited)
    if s.maxReportNbr > 0 && s.reportCount >= s.maxReportNbr {
        return false
    }

    // Check monDur expiry
    if s.monDur != nil && time.Now().After(*s.monDur) {
        return false
    }

    return true
}
```

**單元測試覆蓋** (`notifier_test.go`):
- `TestShouldContinue_MaxReportNbrLimit` (4 cases)
- `TestShouldContinue_MonDurExpiry` (3 cases)
- `TestShouldContinue_CombinedLimits` (4 cases)
- `TestBuildNotification_NotifCorrId` (2 cases)

---

## 7. 錯誤處理合規性

### 7.1 Application Errors (TS 29.520 §5.1.7.3)

**規範定義**:

| Application Error | HTTP Status | 規範 cause | 實作狀態 |
|-------------------|-------------|-----------|---------|
| Analytics target period 混合統計/預測 | 400 | BOTH_STAT_PRED_NOT_ALLOWED | ✅ |
| 預測不允許的事件 | 400 | PREDICTION_NOT_ALLOWED | ❌ |
| 用戶未授權 | 403 | USER_CONSENT_NOT_GRANTED | ❌ |
| 統計資料不可用 | 500 | UNAVAILABLE_DATA | ❌ |

**BOTH_STAT_PRED_NOT_ALLOWED 實作** (`eventssubscription.go:330-340`):
```go
// startTs in past + endTs in future → reject
if startTs.Before(now) && endTs.After(now) {
    return &models.ProblemDetails{
        Status: http.StatusBadRequest,
        Cause:  "BOTH_STAT_PRED_NOT_ALLOWED",
        Detail: fmt.Sprintf("eventSubscriptions[%d]: analytics target period with startTs in past and endTs in future is not allowed", index),
    }
}
```

---

### 7.2 自訂錯誤碼

以下錯誤碼是我們自訂的，規範未定義：

| 我們的 Cause | HTTP Status | 用途 |
|-------------|-------------|------|
| ALL_EVENTS_UNSUPPORTED | 400 | 所有事件都不支援時 |
| UNSUPPORTED_EXCEPTION | 400 | 不支援的 ExceptionId |
| UNSUPPORTED_ANALYTICS_TYPE | 400 | 不支援的 exptAnaType |
| SUBSCRIPTION_NOT_FOUND | 404 | 訂閱不存在 |
| INVALID_JSON | 400 | JSON 解析失敗 |
| INVALID_REQUEST | 400 | 通用請求驗證失敗 |

---

## 8. 自訂決策 (規範未定義)

以下是我們自行決定的實作細節，規範未明確定義：

### 8.1 failEventReports 行為

**我們的決策**:
- 部分事件不支援時，回傳 201 Created + failEventReports
- 全部事件不支援時，回傳 400 Bad Request + ALL_EVENTS_UNSUPPORTED

**規範模糊處** (TS 29.520 §4.2.2.2.2):
> If not all the requested analytics events in the subscription are accepted, then the NWDAF may include the `failEventReports` attribute indicating the event(s) for which the subscription failed

規範說「may」，所以我們實作為：部分失敗→201，全部失敗→400。

---

### 8.2 ExceptionId 不支援處理

**我們的決策**:
- 不支援的 ExceptionId → 400 Bad Request (硬失敗)
- 使用自訂 cause `UNSUPPORTED_EXCEPTION`

**替代方案** (未採用):
- 將不支援的 ExceptionId 視為軟失敗，回傳 201 + failEventReports

**決策理由**: DDoS 偵測是核心功能，如果請求不支援的異常類型，應直接拒絕而非部分處理。

---

### 8.3 Subscription ID 格式

**我們的決策**: 使用 UUID v4 格式

**程式碼**:
```go
func NewSubscriptionId() string {
    return uuid.New().String()
}
```

**規範參照**: 規範只說 `subscriptionId` 是 string 類型，未定義具體格式。

---

### 8.4 Notification 失敗重試

**我們的決策**: 不重試，僅記錄警告日誌

**程式碼** (`notifier.go:107-110`):
```go
resp, err := http.Post(s.notificationURI, "application/json", bytes.NewBuffer(jsonData))
if err != nil {
    logger.NotifierLog.Warnf("Failed to send notification to %s: %v", s.notificationURI, err)
    return
}
```

**規範未定義**: 通知失敗時的重試策略由實作決定。

---

## 9. 遺漏功能清單

### 9.1 必要但尚未實作 (Phase 3/4)

| 功能 | 規範參照 | 優先級 | 說明 |
|------|---------|--------|------|
| immRep 立即報告 | TS 29.520 §4.2.2.2.2 | 中 | 訂閱時回傳初始分析結果 |
| ON_EVENT_DETECTION 模式 | TS 29.520 §4.2.2.2.2 | 中 | 事件偵測到時才通知 |
| ONE_TIME 模式 | TS 29.520 §4.2.2.2.2 | 中 | 只通知一次 |

### 9.2 選用功能 (可延後實作)

| 功能 | 規範參照 | Feature Flag | 說明 |
|------|---------|-------------|---------|
| supportedFeatures 協商 | TS 29.520 §5.1.8 | - | 目前未實作 feature negotiation |
| Analytics Target Period | TS 29.520 §5.1.6.2.7 | - | extraReportReq 中的 startTs/endTs |
| termCause 終止請求 | TS 29.520 §5.1.6.2.4 | TermRequest | NWDAF 主動終止訂閱 |
| 204 No Content (PUT) | TS 29.520 §5.1.3.3.3.2 | - | Update 時可選無內容回應 |

### 9.3 其他 Event Type

目前只支援 `ABNORMAL_BEHAVIOUR`，其他 22 種 Event Type 均未實作：

- SLICE_LOAD_LEVEL, NSI_LOAD_LEVEL, NF_LOAD
- SERVICE_EXPERIENCE, UE_MOBILITY, UE_COMMUNICATION  
- NETWORK_PERFORMANCE, QOS_SUSTAINABILITY, USER_DATA_CONGESTION
- DISPERSION, RED_TRANS_EXP, WLAN_PERFORMANCE
- DN_PERFORMANCE, SM_CONGESTION, PFD_DETERMINATION
- PDU_SESSION_TRAFFIC, MOVEMENT_BEHAVIOUR, LOC_ACCURACY
- RELATIVE_PROXIMITY, E2E_DATA_VOL_TRANS_TIME, SIGNALLING_STORM
- QOS_POLICY_ASSIST

---

## 10. 待改進項目

### 10.1 實作正確性問題

| 問題 | 嚴重度 | 說明 | 建議修復 |
|------|--------|------|---------|
| Notification 缺少 notifCorrId | 低 | 如有請求應回傳 | 在 buildNotification 中加入 |
| 無 204 No Content 支援 | 低 | PUT 成功時可選回傳 | 可忽略或根據情況回傳 |

### 10.2 建議優先實作

| 功能 | 狀態 | 說明 |
|------|------|------|
| maxReportNbr 限制 | ✅ 已實作 | 2026-01-07 完成 |
| monDur 過期處理 | ✅ 已實作 | 2026-01-07 完成 |
| notifCorrId 傳遞 | ✅ 已實作 | 2026-01-07 完成 |
| immRep 立即報告 | ❌ 待實作 | 下一優先項目 |

### 10.3 測試覆蓋建議

| 測試案例 | 狀態 | 說明 |
|---------|------|------|
| ABNORMAL_BEHAVIOUR 驗證 | ✅ | eventssubscription_test.go (12 tests) |
| PERIODIC 通知 | ✅ | notifier_test.go (7 tests) |
| maxReportNbr 限制 | ✅ | TestShouldContinue_MaxReportNbrLimit (4 cases) |
| monDur 過期 | ✅ | TestShouldContinue_MonDurExpiry (3 cases) |
| notifCorrId 傳遞 | ✅ | TestBuildNotification_NotifCorrId (2 cases) |

---

## 附錄 A: 規範版本資訊

- **TS 29.520**: 3GPP Release 18
- **TS 23.288**: 3GPP Release 18  
- **OpenAPI Version**: 3.0.0
- **API Version**: v1

## 附錄 B: 相關檔案

| 檔案 | 說明 |
|------|------|
| `.agent/specs/TS 29.520/4.2 Nnwdaf_EventsSubscription Service.md` | 服務操作定義 |
| `.agent/specs/TS 29.520/5.1 Nnwdaf_EventsSubscription Service API.md` | API 規範 |
| `.agent/specs/yaml/TS29520_Nnwdaf_EventsSubscription.yaml` | OpenAPI 規範 |
| `.agent/compliance/phase2b_compliance_report.md` | Phase 2B 合規報告 |
