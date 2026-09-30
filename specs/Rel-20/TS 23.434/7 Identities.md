---
spec: TS 23.434
version: 20.1.0
release: '20'
clause: 7
title: 7 Identities
source_archive: 23434-k10.zip
source_document: 23434-k10.docx
content_origin: 3gpp-source
---

# 7 Identities


## 7.1 User identity (User ID)

The VAL user presents the user identity to the identity management server during a user authentication transaction, to provide the identity management client a means for VAL service authentication. In general, since identity management is a common SEAL service, it uses a set of credentials (e.g. biometrics, secureID, username/password) that may not necessarily be tied to a single VAL service. The user credentials uniquely identifies the VAL user to the identity management server.

NOTE: The specific security and authentication mechanisms required in order to use the user ID is specified in 3GPP TS 33.434 \[29\].

## 7.2 VAL user identity (VAL user ID)

The VAL user ID is a unique identifier within the VAL service that represents the VAL user. For example, the VAL user ID may be a URI. The VAL user ID is used for authentication and authorization purposes for providing the VAL service towards the VAL user via the VAL UE. The VAL user ID also indicates the VAL service provider with whom the VAL user has a VAL service agreement. The VAL user may have VAL service agreement with several VAL service providers and thus will have obtained unique VAL user ID per VAL service provider. The VAL user ID can be used to access any SEAL service.

Based on the service agreement, VAL user ID may be mapped to a VAL UE ID.

## 7.3 VAL UE identity (VAL UE ID)

The VAL UE ID is a unique identifier within the VAL service that represents the VAL UE. For example, the VAL UE ID for V2X service is mapped to the StationID as specified in ETSI TS 102 894-2 \[25\]. The VAL UE ID is used to address the VAL UE in order to send VAL messages or to access SEAL services.

Based on the service agreement, GPSI (as specified in 3GPP TS 23.003 \[38\]) may be used as VAL UE identity.

## 7.4 VAL service identity (VAL service ID)

The VAL service ID is a unique identifier that represents the VAL service. A VAL server provides a list of VAL services towards the VAL users or VAL UE. Each VAL service is uniquely identified by a VAL service ID, which is an identifier of the VAL application providing that VAL service. The VAL service ID can be used for policy mapping, QoS handling for VAL communication and VAL message distribution. For example, an identifier of a V2X service, e.g. ITS-AID or PSID specified in ETSI TS 102 965 \[26\] and ISO TS 17419 \[27\], can be used as a V2X service ID.

## 7.5 VAL group identity (VAL group ID)

The VAL group ID is a unique identifier within the VAL service that represents a set of VAL users or VAL UE according to the VAL service. The set of VAL users may belong to the same or different VAL service providers. It indicates the VAL application server where the group is defined.

## 7.6 VAL system identity (VAL system ID)

The VAL system ID is a globally unique identifier representing a VAL system.

## 7.7 VAL Stream ID

A VAL Stream ID is an identity used by the VAL server to identify a VAL stream.
