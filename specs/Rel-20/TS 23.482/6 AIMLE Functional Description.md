---
spec: TS 23.482
version: 20.3.0
release: '20'
clause: 6
title: 6 AIMLE Functional Description
source_archive: 23482-k30.zip
source_document: 23482-k30.docx
content_origin: 3gpp-source
---

# 6 AIMLE Functional Description


## 6.1 Support for ML model retrieval

This functionality covers the retrieval of ML models by an AIMLE client or a VAL server via the AIMLE server. The AIMLE server provides the functionality required to retrieve ML models stored in a ML repository based on a filtering criteria. The functionality is provided to the ML model consumer via an API following a request/response model or a subscribe/notify model.

## 6.2 Support for ML model training

This functionality enables the AIMLE server to support ML model training based on requests from a VAL server. The AIMLE server provides the functionality to train a specific ML model or assist the ML model training at VAL server/client(s).

## 6.3 Support for FL member registration

This functionality covers the registration, registration update and de-registration of the candidate FL member to the ML repository which is keeping the FL member registrations. Such candidate member can be a VAL server functionality or an enabler layer functionality (e.g. AIMLE server) which is registering to the ML repository/registry to act as FL member for a given application event (analytics event or event triggered by a VAL layer application server).

If the candidate FL member is a VAL server, this functionality covers the registration, registration update, and deregistration of the VAL server to the AIMLE server.

## 6.4 Support for FL events subscription and notification

This functionality enables a consumer (who can be the AIMLE server or a VAL server e.g. acting as FL server) to subscribe for FL related events and getting notified on changes on the availability of the FL members which are to be used for the FL-related task (e.g., training). This capability at the ML repository acting as an AIML service registry supports the subscription for events related to FL members and the notification to the consumer in case of changes. This feature assumes that such FL members (AIMLE or VAL server or AIMLE clients) have previously registered to this registry their availability and capabilities.

## 6.5 Support AI/ML task transfer

This functionality covers the AIMLE support for ML task transfer, which is applicable to scenarios where an AI/ML member cannot finish the assigned AI/ML task during the performing process. In this scenario, the AIMLE server assists the source AI/ML member by support transferring the intermediate AI/ML information (e.g., the intermediate AI/ML operation status and results) to another AI/ML member (target AI/ML member) for further operations to complete the AI/ML task.

## 6.6 Support for AIMLE client registration

This functionality allows an AIMLE client (e.g., AI/ML capable UEs) to register with an AIMLE server. The AIMLE server stores the client information for future interactions. This functionality is crucial for enabling the AIMLE client to participate in AI/ML operations.

## 6.7 Support for AIMLE client discovery

This functionality enables the VAL server to discover available AIMLE clients for AI/ML operations, such as training or inference. The AIMLE server provides the functionality to select suitable AIMLE clients that fulfill the discovery criteria.

NOTE: AIMLE client discovery can be also used to discover VAL clients associated with the AIMLE clients. The VAL client may integrate the AIMLE clients as part of the VAL client software.When permitted by policy and federation agreement, the AIMLE server may additionally support discovery of AIMLE clients in another domain/PLMN by interacting with a partner AIMLE server, and may provide a consolidated discovery result.

## 6.8 Support for AIMLE client selection

This functionality enables the selection of AIMLE clients to participate in AI/ML operations. There are two modes for client selection: VAL server selection and AIMLE server selection. In VAL server selection, the functionality is provided by the AIMLE Server selecting candidate AIMLE clients from the client list provided by the VAL Server. In AIMLE server selection, the functionality is provided by the AIMLE Server selecting candidate AIMLE clients based on the client selection criteria provided by the VAL Server from the AIMLE clients in the ML repository.

When permitted by policy and federation agreement, the AIMLE Server may support selection of AIMLE clients in another domain/PLMN by interacting with a partner AIMLE Server via AIML-E.

## 6.9 Support for AIMLE client participation

This functionality enables the AIMLE server to verify and manage the participation of AIMLE clients in AI/ML operations. The AIMLE client responds with its willingness to perform AI/ML operations based on the information provided by the AIMLE server.

## 6.10 Support for ML model management

This functionality enables the AIMLE server to manage ML models through interaction with the ML repository. The AIMLE server provides the storage and discovery functionality for ML model information. The storage functionality allows the AIMLE server to store ML model information in the repository. The discovery functionality enables the AIMLE server to retrieve information about available ML models via an API following a request/response model.

## 6.11 Support HFL training

This functionality provides the AIMLE server support for horizontal federated learning. This support is applicable to the case where multiple AIMLE clients are expected to locally train the ML model, and the AIMLE server is required to select, configure and coordinate the HFL clients.

## 6.12 Support AIMLE client selection subscription and notification

This functionality is related to the AIMLE client selection subscription request and notification to enable VAL Servers to subscribe for monitoring AIML members who meet criteria for performing an AIMLE service, selecting AIMLE clients (associated with the VAL clients) and receiving notification when there is an update on the selected and re-selected AIML client’s status when re-selection is performed according to AIML member selection criteria.

