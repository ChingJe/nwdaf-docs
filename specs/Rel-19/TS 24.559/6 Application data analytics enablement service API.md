---
spec: TS 24.559
version: 19.4.1
release: '19'
clause: 6
title: 6 Application data analytics enablement service API
source_archive: 24559-j41.zip
source_document: 24559-j41.docx
content_origin: 3gpp-source
---

# 6 Application data analytics enablement service API


## 6.1 General

The clause describes the procedures of the application data analytics enablement service API.

## 6.2 Application performance analytics


### 6.2.1 Service description


#### 6.2.1.1 Overview

The ADAE_ServiceConfiguration API, as defined 3GPP TS 23.436 \[3\], allows the ADAES via ADAE-UU reference point to subscribe to ADAEC to the event of the VAL performance analytics.

### 6.2.2 Service Operations


#### 6.2.2.1 Introduction

The service operation defined for ADAE_ServiceConfiguration API for application performance analytics is shown in the table 6.2.2.1-1.

Table 6.2.2.1-1: Operations for application performance analytics

| Service operation name                | Description                                                                                             | Initiated by |
|---------------------------------------|---------------------------------------------------------------------------------------------------------|--------------|
| Subscribe_VAL_Performance_Analytics   | This service operation is used by ADAES to subscribe to the event of the VAL performance analytics.     | ADAES        |
| Notify_VAL_Performance_Analytics      | This service operation is used by ADAEC to notify about the VAL performance analytics.                  | ADAEC        |
| Unsubscribe_VAL_Performance_Analytics | This service operation is used by ADAES to unsubscribe from the event of the VAL performance analytics. | ADAES        |

#### 6.2.2.2 Subscribe_VAL_Performance_Analytics


##### 6.2.2.2.1 General

This service operation is used by the ADAES for VAL performance analytics event subscription to the ADAEC.

##### 6.2.2.2.2 Subscribing to VAL performance analytics event using Subscribe_VAL_Performance_Analytics service operation

To subscribe to VAL performance analytics event, the ADAES shall send an HTTP POST request with a Request-URI according to the pattern "{apiRoot}/adae-sc/\<apiVersion\>/application-performance" and with a body containing data type AppPerfSub as defined in clause 7.10.1.4.2.2 of 3GPP TS 29.549 \[9\].

Upon receipt of the HTTP POST request, the ADAEC shall:

a\) verify the identity of the ADAES and determine if the ADAES is authorized to subscribe to the VAL performance analytics event; and

b\) if the ADAES:

1\) is not authorized, the ADAEC shall respond to the ADAES with an appropriate error status code; or

2\) is authorized, the ADAEC shall create a new "Individual application performance event subscription" resource and respond to the ADAES with an HTTP "201 Created" status code, including a Location header field containing the URI for the created "Individual application performance event subscription" and the response body including the AppPerfSub data structure containing a representation of the created resource as defined in clause 7.1.3.

#### 6.2.2.3 Notify_VAL_Performance_Analytics


##### 6.2.2.3.1 General

This service operation is used by the ADAEC to send notification to the ADAES with the VAL performance analytics event subscription to the ADAEC.

##### 6.2.2.3.2 Notifying VAL performance analytics event using Notify_VAL_Performance_Analytics service operation

To notify VAL performance analytics event, the ADAEC shall send an HTTP POST request with a Request-URI according to the pattern "{notifUri}" and with a body containing data type AppPerfNotif as defined in clause 7.10.1.4.2.3 of 3GPP TS 29.549 \[9\].

Upon receipt of the HTTP POST request, the ADAES shall respond to the ADAEC with:

a\) if the request is successfully processed, a "204 No Content" status code and process the event notification; or

b\) if errors occur when processing the request, an appropriate error response as specified in clause 7.1.6.

#### 6.2.2.4 Unsubscribe_VAL_Performance_Analytics


##### 6.2.2.4.1 General

This service operation is used by the ADAES to unsubscribe from the VAL performance analytics event.

##### 6.2.2.4.2 Unsubscribing from VAL performance analytics event using Unsubscribe_VAL_Performance_Analytics service operation

To unsubscribe from VAL performance analytics event, the ADAES shall send an HTTP DELETE request to the resource representing the event in the ADAES as specified in clause 7.1.3.3.

Upon receiving the HTTP DELETE request:

a\) the ADAEC shall verify the identity of the ADAES and check if the ADAES is authorized to unsubscribe from the VAL performance analytics event associated with the resource URI "{apiRoot}/adae-sc/\<apiVersion\>/application-performance/{appPerfId}";

b\) if the ADAES is authorized to unsubscribe from the VAL performance analytics event, the ADAEC shall delete the resource pointed by the resource URI "{apiRoot}/adae-sc/\<apiVersion\>/application-performance/{appPerfId}";

c\) if the request is successfully processed, the ADAEC shall respond to the ADAES with a "204 No Content" status code; and

d\) if errors occur when processing the request, the ADAEC shall respond to the ADAES with an appropriate error response as specified in clause 7.1.6.

## 6.3 UE-to-UE session performance analytics


### 6.3.1 Service description


#### 6.3.1.1 Overview

The ADAE_ServiceConfiguration API, as defined 3GPP TS 23.436 \[3\], allows the ADAES via ADAE-UU reference point, to obtain the UE-to-UE session performance analytics from the ADAEC.

### 6.3.2 Service Operations


#### 6.3.2.1 Introduction

The service operation defined for ADAE_ServiceConfiguration API for UE-to-UE session performance analytics is shown in the table 6.3.2.1-1.

Table 6.3.2.1-1: Operations for UE-to-UE session performance analytics

| Service operation name                    | Description                                                                                   | Initiated by |
|-------------------------------------------|-----------------------------------------------------------------------------------------------|--------------|
| Fetch_UE2UE_Session_Performance_Analytics | This service operation is used by ADAES to obtain the UE-to-UE session performance analytics. | ADAES        |

#### 6.3.2.2 Fetch_UE2UE_Session_Performance_Analytics


##### 6.3.2.2.1 General

This service operation is used by the ADAES for obtaining the UE-to-UE session performance analytics from the ADAEC.

##### 6.3.2.2.2 Obtaining UE-to-UE session performance analytics using Fetch_UE2UE_Session_Performance_Analytics service operation

To obtain the UE-to-UE session performance analytics, the ADAES shall send an HTTP POST request with a Request-URI according to the pattern "{apiRoot}/adae-sc/\<apiVersion\>/ue2ue-session-performance/fetch" and with a body containing data type Ue2UePerfReq as defined in clause 7.1.5.2.2.

Upon receipt of the HTTP POST request, the ADAEC shall:

a\) verify the identity of the ADAES and determine if the ADAES is authorized to obtain the UE-to-UE session performance analytics; and

b\) if the ADAES:

1\) is not authorized, the ADAEC shall respond to the ADAES with an appropriate error status code; or

2\) is authorized, the ADAEC shall respond to the ADAES with an HTTP "200 OK" status code with the response body including the Ue2UePerfResp as defined in clause 7.1.3.3.4.2 with the following attributes:

i\) UE-to-UE session performance analytics;

ii\) one or more VAL UEs; and

iii\) identity of the UE-to-UE session performance analytics.

## 6.4 Edge load data collection


### 6.4.1 Service description

The ADAE_ServiceConfiguration API, as defined 3GPP TS 23.436 \[3\], allows the ADAES via ADAE-UU reference point to subscribe to ADAEC to the event of the edge load data collection.

### 6.4.2 Service Operations


#### 6.4.2.1 Introduction

The service operation defined for ADAE_ServiceConfiguration API for edge load data collection is shown in the table 6.4.2.1-1.

Table 6.4.2.1-1: Operations for edge load data collection

| Service operation name                | Description                                                                                         | Initiated by |
|---------------------------------------|-----------------------------------------------------------------------------------------------------|--------------|
| Subscribe_Edge_Load_Data_Collection   | This service operation is used by ADAES to subscribe to the event of the edge load data collection. | ADAES        |
| Notify_Edge_Load_Data_Collection      | This service operation is used by ADAEC to notify about the edge load data collection.              | ADAEC        |
| Unsubscribe_Edge_Load_Data_Collection | This service operation is used by ADAES to unsubscribe from the edge load data collection.          | ADAES        |

#### 6.4.2.2 Subscribe_Edge_Load_Data_Collection


##### 6.4.2.2.1 General

This service operation is used by the ADAES for edge load data collection event subscription to the ADAEC.

##### 6.4.2.2.2 Subscribing to edge load data collection event using Subscribe_Edge_Load_Data_Collection service operation

