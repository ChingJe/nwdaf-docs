---
spec: TS 24.546
version: 19.5.0
release: '19'
clause: C.1
title: C.1 General
source_archive: 24546-j50.zip
source_document: 24546-j50.docx
content_origin: 3gpp-source
---

# C.1 General

The information in this annex provides a normative description of CoAP resource representation and encoding.

## C.1.1 Resource URI structure

All API URIs of SEAL-UU APIs shall be specified as follows:

{apiRoot}/\<apiName\>/\<apiVersion\>

"apiRoot" is configured by means outside the scope of the present document. It includes one of the schemes ("coaps", "coaps+tcp", "coaps+ws"), host and optional port, and an optional prefix string. "apiName" and "apiVersion" shall be set dependent on the API, as defined in the corresponding clauses below.

All resource URIs specified for SEAL-UU APIs shall be defined relative to the above root API URI.

URIs which differ only in the scheme shall point to the same resource.

NOTE: The "apiVersion" will only be increased if the new API version contains backward incompatible changes.

The root structure may be followed by "apiSpecificSuffixes" that are dependent on the API and are defined separately for each API as resource URI where they apply:

{apiRoot}/\<apiName\>/\<apiVersion\>/\<apiSpecificSuffixes\>

## C.1.2 Use of cache

It is recommended for the SEAL clients and servers to support and use the caching mechanism specified in IETF RFC 7252 \[12\]. This implies support and use of the Max-Age and ETag options.

## C.1.3 Error handling

Table C.1.3-1 lists response payload types that are applicable to all APIs and as responses for all requests in the present specification unless otherwise specified. The CoAP client shall mandatorily support the processing of the status code for all the applicable methods, when received in a CoAP response message.

Table C.1.3-1: Response payloads supported for responses to all requests.

