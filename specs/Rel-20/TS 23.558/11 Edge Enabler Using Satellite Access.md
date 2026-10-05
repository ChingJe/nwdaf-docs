---
spec: TS 23.558
version: 20.3.0
release: '20'
clause: 11
title: 11 Edge Enabler Using Satellite Access
source_archive: 23558-k30.zip
source_document: 23558-k30.docx
content_origin: 3gpp-source
---

# 11 Edge Enabler Using Satellite Access


## 11.1 General

This clause provides clarifications on the procedures and information flows necessary for enabling edge application using satellite access.

### 11.1.1 EES determination based on UE serving satellite ID Procedure

This procedure is applicable for EDN deployment with gNB on satellite.

A satellite EDN may be deployed as described in Annex A.2.5. The service provisioning procedure for satellite EDN is the same as clause 8.3.3.2.2 and clause 8.3.3.2.3 with the following clarifications.

When UE connects to the satellite, the service provisioning request message contains the UE serving Satellite ID if available (i.e., serving gNB corresponding Satellite ID, if received). If the service provisioning request message contains the UE serving Satellite ID, then the ECS determines the EES where the Satellite ID in the EES profile matches the UE serving Satellite ID.

When UE connects to the satellite, and the service provisioning request message does not contain the UE serving Satellite ID, then service provisioning response message may contain the list of EES along with the EES Satellite ID, and then the EEC determine EES where the Satellite ID in the EES profile matches the UE serving Satellite ID, if available in the UE.

### 11.1.2 EES determination based on UE location and route Procedure

This procedure and the corresponding EES profile and Service provisioning request are applicable for EDN deployment with gNB on satellite.

A satellite EDN may be deployed as described in Annex A.2.5. The service provisioning procedure for satellite EDN is the same as clause 8.3.3.2.2 and clause 8.3.3.2.3 with the following clarifications:

\- When EEC sends the service request message to the ECS and the request message contains the UE location, predicted/expected UE location, Prediction expiration time, minimum availability time duration for satellite access by the EEC (UE), the ECS determines a list of EES where the Application Satellite coverage availability information (ASCAI) in EES profile matches the UE location, predicted/expected UE location, Prediction expiration time, minimum availability time duration for satellite access by the EEC (UE) and EES ASCAI. Alternatively, the ECS may query an external server with the UE current location and UE predicted/expected location to obtain a list of Satellite IDs and Trajectory IDs corresponding to the UE location and the UE trajectory, then the ECS determines a list of EES based on the Satellite IDs and Trajectory IDs in EES profiles by matching with the Satellite IDs and Trajectory IDs obtained from an external server for the UE. The “Expected AC Geographical Service Area” in the AC profile, as specified in clause 8.2.2, is used by the ECS to request the external server (managed by the satellite service provider or operator) for such satellite IDs and trajectory IDs.

\- When ECS sends the service provisioning response which contains a list of EES information which EES is deployed on MEO or LEO satellites, the EEC determines the EES where the Satellite ID corresponding to the EES in the EDN configuration information matches the UE serving Satellite ID. If the UE serving satellite ID is not available, the EEC may determine the EES based on available information e.g., UE current location, corresponding current time, received ASCAI and how long UE can stay under coverage, etc.

\- In addition, when a serving satellite is providing EES and EAS, and due to discontinuous coverage, the same satellite comes back for serving, then the EEC connects to the same EES and EAS (i.e. to which the EEC had previously connected to) based on available information e.g., UE current location, corresponding current time, received ASCAI and how long UE can stay under coverage, etc, without needing to discover EES and EAS endpoints again.

This procedure is also utilized by the ACR procedures specified in clause 8.8 where EES determination is needed during service provisioning.

#### 11.1.2.1 EES profile

The EES profile is same as clause 8.2.6 with the following additions.

Table 11.1.2.1-1: EES Profile

<table>
<colgroup>
<col style="width: 24%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Information element</th>
<th>Status</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td colspan="3">IEs from Table 8.2.6-1</td>
</tr>
<tr class="even">
<td>Dynamic service area indication</td>
<td>O</td>
<td>Indicates if the service area is mobile or not. The service area is dynamic in case of EES on board a MEO or LEO satellite and not dynamic in case of a GEO satellite.</td>
</tr>
<tr class="odd">
<td>Satellite ID</td>
<td>M</td>
<td>The satellite ID which identifies a satellite EDN.</td>
</tr>
<tr class="even">
<td><p>Trajectory ID</p>
<p>(NOTE)</p></td>
<td>O</td>
<td>For mobile EES, it is linked with the trajectory of the satellite and therefore identifies the trajectory of the EES on board a MEO and LEO satellites.</td>
</tr>
<tr class="odd">
<td>Application Satellite coverage availability information (ASCAI)</td>
<td>O</td>
<td>The (ASCAI) across the Topological/Geographical service area, of the satellite where the EES is onboard, as described in clause 21.2 of 3GPP TS 23.434 [13]</td>
</tr>
<tr class="even">
<td colspan="3">NOTE: The assignment of the trajectory ID for the EES and linking it to an actual trajectory of the satellite can be done by the operator and/or the satellite service provider.</td>
</tr>
</tbody>
</table>

#### 11.1.2.2 Service provisioning request

Table 11.1.2.2-1 is same as clause 8.3.3.3.2 Table 8.3.3.3.2-1 with the following additions.

