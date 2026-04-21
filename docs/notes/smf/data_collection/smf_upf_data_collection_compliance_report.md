# SMF/UPF Data Collection Compliance Report

**Date**: 2026-01-15
**Spec Versions**:
- TS 29.508 V19.4.0 (Nsmf_EventExposure)
- TS 29.564 V19.5.0 (Nupf_EventExposure)

---

## Executive Summary

| Category | Status |
|----------|--------|
| Subscription Request (Client) | ⚠️ Partial |
| Notification Handling | ✅ Compliant |
| Data Storage | ✅ Compliant |
| Model Definitions | ⚠️ Partial |

---

## 1. Subscription Request (NsmfEventExposure)

### 1.1 YAML Schema Reference (TS 29.508)

```yaml
NsmfEventExposure:
  required:
    - notifId
    - notifUri
    - eventSubs
  properties:
    supi, gpsi, anyUeInd, groupId, pduSeId, dnn, snssai...
    eventSubs: [EventSubscription]
```

### 1.2 Implementation vs Spec

| Field | Spec | Implementation | Status |
|-------|------|----------------|--------|
| `notifId` | required | ✅ `uuid.New().String()` | ✅ |
| `notifUri` | required | ✅ Passed as parameter | ✅ |
| `eventSubs` | required | ✅ Constructed | ✅ |
| `supi` | optional | ✅ Set in request | ✅ |
| `gpsi` | optional | ❌ Not implemented | ⚪ |
| `dnn`/`snssai` | optional | ❌ Not set in request | ⚠️ |
| `anyUeInd` | optional | ❌ Not implemented | ⚪ |

