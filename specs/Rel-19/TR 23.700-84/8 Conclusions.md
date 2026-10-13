---
spec: TR 23.700-84
version: 19.0.0
release: '19'
clause: 8
title: 8 Conclusions
source_archive: 23700-84-j00.zip
source_document: 23700-84-j00.docx
content_origin: 3gpp-source
---

# 8 Conclusions


## 8.1 Conclusions for KI#1: Enhancements to LCS to support Direct AI/ML based Positioning

The conclusions for supporting Direct AI/ML based Positioning is based on the following principles:

**Principle \#1:** The LMF is enhanced to perform location calculation based on ML model. The interaction between LMF and MTLF is only for model provisioning and possibly, based on coordination with RAN WGs, for data collection for training.

NOTE 1: When receiving the request for a UE location, the LMF selects an appropriate method to determine the UE location and may select AI/ML positioning as method.

**Principle \#2:** LMF is enhanced to perform model training for AI/ML based Positioning, the trigger for data collection and for model training in LMF is up to implementation.

**Principle \#3:** MTLF is enhanced to perform model training for AI/ML based Positioning.

**- Principle \#3.1:** To retrieve a model to perform location calculation, the LMF discovers a suitable MTLF via NRF.

**- Principle \#3.2:** NWDAF containing MTLF trains ML model for AI/ML based Positioning based on request from the LMF or internal trigger.

**- Principle \#3.3:** MTLF collects training data (e.g. data as defined by RAN for AI/ML based Positioning and historical data stored in ADRF) from data sources.

NOTE 2: Whether LMF can also retrieve a trained model from OAM or not will be discussed in normative phase and need to coordinate with SA5.

**Principle \#4:** LMF or MTLF performs model performance monitoring for AI/ML based Positioning.

**- Principle \#4.1** The result of model performance monitoring may trigger the LMF to change the positioning method, e.g. from AI/ML based positioning to the legacy positioning, or vice versa.

**- Principle** **\#4.2** The result of model performance monitoring may trigger ML model retraining in the training entity.

**- Principle** **\#4.3** The detailed procedure and service for this model performance monitoring will be decided in normative phase.

**Principle \#5:** Data used for model training, inference and model performance monitoring for AI/ML based positioning will be decided by RAN WGs, and SA WG2 will align with RAN WGs. The related procedures for data collection will be coordinated with RAN WGs in the normative phase.

NOTE 3: The user's authorization/consent of collecting UE related training data is needed as specified in TS 23.288 \[5\] and TS 23.273 \[7\].

NOTE 4: Whether and how the NF profile of the LMF at the NRF will be extended to support selection of an LMF that supports AI/ML based Positioning will be discussed in normative phase.

## 8.2 Conclusions for KI#2: 5GC Support for Vertical Federated Learning

**P#2.1: VFL related new functionalities include:**

P#2.1.1 VFL Server: An NWDAF or AF that integrates local training results for the local ML model update in VFL training process. It also coordinates the VFL training process by discovering and selecting VFL clients. In VFL inference process, The VFL server aggregates local inference results from VFL clients to generate the final VFL inference result and sends the final VFL inference result to the consumer. Only one VFL server exists for each VFL process.

P#2.1.2 VFL Client: An NWDAF or AF that holds the local dataset and performs local training and inference as asked by VFL Server. There can be multiple VFL Clients in VFL training and inference.

P#2.1.3 VFL process is associated with an analytics ID (i.e. VFL training of models for analytics ID, VFL inference for an analytics ID) when VFL server is the NWDAF.

P#2.1.4 VFL process is associated with internal AF process (i.e. VFL training of models for internal AF process, VFL inference for internal AF process) when VFL server is AF.

P#2.1.5: 5GC shall support vertical federated learning (VFL), i.e. a federated learning technique without exchanging/sharing local data set or ML models, in the following scenarios:

\- VFL among NWDAFs in a single PLMN.

\- VFL between AF and NWDAF(s) in a single PLMN.

The NWDAFs as VFL Server determines based on internal logic and the operator's policy whether or not to use VLF to provide a particular analytics ID.

