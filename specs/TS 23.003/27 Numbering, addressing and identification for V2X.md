---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: '27'
title: 27 Numbering, addressing and identification for V2X
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 27 Numbering, addressing and identification for V2X

## 27.1 Introduction

This clause describes the format of the parameters used for V2X. For further information on the use of the parameters see TS 23.285 \[117\].

## 27.2 V2X Control Function FQDN

### 27.2.1 General

In order to retrieve V2X communication parameters, the UE needs to connect to the V2X Control Function. The address of the V2X control Function can be provisioned to the UE, or the UE can be pre-configured with the FQDN of the V2X Control Function. If the address of the V2X Control Function is not provisioned, and the UE is not pre-configured with the FQDN of the V2X Control Function FQDN, the UE self-constructs the V2X Control Function FQDN as per the format specified in clause 27.2.2.

### 27.2.2 Format of V2X Control Function FQDN

The V2X Control Function Fully Qualified Domain Name (V2X Control Function FQDN) contains an Operator Identifier that shall uniquely identify the PLMN where the V2X Control Function is located. The V2X Control Function FQDN is composed of six labels. The last two labels shall be "3gppnetwork.org". The third and fourth labels together shall uniquely identify the PLMN. The first two labels shall be "v2xcontrolfunction.epc". The V2X Control Function FQDN shall be constructed as follows:

"v2xcontrolfunction.epc.mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org"

In order to guarantee inter-PLMN DNS translation, the \<MNC\> and \<MCC\> coding used in the "v2xcontrolfunction.epc.mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org" format of the V2X Control Function FQDN shall be:

\- \<MNC\> = 3 digits

\- \<MCC\> = 3 digits

If there are only 2 significant digits in the MNC, one "0" digit shall be inserted at the left side to fill the 3 digits coding of MNC in the V2X Control Function FQDN.

As an example, the V2X Control Function FQDN for MCC 345 and MNC 12 is coded in the DNS as:

"v2xcontrolfunction.epc.mnc012.mcc345.3gppnetwork.org".
