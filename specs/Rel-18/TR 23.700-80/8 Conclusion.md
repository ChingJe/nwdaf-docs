---
spec: TR 23.700-80
version: 18.0.0
release: '18'
clause: 8
title: 8 Conclusion
source_archive: 23700-80-i00.zip
source_document: 23700-80-i00.docx
content_origin: 3gpp-source
---

# 8 Conclusion


## 8.0 General Architecture Principles

The following general architecture principles are agreed as conclusions:

\- Principle \#1: Application AI/ML decisions and their internal operation logic reside at the AF and UE application client and is out of scope of 3GPP. If the AF wants to know the Application layer AI/ML capability in UE application client, this can be done via interaction between the AF and the UE application client, which is out of the 3GPP scope.

\- Principle \#2: The AF does not include the application AI/ML operation type in the request sent to 5GC to request assistance for the application AI/ML operation.

\- Principle \#3: UE capability indication to support specific Application's AI/ML operation type is not required.

\- Principle \#4: 5GC shall not waste the network resources to provide assistance for UEs which do not have proper subscription, e.g., mobility restriction, DNN, S-NSSAI, etc.

\- Principle \#5: Based on application logic, it is the application decision whether to request assistance from 5GC.

\- Principle \#6: The 5GC could provide assistance to AF for UE selection

\- Principle \#7: The support of 5GS to AI/ML-based services should strive to re-use existing NFs and procedures with necessary enhancements in order to fulfil the service requirement.

NOTE 1: More clarifications on the system parameters for the Application AI/ML services are needed as part of the specific KI discussions.

\- Principle \#8: The architecture to support AI/ML-based services should be backward compatible with related 5GS features. It should also be designed to enable future extensions.

\- Principle \#9: Although the network does not need to know about internals of AI/ML services, it needs to be able to distinguish AI/ML related traffic from rest of the user services related traffic. In that sense the network is not completely agnostic of AI/ML traffic.

NOTE 2: While AI/ML traffic is considered as a new traffic category, it is assumed that existing detection mechanisms will be used to distinguish AI/ML related traffic from the rest of the user services related traffic.

\- Principle \#10: The architecture/procedure to support AI/ML-based services and applications should avoid unnecessary signalling in the mobile network and/or between 5GC and AF. The principle should also apply for 5GS assistance to the AF monitoring of the application AI/ML operation utilization of network resources.

\- Principle \#11: Mobile network security and privacy (both UEs and mobile networks) are important and solutions should avoid sensitive information exposure from the mobile networks.

NOTE 3: This principle requires involvement from SA WG3.

NOTE 4: The user consent checking procedure regarding for this Study Item will be sync up with SA3’s conclusion during normative work.

## 8.1 Key Issue \#1: Monitoring of network resource utilization for support of Application AI/ML operations

It is proposed that the architecture principles and specific mechanisms below are considered for the normative work on monitoring of network resource utilization for support of application AI/ML operations:

\- NWDAF-based procedures and data analytics as described in TS 23.288 \[6\] for the Observed Service Experience, User Data Congestion, DN performance Analytics, UE Communication Analytics can be used by the AF as assistance information for Application AI/ML operation. No normative work is required to enhance these analytics for this key issue.

\- When aggregated bit rate monitoring is requested for a set of QoS flows of a group of UEs against the pre-defined Group MBR threshold, the UPF exposure extension proposed by Solution#37 with the new event type 'Bit Rate Reporting per QoS Flow' consumed by NEF as described in Solution \#37 is a simple useful tool to be supported to monitor the aggregated throughput performance for the specific Application AI/ML operation.

\- AF knowledge on the session inactivity time with new monitoring parameters to recognize the current system performance respective to a given UE as described in Solution#1 is to be supported to assist Application AI/ML application operation. The monitoring procedures are based on UPF event exposure as described in Solution \#1. No new normative work is expected at SMF and UPF for this aspect.

\- How the UPF reports traffic/data volume is determined in UPEAS SID. Only after UPEAS study work is completed, AIMLsys will determine if the traffic/data volume is required to report from 5GC to AF.

It is understood that AI/ML applications can be assisted by enabling the monitoring of available resources in the network as above, however this is out of the scope of 3GPP.

## 8.2 Key Issue \#2: 5GC information exposure to UE

No normative work will be progressed in Rel-18 for Key Issue \#2.

NOTE: To facilitate Application AI/ML operation, how and which information are exposed from the AI/ML application server to the UE application via application layer is out of 3GPP scope.

## 8.3 Key Issue \#3: 5GC Information Exposure to authorized 3rd party for Application Layer AI/ML Operation

Proposal for the conclusions for KI#3 are based on solution evaluation in Clause 7.3 and to be used as basis for normative work based on the following principles: i.e.

\- The user consent is required for the UE related information or data analytics to be exposed to the 3rd party.

NOTE 1: The user consent checking procedure to verify the data exposure to AF will be sync up with SA3's conclusion during normative work.

\- UL/DL data rate exposure to AF based on AF request for data rate monitoring with solution without RAN impact.

