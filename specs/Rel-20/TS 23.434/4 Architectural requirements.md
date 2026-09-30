---
spec: TS 23.434
version: 20.1.0
release: '20'
clause: 4
title: 4 Architectural requirements
source_archive: 23434-k10.zip
source_document: 23434-k10.docx
content_origin: 3gpp-source
---

# 4 Architectural requirements


## 4.1 General


### 4.1.1 Description

This subclause specifies the general requirements for SEAL.

### 4.1.2 Requirements

\[AR-4.1.2-a\] The SEAL shall support applications from one or more verticals.

\[AR-4.1.2-b\] The SEAL shall support multiple applications from the same vertical.

\[AR-4.1.2-c\] The SEAL shall offer SEAL services as APIs to the vertical applications.

\[AR-4.1.2-d\] The SEAL shall support notification mechanism for SEAL service events.

\[AR-4.1.2-e\] The API interactions between the vertical application server(s) and SEAL server(s) shall conform to CAPIF as specified in 3GPP TS 23.222 \[8\].

\[AR-4.1.2-f\] The SEAL server(s) shall provide a service API compliant with CAPIF as specified in 3GPP TS 23.222 \[8\].

\[AR-4.1.2-g\] The SEAL shall leverage the satellite connectivity.

\[AR-4.1.2-h\] The SEAL shall support discontinuous coverage of satellite connectivity.

\[AR-4.1.2-i\] The SEAL shall support managing S&F operations regarding satellite connectivity in EPS.

## 4.2 Deployment models


### 4.2.1 Description

This subclause specifies the requirements for various deployment models.

### 4.2.2 Requirements

\[AR-4.2.2-a\] The SEAL shall support deployments in which SEAL services are deployed only within PLMN network.

\[AR-4.2.2-b\] The SEAL shall support deployments in which SEAL services are deployed only outside of PLMN network.

\[AR-4.2.2-c\] The SEAL shall support deployments in which SEAL services are deployed both within and outside the PLMN domain at the same time.

\[AR-4.2.2-d\] The SEAL shall support SEAL capabilities for centralized deployment of vertical applications.

\[AR-4.2.2-e\] The SEAL shall support SEAL capabilities for distributed deployment of vertical applications.

## 4.3 Location management


### 4.3.1 Description

This subclause specifies the requirements for location management service.

### 4.3.2 On-network functional model requirements

\[AR-4.3.2-a\] The SEAL shall enable sharing location data between client and server for vertical applications usage.

\[AR-4.3.2-b\] The SEAL shall support different granularity of location data, as required by the vertical application.

\[AR-4.3.2-c\] The SEAL shall support requests for on-demand location reporting.

\[AR-4.3.2-d\] The SEAL shall support client location reporting based on triggers.

\[AR-4.3.2-e\] The SEAL shall enable vertical applications to receive updates to the location information.

\[AR-4.3.2-f\] The SEAL shall enable sharing the network location information obtained from the 3GPP network systems to the vertical applications.

\[AR-4.3.2-g\] The SEAL shall provide a mechanism to enable vertical applications to obtain a list of UE(s), and the location information of each UE, in the proximity to a designated/requested location.

\[AR-4.3.2-h\] The SEAL shall support the Geofencing.

\[AR-4.3.2-i\] The SEAL shall support the location history store and retrieve.

\[AR-4.3.2-j\] The SEAL shall support the UE location reporting with more information (e.g. velocity) to the vertical application.

\[AR-4.3.2-k\] The SEAL shall support the stored UE location information reusing.

\[AR-4.3.2-l\] The SEAL shall support the adaptive location reporting based on the UE moving trend.

\[AR-4.3.2-m\] The SEAL shall enable optimizing the location service operations when the multiple UEs are sharing the same location.

### 4.3.3 Off-network functional model requirements

\[AR-4.3.3-a\] The SEAL shall support on-demand location reporting and event-triggered location reporting within PC5 communication.

## 4.4 Group management


### 4.4.1 Description

This subclause specifies the requirements for group management service.

### 4.4.2 Requirements

\[AR-4.4.2-a\] The SEAL shall enable group management operations (e.g. CRUDN) by the authorized users or VAL server.

\[AR-4.4.2-b\] The SEAL shall enable creation of group to be used by one or more vertical applications within the same VAL system.

\[AR-4.4.2-c\] The SEAL shall enable two or more groups to be merged (temporarily or permanently) into a single group by the authorized users or VAL server wherein all the group members of the constituent groups are designated as members of the merged group.

## 4.5 Configuration management


### 4.5.1 Description

This subclause specifies the requirements for configuration management service.

### 4.5.2 Requirements

\[AR-4.5.2-a\] The SEAL shall enable configuring service specific configuration data applicable to vertical applications.

\[AR-4.5.2-b\] The SEAL shall support configuring data applicable to different vertical applications.

## 4.6 Key management


### 4.6.1 Description

This subclause specifies the requirements for key management service.

### 4.6.2 Requirements

\[AR-4.6.2-a\] The SEAL shall support secure distribution of security related information (e.g. encryption keys).

\[AR-4.6.2-b\] The SEAL shall support all communications in SEAL ecosystem to be secured.

## 4.7 Identity management


### 4.7.1 Description

This subclause specifies the requirements for identity management service.

### 4.7.2 Requirements

\[AR-4.7.2-a\] The SEAL shall enable the access to SEAL services from the vertical application layer entities to be authorized.

## 4.8 Network resource management


### 4.8.1 Description

This subclause specifies the requirements for network resource management service.

### 4.8.2 Requirements

\[AR-4.8.2-a\] The SEAL shall enable support for unicast bearer establishment and modification to support service KPIs for VAL communications.

\[AR-4.8.2-b\] The SEAL shall enable support for multicast bearer establishment and modification to support service KPIs for VAL communications.

\[AR-4.8.2-c\] The SEAL shall support announcement of multicast bearers to the UEs.

\[AR-4.8.2-d\] The SEAL shall support switching of bearers between unicast and multicast.

\[AR-4.8.2-e\] The SEAL shall support multicast bearer quality detection.

\[AR-4.8.2-f\] The SEAL shall enable support for unicast PDU session establishment and modification to support service KPIs for VAL communications.

\[AR-4.8.2-g\] The SEAL shall enable support for MBS session (multicast or broadcast type) establishment and modification to support service KPIs for VAL communications.

\[AR-4.8.2-h\] The SEAL shall support announcement of MBS session (multicast or broadcast type) to the UEs.

\[AR-4.8.2-i\] The SEAL shall support switching of between unicast PDU session and MBS session (multicast or broadcast type).

\[AR-4.8.2-j\] The SEAL shall support MBS session (multicast or broadcast type) quality detection.
