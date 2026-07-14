---
spec: TS 23.288
version: 18.13.0
release: '18'
clause: '10'
title: 10 ADRF Services
source_archive: 23288-id0.zip
source_document: 23288-id0.docx
source_archive_sha256: 7116a1f4123dcf949818ab56feb6b242eda8819877346ab045654e1d0d8c36aa
source_document_sha256: 5e4ca6e4e350f62a5d4e4443d36ae93c9df679e1e9aea20ccaa44a4b0eb3e196
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 10 ADRF Services


## 10.1 General


Table 10.1-1 shows the ADRF services and ADRF service operations.

ADRF service operations may be used to store data or analytics in the ADRF, retrieve data or analytics from an ADRF, or delete data or analytics from an ADRF.

ADRF service operations may also be used to store ML Model(s) or ML Model address(es) in the ADRF, retrieve ML Model(s) from an ADRF or delete ML Model(s) from an ADRF.

Table 10.1-1: NF services provided by ADRF

| Service Name            | Service Operations         | Operation Semantics | Example Consumer(s) |
|-------------------------|----------------------------|---------------------|---------------------|
| Nadrf_DataManagement    | StorageRequest             | Request / Response  | DCCF, NWDAF, MFAF   |
|                         | StorageSubscriptionRequest | Request / Response  | DCCF, NWDAF         |
|                         | StorageSubscriptionRemoval | Request / Response  | DCCF, NWDAF         |
|                         | RetrievalRequest           | Request / Response  | DCCF, NWDAF         |
|                         | RetrievalSubscribe         | Subscribe / Notify  | DCCF, NWDAF         |
|                         | RetrievalUnsubscribe       |                     | DCCF, NWDAF         |
|                         | RetrievalNotify            |                     | DCCF, NWDAF         |
|                         | Delete                     | Request / Response  | DCCF, NWDAF         |
| Nadrf_MLModelManagement | StorageRequest             | Request / Response  | NWDAF               |
|                         | RetrievalRequest           | Request / Response  | NWDAF               |
|                         | Delete                     | Request / Response  | NWDAF               |

## 10.2 Nadrf_DataManagement service


### 10.2.1 General


Service Description: This service enables the consumer to store, retrieve and remove data or analytics from an ADRF.

### 10.2.2 Nadrf_DataManagement_StorageRequest service operation


**Service operation name:** Nadrf_DataManagement_StorageRequest

**Description:** The consumer NF uses this service operation to request the ADRF to store data or analytics. Data or analytics are provided to the ADRF in the request message.

**Inputs, Required:** Data with timestamp or Analytics with timestamp to be stored, Service operation, Analytics Specification or Data Specification.

"Service Operation" identifies the service used to obtain the data or analytics from a Data Source (e.g. Namf_EventExposure_Subscribe or Nnwdaf_AnalyticsSubscription_Subscribe).

"Analytics Specification or Data Specification" is the "Service Operation" specific required and optional input parameters that identify the data that was stored (e.g. Analytics ID(s) / Event ID (s), Target of Analytics Reporting or Target of Event Reporting, Analytics Filter or Event Filter, etc.). Service Operations and input parameters are defined in clause 7 for NWDAF and in clause 5.2 of TS 23.502 \[3\] for the other NFs.

**Inputs, Optional:** DataSetTag, DSC information, Storage Handling Information, Data Deletion Notification Endpoint (see clause 6.2B.2).

**Outputs Required:** Result Indication.

**Outputs, Optional:** Storage Transaction Identifier, DataSetTag(s), Storage Approach.

### 10.2.3 Nadrf_DataManagement_StorageSubscriptionRequest service operation


**Service operation name:** Nadrf_DataManagement_StorageSubscriptionRequest

**Description:** The consumer (NWDAF or DCCF) uses this service operation to request the ADRF to initiate a subscription for data or analytics (see clause 6.2B.3). Data or analytics provided in notifications as a result of the subsequent subscription by the ADRF are stored in the ADRF.

This service operation provides parameters needed by the ADRF to initiate the subscription (to a DCCF or NWDAF).

**Inputs, Required:** Service operation, Analytics Specification or Data Specification, Target NF (or Set) to subscribe to for notifications.

"Service Operation" identifies the service used to request data or analytics from a Data Source (e.g. Namf_EventExposure_Subscribe or Nnwdaf_AnalyticsSubscription_Subscribe)

"Analytics Specification or Data Specification" is the "Service Operation" specific required and optional input parameters that identify the data to be collected (e.g. Analytics ID(s) / Event ID (s), Target of Analytics Reporting or Target of Event Reporting, Analytics Filter or Event Filter, etc.). Service Operations and input parameters are defined in clause 7 for NWDAF and in clause 5.2 of TS 23.502 \[3\] for the other NFs.

