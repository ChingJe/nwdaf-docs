---
spec: TS 29.549
version: 20.1.0
release: '20'
clause: 9
title: 9 Security
source_archive: 29549-k10.zip
source_document: '29549-k10_0_cover.docx, 29549-k10_1_Main-Body_s00_s06.docx, 29549-k10_2_Main-Body_s07_s09.docx, 29549-k10_3_Annexes_sA_sHistory.docx'
content_origin: 3gpp-source
---

# 9 Security


## 9.1 General

The security aspects of SEAL reference points are specified in 3GPP TS 33.434 \[26\].

## 9.2 SEAL-S security

As specified in clause 5.1.1.8 of 3GPP TS 33.434 \[26\], the protection of SEAL-S reference point shall be supported according to NDS/IP as specified in 3GPP TS 33.210 \[25\].

When CAPIF is not used, then TLS and OAuth 2.0 shall be supported as described in clause 5.1.1.8 of 3GPP TS 33.434 \[26\]. When TLS is used, mutual authentication based on client and server certificates shall be performed between the SEAL server and the service consumer using TLS. After the authentication, the SEAL server determines whether the service consumer is authorized to send requests to the SEAL server. The SEAL server shall authorize the requests from the service consumer using OAuth-based authorization mechanism.

When CAPIF is used, the security mechanisms described in clause 8.2 shall be applied.
