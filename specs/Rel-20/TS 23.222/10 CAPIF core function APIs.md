---
spec: TS 23.222
version: 20.1.0
release: '20'
clause: 10
title: 10 CAPIF core function APIs
source_archive: 23222-k10.zip
source_document: 23222-k10.docx
content_origin: 3gpp-source
---

# 10 CAPIF core function APIs


## 10.1 General

Table 10.1-1 illustrates the CAPIF core function APIs.

Table 10.1-1: List of CAPIF core function APIs

| API Name                                 | API Operations                  | Known Consumer(s)                                                                    | Communication Type |
|------------------------------------------|---------------------------------|--------------------------------------------------------------------------------------|--------------------|
| CAPIF_Open_Discover_Service_API          | Open_Discover_Service_API       | Open                                                                                 | Request/Response   |
| CAPIF_Discover_Service_API               | Discover_Service_API            | API Invoker, CAPIF core function                                                     | Request/ Response  |
|                                          | Subscribe_Event                 | API Invoker                                                                          | Request/ Response  |
|                                          | Update_Event_Subscription       | API Invoker                                                                          | Request/ Response  |
|                                          | Notify_Event                    | API Invoker                                                                          | Notify             |
|                                          | Unsubscribe_Event               | API Invoker                                                                          | Request/ Response  |
| CAPIF_Publish_Service_API                | Publish_Service_API             | API Publishing Function, CAPIF core function                                         | Request/ Response  |
|                                          | Unpublish_Service_API           | API Publishing Function, CAPIF core function                                         | Request/ Response  |
|                                          | Update_Service_API              | API Publishing Function, CAPIF core function                                         | Request/ Response  |
|                                          | Get_Service_API                 | API Publishing Function, CAPIF core function                                         | Request/ Response  |
|                                          | Subscribe_Event                 | API Publishing Function                                                              | Request/ Response  |
|                                          | Update_Event_Subscription       | API Publishing Functionr                                                             | Request/ Response  |
|                                          | Notify_Event                    | API Publishing Function                                                              | Notify             |
|                                          | Unsubscribe_Event               | API Publishing Function                                                              | Request/ Response  |
| CAPIF_Events API                         | Subscribe_Event                 | API Invoker, API Publishing Function, API Management Function, API Exposing Function | Request/ Response  |
|                                          | Update_Event_Subscription       | API Invoker, API Publishing Function, API Management Function, API Exposing Function | Request/ Response  |
|                                          | Notify_Event                    | API Invoker, API Publishing Function, API Management Function, API Exposing Function | Notify             |
|                                          | Unsubscribe_Event               | API Invoker, API Publishing Function, API Management Function, API Exposing Function | Request/ Response  |
| CAPIF_API_Invoker_management API         | Onboard_API_Invoker             | API Invoker                                                                          | Request/ Response  |
|                                          | Offboard_API_Invoker            | API Invoker                                                                          | Request/ Response  |
|                                          | Update_API_Invoker_Details      | API Invoker                                                                          | Request/ Response  |
|                                          | Subscribe_Event                 | API Management Function                                                              | Request/ Response  |
|                                          | Update_Event_Subscription       | API Management Function                                                              | Request/ Response  |
|                                          | Notify_Event                    | API Management Function                                                              | Notify             |
|                                          | Unsubscribe_Event               | API Management Function                                                              | Request/ Response  |
| CAPIF_API_Provider_Management_API        | Register_API_Provider           | API Management Function                                                              | Request/Response   |
|                                          | Update_API_Provider             | API Management Function                                                              | Request/Response   |
|                                          | Deregister_API_Provider         | API Management Function                                                              | Request/Response   |
|                                          | Disenroll_Service_APIs          | API Management Function                                                              | Request/Response   |
| CAPIF_Security API                       | Obtain_Security_Method          | API Invoker                                                                          | Request/ Response  |
|                                          | Obtain_Authorization            | API Invoker                                                                          | Request/ Response  |
|                                          | Obtain_API_Invoker_Info         | API Exposing Function                                                                | Request/ Response  |
|                                          | Revoke_Authorization            | API Exposing Function                                                                | Request/ Response  |
|                                          | Revoke_Authorization_Notify     | API Invoker                                                                          | Notify             |
| CAPIF_Monitoring API                     | Subscribe_Event                 | API Management Function                                                              | Request/ Response  |
|                                          | Update_Event_Subscription       | API Management Function                                                              | Request/ Response  |
|                                          | Notify_Monitoring_Service_Event | API Management Function                                                              | Notify             |
|                                          | Unsubscribe_Event               | API Management Function                                                              | Request/ Response  |
| CAPIF_Logging_API_Invocation API         | Log_API_Invocation              | API exposing function                                                                | Request/ Response  |
| CAPIF_Auditing API                       | Query_API_Invocation_Log        | API management function                                                              | Request/ Response  |
| CAPIF_Access_Control_Policy API          | Obtain_Access_Control_Policy    | API exposing function                                                                | Request/Response   |
| CAPIF_Routing_Info API                   | Obtain_Routing_Info             | API exposing function                                                                | Request/Response   |
| CAPIF_API_Invocation_Error_Reporting API | Report_API_Invocation_Error     | API Invoker                                                                          | Request/Response   |
| CAPIF_AEF_Exposure_Policy                | Select_Exposure_Policy          | API exposing function                                                                | Request/Response   |

