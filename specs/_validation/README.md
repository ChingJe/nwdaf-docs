# Validation report

## Included corpus

All listed specifications are included in full:

- TS 23.288 V18.13.0
- TS 23.501 V18.12.0
- TS 23.502 V18.14.0
- TS 29.500 V18.10.0
- TS 29.508 V18.11.0
- TS 29.510 V18.11.0
- TS 29.520 V18.14.0
- TS 29.564 V18.7.0
- TS 29.571 V18.12.0
- TS 29.575 V18.11.0

TS 23.502 has been replaced with the full specification; the earlier NRF-only scoped extract is no longer used.

## Automated checks

- Files: 4,009.
- Markdown files: 2,106.
- Local links: 2,142 checked; 0 broken.
- Image references: 683 checked; 0 broken.
- Manifest entries: 5,156; 0 missing paths.
- YAML files parsed: 19/19, including the OpenAPI manifest.
- Exact official attachments: 19/19 byte-identical to the supplied ZIPs (18 OpenAPI YAML files and one ABNF file).
- Practical retrieval checks for NRF registration, SMF event subscription, UPF event exposure, ADRF storage and ADRF discovery all returned relevant clause files.
- Final automated status: pass.

## Textual validation

- Pandoc supplied the primary deterministic Word structure extraction.
- `docx2txt` supplied an independent full-text path for DOCX sources.
- TS 29.508 originated as a legacy binary `.doc`; the original file was checked with `antiword`, while LibreOffice produced an intermediate DOCX for structural parsing.
- TS 23.501, TS 23.502, TS 29.564 and TS 29.575 retain identical counts for `shall`, `should`, `may`, negative modal forms, NOTE and EXAMPLE between the primary and independent paths.
- TS 29.508 has lower antiword sequence overlap because antiword omits and rearranges complex table cells. Examined differences were localized to table serialization, including missing table rows and split column labels. The generated Markdown follows the more complete Word-to-DOCX structure rather than antiword's reduced table output.
- Tree reassembly hashes match the normalized Pandoc source for every newly generated specification.

## Image and object preservation

| Specification | Original media | PNG previews | Embedded OLE/Visio payloads |
|---|---:|---:|---:|
| TS 23.501 | 150 | 150 | 150 |
| TS 23.502 | 280 | 153 | 280 |
| TS 29.508 | 10 | 10 | 9 |
| TS 29.564 | 10 | 10 | 9 |
| TS 29.575 | 25 | 25 | 24 |

For TS 23.502, 127 source vectors did not complete bounded PNG conversion. Their original EMF/WMF files and embedded source objects remain present, and Markdown references resolve to either a preview or the original vector.

## Known limitations

- This delivery does not claim page-by-page visual certification of the complete Word documents.
- Some large semantic clauses remain above the nominal 4,200-word target because arbitrary paragraph-level splitting would damage procedure or table context. They are listed in `final-checks.json`.
- OpenAPI files reference schemas from other NF specifications not supplied here. The unresolved filenames are listed in `openapi/manifest.yaml`; no files from another Release were inserted automatically.
- The Markdown corpus is a retrieval derivative. For a high-stakes normative decision, confirm the clause against the supplied original 3GPP Word document and the exact package OpenAPI attachment.

## Detailed records

- `report.json`: concise corpus summary.
- `final-checks.json`: complete automated results.
- `text-cross-check.json`: independent extraction comparisons for the ten specifications.
- `full-extension-build.json`: heading-tree, source-word and asset build statistics for the newly added specifications.
- `visual-sampling.json`: reviewed vector previews and visual limitations.
- `deterministic-corrections.yaml`: deterministic heading and structure corrections.
- `exact-attachment-checks.json`: byte-identity checks for YAML and ABNF attachments.
- `CONVERSION_WORKFLOW.md`: reproducible conversion process.