<table>
<colgroup>
<col style="width: 10%" />
<col style="width: 14%" />
<col style="width: 11%" />
<col style="width: 11%" />
<col style="width: 40%" />
<col style="width: 10%" />
</colgroup>
<tbody>
<tr class="odd">
<td rowspan="13">Response body</td>
<td>Data type</td>
<td>Cardinality</td>
<td><p>Response</p>
<p>Codes</p>
<p>(NOTE)</p></td>
<td>Remarks</td>
<td>Applied Methods</td>
</tr>
<tr class="even">
<td>ProblemDetails</td>
<td>1</td>
<td>4.00 Bad Request</td>
<td>Incorrect parameters were passed in the request.</td>
<td>GET, FETCH, POST PUT, PATCH, iPATCH, DELETE</td>
</tr>
<tr class="odd">
<td>ProblemDetails</td>
<td>1</td>
<td>4.01 Unauthorized</td>
<td>The client is not authorized.</td>
<td>GET, FETCH, POST, PUT, PATCH, iPATCH, DELETE</td>
</tr>
<tr class="even">
<td>ProblemDetails</td>
<td>1</td>
<td>4.02 Bad Option</td>
<td>The request could not be understood by the server due to one or more unrecognized or malformed options.</td>
<td>GET, FETCH, POST, PUT, PATCH, iPATCH, DELETE</td>
</tr>
<tr class="odd">
<td>ProblemDetails</td>
<td>1</td>
<td>4.03 Forbidden</td>
<td><p>This represents the case when the server is able to understand the request but unable to fulfil the request due to errors (e.g. the requested parameters are out of range).</p>
<p>More information may be provided in the "invalidParams" attribute of the "ProblemDetails" structure.</p></td>
<td>GET, FETCH, POST, PUT, PATCH, iPATCH, DELETE</td>
</tr>
<tr class="even">
<td>ProblemDetails</td>
<td>1</td>
<td>4.04 Not Found</td>
<td>The resource URI was incorrect.</td>
<td>GET, POST, PUT, PATCH, iPATCH, DELETE</td>
</tr>
<tr class="odd">
<td>ProblemDetails</td>
<td>1</td>
<td>4.06 Not Acceptable</td>
<td>The content format provided in the "Accept" option is not acceptable by the server.</td>
<td>GET, FETCH,</td>
</tr>
<tr class="even">
<td>ProblemDetails</td>
<td>1</td>
<td>4.13 Request Entity Too Large</td>
<td>If the received CoAP request contains entity larger than the server is able to process, the server shall reject the CoAP request with this status code. The server should include Size1 option in the response with the maximum size of the request entity it can handle.</td>
<td>FETCH, POST, PUT, PATCH, iPATCH,</td>
</tr>
<tr class="odd">
<td>ProblemDetails</td>
<td>1</td>
<td>4.15 Unsupported Content-Format</td>
<td>The code indicates that the resource is in a format which is not supported by the server for the method.</td>
<td>FETCH, POST, PUT, PATCH, iPATCH</td>
</tr>
<tr class="even">
<td>ProblemDetails</td>
<td>1</td>
<td><p>4.22</p>
<p>Unprocessable Entity</p></td>
<td>The code indicates the server is unable to or is incapable of processing the request.</td>
<td>FETCH, PATCH, iPATCH</td>
</tr>
<tr class="odd">
<td>ProblemDetails</td>
<td>1</td>
<td>4.29 Too Many Requests</td>
<td><p>The code indicates that due to excessive traffic which, if continued over time, may lead to (or may increase) an overload situation.</p>
<p>The CoAP option "Max-Age" may be added in the response to indicate how long the client has to wait before making a new request.</p></td>
<td>GET, FETCH, POST, PUT, PATCH, iPATCH, DELETE</td>
</tr>
<tr class="even">
<td>ProblemDetails</td>
<td>1</td>
<td>5.00 Internal Server Error</td>
<td>The server encountered an unexpected condition that prevented it from fulfilling the request.</td>
<td>GET, FETCH, POST, PUT, PATCH, iPATCH, DELETE</td>
</tr>
<tr class="odd">
<td>ProblemDetails</td>
<td>1</td>
<td>5.03 Service Unavailable</td>
<td>The server is unable to handle the request.</td>
<td>GET, FETCH, POST, PUT, PATCH, iPATCH, DELETE</td>
</tr>
<tr class="even">
<td colspan="6"><p>NOTE 1: In addition to the above response codes, the CoAP server may also send other valid CoAP response codes, if applicable. The list of all valid CoAP response codes can be found in CoAP Response Code Registry at IANA [19].</p>
<p>NOTE 2: CBOR encoding and media type "application/concise-problem-details+cbor" ProblemDetails shall be used for ProblemDetails as defined in the IETF RFC 9290 [20].</p></td>
</tr>
</tbody>
</table>

Specific errors are contained in the related API definition for each API.

## C.1.4 Data types applicable to multiple resource representations


### C.1.4.1 General

This clause defines structured data types, simple data types, and enumerations that are applicable to several APIs defined for CoAP resource representations in the present specification and other SEAL specifications and can be referenced from data structures defined in the subsequent clauses and from CoAP resource representations in other SEAL specifications.

NOTE: As a convention, data type names in the present specification follows UpperCamel and parameters follows lowerCamel as specified in clause 5.1.1 of 3GPP TS 29.501 \[23\].

### C.1.4.2 Referenced structured data types

Table C.1.4.2-1 lists structured data types referenced by multiple CoAP resource representations and defined in this specification or in other specifications.

Table C.1.4.2-1: Referenced Structured Data Types

| Data type                  | Reference             | Description                                                           |
|----------------------------|-----------------------|-----------------------------------------------------------------------|
| ValTargetUe                | Clause C.2.1.3.2.4    | Information identifying a VAL user ID or VAL UE ID.                   |
| ScheduledCommunicationTime | Clause C.1.4.4.1      | Defines time schedule for communication.                              |
| LocationQoS                | 3GPP TS 29.572 \[29\] | Defines the location quality of service, of the location information. |

### C.1.4.3 Referenced simple data types and enumerations

The simple datatypes based on the CBOR types are defined in table C.1.4.3-1 and the simple data types defined in table C.1.4.3-2 apply to multiple SEAL-UU APIs.

Table C.1.4.3-1: CBOR-based data types

