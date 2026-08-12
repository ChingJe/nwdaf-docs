# Validation records

This directory records deterministic conversion and validation results for the Release 18 main corpus plus isolated cross-release supplemental references.

## Latest adjustment

- Main corpus remains **22 complete Release 18 specifications**.
- TS 28.104 V18.7.0 and TS 28.105 V18.9.0 are not part of this build.
- Added **TS 28.105 V19.6.0** under `_supplemental/Rel-19/` for forward-looking AI/ML and federated-learning research.
- The Release 19 NRM YAML is isolated from the Release 18 `openapi/` tree to prevent accidental cross-release bundling.

## Automated results

- Build ID: `r18-core-20260812-rel19-ts28105-supplemental`
- Main Release 18 specifications: 22
- Supplemental specifications: 1
- Files: 7079
- Markdown files: 3818
- Manifest entries: 10938
- Local links checked: 3950; broken: 0
- Image references checked: 1123; broken: 0
- Main OpenAPI YAML: 61
- Supplemental Release 19 YAML: 1
- Validation passed: True

## Release 19 TS 28.105 fidelity

- Heading-tree reassembly hash match: True
- Independent normative marker counts match: True
- Token multiset overlap: 0.999372
- Original media: 31
- PNG previews: 31
- Embedded OLE/Visio objects: 9

## Cross-release boundary

The supplemental Release 19 TS 28.105 references `TS28104_MdaNrm.yaml, TS28623_ComDefs.yaml, TS28623_GenericNrm.yaml, TS28623_ThresholdMonitorNrm.yaml, TS29520_Nnwdaf_EventsSubscription.yaml`. These are intentionally not resolved against Release 18 files. In particular, the Release 18 `TS29520_Nnwdaf_EventsSubscription.yaml` may share a filename with a referenced schema, but it is not a same-release dependency for this Release 19 NRM.

## Visual validation

Level B risk-based validation was used. The original Rel-19 DOCX was exported fully to a 142-page PDF; pages covering the federated-learning procedure, FL information model/data types and Stage 3 AI/ML management were sampled. This is not page-by-page certification.
