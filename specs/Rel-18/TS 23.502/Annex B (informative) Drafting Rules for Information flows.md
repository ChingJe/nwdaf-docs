---
spec: TS 23.502
version: 18.14.0
release: '18'
clause: Annex B
title: 'Annex B (informative): Drafting Rules for Information flows'
source_archive: 23502-ie0.zip
source_document: 23502-ie0.docx
content_origin: 3gpp-source
---

# Annex B (informative): Drafting Rules for Information flows

The following drafting rules are recommended for information flows specified in this specification in order to ensure that the Control Plane network functions can be supported with service based interfaces:

1\. Information flows should describe the end to end functionality. NF services in clause 5 shall only be derived from the information flows in clause 4.

2\. Information flows should strive to use type of interactions such as REQUEST/RESPONSE (e.g. location request, location response), SUBSCRIBE/NOTIFY between Core CP NFs. Any other type of interactions described should have justifications for its use.

3\. Information flows should also ensure readability thus the semantics of the REQUEST/RESPONSE should still be maintained (for instance, we need to indicate PDU Session request, PDU Session response and Subscribe for UE location reporting/Notify UE location reporting) for readers and developers to understand the need for a certain transaction.

NOTE: As stated in TS 23.501 \[2\], service based interface is not supported for N1, N2, N4. Thus, the rules are not meant for those interfaces.
