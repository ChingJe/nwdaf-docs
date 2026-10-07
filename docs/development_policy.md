# NWDAF Development Policy

This entry routes workspace development, analysis, documentation, and delivery
to the rules for the task. The repository map is in the workspace `AGENTS.md`.
Detailed rules have one owner; plans define task-specific behavior and acceptance.
Generic workflow descriptions in plans defer to the current shared policy;
their task-specific acceptance criteria remain binding.

## Scope And Common Principles

These policies cover `NWDAF/`, `PyAnLF/`, `PyMTLF/`, `nrf/`, `smf-nwdaf-ext/`,
`udm/`, `udr/`, `adrf/`, and the workspace's supporting code and documentation.
Repository-specific commands retain their explicit scope.

Complete the requested work within its agreed boundaries. Prefer established
owners and existing tools; unrelated refactoring and speculative hardening do
not expand the task. Current user instructions take precedence over older
workflow requirements in plans or skills.

## Workspace Boundaries And Execution

Repositories are independent. Preserve unrelated user changes and modify both
sides of a contract only when the requested behavior requires it. Reference
trees are read-only unless the user authorizes modification.

Discussion, diagnosis, and review authorize inspection and reporting.
Implementation requests authorize in-scope edits and verification. Git
authorization and document-status transitions belong to
[Delivery](development-policy/delivery.md).

Use elevated permissions for network operations and all script/code execution,
including helpers, tests, and local services. This includes execution touching
networking, namespaces, iptables, installation, or paths outside the workspace.
Changing tools does not bypass required permissions.

## Task Routing

| Concern | Reference |
| --- | --- |
| Substantial implementation scope, baseline, or acceptance | [Planning](development-policy/planning.md) |
| Code changes, defect remediation, or repository test commands | [Implementation](development-policy/implementation.md) |
| Test targets, design, scope, or stopping conditions | [Testing](development-policy/testing.md) |
| Ownership, package placement, cross-boundary design, or feasibility | [Architecture](development-policy/architecture.md) |
| Independent code/test review, findings, or plan conformance | [Review](development-policy/review.md) |
| Document language, ownership, or navigation | [Documentation](development-policy/documentation.md) |
| User-review handoff, commit, push, or status updates | [Delivery](development-policy/delivery.md) |

## Loading And Evidence Reuse

Read the sections relevant to the current decision. Follow references when
their subject applies; a link does not make the entire linked document required
reading. Use established context through ordinary follow-ups.

### Recovery After Context Compaction

Use the summary to recover the objective, scope, decisions, progress, and open
items. Consult original instructions, plans, or evidence where context is
missing, uncertain, or changed. Recovery does not require a new plan or record.

### Evidence Applicability

Verification follows the changed behavior and acceptance criteria. Existing
results cover unchanged relevant content and conditions; changed inputs or an
unresolved failure may require new evidence.

## Evidence And Reference Order

Use the target code and tests to establish implementation behavior, and the
active plan for agreed decisions. For standardized behavior, use the relevant
release's OpenAPI and TS corpus under `specs/`. Release 18 is the implementation
baseline; Release 19/20 material applies when the task concerns those releases.
The release's OpenAPI README identifies external dependency coverage.

OpenAPI defines paths, methods, fields, statuses, headers, and schemas; TS
defines procedure intent and roles. Generated code and free5GC exemplars show
implementation patterns without overriding those contracts. Local mirrors do
not establish the latest upstream behavior.

Distinguish specification-defined behavior, observed implementation, inference,
and proposed design. Cite the decisive clause or code for technical conclusions,
including material counterevidence or uncertainty. Keep answers focused on the
question; investigation depth does not require a long response.

## Decision Gates

Continue ordinary implementation choices within the user's scope. Ask when a
decision is needed to change agreed ownership, architecture, external contracts,
product behavior, dependencies, or acceptance; to implement another phase; or
to replace an agreed strategy whose assumptions no longer hold. Required
permissions and genuinely missing inputs may also need user action.

Describe the concrete conflict and available choices. Optional cleanup,
speculative hardening, and future work alone do not block the current task.
