# NWDAF Main Project free5GC Alignment Scan Plan

---

## 1. Purpose

This plan defines how to perform a full implementation scan of the main
`NWDAF/` repository against:

1. the local `free5gc-dev-skill` review workflow and guardrails
2. the actual structure and coding patterns present in
   `resources/references/free5gc-main/NFs/`
3. the local NWDAF development policy and repository conventions

The scan is meant to identify structural, procedural, lifecycle, contract,
configuration, and testing problems that may have accumulated because the
project was developed by a single non-upstream contributor without strict
free5GC convention discipline from the start.

This document plans the scan only. It does not perform refactoring, code
changes, or report final findings yet.

---

## 2. Classification And Storage Location

This plan is stored under:

```text
nwdaf-docs/docs/plans/free5gc/project_scan/
```

Reasoning:

- The work is not feature design for `ANLF`, `MTLF`, `ADRF`, or `Daisy`.
- The main comparison baseline is free5GC implementation practice rather than
  only NWDAF feature behavior.
- The output is a repository-wide scan plan, not a narrow implementation task.

Future documents produced by the same workstream should remain under this
classification when they are primarily about free5GC-aligned repository
inspection, scan findings, remediation sequencing, or follow-up review passes.

Planning documents stay under:

```text
nwdaf-docs/docs/plans/free5gc/project_scan/
```

Issue and findings outputs should instead use a new issue classification under:

```text
nwdaf-docs/docs/issues/free5gc/project_scan/
```

---

## 3. Scope

### 3.1 In Scope

- The main implementation repository: `NWDAF/`
- Package structure and responsibility boundaries
- SBI request and response flow
- Processor and consumer separation
- Context ownership, runtime state, and persistence boundaries
- Configuration, startup, shutdown, concurrency, and observability
- OpenAPI and 3GPP contract alignment where the implementation depends on them
- Test layout, verification flow, and repository hygiene relevant to maintainability

### 3.2 Out Of Scope For This Scan Pass

- Rewriting the implementation during the scan
- Updating 3GPP study notes or skill documentation as part of the same change
- Treating local reference repositories as implementation targets
- Forcing NWDAF to imitate protocol-heavy NF internals that are not applicable
- Judging runtime correctness only from structure without tracing concrete flows

---

## 4. Reference Hierarchy

The scan should use references in this order when evaluating a question:

1. Existing `NWDAF/` code and repository-local conventions
2. `nwdaf-docs/docs/development_policy.md`
3. `free5gc-dev-skill/SKILL.md`
4. Relevant files under `free5gc-dev-skill/references/`
5. `resources/references/free5gc-main/NFs/`
6. `nwdaf-docs/specs/yaml/`
7. Relevant TS text under `nwdaf-docs/specs/`

Important interpretation rule:

- The free5GC skill provides the review vocabulary and trace discipline.
- The reference NFs provide the concrete implementation shape.
- The local NWDAF repository remains the direct subject under review.

---

## 5. Comparison Baselines From free5GC Main

The scan must not compare NWDAF against only one NF. Different free5GC NFs
demonstrate different acceptable patterns.

### 5.1 Primary Positive Baselines

- `NFs/nrf`
  - Compact control-plane NF baseline
  - Clear `cmd -> pkg/service -> internal/sbi -> processor/consumer/context`
  - Useful for startup, lifecycle, and standard SBI layering

- `NFs/udr`
  - Strong example of `server.go` plus separate `router.go`
  - Useful for route construction, auth middleware placement, HTTP concerns,
    and dense processor-per-operation organization

- `NFs/udm`, `NFs/ausf`, `NFs/bsf`, `NFs/nssf`, `NFs/pcf`, `NFs/nef`
  - Additional control-plane comparators for config, service wiring,
    handler/processor boundaries, and generated-model usage

### 5.2 Specialized Baselines

- `NFs/amf`
  - Demonstrates that extra internal domain packages can be valid when they
    represent protocol-specific layers such as `nas`, `ngap`, and `gmm`
  - Useful to avoid wrongly judging every non-`sbi` package as a design flaw

- `NFs/smf`
  - Demonstrates a control-plane NF with substantial non-SBI responsibilities
    such as `pfcp`
  - Useful when judging whether NWDAF-specific packages are legitimate
    interface/domain modules or misplaced business logic

### 5.3 Negative Or Guardrail Baseline

- `NFs/upf`
  - Should not be used as a structural baseline for NWDAF
  - Exists mainly as a reminder that data-plane-driven designs must not be
    used to justify control-plane SBI shortcuts

---

## 6. Scan Objectives

The scan should produce a defensible answer to the following questions:

1. Does `NWDAF/` still follow a recognizable free5GC-style control-plane NF
   structure?
