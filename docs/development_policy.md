# NWDAF Development Policy

This document is the long-term development policy for this project.
Keep stable development rules here.

---

## Core Rules

### Scope control

- Make the smallest change that fully solves the requested problem.
- Do not mix feature work with unrelated refactors.
- Preserve existing behavior unless the task explicitly requires a behavior change.

### Architecture

- Follow the current NWDAF package boundaries instead of moving logic across packages casually.
- Keep shared state in `internal/context/` unless there is a clear ownership reason to move it.
- Prefer extending the current service flow over introducing parallel mechanisms.

### Language

- Code, identifiers, logs, API-facing strings, and committed documents should use English.
- Conversation-only notes may use Traditional Chinese.

### Code quality

- Keep implementations explicit and readable; avoid speculative abstractions.
- Add comments only when the intent is not obvious from the code itself.
- Keep configuration naming aligned with the existing YAML layout and project terminology.
- Follow existing free5GC-style patterns where this repository already uses them.

### Change safety

- If behavior changes, update or add tests in the same change.
- Prefer proving compatibility with existing code paths before introducing new ones.
- When introducing a new internal contract or state machine, document the lifecycle clearly.

---

## Build And Verification

Use the repository `Makefile` targets as the default workflow.

### Standard commands

```bash
make build
make test
make lint
```

### Minimum expectation before commit

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

### Commit subject rules

- Use imperative mood.
- Keep the subject concise and specific.
- Prefer a scope when the affected area is clear, such as `anlf`, `mtlf`, `processor`, `context`, `consumer`, or `docs`.

---

## Reference Order

When implementation details are unclear, use the following references in this order:

1. YAML OpenAPI specs under `.agent/specs/yaml/`
2. Relevant 3GPP TS excerpts under `.agent/specs/`
3. Existing code in this repository
4. free5GC OpenAPI/models under `.agent/openapi/`
5. free5GC reference implementations under `.agent/references/free5gc-main/`

### Practical guidance

- Use YAML OpenAPI specs for schema and field definitions.
- Use 3GPP TS excerpts for procedure intent, role boundaries, and service semantics.
- Use the current repository implementation for local conventions and integration constraints.
- Use free5GC reference code for style alignment, not as a reason to force unrelated rewrites.

---

## Common Workflow

### Typical code change flow

1. Read the relevant package and confirm the existing control flow.
2. Check the applicable spec or schema if the change affects API or 3GPP semantics.
3. Implement the smallest coherent change.
4. Add or update tests.
5. Run verification commands.
6. Write a focused commit message using the format above.

### Typical documentation change flow

1. Update the canonical document first.
2. Keep `.agent/` descriptions short and chat-oriented.
3. Avoid duplicating long-term rules across multiple files.
