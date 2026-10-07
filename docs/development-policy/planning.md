# Planning And Implementation Slices

Use this module when defining a substantial implementation slice or extending
an existing production flow. For ownership or contract decisions, also use
[Architecture And Contracts](architecture.md). For writing the plan itself,
use [Documentation](documentation.md).

## Slice Definition

Before substantial implementation, identify:

1. the behavior or vertical flow being changed;
2. the repositories and behavior owners involved;
3. the external and private contracts involved;
4. the acceptance tests and required verification;
5. explicitly deferred behavior.

Make the smallest complete change within this approved slice. Preserve existing
behavior unless the plan explicitly replaces or removes it. Keep feature work
separate from unrelated refactoring. Complete means closing this slice's
behavior and acceptance criteria, not automatically implementing adjacent
phases, speculative resilience, unused legacy cleanup, unconfirmed integration
risks, or a broader architecture refactor.

A project phase may contain several slices. Keep their scope and evidence
individually reviewable while progressing through the work the user authorized.

For a substantial slice, make acceptance traceable using
[Review And Conformance](review.md#plan-conformance). Explicit commitments in
prose count alongside checklists and acceptance tables.

## Existing-Flow Extension

When a new mode, topology, role, version, or procedure extends a production
flow, name its canonical baseline before the plan is implementation-ready.
Trace trigger, preparation, execution, validation, publication or handoff,
success, failure, timeout, restart, and cleanup where applicable.

For every baseline stage, record one disposition:

- reused without semantic change;
- adapted, identifying the changed owner, data flow, or invariant;
- explicitly replaced;
- approved for deferral;
- not applicable, with evidence and rationale.

A stage cannot be silently omitted. A deferral is invalid when the omitted
stage is needed for a lifecycle claim, completion criterion, or other meaning
asserted by this slice.

Reusing an established semantic name, value, or representation preserves its
preconditions, postconditions, transition invariants, and externally or
internally relied-upon interpretation, wherever that meaning is encoded.
Examples include states, events, artifact roles, result labels, operation
names, status values, schema discriminators, and lifecycle milestones.

If the meaning changes, use a distinct representation or obtain an explicit
semantic contract decision. Acceptance criteria and tests must cover both the
intended differences and the shared semantics retained from the baseline.

## Out-of-scope Work

Classify work outside the active slice using:

| Classification | Meaning |
| --- | --- |
| `future-phase handoff` | Owned by a named later phase |
| `legacy cleanup` | Obsolete or unused code without a current functional failure |
| `optional hardening` | Resilience beyond the required current contract |
| `integration verification gap` | Behavior not yet exercised in the required external environment |
| `unconfirmed risk` | Plausible concern without deterministic evidence |

Record adjacent issues rather than adding them to the slice, unless they
directly prevent its acceptance criteria from being satisfied. Discovery alone
does not make them blockers. Admitted current-slice defects and decisions about
broader remediation follow [Review](review.md#finding-admission) and the
[shared decision gates](../development_policy.md#decision-gates).
