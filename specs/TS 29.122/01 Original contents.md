---
spec: TS 29.122
version: 18.10.0
release: '18'
clause: contents
title: Contents
source_archive: 29122-ia0.zip
source_document: 29122-ia0.doc
source_archive_sha256: bea81e48dfe77bc42deb87684c82fd69b8ac18a0eb1c9f9632f61f094ba95ecf
source_document_sha256: 6bd67a5c13b682bafa1d4b5bb9e860f8d1635bd8829e8a2fc200656ba7a61635
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Contents

Foreword 15

1 Scope 16

2 References 16

3 Definitions and abbreviations 18

3.1 Definitions 18

3.2 Abbreviations 19

4 T8 reference point 19

4.1 Overview 19

4.2 Reference model 20

4.3 Functional elements 21

4.3.1 SCEF 21

4.3.2 SCS/AS 21

4.4 Procedures over T8 reference point 21

4.4.1 Introduction 21

4.4.2 Monitoring Procedures 21

4.4.2.1 General 21

4.4.2.2 Monitoring Events Configuration 21

4.4.2.2.1 General 21

4.4.2.2.2 Monitoring Events Configuration via HSS 23

4.4.2.2.3 Monitoring Events Configuration directly via MME/SGSN 25

4.4.2.2.4 Monitoring Events Configuration via PCRF 26

4.4.2.3 Reporting of Monitoring Event Procedure 27

4.4.2.4 Network-initiated Explicit Monitoring Event Deletion Procedure 27

4.4.2.5 Network initiated notification of applied parameter configuration 28

4.4.3 Procedures for resource management of Background Data Transfer 28

4.4.4 Procedures for changing the chargeable party at session set up or during the session 29

4.4.5 Procedures for Non-IP Data Delivery 30

4.4.5.1 General 30

4.4.5.2 NIDD Configuration 30

4.4.5.2.1 NIDD Configuration for a single UE 30

4.4.5.2.2 NIDD Configuration for a group of UEs 31

4.4.5.3 Mobile Terminated NIDD procedure 31

4.4.5.3.1 Mobile Terminated NIDD for a single UE 31

4.4.5.3.2 Mobile Terminated NIDD for a group of UEs 33

4.4.5.4 Mobile Originated NIDD procedure 34

4.4.5.5 NIDD Authorisation Update procedure 34

4.4.5.6 Port Management Configuration 35

4.4.5.6.1 Port Reservation and Release 35

4.4.5.6.2 Port Notification 36

4.4.6 Procedures for Device Triggering 36

4.4.7 Procedures for Group Message Delivery 37

4.4.7.1 General 37

4.4.7.2 Group Message Delivery via MBMS 37

4.4.7.2.1 General 37

4.4.7.2.2 Group Message Delivery via MBMS by MB2 37

4.4.7.2.3 Group message Delivery via MBMS by xMB 40

4.4.8 Procedures for Reporting of Network Status 41

4.4.8.1 General 41

4.4.8.2 Network Status Reporting Subscription 41

4.4.8.3 Network Status Reporting Notification 42

4.4.9 Procedures for Communication Pattern Parameters Provisioning 42

4.4.10 Procedures for PFD Management 43

4.4.11 Procedures for Enhanced Coverage Restriction Control 45

4.4.12 Procedures for Network Parameter Configuration 46

4.4.12.1 General 46

4.4.12.2 Configuration Request for an individual UE 46

4.4.12.3 Configuration Request for a group of UEs 47

4.4.12.4 Notification of applied parameter configuration 47

4.4.13 Procedures for setting up an AS session with required QoS 47

4.4.14 Procedures for MSISDN-less Mobile Originated SMS 49

4.4.14.1 General 49

4.4.14.2 Delivery of MSISDN-less MO SMS 49

4.4.15 Procedures for RACS Parameter Provisioning 49

5 T8 APIs 50

5.1 Introduction 50

5.2 Information applicable to several APIs 51

5.2.1 Data Types 51

5.2.1.1 Introduction 51

5.2.1.2 Referenced structured data types 54

5.2.1.2.1 Type: SponsorInformation 54

5.2.1.2.2 Type: UsageThreshold 54

5.2.1.2.3 Type: TimeWindow 54

5.2.1.2.4 Type: Acknowledgement 55

5.2.1.2.5 Type: NotificationData 55

5.2.1.2.6 Type: EventReport 55

