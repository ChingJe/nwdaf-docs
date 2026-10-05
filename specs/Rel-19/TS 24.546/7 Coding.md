---
spec: TS 24.546
version: 19.5.0
release: '19'
clause: 7
title: 7 Coding
source_archive: 24546-j50.zip
source_document: 24546-j50.docx
content_origin: 3gpp-source
---

# 7 Coding


## 7.1 VAL user profile document


### 7.1.1 General

### 7.1.2 Application unique ID

The AUID shall be set to the VAL service ID as specified in specific VAL service specification.

### 7.1.3 Data structure

The \<seal-user-profile\> element shall be the root element of the VAL user-profile configuration document.

The \<seal-user-profile\> element:

a\) may include a \<profile-name\> element;

b\) may include a \<status\> element;

c\) may include a \<is-default\> element;

d\) shall include a \<profile-configuration\> element which:

1\) may include a \<common\> element;

2\) may include a \<on-network\> element;

3\) may include a \<off-network\> element; and

e\) shall include a "user-profile-index" attribute; and

f\) may include any other attribute for the purposes of extensibility.

### 7.1.4 XML Schema

The seal user profile configuration document shall be composed according to the following XML schema:

\<?xml version="1.0" encoding="UTF-8"?\>

\<xs:schema

xmlns="urn:3gpp:ns:seal:SealUserProfile:1.0"

targetNamespace="urn:3gpp:ns:seal:SealUserProfile:1.0"

xmlns:xs="http://www.w3.org/2001/XMLSchema"

xmlns:sealup="urn:3gpp:ns:seal:SealUserProfile:1.0"

elementFormDefault="qualified"

attributeFormDefault="unqualified"\>

\<xs:import namespace="http://www.w3.org/XML/1998/namespace"

schemaLocation="http://www.w3.org/2001/xml.xsd"/\>

\<xs:element name="seal-user-profile"\>

\<xs:complexType\>

\<xs:choice minOccurs="1" maxOccurs="unbounded"\>

\<xs:element name="profile-name" type="sealup:NameType"/\>

\<xs:element name="status" type="xs:boolean"/\>

\<xs:element name="is-default" type="xs:boolean"/\>

\<xs:element name="profile-configuration" type="sealup:ProfileConfigurationType"/\>

\<xs:element name="anyExt" type="sealup:anyExtType" minOccurs="0"/\>

\<xs:any namespace="##other" processContents="lax" minOccurs="0" maxOccurs="unbounded"/\>

\</xs:choice\>

\<xs:attribute name="user-profile-index" type="xs:unsignedByte" use="required"/\>

\<xs:anyAttribute namespace="##any" processContents="lax"/\>

\</xs:complexType\>

\</xs:element\>

\<xs:complexType name="NameType"\>

\<xs:simpleContent\>

\<xs:extension base="xs:token"\>

\<xs:attribute ref="xml:lang"/\>

\</xs:extension\>

\</xs:simpleContent\>

\</xs:complexType\>

\<xs:complexType name="ProfileConfigurationType"\>

\<xs:choice minOccurs="1" maxOccurs="unbounded"\>

\<xs:element name="common" type="sealup:CommonType"/\>

\<xs:element name="on-network" type="sealup:OnNetworkType"/\>

\<xs:element name="off-network" type="sealup:OffNetworkType"/\>

\<xs:element name="anyExt" type="sealup:anyExtType" minOccurs="0"/\>

\<xs:any namespace="##other" processContents="lax" minOccurs="0" maxOccurs="unbounded"/\>

\</xs:choice\>

\</xs:complexType\>

\<xs:complexType name="CommonType" /\>

\<xs:complexType name="OnNetworkType" /\>

\<xs:complexType name="OffNetworkType" /\>

\<xs:complexType name="anyExtType"\>

\<xs:sequence\>

\<xs:any namespace="##any" processContents="lax" minOccurs="0" maxOccurs="unbounded"/\>

\</xs:sequence\>

\</xs:complexType\>

\</xs:schema\>

### 7.1.5 Semantics

The \<seal-user-profile\> element is the root element of the XML document.

The \<profile-name\> element of \<seal-user-profile\> element specifies the name of the SEAL user profile configuration document.

