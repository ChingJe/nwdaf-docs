# NWDAF Priority 7 Logging Boundary And Hygiene Alignment Plan

Date: 2026-06-29

Status: Implemented on 2026-06-29 in `NWDAF/` commit `2ff07af`, locally verified

Historical remediation item:

- `Priority 7 - Tighten Logging Boundaries`

Current execution note:

- this plan defined the first dedicated implementation round for Priority 7
  - the implemented scope remained intentionally limited to current log
  ownership, verbosity, message shape, and identifier/payload hygiene
- it did not redesign the still-open Priority 6 semantics around
  post-acceptance activation state or late-failure signaling
- where Priority 6 would eventually require additional observability fields or
  new lifecycle states, that later work should extend the cleaned logging
  boundary from this plan rather than bypass it
- the identifier-retention, hot-path verbosity, and consumer-boundary decisions
  below were treated as fixed execution rules during implementation

2026-06-29 implementation outcome:

- touched handler, processor, consumer, notifier, context, AnLF, MTLF, and
  ADRF-retrieval paths were updated to follow the boundary and message-shape
  rules in this plan
- duplicate generic entry logs were removed from the main touched inbound and
  outbound paths
- high-frequency UPF / analytics / notification detail was materially reduced
  at default `Info`
- default-level exposure of `supi`, UE IP, endpoint, callback URL, and model
  URL was materially reduced in the touched paths
- an intermediate local helper `internal/sbi/consumer/http_error.go` was
  added during implementation but removed before completion because other local
  free5GC exemplar NFs did not use an equivalent consumer helper abstraction
  and the stricter alignment direction preferred per-call-site status errors
- the implementation landed in `NWDAF/` commit `2ff07af`
- local verification completed with:
  - `make build`
  - `make lint`
  - `go test ./...`

---

## 1. Purpose

This plan defines the next free5GC-alignment round for the main `NWDAF/`
repository, limited to error-handling and logging behavior in the current code.

The purpose of this round is to align NWDAF more carefully with:

1. the local free5GC development guidance in
   `nwdaf-docs/docs/free5gc/6. free5GC Development Basics.md`
2. the workspace `free5gc-dev-skill/` logging and review guidance
3. the observable logging shape used by local free5GC reference NFs
4. the current app / service / handler / processor / consumer boundaries that
   already exist in `NWDAF/`

This round is intentionally narrower than a full observability redesign.

It is about:

- who owns logging at each call boundary
- where low-level code should wrap and return errors instead of logging
- reducing duplicate and noisy logs
- reducing high-cardinality and payload-heavy default logs
- making severity levels more consistent with whether the process or request
  can continue
- normalizing message style across the main runtime paths

It is not about:

- introducing a new metrics or tracing system
- redesigning Priority 6 subscription activation semantics
- changing HTTP error-contract shape
- rebuilding package ownership boundaries outside the logging problem
- deciding the long-term free5GC integration level for NRF registration,
  metrics server wiring, or broader NF lifecycle behavior

---

## 2. Planning Basis

This plan is based on the following local sources:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project free5GC Alignment Scan Findings.md`
- `nwdaf-docs/docs/free5gc/6. free5GC Development Basics.md`
- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/review-checklist.md`
- `free5gc-dev-skill/references/concurrency-lifecycle.md`
- `free5gc-dev-skill/references/config-and-lifecycle.md`
- `resources/references/free5gc-main/NFs/bsf/internal/sbi/processor/subscriptions.go`
- `resources/references/free5gc-main/NFs/nef/internal/sbi/processor/ti.go`
- `resources/references/free5gc-main/NFs/pcf/internal/sbi/api_httpcallback.go`
- `resources/references/free5gc-main/NFs/pcf/internal/sbi/consumer/notification.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/api_httpcallback.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/api_subscriberdatamanagement.go`
- the current `NWDAF/` tree on 2026-06-29

Primary exemplar interpretation for this plan:

1. `pcf` callback handlers and callback consumer paths show a useful split
   between HTTP-edge logging and terminal outbound notification logging
2. `bsf` and `nef` show that free5GC often logs handler or procedure entry at
   the effective request boundary, but they do so in repo-local shapes rather
   than through one universal pattern
