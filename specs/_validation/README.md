# Validation records

This directory records deterministic conversion and validation results for the complete Release 18 corpus.

## Latest extension

- Added TS 29.122 V18.10.0 in full, including all 16 official YAML attachments.
- Added TS 29.576 V18.8.0 in full, including both official YAML attachments.
- TS 29.122 originated as a legacy binary `.doc`; the original file was independently checked with `antiword`, while LibreOffice produced an intermediate DOCX for structural parsing.
- OpenAPI attachments are retained byte-for-byte; package versions and file-declared point releases are intentionally not normalized.

## Automated results

- Files: 4264
- Markdown files: 2297
- Manifest entries: 6279
- Local links checked: 2368
- Image references checked: 698
- OpenAPI YAML files parsed: 36
- Exact package attachments checked: 37
- Validation passed: True

## Fidelity method

- Pandoc supplies the primary structured extraction.
- `docx2txt` supplies an independent text path for DOCX sources.
- `antiword` supplies an independent path for legacy DOC sources.
- Tree reassembly hashes match the normalized preprocessed source for both newly added specifications.
- TS 29.576 retains identical `shall`, `should`, `may`, negative modal, NOTE and EXAMPLE counts across Pandoc and `docx2txt`.
- TS 29.122 uses legacy DOC; `antiword` differs in complex table serialization (97.32% token-multiset overlap and 78.54% 12-token sequence overlap). Exact primary/secondary modal counts are retained in `text-cross-check.json`; no LLM correction was applied.
- Modal terms, NOTE and EXAMPLE counts are recorded in `text-cross-check.json`; legacy DOC table serialization can reduce exact sequence overlap without implying prose rewriting.

## Known limitations

- This delivery does not claim page-by-page visual certification of the complete Word documents.
- Complex merged-cell tables remain HTML when Markdown pipe tables cannot represent them safely.
- Some large semantic clauses may exceed the nominal 4,200-word target because arbitrary paragraph-level splitting would damage procedure or table context.
- OpenAPI files may reference schemas from NF specifications not supplied here. Unresolved filenames are listed in `openapi/manifest.yaml`; no files from another Release are inserted automatically.
- The Markdown corpus is a retrieval derivative. For a high-stakes normative decision, confirm the clause against the original 3GPP Word document and exact package attachment.

## Detailed records

- `report.json`: concise corpus summary.
- `final-checks.json`: complete automated results.
- `text-cross-check.json`: independent extraction comparisons.
- `additional-ts29122-ts29576-build.json`: build and asset statistics for this extension.
- `deterministic-corrections.yaml`: heading and structure corrections.
- `CONVERSION_WORKFLOW.md`: reproducible conversion process.
