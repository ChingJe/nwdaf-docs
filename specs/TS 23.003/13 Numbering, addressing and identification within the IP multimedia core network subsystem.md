---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: '13'
title: 13 Numbering, addressing and identification within the IP multimedia core network subsystem
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 13 Numbering, addressing and identification within the IP multimedia core network subsystem

## 13.1 Introduction

This clause describes the format of the parameters needed to access the IP multimedia core network subsystem. For further information on the use of the parameters see TS 23.228 \[24\] and TS 29.163 \[63\]. For more information on the ".3gppnetwork.org" domain name and its applicability, see Annex D of the present document. For more information on the ".invalid" top level domain see IETF RFC 2606 \[64\].

## 13.2 Home network domain name

The home network domain name shall be in the form of an Internet domain name, e.g. operator.com, as specified in IETF RFCIETF RFC 1035 \[19\] and IETF RFCIETF RFC 1123 \[20\]. The home network domain name consists of several labels. Each label shall consist of the alphabetic characters (A-Z and a-z), digits (0-9) and the hyphen (-) in accordance with IETF RFCIETF RFC 1035 \[19\]. Each label shall begin and end with either an alphabetic character or a digit in accordance with IETF RFCIETF RFC 1123 \[20\]. The case of alphabetic characters is not significant.

For 3GPP systems, if there is no ISIM application, the UE shall derive the home network domain name from the IMSI as described in the following steps:

1\. Take the first 5 or 6 digits, depending on whether a 2 or 3 digit MNC is used (see TS 31.102 \[27\]) and separate them into MCC and MNC; if the MNC is 2 digits then a zero shall be added at the beginning.

2\. Use the MCC and MNC derived in step 1 to create the "mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org" domain name.

3\. Add the label "ims." to the beginning of the domain.

An example of a home network domain name is:

IMSI in use: 234150999999999;

where:

\- MCC = 234;

\- MNC = 15; and

\- MSIN = 0999999999,

which gives the home network domain name: ims.mnc015.mcc234.3gppnetwork.org.

The Home Network Domain for a Stand-alone Non-Public Network (SNPN) subscriber with an IMSI-based SUPI type, shall use the IMSI-based derivation described above, and append nid\<NID\> of the SNPN, between the "ims." and "mnc\<MNC\>" labels.

NOTE: The UE takes the NID from "list of subscriber data" as specified in TS 23.122 \[139\], from the entry selected by the UE.

An example of a home SNPN network domain name using the above IMSI and NID 000007ed9d5 is ims.nid000007ed9d5.mnc015.mcc234.3gppnetwork.org.

The Home Network Domain for a Stand-alone Non-Public Network (SNPN) subscriber identified by a SUPI containing a network-specific identifier that takes the form of an NAI consists of the string "ims." appended with the realm part of the NAI.

For 3GPP2 systems, if there is no IMC present, the UE shall derive the home network domain name as described in Annex C of 3GPP2 X.S0013-004 \[67\].

## 13.3 Private User Identity

The private user identity shall take the form of an NAI, and shall have the form username@realm as specified in clause 2.1 of IETF RFC 4282 \[53\].

NOTE 1: It is possible for a representation of the IMSI to be contained within the NAI for the private identity.

For 3GPP systems, the private user identity used for the user shall be as specified in clause 4.2 of TS 24.229 \[81\] and in TS 23.228 \[24\] Annex E.3.1. If the private user identity is not known, the private user identity shall be derived from the IMSI.

The following steps show how to build the private user identity out of the IMSI:

1\. Use the whole string of digits as the username part of the private user identity; and

2\. convert the leading digits of the IMSI, i.e. MNC and MCC, into a domain name, as described in clause 13.2.

The result will be a private user identity of the form "\<IMSI\>@ims.mnc\<MNC\>.mcc\<MCC\>.3gppnetwork.org". For example: If the IMSI is 234150999999999 (MCC = 234, MNC = 15), the private user identity then takes the form "234150999999999@ims.mnc015.mcc234.3gppnetwork.org"..

The private user identity for a Stand-alone Non-Public Network (SNPN) subscriber with an IMSI-based SUPI type, shall use the IMSI-based derivation described above, and append nid\<NID\> of the SNPN, between the "ims." and "mnc\<MNC\>" labels.

NOTE 2: The UE takes the NID from "list of subscriber data" as specified in TS 23.122 \[139\], from the entry selected by the UE.

