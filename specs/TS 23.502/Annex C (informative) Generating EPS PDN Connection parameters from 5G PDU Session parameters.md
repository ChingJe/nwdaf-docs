---
spec: TS 23.502
version: 18.14.0
release: '18'
clause: Annex C
title: 'Annex C (informative): Generating EPS PDN Connection parameters from 5G PDU
  Session parameters'
source_archive: 23502-ie0.zip
source_document: 23502-ie0.docx
source_archive_sha256: 0f24045c49ddf710fb91e51ec74d070cfd13d1e9bafc07809d7e87eae6a61ce8
source_document_sha256: 89536df6095fba3b9c90f1fa06ceaec918dfc6b0fd26a769a06eae7db66bb603
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex C (informative): Generating EPS PDN Connection parameters from 5G PDU Session parameters

This annex specifies how to generate the EPS PDN connection parameters from the 5G PDU Session parameters in SMF+PGW-C.

When the SMF+PGW-C is requested to set up/modify either a PDN connection or a PDU session supporting interworking between EPS and 5GS, the SMF+PGW-C generates the PDN Connection parameters from the PDU session parameters.

When the SMF+PGW-C generates the PDN Connection parameters based on the PDU Session parameters, the following rules hold:

\- PDN type: the PDN type is set to IPv4, IPv6 or IPv4v6 if the PDU Session Type is IPv4, IPv6 or IPv4v6, respectively. The PDN Type is set to Ethernet if the MME, SGW and UE support Ethernet PDN Type, otherwise the PDN type is set to Non-IP for Ethernet and Unstructured PDU Session Types

\- EPS bearer ID: the EBI is requested from the AMF during the establishment of a QoS Flow as described in clause 4.11.1.4.1 for PDU Sessions supporting interworking between EPS and 5GS. The EBI is obtained from MME during the establishment of an EPS Bearer (that is triggered by an establishment of a QoS Flow) as defined in TS 23.401 \[13\] for PDN Connections hosted by SMF+PGW-C. The association between EBI and QoS Flow is stored by the SMF.

\- APN-AMBR: APN-AMBR is set according to operator policy (e.g. taking the Session AMBR into account).

\- EPS QoS parameters (including ARP, QCI, GBR and MBR):

If QoS Flow is mapped to one EPS bearer, ARP, GBR and MBR of the EPS Bearer is set to the ARP, GFBR and MFBR of the corresponding QoS Flow, respectively. For standardized 5QIs, the QCI is one to one mapped to the 5QI. For non-standardized 5QIs, the SMF+PGW-C derives the QCI based on the 5QI and operator policy.

A GBR QoS Flow is mapped 1 to 1 to a GBR dedicated EPS Bearer if an EBI has been assigned. After mobility to EPS traffic flows corresponding to GBR QoS Flow for which no EBI has been assigned will continue flowing on the default EPS bearer if it does not have assigned TFT.

If multiple QoS Flows are mapped to one EPS bearer, the EPS bearer parameters are set based on operator policy, e.g. EPS bearer QoS parameters are set according to the highest QoS of all mapped QoS Flows.

After mobility to EPS traffic flows corresponding to Non-GBR QoS Flows for which no EBI has been assigned will continue flowing on the default EPS Bearer if it does not have assigned TFT.
