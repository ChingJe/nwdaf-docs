---
spec: TS 23.288
version: 20.2.0
release: '20'
clause: 12
title: 12 NEF Services to support network data analytics
source_archive: 23288-k20.zip
source_document: 23288-k20.docx
content_origin: 3gpp-source
---

# 12 NEF Services to support network data analytics


## 12.1 General

Table 12.1-1 illustrates the NEF Services to support network data analytics.

Table 12.1-1: NF services provided by NEF to support network data analytics

| Service Name        | Service Operations | Operation Semantics | Example Consumer(s) |
|---------------------|--------------------|---------------------|---------------------|
| Nnef_VFLTraining    | Subscribe          | Subscribe / Notify  | NWDAF               |
|                     | Unsubscribe        |                     | NWDAF               |
|                     | Notify             |                     | NWDAF               |
|                     | Request            | Request / Response  | NWDAF, AF           |
| Nnef_VFLInference   | Subscribe          | Subscribe / Notify  | NWDAF               |
|                     | Unsubscribe        |                     | NWDAF               |
|                     | Notify             |                     | NWDAF               |
|                     | Request            | Request / Response  | NWDAF               |
| Nnef_VFLNFdiscovery | NwdafDiscovery     | Request / Response  | AF                  |
|                     | NwdafRelease       | Request / Response  | AF                  |
| Nnef_Inference      | Subscribe          | Subscribe / Notify  | NWDAF               |
|                     | Unsubscribe        |                     | NWDAF               |
|                     | Notify             |                     | NWDAF               |
|                     | Request            | Request / Response  | NWDAF               |
| Nnef_Training       | Subscribe          | Subscribe / Notify  | NWDAF               |
|                     | Unsubscribe        |                     | NWDAF               |
|                     | Notify             |                     | NWDAF               |

## 12.2 Nnef_VFLTraining Service


### 12.2.1 General

**Service Description:** This service is provided by an NEF on behalf of either an NWDAF or AF acting as VFL client in training process as defined in clause 6.2H.2.3.

For VFL, this service may also be used by the consumer (i.e. FL Server) to prepare the VFL training as described in in clause 6.2H.2.2.

### 12.2.2 Nnef_VFLTraining_Subscribe service operation

**Service operation name:** Nnef_VFLTraining_Subscribe

**Description:** Subscribes to VFL ML Model training with NWDAF or untrusted AF as VFL client.

**Inputs, Required:**

For new subscription:

\- Analytics ID.

\- VFL correlation ID.

\- Notification Target Address (+ Notification Correlation ID).

When updating a subscription:

\- Subscription Correlation ID.

\- For NWDAF as VFL client, external NWDAF ID.

\- For untrusted AF as VFL client, AF ID.

**Inputs, Optional:** See clause 6.2H.3 for parameters.

**Outputs Required:** When the request is accepted: Subscription Correlation ID (required for management of this subscription). When the request is not accepted, an error response with cause code.

NOTE: The detail reasons in the cause code are up to Stage 3.

**Outputs, Optional:** First corresponding report (i.e. client intermediate training result) is included, if available and if consumer requested immediate reporting (see clause 4.15.1 of TS 23.502 \[3\]).

### 12.2.3 Nnef_VFLTraining_Unsubscribe service operation

**Service operation name:** Nnef_VFLTraining_Unsubscribe

**Description:** Terminate AF VFL training.

**Inputs, Required:** Subscription Correlation ID.

**Inputs, Optional:** None.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** Cause code.

### 12.2.4 Nnef_VFLTraining_Notify service operation

**Service operation name:** Nnef_VFLTraining_Notify

**Description:** NEF notifies the consumer of client intermediate training result of the local ML mode.

**Inputs, Required:**

\- Notification Correlation Information.

**Inputs, Optional:**

\- Client intermediate training result.

\- Delta list, indicate which samples will not be part of the rest of training procedure.

\- Local ML model accuracy monitoring information.

\- Leave request.

\- Iteration number.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** None.

### 12.2.5 Nnef_VFLTraining_Request service operation

**Service operation name:** Nnef_VFLTraining_Request

**Description:** In preparation of VFL training, requests NEF to check at untrusted AF or NWDAF acting as VFL client if it can support requirements for VFL.

**Inputs, Required:**

\- Analytics ID.

\- For NWDAF as VFL client, external NWDAF ID.

\- For untrusted AF as VFL client, AF ID.

\- VFL Interoperability indicator.

**Inputs, Optional:**

\- Suggested VFL Interoperability Information.

\- Suggested list of sample IDs.

\- Suggested feature ID(s).

\- Time window of the data samples.

\- Required minimum sample size.

**Outputs Required:** When the request is accepted: indicate of accepting the ML Model training requirements.

