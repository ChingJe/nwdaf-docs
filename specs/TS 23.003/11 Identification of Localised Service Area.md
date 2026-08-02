---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: '11'
title: 11 Identification of Localised Service Area
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 11 Identification of Localised Service Area

Cells may be grouped into specific localised service areas. Each localised service area is identified by a localised service area identity (LSA ID). No restrictions are placed on what cells may be grouped into a given localised service area.

The LSA ID can either be a PLMN significant number or a universal identity. This shall be known both in the networks and in the SIM.

The LSA ID consists of 24 bits, numbered from 0 to 23, with bit 0 being the LSB. Bit 0 indicates whether the LSA is a PLMN significant number or a universal LSA. If the bit is set to 0 the LSA is a PLMN significant number; if it is set to 1 it is a universal LSA.

The LSA ID shall be composed as shown in figure 19:

![](assets/rendered/image21.wmf)

Figure 19: Structure of LSA ID
