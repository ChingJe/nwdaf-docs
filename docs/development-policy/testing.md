# Testing

Use this module to select, write, and assess tests across the workspace's
languages and repositories. Repository commands and required completion checks
belong to [Implementation](implementation.md#repository-native-verification);
independent test-code review belongs to [Review](review.md#test-code-review).

## Establish The Target

Derive expected behavior from the requirement, applicable contract, or confirmed
defect. Existing supported behavior can establish a regression baseline; a new
implementation alone cannot establish its own expected results.

Ask which behavior the test protects and how it would fail if that behavior were
wrong. These are design questions, not a requirement for per-test forms, written
justifications, or mutation testing. Test count and coverage percentages alone
do not establish that the requested behavior works.

## Decide Whether To Add A Test

Inspect relevant existing coverage first. Reuse or extend it when it already
exercises the required behavior; add tests for concrete gaps. A change does not
automatically require a new test file or a new test. A regression test is useful
when it reproduces the confirmed defect and protects its correction.

Choose cases from changed supported behavior and its meaningful failure paths.
Avoid enumerating unrelated input combinations or speculative features. Plain
prose changes use [document verification](documentation.md#verification).

## Design Effective Evidence

Choose the smallest test boundary that can expose the target failure. Exercise
the production entry point, state, and outcome relevant to the claim. A local
unit test can prove a local rule; a workflow claim needs evidence for the
connections and effects that make that workflow work.

Assert required outputs, state changes, or effects. Existence, type, or call-only
assertions suffice only when that property is itself the requirement. Avoid
assertions tied to incidental wording, internal names, or implementation order.

Mocks may isolate dependencies outside the behavior being tested. Do not mock
away the behavior claimed as verified, seed a fixture with the result that the
production path must create, or calculate expectations by copying the logic
under test. Shared helpers provide evidence when tests actually exercise the
required entry point and state through them; unrelated passing tests do not
prove their composition.

For concurrency behavior, use events, barriers, controllable clocks, or fault
injection as appropriate. Fixed sleeps alone do not establish ordering or
cancellation. Unit/mock results do not prove live service or data-plane
integration; report unavailable required checks as verification gaps.

## Keep The Cost Proportional

Prefer existing fixtures, tools, and test seams. Add scaffolding only when a
concrete coverage gap needs it. A focused regression should not expand into a
test-framework redesign, new production abstractions solely for convenient
mocking, or tests for unnecessary helper infrastructure.

Keep cases that detect distinct relevant failures. Merge or remove redundant
tests introduced by the current change when they add no useful protection.
Existing unrelated test cleanup stays outside the task.

## Run And Stop

Use focused checks while developing, and satisfy the repository's and active
plan's required completion checks. Reuse results while relevant content and
conditions remain unchanged, following the policy's
[evidence applicability](../development_policy.md#evidence-applicability).

Stop when the affected behavior and required acceptance have sufficient evidence.
New changes, failures, or concrete unresolved questions justify additional
verification. Review or Git delivery alone does not justify rerunning checks.
Do not weaken an explicit acceptance requirement to reduce test effort; leave
unverified requirements open or obtain an agreed change.

## Examples

| Change or requirement | Useful evidence and scope |
| --- | --- |
| A purchase must leave the correct balance available on subsequent reads | Exercise purchase followed by a read through the relevant state/cache path; an assertion that an update method was called does not establish the returned balance. |
| Add an optional configuration field | Cover the new behavior and affected default or invalid-input handling; do not generate every combination of unrelated existing settings. |
| Edit ordinary explanatory prose | Check meaning and affected links; avoid tests that freeze wording unless that wording is itself a required contract. |
