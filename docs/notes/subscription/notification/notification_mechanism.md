# Notification Mechanism Research

## Overview

This document consolidates the 3GPP specification requirements for implementing the NWDAF notification mechanism (`Nnwdaf_EventsSubscription_Notify`).

---

## 1. Input Parameters (from Subscription Request)

### 1.1 Notification Target

| Field | Type | Description |
|-------|------|-------------|
| `notificationURI` | URI | Callback URL to send notifications |
| `notifCorrId` | string | Correlation ID, must be returned in notification |

### 1.2 Reporting Configuration (`evtReq`)

From `ReportingInformation` schema:

| Field | Type | Description | Values |
|-------|------|-------------|--------|
| `notifMethod` | enum | Notification method | `PERIODIC`, `ONE_TIME`, `ON_EVENT_DETECTION`, `THRESHOLD` |
| `repPeriod` | integer | Repetition period (seconds) | For PERIODIC mode |
| `maxReportNbr` | integer | Max number of reports | Limits notifications |
| `monDur` | DateTime | Monitoring duration | Subscription expiry time |
| `immRep` | boolean | Immediate report | If true, include first result in 201 response |

### 1.3 Event-Specific Parameters (for ABNORMAL_BEHAVIOUR)

| Field | Type | Description |
|-------|------|-------------|
| `tgtUe` | TargetUeInformation | Target UE(s) - anyUe/supis/intGroupIds |
| `excepRequs` | Exception[] | Specific exception types to monitor |
| `exptAnaType` | ExpectedAnalyticsType | MOBILITY / COMMUN / MOBILITY_AND_COMMUN |
| `networkArea` | NetworkAreaInfo | Target network area |
| `dnns` | string[] | Target DNNs |
| `snssais` | Snssai[] | Target network slices |
| `appIds` | string[] | Target applications |

---

## 2. Notification Data Structure

### 2.1 `NnwdafEventsSubscriptionNotification`

```yaml
NnwdafEventsSubscriptionNotification:
  required:
    - subscriptionId
  properties:
    subscriptionId: string      # Must match original subscription
    notifCorrId: string         # Echo back from subscription
    eventNotifications: EventNotification[]
```

### 2.2 `EventNotification`

```yaml
EventNotification:
  properties:
    event: NwdafEvent           # "ABNORMAL_BEHAVIOUR"
    start: DateTime             # Validity start time
    expiry: DateTime            # Validity end time
    timeStampGen: DateTime      # Generation timestamp
    abnorBehavrs: AbnormalBehaviour[]  # For ABNORMAL_BEHAVIOUR event
    failNotifyCode: NwdafFailureCode   # If failed
    rvWaitTime: integer         # Recommended wait time on failure
```

### 2.3 `AbnormalBehaviour` (Output)

```yaml
AbnormalBehaviour:
  required:
    - excep
  properties:
    supis: string[]             # Affected UE SUPIs
    excep: Exception            # Required - detected exception
    dnn: string                 # Affected DNN
    snssai: Snssai              # Affected slice
    ratio: integer              # Sampling ratio (0-100)
    confidence: integer         # Confidence level
    addtMeasInfo:               # Additional measurement info
      ddosAttack: AddressList   # For SUSPICION_OF_DDOS_ATTACK
      unexpFlowTeps: IpEthFlowDescription[]
      wrgDest: AddressList
```

### 2.4 `Exception`

```yaml
Exception:
  required:
    - excepId
  properties:
    excepId: ExceptionId        # e.g., "SUSPICION_OF_DDOS_ATTACK"
    excepLevel: integer         # Exception severity level
    excepTrend: ExceptionTrend  # UP / DOWN / STABLE / UNKNOW
```

---

## 3. Notification Trigger Modes

| Mode | Description | Implementation |
|------|-------------|----------------|
| `PERIODIC` | Send at fixed intervals | Timer with `repPeriod` seconds |
| `ONE_TIME` | Send once when ready | Single notification then stop |
| `ON_EVENT_DETECTION` | Send when event detected | Real-time detection trigger |
| `THRESHOLD` | Send when threshold crossed | Compare against configured thresholds |

---

## 4. Output Strategy (Stage 3 - TS 29.520)

**Found in Stage 3 YAML spec!** The `OutputStrategy` enum is defined and used in:
- `AnalyticsMetadataRequest.strategy`
- `AnalyticsMetadataInfo.strategy`

### 4.1 OutputStrategy Enum

```yaml
OutputStrategy:
  enum:
    - BINARY
    - GRADIENT
```

| Value | Description (from spec) |
|-------|------------------------|
| `BINARY` | Analytics shall only be reported when the **requested level of accuracy is reached** within a cycle of periodic notification |
| `GRADIENT` | Analytics shall be reported **according with the periodicity** irrespective of whether the requested level of accuracy has been reached or not |

### 4.2 Accuracy Enum

```yaml
Accuracy:
  enum:
    - LOW
    - MEDIUM
    - HIGH
    - HIGHEST
```

Used with Binary strategy to set the accuracy threshold.

### 4.3 Where These Are Used

