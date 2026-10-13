---
spec: TR 23.700-80
version: 18.0.0
release: '18'
clause: 5
title: 5 Key Issues
source_archive: 23700-80-i00.zip
source_document: 23700-80-i00.docx
content_origin: 3gpp-source
---

# 5 Key Issues


## 5.1 Key Issue \#1: Monitoring of network resource utilization for support of Application AI/ML operations

This KI addresses WT#1.1a. To support AI/ML based services/applications, 5GS network resources are utilized for AI/ML model and data distribution, transfer and/or training for various applications.

To monitor the network resource utilization in the 5GS relevant to the UE's performance for supporting AI/ML based services/applications, the following aspects are to be studied:

\- How to monitor network resource utilization relevant to the UE's performance (e.g. such as the data throughput provided to the UE) for supporting Application AI/ML model/data distribution and sharing related operations, including:

\- Whether and how to measure network resource utilization relevant to the UE's performance (as described in TS 22.261 \[2\]) in the 5G system to support Application AI/ML traffic.

NOTE: Network resource utilization measurements relevant to UE's performance are done by the network, additional UE measurements are out of scope of this study. Neither Uu nor any AN impacts are in the scope of this study.

\- Whether and what enhancements (including new monitoring events) are required to existing monitoring mechanisms (e.g. monitoring capability as specified in clause 5.20 of TS 23.501 \[3\]) to capture and/or predict network resource utilization relevant to UE's performance for traffic related to Application AI/ML operations.

## 5.2 Key Issue \#2: 5GC information exposure to UE

As documented in SA WG1 TS 22.261 \[2\], the application layer AI/ML model can be downloaded from AF/AS to UE to facilitate local intelligence, i.e. the UE can do inference and make decisions locally, for the use cases such as image recognition, speech recognition, media quality enhancement, etc. The UE may need information from the 5GC to assist its local decision.

This Key Issue is related to WT#1.1b, and aims at studying whether and how to enhance the 5GC to expose information to the UE to facilitate its Application AI/ML operation (e.g. Model Training, Splitting and inference feedback etc.):

\- Whether and for which purpose there is a need for 5GC to expose information or data analytics to the UE, and what are the information or data analytics that can be provided by 5GC to the UE to assist the Application AI/ML operation at the UE side.

\- How the 5GC exposes such information or data analytics to the UE.

NOTE: SA WG1 feedback on exposure of network information to the UE and security analysis by SA3 will be taken into account before the conclusion of this Key Issue.

## 5.3 Key Issue \#3: 5GC Information Exposure to authorized 3rd party for Application Layer AI / ML Operation

This KI is related to WT#1.1b.

It is required for 5GC to expose different types of assistance information to AF for AI / ML operation. This KI will study the following aspects:

\- Whether and what assistance information and events are exposed from 5GS to AF, more specifically, the prediction of UE and/or network conditions and performance (e.g. location, QoS, load, Congestion, etc.) as described in TS 22.261 \[2\]? How?

NOTE: User consent is assumed only for UE-related information. Coordination with SA WG3 on exposing UE-related information to an AF is required to ensure privacy and security requirements are met.

## 5.4 Key Issue \#4: Enhancing External Parameter Provisioning

This key issue is related to Objective 1.c.

As outlined in clause 4.15.6 of TS 23.502 \[4\], provisioning capability allows an external party to provision the information, such as expected UE behaviour and service specific parameters to 5G network functions. For example, the expected UE behavioural information consists of information on expected UE movement and communication characteristics. Provisioned data can be used by the other NFs. The Expected UE Behaviour parameters characterise the foreseen behaviour of a UE or a group of UEs.

However, it is to be studied how external parameter provisioning to the 5GC can be enhanced for the assistance to Application AI/ML operation.

In order to enhance external parameter provisioning to 5GC for the assistance to Application AI/ML operation, the following needs to be studied:

\- Whether and what additional parameters can be provisioned to 5GC by an external party for the assistance to Application AI/ML operation.

\- How additional parameters are provisioned to 5GC by an external party.

\- How provisioned parameters are retrieved or maintained or used by 5GC NFs (e.g. UDM, AMF or SMF).

NOTE: Privacy aspects will be handled by SA WG3.

## 5.5 Key Issue \#5: 5GC Enhancements to enable Application AI/ML Traffic Transport

This key issue is to address WT#1.1d.

