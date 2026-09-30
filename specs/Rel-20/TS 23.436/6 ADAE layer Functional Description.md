---
spec: TS 23.436
version: 20.2.0
release: '20'
clause: 6
title: 6 ADAE layer Functional Description
source_archive: 23436-k20.zip
source_document: 23436-k20.docx
content_origin: 3gpp-source
---

# 6 ADAE layer Functional Description


## 6.1 Support for application performance analytics

This feature supports the derivation and exposure of application layer analytics to provide insight on the operation and performance of an application (VAL server or EAS, application session), and in particular statistics or prediction on parameters related to e.g. VAL server number of connections for a given time and area, VAL server rate of connection requests, connection probability failure rates, RTT and deviations for a VAL server or VAL UE session, packet loss rates etc. This feature also supports the collection of service experience information from the ADAE clients (as described in clause 8.9) to support application performance analytics.

## 6.2 Support for slice-specific application performance analytics

This feature introduces application layer analytics to provide insight on the performance of the VAL applications when using a given network slice (from a list of subscribed slices for the VAL customer). Such capability provides an analytics service to a consumer who can be either the VAL server (for helping to identify what slice it will use for its applications) or for other consumers such as SEAL NSCE to support on providing analytics (since NSCE doesn't contain an analytics engine for providing analytics on top of NWDAF \[4\] /MDAS \[5\]).

## 6.3 Support for UE-to-UE application performance analytics

This feature supports the derivation and exposure of application layer analytics to predict the performance of an application session among two or more VAL UEs within a service or group. Such prediction relates to application QoS attributes prediction for a given time horizon and area. This can be requested by the VAL server during the session, or the VAL server can subscribe to receive predicted application QoS downgrade indication for an ongoing session. Such analytics will help improving the application service experience and allow the VAL layer to pro-actively adapt to predicted application QoS changes.

## 6.4 Support for location accuracy analytics

This feature supports application layer analytics enablement to allow a VAL server to be notified based on analytics whether the accuracy of a location can be met for a given application and optionally for a given UE/group route. For example, a VAL server may request the ADAE server to provide analytics whether the accuracy of a location for the UEs within a VAL application is predicted to be sustainable or is expected to downgrade in a specific area or for an expected route from location A to location B.

## 6.5 Support for service API analytics

This feature introduces service API analytics to allow a VAL server or any other consumer (e.g. API provider) to be notified on the predicted /statistic availability and service level for the requested service API analytics. Such analytics may be utilized by the API provider to perform actions to avoid service API invocation failures or other actions like throttling/rate limitations. Also, such analytics will support the VAL server to identify if/when to perform an API invocation request based on the API expected status at the given area and time horizon.

## 6.6 Slice usage pattern analytics

Slice usage pattern analytics provides network slice usage pattern analytics based on collected network slice performance and analytics, historical network slice status, and network performance to help the analytics consumer manage the network slice.

## 6.7 Support for edge load analytics

Edge load analytics provide insight on the operation and performance of an EDN and in particular statistics or prediction on parameters related to:

\- the EAS / EES load for one or more EAS/EES

\- edge platform load parameters, which include the aggregated load per EDN or per DNAI due to the edge support services and e.g., load level of edge computational resources.

Such analytics can improve edge support services by allowing the pro-active edge service operation changes to deal with possible edge overload scenarios. For example, this can trigger EAS migration to a different EDN / central DN, or pro-active EAS reselection for a target UE or group of UEs.

## 6.8 Edge computing preparation analytics

This feature introduces exposure of edge computing preparation analytics of the EAS, EES, and/or ECS to the analytics consumer (e.g., the VAL server, ECS, EES). The ADAE server provides the edge computing preparation analytics based on collected edge deployment time information, historical edge computing preparation analytics, instantiation triggering time and registration time from the EDN.

## 6.9 Support for server-to-server performance analytics

This feature supports server-to-server performance analytics to allow an analytics consumer (such as VAL server or EES) to be notified on QoS analytics or predictions between two or more servers. Such prediction relates to QoS attributes prediction for a given time horizon and area. Such analytics allow the VAL layer to pro-actively adapt to predicted QoS changes.

## 6.10 Support for collision detection analytics

This feature supports collision detection analytics to allow an analytics consumer (such as VAL Server, LM server, UAE server, UAS application specific server) to be notified on analytics for collision detection between any target VAL UEs, collision detection between any UEs and target VAL UEs, or collision detection between any UE within the Area of Interest.

## 6.11 Support for location-related UE group analytics

This feature supports location-related UE group analytics to allow an analytics consumer (such as LMS) to be notified on analytics for UE group route or UE group member deviation. Such analytics can be used, e.g. UE group route prediction can be used to formulate application group profile with Expected Group Geographical Service Area as described in 3GPP TS 23.558 \[14\] clause 8.2.11. UE group member deviation prediction can be used for VAL to know which UE group member falls behind other group members or target group member (then VAL can send warning/reminder to the group members).

## 6.12 Support for Application Layer AI/ML Member Capability Analytics

This feature supports Application Layer AI/ML Member Capability Analytics to allow an analytics consumer (such as e.g. VAL Server, AIMLE Server) to be notified on analytics for application layer AI/ML Member capability. Such analytics can be used to support application layer AI/ML services, e.g. supporting FL member selection and reselection.

## 6.13 Support for VAL performance analytics for tethered UEs

This feature supports a new ADAES analytics functionality on tethered VAL connectivity performance. The tethered VAL connectivity performance can be defined as the application session performance corresponding to either only the tethered link (tethered UE and host UE) or the end-to-end VAL performance including the tethered link (as extension of VAL session performance analytics).

## 6.14 Support for DN Energy Analytics

This feature supports a logical functionality at the ADAES to provide analytics on the energy consumption /efficiency of an edge platform (including the EESs / EASs). The DN energy analytics is performed per DNN/ DNAI and may be used to trigger the application server migration to different cloud. The analytics are based on NWDAF analytics and UPF/DN measurements on user plane load as well as edge/app side measurements on the energy consumption.

## 6.15 Support for ML Model Performance Degradation Detection

This feature supports ML model performance degradation detection to allow a consumer (such as e.g. AIMLE Server) to be notified on performance degradation of an ML model. Such detection can be used to support application layer AI/ML operations, e.g. supporting decision making on retrain an ML model.

## 6.16 Support for monitoring ML-enabled analytics correctness

This feature supports providing ADAE analytics performance monitoring where the AIMLE server detects an expected or predicted model degradation performance based on the ML-enabled ADAE analytics performance feedback. Such ADAE analytics may be about the edge load or VAL server or session performance analytics which utilizes ML methods (training by AIMLE) for predicting the end-to-end performance.

## 6.17 Support for Energy information Analytics

This feature supports a logical functionality at the ADAES to provide the energy consumption analytics for location services per VAL UE. The ADAES collects the energy related data from 3GPP CN, ESE server and/or A-ADRF, and generates the analytics (including statistic and prediction) for energy information per VAL UE per location service as requested.

The consumer (i.e. LMS) utilizes the obtained energy information analysis to reduce the energy consumption for the location services per VAL UE.

## 6.18 Support for AI/ML energy consumption analytics

This feature supports AI/ML energy consumption analytics where analytics consumers can request energy consumption information associated with AI/ML operations performed by AIMLE or VAL clients. AI/ML operations are compute intensive and time consuming and energy consumption analytics can be used to measure the impact AI/ML operations has on overall energy consumption of the AIMLE or VAL clients.

## 6.19 Support for AIMLE client energy sustainability analytics

This feature supports a new ADAE analytics service for "AIMLE client energy sustainability analytics" as requested from the Consumer who can be the VAL Server or AIMLE server to the ADAES. These analytics may be used for predicting whether an AI/ML client at the VAL UE can be considered a candidate in a ML training or inference task, or FL process based on its expected energy status/capability.
