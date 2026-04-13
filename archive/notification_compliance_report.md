# 通知機制合規性報告

**日期**: 2026-01-06  
**規格版本**: 3GPP TS 29.520 V18.6.0

---

## 1. 實作項目總覽

| # | 項目 | 實作 | 合規狀態 |
|---|------|------|---------|
| 1 | HTTP POST 到 notificationURI | ✅ | ✅ |
| 2 | subscriptionId 必填 | ✅ | ✅ |
| 3 | eventNotifications 陣列 | ✅ | ✅ |
| 4 | ABNORMAL_BEHAVIOUR → abnorBehavrs | ✅ | ✅ |
| 5 | AbnormalBehaviour.excep 必填 | ✅ | ✅ |
| 6 | PERIODIC 週期通知 | ✅ | ✅ |
| 7 | Consumer 回應 204 | ✅ | ✅ |
| 8 | Update 時重啟 scheduler | ✅ | ✅ |
| 9 | Delete 時停止 scheduler | ✅ | ✅ |
| 10 | notifCorrId 回傳 | ⚠️ | 未實作 |
| 11 | maxReportNbr 限制 | ⚠️ | 未實作 |
| 12 | monDur 監控期限 | ⚠️ | 未實作 |
| 13 | ONE_TIME/ON_EVENT_DETECTION | ⚠️ | 未實作 |

---

## 2. 詳細合規對照

### 2.1 HTTP POST 到 notificationURI

**規格來源**: TS 29.520 §4.2.2.4.2

**原文**:
> "The NWDAF shall invoke this service operation by sending an **HTTP POST** request using the `{notificationURI}` received during the subscription process."

