# NWDAF Development Policy

This document defines NWDAF-specific development rules for the `NWDAF/`
repository in this workspace.

Use this document when the task changes code under `NWDAF/`.

Before editing, also read:

- `AGENTS.md` at the workspace root for workspace boundaries and general rules.
- `free5gc-dev-skill/SKILL.md` when the task needs free5GC-aligned structure,
  SBI behavior, OpenAPI guidance, or NF-style review/debug workflows.

---

## Scope And Boundaries

- Make a right-sized change that fully resolves the requested problem.
- Do not mix feature work with unrelated refactors.
- Preserve existing behavior unless the task explicitly requires a behavior change.
- Prefer complete fixes over narrow patches that leave known gaps, repeated
  follow-up work, or temporary workarounds in place.
- Treat `NWDAF/` as the implementation target unless the user explicitly asks
  to edit `nwdaf-docs/`, `free5gc-dev-skill/`, or `resources/`.
- Do not modify local reference trees such as
  `resources/references/free5gc-main/` or `resources/openapi/openapi/` unless
  the user explicitly asks.
- Do not mix commits across repositories. `NWDAF/`, `nwdaf-docs/`, and
  `free5gc-dev-skill/` are independently tracked repos.

## Architecture

- Follow the current NWDAF package boundaries instead of moving logic across
  packages casually.
- Keep shared state in `internal/context/` unless there is a clear ownership reason to move it.
- Prefer extending the current service flow over introducing parallel mechanisms.
- Keep free5GC-style boundaries where this repository already uses them, such
  as `context`, `consumer`, `processor`, `factory`, `service`, and config-driven
  wiring.

## Language

- `NWDAF/` repository content should use English strictly. This includes code,
  identifiers, comments, logs, API-facing strings, configuration-facing text,
  and committed implementation-oriented documents kept with the main project.
- `nwdaf-docs/` documentation may be written in either English or Traditional
  Chinese. Choose the language that best fits the document's audience and
  purpose, but keep each individual document internally consistent unless a
  bilingual format is intentionally required.
- Conversation-only notes may use Traditional Chinese.

## Code Quality

- Keep implementations explicit and readable; avoid speculative abstractions.
- Add comments only when the intent is not obvious from the code itself.
- Keep configuration naming aligned with the existing YAML layout and project terminology.
- Follow existing free5GC-style patterns where this repository already uses them.

## Change Safety

- If behavior changes, update or add tests in the same change.
- Prefer proving compatibility with existing code paths before introducing new ones.
- When introducing a new internal contract or state machine, document the lifecycle clearly.

## Decision Gates And Blocking Policy

- During planning or design work, stop and ask the user for a decision when
  there are meaningful design options, tradeoffs, or scope choices that affect
  the implementation direction.
- Do not silently replace the agreed plan with a different implementation strategy just because the original approach becomes inconvenient or blocked.
- Do not silently work around a missing dependency, tool, package, fixture, or environment requirement by inventing a weaker local substitute.
- Do not keep an implementation artificially small when doing so leaves a
  known defect, workaround, or incomplete fix in place.
- Stop and ask the user for a decision when the current plan depends on an assumption that turns out to be false.
- Stop and ask the user for a decision when the planned approach is blocked by missing dependencies, tooling, permissions, environment setup, or unavailable reference material.
- Stop and ask the user for a decision when the root cause extends beyond the
  initial task boundary and adjacent changes are needed to close the issue
  safely.
- Stop and ask the user for a decision before changing architecture boundaries, data flow, state flow, external contracts, verification scope, or the main implementation strategy.
- Stop and ask the user for a decision before lowering validation standards, skipping planned tests, or replacing a required dependency with a workaround.
- Stop and ask the user for a decision when the spec, plan, and existing code suggest conflicting directions and the difference affects implementation choices.
- Stop and ask the user for a decision when part of the remaining gap is
  expected to be resolved by planned future work and a temporary containment or
  alternative is being considered in the meantime.
- Small behavior-preserving cleanup may proceed without interruption when it stays within the already approved plan and does not change the chosen technical direction.

### Required Blocker Report

When blocked or when a design decision is needed, report all of the following before continuing:

1. The original plan or assumption.
2. The exact blocker or contradiction that was discovered.
3. The realistic options from that point.
4. The recommended option and its tradeoffs.
5. Whether the plan needs to be revised before implementation continues.

---

## Build And Verification

Use `NWDAF/Makefile` targets as the default workflow.

### Standard Commands

```bash
make build
make test
make lint
make run
```

### Minimum Expectation Before Commit

```bash
make build
make lint
```

Recommended for code changes:

```bash
make test
```

If a change only affects documentation or chat-support materials, a full test run may be skipped,
but the final note should say so explicitly.

## Optional Completion Review

After implementation, the user may ask for a focused review before commit or
before the task is considered complete.

Use this review direction when the user wants to confirm implementation
completeness, free5GC-style alignment, or newly introduced risks.

### Review Focus

Check the following:

1. Whether the implemented result actually satisfies the agreed plan, including
   required scope, behaviors, tests, and follow-up constraints.
