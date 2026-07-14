---
spec: TS 29.500
version: 18.10.0
release: '18'
clause: '4'
title: 4 Service Based Architecture Overview
source_archive: 29500-ia0.zip
source_document: 29500-ia0.docx
source_archive_sha256: 3e0cbb6bc6ceec4dabb597410fb4b55ba03ae82e9b242972f441cd926971f5f2
source_document_sha256: 8cd4994735a1718db905b0aee4ceba4fb479cc7193d82d051a68450f33eba52b
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 4 Service Based Architecture Overview


## 4.1 NF Services


3GPP TS 23.501 \[3\] defines the 5G System Architecture as a Service Based Architecture, i.e. a system architecture in which the system functionality is achieved by a set of NFs providing services to other authorized NFs to access their services.

Control Plane (CP) Network Functions in the 5G System architecture shall be based on the service based architecture.

A NF service is one type of capability exposed by a NF (NF Service Producer) to other authorized NF (NF Service Consumer) through a service based interface. A NF service may support one or more NF service operation(s). See clause 7 of 3GPP TS 23.501 \[3\].

Network Functions may offer different functionalities and thus different NF services. Each of the NF services offered by a Network Function shall be self-contained, acted upon and managed independently from other NF services offered by the same Network Function (e.g. for scaling, healing).

## 4.2 Service Based Interfaces


A service based interface represents how the set of services is provided or exposed by a given NF. This is the interface where the NF service operations are invoked.

The service based Control Plane interfaces within the 5G Core Network are specified in 3GPP TS 23.501 \[3\].

## 4.3 NF Service Framework


### 4.3.1 General


The Service Based Architecture shall support the NF Service Framework that enable the use of NF services as specified in clause 7.1 of 3GPP TS 23.501 \[3\].

The NF Service Framework includes the following mechanisms:

\- NF service registration and de-registration: to make the NRF aware of the available NF instances and supported services (see clause 7.1.5 of 3GPP TS 23.501 \[3\]);

\- NF service discovery: to enable a NF Service Consumer to discover NF Service Producer instance(s) which provide the expected NF service(s) (see clause 7.1.3 of 3GPP TS 23.501 \[3\]);

\- NF service authorization: to ensure the NF Service Consumer is authorized to access the NF service provided by the NF Service Producer (see clause 7.1.4 of 3GPP TS 23.501 \[3\]);

\- Inter service communication: NF Service Consumers and NF Service Producers may communicate directly or indirectly via a Service Communication Proxy (SCP). Whether a NF uses Direct Communication or Indirect Communication via an SCP is based on configuration of the NF.

The stage 3 procedures for NF service registration and de-registration, NF service discovery and NF service authorization are defined in 3GPP TS 29.510 \[8\].

### 4.3.2 NF Service Advertisement URI


When invoking a service operation of a NF Service Producer that use HTTP methods with a message body (i.e. PUT, POST and PATCH), the NF Service Consumer may provide NF Service Advertisement URI(s) in the service operation request, based on operator policy, if it expects that the NF Service Producer may subsequently consume NF service(s) which the NF Service Consumer can provide (as a NF Service Producer).

When receiving NF Service Advertisement URI(s) in a service operation request, the NF Service Producer may store and use the Service Advertisement URI(s) to discover NF services produced by the NF Service Consumer in subsequent procedures, based on operator policy.

The NF Service Advertisement URI identifies the nfInstance resource(s) in the NRF which are registered by NF Service Producer(s).

An example of NF Service Advertisement URI could be represented as:

"{apiRoot}/nnrf-disc/nf-instances?nfInstanceId={nfInstanceId}".

NOTE: The NF Service Advertisement URI can be used e.g. when different NRFs are deployed in the PLMN.

When applicable, the NF Service Advertisement URI(s) shall be carried in HTTP message body.
