---
spec: TS 23.222
version: 20.1.0
release: '20'
clause: 3
title: 3 Definitions and abbreviations
source_archive: 23222-k10.zip
source_document: 23222-k10.docx
content_origin: 3gpp-source
---

# 3 Definitions and abbreviations


## 3.1 Definitions

For the purposes of the present document, the terms and definitions given in 3GPP TR 21.905 \[1\] and the following apply. A term defined in the present document takes precedence over the definition of the same term, if any, in 3GPP TR 21.905 \[1\].

**API:** The means by which an API invoker can access the service.

**API invoker:** The entity which invokes the CAPIF or service APIs.

**API invoker profile:** The set of information associated to an API invoker that allows that API invoker to utilize CAPIF APIs and service APIs.

**API exposing function:** The entity which provides the service communication entry point for the service APIs.

**API exposing function location:** The location information (e.g. civic address, GPS coordinates, data center ID) where the API exposing function providing the service API is located.

**API category:** A name used to group service APIs by the service domain in which they are used (e.g., IoT, V2X). The CAPIF core function maintains a list of API category names.

**API management function:** The entity which enables the API provider to perform administration of the service APIs.

**API publishing function:** The entity that enables the API provider to publish the Service APIs information in order to enable the discovery of APIs by the API invoker.

**Authorization function:** The entity which issues access tokens to the API invokers after successfully authenticating the resource owner and obtaining authorization.

**CAPIF administrator:** An authorized user with special permissions for CAPIF operations.

**Common API framework:** A framework comprising common API aspects that are required to support service APIs.

**Designated CAPIF core function:** The CAPIF core function which is configured as the serving CAPIF core function for interconnection.

**Network Slice Info:** Network slice information of service API which include a list of network slice related identifiers, as described in clause 8.4 of 3GPP TS 23.435 \[13\]).

**Northbound API:** A service API exposed to higher-layer API invokers.

**Onboarding:** One time registration process that enables the API invoker to subsequently access the CAPIF and the service APIs.

**Resource:** The object or component of the API on which the operations are acted upon.

**Resource owner:** A UE user or an MNO subscriber capable of granting access to a protected resource related to the invoked API via resource owner function.

**Resource owner function:** The entity that enables the authorization for resource access and managing and revoking authorization for resource access.

**Resource owner-aware northbound API access:** An API invocation scenario where the API invoker needs an authorization from the resource owner.

**Service API:** The interface through which a component of the system exposes its services to API invokers by abstracting the services from the underlying mechanisms.

**Serving Area Information:** The location information for which the service APIs are being offered to.

**CAPIF provider domain:** A domain that contains an instance of CAPIF core function and may contain API provider domains and API invokers. The CAPIF provider could be a PLMN, SNPN or 3rd party. Throughout this document, PLMN trust domain is often used as the typical deployment of a CAPIF provider domain however SNPN trust domain or 3rd party trust domain are applicable as well.

**PLMN trust domain:** The entities protected by adequate security and controlled by the PLMN operator or a trusted 3<sup>rd</sup> party of the PLMN.

**SNPN trust domain:** The entities protected by adequate security and controlled by the SNPN operator or a trusted 3<sup>rd</sup> party of the SNPN. **3<sup>rd</sup> party trust domain:** The entities protected by adequate security and controlled by the 3<sup>rd</sup> party.

**Application management client:** The application developers utilize the CAPIF APIs using an application management client as an API invoker to obtain the service APIs information to implement the application program. Such application programs are hosted on cloud, edge or on a UE.

**Hosted applications:** The hosted applications (which are programmed to utilize the service APIs) as an API invoker invoke the CAPIF APIs and service APIs as per the application business logic.

**Channel Aggregator Platform:** The Channel Aggregator aggregates the APIs from one or more southbound CAPIF providers with the intention to re-expose such APIs or expose value-added APIs developed using the APIs from the southbound CAPIF providers to the northbound side Application management client or Hosted applications. The Channel Aggregator Platform as API invoker invokes the CAPIF APIs and service APIs as per its business logic.

For the purposes of the present document, the following terms and definitions given in 3GPP TS 32.240 \[6\] apply:

**Offline charging**

**Online charging**

## 3.2 Abbreviations

For the purposes of the present document, the abbreviations given in 3GPP TR 21.905 \[1\] and the following apply. An abbreviation defined in the present document takes precedence over the definition of the same abbreviation, if any, in 3GPP TR 21.905 \[1\].

5GS 5G System

AEF API Exposing Function

AF Application Function

AMF API Management Function

APF API Publishing Function

API Application Program Interface

AS Application Server

BM-SC Broadcast Multicast Service Centre

CAPIF Common API Framework

CRUD Create, Read, Update, Delete

CCF CAPIF Core Function

DDoS Distributed Denial of Service

E-UTRA Evolved Universal Terrestrial Radio Access

EPS Evolved Packet System

ETSI European Telecommunications Standards Institute

GS Group Specification

IP Internet Protocol

MBMS Multimedia Broadcast and Multicast Service

MEC Multi-access Edge Computing

NEF Network Exposure Function

NGSI Next Generation Service Interfaces

NR New Radio

OMA Open Mobile Alliance

OAM Operations, Administration and Maintenance

OWSER OMA Web Services

PC Protocol Converter

PLMN Public Land Mobile Network

REST REpresentational State Transfer

RNAA Resource owner-aware Northbound API Access

RO Resource Owner

ROF Resource Owner Function

RPC Remote Procedure Call

SCEF Service Capability Exposure Function

SCS Service Capability Server

SNPN Stand-alone Non-Public Network

UDDI Universal Description, Discovery and Integration

URI Uniform Resource Identifier

WSDL Web Services Description Language
