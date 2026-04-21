# Inference Aggregation Pipeline

本文件說明 NWDAF AnLF 在執行 ML 推論前，如何從 in-memory ring buffer 蒐集、對齊、聚合多個 UE 的 UPF 流量資料，最終產生一份時間序列輸入送給 ML service。

涵蓋範圍：UPF 通知進入 → ring buffer 寫入 → `fetchHistoricalData` → `alignAndZipInMemory` → `[]TrafficObservation`

---

## 整體架構

```
UPF Notification
      │
      ▼
processUpfNotificationItemUnified()       [processor/upf_notify.go]
      │
      ├─── MongoDB INSERT (for accuracy monitor ground truth)
      │
      └─── TrafficData.RawUpfData append  (in-memory ring buffer)
                    │
                    │  (每個 notification cycle 觸發)
                    ▼
         generateMlBasedUeCommunication() [anlf/analytics.go]
                    │
                    ▼
            fetchHistoricalData()
                    │
                    ▼
          alignAndZipInMemory()
                    │
                    ▼
          []TrafficObservation  ──► ML service Predict()
```

---

## 資料結構

### In-Memory Ring Buffer

```
NWDAFContext.trafficDataStore
  └── map[correlationId] *TrafficDataBucket
            └── map[ipAddress] *TrafficData
                      └── RawUpfData []UpfDataPoint   ← ring buffer
```

- **`TrafficDataBucket`**：一個 `correlationId`（對應一個 SMF 訂閱 / 一個 SUPI）下所有 IP 的容器
- **`TrafficData`**：單一 IP session 的資料，含 DNN、SUPI、SNSSAI 等 metadata
- **`RawUpfData`**：最多 `ringBufferSize`（config，預設 50）筆，FIFO 淘汰最舊的

### Ring Buffer 大小限制

```go
// processor/upf_notify.go
data.RawUpfData = append(data.RawUpfData, dataPoint)
ringBufferSize := factory.NwdafConfig.GetRingBufferSize()
if len(data.RawUpfData) > ringBufferSize {
    drop := len(data.RawUpfData) - ringBufferSize
    data.RawUpfData = data.RawUpfData[drop:]
}
```

> **限制**：`ringBufferSize` 必須 ≥ `inputWindow`，否則 buffer 永遠裝不夠一次推論所需的點數。啟動時 `ReadConfig()` 會發出 WARN。

---

## 推論觸發點

```go
// anlf/analytics.go
func generateMlBasedUeCommunication(...) {
    params           := getUeCommunicationModelParams()
    samplingInterval := params.SamplingIntervalOrDefault()
    si64             := int64(samplingInterval)
    snappedNow       := time.Unix((now.Unix()/si64)*si64, 0)  // floor to grid

    historicalData, dnn := fetchHistoricalData(nwdafSubId, ctx, params, snappedNow)
    resp, _             := mlClient.Predict(modelId, historicalData)
    ...
}
```

`snappedNow` = 當前時間 floor 到 `samplingInterval` 格點，用作全域時間參考基準（Step 2 對齊的錨點）。

---

## alignAndZipInMemory：兩步驟對齊

### 問題背景

UPF 的 `startTime` 由 UPF 自己打上，存在兩種對齊問題：

| 問題 | 範例 | naive floor 的結果 |
|------|------|--------------------|
| **Anchor drift**：每個 IP 的第一筆時間戳不在絕對格點上 | 預期 t=10，實際到 t=9.8 | `floor(9.8/5)*5 = 5` → 分錯格 |
| **Late joiner**：同一個 group 的 IP 在不同時間點才有資料 | IP-A 從 t=0 開始，IP-B 從 t=27 開始（si=5） | 直接用序列 index 對齊會把不同時間段加在一起 |

---

### Step 1：Per-IP Anchor Round（消除 drift，天然 dedup）

```
anchor := RawUpfData[0].Timestamp.Unix()   // 該 IP 的第一筆時間戳

for each dp in RawUpfData:
    n          = round( (dp.Timestamp.Unix() - anchor) / si )
    centerUnix = anchor + n * si

    perIP[centerUnix] = dp   // 同一個 centerUnix 的資料 last-wins（dedup）
```

**效果**：

```
anchor = 0.3, si = 5

dp.Timestamp = 9.8  →  n = round((9.8 - 0.3) / 5) = round(1.9) = 2
                        centerUnix = 0.3 + 2*5 = 10.3  ✓ 正確落入第三格

dp.Timestamp = 1.0  →  n = round((1.0 - 0.3) / 5) = round(0.14) = 0
                        centerUnix = 0.3  （與第一筆相同，last-wins dedup）
```

容許誤差 = ±si/2（round 的邊界在半格處）。

> **設計細節**：`perIP` map 的 key 是精確整數（`anchor + n*si`），同一個 IP session 同一個時槽只會保留一筆，Lock 在此步驟結束後即釋放，不阻塞後續 Step 2 的 map 操作。

---

### Step 2：Global Round（對齊不同 anchor 的 IP）

```
for each (centerUnix, dp) in perIP:
    globalIndex = round( (centerUnix - snappedNow.Unix()) / si )

    buckets[globalIndex].agg  += dp values
    buckets[globalIndex].centerSum   += centerUnix
    buckets[globalIndex].centerCount++
```

**效果（si=5, snappedNow=30）**：

