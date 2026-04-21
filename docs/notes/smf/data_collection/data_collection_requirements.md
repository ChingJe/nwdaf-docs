# Data Collection Module Requirements Report

**Date**: 2026-01-13  
**Focus**: UE Communication Analytics (General)

---

## 1. Data Requirements from SMF/UPF

### 1.1 Required Fields (Stage 2: TS 23.288 §6.7.3.2)

| Field | Source | Description | Priority |
|-------|--------|-------------|----------|
| `UE ID` (SUPI) | SMF | UE identifier | Required |
| `Group ID` | SMF | Internal Group ID | Optional |
| `S-NSSAI` | SMF | Network slice identifier | Required |
| `DNN` | SMF | Data Network Name | Required |
| `Application ID` | SMF | Application identifier | Optional |
| `UE Communication` | UPF (via SMF) | Communication patterns | Required |
| > `Communication start/stop` | UPF | Timestamps | Required |
| > `UL/DL data rate` | UPF | Data rates | Required |
| > `Traffic volume` | UPF | Volume UL/DL | Required |
| `UE session behaviour trends` | SMF | Session state transitions | Optional |
| `PDU Session ID` | SMF | Session identifier | Required |

### 1.2 Available OpenAPI Models

| Model | Path | Key Fields |
|-------|------|------------|
| `NsmfEventExposure` | `model_nsmf_event_exposure.go` | `supi`, `groupId`, `anyUeInd`, `dnn`, `snssai`, `eventSubs` |
| `SmfEventExposureEventSubscription` | `model_smf_event_exposure_event_subscription.go` | `event`, `appIds`, `targetPeriod` |
| `SmfEventExposureEventNotification` | `model_smf_event_exposure_event_notification.go` | `supi`, `dnn`, `snssai`, `pduSeId`, `timeStamp` |

---

## 2. Interface Specification

### 2.1 Service: Nsmf_EventExposure (TS 29.508)

**Purpose**: Subscribe to SMF events for UE session and communication data.

**Operations**:
- `POST /nsmf-event-exposure/v1/subscriptions` - Create subscription
- `DELETE /nsmf-event-exposure/v1/subscriptions/{subId}` - Delete subscription

### 2.2 Subscription Request

```go
type NsmfEventExposure struct {
    Supi       string   // Target UE (IMSI) - single UE
    GroupId    string   // Internal Group ID - multiple UEs
    AnyUeInd   bool     // true = any UE (no specific target)
    Dnn        string   // Filter by DNN
    Snssai     *Snssai  // Filter by S-NSSAI
    NotifUri   string   // Callback URL for NWDAF
    NotifId    string   // Correlation ID
    EventSubs  []SmfEventExposureEventSubscription
}
```

### 2.3 Relevant SMF Events

| SmfEvent | Description | Data Collected |
|----------|-------------|----------------|
| `PDU_SES_EST` | PDU session established | Session start, DNN, S-NSSAI |
| `PDU_SES_REL` | PDU session released | Session end time |
| `QOS_MON` | QoS monitoring | UL/DL delays, data rates |
| `UP_STATUS_INFO` | User plane status | Session activity |
| `DISPERSION` | Traffic dispersion | Traffic patterns |

### 2.4 Notification Response Fields

```go
type SmfEventExposureEventNotification struct {
    Event      SmfEvent    // Event type
    TimeStamp  *time.Time  // Event timestamp
    Supi       string      // UE ID
    Dnn        string      // Data Network Name
    Snssai     *Snssai     // Network Slice
    PduSeId    int32       // PDU Session ID
    AccType    AccessType  // Access type (3GPP/Non-3GPP)
    // For QOS_MON event:
    UlDelays   []int32     // Uplink delays
    DlDelays   []int32     // Downlink delays
}
```

---

## 3. Data Flow

### 3.1 Option A: Notification via SMF (Standard)

```
┌─────────┐  1. Subscribe   ┌───────┐  2. Subscribe N4  ┌───────┐
│  NWDAF  │ ───────────────►│  SMF  │ ─────────────────►│  UPF  │
│         │                 │       │                   │       │
│         │◄───────────────┐│       │◄─────────────────┐│       │
│         │  4. Notify     ││       │  3. Report       ││       │
└─────────┘                └───────┘                   └───────┘
```

