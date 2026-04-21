# NWDAF EventSubscription 實作進度報告

**最後更新**: 2026-03-30
**目前階段**: Phase 11 ✅ + 後續修正與強化完成 + AnLF/MTLF 拆分完成 + Daisy 非同步化 + ADRF E2/E3 整合完成

---

## 1. 專案概述

實作 NWDAF 的 `Nnwdaf_EventsSubscription` 服務，支援：
- **UE_COMMUNICATION** (Rule-Based + ML-Based Analytics)
- **TCN Model Integration** (Time Convolutional Network) for Traffic Prediction
- **Federated Learning** (Daisy FL Framework) for MTLF Training
- **ADRF Integration** (TS 29.575 Storage + Retrieval) for retrain dataset pipeline

---

## 2. 階段完成狀態

| 階段 | 描述 | 狀態 |
|------|------|------|
| Phase 1 | 基礎訂閱 API | ✅ |
| Phase 2A | ABNORMAL_BEHAVIOUR 驗證 | ✅ |
| Phase 2B | evtReq/ExceptionId 驗證 | ✅ |
| Phase 2C | failEventReports + Analytics Target Period | ✅ |
| Phase 3 | Notification 機制 | ✅ |
| Phase 4 | UPF_EVENT 資料蒐集 | ✅ |
| Phase 5 | Consumer Module 重構 | ✅ |
| Phase 6 | Rule-Based Analytics + repPeriod | ✅ |
| Phase 7 | Storage Refactoring | ✅ |
| Phase 8 | ML Model Integration | ✅ |
| Phase 9 | Daisy FL Training Integration | ✅ |
| Phase 10 | MongoDB Time Series Integration | ✅ |
| Phase 11 | MongoDB Analytics + Config-Driven Model Params + ML API + Hot-Swap | ✅ |
| Phase 12 | Daisy Async Callback + ADRF E2/E3 (Storage/Retrieval) | ✅ |

---

## 3. Phase 11（MongoDB + ML/MTLF 基線）

### 3.1 MongoDB Analytics 資料來源

將 ML Analytics 與 Accuracy Monitoring 從 **in-memory** 遷移至 **MongoDB Time Series**。

```
fetchHistoricalData()         lookupGroundTruth()
  ├─ Primary:  MongoDB          ├─ Primary:  MongoDB [from, from+si)
  └─ Fallback: in-memory        └─ Fallback: in-memory scan
```

#### MongoDB Query Layer (`internal/context/db_query.go`)

| 函數 | 用途 |
|------|------|
| `QueryTrafficByCorrelationId` | 單一 corrId 歷史資料（DESC+limit+reverse） |
| `QueryTrafficByMultipleCorrelationIds` | ML 預測輸入（multi corrId `$in`） |
| `QueryTrafficInTimeRange` | Accuracy Monitor ground truth `[$gte from, $lt to)` |

### 3.2 Config-Driven Model Params

```yaml
analytics:
  ueCommunication:
    samplingInterval: 5   # smfRepPeriod + ground truth 窗口
    inputWindow: 30        # ML 輸入點數 + MongoDB queryLookback (=si×iw)
    outputWindow: 1        # ML 預測步數 + CommDur (=ow×si)
```

### 3.3 ML Service API 適配 (`3cbe03e`)

- `TrafficObservation` 改為 10 個扁平欄位（vol, pkts, thr × ul/dl + total）
- `PredictRequest` 移除 `prediction_steps`（新 ML Service 固定預測 1 步）

### 3.4 模型熱替換 (`008c6a2`)

Daisy 訓練完成後 `swapModelAfterRetrain()` 自動執行：

1. 呼叫 `/model/load` 載入新模型
2. 呼叫 `/model/unload` 卸載舊模型
3. 更新所有訂閱的 `MlModelInfo`
4. 重啟 AccuracyMonitor（對新 modelUrl 建立新 store）

---

## 4. AnLF/MTLF 邏輯單元拆分

按 TS 23.288 精神，將散落在 `notifier/` 與 `processor/` 的 AnLF/MTLF 邏輯整理為獨立 package。

### 4.1 新 Package 結構

```
internal/
  anlf/
    anlf.go          ← AnlfService 入口 (NewAnlfService)
    analytics.go     ← 推論管線 (GenerateUeCommunicationAnalytics, GenerateMockAbnormalBehaviours)
    model.go         ← ML 模型初始化 (InitializeMlModel)
    analytics_test.go← aggregateObservationsByTimeBucket 測試
  mtlf/
    mtlf.go          ← MtlfService 入口，含 onModelSwapped callback
    training.go      ← 訓練觸發 (StartTrainingScheduler, startRetrainWorkflow, submitDaisyTask, swapModelAfterRetrain)
```

