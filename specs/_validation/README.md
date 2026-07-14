# Validation report

## Included additions

- TS 23.502 V18.14.0: selected clauses 1, 2, 3, 4.17 and 5.2.7.
- TS 29.510 V18.11.0: full specification, with Annex A represented by exact package YAML attachments.
- TS 29.571 V18.12.0: full specification, with Annex A represented by exact `TS29571_CommonData.yaml`.
- TS 29.500 V18.10.0: full specification and exact `TS29500_CustomHeaders.abnf`.

## Automated checks

- Files: 1575 (946 Markdown).
- Local links: 971 checked; 0 broken.
- Image references: 225 checked; 0 broken.
- Manifest entries: 2321; 0 missing paths.
- Exact attachments: 14/14 byte-identical.
- YAML parse results: 13/13 valid.
- Independent modal/NOTE/EXAMPLE counts match for every newly added specification: True.
- Source-structure samples: 13 clauses; minimum source token overlap 94.86%.

## Validation approach

- Pandoc provided the primary deterministic Word structure extraction.
- `docx2txt` provided an independent full-text path for every newly added DOCX.
- All four new specifications retained identical counts for `shall`, `should`, `may`, their negative forms, NOTE and EXAMPLE between the two extraction paths.
- Selected high-risk clauses were independently read from Word OOXML block order and compared against generated Markdown, including registration/discovery procedures, PUT tables, NFProfile, NwdafInfo, ProblemDetails and PatchItem.
- EMF/WMF previews were manually reviewed for NF registration, heartbeat and Bootstrapping URI figures.

## Known limitations

- Full-document LibreOffice PDF export for the largest 3GPP DOCX files did not finish within the execution limit. This delivery therefore does not claim page-by-page visual certification.
- Structural and textual validation is stronger than visual-layout validation: source block order, drawings, tables and text were checked directly from OOXML.
- Some source table cells split words across Word runs (for example `This` as `T` + `his`); Pandoc correctly joins these in Markdown, which explains lower raw token overlap in a few table samples without indicating lost content.
- TS 23.502 is intentionally partial. Its README explicitly lists the included scope.
- OpenAPI files still reference external API schemas not supplied in the uploaded ZIPs. These are listed in `openapi/manifest.yaml`; no cross-Release files were added automatically.
- Oversized semantic files are listed in `final-checks.json`; they were not split at unsafe arbitrary paragraph boundaries.

## Detailed records

- `report.json`: concise corpus summary.
- `final-checks.json`: complete automated results.
- `text-cross-check.json`: independent text-extraction comparisons.
- `structural-sampling.json`: source OOXML versus Markdown clause samples.
- `visual-sampling.json`: reviewed image previews and visual limitations.
- `deterministic-corrections.yaml`: heading and structure corrections.
- `CONVERSION_WORKFLOW.md`: reproducible conversion process.
