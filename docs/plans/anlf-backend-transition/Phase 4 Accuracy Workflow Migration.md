# Phase 4 Accuracy Workflow Migration

Date: 2026-07-11

Status: Repository-level behavioral parity and R5 cross-process contract verified; full 5GC environment E2E unverified

Parent plan:

- `AnLF Backend Transition Plan.md`

Previous phase:

- `Phase 3.5 Go Package Boundary Consolidation.md`

Follow-up audit:

- `Behavioral Parity Audit.md`
- `Behavioral Parity Remediation Plan.md`
- 2026-07-13 稽核確認 accuracy ownership 與 Go/PyAnLF contract 已切換，但 exact-slot
  matching、UL/DL channel metrics、periodic cadence、traffic scale、scope canonicalization、
  model identity/generation registry 與 lifecycle completion 尚未符合原計畫或搬移前 Go
  行為；目前不得把 repository-level test pass 解讀為 behavioral parity 已完成。

---

## 1. Purpose

本 phase 的目標，是把目前仍留在 `NWDAF/` Go 端的 AnLF-side accuracy workflow
遷移到 `PyAnLF/`，並移除為這段過渡邏輯保留的 Go-side model registry。

Phase 3 已讓 PyAnLF 擁有 observation buffering、analytics generation、model execution
與 report scheduling；但 prediction bookkeeping、ground-truth matching、accuracy metric
generation 仍在 Go。這造成同一個 AnLF runtime 被切成兩半，也使 Go 必須繼續保存
`SharedModelInfo`、`ModelAccuracyStore` 與 model URL correlation。

Phase 4 完成後：

1. PyAnLF 擁有每個 ML Model 的 accuracy monitor
2. PyAnLF 從既有 observation stream 自然形成 ground truth
3. PyAnLF 產生 per-model、per-scope accuracy information
4. Go 將 accuracy information 交給目前的 Go MTLF
5. MTLF 保留 model degradation、retrain 與 reprovision decision policy
6. MTLF/Daisy 的 model provision result 以完整 event 交給 PyAnLF
7. PyAnLF 自行決定受影響的 subscription runtimes 與 replacement/fallback
8. Go 不再以 model URL 維護第二份 model-to-subscription registry

---

## 2. Scope

### 2.1 Included

本 phase 包含：

1. 定義穩定的 `ModelIdentity`，分離 model identity、artifact reference、loaded instance
   與 model generation
2. 將每模型 accuracy monitor 搬到 PyAnLF
3. 將 prediction bookkeeping、ground-truth matching、metric calculation、warmup、
   pending prediction retention 與 reporting trigger 搬到 PyAnLF
4. 保留一個 model monitor 下的多個 monitoring scopes
5. 新增 PyAnLF 到 Go 的 model accuracy report callback
6. 讓 Go MTLF 直接消費新的 accuracy report contract
7. 把現有 mixed accuracy config 拆成 PyAnLF measurement config 與 Go MTLF policy config
8. 讓 MTLF retrain in-flight state 保存 model identity 與 retrain input context
9. 將 Daisy completion 正規化成 model provision event
10. 新增 Go 到 PyAnLF 的完整 model provision event operation
11. 在 external MTLF callback 前同步 model-provision correlation，讓 PyAnLF 可自行 resolve
    affected runtimes
12. 讓 PyAnLF 根據 backend-owned runtime state 決定 replacement fan-out
13. 移除 Go `SharedModelInfo`、`sharedModelRegistry`、`ModelAccuracyStore`、
    `modelAccuracyStores` 與 coordinator model fan-out
14. 縮減 `MlModelInfo`，只保留 Go 標準 procedure 仍需要的 MTLF subscription correlation
15. 補齊 Python domain tests、API tests、Go tests 與 cross-repo live contract tests

### 2.2 Explicitly Excluded

本 phase 不包含：

1. 完整 `Nnwdaf_MLModelMonitor_Register`、`Subscribe`、`Notify`、`Deregister` service
   lifecycle
2. PyAnLF 直接呼叫 Go MTLF package 或未來 PyMTLF
3. PyAnLF 與 PyMTLF 直接通訊
4. 將 MTLF degradation、baseline、decision window 或 retrain policy 搬到 PyAnLF
5. 將 ADRF retrieval、Daisy orchestration 或 training implementation 搬到 PyAnLF
6. 建立 Go-side persistent model catalog
7. 保證 process restart 後延續 pending prediction、monitor window 或 Daisy in-flight task
8. 新增非 `UE_COMMUNICATION` analytics runtime
9. 重寫現有 accuracy metric 或 MTLF policy algorithm
10. 實作 `Nnwdaf_AnalyticsInfo_Request`
11. 因未來 PyMTLF 而預先建立通用 message bus

完整標準 ML Model Monitor service 與 PyMTLF transition 應另立後續工作線。本 phase
只讓內部 contract 使用可對應規格的 model identity、monitoring context 與 accuracy
information 語意。

---

## 3. Planning Basis

本計畫依據以下本地來源：

1. `NWDAF/internal/anlf/accuracy/`
2. `NWDAF/internal/anlf/coordinator/subscription_runtime.go`
3. `NWDAF/internal/anlf/coordinator/model_provision.go`
4. `NWDAF/internal/anlf/contract/`
5. `NWDAF/internal/context/accuracy_store.go`
6. `NWDAF/internal/context/ml_model.go`
7. `NWDAF/internal/mtlf/trigger.go`
8. `NWDAF/internal/mtlf/state_store.go`
9. `NWDAF/internal/mtlf/training.go`
10. `NWDAF/internal/mtlf/adrf_retrieval.go`
11. `NWDAF/internal/mtlf/daisy.go`
12. `NWDAF/pkg/factory/config.go`
13. `PyAnLF/src/py_anlf/core/runtime_manager.py`
14. `PyAnLF/src/py_anlf/core/analytics_runtime.py`
15. `PyAnLF/src/py_anlf/core/observation_store.py`
16. `PyAnLF/src/py_anlf/core/reporting.py`
17. `PyAnLF/src/py_anlf/core/model_manager.py`
18. `nwdaf-docs/specs/TS 23.288/5C Analytics and ML Model Accuracy Monitoring Functional Description.md`
19. `nwdaf-docs/specs/TS 23.288/6 Procedures to Support Network Data Analytics/6.2A Procedure for ML Model Provisioning.md`
20. `nwdaf-docs/specs/TS 23.288/6 Procedures to Support Network Data Analytics/6.2E MTLF-based ML Model Accuracy Monitoring.md`
21. `nwdaf-docs/specs/TS 29.520/4 Services offered by the NWDAF/4.5 Nnwdaf_MLModelProvision Service.md`
22. `nwdaf-docs/specs/openapi/TS29520_Nnwdaf_MLModelProvision.yaml`
23. `nwdaf-docs/specs/openapi/TS29520_Nnwdaf_MLModelMonitor.yaml`
24. `nwdaf-docs/docs/development_policy.md`
25. `free5gc-dev-skill/SKILL.md` 與其 SBI、OpenAPI、lifecycle、concurrency、testing guidance

