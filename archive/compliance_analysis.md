# NWDAF EventSubscription 規範合規性分析報告

本報告比對目前實作與 3GPP 規範（TS 29.520 YAML、TS 23.520 筆記）的符合程度。

---

## 符號說明

| 符號 | 意義 |
|------|------|
| ✅ | 完全符合規範 |
| ⚠️ | 部分符合，有缺漏需補強 |
| ❌ | 未實作或不符合 |

---

## 1. API 端點合規性

### 1.1 POST `/subscriptions` (CreateNWDAFEventsSubscription)

| 項目 | 規範要求 | 實作狀態 | 說明 |
|------|---------|---------|------|
| HTTP Method | POST | ✅ | 正確 |
| Resource URI | `/nnwdaf-eventssubscription/v1/subscriptions` | ✅ | 正確 |
| Request Body | `NnwdafEventsSubscription` | ✅ | 使用 openapi models |
| Response 201 | 回傳訂閱資料 | ✅ | 正確回傳 |
| Location Header | 必須包含新創資源 URI | ✅ | 已實作 |
| Error 400 | ProblemDetails | ✅ | 有驗證錯誤處理 |
| Error 404 | ProblemDetails | ✅ | 支援 |
| **Callback myNotification** | 定義通知回呼機制 | ❌ | **尚未實作 Notify 機制** |

### 1.2 PUT `/subscriptions/{subscriptionId}` (UpdateNWDAFEventsSubscription)

| 項目 | 規範要求 | 實作狀態 | 說明 |
|------|---------|---------|------|
| HTTP Method | PUT | ✅ | 正確 |
| Response 200 | 回傳更新後的訂閱 | ✅ | 正確 |
| Response 204 | 可選的無內容回應 | ❌ | 目前只回傳 200 |
| Error 404 | 訂閱不存在 | ✅ | 有檢查並回傳 404 |

### 1.3 DELETE `/subscriptions/{subscriptionId}` (DeleteNWDAFEventsSubscription)

| 項目 | 規範要求 | 實作狀態 | 說明 |
|------|---------|---------|------|
| HTTP Method | DELETE | ✅ | 正確 |
| Response 204 | No Content | ✅ | 正確 |
| Error 404 | 訂閱不存在 | ✅ | 有檢查並回傳 404 |

### 1.4 Transfer API (`/transfers`)

| 項目 | 規範要求 | 實作狀態 |
|------|---------|---------|
| POST `/transfers` | 訂閱轉移請求 | ❌ 未實作 |
| PUT `/transfers/{transferId}` | 更新轉移 | ❌ 未實作 |
| DELETE `/transfers/{transferId}` | 取消轉移 | ❌ 未實作 |

> **備註**: Transfer API 用於 NWDAF 間的訂閱轉移，屬於進階功能，第一階段可暫不實作。

---

## 2. 資料結構合規性

### 2.1 NnwdafEventsSubscription

| 欄位 | 規範要求 | 實作狀態 | 說明 |
|------|---------|---------|------|
| `eventSubscriptions` | **必填** (required) | ✅ | 有驗證 |
| `notificationURI` | 接收通知的 URI | ✅ | 有驗證必填 |
| `evtReq` | 報告設定 (選填) | ✅ | 有儲存 |
| `notifCorrId` | 關聯 ID (選填) | ✅ | 有儲存 |
| `supportedFeatures` | Feature 協商 | ⚠️ | **未處理 Feature 協商** |
| `eventNotifications` | 即時報告 (選填) | ❌ | 未實作 immRep |
| `failEventReports` | 失敗事件報告 | ❌ | 未實作 |
| `prevSub` | 前一個訂閱資訊 | ❌ | 未實作 (AnaCtxTransfer) |
| `consNfInfo` | 消費者 NF 資訊 | ❌ | 未實作 (AnaSubTransfer) |

### 2.2 EventSubscription

| 欄位 | 規範要求 | 實作狀態 | 說明 |
|------|---------|---------|------|
| `event` | **必填** (NwdafEvent) | ✅ | 有驗證 |
| `tgtUe` | 目標 UE | ✅ | ABNORMAL_BEHAVIOUR 有驗證 |
| `excepRequs` | Exception 清單 | ✅ | ABNORMAL_BEHAVIOUR 有處理 |
| `exptAnaType` | 預期分析類型 | ✅ | 有驗證存在性 |
| `notificationMethod` | 通知方法 | ⚠️ | 有儲存但未處理邏輯 |
| `repetitionPeriod` | 週期性通知間隔 | ⚠️ | 有儲存但未處理邏輯 |
| `networkArea` | 網路區域 | ⚠️ | 有儲存但未驗證必填條件 |