3. `udm` shows that some free5GC NFs log directly in handler packages when
   those handlers are themselves the effective boundary
4. no single exemplar provides a perfect direct match for NWDAF's current
   `handler -> processor -> consumer -> context` layering, so some conclusions
   below are evidence-based inference rather than one-file copying

---

## 3. Logging Rules That This Round Must Satisfy

### 3.1 Local free5GC Development Notes

`nwdaf-docs/docs/free5gc/6. free5GC Development Basics.md` defines the
following baseline:

1. lower-level functions should wrap and return errors with context
2. low-level functions should not log the same error on the way up
3. logging should happen at the highest practical boundary that has the full
   picture of the task
4. logs should be rich in context but not filled with noise
5. severity should help operators prioritize what matters

### 3.2 Workspace Skill Guidance

The workspace `free5gc-dev-skill` adds these directly relevant constraints:

1. preserve the existing NF logger categories
2. wrap lower-level errors with context
3. log at meaningful boundaries such as handlers, lifecycle entrypoints, or
   procedure boundaries
4. avoid duplicate low-level logs
5. keep warning / error / fatal consistent with whether the process can
   continue
6. do not log secrets, subscriber credentials, or full payloads
7. keep `fatal` at lifecycle boundaries, not deep helper functions

### 3.3 Practical Implication For NWDAF

For NWDAF's current architecture, this means:

1. handler, processor, consumer, notifier, AnLF, MTLF, context, and service
   code should not all log the same event in parallel
2. the same request or background job should have one primary log owner per
   boundary
3. low-level parse helpers, transport helpers, and storage helpers should
   prefer error return plus caller context over local `Warnf` / `Errorf`
4. hot-path data-plane or analytics detail should not be emitted by default at
   `Info` unless it is genuinely the operator-facing summary of that path

---

## 4. Corrected Logging Ownership Baseline

The surveyed free5GC references do not show a single universal logging shape.
They show a set of ownership patterns that must be adapted to the touched repo.

### 4.1 Inbound HTTP Boundary

Direct evidence:

1. UDM callback handlers log request-body parse failures at the handler edge
2. PCF callback handlers do the same
3. BSF and NEF log request entry at the layer that directly owns the response

Implication for NWDAF:

- where `NWDAF/` already has a real `internal/sbi/api_*.go` handler layer,
  handler code should remain the primary owner for:
  - request entry logs if one is needed
  - request-body read / deserialize failures
  - top-level translation of returned errors or `ProblemDetails` into HTTP
    responses
- processor code should not repeat generic request-entry logging that the
  handler already emitted

### 4.2 Processor Procedure Boundary

Direct evidence:

1. BSF and NEF processors log procedure entry when that processor is the real
   request owner
2. free5GC processors also log specific business-procedure failures or unusual
   states after parsing is complete

Implication for NWDAF:

- processors should own:
  - procedure-specific failures after the request has crossed the handler edge
  - async follow-up work that is no longer tied to a live HTTP handler
  - summary-level state transitions that the handler cannot see
- processors should not duplicate a generic "received request" or "handle X"
  message when the handler already logged that boundary

### 4.3 Outbound Peer or External Calls

Direct evidence:

1. free5GC consumer code is mixed:
   - some consumer paths log internally because they are effectively terminal
     notification senders
   - other paths primarily return errors
2. PCF outbound notification consumers log internally because they are
   fire-and-forget terminal behavior rather than request/response procedures

Implication for NWDAF:

- for NWDAF consumer methods that return `error` to processor or service code,
  the preferred owner should be the caller's business boundary, not the helper
  that only issued the HTTP request
- consumer-level success and progress logs should be kept only when they are
  the most meaningful boundary for that path
- terminal background senders such as notifier callbacks may still log locally
  because there is no higher synchronous caller that can do so cleanly

### 4.4 Lifecycle Boundary

Direct evidence:

1. the workspace guidance requires `fatal` at lifecycle boundaries only
2. current `NWDAF/` lifecycle `fatal` usage is already concentrated in
   `pkg/service/init.go` and `internal/sbi/server.go`

Implication for NWDAF:

- the current lifecycle `fatal` / panic guard shape should be preserved unless
  a specific misuse is found