\- The AF could subscribe AF Session/ PDU Session related information for the specific AF Session and reuse the existing notification and subscription mechanism.

NOTE 2: UL/DL data rate exposure to AF may consider the conclusions of FS_UPEAS KI#2.

## 8.4 Key Issue \#4: Enhancing External Parameter Provisioning

It is proposed that the specific mechanisms below proposed by KI#4 solutions are considered for the normative work on enhancing external parameter provisioning to support application AI/ML operations:

\- The following new parameters related to expected UE behaviour can be provisioned by the AF to the 5GC:

\- Confidence and/or accuracy levels for entries of the expected UE behaviour parameters in Table 4.15.6.3-1 of TS 23.502 \[4\]. A confidence and/or accuracy threshold may also be provided by the NF requesting expected UE behaviour parameters UDM, indicating that certain confidence and/or level of accuracy must be met for the parameter(s) to be notified by UDM to the NF.

\- Expected Inactivity Time of AIML traffic, provided as a time interval. Confidence and/or accuracy levels may be provided for this new parameter.

## 8.5 Key Issue \#5: 5GC Enhancements to enable Application AI/ML Traffic Transport

First Step: AI/ML data transfer window negotiation mechanism: It is agreed to consider the following principles some of them are based on solution#10, this principle are the basis for KI#5 conclusions:

When the AF requests the network to provide QoS with individual QoS parameters, one or more Requested Alternative QoS Parameter Set(s) in a prioritized order. Each Requested Alternative QoS Parameter Set is comprised of the following individual parameters: Requested 5GS Delay and Requested Guaranteed Flow Bitrate.

\- Principle 1: A new Application Data Transfer (ADT) policy is adopted to support application data transfer. The ADT policy includes a list of recommended time window(s) with predicted QoS parameters per UE per time window for the Application data transfer to start. The predicted QoS corresponds to one of the Requested QoS or Requested Alternative QoS that was requested by the AF.

\- Principle 2: The AF includes QoS requirements and possibly Alternative QoS Profile(s) with a combination of QoS parameters to which the application data traffic transmission is able to adapt , in the ADT policy negotiation request.

NOTE 1: The Alternative QoS Profile also includes the GFBR, however this is part of the AF request for ADT negotiation as it is not possible to derive a GFBR from the Network Performance Analytics and the Data Network Performance Analytics.

NOTE 2: AF includes the expected minimum QoS requirement from the network.

\- Principle 3: The PCF uses the extended Network Performance Analytics of NWDAF to derive the candidate time window(s) that may fulfil the requirements requested from AF. The PCF generates the list of ADT policies and feedback it to the AF. The QoS parameters in the ADT policy are determined by PCF based on the list of individual QoS parameters and the requested Alternative QoS Profile(s) from the AF.

\- The Network Performance Analytics support the following analytics outputs (other outputs may be discussed during normative phase). If and how to extend Network Performance Analytics and which input data to use will be discussed in normative phase.

\- Overall Average and Maximum Packet Delay for UL/DL per UE per time window;

\- Overall Average and Maximum Packet Loss Rate for UL/DL per UE per time window;

\- Overall Average, Minimum and Maximum Traffic rate for UL/DL per UE per time window.

\- Principle 5: The AF selects one of the ADT policies provided by PCF which may most fit for its requirement and indicate it to the PCF.

\- The Application traffic could be non GBR type or GBR type. The AF can provide Alternative QoS profile(s) for GBR flows only.

Second step - Application Data Transmission:

\- When the Application AI/ML begins e.g. to transport intermediate data, local training data, inference results, or model performance data, through 5GS, the AF can leverage the either existing service Nnef_AFsessionWithQoS_Create request message as described in clause 4.15.6.6 of TS 23.502 \[4\], including the specific QoS parameters for the AF session or new AF service which will be further determined during the normative phase.

Charging for Application AIML traffic transport:

\- Based on operator policy, charging of different Application AI/ML traffic needs to be perform, however, do not expect any normative work on this aspect in the SA WG2 specifications.

NOTE 3: An alignment can be required (if any) in the normative based on the outcome of SA WG5.

The AF configuration information used for application AI/ML transport is provisioned by the AF to the UE via application layer and may contain AF address(es), so no normative work is required.

The resolution of the AF address from a FQDN to the IP address of the AF or any associated AI/ML application server(s) is performed using the DNS server address for the PDU Session that is provided to the UE as defined in clause 5.6.10.1 of TS 23.501 \[3\].

No normative work is needed to provide authentication information to the UE for the UE to AI/ML traffic exchange.

## 8.6 Key Issue \#6: QoS and Policy enhancements

It is recommended to use the following conclusions based on solution evaluation in Clause 7.6 as the basis for the normative work on "KI \#6 QoS and Policy enhancements".

### 8.6.1 QoS performance measurement assistance to Application AI/ML operation

The monitoring and reporting resource utilization is performed for those performance KPIs and provided to the AF request for QoS procedures. The KPIs are described in clause 7.10 of TS 22.261 \[2\], i.e. Max. allowed UL/DL end to end latency etc.

