# AnLF Backend Transition Behavioral Parity Audit

Date: 2026-07-13

Status: R1 analytics, R2 accuracy, and R3 model lifecycle findings remediated locally; R4 findings remain open

Related plans:

1. `AnLF Backend Transition Plan.md`
2. `Phase 2 Model Lifecycle Migration.md`
3. `Phase 3 Analytics Runtime Migration.md`
4. `Phase 3.5 Go Package Boundary Consolidation.md`
5. `Phase 4 Accuracy Workflow Migration.md`
6. `Behavioral Parity Remediation Plan.md`

---

## 1. Purpose

本文件保存 Phase 2 至 Phase 4 遷移完成後的 behavioral parity 稽核結果，避免後續對話壓縮、
重新開啟工作階段或進行修復設計時遺失以下資訊：

1. 搬移前 Go implementation 的實際語意與設計理由
2. 目前 PyAnLF implementation 與原行為的差異
3. 已透過 source、git history、舊測試或最小重現確認的問題
4. 外部掃描報告中正確、部分正確與需要修正的判斷
5. 原計畫要求但實際未完成的 tests 與 acceptance criteria
6. 修復工作開始前必須做出的設計決策

本文件是audit與decision record，不直接作為implementation checklist。第15節confirmed
decisions與本文件證據已由`Behavioral Parity Remediation Plan.md`轉成正式R0至R5實作順序；
其餘audit recommendation不得直接視為額外核准scope。

---

## 2. Audit Scope And Baselines

### 2.1 Repository State

稽核時三個 repository 均為 clean worktree：

1. `NWDAF/`
   - branch: `refactor/free5gc-alignment`
   - HEAD: `64fdf0a refactor(anlf): move accuracy workflow to backend`
2. `PyAnLF/`
   - branch: `master`
   - HEAD: `60994c3 feat(accuracy): own model monitoring workflow`
3. `nwdaf-docs/`
   - branch: `main`

### 2.2 Migration Commit Boundaries

NWDAF commits：

1. pre-transition baseline: `0db9584 refactor(anlf): align backend boundary naming`
2. Phase 2: `62e2f9f refactor(anlf): delegate model lifecycle to backend`
3. Phase 3: `6b3442f refactor(anlf): delegate analytics runtime to backend`
4. Phase 3.5: `a7d0693 refactor(anlf): consolidate Go package boundaries`
5. Phase 4: `64fdf0a refactor(anlf): move accuracy workflow to backend`

PyAnLF commits：

1. pre-transition baseline: `987e98e refactor: rename service to PyAnLF`
2. Phase 2: `7a2ebc4 feat(runtime): manage subscription model lifecycle`
3. Phase 3: `6a0c1f4 feat(runtime): own analytics reporting workflow`
4. Phase 4: `60994c3 feat(accuracy): own model monitoring workflow`

### 2.3 Correct Comparison Points

不同問題必須使用不同 baseline：

1. Phase 3 analytics shaping
   - comparison point: `0db9584:internal/anlf/analytics.go`
   - compatibility oracle: `0db9584:internal/anlf/analytics_test.go`
2. Phase 4 accuracy migration
   - comparison point: `a7d0693:internal/anlf/accuracy/`
   - state oracle: `a7d0693:internal/context/accuracy_store.go`
   - compatibility oracle: `a7d0693:internal/anlf/accuracy/monitor_test.go`
3. Lifecycle completion
   - pre-Phase 2 `0db9584` scheduler completion 只會把 subscription 設為 inactive
   - Phase 2 `62e2f9f` 之後才另外 release AnLF model runtime
   - 舊 completion callback 本身沒有完整 release SMF/UPF collection

### 2.4 Review Method

本次稽核使用：

1. current source inspection
2. `git log --follow` 與 commit message reconstruction
3. historical source inspection via `git show <commit>:<path>`
4. historical test inventory comparison
5. current plan acceptance criteria comparison
6. current repository tests
7. read-only minimum reproductions against current PyAnLF code

本次沒有修改 NWDAF 或 PyAnLF implementation。

---

## 3. Executive Conclusion

Phase 3 與 Phase 4 已完成主要 responsibility boundary migration，但沒有完成原計畫要求的
behavior-preserving migration。

目前狀態應描述為：

~~~text
Responsibility migration implemented;
behavioral parity validation incomplete.
~~~

主要原因不是 Python implementation 無法運作，而是它重新定義了數個原本已在 Go 中經過
專門修正、測試與 testbed calibration 的 domain behavior：

1. traffic time-series alignment 與 gap handling
2. source/session aggregation identity
3. prediction target 與 ground-truth slot matching
4. UL/DL metric semantics
5. traffic scale units
6. accuracy monitor cadence 與data readiness
7. monitoring scope identity
8. model identity、generation 與 artifact sharing
9. reporting completion lifecycle

現有 tests 證明 current implementation 在目前測試範圍內自洽，不證明它與搬移前 Go
implementation 等價。

---

## 4. Historical Go Behavior Was Intentional

以下語意有獨立 commits 與 tests，修復時應視為 domain invariants，除非另外核准 replan。

### 4.1 Two-step Time Alignment

Commit `2e2cca4 refactor(anlf): replace sequence-position alignment with two-step timestamp bucketing`
建立：

1. per-IP/session anchor-relative rounding
2. same-session same-center last-wins dedup
3. relative-to-`snappedNow` global alignment
4. late joiner alignment
5. output timestamp 使用 contributing session centers 的平均值

### 4.2 Internal Zero-padding

Commit `3936e7d feat(anlf): add dedup warn, per-slot IP count debug log, and zero-padding`
明確要求 continuous slot range 中缺少的中間 slot 產生 zero-valued observation。

### 4.3 Target Slot And Ground Truth

Commit `e8c8978 fix(anlf): align ue communication target slots` 建立：

1. prediction target 由最後一筆 historical slot 推導
2. ground truth 使用 slot-key equality
3. pending prediction miss/retry/discard lifecycle

Commit `5ca01e9 fix(monitor): delay ground truth lookup until UPF report period has elapsed` 顯示舊設計
曾特別處理 observation 尚未到齊就過早 matching 的問題。

### 4.4 Historical Warmup And Current Readiness Decision

Commit `71a6564 fix(anlf): discard warmup-period predictions before accuracy scoring` 要求 startup
warmup 期間 predictions 與 inference count 不得進入後續評分。

