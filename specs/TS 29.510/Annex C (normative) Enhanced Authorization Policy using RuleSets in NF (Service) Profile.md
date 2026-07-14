---
spec: TS 29.510
version: 18.11.0
release: '18'
clause: Annex C
title: 'Annex C (normative): Enhanced Authorization Policy using RuleSets in NF (Service)
  Profile'
source_archive: 29510-ib0.zip
source_document: 29510-ib0.docx
source_archive_sha256: 84e5ccfacace52d488cdca8f0ee1cb3b6817d390ee9903dc5e52105d58c6a1e6
source_document_sha256: 01abf6c9240ec101eef77e3ffc4bab5aca41dfd0d6d02c7515e02364cea887d9
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex C (normative): Enhanced Authorization Policy using RuleSets in NF (Service) Profile


## C.1 General


When scope of authorizations allowed to NF-Service-Consumers of different PLMNs, S-NSSAIs, SNPNs, NF-Domains etc. are different, it is not always possible for an NF (Service) Producer to register an authorization profile into NRF using allowedXXX parameters alone. The Allowed-ruleset feature addresses such requirements by extending the authorization policy with a prioritized list of RuleSets in the NF (Service) profile.

This clause provides configuration examples and guidance on handling backward compatibility when the Allowed-ruleset feature is deployed.

## C.2 Examples of NF-Producer profile only using RuleSets (i.e. without AllowedXXX parameters) in NF (Service) Profile


This clause provides configuration examples of allowedScopesRuleSet parameter in the NF-Service profile (Clause 6.1.6.2.3).

Following is an example of rules formed by para-phrasing the individual RuleSets registered by an NF-Service-Producer:

> *priority 1 plmns \<\> nfTypes \<\> nfInstances \<\> scopes \<\> allow*
>
> *priority 2 plmns \<\> nfTypes \<\> scopes \<\> allow*
>
> *priority 3 plmns \<\> nfTypes \<\> allow*
>
> *priority 100 deny*

When a NF-Service-Consumer requests an access-token from NRF, the NRF matches the properties of the NF-Service-Consumer (PLMN, SNPN, nfType, NfDomain, S-NSSAIs, NF-Instance Id etc.) against these rules in decreasing order of priority (1 being the highest). <u>If a match is found, search stops, and the matching rule is applied to determine the scope to be granted.</u>

Consider 3 NF-Service Producers ***p1***, ***p2*** and ***p3*** who need to register their NF-Service Profile into NRF. Each of these producers define operation level scopes Op1, Op2 and Op3 and the service-level scope.

\- NF-Service-Producer ***p1*** allows all NF-Service-Consumers with nfType=A to access its resources unrestricted. However NF-Service-Consumers with nfType=B are limited to Op1 and Op2. It may register, in the NF-Service-Profile:

> *priority 2 nfType {B} scopes {Op1, Op2, service level scope} allow*
>
> *priority 5 nfType {A} allow*
>
> *priority 100 deny*

\- NF-Service-Producer ***p2*** allows all NF-Service-Consumers with nfType=A to access its resources unrestricted. NF-Service-Consumers with nfType=B are limited to Op1 and Op2. It additionally wants to allow NF-Service-Consumer with NF-Instance-ID=X to Op3 only. It may register, in the NF-Service-Profile:

> *priority 2 nfInstance-id {X} scopes {Op3, service level scope} allow*
>
> *priority 5 nfType {B} scopes {Op1, Op2, service level scope} allow*
>
> *priority 10 nfType {A} allow*
>
> *priority 100 deny*

\- NF-Service-Producer ***p3*** allows all NF-Service-Consumers of PLMN1 of nfType=A or nfType=B to access its resources unrestricted. However, for the NFs of PLMN2, NF-Service-Consumers of nfType=A are allowed to access its resources unrestricted, but the NF-Service-Consumers of nfType=B are limited to Op1 and Op2. It may register, in the NF-Service-Profile:

> *priority 2 plmn {plmn1} nfType {A,B} allow*
>
> *priority 5 plmn {plmn2} nfType {A} allow*
>
> *priority 10 plmn {plmn2} nfType {B} scopes {Op1, Op2, service level scope} allow*
>
> *priority 100 deny*

Absence of scopes in a rule indicates that all service operations/all scopes are allowed.

