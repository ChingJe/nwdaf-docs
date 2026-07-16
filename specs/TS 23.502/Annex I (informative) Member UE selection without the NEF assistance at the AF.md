---
spec: TS 23.502
version: 18.14.0
release: '18'
clause: Annex I
title: 'Annex I (informative): Member UE selection without the NEF assistance at the
  AF'
source_archive: 23502-ie0.zip
source_document: 23502-ie0.docx
source_archive_sha256: 0f24045c49ddf710fb91e51ec74d070cfd13d1e9bafc07809d7e87eae6a61ce8
source_document_sha256: 89536df6095fba3b9c90f1fa06ceaec918dfc6b0fd26a769a06eae7db66bb603
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex I (informative): Member UE selection without the NEF assistance at the AF

This informative Annex describes an example of the procedure that AF selects the FL members by collecting network exposure information in case that no NEF is present in the 5GS. In this example, QoS Monitoring is used for FL Member UE selection.

Network exposure information as described in clause 4.15 of TS 23.502 \[4\], e.g. UE location reporting from the AMF, user plane information from the UPF and data analytics from NWDAF may be collected and used to assist the AF in application layer Member UE selection e.g. assist in the selection of Member UEs participating in a federating learning operation.

![](assets/original/image279.emf)

Figure I-1: Example of Procedure for Member UE selection without the NEF assistance

1\. \[Optional\] The AF requests the location reporting of the UEs from the AMF by invoking existing Namf_EventExposure_Subscribe (Location Reporting).

2\. \[Optional\] The AF initiates direct notification of QoS Monitoring procedure for delay information for the UEs in the candidate list, as defined in steps 1a-5 of clause 6.4.2.1 of TS 23.548 \[74\].

3\. \[Optional\] The AF requests user plane information, e.g. Throughput UL/DL, Packet transmission, Packet retransmission, for the UEs in the candidate list from UPF.

4\. \[Optional\] The AF requests analytics from NWDAF by invoking the Nnwdaf_AnalyticsSubscription_Subscribe service operation, such as UE Communication, User Data Congestion Analytics, WLAN performance analytics per UE, etc. as defined in TS 23.288 \[50\].

NOTE 1: In Steps 1-4, the AF maintains the candidate list itself and collects the Member UE selection information directly from the 5GC and includes the Target UE Identifier(s) for the UEs in the candidate list in the request.

5\. The AF selects members, e.g. for application layer Member UE selection for FL, based on the information collected in steps 1-4.

NOTE 2: What information needs to be collected by the AF to perform Member UE selection is decided by the AF itself.

NOTE 3: AF may keep monitoring the information from AMF/UPF/SMF/NWDAF to update the members.
