---
spec: TS 29.523
version: 18.7.0
release: '18'
clause: contents
title: Contents
source_archive: 29523-i70.zip
source_document: 29523-i70.docx
source_archive_sha256: 5cd859d1209362653dc38885359730445b9477dac772b9b67bc95f3bbe64cdf0
source_document_sha256: 1f2227da69ad8fc7af4a88265d73ec63df4f366a7ed3eccf8bbbe8b037dba1ba
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Contents

Foreword [5](#foreword)

1 Scope [6](#scope)

2 References [6](#references)

3 Definitions, symbols and abbreviations [7](#definitions-symbols-and-abbreviations)

3.1 Definitions [7](#definitions)

3.2 Abbreviations [7](#abbreviations)

4 Npcf_EventExposure Service [8](#npcf_eventexposure-service)

4.1 Service Description [8](#service-description)

4.1.1 Overview [8](#overview)

4.1.2 Service Architecture [8](#service-architecture)

4.1.3 Network Functions [9](#network-functions)

4.1.3.1 Policy Control Function (PCF) [9](#policy-control-function-pcf)

4.1.3.2 NF Service Consumers [10](#nf-service-consumers)

4.2 Service Operations [10](#service-operations)

4.2.1 Introduction [10](#introduction)

4.2.2 Npcf_EventExposure_Subscribe service operation [10](#npcf_eventexposure_subscribe-service-operation)

4.2.2.1 General [10](#general)

4.2.2.2 Creating a new subscription [11](#creating-a-new-subscription)

4.2.2.3 Modifying an existing subscription [13](#modifying-an-existing-subscription)

4.2.3 Npcf_EventExposure_UnSubscribe service operation [15](#npcf_eventexposure_unsubscribe-service-operation)

4.2.3.1 General [15](#general-1)

4.2.3.2 Unsubscription from event notifications [15](#unsubscription-from-event-notifications)

4.2.4 Npcf_EventExposure_Notify service operation [15](#npcf_eventexposure_notify-service-operation)

4.2.4.1 General [15](#general-2)

4.2.4.2 Notification about subscribed events [15](#notification-about-subscribed-events)

5 Npcf_EventExposure Service API [18](#npcf_eventexposure-service-api)

5.1 Introduction [18](#introduction-1)

5.2 Usage of HTTP [18](#usage-of-http)

5.2.1 General [18](#general-3)

5.2.2 HTTP standard headers [18](#http-standard-headers)

5.2.2.1 General [18](#general-4)

5.2.2.2 Content type [19](#content-type)

5.2.3 HTTP custom headers [19](#http-custom-headers)

5.2.3.1 General [19](#general-5)

5.3 Resources [19](#resources)

5.3.1 Resource Structure [19](#resource-structure)

5.3.2 Resource: Policy Control Events Subscriptions (Collection) [20](#resource-policy-control-events-subscriptions-collection)

5.3.2.1 Description [20](#description)

5.3.2.2 Resource definition [20](#resource-definition)

5.3.2.3 Resource Standard Methods [20](#resource-standard-methods)

5.3.2.3.1 POST [20](#post)

5.3.2.4 Resource Custom Operations [20](#resource-custom-operations)

5.3.3 Resource: Individual Policy Control Events Subscription (Document) [21](#resource-individual-policy-control-events-subscription-document)

5.3.3.1 Description [21](#description-1)

5.3.3.2 Resource definition [21](#resource-definition-1)

5.3.3.3 Resource Standard Methods [21](#resource-standard-methods-1)

5.3.3.3.1 GET [21](#get)

5.3.3.3.2 PUT [22](#put)

5.3.3.3.3 DELETE [23](#delete)

5.3.3.4 Resource Custom Operations [24](#resource-custom-operations-1)

5.4 Custom Operations without associated resources [24](#custom-operations-without-associated-resources)

5.5 Notifications [25](#notifications)

5.5.1 General [25](#general-6)

5.5.2 Policy Control Event Notification [25](#policy-control-event-notification)

5.5.2.1 Description [25](#description-2)

5.5.2.2 Target URI [25](#target-uri)

5.5.2.3 Standard Methods [25](#standard-methods)

5.5.2.3.1 POST [25](#post-1)

5.6 Data Model [26](#data-model)

5.6.1 General [26](#general-7)

5.6.2 Structured data types [28](#structured-data-types)

5.6.2.1 Introduction [28](#introduction-2)

5.6.2.2 Type PcEventExposureSubsc [29](#type-pceventexposuresubsc)

5.6.2.3 Type PcEventExposureNotif [30](#type-pceventexposurenotif)

5.6.2.4 Type ReportingInformation [31](#type-reportinginformation)

5.6.2.5 Type ServiceIdentification [32](#type-serviceidentification)

5.6.2.6 Type EthernetFlowInfo [32](#type-ethernetflowinfo)

5.6.2.7 Type IpFlowInfo [32](#type-ipflowinfo)

5.6.2.8 Type PcEventNotification [33](#type-pceventnotification)

5.6.2.9 Type PduSessionInformation [34](#type-pdusessioninformation)

5.6.2.10 Type SnssaiDnnCombination [34](#type-snssaidnncombination)

5.6.3 Simple data types and enumerations [34](#simple-data-types-and-enumerations)

5.6.3.1 Introduction [34](#introduction-3)

5.6.3.2 Simple data types [34](#simple-data-types)

5.6.3.3 Enumeration: PcEvent [34](#enumeration-pcevent)

5.7 Error handling [35](#error-handling)

5.7.1 General [35](#general-8)

5.7.2 Protocol Errors [35](#protocol-errors)

5.7.3 Application Errors [35](#application-errors)

5.8 Feature negotiation [35](#feature-negotiation)

5.9 Security [36](#security)

Annex A (normative): OpenAPI specification [37](#annex-a-normative-openapi-specification)

A.1 General [37](#a.1-general)

A.2 Npcf_EventExposure API [37](#a.2-npcf_eventexposure-api)

Annex B (informative): Change history [45](#annex-b-informative-change-history)
