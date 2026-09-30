---
spec: TR 23.700-04
version: 20.0.0
release: '20'
clause: 5
title: 5 Use Cases and Key Issues
source_archive: 23700-04-k00.zip
source_document: 23700-04-k00.docx
content_origin: 3gpp-source
---

# 5 Use Cases and Key Issues


## 5.1 Use cases

Editor's note: This clause describes use cases for WT#2 in the WID of this item (see details in SP-250413).

### 5.1.1 Use Case \#1: AI/ML-assisted user plane traffic pattern and behaviour analysis to support efficient performance of User Plane

The UPF is responsible for managing substantial volumes of uplink and downlink traffic originating from UEs' applications or application servers. It must be robust and resilient against abnormal data traffic, for examples, large amount of abnormal or malicious IP packets, flows, bursts, syn-ack signalling. Without actions to mitigate the impact of such potential abnormal or malicious traffic pattern in the user plane, the UPF may experience performance degradation which can lead to resource inefficiency. Use of AI/ML can be studied:

\- to provide analytics on user plane traffic patterns;

\- to apply mitigation strategies using analytics, e.g. avoid abnormal or malicious traffic;

so as to improve efficient performance of user plane as well as UPF's robustness.

### 5.1.2 Use Case \#2: User plane performance optimization with assistance of NWDAF

It is beneficial for the network to know about the traffic pattern in advance in order to optimize user plane performance, e.g. better QoS control. For instance, the network can generate MDBV, Periodicity for TSC service based on the traffic pattern as described in TS 23.501 \[2\].

Currently, the above mechanism can only be applicable for services with special protocols, e.g. TSN. For other kinds of applications, most of them rely on transport layer protocol to mitigate congestion, e.g. TCP or QUIC as described in RFC 793 \[8\] and RFC 9000 \[9\]. Then, the application behaviour (e.g. packet sending rate) is related to the network status, (e.g. the E2E delay and packet loss, etc.). Hence, it is hard to give a precise traffic pattern in advance to the network from the AF/AS.

The analytic capability of NWDAF can generate the statistic or prediction of the traffic pattern for 5GC to do user plane performance optimization, e.g. assistance to QoS control management, in advance.

NOTE 1: Solutions for this use case should aim at avoiding extensive signalling.

NOTE 2: Solutions will reuse existing policy and QoS control as much as possible.

NOTE 3: Solutions shall not include any recommendations by the NWDAF.

## 5.2 Key Issues


### 5.2.1 Key Issue \#1: Transfer of data over UP for UE data collection


#### 5.2.1.1 Description

This key issue aims to provide solutions for UE data collection to meet the requirements for AI/ML for NR air interface with UE-side model training. This process requires the transfer of training data from the UE to the 5G core and then to the OTT server.

This key issue will investigate the following aspects:

\- How to support the standardized transfer of standardized data over UP for UE data collection including how to initiate, terminate and manage UE data transfer over the UP connection.

\- How to support in 5GC the full visibility of standardized UE data contents.

\- How 5GC can support the full controllability of standardized UE data transfer.

\- How to differentiate the traffic for UE data collection from the UE regular traffic in the 5GC.

\- Whether and how the UE knows about what the standardized data to be transferred are.

### 5.2.2 Key Issue \#2: AI/ML-assisted user plane traffic pattern and behaviour analysis to support efficient performance of User Plane


#### 5.2.2.1 Description

Use of AI/ML can be studied to provide analytics on user plane traffic patterns and how to use the analytics to improve user plane performance. In this context, it is proposed to study:

\- How to provide analytics (e.g. enhancing existing in TS 23.288 \[5\] or introducing new) including the aspects of required inputs/outputs to improve performance of user plan.

\- Which consumers can subscribe/request the analytics and how the consumers use the analytics to take actions to improve user plane performance.

\- Whether and how to train ML models and perform the inference for the user plane analytics, considering solutions that minimize the load on Control Plane NFs.