TS 23.288 的責任線是本 phase 的主要語意依據；TS 29.520 與 OpenAPI 用於確認
`modelUniqueId`、`modelProviderId`、`modelUpdateInd`、monitoring context 與 accuracy
information 欄位。free5GC exemplar 只用於維持 Go server/API/processor/client/config 的
結構慣例，不用來發明不存在的 AnLF Backend 標準 API。

---

## 4. Agreed Decisions

以下方向已確認，不在實作中重新選擇：

1. accuracy monitoring 的主體是 ML Model，不是 NWDAF analytics subscription
2. 每個 model identity 只有一個 monitor；monitor 內可依 analytics context 維護多個 scopes
3. PyAnLF 擁有 ground truth，並從後續收到的 observation 自然形成 actual values
4. PyAnLF 產生 accuracy information 後先 callback Go，再由 Go 交給目前的 MTLF
5. 未來若 MTLF 重構成 PyMTLF，仍先維持 `AnLF Backend -> Go -> MTLF Backend` 邊界
6. 不在 Phase 4 實作完整標準 `Nnwdaf_MLModelMonitor` service lifecycle
7. internal accuracy contract 應參考標準欄位語意，但不可宣稱自身就是標準 SBI operation
8. MTLF 保留 degradation、retrain、reprovision 與 model reselection decision
9. PyAnLF 保留 model sharing、replacement、fallback、runtime usage tracking ownership
10. `modelUniqueId` 不是所有 model provision 無條件必填，但啟用標準語意的 accuracy
    monitoring 時必須有穩定 model identity
11. 缺少 `modelUniqueId` 的模型仍可執行 inference，但不得啟動 accuracy monitor
12. 不用 model URL、Daisy TID 或 PyAnLF loaded-instance UUID 偽造 `modelUniqueId`
13. 本地 default model 使用 PyAnLF config 中的固定 model identity
14. 同一 logical model retrain 時沿用 `modelUniqueId`；只有全新模型才取得新 ID
15. 同 ID update 成功後增加 backend generation、清除舊 pending predictions、重新 warmup，
    並重置 active degradation decision window
16. update 失敗時 generation 不變，舊模型繼續服務，Go 與 PyAnLF 都記錄 failure
17. accuracy measurement config 搬到 PyAnLF；MTLF decision policy config 留在 Go
18. 不長期保留新舊兩套 accuracy workflow
19. Daisy callback 不需要成為 model identity owner；MTLF 透過 task ID 找回 in-flight
    model identity
20. external MTLF callback 所需的 notification/provision correlation 必須在 callback 到達前
    同步給 PyAnLF，不能等 callback 後再由 Go resolve subscription

---

## 5. Current State And Assumptions

### 5.1 Current Accuracy Flow

```text
PyAnLF generates analytics report
    -> Go delivers external analytics notification
    -> Go records prediction after successful external delivery
    -> Go per-model monitor searches MongoDB/in-memory ground truth
    -> Go matches prediction and actual values
    -> Go computes per-scope metrics
    -> Go MTLF evaluates degradation policy
    -> Go MTLF starts ADRF/Daisy retraining
```

這使 accuracy bookkeeping 不合理地依附在 external consumer notification success；也讓
PyAnLF 明明擁有 inference 與 observation，卻無法在最接近資料的位置保存 prediction
與形成 ground truth。

### 5.2 Current Model Key

目前 Go 的主要結構都是以 `modelUrl` 為 key：

1. `SharedModelInfo`
2. `sharedModelRegistry`
3. `ModelAccuracyStore`
4. `modelAccuracyStores`
5. per-model monitor goroutine
6. MTLF `MonitorStateStore`
7. MTLF retraining flag
8. retrain completion 的 old/new model correlation

PyAnLF 目前也以 `model_reference`，通常即 URL，作為 `_shared_models` key；實際載入後的
`model_id` 是 process-local UUID。

### 5.3 Existing Hidden Assumptions

現有 `modelUrl` key 隱含：

1. 相同 URL 永遠代表同一 logical model
2. URL 改變必然代表 model identity 改變
3. URL 指向的內容不會被原地更新
4. 共用 URL 的 subscriptions 使用相容的 analytics event、input/output schema 與 cadence
5. 一個 model 下所有 subscriptions 可共用同一組 inference counter 與 retraining flag
6. 第一筆 pending prediction 的 sampling interval 可代表整個 model monitor
7. 目前只有 `UE_COMMUNICATION`，因此 scope 不需要把 Analytics ID 納入 identity

這些假設不應原樣搬到 PyAnLF。

### 5.4 Current Scope Shape

目前 Go 已採兩層結構：

```text
modelUrl
└── monitor
    ├── target/group scope A
    ├── target/group scope B
    └── target/group scope C
```

這個「一個 model monitor、多個 scopes」策略應保留；需要修正的是 model identity、
scope completeness、inference count 與 generation boundary。

### 5.5 Current Daisy Correlation

Daisy callback 只有：

1. `task_id`
2. `model_url`
3. `status`
4. optional `error`

`task_id` 是一次 training task 與 artifact 的 correlation，不是 3GPP model identity。
目前 `inFlightEntry` 只保存 old model URL 與 `ModelAccuracyStore`，成功後再用 old/new URL
驅動 coordinator fan-out。

---

## 6. Target Responsibility Boundary