Commit `df628b0 feat(mtlf): enforce baseline-ready and startup-only warmup` 建立當時的
startup-only policy。R2重新檢查current predictor後確認，input window不足已由`confidence == 0`
明確表示，且confidence-zero已核准不進accuracy baseline；current stateless TCN也沒有需要任意秒數
建立的model state。

Confirmed decision（2026-07-13）：R2移除fixed-duration warmup與`warmup_duration` config，以
confidence作data readiness gate。Generation replacement清除舊generation accuracy state，但不延遲
confidence-positive new-generation predictions。這是取代historical startup warmup的approved change。

### 4.5 Scope And Channel Semantics

Commit `6529345 feat(anlf): add scoped accuracy monitoring foundations` 建立 canonical scope
snapshot 與 per-scope report。

Commit `4ffed92 fix: replace NRMSE with sMAPE and fix in-memory ground truth aggregation` 明確要求
group traffic 必須聚合所有 SUPIs，而不是第一個 match 就完成。

所有舊 metrics 將 UL 與 DL 視為兩個獨立 samples。

### 4.6 Measurement Timestamp

Commit `47cdf0d fix(upf_notify): use startTime as measurement bucket timestamp` 要求使用 UPF
measurement period 的 `startTime`，只有缺少時才 fallback 到 notification `timeStamp`。
此部分目前 Go normalization 仍有保留。

---

## 5. Confirmed Phase 3 Findings

### BP-01: Missing Internal Zero-padding

Severity: P0

Current `AnalyticsRuntime._shape_observations()` 只迭代實際存在的 bucket indexes。

Current behavior：

~~~text
t=0  A
t=30 missing
t=60 B
-> [A, B]
~~~

Historical behavior：

~~~text
-> [A, zero, B]
~~~

Impact：

1. 時間間隔被壓縮
2. predictor 可能把不連續的 30 個有值 buckets 誤判為完整 30-slot window
3. confidence 與模型輸入語意改變
4. prediction target 也跟著改變

Conclusion: confirmed migration regression.

### BP-02: Observation Source Identity Lost Before Shaping

Severity: P0

Current flow：

1. `ObservationStore.observations_for_subscription()` flatten 多個 source buffers
2. returned `SourceObservation` 不帶 `observation_source_id`
3. session key 只有 `(ipv4_address, supi, dnn, snssai)`

不同 correlation/source 的相同 metadata 會被當成同一 session。若 timestamp 映射到同一
center，後寫入者覆寫前者，而不是像舊 Go 一樣先保留 source/session，再於 global bucket
相加。

`_subscription_sources` 是 `set`，因此 flatten ordering 也不應被當成穩定 winner policy。

Recommended internal shape：

~~~text
StoredObservation {
    observation_source_id
    observation
}
~~~

不一定需要修改 external observation payload，因為 source ID 已存在於 endpoint path。

Conclusion: confirmed migration regression.

### BP-03: Global Alignment Origin Changed

Severity: P0

Historical Go：

~~~text
snappedNow = floor(inference now / sampling interval) * sampling interval
globalIndex = GoRound((sessionCenter - snappedNow) / sampling interval)
~~~

Current PyAnLF：

~~~text
latest = max(raw observation timestamps)
globalIndex = PythonRound((sessionCenter - latest) / sampling interval)
~~~

同一批 observations 會得到不同 global index、history endpoint 與 prediction target。

Conclusion: confirmed migration regression.

### BP-04: Output Timestamp Changed From Mean Center To Maximum Raw Timestamp

Severity: P0

Historical Go 以 contributing session centers 的 arithmetic mean 作 slot timestamp。Current
PyAnLF 使用 `max(item.observed_at)`，而且使用 raw timestamp 而非 snapped center。

這會改變 historical last timestamp、analytics output timestamp、prediction target slot 與
accuracy matching target。

Conclusion: confirmed migration regression.

### BP-05: Python Rounding Is Not Go `math.Round`

Severity: P1 conditional

Go `math.Round` 是 half away from zero；Python `round()` 是 half to even。當 timestamp ratio
恰好落在 half-slot boundary 時會得到不同 bucket。

修復時應提供明確的 Go-compatible helper，並同時測試正負 half boundary。

Conclusion: confirmed conditional bug.

### BP-06: Analytics Parity Tests Were Not Implemented

Severity: P0 verification gap

Phase 3 plan 要求 per-session alignment、late joiner aggregation、input window
truncation/padding、output timestamps 與 scheduler limits。

Historical Go tests包含 drift、dedup、multi-IP same/different anchor、input window cap、
zero-padding 與 scope canonicalization。Current Python analytics tests只涵蓋 source refcount、
batch ID dedup、confidence-0 fallback 與 shutdown cleanup。

Conclusion: planned acceptance criteria were not satisfied.

### R1 Closure Update（2026-07-13）

BP-01至BP-06已由R0 Analytics slice與R1 implementation關閉：

1. 建立11組具有historical provenance的analytics golden fixtures
2. 修復前以`8 failed, 3 passed`重現zero-padding、source identity、global origin、mean timestamp與
   rounding差異
3. PyAnLF新增source-aware snapshot與pure two-step alignment
4. Predictor input、first target timestamp、retention、fallback clock與source ordering均有component tests
5. Focused tests為`24 passed`，PyAnLF full suite為`41 passed`

Closure implementation：`PyAnLF@fc59df2`。

這項closure只代表analytics shaping已恢復。BP-07之後的accuracy、identity/generation與reporting
lifecycle findings仍維持open；Phase 3/4整體狀態仍是`behavioral parity validation incomplete`。

---

## 6. Confirmed Phase 4 Accuracy Findings

### BP-07: Ground-truth Tolerance Includes Adjacent Slots

Severity: P0

Current config：

~~~yaml
sampling_interval: 30
matching_tolerance: 30
~~~

Current matcher：

~~~text
abs(observed_at - target_time) <= tolerance
~~~

因此 target 前後一個完整 sampling interval 的 records 都可能被相加。Historical Go 使用
slot-key equality，接受範圍接近半個 sampling interval，並有 adjacent-slot rejection test。

Historical clarification：舊Go的Mongo query會先取`targetSlotTime ± samplingInterval`，總candidate
window寬度為`2*SI`；但每筆record仍須通過slot-key equality，因此該query window不是matching
tolerance。更早期`TargetTime+2*SI` maturity grace後來也已改成derived miss-count policy。

