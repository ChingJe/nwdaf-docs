---
spec: TS 23.288
version: 18.13.0
release: '18'
clause: '5'
title: 5 Network Data Analytics Functional Description
source_archive: 23288-id0.zip
source_document: 23288-id0.docx
source_archive_sha256: 7116a1f4123dcf949818ab56feb6b242eda8819877346ab045654e1d0d8c36aa
source_document_sha256: 5e4ca6e4e350f62a5d4e4443d36ae93c9df679e1e9aea20ccaa44a4b0eb3e196
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 5 Network Data Analytics Functional Description


## 5.1 General


The NWDAF provides analytics to 5GC NFs and OAM as defined in clause 7. An NWDAF may contain the following logical functions:

\- **Analytics logical function (AnLF):** A logical function in NWDAF, which performs inference, derives analytics information (i.e. derives statistics and/or predictions based on Analytics Consumer request) and exposes analytics service i.e. Nnwdaf_AnalyticsSubscription or Nnwdaf_AnalyticsInfo.

\- **Model Training logical function (MTLF):** A logical function in NWDAF, which trains Machine Learning (ML) models and exposes new training services (e.g. providing trained ML Model) as defined in clause 7.5 and clause 7.6.

NOTE 1: NWDAF can contain an MTLF or an AnLF or both logical functions.

NOTE 2: Pre-trained ML Model storage and provisioning to NWDAF is out of the scope of 3GPP.

Analytics information are either statistical information of the past events, or predictive information.

Different NWDAF instances may be present in the 5GC, with possible specializations per type of analytics. The capabilities of a NWDAF instance are described in the NWDAF profile stored in the NRF.

To guarantee the accuracy of analytics output for an Analytics ID, based on the UE abnormal behaviour analytics from itself or other NWDAF including abnormal UE list and the observed time window, the NWDAF is to detect and may delete the input data from the abnormal UE(s) and then may generate a new ML Model and/or analytics outputs for the Analytics ID without the input data related to abnormal UE list during the observed time window and then send/update the ML Model Information and/or analytics outputs to the subscribed NWDAF service consumer.

In order to support NFs to discover and select an NWDAF instance containing MTLF, AnLF, or both, that is able to provide the required service (e.g. analytics exposure or ML Model provisioning) for the required type of analytics, each NWDAF instance should provide the list of supported Analytics IDs (possibly per supported service) when registering to the NRF, in addition to other NRF registration elements of the NF profile. NFs requiring the discovery of an NWDAF instance that provides support for some specific service(s) for a specific type of analytics may query the NRF for NWDAFs supporting the required service(s) and the required Analytics ID(s).

The consumers, i.e. 5GC NFs and OAM, decide how to use the data analytics provided by NWDAF.

The interactions between 5GC NF(s) and the NWDAF take place within a PLMN.

The NWDAF has no knowledge about NF application logic. The NWDAF may use subscription data but only for statistical purpose.

The NWDAF architecture allows for arranging multiple NWDAF instances in a hierarchy/tree with a flexible number of layers/branches. The number and organisation of the hierarchy layers, as well as the capabilities of each NWDAF instance remain deployment choices.

In a hierarchical deployment, NWDAFs may provide data collection exposure capability for generating analytics based on the data collected by other NWDAFs, when DCCF, MFAF are not present in the network.

In order to make NWDAF discoverable in some network deployments, NWDAF may be configured (e.g. for UE mobility analytics) to register in UDM (Nudm_UECM_Registration service operation) for the UE(s) it is serving and for the related Analytics ID(s). Registration in UDM should take place at the time the NWDAF starts serving the UE(s) or collecting data for the UE(s). Deregistration in UDM takes place when NWDAF deletes the analytics context for the UE(s) (see clause 6.1B.4) for a related Analytics ID.

NOTE 3: The procedures for data collection for UE related analytics need to take user consent into account. The user consent for analytics is defined in clause 6.2.9.

## 5.2 NWDAF Discovery and Selection


The NWDAF service consumer selects an NWDAF that supports requested analytics information and required analytics capabilities and/or requested ML Model Information by using the NWDAF discovery principles defined in clause 6.3.13 of TS 23.501 \[2\].

