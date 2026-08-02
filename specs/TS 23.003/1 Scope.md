---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: '1'
title: 1 Scope
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 1 Scope

The present document defines the principal purpose and use of different naming, numbering, addressing and identification resources (i.e. Identifiers (ID)) within the digital cellular telecommunications system and the 3GPP system.

IDs that are covered by this specification includes both public IDs, private IDs and IDs that are assigned to MSs/UEs. Many of the IDs are used temporary in the networks and are allocated and assigned by the operators and some other IDs are allocated and assigned on either global, regional and national level by an administrator. See ITU-T Recommendation E.101 \[122\].

NOTE: Allocation means the process of opening a numbering, naming or addressing resource in a plan for the purpose of its use by a telecommunication service under specified conditions. The allocation in itself does not yet give rights for any user, whether an operator, service provider, user or someone else, to use the resource. Assignment means authorization given to an applicant for the right of use of number, naming or addressing resources under specified conditions.

The present document defines:

0\) the principal purpose and use of International Mobile station Equipment Identities (IMEI) within the digital cellular telecommunications system and the 3GPP system

a\) an identification plan for public networks and subscriptions in the 3GPP systems;

b\) principles of assigning telephone numbers to MSs in the country of registration of the MS;

c\) principles of assigning Mobile Station (MS) roaming numbers to visiting MSs;

d\) an identification plan for location areas, routing areas, and base stations in the GSM system;

e\) an identification plan for MSCs, SGSNs, GGSNs, and location registers in the GSM/UMTS system;

f\) principles of assigning international mobile equipment identities;

g\) principles of assigning zones for regional subscription;

h\) an identification plan for groups of subscribers to the Voice Group Call Service (VGCS) and to the Voice Broadcast Service (VBS); and identification plan for voice group calls and voice broadcast calls; an identification plan for group call areas;

i\) principles for assigning Packet Data Protocol (PDP) addresses to mobile stations;

j\) an identification plan for point-to-multipoint data transmission groups;

k\) an identification plan for CN domain, RNC and service area in the UTRAN system.

l\) an identification plan for mobile subscribers in the WLAN system.

m\) addressing and identification for IMS Service Continuity

n\) an identification plan together with principles of assignment and mapping of identities for the Evolved Packet System; and

o\) addressing and identification for Proximity-based (ProSe) Services.

p\) an identification for Online Charging System (OCS).

q\) an identification plan together with principles of assignment and mapping of identities for the 5G System.

The present document specifies functions, procedures and information which apply to GERAN Iu mode. However, functionality related to GERAN Iu mode is neither maintained nor enhanced.

## 1.1 References

### 1.1.1 Normative references

The following documents contain provisions which, through reference in this text, constitute provisions of the present document.

\- References are either specific (identified by date of publication, edition number, version number, etc.) or non‑specific.

\- For a specific reference, subsequent revisions do not apply.

\- For a non-specific reference, the latest version applies. In the case of a reference to a 3GPP document (including a GSM document), a non-specific reference implicitly refers to the latest version of that document *in the same Release as the present document*.

\[1\] 3GPP TS 21.905: "Vocabulary for 3GPP Specifications ".

\[2\] 3GPP TS 23.008: "Organization of subscriber data".

\[3\] 3GPP TS 23.060: "General Packet Radio Service (GPRS); Service description; Stage 2"

\[4\] 3GPP TS 23.070: "Routeing of calls to/from Public Data Networks (PDN)".

\[5\] 3GPP TS 24.008: "Mobile Radio Interface Layer 3 specification; Core Network Protocols; Stage 3".

\[6\] 3GPP TS 29.060: "GPRS Tunnelling protocol (GTP) across the Gn and Gp interface".

\[7\] 3GPP TS 43.020: "Digital cellular telecommunications system (Phase 2+); Security related network functions".

\[8\] void

\[9\] 3GPP TS 51.011: " Specification of the Subscriber Identity Module - Mobile Equipment (SIM - ME) interface".

\[10\] ITU-T Recommendation E.164: "The international public telecommunication numbering plan".

\[11\] ITU-T Recommendation E.212: "The international identification plan for public networks and subscriptions ".

\[12\] ITU-T Recommendation E.213: "Telephone and ISDN numbering plan for land Mobile Stations in public land mobile networks (PLMN)".

\[13\] ITU-T Recommendation X.121: "International numbering plan for public data networks".

\[14\] IETF RFC 791: "Internet Protocol".

\[15\] IETF RFC 2373: "IP Version 6 Addressing Architecture".