## 10.2 CAPIF_Discover_Service_API API


### 10.2.1 General

**API description:** This API enables the API invoker to communicate with the CAPIF core function to discover the published service API information over CAPIF-1 or CAPIF-1e.

### 10.2.2 Discover_Service_API operation

**API operation name:** Discover_Service_API

**Description:** Provides the published service APIs information.

**Known Consumers:** API invoker.

**Inputs:** Refer subclause 8.7.2.1.

**Outputs:** Refer subclause 8.7.2.2.

See subclause 8.7.3 for the details of usage of this API operation.

### 10.2.3 Subscribe_Event operation

**API operation name:** Subscribe_Event

**Description:** Provides subscription to the CAPIF related event information.

**Known Consumers:** API invoker.

**Inputs:** Refer subclause 8.8.2.1.

**Outputs:** Refer subclause 8.8.2.2.

See subclause 8.8.3 for the details of usage of this API operation.

### 10.2.4 Notify_Event operation

**API operation name:** Notify_Event

**Description:** Provides the relevant CAPIF event information to the subscribed entities.

**Known Consumers:** API invoker.

**Inputs:** Refer subclause 8.8.2.3.

**Outputs:** Refer subclause 8.8.2.4.

See subclause 8.8.4 for the details of usage of this API operation.

### 10.2.5 Unsubscribe_Event operation

**API operation name:** Unsubscribe_Event

**Description:** Unsubscription to the CAPIF event information.

**Known Consumers:** API invoker.

**Inputs:** Refer subclause 8.8.2.5.

**Outputs:** Refer subclause 8.8.2.6.

See subclause 8.8.5 for the details of usage of this API operation.

### 10.2.6 Update_Event_Subscription operation

**API operation name:** Update_Event_Subscription

**Description:** Updates a subscription to CAPIF related event information.

**Known Consumers:** API invoker.

**Inputs:** Refer subclause 8.8.2.7.

**Outputs:** Refer subclause 8.8.2.8.

See subclause 8.8.5a for the details of usage of this API operation.

## 10.3 CAPIF_Publish_Service_API API


### 10.3.1 General

**API description:** This API enables the API publishing function to communicate with the CAPIF core function to publish the service API information and manage the published service API information over CAPIF-4.

NOTE: Stage 3 can decide whether the API for CAPIF_Publish_Service_API can be enabled over CAPIF-4.

### 10.3.2 Publish_Service_API operation

**API operation name:** Publish_Service_API