|           |                                                                                                                                               |
|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------|
| Type name | Description                                                                                                                                   |
| bytes     | Is a "byte string" as defined in IETF RFC 8949 \[17\].                                                                                        |
| boolean   | Is a type which has 2 values "false" and "true" with the values as defined in IETF RFC 8949 \[17\].                                           |
| integer   | As defined in IETF RFC 8949 \[17\].                                                                                                           |
| number    | Is any number as defined in IETF RFC 8949 \[17\]. Precision format (half-precision, single-precision, and double-precision) can be indicated. |
| string    | Is a "text string" as defined in IETF RFC 8949 \[17\].                                                                                        |

Table C.1.4.3-2: Simple data types applicable to multiple CoAP resource representations

|                    |                                                                                                                                                                                                                                                                                  |
|--------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Type name          | Description                                                                                                                                                                                                                                                                      |
| Altitude           | Number indicating value of altitude in the range from minimum value -32767 to maximum value 32767.                                                                                                                                                                               |
| Angle              | Integer indicating a value of the angle in the range from minimum value 0 to maximum value 360.                                                                                                                                                                                  |
| CellId             | String containing a unique identifier of a cell.                                                                                                                                                                                                                                 |
| Confidence         | Integer indicating a value of confidence in the range from minimum value 0 to maximum value 100.                                                                                                                                                                                 |
| ExternalGroupId    | String containing a local identifier followed by "@" and a domain identifier. Both the local identifier and the domain identifier shall be encoded as strings that do not contain any "@" characters. See clauses 4.6.2 and 4.6.3 of 3GPP TS 23.682 \[24\] for more information. |
| DateTime           | Is a string in the standard format described by the "date-time" production in IETF RFC3339 \[25\].                                                                                                                                                                               |
| DayOfWeek          | Integer between and including 1 and 7 denoting a weekday. 1 shall indicate Monday, and the subsequent weekdays shall be indicated with the next higher numbers. 7 shall indicate Sunday.                                                                                         |
| GeographicalAreaId | String identifying a geographical area.                                                                                                                                                                                                                                          |
| InnerRadius        | Integer indicating a value of the inner radius in the range from minimum value 0 to maximum value 327675.                                                                                                                                                                        |
| MbmsSaId           | String containing a unique identifier of a MBMS serving area.                                                                                                                                                                                                                    |
| MbsfnAreaId        | String containing a unique identifier of a MBSFN area.                                                                                                                                                                                                                           |
| Orientation        | Integer indicating a value of orientation angle in the range from minimum value 0 to maximum value 180.                                                                                                                                                                          |
| PlmnId             | String containing a unique identifier of a PLMN.                                                                                                                                                                                                                                 |
| TaId               | String containing a unique identifier of a tracking area.                                                                                                                                                                                                                        |
| Tmgi               | Byte string containing an identifier of Temporary Mobile Group Identity. The contents of Tmgi are coded as octet 3 and above of Temporary Mobile Group Identity (TMGI) IE as defined in 3GPP TS 24.008 \[28\] clause 10.5.6.13.                                                  |
| TimeOfDay          | String with format partial-time or full-time as defined in clause 5.6 of IETF RFC 3339 \[25\]. Examples, 20:15:00, 20:15:00-08:00 (for 8 hours behind UTC).                                                                                                                      |
| Uinteger           | Unsigned integer, i.e. only value 0 and values above 0 are permissible.                                                                                                                                                                                                          |
| Uncertainty        | Number indicating value of uncertainty with minimum value 0.                                                                                                                                                                                                                     |
| Uri                | String providing an URI formatted according to IETF RFC 3986 \[22\].                                                                                                                                                                                                             |

Table C.1.4.3-3 lists simple data types and enumerations referenced by multiple CoAP resource representations defined in this specification or in other specifications.

Table C.1.4.3-3: Enumerations applicable to multiple CoAP resource representations

|            |             |                                       |
|------------|-------------|---------------------------------------|
| Type name  | Reference   | Description                           |
| ConfigType | C.2.1.3.3.1 | Represents the type of configuration. |

### C.1.4.4 Common structured data types


#### C.1.4.4.1 Type: ScheduledCommunicationTime

