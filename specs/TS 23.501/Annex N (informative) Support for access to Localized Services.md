---
spec: TS 23.501
version: 18.12.0
release: '18'
clause: Annex N
title: 'Annex N (informative): Support for access to Localized Services'
source_archive: 23501-ic0.zip
source_document: 23501-ic0.docx
source_archive_sha256: 800fb58a2a3a8c3eeac7084497de566fc730808923d8eea61b84d2555360debe
source_document_sha256: cdab3840e4d278d381f70806e6458f87dff8e5fd238f370e4f65872bbd5088a6
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex N (informative): Support for access to Localized Services

## N.1 General

A Localized Service is a service, which is provided at specific/limited area and/or can be bounded in time. The service can be realized via applications (e.g. live or on-demand audio/video stream, electric game, IMS, etc.), or connectivity (e.g. UE to UE, UE to Data Network, etc.).

A Localized Service provider is an application provider or network operator who make their services localized and that are offered to the end user via a network. A network providing Localized Services can be an SNPN or a PNI-NPN.

## N.2 Enabling access to Localized Services

### N.2.1 General

To enable a PNI-NPN or SNPN to provide access to Localized Services, the PNI-NPN or SNPN operator configures the network with information enabling the UEs to access the Localized Services using the PNI-NPN or SNPN according to any validity of the Localized Services and the information is determined in agreement with the Localized Service Provider e.g.:

a\. Identification of each Localized Service, e.g. to be used in URSP rules.

b\. validity restriction for each Localized Service, e.g. the validity of time and/or location.

c\. service parameters for each Localized Service, e.g. DNN, S-NSSAI and QoS requirements.

d\. service authorization methods, e.g. NSSAA or Secondary authentication/authorization during PDU Session establishment.

To allow the UE to access the PNI-NPN providing access to Localized Services using credentials of the HPLMN, the PNI-NPN can be configured based on the Localized Service agreements between the PNI-NPN and the HPLMN, to allow primary authentication towards the HPLMN.

To allow the UE to access the SNPN providing access to Localized Services using credentials of the Credentials Holder, the SNPN can be configured based on Localized Service agreements between the SNPN and the Credentials Holder, to allow primary authentication towards the Credentials Holder.

To allow the UE to access the SNPN providing access to Localized Services when new credential is required, the SNPN can provide UE onboarding function as specified in clause 5.30.2.10 for the UE to obtain credential and necessary information to access the SNPN, or the UE can leverage existing credential and network connection to get access to a PVS via User Plane to obtain new credential.

To allow the UE to access the PNI-NPN providing access to the Localized Services where NSSAA or secondary authentication/authorization during PDU session establishment is required, the UE can obtain new credential using remote provisioning functionality as defined in clause 5.39.

To allow the UE to access the HPLMN or subscribed SNPN services while being registered in the PNI-NPN or SNPN, the PNI-NPN or SNPN can establish service agreements and configure inter-connect with the HPLMN or subscribed SNPN operator. If a PNI-NPN is providing access to the Localized Services, the existing roaming architecture with home-routed PDU Sessions are used. If an SNPN is providing access to the Localized Services, then the UE can access HPLMN or subscribed SNPN as described in Annex D, clauses D.3, D.6 and D.7.

### N.2.2 Configuration of network to provide access to Localized Services

For configuring the PNI-NPN or SNPN (e.g. creation of network slice/DNN for carrying Localized Service traffic), existing OAM mechanisms can be re-used as per TS 28.557 \[148\] clause 6.3.1, that provides a solution for NPN provisioning by a network slice of a PLMN and for exposure of management capability of PNI-NPN (clause 6.3.2). The attributes to support this management is further documented in TS 28.541 \[149\].

### N.2.3 Session Management aspects

For session management level information and interactions such as monitoring the PNI-NPN or SNPN performance and enabling suitable QoS for UE in the PNI-NPN or SNPN for Localized Service, the following non-exhaustive options can be used:

\- Covered by the SLA between the PNI-NPN or SNPN operator and the Localized Service Provider.

\- Reuse the existing network exposure procedures as specified in TS 23.502 \[3\] clause 4.15, where the Localized Service Provider is taking the AF role and utilizing the exposure capability provided by the PNI-NPN or SNPN.

\- Enable NEF/PCF in the PNI-NPN or SNPN providing access to the Localized Services (via AF of the Localized Service Provider) to receive and forward the validity conditions and QoS requirements of the Localized Services to the AMF/SMF by reusing the existing PCF initiated AM/SM policy association procedures described in TS 23.502 \[3\] clause 4.16.

## N.3 Selection of network providing access to Localized Services

The UE selects an SNPN providing access for Localized Services as described in clause 5.30.2.4.2, clause 5.30.2.4.3 and in TS 23.122 \[17\].

## N.4 Enabling the UE access to Localized Services