2. Whether the implementation still aligns with the relevant free5GC
   conventions, exemplar NF patterns, and local NWDAF structure instead of only
   being functionally workable.
3. Whether the change introduced new problems such as regressions, incomplete
   coverage, inconsistent lifecycle handling, missing config updates, broken
   assumptions, or test gaps.

### Review Inputs

- The current plan document or agreed task scope.
- The actual code and config diff.
- The exemplar NF references used for the implementation when free5GC
  alignment is part of the task.
- Verification results and any checks that were skipped.

### Review Output Expectations

- State clearly whether the plan was fully satisfied, partially satisfied, or
  diverged in some areas.
- Call out free5GC alignment conclusions using concrete exemplar NF evidence
  when that comparison is part of the judgment.
- Lead with bugs, regressions, missing requirements, and missing tests.
- Separate confirmed issues from assumptions or unresolved questions.
- State what was verified and what remains unverified.
- If the implementation diverged from the plan, say whether the fix should be
  additional implementation work or a deliberate replan.

When the review needs detailed free5GC-oriented comparison, use
`free5gc-dev-skill/SKILL.md` and its review guidance as the default review
entry point.

---

## Reference Order

When NWDAF implementation details are unclear, use the following references in
this order:

1. Existing code in `NWDAF/`
2. Local Release 18 OpenAPI YAML under `nwdaf-docs/specs/openapi/`
3. Relevant 3GPP TS material under `nwdaf-docs/specs/`
4. `free5gc-dev-skill/` guidance
5. `resources/openapi/openapi/`
6. `resources/references/free5gc-main/`

### Practical Guidance

- Use existing `NWDAF/` code for local conventions, ownership boundaries, and
  integration constraints.
- Use local OpenAPI YAML for schema, field, request, and response definitions.
  Check `nwdaf-docs/specs/openapi/README.md` before assuming all external
  `$ref` dependencies are present in the corpus.
- Use 3GPP TS material for procedure intent, role boundaries, and service
  semantics.
- Use `free5gc-dev-skill/` for free5GC-style development and review workflow.
- Use free5GC OpenAPI code and reference implementations for alignment, not as
  a reason to force unrelated rewrites.

---

## Commit Format

This commit message format applies only to commits for the main `NWDAF/`
repository. It does not define commit message rules for `nwdaf-docs/`,
`free5gc-dev-skill/`, or other workspace repositories.

The rules in this section apply only to the commit message itself. They do not
require plan documents, task titles, branch names, PR titles, or issue labels
to use the same format.

Use a conventional commit subject, and add a body when extra context is useful.

### Subject

```text
<type>: <summary>
<type>(<scope>): <summary>
```

Examples:

```text
feat(mtlf): add retrieval session logging
fix(processor): split UPF notification items before ADRF storage
docs: refresh development policy
```

### Body

- Use the body to explain why the change was made, key behavior changes, migration notes, or follow-up constraints.
- Keep the body factual and concise.
- Use bullets when listing multiple concrete points.

Example:

```text
feat(mtlf): add retrieval session logging

- log retrieval session creation and completion
- keep existing retrain flow unchanged
- add context needed for callback debugging
```

### Allowed types

- `feat`
- `fix`
- `docs`
- `refactor`
- `test`
- `chore`

### Commit Subject Rules

- Use imperative mood.
- Keep the subject concise and specific.
- Prefer a scope when the affected area is clear, such as `anlf`, `mtlf`, `processor`, `context`, `consumer`, or `docs`.
- Describe the concrete technical change, not an internal tracking label.
- Avoid vague project-management terms such as `phase`, `batch`, `round`, `priority`, or
  similar labels in the subject unless that wording is itself part of the
  externally meaningful change being made.
- A reviewer should be able to infer the main code or document delta from the
  subject alone, without needing separate plan context.

---

## Common Workflow

### Typical Code Change Flow

1. Read the relevant package and confirm the existing control flow.
2. Check the applicable spec or schema if the change affects API or 3GPP
   semantics.
3. If the task needs free5GC alignment, read `free5gc-dev-skill/SKILL.md` and
   the routed references it points to.
4. Confirm the current plan, assumptions, dependencies, and verification scope before editing.
5. If planning or design requires a meaningful decision about scope, tradeoffs,
   or implementation direction, stop and ask the user before continuing.
6. If a blocker, contradiction, or design fork appears, stop and ask the user for a decision instead of silently switching strategies.
7. Replan when the original plan is no longer valid, then continue only after the direction is clarified.
8. Implement the right-sized change that closes the confirmed problem and still
   matches the clarified plan.
9. Add or update tests.
10. Run verification commands.
11. If requested, perform an implementation-completion review against the plan,
    free5GC alignment expectations, and newly introduced risks before commit.
12. Write a focused commit message using the format above.

### Typical Documentation Change Flow

1. Update the canonical document first.
2. Keep workspace-level rules in `AGENTS.md`.
3. Keep NWDAF-specific stable rules in this document.
4. Avoid duplicating long-term rules across multiple files.