Different deployments may require different discovery and selection parameters. Different ways to perform discovery and selection mechanisms depend on different types of analytics/data (NF related analytics/data and UE related analytics/data). NF related refers to analytics/data that do not require a SUPI nor group of SUPIs (e.g. NF load analytics). UE related refers to analytics/data that requires SUPI or group of SUPIs (e.g. UE mobility analytics).

In order to discover an NWDAF containing AnLF using the NRF:

\- If the analytics is related to NF(s) and the NWDAF service consumer (other than an NWDAF) cannot provide an Area of Interest for the requested data analytics, the NWDAF service consumer may select an NWDAF with large serving area from the candidate NWDAFs from discovery response. Alternatively, in case the consumer receives NWDAF(s) with aggregation capability, the consumer preferably selects an NWDAF with aggregation capability with large serving area.

NOTE 1: If the selected NWDAF cannot provide the requested data analytics, e.g. due to the NF(s) to be contacted being out of serving area of the NWDAF, the selected NWDAF might reject the analytics request/subscription or it might query the NRF with the service area of the NF to be contacted to determine another target NWDAF.

\- If the analytics is related to UE(s) and the NWDAF service consumer (other than an NWDAF) cannot provide an Area of Interest for the requested data analytics, the NWDAF service consumer may select an NWDAF with large serving area from the candidate NWDAFs from discovery response. Alternatively, in case the consumer receives NWDAF(s) with aggregation capability, the consumer preferably selects an NWDAF with aggregation capability with large serving area.

NOTE 2: If a selected NWDAF cannot provide analytics for the requested UE(s) (e.g. the NWDAF serves a different serving area), the selected NWDAF might reject the analytics request/subscription or it might determine the AMF serving the UE as specified in clause 6.2.2.1, request UE location information from the AMF and query the NRF with the tracking area where the UE is located to discover another target NWDAF serving the area where the UE(s) is located.

\- If the analytics are related to UE(s) and if NWDAF instances indicate weights for TAIs in their NF profile (see clause 6.3.13 of TS 23.501 \[2\]), the NWDAF service consumer may use the weights for TAIs to decide which NWDAF to select.

\- If the NWDAF service consumer needs to discover an NWDAF containing an AnLF with analytics accuracy checking capability, the consumer may query NRF providing also the analytics accuracy checking capability in the discovery request.

\- If the NWDAF service consumer needs to discover an NWDAF containing AnLF which can use the Model provided by the specific NWDAF containing MTLF (e.g., in case of analytics context transfer), the consumer should discover the NWDAF(s) whose vendor ID is in the ML Model Interoperability indicator of the NWDAF containing MTLF.

If the NWDAF service consumer needs to discover an NWDAF that is able to collect data from particular data sources identified by their NF Set IDs or NF types or to collect data from particular NWDAF Serving Area, the consumer may query NRF providing the NF Set IDs or NF types or Area of Interest in the discovery request.

NOTE 3: The NF Set ID or NF Type of a data source serving a particular UE, can be determined as indicated in Table 5A.2-1.

In order to discover an NWDAF that has registered in UDM for a given UE:

\- NWDAF service consumers or other NWDAFs interested in UE related data or analytics, if supported, may make a query to UDM to discover an NWDAF instance that is already serving the given UE.

If an NWDAF service consumer needs to discover NWDAFs with data collection exposure capability, the NWDAF service consumer may discover via NRF the NWDAF(s) that provide the Nnwdaf_DataManagement service and their associated NF type of data sources or their associated NF Set ID of data sources or NWDAF Serving Area information as defined in clause 6.3.13 of TS 23.501 \[2\].

In order to discover an NWDAF containing MTLF via NRF:

\- When one or more trained ML Models are available for one or more Analytics ID(s) the NWDAF containing MTLF shall include the Analytics ID(s) that is(are) supported per service in the registration towards NRF. The NWDAF containing MTLF may wait to register in NRF the above services until at least one trained model is available. The NWDAF containing MTLF may provide to the NRF a list of Analytics IDs corresponding to the trained ML Models and possibly the ML Model Filter Information for the trained ML Model per Analytics ID(s), if available. In this Release of the specification, only the S-NSSAI(s) and Area(s) of Interest from the ML Model Filter Information for the trained ML Model per Analytics ID(s) may be registered into the NRF during the NWDAF containing MTLF registration. If the NWDAF containing MTLF supports ML Model interoperability, the NWDAF containing MTLF includes, in the registration to the NRF, an ML Model Interoperability indicator for each Analytics ID.

