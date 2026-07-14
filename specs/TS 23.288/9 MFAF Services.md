---
spec: TS 23.288
version: 18.13.0
release: '18'
clause: '9'
title: 9 MFAF Services
source_archive: 23288-id0.zip
source_document: 23288-id0.docx
source_archive_sha256: 7116a1f4123dcf949818ab56feb6b242eda8819877346ab045654e1d0d8c36aa
source_document_sha256: 5e4ca6e4e350f62a5d4e4443d36ae93c9df679e1e9aea20ccaa44a4b0eb3e196
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 9 MFAF Services


## 9.1 General


Table 9.1-1 shows the MFAF services and MFAF service operations.

Table 9.1-1: NF services provided by MFAF

| Service Name            | Service Operations | Operation Semantics | Example Consumer(s)                            |
|-------------------------|--------------------|---------------------|------------------------------------------------|
| Nmfaf_3daDataManagement | Configure          | Request / Response  | DCCF, NWDAF                                    |
|                         | Deconfigure        | Request / Response  | DCCF, NWDAF                                    |
| Nmfaf_3caDataManagement | Notify             | Subscribe / Notify  | NWDAF, PCF, NSSF, AMF, SMF, NEF, AF, ADRF, LMF |
|                         | Fetch              | Request / Response  | NWDAF, PCF, NSSF, AMF, SMF, NEF, AF, ADRF, LMF |
| Nmfaf_ContextManagement | Transfer           | Request / Response  | MFAF                                           |

## 9.2 Nmfaf_3daDataManagement service


### 9.2.1 General


Service Description: The consumer (e.g. DCCF) uses this service to instruct the MFAF to map data or analytics received by the MFAF to out-bound notification endpoints. Configuration of the MFAF by the consumer may include formatting and processing instructions for each notification endpoint as described in clause 5A.4. The sending of historical data or run-time data may be configured/deconfigured using this service.

When the MFAF is configured by the consumer NF, the MFAF provides a Transaction Reference Id. The Consumer NF may use the Transaction Reference Id in subsequent transactions to modify or remove (deconfigure) the sending of data to consumers.

### 9.2.2 Nmfaf_3daDataManagement_Configure service operation


**Service operation name:** Nmfaf_3daDataManagement_Configure

**Description:** The consumer configures or reconfigures the MFAF to map data or analytics received by the MFAF to out-bound notification endpoints and to format and process the out-bound data or analytics. The consumer may also use this service operation with a target MFAF to request that it transfer MFAF UE context(s) from the indicated MFAF.

**Inputs, Required:** None.

**Inputs, Optional:** Data Consumer or Analytics Consumer Information, Formatting Instructions, Processing Instructions, MFAF Notification Information, Transaction Reference Id, ADRF ID, MFAF Transfer Information.

"Data Consumer or Analytics Consumer Information" contains for each notification endpoint, the consumer provided Notification Target Address (+ Analytics Consumer Notification Correlation ID) or other endpoint addresses if provisioned on the DCCF to be used by the MFAF when sending notifications. The consumer includes this information except when initiating a UE context transfer between a source MFAF and a target MFAF.

"MFAF Notification Information" is used to identify Event Notifications received from a Data Source and comprises the MFAF Notification Target Address (+ MFAF Notification Correlation ID). If a Data Source is already supplying the data to the MFAF, the MFAF Notification Information previously provided by the MFAF and used by the DCCF to obtain data from a Data Source is provided as an Input. If a new subscription to a Data Source is needed, the MFAF Notification Information is not specified as an Input and the MFAF provides Notification Information as an output. The MFAF Notification Information may subsequently be used by the DCCF when subscribing to a Data Source. "ADRF ID" is used to identify the ADRF when DCCF requests that the Messaging Framework to store historical data and analytics in the ADRF via Nadrf_DataManagement_StorageRequest. When a Notification Correlation ID is provided, MFAF shall use the Nmfaf_3caDataManagement_Notify service with the notification correlation ID to send the data or analytics to the ADRF.

