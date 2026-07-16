---
spec: TS 23.501
version: 18.12.0
release: '18'
clause: K.2
title: K.2 Port and user plane node management information exchange for time synchronization
source_archive: 23501-ic0.zip
source_document: 23501-ic0.docx
source_archive_sha256: 800fb58a2a3a8c3eeac7084497de566fc730808923d8eea61b84d2555360debe
source_document_sha256: cdab3840e4d278d381f70806e6458f87dff8e5fd238f370e4f65872bbd5088a6
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# K.2 Port and user plane node management information exchange for time synchronization

## K.2.1 Capability exchange

DS-TT and NW-TT indicate time synchronization information they support inside the Port management capabilities (see Table K.1-1).

TSN AF and TSCTSF may determine the PTP functionalities supported by DS-TT and NW-TT by retrieving the following port management information or user plane node management information, respectively:

\- Supported PTP instance types;

\- Supported transport types;

\- Supported PTP delay mechanisms;

\- Grandmaster capability;

\- Supported PTP profiles;

\- Number of supported PTP instances.

NOTE: If NW-TT or DS-TT do not indicate support for any of the PTP profiles and PTP instance types, then TSN AF or TSCTSF assume that the NW-TT or DS-TT only support acting as a PTP Relay instance with the gPTP GM connected on N6.

If DS-TT and NW-TT support the PTP Relay instance type as defined by IEEE 802.1AS \[104\] then DS-TT and NW-TT shall include the IEEE  802.1AS \[104\] PTP profile in the "Supported PTP profiles" in PMIC and UMIC, respectively.

The TSN AF or TSCTSF may retrieve the "Number of supported PTP instances" from NW-TT via UMIC and from DS-TT via PMIC.

## K.2.2 PTP Instance configuration

### K.2.2.1 General

Based on input received from external applications (CNC in case of TSN AF or any AF in case of TSCTSF), TSN AF or TSCTSF may configure PTP instances (identified by PTP Instance ID) in a DS-TT or NW-TT by sending port management information (PMIC, see Table K.1-1) and user plane node management information (UMIC, see Table K.1-2) to DS-TT or NW-TT as described below:

\- use PMIC "PTP instance specification" for configuring DS-TT(s) for PTP instance data sets common for all PTP ports (i.e. defaultDS and TimePropertiesDS) and PTP instance data sets specific for each PTP port (i.e. portDS data set);

\- use UMIC "PTP instance specification" for configuring NW-TT for PTP instance data sets common for all PTP ports;

\- use PMIC "PTP instance specification" for configuring NW-TT for PTP instance data sets specific for each PTP port;

\- use UMIC "Time synchronization information for DS-TT ports" for configuring NW-TT for PTP instance data sets specific for each PTP port for the PTP ports in DS-TT(s).

TSN AF or TSCTSF may also configure PTP instances for DS-TT ports in NW-TT by sending UMIC (see Table K.1-2) to NW-TT to enable NW-TT to operate as a grandmaster on behalf of DS-TT (see clause K.2.2.4 for more details).

For each PTP instance the TSN AF or TSCTSF may provide individual PTP configuration parameters or may provide a PTP profile ID to DS-TT or NW-TT. The DS-TT and NW-TT use the default values as defined in the corresponding PTP Profile, if individual PTP configuration parameters that are covered by the PTP profile are not provided.

NOTE 1: Even if PTP profiles are used to configure DS-TT or NW-TT, individual PTP parameters can still be configured in addition, e.g. domain numbers, transport to use, etc.

To configure DS-TT and NW-TT to operate as a PTP relay instance, TSN AF or TSCTSF shall set the PTP profile (see Table K.1-1) to IEEE Std 802.1AS \[104\].

DS-TT may operate as a PTP relay instance with the gPTP GM connected on N6 until the first PTP instance is configured in the DS-TT by TSN AF or TSCTSF.

To initialize a PTP instance in 5GS, TSN AF or TSCTSF creates a new PTP instance in NW-TT by assigning a new PTP Instance ID and indicating it to the NW-TT in "PTP instance specification" in UMIC and PMIC(s) for each NW-TT port that is part of the PTP instance. TSN AF or TSCTSF then retrieves the "defaultDS.clockIdentity" of the PTP instance in NW-TT via UMIC. NW-TT ensures that the clockIdentity in defaultDS in UMIC matches with the clockIdentity in the portDS.portIdentity in PMIC(s) for a particular PTP Instance ID.

To add a DS-TT port into an existent PTP instance in 5GS, the TSN AF or TSCTSF indicates the PTP Instance ID (to which the DS-TT port is being added) to the DS-TT in "PTP instance specification" in PMIC and indicating the PTP Instance ID to the NW-TT in "Time synchronization information for DS-TT ports" in UMIC for the corresponding DS-TT port.

For a particular PTP instance in NW-TT, the same PTP Instance ID shall be used in "PTP instance specification" in PMIC, in "PTP instance specification" in UMIC and in "Time synchronization information for DS-TT ports" in UMIC.

