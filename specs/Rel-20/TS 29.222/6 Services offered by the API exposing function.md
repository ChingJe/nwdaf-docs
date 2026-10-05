---
spec: TS 29.222
version: 20.1.0
release: '20'
clause: 6
title: 6 Services offered by the API exposing function
source_archive: 29222-k10.zip
source_document: 29222-k10.docx
content_origin: 3gpp-source
---

# 6 Services offered by the API exposing function


## 6.1 Introduction of Services

The table 6.1-1 lists the AEF APIs below the service name. A service description clause for each API gives a general description of the related API.

Table 6.1-1: List of AEF Services

| Service Name     | Service Operations      | Operation Semantics | Consumer(s) |
|------------------|-------------------------|---------------------|-------------|
| AEF_Security_API | Initiate_Authentication | Request/ Response   | API Invoker |
|                  | Revoke_Authorization    | Request/ Response   | CCF         |

Table 6.1-2 summarizes the corresponding APIs defined in this specification.

Table 6.1-2: API Descriptions

| Service Name     | Clause | Description              | OpenAPI Specification File    | apiName      | Annex |
|------------------|--------|--------------------------|-------------------------------|--------------|-------|
| AEF_Security_API | 9.1    | AEF Security API Service | TS29222_AEF_Security_API.yaml | aef-security | A.10  |

## 6.2 AEF_Security_API


### 6.2.1 Service Description


#### 6.2.1.1 Overview

The AEF securityAPI, allows an API invokers via CAPIF-2/2e reference points to request API exposing function to ensure that authentication parameters necessary for authentication of the API invoker are available with the API exposing function. If the necessary authentication parameters are not available, the API exposing function fetches necessary authentication parameters from CAPIF core function to authenticate the API invoker.

The AEF security API, also allows the CAPIF core function via CAPIF-3/3e reference points to request API exposing function to revoke the authorization of service APIs for an API invoker.

### 6.2.2 Service Operations


#### 6.2.2.1 Introduction

The service operation defined for AEF_Security_API is shown in table 6.2.2.1-1.

Table 6.2.2.1-1: Operations of the AEF_Security_API

| Service operation name  | Description                                                                                                                                                              | Initiated by |
|-------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------|
| Initiate_Authentication | This service operation is used by an API Invoker to request API exposing function to confirm necessary authentication data is available to authenticate the API Invoker. | API Invoker  |
| Revoke_Authorization    | This service operation is used by the CCF to request the AEF to revoke the authorization of service API(s) for an API Invoker.                                           | CCF          |

#### 6.2.2.2 Initiate_Authentication


##### 6.2.2.2.1 General

This service operation is used by an API Invoker to initiate authentication with the AEF. On receiving the Initiate_Authentication, the AEF fetches the authentication information of the API Invoker from the CCF, if required.

##### 6.2.2.2.2 API invoker initiating authentication using Initiate_Authentication service operation

To initiate authentication with the AEF, the API Invoker shall send an HTTP POST request message to the AEF with the API Invoker ID to the URI "{apiRoot}/aef-security/v1/check-authentication".

Upon receiving the above described HTTP POST request message, the AEF shall check if the credentials of the API invoker for authentication are available with the AEF. If the credentials of the API invoker for authentication are not available, the AEF shall use the service defined in clause 5.6.2.4.2 to fetch the credentials from the CCF.

The AEF shall store the received credentials and respond to the API Invoker with an HTTP "200 OK" status code.

#### 6.2.2.3 Revoke_Authorization


##### 6.2.2.3.1 General

This service operation is used by CAPIF core function to revoke authorization of service APIs (e.g. due to policy change in the CAPIF core function). On receiving the Revoke_Authorization the API exposing function revokes authorization of the API invoker for the service APIs indicated in the request.

##### 6.2.2.3.2 CAPIF core function initiating revocation using Revoke_Authorization service operation

To revoke authorization, the CAPIF core function shall send an HTTP POST message to the API exposing function with the API invoker ID and a list of service API IDs on the URI "{apiRoot}/aef-security/v1/revoke-authorization".

Upon receiving the HTTP POST message, the API exposing function shall revoke the authorization of the API invoker for the indicated service APIs (e.g. it may update the list of unauthorized APIs locally), and then respond to the CAPIF core function with 200 OK status code.

The CAPIF core function shall also notify the API invoker of the authorization invalidation using the Notification Destination URI received in the Obtain_Security_Method message.
