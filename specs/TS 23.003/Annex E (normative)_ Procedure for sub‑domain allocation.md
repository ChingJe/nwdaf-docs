---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: Annex E
title: 'Annex E (normative): Procedure for sub‑domain allocation'
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex E (normative): Procedure for sub‑domain allocation

When a 3GPP member company identifies the need for a new sub‑domain name of ".3gppnetwork.org", that 3GPP member company shall propose a CR to this specification at the earliest available meeting of the responsible working group for this TS. The CR shall propose a new sub‑domain name. The new sub‑domain proposed shall be formatted in one of the formats as described in the following table.

<table>
<colgroup>
<col style="width: 56%" />
<col style="width: 43%" />
</colgroup>
<thead>
<tr class="header">
<th>Sub‑domain Format</th>
<th>Intended Usage</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>&lt;service_id&gt;.mnc&lt;MNC&gt;.mcc&lt;MCC&gt;.3gppnetwork.org</p>
<p>(see notes 1 and 2)</p></td>
<td>Domain name that is to be resolvable by network nodes only. This format inherently adds protection to the identified node, in that attempted DNS resolutions instigated directly from end user equipment will fail indefinitely.</td>
</tr>
<tr class="even">
<td><p>&lt;service_id&gt;.mnc&lt;MNC&gt;.mcc&lt;MCC&gt;.pub.3gppnetwork.org</p>
<p>(see notes 1 and 2)</p></td>
<td>Domain name that is to be resolvable by UEs and/or network nodes. This format inherently adds global resolution capability, but at the expense of confidentiality of network topology.</td>
</tr>
<tr class="odd">
<td><p>&lt;service_id&gt;.mnc&lt;MNC&gt;.mcc&lt;MCC&gt;.ipxuni.3gppnetwork.org</p>
<p>(see notes 1 and 2)</p></td>
<td><p>Domain name for UNI interface that is to be resolvable by UEs that are connected to an inter-PLMN IP network that has no connectivity to the Internet.</p>
<p>This format inherently adds resolution capability for UEs in closed IP networks e.g. IPX.</p></td>
</tr>
<tr class="even">
<td><p>&lt;service_id&gt;.mcc&lt;MCC&gt;.visited-country.pub.3gppnetwork.org</p>
<p>(see notes 1 and 2)</p></td>
<td>Domain name in the visited country that is to be resolvable by UEs and/or network nodes, which is not specific to an individual operator.</td>
</tr>
<tr class="odd">
<td><p>&lt;service_id&gt;.nid&lt;NID&gt;.mnc&lt;MNC&gt;.mcc&lt;MCC&gt;.3gppnetwork.org</p>
<p>(see notes 1 and 3)</p></td>
<td>Domain name of a Stand-alone non-public network that is to be resolvable by network nodes only. This format inherently adds protection to the identified node, in that attempted DNS resolutions instigated directly from end user equipment will fail indefinitely.</td>
</tr>
</tbody>
</table>

Table E.1: Sub‑domain formats for the "3gppnetwork.org" domain and their respective intended usage

NOTE 1: "\<service_ID\>" is a chosen label, conformant to DNS naming conventions (usually IETF RFC 1035 \[19\] and IETF RFC 1123 \[20\]) that clearly and succinctly describe the service and/or operation that is intended to use this sub‑domain.

NOTE 2: "\<MNC\>" and "\<MCC\>" are the MNC (padded to the left with a zero, if only a 2‑digit MNC) and MCC of a PLMN.

NOTE 3: "NID", "\<MNC\>" and "\<MCC\>" are the NID (hexadecimal digits as specified in clause 12.7), MNC (padded to the left with a zero, if only a 2‑digit MNC) and MCC identifying a Stand-alone Non-Public Network (SNPN).

Care should be taken when choosing which format a domain name should use. Once a format has been chosen, the responsible working group shall then check the CR and either endorse it or reject it. If the CR is endorsed, then the responsible working group shall send an LS to the GSMA NG with TSG-CT in copy. The LS shall describe the following key points:

\- the context

\- the service

\- intended use

\- involved actors

\- proposed new sub‑domain name

GSMA NG will then verify the consistence of the proposal and its usage within the domain's structure and interworking rules (e.g. access to the GRX/IPX Root DNS servers). GSMA NG will then endorse or reject the proposal and inform the responsible working group (in 3GPP) and also TSG CT. It is possible that GSMA NG will also specify, changes to the newly proposed sub‑domain name (e.g. due to requested sub‑domain name already allocated).

NOTE 4: There is no need to request GSMA NG for new labels to the left of an already GSMA NG approved "\<service_ID\>". It is the responsibility of the responsible working group to ensure uniqueness of such new labels.

It should be noted that services already defined to use the ".gprs" domain name will continue to do so and shall not use the new domain name of ".3gppnetwork.org"; this is to avoid destabilising services that are already live.