```
IP-A anchor=0.3:  第 5 筆  centerUnix=25.3
  globalIndex = round((25.3 - 30) / 5) = round(-0.94) = -1

IP-B anchor=27.1: 第 1 筆  centerUnix=27.1
  globalIndex = round((27.1 - 30) / 5) = round(-0.58) = -1  ← 同格 ✓

IP-A anchor=0.3:  第 6 筆  centerUnix=30.3
  globalIndex = round((30.3 - 30) / 5) = round(0.06) = 0
```

結果：
- `globalIndex = -1`：IP-A（center=25.3）+ IP-B（center=27.1）正確加總
- `globalIndex =  0`：只有 IP-A（center=30.3）

---

### 輸出 Ts：Mean of Centers

每個 globalIndex 的輸出時間戳不是絕對格點，而是所有貢獻 IP 的 centerUnix 平均值：

```
meanCenter = round( centerSum / centerCount )

globalIndex -1:  mean(25.3, 27.1) = 26.2 → round → 26
                 Ts = "1970-01-01T00:00:26Z"
```

這保留了真實測量時間，而非強制對齊到人為的格點。

---

### 最終截取與 Zero-Padding

Step 2 結束後，`buckets` map 只包含**有實際資料的 global slot**。但 ML 模型需要等間距的時間序列，中間若有空格必須補 0。

#### 範圍確定

```
endIdx   = keys 中最大值（最新有資料的 slot）
startIdx = max(keys[0], endIdx - inputWindow + 1)
```

- `endIdx - inputWindow + 1`：往回推 `inputWindow` 格，取「理想起點」
- `keys[0]`：第一筆實際資料所在的 slot
- 取兩者較大的，**不在最舊資料左邊補 0**（避免在真實資料之前無限填充）

#### 逐格填充

```
for i in [0, outputLen):
    idx = startIdx + i
    ts  = snappedNow + idx × si      ← 先用格點時間作為預設 Ts

    if buckets[idx] 存在:
        ts      = mean(centerUnix of contributing IPs)
        result[i] = 真實聚合值

    else:                             ← 中間空格
        result[i] = TrafficObservation{Ts: ts}   ← 流量欄位全為 0
```

空格的 Ts 用 `snappedNow + idx×si` 推算，保持序列的等間距性質。

#### 範例

```
資料: t=0 (UlVol=10), t=10 (UlVol=11)
si=5, snappedNow=20, inputWindow=4

global index:
  t=0  → round((0-20)/5)  = -4
  t=10 → round((10-20)/5) = -2

keys = [-4, -2]
endIdx   = -2
startIdx = max(-4, -2-4+1) = max(-4, -5) = -4

輸出 3 格（-4 到 -2）：
  idx=-4 → 有資料 → UlVol=10, Ts=t=0
  idx=-3 → 空格   → UlVol=0,  Ts=snappedNow+(-3×5)=5
  idx=-2 → 有資料 → UlVol=11, Ts=t=10
```

#### 注意事項

- Zero-padding 只填充**資料範圍內的空格**，不在頭尾之外補 0
- `inputWindow` 的作用是限制 `startIdx`：若實際有超過 `inputWindow` 個 slot，從 `endIdx` 往回截取
- Debug log 會輸出每個 slot 的 IP 貢獻數，空格顯示為 `0`，例如 `ipCount/slot:[2,2,0,1,2]`

---

## 完整流程圖

```
每個 corrId
  └── 每個 IP session (TrafficData)
        │
        │ [Lock]
        ├── anchor = RawUpfData[0].Timestamp
        │
        ├── for each dp:
        │     n         = round((dp.Ts - anchor) / si)
        │     center    = anchor + n*si
        │     perIP[center] = dp            ← Step 1: drift 修正 + dedup
        │                                      （覆蓋時發出 Warn）
        │ [Unlock]
        │
        └── for each (center, dp) in perIP:
              idx = round((center - snappedNow) / si)
              buckets[idx].agg += dp        ← Step 2: global 對齊 + 加總
              buckets[idx].centerSum += center
              buckets[idx].centerCount++

排序 bucket keys
→ startIdx = max(keys[0], endIdx - inputWindow + 1)
→ 逐格迭代 [startIdx, endIdx]：有資料填真實值，無資料填零
→ 每個 slot: Ts = mean(centers) 或 snappedNow+idx×si（空格）
→ []TrafficObservation  →  ML service
```

---

## 設定參數

| 參數 | 設定位置 | 預設值 | 說明 |
|------|----------|--------|------|
| `samplingInterval` | `analytics.ueCommunication.samplingInterval` | 10s | UPF report 週期；決定格子寬度 |
| `inputWindow` | `analytics.ueCommunication.inputWindow` | 30 | 送給 ML 的歷史點數 |
| `ringBufferSize` | `analytics.ueCommunication.ringBufferSize` | 50 | 每個 IP session 的 ring buffer 大小；必須 ≥ inputWindow |

---

## 相關檔案

| 檔案 | 職責 |
|------|------|
| `internal/context/traffic_data.go` | `UpfDataPoint`、`TrafficData`、`TrafficDataBucket` 定義 |
| `internal/sbi/processor/upf_notify.go` | UPF 通知處理、ring buffer 寫入、MongoDB 寫入 |
| `internal/anlf/analytics.go` | `alignAndZipInMemory`、`fetchHistoricalData`、`globalBucket` |
| `internal/anlf/analytics_test.go` | 對齊邏輯的單元測試（drift、dedup、late joiner、cap 等） |
| `pkg/factory/config.go` | `ModelParams`、`RingBufferSizeOrDefault`、啟動驗證警告 |
