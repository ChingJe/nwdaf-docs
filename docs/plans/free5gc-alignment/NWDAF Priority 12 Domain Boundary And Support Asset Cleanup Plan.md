# NWDAF Priority 12 Domain Boundary And Support Asset Cleanup Plan

Date: 2026-06-30
Last updated: 2026-07-01

Status: Phase 1 implemented in `NWDAF/` baseline `9b343ef`; Phase 2 implemented in `NWDAF/` commits `0ddbf3c` and `b547727`

Historical remediation item:

- `Priority 12 — Clean Repo And Package Ownership Boundaries`

This plan now tracks Priority 12 as a phased line rather than one undivided
cleanup round.

The central direction of this plan is stricter than the earlier generic note:

1. Daisy integration leaves `internal/sbi` completely.
2. local ML service integration is renamed and repositioned as an
   inference-engine integration owned by `AnLF`, not by `internal/sbi`.
3. `internal/sbi` returns to a narrower role as NWDAF SBI transport edge for
   true NWDAF service operations only.
4. `AnLF` and `MTLF` each gain their own HTTP server instead of attaching
   non-SBI ingress to the main SBI server.

The current document state is:

1. Phase 1 is already implemented in `NWDAF/` baseline commit `9b343ef`.
   That phase moved domain integrations out of shared transport ownership,
   renamed runtime config to `inferenceEngine`, and moved the retrain support
   tools out of the main runtime repo.
2. Phase 2 is now implemented in `NWDAF/` commits `0ddbf3c` and `b547727`.
   That phase completed the transport-meaning and server-topology split: the
   main SBI server now serves NWDAF SBI only, while `AnLF` and `MTLF` own
   their own auxiliary HTTP servers and callback URI derivation.

