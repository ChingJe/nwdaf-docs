# NWDAF Main Project Strict free5GC Alignment Reassessment 2026-06-24

Date: 2026-06-24

## Purpose

This document records a stricter follow-up review of the current `NWDAF/`
implementation after the already-documented free5GC alignment rounds.

This pass has a different purpose from the earlier scan:

1. re-check the parts that have already been touched by
   `nwdaf-docs/docs/plans/free5gc-alignment/`
2. distinguish between:
   - work that is already planned but not started yet
   - work that was already touched but is still not strict enough
   - issues that are not yet written down in the current issue/plan set
3. review the project against current free5GC implementation conventions, not
   only against the repository's local internal consistency

This document is therefore a status-classification issue record, not a new
implementation plan.

## Scope

This reassessment reviewed:

- current `NWDAF/` implementation code
- `nwdaf-docs/docs/issues/free5gc/project_scan/`
- completed and in-progress work under
  `nwdaf-docs/docs/plans/free5gc-alignment/`
- `free5gc-dev-skill/` review and SBI/lifecycle guidance
- multiple local free5GC reference NFs under
  `resources/references/free5gc-main/NFs/`
- local 3GPP TS and OpenAPI YAML under `nwdaf-docs/specs/`

Primary free5GC comparison points used in this pass:

- `UDR`
- `UDM`
- `SMF`
- `NRF`
- `PCF`
- `UPF`

## Verification Rerun

Executed in `NWDAF/` on 2026-06-24:

- `go test ./...`
- `make build`
- `make lint`

Result:

- full Go test suite passed
- build passed
- lint passed with `0 issues`

Interpretation:

- the repository remains buildable and test-green
- the findings below are therefore primarily strict-alignment, boundary, and
  ownership findings rather than compile failures

## Classification Rules Used In This Document

### Class A — Already Written Down, Not Started Yet

The issue is already represented by an existing `Priority` or documented
follow-up item, and the current code still reflects that the work has not been
done yet.

This is not treated as a failed implementation.

### Class B — Already Touched, But Still Not Strict Enough

The issue is related to an area that was already implemented or declared
completed in the current `free5gc-alignment` plan set, but the resulting code
still leaves a meaningful alignment gap.

This can happen because:

- the implementation round intentionally only solved a narrower subset
- the completion wording overstates the actual closure
- the plan itself was not strict enough for the longer-term target

### Class C — Not Yet Written Down As Its Own Issue

The issues below were real in the current code at the time of this
reassessment. Some already had a named home; others needed one to be created in
the current `issues/` and `plans/free5gc-alignment/` set.

These are the most important additions from this reassessment.

## Summary Table

| ID | Topic | Class | Current Home In Existing Docs | Judgment |
| --- | --- | --- | --- | --- |
| 1 | broader lifecycle cancellation and app-owned I/O context | A | Priority 2 residual | already acknowledged, still open |
| 2 | remaining package-global config reads after Priority 4 | B | partially overlaps Priority 4 | resolved later on 2026-06-24 by the residual cleanup follow-up |
| 3 | non-3GPP external clients bypass shared consumer/app ownership | C | none | new issue |
| 4 | runtime config mixed with lab/workflow config | A | Priority 9 / Priority 12 | already known, not started |
| 5 | standalone SBI service vs free5GC NF lifecycle level | A | Priority 11 | already known, not started |
| 6 | OpenAPI/model governance still incomplete | A | Priority 10 | already known, not started |
| 7 | logging boundary and payload hygiene | A | Priority 7 | already known, not started |
| 8 | test ownership is still more package-local than surveyed free5GC baseline | C | dedicated Priority 3 strict follow-up plan | new strictness gap, now recorded |

## Detailed Findings

### 1. Broader lifecycle cancellation and app-owned I/O context remain open

Classification: Class A — already written down, not started yet

Existing home:

- `Priority 2 — Put Long-Running Work Under App Lifecycle Control`
- current status already says "Partially completed" in
  `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`

Why this is still open:

