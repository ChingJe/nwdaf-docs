---
spec: TS 23.288
version: 18.13.0
release: '18'
clause: '8'
title: 8 DCCF Services
source_archive: 23288-id0.zip
source_document: 23288-id0.docx
source_archive_sha256: 7116a1f4123dcf949818ab56feb6b242eda8819877346ab045654e1d0d8c36aa
source_document_sha256: 5e4ca6e4e350f62a5d4e4443d36ae93c9df679e1e9aea20ccaa44a4b0eb3e196
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 8 DCCF Services


## 8.1 General


Table 8.1-1 shows the DCCF services and DCCF service operations.

Table 8.1-1: NF services provided by DCCF

| Service Name            | Service Operations | Operation Semantics | Example Consumer(s)                            |
|-------------------------|--------------------|---------------------|------------------------------------------------|
| Ndccf_DataManagement    | Subscribe          | Subscribe / Notify  | NWDAF, PCF, NSSF, AMF, SMF, NEF, AF, ADRF, LMF |
|                         | Unsubscribe        |                     | NWDAF, PCF, NSSF, AMF, SMF, NEF, AF, ADRF, LMF |
|                         | Notify             |                     | NWDAF, PCF, NSSF, AMF, SMF, NEF, AF, ADRF, LMF |
|                         | Fetch              | Request / Response  | NWDAF, PCF, NSSF, AMF, SMF, NEF, AF, ADRF, LMF |
|                         | Transfer           | Request / Response  | DCCF                                           |
| Ndccf_ContextManagement | Register           | Request / Response  | NWDAF, ADRF                                    |
|                         | Update             | Request / Response  | NWDAF, ADRF                                    |
|                         | Deregister         | Request / Response  | NWDAF, ADRF                                    |

## 8.2 Ndccf_DataManagement service


### 8.2.1 General


**Service Description:** This service enables the consumer to subscribe/unsubscribe for data or analytics via the DCCF, be notified about data or analytics exposed by the DCCF, fetch the subscribed data and have data delivered via the DCCF or via a messaging framework. Historical data, or runtime data may be obtained using this service.

When the subscription is accepted by the DCCF, the consumer NF receives from the DCCF an identifier (Subscription Correlation ID) allowing it to further manage (modify, delete) the subscription.

### 8.2.2 Ndccf_DataManagement_Subscribe service operation


**Service operation name:** Ndccf_DataManagement_Subscribe

**Description:** The consumer subscribes to receive data or analytics (which is regarded as a kind of data), or if the data is already requested from the DCCF, then the subscription is updated. The subscription includes service operation specific parameters that identify the data or analytics to be provided and may include formatting and processing instructions that specify how the data is to be delivered to the consumer. The consumer may also request that data be stored in an ADRF or an NWDAF hosting ADRF functionality. When historical data is being obtained, the consumer may specify the ID of the ADRF or NWDAF containing the data.

**Inputs, Required:** Service operation, Analytics Specification or Data Specification, Notification Target Address(es) (+ Notification Correlation ID (s)).

**Inputs, Optional:** Time Window, NF (or NF-Set) ID, ADRF or NWDAF hosting ADRF information where collected data are to be stored, ADRF ID where historical data are stored, Formatting Instructions, Processing Instructions, user consent check information (i.e. an indication that the data consumer has checked user consent), purpose for data collection, Storage Handling Information.

"Service Operation" identifies the service used by the DCCF to request data or analytics from a Data Source (e.g.: Namf_EventExposure_Subscribe or Nnwdaf_AnalyticsSubscription_Subscribe)

"Analytics Specification or Data Specification" is the "Service Operation" specific required and optional input parameters that identify the data to be collected (e.g. Analytics ID(s) / Event ID (s), Target of Analytics Reporting or Target of Event Reporting, Analytics Filter or Event Filter, etc.). Service Operations and input parameters are defined in clause 7 for NWDAF and in TS 23.502, clause 5.2 for the other NFs.

"Time Window" is the start and stop time when the requested data or analytics was or will be collected. If the Time Window includes a period in the past, then the data or analytics collection is "historical". If the Time Window includes a period in the future, the data or analytics collection is "runtime".

NOTE 1: Time Window parameter is different from the "Analytics target period" defined in clause 6.1.3.

NOTE 2: When Time Window is not provided, the consumer subscribes to runtime data or analytics collection, with no end time specified.

NF (or NF-Set) ID specifies a data source that may provide the data.

ADRF Information specifies that collected data or analytics is to be stored in an ADRF and optionally an ADRF or NWDAF ID.

Formatting Instructions and Processing Instructions are as defined in clause 5A.4.

Storage Handling Information is described in clause 5B.1.

**Outputs Required:** When the subscription is accepted: Subscription Correlation ID (required for management of this subscription). When the subscription is not accepted: An error response.

**Outputs, Optional:** First corresponding event report is included, if available (see clause 4.15.1 of TS 23.502 \[3\]), Requested data, Storage Approach (see clause 5B.1).

NOTE 3: When the Target of Event Reporting or Target of Reporting is a SUPI or a GPSI then the subscription may not be accepted, e.g. for user consent is not granted and an error is sent to the consumer. When the Target of Event Reporting or Target of Reporting is an Internal Group Id, or a list of SUPIs/GPSI(s) or any UE, no error is sent, but a SUPI or GPSI is skipped if user consent is not granted.

### 8.2.3 Ndccf_DataManagement_Unsubscribe service operation


**Service operation name:** Ndccf_DataManagement_Unsubscribe

**Description:** The consumer unsubscribes to DCCF for data or analytics.

Inputs, Required: Subscription Correlation ID.

