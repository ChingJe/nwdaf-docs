# NWDAF Priority 14 HTTP Edge Shape And Flow Alignment Plan

Date: 2026-07-06

Status: Current active tranche completed in code (Phase 1, Phase 1.5, and the current-stage Phase 2 structural alignment are completed)

Historical remediation item:

- `Priority 14 — Align HTTP Edge Shape And Flow Ownership`

Current execution note:

- this plan starts from the already-completed Priority 12 topology split
- the three-server semantic boundary is treated as fixed input, not as a design
  question reopened by this plan
- the next work is about implementation shape, not about collapsing the
  topology back into one shared server
- Phase 1 completed in `NWDAF/` on 2026-07-06 as commit `e8e249a`
- Phase 1.5 then landed in `NWDAF/` on 2026-07-06 across commits `8762b35`
  and `a0fff93`
- the current-stage Phase 2 auxiliary structural alignment then landed in
  `NWDAF/` on 2026-07-06 as commit `1b06411`
- the current active tranche is now complete in code
- any later deeper auxiliary-processor consolidation remains future work rather
  than part of the completed current-stage target

Related issue records:

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project HTTP Edge Shape Alignment Item.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`

---

## 1. Purpose

This plan defines the next alignment round after the completed Priority 12
server-topology split.

The purpose of this round is to make the current three-server runtime shape in
`NWDAF/` look deliberate and internally consistent:

1. the main `internal/sbi` server should look closer to the chosen free5GC SBI
   exemplar shape, with `pcf` as the primary reference
2. the main `internal/sbi` server should also close the immediate HTTPS/TLS gap
   in a way that still looks like a normal free5GC SBI edge
3. `internal/anlf` and `internal/mtlf` should keep their non-SBI semantics, but
   their later shape-alignment work is now explicitly deferred instead of being
   part of the current active tranche
4. inbound HTTP ownership and outbound HTTP ownership should be expressed with
   one stable repository rule:
   - `api -> processor -> consumer` for SBI-facing flows
   - `api -> processor -> client` for non-SBI auxiliary flows

This round is about shape, layering, and ownership consistency.

It is not about:

1. reverting the three-server split from Priority 12
2. deciding that `AnLF` or `MTLF` should become SBI surfaces
3. introducing a shared generic base HTTP server abstraction
4. reopening Priority 11 NRF registration, metrics-server wiring, or broader
   full-NF lifecycle uplift
5. changing analytics behavior, retraining behavior, or callback semantics
   unless a narrow implementation seam requires it
6. forcing Phase 2 auxiliary-server refactoring into the same active execution
   tranche as the `SBI` HTTPS uplift

---

## 2. Fixed Decisions And Constraints

The following decisions are already made and should be treated as hard
constraints during implementation:

1. three owned listeners remain:
   - one main `SBI` listener
   - one `AnLF` auxiliary listener
   - one `MTLF` auxiliary listener
2. `collector` remains part of the main `internal/sbi` surface because it
   receives outward peer-NF callback / collection traffic through the main
   NWDAF HTTP edge
3. Phase 1 is limited to `internal/sbi` and should use `pcf` as the primary
   shape reference
4. Phase 1 should preserve the current `startOwnedServers()` synchronous
   startup-failure cleanup behavior even while the `internal/sbi` transport
   skeleton becomes more `pcf`-like
5. the selected Phase 1 startup approach is:
   - construct the server with `httpwrapper.NewHttp2Server(...)`
   - keep the `ListenAndServe()`-style serving path
   - add a thin synchronous preflight bind check before launching the serving
     goroutine
   - wait for a short startup readiness handshake before `Run()` reports
     success
6. Phase 2 should move `internal/anlf` and `internal/mtlf` toward
   `api -> processor -> client`
7. `consumer` naming remains reserved for SBI-facing or standards-facing
   service consumption
8. `client` naming should be used for non-SBI auxiliary integrations such as
   the inference engine and Daisy
9. the goal is to unify shape and ownership style, not to unify semantics
10. no cross-package shared base HTTP server should be introduced in this round
11. Phase 1.5 is limited to the main `internal/sbi` listener plus its
    `collector` routes
12. `internal/anlf` and `internal/mtlf` remain unchanged in Phase 1.5 and do
    not need HTTPS support in that round
13. Phase 1.5 should keep free5GC-style dual-mode behavior, with
    `configuration.sbi.scheme` selecting `http` or `https`
14. Phase 1.5 config shape should align to free5GC NF YAML style by using
    `configuration.sbi.tls.pem` and `configuration.sbi.tls.key`
15. Phase 1.5 should follow the free5GC-style resource layout expectation of
    sibling `config/` and `cert/` directories, with YAML using `cert/...`
    relative paths
16. dev/test certificate assets for this round are expected to live in
    `nwdaf-resources/` rather than in the main `NWDAF/` repository
17. callback URI scheme should stay consistent with `sbi.scheme` for the
    currently-owned NWDAF callback surface
18. current completion proof should emphasize automated config and local TLS
    listener tests; a full multi-NF runtime smoke test is useful later but is
    not required in the present environment
19. Phase 1.5 should continue using `pcf` as the primary `SBI` server
    exemplar; `amf`, `udm`, and `smf` should be treated as supporting
    references for config-shape and sibling-consistency checks rather than as
    alternative primary server-skeleton drivers

These decisions narrow the implementation space on purpose. The remaining work
is execution, not a fresh architecture vote.

---

## 3. Planning Basis

This plan is based on the following local sources:

- `AGENTS.md`
- `nwdaf-docs/docs/development_policy.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project HTTP Edge Shape Alignment Item.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 12 Domain Boundary And Support Asset Cleanup Plan.md`
- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/exemplar-alignment.md`
- `free5gc-dev-skill/references/nf-architecture.md`
- `free5gc-dev-skill/references/sbi-development.md`
- `free5gc-dev-skill/references/config-and-lifecycle.md`
- `resources/references/free5gc-main/NFs/pcf/internal/sbi/server.go`
- `resources/references/free5gc-main/NFs/amf/internal/sbi/server.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/server.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/consumer/consumer.go`
- `resources/references/free5gc-main/NFs/nef/internal/sbi/server.go`
- `resources/references/free5gc-main/config/smfcfg.yaml`
- `resources/references/free5gc-main/config/amfcfg.yaml`
- `resources/references/free5gc-main/config/udmcfg.yaml`
- the current `NWDAF/` tree on 2026-07-06