| Responsibility | Target owner |
| --- | --- |
| 5GC analytics subscription、SMF/UPF collection | NWDAF Go SBI/processor |
| observation normalization 與 source delivery | NWDAF Go coordinator/client |
| observation retention for inference/ground truth | PyAnLF |
| model identity、generation、artifact 與 loaded instance mapping | PyAnLF |
| model usage and subscription runtime mapping | PyAnLF |
| prediction bookkeeping | PyAnLF |
| ground-truth matching | PyAnLF |
| accuracy metric generation | PyAnLF |
| per-model monitor 與 per-scope state | PyAnLF |
| accuracy callback HTTP edge | NWDAF Go AnLF server/processor |
| baseline、history、decision window | NWDAF Go MTLF |
| model degradation decision | NWDAF Go MTLF |
| ADRF retrain input retrieval | NWDAF Go MTLF |
| Daisy training orchestration | NWDAF Go MTLF |
| complete provision event forwarding | NWDAF Go coordinator/client |
| affected runtime selection and replacement/fallback | PyAnLF |

Go 可以保存標準 procedure 所需的 MTLF subscription ID、notification correlation、
retrain task state 與 MTLF policy state；這不等於 Go 重新擁有 active model runtime。

---

## 7. Model Identity Design

### 7.1 Identity Components

內部 canonical identity：

```text
ModelKey = provider namespace + modelUniqueId
```

建議 contract representation：

```json
{
  "provider_id": "mtlf-instance-1",
  "model_unique_id": 42
}
```

`provider_id` 優先使用 provision 中的 `modelProviderId`；缺少時，由已知 provision source
或本地 config 提供穩定 namespace。`model_unique_id` 必須以 optional/presence-aware 型別
表示，不得用 `0` 代表 missing，因為規格的 `Uinteger` 不應被本地 sentinel 語意污染。

### 7.2 Separate Concepts

必須分開：

| Concept | Meaning | Example |
| --- | --- | --- |
| model identity | logical model identity | `mtlf-instance-1/42` |
| generation | 同一 logical model 的 active revision | `3` |
| artifact reference | 取得該 generation 的位置 | `http://daisy/download/task-b` |
| Daisy task ID | 一次 training task correlation | `task-b` |
| loaded instance ID | PyAnLF process-local loaded object | random UUID |

registry、monitor 與 MTLF policy 不再以 artifact reference 或 loaded instance ID 作 key。

### 7.3 Identity Sources

1. external MTLF provision
   - 使用 notification 中的 `modelUniqueId`
   - provider namespace 使用 `modelProviderId` 或已解析的 MTLF identity
2. PyAnLF default/local model
   - `config.yaml` 明確提供固定 `provider_id` 與 `model_unique_id`
3. retrained model
   - MTLF 從 accuracy report/retrain state 沿用既有 identity
   - Daisy callback 只提供新的 artifact reference
4. new model selected or trained from scratch
   - 由 model provider/MTLF 分配新的 unique ID
   - Phase 4 目前沒有 persistent allocator 時，local from-scratch training 必須從 MTLF
     config取得固定 identity；不得每次 process restart隨機重編

Phase 4 不建立完整 persistent model catalog。由 local config 提供的 ID 必須跨重啟穩定；
dynamic from-scratch ID persistence 留給未來 MTLF/PyMTLF model catalog 工作。

### 7.4 Missing Identity

若模型只有 artifact reference 而沒有 stable identity：

1. PyAnLF 可載入並執行 inference
2. runtime apply outcome 新增明確的 accuracy monitoring state，例如
   `ACTIVE`、`DISABLED`、`UNAVAILABLE_ID`
3. 不建立 accuracy monitor
4. 清楚記錄 model identity missing
5. 不產生 model accuracy report
6. 若未來收到標準 monitor registration/subscription，拒絕該 monitoring request，而不是
   使既有 inference runtime 失效

### 7.5 Generation Transition

同 identity 的 replacement：

1. candidate artifact 先載入並驗證
2. candidate 失敗時保留舊 loaded instance、generation 與 monitor state
3. candidate 成功後才 atomically 切換 active generation
4. 切換成功後停止舊 generation 接收新 predictions
5. 丟棄舊 generation 尚未完成 matching 的 pending predictions
6. 新 generation 重新 warmup
7. PyAnLF report sequence 重新以新 generation 的 namespace 計數
8. Go MTLF 清除該 model 的 active decision windows
9. 可保留 archived metrics，但 archived state 不參與新 generation decision

不同 identity 的 replacement 應建立新 monitor；舊 identity 在沒有 runtime users 後釋放。

---

## 8. Accuracy Monitor Design

### 8.1 Monitor Registry

PyAnLF 目標結構：

```text
accuracy_monitors[ModelKey]
└── ModelAccuracyMonitor
    ├── active_generation
    ├── pending_predictions
    ├── warmup state
    ├── report sequence
    └── scopes[MonitoringScopeKey]
        ├── matched samples
        ├── metric window
        ├── inference count
        └── reporting interval
```

monitor 在第一個帶 stable identity 的 runtime 使用模型時建立；只要仍有任何 runtime
使用該 identity 就保留。最後一個 runtime 離開後，PyAnLF 停止並釋放 monitor。

### 8.2 Monitoring Scope

scope 不應只保存目前的 group/SUPI 字串。canonical scope 至少應涵蓋：

1. Analytics ID / `mLEvent`
2. Target of Analytics Reporting / `tgtUe`
3. analytics or ML event filter
4. 會改變 ground-truth aggregation 意義的必要 runtime context

scope contract 應保留 typed context；內部可另外建立 deterministic `scope_id` 供 map key、
log 與 dedup 使用。不得只傳 opaque scope hash 而丟失 MTLF retrain 所需語意。

### 8.3 Prediction Record

PyAnLF 在 inference 成功、尚未進行 external notification delivery 前，就應建立 prediction
record。至少包含：

1. model identity
2. generation
3. monitoring scope
4. generated time
5. target time/slot
6. predicted values
7. subscription/runtime correlation
8. observation source bindings snapshot

accuracy bookkeeping 不再依賴 external analytics consumer 是否成功收到 notification。

### 8.4 Ground Truth

ground truth 由 PyAnLF 後續收到的 observation 自然形成：

