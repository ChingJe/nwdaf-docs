# UE Communication Compliance Report

**Date**: 2026-01-11  
**Spec Versions**: TS 23.288 V17.x (Stage 2), TS 29.520 V17.10.0 (Stage 3)

---

## Executive Summary

| Category | Status |
|----------|--------|
| Subscription Input Validation | ✅ Partially Compliant |
| Notification Output | ✅ Partially Compliant |
| Data Collection | ❌ Not Implemented (Mock Data) |

---

## 1. Shortcut Scenario: N4 Session Inactivity Timer

### Definition
We are implementing a "Shortcut" for the `N4_SESS_INACT_TIMER_FOR_UE_COMM` subset without full `EneNA` feature support.
- **Scope**: Only `N4_SESS_INACT_TIMER_FOR_UE_COMM` subset is officially supported.
- **Requirement**: `listOfAnaSubsets` must be validated to ensure it targets this specific subset.
- **Assumption**: `EneNA` feature is conceptually active for this shortcut.

---

## 2. Stage 2 Compliance (TS 23.288 §6.7.3)

### §6.7.3.1 Subscription Input

| Requirement | Spec Reference | Implementation | Status |
|-------------|----------------|----------------|--------|
| Analytics ID = "UE Communication" | §6.7.3.1 | ✅ `NwdafEvent_UE_COMMUNICATION` | ✅ |
| Target: SUPI or Internal-Group-ID | §6.7.3.1 | ✅ `validateUeCommunication` checks `supis` or `intGroupIds` | ✅ |
| Filter: S-NSSAI | §6.7.3.1 | ⚠️ Accepted but not processed | ⚠️ |
| Filter: DNN | §6.7.3.1 | ⚠️ Accepted but not processed | ⚠️ |
| Filter: Application ID | §6.7.3.1 | ⚠️ Accepted but not processed | ⚠️ |
| Filter: Area of Interest | §6.7.3.1 | ⚠️ Accepted but not processed | ⚠️ |
| Analytics subset list | §6.7.3.1 | ⚠️ Accepted but not validated | ⚠️ |

### §6.7.3.2 Data Collection

| Input Data | Source | Implementation | Status |
|------------|--------|----------------|--------|
| UE Communication from SMF/UPF | §6.7.3.2 Table | ❌ Mock data used | ❌ |
| Traffic volume (UL/DL) | §6.7.3.2 Table | ✅ Mock values: `ulVol=1MB, dlVol=5MB` | Mock |
| UE session behaviour trends | §6.7.3.2 Table | ❌ Not collected | ❌ |

### §6.7.3.3 Output Analytics

| Output Field | Spec Requirement | Implementation | Status |
|--------------|------------------|----------------|--------|
| UE Communications | Table 6.7.3.3-1 | ✅ `ueComms` array | ✅ |
| Periodic communication indicator | Table 6.7.3.3-1 | ❌ Not in mock | ❌ |
| Periodic time | Table 6.7.3.3-1 | ⚪ `perioTime` exists in model but not set | ❌ |
| Start time / Duration | Table 6.7.3.3-1 | ✅ `ts`, `commDur` | ✅ |
| Traffic characterization | Table 6.7.3.3-1 | ✅ `trafChar` with DNN, ulVol, dlVol | ✅ |
| N4 Session ID + Inactivity Timer | Table 6.7.3.3-1 NOTE 2 | ✅ `sessInactTimer` | ✅ |
| Confidence | Table 6.7.3.3-1 | ✅ `confidence=90` | ✅ |

> **NOTE 2 Gap**: Spec says N4 Session ID "shall only be included if the consumer is SMF". Current implementation always includes it regardless of consumer type.

---

## 3. Stage 3 Compliance (TS 29.520 §4.2)

### §4.2 Subscription Requirements

| Requirement | Spec Line | Implementation | Status |
|-------------|-----------|----------------|--------|
| Feature `UeCommunication` supported | §4.2 line 360 | ✅ Event accepted | ✅ |
| `tgtUe` with `supis` or `intGroupIds` | §4.2 line 361 | ✅ Validated | ✅ |
| Optional `appIds` | §4.2 line 364 | ⚠️ Accepted, not used | ⚠️ |
| Optional `networkArea` | §4.2 line 365 | ⚠️ Accepted, not used | ⚠️ |
| Optional `dnns` | §4.2 line 366 | ⚠️ Accepted, not used | ⚠️ |
| Optional `snssais` | §4.2 line 367 | ⚠️ Accepted, not used | ⚠️ |
| Optional `listOfAnaSubsets` (EneNA feature) | §4.2 line 368 | ❌ Accepted but ignored (Critical Gap for Shortcut) | ❌ |
| Optional `ueCommReqs` (UeCommunicationExt_eNA) | §4.2 line 369 | ❌ Not supported | ❌ |

### §4.2 Notification Requirements

| Requirement | Spec Line | Implementation | Status |
|-------------|-----------|----------------|--------|
| `ueComms` attribute | §4.2 line 644 | ✅ `UeComms` array in notification | ✅ |

---

## 4. Intentional Deviations

| Decision | Rationale |
|----------|-----------|
| **Mock data instead of real collection** | Data collection from SMF/UPF requires `Nsmf_EventExposure` client, which is not yet implemented. |
| **N4 Session ID always included** | Spec requires it only for SMF consumers, but our implementation includes it unconditionally for testing visibility. |
| **Optional filters not processed** | DNN, S-NSSAI, appIds are accepted but do not affect mock data output. This is acceptable for Phase 4 MVP. |
| **Partial EneNA Support** | We accept `listOfAnaSubsets` for the N4 Session Inactivity Timer shortcut without full feature negotiation. |
| **Filtered Subsets** | We may reject or ignore other standard subsets (like `APP_LIST_FOR_UE_COMM`) to focus strictly on the `N4_SESS_INACT_TIMER` shortcut. |

---

## 5. Implementation Gaps (Future Work)

### Priority 0: Shortcut Requirements (Critical)
- [x] Validate `listOfAnaSubsets` contains `N4_SESS_INACT_TIMER_FOR_UE_COMM` if provided.
- [x] Conditionally output `sessInactTimer` only when requested.

### Priority 1: Data Collection
- [ ] Implement `Nsmf_EventExposure_Subscribe` client
- [ ] Collect real UE communication data from UPF

### Priority 2: Filter Processing
- [ ] Apply `dnns` filter to output
- [ ] Apply `snssais` filter to output
- [ ] Apply `appIds` filter to output

### Priority 3: Output Completeness
- [ ] Add `perioComm` indicator
- [ ] Add `perioTime` and `perioTimeVariance`
- [ ] Conditional N4 Session ID based on consumer type

---

## 6. Test Coverage

| Test Type | Count | Coverage |
|-----------|-------|----------|
| Unit tests (validateUeCommunication) | 5 | ✅ |
| Unit tests (generateMockUeCommunication) | 0 | ❌ |
| API tests (uecomm, uecomm-invalid) | 2 | ✅ |