To subscribe to edge load data collection event, the ADAES shall send an HTTP POST request with a Request-URI according to the pattern "{apiRoot}/adae-sc/\<apiVersion\>/edge-load" and with a body containing data type EdgeSub as defined in clause 7.10.7.4.2.2 of 3GPP TS 29.549 \[9\].

Upon receipt of the HTTP POST request, the ADAEC shall:

a\) verify the identity of the ADAES and determine if the ADAES is authorized to subscribe to the edge load data collection event; and

b\) if the ADAES:

1\) is not authorized, the ADAEC shall respond to the ADAES with an appropriate error status code; or

2\) is authorized, the ADAEC shall create a new "Individual edge load event subscription" resource and respond to the ADAES with an HTTP "201 Created" status code, including a Location header field containing the URI for the created "Individual edge load event subscription" and the response body including the EdgeSub data structure containing a representation of the created resource as defined in clause 7.1.3.

#### 6.4.2.3 Notify_Edge_Load_Data_Collection


##### 6.4.2.3.1 General

This service operation is used by the ADAEC to send notification to the ADAES with the edge load data collection event subscription to the ADAEC.

##### 6.4.2.3.2 Notifying edge load data collection event using Notify_Edge_Load_Data_Collection service operation

To notify edge load data collection event, the ADAEC shall send an HTTP POST request with a Request-URI according to the pattern "{notifUri}" and with a body containing data type EdgeNotif as defined in clause 7.10.7.4.2.3 of 3GPP TS 29.549 \[9\];

Upon receipt of the HTTP POST request, the ADAES shall respond to the ADAEC with:

a\) if the request is successfully processed, a "204 No Content" status code and process the event notification; or

b\) if errors occur when processing the request, an appropriate error response as specified in clause 7.1.6.

#### 6.4.2.4 Unsubscribe_Edge_Load_Data_Collection


##### 6.4.2.4.1 General

This service operation is used by the ADAES to unsubscribe from the edge load data collection event.

##### 6.4.2.4.2 Unsubscribing from edge load data collection event using Unsubscribe_Edge_Load_Data_Collection service operation

To unsubscribe from edge load data collection event, the ADAES shall send an HTTP DELETE request to the resource representing the event in the ADAES as specified in clause 7.1.3.6.

Upon receiving the HTTP DELETE request:

a\) the ADAEC shall verify the identity of the ADAES and check if the ADAES is authorized to unsubscribe from the edge load data collection event associated with the resource URI "{apiRoot}/adae-sc/\<apiVersion\>/edge-load/{edgeLdId}";

b\) if the ADAES is authorized to unsubscribe from the edge load data collection event, the ADAEC shall delete the resource pointed by the resource URI "{apiRoot}/adae-sc/\<apiVersion\>/edge-load/{edgeLdId}";

c\) if the request is successfully processed, the ADAEC shall respond to the ADAES with a "204 No Content" status code; and

d\) if errors occur when processing the request, the ADAEC shall respond to the ADAES with an appropriate error response as specified in clause 7.1.6.

## 6.5 Service experience performance analytics


### 6.5.1 General

The ADAE_ServiceConfiguration API, as defined 3GPP TS 23.436 \[3\], allows the ADAES via ADAE-UU reference point to:

\- pull from the ADAEC, the service experience information report.

### 6.5.2 Service Operations


#### 6.5.2.1 Introduction

The service operation defined for ADAE_ServiceConfiguration API for service experience information is shown in the table 6.5.2.1-1.

Table 6.5.2.1-1: Operations for service experience information

| Service operation name                     | Description                                                                            | Initiated by |
|--------------------------------------------|----------------------------------------------------------------------------------------|--------------|
| Pull_Service_Experience_Information_Report | This service operation is used by ADAES to pull service experience information report. | ADAES        |

#### 6.5.2.2 Configure_Triggers_Service_Information_Experience_Report


##### 6.5.2.2.1 General

This service operation is used by the ADAEC to fetch the configuration triggers from the ADAES.

##### 6.5.2.2.2 Configuring service experience information reporting using Configure_Triggers_Service_Information_Experience_Report service operation

To fetch the configuration triggers from the ADAES, if direct DC-Client is available in the UE, the ADAEC may use the direct DC-Client services as defined in clause 4.4.2 of 3GPP TS 26.532 \[5\]. The ADAEC may provide below information as input parameters to the application registration procedure:

a\) external application identifier specific to the ADAEC;

