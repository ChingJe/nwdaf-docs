---
spec: TR 23.700-04
version: 20.0.0
release: '20'
clause: 7
title: 7 Interim agreements
source_archive: 23700-04-k00.zip
source_document: 23700-04-k00.docx
content_origin: 3gpp-source
---

# 7 Interim agreements


## 7.1 Agreed Principles


### 7.1.1 Agreed Principles for KI#1

Editor's note: This clause will include the principles that are agreed as work progresses for the specific KI#1. This may be populated directly or e.g. also when a topic in clause 7.2.1 gets resolved and a principle is agreed.

The interim agreements on principles for KI#1 are as follows:

1\. UP path termination entity:

\- An 5GC NF acts as the UP path termination entity to support the transfer of standardized data over UP for UE data. The UP path termination entity will further transfer the received data to the UMTES.

NOTE 1: The UMTES is the AF/OTT server for UE-side model training.

Editor's note: Which 5GC NF will be defined as the UP path termination entity is FFS.

Editor's note: The data transfer from UP Path termination entity to the UMTES over UP or new interface is FFS.

2\. Discovery of UP path termination entity:

\- (the UMTES discovers UP termination entity) The UP path termination entity may be either locally configured at the UMTES, or the UP path termination entity may be discovered by the UMTES via NRF (if UMTES is trusted) or by NEF (the UMTES is untrusted).

Editor's note: Whether the UP path termination entity can be discovered by other network function is FFS.

The following options allow the UE to learn the UP path termination entity, either a or b.

Editor's note: How to select which option (2a or 2b or co-exist) to use is FFS.

a\) the UE has the UP path termination entity address/FQDN locally configured.

Editor's note: If there are multiple UP path termination entities, whether and how to ensure that the UE and the UMTES connect to the same UP Path termination entity is FFS.

b\) the 5G core network sends UP Path termination entity address (i.e. IP address/FQDN) to the UE in a N1 container via NAS.

Editor's note: It's FFS which 5GC NF sends the address of UP path termination entity to the UE

3\. UP path establishment:

\- The UE establishes a new PDU Session, or uses an existing PDU Session which has been established for UE data transfer, to transfer the standardized UE data, by reusing existing mechanism (i.e. URSP).

4\. Data transfer procedure:

\- UMTES sends the data transfer request to UP path termination entity including the information related to candidate UE(s) for data transfer (e.g. UE list, TAC(s), geographical area), and the requested data id.

Editor's note: How the mapping between the requested data id and the data to be transferred is determined is FFS.

\- A request for data transfer with requested data id is sent to the target UE(s).

Editor's note: Whether, where and how to select the target UE by CN is FFS.

Editor's note: Which entity sends the request to the target UE(s) is FFS.

Editor's note: How to send the data transfer request to the UE, i.e. via NAS signalling or UP is FFS.

Editor's note: Whether and how to select the UE by RAN is FFS.

NOTE 2: The list of target UEs is determined taking into account at least the candidate UE(s) provided by the UMTES and user consent.

\- The UE reports the requested data to the UP path termination entity via the established UP path.

\- The UP path termination entity can send requested data id to gNB via AMF, which may assist the gNB to decide data collection configuration to UE.

Editor's note: Whether and how to align the UE data measurement with the requirement of UE data transfer provided by the 5GC NFEditor's note: Whether the information (e.g. requested data) is sent with the data transfer request or within a separate request is FFS.

Editor's note: Whether the request data id is sent to the gNB depends on RAN's feedback

NOTE 3: Whether and what information to assist data collection configuration will be defined by RAN.

5\. Traffic Differentiation with following two alternatives:

\- Dedicated S-NSSAI/DNN is used to differentiate the traffic of the collected standardized data from the UE regular traffic per PDU Session level.

\- A QoS Flow is used to differentiate the transfer of the collected standardized data from the UE regular traffic per QoS flow level.

6\. Controllability:

\- The UP path termination entity may initiate / stop data transfer procedure

Editor's note: How the UP path termination entity initiate / stop data transfer procedure, i.e. via CP or UP is FFS.

Editor's note: Whether the UP path termination entity send to gNB via AMF an indication that data transfer is allowed for a UE is FFS.

\- The UE may accept / reject data transfer from the UP path termination entity. The UE may stop data transferring based on internal determination.

Editor's note: Whether the UE may request data transfer is FFS.

7\. Visibility:

\- The UP path termination entity verifies/matches the requested data to be transferred and the data that is being reported.

Editor's note: Whether and how to verify/match the requested data to be transferred and the data that is being reported in standardized manner is FFS and requires RAN's feedback on visibility requirements.

8\. User consent:

\- User consent checking with UDM for data transfer is needed.

Editor's note: Where and how to perform user consent check is FFS.

Editor's note: Whether using the existing or new user consent purpose is FFS. And coordination with SA WG3 is required.

### 7.1.2 Agreed Principles for KI#2

Editor's note: This clause will include the principles that are agreed as work progresses for the specific KI#2. This may be populated directly or e.g. also when a topic in clause 7.2.2 gets resolved and a principle is agreed.

#### 7.1.2.1 General

No impact on RAN and UE operation.

To reduce the reporting load of input data sources (e.g. UPF):

\- the analytics consumer may include the Analytics Filter Information (as described in clause 6.1.3 of TS 23.288 \[5\]). The NWDAF may derive related parameters for the event subscription towards the UPF or SMF for input data filtering.

