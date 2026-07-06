# NWDAF Priority 14 HTTP Edge Shape And Flow Alignment Plan

Date: 2026-07-06

Status: Not started

Historical remediation item:

- `Priority 14 — Align HTTP Edge Shape And Flow Ownership`

Current execution note:

- this plan starts from the already-completed Priority 12 topology split
- the three-server semantic boundary is treated as fixed input, not as a design
  question reopened by this plan
- the next work is about implementation shape, not about collapsing the
  topology back into one shared server

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
2. `internal/anlf` and `internal/mtlf` should keep their non-SBI semantics, but
   evolve toward one consistent auxiliary-server implementation shape
3. inbound HTTP ownership and outbound HTTP ownership should be expressed with
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
6. Phase 2 should move `internal/anlf` and `internal/mtlf` toward
   `api -> processor -> client`
7. `consumer` naming remains reserved for SBI-facing or standards-facing
   service consumption
8. `client` naming should be used for non-SBI auxiliary integrations such as
   the inference engine and Daisy
9. the goal is to unify shape and ownership style, not to unify semantics
10. no cross-package shared base HTTP server should be introduced in this round

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

### 5.2 Remaining Shape Gaps In `internal/sbi`

Current `NWDAF/internal/sbi/server.go` still differs from the chosen reference
shape in several technical ways:

1. it uses `gin.New()` with manual logger / recovery setup instead of the
   typical free5GC Gin logger wrapper path
2. it does not currently add `metrics.InboundMetrics()`
3. it manually constructs `http.Server`
4. it manually owns `net.Listener` startup instead of using the more common
   `httpwrapper.NewHttp2Server(...)` path
5. its `https` branch remains intentionally incomplete

None of these invalidate current behavior, but together they mean the main SBI
edge still looks like an intermediate repo-local form rather than a stable
aligned form.

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

At the end of this plan, the intended repository shape is:

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

## 8. Phase 2 — `AnLF` And `MTLF` Shape Alignment

### 8.1 Objective

Phase 2 should reuse the now-stabilized Phase 1 `sbi` shape as an internal
reference, but only at the level of engineering form, not semantics.

### 8.2 Intended Outcome

After Phase 2:

1. `AnLF` and `MTLF` still remain auxiliary non-SBI HTTP surfaces
2. both packages expose a consistent `server + api + processor + client` shape
3. inbound callback parsing no longer lives directly on service-owned workflow
   structs
4. outbound non-SBI integrations are named and owned consistently as `client`

### 8.3 Planned Changes

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
2. make the processor the owner of callback-driven orchestration
3. keep deeper domain behavior where it already belongs unless a narrow move is
   required to remove HTTP concerns from that code

Expected result:

- callback-driven orchestration now has the same ownership style as the main
  SBI edge, without pretending the flows are SBI

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

### 8.4 Explicit Non-Goals For Phase 2

Phase 2 should not:

1. rename auxiliary servers as SBI servers
2. apply SBI-specific OAuth or service-name assumptions to auxiliary flows
3. introduce one cross-package transport abstraction just because the code now
   looks similar
4. merge auxiliary clients back into `internal/sbi/consumer`

---

## 9. Verification Expectations

The implementation should be verified in two layers:

### 9.1 Phase 1 Verification

At minimum:

- `make build`
- `make lint`
- `go test ./...`

Targeted proof expected:

1. existing `internal/sbi/api_*_test.go` coverage still passes
2. `collector` routes remain on the main server surface
3. server startup / shutdown still behaves correctly after the transport-shape
   change

### 9.2 Phase 2 Verification

At minimum:

- `make build`
- `make lint`
- `go test ./...`

Targeted proof expected:

1. auxiliary callback handlers still accept and reject the same payload classes
2. callback-to-processor delegation is directly testable
3. outbound `client` seams are mockable at the processor boundary
4. listener lifecycle and partial-startup cleanup remain covered

---

## 10. Risks And Mitigations

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

### Risk 3. Phase 1 gets mixed with broader Priority 11 scope

Danger:

- adding `httpwrapper`, logger wrappers, or metrics middleware may tempt the
  work into also reopening NRF registration or full metrics-server ownership

Mitigation:

1. keep this plan explicitly bounded to HTTP edge shape
2. leave full Priority 11 questions out unless a hard implementation blocker
   proves they are inseparable

---

## 11. Recommended Execution Order

### Step 1. Implement Phase 1 fully inside `internal/sbi`

Reason:

- the repository needs one stable reference shape before aligning the auxiliary
  servers

### Step 2. Re-review the resulting `sbi` shape against `pcf`

Reason:

- confirm that the implementation actually landed on the intended reference
  shape rather than on an accidental hybrid

### Step 3. Implement Phase 2 in `internal/anlf`

Reason:

- `AnLF` is the smaller auxiliary callback surface and should set the first
  auxiliary pattern

### Step 4. Implement Phase 2 in `internal/mtlf`

Reason:

- `MTLF` has the richer outbound / callback / follow-up orchestration story and
  should follow after the auxiliary pattern is proven once

### Step 5. Re-check lifecycle and naming consistency across all three servers

Reason:

- the end state should read as one coherent repository style without erasing
  semantic differences

---

## 12. Completion Criteria

This plan should be considered complete only when all of the following are
true:

1. `internal/sbi` is materially closer to the intended `pcf`-style server
   skeleton
2. `collector` still clearly belongs to the main SBI surface
3. `AnLF` and `MTLF` no longer expose service-owned HTTP callback handlers as
   the main edge shape
4. `AnLF` and `MTLF` each present a stable `api -> processor -> client` flow
5. the repo uses one stable naming rule:
   - `consumer` for SBI / standards-facing outbound consumption
   - `client` for auxiliary non-SBI outbound integrations
6. no shared base HTTP server abstraction was introduced
7. verification reruns `make build`, `make lint`, and `go test ./...`