\- The ML Model Interoperability indicator comprises a list of NWDAF providers (vendors) that are allowed to retrieve ML Models from this NWDAF containing MTLF. It also indicates that the NWDAF containing MTLF supports the interoperable ML Models requested by the NWDAFs from the vendors in the list.

NOTE 4: The S-NSSAI(s) and Area(s) of Interest from the ML Model Filter Information are within the indicated S-NSSAI and NWDAF Serving Area information in the NF profile of the NWDAF containing MTLF, respectively.

\- During the discovery of NWDAF containing MTLF, a consumer (e.g. an NWDAF containing AnLF, an NWDAF containing MTLF as FL server or FL client) may include in the request the target NF type (i.e. NWDAF), the Analytics ID(s), the S-NSSAI(s), Area(s) of Interest of the Trained ML Model required and Vendor ID. The NRF returns one or more candidate instances of NWDAF containing MTLF to the NF consumer and each candidate instance of NWDAF containing MTLF includes the Analytics ID(s), possibly the ML Model Filter Information for the available trained ML Models and ML Model Interoperability indicator, if available.

\- If the NWDAF service consumer needs to discover an NWDAF containing an MTLF with ML Model accuracy checking capability, the consumer may query NRF also providing the ML Model accuracy checking capability in the discovery request.

In order to discover an NWDAF containing MTLF with Federated Learning (FL) capability via NRF, in addition to the procedures described above for discovering NWDAF containing MTLF:

\- An NWDAF containing MTLF supporting FL as a server shall additionally include FL capability type (i.e. FL server) and may include Time interval supporting FL as FL capability information during the registration in NRF.

\- An NWDAF containing MTLF supporting FL as a client shall additionally include FL capability type (i.e. FL client) and may include Time interval supporting FL as FL capability information during the registration in NRF, and it may also include, NF type(s) and NWDAF Serving Area information and/or NF set ID(s) of the data source(s) where data can be collected as input for local model training.

NOTE 5: An NWDAF containing MTLF may indicate to support both FL server and FL client in the FL capability for specific Analytics ID.

\- During the discovery of NWDAF containing MTLF as FL server, a consumer (e.g. a NWDAF containing MTLF) may include in the request the FL capability type as FL server and may include Time Period of Interest and ML Model Filter information for the trained ML Model(s) per Analytics ID(s), if available. The NRF returns one or more NF profiles of candidate instances of NWDAF satisfying the query parameters.

\- During the discovery of NWDAF containing MTLF as FL client, a consumer (e.g. an FL server) may include in the request FL capability type as FL client and may include Time Period of Interest, a list of NF type(s) and/or NF set ID(s) of the data source(s). The NRF returns one or more NF profiles of candidate instances of NWDAF satisfying the query parameters.

NOTE 6: The service consumer to discover an NWDAF containing MTLF with FL capability is limited to NWDAF containing MTLF in this Release.

A PCF may learn which NWDAFs being used by AMF, SMF and UPF for a specific UE, via signalling described in clause 4.16 of TS 23.502 \[3\]. This enables a PCF to select the same NWDAF instance that is already being used for a specific UE.

In the roaming architecture, the NWDAF with roaming exchange capability (RE-NWDAF) to request analytics or input data is discovered via the NRF. A consumer in the same PLMN as the RE-NWDAF discovers the RE-NWDAF(s) by querying for NWDAF(s) where the roaming exchange capability is indicated in its (their) NF profile. A consumer in a peer PLMN (i.e. RE-NWDAF) discovers the RE-NWDAF(s) by querying for NWDAF(s) in the target PLMN that is (are) supporting the specific services defined for roaming. A RE-NWDAF discovers the RE-NWDAF(s) in a different PLMN (i.e. HPLMN or VPLMN) using the procedure defined in clause 4.17.5 (if delegated discovery is not used) or clause 4.17.10 (if delegated discovery is used) of TS 23.502 \[3\], where the detailed parameters are determined based on the analytics request or subscription from the consumer 5GC NF, operator policy, user consent and/or local configuration.

