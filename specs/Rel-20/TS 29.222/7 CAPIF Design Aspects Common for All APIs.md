---
spec: TS 29.222
version: 20.1.0
release: '20'
clause: 7
title: 7 CAPIF Design Aspects Common for All APIs
source_archive: 29222-k10.zip
source_document: 29222-k10.docx
content_origin: 3gpp-source
---

# 7 CAPIF Design Aspects Common for All APIs


## 7.1 General

CAPIF APIs are RESTful APIs that allow secure access to the capabilities provided by CAPIF.

This document specifies the procedures triggered at different functional entities as a result of API invocation requests and event notifications. The stage-2 level requirements and signalling flows are defined in 3GPP TS 23.222 \[2\].

Several design aspects, as mentioned in the following clauses, are specified in 3GPP TS 29.122 \[14\] and referenced by this specification.

The common API design aspects defined in the clauses under clause 5.2 of 3GPP TS 29.122 \[14\] that are not defined in the following clauses (e.g., clauses 5.2.10, 5.2.11, 5.2.12 of 3GPP TS 29.122 \[14\]) shall also apply to the CAPIF APIs defined in this specification, with the following differences:

\- the CCF/AEF plays the role of the SCEF;

\- the service consumer (e.g., API Invoker, AEF, APF, AMF, CCF) plays the role of the SCS/AS; and

\- the provisions related to the T8 APIs shall apply for the CAPIF APIs.

## 7.2 Data Types


### 7.2.1 General

This clause defines the general guidelines for the structured data types, simple data types and enumerations defined in the present specification and the ones that are referenced from data structures defined in the subsequent clauses.

In addition, data types that are defined in OpenAPI Specification \[3\] can also be referenced from data structures defined in the subsequent clauses.

NOTE: As a convention, data types in the present specification follow the UpperCamel case convention. Attributes of structured data types follow the lowerCamel case convention. Enumerations follow the UPPER_WITH_UNDERSCORE case convention. As an exception, data types that are also defined in OpenAPI Specification \[3\] can use a lower-case case letter in the beginning for consistency.

### 7.2.2 Void

### 7.2.3 Void

## 7.3 Usage of HTTP

For CAPIF APIs, the support of HTTP/1.1 (IETF RFC 9112 \[4\], IETF RFC 9110 \[5\], and IETF RFC 9111 \[8\]) over TLS is mandatory and the support of HTTP/2 (IETF RFC 9113 \[10\]) over TLS is recommended. TLS shall be used as specified in 3GPP TS 33.122 \[16\].

A functional entity desiring to use HTTP/2 shall use the HTTP upgrade mechanism to negotiate applicable HTTP version as described in IETF RFC 9113 \[10\].

## 7.4 Content type

The provisions of clause 5.2.3 of 3GPP TS 29.122 \[14\] shall apply to the CAPIF APIs defined in this specification.

## 7.5 URI structure


### 7.5.1 Resource URI structure

The provisions of clause 5.2.4.1 of 3GPP TS 29.122 \[14\] shall apply to the CAPIF APIs defined in this specification.

### 7.5.2 Custom operations URI structure

The provisions of clause 5.2.4.2 of 3GPP TS 29.122 \[14\] shall apply to the CAPIF APIs defined in this specification.

## 7.6 Notifications

The functional entities

\- shall support the delivery of notifications using a separate HTTP connection towards an address;

\- may support testing delivery of notifications; and

\- may support the delivery of notification using WebSocket protocol (see IETF RFC 6455 \[13\]),

as described in clause 5.2.5 of 3GPP TS 29.122 \[14\], with the following clarifications:

\- the CCF/AEF plays the role of the SCEF; and

\- the service consumer (e.g., API Invoker, AEF, APF, AMF, CCF) plays the role of the SCS/AS.

## 7.7 Error handling

HTTP error handling described in clause 5.2.6 of 3GPP TS 29.122 \[14\] is applicable to the CAPIF APIs defined in the present specification unless specified otherwise, with the following clarifications:

\- the CCF/AEF plays the role of the SCEF; and

\- the service consumer (e.g., API Invoker, AEF, APF, AMF, CCF) plays the role of the SCS/AS.

## 7.8 Feature negotiation

The service consumer or functional entity invoking an API (e.g., API invoker, AEF, the APF, AMF, CCF) and the CCF shall support the feature negotiation procedures defined in clause 5.2.7 of 3GPP TS 29.122 \[14\] to negotiate the supported features, with the following clarifications:

\- the CCF/AEF plays the role of the SCEF; and

\- the service consumer (e.g., API Invoker, AEF, APF, AMF, CCF) plays the role of the SCS/AS.

## 7.9 HTTP custom headers

The HTTP custom headers defined in clause 5.2.8 of 3GPP TS 29.122 \[14\] shall apply to the CAPIF APIs defined in this specification.

## 7.10 Conventions for Open API specification files

The conventions for Open API specification files as specified in clause 5.2.9 of 3GPP TS 29.122 \[14\] shall be applicable for the CAPIF APIs defined in this specifications.

## 7.11 CAPIF vendor-specific extensions

The data model of any the CAPIF API shall be extensible with vendor-specific data as specified in clause 5.2.13.2 of 3GPP TS 29.122 \[14\].

The query parameters used in GET requests in the CAPIF APIs shall be extensible with vendor-specific query parameters as specified in clause 5.2.13.3 of 3GPP TS 29.122 \[14\].