Conclusion: confirmed current-config bug；`matching_tolerance`應移除，不得拿舊`2*SI`概念替代。

### BP-08: Observation-driven Matching Consumes Partial Multi-source Truth

Severity: P0

每次 source batch ingest 後立即逐 subscription 呼叫 accuracy processing。只要任一 candidate
存在，prediction 就從 pending 移除。

問題不只 group 的多 sources：同 model/scope 的多 subscriptions 也可能由第一個
subscription 先達到 `min_samples`，清除共享 scope state，改變其餘 subscriptions 的結果。

Historical Go 是 per-model periodic round，一輪會看到當時所有 correlation IDs。舊版仍不是
strict all-source completeness protocol，但不會由每個 source arrival 立即消耗 prediction。

Conclusion: confirmed race/order-dependent behavior.

### BP-09: UL And DL Collapsed Into One Scalar

Severity: P0

Current prediction與actual都先壓成 `UL + DL`。若 actual 是 UL=100/DL=0，prediction 是
UL=0/DL=100，current metrics為零誤差；historical metrics會正確記錄兩個方向各自100的誤差。

Conclusion: confirmed metric semantics regression.

### BP-10: Metric Formulas Changed

Severity: P0

Historical formulas：

1. MAE/MSE/sMAPE across independent UL and DL channels
2. WAPE denominator是所有 channel actual absolute values 的總和
3. NRMSE denominator是 all-channel mean absolute actual
4. zero actual denominator時 WAPE與NRMSE回0

Current formulas：

1. operate on collapsed traffic scalar
2. NRMSE denominator使用 range/max absolute actual
3. zero denominator用 `1e-9`

Phase 4 plan明確要求 existing Go tests作 compatibility oracle，不允許本 phase重新定義公式。

Conclusion: confirmed plan violation.

### BP-11: Traffic Scale Units Changed

Severity: P0

Historical：

~~~text
TrafficScale = mean(abs(actual UL/DL samples))
PredictedTrafficScale = mean(abs(predicted UL/DL samples))
~~~

Current：

~~~text
ActualTrafficScale = sum(actual UL+DL totals)
PredictedTrafficScale = sum(predicted UL+DL totals)
~~~

Go MTLF仍直接用這些值判斷 `minDecisionTrafficScale`、`maxActualTrafficScale`、
`minPredictedTrafficScale` 與 `predictionOvershootRatio`。因此 testbed policy thresholds
的單位已被改變。

Conclusion: confirmed policy input regression.

### BP-12: `check_interval` Was Planned But Not Implemented

Severity: P0

Phase 4 plan明確要求 PyAnLF依 `check_interval` periodic evaluate mature predictions。Current
config沒有此欄位，monitor也沒有ticker/thread，accuracy reports由observation ingestion驅動。

Impact：

1. report frequency依source數量與arrival pattern變化
2. MTLF baseline建立速度改變
3. `requiredHitsInWindow` 不再代表原本的時間跨度
4. burst observations可快速產生多個policy rounds

Conclusion: confirmed plan violation.

### BP-13: Matched Samples Accumulate Across Ingest Calls

Severity: P0

Current `ScopeState` 保存matched samples直到累積到 `min_samples` 才report。Historical Go每個
periodic check只使用該輪matched pairs；resolved predictions不會累積成下一個measurement round。

這會改變 report window、sample composition 與 MTLF round semantics。

Conclusion: additional confirmed migration difference not listed in the initial scan.

### BP-14: Time-based Warmup Is Redundant With Confidence Readiness

Severity: resolved design change required in R2

Current warmup只跳過processing；predictions仍進入pending，warmup結束後可能被拿來matching。若要
保留historical time gate，這確實是state bug；但R2已確認不再維護第二套time readiness policy。

Conclusion：移除time-based warmup。Confidence-zero不建立prediction；generation replacement清除舊
state；confidence-positive prediction直接進入periodic accuracy lifecycle。

### BP-15: Accuracy Report State Is Cleared Before Delivery Result

Severity: accepted reliability tradeoff; observability required

Current flow：

~~~text
build report
clear matched state and context
call sender
ignore sender boolean result
~~~

Phase 4 failure semantics允許finite retries後drop，因此這不完全是plan violation；但它與舊
in-process callback可靠性不同，且caller無法retry同一state。

2026-07-13已決定保留current finite retry後drop，不建立process-local pending outbox。這是為了
維持implementation simplicity而接受的cross-process reliability差異，不視為historical parity。
實作仍必須保留stable report ID、retry exhaustion error log與delivery failure metric，避免silent
loss；state bookkeeping問題則與delivery policy分開處理。

Conclusion: risk accepted by explicit decision; no outbox remediation in this round.

### BP-16: Pending Prediction And Report Context Become Inconsistent

Severity: P1

產生report後current code清除inference count、subscription IDs、observation source IDs與
matched samples，但尚未matched predictions繼續留在pending。

它們日後匹配時可能得到inference count低估、空retrain context與不一致的window start。
Prediction本身已有subscription/source snapshot，可以重建context，但current matcher沒有這樣做。

Conclusion: confirmed state bookkeeping bug.

### BP-17: Scope Canonicalization Regressed

Severity: P1

Current `monitoring_context()`：

1. 固定取第一個event subscription
2. 不明確尋找 `UE_COMMUNICATION`
3. 不trim/sort/deduplicate target lists
4. 沒有tracked resource fallback
5. 空target/`AnyUe` 不會維持historical unresolved semantics

JSON `sort_keys=True` 只排序object keys，不排序list values。

Conclusion: confirmed migration regression.

### BP-18: Accuracy Compatibility Tests Were Not Implemented

Severity: P0 verification gap

Phase 4 plan要求exact slot matching、late truth、expiry、Go metric fixtures、per-scope inference
count、readiness、same/different identity separation與successful/failed generation replacement。

Current accuracy tests只涵蓋simple scalar metrics、single-source exact timestamp matching與
generation change clears pending state。

Conclusion: planned acceptance criteria were not satisfied.

---

## 7. Model Lifecycle And Identity Findings

### BP-19: Shared Model Registry Still Uses Artifact URL As Identity

Severity: P0

Phase 4 plan要求registry、monitor與MTLF policy不得以artifact reference作key，且要求same
identity reuse與different identity separation tests。