For an SNPN, if IMSI is as above, and for a NID of 000007ed9d5, the private user identity takes the form "234150999999999@ims.nid000007ed9d5.mnc015.mcc234.3gppnetwork.org".

The Private User Identity for a Stand-alone Non-Public Network (SNPN) subscriber identified by a SUPI containing a network-specific identifier that takes the form of an NAI is obtained as follows: the username is the same as the username in the NAI, and the realm portion is identical with the Home domain defined in clause 13.2.

For 3GPP2 systems, if there is no IMC present, the UE shall derive the private user identity as described in Annex C of 3GPP2 X.S0013-004 \[67\].

## 13.4 Public User Identity

A Public User Identity is any identity used by a user within the IMS subsystem for requesting communication to another user.

The Public User Identity shall take the form of either a SIP URI (see IETF RFCIETF RFC 3261 \[26\]) or a Tel URI (see IETF RFC 3966 \[45\]).

The 3GPP specifications describing the interfaces over which Public User Identities are transferred specify the allowed Public User Identity formats, in particular TS 24.229 \[81\] for SIP signalling interfaces, TS 29.229 \[95\] for Cx and Dx interfaces, TS 29.329 \[96\] for Sh interface, TS 29.165 \[97\] for II-NNI interface.

In the case the user identity is a telephone number, it shall be represented either by a Tel URI or by a SIP URI that includes a "user=phone" URI parameter and a "userinfo" part that shall follow the same format as the Tel URI.

According to TS 24.229 \[81\], the UE can use either:

\- a global number as defined in IETF RFC 3966 \[45\] and following E.164 format, as defined by ITU-T Recommendation E.164 \[10\] or

\- a local number, that shall include a "phone-context" parameter that identifies the scope of its validity, as per IETF RFC 3966 \[45\].

According to TS 29.165 \[97\] a global number as defined in IETF RFC 3966 \[45\] shall be used in a tel-URI or in the user portion of a SIP URI with the user=phone parameter when conveyed via a non-roaming II-NNI except when agreement exists between the operators to also allow other kinds of numbers.

According to TS 29.229 \[95\] and TS 29.329 \[96\] the canonical forms of SIP URI and Tel URI shall be used over the corresponding Diameter interfaces.

The canonical form of a SIP URI for a Public User Identity shall take the form "sip:username@domain" as specified in IETF RFC 3261 \[26\], clause 10.3. SIP URI comparisons shall be performed as defined in IETF RFC 3261 \[26\], clause 19.1.4.

The canonical form of a Tel URI for a Public User Identity shall take the form "tel:+\<CC\>\<NDC\>\<SN\>" (max number of digits is 15), that represents an E.164 number and shall contain a global number without parameters and visual separators (see IETF RFC 3966\[45\], clause 3). Tel URI comparisons shall be performed as defined in IETF RFC 3966\[45\], clause 4.

Public User Identities are stored in the HSS either as a distinct Public User Identity or as a Wildcarded Public User Identity. A distinct Public User Identity contains the Public User Identity that is used in routing and it is explicitly provisioned in the HSS.

## 13.4A Wildcarded Public User Identity

Public User Identities may be stored in the HSS as Wildcarded Public User Identities. A Wildcarded Public User Identity represents a collection of Public User Identities that share the same service profile and are included in the same implicit registration set. Wildcarded Public User Identities enable optimisation of the operation and maintenance of the nodes for the case in which a large amount of users are registered together and handled in the same way by the network. The format of a Wildcarded Public User Identity is the same as for the Wildcarded PSI described in clause 13.5.

## 13.4B Temporary Public User Identity

For 3GPP systems, if there is no ISIM application to host the Public User Identity, a Temporary Public User Identity shall be derived, based on the IMSI. The Temporary Public User Identity shall be of the form as described in clause 13.4 and shall consist of the string "sip:" appended with a username and domain portion equal to the IMSI derived Private User Identity, as described in clause 13.2. An example using the same example IMSI from clause 13.2 can be found below:

EXAMPLE: "sip:234150999999999@ims.mnc015.mcc234.3gppnetwork.org".

The temporary public user identity for a Stand-alone Non-Public Network (SNPN) subscriber with an IMSI-based SUPI type, shall use the IMSI-based derivation described above, and append nid\<NID\> of the SNPN, in between the "ims." and "mnc\<MNC\>" labels.