5.2.1.2.7 Type: AccumulatedUsage 55

5.2.1.2.8 Type: FlowInfo 55

5.2.1.2.9 Type: TestNotification 56

5.2.1.2.10 Type: WebsockNotifConfig 56

5.2.1.2.11 Type: LocationArea 56

5.2.1.2.12 Type: ProblemDetails 57

5.2.1.2.13 Type: InvalidParam 57

5.2.1.2.14 Type: PlmnId 58

5.2.1.2.15 Type: ConfigResult 58

5.2.1.2.16 Type: UsageThresholdRm 58

5.2.1.2.17 Type: LocationArea5G 58

5.2.1.2.18 Type: EthFlowInfo 59

5.2.1.3 Referenced Simple data types and enumerations 59

5.2.1.3.1 Introduction 59

5.2.1.3.2 Simple data types 59

5.2.1.3.3 Enumeration: Event 60

5.2.1.3.4 Enumeration: ResultReason 61

5.2.1.4 Conventions for documenting structured data types 61

5.2.2 Usage of HTTP 62

5.2.2.1 General 62

5.2.2.2 Usage of the HTTP PATCH method 62

5.2.3 Content type 62

5.2.4 URI structure 63

5.2.4.1 Resource URI structure 63

5.2.4.2 Custom operations URI structure 63

5.2.4.3 Callback URI structure 63

5.2.5 Notifications 64

5.2.5.1 General 64

5.2.5.2 Notification Delivery using a separate HTTP connection 64

5.2.5.3 Notification Test Event 64

5.2.5.4 Notification Delivery using Websocket 64

5.2.6 Error handling 66

5.2.7 Feature negotiation 68

5.2.8 HTTP custom headers 68

5.2.8.1 General 68

5.2.8.2 Reused HTTP custom headers 68

5.2.8.3.1 General 68

5.2.9 Conventions for Open API specification files 70

5.2.9.1 General 70

5.2.9.2 Formatting of OpenAPI files 70

5.2.9.3 Structured data types 70

5.2.9.4 Info 72

5.2.9.5 Servers 72

5.2.9.6 References to other 3GPP-defined Open API specification files 72

5.2.9.7 Server-initiated communication 73

5.2.9.8 Describing the body of HTTP PATCH requests 73

5.2.9.8.1 General 73

5.2.9.8.2 JSON Merge Patch 73

5.2.9.8.3 JSON PATCH 74

5.2.9.9 Error Responses 74

5.2.9.10 Enumerations 75

5.2.9.11 Read only attribute 76

5.2.9.12 externalDocs 76

5.2.9.13 Operation identifiers 76

5.2.9.14 Usage of the "tags" field 76

5.2.10 Redirection handling 77

5.2.11 Support of Load and Overload Control 77

5.2.12 Query parameters 78

5.2.13 Vendor-specific extensions 78

5.2.13.1 General 78

5.2.13.2 Vendor-specific extensions to the data model 78

5.2.13.3 Vendor-specific query parameters 79

5.3 MonitoringEvent API 80

5.3.1 Overview 80

5.3.2 Data model 80

5.3.2.1 Resource data types 80

5.3.2.1.1 Introduction 80

5.3.2.1.2 Type: MonitoringEventSubscription 83

5.3.2.1.3 Void 95

5.3.2.2 Notification data types 95

5.3.2.2.1 Introduction 95

5.3.2.2.2 Type: MonitoringNotification 95

5.3.2.3 Referenced structured data types 96

5.3.2.3.1 Introduction 96

5.3.2.3.2 Type: MonitoringEventReport 96

5.3.2.3.3 Type: IdleStatusInfo 100

5.3.2.3.4 Type: UePerLocationReport 100

5.3.2.3.5 Type: LocationInfo 101

5.3.2.3.6 Type: FailureCause 103

5.3.2.3.7 Type: PdnConnectionInformation 103

5.3.2.3.8 Type: AppliedParameterConfiguration 104

5.3.2.3.9 Type: ApiCapabilityInfo 104

5.3.2.3.10 Type: MonitoringEventReports 105

5.3.2.3.11 Type: UavPolicy 105

5.3.2.3.11 Type: ConsentRevocNotif 105

5.3.2.3.12 Type: ConsentRevoked 106

5.3.2.3.13 Type: GroupMembListChanges 106

5.3.2.3.14 Void 106

