# AnLF Behavioral Parity Remediation Plan

Date: 2026-07-13

Status: R0 Analytics/Accuracy slices and R1-R3 implemented and locally verified; R4-R5 pending

Parent plan:

- `AnLF Backend Transition Plan.md`

Audit and decision record:

- `Behavioral Parity Audit.md`

Detailed plans:

- `R0 Compatibility Oracle Framework.md`
- `R1 Analytics Shaping Parity.md`
- `R2 Accuracy Measurement Parity.md`
- `R3 Model Identity Generation And Provision Correctness.md`
- `R4 Scheduler And Subscription Lifecycle.md`

Affected implementation repositories:

1. `PyAnLF/`
2. `NWDAF/`

---

## 1. Purpose

Phase 3與Phase 4已完成AnLF responsibility migration，但2026-07-13 audit確認PyAnLF沒有完整
保留搬移前Go analytics shaping、accuracy measurement、model lifecycle與subscription completion
語意。

本計畫的目的，是在不撤銷既有backend ownership與Go package boundary的前提下，恢復已由歷史
Go commits與tests證明的domain behavior，並落實audit中已確認的少數新policy。

完成後應達到：

1. PyAnLF仍擁有model lifecycle、analytics runtime、accuracy monitoring與report scheduling
2. NWDAF Go仍只負責5GC-facing transport、subscription/correlation與cross-domain coordination
3. 模型輸入、prediction target、ground truth、accuracy metrics與MTLF policy input恢復歷史語意
4. model identity/generation不再受artifact URL key污染
5. `maxReportNbr`與`monDur` completion能回到Go subscription lifecycle
6. 相容性由golden fixtures與cross-repo tests證明，不再只以current implementation自洽測試判定

本計畫是修復Phase 3、Phase 4 behavioral parity的獨立工作線，不重開Phase 3.5的Go package
boundary設計，也不將future PyMTLF或transport architecture混入本輪。

---

## 2. Governing Sources And Baselines

### 2.1 Required Local Sources

實作與review依以下順序使用本地來源：

1. `Behavioral Parity Audit.md`
2. 搬移前NWDAF source與tests
3. current `PyAnLF/`與`NWDAF/` implementation
4. `nwdaf-docs/docs/development_policy.md`
5. local 3GPP TS與OpenAPI YAML
6. `free5gc-dev-skill/SKILL.md`與其testing/lifecycle guidance
7. local free5GC reference NFs，只用於Go HTTP edge、lifecycle與dependency injection shape

PyAnLF不是free5GC NF implementation target。Python domain演算法以歷史NWDAF behavior與NWDAF
specific semantics為主要oracle；只有Go callback edge與service wiring需要維持現有free5GC-style
package boundary。

### 2.2 Historical Behavior Baselines

不同修復不得使用單一commit作全部oracle：

| Area | Source baseline | Test oracle |
| --- | --- | --- |
| Analytics shaping | `NWDAF@0db9584:internal/anlf/analytics.go` | `NWDAF@0db9584:internal/anlf/analytics_test.go` |
| Accuracy workflow | `NWDAF@a7d0693:internal/anlf/accuracy/` | `NWDAF@a7d0693:internal/anlf/accuracy/*_test.go` |
| Accuracy state | `NWDAF@a7d0693:internal/context/accuracy_store.go` | corresponding context tests |
| Scheduler completion before lifecycle migration | `NWDAF@0db9584` | notifier tests |
| Scheduler completion after lifecycle migration | `NWDAF@62e2f9f` | notifier/processor tests |
| Current Go package boundary | `NWDAF@64fdf0a` | current Go tests |
| Current PyAnLF ownership | `PyAnLF@60994c3` | current Python tests |

Historical production code不得複製回Go形成第二條runtime path。需要的演算法只移植到PyAnLF，
historical Go只作fixture provenance與expected behavior oracle。

### 2.3 Confirmed Historical Invariants

以下項目視為本輪不可任意改動的invariants：

1. source-aware session identity
2. per-session anchor-relative rounding
3. same-session、same-slot last-wins dedup
4. inference-time snapped global grid
5. cross-source/session summation
6. contributing snapped centers的mean output timestamp
7. continuous slot range與internal zero-padding
8. prediction target slot與ground-truth exact slot-key equality
9. UL、DL作為分離samples
10. historical MAE、MSE、sMAPE、WAPE、NRMSE formulas
11. actual/predicted traffic scale使用mean absolute samples
12. per-model periodic accuracy check round
13. confidence-zero作為accuracy data readiness gate
14. canonical monitoring scope
15. model identity、generation與artifact reference分離
16. per-source/IP fixed-size observation ring buffer

---

## 3. Approved Decisions

本計畫直接採用audit第15節已確認的decisions：