1. prediction 指向 future target slot
2. observation ingestion 持續保存 actual measurements
3. target slot 到達並取得足夠 observation 後進行 matching
4. late observation 可在 bounded retention 內於後續 round 補配
5. 超過 retention/miss policy 後才丟棄 prediction

Phase 4 不再由 Go accuracy monitor查 MongoDB。ADRF/MongoDB 歷史資料仍可由 MTLF
retrain workflow 使用，與 PyAnLF 即時 ground-truth ownership 不衝突。

### 8.5 Metrics And Counts

第一版保留現有 metrics：

1. sMAPE
2. MAE
3. MSE
4. WAPE
5. NRMSE

演算法移植應以現有 Go tests 作 compatibility oracle，不在本 phase 調整公式。

`inferenceNum` 必須明確定義為該 model generation、該 monitoring scope、前次成功
accuracy report 到本次 report 之間的 inference 數量。不得像目前一樣把 model-level
總數複製到每個 scope，造成 MTLF 聚合時重複計數。

### 8.6 Reporting Trigger

目前先使用 periodic measurement/reporting：

1. PyAnLF 依 `check_interval` 檢查成熟 predictions
2. scope 達到 `min_samples` 才產生 report item
3. PyAnLF 可判斷是否達到 AnLF-side reporting threshold，但不做 MTLF degradation decision
4. Go MTLF 接收 report 後執行現有 baseline/window/policy

完整由 MTLF `Subscribe` 傳入 desired metrics、threshold 與 reporting period 的行為不在
Phase 4 實作；目前使用 PyAnLF local config 承接 monitor measurement/report cadence。

---

## 9. Accuracy Report Contract

### 9.1 Operation

新增 AnLF auxiliary server operation：

```text
POST /model-accuracy-reports
```

此 endpoint 是 PyAnLF 與 Go 之間的內部 contract，不是
`Nnwdaf_MLModelMonitor_Notify` 的公開 SBI endpoint。

### 9.2 Payload Shape

建議最小 payload：

```json
{
  "report_id": "acc-mtlf-instance-1-42-2-scope-a-17",
  "report_sequence": 17,
  "generated_at": "2026-07-11T10:00:00Z",
  "model_identity": {
    "provider_id": "mtlf-instance-1",
    "model_unique_id": 42
  },
  "generation": 2,
  "monitoring_context": {
    "analytics_event": "UE_COMMUNICATION",
    "target_ue": {},
    "event_filter": {},
    "scope_id": "scope-a"
  },
  "accuracy_information": {
    "metrics": {
      "sMAPE": 0.12,
      "MAE": 350.0,
      "MSE": 180000.0,
      "WAPE": 0.08,
      "NRMSE": 0.15
    },
    "deviation": 0.12,
    "sample_count": 30,
    "inference_count": 30,
    "actual_traffic_scale": 12000.0,
    "predicted_traffic_scale": 12600.0,
    "window_start": "2026-07-11T09:55:00Z",
    "window_end": "2026-07-11T10:00:00Z"
  },
  "retrain_context": {
    "subscription_ids": ["sub-1", "sub-2"],
    "observation_source_ids": ["corr-1", "corr-2"]
  }
}
```

欄位名稱可以在實作前依 Go/Pydantic model conventions 做小幅修正，但以下語意不可改：

1. report identity 與 model identity 分離
2. generation 必須存在
3. monitoring context 不可只剩 subscription ID
4. metrics 與 inference count 是 per-scope
5. retrain context 是當次 snapshot，不是 Go-owned model registry

### 9.3 Relation To Standard Fields

internal payload 可對應：

| Internal field | Standard-oriented meaning |
| --- | --- |
| `model_unique_id` | `MLModelAccuracyInfo.modelId` |
| `deviation` | `MLModelAccuracyInfo.deviation` |
| `inference_count` | `MLModelAccuracyInfo.inferenceNum` |
| metrics | `modelMetric` / `mlModelAcc` 的本地多 metric extension |
| window | `monitorInterval` |
| analytics event/filter/target | `MLModelMonitorNotify` monitoring context |

目前 contract 保留現有多 metric 與 traffic-scale decision inputs，因此不是標準 schema 的
逐欄複製。未來實作完整 service 時，應在 Go SBI edge 做 generated model mapping。

### 9.4 Acceptance And Idempotency

1. `report_id` 在一個 model generation 內穩定且唯一
2. PyAnLF 對同一 callback retry 必須重用相同 `report_id`
3. Go 對已成功接受的 duplicate report 回 `204`
4. Go 不得因 duplicate report 再次更新 MTLF policy window或觸發 retrain
5. stale generation 回 `409`
6. malformed/invalid identity 回 `400`
7. valid first report 可以建立新的 MTLF policy state，不因本地尚無 model state 回 `404`
8. transient internal failure 回 `503`
9. success 回 `204`

PyAnLF 只對 transport failure、timeout、`5xx` 做有限重試；`400`、`409` 視為
需要記錄但不應無限重試的 contract/state failure。

### 9.5 MTLF In-flight Behavior

若某 model 已在 retraining：

1. Go 仍接受並 deduplicate accuracy report
2. MTLF 可記錄 observability，但不重複 dispatch retrain
3. 不以 `503` 要求 PyAnLF 重送同一批 accuracy information
4. retrain 成功或失敗後再依 policy 決定後續 report 是否可觸發新 task

---

## 10. MTLF Retrain Input Contract

### 10.1 Why Retrain Context Is Needed

目前 MTLF ADRF retrieval 透過 Go `SharedModelInfo` 取得使用某 model URL 的 NWDAF
subscriptions。移除 registry 後，MTLF 仍需要知道觸發 degradation 的 scope 對應哪些資料來源。

PyAnLF 應在 accuracy report 中提供當次 scope 的 retrain context snapshot：

1. NWDAF subscription IDs
2. observation source IDs
3. 未來可加入 ADRF ID、DataSetTag 或其他標準 retrain data reference

這些欄位只供 Go 把 backend-owned usage state 轉成現有 ADRF retrieval inputs，不表示 Go
重新擁有 model-to-subscription mapping。

### 10.2 MTLF State Key

MTLF policy state 改為：

```text
ModelKey + generation + MonitoringScopeKey
```