5.3.2.3.15 Void 106

5.3.2.3.16 Void 106

5.3.2.3.17 Type: UpLocRepAddrAfRm 106

5.3.2.3.18 Type: UpCumEvtRep 107

5.3.2.4 Referenced simple data types and enumerations 107

5.3.2.4.1 Introduction 107

5.3.2.4.2 Simple data types 107

5.3.2.4.3 Enumeration: MonitoringType 107

5.3.2.4.4 Enumeration: ReachabilityType 108

5.3.2.4.5 Enumeration: LocationType 109

5.3.2.4.6 Enumeration: AssociationType 109

5.3.2.4.7 Enumeration: Accuracy 109

5.3.2.4.8 Enumeration: PdnConnectionStatus 110

5.3.2.4.9 Enumeration: PdnType 110

5.3.2.4.10 Enumeration: InterfaceIndication 110

5.3.2.4.11 Enumeration: LocationFailureCause 111

5.3.2.4.12 Enumeration: SubType 111

5.3.2.4.13 Enumeration: SACRepFormat 111

5.3.3 Resource structure 111

5.3.3.1 General 111

5.3.3.2 Resource: Monitoring Event Subscriptions 112

5.3.3.2.1 Introduction 112

5.3.3.2.2 Resource definition 112

5.3.3.2.3 Resource methods 112

5.3.3.3 Resource: Individual Monitoring Event Subscription 115

5.3.3.3.1 Introduction 115

5.3.3.3.2 Resource definition 115

5.3.3.3.3 Resource methods 115

5.3.3.4 Void 119

5.3.3A Notifications 119

5.3.3A.1 General 119

5.3.3A.2 Monitoring Notification 120

5.3.3A.2.1 Description 120

5.3.3A.2.2 Target URI 120

5.3.3A.2.3 Standard Methods 120

5.3.3A.3 User Consent Revocation Notification 121

5.3.3A.3.1 Description 121

5.3.3A.3.2 Target URI 121

5.3.3A.3.3 Operation Definition 122

5.3.4 Used Features 122

5.3.5 Error handling 125

5.3.5.1 General 125

5.3.5.2 Protocol Errors 125

5.3.5.3 Application Errors 125

5.4 ResourceManagementOfBdt API 127

5.4.1 Overview 127

5.4.2 Data model 127

5.4.2.1 Resource data types 127

5.4.2.1.1 Introduction 127

5.4.2.1.2 Type: Bdt 127

5.4.2.1.3 Type: BdtPatch 128

5.4.2.1.4 Type: ExNotification 129

5.4.2.2 Referenced structured data types 129

5.4.2.2.1 Introduction 129

5.4.2.2.2 Type: TransferPolicy 129

5.4.2.3 Referenced simple data types and enumerations 130

5.4.2.3.1 Introduction 130

5.4.2.3.2 Simple data types 130

5.4.3 Resource structure 130

5.4.3.1 General 130

5.4.3.2 Resource: BDT Subscriptions 131

5.4.3.2.1 Introduction 131

5.4.3.2.2 Resource definition 131

5.4.3.2.3 Resource methods 131

5.4.3.3 Resource: Individual BDT Subscription 133

5.4.3.3.1 Introduction 133

5.4.3.3.2 Resource definition 133

5.4.3.3.3 Resource methods 133

5.4.3.4 Void 137

5.4.3A Notifications 137

5.4.3A.1 General 137

5.4.3A.2 BDT Warning Notification 138

5.4.3A.2.1 Description 138

5.4.3A.2.2 Target URI 138

5.4.3A.2.3 Standard Methods 138

5.4.4 Used Features 139

5.4.5 Error handling 139

5.4.5.1 General 139

5.4.5.2 Protocol Errors 139

5.4.5.3 Application Errors 139

5.5 ChargeableParty API 140

5.5.1 Overview 140

5.5.2 Data model 140

5.5.2.1 Resource data types 140

5.5.2.1.1 Introduction 140

5.5.2.1.2 Type: ChargeableParty 140

5.5.2.1.3 Type: ChargeablePartyPatch 142

5.5.3 Resource structure 142

5.5.3.1 General 142

5.5.3.2 Resource: Chargeable Party Transactions 143

5.5.3.2.1 Introduction 143

5.5.3.2.2 Resource definition 143

5.5.3.2.3 Resource methods 143

5.5.3.3 Resource: Individual Chargeable Party Transaction 145