The \<status\> element of \<seal-user-profile\> element is of type "Boolean" and indicates whether this particular SEAL user profile is enabled or disabled.

The \<is-default\> element of \<seal-user-profile\> element is of type "Boolean" and indicates whether this particular SEAL user profile is default profile for VAL user or not.

The "user-profile-index" attribute element of \<seal-user-profile\> element contains a positive number which provides profile id. This attribute is used only when multiple user-profile for a VAL user is supported.

The \<profile-configuration\> element of \<seal-user-profile\> element contains actual profile configuration. The VAL application which uses SEAL user-profile may provide its own profile configuration specific to VAL application.

The VAL service may further extend the \<common\> element of the \<profile-configuration\> element of the \<seal-user-profile\> element to include VAL service specific common user profile configuration.

The VAL service may further extend the \<on-network\> element of the \<profile-configuration\> element of the \<seal-user-profile\> element to include VAL service specific user profile configuration for on-network features.

The VAL service may further extend the \<off-network\> element of the \<profile-configuration\> element of the \<seal-user-profile\> element to include VAL service specific user profile configuration for off-network features.

### 7.1.6 MIME type

The MIME type for VAL user profile configuration shall be set to "vnd.3gpp.seal-user-profile-info+xml".

### 7.1.7 IANA registration template

Your Name:

\<MCC name\>

Your Email Address:

\<MCC email address\>

Media Type Name:

Application

Subtype name:

vnd.3gpp.seal-user-profile-info+xml

Required parameters:

None

Optional parameters:

"charset" the parameter has identical semantics to the charset parameter of the "application/xml" media type as specified in section 9.1 of IETF RFC 7303.

Encoding considerations:

binary.

Security considerations:

Same as general security considerations for application/xml media type as specified in section 9.1 of IETF RFC 7303. In addition, this media type provides a format for exchanging information in SIP or in HTTP. So the security considerations from IETF RFC 3261 apply while exchanging information in SIP and the security considerations from IETF RFC 2616 apply while exchanging information in HTTP.

The information transported in this media type does not include active or executable content.

Mechanisms for privacy and integrity protection of protocol parameters exist. Those mechanisms as well as authentication and further security mechanisms are described in 3GPP TS 24.229.

This media type does not include provisions for directives that institute actions on a recipient's files or other resources.

This media type does not include provisions for directives that institute actions that, while not directly harmful to the recipient, may result in disclosure of information that either facilitates a subsequent attack or else violates a recipient's privacy in any way.

This media type does not employ compression.

Interoperability considerations:

Same as general interoperability considerations for application/xml media type as specified in section 9.1 of IETF RFC 7303. Any unknown XML elements and any unknown XML attributes are to be ignored by recipient of the MIME body.

Published specification:

3GPP TS 24.546 "Configuration management - Service Enabler Architecture Layer for Verticals (SEAL); Protocol specification", available via http://www.3gpp.org/specs/numbering.htm.

Applications Usage:

Applications supporting the SEAL configuration management procedures as described in the published specification.

Fragment identifier considerations:

The handling in section 5 of IETF RFC 7303 applies.

Restrictions on usage:

None

Provisional registration? (standards tree only):

N/A

Additional information:

1\. Deprecated alias names for this type: none

2\. Magic number(s): none

3\. File extension(s): none

4\. Macintosh File Type Code(s): none

5\. Object Identifier(s) or OID(s): none

Intended usage:

Common

Person to contact for further information:

\- Name: \<MCC name\>

\- Email: \<MCC email address\>

\- Author/Change controller:

i\) Author: 3GPP CT1 Working Group/3GPP_TSG_CT_WG1@LIST.ETSI.ORG

ii\) Change controller: \<MCC name\>/\<MCC email address\>

## 7.2 VAL UE configuration document


### 7.2.1 General

### 7.2.2 Application unique ID

The AUID shall be set to the VAL service ID as specified in specific VAL service specification.

### 7.2.3 Data structure

The SEAL UE configuration document structure is specified in this clause.

The \<seal-UE-configuration\> document:

1\) shall include a "domain" attribute;

2\) shall include a \<VAL-UE-id\> element;

3\) may include a \<VAL-service-id\> element;

4\) may include a \<name\> element;

5\) may include a \<common\> element;

6\) may include an \<on-network\> element; and