NOTE: The AIMLE client selection subscription and notification to VAL server is also used for the case where VAL server needs to find some VAL clients (integrating AIMLE client), which can be used by VAL server to request certain AI task operation.

When permitted by policy and federation agreement, this functionality may support monitoring of selected AIMLE clients across domains/PLMNs. If selected AIMLE clients are located in another domain/PLMN, the AIMLE server may establish a selection subscription with a partner AIMLE server in that domain/PLMN to receive notifications on the status of the selected/de-selected AIMLE clients. Based on local monitoring and/or notifications from the partner AIMLE server, the AIMLE server may perform re-selection, including replacement of remote-domain AIMLE clients with local-domain AIMLE clients when available. The AIMLE server may provide the VAL server with consolidated notifications of the selected/de-selected AIMLE clients.

## 6.13 Support for Split AI/ML Operation

Split AI/ML operation is a type of AI/ML operation that allows distributed processing related to ML models into multiple stages on different processing nodes. The intention is to offload the computation-intensive and energy-intensive AI/ML stages to network endpoints, whereas leave the privacy-sensitive and delay-sensitive stage at the end device as described in 3GPP TS 22.261 \[10\].

The following functionality is provided to support split AI/ML operation:

\- An application consuming services from the AI/ML application enablement layer can discover or manage (e.g., create, update, delete) a split operation profile with the AIMLE server for the purpose of consuming results from corresponding instance of a split AI/ML operation pipeline.

\- A VAL server can register with the AIMLE server to indicate its capabilities for acting as a processing node of an instance of a split AI/ML operation pipeline.- An application consuming services from the AI/ML application enablement layer can subscribe with the AIMLE server to receive event notifications related to an instance of a split AI/ML operation pipeline.

NOTE: How to split a ML model is out of scope of this release and the ML model used in a stage needs to be available in the ML repository.

## 6.14 Support data management assistance

This functionality covers the AIMLE assistance in data management related operations in the ML model lifecycle. AIMLE data management assistance is the process of the AIMLE server assisting AIMLE service consumers with managing data operations (data preparation and processing) performed by VAL clients.

## 6.15 Support for Transfer Learning enablement

This functionality covers the support for discovering and selecting pre-trained models transfer learning operations in application enablement layer, where the support is based on the request for either an ML task from VAL layer or for an analytics task from ADAES. Transfer Learning enablement allows the consumer to discover the similar ML models to be used as base models for the TL, as well as to support the selection of the best model to be used as pre-trained model.

## 6.16 Support for FL member grouping

This functionality covers the AIMLE capability to enable the group management of the entities serving as FL clients at the application enablement layer. Such group management is about the creation, monitoring and update of the FL member groups based on the AI/ML operations, which are based on 1) the analytics event/service by ADAES or 2) the VAL requirement for FL support services.

## 6.17 Support vertical federated learning

This functionality is related to the AIMLE support for vertical FL (VFL) among AIMLE clients serving as VFL members. This capability involves the determination of employing VFL based on the ML model training request, the feature alignment and the decision on the AIMLE clients serving as VFL clients, based on the training capability evaluation functionality.

## 6.18 Support for ML model training capability evaluation

This functionality enables AIMLE server to request AIMLE client for ML model training capability evaluation to support FL training (e.g. HFL, VFL). AIMLE client evaluates its capability and availability to join the FL training process and responds to the AIMLE server with the evaluation result (e.g. join the FL training process with test result, or not join with fail reason). The ML model training capability evaluation result can be used by the AIMLE server to select FL members for FL training process (e.g. HFL, VFL).

## 6.19 Support AIML service operations control and management

This functionality enables the VAL server (as AIMLE service consumer) to control the operation mode of an AIMLE service for a given AI/ML operation.

## 6.20 Support for ML model update

This functionality enables the AIMLE server to update trained and deployed ML models by detecting model performance degradation, and triggering ML model re-training and update to fix the observed degradation.

## 6.21 Support for ML model performance monitoring

This functionality covers a capability of AIMLE server for monitoring detecting a degradation relation to an ML operation / analytics operation and translating to an ML model degradation (expected or predicted) and performing an action to alleviate this issue (new model training or re-training).

## 6.22 Support for AIMLE assisted ML model selection

This functionality enables an AIMLE service consumer to request assistance with ML model selection from an AIMLE server. The AIMLE server returns a list of ML models with corresponding model performance.

## 6.23 Support for AIMLE context transfer in edge data networks

This functionality provides support for AIMLE operations performed by AIMLE clients spread across multiple edge service areas in edge data networks. The functionality allows the context transfer between edge AIMLE servers when an AIMLE client moves to a different edge service area.

## 6.24 Support for Assisting Hierarchical Computing

This functionality provides support for assisting hierarchical computing by AIMLE servers. The functionality allows the consumer to request hierarchical computing assistance information to support its operations.

## 6.25 Support for ML model evaluation information management

This functionality covers the management of ML models evaluation information by an AIMLE client or a VAL server via the AIMLE server. ML models evaluation information is associated with ML model(s) and is information required to enable evaluation of a ML model performance and correctness. The AIMLE server provides the functionality required to manage ML model evaluation information stored in a ML repository which facilitates consistent usage, retrieval and updates of such information independently from associated ML model(s). The functionality is provided to the ML model consumer via an API following a request/response model and is required for performing the evaluation and monitoring of performance and correctness for ML model(s).

