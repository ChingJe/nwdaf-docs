---
spec: TS 29.222
version: 20.1.0
release: '20'
clause: 4
title: 4 Overview
source_archive: 29222-k10.zip
source_document: 29222-k10.docx
content_origin: 3gpp-source
---

# 4 Overview


## 4.1 Introduction

In 3GPP, there are multiple northbound API-related specifications. To avoid duplication and inconsistency of approaches between different API specifications and to specify common services (e.g. authorization), 3GPP has considered in 3GPP TS 23.222 \[2\] the development of a common API framework (CAPIF) that includes common aspects applicable to any northbound service APIs.

The present document specifies the APIs needed to support CAPIF.

## 4.2 Service Architecture

3GPP TS 23.222 \[2\] clause 6 specifies the functional entities and domains of the functional model.

## 4.3 Functional Entities


### 4.3.1 API invoker

The API invoker is typically provided by a 3<sup>rd</sup> party application provider who has service agreement with PLMN operator or SNPN. The API invoker may reside within the same trust domain as the PLMN operator network or SNPN.

The API invoker supports several capabilities as defined in 3GPP TS 23.222 \[2\].

### 4.3.2 CAPIF core function

The CAPIF core function (CCF) supports the capabilities as defined in 3GPP TS 23.222 \[2\].

### 4.3.3 API exposing function

The API exposing function (AEF) is the provider of the Service APIs and is also the service communication entry point of the service API to the API invokers as defined in 3GPP TS 23.222 \[2\].

### 4.3.4 API publishing function

The API publishing function (APF) enables the API provider to publish the Service APIs information as defined in 3GPP TS 23.222 \[2\].

### 4.3.5 API management function

The API management function (AMF) enables the API provider to perform administration of the Service APIs. The API capabilities are defined in 3GPP TS 23.222 \[2\].