- Priority 7 should not widen into a lifecycle rewrite

---

## 5. Current NWDAF State

### 5.1 Logging Topology In The Current Repository

Current loggers are already separated by category in:

- `internal/logger/logger.go`

The main runtime layers emitting logs today are:

1. `pkg/service`
2. `internal/sbi/api_*`
3. `internal/sbi/processor`
4. `internal/sbi/consumer`
5. `internal/notifier`
6. `internal/anlf`
7. `internal/mtlf`
8. `internal/context`
9. `pkg/factory`

The existence of logger categories is not the problem.

The problem is that ownership, verbosity, and message shape are not yet aligned
across those layers.

### 5.2 Confirmed Current Problems

The current code has the following confirmed logging problems.

#### Problem 1. Inbound handler and processor logs duplicate each other

Current code:

- `internal/sbi/api_eventssubscription.go`
- `internal/sbi/processor/eventssubscription.go`
- `internal/sbi/api_collector.go`
- `internal/sbi/processor/smf_notify.go`
- `internal/sbi/processor/upf_notify.go`

Confirmed examples:

1. handler entry plus processor entry both log the same subscription request
2. handler success-path receipt of UPF notification plus processor receipt log
   both log the same inbound event
3. callback handlers already log parse and top-level handling failures, while
   processors also log generic procedure receipt

Why this is a problem:

- it violates the local "log at the appropriate boundary" baseline
- it raises noise without adding new decision context
- it makes it harder to tell which log line is the primary owner of the flow

#### Problem 2. Processor and consumer both log the same outbound work

Current code:

- `internal/sbi/processor/data_collection.go`
- `internal/sbi/consumer/smf_service.go`
- `internal/sbi/consumer/mtlf_service.go`
- `internal/sbi/consumer/ml_service.go`
- `internal/sbi/consumer/adrf_service.go`
- `internal/sbi/consumer/daisy_service.go`

Confirmed examples:

1. `TriggerDataCollection(...)` logs SMF subscription reuse and creation while
   `NsmfService.SubscribeToSmf(...)` also logs subscription start and success
2. MTLF provisioning is logged in both processor and consumer layers
3. ML and Daisy client paths log request start and success inside consumer
   methods even though caller layers also know the same operation

Why this is a problem:

- the current outward call stack already returns errors to higher layers
- duplicate progress logs make it unclear whether processor or consumer owns the
  outbound procedure
- some consumer logs expose more raw endpoint or payload detail than the
  higher-level business boundary actually needs

#### Problem 3. Low-level helper functions log and swallow parse failures

Current code:

- `internal/sbi/processor/upf_notify.go`

Confirmed examples:

1. `parsePacketRate(...)` logs warning and silently returns `0`
2. `parseBitRate(...)` logs warning and silently returns `0`

Why this is a problem:

- helper functions are too deep to know whether a malformed field is expected,
  tolerable, or escalation-worthy
- they violate the local wrap-and-propagate rule
- multiple malformed fields in one notification can generate multiple warnings
  without one coherent higher-level summary

#### Problem 4. Some low-level or support packages log returned errors before
the caller can classify them

Current code:

- `internal/context/db_query.go`
- `internal/context/group_resolver.go`
- parts of `internal/anlf/model.go`
- parts of `internal/sbi/consumer/adrf_service.go`

Confirmed pattern:

1. support-layer functions sometimes log transport, storage, or lookup errors
   and also return them
2. this makes it hard for a higher boundary to own the final message

Why this is a problem:

- it weakens the intended error chain
- it makes later message normalization harder
- it can cause the same failure to appear at context, processor, and handler
  levels

#### Problem 5. Default logs expose too much identifier or payload detail

Current code examples:

- `internal/sbi/consumer/smf_service.go`
- `internal/sbi/consumer/ml_service.go`
- `internal/sbi/consumer/consumer.go`
- `internal/notifier/notifier.go`
- `internal/sbi/processor/upf_notify.go`
- `internal/anlf/model.go`
- `internal/mtlf/training.go`

Confirmed exposure patterns:

1. full `supi` values are logged in multiple default `Info` or `Warn` paths
2. UE IP address is logged in UPF processing and context churn logs
3. full callback URIs, service endpoints, model URLs, and local workflow URLs
   are logged at default levels
