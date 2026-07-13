# R2 Accuracy Measurement Parity

Date: 2026-07-13

Status: Implemented, locally verified, and committed

Parent plan:

- `Behavioral Parity Remediation Plan.md`

Test framework:

- `R0 Compatibility Oracle Framework.md`

Audit and decision record:

- `Behavioral Parity Audit.md`

Historical design plan:

- `Phase 4 Accuracy Workflow Migration.md`

Affected repositories:

1. `PyAnLF/`
2. `NWDAF/`，移除accuracy migration完成後的過渡buffer/config與重複measurement gates
3. `nwdaf-docs/`

---

## 1. Purpose

R2恢復Phase 4搬移前Go accuracy measurement的domain behavior，使PyAnLF在相同prediction、
source-aware observations、sampling interval與periodic check time下，產生與historical Go相容的
ground-truth pairs、metrics、traffic scales與per-scope accuracy reports。

本stage與R0 Accuracy slice一起進行：先從historical Go production code與tests手工建立golden
fixtures，保存current PyAnLF的initial failure evidence，再修正PyAnLF。R2不把accuracy ownership搬回
Go，也不撤銷目前`POST /model-accuracy-reports`的internal callback boundary。

R2完成只代表accuracy measurement與MTLF input units恢復。Model identity/generation/provision
atomicity由R3處理；subscription reporting completion與runtime cleanup由R4處理。

---

## 2. Goals

1. 每個model output step建立獨立prediction record
2. Prediction與ground truth全程分開保存UL與DL
3. Accuracy evaluation恢復為per-model periodic check，不由observation ingest觸發
4. 每輪使用check當下可取得的所有source observations作best-effort aggregation
5. Ground truth使用target-slot equality，不使用absolute timestamp tolerance
6. 同source/IP/slot維持last-wins，再跨source/IP聚合UL與DL
7. 未matched prediction依derived miss-count policy保留或清除
8. Matched samples只屬於當次check round，不跨round累積
9. 恢復historical MAE、MSE、sMAPE、WAPE、NRMSE與traffic-scale公式
10. Scope identity對UE Communication target作穩定canonicalization
11. Confidence-zero作為data readiness gate，不再使用固定秒數warmup
12. Confidence-zero不進baseline，但有structured log與counter
13. Report context從本輪matched prediction snapshots重建
14. Accuracy callback維持有限retry後drop，但使用stable report ID並提供失敗observability
15. Go MTLF不修改既有threshold以遷就current錯誤scale
16. Observation storage恢復historical per-source/IP fixed-size ring buffer

---

## 3. Scope

### 3.1 Included

PyAnLF主要修改：

1. `src/py_anlf/core/accuracy/monitor.py`
2. `src/py_anlf/core/accuracy/metrics.py`
3. `src/py_anlf/core/accuracy/identity.py`
4. `src/py_anlf/core/accuracy/reporting.py`
5. `src/py_anlf/core/observation_store.py`
6. `src/py_anlf/core/runtime_manager.py`
7. `src/py_anlf/core/analytics_runtime.py`的internal prediction-step output seam
8. `src/py_anlf/models.py`中必要但不改變wire meaning的accuracy model validation
9. `config/config.yaml`的analytics buffer與accuracy measurement config
10. R0 Accuracy fixtures與`test_accuracy_parity.py`
11. Existing accuracy/runtime/config tests的回歸補強

NWDAF修改：

1. accuracy callback到MTLF policy input的regression tests
2. 移除`groundTruthRetention.ringBufferSize`與Go `RawUpfData` append/cap state
3. 移除Go accuracy policy中已搬到PyAnLF的`CheckInterval`、`WarmupDuration`與
   `MetricsToRecord`殘留欄位/helper
4. 移除Go `minSamples` config與degradation-reference重複sample-count gate
5. 必要的factory、UPF processor與MTLF policy tests更新

R2不修改internal HTTP payload。Scope fallback可使用已同步到PyAnLF的
`ObservationBinding`中`source.supi`與`subscription_scope.original_group_id`，不需要新增Go-owned
resource lookup API。

### 3.2 Excluded

1. R1已完成的analytics shaping algorithm
2. External analytics notification payload與report cadence
3. Model registry key、same-URL reload、artifact cache與provision atomicity
4. Model provision event idempotency
5. `maxReportNbr`、`monDur` completion callback與Go inactive transition
6. SMF/UPF collection cleanup
7. Go observation delivery single-worker、queue、retry與batch contract
8. Expected-source completeness protocol與settling timeout
9. Accuracy report process-local或durable outbox
10. MTLF baseline、z-score、traffic gates與retrain thresholds重新校正
11. One-shot `Nnwdaf_AnalyticsInfo`支援
12. Non-`UE_COMMUNICATION` accuracy monitoring

如果實作發現必須修改上述boundary才能恢復R2 parity，應停止並依development policy提出blocker，
不得以弱化fixture或跳過historical invariant繼續。

---

## 4. Historical Oracle

### 4.1 Primary Baseline

| Behavior | Historical source |
| --- | --- |
| Accuracy monitor與periodic loop | `NWDAF@a7d0693:internal/anlf/accuracy/monitor.go` |
| Prediction state與snapshot/resolve | `NWDAF@a7d0693:internal/context/accuracy_store.go` |
| Per-scope report與metrics | `NWDAF@a7d0693:internal/anlf/accuracy/report.go` |
| Scope canonicalization | `NWDAF@a7d0693:internal/anlf/accuracy/scope.go` |
| Compatibility tests | `NWDAF@a7d0693:internal/anlf/accuracy/*_test.go` |
| Exact target-slot與miss-count policy | commits `e8c8978`、`5ca01e9` |
| Scope snapshot intent | commit `6529345` |
| Historical warmup context | commits `71a6564`、`df628b0`；R2已核准由confidence readiness取代 |
| Scoped MTLF input intent | commit `951a674` |

