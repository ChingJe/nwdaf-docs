# Checkpoint B Detailed Design

## 1. Purpose

Checkpoint B is the policy-cutover phase of the retrain-monitoring work.
Its purpose is to move retrain decision ownership fully into MTLF and switch
from the legacy model-level deviation trigger to a per-scope decision flow
driven by `AccuracyReport`.

This phase delivers:

- MTLF-owned per-scope monitoring state
- report-based retrain decision input
- degradation-path gate on a primary metric
- chronic poor-quality path for models that start bad and stay bad
- per-scope cold-start protection
- configurable `M-of-N` breach policy
- config migration away from legacy EMA / deviation-threshold logic
- retrain lifecycle tests for the new decision path

Checkpoint B intentionally changes retrain behavior. Unlike Checkpoint A, this
phase is expected to replace the current legacy trigger path.

## 2. References

- Parent plan: `../NWDAF Retrain Monitoring Implementation Plan.md`
- Workflow and progress: `../NWDAF Retrain Monitoring Workflow and Progress.md`
- Existing detailed design: `./checkpoint_a_detailed_design.md`
- Project policy: `.agent/docs/development_policy.md`

## 3. Scope

### In Scope

- B1 MTLF per-scope state store and recent buffer
- B2 report-based degradation path and retrain trigger cutover
- B3 per-scope cold-start protection and both-zero handling
- B4 chronic poor-quality path and breach policy design
- B5 config migration and legacy trigger cleanup
- B6 retrain lifecycle, hot-swap, and concurrency tests

### Out of Scope

- threshold tuning based on longer staging observation
- changing the scope-key schema from Checkpoint A
- adding new external SBI surfaces
- removing CSV observability output
- extending scope identity with AoI or analytics-filter dimensions

## 4. Non-Negotiable Constraints

1. MTLF is the sole owner of degradation-policy state after Checkpoint B.
2. Retrain decision must operate at `model + scope` granularity, not a single
   model-level aggregate.
3. `AccuracyReport` remains the authoritative AnLF output; MTLF must not
   reconstruct scope or metric data from live subscription state.
4. The model-level in-flight retrain guard remains in `ModelAccuracyStore`.
5. A single degraded scope is sufficient to trigger model retraining once its
   own decision window reaches the configured threshold.

## 5. Current Baseline

Checkpoint A is already merged. The current project state before Checkpoint B is:

- AnLF emits per-scope `AccuracyReport` values and still emits the legacy
  model-level deviation callback in parallel.
- `processor.NewProcessor` wires only `SetOnDeviationReport(...)` into MTLF.
- `internal/mtlf/trigger.go` still uses:
  - `deviationThreshold`
  - `triggerStrategy`
  - `emaAlpha`
  - `ModelAccuracyStore.IncrementBreaches / ResetBreaches / UpdateEMA`
- `ModelAccuracyStore` still owns legacy trigger counters that logically belong
  to policy rather than to prediction storage.

Checkpoint B replaces that decision path instead of layering a second one on top.

## 6. Detailed Work Design

### B1. MTLF Per-Scope State Store

#### Objective

Give MTLF a dedicated state store for recent per-scope metric history, breach
counting, and basic baseline statistics.

#### Ownership Boundary

After Checkpoint B:

- `internal/context.ModelAccuracyStore` keeps prediction storage, monitor
  lifecycle, and model-level retraining guard
- `internal/mtlf` owns recent metric buffers, baseline statistics, breach
  counters, and scope GC

The state store must live in `internal/mtlf`, not in `internal/context`.

#### Proposed Types

```go
type ringBuffer struct {
    values []float64
    next   int
    filled bool
}

type ScopeState struct {
    scopeKey      string
    metricBuffers map[string]*ringBuffer
    reportCounts  map[string]int
    breachCount   int
    lastUpdate    time.Time
    mu            sync.RWMutex
}

type ModelMonitorState struct {
    scopes map[string]*ScopeState
    mu     sync.RWMutex
}

type MonitorStateStore struct {
    models map[string]*ModelMonitorState
    mu     sync.RWMutex
}
```

Notes:

- buffers are keyed by metric name so all recorded metrics remain available for
  future tuning, even though only one primary metric drives decisions
- `reportCounts[metric]` counts recorded monitor rounds for that metric
- `breachCount` stays scope-local because the decision rule is "any scope may
  trigger retrain"

#### Required APIs

The first implementation should expose a small, explicit surface:

```go
GetOrCreateScope(modelURL, scopeKey string) *ScopeState
DeleteModel(modelURL string)
GCExpiredScopes(now time.Time, ttl time.Duration)

RecordMetric(metric string, value float64, now time.Time)
SampleCount(metric string) int
Mean(metric string) float64
Std(metric string) float64
Min(metric string) float64
Max(metric string) float64
ResetBreach()
IncrementBreach() int
```

#### Lifecycle Rules

- scope state is created lazily on first `AccuracyReport`
- scope state is retained per `modelURL + scopeKey`
- scope state is deleted when:
  - `lastUpdate > scopeStateTTL`
  - model hot-swap completes and the old model URL is removed
- model-level state must be cleaned when retraining swaps to a new model URL

#### Concurrency Rules

- report handling, retrain callbacks, and GC may run concurrently
- `MonitorStateStore` handles model-level map protection
- each `ScopeState` protects its own buffers and breach counter
- no caller should hold both the global store lock and a scope lock longer than
  required for lookup/update

#### Files Expected To Change

- `internal/mtlf/mtlf.go`
- `internal/mtlf/trigger.go`
- new file under `internal/mtlf/` such as `state_store.go`
- `internal/mtlf/trigger_test.go`
- possibly a new dedicated state-store test file under `internal/mtlf/`

### B2. Report-Based Degradation Path Cutover

#### Objective

Make `AccuracyReport` the authoritative decision input and replace the legacy
deviation-based trigger implementation.

#### Cutover Strategy

Processor wiring should switch from:

```go
SetOnDeviationReport(...)
```

to:

```go
SetOnAccuracyReports(...)
```

The new authoritative MTLF entry point should look like:

```go
HandleAccuracyReports(
    modelURL string,
    reports []anlf.AccuracyReport,
    store *context.ModelAccuracyStore,
)
```

The legacy deviation callback should be treated as deprecated after the cutover:

- it may remain in AnLF temporarily to reduce merge risk
- it should no longer be wired into MTLF decision logic
- new tests must target the report-based path only

#### Decision Rule

For each report:

1. read `primaryMetric` from config
2. locate `current := report.Metrics[primaryMetric]`
3. load or create `ScopeState(modelURL, report.ScopeKey)`
4. evaluate:

```text
absGate = current > fixedFloor
baselineReady = existingHistoryCount >= minBufferSamples
if enough baseline:
    zscore = (current - mean) / max(std, minStd)
    relGate = zscore > zScoreThreshold
else:
    relGate = skipped
```

5. degradation trigger rule:

- cold-start phase: no trigger decision; this round only contributes to baseline history
- baseline-ready phase: `scopeTriggeredByDegradation = absGate AND relGate`

6. decision-window handling:

- if `baselineReady == false`, do not record a degradation hit for this round
- else record one degradation hit/miss for this scope
- the degradation path is satisfied when the scope's decision window reaches the
  configured `requiredHitsInWindow` inside the latest `decisionWindowSize`
  rounds
- if any scope satisfies the decision window, set `store.SetRetraining(true)`,
  reset all scope decision windows for that model, and start retrain workflow

#### Logging Contract

Checkpoint B logs should expose policy reasoning explicitly:

```text
scope=<scope> metric=<metric> current=<...> mean=<...> std=<...> zscore=<...>
absGate=<T/F> relGate=<T/F|skipped> baselineReady=<T/F>
degradationHits=<n>/<window> chronicHits=<n>/<window> triggerReason=<...>
```

#### Missing-Metric Handling

If the configured primary metric is absent from a report:

- skip that report for decision purposes
- keep processing the remaining reports
- emit a warning with `modelURL`, `scopeKey`, and the missing metric name

#### Files Expected To Change

- `internal/sbi/processor/processor.go`
- `internal/mtlf/trigger.go`
- `internal/mtlf/trigger_test.go`
- potentially `internal/anlf/anlf.go` only for deprecation comments / callback notes

### B3. Cold-Start Protection And Both-Zero Handling

#### Objective

Prevent unstable early buffers from producing false positives while keeping
recent history truthful and available once baseline becomes usable.

#### Cold-Start Rules

- `minBufferSamples` controls when z-score evaluation becomes active
- `baselineReady` is computed from the existing recent history before the
  current round is inserted
- before that threshold, the report is recorded into recent history but retrain
  decision is skipped entirely
- `minStd` is applied as the denominator floor for z-score computation
- monitor rounds with no ground-truth match remain fully excluded upstream
- scope breach counters must not accumulate during this baseline-building phase

#### Both-Zero Handling

Checkpoint B adopts the policy that an all-both-zero round is still a real
monitoring result and should remain part of recent history.

