---
spec: TS 24.547
version: 19.0.0
release: '19'
clause: 5
title: 5 Functional entities
source_archive: 24547-j00.zip
source_document: 24547-j00.docx
content_origin: 3gpp-source
---

# 5 Functional entities


## 5.1 SEAL identity management client (SIM-C)

The SIM-C is a functional entity that acts as the application client for VAL user identity related transactions.

To be compliant with the HTTP procedures in the present document the SIM-C shall:

\- support the user authentication procedure specified in clause 6.2.2; and

\- support the token exchange procedure specified in clause 6.2.3.

To be compliant with the CoAP procedures in the present document the SIM-C:

\- shall support the role of CoAP client as specified in IETF RFC 7252 \[17\];

\- should support CoAP over TCP and Websocket as specified in IETF RFC 8323 \[18\];

\- shall support IETF RFC 9200 \[19\];

\- shall support OSCORE profile of IETF RFC 9203 \[21\];

\- should support DTLS profile of IETF RFC 9202 \[20\]; and

\- shall support the procedures in clause 6.2.2.

## 5.2 SEAL identity management server (SIM-S)

The SIM-S is a functional entity that authenticates the VAL user's identity by verifying the credentials provided by the VAL user.

To be compliant with the procedures in the present document the SIM-S shall:

\- support the user authentication procedure specified in clause 6.2.2; and

\- support the token exchange procedure specified in clause 6.2.3.

To be compliant with the CoAP procedures in the present document the SIM-S:

\- shall support the role of CoAP server as specified in IETF RFC 7252 \[17\];

\- should support CoAP over TCP and Websocket as specified in IETF RFC 8323 \[18\];

\- shall support IETF RFC 9200 \[19\];

\- shall support OSCORE profile of IETF RFC 9203 \[21\];

\- should support DTLS profile of IETF RFC 9202 \[20\]; and

\- shall support the procedures in clause 6.2.2.