**Implementation**: [smf_service.go](file:///home/x81u/NYCU/lab/NWDAF/internal/sbi/consumer/smf_service.go#L119-L175)

---

## 2. EventSubscription (UPF_EVENT)

### 2.1 YAML Schema (TS 29.508 L510-537)

```yaml
EventSubscription:
  required:
    - event
  properties:
    event: SmfEvent
    upfEvents: [UpfEvent]  # TS 29.564 reference
    bundlingAllowed: boolean (only true)
    bundledEventNotifyUri: Uri
```

### 2.2 Implementation vs Spec

| Field | Spec | Implementation | Status |
|-------|------|----------------|--------|
| `event` | required | ✅ `SmfEvent_UPF_EVENT` | ✅ |
| `upfEvents` | optional | ✅ Constructed with type, measurementTypes | ✅ |
| `bundlingAllowed` | only true | ✅ Set to `true` | ✅ |
| `bundledEventNotifyUri` | optional | ✅ Passed as `upfNotifUri` | ✅ |

**Implementation**: [models.go](file:///home/x81u/NYCU/lab/NWDAF/internal/sbi/consumer/models.go#L19-L27)

---

## 3. UpfEvent (TS 29.564)

### 3.1 YAML Schema (TS 29.564 L420-470)

```yaml
UpfEvent:
  required:
    - type
  properties:
    type: EventType (USER_DATA_USAGE_MEASURES, QOS_MONITORING, ...)
    measurementTypes: [VOLUME_MEASUREMENT, THROUGHPUT_MEASUREMENT, ...]
    granularityOfMeasurement: PER_SESSION | PER_FLOW | PER_APPLICATION
```

### 3.2 Implementation vs Spec

| Field | Spec | Implementation | Status |
|-------|------|----------------|--------|
| `type` | required | ✅ `USER_DATA_USAGE_MEASURES` | ✅ |
| `measurementTypes` | optional | ✅ `[VOLUME, THROUGHPUT]` | ✅ |
| `granularityOfMeasurement` | optional | ✅ `PER_SESSION` | ✅ |
| `appIds` | optional | ❌ Not implemented | ⚪ |
| `trafficFilters` | optional | ❌ Not implemented | ⚪ |

**Implementation**: [models.go](file:///home/x81u/NYCU/lab/NWDAF/internal/sbi/consumer/models.go#L29-L57)

---

## 4. Notification Handling (NotificationItem)

### 4.1 YAML Schema (TS 29.564 L279-336)

```yaml
NotificationItem:
  required:
    - eventType
    - timeStamp
  anyOf:
    - required: [ueIpv4Addr]
    - required: [ueIpv6Prefix]
    - required: [ueMacAddr]
  properties:
    supi, dnn, snssai, ratType, userDataUsageMeasurements...
```

### 4.2 Implementation vs Spec

| Field | Spec | Implementation | Status |
|-------|------|----------------|--------|
| `eventType` | required | ✅ Checked in switch | ✅ |
| `timeStamp` | required | ⚠️ Parsed but not used | ⚠️ |
| `supi` | conditional | ✅ Required in handler | ✅ |
| `ueIpv4Addr` | anyOf | ✅ Logged if no SUPI | ✅ |
| `dnn` | optional | ✅ Stored | ✅ |
| `snssai` | optional | ✅ Stored | ✅ |
| `ratType` | optional | ✅ Stored | ✅ |
| `userDataUsageMeasurements` | optional | ✅ Processed | ✅ |

**Implementation**: [upf_notify.go](file:///home/x81u/NYCU/lab/NWDAF/internal/sbi/processor/upf_notify.go#L58-L134)

---

## 5. Volume/Throughput Measurement

### 5.1 YAML Schema (TS 29.564 L579-607)

```yaml
VolumeMeasurement:
  properties:
    totalVolume: TrafficVolume (int64)
    ulVolume: TrafficVolume
    dlVolume: TrafficVolume
    totalNbOfPackets, ulNbOfPackets, dlNbOfPackets: Uint64

ThroughputMeasurement:
  properties:
    ulThroughput: BitRate (string)
    dlThroughput: BitRate
    ulPacketThroughput, dlPacketThroughput: PacketRate
```

### 5.2 Implementation vs Spec

| Field | Spec Type | Implementation | Status |
|-------|-----------|----------------|--------|
| `ulVolume` | int64 | ✅ `int64` | ✅ |
| `dlVolume` | int64 | ✅ `int64` | ✅ |
| `totalVolume` | int64 | ⚠️ Defined but not used | ⚠️ |
| `totalNbOfPackets` | int64 | ❌ Not implemented | ⚪ |
| `ulThroughput` | string (BitRate) | ✅ `string` | ✅ |
| `dlThroughput` | string (BitRate) | ✅ `string` | ✅ |
| `ulPacketThroughput` | string | ❌ Not implemented | ⚪ |

**Implementation**: [upf_notify.go](file:///home/x81u/NYCU/lab/NWDAF/internal/sbi/processor/upf_notify.go#L45-L56)

---

## 6. Intentional Deviations

| Decision | Rationale |
|----------|-----------|
| SMF 訂閱調用已註解 | SMF 連線未建立，待整合時啟用 |
| 無 `gpsi`/`anyUeInd` | 目前僅支援 SUPI 識別 |
| 無封包計數 | 專注於流量量測，非封包層級 |
| `timeStamp` 未完全利用 | 未用於計算通訊持續時間 |

---

## 7. Implementation Gaps

| Priority | Gap | Recommendation |
|----------|-----|----------------|
| 🔴 High | SMF 訂閱未實際發送 | 待 SMF 整合後啟用 |
| 🟡 Medium | `dnn`/`snssai` 未在訂閱中設定 | 可從 tgtUe 或 config 取得 |
| 🟢 Low | 未實作封包計數 | 依需求添加 |

---

## 8. Verification Checklist

- [x] Go struct tags match YAML property names (camelCase)
- [x] Enum values match YAML: `USER_DATA_USAGE_MEASURES`, `VOLUME_MEASUREMENT`
- [x] Mandatory fields enforced: `event` in subscription
- [x] ProblemDetails used for errors (in smf_service.go)
- [x] Location header parsing implemented correctly