7\) may include any other attribute for the purposes of extensibility.

The \<VAL-UE-id\> element:

1\) may contain a list of \<instance-ID-URN\> elements; and

2\) may contain a list of \<IMEI-range\> elements.

The \<IMEI-range\> element:

1\) shall contain a \<TAC\> element;

2\) may contain a list of \<SNR\> elements; and

3\) may contain \<SNR-range\> element.

The \<SNR-range\> element:

1\) shall contain a \<low-SNR\> element; and

2\) shall contain a \<high-SNR\> element.

### 7.2.4 XML schema

\<?xml version="1.0" encoding="UTF-8"?\>

\<xs:schema

xmlns="urn:3gpp:ns:seal:sealUEConfig:1.0"

targetNamespace="urn:3gpp:ns:seal:sealUEConfig:1.0"

xmlns:xs="http://www.w3.org/2001/XMLSchema"

xmlns:sealuec="urn:3gpp:ns:seal:sealUEConfig:1.0"

elementFormDefault="qualified"

attributeFormDefault="unqualified"\>

\<xs:import namespace="http://www.w3.org/XML/1998/namespace"

schemaLocation="http://www.w3.org/2001/xml.xsd"/\>

\<xs:element name="seal-UE-configuration"\>

\<xs:complexType\>

\<xs:sequence\>

\<xs:choice minOccurs="0" maxOccurs="unbounded"\>

\<xs:element name="VAL-UE-id" type="sealuec:VALUEIDType"/\>

\<xs:element name="VAL-service-id" type="xs:string"/\>

\<xs:element name="name" type="sealuec:NameType"/\>

\</xs:choice\>

\<xs:element name="common" type="sealuec:CommonType"/\>

\<xs:element name="on-network" type="sealuec:On-networkType"/\>

\<xs:element name="anyExt" type="sealuec:anyExtType" minOccurs="0"/\>

\<xs:any namespace="##other" processContents="lax" minOccurs="0" maxOccurs="unbounded"/\>

\</xs:sequence\>

\<xs:attribute name="domain" type="xs:anyURI" use="required"/\>

\<xs:anyAttribute namespace="##any" processContents="lax"/\>

\</xs:complexType\>

\</xs:element\>

\<xs:complexType name="NameType"\>

\<xs:simpleContent\>

\<xs:extension base="xs:token"\>

\<xs:attribute ref="xml:lang"/\>

\<xs:attributeGroup ref="sealuec:IndexType"/\>

\</xs:extension\>

\</xs:simpleContent\>

\</xs:complexType\>

\<xs:complexType name="VALUEIDType"\>

\<xs:choice minOccurs="0" maxOccurs="unbounded"\>

\<xs:element name="instance-ID-URN" type="xs:anyURI"/\>

\<xs:element name="IMEI-range" type="sealuec:IMEI-rangeType"/\>

\<xs:element name="anyExt" type="sealuec:anyExtType" minOccurs="0"/\>

\<xs:any namespace="##other" processContents="lax"/\>

\</xs:choice\>

\<xs:attributeGroup ref="sealuec:IndexType"/\>

\<xs:anyAttribute namespace="##any" processContents="lax"/\>

\</xs:complexType\>

\<xs:complexType name="IMEI-rangeType"\>

\<xs:sequence\>

\<xs:element name="TAC" type="sealuec:tacType"/\>

\<xs:choice minOccurs="0" maxOccurs="unbounded"\>

\<xs:element name="SNR" type="sealuec:snrType"/\>

\<xs:element name="SNR-range" type="sealuec:SNR-rangeType"/\>

\</xs:choice\>

\<xs:element name="anyExt" type="sealuec:anyExtType" minOccurs="0"/\>

\<xs:any namespace="##other" processContents="lax" minOccurs="0" maxOccurs="unbounded"/\>

\</xs:sequence\>

\<xs:attributeGroup ref="sealuec:IndexType"/\>

\<xs:anyAttribute namespace="##any" processContents="lax"/\>

\</xs:complexType\>

\<xs:complexType name="SNR-rangeType"\>

\<xs:sequence\>

\<xs:element name="low-SNR" type="sealuec:snrType"/\>

\<xs:element name="high-SNR" type="sealuec:snrType"/\>

