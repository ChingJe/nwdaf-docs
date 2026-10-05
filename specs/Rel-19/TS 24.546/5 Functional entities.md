---
spec: TS 24.546
version: 19.5.0
release: '19'
clause: 5
title: 5 Functional entities
source_archive: 24546-j50.zip
source_document: 24546-j50.docx
content_origin: 3gpp-source
---

# 5 Functional entities


## 5.1 SEAL configuration management client (SCM-C)

The SCM-C functional entity acts as the application client for configuration related transactions.

To be compliant with the HTTP procedures in the present document the SCM-C:

\- shall support the role of XCAP client as specified in IETF RFC 4825 \[3\];

\- shall support the role of XDMC as specified in OMA OMA-TS-XDM_Core-V2_1 \[4\];

\- shall support the procedures in clause 6.2.2;

\- shall support the procedures in clause 6.2.3;

\- shall support the procedures in clause 6.2.4;and

\- shall support the procedures in clause 6.2.5.

To be compliant with the CoAP procedures in the present document the SCM-C:

\- shall support the role of CoAP client as specified in IETF RFC 7252 \[12\];

\- shall support the capability to observe resources as specified in IETF RFC 7641 \[14\];

\- shall support the block-wise transfer as specified in IETF RFC 7959 \[13\];

\- may support the robust block transfer as specified in IETF  RFC 9177  \[21\];

\- should support CoAP over TCP and Websocket as specified in IETF RFC 8323 \[15\];

\- shall support CBOR encoding as specified in IETF RFC 8949 \[17\];

\- shall support the procedures in clause 6.2.2;

\- shall support the procedures in clause 6.2.3;

\- shall support the procedures in clause 6.2.4;and

\- shall support the procedures in clause 6.2.5

NOTE 1: The security mechanism to be supported for the CoAP procedures is described in 3GPP TS 24.547 \[5\].

NOTE 2: Support for TCP for the CoAP procedures is required if the client connects over the network which blocks or impedes the use of UDP, e.g. when NATs are present in the communication path.

NOTE 3: The CoAP protocol supports mechanism for reliable message exchange over UDP. Use of TCP can also be beneficial if reliable transport is required for other reasons, e.g. better observability of resources. Usage of CoAP over TCP is an implementation choice.

NOTE 4: Support for the robust block transfer mechanism for the CoAP procedures is beneficial in environments where packet loss is highly asymmetrical and where performance optimization of block transfers is required.

## 5.2 SEAL configuration management server (SCM-S)

The SCM-S is a functional entity used to configure one or more vertical applications with 3GPP system related vertical applications provisioning information and configure data on the SEAL configuration management client.

To be compliant with the HTTP procedures in the present document the SCM-S:

\- shall support the role of XCAP server as specified in IETF RFC 4825 \[3\];

\- shall support the role of XDMS as specified in OMA OMA-TS-XDM_Core-V2_1 \[4\];

\- shall support the procedures in clause 6.2.2;

\- shall support the procedures in clause 6.2.3;

\- shall support the procedures in clause 6.2.4;

\- shall support the procedures in clause 6.2.5.

To be compliant with the CoAP procedures in the present document the SCM-C:

\- shall support the role of CoAP server as specified in IETF RFC 7252 \[12\];

\- shall support the capability to observer resources as specified in IETF RFC 7641 \[14\];

\- shall support the block-wise transfer as specified in IETF RFC 7959 \[13\];

\- shall support the robust block transfer as specified in IETF RFC 9177 \[21\];

\- shall support CoAP over TCP and Websocket as specified in IETF RFC 8323 \[15\];

\- shall support CBOR encoding as specified in IETF RFC 8949 \[17\];

\- shall support the procedures in clause 6.2.2;

\- shall support the procedures in clause 6.2.3;

\- shall support the procedures in clause 6.2.4;and

\- shall support the procedures in clause 6.2.5.

NOTE: The security mechanism to be supported for the CoAP procedures is described in 3GPP TS 24.547 \[5\]