---

## 4. Exemplar Choice And Interpretation

### 4.1 Why `pcf` Is The Primary `sbi` Shape Reference

The chosen local reference for `internal/sbi` is `pcf`.

This is not because `pcf` is the only valid free5GC server shape. It is because
it matches the intended direction most closely:

1. one `Server` type owns the router and HTTP server
2. route groups are assembled directly in `server.go`
3. handler methods live at the server boundary and delegate inward to a
   processor
4. outbound service usage follows the `consumer` pattern
5. app lifecycle owns consumer, processor, and server assembly

This makes `pcf` a good reference for Phase 1 because the target is not a
generic `newRouter(...)` rewrite. The target is a cleaner `sbi` server skeleton
that still looks like a free5GC NF HTTP edge.

### 4.2 What This Plan Learns From Other Exemplars

`amf`, `udm`, `nrf`, and `nef` are still useful, but as supporting evidence:

1. `amf` and `udm` show that a `newRouter(...)` helper is common but not
   mandatory
2. `nrf` shows that route registration may also be factored into an
   `applyService()` helper while still staying in the same server package
3. `udm` and other NFs confirm that outbound standards-facing HTTP usually
   lives under `internal/sbi/consumer`
4. none of the surveyed NF exemplars suggest turning project-local auxiliary
   HTTP integrations into fake SBI services just for shape uniformity

The intended inference is therefore:

1. `sbi` should become more exemplar-aligned
2. `anlf` and `mtlf` should become more structurally consistent
3. but non-SBI flows should stay explicitly non-SBI

### 4.3 Phase 1.5 Exemplar Rule

Phase 1.5 should not reopen the primary exemplar choice.

The intended reference split is:

1. `pcf` remains the primary exemplar for the `SBI` server skeleton,
   transport-branch shape, and lifecycle form
2. `amf`, `udm`, and `smf` are supporting references for the operator-facing
   config shape, `tls.pem/key` conventions, and sibling-NF consistency checks
3. supporting references may sharpen the config and verification details, but
   they should not displace `pcf` as the main `server.go` direction

---

## 5. Current NWDAF State

### 5.1 What Priority 12 Already Solved

Priority 12 already solved the semantic server-boundary problem:

1. `pkg/service` owns three listeners
2. `internal/sbi` is no longer used as a late route-injection hub for `AnLF`
   and `MTLF`
