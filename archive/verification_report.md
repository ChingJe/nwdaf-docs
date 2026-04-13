# 規範合規性總驗證報告 v2

**更新**：加入 Stage 2 (TS 23.288) ↔ Stage 3 (TS 29.520) 對應分析

---

## 1. 規範對應關係

> **NOTE**: TS 23.520 明確指出：「This service corresponds to the `Nnwdaf_AnalyticsSubscription` service defined in 3GPP TS 23.288.」

### 1.1 服務操作對照表

| Stage 2 (TS 23.288) | Stage 3 (TS 29.520) | HTTP | 實作狀態 |
|---------------------|---------------------|------|---------|
| `_Subscribe` | `POST /subscriptions` | 建立訂閱 | ✅ |
| `_Subscribe` (w/ Correlation ID) | `PUT /subscriptions/{id}` | 修改訂閱 | ✅ |
| `_Unsubscribe` | `DELETE /subscriptions/{id}` | 取消訂閱 | ✅ |
| `_Notify` | Callback to `notificationURI` | 通知 | ❌ |
| `_Transfer` | `POST /transfers` | 轉移 | ❌ |

---

## 2. 輸入參數合規性 (7.2.2 對照)

### 2.1 必要參數 (Required)

| Stage 2 參數 | Stage 3 參數 | 驗證 |
|-------------|-------------|------|
| Analytics ID(s) | `event` | ✅ 驗證必填 |
| Target of Analytics Reporting | `tgtUe` | ✅ supis/intGroupIds/anyUe |
| Notification Target Address | `notificationURI` | ✅ 驗證必填 |
| Notification Correlation ID | `notifCorrId` | ⚠️ 未強制 |

### 2.2 可選參數 (Optional)

| Stage 2 參數 | Stage 3 參數 | 狀態 |
|-------------|-------------|------|
| Analytics Filter Information | `networkArea`, `snssais`, `dnns`, `appIds` | ✅ anyUe 驗證 |
| Subscription Correlation ID | subscriptionId (PUT) | ✅ 已實作 |
| Preferred level of accuracy | `evtReq.accuracy` | ⚠️ 未使用 |
| Reporting Thresholds | 各事件特定參數 | ⚠️ 未處理 |
| Analytics Feedback | `feedback` | ❌ 未實作 |

---

## 3. ABNORMAL_BEHAVIOUR 特定驗證

### 3.1 excepRequs / exptAnaType 互斥

| 來源 | 說明 | 實作 |
|------|------|------|
| TS 23.288 6.1.4.5 | Either...or... | ✅ |
| TS 29.520 YAML L732-733 | `not: required: [excepRequs, exptAnaType]` | ✅ |

### 3.2 anyUe 條件驗證 (筆記 6.3.5)

| 條件 | 規範要求 | 實作 |
|------|---------|------|
| Mobility 相關 | 需 networkArea 或 snssais | ✅ |
| Commun 相關 | 需 networkArea/appIds/dnns/snssais | ✅ |
| 不可混合 | mobility+commun 不可同時 | ✅ |

---

## 4. 筆記重點對照

### 4.1 已符合

| 筆記章節 | 內容 | 實作對應 |
|---------|------|---------|
| 7.2.1 | Subscription Correlation ID 分配 | `NewSubscriptionId()` |
| 7.2.2 | Target 驗證 (SUPI/Group/AnyUE) | `validateAbnormalBehaviour()` |
| 7.2.2 | Notification Callback URI | `notificationURI` 必填 |

### 4.2 待實作 (按優先級)

| 優先級 | 筆記章節 | 功能 | 說明 |
|--------|---------|------|------|
| ⭐⭐⭐ | 7.2.4 | Notify 機制 | 需實作 callback 發送 |
| ⭐⭐ | 7.2.2 Output | Immediate Report | Response 夾帶首次結果 |
| ⭐⭐ | 7.2.4 | Termination Request | NWDAF 過載時終止 |
| ⭐ | 7.2.2 NOTE 5 | User Consent | 用戶授權檢查 |
| ⭐ | 7.2.5 | Transfer API | 訂閱轉移 |

---

## 5. 結論

| 項目 | 狀態 |
|------|------|
| Stage 2 ↔ Stage 3 對應 | ✅ 確認一致 |
| CRUD 操作 | ✅ 已完成 |
| ABNORMAL_BEHAVIOUR 驗證 | ✅ 符合規範 |
| Notification 機制 | ❌ 下一階段 |

**當前實作覆蓋率**：Stage 2 7.2.2/7.2.3 約 80% 核心功能