`a7d0693`是主要pre-migration baseline，但每個fixture仍須記錄最接近該行為的feature commit或test
name，不能只以單一commit概括所有accuracy semantics。

### 4.2 Historical Invariants

實作與fixture必須固定以下行為：

1. 一個model有自己的periodic accuracy check lifecycle
2. Check先取得pending prediction snapshot，再在round結束時只resolve該snapshot內的records
3. Snapshot後加入的prediction不被該round刪除或增加miss count
4. 每個prediction保留獨立UL、DL、target slot、subscription與scope snapshot
5. Actual timestamp相對target slot使用Go `math.Round`語意計算slot key
6. 只有slot key為0的observation可作ground truth
7. Candidate query可使用`target ± sampling_interval`，但它不是matching tolerance
8. 同一source/IP在target slot內的後到record覆寫先到record
9. 不同source/IP的target-slot values分別聚合後再相加
10. UL與DL各自是metric sample；一個prediction/actual pair產生兩個numeric samples
11. `sample_count`仍表示prediction/actual pair數，不是展開後的channel數
12. WAPE與NRMSE在actual denominator為0時回0
13. Actual與predicted traffic scale使用all-channel mean absolute value
14. Matched prediction在本輪移除；未matched prediction增加miss count
15. `max_miss_count = ceil(2 * sampling_interval / check_interval) + 1`
16. 每個scope獨立套用`min_matched_predictions`
17. Scope target list會trim、sort、deduplicate
18. Prediction保存建立當下的scope，不受後續subscription update回寫
19. 每個source/IP session最多保留`ring_buffer_size`筆raw observations，overflow從最舊record移除

### 4.3 Approved Differences

以下不是需要恢復的regression，而是已核准的新語意：

1. PyAnLF維持accuracy owner，不搬回Go
2. 不使用固定秒數warmup；confidence-zero prediction作為資料未ready的gate，另記log/counter
3. Model generation replacement清除舊generation accuracy state，但新generation的confidence-positive
   predictions可立即進入正常periodic lifecycle
4. Multi-source採periodic best-effort，不等待expected source completeness
5. Accuracy callback有限retry後drop，不建立outbox
6. Report採目前identity/generation-aware internal contract
7. `MonitoringContext`保存event、target與filter，不退回只有human-readable Go scope key
8. Accuracy state以per-scope inference count取代historical model-global count複製
9. Scope無法從concrete target或bindings解析時，只停用該subscription的accuracy bookkeeping；
   analytics/inference維持正常

Approved difference fixtures必須標記`classification: approved_change`，不得冒充historical invariant。

---

## 5. Current Mismatch

| Audit finding | Current behavior | Required behavior |
| --- | --- | --- |
| BP-07 | `abs(observed-target) <= tolerance` | Go-compatible target slot-key equality |
| BP-08 | 每次source ingest立即消耗prediction | per-model periodic best-effort round |
| BP-09 | UL與DL壓成單一total | UL/DL分離保存與計算 |
| BP-10 | NRMSE與zero denominator公式改變 | 恢復historical formulas |
| BP-11 | traffic scale使用sum | 使用all-channel mean absolute value |
| BP-12 | 沒有`check_interval` worker | config-driven periodic check |
| BP-13 | matched samples跨ingest累積 | round-local samples |
| BP-14 | current time-based warmup保留無效prediction state | 移除time-based warmup，以confidence readiness取代 |
| BP-15 | sender結果被忽略且無失敗counter | finite-drop保留，但補stable ID與observability |
| BP-16 | report後pending context不一致 | context由prediction snapshot重建 |
| BP-17 | 固定取第一個event且list未canonicalize | 明確找UE Communication並normalize |
| BP-18 | 缺少compatibility tests | 建立R0 Accuracy fixture suite |
| BP-28 | 多步prediction壓成一筆 | 每個model output step一筆record |
| BP-29 | confidence-zero沒有明確observability | skip baseline並記log/counter |
| BP-30 | measurement config不完整 | 補check/metric config與derived lifecycle |
| BP-31 | per-source time retention取代per-session count cap | 恢復per-source/IP fixed-size ring buffer |

---

## 6. Target Package Shape

```text
PyAnLF/src/py_anlf/core/
├── analytics_runtime.py
├── observation_store.py
├── runtime_manager.py
└── accuracy/
    ├── identity.py
    ├── metrics.py
    ├── monitor.py
    └── reporting.py

PyAnLF/tests/
├── fixtures/behavioral_parity/
│   ├── README.md
│   ├── accuracy_matching_cases.json
│   ├── accuracy_metric_cases.json
│   └── accuracy_scope_cases.json
├── test_accuracy_parity.py
├── test_accuracy_monitor.py
├── test_accuracy_reporting.py
└── test_runtime_manager.py
```

Responsibility：

1. `analytics_runtime.py`產生external analytics report與internal per-step prediction outcomes
2. `observation_store.py`擁有source/session ring buffers、binding、batch dedup與immutable snapshot
3. `runtime_manager.py`在inference完成後、external report delivery前記錄prediction
4. `identity.py`只負責UE Communication monitoring context canonicalization
5. `metrics.py`提供不依賴runtime state的pure historical metric functions
6. `monitor.py`擁有prediction state、confidence readiness、periodic round、matching與report construction
7. `reporting.py`只擁有accuracy callback transport、retry classification與delivery logs

本stage不為每個小函式建立新module。若`monitor.py`在實作後仍過度混合，可在不改變上述責任的
前提下拆出`matching.py`；不得為預期中的R3/R4需求先建立空抽象。

---

## 7. Internal Data Design

### 7.1 Analytics Generation Result

目前external `AnalyticsUeCommunication`會把多個model outputs聚合成一個communication。R2不能從該
聚合結果還原每一個future slot，因此新增internal result，不改external report shape：

