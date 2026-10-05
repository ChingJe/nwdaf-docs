---
spec: TS 24.546
version: 19.5.0
release: '19'
clause: Annex A
title: 'Annex A (normative): Parameters for different operations'
source_archive: 24546-j50.zip
source_document: 24546-j50.docx
content_origin: 3gpp-source
---

# Annex A (normative): Parameters for different operations


## A.1 Creating configuration update event subscription


### A.1.1 General

The information in this annex provides a normative description of the parameters which will be sent by SCM-C while creating configuration update event subscription and the parameters which will be sent by SCM-S as a response to request for creating subscription.

### A.1.2 Client side parameters

The SCM-C shall convey the following parameters while sending request for creating configuration update event subscription.

Table A.1.2-1: Client side parameters for creating configuration update event subscription

| Parameter         | Description                                                                                                     |
|-------------------|-----------------------------------------------------------------------------------------------------------------|
| Callback-URI      | REQUIRED. Represents where to send HTTP notifications                                                           |
| Subscription Info | REQUIRED. Represents a space-separated list of the subscription type information as specified in table A.1.2-2. |

Table A.1.2-2: Subscription information

<table>
<colgroup>
<col style="width: 18%" />
<col style="width: 81%" />
</colgroup>
<thead>
<tr class="header">
<th>Parameter</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Event</td>
<td><p>REQUIRED. Represents the type of notification which client requires. This specification defines following type of notifications:</p>
<p>- 0x01: SUBSCRIBE_USER_PROFILE_MODIFICATION</p>
<p>- 0x02: SUBSCRIBE_UE_CONFIG_MODIFICATION</p></td>
</tr>
<tr class="even">
<td>expiry time</td>
<td>REQUIRED. Represents the time in seconds up to which the subscription is desired to be kept active and the time after which the subscribed event shall stop generating notifications.</td>
</tr>
</tbody>
</table>

### A.1.3 Server side parameters

The SCM-S shall convey the following parameters while sending response to the creating configuration update event subscription request.

Table A.1.3-1: Server side parameters for response to creating configuration update event subscription

| Parameter | Description                                                   |
|-----------|---------------------------------------------------------------|
| Identity  | REQUIRED. A unique string representing subscription identity. |

## A.2 Retrieve VAL UE configuration data


### A.2.1 Client side parameters

The SGM-C shall convey the following parameters, if available, while sending request to retrieve a VAL UE configuration data.

Table A.1.2-1: Client side parameters to retrieve VAL UE configuration data

| Parameter          | Description                                                                                                                                |
|--------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| VAL UE Information | OPTIONAL. Represents additional UE related information required to identify the configuration data (e.g. device type, device vendor, etc). |
