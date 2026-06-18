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

---

## Common Workflow

### Typical Code Change Flow

1. Read the relevant package and confirm the existing control flow.
2. Check the applicable spec or schema if the change affects API or 3GPP
   semantics.
3. If the task needs free5GC alignment, read `free5gc-dev-skill/SKILL.md` and
   the routed references it points to.
4. Implement the smallest coherent change.
5. Add or update tests.
6. Run verification commands.
7. Write a focused commit message using the format above.

### Typical Documentation Change Flow

1. Update the canonical document first.
2. Keep workspace-level rules in `AGENTS.md`.
3. Keep NWDAF-specific stable rules in this document.
4. Avoid duplicating long-term rules across multiple files.