Table C.1.4.4.1-1: Definition of type ScheduledCommunicationTime

| Attribute name | Data type        | P   | Cardinality | Description                                                                       | Applicability |
|----------------|------------------|-----|-------------|-----------------------------------------------------------------------------------|---------------|
| daysOfWeek     | array(DayOfWeek) | O   | 0..6        | Identifies the day(s) of the week. If absent, it indicates every day of the week. |               |
| timeOfDayStart | TimeOfDay        | O   | 0..1        | Identifies the start time of the day.                                             |               |
| timeOfDayEnd   | TimeOfDay        | O   | 0..1        | Identifies the end time of the day.                                               |               |

#### C.1.4.4.2 Type: ProblemDetails

Table C.1.4.4.2-1: Definition of the ProblemDetails data type

<table>
<colgroup>
<col style="width: 18%" />
<col style="width: 19%" />
<col style="width: 11%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Attribute name</th>
<th>Data type</th>
<th>Cardinality</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>title</td>
<td>string</td>
<td>0..1</td>
<td>A short, human-readable summary of the problem type. It should not change from occurrence to occurrence of the problem.</td>
</tr>
<tr class="even">
<td>detail</td>
<td>string</td>
<td>0..1</td>
<td>A human-readable explanation specific to this occurrence of the problem.</td>
</tr>
<tr class="odd">
<td>instance</td>
<td>Uri</td>
<td>0..1</td>
<td>A URI reference that identifies the specific occurrence of the problem.</td>
</tr>
<tr class="even">
<td>cause</td>
<td>string</td>
<td>0..1</td>
<td><p>A machine-readable application error cause specific to this occurrence of the problem.</p>
<p>This IE should be present and provide application-related error information, if available.</p></td>
</tr>
<tr class="odd">
<td colspan="4"><p>NOTE 1: See IETF RFC 9290 [20] for detailed information and guidance for each attribute.</p>
<p>NOTE 2: Additional attributes may be defined per API.</p></td>
</tr>
</tbody>
</table>

#### C.1.4.4.3 Type: GeographicalCoordinates

Table C.1.4.4.3-1: Definition of type GeographicalCoordinates

| Attribute name | Data type | P   | Cardinality | Description                                                                                                                  |
|----------------|-----------|-----|-------------|------------------------------------------------------------------------------------------------------------------------------|
| lon            | number    | M   | 1           | Longitude expressed as a number with double precision in the range from and including value -180 to and including value 180. |
| lat            | number    | M   | 1           | Latitude expressed as a number with double precision in the range from and including value -90 to and including value 90.    |

#### C.1.4.4.4 Type: GeographicArea

Table C.1.4.4.4-1: Definition of type GeographicArea as a list of mutually exclusive alternatives

| Data type                | Cardinality | Discriminator property name | Discriminator mapping      | Description                                                                                  |
|--------------------------|-------------|-----------------------------|----------------------------|----------------------------------------------------------------------------------------------|
| Point                    | 1           | shape                       | POINT                      | Geographical area consisting of a single point, represented by its longitude and latitude.   |
| PointUncertaintyCircle   | 1           | shape                       | POINT_UNCERTAINTY_CIRCLE   | Geographical area consisting of a point and an uncertainty value.                            |
| PointUncertaintyEllipse  | 1           | shape                       | POINT_UNCERTAINTY_ELLIPSE  | Geographical area consisting of a point, plus an uncertainty ellipse and a confidence value. |
| Polygon                  | 1           | shape                       | POLYGON                    | Geographical area consisting of a list of points (between 3 to 15 points).                   |
| PointAltitude            | 1           | shape                       | POINT_ALTITUDE             | Geographical area consisting of a point and an altitude value.                               |
| PointAltitudeUncertainty | 1           | shape                       | POINT_ALTITUDE_UNCERTAINTY | Geographical area consisting of a point, an altitude value and an uncertainty value.         |
| EllipsoidArc             | 1           | shape                       | ELLIPSOID_ARC              | Geographical are consisting of an ellipsoid arc.                                             |

#### C.1.4.4.5 Type: Point

