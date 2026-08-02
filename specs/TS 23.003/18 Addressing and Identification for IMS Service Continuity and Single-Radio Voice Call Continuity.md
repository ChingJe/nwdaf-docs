---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: '18'
title: 18 Addressing and Identification for IMS Service Continuity and Single-Radio Voice Call Continuity
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 18 Addressing and Identification for IMS Service Continuity and Single-Radio Voice Call Continuity

## 18.1 Introduction

This clause describes the format of the parameters needed for the support of IMS Service Continuity. For further information on the use of the parameters see TS 23.237 \[71\] and also TS 23.292 \[70\].

## 18.2 CS Domain Routeing Number (CSRN)

A CS Domain Routeing Number (CSRN) is a number that is used to route a call from the IM CN subsystem to the user in the CS domain. The structure is as defined in clause 3.4.

## 18.3 IP Multimedia Routeing Number (IMRN)

An IP Multimedia Routeing Number (IMRN) is a routable number that points to the IM CN subsystem. In a roaming scenario, the IMRN has the same structure as an international ISDN number (see clause 3.4). The Tel URI format of the IMRN (see IETF RFC 3966 \[45\]) is treated as a PSI (see clause 13.5) within the IM CN subsystem.

## 18.4 Session Transfer Number (STN)

A Session Transfer Number (STN) is a public telecommunication number, as defined by ITU-T Recommendation E.164 \[10\] and is used by the UE to request Session Transfer of the media path from PS to CS access.

## 18.5 Session Transfer Identifier (STI)

A Session Transfer Identifier (STI) is a SIP URI or SIP dialogue ID (see IETF RFC 3261 \[26\] for more information) and is used by the UE to request Session Transfer of a media path.

## 18.6 Session Transfer Number for Single Radio Voice Call Continuity (STN-SR)

The Session Transfer Number for Single Radio Voice Call Continuity (STN-SR) is a public telecommunication number, as defined by ITU-T Recommendation E.164 \[10\] and is used by the MSC Server to request session transfer of the media path from the PS domain to CS domain.

## 18.7 Correlation MSISDN

A Correlation MSISDN (C-MSISDN) is an MSISDN (see clause 3.3) that is used for correlation of sessions at access transfer and to route a call from the IM CN subsystem to the same user in the CS domain. The C-MSISDN is equal to the MSISDN or the basic MSISDN if multinumbering option is used (see TS 23.008 \[2\], clause 2.1.3) of the CS access. Any MSISDN of a user that can be used for TS11 (telephony) in the CS domain which is not shared by more than one IMS Private Identity in an IMS CN subsystem, can serve as the user's C-MSISDN.

The C-MSISDN is bound to the IMS Private User Identity and is uniquely assigned per IMSI and IMS Private User Identity.

If A-MSISDN is available it shall be used as the C-MSISDN. For the definition of A-MSISDN refer to clause 18.9.

## 18.8 Transfer Identifier for CS to PS Single Radio Voice Call Continuity (STI-rSR)

A Session Transfer Identifier for CS to PS Single Radio Voice Call Continuity (STI-rSR) is a SIP URI (see IETF RFC 3261 \[26\] for more information) and is used by the UE to request access transfer of a media path.

## 18.9 Additional MSISDN

An Additional MSISDN (A-MSISDN) is an MSISDN (see clause 3.3) that is assigned to a user with PS subscription in addition to the already assigned MSISDN(s).

The structure of an A-MSISDN should follow the structure of an MSISDN number as defined in clause 3.3.

The A-MSISDN shall be able to be used for TS11 (telephony) in the CS domain and shall be uniquely assigned per IMSI.
