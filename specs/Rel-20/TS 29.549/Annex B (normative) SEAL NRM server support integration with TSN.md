---
spec: TS 29.549
version: 20.1.0
release: '20'
clause: Annex B
title: 'Annex B (normative): SEAL NRM server support integration with TSN'
source_archive: 29549-k10.zip
source_document: '29549-k10_0_cover.docx, 29549-k10_1_Main-Body_s00_s06.docx, 29549-k10_2_Main-Body_s07_s09.docx, 29549-k10_3_Annexes_sA_sHistory.docx'
content_origin: 3gpp-source
---

# Annex B (normative): SEAL NRM server support integration with TSN

When the SEAL Network Resource Management (NRM) server act as a TSN AF, the NRM server shall support integration with TSN including 5GS Bridge information reporting as defined in clause 14.3.8.2 of 3GPP TS 23.434 \[2\] and 5GS Bridge configuration as defined in clause 14.3.8.3 of 3GPP TS 23.434 \[2\].

The 5GS integration with TSN only support fully-centralized model as defined in IEEE Std 802.1Qcc-2018 \[29\], the NRM server acts as a TSN AF as defined in clause 14.2.2.2 of 3GPP TS 23.434 \[2\], shall support the TSN bridge information report as defined in clause 14.3.2.29 of 3GPP TS 23.434 \[2\], TSN bridge information confirmation as defined in clause 14.3.2.30 of 3GPP TS 23.434 \[2\], TSN bridge configuration request as defined in clause 14.3.2.31 of 3GPP TS 23.434 \[2\] and TSN bridge configuration response as defined in clause 14.3.2.32 of 3GPP TS 23.434 \[2\]. TSN CNC (as defined in IEEE 802.1Qcc \[29\]) via the NRM-S reference point configures the TSN flows in the 5GS. As a TSN AF, the SEAL NRM server shall interact with the 5GS PCF over the N5 reference point to configure the 5G QoS and TSCAI parameters in 5GS as defined in clause 14.2.2.24 of 3GPP TS 29.514 \[30\].