| Decision | Approved behavior |
| --- | --- |
| Multi-source ground truth | historical periodic best-effort；每輪使用當下已到資料 |
| Accuracy callback delivery | finite retry後drop；不建立outbox |
| Subscription completion cleanup | release PyAnLF runtime並將Go subscription設inactive |
| Same-URL model update | identity/generation改變時必須reload，URL可相同 |
| Artifact cache isolation | identified cache key包含model identity、generation與full artifact reference |
| Confidence-zero accuracy | 不進baseline，但保留log與counter |
| Accuracy sample accumulation | per-check-round，不跨round累積 |
| Observation delivery concurrency | 保留single global worker |
| Provision event idempotency | 只保證Go到PyAnLF internal forwarding retry |
| Completion callback delivery | process-local tombstone，retry至Go確認或shutdown |
| Post-completion observation forwarding | source沒有active subscription時停止backend enqueue |
| Ground-truth timing | exact slot equality；以derived miss count保留`2*SI`等待直覺 |
| R0 commit strategy | 先在本地證明failure，tests與對應fix一起commit |
| R1 observation snapshot cutover | Analytics與Accuracy共用source-aware snapshot；Accuracy只作data-shape adaptation |
| Observation buffer bound | 恢復historical per-source/IP `ringBufferSize=50`；移除time-based `source_retention` |
| Unresolved accuracy scope | concrete target優先、bindings fallback；仍無法解析時只skip accuracy並記錄observability |
| Accuracy metric output | 固定回報全部五種historical metrics；不啟用`metrics_to_record` filtering |
| Accuracy sample-count ownership | PyAnLF以`min_matched_predictions`作唯一report gate；Go不再重複admission |
| Go transitional ground-truth buffer | PyAnLF ring恢復後移除Go `RawUpfData`與對應config/write path |

另外保留Phase 3、Phase 4已核准的語意：

1. `immRep=false`時先等待一個`repPeriod`，不恢復舊版無條件立即report的bug
2. fixed-duration warmup由confidence readiness取代；generation replacement清除舊accuracy state
3. PyAnLF是`UE_COMMUNICATION`必要runtime，不加入Go fallback scheduler
4. 未支援的`Nnwdaf_AnalyticsInfo_Request`不新增speculative request/response API

---

## 4. Scope And Non-goals

### 4.1 In Scope

1. Historical golden fixtures與compatibility tests
2. PyAnLF observation shaping修復
3. PyAnLF prediction/ground-truth/metrics/cadence修復
4. Scope canonicalization與prediction bookkeeping修復
5. PyAnLF model registry、cache key與generation replacement修復
6. Go/PyAnLF provision `event_id` contract
7. PyAnLF scheduler completion與Go inactive callback
8. Transient scheduler error recovery
9. Accuracy finite-drop、completion retry與provision dedup observability/tests
10. Cross-repo HTTP contract與MTLF input compatibility verification
11. Go端已無reader的ground-truth buffer與重複measurement config/gate清理

### 4.2 Explicit Non-goals

1. Per-source observation delivery workers
2. Durable/replayable observation transport
3. Accuracy report process-local或durable outbox
4. External MTLF callback end-to-end deduplication
5. Provision event registry跨PyAnLF restart持久化
6. Completion tombstone跨PyAnLF restart持久化
7. `maxReportNbr`/`monDur`後清理SMF/UPF collection references
8. 重新校正MTLF thresholds或改寫MTLF policy
9. 完整`Nnwdaf_MLModelMonitor` service lifecycle
10. PyMTLF transition
11. Non-`UE_COMMUNICATION` analytics/accuracy migration
12. Artifact signing或完整供應鏈驗證

如果實作發現必須改動上述non-goals才能安全完成，應依development policy停止並重新提出
blocker，不得以隱含擴張scope處理。

---

## 5. Repository And Package Boundary

### 5.1 PyAnLF Ownership

主要修改位置：

| Package/file | Planned responsibility |
| --- | --- |
| `core/observation_store.py` | 保存source identity並提供deterministic source-aware snapshots |
| `core/analytics_runtime.py` | historical alignment、aggregation、padding與prediction targets |
| `core/accuracy/identity.py` | canonical UE Communication scope |
| `core/accuracy/metrics.py` | historical UL/DL-aware metrics |
| `core/accuracy/monitor.py` | periodic check、matching、readiness與bookkeeping |
| `core/accuracy/reporting.py` | finite retry/drop與failure counters |
| `core/model_manager.py` | generation-aware artifact cache/load |
| `core/runtime_manager.py` | model catalog、atomic replacement、provision dedup、completion orchestration |
| `core/reporting.py` | scheduler completion與transient error behavior |
| `models.py` | internal provision/completion contract models |
| `sbi/routers/analytics.py` | internal backend endpoints，不承載domain logic |
| `config/config.yaml` | analytics/accuracy/lifecycle settings ownership |

可以新增責任明確的helper modules，例如：

```text
src/py_anlf/core/
├── alignment.py
├── completion.py
└── provision.py
```

只有在`runtime_manager.py`無法保持可讀性時才拆分；不得為了外型加入沒有獨立state/lifecycle的
空package。

### 5.2 NWDAF Ownership

主要修改位置：

| Package/file | Planned responsibility |
| --- | --- |
| `internal/anlf/contract/model_provision.go` | provision `event_id` |
| `internal/anlf/contract/runtime.go`或新`runtime_completion.go` | completion callback payload |
| `internal/anlf/api_runtime_completion.go` | route declaration、JSON validation、status mapping |
| `internal/anlf/processor/` | completion use-case entry與outcome mapping |
| `internal/anlf/coordinator/` | idempotent subscription inactive transition |
| `internal/anlf/coordinator/model_provision.go` | stable event ID creation/propagation |
| `internal/anlf/coordinator/subscription_runtime.go` | completion callback URI construction |
| `internal/mtlf/training.go` | Daisy task-derived provision event ID |
| `internal/sbi/processor/eventssubscription.go` | 只在既有ownership需要時提供narrow inactive operation |
| `internal/sbi/processor/upf_notify.go` | completed subscription的backend forwarding gate |
| `pkg/factory/`、`config/` | 只有新增callback URI或timeout確有config需求時才調整 |

Go端維持Phase 3.5 shape：

