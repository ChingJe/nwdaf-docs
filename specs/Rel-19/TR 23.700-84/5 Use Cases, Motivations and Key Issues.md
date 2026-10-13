---
spec: TR 23.700-84
version: 19.0.0
release: '19'
clause: 5
title: '5 Use Cases, Motivations and Key Issues'
source_archive: 23700-84-j00.zip
source_document: 23700-84-j00.docx
content_origin: 3gpp-source
---

# 5 Use Cases, Motivations and Key Issues


## 5.1 Use Cases


### 5.1.0 Guidelines

Use cases as captured in the following sub-clauses are related to Key Issue \#2, Key Issue \#3 and Key Issue \#4. Table 5.2.0-1 shows the mapping of Key Issues to Use Cases.

NOTE 1: Capturing use cases for Key Issue \#4 is optional.

NOTE 2: For KI#1, use cases are based on Positioning accuracy enhancements, case 2b and case 3b (as defined in TR 38.843 \[6\]). No additional use cases for KI \#1 will be defined in this TR.

### 5.1.1 Use Case \#1: NWDAF-assisted QoS enhancement

Currently, the QoS parameters are determined by the PCF based on its knowledge, e.g. AF requirements, analytics provided by the NWDAF, etc. After applying the determined QoS parameters to the service, the PCF may determine whether or not the current QoS can fully satisfy the service requirements based on the Service Experience analytics provided by the NWDAF. If the current QoS cannot satisfy the service requirements, the PCF may update the QoS parameters and informs the new parameters to SMF. Then the PCF may require new Service Experience analytics to check whether the updated QoS parameters can satisfy the service requirements. Based on this current framework, it may require several iterations to work out the ideal QoS parameters.

Using its knowledge based on data collection, NWDAF can assist the PCF in determining QoS parameters that can achieve the expected service experience requirements.

### 5.1.2 Use Case \#2: Enhancements to QoS Determination with NWDAF Assistance

A use case is provided for how the network can benefit from the NWDAF-assistance for QoS determination and setup for the purpose of optimising the overall network performance and signalling based on operator's policy.

After UE registers with the 5GS, a PDU session set up might be required. Each PDU session is associated with a default QoS rule which provides a default QoS treatment for data flows. Currently the characteristics of the default QoS is determined by the subscribed default values (for parameters such as 5QI, ARP) which the SMF may obtain from the UDM. The default QoS rule might be sufficient for basic browsing or instant messaging over IP, whereas it may not able to satisfy the relatively high service requirements, i.e. of video streaming applications which require better QoS treatment. For example, for V2X and XRM services, the applications may require transmitting traffic with Guaranteed Bit Rate (GBR) or to use certain standardised 5QI values even for non-GBR QoS flows. Therefore, the default QoS requirements may not be able to support such applications.

When the QoS flows with different requirements from the default flow are required, modification to the PDU session and thereby to establish a new QoS flow with the required characteristics might be needed. Such modification will result in significant system-wide signalling, including NAS signalling messages between the UE and the 5GC, signalling within 5GC (i.e. signalling between SMF, UPF, PCF), signalling between 5GC and RAN, and also the RRC messages between the RAN and the UE, etc.

In order to optimise the network performance by determining QoS in a more intelligent manner, it would be beneficial for the 5GC to leverage NWDAF assistance. For example, when the UE or network trigger PDU session establishment or modification for a new QoS flow with QoS requirements driven by a user or service, it would be beneficial if the QoS characteristics are determined by the network by considering the predictions and measurements of some UE and network related information and also service related information (e.g. service requirements provided by the AF). The information considered by the 5GC could be some patterns in terms of frequency of use of one or more services and the potential QoS requirements to be emerged from the UE subsequently, the QoS sustainability of the UE or of an area the UE belongs to, the corresponding UE locations, the service requirements provided by the AF, etc. Therefore, the PDU session and QoS flow can be established or modified in a more 'future proof' and multiple-service-compatible manner and reduce the potential modifications of the existing QoS flow and the corresponding policy control, e.g. PCC rules.

