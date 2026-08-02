# Validation records

This directory records deterministic conversion and validation results for the complete Release 18 corpus.

## Latest extension

- Added TS 29.505 V18.8.0 in full.
- Added the official `TS29505_Subscription_Data.yaml` attachment byte-for-byte.
- Closed the direct `TS29504_Nudr_DR.yaml` → `TS29505_Subscription_Data.yaml` dependency.

## Automated results

- Build ID: `r18-corpus-20260803-ts29505`
- Specifications: 22
- Files: 6927
- Markdown files: 3746
- Manifest entries: 10531
- Local links checked: 3876
- Image references checked: 1092
- OpenAPI YAML files parsed: 61/61
- Official YAML/ABNF attachments checked: 63/63
- Validation passed: True

## TS 29.505 fidelity

- Heading-tree reassembly hash match: True
- Independent normative marker counts match: True
- Token multiset overlap: 0.999891
- Original media: 5
- PNG previews: 5
- Embedded OLE/Visio objects: 5

## Fidelity method

- Pandoc supplies the primary structured extraction.
- `docx2txt` supplies an independent text path from the original DOCX.
- The heading tree is reassembled and SHA-256 checked before file partitioning.
- Complex merged-cell tables remain HTML when Markdown tables cannot safely represent them.
- The official YAML is retained byte-for-byte.
- Original EMF and embedded Visio/OLE payloads are retained with PNG previews.

## Visual validation

TS 29.505 used Level B risk-based visual validation. All five extracted diagrams were rendered and inspected. This build does not claim page-by-page visual certification of the complete Word document.

## Known limitations

- Some atomic clauses may exceed the nominal 4,200-word target when a safe semantic split is unavailable.
- OpenAPI files still reference schemas from specifications outside the current project scope. These are listed in `openapi/manifest.yaml`.
- The Markdown corpus is a retrieval derivative. High-stakes normative decisions should be checked against the original 3GPP Word document and exact package attachment.

## Detailed records

- `build.json`: clean-build identity and parent corpus.
- `environment.json`: tool versions.
- `source-inventory-ts29505.json`: source hashes and members.
- `commands-ts29505.log`: executed conversion phases.
- `report.json`: concise corpus summary.
- `final-checks.json`: complete automated results.
- `text-cross-check.json`: independent extraction comparisons.
- `additional-ts29505-build.json`: TS 29.505 tree and asset statistics.
- `official-attachment-checks.json`: byte-level attachment checks.
- `visual-sampling-ts29505.json`: visual QA scope.
- `deterministic-corrections.yaml`: heading and structure corrections.
- `CONVERSION_WORKFLOW.md`: reproducible conversion process v2.
