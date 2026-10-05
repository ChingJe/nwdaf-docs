---
spec: TS 23.222
version: 20.1.0
release: '20'
clause: 11
title: 11 API exposing function APIs
source_archive: 23222-k10.zip
source_document: 23222-k10.docx
content_origin: 3gpp-source
---

# 11 API exposing function APIs


## 11.1 General

Table 11.1-1 illustrates the API exposing function APIs.

Table 11.1-1: List of API exposing function APIs

| API Name         | API Operations          | Known Consumer(s)   | Communication Type |
|------------------|-------------------------|---------------------|--------------------|
| AEF_Security API | Revoke_Authorization    | CAPIF Core Function | Request/ Response  |
|                  | Initiate_Authentication | API Invoker         | Request/ Response  |

## 11.2 AEF_Security API


### 11.2.1 General

**API description:** This API allows CAPIF core function to revoke access to service APIs and API invokers to request the authentication parameters necessary for authentication of the API invoker available with the API exposing function.

### 11.2.2 Revoke_Authorization operation

**API operation name:** Revoke_Authorization

**Description:** Revokes API invoker authorization to access service API.

**Known Consumers:** CAPIF core function.

**Inputs:** Refer subclause 8.23.2.

**Outputs:** Refer subclause 8.23.2.

See subclause 8.23.4 for the details of usage of this API operation.

### 11.2.3 Initiate_Authentication operation

**API operation name:** Initiate_Authentication

**Description:** Authentication between the API invoker and the AEF prior to service API invocation.

**Known Consumers:** API Invoker.

**Inputs:** Refer subclause 8.14.2.

**Outputs:** Refer subclause 8.14.2.

See subclause 8.14.3 for the details of usage of this API operation.
