# Checkpoint C Detailed Design — Retrain Analysis Report Tool

## 1. Purpose

Checkpoint C needs a lightweight offline tool to inspect runtime experiments
after Checkpoint A/B have produced CSV and policy logs.

The tool should answer:

- how each metric changes over time per model and scope
- when each retrain path becomes eligible
- when each path-specific decision signal becomes true
- how `degradationHits` / `chronicHits` accumulate in the decision window
- which scope/path caused a retrain trigger
- whether hot-swap improved metrics and avoided immediate retrain loops
- whether `trafficScale` is high enough to justify chronic decisions

This is analysis tooling only. It must not be part of NWDAF runtime decision
logic.

## 2. Tool Shape

### 2.1 Recommended Form

Implement a Python offline report generator:

```text
tools/retrain_analysis/retrain_report.py
```

The first version should generate a single self-contained HTML report.

No server mode is required for v1. A static HTML artifact is easier to share,
archive with experiment outputs, and compare across runs.

### 2.2 CLI

Preferred directory-mode CLI:

```bash
python tools/retrain_analysis/retrain_report.py \
  --input .agent/tmp/0422-23 \
  --out .agent/tmp/0422-23/report.html
```

Optional explicit-input CLI:

```bash
python tools/retrain_analysis/retrain_report.py \
  --log .agent/tmp/0422-23/nwdaf.log \
  --metrics .agent/tmp/0422-23/accuracy/metrics_20260422_155554.csv \
  --pairs .agent/tmp/0422-23/accuracy/pairs_20260422_155554.csv \
  --config .agent/tmp/0422-23/nwdafcfg.yaml \
  --out .agent/tmp/0422-23/report.html
```

Directory mode should auto-discover:

- `nwdaf.log`
- `nwdafcfg.yaml`
- `accuracy/metrics_*.csv`
- optionally `accuracy/pairs_*.csv`

If multiple metrics CSV files are present, concatenate them in timestamp order.

## 3. Data Sources

### 3.1 `metrics_*.csv`

Primary source for metric time series.

Expected long-format schema:

```text
timestamp,model,scope,metric,current,sampleCount,inferenceNum,windowStart,windowEnd
```

Important semantics:

- `current` is the current value of one metric for one model/scope/round
- the same timestamp/model/scope may appear once per metric
- charting should filter or pivot by `metric`
- `sampleCount` is matched-pair count for the scope in that monitor round
- `windowStart` / `windowEnd` are the target-time range covered by the matched predictions

### 3.2 `nwdaf.log`

Primary source for policy state and lifecycle events.

The parser should extract:

- `Accuracy policy [...]`
- `Retrain trigger [...]`
- `Async training request accepted`
- `Async training complete`
- `Async training failed`
- `Starting model hot-swap`
- `Model hot-swap completed successfully`
- `Accuracy monitor warmup`
- `Accuracy monitor warmup complete`
- `Accuracy monitor warmup skipped`
- `Not enough samples`

The tool should support both the current log schema and the earlier Checkpoint B
schema so older experiment folders remain readable.

Current policy fields:

```text
degradationEligible
degradationSignal
baselineReady
trafficScale
chronicEligible
chronicSignal
chronicValue
degradationHits
chronicHits
hitReason
```

Legacy aliases:

```text
absGate -> degradationEligible
relGate -> degradationSignal
triggerReason -> hitReason
```

### 3.3 `nwdafcfg.yaml`

Optional source for thresholds and experiment metadata.

Read at least:

- `accuracyMonitor.primaryMetric`
- `accuracyMonitor.fixedFloor`
- `accuracyMonitor.zScoreThreshold`
- `accuracyMonitor.decisionWindowSize`
- `accuracyMonitor.requiredHitsInWindow`
- `accuracyMonitor.minBufferSamples`
- `accuracyMonitor.minStd`
- `accuracyMonitor.chronicPolicy.enabled`
- `accuracyMonitor.chronicPolicy.metric`
- `accuracyMonitor.chronicPolicy.aggregator`
- `accuracyMonitor.chronicPolicy.percentile`
- `accuracyMonitor.chronicPolicy.threshold`
- `accuracyMonitor.chronicPolicy.minTrafficScale`