Based on this use case, potential enhancements to 5GC functionality e.g. of the NWDAF, PCF, to enhance the policy control and QoS by considering operator's policies, will improve the network performance and UE experience significantly.

### 5.1.3 Use Case \#3: NWDAF assistance in device signalling storm prevention and mitigation

In some scenarios, e.g. NB-IoT CP optimization scenario, UEs send small data over NAS signalling. In case e.g. the application on the NB-IoT UEs is not implemented correctly, e.g. report data at the same time, the NB-IoT devices in some area may be active at the same time so that a large amount of NAS signalling may be transmitted into the network, which may cause signalling storm in that area during that period of time.

Indeed, negotiation with the application provider to mitigate the signalling is useful. However, this use case considers that many e.g. IoT application providers will emerge into the market, and some of them may not conduct such mitigation.

In such case, NWDAF can provide some analytics or prediction on behaviours of e.g. NB-IoT devices and the network status within some area of interest to help network to handle these e.g. NB-IoT devices to avoid signalling storm.

NWDAF has already provided some Analytics IDs related to abnormal behaviours and load status for network, e.g. "NF load information", "Abnormal behaviour", "Suspicions of DDoS attack", "Session Management Congestion Control Experience", etc. in TS 23.288 \[5\] to assist in protecting the network from signalling storm and abnormal network functions from different aspects. However, the prevention of signalling storm caused by large amount of signalling e.g. NB-IoT scenario with the assistance of NWDAF is not studied. Apparently, existing Analytics IDs may not be enough for the network to handle a signalling storm. Further investigation is required on how to prevent and mitigate signalling storm in e.g. NB-IoT scenario.

### 5.1.4 Use Case \#4: Motivation and Support for VFL in 5GC

It is well known in the AI/ML literature that VFL is a federated learning setting where multiple parties perform training on data sets that share the same sample space but differ in feature space. Because of this, an alignment in sample and feature spaces among participating entities is usually required before applying VFL. VFL further allows to perform joint training without exposing raw data, with each entity owning its own model but not needing the same model architectures. This is a way in which VFL differs from HFL. TS 23.288 \[5\] provides NWDAF specification support for HFL but no VFL support is available, and the existing procedures defined for HFL may be enhanced to support VFL, as mentioned in clause 5.2.4.

This use case proposes to support VFL in 5GC for analytics derivation leveraging sample and optionally (if needed) feature alignment between the entities participating in VFL, and where the main entity facilitating the VFL operation is NWDAF and other entities may be other NWDAF instances. In a multi-vendor scenario, this would allow participating NWDAF instances to collaborate in VFL without the need for model sharing.

Some of the motivations to introduce VFL in 5GC are as follows:

\- VFL may support distributed inference.

\- A more flexible technique: In VFL vendor specific local models and features can be deployed in each participant, so that it is possible that each participant selects or configures the local model to be used, as such vendor or operator specific local models and features, including not standardized features, are simpler to implement comparing with HFL.

In one PLMN where multiple NWDAFs are deployed, each NWDAF instance may perform data collection according to their available data sources. Depending on the Analytics ID and the deployment scenario however, the different NWDAF instances may share the same sample space or train on different sample spaces. VFL would be beneficial on the former case. Furthermore, in VFL each NWDAF instance does not collect the same input data for the same Analytics ID.

### 5.1.5 Use Case \#5: NWDAF support for observed service experience analytics based on VFL

When NWDAF provides observed service experience analytics, as in other analytics that require input data from the AF, policies in the PLMN and or the AF may prevent raw data to be exchanged directly between NWDAF and an external AF, as NWDAF is in the PLMN and the AF is outside the PLMN and the user data has high privacy protection needs.

