---
spec: TS 29.518
version: 18.14.0
release: '18'
clause: contents
title: Contents
source_archive: 29518-ie0.zip
source_document: 29518-ie0.docx
source_archive_sha256: aa2789dac49d4f52e56fcc9d04277a5122a40fdf4c22faf877a7832f844daae1
source_document_sha256: 263e0bf7e06ccfa30ab69e661e2a65e0fedf654b22d2675975add5f47f8acad0
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Contents

Foreword 16

1 Scope 17

2 References 17

3 Definitions and abbreviations 19

3.1 Definitions 19

3.2 Abbreviations 19

4 Overview 20

4.1 Introduction 20

5 Services offered by the AMF 21

5.1 Introduction 21

5.2 Namf_Communication Service 23

5.2.1 Service Description 23

5.2.2 Service Operations 23

5.2.2.1 Introduction 23

5.2.2.2 UE Context Operations 24

5.2.2.2.1 UEContextTransfer 24

5.2.2.2.1.1 General 24

5.2.2.2.1.2 Retrieve UE Context after successful UE authentication 26

5.2.2.2.2 RegistrationStatusUpdate 27

5.2.2.2.2.1 General 27

5.2.2.2.3 CreateUEContext 28

5.2.2.2.3.1 General 28

5.2.2.2.3.2 Create UE Context with AMF Relocation 30

5.2.2.2.4 ReleaseUEContext 31

5.2.2.2.4.1 General 31

5.2.2.2.5 RelocateUEContext 32

5.2.2.2.5.1 General 32

5.2.2.2.6 CancelRelocateUEContext 33

5.2.2.2.6.1 General 33

5.2.2.3 UE Specific N1N2 Message Operations 34

5.2.2.3.1 N1N2MessageTransfer 34

5.2.2.3.1.1 General 34

5.2.2.3.1.2 Detailed behaviour of the AMF 36

5.2.2.3.2 N1N2Transfer Failure Notification 39

5.2.2.3.3 N1N2MessageSubscribe 40

5.2.2.3.3.1 General 40

5.2.2.3.4 N1N2MessageUnSubscribe 41

5.2.2.3.4.1 General 41

5.2.2.3.5 N1MessageNotify 41

5.2.2.3.5.1 General 41

5.2.2.3.5.2 Using N1MessageNotify in the Registration with AMF Re-allocation Procedure 42

5.2.2.3.5.3 Using N1MessageNotify in the UE Assisted and UE Based Positioning Procedure 42

5.2.2.3.5.4 Using N1MessageNotify in the UE Configuration Update for transparent UE Policy delivery 43

5.2.2.3.5.5 Using N1MessageNotify in the LCS Event Report, Event Reporting in RRC INACTIVE state procedures, LCS Cancel Location and LCS Periodic-Triggered Invoke Procedures 43

5.2.2.3.5.6 Using N1MessageNotify in the UE triggered policy provisioning procedure to request UE policies 43

5.2.2.3.5.7 Using N1MessageNotify in the procedures applicable to a PRU 44

5.2.2.3.6 N2InfoNotify 44

5.2.2.3.6.1 General 44

5.2.2.3.6.2 Using N2InfoNotify during Inter NG-RAN node N2 based handover procedure 45

5.2.2.3.6.3 Using N2InfoNotify during Location Services procedures 46

5.2.2.3.6.4 Using N2InfoNotify during AMF planned removal procedure with UDSF deployed procedure 46

5.2.2.4 Non-UE N2 Message Operations 47

5.2.2.4.1 NonUeN2MessageTransfer 47

5.2.2.4.1.1 General 47

5.2.2.4.1.2 Obtaining Non UE Associated Network Assistance Data Procedure 47

5.2.2.4.1.3 Warning Request Transfer Procedure 48

5.2.2.4.1.4 Configuration Transfer Procedure 48

5.2.2.4.1.5 RIM Information Transfer Procedures 48

5.2.2.4.1.6 Broadcast of Assistance Data by an LMF 49

5.2.2.4.1.7 Management of network timing synchronization status monitoring procedures 49

5.2.2.4.2 NonUeN2InfoSubscribe 49

5.2.2.4.2.1 General 49

5.2.2.4.3 NonUeN2InfoUnSubscribe 50

5.2.2.4.3.1 General 50

5.2.2.4.4 NonUeN2InfoNotify 51

5.2.2.4.4.1 General 51

5.2.2.4.4.2 Using NonUeN2InfoNotify during Location Services procedures 51

5.2.2.4.4.3 Use of NonUeN2InfoNotify for PWS related events 51

5.2.2.4.4.4 Using NonUeN2InfoNotify during network timing synchronization status monitoring procedure 52

5.2.2.5 AMF Status Change Operations 53

5.2.2.5.1 AMFStatusChangeSubscribe 53

5.2.2.5.1.1 General 53