MTLF retrain in-flight state至少保存：

```text
task ID
model identity
source generation
source artifact reference if needed for training
monitoring context
retrain input context
training state
```

不再保存 `ModelAccuracyStore` pointer，也不再以 old model URL 作主要 identity。

### 10.3 Restart Boundary

Phase 4 維持現有 runtime-only guarantee：

1. Go restart 會失去 Daisy in-flight state
2. PyAnLF restart 會失去 pending predictions 與 active monitor windows
3. restart 後由 subscription/runtime re-apply 重新建立狀態
4. 不為此建立臨時 persistent catalog 或 queue

這個限制必須明確記錄與測試可觀察行為，但不是本 phase blocker。

---

## 11. Model Provision Event Contract

### 11.1 Correlation Precondition

完整 provision event 要由 PyAnLF 自行找 affected runtimes，backend 必須在 callback 到達前
保存標準 procedure correlation。目前 Go 是 callback 到達後才用 `notifCorreId` 或 MTLF
subscription ID resolve NWDAF subscription，再建立 per-subscription apply；這個順序不能直接
刪除而不提供替代。

新增獨立 state-sync operation：

```text
PUT /subscriptions/{subscriptionId}/model-provision-binding
```

建議 payload：

```json
{
  "runtime_revision": 1,
  "notification_correlation_id": "sub-1",
  "mtlf_subscription_id": "mtlf-sub-1",
  "provider_id": "mtlf-instance-1"
}
```

同步順序：

1. Go 在呼叫 external MTLF `Subscribe` 前，先同步已知的 notification correlation 與
   provider namespace
2. 若 pre-subscribe sync 失敗，不建立可能立即 callback、但 backend 無法 resolve 的 MTLF
   subscription
3. MTLF `Subscribe` 成功並回傳 subscription ID 後，再同步完整 binding
4. subscription update/delete 時同步更新或隨 runtime release 一起清理
5. PyAnLF 以 `notifCorreId`、provision subscription ID、provider identity 與 monitoring
   context resolve affected runtimes

這個 binding 是標準 procedure correlation 的 backend copy，不包含 active model usage
decision，也不重新建立 Go-side model registry。

### 11.2 Operation

新增 Go 到 PyAnLF operation：

```text
POST /model-provision-events
```

Go 不再把一個 MTLF callback 展開成多個 `ApplySubscriptionRuntime`。subscription apply
仍用於建立或更新 subscription runtime context；model material update 則使用完整 provision
event operation。

### 11.3 Event Sources

所有 model material update 收斂為同一 contract：

1. external `Nnwdaf_MLModelProvision_Notify`
2. Daisy retrain completion
3. current local MTLF training completion

Go 可以驗證、正規化與補上已知 procedure correlation，但不可先查 Go registry 決定
affected subscriptions。

### 11.4 Minimum Event Shape

```json
{
  "source": "DAISY_RETRAIN",
  "model_identity": {
    "provider_id": "mtlf-instance-1",
    "model_unique_id": 42
  },
  "model_update_ind": true,
  "artifact": {
    "mLModelUrl": "http://daisy/download/task-b"
  },
  "analytics_event": "UE_COMMUNICATION",
  "notification_correlation": {
    "mtlf_subscription_id": "mtlf-sub-1",
    "provision_subscription_id": "prov-sub-1"
  },
  "training_task_id": "task-b"
}
```

external MTLF payload中的 `modelUniqueId`、`modelProviderId`、`modelUpdateInd`、
`mLEventFilter`、`tgtUe`、validity 與 artifact information 應盡可能保留，不應為了本地
convenience 壓縮成 old/new URL pair。

### 11.5 Backend Processing

PyAnLF 收到 event 後：

1. 驗證 model identity 與 artifact
2. 以 provision correlation、current model usage 與 monitoring context 找到 affected runtimes
3. 對同 identity update 準備 candidate generation
4. candidate load/validation 成功後 atomically replacement
5. candidate 失敗時保留舊 generation
6. 更新 model usage、monitor generation 與 runtime revision/state
7. 回傳 aggregate outcome 與 affected runtime count

Go 只記錄 outcome，不逐 subscription 主導 replacement。

### 11.6 Daisy Completion

Daisy 仍只需回傳 `task_id`、`status`、`model_url`、`error`。

MTLF 透過 `task_id` 找回 in-flight model identity：

```text
Daisy task-b success + new URL
    -> MTLF in-flight model identity mtlf-instance-1/42
    -> normalized provision event
       modelUniqueId=42
       modelUpdateInd=true
       artifact URL=/download/task-b
    -> PyAnLF replacement
```

Daisy TID 不寫入 `modelUniqueId`。

---

## 12. Configuration Migration

### 12.1 PyAnLF Measurement Config

PyAnLF 接手：

1. `enabled`
2. `check_interval`
3. `min_samples`
4. `warmup_duration`
5. `metrics_to_record`
6. prediction/ground-truth retention
7. late observation / miss policy
8. matching tolerance與 slot alignment parameters
9. accuracy report callback timeout、retry、dedup capacity
10. default model `provider_id`、`model_unique_id` 與 artifact reference

建議結構：

```yaml
model:
  default:
    provider_id: local
    model_unique_id: 1
    reference: artifacts/initial

accuracy_monitor:
  enabled: true
  check_interval: 90
  min_samples: 2
  warmup_duration: 20
  prediction_retention: 1200
  metrics_to_record:
    - sMAPE
    - MAE
    - MSE
    - WAPE
    - NRMSE
  report_delivery:
    request_timeout: 5
    max_retries: 3
    retry_interval: 1
```

實際 key naming 應遵守 PyAnLF 現有 snake_case config style。

### 12.2 Go MTLF Policy And Provider Config

Go 保留並從 `accuracyMonitor` 改成 responsibility-oriented policy section：

1. `primaryMetric`
2. `recentBufferSize`
3. `minBufferSamples`
4. `minStd`
5. `fixedFloor`
6. `zScoreThreshold`
7. `decisionWindowSize`
8. `requiredHitsInWindow`
9. `scopeStateTTL`
10. degradation policy
11. chronic policy
12. low-traffic overprediction policy

建議名稱：

