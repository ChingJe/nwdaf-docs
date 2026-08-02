---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: '25'
title: 25 Identification of Online Charging System
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 25 Identification of Online Charging System

## 25.1 Introduction

This clause describes the format of the home network domain name of the Online Charging System (OCS), needed to access the Online Charging System. For further information on the use of this home network domain name, see TS 29.212 \[106\]. For more information on the ".3gppnetwork.org" domain name and its applicability, see Annex D of the present document.

## 25.2 Home network domain name

The home network domain name of the OCS shall be in the form of an Internet domain name, e.g. operator.com, as specified in IETF RFC 1035 \[19\] and IETF RFC 1123 \[20\]. The home network domain of the OCS consists of several labels. Each label shall consist of the alphabetic characters (A-Z and a-z), digits (0-9) and the hyphen (-) in accordance with IETF RFC 1035 \[19\]. Each label shall begin and end with either an alphabetic character or a digit in accordance with IETF RFC 1123 \[20\]. The case of alphabetic characters is not significant.

If the home network domain of the OCS is not known (e.g. through an available static address or through its reception from another node), it shall be:

\- in the form of "ocs.mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org", where "\<MNC\>" and "\<MCC\>" fields correspond to the MNC and MCC of the operator's PLMN to which the OCS belongs. Both the "\<MNC\>" and "\<MCC\>" fields are 3 digits long. If the MNC of the PLMN is 2 digits, then a zero shall be added at the beginning; and

\- derived from the subscriber's IMSI, as described in the following steps:

1\. take the first 5 or 6 digits, depending on whether a 2 or 3 digit MNC is used (see TS 31.102 \[27\]) and separate them into MCC and MNC; if the MNC is 2 digits then a zero shall be added at the beginning;

2\. use the MCC and MNC derived in step 1 to create the "mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org" domain name;

3\. add the label "ocs" to the beginning of the domain name.

An example of a home network domain name is:

IMSI in use: 234150999999999;

Where:

MCC = 234;

MNC = 15;

MSIN = 0999999999;

Which gives the home network domain name: ocs.mnc015.mcc234.3gppnetwork.org.

NOTE: It is implementation dependent to determine that the length of the MNC is 2 or 3 digits.