### 4.2 職責邊界

| 單元 | 職責 |
|------|------|
| **AnLF** | 執行推論管線、載入 ML 模型、記錄 prediction（供 accuracy 使用） |
| **MTLF** | 訓練排程、接收 accuracy 報告、決策 retrain、模型熱替換 |
| **Processor** | 協調者：wiring callbacks、lifecycle management |

### 4.3 循環依賴解法

`anlf` 與 `mtlf` 彼此不 import；Processor 透過 callback 注入打通：

```
AnLF ← Processor (dependency injection)
MTLF ← Processor (dependency injection)
MTLF.onModelSwapped → Processor.StartAccuracyMonitorForModel
```

### 4.4 刪除的舊檔

| 舊位置 | 移至 |
|--------|------|
| `internal/notifier/analytics.go` | `internal/anlf/analytics.go` |
| `internal/notifier/analytics_test.go` | `internal/anlf/analytics_test.go` |
| `internal/sbi/processor/ml_model.go` | `internal/anlf/model.go` |
| `internal/sbi/processor/mtlf_training.go` | `internal/mtlf/training.go` |

---

## 5. 後續修正與強化

### 5.0 MongoDB 啟動阻塞修正

**問題**：`mongoapi.SetMongoDB` 不驗證真實連線；`CreateCollection` 使用 `context.Background()`（無 timeout），MongoDB 未啟動時 NWDAF 永久阻塞於此，即使仍顯示 "Successfully connected"。

**修正**（`pkg/service/init.go`）：加上 5 秒 timeout context：

```go
collCtx, collCancel := context.WithTimeout(context.Background(), 5*time.Second)
defer collCancel()
collErr := mongoapi.Client.Database(mongodb.Name).CreateCollection(collCtx, ...)
```

MongoDB 不可用時 5 秒後返回錯誤（debug log），NWDAF 繼續正常啟動，降級為 in-memory fallback。

### 5.1 UPF 量測欄位擴充 (`fadd52b`)

按 TS 29.564 / TS 29.571 補齊所有量測欄位：

| 類型 | 新增欄位 |
|------|---------|
| `VolumeMeasurement` | `totalNbOfPackets`, `ulNbOfPackets`, `dlNbOfPackets`；修正 `totalVolume` 未寫入的問題 |
| `ThroughputMeasurement` | `ulPacketThroughput`, `dlPacketThroughput`（pps） |
| 解析 | `parseBitRate()` 支援 bps→Tbps；`parsePacketRate()` 支援 pps→Tpps |

BSON schema 所有新欄位加 `omitempty`，向下相容舊資料。

### 5.2 Group 訂閱時序聚合修正 (`e262e3b`)

**問題**：Group 訂閱下多個 SUPI 各自形成一條時序，原本的 `fetchHistoricalData` 直接串接成 N×inputWindow 筆，ML 模型讀到混亂的多 SUPI 序列。

**修正**：新增 `aggregateObservationsByTimeBucket(obs, samplingInterval)`：

- 依 `samplingInterval` 對齊 bucket，將同一時間窗的所有 SUPI 資料加總
- 10 個數值欄位全部加總（vol, pkts, thr × ul/dl + total）
- 輸出按時間戳升序排列
- MongoDB query limit 改為 `inputWindow × len(corrIds)` 保留足夠原始資料

新增 `analytics_test.go` 7 個測試案例覆蓋：guard 條件、單 SUPI 通過、3-SUPI 聚合、時間戳對齊、排序輸出、非法 Ts 略過、10 欄位加總。

### 5.3 Accuracy Monitor 指標改進 (`4ffed92`)

**指標替換**：`computeNRMSE` → `computeSMAPE`

| 指標 | 問題 |
|------|------|
| NRMSE | 全域均值正規化，burst 流量誤差被放大；無上界 |
| sMAPE | 每個樣本用 `(|actual|+|pred|)/2` 正規化；值域 [0, 2]；不同流量量級行為一致 |

```go
// 每個 UL/DL channel 為獨立樣本
sumSMAPE += |pred - actual| / ((|actual| + |pred|) / 2)
```

`pseudoAccuracy = max(0, 100 - sMAPE×50)`（sMAPE=0→100%，sMAPE=2→0%）