```text
AnalyticsGenerationResult {
  report
  prediction_steps[]
}

PredictionStep {
  target_slot_time
  predicted_ul_volume
  predicted_dl_volume
  confidence
}
```

規則：

1. `target_slot_time = last_shaped_slot + (step_index + 1) * sampling_interval`
2. Step順序使用predictor output order
3. External report仍沿用目前核准的aggregation與wire contract
4. Current predictor只有一個step時，external behavior完全不變
5. Fallback或inference exception只產生confidence-zero external analytics，不建立accuracy record
6. Tests使用multi-step fake predictor固定BP-28，即使current production model仍只有一步

### 7.2 Prediction Record

Target record至少包含：

```text
Prediction {
  record_id
  subscription_id
  monitoring_context_snapshot
  model_identity
  generation
  generated_at
  target_slot_time
  sampling_interval_seconds
  predicted_ul_volume
  predicted_dl_volume
  observation_source_ids
  miss_count
}
```

Requirements：

1. `record_id`在該model monitor內單調遞增，用於snapshot/commit
2. Model identity與generation在建立時固定
3. Scope、subscription與source IDs使用建立時snapshot
4. Source IDs排序並去重，避免後續binding update回寫historical prediction
5. Sampling interval使用建立prediction時的effective runtime value
6. Volumes維持integer domain，不先轉成UL+DL scalar
7. Confidence-zero不建立record
8. Detach subscription時清除該subscription尚未resolve的records

### 7.3 Monitor State

```text
ModelMonitor {
  identity
  generation
  runtime_ids
  scopes[scope_id]
  next_record_id
  worker_stop/wake
}

ScopeState {
  context
  pending_predictions
  inference_count_since_report
  next_report_sequence
}
```

Matched actual/predicted samples不存放在long-lived `ScopeState`；它們是單次check round的local
values。這直接防止BP-13與report後context清空不一致。

`inference_count_since_report`只計入該scope、該generation、非confidence-zero prediction records。
達成`min_matched_predictions`並建立report後歸零；未建立report時可繼續計數，但matched samples本身不得保留到
下一round。Finite delivery最終失敗仍視為該report round已結束，因此不回滾count或重建舊samples。

### 7.4 Per-session Observation Ring

Historical Go的`RawUpfData`不是time-based TTL，而是每個correlation ID/IP session固定筆數：

```text
SourceBuffer
└── sessions[ipv4_address]
    └── observations (oldest -> newest, maxlen=ring_buffer_size)
```

Requirements：

1. Session ownership至少以`(observation_source_id, ipv4_address)`區分
2. 每個session保留ingest order
3. Append後超過`ring_buffer_size`時只移除該session最舊records
4. 不因其他session流量較多而擠掉本session observations
5. Subscription snapshot仍以stable source order展平為`StoredObservation`
6. Last binding release釋放整個source及其session buffers
7. `batch_dedup_capacity`只限制processed batch IDs，不得與observation容量共用
8. `ring_buffer_size`必須大於等於`input_window`

Current `SourceBuffer.observations`的single flat list需調整為可維持per-session cap的結構；對外
source-aware snapshot contract不變，因此R1 alignment algorithm不需改寫。

---

## 8. Scope Canonicalization

### 8.1 Event Selection

1. 在`event_subscriptions`中明確尋找`UE_COMMUNICATION`
2. 不使用第一個event作隱含fallback
3. 找不到UE Communication時不建立accuracy prediction
4. Mixed-event subscription中event順序不得影響scope

### 8.2 Target Normalization

1. `supis`與`intGroupIds`中的值trim whitespace
2. 移除空值
3. Sort並deduplicate set-like ID lists
4. 同時具有group與SUPI時兩者都保留
5. 空target或`anyUe=true`維持unresolved，不產生通用scope把不同UE混在一起

若subscription target unresolved，使用runtime目前已同步的bindings回退：

1. `subscription_scope.original_group_id`組成group IDs
2. 沒有original group的binding使用`source.supi`組成direct SUPIs
3. 回退結果仍經過相同trim/sort/deduplicate
4. Bindings仍無法形成target時不啟動該prediction的accuracy bookkeeping

Confirmed decision（2026-07-13）：採用上述concrete target、binding fallback、最後skip accuracy的
三段式處理。無法解析scope不建立共用`unknown` baseline；記structured log與
`accuracy_scope_unresolved_total`，但不影響analytics/inference。

### 8.3 Stable Scope ID

`MonitoringContext`保留normalized：

1. `analytics_event`
2. `target_ue`
3. `event_filter`
4. `scope_id`

`scope_id`由canonical JSON的SHA-256穩定前綴建立。Object keys排序；只有已知set-like target ID
lists排序，未知filter arrays維持原始順序，避免擅自改變可能具有順序語意的欄位。相同語意的SUPI/
group ordering與duplicates必須得到同一scope ID。

---

## 9. Accuracy Monitor Lifecycle

### 9.1 Worker Ownership

1. 每個active model identity擁有一個periodic worker
2. 第一個runtime attach時建立monitor與worker
3. 後續相同identity attach只增加runtime reference
4. 最後一個runtime detach時停止worker並移除monitor
5. Registry shutdown停止並join所有workers
6. Worker不持有registry lock執行observation snapshot、metric calculation或HTTP sender

Model identity/generation registry key的完整修正屬R3。R2沿用current `(provider_id,
model_unique_id)` monitor key與generation field，但R2 state transition不得重新以artifact URL作accuracy
identity。

### 9.2 Accuracy Readiness

R2不再使用固定秒數warmup：

1. Predictor因歷史資料不足而產生`confidence == 0`時不建立accuracy prediction
2. 每次confidence-zero記structured log與counter
3. Confidence-positive prediction立即進入正常pending lifecycle
4. Worker建立後第一個check仍依正常`check_interval`發生，不額外等待warmup timer
5. Generation replacement原子清除舊generation pending records、per-scope inference counters與舊round
   commit資格
