# NWDAF implementation-related 3GPP Release 18 specifications

This corpus is organized for progressive loading by development agents. All listed specifications are now included in full.

## Specification map

| Specification | Version | Included scope | Primary use |
|---|---:|---|---|
| [TS 23.288](TS%2023.288/README.md) | 18.13.0 | Full | NWDAF architecture, analytics, ML and data-repository procedures |
| [TS 23.501](TS%2023.501/README.md) | 18.12.0 | Full | 5GS architecture, NF roles, selection and storage architecture |
| [TS 23.502](TS%2023.502/README.md) | 18.14.0 | Full | 5GS Stage 2 procedures, including full registration, session, exposure and policy flows |
| [TS 29.500](TS%2029.500/README.md) | 18.10.0 | Full | Common SBA HTTP behaviour, headers, errors and routing |
| [TS 29.508](TS%2029.508/README.md) | 18.11.0 | Full | SMF Session Management Event Exposure API |
| [TS 29.510](TS%2029.510/README.md) | 18.11.0 | Full | NRF management, discovery, access token and bootstrapping APIs |
| [TS 29.520](TS%2029.520/README.md) | 18.14.0 | Full | NWDAF Stage 3 APIs and data models |
| [TS 29.564](TS%2029.564/README.md) | 18.7.0 | Full | UPF Event Exposure and UE private IP/identifier APIs |
| [TS 29.571](TS%2029.571/README.md) | 18.12.0 | Full | Common SBI data types and ProblemDetails |
| [TS 29.575](TS%2029.575/README.md) | 18.11.0 | Full | ADRF data/analytics and ML model management APIs |
| [Official OpenAPI YAML](openapi/README.md) | Package-specific | Exact attachments | Machine-readable API paths and schemas |

## Common workflow: NWDAF registration and NF discovery

1. Read TS 23.288 clauses 5.1 and 5.2 for NWDAF-specific profile and discovery semantics.
2. Read TS 23.501 for the architecture and NF selection principles.
3. Read TS 23.502 clauses 4.17 and 5.2.7 for the Stage 2 lifecycle and discovery flow.
4. Read TS 29.510 and its exact OpenAPI YAML for registration, heartbeat, update, deregistration and discovery.
5. Resolve shared data types through TS 29.571 and TS29571_CommonData.yaml.
6. Consult TS 29.500 for common SBI HTTP behaviour.

## Common workflow: SMF/UPF data collection

1. Use TS 23.288 for the NWDAF data-collection procedure and analytics-specific conditions.
2. Use TS 23.502 clauses 4.3, 4.4 and 4.15.4 for session, N4 and internal event-exposure context.
3. Use TS 29.508 and TS29508_Nsmf_EventExposure.yaml for SMF subscriptions and notifications.
4. Use TS 29.564 and its two YAML attachments for direct UPF service exposure.
5. Use TS 23.501 to understand SMF/UPF selection, serving scope and user-plane architecture.

## Common workflow: ADRF integration

1. Use TS 23.288 clauses describing ADRF storage, retrieval, DataSetTag and ML-model procedures.
2. Use TS 23.501 for ADRF architecture, discovery and selection.
3. Use TS 29.510 to discover the ADRF and select its service endpoint.
4. Use TS 29.575 and its exact YAML attachments for Nadrf_DataManagement and Nadrf_MLModelManagement.
5. Use TS 29.571 and TS 29.500 for common schemas and HTTP behaviour.

## Fidelity boundary

- Clause wording is extracted deterministically from the supplied Word documents and is not rewritten by an LLM.
- Generated navigation is explicitly marked.
- Exact YAML and ABNF package attachments are retained byte-for-byte.
- Diagrams link to PNG previews where bounded rendering succeeded; original EMF/WMF and embedded OLE/Visio payloads are retained for every source image.
- Legacy DOC conversion and vector-rendering limitations are documented under [`_validation/`](_validation/README.md).
- For high-stakes interpretation, the supplied original Word document remains the final reference.
