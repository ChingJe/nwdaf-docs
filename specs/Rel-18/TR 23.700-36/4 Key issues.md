---
spec: TR 23.700-36
version: 18.1.0
release: '18'
clause: 4
title: 4 Key issues
source_archive: 23700-36-i10.zip
source_document: 23700-36-i10.doc
content_origin: 3gpp-source
---

# 4 Key issues


## 4.1 Key issue #1: Support for application performance analytics

Data analytics related to the application end-to-end QoS and in particular statistics and predictions on the application server or application session performance and load can be useful for the application specific layer, so as to proactively identify potential adaptations of the application service and to trigger adaptations at the communication layer. One example is the utilization of analytics by the application specific layer e.g. for selecting the least loaded EAS for an application session, or for selecting the optimal PLMN for communicating the application service in a given area.

This key issue will study:

\- whether and how the application data analytics enablement service provides application QoS related analytics for the application session /service?

\- whether and how the application data analytics enablement service provides application QoS related analytics tailored for different communication means (e.g. different PLMNs, RATs, slices)?

\- what data needs to be collected from 3GPP system and application specific layer for performing application QoS related analytics?

\- how to enable the exposure of application QoS related analytics to the vertical / ASP in a unified manner?

## 4.2 Key issue #2: Support for edge analytics enablement

Edge deployments are vitally important for applications that require performance levels that cannot be met by existing cloud deployments. Edge data analytics may relate to stats/predictions on computational resources and expected/predicted load of the platform which hosts the edge applications and may be necessary to be exposed as a service to EAS (which can be either edge native applications or edge enhanced applications at a centralized cloud). Particularly, for edge native applications which need to be light designed and high portable, the use of edge analytics at the edge platform can help improving the application service operation.

The support for edge analytics at the enablement layer (related to the edge performance, failure, service availability), would be useful for the edge applications to allow for dynamically deciding to scale-in, scale-out, migrate from the edge to the cloud in heavy load situations, or migrate from the cloud to the edge to improve the quality of experience for the end user. Also, the need for edge application relocation between edge platforms could be supported by using analytics which can be leveraged by the EDGEAPP layer and could be exposed as a service to the application developer for supporting the edge service operation optimization.

Hence, in this key issue the following points shall be studied:

\- Whether and what edge data are needed to be collected by the application data analytics enablement server to allow for edge analytics enablement (related to the edge performance, failure, service availability)?

\- Whether and how the application data analytics enablement server (deployed in an edge or centralized data network) can be utilized by the corresponding Edge Enabler layer architecture (as specified in 3GPP TS 23.558 \[6\]) to optimize edge services?

\- Whether and how the analytics enablement layer needs to align with the EDGEAPP layer for allowing the edge services to utilize edge analytics enablement service to optimize their operation (e.g. triggering pro-active ACR based on edge analytics)?

\- Whether and how the application data analytics enablement server needs to align with ETSI MEC system to utilize MEC services?

## 4.3 Key issue #3: Support for data collection for application layer analytics

For deriving application layer analytics, the data collection may be provided by different sources (e.g. vertical-specific server, application of the UE, EAS, 3<sup>rd</sup> party server, SEAL/SEALDD) and it needs to be identified how these data can be collected to allow for stats/predictions by the analytics enablement layer.

The application data analytics enablement layer needs to be capable of receiving data from different data producers and prepare the data to be used for deriving analytics. Such data can be measurements or analytics from the 5GS (5GC, OAM), the applications of the VAL UEs, other application enablers etc.

For example, for application QoS related analytics, such data can be potentially derived by the OAM, monitoring of network QoS by 5GC, subscribing and receiving QoS and network analytics from NWDAF, performance data from the application server, QoS data from enabler layer client-server sessions, etc. The consumer of the ADAE service may not be aware of the data that need to be collected from different sources, however the ADAE needs to be capable of selecting the optimal sources to collect data, subscribe to different data producers and also retrieve supplementary data samples based on the data producers' availability and load.

Hence, this key issue will discuss the following open issues:

\- How to enable the collection and preparation of data at the application data analytics enablement service for data analytics derivation, when the data to be collected target the same performance metrics and are originated from different sources (UE, networking layer, application specific layer, non-3gpp domains)?

\- Whether and how the application data analytics enablement layer needs to collect data from multiple sources, at the DN side or locally at the VAL UE side?

\- Whether and how to leverage the UE data collection support provided by the SA4 EVEX study?

## 4.4 Key issue #4: Key Issue on interactions with SEAL services

SEAL is the service enabler architecture layer common to all vertical applications over 3GPP systems. It provides the functions like location management, group management, configuration management, identity management, key management, network resource management and network slice capability management as defined in 3GPP TS 23.434 \[5\].

This key issue will study:

\- the applicability of the usage of SEAL services for application data analytics enablement services considering different deployment and business models

\- whether any enhancements are required at the SEAL services for exposing data to the application data analytics enablement service?

\- whether and how application data analytics at the application data analytics enablement service can be used to optimize SEAL service operation?

NOTE: This KI does not preclude the case where ADAE service is a SEAL service.

## 4.5 Key issue #5: Support for slice-related application data analytics

Data analytics related to slicing are provided by the 5GS, from NWDAF (e.g. slice load analytics) and MDAS (e.g. NSI/NSSI performance analytics). The slice capability enablement layer (based on NSCALE) discusses enhancements to NSCE SEAL service (as specified in TS 23.434 \[5\]). According to Solution \#5 of TR 23.700-99, the NSCE server is expected to consume 5GS services related to analytics (from MDAS, NWDAF) and to re-expose them to the VAL server (slice customer).

