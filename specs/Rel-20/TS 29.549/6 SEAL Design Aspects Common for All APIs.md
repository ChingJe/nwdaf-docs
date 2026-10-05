---
spec: TS 29.549
version: 20.1.0
release: '20'
clause: 6
title: 6 SEAL Design Aspects Common for All APIs
source_archive: 29549-k10.zip
source_document: '29549-k10_0_cover.docx, 29549-k10_1_Main-Body_s00_s06.docx, 29549-k10_2_Main-Body_s07_s09.docx, 29549-k10_3_Annexes_sA_sHistory.docx'
content_origin: 3gpp-source
---

# 6 SEAL Design Aspects Common for All APIs


## 6.1 General

SEAL APIs allow secure access to the capabilities provided by SEAL.

This document specifies the procedures triggered at different functional entities as a result of API invocation requests and event notifications. The stage-2 level requirements and signalling flows are defined in 3GPP TS 23.434 \[2\], 3GPP TS 23.433 \[34\] (for SEALDD), 3GPP TS 23.435 \[41\] (for NSCE), 3GPP TS 23.437 \[45\] (for Spatial Map and Spatial Anchor services) and 3GPP TS 23.438 \[46\] (for Digital Asset management services).

Several design aspects, as mentioned in the following clauses, are specified in 3GPP TS 29.122 \[3\] and referenced by this specification.

## 6.2 Data Types


### 6.2.1 General

This clause defines structured data types, simple data types and enumerations that are applicable to several APIs defined in the present specification and can be referenced from data structures defined in the subsequent clauses.

In addition, data types that are defined in OpenAPI Specification \[15\] can also be referenced from data structures defined in the subsequent clauses.

NOTE: As a convention, data types names in the present specification are with an upper-case letter in the beginning. Parameters are with a lower-case letter in the beginning. As an exception, data types that are also defined in OpenAPI Specification \[15\] can use a lower-case case letter in the beginning for consistency.

Table 6.2.1-1 specifies data types re-used by the SEAL from other specifications, including a reference to their respective specifications and when needed, a short description of their use within the SEAL.

Table 6.2.1-1: Re-used Data Types

<table>
<colgroup>
<col style="width: 21%" />
<col style="width: 23%" />
<col style="width: 54%" />
</colgroup>
<thead>
<tr class="header">
<th>Data type</th>
<th>Reference</th>
<th>Comments</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Uri</td>
<td>3GPP TS 29.122 [3]</td>
<td></td>
</tr>
<tr class="even">
<td>WebsockNotifConfig</td>
<td>3GPP TS 29.122 [3]</td>
<td><p>Following clarifications apply:</p>
<p>- The SCEF is the SEAL Server; and</p>
<p>- The SCS/AS is the service consumer.</p></td>
</tr>
</tbody>
</table>

### 6.2.2 Referenced structured data types

Table 6.2.2-1 lists structured data types defined in this specification referenced by multiple services:

Table 6.2.2-1: Referenced Structured Data Types

| Data type        | Reference          | Description                             |
|------------------|--------------------|-----------------------------------------|
| VALGroupDocument | Clause 7.2.1.4.2.2 | VAL Group document information.         |
| ProfileDoc       | Clause 7.3.1.4.2.2 | VAL User or VAL UE profile information. |

### 6.2.3 Referenced Simple data types and enumerations

Following simple data types defined in Table 6.2.3-1 are applicable to several APIs in this document:

Table 6.2.3-1: Simple data types applicable to several APIs

|           |           |             |
|-----------|-----------|-------------|
| Type name | Reference | Description |
|           |           |             |

## 6.3 Usage of HTTP

For SEAL APIs, support of HTTP/1.1 (IETF RFC 9112 \[5\], IETF RFC 9110 \[6\] and IETF RFC 9111 \[9\]) over TLS is mandatory and support of HTTP/2 (IETF RFC 9113 \[12\]) over TLS is recommended.

A functional entity desiring to use HTTP/2 shall use the HTTP upgrade mechanism to negotiate applicable HTTP version as described in IETF RFC 9113 \[12\].

