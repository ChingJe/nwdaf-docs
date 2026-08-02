---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: '21'
title: 21 Addressing and Identification for Dual Stack Mobile IPv6 (DSMIPv6)
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 21 Addressing and Identification for Dual Stack Mobile IPv6 (DSMIPv6)

## 21.1 Introduction

This clause describes the format of the parameters needed by the UE to use Dual Stack Mobile IPv6 (DSMIPv6 as specified in TS 23.327 \[76\] and TS 23.402 \[68\].

## 21.2 Home Agent – Access Point Name (HA-APN)

### 21.2.1 General

The HA-APN is composed of two parts as follows:

\- The HA-APN Network Identifier; this defines to which external network the HA is connected.

\- The HA-APN Operator Identifier; this defines in which PLMN the HA serving the HA-APN is located.

The HA-APN Operator Identifier is placed after the HA-APN Network Identifier. The HA-APN consisting of both the Network Identifier and Operator Identifier corresponds to a FQDN of a HA; the HA-APN has, after encoding as defined in the paragraph below, a maximum length of 100 octets.

The structure of the HA-APN shall follow the Name Syntax defined in IETF RFC 2181 \[18\], IETF RFC 1035 \[19\] and IETF RFC 1123 \[20\]. The HA-APN consists of one or more labels.

When encoded as a sequence of octets, each label is coded as a one octet length field followed by that number of octets coded as 8 bit ASCII characters.

When encoded as text string and for the purpose of presentation, a HA-APN is usually displayed as a string in which the labels are separated by dots (e.g. "Label1.Label2.Label3").

Following IETF RFC 1035 \[19\] the labels shall consist only of the alphabetic characters (A-Z and a-z), digits (0-9) and the hyphen (-). Following IETF RFC 1123 \[20\], the label shall begin and end with either an alphabetic character or a digit. The case of alphabetic characters is not significant. The HA-APN is not terminated by a length byte of zero.

Different stage 3 protocol specifications may specify different ways of W-APN encoding taking precedence over definitions from this clause.

### 21.2.2 Format of HA-APN Network Identifier

The HA-APN Network Identifier follows the format defined for APNs in clause 9.1.1. In addition to what has been defined in clause 9.1.1 the HA-APN Network Identifier shall not contain "ha-apn." or "w-apn." and not end in ".3gppnetwork.org".

A HA-APN Network Identifier may be used to access a service associated with a HA. This may be achieved by defining:

\- a HA-APN which corresponds to a FQDN of a HA, and which is locally interpreted by the HA as a request for a specific service, or

\- a HA-APN Network Identifier consisting of 3 or more labels and starting with a Reserved Service Label, or a HA-APN Network Identifier consisting of a Reserved Service Label alone, which indicates a HA by the nature of the requested service. Reserved Service Labels and the corresponding services they stand for shall be agreed between operators who have roaming agreements.

As an example, the HA-APN for MCC 345 and MNC 12 is coded in the DNS as:

"internet.ha-apn.mnc012.mcc345.pub.3gppnetwork.org".

where "internet" is the HA-APN Network Identifier and "mnc012.mcc345.pub.3gppnetwork.org " is the HA-APN Operator Identifier.

### 21.2.3 Format of HA-APN Operator Identifier

The HA-APN Operator Identifier is composed of six labels. The last three labels shall be "pub.3gppnetwork.org". The second and third labels together shall uniquely identify the PLMN. The first label distinguishes the domain name as a HA-APN.

For each operator, there is a default HA-APN Operator Identifier (i.e. domain name). This default HA-APN Operator Identifier is derived from the IMSI as follows:

"ha-apn.mnc\<MNC\>.mcc\<MCC\>.pub.3gppnetwork.org"

where:

"mnc" and "mcc" serve as invariable identifiers for the following digits.

\<MNC\> and \<MCC\> are derived from the components of the IMSI defined in clause 2.2.

Alternatively, the default HA‑APN Operator Identifier is derived using the MNC and MCC of the VPLMN.

In order to guarantee inter-PLMN DNS translation, the \<MNC\> and \<MCC\> coding used in the "ha‑apn.mnc\<MNC\>.mcc\<MCC\>.pub.3gppnetwork.org" format of the HA-APN OI shall be:

\- \<MNC\> = 3 digits

\- \<MCC\> = 3 digits

If there are only 2 significant digits in the MNC, one "0" digit shall be inserted at the left side to fill the 3 digits coding of MNC in the HA-APN OI.

As an example, the HA-APN OI for MCC 345 and MNC 12 is coded in the DNS as:

"ha-apn.mnc012.mcc345.pub.3gppnetwork.org".
