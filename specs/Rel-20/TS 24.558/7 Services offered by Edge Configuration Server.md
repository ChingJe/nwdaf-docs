---
spec: TS 24.558
version: 20.0.0
release: '20'
clause: 7
title: 7 Services offered by Edge Configuration Server
source_archive: 24558-k00.zip
source_document: 24558-k00.docx
content_origin: 3gpp-source
---

# 7 Services offered by Edge Configuration Server


## 7.1 Introduction

The table 7.1-1 lists the Edge Configuration Server APIs below the service name. A service description clause for each API gives a general description of the related API.

Table 7.1-1: List of ECS Service APIs

| Service Name             | Service Operations | Operation Semantics | Consumer(s) |
|--------------------------|--------------------|---------------------|-------------|
| Eecs_ServiceProvisioning | Request            | Request/Response    | EEC         |
|                          | Subscribe          | Subscribe/Notify    | EEC         |
|                          | Notify             |                     |             |
|                          | UpdateSubscription |                     |             |
|                          | Unsubscribe        |                     |             |

Table 7.1-2 summarizes the corresponding Edge Configuration Server APIs defined in this specification.

Table 7.1-2: API Descriptions

| Service Name             | Clause | Description               | OpenAPI Specification File            | apiName                  | Annex |
|--------------------------|--------|---------------------------|---------------------------------------|--------------------------|-------|
| Eecs_ServiceProvisioning | 7.2    | Eecs Service Provisioning | TS24558_Eecs_ServiceProvisioning.yaml | eecs-serviceprovisioning | B.1   |

## 7.2 Eecs_ServiceProvisioning Service


### 7.2.1 Service Description

The Eecs_ServiceProvisioning API, as defined in 3GPP TS 23.558 \[2\], allows an EEC via the Eecs interface to obtain service provisioning information as a one-time request or to subscribe for reporting from the ECS.

### 7.2.2 Service Operations


#### 7.2.2.1 Introduction

The service operation defined for Eecs_ServiceProvisioning API is shown in the table 7.2.2.1-1.

Table 7.2.2.1-1: Operations of the Eecs_ServiceProvisioning API

|                                             |                                                                                                                                  |              |
|---------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------|--------------|
| Service operation name                      | Description                                                                                                                      | Initiated by |
| Eecs_ServiceProvisioning_Request            | This service operation is used by the EEC to request for one-time service provisioning information.                              | EEC          |
| Eecs_ServiceProvisioning_Subscribe          | This service operation is used by the EEC to subscribe to ECS for reporting of service provisioning information.                 | EEC          |
| Eecs_ServiceProvisioning_Notify             | This service operation is used by the ECS to notify the EEC about the service provisioning information.                          | ECS          |
| Eecs_ServiceProvisioning_UpdateSubscription | This service operation is used by the EEC to update its subscription at ECS for reporting of service provisioning information.   | EEC          |
| Eecs_ServiceProvisioning_Unsubscribe        | This service operation is used by the EEC to remove its subscription from ECS for reporting of service provisioning information. | EEC          |

#### 7.2.2.2 Eecs_ServiceProvisioning_Request


##### 7.2.2.2.1 General

This service operation is used by the EEC to request for one-time service provisioning information.

##### 7.2.2.2.2 EEC requesting service provisioning information using Eecs_ServiceProvisioning_Request operation

To request for the one-time service provisioning information, the EEC shall send an HTTP POST request (custom operation: "Request") to the ECS with the request URI set to"{apiRoot}/eecs-serviceprovisioning/\<apiVersion\>/request". And the body including the ECSServProvReq data structure, as specified in clause 8.1.5.2.2.

Upon receiving the HTTP POST message from the EEC, the ECS shall:

a\) process the EEC service provisioning request information;

b\) verify and check if the EEC is authorized to request service provisioning information from ECS;

c\) if the EEC is authorized to request service provisioning information from ECS, then the ECS:

1\) may obtain the UE's location as specified in clause 5.3 of 3GPP TS 29.122 \[3\];