5.5.3.3.1 Introduction 145

5.5.3.3.2 Resource definition 145

5.5.3.3.3 Resource methods 146

5.5.3.4 Void 148

5.5.3A Notifications 148

5.5.3A.1 General 148

5.5.3A.2 Event Notification 149

5.5.3A.2.1 Description 149

5.5.3A.2.2 Target URI 149

5.5.3A.2.3 Standard Methods 149

5.5.4 Used Features 150

5.5.5 Error handling 150

5.5.5.1 General 150

5.5.5.2 Protocol Errors 150

5.5.5.3 Application Errors 150

5.6 NIDD API 151

5.6.1 Overview 151

5.6.2 Data model 151

5.6.2.1 Resource data types 151

5.6.2.1.1 Introduction 151

5.6.2.1.2 Type: NiddConfiguration 152

5.6.2.1.3 Type: NiddDownlinkDataTransfer 154

5.6.2.1.4 Type: NiddUplinkDataNotification 156

5.6.2.1.5 Type: NiddDownlinkDataDeliveryStatusNotification 156

5.6.2.1.6 Type: NiddConfigurationStatusNotification 157

5.6.2.1.7 Type: NiddConfigurationPatch 157

5.6.2.1.8 Type: GmdNiddDownlinkDataDeliveryNotification 158

5.6.2.1.9 Type: ManagePort 158

5.6.2.1.10 Type: ManagePortNotification 159

5.6.2.1.11 Type: NiddDownlinkDataTransferPatch 159

5.6.2.2 Referenced structured data types 160

5.6.2.2.1 Introduction 160

5.6.2.2.2 Type: RdsPort 160

5.6.2.2.3 Type: GmdResult 161

5.6.2.2.4 Type: NiddDownlinkDataDeliveryFailure 161

5.6.2.2.5 Type: RdsDownlinkDataDeliveryFailure 161

5.6.2.3 Referenced simple data types and enumerations 162

5.6.2.3.1 Introduction 162

5.6.2.3.2 Simple data types 162

5.6.2.3.3 Enumeration: PdnEstablishmentOptions 162

5.6.2.3.4 Enumeration: DeliveryStatus 162

5.6.2.3.5 Enumeration: NiddStatus 163

5.6.2.3.6 Enumeration: PdnEstablishmentOptionsRm 163

5.6.2.3.7 Enumeration: ManageEntity 164

5.6.2.3.8 Enumeration: SerializationFormat 164

5.6.3 Resource structure 164

5.6.3.1 General 164

5.6.3.2 Resource: NIDD Configurations 165

5.6.3.2.1 Introduction 165

5.6.3.2.2 Resource definition 166

5.6.3.2.3 Resource methods 166

5.6.3.3 Resource: Individual NIDD Configuration 167

5.6.3.3.1 Introduction 167

5.6.3.3.2 Resource definition 168

5.6.3.3.3 Resource methods 168

5.6.3.4 Resource: NIDD downlink data deliveries 170

5.6.3.4.1 Introduction 170

5.6.3.4.2 Resource definition 171

5.6.3.4.3 Resource methods 171

5.6.3.5 Resource: Individual NIDD downlink data delivery 174

5.6.3.5.1 Introduction 174

5.6.3.5.2 Resource definition 174

5.6.3.5.3 Resource methods 174

5.6.3.6 Void 179

5.6.3.7 Void 179

5.6.3.8 Void 179

5.6.3.9 Resource: Individual ManagePort Configuration 179

5.6.3.9.1 Introduction 179

5.6.3.9.2 Resource definition 179

5.6.3.9.3 Resource methods 179

5.6.3.10 Void 182

5.6.3.11 Resource: ManagePort Configurations 182

5.6.3.11.1 Introduction 182

5.6.3.11.2 Resource definition 183

5.6.3.11.3 Resource methods 183

5.6.3A Notifications 184

5.6.3A.1 General 184

5.6.3A.2 NIDD Configuration Update Notification 184

5.6.3A.2.1 Description 184

5.6.3A.2.2 Target URI 185

5.6.3A.2.3 Standard Methods 185

5.6.3A.3 NIDD Downlink Data Delivery Status Notification 186

5.6.3A.3.1 Description 186

5.6.3A.3.2 Target URI 186

5.6.3A.3.3 Standard Methods 186