| Schema | Field | Description |
|--------|-------|-------------|
| `AnalyticsMetadataRequest` | `strategy` | Consumer specifies preferred strategy |
| `AnalyticsMetadataInfo` | `strategy` | NWDAF reports which strategy was used |
| `AnalyticsMetadataInfo` | `accuracy` | NWDAF reports achieved accuracy level |

### 4.4 Note on Applicability

These fields are under `AnalyticsMetadataRequest` which is tied to the **"Aggregation"** feature. For basic ABNORMAL_BEHAVIOUR notifications:

| Scenario | Approach |
|----------|----------|
| Real-time DDoS detection | Use `notifMethod = ON_EVENT_DETECTION` in `evtReq` |
| Periodic reporting | Use `notifMethod = PERIODIC` + `repPeriod` in `evtReq` |

The Output Strategy (BINARY/GRADIENT) is more relevant when accuracy tracking is enabled.

---

## 5. Implementation Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    Notification Flow                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Subscription Created                                        │
│     └─ Store: notificationURI, notifCorrId, evtReq             │
│                                                                 │
│  2. Start Monitoring (based on notifMethod)                     │
│     ├─ PERIODIC: Start timer with repPeriod                    │
│     ├─ ONE_TIME: Wait for first result                         │
│     └─ ON_EVENT_DETECTION: Register detection callback         │
│                                                                 │
│  3. Analytics Engine Produces Result                            │
│     └─ AbnormalBehaviour with Exception detected               │
│                                                                 │
│  4. Send Notification                                           │
│     ├─ POST to notificationURI                                  │
│     ├─ Include subscriptionId, notifCorrId                     │
│     └─ Include eventNotifications with abnorBehavrs            │
│                                                                 │
│  5. Consumer Response                                           │
│     └─ 204 No Content (success)                                │
│                                                                 │
│  6. Check Termination Conditions                                │
│     ├─ maxReportNbr reached?                                   │
│     ├─ monDur expired?                                         │
│     └─ If yes, stop notifications                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. HTTP Request Format

### 5.1 Request

```http
POST {notificationURI} HTTP/1.1
Content-Type: application/json

{
  "subscriptionId": "c2b9cabc-572b-42fd-8357-c896bd84a57d",
  "notifCorrId": "correlation-123",
  "eventNotifications": [
    {
      "event": "ABNORMAL_BEHAVIOUR",
      "timeStampGen": "2026-01-05T16:00:00Z",
      "abnorBehavrs": [
        {
          "supis": ["imsi-123456789"],
          "excep": {
            "excepId": "SUSPICION_OF_DDOS_ATTACK",
            "excepLevel": 5,
            "excepTrend": "UP"
          },
          "dnn": "internet",
          "confidence": 85,
          "addtMeasInfo": {
            "ddosAttack": {
              "ipv4Addrs": ["192.168.1.100", "192.168.1.101"]
            }
          }
        }
      ]
    }
  ]
}
```

### 5.2 Response

```http
HTTP/1.1 204 No Content
```

---

## 6. Error Handling

### 6.1 Failure Notification

When analytics cannot be produced:

```json
{
  "subscriptionId": "c2b9cabc-572b-42fd-8357-c896bd84a57d",
  "eventNotifications": [
    {
      "event": "ABNORMAL_BEHAVIOUR",
      "failNotifyCode": "UNAVAILABLE_DATA",
      "rvWaitTime": 60
    }
  ]
}
```

### 6.2 Failure Codes

| Code | Description |
|------|-------------|
| `UNAVAILABLE_DATA` | Required data not available |
| `BOTH_STAT_PRED_NOT_ALLOWED` | Statistics and prediction cannot be requested together |
| `UNSUPPORTED_MATCH_DIR` | Unsupported matching direction |

---

## 7. Required Implementation Components

### 7.1 Consumer Module (HTTP Client)

```go
// internal/sbi/consumer/notification.go
type NotificationSender interface {
    SendNotification(uri string, notification *models.NnwdafEventsSubscriptionNotification) error
}
```

### 7.2 Notification Manager

```go
// internal/notification/manager.go
type NotificationManager struct {
    subscriptions map[string]*SubscriptionContext
}

type SubscriptionContext struct {
    Subscription  *context.Subscription
    Timer         *time.Timer          // For PERIODIC
    ReportCount   int                  // Track against maxReportNbr
    ExpiryTime    time.Time            // From monDur
}
```

### 7.3 Analytics Integration Point

```go
// Callback when analytics engine produces result
func (m *NotificationManager) OnAnalyticsResult(
    subscriptionId string,
    result *models.AbnormalBehaviour,
) error
```

---

## 8. Implementation Priority

| Priority | Component | Description |
|----------|-----------|-------------|
| ⭐⭐⭐ | NotificationSender | HTTP POST client |
| ⭐⭐⭐ | NotificationManager | Subscription lifecycle |
| ⭐⭐ | PERIODIC timer | Timer-based notifications |
| ⭐⭐ | ON_EVENT_DETECTION | Event-triggered notifications |
| ⭐ | Failure handling | failNotifyCode support |
| ⭐ | monDur/maxReportNbr | Termination conditions |
