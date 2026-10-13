---
spec: TR 23.700-80
version: 18.0.0
release: '18'
clause: 4
title: 4 Architectural Requirements and Assumptions
source_archive: 23700-80-i00.zip
source_document: 23700-80-i00.docx
content_origin: 3gpp-source
---

# 4 Architectural Requirements and Assumptions


## 4.1 Architectural Requirements

The Application AI/ML operation logic is controlled by an Application Function (AF).

Any AF request to the 5G System in the context of 5GS assistance to Application AI/ML operation should be authorized by the 5GC.

The 5G System should ensure the performance KPIs specified in TS 22.261 \[2\] clause 7.10, to support the Application AI/ML traffic.

## 4.2 Architectural Assumptions

To support AI/ML based services/applications via 5GS, the following architectural assumptions are made in the present study:

\- In Rel-18, an Application AI/ML operation is conducted within a single slice, i.e. all UEs which are involved in a given Application AI/ML operation are served by the same S-NSSAI and the Application Function (AF) belongs to this S-NSSAI.

\- In Rel-18, roaming is not supported, i.e. inter PLMN coordination aspects will not be studied.

\- 5GC can differentiate Application Layer AI/ML traffic by using existing mechanisms defined in TS 23.501 \[3\].

NOTE 1: The 5G System architecture to support the Application AI/ML operation in the current release should not preclude the support of roaming in a future release.

NOTE 2: Whether 5GC needs to be aware of the Application Layer AI/ML operation type is documented in clause 8.

NOTE 3: Different KPI requirements, as described in the subclause 7.10 of the TS 22.261 \[2\], need to be taken into account for AI/ML model transfer in 5GS. How to map these KPI requirements into 5GS QoS parameters is discussed under conclusions for KI#6.

## 4.3 Reference Architecture

The architecture, framework, definitions and terminology specified in TS 23.501 \[3\], TS 23.502 \[4\], TS 23.503 \[5\] and TS 23.288 \[6\] are the baselines for the present study.