5.2.2.5.1.2 Creation of a subscription 53

5.2.2.5.1.3 Modification of a subscription 53

5.2.2.5.2 AMFStatusChangeUnSubscribe 54

5.2.2.5.2.1 General 54

5.2.2.5.3 AMFStatusChangeNotify 55

5.2.2.5.3.1 General 55

5.2.2.6 EBIAssignment 55

5.2.2.6.1 General 55

5.3 Namf_EventExposure Service 57

5.3.1 Service Description 57

5.3.2 Service Operations 63

5.3.2.1 Introduction 63

5.3.2.2 Subscribe 63

5.3.2.2.1 General 63

5.3.2.2.2 Creation of a subscription 64

5.3.2.2.3 Modification of a subscription 66

5.3.2.2.4 Remove or add group member UE(s) for a group subscription 67

5.3.2.3 Unsubscribe 68

5.3.2.3.1 General 68

5.3.2.4 Notify 68

5.3.2.4.1 General 68

5.3.2.4.2 Event Subscription Synchronization for specific UE 69

5.4 Namf_MT Service 70

5.4.1 Service Description 70

5.4.2 Service Operations 70

5.4.2.1 Introduction 70

5.4.2.2 EnableUEReachability 70

5.4.2.2.1 General 70

5.4.2.3 ProvideDomainSelectionInfo 72

5.4.2.3.1 General 72

5.4.2.4 EnableGroupReachability 73

5.4.2.4.1 General 73

5.4.2.5 UEReachabilityInfoNotify 74

5.4.2.5.1 General 74

5.5 Namf_Location Service 74

5.5.1 Service Description 74

5.5.2 Service Operations 75

5.5.2.1 Introduction 75

5.5.2.2 ProvidePositioningInfo 75

5.5.2.2.1 General 75

5.5.2.3 EventNotify 77

5.5.2.3.1 General 77

5.5.2.4 ProvideLocationInfo 78

5.5.2.4.1 General 78

5.5.2.5 CancelLocation 78

5.5.2.5.1 General 78

5.6 Namf_MBSBroadcast Service 79

5.6.1 Service Description 79

5.6.2 Service Operations 79

5.6.2.1 Introduction 79

5.6.2.2 ContextCreate 79

5.6.2.3 ContextUpdate 81

5.6.2.4 ContextRelease 82

5.6.2.5 ContextStatusNotify 83

5.7 Namf_MBSCommunication Service 85

5.7.1 Service Description 85

5.7.2 Service Operations 85

5.7.2.1 Introduction 85

5.7.2.2 N2MessageTransfer 85

5.7.2.3 Notify 86

6 API Definitions 87

6.1 Namf_Communication Service API 87

6.1.1 API URI 87

6.1.2 Usage of HTTP 88

6.1.2.1 General 88

6.1.2.2 HTTP standard headers 88

6.1.2.2.1 General 88

6.1.2.2.2 Content type 88

6.1.2.3 HTTP custom headers 88

6.1.2.3.1 General 88

6.1.2.4 HTTP multipart messages 88

6.1.3 Resources 90

6.1.3.1 Overview 90

6.1.3.2 Resource: Individual ueContext 91

6.1.3.2.1 Description 91

6.1.3.2.2 Resource Definition 92

6.1.3.2.3 Resource Standard Methods 92

6.1.3.2.3.1 PUT 92

6.1.3.2.4 Resource Custom Operations 94

6.1.3.2.4.1 Overview 94

6.1.3.2.4.2 Operation: release (POST) 94

6.1.3.2.4.2.1 Description 94

6.1.3.2.4.2.2 Operation Definition 94

6.1.3.2.4.3 Operation: assign-ebi (POST) 95

6.1.3.2.4.3.1 Description 95

6.1.3.2.4.3.2 Operation Definition 95

6.1.3.2.4.4 Operation: transfer (POST) 98

6.1.3.2.4.4.1 Description 98

6.1.3.2.4.4.2 Operation Definition 98

6.1.3.2.4.5 Operation: transfer-update (POST) 99

6.1.3.2.4.5.1 Description 99

6.1.3.2.4.5.2 Operation Definition 100

6.1.3.2.4.6 Operation: relocate (POST) 101

6.1.3.2.4.6.1 Description 101

6.1.3.2.4.6.2 Operation Definition 101

6.1.3.2.4.7 Operation: cancel-relocate (POST) 102

6.1.3.2.4.7.1 Description 102

6.1.3.2.4.7.2 Operation Definition 102

6.1.3.3 Resource: N1N2 Subscriptions Collection for Individual UE Contexts 103

6.1.3.3.1 Description 103

6.1.3.3.2 Resource Definition 103

6.1.3.3.3 Resource Standard Methods 104

6.1.3.3.3.1 POST 104

6.1.3.3.4 Resource Custom Operations 105