3. `internal/anlf/server.go` and `internal/mtlf/server.go` own their own
   ingress transport
4. callback URIs for the touched `AnLF` and `MTLF` flows derive from owned
   auxiliary-server config

This current plan must not undermine those decisions.

### 5.2 Remaining Active Gaps In `internal/sbi` After Phase 1

Phase 1 already closed the broader `internal/sbi` server-shape divergence:

1. Gin construction now follows the free5GC logger-helper style
2. inbound metrics middleware is now attached
3. server construction now uses `httpwrapper.NewHttp2Server(...)`
4. the owned-startup lifecycle contract is preserved with preflight bind plus
   readiness confirmation

The active remaining gap inside `internal/sbi` is now narrower:

1. the `https` branch remains intentionally incomplete
2. config and certificate-path shape are still not aligned to the common
   free5GC `sbi.tls.pem/key` plus `config/` and `cert/` convention

This means the main `SBI` edge is no longer waiting for a broad shape cleanup.
It is waiting for transport-completion and config-shape completion.

### 5.3 Remaining Shape Gaps In `AnLF` And `MTLF`

Current `internal/anlf` and `internal/mtlf` each already own the right ingress
surface, but their implementation shape is still minimal:

1. routes are registered directly against service methods
2. HTTP body parsing and transport response shaping still live on service-owned
   callback handlers
3. outbound integrations matter heavily in both domains, but are not yet
   expressed under one deliberate `api -> processor -> client` model

This leaves the repository with correct semantics but an incomplete shape story.

---

## 6. Target Architecture Outcome

Across the documented full roadmap, the intended repository shape is:

### 6.1 `internal/sbi`

- one `Server`
- free5GC-style router / middleware / HTTP-server assembly
- handlers owned at the HTTP edge
- `processor` as the inward orchestration owner
- `consumer` as the standards-facing outbound owner

Canonical shape:

`server -> api handler -> processor -> consumer`

### 6.2 `internal/anlf`

- one auxiliary `Server`
- handlers separated from service/domain logic
- a dedicated processor shape for inbound HTTP-triggered workflows
- non-SBI outbound integrations expressed as `client`

Canonical shape:

`server -> api handler -> processor -> client`

### 6.3 `internal/mtlf`

- one auxiliary `Server`
- handlers separated from service/domain logic
- a dedicated processor shape for inbound HTTP-triggered workflows
- non-SBI outbound integrations expressed as `client`
- standards-facing reuse such as ADRF access may still enter through an
  existing narrower seam, but outward non-SBI integrations should remain named
  as `client`

Canonical shape:

`server -> api handler -> processor -> client`

### 6.4 What Must Not Happen

The following outcomes are explicitly out of bounds:

1. renaming all HTTP servers to one generic shared abstraction
2. moving `AnLF` or `MTLF` callback ingress back under the main SBI server
3. applying SBI-specific auth / naming / service-registration assumptions to
   `AnLF` or `MTLF`
4. building one shared `baseHTTPServer` that forces all three packages to share
   transport semantics

---

## 7. Phase 1 — `internal/sbi` Aggressive Shape Alignment

### 7.1 Objective

Phase 1 should complete the `internal/sbi` cleanup in one concentrated round so
that the repository has one clear reference implementation for later Phase 2
work.

### 7.2 Intended Outcome

After Phase 1:

1. `internal/sbi` keeps its current semantic scope
2. `collector` remains on the main SBI server
3. the server skeleton is materially closer to `pcf`
4. the package presents one deliberate `api -> processor -> consumer` shape

### 7.3 Planned Changes

#### A. Align router and middleware setup to the chosen free5GC skeleton

Implementation direction:

1. replace the current raw `gin.New()` setup with the usual
   free5GC-style Gin logger construction path
2. add inbound metrics middleware in the same place the reference NF servers do
3. preserve the existing logger categories and current handler behavior unless
   a narrow transport reshaping requires an adjustment

Expected result:

- the main SBI edge no longer looks like a custom local server scaffold

#### B. Align HTTP server construction and start path

Implementation direction:

1. replace the manual `http.Server` plus `net.Listener` ownership path with the
   local free5GC-style `httpwrapper.NewHttp2Server(...)` setup
2. align the `Run()` / `startServer()` scheme switch with the common
   `ListenAndServe` / `ListenAndServeTLS` skeleton as the main serving path