\[16\] 3GPP TS 25.401: "UTRAN Overall Description".

\[17\] 3GPP TS 25.413: "UTRAN Iu Interface RANAP Signalling".

\[18\] IETF RFC 2181: "Clarifications to the DNS Specification".

\[19\] IETF RFC 1035: "Domain Names - Implementation and Specification".

\[20\] IETF RFC 1123: "Requirements for Internet Hosts -- Application and Support".

\[21\] IETF RFC 2462: "IPv6 Stateless Address Autoconfiguration".

\[22\] IETF RFC 3041: "Privacy Extensions for Stateless Address Autoconfiguration in IPv6".

\[23\] 3GPP TS 23.236: "Intra Domain Connection of RAN Nodes to Multiple CN Nodes".

\[24\] 3GPP TS 23.228: "IP Multimedia (IM) Subsystem – Stage 2"

\[25\] Void

\[26\] IETF RFC 3261: "SIP: Session Initiation Protocol"

\[27\] 3GPP TS 31.102: "Characteristics of the USIM Application."

\[28\] Void

\[29\] 3GPP TS 44.118: "Radio Resource Control (RRC) Protocol, Iu Mode".

\[30\] Void

\[31\] 3GPP TS 29.002: "Mobile Application Part (MAP) specification"

\[32\] 3GPP TS 22.016: "International Mobile Equipment Identities (IMEI)"

\[33\] Void

\[34\] Void

\[35\] 3GPP TS 45.056: "CTS-FP Radio Sub-system"

\[36\] 3GPP TS 42.009: "Security aspects"

\[37\] 3GPP TS 25.423: "UTRAN Iur interface RNSAP signalling"

\[38\] 3GPP TS 25.419: "UTRAN Iu-BC interface: Service Area Broadcast Protocol (SABP)"

\[39\] 3GPP TS 25.410: "UTRAN Iu Interface: General Aspects and Principles"

\[40\] ISO/IEC 7812: "Identification cards - Numbering system and registration procedure for issuer identifiers"

\[41\] Void

\[42\] 3GPP TS 33.102 "3G security; Security architecture"

\[43\] 3GPP TS 43.130: "Iur‑g interface; Stage 2"

\[45\] IETF RFC 3966: "The tel URI for Telephone Numbers"

\[46\] 3GPP TS 44.068: "Group Call Control (GCC) protocol".

\[47\] 3GPP TS 44.069: "Broadcast Call Control (BCC) Protocol ".

\[48\] 3GPP TS 24.234 Release 12: "3GPP System to WLAN Interworking; UE to Network protocols; Stage 3".

\[49\] Void

\[50\] IETF RFC 4187: "EAP AKA Authentication".

\[51\] IETF RFC 4186: "EAP SIM Authentication".

\[52\] 3GPP TS 23.246: "Multimedia Broadcast/Multicast Service (MBMS); Architecture and functional description"

\[53\] IETF RFC 4282: "The Network Access Identifier".

\[54\] IETF RFC 2279: "UTF-8, a transformation format of ISO 10646".

\[55\] 3GPP TS 33.234 Release 12: "Wireless Local Area Network (WLAN) interworking security".

\[56\] Void

\[58\] 3GPP TS 33.221 "Generic Authentication Architecture (GAA); Support for Subscriber Certificates".

\[60\] IEEE 1003.1-2004, Part 1: Base Definitions

\[61\] 3GPP TS 43.318: "Generic Access to the A/Gb interface; Stage 2"

\[62\] 3GPP TS 44.318: "Generic Access (GA) to the A/Gb interface; Mobile GA interface layer 3 specification"

\[63\] 3GPP TS 29.163: "Interworking between the IP Multimedia (IM) Core Network (CN) subsystem and Circuit Switched (CS) networks"

\[64\] IETF RFC 2606: "Reserved Top Level DNS Names"

\[65\] Void

\[66\] 3GPP TS 51.011 Release 4: "Specification of the Subscriber Identity Module - Mobile Equipment (SIM - ME) interface"

\[67\] 3GPP2 X.S0013-004: "IP Multimedia Call Control Protocol based on SIP and SDP; Stage 3"

\[68\] 3GPP TS 23.402: "Architecture Enhancements for non-3GPP accesses"

\[69\] 3GPP TS 33.402: "3GPP System Architecture Evolution: Security Aspects of non-3GPP accesses"

\[70\] 3GPP TS 23.292: "IP Multimedia Subsystem (IMS) Centralized Services; Stage 2"