b\) application service provider identifier specific to the ADAES;

c\) callback listener of the ADAEC to receive the future response; and

d\) consent for the UE identity (i.e. GPSI) to be included in data reports, sent to the DC-AF.

Upon receiving the request, the DC-AF returns "DataReportingSession" resource as defined in clause 7.3.2.1 of 3GPP TS 26.532 \[5\] to DC-Client in the response message and in the "reportingRules" attribute, the "DataDomain" is set to "APPLICATION_SPECIFIC" and the "applicationSpecificRecords" container in the DataReportingRule data type shall contain the triggers as specified in the ConRepTrigger data type defined in table 6.5.2.2.2-1.

On success, the DC-Client provides the DataReportingSession data type as defined in clause 7.3.2.1 of 3GPP TS 26.532 \[5\] to the ADAEC.

Table 6.5.2.2.2-1: Definition of type ConfRepTrigger

| Attribute name  | Data type     | P   | Cardinality | Description                                                                                                                                                                              | Applicability |
|-----------------|---------------|-----|-------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|
| valServerIds    | array(string) | M   | 1..N        | Identities of one or more VAL servers, for which the configuration of the service experience information report applies.                                                                 |               |
| triggCrit       | string        | M   | 1           | Information criteria about the triggers on which the service experience is information to be reported for the VAL server and is set to value "TRIGGER_CRITERIA".                         |               |
| commonTriggCrit | string        | O   | 0..1        | Information criteria about the triggers (applicable to all VAL servers) on which the service experience information is fetched and is set to value "COMMON_TRIGGER_CRITERIA".            |               |
| srvExpMeas      | DurationSec   | O   | 0..1        | Information about the service experience information measurements which needs to be fetched and included in the report. If not present, by default end-to-end response time is measured. |               |
| notifyTarget    | string        | O   | 0..1        | The target address which is notified.                                                                                                                                                    |               |

#### 6.5.2.3 Void

#### 6.5.2.4 Push_Service_Experience_Information_Report


##### 6.5.2.4.1 General

This service operation is used by the ADAEC to push the service experience information report to the ADAES.

##### 6.5.2.4.2 Pushing service experience information report using Push_Service_Experience_Information_Report service operation

When Direct DC-Client is available in the UE, to push the service experience information report to the ADAES based on the request from VAL client or trigger conditions meeting, the ADAEC shall:

a\) create the service experience information report as defined in SrvExpInfoRep data type in table 7.1.5.2.7-1; and

b\) invoke the "reportUeData" method as defined in clause 4.4.4 of 3GPP TS 26.532 \[5\] and provide DataReport data type as defined in clause 7.3.2.3 of 3GPP TS 26.532 \[5\] as input parameter with the "applicationSpecificRecords" attribute set with the SrvExpInfoRep data type in table 7.1.5.2.7-1.

On receiving the service experience information request, the ADAES shall process the report from ADAEC to determine/predict analytics and initiate further actions as defined in clause 8.9.2.1 of 3GPP TS 23.436 \[3\].

#### 6.5.2.5 Pull_Service_Experience_Information_Report


##### 6.5.2.5.1 General

This service operation is used by the ADAES to pull the service experience information report from the ADAEC.

##### 6.5.2.5.2 Pulling service experience information report using Pull_Service_Experience_Information_Report service operation

To pull the service experience information report from the ADAEC, the ADAES shall send an HTTP POST request with a Request-URI according to the pattern "{apiRoot}/adae-sc/\<apiVersion\>/service-experience/pull" and with a body containing data type PullSrvExpInfo as defined in clause 7.1.5.2.6.

Upon receipt of the HTTP POST request:

a\) the ADAEC shall verify the identity of the ADAES and determine if the ADAES is authorized to pull the service experience information report; and

b\) if the ADAES:

1\) is not authorized, the ADAEC shall respond to the ADAES with an appropriate error status code; or

2\) is authorized, the ADAEC shall respond to the ADAES with an HTTP "200 OK" status code and with a body containing data type SrvExpInfoRep as defined in clause 7.1.5.2.7.

Upon receipt of the HTTP POST request, the ADAES shall respond to the ADAEC with a "204 No Content" status code and process the report.

## 6.6 Collision detection analytics


### 6.6.1 Service description


#### 6.6.1.1 Overview

The ADAE_ServiceConfiguration API, as defined 3GPP TS 23.436 \[3\], allows the ADAES via ADAE-UU reference point, to obtain the collision detection analytics from the ADAEC.