Furthermore, NWDAF and AF may have different features of the same sample identity, which is a requirement of VFL. In such cases, VFL can be very helpful to break the data isolation and enable joint training between NWDAF and AF. However, regardless of the entities involved in VFL, the application of VFL among two entities requires alignment of samples and features to make sure the above VFL requirement is addressed.

Additionally, since the inference for VFL is also a distributed inference, no raw data will be shared in the inference as well as in the training. Each entity uses local data to do the inference. And the output will be gathered to get the final result.

The use case for observed service experience analytics is illustrated as follows. It provides the real user feedback to the network so that the network could self-optimize and offer customized services according to the true user needs. Due to the issue of data privacy, the AF(s) and NWDAF may not be able to exchange the data directly (The NWDAF may interact with several AFs, e.g. to provide Service Experience for a Network Slice). By leveraging VFL technology with the situation that the datasets of distributed nodes, NWDAF and AF can jointly participate to train an ML model for observed service experience. And they would do inference after the training respectively to generate the final result.

NWDAF and AF(s) would collect their local training data, respectively (e.g. access speed, network access delay for NWDAF, stall time, frame rate for AF). Note that data collection at the AF is out of the scope of this use case.

Two scenarios are identified in this case:

Scenario 1: NWDAF initiates VFL training process.

Scenario 2: AF initiates VFL training process.

This use case is also applicable for other analytics where AF and NWDAF can collaborate, e.g. DN Performance Analytics.

NOTE 1: This use case is applicable only if the AF is capable of participating in VFL procedure as a training entity. AF ML model training specification is out of scope.

NOTE 2: When an AF initiates the VFL, then only one AF is involved. When the NWDAF initiates the VFL, then multiple AFs can be involved.

### 5.1.6 Use Case \#6: Analytics-assisted prevention of abnormal NF behaviour causing signalling storm and mitigation of its impact in the network

The presented use case is to elaborate on how analytics can assist entities in 5GC, e.g. NFs, OAM, etc. to prevent and mitigate the impact of abnormal behaviour i.e. signalling storm in the network.

The network may, based on the analytics, discover abnormal NF behaviour, i.e. signalling storms, and begin an enforcement action that can contribute to prevention of signalling storm or mitigation the impact of it.

## 5.2 Key Issues


### 5.2.0 Mapping of Key Issues to Use Cases

Table 5.2.0-1: Mapping of Key Issues to Use Cases

| Key Issues | Use cases |
|------------|-----------|
| 2          | 4, 5      |
| 3          | 1, 2      |
| 4          | 3, 6      |

### 5.2.1 Key Issue \#1: Enhancements to LCS to support Direct AI/ML based Positioning

This key issue aims to provide solutions for whether and how to consider enhancements to support AI/ML based positioning for case 2b, 3b as defined in TR 38.843 \[6\], which will investigate the following aspects:

\- Study whether and how an AI/ML model for direct AI/ML positioning (i.e. case 2b/3b) is handled:

\- Which entity trains the model for direct AI/ML positioning and if the entity that train the model and the consumer are different, how the model consumer gets the trained AI/ML model;

\- Which entity act as the model consumer that will use the trained model to perform inference and/or derive UE position;

\- Define procedures for data collection with objective to train AI/ML models for direct AI/ML positioning.

\- Whether and how to support direct AI/ML positioning at LMF with additional 5GC enhancements.

\- How to monitor model performance for ML models used for direct AI/ML based positioning.

NOTE 1: UE data collection, model delivery and transfer to the UE and model identification/management are not within the scope of this key issue.

NOTE 2: Any data to be collected from UE/RAN by LMF for the model training/model inference/model performance monitoring for LMF-side model is assumed to be defined by RAN WGs.

NOTE 3: Any potential impacts for case 1/2a/3a in TR 38.843 \[6\], are out of the scope and any potential alignment work will be based on the possible requirements defined by RAN WGs considering the conclusions in TR 38.843 \[6\].

### 5.2.2 Key Issue \#2: 5GC Support for Vertical Federated Learning

