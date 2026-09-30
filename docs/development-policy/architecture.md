# Architecture And Contracts

Use this module for ownership, new Go packages, contracts, or designs crossing
component, process, repository, protocol, storage, or trust boundaries. It also
applies to feasibility advice without file edits. Substantial slice planning
uses [Planning](planning.md); actual code changes use
[Implementation](implementation.md). Apply the free5GC skill only to its
relevant technical boundary, as routed by the workspace guide.

## Ownership And Package Placement

- Follow established repository and package boundaries. In `NWDAF/`, preserve
  the applicable `context`, `consumer`, `processor`, `factory`, `service`, and
  config-driven structure.
- Keep shared Go runtime state in `internal/context/` unless the approved
  ownership design specifies otherwise.
- Prefer extending the existing service flow to a parallel command path.
- Keep Python business state in the backend owning the behavior; Go NF
  package structure does not govern Python internals.
- For a new internal contract or state machine, document its owner,
  lifecycle, failure boundary, and restart behavior.

### New Go Package Gate

A new Go package or directory is an architecture decision. Before adding one:

1. Identify the behavior owner, callers, transport boundary, state/lifecycle
   owner, and dependency direction.
2. Inspect the target repository for an existing owner package.
3. Inspect the applicable free5GC exemplar and identify the package shape and
   boundary it demonstrates.
4. Prefer a file in the existing owner. A shared parser, validator, schema or
   protocol name, or avoiding duplication alone does not justify a package.
5. Create a package only for a distinct, stable responsibility that cannot fit
   an existing owner without reversing dependencies, introducing an import
   cycle, or mixing unrelated lifecycles.
6. Classify by owner and transport, not payload shape. Standard field names or
   `ProblemDetails` do not turn a private Go-backend API into external SBI or
   justify placing it under `internal/sbi`.
7. With no clear repository precedent or applicable exemplar, record the
   proposed ownership in the active plan and discuss it before implementation.

Review must enumerate new packages and check necessity, dependencies, and
exemplars. Passing tests establish behavior, not correct package placement.

## End-to-End Data Flow

Before calling a cross-boundary design simple, feasible, or implementation-ready,
trace the complete directional flow from authoritative producer through
transport and stored state to final consumer. Establish:

1. the behavior or acceptance criterion requiring the change;
2. the authoritative producer of every required value;
3. its transport through each boundary;
4. its runtime owner and stored representation at each stage;
5. whether each input exists exactly where it is consumed, through an inbound
   contract, local state, configuration, or authenticated request context;
6. whether an expected validation value is independently obtainable, rather
   than derived from the same untrusted input being checked;
7. lifecycle, failure, restart, timeout, and stale-data behavior as applicable;
8. all affected repositories and external, standard-shaped, or private
   contracts;
9. focused boundary tests and required end-to-end verification.

Check request, response, callback, retry, and recovery directions separately;
their information and ownership can differ. Qualify instances and directions
when a component type has multiple roles, such as the PyMTLF owned by the Root
NWDAF or a Branch NWDAF receiving from the Root NWDAF.

A receiver-local field, schema, validator, configuration entry, or helper is
not a complete solution if upstream production and propagation are unresolved.
Missing data at the consuming boundary is an architecture or contract change,
not an implementation-local assumption.

Separate locally implementable code, end-to-end implementable behavior,
architecture requiring a decision, optional hardening, and unconfirmed risk.
State unknown sources or boundaries before recommending implementation or
estimating complexity; estimate the complete flow, not only the final code.

## Standard Boundary Levels

### External SBI

Follow the applicable OpenAPI operation's method, path, request parameters and
body, success status and representation, required headers such as `Location`,
declared error statuses, and applicable `ProblemDetails` behavior.

Use generated models where available. When generation lacks an external schema,
record the exact dependency gap before introducing an isolated compatibility
type.

### Standard-shaped Private Boundary

A private Go-to-backend route preserves only the standard properties explicitly
listed in the active plan, normally method semantics, request/response models,
identifiers and correlations, success status/headers/representation, and
operation-specific peer-error forwarding.

TLS, OAuth, NRF registration, other shared 3GPP transport features, and extra
recovery are included only when explicitly planned. Private paths can differ
from public standard paths; plain HTTP suffices when it is the confirmed
internal-security scope.

### Python Backends

PyAnLF and PyMTLF are not independent standardized NFs. Local 3GPP/OpenAPI
defines Go-facing payload semantics; normal Python service structure governs
internal runtime, repositories, queues, scheduling, training, and model logic.

## Experimental Schema Evolution

Project-defined schemas used only by the current experiment evolve in place.
Update producer, consumer, stored representation, fixtures, and tests together,
then remove superseded shapes. Regenerate disposable experimental artifacts and
local state rather than maintaining speculative migrations.

Ignored legacy fields, aliases, fallback readers, migration branches, and
parallel models need a confirmed current consumer, non-disposable state, or an
explicit user decision recorded in the active plan. The same applies to
project-defined `v1`/`v2` names, versioned paths, `schemaVersion`, and equivalent
markers used solely to distinguish obsolete experimental shapes.

Original 3GPP schemas are exempt: preserve applicable standard information
elements, field semantics, protocol behavior, compatibility rules, and release
distinctions. The surrounding experiment is not a reason to remove or rename
a standard field or version concept.