6. 新generation的confidence-positive predictions不因任意秒數被丟棄

Historical Go的startup warmup曾用來抑制資料不足時的startup noise；current predictor已直接以
confidence表示input window readiness，繼續維護第二套time gate只會丟棄可用prediction。這是
2026-07-13確認的approved change，fixture應標記`approved_change`。

### 9.3 Deterministic Test Seam

Production worker使用UTC wall clock與`Event.wait()`。Domain tests必須能：

1. 注入clock
2. 關閉background worker
3. 以明確`check_time`呼叫單次due check
4. 不用`sleep`等待matching、generation reset或expiry正確性

Thread lifecycle另以event/barrier component tests驗證；不得用長時間sleep賭scheduler ordering。

---

## 10. Periodic Check Algorithm

每個model check round分為snapshot、evaluate、commit、deliver四段。

### 10.1 Snapshot

在monitor lock內：

1. 驗證monitor仍active且generation一致
2. 為每個scope複製pending prediction records
3. 保存本輪snapshot record IDs
4. 保存本輪generation/version
5. 不移除或改寫state

### 10.2 Evaluate

離開monitor lock後：

1. 依prediction subscription取得R1 source-aware observation snapshot
2. 只使用prediction snapshot中的`observation_source_ids`
3. 執行target-slot matching與UL/DL aggregation
4. 建立每scope本輪matched pairs
5. 標記snapshot內未matched IDs
6. 每個prediction的matching互相獨立

不等待所有expected sources。Check當下已到達的sources構成本輪best-effort ground truth；晚到資料可在
下一round匹配仍pending的prediction，但已matched prediction不重新打開。

### 10.3 Commit

重新取得monitor lock：

1. 若monitor已移除或generation/version改變，丟棄舊evaluation result
2. 只resolve snapshot IDs
3. Matched records移除
4. Unmatched records增加`miss_count`
5. `miss_count >= max_miss_count`時移除並增加expiry counter
6. Snapshot後新增records保持原狀
7. 每scope matched pair數達`min_matched_predictions`才建立report
8. 未達`min_matched_predictions`時本輪matched samples仍discard，不跨round累積
9. Report sequence與inference count在state commit時原子更新

### 10.4 Deliver

離開monitor lock後傳送immutable reports。Sender retry期間prediction record、attach/detach與下一個
state transition不得被HTTP lock住。Report object與`report_id`在所有retry中不變。

### 10.5 Miss-count Derivation

```text
max_miss_count = ceil(2 * sampling_interval_seconds / check_interval_seconds) + 1
```

規則：

1. 每個prediction使用自己snapshot的sampling interval
2. `check_interval`來自active accuracy config
3. 結果至少為1
4. `2*SI`只代表target slot完成與UPF notification delay的等待尺度
5. 不恢復`TargetTime + 2*SI`一次性maturity gate
6. 不把`2*SI`解釋成matching tolerance

---

## 11. Ground-truth Matching

### 11.1 Slot Identity

對candidate observation：

```text
slot_key = go_round(
  (observation_unix_second - target_slot_unix_second)
  / sampling_interval_seconds
)
```

只有`slot_key == 0`可配對。使用R1 `alignment.go_round()`與相同Unix whole-second conversion，
不得再次實作Python builtin `round()`版本。

可先以`target_slot_time ± sampling_interval`作candidate prefilter以降低掃描量；相鄰slot即使落在
prefilter window內仍須由slot-key equality拒絕。

### 11.2 Dedup And Aggregation

對單一prediction：

1. Filter到prediction保存的source IDs
2. 依source snapshot中的ingest order處理records
3. Session key使用`(observation_source_id, ipv4_address)`
4. 同session且slot key為0的後到record覆寫先到record
5. 不同source/IP sessions分別保留
6. 最後將所有保留sessions的UL相加、DL相加
7. 至少有一個session contributor才視為matched

這對應historical `correlation ID + IP session` last-wins。不得將source ID在flatten後丟失，也不得
把同source內多筆同slot record全部相加。

### 11.3 Observation Buffer Boundary

R2移除`source_retention` time policy，恢復historical `ring_buffer_size` count policy。這個buffer同時
提供analytics input與accuracy ground-truth lookup，但不控制：

1. Go的SMF/UPF subscription或HTTP source lifecycle
2. Prediction miss/retry/discard lifecycle
3. Model lifecycle
4. ADRF/MongoDB retention

Prediction是否等待late ground truth仍由derived `max_miss_count`決定。正常設定
`ring_buffer_size >= input_window`時，current `50 >= 30`也遠高於accuracy在數個check rounds內所需的
近期actual records。

---

## 12. Metric And Report Semantics

### 12.1 Channel Expansion

每個matched pair：

```text
actual_samples    += [actual_ul, actual_dl]
predicted_samples += [predicted_ul, predicted_dl]
```

`sample_count`等於matched pair數；metric denominator使用展開後的`2 * sample_count`個channel
samples。

### 12.2 Formulas

```text
MAE   = mean(abs(predicted - actual))
MSE   = mean((predicted - actual)^2)
sMAPE = mean(2 * abs(predicted - actual)
             / (abs(actual) + abs(predicted)))
WAPE  = sum(abs(predicted - actual)) / sum(abs(actual))
NRMSE = sqrt(MSE) / mean(abs(actual))
```

1. sMAPE單一channel的actual與predicted皆0時該term為0
2. WAPE denominator為0時回0
3. NRMSE denominator為0時回0
4. 不使用epsilon將zero denominator轉成巨大數值
5. Floating comparison tolerance只存在tests，不進production formula

### 12.3 Traffic Scales

```text
actual_traffic_scale    = mean(abs(actual_samples))
predicted_traffic_scale = mean(abs(predicted_samples))
```