1. `internal/anlf/server.go`組合routes
2. route declaration保留在對應`api_*.go`，不新增`routes.go`
3. API只parse/validate/map HTTP
4. processor執行procedure entry
5. coordinator處理cross-domain state transition
6. 不把PyAnLF domain algorithm搬回Go

---

## 6. Execution Overview

本輪依R0至R5順序執行：

| Stage | Main outcome | Repositories |
| --- | --- | --- |
| R0 | Historical oracle與failing compatibility evidence | PyAnLF, docs |
| R1 | Analytics shaping parity | PyAnLF |
| R2 | Accuracy measurement parity與Go過渡state清理 | PyAnLF, NWDAF |
| R3 | Model identity/generation/provision correctness | PyAnLF, NWDAF |
| R4 | Scheduler completion與reliability boundary | PyAnLF, NWDAF |
| R5 | Full verification、review與status closure | all three repos |

R1依賴R0；R2依賴R1；R3可在R2後獨立進行；R4依賴current runtime ownership但不依賴R3
algorithm。為降低cross-repo debugging成本，實際落地仍按表中順序，不平行修改R2至R4。

R0是貫穿R1至R4的compatibility framework，不是先完整結束才進入R1的一次性stage。實際上依
`R0 Analytics slice + R1`、`R0 Accuracy slice + R2`、`R0 Model slice + R3`、
`R0 Reporting slice + R4`落地。共通fixture與failure-evidence規則由
`R0 Compatibility Oracle Framework.md`定義；各stage只選取對應slice並補充domain-specific內容。

---

## 7. R0: Compatibility Oracle And Containment

Detailed plan: `R0 Compatibility Oracle Framework.md`

### 7.1 Objective

先建立不依賴current PyAnLF output的expected behavior，證明修復前tests會捕捉到已確認regressions。

### 7.2 Fixture Layout

新增：

```text
PyAnLF/tests/
├── fixtures/
│   └── behavioral_parity/
│       ├── README.md
│       ├── analytics_*.json
│       ├── accuracy_*.json
│       └── lifecycle_*.json
├── test_analytics_parity.py
├── test_accuracy_parity.py
├── test_model_lifecycle_parity.py
└── test_reporting_lifecycle.py
```

每個fixture必須記錄：

1. fixture ID與目的
2. historical source commit/path/test name
3. sampling interval、inference/check time等clock inputs
4. source observations與subscription/runtime inputs
5. expected shaped sequence、target slots或accuracy report
6. 是否屬historical invariant或approved behavior change

時間相關tests必須使用injected/frozen clock，不得直接依賴wall clock或`sleep`取得正確性。

### 7.3 Minimum Fixture Set

Analytics fixtures：

1. clean single-source sequence
2. missing middle slot zero-padding
3. anchor drift
4. same-session duplicate last-wins
5. two sources with same anchor
6. two sources with different anchors
7. same metadata but different source IDs
8. input-window cap
9. mean output timestamp
10. positive/negative half-slot rounding

Accuracy fixtures：

1. adjacent-slot rejection
2. staggered multi-source best-effort check
3. UL/DL opposite-direction error
4. historical metrics and traffic scales
5. all-zero actual traffic
6. late ground truth before miss-count discard
7. unmatched prediction miss/discard
8. confidence readiness與generation state reset
9. confidence-zero exclusion
10. per-round minimum matched predictions
11. canonical SUPI/group ordering and duplicate removal
12. mixed-event subscription where`UE_COMMUNICATION`不是first event
13. per-source/IP ring buffer overflow

Lifecycle fixtures：

1. different identities with same artifact URL
2. same identity/same URL new generation
3. duplicate provision event
4. concurrent duplicate provision event
5. replacement load failure keeps old generation
6. `maxReportNbr` completion
7. `monDur` completion
8. completion callback retry/dedup
9. scheduler transient generation failure

### 7.4 Test-first Rule

R0先在本地證明selected compatibility tests對current implementation失敗，並保存failure inventory。
不得修改fixture expected values讓current code通過。

Confirmed workflow：不建立獨立red-test commit。R0 tests必須先在本地執行並保存initial failure
evidence，再與對應R1至R4 fix一起commit，確保每個正式implementation commit保持green。不得因此
省略test-first failure verification或把current output改成expected。

### 7.5 Completion Criteria

1. 所有P0/P1 findings至少有一個fixture或明確test mapping
2. fixture provenance不引用current PyAnLF output作expected
3. 已確認current failures與audit reproductions一致
4. Current非parity tests仍能獨立執行
5. 文件列出尚未能fixture化的environment-only scenarios

---

## 8. R1: Restore Analytics Shaping

Detailed plan: `R1 Analytics Shaping Parity.md`

### 8.1 Source-aware Observation Snapshot

`ObservationStore`不得在subscription query時丟失source ID。Internal snapshot使用類似：

```text
StoredObservation {
  observation_source_id
  observation
}
```

不要求修改Go到PyAnLF observation HTTP body；route path中的`source_id`已提供identity，PyAnLF在
ingest時將它與observation一起保存。

Snapshot要求：

1. source IDs採排序後遍歷，消除`set` iteration nondeterminism
2. source內保留ingest order供same-slot last-wins
3. batch dedup與retention現有語意保持
4. 同一source由多個subscriptions引用時不複製storage

### 8.2 Session Identity

Session key至少包含：

```text
observation_source_id
ipv4_address
supi
dnn
snssai
```

不同source即使其他metadata相同也必須先作獨立session alignment，之後才在global bucket相加。

### 8.3 Go-compatible Rounding

不得使用Python builtin `round()`作slot identity。新增明確helper實作Go `math.Round`等價的
half-away-from-zero，並以正負`.5` tests固定行為。