4. request JSON or raw response body is logged or embedded in error strings in
   several consumer paths
5. full throughput strings or detailed per-item telemetry values are logged on
   hot paths

Why this is a problem:

- free5GC exemplars do allow some identifiers, but the current NWDAF mix is
  broader and noisier than the direct diagnostic value justifies
- high-cardinality identifiers make logs harder to scan and aggregate
- request and response body logging makes later redaction and boundary cleanup
  harder

#### Problem 6. Hot-path observability data is emitted too aggressively at
`Info`

Current code examples:

- `internal/sbi/processor/upf_notify.go`
- `internal/anlf/analytics.go`
- `internal/anlf/monitor.go`
- `internal/mtlf/trigger.go`
- `internal/notifier/notifier.go`

Confirmed examples:

1. every UPF item can emit `UPF VOLUME` and `UPF THROUGHPUT` logs
2. every ML inference can emit full aggregate slot detail plus inference summary
3. every accuracy-policy evaluation can emit a long `Info` message with many
   computed values
4. every periodic notification success can emit an `Info` line

Why this is a problem:

- this is exactly the kind of noise the local free5GC logging notes warn about
- normal operation can produce large log volume even when nothing is wrong
- critical warnings or errors become harder to see

#### Problem 7. Severity levels drift away from actual operator impact

Current code examples:

- `internal/sbi/processor/data_collection.go`
- `internal/sbi/api_mlmodelprovision.go`
- `internal/sbi/processor/upf_notify.go`
- `internal/mtlf/trigger.go`

Confirmed examples:

1. expected shutdown races or already-deleted async work can appear as
   `Warnf`
2. missing optional or late local state in callback flows is sometimes logged
   as warning without a clear distinction between expected staleness and real
   fault
3. malformed sub-fields inside otherwise-processable notifications are logged as
   warnings at helper depth rather than at one classified boundary

Why this is a problem:

- `warn` should communicate a condition that deserves operator attention
- if expected or tolerated conditions are logged too loudly, genuine warnings
  lose value

#### Problem 8. Message style is inconsistent across layers

Current code examples:

- `SwapModel: ...`
- `UPF VOLUME: ...`
- `Handle CreateSubscription`
- `Processing CreateSubscription request`
- `Async training request accepted: ...`
- `Received UPF notification, items: ...`
- `Subscribing to SMF: endpoint=..., supi=..., notifId=...`

Why this is a problem:

- message verbs, prefixes, casing, and summary level vary by package rather
  than by boundary role
- the same operation can have "handle", "processing", "created", and
  "accepted" style logs spread across several layers
- free5GC itself is not perfectly uniform, but NWDAF can still tighten its own
  local style substantially

Observed style fragmentation inside current `NWDAF/`:

1. some inbound handlers use free5GC-like `Handle <Procedure>` entry logs
2. some callback handlers use `Received ...` wording or log only failures
3. some processors use `Processing ...` while others use `Received ...`
4. some consumers use natural-language transport verbs such as `Subscribing`,
   `Unsubscribing`, or `Initializing`
5. notifier and MTLF paths use procedural narrative wording such as
   `Triggering`, `Submitting`, or `... accepted`

Why this additional observation matters:

- the current inconsistency is not only duplicate ownership; it is also missing
  local style discipline
- without a documented NWDAF-specific message convention, cleanup work may
  remove duplicate logs but still leave the repo with multiple incompatible
  sentence styles
- this round should therefore normalize both boundary ownership and the default
  message shape used at those boundaries

#### Problem 9. Context and store mutation logs are too chatty at default
levels

Current code examples:

- `internal/context/context.go`
- `internal/context/traffic_data.go`

Confirmed patterns:

1. subscription add / update / delete events are emitted at `Info`
2. SMF subscription creation / deletion and resource tracking churn are partly
   emitted at `Info`
3. some CTX-layer logs include correlation IDs, SUPIs, or IPs during normal
   state mutation

Why this is a problem:

- context/store packages are not usually the best operator-facing boundary
- these logs are useful for diagnosis, but more often at `Debug`
- default `Info` should prefer user-visible or procedure-visible summaries