Table C.1.4.4.5-1: Definition of type Point

| Attribute name | Data type               | P   | Cardinality | Description                                                             |
|----------------|-------------------------|-----|-------------|-------------------------------------------------------------------------|
| shape          | SupportedGADShapes      | M   | 1           | It shall take the value "POINT".                                        |
| point          | GeographicalCoordinates | M   | 1           | Indicates a geographic point represented by its longitude and latitude. |

#### C.1.4.4.6 Type: PointUncertaintyCircle

Table C.1.4.4.6-1: Definition of type PointUncertaintyCircle

| Attribute name | Data type               | P   | Cardinality | Description                                                             |
|----------------|-------------------------|-----|-------------|-------------------------------------------------------------------------|
| shape          | SupportedGADShapes      | M   | 1           | It shall take the value "POINT_UNCERTAINTY_CIRCLE".                     |
| point          | GeographicalCoordinates | M   | 1           | Indicates a geographic point represented by its longitude and latitude. |
| uncertainty    | Uncertainty             | M   | 1           | Indicates the uncertainty value.                                        |

#### C.1.4.4.7 Type: PointUncertaintyEllipse

Table C.1.4.4.7-1: Definition of type PointUncertaintyEllipse

| Attribute name     | Data type               | P   | Cardinality | Description                                                             |
|--------------------|-------------------------|-----|-------------|-------------------------------------------------------------------------|
| shape              | SupportedGADShapes      | M   | 1           | It shall take the value "POINT_UNCERTAINTY_ELLIPSE".                    |
| point              | GeographicalCoordinates | M   | 1           | Indicates a geographic point represented by its longitude and latitude. |
| uncertaintyEllipse | UncertaintyEllipse      | M   | 1           | Indicates an uncertainty ellipse.                                       |
| confidence         | Confidence              | M   | 1           | Indicates the value of confidence.                                      |

#### C.1.4.4.8 Type: Polygon

Table C.1.4.4.8-1: Definition of type Polygon

| Attribute name | Data type                      | P   | Cardinality | Description                                             |
|----------------|--------------------------------|-----|-------------|---------------------------------------------------------|
| shape          | SupportedGADShapes             | M   | 1           | It shall take the value "POLYGON".                      |
| pointList      | array(GeographicalCoordinates) | M   | 3..15       | Array with up to15 items, where each item is a "point". |

#### C.1.4.4.9 Type: PointAltitude

Table C.1.4.4.9-1: Definition of type PointAltitude

| Attribute name | Data type               | P   | Cardinality | Description                                                             |
|----------------|-------------------------|-----|-------------|-------------------------------------------------------------------------|
| shape          | SupportedGADShapes      | M   | 1           | It shall take the value "POINT_ALTITUDE".                               |
| point          | GeographicalCoordinates | M   | 1           | Indicates a geographic point represented by its longitude and latitude. |
| altitude       | Altitude                | M   | 1           | Indicates the value of altitude.                                        |

#### C.1.4.4.10 Type: PointAltitudeUncertainty

Table C.1.4.4.10-1: Definition of type PointAltitudeUncertainty

| Attribute name      | Data type               | P   | Cardinality | Description                                                             |
|---------------------|-------------------------|-----|-------------|-------------------------------------------------------------------------|
| shape               | SupportedGADShapes      | M   | 1           | It shall take the value "POINT_ALTITUDE_UNCERTAINTY".                   |
| point               | GeographicalCoordinates | M   | 1           | Indicates a geographic point represented by its longitude and latitude. |
| altitude            | Altitude                | M   | 1           | Indicates the value of altitude.                                        |
| uncertaintyEllipse  | UncertaintyEllipse      | M   | 1           | Indicates the uncertainty ellipse                                       |
| uncertaintyAltitude | Uncertainty             | M   | 1           | Indicates the uncertainty of the altitude.                              |
| confidence          | Confidence              | M   | 1           | Indicates the value of confidence.                                      |

#### C.1.4.4.11 Type: EllipsoidArc

Table C.1.4.4.11-1: Definition of type EllipsoidArc

