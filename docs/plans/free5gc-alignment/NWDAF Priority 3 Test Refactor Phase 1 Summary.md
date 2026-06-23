# NWDAF Priority 3 Test Refactor Phase 1 Summary

Date: 2026-06-23

Status: Phase 1 Completed

Related plan:

- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 3 Test Refactor Detailed Plan.md`

---

## 1. Phase 1 Scope

This phase completed the highest-value Priority 3 work needed to make later
alignment and contract cleanup safer:

- add direct handler coverage under `internal/sbi/`
- introduce `gock` and `gomock` where they provide the clearest seam value
- normalize the most important consumer tests
- reduce processor tests that carried transport setup
- improve notifier cancellation determinism
- remove obsolete in-repo `NWDAF/docs/` and `NWDAF/test/` material that no
  longer reflects the intended repository shape

This phase does **not** mean the entire original Priority 3 ideal end state is
 complete. It means the first implementation stage is complete and the remaining
work is now narrower and lower-risk.

---

## 2. Phase 1 Outcome

Completed in `NWDAF/` on branch `refactor/free5gc-alignment`:

- handler tests added for:
  - `internal/sbi/api_eventssubscription.go`
  - `internal/sbi/api_collector.go`
  - `internal/sbi/api_mlmodelprovision.go`
  - `internal/sbi/api_daisy_callback.go`
  - `internal/sbi/api_adrf_notify.go`
- `gock` added and used for standardized consumer tests
- `gomock` added and used for handler/processor-facing seams where introduced
- `NmtlfService` migrated to generated OpenAPI client ownership with client
  caching keyed by endpoint
- `NsmfService` left on raw HTTP with explicit replacement test coverage because
  the current local `openapi` dependency snapshot does not safely model the
  required request fields
- processor tests refactored away from real local HTTP servers for key SMF
  collection and update reconciliation paths
- notifier cancellation test made more deterministic
- stale low-value test `TestEmbeddedMethodPromotion` removed and replaced by
  behavior-oriented delegation proof
- obsolete `NWDAF/docs/` and `NWDAF/test/` trees removed

---

## 3. Verification

Commands run for this phase:

```bash
make build
go test ./...
make lint
go test ./internal/sbi/... -cover
go test ./internal/notifier ./internal/context -cover
```

Result:

- build passed
- full Go test suite passed
- lint passed

---

## 4. Boundary Matrix

This matrix is the explicit Phase 1 ownership snapshot requested by the
Priority 3 plan.

| Production Area | Intended Test Seam | Current Test Owners | Phase 1 Status |
| --- | --- | --- | --- |
| `internal/sbi/api_eventssubscription.go` | HTTP handler | `internal/sbi/api_eventssubscription_test.go` | Completed |
| `internal/sbi/api_collector.go` | HTTP handler | `internal/sbi/api_collector_test.go` | Completed |
| `internal/sbi/api_mlmodelprovision.go` | HTTP handler | `internal/sbi/api_mlmodelprovision_test.go` | Completed |
| `internal/sbi/api_daisy_callback.go` | HTTP handler | `internal/sbi/api_daisy_callback_test.go` | Completed |
| `internal/sbi/api_adrf_notify.go` | HTTP handler | `internal/sbi/api_adrf_notify_test.go` | Completed |
| `internal/sbi/consumer/smf_service.go` | Outbound consumer | `internal/sbi/consumer/smf_service_test.go` | Completed with raw-HTTP exception |
| `internal/sbi/consumer/mtlf_service.go` | Outbound consumer | `internal/sbi/consumer/mtlf_service_test.go` | Completed |
| `internal/sbi/consumer/ml_service.go` | Outbound consumer | `internal/sbi/consumer/ml_service_test.go` | Existing tests kept; not yet normalized |
| `internal/sbi/consumer/daisy_service.go` | Outbound consumer | `internal/sbi/consumer/daisy_service_test.go` | Existing tests kept; not yet normalized |
| `internal/sbi/consumer/adrf_service.go` | Outbound consumer | none yet | Deferred to Phase 2 |
| `internal/sbi/processor/data_collection.go` | Procedure/state logic | `internal/sbi/processor/data_collection_test.go` | Key transport-heavy cases cleaned up |
| `internal/sbi/processor/eventssubscription.go` | Procedure/state logic | `internal/sbi/processor/eventssubscription_test.go` | Key reconciliation path strengthened |
| `internal/notifier/notifier.go` | Lifecycle/cancellation | `internal/notifier/notifier_test.go` | Improved determinism |

Classification after Phase 1:

- unit-level handler seams:
  - `internal/sbi/api_*_test.go`
- unit-level consumer seams:
  - `internal/sbi/consumer/*_test.go`
- unit-level processor seams:
  - `internal/sbi/processor/*_test.go`
- lifecycle seams:
  - `internal/notifier/*_test.go`
- integration-oriented assets:
  - removed from `NWDAF/` in this phase because the old local `test/` tree was
    no longer treated as the canonical default testing story

---

## 5. Coverage Snapshot

Coverage numbers are support signals only. They are included here because the
plan explicitly required package-level reporting for touched seams.

| Command Result | Coverage |
| --- | --- |
| `go test ./internal/sbi/... -cover` `internal/sbi` | 54.1% |
| `go test ./internal/sbi/... -cover` `internal/sbi/consumer` | 47.8% |
| `go test ./internal/sbi/... -cover` `internal/sbi/processor` | 64.2% |
| `go test ./internal/notifier ./internal/context -cover` `internal/notifier` | 77.2% |
| `go test ./internal/notifier ./internal/context -cover` `internal/context` | 69.5% |

Interpretation:

- handler coverage now exists across every `internal/sbi/api_*.go` file
- processor coverage is materially stronger at the update reconciliation and
  data-collection seam
- consumer coverage is improved for standardized SMF/MTLF paths, but still not
  uniform across all outbound clients

---

## 6. Explicit Gaps After Phase 1

These gaps remain intentionally visible and should drive Phase 2.

### 6.1 `NsmfService` Still Uses Raw HTTP

`NsmfService` was **not** migrated to a generated OpenAPI request path.

Reason:

- the current local dependency snapshot `github.com/free5gc/openapi v1.2.3`
  does not safely model the SMF request fields NWDAF currently needs for the
  UPF event workflow, especially the `upfEvents` and
  `bundledEventNotifyUri`-driven shape

Current mitigation:

- direct `gock`-based consumer tests now protect request method, path, body
  shape, unsubscribe behavior, and non-2xx handling
- processor reconciliation tests also protect the higher-level replacement flow

Status:

- acceptable as a Phase 1 exception
- still below the original ideal end state from the detailed plan

### 6.2 `gomock` Is Only Partially Introduced

Phase 1 introduced `gomock` at the seams that immediately improved handler and
consumer-facing tests.

Not yet complete:

- broader handwritten app stubs in other processor/app-adjacent tests were not
  all replaced in this phase

Status:

- successful incremental introduction
- not full seam consolidation

### 6.3 Consumer Test Strategy Is Not Yet Uniform

Current state:

- `SMF` and `MTLF` were explicitly normalized
- `ML service` and `Daisy` still retain older test styles
- `ADRF` has no direct consumer test yet

Status:

- enough for a Phase 1 safety-net improvement
- not yet one fully consistent outbound-mocking pattern across every consumer

### 6.4 ADRF Consumer Coverage Is Still Missing

`AdrfClient` remains untested directly at the consumer boundary.

Reason for deferral:

- Phase 1 prioritized the highest-risk standardized SBI paths and the new
  handler seam
- ADRF in this repository is currently treated as a custom/local integration
  client rather than a locally-governed generated-client path

Status:

- not blocked on new architectural decisions if the next step is only to add
  tests
- appropriate Phase 2 target

---

## 7. Phase 2 Entry Scope

The most direct Phase 2 follow-up items are:

1. add direct `AdrfClient` consumer tests
2. decide whether to normalize `MlServiceClient` and `DaisyClient` tests in the
   same style pass
3. expand `gomock` into the next highest-value handwritten app seams
4. decide whether the SMF raw-HTTP exception should stay documented as-is or be
   followed by contract/governance work to enable a safe generated-client path

---

## 8. Phase 1 Completion Judgment

Phase 1 is considered complete because:

- the highest-risk missing seams now have direct coverage
- later Priority 4/5 work no longer depends on the previous under-tested HTTP
  boundary
- the remaining gaps are now explicit, narrower, and mostly localized to
  consumer normalization rather than broad safety-net absence