When the request is not accepted, an error response with cause code (e.g. VFL Client does not meet the VFL training requirements.

NOTE: The detail reasons in the cause code are up to Stage 3.

**Outputs, Optional:**

\- The list of supported Feature ID(s).

\- VFL Interoperability information that the VFL Client supports.

\- The list of sample IDs accepted within the sample IDs suggested by the VFL Server.

\- The list of supported sample IDs.

## 12.3 Nnef_VFLInference Service


### 12.3.1 General

**Service Description:** This service is provided by by an NEF on behalf of an AF acting as VFL client and enables an VFL server as consumer to request or subscribe/unsubscribe for a VFL inference.

When the subscription is accepted by the AF, the consumer receives from the NWDAF an identifier (Subscription Correlation ID) allowing to further manage (modify, delete) this subscription.

### 12.3.2 Nnef_VFLInference_Subscribe service operation

**Service operation name:** Nnef_VFLInference_Subscribe

**Description:** Subscribe to VFL inference.

**Inputs, Required:**

For new subscription:

\- Notification Target Address (+ Notification Correlation ID).

\- VFL Correlation ID.

\- Target of VFL inference.

When updating a subscription:

\- Subscription Correlation ID.

\- For NWDAF as VFL client, external NWDAF ID.

\- For untrusted AF as VFL client, AF ID.

**Inputs, Optional:**

\- VFL inference filter.

\- Data time window.

\- Time when intermediate local result is needed.

\- Dataset Statistical Properties.

\- Analytics metadata request.

**Outputs Required:** When the subscription is accepted: Subscription Correlation ID (required for management of this subscription). When the subscription is not accepted, an error response.

**Outputs, Optional:** Client intermediate results.

### 12.3.3 Nnef_VFLInference_Unsubscribe service operation

**Service operation name:** Nnef_VFLInference_Unsubscribe

**Description:** Unsubscribe to VFL inference.

**Inputs, Required:** Subscription Correlation ID.

**Inputs, Optional:** None.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** None.

### 12.3.4 Nnef_VFLInference_Notify service operation

**Service operation name:** Nnef_VFLInference_Notify

**Inputs, Required:**

\- Notification Correlation Information.

**Inputs, Optional:**

\- Client intermediate results.

\- Analytics Metadata Information.

**Outputs, Required:** Operation execution result indication.

O**utputs, Optional:** None.

### 12.3.5 Nnef_VFLInference_Request service operation

**Service operation name:** Nnef_VFLInference_Request

**Description:** The consumer requests the NWDAF or untrusted AF VFL Client to perform a one-time VFL inference.

**Inputs, Required:**

\- Target of VFL inference.

\- VFL Correlation ID.

\- Analytics ID.

\- For NWDAF as VFL client, external NWDAF ID.

\- For untrusted AF as VFL client, AF ID.

**Inputs, Optional:**

\- VFL inference filter.

\- Data time window.

\- Time when intermediate local result is needed.

\- Dataset Statistical Properties.

\- Analytics metadata request.

**Outputs, Required:** If the request is accepted, then client intermediate results. When the request is not accepted, an error response.

**Outputs, Optional:**

\- Analytics Metadata Information.

## 12.4 Nnef_VFLNFDiscovery Service


### 12.4.1 General

**Service Description:** This service is provided by an NEF towards an untrusted AF acting as VFL server to enable the AF to interact with NWDAFs acting as VFL client. It enables the AF to detect NWDAFs as VFL clients as described in in clause 6.2H.2.1.

### 12.4.2 Nnef_VFLNFDiscovery_NwdafDiscovery service operation

**Service operation name:** Nnef_VFLNFDiscovery_NwdafDiscovery

**Description:** The consumer requests the NEF to discover NWDAF(s) acting as VFL client.

**Inputs, Required:**

\- Analytics ID.

\- required NF type (i.e. NWDAF type).

\- VFL capability type (i.e. VFL client).

**Inputs, Optional:**

\- VFL interoperability indicator(s).

\- Required feature IDs.

\- Time Period of Interest.

\- Service Area.

**Outputs, Required:** If the request is accepted, external NWDAF ID(s), VFL interoperability indicator(s). When the request is not accepted, an error response.

**Outputs, Optional:**

\- The list of supported Feature ID(s).

\- Service Area.

\- Time interval supporting VFL training.

### 12.4.3 Nnef_VFLNFDiscovery_NwdafRelease service operation

**Service operation name:** Nnef_VFLNFDiscovery_NwdafDiscovery

**Description:** The consumer informs the NEF that it mo longer wants .to use an assigned temporary NWDAF ID.

**Inputs, Required:**

\- external NWDAF ID.

**Inputs, Optional:** None.

**Outputs, Required:** None.

**Outputs, Optional:** None.

## 12.5 Nnef_Inference Service


### 12.5.1 General

**Service Description:** This service is provided by an NEF on behalf of an AF acting as VFL server and enables an NWDAF as consumer to request or subscribe/unsubscribe for a VFL inference.

### 12.5.2 Nnef_Inference_Subscribe service operation

**Service operation name:** Nnef_Inference_Subscribe

**Description:** Subscribe to VFL inference.

**Inputs, Required:**

For new subscription:

\- Notification Target Address (+ Notification Correlation ID).

\- Analytics ID.

\- Target of Analytics Reporting.

\- (only when the target VFL server is an untrusted AF) AF ID.

When updating a subscription:

\- Subscription Correlation ID.

**Inputs, Optional:**

\- Analytics Reporting Information (with parameters as defined in clause 6.1.3):

\- Event Reporting parameters defined in Table 4.15.1-1 of TS 23.502 \[3\].

\- Analytics Filter.

\- VFL Correlation ID.

**Outputs Required:** When the subscription is accepted: Subscription Correlation ID (required for management of this subscription). When the subscription is not accepted, an error response.

**Outputs, Optional:** First corresponding inference report is included, if available and if analytics consumer requested immediate reporting (see clause 4.15.1 of TS 23.502 \[3\]).

### 12.5.3 Nnef_Inference_Unsubscribe service operation

**Service operation name:** Nnef_Inference_Unsubscribe

**Description:** Unsubscribe to VFL inference.

**Inputs, Required:** Subscription Correlation ID.

**Inputs, Optional:** None.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** None.

### 12.5.4 Nnef_Inference_Notify service operation

**Inputs, Required:**

\- Notification Correlation Information.

**Inputs, Optional:**

\- Inference results:

\- Set of the tuple (Analytics ID, Analytics specific parameters): this parameter shall be present if output analytics are reported.

\- Validity period.

\- Confidence.

\- Analytics Metadata Information.

\- Analytics Accuracy Information.

\- Revised waiting time.

\- Termination Request: this parameter indicates that AF requests to terminate the inference subscription, i.e. AF will not provide further notifications related to this subscription, with cause value.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** None.

### 12.5.5 Nnef_Inference_Request service operation

**Service operation name:** Nnef_Inference_Request

**Description:** The consumer requests the VFL server to perform a one-time VFL inference.

**Inputs, Required:**

\- Analytics ID.

\- Target of Analytics Reporting.

\- (only when the target VFL server is an untrusted AF) AF ID.

**Inputs, Optional:**

\- Analytics Reporting Information (with parameters as defined in clause 6.1.3).

\- Analytics Filter.

\- VFL Correlation ID.

**Outputs, Required:** If the request is accepted, then VFL inference results, i.e. set of the tuple (Analytics ID, Analytics specific parameters). When the request is not accepted, an error response.

**Outputs, Optional:**

\- Validity period.

\- Confidence.

\- Analytics Metadata Information.

\- Analytics Accuracy Information.

## 12.6 Nnef_Training Service


### 12.6.1 General

**Service Description:** This service is provided by NEF AF acting on behalf of an untrusted AF as VFL server and enables an NWDAF as consumer to request the AF to perform model training as defined in clause 6.2H.2.3 under the supervision of the consumer.

### 12.6.2 Nnef_Training_Subscribe service operation

**Service operation name:** Nnef_Training_Subscribe

**Description:** Subscribes to ML Model training with AF as VFL server.

**Inputs, Required:**

For new subscription:

\- Analytics ID as defined in Table 7.1-2.

\- Notification Target Address (+ Notification Correlation ID).

\- AF ID.

When updating a subscription:

\- Subscription Correlation ID.

**Inputs, Optional:** See clause 6.2H.3 for parameters.

**Outputs Required:** When the request is accepted: Subscription Correlation ID (required for management of this subscription). When the request is not accepted, an error response with cause code.

NOTE: The detail reasons in the cause code are up to Stage 3.

**Outputs, Optional:** None.

### 12.6.3 Nnef_Training_Unsubscribe service operation

**Service operation name:** Nnef_Training_Unsubscribe

**Description:** Terminate AF ML Model training.

**Inputs, Required:** Subscription Correlation ID.

**Inputs, Optional:** None.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** Cause code.

### 12.6.4 Nnef_Training_Notify service operation

**Service operation name:** Nnef_Training_Notify

**Description:** AF notifies the consumer of training progress.

**Inputs, Required:**

\- Notification Correlation Information.

**Inputs, Optional:**

\- Indication whether Training is complete.

\- Estimated VFL training completion time.

\- VFL correlation ID.

\- VFL status report.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** None.
