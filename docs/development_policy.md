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

- Make the smallest change that fully solves the requested problem.
- Do not mix feature work with unrelated refactors.
- Preserve existing behavior unless the task explicitly requires a behavior change.
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

- Code, identifiers, logs, API-facing strings, and committed documents should use English.
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

- Do not silently replace the agreed plan with a different implementation strategy just because the original approach becomes inconvenient or blocked.
- Do not silently work around a missing dependency, tool, package, fixture, or environment requirement by inventing a weaker local substitute.
- Stop and ask the user for a decision when the current plan depends on an assumption that turns out to be false.
- Stop and ask the user for a decision when the planned approach is blocked by missing dependencies, tooling, permissions, environment setup, or unavailable reference material.
- Stop and ask the user for a decision before changing architecture boundaries, data flow, state flow, external contracts, verification scope, or the main implementation strategy.
- Stop and ask the user for a decision before lowering validation standards, skipping planned tests, or replacing a required dependency with a workaround.
- Stop and ask the user for a decision when the spec, plan, and existing code suggest conflicting directions and the difference affects implementation choices.
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

---

## Reference Order

When NWDAF implementation details are unclear, use the following references in
this order:

1. Existing code in `NWDAF/`
2. Local OpenAPI YAML under `nwdaf-docs/specs/yaml/`
3. Relevant 3GPP TS material under `nwdaf-docs/specs/`
4. `free5gc-dev-skill/` guidance
5. `resources/openapi/openapi/`
6. `resources/references/free5gc-main/`

### Practical Guidance

- Use existing `NWDAF/` code for local conventions, ownership boundaries, and
  integration constraints.
- Use local OpenAPI YAML for schema, field, request, and response definitions.
- Use 3GPP TS material for procedure intent, role boundaries, and service
  semantics.
- Use `free5gc-dev-skill/` for free5GC-style development and review workflow.
- Use free5GC OpenAPI code and reference implementations for alignment, not as
  a reason to force unrelated rewrites.

---

## Commit Format

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
- Avoid vague project-management terms such as `phase`, `batch`, `round`, or
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
5. If a blocker, contradiction, or design fork appears, stop and ask the user for a decision instead of silently switching strategies.
6. Replan when the original plan is no longer valid, then continue only after the direction is clarified.
7. Implement the smallest coherent change that still matches the confirmed plan.
8. Add or update tests.
9. Run verification commands.
10. Write a focused commit message using the format above.

### Typical Documentation Change Flow

1. Update the canonical document first.
2. Keep workspace-level rules in `AGENTS.md`.
3. Keep NWDAF-specific stable rules in this document.
4. Avoid duplicating long-term rules across multiple files.