**實作**: [notifier.go L106](file:///home/x81u/NYCU/lab/NWDAF/internal/notifier/notifier.go#L106)
```go
resp, err := http.Post(s.notificationURI, "application/json", bytes.NewBuffer(jsonData))
```

---

### 2.2 subscriptionId 必填

**規格來源**: TS 29.520 §5.1.6.2.4 Table 5.1.6.2.4-1

> | Attribute | P | Description |
> |-----------|---|-------------|
> | `subscriptionId` | **M** | String identifying a subscription |

**YAML 定義** (L587-590):
```yaml
NnwdafEventsSubscriptionNotification:
  required:
    - subscriptionId
```

**實作**: [notifier.go L134-137](file:///home/x81u/NYCU/lab/NWDAF/internal/notifier/notifier.go#L134-137)
```go
return models.NnwdafEventsSubscriptionNotification{
    SubscriptionId:     s.subscriptionId,
    EventNotifications: eventNotifications,
}
```

---

### 2.3 eventNotifications 陣列

**規格來源**: TS 29.520 §4.2.2.4.2

**原文**:
> "If the notification is for analytics information, the `eventNotifications` attribute for each event **shall include** the `event` identifier..."

**實作**: [notifier.go L122-132](file:///home/x81u/NYCU/lab/NWDAF/internal/notifier/notifier.go#L122-132)

---

### 2.4 ABNORMAL_BEHAVIOUR → abnorBehavrs

**規格來源**: TS 29.520 §4.2.2.4.2 Table

> | Subscribed Event | Notification Attribute |
> |------------------|------------------------|
> | `ABNORMAL_BEHAVIOUR` | `abnorBehavrs` |

**實作**: [notifier.go L127-128](file:///home/x81u/NYCU/lab/NWDAF/internal/notifier/notifier.go#L127-128)
```go
eventNotification := models.NwdafEventsSubscriptionEventNotification{
    Event:        eventSub.Event,
    AbnorBehavrs: generateMockAbnormalBehaviours(),
}
```

---

### 2.5 AbnormalBehaviour.excep 必填

**規格來源**: TS 29.520 YAML L1538-1539

```yaml
AbnormalBehaviour:
  required:
    - excep
```

**實作**: [analytics.go L19-22](file:///home/x81u/NYCU/lab/NWDAF/internal/notifier/analytics.go#L19-22)
```go
Excep: &models.Exception{
    ExcepId:    models.ExceptionId_SUSPICION_OF_DDOS_ATTACK,
    ExcepLevel: 3,
    ExcepTrend: models.ExceptionTrend_UP,
},
```

---

### 2.6 PERIODIC 週期通知

**規格來源**: TS 29.520 §4.2.2.2.2

**原文**:
> "If the event notification method `PERIODIC` is selected via the `notificationMethod` attribute, repetition period as `repetitionPeriod` attribute."

**§4.2.2.4.2 補充**:
> "If both the repetition period (`repPeriod` or `repetitionPeriod`) and the `offsetPeriod` attributes are present..."

**實作**: [notifier.go L77-93](file:///home/x81u/NYCU/lab/NWDAF/internal/notifier/notifier.go#L77-93)
```go
ticker := time.NewTicker(time.Duration(s.repPeriod) * time.Second)
```

---

### 2.7 Consumer 回應 204

**規格來源**: TS 29.520 §4.2.2.4.2

**原文**:
> "Upon receiving the HTTP POST request:
> - **Success:** Store the notification and respond with **"204 No Content"**."

**實作**: [notifier.go L113-117](file:///home/x81u/NYCU/lab/NWDAF/internal/notifier/notifier.go#L113-117)
```go
if resp.StatusCode == http.StatusNoContent {
    logger.NotifierLog.Infof("Notification sent successfully to %s", s.notificationURI)
}
```

---

### 2.8 Update 時重啟 scheduler

**規格來源**: 規格隱含 (訂閱更新應反映新參數)

**實作**: [eventssubscription.go L133-178](file:///home/x81u/NYCU/lab/NWDAF/internal/sbi/processor/eventssubscription.go#L133-178)
```go
// Stop existing scheduler before updating
if existing.Scheduler != nil {
    existing.Scheduler.Stop()
}
// ... update ...
// Start new scheduler if PERIODIC
```

---

### 2.9 Delete 時停止 scheduler

**規格來源**: 規格隱含 (刪除訂閱應停止通知)

**實作**: [eventssubscription.go L195-198](file:///home/x81u/NYCU/lab/NWDAF/internal/sbi/processor/eventssubscription.go#L195-198)
```go
if subscription.Scheduler != nil {
    subscription.Scheduler.Stop()
}
```

---

## 3. 待實作項目

### 3.1 notifCorrId 回傳

**規格來源**: TS 29.520 §5.1.6.2.4

> | Attribute | P | Description |
> |-----------|---|-------------|
> | `notifCorrId` | O | Notification correlation identifier |

**狀態**: 目前 notification 未包含此欄位

---

### 3.2 maxReportNbr 限制

**規格來源**: TS 29.520 §4.2.2.2.2

> "Maximum Number of Reports in the `maxReportNbr` attribute."

**狀態**: 欄位已儲存，但未實作計數器邏輯

---

### 3.3 monDur 監控期限

**規格來源**: TS 29.520 §4.2.2.2.2

> "Monitoring duration in the `monDur` attribute."

**狀態**: 欄位已儲存，但未實作到期檢查

---

### 3.4 其他通知模式

**規格來源**: TS 29.520 §4.2.2.2.2

> "`notifMethod` attribute: `PERIODIC`, `ONE_TIME`, `ON_EVENT_DETECTION`, `THRESHOLD`"

**狀態**: 僅實作 PERIODIC

---

## 4. Mock 資料合規性

### 4.1 AbnormalBehaviour 欄位

| 欄位 | 規格 | 實作 |
|------|------|------|
| `excep` | M | ✅ SUSPICION_OF_DDOS_ATTACK |
| `excep.excepLevel` | O | ✅ 3 |
| `excep.excepTrend` | O | ✅ UP |
| `supis` | O | ✅ ["imsi-123456789012345"] |
| `dnn` | O | ✅ "internet" |
| `ratio` | O | ✅ 15 |
| `confidence` | O | ✅ 85 |
| `addtMeasInfo.ddosAttack` | O | ✅ IPv4 列表 |

---

## 5. 合規總結

| 類別 | 數量 | 狀態 |
|------|------|------|
| 核心通知機制 | 9 | ✅ 完全合規 |
| 進階功能 | 4 | ⚠️ 待實作 |
| Mock 資料 | 8 欄位 | ✅ 符合 schema |

> **備註**: 進階功能 (notifCorrId、maxReportNbr、monDur、其他模式) 為後續階段實作項目。