不得改Go `minDecisionTrafficScale`、`maxActualTrafficScale`、`minPredictedTrafficScale`或
`predictionOvershootRatio`來配合current sum-based units。

### 12.4 Per-scope Report

每個達到`min_matched_predictions`的scope建立一份report：

1. `monitoring_context`使用prediction保存的scope snapshot
2. `window_start`為本輪matched predictions最早target slot
3. `window_end`為本輪matched predictions最晚target slot
4. `retrain_context.subscription_ids`取本輪matched predictions union並排序
5. `retrain_context.observation_source_ids`取本輪matched prediction snapshots union並排序
6. `inference_count`為該scope、該generation、上次report後接受的prediction record數
7. Report後只重設該scope inference count，不清除其他scope
8. Pending predictions保留自身context，不依賴report-level global sets
9. Sequence在model generation與scope內單調遞增

`deviation`維持sMAPE相容欄位。PyAnLF固定計算並回報`sMAPE`、`MAE`、`MSE`、`WAPE`與`NRMSE`；
不加入`metrics_to_record` filtering。Go `primaryMetric`仍負責選擇MTLF decision使用的指標。

---

## 13. Delivery Semantics And Observability

### 13.1 Retry Classification

沿用現有contract：

1. HTTP `204`成功
2. Timeout、transport error與`5xx`有限retry
3. `4xx`記錄rejection，不重試
4. Shutdown中止尚未完成的retry
5. 所有attempt重用同一個serialized report與`report_id`

### 13.2 Accepted Finite-drop Policy

R2不建立outbox。Retry exhausted後：

1. 該report視為drop
2. 不把matched samples放回pending
3. 不回滾report sequence或inference count
4. 記error log與`delivery_failure_total`
5. 下一個periodic round可建立新的report與report ID

這是已核准、弱於historical in-process callback的delivery guarantee。Tests必須固定failure不會破壞
下一round，而不是把delivery失敗偽裝成成功。

### 13.3 Minimum Counters

PyAnLF至少提供可測試的in-memory counters與structured logs：

1. `confidence_zero_total`
2. `prediction_expired_total`
3. `accuracy_report_delivery_failure_total`
4. `accuracy_scope_unresolved_total`

本stage不導入新的metrics server。若目前沒有exporter，counter先由registry-owned stats提供，並在
log中包含model identity、generation、scope或report ID。

---

## 14. Config Ownership And Validation

Target sample config：

```yaml
analytics:
  ue_communication:
    sampling_interval: 30
    input_window: 30
    output_window: 1
    ring_buffer_size: 50
    batch_dedup_capacity: 2048

accuracy_monitor:
  enabled: true
  check_interval: 90
  min_matched_predictions: 2
  report_delivery:
    callback_uri: "http://127.0.0.1:9091/model-accuracy-reports"
    request_timeout: 5
    max_retries: 3
    retry_interval: 1
```

Changes：

1. 新增`check_interval`
2. 將PyAnLF `min_samples`改名為`min_matched_predictions`，不保留舊名稱過渡
3. 不搬移未實際生效的`metricsToRecord`；固定回報全部五種metrics
4. 移除`matching_tolerance`
5. 移除`prediction_retention`
6. 以`ring_buffer_size`取代`source_retention`
7. Go只保留真正的MTLF decision policy config

Defaults與validation：

1. `check_interval`省略時使用historical 60秒；sample/testbed明確設90秒
2. `min_matched_predictions`必須大於0
3. Sampling interval與output window必須大於0
4. `ring_buffer_size >= input_window`
5. `min_samples`、`source_retention`、`warmup_duration`、`matching_tolerance`、
   `prediction_retention`與`metrics_to_record`不保留typed ownership、dual-read或sample config；依
   2026-07-13追加決策，normal YAML unknown-field behavior可靜默忽略舊值

### 14.1 Config Provenance Audit

2026-07-13對照`NWDAF@0db9584`、current NWDAF與current PyAnLF後，R2相關欄位分類如下：

| Parameter | Provenance / historical behavior | R2 disposition |
| --- | --- | --- |
| `sampling_interval` | historical Go active domain config | 搬到PyAnLF並保留 |
| `input_window` | historical Go active domain config | 搬到PyAnLF並保留 |
| `output_window` | historical Go active domain config | 搬到PyAnLF並保留 |
| `ring_buffer_size` | historical Go active；per correlation/IP cap，default 50 | 搬到PyAnLF並恢復 |
| `lookbackBuffer` | historical Go config存在，但latest in-memory inference path未使用 | 不搬移，不重建dead setting |
| `source_retention` | Phase 3新增time-based policy | 移除，由ring buffer取代 |
| `batch_dedup_capacity` | Go-to-Py HTTP retry後新增idempotency state bound | 保留；屬transport boundary必要設定 |
| analytics `report_delivery` | PyAnLF成為report owner後新增HTTP transport policy | 保留 |
| accuracy `enabled` | historical Go active | 搬到PyAnLF measurement side並保留；Go另有MTLF policy enable |
| `check_interval` | historical Go active，default 60/testbed 90 | 搬到PyAnLF並恢復 |
| accuracy `minSamples` | historical Go monitor與MTLF共用；current PyAnLF移植為`min_samples` | PyAnLF改名為`min_matched_predictions`並作為唯一measurement/sample-count gate；移除Go同名config與重複admission gate |
| `warmup_duration` | historical Go active；current confidence gate已取代 | 依confirmed decision移除 |
| `prediction_retention` | Phase 4 Python migration新增 | 移除，由derived miss count取代 |
| `matching_tolerance` | Phase 4 Python migration新增且造成adjacent-slot bug | 移除，使用exact slot equality |
| `metrics_to_record` | historical Go config存在，但production monitor始終建立全部五種metrics | 不搬移、不啟用filtering；PyAnLF固定回報全部五種metrics |
| accuracy `report_delivery` | accuracy跨process callback後新增transport policy | 保留 |
| Go `groundTruthRetention.ringBufferSize` | Phase 3/4過渡buffer；current Go只有write path，已無reader | R2完成Py ring後移除config、`RawUpfData` state與append/cap path |

