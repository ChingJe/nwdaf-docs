---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: contents
title: Contents
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Contents

1 Scope [12](#scope)

1.1 References [13](#references)

1.1.1 Normative references [13](#normative-references)

1.1.2 Informative references [18](#informative-references)

1.2 Abbreviations [18](#abbreviations)

1.3 General comments to references [19](#general-comments-to-references)

1.4 Conventions on bit ordering [19](#conventions-on-bit-ordering)

2 Identification of mobile subscribers [19](#identification-of-mobile-subscribers)

2.1 General [19](#general)

2.2 Composition of IMSI [20](#composition-of-imsi)

2.2A Subscription Permanent Identifier (SUPI) [20](#a-subscription-permanent-identifier-supi)

2.2B Subscription Concealed Identifier (SUCI) [21](#b-subscription-concealed-identifier-suci)

2.3 Allocation and assignment principles [24](#allocation-and-assignment-principles)

2.4 Structure of TMSI [24](#structure-of-tmsi)

2.5 Structure of LMSI [25](#structure-of-lmsi)

2.6 Structure of TLLI [25](#structure-of-tlli)

2.7 Structure of P-TMSI Signature [26](#structure-of-p-tmsi-signature)

2.8 Globally Unique Temporary UE Identity (GUTI) [26](#globally-unique-temporary-ue-identity-guti)

2.8.1 Introduction [26](#introduction)

2.8.2 Mapping between Temporary and Area Identities for the EUTRAN and the UTRAN/GERAN based systems [27](#mapping-between-temporary-and-area-identities-for-the-eutran-and-the-utrangeran-based-systems)

2.8.2.0 Introduction [27](#introduction-1)

2.8.2.1 Mapping from GUTI to RAI, P-TMSI and P-TMSI signature [27](#mapping-from-guti-to-rai-p-tmsi-and-p-tmsi-signature)

2.8.2.1.1 Introduction [27](#introduction-2)

2.8.2.1.2 Mapping in the UE [27](#mapping-in-the-ue)

2.8.2.1.3 Mapping in the old MME [28](#mapping-in-the-old-mme)

2.8.2.2 Mapping from RAI and P-TMSI to GUTI [28](#mapping-from-rai-and-p-tmsi-to-guti)

2.8.2.2.1 Introduction [28](#introduction-3)

2.8.2.2.2 Mapping in the UE [28](#mapping-in-the-ue-1)

2.8.2.2.3 Mapping in the new MME [29](#mapping-in-the-new-mme)

2.9 Structure of the S-Temporary Mobile Subscriber Identity (S-TMSI) [29](#structure-of-the-s-temporary-mobile-subscriber-identity-s-tmsi)

2.10 5G Globally Unique Temporary UE Identity (5G-GUTI) [29](#g-globally-unique-temporary-ue-identity-5g-guti)

2.10.1 Introduction [29](#introduction-4)

2.10.2 Mapping between Temporary Identities for the 5GS and the E-UTRAN [30](#mapping-between-temporary-identities-for-the-5gs-and-the-e-utran)

2.10.2.0 Introduction [30](#introduction-5)

2.10.2.1 Mapping from 5G-GUTI to GUTI [30](#mapping-from-5g-guti-to-guti)

2.10.2.1.1 Introduction [30](#introduction-6)

2.10.2.1.2 Mapping in the UE [30](#mapping-in-the-ue-2)

2.10.2.1.3 Mapping in the old AMF [31](#mapping-in-the-old-amf)

2.10.2.2 Mapping from GUTI to 5G-GUTI [31](#mapping-from-guti-to-5g-guti)

2.10.2.2.1 Introduction [31](#introduction-7)

2.10.2.2.2 Mapping in the UE [31](#mapping-in-the-ue-3)

2.10.2.2.3 Mapping in the new AMF [31](#mapping-in-the-new-amf)

2.11 Structure of the 5G-S-Temporary Mobile Subscriber Identity (5G-S-TMSI) [32](#structure-of-the-5g-s-temporary-mobile-subscriber-identity-5g-s-tmsi)

2.12 Structure of the Truncated 5G-S-Temporary Mobile Subscriber Identity (Truncated 5G-S-TMSI) [32](#structure-of-the-truncated-5g-s-temporary-mobile-subscriber-identity-truncated-5g-s-tmsi)

3 Numbering plan for mobile stations [32](#numbering-plan-for-mobile-stations)

3.1 General [32](#general-1)

3.2 Numbering plan requirements [33](#numbering-plan-requirements)

3.3 Structure of Mobile Subscriber ISDN number (MSISDN) [33](#structure-of-mobile-subscriber-isdn-number-msisdn)

3.4 Mobile Station Roaming Number (MSRN) for PSTN/ISDN routeing [34](#mobile-station-roaming-number-msrn-for-pstnisdn-routeing)

3.5 Structure of Mobile Station International Data Number [34](#structure-of-mobile-station-international-data-number)

3.6 Handover Number [34](#handover-number)

3.7 Structure of an IP v4 address [35](#structure-of-an-ip-v4-address)

3.8 Structure of an IP v6 address [35](#structure-of-an-ip-v6-address)

4 Identification of location areas and base stations [35](#identification-of-location-areas-and-base-stations)

4.1 Composition of the Location Area Identification (LAI) [35](#composition-of-the-location-area-identification-lai)

4.2 Composition of the Routing Area Identification (RAI) [35](#composition-of-the-routing-area-identification-rai)

4.3 Base station identification [36](#base-station-identification)

4.3.1 Cell Identity (CI) and Cell Global Identification (CGI) [36](#cell-identity-ci-and-cell-global-identification-cgi)

4.3.2 Base Station Identify Code (BSIC) [36](#base-station-identify-code-bsic)

4.4 Regional Subscription Zone Identity (RSZI) [37](#regional-subscription-zone-identity-rszi)

4.5 Location Number [38](#location-number)

4.6 Composition of the Service Area Identification (SAI) [38](#composition-of-the-service-area-identification-sai)

4.7 Closed Subscriber Group [38](#closed-subscriber-group)

4.8 HNB Name [38](#hnb-name)

4.9 CSG Type [38](#csg-type)

4.10 HNB Unique Identity [39](#hnb-unique-identity)

4.11 HRNN [39](#hrnn)

5 Identification of MSCs, GSNs, location registers and CSSs [39](#identification-of-mscs-gsns-location-registers-and-csss)

5.1 Identification for routeing purposes [39](#identification-for-routeing-purposes)

5.2 Identification of HLR for HLR restoration application [40](#identification-of-hlr-for-hlr-restoration-application)

5.3 Identification of the HSS for SMS [40](#identification-of-the-hss-for-sms)

6 International Mobile Station Equipment Identity, Software Version Number and Permanent Equipment Identifier [40](#international-mobile-station-equipment-identity-software-version-number-and-permanent-equipment-identifier)

6.1 General [40](#general-2)

6.2 Composition of IMEI and IMEISV [40](#composition-of-imei-and-imeisv)

6.2.1 Composition of IMEI [40](#composition-of-imei)

6.2.2 Composition of IMEISV [41](#composition-of-imeisv)

6.3 Allocation principles [42](#allocation-principles)

6.4 Permanent Equipment Identifier (PEI) [42](#permanent-equipment-identifier-pei)

7 Identification of Voice Group Call and Voice Broadcast Call Entities [42](#identification-of-voice-group-call-and-voice-broadcast-call-entities)

7.1 Group Identities [42](#group-identities)

7.2 Group Call Area Identification [43](#group-call-area-identification)

7.3 Voice Group Call and Voice Broadcast Call References [43](#voice-group-call-and-voice-broadcast-call-references)

8 SCCP subsystem numbers [43](#sccp-subsystem-numbers)

8.1 Globally standardized subsystem numbers used for GSM/UMTS [43](#globally-standardized-subsystem-numbers-used-for-gsmumts)

8.2 National network subsystem numbers used for GSM/UMTS [44](#national-network-subsystem-numbers-used-for-gsmumts)

9 Definition of Access Point Name [44](#definition-of-access-point-name)

9A Definition of Data Network Name [44](#a-definition-of-data-network-name)

9.0 General [45](#general-3)

9.1 Structure of APN [45](#structure-of-apn)

9.1.1 Format of APN Network Identifier [45](#format-of-apn-network-identifier)

9.1.2 Format of APN Operator Identifier [46](#format-of-apn-operator-identifier)

9.2 Definition of the Wild Card APN [46](#definition-of-the-wild-card-apn)

9.2.1 Coding of the Wild Card APN [46](#coding-of-the-wild-card-apn)

9.3 Definition of Emergency APN [47](#definition-of-emergency-apn)

10 Identification of the Cordless Telephony System entities [47](#identification-of-the-cordless-telephony-system-entities)

10.1 General description of CTS‑MS and CTS‑FP Identities [47](#general-description-of-ctsms-and-ctsfp-identities)

10.2 CTS Mobile Subscriber Identities [47](#cts-mobile-subscriber-identities)

10.2.1 General [47](#general-4)

10.2.2 Composition of the CTSMSI [47](#composition-of-the-ctsmsi)

10.2.3 Allocation principles [47](#allocation-principles-1)

10.2.4 CTSMSI hexadecimal representation [48](#ctsmsi-hexadecimal-representation)

10.3 Fixed Part Beacon Identity [48](#fixed-part-beacon-identity)

10.3.1 General [48](#general-5)

10.3.2 Composition of the FPBI [48](#composition-of-the-fpbi)

10.3.2.1 FPBI general structure [48](#fpbi-general-structure)

10.3.2.2 FPBI class A [49](#fpbi-class-a)

10.3.2.3 FPBI class B [49](#fpbi-class-b)

10.3.3 Allocation principles [49](#allocation-principles-2)

10.4 International Fixed Part Equipment Identity [50](#international-fixed-part-equipment-identity)

10.4.1 General [50](#general-6)

10.4.2 Composition of the IFPEI [50](#composition-of-the-ifpei)

10.4.3 Allocation and assignment principles [50](#allocation-and-assignment-principles-1)

10.5 International Fixed Part Subscription Identity [50](#international-fixed-part-subscription-identity)

10.5.1 General [50](#general-7)

10.5.2 Composition of the IFPSI [51](#composition-of-the-ifpsi)

10.5.3 Allocation and assignment principles [51](#allocation-and-assignment-principles-2)

11 Identification of Localised Service Area [51](#identification-of-localised-service-area)

12 Identification of PLMN, RNC, Service Area, CN domain and Shared Network Area [52](#identification-of-plmn-rnc-service-area-cn-domain-and-shared-network-area)

12.1 PLMN Identifier [52](#plmn-identifier)

12.2 CN Domain Identifier [52](#cn-domain-identifier)

12.3 CN Identifier [52](#cn-identifier)

12.4 RNC Identifier [53](#rnc-identifier)

12.5 Service Area Identifier [53](#service-area-identifier)

12.6 Shared Network Area Identifier [53](#shared-network-area-identifier)

12.7 Stand-Alone Non-Public Network Identifier [53](#stand-alone-non-public-network-identifier)

12.7.1 Network Identifier (NID) [53](#network-identifier-nid)

12.7.2 NID of assignment mode 0 [54](#nid-of-assignment-mode-0)

12.7.3 Group ID for Network Selection (GIN) [54](#group-id-for-network-selection-gin)

13 Numbering, addressing and identification within the IP multimedia core network subsystem [55](#numbering-addressing-and-identification-within-the-ip-multimedia-core-network-subsystem)

13.1 Introduction [55](#introduction-8)

13.2 Home network domain name [55](#home-network-domain-name)

13.3 Private User Identity [56](#private-user-identity)

13.4 Public User Identity [56](#public-user-identity)

13.4A Wildcarded Public User Identity [57](#a-wildcarded-public-user-identity)

13.4B Temporary Public User Identity [57](#b-temporary-public-user-identity)

13.5 Public Service Identity (PSI) [58](#public-service-identity-psi)

13.5A Private Service Identity [58](#a-private-service-identity)

13.6 Anonymous User Identity [59](#anonymous-user-identity)

13.7 Unavailable User Identity [59](#unavailable-user-identity)

13.8 Instance-ID [59](#instance-id)

13.9 XCAP Root URI [59](#xcap-root-uri)

13.9.1 XCAP Root URI on Ut interface [59](#xcap-root-uri-on-ut-interface)

13.9.1.1 General [59](#general-8)

13.9.1.2 Format of XCAP Root URI [59](#format-of-xcap-root-uri)

13.10 Default Conference Factory URI for MMTel [60](#default-conference-factory-uri-for-mmtel)

13.11 Unknown User Identity [60](#unknown-user-identity)

13.12 Default WWSF URI [61](#default-wwsf-uri)

13.12.1 General [61](#general-9)

13.12.2 Format of the Default WWSF URI [61](#format-of-the-default-wwsf-uri)

13.13 IMEI based identity [61](#imei-based-identity)

14 Numbering, addressing and identification for 3GPP System to WLAN Interworking [62](#numbering-addressing-and-identification-for-3gpp-system-to-wlan-interworking)

14.1 Introduction [62](#introduction-9)

14.2 Home network realm [62](#home-network-realm)

14.3 Root NAI [62](#root-nai)

14.4 Decorated NAI [63](#decorated-nai)

14.4A Fast Re‑authentication NAI [63](#a-fast-reauthentication-nai)

14.5 Temporary identities [64](#temporary-identities)

14.6 Alternative NAI [64](#alternative-nai)

14.7 W-APN [64](#w-apn)

14.7.1 Format of W-APN Network Identifier [65](#format-of-w-apn-network-identifier)

14.7.2 Format of W-APN Operator Identifier [65](#format-of-w-apn-operator-identifier)

14.7.3 Alternative Format of W-APN Operator Identifier [66](#alternative-format-of-w-apn-operator-identifier)

14.8 Emergency Realm and Emergency NAI for Emergency Cases [66](#emergency-realm-and-emergency-nai-for-emergency-cases)

15 Identification of Multimedia Broadcast/Multicast Service [67](#identification-of-multimedia-broadcastmulticast-service)

15.1 Introduction [67](#introduction-10)

15.2 Structure of TMGI [67](#structure-of-tmgi)

15.3 Structure of MBMS SAI [68](#structure-of-mbms-sai)

15.4 Home Network Realm [68](#home-network-realm-1)

15.5 Addressing and identification for Bootstrapping MBMS Service Announcement [69](#addressing-and-identification-for-bootstrapping-mbms-service-announcement)

16 Numbering, addressing and identification within the GAA subsystem [69](#numbering-addressing-and-identification-within-the-gaa-subsystem)

16.1 Introduction [69](#introduction-11)

16.2 BSF address [69](#bsf-address)

17 Numbering, addressing and identification within the Generic Access Network [70](#numbering-addressing-and-identification-within-the-generic-access-network)

17.1 Introduction [70](#introduction-12)

17.2 Network Access Identifiers [70](#network-access-identifiers)

17.2.1 Home network realm [70](#home-network-realm-2)

17.2.2 Full Authentication NAI [71](#full-authentication-nai)

17.2.3 Fast Re‑authentication NAI [71](#fast-reauthentication-nai)

17.3 Node Identifiers [72](#node-identifiers)

17.3.1 Home network domain name [72](#home-network-domain-name-1)

17.3.2 Provisioning GANC-SEGW identifier [72](#provisioning-ganc-segw-identifier)

17.3.3 Provisioning GANC identifier [73](#provisioning-ganc-identifier)

18 Addressing and Identification for IMS Service Continuity and Single-Radio Voice Call Continuity [73](#addressing-and-identification-for-ims-service-continuity-and-single-radio-voice-call-continuity)

18.1 Introduction [73](#introduction-13)

18.2 CS Domain Routeing Number (CSRN) [73](#cs-domain-routeing-number-csrn)

18.3 IP Multimedia Routeing Number (IMRN) [74](#ip-multimedia-routeing-number-imrn)

18.4 Session Transfer Number (STN) [74](#session-transfer-number-stn)

18.5 Session Transfer Identifier (STI) [74](#session-transfer-identifier-sti)

18.6 Session Transfer Number for Single Radio Voice Call Continuity (STN-SR) [74](#session-transfer-number-for-single-radio-voice-call-continuity-stn-sr)

18.7 Correlation MSISDN [74](#correlation-msisdn)

18.8 Transfer Identifier for CS to PS Single Radio Voice Call Continuity (STI-rSR) [74](#transfer-identifier-for-cs-to-ps-single-radio-voice-call-continuity-sti-rsr)

18.9 Additional MSISDN [74](#additional-msisdn)

19 Numbering, addressing and identification for the Evolved Packet Core (EPC) [75](#numbering-addressing-and-identification-for-the-evolved-packet-core-epc)

19.1 Introduction [75](#introduction-14)

19.2 Home Network Realm/Domain [75](#home-network-realmdomain)

19.3 3GPP access to non-3GPP access interworking [75](#gpp-access-to-non-3gpp-access-interworking)

19.3.1 Introduction [75](#introduction-15)

19.3.2 Root NAI [76](#root-nai-1)

19.3.3 Decorated NAI [76](#decorated-nai-1)

19.3.4 Fast Re‑authentication NAI [77](#fast-reauthentication-nai-1)

19.3.5 Pseudonym Identities [78](#pseudonym-identities)

19.3.6 Emergency NAI for Limited Service State [79](#emergency-nai-for-limited-service-state)

19.3.7 Alternative NAI [79](#alternative-nai-1)

19.3.8 Keyname NAI [79](#keyname-nai)

19.3.9 IMSI-based Emergency NAI [80](#imsi-based-emergency-nai)

19.3.10 High Priority Access NAI [80](#high-priority-access-nai)

19.4 Identifiers for Domain Name System procedures [80](#identifiers-for-domain-name-system-procedures)

19.4.1 Introduction [80](#introduction-16)

19.4.2 Fully Qualified Domain Names (FQDNs) [81](#fully-qualified-domain-names-fqdns)

19.4.2.1 General [81](#general-10)

19.4.2.2 Access Point Name FQDN (APN-FQDN) [81](#access-point-name-fqdn-apn-fqdn)

19.4.2.2.1 Structure [81](#structure)

19.4.2.2.2 Void [82](#void)

19.4.2.2.3 Void [82](#void-1)

19.4.2.2.4 Void [82](#void-2)

19.4.2.3 Tracking Area Identity (TAI) [82](#tracking-area-identity-tai)

19.4.2.4 Mobility Management Entity (MME) [82](#mobility-management-entity-mme)

19.4.2.5 Routing Area Identity (RAI) - EPC [83](#routing-area-identity-rai---epc)

19.4.2.6 Serving GPRS Support Node (SGSN) within SGSN pool [83](#serving-gprs-support-node-sgsn-within-sgsn-pool)

19.4.2.7 Target RNC-ID for U-TRAN [83](#target-rnc-id-for-u-tran)

19.4.2.8 DNS subdomain for operator usage in EPC [84](#dns-subdomain-for-operator-usage-in-epc)

19.4.2.9 ePDG FQDN and Visited Country FQDN for non-emergency bearer services [84](#epdg-fqdn-and-visited-country-fqdn-for-non-emergency-bearer-services)

19.4.2.9.1 General [84](#general-11)

19.4.2.9.2 Operator Identifier based ePDG FQDN [84](#operator-identifier-based-epdg-fqdn)

19.4.2.9.3 Tracking/Location Area Identity based ePDG FQDN [85](#trackinglocation-area-identity-based-epdg-fqdn)

19.4.2.9.4 Visited Country FQDN [86](#visited-country-fqdn)

19.4.2.9.5 Replacement field used in DNS-based Discovery of regulatory requirements [86](#replacement-field-used-in-dns-based-discovery-of-regulatory-requirements)

19.4.2.9A ePDG FQDN for emergency bearer services [87](#a-epdg-fqdn-for-emergency-bearer-services)

19.4.2.9A.1 General [87](#a.1-general)

19.4.2.9A.2 Operator Identifier based Emergency ePDG FQDN [87](#a.2-operator-identifier-based-emergency-epdg-fqdn)

19.4.2.9A.3 Tracking/Location Area Identity based Emergency ePDG FQDN [87](#a.3-trackinglocation-area-identity-based-emergency-epdg-fqdn)

19.4.2.9A.4 Visited Country Emergency FQDN [88](#a.4-visited-country-emergency-fqdn)

19.4.2.9A.5 Replacement field used in DNS-based Discovery of regulatory requirements for emergency services [88](#a.5-replacement-field-used-in-dns-based-discovery-of-regulatory-requirements-for-emergency-services)

19.4.2.9A.6 Country based Emergency Numbers FQDN [88](#a.6-country-based-emergency-numbers-fqdn)

19.4.2.9A.7 Replacement field used in DNS-based Discovery of Emergency Numbers [89](#a.7-replacement-field-used-in-dns-based-discovery-of-emergency-numbers)

19.4.2.10 Global eNodeB-ID for eNodeB [89](#global-enodeb-id-for-enodeb)

19.4.2.11 Local Home Network identifier [89](#local-home-network-identifier)

19.4.2.12 UCMF [90](#ucmf)

19.4.2.13 PGW Set FQDN [90](#pgw-set-fqdn)

19.4.3 Service and Protocol service names for 3GPP [90](#service-and-protocol-service-names-for-3gpp)

19.5 Access Network Identity [92](#access-network-identity)

19.6 E-UTRAN Cell Identity (ECI) and E-UTRAN Cell Global Identification (ECGI) [92](#e-utran-cell-identity-eci-and-e-utran-cell-global-identification-ecgi)

19.6A NR Cell Identity (NCI) and NR Cell Global Identity (NCGI) [92](#a-nr-cell-identity-nci-and-nr-cell-global-identity-ncgi)

19.7 Identifiers for communications with packet data networks and applications [93](#identifiers-for-communications-with-packet-data-networks-and-applications)

19.7.1 Introduction [93](#introduction-17)

19.7.2 External Identifier [93](#external-identifier)

19.7.3 External Group Identifier [93](#external-group-identifier)

19.8 TWAN Operator Name [94](#twan-operator-name)

19.9 IMSI-Group Identifier [94](#imsi-group-identifier)

19.10 Presence Reporting Area Identifier (PRA ID) [94](#presence-reporting-area-identifier-pra-id)

19.11 Dedicated Core Networks Identifier [95](#dedicated-core-networks-identifier)

20 Addressing and Identification for IMS Centralized Services [95](#addressing-and-identification-for-ims-centralized-services)

20.1 Introduction [95](#introduction-18)

20.2 UE based solution [95](#ue-based-solution)

20.3 Network based solution [95](#network-based-solution)

20.3.1 General [95](#general-12)

20.3.2 Home network domain name [96](#home-network-domain-name-2)

20.3.3 Private User Identity [96](#private-user-identity-1)

20.3.4 Public User Identity [96](#public-user-identity-1)

20.3.5 Conference Factory URI [97](#conference-factory-uri)

21 Addressing and Identification for Dual Stack Mobile IPv6 (DSMIPv6) [97](#addressing-and-identification-for-dual-stack-mobile-ipv6-dsmipv6)

21.1 Introduction [97](#introduction-19)

21.2 Home Agent – Access Point Name (HA-APN) [97](#home-agent-access-point-name-ha-apn)

21.2.1 General [97](#general-13)

21.2.2 Format of HA-APN Network Identifier [97](#format-of-ha-apn-network-identifier)

21.2.3 Format of HA-APN Operator Identifier [98](#format-of-ha-apn-operator-identifier)

22 Addressing and identification for ANDSF [98](#addressing-and-identification-for-andsf)

22.1 Introduction [98](#introduction-20)

22.2 ANDSF Server Name (ANDSF-SN) [98](#andsf-server-name-andsf-sn)

22.2.1 General [98](#general-14)

22.2.2 Format of ANDSF-SN [99](#format-of-andsf-sn)

23 Numbering, addressing and identification for the OAM System [99](#numbering-addressing-and-identification-for-the-oam-system)

23.1 Introduction [99](#introduction-21)

23.2 OAM System Realm/Domain [99](#oam-system-realmdomain)

23.3 Identifiers for Domain Name System procedures [100](#identifiers-for-domain-name-system-procedures-1)

23.3.1 Introduction [100](#introduction-22)

23.3.2 Fully Qualified Domain Names (FQDNs) [100](#fully-qualified-domain-names-fqdns-1)

23.3.2.1 General [100](#general-15)

23.3.2.2 Relay Node Vendor-Specific OAM System [100](#relay-node-vendor-specific-oam-system)

23.3.2.3 Multi-vendor eNodeB Plug-and Play Vendor-Specific OAM System [101](#multi-vendor-enodeb-plug-and-play-vendor-specific-oam-system)

23.3.2.3.1 General [101](#general-16)

23.3.2.3.2 Certification Authority server [101](#certification-authority-server)

23.3.2.3.3 Security Gateway [101](#security-gateway)

23.3.2.3.4 Element Manager [102](#element-manager)

24 Numbering, addressing and identification for Proximity-based Services (ProSe) [102](#numbering-addressing-and-identification-for-proximity-based-services-prose)

24.1 Introduction [102](#introduction-23)

24.2 ProSe Application ID [102](#prose-application-id)

24.2.1 General [102](#general-17)

24.2.2 Format of ProSe Application ID Name in ProSe Application ID [103](#format-of-prose-application-id-name-in-prose-application-id)

24.2.3 Format of PLMN ID in ProSe Application ID [103](#format-of-plmn-id-in-prose-application-id)

24.2.4 Usage of wild cards in place of PLMN ID in ProSe Application ID [103](#usage-of-wild-cards-in-place-of-plmn-id-in-prose-application-id)

24.2.5 Informative examples of ProSe Application ID [104](#informative-examples-of-prose-application-id)

24.3 ProSe Application Code [104](#prose-application-code)

24.3.1 General [104](#general-18)

24.3.2 Format of PLMN ID in ProSe Application Code [104](#format-of-plmn-id-in-prose-application-code)

24.3.3 Format of temporary identity in ProSe Application Code [105](#format-of-temporary-identity-in-prose-application-code)

24.3A ProSe Application Code Prefix [105](#a-prose-application-code-prefix)

24.3B ProSe Application Code Suffix [105](#b-prose-application-code-suffix)

24.4 EPC ProSe User ID [105](#epc-prose-user-id)

24.4.1 General [105](#general-19)

24.4.2 Format of EPC ProSe User ID [106](#format-of-epc-prose-user-id)

24.5 Home PLMN ProSe Function Address [106](#home-plmn-prose-function-address)

24.6 ProSe Restricted Code [106](#prose-restricted-code)

24.7 ProSe Restricted Code Prefix [106](#prose-restricted-code-prefix)

24.8 ProSe Restricted Code Suffix [106](#prose-restricted-code-suffix)

24.9 ProSe Query Code [107](#prose-query-code)

24.10 ProSe Response Code [107](#prose-response-code)

24.11 ProSe Discovery UE ID [107](#prose-discovery-ue-id)

24.11.1 General [107](#general-20)

24.11.2 Format of ProSe Discovery UE ID [107](#format-of-prose-discovery-ue-id)

24.12 ProSe UE ID [107](#prose-ue-id)

24.13 ProSe Relay UE ID [107](#prose-relay-ue-id)

24.14 User Info ID [108](#user-info-id)

24.15 Relay Service Code [108](#relay-service-code)

24.16 Discovery Group ID [108](#discovery-group-id)

24.17 Service ID [108](#service-id)

25 Identification of Online Charging System [108](#identification-of-online-charging-system)

25.1 Introduction [108](#introduction-24)

25.2 Home network domain name [108](#home-network-domain-name-3)

26 Numbering, addressing and identification for Mission Critical Services [109](#numbering-addressing-and-identification-for-mission-critical-services)

26.1 Introduction [109](#introduction-25)

26.2 Domain name for MC services confidentiality protection of MC services identities [109](#domain-name-for-mc-services-confidentiality-protection-of-mc-services-identities)

27 Numbering, addressing and identification for V2X [109](#numbering-addressing-and-identification-for-v2x)

27.1 Introduction [109](#introduction-26)

27.2 V2X Control Function FQDN [110](#v2x-control-function-fqdn)

27.2.1 General [110](#general-21)

27.2.2 Format of V2X Control Function FQDN [110](#format-of-v2x-control-function-fqdn)

28 Numbering, addressing and identification for 5G System (5GS) [110](#numbering-addressing-and-identification-for-5g-system-5gs)

28.1 Introduction [110](#introduction-27)

28.2 Home Network Domain [110](#home-network-domain)

28.3 Identifiers for Domain Name System procedures [111](#identifiers-for-domain-name-system-procedures-2)

28.3.1 Introduction [111](#introduction-28)

28.3.2 Fully Qualified Domain Names (FQDNs) [111](#fully-qualified-domain-names-fqdns-2)

28.3.2.1 General [111](#general-22)

28.3.2.2 N3IWF FQDN [111](#n3iwf-fqdn)

28.3.2.2.1 General [111](#general-23)

28.3.2.2.2 Operator Identifier based N3IWF FQDN [112](#operator-identifier-based-n3iwf-fqdn)

28.3.2.2.3 Tracking Area Identity based N3IWF FQDN [112](#tracking-area-identity-based-n3iwf-fqdn)

28.3.2.2.4 Visited Country FQDN for N3IWF [113](#visited-country-fqdn-for-n3iwf)

28.3.2.2.5 Replacement field used in DNS-based Discovery of regulatory requirements [114](#replacement-field-used-in-dns-based-discovery-of-regulatory-requirements-1)

28.3.2.2.6 FQDN for SNPN N3IWF [115](#fqdn-for-snpn-n3iwf)

28.3.2.2.7 Replacement field used in DNS-based Discovery of SNPN N3IWF for regulatory requirements [117](#replacement-field-used-in-dns-based-discovery-of-snpn-n3iwf-for-regulatory-requirements)

28.3.2.2.8 Prefixed Operator Identifier based N3IWF FQDN [118](#prefixed-operator-identifier-based-n3iwf-fqdn)

28.3.2.2.9 Prefixed Tracking Area Identity based N3IWF FQDN [118](#prefixed-tracking-area-identity-based-n3iwf-fqdn)

28.3.2.3 PLMN level and Home NF Repository Function (NRF) FQDN [119](#plmn-level-and-home-nf-repository-function-nrf-fqdn)

28.3.2.3.1 General [119](#general-27)

28.3.2.3.2 Format of NRF FQDN [119](#format-of-nrf-fqdn)

28.3.2.3.3 NRF URI [119](#nrf-uri)

28.3.2.4 Network Slice Selection Function (NSSF) FQDN [120](#network-slice-selection-function-nssf-fqdn)

28.3.2.4.1 General [120](#general-28)

28.3.2.4.2 Format of NSSF FQDN [120](#format-of-nssf-fqdn)

28.3.2.4.3 NSSF URI [120](#nssf-uri)

28.3.2.5 AMF Name [120](#amf-name)

28.3.2.6 5GS Tracking Area Identity (TAI) FQDN [121](#gs-tracking-area-identity-tai-fqdn)

28.3.2.7 AMF Set FQDN [121](#amf-set-fqdn)

28.3.2.8 AMF Instance FQDN [122](#amf-instance-fqdn)

28.3.2.9 SMF Set FQDN [123](#smf-set-fqdn)

28.3.2.10 Short Message Service Function (SMSF) FQDN [123](#short-message-service-function-smsf-fqdn)

28.3.2.11 5G DDNMF FQDN [123](#g-ddnmf-fqdn)

28.4 Information for Network Slicing [124](#information-for-network-slicing)

28.4.1 General [124](#general-29)

28.4.2 Format of the S-NSSAI [124](#format-of-the-s-nssai)

28.4.3 Ranges of S-NSSAIs [124](#ranges-of-s-nssais)

28.4.4 Network Slice Instance Identifier (NSI ID) [125](#network-slice-instance-identifier-nsi-id)

28.4.5 Network Slice Admission Control (NSAC) Service Area Identifier (SAI) [125](#network-slice-admission-control-nsac-service-area-identifier-sai)

28.5 NF FQDN Format for Inter PLMN Routing [125](#nf-fqdn-format-for-inter-plmn-routing)

28.5.1 General [125](#general-30)

28.5.2 Telescopic FQDN [125](#telescopic-fqdn)

28.6 5GS Tracking Area Identity (TAI) [125](#gs-tracking-area-identity-tai)

28.7 Network Access Identifier (NAI) [126](#network-access-identifier-nai)

28.7.1 Introduction [126](#introduction-29)

28.7.2 NAI format for SUPI [126](#nai-format-for-supi)

28.7.3 NAI format for SUCI [126](#nai-format-for-suci)

28.7.4 Emergency NAI for Limited Service State [128](#emergency-nai-for-limited-service-state-1)

28.7.5 Alternative NAI [128](#alternative-nai-2)

28.7.6 NAI used for 5G registration via trusted non-3GPP access [128](#nai-used-for-5g-registration-via-trusted-non-3gpp-access)

28.7.7 NAI used by N5CW devices via trusted non-3GPP access [129](#nai-used-by-n5cw-devices-via-trusted-non-3gpp-access)

28.7.7.0 General [129](#general-31)

28.7.7.1 Decorated NAI used for N5CW devices via trusted non-3GPP access [130](#decorated-nai-used-for-n5cw-devices-via-trusted-non-3gpp-access)

28.7.7.2 Decorated NAI used for N5CW devices via trusted non-3GPP access for SNPN [130](#decorated-nai-used-for-n5cw-devices-via-trusted-non-3gpp-access-for-snpn)

28.7.8 NAI format for 5G-GUTI [131](#nai-format-for-5g-guti)

28.7.9 Decorated NAI format for SUCI [131](#decorated-nai-format-for-suci)

28.7.9.1 General [131](#general-32)

28.7.9.2 Decorated NAI used for 5G NSWO [132](#decorated-nai-used-for-5g-nswo)

28.7.10 NAI format for UP-PRUK ID [133](#nai-format-for-up-pruk-id)

28.7.11 NAI format for CP-PRUK ID [133](#nai-format-for-cp-pruk-id)

28.7.12 NAI used for 5G NSWO [134](#nai-used-for-5g-nswo)

28.8 Generic Public Subscription Identifier (GPSI) [135](#generic-public-subscription-identifier-gpsi)

28.9 Internal-Group Identifier [135](#internal-group-identifier)

28.10 Presence Reporting Area Identifier (PRA ID) [135](#presence-reporting-area-identifier-pra-id-1)

28.11 CAG-Identifier [135](#cag-identifier)

28.12 NF Set Identifier (NF Set ID) [136](#_Toc233355265)

28.13 NF Service Set Identifier (NF Service Set ID) [137](#nf-service-set-identifier-nf-service-set-id)

28.14 Data Network Access Identifier (DNAI) [138](#data-network-access-identifier-dnai)

28.15 Global Cable Identifier (GCI) [138](#global-cable-identifier-gci)

28.15.1 Introduction [138](#introduction-30)

28.15.2 NAI format for SUPI containing a GCI [138](#nai-format-for-supi-containing-a-gci)

28.15.3 User Location Information for RG accessing the 5GC via W-5GCAN (HFC Node ID) [138](#user-location-information-for-rg-accessing-the-5gc-via-w-5gcan-hfc-node-id)

28.15.4 GCI [138](#_Toc233355272)

28.15.5 NAI format for SUCI containing a GCI [138](#nai-format-for-suci-containing-a-gci)

28.16 Global Line Identifier (GLI) [139](#global-line-identifier-gli)

28.16.1 Introduction [139](#introduction-31)

28.16.2 NAI format for SUPI containing a GLI [139](#nai-format-for-supi-containing-a-gli)

28.16.3 User Location Information for RG accessing the 5GC via W-5GBAN [139](#user-location-information-for-rg-accessing-the-5gc-via-w-5gban)

28.16.4 GLI [139](#gli)

28.16.5 NAI format for SUCI containing a GLI [139](#nai-format-for-suci-containing-a-gli)

28.17 DNS subdomain for operator usage in 5GC [140](#dns-subdomain-for-operator-usage-in-5gc)

28.18 NF FQDN Format for Inter SNPN Routing [140](#nf-fqdn-format-for-inter-snpn-routing)

28.18.1 General [140](#general-33)

28.19 User Location Information for UE accessing to 5G-RG acting as a TNAP [140](#user-location-information-for-ue-accessing-to-5g-rg-acting-as-a-tnap)

28.20 LCS Identifiers [140](#lcs-identifiers)

28.20.1 General [140](#general-34)

28.20.2 LCS Session Identity [140](#lcs-session-identity)

28.20.2.1 LCS correlation identifier [140](#lcs-correlation-identifier)

28.20.2.2 Routing identifier [140](#routing-identifier)

28.20.2.3 Deferred routing identifier [141](#deferred-routing-identifier)

28.20.3 LCS User Plane Connection/Binding ID [141](#lcs-user-plane-connectionbinding-id)

28.21 MPQUIC Identifiers [141](#mpquic-identifiers)

28.21.1 General [141](#general-35)

28.21.2 Context ID [141](#context-id)

29 Numbering, addressing and identification for RACS [141](#numbering-addressing-and-identification-for-racs)

29.1 Introduction [141](#introduction-32)

29.2 UE radio capability ID [141](#ue-radio-capability-id)

30 Identification of 5GS Multicast and Broadcast Services [142](#identification-of-5gs-multicast-and-broadcast-services)

30.1 Introduction [142](#introduction-33)

30.2 Structure of TMGI [142](#structure-of-tmgi-1)

30.3 Structure of Area Session ID [143](#structure-of-area-session-id)

30.4 Structure of MBS Frequency Selection Area ID [143](#_Toc233355301)

30.5 Structure of Associated Session ID [144](#structure-of-associated-session-id)

Annex A (informative): Colour Codes [144](#annex-a-informative-colour-codes)

A.1 Utilization of the BSIC [144](#a.1-utilization-of-the-bsic)

A.2 Guidance for planning [144](#a.2-guidance-for-planning)

A.3 Example of PLMN Colour Codes (NCCs) for the European region [145](#a.3-example-of-plmn-colour-codes-nccs-for-the-european-region)

Annex B (normative): IMEI Check Digit computation [146](#annex-b-normative-imei-check-digit-computation)

B.1 Representation of IMEI [146](#b.1-representation-of-imei)

B.2 Computation of CD for an IMEI [146](#b.2-computation-of-cd-for-an-imei)

B.3 Example of computation [146](#b.3-example-of-computation)

Annex C (normative): Naming convention [148](#annex-c-normative-naming-convention)

C.1 Routing Area Identities [148](#c.1-routing-area-identities)

C.2 GPRS Support Nodes [149](#c.2-gprs-support-nodes)

C.3 Target ID [149](#c.3-target-id)

Annex D (informative): Applicability and use of the ".3gppnetwork.org" domain name [149](#annex-d-informative-applicability-and-use-of-the-.3gppnetwork.org-domain-name)

Annex E (normative): Procedure for sub‑domain allocation [151](#annex-e-normative-procedure-for-subdomain-allocation)

Annex F (informative): Change history [153](#annex-f-informative-change-history)