**同時修正** `lookupGroundTruth` in-memory fallback：原本在第一個 SUPI 匹配後即返回，改為累加 Group 內所有 SUPI 的 UL/DL，與 MongoDB path 行為一致。

### 5.4 Daisy 非同步訓練與 hot-swap 協作強化（2026-03-19 ~ 2026-03-24）

對應 commit：`dadbfab`, `c5d18c8`, `38e659e`, `f2a8b9f`, `f4e8dbd`

- Daisy 訓練改為 **async-only**：移除 sync 路徑，統一由 `submitDaisyTask()` 送出，避免阻塞
- 引入 callback 完成流：`/mtlf/training-complete` 回來後由 `HandleTrainingComplete()` 收斂 in-flight 任務
- Daisy task payload 增加 `MODEL_META`，讓 Daisy 側可帶回/追蹤模型資訊
- hot-swap 中 ML Service 操作委派至 AnLF callback（`onModelSwapReady`），明確化 AnLF/MTLF 職責邊界
- AnLF 加入 `WaitLoaded` 保護，避免 retrain 後重複載入相同模型

### 5.5 ADRF E2：即時資料存入（2026-03-25）

對應 commit：`5dbbb13`

- 新增 `AdrfClient.StorageRequest()`（`POST /nadrf-datamanagement/v1/data-store-records`）
- `Processor` 端加入 `adrfBuffer`，支援以 `storageThreshold` 控制 flush
- 新增 `AdrfSmfInfo` 保存 SMF 訂閱上下文，寫入 `NadrfDataStoreRecord` 的 `dataSub.smfDataSub`
- ADRF 未啟用時不影響原有流程（nil-guard）

### 5.6 ADRF E3：Retrain 前資料回補 + Daisy UploadData（2026-03-26 ~ 2026-03-27）

對應 commit：`273a466`, `7dc4307`, `2d6534c`, `db0e1f2`

- retrain 路徑改為 `startRetrainWorkflow()`：ADRF 開啟時先執行 `runAdrfRetrainWorkflow()`
- 新增 `RetrievalSubscribe` / `RetrievalRequest` / `RetrievalUnsubscribe` consumer API
- 新增 callback endpoint：`POST /collector/retrieval-notify`（接收 `fetchCorrIds`, `terminationReq`）
- 以 `TID` 為 retrain job key（`activeJobs sync.Map`），用 `fetchCh` 串接 callback 與抓取迴圈
- `runFetchLoop()` 逐批抓 ADRF record 並呼叫 Daisy `/upload_data` 建 retrain dataset
- 收斂條件：`terminationReq` 計數達標或 watchdog timeout 後關閉 `fetchCh`，再提交 Daisy 訓練
- 補強可觀測性：notify/session/per-ID log，便於 E2E trace 與故障定位

### 5.7 ADRF 存入精度修正（per-item Storage，2026-03-26）

對應 commit：`fb28b9b`

- `HandleUpfNotification()` 轉發 ADRF 前，先把一份 notification 拆成逐 `NotificationItem` 單筆存入
- 避免多 item 打包導致 ADRF 時間窗命中一筆卻回整包，提升 retrieval 精度
- 與既有 `startTime` bucket 對齊策略一致

---

## 6. 關鍵架構說明

### 6.1 儲存架構

- **MongoDB Time Series Collection** (`nwdaf.upfTrafficData`)：主要儲存；`metadata` 含 `ipAddr`, `correlationId`, `supi`, `groupId`, `dnn`
- **In-Memory Ring Buffer**（cap=50，≈8 分鐘）：MongoDB 不可用時的 fallback
- **ADRF Data Store**：retrain 前資料回補來源（`StorageRequest` / `RetrievalRequest`）
- **寫入方式**：`InsertOne`（Time Series 不支援 Upsert）

### 6.2 Group 訂閱流程

```
收到 Group 訂閱 → GroupResolver 解析 SUPIs → 每個 SUPI 建立 SMF 訂閱
→ 蒐集各 SUPI UPF 資料 → aggregateObservationsByTimeBucket → ML 輸入
```

### 6.3 Accuracy Monitor 流程

```
warmup → 定期 checkModelAccuracy()
  → ConsumeMaturePredictions() → lookupGroundTruth()
  → computeSMAPE() → consecutive/EMA trigger strategy
  → 超過閾值 → startRetrainWorkflow() → swapModelAfterRetrain()
```

### 6.4 ADRF + Daisy Retrain 流程（E3）

