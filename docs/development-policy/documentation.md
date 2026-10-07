# Documentation

Use this module when drafting, changing, or reviewing documentation, including
plans and durable review records. Substantial implementation planning also uses
[Planning](planning.md); finding/conformance work uses [Review](review.md).
Preparing user-review or Git delivery uses [Delivery](delivery.md).

## Language

Select prose language in this order:

1. explicit user instruction;
2. the existing document's dominant prose language;
3. the established language of current sibling documents in the same series;
4. the repository default.

New documents extending a series inherit its language unless the user requests
otherwise. `nwdaf-docs/` supports English and Traditional Chinese, consistently
within each document. Implementation-facing documents in production repositories
default to English unless their established convention differs.

Code identifiers, API names, paths, configuration keys, protocol terms, status
values, and direct quotations do not change the selected prose language.
Headings, sentences, explanations, table labels, checklists, captions, and
decision/status prose use it consistently.

Review changed prose in context for language consistency. Consult sibling
documents when the series convention is needed to choose language or style.
Language and technical content can be reviewed together; a small edit does not
require a separate full-document language pass or a language-check report.

## Document Ownership

Workspace environment and reference routing belong in `AGENTS.md`; shared
boundaries and policy selection belong in the policy entry; detailed rules
belong in their modules. Plans own task-specific decisions and acceptance,
and link to shared workflow rules rather than restating them.

Create/update a durable review record only if requested by the user or required
by the active plan. Maintain one ledger per implementation phase, recording
finding ID/status, owner phase, confirmed evidence, remediation, verification,
and closing commit when available. Append short iteration records rather than
new full documents for each remediation pass.

A separate document is appropriate when architecture, product scope, or the
canonical plan changes. Preserve historical review facts rather than rewriting
history to consolidate old files. Current statuses are synchronized at the
[delivery events](delivery.md#review-confirmation-and-document-status).

## Verification

Confirm that changed content meets the requirements and affected links remain
usable. Official specifications and attachments follow
[Specification Conversion](../spec_conversion.md). Prose-only changes do not
need application tests.