### 3.2 Option B: Direct UPF Notification (Supported)

```
┌─────────┐  1. Subscribe   ┌───────┐  2. Subscribe N4  ┌───────┐
│  NWDAF  │ ───────────────►│  SMF  │ ─────────────────►│  UPF  │
│         │                 │       │   (notifUri=NWDAF)│       │
│         │◄────────────────────────────────────────────┤       │
│         │        3. Notify (directly from UPF)        │       │
└─────────┘                                             └───────┘
```

> **Note**: SMF can configure UPF to send notifications directly to NWDAF's notifUri.

---

## 4. Multi-UE Support Strategies

### 4.1 Options

| Strategy | Mechanism | Pros | Cons |
|----------|-----------|------|------|
| **Per-UE Subscription** | Create one subscription per SUPI | Simple, precise | Many subscriptions, high overhead |
| **Group Subscription** | Use `groupId` | Efficient for known groups | Requires group setup in UDM |
| **Any UE** | Set `anyUeInd=true` | Collect all UE data | Large data volume, filter needed |
| **DNN/S-NSSAI Filter** | Use `dnn` + `snssai` filters | Balance scope/efficiency | May still get unwanted UEs |

### 4.2 Recommendation

For our use case (UE_COMMUNICATION with specified SUPIs):
1. **Phase 1**: Use per-UE subscriptions (simple, direct control)
2. **Future**: Consider `anyUeInd` + filter by subscription's `tgtUe.supis`

---

## 5. Data Aggregation Strategies

### 5.1 Options

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **No Aggregation** | Forward notifications immediately | Real-time analytics |
| **Time-based** | Aggregate over fixed intervals (e.g., 30s) | Periodic reporting |
| **Count-based** | Aggregate N events before notify | Batch processing |
| **Session-based** | Aggregate per PDU session lifecycle | Session analytics |

### 5.2 Recommendation

For `evtReq.repPeriod` based subscriptions:
1. Store incoming SMF notifications in memory
2. On repPeriod timer: aggregate stored data → generate analytics → notify consumer
3. Clear buffer after notification

---

## 6. Implementation Plan (No NRF)

### 6.1 Configuration

```yaml
dataCollection:
  smf:
    enabled: true
    endpoints:    # Static SMF list (no NRF)
      - "http://smf1:29502"
    subscriptionDuration: 3600  # seconds
```

### 6.2 Package Structure

```
internal/
├── collector/
│   ├── collector.go        # Interface & types
│   ├── smf_client.go       # Nsmf_EventExposure client
│   ├── notification_handler.go  # Receive SMF callbacks
│   └── data_store.go       # In-memory data storage
```

### 6.3 Required Components

| Component | Description |
|-----------|-------------|
| **SMF Client** | HTTP client for Nsmf_EventExposure API |
| **Subscription Manager** | Create/delete subscriptions on consumer subscribe |
| **Notification Endpoint** | HTTP handler for SMF callbacks (/collector/notify) |
| **Data Store** | Store collected data keyed by SUPI |

---

## 7. Input/Output Field Mapping

### From SMF Notification → NWDAF Output

| SMF Notification Field | NWDAF Output Field | Model |
|------------------------|-------------------|-------|
| `supi` | Match with `tgtUe.supis` | - |
| `dnn` | `trafChar.dnn` | `TrafficCharacterization` |
| `snssai` | `trafChar.snssai` | `TrafficCharacterization` |
| `timeStamp` | `ts` | `UeCommunication` |
| `pduSeId` | (internal tracking) | - |
| (calculated) duration | `commDur` | `UeCommunication` |

### Calculation Logic

```go
// When PDU_SES_EST received: store startTime
// When PDU_SES_REL received: commDur = endTime - startTime
```

---

## 8. Decisions Made

| Question | Decision | Rationale |
|----------|----------|-----------|
| SMF Discovery | Static configuration | No NRF in current phase |
| Multi-UE | Per-UE subscription | Simple, matches tgtUe.supis |
| Aggregation | Time-based (repPeriod) | Aligns with existing notification scheduler |
| UPF Notification | Via SMF (Option A) | Simpler, SMF handles N4 complexity |