**Description:** Publish the service API information.

**Known Consumers:** API publishing function, CAPIF core function.

**Inputs:** Refer subclause 8.3.2.1, 8.25.2.1.

**Outputs:** Refer subclause 8.3.2.2, 8.25.2.2.

See subclause 8.3.3 and 8.25.3.1 for the details of usage of this API operation.

### 10.3.3 Unpublish_Service_API operation

**API operation name:** Unpublish_Service_API

**Description:** Remove the published service API information.

**Known Consumers:** API publishing function, CAPIF core function.

**Inputs:** Refer subclause 8.4.2.1, 8.25.2.5.

**Outputs:** Refer subclause 8.4.2.2, 8.25.2.6.

See subclause 8.4.3 and 8.25.3.4 for the details of usage of this API operation.

### 10.3.4 Update_Service_API operation

**API operation name:** Update_Service_API

**Description:** Update the published service API information.

**Known Consumers:** API publishing function, CAPIF core function.

**Inputs:** Refer subclause 8.6.2.1, 8.25.2.9.

**Outputs:** Refer subclause 8.6.2.2, 8.25.2.10.

See subclause 8.6.3 and 8.25.3.6 for the details of usage of this API operation.

### 10.3.5 Get_Service_API operation

**API operation name:** Get_Service_API

**Description:** Retrieve the published service API information.

**Known Consumers:** API publishing function, CAPIF core function.

**Inputs:** Refer subclause 8.5.2.1, 8.25.2.7.

**Outputs:** Refer subclause 8.5.2.2, 8.25.2.8.

See subclause 8.5.3 and 8.25.3.5 for the details of usage of this API operation.

### 10.3.6 Subscribe_Event operation

**API operation name:** Subscribe_Event

**Description:** Provides subscription to the CAPIF related event information.

**Known Consumers:** API publishing function.

**Inputs:** Refer subclause 8.8.2.1.

**Outputs:** Refer subclause 8.8.2.2.

See subclause 8.8.3 for the details of usage of this API operation.

### 10.3.7 Notify_Event operation

**API operation name:** Notify_Event

**Description:** Provides the relevant CAPIF event information to the subscribed entities.

**Known Consumers:** API publishing function.

**Inputs:** Refer subclause 8.8.2.3.

**Outputs:** Refer subclause 8.8.2.4.

See subclause 8.8.4 for the details of usage of this API operation.

### 10.3.8 Unsubscribe_Event operation

**API operation name:** Unsubscribe_Event

**Description:** Unsubscription to the CAPIF event information.

**Known Consumers:** API publishing function.

**Inputs:** Refer subclause 8.8.2.5.

**Outputs:** Refer subclause 8.8.2.6.

See subclause 8.8.5 for the details of usage of this API operation.

### 10.3.9 Update_Event_Subscription operation

**API operation name:** Update_Event_Subscription

**Description:** Updates a subscription to CAPIF related event information.

**Known Consumers:** API publishing function.

**Inputs:** Refer subclause 8.8.2.7.

**Outputs:** Refer subclause 8.8.2.8.

See subclause 8.8.5a for the details of usage of this API operation.

## 10.4 CAPIF_Events API


### 10.4.1 General

**API description:** This API enables the API subscribing entity to communicate with the CAPIF core function to subscribe to and unsubscribe from CAPIF events and receive subsequent notification of CAPIF events. This API is used for the subscription to and notifications of those CAPIF events that are not bound to any of the other CAPIF core function APIs. The following are the key functionalities:

\- API invoker subscribes to CAPIF events over CAPIF-1 or CAPIF-1e.

\- API invoker receives notifications for subscribed CAPIF events over CAPIF-1 or CAPIF-1e.

\- API invoker unsubscribes from CAPIF events over CAPIF-1 or CAPIF-1e.

\- API invoker updates subscriptions for CAPIF events over CAPIF-1 or CAPIF-1e.

