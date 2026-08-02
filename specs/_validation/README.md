# Validation records

This directory records deterministic conversion and validation results for the complete Release 18 corpus.

## Latest extension

- Added TS 23.003 V18.8.0 in full.
- Added TS 29.502 V18.11.0 in full, including the exact Nsmf_PDUSession YAML and custom-header ABNF attachment.
- Added TS 29.504 V18.13.0 in full, including the exact Nudr_DataRepository and Nudr_GroupIDmap YAML files and custom-header ABNF attachment.
- All three sources are native DOCX and were independently extracted with `docx2txt`.

## Automated results

- Specifications: 21
- Files: 6745
- Markdown files: 3587
- Manifest entries: 9897
- Local links checked: 3716
- Image references checked: 1087
- OpenAPI YAML files parsed: 60
- Official YAML/ABNF attachments checked: 62/63
- Validation passed: True

## Fidelity method

- Pandoc supplies the primary structured extraction.
- `docx2txt` supplies an independent text path for native DOCX sources.
- Heading trees are reassembled and hash-checked before file partitioning.
- Official YAML and ABNF attachments are retained byte-for-byte.
- Complex merged-cell tables remain HTML when Markdown pipe tables cannot represent them safely.
- Original EMF/WMF and embedded OLE/Visio payloads are retained. PNG previews are produced where the conversion tool can render the source vector reliably.

## Known limitations

- This delivery does not claim page-by-page visual certification of every Word page.
- Six small WMF objects in TS 23.003 did not produce reliable PNG previews in the bounded vector-rendering pipeline; their original WMF files are retained and all Markdown references resolve to those originals.
- Some large semantic clauses may exceed the nominal 4,200-word target because arbitrary paragraph splitting would damage procedure or table context.
- OpenAPI files may reference schemas from NF specifications not supplied here. Unresolved filenames are listed in `openapi/manifest.yaml`.
- The Markdown corpus is a retrieval derivative. For a high-stakes normative decision, confirm the clause against the original 3GPP Word document and exact package attachment.

## Detailed records

- `report.json`: concise corpus summary.
- `final-checks.json`: complete automated results.
- `text-cross-check.json`: independent extraction comparisons.
- `additional-ts23003-ts29502-ts29504-build.json`: build and asset statistics for this extension.
- `new-attachment-checks.json`: byte-level checks for the new YAML and ABNF attachments.
- `deterministic-corrections.yaml`: heading and structure corrections.
- `CONVERSION_WORKFLOW.md`: reproducible conversion process.
