---
spec: TR 23.700-36
version: 18.1.0
release: '18'
clause: 8
title: 8 Overall evaluation
source_archive: 23700-36-i10.zip
source_document: 23700-36-i10.doc
content_origin: 3gpp-source
---

# 8 Overall evaluation


## 8.1 General

The following clauses contain an overall evaluation of the solutions presented in this technical report, their applicability to the identified key issues and possible dependencies to other groups. This clause also includes a summary of the inputs/outputs and corresponding analytics APIs to be considered for the normative phase.

## 8.2 Solution evaluations


### 8.2.1 General

All the key issues and solutions specified in this technical report are listed in Table 8.2.1-1. This table includes the mapping of the key issues to the solutions and corresponding solution evaluations.

Table 8.2.1-1 Key issue and solutions

|                                                                           |                                                                               |                                    |
|---------------------------------------------------------------------------|-------------------------------------------------------------------------------|------------------------------------|
| Key issues                                                                | Solution                                                                      | Dependency on other working groups |
| Key issue #1: Support for application performance analytics               | Solution \#1: Support for application performance analytics                   |                                    |
|                                                                           | Solution \#2: Data Analytics Enablement                                       |                                    |
|                                                                           | Solution \#4: Support for performance analytics for UE-to-UE sessions         |                                    |
|                                                                           | Solution \#5: Service experience to support application performance analytics |                                    |
| Key issue #2: Support for edge analytics enablement                       | Solution \#2: Data Analytics Enablement                                       |                                    |
|                                                                           | Solution \#3: Support for edge load analytics                                 |                                    |
| Key issue #3: Support for data collection for application layer analytics | Solution \#2: Data Analytics Enablement                                       |                                    |
| Key issue #4: Key Issue on interactions with SEAL services                | Solution \#2: Data Analytics Enablement                                       |                                    |
|                                                                           | Solution \#6: Support for slice related application data analytics            |                                    |
|                                                                           | Solution \#7: Slice configuration recommendation                              |                                    |
|                                                                           | Solution \#8: Location accuracy analytics                                     |                                    |
| Key issue #5: Support for slice-related application data analytics        | Solution \#6: Support for slice related application data analytics            |                                    |
| Key issue #6: Support for slice configuration recommendation enablement   | Solution \#7: Slice configuration recommendation                              |                                    |
| Key issue #7: support for location accuracy analytics                     | Solution \#8: Location accuracy analytics                                     |                                    |
| Key issue #8: Support for service API capability analytics                | Solution \#9 : Service API analytics                                          |                                    |

More specifically,

\- For Key issue #1 (Support for application performance analytics), Solution \#1 and Solution \#4 provide analytics capabilities for application performance targeting the UE-to-network and UE-to-UE sessions respectively. Solution \#2 provides a generic mechanism which can be used for data analytics enablement and can be adopted by Solutions \#1 and \#4; whereas Solution \#5 provides methods for collecting/notifying service experience data from the UE and can be seen as further elaboration of the collection/notification means based on Solution \#1. There is no identified conflict among the solutions, and the solutions can complement each other.

\- For Key issue #2 (Support for edge analytics enablement), Solution \#3 provides analytics functionality for edge load, whereas Solution \#2 provides a generic mechanism which can be used for edge load analytics enablement and can be adopted by Solution \#3 (whereas it is not mandatory). Solution \#2 may require enhancements in eEDGEAPP to allow the collection of the EES load information.

\- For Key issue #3 (Support for data collection for application layer analytics), Solution \#2 provides a generic mechanism for data collection enablement to be used for ADAE analytics derivation.

\- For Key issue #4 (Key Issue on interactions with SEAL services), solutions \#2, \#6, \#7, \#8 have interactions with other SEAL services. Solution \#2 provides a generic mechanism for data collection enablement to be used for ADAE analytics derivation. Additionally, Solution \#6 and \#7 interact with SEAL NSCE server for analytics derivation. Finally Solution \#8 collects location reports from SEAL LMS and may require location enhancements of SEAL LMS. There is no identified conflict among the solutions, and the solutions can complement each other.

\- For Key issue #5 (Support for slice-related application data analytics), Solution \#6 provides slice-related application analytics and is optionally interacting with SEAL NSCE service (no enhancements are needed at SEAL NSCE).

\- For Key issue #6 (Support for slice configuration recommendation enablement), Solution \#7 provides a support capability for slice configuration recommendation and is interacting with SEAL NSCE service (no enhancements are needed at SEAL NSCE).

