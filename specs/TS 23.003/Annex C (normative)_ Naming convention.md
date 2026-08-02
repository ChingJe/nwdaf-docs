---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: Annex C
title: 'Annex C (normative): Naming convention'
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex C (normative): Naming convention

This normative annex defines a naming convention which will make it possible for DNS servers to translate logical names for GSNs and RAs to physical IP addresses. The use of logical names is optional, but if the option is used, it shall comply with the naming convention described in this annex. The fully qualified domain names used throughout this annex shall follow the general encoding rules specified in clause 19.4.2.1.

## C.1 Routing Area Identities

This clause describes a possible way to support inter-PLMN roaming.

When an MS roams between two SGSNs within the same PLMN, the new SGSN finds the address of the old SGSN from the identity of the old RA. Thus, each SGSN can determine the address of every other SGSN in the PLMN.

When an MS roams from an SGSN in one PLMN to an SGSN in another PLMN, the new SGSN may be unable to determine the address of the old SGSN. Instead, the SGSN transforms the old RA information to a logical name of the form:

racAAAA.lacBBBB.mncYYY.mccZZZ.gprs

A and B shall be Hex coded digits; Y and Z shall be encoded as single digits (in the range 0-9).

If there are less than 4 significant digits in AAAA or BBBB, one or more "0" digit(s) is/are inserted at the left side to fill the 4 digit coding. If there are only 2 significant digits in YYY, a "0" digit is inserted at the left side to fill the 3 digit coding.

As an example, the logical name for RAC 123A, LAC 234B, MCC 167 and MNC 92 will be coded in the DNS server as:  
*rac123A.lac234B.mnc092.mcc167.gprs*.

The SGSN may then acquire the IP address of the old SGSN from a DNS server, using the logical address. Introducing the DNS concept in GPRS enables operators to use logical names instead of IP addresses when referring to nodes (e.g. GSNs), thus providing flexibility and transparency in addressing. Each PLMN should include at least one DNS server (which may optionally be connected via the DNS service provided by the GSM Association). Note that these DNS servers are GPRS internal entities, unknown outside the GPRS system.

The above implies that at least MCC \|\| MNC \|\| LAC \|\| RAC (= RAI) is sent as the RA parameter over the radio interface when an MS roams to another RA.

If for any reason the new SGSN fails to obtain the address of the old SGSN, the new SGSN takes the same actions as when the corresponding event occurs within one PLMN.

Another way to support seamless inter-PLMN roaming is to store the SGSN IP addresses in the HLR and request them when necessary.

If Intra Domain Connection of RAN Nodes to Multiple CN Nodes (see TS 23.236 \[23\]) is applied then the Network Resource Identifier (NRI) identifies uniquely a given SGSN node out of all the SGSNs serving the same pool area.

If the new SGSN is not able to extract the NRI from the old P-TMSI, it shall retrieve the address of the default SGSN (see TS 23.236 \[23\]) serving the old RA, using the logical name described earlier in this clause. The default SGSN in the old RA relays the GTP signalling to the old SGSN identified by the NRI in the old P‑TMSI unless the default SGSN itself is the old SGSN.

If the new SGSN is able to extract the NRI from the old P-TMSI, then it shall attempt to derive the address of the old SGSN from the NRI and the old RAI. NRI-to-SGSN assignments may be either configured (by O&M) in the new SGSN, or retrieved from a DNS server. If a DNS server is used, it shall be queried using the following logical name, derived from the old RAI and NRI information:

nriCCCC.racDDDD.lacEEEE.mncYYY.mccZZZ.gprs

C, D and E shall be Hex coded digits, Y and Z shall be encoded as single digits (in the range 0-9). If there are less than 4 significant digits in CCCC, DDDD or EEEE, one or more "0" digit(s) is/are inserted at the left side to fill the 4 digit coding. If there are only 2 significant digits in YYY, a "0" digit is inserted at the left side to fill the 3 digits coding.

As an example, the logical name for NRI 3A, RAC 123A, LAC 234B, MCC 167 and MNC 92 will be coded in the DNS server as:  
*nri003A.rac123A.lac234B.mnc092.mcc167.gprs*.

If for any reason the new SGSN fails to obtain the address of the old SGSN using this method, then as a fallback method it shall retrieve the address of the default SGSN serving the old RA.

## C.2 GPRS Support Nodes

This clause defines a naming convention for GSNs.

It shall be possible to refer to a GSN by a logical name which shall then be translated into a physical IP address. This clause proposes a GSN naming convention which would make it possible for an internal GPRS DNS server to make the translation.

An example of how a logical name of an SGSN could appear is:  
*sgsnXXXX.mncYYY.mccZZZ.gprs*

X, shall be Hex coded digits, Y andZz shall be encoded as single digits (in the range 0-9)*.*

If there are less than 4 significant digits in XXXX one or more "0" digit(s) is/are inserted at the left side to fill the 4 digits coding. If there are only 2 significant digits in YYY, a "0" digit is inserted at the left side to fill the 3 digit coding.

As an example, the logical name for SGSN 1B34, MCC 167 and MNC 92 will be coded in the DNS server as:  
*sgsn1B34. mnc092.mcc167.gprs*

## C.3 Target ID

This clause describes a possible way to support SRNS relocation.

In UMTS, when SRNS relocation is executed, a target ID which consists of MCC, MNC and RNC ID is used as routeing information to route to the target RNC via the new SGSN. An old SGSN shall resolve a new SGSN IP address by a target ID to send the Forward Relocation Request message to the new SGSN.

It shall be possible to refer to a target ID by a logical name which shall be translated into an SGSN IP address to take into account inter-PLMN handover. The old SGSN transforms the target ID information into a logical name of the form:

rncXXXX.mncYYY.mccZZZ.gprs

X shall be Hex coded digits; Y and Z shall be encoded as single digits (in the range 0-9). If there are less than 4 significant digits in XXXX, one or more "0" digit(s) is/are inserted at the left side to fill the 4 digits coding. If there are only 2 significant digits in YYY, a "0" digit is inserted at the left side to fill the 3 digit coding. Then, for example, a DNS server is used to translate the logical name to an SGSN IP address.

As an example, the logical name for RNC 1B34, MCC 167 and MNC 92 will be coded in the DNS server as:  
*rnc1B34.mnc092.mcc167.gprs*
