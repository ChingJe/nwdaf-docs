---
spec: TS 23.222
version: 20.1.0
release: '20'
clause: Annex H
title: 'Annex H (informative): AEF instantiation'
source_archive: 23222-k10.zip
source_document: 23222-k10.docx
content_origin: 3gpp-source
---

# Annex H (informative): AEF instantiation

As illustrated in figure H-1, CAPIF architecture supports AEFs from multiple API providers (e.g. MNO, 3<sup>rd</sup> party). The instances of such AEFs are managed by the respective management systems of the API provider (e.g. PLMN management system, 3<sup>rd</sup> party management system).

The AMF monitors the events reported by the CAPIF core function. These events can be utilized to trigger AEF instantiation by the PLMN management system or 3<sup>rd</sup> party management system. The AMF can utilize the services of PLMN or 3<sup>rd</sup> party management systems (e.g. trigger instantiation of corresponding AEFs) via the APIs exposed by the PLMN or 3<sup>rd</sup> party management systems. It is upto implementation of CAPIF to utilize the services of the PLMN and/or 3<sup>rd</sup> party management systems for AEF instantiation.

![](assets/rendered/image85.png)

Figure H-1: AEF instantiation
