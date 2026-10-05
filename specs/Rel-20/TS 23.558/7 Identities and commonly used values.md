---
spec: TS 23.558
version: 20.3.0
release: '20'
clause: 7
title: 7 Identities and commonly used values
source_archive: 23558-k30.zip
source_document: 23558-k30.docx
content_origin: 3gpp-source
---

# 7 Identities and commonly used values


## 7.1 General

The following clauses list identities and commonly used values that are used in this technical specification.

## 7.2 Identities


### 7.2.1 General

The following clauses specify a collection of identities that are associated with entities defined and being used in this specification.

### 7.2.2 Edge Enabler Client ID (EECID)

The EECID is a globally unique value that identifies an EEC.

NOTE: Security and privacy aspects related to EECID are specified in 3GPP TS 33.558 \[23\].

### 7.2.3 Edge Enabler Server ID (EESID)

The EESID identifies an EES and each EES connected with the PLMN has a unique EESID within PLMN domain.

### 7.2.4 Edge Application Server ID (EASID)

The EASID is a globally unique identifier which identifies a particular application for e.g. SA6Video, SA6Game etc. All EAS instances (e.g. of SA6Video application) will share the same EASID.

NOTE: The definition of the EASID is in 3GPP TS 24.558 \[29\] and 3GPP TS 29.558 \[30\].

### 7.2.5 Application Client ID (ACID)

The ACID identifies the client side of a particular application, for e.g. SA6Video viewer, SA6MsgClient etc. For example, all SA6MsgClient clients will share the same ACID.

In case that the UE is running mobile OS, the ACID is a pair of OSId and OSAppId.

### 7.2.6 UE ID

The UE Identifier (UE ID) uniquely identifies a particular UE within a PLMN domain. UE ID can be:

a\) a GPSI, as defined in 3GPP TS 23.501 \[2\].

NOTE 1: For user's privacy reasons, GPSI in the form of MSISDN can be used only after obtaining user's consent.

NOTE 2: To protect user's privacy, if MSISDN cannot be used then AF-specific UE ID which is a GPSI in the form of an External ID may either be acquired through the NEF's Nnef_UEId_Get service operation (see TS 23.502 clause 4.15.10) or other out of scope means (e.g. pre-configuration).

b\) an EEL-generated Edge UE ID, as defined in clause 7.2.9.

### 7.2.7 UE Group ID

The UE Group ID uniquely identifies a group of UE within a PLMN domain. Following identities are examples that can be used:

a\) internal group ID, as defined in 3GPP TS 23.501 \[2\]; and

b\) external group ID, as defined in 3GPP TS 23.501 \[2\].

### 7.2.8 EEC Context ID

The EEC Context ID is a globally unique value which identifies a set of parameters associated with the EEC (e.g., due to registration) and maintained in the Edge Enabler Layer by EESs.

If the EEC registration request does not include a previously assigned EEC Context ID value, the receiver EES assigns a new EEC Context ID and creates an EEC Context as described in the Table 8.2.8-1.

Providing a previously assigned EEC Context ID at registration allows maintaining the EEC Context in the Edge Enabler Layer beyond the lifetime of a registration, subject to policies. If the EEC registration request does include a previously assigned EEC Context ID value, after EEC Context relocation, the receiver EES may assign a new EEC Context ID, subject to implementation and local policies.

NOTE: The EEC Context ID may be implemented as combination of other IDs (e.g., EES ID and registration ID). How the EEC Context ID is specified or assigned is out of scope of this specification.

### 7.2.9 Edge UE ID

The Edge UE ID is an identifier that is associated with the UE ID (GPSI as per clause 7.2.6) and managed by EES. The EES can generate Edge UE ID as needed (e.g. as per ECSP policy). Edge UE ID can be shared with the EAS directly (using UE Identifier API as per clause 8.6.5) and/or indirectly via AC (using UE ID request as per clause 8.14.2.6) in order for it to be used by the EAS over EDGE-3 interactions when it is not desired to share the UE ID (GPSI as per clause 7.2.6) with the EAS. The Edge UE ID can be temporary to limit the access of the EAS when needed.

NOTE: The Edge UE ID is not applicable for EDGE-7 interactions.

### 7.2.10 EAS bundle information

The EAS bundle information includes EAS bundle type, a list of EASIDs or a EAS bundle ID. The EAS bundle information may also include main EASID and EAS bundle requirements. EAS bundle ID establishes an association between the EASs. When included in the EAS profile, EAS bundle ID denotes the bundle to which the EAS belongs. When included in the AC profile EAS bundle ID is used to perform different Edge Enabler Layer operations, such as EAS discovery. Edge Enabler Layer handles the EASs belonging to the same bundle as required by related EAS bundle requirements as described in clause 8.2.10.

