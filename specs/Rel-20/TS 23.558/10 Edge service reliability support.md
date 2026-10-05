---
spec: TS 23.558
version: 20.3.0
release: '20'
clause: 10
title: 10 Edge service reliability support
source_archive: 23558-k30.zip
source_document: 23558-k30.docx
content_origin: 3gpp-source
---

# 10 Edge service reliability support


## 10.1 General

By adopting the NF set and NF service set approach, as specified in clause 5.21.3 of 3GPP TS 23.501 \[2\], edge server set enable edge service producers (e.g. EES/ECS) in the data network to provide reliable services to the edge service consumers (e.g. EAS, EEC).

Equivalent edge servers may be grouped into edge server sets, e.g. several EES instances are grouped into an EES set. Edge servers within an edge server set are interchangeable because they share the same context data, and may be deployed in different locations, e.g. different data centres.

NOTE 1: Determining equivalent EESs (e.g. based on KPIs, service area) to group into an EES set (e.g. by O&M or other means) is not specified in the present document.

UE is not required to host multiple instances of the same EEC, but the EEC is able to perform ECS/EES re-selection due to ECS/EES service failure.

NOTE 2: EEC performing ECS/EES re-selection due to ECS/EES service failure is not specified in the present document.
