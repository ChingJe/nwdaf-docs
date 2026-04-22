# Checkpoint A Detailed Design

## 1. Purpose

Checkpoint A is the observability phase of the retrain-monitoring work.
Its purpose is to make scope-level accuracy data visible without changing the
current retrain decision behavior.

This phase delivers:

- scope snapshotting on each prediction
- multi-metric computation for each monitoring scope
- an internal `AccuracyReport` contract from AnLF
- CSV output for offline analysis
- compatibility tests proving the legacy sMAPE-triggered retrain path is unchanged

Checkpoint A must remain compatible with the current production logic:

- legacy retrain decision still uses model-level `sMAPE`
- legacy `consecutive` / `ema` behavior stays intact
- no new MTLF decision state is introduced in this phase

## 2. References

- Parent plan: `../NWDAF Retrain Monitoring Implementation Plan.md`
- Workflow and progress: `../NWDAF Retrain Monitoring Workflow and Progress.md`
- Project policy: `.agent/docs/development_policy.md`

## 3. Scope

### In Scope

- A1 `PredictionRecord.ScopeKey` materialization
- A2 candidate metrics: `sMAPE`, `MAE`, `MSE`, `WAPE`, `NRMSE`
- A3 internal AnLF report contract for per-scope metrics
- A4 CSV dumps for metrics and matched pairs
- A5 compatibility and regression tests for legacy decision behavior

### Out of Scope

- MTLF per-scope baseline or ring buffer
- primary-metric decision cutover
- degradation dual-gate / new retrain policy flow
- cold-start policy on the MTLF side
- config cleanup of legacy trigger fields

## 4. Non-Negotiable Constraints

1. Checkpoint A must not change when retraining is triggered.
2. `ScopeKey` must be snapshotted when the prediction is created.
3. Direct SUPI subscriptions and group-based subscriptions must both work.
4. Unsupported or ambiguous target combinations must be skipped explicitly, not silently folded into a model-level aggregate.

## 5. Directory Layout

Checkpoint-specific detailed design files live under:

```text
.agent/docs/plans/mtlf/retrain_monitoring/checkpoints/
  checkpoint_a_detailed_design.md
```

Future checkpoints should follow the same pattern:

```text
checkpoint_b_detailed_design.md
checkpoint_c_detailed_design.md
```

## 6. Detailed Work Design

### A1. Scope Key Materialization

#### Objective

Persist a canonical monitoring-scope identifier into each prediction record so
later monitoring does not depend on live subscription state.

#### Source Of Truth

The preferred source of scope information is the original subscription target:

- `Subscription.EventSubs[*].TgtUe`

The current resource tracking remains useful as a fallback for legacy group
resolution context:

- `NwdafSubResource.OriginalGroupId`
- `NwdafSubResource.Supi`

#### Canonicalization Rules

The canonical string format is:

- `group:<groupId>`
- `groups:<sorted-groupIds>`
- `supi:<supi>`
- `supis:<sorted-supis>`

Rules:

- sort multi-value targets before formatting
- remove empty entries before formatting
- do not mix group and SUPI values into one fallback key unless the original subscription explicitly contains both
- treat `AnyUe` as unsupported in Checkpoint A and skip snapshot creation with a clear log

#### Snapshot Timing

`ScopeKey` is resolved when `PredictionRecord` is created in AnLF, not during
accuracy checking.

#### Data Model Change

Add to `PredictionRecord`:

```go
ScopeKey string
```

#### Helper Design

Introduce a dedicated helper with explicit inputs:

```go
resolveMonitoringScope(sub *context.Subscription) (string, bool)
```

The boolean indicates whether the target can be canonicalized safely.

#### Failure Handling

If the target cannot be canonicalized:

- do not create a fallback model-level key
- skip prediction recording for monitoring purposes
- keep inference response behavior unchanged
- emit a warning log containing `nwdafSubId`

#### Files Expected To Change

- `internal/context/accuracy_store.go`
- `internal/anlf/analytics.go`
- potentially a new helper file under `internal/anlf/`
- tests under `internal/anlf/` and `internal/context/`

### A2. Candidate Metric Computation

#### Objective

Compute multiple candidate metrics per scope while keeping legacy decision logic
on model-level `sMAPE`.

#### Metric Set

- `sMAPE`
- `MAE`
- `MSE`
- `WAPE`
- `NRMSE`

#### Computation Contract

Add a unified entry point:

```go
computeAll(pairs []matchedPair) map[string]float64
```

Design notes:

- `computeSMAPE` remains available for legacy logic
- `WAPE` returns `0` when the denominator is `0`
- `NRMSE` denominator choice must be fixed and documented in tests
- all metric functions must avoid `NaN` and `Inf`

