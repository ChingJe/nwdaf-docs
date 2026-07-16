---
spec: TS 23.502
version: 18.14.0
release: '18'
clause: Annex J
title: 'Annex J (informative): Support for Personal IoT Networks'
source_archive: 23502-ie0.zip
source_document: 23502-ie0.docx
source_archive_sha256: 0f24045c49ddf710fb91e51ec74d070cfd13d1e9bafc07809d7e87eae6a61ce8
source_document_sha256: 89536df6095fba3b9c90f1fa06ceaec918dfc6b0fd26a769a06eae7db66bb603
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex J (informative): Support for Personal IoT Networks

## J.1 Procedure for PIN service

![](assets/original/image280.emf)

Figure J.1-1: Procedure for PIN service

PIN is a subscribed service, and a user needs to coordinate with the Mobile Network Operator to subscribe for PIN service. When a user subscribes for a PIN, the subscription data includes the (DNN, S NSSAI) combination allocated by the MNO for the PIN service. The PEGC(s) are then provisioned with appropriate URSP rules to enable the PEGC UE to route the PIN traffic using the (DNN, S NSSAI) combination allocated for the PIN. Figure J.1-1 provides a high level procedure for PIN service.

Step 1: Step 1 is performed using O&M.

A user subscribes to the Mobile Network Operator (MNO) for PIN service. The user provides the list of PEGC(s) that are part of the PIN. The MNO verifies the request, performs necessary checks e.g. whether the UEs are allowed to act as PEGC, whether all the requested PEGC are part of the same UDM group etc. If the request is authorized by the MNO, the MNO:

\- allocates a dedicated (DNN, S NSSAI) combination for the PIN;

\- if the PIN has a single PEGC, then updates the PEGC subscription with the (DNN, S NSSAI) combination allocated for the PIN;

\- if the PIN has more than one PEGC and 5G VN Group is used for a PIN, then creates a group subscription following the 5G VN group management principles as specified in clause 5.29.2 of TS 23.501 \[2\]. The information on the External Group ID and associated (DNN, S-NSSAI) combination is provided to the AF for PIN;

\- if local switching is required, configures in the SMF set and/or in the NRF that the DNN allocated for the PIN is served by a specific SMF set.

NOTE: It is assumed that all PEGCs that are members of a PIN are part of the same UDM Group ID. If the PEGCs requested by the user for PIN creation are not part of the same UDM Group ID, the MNO migrates all the PEGCs into a single UDM Group ID for creating the group subscription.

Step 2: For routing PIN traffic by the PEGC, the AF for PIN provides guidance for URSP generation to the 5GC. The AF for PIN uses a UE ID (i.e. GPSI) as the target UE if the PIN contains a single PEGC. If the PIN contains more than one PEGC and 5G VN Group is used for a PIN, then the AF uses External Group ID as the target UEs for providing URSP guidance to the 5GC. The AF request contains (DNN, S NSSAI) combination allocated to the user for the PIN service and the traffic descriptor components in the URSP rule request from the AF for PIN contains the PIN ID.

The NEF authorizes the request received from the AF for PIN and stores the information in the UDR as "Application Data".

The NEF can use the procedure for authorization of service specific parameter provisioning as specified in clause 4.15.6.7a to authorize the AF request by the UDM. In this case:

\- if the request is for an individual UE, the UDM checks if the (DNN, S NSSAI) combination in the AF request is allowed for the UE;

\- if the request is for a group of UEs and 5G VN Group is used for a PIN, the UDM checks whether the group related data (e.g. (DNN, S-NSSAI) combination group related data, see table 4.15.6.3b-1) is authorized for the group.

If the AF request is authorized, the NEF stores the AF requested information in the UDR as the "Application Data" (Data Subset setting to "Service specific information").

Step 3: The PCF receives a Nudr_DM_Notify notification of data change from the UDR, generates the URSP rules and initiates UE Policy delivery as specified in clause 4.2.4.3 to provision the URSP rules in the PEGC(s). For routing of PIN traffic by the PEGC(s), the URSP policies provided to the PEGC UE(s) contain URSP rule with PIN ID as traffic descriptor.

Step 4: The AF for PIN provides QoS requirements for the PIN traffic following procedures for AF requested QoS for a UE or group of UEs not identified by a UE address as specified in clause 4.15.6.14.

Step 5: When the PEGC(s) detect PIN traffic, it uses the provisioned URSP rules to identify PDU session to route the traffic as specified in clause 6.6.2.3 of TS 23.503 \[20\]. The 5GC further performs session management and user plane management as described in Annex P, clause P.2 of TS 23.501 \[2\].

When 5G VN Group is not used for a PIN and if the PIN contains more than one PEGCs, then the AF request for URSP guidance and QoS requirements is targeted to each individual PEGCs that are part of the PIN.
