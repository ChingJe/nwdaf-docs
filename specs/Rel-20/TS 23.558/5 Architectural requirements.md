---
spec: TS 23.558
version: 20.3.0
release: '20'
clause: 5
title: 5 Architectural requirements
source_archive: 23558-k30.zip
source_document: 23558-k30.docx
content_origin: 3gpp-source
---

# 5 Architectural requirements


## 5.1 General

This clause specifies architectural requirements for enabling edge applications in different functional aspects.

## 5.2 Architectural requirements


### 5.2.1 General requirements


#### 5.2.1.1 General

This clause specifies general requirements for the architecture.

#### 5.2.1.2 Requirements

\[AR-5.2.1.2-a\] The application layer architecture shall support deployment of EAS(s) and AC(s) with or without modifications compared to their existing deployments.

\[AR-5.2.1.2-b\] The application layer architecture shall support different deployment models in conjunction with an operator's 3GPP network.

\[AR-5.2.1.2-c\] The application layer architecture shall be compatible with the 3GPP network system.

### 5.2.2 Edge configuration data


#### 5.2.2.1 General

This clause specifies the requirements for edge configuration data.

#### 5.2.2.2 Requirements

\[AR-5.2.2.2-a\] The application layer architecture shall provide mechanisms to provide configuration parameters to an authorized EEC to access the EES(s).

### 5.2.3 Registration


#### 5.2.3.1 General

This clause specifies the requirements for EEC, EAS and EES registration.

#### 5.2.3.2 EEC registration

\[AR-5.2.3.2-a\] The application layer architecture shall provide mechanisms for an EEC to register onto the EES.

\[AR-5.2.3.2-b\] The application layer architecture shall provide mechanisms for an EEC to de-register from the EES.

\[AR-5.2.3.2-c\] The application layer architecture shall provide mechanisms for the EES to detect an abnormal termination of an EEC registration.

#### 5.2.3.3 EAS registration

\[AR-5.2.3.3-a\] The application layer architecture shall provide mechanisms for an EAS to register to the EES.

\[AR-5.2.3.3-b\] The application layer architecture shall support EAS exposing its availability, which varies with time, location, etc.

\[AR-5.2.3.3-c\] The application layer architecture shall provide mechanisms so that the EASs are uniquely identifiable.

\[AR-5.2.3.3-d\] The application layer architecture shall provide mechanisms for an EAS to de-register from the EES.

\[AR-5.2.3.3-e\] The application layer architecture shall provide mechanisms for the EES to detect an abnormal termination of an EAS registration.

#### 5.2.3.4 EES registration

\[AR-5.2.3.4-a\] The application layer architecture shall provide mechanisms for an EES to register onto the ECS.

\[AR-5.2.3.4-b\] The application layer architecture shall support EES to publish EAS information on the ECS.

\[AR-5.2.3.4-c\] The application layer architecture shall support EES to update the published EAS information on the ECS.

\[AR-5.2.3.4-d\] The application layer architecture shall provide mechanisms for an EES to de-register from the ECS.

\[AR-5.2.3.4-e\] The application layer architecture shall provide mechanisms for the ECS to detect an abnormal termination of an EES registration.

### 5.2.4 EAS discovery


#### 5.2.4.1 General

This clause specifies the requirements for EAS discovery.

#### 5.2.4.2 Requirements

\[AR-5.2.4.2-a\] The application layer architecture shall provide mechanisms for an EEC to discover available EASs.

\[AR-5.2.4.2-b\] The application layer architecture shall provide relevant configuration information of the EASs to the EEC, in order to enable communication between ACs and the EASs.

### 5.2.5 Capability exposure to EASs


#### 5.2.5.1 General

This clause specifies the requirements for capability exposure to EAS.

#### 5.2.5.2 Requirements

\[AR-5.2.5.2-a\] The application layer architecture shall support exposure of 3GPP network's capabilities to the EASs.

\[AR-5.2.5.2-b\] The application layer architecture shall support exposure of EES's capabilities to the EASs.

\[AR-5.2.5.2-c\] The application layer architecture shall support exposure of EAS's capabilities to the other EASs.

### 5.2.6 Security


#### 5.2.6.1 General

This clause specifies the security requirements.

#### 5.2.6.2 Requirements

\[AR-5.2.6.2-a\] The application layer architecture shall provide mechanisms for the Edge Computing Service Provider to authorize the usage of Edge Computing services by the EEC.