\<xs:element name="anyExt" type="sealuec:anyExtType" minOccurs="0"/\>

\<xs:any namespace="##other" processContents="lax" minOccurs="0" maxOccurs="unbounded"/\>

\</xs:sequence\>

\<xs:attributeGroup ref="sealuec:IndexType"/\>

\<xs:anyAttribute namespace="##any" processContents="lax"/\>

\</xs:complexType\>

\<xs:simpleType name="tac-baseType"\>

\<xs:restriction base="xs:decimal"\>

\<xs:totalDigits value="8"/\>

\</xs:restriction\>

\</xs:simpleType\>

\<xs:complexType name="tacType"\>

\<xs:simpleContent\>

\<xs:extension base="sealuec:tac-baseType"\>

\<xs:attributeGroup ref="sealuec:IndexType"/\>

\<xs:anyAttribute namespace="##any" processContents="lax"/\>

\</xs:extension\>

\</xs:simpleContent\>

\</xs:complexType\>

\<xs:simpleType name="snr-baseType"\>

\<xs:restriction base="xs:decimal"\>

\<xs:totalDigits value="6"/\>

\</xs:restriction\>

\</xs:simpleType\>

\<xs:complexType name="snrType"\>

\<xs:simpleContent\>

\<xs:extension base="sealuec:snr-baseType"\>

\<xs:attributeGroup ref="sealuec:IndexType"/\>

\<xs:anyAttribute namespace="##any" processContents="lax"/\>

\</xs:extension\>

\</xs:simpleContent\>

\</xs:complexType\>

\<xs:complexType name="CommonType" /\>

\<xs:complexType name="On-networkType" /\>

\<xs:attributeGroup name="IndexType"\>

\<xs:attribute name="index" type="xs:token"/\>

\</xs:attributeGroup\>

\<xs:complexType name="anyExtType"\>

\<xs:sequence\>

\<xs:any namespace="##any" processContents="lax" minOccurs="0" maxOccurs="unbounded"/\>

\</xs:sequence\>

\</xs:complexType\>

\</xs:schema\>

### 7.2.5 Semantics

The "domain" attribute of the \<seal-UE-configuration\> element contains the domain name of the VAL service.

The \<name\> element of the \<seal-UE- configuration\> element contains the user displayable name of the SEAL UE configuration document.

The creator of the SEAL UE configuration document may include an \<VAL-UE-id\> element in the version of the SEAL UE configuration document that is uploaded to the SCM-S. If an \<VAL-UE-id\> element is included then the SEAL UE configuration document applies only to the VAL UE(s) identified by the \<VAL-UE-id\> element. If no \<VAL-UE-id\> element is included then the SEAL UE configuration document applies to all the VAL UEs of the domain.

The \<VAL-service-id\> element contains identify of the VAL service for which the configuration document is applicable.

If one or more optional \<instance-ID-URN\> elements is included in the \<VAL-UE-id\> element then the SEAL UE configuration document applies to the VAL UE with an instance ID equal to the instance ID contained in the \<Instance-ID-URN\> element.

The \<TAC\> element of the \<IMEI-range\> element contains the Type Allocation Code of the VAL UE.

The optional \<SNR\> element of the \<IMEI-range\> element contains the individual serial number uniquely identifying VAL UE within the Type Allocation Code contained in the \<TAC\> element that the SEAL UE configuration document applies to.

If an optional \<SNR-range\> element is included within the \<IMEI-range\> element then the SEAL UE configuration document applies to all VAL UEs within the Type Allocation Code contained in the \<TAC\> element with the serial number equal or greater than the serial number contained in the \<low-SNR\> element and less than or equal to the serial number contained in the \<high-SNR\> element.

If no \<SNR\> element nor \<SNR-range\> element is included within the \<IMEI-range\> element then the SEAL UE configuration document applies to all the VAL UE(s) with the Type Allocation Code contained within the \<TAC\> element of the \<IMEI-range\> element.

If no \<VAL-UE-id\> element is included then the SEAL UE configuration document applies to all VAL UEs of the VAL service identified in the "domain" attribute.

The VAL service may further extend the \<common\> element of the \<seal-UE-configuration\> to include VAL service specific common UE configuration.