P#2.1.6: NWDAF and AF are the only NFs that may act with any of the above VFL functionalities, i.e. VFL Server and VFL client.

**P#2.2: For registration and discovery of VFL entities:**

P#2.2.1 The NWDAF as VFL client shall register to NRF with NF profile including VFL capability information (VFL capability type (i.e. VFL Clients) ). For an untrusted AF as the VFL client, the NEF registers based on configuration at the NRF within its NF profile information about the AF as specified in clause 6.2.2.3 of TS 23.288 \[5\] and includes as part of the information about the AF an VFL capability information (VFL capability type (i.e. VFL Clients) ).

P#2.2.2 The NWDAF as VFL server will select NWDAF(s) as candidate VFL client(s) and /or AF(s) (via NEF profile for untrusted AF) from NRF for VFL training process.

P#2.2.3 The AF (via NEF in case of untrusted AF) as VFL server will select candidate NWDAF(s) as VFL client(s) from NRF for VFL training process.

NOTE 1: Whether and how Vendor specific feature information in P#2.2.1 is included will be determined in the normative phase.

**P#2.3: For sample alignment for VFL:**

P#2.3.1: For NWDAF acting as VFL Server, NWDAF triggers sample alignment, may query NWDAFs or AFs acting as VFL clients to query availability of samples, and generates the interclause of samples.

P#2.3.2 In case of VFL between the NWDAF as VFL server and AF as VFL client, the NEF may be involved in sample alignment with mapping of information (e.g. internal versus external information) , then NWDAF as VFL server will determine the final list of participants supporting the samples for VFL training.

P#2.3.3: For AF acting as VFL Server and NWDAFs acting as VFL clients, AF performs sample alignment, selects samples to be used in the training process and generates the interclause of samples.

P#2.3.4: Feature description information may be registered to the NRF or locally configured in the NWDAF or the AF, negotiated between VFL server and VFL client when performing feature alignment. Feature alignment is optional in a VFL process.

NOTE 2: Whether and how the feature alignment is supported will be determined in normative phase.

NOTE 3: The procedure to perform sample alignment is not agreed yet, i.e. it can be done in a standalone procedure or part of the preparation phase, which will be determined in normative phase.

NOTE 4: How NEF assists sample alignment in P#2.3.3 will be determined in normative phase.

NOTE 5: For P#2.3.2 and P#2.3.3, whether and how NEF supports to be involved in generating the intersection of candidate samples is to be discussed in normative phase.

**P#2.4: For VFL training process:**

P#2.4.1: Either the NWDAF or the AF can act as VFL server and initiate VFL training process with the VFL client(s).

P#2.4.2: If an untrusted AF is involved in the VFL training process, their interactions with the NWDAF(s) are via NEF. When the NWDAF acts as VFL Server, the NWDAF can receive labels from an AF.

P#2.4.3: An identifier is allocated by a VFL server, which is used to correlate the participants during the VFL training and subsequent VFL inference processes and it is associated with the distributed ML Models in the VFL joint model training process.

P#2.4.4: VFL Clients compute the intermediate results for their local ML models involved in the VFL training and provide reports with the intermediate results to the AF or NWDAF acting as VFL server.

P#2.4.5: VFL clients may also provide intermediate results (e.g. gradient information, loss information) to other VFL clients as instructed by the VFL server.

P#2.4.6: An AF or NWDAF acting as VFL server aggregates intermediate results from VFL client(s), trains a local model, computes intermediate results based on its local ML model, and sends the intermediate results towards VFL clients involved in the joint VFL training process.

P#2.4.7: VFL server may compute different intermediate training information (e.g. gradient information, loss information) for updating its own local model and the models of VFL clients during the VFL training process after processing the received intermediate results (that may include convergence reports), sends the updates to the VFL client(s), and the VFL server/client(s) update their local ML model based on the received information. The VFL server determines when the VFL training process terminates, then it will inform the VFL Clients that the training ends.

NOTE 6: Whether further specification on how the AF provides the label to NWDAF is needed in P#2.4.2 will be discussed in normative work.

