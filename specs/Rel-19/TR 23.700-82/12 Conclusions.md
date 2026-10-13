---
spec: TR 23.700-82
version: 19.1.0
release: '19'
clause: 12
title: 12 Conclusions
source_archive: 23700-82-j10.zip
source_document: 23700-82-j10.docx
content_origin: 3gpp-source
---

# 12 Conclusions


## 12.0 General conclusions and recommendations

The present technical report described the AI/ML enablement capabilities for supporting vertical use cases. This technical report fulfills the objectives of the study on application architecture for enabling AIML services. The report includes the following:

1\. Definition of terms and abbreviations used in the study (clause 3);

2\. Analysis on AI/ML support in 3GPP (clause 4);

2\. Key issues identified by the study (clause 5);

3\. Architectural requirements and detailed application architecture for enabling AI/ML services (clause 6 and 7);

4\. Individual solutions addressing the key issues (clause 8);

5\. Deployment scenarios (clause 9), Business relationships (clause 10); and

6\. Overall evaluations of all the solutions (clause 11); For normative work in 3GPP Rel-19, it is recommended to define the following architecture enhancements as also described in clause 7:

\- AIMLE to be defined as a new SEAL functionality, for supporting the AI/ML enablement for both VAL applications and ADAE-defined analytics services.

\- ML Repository as new registry/repository which can be deployed internally to AIMLE or as new SEAL service and will support storage and registration of ML model information and candidate AIML/FL member information as well as information related to ML/FL members for a given ML task (e.g., analytics ID or model ID/profile).

\- ADAES to be enhanced to support additional/enhanced analytics based on the concluded solutions.

In addition, it is recommended to define:

1\. Terms and abbreviations, the definition of terms and abbreviations captured in clause 3 will be reused.

2\. Architectural requirements identified in clause 6 will be used as baseline architectural requirements; such requirements include also per functionality-imposed requirements for the enhanced architecture for AIML Enablement.

3\. Application architecture for enabling AI/ML services, the architectures as specified in clause 7 will be used as baseline architecture.

4\. Deployment scenarios will be considered as captured in clause 10. Additional deployment models and their implications on the solutions will be considered.

5\. The definition of the corresponding enablement APIs based on the concluded solutions as summarized in clause 11.1.

## 12.1 Conclusions of key issue #1

Following solution considerations will be considered for the normative work for KI #1:

\- Solution #11 can be considered for normative work for AIML service lifecycle management, and \#12 can be considered as candidate for normative work for AI/ML model lifecycle management.

\- Solution #13 can be considered for normative work as a generic architecture for leveraging the existing ADAE services and 5GC capabilities for the application layer AI/ML services.

\- Solution #17 can be considered for normative work with necessary update for detail AIML task.

\- Solutions #23 and #29 can be considered for normative work for support AIML services in edge computing scenarios.

\- Solution \#31, with the potential modification that allows the Edge AIML enablement server to directly interact with the ML repository in order to obtain AI/ML models, can be considered for normative work for support AIML services in edge computing scenarios.

NOTE 1: The possible impact of solutions related to edge on the AIMLAPP architecture will be further discussed.

> NOTE 2: The rationality and benefits of Solution \#31 will be further elaborated in the normative phase.

## 12.2 Conclusions of key issue #2

Following solution considerations will be considered for the normative work for KI #2:

\- Solution #4 can be considered for normative work to support ML-enabled ADAE analytics.

\- Solution #5 can be considered can be considered for normative work for ML model storage and discovery.

\- The common parts of Solution #4 and Solution #5 for ML model discovery can be merged in the normative phase.

\- Solution #15 can be considered for normative work for DN Energy Analytics.

\- Solution #13 can be considered for normative work for analytics collection from ADAE to support application layer AI/ML services.

\- Solution #27 can be considered for normative work for application layer AI/ML Member capability analytics for to support application layer AI/ML services.

## 12.3 Conclusions of key issue #3

Following solution considerations will be considered for the normative work for KI #3:

\- Solutions #13 and #22 can be considered for normative work.

\- Solutions #2, #6, and #7 can be merged for FL member discovery and selection, the solution after merging can be considered for normative work.

\- Solution #8 can be considered for normative work for generic AIMLE client registration.

\- Solution #9 can be considered as candidate for normative work as extension of the generic procedure in Solution #8 for FL member registration and update to ML Repository via AIMLE,

\- Solution #16 can be considered for normative work for supporting management of FL groups.

\- Solution #17 can be considered for normative work with necessary update for detail AIML task (e.g. HFL training in Solution #22).

\- Solution #21 can be considered for normative work with modification as needed.

\- Conclusions for Solution #24 and #25 are given in clause 12.5.

\- Solution #26 can be considered for normative work for support FL member grouping.

\- Conclusions for Solutions #3, #10 and #14, and additional conclusion for Solution #6 are given in clause 12.7.

## 12.4 Conclusions of key issue #4

Following solution considerations will be considered for the normative work for KI #4:

\- Solution #18 can be considered for normative phase.

## 12.5 Conclusions of Key Issue #5

The following principles will be considered in the normative phase for KI #5.

Split AI/ML operation:

\- Solution #19 will be considered for AIML split operation pipeline discovery (request/response), events subscription/notification and split an operation pipeline creation/registration (request/response). AIML split operation profile provides the information for exposing and consuming split AI/ML analytics.

\- Solution #28 will be considered for AIML split operation pipeline assistance information subscription/notification. AIML split operation assistance information provides the information for exposing and consuming split AI/ML analytics.

NOTE 1: Possible merge or alignment of the subscribe/notify model and AIML split operation information in Solution #19 and Solution #28 will be considered in the normative phase.

\- Solution #23 can rely on solution #19 and #28 to manage and expose created AIML split operation pipeline information in edge scenarios. Further discussions will happen in the normative phase to understand how the capabilities of the CES and EES can be enhanced for creating AIML split operation pipelines in edge areas.

Solution #19 and Solution #28 have a complementary relationship; Solution #23 addresses different aspects of split AIML operation related to edge and cloud.

Transfer of AI/ML models:

\- Solution #24 will be considered for distribution of AIML models (request/response).

\- Solution #25 will be considered for distribution of AIML models (subscribe/notify).

NOTE 2: Possible alignment of the request/response and subscribe/notify models in Solution #24 and Solution #25 will be considered in normative phase.

Solution #24 and Solution #25 have a complementary relationship.

## 12.6 Conclusions of key issue #6

Following solution considerations will be considered for the normative work for KI #6:

\- Solution #12 can be considered for normative work for ML model lifecycle management,

\- Solution #20 can be considered for normative work for supporting the discovery and selection of models.

\- The common parts of Solution #12 and Solution #20 for ML model re-training can be merged in the normative phase.

\- Solution #30 can be considered for normative work for assistance on the transfer of intermediate AI/ML operaton status and results.

## 12.7 Conclusions of key issue #7

Following solution considerations will be considered for the normative work for KI #7:

\- Solution #6 and Solution #14 are complementing each other and can be considered for the normative work. It is recommended to specify Solution #6 and Solution #14 as a part the AIMLE Client Discovery and Selection API.

\- Solution #2 can be merged into Solution #7. Solution #7 can be considered for the normative work. It is recommended to specify Solution #7 as a part of AIMLE Client Discovery and Selection API.

\- Solution #3 covered by Solution #10, and Solution #10 can be merged with Solution #8. Solution #8 merged with Solution #10 can be considered for the normative work. It is recommended to specify Solution #8 merged with Solution #10 in the AIMLE Client Registration API.

\- Solution #30 can be considered for the normative work.
