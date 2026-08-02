---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: '26'
title: 26 Numbering, addressing and identification for Mission Critical Services
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 26 Numbering, addressing and identification for Mission Critical Services

## 26.1 Introduction

This clause describes the format of the parameters used for Mission Critical Services.

For further information on the use of the parameters see TS 23.280 \[114\].

## 26.2 Domain name for MC services confidentiality protection of MC services identities

A Domain Name for MC Services confidentiality protection used in a host part of a SIP URI indicates that the user part of the SIP URI contains a confidentiality protected MC Services identity. This Domain Name shall be the string "mc1-encrypted.3gppnetwork.org".

Protected MCPTT identities are constructed according to TS 24.379 \[111\].

Protected MCData identities are constructed according to TS 24.282 \[116\].

Protected MCVideo identities are constructed according to TS 24.281 \[115\].
