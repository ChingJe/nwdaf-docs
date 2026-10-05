---
spec: TS 23.222
version: 20.1.0
release: '20'
clause: Annex D
title: 'Annex D (informative): CAPIF relationship with external API frameworks'
source_archive: 23222-k10.zip
source_document: 23222-k10.docx
content_origin: 3gpp-source
---

# Annex D (informative): CAPIF relationship with external API frameworks

This annex provides the relationship of CAPIF with the OMA Network APIs and the ETSI MEC API framework. The relationship of CAPIF with these external API frameworks is illustrated in the table D-1. "Yes" means that the external API framework supports the CAPIF functionality, "No" means that the API framework does not support the CAPIF functionality, and "Partial" means that it provides a mechanism that partially supports the CAPIF functionality.

Table D-1: CAPIF relationship with external API frameworks

<table>
<colgroup>
<col style="width: 28%" />
<col style="width: 12%" />
<col style="width: 24%" />
<col style="width: 13%" />
<col style="width: 21%" />
</colgroup>
<thead>
<tr class="header">
<th rowspan="2"><p>CAPIF functionalities</p></th>
<th colspan="2">OMA Network APIs</th>
<th colspan="2">ETSI MEC API framework</th>
</tr>
<tr class="odd">
<th>Supported</th>
<th>Reference</th>
<th>Supported</th>
<th>Reference</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Publish and discover service API information</td>
<td><p>Partial</p>
<p>(see NOTE)</p></td>
<td>OMA-TS-NGSI_Registration_and_Discovery [11]</td>
<td>Yes</td>
<td>ETSI GS MEC 011 [7]</td>
</tr>
<tr class="even">
<td>Topology hiding of the service</td>
<td>Yes</td>
<td>Individual API exposing function</td>
<td>Yes</td>
<td>Individual API exposing function</td>
</tr>
<tr class="odd">
<td>API invoker authentication to access service APIs</td>
<td>Partial</td>
<td>OMA-ER_Autho4API [9]</td>
<td>Partial</td>
<td>ETSI GS MEC 009 [8]</td>
</tr>
<tr class="even">
<td>API invoker authorization to access service APIs</td>
<td>Partial</td>
<td>OMA-ER_Autho4API [9]</td>
<td>Partial</td>
<td>ETSI GS MEC 009 [8]</td>
</tr>
<tr class="odd">
<td>Lifecycle management of service APIs</td>
<td>No</td>
<td></td>
<td>No</td>
<td></td>
</tr>
<tr class="even">
<td>Monitoring service API invocations</td>
<td>No</td>
<td></td>
<td>No</td>
<td></td>
</tr>
<tr class="odd">
<td>Logging API invoker onboarding and service API invocations</td>
<td>No</td>
<td></td>
<td>No</td>
<td></td>
</tr>
<tr class="even">
<td>Auditing service API invocations</td>
<td>No</td>
<td></td>
<td>No</td>
<td></td>
</tr>
<tr class="odd">
<td>Onboarding API invoker to CAPIF</td>
<td>No</td>
<td></td>
<td>No</td>
<td></td>
</tr>
<tr class="even">
<td>CAPIF authentication of API invokers</td>
<td>No</td>
<td></td>
<td>No</td>
<td></td>
</tr>
<tr class="odd">
<td>Service API access control</td>
<td>Partial</td>
<td>OMA-ER_Autho4API [9]</td>
<td>Partial</td>
<td>ETSI GS MEC 009 [8]</td>
</tr>
<tr class="even">
<td>Secure API communication</td>
<td>Yes</td>
<td>OMA-ER_Autho4API [9]</td>
<td>Yes</td>
<td>ETSI GS MEC 009 [8]</td>
</tr>
<tr class="odd">
<td>Policy configuration</td>
<td>No</td>
<td></td>
<td>No</td>
<td></td>
</tr>
<tr class="even">
<td>API protocol stack model</td>
<td>Partial</td>
<td>for REST: OMA-TS_REST_NetAPI_Common [10]</td>
<td>Partial</td>
<td>for REST:<br />
ETSI GS MEC 009 [8]</td>
</tr>
<tr class="odd">
<td>API security protocol</td>
<td>Partial</td>
<td>OMA-ER_Autho4API [9]</td>
<td>Partial</td>
<td>ETSI GS MEC 009 [8]</td>
</tr>
<tr class="even">
<td>CAPIF support for service APIs from multiple providers</td>
<td>No</td>
<td></td>
<td>No</td>
<td></td>
</tr>
<tr class="odd">
<td colspan="5">NOTE: OMA-TS-NGSI_Registration_and_Discovery [11] is only applicable to a specific type of web services (OWSER using UDDI and WSDL).</td>
</tr>
</tbody>
</table>
