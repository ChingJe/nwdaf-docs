---
spec: TS 23.501
version: 18.12.0
release: '18'
clause: Annex C
title: 'Annex C (informative): Guidelines and Principles for Compute-Storage Separation'
source_archive: 23501-ic0.zip
source_document: 23501-ic0.docx
source_archive_sha256: 800fb58a2a3a8c3eeac7084497de566fc730808923d8eea61b84d2555360debe
source_document_sha256: cdab3840e4d278d381f70806e6458f87dff8e5fd238f370e4f65872bbd5088a6
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex C (informative): Guidelines and Principles for Compute-Storage Separation

5G System Architecture allows any NF/NF Service to store and retrieve its unstructured data (e.g. UE contexts) into/from a Storage entity (e.g. UDSF) as stated in clause 4.2.5 in this release of the specification. This clause highlights some assumptions, principles regarding NF/NF services that use this Storage entity for storing unstructured data:

1\. It is up to the Network Function implementation to determine whether the Storage entity is used as a Primary Storage (in which case the corresponding context stored within the NF/NF Service is deleted after storage in the Storage entity) or the Storage entity is used as a Secondary Storage (in which case the corresponding context within the NF/NF Service is stored).

2\. It is up to the NF/NF Service implementation to determine the trigger (e.g. at the end of Registration procedure, Service Request procedure etc) for storing unstructured data (e.g. UE contexts) in the Storage entity but it is a good practice for NF/NF service to store stable state in the Storage entity.

3\. Multiple NF/NF service instances may require to access the same stored data in the Storage entity (e.g. UE context), around the same time, then the resolution the race condition is implementation specific.

4\. All NFs within the same NF Set are assumed to have access to the same unstructured data stored within the Storage entity.

5\. AMF planned removal with UDSF (clause 5.21.2.2.1) and AMF auto-recovery (with UDSF option in clause 5.21.2.3) assume that a storage entity/UDSF is used either as a primary storage or secondary storage by the AMF for storing UE contexts.

6\. It is up to implementation of the Storage entity to make sure that only NFs that are authorized for a certain data record can access this data record.
