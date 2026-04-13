# NWDAF UE Communication 訂閱指南

**最後更新**: 2026-01-14

本文件說明如何向 NWDAF 訂閱 UE Communication 分析資料。

---

## 1. API 概覽

| 項目 | 說明 |
|------|------|
| **端點** | `POST /nnwdaf-eventssubscription/v1/subscriptions` |
| **事件類型** | `UE_COMMUNICATION` |
| **規格** | 3GPP TS 29.520, TS 23.288 §6.7 |

---

## 2. 必要欄位

| 欄位 | 必填 | 說明 |
|------|------|------|
| `eventSubscriptions[].event` | ✅ | 設為 `"UE_COMMUNICATION"` |
| `eventSubscriptions[].tgtUe` | ✅ | 目標 UE 資訊 |
| `eventSubscriptions[].tgtUe.supis` | ✅* | SUPI 陣列 |
| `eventSubscriptions[].tgtUe.intGroupIds` | ✅* | 群組 ID 陣列 |
| `notificationURI` | ✅ | 通知 callback URL |

> **Note**: `supis` 和 `intGroupIds` 至少需提供其一。

---

## 3. 選填欄位

| 欄位 | 說明 |
|------|------|
| `notifCorrId` | 通知關聯 ID (原樣回傳) |
| `evtReq.notifMethod` | `PERIODIC` 或 `THRESHOLD` |
| `evtReq.repPeriod` | 通知週期 (秒) |
| `evtReq.maxReportNbr` | 最大通知次數 |
| `evtReq.monDur` | 監控期間 (ISO 8601 DateTime) |
| `eventSubscriptions[].tgtPeriod` | 分析時間範圍 |

---

## 4. 訂閱範例

### 4.1 基本訂閱 (單一 UE)

```json
{
  "eventSubscriptions": [
    {
      "event": "UE_COMMUNICATION",
      "tgtUe": {
        "supis": ["imsi-208930000000001"]
      }
    }
  ],
  "notificationURI": "http://consumer:8080/nwdaf-callback"
}
```

### 4.2 週期性通知

```json
{
  "eventSubscriptions": [
    {
      "event": "UE_COMMUNICATION",
      "tgtUe": {
        "supis": ["imsi-208930000000001"]
      }
    }
  ],
  "notificationURI": "http://consumer:8080/nwdaf-callback",
  "notifCorrId": "my-correlation-001",
  "evtReq": {
    "notifMethod": "PERIODIC",
    "repPeriod": 60,
    "maxReportNbr": 10
  }
}
```

### 4.3 多 UE 訂閱

```json
{
  "eventSubscriptions": [
    {
      "event": "UE_COMMUNICATION",
      "tgtUe": {
        "supis": [
          "imsi-208930000000001",
          "imsi-208930000000002",
          "imsi-208930000000003"
        ]
      }
    }
  ],
  "notificationURI": "http://consumer:8080/nwdaf-callback"
}
```

### 4.4 使用群組 ID

```json
{
  "eventSubscriptions": [
    {
      "event": "UE_COMMUNICATION",
      "tgtUe": {
        "intGroupIds": ["group-vip-users"]
      }
    }
  ],
  "notificationURI": "http://consumer:8080/nwdaf-callback"
}
```

---

## 5. 回應說明

### 5.1 成功 (201 Created)

```json
{
  "eventSubscriptions": [...],
  "notificationURI": "http://consumer:8080/nwdaf-callback"
}
```

Location header 包含訂閱 ID: `/nnwdaf-eventssubscription/v1/subscriptions/{subscriptionId}`

### 5.2 錯誤回應

| HTTP Code | Cause | 說明 |
|-----------|-------|------|
| 400 | `INVALID_REQUEST` | 缺少必要欄位 |
| 400 | `INVALID_REQUEST` | `tgtUe` 必須包含 `supis` 或 `intGroupIds` |

---

## 6. 通知格式

Consumer 會收到 POST 到 `notificationURI`:

```json
{
  "notifCorrId": "my-correlation-001",
  "eventNotifications": [
    {
      "event": "UE_COMMUNICATION",
      "ueComms": [
        {
          "supi": "imsi-208930000000001",
          "commDur": 3600,
          "trafVolume": {
            "ulVol": 1048576,
            "dlVol": 5242880
          }
        }
      ],
      "timeStamp": "2026-01-14T09:00:00Z"
    }
  ]
}
```

---

## 7. 刪除訂閱

```bash
DELETE /nnwdaf-eventssubscription/v1/subscriptions/{subscriptionId}
```

回應: `204 No Content`

---

## 8. 完整 curl 範例

```bash
# 建立訂閱
curl -X POST http://localhost:29520/nnwdaf-eventssubscription/v1/subscriptions \
  -H "Content-Type: application/json" \
  -d '{
    "eventSubscriptions": [{
      "event": "UE_COMMUNICATION",
      "tgtUe": {
        "supis": ["imsi-208930000000001"]
      }
    }],
    "notificationURI": "http://my-server:8080/callback"
  }'

# 刪除訂閱
curl -X DELETE http://localhost:29520/nnwdaf-eventssubscription/v1/subscriptions/{subscriptionId}
```
