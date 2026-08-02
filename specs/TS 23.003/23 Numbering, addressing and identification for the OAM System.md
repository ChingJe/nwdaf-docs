---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: '23'
title: 23 Numbering, addressing and identification for the OAM System
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 23 Numbering, addressing and identification for the OAM System

## 23.1 Introduction

This clause describes some information needed to access the OAM system as specified in TS 36.300 \[91\]. For more information on the ".3gppnetwork.org" domain name and its applicability, see Annex D of the present document.

## 23.2 OAM System Realm/Domain

The OAM System Realm/Domain shall be in the form of an Internet domain name, e.g. operator.com, as specified in IETF RFC 1035 \[19\] and IETF RFC 1123 \[20\]. The OAM System Realm/Domain consists of several labels. Each label shall consist of the alphabetic characters (A-Z and a-z), digits (0-9) and the hyphen (-) in accordance with IETF RFC 1035 \[19\]. Each label shall begin and end with either an alphabetic character or a digit in accordance with IETF RFC 1123 \[20\]. The case of alphabetic characters is not significant.

The OAM System Realm/Domain shall be in the form of "oam.mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org", where "\<MNC\>" and "\<MCC\>" fields correspond to the MNC and MCC of the operator's PLMN. Both the "\<MNC\>" and "\<MCC\>" fields are 3 digits long. If the MNC of the PLMN is 2 digits, then a zero shall be added at the beginning.

For example, the OAM System Realm/Domain of an IMSI shall be derived as described in the following steps:

1\. take the first 5 or 6 digits, depending on whether a 2 or 3 digit MNC is used (see TS 31.102 \[27\]) and separate them into MCC and MNC; if the MNC is 2 digits then a zero shall be added at the beginning;

2\. use the MCC and MNC derived in step 1 to create the "mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org" domain name;

3\. add the label "oam" to the beginning of the domain name.

An example of an OAM System Realm/Domain is:

IMSI in use: 234150999999999;

Where:

MCC = 234;

MNC = 15;

MSIN = 0999999999;

Which gives the OAM System Realm/Domain name: oam.mnc015.mcc234.3gppnetwork.org.

NOTE: If it is not possible for a Relay Node to identify whether a 2 or 3 digit MNC is used (e.g. USIM is inserted and the length of MNC in the IMSI is not available in the "Administrative data" data file), it is implementation dependent how the Relay Node determines the length of the MNC (2 or 3 digits).

## 23.3 Identifiers for Domain Name System procedures

### 23.3.1 Introduction

This clause describes Domain Name System (DNS) related identifiers used by the procedures specified in TS 29.303 \[73\].

### 23.3.2 Fully Qualified Domain Names (FQDNs)

#### 23.3.2.1 General

See clause 19.4.2.1.

#### 23.3.2.2 Relay Node Vendor-Specific OAM System

As part of the startup procedure, relay nodes (see TS 36.300 \[91\], clause 4.7) needs to discover its Operations and Maintainence (OAM) system. A relay node vendor-specific OAM system within an operator's network is identified using the relay node type allocation code from IMEI or IMEISV (IMEI-TAC), MNC and MCC from IMSI and in some cases also tracking area code information associated to the eNB serving the relay node.

A subdomain name for use by EUTRAN OAM system nodes shall be derived from the MNC and MCC by adding the label "eutran" to the beginning of the OAM System Realm/Domain (see clause 23.2).

The vendor-specific relay node OAM system FQDN shall be constructed as following:

\- tac-lb\<TAC-low-byte\>.tac-hb\<TAC-high-byte\>.imei-tac\<IMEI-TAC\>.eutran-rn.oam.mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org

The IMEI-TAC is 8 decimal digits (see clause 6.2).

NOTE: IMEI-TAC is used for the type allocation code from IMEI or IMEISV instead of TAC in this clause in order to separate it from the tracking area code (TAC).

The TAC is a 16 bit integer. \<TAC-high-byte\> is the hexadecimal string of the most significant byte in the TAC and \<TAC-low-byte \> is the hexadecimal string of the least significant byte. If there are less than 2 significant digits in \<TAC-high-byte\> or \<TAC-low-byte \>, "0" digit(s) shall be inserted at the left side to fill the 2 digit coding.

#### 23.3.2.3 Multi-vendor eNodeB Plug-and Play Vendor-Specific OAM System

##### 23.3.2.3.1 General

This clause describes the Fully Qualified Domain Names (FQDNs) used in Multi Vendor Plug and Connect (MvPnC) procedures (see TS 32.508 \[102\]).

The FQDNs used in MvPnC shall be in the form of an Internet domain name and follow the general encoding rules specified in clause 19.4.2.1.

The format of FQDNs used in MvPnC shall follow the "\<vendor ID\>.\<system\>.\<OAM realm\>" pattern.

NOTE: "\<vendor ID\>.\<system\>.oam" represents the \<service_id\> shown in the first row of table E.1.

The \<vendor ID\> label is optional and is only used in the operator deployments where multiple instances of a particular network entity type are not provided by the same vendor. If present, the \<vendor ID\> label shall be in the form "vendor\<ViD\>", where \<ViD\> field corresponds to the ID of the vendor.

The format of the ViD is vendor specific.

The details of the \<system\> label are specified in the clauses below.

##### 23.3.2.3.2 Certification Authority server

The Certification Authority server (CA/RA) FQDN shall be derived as follows. The "cara" \<system\> label is added in front of the operator's OAM realm domain name:

cara.oam.mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org

If particular operator deployment scenarios where there are multiple CA/RA servers (one per vendor), the \<vendor ID\> label is added in front of the "cara" label:

vendor\<ViD\>.cara.oam.mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org

An example of a CA/RA FQDN is:

MCC = 123;

MNC = 45;

ViD = abcd;

which gives the CA/RA FQDN: "cara.oam.mnc045.mcc123.3gppnetwork.org" and "vendorabcd.cara.mnc045.mcc123.3gppnetwork.org".

##### 23.3.2.3.3 Security Gateway

The Security Gateway (SeGW) FQDN shall be derived as follows. The "segw" \<system\> label is added in front of the operator's OAM realm domain name:

segw.oam.mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org

If particular operator deployment scenarios where there are multiple Security Gateways (one per vendor), the \<vendor ID\> label is added in front of the "segw" label:

vendor\<ViD\>.segw.oam.mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org

An example of a SeGW FQDN is:

MCC = 123;

MNC = 45;

ViD = abcd;

which gives the SeGW FQDN: "segw.oam.mnc045.mcc123.3gppnetwork.org" and "vendorabcd.segw.mnc045.mcc123.3gppnetwork.org".

##### 23.3.2.3.4 Element Manager

The Element Manager (EM) FQDN shall be derived as follows. The "em" \<system\> label is added in front of the operator's OAM realm domain name:

em.oam.mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org

If particular operator deployment scenarios where there are multiple Element Managers (one per vendor), the \<vendor ID\> label is added in front of the "em" label:

vendor\<ViD\>.em.oam.mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org

An example of a EM FQDN is:

MCC = 123;

MNC = 45;

ViD = abcd;

which gives the EM FQDN: "em.oam.mnc045.mcc123.3gppnetwork.org" and "vendorabcd.em.mnc045.mcc123.3gppnetwork.org".