### 8.4 Two-step Alignment

每次inference shaping依以下順序：

1. 依session保留資料中的第一筆ingest observation作anchor，不先按timestamp重新排序
2. `round((observed_at-anchor)/sampling_interval)`取得session step
3. 算出snapped session center
4. 同session同center採last-wins
5. 以inference clock向下切齊sampling grid取得`snapped_now`
6. 將session centers映射到相對`snapped_now`的global index
7. 同global index的不同source/session traffic欄位相加
8. non-empty bucket timestamp使用contributing snapped centers平均

Global origin不得改用latest raw observation timestamp。

### 8.5 Continuous Range And Padding

形成global buckets後：

1. 建立所需continuous index range
2. 中間缺少bucket時建立all-zero `TrafficObservation`
3. zero bucket timestamp使用global grid center
4. 最後才套用`input_window`上限
5. 不因zero-padding把range向未來延伸超過inference-time grid

不足`input_window`的前導padding是否由predictor處理，必須維持目前model contract；R1的必要條件是
不能壓縮已存在範圍內的internal gaps。

### 8.6 Prediction Target

Analytics report中的prediction timestamp由最後一個shaped slot加一個sampling interval推導，不
使用latest raw observation。若`output_window > 1`，內部必須保留每個step target slot供R2
bookkeeping；external UE Communication仍可依既有contract聚合report payload。

### 8.7 Tests

至少通過R0 analytics fixtures，並保留：

1. source reference counting
2. batch ID dedup
3. retention cleanup
4. confidence-zero fallback
5. deterministic repeated-run output

### 8.8 Completion Criteria

1. BP-01至BP-06 fixtures全數通過
2. 相同input與clock重複執行輸出完全相同
3. PyAnLF不再使用builtin `round()`建立slot identity
4. Observation source ID不在shaping前遺失
5. 沒有修改Go observation worker concurrency或delivery guarantee

### 8.9 Implementation Result（2026-07-13）

R0 Analytics slice與R1已完成local implementation及verification。PyAnLF已使用source-aware snapshot、
injected clock與獨立pure alignment恢復historical shaping；11組golden fixtures、runtime/store regression
與existing full suite均通過。

Verification：

```text
focused R1 tests: 24 passed
PyAnLF full tests: 41 passed
git diff --check: passed
```

Implementation commit為`PyAnLF@fc59df2`。R1沒有修改NWDAF、external/internal HTTP contract、observation worker、
accuracy domain semantics、model lifecycle或scheduler completion。下一個implementation stage為R0
Accuracy slice與R2。

---

## 9. R2: Restore Accuracy Measurement Semantics

### 9.1 Prediction Record

Prediction state至少保存：

```text
Prediction {
  subscription_id
  monitoring_context_snapshot
  model_identity
  generation
  generated_at
  target_slot_time
  predicted_ul_volume
  predicted_dl_volume
  observation_source_ids
  miss_count
}
```

每個model output step建立獨立record，不得將多步prediction壓成第一個target的一筆total traffic。

### 9.2 Confidence-zero And Readiness

1. confidence-zero不建立baseline prediction record
2. 每次發生時記structured log與in-memory counter
3. 不新增low-confidence accuracy report
4. 不使用固定秒數warmup或`warmup_duration` config
5. confidence-positive prediction立即進入正常periodic lifecycle
6. 新generation清除舊generation pending predictions與inference counters，不延遲新prediction

### 9.3 Periodic Check Loop

Accuracy evaluation回到per-model periodic loop：

1. `check_interval`到達時取得model所有scopes的prediction snapshot
2. 不恢復較早期`ConsumeMaturePredictions(2*SI)`的一次性maturity gate
3. 同一round聚合當下已到達的所有sources
4. 不等待expected source completeness
5. 本輪matched samples只供本輪report
6. 未達`min_matched_predictions`不跨round累積matched samples
7. 未matched prediction增加miss count
8. miss count達derived threshold時discard
9. round結束後只移除matched/discarded records，不影響snapshot後新增records

Latest pre-migration baseline使用：

```text
max_miss_count = ceil(2 * sampling_interval / check_interval) + 1
```

`2*sampling_interval`保留「一格等待target slot完成、一格容許UPF notification延遲」的設計
直覺，但轉換成依`check_interval`推導的retry/discard次數，不作matching tolerance或獨立maturity
config。需以sampling/check interval組合tests固定rounding與至少一次retry語意。

`check_interval`預設與testbed config使用90秒；若config省略，可保留historical default 60秒，但
default與sample config都必須有tests。

### 9.4 Ground-truth Matching

1. 使用prediction的`target_slot_time`
2. 每筆observation以Go-compatible rounding取得相對target slot key
3. 只接受slot key等於0的records
4. 每個source/session/slot先last-wins
5. 再跨source/session分別聚合UL與DL
6. 不使用`abs(timestamp-target)<=matching_tolerance`
7. `matching_tolerance`從model與active config直接移除，不保留deprecated/ignored欄位
8. MongoDB可先查`target_slot_time ± sampling_interval`作candidate prefilter，但每筆candidate仍必須
   通過slot key equality

Candidate query總寬度雖為`2*sampling_interval`，它不是matching tolerance。Go-compatible rounding
使slot key 0只接受接近target的半格範圍；相鄰slot即使位於query window仍必須拒絕。

### 9.5 Metric Formulas

每個prediction/actual pair展開為兩個samples：

```text
actual    = [actual_ul, actual_dl, ...]
predicted = [pred_ul, pred_dl, ...]
```