NOTE: The UE takes the NID from "list of subscriber data" as specified in TS 23.122 \[139\], from the entry selected by the UE.

EXAMPLE for an SNPN: "sip:234150999999999@ims.nid000007ed9d5.mnc015.mcc234.3gppnetwork.org".

The temporary Public User Identity for a Stand-alone Non-Public Network (SNPN) subscriber identified by a SUPI containing a network-specific identifier that takes the form of an NAI consists of the string "sip:" appended with a username and realm portion equal to the NAI SUPI derived Private User Identity.

For 3GPP2 systems, if there is no IMC present, the UE shall derive the public user identity as described in Annex C of 3GPP2 X.S0013-004 \[67\].

## 13.5 Public Service Identity (PSI)

The public service identity shall take the form of either a SIP URI (see IETF RFCIETF RFC 3261 \[26\]) or a Tel URI (see IETF RFC 3966 \[45\]). A public service identity identifies a service, or a specific resource created for a service on an application server. The domain part is pre-defined by the IMS operators and the IMS system provides the flexibility to dynamically create the user part of the PSIs.

The PSIs are stored in the HSS either as a distinct PSI or as a wildcarded PSI. A distinct PSI contains the PSI that is used in routing , whilst a wildcarded PSI represents a collection of PSIs. Wildcarded PSIs enable optimisation of the operation and maintenance of the nodes. A wildcarded PSI consists of a delimited regular expression located either in the userinfo portion of the SIP URI or in the telephone-subscriber portion of the Tel URI. The regular expression in the wildcarded PSI shall take the form of Extended Regular Expressions (ERE) as defined in chapter 9 in IEEE 1003.1-2004 Part 1 \[60\]. The delimiter shall be the exclamation mark character ("!"). If more than two exclamation mark characters are present in the userinfo portion or telephone-subscriber portion of a wildcarded PSI then the outside pair of exclamation mark characters is regarded as the pair of delimiters (i.e. no exclamation mark characters are allowed to be present in the fixed parts of the userinfo portion or telephone-subscriber portion).

When stored in the HSS, the wildcarded PSI shall include the delimiter character to indicate the extent of the part of the PSI that is wildcarded. It is used to separate the regular expression from the fixed part of the wildcarded PSI.

Example: The following PSI could be stored in the HSS - "sip:chatlist!.\*!@example.com".

When used on an interface, the exclamation mark characters within a PSI shall not be interpreted as delimiter..

Example: The following PSIs communicated in interface messages to the HSS will match to the wildcarded PSI of "sip:chatlist!.\*!@example.com" stored in the HSS:

sip:chatlist1@example.com

sip:chatlist2@example.com

sip:chatlist42@example.com

sip:chatlistAbC@example.com

sip:chatlist!1@example.com

Note that sip:chatlist1@example.com and sip:chatlist!1@example.com are regarded different specific PSIs, both matching the wildcarded PSI sip:chatlist!.\*!@example.com.

When used by an application server to identify a specific resource (e.g. a chat session) over Inter Operator Network to Network Interface (II-NNI), the PSI should be a SIP URI without including a port number.

NOTE: Based on local configuration policy, a PSI can be routed over Inter Operator Network to Network Interface (II-NNI). Details of this routing are operator specific and out of scope of this specification.

## 13.5A Private Service Identity

The Private Service Identity is applicable to a PSI user and is similar to a Private User Identity in the form of a Network Access Identifier (NAI), which is defined in IETF RFC 4282 \[53\]. The Private Service Identity is operator defined and although not operationally used for registration, authorisation and authentication in the same way as Private User Identity, it enables Public Service Identities to be associated to a Private Service Identity which is required for compatibility with the Cx procedures.

## 13.6 Anonymous User Identity

The Anonymous User Identity shall take the form of a SIP URI (see IETF RFC 3261 \[26\]). A SIP URI for an Anonymous User Identity shall take the form "sip:user@domain". The user part shall be the string "anonymous" and the domain part shall be the string "anonymous.invalid". The full SIP URI for Anonymous User Identity is thus:

"sip:anonymous@anonymous.invalid"

For more information on the Anonymous User Identity and when it is used, see TS 29.163 \[63\].

## 13.7 Unavailable User Identity

