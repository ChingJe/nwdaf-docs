# NWDAF implementation-related 3GPP Release 18 specifications

This corpus is organized for progressive loading by development agents. All listed specifications are included in full.

## Specification map

| Specification | Version | Included scope | Primary use |
|---|---:|---|---|
| [TS 23.003](TS%2023.003/README.md) | 18.8.0 | Full | Common numbering, addressing and identifier semantics for SUPI, SUCI, GPSI, PEI, PLMN, DNN and 5GS identifiers |
| [TS 23.288](TS%2023.288/README.md) | 18.13.0 | Full | NWDAF architecture, analytics, ML and data-repository procedures |
| [TS 23.501](TS%2023.501/README.md) | 18.12.0 | Full | 5GS architecture, NF roles, selection and storage architecture |
| [TS 23.502](TS%2023.502/README.md) | 18.14.0 | Full | 5GS Stage 2 procedures, including full registration, session, exposure and policy flows |
| [TS 29.122](TS%2029.122/README.md) | 18.10.0 | Full | T8 northbound APIs and shared schemas reused by NWDAF, NRF, SMF and ADRF OpenAPI definitions |
| [TS 29.500](TS%2029.500/README.md) | 18.10.0 | Full | Common SBA HTTP behaviour, headers, errors and routing |
| [TS 29.502](TS%2029.502/README.md) | 18.11.0 | Full | SMF PDU Session and SM Context lifecycle APIs, procedures, multipart messages and data models |
| [TS 29.503](TS%2029.503/README.md) | 18.13.0 | Full | UDM subscriber data, UE context, event exposure, authentication and identifier services |
| [TS 29.504](TS%2029.504/README.md) | 18.13.0 | Full | UDR data repository and group-ID mapping services for subscription, policy, exposure and application data |
| [TS 29.508](TS%2029.508/README.md) | 18.11.0 | Full | SMF Session Management Event Exposure API |
| [TS 29.510](TS%2029.510/README.md) | 18.11.0 | Full | NRF management, discovery, access token and bootstrapping APIs |
| [TS 29.518](TS%2029.518/README.md) | 18.14.0 | Full | AMF communication, event exposure, location, mobility and messaging services |
| [TS 29.520](TS%2029.520/README.md) | 18.14.0 | Full | NWDAF Stage 3 APIs and data models |
| [TS 29.523](TS%2029.523/README.md) | 18.7.0 | Full | PCF policy-control event subscriptions, notifications and event data models |
| [TS 29.552](TS%2029.552/README.md) | 18.7.0 | Full | NWDAF Stage 3 signalling flows across analytics, data collection, ML, ADRF, DCCF and MFAF procedures |
| [TS 29.554](TS%2029.554/README.md) | 18.3.0 | Full | PCF background-data-transfer policy control API and shared BDT policy types |
| [TS 29.564](TS%2029.564/README.md) | 18.7.0 | Full | UPF Event Exposure and UE private IP/identifier APIs |
| [TS 29.571](TS%2029.571/README.md) | 18.12.0 | Full | Common SBI data types and ProblemDetails |
| [TS 29.574](TS%2029.574/README.md) | 18.12.0 | Full | DCCF data subscription, notification, fetch, transfer and data-collection-profile APIs |
| [TS 29.575](TS%2029.575/README.md) | 18.11.0 | Full | ADRF data/analytics and ML model management APIs |
| [TS 29.576](TS%2029.576/README.md) | 18.8.0 | Full | MFAF configuration, fetch and notification services for messaging-framework-based data delivery |

## Common workflow: identifiers, NF profiles and API fields

1. Use TS 23.003 for the normative composition and semantics of SUPI, SUCI, GPSI, PEI, PLMN, DNN and 5GS identifiers.
2. Use TS 29.571 and exact CommonData YAML for machine-readable shared representations.
3. Use TS 29.510 NFProfile definitions when those identifiers appear in registration or discovery.

## Common workflow: NWDAF registration and NF discovery

1. Read TS 23.288 clauses 5.1 and 5.2 for NWDAF-specific profile and discovery semantics.
2. Read TS 23.501 for architecture and NF selection principles.
3. Read TS 23.502 clauses 4.17 and 5.2.7 for the Stage 2 lifecycle and discovery flow.
4. Read TS 29.510 and its exact OpenAPI YAML for registration, heartbeat, update, deregistration and discovery.
5. Resolve identifiers through TS 23.003 and shared data types through TS 29.571 and TS 29.122.
6. Consult TS 29.500 for common SBI HTTP behaviour.

## Common workflow: SMF, PDU Session and UPF data collection

1. Use TS 23.288 for NWDAF data-collection procedures and analytics-specific conditions.
2. Use TS 23.502 clauses 4.3, 4.4 and 4.15.4 for PDU Session, N4 and event-exposure context.
3. Use TS 29.502 and TS29502_Nsmf_PDUSession.yaml for SM Context and PDU Session lifecycle details.
4. Use TS 29.508 and TS29508_Nsmf_EventExposure.yaml for SMF subscriptions and notifications.
5. Use TS 29.564 and its exact YAML attachments for direct UPF service exposure.

## Common workflow: UDM and UDR data access

1. Use TS 29.503 for UDM subscriber-data, UE-context and event-exposure services.
2. Use TS 29.504 and its exact YAML attachments for the UDR data repository and group-ID mapping APIs.
3. Use TS 23.502 for the surrounding subscriber-data and NF interaction procedures.
4. Use TS 23.003 for identifier semantics and TS 29.571 for shared API representations.

## Common workflow: ADRF, DCCF and messaging-framework integration

1. Use TS 23.288 for ADRF storage, retrieval, DataSetTag and ML-model procedures.
2. Use TS 23.501 for ADRF architecture, discovery and selection.
3. Use TS 29.575 for ADRF data and ML-model APIs.
4. Use TS 29.574 for DCCF coordination and TS 29.576 for MFAF delivery.
5. Use TS 29.552 to trace concrete Stage 3 signalling sequences.

## Fidelity boundary

- Clause wording is extracted deterministically from the supplied Word documents and is not rewritten by an LLM.
- Generated navigation is explicitly marked.
- Exact YAML and ABNF package attachments are retained byte-for-byte.
- Diagrams link to PNG previews where conversion succeeds; original EMF/WMF and embedded OLE/Visio payloads are retained.
- Conversion and validation limitations are documented under [`_validation/`](_validation/README.md).
- For high-stakes interpretation, the supplied original Word document remains the final reference.