### 6.6.2 Service operations


#### 6.6.2.1 Introduction

The service operations defined for ADAE_ServiceConfiguration API for collision detection analytics are shown in the table 6.6.2.1-1.

Table 6.6.2.1-1: Operations for collision detection analytics

| Service operation name          | Description                                                                                | Initiated by |
|---------------------------------|--------------------------------------------------------------------------------------------|--------------|
| Subscribe_Collision_Detection   | This service operation is used by ADAES to subscribe to the event collision detection.     | ADAES        |
| Notify_Collision_Detection      | This service operation is used by ADAEC to notify about the collision detection analytics. | ADAEC        |
| Unsubscribe_Collision_Detection | This service operation is used by ADAES to unsubscribe to the event collision detection.   | ADAES        |

#### 6.6.2.2 Subscribe_Collision_Detection


##### 6.6.2.2.1 General

This service operation is used by the ADAES for obtaining the collision detection analytics from the ADAEC.

##### 6.6.2.2.2 Subscribing to collision detection analytics using Subscribe_Collision_Detection service operation

To obtain the collision detection analytics, the ADAES shall send an HTTP POST request with a Request-URI according to the pattern "{apiRoot}/adae-sc/\<apiVersion\>/collision-detection" and with a body containing data type CollisionDetectionSub as defined in clause 7.10.10.4.2.2 of 3GPP TS 29.549 \[9\].

Upon receipt of the HTTP POST request, the ADAEC shall:

a\) verify the identity of the ADAES and determine if the ADAES is authorized to subscribe to the collision detection analytics; and

b\) if the ADAES:

1\) is not authorized, the ADAEC shall respond to the ADAES with an appropriate error status code; or

2\) is authorized, the ADAC shall create a new "Individual collision detection analytics subscription" resource and respond to the ADAES with an HTTP "201 Created" status code with a Location header field containing the URI for the created "Individual collision detection analytics subscription" resource and the CollisionDetectionSub data structure in the response body containing a representation of the created resource as defined in clause 7.1.3.

#### 6.6.2.3 Notify_Collision_Detection


##### 6.6.2.3.1 General

This service operation is used by the ADAEC to notify the ADAES about the collision detection analytics event.

##### 6.6.2.3.2 Notifying collision detection analytics using Notify_Collision_Detection service operation

To notify collision detection analytics, the ADAEC shall send an HTTP POST request with a Request-URI according to the pattern "{notifUri}" and with a body containing data type CollisionDetectionNotify as defined in clause 7.10.10.4.2.3 of 3GPP TS 29.549 \[9\].

Upon receipt of the HTTP POST request, the ADAES shall respond to the ADAEC with:

a\) if the request is successfully processed, a "204 No Content" status code and process the event notification; or

b\) if error occurs when processing the request, an appropriate error response as specified in clause 7.1.6.

#### 6.6.2.4 Unsubscribe_Collision_Detection


##### 6.6.2.4.1 General

This service operation is used by the ADAEC to unsubscribe from the collision detection analytics.

##### 6.6.2.4.2 Unsubscribing from collision detection analytics using Unsubscribe_Collision_Detection service operation

To unsubscribe from collision detection analytics, the ADAES shall send an HTTP DELETE request to the "Individual collision detection analytics subscription" resource as specified in clause 7.1.3.10.

Upon receiving the HTTP DELETE request:

a\) the ADAEC shall verify the identity of the ADAES and check if the ADAES is authorized to unsubscribe from the collision detection analytics associated with the resource URI "{apiRoot}/adae-sc/\<apiVersion\>/collision-detection/{collisionDetectionId}";

b\) if the ADAES is authorized to unsubscribe from the collision detection analytics, the ADAEC shall delete the resource pointed by the resource URI "{apiRoot}/adae-sc/\<apiVersion\>/collision-detection/{collisionDetectionId}";

c\) if the request is successfully processed, the ADAEC shall respond to the ADAES with a "204 No Content" status code; and

d\) if error occurs when processing the request, the ADAEC shall respond to the ADAES with an appropriate error response as specified in clause 7.1.6.

## 6.7 Location-related UE Group Analytics


### 6.7.1 Service description


#### 6.7.1.1 Overview