6.1.3.4 Resource: N1N2 Individual Subscription 105

6.1.3.4.1 Description 105

6.1.3.4.2 Resource Definition 105

6.1.3.4.3 Resource Standard Methods 105

6.1.3.4.3.1 DELETE 105

6.1.3.4.4 Resource Custom Operations 106

6.1.3.5 Resource: N1N2 Messages Collection 106

6.1.3.5.1 Description 106

6.1.3.5.2 Resource Definition 107

6.1.3.5.3 Resource Standard Methods 107

6.1.3.5.3.1 POST 107

6.1.3.6 Resource: subscriptions collection 111

6.1.3.6.1 Description 111

6.1.3.6.2 Resource Definition 111

6.1.3.6.3 Resource Standard Methods 112

6.1.3.6.3.1 POST 112

6.1.3.7 Resource: individual subscription 113

6.1.3.7.1 Description 113

6.1.3.7.2 Resource Definition 113

6.1.3.7.3 Resource Standard Methods 113

6.1.3.7.3.1 DELETE 113

6.1.3.7.3.2 PUT 114

6.1.3.8 Resource: Non UE N2 Messages Collection 116

6.1.3.8.1 Description 116

6.1.3.8.2 Resource Definition 116

6.1.3.8.3 Resource Standard Methods 116

6.1.3.8.4 Resource Custom Operations 116

6.1.3.8.4.1 Overview 116

6.1.3.8.4.2 Operation: transfer 116

6.1.3.8.4.2.1 Description 116

6.1.3.8.4.2.2 Operation Definition 116

6.1.3.9 Resource: Non UE N2 Messages Subscriptions Collection 118

6.1.3.9.1 Description 118

6.1.3.9.2 Resource Definition 118

6.1.3.9.3 Resource Standard Methods 118

6.1.3.9.3.1 POST 118

6.1.3.9.4 Resource Custom Operations 119

6.1.3.10 Resource: Non UE N2 Message Notification Individual Subscription 120

6.1.3.10.1 Description 120

6.1.3.10.2 Resource Definition 120

6.1.3.10.3 Resource Standard Methods 120

6.1.3.10.3.1 DELETE 120

6.1.3.10.4 Resource Custom Operations 121

6.1.4 Custom Operations without associated resources 121

6.1.5 Notifications 121

6.1.5.1 General 121

6.1.5.2 AMF Status Change Notification 122

6.1.5.2.1 Description 122

6.1.5.2.2 Notification Definition 122

6.1.5.2.3 Notification Standard Methods 122

6.1.5.2.3.1 POST 122

6.1.5.3 Non UE N2 Information Notification 123

6.1.5.3.1 Description 123

6.1.5.3.2 Notification Definition 123

6.1.5.3.3 Notification Standard Methods 124

6.1.5.3.3.1 POST 124

6.1.5.4 N1 Message Notification 124

6.1.5.4.1 Description 124

6.1.5.4.2 Notification Definition 125

6.1.5.4.3 Notification Standard Methods 125

6.1.5.4.3.1 POST 125

6.1.5.5 UE Specific N2 Information Notification 126

6.1.5.5.1 Description 126

6.1.5.5.2 Notification Definition 126

6.1.5.5.3 Notification Standard Methods 126

6.1.5.5.3.1 POST 126

6.1.5.6 N1N2 Transfer Failure Notification 127

6.1.5.6.1 Description 127

6.1.5.6.2 Notification Definition 127

6.1.5.6.3 Notification Standard Methods 128

6.1.5.6.3.1 POST 128

6.1.5.7 Void 129

6.1.6 Data Model 129

6.1.6.1 General 129

6.1.6.2 Structured data types 136

6.1.6.2.1 Introduction 136

6.1.6.2.2 Type: SubscriptionData 137

6.1.6.2.3 Type: AmfStatusChangeNotification 137

6.1.6.2.4 Type: AmfStatusInfo 137

6.1.6.2.5 Type: AssignEbiData 138

6.1.6.2.6 Type: AssignedEbiData 138

6.1.6.2.7 Type: AssignEbiFailed 139

6.1.6.2.8 Type: UEContextRelease 139

6.1.6.2.9 Type: N2InformationTransferReqData 139

6.1.6.2.10 Type: NonUeN2InfoSubscriptionCreateData 140

6.1.6.2.11 Type: NonUeN2InfoSubscriptionCreatedData 141

6.1.6.2.12 Type: UeN1N2InfoSubscriptionCreateData 141

6.1.6.2.13 Type: UeN1N2InfoSubscriptionCreatedData 141

6.1.6.2.14 Type: N2InformationNotification 142

6.1.6.2.15 Type: N2InfoContainer 145

6.1.6.2.16 Type: N1MessageNotification 146

6.1.6.2.17 Type: N1MessageContainer 148

6.1.6.2.18 Type: N1N2MessageTransferReqData 149

