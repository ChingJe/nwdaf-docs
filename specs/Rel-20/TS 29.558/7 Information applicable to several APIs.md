---
spec: TS 29.558
version: 20.0.0
release: '20'
clause: 7
title: 7 Information applicable to several APIs
source_archive: 29558-k00.zip
source_document: 29558-k00.docx
content_origin: 3gpp-source
---

# 7 Information applicable to several APIs


## 7.1 General

The EDGEAPP APIs as specified in this document allow secure access to the capabilities provided by the functional entities, Edge Enabler Server and Edge Configuration Server.

This document specifies the procedures triggered at different functional entities as a result of API invocation requests and event notifications. The stage-2 level requirements and signalling flows are defined in 3GPP TS 23.558 \[2\].

Several design aspects, as mentioned in the following clauses, are specified in 3GPP TS 29.122 \[6\] and referenced by this specification.

## 7.2 Data Types


### 7.2.1 General

This clause defines structured data types, simple data types and enumerations that are applicable to several APIs defined in the present specification and can be referenced from data structures defined in the subsequent clauses.

In addition, data types that are defined in OpenAPI 3.0.0 Specification \[3\] can also be referenced from data structures defined in the subsequent clauses.

NOTE: As a convention, data type's names in the present specification are with an upper-case letter in the beginning. Parameters are with a lower-case letter in the beginning. As an exception, data types that are also defined in OpenAPI 3.0.0 Specification \[3\] can use a lower-case case letter in the beginning for consistency.

Table 7.2.1-1 specifies data types re-used by APIs in this specification from other specifications, including a reference to their respective specifications and when needed, a short description of their use within the APIs of this specification.

Table 7.2.1-1: Re-used Data Types

| Data type | Reference            | Comments |
|-----------|----------------------|----------|
| Uri       | 3GPP TS 29.122 \[6\] |          |

### 7.2.2 Referenced structured data types

Table 7.2.2-1 lists structured data types defined in this specification referenced by multiple services:

Table 7.2.2-1: Referenced Structured Data Types

| Data type | Reference | Description |
|-----------|-----------|-------------|
|           |           |             |

### 7.2.3 Referenced simple data types and enumerations

Following simple data types defined in Table 7.2.3-1 are applicable to several APIs in this document:

Table 7.2.3-1: Simple data types applicable to several APIs

|           |           |             |
|-----------|-----------|-------------|
| Type name | Reference | Description |
|           |           |             |

## 7.3 Usage of HTTP

For all the APIs in this document, the usage of HTTP shall be as specified in clause 5.2.2 of 3GPP TS 29.122 \[6\], with the clarification that SCA/AS is the functional entity invoking an EDGEAPP API.

For all the APIs, to provide the interface protection over EDGE-3/6/9/10 interfaces, HTTP over TLS shall be used as specified in 3GPP TS 33.558 \[20\].

## 7.4 Content type

The content type for the HTTP messages shall be as specified in clause 5.2.3 of 3GPP TS 29.122 \[6\].

## 7.5 URI structure


### 7.5.1 Resource URI structure

The resource URI structure of all the APIs specified in this document shall be as specified in clause 5.2.4 of 3GPP TS 29.122 \[6\].

### 7.5.2 Custom operations URI structure

The custom operation definition is in Annex C of 3GPP TS 29.501 \[5\].

The URI of a custom operation which is associated with a resource shall have the following structure:

**{apiRoot}/\<apiName\>/\<apiVersion\>/\<apiSpecificResourceUriPart\>/\<custOpName\>**

Custom operations can also be associated with the service instead of a resource. The URI of a custom operation which is not associated with a resource shall have the following structure:

**{apiRoot}/\<apiName\>/\<apiVersion\>/\<custOpName\>**

In the above URI structures, "apiRoot", "apiName", "apiVersion" and "apiSpecificResourceUriPart" are as defined in clause 7.5.1 and "custOpName" represents the name of the custom operation as defined in clause 5.1.3.2 of 3GPP TS 29.501 \[5\].

## 7.6 Notifications

The functional entities

\- shall support the delivery of notifications using a separate HTTP connection towards an address;

\- may support testing delivery of notifications; and

\- may support the delivery of notification using WebSocket protocol (see IETF RFC 6455 \[7\]),

as described in 3GPP TS 29.122 \[6\], with the following clarifications:

\- the SCEF is EES for Eees APIs; and

\- the SCS/AS is the Subscriber entity invoking an EDGEAPP API.

## 7.7 Error handling

Response bodies for error handling, as described in clause 5.2.6 of 3GPP TS 29.122 \[6\], are applicable to all APIs in the present specification unless specified otherwise, with the following clarifications:

\- the SCEF is EES for Eees APIs or ECS for Eecs APIs ; and

\- the SCS/AS is the functional entity invoking an EDGEAPP API

## 7.8 Feature negotiation

The functional entity invoking an API (i.e. the EAS server) and the EDGEAPP server use feature negotiation procedures defined in clause 5.2.7 of 3GPP TS 29.122 \[6\] to negotiate the supported features, with the following clarifications:

\- description of the SCEF applies to the EES for Eees APIs or ECS for Eecs APIs; and

\- description of the SCS/AS applies to the functional entity invoking an EDGEAPP API.

## 7.9 HTTP headers

The HTTP headers described in 3GPP TS 29.122 \[6\] are applicable to all APIs in this document.

## 7.10 Conventions for Open API specification files

The conventions for Open API specification files as specified in clause 5.2.9 of 3GPP TS 29.122 \[6\] shall be applicable for all APIs in this document.
