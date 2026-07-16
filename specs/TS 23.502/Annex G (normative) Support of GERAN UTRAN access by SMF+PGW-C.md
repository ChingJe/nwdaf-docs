---
spec: TS 23.502
version: 18.14.0
release: '18'
clause: Annex G
title: 'Annex G (normative): Support of GERAN/UTRAN access by SMF+PGW-C'
source_archive: 23502-ie0.zip
source_document: 23502-ie0.docx
source_archive_sha256: 0f24045c49ddf710fb91e51ec74d070cfd13d1e9bafc07809d7e87eae6a61ce8
source_document_sha256: 89536df6095fba3b9c90f1fa06ceaec918dfc6b0fd26a769a06eae7db66bb603
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex G (normative): Support of GERAN/UTRAN access by SMF+PGW-C

This annex applies when the SMF+PGW-C is enhanced to support GERAN/UTRAN access via Gn/Gp interface as specified in Annex L of TS 23.501 \[2\].

NOTE 1: For the interface with the serving node of the UE, the SMF+PGW-C is assumed to behave as the Control Plane of the PGW described in Annex D of TS 23.401 \[13\].

SMF+PGW-C is selected by the SGSN using existing mechanism as specified in Annex A of TS 23.060 \[79\].

NOTE 2 If network deployment requires both SMF+PGW-C and legacy PGW, selection of SMF+PGW-C by SGSN can be achieved based on e.g. APN and optionally APN-OI Replacement as specified Annex A of TS 23.060 \[79\].

When a SMF+PGW-C is used for GERAN/UTRAN access, at PDP context activation, the SMF+PGW-C allocates a PDU Session ID (in the network range) and uses this PDU Session ID over SBI interface (e.g. N7).

The following procedures from TS 23.060 \[79\] are not supported when SMF+PGW-C is used for GERAN/UTRAN access:

\- Network requested PDP Context Activation Procedure (clause 9.2.2.2 of TS 23.060 \[79\]).

\- Secondary PDP Context Activation Procedure (clause 9.2.2.1.1 of TS 23.060 \[79\]).

\- Network Requested Secondary PDP Context Activation Procedure using Gn (clause 9.2.2.1.3 of TS 23.060 \[79\]).

When SMF+PGW-C is used for GERAN/UTRAN access and interacts with PCF, the SMF+PGW-C uses SM policy association procedures as specified in clause 4.11.0a.2 with the following modification:

\- The SMF+PGW-C performs mapping of QoS parameters as follows:

\- The SMF+PGW-C maps the Release 99 QoS parameters received from Gn/Gp interface to EPS QoS parameters as specified in Annex E of TS 23.401 \[13\], which is then used to derive QoS parameters over N7 interface as specified in clause 4.11.0a.2.

\- the SMF+PGW-C uses QoS parameters over N7 interface to derive EPS QoS parameters as specified in clause 4.11.0a.2, which is then mapped to Release 99 QoS parameters for Gn/Gp interface as specified in Annex E of TS 23.401 \[13\].

\- For SM Policy Association Establishment Procedure, the SMF+PGW-C invokes Npcf_SMPolicyControl_Create Service operation taking input from the information elements received in Create PDP Context Request message (specified in TS 23.060 \[79\]), including mapping of QoS parameters as mentioned above as well as GERAN/UTRAN location management related information.

\- For SM Policy Association Modification procedure initiated by the SMF+PGW-C, the SMF+PGW-C invokes Npcf_SMPolicyControl_Update Service operation taking input from the information elements received in Update PDP Context Request message (specified in TS 23.060 \[79\]), including mapping of QoS parameters as mentioned above, as well as GERAN/UTRAN location management related information.

\- For SM Policy Association Modification procedure initiated by the PCF, the SMF+PGW-C may receive PCC Rules and PDU Session Policy Information. The SMF+PGW-C performs mapping of QoS parameters as mentioned above.

\- For SM Policy Association Termination procedure, the SMF+PGW-C invokes Npcf_SMPolicyControl_Delete service operation (including GERAN/UTRAN location management related information) when receiving Delete PDP Context Request message (specified in TS 23.060 \[79\]).

\- Even though N7 supports Ethernet PDU Session Type, as Ethernet PDN Type is not supported in GERAN/UTRAN, it is assumed that if the UE moves from E-UTRAN to GERAN/UTRAN, Ethernet PDN connections are released and thus no information related with Ethernet PDU Session Type shall be exchanged over N7 when a UE is served by GERAN/UTRAN.

\- Even though GERAN/UTRAN specifications foresee other alternatives, the Bearer Binding is performed by the SMF+PGW-C acting as a PGW.

\- Access Network Information reporting with a granularity of GERAN/UTRAN cell is supported over N7 and N5.

When the UE moves between E-UTRAN and GERAN/UTRAN, the SMF+PGW-C may invoke SM Policy Association Modification procedure based on the Policy Control Request Triggers as specified in TS 23.503 \[20\].

NOTE 3: Support for IP address preservation upon indirect mobility between 5GS and GERAN/UTRAN for PDN sessions established in EPC is described in clause 5.17.2.4 of TS 23.501 \[2\]. IP address preservation is not supported for direct mobility between 5GS and GERAN/UTRAN, nor for indirect mobility cases when the PDN session is established in 5GS or in GERAN/UTRAN.

NOTE 4: Usage of SMF+PGW-C to serve a PDP context requires no change to SGSN(s) (and thus to roaming partners in Home Routed roaming) as it is assumed that DNS records are properly configured to map APN to SMF+PGW-C acting as PGW.
