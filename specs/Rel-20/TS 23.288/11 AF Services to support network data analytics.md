---
spec: TS 23.288
version: 20.2.0
release: '20'
clause: 11
title: 11 AF Services to support network data analytics
source_archive: 23288-k20.zip
source_document: 23288-k20.docx
content_origin: 3gpp-source
---

# 11 AF Services to support network data analytics


## 11.1 General

Table 11.1-1 illustrates the AF Services to support network data analytics.

Table 11.1-1: NF services provided by AF to support network data analytics

| Service Name     | Service Operations | Operation Semantics | Example Consumer(s) |
|------------------|--------------------|---------------------|---------------------|
| Naf_VFLTraining  | Subscribe          | Subscribe / Notify  | NWDAF, NEF          |
|                  | Unsubscribe        |                     | NWDAF, NEF          |
|                  | Notify             |                     | NWDAF, NEF          |
|                  | Request            | Request / Response  | NWDAF, NEF          |
| Naf_VFLInference | Subscribe          | Subscribe / Notify  | NWDAF               |
|                  | Unsubscribe        |                     | NWDAF               |
|                  | Notify             |                     | NWDAF               |
|                  | Request            | Request / Response  | NWDAF               |
| Naf_Inference    | Subscribe          | Subscribe / Notify  | NWDAF, NEF          |
|                  | Unsubscribe        |                     | NWDAF, NEF          |
|                  | Notify             |                     | NWDAF, NEF          |
|                  | Request            | Request / Response  | NWDAF, NEF          |
| Naf_Training     | Subscribe          | Subscribe / Notify  | NWDAF, NEF          |
|                  | Unsubscribe        |                     | NWDAF, NEF          |
|                  | Notify             |                     | NWDAF, NEF          |

## 11.2 Naf_VFLTraining Service


### 11.2.1 General

Service Description: This service is provided by an AF acting as VFL client and enables an NWDAF VFL server or an NEF acting on its behalf as consumer to request the AF to participate in VFL model training as VFL client and train a local model as defined in clause 6.2H.2.3.

For VFL, this service may also be used by the consumer (i.e. VFL Server) to prepare the VFL training as described in in clause 6.2H.2.2.

### 11.2.2 Naf_VFLTraining_Subscribe service operation

**Service operation name:** Naf_VFLTraining_Subscribe

**Description:** Subscribes to VFL ML Model training with AF as VFL client.

**Inputs, Required:**

For new subscription:

\- Analytics ID as defined in Table 7.1-2.

\- VFL correlation ID.

\- Notification Target Address (+ Notification Correlation ID).

When updating a subscription:

\- Subscription Correlation ID

**Inputs, Optional:** See clause 6.2H.3 for parameters.

**Outputs Required:** When the request is accepted: Subscription Correlation ID (required for management of this subscription). When the request is not accepted, an error response with cause code.

NOTE: The detail reasons in the cause code are up to Stage 3.

**Outputs, Optional:** First corresponding report (i.e. client intermediate training results) is included, if available and if consumer requested immediate reporting (see clause 4.15.1 of TS 23.502 \[3\]).

### 11.2.3 Naf_VFLTraining_Unsubscribe service operation

**Service operation name:** Naf_VFLTraining_Unsubscribe

**Description:** Terminate AF VFL training.

**Inputs, Required:** Subscription Correlation ID.

**Inputs, Optional:** None.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** Cause code

### 11.2.4 Naf_VFLTraining_Notify service operation

**Service operation name:** Naf_VFLTraining_Notify

**Description:** AF notifies the consumer of client intermediate training result of the local ML mode (for VFL).

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

### 11.2.5 Naf_VFLTraining_Request service operation

**Service operation name:** Naf_VFLTraining_Request

**Description:** In preparation of VFL training, requests AF VFL client to check if it can support requirements for VFL.

**Inputs, Required:**

\- Analytics ID.

\- VFL interoperability indicator.

**Inputs, Optional:**

\- Suggested VFL Interoperability Information.

\- Suggested list of sample IDs.

\- Suggested feature ID(s).

\- Time window of the data samples.

\- Required minimum sample size.

**Outputs Required:** When the request is accepted: indicate of accepting the ML Model training requirements.

When the request is not accepted, an error response with cause code (e.g. AF does not meet the VFL training requirements).

NOTE: The detail reasons in the cause code are up to Stage 3.

**Outputs, Optional:**

\- The list of supported Feature ID(s).

\- VFL Interoperability information that the VFL Client supports.

\- The list of sample IDs accepted within the sample IDs suggested by the VFL Server.

\- The list of supported sample IDs.

## 11.3 Naf_VFLInference Service


### 11.3.1 General

**Service Description:** This service is provided by AF acting as VFL client and enables an VFL server as consumer to request or subscribe/unsubscribe for a VFL inference.

When the subscription is accepted by the AF, the consumer receives from the NWDAF an identifier (Subscription Correlation ID) allowing to further manage (modify, delete) this subscription.

### 11.3.2 Naf_VFLInference_Subscribe service operation

**Service operation name:** Naf_VFLInference_Subscribe

**Description:** Subscribe to VFL inference.

**Inputs, Required:**

For new subscription:

\- Notification Target Address (+ Notification Correlation ID).

\- VFL Correlation ID.

\- Target of VFL inference.

\- Analytics ID.

When updating a subscription:

\- Subscription Correlation ID.

**Inputs, Optional:**

\- VFL inference filter.

\- Data time window.

\- Time when intermediate local result is needed.

\- Dataset Statistical Properties.

