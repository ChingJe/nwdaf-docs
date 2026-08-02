---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: '14'
title: 14 Numbering, addressing and identification for 3GPP System to WLAN Interworking
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 14 Numbering, addressing and identification for 3GPP System to WLAN Interworking

## 14.1 Introduction

This clause describes the format of the parameters needed to access the 3GPP system supporting the WLAN interworking. For further information on the use of the parameters see TS 24.234 \[48\]. For more information on the ".3gppnetwork.org" domain name and its applicability, see Annex D of the present document.

NOTE: The WLAN Network Selection and WLAN/3GPP Radio Interworking features supersede the I-WLAN feature from Rel-12 onwards, therefore all I-WLAN related requirements specified in the present Clause are no longer maintained.

## 14.2 Home network realm

The home network realm shall be in the form of an Internet domain name, e.g. operator.com, as specified in RFC 1035 \[19\].

When attempting to authenticate within WLAN access, the WLAN UE shall derive the home network domain name from the IMSI as described in the following steps:

1\. take the first 5 or 6 digits, depending on whether a 2 or 3 digit MNC is used (see TS 31.102 \[27\], TS 51.011 \[66\]) and separate them into MCC and MNC; if the MNC is 2 digits then a zero shall be added at the beginning;

2\. use the MCC and MNC derived in step 1 to create the "mnc\<MNC\>.mcc\<MCC\>. 3gppnetwork.org" domain name;

3\. add the label "wlan." to the beginning of the domain name.

An example of a WLAN NAI realm is:

IMSI in use: 234150999999999;

Where:

MCC = 234;

MNC = 15;

MSIN = 0999999999

Which gives the home network domain name: wlan.mnc015.mcc234.3gppnetwork.org.

NOTE: If it is not possible for the WLAN UE to identify whether a 2 or 3 digit MNC is used (e.g. SIM is inserted and the length of MNC in the IMSI is not available in the "Administrative data" data file), it is implementation dependent how the WLAN UE determines the length of the MNC (2 or 3 digits).

## 14.3 Root NAI

The Root NAI shall take the form of a NAI, and shall have the form username@realm as specified in clause 2.1 of IETF RFC 4282 \[53\].

The username part format of the Root NAI shall comply with IETF RFC 4187 \[50\] when EAP AKA authentication is used and with IETF RFC 4186 \[51\], when EAP SIM authentication is used.

When the username part includes the IMSI, the Root NAI shall be built according to the following steps:

1\. Generate an identity conforming to NAI format from IMSI as defined in EAP SIM \[51\] and EAP AKA \[50\] as appropriate;

2\. Convert the leading digits of the IMSI, i.e. MNC and MCC, into a domain name, as described in clause 14.2.

The result will be a root NAI of the form:

"0\<IMSI\>@wlan.mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org", for EAP AKA authentication and "1\<IMSI\>@wlan.mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org", for EAP SIM authentication

For example, for EAP AKA authentication: If the IMSI is 234150999999999 (MCC = 234, MNC = 15), the root NAI then takes the form 0234150999999999@wlan.mnc015.mcc234.3gppnetwork.org.

## 14.4 Decorated NAI

The Decorated NAI shall take the form of a NAI and shall have the form 'homerealm!username@otherrealm' as specified in clause 2.7 of the IETF RFC 4282 \[53\].