### 5.3 Areas That Are Already Acceptable For This Round

The following areas do not currently require broad Priority 7 redesign:

1. logger category structure in `internal/logger/logger.go`
2. lifecycle `Fatal` ownership in:
   - `pkg/service/init.go`
   - `internal/sbi/server.go`
3. config-load logging in `pkg/factory/config.go`, except for any future body /
   payload hygiene follow-up if the implementation later adds more detail there

---

## 6. Proposed Fixed Decisions For This Round

The following decisions are recommended as defaults for implementation unless a
stronger local reason is found in code review.

### 6.1 Inbound Logging Owner

For flows that already have `internal/sbi/api_*.go` handlers:

1. handler owns HTTP-edge entry logging if one entry log is needed
2. handler owns request-body read and deserialize failure logging
3. handler owns top-level translation of processor failure into HTTP response
4. processor must not repeat generic request-entry or request-success logs that
   the handler already covers

### 6.2 Outbound Logging Owner

For consumer methods that return `error`:

1. processor or service owns the business-level failure log
2. consumer should prefer wrapped returns over progress logging
3. consumer may keep local logs only for:
   - terminal fire-and-forget notifier behavior
   - transport-close debug notes
   - lifecycle or cleanup paths with no higher synchronous caller

### 6.3 Identifier Retention Policy

Decision status:

- fixed for this round on 2026-06-29

Execution policy for this round:

1. keep resource identifiers such as `subscriptionId`, `notifId`,
   `correlationId`, `taskId`, `afID`, and `subID` in default logs where they
   are the primary routing key
2. treat `supi`, UE IP, callback URI, full peer endpoint, and model URL as
   higher-sensitivity or higher-cardinality values:
   - `Info` and `Warn` should prefer summary, reduced precision, or masking
     rather than full-value retention
   - full-value retention should move to `Debug` unless the touched boundary
     truly cannot diagnose the issue without it
   - for `supi` and `groupId`, do not assume masking alone is sufficient;
     summary-only logging is preferred where the identifier is not the primary
     routing key
3. do not emit full request or response bodies by default
4. when an operation can be adequately identified by `subscriptionId`,
   `correlationId`, or `taskId`, prefer those workflow IDs over logging the
   complete subscriber or endpoint identity

### 6.4 Severity Policy

Recommended default policy:

1. `Info` for lifecycle milestones or business summaries
2. `Debug` for high-frequency state churn, per-item telemetry, reusable
   resource churn, and detailed analytics diagnostics
3. `Warn` for tolerated but operator-meaningful anomalies
4. `Error` for failed work items or failed procedure boundaries
5. `Fatal` only at lifecycle boundaries already established today

### 6.5 Message Style Policy

Recommended local style:

1. there is no evidence of one universal cross-NF HTTP logging template that
   NWDAF must copy verbatim; the practical target for this round is strong
   consistency inside NWDAF itself
2. prefer concise action-oriented messages tied to boundary role rather than
   free-form narrative text
3. avoid function-name prefixes such as `SwapModel:` or `runAdrf...:` in
   default log text unless the function name itself is externally meaningful
4. avoid all-caps event banners such as `UPF VOLUME` on default `Info`
5. when possible, log one summary line at the owning boundary instead of many
   step-by-step lines across helper layers
6. prefer a stable local text shape:
   - `<Action>: key=value key=value`
   - if no key/value fields are needed, `<Action>` alone is acceptable
7. do not make raw HTTP method/path the primary default log shape; procedure or
   business-action names are more consistent with the observed free5GC
   references and more useful for NWDAF operators

Boundary-specific convention for this round:

1. handler entry logs:
   - use `Handle <Procedure>`
   - when an identifier is useful, prefer `Handle <Procedure>: sub=<id>` style
     over sentence-like prose
   - callback handlers should follow the same convention rather than a separate
     `Received ...` family
2. handler parse and deserialize failures:
   - free5GC-like request parsing phrases such as `Get Request Body error: ...`
     and `[Request Body] ...` may remain where the code already follows that
     shape
   - new touched paths should keep that boundary-specific error wording concise
     and should not add raw payload dumps