The Unavailable User Identity shall take the form of a SIP URI (see IETF RFC 3261 \[26\]). A SIP URI for an Unavailable User Identity shall take the form "sip:user@domain". The user part shall be the string "unavailable" and the domain part shall be the string "unknown.invalid". The full SIP URI for Unavailable User Identity is thus:

"sip:unavailable@unknown.invalid"

For more information on the Unavailable User Identity and when it is used, see TS 29.163 \[63\].

## 13.8 Instance-ID

An instance-id is a SIP Contact header parameter that uniquely identifies the SIP UA performing a registration.

When an IMEI is available, the instance-id shall take the form of a IMEI URN (see RFC 7254 \[79\]). The format of the instance-id shall take the form "urn:gsma:imei:\<imeival\>" where by the imeival shall contain the IMEI encoded as defined in RFC 7254 \[79\]. The optional \<sw-version-param\> and \<imei-version-param\> parameters shall not be included in the instance-id. RFC 7255 \[104\] specifies additional considerations for using the IMEI as an instance-id. An example of such an instance-id is as follows:

EXAMPLE: urn:gsma:imei:90420156-025763-0

If no IMEI is available, the instance-id shall take the form of a string representation of a UUID as a URN as defined in IETF RFC 4122 \[80\]. An example of such an instance-id is as follows:

EXAMPLE: urn:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6

For more information on the instance-id and when it is used, see TS 24.229 \[81\].

## 13.9 XCAP Root URI

### 13.9.1 XCAP Root URI on Ut interface

#### 13.9.1.1 General

XCAP Root URI is an HTTP URI that represents the XCAP Root. Although a valid URI, the XCAP Root URI does not correspond to an actual resource.

#### 13.9.1.2 Format of XCAP Root URI

The XCAP Root URI, as defined in IETF RFC 4825 \[94\], is an HTTP URI that should have the following format:

"http://xcap.\<domain\>"

in which "\<domain\>" identifies the domain hosting the XCAP server.

NOTE 1: The XCAP Root URI does not contain a port portion or an abs path portion of a standard HTTP URI.

If a preconfigured or provisioned XCAP Root URI is available then the UE shall use it. When a preconfigured or provisioned XCAP Root URI does not exist then the UE shall create the XCAP Root URI as follows:

\- The first label shall be "xcap".

\- The next label(s) shall identify the home network as follows:

1\. When the UE has an ISIM, the domain name from the IMPI shall be used (see TS 31.103 \[93\]) as follows:

a\. if the last two labels of the domain name from the IMPI are "3gppnetwork.org":

i\. the next labels shall be all labels of the domain name from the IMPI apart from the last two labels; and

ii\. the last three labels shall be "pub.3gppnetwork.org";

b\. if the last two labels of the domain name from the IMPI are other than the "3gppnetwork.org":

i\. the next labels shall be all labels of the domain name from the IMPI;

2\. When the UE has a USIM and does not have ISIM, the home network shall be "ims.mnc\<MNC\>.mcc\<MCC\>.pub.3gppnetwork.org" where \<MNC\> and \<MCC\> shall be derived from the components of the IMSI defined in clause 2.2. If there are only two significant digits in the MNC, one "0" digit shall be inserted at the left side to fill the 3 digits coding of MNC in the FQDN of XCAP Root URI.

As an example for the case when the UE has ISIM, where the IMPI is "user@operator.com", the overall XCAP Root URI used by the UE would be:

"http://xcap.operator.com".

As an example for the case when the UE has ISIM, where the IMPI is "234150999999999@ims.mnc015.mcc234.3gppnetwork.org", the overall XCAP Root URI used by the UE would be:

"xcap.ims.mnc015.mcc234.pub.3gppnetwork.org".

As an example for the case when the UE has USIM and does not have ISIM, where the MCC is 345 and the MNC is 12, the overall XCAP Root URI created and used by the UE would be:

"xcap.ims.mnc012.mcc345.pub.3gppnetwork.org"

## 13.10 Default Conference Factory URI for MMTel

The Default Conference Factory URI for MMTel shall take the form of a SIP URI (see IETF RFC 3261 \[26\]) with a host portion set to the home network domain name as described in clause 13.2 prefixed with "conf-factory.". The user portion shall be set to "mmtel".

Examples of the Default Conference Factory URI for MMTel can be found below:

EXAMPLE 1: "sip:mmtel@conf-factory.operator.com"

