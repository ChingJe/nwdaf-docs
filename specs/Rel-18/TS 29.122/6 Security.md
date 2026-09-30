---
spec: TS 29.122
version: 18.10.0
release: '18'
clause: '6'
title: 6 Security
source_archive: 29122-ia0.zip
source_document: 29122-ia0.doc
content_origin: 3gpp-source
---

# 6 Security

TLS shall be used to support the security communication between the SCEF and the SCS/AS over T8 as defined in clause 5.5 of 3GPP TS 33.187 \[35\]. The access to the SCEF northbound APIs shall be authorized by means of OAuth2 protocol (see IETF RFC 6749 \[51\]), based on local configuration, using the "Client Credentials" authorization grant. If OAuth2 is used, a client, prior to consuming services offered by the SCEF Northbound APIs, shall obtain a "token" from the authorization server.
