# SMF Event Exposure 資料蒐集需求分析

**日期**: 2026-01-13  
**目的**: 分析 NWDAF 如何透過 SMF/UPF Event Exposure 蒐集 UE Communication 所需資料

---

## 1. TS 23.288 Table 6.7.3.2-1 必要輸入資料

| 輸入資料 | 來源 | 說明 |
|---------|------|------|
| UE ID / Group ID | SMF, AF | SUPI (SMF), GPSI (AF) |
| S-NSSAI | SMF | 網路切片識別 |
| DNN | SMF | Data Network Name |
| Application ID | SMF, AF | 應用程式識別 |
| **UE communication** | **UPF**, AF | 通訊描述 (start/stop, volume) |
| > Communication start/stop | UPF, AF | 時間戳記 |
| > UL/DL data rate | UPF, AF | 資料速率 |
| > Traffic volume | UPF, AF | 流量大小 |
| UE session behaviour trends | SMF | Session 建立/釋放趨勢 |

**關鍵發現**: Traffic volume 和 Communication start/stop 來自 **UPF**，非 SMF 直接提供。

---

## 2. SMF Event Types (TS 29.508)

### 2.1 已驗證的 SmfEvent

| Event ID | 用途 | 回傳資料 |
|----------|------|---------|
| `PDU_SES_EST` | Session 建立 | supi, dnn, snssai, pduSeId, ratType, timeStamp |
| `PDU_SES_REL` | Session 釋放 | supi, pduSeId, timeStamp |
| `UP_STATUS_INFO` | UP 狀態變化 | pduSessInfos[].pduSessStatus (ACTIVATED/DEACTIVATED) |
| `UPF_EVENT` | UPF 事件訂閱 | 透過 upfEvents 指定具體事件 |
| `QOS_MON` | QoS 監控 | ulDelays, dlDelays, rtDelays |

### 2.2 UP_STATUS_INFO 詳細結構

```yaml
pduSessInfos:
  - pduSessId: 1
    sessInfo:
      n4SessId: "xxx"
      sessInactiveTimer: 120
      pduSessStatus: "ACTIVATED" | "DEACTIVATED"
```

**用途**: 追蹤 Activated ↔ Deactivated 轉換計算 `commDur`

---

## 3. UPF 直接回報機制 (TS 29.564)

### 3.1 訂閱流程

```
NWDAF → SMF: Nsmf_EventExposure_Subscribe (event: UPF_EVENT, upfEvents: [...])
SMF → UPF: 設定 Usage Reporting 規則
UPF → NWDAF: Nupf_EventExposure_Notify (直接回報)
```

### 3.2 UPF_EVENT 訂閱參數

根據 TS 29.508 EventSubscription:

```yaml
event: "UPF_EVENT"
upfEvents:
  - type: "USER_DATA_USAGE_MEASURES"
bundledEventNotifyUri: "https://nwdaf/collector/upf-notify"  # UPF 直接回報 URI
```

### 3.3 UPF 可回報的資料

| 欄位 | 說明 |
|------|------|
| `dataVol` (VolumeTimedReport) | UL/DL 流量大小 + 時間區間 |
| `ulDataRate`, `dlDataRate` | 即時速率 |
| `appId` | 偵測到的應用程式 |
| `timeWindow` | 量測時間視窗 |

### 3.4 DataVolumeInformation 結構 (TS 29.508)

```yaml
dataVolInfoData:
  - dataVol:
      startTimeStamp: "2026-01-13T10:00:00Z"
      endTimeStamp: "2026-01-13T10:05:00Z"
      downlinkVolume: 5120000  # bytes
      uplinkVolume: 1024000    # bytes
    upfIds:
      - upfId: "upf-1"
```

---

## 4. UPF Event Exposure 詳細規格 (TS 29.564)

### 4.1 EventType 列舉 (Nupf_EventExposure)

| EventType | 說明 | 用途 |
|-----------|------|------|
| `QOS_MONITORING` | QoS 監控量測 | 延遲、吞吐量 |
| `USER_DATA_USAGE_MEASURES` | 使用者資料用量量測 | **Traffic volume** |
| `USER_DATA_USAGE_TRENDS` | 使用者資料用量趨勢 | 資料速率統計 |
| `TSC_MNGT_INFO` | TSC 管理資訊 | 時間敏感通訊 |
| `UE_NAT_MAPPING_INFO` | UE NAT 映射資訊 | NAT 支援 |
| `SUBSCRIPTION_TERMINATION` | 訂閱終止通知 | Session 結束 |

### 4.2 UpfEvent Schema (TS 29.564)

```yaml
UpfEvent:
  type: object
  required:
    - type
  properties:
    type:
      $ref: '#/components/schemas/EventType'
    measurementTypes:              # 量測類型
      type: array
      items:
        enum: [VOLUME_MEASUREMENT, THROUGHPUT_MEASUREMENT, APPLICATION_RELATED_INFO]
    appIds:                        # 過濾指定 Application
      type: array
      items:
        $ref: 'ApplicationId'
    granularityOfMeasurement:      # 量測粒度
      enum: [PER_APPLICATION, PER_SESSION, PER_FLOW]
    reportingSuggestionInfo:       # 報告建議 (delay tolerant)
      $ref: '#/components/schemas/ReportingSuggestionInformation'
```