Current PyAnLF：

~~~text
_shared_models: dict[model_reference, SharedModelRuntime]
~~~

Consequences：

1. different identity + same URL會取得第一個shared runtime
2. second runtime保存第一個identity，而非request identity
3. later provision correlation與accuracy identity被污染

Minimum reproduction confirmed：

~~~text
apply provider/1 at http://models/latest
apply provider/2 at http://models/latest
second runtime identity == provider/1
~~~

Conclusion: confirmed plan violation and current bug.

Closure（2026-07-14）：Closed by `PyAnLF@01620ce`。Identified shared runtime現在以
`(provider_id, model_unique_id, generation)`為key，anonymous runtime使用獨立full-reference registry；
different identity/same URL tests確認不再共享錯誤identity或model instance。

### BP-20: Same URL New Generation Does Not Reload

Severity: P0 conditional on mutable/reused URL

`apply_provision_event()` 會計算next generation，但 `_get_or_load_shared_model()` 只要URL已存在
就直接回傳舊 `SharedModelRuntime`。

Minimum reproduction confirmed：

~~~text
current generation = 1
provision same identity + same URL as update
response status = APPLIED
response generation = 1
model load calls remain 1
~~~

此結果會假裝成功但沒有載入新artifact。

Conclusion: confirmed conditional major bug.

Closure（2026-07-14）：Closed by `PyAnLF@01620ce`。新的internal event ID代表新generation event；即使
artifact URL相同仍會重新呼叫model load並在成功commit後切換generation。Duplicate same event ID則回原response，
不重複load或增加generation。

### BP-21: Artifact Cache Uses URL Last Segment

Severity: P2 pre-existing risk

`ModelManager` 以URL最後一段作cache directory；相同最後segment的不同server URL會碰撞，同URL
content update也不會重新下載。

此行為在Phase 2前已存在，不應標成Phase 4 migration regression；但generation成為正式concept
後，現有cache key不再足以支援target design。

Recommended direction：

1. runtime registry key使用 `(provider_id, model_unique_id, generation)`
2. identified cache至少使用model identity + generation + full URL hash
3. artifact digest有contract source時再加入驗證

Conclusion: valid risk, incorrect attribution if called a new Phase 4 bug.

Closure（2026-07-14）：Closed for R3 target behavior by `PyAnLF@01620ce`。Identified HTTP(S) cache key包含
canonical identity、generation與full reference hash；anonymous cache也使用full-reference hash。Host-tail、
same-URL/different-generation、different-identity與concurrent cache tests已加入。Digest與cache GC仍是future work。

### BP-22: Provision Event Commit Is Not Atomic Across Runtimes

Severity: P1 concurrency

Phase 4 plan要求candidate成功後atomically replacement所有affected runtimes。Current code逐
runtime取得subscription lock，並在每次迭代分開取得state lock。Reporting threads可能在fan-out
中途看到部分runtimes已切換、部分仍是舊generation。

Current mutation path通常不會在commit loop內回傳error，但這不等於externally observable
atomicity。

Conclusion: additional design/implementation gap.

Closure（2026-07-14）：Closed by `PyAnLF@01620ce`。Provision先保存affected set/runtime/model/binding snapshot，
lock外prepare candidate，再依subscription stripe index固定順序取locks並re-resolve；任一stale即整批拒絕並卸載
candidate。Success時runtime assignments、shared references、current generation與accuracy metadata在同一state-lock
critical section切換。

### BP-23: Explicit Provision Event Idempotency Is Missing

Severity: P1

Phase 4 verification plan要求duplicate provision event idempotency。Current runtime manager沒有
processed event ID/fingerprint registry；部分duplicate case只因URL cache而碰巧no-op，不能視為
contract-level idempotency。

2026-07-13已決定在internal contract加入explicit `event_id`，保證Go到PyAnLF同一 forwarding
operation的retry idempotency。Go建立event時只產生一次ID並在retry重用；Daisy path可由stable
`training_task_id`衍生。External MTLF重送一個新的標準callback request不在本輪保證範圍，因為
現有payload沒有可可靠區分duplicate與same-URL new generation的event UUID。

Conclusion: additional planned-test and contract gap; idempotency scope resolved by Decision 8.

Closure（2026-07-14）：Closed by `PyAnLF@01620ce`與`NWDAF@e6d295b`。Required `event_id`已同步進入
Go/Python private contract；Daisy採stable task ID derivation，external MTLF ingress採per-event UUID；Go bounded retry
重用同一body/ID。PyAnLF completed/in-flight registry處理sequential/concurrent duplicate、payload conflict、failure retry
與bounded eviction。

---

## 8. Reporting And Subscription Lifecycle Findings

### BP-24: Scheduler Completion Does Not Notify Go

Severity: P1

Current PyAnLF scheduler在以下情況直接結束thread：

1. `maxReportNbr` reached
2. `monDur` reached
3. report generation exception

沒有callback Go，也沒有完整local runtime release。

Result：

1. Go subscription仍active
2. PyAnLF runtime/model/binding仍配置
3. source buffer持續ingest
4. Go data collection持續存在

Historical correction：

1. pre-Phase 2 completion會把Go subscription設inactive
2. Phase 2後另會release AnLF runtime
3. old completion callback沒有完整release SMF/UPF collection

因此regression是「連inactive/model runtime completion都遺失」，而不是「舊版原本會完整清理
所有資源」。

Recommended lifecycle shape：

1. PyAnLF先完成本地scheduler/runtime/model/binding completion
2. PyAnLF建立stable completion event與process-local tombstone
3. callback失敗時持續retry至Go確認或PyAnLF process shutdown
4. Go idempotently標記subscription inactive，但本輪不清理SMF/UPF data collection references
5. Go callback不得同步DELETE仍在等待callback response的Py scheduler，避免waiting cycle

Conclusion: confirmed lifecycle regression; completion behavior and delivery guarantee resolved by
Decision 3 and Decision 9. Full collection cleanup remains future lifecycle work.

### BP-25: Report Generation Exception Permanently Stops Scheduler

Severity: P1

Inference exception已在analytics generation內轉成fallback，但其他parsing/state exception會讓
scheduler thread永久結束，Go沒有任何runtime-failed/completed signal。

應區分：

1. transient generation error：log並保留下一個tick
2. stale/released runtime：正常停止
3. unrecoverable runtime state：完成或失敗callback Go