\- API exposing function subscribes to CAPIF events over CAPIF-3.

\- API exposing function receives notifications for subscribed CAPIF events over CAPIF-3.

\- API exposing function unsubscribes from CAPIF events over CAPIF-3.

\- API exposing function updates subscriptions for CAPIF events over CAPIF-3.

\- API publishing function subscribes to CAPIF events over CAPIF-4.

\- API publishing function receives notifications for subscribed CAPIF events over CAPIF-4.

\- API publishing function unsubscribes from CAPIF events over CAPIF-4.

\- API publishing function updates subscriptions for CAPIF events over CAPIF-4.

\- API management function subscribes to CAPIF events over CAPIF-5.

\- API management function receives notifications for subscribed CAPIF events over CAPIF-5.

\- API management function unsubscribes from CAPIF events over CAPIF-5.

\- API management function updates subscriptions for CAPIF events over CAPIF-5.

NOTE: Stage 3 can further decide if CAPIF_Events API can be further fine grained into more APIs.

### 10.4.2 Subscribe_Event operation

**API operation name:** Subscribe_Event

**Description:** Provides subscription to the CAPIF related event information.

**Known Consumers:** API invoker, API publishing function, API management function, API exposing function.

**Inputs:** Refer subclause 8.8.2.1.

**Outputs:** Refer subclause 8.8.2.2.

See subclause 8.8.3 for the details of usage of this API operation.

### 10.4.3 Notify_Event operation

**API operation name:** Notify_Event

**Description:** Provides the relevant CAPIF event information to the subscribed entities.

**Known Consumers:** API invoker, API publishing function, API management function, API exposing function.

**Inputs:** Refer subclause 8.8.2.3.

**Outputs:** Refer subclause 8.8.2.4.

See subclause 8.8.4 for the details of usage of this API operation.

### 10.4.4 Unsubscribe_Event operation

**API operation name:** Unsubscribe_Event

**Description:** Unsubscription to the CAPIF event information.

**Known Consumers:** API invoker, API publishing function, API management function, API exposing function.

**Inputs:** Refer subclause 8.8.2.5.

**Outputs:** Refer subclause 8.8.2.6.

See subclause 8.8.5 for the details of usage of this API operation.

### 10.4.5 Update_Event_Subscription operation

**API operation name:** Update_Event_Subscription

**Description:** Updates a subscription to CAPIF related event information.

**Known Consumers:** API invoker, API publishing function, API management function, API exposing function.

**Inputs:** Refer subclause 8.8.2.7.

**Outputs:** Refer subclause 8.8.2.8.

See subclause 8.8.5a for the details of usage of this API operation.

## 10.5 CAPIF_API_invoker_management API


### 10.5.1 General

**API description:** This API enables the API invoker to communicate with the CAPIF core function to enroll as a registered user of CAPIF and manage the enrollment information over CAPIF-1 or CAPIF-1e.

### 10.5.2 Onboard_API_Invoker operation

**API operation name:** Onboard_API_Invoker

**Description:** Enrolls the API invoker as a recognized user of the CAPIF.

**Known Consumers:** API invoker.

**Inputs:** Refer subclause 8.1.2.1.

**Outputs:** Refer subclause 8.1.2.2.

See subclause 8.1.3 for the details of usage of this API operation.

### 10.5.2a Update_API_Invoker_Details operation

**API operation name:** Update_API_Invoker_Details

**Description:** Enrolls/disenrolls service APIs from the API invoker profile.

**Known Consumers:** API invoker.

**Inputs:** Refer subclause 8.26.2.1.

**Outputs:** Refer subclause 8.26.2.2.

See subclause 8.26.3 for the details of usage of this API operation.

### 10.5.3 Offboard_API_Invoker operation

**API operation name:** Offboard_API_Invoker

**Description:** Cancels enrollment of the API invoker as a recognized user of the CAPIF.

**Known Consumers:** API invoker.

**Inputs:** Refer subclause 8.2.2.1.

