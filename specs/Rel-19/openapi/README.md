# Release 19 supplemental OpenAPI / NRM attachments

- [TS28105_AiMlNrm.yaml](TS28105_AiMlNrm.yaml) — copied byte-for-byte from `28105-j60.zip`.

## Release isolation

Dependencies are intentionally **not** resolved against the main Release 18 OpenAPI directory. The attachment references:

- `TS28104_MdaNrm.yaml`
- `TS28623_ComDefs.yaml`
- `TS28623_GenericNrm.yaml`
- `TS28623_ThresholdMonitorNrm.yaml`
- `TS29520_Nnwdaf_EventsSubscription.yaml`

Some filenames may exist in the Release 18 main corpus; using them to bundle this Release 19 schema would be a deliberate cross-release comparison, not a valid same-release bundle.
