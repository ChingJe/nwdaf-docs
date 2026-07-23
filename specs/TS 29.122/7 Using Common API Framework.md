---
spec: TS 29.122
version: 18.10.0
release: '18'
clause: '7'
title: 7 Using Common API Framework
source_archive: 29122-ia0.zip
source_document: 29122-ia0.doc
source_archive_sha256: bea81e48dfe77bc42deb87684c82fd69b8ac18a0eb1c9f9632f61f094ba95ecf
source_document_sha256: 6bd67a5c13b682bafa1d4b5bb9e860f8d1635bd8829e8a2fc200656ba7a61635
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 7 Using Common API Framework

## 7.1 General

When CAPIF is used with the SCEF, the SCEF shall support the following as defined in 3GPP TS 29.222 \[48\]:

\- the API exposing function and related APIs over CAPIF-2/2e and CAPIF-3/3e reference points;

\- the API publishing function and related APIs over CAPIF-4/4e reference point;

\- the API management function and related APIs over CAPIF-5/5e reference point; and

\- at least one of the the security methods for authentication and authorization, and related security mechanisms.

In a centralized deployment as defined in 3GPP TS 23.222 \[47\], where the CAPIF core function and API provider domain functions are co-located, the interactions between the CAPIF core function and API provider domain functions may be independent of CAPIF-3/3e, CAPIF-4/4e and CAPIF-5/5e reference points.

When CAPIF is used with the SCEF, the SCEF shall register all the features for northbound APIs in the CAPIF Core Function.

## 7.2 Security

When CAPIF is used for external exposure, before invoking the API exposed by the SCEF, the SCS/AS as API invoker shall negotiate the security method (PKI, TLS-PSK or OAUTH2) with CAPIF core function and ensure the SCEF has enough credential to authenticate the SCS/AS (see 3GPP TS 29.222 \[48\], clause 5.6.2.2 and clause 6.2.2.2).

If PKI or TLS-PSK is used as the selected security method between the AF and the NEF, upon API invocation, the NEF shall retrieve the authorization information from the CAPIF core function as described in 3GPP TS 29.222 \[48\], clause 5.6.2.4.

As indicated in 3GPP TS 33.122 \[53\], the access to the T8 APIs may be authorized by means of the OAuth2 protocol (see IETF RFC 6749 \[51\]), using the "Client Credentials" authorization grant, where the CAPIF core function (see 3GPP TS 29.222 \[48\]) plays the role of the authorization server.

NOTE 1: In this release, only "Client Credentials" authorization grant is supported.

If OAuth2 is used as the selected security method between the SCS/AS and the SCEF, the SCS/AS, prior to consuming services offered by the T8 APIs, shall obtain a "token" from the authorization server, by invoking the Obtain_Authorization service, as described in 3GPP TS 29.222 \[48\], clause 5.6.2.3.2.

The T8 APIs do not define any scopes for OAuth2 authorization. It is the SCEF responsibility to check whether the SCS/AS is authorized to use an API based on the "token". Once the SCEF verifies the "token", it shall check whether the NEF identifier in the "token" matches its own published identifier, and whether the API name in the "token" matches its own published API name. If those checks are passed, the AF has full authority to access any resource or operation for the invoked API.

NOTE 2: The security requirement in the current clause does not apply for the MsisdnLessMoSms API since it is the SCEF initiated interaction with the SCS/AS. How the security scheme works for the MsisdnLessMoSms API is left to configuration.

NOTE 3: For aforementioned security methods, the SCEF needs to apply admission control according to access control policies after performing the authorization checks.