**Outputs:** Refer subclause 8.2.2.2.

See subclause 8.2.3 for the details of usage of this API operation.

### 10.5.4 Subscribe_Event operation

**API operation name:** Subscribe_Event

**Description:** Provides subscription to the CAPIF related event information.

**Known Consumers:** API management function.

**Inputs:** Refer subclause 8.8.2.1.

**Outputs:** Refer subclause 8.8.2.2.

See subclause 8.8.3 for the details of usage of this API operation.

### 10.5.5 Notify_Event operation

**API operation name:** Notify_Event

**Description:** Provides the relevant CAPIF event information to the subscribed entities.

**Known Consumers:** API management function.

**Inputs:** Refer subclause 8.8.2.3.

**Outputs:** Refer subclause 8.8.2.4.

See subclause 8.8.4 for the details of usage of this API operation.

### 10.5.6 Unsubscribe_Event operation

**API operation name:** Unsubscribe_Event

**Description:** Unsubscription to the CAPIF event information.

**Known Consumers:** API management function.

**Inputs:** Refer subclause 8.8.2.5.

**Outputs:** Refer subclause 8.8.2.6.

See subclause 8.8.5 for the details of usage of this API operation.

### 10.5.7 Update_Event_Subscription operation

**API operation name:** Update_Event_Subscription

**Description:** Updates a subscription to CAPIF related event information.

**Known Consumers:** API management function.

**Inputs:** Refer subclause 8.8.2.7.

**Outputs:** Refer subclause 8.8.2.8.

See subclause 8.8.5a for the details of usage of this API operation.

## 10.6 CAPIF_Security API


### 10.6.1 General

**API description:** This API enables the API invoker to communicate with the CAPIF core function to authenticate and obtain authorization to access service APIs over CAPIF-1 or CAPIF-1e. This API also enables the API exposing function (AEF) to obtain API invoker information and revoke API invoker authorization over CAPIF-3.

### 10.6.2 Obtain_Security_Method operation

**API operation name:** Obtain_Security_Method

**Description:** Obtain information about service API security method with CAPIF core function for service API invocations.

**Known Consumers:** API invoker.

**Inputs:** Refer subclause 8.10A.2.

**Outputs:** Refer subclause 8.10A.2.

See subclause 8.10A.3 for the details of usage of this API operation.

### 10.6.3 Obtain_Authorization operation

**API operation name:** Obtain_Authorization

**Description:** Provides the authorization information to access relevant service API.

**Known Consumers:** API invoker.

**Inputs:** Refer subclause 8.11.2.

**Outputs:** Refer subclause 8.11.2.

See subclause 8.11.3 for the details of usage of this API operation.

### 10.6.4 Obtain_API_Invoker_Info operation

**API operation name:** Obtain_API_Invoker_Info

**Description:** Obtains the API invoker information.

**Known Consumers:** API exposing function.

**Inputs:** Refer subclause 8.16.2.3.

**Outputs:** Refer subclause 8.16.2.4.

See subclause 8.16.3 for the details of usage of this API operation.

### 10.6.5 Revoke_Authorization operation

**API operation name:** Revoke_Authorization

**Description:** Revokes API invoker authorization to access service API.

**Known Consumers:** API exposing function.

**Inputs:** Refer subclause 8.23.2.

**Outputs:** Refer subclause 8.23.2.

See subclause 8.23.3 for the details of usage of this API operation.

### 10.6.6 Revoke_Authorization_Notify operation

**API operation name:** Revoke_Authorization_Notify

**Description:** Provides to the API invoker authorization revocation information to access service API.

**Known Consumers:** API invoker.

**Inputs:** Refer subclause 8.23.2.3.

**Outputs:** Refer subclause 8.23.2.4.

See clauses 8.23.3 and 8.23.4 for the details of usage of this API operation.

## 10.7 CAPIF_Monitoring API


### 10.7.1 General