Missing config should not fail report generation; it should only disable
threshold overlays and display an explicit warning.

### 3.4 `pairs_*.csv`

Optional source for raw prediction/actual drilldown.

Do not make this mandatory in v1. Pairs data can be large and is only needed
when debugging specific metric spikes.

v1 may include a small table with row counts and optionally sampled pairs, but
full drilldown can be deferred.

## 4. Internal DataFrames

Use pandas and normalize data into these tables.

### 4.1 `metrics_df`

```text
timestamp
model
scope
metric
current
sampleCount
inferenceNum
windowStart
windowEnd
```

Parsing requirements:

- parse timestamps into timezone-aware or consistently timezone-naive datetime
- coerce numeric fields with invalid values becoming `NaN`
- preserve full model URL but derive a shorter display label

### 4.2 `policy_df`

```text
timestamp
model
scope
metric
current
mean
std
zscore
degradationEligible
degradationSignal
baselineReady
trafficScale
chronicEligible
chronicSignal
chronicValue
degradationHits
degradationRequired
chronicHits
chronicRequired
hitReason
```

Parsing requirements:

- `degradationSignal` may be true, false, or skipped
- keep skipped as a distinct state rather than coercing it to false
- split `degradationHits=2/3` into `degradationHits=2`,
  `degradationRequired=3`
- split `chronicHits=2/3` similarly
- support legacy aliases listed in §3.2

### 4.3 `events_df`

```text
timestamp
eventType
model
scope
reason
detail
```

Suggested `eventType` values:

- `warmup_start`
- `warmup_complete`
- `warmup_skipped`
- `not_enough_samples`
- `retrain_trigger`
- `training_accepted`
- `training_complete`
- `training_failed`
- `hot_swap_start`
- `hot_swap_complete`

## 5. Metric Semantics To Display

The report should explicitly state current metric semantics so users do not
misread charts.

### 5.1 UL / DL Aggregation

UL and DL are equal channel observations.

Each matched pair contributes:

```text
UL channel: predUl vs actualUl
DL channel: predDl vs actualDl
```

Metrics are aggregated at channel level:

| Metric | Formula |
|--------|---------|
| `MAE` | `sum(abs(errUL) + abs(errDL)) / (pairCount * 2)` |
| `MSE` | `sum(errUL^2 + errDL^2) / (pairCount * 2)` |
| `WAPE` | `sum(abs(errUL) + abs(errDL)) / sum(abs(actualUL) + abs(actualDL))` |
| `NRMSE` | `sqrt(MSE) / mean(abs(actualUL), abs(actualDL))` |

### 5.2 Traffic Scale

`trafficScale` is a channel-level actual-volume magnitude:

```text
trafficScale = sum(abs(actualUL) + abs(actualDL)) / (pairCount * 2)
```

If UPF volume fields are bytes, `trafficScale` is bytes per channel
observation. It is not throughput and not bytes/sec.

Policy logs show the recent-buffer mean used by chronic eligibility, not just a
single raw monitor-round value.

## 6. Report Layout

### 6.1 Experiment Summary

Show:

- input directory
- log time range
- number of models
- number of scopes
- metrics CSV row count
- policy log row count
- retrain trigger count
- training accepted / complete / failed counts
- hot-swap count
- config threshold summary

### 6.2 Global Event Timeline

Show lifecycle events as markers:

- warmup start / complete / skipped
- not enough samples
- retrain trigger
- async training accepted / complete / failed
- hot-swap start / complete

The timeline should make it obvious whether hot-swap is followed by startup
warmup or by `warmup skipped`.

### 6.3 Metric Time Series

