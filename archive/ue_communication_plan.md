# UE Communication Implementation Plan

**Last Updated**: 2026-01-11 22:31

---

## Goal

Enable `UE_COMMUNICATION` event support for N4 Session Inactivity Timer Optimization.

---

## Spec Verification Summary

### Stage 3 TS 29.520 §4.2 (Subscription Input)
| Field | Requirement | Source |
|-------|-------------|--------|
| `event` | `UE_COMMUNICATION` | Required |
| `tgtUe` | `supis` or `intGroupIds` | **Required** (line 361) |
| `listOfAnaSubsets` | Optional, may include `N4_SESS_INACT_TIMER_FOR_UE_COMM` | (line 368) |
| `appIds`, `networkArea`, `dnns`, `snssais` | Optional | (lines 364-367) |

### Stage 3 TS 29.520 §5.1 (Notification Output)
| Field | Type | Requirement | Source |
|-------|------|-------------|--------|
| `ueComms` | `array(UeCommunication)` | **Mandatory for UE_COMMUNICATION** | line 891 |

### YAML Spec: UeCommunication Required Fields
```yaml
required:
  - commDur      # Duration in seconds
  - trafChar     # Traffic Characterization
  - ts | recurringTime  # One of these is required
```

### YAML Spec: SessInactTimerForUeComm
```yaml
SessInactTimerForUeComm:
  required:
    - n4SessId           # PDU Session ID
    - sessInactiveTimer  # Duration in seconds
```

---

## Phase 4.1: Subscription Validation (Input)

### File: `internal/sbi/processor/eventssubscription.go`

**1. Update `validateEventSubscription` (~line 327)**
```go
// For UE_COMMUNICATION, validate tgtUe requirements
if eventSub.Event == models.NwdafEvent_UE_COMMUNICATION {
    if err := p.validateUeCommunication(eventSub); err != nil {
        return err
    }
}
```

**2. Add new function**
```go
// validateUeCommunication validates UE_COMMUNICATION specific requirements
// Per TS 29.520 §4.2: tgtUe with supis or intGroupIds is REQUIRED
func (p *Processor) validateUeCommunication(
    eventSub *models.NwdafEventsSubscriptionEventSubscription,
) *models.ProblemDetails {
    // Rule: tgtUe with supis or intGroupIds is required
    if eventSub.TgtUe == nil {
        return &models.ProblemDetails{
            Status: http.StatusBadRequest,
            Cause:  "INVALID_REQUEST",
            Detail: "tgtUe is required for UE_COMMUNICATION",
        }
    }
    if len(eventSub.TgtUe.Supis) == 0 && len(eventSub.TgtUe.IntGroupIds) == 0 {
        return &models.ProblemDetails{
            Status: http.StatusBadRequest,
            Cause:  "INVALID_REQUEST",
            Detail: "tgtUe must contain supis or intGroupIds for UE_COMMUNICATION",
        }
    }
    return nil
}
```

**3. Update `collectFailEventReports` to allow UE_COMMUNICATION**
```go
case models.NwdafEvent_UE_COMMUNICATION:
    return nil // Supported
```

---

## Phase 4.2: Notification Generation (Output)

### File: `internal/notifier/notifier.go`

**1. Update `buildNotification` (~line 197)**
```go
case models.NwdafEvent_UE_COMMUNICATION:
    eventNotification := models.NwdafEventsSubscriptionEventNotification{
        Event:   eventSub.Event,
        UeComms: []models.UeCommunication{generateMockUeCommunication()},
    }
    eventNotifications = append(eventNotifications, eventNotification)
```

**2. Add new function**
```go
// generateMockUeCommunication generates mock data for UE Communication analytics
// Per YAML spec: commDur, trafChar, ts are REQUIRED. sessInactTimer is OPTIONAL.
func generateMockUeCommunication() models.UeCommunication {
    now := time.Now()
    return models.UeCommunication{
        CommDur:    int32(rand.Intn(3600) + 60),  // 60-3660 seconds
        Ts:         &now,
        TrafChar: &models.TrafficCharacterization{
            Dnn:   "internet",
            UlVol: 1024000,  // 1MB
            DlVol: 5120000,  // 5MB
        },
        SessInactTimer: &models.SessInactTimerForUeComm{
            N4SessId:          int32(rand.Intn(5) + 1),
            SessInactiveTimer: int32(rand.Intn(300) + 60),
        },
    }
}
```

---

## Phase 4.3: Unit Tests

### `eventssubscription_test.go`
| Test | Input | Expected |
|------|-------|----------|
| `TestValidateUeCommunication_ValidSupis` | supis present | nil |
| `TestValidateUeCommunication_ValidIntGroupIds` | intGroupIds present | nil |
| `TestValidateUeCommunication_MissingTgtUe` | tgtUe nil | 400 |
| `TestValidateUeCommunication_EmptyTgtUe` | supis=[], intGroupIds=[] | 400 |

### `notifier_test.go`
| Test | Description | Expected |
|------|-------------|----------|
| `TestBuildNotification_UeCommunication` | Build notification | Contains `ueComms` with required fields |

---

## API Example

### Request
```json
POST /nnwdaf-eventssubscription/v1/subscriptions
{
  "eventSubscriptions": [{
    "event": "UE_COMMUNICATION",
    "tgtUe": {"supis": ["imsi-208930000000003"]},
    "listOfAnaSubsets": ["N4_SESS_INACT_TIMER_FOR_UE_COMM"]
  }],
  "notificationURI": "http://localhost:9090/callback",
  "evtReq": {"notifMethod": "PERIODIC", "repPeriod": 10}
}
```

### Notification
```json
{
  "subscriptionId": "xxx",
  "eventNotifications": [{
    "event": "UE_COMMUNICATION",
    "ueComms": [{
      "commDur": 300,
      "ts": "2026-01-11T22:30:00Z",
      "trafChar": {"dnn": "internet", "ulVol": 1024000, "dlVol": 5120000},
      "sessInactTimer": {"n4SessId": 1, "sessInactiveTimer": 120}
    }]
  }]
}
```
