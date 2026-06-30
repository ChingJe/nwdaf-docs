# NWDAF Priority 12 Domain Boundary And Support Asset Cleanup Plan

Date: 2026-06-30
Last updated: 2026-07-01

Status: Drafted, not started

Historical remediation item:

- `Priority 12 — Clean Repo And Package Ownership Boundaries`

This plan refines the previously brief Priority 12 description into a concrete
implementation round for the main `NWDAF/` repository.

The central direction of this plan is stricter than the earlier generic note:

1. Daisy integration leaves `internal/sbi` completely.
2. local ML service integration is renamed and repositioned as an
   inference-engine integration owned by `AnLF`, not by `internal/sbi`.
3. `internal/sbi` returns to a narrower role as NWDAF SBI transport edge for
   true NWDAF service operations only.

Related issue records:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project free5GC Alignment Scan Findings.md`

---

## 1. Purpose

This plan defines the next implementation round for the still-open portions of:

- `Priority 12 — Clean Repo And Package Ownership Boundaries`

The purpose of this round is:

1. remove project-local Daisy workflow ownership from `internal/sbi`
2. remove local inference-engine integration ownership from `internal/sbi`
3. give `AnLF` and `MTLF` direct ownership of their own non-3GPP external
   integration seams
4. classify which non-runtime support assets remain in the main repo and which
   should move to a dedicated support location

This round is intentionally about ownership and package boundaries.

It is about:

- where Daisy workflow code belongs
- where inference-engine client code belongs
- which local callback routes are standards-facing SBI edges versus project-
  local workflow ingress
- how much support tooling belongs in the main implementation repo

It is not about:

- reopening the completed app-boundary reconstruction from Priority 4
- reopening the completed lifecycle-ownership closure from Priority 2
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
rounds makes the remaining problem narrower and clearer:

1. the app/service/runtime boundary is already materially improved
2. Daisy and ML client creation no longer happens ad hoc inside `AnLF` and
   `MTLF`
3. however, those integrations still live under `internal/sbi/consumer`, which
   is the wrong ownership boundary for project-local external systems
4. Daisy callback ingress still lives under generic SBI files even though the
   behavior is MTLF-local workflow completion

This means the remaining issue is no longer "who creates the client".

It is now "which package family is allowed to own non-3GPP external-system
integrations and their local callback contracts".

Without a dedicated Priority 12 plan:

1. Daisy callback cleanup and inference-engine cleanup can be mistaken for the
   already-completed Priority 4 ownership round
2. `internal/sbi` can keep accumulating project-local workflow logic just
   because it already hosts the shared HTTP server
3. naming cleanup around "ML service" versus local inference-engine role can be
   delayed indefinitely, leaving the wrong architectural message in the code
4. route-level exceptions can keep `internal/sbi` entangled with
   inference-engine or Daisy-local behavior even after client moves

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
- the current `NWDAF/` tree on 2026-06-30

This plan also uses direct inspection of current remaining code paths in:

- `NWDAF/internal/sbi/consumer/`
- `NWDAF/internal/sbi/api_daisy_callback.go`
- `NWDAF/internal/sbi/api_mlmodelprovision.go`
- `NWDAF/internal/sbi/server.go`
- `NWDAF/internal/sbi/processor/processor.go`
- `NWDAF/internal/anlf/`
- `NWDAF/internal/mtlf/`
- `NWDAF/tools/retrain_replay/`
- `NWDAF/tools/retrain_analysis/`

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

### 5.2 Confirmed Remaining Package-Boundary Problem

The remaining problem is that project-local external integrations still live in
the wrong package family:

1. `internal/sbi/consumer/consumer.go`
   - still defines `MlServiceAPI` and `DaisyServiceAPI`
   - still assembles those clients inside `NewConsumer(...)`
2. `internal/sbi/consumer/ml_service.go`
   - still implements the local inference-engine HTTP client under an SBI
     consumer package
3. `internal/sbi/consumer/daisy_service.go`
   - still implements the Daisy training-system HTTP client under an SBI
     consumer package
4. `internal/sbi/processor/processor.go`
   - still acts as the assembly bridge that carries both integrations into
     `AnLF` and `MTLF`

This means Priority 4 fixed who owns client construction more strictly than
before, but it did not yet fix which package family is allowed to own those
integrations.

### 5.3 Confirmed Daisy Workflow Ownership Problem

The current NWDAF tree still defines Daisy callback ingress under generic SBI
files:

1. `internal/sbi/api_daisy_callback.go`
   - owns the Daisy callback payload type
   - owns the Daisy callback HTTP handler
2. `internal/sbi/server.go`
   - mounts Daisy callback routes directly under the shared SBI server
3. `internal/sbi/processor/processor.go`
   - exposes `HandleDaisyCallback(...)` only to forward to `MTLF`

This means the current route path is:

`internal/sbi` HTTP edge -> `internal/sbi/processor` adapter -> `internal/mtlf`

That is too broad for a project-local MTLF workflow.

### 5.4 Confirmed Inference-Integration Ownership Problem

The current NWDAF tree still models local inference integration as if it were
an SBI consumer concern:

1. `internal/anlf/anlf.go`
   - depends on `internal/sbi/consumer.MlServiceAPI`
2. `internal/anlf/model.go`
   - performs model load/unload lifecycle through that injected client
3. `internal/anlf/analytics.go`
   - performs prediction through that injected client
4. the name `MlService` is still used even though the local role is really an
   inference-engine integration, not a standards-facing NWDAF SBI peer

This is the key reason `AnLF` belongs in the core Priority 12 scope and not as
an afterthought.

### 5.5 Confirmed Support-Asset Boundary Problem

The current repository no longer carries the old `test/fake_*` Python harness
tree on the active refactor branch, but it still carries non-runtime support
assets whose repo placement is not yet explicitly governed:

1. `tools/retrain_replay/`
2. `tools/retrain_analysis/`
3. workflow-oriented configuration and analysis material that is useful for the
   local testbed line but is not part of the core NWDAF runtime itself

For the currently identified retrain replay and retrain analysis tooling, this
plan fixes the outcome: they leave `NWDAF/` and move to the sibling workspace
repository `nwdaf-resources/`.

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

### 7.2 `internal/anlf`

`internal/anlf` should own:

1. the local inference-engine interface
2. the local inference-engine implementation package
3. model load / unload / prediction orchestration
4. tests for inference-engine client behavior and `AnLF` integration behavior

### 7.3 `internal/mtlf`

`internal/mtlf` should own:

1. the Daisy integration interface
2. the Daisy integration implementation package
3. Daisy async callback payload contract for the local training workflow
4. callback-to-workflow completion routing
5. tests for Daisy workflow and callback completion behavior

### 7.4 Repo Scope

The main `NWDAF/` repo should no longer host retrain replay or retrain analysis
tooling.

Those assets move to the sibling workspace repository:

- `nwdaf-resources/tools/retrain_replay/`
- `nwdaf-resources/tools/retrain_analysis/`

The end state must be intentional and documented, not accidental.

---

## 8. Scope

### 8.1 In Scope

1. moving Daisy client ownership out of `internal/sbi`
2. moving Daisy callback payload/handler ownership out of `internal/sbi`
3. renaming local `ML service` concepts to `inferenceEngine`
4. moving inference-engine client ownership out of `internal/sbi`
5. narrowing `processor` seams so `AnLF` and `MTLF` stop depending on `sbi`
   integration ownership
6. classifying and, where approved by the plan, relocating non-runtime support
   assets such as retrain replay and analysis tooling
7. updating tests and docs to match the new ownership boundaries

### 8.2 Conditionally In Scope

The following may be included only if they are required to preserve one
coherent end state:

1. narrow app/service assembly updates in `pkg/service/` if construction
   ownership must move out of `processor`

### 8.3 Explicitly Out Of Scope

1. NRF registration or metrics-server wiring
2. broader config-scope separation from Priority 9
3. OpenAPI/model-governance redesign from Priority 10
4. late-failure semantics redesign from Priority 6
5. changing the actual inference or retrain policy behavior unless required by
   the ownership move
6. broad renames of 3GPP-standardized API names in standardized callback or
   payload types

---

## 9. Proposed Workstreams

### Workstream A — Define The New Ownership Map

Objectives:

1. inventory which `internal/sbi` files currently own Daisy or inference
   integration responsibilities
2. assign each responsibility to `AnLF`, `MTLF`, `internal/sbi`, or repo
   support tooling
3. settle the local naming target for `inferenceEngine`

Required outcomes:

1. one ownership table exists before package moves begin
2. `ML service` rename target is explicit: `inferenceEngine`
3. any current inference-engine callback route is classified for complete
   removal from `internal/sbi`

### Workstream B — Move Inference Integration To `AnLF`

Objectives:

1. remove `MlServiceAPI` and the concrete client from `internal/sbi/consumer`
2. create an `AnLF`-owned inference seam and implementation package
3. stop having `processor` carry inference integration as an SBI concern

Required outcomes:

1. `internal/anlf` no longer imports inference integration from
   `internal/sbi/consumer`
2. local code and tests use inference-oriented naming
3. model load / unload / predict tests move with the new ownership

### Workstream C — Move Daisy Integration To `MTLF`

Objectives:

1. remove `DaisyServiceAPI` and the concrete Daisy client from
   `internal/sbi/consumer`
2. move Daisy callback payload and handling ownership to `internal/mtlf`
3. narrow or remove the processor-only Daisy callback bridge

Required outcomes:

1. Daisy workflow code lives under `internal/mtlf`
2. no Daisy-specific files remain under `internal/sbi`
3. Daisy route and handler ownership no longer sits under `internal/sbi`

### Workstream D — Re-narrow `internal/sbi`

Objectives:

1. leave `internal/sbi` with standards-facing NWDAF transport edges only
2. remove the remaining inference-engine and Daisy callback ingress from
   `internal/sbi`
3. remove package-local assumptions that `internal/sbi` owns all external HTTP
   integrations

Required outcomes:

1. `internal/sbi/consumer` contains only SBI-appropriate interactions after the
   move
2. `internal/sbi/server.go` no longer imports Daisy-specific or
   inference-engine-specific route logic
3. `internal/sbi` no longer owns inference-engine callback handling

### Workstream E — Classify Support Assets

Objectives:

1. inventory non-runtime support assets still present in `NWDAF/`
2. move retrain replay and retrain analysis tooling into `nwdaf-resources/`
3. document that the main runtime repo no longer owns those workflow tools

Required outcomes:

1. `tools/retrain_replay` no longer exists under `NWDAF/`
2. `tools/retrain_analysis` no longer exists under `NWDAF/`
3. `nwdaf-resources/` is initialized as its own repository and carries the
   moved tooling
4. supporting docs explain the intended home of the moved assets

---

## 10. Recommended Implementation Order

1. complete Workstream A first
2. implement Workstream B before Workstream C
3. implement Workstream C next
4. implement Workstream D immediately after the two ownership moves
5. finish with Workstream E once the code boundary is stable

Recommended commit grouping:

- `refactor(anlf): move inference engine integration out of sbi`
- `refactor(mtlf): move Daisy workflow integration out of sbi`
- `refactor(sbi): narrow transport ownership to standards-facing edges`
- `docs(repo): move workflow tooling into nwdaf-resources`

These subjects are examples, not mandatory final commit messages.

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
6. retrain replay and retrain analysis tooling no longer live in `NWDAF/` and
   are documented under `nwdaf-resources/`
7. verification shows no regression in the touched `AnLF`, `MTLF`, `SBI`, and
   service assembly paths

---

## 12. Verification Plan

Minimum focused verification for the touched code:

```bash
go test ./internal/anlf
go test ./internal/mtlf
go test ./internal/sbi
go test ./internal/sbi/processor
go test ./internal/sbi/consumer
```

Full verification target after the round:

```bash
make build
go test ./...
make lint
```

Review checks for this round:

1. does `internal/sbi` still own any Daisy-specific production code?
2. does `internal/sbi` still own any local inference-engine production code or
   callback routing?
3. did `AnLF` gain an integration seam without reintroducing package-global
   config or ad hoc client creation?
4. did `MTLF` gain Daisy ownership without reopening the completed lifecycle
   work from Priority 2?
5. are retained support assets intentionally documented instead of passively
   remaining in place?

Verification should include focused config tests proving that
`configuration.mlService` is rejected and `configuration.inferenceEngine` is
accepted.

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

---

## 14. Expected Outcome

After Priority 12, the repository should communicate a much clearer ownership
story:

1. `internal/sbi` means NWDAF SBI transport edge
2. `internal/anlf` owns inference-engine integration
3. `internal/mtlf` owns Daisy integration
4. `nwdaf-resources/` owns replay and analysis workflow tooling rather than the
   main `NWDAF/` repo

That outcome is narrower, easier to defend, and easier to extend than the
current "everything external HTTP-related lives under `internal/sbi`" shape.