Conclusion: confirmed lifecycle risk.

### BP-26: `immRep` Difference Is Intentional

Severity: no defect

Historical Go scheduler無條件立即送第一份report，沒有正確使用 `immRep`。Current PyAnLF：

1. `immRep=true` 立即送
2. `immRep=false` 等待第一個 `repPeriod`

Phase 3已明確決策採current behavior。不得為了parity恢復舊bug。

Conclusion: intentional behavior correction.

---

## 9. Observation Transport Finding

### BP-27: Global Delivery Worker Causes Head-of-line Blocking

Severity: P2 operational risk

Current Go observation delivery使用一個global queue與一個worker。某source執行request timeout、
retry與retry interval時，其他sources必須等待。Queue滿後batch會被drop並記log；UPF callback
仍完成，不會要求source重送。

Important classification：

1. 這是新process boundary帶來的真實failure mode
2. Phase 3 plan已明確選擇bounded in-memory queue、有限retry與overflow drop
3. queue data loss也已列入Phase 3 risks
4. 因此它不是未揭露的parity bug
5. 改成per-source ordered workers或durable outbox屬新的architecture decision

Conclusion: valid risk and possible future improvement, but not a blocker for restoring analytics algorithm parity.

---

## 10. Additional Findings Not In The Initial Scan

### BP-28: Multi-step Prediction Bookkeeping Is Collapsed

Severity: P1 conditional

`AnalyticsRuntime` 可將多個prediction steps合併成一個communication，accuracy manager只建立一筆
prediction並使用第一個target timestamp。

Historical Go會為每個prediction step分別建立prediction record與target slot。

Current config `output_window=1`，current predictor也只回一筆，因此目前未觸發；若output window
或model output增加，會把多個future slots總和拿去比第一個slot ground truth。

### BP-29: Confidence-zero Predictions Are Skipped

Severity: P1 semantics clarification; decision resolved

Current runtime只有 `confidence > 0` 才記accuracy prediction。Historical Go沒有在已取得model
output後顯式檢查confidence，但一般insufficient-history path會直接建立confidence-zero fallback
analytics，並不建立accuracy prediction record。

因此本輪正式採用以下語意：

1. confidence-zero prediction不進accuracy baseline
2. 保留log與metric，讓使用者能觀察資料不足或low-confidence發生
3. 不在本輪建立獨立unavailable/low-confidence report state

這符合原本「無參考價值的資料不足prediction不應污染baseline」的設計意圖，並取代fixed-duration
warmup；與historical code在罕見backend model output本身回傳confidence-zero時的無條件record差異，
視為已核准的語意明確化，必須加入compatibility/policy test。

### BP-30: Phase 4 Measurement Config Is Incomplete

Severity: P1 plan gap

R2所需的以下active config/validation未完整存在：

1. `check_interval`
2. derived `max_miss_count = ceil(2*SI/checkInterval)+1`
3. explicit late observation/miss policy
4. exact slot parameters validation

Current code只讀enabled、min samples、warmup、prediction retention與matching tolerance。
Confirmed remediation decision是移除`prediction_retention`與`matching_tolerance`的active config
ownership，並一併移除已由confidence readiness取代的`warmup_duration`；prediction改回derived
miss-count lifecycle。後續config provenance檢查另確認`source_retention`也是Phase 3新增語意，應由
historical per-source/IP count-bounded ring取代。

Historical `metricsToRecord`雖存在於config與helper，production monitor仍固定建立全部五種metrics，沒有
依該欄位filter。Confirmed decision是不在R2首次啟用此新behavior：PyAnLF固定回報全部五種metrics，
Go `primaryMetric`只選擇MTLF policy使用的metric。Go端已不屬measurement owner的check/min/warmup/
metric-list殘留欄位與helper一併移除，避免形成虛假的雙重ownership。

### BP-31: Observation Buffer Bound Changed From Per-session Count To Per-source Time

Severity: P1 shared analytics/accuracy state regression

Historical Go在每個correlation ID/IP session的`RawUpfData`使用`ringBufferSize`：append後若超過容量，
只移除該session最舊records；default與testbed值為50，且validation要求
`ringBufferSize >= inputWindow`。

Current PyAnLF把一個source的所有sessions放在同一flat list，使用`source_retention`按timestamp清除。
這同時改變：

1. buffer identity從per-session變成per-source
2. bound從record count變成wall-clock age
3. 不同session資料量對memory與retained history的影響
4. 低頻或jittered observations可保留的實際筆數

Confirmed decision（2026-07-13）：恢復per-source/IP fixed-size ring，config使用
`ring_buffer_size: 50`並驗證不小於`input_window`；移除`source_retention`。Prediction expiry仍由
derived miss count處理，`batch_dedup_capacity`維持獨立transport/idempotency用途。PyAnLF ring恢復後，
current Go已無production reader的`RawUpfData`、`groundTruthRetention` config與append/cap path一併
移除；SourceObservation forwarding、MongoDB與ADRF路徑必須保持不變。

---

## 11. Initial Scan Assessment

以下表格保存對外部掃描報告21個原始項目的判定。

| Item | Assessment | Audit conclusion |
| --- | --- | --- |
| 1 zero-padding | Confirmed | P0 migration regression |
| 2 source collision | Confirmed | P0，且source ordering不可靠 |
| 3 global origin/timestamp | Confirmed | P0 migration regression |
| 4 rounding | Confirmed conditional | P1 half-slot bug |
| 5 tolerance adjacent slot | Confirmed | P0 current-config bug |
| 6 first source consumes truth | Confirmed | P0；scope/multi-subscription範圍更大 |
| 7 UL/DL collapse | Confirmed | P0 metric regression |
| 8 NRMSE changed | Confirmed | P0 plan violation |
| 9 zero traffic changed | Confirmed | P0 if preserving old metrics |
| 10 traffic scale sum vs mean | Confirmed | P0 policy input unit regression |
| 11 cadence changed | Confirmed | P0；plan explicitly required periodic check |
| 12 warmup changed | Confirmed but superseded | R2移除time-based warmup，以confidence readiness取代 |
| 13 failed delivery clears state | Confirmed behavior | reliability risk；finite-drop was planned policy |
| 14 pending/context mismatch | Confirmed | P1 bookkeeping bug |
| 15 scheduler completion leak | Partially correct | inactive/model cleanup regression；old SMF cleanup claim overstated |
| 16 scheduler exception stops | Confirmed | P1 lifecycle risk |
| 17 `immRep` changed | Intentional | current behavior should remain |
| 18 scope canonicalization | Confirmed | P1 regression |
| 19 URL-only shared model | Confirmed and worse | different identity is overwritten by first shared identity |
| 20 artifact cache | Valid pre-existing risk | not introduced by Phase 4 |
| 21 global worker HOL | Valid known risk | explicitly accepted Phase 3 architecture tradeoff |