Related issue records:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project free5GC Alignment Scan Findings.md`

---

## 1. Purpose

This plan records the implementation round that closed the still-open portions of:

- `Priority 12 — Clean Repo And Package Ownership Boundaries`

The purpose of this round was:

1. remove project-local Daisy workflow ownership from `internal/sbi`
2. remove local inference-engine integration ownership from `internal/sbi`
3. give `AnLF` and `MTLF` direct ownership of their own non-3GPP external
   integration seams
4. finish the remaining transport-boundary split after the already-landed
   package-ownership move

This round is intentionally about ownership and package boundaries.

The implemented Phase 2 scope was about:

- which inbound HTTP surfaces belong to the main NWDAF SBI server
- which inbound HTTP surfaces belong to `AnLF` and `MTLF`
- how callback URIs should be derived once those surfaces stop sharing one Gin
  server
- how lifecycle wiring should represent three owned inbound servers in one
  process

It is not about:

- reopening the completed app-boundary reconstruction from Priority 4
- reopening the completed lifecycle-ownership closure from Priority 2
- redoing the already-landed Phase 1 package moves in `NWDAF/` baseline
  `9b343ef`
- redesigning Priority 6 post-acceptance activation semantics
- solving Priority 10 OpenAPI/model governance in the same change
- deciding Priority 11 NRF/metrics/full free5GC integration level
- mixing this work with unrelated analytics behavior changes

---

## 2. Why A Separate Priority-12 Plan Is Needed Now

The earlier remediation file described Priority 12 correctly at a high level,
but it did not yet pin down the package-boundary direction tightly enough for
implementation.

The current repository state after the completed Priority 2, 4, 5, 7, and 8
rounds and after the landed Priority 12 baseline commit `9b343ef` makes the
remaining problem narrower and clearer:

1. the app/service/runtime boundary is already materially improved
2. Daisy and ML client creation no longer happens ad hoc inside `AnLF` and
   `MTLF`
3. production inference-engine and Daisy ownership no longer lives under
   `internal/sbi/consumer`
4. however, the active runtime still uses one main Gin server and late route
   injection from `pkg/service` for non-SBI domain ingress
5. that means the package move landed, but the transport meaning is still not
   aligned with the intended free5GC-style boundary story

This means the remaining issue is no longer "who creates the client" or
"which package owns the local external client implementation".

It is now "which owned server surface is allowed to expose non-SBI ingress and
how that server should be wired into the app lifecycle".

Without a dedicated Priority 12 plan:

1. Daisy callback cleanup and inference-engine cleanup can be mistaken for the
   already-completed Priority 4 ownership round
2. `internal/sbi` can keep accumulating project-local workflow logic just
   because it already hosts the shared HTTP server
3. naming cleanup around "ML service" versus local inference-engine role can be
   delayed indefinitely, leaving the wrong architectural message in the code
4. route-level exceptions can keep `internal/sbi` entangled with
   inference-engine or Daisy-local behavior even after client moves
5. keeping non-SBI ingress on the shared Gin server would preserve the wrong
   transport meaning even if the handlers moved package

This plan therefore turns the old broad Priority 12 note into one explicit
implementation direction with fixed boundary rules.

---

## 3. Planning Basis

This plan is based on the following local sources:

- `AGENTS.md`
- `nwdaf-docs/docs/development_policy.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project free5GC Alignment Scan Findings.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 4 External Client Ownership And Priority 2 Residual Lifecycle Hardening Plan.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 2 Remaining Lifecycle Completion Plan.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 7 Logging Boundary And Hygiene Alignment Plan.md`
- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/exemplar-alignment.md`
- `free5gc-dev-skill/references/review-checklist.md`
- `free5gc-dev-skill/references/nf-architecture.md`
- `free5gc-dev-skill/references/sbi-development.md`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/consumer/consumer.go`
- `resources/references/free5gc-main/NFs/udr/internal/sbi/consumer/consumer.go`
- `resources/references/free5gc-main/NFs/pcf/internal/sbi/server.go`
- `resources/references/free5gc-main/config/udrcfg.yaml`
- `resources/openapi/openapi/models/model_nwdaf_ml_model_prov_notif.go`
- the current `NWDAF/` tree and baseline commit `9b343ef` on 2026-07-01

This plan also uses direct inspection of the landed baseline and the remaining
topology-sensitive code paths in:

- `NWDAF/internal/sbi/server.go`
- `NWDAF/internal/sbi/processor/processor.go`
- `NWDAF/internal/anlf/`
- `NWDAF/internal/mtlf/`
- `NWDAF/pkg/service/init.go`
- `NWDAF/pkg/factory/config.go`

---

## 4. Exemplar Choice And Evidence Basis

This Priority 12 plan uses the following free5GC exemplar set:

1. primary exemplar: `udm`
   - `internal/sbi/consumer/consumer.go`
   - evidence used:
     - `internal/sbi/consumer` is for NF-facing or standards-facing consumer
       assembly, not for arbitrary project-local runtime integrations
2. secondary exemplar: `udr`
   - `internal/sbi/consumer/consumer.go`
   - `config/udrcfg.yaml`
   - evidence used:
     - consumer ownership remains tied to standardized NF/service interactions
     - runtime config is centralized and not mixed with a per-NF Python harness
       tree
3. secondary exemplar: `pcf`
   - `internal/sbi/server.go`
   - evidence used:
     - `internal/sbi/server.go` is the HTTP/SBI edge and route owner
     - route ownership should stay tied to true SBI service edges, not
       project-local workflow integrations

Direct evidence from these exemplars supports the following rules:

1. `internal/sbi` is a transport/service edge, not the default home for every
   external HTTP client in the repository
2. standardized or standards-adjacent NF interactions fit naturally under a
   consumer package
3. local domain integrations that are not part of the NF's SBI contract should
   prefer domain-owned seams

There is no exact free5GC exemplar for:

1. NWDAF's project-local Daisy retrain workflow
2. NWDAF's project-local inference engine
3. the exact split between current ML-model-provision handling and a local
   non-3GPP inference engine

Those parts of this plan are therefore inference from free5GC package-role
discipline plus the existing NWDAF `AnLF` / `MTLF` domain split, not direct
one-to-one feature copying.

---

## 5. Current NWDAF State

### 5.1 What Is Already Completed

The current repository already reflects the intended structural progress from
the completed earlier rounds:

1. app/service lifecycle ownership is materially narrowed and explicit
2. `AnLF` and `MTLF` no longer instantiate Daisy or ML clients ad hoc inside
   their runtime logic
3. config-global drift in already-touched runtime paths is materially reduced
4. notifier/runtime ownership and logging boundaries are materially improved

This plan therefore does not need to reopen the completed Priority 2 and 4
ownership decisions.

### 5.2 Confirmed Phase-1 Completion Baseline

Phase 1 is now materially landed in `NWDAF/` baseline commit `9b343ef`:

1. local inference-engine client ownership moved to `internal/anlf`
2. Daisy client and callback ownership moved to `internal/mtlf`
3. production Daisy and inference-engine files were removed from
   `internal/sbi`
4. runtime config naming was moved from `mlService` to `inferenceEngine`
5. retrain replay and retrain analysis tooling left `NWDAF/` for
   `nwdaf-resources/`

This meant the remaining Priority 12 gap was no longer package placement
alone.

### 5.3 Confirmed Phase-2 Topology Completion

The previously open shared-server semantics problem is now closed in `NWDAF/`
by commit `0ddbf3c` and the follow-up contract cleanup commit `b547727`:

1. `pkg/service/init.go`
   - explicitly owns three inbound listeners:
     - the main SBI server
     - the `AnLF` auxiliary server
     - the `MTLF` auxiliary server
   - starts owned listeners before callback-dependent background work
   - stops all three listeners explicitly during termination
2. `internal/sbi/server.go`
   - no longer exposes router access for late domain-route injection
   - binds its listener explicitly and serves only the main NWDAF SBI surface
3. `internal/anlf` and `internal/mtlf`
   - now each own their own inbound HTTP listener implementation under
     `internal/anlf/server.go` and `internal/mtlf/server.go`

The runtime meaning is now:

`main SBI server` + `owned AnLF auxiliary server` + `owned MTLF auxiliary server`

That is materially aligned with the intended free5GC-style story that the main
NF HTTP server should primarily serve SBI semantics.

### 5.4 Confirmed Callback-URI Ownership Completion

The callback-URI ownership split is also now closed:

1. `MTLF` Daisy callback URLs are derived from `configuration.mtlf.server`
2. external MTLF provision-notification callback URLs are derived from
   `configuration.anlf.server`
3. the old `externalMtlf.notifUri` override path was removed from the active
   runtime contract in follow-up commit `b547727`
4. focused tests now cover:
   - `AnLF` auxiliary callback URI derivation
   - `MTLF` auxiliary callback URI derivation
   - partial-startup failure cleanup after listener bind errors

### 5.5 Confirmed Support-Asset Baseline Completion

For the currently identified retrain replay and retrain analysis tooling, the
repo-scope move is already completed in the current baseline:

1. `NWDAF/` no longer carries those tool trees
2. the moved support assets now live under the sibling repository
   `nwdaf-resources/`

This plan keeps that result as part of Priority 12 history, but it is not the
main open implementation gap for the active next phase.

---

## 6. Fixed Decisions For This Round

The following decisions are treated as fixed for Priority 12:

1. Daisy integration leaves `internal/sbi` completely.
   No Daisy-specific client implementation, callback payload type, callback
   handler, or route-construction helper remains under `internal/sbi`.
2. Local ML service integration is renamed in local code and docs to an
   `inferenceEngine` concept.
   The local integration should stop advertising itself through
   `MlService*`-named boundaries except where a 3GPP-standardized external
   contract explicitly requires that naming.
3. Inference-engine integration leaves `internal/sbi` completely.
   It becomes `AnLF`-owned domain integration rather than an SBI consumer.
4. `internal/sbi` keeps standards-facing NWDAF SBI service edges only.
   Inference-engine and Daisy-specific routes, handlers, payload definitions,
   and registration helpers leave `internal/sbi` completely.
5. `AnLF` owns the local inference-engine seam.
   This includes its interface, implementation package, request/response
   compatibility types, and tests.
6. `MTLF` owns the Daisy seam.
   This includes its interface, implementation package, local callback
   contract, and workflow tests.
7. `processor` should stop acting as a generic transport bridge for these local
   integrations once narrower domain-owned seams exist.
8. The config rename is a strict break.
   Local runtime config should stop accepting `mlService` and should use
   `inferenceEngine` only.
9. `tools/retrain_replay` and `tools/retrain_analysis` move out of `NWDAF/`.
   Their new home is a sibling workspace repository:
   `nwdaf-resources/`.
10. `internal/sbi` remains the HTTP server for the NWDAF SBI surface only.
    `AnLF` and `MTLF` each own a separate inbound HTTP server for their
    non-SBI integration surfaces.
11. Daisy or inference-engine ingress must not be mounted onto the main SBI
    Gin engine through `Router()` escape hatches or equivalent late route
    injection.
12. `AnLF` and `MTLF` auxiliary servers are process-local runtime surfaces,
    not NRF-registered NF services in this round.
13. `AnLF` and `MTLF` auxiliary server config starts with a narrow shape.
    The plan should use server-local bind information first, not a full copy
    of every SBI config field.
14. callback URIs for model-provision and Daisy-complete flows should be
    derived from owned server config rather than remaining as hard-coded
    shared-SBI paths.

---

## 7. Target End State

### 7.1 `internal/sbi`

`internal/sbi` should own:

1. NWDAF standards-facing HTTP/SBI route registration
2. handler-level request parsing and response writing for real NWDAF SBI
   operations
3. thin app- or domain-owned delegation seams

`internal/sbi` should not own:

1. Daisy workflow client code
2. Daisy callback payload definitions
3. Daisy workflow orchestration
4. inference-engine client code
5. inference-engine request/response compatibility types
6. inference-engine or Daisy callback routes, handlers, or registration helpers
7. the only HTTP listener for the whole process once non-SBI domain ingress is
   separated intentionally

### 7.2 Main SBI Server

The main NWDAF SBI server should own:

1. only standards-facing NWDAF SBI APIs
2. only routes whose semantics belong to the primary NWDAF SBI surface
3. its own lifecycle start/stop wiring under `pkg/service`

The main NWDAF SBI server should not own:

1. Daisy callback ingress
2. inference-engine callback ingress
3. route assembly for `AnLF` or `MTLF` auxiliary HTTP surfaces
4. exported router escape hatches used only to attach non-SBI domain routes

### 7.3 `internal/anlf`

`internal/anlf` should own:

1. the local inference-engine interface
2. the local inference-engine implementation package
3. model load / unload / prediction orchestration
4. its own inbound HTTP server for model-provision or monitor-related
   non-SBI ingress
5. tests for inference-engine client behavior and `AnLF` integration behavior

### 7.4 `internal/mtlf`

`internal/mtlf` should own:

1. the Daisy integration interface
2. the Daisy integration implementation package
3. Daisy async callback payload contract for the local training workflow
4. callback-to-workflow completion routing
5. its own inbound HTTP server for Daisy or training-related non-SBI ingress
6. tests for Daisy workflow and callback completion behavior

### 7.5 Auxiliary Server Semantics

The target runtime shape for this round is:

1. one main NWDAF SBI server
2. one `AnLF`-owned auxiliary HTTP server
3. one `MTLF`-owned auxiliary HTTP server

These three listeners are all part of the same NWDAF process, but they do not
share a single generic Gin engine.

This plan treats that distinction as important:

1. multiple SBI-style service APIs may share one NF process
2. but project-local or domain-local ingress should not be smuggled into the
   main SBI server merely because it is convenient
3. server ownership should follow transport meaning, not only package import
   convenience

### 7.5.1 Auxiliary Server Ownership And Lifecycle Shape

The intended lifecycle shape for these three listeners is deliberately aligned
with the way free5GC control-plane NFs keep transport servers under
`pkg/service` ownership while the server implementation stays in the
transport-owning package.

The target ownership split is:

1. `pkg/service` owns process lifecycle for all inbound servers in the NWDAF
   process
2. `internal/sbi/server.go` owns only the main NWDAF SBI listener
3. `internal/anlf/server.go` owns the `AnLF` auxiliary listener
4. `internal/mtlf/server.go` owns the `MTLF` auxiliary listener

This means the repository should not introduce:

1. a generic shared server manager abstraction only to avoid explicit fields
   on the app struct
2. `AnLF` or `MTLF` code that secretly starts its own listener outside
   `pkg/service`
3. a return to the current shared-Gin route mounting pattern

The expected app-level shape is explicit:

1. `NwdafApp` holds the main SBI server plus one `AnLF` auxiliary server plus
   one `MTLF` auxiliary server
2. `NewApp` constructs all enabled and required inbound servers before runtime
   work starts
3. `Start` starts all owned listeners before callback-dependent background work
   is allowed to run
4. `Terminate` and the shutdown path stop all owned listeners explicitly
5. the app `WaitGroup` remains the single owner-visible synchronization point
   for the long-lived server goroutines

This plan intentionally keeps transport implementation and lifecycle ownership
separate:

1. `internal/anlf` and `internal/mtlf` own their transport implementations
   because those listeners belong to those domains
2. `pkg/service` still owns startup, shutdown, failure policy, and process
   convergence because those concerns belong to the app lifecycle boundary

### 7.6 Repo Scope

The main `NWDAF/` repo should no longer host retrain replay or retrain analysis
tooling.

Those assets move to the sibling workspace repository:

- `nwdaf-resources/tools/retrain_replay/`
- `nwdaf-resources/tools/retrain_analysis/`

The end state must be intentional and documented, not accidental.

---

## 8. Scope

### 8.1 In Scope

1. splitting the current shared inbound HTTP surface into:
   - a main NWDAF SBI server
   - an `AnLF` auxiliary server
   - an `MTLF` auxiliary server
2. updating callback URI derivation to use owned auxiliary-server config
3. removing the current late route injection pattern from `pkg/service`
4. removing `Router()`-style shared-engine escape hatches that only exist to
   support non-SBI route mounting
5. updating tests and docs to match the new ownership boundaries and server
   topology

### 8.2 Conditionally In Scope

The following may be included only if they are required to preserve one
coherent end state:

1. narrow app/service assembly updates in `pkg/service/` if construction
   ownership must move out of the current shared-server wiring
2. narrow app/service lifecycle updates if separate auxiliary server startup
   and shutdown need new owned wiring

### 8.3 Explicitly Out Of Scope

1. NRF registration or metrics-server wiring
2. broader config-scope separation from Priority 9
3. OpenAPI/model-governance redesign from Priority 10
4. late-failure semantics redesign from Priority 6
5. changing the actual inference or retrain policy behavior unless required by
   the ownership move
6. broad renames of 3GPP-standardized API names in standardized callback or
   payload types
7. NRF registration of the new `AnLF` / `MTLF` auxiliary HTTP surfaces
8. broad SCP, OAuth, or certificate expansion for the new auxiliary HTTP
   surfaces beyond what is already required by the current local runtime
9. redoing the already-landed Phase 1 package moves in baseline `9b343ef`
10. reintroducing moved workflow tooling into `NWDAF/`

---

## 9. Proposed Workstreams

### Workstream A — Define The New Ownership Map

Objectives:

1. inventory which `internal/sbi` files currently own Daisy or inference
   integration responsibilities
2. assign each responsibility to `AnLF`, `MTLF`, `internal/sbi`, or repo
   support tooling
3. settle the local naming target for `inferenceEngine`
4. settle the inbound server topology explicitly instead of leaving route
   placement as an implementation detail

Required outcomes:

1. one ownership table exists before package moves begin
2. `ML service` rename target is explicit: `inferenceEngine`
3. any current inference-engine callback route is classified for complete
   removal from `internal/sbi`
4. the shared-Gin-server approach is explicitly rejected for non-SBI ingress

### Workstream B — Move Inference Integration To `AnLF`

Objectives:

1. create an `AnLF`-owned inbound server for model-provision and related
   callback ingress
2. move model-provision or monitor-related ingress off the shared SBI server
3. derive `AnLF` callback URIs from `AnLF` server config
4. keep inference-engine ownership under `internal/anlf` while changing only
   the remaining transport topology

Required outcomes:

1. model-provision callback ingress no longer depends on the shared SBI Gin
   engine
2. the active `AnLF` runtime surface has its own listener and bind settings
3. the Phase 1 inference-engine ownership move remains intact
4. local code and tests keep inference-oriented naming

### Workstream C — Move Daisy Integration To `MTLF`

Objectives:

1. create an `MTLF`-owned inbound server for Daisy training-complete ingress
2. move Daisy callback ingress off the shared SBI server
3. derive Daisy callback URLs from `MTLF` server config
4. keep Daisy ownership under `internal/mtlf` while changing only the
   remaining transport topology

Required outcomes:

1. Daisy callback ingress no longer depends on the shared SBI Gin engine
2. the active `MTLF` runtime surface has its own listener and bind settings
3. the Phase 1 Daisy ownership move remains intact
4. no Daisy-specific route assembly remains attached to the main SBI server

### Workstream D — Re-narrow `internal/sbi`

Objectives:

1. leave `internal/sbi` with standards-facing NWDAF transport edges only
2. remove the remaining inference-engine and Daisy callback ingress from
   `internal/sbi`
3. remove package-local assumptions that `internal/sbi` owns all external HTTP
   integrations
4. remove any `Router()`-style escape hatch that exists only to attach
   non-SBI domain routes after server construction

Required outcomes:

1. `internal/sbi/consumer` contains only SBI-appropriate interactions after the
   move
2. `internal/sbi/server.go` no longer imports Daisy-specific or
   inference-engine-specific route logic
3. `internal/sbi` no longer owns inference-engine callback handling
4. main-server construction remains self-contained and no longer expects later
   domain-route injection from `pkg/service`

### Workstream E — Introduce Auxiliary Server Config And Lifecycle

Objectives:

1. define narrow `AnLF` and `MTLF` server config blocks
2. derive callback URIs from owned server bind settings
3. make `pkg/service` start and stop three owned servers explicitly
4. keep auxiliary-server implementation in `internal/anlf` and `internal/mtlf`
   while preserving `pkg/service` as the lifecycle owner
5. make listener bind failure observable before background work begins

Required outcomes:

1. `configuration.anlf.server` exists with owned bind settings
2. `configuration.mtlf.server` exists with owned bind settings
3. hard-coded shared-SBI callback paths are removed from the touched runtime
   flows
4. startup failure for any required auxiliary server fails the app fast
5. shutdown wiring covers the main SBI server plus both auxiliary servers
6. `pkg/service` explicitly owns `sbiServer`, `anlfServer`, and `mtlfServer`
   rather than hiding them behind a generic server registry
7. `internal/anlf/server.go` and `internal/mtlf/server.go` each build and own
   only their own router and `http.Server`
8. listener start order is explicit: listeners first, callback-dependent
   background work second
9. termination order is explicit and bounded: cancel context, stop owned
   background work, shut down auxiliary listeners, shut down the main SBI
   listener, then wait for goroutine convergence
10. listener binding should fail fast during startup rather than surfacing only
    later from a detached goroutine

Implementation notes for this workstream:

1. the plan should prefer synchronous listener acquisition followed by
   goroutine-based serving so bind errors return directly to `pkg/service`
2. `Run()` or the equivalent start path for each owned listener should be able
   to report address-in-use or invalid-bind errors before callback-producing
   workflows start
3. this round does not require a generic multi-server framework; an explicit
   app-owned three-server shape is preferred because it matches free5GC's
   normal lifecycle style more closely
4. startup-triggered training or other callback-dependent background work must
   not begin before the corresponding auxiliary listener is confirmed ready
5. shutdown must keep timeout-bounded HTTP server stop behavior rather than
   waiting indefinitely on external peers

### Workstream F — Close Phase Bookkeeping

Objectives:

1. document the split between the landed Phase 1 baseline and the planned
   Phase 2 server-topology work
2. keep the remediation files aligned with the actual baseline commit state
3. preserve the support-asset move as completed Priority 12 history without
   making it look like an open code task

Required outcomes:

1. plan wording matches baseline commit `9b343ef`
2. remediation status wording reflects that Phase 1 is complete and Phase 2 is
   implemented
3. support-asset movement remains documented as completed history
4. no document still describes the old pre-baseline package state as current

---

## 10. Recommended Implementation Order

1. complete Workstream A first
2. implement Workstream B before Workstream C
3. implement Workstream C next
4. implement Workstream D immediately after the two ownership moves
5. implement Workstream E before final verification so callback URIs and
   lifecycle wiring match the new topology
6. finish with Workstream F once the code boundary is stable

Implemented commit grouping:

- `refactor(anlf): add owned auxiliary ingress server`
- `refactor(mtlf): add owned auxiliary ingress server`
- `refactor(sbi): narrow transport ownership to standards-facing edges`
- `refactor(service): add owned anlf and mtlf auxiliary servers`
- `docs(priority12): split baseline cleanup from server-topology phase`
- `refactor(service): split anlf and mtlf auxiliary servers`
- `fix(config): remove stale mtlf callback override`

The first four subjects were planning examples. The last two are the commits
that actually landed the server-topology phase in `NWDAF/`.

---

## 11. Acceptance Criteria

Priority 12 is considered complete when all of the following are true:

1. no Daisy client, Daisy callback payload, or Daisy callback handler remains
   under `internal/sbi`
2. no local inference-engine client or inference-engine compatibility model
   remains under `internal/sbi`
3. no inference-engine callback route, handler, payload, or registration helper
   remains under `internal/sbi`
4. `AnLF` owns the inference integration seam and `MTLF` owns the Daisy
   integration seam
5. `processor` no longer acts as the default assembly bridge for these local
   external integrations unless a narrower documented seam still requires it
6. the main SBI server no longer carries Daisy or inference-engine ingress
7. `AnLF` and `MTLF` each expose their own auxiliary inbound HTTP server
8. callback URIs for the touched flows are derived from owned auxiliary-server
   config instead of hard-coded shared-SBI paths
9. retrain replay and retrain analysis tooling no longer live in `NWDAF/` and
   are documented under `nwdaf-resources/`
10. verification shows no regression in the touched `AnLF`, `MTLF`, `SBI`, and
    service assembly paths
11. `pkg/service` is the visible lifecycle owner for all three inbound
    listeners in the NWDAF process
12. `internal/anlf` and `internal/mtlf` each own their own `server.go` or
    equivalent transport implementation instead of reusing the main SBI router
13. no `Router()`-style late route injection remains in the final runtime path
14. startup can fail fast on listener bind errors before callback-dependent
    background work begins
15. auxiliary-listener callback URI derivation does not depend on the main SBI
    bind address

---

## 12. Verification Plan

Minimum focused verification for the touched code:

```bash
go test ./internal/anlf
go test ./internal/mtlf
go test ./internal/sbi
go test ./internal/sbi/processor
go test ./internal/sbi/consumer
go test ./pkg/service
go test ./pkg/factory
```

Full verification target after the round:

```bash
make build
go test ./...
make lint
```

Verification rerun after the final follow-up cleanup commit `b547727`:

- `go test ./...`
- `make build`
- `make lint`

Result:

- full Go test suite passed
- build passed
- `golangci-lint` reported `0 issues`

Review checks for this round:

1. does `internal/sbi` still own any Daisy-specific production code?
2. does `internal/sbi` still own any local inference-engine production code or
   callback routing?
3. did `AnLF` gain an integration seam without reintroducing package-global
   config or ad hoc client creation?
4. did `MTLF` gain Daisy ownership without reopening the completed lifecycle
   work from Priority 2?
5. do `AnLF` and `MTLF` each own their own inbound server rather than using a
   shared SBI Gin engine?
6. were hard-coded callback URIs removed from the touched flows?
7. are retained support assets intentionally documented instead of passively
   remaining in place?

Verification should include focused config tests proving that
`configuration.inferenceEngine` is accepted and that the new
`configuration.anlf.server` / `configuration.mtlf.server` blocks are validated
correctly.

Verification should also include lifecycle-oriented checks for the new server
topology:

1. a focused service test that proves `pkg/service` starts and stops all three
   owned listeners
2. a focused startup-failure test that proves auxiliary-listener bind failure
   aborts startup before callback-dependent background work proceeds
3. a focused shutdown test that proves server stop remains bounded and does not
   leave auxiliary listeners running after app termination
4. config tests that reject auxiliary callback-URI derivation from unusable
   wildcard bind addresses if that remains the chosen narrow config contract
5. focused `AnLF` and `MTLF` tests that prove callback URLs are derived from
   owned auxiliary-server config rather than from `GetSbiUri()`

---

## 13. Resolved Direction Before Implementation

The following implementation-direction decisions are fixed:

1. config rename strategy:
   - strict break from `mlService`
   - runtime config uses `inferenceEngine` only
   - no compatibility alias remains
2. naming target:
   - Go and YAML naming use `InferenceEngine` / `inferenceEngine` according to
     local Go and YAML casing conventions
3. retained tooling scope:
   - `tools/retrain_replay` moves out of `NWDAF/`
   - `tools/retrain_analysis` moves out of `NWDAF/`
   - both move to the sibling workspace repository `nwdaf-resources/`
4. `internal/sbi` scope:
   - inference-engine and Daisy-specific ownership leaves `internal/sbi`
     completely
   - no inference-engine or Daisy route, handler, payload, or helper remains
     there
5. server topology:
   - the main SBI server serves NWDAF SBI only
   - `AnLF` and `MTLF` each own a separate auxiliary HTTP server
   - shared-Gin late route injection is not an allowed end state
6. auxiliary server registration scope:
   - `AnLF` and `MTLF` auxiliary servers are runtime-owned local surfaces in
     this round
   - they are not NRF-registered NF services in this round
7. auxiliary server config shape:
   - start with narrow server-local bind config
   - do not copy the full SBI config contract unless a later requirement
     forces it
8. lifecycle owner:
   - `pkg/service` remains the lifecycle owner for all inbound listeners in
     the process
   - `internal/anlf` and `internal/mtlf` own transport implementation, not
     top-level process lifecycle
9. app shape:
   - `NwdafApp` should explicitly hold `sbiServer`, `anlfServer`, and
     `mtlfServer`
   - do not introduce a generic server manager abstraction only to reduce
     visible fields
10. startup ordering:
   - required listeners start before callback-dependent background work
   - bind failure is treated as startup failure, not as a deferred warning
11. shutdown ordering:
   - app shutdown cancels context, stops owned background work, shuts down
     auxiliary listeners, shuts down the main SBI listener, and then waits for
     goroutine convergence
12. callback URI derivation:
   - touched Daisy and inference-related callback URIs derive from owned
     auxiliary-server config
   - they must not derive from the main SBI URI

---

## 14. Expected Outcome

After Priority 12, the repository should communicate a much clearer ownership
story:

1. `internal/sbi` means NWDAF SBI transport edge
2. the main SBI server serves NWDAF SBI only
3. `internal/anlf` owns inference-engine integration and its auxiliary HTTP
   ingress
4. `internal/mtlf` owns Daisy integration and its auxiliary HTTP ingress
5. `nwdaf-resources/` owns replay and analysis workflow tooling rather than the
   main `NWDAF/` repo

That outcome is narrower, easier to defend, and easier to extend than the
current "everything external HTTP-related lives under `internal/sbi`" shape.