6.1.6.2.19 Type: N1N2MessageTransferRspData 153

6.1.6.2.20 Type: RegistrationContextContainer 154

6.1.6.2.21 Type: AreaOfValidity 156

6.1.6.2.22 Void 156

6.1.6.2.23 Type: UeContextTransferReqData 156

6.1.6.2.24 Type: UeContextTransferRspData 157

6.1.6.2.25 Type: UeContext 158

6.1.6.2.26 Type: N2SmInformation 169

6.1.6.2.27 Type: N2InfoContent 170

6.1.6.2.28 Type: NrppaInformation 170

6.1.6.2.29 Type: PwsInformation 171

6.1.6.2.30 Type: N1N2MsgTxfrFailureNotification 173

6.1.6.2.31 Type: N1N2MessageTransferError 173

6.1.6.2.32 Type: N1N2MsgTxfrErrDetail 174

6.1.6.2.33 Type: N2InformationTransferRspData 174

6.1.6.2.34 Type: MmContext 175

6.1.6.2.35 Type: SeafData 179

6.1.6.2.36 Type: NasSecurityMode 179

6.1.6.2.37 Type: PduSessionContext 180

6.1.6.2.38 Type: NssaiMapping 185

6.1.6.2.39 Type: UeRegStatusUpdateReqData 186

6.1.6.2.40 Type: AssignEbiError 186

6.1.6.2.41 Type: UeContextCreateData 187

6.1.6.2.42 Type: UeContextCreatedData 188

6.1.6.2.43 Type: UeContextCreateError 188

6.1.6.2.44 Type: NgRanTargetId 189

6.1.6.2.45 Type: N2InformationTransferError 189

6.1.6.2.46 Type: PWSResponseData 189

6.1.6.2.47 Type: PWSErrorData 190

6.1.6.2.48 Void 190

6.1.6.2.49 Type: NgKsi 190

6.1.6.2.50 Type: KeyAmf 190

6.1.6.2.51 Type: ExpectedUeBehavior 190

6.1.6.2.52 Type: UeRegStatusUpdateRspData 191

6.1.6.2.53 Type: N2RanInformation 191

6.1.6.2.54 Type: N2InfoNotificationRspData 191

6.1.6.2.55 Type: SmallDataRateStatusInfo 191

6.1.6.2.56 Type: SmfChangeInfo 192

6.1.6.2.57 Type: V2xContext 192

6.1.6.2.58 Type: ImmediateMdtConf 193

6.1.6.2.59 Type: V2xInformation 195

6.1.6.2.60 Type: EpsNasSecurityMode 195

6.1.6.2.61 Type: UeContextRelocateData 196

6.1.6.2.62 Type: UeContextRelocatedData 196

6.1.6.2.63 Void 196

6.1.6.2.64 Type: EcRestrictionDataWb 197

6.1.6.2.65 Type: ExtAmfEventSubscription 197

6.1.6.2.66 Type: AmfEventSubscriptionAddInfo 198

6.1.6.2.67 Type: UeContextCancelRelocateData 200

6.1.6.2.68 Type: UeDifferentiationInfo 200

6.1.6.2.69 Type: CeModeBInd 201

6.1.6.2.70 Type: LteMInd 201

6.1.6.2.71 Type: NpnAccessInfo 201

6.1.6.2.72 Type: ProseContext 202

6.1.6.2.73 Type: AnalyticsSubscription 202

6.1.6.2.74 Type: NwdafSubscription 203

6.1.6.2.75 Type: UpdpSubscriptionData 203

6.1.6.2.76 Type: ProSeInformation 203

6.1.6.2.77 Type: ReleaseSessionInfo 203

6.1.6.2.78 Type: AreaOfInterestEventState 204

6.1.6.2.79 Type: TssInformation 204

6.1.6.2.80 Type: AmPolicyInfoContainer 205

6.1.6.2.81 Type: RslpInformation 205

6.1.6.2.82 Type: A2xContext 205

6.1.6.2.83 Type: A2xInformation 205

6.1.6.2.84 Type: LcsUpContext 206

6.1.6.2.85 Type: DeregInactTimerInfo 206

6.1.6.2.86 Type: TssRspPerNgran 206

6.1.6.2.87 Type: SliceReplacementMapping 206

6.1.6.2.88 Type: SliceDeregInactConfig 206

6.1.6.3 Simple data types and enumerations 207

6.1.6.3.1 Introduction 207

6.1.6.3.2 Simple data types 207

6.1.6.3.3 Enumeration: StatusChange 207

6.1.6.3.4 Enumeration: N2InformationClass 208

6.1.6.3.5 Enumeration: N1MessageClass 208

6.1.6.3.6 Enumeration: N1N2MessageTransferCause 209

6.1.6.3.7 Enumeration: UeContextTransferStatus 210

6.1.6.3.8 Enumeration: N2InformationTransferResult 210