3. keep one thin synchronous preflight bind check in `Run()` so
   `pkg/service.startOwnedServers()` still fails and rolls back auxiliary
   listeners as one startup unit when the SBI port is unavailable
4. either support the currently-configured TLS path in the normal skeleton or
   make any temporary limitation explicit at the same lifecycle boundary, not
   hidden in a transport helper

Expected result:

- the main server startup path now looks like a normal free5GC NF SBI server
- the current NWDAF three-listener startup rollback behavior remains intact

#### B.1 Why the preflight bind check is intentional

The local `pcf` / `udm` style starts serving asynchronously after `Run()`
returns, which is fine for a standalone main SBI server but is not a perfect
match for NWDAF's current three-listener owned-startup unit.

This plan therefore deliberately keeps a very small local adaptation:

1. preflight-bind the configured SBI address synchronously
2. fail early if the address is unavailable
3. close the temporary listener immediately
4. then launch the normal `ListenAndServe()`-based server goroutine

This is a conscious lifecycle-preservation choice, not an accidental failure to
copy the exemplar exactly.

### 7.3.1 Phase 1 Implementation Status

Phase 1 has now landed in `NWDAF/` as commit `e8e249a`.

What was implemented:

1. `internal/sbi` now uses free5GC-style Gin construction via
   `logger_util.NewGinWithLogrus(...)`
2. `metrics.InboundMetrics()` is now attached at the main SBI edge
3. the SBI HTTP server is now constructed with `httpwrapper.NewHttp2Server(...)`
4. the start path now uses the `ListenAndServe()`-style serving skeleton
5. a synchronous preflight bind check remains in place
6. `Run()` now additionally waits for a short readiness handshake before
   reporting startup success, so the local three-listener lifecycle contract
   remains stronger than a plain fire-and-forget goroutine launch

Verification completed for the landed Phase 1 implementation:

- `go test ./...`
- `make build`
- `make lint`

Implementation status for this plan's current active tranche:

1. Phase 1 external `SBI` shape alignment is complete
2. Phase 1.5 external `SBI` HTTPS uplift is complete in code
3. the only remaining roadmap item in this plan is the later-deferred Phase 2
   auxiliary-server shape alignment, which is intentionally outside the
   completed current active tranche

#### C. Keep the PCF-style route-assembly shape

Implementation direction:

1. keep route assembly centered in `server.go`, consistent with the chosen
   `pcf` reference direction
2. preserve `Route` plus `applyRoutes` helpers unless a touched refactor makes a
   closer helper form clearly preferable
3. keep `collector` grouped under the main server surface

Expected result:

- the package aligns to `pcf` by design rather than drifting toward a mixed
  `pcf` / `udm` hybrid without reason

#### D. Tighten `api -> processor -> consumer` expression

Implementation direction:

1. keep HTTP parse / response shaping in `api_*` files
2. keep orchestration in `internal/sbi/processor`
3. keep standards-facing outbound access in `internal/sbi/consumer`
4. avoid moving domain logic back upward into handlers or server construction

Expected result:

- `internal/sbi` becomes the repo's explicit reference shape for later edge
  work

### 7.4 Explicit Non-Goals For Phase 1

Phase 1 should not:

1. move `collector` out of `internal/sbi`
2. collapse the three-server topology
3. redesign Priority 11 full-NF concerns such as NRF registration or a
   separate metrics server
4. restructure `AnLF` or `MTLF` in the same code change

---

## 8. Phase 1.5 — External `SBI` HTTPS Uplift

### 8.1 Objective

Phase 1.5 should close the immediate HTTPS/TLS support gap on the main
free5GC-facing NWDAF edge without expanding that transport decision into the
auxiliary listeners.

### 8.2 Intended Outcome

After Phase 1.5:

1. the main `internal/sbi` server still keeps the Phase 1 `pcf`-style
   skeleton and lifecycle behavior
2. the same server can run in free5GC-style dual mode, with `http` or `https`
   selected by `configuration.sbi.scheme`
3. `collector` moves with the same main `SBI` transport mode because it is
   already part of the main outward NWDAF callback surface
4. config shape and path conventions now look like normal free5GC NF config
5. `internal/anlf` and `internal/mtlf` remain unchanged and are not pulled into
   HTTPS support by implication

### 8.3 Planned Changes

#### A. Align `SBI` config shape to free5GC NF TLS conventions

Implementation direction:

1. extend `configuration.sbi` with a nested `tls` block containing `pem` and
   `key`