5.6.3A.4 NIDD Uplink Data Notification 188

5.6.3A.4.1 Description 188

5.6.3A.4.2 Target URI 188

5.6.3A.4.3 Standard Methods 188

5.6.3A.5 ManagePort Notification 189

5.6.3A.5.1 Description 189

5.6.3A.5.2 Target URI 189

5.6.3A.5.3 Standard Methods 190

5.6.4 Used Features 191

5.6.5 Error handling 191

5.6.5.1 General 191

5.6.5.2 Protocol Errors 191

5.6.5.3 Application Errors 191

5.7 DeviceTriggering API 192

5.7.1 Overview 192

5.7.2 Data model 192

5.7.2.1 Resource data types 192

5.7.2.1.1 Introduction 192

5.7.2.1.2 Type: DeviceTriggering 193

5.7.2.1.3 Type: DeviceTriggeringDeliveryReportNotification 194

5.7.2.1.4 Type: DeviceTriggeringPatch 195

5.7.2.2 Referenced simple data types and enumerations 195

5.7.2.2.1 Introduction 195

5.7.2.2.2 Simple data types 195

5.7.2.2.3 Enumeration: DeliveryResult 195

5.7.2.2.4 Enumeration: Priority 196

5.7.3 Resource structure 196

5.7.3.1 General 196

5.7.3.2 Resource: Device Triggering Transactions 197

5.7.3.2.1 Introduction 197

5.7.3.2.2 Resource definition 197

5.7.3.2.3 Resource methods 197

5.7.3.3 Resource: Individual Device Triggering Transaction 199

5.7.3.3.1 Introduction 199

5.7.3.3.2 Resource definition 199

5.7.3.3.3 Resource methods 199

5.7.3.4 Void 203

5.7.3A Notifications 203

5.7.3A.1 General 203

5.7.3A.2 Device Triggering Delivery Report Notification 204

5.7.3A.2.1 Description 204

5.7.3A.2.2 Target URI 204

5.7.3A.2.3 Standard Methods 204

5.7.4 Used Features 205

5.7.5 Error handling 205

5.7.5.1 General 205

5.7.5.2 Protocol Errors 205

5.7.5.3 Application Errors 205

5.8 GMD via MBMS related APIs 206

5.8.1 Overview 206

5.8.2 GMDviaMBMSbyMB2 API 206

5.8.2.1 Data model 206

5.8.2.1.1 Resource data types 206

5.8.2.2 Resource structure 209

5.8.2.2.1 General 209

5.8.2.2.2 Resource: TMGI Allocation 210

5.8.2.2.3 Resource: Individual TMGI Allocation 212

5.8.2.2.4 Resource: GMD via MBMS by MB2 216

5.8.2.2.5 Resource: Individual GMD via MBMS by MB2 218

5.8.2.2.6 Void 222

5.8.2.2A Notifications 222

5.8.2.2A.1 General 222

5.8.2.2A.2 GMD via MBMS by MB2 Notification 223

5.8.2.3 Used Features 224

5.8.2.4 Error handling 224

5.8.2.4.1 General 224

5.8.2.4.2 Protocol Errors 224

5.8.2.4.3 Application Errors 224

5.8.3 GMDviaMBMSbyxMB API 225

5.8.3.1 Data model 225

5.8.3.1.1 Resource data types 225

5.8.3.1.2 Referenced simple data types and enumerations 228

5.8.3.2 Resource structure 229

5.8.3.2.1 General 229

5.8.3.2.2 Resource: xMB Services 230

5.8.3.2.3 Resource: Individual xMB Service 232

5.8.3.2.4 Resource: GMD via MBMS by xMB 234

5.8.3.2.5 Resource: Individual GMD via MBMS by xMB 236

5.8.3.2.6 Void 240

5.8.3.2A Notifications 240

5.8.3.2A.1 General 240

5.8.3.2A.2 GMD via MBMS by xMB Notification 241

5.8.3.3 Used Features 242

5.8.3.4 Error handling 242

5.8.3.4.1 General 242

5.8.3.4.2 Protocol Errors 242

5.8.3.4.3 Application Errors 242

5.9 ReportingNetworkStatus API 243

5.9.1 Overview 243

5.9.2 Data model 243

5.9.2.1 Resource data types 243

5.9.2.1.1 Introduction 243

5.9.2.1.2 Type: NetworkStatusReportingSubscription 243

