# Release 19 OpenAPI attachments

Official package attachments are copied byte-for-byte, without schema rewriting.

## Supplied packages

- `TS28105_AiMlNrm.yaml` — retained from `28105-j60.zip`.
- **TS 24.559**, `24559-j41.zip` (1 YAML file):
  - [TS24559_ADAE_ServiceConfiguration.yaml](TS24559_ADAE_ServiceConfiguration.yaml)

## External dependencies and release isolation

All files in this directory belong to Release 19. Missing external `$ref` targets are listed below. They remain unresolved; same-named files from another release must not be substituted automatically. This directory is an attachment collection, not a complete bundled API dependency set.

- `TS28104_MdaNrm.yaml`
- `TS28623_ComDefs.yaml`
- `TS28623_GenericNrm.yaml`
- `TS28623_ThresholdMonitorNrm.yaml`
- `TS29122_CommonData.yaml`
- `TS29520_Nnwdaf_EventsSubscription.yaml`
- `TS29523_Npcf_EventExposure.yaml`
- `TS29549_SS_ADAE_CollisionDetectionAnalytics.yaml`
- `TS29549_SS_ADAE_EdgeLoadAnalytics.yaml`
- `TS29549_SS_ADAE_LocationRelatedUeGroupAnalytics.yaml`
- `TS29549_SS_ADAE_Ue2UePerformanceAnalytics.yaml`
- `TS29549_SS_ADAE_VALPerformanceAnalytics.yaml`
- `TS29549_SS_UserProfileRetrieval.yaml`
- `TS29571_CommonData.yaml`

Word-to-Markdown conversion may not preserve the indentation of OpenAPI listings in Annexes. Use these official YAML attachments for machine-readable schemas.