#### Aggregation Strategy

In `checkModelAccuracy`:

1. consume mature predictions
2. match ground truth for each prediction
3. group matched pairs by `ScopeKey`
4. compute all metrics per scope
5. separately compute the legacy model-level `sMAPE` across all matched pairs

The grouped metrics are for observability only in this phase.

### A3. Internal AccuracyReport Contract

#### Objective

Define the internal AnLF output shape that future MTLF policy logic will consume.

#### Compatibility Strategy

Checkpoint A keeps the legacy callback for retrain decision and adds a new
report-oriented callback for observability and future migration.

This avoids forcing a policy rewrite in the same phase.

#### Proposed Types

```go
type AccuracyReport struct {
    ModelURL     string
    ScopeKey     string
    NwdafSubID   string
    Metrics      map[string]float64
    SampleCount  int
    InferenceNum int
    WindowStart  time.Time
    WindowEnd    time.Time
}
```

Optional AnLF callback:

```go
func(modelURL string, reports []AccuracyReport, store *context.ModelAccuracyStore)
```

#### Delivery Rules

- one `AccuracyReport` per scope per monitor round
- reports are emitted only after `minSamples` passes for the legacy path
- report emission must not block or change the existing deviation callback behavior
- if there are no matched pairs for a scope, no report is emitted for that scope

#### Ownership In Checkpoint A

- AnLF owns report production
- MTLF may receive the reports but does not own any new decision logic yet

### A4. CSV Observability Output

#### Objective

Persist enough raw and aggregate data to compare metrics offline and inspect
scope-specific degradation.

#### Files

- `metrics_<ts>.csv`
- `pairs_<ts>.csv`

#### Minimum Metrics CSV Schema For A

Checkpoint A should start with the core columns required to validate scope and
metric behavior:

```text
timestamp,model,scope,metric,current,sampleCount,inferenceNum,windowStart,windowEnd
```

Derived columns such as `mean`, `std`, `zscore`, `cv`, and `slope` belong to
Checkpoint B or later, because baseline state is not introduced in A.

#### Minimum Pairs CSV Schema For A

```text
checkTime,model,scope,nwdafSubId,predictedAt,targetTime,predUl,actualUl,predDl,actualDl
```

#### Write Rules

- create one CSV file pair per NWDAF process start
- use `encoding/csv`
- flush after each monitor round
- close writers when monitor context ends
- when there is no ground truth match, write the pair row with empty actual fields

#### Config Handling In A

Checkpoint A may add CSV config flags now, but must not remove legacy trigger
config in this phase.

### A5. Compatibility Test Design

#### Goal

Prove that the new observability code does not alter existing retrain behavior.

#### Required Test Areas

- scope canonicalization
  - single group
  - multiple groups with sorting
  - single SUPI
  - multiple SUPIs with sorting
  - unsupported target handling
- scope snapshot stability
  - prediction keeps original `ScopeKey` after subscription update or deletion
- metric correctness
  - perfect prediction
  - both-zero cases
  - large error
  - mixed values
- per-scope grouping
  - group scope and SUPI scope do not contaminate each other
- report generation
  - report fields are populated correctly
  - empty-scope report is not emitted
- legacy compatibility
  - old model-level `sMAPE` path still drives retrain decision
  - existing threshold behavior is unchanged
  - existing EMA and consecutive tests continue to pass
- CSV output
  - headers are correct
  - rows are flushed and file handles close cleanly

## 7. Implementation Order

Recommended order inside Checkpoint A:

1. add `ScopeKey` to `PredictionRecord`
2. implement scope canonicalization and snapshotting
3. refactor `checkModelAccuracy` to group by scope
4. add multi-metric computation
5. add `AccuracyReport`
6. add CSV output
7. add compatibility tests

This order keeps the legacy decision path available at every intermediate step.

## 8. Acceptance Criteria

Checkpoint A is ready for review when all of the following are true:

1. `PredictionRecord.ScopeKey` is stored at prediction time.
2. Per-scope metrics are visible in logs and/or tests.
3. `AccuracyReport` can be emitted without changing the retrain trigger path.
4. CSV files are produced with stable headers and valid escaping.
5. Legacy retrain decision remains model-level `sMAPE`.
6. `make build`, `make lint`, and relevant `go test` pass.

## 9. Notes For Checkpoint B

Checkpoint A should intentionally stop before:

- introducing per-scope state on the MTLF side
- replacing the legacy callback as the retrain decision owner
- computing baseline-dependent fields in CSV
- removing old config fields

Those changes belong to Checkpoint B and should build on the contracts defined here.