5.9.2.1.3 Type: NetStatusRepSubsPatch 244

5.9.2.2 Notification data types 245

5.9.2.2.1 Introduction 245

5.9.2.2.2 Type: NetworkStatusReportingNotification 245

5.9.2.3 Referenced simple data types and enumerations 246

5.9.2.3.1 Introduction 246

5.9.2.3.2 Simple data types 246

5.9.2.3.3 Enumeration: CongestionType 246

5.9.3 Resource structure 246

5.9.3.1 General 246

5.9.3.2 Resource: Network Status Reporting Subscriptions 247

5.9.3.2.1 Introduction 247

5.9.3.2.2 Resource definition 247

5.9.3.2.3 Resource methods 247

5.9.3.3 Resource: Individual Network Status Reporting Subscription 249

5.9.3.3.1 Introduction 249

5.9.3.3.2 Resource definition 249

5.9.3.3.3 Resource methods 249

5.9.3.4 Void 253

5.9.3A Notifications 253

5.9.3A.1 General 253

5.9.3A.2 Network Status Reporting Notification 254

5.9.3A.2.1 Description 254

5.9.3A.2.2 Target URI 254

5.9.3A.2.3 Standard Methods 254

5.9.4 Used Features 255

5.9.5 Error handling 255

5.9.5.1 General 255

5.9.5.2 Protocol Errors 255

5.9.5.3 Application Errors 255

5.10 CpProvisioning API 256

5.10.1 Overview 256

5.10.2 Data model 256

5.10.2.1 Resource data types 256

5.10.2.1.1 Introduction 256

5.10.2.1.2 Type: CpInfo 257

5.10.2.2 Referenced structured data types 259

5.10.2.2.1 Introduction 259

5.10.2.2.2 Type: CpParameterSet 259

5.10.2.2.3 Type: ScheduledCommunicationTime 261

5.10.2.2.4 Type: CpReport 261

5.10.2.2.5 Type: UmtLocationArea5G 261

5.10.2.2.6 Type: AppExpUeBehaviour 263

5.10.2.3 Referenced simple data types and enumerations 264

5.10.2.3.1 Introduction 264

5.10.2.3.2 Simple data types 264

5.10.2.3.3 Enumeration: CommunicationIndicator 264

5.10.2.3.4 Enumeration: StationaryIndication 264

5.10.2.3.5 Enumeration: CpFailureCode 264

5.10.2.3.6 Enumeration: BatteryIndication 265

5.10.2.3.7 Enumeration: TrafficProfile 265

5.10.2.3.8A Enumeration: ScheduledCommunicationType 265

5.10.3 Resource structure 266

5.10.3.1 General 266

5.10.3.2 Resource: CP Provisioning Subscriptions 266

5.10.3.2.1 Introduction 266

5.10.3.2.2 Resource definition 267

5.10.3.2.3 Resource methods 267

5.10.3.3 Resource: Individual CP Provisioning Subscription 269

5.10.3.3.1 Introduction 269

5.10.3.3.2 Resource definition 269

5.10.3.3.3 Resource methods 269

5.10.3.4 Resource: Individual CP Set Provisioning 272

5.10.3.4.1 Introduction 272

5.10.3.4.2 Resource definition 272

5.10.3.4.3 Resource methods 272

5.10.4 Used Features 275

5.10.5 Error handling 275

5.10.5.1 General 275

5.10.5.2 Protocol Errors 275

5.10.5.3 Application Errors 275

5.11 PfdManagement API 276

5.11.1 Overview 276

5.11.2 Data model 276

5.11.2.1 Resource data types 276

5.11.2.1.1 Introduction 276

5.11.2.1.2 Type: PfdManagement 277

5.11.2.1.3 Type: PfdData 278

5.11.2.1.4 Type: Pfd 278

5.11.2.1.5 Type: PfdReport 279

5.11.2.1.6 Type: UserPlaneLocationArea 279

5.11.2.1.7 Type: PfdManagementPatch 280

5.11.2.2 Referenced simple data types and enumerations 280

5.11.2.2.1 Introduction 280

5.11.2.2.2 Simple data types 280

5.11.2.2.3 Enumeration: FailureCode 281

5.11.2.2.4 Enumeration: DomainNameProtocol 281

5.11.3 Resource structure 281

5.11.3.1 General 281

5.11.3.2 Resource: PFD Management Transactions 282