The realm part of Decorated NAI consists of 'otherrealm', see the IETF draft  2486-bisRFC 4282 \[53\]. 'Homerealm' is the realm as specified in clause 14.2, using the HPLMN ID ('homeMCC' + 'homeMNC)'. 'Otherrealm' is the realm built using the PLMN ID (visitedMCC + visited MNC) of the PLMN selected as a result of WLAN PLMN selection (see TS 24.234 \[48\]).

The username part format of the Root NAI shall comply with IETF RFC 4187 \[50\] when EAP AKA authentication is used and with IETF RFC 4186 \[51\], when EAP SIM authentication is used.

When the username part of Decorated NAI includes the IMSI, it shall be built following the same steps specified for Root NAI in clause 14.3.

The result will be a decorated NAI of the form:

"wlan.mnc\<homeMNC\>.mcc\<homeMCC\>.3gppnetwork.org !0\<IMSI\>@wlan.mnc\<visitedMNC\>.mcc\<visitedMCC\>.3gppnetwork.org", for EAP AKA authentication and " wlan.mnc\<homeMNC\>.mcc\<homeMCC\>.3gppnetwork.org !1\<IMSI\>@wlan.mnc\<visitedMNC\>.mcc\<visitedMCC\>.3gppnetwork.org ", for EAP SIM authentication

For example, for EAP AKA authentication: If the IMSI is 234150999999999 (MCC = 234, MNC = 15) and the PLMN ID of the Selected PLMN is MCC = 610, MNC = 71 then the Decorated NAI takes the form wlan.mnc015.mcc234.3gppnetwork.org!0234150999999999@wlan.mnc071.mcc610.3gppnetwork.org.

NOTE: the 'otherrealm' specified in the present document is resolved by the WLAN AN. If the WLAN AN does not have access to the GRX, then the WLAN AN should resolve the realm by other means e.g. static look-up table, private local DNS server acting as an authoritative name server for that sub-domain.

## 14.4A Fast Re‑authentication NAI

The Fast Re-authentication NAI in both EAP-SIM and EAP-AKA shall take the form of a NAI as specified in clause 2.1 of IETF RFC 4282  \[53\]. If the 3GPP AAA server does not return a complete NAI, the Fast Re-authentication NAI shall consist of the username part of the fast re-authentication identity as returned from the 3GPP AAA server and the same realm as used in the permanent user identity. If the 3GPP AAA server returns a complete NAI as the re-authentication identity, then this NAI shall be used. The username part of the fast re-authentication identity shall be decorated as described in 14.4 if the Selected PLMN is different from the HPLMN.

NOTE: The permanent user identity is either the root or decorated NAI as defined in clauses 14.3 and 14.4.

EXAMPLE 1: If the fast re-authentication identity returned by the 3GPP AAA Server is 458405627015 and the IMSI is 234150999999999 (MCC = 234, MNC = 15), the Fast Re-authentication NAI for the case when NAI decoration is not used takes the form: 458405627015@wlan.mnc015.mcc234.3gppnetwork.org

EXAMPLE 2: If the fast re-authentication identity returned by the 3GPP AAA Server is "458405627015@aaa1.wlan.mnc015.mcc234.3gppnetwork.org" and the IMSI is 234150999999999 (MCC = 234, MNC = 15), the Fast Re-authentication NAI for the case when NAI decoration is not used takes the form: 458405627015@aaa1.wlan.mnc015.mcc234.3gppnetwork.org

EXAMPLE 3: If the fast re-authentication identity returned by the 3GPP AAA Server is 458405627015 and the IMSI is 234150999999999 (MCC = 234, MNC = 15), and the PLMN ID of the Selected PLMN is MCC = 610, MNC = 71, the Fast Re-authentication NAI takes the form: wlan.mnc015.mcc234.3gppnetwork.org !458405627015@wlan.mnc071.mcc610.3gppnetwork.org

## 14.5 Temporary identities

The Temporary identities (Pseudonyms and re-authentication identities) shall take the form of a NAI username as specified in clause 2.1 of the IETF RFC 4282 \[53\].

Temporary identity shall be generated as specified in clause 6.4.1 of TS 33.234 \[55\]. This part of the temporary identity shall follow the UTF-8 transformation format specified in IETF RFC 2279 \[54\] except for the following reserved hexadecimal octet value:

FF.

When the temporary identity username is coded with FF, this reserved value is used to indicate the special case when no valid temporary identity exists in the WLAN UE (see TS 24.234 \[48\]). The network shall not allocate a temporary identity with the whole username coded with the reserved hexadecimal value FF.

For EAP-AKA authentication, the username portion of the pseudonym identity shall be prepended with the single digit "2" and the username portion of the fast re-authentication identity shall be prepended with the single digit "4" as specified in clause 4.1.1.7 of IETF RFC 4187 \[50\].

For EAP-SIM authentication, the username portion of the pseudonym identity shall be prepended with the single digit "3" and the username portion of the fast re-authentication identity shall be prepended with the single digit "5" as specified in clause 4.2.1.7 of IETF RFC 4186 \[51\].

## 14.6 Alternative NAI

The Alternative NAI shall take the form of a NAI, i.e. 'any_username@REALM' as specified of IETF RFC 4282 \[53\]. The Alternative NAI shall not be routable from any AAA server.

The Alternative NAI shall contain a username part which is not derived from the IMSI. The username part shall not be a null string.

The REALM part of the NAI shall be "unreachable.3gppnetwork.org".

The result shall be an NAI in the form of:

"\<any_non_null_string\>@unreachable.3gppnetwork.org"

## 14.7 W-APN

The W-APN is composed of two parts as follows:

\- The W-APN Network Identifier; this defines to which external network the PDG is connected.

\- The W-APN Operator Identifier; this defines in which PLMN the PDG serving the W-APN is located.

The W-APN Operator Identifier is placed after the W-APN Network Identifier. The W-APN consisting of both the Network Identifier and Operator Identifier corresponds to a FQDN of a PDG; the W-APN has, after encoding as defined in the paragraph below, a maximum length of 100 octets.

The structure of the W-APN shall follow the Name Syntax defined in IETF RFC 2181 \[18\],  1035 \[19\] and IETF RFC 1123 \[20\]. The W-APN consists of one or more labels.

When encoded as a sequence of octets, each label is coded as a one octet length field followed by that number of octets coded as 8 bit ASCII characters.

When encoded as text string and for the purpose of presentation, a W-APN is usually displayed as a string in which the labels are separated by dots (e.g. "Label1.Label2.Label3")

Following IETF RFC 1035 \[19\] the labels shall consist only of the alphabetic characters (A-Z and a-z), digits (0-9) and the hyphen (-). Following IETF RFC 1123 \[20\], the label shall begin and end with either an alphabetic character or a digit. The case of alphabetic characters is not significant. The W-APN is not terminated by a length byte of zero.

The W-APN for the support of IMS Emergency calls shall take the form of a common, reserved Network Identifier described in clause 14.7.1 together with the usual W-APN Operator Identifier as described in clause 14.7.2.

Different stage 3 protocol specifications may specify different ways of W-APN encoding taking precedence over definitions from this clause.

### 14.7.1 Format of W-APN Network Identifier

The W-APN Network Identifier follows the format defined for APNs in clause 9.1.1. In addition to what has been defined in clause 9.1.1 the W-APN Network Identifier shall not contain "w-apn." and not end in ".3gppnetwork.org".

A W-APN Network Identifier may be used to access a service associated with a PDG. This may be achieved by defining:

\- a W-APN which corresponds to a FQDN of a PDG, and which is locally interpreted by the PDG as a request for a specific service, or

\- a W-APN Network Identifier consisting of 3 or more labels and starting with a Reserved Service Label, or a W-APN Network Identifier consisting of a Reserved Service Label alone, which indicates a PDG by the nature of the requested service. Reserved Service Labels and the corresponding services they stand for shall be agreed between operators who have WLAN roaming agreements.

The W-APN Network Identifier for the support of IMS Emergency calls shall take the form of a common, reserved Network Identifier of the form "sos".

As an example, the W-APN for MCC 345 and MNC 12 is coded in the DNS as:

"sos.w-apn.mnc012.mcc345.pub.3gppnetwork.org".

where "sos" is the W-APN Network Identifier and " mnc012.mcc345.pub.3gppnetwork.org " is the W-APN Operator Identifier.

### 14.7.2 Format of W-APN Operator Identifier

The W-APN Operator Identifier is composed of six labels. The last three labels shall be "pub.3gppnetwork.org". The second and third labels together shall uniquely identify the PLMN. The first label distinguishes the domain name as a W-APN.

For each operator, there is a default W-APN Operator Identifier (i.e. domain name). This default W-APN Operator Identifier is derived from the IMSI as follows:

"w-apn.mnc\<MNC\>.mcc\<MCC\>.pub.3gppnetwork.org"

where:

"mnc" and "mcc" serve as invariable identifiers for the following digits.

\<MNC\> and \<MCC\> are derived from the components of the IMSI defined in clause 2.2.

Alternatively, the default W‑APN Operator Identifier is derived using the MNC and MCC of the VPLMN. See TS 24.234 \[48\] for more information.

The default W-APN Operator Identifier is used in both non‑roaming and roaming situations when attempting to translate a W-APN consisting only of a Network Identifier into the IP address of the PDG in the HPLMN.

In order to guarantee inter-PLMN DNS translation, the \<MNC\> and \<MCC\> coding used in the "w‑apn.mnc\<MNC\>.mcc\<MCC\>.pub.3gppnetwork.org" format of the W-APN OI shall be:

\- \<MNC\> = 3 digits

\- \<MCC\> = 3 digits

If there are only 2 significant digits in the MNC, one "0" digit shall be inserted at the left side to fill the 3 digits coding of MNC in the W-APN OI.

As an example, the W-APN OI for MCC 345 and MNC 12 is coded in the DNS as:

"w-apn.mnc012.mcc345.pub.3gppnetwork.org".

### 14.7.3 Alternative Format of W-APN Operator Identifier

For situations when the PDG serving the W-APN is located in such network that is not part of the GRX (i.e. the Interoperator IP backbone), the default Operator Identifier described in clause 14.7.2 is not available for use. This restriction originates from the ".3gppnetwork.org" domain, which is only available in GRX DNS for actual use. Thus an alternative format of W-APN Operator Identifier is required for this case.

The Alternative W-APN Operator Identifiers shall be constructed as follows:

"w-apn.\<valid operator's REALM\>"

where:

\<valid operator's REALM\> corresponds to REALM names owned by the operator hosting the PDG serving the desired W-APN.

REALM names are required to be unique, and are piggybacked on the administration of the Public Internet DNS namespace. REALM names may also belong to the operator of the VPLMN.

As an example, the W-APN OI for the Operator REALM "notareal.com" is coded in the Public Internet DNS as:

"w-apn.notareal.com".

## 14.8 Emergency Realm and Emergency NAI for Emergency Cases

The emergency realm shall be of the form of a home network realm as described in clause 14.2 prefixed with the label "sos." at the beginning of the domain name.

An example of a WLAN emergency NAI realm is:

IMSI in use: 234150999999999;

Where:

MCC = 234;

MNC = 15;

MSIN = 0999999999

Which gives the home network domain name: sos.wlan.mnc015.mcc234.3gppnetwork.org.

The NAI for emergency cases shall be of the form as specified in clauses 14.3 and 14.4, with the addition of the emergency realm as described above for PLMNs where the emergency realm is supported.

When UE is using I-WLAN as the access network for IMS emergency calls and IMSI is not available, the Emergency NAI shall be an NAI compliant with IETF RFC 4282 \[53\] consisting of username and realm, either constructed with IMEI or MAC address, as specified in TS 33.234 \[55\]. The exact format shall be:

imei\<IMEI\>@sos.wlan.mnc\<visitedMNC\>.mcc\<visitedMCC\>.3gppnetwork.org

or if IMEI is not available,

mac\<MAC\>@sos.wlan.mnc\<visitedMNC\>.mcc\<visitedMCC\>.3gppnetwork.org

The realm part of the above NAI consists of the realm built using the PLMN ID (visitedMCC + visitedMNC) of the PLMN selected as a result of the network selection procedure, as specified in clause 5.2.5.4 of the TS 24.234 \[48\].

The MNC and MCC shall be with 3 digits coded. If there are only 2 significant digits in the MNC, one "0" digit shall be inserted at the left side to fill the 3 digits coding of MNC in the realm of the NAI.

For example, if the IMEI is 219551288888888, and the selected PLMN is with MCC 345 and MNC 12, the Emergency NAI then takes the form of imei219551288888888@sos.wlan.mnc012.mcc345.3gppnetwork.org.

For example, if the MAC address is 44-45-53-54-00-AB, and the selected PLMN is with MCC 345 and MNC 12, the Emergency NAI then takes the form of mac4445535400AB@sos.wlan.mnc012.mcc345.3gppnetwork.org, where the MAC address is represented in hexadecimal format without separators.