2\) if the EdgeApp_2 feature is supported and if "plmnId" attribute is not provided within the "connInfo" attribute, the ECS may obtain the UE roaming status and serving PLMN identifier by invoking NEF monitoring API using the 3GPP core network capabilities. In case of UE is roaming, the ECS may use the serving PLMN identifier to identify the partner ECS information like ECS end point, DNN and S-NSSAI to update the "redirectedECS" attribute of the ECSServProvResp data type;

i\) if the EdgeApp_3 feature is supported and if "predictExpTime" attribute is provided by which the UE moves to the predicted location, the ECS may also identify those EES with the non instantiated EAS that can be instantiated before "predictExpTime" based on the EAS deployment time information received from ADAES (as specified in clause 8.8.2 of 3GPP TS 23.436 \[9\]) or locally available information;

2A) if the 5gSatApp feature is supported and if the "satelliteId" attribute is provided by the EEC, then the ECS identifies those EES(s) providing the "ueServingSat" information" in their EES profile with "satId" attribute as specified in clause 9.1.5.2.12 of 3GPP TS 29.558 \[4\] matching the "satelliteId" attribute;

3\) if the "acProfs" attribute is provided by the EEC without the "appGroupProfile" attribute in the "appInfo" attribute, the ECS identifies the EES(s) based on the provided "acProfs" attribute and the UE location, and if the enNB1 feature is supported, the "userLocation" attribute may be provided in the "locInf" attribute within the ECSServProvReq data type;

i\) if acSvcContSupp information is included in the AC Profile, the matching EES has to support ACRScenario indicated in the acSvcContSupp information; and

ii\) for each AC Profile, if eass information is included in the AC Profile, the ECS identifies the matching EES such that the EES profile matches easId information. ECS may also include EAS instantiation information using "easInstInfos" attribute in eass information;

4\) if the EdgeApp_2 feature is supported and the EEC provided the "appGroupProfile" attribute in the "appInfo" attribute;

i\) then the ECS identifies the matching EES(s) based on the application group identity shared in "appGrpId" attribute and may also utilize the location information in the "expectedSvcArea" attribute if provided; or

ii\) the ECS retrieves the EES(s) information corresponding to the application group ID as specified in clause 6.4 of 3GPP TS 29.558 \[4\];

NOTE 1: How the EES determines the validity of the application group is implementation specific.

5\) if neither application group profile nor AC profiles(s) are provided:

i\. if available, the ECS identifies the EES(s) based on the UE-specific service information at the ECS and the UE location; and

ii\. ECS identifies the EES(s) by applying the ECSP policy (e.g. based on the UE location);

6\) if the EdgeApp_2 feature is supported:

i\. the ECS may identify the EES based on the EEC service continuity support information and EES service continuity support information; and

ii\. if the EEC provided the list of desired ECSP identifiers within the "ecspIds" attribute, the ECS shall identify the matching EES(s) based on the registered ECSP identifier in EES profile and the received list of desired ECSP identifiers; and

iii\) in case of no matching EES(s), the ECS identifies those partner ECS that satisfies the requirements. If required by the ECSP policies, the ECS may:

A\) trigger ECS discovery procedure as specified in clause 6.6 of 3GPP TS 29.558 \[4\] to discover the partner ECS(s); and

B\) trigger service provisioning information retrieval procedure as specified in clause 6.5 of 3GPP TS 29.558 \[4\] to obtain service provisioning information from the partner ECS; and

iv\) if the ECS is provisioned with authentication methods supported by matching EES(s) as specified in clause 6.3 of 3GPP TS 33.558 \[7\], then the ECS may include the "eesAuthMethods" attribute for each candidate EES(s) as specified in clause 8.1.5.2.9 in the ECSServProvResp and if multiple authentication methods are supported by the EES, then it is left to the EEC implementation to choose any method it supports when it communicates with the EES; and

7\) shall determine other information that needs to be provisioned, as defined in the EDNConfigInfo data structure as specified in clause 8.1.5.2.7 and shall include it in the ECSServProvResp; and

8\) if the EdgeApp_3 feature is supported:

i\) if the EEC provided "appInfo" attribute includes both the "appGroupProfile" attribute and the "easBundleInfos" attribute, then the ECS consider this as common EAS bundle scenario and identifies the EES(s) by using these information; and

9\) if the EdgeApp_2_Ext1 feature is supported:

i\) if the EEC provided the "tunnel_Info" attribute, then the ECS consider this tunnel information to identify the EES(s) else the ECS may obtain the N6 tunnel information by invoking NEF API using the 3GPP core network capabilities to identify the EES(s) close to the EEC for optimal N6 routing; and

d\) if the ECS is able to determine service provisioning information using the inputs in service provisioning request, UE-specific service information at the ECS or the ECSP's policy, then the ECS returns an HTTP "200 OK" status code response with the response body including the ECSServProvResp data structure which may include the lifetime of the provided EDN configuration information.

If the EdgeApp_2 feature is supported and in case of roaming, if the partner ECS(s) information has been identified in step c.2), the ECS sends a successful response including the "redirectedECS" attribute containing the list of ECS(s) configuration information to which the EEC shall redirect the service provisioning request.

If the inputs in service provisioning request do not match any EDN configuration information (i.e. there is no client side error), the ECS sends an HTTP "204 No Content" status code response code.

Otherwise, the ECS shall reject the service provisioning request and respond with an appropriate failure cause.

The EEC may cache the service provisioning information (e.g. EES endpoint). If the lifeTime attribute is included in the service provisioning response, then the EEC may cache and reuse the service provisioning information only for the duration specified by the lifeTime attribute.

The EEC may initiate service provisioning procedure with ECS(s) provided in the "redirectedECS" attribute of the "ECSServProvResp" data type. If the UE is roaming to a V-PLMN, then EEC may establish a connection with the partner ECS that can be a HR-SBO PDU session or an LBO PDU session.

The EEC may select one or more EES to perform EAS discovery, for multiple EES(s) case, if instantiable EAS information using "easInstInfos" attribute for an EAS is not available, or the instantiable EAS information using "easInstInfos" attribute is set to instantiated or instantiable.

The EEC may consider the instantiable EAS information using "easInstInfos" attribute and the associated instantiation criteria to mitigate the waste of EDN resources for EAS discovery. The EEC selects one EES, if the EAS instantiation status corresponding to the EASID requested by AC/EEC is instantiable but not yet instantiated (i.e. no instantiated EAS).

NOTE 2: If the EAS instantiation fails based on the selected EES, the EEC retries the EAS discovery request to another EES ((e.g. selecting another one EES based on the instantiable EAS information).

NOTE 3: How EEC maintains the service provisioning information is implementation specific.

#### 7.2.2.3 Eecs_ServiceProvisioning_Subscribe


##### 7.2.2.3.1 General

This service operation is used by the EEC to subscribe to ECS, for reporting of service provisioning information when changes to provisioning information occur which are of interest to EEC.

##### 7.2.2.3.2 EEC subscribing to service provisioning information from ECS using Eecs_ServiceProvisioning_Subscribe operation

To subscribe to changes to service provisioning information at the ECS, the EEC shall send an HTTP POST message to the ECS on the Service Provisioning Subscriptions resource. The body of the POST message may include Notification Target Address (e.g. URL, provided with in "notificationDestination" attribute), the UE identifier (e.g. GPSI), connectivity information, proposed expiration time, AC Profile information and if the EdgeApp_2 feature is supported the list of desired ECSP identifiers and indication whether the application triggering is required with in "eecTriggerRequest" attribute, as specified in clause 8.1.2.3.3.1.

If the "eecTriggerRequest" attribute is included then the "notificationDestination" attribute shall not be included.

Upon receiving the HTTP POST message from the EEC, the ECS shall:

a\) process the EEC service provisioning subscription request;

b\) verify and check if the EEC is authorized to subscribe for the service provisioning information; and

c\) if the EEC is authorized to subscribe for the service provisioning information, then the ECS;

1\) may obtain the UE's location as specified in clause 5.3 of 3GPP TS 29.122 \[3\];