\- The Guarantee Bit Rate is to be monitored using the existing reporting on whether GFBR can or cannot be fulfilled as defined in clause 5.7.2.4.1a of TS 23.501 \[3\] (Notification control with no Alternative QoS), which is already supported.

\- Existing QoS monitoring mechanism for URLLC services is reused for AI/ML traffic.

\- The AF can consume UE Communication Analytics for assisting the AI/ML application operations.

The AF may provide additional QoS parameter sets in Nnef_AFsessionWithQoS_Create service operation, together with timing (start time, duration) information for each QoS parameter set, and the PCF controls the timing that is provided by the AF and derives PCC rules according to the different QoS parameter sets and provides these rules to the SMF at the time indicated by the AF.

### 8.6.2 QoS request for a group of UEs

For AIML each of the members of the group has a PDU Session established, and the members of the group have been selected using procedures defined in conclusions to KI#7, such as location or QoS monitoring. In order to request QoS for the AIML communication with each of the members of the group, extensions to the procedure for the AF request with QoS, and Nnef_AFsessionwithQoS to provide a list of UE IP address are required. The QoS parameters that are provided apply for each of the UE to AIML application communication (i.e. the QoS parameters are not related to the aggregated QoS for all FL members), as such those are provided to the PCF in the Npcf_PolicyAuthorization. For the scenario where there are more than one PCFs serving the list of UEs whose group QoS is requested, extensions to BSF management discovery to provide a list of PCF addresses based on the UE addresses are supported.

No requirements to provide QoS for a group of UEs that have no PDU Session Established, as those cannot be selected as group members based on QoS monitoring, however if this is a requirement, conclusions on in GMEC study, Key Issue \#3: NEF exposure framework for provisioning of traffic characteristics and monitoring of performance characteristics applies, as this is resolved there.

No new 5G QoS parameter is required to be introduced to support group related Application AI/ML operation over 5GS.

### 8.6.3 Performance KPIs mapping to 5GS QoS parameters

It is concluded on the performance KPIs mapping to 5GS QoS parameters:

\- The 5GS should bind the AI/ML traffic to distinct QoS flow (i.e. not sharing QoS flow with other applications).

\- Whether existing 5QI values can be applied for AI/ML traffic or new 5QI values should be defined can be decided during normative phase.

## 8.7 Key Issue \#7: 5GS Assistance to Federated Learning Operation

\- The new member selection assistance functionality will be specified in the normative phase. The new member selection assistance functionality is defined as a functionality offered by the 5GC upon receiving a request from the AF to provide a list of candidate UEs and additional information.

\- The new member selection assistance functionality can be used by the AI/ML based application to select FL members.

\- The new member selection assistance functionality can provide a list of recommended UEs based on the parameters contained in the request from the AF.

\- The AF includes the following parameters as part of the request service operation to the 5GC:

\- UE list (a UE or a UE list).

\- The 5GC provides the following parameters as part of the response service operation to the AF:

\- List(s) of candidate UEs which can be selected by the AF.

NOTE 1: Any further parameters to be contained in the request/response from/to the AF and the procedure for selecting the list of candidate UEs will be discussed in the normative.

\- The new member selection assistance functionality will be hosted in NEF.

NOTE 2: If this new member selection assistance functionality is not used, AF can select the AI/ML members based on existing network exposure information as described in clause 4.15 of TS 23.502 \[4\].

NOTE 3: How to support the trusted AF with member selection assistance functionality will be addressed during the normative phase.

### 8.7.1 Performance monitoring/exposure

For the Key Issue \#7 on performance monitoring/exposure, the following principle is proposed for the normative work.

\- The group of UEs performances are provided by NEF to AF to assist the FL operation. These performances correspond to the QoS monitoring parameters, as defined in clause 6.1.3.18 of TS 23.503 \[5\].

### 8.7.2 Analytics to Assist Federated Learning Operation

AF can request NWDAF (via NEF if needed) to provide existing, enhanced or new analytics to assist with federated learning operation. NWDAF Analytics services will be enhanced as follows:

\- New latency performance analytics are introduced for NWDAF.

NOTE 1: The specific input and output parameters for the latency performance analytics among the group of UEs to assist the Application AI/ML FL operation will be determined during the normative phase.

\- UE mobility analytics are enhanced as follows: a list of areas can be provided by the AF, which helps the NWDAF to provide UE's direction and geographical distribution of the UEs to the AF.

\- WLAN performance analytics are enhanced to allow analytics outputs on a per UE granularity. For UEs' list provided by the AF, the NWDAF can provide WLAN performance analytics per UE granularity to assist AF to determine the final list of member UEs.

\- QoS sustainability analytics enhancements from eNA_Ph3 normative work can be used.

NOTE 2: Whether QoS sustainability analytics need to be enhanced on a per UE granularity or Service Experience analytics can be used can be discussed during the normative phase.

\- DN performance analytics are enhanced to include aggregated UE's traffic rate, variance of the traffic rate among UEs, variance of packet delay among UEs, variance of packet loss rate among UEs as well as top UEs who cannot meet the traffic rate, packet delay and loss rate threshold.