```yaml
mtlf:
  modelProvider:
    providerId: local-mtlf
    bootstrapModelUniqueId: 1
  accuracyPolicy:
    enabled: true
    primaryMetric: WAPE
    recentBufferSize: 12
    minBufferSamples: 5
    minStd: 0.14
    fixedFloor: 0.05
    zScoreThreshold: 1.3
    decisionWindowSize: 5
    requiredHitsInWindow: 3
    scopeStateTTL: 600
```

`modelProvider` 只供目前 local from-scratch/startup training 在沒有來源 model identity 時
建立穩定 identity。由 accuracy report 觸發的 retrain 必須沿用 report 中的 model identity，
不得改用 bootstrap ID。未來 persistent model catalog 取代這個 bootstrap 方法後，再移除
該 config。

舊 `accuracyMonitor` 不保留雙讀過渡。切換 commit 必須同步更新 config schema、default
config、validation 與 tests。

### 12.3 Retention Constraint

PyAnLF validation 必須確保 observation/prediction retention 足以涵蓋：

```text
prediction horizon + expected observation delay + at least one accuracy check interval
```

若不足，startup config validation 應失敗或至少明確拒絕啟用 accuracy monitor，不可靜默
造成所有 prediction 都在 ground truth 到達前過期。

---

## 13. Planned Code Changes

### 13.1 PyAnLF

建議 package shape：

```text
src/py_anlf/core/accuracy/
├── __init__.py
├── identity.py
├── monitor.py
├── matching.py
├── metrics.py
└── reporting.py
```

責任：

1. `identity.py`
   - model identity、generation、monitoring scope canonicalization
2. `monitor.py`
   - per-model monitor lifecycle、prediction state、warmup、scope registry
3. `matching.py`
   - prediction/observation slot matching、late data、retention/miss policy
4. `metrics.py`
   - compatibility-preserving metrics
5. `reporting.py`
   - report construction、stable ID、sequence、delivery/retry

其他預期修改：

1. `models.py`
   - model identity、accuracy report、provision binding、provision event DTO
2. `core/runtime_manager.py`
   - identity-aware shared models、provision correlation、monitor usage lifecycle、complete
     provision handling
3. `core/analytics_runtime.py`
   - inference outcome 在產生時交給 accuracy monitor
4. `core/observation_store.py`
   - 提供 accuracy matching 所需的 bounded actual observation access
5. `sbi/routers/analytics.py`
   - 新增 model provision event operation，或依責任拆出 model router
6. `config/config.yaml`
   - default identity 與 accuracy measurement config

### 13.2 NWDAF Go

預期新增或調整：

1. `internal/anlf/contract/`
   - `model_identity.go`
   - `model_accuracy_report.go`
   - provision binding DTO
   - 擴充 `model_provision.go`
2. `internal/anlf/`
   - `api_model_accuracy_report.go`
   - route declaration維持在對應 API file，不新增 `routes.go`
3. `internal/anlf/processor/`
   - accuracy callback validation/use-case entry
4. `internal/anlf/client/`
   - provision binding state sync
   - complete model provision event transport
5. `internal/anlf/coordinator/`
   - provision event translation/forwarding
   - 移除 per-subscription model fan-out 與 accuracy hooks
6. `internal/mtlf/`
   - accuracy report input type
   - identity/generation/scope keyed policy state
   - retrain in-flight model identity與 retrain context
   - Daisy completion to provision event
7. `internal/sbi/notifier/`
   - 移除 transitional `RecordAnalyticsReport` hook
8. `internal/context/`
   - 移除 accuracy store與 shared model registry
   - 縮減 `MlModelInfo`
9. `pkg/factory/`、`config/nwdafcfg.yaml`
   - config ownership split
10. `pkg/service/`
   - 移除 Go accuracy monitor construction/wiring

### 13.3 Expected Removals

替代 flow 驗證完成後移除：

1. `internal/anlf/accuracy/`
2. `internal/context/accuracy_store.go`
3. `ModelAccuracyStore`
4. `PredictionRecord`
5. `SharedModelInfo`
6. `sharedModelRegistry`
7. `modelAccuracyStores`
8. coordinator accuracy monitor interface
9. `ApplyRetrainedModel`
10. `applyModelReferenceCorrelation`
11. `removeModelReferenceCorrelation` 中 backend-owned model cleanup
12. notifier accuracy recorder hook
13. MTLF 對 `ModelAccuracyStore` pointer 的依賴
14. Go-side model URL作為 MTLF monitor key 的使用

---

## 14. Implementation Sequence

### Step 1: Introduce Identity Contract

1. 在 Go/Python contract加入 `ModelIdentity`、generation 與 presence-aware unique ID
2. 擴充 local default model config
3. 擴充 internal provision DTO，保留 newer OpenAPI 的 `modelUniqueId`、
   `modelProviderId`、`modelUpdateInd`
4. 新增 model-provision binding state-sync contract
5. 在 external MTLF subscription 前同步 notification correlation，取得 MTLF subscription
   ID 後補齊 binding
6. 加入 JSON contract、ordering 與 rollback tests
7. 此步不切換 accuracy behavior

### Step 2: Build PyAnLF Accuracy Domain

1. 建立 per-model monitor registry
2. 將現有 Go metric tests移植為 Python compatibility tests
3. 接入 inference outcome bookkeeping
4. 接入 observation-based ground truth matching
5. 實作 generation/warmup/retention/scope behavior
6. 先使用 fake report sink驗證，不切換 Go consumer

### Step 3: Add Accuracy Callback

1. 新增 Go contract、API handler與 processor
2. MTLF 改接 identity/generation/scope report
3. 實作 report idempotency與 status semantics
4. PyAnLF 接上真實 callback sender
5. 執行 cross-repo contract test

### Step 4: Cut Over Accuracy Ownership

1. 關閉 Go prediction recorder hook
2. 由 PyAnLF 成為唯一 accuracy report producer
3. MTLF retrain guard從 `ModelAccuracyStore` 移到 MTLF-owned in-flight state
4. 將 ADRF retrieval改用 report retrain context
5. 驗證 multi-subscription、multi-scope、late ground truth與 retrain trigger

### Step 5: Introduce Complete Provision Event