NOTE 2: The TSN AF or TSCTSF creates a PTP Instance in the NW-TT or DS-TT by using the "Set parameter" operation code as described in TS 24.539 \[139\]. The NW-TT or DS-TT determines that this "Set parameter" operation creates a new PTP Instance based on the PTP Instance ID that does not correspond to any of the configured PTP Instances in the "PTP instance specification" and "Time synchronization information for DS-TT ports" (for NW-TT) or in the "PTP instance specification" (for DS-TT).

The TSN AF or TSCTSF then initializes the PTP instance in the DS-TT by setting the applicable PTP instance data sets common for all PTP ports (i.e. defaultDS and TimePropertiesDS), (including"defaultDS.clockIdentity") via "PTP instance specification" in PMIC to the same value as retrieved from the NW-TT via "PTP instance specification" in UMIC. The TSN AF or TSCTSF also enables the PTP instance by setting the defaultDS.instanceEnable = TRUE to DS-TT via PMIC and to NW-TT via UMIC (if applicable). The TSN AF or TSCTSF can initialize any number of PTP instances:

a\) among the DS-TT(s) and NW-TT that are part of the same set of PTP instances in 5GS; up to the maximum number of supported PTP instances by the NW-TT or DS-TT that supports the lowest number of supported PTP instances; and

b\) in the NW-TT; up to the maximum number of supported PTP instances by the NW-TT.

NOTE 3: How the TSN AF or TSCTSF assign NW-TT port(s) of one NW-TT to different PTP instances is up to implementation.

To remove a DS-TT port from a PTP instance in 5GS, the TSN AF or TSCTSF deletes the PTP instance in DS-TT using PMIC and in NW-TT using UMIC as specified in TS 24.539 \[139\]. To remove a NW-TT port from a PTP instance in 5GS, the TSN AF or TSCTSF deletes the PTP instance in NW-TT using PMIC as specified in TS 24.539 \[139\]. If a PTP instance in 5GS is no more needed the TSN AF or TSCTSF may delete the PTP instance in NW-TT using UMIC as specified in TS 24.539 \[139\].

### K.2.2.2 Configuration for Sync and Announce reception timeouts

The NW-TT shall be able to determine the timeout of the reception of (g)PTP Announce (when the 5GS operates as a time-aware system or Boundary Clock) and gPTP Sync messages (when the 5GS operates as time-aware system). To enable this, the TSCTSF or TSN AF shall configure the NW-TT for the following information via PMIC for each PTP port in NW-TT and "Time synchronization information for each DS-TT port" element in UMIC for each PTP port in DS-TT:

portDS.announceReceiptTimeout (for time-aware system and Boundary Clock);

portDS.syncReceiptTimeout (for time-aware system);

portDS.logAnnounceInterval (for Boundary Clock).

portDS.initialLogAnnounceInterval, portDS.useMgtSettableLogAnnounceInterval and portDS.mgtSettableLogAnnounceInterval (for time-aware system).

### K.2.2.3 Configuration for PTP port states

The PTP port states may be determined by NW-TT either via:

\- Method a), BMCA procedure.

\- Method b), local configuration.

When Method b) is used, the TSN AF or TSCTSF sets the defaultDS.externalPortConfigurationEnabled (per PTP instance) in UMIC to TRUE and sets the value of externalPortConfigurationPortDS.desiredState (per PTP port) in UMIC for each DS-TT port and in PMIC for each NW-TT port for the (g)PTP domain.

### K.2.2.4 Configuration for PTP grandmaster function

The following options may be supported (per DS-TT) for the 5GS to generate the Sync, Follow_Up and Announce messages for the Leader ports on the DS-TT:

a\) NW-TT generates the Sync, Follow_Up and Announce messages on behalf of DS-TT (e.g. if DS-TT does not support this).

b\) DS-TT generates the Sync, Follow_Up and Announce messages in this DS-TT.

TSN AF and TSCTSF may use the elements in port and user plane node management information container to determine the PTP grandmaster functionality supported by DS-TT and NW-TT and may configure the DS-TT and NW-TT ports to operate as in option a) or b) as follows:

\- The "PTP grandmaster capable" element and the "gPTP grandmaster capable" element in PMIC are used to indicate the support for PTP or gPTP grandmaster capability, respectively, in each DS-TT. If the TSN AF or TSCTSF determines the DS-TT supports grandmaster capability (PTP or gPTP grandmaster capable is TRUE), then either option a) or b) can be used for the PTP instance(s) in the DS-TT. Otherwise, only option a) can be used for the PTP instance(s) in the DS-TT.

\- To enable option a) for PTP ports in DS-TT, the TSN AF or TSCTSF sets the element "Grandmaster on behalf of DS-TT enabled" TRUE (per PTP instance per DS-TT) in UMIC for the respective DS-TT port and the TSN AF or TSCTSF sets the element "Grandmaster enabled" FALSE (per PTP instance per DS-TT) in PMIC to the respective DS-TT port.

