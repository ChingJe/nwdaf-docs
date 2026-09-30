---
spec: TS 23.482
version: 20.3.0
release: '20'
clause: 4
title: 4 Architectural requirements
source_archive: 23482-k30.zip
source_document: 23482-k30.docx
content_origin: 3gpp-source
---

# 4 Architectural requirements


## 4.1 General requirements

\[AR-4.1-a\] The AIML enablement layer shall be able to support one or more VAL applications.

\[AR-4.1-b\] Supported AIML enablement capabilities shall be offered as APIs to the VAL applications.

\[AR-4.1-c\] The AIML enablement layer shall support interaction with 3GPP network system to consume network and AI/ML support services.

\[AR-4.1-d\] The AIMLE client shall be capable to communicate with one or more AIMLE servers of the same AIMLE service provider.

\[AR-4.1-e\] The AIML enablement layer shall support multi-operator service scenarios, where an AIMLE server may serve AIMLE clients associated with different PLMNs, subject to service agreement/policy.

\[AR-4.1-f\] The AIML enablement layer shall support interworking between AIMLE servers (e.g., central and edge AIMLE servers, and/or AIMLE servers of different PLMNs) for providing AIMLE services in distributed deployments.

\[AR-4.1-g\] The AIMLE enablement layer shall be capable to support AIMLE service continuity in roaming scenarios, including interaction with an AIMLE server over HPLMN or VPLMN according to roaming principles and policies.

\[AR-4.1-h\] The AIML enablement layer shall support discovery of AIMLE servers in hierarchical, distributed, and multi-operator deployments.

\[AR-4.1-i\] The AIML enablement layer shall support federated AI service, subject to service agreement/policy.

## 4.2 AIML capability related requirements

\[AR-4.2-a\] The AIMLE server shall be capable of provisioning and exposing ML client information.

\[AR-4.2-b\] The AIMLE server shall be capable of supporting the registration, discovery, and selection of AIMLE clients which participate as ML members in AIML service lifecycle.

\[AR-4.2-c\] The AIMLE layer shall be capable of supporting ML service lifecycle operations (e.g., ML model training).

\[AR-4.2-d\] The AIMLE server shall be capable of supporting discovery and provisioning of AIML models.

\[AR-4.2-e\] The AIMLE server shall be capable of supporting ML model inference for authorized AIMLE service consumers.

\[AR-4.2-f\] The AIMLE server shall be capable of supporting ML model maintenance.

\[AR-4.2-g\] The ML repository shall be capable of storing, retrieving, and removing ML model evaluation information.

\[AR-4.2-h\] The AIMLE server shall be capable of supporting ML model inference service performance management and feedback.

\[AR-4.2-i\] The AIMLE data management assistance capability shall be capable of supporting data drift detection.