2\) shall create a new resource with the Service Provisioning Subscriptions resource as specified in clause 8.1.2.3;

3\) if the ECS determines the EES information using the inputs in service provisioning subscription request, UE-specific service information at the ECS or the ECSP policy, then the ECS returns the service provisioning subscription response, which includes the subscription identifier and may include the expiration time, indicating when the subscription will automatically expire. Otherwise, the ECS shall reject the service provisioning subscription request and respond with an appropriate failure cause and

4\) if EdgeApp_2 feature is supported and the EEC required EEC Trigger Request by setting the "eecTriggerRequest" attribute to true, then ECS may send trigger towards the EEC to perform service provisioning.

If the expiration time is provided, then to maintain the subscription, the EEC shall send a Service provisioning subscription update request (as described in clause 7.2.2.5) prior to the expiration time. If a Service provisioning subscription update request is not received prior to the expiration time, the ECS shall treat the EEC as implicitly unsubscribed and remove the corresponding service provisioning subscription resource.

#### 7.2.2.4 Eecs_ServiceProvisioning_Notify


##### 7.2.2.4.1 General

This service operation is used by the ECS to notify the EEC about the service provisioning information.

##### 7.2.2.4.2 ECS notifying the service provisioning information to EEC using Eecs_ServiceProvisioning_Notify operation

The ECS determines to notify the EEC with the service provisioning information, when an event occurs at the ECS that satisfies trigger conditions for updating service provisioning of a subscribed EEC.

The ECS may obtain the UE's location as specified in clause 5.3 of 3GPP TS 29.122 \[3\]. If the EdgeApp_2 feature is supported and if "plmnId" attribute is not provided within the "connInfo" attribute of the "ECSServProvSubscription" data type, the ECS may obtain the UE roaming status and serving PLMN identifier by invoking NEF monitoring API using the 3GPP core network capabilities. In case of UE is roaming, the ECS may use the serving PLMN identifier to identify the partner ECS information like ECS end point, DNN and S-NSSAI to update the "redirectedECS" attribute of the ECSServProvResp data type. If AC profile(s) were provided by the EEC during subscription creation, the ECS identifies the EES(s) based on the provided AC profile(s) and the UE location.

NOTE 1: How the ECS identifies the EES(s) based on the provided AC profile(s) and the UE location is implementation specific.

If AC profiles(s) were not provided, then if available, the ECS identifies the EES(s) based on the UE-specific service information at the ECS and the UE location. The ECS may also identify the EES(s) by applying the ECSP policy (e.g. based only on the UE location). If the EdgeApp_2 feature is supported and the ECS received the list of desired ECSP identifiers, the ECS identifies the EES(s) based on the registered ECSP identifier in EES profile and the received list of desired ECSP identifiers.

The ECS also determines other information that needs to be provisioned, as defined in the EDNConfigInfo data structure as specified in clause 8.1.5.2.7 and shall include it in the ServProvNotification data structure.

If the EdgeApp_2 feature is supported and in case of no matching EES(s), the ECS identifies those partner ECS that satisfies the requirements. If required by the ECSP policies, the ECS may:

A\) trigger ECS discovery procedure as specified in clause 6.6 of 3GPP TS 29.558 \[4\] to discover the partner ECS(s); and

B\) trigger service provisioning information retrieval procedure as specified in clause 6.5 of 3GPP TS 29.558 \[4\] to obtain service provisioning information from the partner ECS.

To notify the service provisioning information events, the ECS shall send an HTTP POST message using the Notification Destination URI received in the subscription request, as specified in clause 8.1.4.2. If the EdgeApp_2 feature is supported and in case of roaming, if the partner ECS(s) information has been identified, the ECS sends a notification including the "redirectedECS" attribute containing the list of ECS(s) configuration information to which the EEC shall redirect the service provisioning request.