This means:

- the report remains valid for observability
- the metric value remains valid for the MTLF recent buffer
- `minBufferSamples` may be satisfied by these rounds
- retrain suppression is handled by policy gates rather than by filtering the
  data out of baseline history

#### Insert Rules

For each metric in `report.Metrics`:

- always evaluate the current report for logging
- always record the metric into the ring buffer when a scope report exists
- when `baselineReady == false`, do not let this round affect breach counting or
  retrain trigger state
- no extra `BaselineEligible` contract field is introduced in Checkpoint B

This keeps data handling simple and lets the retrain decision paths own trigger
semantics without inventing a second eligibility channel.

#### Files Expected To Change

- `internal/mtlf/trigger.go`
- `internal/mtlf/trigger_test.go`

### B4. Chronic Poor-Quality Path And Breach Policy Design

#### Objective

Add a second retrain path that captures scopes whose model quality is bad from
the beginning and remains bad, even when no strong relative anomaly is present.
At the same time, generalize the current strict consecutive-breach logic into a
shared `M-of-N` decision window.

#### Why This Is Needed

- the degradation path is intentionally a relative-anomaly detector
- it is effective when the baseline is mostly healthy and then worsens
- it is not sufficient when the baseline itself is already poor and highly
  variable
- in those cases, `fixedFloor` may pass but `z-score` may stay low because the
  recent baseline is already noisy

#### Chronic Path Principles

- the name `chronic poor-quality path` is intentional: it targets scopes whose
  quality remains poor over time, not scopes that merely show a new degradation
  spike relative to baseline
- the chronic path must use a sustained-quality signal, not a relative-anomaly signal
- `std > mean` or other high-variance indicators are not sufficient trigger
  conditions by themselves
- the first implementation should prefer normalized quality metrics over raw
  `MAE`
- the first implementation should not switch metrics by traffic regime
- traffic-regime handling should instead use an eligibility guard such as
  `minTrafficScale`

#### Chronic Metric Candidates

- `WAPE`
- `NRMSE`
- `MAE`

`MAE` remains available only as an explicit opt-in because it is sensitive to
traffic scale.

#### Aggregation Candidates

Checkpoint B should expose only:

- `mean`
- `percentile`

`median` is not a separate core mode; users should express it as
`percentile=50`.

#### Candidate Rule

```text
chronicValue = aggregate(recent chronicMetric history)
chronicEligible =
    baselineReady
    AND trafficScaleIsMeaningful
    AND notInWarmup
    AND notRetrainingInFlight

scopeTriggeredByChronic =
    chronicEligible
    AND chronicValue > chronicThreshold
```

#### Breach Policy Generalization

Instead of a strict reset-on-miss counter, Checkpoint B should generalize the
trigger dampening rule to:

```text
within the latest M rounds:
    if hits >= N:
        decision window satisfied
```

Constraints:

- `1 <= N <= M`
- `M = N` degenerates to the current strict consecutive behavior
- degradation path and chronic path may share one window config in the first
  implementation, even if internal bookkeeping remains per-path

#### Config Surface

Checkpoint B should add config for:

- `decisionWindowSize`
- `requiredHitsInWindow`
- `chronicPolicy.enabled`
- `chronicPolicy.metric`
- `chronicPolicy.aggregator`
- `chronicPolicy.percentile`
- `chronicPolicy.threshold`
- `chronicPolicy.minTrafficScale`

#### Implementation Status

This design has landed on branch `feat/retrain-monitoring-degradation-policy`
in commit `2406fdc`.

Current code now uses:

- degradation path plus chronic poor-quality path
- configurable `decisionWindowSize / requiredHitsInWindow`
- `chronicPolicy.*` config with shared window config and per-path hit tracking

### B5. Config Migration And Legacy Cleanup

#### Objective

Replace legacy trigger config with parameters that match the new policy model,
and remove obsolete trigger state from the project.

#### Config Migration

`AccuracyMonitorConfig` should keep:

- `enabled`
- `checkInterval`
- `minSamples`
- `warmupDuration`
- `consecutiveBreaches` (kept only as a backward-compatible fallback when the
  newer decision-window fields are omitted)
- `csvDumpDir`
- `csvDumpEnabled`

It should add:

- `metricsToRecord []string`
- `primaryMetric string`
- `recentBufferSize int`
- `minBufferSamples int`
- `minStd float64`
- `fixedFloor float64`
- `zScoreThreshold float64`
- `scopeStateTTL int`

It should remove:

- `deviationThreshold`
- `triggerStrategy`
- `emaAlpha`

#### Default Policy Values

The detailed plan should start from the parent-plan defaults:

- `primaryMetric = "MAE"`
- `recentBufferSize = 20`
- `minBufferSamples = 8`
- `minStd = 0.01`
- `zScoreThreshold = 3.0`
- `scopeStateTTL = 600`
- `consecutiveBreaches = 3` for the currently implemented path
- `decisionWindowSize = 3`
- `requiredHitsInWindow = 3`

`fixedFloor` is intentionally provisional and should be implemented with a
helper default, then tuned later from observed MAE scale.

`warmupDuration` remains in config, but its semantics must be clarified in the
implementation and docs: it is a startup-only AnLF monitor warmup, not a
post-hot-swap grace period.

#### Project Cleanup

Checkpoint B should remove legacy trigger ownership from `ModelAccuracyStore`:

- remove `consecutiveBreaches`
- remove `emaDeviation`
- remove `emaInitialized`
- remove the associated helper methods

`UpdateDeviation / GetDeviation` may remain temporarily if still useful for
debugging, but they must no longer drive retrain policy.

#### YAML Update

`config/nwdafcfg.yaml` should be migrated in the same checkpoint so the runtime
configuration matches the new implementation. The example config must not retain
dead fields after the cutover.

#### Files Expected To Change

- `pkg/factory/config.go`
- `pkg/factory/model_params_test.go`
- `config/nwdafcfg.yaml`
- `internal/context/accuracy_store.go`
- `internal/context/accuracy_store_test.go`
- `internal/mtlf/trigger.go`
- `internal/mtlf/trigger_test.go`

### B6. Retrain Lifecycle And Compatibility Test Design

#### Goal

Prove that the new report-based decision path is correct, race-safe, and
compatible with the existing retrain / hot-swap lifecycle.

#### Required Test Areas

- state store
  - lazy create per `model + scope`
  - ring-buffer overwrite behavior
  - mean / std / min / max correctness
  - TTL-based GC
- report-based gate
  - low error + high z-score does not trigger
  - high error + low z-score does not trigger
  - both gates pass until the `M-of-N` window is satisfied triggers retrain
  - window hit/miss bookkeeping behaves correctly as old rounds expire
  - one triggering scope does not contaminate a second scope
- cold start
  - before `minBufferSamples`, reports build baseline only and do not trigger
    retrain or breach accumulation
  - after `minBufferSamples`, `relGate` is enforced
  - `minStd` prevents z-score explosion when variance is near zero
  - both-zero rounds are retained in recent history and do not require a
    special eligibility flag
- chronic path
  - `WAPE / NRMSE / MAE` chronic metric selection is parsed correctly
  - `mean / percentile` aggregation works; `percentile=50` behaves as median
  - `minTrafficScale` suppresses chronic decisions in low-volume regions
  - chronic path and degradation path can trigger independently but dispatch
    retrain only once
- startup-only warmup
  - `warmupDuration` is consumed only on the first monitor start after NWDAF
    boot
  - hot-swap restarts must not re-enter warmup or discard fresh post-swap
    predictions under the name of warmup
- processor wiring
  - `SetOnAccuracyReports` is the active MTLF input
  - legacy deviation callback is no longer part of decision tests
- retrain lifecycle
  - in-flight retraining skips further report evaluation
  - failed training clears the retraining guard
  - successful hot-swap deletes old model monitoring state
  - old scope breach state does not leak into the new model URL
- cleanup
  - removed legacy helpers are no longer referenced
  - config defaults and YAML parsing work without removed fields
- concurrency
  - `go test -race ./internal/mtlf/...`

#### Integration Validation Focus

The first runtime validation for Checkpoint B should verify:

- retrain logs show per-scope gate reasoning
- one degraded scope can trigger retrain even if another scope is stable
- post-swap monitoring starts from fresh MTLF state for the new model URL
- post-swap monitor restart does not perform startup warmup again
- the example config and runtime behavior match the documented defaults

## 7. Implementation Order

Recommended order inside Checkpoint B:

1. add the MTLF state-store types and unit tests
2. switch processor wiring to the report-based callback
3. rewrite `internal/mtlf/trigger.go` around per-scope degradation path
4. add `chronic poor-quality path` and `M-of-N` decision-window handling
5. migrate config and remove legacy trigger fields/helpers
6. add retrain lifecycle and race-focused tests

## 8. Expected Branch

Checkpoint B should be implemented on:

```text
feat/retrain-monitoring-degradation-policy
```