- the notifier root ownership problem was fixed
- however, many real runtime network and shutdown paths still derive timeout
  contexts from `context.Background()` instead of the app cancellation tree

Evidence in current code:

- service startup MongoDB ping and collection setup:
  - `NWDAF/pkg/service/init.go:137`
  - `NWDAF/pkg/service/init.go:151`
- notifier outbound notification send:
  - `NWDAF/internal/notifier/notifier.go:196`
- SMF consumer:
  - `NWDAF/internal/sbi/consumer/smf_service.go:124`
  - `NWDAF/internal/sbi/consumer/smf_service.go:169`
- MTLF provision consumer:
  - `NWDAF/internal/sbi/consumer/mtlf_service.go:75`
  - `NWDAF/internal/sbi/consumer/mtlf_service.go:104`
- Daisy client:
  - `NWDAF/internal/sbi/consumer/daisy_service.go:94`
  - `NWDAF/internal/sbi/consumer/daisy_service.go:140`
- ML service client:
  - `NWDAF/internal/sbi/consumer/ml_service.go:120`
  - `NWDAF/internal/sbi/consumer/ml_service.go:172`
  - `NWDAF/internal/sbi/consumer/ml_service.go:223`
- ADRF client:
  - `NWDAF/internal/sbi/consumer/adrf_service.go:171`
  - `NWDAF/internal/sbi/consumer/adrf_service.go:207`
  - `NWDAF/internal/sbi/consumer/adrf_service.go:256`
- SBI server shutdown:
  - `NWDAF/internal/sbi/server.go:163`

Strict free5GC alignment implication:

- this is not a new issue
- it should remain classified as unfinished lifecycle hardening
- the current code is better than the original scan state, but it still does
  not fully behave like an app-owned free5GC runtime tree

Recommended handling:

- keep this under the existing Priority 2 follow-up lineage
- do not relabel it as a new architecture issue

### 2. Package-global runtime config reads remain after the app-boundary round

Classification: Class B — already touched, but still not strict enough

Existing home:

- overlaps the completed `Priority 4 — Rebuild One Real App Boundary`
- explicitly related to the "global-state drift" already mentioned in that plan

Why this is a residual problem:

- Priority 4 correctly removed some important `factory.NwdafConfig` reads from
  consumer construction and processor-adjacent setup
- however, the current code still has many package-global config reads in
  runtime paths that already have access to the owning app

Evidence in current code:

- AnLF model load/hot-swap:
  - `NWDAF/internal/anlf/model.go:16`
  - `NWDAF/internal/anlf/model.go:51`
- AnLF monitor startup and ground-truth lookup:
  - `NWDAF/internal/anlf/monitor.go:22`
  - `NWDAF/internal/anlf/monitor.go:57`
  - `NWDAF/internal/anlf/monitor.go:263`
- MTLF callback URL, startup trigger, retrain flow, hot-swap guard:
  - `NWDAF/internal/mtlf/training.go:28`
  - `NWDAF/internal/mtlf/training.go:43`
  - `NWDAF/internal/mtlf/training.go:87`
  - `NWDAF/internal/mtlf/training.go:171`
- data collection and model-provisioning entrypoints:
  - `NWDAF/internal/sbi/processor/data_collection.go:52`
  - `NWDAF/internal/sbi/processor/data_collection.go:247`
- UPF notification persistence and ring-buffer sizing:
  - `NWDAF/internal/sbi/processor/upf_notify.go:299`
  - `NWDAF/internal/sbi/processor/upf_notify.go:309`

Why this matters:

- the current Priority 4 documents state a stricter target than the present
  code actually reaches
- the canonical app boundary exists, but runtime truth is still partly split
  between the app object and a package-global config singleton

Judgment:

- this should not be treated as "Priority 4 was useless"
- it should be treated as "Priority 4 materially improved the boundary, but the
  stricter closure claim is broader than the current implementation reality"

Recommended handling:

- record this as a residual Priority 4 strictness gap
- do not create a disconnected new issue unless the follow-up scope grows much
  larger than boundary cleanup

