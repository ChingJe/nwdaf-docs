---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: 9A
title: 9A Definition of Data Network Name
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 9A Definition of Data Network Name

In 5GS, the Data Network Name (DNN) is equivalent to an APN in EPS. The DNN is a reference to a data network, it may be used e.g. to select SMF or UPF.

The requirements for APN in clause 9 shall apply for DNN in a 5GS as well.

In SNPNs the DNN Operator Identifier shall take the format

"nid\<NID\>.mnc\<MNC\>.mcc\<MCC\>.gprs"

where:

"nid","mnc" and "mcc" serve as invariable identifiers for the following digits.

## 9.0 General

Access Point Name as used in the Domain Name System (DNS) Procedures defined in TS 29.303 \[73\] is specified in clause 19.4.2.2.

## 9.1 Structure of APN

The APN is composed of two parts as follows:

\- The APN Network Identifier; this defines to which external network the GGSN/PGW is connected and optionally a requested service by the MS. This part of the APN is mandatory.

\- The APN Operator Identifier; this defines in which PLMN GPRS/EPS backbone the GGSN/PGW is located. This part of the APN is optional.

NOTE 1: The APN Operator Identifier is mandatory on certain interfaces, see the relevant stage 3 specifications.

The APN Operator Identifier is placed after the APN Network Identifier. An APN consisting of both the Network Identifier and Operator Identifier corresponds to a DNS name of a GGSN/PGW; the APN has, after encoding as defined in the paragraph below, a maximum length of 100 octets.

The structure of the APN shall follow the Name Syntax defined in RFC 2181 \[18\], RFC 1035 \[19\] and RFC 1123 \[20\]. The APN consists of one or more labels.

When encoded as a sequence of octets each label is coded as a one octet length field followed by that number of octets coded as 8 bit ASCII characters.

When encoded as text string and for the purpose of presentation, an APN is usually displayed as a string in which the labels are separated by dots (e.g. "Label1.Label2.Label3").

Following RFC 1035 \[19\] the labels shall consist only of the alphabetic characters (A-Z and a-z), digits (0-9) and the hyphen (-). Following RFC 1123 \[20\], the label shall begin and end with either an alphabetic character or a digit. The case of alphabetic characters is not significant. The APN is not terminated by a length byte of zero.

NOTE 2: A length byte of zero is added by the SGSN/MME at the end of the APN before interrogating a DNS server.

Different stage 3 protocol specifications may specify different ways of APN encoding taking precedence over definitions from this clause.

### 9.1.1 Format of APN Network Identifier

The APN Network Identifier shall contain at least one label and shall have, after encoding as defined in clause 9.1 above, a maximum length of 63 octets. An APN Network Identifier shall not start with any of the strings "rac", "lac", "sgsn" or "rnc", and it shall not end in ".gprs", i.e. the last label of the APN Network Identifier shall not be "gprs". Further, it shall not take the value "\*".

In order to guarantee uniqueness of APN Network Identifiers within or between GPRS/EPS PLMN, an APN Network Identifier containing more than one label shall correspond to an Internet domain name. This name should only be allocated by the PLMN if that PLMN belongs to an organisation which has officially reserved this name in the Internet domain. Other types of APN Network Identifiers are not guaranteed to be unique within or between GPRS/EPS PLMNs.

An APN Network Identifier may be used to access a service associated with a GGSN/PGW. This may be achieved by defining:

\- an APN which corresponds to a FQDN of a GGSN/PGW, and which is locally interpreted by the GGSN/PGW as a request for a specific service, or

\- an APN Network Identifier consisting of 3 or more labels and starting with a Reserved Service Label, or an APN Network Identifier consisting of a Reserved Service Label alone, which indicates a GGSN/PGW by the nature of the requested service. Reserved Service Labels and the corresponding services they stand for shall be agreed between operators who have GPRS/EPS roaming agreements.

### 9.1.2 Format of APN Operator Identifier

The APN Operator Identifier is composed of three labels. The last label (or domain) shall be "gprs". The first and second labels together shall uniquely identify the GPRS/EPS PLMN.

For each operator, there is a default APN Operator Identifier (i.e. domain name). This default APN Operator Identifier is derived from the IMSI as follows:

"mnc\<MNC\>.mcc\<MCC\>.gprs"

where:

"mnc" and "mcc" serve as invariable identifiers for the following digits.

\<MNC\> and \<MCC\> are derived from the components of the IMSI defined in clause 2.2.

This default APN Operator Identifier is used for home routed inter-PLMN roaming situations when attempting to translate an APN consisting only of a Network Identifier into the IP address of the GGSN/PGW in the HPLMN. The PLMN may provide DNS translations for other, more human-readable, APN Operator Identifiers in addition to the default Operator Identifier described above.

Alternatively, in the roaming case if the GGSN/PGW from the VPLMN is to be selected, the APN Operator Identifier for the UE is constructed from the serving network PLMN ID. In this case, the APN-OI replacement field, if received, shall be ignored.

In order to guarantee inter-PLMN DNS translation, the \<MNC\> and \<MCC\> coding used in the "mnc\<MNC\>.mcc\<MCC\>.gprs" format of the APN OI shall be:

\- \<MNC\> = 3 digits

\- \<MCC\> = 3 digits

\- If there are only 2 significant digits in the MNC, one "0" digit is inserted at the left side to fill the 3 digits coding of MNC in the APN OI.

As an example, the APN OI for MCC 345 and MNC 12 will be coded in the DNS as "mnc012.mcc345.gprs".

The APN-OI replacement is used for selecting the GGSN/PGWfor non-roaming and home routed scenarios. The format of the domain name used in the APN-OI replacement field (as defined in TS 23.060 \[3\] and TS 23.401 \[72\]) is the same as the default APN-OI as defined above except that it may be preceded by one or more labels each separated by a dot. It is up to the operators to determine what labels shall precede the "mnc\<MNC\>.mcc\<MCC\>.gprs" trailing labels (see clause 5.1.1.1 in TS 29.303 \[73\] and also clause 13.1 in TS 23.060 \[3\]).

EXAMPLE 1: province1.mnc012.mcc345.gprs

EXAMPLE 2: ggsn-cluster-A.provinceB.mnc012.mcc345.gprs

The APN constructed using the APN-OI replacement field is only used for DNS translation. The APN when being sent to other network entities over GTP interfaces shall follow the rules as specified in TS 23.060 \[3\] and TS 23.401 \[72\].

## 9.2 Definition of the Wild Card APN

The APN field in the HLR may contain a wild card APN if the HPLMN operator allows the subscriber to access any network of a given PDP Type. If an SGSN has received such a wild card APN, it may either choose the APN Network Identifier received from the Mobile Station or a default APN Network Identifier for addressing the GGSN when activating a PDP context.

### 9.2.1 Coding of the Wild Card APN

The wild card APN is coded as an APN with "\*" as its single label, (i.e. a length octet with value one, followed by the ASCII code for the asterisk).

## 9.3 Definition of Emergency APN

The Emergency APN (Em-APN) is an APN used to derive a PDN GW selected for IMS Emergency call support. The exact encoding of the Em-APN is the responsibility of each PLMN operator as it is only valid within a given PLMN.
