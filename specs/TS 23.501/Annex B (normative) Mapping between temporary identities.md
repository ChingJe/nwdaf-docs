---
spec: TS 23.501
version: 18.12.0
release: '18'
clause: Annex B
title: 'Annex B (normative): Mapping between temporary identities'
source_archive: 23501-ic0.zip
source_document: 23501-ic0.docx
source_archive_sha256: 800fb58a2a3a8c3eeac7084497de566fc730808923d8eea61b84d2555360debe
source_document_sha256: cdab3840e4d278d381f70806e6458f87dff8e5fd238f370e4f65872bbd5088a6
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex B (normative): Mapping between temporary identities

When interworking procedures with N26 are used and the UE performs idle mode mobility from 5GC to EPC the following mapping from 5G GUTI to EPS GUTI applies:

\- 5G \<MCC\> maps to EPS \<MCC\>

\- 5G \<MNC\> maps to EPS \<MNC\>

\- 5G \<AMF Region ID\> and 5G \<AMF Set ID\> maps to EPS \<MMEGI\> and part of EPS \<MMEC\>

\- 5G \<AMF Pointer\> map to part of EPS \<MMEC\>

\- 5G \<5G-TMSI\> maps to EPS \<M-TMSI\>

NOTE 1: The mapping described above does not necessarily imply the same size for the 5G GUTI and EPS GUTI fields that are mapped. The size of 5G GUTI fields and other mapping details will be defined in TS 23.003 \[19\].

NOTE 2: To support interworking with the legacy EPC core network entity (i.e. when MME is not updated to support interworking with 5GS), it is assumed that the 5G \<AMF Region ID\> and EPS \<MMEGI\> is partitioned to avoid overlapping values in order to enable discovery of source node (i.e. MME or AMF) without ambiguity. Once the EPS in the PLMN has been updated to support interworking with 5GS, the full address space of the AMF Region ID can be used for 5GS.
