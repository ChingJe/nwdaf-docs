---
spec: TS 29.502
version: 18.11.0
release: '18'
source_archive: 29502-ib0.zip
source_document: 29502-ib0.docx
source_archive_sha256: c9132a7cf5493d033470dbbfe714121e0707138e99674c8aaf12bdab4841b264
source_document_sha256: 261580abdd73406068efdbaa9682cbdc3dbd5d31d88244d50c06d6a69c12945c
clause: contents
title: Contents
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Contents

Foreword [10](#foreword)

1 Scope [11](#scope)

2 References [11](#references)

3 Definitions and abbreviations [13](#definitions-and-abbreviations)

3.1 Definitions [13](#definitions)

3.2 Abbreviations [13](#abbreviations)

4 Overview [14](#overview)

4.1 Introduction [14](#introduction)

5 Services offered by the SMF [14](#services-offered-by-the-smf)

5.1 Introduction [14](#introduction-1)

5.2 Nsmf_PDUSession Service [15](#nsmf_pdusession-service)

5.2.1 Service Description [15](#service-description)

5.2.2 Service Operations [16](#service-operations)

5.2.2.1 Introduction [16](#introduction-2)

5.2.2.2 Create SM Context service operation [17](#create-sm-context-service-operation)

5.2.2.2.1 General [17](#general)

5.2.2.2.2 EPS to 5GS Idle mode mobility using N26 interface (with or without data forwarding) [21](#eps-to-5gs-idle-mode-mobility-using-n26-interface-with-or-without-data-forwarding)

5.2.2.2.3 EPS to 5GS Handover Preparation using N26 interface [23](#eps-to-5gs-handover-preparation-using-n26-interface)

5.2.2.2.4 I-SMF Insertion, Change or Removal during Xn based Handover [24](#i-smf-insertion-change-or-removal-during-xn-based-handover)

5.2.2.2.5 I-SMF Insertion, Change or Removal during N2 based Handover [25](#i-smf-insertion-change-or-removal-during-n2-based-handover)

5.2.2.2.6 Service Request with I-SMF insertion/change/removal or with V-SMF change [25](#service-request-with-i-smf-insertionchangeremoval-or-with-v-smf-change)

5.2.2.2.7 Registration procedure for a UE with a PDU session with I-SMF or V-SMF insertion, change and removal [26](#registration-procedure-for-a-ue-with-a-pdu-session-with-i-smf-or-v-smf-insertion-change-and-removal)

5.2.2.2.8 SMF Context Transfer procedure, LBO or no Roaming, no I-SMF [27](#smf-context-transfer-procedure-lbo-or-no-roaming-no-i-smf)

5.2.2.2.9 I-SMF Context Transfer procedure [27](#i-smf-context-transfer-procedure)

5.2.2.2.10 Handover between 3GPP and non-3GPP accesses with I-SMF insertion/removal or V-SMF change [27](#handover-between-3gpp-and-non-3gpp-accesses-with-i-smf-insertionremoval-or-v-smf-change)

5.2.2.2.11 Void [28](#void)

5.2.2.2.12 SMF triggered I-SMF selection/removal or V-SMF selection [28](#smf-triggered-i-smf-selectionremoval-or-v-smf-selection)

5.2.2.3 Update SM Context service operation [29](#update-sm-context-service-operation)

5.2.2.3.1 General [29](#general-1)

5.2.2.3.2 Activation and Deactivation of the User Plane connection of a PDU session [32](#activation-and-deactivation-of-the-user-plane-connection-of-a-pdu-session)

5.2.2.3.2.1 General [32](#general-2)

5.2.2.3.2.2 Activation of User Plane connectivity of a PDU session [32](#activation-of-user-plane-connectivity-of-a-pdu-session)

5.2.2.3.2.3 Deactivation of User Plane connectivity of a PDU session [34](#deactivation-of-user-plane-connectivity-of-a-pdu-session)

5.2.2.3.2.4 Changing the access type of a PDU session from non-3GPP access to 3GPP access during a Service Request procedure [35](#changing-the-access-type-of-a-pdu-session-from-non-3gpp-access-to-3gpp-access-during-a-service-request-procedure)

5.2.2.3.3 Xn Handover [35](#xn-handover)

5.2.2.3.4 N2 Handover [37](#n2-handover)

5.2.2.3.4.1 General [37](#general-3)

5.2.2.3.4.2 N2 Handover Preparation [37](#n2-handover-preparation)

5.2.2.3.4.3 N2 Handover Execution [39](#n2-handover-execution)

5.2.2.3.4.4 N2 Handover Cancellation [40](#n2-handover-cancellation)

5.2.2.3.5 Handover between 3GPP and untrusted non-3GPP access procedures [41](#handover-between-3gpp-and-untrusted-non-3gpp-access-procedures)

5.2.2.3.5.1 General [41](#general-4)

5.2.2.3.5.2 Handover of a PDU session without AMF change or with target AMF in same PLMN [41](#handover-of-a-pdu-session-without-amf-change-or-with-target-amf-in-same-plmn)

5.2.2.3.6 Inter-AMF change or mobility [41](#inter-amf-change-or-mobility)

5.2.2.3.7 RAN Initiated QoS Flow Mobility [42](#ran-initiated-qos-flow-mobility)

5.2.2.3.8 EPS to 5GS Handover using N26 interface [43](#eps-to-5gs-handover-using-n26-interface)

5.2.2.3.8.1 General [43](#general-5)

5.2.2.3.8.2 EPS to 5GS Handover Preparation [43](#eps-to-5gs-handover-preparation)

5.2.2.3.8.3 EPS to 5GS Handover Execution [44](#eps-to-5gs-handover-execution)

5.2.2.3.8.4 EPS to 5GS Handover Cancellation [44](#eps-to-5gs-handover-cancellation)

5.2.2.3.8.5 EPS to 5GS Handover Failure [44](#eps-to-5gs-handover-failure)

5.2.2.3.9 5GS to EPS Handover using N26 interface [44](#gs-to-eps-handover-using-n26-interface)

5.2.2.3.9.1 General [44](#general-6)

5.2.2.3.9.2 Data forwarding tunnels setup during 5GS to EPS handover [45](#_Toc217821581)

5.2.2.3.9.3 Indirect data forwarding tunnels removal for 5GS to EPS handover cancellation or failure [45](#_Toc217821582)

5.2.2.3.10 P-CSCF Restoration Procedure via AMF [46](#p-cscf-restoration-procedure-via-amf)

5.2.2.3.11 AMF requested PDU Session Release due to duplicated PDU Session Id [46](#amf-requested-pdu-session-release-due-to-duplicated-pdu-session-id)

5.2.2.3.12 AMF requested PDU Session Release due to slice not available [46](#amf-requested-pdu-session-release-due-to-slice-not-available)

5.2.2.3.13 Indirect Data Forwarding Tunnel establishment during N2 based Handover with I-SMF [47](#indirect-data-forwarding-tunnel-establishment-during-n2-based-handover-with-i-smf)

5.2.2.3.13A Indirect Data Forwarding Tunnel removal during N2 based Handover with I-SMF [47](#a-indirect-data-forwarding-tunnel-removal-during-n2-based-handover-with-i-smf)

5.2.2.3.14 Request to forward buffered downlink data packets at I-UPF [48](#request-to-forward-buffered-downlink-data-packets-at-i-upf)

5.2.2.3.15 Connection Suspend procedure [49](#connection-suspend-procedure)

5.2.2.3.16 Connection Resume in CM-IDLE with Suspend procedure [49](#connection-resume-in-cm-idle-with-suspend-procedure)

5.2.2.3.17 AMF requested PDU Session Release due to Network Slice-Specific Authentication and Authorization failure or revocation [50](#amf-requested-pdu-session-release-due-to-network-slice-specific-authentication-and-authorization-failure-or-revocation)

5.2.2.3.18 5GS to EPS Idle mode mobility using N26 interface with data forwarding [50](#gs-to-eps-idle-mode-mobility-using-n26-interface-with-data-forwarding)

5.2.2.3.19 AMF requested PDU Session Release due to Control Plane Only indication associated with PDU Session is not applicable any longer [51](#amf-requested-pdu-session-release-due-to-control-plane-only-indication-associated-with-pdu-session-is-not-applicable-any-longer)

5.2.2.3.20 AMF requested PDU Session Release due to ODB changes [51](#amf-requested-pdu-session-release-due-to-odb-changes)

5.2.2.3.21 N9 Forwarding Tunnel establishment between Branching Points or UL CLs controlled by different I-SMFs [51](#n9-forwarding-tunnel-establishment-between-branching-points-or-ul-cls-controlled-by-different-i-smfs)

5.2.2.3.22 Remote UE Report during 5G ProSe Communication via 5G ProSe Layer-3 UE-to-Network Relay without N3IWF procedure [52](#remote-ue-report-during-5g-prose-communication-via-5g-prose-layer-3-ue-to-network-relay-without-n3iwf-procedure)

5.2.2.3.23 AMF requested PDU Session Release due to V/I-SMF failure [52](#amf-requested-pdu-session-release-due-to-vi-smf-failure)

5.2.2.3.24 Connection Inactive procedure with CN based MT communication handling [53](#connection-inactive-procedure-with-cn-based-mt-communication-handling)

5.2.2.3.25 UE Triggered Connection Resume in RRC Inactive procedure [53](#ue-triggered-connection-resume-in-rrc-inactive-procedure)

5.2.2.3.26 AMF requested PDU Session Release due to Network Slice instance not available [54](#amf-requested-pdu-session-release-due-to-network-slice-instance-not-available)

5.2.2.3.27 AMF requested PDU Session Release due to MBSR not authorized [55](#amf-requested-pdu-session-release-due-to-mbsr-not-authorized)

5.2.2.4 Release SM Context service operation [55](#release-sm-context-service-operation)

5.2.2.4.1 General [55](#general-7)

5.2.2.5 Notify SM Context Status service operation [57](#notify-sm-context-status-service-operation)

5.2.2.5.1 General [57](#general-8)

5.2.2.6 Retrieve SM Context service operation [60](#retrieve-sm-context-service-operation)

5.2.2.6.1 General [60](#general-9)

5.2.2.7 Create service operation [63](#create-service-operation)

5.2.2.7.1 General [63](#general-10)

5.2.2.7.2 EPS to 5GS Idle mode mobility [67](#eps-to-5gs-idle-mode-mobility)

5.2.2.7.3 EPS to 5GS Handover Preparation [68](#eps-to-5gs-handover-preparation-1)

5.2.2.7.4 N2 Handover Preparation with I-SMF Insertion [69](#n2-handover-preparation-with-i-smf-insertion)

5.2.2.7.5 Xn Handover with I-SMF Insertion [69](#xn-handover-with-i-smf-insertion)

5.2.2.7.6 UE Triggered Service Request with I-SMF Insertion [69](#ue-triggered-service-request-with-i-smf-insertion)

5.2.2.8 Update service operation [70](#update-service-operation)

5.2.2.8.1 General [70](#general-11)

5.2.2.8.2 Update service operation towards H-SMF or SMF [72](#update-service-operation-towards-h-smf-or-smf)

5.2.2.8.2.1 General [72](#general-12)

5.2.2.8.2.2 UE or network (e.g. AMF, V-SMF, I-SMF) requested PDU session modification [73](#ue-or-network-e.g.-amf-v-smf-i-smf-requested-pdu-session-modification)

5.2.2.8.2.3 UE requested PDU session release [73](#ue-requested-pdu-session-release)

5.2.2.8.2.4 EPS to 5GS Handover Execution [74](#eps-to-5gs-handover-execution-1)

5.2.2.8.2.5 Handover between 3GPP access and untrusted or trusted non-3GPP access [74](#handover-between-3gpp-access-and-untrusted-or-trusted-non-3gpp-access)

5.2.2.8.2.6 P-CSCF Restoration Procedure via AMF [75](#p-cscf-restoration-procedure-via-amf-1)

5.2.2.8.2.7 Addition of PSA and BP or UL CL controlled by I-SMF [75](#addition-of-psa-and-bp-or-ul-cl-controlled-by-i-smf)

5.2.2.8.2.8 Removal of PSA and BP or UL CL controlled by I-SMF [76](#removal-of-psa-and-bp-or-ul-cl-controlled-by-i-smf)

5.2.2.8.2.9 Change of PSA for IPv6 multi-homing or UL CL controlled by I-SMF [76](#change-of-psa-for-ipv6-multi-homing-or-ul-cl-controlled-by-i-smf)

5.2.2.8.2.10 PDU Session modification with I-SMF or V-SMF change [77](#pdu-session-modification-with-i-smf-or-v-smf-change)

5.2.2.8.2.11 Sending by I-SMF of N4 notifications related with traffic usage reporting [77](#sending-by-i-smf-of-n4-notifications-related-with-traffic-usage-reporting)

5.2.2.8.2.12 N2 Handover Execution with I-SMF Insertion [77](#n2-handover-execution-with-i-smf-insertion)

5.2.2.8.2.13 N2 Handover Cancellation with I-SMF Insertion [78](#n2-handover-cancellation-with-i-smf-insertion)

5.2.2.8.2.14 EPS to 5GS Handover Cancellation [78](#eps-to-5gs-handover-cancellation-1)

5.2.2.8.2.15 5G-AN requested PDU session resource release [79](#g-an-requested-pdu-session-resource-release)

5.2.2.8.2.16 Xn Handover with or without I-SMF or V-SMF Change [79](#xn-handover-with-or-without-i-smf-or-v-smf-change)

5.2.2.8.2.17 EPS to 5GS Handover Failure [80](#eps-to-5gs-handover-failure-1)

5.2.2.8.2.18 EPS Bearer ID revocation [80](#eps-bearer-id-revocation)

5.2.2.8.2.19 Network requested PDU session release [80](#network-requested-pdu-session-release)

5.2.2.8.2.20 N2 Handover Execution with or without I-SMF or V-SMF Change [80](#n2-handover-execution-with-or-without-i-smf-or-v-smf-change)

5.2.2.8.2.21 Reporting of satellite backhaul change to (H-)SMF [81](#reporting-of-satellite-backhaul-change-to-h-smf)

5.2.2.8.2.22 Simultaneous change of PSA and BP or UL CL controlled by I-SMF [81](#simultaneous-change-of-psa-and-bp-or-ul-cl-controlled-by-i-smf)

5.2.2.8.2.23 Service Request without I-SMF/V-SMF Change or with I-SMF/V-SMF Change or with I-SMF Insertion [82](#service-request-without-i-smfv-smf-change-or-with-i-smfv-smf-change-or-with-i-smf-insertion)

5.2.2.8.2.24 Remote UE Report during 5G ProSe Communication via 5G ProSe Layer-3 UE-to-Network Relay without N3IWF [83](#remote-ue-report-during-5g-prose-communication-via-5g-prose-layer-3-ue-to-network-relay-without-n3iwf)

5.2.2.8.3 Update service operation towards V-SMF or I-SMF [83](#update-service-operation-towards-v-smf-or-i-smf)

5.2.2.8.3.1 General [83](#general-13)

5.2.2.8.3.2 Network (e.g. H-SMF, SMF) requested PDU session modification [84](#network-e.g.-h-smf-smf-requested-pdu-session-modification)

5.2.2.8.3.3 Network (e.g. H-SMF, SMF) or UE requested PDU session release [85](#network-e.g.-h-smf-smf-or-ue-requested-pdu-session-release)

5.2.2.8.3.4 Handover between 3GPP and untrusted non-3GPP access, from 5GC-N3IWF to EPS or from 5GS to EPC/ePDG [85](#handover-between-3gpp-and-untrusted-non-3gpp-access-from-5gc-n3iwf-to-eps-or-from-5gs-to-epcepdg)

5.2.2.8.3.5 EPS Bearer ID assignment [86](#eps-bearer-id-assignment)

5.2.2.8.3.6 Addition of PSA and BP or UL CL controlled by I-SMF [86](#addition-of-psa-and-bp-or-ul-cl-controlled-by-i-smf-1)

5.2.2.8.3.7 Removal of PSA and BP or UL CL controlled by I-SMF [87](#removal-of-psa-and-bp-or-ul-cl-controlled-by-i-smf-1)

5.2.2.8.3.8 Change of PSA for IPv6 multi-homing or UL CL controlled by I-SMF [87](#change-of-psa-for-ipv6-multi-homing-or-ul-cl-controlled-by-i-smf-1)

5.2.2.8.3.9 Policy update procedures with an I-SMF [88](#policy-update-procedures-with-an-i-smf)

5.2.2.8.3.10 Simultaneous change of PSA and BP or UL CL controlled by I-SMF [88](#simultaneous-change-of-psa-and-bp-or-ul-cl-controlled-by-i-smf-1)

5.2.2.8.3.11 Network (e.g. AMF) triggered network slice replacement with PDU session retained [89](#network-e.g.-amf-triggered-network-slice-replacement-with-pdu-session-retained)

5.2.2.9 Release service operation [90](#release-service-operation)

5.2.2.9.1 General [90](#general-14)

5.2.2.10 Notify Status service operation [91](#notify-status-service-operation)

5.2.2.10.1 General [91](#general-15)

5.2.2.11 Send MO Data service operation [93](#send-mo-data-service-operation)

5.2.2.11.1 General [93](#general-16)

5.2.2.12 Transfer MO Data service operation [94](#transfer-mo-data-service-operation)

5.2.2.12.1 General [94](#general-17)

5.2.2.13 Transfer MT Data service operation [94](#transfer-mt-data-service-operation)

5.2.2.13.1 General [94](#general-18)

5.2.2.14 Retrieve service operation [95](#retrieve-service-operation)

5.2.2.14.1 General [95](#general-19)

5.2.3 General procedures [96](#general-procedures)

5.2.3.1 Transfer of NAS SM information between UE and H-SMF for Home Routed PDU sessions [96](#transfer-of-nas-sm-information-between-ue-and-h-smf-for-home-routed-pdu-sessions)

5.2.3.1.1 General [96](#general-20)

5.2.3.1.2 V-SMF Behaviour [96](#v-smf-behaviour)

5.2.3.1.3 H-SMF Behaviour [97](#h-smf-behaviour)

5.2.3.2 Transfer of NAS SM information between UE and SMF for PDU sessions with an I-SMF [97](#transfer-of-nas-sm-information-between-ue-and-smf-for-pdu-sessions-with-an-i-smf)

5.2.3.2.1 General [97](#general-21)

5.2.3.3 Detection and handling of late arriving requests [97](#detection-and-handling-of-late-arriving-requests)

5.2.3.3.1 Handling of requests which collide with an existing SM context or PDU session context [97](#handling-of-requests-which-collide-with-an-existing-sm-context-or-pdu-session-context)

5.2.3.3.1.1 General [97](#general-22)

5.2.3.3.1.2 Principles [97](#principles)

5.2.3.3.2 Detection and handling of requests which have timed out at the HTTP client [98](#detection-and-handling-of-requests-which-have-timed-out-at-the-http-client)

5.2.3.3.2.1 General [98](#general-23)

5.2.3.4 UE Location Information [98](#ue-location-information)

6 API Definitions [99](#api-definitions)

6.1 Nsmf_PDUSession Service API [99](#nsmf_pdusession-service-api)

6.1.1 API URI [99](#api-uri)

6.1.2 Usage of HTTP [99](#usage-of-http)

6.1.2.1 General [99](#general-24)

6.1.2.2 HTTP standard headers [99](#http-standard-headers)

6.1.2.2.1 General [99](#general-25)

6.1.2.2.2 Content type [99](#content-type)

6.1.2.3 HTTP custom headers [100](#http-custom-headers)

6.1.2.3.1 General [100](#general-26)

6.1.2.3.2 3gpp-Sbi-Origination-Timestamp [100](#gpp-sbi-origination-timestamp)

6.1.2.4 HTTP multipart messages [100](#http-multipart-messages)

6.1.2.5 HTTP/2 request retries [101](#http2-request-retries)

6.1.3 Resources [101](#resources)

6.1.3.1 Overview [101](#overview-1)

6.1.3.2 Resource: SM contexts collection [103](#resource-sm-contexts-collection)

6.1.3.2.1 Description [103](#description)

6.1.3.2.2 Resource Definition [103](#resource-definition)

6.1.3.2.3 Resource Standard Methods [104](#resource-standard-methods)

6.1.3.2.3.1 POST [104](#post)

6.1.3.2.4 Resource Custom Operations [107](#resource-custom-operations)

6.1.3.3 Resource: Individual SM context [107](#resource-individual-sm-context)

6.1.3.3.1 Description [107](#description-1)

6.1.3.3.2 Resource Definition [108](#resource-definition-1)

6.1.3.3.3 Resource Standard Methods [108](#resource-standard-methods-1)

6.1.3.3.4 Resource Custom Operations [108](#resource-custom-operations-1)

6.1.3.3.4.1 Overview [108](#overview-2)

6.1.3.3.4.2 Operation: modify [108](#operation-modify)

6.1.3.3.4.2.1 Description [108](#description-2)

6.1.3.3.4.2.2 Operation Definition [108](#operation-definition)

6.1.3.3.4.3 Operation: release [111](#operation-release)

6.1.3.3.4.3.1 Description [111](#description-3)

6.1.3.3.4.3.2 Operation Definition [111](#operation-definition-1)

6.1.3.3.4.4 Operation: retrieve [112](#operation-retrieve)

6.1.3.3.4.4.1 Description [112](#description-4)

6.1.3.3.4.4.2 Operation Definition [112](#operation-definition-2)

6.1.3.3.4.5 Operation: send-mo-data [113](#operation-send-mo-data)

6.1.3.3.4.5.1 Description [113](#description-5)

6.1.3.3.4.5.2 Operation Definition [113](#operation-definition-3)

6.1.3.4 Void [115](#void-1)

6.1.3.5 Resource: PDU sessions collection (H-SMF or SMF) [115](#resource-pdu-sessions-collection-h-smf-or-smf)

6.1.3.5.1 Description [115](#description-6)

6.1.3.5.2 Resource Definition [115](#resource-definition-2)

6.1.3.5.3 Resource Standard Methods [115](#resource-standard-methods-2)

6.1.3.5.3.1 POST [115](#post-1)

6.1.3.5.4 Resource Custom Operations [118](#resource-custom-operations-2)

6.1.3.5.4.1 Overview [118](#overview-3)

6.1.3.6 Resource: Individual PDU session (H-SMF or SMF) [118](#resource-individual-pdu-session-h-smf-or-smf)

6.1.3.6.1 Description [118](#description-7)

6.1.3.6.2 Resource Definition [119](#resource-definition-3)

6.1.3.6.3 Resource Standard Methods [119](#resource-standard-methods-3)

6.1.3.6.4 Resource Custom Operations [119](#resource-custom-operations-3)

6.1.3.6.4.1 Overview [119](#overview-4)

6.1.3.6.4.2 Operation: modify [119](#operation-modify-1)

6.1.3.6.4.2.1 Description [119](#description-8)

6.1.3.6.4.2.2 Operation Definition [119](#operation-definition-4)

6.1.3.6.4.3 Operation: release [121](#operation-release-1)

6.1.3.6.4.3.1 Description [121](#description-9)

6.1.3.6.4.3.2 Operation Definition [121](#operation-definition-5)

6.1.3.6.4.4 Operation: transfer-mo-data [122](#operation-transfer-mo-data)

6.1.3.6.4.4.1 Description [122](#description-10)

6.1.3.6.4.4.2 Operation Definition [122](#operation-definition-6)

6.1.3.6.4.5 Operation: retrieve [123](#operation-retrieve-1)

6.1.3.6.4.5.1 Description [123](#description-11)

6.1.3.6.4.5.2 Operation Definition [123](#operation-definition-7)

6.1.3.7 Resource: Individual PDU session (V-SMF or I-SMF) [124](#resource-individual-pdu-session-v-smf-or-i-smf)

6.1.3.7.1 Description [124](#description-12)

6.1.3.7.2 Resource Definition [124](#resource-definition-4)

6.1.3.7.3 Resource Standard Methods [124](#resource-standard-methods-4)

6.1.3.7.3.1 POST [124](#post-2)

6.1.3.7.4 Resource Custom Operations [126](#resource-custom-operations-4)

6.1.3.7.4.1 Overview [126](#overview-5)

6.1.3.7.4.2 Operation: modify [126](#operation-modify-2)

6.1.3.7.4.2.1 Description [126](#description-13)

6.1.3.7.4.2.2 Operation Definition [126](#operation-definition-8)

6.1.3.7.4.3 Operation: transfer-mt-data [129](#operation-transfer-mt-data)

6.1.3.7.4.3.1 Description [129](#description-14)

6.1.3.7.4.3.2 Operation Definition [129](#operation-definition-9)

6.1.4 Custom Operations without associated resources [130](#custom-operations-without-associated-resources)

6.1.5 Notifications [130](#notifications)

6.1.5.1 General [130](#general-27)

6.1.5.2 SM Context Status Notification [131](#sm-context-status-notification)

6.1.5.2.1 Description [131](#description-15)

6.1.5.2.2 Notification Definition [131](#notification-definition)

6.1.6 Data Model [132](#data-model)

6.1.6.1 General [132](#general-28)

6.1.6.2 Structured data types [139](#structured-data-types)

6.1.6.2.1 Introduction [139](#introduction-3)

6.1.6.2.2 Type: SmContextCreateData [140](#type-smcontextcreatedata)

6.1.6.2.3 Type: SmContextCreatedData [156](#type-smcontextcreateddata)

6.1.6.2.4 Type: SmContextUpdateData [160](#type-smcontextupdatedata)

6.1.6.2.5 Type: SmContextUpdatedData [170](#type-smcontextupdateddata)

6.1.6.2.6 Type: SmContextReleaseData [174](#type-smcontextreleasedata)

6.1.6.2.7 Type: SmContextRetrieveData [177](#type-smcontextretrievedata)

6.1.6.2.8 Type: SmContextStatusNotification [180](#type-smcontextstatusnotification)

6.1.6.2.9 Type: PduSessionCreateData [184](#type-pdusessioncreatedata)

6.1.6.2.10 Type: PduSessionCreatedData [194](#type-pdusessioncreateddata)

6.1.6.2.11 Type: HsmfUpdateData [201](#type-hsmfupdatedata)

6.1.6.2.12 Type: HsmfUpdatedData [212](#type-hsmfupdateddata)

6.1.6.2.13 Type: ReleaseData [216](#type-releasedata)

6.1.6.2.14 Type: HsmfUpdateError [217](#type-hsmfupdateerror)

6.1.6.2.15 Type: VsmfUpdateData [218](#type-vsmfupdatedata)

6.1.6.2.16 Type: VsmfUpdatedData [223](#type-vsmfupdateddata)

6.1.6.2.17 Type: StatusNotification [227](#type-statusnotification)

6.1.6.2.18 Type: QosFlowItem [229](#type-qosflowitem)

6.1.6.2.19 Type: QosFlowSetupItem [230](#type-qosflowsetupitem)

6.1.6.2.20 Type: QosFlowAddModifyRequestItem [231](#type-qosflowaddmodifyrequestitem)

6.1.6.2.21 Type: QosFlowReleaseRequestItem [231](#type-qosflowreleaserequestitem)

6.1.6.2.22 Type: QosFlowProfile [232](#type-qosflowprofile)

6.1.6.2.23 Type: GbrQosFlowInformation [233](#type-gbrqosflowinformation)

6.1.6.2.24 Type: QosFlowNotifyItem [233](#type-qosflownotifyitem)

6.1.6.2.25 Type: Void [234](#type-void)

6.1.6.2.26 Type: Void [234](#type-void-1)

6.1.6.2.27 Type: SmContextRetrievedData [234](#type-smcontextretrieveddata)

6.1.6.2.28 Type: TunnelInfo [235](#type-tunnelinfo)

6.1.6.2.29 Type: StatusInfo [235](#type-statusinfo)

6.1.6.2.30 Type: VsmfUpdateError [236](#type-vsmfupdateerror)

6.1.6.2.31 Type: EpsPdnCnxInfo [238](#type-epspdncnxinfo)

6.1.6.2.32 Type: EpsBearerInfo [238](#type-epsbearerinfo)

6.1.6.2.33 Type: PduSessionNotifyItem [238](#type-pdusessionnotifyitem)

6.1.6.2.34 Type: EbiArpMapping [239](#type-ebiarpmapping)

6.1.6.2.35 Type: SmContextCreateError [239](#type-smcontextcreateerror)

6.1.6.2.36 Type: SmContextUpdateError [240](#type-smcontextupdateerror)

6.1.6.2.37 Type: PduSessionCreateError [241](#type-pdusessioncreateerror)

6.1.6.2.38 Type: MmeCapabilities [242](#type-mmecapabilities)

6.1.6.2.39 Type: SmContext [243](#type-smcontext)

6.1.6.2.40 Type: ExemptionInd [251](#type-exemptionind)

6.1.6.2.41 Type: PsaInformation [252](#type-psainformation)

6.1.6.2.42 Type: DnaiInformation [252](#type-dnaiinformation)

6.1.6.2.43 Type: N4Information [253](#type-n4information)

6.1.6.2.44 Type: IndirectDataForwardingTunnelInfo [254](#type-indirectdataforwardingtunnelinfo)

6.1.6.2.45 Type: SmContextReleasedData [254](#type-smcontextreleaseddata)

6.1.6.2.46 Type: ReleasedData [255](#type-releaseddata)

6.1.6.2.47 Type: SendMoDataReqData [255](#type-sendmodatareqdata)

6.1.6.2.48 Type: CnAssistedRanPara [256](#type-cnassistedranpara)

6.1.6.2.49 Type: UlclBpInformation [256](#type-ulclbpinformation)

6.1.6.2.50 Type: TransferMoDataReqData [256](#type-transfermodatareqdata)

6.1.6.2.51 Type: TransferMtDataReqData [257](#_Toc217821817)

6.1.6.2.52 Type: TransferMtDataError [257](#type-transfermtdataerror)

6.1.6.2.53 Type: TransferMtDataAddInfo [257](#type-transfermtdataaddinfo)

6.1.6.2.54 Type: VplmnQos [258](#type-vplmnqos)

6.1.6.2.55 Type: DdnFailureSubs [258](#type-ddnfailuresubs)

6.1.6.2.56 Type: RetrieveData [259](#type-retrievedata)

6.1.6.2.57 Type: RetrievedData [259](#type-retrieveddata)

6.1.6.2.58 Type: SecurityResult [259](#type-securityresult)

6.1.6.2.59 Type: UpSecurityInfo [260](#type-upsecurityinfo)

6.1.6.2.60 Type: DdnFailureSubInfo [260](#type-ddnfailuresubinfo)

6.1.6.2.61 Type: AlternativeQosProfile [261](#type-alternativeqosprofile)

6.1.6.2.62 Type: ProblemDetailsAddInfo [261](#type-problemdetailsaddinfo)

6.1.6.2.63 Type: ExtProblemDetails [261](#type-extproblemdetails)

6.1.6.2.64 Type: QosMonitoringInfo [262](#type-qosmonitoringinfo)

6.1.6.2.65 Type: IpAddress [262](#type-ipaddress)

6.1.6.2.66 Type: RedundantPduSessionInformation [262](#type-redundantpdusessioninformation)

6.1.6.2.67 Type: QosFlowTunnel [262](#type-qosflowtunnel)

6.1.6.2.68 Type: TargetDnaiInfo [263](#type-targetdnaiinfo)

6.1.6.2.69 Type: AfCoordinationInfo [263](#type-afcoordinationinfo)

6.1.6.2.70 Type: NotificationInfo [263](#type-notificationinfo)

6.1.6.2.71 Type: AnchorSmfFeatures [263](#type-anchorsmffeatures)

6.1.6.2.72 Type: HrsboInfoFromVplmn [265](#type-hrsboinfofromvplmn)

6.1.6.2.73 Type: HrsboInfoFromHplmn [268](#type-hrsboinfofromhplmn)

6.1.6.2.74 Type: EasInfoToRefresh [271](#type-easinfotorefresh)

6.1.6.2.75 Type: EcnMarkingCongestionInfoReq [271](#type-ecnmarkingcongestioninforeq)

6.1.6.2.76 Type: EcnMarkingCongestionInfoStatus [272](#type-ecnmarkingcongestioninfostatus)

6.1.6.2.77 Type: TscAssistanceInformation [272](#type-tscassistanceinformation)

6.1.6.2.78 Type: N6JitterInformation [272](#type-n6jitterinformation)

6.1.6.2.79 Type: TrafficInfluenceInfo [273](#type-trafficinfluenceinfo)

6.1.6.2.80 Type: TrafficInfluenceData [273](#type-trafficinfluencedata)

6.1.6.3 Simple data types and enumerations [273](#simple-data-types-and-enumerations)

6.1.6.3.1 Introduction [273](#introduction-4)

6.1.6.3.2 Simple data types [274](#simple-data-types)

6.1.6.3.3 Enumeration: UpCnxState [277](#enumeration-upcnxstate)

6.1.6.3.4 Enumeration: HoState [277](#enumeration-hostate)

6.1.6.3.5 Enumeration: RequestType [277](#enumeration-requesttype)

6.1.6.3.6 Enumeration: RequestIndication [277](#enumeration-requestindication)

6.1.6.3.7 Enumeration: NotificationCause [278](#enumeration-notificationcause)

6.1.6.3.8 Enumeration: Cause [278](#enumeration-cause)

6.1.6.3.9 Enumeration: ResourceStatus [282](#enumeration-resourcestatus)

6.1.6.3.10 Enumeration: DnnSelectionMode [283](#enumeration-dnnselectionmode)

6.1.6.3.11 Enumeration: EpsInterworkingIndication [283](#enumeration-epsinterworkingindication)

6.1.6.3.12 Enumeration: N2SmInfoType [284](#enumeration-n2sminfotype)

6.1.6.3.13 Enumeration: MaxIntegrityProtectedDataRate [285](#enumeration-maxintegrityprotecteddatarate)

6.1.6.3.14 Enumeration: MaReleaseIndication [285](#enumeration-mareleaseindication)

6.1.6.3.15 Enumeration: SmContextType [285](#enumeration-smcontexttype)

6.1.6.3.16 Enumeration: PsaIndication [285](#enumeration-psaindication)

6.1.6.3.17 Enumeration: N4MessageType [285](#enumeration-n4messagetype)

6.1.6.3.18 Enumeration: QosFlowAccessType [286](#enumeration-qosflowaccesstype)

6.1.6.3.19 Enumeration: UnavailableAccessIndication [286](#enumeration-unavailableaccessindication)

6.1.6.3.20 Enumeration: ProtectionResult [286](#enumeration-protectionresult)

6.1.6.3.21 Enumeration: QosMonitoringReq [286](#enumeration-qosmonitoringreq)

6.1.6.3.22 Enumeration: Rsn [287](#enumeration-rsn)

6.1.6.3.23 Enumeration: SmfSelectionType [287](#enumeration-smfselectiontype)

6.1.6.3.24 Enumeration: PduSessionContextType [287](#enumeration-pdusessioncontexttype)

6.1.6.3.25 Enumeration: PendingUpdateInfo [287](#enumeration-pendingupdateinfo)

6.1.6.3.26 Enumeration: EstablishmentRejectionCause [287](#enumeration-establishmentrejectioncause)

6.1.6.3.27 Enumeration: EcnMarkingReq [288](#enumeration-ecnmarkingreq)

6.1.6.3.28 Enumeration: CongestionInfoReq [288](#enumeration-congestioninforeq)

6.1.6.3.29 Enumeration: ActivationStatus [288](#enumeration-activationstatus)

6.1.6.4 Binary data [288](#binary-data)

6.1.6.4.1 Introduction [288](#introduction-5)

6.1.6.4.2 N1 SM Message [289](#n1-sm-message)

6.1.6.4.3 N2 SM Information [289](#n2-sm-information)

6.1.6.4.4 n1SmInfoFromUe, n1SmInfoToUe, unknownN1SmInfo [290](#n1sminfofromue-n1sminfotoue-unknownn1sminfo)

6.1.6.4.5 N4 Message Payload [292](#n4-message-payload)

6.1.6.4.6 Mobile Originated Data [292](#mobile-originated-data)

6.1.6.4.7 Mobile Terminated Data [293](#mobile-terminated-data)

6.1.7 Error Handling [293](#error-handling)

6.1.7.1 General [293](#general-29)

6.1.7.2 Protocol Errors [293](#protocol-errors)

6.1.7.3 Application Errors [293](#application-errors)

6.1.8 Feature Negotiation [297](#feature-negotiation)

6.1.9 Security [303](#security)

6.1.10 HTTP redirection [303](#http-redirection)

Annex A (normative): OpenAPI specification [304](#annex-a-normative-openapi-specification)

A.1 General [304](#a.1-general)

A.2 Nsmf_PDUSession API [304](#a.2-nsmf_pdusession-api)

Annex B (Informative): HTTP Multipart Messages [375](#annex-b-informative-http-multipart-messages)

B.1 Example of HTTP multipart message [375](#b.1-example-of-http-multipart-message)

B.1.1 General [375](#b.1.1-general)

B.1.2 Example HTTP multipart message with N1 SM Message binary data [375](#b.1.2-example-http-multipart-message-with-n1-sm-message-binary-data)

Annex C (Normative): ABNF grammar for 3GPP SBI HTTP custom headers [375](#annex-c-normative-abnf-grammar-for-3gpp-sbi-http-custom-headers)

C.1 General [375](#c.1-general)

C.2 ABNF definitions (Filename: "TS29502_CustomHeaders.abnf") [376](#c.2-abnf-definitions-filename-ts29502_customheaders.abnf)

Annex D (Informative): Charging Identifier Handling [377](#annex-d-informative-charging-identifier-handling)

D.1 Usage of Charging ID and SMF Charging ID [377](#d.1-usage-of-charging-id-and-smf-charging-id)

D.1.1 General [377](#d.1.1-general)

D.1.2 HPLMN supporting the SMF Charging ID [377](#d.1.2-hplmn-supporting-the-smf-charging-id)

D.1.3 HPLMN not supporting the SMF Charging ID [377](#d.1.3-hplmn-not-supporting-the-smf-charging-id)

D.1.4 Transfer of (SMF) Charging ID between SMFs [377](#d.1.4-transfer-of-smf-charging-id-between-smfs)

Annex E (informative): Change history [379](#annex-e-informative-change-history)
