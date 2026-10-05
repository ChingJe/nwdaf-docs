---
spec: TS 29.558
version: 20.0.0
release: '20'
clause: 10
title: 10 Using Common API Framework
source_archive: 29558-k00.zip
source_document: 29558-k00.docx
content_origin: 3gpp-source
---

# 10 Using Common API Framework


## 10.1 General

EES may expose its services to EAS with support of CAPIF. Also, the EES may also re-expose the network capabilities of the 3GPP core network to the EAS(s) with support of CAPIF architecture, as specified in 3GPP TS 23.558 \[2\]. When CAPIF is used with EES services, the EES shall support the following as defined in 3GPP TS 29.222 \[17\]:

\- the API exposing function and related APIs over CAPIF-2/2e and CAPIF-3/3e reference points;

\- the API publishing function and related APIs over CAPIF-4/4e reference point;

\- the API management function and related APIs over CAPIF-5/5e reference point; and

\- at least one of the security methods for authentication and authorization, and related security mechanisms.

The EAS supports the role of API Invoker as specified in 3GPP TS 29.222 \[17\]. In a centralized deployment as defined in 3GPP TS 23.222 \[17\], where the CAPIF core function and API provider domain functions are co-located, the interactions between the CAPIF core function and API provider domain functions may be independent of CAPIF-3/3e, CAPIF-4/4e and CAPIF-5/5e reference points.

When CAPIF is used with an EES service, the EES shall register all the features for northbound APIs in the CAPIF Core Function.

The EAS may expose its services to other EAS(s) using CAPIF as specified in 3GPP TS 23.558 \[2\]. When CAPIF is used for exposure of EAS services to other EAS(s), the above procedure shall be applicable with the following differences:

\- the provisions related to the EES apply to the EAS;

\- the provisions related to the EAS apply to the other EAS(s) as consumer(s) of the exposed EAS service APIs.

## 10.2 Security

When CAPIF is used for managing the exposure of the EEL APIs, before invoking an API exposed by the EEL entities (e.g., EES, EAS, ECS), the service consumer (e.g., EAS), acting as an API Invoker, shall negotiate the security method (PKI, TLS-PSK or OAUTH2) with the CAPIF Core Function and ensure that the EEL entities (e.g., EES, ECS) have enough credential to authenticate the service consumer as defined in clauses 5.6.2.2 and 6.2.2.2 of 3GPP TS 29.222 \[17\].

If PKI or TLS-PSK is used as the selected security method between the service consumer and the EEL entities, upon API invocation, the EEL entities shall retrieve the authorization information from the CAPIF Core Function as described in clause 5.6.2.4 of 3GPP TS 29.222 \[17\].

As indicated in 3GPP TS 33.122 \[18\], the access to the EEL APIs may be authorized by means of the OAuth2 protocol (see IETF RFC 6749 \[19\]), where the CAPIF Core Function (see 3GPP TS 29.222 \[17\]) plays the role of the authorization server.

If OAuth2 is used as the selected security method between the service consumer and the EEL entities, then the service consumer, prior to consuming services offered by the APIs exposed by the EEL entities, shall obtain a "token" from the authorization server, by invoking the Obtain_Authorization service, as described in clause 5.6.2.3.2 of 3GPP TS 29.222 \[17\].

The EES APIs do not define any scopes for OAuth2 authorization in the present specification. For the definition and handling of scopes for OAuth2 authorization in CAPIF, see 3GPP TS 29.222 \[17\].

It is the EEL entities responsibility to check whether the service consumer is authorized to use an API based on the provided "token". Once the EEL entities verifies the "token", it shall check whether the EEL entities identifier in the "token" matches its own published identifier, whether the API name in the "token" matches its own published API name and whether the granted scope (see 3GPP TS 29.222 \[17\]) in the "token" is authorized. If those checks are passed, the service consumer has full authority to access any resource(s) and/or operation(s) for the invoked service API and that are within the limits of the granted scope in the "token".

NOTE: For the aforementioned security methods, the EEL entities need to apply admission control according to access control policies after performing the authorization checks.