```
Accuracy breach
  → startRetrainWorkflow(oldModelUrl)
  → runAdrfRetrainWorkflow()
    → per-SUPI RetrievalSubscribe(notifCorrId=TID)
    → ADRF callback /collector/retrieval-notify (fetchCorrIds, terminationReq)
    → runFetchLoop: RetrievalRequest(ids) → Daisy /upload_data(TID, group_id, upfEventNotifs)
    → convergence/watchdog → cleanup unsubscribe
  → submitDaisyTask(TID) to /publish_task
  → Daisy callback /mtlf/training-complete
  → HandleTrainingComplete() → swapModelAfterRetrain()
```

---

## 7. 測試覆蓋

| 測試套件 | 指令 | 重點 |
|---------|------|------|
| 全部單元測試 | `go test ./internal/... ./pkg/factory/... -v` | |
| DB Query | `go test ./internal/context/... -run Query\|Reverse\|Mongo` | 3 函數 × 10 cases |
| Analytics 聚合 | `go test ./internal/anlf/... -run Aggregate` | 7 cases |
| Accuracy Monitor | `go test ./internal/sbi/processor/... -run SMAPE\|Trigger\|EMA` | sMAPE + trigger strategies |
| ModelParams | `go test ./pkg/factory/... -run ModelParams` | 8 cases |
| Daisy Consumer | `go test ./internal/sbi/consumer/... -run Daisy` | async trigger / callback_url / reject path |
| ADRF E2E（文件化） | `docs/testing.md` §7 | 以 fake ADRF + Daisy 驗證 Storage/Retrieval/UploadData 流程 |

---

## 8. 合規狀態

| 類別 | 狀態 |
|------|------|
| 核心訂閱機制 | ✅ |
| UE_COMMUNICATION (Rule-Based & ML-Based) | ✅ |
| 通知機制 (PERIODIC) | ✅ |
| UPF_EVENT 資料蒐集 | ✅ |
| Volume / Throughput 欄位 (TS 29.564) | ✅ |
| Consumer Module 模式 | ✅ |
| Zero Confidence 序列化 (TS 23.288 §6.2) | ✅ |
| repPeriod Propagation (TS 29.508) | ✅ |
| ML 模型整合 (TCN) | ✅ |
| Group ID 解析 (TS 23.502) | ✅ |
| Group 訂閱時序聚合 | ✅ |
| MTLF 訓練觸發 (Daisy FL) | ✅ |
| MongoDB 時序儲存 (Native Time Series) | ✅ |
| Analytics MongoDB 資料讀取 (MongoDB-first) | ✅ |
| Config-Driven Model Params | ✅ |
| ML Service API 適配 | ✅ |
| ML 模型熱替換 (Hot-Swap) | ✅ |
| Accuracy Monitor (sMAPE + trigger) | ✅ |
| Daisy 非同步訓練 callback（202 Accepted + completion callback） | ✅ |
| ADRF StorageRequest（TS 29.575） | ✅ |
| ADRF RetrievalSubscribe/Request/Unsubscribe（TS 29.575） | ✅ |
| Retrain 前 ADRF→Daisy Dataset Pipeline（TID 對齊） | ✅ |
| UPF 通知逐 item ADRF 存入（time-window 精度修正） | ✅ |

---

## 9. 近期重點 Commit（Daisy + ADRF）

| 日期 | Commit | 摘要 |
|------|--------|------|
| 2026-03-27 | `2d6534c` | `runFetchLoop` 加入 per-ID debug log |
| 2026-03-27 | `db0e1f2` | 訓練提交 log 收斂到 `submitDaisyTask` |
| 2026-03-26 | `fb28b9b` | UPF notification 拆 item 後再 ADRF 存入 |
| 2026-03-26 | `7dc4307` | retrieval session/notify logging 補強 |
| 2026-03-26 | `273a466` | ADRF retrieval + Daisy upload retrain 路徑 |
| 2026-03-25 | `5dbbb13` | ADRF StorageRequest（E2）完成 |
| 2026-03-24 | `f4e8dbd` | hot-swap ML Service 操作委派 AnLF |
| 2026-03-24 | `f2a8b9f` | AnLF `WaitLoaded` 防止重複載入 |
| 2026-03-24 | `38e659e` | Daisy task 新增 `MODEL_META` |
| 2026-03-23 | `c5d18c8` | 移除 sync Daisy mode，改 async-only |
| 2026-03-19 | `dadbfab` | Daisy async callback mode 上線 |