5.11.3.2.1 Introduction 282

5.11.3.2.2 Resource definition 282

5.11.3.2.3 Resource methods 283

5.11.3.3 Resource: Individual PFD Management Transaction 285

5.11.3.3.1 Introduction 285

5.11.3.3.2 Resource definition 285

5.11.3.3.3 Resource methods 285

5.11.3.4 Resource: Individual Application PFD Management 289

5.11.3.4.1 Introduction 289

5.11.3.4.2 Resource definition 289

5.11.3.4.3 Resource methods 290

5.11.3.5 Void 293

5.11.3A Notifications 293

5.11.3A.1 General 293

5.11.3A.2 PFD Management Notification 294

5.11.3A.2.1 Description 294

5.11.3A.2.2 Target URI 294

5.11.3A.2.3 Standard Methods 294

5.11.4 Used Features 295

5.11.5 Error handling 295

5.11.5.1 General 295

5.11.5.2 Protocol Errors 295

5.11.5.3 Application Errors 295

5.12 ECRControl API 296

5.12.1 Overview 296

5.12.2 Data model 296

5.12.2.1 Data types 296

5.12.2.1.1 Introduction 296

5.12.2.1.2 Type: ECRControl 296

5.12.2.1.3 Type: ECRData 297

5.12.2.1.4 Type: PlmnEcRestrictionDataWb 298

5.12.3 Custom Operations without associated resources 298

5.12.3.1 Overview 298

5.12.3.2 Operation: query 298

5.12.3.2.1 Description 298

5.12.3.2.2 Operation Definition 298

5.12.3.3 Operation: configure 299

5.12.3.3.1 Description 299

5.12.3.3.2 Operation Definition 300

5.12.4 Used Features 300

5.12.5 Error handling 301

5.12.5.1 General 301

5.12.5.2 Protocol Errors 301

5.12.5.3 Application Errors 301

5.13 NpConfiguration API 301

5.13.1 Overview 301

5.13.2 Data model 301

5.13.2.1 Resource data types 301

5.13.2.1.1 Introduction 301

5.13.2.1.2 Type: NpConfiguration 302

5.13.2.1.3 Type: NpConfigurationPatch 304

5.13.2.1.4 Type: ConfigurationNotification 304

5.13.3 Resource structure 305

5.13.3.1 General 305

5.13.3.2 Resource: NP Configurations 305

5.13.3.2.1 Introduction 305

5.13.3.2.2 Resource definition 305

5.13.3.2.3 Resource methods 305

5.13.3.3 Resource: Individual NP Configuration 307

5.13.3.3.1 Introduction 307

5.13.3.3.2 Resource definition 307

5.13.3.3.3 Resource methods 308

5.13.3.4 Void 311

5.13.3A Notifications 311

5.13.3A.1 General 311

5.13.3A.2 Configuration Notification 312

5.13.3A.2.1 Description 312

5.13.3A.2.2 Target URI 312

5.13.3A.2.3 Standard Methods 312

5.13.4 Used Features 313

5.13.5 Error handling 314

5.13.5.1 General 314

5.13.5.2 Protocol Errors 314

5.13.5.3 Application Errors 314

5.14 AsSessionWithQoS API 314

5.14.1 Overview 314

5.14.2 Data model 314

5.14.2.1 Resource data types 314

5.14.2.1.1 Introduction 314

5.14.2.1.2 Type: AsSessionWithQoSSubscription 318

5.14.2.1.3 Type: AsSessionWithQoSSubscriptionPatch 322

5.14.2.1.4 Type: UserPlaneNotificationData 325

5.14.2.1.5 Type: UserPlaneEventReport 326

5.14.2.1.6 Type: QosMonitoringInformation 329

5.14.2.1.7 Type: QosMonitoringInformationRm 331

5.14.2.1.8 Type: QosMonitoringReport 334

5.14.2.1.9 Type: TscQosRequirement 335

5.14.2.1.10 Type: TscQosRequirementRm 335

5.14.2.1.11 Type AdditionInfoAsSessionWithQos 336

5.14.2.1.12 Type: ProblemDetailsAsSessionWithQos 336

5.14.2.1.13 Type AsSessionMediaComponent 336

5.14.2.1.14 Type AsSessionMediaComponentRm 338

5.14.2.1.15 Type: MultiModalFlows 340

5.14.2.1.16 Type: UeAddInfo 340

