---
spec: TS 23.502
version: 18.14.0
release: '18'
clause: Annex A
title: 'Annex A (informative): Drafting rules and conventions for NF services'
source_archive: 23502-ie0.zip
source_document: 23502-ie0.docx
source_archive_sha256: 0f24045c49ddf710fb91e51ec74d070cfd13d1e9bafc07809d7e87eae6a61ce8
source_document_sha256: 89536df6095fba3b9c90f1fa06ceaec918dfc6b0fd26a769a06eae7db66bb603
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex A (informative): Drafting rules and conventions for NF services

## A.1 General

This informative Annex provides drafting rules and conventions followed in this technical specification (and TS 23.501 \[2\]) for the definition of NF services offered over the service-based interfaces.

## A.2 Naming

### A.2.1 Service naming

Each NF service provided by a service-based interface shall be named and referred to according to the following nomenclature:

*- Nnfname_ServiceName*, where *Nnfname* is the service-based interface where the NF service is invoked. See clause 4.2.5 of TS 23.501 \[2\] for the list of service-based interfaces in the 5GS Architecture.

Example (illustrative): *Namf_Registration.*

### A.2.2 Service operation naming

If a service contains multiple independent operations, each operation shall be named and referred to according to the following nomenclature:

*- Nnfname_ServiceName_ServiceOperation\[Method\]*, where the *ServiceName* represents the actual NF service. The *ServiceOperation* itself defines the available service functionality which can be addressed by a specific operation. The *Method(s)* is/are the action(s), how the ServiceOperation can be used. It can be created, read, updated or deleted.

Example (illustrative): *Namf_Session_Registration\[Create\]*, *Namf_Session_Registration\[Delete\]*

In general, this operation naming structure for the given example is depicted in a tree-structure diagram:

![](assets/original/image267.emf)

Figure A.2.2-1: Service Operation Naming and its Methods

## A.3 Representation in an information flow

Invoking a service or service operation within an information flow is represented using a disaggregated representation (see figure A.3-1).

The disaggregated representations on figure A.3-1 shall be used as follows:

\- The \<step\> represents the actual step number in the information flow e.g. ″7.″.

\- Representation a) shall be used when the step is required.

\- Representation b) shall be used when the step is optional or conditional.

![](assets/original/image268.emf)

Figure A.3-1: Disaggregated representation of a NF service or service operation in information flows

NOTE: Depending on the information flow, the order of NF Producer and NF Consumer can be reversed.

## A.4 Reference to services and service operations in procedures

Whenever a procedure needs to refer to the service or service operation of a service-based interface, the naming in clause A.2 shall be used, using italic font. Unless otherwise obvious in the text, the NF Consumer of the service or service operation shall be indicated within parenthesis after the service or service operation name.

\- \<Nnfname_ServiceName\<\_OperationName\>\> (\<NF Name Consumer\>)

Example: e.g. *Namf_Registration_RelocationRequest (AMF)*

## A.5 Service and service operation description template

The description of a service or service operation in this specification shall be done according to the following template.

NOTE: The heading level should follow that of the actual clause where the service is specified.

**X.x \<*Nnfname*\_ServiceName\<\_OperationName\>\>**

**X.x.1 Description**

**Service or service operation name:** \<Nnfname_ServiceName\<\_OperationName\>\>.

**Description:** \<short descriptive text\>.

**Known NF Consumers:** \<list of NFs\>.

**Inputs, Required:** \<list of parameters\> *-- Parameters required from NF Consumer for successful completion of the service or service operation. Parameters required for the operation of the underlying protocol shall not be listed.*

**Inputs, Optional:** \<list of parameters\> *-- Additional parameters that may be provided by NF Consumer for execution of the service or service operation. Parameters required for the operation of the underlying protocol shall not be listed.*

**Outputs, Required:** \<list of parameters\>, \< Nnfname_ServiceNameX\<\_OperationNameY \>\>, \<Other\> *-- Parameters provided to NF Consumer and/or service triggered upon successful completion of the service and/or other (e.g. procedure triggered). Parameters required for the operation of the underlying protocol shall not be listed.*

**Outputs, Optional:** \<list of parameters\> -- *Additional parameters provided to NF Consumer upon successful completion of the service or service operation. Parameters required for the operation of the underlying protocol shall not be listed.*

**X.x.2 Service/service operation information flow**

\<Information flow of the service or service operation offered by NF Producer to NF Consumer over the NF Producer service-based interface\>.

NOTE: This information flow can require invoking other services. In this case, the invoked services are represented as described in clause A.3.

## A.6 Design Guidelines for NF services

Clause 7.1.1 of TS 23.501 \[2\] defines the criteria for defining the NF services. The following clauses identify the design guidelines that shall be considered for identifying the NF services.

### A.6.1 Self-Containment

The following design guidelines are used for identifying self-contained NF services.

\- Each NF service operates on its own set of context(s). A context refers to a state or a software resource or an internal data storage. The NF service operations can create, read, update or delete the context(s).

\- Any direct access of a context(s) owned by a NF service is be made by the service operations of that NF service. Services provided by the same NF can communicate internally within the NF.

### A.6.2 Reusability

The following design guidelines are used for specifying NF services to be reusable.

\- NF service operations are specified such that other NF can potentially invoke them in future, if required.

\- The service operations may be usable in multiple system procedures specified in clause 4 of this specification.

\- Using clause 4 of the current document, the system procedures in which the NF service operations can be used are considered and based on that the parameters for the NF service operations are clearly listed.

NOTE: It is possible that, when mapping an end to end call flow to service based architecture, one step in the call flow may map to multiple NF service operation invocations. This specification clearly identifies each NF service operation invocation in the call flow. Protocol optimization of multiple NF service operation invocations are left for TS 29.500 \[17\] consideration.

### A.6.3 Use Independent Management Schemes

The mechanisms for independent management schemes are not in scope of this specification.
