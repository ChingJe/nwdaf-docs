---
spec: TS 23.502
version: 18.14.0
release: '18'
clause: Annex H
title: 'Annex H (normative): Support of EAP-based secondary authentication and authorization
  by DN-AAA over EPC'
source_archive: 23502-ie0.zip
source_document: 23502-ie0.docx
source_archive_sha256: 0f24045c49ddf710fb91e51ec74d070cfd13d1e9bafc07809d7e87eae6a61ce8
source_document_sha256: 89536df6095fba3b9c90f1fa06ceaec918dfc6b0fd26a769a06eae7db66bb603
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex H (normative): Support of EAP-based secondary authentication and authorization by DN-AAA over EPC

## H.1 Introduction

Secondary authentication/authorization by a DN-AAA Server during the establishment of a PDN connection over 3GPP access to EPC, is supported based on following principles:

\- A SMF+PGW-C shall be used to serve DNN(s) requiring secondary authentication/authorization by a DN-AAA Server.

\- For secondary authentication/authorization by a DN-AAA Server, the SMF+PGW-C runs the same procedures with PCF, UDM and DN-AAA and uses the same corresponding interfaces, as defined in clause 4.3.2, regardless of whether the UE is served by EPC or 5GC.

\- If the UE has included the PDU Session ID in PCO, the UE may indicate in the PCO within the PDN connection establishment request its support for EAP-based secondary authentication and authorization by DN-AAA over EPC. The UE may also include the DN-specific identity in the PCO. The SMF+PGW-C may reject the PDN connection establishment if the UE does not support EAP-based secondary authentication and authorization by DN-AAA over EPC while local policies tell that secondary authentication and authorization by DN-AAA is mandatory to access to the DN. When a PDU Session is established, the UE may also indicate via 5GSM capability parameter that it supports secondary DN authentication and authorization over EPC.

\- The interface towards the UE is different (usage of EPC NAS instead of 5GC NAS) between the EPC and 5GC cases.

\- The MME and SGW are not impacted by the procedure. Specific exchanges between the UE and the SMF+PGW-C for secondary authentication/authorization by a DN-AAA Server are carried via PCO. This includes the support of EAP exchanges between the UE and the DN-AAA Server.

\- As it is only possible to exchange PCO once between the UE and the PGW during PDN connection establishment, the PDN connection is established before EAP-based secondary authentication/authorization by a DN-AAA Server takes place.

\- When secondary authentication/authorization by a DN-AAA Server has successfully taken place, the SMF+PGW-C allows traffic exchange at the UPF and indicates to the UE that User plane traffic is now possible.

## H.2 Procedures

### H.2.1 Secondary authentication and authorization by DN-AAA at PDN Connection Establishment

In the figure H.2.1-1, the execution of the secondary authentication and authorization by DN-AAA is specified. The procedure assumes that:

\- The APN is associated with the selection of a SMF+PGW-C to serve APN(s) that require secondary authentication and authorization by DN-AAA at PDN connection establishment.

\- The SMF+PGW-C is configured with local policies indicating that the APN requires secondary authentication and authorization by DN-AAA at PDN connection establishment.

![](assets/original/image278.emf)

Figure H.2.1-1: EAP-based secondary authentication and authorization by DN-AAA at PDN connection establishment

0\. As steps 1 - 13 of Figure 5.3.2.1-1 in TS 23.401 \[13\] (Attach Request) or as steps 1 to 3 of Figure 5.10.2 in TS 23.401 \[13\] (UE requested PDN connectivity) with following modifications: The UE may indicate in PCO its capability to support EAP-based secondary DN authentication over EPC if the UE included the PDU Session ID in PCO. The UE may also include the DN-specific identity.

1\. The SMF+PGW-C gets subscription data from UDM as defined in step 4 of Figure 4.3.2.2.1-1 (not shown in Figure H.2.1-1). The procedure assumes that SMF configuration or subscription data from UDM require EAP-based secondary authentication and authorization by DN-AAA.

Secondary DN authorization may be invoked as described in TS 29.561 \[63\]. During this step the DN-AAA may provide an IP address for the UE and other DN authorization data as described in clause 5.6.6 of TS 23.501 \[2\].

