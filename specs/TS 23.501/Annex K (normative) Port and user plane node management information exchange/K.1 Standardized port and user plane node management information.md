---
spec: TS 23.501
version: 18.12.0
release: '18'
clause: K.1
title: K.1 Standardized port and user plane node management information
source_archive: 23501-ic0.zip
source_document: 23501-ic0.docx
source_archive_sha256: 800fb58a2a3a8c3eeac7084497de566fc730808923d8eea61b84d2555360debe
source_document_sha256: cdab3840e4d278d381f70806e6458f87dff8e5fd238f370e4f65872bbd5088a6
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# K.1 Standardized port and user plane node management information

Table K.1-1 and Table K.1-2 list standardized port management information and user plane node management information, respectively.

Table K.1-1: Standardized port management information

<table>
<colgroup>
<col style="width: 37%" />
<col style="width: 7%" />
<col style="width: 7%" />
<col style="width: 14%" />
<col style="width: 13%" />
<col style="width: 21%" />
</colgroup>
<thead>
<tr class="header">
<th>Port management information</th>
<th colspan="2">Applicability (see NOTE 6)</th>
<th>Supported operations by TSN AF</th>
<th>Supported operations by TSCTSF</th>
<th>Reference</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td></td>
<td>DS-TT</td>
<td>NW-TT</td>
<td>(see NOTE 1)</td>
<td>(see NOTE 1)</td>
<td></td>
</tr>
<tr class="even">
<td><strong>General</strong></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>Port management capabilities (see NOTE 2)</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>R</td>
<td></td>
</tr>
<tr class="even">
<td><strong>Bridge delay related information</strong></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>txPropagationDelay</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>-</td>
<td>IEEE Std 802.1Q [98] clause 12.32.2.1</td>
</tr>
<tr class="even">
<td>txPropagationDelayDeltaThreshold (see NOTE 23)</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><strong>Traffic class related information</strong></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Traffic class table</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1Q [98] clause 12.6.3 and clause 8.6.6.</td>
</tr>
<tr class="odd">
<td><strong>Gate control information</strong></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>queueMaxSDUTable</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>IEEE Std 802.1Q [98], clause 12.29.1</td>
</tr>
<tr class="odd">
<td>&gt; queueMaxSDU</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td></td>
<td>IEEE Std 802.1Q [98], clause 12.29.1</td>
</tr>
<tr class="even">
<td>&gt; TransmissionOverrun (see NOTE 3)</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td></td>
<td>IEEE Std 802.1Q [98], clause 12.29.1</td>
</tr>
<tr class="odd">
<td>GateEnabled</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1Q [98] Table 12-32</td>
</tr>
<tr class="even">
<td>AdminGateStates</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td></td>
<td>IEEE Std 802.1Q [98] Table 12-32</td>
</tr>
<tr class="odd">
<td>AdminBaseTime</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1Q [98] Table 12-32</td>
</tr>
<tr class="even">
<td>AdminControlList</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1Q [98] Table 12-32</td>
</tr>
<tr class="odd">
<td>AdminCycleTime (see NOTE 3)</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1Q [98] Table 12-32</td>
</tr>
<tr class="even">
<td>AdminControlListLength (see NOTE 3)</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1Q [98] Table 12-32</td>
</tr>
<tr class="odd">
<td>AdminCycleTimeExtension</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1Q [98] Table 12-32</td>
</tr>
<tr class="even">
<td>Tick granularity</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>-</td>
<td>IEEE Std 802.1Q [98] Table 12-32</td>
</tr>
<tr class="odd">
<td>SupportedListMax</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>-</td>
<td>IEEE Std 802.1Q [98] Table 12-32</td>
</tr>
<tr class="even">
<td><p><strong>General Neighbor discovery configuration</strong></p>
<p><strong>(NOTE 4)</strong></p></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>adminStatus</td>
<td>D</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] clause 9.2.5.1</td>
</tr>
<tr class="even">
<td>lldpV2LocChassisIdSubtype</td>
<td>D</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="odd">
<td>lldpV2LocChassisId</td>
<td>D</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="even">
<td>lldpV2MessageTxInterval</td>
<td>D</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="odd">
<td>lldpV2MessageTxHoldMultiplier</td>
<td>D</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="even">
<td><strong>NW-TT port neighbor discovery configuration</strong></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>lldpV2LocPortIdSubtype</td>
<td></td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="even">
<td>lldpV2LocPortId</td>
<td></td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="odd">
<td>OtlvSet</td>
<td></td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEC/IEEE 60802 [195], clause 6.5.2.4.5 and clause 6.5.2.4.6.</td>
</tr>
<tr class="even">
<td><strong>DS-TT port neighbor discovery configuration</strong></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>lldpV2LocPortIdSubtype</td>
<td>D</td>
<td></td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="even">
<td>lldpV2LocPortId</td>
<td>D</td>
<td></td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="odd">
<td>OtlvSet</td>
<td>D</td>
<td></td>
<td>RW</td>
<td>-</td>
<td>IEC/IEEE 60802 [195], clause 6.5.2.4.5 and clause 6.5.2.4.6.</td>
</tr>
<tr class="even">
<td><strong>Neighbor discovery information for each discovered neighbor of NW-TT (NOTE 26)</strong></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>lldpV2RemChassisIdSubtype</td>
<td></td>
<td>X</td>
<td>R</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="even">
<td>lldpV2RemChassisId</td>
<td></td>
<td>X</td>
<td>R</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="odd">
<td>lldpV2RemPortIdSubtype</td>
<td></td>
<td>X</td>
<td>R</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="even">
<td>lldpV2RemPortId</td>
<td></td>
<td>X</td>
<td>R</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="odd">
<td>TTL</td>
<td></td>
<td>X</td>
<td>R</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] clause 8.5.4</td>
</tr>
<tr class="even">
<td>OtlvSet</td>
<td></td>
<td>X</td>
<td>R</td>
<td>-</td>
<td>IEC/IEEE 60802 [195], clause 6.5.2.4.5 and clause 6.5.2.4.6.</td>
</tr>
<tr class="odd">
<td><p><strong>Neighbor discovery information for each discovered neighbor of DS-TT</strong></p>
<p><strong>(NOTE 5)</strong></p></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>lldpV2RemChassisIdSubtype</td>
<td>D</td>
<td></td>
<td>R</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="odd">
<td>lldpV2RemChassisId</td>
<td>D</td>
<td></td>
<td>R</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="even">
<td>lldpV2RemPortIdSubtype</td>
<td>D</td>
<td></td>
<td>R</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="odd">
<td>lldpV2RemPortId</td>
<td>D</td>
<td></td>
<td>R</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="even">
<td>TTL</td>
<td>D</td>
<td></td>
<td>R</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] clause 8.5.4.1</td>
</tr>
<tr class="odd">
<td>OtlvSet</td>
<td>D</td>
<td></td>
<td>R</td>
<td>-</td>
<td>IEC/IEEE 60802 [195], clause 6.5.2.4.5 and clause 6.5.2.4.6.</td>
</tr>
<tr class="even">
<td>Information for deterministic networking for each NW-TT port (NOTE 27)</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>Interface information</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Interface type</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8343 [151]</td>
</tr>
<tr class="odd">
<td>Interface enabled status</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8343 [151]</td>
</tr>
<tr class="even">
<td>phys-address</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8343 [151]</td>
</tr>
<tr class="odd">
<td>IPv4 information</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>IPv4 enabled status</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="odd">
<td>IPv4 forwarding status</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="even">
<td>IPv4 MTU</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="odd">
<td>Information for each IPv4 address</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>IPv4 address</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="odd">
<td>prefix-length</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="even">
<td>netmask</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="odd">
<td>origin</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="even">
<td>Information for each IPv4 neighbor</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>IPv4 address</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="even">
<td>link-layer-address</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="odd">
<td>origin</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="even">
<td>IPv6 information</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>IPv6 enabled status</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="even">
<td>IPv6 forwarding status</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="odd">
<td>IPv6 MTU</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="even">
<td>Information for each IPv6 address</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>IPv6 address</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="even">
<td>prefix-length</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="odd">
<td>origin</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="even">
<td>status</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="odd">
<td>Information for each IPv6 neighbor</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>IPv6 address</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="odd">
<td>link-layer-address</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="even">
<td>origin</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="odd">
<td>is-router</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="even">
<td>state</td>
<td></td>
<td>X</td>
<td></td>
<td>R</td>
<td>IETF RFC 8344 [152]</td>
</tr>
<tr class="odd">
<td><p><strong>Stream Parameters</strong></p>
<p><strong>(NOTE 11)</strong></p></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>MaxStreamFilterInstances</td>
<td>X</td>
<td></td>
<td>R</td>
<td>-</td>
<td><p>IEEE Std 802.1Q [98]</p>
<p>clause 12.31.1.1</p></td>
</tr>
<tr class="odd">
<td>MaxStreamGateInstances</td>
<td>X</td>
<td></td>
<td>R</td>
<td>-</td>
<td><p>IEEE Std 802.1Q [98]</p>
<p>clause 12.31.1.2</p></td>
</tr>
<tr class="even">
<td>MaxFlowMeterInstances</td>
<td>X</td>
<td></td>
<td>R</td>
<td>-</td>
<td><p>IEEE Std 802.1Q [98]</p>
<p>clause 12.31.1.3</p></td>
</tr>
<tr class="odd">
<td>SupportedListMax</td>
<td>X</td>
<td></td>
<td>R</td>
<td>-</td>
<td><p>IEEE Std 802.1Q [98]</p>
<p>clause 12.31.1.4</p></td>
</tr>
<tr class="even">
<td><p><strong>Per-Stream Filtering and Policing information</strong></p>
<p>(NOTE 10)</p></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Stream Filter Instance Table</p>
<p>(NOTE 8)</p></td>
<td></td>
<td></td>
<td></td>
<td>-</td>
<td>IEEE Std 802.1Q [98] Table 12-35</td>
</tr>
<tr class="even">
<td>&gt; StreamFilterInstanceIndex</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1Q [98] Table 12-35</td>
</tr>
<tr class="odd">
<td>&gt; Stream Identification type</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE 802.1CB [83] clause 9.1.1.6</td>
</tr>
<tr class="even">
<td>&gt; Stream Identification Controlling Parameters</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td><p>IEEE 802.1CB [83] clauses 9.1.2, 9.1.3, 9.1.4</p>
<p>(NOTE 12)</p></td>
</tr>
<tr class="odd">
<td>&gt; PrioritySpec</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1Q [98] Table 12-35</td>
</tr>
<tr class="even">
<td>&gt; StreamGateInstanceID</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1Q [98] Table 12-35</td>
</tr>
<tr class="odd">
<td><p>Stream Gate Instance Table</p>
<p>(NOTE 9)</p></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>IEEE Std 802.1Q [98] Table 12-33</td>
</tr>
<tr class="even">
<td>StreamGateInstanceIndex</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1Q [98] Table 12-36</td>
</tr>
<tr class="odd">
<td>StreamGateAdminBaseTime</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1Q [98] Table 12-36</td>
</tr>
<tr class="even">
<td>StreamGateAdminControlList</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1Q [98] Table 12-36</td>
</tr>
<tr class="odd">
<td>StreamGateAdminCycleTime</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1Q [98] Table 12-36</td>
</tr>
<tr class="even">
<td>StreamGateTickGranularity</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>-</td>
<td>IEEE Std 802.1Q [98] Table 12-36</td>
</tr>
<tr class="odd">
<td>StreamGateAdminCycleTimeExtension</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>-</td>
<td>IEEE Std 802.1Q [98] Table 12-36</td>
</tr>
<tr class="even">
<td><strong>Time Synchronization Information</strong></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>TSN Time domain number (NOTE 24)</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Supported PTP instance types (NOTE 13)</td>
<td>X</td>
<td></td>
<td>R</td>
<td>R</td>
<td>IEEE Std 1588 [126] clause 8.2.1.5.5</td>
</tr>
<tr class="odd">
<td>Supported transport types (NOTE 14)</td>
<td>X</td>
<td></td>
<td>R</td>
<td>R</td>
<td></td>
</tr>
<tr class="even">
<td>Supported delay mechanisms (NOTE 15)</td>
<td>X</td>
<td></td>
<td>R</td>
<td>R</td>
<td>IEEE Std 1588 [126] clause 8.2.15.4.4</td>
</tr>
<tr class="odd">
<td>PTP grandmaster capable (NOTE 16)</td>
<td>X</td>
<td></td>
<td>R</td>
<td>R</td>
<td></td>
</tr>
<tr class="even">
<td>gPTP grandmaster capable (NOTE 17)</td>
<td>X</td>
<td></td>
<td>R</td>
<td>R</td>
<td></td>
</tr>
<tr class="odd">
<td>Supported PTP profiles (NOTE 18)</td>
<td>X</td>
<td></td>
<td>R</td>
<td>R</td>
<td></td>
</tr>
<tr class="even">
<td>Number of supported PTP instances</td>
<td>X</td>
<td></td>
<td>R</td>
<td>R</td>
<td></td>
</tr>
<tr class="odd">
<td><strong>PTP instance specification</strong></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>PTP Instance ID (NOTE 25)</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td></td>
</tr>
<tr class="odd">
<td>&gt; PTP profile (NOTE 19)</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td></td>
</tr>
<tr class="even">
<td>&gt; Transport type (NOTE 20)</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td></td>
</tr>
<tr class="odd">
<td>&gt; Grandmaster enabled (NOTE 21)</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td></td>
</tr>
<tr class="even">
<td><strong>IEEE Std 1588 [126] data sets (NOTE 22)</strong></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>&gt; defaultDS.clockIdentity</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.2.2</td>
</tr>
<tr class="even">
<td>&gt; defaultDS.clockQuality.clockClass</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.3.1.2</td>
</tr>
<tr class="odd">
<td>&gt; defaultDS.clockQuality.clockAccuracy</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.3.1.3</td>
</tr>
<tr class="even">
<td>&gt; defaultDS.clockQuality.offsetScaledLogVariance</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.3.1.4</td>
</tr>
<tr class="odd">
<td>&gt; defaultDS.priority1</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.4.1</td>
</tr>
<tr class="even">
<td>&gt; defaultDS.priority2</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.4.2</td>
</tr>
<tr class="odd">
<td>&gt; defaultDS.domainNumber</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.4.3</td>
</tr>
<tr class="even">
<td>&gt; defaultDS.sdoId</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.4.5</td>
</tr>
<tr class="odd">
<td>&gt; defaultDS.instanceEnable</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.5.2</td>
</tr>
<tr class="even">
<td>&gt; defaultDS.instanceType</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.5.5</td>
</tr>
<tr class="odd">
<td>&gt; portDS.portIdentity</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.2.1</td>
</tr>
<tr class="even">
<td>&gt; portDS.portState</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 1588 [126] clause 8.2.15.3.1</td>
</tr>
<tr class="odd">
<td>&gt; portDS.logMinDelayReqInterval</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.3.2</td>
</tr>
<tr class="even">
<td>&gt; portDS.logAnnounceInterval</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.4.1</td>
</tr>
<tr class="odd">
<td>&gt; portDS.announceReceiptTimeout</td>
<td></td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.4.2</td>
</tr>
<tr class="even">
<td>&gt; portDS.logSyncInterval</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.4.3</td>
</tr>
<tr class="odd">
<td>&gt; portDS.delayMechanism</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.4.4</td>
</tr>
<tr class="even">
<td>&gt; portDS.logMinPdelayReqInterval</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.4.5</td>
</tr>
<tr class="odd">
<td>&gt; portDS.versionNumber</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.4.6</td>
</tr>
<tr class="even">
<td>&gt; portDS.minorVersionNumber</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.4.7</td>
</tr>
<tr class="odd">
<td>&gt; portDS.delayAsymmetry</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.4.8</td>
</tr>
<tr class="even">
<td>&gt; portDS.portEnable</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.5.1</td>
</tr>
<tr class="odd">
<td>&gt; timePropertiesDS.currentUtcOffset</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.4.2</td>
</tr>
<tr class="even">
<td>&gt; timePropertiesDS.timeSource</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.4.9</td>
</tr>
<tr class="odd">
<td>&gt; externalPortConfigurationPortDS.desiredState</td>
<td></td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 15.5.3.7.15.1</td>
</tr>
<tr class="even">
<td><strong>IEEE Std 802.1AS [104] data sets (NOTE 22)</strong></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>&gt; defaultDS.clockIdentity</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.2</td>
</tr>
<tr class="even">
<td>&gt; defaultDS.clockQuality.clockClass</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.4.2</td>
</tr>
<tr class="odd">
<td>&gt; defaultDS.clockQuality.clockAccuracy</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.4.3</td>
</tr>
<tr class="even">
<td>&gt; defaultDS.clockQuality.offsetScaledLogVariance</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.4.4</td>
</tr>
<tr class="odd">
<td>&gt; defaultDS.priority1</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.5</td>
</tr>
<tr class="even">
<td>&gt; defaultDS.priority2</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.6</td>
</tr>
<tr class="odd">
<td>&gt; defaultDS.timeSource</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.15</td>
</tr>
<tr class="even">
<td>&gt; defaultDS.domainNumber</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.16</td>
</tr>
<tr class="odd">
<td>&gt; defaultDS.sdoId</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.4.3</td>
</tr>
<tr class="even">
<td>&gt; defaultDS.instanceEnable</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.19</td>
</tr>
<tr class="odd">
<td>&gt; portDS.portIdentity</td>
<td></td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.2</td>
</tr>
<tr class="even">
<td>&gt; portDS.portState</td>
<td></td>
<td>X</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.3</td>
</tr>
<tr class="odd">
<td>&gt; portDS.ptpPortEnabled</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.4</td>
</tr>
<tr class="even">
<td>&gt; portDS.delayMechanism</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.5</td>
</tr>
<tr class="odd">
<td>&gt; portDS.isMeasuringDelay</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.6</td>
</tr>
<tr class="even">
<td>&gt; portDS.asCapable</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.7</td>
</tr>
<tr class="odd">
<td>&gt; portDS.meanLinkDelay</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.8</td>
</tr>
<tr class="even">
<td>&gt; portDS.meanLinkDelayThresh</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.9</td>
</tr>
<tr class="odd">
<td>&gt; portDS.delayAsymmetry</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.10</td>
</tr>
<tr class="even">
<td>&gt; portDS.neighborRateRatio</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.11</td>
</tr>
<tr class="odd">
<td>&gt; portDS.initialLogAnnounceInterval</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.12</td>
</tr>
<tr class="even">
<td>&gt; portDS.currentLogAnnounceInterval</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.13</td>
</tr>
<tr class="odd">
<td>&gt; portDS.useMgtSettableLogAnnounceInterval</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.14</td>
</tr>
<tr class="even">
<td>&gt; portDS.mgtSettableLogAnnounceInterval</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.15</td>
</tr>
<tr class="odd">
<td>&gt; portDS.announceReceiptTimeout</td>
<td></td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.16</td>
</tr>
<tr class="even">
<td>&gt; portDS.initialLogSyncInterval</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.17</td>
</tr>
<tr class="odd">
<td>&gt; portDS.currentLogSyncInterval</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.18</td>
</tr>
<tr class="even">
<td>&gt; portDS.useMgtSettableLogSyncInterval</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.19</td>
</tr>
<tr class="odd">
<td>&gt; portDS.mgtSettableLogSyncInterval</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.20</td>
</tr>
<tr class="even">
<td>&gt; portDS.syncReceiptTimeout</td>
<td></td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.21</td>
</tr>
<tr class="odd">
<td>&gt; portDS.syncReceiptTimeoutTimeInterval</td>
<td></td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.22</td>
</tr>
<tr class="even">
<td>&gt; portDS.initialLogPdelayReqInterval</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.23</td>
</tr>
<tr class="odd">
<td>&gt; portDS.currentLogPdelayReqInterval</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.24</td>
</tr>
<tr class="even">
<td>&gt; portDS.useMgtSettableLogPdelayReqInterval</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.25</td>
</tr>
<tr class="odd">
<td>&gt; portDS.mgtSettableLogPdelayReqInterval</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.26</td>
</tr>
<tr class="even">
<td>&gt; portDS.initialLogGptpCapableMessageInterval</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.27</td>
</tr>
<tr class="odd">
<td>&gt; portDS.currentLogGptpCapableMessageInterval</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.28</td>
</tr>
<tr class="even">
<td>&gt; portDS.useMgtSettableLogGptpCapableMessageInterval</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.29</td>
</tr>
<tr class="odd">
<td>&gt; portDS.mgtSettableLogGptpCapableMessageInterval</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.30</td>
</tr>
<tr class="even">
<td>&gt; portDS.initialComputeNeighborRateRatio</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.31</td>
</tr>
<tr class="odd">
<td>&gt; portDS.currentComputeNeighborRateRatio</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.32</td>
</tr>
<tr class="even">
<td>&gt; portDS.useMgtSettableComputeNeighborRateRatio</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.33</td>
</tr>
<tr class="odd">
<td>&gt; portDS.mgtSettableComputeNeighborRateRatio</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.34</td>
</tr>
<tr class="even">
<td>&gt; portDS.initialComputeMeanLinkDelay</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.35</td>
</tr>
<tr class="odd">
<td>&gt; portDS.currentComputeMeanLinkDelay</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.36</td>
</tr>
<tr class="even">
<td>&gt; portDS.useMgtSettableComputeMeanLinkDelay</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.37</td>
</tr>
<tr class="odd">
<td>&gt; portDS.mgtSettableComputeMeanLinkDelay</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.38</td>
</tr>
<tr class="even">
<td>&gt; portDS.allowedLostResponses</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.39</td>
</tr>
<tr class="odd">
<td>&gt; portDS.allowedFaults</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.40</td>
</tr>
<tr class="even">
<td>&gt; portDS.gPtpCapableReceiptTimeout</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.41</td>
</tr>
<tr class="odd">
<td>&gt; portDS.versionNumber</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.42</td>
</tr>
<tr class="even">
<td>&gt; portDS.nup</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.43</td>
</tr>
<tr class="odd">
<td>&gt; portDS.ndown</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.44</td>
</tr>
<tr class="even">
<td>&gt; portDS.oneStepTxOper</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.45</td>
</tr>
<tr class="odd">
<td>&gt; portDS.oneStepReceive</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.46</td>
</tr>
<tr class="even">
<td>&gt; portDS.oneStepTransmit</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.47</td>
</tr>
<tr class="odd">
<td>&gt; portDS.initialOneStepTxOper</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.48</td>
</tr>
<tr class="even">
<td>&gt; portDS.currentOneStepTxOper</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.49</td>
</tr>
<tr class="odd">
<td>&gt; portDS.useMgtSettableOneStepTxOper</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.50</td>
</tr>
<tr class="even">
<td>&gt; portDS.mgtSettableOneStepTxOper</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.51</td>
</tr>
<tr class="odd">
<td>&gt; portDS.syncLocked</td>
<td>X</td>
<td>X</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.52</td>
</tr>
<tr class="even">
<td>&gt; portDS.pdelayTruncatedTimestampsArray</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.53</td>
</tr>
<tr class="odd">
<td>&gt; portDS.minorVersionNumber</td>
<td>X</td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.54</td>
</tr>
<tr class="even">
<td>&gt; timePropertiesDS.currentUtcOffset</td>
<td>X</td>
<td></td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.5.2</td>
</tr>
<tr class="odd">
<td>&gt; externalPortConfigurationPortDS.desiredState</td>
<td></td>
<td>X</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.12.2</td>
</tr>
<tr class="even">
<td colspan="6"><p>NOTE 1: R = Read only access; RW = Read/Write access; ― = not supported.</p>
<p>NOTE 2: Indicates which standardized and deployment-specific port management information is supported by DS-TT or NW-TT.</p>
<p>NOTE 3: AdminCycleTime, AdminControlListLength and TransmissionOverrun are optional for gate control information.</p>
<p>NOTE 4: If DS-TT supports neighbor discovery, then TSN AF sends the general neighbor discovery configuration for DS-TT Ethernet ports to DS-TT. If DS-TT does not support neighbor discovery, then TSN AF sends the general neighbor discovery configuration for DS-TT Ethernet ports to NW-TT using the User Plane Node Management Information Container (refer to Table K.1-2) and NW-TT performs neighbor discovery on behalf on DS-TT. When a parameter in this group is changed, it is necessary to provide the change to every DS-TT and the NW-TT that belongs to the 5GS TSN bridge. It is mandatory that the general neighbor discovery configuration is identical for all DS-TTs and the NW-TTs that belongs to the bridge.</p>
<p>NOTE 5: If DS-TT supports neighbor discovery, then TSN AF retrieves neighbor discovery information for DS-TT Ethernet ports from DS-TT. TSN AF indicates the neighbor discovery information for each discovered neighbor of DS-TT port to CNC. If DS-TT does not support neighbor discovery, then TSN AF retrieves neighbor discovery information for DS-TT Ethernet ports from NW-TT, using the User Plane Node Management Information Container (refer to Table K.1-2), the NW-TT performing neighbor discovery on behalf on DS-TT.</p>
<p>NOTE 6: X = applicable; D = applicable when validation and generation of LLDP frames is processed at the DS-TT.</p>
<p>NOTE 7: Void.</p>
<p>NOTE 8: There is a Stream Filter Instance Table per Stream.</p>
<p>NOTE 9: There is a Stream Gate Instance Table per Gate.</p>
<p>NOTE 10: TSN AF indicates the support for PSFP to the CNC only if each DS-TT and NW-TT of the 5GS bridge has indicated support of PSFP. DS-TT indicates support of PSFP using port management capabilities, i.e. by indicating support for the Per-Stream Filtering and Policing information and by setting higher than zero values for MaxStreamFilterInstances, MaxStreamGateInstances, MaxFlowMeterInstances, SupportedListMax parameters. When available, TSN AF uses the PSFP information for determination of the traffic pattern information as described in Annex I. The PSFP information can be used at the DS-TT (if supported) and at the NW-TT (if supported) for the purpose of per-stream filtering and policing as defined in clause 8.6.5.2.1 of IEEE Std 802.1Q [98].</p>
<p>NOTE 11: TSN AF composes a Stream Parameter Table towards the CNC. It is up to TSN AF how it composes the Stream Parameter Table based on the numerical values as received from DS-TT and NW-TT port(s) and for the bridge for each individual parameter.</p>
<p>NOTE 12: The set of Stream Identification Controlling Parameters depends on the Stream Identification type value as defined in IEEE Std 802.1CB [83] Table 9-1 and clauses 9.1.2, 9.1.3, 9.1.4.</p>
<p>NOTE 13: Enumeration of supported PTP instance types. Allowed values as defined in clause 8.2.1.5.5 of IEEE Std 1588 [126].</p>
<p>NOTE 14: Enumeration of supported transport types. Allowed values: IPv4 (as defined in Annex C of IEEE Std 1588 [126]), IPv6 (as defined in IEEE Std 1588 [126] Annex D), Ethernet (as defined in Annex E of IEEE Std 1588 [126]).</p>
<p>NOTE 15: Enumeration of supported PTP delay mechanisms. Allowed values as defined in clause 8.2.15.4.4 of IEEE Std 1588 [126].</p>
<p>NOTE 16: Indicates whether DS-TT supports acting as a PTP grandmaster.</p>
<p>NOTE 17: Indicates whether DS-TT supports acting as a gPTP grandmaster.</p>
<p>NOTE 18: Enumeration of supported PTP profiles, each identified by PTP profile ID, as defined in clause 20.3.3 of IEEE Std 1588 [126].</p>
<p>NOTE 19: PTP profile to apply, identified by PTP profile ID, as defined in clause 20.3.3 of IEEE Std 1588 [126].</p>
<p>NOTE 20: Transport type to use. Allowed values: IPv4 (as defined in Annex C of IEEE Std 1588 [126]), IPv6 (as defined in IEEE Std 1588 [126] Annex D), Ethernet (as defined in Annex E of IEEE Std 1588 [126]).</p>
<p>NOTE 21: Indicates whether to act as grandmaster or not, i.e. whether to send Announce, Sync and optionally Follow_Up messages.</p>
<p>NOTE 22: The IEEE Std 802.1AS [104] data sets apply if the IEEE 802.1AS PTP profile is used; otherwise the IEEE Std 1588 [126] data sets apply.</p>
<p>NOTE 23: Indicates how much the txPropagationDelay needs to change so that DS-TT/NW-TT report a change in txPropagationDelay to TSN AF. This is optional for NW-TT.</p>
<p>NOTE 24: Indicates the gPTP domain (identified by a domain number) that is assumed by the CNC as the reference clock for time information in the scheduled traffic (gate control) information, PSFP information and bridge delay related information. This is optional for NW-TT.</p>
<p>NOTE 25: PTP Instance ID uniquely identifies a PTP instance within the user plane node.</p>
<p>NOTE 26: TSN AF indicates the neighbor discovery information for each discovered neighbor of NW-TT port to CNC.</p>
<p>NOTE 27: Applicable in case of interworking with IETF Deterministic Networking.</p></td>
</tr>
</tbody>
</table>