`server`、`log`與model bundle/identity欄位不是R2 accuracy policy。PyAnLF成為獨立process後需要自己的
server/log config；model identity、default artifact與architecture config由R3 provenance review處理，
不在R2順便改寫。

### 14.2 Confirmed Config Decisions

| Item | Current facts | Confirmed decision |
| --- | --- | --- |
| `metrics_to_record` | Historical config有欄位，但production behavior永遠產生全部五種metrics | 不首次啟用新filter；固定產生與傳送sMAPE、MAE、MSE、WAPE、NRMSE |
| Go `groundTruthRetention.ringBufferSize` | Py ring恢復後，Go `RawUpfData`只剩append/cap且沒有production reader | 移除Go buffer/config；保留observation轉送、MongoDB與ADRF既有行為 |
| `min_samples` / Go `minSamples` | Py用於measurement report gate；Go仍用同名欄位作degradation reference admission | Py改名為`min_matched_predictions`並成為唯一sample-count gate；Go信任已送達report並移除重複threshold |

---

## 15. Planned Code Changes

### 15.1 PyAnLF

| File | Planned change |
| --- | --- |
| `core/analytics_runtime.py` | 回傳external report與per-step internal prediction outcomes |
| `core/observation_store.py` | 以per-source/IP fixed-size rings取代flat time-retained list |
| `core/runtime_manager.py` | 移除ingest-driven processing；記錄每個accepted step；提供store lookup給monitor |
| `core/accuracy/identity.py` | UE Communication selection、target normalization、binding fallback、stable scope ID |
| `core/accuracy/metrics.py` | UL/DL-aware historical pure formulas與traffic scales |
| `core/accuracy/monitor.py` | Prediction IDs、per-model worker、readiness、snapshot/evaluate/commit與report state |
| `core/accuracy/reporting.py` | Stable retry payload、classification與failure result/log |
| `models.py` | 保持wire fields，補必要validation；不加入transport-only migration DTO |
| `config/config.yaml` | 恢復ring buffer config並移除time/tolerance/prediction retention |
| `tests/fixtures/behavioral_parity/` | R0 Accuracy golden fixtures與provenance |
| `tests/test_accuracy_parity.py` | Historical/approved behavior compatibility tests |
| Existing tests | 改為frozen clock與manual check seam，補thread/config regression |

### 15.2 NWDAF

R2需清除PyAnLF接管後已無production reader或重複生效的Go過渡state/config：

| File | Planned change |
| --- | --- |
| `config/nwdafcfg.yaml` | 移除`groundTruthRetention`與`accuracyPolicy.minSamples` |
| `pkg/factory/config.go` | 移除`GroundTruthRetentionConfig`、`GetRingBufferSize()`與accuracy中已不屬Go的check/min/warmup/metric-list欄位及helper |
| `internal/context/traffic_data.go` | 移除`RawUpfData` |
| `internal/sbi/processor/upf_notify.go` | 移除`RawUpfData` append/cap；保留SourceObservation建立、backend forwarding、MongoDB與ADRF既有路徑 |
| `internal/mtlf/trigger.go` | 移除degradation-reference的重複sample-count admission gate，讓PyAnLF成為唯一measurement gate |
| Existing tests | 更新factory、UPF notify與MTLF policy tests，證明移除過渡state不改其他資料路徑 |

Targeted regression仍需確認：

1. Go callback接受restored metric values與traffic-scale units
2. `sample_count`仍是pair count，且PyAnLF不會送出未達`min_matched_predictions`的report
3. `inference_count`與per-scope report不被重複聚合
4. Duplicate `report_id`仍不重複更新MTLF policy
5. Existing primary metric、baseline、traffic gate與retrain thresholds不因R2改動

如果PyAnLF修復必須變更`ModelAccuracyReport` wire shape，應停止並提出cross-repo contract decision，
不得直接在R2 implementation中同步修改兩端。

---

## 16. Implementation Sequence

### Step 1: Build R0 Accuracy Oracle

1. 從historical Go code/tests手工建立accuracy fixtures
2. 每個fixture記錄commit、source path、test name與classification
3. 實作fixture loader與pure expected assertions
4. 在current PyAnLF執行並保存initial failure evidence
5. 不commit red tests；依Decision 12與fix一起保持正式commit green

### Step 2: Restore Pure Domain Functions

1. 修正UL/DL metrics與traffic scales
2. 修正Go-compatible target slot helper reuse
3. 修正scope canonicalization與binding fallback
4. 恢復per-source/IP fixed-size observation rings
5. 先讓metric、matching、scope與ring fixtures通過

### Step 3: Restore Prediction Shape

1. 增加internal analytics generation result
2. 每output step建立獨立prediction
3. 保存sampling interval、scope與source snapshots
4. 增加confidence-zero log/counter
5. 驗證external analytics report shape沒有改變

### Step 4: Restore Periodic Monitor

1. 移除`ingest()`中的immediate accuracy processing
2. 建立per-model worker與manual check seam
3. 移除time-based warmup，以confidence readiness與generation reset取代
4. 實作snapshot/evaluate/commit state transition
5. 實作derived miss count與round-local samples
6. 驗證concurrent add/replace/detach不破壞snapshot isolation

### Step 5: Restore Report Semantics

1. 從本輪matched predictions建立per-scope report
2. 恢復window、context、inference count與traffic scales
3. 補stable delivery ID與failure counters
4. 驗證failed delivery後下一round正常

### Step 6: Cut Over Config