Usage of HTTP over TLS and the TLS profiles shall be as specified in clause 5.1.1.4 of 3GPP TS 33.434 \[26\].

## 6.4 Content type

The bodies of HTTP request and successful HTTP responses shall be encoded in JSON format (see IETF RFC 8259 \[13\]).

The MIME media type that shall be used within the related Content-Type header field is "application/json", as defined in IETF RFC 8259 \[13\].

The JSON objects defined in clause 5.2.3 of 3GPP TS 29.122 \[3\] for the HTTP PATCH request shall be supported.

NOTE: This release only supports the content type JSON.

## 6.5 URI structure


### 6.5.1 Resource URI structure

All API URIs of SEAL APIs shall be:

**{apiRoot}/\<apiName\>/\<apiVersion\>**

"apiRoot" is configured by means outside the scope of the present document. It includes the scheme ("https"), host and optional port, and an optional prefix string. "apiName" and "apiVersion" shall be set dependent on the API, as defined in the corresponding clauses below.

All resource URIs in the clauses below are defined relative to the above root API URI.

NOTE 1: The "apiVersion" will only be increased if the new API version contains backward incompatible changes. Otherwise, the supported feature mechanism defined in clause 6.8 can be used to negotiate extensions.

NOTE 2: A different root structure can be used when the resource URI is preconfigured in the API invoking entity.

The root structure may be followed by "apiSpecificSuffixes" that are dependent on the API and are defined separately for each API as resource URI where they apply:

**{apiRoot}/\<apiName\>/\<apiVersion\>/\<apiSpecificSuffixes\>**

### 6.5.2 Custom operations URI structure

The custom operation definition is in Annex C of 3GPP TS 29.501 \[14\].

The URI of a custom operation which is associated with a resource shall have the following structure:

**{apiRoot}/\<apiName\>/\<apiVersion\>/\<apiSpecificResourceUriPart\>/\<custOpName\>**

Custom operations can also be associated with the service instead of a resource. The URI of a custom operation which is not associated with a resource shall have the following structure:

**{apiRoot}/\<apiName\>/\<apiVersion\>/\<custOpName\>**

In the above URI structures, "apiRoot", "apiName", "apiVersion" and "apiSpecificResourceUriPart" are as defined in clause 6.5.1 and "custOpName" represents the name of the custom operation as defined in clause 5.1.3.2 of 3GPP TS 29.501 \[14\]

## 6.6 Notifications

The functional entities

\- shall support the delivery of notifications using a separate HTTP connection towards an address (see clause 5.2.5.2 of 3GPP TS 29.122 \[3\]);

\- may support testing delivery of notifications (see clause 5.2.5.3 of 3GPP TS 29.122 \[3\]); and

\- may support the delivery of notification using WebSocket protocol (see IETF RFC 6455 \[4\] and clause 5.2.5.4 of 3GPP TS 29.122 \[3\]),

as described in 3GPP TS 29.122 \[3\], with the following clarifications:

\- the SCEF is the SEAL server; and

\- the SCS/AS is the Subscriber.

## 6.7 Error Handling

Response bodies and error handling described in clause 5.2.6 of 3GPP TS 29.122 \[3\] are applicable to all APIs in the present specification unless specified otherwise, with the following clarifications:

\- the SCEF is the SEAL server; and

\- the SCS/AS is the functional entity invoking an API.

## 6.8 Feature negotiation

The functional entity invoking an API (i.e., the service consumer) and the SEAL server use feature negotiation procedures defined in 3GPP TS 29.122 \[3\] to negotiate the supported features, with the following clarifications:

\- description of the SCEF applies to the SEAL server; and

\- description of the SCS/AS applies to the functional entity invoking an API.

## 6.9 HTTP headers

The HTTP headers and the HTTP custom headers described in 3GPP TS 29.122 \[3\] are applicable to all APIs in this document.

## 6.10 Conventions for Open API specification files

The conventions for Open API specification files as specified in clause 5.2.9 of 3GPP TS 29.122 \[3\] shall be applicable for all APIs in this document.
