---
spec: TS 29.222
version: 20.1.0
release: '20'
clause: 9
title: 9 AEF API Definition
source_archive: 29222-k10.zip
source_document: 29222-k10.docx
content_origin: 3gpp-source
---

# 9 AEF API Definition


## 9.1 AEF_Security_API


### 9.1.1 API URI

The AEF_Security_API service shall use the AEF_Security_API.

The request URIs used in HTTP requests from the API invoker towards the API exposing function shall have the Resource URI structure defined in clause 7.5 with the following clarifications:

\- The \<apiName\> shall be "aef-security".

\- The \<apiVersion\> shall be "v1".

\- The \<custOpName\> shall be set as described in clause 9.1.2a.

All the resource URIs and the custom operation URIs specified in the clauses below are defined relative to the above API URI.

### 9.1.2 Resources

There is no resource defined for this API.

### 9.1.2A Custom Operations without associated resources


#### 9.1.2A.1 Overview

Custom operations used for this API are summarized in table 9.1.2A.1-1. "{apiRoot}" and "\<apiVersion\>" are set as described in clause 7.5 and clause 9.1.1 respectively.

Table 9.1.2A.1-1: Custom operations without associated resources

| Operation name       | Custom operation URI  | Mapped HTTP method | Description                            |
|----------------------|-----------------------|--------------------|----------------------------------------|
| check-authentication | /check-authentication | POST               | Check authentication request.          |
| revoke-authorization | /revoke-authorization | POST               | Revoke authorization for service APIs. |

#### 9.1.2A.2 Operation: check-authentication


##### 9.1.2A.2.1 Description

This custom operation allows the API Invoker to confirm from the AEF, that necessary authentication data is available to authenticate the API Invoker on API invocation.

##### 9.1.2A.2.2 Operation Definition

This method shall support the URI query parameters specified in table 9.1.2A.2.2-1.

Table 9.1.2A.2.2-1: URI query parameters supported by the POST method on this resource

| Name | Data type | P   | Cardinality | Description |
|------|-----------|-----|-------------|-------------|
| n/a  |           |     |             |             |

This operation shall support the request and response data structures, and response codes specified in tables 9.1.2A.2.2-2 and 9.1.2A.2.2-3.

Table 9.1.2A.2.2-2: Data structures supported by the POST Request Body on this operation

|                        |     |             |                                    |
|------------------------|-----|-------------|------------------------------------|
| Data type              | P   | Cardinality | Description                        |
| CheckAuthenticationReq | M   | 1           | Authentication check request data. |

Table 9.1.2A.2.2-3: Data structures supported by the POST Response Body on this operation

<table>
<colgroup>
<col style="width: 16%" />
<col style="width: 4%" />
<col style="width: 12%" />
<col style="width: 11%" />
<col style="width: 54%" />
</colgroup>
<tbody>
<tr class="odd">
<td>Data type</td>
<td>P</td>
<td>Cardinality</td>
<td><p>Response</p>
<p>codes</p></td>
<td>Description</td>
</tr>
<tr class="even">
<td>CheckAuthenticationRsp</td>
<td>M</td>
<td>1</td>
<td>200 OK</td>
<td>The request was successful.</td>
</tr>
<tr class="odd">
<td>n/a</td>
<td></td>
<td></td>
<td>307 Temporary Redirect</td>
<td><p>Temporary redirection.</p>
<p>The response shall include a Location header field containing an alternative URI of the resource located in an alternative AEF.</p>
<p>Redirection handling is described in clause 5.2.10 of 3GPP TS 29.122 [14].</p></td>
</tr>
<tr class="even">
<td>n/a</td>
<td></td>
<td></td>
<td>308 Permanent Redirect</td>
<td><p>Permanent redirection.</p>
<p>The response shall include a Location header field containing an alternative URI of the resource located in an alternative AEF.</p>
<p>Redirection handling is described in clause 5.2.10 of 3GPP TS 29.122 [14].</p></td>
</tr>
<tr class="odd">
<td colspan="5">NOTE: The mandatory HTTP error status codes for the HTTP POST method listed in table 5.2.6-1 of 3GPP TS 29.122 [14] shall also apply.</td>
</tr>
</tbody>
</table>

Table 9.1.2A.2.2-4: Headers supported by the 307 Response Code on this resource

|          |           |     |             |                                                                            |
|----------|-----------|-----|-------------|----------------------------------------------------------------------------|
| Name     | Data type | P   | Cardinality | Description                                                                |
| Location | string    | M   | 1           | Contains an alternative URI of the resource located in an alternative AEF. |