3. processor logs:
   - do not emit generic entry logs when the handler already owns that entry
   - use procedure summary or state-change wording such as
     `CreateSubscription: created sub=<id>`
   - use the same procedure name through success, partial-failure, and cleanup
     summaries instead of switching between `Processing`, `Created`, and
     unrelated helper names
4. consumer logs:
   - use business action names rather than conversational transport phrases
   - prefer shapes such as `CreateSmfSubscription: ...`,
     `DeleteMtlfSubscription: ...`, `LoadMlModel: ...`
   - avoid `Subscribing to ...`, `Initializing ...`, or `Unsubscribing ...`
     when that wording only restates the caller-owned procedure
5. terminal notifier or sender logs:
   - use explicit send/result wording such as `SendNotification: ...`
   - keep these lines as short summaries rather than embedding full URI or
     payload detail
6. hot-path telemetry summaries:
   - do not use banner-style labels such as `UPF VOLUME` or `UPF THROUGHPUT`
     at default `Info`
   - if a summary is retained at `Info`, use one procedure-oriented line such as
     `UpfNotification: processed corr=<id> items=<n>`

### 6.6 Hot-Path Observability Verbosity

Decision status:

- fixed for this round on 2026-06-29

Execution policy for this round:

1. high-frequency telemetry and analytics detail should not remain at default
   `Info` unless the touched log line is the primary operator-facing summary of
   that procedure
2. the following current NWDAF log families should be treated as default
   candidates for `Debug` or summary reduction:
   - per-item UPF volume and throughput detail
   - full ANLF aggregate-slot detail
   - detailed ML inference value dumps
   - detailed MTLF accuracy-policy evaluation state
   - periodic notification success-per-send logs
3. `Info` should be preserved for concise lifecycle or procedure summaries such
   as successful subscription creation, background workflow start/stop,
   retraining trigger, or model-swap completion

### 6.7 Consumer Quietness Rule

Decision status:

- fixed for this round on 2026-06-29

Execution policy for this round:

1. terminal sender paths may log locally because they are the final practical
   boundary for that work item
2. ordinary request/response helper consumers that return `error` should prefer
   wrap-and-return over routine progress logging
3. for touched consumer paths, "subscribing", "created", "deleted", request
   payload, and endpoint logs should only remain where no clearer caller-owned
   business boundary exists

---

## 7. Detailed Implementation Plan

### Workstream A. Establish Boundary Ownership And Remove Duplicates

Targets:

- `internal/sbi/api_eventssubscription.go`
- `internal/sbi/api_collector.go`
- `internal/sbi/api_mlmodelprovision.go`
- `internal/sbi/processor/eventssubscription.go`
- `internal/sbi/processor/smf_notify.go`
- `internal/sbi/processor/upf_notify.go`
- `internal/sbi/processor/data_collection.go`
- `internal/sbi/consumer/*.go`

Changes:

1. keep one owner for generic inbound request entry
2. remove processor-level generic "processing request" logs when the handler
   already logs that boundary
3. remove consumer-level success/progress logs that merely restate processor
   work
4. keep processor logs only when they add business-state context that the
   handler does not know
5. align touched handler entry logs to one `Handle <Procedure>` family
6. replace touched callback `Received ...` entry logs with the same handler
   family where the line is still needed

Expected result:

- one primary log owner per inbound or outbound step
- lower noise without losing fault localization
- touched inbound paths present one consistent handler-entry style

### Workstream B. Replace Helper-Level Warnings With Wrapped Returns

Targets:

- `internal/sbi/processor/upf_notify.go`
- `internal/context/db_query.go`
- selected support helpers in `internal/anlf` and `internal/sbi/consumer`

Changes:

1. refactor local parse helpers such as `parsePacketRate(...)` and
   `parseBitRate(...)` to return error context instead of logging directly
2. log malformed sub-field issues once at the owning UPF item boundary, not
   once per leaf helper
3. where support-layer methods both log and return, remove the lower-level log
   unless it is the final boundary for that path
4. preserve or improve `fmt.Errorf(...: %w)` context so callers still know the
   failing operation

Expected result:

- cleaner error chains
- fewer duplicate warnings
- clearer ownership for malformed-field and transport/storage failures

### Workstream C. Reduce Identifier And Payload Exposure