NOTE 7: Whether and how P#2.4.5 is supported will be determined in normative phase.

NOTE 8: How the initial model is provided to VFL clients is to be discussed in normative phase.

**P#2.5: For VFL inference process:**

P#2.5.1: NWDAF acting as VFL server scenario, the NF consumer of analytics will obtain the required output based on the VFL inference process coordinated by VFL server, which is generated via the VFL inference process between VFL sever and corresponding AFs or NWDAF acting as VFL client(s):

\- For NWDAF acting as VFL server, NWDAF triggers the VFL inference phase after receiving the subscription or request for analytics and delivers the analytics to the NF consumer.

P#2.5.2: For the AF acting as a VFL server, the AF acting as VFL server can also start an inference process with corresponding NWDAF acting as VFL client(s). The AF may be triggered to start the inference by a 5GC consumer (i.e. NWDAF containing AnLF). Interactions are via NEF if AF is untrusted.

P#2.5.3: Before performing the VFL inference, the NWDAF acting as VFL server determines corresponding AF(s) and/or NWDAF(s) as VFL client(s) for the inference process based on the same identifier used in the VFL training process. Related details will be specified in the normative phase.

P#2.5.4: The VFL inference process may be controlled by a set of requirements, i.e. whether the determination if all VFL participants associated with the same identifier are needed in the VFL inference process may be based e.g. on accuracy requirements, the VFL signalling and load cost, contribution weights of each client, and temporal availability of output from VFL participants.

P#2.5.5: When performing VFL model performance monitoring, inference data can be used for model retraining, which is aligned with R18.

NOTE 9: Whether and how P#2.5.4 is supported will be determined in normative phase.

NOTE 10: Whether the NWDAF supports both AnLF and MTLF or not during VFL training and inference is to be discussed in normative phase.

NOTE 11: How multiple NWDAFs are involved in VFL when AF is acting as VFL server is defined in normative phase.

NOTE 12: The details of the accuracy monitoring related to VFL process will be defined in the normative phase.

NOTE 13: The details of (optionally new) services and detailed list of parameters to enable the VFL processes will be defined in the normative phase.

NOTE 14: Details of the ML model storage will be defined in the normative phase.

## 8.3 Conclusions for KI#3: NWDAF-assisted policy control and QoS enhancement

Following conclusions are proposed for KI#3:

Based on PCF request, NWDAF may provide assistance information analytics to PCF to assist the QoS determination and/or modification of QoS parameters. The PCF may take the assistance information into account to determine QoS parameters for a QoS flow, based on the operator's policy.

After the PCF determines the QoS parameters, if updated assistance information is received from NWDAF (the updated assistance information may include candidate QoS parameters, the analytics of the duration and usage of established QoS Flows), the PCF may determine to modify the QoS parameters for a QoS flow and provide the modified QoS towards the SMF, based on the operator's policy.

In particular:

\- The PCF may send a analytics request or subscription to NWDAF for QoS and policy assistance information analytics. The PCF may include an analytics ID, Filter Information, one or multiple sets of QoS parameters, the candidate value list(s) for each QoS parameter, requesting expected service experience (e.g. QoE), etc.

NOTE 1: Details of the input parameters in the PCF request will be determined in normative stage.

NOTE 2: Whether to introduce a new analytics ID or reuse existing one will be determined in normative stage.

\- Based on the PCF request, the QoS and policy assistance information analytics provided by NWDAF may include: expected QoE can be achieved by the candidate QoS parameters (the values of the QoS parameters are within the candidate value list provided by the PCF).

NOTE 3: Whether the time window(s) associated with QoS parameters can be provided by PCF and can be then used to determine the candidate QoS will be discussed in the normative work.

NOTE 4: Whether to introduce new or reuse existing service/ service operation for providing candidate QoS will be determined in normative phase.

\- The PCF may request one or more analytics from the NWDAF that are used within the same analytics target period, i.e. Observed service experience, QoS sustainability, Network Performance analytics, analytics of the duration and usage of the established QoS Flows.

NOTE 5: Possible enhancements to the existing analytics and/or introducing new analytics will be determined in the normative phase.

