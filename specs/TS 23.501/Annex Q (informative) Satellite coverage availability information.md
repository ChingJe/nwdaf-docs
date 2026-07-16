---
spec: TS 23.501
version: 18.12.0
release: '18'
clause: Annex Q
title: 'Annex Q (informative): Satellite coverage availability information'
source_archive: 23501-ic0.zip
source_document: 23501-ic0.docx
source_archive_sha256: 800fb58a2a3a8c3eeac7084497de566fc730808923d8eea61b84d2555360debe
source_document_sha256: cdab3840e4d278d381f70806e6458f87dff8e5fd238f370e4f65872bbd5088a6
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex Q (informative): Satellite coverage availability information

The protocol and format of satellite coverage availability information to be provisioned to the UE via PDU session or SMS is not defined in this release of the specification, but this annex provides some examples on the information that constitutes input to the source of satellite coverage availability information e.g. external server and the output it provides to the UE. Satellite coverage availability information can be indicated to the UE by indications corresponding to whether or not coverage is available for a specific satellite RAT Type for a particular location and time, where:

\- These indications can be Boolean "True" (e.g. coverage available) and "False" (coverage not available);

\- locations can correspond to grid points in a fixed array (e.g. rectangular, hexagonal);

\- Coverage availability times may occur at fixed periodic intervals; and

\- Coverage availability information is per RAT Type. The information provisioned to the UE can include coverage information on only one PLMN or multiple PLMNs.

If Satellite coverage availability information indicates coverage is available then additional information on whether PLMN is allowed to operate in that location can be provided to the UE.

In order for the source of satellite coverage availability information to provide accurate information to the UE, a UE might indicate for example the following information to a source of satellite coverage availability information (e.g. an external server):

\- Serving PLMN ID (if not already known or implied).

\- One or more satellite RAT Types (where satellite coverage availability information is then expected for these one or more RAT Types).

\- List of supported satellite frequency bands (if not implied by the particular RAT Types).

\- Present UE location (e.g. latitude and longitude) for a reference grid point (e.g. the most Southerly and then most Westerly grid point).

\- Type of Array (e.g. rectangular or hexagonal).

\- Minimum elevation angle.

Based on the above information provided by the UE, satellite coverage availability information could be delivered to the UE as a sequence of time durations for each grid point where each time duration includes an indication of coverage availability or unavailability one example of many alternatives as illustrated below for a particular grid point with N different durations:

Satellite coverage availability information at a given grid point = \<N\> \<Binary 0 or 1\>\<Duration 1\> \<Binary 0 or 1\>\<Duration 2\> . . . . \<Binary 0 or 1\>\<Duration N\>

The above would be concatenated for all of the grid points to produce the satellite coverage availability information.

When SMS is used to deliver the satellite coverage availability information, the UE input and satellite coverage availability information output can be delivered in a series of concatenated SMS messages using possibly the same format.