恢復：

1. `MAE = mean(abs(predicted-actual))`
2. `MSE = mean((predicted-actual)^2)`
3. `sMAPE = mean(2*abs(predicted-actual)/(abs(actual)+abs(predicted)))`
4. denominator兩者皆0的sMAPE term為0
5. `WAPE = sum(abs(error))/sum(abs(actual))`
6. WAPE denominator為0時回0
7. `NRMSE = RMSE/mean(abs(actual))`
8. NRMSE denominator為0時回0
9. `actual_traffic_scale = mean(abs(actual))`
10. `predicted_traffic_scale = mean(abs(predicted))`

不得修改Go MTLF thresholds來配合current錯誤scale。

### 9.6 Scope Canonicalization

1. 明確尋找`UE_COMMUNICATION` event subscription，不固定取第一個event
2. SUPI與group list作trim、sort、deduplicate
3. target/filter map使用stable canonical serialization
4. 空target與`AnyUe`維持historical unresolved semantics
5. 可取得tracked resources時保留historical fallback
6. prediction record保存scope snapshot，後續subscription update不回寫舊record identity

### 9.7 Report State And Delivery

1. report context由本輪matched predictions重建，不依賴可能已清空的global sets
2. inference count定義與historical per-scope round一致
3. 未matched pending records不能在report後失去subscription/source context
4. accuracy sender維持finite retry後drop
5. retry使用同一stable report ID
6. exhausted retry記error log與delivery failure counter
7. delivery failure後下一個round仍可正常建立獨立report
8. 不建立pending outbox，不等待204才保留本輪samples

### 9.8 Config Ownership

PyAnLF `accuracy_monitor`至少明確擁有：

```yaml
accuracy_monitor:
  enabled: true
  check_interval: 90
  min_matched_predictions: 2
```

`matching_tolerance`與`prediction_retention`從active accuracy config移除。Prediction lifecycle由
derived `max_miss_count`負責。`analytics.ue_communication.source_retention`移除，observation storage
恢復historical per-source/IP `ring_buffer_size` count bound；它與prediction matching/expiry policy分離。

PyAnLF固定計算並回報sMAPE、MAE、MSE、WAPE、NRMSE，不新增historical config雖存在但production
未生效的metric filtering。Go只保留MTLF decision policy，例如primary metric、baseline buffers、
z-score、traffic gates與retrain thresholds；`primaryMetric`決定使用哪個已回報metric。Measurement config
不得同時在兩端各自生效。

R2同時清除Go端已失去reader或重複生效的過渡責任：

1. 移除`groundTruthRetention.ringBufferSize`與`RawUpfData` append/cap state
2. 保留SourceObservation轉送、MongoDB與ADRF資料路徑
3. 移除Go accuracy的check/min/warmup/metric-list殘留config與helper
4. 由PyAnLF以`min_matched_predictions`作唯一report gate，移除Go MTLF的重複sample-count admission gate

### 9.9 Tests

除了R0 accuracy fixtures，至少增加：

1. two scopes同model同round隔離
2. prediction在snapshot後加入不被錯誤移除
3. two matched、one pending後report context一致
4. failed accuracy callback後下一round可繼續
5. generation replacement清除舊state且新generation不受time-based warmup阻擋
6. confidence-zero counter/log path
7. Go MTLF收到的metrics與traffic scales等於historical fixture
8. 多組sampling/check interval的derived max miss count
9. Mongo candidate window包含相鄰slot但slot equality仍拒絕
10. 固定回報全部五種metrics，不因config缺少metric list而改變
11. Go移除`RawUpfData`後observation forwarding、MongoDB與ADRF行為不變
12. PyAnLF未達`min_matched_predictions`不送report；Go不再以第二個threshold重複拒絕

### 9.10 Completion Criteria

1. BP-07至BP-18與BP-28至BP-31對應tests通過
2. UL/DL direction error不再互相抵消
3. adjacent slot不會進入ground truth
4. MTLF policy input單位恢復且不修改threshold config
5. periodic cadence不依observation ingest數量改變
6. confidence-zero與finite-drop是唯一明確核准的相關差異
7. Go過渡ground-truth buffer與重複measurement gate已移除

---

## 10. R3: Correct Model Identity, Generation And Provision

Detailed implementation plan：

- `R3 Model Identity Generation And Provision Correctness.md`

### 10.1 Target Model State

PyAnLF分離：

```text
ModelIdentityKey = (provider_id, model_unique_id)
ModelRuntimeKey  = (provider_id, model_unique_id, generation)
ArtifactLocator  = model_reference
```

建議state：

1. `current_generation[ModelIdentityKey]`
2. `shared_models[ModelRuntimeKey]`
3. runtime保存`ModelRuntimeKey`與artifact reference
4. artifact reference只作取得位置，不作identity key

### 10.2 Candidate Prepare And Commit

Provision event處理分三段：

1. Resolve affected runtimes與previous generation
2. Prepare/load candidate，不修改active runtimes
3. Commit所有仍符合expected state的runtimes，再release old references

要求：

1. candidate load失敗時active runtimes完全不變
2. same URL new generation仍建立新的candidate/load attempt
3. commit前驗證affected runtime revision/model identity沒有stale
4. externally observable runtime state不得出現一半old、一半new的中間狀態
5. commit後才更新accuracy generation與discard old pending predictions
6. old model只在沒有runtime reference後unload

實作可使用global state lock完成短暫metadata commit；model download/load不得在global lock內執行。
若需要多把subscription locks，必須固定排序並加入concurrency tests。