### 4.3 UserDataUsageMeasurements Schema

```yaml
UserDataUsageMeasurements:
  properties:
    appId:                         # 偵測到的應用程式
      $ref: 'ApplicationId'
    flowInfo:                      # 流量描述
      $ref: 'FlowInformation'
    volumeMeasurement:             # 流量大小
      $ref: '#/components/schemas/VolumeMeasurement'
    throughputMeasurement:         # 吞吐量
      $ref: '#/components/schemas/ThroughputMeasurement'
    applicationRelatedInformation: # 應用相關資訊 (URL, domain)
      $ref: '#/components/schemas/ApplicationRelatedInformation'
```

### 4.4 VolumeMeasurement Schema

```yaml
VolumeMeasurement:
  properties:
    totalVolume:      # 總流量
      $ref: 'TrafficVolume'
    ulVolume:         # 上行流量
      $ref: 'TrafficVolume'
    dlVolume:         # 下行流量
      $ref: 'TrafficVolume'
    totalNbOfPackets: # 總封包數
      $ref: 'Uint64'
    ulNbOfPackets:    # 上行封包數
      $ref: 'Uint64'
    dlNbOfPackets:    # 下行封包數
      $ref: 'Uint64'
```

### 4.5 NotificationItem Schema (UPF → NWDAF)

```yaml
NotificationItem:
  required:
    - eventType
    - timeStamp
  anyOf:
    - required: [ueIpv4Addr]
    - required: [ueIpv6Prefix]
  properties:
    eventType:                     # 事件類型
      $ref: '#/components/schemas/EventType'
    supi:                          # UE ID
      $ref: 'Supi'
    dnn:                           # DNN
      $ref: 'Dnn'
    snssai:                        # S-NSSAI
      $ref: 'Snssai'
    timeStamp:                     # 時間戳記
      $ref: 'DateTime'
    startTime:                     # 量測開始時間
      $ref: 'DateTime'
    userDataUsageMeasurements:     # 用量資料
      type: array
      items:
        $ref: '#/components/schemas/UserDataUsageMeasurements'
```

---

## 5. TS 23.502 §4.15.4.5 程序說明

### 5.1 訂閱選項

| 選項 | 說明 |
|------|------|
| **Via SMF** | NWDAF → SMF → UPF, UPF 直接回報 NWDAF |
| **Direct to UPF** | NWDAF 直接訂閱 UPF (需知道 UE IP) |

### 5.2 Via SMF 流程

```
1. NWDAF → NRF: Discover SMF
2. NWDAF → SMF: Nsmf_EventExposure_Subscribe (UPF_EVENT)
3. SMF → UPF: Nupf_EventExposure_Subscribe 或 N4 Session Modification
4. UPF → NWDAF: Nupf_EventExposure_Notify (直接)
```

### 5.3 Input Parameters (Table 4.15.4.5.1-1)

| 參數 | 說明 |
|------|------|
| `UE identification` | SUPI 或 UE IP |
| `Any UE` | 訂閱任意 UE |
| `DNN`, `S-NSSAI` | 過濾條件 |
| `Type of Measurement` | VOLUME_MEASUREMENT 等 |
| `Granularity of Measurement` | PER_SESSION / PER_APPLICATION |
| `Application ID` | 指定應用 |
| `Reporting suggestion information` | delay tolerant 設定 |

---

## 6. 建議訂閱策略

### 6.1 基本訂閱 (SMF 回報)

```json
{
  "notifUri": "https://nwdaf/collector/notify",
  "notifId": "nwdaf-sub-001",
  "supi": "imsi-208930000000003",
  "eventSubs": [
    { "event": "PDU_SES_EST" },
    { "event": "PDU_SES_REL" },
    { "event": "UP_STATUS_INFO" }
  ]
}
```

**取得**: Session metadata (DNN, S-NSSAI), session lifecycle, UP status

### 6.2 進階訂閱 (含 UPF 直接回報)

```json
{
  "eventSubs": [
    { "event": "PDU_SES_EST" },
    { "event": "PDU_SES_REL" },
    { "event": "UP_STATUS_INFO" },
    {
      "event": "UPF_EVENT",
      "upfEvents": [
        {
          "type": "USER_DATA_USAGE_MEASURES",
          "measurementTypes": ["VOLUME_MEASUREMENT"],
          "granularityOfMeasurement": "PER_SESSION"
        }
      ],
      "bundledEventNotifyUri": "https://nwdaf/collector/upf-notify"
    }
  ]
}
```

**額外取得**: Traffic volume (UL/DL), Application ID

---

## 7. 實作建議

### 7.1 需新增的端點