Targets:

- `internal/sbi/consumer/smf_service.go`
- `internal/sbi/consumer/ml_service.go`
- `internal/sbi/consumer/daisy_service.go`
- `internal/sbi/consumer/consumer.go`
- `internal/notifier/notifier.go`
- `internal/sbi/processor/upf_notify.go`
- `internal/anlf/model.go`
- `internal/mtlf/training.go`
- `internal/context/context.go`
- `internal/context/traffic_data.go`

Changes:

1. replace full request JSON dumps with small summaries or remove them
2. stop embedding full response bodies into user-facing log lines unless they
   are truncated and justified
3. reduce default exposure of:
   - `supi`
   - UE IP
   - callback URI
   - model URL
   - full peer endpoint
4. where a full value is still necessary for debugging, move it to `Debug` or
   a masked format

Expected result:

- default logs remain useful without becoming payload dumps
- high-cardinality identifiers no longer dominate normal operation logs

### Workstream D. Move Hot-Path Detail To Debug Or Summary Form

Targets:

- `internal/sbi/processor/upf_notify.go`
- `internal/notifier/notifier.go`
- `internal/anlf/analytics.go`
- `internal/anlf/monitor.go`
- `internal/mtlf/trigger.go`
- `internal/mtlf/training.go`
- `internal/mtlf/adrf_retrieval.go`

Changes:

1. demote per-item UPF volume / throughput detail from `Info` to `Debug`, or
   replace it with a smaller summary line
2. demote detailed ANLF aggregate-slot and inference-dump logs to `Debug`
3. demote large MTLF accuracy-policy state dumps to `Debug`
4. keep only concise trigger / completion summaries at `Info` or `Warn`
5. review periodic notifier success logs and keep only the minimal summary
   needed at default levels

Expected result:

- default `Info` becomes a small operational summary stream
- detailed telemetry remains available when explicitly requested through log
  level

### Workstream E. Normalize Severity And Message Shape

Targets:

- all touched files in Workstreams A to D

Changes:

1. downgrade expected shutdown races, stale subscription cleanup paths, and
   non-critical fallback conditions where appropriate
2. keep warnings for genuine anomalies that still deserve operator attention
3. standardize message tone toward concise action/result wording
4. remove unnecessary function-name prefixes and banner-style text where a
   plain English action statement is clearer
5. normalize touched logs toward boundary-specific templates:
   - handler: `Handle <Procedure>`
   - processor: `<Procedure>: <result/condition> key=value`
   - consumer: `<Action>: <result/condition> key=value`
   - terminal sender: `SendNotification: <result/condition> key=value`
6. prefer key/value summaries over sentence-style prose where fields are
   present
7. do not introduce raw HTTP method/path as the primary default log format
   during this cleanup round

Expected result:

- warning and error signals carry more real weight
- log text becomes easier to scan across packages
- touched NWDAF boundaries read like one local convention rather than several
  package-specific dialects

### Workstream F. Reclassify Context-Layer Churn Logs

Targets:

- `internal/context/context.go`
- `internal/context/traffic_data.go`
- `internal/context/group_resolver.go`

Changes:

1. keep context initialization or large lifecycle summaries at `Info`
2. move normal mutation churn such as add / update / delete of cached runtime
   state to `Debug` unless it is the primary business summary of the procedure
3. keep invariant-violation or impossible-state logs at warning or error as
   appropriate

Expected result:

- CTX logs remain useful for debugging without crowding normal operation output

---

## 8. Suggested File-Level Prioritization

The implementation should proceed in this order:

1. `internal/sbi/api_eventssubscription.go`
2. `internal/sbi/processor/eventssubscription.go`
3. `internal/sbi/api_collector.go`
4. `internal/sbi/processor/smf_notify.go`
5. `internal/sbi/processor/upf_notify.go`
6. `internal/sbi/processor/data_collection.go`
7. `internal/sbi/consumer/smf_service.go`
8. `internal/sbi/consumer/mtlf_service.go`
9. `internal/sbi/consumer/ml_service.go`
10. `internal/sbi/consumer/daisy_service.go`
11. `internal/sbi/consumer/adrf_service.go`
12. `internal/notifier/notifier.go`
13. `internal/anlf/model.go`
14. `internal/anlf/analytics.go`
15. `internal/anlf/monitor.go`
16. `internal/mtlf/trigger.go`
17. `internal/mtlf/training.go`
18. `internal/mtlf/adrf_retrieval.go`
19. `internal/context/context.go`
20. `internal/context/traffic_data.go`
21. `internal/context/db_query.go`
22. `internal/context/group_resolver.go`