The access to a Localized Service is made available in a specific area and/or a specific period of time.

After the UE has successfully registered to a PNI-NPN/SNPN providing access to the Localized Service, the UE can be configured with URSP rules using existing principles (see clause 6.6.2.2 of TS 23.503 \[45\]).

The URSP rules can include an association between the UE application and the DNN/S-NSSAI which is meant for a particular Localized Service. The URSP rules can also include "Route Selection Validation Criteria" as described in Table 6.6.2.1-3 of TS 23.503 \[45\], with the time/location defined for the particular Localized Service.

The existing LADN feature described in clause 5.6.5 can also be used for enabling the UE access to Localized Service which is defined by a LADN DNN. The S-NSSAI used for a Localized Service can be restricted to a specific area and time as described in clause 5.15.

## N.5 Support for leaving network that provides access to Localized Services

When Localized Services in a network are completed, all UEs that are registered with the network are expected to be transferred to other network or to other network resources (e.g. other cells) within the same network, potentially within a relatively short timeframe. The other network can be HPLMN, VPLMN or another SNPN.

UE can stop using the network resources for Localized Services for numerous reasons, e.g. when one or more of the following conditions apply:

\- Localized Services in a network are completed.

\- Validity conditions of network selection information are no longer met.

\- The user decides to stop using the Localized Services before they are completed.

\- A policy decision is taken by the network, with the effect that the UE is deregistered before the Localized Services are completed.

NOTE: The list is not an exhaustive list and UE can stop using the network resources for Localized Services due to other reasons e.g. UE loses coverage, power off.

When large number of UEs move to other network (i.e. HPLMN, VPLMN or another SNPN) or other network resources within a relatively short timeframe, the total signalling involved can cause signalling overload in the target network.

Existing mechanisms for Control Plane Load Control, Congestion and Overload Control described in clause 5.19 and access control and barring described in clause 5.2.5 can be used to mitigate the signalling overload caused by returning UEs. For further enhancement of mitigation of signalling overload, additional mechanisms can be implemented to ensure spreading of the load that returning UEs cause. Such mechanisms are implementation-specific, but some guidelines that can be considered are described below:

\- The time validity of the network selection information given to a UE can be set somewhat longer than the actual duration of the service, e.g. users will by themselves disable Localized Service and the UE then stops using the connectivity to access the Localized Service, thus causing the UE to be moved, e.g. by performing normal network selection.

\- The time validity of the network selection information given to a UE can be different for each UE so that each UE performs network selection at a different time to distribute returning UEs.

\- When the AMF after end of Localized Services triggers deregistration of UEs, the deregistration requests can be sent at a certain rate in an adaptive and distributed manner, with the effect that the signalling load on both the source network and the target network is limited.

\- When the AMF after end of Localized Services triggers UE configuration update procedure, e.g. to remove S-NSSAI from the Allowed NSSAI (if dedicated S-NSSAI is used for the Localized Service), the requests can be sent at a certain rate, with the effect that the signalling load in the network is limited.

When the NAS level congestion control is activated at AMF as specified in clause 5.19.7.2, to prevent a UE staying in an SNPN for accessing for Localized Services but not able to get services from the SNPN due to the congestion, additional mechanism can be implemented. Such mechanisms are implementation-specific but some guidelines that can be considered are described below:

\- the AMF can determine whether to reject the UE with a proper cause without Mobility Management back-off timer to allow the UE to reselect another SNPN for Localized Services.

## N.6 Configuration of Credentials Holder for determining SNPN selection information

To enable the HPLMN or the subscribed SNPN acting as Credentials Holder to generate and provision UEs with SNPN selection information for discovery and selection of SNPNs providing Localized Services, based on Localized Service agreements between the Localized Service Provider or the SNPN providing Localized Services and the HPLMN or the subscribed SNPN acting as Credentials Holder, the Localized Service Provider or the SNPN providing access to Localized Services can provide configuration information for SNPN selection to the HPLMN or the subscribed SNPN acting as Credentials Holder. The configuration information for SNPN selection may contain at least one of the following parameters:

a\. SNPN ID or GIN of the SNPN providing access to one or more Localized Services;

b\. Identification of each Localized Service;

c\. validity information (e.g. time validity information and optionally location validity information) for each Localized Service and/or location assistance information; and/or

d\. List of UE IDs (e.g. GPSIs or External Group ID) identifying the UEs subscribed with a Localized Service.

NOTE: How HPLMN or subscribed SNPN obtains the information above as part of the Localized Service agreements is out of 3GPP scope.

The operator of the HPLMN or the subscribed SNPN acting as Credentials Holder then may use the information received from the SNPN providing Localized Services and/or Localized Service Provider to create or update the Credentials Holder controlled prioritized lists of preferred SNPNs/GINs for accessing Localized Services and provision the UEs using the Steering of Roaming procedure as defined in TS 23.122 \[17\].
