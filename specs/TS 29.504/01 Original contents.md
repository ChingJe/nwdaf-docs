---
spec: TS 29.504
version: 18.13.0
release: '18'
source_archive: 29504-id0.zip
source_document: 29504-id0.docx
source_archive_sha256: 08eb3fe98bf504d4516fefd5b25d7d85d32137d58050b96e05c110afd341706b
source_document_sha256: 819017105e6641a94e926626c4b8c28c960eff13fd1c084dc9c76b2ef745ba6a
clause: contents
title: Contents
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Contents

Foreword [6](#foreword)

1 Scope [7](#scope)

2 References [7](#references)

3 Definitions and abbreviations [8](#definitions-and-abbreviations)

3.1 Definitions [8](#definitions)

3.2 Abbreviations [8](#abbreviations)

4 Overview [8](#overview)

5 Services offered by the UDR [9](#services-offered-by-the-udr)

5.1 Introduction [9](#introduction)

5.2 Nudr_DataRepository Service [10](#nudr_datarepository-service)

5.2.1 Service Description [10](#service-description)

5.2.1.1 Service and operation description [10](#service-and-operation-description)

5.2.1.2 Service operation and data access authorization [10](#service-operation-and-data-access-authorization)

5.2.2 Service Operations [11](#service-operations)

5.2.2.1 Introduction [11](#introduction-1)

5.2.2.2 Query [11](#query)

5.2.2.2.1 General [11](#general)

5.2.2.2.2 Data retrieval [11](#data-retrieval)

5.2.2.2.3 Retrieval of subset of a resource [12](#retrieval-of-subset-of-a-resource)

5.2.2.3 Create [13](#create)

5.2.2.3.1 General [13](#general-1)

5.2.2.3.2 Data Creation using PUT [14](#data-creation-using-put)

5.2.2.3.3 Data Creation using POST [14](#data-creation-using-post)

5.2.2.4 Delete [15](#delete)

5.2.2.4.1 General [15](#general-2)

5.2.2.4.2 Deleting Data [15](#deleting-data)

5.2.2.5 Update [15](#update)

5.2.2.5.1 General [15](#general-3)

5.2.2.5.2 Data Update using PATCH [15](#data-update-using-patch)

5.2.2.5.3 Data Update using PUT [16](#data-update-using-put)

5.2.2.6 Subscribe [16](#subscribe)

5.2.2.6.1 General [16](#general-4)

5.2.2.6.2 NF service consumer subscribes to notifications to UDR [17](#nf-service-consumer-subscribes-to-notifications-to-udr)

5.2.2.6.3 Stateless UDM subscribes to notifications to UDR [17](#_Toc232472931)

5.2.2.7 Unsubscribe [18](#_Toc232472932)

5.2.2.7.1 General [18](#general-5)

5.2.2.7.2 Unsubscribe service operation [18](#unsubscribe-service-operation)

5.2.2.8 Notify [19](#notify)

5.2.2.8.1 General [19](#general-6)

5.2.2.8.2 Notification to NF service consumer on data change [19](#notification-to-nf-service-consumer-on-data-change)

5.2.2.8.3 Notification to stateless UDM on data change [19](#notification-to-stateless-udm-on-data-change)

5.2.2.9 DataRestorationNotification [20](#datarestorationnotification)

5.2.2.9.1 General [20](#general-7)

5.2.2.9.2 Notification on Data Restoration [20](#notification-on-data-restoration)

5.3 Nudr_GroupIDmap Service [21](#nudr_groupidmap-service)

5.3.1 Service Description [21](#service-description-1)

5.3.1.1 Service and operation description [21](#service-and-operation-description-1)

5.3.2 Service Operations [21](#service-operations-1)

5.3.2.1 Introduction [21](#introduction-2)

5.3.2.2 Query [22](#query-1)

5.3.2.2.1 General [22](#general-8)

5.3.2.2.2 NF Group ID retrieval [22](#nf-group-id-retrieval)

5.3.2.3 QueryRID [22](#queryrid)

5.3.2.3.1 General [22](#general-9)

5.3.2.3.2 Routing IDs retrieval [22](#routing-ids-retrieval)

5.3.2.4 Subscribe [23](#subscribe-1)

5.3.2.4.1 General [23](#general-10)

5.3.2.4.2 NF service consumer subscribes to notifications to UDR [23](#nf-service-consumer-subscribes-to-notifications-to-udr-1)

5.3.2.5 Unsubscribe [24](#unsubscribe-1)

5.3.2.5.1 General [24](#general-11)

5.3.2.5.2 Unsubscribe service operation [24](#unsubscribe-service-operation-1)

5.3.2.6 Notify [24](#notify-1)

5.3.2.6.1 General [24](#general-12)

5.3.2.6.2 Notification to NF service consumer on data change [24](#notification-to-nf-service-consumer-on-data-change-1)

6 API Definitions [25](#api-definitions)

6.1 Nudr_DataRepository Service API [25](#nudr_datarepository-service-api)

6.1.1 API URI [25](#api-uri)

6.1.2 Usage of HTTP [25](#usage-of-http)

6.1.2.1 General [25](#general-13)

6.1.2.2 HTTP standard headers [26](#http-standard-headers)

6.1.2.2.1 General [26](#general-14)

6.1.2.2.2 Content type [26](#content-type)

6.1.2.2.3 Cache-Control [26](#cache-control)

6.1.2.2.4 ETag [26](#etag)

6.1.2.2.5 If-None-Match [26](#if-none-match)

6.1.2.2.5a If-Match [26](#a-if-match)

6.1.2.2.6 Last-Modified [26](#last-modified)

6.1.2.2.7 If-Modified-Since [27](#if-modified-since)

6.1.2.2.8 When to Use Entity-Tags and Last-Modified Dates [27](#when-to-use-entity-tags-and-last-modified-dates)

6.1.2.3 HTTP custom headers [27](#http-custom-headers)

6.1.2.3.1 General [27](#general-15)

6.1.2.3.2 3gpp-Sbi-Message-Priority [27](#gpp-sbi-message-priority)

6.1.2.3.3 3gpp-Sbi-Notification-Correlation [27](#gpp-sbi-notification-correlation)

6.1.2.3.4 3gpp-Sbi-Etags [27](#gpp-sbi-etags)

6.1.3 Resources [28](#resources)

6.1.3.1 Overview [28](#overview-1)

6.1.3.2 SubscriptionData [29](#subscriptiondata)

6.1.3.3 PolicyData [29](#policydata)

6.1.3.4 StructuredDataForExposure [29](#structureddataforexposure)

6.1.3.5 ApplicationData [29](#applicationdata)

6.1.3.6 Resource: DataRestorationEvents [29](#resource-datarestorationevents)

6.1.3.6.1 Description [29](#description)

6.1.3.6.2 Resource Definition [29](#resource-definition)

6.1.3.6.3 Resource Standard Methods [29](#resource-standard-methods)

6.1.4 Custom Operations without associated resources [30](#custom-operations-without-associated-resources)

6.1.5 Notifications [30](#notifications)

6.1.5.1 General [30](#general-16)

6.1.5.2 Data Change Notification [30](#data-change-notification)

6.1.5.3 Data Restoration Notification [30](#data-restoration-notification)

6.1.5a Data Model [32](#a-data-model)

6.1.5a.1 General [32](#a.1-general)

6.1.5a.2 Structured data types [32](#a.2-structured-data-types)

6.1.5a.2.1 Introduction [32](#a.2.1-introduction)

6.1.5a.2.2 Type: DataRestorationNotification [33](#a.2.2-type-datarestorationnotification)

6.1.6 Error Handling [33](#error-handling)

6.1.7 Security [34](#security)

6.1.8 Feature negotiation [38](#feature-negotiation)

6.2 Nudr_GroupIDmap Service API [43](#nudr_groupidmap-service-api)

6.2.1 API URI [43](#api-uri-1)

6.2.2 Usage of HTTP [43](#usage-of-http-1)

6.2.2.1 General [43](#general-17)

6.2.2.2 HTTP standard headers [43](#http-standard-headers-1)

6.2.2.2.1 General [43](#general-18)

6.2.2.2.2 Content type [43](#content-type-1)

6.2.2.2.3 Cache-Control [43](#cache-control-1)

6.2.2.2.4 ETag [43](#etag-1)

6.2.2.2.5 If-None-Match [44](#if-none-match-1)

6.2.2.2.6 Last-Modified [44](#last-modified-1)

6.2.2.2.7 If-Modified-Since [44](#if-modified-since-1)

6.2.2.2.8 When to Use Entity-Tags and Last-Modified Dates [44](#when-to-use-entity-tags-and-last-modified-dates-1)

6.2.2.3 HTTP custom headers [44](#http-custom-headers-1)

6.2.2.3.1 General [44](#general-19)

6.2.3 Resources [44](#resources-1)

6.2.3.1 Overview [44](#overview-2)

6.2.3.2 Resource NfGroupIds [45](#resource-nfgroupids)

6.2.3.2.1 Description [45](#description-1)

6.2.3.2.2 Resource Definition [45](#resource-definition-1)

6.2.3.2.3 Resource Standard Methods [45](#resource-standard-methods-1)

6.2.3.3 Resource RoutingIds [46](#resource-routingids)

6.2.3.3.1 Description [46](#description-2)

6.2.3.3.2 Resource Definition [46](#resource-definition-2)

6.2.3.3.3 Resource Standard Methods [46](#resource-standard-methods-2)

6.2.3.4 Resource Subscriptions [47](#resource-subscriptions)

6.2.3.4.1 Description [47](#description-3)

6.2.3.4.2 Resource Definition [47](#resource-definition-3)

6.2.3.4.3 Resource Standard Methods [47](#resource-standard-methods-3)

6.2.3.5 Resource IndividualSubscription [47](#resource-individualsubscription)

6.2.3.5.1 Description [47](#description-4)

6.2.3.5.2 Resource Definition [47](#resource-definition-4)

6.2.3.5.3 Resource Standard Methods [48](#resource-standard-methods-4)

6.2.4 Custom Operations without associated resources [49](#custom-operations-without-associated-resources-1)

6.2.5 Notifications [49](#notifications-1)

6.2.5.1 General [49](#general-20)

6.2.5.2 Data Change Notification [49](#data-change-notification-1)

6.2.6 Data Model [50](#data-model)

6.2.6.1 General [50](#general-21)

6.2.6.2 Structured data types [51](#structured-data-types)

6.2.6.2.1 Introduction [51](#introduction-3)

6.2.6.2.2 void [51](#void)

6.2.6.2.3 Type: RoutingIdResult [51](#type-routingidresult)

6.2.6.2.4 Type: SubscriptionData [51](#type-subscriptiondata)

6.2.6.2.5 Type: GroupIdMapNotify [52](#type-groupidmapnotify)

6.2.6.3 Simple data types and enumerations [52](#simple-data-types-and-enumerations)

6.2.6.3.1 Introduction [52](#introduction-4)

6.2.6.3.2 Simple data types [52](#simple-data-types)

6.2.7 Error Handling [53](#error-handling-1)

6.2.7.1 General [53](#general-22)

6.2.7.2 Protocol Errors [53](#protocol-errors)

6.2.7.3 Application Errors [53](#application-errors)

6.2.8 Security [53](#security-1)

6.2.9 Feature Negotiation [53](#feature-negotiation-1)

Annex A (normative): OpenAPI specification [54](#annex-a-normative-openapi-specification)

A.1 General [54](#a.1-general-1)

A.2 Nudr_DataRepository API [54](#a.2-nudr_datarepository-api)

A.3 Nudr_GroupIDmap API [61](#a.3-nudr_groupidmap-api)

Annex B (Normative): ABNF grammar for 3GPP SBI HTTP custom headers [68](#annex-b-normative-abnf-grammar-for-3gpp-sbi-http-custom-headers)

B.1 General [68](#b.1-general)

B.2 ABNF definitions (Filename: "TS29504_CustomHeaders.abnf") [68](#b.2-abnf-definitions-filename-ts29504_customheaders.abnf)

Annex C (informative): Change history [70](#annex-c-informative-change-history)