Reason for this order:

1. fix boundary duplication first
2. then fix helper-level error propagation in the same touched procedures
3. then clean high-volume hot paths
4. then clean context and support-package churn

---

## 9. Verification Plan

The implementation round should verify both behavior and log-shape outcomes.

### 9.1 Focused Tests

Run at minimum:

```bash
go test ./internal/sbi
go test ./internal/notifier
go test ./internal/anlf
go test ./internal/mtlf
go test ./internal/context
```

### 9.2 Full Repository Checks

Run after the focused logging cleanup:

```bash
make build
go test ./...
make lint
```

### 9.3 Log-Shape Spot Checks

Use repo-local text scans before and after implementation:

1. scan for request-body dumps such as `string(jsonData)`
2. scan for error strings that embed full response bodies such as `body=%s`
3. scan for high-frequency `Info` banners such as `UPF VOLUME`,
   `UPF THROUGHPUT`, `latest aggregated slot`, and `Accuracy policy`
4. scan for duplicated handler and processor entry logs on the same procedure
5. scan for mixed entry families such as `Handle ...`, `Processing ...`, and
   `Received ...` on the same touched procedure
6. scan for consumer sentence-style transport verbs such as `Subscribing to`,
   `Unsubscribing from`, and `Initializing ... from URL`

### 9.4 Verification Reporting

The implementation note should report:

1. which packages were tested
2. which log-shape scans changed
3. which intentionally retained identifier logs remain and why
4. any residual noisy or exposed logs that were deliberately deferred

---

## 10. Acceptance Criteria

This Priority 7 round should be considered complete only when all of the
following are true:

1. no touched inbound path has duplicate generic request-entry logs in both
   handler and processor layers
2. no touched outbound request/response consumer path logs routine progress at
   two layers without a clear ownership reason
3. helper-level parse or transport functions in the touched paths no longer log
   and silently swallow errors that should be wrapped or classified higher up
4. full request-body dumps are removed from the touched default log paths
5. default-level exposure of `supi`, UE IP, callback URI, model URL, and full
   endpoint data is materially reduced in the touched paths
6. high-frequency per-item telemetry logs in the touched paths are no longer
   emitted at default `Info` unless they are intentionally retained as the
   primary operational summary
7. warning and error levels in the touched paths better match actual operator
   impact
8. lifecycle `Fatal` ownership remains limited to the current accepted
   lifecycle boundaries
9. touched handler, processor, consumer, and terminal sender paths follow one
   documented NWDAF-local message convention instead of mixing `Handle`,
   `Processing`, `Received`, and free-form transport prose for the same
   boundary role
10. the touched packages still pass focused tests, full repository build, full
   Go tests, and lint

### 10.1 Completion Note

Current judgment on 2026-06-29:

- acceptance criteria were satisfied for the current intended Priority 7 scope
- completion was based on the implemented `NWDAF/` workspace state plus local
  verification through `make build`, `make lint`, and `go test ./...`
- the implemented code landed in `NWDAF/` commit `2ff07af`
- this completion judgment does not claim closure of:
  - Priority 6 late-failure signaling semantics
  - broader lifecycle-scope or integration-level decisions outside this plan

---

## 11. Explicit Non-Goals And Follow-Up Boundaries

This plan intentionally does not close the following later work:

1. defining Priority 6 acceptance-state and late-failure signaling semantics
2. adding structured tracing or metrics labels as a separate observability
   layer
3. broader package ownership cleanup under Priority 12
4. broader integration-level decisions under Priority 11
5. future generated-contract or callback redesign work under Priority 10

If later Priority 6 work needs new logs for state transitions such as
"accepted but activation degraded later", those new logs should be added on top
of the cleaner boundary and severity rules established by this Priority 7 plan,
not by restoring the current duplicate or noisy patterns.
