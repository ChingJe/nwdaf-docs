---
spec: TS 29.510
version: 18.11.0
release: '18'
clause: Annex B
title: 'Annex B (normative): NF Profile changes in NFRegister and NFUpdate responses'
source_archive: 29510-ib0.zip
source_document: 29510-ib0.docx
source_archive_sha256: 84e5ccfacace52d488cdca8f0ee1cb3b6817d390ee9903dc5e52105d58c6a1e6
source_document_sha256: 01abf6c9240ec101eef77e3ffc4bab5aca41dfd0d6d02c7515e02364cea887d9
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex B (normative): NF Profile changes in NFRegister and NFUpdate responses


## B.1 General


In the NFRegister and NFUpdate (NF Profile Complete Replacement and NF Profile Partial Update) service operations, a NF Service Consumer may indicate to the NRF that it supports receiving NF Profile changes in the response from the NRF, by including the nfProfileChangesSupportInd and/or the nfProfilePartialUpdateChangesSupportInd attributes set to "true" in the NFProfile it registers to or replaces in the NRF.

NOTE 1: For NFRegister and NFUpdate (NF Profile Complete Replacement), the NF Service Consumer can indicate its support of the corresponding capability during the initial NFRegister operation or during the NF Profile Complete Replacement.

NOTE 2: For NF Profile Partial Update (which uses the HTTP PATCH operation), the NF Service Consumer can indicate its support of the corresponding capability during the initial NFRegister operation, or during an NF Profile Complete Replacement (i.e., in the content of the corresponding HTTP PUT request), and can also indicate support of this capability after the initial registration, in a PATCH request, by setting to true the nfProfilePartialUpdateChangesSupportInd attribute.

The NRF may return NF Profile changes, instead of the complete NF Profile, in NFRegister or NFUpdate responses, if the NF Service Consumer has indicated corresponding support in its NFProfile data. When doing so, the NRF shall include in the NF Profile returned in the response:

\- attributes that are mandatory to include in the NF Profile; if an optional IE is included (e.g. nfServices), attributes that are mandatory to include in this optional IE (e.g. serviceInstanceId) shall also be included;

\- optional or conditional IEs that have been changed or added by the NRF; and

\- the nfProfileChangesInd IE set to "true", indicating that the returned profile contains NF profile changes.

EXAMPLE 1: The NRF does not change the NF Profile received in the request.

The NRF response contains a NFProfile with just the following IEs:

\- nfInstanceId, nfType, nfStatus; and

\- nfProfileChangesInd IE set to "true".

EXAMPLE 2: The NRF modifies or adds the heartbeatTimer attribute to the NF Profile received in the request.

The NRF response contains a NFProfile with just the following IEs:

\- nfInstanceId, nfType, nfStatus;

\- heartbeatTimer with NRF chosen value;

\- nfProfileChangesInd IE set to "true".