\[71\] 3GPP TS 23.237: "IP Multimedia Subsystem (IMS) Service Continuity"

\[72\] 3GPP TS 23.401: "General Packet Radio Service (GPRS) enhancements for Evolved Universal Terrestrial Radio Access Network (E-UTRAN) access"

\[73\] 3GPP TS 29.303: "Domain Name System Procedures; Stage 3"

\[74\] IETF RFC 3958: "Domain-Based Application Service Location Using SRV RRs and the Dynamic Delegation Discovery Service (DDDS)"

\[75\] Void

\[76\] 3GPP TS 23.237: "Mobility between 3GPP-Wireless Local Area Network (WLAN) interworking and 3GPP systems"

\[77\] 3GPP TS 24.302: "Access to the 3GPP Evolved Packet Core (EPC) via non-3GPP access networks; Stage 3"

\[78\] 3GPP TS 29.273: "Evolved Packet System; 3GPP EPS AAA Interfaces"

\[79\] IETF RFC 7254: "A Uniform Resource Name Namespace for the Global System for Mobile Communications Association (GSMA) and the International Mobile station Equipment Identity (IMEI)".

\[80\] IETF RFC 4122: "A Universally Unique IDentifier (UUID) URN Namespace".

\[81\] 3GPP TS 24.229: "IP multimedia call control protocol based on Session Initiation Protocol (SIP) and Session Description Protocol (SDP); Stage 3".

\[82\] IETF RFC5448: "Improved Extensible Authentication Protocol Method for 3rd Generation Authentication and Key Agreement (EAP-AKA') "

\[83\] 3GPP TS 22.011: "Service accessibility".

\[84\] 3GPP TS 36.413: "Evolved Universal Terrestrial Radio Access Network (E-UTRAN) ; S1 Application Protocol (S1AP)".

\[85\] Guidelines for use of a 48-bit Extended Unique Identifier (EUI-48™), http://standards.ieee.org/regauth/oui/tutorials/EUI48.html

\[86\] GUIDELINES FOR 64-BIT GLOBAL IDENTIFIER (EUI-64) REGISTRATION AUTHORITY, <http://standards.ieee.org/regauth/oui/tutorials/EUI64.html>

\[87\] The Broadband Forum TR-069: "CPE WAN Management Protocol v1.1", Issue 1 Amendment 2, December 2007

\[88\] 3GPP TS 29.274: "Evolved General Packet Radio Service (GPRS) Tunnelling Protocol for Control plane (GTPv2-C); Stage 3".

\[89\] 3GPP TS 33.401: "3GPP System Architecture Evolution: Security Architecture".

\[90\] 3GPP TS 24.301: "Non-Access-Stratum (NAS) protocol for Evolved Packet System (EPS); Stage 3".

\[91\] 3GPP TS 36.300: " Evolved Universal Terrestrial Radio Access (E-UTRA) and Evolved Universal Terrestrial Radio Access Network (E-UTRAN); Overall description; Stage 2".

\[92\] 3GPP TS 23.216: "Single Radio Voice Call Continuity (SRVCC)".

\[93\] 3GPP TS 31.103: "IP Multimedia Services Identity Module (ISIM) application".

\[94\] IETF RFC 4825: "The Extensible Markup Language (XML) Configuration Access Protocol (XCAP)".

\[95\] 3GPP TS 29.229: " Cx and Dx interfaces based on the Diameter protocol; Protocol details".

\[96\] 3GPP TS 29.329: " Sh Interface based on the Diameter protocol; Protocol details".

\[97\] 3GPP TS 29.165: "Inter-IMS Network to Network Interface (NNI); Stage 3".

\[98\] 3GPP TS 23.682: "Architecture Enhancements to facilitate communications with Packet Data Networks and Applications".

\[99\] 3GPP TS 44.018: "Mobile radio interface layer 3 specification; Radio Resource Control (RRC) protocol".

\[100\] 3GPP TS 44.060: "General Packet Radio Service (GPRS); Mobile Station (MS) – Base Station System (BSS) interface; Radio Link Control / Medium Access Control (RLC/MAC) protocol".

\[101\] 3GPP TS 23.251: "Network Sharing; Architecture and functional description".

\[102\] 3GPP TS 32.508: "Procedure flows for multi-vendor plug-and-play eNB connection to the network".

\[103\] 3GPP TS 23.303: "Proximity-based services (ProSe)".

\[104\] IETF RFC 7255: "Using the International Mobile station Equipment Identity (IMEI) Uniform Resource Name (URN) as an Instance ID".

