---
spec: TS 29.222
version: 20.1.0
release: '20'
clause: 10
title: 10 Security
source_archive: 29222-k10.zip
source_document: 29222-k10.docx
content_origin: 3gpp-source
---

# 10 Security


## 10.1 General

Security methods for CAPIF are specified in 3GPP TS 33.122 \[16\].

## 10.2 CAPIF-1/1e security

Secure communication between API invoker and CAPIF core function over CAPIF-1/1e reference points, using a TLS protocol based connection is defined in 3GPP TS 33.122 \[16\].

For Onboard_API_Invoker service operation of the CAPIF_API_Invoker_Management_API, the TLS protocol based connection shall be established using server certificate as defined in 3GPP TS 33.122 \[16\].

For rest of the CAPIF APIs, the TLS protocol based connection shall be established with certificate based mutual authentication as defined in 3GPP TS 33.122 \[16\].

## 10.3 CAPIF-2/2e security and securely invoking service APIs

For secure communication between API invoker and API exposing function and ensuring secure invocations of service APIs, the API invoker:

\- shall negotiate the security method with the CAPIF core function using the Obtain_Security_Method service operation of the CAPIF_Security_API;

\- shall initiate the authentication with the API exposing function using the Initiate_Authentication service operation of the AEF_Security_API; and

\- shall establish a secure connection with the API exposing function as defined in 3GPP TS 33.122 \[16\], using the method negotiated with the CAPIF core function.