Upon receiving the HTTP POST message, the EEC shall process the service provisioning information. The EEC may cache the service provisioning information (e.g. EES endpoint). If the lifeTime attribute is included in the service provisioning response, then the EEC may cache and reuse the service provisioning information only for the duration specified by the lifeTime attribute. The EEC may initiate service provisioning procedure with ECS(s) provided in the "redirectedECS" attribute of the "ECSServProvResp" data type. If the UE is roaming to a V-PLMN, the EEC may establish a connection with the partner ECS that can be a HR-SBO PDU session or an LBO PDU session. If the ECS provided information regarding the service continuity support of individual EESs, the EEC may take this information into account when selecting an EES for EEC registration, EAS discovery or T-EAS discovery, respectively.

NOTE 2: How the EEC maintains the service provisioning information is implementation specific.

NOTE 3: If the EEC provided an indication to support application triggering in "eecTriggerRequest" attribute of the Service Provisioning subscription request, then the ECS sends the trigger message towards the EEC by invoking application triggering services or DeviceTrigerring API using 3GPP core network capabilities in order to avoid sending the service provisioning notify.

#### 7.2.2.5 Eecs_ServiceProvisioning_UpdateSubscription


##### 7.2.2.5.1 General

This service operation is used by the EEC to update its subscription at the ECS, for reporting of service provisioning information.

##### 7.2.2.5.2 EEC updating service provisioning information subscription at ECS using Eecs_ServiceProvisioning_UpdateSubscription operation

To update service provisioning information subscription at the ECS, the EEC shall send an HTTP PATCH message (for partial modification) or HTTP PUT message (for fully replacement) to the ECS on resource URI identifying the Individual Service Provisioning Subscription resource representation, as specified in clause 8.1.2.4.3.3 for an HTTP PATCH message and in clause 8.1.2.4.3.1 for an HTTP PUT message.

The PATCH message includes the parameters (AC Profiles, proposed expiry time, service continuity support or list of connectivity information) that need to be replaced in the existing subscription resource.

The PUT message shall replace all properties of the existing resource with the service provisioning information in the request. The values of the eecId and ueId provided during the subscription creation shall not be changed.

Upon receiving the HTTP PATCH or PUT message from the EEC, the ECS:

a\) shall check the update subscription message from the EEC to see if the EEC is authorized to modify the requested subscription resource;

b\) if the EEC is authorized to update the service provisioning subscription and the eecId of the requesting EEC and the eecId in the resource match, then the ECS;

1\) may obtain the UE's location as specified in clause 5.3 of 3GPP TS 29.122 \[3\];

2\) shall update the resource identified by Resource URI of the service provisioning subscription with the updated information received in the HTTP PATCH or PUT request message;

3\) shall return the service provisioning subscription response. The ECS may send "200 OK" response code which includes the subscription identifier and the expiration time, indicating when the subscription will automatically expire. Otherwise, the EES sends "204 No Content" response code.

If the expiration time is provided, the EEC shall send a service provisioning subscription update request prior to the expiration time if the EEC wants to maintain the subscription. If a service provisioning subscription update request is not received prior to the expiration time, the ECS shall treat the EEC as implicitly unsubscribed and remove the corresponding service provisioning subscription resource.

#### 7.2.2.6 Eecs_ServiceProvisioning_Unsubscribe


##### 7.2.2.6.1 General

This service operation is used by the EEC to remove its subscription from the ECS for reporting of service provisioning information.

##### 7.2.2.6.2 EEC unsubscribing to service provisioning subscription from ECS using Eecs_ServiceProvisioning_Unsubscribe operation

To unsubscribe service provisioning subscription from the ECS, the EEC shall send an HTTP DELETE message to the ECS, on the resource URI identifying the individual service provisioning subscription resource representation as specified in clause 8.1.2.4.3.2. Upon receiving the HTTP DELETE request, the ECS:

a\) shall verify and check if the EEC is authorized to unsubscribe the Individual Service Provisioning Subscription resource;

b\) if the EEC is authorized to delete the Individual Service Provisioning Subscription resource, then the ECS shall unsubscribe the EEC service provisioning subscription identified by the subscriptionId;

c\) shall return the "204 Not Content" message to the EEC, indicating the successful removal of the subscription information.