| Attribute name    | Data type               | P   | Cardinality | Description                                                             |
|-------------------|-------------------------|-----|-------------|-------------------------------------------------------------------------|
| shape             | SupportedGADShapes      | M   | 1           | It shall take the value "ELLIPSOID_ARC".                                |
| point             | GeographicalCoordinates | M   | 1           | Indicates a geographic point represented by its longitude and latitude. |
| innerRadius       | InnerRadius             | M   | 1           | Indicates the value of inner radius of the Ellipsoid Arc.               |
| uncertaintyRadius | Uncertainty             | M   | 1           | Indicates the uncertainty radius of the Ellipsoid Arc.                  |
| offsetAngle       | Angle                   | M   | 1           | Indicates the offset angle of the Ellipsoid Arc.                        |
| includedAngle     | Angle                   | M   | 1           | Indicates the included angle of the Ellipsoid Arc.                      |
| confidence        | Confidence              | M   | 1           | Indicates the value of confidence.                                      |

#### C.1.4.4.12 Type: UncertaintyEllipse

Table C.1.4.4.12-1: Definition of type UncertaintyEllipse

| Attribute name   | Data type   | P   | Cardinality | Description                                               |
|------------------|-------------|-----|-------------|-----------------------------------------------------------|
| semiMajor        | Uncertainty | M   | 1           | Indicates the semi-major axis of the uncertainty ellipse. |
| semiMinor        | Uncertainty | M   | 1           | Indicates the semi-minor axis of the uncertainty ellipse. |
| orientationMajor | Orientation | M   | 1           | Indicates the orientation angle of the major axis.        |

#### C.1.4.4.13 Type: SatCov

Table C.1.4.4.13-1: Definition of type SatCov

| Attribute name | Data type                         | P   | Cardinality | Description                                                                                                                                                    | Applicability |
|----------------|-----------------------------------|-----|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|
| satelliteId    | Uinteger                          | M   | 1           | Indicates the satellite ID of the satellite coverage information. The value range is from 0 to 255 (see SatelliteId IE in 3GPP TS 36.331 \[33\] clause 6.3.6). |               |
| geographicArea | GeographicArea                    | O   | 0..1        | Indicates the geographic location information for the satellite coverage                                                                                       |               |
| timeWindow     | array(ScheduledCommunicationTime) | O   | 0..N        | Indicates the time window for the satellite coverage                                                                                                           |               |
| ratType        | RatType                           | O   | 0..1        | Indicates the satellite access type.                                                                                                                           |               |

### C.1.4.5 Common enumerations


#### C.1.4.5.1 Enumeration: SupportedGADShapes

Table C.1.4.5.1-1: Enumeration SupportedGADShapes

| Enumeration value                    | Description                                             |
|--------------------------------------|---------------------------------------------------------|
| POINT                                | Ellipsoid Point                                         |
| POINT_UNCERTAINTY_CIRCLE             | Ellipsoid point with uncertainty circle                 |
| POINT_UNCERTAINTY_ELLIPSE            | Ellipsoid point with uncertainty ellipse                |
| POLYGON                              | Polygon                                                 |
| POINT_ALTITUDE                       | Ellipsoid point with altitude                           |
| POINT_ALTITUDE_UNCERTAINTY           | Ellipsoid point with altitude and uncertainty ellipsoid |
| ELLIPSOID_ARC                        | Ellipsoid Arc                                           |
| LOCAL_2D_POINT_UNCERTAINTY_ELLIPSE   | Local 2D point with uncertainty ellipse                 |
| LOCAL_3D_POINT_UNCERTAINTY_ELLIPSOID | Local 3D point with uncertainty ellipsoid               |

#### C.1.4.5.2 Enumeration: RatType

Table C.1.4.5.2-1: RatType

| Enumeration value | Description                          | Applicability |
|-------------------|--------------------------------------|---------------|
| NR_LEO            | NR (LEO) satellite access type.      |               |
| NR_MEO            | NR (MEO) satellite access type.      |               |
| NR_GEO            | NR (GEO) satellite access type.      |               |
| NR_OTHER_SAT      | NR (OTHERSAT) satellite access type. |               |