### 10.3 Artifact Cache

Cache key至少包含：

```text
sha256(canonical model identity + generation + full artifact reference)
```

不同identity、不同generation，以及不同host但相同URL尾段均不得碰撞。Hash input使用無歧義canonical
serialization。若未來contract提供digest，再把digest納入validation；本輪不要求新增外部digest來源。

### 10.4 Provision Event Contract

Go與Python internal contract新增required `event_id`：

```text
ModelProvisionEvent {
  event_id
  source
  model_identity
  model_update_ind
  artifact
  analytics_event
  notification_correlation
  training_task_id
}
```

ID規則：

1. Daisy retrain completion使用`daisy:<training_task_id>`或等價stable derivation
2. External MTLF callback在Go normalization時產生一次UUID
3. 同一Go forwarding operation的retry必須重用ID
4. External MTLF重新送來的新HTTP callback不保證取得相同ID
5. 不使用canonical content fingerprint，避免same-URL new generation被誤判duplicate

### 10.5 Process-local Idempotency

PyAnLF維護bounded processed-event registry：

1. key為`event_id`
2. value保存成功response的status、affected count與generation
3. duplicate成功event回傳原response，不再次load/commit
4. failed event不寫入completed registry
5. concurrent duplicate只能有一個執行者，其餘等待或取得相同結果
6. registry只保證process lifetime，不宣稱restart durability

Capacity需有bounded default與unit tests；可沿用現有dedup capacity pattern，不新增persistent store。

### 10.6 Tests

1. different identity/same URL不共享identity/generation
2. same identity/same URL new event產生new generation與load
3. duplicate event回同一response且只load一次
4. concurrent duplicate只commit一次
5. same content/different event ID可形成next generation
6. candidate load failure保留全部old runtimes
7. affected runtime concurrent update造成stale時不partial commit
8. cache full-reference collision test
9. Daisy retry使用相同event ID
10. Go/PyAnLF real HTTP duplicate contract

### 10.7 Completion Criteria

1. BP-19至BP-23 tests通過
2. identity/generation不再由URL-only registry決定
3. same-URL update不回假成功
4. duplicate internal event不增加generation
5. replacement failure仍由舊generation提供analytics
6. Go與PyAnLF contract同一cutover完成，沒有長期optional `event_id`

---

## 11. R4: Complete Scheduler And Subscription Lifecycle

Detailed implementation plan:

- `R4 Scheduler And Subscription Lifecycle.md`

### 11.1 Completion Conditions

只有以下正常條件建立completion event：

1. `maxReportNbr` reached
2. `monDur` reached

Explicit DELETE/update replacement使用既有release/cancel path，不應再產生completion callback。
Transient report generation error只log並等待下一個tick，不把subscription完成。Stale或已release
runtime正常停止，不建立completion tombstone。

### 11.2 Internal Completion Contract

Go `anlfServer`新增：

```text
POST /subscriptions/{subscriptionId}/runtime-completions
```

Request：

```text
RuntimeCompletionEvent {
  completion_id
  subscription_id
  runtime_revision
  reason
  completed_at
  last_report_sequence
}
```

`reason`本輪只允許：

1. `MAX_REPORTS_REACHED`
2. `MONITORING_DURATION_EXPIRED`

Apply runtime request新增明確`runtime_completion_callback_uri`，由Go coordinator依anlfServer URI與
subscription ID建立。不得由PyAnLF自行修改analytics report callback URI字串推導endpoint。

### 11.3 PyAnLF Completion Order

正常completion順序：

1. scheduler停止產生新reports
2. runtime manager驗證subscription ID與revision仍active
3. detach model、accuracy monitor與observation bindings
4. 從active runtime map移除runtime
5. 建立stable completion event
6. 寫入process-local tombstone
7. completion worker callback Go
8. 收到`204`後移除tombstone

Completion不得由scheduler thread同步等待一條又會回頭join/delete自己的recursive release path。

### 11.4 Completion Tombstone

每個completed subscription/revision最多一筆：

```text
CompletionTombstone {
  event
  attempt_count
  next_attempt_at
}
```

Guarantee：

1. 網路錯誤、timeout、5xx持續retry
2. 使用獨立`runtime_completion_delivery` config，不共用有限次accuracy/report retry policy
3. 收到Go `204`才移除
4. PyAnLF shutdown停止retry並放棄process-local tombstones
5. 不跨restart恢復
6. tombstone數量由已完成但未ack subscriptions界定，需有count/error logs

```yaml
runtime_completion_delivery:
  request_timeout: 5
  retry_interval: 1
```

此config不提供`max_retries`；所有非`204`結果保留tombstone並依固定bounded interval重試。`400`
視為contract error並持續記error，`409`視為Go revision尚未收斂，兩者都不得busy-loop或靜默drop。

### 11.5 Go Callback Semantics

Go handler沿用API -> processor -> coordinator：

1. malformed/path-body mismatch回`400`
2. current subscription同revision時idempotently設`IsActive=false`並回`204`
3. 已inactive、已刪除或相同completion重送時回`204`
4. completion revision舊於current runtime時視為已消費的stale event，log後回`204`，不得關閉新revision
5. completion revision高於current known revision時回`409`並記error
6. handler不呼叫PyAnLF DELETE，不清理SMF/UPF collection
7. processor/coordinator不依賴broad SBI server

Go可以保存bounded completion ID dedup state，但正確性不能只依賴它；subscription state/revision判斷
本身必須idempotent。

### 11.6 Observation Forwarding After Completion