Table 9.1.2A.2.2-5: Headers supported by the 308 Response Code on this resource

|          |           |     |             |                                                                            |
|----------|-----------|-----|-------------|----------------------------------------------------------------------------|
| Name     | Data type | P   | Cardinality | Description                                                                |
| Location | string    | M   | 1           | Contains an alternative URI of the resource located in an alternative AEF. |

#### 9.1.2A.3 Operation: revoke-authorization


##### 9.1.2A.3.1 Description

This custom operation allows the CAPIF core function to request the API exposing function to revoke the authorization of the API invoker for the indicated service APIs.

##### 9.1.2A.3.2 Operation Definition

This method shall support the URI query parameters specified in table 9.1.2A.3.2-1.

Table 9.1.2A.3.2-1: URI query parameters supported by the POST method on this resource

| Name | Data type | P   | Cardinality | Description |
|------|-----------|-----|-------------|-------------|
| n/a  |           |     |             |             |

This operation shall support the request and response data structures, and response codes specified in tables 9.1.2A.3.2-2 and 9.1.2A.3.2-3.

Table 9.1.2A.3.2-2: Data structures supported by the POST Request Body on this operation

|                        |     |             |                                       |
|------------------------|-----|-------------|---------------------------------------|
| Data type              | P   | Cardinality | Description                           |
| RevokeAuthorizationReq | M   | 1           | Authorization revocation request data |

Table 9.1.2A.3.2-3: Data structures supported by the POST Response Body on this operation

<table>
<colgroup>
<col style="width: 16%" />
<col style="width: 4%" />
<col style="width: 12%" />
<col style="width: 11%" />
<col style="width: 54%" />
</colgroup>
<tbody>
<tr class="odd">
<td>Data type</td>
<td>P</td>
<td>Cardinality</td>
<td><p>Response</p>
<p>codes</p></td>
<td>Description</td>
</tr>
<tr class="even">
<td>RevokeAuthorizationRsp</td>
<td>M</td>
<td>1</td>
<td>200 OK</td>
<td>The request was successful.</td>
</tr>
<tr class="odd">
<td>n/a</td>
<td></td>
<td></td>
<td>307 Temporary Redirect</td>
<td><p>Temporary redirection, during authorization revocation. The response shall include a Location header field containing an alternative URI of the resource located in an alternative API exposing function.</p>
<p>Redirection handling is described in clause 5.2.10 of 3GPP TS 29.122 [14].</p></td>
</tr>
<tr class="even">
<td>n/a</td>
<td></td>
<td></td>
<td>308 Permanent Redirect</td>
<td><p>Permanent redirection, during authorization revocation. The response shall include a Location header field containing an alternative URI of the resource located in an alternative API exposing function.</p>
<p>Redirection handling is described in clause 5.2.10 of 3GPP TS 29.122 [14].</p></td>
</tr>
<tr class="odd">
<td colspan="5">NOTE: The mandatory HTTP error status codes for the POST method listed in table 5.2.6-1 of 3GPP TS 29.122 [14] also apply.</td>
</tr>
</tbody>
</table>

Table 9.1.2A.3.2-4: Headers supported by the 307 Response Code on this resource

|          |           |     |             |                                                                                     |
|----------|-----------|-----|-------------|-------------------------------------------------------------------------------------|
| Name     | Data type | P   | Cardinality | Description                                                                         |
| Location | string    | M   | 1           | An alternative URI of the resource located in an alternative API exposing function. |

Table 9.1.2A.3.2-5: Headers supported by the 308 Response Code on this resource

|          |           |     |             |                                                                                     |
|----------|-----------|-----|-------------|-------------------------------------------------------------------------------------|
| Name     | Data type | P   | Cardinality | Description                                                                         |
| Location | string    | M   | 1           | An alternative URI of the resource located in an alternative API exposing function. |

### 9.1.3 Notifications

There are no notifications defined for this API in this release of the specification.

### 9.1.4 Data Model


#### 9.1.4.1 General

This clause specifies the application data model supported by the API. Data types listed in clause 7.2 apply to this API.

Table 9.1.4.1-1 specifies the data types defined specifically for the AEF_Security_API service.

Table 9.1.4.1-1: AEF_Security_API specific Data Types

| Data type              | Section defined  | Description                                        | Applicability |
|------------------------|------------------|----------------------------------------------------|---------------|
| CheckAuthenticationReq | Clause 9.1.4.2.2 | Represents authentication check request data.      |               |
| CheckAuthenticationRsp | Clause 9.1.4.2.3 | Represents authentication check response data.     |               |
| RevokeAuthorizationReq | Clause 9.1.4.2.4 | Represents authorization revocation request data.  |               |
| RevokeAuthorizationRsp | Clause 9.1.4.2.5 | Represents authorization revocation response data. |               |