Interactive charts should support filtering by:

- model
- scope
- metric

Plot `current` over time.

Threshold overlays:

- if selected metric is `primaryMetric`, draw `fixedFloor`
- if selected metric is chronic metric, draw `chronicPolicy.threshold`

Hover should show:

- full timestamp
- model URL
- scope
- metric
- current
- sampleCount
- inferenceNum
- windowStart / windowEnd

### 6.4 Policy State Timeline

For each selected model/scope, show:

- `baselineReady`
- `degradationEligible`
- `degradationSignal`
- `chronicEligible`
- `chronicSignal`
- `degradationHits`
- `chronicHits`
- `hitReason`

Boolean states can be visualized as step charts or heatmap rows.

`degradationSignal=skipped` should be shown as a third state, e.g. gray.

### 6.5 Chronic Detail

Show:

- `trafficScale` over time
- `chronicValue` over time
- `minTrafficScale` threshold line
- `chronicPolicy.threshold` line
- chronic hits in decision window

This view should answer whether chronic triggers happened because of sustained
poor quality under meaningful traffic or because thresholds are too permissive.

### 6.6 Trigger Explanation

For each retrain trigger, show:

- trigger time
- model
- scope
- reason
- current primary metric
- `degradationHits / required`
- `chronicHits / required`
- previous `decisionWindowSize` policy rows for the same model/scope
- nearest metric time-series values around the trigger

This section is the main debugging aid for "why did retrain happen?".

## 7. Implementation Dependencies

Recommended Python dependencies:

- `pandas`
- `plotly`
- `jinja2`
- `pyyaml`

Keep the tool lightweight. Avoid a frontend build chain in v1.

The generated HTML may use embedded Plotly JS for portability. If file size
becomes an issue, add an option later to reference Plotly from CDN.

## 8. Parser Design

### 8.1 Log Parsing

Use regular expressions targeted at known message bodies rather than attempting
to parse every log line.

Policy line pattern should extract the message inside:

```text
msg="Accuracy policy [...] ..."
```

Then parse key-value tokens from the message.

Required behavior:

- tolerate extra unknown fields
- tolerate missing fields by filling `NaN` / `None`
- support current and legacy policy field names
- preserve `skipped` string for tri-state signals

### 8.2 Model Display Labels

Model URLs are long. Derive stable display labels:

- `initial` for `http://placeholder/download/initial`
- last path segment for Daisy download URLs
- fallback to `model-<index>` when the path segment is empty

Keep full URL in hover/detail text.

## 9. Validation Plan

Use `.agent/tmp/0422-23` as the first fixture-like manual validation case.

Expected observations:

- exactly one retrain trigger
- one async training accepted event
- one async training complete event
- one hot-swap start and complete
- first model triggers by chronic path
- hot-swap monitor shows `warmup skipped`
- new model does not immediately retrigger within the captured window
- WAPE/chronicValue decreases after swap

Tool validation should include:

- generated HTML exists
- metrics rows are loaded
- policy rows are loaded
- retrain trigger appears in summary and timeline
- trigger explanation identifies chronic path
- legacy log aliases can still parse older `absGate / relGate / triggerReason`
  logs if available

## 10. Out Of Scope For v1

- live web server mode
- database integration
- Prometheus/Grafana export
- automatic threshold recommendation
- comparing multiple experiment directories
- full raw-pair drilldown UI
- editing config from the report

These can be added after the single-report workflow proves useful.

## 11. Suggested Implementation Order

1. Create CLI and input discovery.
2. Parse metrics CSV into `metrics_df`.
3. Parse policy and lifecycle log lines into `policy_df` and `events_df`.
4. Parse optional config thresholds.
5. Generate a plain HTML summary table first.
6. Add Plotly metric time-series charts.
7. Add policy state timeline.
8. Add chronic detail view.
9. Add trigger explanation section.
10. Validate against `.agent/tmp/0422-23`.

