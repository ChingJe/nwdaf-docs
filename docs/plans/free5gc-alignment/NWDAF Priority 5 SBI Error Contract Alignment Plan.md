# NWDAF Priority 5 SBI Error Contract Alignment Plan

Date: 2026-06-23

Status: Implemented for covered scope

Historical remediation item:

- `Priority 5 — Normalize SBI Error Contracts`

2026-06-23 baseline correction:

- the first draft of this plan leaned too quickly from YAML-level intent to a
  local helper shape
- the corrected plan below is re-based on actual free5GC NF implementation
  patterns, plus the current `github.com/free5gc/openapi` dependency snapshot
  used by `NWDAF/`

Current execution note:

- after the 2026-06-23 reassessment in
  `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`,
  this remains the next intended implementation round
- however, the round is now split into outward contract alignment first and
  parse/model-path alignment second
- Phase A outward contract alignment is now implemented in the current
  `NWDAF/` tree
- the currently covered Phase B parse/model alignment work is now also
  implemented in the current `NWDAF/` tree
- residual work is limited to intentionally excluded callbacks or future
  generated-contract availability

2026-06-23 implementation update:

- Phase A is closed in code for the reviewed handler boundary
- the current implementation now:
  - returns `models.ProblemDetails` instead of map-style `"error"` bodies
  - writes problem responses through an NF-local
    `internal/util/GinProblemJson(...)` helper
  - sets `Content-Type: application/problem+json`
  - uses shared `openapi.ProblemDetailsMalformedReqSyntax(...)` and
    `openapi.ProblemDetailsSystemFailure(...)` helpers for the common
    handler-originated cases
- focused verification run:
  - `go test ./internal/sbi`
- Phase B is now implemented for the covered endpoint set in this batch

2026-06-23 Phase B implementation update:

- completed handler-side changes:
  - `api_eventssubscription.go` create/update now parse through
    `GetRawData() + openapi.Deserialize(...)`
  - `api_collector.go` SMF callback now parses through
    `GetRawData() + openapi.Deserialize(...)`
  - `api_mlmodelprovision.go` now uses generated
    `[]models.NwdafMlModelProvNotif` instead of NWDAF-local duplicate structs
  - `api_adrf_notify.go` now reuses generated nested
    `models.FetchInstruction` while keeping a local top-level callback type
- explicitly not covered in this batch:
  - `POST /collector/upf-notify`
  - `POST /mtlf/training-complete`
  - an exact generated top-level ADRF retrieval callback model
- reasons:
  - `upf-notify` still depends on NWDAF-local `processor.UpfNotificationData`
    and no matching generated `openapi/models` type has been confirmed in the
    current dependency snapshot
  - Daisy callback is a project-local integration path rather than a verified
    free5GC standards-facing baseline
  - ADRF retrieval callback still lacks an exact generated top-level model in
    `github.com/free5gc/openapi v1.2.3`, so only partial alignment is safe
- focused verification runs:
  - `go test ./internal/sbi`
  - `make build`

---

## 1. Purpose

This plan defines the next refactor round for the main `NWDAF/` repository,
limited to the HTTP error-contract boundary of the current SBI server.

The purpose of this round is to align NWDAF more carefully with:

1. local Release 18 3GPP OpenAPI YAML under `nwdaf-docs/specs/openapi/`
2. actual free5GC reference NF implementations under
   `resources/references/free5gc-main/NFs/`
3. the current `github.com/free5gc/openapi v1.2.3` dependency used by
   `NWDAF/`

This round is intentionally narrower than the first draft had implied.

It is about:

- HTTP status and error-body shape
- `ProblemDetails` usage
- media type for problem responses
- choosing the correct free5GC-aligned response-writing primitives
- clarifying which endpoints can realistically move toward
  `openapi.Deserialize(...)` in the current dependency snapshot

It is not about:

- processor business logic
- late-failure signaling after resource acceptance
- logging cleanup
- app-boundary reconstruction
- repository/package ownership changes

---

## 2. Planning Basis

This plan is based on the following local sources:

- `nwdaf-docs/specs/openapi/TS29520_Nnwdaf_EventsSubscription.yaml`
- `nwdaf-docs/specs/openapi/TS29520_Nnwdaf_DataManagement.yaml`
- `nwdaf-docs/specs/openapi/TS29520_Nnwdaf_MLModelProvision.yaml`
- `nwdaf-docs/specs/openapi/TS29520_Nnwdaf_MLModelTraining.yaml`
- `nwdaf-docs/specs/openapi/TS29571_CommonData.yaml`
- `nwdaf-docs/specs/openapi/TS29575_Nadrf_DataManagement.yaml`
- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/sbi-development.md`
- `free5gc-dev-skill/references/openapi-contract.md`
- `resources/openapi/openapi/problem_details.go`
- `resources/references/free5gc-main/NFs/udr/internal/util/problem_json.go`
- `resources/references/free5gc-main/NFs/udr/internal/util/util.go`
- `resources/references/free5gc-main/NFs/nrf/internal/util/problem_json.go`
- `resources/references/free5gc-main/NFs/nssf/internal/util/problem_json.go`
- `resources/references/free5gc-main/NFs/nef/internal/sbi/api_callback.go`
- `resources/references/free5gc-main/NFs/amf/internal/sbi/api_httpcallback.go`
- `resources/references/free5gc-main/NFs/smf/internal/sbi/api_pdusession.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/api_httpcallback.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/api_eventexposure.go`
- the current `NWDAF/` tree on 2026-06-23

---

## 3. Corrected Alignment Baseline

### 3.1 YAML Contract Baseline

The local YAML snapshots still establish the outward contract target:

- standards-based NWDAF APIs and callback receivers use
  `application/problem+json`
- their error body schema is `ProblemDetails`

This remains the standards baseline, but it does not by itself decide which
local helper or parse path NWDAF should use.

### 3.2 free5GC Implementation Baseline

The actual free5GC implementation pattern is more specific:

1. Shared `ProblemDetails` constructors already exist in the common
   `openapi` module.
   `resources/openapi/openapi/problem_details.go` provides:
   - `ProblemDetailsSystemFailure(...)`
   - `ProblemDetailsMalformedReqSyntax(...)`
   - other generic helpers
2. Several NFs add only a very thin NF-local writer under `internal/util/`.
   Examples:
   - UDR `internal/util/problem_json.go`
   - NRF `internal/util/problem_json.go`
   - NSSF `internal/util/problem_json.go`
   These helpers primarily set `Content-Type: application/problem+json` and
   write the provided `ProblemDetails`.
3. Many standards-facing handlers use:
   - `GetRawData()`
   - then `openapi.Deserialize(...)`
   Examples:
   - UDM `api_httpcallback.go`
   - UDM `api_eventexposure.go`
   - AMF `api_httpcallback.go`
   - NEF `api_callback.go`
4. free5GC does not use one universal parse path.
   Some handlers, such as SMF `api_pdusession.go`, still use
   `ShouldBindJSON(...)` for certain request forms.
   Therefore parse-path alignment must be decided endpoint by endpoint.
5. For malformed request syntax, free5GC commonly uses title
   `Malformed request syntax`.
   The shared `openapi.ProblemDetailsMalformedReqSyntax(...)` helper does not
   set a custom cause like `INVALID_JSON`.
6. For system failures, free5GC commonly uses title `System failure` with
   cause `SYSTEM_FAILURE`.
7. `MANDATORY_IE_MISSING` exists in free5GC, but it often appears at the
   processor or deeper validation boundary rather than as a generic parse
   helper result.

### 3.3 Corrected Implications For NWDAF

The real free5GC-aligned target is therefore:

- reuse shared `openapi.ProblemDetails...` constructors where they already
  exist
- if NWDAF wants a problem-response writer helper, place it in an NF-local
  utility package, not as a new `internal/sbi`-local family
- treat outward response-shape alignment and parse/model-path alignment as two
  separate decisions
- do not let Daisy local callback behavior define the standards-facing
  alignment target

---

## 4. Current NWDAF State

2026-06-23 state note after Phase A:

- the table below reflects the baseline that informed the split between Phase A
  and Phase B
- after the current implementation round, the major response-shape gaps listed
  there are now closed
- the remaining live work in this batch is primarily parse/model alignment
  rather than outward problem-body normalization

### 4.1 Current Handler Inventory Inside This Boundary

The current review is limited to:

- `internal/sbi/api_eventssubscription.go`
- `internal/sbi/api_collector.go`
- `internal/sbi/api_mlmodelprovision.go`
- `internal/sbi/api_adrf_notify.go`
- `internal/sbi/api_daisy_callback.go`
- the direct handler tests that lock these responses

### 4.2 Current Endpoint Matrix

| Endpoint | Contract Class | Current Parse Path | Current Response Shape Gap | Parse/Model Alignment Note |
| --- | --- | --- | --- | --- |
| `POST /nnwdaf-eventssubscription/v1/subscriptions` | 3GPP NWDAF API | `ShouldBindJSON` | already returns `ProblemDetails`, but not through a confirmed free5GC-style writer path | exact generated model exists; a later move toward `openapi.Deserialize(...)` is realistic |
| `PUT /nnwdaf-eventssubscription/v1/subscriptions/{id}` | 3GPP NWDAF API | `ShouldBindJSON` | same as above | exact generated model exists |
| `DELETE /nnwdaf-eventssubscription/v1/subscriptions/{id}` | 3GPP NWDAF API | no body parse | processor-returned `ProblemDetails` are not yet written through one confirmed path | no parse refactor needed |
| `POST /collector/notify` | peer NF callback | `BindJSON` | outward error contract is close, but still not on a verified free5GC-style write path | exact generated model exists |
| `POST /collector/upf-notify` | peer NF callback | `BindJSON` | same as above | payload is local, not current openapi-generated |
| `POST /collector/retrieval-notify` | ADRF standard callback | `ShouldBindJSON` | still returns ad hoc `gin.H` on parse failure | exact ADRF callback type is not present in the current dependency snapshot |
| `POST /mlmodel-notify` | 3GPP ML-model-provision callback | `ShouldBindJSON` | still returns ad hoc `gin.H` on parse failure | exact generated model exists as `models.NwdafMlModelProvNotif` |
| `POST /mtlf/training-complete` | local Daisy callback | `ShouldBindJSON` | still returns ad hoc `gin.H` | local integration path; not a free5GC standards-facing baseline |

### 4.3 Current Problems Confirmed In Code

Confirmed current issues:

1. The same server exposes multiple incompatible error-body shapes.
2. NWDAF has not yet chosen one verified free5GC-style problem-response path.
3. Response-shape cleanup and parse/model-path cleanup had been treated as one
   item, but they are not the same refactor.
4. Some endpoints already have exact generated-model coverage in the current
   dependency snapshot, while others do not.
5. Daisy callback behavior is local and should not drive official-alignment
   decisions.

---

## 5. Boundary For This Round

This batch should be split into two phases.

### Phase A — Outward ProblemDetails Contract Alignment

In scope:

- handler-generated parse errors
- handler-generated internal/system-failure errors
- how processor-returned `ProblemDetails` are written to the client
- `Content-Type: application/problem+json`
- replacing ad hoc `gin.H{"error": ...}` bodies
- updating direct handler tests for the chosen outward contract

### Phase B — Parse / Model Alignment Follow-Up

In scope only after Phase A:

- deciding endpoint by endpoint whether to move toward
  `GetRawData() + openapi.Deserialize(...)`
- switching to exact generated models where the current dependency snapshot
  already provides them
- deferring endpoints that would blur into broader model-governance work

Explicitly out of scope for Phase A:

- redesigning processor return types
- changing subscription procedure semantics
- changing callback success behavior
- moving Daisy callback ownership out of `internal/sbi`
- broad generated-code or OpenAPI-regeneration work
- reworking `pkg/app`, `consumer.NewConsumer()`, or lifecycle ownership

---

## 6. Execution Target

### 6.1 Phase A Target

All handler-generated failures in the listed files should return
`models.ProblemDetails` rather than ad hoc JSON maps.

The preferred free5GC-style primitive stack is:

- shared `openapi.ProblemDetails...` constructors where available
- optional NF-local writer under `internal/util/` if NWDAF needs one shared
  place to force `application/problem+json`

This phase should not introduce a new `internal/sbi`-local helper family for
problem responses.

### 6.2 Phase B Target

Only after Phase A is stable:

- standard endpoints with exact generated types may move toward
  `GetRawData() + openapi.Deserialize(...)`
- endpoints without an exact generated type may stay on local structs for now
  or be deferred

Phase B non-coverage that should be tracked explicitly:

- `POST /collector/upf-notify`
  - reason: the current NWDAF payload uses local
    `processor.UpfNotificationData`, and no matching
    `github.com/free5gc/openapi/models` type has been confirmed in the current
    dependency snapshot
  - implication: it remains inside the Phase A outward error-contract baseline
    but is not part of the official parse/model alignment batch
- `POST /mtlf/training-complete`
  - reason: this is a Daisy local integration callback rather than a verified
    free5GC standards-facing NF baseline
  - implication: keep the outward `ProblemDetails` behavior aligned, but do not
    let it drive Phase B parse/model decisions
- `POST /collector/retrieval-notify`
  - reason: the current dependency snapshot does not provide an exact
    generated top-level ADRF retrieval callback model
  - implication: Phase B may only apply partial alignment there, such as using
    generated nested types where available, while retaining a local top-level
    request struct

### 6.3 Endpoint Policy By Class

#### Class A — 3GPP NWDAF API endpoints

Includes:

- `POST /nnwdaf-eventssubscription/v1/subscriptions`
- `PUT /nnwdaf-eventssubscription/v1/subscriptions/{id}`
- `DELETE /nnwdaf-eventssubscription/v1/subscriptions/{id}`

Policy:

- preserve success-path behavior
- preserve processor-owned business errors
- align the outward problem writer first
- treat parse-path migration as a later sub-step

#### Class B — standards-based callback receivers

Includes:

- `POST /mlmodel-notify`
- `POST /collector/retrieval-notify`
- `POST /collector/notify`
- `POST /collector/upf-notify`

Policy:

- align callback failures to `ProblemDetails`
- remove ad hoc `gin.H` bodies
- only migrate parse/model paths where the current dependency snapshot supports
  it clearly

#### Class C — local integration callback

Includes:

- `POST /mtlf/training-complete`

Policy:

- keep it consistent with the server-wide outward problem shape
- do not use it as the baseline for official free5GC alignment decisions

---

## 7. Recommended Modification Plan

### Workstream A — Lock The Correct free5GC Primitive Choice

Before touching more code, the implementation target should be fixed as:

- use `openapi.ProblemDetailsMalformedReqSyntax(...)` and
  `openapi.ProblemDetailsSystemFailure(...)` where they fit
- if NWDAF needs one writer, place it in `internal/util/`
- avoid introducing a new `internal/sbi`-specific helper family for this
  concern

### Workstream B — Execute Phase A First

Normalize the outward response contract in:

- `api_eventssubscription.go`
- `api_collector.go`
- `api_mlmodelprovision.go`
- `api_adrf_notify.go`
- `api_daisy_callback.go`

Required outcomes:

- no handler returns map-style `"error"` bodies
- problem responses use `application/problem+json`
- handler-generated `500` problems use `System failure` / `SYSTEM_FAILURE`
- parse failures use `Malformed request syntax`
- do not add a new custom malformed-JSON cause unless NWDAF explicitly needs
  that divergence and documents why

### Workstream C — Scope Phase B Endpoint By Endpoint

After Phase A:

- `api_eventssubscription.go`: exact generated model exists, so a later move
  toward `openapi.Deserialize(...)` is realistic
- `api_collector.go`: SMF callback has exact generated coverage; UPF callback
  remains local
- `api_mlmodelprovision.go`: exact generated model exists as
  `models.NwdafMlModelProvNotif`
- `api_adrf_notify.go`: exact ADRF callback type is not present in the current
  dependency snapshot; keep local type or defer
- `api_daisy_callback.go`: local integration path; keep out of the official
  parse/model baseline

Current Phase B execution intent:

- migrate `api_eventssubscription.go` create/update handlers to the
  `GetRawData() + openapi.Deserialize(...)` path while preserving the current
  processor contract
- migrate `api_collector.go` SMF callback parsing to the same path
- replace NWDAF-local ML model provision notification structs with generated
  `openapi/models` types
- keep `api_collector.go` UPF callback on its existing local payload contract
- apply only partial alignment in `api_adrf_notify.go` because only the nested
  `FetchInstruction` type is generated in the current dependency snapshot

### Workstream D — Update Direct Handler Tests

Only after the above target is fixed:

- callback error tests should decode `models.ProblemDetails`
- tests should assert `Content-Type: application/problem+json`
- success-path behavior should stay unchanged

---

## 8. Recommended Implementation Order

1. Freeze the corrected target primitive choice.
2. Complete Phase A across standards-facing handlers.
3. Update handler tests to lock the outward contract.
4. Re-assess Phase B separately with generated-model availability in hand.

Reasoning:

- the first urgent gap is inconsistent outward HTTP contract
- parse/model alignment is real, but it is not one-size-fits-all
- forcing parse-path uniformity too early would blur into OpenAPI/model
  governance work

---

## 9. Acceptance Criteria

Phase A is complete when all of the following are true:

1. No handler in `internal/sbi/api_*.go` returns `gin.H{"error": ...}`.
2. Handler-generated parse failures return `models.ProblemDetails`.
3. Handler-generated internal failures return `models.ProblemDetails`.
4. Error responses produced through the chosen shared path advertise
   `application/problem+json`.
5. The implementation path matches an actually verified free5GC style:
   shared `openapi` constructors plus, if chosen, an NF-local utility writer.
6. Direct handler tests explicitly prove the new body shape and media type.
7. Success responses and processor-owned semantics remain unchanged.

Phase A status on 2026-06-23:

- complete in the current `NWDAF/` tree
- verified with `go test ./internal/sbi`

Phase B completion target:

1. `api_eventssubscription.go` create/update parse through
   `GetRawData() + openapi.Deserialize(...)`.
2. `api_collector.go` SMF callback parse through
   `GetRawData() + openapi.Deserialize(...)`.
3. `api_mlmodelprovision.go` uses generated
   `[]models.NwdafMlModelProvNotif` instead of NWDAF-local duplicate structs.
4. `api_adrf_notify.go` keeps a local top-level callback type but reuses
   generated nested types where possible.
5. The plan explicitly records uncovered endpoints and the reason each one
   stays outside the official parse/model alignment batch.

Phase B is intentionally not a completion gate for the first outward-contract
round and is now the active remaining portion of this batch.

---

## 10. Deferred Follow-Up

This batch does not by itself close:

- post-subscription activation and late-failure signaling
- logging cleanup and payload hygiene
- broader generated-versus-handwritten model governance
- app-boundary reconstruction
- Daisy/MTLF ownership relocation out of `internal/sbi`
- full free5GC integration-level decisions such as NRF registration or metrics

Those items should remain in their own later remediation rounds.

---

## 11. Verification Plan

Minimum targeted verification for this round:

```bash
go test ./internal/sbi/... -run 'TestHandle(CreateSubscription|UpdateSubscription|CollectorNotify|UpfNotify|MlModelProvisionNotify|AdrfRetrievalNotify|DaisyTrainingComplete)'
make build
go test ./...
```

Recommended additional check:

```bash
make lint
```

Interpretation rules:

- handler test pass proves the HTTP error-contract boundary only
- full `go test ./...` proves repository compatibility at unit/module level
- none of these checks prove end-to-end cross-NF callback interoperability

---

## 12. Implementation Notes For The Next Round

The next implementation round should prefer the smallest coherent change:

- do not mix this with app-boundary cleanup
- do not move files between packages
- do not replace local ML-model callback structs in the same change
- do not redesign Daisy payload semantics in the same change

The intended output of this round is a cleaner, more official-looking HTTP
error surface that makes later refactors safer.