---

## 12. Minimum Reproduction Evidence

2026-07-13以current PyAnLF source執行read-only minimum reproductions，得到：

~~~text
gap-shape:
[(2026-01-01T00:00:00Z, UL=1),
 (2026-01-01T00:01:00Z, UL=2)]
~~~

Expected historical shape包含 `00:00:30Z` zero slot。

~~~text
same-session-collision:
input UL=10 and UL=20 at same timestamp/session
output UL=20
~~~

Expected cross-source aggregation在source identity不同時是UL=30。

~~~text
collapsed-direction-metrics:
actual total=100
predicted total=100
sMAPE=MAE=MSE=WAPE=NRMSE=0
~~~

這個scalar representation無法辨識actual UL=100/DL=0與predicted UL=0/DL=100。

~~~text
tolerance-actual-scale:
target-30s UL=10
target     UL=20
reported actual scale=30
~~~

證實相鄰slot被納入。

~~~text
different-id-same-url:
requested second identity=provider/2
stored second runtime identity=provider/1
~~~

~~~text
same-url-new-generation:
provision response generation=1
model load call count=1
~~~

證實same-URL update沒有載入新generation。

---

## 13. Current Verification Result And Its Limit

2026-07-13重新執行：

1. `PyAnLF: uv run pytest -q`
   - 24 passed
2. `NWDAF: go test ./...`
   - passed

這些結果只證明current tests通過。它們沒有覆蓋本audit列出的historical parity fixtures，因此
不得用來否定confirmed findings。

三個repository在測試後仍為clean worktree。

---

## 14. Recommended Remediation Order

以下順序是audit recommendation，尚未形成正式phase plan。

### R0: Containment And Oracle

1. 在parity恢復前，不應以current accuracy output校正testbed thresholds或宣稱retrain結果有效
2. 將historical Go tests移植為PyAnLF compatibility tests，先確認它們在current code失敗
3. 對每個fixture記錄baseline commit、input、expected historical sequence/report
4. 將Phase 3/4文件狀態維持behavioral parity incomplete

### R1: Restore Analytics Shaping

1. ObservationStore內部保留source ID
2. session key包含source ID與session identity
3. 移植per-session anchor round與last-wins dedup
4. 使用Go-compatible rounding
5. global origin使用inference-time snapped grid
6. global bucket sum跨source/session traffic
7. output timestamp使用mean snapped centers
8. 建立continuous output range與internal zero-padding
9. 完整移植historical analytics tests

### R2: Restore Accuracy Measurement Semantics

1. prediction保存UL/DL channels與per-step target slot
2. observation query保留source/session identity
3. per-source/session/slot last-wins後再跨source聚合
4. 改回slot-key equality
5. 使用per-model periodic `check_interval`
6. 每輪同時處理model所有scopes/subscriptions
7. 指定maturity/settling policy
8. 完整移植Go metric與traffic-scale formulas
9. confidence-zero不進pending，generation replacement清除舊accuracy state
10. 修正pending/context/inference bookkeeping
11. 補齊late truth、expiry、multi-source與failure tests

### R3: Correct Identity And Generation

1. shared runtime key改為model identity + generation
2. different identities即使artifact URL相同也分離
3. same identity new generation一定prepare新candidate
4. candidate commit對affected runtimes具可驗證的atomic boundary
5. identified cache key加入model identity、generation與full URL hash
6. internal provision event加入stable `event_id`，Go retry重用同一ID
7. 增加same/different identity、same URL update與duplicate event tests

### R4: Complete Lifecycle And Reliability

1. 定義runtime completed/failed internal callback
2. PyAnLF local completion後保留process-local tombstone，retry callback至Go確認或shutdown
3. scheduler transient error可恢復
4. 保留accuracy report finite retry後drop，補足retry exhaustion observability
5. Go completion handler idempotently標記inactive，本輪不新增SMF/UPF collection cleanup
6. 補callback retry、completion dedup、shutdown與cleanup boundary tests

### R5: Cross-language And End-to-end Verification

至少建立：

1. historical Go-derived golden fixtures
2. PyAnLF shaping output comparison
3. PyAnLF accuracy report comparison
4. Go MTLF policy input unit comparison
5. real Go/PyAnLF HTTP contract test
6. 環境允許時執行SMF/UPF/MTLF/ADRF/Daisy end-to-end

---

## 15. Required Decisions Before Remediation Implementation

Confirmed compatibility principle（2026-07-13）：本輪以最大程度恢復搬移前Go behavior為主，
不把transport、concurrency或strict completeness改進混入parity remediation。所有decision
gates已依下列紀錄確認；明確核准的可靠性或policy差異必須有測試與observability。

### Decision 1: Multi-source Ground-truth Completion

Options：

1. historical-style periodic best-effort
   - check round到達時聚合已到資料
   - 最接近舊behavior
   - 無法保證所有sources都到齊
2. expected-source completeness + timeout
   - prediction snapshot中的sources全部到達才finalize
   - timeout後依policy做partial/miss
   - reliability較好，但屬正式behavior change

Audit recommendation：先採historical-style periodic best-effort恢復parity，再把strict completeness
作為另一次有fixtures與threshold recalibration的改進。

Confirmed decision（2026-07-13）：選項1。使用per-model periodic check round，聚合該輪已到達
的所有source資料；expected-source completeness與timeout policy留作future improvement。

### Decision 2: Accuracy Callback Delivery Guarantee

Options：

1. 保留current finite retry後drop
2. process-local pending outbox，204後才commit/clear
3. durable outbox across restart

Audit reliability recommendation原為選項2，因為較接近舊in-process callback；但選項1實作較
簡單，也符合Phase 4既有finite-retry contract。

Confirmed decision（2026-07-13）：選項1。Accuracy callback維持有限retry，耗盡後drop，不建立
process-local或durable outbox。此選擇明確接受比historical same-process direct callback更弱的
delivery guarantee；必須提供retry exhaustion error log與delivery failure metric，且測試應驗證
失敗後state不會破壞後續accuracy rounds。