6.1.6.3.9 Enumeration: CipheringAlgorithm 210

6.1.6.3.10 Enumeration: IntegrityAlgorithm 210

6.1.6.3.11 Enumeration: SmsSupport 210

6.1.6.3.12 Enumeration: ScType 211

6.1.6.3.13 Enumeration: KeyAmfType 211

6.1.6.3.14 Enumeration: TransferReason 211

6.1.6.3.15 Enumeration: PolicyReqTrigger 211

6.1.6.3.16 Enumeration: RatSelector 212

6.1.6.3.17 Enumeration: NgapIeType 212

6.1.6.3.18 Enumeration: N2InfoNotifyReason 212

6.1.6.3.19 Enumeration: SmfChangeIndication 212

6.1.6.3.20 Enumeration: SbiBindingLevel 213

6.1.6.3.21 Enumeration: EpsNasCipheringAlgorithm 213

6.1.6.3.22 Enumeration: EpsNasIntegrityAlgorithm 213

6.1.6.3.23 Enumeration: PeriodicCommunicationIndicator 213

6.1.6.3.24 Enumeration: UuaaMmStatus 213

6.1.6.3.25 Enumeration: ReleaseCause 214

6.1.6.3.26 Enumeration: NgranFailureInfo 214

6.1.6.3.27 Enumeration: XrDeviceWith2Rx 214

6.1.6.4 Binary data 214

6.1.6.4.1 Introduction 214

6.1.6.4.2 N1 Message 215

6.1.6.4.3 N2 Information 216

6.1.6.4.3.1 Introduction 216

6.1.6.4.3.2 NGAP IEs 216

6.1.6.4.3.3 NGAP Messages 218

6.1.6.4.4 Mobile Terminated Data 219

6.1.6.4.5 GTP-C Message 219

6.1.7 Error Handling 220

6.1.7.1 General 220

6.1.7.2 Protocol Errors 220

6.1.7.3 Application Errors 220

6.1.8 Feature Negotiation 223

6.1.9 Security 227

6.1.10 HTTP redirection 227

6.2 Namf_EventExposure Service API 227

6.2.1 API URI 227

6.2.2 Usage of HTTP 228

6.2.2.1 General 228

6.2.2.2 HTTP standard headers 228

6.2.2.2.1 General 228

6.2.2.2.2 Content type 228

6.2.2.3 HTTP custom headers 229

6.2.2.3.1 General 229

6.2.3 Resources 229

6.2.3.1 Overview 229

6.2.3.2 Resource: Subscriptions collection 229

6.2.3.2.1 Description 229

6.2.3.2.2 Resource Definition 229

6.2.3.2.3 Resource Standard Methods 230

6.2.3.2.3.1 POST 230

6.2.3.2.4 Resource Custom Operations 231

6.2.3.3 Resource: Individual subscription 231

6.2.3.3.1 Description 231

6.2.3.3.2 Resource Definition 231

6.2.3.3.3 Resource Standard Methods 232

6.2.3.3.3.1 PATCH 232

6.2.3.3.3.2 DELETE 233

6.2.3.3.4 Resource Custom Operations 234

6.2.4 Custom Operations without associated resources 234

6.2.5 Notifications 234

6.2.5.1 Void 234

6.2.5.2 AMF Event Notification 234

6.2.5.2.1 Notification Definition 234

6.2.5.2.3 Notification Standard Methods 235

6.2.5.2.3.1 POST 235

6.2.6 Data Model 236

6.2.6.1 General 236

6.2.6.2 Structured data types 238

6.2.6.2.1 Introduction 238

6.2.6.2.2 Type: AmfEventSubscription 239

6.2.6.2.3 Type: AmfEvent 242

6.2.6.2.4 Type: AmfEventNotification 249

6.2.6.2.5 Type: AmfEventReport 250

6.2.6.2.6 Type: AmfEventMode 255

6.2.6.2.7 Type: AmfEventState 257

6.2.6.2.8 Type: RmInfo 258

6.2.6.2.9 Type: CmInfo 258

6.2.6.2.10 Void 258

6.2.6.2.11 Type: CommunicationFailure 258

6.2.6.2.12 Type: AmfCreateEventSubscription 258

6.2.6.2.13 Type: AmfCreatedEventSubscription 259

6.2.6.2.14 Type: AmfUpdateEventSubscriptionItem 260

6.2.6.2.15 Type: AmfUpdatedEventSubscription 263

6.2.6.2.16 Type: AmfEventArea 264

6.2.6.2.17 Type: LadnInfo 264

6.2.6.2.18 Type: AmfUpdateEventOptionItem 265

6.2.6.2.19 Type: 5GsUserStateInfo 265

6.2.6.2.20 Type: TrafficDescriptor 266

6.2.6.2.21 Type: UEIdExt 266

6.2.6.2.22 Type: AmfEventSubsSyncInfo 266