5.14.2.2 Referenced simple data types and enumerations 341

5.14.2.2.1 Introduction 341

5.14.2.2.2 Simple data types 341

5.14.2.2.3 Enumeration: UserPlaneEvent 341

5.14.3 Resource structure 342

5.14.3.1 General 342

5.14.3.2 Resource: AS Session with Required QoS subscriptions 343

5.14.3.2.1 Introduction 343

5.14.3.2.2 Resource definition 343

5.14.3.2.3 Resource methods 343

5.14.3.3 Resource: Individual AS Session with Required QoS Subscription 345

5.14.3.3.1 Introduction 345

5.14.3.3.2 Resource definition 346

5.14.3.3.3 Resource methods 346

5.14.3.4 Void 350

5.14.3A Notifications 350

5.14.3A.1 General 350

5.14.3A.2 Event Notification 350

5.14.3A.2.1 Description 350

5.14.3A.2.2 Target URI 350

5.14.3A.2.3 Standard Methods 350

5.14.4 Used Features 351

5.14.5 Error handling 353

5.14.5.1 General 353

5.14.5.2 Protocol Errors 353

5.14.5.3 Application Errors 353

5.15 MsisdnLessMoSms API 354

5.15.1 Overview 354

5.15.2 Data model 354

5.15.2.1 Notification data types 354

5.15.2.1.1 Introduction 354

5.15.2.1.2 Type: MsisdnLessMoSmsNotification 354

5.15.2.1.3 Type: MsisdnLessMoSmsNotificationReply 355

5.15.3 Resource structure 355

5.15.3.1 General 355

5.15.3.2 MSISDN-less MO SMS Notification 355

5.15.3.2.1 Introduction 355

5.15.3.2.2 Resource definition 356

5.15.3.2.3 Standard methods 356

5.15.4 Used Features 357

5.15.5 Error handling 357

5.15.5.1 General 357

5.15.5.2 Protocol Errors 357

5.15.5.3 Application Errors 357

5.16 RacsParameterProvisioning API 357

5.16.1 Overview 357

5.16.2 Data model 358

5.16.2.1 Resource data types 358

5.16.2.1.1 Introduction 358

5.16.2.1.2 Type: RacsProvisioningData 358

5.16.2.1.3 Type: RacsFailureReport 359

5.16.2.1.4 Type: RacsConfiguration 359

5.16.2.1.5 Type: RacsProvisioningDataPatch 360

5.16.2.1.6 Type: RacsConfigurationRm 360

5.16.2.2 Referenced simple data types and enumerations 361

5.16.2.2.1 Introduction 361

5.16.2.2.2 Simple data types 361

5.16.2.2.3 Enumeration: RacsFailureCode 361

5.16.3 Resource structure 361

5.16.3.1 General 361

5.16.3.2 Resource: RACS Parameter Provisionings 362

5.16.3.2.1 Introduction 362

5.16.3.2.2 Resource definition 362

5.16.3.2.3 Resource methods 362

5.16.3.3 Resource: Individual RACS Parameter Provisioning 364

5.16.3.3.1 Introduction 364

5.16.3.3.2 Resource definition 364

5.16.3.3.3 Resource methods 364

5.16.4 Used Features 368

5.16.5 Error handling 368

5.16.5.1 General 368

5.16.5.2 Protocol Errors 368

5.16.5.3 Application Errors 368

6 Security 369

7 Using Common API Framework 369

7.1 General 369

7.2 Security 369

Annex A (normative): OpenAPI representation for the APIs defined in the present document 371

A.1 General 371

A.2 Data Types applicable to several APIs 371

A.3 MonitoringEvent API 380

A.4 ResourceManagementOfBdt API 398

A.5 ChargeableParty API 404

A.6 NIDD API 410

A.7 DeviceTriggering API 427

A.8 GMDViaMBMS APIs 434

A.8.1 GMDviaMBMSbyMB2 API 434

A.8.2 GMDviaMBMSbyxMB API 445

A.9 ReportingNetworkStatus API 455

A.10 CpProvisioning API 461

A.11 PfdManagement API 472

A.12 ECRControl API 483

A.13 NpConfiguration API 485

A.14 AsSessionWithQoS API 492

A.15 MsisdnLessMoSms API 508

A.16 RacsParameterProvisioning API 510

Annex B (informative): TS Skeleton Template 516

Annex C (informative): Change history 517