The VAL service may further extend the \<on-network\> element of the \<seal-UE-configuration\> to include VAL service specific UE configuration for on-network features.

### 7.2.6 MIME type

The MIME type for VAL user profile configuration shall be set to "vnd.3gpp.seal-ue-config-info+xml".

### 7.2.7 IANA registration template

Your Name:

\<MCC name\>

Your Email Address:

\<MCC email address\>

Media Type Name:

Application

Subtype name:

vnd.3gpp.seal-ue-config-info+xml

Required parameters:

None

Optional parameters:

"charset" the parameter has identical semantics to the charset parameter of the "application/xml" media type as specified in section 9.1 of IETF RFC 7303.

Encoding considerations:

binary.

Security considerations:

Same as general security considerations for application/xml media type as specified in section 9.1 of IETF RFC 7303. In addition, this media type provides a format for exchanging information in SIP or in HTTP. So the security considerations from IETF RFC 3261 apply while exchanging information in SIP and the security considerations from IETF RFC 2616 apply while exchanging information in HTTP.

The information transported in this media type does not include active or executable content.

Mechanisms for privacy and integrity protection of protocol parameters exist. Those mechanisms as well as authentication and further security mechanisms are described in 3GPP TS 24.229.

This media type does not include provisions for directives that institute actions on a recipient's files or other resources.

This media type does not include provisions for directives that institute actions that, while not directly harmful to the recipient, may result in disclosure of information that either facilitates a subsequent attack or else violates a recipient's privacy in any way.

This media type does not employ compression.

Interoperability considerations:

Same as general interoperability considerations for application/xml media type as specified in section 9.1 of IETF RFC 7303. Any unknown XML elements and any unknown XML attributes are to be ignored by recipient of the MIME body.

Published specification:

3GPP TS 24.546 "Configuration management - Service Enabler Architecture Layer for Verticals (SEAL); Protocol specification", available via http://www.3gpp.org/specs/numbering.htm.

Applications Usage:

Applications supporting the SEAL configuration management procedures as described in the published specification.

Fragment identifier considerations:

The handling in section 5 of IETF RFC 7303 applies.

Restrictions on usage:

None

Provisional registration? (standards tree only):

N/A

Additional information:

1\. Deprecated alias names for this type: none

2\. Magic number(s): none

3\. File extension(s): none

4\. Macintosh File Type Code(s): none

5\. Object Identifier(s) or OID(s): none

Intended usage:

Common

Person to contact for further information:

\- Name: \<MCC name\>

\- Email: \<MCC email address\>

\- Author/Change controller:

i\) Author: 3GPP CT1 Working Group/3GPP_TSG_CT_WG1@LIST.ETSI.ORG

ii\) Change controller: \<MCC name\>/\<MCC email address\>

## 7.3 VAL UE satellite information document


### 7.3.1 General

### 7.3.2 Application unique ID

The AUID shall be set to the VAL service ID as specified in specific VAL service specification.

### 7.3.3 Data structure

The SEAL UE satellite information document structure is specified in this clause.

The \<satellite-info\> document:

a\) may include a \<VAL-service-id\> element;

b\) may include a \<asca-info\> element; and

c\) may include any other attribute for the purposes of extensibility.

The \<asca-info\> element shall include \<satellite-coverage-list\> element.

The \<satellite-coverage-list\> shall include one or more \<satellite-coverage\> element.

The \<satellite-coverage\> element:

a\) shall contain a \<satellite-id\> element;

b\) shall contain a \<geographic-area\> element which:

1\) may include a \<point-uncertainty-circle\> element;

2\) may include a \<point-uncertainty-ellipse\> element; and

3\) may include a \<polygon\> element;

c\) shall include one or more \<time-window\> element which includes:

1\) a \<start-time\> element; and

2\) a \<stop-time\> element; and

d\) shall include a \<rat-type\> element.

### 7.3.4 XML schema

\<?xml version="1.0" encoding="UTF-8"?\>

\<xs:schema

xmlns="urn:3gpp:ns:seal:sealsatelliteinfo:1.0"

targetNamespace="urn:3gpp:ns:seal:sealsatelliteinfo:1.0"

xmlns:xs="http://www.w3.org/2001/XMLSchema"

xmlns:sealuesat="urn:3gpp:ns:seal:sealsatelliteinfo:1.0"