Resolution update later on 2026-06-24:

- the residual cleanup was then implemented in `NWDAF/`
- committed as `a912581` in the local `NWDAF/` repository
- the covered app-owned runtime paths in `internal/anlf`, `internal/mtlf`,
  `internal/sbi/processor`, and the adjacent notifier-owned UE analytics path
  no longer read `factory.NwdafConfig` directly
- this means Finding 2 was accurate at reassessment time, but should now be
  treated as resolved for its defined scope

### 3. Non-3GPP external clients still bypass the shared consumer/app ownership model

Classification: Class C — now recorded as a dedicated Priority 3 strict
follow-up plan

Why this is a distinct issue:

- after Priority 4, the project now has a recognizable shared app boundary and
  a real `internal/sbi/consumer` package
- however, several important external dependencies still bypass that boundary
  and are instantiated directly inside AnLF or MTLF

Evidence in current code:

- AnLF creates ML service clients directly:
  - `NWDAF/internal/anlf/model.go:23`
  - `NWDAF/internal/anlf/model.go:88`
- MTLF creates Daisy clients directly:
  - `NWDAF/internal/mtlf/training.go:129`

Why this is not the same as the older consumer-construction issue:

- the earlier Priority 4 problem was that the consumer package itself was not
  app-driven enough
- the current problem is that the project still has multiple external-client
  ownership styles:
  - standards-aligned or semi-standard NF interaction under `internal/sbi/consumer`
  - non-3GPP external systems instantiated ad hoc inside domain packages

Why this matters for the final goal:

- the user-stated goal is not only 3GPP compliance
- it is also "align with free5GC development conventions as much as possible"
- under that target, even project-local integrations should still have a clear
  ownership and dependency-injection story

Recommended handling:

- create a new follow-up issue for external-client ownership policy
- scope it around Daisy and ML service first
- ADRF may be folded into the same discussion if desired

### 4. Runtime config is still mixed with lab/workflow config and payload templates

Classification: Class A — already written down, not started yet

Existing home:

- `Priority 9 — Separate Runtime Config From Lab / Workflow Config`
- also related to the later repository-scope cleanup in `Priority 12`

Evidence in current code and config:

- one config file currently carries:
  - core SBI runtime config
  - SMF callback URIs
  - static group membership fallback data
  - MTLF startup behavior
  - Daisy task payload template
  - ADRF retrain retrieval workflow settings
- concrete examples:
  - `NWDAF/config/nwdafcfg.yaml:19`
  - `NWDAF/config/nwdafcfg.yaml:51`
  - `NWDAF/config/nwdafcfg.yaml:65`
  - `NWDAF/config/nwdafcfg.yaml:107`
  - `NWDAF/config/nwdafcfg.yaml:153`

Additional evidence in runtime code:

- hard-coded callback fallbacks still exist:
  - `NWDAF/internal/sbi/processor/data_collection.go:78`
  - `NWDAF/internal/sbi/processor/data_collection.go:79`
  - `NWDAF/internal/sbi/processor/data_collection.go:300`

Judgment:

- this is an expected open issue under the current plan set
- it should not be misclassified as a bad implementation of Priority 8
- Priority 8 correctly hardened the current config path; it did not claim to
  solve config-scope separation

### 5. The repository still behaves as a standalone SBI service, not a fuller free5GC NF

Classification: Class A — already written down, not started yet

Existing home:

- `Priority 11 — Decide The Intended free5GC Integration Level`

Evidence in current code and docs:

- README still documents HTTP-only standalone runtime behavior:
  - `NWDAF/README.md:10`
- `nrfUri` remains intentionally unwired:
  - `NWDAF/README.md:13`
  - `NWDAF/config/nwdafcfg.yaml:17`
- metrics server is still unwired:
  - `NWDAF/README.md:14`
- service startup and shutdown remain local-only:
  - `NWDAF/pkg/service/init.go:127`
  - `NWDAF/pkg/service/init.go:200`