| 端點 | 用途 |
|------|------|
| `POST /collector/notify` | 接收 SMF 通知 (已實作) |
| `POST /collector/upf-notify` | 接收 UPF 直接通知 (**需新增**) |

### 7.2 UPF 通知處理 (Nupf_EventExposure_Notify)

```go
// 處理 UPF 直接回報
type UpfNotificationItem struct {
    EventType                   string
    Supi                        string
    Dnn                         string
    TimeStamp                   time.Time
    UserDataUsageMeasurements   []UserDataUsageMeasurements
}

// 提取流量資料
for _, item := range notification.NotificationItems {
    for _, usage := range item.UserDataUsageMeasurements {
        ulVolume := usage.VolumeMeasurement.UlVolume
        dlVolume := usage.VolumeMeasurement.DlVolume
        appId := usage.AppId
    }
}
```

### 7.3 OpenAPI Models 需求

目前 `internal/openapi/models` 可能缺少:
- `UpfEvent` struct
- `UserDataUsageMeasurements` struct  
- `VolumeMeasurement` struct
- `NotificationItem` struct (Nupf)

可能需要從 TS 29.564 YAML 生成或手動實作。

---

## 8. YAML 規格驗證 (已確認)

### 8.1 TS 29.508 EventSubscription 結構

```yaml
# TS29508_Nsmf_EventExposure.yaml (line 510-528)
EventSubscription:
  properties:
    event:
      $ref: '#/components/schemas/SmfEvent'
    upfEvents:                      # ⚠️ OpenAPI 缺少此欄位
      type: array
      items:
        $ref: 'TS29564_Nupf_EventExposure.yaml#/components/schemas/UpfEvent'
    bundledEventNotifyUri:          # ⚠️ OpenAPI 缺少此欄位
      $ref: 'TS29571_CommonData.yaml#/components/schemas/Uri'
    bundlingAllowed:
      type: boolean
  required:
    - event
```

### 8.2 OpenAPI Gap 分析

| YAML 欄位 | 說明 | OpenAPI 狀態 |
|-----------|------|-------------|
| `EventSubscription.upfEvents` | UPF 事件列表 | ❌ **缺少** |
| `EventSubscription.bundledEventNotifyUri` | UPF 直接回報 URI | ❌ **缺少** |
| `EventSubscription.bundlingAllowed` | 允許批次通知 | ❌ **缺少** |
| `UpfEvent` (TS 29.564) | UPF 事件定義 | ❌ **完全缺少** |
| `UserDataUsageMeasurements` | 用量量測 | ❌ **完全缺少** |
| `VolumeMeasurement` | 流量量測 | ❌ **完全缺少** |

### 8.3 現有 OpenAPI models (已確認)

```
.agent/openapi/openapi/models/
├── model_smf_event_exposure_event_subscription.go  # 缺少 upfEvents
├── model_upf_information.go                        # 只有 UpfId, UpfAddr
└── (無 UpfEvent 相關)
```

`SmfEventExposureEventSubscription` 現有欄位:
- `Event`, `DnaiChgType`, `DddTraDescriptors`, `DddStati`, `AppIds`, 
- `TargetPeriod`, `TransacDispInd`, `TransacMetrics`, `UeIpAddr`
- **缺少**: `upfEvents`, `bundledEventNotifyUri`, `bundlingAllowed`

---

## 9. 筆記正確性驗證

| 筆記內容 | 驗證結果 |
|---------|---------|
| UP_STATUS_INFO 提供 pduSessStatus | ✅ 正確 (PduSessionStatus: ACTIVATED/DEACTIVATED) |
| UPF_EVENT 由 UPF 直接回報 | ✅ 正確 (TS 29.564 + bundledEventNotifyUri) |
| 流量資料不在 SMF 通知中 | ✅ 正確 (需透過 DataVolumeInformation 或 UPF 回報) |
| PDU_SES_EST/REL 提供 Session lifecycle | ✅ 正確 |
| **upfEvents 欄位定義** | ✅ 已驗證 (TS 29.508 line 510-515) |
| **bundledEventNotifyUri 欄位** | ✅ 已驗證 (TS 29.508 line 527-528) |

---

## 10. 結論

1. **SMF 可直接提供**: UE ID, S-NSSAI, DNN, Session lifecycle, UP status
2. **需透過 UPF_EVENT**: Traffic volume, Application ID, Data rate
3. **UPF 直接回報**: 訂閱 UPF_EVENT 時可設定 `bundledEventNotifyUri` 讓 UPF 直接通知 NWDAF
4. **OpenAPI 缺口**: 需新增/擴充 `SmfEventExposureEventSubscription` 及 TS 29.564 相關 models
5. **實作優先級**:
   - Phase 1: PDU_SES_EST, PDU_SES_REL, UP_STATUS_INFO (SMF) ✅ 已完成
   - Phase 2: UPF_EVENT + UPF 直接回報 endpoint (需額外端點 + OpenAPI 擴充)