Completion依決策不會unsubscribe或刪除SMF/UPF collection，但PyAnLF local runtime/binding已釋放。
為避免Go持續把資料送到不存在的backend source並反覆觸發404/retry：

1. UPF notification仍寫入Go traffic storage與ADRF buffer
2. enqueue PyAnLF前檢查該correlation source是否至少有一個active NWDAF subscription
3. 沒有active backend consumer時跳過enqueue並記debug/counter，不刪除SMF subscription
4. shared source仍有其他active subscriptions時維持一次source-scoped delivery
5. gate放在producer/context query，不修改single global worker、queue ordering或retry semantics

此項是cross-process architecture下維持Decision 3 cleanup boundary所需的forwarding control，不是
Decision 7所排除的transport concurrency重構。

### 11.7 Scheduler Error Recovery

1. inference error沿用confidence-zero fallback
2. transient generation/parsing exception記log，sequence/report attempt維持historical
   increment-before-send語意
3. 下一個`repPeriod`繼續嘗試
4. 重複exception不得busy-loop
5. unexpected scheduler thread exit必須有error log
6. 不因一次exception建立normal completion event

### 11.8 Tests

PyAnLF：

1. max reports local release與tombstone
2. monitoring duration local release與tombstone
3. callback timeout/5xx retry至204
4. duplicate completion只保留一筆tombstone
5. shutdown停止retry
6. explicit release不送completion
7. stale revision completion不影響new runtime
8. transient generation error後next tick繼續

NWDAF：

1. route/body validation
2. active same-revision變inactive
3. duplicate callback idempotent
4. missing subscription回204
5. old revision不關閉new subscription
6. future revision回409
7. callback不呼叫backend release或data collection cleanup
8. server route與dependency injection wiring
9. inactive-only source停止backend enqueue但仍保存traffic/ADRF data
10. shared source仍有active subscription時繼續enqueue

Cross-repo：

1. real PyAnLF scheduler completion -> real Go anlfServer -> 204
2. first callback 5xx/timeout、retry後成功
3. Go update與old completion race

### 11.9 Completion Criteria

1. BP-24、BP-25 tests通過
2. max reports/monitoring duration不再留下active PyAnLF runtime
3. Go subscription最終成為inactive
4. callback retry不形成recursive release或deadlock
5. SMF/UPF collection cleanup沒有被偷偷加入
6. single observation worker保持不變
7. completed-only source不持續產生backend 404/retry

---

## 12. R5: Verification And Closure

### 12.1 PyAnLF Verification

最低執行：

```bash
uv run pytest -q
```

若project提供formatter、lint或type checker，依`pyproject.toml`正式entrypoint執行，不臨時發明
替代command。

結果需分開報告：

1. existing unit/API tests
2. analytics golden fixtures
3. accuracy golden fixtures
4. lifecycle/concurrency tests
5. real HTTP contract tests

### 12.2 NWDAF Verification

最低執行：

```bash
go test ./...
make lint
make build
go test -race ./internal/anlf/... ./internal/sbi/processor/... ./internal/mtlf/...
```

若full race受到既有global context test isolation影響，必須列出失敗package、原因與targeted race
結果，不得只省略不報。

### 12.3 Cross-repo Contract Verification

至少驗證：

1. runtime apply包含analytics與completion callback URIs
2. source observations產生historical-compatible shaped input
3. analytics callback仍可由Go轉送external consumer
4. accuracy report欄位與MTLF input units正確
5. duplicate provision event不重複generation
6. same URL new event確實切換generation
7. completion callback retry、dedup與stale revision
8. runtime delete/update不誤發completion

### 12.4 Environment-level Verification

環境允許時執行：

1. SMF/UPF periodic data collection
2. multi-source UE Communication subscription
3. external MTLF model provision
4. Daisy retrain completion
5. ADRF retrain input flow
6. `maxReportNbr`與`monDur` lifecycle

若環境不允許，文件必須保持`environment-level E2E unverified`，不能用unit/live-contract取代。

### 12.5 Final Code Review

Review至少檢查：

1. 每個BP finding是否有code或explicit accepted-risk closure
2. golden expected是否來自historical oracle
3. PyAnLF是否仍是domain owner
4. Go callback是否維持API/processor/coordinator boundary
5. lock order、scheduler shutdown與callback retry是否可能deadlock/race
6. config ownership是否重新重複
7. non-goals是否被意外混入
8. failure logs/counters是否足以觀察accepted drops

---

## 13. Cross-repository Contract Cutover

R3與R4包含成對contract變更，必須各repo獨立commit，但以commit pair作部署單位。

### 13.1 R3 Cutover

1. NWDAF `ModelProvisionEvent`新增required `event_id`
2. PyAnLF model與endpoint同步要求`event_id`
3. 雙方unit tests各自通過
4. Cross-repo test使用新contract通過
5. 不長期保留missing-event-ID fallback

### 13.2 R4 Cutover

1. NWDAF先具備completion callback route與processor wiring
2. Apply runtime contract加入completion callback URI
3. PyAnLF scheduler/runtime manager開始送completion event
4. 雙方unit tests與live retry test通過
5. Deployment文件要求Go callback edge可達後再啟用新PyAnLF build

Repository commit不得混在同一git commit；docs progress commit另在`nwdaf-docs/`完成。

---

## 14. Suggested Commit Checkpoints

Commit subjects在實作時依實際diff調整，不使用僅表示project phase的模糊名稱。建議checkpoint：

