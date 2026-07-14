---
spec: TS 23.288
version: 18.13.0
release: '18'
clause: Annex A
title: 'Annex A (informative): Methods to handle NAT on IPv4 between UE and AF'
source_archive: 23288-id0.zip
source_document: 23288-id0.docx
source_archive_sha256: 7116a1f4123dcf949818ab56feb6b242eda8819877346ab045654e1d0d8c36aa
source_document_sha256: 5e4ca6e4e350f62a5d4e4443d36ae93c9df679e1e9aea20ccaa44a4b0eb3e196
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex A (informative): Methods to handle NAT on IPv4 between UE and AF


## A.1 Methods to handle NAT on IPv4 between UE and AF


The following methods can be used to handle the case when there is a NAT between the UE and AF for data collection:

NOTE: These methods can be used both when there is a NAT between the UE and the AF, and when there is no NAT between the UE and the AF.

1\) Use IPv6 instead of IPv4 and then use any of the procedures in clauses 6.2.8.2.4.2 to 6.2.8.2.4.4.

2\) Provide GPSI via header enrichment as described in TS 29.244 \[17\].

3\) Have GPSI as part of the authentication information, or via in-band signalling.

4\) At the establishment of the user plane connection between the UE Application and AF, the AF can use the procedure in clause 4.15.10 of TS 23.502 \[3\] to get the GPSI.

5\) At the establishment of the user plane connection between the UE Application and a trusted AF, the AF can use the steps 3 to 8 in clause 4.15.10 of TS 23.502 \[3\], where NEF is replaced by the AF, to retrieve the SUPI of the UE.

In methods 2) to 4), the AF can correlate the UE public IP address and port with the SUPI/GPSI.