\- To enable option b) for PTP ports in DS-TT, the TSN AF or TSCTSF sets the element "Grandmaster on behalf of DS-TT enabled" FALSE in UMIC (per PTP instance per DS-TT) for the respective port and the TSN AF or TSCTSF sets the element "Grandmaster enabled" TRUE (per PTP instance per DS-TT) in PMIC to the respective DS-TT port.

\- To enable either option a) or option b) for a PTP instance, the TSN AF or TSCTSF sets the element "Grandmaster candidate enabled" TRUE (per PTP instance) in UMIC.

\- When option b) is used for one or more PTP ports in DS-TT(s), the TSN AF or TSCTSF shall use the elements in defaultDS in PMIC for the respective DS-TT(s) and in UMIC for NW-TT to ensure that all PTP ports in the DS-TT(s) and NW-TT in particular PTP instance are distributing the same values of grandmasterPriority1, grandmasterClockQuality, grandmasterPriority2, grandmasterIdentity and timeSource message fields in Announce messages.

### K.2.2.5 Configuration for Sync and Announce intervals

The TSN AF or TSCTSF uses the values in portDS.logSyncInterval (for Boundary Clock) or portDS.initialLogSyncInterval, portDS.useMgtSettableLogSyncInterval and portDS.mgtSettableLogSyncInterval (for time-aware system) to configure the interval for the Sync messages (per PTP port) as described in IEEE Std 1588 \[126\] or IEEE Std 802.1AS \[104\], respectively. The TSCTSF or TSN AF configures those values as follows:

\- TSCTSF or TSN AF use PMIC to configure the values for the PTP ports in NW-TT.

\- TSCTSF or TSN AF use the "Time synchronization information for each DS-TT port" element in UMIC to configure the values for PTP ports in DS-TT(s) if NW-TT acts as GM on behalf of those DS-TTs.

\- TSCTSF or TSN AF use PMIC to configure the values for the PTP ports in DS-TT if the DS-TT is capable of acting as a GM.

When the NW-TT generates the (g)PTP Sync messages on behalf of the DS-TT, the NW-TT uses the values in the element "Time synchronization information for each DS-TT port" in UMIC to determine the Sync interval for the PTP ports the respective DS-TT. When DS-TT generates the (g)PTP Sync messages, the DS-TT uses the values in PMIC to determine the Sync interval for the PTP ports in this DS-TT.

The TSN AF or TSCTSF uses the values in portDS.logAnnounceInterval (for Boundary Clock) or portDS.initialLogAnnounceInterval, portDS.useMgtSettableLogAnnounceInterval and portDS.mgtSettableLogAnnounceInterval (for time-aware system) to configure the interval for the Announce messages (per PTP port) as described in IEEE Std 1588 \[126\] and IEEE Std 802.1AS \[104\], respectively. The TSCTSF or TSN AF configures those values as follows:

\- TSCTSF or TSN AF use PMIC to configure the values for the PTP ports in NW-TT.

\- TSCTSF or TSN AF use the "Time synchronization information for each DS-TT port" element in UMIC to configure the values for PTP ports in DS-TT(s) if NW-TT acts as GM on behalf of those DS-TTs.

\- TSCTSF or TSN AF use PMIC to configure the values for the PTP ports in DS-TT if the DS-TT is capable of acting as a GM.

When the NW-TT generates the (g)PTP Announce messages on behalf of the DS-TT, the NW-TT uses the values in the element "Time synchronization information for each DS-TT port" in UMIC to determine the Announce interval for the PTP ports the respective DS-TT. When DS-TT generates the (g)PTP Announce messages, the DS-TT uses the values in PMIC to determine the Announce interval for the PTP ports in this DS-TT.

### K.2.2.6 Configuration for transport protocols

The procedure described in this clause is applicable when the PTP Profile that is used for the PTP instance in 5GS defines multiple permitted transport protocols.

TSN AF or TSCTSF may use the element "Supported transport types" in port management information container (per DS-TT) to determine the supported transport types in the DS-TT. TSN AF or TSCTSF may use the element "Supported transport types" in UMIC (per NW-TT) to determine the supported transport types in the NW-TT.

The TSN AF or TSCTSF may use the element "Transport type" (per PTP instance) in PMIC to configure the transport protocol in use for the PTP instance in DS-TT. The TSN AF or TSCTSF may use the element "Transport type" (per PTP instance) in UMIC to configure the transport protocol in use for the PTP instance in NW-TT.

The PTP instance shall be configured to use one of the following transport protocols:

1\) Ethernet as described in Annex E of IEEE Std 1588 \[126\]. The Ethertype as defined for PTP shall be used. The related Ethernet frames carry the PTP multicast Ethernet destination MAC address.

2\) UDP over IPv4 as described in Annex C of IEEE Std 1588 \[126\],

3\) UDP over IPv6 as described in Annex D of IEEE Std 1588 \[126\].

Option 1 applies to Ethernet PDU Session type. Options 2 and 3 apply to IP PDU Session type or Ethernet PDU Session type with IP payload.