Table 9.1.4.1-2 specifies data types re-used by the AEF_Security_API service.

Table 9.1.4.1-2: Re-used Data Types

| Data type            | Reference             | Comments                                                                                                                               | Applicability |
|----------------------|-----------------------|----------------------------------------------------------------------------------------------------------------------------------------|---------------|
| SecurityNotification | Clause 8.5.4.2.5      | Represents information about the revoked APIs.                                                                                         |               |
| SupportedFeatures    | 3GPP TS 29.571 \[19\] | Represents the list of supported feature(s) and used to negotiate the applicability of the optional features defined in Table 9.1.6-1. |               |

#### 9.1.4.2 Structured data types


##### 9.1.4.2.1 Introduction

This clause defines the structures to be used in resource representations for the AEF_Security_API.

##### 9.1.4.2.2 Type: CheckAuthenticationReq

Table 9.1.4.2.2-1: Definition of type CheckAuthenticationReq

| Attribute name    | Data type         | P   | Cardinality | Description                                                                                              | Applicability |
|-------------------|-------------------|-----|-------------|----------------------------------------------------------------------------------------------------------|---------------|
| apiInvokerId      | string            | M   | 1           | API invoker ID assigned by the CAPIF core function to the API invoker while on-boarding the API invoker. |               |
| supportedFeatures | SupportedFeatures | M   | 1           | Used to negotiate the supported optional features of the API as described in clause 7.8.                 |               |

##### 9.1.4.2.3 Type: CheckAuthenticationRsp

Table 9.1.4.2.3-1: Definition of type CheckAuthenticationRsp

| Attribute name    | Data type         | P   | Cardinality | Description                                                                              | Applicability |
|-------------------|-------------------|-----|-------------|------------------------------------------------------------------------------------------|---------------|
| supportedFeatures | SupportedFeatures | M   | 1           | Used to negotiate the supported optional features of the API as described in clause 7.8. |               |

##### 9.1.4.2.4 Type: RevokeAuthorizationReq

Table 9.1.4.2.4-1: Definition of type RevokeAuthorizationReq

| Attribute name    | Data type            | P   | Cardinality | Description                                                                              | Applicability |
|-------------------|----------------------|-----|-------------|------------------------------------------------------------------------------------------|---------------|
| revokeInfo        | SecurityNotification | M   | 1           | It contains detailed revocation information.                                             |               |
| supportedFeatures | SupportedFeatures    | M   | 1           | Used to negotiate the supported optional features of the API as described in clause 7.8. |               |

##### 9.1.4.2.5 Type: RevokeAuthorizationRsp

Table 9.1.4.2.5-1: Definition of type RevokeAuthorizationRsp

| Attribute name    | Data type         | P   | Cardinality | Description                                                                              | Applicability |
|-------------------|-------------------|-----|-------------|------------------------------------------------------------------------------------------|---------------|
| supportedFeatures | SupportedFeatures | M   | 1           | Used to negotiate the supported optional features of the API as described in clause 7.8. |               |

#### 9.1.4.3 Simple data types and enumerations

None.

### 9.1.5 Error Handling


#### 9.1.5.1 General

HTTP error handling shall be supported as specified in clause 7.7.

In addition, the requirements in the following clauses shall apply.

#### 9.1.5.2 Protocol Errors

In this Release of the specification, there are no additional protocol errors applicable for the AEF_Security_API.

#### 9.1.5.3 Application Errors

The application errors defined for the AEF_Security_API are listed in table 9.1.5.3-1.

Table 9.1.5.3-1: Application errors

| Application Error | HTTP status code | Description | Applicability |
|-------------------|------------------|-------------|---------------|
|                   |                  |             |               |

### 9.1.6 Feature negotiation

General feature negotiation procedures are defined in clause 7.8.

Table 9.1.6-1: Supported Features

<table>
<colgroup>
<col style="width: 16%" />
<col style="width: 23%" />
<col style="width: 60%" />
</colgroup>
<thead>
<tr class="header">
<th>Feature number</th>
<th>Feature Name</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
<td>CAPIF_Ext1</td>
<td><p>Indicates the support of the enhancements for CAPIF functionality.</p>
<p>Within this feature, the following enhancements are covered:</p>
<p>- Support to communicate the access token during the revocation of the authorization of the API Invoker initiated by CCF.</p></td>
</tr>
</tbody>
</table>
