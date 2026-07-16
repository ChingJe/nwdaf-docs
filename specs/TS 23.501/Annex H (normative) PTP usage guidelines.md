---
spec: TS 23.501
version: 18.12.0
release: '18'
clause: Annex H
title: 'Annex H (normative): PTP usage guidelines'
source_archive: 23501-ic0.zip
source_document: 23501-ic0.docx
source_archive_sha256: 800fb58a2a3a8c3eeac7084497de566fc730808923d8eea61b84d2555360debe
source_document_sha256: cdab3840e4d278d381f70806e6458f87dff8e5fd238f370e4f65872bbd5088a6
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex H (normative): PTP usage guidelines

## H.1 General

This Annex provides guidelines on the use of certain specific IEEE parameters and protocol messages in the case of TSN as described in clause 5.27.

## H.2 Signalling of ingress time for time synchronization

The ingress timestamp (TSi) of the PTP event (e.g. Sync) message is provided from the ingress TT (NW-TT/UPF or DS-TT/UE) to the egress TT, if supported in the PTP messages as described in clauses 5.27.1.2.2.1 and 5.27.1.2.2.2 using the Suffix field defined in clause 13.4 of IEEE Std 1588 \[126\]. The structure of the Suffix field follows the recommendation of clause 14.3 of IEEE Std 1588 \[126\], with an organizationId specific to 3GPP, an organizationSubType referring to an ingress timestamp and data field that carries the ingress timestamp encoded as specified in clause 5.3.3 of IEEE Std 1588 \[126\]. TS 24.535 \[117\] specifies the coding of the ingress timestamp in the (g)PTP messages between a DS-TT and a NW-TT.

## H.3 Void

## H.4 Path and Link delay measurements

The procedure described in this clause is applicable if DS-TT and NW-TT support operating as a boundary clock or as a time-aware system or as peer to peer Transparent Clock or end to end Transparent Clock and when the PTP instance in 5GS is configured to operate as a time-aware system or as a Boundary Clock or as peer to peer Transparent Clock or as end to end Transparent Clock. Whether DS-TT/NW-TT support operating as a boundary clock or peer to peer Transparent Clock or end to end Transparent Clock or as a time-aware system (support of the IEEE Std 802.1AS \[104\] PTP profile) may be determined as described in clause K.2.1.

PTP ports in DS-TT and NW-TT may support the following delay measurement mechanisms:

\- Delay request-response mechanism as described in clause 11.3 of IEEE Std 1588 \[126\];

\- Peer-to-peer delay mechanism as defined in clause 11.4 of IEEE Std 1588 \[126\];

\- Common Mean Link Delay Service.

Depending on the measurement mechanisms supported by DS-TT and NW-TT as well as the configured clock mode of 5GS, the PTP ports in DS-TT and NW-TT are configured as follows:

\- PTP ports configured to operate as a time-aware system according to IEEE Std 802.1AS \[104\] may be configured to use the peer-to-peer delay mechanism or Common Mean Link Delay Service;

\- PTP ports configured to operate as a Boundary Clock according to IEEE Std 1588 \[126\] may be configured to use the delay request-response mechanism, the peer-to-peer delay mechanism or Common Mean Link Delay Service.

\- PTP ports in 5GS configured to operate as a peer-to-peer Transparent Clock according to IEEE Std 1588 \[126\] shall use the peer-to-peer delay mechanism.

\- PTP ports in 5GS configured to operate as an end-to-end Transparent Clock according to IEEE Std 1588 \[126\] do not actively participate in path and link measurements mechanisms but shall calculate and add residence time and delay asymmetry information to PTP messages as defined in clause 10.2.2 of IEEE Std 1588 \[126\].

If DS-TT and NW-TT support operating as an end-to-end Transparent Clock, then the residence time for one-step operation as an end-to-end Transparent Clock for the path and link measurements is calculated as follows:

\- Upon reception of a PTP Delay_Req/Pdelay_Req/Pdely_Resp message from the upstream PTP instance, the ingress TT (i.e. NW-TT or DS-TT) makes an ingress timestamping (TSi) using 5G internal system clock for the message.

\- The ingress timestamp is conveyed to the egress TT via the PDU Session as described in clause H.2.

\- The PTP port in the egress TT then creates egress timestamping (TSe) using 5G internal system clock for the PTP message for external PTP network. The difference between TSi and TSe is considered as the calculated residence time spent within the 5G system for this PTP message expressed in 5GS time. If needed, the PTP port in the egress TT convert the calculated resident time in 5GS into the residence time expressed in PTP GM time e.g. by means of the factor as specified in Equation (6) of clause 12.2.2 of IEEE Std 1588 \[126\].

\- The PTP port in the egress TT modifies the payload of the PTP Delay_Req/Pdelay_Req/Pdelay_Resp message that it sends towards the downstream PTP instance as follows:

\- Adds the calculated residence time to the correction field.

\- Removes Suffix field that contains TSi.

If DS-TT and NW-TT support operating as an end-to-end Transparent Clock, then the residence time for two-step operation as an end-to-end Transparent Clock for the path and link measurements is calculated as follows:

\- Upon reception of a PTP Delay_Req/Pdelay_Req/Pdelay_Resp message from the upstream PTP instance, the ingress TT (i.e. NW-TT or DS-TT) makes an ingress timestamping (TSi) using 5G internal system clock for the message.

\- If the ingress TT receives a Pdelay_Resp message with the twoStepFlag set to FALSE, then the ingress TT modifies the twoStepFlag to TRUE and creates a PTP Pdelay_Resp_Follow_Up message.

\- The ingress timestamp is conveyed to the egress TT via the PDU Session as described in clause H.2.

\- The PTP port in the egress TT then creates egress timestamping (TSe) using 5G internal system clock for the PTP message for external PTP network. The difference between TSi and TSe is considered as the calculated residence time spent within the 5G system for this PTP message expressed in 5GS time. If needed, the PTP port in the egress TT converts the calculated residence time in 5GS into the residence time expressed in PTP GM time, e.g. by means of the factor as specified in Equation (6) of clause 12.2.2 of IEEE Std 1588 \[126\]. The egress TT then stores the calculated residence time expressed in PTP GM time and removes Suffix field that contains TSi before sending the PTP Delay_Req/Pdelay_Req/Pdelay_Resp message towards the downstream PTP instance.

\- Upon reception of the PTP Delay_Resp message associated with the PTP Delay_Req, the egress TT for the PTP Delay_Req message (i.e. the ingress TT for the PTP Delay_Resp message) modifies the payload of the PTP Delay_Resp message that it sends towards the ingress TT of the PTP Delay_Req message (i.e. egress TT for the PTP Delay_Resp message) as follows:

\- Adds the (previously stored) calculated residence time to the correction field.

\- Upon reception (or local creation) of the PTP Pdelay_Resp_Follow_Up message associated with the previously received PTP Pdelay_Resp message, the ingress TT for the PTP Pdelay_Resp_Follow_Up message modifies the payload of the PTP Pdelay_Resp_Follow_Up message that it sends towards the egress TT for the PTP Pdelay_Resp_Follow_Up message as follows:

\- Adds the (previously stored) calculated residence time of the associated PTP Pdelay_Req message to the correction field.

\- Upon reception of the PTP Pdelay_Resp_Follow_Up message associated with the PTP Pdelay_Resp, the egress TT for PTP Pdelay_Resp_Follow_Up message modifies the payload of the PTP Pdelay_Resp_Follow_Up message that it sends towards the downstream PTP instance as follows:

\- Adds the (previously stored) calculated residence time of the associated PTP Pdelay_Resp messages to the correction field.