Rule with no identification of NF-Consumer (e.g. priority 100 rule in example above) indicates the rule applies to all.

Similar examples apply to allowedRuleSet parameter in NF-Profile (Clause 6.1.6.2.2), however without the scopes parameter, as in the case of NF-Profile, the allowedRuleSet parameter is used to determine whether an NF-Consumer is allowed or not allowed to access the NF-Producer.

## C.3 Example of NF-Producer profile using RuleSets and AllowedXXX parameters in NF (Service) Profile


When a NF (Service) producer registers both the AllowedXXX parameters and the allowedScopesRuleSet parameter in the NF-Service Profile, the authorization scopes assigned to an NF-Consumer are determined by performing a logical OR operation of the two sets of parameters. The helps implementations to use Allowed-ruleset feature only when the authorization policy cannot be configured using the AllowedXXX parameters alone.

This clause provides configuration examples with allowedScopesRuleSet parameter in the NF-Service profile (Clause 6.1.6.2.3) along with allowedXXX parameters.

Consider the example of NF-Service-Producer ***p3*** in Annex C.2, which allows all NF-Service-Consumers of PLMN1 of nfType=A or nfType=B to access its resources unrestricted. However, for the NFs of PLMN2, NF-Service-Consumers of nfType=A are allowed to access its resources unrestricted, but the NF-Service-Consumers of nfType=B are limited to Op1 and Op2. This can be achieved by configuring the rules as:

> *{*
>
> *allowedPLMNs = plmn1*
>
> *allowedNfTypes = A,B*
>
> *}*
>
> *OR*
>
> *{*
>
> *priority 5 plmn {plmn2} nfType {A} allow*
>
> *priority 10 plmn {plmn2} nfType {B} scopes {Op1, Op2, service level scope} allow*

                              }

Similar examples apply with allowedRuleSet parameter in NF-Profile (Clause 6.1.6.2.2), however without the scopes parameter, as in the case of NF-Profile, the allowedRuleSet parameter is used to determine whether an NF-Consumer is allowed or not allowed to access the NF-Producer.

## C.4 Backward Compatibility


This clause provides examples for addressing backward compatibility issues with the Allowed-ruleset feature.

Consider an NF-Producer ***p1*** supporting Allowed-ruleset feature which registers its following NF-Service profile into NRF utilizing the new allowedScopesRuleSet parameter:

> *priority 2 nfInstance-id {X} scopes {Op3, service level scope} allow*
>
> *priority 5 nfType {B} scopes {Op1, Op2, service level scope} allow*
>
> *priority 10 nfType {A} allow*
>
> *priority 15 nssais {1,2} scopes {Op1, service level scope} allow*
>
> *priority 100 deny*

Consider an NF-Consumer ***c1*** which does **<u>not</u>** support the Allowed-ruleset feature and discovers NF-Producer ***p1*** using Nnrf_NFDiscovery service.

\- If the highest priority rule matching NF-Consumer ***c1*** is priority 2, the NRF may include, in the Nnrf_NFDiscovery_Get response, parameter *allowedOperationsPerNfInstance*, containing applicable scopes.

\- If the highest priority rule matching NF-Consumer ***c1*** is priority 5, the NRF may include, in the Nnrf_NFDiscovery_Get response, parameter *allowedOperationsPerNfType*, containing applicable scopes.

\- If the highest priority rule matching NF-Consumer ***c1*** is priority 10, the NRF may not include, in the Nnrf_NFDiscovery_Get response, the parameters *allowedOperationsPerNfInstance* and *allowedOperationsPerNfType*.

\- If the highest priority rule matching NF-Consumer ***c1*** is priority 15, the NRF may include, in the Nnrf_NFDiscovery_Get response, any of the parameters *allowedOperationsPerNfInstance* or *allowedOperationsPerNfType*, containing applicable scopes.

If entities (e.g. SCP) of a network are configured to use Complete Profile Subscription or Complete Profile Discovery feature, the Allowed-ruleset feature shall only be deployed if all these entities in the network have been upgraded to support and use the Allowed-ruleset feature.

NF-Producers shall only be configured to use the Allowed-ruleset feature after NRFs in the network are upgraded and configured to use the Allowed-ruleset feature, as otherwise NF-Producers would need to translate RuleSets to existing service access control parameters (i.e. allowedPlmns, allowedSnpns, allowedNfTypes etc.) which may not always be possible.
