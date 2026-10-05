---
spec: TS 29.549
version: 20.1.0
release: '20'
clause: 8
title: 8 Using Common API Framework
source_archive: 29549-k10.zip
source_document: '29549-k10_0_cover.docx, 29549-k10_1_Main-Body_s00_s06.docx, 29549-k10_2_Main-Body_s07_s09.docx, 29549-k10_3_Annexes_sA_sHistory.docx'
content_origin: 3gpp-source
---

# 8 Using Common API Framework


## 8.1 General

When CAPIF is used with a SEAL service, the SEAL server shall support the following as defined in 3GPP TS 29.222 \[16\]:

\- the API exposing function and related APIs over CAPIF-2/2e and CAPIF-3/3e reference points;

\- the API publishing function and related APIs over CAPIF-4/4e reference point;

\- the API management function and related APIs over CAPIF-5/5e reference point; and

\- at least one of the security methods for authentication and authorization, and related security mechanisms.

In a centralized deployment as defined in 3GPP TS 23.222 \[17\], where the CAPIF core function and API provider domain functions are co-located, the interactions between the CAPIF core function and API provider domain functions may be independent of CAPIF-3/3e, CAPIF-4/4e and CAPIF-5/5e reference points.

When CAPIF is used with a SEAL service, the SEAL server shall register all the features for northbound APIs in the CAPIF Core Function.

## 8.2 Security

When CAPIF is used for managing the exposure of the SEAL APIs, before invoking an API exposed by the SEAL Server, the service consumer (e.g., VAL Server), acting as an API Invoker, shall negotiate the security method (PKI, TLS-PSK or OAUTH2) with CAPIF Core Function and ensure that the SEAL Server has enough credentials to authenticate the service consumer as defined in clauses 5.6.2.2 and 6.2.2.2 3GPP TS 29.222 \[16\].

If PKI or TLS-PSK is used as the selected security method between the service consumer and the SEAL Server, upon API invocation, the SEAL Server shall retrieve the authorization information from the CAPIF Core Function as described in clause 5.6.2.4 of 3GPP TS 29.222 \[16\].

As indicated in 3GPP TS 33.122 \[18\], the access to the SEAL APIs may be authorized by means of the OAuth2 protocol (see IETF RFC 6749 \[19\]), where the CAPIF Core Function (see 3GPP TS 29.222 \[16\]) plays the role of the authorization server.

If OAuth2 is used as the selected security method between the service consumer and the SEAL Server, the service consumer, prior to consuming services offered by the SEAL APIs, shall obtain a "token" from the authorization server, by invoking the Obtain_Authorization service, as described in clause 5.6.2.3.2 of 3GPP TS 29.222 \[16\].

The SEAL APIs do not define any scopes for OAuth2 authorization in the present specification. For the definition and handling of scopes for OAuth2 authorization in CAPIF, see 3GPP TS 29.222 \[16\].

It is the SEAL Server responsibility to check whether the service consumer is authorized to use an API based on the "token". Once the SEAL Server verifies the "token", it shall check whether the SEAL Server identifier in the "token" matches its own published identifier, whether the API name in the "token" matches its own published API name and whether the granted scope (see 3GPP TS 29.222 \[16\]) in the "token" is authorized. If those checks are passed, the service consumer has full authority to access any resource(s) and/or operation(s) for the invoked service API and that are within the limits of the granted scope in the "token".

NOTE: For the aforementioned security methods, the SEAL Server needs to apply admission control according to access control policies after performing the authorization checks.