1. 新增 PyAnLF model provision endpoint
2. 驗證 external MTLF callback 可由 pre-synced binding resolve
3. external MTLF callback forwarding改為完整 event
4. Daisy completion改為 identity-preserving provision event
5. PyAnLF 根據 backend state決定 affected runtimes
6. 驗證 replacement success、failure fallback、duplicate completion與 stale generation

### Step 6: Remove Transitional Go State

1. 移除 coordinator fan-out
2. 移除 Go accuracy package與 context stores
3. 移除 shared model registry
4. 縮減 `MlModelInfo`
5. 移除 service wiring與 obsolete interfaces/tests
6. 檢查 `go list` dependency graph與 package boundary

### Step 7: Split Config And Final Verification

1. 搬移 PyAnLF measurement config
2. rename Go MTLF `accuracyMonitor`為 `accuracyPolicy`
3. 移除舊 config 雙讀
4. 執行所有 unit、API、race、lint、build與 live tests
5. 更新主計畫與 phase completion紀錄

每一步可以形成獨立 commit checkpoint，但 Phase 4 只有在舊 Go accuracy flow與 shared model
registry 已移除後才算完成。

---

## 15. Failure Semantics

### 15.1 Accuracy Runtime

1. missing model identity
   - inference繼續
   - monitor不啟動
   - 明確 log
2. insufficient observations
   - 不產生錯誤 report
   - pending prediction保留到 retention/miss boundary
3. malformed observation
   - 沿用 Phase 3 observation contract rejection
4. metric calculation failure
   - 隔離該 monitor round並記錄，不使 analytics report scheduler停止
5. callback failure
   - stable report ID有限重試
   - 超過 retry budget後記錄 dropped delivery，不阻塞 analytics generation

### 15.2 MTLF Decision And Retrain

1. duplicate accuracy report
   - 不重複更新 window或觸發 retrain
2. stale generation
   - 拒絕或忽略，不污染 current policy state
3. retrain already in flight
   - 接受 report但不重複 dispatch
4. Daisy failure
   - 清除 in-flight retrain guard
   - current generation繼續服務
5. unknown Daisy task
   - 記錄並忽略，不建立無來源 provision event

### 15.3 Model Replacement

1. candidate acquisition/load failure
   - old generation保持 active
   - generation不增加
   - monitor不切換
   - Go與PyAnLF都記錄 failure/fallback
2. partial runtime transition
   - shared candidate commit應由 backend保證一致性
   - 不由 Go逐 subscription補償
3. duplicate provision event
   - backend依 event/task identity idempotently處理
4. no affected runtimes
   - 回傳 accepted/no-op或明確 outcome，不由 Go猜測 subscription fan-out
5. provision binding sync failure
   - external MTLF subscribe前失敗時，不建立 MTLF subscription
   - subscribe成功後的 completion sync失敗時，執行 rollback或讓 procedure保持明確失敗，
     不靜默留下無法 resolve的 callback relation

---

## 16. Verification Plan

### 16.1 PyAnLF Unit Tests

至少涵蓋：

1. model identity equality、provider namespace與 missing identity
2. same identity reuse與 different identity separation
3. one model monitor shared by multiple runtimes
4. multiple scopes remain isolated
5. prediction bookkeeping occurs independently of external notification success
6. exact slot ground-truth matching
7. late ground truth matches before expiry
8. expired prediction discard
9. metrics 與現有 Go compatibility fixtures 一致
10. per-scope inference count不重複
11. warmup behavior
12. same-ID successful generation replacement
13. replacement failure preserves old generation
14. last runtime release stops monitor
15. missing-ID model does not start monitor

### 16.2 PyAnLF API Tests

至少涵蓋：

1. provision binding path/body subscription mismatch
2. provision binding stale runtime revision
3. provision binding update與 runtime release cleanup
4. model provision event validation
5. model identity/artifact/update parsing
6. duplicate provision event idempotency
7. replacement outcome response
8. accuracy callback stable report ID與 retry classification
9. config validation for retention boundary

### 16.3 Go Unit Tests

至少涵蓋：

1. accuracy report JSON parsing與 identity validation
2. handler status mapping
3. report dedup與 stale generation
4. MTLF state key使用 model identity + generation + scope
5. existing degradation/chronic/low-traffic policy behavior保持
6. retrain in-flight guard不再依賴 `ModelAccuracyStore`
7. retrain context映射到 ADRF retrieval targets
8. Daisy completion沿用 model identity
9. Daisy TID不成為 model unique ID
10. full provision event forwarding
11. provision binding在 external MTLF subscribe前建立並於 release清理
12. callback可在不使用 Go fan-out的情況下 resolve backend runtimes
13. replacement failure log/outcome behavior
14. subscription deletion不再操作 Go shared model registry
15. config split與 validation

### 16.4 Cross-repo Live Contract Test

至少驗證：

1. 啟動真實 PyAnLF與 Go test callback/processor path
2. 建立兩個使用相同 model identity、不同 scopes的 subscriptions
3. 確認 PyAnLF只建立一個 model monitor
4. 傳送 observation並產生 predictions
5. 傳送後續 observation形成 ground truth
6. PyAnLF callback兩個隔離的 scope accuracy reports
7. Go MTLF接受 report並更新 policy state
8. 以低門檻或 fake policy觸發 retrain
9. fake/real Daisy completion回傳新 URL
10. Go送出相同 model ID、`modelUpdateInd=true`的 provision event
11. PyAnLF自行更新所有 affected runtimes
12. replacement failure時舊 generation仍能產生 analytics
13. release全部 runtimes後 monitor被停止
14. external provision callback使用預先同步的 binding完成 runtime resolution

### 16.5 Repository-level Commands

`NWDAF/`：

```bash
go test ./...
make lint
make build
go test -race ./internal/anlf/... ./internal/sbi/... ./internal/context ./internal/mtlf/...
```

`PyAnLF/`：

```bash
pytest
```

另執行：

1. `git diff --check`
2. Go package dependency inspection
3. 真實 cross-repo HTTP contract test

若完整 5GC、MongoDB、ADRF、Daisy環境不可用，必須分開說明 unit/API/live-contract 已驗證
到哪一層，不可把 local callback test宣稱為完整 end-to-end validation。

---

## 17. Completion Criteria

Phase 4 完成必須同時滿足：

