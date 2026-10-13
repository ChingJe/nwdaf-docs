---
spec: TR 23.700-82
version: 19.1.0
release: '19'
clause: 3
title: '3 Definitions of terms, symbols and abbreviations'
source_archive: 23700-82-j10.zip
source_document: 23700-82-j10.docx
content_origin: 3gpp-source
---

# 3 Definitions of terms, symbols and abbreviations


## 3.1 Terms

For the purposes of the present document, the terms given in 3GPP TR 21.905 \[1\] and the following apply. A term defined in the present document takes precedence over the definition of the same term, if any, in 3GPP TR 21.905 \[1\].

**ML model**: According to 3GPP TS 28.105 \[9\], mathematical algorithm that can be "trained" by data and human expert input as examples to replicate a decision an expert would make when provided that same information.

**ML model lifecycle**: The lifecycle of an ML model includes data collection, data processing, model training, model verification, model, instantiation and deployment, model monitoring and termination of ML model components.

**ML model training**: According to 3GPP TS 28.105 \[9\], ML model training includes capabilities of an ML training function or service to take data, run it through an ML model, derive the associated loss and adjust the parameterization of that ML model based on the computed loss.

**ML model inference**: According to 3GPP TS 28.105 \[9\], ML model training includes capabilities of an ML model inference function that employs an ML model and/or AI decision entity to conduct inference.

**AI/ML enablement**: an application enablement framework consisting of one or more AI/ML enabler capabilities based on implementation. Such function can be deployed as (or within) an enablement layer server (e.g. SEAL or ADAES) or client.

**AI/ML client**: an application layer entity (also referred as ML client) which is an AI/ML endpoint, and performs client-side operations (e.g. related to the ML model lifecycle). Such AI/ML client can be a VAL client or AIML enabler client and may be configured e.g. to provide ML model training and inference locally e.g. at the VAL UE side.

**AI/ML server**: an application layer entity which is an AI/ML endpoint, and performs server-side operations (e.g. related to the ML model lifecycle). Such AI/ML server can be a VAL server or AIML enabler server.

**FL member**: An FL member or participant is an entity which has a role in the FL process. An FL member can be an FL client performing ML model training, or an FL server performing aggregation/collaboration for the FL process. The FL member in this study is assumed to be either a functionality at the VAL UE or at the network/server side (AIML enablement server, VAL server).

**FL client**: An FL member which locally trains the ML model as requested by the FL server.

**FL server**: An FL member which generates global ML model by aggregating local model information from FL clients.

**VAL client:** According to 3GPP TS 23.434 \[7\], VAL client is an entity that provides the client side functionalities corresponding to the vertical applications.

**VAL server:** According to 3GPP TS 23.434 \[7\], VAL server is a generic name for the server application function of a specific VAL service.

## 3.2 Symbols

For the purposes of the present document, the following symbols apply:

\<symbol\> \<Explanation\>

## 3.3 Abbreviations

For the purposes of the present document, the abbreviations given in 3GPP TR 21.905 \[1\] and the following apply. An abbreviation defined in the present document takes precedence over the definition of the same abbreviation, if any, in 3GPP TR 21.905 \[1\].

ADAES Application Data Analytics Enablement Server

A-ADRF Application layer - Analytical Data Repository Function

A-DCCF Application layer - Data Collection and Coordination Function

AIMLE AI/ML enablement AnLF Analytics logical function

ASP Application Service Provider

A-DCCF Application layer - Data Collection and Coordination Function

MDAS Management Domain Analytics Service

MFAF Messaging Framework Adaptor Function

MTLF Model Training logical function

MTME ML Model Training and Management Entity

MLT ML training Function

NEF Network Exposure Function

NWDAF Network Data Analytics Function

OAM Operation, Administration and Maintenance

SEAL Service Enabler Architecture Layer

SEALDD SEAL Data Delivery

VFL Vertical Federated Learning

VAL Vertical Application Layer
