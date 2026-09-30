---
spec: TS 23.482
version: 20.3.0
release: '20'
clause: 3
title: '3 Definitions of terms, symbols and abbreviations'
source_archive: 23482-k30.zip
source_document: 23482-k30.docx
content_origin: 3gpp-source
---

# 3 Definitions of terms, symbols and abbreviations


## 3.1 Terms

For the purposes of the present document, the terms given in 3GPP TR 21.905 \[1\] and the following apply. A term defined in the present document takes precedence over the definition of the same term, if any, in 3GPP TR 21.905 \[1\].

**AIMLE client set identifier:** an identifier of the set of selected AIMLE clients.

**AI/ML intermediate model**: For federated learning, members need to train models for multiple rounds, intermediate models indicate the model which do not meet the required training rounds and/or meet the requirements of the federation training.

**AI/ML operation:** also known as AI/ML task, it refers to the specific execution step in AI/ML lifecycle, it can include data management, model training etc.

**AIMLE service**: An AIMLE service is an AIMLE capability which aims assisting in performing or enabling one or more AIML operations.

**FL client**: An FL member which locally trains the ML model as requested by the FL server. Such FL client functionality can be at the network (e.g. AIMLE server with FL client capability) or at the device side (e.g. AIMLE client with FL client capability).

**FL member**: An FL member or participant is an entity which has a role in the FL process. An FL member can be an FL client performing ML model training, or an FL server performing aggregation/collaboration for the FL process.

**FL server**: An FL member which generates global ML model by aggregating local model information from FL clients.

**ML model**: According to 3GPP TS 28.105 \[8\], mathematical algorithm that can be "trained" by data and human expert input as examples to replicate a decision an expert would make when provided that same information.

**ML model inference**: According to 3GPP TS 28.105 \[8\], ML model inference includes capabilities of an ML model inference function that employs an ML model and/or AI decision entity to conduct inference.

**ML model lifecycle**: The lifecycle of an ML model aka ML model operational workflow consists of a sequence of ML operations for a given ML task / job (such job can be an analytics task or a VAL automation task). This definition is aligned with the 3GPP definition on ML model lifecycle according to 3GPP TS 28.105 \[8\].

**ML model training**: According to 3GPP TS 28.105 \[8\], ML model training includes capabilities of an ML training function or service to take data, run it through an ML model, derive the associated loss and adjust the parameterization of that ML model based on the computed loss.

**Split AI/ML operation pipeline:** A Split AI/ML operation pipeline is a workflow for ML model inference in which AI/ML endpoints are organized and collaborate to process ML models in sequential stages, where processing at each stage involves ML model inference on the output of the previous stage.

**Vertical Federated Learning (VFL):** A federated learning technique in which multiple VFL members jointly perform training and inference on a common sample space with different feature sets, without exchanging raw data.

**VFL process**: A process of performing training or inference using the VFL technique.

**VFL member:** An entity that takes part in a VFL process, either as a VFL server or a VFL client.

**VFL server:** A VFL member that coordinates the VFL process by aggregating results from VFL clients and propagating updates to them during training.

**VFL client:** A VFL member that performs local training and inference, sends results to the VFL server, and updates local parameters based on information received from the VFL server.

**VFL sample alignment:** A stage in a VFL process during which samples are aligned among VFL members, ensuring that training and inference are performed consistently across members on the same sample space.

## 3.2 Symbols

For the purposes of the present document, the following symbols apply:

\<symbol\> \<Explanation\>

## 3.3 Abbreviations

For the purposes of the present document, the abbreviations given in 3GPP TR 21.905 \[1\] and the following apply. An abbreviation defined in the present document takes precedence over the definition of the same abbreviation, if any, in 3GPP TR 21.905 \[1\].

ADAES Application Data Analytics Enablement Server

AIMLE AI/ML enablement

ASP Application Service Provider

FL Federated Learning

NEF Network Exposure Function

NWDAF Network Data Analytics Function

OAM Operation, Administration and Maintenance

SEAL Service Enabler Architecture Layer

SEALDD SEAL Data Delivery

VAL Vertical Application Layer

VFL Vertical Federated Learning