2. Where do current files or flows violate expected handler, processor,
   consumer, service, factory, or context boundaries?
3. Which domain-specific packages are justified, and which appear to be
   overflow areas caused by missing architectural discipline?
4. Where does the project diverge from free5GC implementation practice in ways
   that increase regression risk, review difficulty, or maintenance cost?
5. Which issues are confirmed defects, which are structural risks, and which
   are verification gaps only?
6. What remediation batches should later be prioritized, and in what order?

---

## 7. Expected Deliverables

The full scan should eventually produce:

1. A findings report ordered by severity
2. File-level evidence with exact references
3. A distinction between confirmed problems, probable risks, and open questions
4. A remediation grouping plan
5. A verification gap summary describing what cannot be proven locally

Suggested follow-up document placement:

```text
nwdaf-docs/docs/issues/free5gc/project_scan/
```

Examples:

- `NWDAF Main Project free5GC Alignment Scan Findings.md`
- `NWDAF Main Project Remediation Batches.md`

---

## 8. Scan Method

### 8.1 General Rules

- Trace behavior from entrypoints inward before judging structure.
- Compare against actual free5GC NF code, not only abstract skill statements.
- Do not label a difference as a defect unless the responsibility split or
  lifecycle implication is clear.
- Separate code-quality concerns from repository-hygiene concerns.
- Record whether each concern is:
  - confirmed issue
  - likely risk
  - open question needing deeper verification

### 8.2 Evidence Style

Each finding should capture:

- affected file
- affected function or type
- trace path
- free5GC comparison target
- why the current shape is risky or inconsistent
- whether there is a user-visible or maintenance impact
- whether tests currently cover the path

---

## 9. Planned Scan Phases

### 9.1 Phase A: Repository Orientation And Baseline Mapping

Purpose:

- Build a concrete map of the NWDAF repository before reviewing behavior.

NWDAF targets:

- `README.md`
- `Makefile`
- `go.mod`
- `cmd/`
- `pkg/`
- `internal/`
- `config/`
- `test/`
- `.golangci.yml`

free5GC comparisons:

- `NFs/nrf`
- `NFs/udr`
- `NFs/udm`

Questions:

- Is the repository shaped like a control-plane NF or drifting toward a
  project-specific layout?
- Are build, lint, and test entrypoints coherent?
- Are there obvious mismatches between README, `go.mod`, Makefile, and actual
  repository contents?
- Are local development artifacts committed into the repo in ways that differ
  from normal free5GC repository hygiene?

Primary output:

- A repository map and a list of initial hotspots for deeper tracing

### 9.2 Phase B: Startup, Service Wiring, And Lifecycle Ownership

Purpose:

- Confirm that application lifecycle responsibilities are placed in the right
  layers and that startup/shutdown flow is coherent.

Trace:

```text
cmd/main.go
-> pkg/factory/config.go
-> pkg/service/init.go
-> internal/context/*
-> internal/sbi/server.go
-> shutdown path
```

free5GC comparisons:

- `NFs/nrf/pkg/service/init.go`
- `NFs/udr/pkg/service/*`
- `NFs/ausf/pkg/service/*`

Questions:

- Does `cmd` only bootstrap and start the app?
- Does `pkg/service` own lifecycle rather than leaking it into handlers or
  domain packages?
- Is context initialized before processors and consumers depend on it?
- Are MongoDB setup, schedulers, server startup, and termination owned by the
  right layer?
- Can shutdown hang, skip cleanup, or bypass goroutine ownership?

Potential extra problem classes to watch:

- global mutable state
- service methods doing too much unrelated work
- shutdown paths that do not mirror startup paths

### 9.3 Phase C: SBI Server, Router, And Handler Boundaries

Purpose:

- Review the inbound HTTP/SBI boundary against free5GC control-plane norms.

Trace:

```text
internal/sbi/server.go
-> route registration
-> api_*.go
-> processor
-> response
```

NWDAF targets:

- `internal/sbi/server.go`
- `internal/sbi/api_eventssubscription.go`
- `internal/sbi/api_collector.go`
- `internal/sbi/api_mlmodelprovision.go`
- `internal/sbi/api_daisy_callback.go`
- `internal/sbi/api_adrf_notify.go`

free5GC comparisons:

- `NFs/nrf/internal/sbi/server.go`
- `NFs/nrf/internal/sbi/router.go`
- `NFs/udr/internal/sbi/server.go`
- `NFs/udr/internal/sbi/router.go`

Questions:

- Are server, router, and handler responsibilities separated clearly enough?
- Does the lack or presence of a separate `router.go` create real coupling?
- Do handlers stay at HTTP boundary level?
- Are `ProblemDetails`, status codes, and JSON responses handled consistently?
- Are middleware concerns placed at the right layer?

