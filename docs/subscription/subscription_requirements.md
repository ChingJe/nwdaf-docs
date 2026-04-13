# NWDAF EventSubscription 完整功能需求報告

本報告根據你提供的規範來源（TS 29.520 YAML、TS 23.520 筆記），詳細列出 EventSubscription 服務需要實作的所有功能。

---

## 目錄

1. [API 端點總覽](#1-api-端點總覽)
2. [資料結構需求](#2-資料結構需求)
3. [ABNORMAL_BEHAVIOUR 事件詳細需求](#3-abnormal_behaviour-事件詳細需求)
4. [Notification 機制](#4-notification-機制)
5. [錯誤處理](#5-錯誤處理)
6. [Feature 協商](#6-feature-協商)
7. [實作優先級建議](#7-實作優先級建議)

---

## 1. API 端點總覽

### 1.1 必須實作的端點

| 資源 | URI | 方法 | 說明 | 目前狀態 |
|------|-----|------|------|---------|
| **Subscriptions** | `/subscriptions` | POST | 建立訂閱 | ✅ 已實作 |
| **Individual Subscription** | `/subscriptions/{subscriptionId}` | PUT | 更新訂閱 | ✅ 已實作 |
| | | DELETE | 刪除訂閱 | ✅ 已實作 |

### 1.2 可選端點（進階功能）

| 資源 | URI | 方法 | 說明 | 目前狀態 |
|------|-----|------|------|---------|
| **Transfers** | `/transfers` | POST | 訂閱轉移請求 | ❌ 未實作 |
| **Individual Transfer** | `/transfers/{transferId}` | PUT | 更新轉移 | ❌ 未實作 |
| | | DELETE | 取消轉移 | ❌ 未實作 |

> **備註**：Transfer API 用於 NWDAF 間的訂閱轉移，屬於 `AnaSubTransfer` Feature，優先級較低。

---

## 2. 資料結構需求

### 2.1 NnwdafEventsSubscription（請求/回應主體）

| 欄位 | 型別 | 必填 | 說明 | 目前狀態 |
|------|------|------|------|---------|
| `eventSubscriptions` | array(EventSubscription) | **M** | 訂閱的事件清單（至少 1 個）| ✅ 有驗證 |
| `notificationURI` | Uri | **C** | 通知回呼 URI（POST/PUT 必填）| ✅ 有驗證 |
| `evtReq` | ReportingInformation | O | 報告設定（週期、方法等）| ⚠️ 有儲存，未處理 |
| `notifCorrId` | string | O | 通知關聯 ID | ⚠️ 有儲存 |
| `supportedFeatures` | SupportedFeatures | C | Feature 協商 | ❌ 未實作 |
| `eventNotifications` | array(EventNotification) | C | 即時報告（若 immRep=true）| ❌ 未實作 |
| `failEventReports` | array(FailureEventInfo) | O | 失敗事件報告 | ❌ 未實作 |
| `prevSub` | PrevSubInfo | O | 前一個訂閱資訊 | ❌ 未實作 |
| `consNfInfo` | ConsumerNfInformation | O | 消費者 NF 資訊 | ❌ 未實作 |

### 2.2 EventSubscription（單一事件訂閱）

#### 2.2.1 通用欄位

| 欄位 | 型別 | 必填 | 說明 | 目前狀態 |
|------|------|------|------|---------|
| `event` | NwdafEvent | **M** | 事件類型（如 ABNORMAL_BEHAVIOUR）| ✅ 有驗證 |
| `notificationMethod` | NotificationMethod | O | 通知方法（PERIODIC/THRESHOLD/ONE_TIME）| ⚠️ 有儲存 |
| `repetitionPeriod` | DurationSec | C | 週期性通知間隔（秒）| ⚠️ 有儲存 |
| `tgtUe` | TargetUeInformation | O | 目標 UE 資訊（supis/intGroupIds/anyUe）| ✅ 有驗證 |
| `networkArea` | NetworkAreaInfo | C | 網路區域（某些事件必填）| ⚠️ 有儲存 |
| `snssais` | array(Snssai) | C | 網路切片 ID | ⚠️ 有儲存 |
| `dnns` | array(Dnn) | C | DNN 識別 | ⚠️ 有儲存 |
| `appIds` | array(ApplicationId) | C | 應用識別 | ⚠️ 有儲存 |
| `extraReportReq` | EventReportingRequirement | O | 額外報告需求 | ⚠️ 有儲存 |
| `listOfAnaSubsets` | array(AnalyticsSubset) | O | 分析子集 | ⚠️ 有儲存 |

#### 2.2.2 ABNORMAL_BEHAVIOUR 專屬欄位

| 欄位 | 型別 | 必填 | 說明 | 目前狀態 |
|------|------|------|------|---------|
| `excepRequs` | array(Exception) | C | Exception ID 與閾值清單 | ✅ 有驗證存在 |
| `exptAnaType` | ExpectedAnalyticsType | C | 預期分析類型（MOBILITY/COMMUN）| ✅ 有驗證存在 |
| `exptUeBehav` | ExpectedUeBehaviourData | O | 預期 UE 行為 | ⚠️ 有儲存 |

---

## 3. ABNORMAL_BEHAVIOUR 事件詳細需求

根據 **TS 23.520 Clause 4.2.2.2.2**，ABNORMAL_BEHAVIOUR 事件有以下特定規則：

### 3.1 必填條件

| 規則 | 說明 | 目前狀態 |
|------|------|---------|
| `tgtUe` 必須存在 | 必須包含 `supis`、`intGroupIds`、或 `anyUe=true` | ✅ 已實作 |
| `excepRequs` 或 `exptAnaType` 二擇一 | 必須提供其中之一 | ✅ 已實作 |
| `excepRequs` 與 `exptAnaType` 互斥 | **不可同時提供** | ❌ **未實作** |

### 3.2 anyUe=true 時的額外條件

| 條件 | 說明 | 目前狀態 |
|------|------|---------|
| 若為 mobility 相關 | 需提供 `networkArea` 或 `snssais` | ❌ **未實作** |
| 若為 commun 相關 | 需提供 `networkArea`、`appIds`、`dnns`、或 `snssais` 之一 | ❌ **未實作** |
| 不可同時請求 mobility 與 commun | 若 anyUe=true，不可混合 | ❌ **未實作** |

### 3.3 exptAnaType 對應 ExceptionId 映射

當使用 `exptAnaType` 時，NWDAF 需自動推導對應的 Exception IDs：

| exptAnaType | 對應 Exception IDs |
|-------------|-------------------|
| MOBILITY | UNEXPECTED_UE_LOCATION, PING_PONG_ACROSS_CELLS, UNEXPECTED_WAKEUP, UNEXPECTED_RADIO_LINK_FAILURES |
| COMMUN | UNEXPECTED_LONG_LIVE_FLOW, UNEXPECTED_LARGE_RATE_FLOW, **SUSPICION_OF_DDOS_ATTACK**, WRONG_DESTINATION_ADDRESS, TOO_FREQUENT_SERVICE_ACCESS |
| MOBILITY_AND_COMMUN | 上述全部 |

**實作狀態**：❌ **未實作映射邏輯**

### 3.4 Exception IDs（可用於 DDoS 偵測）

```
UNEXPECTED_UE_LOCATION         - 非預期 UE 位置
UNEXPECTED_LONG_LIVE_FLOW      - 非預期長連線
UNEXPECTED_LARGE_RATE_FLOW     - 非預期大流量
UNEXPECTED_WAKEUP              - 非預期喚醒
SUSPICION_OF_DDOS_ATTACK       - DDoS 攻擊嫌疑 ⭐
WRONG_DESTINATION_ADDRESS      - 錯誤目標地址
TOO_FREQUENT_SERVICE_ACCESS    - 過於頻繁的服務存取
UNEXPECTED_RADIO_LINK_FAILURES - 非預期無線連結失敗
PING_PONG_ACROSS_CELLS         - 跨 Cell 乒乓效應
```

---

## 4. Notification 機制

### 4.1 概述

訂閱成功後，NWDAF 需主動發送通知到 `notificationURI`。

**實作狀態**：❌ **完全未實作**

### 4.2 通知觸發條件

| 通知方法 | 觸發時機 |
|---------|---------|
| `PERIODIC` | 每隔 `repetitionPeriod` 秒發送一次 |
| `ONE_TIME` | 訂閱後立即發送一次 |
| `THRESHOLD` | 當超過閾值時發送 |
| `ON_EVENT_DETECTION` | 當偵測到事件時發送 |

### 4.3 通知資料結構 (NnwdafEventsSubscriptionNotification)

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `subscriptionId` | string | **M** | 訂閱 ID |
| `eventNotifications` | array(EventNotification) | C | 事件通知內容 |
| `notifCorrId` | string | O | 關聯 ID |
| `termCause` | TermCause | O | 終止原因 |

### 4.4 EventNotification 結構（ABNORMAL_BEHAVIOUR）

| 欄位 | 說明 |
|------|------|
| `event` | 事件類型 = "ABNORMAL_BEHAVIOUR" |
| `abnorBehavrs` | array(AbnormalBehaviour) - 異常行為資訊 |
| `start` | 統計開始時間 |
| `expiry` | 有效期限 |
| `timeStampGen` | 生成時間戳 |

### 4.5 AbnormalBehaviour 結構

```json
{
  "supis": ["imsi-123456789012345"],
  "excep": {
    "excepId": "SUSPICION_OF_DDOS_ATTACK",
    "excepLevel": 3,
    "excepTrend": "UP"
  },
  "ratio": 85,
  "confidence": 90,
  "addtMeasInfo": {
    "numOfUeNotifs": 1000,
    "svcExpInfos": [...]
  }
}
```

### 4.6 通知發送方式

```
POST {notificationURI}
Content-Type: application/json

[
  {
    "subscriptionId": "xxx-xxx-xxx",
    "eventNotifications": [{
      "event": "ABNORMAL_BEHAVIOUR",
      "abnorBehavrs": [...]
    }]
  }
]
```

**預期回應**：`204 No Content`

---

## 5. 錯誤處理

### 5.1 必須支援的錯誤碼

| HTTP Status | Cause | 說明 | 目前狀態 |
|-------------|-------|------|---------|
| 400 | INVALID_REQUEST | 請求格式錯誤 | ✅ 已實作 |
| 400 | INVALID_JSON | JSON 解析失敗 | ✅ 已實作 |
| 400 | BOTH_STAT_PRED_NOT_ALLOWED | 不允許同時請求統計與預測 | ❌ 未實作 |
| 400 | PREDICTION_NOT_ALLOWED | 此事件不支援預測 | ❌ 未實作 |
| 403 | USER_CONSENT_NOT_GRANTED | 用戶未授權 | ❌ 未實作 |
| 403 | NO_ROAMING_SUPPORT | 不支援漫遊分析 | ❌ 未實作 |
| 404 | SUBSCRIPTION_NOT_FOUND | 訂閱不存在 | ✅ 已實作 |
| 500 | UNAVAILABLE_DATA | 資料不可用 | ❌ 未實作 |

### 5.2 ProblemDetails 結構

```json
{
  "status": 400,
  "cause": "INVALID_REQUEST",
  "detail": "excepRequs and exptAnaType cannot be provided together",
  "instance": "/subscriptions/xxx"
}
```

---

## 6. Feature 協商

### 6.1 機制

在 POST/PUT 請求中，消費者可以提供 `supportedFeatures` 欄位來協商功能。

### 6.2 與 ABNORMAL_BEHAVIOUR 相關的 Feature

| Feature # | 名稱 | 說明 | 重要性 |
|-----------|------|------|--------|
| 5 | AbnormalBehaviour | 基本異常行為支援 | ⭐⭐⭐ 核心 |
| 28 | EnAbnormalBehaviour | 增強異常行為 | ⭐⭐ 建議 |
| 11 | EneNA | 增強網路分析需求 | ⭐⭐ 建議 |
| 27 | ENAExt | 一般增強（含 useCaseCxt）| ⭐ 可選 |
| 47 | AnalyticsAccuracy | 分析準確度資訊 | ⭐ 可選 |

### 6.3 Feature 位元碼計算

`supportedFeatures` 是一個十六進制字串，每個 bit 代表一個 feature：

```
Feature 5  (AbnormalBehaviour) = bit 4  = 0x10
Feature 11 (EneNA)             = bit 10 = 0x400
Feature 28 (EnAbnormalBehaviour) = bit 27 = 0x8000000

組合：0x8000410
```

---

## 7. 實作優先級建議

### 7.1 Phase 1：基礎完善（已完成 80%）

| 項目 | 狀態 | 行動 |
|------|------|------|
| POST/PUT/DELETE API | ✅ | - |
| 基本驗證 | ✅ | - |
| `excepRequs`/`exptAnaType` 互斥驗證 | ❌ | **需補足** |
| `anyUe=true` 條件驗證 | ❌ | **需補足** |
| 不支援事件類型的拒絕 | ❌ | **需補足** |

### 7.2 Phase 2：Notification 機制（關鍵功能）

| 項目 | 說明 |
|------|------|
| HTTP Client 發送模組 | 向 notificationURI 發送 POST |
| 通知排程器 | 處理 PERIODIC、ONE_TIME 等觸發 |
| 通知資料結構 | 組裝 NnwdafEventsSubscriptionNotification |
| 重試機制 | 通知失敗時的重試策略 |

### 7.3 Phase 3：分析引擎整合

| 項目 | 說明 |
|------|------|
| 資料收集 | 從 UPF/SMF 收集流量資料 |
| DDoS 偵測模型 | ML/規則引擎分析 |
| 異常事件產生 | 產生 AbnormalBehaviour 輸出 |

### 7.4 Phase 4：進階功能

| 項目 | 說明 |
|------|------|
| Feature 協商 | supportedFeatures 處理 |
| immRep 即時報告 | 訂閱時立即回傳可用分析 |
| failEventReports | 部分失敗報告 |
| Transfer API | 訂閱轉移 |
| User Consent | 用戶同意檢查 |

---

## 8. 快速檢查清單

### 目前已實作 ✅

- [x] POST /subscriptions - 建立訂閱
- [x] PUT /subscriptions/{id} - 更新訂閱
- [x] DELETE /subscriptions/{id} - 刪除訂閱
- [x] Location Header 回傳
- [x] eventSubscriptions 必填驗證
- [x] notificationURI 必填驗證
- [x] event 必填驗證
- [x] ABNORMAL_BEHAVIOUR tgtUe 驗證
- [x] excepRequs 或 exptAnaType 存在驗證

### 待實作（驗證完善）⚠️

- [ ] excepRequs 與 exptAnaType 互斥驗證
- [ ] anyUe=true 時的額外條件驗證
- [ ] 不支援事件類型的拒絕（如 UE_MOBILITY）
- [ ] exptAnaType → ExceptionIds 映射

### 待實作（核心功能）❌

- [ ] Notification 發送機制
- [ ] 週期性通知排程
- [ ] immRep 即時報告
- [ ] Feature 協商 (supportedFeatures)
- [ ] 分析引擎整合

### 待實作（進階功能）🔮

- [ ] Transfer API
- [ ] User Consent 檢查
- [ ] failEventReports
- [ ] 重定向支援 (307/308)
