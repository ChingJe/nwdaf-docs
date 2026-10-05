---
spec: TS 29.558
version: 20.0.0
release: '20'
clause: 6A
title: '6A Services offered by the Cloud Application Server (CAS)'
source_archive: 29558-k00.zip
source_document: 29558-k00.docx
content_origin: 3gpp-source
---

# 6A Services offered by the Cloud Application Server (CAS)


## 6A.1 Introduction

The table 6A.1-1 lists the CAS APIs below with the service name. A service description clause for each API gives a general description of the related API.

Table 6A.1-1: List of CAS Service APIs

| Service Name     | Service Operations | Operation Semantics | Consumer(s) |
|------------------|--------------------|---------------------|-------------|
| Ecas_SelectedEES | Declare            | Request/Response    | EES         |

Table 6A.1-2 summarizes the corresponding CAS APIs defined in this specification.

Table 6A.1-2: API Descriptions

| Service Name     | Clause | Description                                                            | OpenAPI Specification File    | apiName           | Annex |
|------------------|--------|------------------------------------------------------------------------|-------------------------------|-------------------|-------|
| Ecas_SelectedEES | 8A.1   | The service consumer declares the selected EES information to the CAS. | TS29558_Ecas_SelectedEES.yaml | ecas-selected-ees | A.14  |

## 6A.2 Ecas_SelectedEES Service


### 6A.2.1 Service Description

The Ecas_SelectedEES API, as defined in 3GPP TS 23.558 \[2\], allows a service consumer (e.g., EES) to inform the CAS of the selected EES during ACR.

### 6A.2.2 Service Operations


#### 6A.2.2.1 Introduction

The service operation defined for Ecas_SelectedEES API is shown in the table 6A.2.2.1-1.

Table 6A.2.2.1-1: Operations of the Ecas_SelectedEES API

|                          |                                                                                                                              |              |
|--------------------------|------------------------------------------------------------------------------------------------------------------------------|--------------|
| Service operation name   | Description                                                                                                                  | Initiated by |
| Ecas_SelectedEES_Declare | This service operation is used by the service consumer to inform the CAS of the selected EES during the ACR from EAS to CAS. | e.g., EES    |

#### 6A.2.2.2 Ecas_SelectedEES_Request


##### 6A.2.2.2.1 General

This service operation is used by a service consumer to inform the CAS of the selected EES.

##### 6A.2.2.2.2 Service consumer informing the CAS of the selected EES using Ecas_SelectedEES_Declare operation

To inform the CAS of the selected EES during the ACR, the service consumer shall send an HTTP POST request to the CAS targeting the URI of the corresponding custom operation (i.e., "Declare"), with the request body including the SelEESDecInfo data structure defined in clause 8A.1.6.2.2.

Upon reception of the HTTP POST request message from the service consumer, the CAS shall:

1\. check whether the service consumer is authorized to declare the selected EES information;

2\. if the service consumer is authorized, process the request, store the received information and respond with an HTTP "204 No Content" status code; and

3\. on failure, the CAS shall take proper error handling actions, as specified in clause 8A.1.7, and respond to the service consumer with an appropriate error status code.