when the UE has a home network domain name of operator.com.

EXAMPLE 2: "sip:mmtel@conf-factory.ims.mnc015.mcc234.3gppnetwork.org"

for 3GPP systems, when the UE with no ISIM application has a home network domain name of ims.mnc015.mcc234.3gppnetwork.org derived from the same example IMSI as described in clause 13.2.

## 13.11 Unknown User Identity

The Unknown User Identity shall take the form of a SIP URI (see IETF RFCIETF RFC 3261 \[26\]). A SIP URI for an Unknown User Identity shall take the form "sip:user@domain". The user part shall be the string "unknown" and the domain part shall be the string "unknown.invalid". The full SIP URI for Unknown User Identity is thus:

"sip:unknown@unknown.invalid"

For more information on the Unknown User Identity and when it is used, see TS 29.163 \[63\], clauses 7.4.6 and 7.5.4.

## 13.12 Default WWSF URI

### 13.12.1 General

Default WWSF URI is an HTTP URI that represents the WebRTC Web Server Function (WWSF) defined in TS 23.228 \[24\].

### 13.12.2 Format of the Default WWSF URI

The Default WWSF URI is an HTTP URI that should have the following format:

"http://wwsf.\<domain\>"

in which "\<domain\>" identifies the domain hosting the WWSF.

If a preconfigured or provisioned WWSF URI is available then the UE shall use it. When a preconfigured or provisioned WWSF URI does not exist then the UE shall create the Default WWSF URI as follows:

\- The first label shall be "wwsf".

\- The next label(s) shall identify the home network as follows:

1\. When the UE has an ISIM, the domain name from the IMPI shall be used (see TS 31.103 \[93\]) as follows:

a\. if the last two labels of the domain name from the IMPI are "3gppnetwork.org":

i\. the next labels shall be all labels of the domain name from the IMPI apart from the last two labels; and

ii\. the last three labels shall be "pub.3gppnetwork.org";

b\. if the last two labels of the domain name from the IMPI are other than the "3gppnetwork.org":

i\. the next labels shall be all labels of the domain name from the IMPI;

2\. When the UE has a USIM and does not have an ISIM, the home network shall be "ims.mnc\<MNC\>.mcc\<MCC\>.pub.3gppnetwork.org" where \<MNC\> and \<MCC\> shall be derived from the components of the IMSI defined in clause 2.2. If there are only two significant digits in the MNC, one "0" digit shall be inserted at the left side to fill the 3 digits coding of MNC in the FQDN of WWSF URI.

As an example for the case when the UE has the ISIM, where the IMPI is "user@operator.com", the Default WWSF URI used by the UE would be:

EXAMPLE 1: "http://wwsf.operator.com".

As an example for the case when the UE has the ISIM, where the IMPI is "234150999999999@ims.mnc015.mcc234.3gppnetwork.org", the Default WWSF URI used by the UE would be:

EXAMPLE 2: "http://wwsf.ims.mnc015.mcc234.pub.3gppnetwork.org".

As an example for the case when the UE has the USIM and does not have the ISIM, where the MCC is 345 and the MNC is 12, the Default WWSF URI created and used by the UE would be:

EXAMPLE 3: "http://wwsf.ims.mnc012.mcc345.pub.3gppnetwork.org".

## 13.13 IMEI based identity

The IMEI based identity shall take the form of a SIP URI (see IETF RFC 3261 \[26\]). The IMEI based identity is included in P-Preferred-Identity header field of SIP INVITE request by the UE and used in cases of unauthenticated emergency sessions as specified in clause 5.1.6.8.2 of TS 24.229 \[81\]. A SIP URI for an IMEI based identity shall take the form "sip:user@domain" where by the user part shall contain the IMEI. The IMEI shall be encoded according to ABNF of imeival as defined in IETF RFC 7254 \[79\]. The domain part shall contain the home network domain named derived as specified in clause 13.2.

An example for the case when the UE has a home network domain name of operator.com is:

EXAMPLE 1: "sip:90420156-025763-0@operator.com"

An example for 3GPP systems, when the UE with no ISIM application has a home network domain name of ims.mnc015.mcc234.3gppnetwork.org derived from the same example IMSI from clause 13.2 is:

EXAMPLE 2: "sip:90420156-025763-0@ims.mnc015.mcc234.3gppnetwork.org"