\[105\] 3GPP TS 26.346: "Multimedia Broadcast/Multicast Service (MBMS); Protocols and codecs".

\[106\] 3GPP TS 29.212: "Policy and Charging Control (PCC); Reference points".

\[107\] 3GPP TS 23.203: "Policy and charging control architecture".

\[108\] 3GPP TS 29.272: "Evolved Packet System (EPS); Mobility Management Entity (MME) and Serving GPRS Support Node (SGSN) related interfaces based on Diameter protocol".

\[110\] Void.

\[111\] 3GPP TS 24.379: "Mission Critical Push To Talk (MCPTT) call control Protocol specification".

\[112\] 3GPP TS 43.064: "General Packet Radio Service (GPRS); Overall description of the GPRS Radio Interface; Stage 2".

\[113\] IETF RFC 6696: "EAP Extensions for the EAP Re-authentication Protocol (ERP)".

\[114\] 3GPP TS 23.280: "Common functional architecture to support mission critical services".

\[115\] 3GPP TS 24.281: "Mission Critical Video (MCVideo) signalling control; Protocol specification".

\[116\] 3GPP TS 24.282: "Mission Critical Data (MCData) signalling control; Protocol specification".

\[117\] 3GPP TS 23.285: "Architecture enhancements for V2X services".

\[118\] 3GPP TS 24.116: "Stage 3 aspects of system architecture enhancements for TV services".

\[119\] 3GPP TS 23.501: "System Architecture for the 5G System; Stage 2".

\[120\] 3GPP TS 23.502: "Procedures for the 5G System; Stage 2".

\[121\] 3GPP TS 23.503: "Policy and Charging Control Framework for the 5G System; Stage 2".

\[122\] ITU-T Recommendation E.101: "Definitions of terms used for identifiers (names, numbers, addresses and other identifiers) for public telecommunication services and networks in the E-series Recommendations".

\[123\] 3GPP TS 38.413: "NG Radio Access Network (NG-RAN); NG Application Protocol (NGAP)".

\[124\] 3GPP TS 33.501: "Security architecture and procedures for 5G system".

\[125\] 3GPP TS 24.501: "Non-Access-Stratum (NAS) protocol for 5G System (5GS); stage 3".

\[126\] IETF RFC 7542: "The Network Access Identifier".

\[127\] IETF RFC 2818: "HTTP over TLS".

\[128\] 3GPP TS 29.501: "5G System; Principles and Guidelines for Services Definition; Stage 3".

\[129\] 3GPP TS 29.571: "5G System; Common Data Types for Service Based Interfaces; Stage 3".

\[130\] 3GPP TS 29.510: "5G System; Network Function Repository Services; Stage 3".

\[131\] 3GPP TS 23.316: "Wireless and wireline convergence access support for the 5G System (5GS); Stage 2".

\[132\] IETF RFC 7042: "IANA Considerations and IETF Protocol and Documentation Usage for IEEE 802 Parameters".

\[133\] BBF WT-470: "5G FMC Architecture".

\[134\] CableLabs WR-TR-5WWC-ARCH: "5G Wireless Wireline Converged Core Architecture".

\[135\] CableLabs DOCSIS MULPI: "Data-Over-Cable Service Interface Specifications DOCSIS 3.1, MAC and Upper Layer Protocols Interface Specification".

\[136\] IEEE "Guidelines for Use of Extended Unique Identifier (EUI), Organizationally Unique Identifier (OUI), and Company ID (CID)", <https://standards.ieee.org/content/dam/ieee-standards/standards/web/documents/tutorials/eui.pdf>

\[137\] 3GPP TS 36.331: "Evolved Universal Terrestrial Radio Access (E-UTRA); Radio Resource Control (RRC); Protocol specification"

\[138\] 3GPP TS 38.331: "NR; Radio Resource Control (RRC); Protocol Specification".

\[139\] 3GPP TS 23.122: "Non-Access-Stratum (NAS) functions related to Mobile Station (MS) in idle mode".

\[140\] 3GPP TS 23.247: "Architectural enhancements for 5G multicast-broadcast services".

\[141\] 3GPP TS 38.300: "NR; NR and NG-RAN Overall Description; Stage 2".

\[142\] 3GPP TS 33.503: "Security Aspects of Proximity based Services (ProSe) in the 5G System (5GS)".

\[143\] 3GPP TS 23.304: "Proximity based Services (ProSe) in the 5G System (5GS); Stage 2".

