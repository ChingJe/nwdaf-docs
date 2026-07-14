---
spec: TS 29.500
version: 18.10.0
release: '18'
clause: Annex B
title: 'Annex B (normative): 3gpp-Sbi-Callback Types'
source_archive: 29500-ia0.zip
source_document: 29500-ia0.docx
source_archive_sha256: 3e0cbb6bc6ceec4dabb597410fb4b55ba03ae82e9b242972f441cd926971f5f2
source_document_sha256: 8cd4994735a1718db905b0aee4ceba4fb479cc7193d82d051a68450f33eba52b
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex B (normative): 3gpp-Sbi-Callback Types


This annex specifies allowed 3GPP SBI callback type values for the "3gpp-Sbi-Callback" HTTP custom header specified in clause 5.2.3.2.3.

Table B-1 contains a non-exhaustive list of callbacks that are invoked in the 5GS.

Table B-1: Non-exhaustive list of values for the "3gpp-Sbi-Callback" Custom HTTP Header

| Value for "3gpp-Sbi-Callback" Custom HTTP Header | Reference                                                                            |
|--------------------------------------------------|--------------------------------------------------------------------------------------|
| "Nsmf_PDUSession_Update"                         | 3GPP TS 29.502 \[28\], Clause 5.2.2.8.3.2, 5.2.2.8.3.3, 5.2.2.8.3.4 and 5.2.2.8.3.5. |
| "Nsmf_PDUSession_StatusNotify"                   | 3GPP TS 29.502 \[28\], Clause 5.2.2.10.                                              |
| "Nudm_SDM_Notification"                          | 3GPP TS 29.503 \[29\], Clause 6.1.5.2                                                |
| "Nudm_UECM_DeregistrationNotification"           | 3GPP TS 29.503 \[29\], Clause 6.2.5.2                                                |
| "Nudm_UECM_PCSCFRestorationNotification"         | 3GPP TS 29.503 \[29\], Clause 6.2.5.3                                                |
| "Nnrf_NFManagement_NFStatusNotify"               | 3GPP TS 29.510 \[8\], Clause 6.1.5.2.                                                |
| "Namf_EventExposure_Notify"                      | 3GPP TS 29.518 \[31\], Clause 6.2.5.2.                                               |
| "Npcf_UEPolicyControl_UpdateNotify"              | 3GPP TS 29.525 \[35\], Clauses 4.2.4, 5.5.2 and 5.5.3.                               |
| "Nnssf_NSSAIAvailability_Notification"           | 3GPP TS 29.531 \[32\], Clause 6.2.5.2                                                |
| "Namf_Communication_AMFStatusChangeNotify"       | 3GPP TS 29.518 \[31\], Clause 6.1.5.2.                                               |
| "Ngmlc_Location_EventNotify"                     | 3GPP TS 29.515 \[40\], Clause 6.1.4.2.                                               |
| "Nchf_ConvergedCharging_Notify"                  | 3GPP TS 32.291 \[42\], Clause 6.1.5.2                                                |
| "Nnssaaf_NSSAA_ReAuthentication"                 | 3GPP TS 29.526 \[44\], Clause 6.1.5.2.                                               |
| "Nnssaaf_NSSAA_Revocation"                       | 3GPP TS 29.526 \[44\], Clause 6.1.5.3.                                               |
| "N5g-ddnmf_Discovery_MonitorUpdateResult"        | 3GPP TS 29.555 \[46\], Clause 6.1.5.2.                                               |
| "N5g-ddnmf_Discovery_MatchInformation"           | 3GPP TS 29.555 \[46\], Clause 6.1.5.3.                                               |
| …                                                | …                                                                                    |

For notification and callback service operations (used across PLMNs or within a PLMN) that are not part of TableB.1, the value of the header shall be constructed as follows:

"\<API name taken from the heading of the relevant annex A.x as defined in the corresponding 3GPP TS of that API\>\_\<name of the callback service operation in the corresponding OpenAPI specification file\>"

EXAMPLE: Nsmf_PDUSession_smContextStatusNotification (for the Notify SM Context Status service operation)

where the "smContextStatusNotification" correspond to:

callbacks:

smContextStatusNotification:

'{\$request.body#/smContextStatusUri}':

NOTE: Several values in Table B-1 do not comply with the construction rule for the "3gpp-Sbi-Callback" HTTP header described in this clause; in those cases, the values explicitly included in Table B-1 take precedence over the construction rule.

Values for the "3gpp-Sbi-Callback" Custom HTTP Header shall be case-insensitive.
