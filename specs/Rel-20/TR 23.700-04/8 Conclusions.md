---
spec: TR 23.700-04
version: 20.0.0
release: '20'
clause: 8
title: 8 Conclusions
source_archive: 23700-04-k00.zip
source_document: 23700-04-k00.docx
content_origin: 3gpp-source
---

# 8 Conclusions


## 8.1 Conclusions for KI#1

No agreement on this KI was reached and no work is proposed for normative phase.

## 8.2 Conclusions for KI#2


### 8.2.1 General

To reduce the reporting load of input data sources (e.g. UPF):

\- The analytics consumer may include the Analytics Filter Information (as described in clause 6.1.3 of TS 23.288 \[5\]). The NWDAF may derive related parameters for the event subscription towards the UPF or SMF for input data filtering.

\- The NF consumer supports the provisioning of the skip reporting instruction information in the subscription request, then the UPF do not send the event reports if they meet the skip reporting instruction, or UPF may bundle event reports of multiple subscriptions in a same Notify request, see TS 29.564 \[17\].

### 8.2.2 Conclusions for KI#2 Use Case#1

Regarding KI \#2, the following principles are concluded for UC#1:

\- A new Analytics ID "Abnormal user plane traffic" will be introduced for UC#1. The consumer may be SMF, PCF, UPF and AF.

NOTE 1: Whether OAM can be the Analytics consumer will be determined in the normative phase, coordination with SA WG5 is needed.

\- The output analytics, both statistics and/or predictions, include at least:

\- Identification of the traffic flow (e.g. traffic flow descriptors).

\- Type of abnormal traffic.

\- Information of the detected anomality, such as (e.g. malformed, unexpected traffic volume, burst, etc.), malicious (e.g. DDoS).

\- Volume, rate or burst size of abnormal traffic.

\- Identifiers/addresses of UPF(s), interface if applicable, list of UE(s), list of PDU session(s) affected by identified abnormal traffic type.

\- Identifiers/ information of the traffic source, e.g. IP packet filter(s), IP Protocol (TCP, UDP, etc.), application ID, etc.

\- Time window (e.g. start, duration).

\- To derive the output analytics, the following input data may be collected from the UPF, SMF or AF. (If individual UEs are targeted, the NWDAF subscribes at the SMF to obtain related information from the UPF):

\- Information identifying the associated application/service data flow(s) (e.g. packet filter, the traffic flow, Source and/or, destination IP address and port, Protocol Type, URL lists, QFI, Application ID).

\- Identifiers of corresponding UEs.

\- Information of traffic characteristics from SMF or UPF, e.g. measured UL/DL data volumes, measured UL/DL data rates.

\- Information of UP pattern, e.g. pattern type, including malformed, unknown, duplicate, fragmented, etc.

NOTE 2: Whether any additional traffic identifier is needed will be determined during normative work.

\- Traffic characteristics of normal traffic and abnormal traffic from external server/AF, and the abnormal type for the abnormal traffic, which can be used by the NWDAF to determine whether the input traffic is abnormal traffic by comparing the characteristics of input traffic with the characteristics of normal traffic or abnormal traffic.

The event subscription by NWDAF to UPF can Target any UE or specific UEs and provides information about targeted traffic abnormalities, thresholds for the volume/burst and/or sampling periods. When the NWDAF requests to report traffic abnormalities, then the UPF only reports when corresponding traffic abnormalities are detected and threshold are exceeded.

NOTE 3: How UPF provides information of traffic abnormalities to NWDAF will be determined in the normative phase.

NOTE 4: Related UPF and SMF events will be determined in the normative phase. Whether the "User Data Usage Measures" event or another existing event is extended or new event(s) are defined will be determined in the normative phase.

The SMF as consumer of NWDAF analytics may determine the mitigation actions for example:

\- UPF reselection to distribute the load across UPF instances (as defined in clause 6.3.3 of TS 23.501 \[2\] and clause 4.3.5 of TS 23.502 \[3\]).