"MFAF Transfer Information" consists of a Transfer Initiation flag together with ID of the MFAF currently holding the UE context, UE MFAF Context Identifier(s) (i.e. the Transaction Reference ID(s) that was returned to the consumer when using the Nmfaf_3daDataManagement_Configure service operation to configure or update the MFAF).

**Outputs Required:** Operation execution result indication.

**Outputs, Optional:** MFAF Notification Information, Transaction Reference Id.

### 9.2.3 Nmfaf_3daDataManagement_Deconfigure service operation


**Service operation name:** Nmfaf_3daDataManagement_Deconfigure

**Description:** The consumer configures the MFAF to stop mapping data or analytics received by the MFAF to one or more out-bound notification endpoints.

**Inputs, Required:** Data Consumer or Analytics Consumer Information, Transaction Reference Id.

**Inputs, Optional:** None.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** None.

## 9.3 Nmfaf_3caDataManagement service


### 9.3.1 General


Service Description: This service is used to supply data or analytics from the MFAF to notification endpoints. Notifications may contain data or analytics, or an indication of availability of data or analytics.

### 9.3.2 Nmfaf_3caDataManagement_Notify service operation


**Service operation name:** Nmfaf_3caDataManagement_Notify

**Description:** Provides data or analytics or notification of availability of data or analytics to notification endpoints.

**Inputs, Required:** Notification Correlation Information.

**Inputs, Optional:** Requested Data with timestamp or Analytics with timestamp, Fetch Instructions.

NOTE 1: If the MFAF has received the notifications from another source (e.g. NWDAF) without a timestamp, then the MFAF adds itself a timestamp based on the time it received the notification.

Fetch Instructions indicate whether the data or analytics are to be fetched by the Consumer. If the data or analytics are to be fetched, the fetch instructions include an address from which the data may be fetched, one or more Fetch Correlation IDs and a deadline to fetch the data (Fetch Deadline).

NOTE 2: Data or Analytics provided in notifications can be processed and formatted according to the Processing and Formatting Instructions provided by the Consumer.

**Outputs, Required:** None.

**Outputs, Optional:** None.

### 9.3.3 Nmfaf_3caDataManagement_Fetch service operation


**Service operation name:** Nmfaf_3caDataManagement_Fetch

**Description:** Consumer retrieves from the MFAF, data or analytics (which is regarded as a kind of data) as indicated by Nmfaf_3caDataManagement_Notify Fetch Instructions.

**Inputs, Required:** Set of Fetch Correlation ID(s).

**Inputs, Optional:** None.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** Requested Data or Analytics.

NOTE: Data or Analytics provided in notifications can be processed and formatted according to the Processing and Formatting Instructions provided by the Consumer.

## 9.4 Nmfaf_ContextManagement service


### 9.4.1 General


Service Description: This service is used to transfer MFAF UE context information to a consumer (e.g. a target MFAF). It may be triggered by a Nmfaf_3daDataManagement_Configure request.

### 9.4.2 Nmfaf_ContextManagement_Transfer service operation


**Service operation name:** Nmfaf_ContextManagement_Transfer

**Description:** Provides MFAF UE context to a consumer.

**Inputs, Required:** (Set of) UE MFAF Context Identifier(s) (i.e. the Transaction Reference ID that was returned to the consumer when using the Nmfaf_3daDataManagement_Configure service operation to configure or update the MFAF).

NOTE: These identifiers can have been received from the DCCF via the Nmfaf_3daDataManagement_Configure request (see clause 6.2.6.3.8).

**Inputs, Optional:** None.

**Outputs, Required:** Operation execution result indication. For success Set of MFAF Context(s) (i.e. for each Transaction Reference ID provided as input Data Consumer Information, optionally Formatting Instructions and Processing Instructions, and possible buffered partial contents of pending notifications).

**Outputs, Optional:** None.
