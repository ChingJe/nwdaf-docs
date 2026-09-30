---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
clause: '8'
title: 8 SCCP subsystem numbers
content_origin: 3gpp-source
---

# 8 SCCP subsystem numbers

Subsystem numbers are used to identify applications within network entities which use SCCP signalling. In GSM and UMTS, subsystem numbers may be used between PLMNs, in which case they are taken from the globally standardized range (1 - 31) or the part of the national network range (129 - 150) reserved for GSM/UMTS use between PLMNs. For use within a PLMN, they are taken from the part of the national network range (32 - 128 & 151 - 254) not reserved for GSM/UMTS use between PLMNs.

## 8.1 Globally standardized subsystem numbers used for GSM/UMTS

The following globally standardised subsystem numbers have been allocated for use by GSM/UMTS:

0000 0110 HLR (MAP);

0000 0111 VLR (MAP);

0000 1000 MSC (MAP);

0000 1001 EIR (MAP);

0000 1010 is allocated for evolution (possible Authentication Centre).

## 8.2 National network subsystem numbers used for GSM/UMTS

The following national network subsystem numbers have been allocated for use within GSM/UMTS networks:

1111 1000 CSS (MAP);

1111 1001 PCAP;

1111 1010 BSC (BSSAP-LE);

1111 1011 MSC (BSSAP-LE);

1111 1100 SMLC (BSSAP-LE);

1111 1101 BSS O&M (A interface);

1111 1110 BSSAP (A interface).

The following national network subsystem numbers have been allocated for use within and between GSM/UMTS networks:

1000 1110 RANAP;

1000 1111 RNSAP;

1001 0001 GMLC (MAP);

1001 0010 CAP;

1001 0011 gsmSCF (MAP) or IM-SSF (MAP) or Presence Network Agent;

1001 0100 SIWF (MAP);

1001 0101 SGSN (MAP);

1001 0110 GGSN (MAP).