\- Configuring UPF to enforce the downlink traffic suppression (e.g., selective packet dropping or packet per second limitations) or shape abnormal traffic, to enforce bandwidth limitations. For each observed traffic anomaly that can be reported by the NWDAF, corresponding mitigation actions are configured in the SMF. Then, the SMF provides them to the UPF. The SMF may use actions on non-PDU session level applicable to any PDU session.

The UPF as consumer of NWDAF analytics may for instance take the following actions upon the detection of the abnormal traffic:

\- Downlink traffic suppression (e.g. selective packet dropping or rate limitations).

\- When the UPF is consumer, the subscription initiation follows the "Subscribe-Notify" as in clause 7.1.2 of TS 23.501 \[2\] (Figure 7.1.2-3). In this case, the SMF subscribes to NWDAF services on the new Analytics ID on abnormal data packets patterns on behalf of the UPF and indicate that reduced output is requested, and the NWDAF can send directly the analytics to the UPF for activation and deactivation of the related rules in the UPF for the abnormal data packet handling, such as enforcing the downlink traffic suppression (e.g., selective packet dropping or packet per second limitations) or adjust the packet processing resources.

NOTE 5: UPF will only receive Analytics with reduced Output (i.e. type of abnormal traffic and traffic descriptor).

NOTE 6: Whether and how to configure the UPF with actions from OAM or SMF for abnormal traffic handling will be determined during normative work.

As stated in the architecture assumptions the UPF can detect abnormal traffic. When the abnormal traffic is detected the UPF can take actions to improve user plane performance using existing PDRs, and associated N4 rules such as QER or FAR, this is independent on the use of analytics.

The PCF as consumer of NWDAF analytics may for instance take the following actions upon the detection of the abnormal traffic:

\- Policy creation or update and provisioning to SMF, e.g. executing traffic gating or shaping, enforcing bandwidth parameters (e.g. rate limiting) or adjusting QoS parameters.

The AF (e.g. IoT server) as consumer of NWDAF analytics could take action to correct abnormal traffic.

### 8.2.3 Conclusions for KI#2 Use Case#2

The following conclusions for KI#2 Use Case#2 will be supported by normative work:

\- A new analytics ID will be defined to support UC#2.

\- The service consumer of the analytics service is PCF. The consumer PCF may take the output Analytics into account, e.g. to generate or update PCC rules by adjusting the QoS.

\- The NWDAF may provide the output analytics on traffic pattern information to the consumer, including:

\- Maximum Burst Size/ Maximum Data Burst Volume, Maximum Bitrate, Guaranteed Bitrate, 5GS delay, Packet Error Rate, Periodicity.

\- Corresponding information of the traffic flow, e.g. the Application ID and/or Packet Filter Set, identifier of the service data flow(s) of the target application(s).

\- Identifiers of affected UEs, e.g. UE ID(s).

\- To derive the output analytics, the following input data may be collected from the UPF/SMF (If individual UEs are targeted, the NWDAF subscribes at the SMF to obtain related information from the UPF) to support the NWDAF-based analytics:

\- Information identifying the associated application/service data flow(s), e.g. packet filter, application ID of the traffic flow.

\- Information identifying the associated UEs.

\- Information related to the data flow characteristics, including, N6 delay, congestion information, indication that UPF drops packets and packet drop rate, data volume, packet delay, data rate, periodicity, and packet interval.

NOTE 1: The indication of UPF dropping packets and packet drop rate is only applied to the abnormal scenarios and the scenarios when UPF dropped buffered data.

NOTE 2: Whether the "User Data Usage Measures" event or another existing event is extended or new event(s) are defined for NWDAF data collection will be determined in the normative phase.

\- The analytics consumer may include the Analytics Filter Information in the request, such as application description, UE ID/address, DNN and S-NSSAI, etc.

NOTE 3: the detailed information included in Analytics Filter Information will be determined in normative phase.

\- To reduce UPF load of data reporting, the NWDAF may configure thresholds for traffic characteristics in event subscription. The UPF may only report data when the thresholds are exceeded.