2. add the matching config getters and default-path constants in the same
   spirit as the surveyed free5GC NFs
3. keep the YAML shape aligned with the local free5GC config exemplars rather
   than inventing a repo-local flag such as `enableHttps`

Expected result:

- NWDAF HTTPS config looks like a normal free5GC NF `sbi` block

#### B. Keep free5GC-style dual-mode selection by `sbi.scheme`

Implementation direction:

1. allow `configuration.sbi.scheme` to select `http` or `https`
2. require `configuration.sbi.tls.pem` and `configuration.sbi.tls.key` when
   the selected scheme is `https`
3. keep `http` as a valid mode so local behavior can stay compatible with
   existing setups that have not switched to TLS yet

Expected result:

- one binary and one server skeleton support both transport modes, with config
  deciding which one is active

#### C. Extend the existing `SBI` server lifecycle to support HTTPS

Implementation direction:

1. replace the current explicit `https` rejection branch with a normal
   `ListenAndServeTLS(...)` start path
2. preserve the already-landed Phase 1 preflight bind check and startup
   readiness handshake so the local three-listener startup contract does not
   regress
3. if touched during alignment, thread any TLS key-log path wiring through the
   usual `cmd -> service -> sbi` route rather than introducing a local shortcut
4. keep the concrete start-path shape anchored to the `pcf`-style `SBI`
   server skeleton rather than drifting toward a different primary NF server
   form without an explicit replan

Expected result:

- HTTPS support lands as a narrow transport uplift on top of the already-fixed
  Phase 1 lifecycle skeleton

#### D. Keep callback ownership direct, but validate callback-scheme consistency

Implementation direction:

1. keep callback URIs such as the current `smf.notifUris.*` as direct callback
   URLs rather than forcing an NRF-discovery rewrite in this round
2. validate that the relevant callback URL scheme matches `sbi.scheme` for the
   currently-owned NWDAF callback surface
3. leave broader peer-discovery redesign questions outside this round

Expected result:

- callback configuration stays semantically correct while avoiding easy
  transport-mismatch mistakes

#### E. Follow free5GC-style `config/` plus `cert/` layout, but keep cert assets in `nwdaf-resources/`

Implementation direction:

1. assume the same broad layout style used by local free5GC config examples:
   sibling `config/` and `cert/` directories
2. keep YAML values in free5GC-style relative form such as `cert/nwdaf.pem`
   and `cert/nwdaf.key`
3. place the initial dev/test certificate assets in `nwdaf-resources/`
4. use one explicit dev/test CA and one NWDAF leaf certificate so later
   trust-testing has a cleaner base than a one-off leaf-only self-signed cert

Expected result:

- resource layout and YAML shape feel familiar to free5GC-style operators while
  keeping test certificate assets outside the main implementation repository

### 8.4 Explicit Non-Goals For Phase 1.5

Phase 1.5 should not:

1. move `AnLF` or `MTLF` onto HTTPS
2. pull the deferred Phase 2 server-shape refactor into the same change line
3. redesign `smf.endpoints` around NRF discovery in this round
4. promise full multi-NF trust-chain integration in the current environment
5. introduce production-grade certificate lifecycle management or secret
   rotation behavior

### 8.5 Implementation Status

Phase 1.5 is now completed in `NWDAF/` across two code commits:

1. `8762b35`
   - enables free5GC-style dual `http` / `https` operation on the main
     external `SBI` listener
   - aligns config shape to `configuration.sbi.tls.pem` and
     `configuration.sbi.tls.key`
   - adds callback-scheme validation for the main owned `collector` surface
   - adds focused config and lifecycle coverage, including a local HTTPS
     listener proof
   - documents that the current startup readiness check guarantees
     listener-readiness and not a full TLS client handshake proof
2. `a0fff93`
   - closes the remaining callback-ownership gap by routing ADRF retrieval
     notifications through the main `SBI` `collector` surface
   - keeps Daisy training-completion callbacks on the auxiliary `MTLF`
     listener
   - separates `MTLF`-owned and `SBI`-owned callback URL builders explicitly

Verification completed for the landed Phase 1.5 implementation:

- `go test ./...`
- `make build`
- `make lint`

Phase 1.5 therefore satisfies the active-tranche completion target for this
plan. The only remaining roadmap content in this document is the still-deferred
Phase 2 auxiliary-server follow-up.

---

