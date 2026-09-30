# NWDAF implementation-related 3GPP specifications

This corpus is organized by 3GPP release so that specification text, package
attachments and schema dependencies from different releases remain isolated.

## Releases

| Release | Status | Specifications | Entry point |
|---|---|---:|---|
| Release 18 | Primary implementation corpus | 22 | [Rel-18](Rel-18/README.md) |
| Release 19 | Supplemental research corpus | 1 | [Rel-19](Rel-19/README.md) |
| Release 20 | Reference corpus | 5 | [Rel-20](Rel-20/README.md) |

## Layout rules

- Each release is self-contained under `Rel-<number>/`.
- A release manifest describes only specifications from that release.
- OpenAPI and other package attachments stay inside their source release.
- Cross-release references must name both releases explicitly and must not be
  resolved as same-release dependencies.
- The concise conversion guide is maintained in
  [`docs/spec_conversion.md`](../docs/spec_conversion.md).

## Fidelity boundary

- Clause wording is extracted deterministically from the supplied Word
  documents and is not rewritten by an LLM.
- Generated navigation is explicitly marked.
- Exact YAML and ABNF package attachments are retained byte-for-byte.
- Diagrams link to PNG previews where conversion succeeds; original EMF/WMF
  and embedded OLE/Visio payloads are retained.
- For high-stakes interpretation, the supplied original Word document remains
  the final reference.
