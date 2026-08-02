---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: '22'
title: 22 Addressing and identification for ANDSF
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 22 Addressing and identification for ANDSF

## 22.1 Introduction

This clause describes the format of the parameters needed by the UE to use Access Network Discovery and Selection Function (ANDSF) as specified in TS 23.402 \[68\].

## 22.2 ANDSF Server Name (ANDSF-SN)

### 22.2.1 General

ANDSF Server Name (ANDSF-SN) is used by UE to discover ANDSF Server in the network.

### 22.2.2 Format of ANDSF-SN

The ANDSF-SN is composed of six labels. The last three labels shall be "pub.3gppnetwork.org". The second and third labels together shall uniquely identify the PLMN. The first label shall be "andsf".

The ANDSF-SN is derived from the IMSI or Visited PLMN Identity as follows:

"andsf.mnc\<MNC\>.mcc\<MCC\>.pub.3gppnetwork.org"

where:

"mnc" and "mcc" serve as invariable identifiers for the following digits.

\- When contacting Visited ANDSF (V-ANDSF), the \<MNC\> and \<MCC\> shall be derived from the Visited PLMN Identity as defined in clause 12.1.

\- When contacting Home ANDSF (H-ANDSF), the \<MNC\> and \<MCC\> shall be derived from the components of the IMSI defined in clause 2.2.

In order to guarantee inter-PLMN DNS translation, the \<MNC\> and \<MCC\> coding used in the "andsf.mnc\<MNC\>.mcc\<MCC\>.pub.3gppnetwork.org" format of the ANDSF-SN shall be:

\- \<MNC\> = 3 digits

\- \<MCC\> = 3 digits

If there are only 2 significant digits in the MNC, one "0" digit shall be inserted at the left side to fill the 3 digits coding of MNC in the ANDSF-SN.

As an example, the ANDSF-SN OI for MCC 345 and MNC 12 is coded in the DNS as:

"andsf.mnc012.mcc345.pub.3gppnetwork.org".
