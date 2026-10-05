---
spec: TS 33.434
version: 19.0.0
release: '19'
clause: 4
title: 4 SEAL security requirements
source_archive: 33434-j00.zip
source_document: 33434-j00.docx
content_origin: 3gpp-source
---

# 4 SEAL security requirements


## 4.1 VAL user authentication and authorization

\[SEAL-SEC-4.1-a\] All users of the VAL Service shall be authenticated.

\[SEAL-SEC-4.1-b\] The VAL Client and the VAL Server shall mutually authenticate each other prior to providing the VAL UE with the VAL Service User profile and access to user-specific services.

\[SEAL-SEC-4.1-c\] The transmission of configuration data and user profile data between an authorized VAL server in the network and the VAL UE shall be confidentiality protected, integrity protected and protected from replays.

\[SEAL-SEC-4.1-d\] The VAL service should take measures to detect and mitigate DoS attacks to minimize the impact on the network and on VAL users.

\[SEAL-SEC-4.1-e\] The VAL service shall provide a means to support confidentiality of VAL user identities.

\[SEAL-SEC-4.1-f\] The VAL service shall provide a means to support confidentiality of VAL signalling.

## 4.2 Inter-domain

\[SEAL-SEC-4.2-a\] VAL systems should take measures to protect themselves from external attacks at the system border.