---

## 3. ABNORMAL_BEHAVIOUR 事件驗證合規性

根據 **TS 23.520 Clause 4.2.2.2.2**:

| 規範要求 | 實作狀態 | 說明 |
|---------|---------|------|
| `tgtUe` 必須包含 `supis`, `intGroupIds`, 或 `anyUe=true` | ✅ | Line 172-182 有完整驗證 |
| 必須提供 `excepRequs` **或** `exptAnaType` | ✅ | Line 185-195 有驗證 |
| `excepRequs` 與 `exptAnaType` 不可同時提供 (YAML line 732-733: `not: required: [excepRequs, exptAnaType]`) | ❌ | **缺少互斥驗證** |
| 若 `anyUe=true` 且為 mobility 相關，需提供 `networkArea` 或 `snssais` | ❌ | **未實作此條件驗證** |
| 若 `anyUe=true` 且為 commun 相關，需提供 `networkArea`, `appIds`, `dnns`, 或 `snssais` 之一 | ❌ | **未實作此條件驗證** |

### exptAnaType 對應 ExceptionId 映射

規範要求 NWDAF 根據 `exptAnaType` 推導對應的 Exception IDs:

| exptAnaType | 對應 ExceptionId |
|-------------|------------------|
| `MOBILITY` | UNEXPECTED_UE_LOCATION, PING_PONG_ACROSS_CELLS, UNEXPECTED_WAKEUP, UNEXPECTED_RADIO_LINK_FAILURES |
| `COMMUN` | UNEXPECTED_LONG_LIVE_FLOW, UNEXPECTED_LARGE_RATE_FLOW, **SUSPICION_OF_DDOS_ATTACK**, WRONG_DESTINATION_ADDRESS, TOO_FREQUENT_SERVICE_ACCESS |
| `MOBILITY_AND_COMMUN` | 上述全部 |

**實作狀態**: ❌ **未實作此映射邏輯**

---

## 4. 重要缺失功能

### 4.1 高優先級 (應儘快補足)

1. **Notification 機制 (Nnwdaf_EventsSubscription_Notify)**
   - 規範要求：成功建立訂閱後，需依週期或事件觸發發送通知到 `notificationURI`
   - 目前狀態：完全未實作
   - 影響：無法真正提供分析結果給消費者

2. **excepRequs/exptAnaType 互斥驗證**
   - YAML 明確定義：`not: required: [excepRequs, exptAnaType]`
   - 目前狀態：可同時提供兩者而不會報錯

3. **immRep (Immediate Report) 支援**
   - 若 `evtReq.immRep = true`，應在訂閱回應中包含目前可用的分析報告
   - 目前狀態：未實作

### 4.2 中優先級 (功能完整性)

1. **Feature 協商 (supportedFeatures)**
   - 不同 Feature 控制不同欄位的必填/可選性
   - 目前狀態：未處理

2. **anyUe 條件驗證**
   - ABNORMAL_BEHAVIOUR 中若 `anyUe=true`，需額外驗證 `networkArea`/`snssais` 等欄位

3. **failEventReports 處理**
   - 若部分事件無法處理，應在回應中標記失敗事件

### 4.3 低優先級 (進階功能)

1. **Transfer API** (`/transfers`) - 訂閱轉移
2. **User Consent 檢查** - 透過 UDM 驗證用戶同意
3. **Roaming 支援** - RoamingAnalytics feature

---

## 5. 總結評估

| 分類 | 符合程度 |
|------|---------|
| **API 端點結構** | ✅ 85% (缺 Transfer API) |
| **核心資料結構** | ⚠️ 70% (需補 Feature 協商) |
| **ABNORMAL_BEHAVIOUR 驗證** | ⚠️ 60% (需補互斥驗證與 anyUe 條件) |
| **Notification 機制** | ❌ 0% (完全未實作) |

### 整體符合度: **約 55%** (基礎框架完成，核心功能待補)

---

## 6. 建議修正優先順序

1. ⬆️ 補足 `excepRequs`/`exptAnaType` 互斥驗證
2. ⬆️ 補足 `anyUe=true` 時的條件驗證
3. ⬆️ 實作 `exptAnaType` → Exception IDs 映射
4. ⬆️ 實作 Notification 發送機制 (Phase 4)
5. 補足 `immRep` 即時報告功能
6. 處理 `supportedFeatures` 協商