1. 新增check interval；固定回報全部五種metrics，不新增metric filtering config
2. 移除tolerance與prediction retention
3. 以`ring_buffer_size`取代`source_retention`
4. 補defaults、`ring_buffer_size >= input_window` validation與overflow tests
5. 移除Go `groundTruthRetention`、`RawUpfData`與已不生效的accuracy measurement config/helper
6. 移除Go `minSamples`與degradation-reference重複admission gate
7. 更新兩端sample config，不保留雙讀過渡

### Step 7: Cross-repository Regression

1. 在NWDAF加入或補足MTLF policy-input與UPF forwarding regression tests
2. 確認Go移除buffer後仍建立並轉送相同SourceObservation，且MongoDB/ADRF行為不變
3. 執行PyAnLF focused/full tests
4. 執行NWDAF targeted/full tests、build與lint
5. 執行targeted race/concurrency tests
6. 更新audit findings、parent plan與implementation progress

每一步可形成獨立green commit checkpoint；不得以current tests自洽取代R0 fixture parity。

---

## 17. Test Plan

### 17.1 R0 Accuracy Fixtures

最低fixture inventory：

1. `accuracy-adjacent-slot-rejected`
2. `accuracy-half-slot-go-rounding`
3. `accuracy-staggered-multi-source-periodic-best-effort`
4. `accuracy-source-ip-last-wins`
5. `accuracy-ul-dl-opposite-direction-error`
6. `accuracy-historical-all-metrics`
7. `accuracy-zero-actual-traffic`
8. `accuracy-historical-traffic-scales`
9. `accuracy-late-truth-before-miss-discard`
10. `accuracy-unmatched-miss-discard`
11. `accuracy-confidence-readiness-and-generation-reset`
12. `accuracy-confidence-zero-excluded`
13. `accuracy-per-round-min-matched-predictions`
14. `accuracy-canonical-target-order-and-dedup`
15. `accuracy-mixed-event-selects-ue-communication`
16. `accuracy-binding-scope-fallback`
17. `accuracy-multi-step-records`
18. `accuracy-failed-callback-next-round`
19. `observation-per-session-ring-buffer-cap`

### 17.2 Pure Domain Tests

1. All metric formulas逐值比較historical expected
2. Zero denominator behavior
3. Pair count與channel count distinction
4. Slot-key exact/adjacent/positive-half/negative-half
5. Same source/IP last-wins與cross-source sum
6. Scope target normalization與filter canonicalization
7. Derived max miss count多組SI/check interval組合
8. Ring overflow只移除同session最舊record
9. Report固定包含全部五種metrics，不存在config-driven filtering

### 17.3 Monitor Component Tests

1. One worker per model identity
2. First attach/start與last detach/stop
3. Two scopes同model同round隔離
4. Prediction在snapshot後加入不被移除或missed
5. Two matched、one pending後context一致
6. Matched samples未達min不跨round累積
7. Late truth在後續round成功匹配
8. Miss threshold到達才discard
9. Confidence-zero不建立pending prediction
10. Generation replacement清除舊state並使舊round commit失效
11. Subscription detach只移除該subscription records
12. Shutdown停止workers

### 17.4 Runtime Integration Tests

1. Observation ingest不直接觸發accuracy report
2. Periodic/manual tick才evaluate
3. Multi-step predictor建立多筆不同target records
4. Confidence-zero不建立record
5. External analytics report維持目前shape
6. Binding update後新prediction使用新snapshot，舊prediction維持舊snapshot

### 17.5 Delivery Tests

1. Retry重用stable report ID與payload
2. `204`成功
3. `5xx`/transport failure有限retry
4. `4xx`不重試
5. Exhaustion增加counter並記錄error
6. Delivery failure後下一round建立新report

### 17.6 NWDAF Regression Tests

1. Historical fixture metrics與traffic scales進入MTLF時數值不變
2. UL/DL opposite-direction error不被視為零deviation
3. Duplicate report ID不重複更新policy window
4. Existing MTLF threshold config完全不修改
5. 移除Go `RawUpfData`後，SourceObservation forwarding、MongoDB與ADRF路徑維持原行為
6. Go config不再接受或宣稱擁有ground-truth ring、check/min/warmup/metric-list measurement settings
7. MTLF不再用第二個`minSamples`拒絕PyAnLF已建立的accuracy report

---

## 18. Verification Commands

實作時依實際test names調整focused commands，但最低執行：

```bash
cd PyAnLF
uv run pytest -q tests/test_accuracy_parity.py
uv run pytest -q tests/test_accuracy_monitor.py tests/test_accuracy_reporting.py tests/test_runtime_manager.py
uv run pytest -q
```

```bash
cd NWDAF
go test ./internal/anlf/... ./internal/mtlf/...
go test -race ./internal/anlf/... ./internal/mtlf/...
make test
make build
make lint
```

依workspace policy，test/script execution使用elevated permission。若環境無法執行full NWDAF suite，
不得以PyAnLF tests取代；應記錄未執行項目、原因與remaining risk。

---

## 19. Risks And Controls

### 19.1 Scheduler Concurrency

Risk：periodic worker與prediction add、detach、generation replacement同時發生時誤刪新record。

Control：stable record ID、snapshot/evaluate/commit三段式、generation/version guard、sender outside lock、
targeted race tests。

### 19.2 Partial Multi-source Truth

Risk：periodic best-effort仍不能保證所有source在同一round前到齊。

Control：這是已核准的historical-style tradeoff；保留source snapshots與fixture。Expected-source
completeness/timeout另列future work，不在R2偷渡。

### 19.3 Finite-drop Delivery

Risk：cross-process callback retry耗盡後report永久遺失。

Control：stable report ID、有限retry、failure counter/error log、下一round independence。Outbox不在
本stage範圍。

### 19.4 Ring Capacity Misconfiguration

Risk：`ring_buffer_size < input_window`使模型永遠只能得到不足的history。

Control：config validation拒絕該analytics runtime設定，並以per-session overflow/boundary tests固定行為。