## 6.26 Support for AIMLE server registration procedures in hierarchical AIMLE deployments

This functionality provides support for AIMLE server registration procedures in hierarchical AIMLE deployments. The functionality allows the consumer to request registration, update registration, and de-registration procedures to support hierarchical AIMLE deployments.

## 6.27 Support for ML model training in hierarchical AIMLE deployments

This functionality provides support for ML model training in hierarchical AIMLE deployments. The functionality allows a central AIMLE server to obtain support from edge AIMLE servers for ML model training. The central AIMLE server can request ML model training from multiple edge AIMLE servers to fulfill requirements for obtaining the required number of AIMLE clients or for requesting AIMLE clients with a particular dataset identifier.

## 6.28 Support for ML model split learning using relays

This functionality covers a capability at AIMLE for identifying and determining an intermediate entity (VAL UE) to serve as relay for a split learning operation, and particularly a split AI/ML operation. Such intermediate entity can be an application layer relay UE which either doesn’t provide any processing and is not aware of the data to relay (encrypted), or 2) intermediate node which undertakes some layers of the split operations (e.g. layers 5-15) instead of the source VAL UE.

## 6.29 Support for ML model maintenance

This functionality covers a capability of AIMLE server for ML model maintenance. The functionality enables consumer to request AIMLE server for supportting ML model performance maintenance to align with consumer’s expectations. It also enables the consumer to receive notifications periodically or event triggered on the ML model maintance. Such notifications can include e.g. updated ML model which performance is aligned with consumer's expectations, and/or ML model status (e.g. updated ML model ID, ML model version).

## 6.30 Support for ML model performance evaluation with information collection from ML model consumer

This functionality covers a capability of AIMLE server for evaluating ML model performance with information collected from ML model consumer(s), and reporting to consumer with aggregated information regarding the performance evaluation of the ML model.

## 6.31 Support for AIMLE service migration in UE roaming scenarios

This functionality covers a capability at AIMLE for supporting the continuity of the ML training / inference capability in scenarios where the ML model training / inference happens at the UE (e.g. AIMLE client) while the UE client is expected to move to an area served by another PLMN.

## 6.32 Support for federated AIML service enablement

This functionality covers a capability at AIMLE for supporting a federated AIML service, where the federated AIML service is defined as an AI/ML service which requires the joint operation of subsequent AI operations provided by different providers (MNO, ECSP, CSP, Device owner, App developer). The capability involves translating a request for an AI/ML service to a requirement for a federated AI service, given the resource conditions and the federation agreements in place.

## 6.33 Support for AIMLE server discovery

This functionality enables the VAL server or AIMLE clients to discover the available AIMLE servers for AI/ML operations, such as training or inference. The AIMLE server provides the functionality to discover and provide the suitable alternate AIMLE servers that fulfill the discovery criteria. The need for availing services from another AIMLE server may arise at AIMLE client/VAL server for various conditions like change in AIMLE client's ML task requirement, AIMLE client's mobility planning/requirements etc.

## 6.34 Support for ML model inference

This functionality enables an AIMLE service consumer (e.g., VAL server, AIMLE client) to request the AIMLE server to perform ML model inference based on a specified ML model (or ML model requirement information). The AIMLE server authenticates/authorizes the request, selects the suitable ML model and/or inference service instance as applicable, performs the inference operation, and returns the inference result (and/or endpoint information to retrieve the result) to the requestor.

This functionality is also supported in hierarchical AIMLE deployments scenario. In this case, a central AIMLE server can distribute inference tasks to multiple edge AIMLE servers for parallel execution and aggregate responses, enabling efficient use of distributed computing resources (e.g., offloading heavy or data-local computation to edge AIMLE servers).

This functionality also supports ML model collaborative inference between an AIMLE client and an edge AIMLE server. In this case, the AIMLE client performs part of ML model inference locally and provides intermediate inference output to the edge AIMLE server, and the edge AIMLE server performs the remaining part of the ML model inference and returns the final inference result to the AIMLE client.

## 6.35 Support for AIML Server discovery for ML inference service

This functionality enables an authorized AIMLE service consumer (e.g., VAL server) to discover available AIML Server discovery for ML inference service to invocation. The AIMLE server returns one or more candidate inference service instances (e.g., service identifier and service access information such as endpoint/interface profile, and optionally model-related information), so that the requestor can select an appropriate inference service instance without requiring internal knowledge of the AIMLE implementation. This functionality can be used for both centralized and edge deployment scenarios.

## 6.36 Support for AI inference service performance management and feedback

This functionality enables AI inference service performance management and feedback, where inference clients (e.g., VAL server, AIMLE client) can provide structured feedback/observations on inference execution (e.g., inference result too late, failure, degraded accuracy/confidence) to the AIMLE server. The AIMLE server validates and aggregates such feedback together with runtime observations, and can use aggregated information to optimize subsequent inference service discovery and selection decisions (e.g., updating service performance profiles or candidate ranking used in discovery responses).