\[AR-5.2.6.2-b\] The application layer architecture shall provide mechanisms for the Edge Computing Service Provider to authorize the usage of Edge Computing services by the EASs.

\[AR-5.2.6.2-c\] Communication between the functional entities of the application layer architecture shall be protected.

\[AR-5.2.6.2-d\] The authentication and authorization for the use of Edge Computing services shall support the deployment where the functional entities providing the Edge Computing services are in the same trust domain as the 3GPP system, different trust domains or both.

\[AR-5.2.6.2-e\] The application layer architecture shall support the use of either 3GPP credentials or application specific credentials or both for different deployment needs, for the communication between the UE and the functional entities providing the Edge Computing service.

\[AR-5.2.6.2-f\] The application layer architecture shall support mutual authentication and authorization check between clients and servers or servers and servers that interact.

\[AR-5.2.6.2-g\] The application layer architecture shall support EASs to obtain user's authorization in order to access to user's sensitive information (e.g. user's location).

\[AR-5.2.6.2-h\] The application layer architecture shall provide mechanisms to support privacy protection of the user.

NOTE 1: Security and privacy related procedures are specified in 3GPP TS 33.558 \[23\].

NOTE 2: EAS obtained user consent requirement in \[AR-5.2.6.2-g\] is not supported in the current release.

### 5.2.7 Subscription service


#### 5.2.7.1 General

This clause specifies the requirements for a subscription service.

#### 5.2.7.2 Requirements

\[AR-5.2.7.2-a\] The application layer architecture shall provide subscription and notification mechanisms enabling an EEC to receive changes in dynamic information of EASs from an EES.

\[AR-5.2.7.2-b\] The application layer architecture shall provide subscription and notification mechanisms enabling an EEC to receive changes in availability of EASs from an EES.

\[AR-5.2.7.2-c\] The application layer architecture shall provide subscription and notification mechanisms enabling an EEC to receive changes in EES's information and availability status (e.g. EES endpoint change or EES is about to become unavailable due to overload, maintenance window, etc.) from an ECS.

\[AR-5.2.7.2-d\] The application layer architecture shall provide subscription and notification mechanisms enabling an EAS to receive information about relevant changes in AC(s) information of a UE.

\[AR-5.2.7.2-e\] The application layer architecture shall provide subscription and notification mechanisms enabling an EAS to receive information about relevant reports in UE location.

\[AR-5.2.7.2-f\] The application layer architecture shall provide subscription and notification mechanisms enabling to receive changes in service continuity.

### 5.2.8 Traffic management


#### 5.2.8.1 General

This clause specifies the requirements for traffic management.

#### 5.2.8.2 Requirements

\[AR-5.2.8.2-a\] The application layer architecture shall support AF influence on traffic routing over N6 interface.

\[AR-5.2.8.2-b\] The application layer architecture should be able to monitor the network status (e.g. traffic volume, throughput, network load, etc.) that may impact the application KPIs.

### 5.2.9 Lifecycle management


#### 5.2.9.1 General

This clause specifies the requirements for lifecycle management.

#### 5.2.9.2 Requirements

\[A.5.2.9.2-a\] The application layer architecture shall support interactions with a lifecycle management system.

### 5.2.10 Edge application KPIs


#### 5.2.10.1 General

This clause specifies the requirements for edge application KPIs.

#### 5.2.10.2 Requirements

\[AR-5.2.10.2-a\] The application layer architecture shall provide mechanisms for the EAS to publish its KPIs or application level requirements when available (e.g. upon new application on-boarding).

\[AR-5.2.10.2-b\] The application layer architecture shall provide mechanisms for the EAS to update its KPIs or application level requirements.

### 5.2.11 Service continuity


#### 5.2.11.1 General

This clause specifies the requirements for service continuity.

#### 5.2.11.2 Requirements

\[AR-5.2.11.2-a\] The application layer architecture shall provide mechanisms to support service continuity such that the Application Context with a S-EAS is transferred to a T-EAS.

\[AR-5.2.11.2-b\] The application layer architecture shall provide mechanisms to support service continuity such that the Application Context with an EAS is transferred to a CAS.

\[AR-5.2.11.2-c\] The application layer architecture shall provide mechanisms to support service continuity such that the Application Context with a CAS is transferred to an EAS.
