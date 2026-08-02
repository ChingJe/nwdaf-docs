---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: '17'
title: 17 Numbering, addressing and identification within the Generic Access Network
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 17 Numbering, addressing and identification within the Generic Access Network

## 17.1 Introduction

This clause describes the format of the parameters needed to access the Generic Access Network (GAN). For further information on the use of the parameters and GAN in general, see TS 43.318 \[61\] and TS 44.318 \[62\]. For more information on the ".3gppnetwork.org" domain name and its applicability, see Annex D of the present document.

## 17.2 Network Access Identifiers

### 17.2.1 Home network realm

The home network realm shall be in the form of an Internet domain name, e.g. operator.com, as specified in IETF RFC 1035 \[19\] and IETF RFC 1123 \[20\]. The home network realm consists of several labels. Each label shall consist of the alphabetic characters (A-Z and a-z), digits (0-9) and the hyphen (-) in accordance with IETF RFC 1035 \[19\]. Each label shall begin and end with either an alphabetic character or a digit in accordance with IETF RFC 1123 \[20\]. The case of alphabetic characters is not significant.

The UE shall derive the home network realm from the IMSI as described in the following steps:

1\. take the first 5 or 6 digits, depending on whether a 2 or 3 digit MNC is used (see TS 31.102 \[27\], TS 51.011 \[66\]) and separate them into MCC and MNC; if the MNC is 2 digits then a zero shall be added at the beginning;

2\. use the MCC and MNC derived in step 1 to create the "mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org" network realm;

3\. add the label "gan." to the beginning of the network realm.

An example of a home network realm is:

IMSI in use: 234150999999999;

Where:

MCC = 234;

MNC = 15;

MSIN = 0999999999,

Which gives the home network realm: gan.mnc015.mcc234.3gppnetwork.org.

NOTE: If it is not possible for the UE to identify whether a 2 or 3 digit MNC is used (e.g. SIM is inserted and the length of MNC in the IMSI is not available in the "Administrative data" data file), it is implementation dependent how the UE determines the length of the MNC (2 or 3 digits).

### 17.2.2 Full Authentication NAI

The Full Authentication NAI in both EAP‑SIM and EAP‑AKA shall take the form of an NAI as specified in clause 2.1 of IETF RFC 4282 \[53\]. The format of the Full Authentication NAI shall comply with IETF RFC 4187 \[50\] when EAP‑AKA authentication is used and with IETF RFC 4186 \[51\], when EAP‑SIM authentication is used. The realm used shall be a home network realm as defined in clause 17.2.1.

The result will therefore be an identity of the form:

"0\<IMSI\>@gan.mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org", for EAP‑AKA authentication and "1\<IMSI\>@gan.mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org", for EAP‑SIM authentication

EXAMPLE 1: For EAP AKA authentication: If the IMSI is 234150999999999 (MCC = 234, MNC = 15), the Full Authentication NAI takes the form 0234150999999999@gan.mnc015.mcc234.3gppnetwork.org.

EXAMPLE 2: For EAP SIM authentication: If the IMSI is 234150999999999 (MCC = 234, MNC = 15), the Full Authentication NAI takes the form 1234150999999999@gan.mnc015.mcc234.3gppnetwork.org.

### 17.2.3 Fast Re‑authentication NAI

The Fast Re-authentication NAI in both EAP-SIM and EAP-AKA shall take the form of an NAI as specified in clause 2.1 of IETF RFC 4282 \[53\]. The UE shall use the re-authentication identity received during the previous EAP‑SIM or EAP‑AKA authentication procedure. If such an NAI contains a realm part then the UE should not modify it, otherwise it shall use a home network realm as defined in clause 17.2.1.

The result will therefore be an identity of the form:

"\<re‑authentication_ID_username\>@\<re‑authentication_ID_realm\> for both EAP‑SIM and EAP‑AKA authentication when a realm is present in the re‑authentication identity received during the previous EAP‑SIM or EAP‑AKA authentication procedure and

"\<re‑authentication_ID_username\>@gan.mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org", for both EAP‑SIM and EAP‑AKA authentication when a realm is *not* present in the re‑authentication identity received during the previous EAP‑SIM or EAP‑AKA authentication procedure.

EXAMPLE 1: If the re‑authentication identity is "12345" and the IMSI is 234150999999999 (MCC = 234, MNC = 15), the Fast Re‑authentication NAI takes the form 12345@gan.mnc015.mcc234.3gppnetwork.org

EXAMPLE 2: If the re‑authentication identity is "12345@aaa1.gan.mnc015.mcc234.3gppnetwork.org", the Fast Re‑authentication NAI takes the form 12345@aaa1.gan.mnc015.mcc234.3gppnetwork.org