"Target NF (or Set) to subscribe to for notifications" may be a DCCF or NWDAF that can provide the data or analytics

**Inputs, Optional:** Formatting Instructions, Processing Instructions, DataSetTag, Storage Handling Information, Data Deletion Notification Endpoint (see clause 6.2B.3).

Formatting Instructions and Processing Instructions are as defined in clause 5A.4.

**Outputs Required:** Transaction Reference ID, DataSetTag(s) (if any).

**Outputs, Optional:** Storage Approach.

NOTE: The parameters used in this service operation, including Formatting and Processing Instructions (if provided) are used by the ADRF when subscribing to a DCCF or NWDAF for Data or Analytics to be stored.

### 10.2.4 Nadrf_DataManagement_StorageSubscriptionRemoval service operation


**Service operation name:** Nadrf_DataManagement_StorageSubscriptionRemoval

**Description:** The consumer NF uses this service operation to request that the ADRF no longer subscribes to data or analytics it is collecting and storing.

**Inputs, Required:** one of the following:

\- Transaction Reference ID provided in the Nadrf_DataManagement_StorageSubscriptionRequest Output; or

\- DataSetTag.

**Inputs, Optional:** None.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** None.

### 10.2.5 Nadrf_DataManagement_RetrievalRequest service operation


**Service operation name:** Nadrf_DataManagement_RetrievalRequest

**Description:** The consumer NF uses this service operation to retrieve stored data or analytics from the ADRF. The Nadrf_DataManagement_RetrievalRequest response either contains the data or analytics, or provides instructions for fetching the data or analytics. The Nadrf_DataManagement_RetrievalRequest may be unsolicited (e.g. when the consumer itself has known "Storage Transaction Identifier") or sent in response to a Fetch Instructions received from the ADRF in an Nadrf_DataManagement_RetrievalNotify.

**Inputs, Required:** one of the following:

\- Storage Transaction Identifier; or

\- Fetch Correlation ID(s) if the RetrievalRequest is in response to a Fetch Instruction received from the ADRF in an Nadrf_DataManagement_RetrievalNotify; or

\- DataSetTag.

**Inputs, Optional:** None.

**Outputs Required:** Result Indication.

**Outputs, Optional:** Data or Analytics, DSC information.

### 10.2.6 Nadrf_DataManagement_RetrievalSubscribe service operation


**Service operation name:** Nadrf_DataManagement_RetrievalSubscribe

**Description:** The consumer NF uses this service operation to retrieve stored data or analytics from the ADRF and to receive future notifications containing the corresponding data or analytics received by ADRF.

**Inputs, Required:** one of the following:

\- Service Operation, Analytics Specification or Data Specification, Time Window; or

\- DataSetTag.

"Service Operation" identifies the service used to obtain the data or analytics from a Data Source (e.g. Namf_EventExposure_Subscribe or Nnwdaf_AnalyticsSubscription_Subscribe).

"Analytics Specification or Data Specification" is the "Service Operation" specific required and optional input parameters that identify the data that was stored (e.g. Analytics ID(s) / Event ID (s), Target of Analytics Reporting or Target of Event Reporting, Analytics Filter or Event Filter, etc.). Service Operations and input parameters are defined in clause 7 for NWDAF and in clause 5.2 of TS 23.502 \[3\] for the other NFs.

"Time Window" is the start and stop time when the requested data or analytics was collected. If Time Window includes a period in the future, subsequent notifications containing the requested data or analytics received by the ADRF are sent to the notification endpoint.

"DataSetTag" indicates all data records stored or being collected and stored by ADRF which are associated to that attribute.

**Inputs, Optional:** Fetch flag.

**Outputs Required:** Result Indication.

**Outputs, Optional:** Subscription Correlation ID.

### 10.2.7 Nadrf_DataManagement_RetrievalUnsubscribe service operation


**Service operation name:** Nadrf_DataManagement_RetrievalUnsubscribe

**Description:** The consumer NF uses this service operation to request that the ADRF no longer sends data or analytics to a notification endpoint.

**Inputs, Required:** Subscription Correlation ID.

**Inputs, Optional:** None.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** None.

### 10.2.8 Nadrf_DataManagement_RetrievalNotify service operation


**Service operation name:** Nadrf_DataManagement_RetrievalNotify

Description: This service operation provides consumers with either data or analytics from an ADRF, or instructions to fetch the data or analytics from an ADRF. The notifications are provided to consumers that have subscribed using the Nadrf_DataManagement_RetrievalSubscribe service operation. Historical data or analytics may be retrieved from ADRF storage and data received in the future be sent when obtained by the ADRF. The ADRF may also notify the consumer instance when Data or Analytics is to be deleted

**Inputs, Required:** Notification Correlation Information, time stamp representing time when ADRF completed preparation of the requested data.