\[144\] 3GPP TS 24.526: "UE policies for 5G System (5GS); Stage 3".

\[145\] 3GPP TS 24.368: "Non-Access Stratum (NAS) configuration Management Object (MO)".

\[146\] 3GPP TS 23.273: "5G System (5GS); Location Services (LCS); Stage 2".

\[147\] 3GPP TS 24.572: "5G System (5GS); User Plane Location Services (LCS) Protocols And Procedures; Stage 3".

\[148\] 3GPP TS 29.518: "5G System; Access and Mobility Management Services; Stage 3".

\[149\] 3GPP TS 29.572: "5G System; Location Management Services; Stage 3".

\[150\] IETF RFC 9298: "Proxying UDP in HTTP".

\[151\] 3GPP TS 24.193: "5G System; Access Traffic Steering, Switching and Splitting (ATSSS); Stage 3".

### 1.1.2 Informative references

\[44\] Void

\[57\] GSMA PRD IR.34 "Inter‑PLMN Backbone Guidelines"

\[59\] Void

\[109\] GSMA TS.06 "IMEI Allocation and Approval Process" http://www.gsma.com/newsroom/gsmadocuments/

## 1.2 Abbreviations

For the purposes of the present document, the abbreviations defined in TS 21.905 \[1\] and the following apply. An abbreviation defined in the present document takes precedence over the definition of the same abbreviation, if any, in TR 21.905 \[1\].

5G-BRG 5G Broadband Residential Gateway

5G-CRG 5G Cable Residential Gateway

5G-GUTI 5G Globally Unique Temporary Identifier

5G NSWO 5G Non-Seamless WLAN offload

5G-RG 5G Residential Gateway

5GS 5G System

5G-S-TMSI 5G S-Temporary Mobile Subscription Identifier

AMF Access and Mobility Management Function

CP-PRUK Control Plane Prose Remote User Key

EPS Evolved Packet System

ER EAP Re-authentication

ERP EAP Re-authentication Protocol

FN-BRG Fixed Network Broadband Residential Gateway

FN-CRG Fixed Network Cable RGGUAMI Globally Unique AMF Identifier

FN-RG Fixed Network RG

GUTI Globally Unique Temporary UE Identity

GCI Global Cable Identifier

GLI Global Line Identifier

GUTI Globally Unique Temporary UE Identity

HFC Hybrid Fiber Coax

HRNN Human Readable Network Name

ICS IMS Centralized Services

MPS Multimedia Priority Service

MTC Machine Type Communication

N5CW Non 5G Capable over WLAN

NCGI NR Cell Global Identity

NCI NR Cell Identity

NSI Network Specific Identifier

OCS Online Charging System

PEI Permanent Equipment Identifier

RACS Radio Capability Signalling Optimisation

RG Residential Gateway

SNPN Stand-alone Non-Public Network

SUCI Subscription Concealed Identifier

SUPI Subscription Permanent Identifier

TWAP Trusted WLAN AAA Proxy

UP-PRUK User Plane ProSe Remote User Key

UUID Universally Unique Identifier

V2X Vehicle-to-Everything

W-5GAN Wireline 5G Access Network

W-5GCAN Wireline 5G Cable Access Network

W-5GBAN Wireline BBF Access NetworkWebRTC Web Real-Time Communication

WLAN Wireless Local Area Network

WWSF WebRTC Web Server Function

## 1.3 General comments to references

The identification plan for public networks and subscriptions defined below is that defined in ITU-T Recommendation E.212.

The ISDN numbering plan for MSs and the allocation of mobile station roaming numbers is that defined in ITU-T Recommendation E.213. Only one of the principles for allocating ISDN numbers is proposed for PLMNs. Only the method for allocating MS roaming numbers contained in the main text of ITU-T Recommendation E.213 is recommended for use in PLMNs. If there is any difference between the present document and the ITU-T Recommendations, the former shall prevail.

For terminology, see also ITU-T Recommendations E.101, E.164 and X.121.

## 1.4 Conventions on bit ordering

The following conventions hold for the coding of the different identities appearing in the present document and in other GSM Technical Specifications if not indicated otherwise:

\- the different parts of an identity are shown in the figures in order of significance;

\- the most significant part of an identity is on the left part of the figure and the least significant on the right.

When an identity appears in other Technical Specifications, the following conventions hold if not indicated otherwise:

\- digits are numbered by order of significance, with digit 1 being the most significant;

\- bits are numbered by order of significance, with the lowest bit number corresponding to the least significant bit.