- SBI server is still a lightweight local server rather than a free5GC helper
  path with inbound metrics middleware and TLS/H2C support:
  - `NWDAF/internal/sbi/server.go:81`
  - `NWDAF/internal/sbi/server.go:115`

Reference comparison:

- UDR app lifecycle includes registration, deregistration, metrics, and server
  ownership:
  - `resources/references/free5gc-main/NFs/udr/pkg/service/init.go:152`
  - `resources/references/free5gc-main/NFs/udr/pkg/service/init.go:165`
  - `resources/references/free5gc-main/NFs/udr/pkg/service/init.go:186`
- UDR server uses inbound metrics and shared server helpers:
  - `resources/references/free5gc-main/NFs/udr/internal/sbi/server.go:88`
  - `resources/references/free5gc-main/NFs/udr/internal/sbi/server.go:97`

Judgment:

- this is still a deliberate scope decision that has not been made yet
- it is not evidence that the completed earlier priorities were wrong
- it should remain explicitly parked under Priority 11

### 6. OpenAPI/model governance remains incomplete, but the remaining gap is now narrower

Classification: Class A — already written down, not started yet

Existing home:

- `Priority 10 — Establish OpenAPI / Model Governance`

Important clarification from this reassessment:

- not every handwritten model in current NWDAF should be treated as an outright
  mistake
- some of them exist because the current pinned dependency snapshot does not yet
  expose the exact required Release 18/19 or extension shape

Confirmed current evidence:

- SMF `UPF_EVENT` extension models remain local:
  - `NWDAF/internal/sbi/consumer/models.go:13`
- the local 3GPP YAML does include those fields:
  - `nwdaf-docs/specs/yaml/TS29508_Nsmf_EventExposure.yaml:510`
  - `nwdaf-docs/specs/yaml/TS29508_Nsmf_EventExposure.yaml:516`
  - `nwdaf-docs/specs/yaml/TS29508_Nsmf_EventExposure.yaml:527`
- ADRF retrieval callback still uses a local top-level payload:
  - `NWDAF/internal/sbi/api_adrf_notify.go:15`

Important narrowing:

- one previously-known duplication is already fixed:
  - ML Model Provision notification now uses generated
    `models.NwdafMlModelProvNotif`
  - see `NWDAF/internal/sbi/api_mlmodelprovision.go:22`

Judgment:

- this remains a valid open Priority 10 issue
- it should now be framed more precisely as:
  - identify which local models are dependency-gap bridges
  - identify which are project-local invention
  - document the regeneration and migration rule

### 7. Logging boundaries remain loose, but this is still an expected later cleanup

Classification: Class A — already written down, not started yet

Existing home:

- `Priority 7 — Tighten Logging Boundaries`

Evidence in current code:

- raw or near-raw outbound payload logging:
  - `NWDAF/internal/sbi/consumer/smf_service.go:167`
  - `NWDAF/internal/sbi/consumer/daisy_service.go:92`
- fatal logging in runtime control paths:
  - `NWDAF/pkg/service/init.go:177`
  - `NWDAF/pkg/service/init.go:187`
  - `NWDAF/internal/sbi/server.go:174`

Judgment:

- this is not new
- the current state is consistent with the already-documented "not started yet"
  cleanup bucket

### 8. Test ownership is still more package-local than the surveyed free5GC baseline

Classification: Class C — not yet written down as its own issue

Why this matters:

- Priority 3 correctly improved coverage and seam quality
- however, the current ownership shape of mocks and test seams is still more
  local and ad hoc than the surveyed free5GC baseline

Evidence in current NWDAF tree:

- `internal/sbi` keeps generated mocks locally:
  - `NWDAF/internal/sbi/mock_interfaces_test.go:1`
- `internal/sbi/processor` keeps its own local mock seam:
  - `NWDAF/internal/sbi/processor/mock_interfaces_test.go:15`
- handler tests use a package-local fake app:
  - `NWDAF/internal/sbi/api_test_helpers_test.go:20`

Reference comparison:

- surveyed free5GC NFs commonly expose reusable app-level mocks under app or
  service packages:
  - `resources/references/free5gc-main/NFs/smf/pkg/app/mock.go:1`
  - `resources/references/free5gc-main/NFs/udm/pkg/app/mock.go:1`
- some NFs also keep SBI-local mocks where the seam is SBI-specific:
  - `resources/references/free5gc-main/NFs/udr/internal/sbi/server_mock.go:1`

Important clarification:

- the mere existence of `NWDAF/internal/sbi/mock_interfaces_test.go` is **not**
  by itself a free5GC-alignment defect
- free5GC reference NFs also sometimes keep SBI-local generated mocks
- the stricter issue is that NWDAF still lacks a cleaner shared mock ownership
  pattern for the now-canonical app boundary

Judgment:

- this should not be elevated above lifecycle, config-scope, or integration
  level decisions
- but it is a real strictness gap that needed a clearer named home than the
  earlier completed Priority 3 wording provided

Recommended handling:

- record it as a narrower strict follow-up in the Priority 3 lineage rather
  than under Priority 12 repository/package ownership cleanup
- the dedicated plan is now:
  `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 3 Strict Test Ownership Alignment Plan.md`

## Clarifications And Non-Findings

### `internal/sbi/mock_interfaces_test.go` is not, by itself, proof of bad alignment

The user explicitly raised this file as a concern.

This reassessment does **not** classify the file itself as a confirmed
free5GC-style defect.

Reason:

- reference free5GC trees also contain SBI-local generated mocks, for example:
  - `resources/references/free5gc-main/NFs/udr/internal/sbi/server_mock.go`

The more accurate issue is the broader test-ownership pattern described in
Finding 8, not the simple presence of one mock file under `internal/sbi/`.

### The lack of a standalone `router.go` file is not treated as a defect

Some free5GC NFs use `router.go` or `routes.go`, but not all do.

The current NWDAF route registration being embedded in `internal/sbi/server.go`
is therefore not treated as a standalone misalignment issue in this pass.

### The Daisy callback being exposed under `internal/sbi` is not the core problem

The HTTP binding itself belongs at the SBI edge.

The more important question is ownership behind that edge:

- contract classification
- client ownership
- workflow/package ownership
- config scope

Those are already captured above.

### `UPF_EVENT` local extension support is not classified as an immediate defect

The local TS/YAML material confirms that the project is not inventing the
`upfEvents`, `bundlingAllowed`, and `bundledEventNotifyUri` concepts from
nothing.

This remains governance work, not a direct bug report.

## Recommended Issue Mapping After This Reassessment

### Keep under existing issue/plan lineage

1. broader lifecycle cancellation and app-owned I/O context
   - keep under Priority 2 residual work
2. runtime config vs lab/workflow config separation
   - keep under Priority 9
3. integration-level decision about NRF/metrics/full NF behavior
   - keep under Priority 11
4. OpenAPI/model governance
   - keep under Priority 10
5. logging boundary cleanup
   - keep under Priority 7
6. stricter test-ownership alignment around shared app mocks and local mock
   placement
   - keep under the dedicated Priority 3 strict follow-up plan:
     `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 3 Strict Test Ownership Alignment Plan.md`

### Re-open or annotate completed areas with stricter residual notes

1. remaining package-global config reads after Priority 4
   - annotate Priority 4 completion status with a stricter residual note

### Add as new issues not yet clearly written down

1. external-client ownership policy for Daisy / ML service style integrations

## Final Judgment

The most important distinction from this reassessment is:

1. several open problems are open simply because those priorities were never
   claimed as finished
2. one important already-touched area, the app-boundary/global-config story,
   is materially improved but not yet as strict as the completed Priority 4
   wording suggests
3. a smaller set of structural issues needed clearer naming in the current
   issue set
4. after later same-day documentation updates, stricter test/mock ownership
   now has a dedicated Priority 3 follow-up home, while non-3GPP external
   client ownership still does not

This means the current document set is directionally correct, but not yet a
complete record of the remaining work needed to approach the user's stated
final goal of being as close as practical to free5GC development conventions.