xmlns:sealloc="urn:3gpp:ns:sealLocationInfo:1.0"

elementFormDefault="qualified"

attributeFormDefault="unqualified"\>

\<!-- the sealloc.xsd shall be referenced from 3GPP TS 24.545 --\>

\<xs:import namespace="urn:3gpp:ns:sealLocationInfo:1.0" schemaLocation="sealloc.xsd"/\>

\<xs:import namespace="http://www.w3.org/XML/1998/namespace"

schemaLocation="http://www.w3.org/2001/xml.xsd"/\>

\<xs:element name="satellite-info"\>

\<xs:complexType\>

\<xs:choice maxOccurs="unbounded"\>

\<xs:element name="VAL-service-id" type="xs:string"/\>

\<xs:element name="asca-info" type="sealuesat:AscaInfoType"/\>

\<xs:element name="anyExt" type="sealuesat:anyExtType" minOccurs="0"/\>

\<xs:any namespace="##other" processContents="lax" minOccurs="0" maxOccurs="unbounded"/\>

\</xs:choice\>

\<xs:anyAttribute namespace="##any" processContents="lax"/\>

\</xs:complexType\>

\</xs:element\>

\<xs:complexType name="anyExtType"\>

\<xs:sequence\>

\<xs:any namespace="##any" processContents="lax" minOccurs="0" maxOccurs="unbounded"/\>

\</xs:sequence\>

\</xs:complexType\>

\<xs:complexType name="AscaInfoType"\>

\<xs:sequence\>

\<xs:element name="satellite-coverage-list" type="sealuesat:SatelliteCoverageListType"/\>

\<xs:element name="anyExt" type="sealuesat:anyExtType" minOccurs="0"/\>

\<xs:any namespace="##other" processContents="lax"/\>

\</xs:sequence\>

\<xs:anyAttribute namespace="##any" processContents="lax"/\>

\</xs:complexType\>

\<xs:complexType name="SatelliteCoverageListType"\>

\<xs:choice minOccurs="0" maxOccurs="unbounded"\>

\<xs:element name="satellite-coverage" type="sealuesat:SatelliteCoverageType"/\>

\<xs:element name="anyExt" type="sealuesat:anyExtType" minOccurs="0"/\>

\<xs:any namespace="##other" processContents="lax"/\>

\</xs:choice\>

\<xs:anyAttribute namespace="##any" processContents="lax"/\>

\</xs:complexType\>

\<xs:complexType name="SatelliteCoverageType"\>

\<xs:choice minOccurs="0" maxOccurs="unbounded"\>

\<xs:element name="satellite-id" type="unsignedByte"\>

\<xs:annotation\>

\<xs:documentation\>

SatelliteId per 3GPP TS 36.331 clause 6.3.6: unsigned 1 octet integer, range 0..255.

\</xs:documentation\>

\</xs:annotation\>

\</xs:element\>

\<xs:element name="geographic-area" type="sealuesat:GeographicAreaType"/\>

\<xs:element name="time-window" type="sealuesat:tTimeWindowType"/\>

\<xs:element name="rat-type" type="xs:string"/\>

\<xs:element name="anyExt" type="sealuesat:anyExtType" minOccurs="0"/\>

\<xs:any namespace="##other" processContents="lax"/\>

\</xs:choice\>

\<xs:anyAttribute namespace="##any" processContents="lax"/\>

\</xs:complexType\>

\<xs:complexType name="GeographicAreaType"\>

\<xs:choice minOccurs="0" maxOccurs="unbounded"\>

\<xs:element name="point-uncertainty-circle" type="sealuesat:tPointUncertaintyCircle"/\>

\<xs:element name="point-uncertainty-ellipse" type="sealuesat:PointUncertaintyEllipse"/\>

\<xs:element name="polygon" type="sealuesat:tPolygonAreaType"/\>

\<xs:element name="anyExt" type="sealuesat:anyExtType" minOccurs="0"/\>

\<xs:any namespace="##other" processContents="lax"/\>

\</xs:choice\>

\<xs:anyAttribute namespace="##any" processContents="lax"/\>

\</xs:complexType\>

\<xs:complexType name="tPointUncertaintyCircle"\>

\<xs:sequence\>