Different levels of interactions are expected between UE and AF as AI/ML endpoints based on TS 22.261 \[2\] to exchange AI/ML model, intermediate data, local training data, inference results or model performance as Application AI/ML traffic(s).

The following aspects are to be studied, in regard to how to assist the Application AI/ML operation:

\- Whether and how the existing 5GC data transfer/traffic routing mechanisms are re-used or enhanced to support the transmission of the Application AI/ML traffic(s) over 5GS between AI/ML endpoints (i.e. UE and AF)?

NOTE 1: Privacy aspects will be handled by SA WG3.

\- Whether new 3GPP charging mechanisms are required for identification and charging of Application AI/ML traffic over the user plane that cannot be attributed or charged to the end user?

NOTE 2: Charging aspects will be done in collaboration with SA WG5.

## 5.6 Key Issue \#6: QoS and Policy enhancements

This key issue is applicable to Objective 2 (i.e. WT#1.2) which addresses QoS and Policy enhancements to support Application AI/ML operation traffic for AI/ML model transfer in 5GS, to address SA WG1 requirement as specified in TS 22.261 \[2\].

As defined in requirements specified in TS 22.261 \[2\], the 5G system can at least support three types of AI/ML operations:

\- AI/ML operation splitting between AI/ML endpoints.

\- AI/ML model/data distribution and sharing over 5G system.

\- Distributed/Federated Learning over 5G system.

In order to conclude whether the 5G system meets QoS performance requirements as specified in clause 7.10 of TS 22.261 \[2\], this study needs to determine whether any additional QoS and/or policy enhancements to the 5G system are required

Therefore, this key issue will study solutions on below aspect(s) for the possible QoS and Policy enhancements to support Application AI/ML operation:

\- Whether the current QoS model for 5GS defined in clause 5.7 of TS 23.501 \[3\] needs to be enhanced to support performance KPIs defined in clause 7.10 of TS 22.261 \[2\]. If new QoS parameters are needed whether the existing Policy framework in TS 23.503 \[5\] needs to be enhanced to account for the new QoS parameters and how.

\- Whether the current QoS model for 5GS defined in clause 5.7 of TS 23.501 \[3\] needs to be enhanced to account for the following types of application AI/ML traffic:

i\) data collected for training and inference and shared by the application server or UE over the 5GS; and

ii\) AI/ML models trained by the application server or UE shared over the 5GS.

\- Whether the current QoS Monitoring can be used to expose, to an authorized 3rd party, monitoring and status information, regarding resource utilization of network services associated to this 3rd party AI/ML application, as specified in clause 6.40 of TS 22.261 \[2\]. If not, what enhancements are needed.

\- Whether the current QoS model defined in clause 5.7 of TS 23.501 \[3\] and policy framework defined in TS 23.503 \[5\] provides support to assist the application AI/ML operation to be informed on predictions of changing network conditions (i.e. bitrate, latency, reliability) in a timely manner. If not, what enhancements are needed.

## 5.7 Key Issue \#7: 5GS Assistance to Federated Learning Operation

This KI is to study whether and how 5GS provides assistance to AF and the UE for the AF and UE to manage the FL operation and model distribution/redistribution (i.e. FL member selection, group performance monitoring, adequate network resources allocation and guarantee) to facilitate collaborative Application AI/ML based Federated Learning operation between the application clients running on the UEs and the Application Servers.

In order to provide assistance to the AF and the UE for FL operations, it is proposed to study the following aspects:

**On assistance to selection of UEs for FL operation:**

\- Whether, how and what information provided by 5GC to the AF can help the AF to select and manage the group of UEs which will be part of FL operation.

NOTE: The FL group management should be controlled and managed by the AF.

\- Whether, how and what information is required by the 5GC in order to assist the AF for selecting and managing the group of UEs which will be part of FL operation.

**On performance monitoring/exposure:**

\- How to monitor and expose a UE or a group of UEs performance (e.g. aggregated QoS parameters) as described in TS 22.261 \[2\] related to FL operations.

\- Whether and what existing or new monitoring events (e.g. QoS, location, load, congestion) are required to capture specific System Performance and Predictions for traffic related to AI/ML operations for FL operation.

**On FL performance:**

\- How to assist AF to increase the FL performance (e.g. to manage latency divergence) among UEs when the application server receives the local ML model training information from different UEs in order to perform global model update.
