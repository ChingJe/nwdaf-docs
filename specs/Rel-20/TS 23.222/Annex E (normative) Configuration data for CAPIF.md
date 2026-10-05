---
spec: TS 23.222
version: 20.1.0
release: '20'
clause: Annex E
title: 'Annex E (normative): Configuration data for CAPIF'
source_archive: 23222-k10.zip
source_document: 23222-k10.docx
content_origin: 3gpp-source
---

# Annex E (normative): Configuration data for CAPIF

The configuration data is stored in the CAPIF core function and provided by the CAPIF administrator.

The configuration data for CAPIF is specified in table E-1.

Table E-1: Configuration data for CAPIF

<table>
<colgroup>
<col style="width: 35%" />
<col style="width: 64%" />
</colgroup>
<thead>
<tr class="header">
<th>Reference</th>
<th>Parameter description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td rowspan="4">Subclause 4.2.2</td>
<td>List of published service API discovery restrictions</td>
</tr>
<tr class="even">
<td>&gt; Service API identification</td>
</tr>
<tr class="odd">
<td>&gt; API invoker identity information</td>
</tr>
<tr class="even">
<td>&gt; Service API category</td>
</tr>
<tr class="odd">
<td rowspan="5">Subclause 4.3.2</td>
<td>List of service API authorization restrictions per API invoker</td>
</tr>
<tr class="even">
<td>&gt; API Invoker identity information</td>
</tr>
<tr class="odd">
<td>&gt; Service API identification</td>
</tr>
<tr class="even">
<td><p>&gt; Per resource, operation authorization information.</p>
<p>(see NOTE 2)</p></td>
</tr>
<tr class="odd">
<td>&gt; Network slice identification</td>
</tr>
<tr class="even">
<td rowspan="3">Subclause 4.7.2</td>
<td>List of service API log storage durations</td>
</tr>
<tr class="odd">
<td>&gt; Service API identification</td>
</tr>
<tr class="even">
<td>&gt; Service API log storage duration (in hours) (see NOTE 1)</td>
</tr>
<tr class="odd">
<td rowspan="3">Subclause 4.7.4</td>
<td>List of API invoker interactions log storage durations</td>
</tr>
<tr class="even">
<td>&gt; Service API identification</td>
</tr>
<tr class="odd">
<td>API invoker interactions log storage duration (in hours) (see NOTE 1)</td>
</tr>
<tr class="even">
<td rowspan="7">Subclause 4.10</td>
<td>List of access control policy (for an API provider domain) per API invoker and optionally per network slice</td>
</tr>
<tr class="odd">
<td>&gt; Volume limit on service API invocations (total number of invocations allowed)</td>
</tr>
<tr class="even">
<td>&gt; Time limit on service API invocations (The time range of the day during which the service API invocations are allowed)</td>
</tr>
<tr class="odd">
<td>&gt; Rate limit on service API invocations (allowed service API invocations per second)</td>
</tr>
<tr class="even">
<td>&gt; Service API identification</td>
</tr>
<tr class="odd">
<td>&gt; API invoker identity information</td>
</tr>
<tr class="even">
<td>&gt; Network Slice Info</td>
</tr>
<tr class="odd">
<td rowspan="4">Subclause 8.34.3 and 8.34a.3</td>
<td>Group context information</td>
</tr>
<tr class="even">
<td>&gt; Group identifier</td>
</tr>
<tr class="odd">
<td>&gt; List of UE IDs (group members)</td>
</tr>
<tr class="even">
<td>&gt; UE identifier of the Group Resource Owner</td>
</tr>
<tr class="odd">
<td></td>
<td>&gt; Authorization information related to the list of supported applications</td>
</tr>
<tr class="even">
<td></td>
<td>&gt;&gt; Application identifier</td>
</tr>
<tr class="odd">
<td></td>
<td>&gt;&gt; Purpose</td>
</tr>
<tr class="even">
<td></td>
<td>&gt;&gt; Scope</td>
</tr>
<tr class="odd">
<td rowspan="2">Subclause 8.38</td>
<td>List of service API allowed for dissemination during Open Discovery</td>
</tr>
<tr class="even">
<td>&gt; Service API identification</td>
</tr>
<tr class="odd">
<td rowspan="5">Subclause 8.42.3</td>
<td>List of exposure policies</td>
</tr>
<tr class="even">
<td>&gt; Roaming policy</td>
</tr>
<tr class="odd">
<td>&gt;&gt; List of allowed VPLMN ID (NOTE 3)</td>
</tr>
<tr class="even">
<td>&gt;&gt; Service API identification (optional)</td>
</tr>
<tr class="odd">
<td>&gt;&gt; API invoker identity information (optional)</td>
</tr>
<tr class="even">
<td colspan="2"><p>NOTE 1: If no value is set for the duration, the duration is assumed to be unlimited.</p>
<p>NOTE 2: The API invoker request for access authorization to specific resources or operations of a Service API is specified in 3GPP TS 33.122 [12].</p>
<p>NOTE 3: When the target UE is roaming in a VPLMN included in the list of allowed VPLMN IDs, the service API can be authorized for the indicated API invoker</p></td>
</tr>
</tbody>
</table>
