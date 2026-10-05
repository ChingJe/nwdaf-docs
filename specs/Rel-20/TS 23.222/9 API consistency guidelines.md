---
spec: TS 23.222
version: 20.1.0
release: '20'
clause: 9
title: 9 API consistency guidelines
source_archive: 23222-k10.zip
source_document: 23222-k10.docx
content_origin: 3gpp-source
---

# 9 API consistency guidelines


## 9.1 General

This clause specifies the API consistency guidelines for all northbound APIs utilizing CAPIF architecture. The guidelines are categorized as follows:

\- fundamental API guidelines, applicable to all northbound APIs utilizing CAPIF; and

\- architecture design considerations, applicable to all northbound APIs utilizing CAPIF.

The API guidelines are also applicable for CAPIF APIs specified in the current specification.

## 9.2 Fundamental API Guidelines

The specification of each northbound API utilizing the common API framework should define:

1\. the function of the API;

2\. the resource(s) or endpoints involved;

3\. the list of supported operations and their usage;

4\. the list of input and output parameters along with applicable schemas, as required;

5\. the list of supported response codes;

6\. the behaviour of the network entity exposing the APIs (e.g. the CAPIF core function or the API exposing function) for each supported operation;

7\. the list of applicable data types; and

8\. the list of applicable protocols and data serialization formats.

In order to facilitate the consistency of the northbound APIs utilizing the common API framework it is recommended to adopt the guidelines which define the following:

1\. consistent nomenclature for the operations, data structures and resources/endpoints;

2\. design principles for the use of operations for common tasks; and

3\. a template for the consistent documentation of APIs.

The northbound APIs utilizing the common API framework should support the following properties:

1\. be extensible, such that it is possible to accommodate future requirements, including vendor-specific needs;

NOTE: The extension does not replace any existing function in Northbound APIs.

2\. support access control mechanisms;

3\. support charging, if applicable; and

4\. be backward and forward compatible with different versions of the same API.

The guidelines above are generic with regard to the API architecture. They are valid for network APIs that follow the RESTful architectural style and that expose resources towards the API invoker, as well as for network APIs of other architectures that expose general network endpoints towards the API invoker. A network endpoint represents one end of a communication channel through which the API consumer communicates with the API producer, using messages of a protocol defined by the API architecture. A resource is identified, and the corresponding endpoint is addressed, by a resource identifier (such as a URI).

## 9.3 Architecture design considerations

Northbound APIs utilizing common API framework should adhere to RESTful architecture, whenever possible. Service operations can use custom API operations (RPC-style interaction), when it is seen a better fit for the style of interaction to model, e.g. non-CRUD service operations.

NOTE: The selection of a particular API style is specific to each API implementation, and subject to Stage 3 scope.

The API design:

1\. should have a uniform interface that conveys the resource/data model of the API to its client developers and:

a\. the implementation of the resource(s)/operations involved in the APIs should be hidden from the client, and adequate operations should be designed to operate on the resource(s)/data;

b\. any single API should be atomic;

c\. all resources/operations involved in APIs should be accessible through a common approach, and resources/data should be similarly modified using a consistent approach;

2\. should allow the client (such as the API invoker) and the server (such as the CAPIF core function or the API exposing function) to evolve independently, i.e. the client should not have to be aware of the execution aspects of the APIs on the server;

3\. should be stateless such that each request from the client (such as the API invoker) to the server (such as the CAPIF core function or the API exposing function) contains all of the information necessary for the server to understand the request;

4\. should define the usage of standard operations, such as Create, Read, Update and Delete, consistently along with the applicable response codes;

5\. should allow to label responses as cacheable or non-cacheable, to improve network efficiency by supporting caching in the client (such as the API invoker), if applicable in the API architecture;

6\. should prevent unwanted modification of the resources/data during invocation of APIs; and

7\. should support version control.
