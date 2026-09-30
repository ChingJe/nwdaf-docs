---
spec: TR 23.700-04
version: 20.0.0
release: '20'
clause: 4
title: 4 Architectural Assumptions and Requirements
source_archive: 23700-04-k00.zip
source_document: 23700-04-k00.docx
content_origin: 3gpp-source
---

# 4 Architectural Assumptions and Requirements


## 4.1 Architectural Assumptions

The present study will not consider service-based interfaces with RAN nor with the UE (e.g. neither registration to NRF nor subscription to NRF).

The architecture for the present study shall comply with 5GS framework as specified in TS 23.501 \[2\], TS 23.502 \[3\] and TS 23.503 \[4\] and TS 23.288 \[5\].

NOTE 1: This study considers user consent and privacy as currently defined in SA WG2 specifications.

NOTE 2: Potential user consent and privacy enhancements are under SA WG3 scope, then SA WG2 may need to align with SA WG3.

## 4.2 Architectural Requirements

For the standardized transfer of standardized data over UP for UE data collection to meet requirements for AI/ML for NR air interface operation with UE-side model training, the following architectural requirements shall be considered:

\- The MNO has full control of the standardized data collection transfer process and can manage data transfer to the server for UE-side data collection, without the need of SLA for this purpose. This includes initiating, terminating and fully managing data transfer.

\- The MNO has full visibility for standardized data.

\- Roaming case is not supported in this Release.

NOTE 1: The requirements to study UP Data Collection (for Option 2) are documented in RAN LSs RP-243316 and RP-242389.

NOTE 2: The requirements elaborated in LSs RP-242389 and R2-2411152 will be considered for investigating and evaluating potential solutions documented for this key issue.

NOTE 3: Full visibility allows the MNO to verify/match the data specified/configured to be collected and the data that is being reported.

For the enhancements of 5GC analytics to support efficient performance of user plane, the following architectural requirements shall be considered:

\- The study shall not address how UPF detects wrong or malformed or duplicated packets.

\- This study shall have neither RAN nor UE impacts.

\- Solutions shall minimize the load on Control Plane NFs due to too much reporting from UPF.

\- The existing UPF event exposure should be reused as much as possible.
