# 資料撈取邏輯：MongoDB Time Series Query + In-Memory Fallback

> 本文件詳細說明 NWDAF 如何從 MongoDB Time Series Collection 與 in-memory 中撈取 UPF 流量資料，用於 ML 預測和 Accuracy Monitoring。

---

## 1. 資料寫入端（upf_notify.go）

每次收到 UPF 通知，資料**同時**寫入兩處：

### In-Memory（Ring Buffer）
```go
// traffic_data.go
const maxInMemoryDataPoints = 50  // 50 × 10s ≈ 8.3 min

func (d *TrafficData) AppendDataPoint(point UpfDataPoint) {
    d.mu.Lock()
    defer d.mu.Unlock()
    d.RawUpfData = append(d.RawUpfData, point)
    // Ring-buffer: 超過上限就丟掉最舊的
    if len(d.RawUpfData) > maxInMemoryDataPoints {
        drop := len(d.RawUpfData) - maxInMemoryDataPoints
        d.RawUpfData = d.RawUpfData[drop:]
    }
    d.LastUpdate = point.Timestamp
}
```

### MongoDB
```go
// upf_notify.go L174-181
record := nwdaf_context.UpfTrafficRecord{
    Metadata: nwdaf_context.UpfTrafficMetaData{
        IpAddr:        ipAddr,
        CorrelationId: bucket.CorrelationId,
        Supi:          item.Supi,
        GroupId:       groupId,
        Dnn:           item.Dnn,
    },
    Timestamp: item.TimeStamp,
    UlVolume:  usage.VolumeMeasurement.UlVolume,
    DlVolume:  usage.VolumeMeasurement.DlVolume,
}
coll := mongoapi.Client.Database(dbName).Collection("nwdaf.upfTrafficData")
coll.InsertOne(context.Background(), record)
```

MongoDB document 結構（Time Series Collection）：
```json
{
    "metadata": {
        "ipAddr": "192.168.1.1",
        "correlationId": "corr-abc-123",
        "supi": "imsi-208930000000001",
        "groupId": "",
        "dnn": "internet"
    },
    "timestamp": ISODate("2026-03-02T16:30:10Z"),
    "ulVolume": 12345,
    "dlVolume": 67890
}
```

---

## 2. ML 預測資料撈取（analytics.go）

### 入口：`fetchHistoricalData(nwdafSubId, ctx)`

```go
func fetchHistoricalData(
    nwdafSubId string,
    ctx *nwdaf_context.NWDAFContext,
) ([]TrafficObservation, string) {
```

### 2.1 Primary：MongoDB 查詢

**第一步 — 取得所有 correlationId**

一個 nwdafSubId 可能對應多個 SMF 訂閱（多個 SUPI 或多個 SMF endpoint），每個 SMF 訂閱有獨立的 `correlationId`。

```go
corrIds := ctx.GetCorrelationIdsByNwdafSubId(nwdafSubId)
// 例如: ["corr-001", "corr-002"]
```

追蹤來源：
```
NwdafSubResource { SmfEndpoint, Supi, CorrelationId }
      ↑
nwdafSubResourcesMap[nwdafSubId] = []NwdafSubResource{...}
```

**第二步 — 查詢 MongoDB**

```go
since := time.Now().Add(-5 * time.Minute)  // DefaultQueryWindow
records, err := nwdaf_context.QueryTrafficByMultipleCorrelationIds(
    dbName, corrIds, since, 30,  // DefaultMaxDataPoints
)
```

實際 MongoDB filter：
```json
{
    "metadata.correlationId": { "$in": ["corr-001", "corr-002"] },
    "timestamp": { "$gte": ISODate("2026-03-02T16:25:10Z") }
}
```

find options：
```json
{
    "sort": { "timestamp": 1 },
    "limit": 30
}
```

**為什麼這樣設計？**
- `$in` correlationIds：一個訂閱可能追蹤多個 UE（Group ID 解析後多個 SUPI）
- `timestamp >= now - 5min`：只取最近 5 分鐘，確保資料時效性
- `sort timestamp ASC`：時間序列排列，ML 模型要求時間正序輸入
- `limit 30`：TCN 模型最多接受 30 個 data points（30 × 10s = 5 min）

**第三步 — 轉換為 ML 輸入格式**

```go
func upfRecordsToObservations(records []UpfTrafficRecord) ([]TrafficObservation, string) {
    obs := make([]TrafficObservation, 0, len(records))
    dnn := "internet"
    for _, r := range records {
        obs = append(obs, TrafficObservation{
            Ts: r.Timestamp.Format(time.RFC3339),
            TrafChar: TrafficCharacterization{
                UlVol: r.UlVolume,
                DlVol: r.DlVolume,
            },
        })
        if r.Metadata.Dnn != "" {
            dnn = r.Metadata.Dnn
        }
    }
    return obs, dnn
}
```