### Decision 3: Subscription Completion Cleanup

`maxReportNbr`/`monDur` completion的候選cleanup boundary：

1. 只停止reporting
2. 停止PyAnLF runtime並將Go subscription設inactive
3. 完整清理PyAnLF runtime、Go subscription與SMF/UPF source references

Audit ideal recommendation：選3可避免collection leak；但historical parity baseline應以Phase 2
完成後的`62e2f9f`為準，當時completion會將Go subscription設為inactive並release AnLF runtime，
不會清理SMF/UPF data collection references。

Confirmed decision（2026-07-13）：本輪選項2。PyAnLF先停止並release local runtime，再callback
Go將subscription設為inactive；不在本輪加入SMF/UPF collection cleanup。完整collection cleanup
另列future lifecycle improvement，避免混入parity restoration。

### Decision 4: Same URL Artifact Mutability

Options：

1. contract強制artifact URL immutable，每generation必須新URL
2. 允許same URL update，以generation/digest強制reload

Audit recommendation：runtime design必須支援2；即使部署慣例通常產生新Daisy task URL，也不應讓
same-URL update回傳假成功。

Confirmed decision（2026-07-13）：選項2。shared runtime以model identity與generation識別；
generation改變時即使artifact URL相同也必須prepare/reload新candidate。Artifact digest可作完整性
驗證，但不阻塞第一輪identity/generation修復。

Confirmed refinement（2026-07-14）：identified artifact cache key同時納入model identity、generation與
full artifact reference。不同identity即使使用相同URL與相同generation number也不共用disk cache；
未來取得artifact digest後再評估安全的cross-identity cache deduplication。

### Decision 5: Confidence-zero Accuracy

Options：

1. 完整保留舊Go，model output不因confidence-zero被排除
2. confidence-zero不進accuracy baseline，但提供log/metric
3. 建立獨立unavailable/low-confidence report state

Confirmed decision（2026-07-13）：選項2。confidence-zero不進accuracy baseline，但必須有log與
metric。這主要明確化historical insufficient-history fallback的原設計意圖，並作為R2唯一的data
readiness gate；不在本輪增加獨立low-confidence report state或fixed-duration warmup。

### Decision 6: Accuracy Round Sample Accumulation

Options：

1. historical per-check-round samples，不跨round累積
2. current accumulate-until-min-samples

Audit recommendation：選1以恢復MTLF cadence與threshold calibration。

Confirmed decision（2026-07-13）：選項1。每個check round只使用該輪成功matching的samples，
不跨round累積到`min_matched_predictions`，以恢復historical report cadence與MTLF threshold單位。Prediction
等待語意另由Decision 11的derived miss-count policy處理，不恢復舊maturity consume gate。

### Decision 7: Observation Delivery Concurrency

Options：

1. 保留single global worker，先不擴張本次parity修復
2. per-source ordered workers
3. durable/replayable transport

Audit recommendation：本次先選1；R1/R2恢復後再根據實驗pressure與drop metrics決定是否另開工作。

Confirmed decision（2026-07-13）：本輪parity remediation採選項1，維持現有single global
worker，不修改Go到PyAnLF的observation delivery concurrency、queue或retry架構。
Head-of-line blocking與queue overflow仍屬已知風險，但記為future transport work，不納入
本輪historical analytics/accuracy behavior restoration，避免同時引入新的ordering與delivery
semantics。

### Decision 8: Model Provision Event Idempotency Scope

Options：

1. 只保證Go到PyAnLF同一forwarding operation的retry idempotency
2. 連external MTLF重送的標準callback request也要deduplicate
3. event registry跨PyAnLF restart持久化

Confirmed decision（2026-07-13）：選項1。Internal `ModelProvisionEvent`新增stable `event_id`；
Go在建立event時產生一次並於所有internal retries重用，PyAnLF以process-local processed-event
registry處理duplicate。Daisy path優先由`training_task_id`衍生stable ID。External MTLF payload
目前沒有可靠event UUID，而canonical content fingerprint無法區分duplicate與已核准的same-URL
new generation，因此選項2、3不納入本輪。

### Decision 9: Subscription Completion Callback Delivery

Options：

1. finite retry後drop completion callback
2. process-local completion tombstone，retry至Go確認或PyAnLF shutdown
3. durable completion outbox across restart

Confirmed decision（2026-07-13）：選項2。PyAnLF先release local runtime，再保存每個completed
subscription一筆stable tombstone；Go callback回覆成功前持續retry，收到確認後移除。Go handler
必須idempotent，且只將subscription設為inactive；本輪不清理SMF/UPF collection references。
此delivery guarantee刻意強於accuracy report的finite-drop policy，因為completion遺失會留下錯誤
lifecycle state。

### Decision 10: Post-completion Observation Forwarding

Options：

1. 保留SMF/UPF collection，但source沒有active NWDAF subscription時停止backend enqueue
2. 即使PyAnLF runtime已release仍持續enqueue並接受404/retry
3. completion時完整unsubscribe/cleanup SMF/UPF collection

Confirmed decision（2026-07-13）：選項1。UPF traffic storage與ADRF forwarding繼續，且不刪除
SMF/UPF subscription；只有Go到PyAnLF的backend forwarding在該source已沒有active consumer時
停止。Shared source仍有active subscriptions時繼續source-scoped delivery。這不修改single worker
concurrency或queue semantics。

### Decision 11: `2*SamplingInterval` And Matching Policy

Historical clarification：

1. `5ca01e9`曾使用`TargetTime+2*SI`作prediction maturity grace
2. `e8c8978`後移除single-consume maturity gate，改成每round retry pending predictions
3. latest baseline使用`ceil(2*SI/checkInterval)+1`推導max miss count
4. Mongo的`targetSlotTime ± SI`只是candidate query window，final match仍為slot-key equality
5. analytics `lookbackBuffer`的default `2*SI`是另一個已移除的historical query setting

Confirmed decision（2026-07-13）：移除PyAnLF `matching_tolerance`與active
`prediction_retention` config；恢復exact slot-key equality與derived max miss count。`2*SI`只保留為
retry/discard時間尺度，不作matching tolerance。後續BP-31另將observation storage從time-based
`source_retention`恢復為per-source/IP count-bounded ring；兩者不是同一種retention policy。