## 5.3 Federated Learning (FL) among multiple NWDAFs


NOTE 1: In this Release of the specification, Federated Learning (FL) only refers to Horizontal Federated Learning.

Federated learning among multiple NWDAFs is a machine learning technique in core network that trains an ML Model across multiple decentralized entities holding local data set, without exchanging/sharing local data set. This approach stands in contrast to centralized machine learning techniques where all the local datasets are uploaded to one server, thus allowing to address critical issues such as data privacy, data security, data access rights.

NOTE 2: Horizontal Federated Learning is supported among multiple NWDAFs, which means the local data set in different FL client NWDAFs have the same feature space for different samples (e.g. UE IDs).

For Federated Learning supported by multiple NWDAFs containing MTLF, there is one NWDAF containing MTLF acting as FL server (called FL server NWDAF for short) and multiple NWDAFs containing MTLF acting as FL client (called FL client NWDAF for short), the main functionality includes:

**FL server NWDAF:**

\- discovers and selects FL client NWDAFs to participant in an FL procedure

\- requests FL client NWDAFs to do local model training and to report local model information.

\- generates global ML Model by aggregating local model information from FL client NWDAFs.

\- sends the global ML Model back to FL client NWDAFs to perform an additional training iteration if needed.

**FL client NWDAF:**

\- locally trains ML Model as tasked by the FL server NWDAF with the available local data set, which includes the data that may not be allowed to be shared with other FL client NWDAFs due to e.g. data privacy, data security, data access rights.

\- reports the trained local ML Model information to the FL server NWDAF.

\- receives the global ML Model from FL server NWDAF and perform an additional training iteration if needed.

FL server NWDAF or FL client NWDAF register to NRF with their FL capability information as described in clause 5.2.

The NWDAF containing MTLF determines to train an ML Model either based on local configuration or when it receives a request from NWDAF containing AnLF. The NWDAF containing MTLF further determines whether the ML Model should be trained via FL mechanism based on Analytic ID, Service Area/DNAI or when data can not be obtained directly from data producer NF (e.g. due to data privacy, data security). The NWDAF containing AnLF is not aware whether the ML Model is trained based on FL or not.

If the NWDAF containing MTLF can act as an FL server for the ML Model training, then FL procedure is initiated by the NWDAF containing MTLF as FL server NWDAF directly.

If the NWDAF containing MTLF determines to train an ML Model based on local configuration and the FL mechanism is required, but the NWDAF containing MTLF can't act as an FL server, the NWDAF containing MTLF should discover an FL server NWDAF as described in clause 5.2 and request the FL server NWDAF to provide the trained ML Model as described in clause 6.2C.2.2. The FL server NWDAF may determine to initiate FL procedure before providing the ML Model.

If the ML Model training is triggered by the request from NWDAF containing AnLF, the NWDAF containing MTLF determines the FL mechanism is required but it can not act as an FL server, the NWDAF containing MTLF should discover an FL server NWDAF as described in clause 5.2 and request the FL server NWDAF to provide the trained ML Model as described in clause 6.2C.2.2. The Notification Target Address and the Notification Correlation ID from the NWDAF containing AnLF is provided in the request message sent to the FL server NWDAF. The FL server NWDAF may determine to initiate FL procedure before providing the ML Model. The FL server NWDAF sends the ML Model information to the notification endpoint (e.g. the NWDAF containing AnLF) after the ML Model training success.

NOTE 3: The security procedure on authorizating FL server to initiate FL procedure on the FL client(s) is described in Annex X, clause X.9 of TS 33.501 \[49\]. The security procedure authorizing an MTLF to request ML Models on behalf of an AnLF to another MTLF (e.g., FL server NWDAF) is described in Annex X, clause X.10 of TS 33.501 \[49\].

Before FL procedure is initiated by FL server NWDAF, appropriate FL client NWDAFs should be discovered by FL server NWDAF as described in clause 5.2.

When starting an FL procedure, the FL server NWDAF is to provide an initial model to each FL client NWDAF, and then each FL client NWDAF is to perform local model training using its local data set. The detailed procedure for FL among Multiple NWDAFs is described in clause 6.2C.