6.2.6.2.23 Type: AmfEventSubscriptionInfo 267

6.2.6.2.24 Type: TargetArea 267

6.2.6.2.25 Type: SnssaiTaiMapping 267

6.2.6.2.26 Type: SupportedSnssai 268

6.2.6.2.27 Type: UeInAreaFilter 268

6.2.6.2.28 Type: IdleStatusIndication 269

6.2.6.2.29 Type: UeAccessBehaviorReportItem 270

6.2.6.2.30 Type: UeLocationTrendsReportItem 271

6.2.6.2.31 Type: DispersionArea 272

6.2.6.2.32 Type: MmTransactionLocationReportItem 272

6.2.6.2.33 Type: MmTransactionSliceReportItem 273

6.2.6.2.34 Type: SliceAreaRestrictionInfo 273

6.2.6.3 Simple data types and enumerations 273

6.2.6.3.1 Introduction 273

6.2.6.3.2 Simple data types 273

6.2.6.3.3 Enumeration: AmfEventType 274

6.2.6.3.4 Enumeration: AmfEventTrigger 276

6.2.6.3.5 Enumeration: LocationFilter 277

6.2.6.3.6 Void 277

6.2.6.3.7 Enumeration: UeReachability 277

6.2.6.3.8 Void 277

6.2.6.3.9 Enumeration: RmState 277

6.2.6.3.10 Enumeration: CmState 277

6.2.6.3.11 Enumeration: 5GsUserState 278

6.2.6.3.12 Enumeration: LossOfConnectivityReason 278

6.2.6.3.13 Enumeration: ReachabilityFilter 278

6.2.6.3.14 Enumeration: UeType 278

6.2.6.3.15 Enumeration: AccessStateTransitionType 279

6.2.6.3.16 Enumeration: SubTerminationReason 279

6.2.6.4 Binary data 279

6.2.7 Error Handling 279

6.2.7.1 General 279

6.2.7.2 Protocol Errors 279

6.2.7.3 Application Errors 279

6.2.8 Feature Negotiation 280

6.2.9 Security 283

6.2.10 HTTP redirection 283

6.3 Namf_MT Service API 284

6.3.1 API URI 284

6.3.2 Usage of HTTP 284

6.3.2.1 General 284

6.3.2.2 HTTP standard headers 284

6.3.2.2.1 General 284

6.3.2.2.2 Content type 284

6.3.2.3 HTTP custom headers 285

6.3.2.3.1 General 285

6.3.3 Resources 285

6.3.3.1 Overview 285

6.3.3.2 Resource: ueReachInd 285

6.3.3.2.1 Description 285

6.3.3.2.2 Resource Definition 286

6.3.3.2.3 Resource Standard Methods 286

6.3.3.2.3.1 PUT 286

6.3.3.2.4 Resource Custom Operations 288

6.3.3.3 Resource: ueContext 288

6.3.3.3.1 Description 288

6.3.3.3.2 Resource Definition 288

6.3.3.3.3 Resource Standard Methods 288

6.3.3.3.3.1 GET 288

6.3.3.3.4 Resource Custom Operations 290

6.3.3.4 Resource: ueContexts collection 290

6.3.3.4.1 Description 290

6.3.3.4.2 Resource Definition 290

6.3.3.4.3 Resource Standard Methods 290

6.3.3.4.4 Resource Custom Operations 290

6.3.3.4.4.1 Overview 290

6.3.3.4.4.2 Operation: enable-group-reachability 290

6.3.3.4.4.2.1 Description 290

6.3.3.4.4.2.2 Operation Definition 290

6.3.4 Custom Operations without associated resources 292

6.3.5 Notifications 292

6.3.5.1 General 292

6.3.5.2 UE Reachability Info Notify 292

6.3.5.2.1 Notification Definition 292

6.3.5.2.3 Notification Standard Methods 292

6.3.5.2.3.1 POST 292

6.3.6 Data Model 293

6.3.6.1 General 293

6.3.6.2 Structured data types 294

6.3.6.2.1 Introduction 294

6.3.6.2.2 Type: EnableUeReachabilityReqData 295

6.3.6.2.3 Type: EnableUeReachabilityRspData 295

6.3.6.2.4 Type: UeContextInfo 296

6.3.6.2.5 Type: ProblemDetailsEnableUeReachability 296

6.3.6.2.6 Type: AdditionInfoEnableUeReachability 296

6.3.6.2.7 Type: EnableGroupReachabilityReqData 297

6.3.6.2.8 Type: EnableGroupReachabilityRspData 297

6.3.6.2.9 Type: UeInfo 297

6.3.6.2.10 Type: ReachabilityNotificationData 297

6.3.6.2.11 Type: ReachableUeInfo 298

6.3.6.2.12 Type: QosFlowInfo 298

6.3.6.3 Simple data types and enumerations 298