Table K.1-2: Standardized user plane node management information

<table style="width:100%;">
<colgroup>
<col style="width: 50%" />
<col style="width: 14%" />
<col style="width: 13%" />
<col style="width: 21%" />
</colgroup>
<thead>
<tr class="header">
<th>User plane node management information</th>
<th>Supported operations by TSN AF</th>
<th>Supported operations by TSCTSF</th>
<th>Reference</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td></td>
<td>(see NOTE 1)</td>
<td>(see NOTE 1)</td>
<td></td>
</tr>
<tr class="even">
<td><strong>Information for 5GS Bridge/Router</strong></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>User plane node Address</td>
<td>R</td>
<td>R</td>
<td></td>
</tr>
<tr class="even">
<td>User plane node ID</td>
<td>R</td>
<td>R</td>
<td></td>
</tr>
<tr class="odd">
<td>NW-TT port numbers</td>
<td>R</td>
<td>R</td>
<td></td>
</tr>
<tr class="even">
<td><strong>Traffic forwarding information</strong></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>Static Filtering Entry (NOTE 3)</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1Q [98] clause 8.8.1</td>
</tr>
<tr class="even">
<td><p><strong>General Neighbor discovery configuration</strong></p>
<p><strong>(NOTE 2)</strong></p></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>adminStatus</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] clause 9.2.5.1</td>
</tr>
<tr class="even">
<td>lldpV2LocChassisIdSubtype</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="odd">
<td>lldpV2LocChassisId</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="even">
<td>lldpV2MessageTxInterval</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="odd">
<td>lldpV2MessageTxHoldMultiplier</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="even">
<td><strong>DS-TT port neighbor discovery configuration for DS-TT ports (NOTE 4)</strong></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><strong>&gt;DS-TT port neighbor discovery configuration for each DS-TT port</strong></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>&gt;&gt; DS-TT port number</td>
<td>RW</td>
<td>-</td>
<td></td>
</tr>
<tr class="odd">
<td>&gt;&gt; lldpV2LocPortIdSubtype</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="even">
<td>&gt;&gt; lldpV2LocPortId</td>
<td>RW</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="odd">
<td>&gt;&gt; OtlvSet</td>
<td>RW</td>
<td>-</td>
<td>IEC/IEEE 60802 [195], clause 6.5.2.4.5 and clause 6.5.2.4.6.</td>
</tr>
<tr class="even">
<td><p><strong>Discovered neighbor information for DS-TT ports</strong></p>
<p><strong>(NOTE 4)</strong></p></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p><strong>&gt;Discovered neighbor information for each DS-TT port</strong></p>
<p><strong>(NOTE 4)</strong></p></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>&gt;&gt; DS-TT port number</td>
<td>R</td>
<td>-</td>
<td></td>
</tr>
<tr class="odd">
<td>&gt;&gt; lldpV2RemChassisIdSubtype</td>
<td>R</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="even">
<td>&gt;&gt; lldpV2RemChassisId</td>
<td>R</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="odd">
<td>&gt;&gt; lldpV2RemPortIdSubtype</td>
<td>R</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="even">
<td>&gt;&gt; lldpV2RemPortId</td>
<td>R</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] Table 11-2</td>
</tr>
<tr class="odd">
<td>&gt;&gt; TTL</td>
<td>R</td>
<td>-</td>
<td>IEEE Std 802.1AB [97] clause 8.5.4.1</td>
</tr>
<tr class="even">
<td>&gt;&gt; OtlvSet</td>
<td>R</td>
<td>-</td>
<td>IEC/IEEE 60802 [195], clause 6.5.2.4.5 and clause 6.5.2.4.6.</td>
</tr>
<tr class="odd">
<td><strong>Stream Parameters (NOTE 5)</strong></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>MaxStreamFilterInstances</td>
<td>R</td>
<td>-</td>
<td><p>IEEE Std 802.1Q [98]</p>
<p>Table 12-34</p></td>
</tr>
<tr class="odd">
<td>MaxStreamGateInstances</td>
<td>R</td>
<td>-</td>
<td><p>IEEE Std 802.1Q [98]</p>
<p>Table 12-34</p></td>
</tr>
<tr class="even">
<td>MaxFlowMeterInstances</td>
<td>R</td>
<td>-</td>
<td><p>IEEE Std 802.1Q [98]</p>
<p>Table 12-34</p></td>
</tr>
<tr class="odd">
<td>SupportedListMax</td>
<td>R</td>
<td>-</td>
<td><p>IEEE Std 802.1Q [98]</p>
<p>Table 12-34</p></td>
</tr>
<tr class="even">
<td><strong>Time synchronization information</strong></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>Supported PTP instance types (NOTE 6)</td>
<td>R</td>
<td>R</td>
<td></td>
</tr>
<tr class="even">
<td>Supported transport types (NOTE 7)</td>
<td>R</td>
<td>R</td>
<td></td>
</tr>
<tr class="odd">
<td>Supported delay mechanisms (NOTE 8)</td>
<td>R</td>
<td>R</td>
<td></td>
</tr>
<tr class="even">
<td>PTP grandmaster capable (NOTE 9)</td>
<td>R</td>
<td>R</td>
<td></td>
</tr>
<tr class="odd">
<td>gPTP grandmaster capable (NOTE 10)</td>
<td>R</td>
<td>R</td>
<td></td>
</tr>
<tr class="even">
<td>Supported PTP profiles (NOTE 11)</td>
<td>R</td>
<td>R</td>
<td></td>
</tr>
<tr class="odd">
<td>Number of supported PTP instances</td>
<td>R</td>
<td>R</td>
<td></td>
</tr>
<tr class="even">
<td><strong>Time synchronization information for PTP instances (NOTE 16)</strong></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><strong>&gt; PTP instance specification</strong></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>&gt;&gt; PTP Instance ID (NOTE 17)</td>
<td>RW</td>
<td>RW</td>
<td></td>
</tr>
<tr class="odd">
<td>&gt;&gt; PTP profile (NOTE 12)</td>
<td>RW</td>
<td>RW</td>
<td></td>
</tr>
<tr class="even">
<td>&gt;&gt; Transport type (NOTE 13)</td>
<td>RW</td>
<td>RW</td>
<td></td>
</tr>
<tr class="odd">
<td>&gt;&gt; Grandmaster candidate enabled</td>
<td>RW</td>
<td>RW</td>
<td></td>
</tr>
<tr class="even">
<td><strong>IEEE Std 1588 [126] data sets (NOTE 15)</strong></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>&gt;&gt; defaultDS.clockIdentity</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.2.2</td>
</tr>
<tr class="even">
<td>&gt;&gt; defaultDS.clockQuality.clockClass</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.3.1.2</td>
</tr>
<tr class="odd">
<td>&gt;&gt; defaultDS.clockQuality.clockAccuracy</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.3.1.3</td>
</tr>
<tr class="even">
<td>&gt;&gt; defaultDS.clockQuality.offsetScaledLogVariance</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.3.1.4</td>
</tr>
<tr class="odd">
<td>&gt;&gt; defaultDS.priority1</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.4.1</td>
</tr>
<tr class="even">
<td>&gt;&gt; defaultDS.priority2</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.4.2</td>
</tr>
<tr class="odd">
<td>&gt;&gt; defaultDS.domainNumber</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.4.3</td>
</tr>
<tr class="even">
<td>&gt;&gt; defaultDS.sdoId</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.4.5</td>
</tr>
<tr class="odd">
<td>&gt;&gt; defaultDS.instanceEnable</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.5.2</td>
</tr>
<tr class="even">
<td>&gt;&gt; defaultDS.externalPortConfigurationEnabled</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.5.3</td>
</tr>
<tr class="odd">
<td>&gt;&gt; defaultDS.instanceType</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.1.5.5</td>
</tr>
<tr class="even">
<td>&gt;&gt; timePropertiesDS.currentUtcOffset</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.4.2</td>
</tr>
<tr class="odd">
<td>&gt;&gt; timePropertiesDS.timeSource</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.4.9</td>
</tr>
<tr class="even">
<td><strong>IEEE Std 802.1AS [104] data sets (NOTE 15)</strong></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>&gt;&gt; defaultDS.clockIdentity</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.2</td>
</tr>
<tr class="even">
<td>&gt;&gt; defaultDS.clockQuality.clockClass</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.4.2</td>
</tr>
<tr class="odd">
<td>&gt;&gt; defaultDS.clockQuality.clockAccuracy</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.4.3</td>
</tr>
<tr class="even">
<td>&gt;&gt; defaultDS.clockQuality.offsetScaledLogVariance</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.4.4</td>
</tr>
<tr class="odd">
<td>&gt;&gt; defaultDS.priority1</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.5</td>
</tr>
<tr class="even">
<td>&gt;&gt; defaultDS.priority2</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.6</td>
</tr>
<tr class="odd">
<td>&gt;&gt; defaultDS.timeSource</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.15</td>
</tr>
<tr class="even">
<td>&gt;&gt; defaultDS.domainNumber</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.16</td>
</tr>
<tr class="odd">
<td>&gt;&gt; defaultDS.sdoId</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.18</td>
</tr>
<tr class="even">
<td>&gt;&gt; defaultDS.externalPortConfigurationEnabled</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.4.3</td>
</tr>
<tr class="odd">
<td>&gt;&gt; defaultDS.instanceEnable</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.2.19</td>
</tr>
<tr class="even">
<td>&gt;&gt; timePropertiesDS.currentUtcOffset</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.5.2</td>
</tr>
<tr class="odd">
<td><strong>Time synchronization information for DS-TT ports</strong></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td><strong>&gt; Time synchronization information for each DS-TT port</strong></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>&gt; DS-TT port number</td>
<td>RW</td>
<td>RW</td>
<td></td>
</tr>
<tr class="even">
<td><strong>&gt;&gt; Time synchronization information for each PTP Instance</strong></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>&gt;&gt; PTP Instance ID (NOTE 17)</td>
<td>RW</td>
<td>RW</td>
<td></td>
</tr>
<tr class="even">
<td>&gt;&gt; Grandmaster on behalf of DS-TT enabled (NOTE 14)</td>
<td>RW</td>
<td>RW</td>
<td></td>
</tr>
<tr class="odd">
<td><strong>IEEE Std 1588 [126] data sets (NOTE 15)</strong></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.portIdentity</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.2.1</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.portState</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 1588 [126] clause 8.2.15.3.1</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.logMinDelayReqInterval</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.3.2</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.logAnnounceInterval</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.4.1</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.announceReceiptTimeout</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.4.2</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.logSyncInterval</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.4.3</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.delayMechanism</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.4.4</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.logMinPdelayReqInterval</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.4.5</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.versionNumber</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.4.6</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.minorVersionNumber</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.4.7</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.delayAsymmetry</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.4.8</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.portEnable</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 8.2.15.5.1</td>
</tr>
<tr class="even">
<td>&gt;&gt; externalPortConfigurationPortDS.desiredState</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 1588 [126] clause 15.5.3.7.15.1</td>
</tr>
<tr class="odd">
<td><strong>IEEE Std 802.1AS [104] data sets (NOTE 15)</strong></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.portIdentity</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.2</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.portState</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.3</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.ptpPortEnabled</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.4</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.delayMechanism</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.5</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.isMeasuringDelay</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.6</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.asCapable</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.7</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.meanLinkDelay</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.8</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.meanLinkDelayThresh</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.9</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.delayAsymmetry</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.10</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.neighborRateRatio</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.11</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.initialLogAnnounceInterval</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.12</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.currentLogAnnounceInterval</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.13</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.useMgtSettableLogAnnounceInterval</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.14</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.mgtSettableLogAnnounceInterval</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.15</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.announceReceiptTimeout</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.16</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.initialLogSyncInterval</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.17</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.currentLogSyncInterval</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.18</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.useMgtSettableLogSyncInterval</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.19</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.mgtSettableLogSyncInterval</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.20</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.syncReceiptTimeout</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.21</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.syncReceiptTimeoutTimeInterval</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.22</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.initialLogPdelayReqInterval</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.23</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.currentLogPdelayReqInterval</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.24</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.useMgtSettableLogPdelayReqInterval</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.25</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.mgtSettableLogPdelayReqInterval</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.26</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.initialLogGptpCapableMessageInterval</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.27</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.currentLogGptpCapableMessageInterval</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.28</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.useMgtSettableLogGptpCapableMessageInterval</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.29</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.mgtSettableLogGptpCapableMessageInterval</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.30</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.initialComputeNeighborRateRatio</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.31</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.currentComputeNeighborRateRatio</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.32</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.useMgtSettableComputeNeighborRateRatio</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.33</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.mgtSettableComputeNeighborRateRatio</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.34</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.initialComputeMeanLinkDelay</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.35</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.currentComputeMeanLinkDelay</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.36</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.useMgtSettableComputeMeanLinkDelay</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.37</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.mgtSettableComputeMeanLinkDelay</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.38</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.allowedLostResponses</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.39</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.allowedFaults</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.40</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.gPtpCapableReceiptTimeout</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.41</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.versionNumber</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.42</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.nup</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.43</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.ndown</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.44</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.oneStepTxOper</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.45</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.oneStepReceive</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.46</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.oneStepTransmit</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.47</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.initialOneStepTxOper</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.48</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.currentOneStepTxOper</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.49</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.useMgtSettableOneStepTxOper</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.50</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.mgtSettableOneStepTxOper</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.51</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.syncLocked</td>
<td>R</td>
<td>R</td>
<td>IEEE Std 802.1AS [104] clause 14.8.52</td>
</tr>
<tr class="odd">
<td>&gt;&gt; portDS.pdelayTruncatedTimestampsArray</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.53</td>
</tr>
<tr class="even">
<td>&gt;&gt; portDS.minorVersionNumber</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.8.54</td>
</tr>
<tr class="odd">
<td>&gt;&gt; externalPortConfigurationPortDS.desiredState</td>
<td>RW</td>
<td>RW</td>
<td>IEEE Std 802.1AS [104] clause 14.12.2</td>
</tr>
<tr class="even">
<td><strong>Time synchronization status (TSS) information</strong></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>&gt; Synchronization state</td>
<td>-</td>
<td>R</td>
<td>Table 5.27.1.12-1</td>
</tr>
<tr class="even">
<td>&gt; Clock quality</td>
<td>-</td>
<td>R</td>
<td>Table 5.27.1.12-1</td>
</tr>
<tr class="odd">
<td>&gt;&gt; Traceable to UTC</td>
<td>-</td>
<td>R</td>
<td>Table 5.27.1.12-1</td>
</tr>
<tr class="even">
<td>&gt;&gt; Traceable to GNSS</td>
<td>-</td>
<td>R</td>
<td>Table 5.27.1.12-1</td>
</tr>
<tr class="odd">
<td>&gt;&gt; Frequency stability</td>
<td>-</td>
<td>R</td>
<td>Table 5.27.1.12-1</td>
</tr>
<tr class="even">
<td>&gt;&gt; Clock accuracy</td>
<td>-</td>
<td>R</td>
<td>Table 5.27.1.12-1</td>
</tr>
<tr class="odd">
<td>&gt; Parent time source</td>
<td>-</td>
<td>R</td>
<td>Table 5.27.1.12-1</td>
</tr>
<tr class="even">
<td colspan="4"><p>NOTE 1: R = Read only access; RW = Read/Write access; ― = not supported.</p>
<p>NOTE 2: General neighbor discovery information is included only when NW-TT performs neighbor discovery on behalf of DS-TT. When a parameter in this group is changed, it is necessary to provide the change to every DS-TT and the NW-TT that belongs to the 5GS TSN bridge.</p>
<p>NOTE 3: If the Static Filtering Entry information is present, UPF/NW-TT can use Static Filtering Entry information for forwarding TSC traffic, as specified in clause 5.8.2.5.3.</p>
<p>NOTE 4: DS-TT discovery configuration and DS-TT discovery information are used only when DS-TT does not support LLDP and NW-TT performs neighbor discovery on behalf of DS-TT. TSN AF indicates the discovered neighbor information for each DS-TT port to CNC.</p>
<p>NOTE 5: TSN AF indicates the support for PSFP to the CNC only if each DS-TT and NW-TT of the 5GS bridge have indicated support of PSFP. The support of PSFP at the NW-TT ports is expressed by setting higher than zero values for MaxStreamFilterInstances, MaxStreamGateInstances, MaxFlowMeterInstances, SupportedListMax parameters.</p>
<p>NOTE 6: Enumeration of supported PTP instance types. Allowed values as defined in clause 8.2.1.5.5 of IEEE Std 1588 [126].</p>
<p>NOTE 7: Enumeration of supported transport types. Allowed values: IPv4 (as defined in IEEE Std 1588 [126] Annex C), IPv6 (as defined in IEEE Std 1588 [126] Annex D), Ethernet (as defined in Annex E of IEEE Std 1588 [126]).</p>
<p>NOTE 8: Enumeration of supported PTP delay mechanisms. Allowed values as defined in clause 8.2.15.4.4 of IEEE Std 1588 [126].</p>
<p>NOTE 9: Indicates whether NW-TT supports acting as a PTP grandmaster.</p>
<p>NOTE 10: Indicates whether NW-TT supports acting as a gPTP grandmaster.</p>
<p>NOTE 11: Enumeration of supported PTP profiles, each identified by PTP profile ID, as defined in clause 20.3.3 of IEEE Std 1588 [126].</p>
<p>NOTE 12: PTP profile to apply, identified by PTP profile ID, as defined in clause 20.3.3 of IEEE Std 1588 [126].</p>
<p>NOTE 13: Transport type to use. Allowed values: IPv4 (as defined in Annex C of IEEE Std 1588 [126]), IPv6 (as defined in IEEE Std 1588 [126] Annex D), Ethernet (as defined in Annex E of IEEE Std 1588 [126]).</p>
<p>NOTE 14: Indicates whether to act as grandmaster on behalf of a DS-TT port or not if 5GS is determined to be the grandmaster clock, i.e. whether to send Announce, Sync and optionally Follow_Up messages on behalf of DS-TT.</p>
<p>NOTE 15: The IEEE Std 802.1AS [104] data sets apply if the IEEE 802.1AS PTP profile is used; otherwise, the IEEE Std 1588 [126] data sets apply.</p>
<p>NOTE 16: Specifies the default data set for each PTP instance identified by PTP instance ID within the user plane node.</p>
<p>NOTE 17: PTP Instance ID uniquely identifies a PTP instance within the user plane node.</p></td>
</tr>
</tbody>
</table>