**Inputs, Optional:** Requested Data or Analytics, Fetch Instructions, Termination Request, DSC information, Data or Analytics Deletion Alert.

Fetch Instructions indicate whether the data or analytics are to be fetched from the ADRF by the Consumer. If the data or analytics are to be fetched, the fetch instructions include an address from which the data may be fetched, one or more Fetch Correlation IDs. and a deadline to fetch the data (Fetch Deadline).

Data or Analytics are fetched using the Nadrf_DataManagement_RetrievalRequest service operation.

Termination Request indicates that the ADRF requests to terminate the subscription, i.e. ADRF will not provide further notifications related to this subscription, e.g. when all data or analytics requested by the consumer have been provided to the consumer.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** Data or Analytics Retrieval Indication.

The Data or Analytics Retrieval Indication may be sent if the notification contains a Data or Analytics Deletion Alert. The Data or Analytics Retrieval Indication indicates if the consumer will retrieve stored data or analytics prior to deletion (see clause 6.2B.3).

### 10.2.9 Nadrf_DataManagement_Delete


**Service operation name:** Nadrf_DataManagement_Delete

**Description:** This service operation instructs the ADRF to delete stored data.

**Inputs, Required:** One of the following:

\- Storage Transaction Identifier;

\- Service Operation, Analytics Specification or Data Specification and Time Window; or

\- DataSetTag.

**Inputs, Optional:** None.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** None.

## 10.3 Nadrf_MLModelManagement service


### 10.3.1 General


Service Description: This service enables the consumer to store and update, retrieve and remove ML Model(s) or ML Model address(es) from an ADRF.

### 10.3.2 Nadrf_MLModelManagement_StorageRequest service operation


**Service operation name:** Nadrf_MLModelManagement_StorageRequest

**Description:** The consumer NF uses this service operation to request the ADRF to store or update ML Model(s). ML Model(s) or ML Model address(es) stored in NWDAF containing MTLF are provided to the ADRF in the request message.

**Inputs, Required:** NF instance ID of the NWDAF containing MTLF and set of:

\- (Updated) ML Model(s); or

\- the tuple (unique ML Model identifier and address of (updated) ML Model file and (updated) Storage size required for each of the ML Models).

The ADRF downloads ML Model file using the ML Model addresses provided by the NWDAF containing MTLF. How the ADRF locally stores the ML Model is left for implementation.

NOTE 1: The ADRF does not store the ML Model file address(es) received from NWDAF containing MTLF.

**Inputs, Optional:** Allowed NF instance list for the ML Model identifier, Storage Transaction Identifier.

If Storage Transaction Identifier is included in the input, then ADRF updates the ML Model(s) corresponding to the Storage Transaction Identifier with the updated ML model(s) or updated ML Model file address(es) provided by consumer NF. A new Storage Transaction Identifier may be generated and sent to consumer NF by ADRF after the ML Model update process.

**Outputs Required:**

\- ML Model storage or ML Model Update Result Indication;

\- \[Conditional\] one or more tuples of unique ML Model identifier and (updated) ML Model address of Model file stored in ADRF.

NOTE 2: The definition of ML file address such as e.g. name, location and access method is up to stage 3.

**Outputs, Optional:** Storage Transaction Identifier.

### 10.3.3 Nadrf_MLModelManagement_Delete service operation


**Service operation name:** Nadrf_MLModelManagement_Delete

**Description:** This service operation instructs the ADRF to delete stored ML Model file(s) or ML Model address(es).

**Inputs, Required:**

\- Storage Transaction Identifier; or

\- one or more unique ML Model identifier(s).

When a Storage Transaction Identifier is given, ADRF shall delete all the ML models that stored in the corresponding storage transaction.

**Inputs, Optional:** None.

**Outputs, Required:** One of the following:

\- Operation execution result indication (i.e. ML Model deleted, ML Model not found, ML Model found but not deleted), if for all of the involved ML Model(s) have the same execution result.

\- One or more tuples (each includes the unique ML Model identifier and corresponding operation execution result indication).

**Outputs, Optional:** None.

### 10.3.4 Nadrf_MLModelManagement_RetrievalRequest service operation


**Service operation name:** Nadrf_MLModelManagement_RetrievalRequest

**Description:** The consumer NF uses this service operation to request the ADRF to get the ML Model(s) stored in ADRF.

**Inputs, Required:**

\- Storage Transaction Identifier; or

\- one or more tuples of unique ML Model identifier(s).

**Inputs, Optional:** None.

**Outputs Required:** Result Indication.

\- \[Conditional\] one or more tuples of unique ML Model identifiers and address of ML Model file stored in ADRF.

NOTE: The definition of address of ML Model file such as e.g. name, location and access method is up to stage 3.

**Outputs, Optional:** None.
