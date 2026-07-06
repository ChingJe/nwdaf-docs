# NWDAF Main Project HTTP Edge Shape Alignment Item

Date: 2026-07-06

Status: Open

Historical remediation item:

- `Priority 14 — Align HTTP Edge Shape And Flow Ownership`

## Scope

This item records a follow-up alignment gap discovered after the completed
Priority 12 server-topology split.

The scope is intentionally limited to the current three-server runtime shape in
`NWDAF/`:

- the main `internal/sbi` server
- the `internal/anlf` auxiliary server
- the `internal/mtlf` auxiliary server

It is about HTTP edge shape, handler / processor / outbound-call ownership, and
free5GC-style implementation consistency.

It is not about reopening the already-completed decision to split the runtime
into three HTTP listeners.

## Problem Statement

Priority 12 closed the transport-meaning problem correctly:

1. `internal/sbi` now owns only the main NWDAF SBI ingress
2. `AnLF` and `MTLF` now own their own non-SBI auxiliary HTTP ingress
3. callback URIs for the touched non-SBI flows now derive from owned auxiliary
   server config

However, the resulting code still carries a second-order alignment gap:

1. the main `internal/sbi/server.go` shape is still not closely aligned with
   the common free5GC NF SBI server skeleton represented most clearly by local
   `pcf` reference code
2. `internal/anlf/server.go` and `internal/mtlf/server.go` currently duplicate
   a narrow manual server scaffold that still looks closer to the older
   pre-alignment NWDAF shape than to a deliberate post-Priority-12 design
3. the three inbound HTTP surfaces now have the correct semantic split, but do
   not yet present a consistent engineering shape for route assembly, handler
   ownership, processor ownership, and outbound client naming

The current runtime therefore has the right boundaries, but not yet the right
implementation shape for those boundaries.

## Confirmed Current Gaps

### 1. `internal/sbi` is semantically right but technically not yet in the intended PCF-style shape

Current `NWDAF/internal/sbi/server.go` still differs from the chosen local
free5GC exemplar direction in several concrete ways:

1. it uses `gin.New()` plus manual Gin logger / recovery setup instead of the
   more typical `logger_util.NewGinWithLogrus(...)` plus
   `metrics.InboundMetrics()` shape used by exemplar NFs such as `pcf`, `udm`,
   and `amf`
2. it assembles `http.Server` manually and explicitly manages `net.Listener`
   ownership instead of using the usual `httpwrapper.NewHttp2Server(...)`
   helper path
3. `https` support is still a placeholder branch rather than the same
   `ListenAndServe` / `ListenAndServeTLS` skeleton used by the reference NF
   server implementations

This is not a functional correctness bug today, but it keeps the NWDAF SBI
edge looking repository-local rather than deliberately free5GC-aligned.

### 2. `AnLF` and `MTLF` still mix HTTP edge handling with service-owned callback logic

Current `internal/anlf` and `internal/mtlf` each have their own server now, but
their runtime shape is still thin and inconsistent relative to the desired
follow-up direction:

1. `server.go` registers routes directly against service methods
2. the inbound callback handlers are still owned by service structs
3. outbound integrations are important in these packages, but their call shape
   is not yet normalized to the same inbound/outbound layering story that
   already exists in `internal/sbi`

This means the auxiliary server split is functionally correct, but the current
shape does not yet give the repo one clear pattern such as:

- `api -> processor -> consumer` for SBI-facing flows
- `api -> processor -> client` for non-SBI auxiliary flows

### 3. Outbound naming and ownership are now semantically split, but not yet expressed in one stable pattern

The current code already hints at the desired distinction:

1. `internal/sbi` owns `consumer` for standards-facing or NF-facing outbound
   service consumption
2. `internal/anlf` uses an inference-engine client
3. `internal/mtlf` uses a Daisy client and also depends on ADRF access

But the naming and call ownership are not yet documented and enforced as one
deliberate repo rule. Without that rule, later implementation can easily drift
back into mixed handler/service/client responsibilities.

## Why This Is A Real Project Item

This gap matters even though the current code is functional:

1. future `NWDAF/` work will continue touching HTTP edges, callbacks, and
   domain integrations
2. without one deliberate post-Priority-12 implementation shape, future code
   review will keep re-opening the same questions about `server`, `api`,
   `processor`, `consumer`, and `client`
3. the repository currently has the correct boundary semantics but still lacks
   one stable engineering story for how those boundaries should be expressed in
   code

This makes the issue architectural and maintenance-relevant, not merely
cosmetic.

## Fixed Decisions For The Follow-Up Plan

The follow-up work for this item should treat the following decisions as fixed:

1. keep three owned HTTP listeners
2. keep `collector` on the main `internal/sbi` server because it is part of the
   outward NWDAF callback / collection surface exposed to peer NFs
3. use `pcf` as the primary free5GC shape reference for the `internal/sbi`
   follow-up
4. keep `consumer` naming for `internal/sbi` outbound standards-facing calls
5. use `client` naming for `internal/anlf` and `internal/mtlf` outbound
   non-SBI integrations
6. align shapes and ownership patterns, but do not introduce a shared
   cross-package base HTTP server abstraction

### Phase 1 Lifecycle Decision Record

During pre-implementation review for Phase 1, one concrete `internal/sbi`
startup-lifecycle question required an explicit choice.

The current NWDAF `pkg/service.startOwnedServers()` path treats the three owned
listeners as one startup unit:

1. `AnLF` starts
2. `MTLF` starts
3. `SBI` starts
4. if `SBI` bind fails synchronously, the already-started auxiliary listeners
   are immediately shut down and the app startup fails as one unit

A direct copy of the common free5GC `pcf` / `udm` `Run() -> go
ListenAndServe()` skeleton would weaken that current behavior because the SBI
bind failure would move into an asynchronous goroutine after
`startOwnedServers()` already returned success.

The chosen resolution for Phase 1 is therefore:

1. keep the `pcf`-style server skeleton direction
2. move `internal/sbi` onto `httpwrapper.NewHttp2Server(...)`
3. use the `ListenAndServe()`-style start path in the owned server
4. but preserve a thin synchronous preflight bind check before launching the
   serving goroutine

This choice was selected because it preserves the current startup-unit cleanup
semantics while still aligning the visible `sbi.Server` transport skeleton much
more closely with the chosen free5GC exemplar.

## Planned Resolution Shape

The planned resolution is intentionally phased:

1. Phase 1
   - aggressive internal cleanup inside `internal/sbi` only
   - target shape follows the local `pcf` SBI server style
   - preserve the current three-server topology and the current semantic
     boundary decisions from Priority 12
2. Phase 2
   - align `internal/anlf` and `internal/mtlf` to a consistent
     `server + api + processor + client` shape
   - keep them explicitly non-SBI in naming and role

The detailed implementation plan for this item lives in:

- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 14 HTTP Edge Shape And Flow Alignment Plan.md`