## 9. Phase 2 — `AnLF` And `MTLF` Shape Alignment
### 9.1 Objective

Phase 2 should reuse the now-stabilized Phase 1 `sbi` shape as an internal
reference, but only at the level of engineering form, not semantics.

For the currently selected Phase 2 scope, the goal is structural alignment:

1. make the auxiliary listeners look deliberate and consistent with the rest of
   the repository
2. give `AnLF` and `MTLF` explicit `server + api + processor + client`
   surfaces
3. keep callback ingress, callback delegation, and outbound non-SBI HTTP usage
   clearly separated

This Phase 2 scope does not require all remaining local domain logic to move
into `processor` immediately.

Deeper consolidation of business procedure ownership, or later simplification
when more logic moves to Python services, should be treated as future work
rather than as a hidden blocker on Phase 2 completion.

### 9.1.1 Current Implementation Status

The currently selected Phase 2 structural-alignment scope has now landed in
`NWDAF/` on 2026-07-06 as commit `1b06411`.

A follow-up stabilization pass then landed in `NWDAF/` on 2026-07-06 as commit
`e2f677c`.

What was implemented:

1. `AnLF` and `MTLF` now expose explicit auxiliary `api_*` callback handlers
2. both packages now carry dedicated `processor/` seams for callback
   delegation
3. inference-engine and Daisy outbound HTTP integrations now live under
   package-local `client/` directories
4. auxiliary server construction now follows the repository's aligned HTTP edge
   style more closely while preserving their non-SBI semantics
5. remaining deeper local workflow consolidation is left intentionally as
   future work
6. `AnLF` callback action execution is now batched across the full callback
   body instead of spawning one owned batch per top-level notification
7. `AnLF` now cleans up old shared-model and monitor state when a subscription
   switches to a new model URL, while still recording the replacement
   failure-path rollback gap as a known limitation for later work

This means the current-stage Phase 2 scope is complete in code even though the
longer-horizon Python-oriented simplification work remains open by design.

### 9.2 Intended Outcome

After Phase 2:

1. `AnLF` and `MTLF` still remain auxiliary non-SBI HTTP surfaces
2. both packages expose a consistent `server + api + processor + client` shape
3. inbound callback parsing no longer lives directly on service-owned workflow
   structs
4. outbound non-SBI integrations are named and owned consistently as `client`
5. package-local domain logic that still owns state, monitor loops, retrain
   bookkeeping, or hot-swap mechanics may remain in the root package when
   moving it immediately would not materially improve the current design

The intended Phase 2 package shape is therefore closer to:

```text
internal/anlf/
  server.go
  api_*.go
  anlf.go
  analytics.go
  model.go
  monitor.go
  scope.go
  inference_engine.go
  processor/
    processor.go
    ...
  client/
    inference_engine.go

internal/mtlf/
  server.go
  api_*.go
  mtlf.go
  training.go
  trigger.go
  adrf_retrieval.go
  state_store.go
  daisy.go
  processor/
    processor.go
    ...
  client/
    daisy.go
```

This layout is the current-stage target.

### 9.3 Planned Changes

#### A. Introduce explicit API handlers for auxiliary callback ingress

Implementation direction:

1. move HTTP request parse / validation / response logic out of direct
   service-method ownership
2. add explicit `api_*` files for the auxiliary callback endpoints
3. keep route registration in the package-owned server, not in `pkg/service`

Expected result:

- `AnLF` and `MTLF` no longer expose a service-owned callback shape at the HTTP
  edge

#### B. Introduce processor ownership for inbound callback workflows

Implementation direction:

1. introduce a dedicated processor layer in each package or an equivalent
   package-local processor type
2. make the processor the explicit callback delegation seam between the HTTP
   edge and the package-local workflow logic
3. move only the callback procedure control that materially improves current
   ownership clarity
4. keep deeper domain behavior where it already belongs unless a narrow move is
   required to remove HTTP concerns from that code
5. do not require all stateful local actions, monitor loops, retrain
   bookkeeping, or hot-swap logic to move into `processor` in the current
   stage

Expected result:

- callback-driven ingress now has the same visible ownership style as the main
  SBI edge, without pretending the flows are SBI
- `processor` becomes the stable entry seam for later refactoring, even if some
  current local workflow logic still lives in the root package during this
  stage

#### C. Normalize outbound naming as `client`

Implementation direction:

1. place inference-engine outbound integration under `internal/anlf/client`
2. place Daisy outbound integration under `internal/mtlf/client`
3. preserve `consumer` naming for standards-facing or SBI-facing outbound seams
4. keep ADRF access on the narrower standards-facing seam already used by the
   current code unless a later separate plan deliberately re-homes it

Expected result:

- the repo gains one stable naming rule:
  - `consumer` for standards-facing service consumption
  - `client` for non-SBI auxiliary integrations

#### D. Align lifecycle and logging style without sharing a base server

Implementation direction:

1. make `Run()` / `Shutdown()` / panic-guard / error-log style look more
   consistent across all three servers
2. do not force `AnLF` or `MTLF` to inherit SBI-only assumptions
3. do not introduce a generic transport superclass or base package

Expected result:

- the three servers look like one coherent repository family while still
  preserving different semantics

### 9.4 Explicit Non-Goals For Phase 2

Phase 2 should not:

1. rename auxiliary servers as SBI servers
2. apply SBI-specific OAuth or service-name assumptions to auxiliary flows
3. introduce one cross-package transport abstraction just because the code now
   looks similar
4. merge auxiliary clients back into `internal/sbi/consumer`
5. require a full migration of all remaining local business logic into
   `processor` before this phase can be considered complete
6. treat the future Python-side migration of analytics or training logic as a
   prerequisite for Phase 2 completion

### 9.5 Future Work Beyond Phase 2

The following items are intentionally outside the current Phase 2 completion
gate:

1. pushing more `AnLF` or `MTLF` business procedure logic out of the Go root
   packages and into thinner callback-oriented processors
2. reshaping Go-side packages further once analytics or retraining logic moves
   behind Python-owned services
3. revisiting whether the Go side should eventually become a more conventional
   `api -> processor -> client` shell after local stateful responsibilities
   shrink
4. revisiting the `AnLF` ML-model replacement failure path so old shared-model
   and monitor state are not detached until the new model is proven ready;
   this is currently recorded as a known limitation rather than an active fix
   target because the longer-horizon direction is to move more of this
   replacement workflow behind Python-owned services

These Python-oriented follow-up items are now tracked primarily under:

- `nwdaf-docs/docs/plans/anlf-backend-transition/AnLF Backend Transition Plan.md`

These are valid later continuations, but they are not evidence that the
current-stage Phase 2 target remains unfinished.

---

## 10. Verification Expectations

The implementation should be verified in two layers:

### 10.1 Phase 1 Verification

At minimum:

- `make build`
- `make lint`
- `go test ./...`

Targeted proof expected:

1. existing `internal/sbi/api_*_test.go` coverage still passes
2. `collector` routes remain on the main server surface
3. server startup / shutdown still behaves correctly after the transport-shape
   change

### 10.2 Phase 1.5 Verification

At minimum:

- `make build`
- `make lint`
- `go test ./...`

Targeted proof expected:

1. config validation accepts `http` and `https` in the intended cases
2. `https` mode requires `sbi.tls.pem` and `sbi.tls.key`
3. the owned NWDAF callback URL scheme remains consistent with
   `configuration.sbi.scheme`
4. the main `SBI` listener can still be started and shut down under automated
   test control after the HTTPS branch is introduced
5. if a local package-level TLS listener proof is practical in the touched test
   seam, it should be preferred; if not, the missing proof should be stated
   explicitly rather than hidden
6. a full multi-NF runtime smoke test is not required in the current
   environment and should be tracked separately if later needed

### 10.3 Phase 2 Verification

At minimum:

- `make build`
- `make lint`
- `go test ./...`

Targeted proof expected:

1. auxiliary callback handlers still accept and reject the same payload classes
2. callback-to-processor delegation is directly testable
3. outbound `client` seams are mockable at the processor boundary
4. listener lifecycle and partial-startup cleanup remain covered
5. Phase 2 verification should not require proving that all remaining local
   domain logic has already been migrated into `processor`

---

## 11. Risks And Mitigations

### Risk 1. The work drifts into a hidden semantic rewrite

Danger:

- `AnLF` / `MTLF` could be reshaped in a way that accidentally makes them look
  like fake SBI services

Mitigation:

1. keep `consumer` reserved for standards-facing service consumption
2. use `client` for auxiliary outbound integrations
3. treat the three-server semantic split as fixed

### Risk 2. Over-abstraction creates a repo-wide base transport layer

Danger:

- shape alignment can tempt implementation toward a shared generic HTTP server
  base that encodes the wrong semantics