## 17.3 Node Identifiers

### 17.3.1 Home network domain name

The home network domain name shall be in the form of an Internet domain name, e.g. operator.com, as specified in IETF RFC 1035 \[19\] and IETF RFC 1123 \[20\]. The home network domain name consists of several labels. Each label shall consist of the alphabetic characters (A-Z and a-z), digits (0-9) and the hyphen (-) in accordance with IETF RFC 1035 \[19\]. Each label shall begin and end with either an alphabetic character or a digit in accordance with IETF RFC 1123 \[20\]. The case of alphabetic characters is not significant.

The UE shall derive the home network domain name from the IMSI as described in the following steps:

1\. take the first 5 or 6 digits, depending on whether a 2 or 3 digit MNC is used (see TS 31.102 \[27\], TS 51.011 \[66\]) and separate them into MCC and MNC; if the MNC is 2 digits then a zero shall be added at the beginning;

2\. use the MCC and MNC derived in step 1 to create the "mnc\<MNC\>.mcc\<MCC\>.pub.3gppnetwork.org" domain name;

3\. add the label "gan." to the beginning of the domain name.

An example of a home network domain name is:

IMSI in use: 234150999999999;

Where:

MCC = 234;

MNC = 15;

MSIN = 0999999999,

Which gives the home network domain name: gan.mnc015.mcc234.pub.3gppnetwork.org.

NOTE: If it is not possible for the UE to identify whether a 2 or 3 digit MNC is used (e.g. SIM is inserted and the length of MNC in the IMSI is not available in the "Administrative data" data file), it is implementation dependent how the UE determines the length of the MNC (2 or 3 digits).

### 17.3.2 Provisioning GANC-SEGW identifier

The Provisioning GANC‑SEGW identifier shall take the form of a fully qualified domain name (FQDN) as specified in IETF RFC 1035 \[19\] and IETF RFC 1123 \[20\]. The Provisioning GANC‑SEGW identifier consists of several labels. Each label shall consist of the alphabetic characters (A-Z and a-z), digits (0-9) and the hyphen (-) in accordance with IETF RFC 1035 \[19\]. Each label shall begin and end with either an alphabetic character or a digit in accordance with IETF RFC 1123 \[20\]. The case of alphabetic characters is not significant.

If the (U)SIM is not provisioned with the FQDN or IP address of the Provisioning GANC‑SEGW, the UE derives an FQDN from the IMSI to identify the Provisioning GANC‑SEGW. The UE shall derive such an FQDN as follows:

1\. create a domain name as specified in 17.3.1;

2\. add the label "psegw." to the beginning of the domain name.

An example of an FQDN for a Provisioning GANC-SEGW is:

IMSI in use: 234150999999999;

Where:

MCC = 234;

MNC = 15;

MSIN = 0999999999,

Which gives the FQDN: psegw.gan.mnc015.mcc234.pub.3gppnetwork.org.

NOTE: If it is not possible for the UE to identify whether a 2 or 3 digit MNC is used (e.g. SIM is inserted and the length of MNC in the IMSI is not available in the "Administrative data" data file), it is implementation dependent how the UE determines the length of the MNC (2 or 3 digits).

### 17.3.3 Provisioning GANC identifier

The Provisioning GANC identifier shall take the form of a fully qualified domain name (FQDN) as specified in IETF RFC 1035 \[19\] and IETF RFC 1123 \[20\]. The Provisioning GANC identifier consists of several labels. Each label shall consist of the alphabetic characters (A-Z and a-z), digits (0-9) and the hyphen (-) in accordance with IETF RFC 1035 \[19\]. Each label shall begin and end with either an alphabetic character or a digit in accordance with IETF RFC 1123 \[20\]. The case of alphabetic characters is not significant.

If the (U)SIM is not provisioned with the FQDN or IP address of the Provisioning GANC, the UE derives an FQDN from the IMSI to identify the Provisioning GANC. The UE shall derive such an FQDN as follows:

1\. create a domain name as specified in 17.3.1;

2\. add the label "pganc." to the beginning of the domain name.

An example of an FQDN for a Provisioning GANC is:

IMSI in use: 234150999999999;

Where:

MCC = 234;

MNC = 15;

MSIN = 0999999999,

Which gives the FQDN: pganc.gan.mnc015.mcc234.pub.3gppnetwork.org.

NOTE: If it is not possible for the UE to identify whether a 2 or 3 digit MNC is used (e.g. SIM is inserted and the length of MNC in the IMSI is not available in the "Administrative data" data file), it is implementation dependent how the UE determines the length of the MNC (2 or 3 digits).