### 19.5 Scope Drift

Risk：subscription update或binding reorder改變舊prediction identity。

Control：canonical context與source IDs在prediction creation時snapshot。

### 19.6 Threshold Compatibility

Risk：MTLF thresholds已在實驗中依current錯誤sum units重新校正。

Control：R2不修改threshold；若確認已使用新單位，依parent blocker rule停止並提出migration decision。

---

## 20. Blocker And Replan Conditions

遇到以下情況停止實作並提出decision：

1. Historical Go production code與tests對同一fixture有不可解衝突
2. Current predictor不能提供per-step values而必須改external analytics contract
3. Scope fallback無法從subscription或bindings取得且需要新增Go runtime contract
4. Exact-slot matching必須重新依賴Go MongoDB才能成立
5. Per-session ring無法在不改變R1 source-aware snapshot contract下實作
6. MTLF thresholds已依sum-based traffic scale投入使用
7. Per-model worker無法在current runtime lock hierarchy安全停止或replace generation
8. Required tests因dependency/environment缺失無法建立
9. 修復需要改變R3/R4責任、delivery guarantee或approved decisions

Blocker report需說明原假設、證據、選項、建議與是否要先修改本計畫。

---

## 21. Completion Criteria

R2只有在以下全部成立時才可標記completed：

1. R0 Accuracy fixtures具有historical/approved provenance
2. Initial failure evidence已記錄，正式commits保持green
3. BP-07至BP-18與BP-28至BP-31都有對應tests與closure
4. Adjacent slot不進ground truth
5. Staggered multi-source不再由單次ingest立即消耗prediction
6. UL/DL direction error不再互相抵消
7. All metrics、zero denominator與traffic scales恢復expected values
8. Periodic cadence不依observation batch數量改變
9. Confidence-zero不進pending；generation replacement清除舊generation state
10. Confidence-zero exclusion具有log/counter test
11. Pending/new records在round commit後state一致
12. Scope canonicalization與binding fallback tests通過
13. Failed callback後下一round正常，delivery failure可觀察
14. Per-session ring cap與`ring_buffer_size >= input_window` tests通過
15. PyAnLF固定回報全部五種metrics，沒有新增historically inactive filtering behavior
16. Go過渡`RawUpfData`、ground-truth retention config與measurement殘留欄位已移除
17. PyAnLF是唯一sample-count report gate，Go不再重複拒絕report
18. PyAnLF focused與full tests通過
19. NWDAF MTLF policy-input與UPF forwarding regression tests通過
20. NWDAF build、lint、required test/race verification通過或明確記錄environment gap
21. 完整code review沒有未處理P0/P1 finding
22. Parent plan、audit與Phase 4 remediation status已更新

R2完成後可說Phase 3/4主要data semantics已恢復，但整體behavioral parity仍須完成R3與R4。

---

## 22. Progress Record

2026-07-13完成R2 implementation與local verification。

### 22.1 Initial Failure Evidence

新增R0 Accuracy fixture suite後，current PyAnLF在test collection階段因缺少
`calculate_pairs`失敗。這證明當時production code尚未提供UL/DL pair-aware compatibility seam；
後續fixture expected values均由historical Go source/tests人工核對，沒有由修正後PyAnLF output回填。

### 22.2 Implemented Result

PyAnLF：

1. 建立per-model periodic accuracy worker與deterministic manual check seam
2. Prediction改為per-output-step、UL/DL分離、source/scope/generation snapshot
3. Ground truth恢復Go rounding、exact slot equality、source/IP last-wins與cross-session sum
4. Metrics與traffic scale恢復historical all-channel formulas及zero denominator behavior
5. Matched samples改為round-local，pending records使用snapshot/evaluate/commit與derived miss count
6. Confidence-zero取代fixed warmup並加入in-memory counters/logs
7. Scope明確選UE Communication，target ID canonicalization並支援binding fallback
8. Observation store恢復per-source/IP fixed-size ring；移除time-based retention
9. Accuracy sender保留finite retry/drop，report ID/payload穩定並提供failure observability
10. 修正overlapping checks可能重複commit同一prediction的並行問題

NWDAF：

1. 移除`RawUpfData`、`groundTruthRetention`與只寫不讀的Go ring path
2. 移除Go accuracy measurement欄位與`metricsToRecord` helper
3. 移除MTLF對`SampleCount`的第二道baseline admission gate
4. UPF processor tests改為直接驗證送往PyAnLF的`SourceObservation`
5. 保留既有MTLF thresholds、MongoDB/ADRF與single observation-delivery worker

### 22.3 Verification

已通過：

1. Initial focused suite：修正前collection failure已保存於本次實作紀錄
2. `cd PyAnLF && uv run pytest -q`：74 passed
3. `cd NWDAF && make test`：passed；live PyAnLF contract test因未設定`PYANLF_LIVE_ENDPOINT`而skip
4. `go test -race ./internal/anlf/... ./internal/mtlf/... ./internal/sbi/processor`：passed
5. `cd NWDAF && make build`：passed
6. `cd NWDAF && make lint`：0 issues
7. 兩個implementation repo的`git diff --check`：passed

### 22.4 Approved Deviation And Remaining Scope

1. 依2026-07-13追加決策，舊YAML欄位不特別拒絕；欄位已從typed config與sample移除，normal
   YAML unknown-field behavior可靜默忽略舊值。
2. 未執行實際Go/PyAnLF live HTTP E2E；本環境以unit/component/cross-boundary contract tests為主。
3. Single global observation-delivery worker、finite-drop accuracy callback與best-effort multi-source仍是已接受風險。
4. Model identity/generation/provision atomicity仍由R3處理；scheduler completion與runtime cleanup仍由R4處理。
5. Implementation commits已建立：`PyAnLF@82f9941`與`NWDAF@195e130`；本節由後續
   `nwdaf-docs` progress commit保存。