Table 11.1.2.2-1: Service provisioning request

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 16%" />
<col style="width: 50%" />
</colgroup>
<tbody>
<tr class="odd">
<td>Information element</td>
<td>Status</td>
<td>Description</td>
</tr>
<tr class="even">
<td colspan="3">IEs from Table 8.3.3.3.2-1</td>
</tr>
<tr class="odd">
<td>UE serving Satellite ID<br />
(NOTE 1)</td>
<td>O</td>
<td>The identifier of the satellite which is serving UE in current location.</td>
</tr>
<tr class="even">
<td><p>UE location</p>
<p>(NOTE 2)</p></td>
<td>O</td>
<td>The location information of the UE. The UE location is described in clause 7.3.2, or Present UE location (e.g. latitude and longitude) for a reference grid point.</td>
</tr>
<tr class="odd">
<td>Minimum availability time duration for satellite access</td>
<td>O</td>
<td><p>This is the minimum time duration for which the availability of the Satellite access is required by the EEC (UE).</p>
<p>The value for this IE is determined by the EEC based on the applications requirement.</p></td>
</tr>
<tr class="even">
<td colspan="3"><p>NOTE 1: The UE serving Satellite ID are available to EEC if the UE has received it from satellite access.</p>
<p>NOTE 2: This IE is not an additional IE but update an existing IE in Table 8.3.3.3.2-1.</p></td>
</tr>
</tbody>
</table>

#### 11.1.2.3 Service provisioning response

Table 11.1.2.3-1 is same as clause 8.3.3.3.3 Table 8.3.3.3.3-2 with the following additions.

Table 11.1.2.3-1: EDN configuration information

|                                |        |                                                        |
|--------------------------------|--------|--------------------------------------------------------|
| Information element            | Status | Description                                            |
| **IEs from Table 8.3.3.3.2-1** |        |                                                        |
| List of EESs                   | M      | List of EESs of the EDN.                               |
| \> EESID                       | M      | The identifier of the EES                              |
| \> EES Endpoint                | M      | The endpoint address (e.g. URI, IP address) of the EES |
| **New IEs**                    |        |                                                        |
| \> Satellite ID                | O      | The satellite ID which identifies a satellite EDN.     |
| **IEs from Table 8.3.3.3.2-1** |        |                                                        |

### 11.1.3 Application Server Relocation during satellite access


#### 11.1.3.1 General

Serving satellite mobility is another possible trigger for ACR other than the trigger scenarios described in clause 8.8.1.1A. The EEC, EAS, and EES detect the need for ACR due to the serving satellite mobility.

Editor’s Note: The exact scenario which support the above assumption will be addressed latter.

NOTE 1: The support of Application Server Relocation and potentical EES relocation does not requires the network layer connection continuity.The EEC may detect the need, or potential need, for ACR based on ASCAI information such as geographical service area and availability time period available in the EES profile received in service provisioning to predict the coverage end of S-EES.

The EAS may detect the need, or potential need, for ACR by subscribing to the service continuity related events (eg., "out of service area" event as described in clause 8.6.3.2.2) and receive corresponding notifications (eg., "out of service area" event as described in clause 8.6.3.2.3).

The EES may detect the need, or potential need, for ACR through the ASCAI information (such as geographical service area and availability time period) available in the EES profile.

The serving satellite mobility triggered ACR event needs additional clarifications in the clauses 4.6, 8.6.3, 8.8.1 and 8.8.3 and 8.8.3 as described below.

#### 11.1.3.2 Updates to General clauses 4.6, 8.6.3 and 8.8.1

Clause 4.6 is clarified to consider scenario when a serving satellite moves outside the expected/predicted location of UE, different EASs can be more suitable for serving the UE.

"Out of service area" in clause 8.6.3 is clarified that this event supports to detect whether a UE is out of the service area of the subscribing EAS (which is either deployed on ground or onboard satellite). Clause 8.6.3.2.2 step 1b is clarified to - The EAS may include the "out of service area" event to indicate the EES to notify the EAS when the EES detects that a UE is out of the subscribed EAS service area (which is either deployed on ground or onboard satellite). Clause 8.6.3.2.3 step 2 is clarified to - The EES includes the ACR management event notification information of the UE(s), and optionally the timestamp. If the event triggering the notification is user plane path change or serving area of interest change due to UE mobility or serving satellite mobility, the timestamp can be included to indicate the age of the user plane path management or mobility event notification information.

The serving satellite mobility triggered ACR event needs the following updates to clause 8.8.1.1.

When a UE loses coverage due to serving satellite mobility, different EASs can be more suitable for serving the ACs in the UE.

UE coverage by the serving satellite may be lost when:

\- UE is stationary and satellite is orbiting;

\- UE is mobile and satellite is orbiting in the same direction of UE; and

\- UE is mobile and satellite is orbiting in the opposite direction of UE.

Following intra-EDN, inter-EDN, between EDN and Cloud, and LADN (overlapping LADN service areas) related scenarios supported for service continuity additionally includes mobility of serving satellite.

A detection entity detects the need, or potential need, for ACR by monitoring various aspects including predicted serving satellite mobility based on ASCAI information available in EES profile.

ACR can be performed for service continuity planning, also when the serving satellite is predicted to move outside the the expected/predicted UE location. In such a case the T-EAS is expected to provide service to the UE either when it moves to the expected location or when the serving satellite is expected to move outside the expected/predicted UE location.

Clause 8.8.1.1A is clarified that apart from UE mobility serving satellite mobility is also a trigger for ACR. Implementation of ACR with service continuity planning as described in clause 8.8.1.2 also utilizes ASCAI information available in EES profile.
