---
spec: TS 24.560
version: 20.1.0
release: '20'
clause: 7
title: 7 Using common API framework
source_archive: 24560-k10.zip
source_document: 24560-k10.docx
content_origin: 3gpp-source
---

# 7 Using common API framework


## 7.1 General

When CAPIF is used with a AIMLE server service, the AIMLE server shall support the following functionalities as defined in 3GPP TS 29.222 \[6\]:

\- the API exposing function and the related APIs over CAPIF-2/2e and CAPIF-3/3e reference points;

\- the API publishing function and the related APIs over CAPIF-4/4e reference point;

\- the API management function and the related APIs over CAPIF-5/5e reference point; and

\- at least one of the security methods for authentication and authorization, and the related security mechanisms.

In a centralized deployment as defined in 3GPP TS 23.222 \[3\], where the CAPIF core function and the API provider domain functions are co-located, the interactions between the CAPIF core function and the API provider domain functions may be independent of the CAPIF-3/3e, CAPIF-4/4e and CAPIF-5/5e reference points.

When CAPIF is used with a AIMLE server service, the AIMLE server shall register all the northbound APIs features in the CAPIF core function.

## 7.2 Security

When CAPIF is used for external exposure, before invoking an API exposed by the AIMLE server, the service API consumer (e.g. AIMLE client) acting as an API invoker shall negotiate the security method (PKI, TLS-PSK or OAuth 2.0) with the CAPIF core function and ensure that the AIMLE server has enough credentials to authenticate the service API consumer (e.g. AIMLE client), as defined in clauses 5.6.2.2 and 6.2.2.2 of 3GPP TS 29.222 \[6\].

If PKI or TLS-PSK is selected as the security method to be used between the service API consumer (e.g. AIMLE client) and the AIMLE server, upon API invocation, the AIMLE server shall retrieve the authorization information from the CAPIF core function as described in clause 5.6.2.4 of 3GPP TS 29.222 \[6\].

As indicated in 3GPP TS 33.122 \[12\], the access to the AIMLE server APIs may be authorized by means of the OAuth 2.0 protocol (see IETF RFC 6749 \[13\]), using the "Client Credentials" authorization grant, where the CAPIF core function (see 3GPP TS 29.222 \[6\]) plays the role of the authorization server.

NOTE 1: In this release, only "Client Credentials" authorization grant is supported.

If OAuth 2.0 is selected as the security method to be used between the service API consumer (e.g. AIMLE client) and the AIMLE server, the service API consumer (e.g. AIMLE client) shall, prior to consuming the services offered by the AIMLE server APIs, obtain a "token" from the authorization server, by invoking the Obtain_Authorization service operation as described in clause 5.6.2.3.2 of 3GPP TS 29.222 \[6\].

The AIMLE server APIs do not define any scopes for OAuth 2.0 authorization. It is the AIMLE server responsibility to check whether the service API consumer (e.g. AIMLE client) is authorized to use an API based on the provided "token". Once the AIMLE server verifies the "token", it shall check whether the AIMLE server identifier in the "token" matches its own published identifier, and whether the API name in the "token" matches its own published API name. If those checks are passed, the service API consumer (e.g. AIMLE client) has full authority to access any resource or operation provided by the invoked API.

NOTE 2: For the aforementioned security methods, the AIMLE server needs to apply admission control according to access control policies after performing the authorization checks.