**API description:** This API enables the API management function to communicate with the CAPIF core function to subscribe to and unsubscribe from CAPIF events related to monitoring and receive subsequent notification of CAPIF monitoring events over CAPIF-5.

NOTE: Stage 3 can decide whether the API for CAPIF_Monitoring can be enabled over CAPIF-5.

### 10.7.2 Subscribe_Event operation

**API operation name:** Subscribe_Event

**Description:** Provides subscription to the CAPIF related event information.

**Known Consumers:** API management function.

**Inputs:** Refer subclause 8.8.2.1.

**Outputs:** Refer subclause 8.8.2.2.

See subclause 8.8.3 for the details of usage of this API operation.

### 10.7.3 Notify_Monitoring_Service_Event operation

**API operation name:** Notify_Monitoring_Service_Event

**Description:** Provides the notification of the events related to monitoring service API invocations to the subscribed API management function.

**Known Consumers:** API management function.

**Inputs:** Refer subclause 8.21.2.1.

**Outputs:** Refer subclause 8.21.2.2.

See subclause 8.21.3 for the details of usage of this API operation.

### 10.7.4 Unsubscribe_Event operation

**API operation name:** Unsubscribe_Event

**Description:** Unsubscription to the CAPIF event information.

**Known Consumers:** API management function.

**Inputs:** Refer subclause 8.8.2.5.

**Outputs:** Refer subclause 8.8.2.6.

See subclause 8.8.5 for the details of usage of this API operation.

### 10.7.5 Update_Event_Subscription operation

**API operation name:** Update_Event_Subscription

**Description:** Updates a subscription to CAPIF related event information.

**Known Consumers:** API management function.

**Inputs:** Refer subclause 8.8.2.7.

**Outputs:** Refer subclause 8.8.2.8.

See subclause 8.8.5a for the details of usage of this API operation.

## 10.8 CAPIF_Logging_API_Invocation API


### 10.8.1 General

**API description:** This API enables the API exposing function to communicate with the CAPIF core function to log the information related to service API invocation over CAPIF-3.

NOTE: Stage 3 can decide whether the API for CAPIF_Logging_API_Invocation can be enabled over CAPIF‑3.

### 10.8.2 Log_API_Invocation operation

**API operation name:** Log_API_Invocation

**Description:** Enables to log API invocation information.

**Known Consumers:** API exposing function.

**Inputs:** Refer subclause 8.19.2.1.

**Outputs:** Refer subclause 8.19.2.2.

See subclause 8.19.3 for the details of usage of this API operation.

## 10.9 CAPIF_Auditing API


### 10.9.1 General

**API description:** This API enables the API management function to communicate with the CAPIF core function to retrieve the log information related to service API invocation over CAPIF-5.

NOTE: Stage 3 can decide whether the API for CAPIF_Auditing can be enabled over CAPIF‑5.

### 10.9.2 Query\_ API_Invocation_Log operation

**API operation name:** Query_API_Invocation_Log

**Description:** Query the API invocation log information.

**Known Consumers:** API management function.

**Inputs:** Refer subclause 8.22.2.1.

**Outputs:** Refer subclause 8.22.2.2.

See subclause 8.22.3 for the details of usage of this API operation.

## 10.10 CAPIF_Access_Control_Policy API


### 10.10.1 General

**API description:** This API enables the API exposing function to obtain the policy to perform access control on the service API invocations.

### 10.10.2 Obtain_Access_Control_Policy operation

**API operation name:** Obtain_Access_Control_Policy

**Description:** Allows obtaining the policy to perform access control on the service API invocations.

**Known Consumers:** API exposing function.

**Inputs:** Refer subclause 8.12.2.1.

**Outputs:** Refer subclause 8.12.2.2.

See subclause 8.12.3 for the details of usage of this API operation.

## 10.11 CAPIF_Routing_Info API


### 10.11.1 General

**API description:** This API enables the API exposing function to obtain the routing information to forward the API invocation to another API exposing function.