Mitigation:

1. align by repeated structure, not by one shared parent abstraction
2. only factor helpers that are transport-trivial and semantics-neutral

### Risk 3. Phase 1.5 gets mixed with broader Priority 11 or Phase 2 scope

Danger:

- adding HTTPS support may tempt the work into also reopening NRF registration,
  full metrics-server ownership, or the deferred auxiliary-server reshaping

Mitigation:

1. keep this plan explicitly bounded to the main external `SBI` edge
2. leave full Priority 11 questions and deferred Phase 2 work out unless a
   hard implementation blocker proves they are inseparable

### Risk 4. Test-environment limits are mistaken for design failure

Danger:

- the current workspace may not be ideal for a full runtime HTTPS smoke flow,
  and that can create pressure either to over-promise or to under-document the
  actual proof achieved

Mitigation:

1. prefer automated config and local listener proof inside `go test`
2. state clearly what could not be proven end-to-end in the current
   environment
3. keep any later multi-NF runtime smoke as a separate verification follow-up,
   not as an unspoken hidden requirement

---

## 12. Recommended Execution Order

### Step 1. Record Phase 1 as the stable `SBI` shape baseline

Reason:

- the repository already has one stable `SBI` shape baseline, and later work
  should build on that landed result rather than reopening it casually

### Step 2. Implement Phase 1.5 inside the existing `internal/sbi` ownership line

Reason:

- HTTPS is the current active follow-up and should stay narrowly attached to the
  already-aligned `SBI` transport/lifecycle skeleton

Status:

- completed in `NWDAF/` on 2026-07-06 across commits `8762b35` and `a0fff93`

### Step 3. Re-review the resulting `SBI` transport shape against free5GC server and config exemplars

Reason:

- confirm that the uplift still matches the intended `pcf` / `amf` / `udm` /
  `smf` style instead of drifting into a repo-local HTTPS convention

### Step 4. Leave Phase 2 documented but deferred until it is explicitly selected

Reason:

- the current user direction does not want to mix auxiliary-server reshaping
  into the active HTTPS tranche

### Step 5. If Phase 2 is later reopened, implement `internal/anlf` before `internal/mtlf`

Reason:

- `AnLF` is still the smaller auxiliary surface and remains the cleaner place
  to prove a future auxiliary-server pattern first

### Step 6. Treat deeper processor consolidation as later optional continuation

Reason:

- future Python-side ownership changes may alter how much local Go business
  logic is worth retaining or relocating
- the current Phase 2 target should stop once the auxiliary package shape,
  callback delegation seam, and `client` ownership are in place

---

## 13. Completion Criteria

For the current active tranche, this plan should be considered complete only
when all of the following are true:

1. `internal/sbi` is materially closer to the intended `pcf`-style server
   skeleton
2. `collector` still clearly belongs to the main SBI surface
3. the main external `SBI` edge can run in free5GC-style dual `http` /
   `https` mode
4. `SBI` config uses the intended free5GC-style `tls.pem` / `tls.key` shape
5. the current NWDAF-owned callback URLs do not silently drift to a scheme that
   contradicts `sbi.scheme`
6. the repo uses one stable naming rule:
   - `consumer` for SBI / standards-facing outbound consumption
   - `client` for auxiliary non-SBI outbound integrations
7. no shared base HTTP server abstraction was introduced
8. `AnLF` and `MTLF` were not accidentally pulled into the HTTPS uplift by
   implication
9. verification reruns `make build`, `make lint`, and `go test ./...`

Current assessment:

- all active-tranche completion criteria above are now satisfied in code
- the current-stage Phase 2 auxiliary structural-alignment target is also now
  satisfied in code by `NWDAF/` commit `1b06411`
- the remaining future work is deeper processor/business-logic consolidation,
  which stays intentionally outside the current completion gate
- Python-side `AnLF` backend migration is now tracked separately under
  `nwdaf-docs/docs/plans/anlf-backend-transition/AnLF Backend Transition Plan.md`

Deferred future continuation:

- If Phase 2 is later selected, its own auxiliary-server shape criteria should
  be tracked as a separate completion gate at that time rather than being
  treated as a hidden requirement of the current HTTPS tranche.
- If a later Python-oriented refactor wants thinner Go-side processors and
  clients, that follow-up should be tracked as new future work rather than as a
  retroactive claim that the current-stage Phase 2 target was incomplete.