最終送進 ML Service 的 JSON：
```json
{
    "model_id": "tcn-model-001",
    "historical_data": [
        { "ts": "2026-03-02T16:25:10+08:00", "traf_char": { "ul_vol": 100, "dl_vol": 200 } },
        { "ts": "2026-03-02T16:25:20+08:00", "traf_char": { "ul_vol": 110, "dl_vol": 210 } },
        ...
    ],
    "prediction_steps": 5
}
```

### 2.2 Fallback：In-Memory

當 MongoDB 不可用（client == nil 或 dbName 為空）或查詢失敗/無結果時：

```go
func inMemoryToObservations(ctx, nwdafSubId) ([]TrafficObservation, string) {
    trafficDataList := ctx.GetTrafficDataByNwdafSubId(nwdafSubId)
    // 走 nwdafSubId → correlationIds → TrafficDataBucket → TrafficData → RawUpfData
    // 遍歷所有 TrafficData，收集 RawUpfData（ring buffer 內最多 50 筆）
    // 最後只取最後 30 筆（DefaultMaxDataPoints）
}
```

in-memory 路徑：
```
nwdafSubResourcesMap[nwdafSubId]
    → [NwdafSubResource{ CorrelationId: "corr-001" }, ...]
        → trafficDataStore["corr-001"] = TrafficDataBucket
            → dataMap["192.168.1.1"] = TrafficData
                → RawUpfData[50]  // ring buffer
```

---

## 3. Accuracy Monitor Ground Truth（accuracy_monitor.go）

### 入口：`lookupGroundTruth(ctx, pred)`

Prediction 記錄了 `TargetTime`（預測的目標時間點），需要找那個時間附近**實際發生**的流量資料。

### 3.1 Primary：MongoDB 精準時間區間

```go
from := pred.TargetTime                    // e.g. 16:30:00
to   := pred.TargetTime.Add(10 * time.Second) // e.g. 16:30:10

corrIds := ctx.GetCorrelationIdsByNwdafSubId(pred.NwdafSubId)
records, err := nwdaf_context.QueryTrafficInTimeRange(dbName, corrIds, from, to)
```

MongoDB filter：
```json
{
    "metadata.correlationId": { "$in": ["corr-001"] },
    "timestamp": {
        "$gte": ISODate("2026-03-02T16:30:00Z"),
        "$lt":  ISODate("2026-03-02T16:30:10Z")
    }
}
```

**聚合多筆結果**（同一時間窗口可能有多個 IP 的資料）：
```go
var ulVol, dlVol int64
for _, r := range records {
    ulVol += r.UlVolume
    dlVol += r.DlVolume
}
return &groundTruth{ulVol: ulVol, dlVol: dlVol}
```

### 3.2 Fallback：In-Memory 掃描

```go
for _, td := range dataList {
    td.Lock()
    for _, dp := range td.RawUpfData {
        diff := dp.Timestamp.Sub(pred.TargetTime)
        if diff >= 0 && diff < 10*time.Second {
            // 找到符合的資料點
            return &groundTruth{ulVol: dp.UlVolume, dlVol: dp.DlVolume}
        }
    }
    td.Unlock()
}
```

注意 in-memory fallback 只回傳**第一筆**符合的（不聚合），而 MongoDB 版本會**聚合所有筆**。

---

## 4. db_query.go 三個查詢函數

完整 code 位於 `internal/context/db_query.go`。

### QueryTrafficByCorrelationId
- 用途：查詢單一 correlationId 的最近資料
- filter: `{ metadata.correlationId: X, timestamp: { $gte: since } }`
- options: `sort timestamp ASC, limit N`

### QueryTrafficByMultipleCorrelationIds
- 用途：ML 預測（一個 nwdafSubId 可能有多個 correlationId）
- filter: `{ metadata.correlationId: { $in: [...] }, timestamp: { $gte: since } }`
- options: `sort timestamp ASC, limit N`

### QueryTrafficInTimeRange
- 用途：Accuracy Monitor 的 ground truth lookup
- filter: `{ metadata.correlationId: { $in: [...] }, timestamp: { $gte: from, $lt: to } }`
- options: `sort timestamp ASC`（無 limit，通常只有 1-2 筆）

三個函數共用的安全機制：
```go
if !IsMongoAvailable() {  // mongoapi.Client != nil
    return nil, nil
}
ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
defer cancel()
```

---

## 5. 資料流關鍵路徑對照表

| 用途 | 函數 | MongoDB filter | Fallback |
|------|------|---------------|----------|
| ML 預測輸入 | `fetchHistoricalData()` | `corrId $in, ts >= now-5min, limit 30` | in-memory 最後 30 筆 |
| Ground Truth | `lookupGroundTruth()` | `corrId $in, ts ∈ [target, target+10s)` | in-memory 單筆掃描 |

## 6. 相關常數

```go
// db_query.go
const dbQueryTimeout       = 10 * time.Second
const DefaultQueryWindow   = 5 * time.Minute    // ML 預測取最近 5 分鐘
const DefaultMaxDataPoints = 30                 // 最多 30 筆資料點

// traffic_data.go
const maxInMemoryDataPoints = 50                // ring buffer 上限
```
