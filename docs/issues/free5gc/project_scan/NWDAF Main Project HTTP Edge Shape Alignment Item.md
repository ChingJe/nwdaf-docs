# NWDAF Main Project HTTP Edge Shape Alignment Item

Date: 2026-07-06

Status: Open

Historical remediation item:

- `Priority 14 — Align HTTP Edge Shape And Flow Ownership`

Current progress:

- Phase 1 completed in `NWDAF/` on 2026-07-06 as commit `e8e249a`
- Phase 1.5 code implementation is now completed in `NWDAF/` on 2026-07-06
  across commits `8762b35` and `a0fff93`
- the currently selected Phase 2 auxiliary structural-alignment scope then
  landed in `NWDAF/` on 2026-07-06 as commit `1b06411`
- any later deeper processor/business-logic consolidation remains future work
  rather than an indication that the current-stage Phase 2 target is still
  open

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

### 1. `internal/sbi` is now mostly aligned in shape, but still carries the transport-completion gap

Phase 1 closed most of the earlier `internal/sbi` shape divergence:

1. the router now uses the free5GC-style Gin logger helper
2. inbound metrics middleware is now attached
3. the server is now built through `httpwrapper.NewHttp2Server(...)`
4. the owned-startup lifecycle contract is now preserved with preflight bind
   plus readiness confirmation

The still-open `internal/sbi` gap is narrower now:

1. `https` support is still a placeholder branch rather than the same
   `ListenAndServe` / `ListenAndServeTLS` skeleton used by the reference NF
   server implementations
2. the `SBI` config shape and certificate-path conventions are not yet aligned
   to the common free5GC `sbi.tls.pem/key` pattern

This is no longer a broad server-shape issue. It is now a transport-completion
issue on an otherwise much more aligned main `SBI` edge.

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

### Phase 1 Implementation Status

Phase 1 has now landed in `NWDAF/` as commit `e8e249a`.

The implemented result keeps the intended `pcf`-style server skeleton while
refining the final startup contract beyond the earlier design-decision note:

1. router construction now uses the free5GC-style Gin logger helper
2. inbound SBI metrics middleware is attached at the server boundary
3. HTTP server construction now uses `httpwrapper.NewHttp2Server(...)`
4. startup still performs a synchronous preflight bind check
5. `Run()` now also waits for a short startup readiness handshake before
   reporting success, so `startOwnedServers()` once again means the main SBI
   listener is actually accepting connections rather than merely having its
   serving goroutine dispatched

This means the still-open part of the issue is no longer Phase 1 `internal/sbi`
cleanup. The next active work is the narrower external `SBI` HTTPS uplift in
Phase 1.5, while the earlier-planned auxiliary-server alignment is now parked
as a later follow-up rather than immediate execution.

### Phase 1.5 Decision Record

After Phase 1 review and later free5GC-alignment comparison, the next active
follow-up is no longer Phase 2. It is a narrower HTTPS/TLS uplift on the main
external NWDAF edge only.

The selected decisions are:

1. Phase 1.5 applies only to the main `internal/sbi` server and the
   `collector` routes that already live on that server
2. `internal/anlf` and `internal/mtlf` are explicitly outside the Phase 1.5
   HTTPS scope and are not required to adopt HTTPS in this round
3. the runtime should keep free5GC-style dual-mode operation, with `http` or
   `https` selected by `configuration.sbi.scheme`
4. `SBI` config shape should align to free5GC NF config style by using
   `configuration.sbi.tls.pem` and `configuration.sbi.tls.key`
5. cert and config layout should follow the same broad free5GC convention of
   sibling `config/` and `cert/` directories, with YAML using `cert/...`
   relative paths rather than introducing a custom HTTPS-specific path scheme
6. the initial certificate strategy should use explicit dev/test assets kept in
   `nwdaf-resources/`, with one test CA and one NWDAF leaf certificate
7. callback URIs remain direct callback URLs rather than NRF-discovered
   service endpoints, but the relevant callback URL scheme should be kept
   consistent with `sbi.scheme`
8. verification for this round should focus on config validation, lifecycle
   behavior, and local HTTPS listener proof that can run under `go test`; a
   full multi-NF runtime smoke test is useful later but is not required in the
   current implementation environment
9. the primary Phase 1.5 exemplar for `SBI` server transport uplift should
   remain `pcf`; other NFs such as `amf`, `udm`, and `smf` are supporting
   references for config-shape and sibling-consistency checks rather than the
   main server-skeleton driver

### Phase 1.5 Implementation Status

Phase 1.5 has now landed in `NWDAF/` across two code commits:

1. `8762b35`
   - adds free5GC-style dual-mode `http` / `https` support on the main
     external `SBI` listener
   - aligns config shape to `configuration.sbi.tls.pem` and
     `configuration.sbi.tls.key`
   - validates owned callback-scheme consistency for the main `collector`
     surface
   - adds local HTTPS lifecycle/config proof and records the current
     listener-readiness semantics explicitly
2. `a0fff93`
   - corrects the remaining callback-ownership seam so ADRF retrieval
     notifications are routed through the main `SBI` `collector` surface
     rather than through the auxiliary `MTLF` listener
   - keeps Daisy training-completion callbacks on the owned `MTLF` listener
     while separating the two callback URL builders explicitly

The current active tranche for this item is therefore complete in code:

1. the main external `SBI` edge now supports free5GC-style dual `http` /
   `https` operation
2. `collector` remains on the main `SBI` edge and now also owns the ADRF
   retrieval callback path consistently
3. verification for the Phase 1.5 code line passed through `go test ./...`,
   `make build`, and `make lint`

### Phase 2 Current-Stage Implementation Status

The currently selected structural-alignment scope for Phase 2 has now landed in
`NWDAF/` as commit `1b06411`.

What this completed:

1. `internal/anlf` and `internal/mtlf` now expose explicit `server + api +
   processor + client` package surfaces
2. auxiliary callback parse / validation logic is now separated from
   service-owned methods
3. non-SBI outbound HTTP integrations for inference-engine and Daisy now live
   under package-local `client/` directories
4. auxiliary server construction now follows a more aligned repository HTTP
   edge style while preserving non-SBI semantics

What this intentionally did not require:

1. moving all remaining local domain logic into `processor`
2. treating later Python-oriented simplification as a prerequisite for closing
   the current-stage Phase 2 target

The remaining follow-up is therefore future work, not a sign that the current
stage of this item failed to complete.

The remaining open part of this overall issue is no longer Phase 1.5. It is
only the later-deferred Phase 2 auxiliary-server shape-alignment roadmap.

## Planned Resolution Shape

The planned resolution is intentionally phased:

1. Phase 1
   - aggressive internal cleanup inside `internal/sbi` only
   - target shape follows the local `pcf` SBI server style
   - preserve the current three-server topology and the current semantic
     boundary decisions from Priority 12
2. Phase 1.5
   - add free5GC-style dual-mode `http` / `https` support to the main
     `internal/sbi` edge only
   - keep `collector` under the main `SBI` server and move it with that
     transport decision
   - align config and certificate layout to the free5GC `config/` plus
     `cert/` convention without expanding HTTPS scope to auxiliary listeners
3. Phase 2
   - align `internal/anlf` and `internal/mtlf` to a consistent
     `server + api + processor + client` shape
   - keep them explicitly non-SBI in naming and role

Current execution intent:

1. Phase 1 is complete
2. Phase 1.5 is complete in code
3. Phase 2 remains documented for continuity, but is currently deferred rather
   than being part of the immediate implementation queue

The detailed implementation plan for this item lives in:

- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 14 HTTP Edge Shape And Flow Alignment Plan.md`