| Repository | Checkpoint |
| --- | --- |
| PyAnLF | add historical analytics/accuracy fixtures |
| PyAnLF | restore source-aware analytics shaping |
| PyAnLF | restore periodic accuracy measurement |
| NWDAF | identify model provision events |
| PyAnLF | apply provision events idempotently by identity/generation |
| NWDAF | receive AnLF runtime completion callbacks |
| PyAnLF | report runtime completion with retry tombstones |
| nwdaf-docs | record remediation progress and verification |

每個checkpoint都應包含對應tests。不要在同一commit混入observation worker並行化、MTLF policy
recalibration或其他future refactor。

---

## 15. Risk And Failure Matrix

| Risk/failure | Required behavior | Verification |
| --- | --- | --- |
| Missing source ID | 不得跨source last-wins覆寫 | same-metadata/different-source fixture |
| Internal time gap | zero-pad，不壓縮時間 | missing-middle-slot fixture |
| Half-slot timestamp | Go-compatible round | positive/negative half tests |
| Adjacent ground truth | reject | adjacent-slot fixture |
| Staggered sources | periodic best-effort | frozen-clock multi-source test |
| Accuracy callback down | finite retry後drop並計數 | sender failure test |
| Confidence zero | 不進baseline並計數 | incomplete-window test |
| Same URL new generation | reload candidate | load-call/generation test |
| Duplicate provision retry | 回原response，不重套 | duplicate/concurrent test |
| Candidate load failure | 全部runtime保留舊model | rollback test |
| Completion callback down | tombstone持續retry | fail-then-204 test |
| Old completion after update | 不關閉new revision | stale completion test |
| PyAnLF restart | tombstone/event dedup遺失可接受 | documented limitation |
| Observation queue HOL | 本輪不修 | documented future work |

---

## 16. Phase Completion Rules

### 16.1 R0 Complete

Historical oracle、fixtures與initial failure evidence完整，但implementation仍可不相容。

### 16.2 R1 Complete

Analytics shaping fixtures通過，Phase 3仍不能單獨標記full parity，因R2 accuracy尚未完成。

### 16.3 R2 Complete

Accuracy fixtures與MTLF input compatibility通過，Phase 3/4主要data semantics恢復。

2026-07-14 result：R2已完成implementation、local verification與repository commits：
`PyAnLF@82f9941`、`NWDAF@195e130`。PyAnLF full tests為74 passed；NWDAF full tests、targeted
race、build與lint通過。Live Go/PyAnLF HTTP contract test因未設定`PYANLF_LIVE_ENDPOINT`而skip；
舊YAML欄位依追加決策採normal unknown-field靜默忽略，不保留typed ownership或sample config。

### 16.4 R3 Complete

Identity/generation、same URL reload、atomic replacement與provision idempotency通過。

2026-07-14 result：R3已完成repository-level implementation、local verification與repository commits：
`PyAnLF@01620ce`、`NWDAF@e6d295b`。Identified runtime已改用identity/generation key；anonymous
runtime獨立；same URL new event會prepare新generation；multi-runtime replacement採snapshot revalidation與atomic
metadata commit；internal event ID提供process-local sequential/concurrent retry idempotency；artifact cache依identity、
generation與full reference隔離。

PyAnLF full tests為105 passed；NWDAF focused/full tests、targeted race、build與lint通過，完整review沒有未處理
P0/P1 finding。FastAPI與Go client contract tests已通過；live Go/PyAnLF process test因未設定
`PYANLF_LIVE_ENDPOINT`而skip，因此狀態是repository-level completed，不宣稱environment-level HTTP E2E。

### 16.5 R4 Complete

Scheduler completion、transient error recovery與Go inactive callback通過。

### 16.6 Remediation Complete

只有以下全部滿足才能把parent plan中的Phase 3、Phase 4恢復為無條件`Completed`：

1. R0至R4 completion criteria全部通過
2. PyAnLF full tests通過
3. NWDAF full tests、lint、build與targeted race通過
4. Cross-repo contract tests通過
5. Audit BP-01至BP-30逐項有closure
6. Approved behavior changes與accepted risks有tests/observability
7. 未執行的environment E2E被明確記錄
8. 完整code review沒有未處理P0/P1 finding
9. `Behavioral Parity Audit.md`與本文件更新實際結果/偏差

接近完成、token/時間不足或只通過current unit tests都不是降低completion標準的理由。

---

## 17. Blocker And Replan Conditions

實作遇到以下情況必須停止並回到設計：

1. Historical tests彼此矛盾，無法建立唯一expected behavior
2. Current model input contract無法表示historical zero-padded sequence
3. Exact-slot matching需要Go重新擁有ground truth
4. MTLF threshold實際已依current錯誤scale重新校正並投入使用
5. Standard/external MTLF callback必須支援same-URL duplicate與new generation的可區分identity
6. Atomic model commit無法在current lock hierarchy安全完成
7. Completion callback需要清理SMF/UPF sources才能避免實際錯誤
8. Required tests因dependency/environment缺失無法建立
9. 任一修復必須改變approved decisions或non-goals

Blocker report必須包含原假設、證據、選項、建議與是否需要修訂本計畫。

---

## 18. Documentation And Progress Recording

每個stage完成後更新本文件：

1. implementation commits
2. 實際修改範圍
3. 通過的fixture/test commands
4. 與計畫的偏差
5. 未執行的驗證
6. 新發現但不在scope的future work

同時更新：

1. `Behavioral Parity Audit.md` finding closure
2. `AnLF Backend Transition Plan.md` summary status
3. Phase 3/4文件的follow-up remediation reference

在remediation complete前，狀態固定維持：

```text
Responsibility migration implemented;
behavioral parity validation incomplete.
```