6.3.6.3.1 Introduction 298

6.3.6.3.2 Simple data types 298

6.3.6.3.3 Enumeration: UeContextInfoClass 298

6.3.6.4 Binary data 299

6.3.7 Error Handling 299

6.3.7.1 General 299

6.3.7.2 Protocol Errors 299

6.3.7.3 Application Errors 299

6.3.8 Feature Negotiation 299

6.3.9 Security 300

6.3.10 HTTP redirection 301

6.4 Namf_Location Service API 301

6.4.1 API URI 301

6.4.2 Usage of HTTP 301

6.4.2.1 General 301

6.4.2.2 HTTP standard headers 302

6.4.2.2.1 General 302

6.4.2.2.2 Content type 302

6.4.2.3 HTTP custom headers 302

6.4.2.3.1 General 302

6.4.3 Resources 302

6.4.3.1 Overview 302

6.4.3.2 Resource: Individual UE Context 303

6.4.3.2.1 Description 303

6.4.3.2.2 Resource Definition 303

6.4.3.2.3 Resource Standard Methods 303

6.4.3.2.4 Resource Custom Operations 304

6.4.3.2.4.1 Overview 304

6.4.3.2.4.2 Operation: provide-pos-info (POST) 304

6.4.3.2.4.2.1 Description 304

6.4.3.2.4.2.2 Operation Definition 304

6.4.3.2.4.3 Operation: provide-loc-info (POST) 306

6.4.3.2.4.3.1 Description 306

6.4.3.2.4.3.2 Operation Definition 306

6.4.3.2.4.4 Operation: cancel-pos-info (POST) 307

6.4.3.2.4.4.1 Description 307

6.4.3.2.4.4.2 Operation Definition 307

6.4.4 Custom Operations without associated resources 308

6.4.5 Notifications 309

6.4.5.1 General 309

6.4.5.2 Event Notify 309

6.4.5.2.1 Description 309

6.4.5.2.2 Notification Definition 309

6.4.5.2.3 Notification Standard Methods 309

6.4.5.2.3.1 POST 309

6.4.6 Data Model 310

6.4.6.1 General 310

6.4.6.2 Structured data types 314

6.4.6.2.1 Introduction 314

6.4.6.2.2 Type: RequestPosInfo 315

6.4.6.2.3 Type: ProvidePosInfo 320

6.4.6.2.4 Type: NotifiedPosInfo 324

6.4.6.2.5 Type: RequestLocInfo 328

6.4.6.2.6 Type: ProvideLocInfo 329

6.4.6.2.7 Type: CancelPosInfo 329

6.4.6.2.11 Type: ProvidePosInfoExt 330

6.4.6.2.12 Type: NotifiedPosInfoExt 330

6.4.6.3 Simple data types and enumerations 330

6.4.6.3.1 Introduction 330

6.4.6.3.2 Simple data types 330

6.4.6.3.3 Enumeration: LocationType 331

6.4.6.3.4 Enumeration: LocationEvent 331

6.4.6.3.5 Enumeration: LocationPrivacyVerResult 331

6.4.6.3.6 Enumeration: LpHapType 332

6.4.7 Error Handling 332

6.4.7.1 General 332

6.4.7.2 Protocol Errors 332

6.4.7.3 Application Errors 332

6.4.8 Feature Negotiation 332

6.4.9 Security 333

6.4.10 HTTP redirection 333

6.5 Namf_MBSBroadcast Service API 334

6.5.1 API URI 334

6.5.2 Usage of HTTP 334

6.5.2.1 General 334

6.5.2.2 HTTP standard headers 334

6.5.2.2.1 General 334

6.5.2.2.2 Content type 334

6.5.2.3 HTTP custom headers 335

6.5.2.3.1 General 335

6.5.2.4 HTTP multipart messages 335

6.5.3 Resources 336

6.5.3.1 Overview 336

6.5.3.2 Resource: Broadcast MBS session contexts collection 336

6.5.3.2.1 Description 336

6.5.3.2.2 Resource Definition 336

6.5.3.2.3 Resource Standard Methods 337

6.5.3.2.3.1 POST 337

6.5.3.2.4 Resource Custom Operations 338

6.5.3.3 Resource: Individual broadcast MBS session context 338

6.5.3.3.1 Description 338

6.5.3.3.2 Resource Definition 338

6.5.3.3.3 Resource Standard Methods 338

6.5.3.3.3.1 DELETE 338

6.5.3.3.4 Resource Custom Operations 339

6.5.3.2.4.2 Operation: update (POST) 339

6.5.3.2.4.2.1 Description 339

6.5.3.2.4.2.2 Operation Definition 339

6.5.4 Custom Operations without associated resources 340

6.5.5 Notifications 340

6.5.5.1 General 340

6.5.5.2 Broadcast MBS Session Context Status Notification 341