1. PyAnLF 擁有 prediction bookkeeping與 ground truth
2. PyAnLF 擁有每模型 monitor與 per-scope accuracy generation
3. model identity不再以 URL或 runtime UUID表示
4. missing-ID model可 inference但不啟動 monitoring
5. Go MTLF可消費 identity/generation/scope aware report
6. MTLF degradation/retrain policy仍在 Go且主要行為不變
7. Daisy retrain沿用 logical model ID並產生新 artifact/generation
8. complete provision event由 Go轉交 PyAnLF
9. model-provision binding在 callback前同步並由 PyAnLF保存
10. affected runtime selection與 fallback由 PyAnLF決定
11. Go coordinator不再做 model fan-out
12. `SharedModelInfo`與 shared model registry已移除
13. `ModelAccuracyStore`與 Go accuracy package已移除
14. Go/Python config ownership已拆分
15. Python unit/API tests、Go tests、targeted race、lint、build與 cross-repo contract通過
16. 文件記錄實際 verification與任何未完成的環境級測試

如果 PyAnLF accuracy flow已加入但 Go舊 flow仍作為長期 fallback，或 Go仍以 model URL
維護 replacement fan-out，本 phase 不算完成。

### 17.1 Implementation Record

2026-07-11 完成的是 Phase 4 的 repository-level responsibility migration 與 contract
cutover。2026-07-13 稽核確認下列紀錄不構成 historical Go accuracy behavioral parity
證明；已知 matching、metric、cadence、identity 與 lifecycle 差異統一記錄於
`Behavioral Parity Audit.md`：

1. `NWDAF/` commit：`64fdf0a refactor(anlf): move accuracy workflow to backend`
2. `PyAnLF/` commit：`60994c3 feat(accuracy): own model monitoring workflow`
3. PyAnLF現在擁有model identity/generation mapping、prediction bookkeeping、observation-based
   ground truth、per-model/per-scope monitor、metric calculation與accuracy report delivery
4. Go AnLF auxiliary server新增`POST /model-accuracy-reports`，並將identity/generation-aware
   report交給Go MTLF decision policy
5. external MTLF subscription建立前後會同步model-provision binding；completion sync失敗時
   rollback external subscription
6. external MTLF callback與Daisy completion會正規化成完整model provision event，再由
   PyAnLF自行resolve affected runtimes與執行replacement/fallback
7. startup Daisy training透過`mtlf.modelProvider`取得固定bootstrap identity
8. Go notifier不再記錄prediction；MTLF retrain guard與ADRF retrain inputs不再依賴Go AnLF
   model usage registry
9. `internal/anlf/accuracy`、`ModelAccuracyStore`、`SharedModelInfo`、兩個context registry與
   coordinator model fan-out均已移除
10. Go config已切換為`mtlf.accuracyPolicy`；PyAnLF `accuracy_monitor`持有measurement、
    matching、retention與delivery設定

實際驗證結果：

1. `PyAnLF/`：`uv run pytest`，24項測試通過
2. `NWDAF/`：`make lint`、`go test ./...`與`make build`通過
3. `NWDAF/`：AnLF、MTLF、context與SBI targeted race tests通過
4. 真實Go到PyAnLF HTTP contract通過，涵蓋runtime apply、observation binding、observation
   ingestion、analytics callback與runtime release
5. `go list -deps ./cmd`確認主程式不再依賴`internal/anlf/accuracy`
6. 兩個implementation repos的`git diff --check`通過

尚未執行完整5GC、external MTLF、MongoDB/ADRF與Daisy同時運行的環境級實驗。上述結果只
代表repository responsibility migration、unit/API/race/build與local live contract已完成，
既不宣稱完整部署環境的端到端實驗已完成，也不宣稱與搬移前Go行為完全等價。

### 17.2 R5 Closure Update

2026-07-14 R5重新驗證historical metric、matching、scope與periodic monitor fixtures，並補上canonical
PyAnLF accuracy JSON經Go AnLF API、processor與MTLF policy state的cross-repository test。UL/DL metrics、
traffic scales、sample count、identity/generation與scope均未在Go boundary被重算或改單位。Repository-level
behavioral parity與cross-process contract已驗證；完整5GC、external MTLF、Daisy與ADRF V3仍未驗證。

---

## 18. Risks And Controls

1. model identity與artifact reference仍混用
   - 以不同型別與 contract欄位強制分離
2. same-ID update混入舊 predictions
   - prediction綁定 generation，成功 replacement後清空舊 pending state
3. multi-scope inference count重複
   - counter定義為 per-scope interval並加入測試
4. retention不足導致永遠沒有 ground truth
   - config validation與 late-arrival tests
5. accuracy callback重送造成重複 retrain
   - stable report ID與 Go dedup
6. 移除 SharedModelInfo後 ADRF retrieval失去資料來源
   - accuracy report攜帶 retrain context snapshot後才移除 registry
7. callback先於 backend correlation到達而無法 resolve runtime
   - external MTLF subscribe前先同步 notification correlation，並測試 ordering/rollback
8. provision event切換時部分 runtimes更新
   - backend candidate/commit/fallback tests，不在 Go實作 fan-out補償
9. config ownership拆分改變 threshold behavior
   - measurement與decision欄位逐項 mapping，保留 Go policy fixtures
10. 將未來 PyMTLF需求過早塞入 contract
   - 只使用 model identity、monitoring context、accuracy information與 provision event
11. 一次移除過多過渡 state難以 review
    - 依 Step 1至Step 7 checkpoint落地，最後才刪除舊 flow

---

## 19. Future Work

Phase 4 之後可另外規劃：

1. MTLF backend boundary與 PyMTLF repository
2. 完整 `Nnwdaf_MLModelMonitor` Register/Subscribe/Notify/Deregister
3. MTLF/PyMTLF persistent model catalog
4. process restart後的 durable model、monitor與 retrain state
5. `DataSetTag`、ADRF ID與標準 retrain data reference完整整合
6. 多個 NWDAF AnLF向同一 MTLF提供 accuracy information
7. non-`UE_COMMUNICATION` accuracy monitor
8. analytics feedback information

這些 future work 不影響 Phase 4 在目前單一 NWDAF內完成 AnLF accuracy ownership
migration。
