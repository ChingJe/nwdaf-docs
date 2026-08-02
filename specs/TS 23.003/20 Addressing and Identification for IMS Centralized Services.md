---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: '20'
title: 20 Addressing and Identification for IMS Centralized Services
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 20 Addressing and Identification for IMS Centralized Services

## 20.1 Introduction

This clause describes the format of the parameters needed specifically for IMS Centralized Services (ICS). For further information on the use of ICS parameters, see TS 23.292 \[70\].

## 20.2 UE based solution

In this solution, the UE is provisioned with an ICS specific client that simply reuses IMS registration as defined in TS 23.228 \[24\]. Therefore, ICS capable UE shall reuse the identities defined in clause 13.

## 20.3 Network based solution

### 20.3.1 General

In this solution the MSC Server enhanced for ICS performs a special IMS registration on behalf of the UE. Thus, the MSC Server enhanced for ICS shall use a Private User Identity and Temporary Public User Identity that are different to those defined in clause 13 (see TS 23.292 \[70\], clause 4.6.2 for more information). Furthermore, the MSC Server enhanced for ICS derives a Conference Factory URI that is known to the home IMS. These are defined in the following clauses.

### 20.3.2 Home network domain name

The home network domain name shall be in the form of an Internet domain name, e.g. operator.com, as specified in IETF RFC 1035 \[19\] and IETF RFC 1123 \[20\]. The home network domain name consists of several labels. Each label shall consist of the alphabetic characters (A-Z and a-z), digits (0-9) and the hyphen (-) in accordance with IETF RFC 1035 \[19\]. Each label shall begin and end with either an alphabetic character or a digit in accordance with IETF RFC 1123 \[20\]. The case of alphabetic characters is not significant.

The MSC Server enhanced for ICS shall derive the home network domain name from the subscriber's IMSI as described in the following steps:

1\. Take the first 5 or 6 digits, depending on whether a 2 or 3 digit MNC is used (see TS 31.102 \[27\]) and separate them into MCC and MNC; if the MNC is 2 digits then a zero shall be added at the beginning.

2\. Use the MCC and MNC derived in step 1 to create the "mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org" domain name.

3\. Add the label "ics." to the beginning of the domain.

An example of a home network domain name is:

IMSI in use: 234150999999999;

where:

\- MCC = 234;

\- MNC = 15; and

\- MSIN = 0999999999,

which gives the home network domain name: ics.mnc015.mcc234.3gppnetwork.org

### 20.3.3 Private User Identity

The Private User Identity shall take the form of an NAI, and shall have the form "username@realm" as specified in clause 2.1 of IETF RFC 4282 \[53\].

The MSC Server enhanced for ICS shall derive the Private User Identity from the subscriber's IMSI as follows:

1\. Use the whole string of digits as the username part of the private user identity; and

2\. convert the leading digits of the IMSI, i.e. MNC and MCC, into a domain name, as described in clause 20.3.2.

The result will be a Private User Identity of the form "\<IMSI\>@ics.mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org". For example if the IMSI is 234150999999999 (MCC = 234, MNC = 15), the private user identity then takes the form 234150999999999@ics.mnc015.mcc234.3gppnetwork.org

### 20.3.4 Public User Identity

The Public User Identity shall take the form of a SIP URI (see IETF RFC 3261 \[26\]), and shall have the form "sip:username@domain".

The MSC Server enhanced for ICS shall derive the Public User Identity from the subscriber's IMSI. The Public User Identity shall consist of the string "sip:" appended with a username and domain portion equal to the IMSI derived Private User Identity described in clause 20.3.3. An example using the same example IMSI from clause 20.3.3 can be found below:

EXAMPLE: "sip:234150999999999@ics.mnc015.mcc234.3gppnetwork.org".

### 20.3.5 Conference Factory URI

The Conference Factory URI shall take the form of a SIP URI (see IETF RFC 3261 \[26\]) with a host portion set to the home network domain name as described in clause 20.3.2 prefixed with "conf-factory.". An example using the same example IMSI from clause 20.3.2 can be found below:

EXAMPLE: "sip:conf-factory.ics.mnc015.mcc234.3gppnetwork.org".

The user portion of the SIP URI is optional and implementation specific.