NOTE 6 Details of the input data and corresponding data sources for analytics will be determined in normative stage.

NOTE 7: To provide the assistance for policy control and QoS, NWDAF shall not be aware of PCF internal logic.

NOTE 8: NWDAF can monitor the accuracy performance of the assistance information analytics based on a request from PCF. Whether to reuse the existing procedures for Analytics Accuracy Monitoring (defined in clause 6.2D of TS 23.288 \[5\]) or not can be decided in the normative phase.

## 8.4 Conclusions for KI#4: NWDAF-assisted Network Abnormal Behaviour Mitigation and Prevention

It is agreed to use following principles as baseline for normative work.

\- By means of statistics and predictions, NWDAF supports assistance to signalling storm mitigation and prevention by statistically learning the expected normal level of signalling and identifying a significant deviation indicating a signalling storm caused by massive signalling of UEs and/or caused by NFs signalling.

\- A new Analytics ID to support Signalling storm Mitigation and Prevention caused by NFs signalling will be defined.

\- It will be determined in the normative phase whether to define a new Analytics ID or enhance an existing Analytics ID to support Signalling storm Mitigation and Prevention caused by massive signalling of UEs.

\- Analytics Filter includes information to assist the NWDAF for the signalling storm as defined above.

NOTE 1: The consumers of the analytics ID and the parameters of the analytics request/subscribe (determined by the consumer) are to be specified in normative work.

\- The NWDAF identifies target NFs to obtain data based on the Analytics Filter.

\- The following Input Data are agreed to be collected from NF, MDAF, AF and/or via NRF to support generation of the analytics by NWDAF:

\- Information related to control plane signalling exchange, including:

\- Request Type of control messages (and related posterior messages), message counts, success and failures, failure cause, fault information, response times for N1 and/or N2 interfaces and 5GC SBIs, taking information as described in clause 6.35.2.2 and table 6.39.1-1.

\- Information on NF contexts, including:

\- UE related number, state transition and timer information of a group of UEs as described in clause 6.35.2.2 and table 6.39.1-1;

\- NF load, status & resource usage as described in table 6.39.1-1.

\- UE related data per group, including:

\- input data of existing UE related analytics.

\- MDAF data, including:

\- 5GC Control Plane congestion status/level and/or aggregated data.

\- AF data, including:

\- user active/inactive information as described in table 6.36.1-2.

NOTE 2: The details of input data (including source and detailed data description) are to be specified in normative work. Input data already available in earlier releases are re-used.

NOTE 3: The NWDAF collects UE related data via NFs (e.g. AMF, etc.) as specified in TS 23.288 \[5\].

\- In order to reduce signalling overhead, offline bulk data report (clause 6.2.6.1 in TS 23.288 \[5\]) for UE related data collection may be used.

\- The following Output Data from NWDAF are agreed to be included in Analytics output:

\- Statistics Analytics, including:

\- Target NF ID, cause of the signalling storm (i.e. signalling storm caused by UEs and signalling storm caused by NF), NF lists or a group of UEs, statistics corresponding to the input data.

\- Prediction Analytics, including:

\- Target NF ID, cause of the signalling storm, NF lists or a group of UEs, prediction corresponding to the input data, confidence level of the prediction.

NOTE 4: Whether more IEs are needed will be discussed and specified in normative work.

\- Examples of mitigation or prevention for NFs will be discussed and provided in the normative work, considering a similar format as in table 6.7.5.3-3 in TS 23.288 \[5\] as a basis, the final mitigation or prevention operations are based on operator's policy/configuration and NF implementation. The mitigation and prevention described in Solutions \#35 and \#39 will be consider for used as a basis for such examples:

\- UE-related configurations (back-off timers, thresholds, slice or priority based overload control, etc.);

\- NF-related configuration ((re)discovery/selection, prioritizations, threshold, etc.).

NOTE 5: Examples of prevention/mitigation actions based on NWDAF analytics on signalling storm prediction and/or detection may be further refined during the course of the normative work.

NOTE 6: Whether SCP entity itself can be a source of input data will be discussed and specified in normative work.