If further analytics is required on top of the consumed analytics services (MDAS/NWDAF), the ADAES can be utilized by NSCE service to perform further analytics related to applications for certain slice / NSI. Such analytics service is not overlapping with NWDAF/MDAS services since it will provide at application layer data analytics (per session or VAL server) which are bound to a given slice or NSI (e.g. per VAL session performance statistics when using slice \#1).

This key issue will investigate:

\- what is the possible interaction between NSCE service and ADAES, for providing application layer analytics bound to a slice or an NSI?

\- whether and what data need to be collected by NSCE layer for supporting per slice or NSI app layer analytics?

## 4.6 Key issue #6: Support for slice configuration recommendation enablement

Slice data analysis can analyze the slice usage pattern based on the collected network slice performance and analytics, and provide analysis-based slice management suggestions, such as the slice scale in and scale out, which can be exposed to VAL or provided to NSCE as a service. One example is, to support the application layer automatic network slice lifecycle management, in which the NSCE server is supposed to send out some management recommendation based on the collected network slice performance analytics from the 5GC, OAM and the application layer. The recommendation is usually an empirical value given by experienced network operations, ADAES can help to output the recommendation according to the analysis based on historical network slice status and network performance.

Hence, in this key issue the following points shall be studied:

\- How ADEAS supports the slice configuration recommendation based on the slice related information from NSCE.

## 4.7 Key issue #7: Support for location accuracy analytics

According to SA1 TS 22.261 (6.27) and TS 22.104, positioning services aim to support verticals and applications with positioning accuracies better than 10 meters, thus more accurate than the ones of TS 22.071 for LCS. High accuracy positioning is characterized by ambitious system requirements for positioning accuracy in many verticals and applications, including regulatory needs. For example, on the factory floor, it is important to locate assets and moving objects such as forklifts, or parts to be assembled. Similar needs exist in transportation and logistics, for example rail, road and use of UAVs. In some road user cases, UE's supporting V2X application(s) are also applicable to such needs. In cases such as guided vehicles (e.g. industry, UAVs) and positioning of objects involved in safety-related functions, availability needs to be very high. In SA1, different service levels are mapped to different positioning performance attributes including vertical and horizontal accuracies. Such accuracies (e.g. cm-level, dm-level, meter-level) may depend on the positioning methods which are used, the LCS producers, as well as the UE mobility and the environment.

When the VAL consumer requests a positioning service, the accuracy is calculated at the entity which produces a location estimate and whether the accuracy can be maintained along an application session (for a given time/area) is challenging to answer at the time of the request/subscription. In this scenario, there needs to be a translation of the per UE location report accuracy to an expected /predictive location accuracy derivation for the application requiring positioning services. Such location accuracy analytics and in particular the sustainability of vertical and horizontal accuracy per VAL application (e.g. group of field devices in industrial use cases) based on per UE reported location accuracies could be needed to make sure that LMS will meet the VAL customer location reporting requirements for a given time/area of location request validity. Such information will help deciding from application side whether for a particular service (e.g. process automation, AR in factories) adaptation of the application behavior if the accuracy cannot be maintained e.g. programs the IIOT devices to maintain a bigger distance etc.

This key issue aims to investigate:

\- whether and how ADAES needs to be enhanced to perform analytics on vertical and horizontal accuracy for positioning services requested by a VAL customer?

\- what criteria need to be considered (e.g. environment, UE mobility, service type, positioning method, fusion) and what data are needed to be collected from 5GS (e.g. NWDAF, LMF) and VAL side for performing location accuracy analytics for the VAL application?

\- what enhancements are needed in SEAL LMS to support location accuracy analytics/data per VAL application?

## 4.8 Key issue #8: Support for service API capability analytics

The service APIs (assuming also EAS provided APIs, enablement service APIs and OAM API), cannot be assumed uniformly available and offering the same service level across the entire network. For CIoT service, 3GPP SA2 has already defined a NEF monitoring service to allow the AF to monitor the API availability and service level (e.g. via invoking a Nnef_APISupportCapability API as part of the Monitoring Event in TS 23.522) for the target API. However, this doesn't provide analytics on NEF/SCEF APIs and doesn't support all ranges of service APIs (produced or offered at the platform) and focuses on the CIoT scenarios. Furthermore, CAPIF supports the monitoring of service API invocations and can provide API monitoring via the *Availability of service APIs event notification* or *Service Discover Response* as specified in TS 23.222.

Service API analytics (such as the statistics on the successful/failed API invocation or predicted API availability for a given deployment) can be a tool to be used by the API provider (ASP, ECSP, MNO) to help optimizing the API usage by enabling him to trigger API related actions like API mashups, API rate limitations/throttling events, or pro-actively detecting API termination point changes which may affect service performance. Such service could be also useful for the API invoker to allow for early notification on expected API unavailabilities.

One example for such API analytics can be the statistics or prediction of NEF API or SEAL API invocation request failure probability, or the predicted number of API invocations for a particular EDN area and time of day or even the number of unauthorized API invocation requests. Such analytics can be matched to different APIs and API operations and can be used as a service for example to help the service API invoker to identify what is the best time and means to perform a request e.g. so as to avoid possible failure due to high number of invocations expected for this service API.

This key issue will investigate:

\- whether and how the application data analytics enablement service needs to provide data analytics for service APIs?

\- what data / API logs and from which entities need to be collected for performing service API analytics?

\- what enhancements are needed in CAPIF (CCF, API management function) for supporting service API analytics?

NOTE: Data/analytics related to MNO provided APIs are only possible if ADAES is deployed by the MNO. For ADAES outside MNO domain, this key issue covers only non-MNO provided APIs.
