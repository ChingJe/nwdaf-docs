---
spec: TS 29.558
version: 20.0.0
release: '20'
clause: 6B
title: '6B Services offered by the Cloud Enabler Server (CES)'
source_archive: 29558-k00.zip
source_document: 29558-k00.docx
content_origin: 3gpp-source
---

# 6B Services offered by the Cloud Enabler Server (CES)


## 6B.1 Introduction

Table 6B.1-1 lists the CES APIs defined in this specification.

Table 6B.1-1: List of CES Service APIs

| Service Name | Service Operations | Operation Semantics | Consumer(s) |
|--------------|--------------------|---------------------|-------------|
|              |                    |                     |             |

Table 6B.1-2 summarizes the corresponding CES APIs defined in this specification.

Table 6B.1-2: API Descriptions

| Service Name | Clause | Description | OpenAPI Specification File | apiName | Annex |
|--------------|--------|-------------|----------------------------|---------|-------|
|              |        |             |                            |         |       |

Table 6B.1-3 lists the EES APIs that are defined in this specification and may be reused (i.e., exposed) by the CES.

Table 6B.1-3: API Descriptions of service APIs reused by the CES

| Service Name                                           | Clause | OpenAPI Specification File                | apiName                   | Annex  |
|--------------------------------------------------------|--------|-------------------------------------------|---------------------------|--------|
| Eees_EASRegistration                                   | 5.2    | TS29558_Eees_EASRegistration.yaml         | eees-easregistration      | A.2    |
| Eees_UELocation                                        | 5.3    | TS29558_Eees_UELocation.yaml              | eees-uelocation           | A.3    |
| Eees_UEIdentifier                                      | 5.4    | TS29558_Eees_UEIdentifier.yaml            | eees-ueidentifier         | A.4    |
| Eees_AppClientInformation                              | 5.5    | TS29558_Eees_AppClientInformation.yaml    | eees-appclientinformation | A.5    |
| Eees_SessionWithQoS                                    | 5.6    | TS29558_Eees_SessionWithQoS.yaml          | eees-session-with-qos     | A.6    |
| Eees_ACRManagementEvent                                | 5.8    | TS29558_Eees_ACRManagementEvent.yaml      | eees-acrmgntevent         | A.7    |
| Eees_EECContextRelocation                              | 5.10   | TS29558_Eees_EECContextRelocation.yaml    | eees-eeccontextreloc      | A.8    |
| Eees_EELManagedACR                                     | 5.11   | TS29558_Eees_EELManagedACR.yaml           | eees-eel-acr              | A.9    |
| Eees_ACRStatusUpdate                                   | 5.12   | TS29558_Eees_ACRStatusUpdate.yaml         | eees-acrstatus-update     | A.10   |
| Eees_ACRParameterInformation                           | 5.13   | TS29558_Eees_ACRParameterInformation.yaml | eees-acr-param            | A.13   |
| Eees_CommonEASAnnouncement                             | 5.14   | TS29558_Eees_CommonEASAnnouncement.yaml   | eees-ceas                 | A.15   |
| Eees_TrafficInfluenceEAS                               | 5.15   | TS29558_Eees_TrafficInfluenceEAS.yaml     | eees-tie                  | A.17   |
| Eees_EASDiscovery                                      | (NOTE) | TS24558_Eees_EASDiscovery.yaml            | eees-easdiscovery         | (NOTE) |
| Eees_AppContextRelocation                              | (NOTE) | TS24558_AppContextRelocation              | eees-appctxtreloc         | (NOTE) |
| NOTE: These APIs are defined in 3GPP TS 24.558 \[14\]. |        |                                           |                           |        |

NOTE: In the provisions defining the above APIs reused by the CES and listed in Table 6B.1-3, the CES takes the role of the EES.
