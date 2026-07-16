---
spec: TS 23.501
version: 18.12.0
release: '18'
clause: Annex I
title: 'Annex I (normative): TSN usage guidelines'
source_archive: 23501-ic0.zip
source_document: 23501-ic0.docx
source_archive_sha256: 800fb58a2a3a8c3eeac7084497de566fc730808923d8eea61b84d2555360debe
source_document_sha256: cdab3840e4d278d381f70806e6458f87dff8e5fd238f370e4f65872bbd5088a6
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex I (normative): TSN usage guidelines

## I.1 Determination of traffic pattern information

As described in clause 5.27.2, the calculation of the TSCAI relies upon mapping of information for the TSN stream(s) based upon certain IEEE standard information.

Additional traffic pattern parameters such as maximum burst size and maximum flow bitrate can be mapped to MDBV and GFBR.

The traffic pattern parameter determination based on PSFP (IEEE Std 802.1Q \[98\]), when available, is as follows:

\- Periodicity of a TSN stream is set equal to StreamGateAdminCycleTime if there is only one StreamGateControlEntry with a StreamGateStatesValue set to Open in the StreamGateAdminControlList. If there is more than one StreamGateGateControlEntry with a StreamGateStatesValue set to Open in the StreamGateAdminControlList, then the Periodicity of the TSN Stream is set equal to sum of the timeIntervalValues from the first gate open instance to a next gate open instance in the StreamGateAdminControlList. For aggregated TSN streams with same periodicity and compatible Burst Arrival Times, the periodicity of the aggregated flow of these TSN Streams is set equal to StreamGateAdminCycleTime received from CNC for one of the TSN streams that are aggregated.

NOTE 1: Given that only TSN streams that have the same periodicity and compatible Burst Arrival Time can be aggregated, the StreamGateAdminCycleTime for those TSN streams is assumed to be the same.

\- Burst Arrival time of a TSN stream at the ingress port is determined based on the following conditions:

\- The Burst Arrival Time of a TSN Stream should be set to StreamGateAdminBaseTime plus the sum of the timeIntervalValues for which the StreamGateStatesValue is Closed in the StreamGateAdminControlList until the first gate open time (i.e. until StreamGateStatesValue set to Open is found). If the StreamGateStatesValue is Open for the first timeIntervalValue, then the Burst Arrival time is set to StreamGateAdminBaseTime. For aggregated TSN streams, the arrival time is calculated similarly, but using the time interval to the first StreamGateStatesValue that is Open from the aggregated TSN streams.

\- Burst Size of a TSN stream at the ingress port (which is useful to map to MDBV) is determined based on the following conditions:

\- The Burst Size may be determined from TSN Stream gate control operations in the StreamGateAdminControlList. If in the StreamGateAdminControlList, IntervalOctetMax is provided for a StreamGateControlEntry with an "open" StreamGateStatesValue, the Burst Size is set to the IntervalOctetMax for that control list entry. If IntervalOctetMax is not provided, the Burst Size is set to the timeIntervalValue (converted from ns to s) of the StreamGateControlEntry with an "open" StreamGateStatesValue multiplied by the port bitrate.

\- When multiple compatible TSN Streams are aggregated, the Burst Size is set to the sum of the Burst Sizes for each TSN stream as determined above.

\- Maximum Flow Bitrate of a TSN stream (which is useful to map to GBR) is determined as follows:

\- The Maximum Flow Bitrate of a TSN Stream is equal to the summation of all timeIntervalValue (converted from ns to s) with StreamGateStatesValue = Open, multiplied by the bitrate of the corresponding port and divided by StreamGateAdminCycleTime. For aggregated TSN streams, the same calculation is performed over the burst of aggregated streams (calculated using superposition, i.e. timeIntervalValue with StreamGateStatesValue = Open of every stream is summed up, as they are assumed to have same periodicity, compatible Burst arrival time and same traffic class if they are to be aggregated.

When CNC configures the PSFP information to the TSN AF, the TSN AF may use local information (e.g. local configuration) to map the PSFP information to an ingress port and/or egress port of the 5GS bridge.

NOTE 2: As an example, for the local configuration, the PSFP can use either the destination MAC address and VLAN identifier, or the source MAC address and VLAN identifier for stream identification. The TSN AF is pre-configured with either the MAC address of Ethernet hosts behind a given DS-TT port (identified by the DS-TT port MAC address), or the VLAN identifier used over a given DS-TT port, or both. When the TSN AF determines that one of the known Ethernet host's MAC address appears as a source or destination MAC address, it can identify that the ingress or egress port is the associated DS-TT port.