\- Analytics metadata request.

**Outputs Required:** When the subscription is accepted: Subscription Correlation ID (required for management of this subscription). When the subscription is not accepted, an error response.

**Outputs, Optional:** Client intermediate results.

### 11.3.3 Naf_VFLInference_Unsubscribe service operation

**Service operation name:** Naf_VFLInference_Unsubscribe

**Description:** Unsubscribe to VFL inference.

**Inputs, Required:** Subscription Correlation ID.

**Inputs, Optional:** None.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** None.

### 11.3.4 Naf_VFLInference_Notify service operation

**Service operation name:** Naf_VFLInference_Notify

**Description:** Notify VFL inference result.

**Inputs, Required:**

\- Notification Correlation Information.

**Inputs, Optional:**

\- Client intermediate results.

\- Analytics Metadata Information.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** None.

### 11.3.5 Naf_VFLInference_Request service operation

**Service operation name:** Naf_VFLInference_Request

**Description:** The consumer requests the AF to perform a one-time VFL inference.

**Inputs, Required:**

\- Target of VFL inference.

\- VFL Correlation ID.

\- Analytics ID.

**Inputs, Optional:**

\- VFL inference filter.

\- Data time window.

\- Time when intermediate local result is needed.

\- Dataset Statistical Properties.

\- Analytics metadata request.

**Outputs, Required:** If the request is accepted, then client intermediate results. When the request is not accepted, an error response.

**Outputs, Optional:**

\- Analytics Metadata Information.

## 11.4 Naf_Inference Service


### 11.4.1 General

**Service Description:** This service is provided by AF acting as VFL server and enables an NWDAF or an NEF acting on its behalf as consumer to request or subscribe/unsubscribe for a VFL inference.

### 11.4.2 Naf_Inference_Subscribe service operation

**Service operation name:** Naf_Inference_Subscribe

**Description:** Subscribe to VFL inference.

**Inputs, Required:**

For new subscription:

\- Notification Target Address (+ Notification Correlation ID).

\- Analytics ID.

\- Target of Analytics Reporting.

When updating a subscription:

\- Subscription Correlation ID.

**Inputs, Optional:**

\- Analytics Reporting Information (with parameters as defined in clause 6.1.3):

\- Event Reporting parameters defined in Table 4.15.1-1 of TS 23.502 \[3\].

\- Analytics Filter.

**Outputs Required:** When the subscription is accepted: Subscription Correlation ID (required for management of this subscription). When the subscription is not accepted, an error response.

**Outputs, Optional:** First corresponding inference report is included, if available and if analytics consumer requested immediate reporting (see clause 4.15.1 of TS 23.502 \[3\]).

### 11.4.3 Naf_Inference_Unsubscribe service operation

**Service operation name:** Naf_Inference_Unsubscribe

**Description:** Unsubscribe to VFL inference.

**Inputs, Required:** Subscription Correlation ID.

**Inputs, Optional:** None.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** None.

### 11.4.4 Naf_Inference_Notify service operation

**Service operation name:** Naf_Inference_Notify

**Description:** Notify VFL inference result.

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

### 11.4.5 Naf_Inference_Request service operation

**Service operation name:** Naf_Inference_Request

**Description:** The consumer requests the AF to perform a one-time VFL inference.

**Inputs, Required:**

\- Analytics ID.

\- Target of Analytics Reporting.

**Inputs, Optional:**

\- Analytics Reporting Information (with parameters as defined in clause 6.1.3).

\- Analytics Filter.

\- VFL correlation ID.

**Outputs, Required:** If the request is accepted, then VFL inference results, i.e. set of the tuple (Analytics ID, Analytics specific parameters). When the request is not accepted, an error response.

**Outputs, Optional:**

\- Validity period.

\- Confidence.

\- Analytics Metadata Information.

\- Analytics Accuracy Information.

## 11.5 Naf_Training Service


### 11.5.1 General

**Service Description:** This service is provided by an AF acting as VFL server and enables an NWDAF or an NEF acting on its behalf as consumer to request the AF to perform model training as defined in clause 6.2H.2.3 under the supervision of the consumer.

### 11.5.2 Naf_Training_Subscribe service operation

**Service operation name:** Naf_Training_Subscribe

**Description:** Subscribes to ML Model training with AF as VFL server.

**Inputs, Required:**

For new subscription:

\- Analytics ID as defined in Table 7.1-2.

\- Notification Target Address (+ Notification Correlation ID).

When updating a subscription:

\- Subscription Correlation ID.

**Inputs, Optional:** See clause 6.2H.3 for parameters.

**Outputs Required:** When the request is accepted: Subscription Correlation ID (required for management of this subscription). When the request is not accepted, an error response with cause code.

NOTE: The detail reasons in the cause code are up to Stage 3.

**Outputs, Optional:** None.

### 11.5.3 Naf_Training_Unsubscribe service operation

**Service operation name:** Naf_Training_Unsubscribe

**Description:** Terminate AF ML Model training.

**Inputs, Required:** Subscription Correlation ID.

**Inputs, Optional:** None.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** Cause code.

### 11.5.4 Naf_Training_Notify service operation

**Service operation name:** Naf_Training_Notify

**Description:** AF notifies the consumer of training progress

**Inputs, Required:**

\- Notification Correlation Information.

**Inputs, Optional:**

\- Indication whether Training is complete.

\- Estimated VFL training completion time.

\- VFL correlation ID.

\- VFL status report.

**Outputs, Required:** Operation execution result indication.

**Outputs, Optional:** None.