This key issue aims to provide solutions for enabling 5GC support for vertical federated learning (VFL) involving NWDAF and/or AF, where no raw data need to be exchanged but some level of coordination is still required when training and inference are performed on local models. In particular, datasets used for each local model need to share the same samples while holding different features.

In Rel-18, ML model sharing between NWDAFs has been studied as a part of Horizontal Federated Learning. However, federated learning between NWDAF and AF has not been studied (e.g. when the NWDAFs and/or AFs are in different domains, locations, regions etc).

Vertical Federated Learning (VFL) can be considered as an alternative mechanism for distributed functionalities of an ML model. Note that, as scoped in Rel-19, NWDAF and/or AF may be involved for VFL.

This Key Issue aims to study architecture enhancement to support VFL, which allows the cooperative AI/ML training and inference with the following aspects:

\- Identify VFL use cases and under which conditions, and for which entities these VFL use cases show that VFL is justified to train ML models.

\- Whether and how to support architecture enhancement for supporting VFL for model training and/or inference. In particular:

\- Whether and how the existing NF discovery and selection needs to be enhanced.

\- Whether and how ML Model training and/or inference related procedures need to be enhanced to support VFL.

\- Whether and how to do performance monitoring for the ML model trained via VFL.

\- Whether and how to provide ML Models to the participants in the VFL training process.

\- How to support sample and feature alignment among the participating network entities when performing VFL.

NOTE 1: Application layer-based VFL requiring communication between AFs and/or UEs application client, is out of scope.

NOTE 2: During the study on this KI, consultation with SA WG3 is required for handling security aspects.

NOTE 3: RAN and UE aspects are out of scope.

NOTE 4: The existing procedures defined for Horizontal FL in TS 23.288 \[5\] will be taken into account when studying the procedure for VFL.

### 5.2.3 Key Issue \#3: NWDAF-assisted policy control and QoS enhancement

The NWDAF can gather quite a lot of data from 5GC NFs, AF and OAM and thus may further assist the PCF in making PCC decisions (which traditionally determine QoS parameters based on its own data and knowledge as well optional statistics and predictions collected from the NWDAF).

This key issue aims to study whether and what is additionally needed to be supported in order to enhance 5GC NF operations related to policy control and QoS with the assistance of the NWDAF.

In this key issue, the following aspects will be studied:

\- Identification of use cases where policy control and QoS can be further enhanced with assistance from NWDAF.

\- Whether and how to introduce new 5GC functionality e.g. of the NWDAF and/or PCF to enhance the policy control and QoS, considering operator's policies.

\- Whether and what additional input information is needed by the NWDAF for providing an assistance to policy control and QoS, and how to gather it.

\- Whether and what output information, on top of already provided, the NWDAF can provide to assist with policy control and QoS enhancements.

\- Whether and how to evaluate the quality of the enhanced NWDAF assistance to policy control and QoS.

NOTE: The study will focus primarily on existing enforcement mechanisms when available and identify new ones only when no existing ones can be used.

### 5.2.4 Key Issue \#4: NWDAF enhancements to support network abnormal behaviours (i.e. signalling storm) mitigation and prevention

This key issue aims to provide solutions for prediction, detection, prevention, and mitigation of network abnormal behaviours, i.e. signalling storm, with the assistance of NWDAF. In particular, the following aspects will be addressed:

\- Identify scenarios that can result in a signalling storm situation.

\- Whether and how existing analytics or new analytics can be used to assist detection and/or prediction of signalling storm, including aspects of input /output data that needs to be collected/provided by the NWDAF.

What NF(s) will be consumer of such analytics and whether and how they can use them:

\- Whether and how signalling storm can be prevented and/or mitigated based on the inputs provided by NWDAF.

NOTE 1: In terms of data access right, privacy and security improvement, cooperation with SA WG3 is needed.

NOTE 2: The study of this key issue will consider the study/work done by SA WG5 and CT WG4 in this regard already and collaborate with SA WG5/CT WG4 regarding the handling of abnormal network behaviours.