NOTE 1: Both EAS bundle ID and EAS bundle requirements are provided by the ASP.

NOTE 2: Bundle ID is necessary when the affinity between bundled EASs is strong (e.g., co-deployment and co-migration is essential) and the related ASPs, which provide the AC and bundled EASs, established the bundle. List of EASIDs is required when the affinity between the bundled EASs is weak (e.g., co-deployment and co-migration is only "nice to have").

NOTE 3: Following types of EAS bundles are considered in this release:

\- Direct bundle, where AC interacts with multiple EASs of the EAS bundle directly with no coordination between the EASs; and

\- Proxy bundle, where the AC interacts with one EAS of the EAS bundle which in turn coordinates with other EASs of the EAS bundle to provide services to the AC by exchanging Application Data Traffic with the other EASs, which is out-of-scope of this specification.

NOTE 4: Discovery of EAS Service APIs via CAPIF for the proxy bundle type is not considered in this release.

### 7.2.11 Application Group ID

Application Group ID uniquely identifies a group of UEs using the same application. It is allocated (either dynamically or pre-configured in the AC) by the ASP and is unique within the application. ACs supporting the same application on different OS (e.g., iOS, Android), and therefore differing ACIDs (as specified in clause 7.2.5), can have the same Application Group ID.

NOTE 1: In this version of specification, Application Group ID is assumed to be unique per EAS ID. Ensuring the uniqueness of the Application Group ID per EAS ID is out of scope of this specification.

NOTE 2: In this version of specification, an Application Group is assumed to correspond to a single application.

## 7.3 Commonly used values


### 7.3.1 General

### 7.3.2 UE location

The UE location identifies where the UE is connected to the network or the position of the UE. It provides consistent definition of the UE's location across the UE and network entities. Following values are examples of UE locations that can be used:

a\) Cell Identity, Tracking Area Identity, GPS Coordinates or civic addresses as defined in 3GPP TS 23.502 \[3\] clause 4.15.3.

### 7.3.3 Service areas


#### 7.3.3.1 General

ECSPs and ASPs may allow access to Edge Computing service from specific areas i.e. allowing only the UEs within that area to access functional entities resident in the EDN. This area is called service area.

Some functional elements make decisions based on the topological location of the UE, (e.g. the cell it is connected to) while others make decisions based on the UE's geographical location (e.g. its geographical coordinates).

Functional elements that are aware of both topological and geographical information can translate one value to the other.

#### 7.3.3.2 Topological Service Area

A Topological Service Area is defined in relationship with a UE's point of connection to the network, such as: a collection of Cell IDs, Tracking Area Identities or the PLMN ID. Any UE that is attached to the Core Network from a cell whose ID is in this list, can be served by the functional entity in the EDN that is configured to serve that Topological Service Area.

NOTE: Topological Service Area information is not applicable for untrusted functional elements (EESs and/or EASs deployed outside the MNO trust domain).

#### 7.3.3.3 Geographical Service Area

A Geographical Service Area is an area that is specified by geographical units as defined in 3GPP TS 23.032 \[21\], such as: Geographical coordinates, an area that is defined as a circle whose centre is denoted by geographical coordinates, an area that is defined by a polygon whose corners are denoted by geographical coordinates. A Geographical Service Area can also be expressed in other ways such as: a well-known buildings, parks, arenas, civic addresses or ZIP code etc.

Applications can be configured to serve UEs that are in a specified geographical area and deny service from UEs that are not located in that area.

NOTE: Whether and how geographical information is used by applications to provide or deny service is out of scope.

#### 7.3.3.4 EDN service area

A service area from which the access to the EDN is allowed. ECSPs can use LADNs, as described in Annex A.2.4 of this document, to deploy EDNs with access restricted from specific areas. When an EDN is deployed using LADN, the EDN service area is same as the LADN service area and rules specified for LADN apply to the UE, as specified in 3GPP TS 23.501 \[2\].

In a deployment using DNs other than LADNs, the EDN service area is the whole PLMN for non-roaming scenario.

NOTE 1: The EDN service area for roaming scenario is out of scope in this release of the specification.

NOTE 2: For the purpose of restricting the access to the EES from specific areas, ECSP can use the EES service area, which is specified in clause 7.3.3.5, even if the EDN service area is the whole PLMN.

The EDN service area may be expressed as a Topological Service Area.

#### 7.3.3.5 EES Service Area

A service area from which the access to the EES is allowed. This service area is equal to or a subset of the service area of the EDN in which the EES resides.

The EES service area may be expressed as a Topological Service Area, a Geographical Service Area or both.

#### 7.3.3.6 EAS service area

A service area from which the access to the EAS is allowed. This service area is equal to or a subset of the service area of the EES which serves the EAS.

The EAS service area may be expressed as a Topological Service Area, a Geographical Service Area or both.