\- For Key issue #7 (support for location accuracy), Solution \#8 provides location accuracy analytics functionality and is optionally interacting with SEAL LMS or FLF service (enhancements are needed at SEAL LMS if used as data source).

\- For Key issue #8 (Support for service API capability analytics), Solution \#9 provides a support capability for service API analytics.

### 8.2.2 ADAE analytics services

Table 8.2.2-1 provides an overview of the analytics services which are provided by ADAE layer, based on the individual solutions.

Table 8.2.2-1 ADAE analytics services

<table>
<colgroup>
<col style="width: 19%" />
<col style="width: 8%" />
<col style="width: 19%" />
<col style="width: 16%" />
<col style="width: 19%" />
<col style="width: 15%" />
</colgroup>
<tbody>
<tr class="odd">
<td>Analytics Event</td>
<td>Solution</td>
<td>Inputs</td>
<td>Data Collection Sources</td>
<td>Analytics Outputs</td>
<td>Type of analytics</td>
</tr>
<tr class="even">
<td>VAL server performance analytics</td>
<td>Sol #1, Sol #5</td>
<td>per VAL server performance measurements, historical data/stats for VAL server performance, network / KPI monitoring from 5GS</td>
<td><p>1. VAL UE</p>
<p>2. VAL Server</p>
<p>3. OAM</p>
<p>4. 5GC (NWDAF, NEF)</p></td>
<td>Analytics on application QoS metrics per VAL server</td>
<td>Prediction, statistics</td>
</tr>
<tr class="odd">
<td>VAL UE/session performance analytics</td>
<td>Sol #1, Sol #5</td>
<td>per VAL session performance measurements</td>
<td><p>1. VAL UE</p>
<p>2. VAL Server</p></td>
<td>Analytics on application QoS metrics per VAL session</td>
<td>Prediction</td>
</tr>
<tr class="even">
<td>VAL UE-to-UE session performance analytics</td>
<td>Sol #4</td>
<td>per UE-to-UE session performance measurements</td>
<td>1. VAL UEs</td>
<td>Analytics on application QoS metric change for UE-to-UE session</td>
<td>Prediction</td>
</tr>
<tr class="odd">
<td>edge load analytics</td>
<td>Sol #3</td>
<td>edge platform load data, EAS/EES load data, DN performance analytics, UPF load analytics</td>
<td><p>1. OAM / MDAS</p>
<p>2. 5GC / NWDAF</p>
<p>3. SEALDD server</p>
<p>4. EES</p>
<p>5. EAS</p>
<p>5. MEP / RNIS</p></td>
<td><p>1. stats / predictions on the EDN load conditions,</p>
<p>2. EES or EAS load stats/predictions, 3. recommendation for pro-active EAS relocation trigger</p></td>
<td>Prediction, statistics</td>
</tr>
<tr class="even">
<td>Slice related performance analytics</td>
<td>Sol #6</td>
<td>per slice measurements and analytics, application session performance analytics, historical data on slice information</td>
<td><p>1. OAM or NSCE</p>
<p>2. 5GC / NWDAF</p>
<p>3. VAL UEs</p></td>
<td>Statistics / prediction for the VAL application QoS for one or more requested S-NSSAIs/NSIs</td>
<td>Prediction, statistics</td>
</tr>
<tr class="odd">
<td>Location accuracy analytics</td>
<td>Sol #8</td>
<td><p>UE mobility analytics,</p>
<p>UE location reports and achieved accuracy,</p>
<p>historical location accuracy statistics for target VAL service area or VAL UE</p></td>
<td><p>1. SEAL LMS / FLS</p>
<p>2. 5GC / NWDAF</p>
<p>3. A-ADRF</p></td>
<td>a predictive location accuracy sustainability or change indication</td>
<td>Prediction</td>
</tr>
<tr class="even">
<td>Service API analytics</td>
<td>Sol #9</td>
<td>Service API logs for requested APIs, historical data / statistics on service API availability and service level</td>
<td><p>1. CCF</p>
<p>2. A-ADRF</p></td>
<td>stats / predictions for service API(s)</td>
<td>Prediction, statistics</td>
</tr>
<tr class="odd">
<td>Slice configuration recommendation</td>
<td>Sol #7</td>
<td>per slice measurements and analytics, historical data on slice information</td>
<td><p>1. SEAL NSCE</p>
<p>2. OAM</p>
<p>3. NWDAF</p>
<p>4. A-ADRF</p></td>
<td>Statistics for the network slice configuration recommendation for one or more requested S-NSSAIs</td>
<td>Statistics</td>
</tr>
</tbody>
</table>