Priority hotspot:

- NWDAF currently appears to register routes directly in `server.go`; this
  should be reviewed against free5GC patterns but not treated as an issue by
  default until coupling impact is proven.

### 9.4 Phase D: Processor, Consumer, And Procedure Ownership

Purpose:

- Determine whether business procedures, outbound calls, and orchestration are
  placed in the intended layers.

Trace:

```text
api_*.go
-> internal/sbi/processor/*
-> internal/sbi/consumer/*
-> internal/context/* or domain packages
```

free5GC comparisons:

- `NFs/nrf/internal/sbi/processor/*`
- `NFs/nrf/internal/sbi/consumer/*`
- `NFs/udr/internal/sbi/processor/*`
- `NFs/udm/internal/sbi/consumer/*`

Questions:

- Does processor code own the actual procedure logic?
- Are outbound peer NF or external service calls isolated in consumers?
- Do handlers bypass processors?
- Do processors bypass consumers with raw HTTP logic?
- Are return types aligned with generated models and local helper patterns?

Potential extra problem classes to watch:

- domain logic split across multiple unrelated packages without a clear owner
- hidden HTTP logic outside consumers
- duplicated validation across handlers and processors

### 9.5 Phase E: Domain Package Legitimacy Review

Purpose:

- Judge whether NWDAF-specific internal packages represent valid domain modules
  or architectural overflow.

NWDAF targets:

- `internal/anlf/`
- `internal/mtlf/`
- `internal/notifier/`

free5GC comparisons:

- `NFs/amf/internal/{gmm,nas,ngap}`
- `NFs/smf/internal/pfcp`
- `NFs/nrf/internal/*`

Questions:

- Does each domain package have a clear interface/domain reason to exist?
- Is code in these packages truly NF-private and cohesive?
- Have they absorbed responsibilities that should remain in processor,
  consumer, service, or context?
- Are their boundaries understandable without tracing unrelated packages?

This phase is critical because NWDAF already contains more domain-specific
packages than a minimal control-plane NF baseline.

### 9.6 Phase F: Context, Runtime State, And Persistence Boundaries

Purpose:

- Review how runtime state is owned, mutated, and persisted.

NWDAF targets:

- `internal/context/context.go`
- `internal/context/db_models.go`
- `internal/context/db_query.go`
- `internal/context/accuracy_store.go`
- `internal/context/ml_model.go`
- `internal/context/traffic_data.go`
- `internal/context/group_resolver.go`

free5GC comparisons:

- `NFs/nrf/internal/context/*`
- `NFs/udr/internal/context/*`
- `NFs/smf/internal/context/*`

Questions:

- Is runtime state centralized in context or scattered across packages?
- Is global state used where instance-owned state would be safer?
- Are persistence concerns mixed with unrelated business logic?
- Are in-memory store semantics explicit?
- Are context structures aligned with actual lifecycle ownership?

Potential extra problem classes to watch:

- state duplication
- unclear cache invalidation rules
- DB helpers that are too aware of procedure semantics

### 9.7 Phase G: Configuration, Defaults, And Wiring Consistency

Purpose:

- Validate that config schema, defaults, sample config, and runtime wiring stay
  aligned.

NWDAF targets:

- `pkg/factory/config.go`
- `config/nwdafcfg.yaml`
- config-driven service wiring throughout `pkg/service` and `internal/*`

free5GC comparisons:

- `NFs/nrf/pkg/factory/*`
- `NFs/amf/pkg/factory/*`
- `NFs/smf/pkg/factory/*`

Questions:

- Are config structs and YAML layout coherent?
- Are default values and helper methods traceable and non-contradictory?
- Is validation present where the repository depends on it?
- Are config values read in one layer but enforced in another without clear rules?
- Are there toolchain or version consistency issues between README, `go.mod`,
  and expected workflows?

Potential extra problem classes to watch:

- configuration bloat without ownership boundaries
- runtime state leaking into config structs
- weak validation for cross-field constraints

### 9.8 Phase H: External Interaction And Contract Alignment

Purpose:

- Review the repository's usage of standardized models, API contracts, and
  external service integrations.

NWDAF targets:

- `internal/sbi/api_*.go`
- `internal/sbi/processor/*`
- `internal/sbi/consumer/*`
- custom output or adapter models such as `internal/notifier/models_output.go`

free5GC comparisons:

- control-plane NF processors and consumers across `nrf`, `udm`, `udr`, `pcf`
- local `github.com/free5gc/openapi` dependency usage in `NWDAF/go.mod`
- local OpenAPI YAML in `nwdaf-docs/specs/yaml/`

