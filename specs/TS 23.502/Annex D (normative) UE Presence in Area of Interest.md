---
spec: TS 23.502
version: 18.14.0
release: '18'
clause: Annex D
title: 'Annex D (normative): UE Presence in Area of Interest'
source_archive: 23502-ie0.zip
source_document: 23502-ie0.docx
source_archive_sha256: 0f24045c49ddf710fb91e51ec74d070cfd13d1e9bafc07809d7e87eae6a61ce8
source_document_sha256: 89536df6095fba3b9c90f1fa06ceaec918dfc6b0fd26a769a06eae7db66bb603
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex D (normative): UE Presence in Area of Interest

## D.1 Determination of UE presence in Area of Interest by AMF

If the AMF has requested NG-RAN location reporting as specified in clause 4.10 for the Area Of Interest and UE is in CM-CONNECTED state, including RRC_CONNECTED and RRC_INACTIVE state, the AMF determines the UE presence in the Area of Interest as the reported value from the NG-RAN, as specified in clause D.2.

In the case the UE is served by a MBSR, the AMF may consider the additional ULI provided by the NG-RAN node (as defined in clause 5.35A.6 of TS 23.501 \[2\]) when determining the UE presence in Area of Interest.

If the AMF has requested N2 Notification as specified in clause 4.8.3, the AMF determines the UE presence in Area Of Interest as follows, taking N2 Notification from NG-RAN into consideration:

\- IN:

\- if the UE is in CM-CONNECTED with RRC_CONNECTED state and if the last received User Location Information for the UE is inside the Area Of Interest service area; or

NOTE 1: The above is valid e.g. under the condition that Area Of Interest border coincides with NG-RAN node service area border(s).

\- if the UE is in CM-CONNECTED and if the UE is inside a Registration Area which is completely contained within the Area Of Interest.

\- OUT:

\- if the UE is in CM-CONNECTED with RRC_CONNECTED state and if the last received User Location Information for the UE is outside the Area Of Interest; or

NOTE 2: The above is valid e.g. under the condition that Area Of Interest border coincides with NG-RAN node service area border(s).

\- if the UE is in CM-CONNECTED and if UE is inside a Registration Area which does not contain any part of Area Of Interest.

\- UNKNOWN:

if none of above conditions for IN or OUT is met.

Otherwise, AMF determines the UE presence of Area Of Interest as follows:

\- IN:

\- if the UE is inside the Area Of Interest service area and if the UE is in CM-CONNECTED state; or

\- if the Area Of Interest service area is indicated as a RAN node identity and the Parameter Type with value "RAN timing synchronization status change event" is included and the UE has indicated a support for registration update procedure due to RAN timing synchronization status as described in clause 5.3.4.4 of TS 23.501 \[2\] and the most recent N2 connection for the UE is via a RAN Node that is included in the Area Of Interest service area; or

\- if the Parameter Type with value "Adjust AoI based on RA" is included and the UE is inside a Registration Area which contains at least one Tracking Area that is contained within the Area Of Interest; or

\- if the UE is inside a Registration Area which is contained within the Area Of Interest.

\- OUT:

\- if the UE is outside the Area Of Interest in CM-CONNECTED and the Parameter Type with value "Adjust AoI based on RA" is not included; or

\- if the Area Of Interest service area is indicated as a RAN node identity and the Parameter Type with value "RAN timing synchronization status change event" is included and the UE has indicated a support for registration update procedure due to RAN timing synchronization status as described in clause 5.3.4.4 of TS 23.501 \[2\] and the most recent N2 connection for the UE is via a RAN Node that is not included in the Area Of Interest service area; or

\- if UE is inside a Registration Area which does not contain any part of Area Of Interest.

\- UNKNOWN:

if none of above conditions for IN or OUT is met.

## D.2 Determination of UE presence in Area of Interest by NG-RAN

If the AMF has requested for the Area of Interest, NG-RAN determines the UE presence of Area Of Interest as follows:

\- IN:

\- if the UE is inside the Area Of Interest and the UE is in RRC_CONNECTED state; or

\- if the UE is inside an RNA which is completely contained within the Area Of Interest.

\- OUT:

\- if the UE is outside the Area Of Interest in RRC_CONNECTED state; or

\- if UE is inside an RNA which does not contain any part of Area Of Interest.

\- UNKNOWN:

\- if none of above conditions for IN or OUT is met.