6.5.5.2.1 Description 341

6.5.5.2.2 Target URI 341

6.5.5.2.3 Notification Standard Methods 341

6.5.5.2.3.1 POST 341

6.5.6 Data Model 342

6.5.6.1 General 342

6.5.6.2 Structured data types 343

6.5.6.2.1 Introduction 343

6.5.6.2.2 Type: ContextCreateReqData 343

6.5.6.2.3 Type: ContextCreateRspData 343

6.5.6.2.4 Type: ContextStatusNotification 344

6.5.6.2.5 Type: ContextUpdateReqData 345

6.5.6.2.6 Type: ContextUpdateRspData 345

6.5.6.2.7 Type: N2MbsSmInfo 346

6.5.6.2.8 Type: OperationEvent 346

6.5.6.2.9 Type: NgranFailureEvent 346

6.5.6.2.10 Type: ContextStatusNotificationResponse 347

6.5.6.3 Simple data types and enumerations 347

6.5.6.3.1 Introduction 347

6.5.6.3.2 Simple data types 347

6.5.6.3.3 Enumeration: OperationStatus 347

6.5.6.3.4 Enumeration: NgapIeType 348

6.5.6.3.5 Enumeration: OpEventType 348

6.5.6.3.6 Enumeration: NgranFailureIndication 348

6.5.6.4 Binary data 348

6.5.6.4.1 Introduction 348

6.5.6.4.2 N2 Information 349

6.5.6.4.2.1 Introduction 349

6.5.6.4.2.2 NGAP IEs 349

6.5.7 Error Handling 349

6.5.7.1 General 349

6.5.7.2 Protocol Errors 349

6.5.7.3 Application Errors 349

6.5.8 Feature Negotiation 350

6.5.9 Security 350

6.5.10 HTTP redirection 350

6.6 Namf_MBSCommunication Service API 350

6.6.1 API URI 350

6.6.2 Usage of HTTP 351

6.6.2.1 General 351

6.6.2.2 HTTP standard headers 351

6.6.2.2.1 General 351

6.6.2.2.2 Content type 351

6.6.2.3 HTTP custom headers 351

6.6.2.3.1 General 351

6.6.2.4 HTTP multipart messages 352

6.6.3 Resources 352

6.6.3.1 Overview 352

6.6.3.1 Resource: N2 Message Handler (Custom Operation) 353

6.6.3.1.1 Description 353

6.6.3.1.2 Resource Definition 353

6.6.3.1.3 Resource Standard Methods 353

6.6.3.1.4 Resource Custom Operations 353

6.6.3.1.4.1 Overview 353

6.6.3.1.4.2 Operation: transfer 353

6.6.3.1.4.2.1 Description 353

6.6.3.1.4.2.2 Operation Definition 353

6.6.4 Custom Operations without associated resources 354

6.6.5 Notifications 354

6.6.5.1 General 354

6.6.5.2 Notification 355

6.6.5.2.1 Description 355

6.6.5.2.2 Notification Definitionn 355

6.6.5.2.3 Notification Standard Methods 355

6.6.5.2.3.1 POST 355

6.6.6 Data Model 356

6.6.6.1 General 356

6.6.6.2 Structured data types 357

6.6.6.2.1 Introduction 357

6.6.6.2.2 Type: MbsN2MessageTransferReqData 357

6.6.6.2.3 Type: MbsN2MessageTransferRspData 357

6.6.6.2.4 Type: N2MbsSmInfo 358

6.6.6.2.5 Type: Notification 358

6.6.6.2.6 Type: RanFailure 358

6.6.6.3 Simple data types and enumerations 358

6.6.6.3.1 Introduction 358

6.6.6.3.2 Simple data types 358

6.6.6.3.3 Enumeration: MbsNgapIeType 359

6.6.6.3.4 Enumeration: RanFailureIndication 359

6.6.6.4 Binary data 359

6.6.6.4.1 Introduction 359

6.6.6.4.2 N2 Information 359

6.6.6.4.2.1 Introduction 359

6.6.6.4.2.2 NGAP IEs 359

6.6.7 Error Handling 360

6.6.7.1 General 360

6.6.7.2 Protocol Errors 360

6.6.7.3 Application Errors 360

6.6.8 Feature Negotiation 360

6.6.9 Security 361

6.6.10 HTTP redirection 361

Annex A (normative): OpenAPI specification 361

A.1 General 361

A.2 Namf_Communication API 362

A.3 Namf_EventExposure API 421

A.4 Namf_MT 436

A.5 Namf_Location 442

A.6 Namf_MBSBroadcast API 450

A.7 Namf_MBSCommunication API 462

Annex B (Informative): HTTP Multipart Messages 466

B.1 Example of HTTP multipart message 466

B.1.1 General 466

B.1.2 Example HTTP multipart message with N2 Information binary data 466

Annex C (informative): Change history 467