2a. If dynamic PCC is to be used for the PDU Session, the SMF+PGW-C performs an SM Policy Association Establishment procedure as defined in clause 4.16.4 and if Secondary DN authorization has been invoked in step 1, provides to the PCF the PDN Connection parameters received from the DN AAA at step 1 as described in step 5 of Figure 4.3.2.3-1. In this step the SMF+PGW-C may retrieve the PDU Session related policy information and the PCC rule(s) from the PCF, e.g. the authorized Session AMBR.

2b. UPF selection and N4 session establishment is executed with the difference that the SMF+PGW-C configures the UPF+PGW-U to block any UE traffic over the PDN Connection (until the Secondary DN authentication and authorization has been done and is successful).

3\. Steps 15-24 in Figure 5.3.2.1-1 of TS 23.401 \[13\] or steps 5-16 in Figure 5.10.2 of TS 23.401 \[13\].

During the Attach procedure, at step 15 in Figure 5.3.2.1-1 of TS 23.401 \[13\] or during UE requested PDN connectivity in step 5 in Figure 5.10.2 of TS 23.401 \[13\], the SMF+PGW-C includes in PCO, an Indication to the UE that "UpLink Data is NOT ALLOWED" on the PDN connection. The UE shall not send Uplink data to the network, until it receives an indication further from the network that "UpLink Data is ALLOWED".

NOTE: How the Indication that Uplink data allowed/not allowed is carried in PCO is defined in TS 24.501 \[25\].

4\. \[Conditional\] The PGW-C+SMF initiates EAP-based authentication by sending EAP-Request as described in step 2 of Figure 4.3.2.3-1.

5\. Multiple round-trip messages as required by the authentication method used by DN-AAA may follow. The PCO including the authentication message from the DN-AAA is transferred to the UE by the SMF+PGW-C in Update Bearer Request and then over S1 by Downlink NAS Transport (steps 4b-4d). The response from the UE is transferred to the SMF+PGW-C in an Uplink NAS Transport over S1 and Update Bearer Response (steps 4e-4g) over EPS.

6\. Secondary authentication and authorization by DN-AAA procedure continues as described in step 4 of Figure 4.3.2.3-1.

7\. The SMF+PGW-C updates the N4 rules in the UPF+PGW-U to allow traffic over the PDN Connection. If dynamic PCC is to be used for the PDU Session and the SMF+PGW-C received DN Authorization information from the DN-AAA as part of step 5 or 6 that is different compared to the value received in step 2, the SMF+PGW-C contacts the PCF to update the PDN Connection as described in step 5 of Figure 4.3.2.3-1

8\. The SMF+PGW-C updates the UE by invoking the PDN GW initiated bearer modification without QoS update procedure (figure 5.4.3-1 of TS 23.401 \[13\]) initiated by sending an Update Bearer Request message to the SGW. The PCO includes an indication that "UpLink Data is ALLOWED". The UE confirms the update (see clause 5.4.3 of TS 23.401 \[13\]).

If the UE IP address is to be delivered to the UE over user plane (via Router advertisement or DHCP) then the UE IP address is only delivered to the UE after step 8.

9\. As in step 6 of Figure 4.3.2.3-1.

The DN-AAA Server may revoke the authorization for a PDN connection or update DN authorization data for a PDN connection. According to the request from DN-AAA Server, the SMF+PGW-C may release or update the PDN connection.

At any time after the PDN connection establishment, the DN-AAA Server or SMF+PGW-C may initiate Secondary Re-authentication procedure for the PDN connection as described in clause 4.3.2.3. Steps 4a-4h are performed to transfer the Secondary Re-authentication message between the DN-AAA Server and the UE. The Secondary Re-authentication procedure may start from step 4a (DN-AAA initiated Secondary Re-authentication procedure) or step 4b (SMF+PGW-C initiated Secondary Re-authentication procedure).

During Secondary Re-authentication, if the SMF+PGW-C receives an indication from the MME that the UE is unreachable then it informs the DN-AAA Server that UE is not reachable for re-authentication. Based on this indication from SMF+PGW-C, the DN-AAA Server may decide to keep the PDN connection or request to release it.

DN-AAA may initiate DN-AAA Re-authorization without performing re-authentication based on local policy. DN-AAA Re-authorization procedure may involve steps 5 and 6 of Figure H.2.1-1 above.

During Secondary Re-authentication/Re-authorization, if the SMF+PGW-C receives DN Authorization Profile Index and/or DN authorized Session AMBR, the SMF+PGW-C reports the received value(s) to the PCF (as described in TS 23.501 \[2\]) by triggering the Policy Control Request Trigger as described in TS 23.503 \[20\].
