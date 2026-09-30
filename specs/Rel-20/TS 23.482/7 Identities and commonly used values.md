---
spec: TS 23.482
version: 20.3.0
release: '20'
clause: 7
title: 7 Identities and commonly used values
source_archive: 23482-k30.zip
source_document: 23482-k30.docx
content_origin: 3gpp-source
---

# 7 Identities and commonly used values


## 7.1 General

The common identities for SEAL and ADAES refer to 3GPP TS 23.434 \[5\] and 3GPP TS 23.436 \[4\] respectively. The following clauses list the additional identities and commonly used values for AIMLE Service.

## 7.2 AIMLE server ID

The AIMLE server ID uniquely identifies the AIML enablement server.

## 7.3 AIMLE client ID

The AIMLE client ID uniquely identifies the AIML enablement client.

## 7.4 ML repository ID

The ML repository ID uniquely identifies the ML-related repository function, which is used for storing the ML models and ML related information, as well as for serving as registry for the ML / FL members.

## 7.5 ML model ID

The ML model ID uniquely identifies the application-layer ML model.

## 7.6 FL member ID

The FL member ID uniquely identifies the participant entity in a Federated Learning process which is supported by AIMLE service, e.g., AIMLE server ID, VAL server ID or EAS ID.

## 7.7 AIMLE service area

The AIMLE service area is the area where the AIML Enablement server owner provides its AIML support services. The AIMLE service area can be expressed as a Topological Service Area (e.g., a list of TA), a Geographical Service Area (e.g., geographical coordinates) or both.

## 7.8 ML model profile ID

The ML model profile ID uniquely identifies a ML model profile that is created by the ML repository as a result of a successful ML model storage request by a AIMLE server.

## 7.9 AIMLE client set ID

The AIMLE client set ID is a unique identifier assigned by an AIMLE server that is associated with the set of AIMLE clients that have been selected and have agreed to participate in performing AI/ML related operations, e.g., ML model training.
