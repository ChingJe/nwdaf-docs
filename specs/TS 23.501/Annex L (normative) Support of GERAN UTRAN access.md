---
spec: TS 23.501
version: 18.12.0
release: '18'
clause: Annex L
title: 'Annex L (normative): Support of GERAN/UTRAN access'
source_archive: 23501-ic0.zip
source_document: 23501-ic0.docx
source_archive_sha256: 800fb58a2a3a8c3eeac7084497de566fc730808923d8eea61b84d2555360debe
source_document_sha256: cdab3840e4d278d381f70806e6458f87dff8e5fd238f370e4f65872bbd5088a6
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex L (normative): Support of GERAN/UTRAN access

This annex applies when the SMF+PGW-C is enhanced to support UE accessing the network via GERAN/UTRAN over Gn/Gp interface. For this scenario, the SMF+PGW-C uses N7 interface to interact with PCF and the N40 interface to interact with CHF.

NOTE 1: For the interface with the serving node of the UE, the SMF+PGW-C is assumed to behave as the Control Plane of the PGW described in Annex D of TS 23.401 \[26\].

SMF+PGW-C selection by SGSN is specified in Annex G of TS 23.502 \[3\].

The SMF+PGW-C interacting with PCF for GERAN/UTRAN access is specified in Annex G of TS 23.502 \[3\].

The functional description for SMF+PGW-C interacting with PCF to support GERAN/UTRAN access is specified in TS 23.503 \[45\].

NOTE 2: Support for IP address preservation upon mobility between 5GS and GERAN/UTRAN for PDN sessions established in EPC is described in clause 5.17.2.4. IP address preservation is not supported for direct mobility between 5GS and GERAN/UTRAN, nor for indirect mobility cases when the PDN session is established in 5GS or in GERAN/UTRAN.

The charging services on SMF+PGW-C interactions with CHF for GERAN/UTRAN access are specified in TS 32.255 \[68\].
