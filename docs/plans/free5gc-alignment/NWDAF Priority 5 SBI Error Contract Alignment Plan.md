# NWDAF Priority 5 SBI Error Contract Alignment Plan

Date: 2026-06-23

Status: Planned

Historical remediation item:

- `Priority 5 — Normalize SBI Error Contracts`

Current execution note:

- after the 2026-06-23 reassessment in
  `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`,
  this is the next intended implementation round even though the historical
  section title remains "Priority 5"

---

## 1. Purpose

This plan defines the next refactor round for the main `NWDAF/` repository,
limited to the HTTP error-contract boundary of the current SBI server.

The purpose of this round is to make NWDAF error responses more clearly aligned
with:

1. local 3GPP OpenAPI YAML under `nwdaf-docs/specs/yaml/`
2. free5GC reference NF handler conventions under
   `resources/references/free5gc-main/NFs/`
3. the current repository's own handler and test structure

This round is intentionally narrow.

It is about:

- HTTP status and error-body shape
- `ProblemDetails` usage
- media type for problem responses
- handler-side parse and required-input failure behavior
- test expectations at the handler boundary

It is not about:

- processor business logic
- late-failure signaling after resource acceptance
- logging cleanup
- app-boundary reconstruction
- model-governance or generated-model replacement
- repository/package ownership changes

---

## 2. Planning Basis

This plan is based on the following local sources:

- `nwdaf-docs/specs/yaml/TS29520_Nnwdaf_EventsSubscription.yaml`
- `nwdaf-docs/specs/yaml/TS29520_Nnwdaf_MLModelProvision.yaml`
- `nwdaf-docs/specs/yaml/TS29520_Nnwdaf_MLModelTraining.yaml`
- `nwdaf-docs/specs/yaml/TS29575_Nadrf_DataManagement.yaml`
- `nwdaf-docs/specs/yaml/TS29571_CommonData.yaml`
- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/sbi-development.md`
- `free5gc-dev-skill/references/openapi-contract.md`
- `resources/references/free5gc-main/NFs/udr/internal/util/problem_json.go`
- `resources/references/free5gc-main/NFs/udr/internal/util/util.go`
- `resources/references/free5gc-main/NFs/nrf/internal/util/problem_json.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/api_httpcallback.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/api_eventexposure.go`
- the current `NWDAF/` tree on 2026-06-23

---

## 3. Official Alignment Baseline

### 3.1 3GPP YAML Baseline

The local YAML snapshots show a consistent contract for standards-based error
responses:

- `TS29520_Nnwdaf_EventsSubscription.yaml` defines `400`, `500`, and `default`
  responses for the subscription resource and its notification callback through
  `TS29571_CommonData.yaml`.
- `TS29520_Nnwdaf_MLModelProvision.yaml` does the same for both the resource
  operations and the `notifUri` callback carrying
  `NwdafMLModelProvNotif` items.
- `TS29520_Nnwdaf_MLModelTraining.yaml` does the same for ML-training
  callbacks.
- `TS29575_Nadrf_DataManagement.yaml` does the same for ADRF
  data-retrieval-subscription callbacks.

`TS29571_CommonData.yaml` is the key shared baseline:

- `400`, `401`, `403`, `404`, `408`, `409`, `410`, `411`, `412`, `413`, `414`,
  `415`, `429`, `500`, `501`, `502`, `503`, and `504` all use
  `application/problem+json`
- their body schema is `ProblemDetails`

This means that for standards-based NWDAF APIs and standards-based callback
receivers, the intended outward-facing error format is not an ad hoc JSON map.
It is a `ProblemDetails`-shaped response aligned with TS 29.571.

### 3.2 free5GC Reference NF Baseline

The free5GC reference NFs do not all use one identical helper, but they do
show a stable direction:

- UDR and NRF provide `GinProblemJson(...)` helpers that write
  `ProblemDetails` and set `Content-Type: application/problem+json`
- UDR utility functions such as `ProblemDetailsSystemFailure(...)` and
  `ProblemDetailsMalformedReqSyntax(...)` centralize common error shapes
- UDM callback handlers such as `api_httpcallback.go` and ordinary handlers
  such as `api_eventexposure.go` build `models.ProblemDetails` for parse and
  system failures instead of returning ad hoc map bodies

Important implication:

- free5GC reference code is not proving that every NF must share one exact
  helper name
- it is proving that standards-facing handler failures should converge on
  `ProblemDetails`, with recognizable titles such as:
  - `Malformed request syntax`
  - `System failure`
  - `Mandatory IEs are missing`

This is the alignment target for NWDAF.

---

## 4. Current NWDAF State

### 4.1 Current Handler Inventory Inside This Boundary

The current error-contract review is limited to:

- `internal/sbi/api_eventssubscription.go`
- `internal/sbi/api_collector.go`
- `internal/sbi/api_mlmodelprovision.go`
- `internal/sbi/api_adrf_notify.go`
- `internal/sbi/api_daisy_callback.go`
- the direct handler tests that currently lock these responses

### 4.2 Current Endpoint Matrix

| Endpoint | Contract Class | Current Parse Path | Current Error Body Style | Current Gap |
| --- | --- | --- | --- | --- |
| `POST /nnwdaf-eventssubscription/v1/subscriptions` | 3GPP NWDAF API | `ShouldBindJSON` | `ProblemDetails` | closest to target, but still returned as ordinary JSON and not through one shared problem writer |
| `PUT /nnwdaf-eventssubscription/v1/subscriptions/{id}` | 3GPP NWDAF API | `ShouldBindJSON` | `ProblemDetails` | same as above |
| `DELETE /nnwdaf-eventssubscription/v1/subscriptions/{id}` | 3GPP NWDAF API | no body parse | processor-returned `ProblemDetails` only | no local helper unifies outbound problem response handling |
| `POST /collector/notify` | peer NF callback | `BindJSON` | `ProblemDetails` | parse path differs from nearby handlers and weakens explicit response control |
| `POST /collector/upf-notify` | peer NF callback | `BindJSON` | `ProblemDetails` | same gap as above |
| `POST /collector/retrieval-notify` | ADRF standard callback | `ShouldBindJSON` | `gin.H{"error": ...}` | not aligned to TS 29.571-style problem response |
| `POST /mlmodel-notify` | 3GPP ML-model-provision callback | `ShouldBindJSON` | `gin.H{"error": ...}` | not aligned to TS 29.571-style problem response |
| `POST /mtlf/training-complete` | local Daisy callback | `ShouldBindJSON` | `gin.H{"error": ...}` | local-only contract and inconsistent with the rest of the server |

### 4.3 Current Problems Confirmed In Code

Confirmed current issues:

1. The same server exposes multiple incompatible error-body shapes.
   Some handlers return `models.ProblemDetails`, while others return
   `gin.H{"error": ...}`.
2. The collector handlers still use `c.BindJSON(...)` while neighboring
   handlers use `c.ShouldBindJSON(...)`.
3. No local shared helper currently owns the outbound problem-response write
   path.
4. Current handler-generated `ProblemDetails` payloads are minimal and do not
   yet intentionally align with the stronger free5GC/reference pattern for
   titles such as `Malformed request syntax` or `System failure`.
5. Current NWDAF handlers do not explicitly advertise
   `application/problem+json` for problem responses.

### 4.4 Current Tests That Lock The Divergence

The current direct handler tests correctly prove the present behavior, but they
also lock in the current inconsistency:

- `api_eventssubscription_test.go` expects `ProblemDetails` for invalid JSON
- `api_collector_test.go` expects `ProblemDetails` for invalid JSON
- `api_mlmodelprovision_test.go` expects a decoded `map[string]string`
  containing `"error": "Invalid notification format"`
- `api_adrf_notify_test.go` expects a decoded `map[string]string`
  containing `"error": "invalid payload"`
- `api_daisy_callback_test.go` expects the same map-style body for invalid
  payload, while the missing-`task_id` case only checks the status code

This is useful because the handler boundary is now directly testable, but it
also means the next refactor must update tests and production code together.

---

## 5. Boundary For This Round

This plan stops at the error-contract boundary.

In scope:

- handler-generated parse errors
- handler-generated missing-required-input errors
- handler-generated internal/system-failure errors
- how processor-returned `ProblemDetails` are written to the client
- content type and response body shape for problem responses
- test updates needed to lock the new contract

Explicitly out of scope:

- redesigning processor return types
- changing subscription procedure semantics
- changing callback success behavior
- changing log categories or reducing log duplication
- moving Daisy callback ownership out of `internal/sbi`
- replacing local ML-model callback structs with generated types
- reworking `pkg/app`, `consumer.NewConsumer()`, or lifecycle ownership

---

## 6. Target Contract For This Round

### 6.1 High-Level Rule

All handler-generated failures in `internal/sbi/api_*.go` should return
`models.ProblemDetails` rather than ad hoc JSON maps.

### 6.2 Endpoint Policy By Class

#### Class A — 3GPP NWDAF API endpoints

Includes:

- `POST /nnwdaf-eventssubscription/v1/subscriptions`
- `PUT /nnwdaf-eventssubscription/v1/subscriptions/{id}`
- `DELETE /nnwdaf-eventssubscription/v1/subscriptions/{id}`

Policy:

- preserve existing success status codes and bodies
- preserve processor-owned business errors
- standardize parse and write behavior around one shared local
  `ProblemDetails` writer

#### Class B — standards-based callback receivers

Includes:

- `POST /mlmodel-notify`
- `POST /collector/retrieval-notify`
- `POST /collector/notify`
- `POST /collector/upf-notify`

Policy:

- align callback failures to `ProblemDetails`
- treat them as real external HTTP contracts, not as internal-only hooks
- remove ad hoc `gin.H` bodies

#### Class C — local integration callback

Includes:

- `POST /mtlf/training-complete`

Policy:

- this endpoint is not a 3GPP NWDAF service operation
- however, it should still align to the same local server-wide problem shape
  unless there is a strong local-integration reason not to
- this plan assumes there is no value in preserving a special Daisy-only
  `{"error": ...}` response format

### 6.3 ProblemDetails Shape Policy

This round should converge on these local rules:

1. Parse failure:
   - status `400`
   - title `Malformed request syntax`
   - detail contains the parse failure context
   - keep a local cause when it already exists and is stable enough to preserve
     without widening scope; otherwise title/detail consistency is more
     important than inventing new causes aggressively
2. Missing required handler-level input:
   - status `400`
   - title should be either `Malformed request syntax` or
     `Mandatory IEs are missing`
   - choose one explicit local rule and use it consistently
3. Handler-level internal/system failure:
   - status `500`
   - title `System failure`
   - cause `SYSTEM_FAILURE`
4. Processor-returned `ProblemDetails`:
   - do not reinterpret them in handlers
   - write them through the same shared problem-response path

### 6.4 Media Type Policy

For error responses that carry `ProblemDetails`, the local writer should set:

- `Content-Type: application/problem+json`

This is the closest direct alignment with `TS29571_CommonData.yaml` and with
the UDR/NRF helper pattern in the reference NFs.

---

## 7. Modification Plan

### Workstream A — Introduce One Shared Problem-Response Writer

Add one local helper in `internal/sbi/` for writing `ProblemDetails`.

Required behavior:

- accept `*models.ProblemDetails`
- write status from `pd.Status`
- serialize the body through Gin
- force `Content-Type: application/problem+json`

Why first:

- it removes repeated response-writing code
- it gives the later handler conversions one stable path

### Workstream B — Normalize Parse Paths

Convert handlers so the parse boundary is explicit and consistent.

Required outcomes:

- replace `BindJSON` in `api_collector.go` with `ShouldBindJSON`
- keep direct handler control over when error responses are written
- do not broaden this round into route or middleware refactors

### Workstream C — Replace Ad Hoc Callback Error Bodies

Convert these handlers away from `gin.H` error bodies:

- `api_mlmodelprovision.go`
- `api_adrf_notify.go`
- `api_daisy_callback.go`

Required outcomes:

- invalid JSON and missing mandatory local fields return `ProblemDetails`
- success responses remain `204 No Content`
- local callback behavior becomes consistent with the rest of the server

### Workstream D — Tighten ProblemDetails Content

Normalize the actual problem payload content used by handlers.

Required outcomes:

- parse failures use a recognizable free5GC-aligned title
- handler-generated `500` responses use `System failure`
- collector internal failures stop using a local `INTERNAL_ERROR` shape when
  `SYSTEM_FAILURE` is the clearer alignment target

Note:

- this round should not try to rewrite every processor-generated cause string
- only handler-originated problems are in scope

### Workstream E — Update Direct Handler Tests

Update current tests to lock the new boundary:

- callback error tests should decode `models.ProblemDetails`, not `map[string]string`
- add assertions for `Content-Type: application/problem+json`
- keep current success-path coverage
- keep failure-path coverage close to the handler boundary

---

## 8. Recommended Implementation Order

1. Add the shared `ProblemDetails` writer helper.
2. Convert `api_collector.go` to `ShouldBindJSON` plus the shared writer.
3. Convert `api_mlmodelprovision.go` and `api_adrf_notify.go`.
4. Convert `api_daisy_callback.go`.
5. Normalize handler-generated titles and causes.
6. Update direct handler tests to match the new contract.
7. Run focused verification before any broader test pass.

Reasoning:

- start with the least controversial structural helper
- then normalize the already-`ProblemDetails` handlers
- then convert the ad hoc callback handlers
- leave the local Daisy variant after the standards-based callback paths are
  settled

---

## 9. Acceptance Criteria

This round is complete when all of the following are true:

1. No handler in `internal/sbi/api_*.go` returns `gin.H{"error": ...}`.
2. Handler-generated parse failures return `models.ProblemDetails`.
3. Handler-generated internal failures return `models.ProblemDetails`.
4. Collector parse paths no longer use `BindJSON`.
5. Error responses produced through the new shared path advertise
   `application/problem+json`.
6. Direct handler tests explicitly prove the new body shape and media type.
7. Success responses and processor-owned business semantics remain unchanged.

---

## 10. Non-Goals And Deferred Follow-Up

This round does not close the following larger topics:

- post-subscription activation and late-failure signaling
- logging cleanup and payload hygiene
- generated-versus-handwritten model governance
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