Questions:

- Are generated models used where they should be?
- Are custom structs introduced without a strong boundary reason?
- Do routes, methods, status codes, and error bodies align with contract sources?
- Are external service adapters separated from standardized SBI payload handling?

Potential extra problem classes to watch:

- ad hoc request/response models
- weak traceability to 3GPP terminology
- conversion layers that duplicate standardized models unnecessarily

### 9.9 Phase I: Concurrency, Scheduling, Shutdown, And Observability

Purpose:

- Review goroutine ownership, stop conditions, timers, and logging/metrics
  patterns.

NWDAF targets:

- `pkg/service/init.go`
- scheduler start points in processors
- monitoring or training loops in `internal/anlf/` and `internal/mtlf/`
- server shutdown behavior
- logging usage in `internal/logger/`

free5GC comparisons:

- `NFs/nrf/pkg/service/init.go`
- lifecycle and server patterns across `udr`, `udm`, `ausf`

Questions:

- Can every goroutine stop?
- Is WaitGroup ownership consistent?
- Are network calls bounded by context or timeout?
- Are logs emitted at meaningful boundaries rather than duplicated everywhere?
- Are observability hooks integrated in the service layer or scattered?

Potential extra problem classes to watch:

- schedulers started without symmetric stop behavior
- shutdown paths using background contexts where request-scoped cancellation
  should matter
- logs that expose too much low-level implementation detail

### 9.10 Phase J: Tests, Verification Strategy, And Repository Hygiene

Purpose:

- Review whether the repository can realistically support safe refactoring and
  whether committed artifacts match the expected development workflow.

NWDAF targets:

- package-level `*_test.go`
- `test/scripts/test_api.sh`
- `test/fake_*`
- local virtual environments, caches, or generated runtime artifacts under `test/`

free5GC comparisons:

- package-local tests in `NFs/*`
- free5GC CI-oriented repo cleanliness and module-focused test patterns

Questions:

- Are tests aligned with architectural layers?
- Which critical flows have no direct package-level test coverage?
- Are helper test services separated cleanly from repository runtime artifacts?
- Do local committed artifacts suggest workflow drift that will complicate future review?

Priority hotspot:

- Repository-hygiene items such as committed `.venv` or `__pycache__` should be
  recorded separately from architectural findings, but they still matter for
  maintainability and review quality.

### 9.11 Phase K: Findings Synthesis And Remediation Grouping

Purpose:

- Convert detailed observations into a sequence that can later be acted on.

Output groups should distinguish:

- architecture boundary issues
- SBI flow issues
- lifecycle and concurrency risks
- config and contract mismatches
- test and hygiene gaps

Each issue should also be tagged by likely remediation style:

- localized fix
- package reshaping
- cross-cutting cleanup
- verification-only follow-up

---

## 10. Initial Hotspots Already Worth Prioritizing In The Scan

These are not final findings. They are reasons to prioritize certain traces:

- `internal/sbi/server.go` appears to include route registration directly,
  unlike several free5GC control-plane NFs that separate router construction.
- `pkg/service/init.go` already appears to own MongoDB setup, shutdown
  listening, scheduler startup, and SBI server startup in one place, which may
  be valid but warrants careful lifecycle review.
- `internal/anlf/`, `internal/mtlf/`, and `internal/notifier/` indicate strong
  domain growth and need legitimacy review against AMF/SMF-style exceptions.
- `test/` contains local environment artifacts that do not resemble normal
  free5GC repository cleanliness and should be tracked as repository-hygiene
  concerns.
- `README.md`, `go.mod`, and actual toolchain assumptions may need a dedicated
  consistency check.

---

## 11. Execution Order

Recommended order for the full scan:

1. Phase A: repository orientation
2. Phase B: startup and lifecycle
3. Phase C: SBI inbound boundaries
4. Phase D: processor and consumer ownership
5. Phase E: domain package legitimacy
6. Phase F: context and persistence boundaries
7. Phase G: config and wiring
8. Phase H: contract alignment
9. Phase I: concurrency and observability
10. Phase J: tests and hygiene
11. Phase K: findings synthesis

Reasoning:

- Early phases establish the architecture map needed to judge later details.
- Domain-package review should happen only after request and lifecycle flows are
  understood.
- Findings should not be synthesized before contract, concurrency, and test
  gaps are inspected.

---

## 12. Completion Criteria

The scan is complete when:

- every major runtime entrypoint has been traced end-to-end
- every major internal package has been assigned a structural judgment
- free5GC comparisons have been made against actual NF code, not memory
- findings distinguish confirmed issues from risks and open questions
- remediation work can be sequenced without repeating the scan from scratch
