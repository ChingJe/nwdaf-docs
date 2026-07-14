# NWDAF and NRF implementation-related 3GPP Release 18 specifications

This corpus is organized for progressive loading by development agents.

## Specification map

| Specification | Version | Included scope | Primary use |
|---|---:|---|---|
| [TS 23.288](TS%2023.288/README.md) | 18.13.0 | Full | NWDAF architecture, NF Profile semantics and analytics procedures |
| [TS 29.520](TS%2029.520/README.md) | 18.14.0 | Full | NWDAF Stage 3 APIs and data models |
| [TS 23.502](TS%2023.502/README.md) | 18.14.0 | NRF-related clauses | NF registration, update, deregistration, discovery and NRF service operations |
| [TS 29.510](TS%2029.510/README.md) | 18.11.0 | Full | Nnrf_NFManagement, Nnrf_NFDiscovery, Nnrf_AccessToken and Nnrf_Bootstrapping |
| [TS 29.571](TS%2029.571/README.md) | 18.12.0 | Full | Common SBI schemas, patch items and ProblemDetails |
| [TS 29.500](TS%2029.500/README.md) | 18.10.0 | Full | Common SBA HTTP behavior, headers, errors, retries and routing |
| [Official OpenAPI YAML](openapi/README.md) | Package-specific | Exact attachments | Machine-readable API paths and schemas |

## Common workflow: NWDAF registration and discovery

1. Read TS 23.288 clauses 5.1 and 5.2 for the NWDAF-specific profile and discovery semantics.
2. Read TS 23.502 clauses 4.17 and 5.2.7 for the Stage 2 lifecycle and service-operation flow.
3. Read TS 29.510 clauses 5 and 6 for Stage 3 resource methods and data models.
4. Use `TS29510_Nnrf_NFManagement.yaml` and `TS29510_Nnrf_NFDiscovery.yaml` for exact schemas.
5. Resolve shared types through TS 29.571 and `TS29571_CommonData.yaml`.
6. Consult TS 29.500 for generic HTTP headers, status codes, retries, indirect communication and routing behavior.
7. Use TS 29.510 Nnrf_AccessToken and its YAML when OAuth2 authorization is enabled.

## Fidelity boundary

- Clause wording is extracted deterministically from the supplied Word documents and is not rewritten by an LLM.
- Generated navigation is explicitly marked.
- Exact YAML and ABNF package attachments are retained byte-for-byte.
- Diagrams use PNG previews while original vector media and, for full specifications, embedded OLE/Visio files are retained.
- TS 23.502 is intentionally partial; its README lists the included clauses.
- See [`_validation/README.md`](_validation/README.md) for validation results and limitations.