\- the NF consumer supports the provisioning of the skip reporting instruction information in the subscription request, then the UPF do not send the event reports if they meet the skip reporting instruction, or UPF may bundle event reports of multiple subscriptions in a same Notify request, see TS 29.564 \[17\].

#### 7.1.2.2 Agreed Principles for Use Case \#1

\- The service consumer of the analytics service may be SMF, PCF, OAM, UPF, and AF. The consumer may take the output Analytics into account.

\- To provide analytics to the consumers, the NWDAF may provide the output analytics, including the statistics and predictions of:

\- Type of abnormal traffic, e.g. abnormal traffic due to DDoS, abnormal data packets patterns, unexpected traffic volume or burst.

\- Volume, rate or burst size of abnormal traffic.

\- Identifiers/addresses of affected UPF(s), UE(s), PDU session(s).

\- Identifiers/ information of the source, e.g. IP packet filter(s), IP Protocol (TCP, UDP, etc.), application ID, etc.

\- To derive the output analytics, the following input data may be collected from the UPF or SMF, to support the NWDAF-based analytics:

\- From UPF/SMF:

\- Information to identify the traffic pattern that can contain:

\- Source of traffic (e.g. IP packet filter(s), IP Protocol (TCP, UDP, etc.), application ID, etc.).

\- Traffic flow filters.

\- Identification of corresponding UEs.

\- Information of traffic characteristics.

\- The SMF as consumer of NWDAF analytics may for instance take the following actions upon the detection of the abnormal traffic:

\- UPF reselection to distribute the load across UPF instances (as defined in clause 6.3.3 of TS 23.501 \[2\]).

\- Configuring UPF to block or shape abnormal traffic, to enforce bandwidth limitations.

\- UPF as consumer of NWDAF analytics may for instance take the following actions:

\- Downlink traffic suppression (e.g., selective packet dropping or rate limitations).

\- adjust packet processing resources.

NOTE 1: UPF will only receive Analytics with reduced Output (i.e. type of abnormal traffic and traffic descriptor). UPF has operator-configured policies corresponding to the types of abnormal traffic, that can be activated for the received traffic descriptor. This does not apply on PDU-session level.

\- The PCF as consumer of NWDAF analytics may for instance take the following actions:

\- Policies creation or update and provisioning to SMF, e.g. executing traffic gating or shaping, enforcing bandwidth parameters (e.g. rate limiting to 5Mbps) or adjusting QoS parameters.

\- As stated in the architecture assumptions the UPF can detect abnormal traffic. When the abnormal traffic is detected the UPF can take actions to improve user plane performance using existing PDRs, and associated N4 rules such as QER or FAR, this is independent on the use of analytics.

NOTE 2: This can be done either by detection of abnormal traffic at UPF with reporting to SMF that results in related N4 rules or directly detection and enforcement of N4 rules at the UPF.

#### 7.1.2.3 Agreed Principles for Use Case \#2

\- A new analytics ID will be defined to support UC#2.

\- The service consumer of the analytics service is PCF. The PCF may take the analytics outputs into account to derive the QoS for the target application(s).

\- The NWDAF may provide the output analytics on traffic pattern information, including:

\- Maximum Burst Size/ Maximum Data Burst Volume, Maximum Bitrate, Guaranteed Bitrate, 5GS delay, Packet Error Rate, Periodicity.

\- Corresponding information of the traffic flow, e.g. the Application ID and/or Packet Filter Set, identifier of the service data flow(s) of the target application(s).

\- Identifiers of affected UEs, e.g. UE ID(s).

\- To derive the output analytics, the following input data may be collected from the UPF/SMF (If individual UEs are targeted, the NWDAF subscribes at the SMF to obtain related information from the UPF) to support the NWDAF-based analytics:

\- Information identifying the associated application/service data flow(s), e.g. packet filter, application ID of the traffic flow.

\- Information identifying the associated UEs.

\- Information related to the data flow characteristics, including, N6 delay, congestion information, indication that UPF drops packets and packet drop rate, data volume, packet delay, data rate, periodicity, and packet interval.

NOTE 1: The indication of UPF dropping packets and packet drop rate is only applied to the abnormal scenarios and the scenarios when UPF dropped buffered data.

NOTE 2: Whether the "User Data Usage Measures" event or another existing event is extended or new event(s) are defined for NWDAF data collection will be determined in the normative phase.

\- The consumer PCF may take the output Analytics into account, e.g. to generate or update PCC rules by adjusting the QoS.

\- The analytics consumer may include the Analytics Filter Information in the request, such as application description, UE ID/address, DNN and S-NSSAI, etc.

NOTE 3: The detailed information included in Analytics Filter Information will be determined in normative phase.

## 7.2 Topics for further consideration


### 7.2.1 Topics for further consideration for KI#1

Editor's note: This clause will include the topics for further consideration as work progresses for the specific KI#1. Eventually this clause should only contain topics for further consideration that did not result in agreements (i.e. in agreed principle(s) in a clause 7.1.1) and can either be then marked as not pursued or postponed to a future Release.

### 7.2.2 Topics for further consideration for KI#2

Editor's note: This clause will include the topics for further consideration as work progresses for the specific KI#2. Eventually this clause should only contain topics for further consideration that did not result in agreements (i.e. in agreed principle(s) in clause 7.1.2) and can either be then marked as not pursued or postponed to a future release.