### 10.11.2 Obtain_Routing_Info operation

**API operation name:** Obtain_Routing_Info

**Description:** Allows obtaining the API routing information.

**Known Consumers:** API exposing function.

**Inputs:** Refer subclause 8.27.2.1.

**Outputs:** Refer subclause 8.27.2.2.

See subclause 8.27.3 for the details of usage of this API operation.

## 10.12 CAPIF_API_provider_management API


### 10.12.1 General

**API description:** This API enables the API Management Function to communicate with the CAPIF core function to register the API provider domain functions as authorized users of the CAPIF functionalities.

### 10.12.2 Register_API_Provider operation

**API operation name:** Register_API_Provider

**Description:** Registers the API provider domain functions as authorized users of the CAPIF.

**Known Consumers:** API Management Function.

**Inputs:** Refer subclause 8.28.2.1.

**Outputs:** Refer subclause 8.28.2.2.

See subclause 8.28.3 for the details of usage of this API operation.

### 10.12.3 Update_API_Provider operation

**API operation name:** Update_API_Provider

**Description:** Updates registration information of the API provider domain functions.

**Known Consumers:** API Management Function.

**Inputs:** Refer subclause 8.29.2.1.

**Outputs:** Refer subclause 8.29.2.2.

See subclause 8.29.3 for the details of usage of this API operation.

### 10.12.4 Deregister_API_Provider operation

**API operation name:** Deregister_API_Provider

**Description:** De-registers the API provider domain functions on the CAPIF core function.

**Known Consumers:** API Management Function.

**Inputs:** Refer subclause 8.30.2.1.

**Outputs:** Refer subclause 8.30.2.2.

See subclause 8.30.3 for the details of usage of this API operation.

### 10.12.5 Disenroll_Service_APIs operation

**API operation name:** Disenroll_Servoce_APIs

**Description:** to remove enrolled service APIs for the API invoker initiated by AMF.

**Known Consumers:** API Management Function.

**Inputs:** Refer subclause 8.39.2.1.

**Outputs:** Refer subclause 8.39.2.2.

See subclause 8.39.3 for the details of usage of this API operation.

## 10.13 CAPIF_Open_Discover_Service_API API


### 10.13.1 General

**API description:** This API enables the API invoker to discover service API information about the available set of APIs offered by CCF before onboarding

### 10.13.2 Open_Discover_Service_API operation

**API operation name:** Open_Discover_Service_API

**Description:** Provides the published service APIs information before onboarding.

**Known Consumers:** Open.

**Inputs:** Refer subclause 8.38.2.1.

**Outputs:** Refer subclause 8.38.2.2.

See subclause 8.38.3 for the details of usage of this API operation.

## 10.14 CAPIF_API_Invocation_Error_Reporting API


### 10.14.1 General

**API description**: This API enables the API invoker to communicate with the CAPIF core function to report service API invocation errors over CAPIF-1 or CAPIF-1e.

### 10.14.2 Report_API_Invocation_Error operation

**API operation name**: Report_API_Invocation_Error

**Description**: Enables the API invoker to report API invocation error information to the CAPIF core function.

**Known Consumers**: API invoker.

**Inputs**: Refer subclause 8.40.2.1.

**Outputs**: Refer subclause 8.40.2.2.

See subclause 8.40.3 for the details of usage of this API operation.

## 10.15 CAPIF_AEF_Exposure_Policy API


### 10.15.1 General

**API description:** This API enables the API exposing function to fetch exposure policies (e.g. roaming considerations) during the service API invocations.

### 10.15.2 AEF_Exposure_Policy operation

**API operation name:** AEF_Exposure_Policy

**Description:** Indicates the exposure policies the AEF fetches to control the service API invocations.

**Known Consumers:** API exposing function.

**Inputs:** Refer subclause 8.42.2.1.

**Outputs:** Refer subclause 8.42.2.2.

See subclause 8.42.3 for the details of usage of this API operation.
