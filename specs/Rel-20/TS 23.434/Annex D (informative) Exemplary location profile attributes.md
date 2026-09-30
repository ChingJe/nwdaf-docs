---
spec: TS 23.434
version: 20.1.0
release: '20'
clause: Annex D
title: 'Annex D (informative): Exemplary location profile attributes'
source_archive: 23434-k10.zip
source_document: 23434-k10.docx
content_origin: 3gpp-source
---

# Annex D (informative): Exemplary location profile attributes

The table D-1 shows the example of attributes that can be used for the location profiles.

Table D-1: Exemplary location profile attributes

<table style="width:100%;">
<colgroup>
<col style="width: 10%" />
<col style="width: 16%" />
<col style="width: 20%" />
<col style="width: 12%" />
<col style="width: 15%" />
<col style="width: 10%" />
<col style="width: 13%" />
</colgroup>
<thead>
<tr class="header">
<th>Profile ID / name</th>
<th>Vertical / use case/environment</th>
<th>Positioning Service Level (for IIOT) / QoS / accuracy</th>
<th>Positioning Method(s) / Priorities</th>
<th>Involved 3GPP functionalities / Priorities</th>
<th>Required APIs / API info</th>
<th>Other</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Location profile #1</td>
<td>Industrial scenario; indoors; mobile robots/ AGVs</td>
<td><p>Service Level 6 /</p>
<p>cm level accuracy / absolute/relative/ both</p></td>
<td><p>1. DL-TDOA,</p>
<p>2. UL-TDOA,</p>
<p>3. Multi-RTT methods,</p>
<p>4. WLAN, 5. motion sensors,</p>
<p>6. Bluetooth</p></td>
<td><p>1. LMF</p>
<p>2. RAN-LMC, 3. SEAL LMS</p></td>
<td>NEF APIs, SEAL APIs</td>
<td>Verification / augmentation required</td>
</tr>
<tr class="even">
<td>Location profile #2</td>
<td><p>V2X;</p>
<p>outdoor</p></td>
<td>Decimeter level accuracy /... absolute/relative/both</td>
<td><p>1. DL-TDOA,</p>
<p>2. Multi-RTT methods,</p>
<p>3. GNSS-RTK,</p>
<p>4. Sensor fusion,</p>
<p>5. A-GPS</p></td>
<td><p>1. LMF</p>
<p>2. SEAL LMS</p>
<p>3. Other UEs</p></td>
<td>NEF APIs, MEC APIs</td>
<td>Support for sidelink positioning</td>
</tr>
</tbody>
</table>