\<xs:element name="point" type="sealloc:tPointCoordinate"/\>

\<xs:element name="uncertain-value" type="sealuesat:tUncertaintyType"/\>

\<xs:any namespace="##other" processContents="lax" minOccurs="0" maxOccurs="unbounded"/\>

\<xs:element name="anyExt" type="sealuesat:anyExtType" minOccurs="0"/\>

\</xs:sequence\>

\<xs:anyAttribute namespace="##any" processContents="lax"/\>

\</xs:complexType\>

\<xs:simpleType name="tUncertaintyType"\>

\<xs:annotation\>

\<xs:documentation\>

Indicates value of uncertainty in meters.

\</xs:documentation\>

\</xs:annotation\>

\<xs:restriction base="xs:float"\>

\<xs:minInclusive value="0"/\>

\</xs:restriction\>

\</xs:simpleType\>

\<xs:complexType name="PointUncertaintyEllipse"\>

\<xs:choice minOccurs="0" maxOccurs="unbounded"\>

\<xs:element name="point" type="sealloc:tPointCoordinate"/\>

\<xs:element name="uncertainty-ellipse" type="sealuesat:UncertaintyEllipse"/\>

\<xs:element name="confidence" type="xs:string"/\>

\<xs:element name="anyExt" type="sealuesat:anyExtType" minOccurs="0"/\>

\<xs:any namespace="##other" processContents="lax"/\>

\</xs:choice\>

\<xs:anyAttribute namespace="##any" processContents="lax"/\>

\</xs:complexType\>

\<xs:complexType name="UncertaintyEllipse"\>

\<xs:choice minOccurs="0" maxOccurs="unbounded"\>

\<xs:element name="semi-major" type="sealuesat:tUncertaintyType"/\>

\<xs:element name="semi-minor" type="sealuesat:tUncertaintyType"/\>

\<xs:element name="orientation-major " type="sealuesat:tOrientationType"/\>

\<xs:element name="anyExt" type="sealuesat:anyExtType" minOccurs="0"/\>

\<xs:any namespace="##other" processContents="lax"/\>

\</xs:choice\>

\<xs:anyAttribute namespace="##any" processContents="lax"/\>

\</xs:complexType\>

\<xs:simpleType name="tOrientationType"\>

\<xs:annotation\>

\<xs:documentation\>

Indicates value of orientation angle.

\</xs:documentation\>

\</xs:annotation\>

\<xs:restriction base="xs:integer"\>

\<xs:minInclusive value="0"/\>

\<xs:maxInclusive value="180"/\>

\</xs:restriction\>

\</xs:simpleType\>

\<xs:complexType name="tPolygonAreaType"\>

\<xs:sequence\>

\<xs:element name="point-list" type="sealloc:tPointCoordinate" minOccurs="3" maxOccurs="15"/\>

\<xs:any namespace="##other" processContents="lax" minOccurs="0" maxOccurs="unbounded"/\>

\<xs:element name="anyExt" type="sealuesat:anyExtType" minOccurs="0"/\>

\</xs:sequence\>

\<xs:anyAttribute namespace="##any" processContents="lax"/\>

\</xs:complexType\>

\<xs:complexType name="tTimeWindowType"\>

\<xs:sequence\>

\<xs:element name="start-time" type="xs:dateTime"/\>

\<xs:element name="stop-time" type="xs:dateTime"/\>

\<xs:any namespace="##other" processContents="lax" minOccurs="0" maxOccurs="unbounded"/\>

\<xs:element name="anyExt" type="sealuesat:anyExtType" minOccurs="0"/\>

\</xs:sequence\>

\<xs:anyAttribute namespace="##any" processContents="lax"/\>

\</xs:complexType\>

\</xs:schema\>

### 7.3.5 Semantics

The \<satellite-info\> element is the root element of the XML document. The \<satellite-info\> element contains the \<VAL-service-id\> and \<asca-info\> sub-elements.

\<VAL-service-id\> is an optional element set to the the identity of the VAL service.

The \<asca-info\> element shall include \<satellite-coverage-list\> element.

The \<satellite-coverage-list\> shall include one or more \<satellite-coverage\> element.

The \<satellite-coverage\> element:

a\) shall contain a \<satellite-id\> element set to the dedicated satellite ID for the requested VAL UE. The value part of the satellite-id is an unsigned byte coded value from 0 to 255 (see SatelliteId IE in 3GPP TS 36.331 \[33\] clause 6.3.6);

b\) shall contain a \<geographic-area\> element set to the geographic location information for the satellite coverage or the current location data for the VAL UE. The\<geographic-area\> element:

1\) may include a \<point-uncertainty-circle\> element contains a location co-ordinate as defined as "tPointCoordinate";

2\) may include a \<point-uncertainty-ellipse\> element contains a location co-ordinate as defined as "tPointCoordinate"; and

3\) may include a \<polygon\> element contains a location co-ordinate as defined as "tPointCoordinate";

c\) shall include one or more \<time-window\> element which includes:

1\) a \<start-time\> element set to the start time of the periods, consisting of a string with format "DateTime"; and

2\) a \<stop-time\> element set to the end time of the periods, consisting of a string with format "DateTime"; and

d\) shall include a \<rat-type\> element indicatings the satellite RAT types corresponding to the satellite availability in the indicated geographical area. Table 7.3.5-1 provides the valid values of rat-type.

Table 7.3.5-1: Rat-type

| Value                         | Description                         |
|-------------------------------|-------------------------------------|
| "NR_LEO"                      | NR (LEO) satellite access type      |
| "NR_MEO"                      | NR (MEO) satellite access type      |
| "NR_GEO"                      | NR (GEO) satellite access type      |
| "NR_OTHER_SAT"                | NR (OTHERSAT) satellite access type |
| Other value shall be ignored. |                                     |

### 7.3.6 MIME type

The MIME type for VAL user profile configuration shall be set to "vnd.3gpp.seal-satellite-info+xml".

### 7.3.7 IANA registration template

Your Name:

\<MCC name\>

Your Email Address:

\<MCC email address\>

Media Type Name:

Application

Subtype name:

application/vnd.3gpp.seal-satellite-info+xml

Required parameters:

None

Optional parameters:

"charset" the parameter has identical semantics to the charset parameter of the "application/xml" media type as specified in section 9.1 of IETF RFC 7303.

Encoding considerations:

binary.

Security considerations:

Same as general security considerations for application/xml media type as specified in section 9.1 of IETF RFC 7303. In addition, this media type provides a format for exchanging information in SIP or in HTTP. So the security considerations from IETF RFC 3261 apply while exchanging information in SIP and the security considerations from IETF RFC 2616 apply while exchanging information in HTTP.

The information transported in this media type does not include active or executable content.

Mechanisms for privacy and integrity protection of protocol parameters exist. Those mechanisms as well as authentication and further security mechanisms are described in 3GPP TS 24.229.

This media type does not include provisions for directives that institute actions on a recipient's files or other resources.

This media type does not include provisions for directives that institute actions that, while not directly harmful to the recipient, may result in disclosure of information that either facilitates a subsequent attack or else violates a recipient's privacy in any way.

This media type does not employ compression.

Interoperability considerations:

Same as general interoperability considerations for application/xml media type as specified in section 9.1 of IETF RFC 7303. Any unknown XML elements and any unknown XML attributes are to be ignored by recipient of the MIME body.

Published specification:

3GPP TS 24.546 "Configuration management - Service Enabler Architecture Layer for Verticals (SEAL); Protocol specification", available via http://www.3gpp.org/specs/numbering.htm.

Applications Usage:

Applications supporting the SEAL configuration management for the use of application satellite coverage information procedures as described in the published specification.

Fragment identifier considerations:

The handling in section 5 of IETF RFC 7303 applies.

Restrictions on usage:

None

Provisional registration? (standards tree only):

N/A

Additional information:

1\. Deprecated alias names for this type: none

2\. Magic number(s): none

3\. File extension(s): none

4\. Macintosh File Type Code(s): none

5\. Object Identifier(s) or OID(s): none

Intended usage:

Common

Person to contact for further information:

\- Name: \<MCC name\>

\- Email: \<MCC email address\>

\- Author/Change controller:

i\) Author: 3GPP CT1 Working Group/3GPP_TSG_CT_WG1@LIST.ETSI.ORG

ii\) Change controller: \<MCC name\>/\<MCC email address\>
