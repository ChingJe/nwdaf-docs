---
spec: TS 23.436
version: 20.2.0
release: '20'
clause: 4
title: 4 Architectural requirements
source_archive: 23436-k20.zip
source_document: 23436-k20.docx
content_origin: 3gpp-source
---

# 4 Architectural requirements


## 4.1 General Description

The following clauses specify the requirements for application data analytics enablement service.

## 4.2 General Requirements

\[AR-4.2-a\] The ADAE client and the ADAE server shall support one or more VAL applications.

\[AR-4.2-b\] Supported ADAE capabilities shall be offered as APIs to the VAL applications.

\[AR-4.2-c\] The ADAE shall support interaction with 3GPP network system to consume network and management data analytics services.

\[AR-4.2-d\] The ADAE client shall be capable to communicate with one or more ADAE servers of the same ADAE service provider.

## 4.3 ADAE internal architecture requirements

\[AR-4.3-a\] The ADAE layer shall be able to provide a data collection coordination functionality to enable the collection from diverse data sources (OAM, 5GC, UE) per application data analytics event type.

\[AR-4.3-b\] The ADAE layer shall include a data analytics repository function to store application data analytics.

\[AR-4.3-c\] The data collection coordination and repository capabilities may be offered as APIs to ADAE server.

## 4.4 ADAE capability related requirements

\[AR-4.4-a\] The ADAE server shall be capable of providing data analytics for the VAL server performance.

\[AR-4.4-b\] The ADAE server shall be capable of providing data analytics for the VAL application sessions (for both Uu-based and PC5-based sessions).

\[AR-4.4-c\] The ADAE server shall be able to collect application performance measurements and analytics from one or more ADAE clients.

\[AR-4.4-d\] The ADAE server shall be capable of collecting edge data from one or more edge platforms

\[AR-4.4-e\] The ADAE server shall enable the exposure of edge data analytics to the VAL applications

\[AR-4.4-f\] The ADAE server shall be capable of providing data analytics for the VAL server or VAL session performance for a requested slice or slice instance.

\[AR-4.4-g\] The ADAE server shall be capable of providing data analytics for the location accuracy of one or more VAL UEs.

\[AR-4.4-h\] The ADAE server shall be capable of providing data analytics related to the availability and status of one or more service APIs.