**Inputs, Optional:** None.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** The pending (events) data that is not sent to the consumer yet.

### 8.2.4 Ndccf_DataManagement_Notify service operation


**Service operation name:** Ndccf_DataManagement_Notify

**Description:** DCCF notifies the consumer instance of the requested data or analytics (which is regarded as a kind of data) according to the request, or notifies of the availability of previously subscribed Data or Analytics when data delivery is via the DCCF. The DCCF may also notify the consumer instance when Data or Analytics is to be deleted.

**Inputs, Required:** Notification Correlation Information, time stamp representing time when DCCF completed preparation of the requested data.

**Inputs, Optional:** Requested Data with timestamp or Analytics with timestamp, Fetch Instructions, Termination Request, Data or Analytics Deletion Alert.

NOTE 1: If the DCCF has received the notifications from another source (e.g. NWDAF) without a timestamp, then the DCCF adds itself a timestamp based on the time it received the notification.

Fetch Instructions indicate whether the data or analytics are to be fetched by the Consumer. If the data or analytics are to be fetched, the fetch instructions include an address from which the data may be fetched, one or more Fetch Correlation IDs and a deadline to fetch the data (Fetch Deadline).

Termination Request indicates that the DCCF requests to terminate the data management subscription, i.e. DCCF will not provide further notifications related to this subscription.

Pending Notification Cause indicates the cause of the pending notification of the stored unsent data, e.g. the data cannot be collected any more due to UE moved out of DCCF serving area.

Data or Analytics Deletion Alert is described in clause 5B.1.

NOTE 2: Data or Analytics provided in notifications are processed and formatted according to the Processing and Formatting Instructions provided by the Consumer in Ndccf_DataManagement_Subscribe.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** Data or Analytics Retrieval Indication.

The Data or Analytics Retrieval Indication may be sent if the notification contains a Data or Analytics Deletion Alert. The Data or Analytics Retrieval Indication indicates if the consumer will retrieve stored data or analytics prior to deletion (see clause 6.2B.3).

### 8.2.5 Ndccf_DataManagement_Fetch service operation


**Service operation name:** Ndccf_DataManagement_Fetch

**Description:** Consumer retrieves from the DCCF, data or analytics (which is regarded as a kind of data) as indicated by Ndccf_DataManagement_Notify Fetch Instructions.

**Inputs, Required:** Set of Fetch Correlation ID(s).

**Inputs, Optional:** None.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** Requested data or Analytics.

### 8.2.6 Ndccf_DataManagement_Transfer service operation


**Service operation name:** Ndccf_DataManagement_Transfer

**Description:** Source DCCF transfers UE data subscription context to the target DCCF.

**Inputs, Required:** UE data subscription context(UE ID, Data Specification, Notification Target Address(es) (+ Notification Correlation ID(s))).

**Inputs, Optional:** Service Operation, Time Window, NF (or NF-Set) ID, ADRF or NWDAF hosting ADRF information where data are to be stored.

**Outputs Required:** Subscription Correlation ID, Operation execution result indication.

**Outputs, Optional:** None.

## 8.3 Ndccf_ContextManagement service


### 8.3.1 General


**Service Description:** This service enables the consumer to register collected data or analytics with the DCCF.

When the DCCF is configured by the consumer NF, the DCCF supplies a Transaction Reference Id. The Consumer NF may use the Transaction Reference Id in subsequent transactions to update or delate the context in the DCCF.

### 8.3.2 Ndccf_ContextManagement_Register service operation


**Service operation name:** Ndccf_ContextManagement_Register

**Description:** The consumer NF uses this service operation to register data or analytics it is collecting to the DCCF. The registration includes a service operation specific Analytics/Data Specification that identifies the data or analytics that are being collected or has been collected.

**Inputs, Required:** Service Operation, Analytics/Data Specification, NWDAF ID or ADRF ID.

**Inputs, Optional:** None

NOTE: The input parameters are defined as:

\- "Service Operation" identifies the service used to collect the data or analytics from a Data Source (e.g. Namf_EventExposure_Subscribe or Nnwdaf_AnalyticsSubscription_Subscribe).

\- "Analytics/Data Specification" is the "Service Operation" specific required and optional input parameters that identify the collected data (i.e. Analytics ID(s) / Event ID (s), Target of Analytics Reporting or Target of Event Reporting, Analytics Filter or Event Filter, etc.). NF Service Operations and input parameters are defined in clause 7 and clause 5.2 of TS 23.502 \[3\].

\- NWDAF ID or ADRF ID specify the ADRF or NWDAF with the stored data.

**Outputs Required:** Transaction Reference ID(s), Operation execution result indication.

**Outputs, Optional:** None.

### 8.3.3 Ndccf_ContextManagement_Update service operation


**Service operation name:** Ndccf_ContextManagement_Update

**Description:** The consumer NF uses this service operation to update a registration of data or analytics to the DCCF. The registration update includes a service operation specific Analytics/Data Specification that identifies the data or analytics that is being collected or has been collected.

**Inputs, Required:** Transaction Reference ID(s), Service Operation, Analytics/Data Specification

**Inputs, Optional:** None

NOTE: The input parameters are defined in clause 8.3.2.

**Outputs Required:** Transaction Reference ID(s), Operation execution result indication.

### 8.3.4 Ndccf_ContextManagement_Deregister service operation


**Service operation name:** Ndccf_ContextManagement_Deregister

**Description:** The consumer NF uses this service operation to delete a registration of data or analytics to the DCCF.

**Inputs, Required:** Transaction Reference ID(s)

**Inputs, Optional:** None

**Outputs Required:** Transaction Reference ID(s), Operation execution result indication.

**Outputs, Optional:** None.