The ADAE_ServiceConfiguration API, as defined 3GPP TS 23.436 \[3\], allows the ADAES via ADAE-UU reference point, to obtain the location-related UE group analytics from the ADAEC.

### 6.7.2 Service operations


#### 6.7.2.1 Introduction

The service operations defined for ADAE_ServiceConfiguration API for location-related UE group analytics are shown in the table 6.7.2.1-1.

Table 6.7.2.1-1: Operations for Location-related UE Group Analytics

| Service operation name        | Description                                                                                    | Initiated by |
|-------------------------------|------------------------------------------------------------------------------------------------|--------------|
| Subscribe_UE_Group_Location   | This service operation is used by ADAES to subscribe to location-related UE group analytics.   | ADAES        |
| Notify_UE_Group_Location      | This service operation is used by ADAEC to notify about location-related UE group analytics.   | ADAEC        |
| Unsubscribe_UE_Group_Location | This service operation is used by ADAES to unsubscribe to location-related UE group analytics. | ADAES        |

#### 6.7.2.2 Subscribe_UE_Group_Location


##### 6.7.2.2.1 General

This service operation is used by the ADAES for obtaining the location-related UE group analytics from the ADAEC.

##### 6.7.2.2.2 Obtaining location-related UE group analytics using Subscribe_UE_Group_Location service operation

To obtain the location-related UE group analytics, the ADAES shall send an HTTP POST request with a Request-URI according to the pattern "{apiRoot}/adae-sc/\<apiVersion\>/ue-group-loc-analytics" and with a body containing data type LocRelUeGroupSub as defined in clause 7.10.9.4.2.2 of 3GPP TS 29.549 \[9\].

Upon receipt of the HTTP POST request, the ADAEC shall:

a\) verify the identity of the ADAES and determine if the ADAES is authorized to obtain the location-related UE group analytics; and

b\) if the ADAES:

1\) is not authorized, the ADAEC shall respond to the ADAES with an appropriate error status code; or

2\) is authorized, the ADAC shall create a new "Individual location-related UE group analytics subscription" resource and respond to the ADAES with an HTTP "201 Created" status code with a Location header field containing the URI of the created "Individual location-related UE group analytics subscription" resource and the LocRelUeGroupSub data structure in the response body containing a representation of the created resource as defined in clause 7.1.3.

#### 6.7.2.3 Notify_UE_Group_Location


##### 6.7.2.3.1 General

This service operation is used by the ADAEC to notify the ADAES about the location-related UE group analytics event.

##### 6.7.2.3.2 Notifying location-related UE group analytics event using Notify_UE_Group_Location service operation

To notify location-related UE group analytics event, the ADAEC shall send an HTTP POST request with a Request-URI according to the pattern "{notifUri}" and with a body containing data type LocRelUeGroupNotif as defined in clause 7.10.9.4.2.3 of 3GPP TS 29.549 \[9\].

Upon receipt of the HTTP POST request, the ADAES shall respond to the ADAEC with:

a\) if the request is successfully processed, a "204 No Content" status code and process the event notification; or

b\) if error occurs when processing the request, an appropriate error response as specified in clause 7.1.6.

#### 6.7.2.4 Unsubscribe_UE_Group_Location


##### 6.7.2.4.1 General

This service operation is used by the ADAEC to unsubscribe from the location-related UE group analytics event.

##### 6.7.2.4.2 Unsubscribing from location-related UE group analytics event using Unsubscribe_UE_Group_Location service operation

To unsubscribe from location-related UE group analytics event, the ADAES shall send an HTTP DELETE request to the "Individual location-related UE group analytics subscription" resource as specified in clause 7.1.3.12.

Upon receiving the HTTP DELETE request:

a\) the ADAEC shall verify the identity of the ADAES and check if the ADAES is authorized to unsubscribe from the location-related UE group analytics event associated with the resource URI "{apiRoot}/adae-sc/\<apiVersion\>/ue-group-loc-analytics/{ueGroupLocId}";

b\) if the ADAES is authorized to unsubscribe from the location-related UE group analytics event, the ADAEC shall delete the resource pointed by the resource URI "{apiRoot}/adae-sc/\<apiVersion\>/ue-group-loc-analytics/{ueGroupLocId}";

c\) if the request is successfully processed, the ADAEC shall respond to the ADAES with a "204 No Content" status code; and

d\) if error occurs when processing the request, the ADAEC shall respond to the ADAES with an appropriate error response as specified in clause 7.1.6.