### Decision 12: R0 Test-first Commit Strategy

Options：

1. 先commit明知失敗的compatibility tests
2. 本地先證明tests失敗，再與對應fix一起commit

Confirmed decision（2026-07-13）：選項2。必須保存initial failure evidence，但正式implementation
commits保持green；不得因此跳過test-first驗證或使用current PyAnLF output建立expected values。

### Decision 13: Accuracy Metric Output Selection

Historical clarification：`metricsToRecord`存在於Go config與helper，但production accuracy monitor固定
建立sMAPE、MAE、MSE、WAPE、NRMSE，沒有依該欄位過濾。

Confirmed decision（2026-07-13）：不在PyAnLF首次啟用historically inactive filtering。PyAnLF固定
計算並回報全部五種metrics；Go `primaryMetric`只負責選擇MTLF decision使用的metric。兩端移除
容易讓使用者誤以為會生效的metric-list config。

### Decision 14: Accuracy Sample-count Ownership

Current PyAnLF使用`min_samples`決定是否建立measurement report；current Go MTLF另用同名欄位決定
是否將report納入degradation reference，形成兩個責任不同但名稱相同的threshold。

Confirmed decision（2026-07-13）：PyAnLF將此設定改名為`min_matched_predictions`，並作為唯一
sample-count report gate。未達門檻時不送report；
已送達Go的report直接進入既有MTLF decision flow，不再以第二個`minSamples`重複拒絕。Primary metric、
baseline window、traffic gate、z-score與retrain thresholds仍由Go擁有。舊PyAnLF `min_samples`不保留
雙讀或deprecated alias，避免與模型input sample數量混淆。

### Decision 15: Go Transitional Ground-truth Buffer

Current Go仍把UPF observations append/cap到`RawUpfData`，但production accuracy reader已搬到PyAnLF，
repository scan確認只剩write path。PyAnLF恢復historical per-source/IP ring後，雙端buffer沒有相容性
價值。

Confirmed decision（2026-07-13）：R2移除Go `RawUpfData`、`groundTruthRetention.ringBufferSize`與
append/cap path。UPF SourceObservation建立與backend forwarding、MongoDB storage、ADRF forwarding
保持不變，並以targeted tests固定。

---

## 16. Required Golden Fixtures

最低fixture set：

1. clean single-source sequence
2. missing middle slot
3. anchor drift
4. same-session duplicate last-wins
5. two sources with same anchor
6. two sources with different anchors
7. two sources with same metadata but different source IDs
8. staggered multi-source arrival
9. input-window cap
10. mean output timestamp
11. positive and negative half-slot rounding
12. adjacent-slot ground truth rejection
13. UL/DL opposite-direction error
14. zero actual traffic
15. late ground truth before expiry
16. unmatched prediction expiry
17. confidence readiness and generation state reset
18. failed accuracy callback
19. same URL new generation
20. different identity same URL
21. duplicate provision event
22. `maxReportNbr` completion
23. `monDur` completion
24. reordered/duplicated SUPI/group target list
25. mixed-event subscription where `UE_COMMUNICATION` is not first
26. multi-step prediction target slots

Golden fixture expected values應由historical Go implementation/tests產生或人工核對，不得以current
PyAnLF output回填成新的expected results。

---

## 17. Documentation Status Rule

### 17.1 R2 Closure Update

2026-07-13完成R2 local implementation與verification。BP-07至BP-18、BP-28至BP-31已由以下
production changes與tests closure：exact slot matching、periodic per-model evaluation、UL/DL channel
semantics、historical metrics/traffic scales、round-local samples、confidence readiness、finite-drop
observability、prediction-owned context、scope canonicalization、per-step bookkeeping及per-source/IP ring。

Go端同步移除`RawUpfData`、ground-truth retention config與MTLF第二道sample gate。PyAnLF 74 tests、
NWDAF full tests、targeted race、build與lint均通過。該次R2 closure時live cross-process HTTP E2E尚未執行，
R3 identity/generation與R4 reporting completion findings仍保持open，因此Phase 3/4仍不得標記整體parity完成；
後續R3 closure見下一節。

### 17.2 R3 Closure Update

2026-07-14完成R3 repository-level implementation與verification。BP-19至BP-23由identity/generation registry、
anonymous isolation、same-URL reload、atomic affected-runtime commit、generation-aware full-reference cache、required
event ID與process-local idempotency closure。

Commits為`PyAnLF@01620ce`與`NWDAF@e6d295b`。PyAnLF 105 tests、NWDAF full/focused tests、targeted race、
build與lint均通過；完整review沒有未處理P0/P1 finding。FastAPI與Go client contract tests已驗證duplicate/retry
composition，但live cross-process tests因未設定`PYANLF_LIVE_ENDPOINT`而skip，所以維持
`environment-level HTTP E2E unverified`。

R3不是純historical Go parity：shared loading/fallback/release保留historical invariants；identity/generation separation
與same-URL reload是migration adaptation；multi-runtime atomic commit與explicit event ID是新增的cross-process
correctness guarantees。

### 17.3 Remaining Status Rule

在以下條件完成前，Phase 3與Phase 4不得恢復成無條件 `Completed`：

1. historical analytics shaping fixtures在PyAnLF通過
2. exact-slot、UL/DL與metric compatibility fixtures通過
3. identity/generation tests通過
4. reporting completion semantics完成決策與測試
5. 文件明確區分保留舊behavior與核准的新behavior
6. repository tests與cross-repo contract tests重新通過

Phase 3.5是Go package boundary的behavior-preserving refactor，目前沒有證據需要撤銷其完成狀態。
Phase 1 naming/boundary work也沒有在本audit發現主要behavior regression。

---

## 18. Immediate Next Step

R0至R3的對應implementation已完成。下一步應依`Behavioral Parity Remediation Plan.md`進入R4：

1. 完成scheduler正常終止時的runtime-completed callback
2. 讓Go idempotently標記subscription inactive並release backend runtime
3. 補transient scheduler error recovery與completion retry/tombstone
4. 維持SMF/UPF collection cleanup不在本輪的既定boundary
5. R4後執行R5 verification與status closure

本audit的主要原則是：

> 將搬移前Go logic視為behavioral baseline；只有經明確決策、測試與policy recalibration的差異，
> 才能成為新的PyAnLF正式語意。
