---
spec: TS 29.503
version: 18.13.0
release: '18'
clause: contents
title: Contents
source_archive: 29503-id0.zip
source_document: 29503-id0.docx
source_archive_sha256: 4c773b0d93af7d82fab12329d4c7d06807cc92271da897e23ee94e89a7112afa
source_document_sha256: 6c856fd8095c9d5afe17dbc8d23248638223b9f6c80bf2129176c4a516b50fef
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Contents

Foreword 23

1 Scope 24

2 References 24

3 Definitions and abbreviations 27

3.1 Definitions 27

3.2 Abbreviations 27

4 Overview 28

4.1 Introduction 28

5 Services offered by the UDM 30

5.1 Introduction 30

5.2 Nudm_SubscriberDataManagement Service 31

5.2.1 Service Description 31

5.2.2 Service Operations 31

5.2.2.1 Introduction 31

5.2.2.2 Get 32

5.2.2.2.1 General 32

5.2.2.2.2 Slice Selection Subscription Data Retrieval 33

5.2.2.2.3 Access and Mobility Subscription Data Retrieval 34

5.2.2.2.4 SMF Selection Subscription Data Retrieval 35

5.2.2.2.5 Session Management Subscription Data Retrieval 36

5.2.2.2.6 SMS Subscription Data Retrieval 36

5.2.2.2.7 SMS Management Subscription Data Retrieval 37

5.2.2.2.8 UE Context In SMF Data Retrieval 37

5.2.2.2.9 Retrieval Of Multiple Data Sets 38

5.2.2.2.10 Identifier Translation 38

5.2.2.2.11 Shared Subscription Data Retrieval 39

5.2.2.2.12 UE Context In SMSF Data Retrieval 40

5.2.2.2.13 Trace data Retrieval 40

5.2.2.2.14 Group Identifier Translation 41

5.2.2.2.15 LCS Privacy Data Retrieval 42

5.2.2.2.16 LCS Mobile Originated Data Retrieval 43

5.2.2.2.17 Enhanced Coverage Restriction Data Retrieval 43

5.2.2.2.18 V2X Subscription Data Retrieval 44

5.2.2.2.19 LCS Broadcast Assistance Subscription Data Retrieval 44

5.2.2.2.20 UE Context In AMF Data Retrieval 45

5.2.2.2.21 Individual Shared Subscription Data Retrieval 45

5.2.2.2.22 Prose Subscription Data Retrieval 46

5.2.2.2.23 5MBS Subscription Data Retrieval 46

5.2.2.2.24 User Consent Subscription Data Retrieval 47

5.2.2.2.25 Multiple Identifiers Translation 47

5.2.2.2.26 Time Synchronization Subscription Data Retrieval 48

5.2.2.2.27 LCS Subscription Data Retrieval 48

5.2.2.2.28 Ranging and Sidelink Positioning Subscription Data Retrieval 49

5.2.2.2.29 A2X Subscription Data Retrieval 49

5.2.2.2.30 Ranging and Sidelink Positioning Privacy Data Retrieval 50

5.2.2.3 Subscribe 50

5.2.2.3.1 General 50

5.2.2.3.2 Subscription to notifications of data change 50

5.2.2.3.3 Subscription to notifications of shared data change 51

5.2.2.4 Unsubscribe 52

5.2.2.4.1 General 52

5.2.2.4.2 Unsubscribe to notifications of data change 52

5.2.2.4.3 Unsubscribe to notifications of shared data change 53

5.2.2.5 Notification 53

5.2.2.5.1 General 53

5.2.2.5.2 Data Change Notification To NF 53

5.2.2.5.3 UDR-initiated Data Restoration Notification 54

5.2.2.6 Info 55

5.2.2.6.1 General 55

5.2.2.6.2 Providing acknowledgement of Steering of Roaming 55

5.2.2.6.3 Providing acknowledgement of UE parameters update 56

5.2.2.6.4 Providing acknowledgement of UE for Network Slicing Subscription Change 56

5.2.2.6.5 Providing acknowledgement of UE for CAG configuration change 57

5.2.2.6.6 Triggering Update of Steering Of Roaming information 57

5.2.2.7 ModifySubscription 58

5.2.2.7.1 General 58

5.2.2.7.2 Modification of a subscription to notifications of data change 58

5.2.2.7.3 Modification of a subscription to notifications of shared data change 59

5.3 Nudm_UEContextManagement Service 59

5.3.1 Service Description 59

5.3.2 Service Operations 59

5.3.2.1 Introduction 59

5.3.2.2 Registration 61

5.3.2.2.1 General 61

5.3.2.2.2 AMF registration for 3GPP access 61

5.3.2.2.3 AMF registration for non 3GPP access 62

5.3.2.2.4 SMF registration 63

5.3.2.2.5 SMSF Registration for 3GPP Access 64

5.3.2.2.6 SMSF Registration for Non 3GPP Access 65

5.3.2.2.7 IP-SM-GW registration 66

5.3.2.2.8 NWDAF registration 66

5.3.2.3 DeregistrationNotification 67

5.3.2.3.1 General 67

5.3.2.3.2 UDM initiated NF Deregistration 67

5.3.2.4 Deregistration 69

5.3.2.4.1 General 69

5.3.2.4.2 AMF deregistration for 3GPP access 69

5.3.2.4.3 AMF deregistration for non-3GPP access 69

5.3.2.4.4 SMF deregistration 70

5.3.2.4.5 SMSF Deregistration for 3GPP Access 71

5.3.2.4.6 SMSF Deregistration for Non 3GPP Access 71

5.3.2.4.7 IP-SM-GW deregistration 72

5.3.2.4.8 NWDAF deregistration 72

5.3.2.5 Get 72

5.3.2.5.1 General 72

5.3.2.5.2 Amf3GppAccessRegistration Information Retrieval 73

5.3.2.5.3 AmfNon3GppAccessRegistration Information Retrieval 73

5.3.2.5.4 Void 74

5.3.2.5.5 SmsfRegistration Information Retrieval for 3GPP Access 74

5.3.2.5.6 SmsfRegistration Information Retrieval for Non-3GPP Access 75

5.3.2.5.7 SmfRegistration Information Retrieval 75

5.3.2.5.8 Individual SmfRegistration Information Retrieval 76

5.3.2.5.9 Location Information Retrieval 76

5.3.2.5.10 Retrieval Of Multiple UE Registration Data Sets 77

5.3.2.5.11 IP-SM-GW Registration Information Retrieval 77

5.3.2.5.12 NwdafRegistration Information Retrieval 78

5.3.2.6 Update 78

5.3.2.6.1 General 78

5.3.2.6.2 Update A Parameter (e.g. PEI) in the AMF Registration For 3GPP Access 78

5.3.2.6.3 Update A Parameter (e.g. PEI) in the AMF Registration For Non 3GPP Access 79

5.3.2.6.4 Update A Parameter (e.g. analyticsId(s)) in the NWDAF Registration 80

5.3.2.6.5 Update A Parameter (e.g. PGW FQDN) in the SMF Registration 81

5.3.2.6.6 Update A Parameter (e.g. Memory Available) in the SMSF Registration for 3GPP Access 81

5.3.2.6.7 Update A Parameter (e.g. Memory Available) in the SMSF Registration for non-3GPP Access 82

5.3.2.7 P-CSCF-RestorationNotification 82

5.3.2.7.1 General 82

5.3.2.7.2 UDM initiated P-CSCF-Restoration 82

5.3.2.8 P-CSCF-RestorationTrigger 83

5.3.2.8.1 General 83

5.3.2.8.2 P-CSCF-RestorationTrigger 83

5.3.2.9 AMFDeregistration 83

5.3.2.9.1 General 83

5.3.2.9.2 AMF-Deregistration 84

5.3.2.10 PEI-Update 84

5.3.2.10.1 General 84

5.3.2.10.2 PEI Update 84

5.3.2.11 Roaming Information Update 85

5.3.2.11.1 General 85

5.3.2.11.2 Roaming Information Update 85

5.3.2.12 DataRestorationNotification 85

5.3.2.12.1 General 85

5.3.2.12.2 UDR-initiated Data Restoration 85

5.3.2.13 SendRoutingInfoForSM 86

5.3.2.13.1 General 86

5.3.2.13.2 Send Routing Information For SM 86

5.3.2.14 StaleCheckNotification 87

5.3.2.14.1 General 87

5.3.2.14.2 UDM initiated Stale SMF Registration Check 87

5.3.2.15 Re-AuthenticationNotification 88

5.3.2.15.1 General 88

5.3.2.15.2 Reauthentication Notify 88

5.3.2.16 AuthTrigger 88

5.3.2.16.1 General 88

5.3.2.16.2 Auth Trigger 88

5.4 Nudm_UEAuthentication Service 89

5.4.1 Service Description 89

5.4.2 Service Operations 89

5.4.2.1 Introduction 89

5.4.2.2 Get 90

5.4.2.2.1 General 90

5.4.2.2.2 Authentication Information Retrieval 90

5.4.2.2.3 FN-RG Authentication 91

5.4.2.3 ResultConfirmationInform 91

5.4.2.3.1 General 91

5.4.2.3.2 Authentication Confirmation 92

5.4.2.3.3 Authentication Result Removal 92

5.4.2.4 GetHssAv 93

5.4.2.4.1 General 93

5.4.2.4.2 HSS Authentication Vector Retrieval 93

5.4.2.5 GetGbaAv 93

5.4.2.5.1 General 93

5.4.2.5.2 GBA Authentication Vector Retrieval 93

5.4.2.6 GetProseAv 94

5.4.2.6.1 General 94

5.4.2.6.2 ProSe Authentication Vector Retrieval 94

5.4.2.7 Notification 95

5.4.2.7.1 General 95

5.4.2.7.2 UDR-initiated Data Restoration Notification 95

5.5 Nudm_EventExposure Service 95

5.5.1 Service Description 95

5.5.2 Service Operations 96

5.5.2.1 Introduction 96

5.5.2.2 Subscribe 96

5.5.2.2.1 General 96

5.5.2.2.2 Subscription to Notification of event occurrence 96

5.5.2.2.3 Void 99

5.5.2.3 Unsubscribe 99

5.5.2.3.1 General 99

5.5.2.3.2 Unsubscribe to notifications of event occurrence 99

5.5.2.4 Notify 99

5.5.2.4.1 General 99

5.5.2.4.2 Event Occurrence Notification 99

5.5.2.4.3 Monitoring Revocation Notification 100

5.5.2.4.4 UDR-initiated Data Restoration 101

5.5.2.5 ModifySubscription 101

5.5.2.5.1 General 101

5.5.2.5.2 Modification of a subscription 101

5.5.2.5.3 Remove or add group member UE(s) for a group subscription 102

5.6 Nudm_ParameterProvision Service 103

5.6.1 Service Description 103

5.6.2 Service Operations 103

5.6.2.1 Introduction 103

5.6.2.2 Update 103

5.6.2.2.1 General 103

5.6.2.2.2 Subscription data update 104

5.6.2.2.3 5G VN Group modification 104

5.6.2.2.4 SoR Information update 105

5.6.2.2.5 Parameter Provisioning Data Entry per AF update 106

5.6.2.2.6 5G-MBS-Group modification 107

5.6.2.3 Create 107

5.6.2.3.1 General 107

5.6.2.3.2 5G-VN-Group creation 107

5.6.2.3.3 Parameter Provisioning Data Entry per AF creation 108

5.6.2.3.4 5G-MBS-Group creation 109

5.6.2.4 Delete 109

5.6.2.4.1 General 109

5.6.2.4.2 5G-VN-Group deletion 110

5.6.2.4.3 Parameter Provisioning Data Entry per AF deletion 110

5.6.2.4.4 5G-MBS-Group deletion 111

5.6.2.5 Get 112

5.6.2.5.1 General 112

5.6.2.5.2 5G-VN-Group get 112

5.6.2.5.3 Parameter Provisioning Data Entry per AF get 112

5.6.2.5.4 5G-MBS-Group get 113

5.7 Nudm_NIDDAuthorization Service 114

5.7.1 Service Description 114

5.7.2 Service Operations 114

5.7.2.1 Introduction 114

5.7.2.2 Get 114

5.7.2.2.1 General 114

5.7.2.2.2 NIDD Authorization Data Retrieval 114

5.7.2.3 Notification 115

5.7.2.3.1 General 115

5.7.2.3.2 NIDD Authorization Data Update Notification 115

5.8 Nudm_MT Service 116

5.8.1 Service Description 116

5.8.2 Service Operations 116

5.8.2.1 Introduction 116

5.8.2.2 ProvideUeInfo 116

5.8.2.2.1 General 116

5.8.2.2.2 UE Information Retrieval 116

5.8.2.3 ProvideLocationInfo 117

5.8.2.3.1 General 117

5.8.2.3.2 Network Provided Location Information Request 117

5.9 Nudm_ServiceSpecificAuthorization Service 118

5.9.1 Service Description 118

5.9.2 Service Operations 118

5.9.2.1 Introduction 118

5.9.2.2 Create 118

5.9.2.2.1 General 118

5.9.2.2.2 Service Specific Authorization Data Retrieval 118

5.9.2.3 UpdateNotify 119

5.9.2.3.1 General 119

5.9.2.3.2 Service Specific Authorization Data Update Notification 119

5.10 Nudm_ReportSMDeliveryStatus Service 121

5.10.1 Service Description 121

5.10.2 Service Operations 121

5.10.2.1 Introduction 121

5.10.2.2 Request 121

5.10.2.2.1 General 121

5.10.2.2.2 Report the SM-Delivery Status 121

5.11 Nudm_UEIdentifier Service 122

5.11.1 Service Description 122

5.11.2 Service Operations 122

5.11.2.1 Introduction 122

5.11.2.2 Deconceal 122

6 API Definitions 122

6.1 Nudm_SubscriberDataManagement Service API 122

6.1.1 API URI 122

6.1.2 Usage of HTTP 123

6.1.2.1 General 123

6.1.2.2 HTTP standard headers 123

6.1.2.2.1 General 123

6.1.2.2.2 Content type 123

6.1.2.2.3 Cache-Control 123

6.1.2.2.4 ETag 123

6.1.2.2.5 If-None-Match 124

6.1.2.2.6 Last-Modified 124

6.1.2.2.7 If-Modified-Since 124

6.1.2.2.8 When to Use Entity-Tags and Last-Modified Dates 124

6.1.2.3 HTTP custom headers 124

6.1.2.3.1 General 124

6.1.3 Resources 124

6.1.3.1 Overview 124

6.1.3.2 Resource: Nssai (Document) 129

6.1.3.2.1 Description 129

6.1.3.2.2 Resource Definition 129

6.1.3.2.3 Resource Standard Methods 129

6.1.3.3 Resource: SdmSubscriptions (Collection) 131

6.1.3.3.1 Description 131

6.1.3.3.2 Resource Definition 131

6.1.3.3.3 Resource Standard Methods 131

6.1.3.4 Resource: Individual subscription (Document) 132

6.1.3.4.1 Description 132

6.1.3.4.2 Resource Definition 132

6.1.3.4.3 Resource Standard Methods 132

6.1.3.5 Resource: AccessAndMobilitySubscriptionData (Document) 134

6.1.3.5.1 Description 134

6.1.3.5.2 Resource Definition 134

6.1.3.5.3 Resource Standard Methods 134

6.1.3.5.4 Resource Custom Operations 136

6.1.3.6 Resource: SmfSelectionSubscriptionData (Document) 137

6.1.3.6.1 Description 137

6.1.3.6.2 Resource Definition 137

6.1.3.6.3 Resource Standard Methods 137

6.1.3.7 Resource: UeContextInSmfData (Document) 138

6.1.3.7.1 Description 138

6.1.3.7.2 Resource Definition 138

6.1.3.7.3 Resource Standard Methods 138

6.1.3.8 Resource: SessionManagementSubscriptionData (Document) 139

6.1.3.8.1 Description 139

6.1.3.8.2 Resource Definition 139

6.1.3.8.3 Resource Standard Methods 140

6.1.3.9 Resource: SMSSubscriptionData (Document) 141

6.1.3.9.1 Description 141

6.1.3.9.2 Resource Definition 141

6.1.3.9.3 Resource Standard Methods 141

6.1.3.10 Resource: SMSManagementSubscriptionData (Document) 142

6.1.3.10.1 Description 142

6.1.3.10.2 Resource Definition 142

6.1.3.10.3 Resource Standard Methods 143

6.1.3.11 Resource: Supi (Document) 144

6.1.3.11.1 Description 144

6.1.3.11.2 Resource Definition 144

6.1.3.11.3 Resource Standard Methods 144

6.1.3.12 Resource: IdTranslationResult (Document) 146

6.1.3.12.1 Description 146

6.1.3.12.2 Resource Definition 146

6.1.3.12.3 Resource Standard Methods 146

6.1.3.13 Resource: SorAck (Document) 148

6.1.3.13.1 Description 148

6.1.3.13.2 Resource Definition 148

6.1.3.13.3 Resource Standard Methods 148

6.1.3.14 Resource: TraceData (Document) 149

6.1.3.14.1 Description 149

6.1.3.14.2 Resource Definition 149

6.1.3.14.3 Resource Standard Methods 149

6.1.3.15 Resource: SharedData (Collection) 150

6.1.3.15.1 Description 150

6.1.3.15.2 Resource Definition 150

6.1.3.15.3 Resource Standard Methods 150

6.1.3.16 Resource: SharedDataSubscriptions (Collection) 151

6.1.3.16.1 Description 151

6.1.3.16.2 Resource Definition 151

6.1.3.16.3 Resource Standard Methods 152

6.1.3.17 Resource: Individual subscription (Document) 152

6.1.3.17.1 Description 152

6.1.3.17.2 Resource Definition 153

6.1.3.17.3 Resource Standard Methods 153

6.1.3.18 Resource: UeContextInSmsfData (Document) 154

6.1.3.18.1 Description 154

6.1.3.18.2 Resource Definition 154

6.1.3.18.3 Resource Standard Methods 155

6.1.3.19 Resource: UpuAck (Document) 155

6.1.3.19.1 Description 155

6.1.3.19.2 Resource Definition 155

6.1.3.19.3 Resource Standard Methods 156

6.1.3.20 Resource: GroupIdentifiers (Document) 156

6.1.3.20.1 Description 156

6.1.3.20.2 Resource Definition 156

6.1.3.20.3 Resource Standard Methods 156

6.1.3.21 Resource: SnssaisAck (Document) 158

6.1.3.21.1 Description 158

6.1.3.21.2 Resource Definition 158

6.1.3.21.3 Resource Standard Methods 158

6.1.3.22 Resource: CagAck (Document) 158

6.1.3.22.1 Description 158

6.1.3.22.2 Resource Definition 159

6.1.3.22.3 Resource Standard Methods 159

6.1.3.23 Resource: LcsPrivacySubscriptionData (Document) 159

6.1.3.23.1 Description 159

6.1.3.23.2 Resource Definition 159

6.1.3.23.3 Resource Standard Methods 160

6.1.3.24 Resource: LcsMobileOriginatedSubscriptionData (Document) 160

6.1.3.24.1 Description 160

6.1.3.24.2 Resource Definition 160

6.1.3.24.3 Resource Standard Methods 161

6.1.3.25 Resource: EnhancedCoverageRestrictionData 161

6.1.3.25.1 Description 161

6.1.3.25.2 Resource Definition 161

6.1.3.25.3 Resource Standard Methods 162

6.1.3.26 Resource: UeContextInAmfData (Document) 162

6.1.3.26.1 Description 162

6.1.3.26.2 Resource Definition 162

6.1.3.26.3 Resource Standard Methods 163

6.1.3.27 Resource: V2xSubscriptionData (Document) 163

6.1.3.27.1 Description 163

6.1.3.27.2 Resource Definition 163

6.1.3.27.3 Resource Standard Methods 163

6.1.3.28 Resource: LcsBroadcastAssistanceSubscriptionData (Document) 164

6.1.3.28.1 Description 164

6.1.3.28.2 Resource Definition 164

6.1.3.28.3 Resource Standard Methods 164

6.1.3.29 Resource: IndividualSharedData (Document) 165

6.1.3.29.1 Description 165

6.1.3.29.2 Resource Definition 165

6.1.3.29.3 Resource Standard Methods 165

6.1.3.30 Resource: ProseSubscriptionData (Document) 166

6.1.3.30.1 Description 166

6.1.3.30.2 Resource Definition 166

6.1.3.30.3 Resource Standard Methods 166

6.1.3.31 Resource: MbsSubscriptionData (Document) 167

6.1.3.31.1 Description 167

6.1.3.31.2 Resource Definition 167

6.1.3.31.3 Resource Standard Methods 167

6.1.3.32 Resource: UcSubscriptionData (Document) 168

6.1.3.32.1 Description 168

6.1.3.32.2 Resource Definition 168

6.1.3.32.3 Resource Standard Methods 168

6.1.3.33 Resource: MultipleIdentifiers (Document) 169

6.1.3.33.1 Description 169

6.1.3.33.2 Resource Definition 169

6.1.3.33.3 Resource Standard Methods 169

6.1.3.34 Resource: TimeSyncSubscriptionData (Document) 170

6.1.3.34.1 Description 170

6.1.3.34.2 Resource Definition 170

6.1.3.35.3 Resource Standard Methods 170

6.1.3.36 Resource: LcsSubscriptionData (Document) 171

6.1.3.36.1 Description 171

6.1.3.36.2 Resource Definition 171

6.1.3.36.3 Resource Standard Methods 172

6.1.3.37 Resource: RangingSlPosSubscriptionData (Document) 172

6.1.3.37.1 Description 172

6.1.3.37.2 Resource Definition 172

6.1.3.37.3 Resource Standard Methods 173

6.1.3.38 Resource: A2xSubscriptionData (Document) 173

6.1.3.38.1 Description 173

6.1.3.38.2 Resource Definition 173

6.1.3.39 Resource: RangingSlPrivacySubscriptionData (Document) 174

6.1.3.39.1 Description 174

6.1.3.39.2 Resource Definition 174

6.1.3.39.3 Resource Standard Methods 175

6.1.4 Custom Operations without associated resources 175

6.1.5 Notifications 175

6.1.5.1 General 175

6.1.5.2 Data Change Notification 176

6.1.5.3 Data Restoration Notification 177

6.1.6 Data Model 178

6.1.6.1 General 178

6.1.6.2 Structured data types 185

6.1.6.2.1 Introduction 185

6.1.6.2.2 Type: Nssai 186

6.1.6.2.3 Type: SdmSubscription 187

6.1.6.2.4 Type: AccessAndMobilitySubscriptionData 191

6.1.6.2.5 Type: SmfSelectionSubscriptionData 197

6.1.6.2.6 Type: DnnInfo 198

6.1.6.2.7 Type: SnssaiInfo 198

6.1.6.2.8 Type: SessionManagementSubscriptionData 199

6.1.6.2.9 Type: DnnConfiguration 202

6.1.6.2.10 Void 205

6.1.6.2.11 Type: PduSessionTypes 205

6.1.6.2.12 Type: SscModes 205

6.1.6.2.13 Type: SmsSubscriptionData 205

6.1.6.2.14 Type: SmsManagementSubscriptionData 206

6.1.6.2.15 Type: SubscriptionDataSets 206

6.1.6.2.16 Type: UeContextInSmfData 207

6.1.6.2.17 Type: PduSession 207

6.1.6.2.18 Type: IdTranslationResult 208

6.1.6.2.19 Void 209

6.1.6.2.20 Void 209

6.1.6.2.21 Type: ModificationNotification 209

6.1.6.2.22 Type: IpAddress 209

6.1.6.2.23 Type: UeContextInSmsfData 209

6.1.6.2.24 Type: SmsfInfo 210

6.1.6.2.25 Type: AcknowledgeInfo 210

6.1.6.2.26 Type: SorInfo 211

6.1.6.2.27 Type: SharedData 214

6.1.6.2.28 Type: PgwInfo 215

6.1.6.2.29 Type: TraceDataResponse 215

6.1.6.2.30 Type: SteeringContainer 215

6.1.6.2.31 Type: SdmSubsModification 216

6.1.6.2.32 Type: EmergencyInfo 216

6.1.6.2.33 Type: UpuInfo 217

6.1.6.2.34 Type: GroupIdentifiers 218

6.1.6.2.35 Type: NiddInformation 218

6.1.6.2.36 Type: CagData 219

6.1.6.2.37 Type: CagInfo 219

6.1.6.2.38 Type: AdditionalSnssaiData 220

6.1.6.2.39 Type: VnGroupData 221

6.1.6.2.40 Type: AppDescriptor 222

6.1.6.2.41 Type: AppPortId 222

6.1.6.2.42 Type: LcsPrivacyData 223

6.1.6.2.43 Type: Lpi 223

6.1.6.2.44 Type: UnrelatedClass 223

6.1.6.2.45 Type: PlmnOperatorClass 224

6.1.6.2.46 Type: ValidTimePeriod 224

6.1.6.2.47 Type: LcsMoData 224

6.1.6.2.48 Type: EcRestrictionDataWb 224

6.1.6.2.49 Type: ExpectedUeBehaviourData 225

6.1.6.2.50 Void 226

6.1.6.2.51 Void 226

6.1.6.2.52 Type: SuggestedPacketNumDl 226

6.1.6.2.53 Void 226

6.1.6.2.54 Type: FrameRouteInfo 226

6.1.6.2.55 Type: SorUpdateInfo 226

6.1.6.2.56 Type: EnhancedCoverageRestrictionData 226

6.1.6.2.57 Type: EdrxParameters 227

6.1.6.2.58 Type: PtwParameters 227

6.1.6.2.59 Void 228

6.1.6.2.60 Void 228

6.1.6.2.61 Type: Void 228

6.1.6.2.62 Type: ExternalUnrelatedClass 228

6.1.6.2.63 Type: AfExternal 228

6.1.6.2.64 Type: LcsClientExternal 228

6.1.6.2.65 Type: LcsClientGroupExternal 229

6.1.6.2.66 Type: ServiceTypeUnrelatedClass 229

6.1.6.2.67 Type: UeId 229

6.1.6.2.68 Type: DefaultUnrelatedClass 230

6.1.6.2.69 Type: ContextInfo 230

6.1.6.2.70 Type: UeContextInAmfData 230

6.1.6.2.71 Type: V2xSubscriptionData 231

6.1.6.2.72 Type: LcsBroadcastAssistanceTypesData 232

6.1.6.2.73 Type: DatasetNames 236

6.1.6.2.74 Type: PlmnRestriction 236

6.1.6.2.75 Type: PcfSelectionAssistanceInfo 236

6.1.6.2.76 Type: ProseSubscriptionData 237

6.1.6.2.77 Type: IpIndex 237

6.1.6.2.78 Type: AerialUeSubscriptionInfo 237

6.1.6.2.79 Type: SmSubsData 237

6.1.6.2.80 Type: ExtendedSmSubsData 237

6.1.6.2.81 Type: AmfInfo 238

6.1.6.2.82 Type: ProSeAllowedPlmn 238

6.1.6.2.83 Type: ImmediateReport 238

6.1.6.2.84 Type: MbsSubscriptionData 238

6.1.6.2.85 Type: UcSubscriptionData 239

6.1.6.2.86 Type: UeContextInSmfDataSubFilter 239

6.1.6.2.87 Type: UeIdentifiers 239

6.1.6.2.88 Type: SupiInfo 239

6.1.6.2.89 Type: TimeSyncData 240

6.1.6.2.90 Type: TimeSyncSubscriptionData 240

6.1.6.2.91 Type: AfRequestAuthorization 240

6.1.6.2.92 Type: TimeSyncServiceId 241

6.1.6.2.93 Type: LcsSubscriptionData 241

6.1.6.2.94 Type: ConditionalCagInfo 242

6.1.6.2.95 Void 242

6.1.6.2.96 Void 242

6.1.6.2.97 Type: RangingSlPosSubscriptionData 242

6.1.6.2.98 Type: RangingSlPosPlmn 243

6.1.6.2.99 Type: AppSpecificExpectedUeBehaviourData 243

6.1.6.2.100 Type: ExpectedUeBehaviourThreshold 244

6.1.6.2.101 Type: A2xSubscriptionData 246

6.1.6.2.102 Type: RangingSlPosQos 247

6.1.6.2.103 Type: MbsrOperationAllowed 247

6.1.6.2.104 Type: GpsiInfo 248

6.1.6.2.105 Type: DnnLadnServiceAreas 248

6.1.6.2.106 Type: DnnLadnServiceArea 248

6.1.6.2.107 Type: GptpAllowedInfo 249

6.1.6.2.108 Type: AstiAllowedInfo 250

6.1.6.2.109 Type: MbsrLocationInfo 250

6.1.6.2.110 Type: MbsrTimeInfo 251

6.1.6.2.111 Type: RecurTime 251

6.1.6.2.112 Type: RangingSlPrivacyData 251

6.1.6.2.113 Type: Rslppi 252

6.1.6.2.114 Type: RangingSlUnrelatedClass 252

6.1.6.2.115 Type: RangingSlPlmnOperatorClass 252

6.1.6.2.116 Type: RangingSlDefaultUnrelatedClass 253

6.1.6.2.117 Type: RangingSlExternalUnrelatedClass 253

6.1.6.2.118 Type:RangingSlAfExternal 253

6.1.6.2.119 Type: RangingSlLcsClientExternal 254

6.1.6.2.120 Type: RangingSlAppIDUnrelatedClass 254

6.1.6.3 Simple data types and enumerations 254

6.1.6.3.1 Introduction 254

6.1.6.3.2 Simple data types 254

6.1.6.3.3 Enumeration: DataSetName 257

6.1.6.3.4 Void 257

6.1.6.3.5 Void 257

6.1.6.3.6 Void 257

6.1.6.3.7 Enumeration: PduSessionContinuityInd 257

6.1.6.3.8 Enumeration: LocationPrivacyInd 258

6.1.6.3.9 Enumeration: PrivacyCheckRelatedAction 258

6.1.6.3.10 Enumeration: LcsClientClass 258

6.1.6.3.11 Enumeration: LcsMoServiceClass 258

6.1.6.3.12 Enumeration: OperationMode 259

6.1.6.3.13 Enumeration: SorUpdateIndicator 259

6.1.6.3.14 Enumeration: CodeWordInd 259

6.1.6.3.15 Enumeration: MdtUserConsent 259

6.1.6.3.16 Enumeration: SharedDataTreatmentInstruction 260

6.1.6.3.17 Enumeration: GpsiType 260

6.1.6.3.18 Enumeration: AerialUeIndication 260

6.1.6.3.19 Enumeration: ProseDirectAllowed 261

6.1.6.3.20 Enumeration: UcPurpose 261

6.1.6.3.21 Enumeration: UserConsent 262

6.1.6.3.22 Enumeration: NsacAdmissionMode 262

6.1.6.3.23 Enumeration: PruInd 262

6.1.6.3.24 Enumeration: AreaUsageInd 262

6.1.6.3.25 Void 263

6.1.6.3.26 Void 263

6.1.6.3.27 Void 263

6.1.6.3.28 Enumeration: RangingSlPosAllowed 263

6.1.6.3.29 Enumeration: ExpectedUeBehaviourDataset 263

6.1.6.3.30 Enumeration: UpLoCRepIndAf 264

6.1.6.3.31 Enumeration: RecurType 264

6.1.6.3.32 Enumeration: RangingSlPrivacyInd 264

6.1.6.3.33 Enumeration: RangingSlPrivacyCheckRelatedAction 264

6.1.7 Error Handling 264

6.1.7.1 General 264

6.1.7.2 Protocol Errors 265

6.1.7.3 Application Errors 265

6.1.8 Feature Negotiation 265

6.1.9 Security 269

6.1.10 HTTP redirection 270

6.2 Nudm_UEContextManagement Service API 271

6.2.1 API URI 271

6.2.2 Usage of HTTP 271

6.2.2.1 General 271

6.2.2.2 HTTP standard headers 271

6.2.2.2.1 General 271

6.2.2.2.2 Content type 271

6.2.2.3 HTTP custom headers 271

6.2.2.3.1 General 271

6.2.3 Resources 272

6.2.3.1 Overview 272

6.2.3.2 Resource: Amf3GppAccessRegistration (Document) 277

6.2.3.2.1 Description 277

6.2.3.2.2 Resource Definition 277

6.2.3.2.3 Resource Standard Methods 277

6.2.3.2.4 Resource Custom Operations 280

6.2.3.3 Resource: AmfNon3GppAccessRegistration (Document) 281

6.2.3.3.1 Description 281

6.2.3.3.2 Resource Definition 281

6.2.3.3.3 Resource Standard Methods 282

6.2.3.4 Resource: SmfRegistrations 284

6.2.3.4.1 Description 284

6.2.3.4.2 Resource Definition 284

6.2.3.4.3 Resource Standard Methods 284

6.2.3.5 Resource: IndividualSmfRegistration (Document) 285

6.2.3.5.1 Resource Definition 285

6.2.3.5.2 Resource Standard Methods 285

6.2.3.6 Resource: Smsf3GppAccessRegistration (Document) 288

6.2.3.6.1 Description 288

6.2.3.6.2 Resource Definition 288

6.2.3.6.3 Resource Standard Methods 289

6.2.3.7 Resource: SmsfNon3GppAccessRegistration (Document) 292

6.2.3.7.1 Description 292

6.2.3.7.2 Resource Definition 292

6.2.3.7.3 Resource Standard Methods 292

6.2.3.8 Resource: Location 295

6.2.3.8.1 Description 295

6.2.3.8.2 Resource Definition 296

6.2.3.8.3 Resource Standard Methods 296

6.2.3.9 Resource: Registrations 296

6.2.3.9.1 Description 296

6.2.3.9.2 Resource Definition 296

6.2.3.9.3 Resource Standard Methods 297

6.2.3.9.4 Resource Custom Operations 298

6.2.3.10 Resource: IpSmGwRegistration 299

6.2.3.10.1 Description 299

6.2.3.10.2 Resource Definition 299

6.2.3.10.3 Resource Standard Methods 299

6.2.3.11 Resource: NwdafRegistration (Document) 301

6.2.3.11.1 Resource Definition 301

6.2.3.11.2 Resource Standard Methods 301

6.2.3.12 Resource: NwdafRegistrations 303

6.2.3.12.1 Description 303

6.2.3.12.2 Resource Definition 303

6.2.3.12.3 Resource Standard Methods 303

6.2.3.13 Resource: AuthTrigger 304

6.2.3.13.1 Description 304

6.2.3.13.2 Resource Definition 304

6.2.3.13.3 Resource Standard Methods 304

6.2.3.13.4 Resource Custom Operations 304

6.2.4 Custom Operations without associated resources 305

6.2.4.1 Overview 305

6.2.4.2 Operation: Trigger P-CSCF Restoration 305

6.2.4.2.1 Description 305

6.2.4.2.2 Operation Definition 305

6.2.5 Notifications 306

6.2.5.1 General 306

6.2.5.2 Deregistration Notification 306

6.2.5.3 P-CSCF Restoration Notification 307

6.2.5.4 Data Restoration Notification 308

6.2.5.5 Stale Check Notification 309

6.2.5.6 Re-AuthenticationNotification Notification 311

6.2.6 Data Model 312

6.2.6.1 General 312

6.2.6.2 Structured data types 315

6.2.6.2.1 Introduction 315

6.2.6.2.2 Type: Amf3GppAccessRegistration 316

6.2.6.2.3 Type: AmfNon3GppAccessRegistration 321

6.2.6.2.4 Type: SmfRegistration 326

6.2.6.2.5 Type: DeregistrationData 329

6.2.6.2.6 Type: SmsfRegistration 330

6.2.6.2.7 Type: Amf3GppAccessRegistrationModification 332

6.2.6.2.8 Type: AmfNon3GppAccessRegistrationModification 333

6.2.6.2.9 Type: PcscfRestorationNotification 333

6.2.6.2.10 Type: NetworkNodeDiameterAddress 333

6.2.6.2.11 Type: EpsIwkPgw 334

6.2.6.2.12 Type: TriggerRequest 334

6.2.6.2.13 Type: AmfDeregInfo 334

6.2.6.2.14 Type: EpsInterworkingInfo 334

6.2.6.2.15 Type: LocationInfo 334

6.2.6.2.16 Type: RegistrationLocationInfo 335

6.2.6.2.17 Type: VgmlcAddress 335

6.2.6.2.18 Type: PeiUpdateInfo 335

6.2.6.2.19 Type: RegistrationDataSets 335

6.2.6.2.20 Type: IpSmGwRegistration 336

6.2.6.2.20A Type: SmfRegistrationInfo 336

6.2.6.2.21 Type: NwdafRegistration 337

6.2.6.2.22 Type: NwdafRegistrationModification 337

6.2.6.2.23 Type: SmfRegistrationModification 337

6.2.6.2.24 Type: RoamingInfoUpdate 338

6.2.6.2.25 Type: DataRestorationNotification 339

6.2.6.2.26 Type: PcscfAddress 339

6.2.6.2.27 Type: NwdafRegistrationInfo 339

6.2.6.2.28 Type: RoutingInfoSmRequest 340

6.2.6.2.29 Type: RoutingInfoSmResponse 340

6.2.6.2.30 Type: IpSmGwInfo 340

6.2.6.2.31 Type: IpSmGwGuidance 340

6.2.6.2.32 Type: SmsRouterInfo 341

6.2.6.2.33 Type: SmsfRegistrationModification 341

6.2.6.2.34 Type: PduSessionIds 341

6.2.6.2.35 Type: ReauthNotificationInfo 341

6.2.6.2.36 Type: AuthTriggerInfo 342

6.2.6.2.37 Type: DeregistrationRespData 342

6.2.6.3 Simple data types and enumerations 342

6.2.6.3.1 Introduction 342

6.2.6.3.2 Simple data types 342

6.2.6.3.3 Enumeration: DeregistrationReason 342

6.2.6.3.4 Enumeration: ImsVoPs 343

6.2.6.3.5 Enumeration: RegistrationReason 343

6.2.6.3.6 Enumeration: RegistrationDataSetName 344

6.2.6.3.7 Enumeration: UeReachableInd 344

6.2.7 Error Handling 344

6.2.7.1 General 344

6.2.7.2 Protocol Errors 344

6.2.7.3 Application Errors 344

6.2.8 Feature Negotiation 346

6.2.9 Security 346

6.2.10 HTTP redirection 347

6.3 Nudm_UEAuthentication Service API 347

6.3.1 API URI 347

6.3.2 Usage of HTTP 347

6.3.2.1 General 347

6.3.2.2 HTTP standard headers 347

6.3.2.2.1 General 347

6.3.2.2.2 Content type 347

6.3.2.3 HTTP custom headers 348

6.3.2.3.1 General 348

6.3.3 Resources 348

6.3.3.1 Overview 348

6.3.3.2 Resource: SecurityInformation (Custom operation) 350

6.3.3.2.1 Description 350

6.3.3.2.2 Resource Definition 351

6.3.3.2.3 Resource Standard Methods 351

6.3.3.2.4 Resource Custom Operations 351

6.3.3.3 Resource: AuthEvents (Collection) 352

6.3.3.3.1 Description 352

6.3.3.3.2 Resource Definition 352

6.3.3.3.3 Resource Standard Methods 352

6.3.3.4 Resource: SecurityInformationForRg 353

6.3.3.4.1 Description 353

6.3.3.4.2 Resource Definition 353

6.3.3.4.3 Resource Standard Methods 354

6.3.3.5 Resource: HssSecurityInformation (Custom operation) 354

6.3.3.5.1 Description 354

6.3.3.5.2 Resource Definition 354

6.3.3.5.3 Resource Standard Methods 355

6.3.3.5.4 Resource Custom Operations 355

6.3.3.6 Resource: Individual AuthEvent 356

6.3.3.6.1 Resource Definition 356

6.3.3.6.2 Resource Standard Methods 356

6.3.3.7 Resource: GbaSecurityInformation (Custom operation) 357

6.3.3.7.1 Description 357

6.3.3.7.2 Resource Definition 357

6.3.3.7.3 Resource Standard Methods 357

6.3.3.7.4 Resource Custom Operations 357

6.3.3.8 Resource: ProSeSecurityInformation (Custom operation) 358

6.3.3.8.1 Description 358

6.3.3.8.2 Resource Definition 358

6.3.3.8.3 Resource Standard Methods 358

6.3.3.8.4 Resource Custom Operations 359

6.3.3.8.4.1 Overview 359

6.3.3.8.4.2 Operation: generate-av 359

6.3.4 Custom Operations without associated resources 359

6.3.5 Notifications 360

6.3.5.1 General 360

6.3.5.2 Data Restoration Notification 360

6.3.6 Data Model 361

6.3.6.1 General 361

6.3.6.2 Structured data types 363

6.3.6.2.1 Introduction 363

6.3.6.2.2 Type: AuthenticationInfoRequest 364

6.3.6.2.3 Type: AuthenticationInfoResult 365

6.3.6.2.4 Type: AvEapAkaPrime 366

6.3.6.2.5 Type: Av5GHeAka 366

6.3.6.2.6 Type: ResynchronizationInfo 366

6.3.6.2.7 Type: AuthEvent 367

6.3.6.2.8 Type: AuthenticationVector 367

6.3.6.2.9 Type: RgAuthCtx 368

6.3.6.2.10 Type: HssAuthenticationInfoRequest 368

6.3.6.2.11 Type: HssAuthenticationInfoResult 368

6.3.6.2.12 Type: HssAuthenticationVectors 368

6.3.6.2.13 Type: AvEpsAka 369

6.3.6.2.14 Type: AvImsGbaEapAka 369

6.3.6.2.15 Type: GbaAuthenticationInfoRequest 369

6.3.6.2.16 Type: GbaAuthenticationInfoResult 369

6.3.6.2.17 Type: ProSeAuthenticationInfoRequest 369

6.3.6.2.18 Type: ProSeAuthenticationInfoResult 370

6.3.6.2.19 Type: ProSeAuthenticationVectors 370

6.3.6.3 Simple data types and enumerations 370

6.3.6.3.1 Introduction 370

6.3.6.3.2 Simple data types 370

6.3.6.3.3 Enumeration: AuthType 371

6.3.6.3.4 Enumeration: AvType 371

6.3.6.3.5 Enumeration: HssAuthType 371

6.3.6.3.6 Enumeration: HssAvType 371

6.3.6.3.7 Enumeration: HssAuthTypeInUri 372

6.3.6.3.8 Enumeration: AccessNetworkId 372

6.3.6.3.9 Enumeration: NodeType 372

6.3.6.3.10 Enumeration: GbaAuthType 372

6.3.7 Error Handling 372

6.3.7.1 General 372

6.3.7.2 Protocol Errors 373

6.3.7.3 Application Errors 373

6.3.8 Feature Negotiation 373

6.3.9 Security 373

6.3.10 HTTP redirection 374

6.4 Nudm_EventExposure Service API 374

6.4.1 API URI 374

6.4.2 Usage of HTTP 374

6.4.2.1 General 374

6.4.2.2 HTTP standard headers 375

6.4.2.2.1 General 375

6.4.2.2.2 Content type 375

6.4.2.3 HTTP custom headers 375

6.4.2.3.1 General 375

6.4.3 Resources 375

6.4.3.1 Overview 375

6.4.3.2 Resource: EeSubscriptions (Collection) 376

6.4.3.2.1 Description 376

6.4.3.2.2 Resource Definition 376

6.4.3.2.3 Resource Standard Methods 376

6.4.3.3 Resource: Individual subscription (Document) 377

6.4.3.3.1 Resource Definition 377

6.4.3.3.2 Resource Standard Methods 378

6.4.4 Custom Operations without associated resources 379

6.4.5 Notifications 379

6.4.5.1 General 379

6.4.5.2 Event Occurrence Notification 380

6.4.5.3 Monitoring Revocation Notification 380

6.4.5.4 Data Restoration Notification 381

6.4.6 Data Model 382

6.4.6.1 General 382

6.4.6.2 Structured data types 384

6.4.6.2.1 Introduction 384

6.4.6.2.2 Type: EeSubscription 385

6.4.6.2.3 Type: MonitoringConfiguration 388

6.4.6.2.4 Type: MonitoringReport 391

6.4.6.2.5 Type: Report 392

6.4.6.2.6 Type: ReportingOptions 393

6.4.6.2.7 Type: ChangeOfSupiPeiAssociationReport 395

6.4.6.2.8 Type: RoamingStatusReport 395

6.4.6.2.9 Type: CreatedEeSubscription 396

6.4.6.2.10 Type: LocationReportingConfiguration 398

6.4.6.2.11 Type: CnTypeChangeReport 398

6.4.6.2.12 Type: ReachabilityForSmsReport 399

6.4.6.2.13 Type: DatalinkReportingConfiguration 399

6.4.6.2.14 Type: CmInfoReport 399

6.4.6.2.15 Type: LossConnectivityCfg 399

6.4.6.2.16 Type: PduSessionStatusCfg 400

6.4.6.2.17 Type: LossConnectivityReport 400

6.4.6.2.18 Type: LocationReport 400

6.4.6.2.19 Type: PdnConnectivityStatReport 400

6.4.6.2.20 Type: ReachabilityReport 400

6.4.6.2.21 Type: ReachabilityForDataConfiguration 401

6.4.6.2.22 Type: EeMonitoringRevoked 401

6.4.6.2.23 Type: MonitoringEvent 401

6.4.6.2.24 Type: FailedMonitoringConfiguration 402

6.4.6.2.25 Type: MonitoringSuspension 402

6.4.6.2.26 Type: GroupMembListChanges 402

6.4.6.2.27 Type: EeSubscriptionErrorAddInfo 403

6.4.6.2.28 Type: EeSubscriptionError 403

6.4.6.3 Simple data types and enumerations 403

6.4.6.3.1 Introduction 403

6.4.6.3.2 Simple data types 403

6.4.6.3.3 Enumeration: EventType 405

6.4.6.3.4 Enumeration: LocationAccuracy 408

6.4.6.3.5 Enumeration: CnType 408

6.4.6.3.6 Enumeration: AssociationType 408

6.4.6.3.7 Enumeration: EventReportMode 408

6.4.6.3.8 Enumeration: ReachabilityForSmsConfiguration 409

6.4.6.3.9 Enumeration: PdnConnectivityStatus 409

6.4.6.3.10 Enumeration: ReachabilityForDataReportConfig 409

6.4.6.3.11 Enumeration: RevokedCause 409

6.4.6.3.12 Enumeration: FailedCause 410

6.4.6.3.13 Enumeration: SubscriptionType 410

6.4.7 Error Handling 410

6.4.7.1 General 410

6.4.7.2 Protocol Errors 410

6.4.7.3 Application Errors 410

6.4.8 Feature Negotiation 412

6.4.9 Security 412

6.4.10 HTTP redirection 413

6.5 Nudm_ParameterProvision Service API 413

6.5.1 API URI 413

6.5.2 Usage of HTTP 413

6.5.2.1 General 413

6.5.2.2 HTTP standard headers 414

6.5.2.2.1 General 414

6.5.2.2.2 Content type 414

6.5.2.3 HTTP custom headers 414

6.5.2.3.1 General 414

6.5.3 Resources 414

6.5.3.1 Overview 414

6.5.3.2 Resource: PpData 416

6.5.3.2.1 Description 416

6.5.3.2.2 Resource Definition 416

6.5.3.2.3 Resource Standard Methods 416

6.5.3.3 Resource: 5GVnGroupConfiguration 417

6.5.3.3.1 Description 417

6.5.3.3.2 Resource Definition 418

6.5.3.3.3 Resource Standard Methods 418

6.5.3.4 Resource: ParameterProvisioningDataEntry 420

6.5.3.4.1 Description 420

6.5.3.4.2 Resource Definition 421

6.5.3.4.3 Resource Standard Methods 421

6.5.3.5 Resource: MulticastMbsGroupMemb 423

6.5.3.5.1 Description 423

6.5.3.5.2 Resource Definition 423

6.5.3.5.3 Resource Standard Methods 423

6.5.4 Custom Operations without associated resources 425

6.5.5 Notifications 426

6.5.6 Data Model 426

6.5.6.1 General 426

6.5.6.2 Structured data types 427

6.5.6.2.1 Introduction 427

6.5.6.2.2 Type: PpData 428

6.5.6.2.3 Type: CommunicationCharacteristics 429

6.5.6.2.4 Type: PpSubsRegTimer 429

6.5.6.2.5 Type: PpActiveTime 430

6.5.6.2.6 Type: 5GVnGroupConfiguration 430

6.5.6.2.7 Type: 5GVnGroupData 432

6.5.6.2.8 Type: ExpectedUeBehaviour 433

6.5.6.2.9 Void 434

6.5.6.2.10 Type: LocationArea 434

6.5.6.2.11 Type: NetworkAreaInfo 434

6.5.6.2.12 Type: EcRestriction 434

6.5.6.2.13 Type: PlmnEcInfo 435

6.5.6.2.14 Type: PpDlPacketCountExt 435

6.5.6.2.15 Type: PpMaximumResponseTime 436

6.5.6.2.16 Type: PpMaximumLatency 436

6.5.6.2.17 Type: LcsPrivacy 437

6.5.6.2.18 Type: UmtTime 437

6.5.6.2.19 Type: PpDataEntry 438

6.5.6.2.20 Type: CommunicationCharacteristicsAF 439

6.5.6.2.21 Type: EcsAddrConfigInfo 439

6.5.6.2.22 Type: 5MbsAuthorizationInfo 439

6.5.6.2.23 Type: MulticastMbsGroupMemb 440

6.5.6.2.24 Type: DnnSnssaiSpecificGroup 440

6.5.6.2.25 Type: AfReqDefaultQoS 440

6.5.6.2.26 Type: ExpectedUeBehaviourExtension 441

6.5.6.2.27 Type: MbsAssistanceInfo 441

6.5.6.2.28 Type: AppSpecificExpectedUeBehaviour 441

6.5.6.2.29 Type: MaxGroupDataRate 442

6.5.6.2.30 Type: 5GVnGroupConfigurationModification 442

6.5.6.2.31 Type: 5GVnGroupDataModification 443

6.5.6.2.32 Type: RangingSlPrivacy 444

6.5.6.2.33 Type: SupportedPlmn 444

6.5.6.3 Simple data types and enumerations 444

6.5.6.3.1 Introduction 444

6.5.6.3.2 Simple data types 444

6.5.6.3.3 Void 445

6.5.6.3.4 Void 445

6.5.6.3.5 Enumeration: 5GVnGroupCommunicationType 445

6.5.6.3.6 Enumeration: EcsAuthMethod 445

6.5.7 Error Handling 445

6.5.7.1 General 445

6.5.7.2 Protocol Errors 445

6.5.7.3 Application Errors 445

6.5.8 Feature Negotiation 446

6.5.9 Security 446

6.5.10 HTTP redirection 446

6.6 Nudm_NIDDAuthorization Service API 447

6.6.1 API URI 447

6.6.2 Usage of HTTP 447

6.6.2.1 General 447

6.6.2.2 HTTP standard headers 447

6.6.2.2.1 General 447

6.6.2.2.2 Content type 447

6.6.2.3 HTTP custom headers 448

6.6.2.3.1 General 448

6.6.3 Resources 448

6.6.3.1 Overview 448

6.6.3.2 Resource: ueIdentity (Document) 448

6.6.3.2.1 Description 448

6.6.3.2.2 Resource Definition 448

6.6.3.2.3 Resource Standard Methods 449

6.6.3.2.4 Resource Custom Operations 449

6.6.4 Custom Operations without associated resources 450

6.6.5 Notifications 450

6.6.5.1 General 450

6.6.5.2 Nidd Authorization Data Update Notification 450

6.6.6 Data Model 451

6.6.6.1 General 451

6.6.6.2 Structured data types 451

6.6.6.2.1 Introduction 451

6.6.6.2.2 Type: AuthorizationData 451

6.6.6.2.3 Type: UserIdentifier 452

6.6.6.2.4 Type: NiddAuthUpdateInfo 452

6.6.6.2.5 Type: NiddAuthUpdateNotification 452

6.6.6.2.6 Type: AuthorizationInfo 453

6.6.6.3 Simple data types and enumerations 453

6.6.6.3.1 Introduction 453

6.6.6.3.2 Simple data types 453

6.6.6.3.3 Enumeration: NiddCause 454

6.6.7 Error Handling 454

6.6.7.1 General 454

6.6.7.2 Protocol Errors 454

6.6.7.3 Application Errors 454

6.6.8 Feature Negotiation 454

6.6.9 Security 454

6.6.10 HTTP redirection 455

6.7 Nudm_MT Service API 455

6.7.1 API URI 455

6.7.2 Usage of HTTP 455

6.7.2.1 General 455

6.7.2.2 HTTP standard headers 455

6.7.2.2.1 General 455

6.7.2.2.2 Content type 455

6.7.2.3 HTTP custom headers 456

6.7.2.3.1 General 456

6.7.3 Resources 456

6.7.3.1 Overview 456

6.7.3.2 Resource: UeInfo 456

6.7.3.2.1 Description 456

6.7.3.2.2 Resource Definition 456

6.7.3.2.3 Resource Standard Methods 457

6.7.3.3 Resource: LocationInfo 457

6.7.3.3.1 Description 457

6.7.3.3.2 Resource Definition 457

6.7.3.3.3 Resource Standard Methods 458

6.7.3.3.4 Resource Custom Operations 458

6.7.4 Custom Operations without associated resources 458

6.7.5 Notifications 459

6.7.6 Data Model 459

6.7.6.1 General 459

6.7.6.2 Structured data types 459

6.7.6.2.1 Introduction 459

6.7.6.2.2 Type: UeInfo 460

6.7.6.2.3 Type: LocationInfoRequest 460

6.7.6.2.4 Type: LocationInfoResult 461

6.7.6.2.5 Type: 5GSrvccInfo 461

6.7.6.3 Simple data types and enumerations 461

6.7.7 Error Handling 461

6.7.7.1 General 461

6.7.7.2 Protocol Errors 461

6.7.7.3 Application Errors 461

6.7.8 Feature Negotiation 462

6.7.9 Security 462

6.7.10 HTTP redirection 462

6.8 Nudm_ServiceSpecificAuthorization Service API 462

6.8.1 API URI 462

6.8.2 Usage of HTTP 463

6.8.2.1 General 463

6.8.2.2 HTTP standard headers 463

6.8.2.2.1 General 463

6.8.2.2.2 Content type 463

6.8.2.3 HTTP custom headers 463

6.8.2.3.1 General 463

6.8.3 Resources 463

6.8.3.1 Overview 463

6.8.3.2 Resource: ServiceType (Document) 464

6.8.3.2.1 Description 464

6.8.3.2.2 Resource Definition 464

6.8.3.2.3 Resource Standard Methods 464

6.8.3.2.4 Resource Custom Operations 465

6.8.4 Custom Operations without associated resources 466

6.8.5 Notifications 466

6.8.5.1 General 466

6.8.5.2 Service Specific Data Update Notification 466

6.8.6 Data Model 467

6.8.6.1 General 467

6.8.6.2 Structured data types 468

6.8.6.2.1 Introduction 468

6.8.6.2.2 Type: AuthUpdateNotification 468

6.8.6.2.3 Type: AuthUpdateInfo 468

6.8.6.2.4 Type: ServiceSpecificAuthorizationInfo 469

6.8.6.2.5 Type: ServicepecificAuthorizationData 469

6.8.6.2.6 Type: AuthorizationUeId 470

6.8.6.2.7 Type: ServicepecificAuthorizationRemoveData 470

6.8.6.3 Simple data types and enumerations 470

6.8.6.3.1 Introduction 470

6.8.6.3.2 Enumeration: ServiceType 470

6.8.6.3.3 Enumeration: InvalidCause 470

6.8.7 Error Handling 470

6.8.7.1 General 470

6.8.7.2 Protocol Errors 470

6.8.7.3 Application Errors 471

6.8.8 Feature Negotiation 471

6.8.9 Security 471

6.8.10 HTTP redirection 471

6.9 Nudm_ReportSMDeliveryStatus Service API 472

6.9.1 API URI 472

6.9.2 Usage of HTTP 472

6.9.2.1 General 472

6.9.2.2 HTTP standard headers 472

6.9.2.2.1 General 472

6.9.2.2.2 Content type 472

6.9.2.3 HTTP custom headers 472

6.9.2.3.1 General 472

6.9.3 Resources 473

6.9.3.1 Overview 473

6.9.3.2 Resource: SmDeliveryStatus (Document) 473

6.9.3.2.1 Description 473

6.9.3.2.2 Resource Definition 473

6.9.3.2.3 Resource Standard Methods 473

6.9.3.2.4 Resource Custom Operations 474

6.9.4 Custom Operations without associated resources 474

6.9.5 Notifications 474

6.9.6 Data Model 474

6.9.6.1 General 474

6.9.6.2 Structured data types 475

6.9.6.2.1 Introduction 475

6.9.6.2.2 Type: SmDeliveryStatus 475

6.9.6.3 Simple data types and enumerations 475

6.9.6.3.1 Introduction 475

6.9.7 Error Handling 475

6.9.7.1 General 475

6.9.7.2 Protocol Errors 475

6.9.7.3 Application Errors 475

6.9.8 Feature Negotiation 476

6.9.9 Security 476

6.9.10 HTTP redirection 476

6.10 Nudm_UEIdentifier Service API 476

6.10.1 API URI 476

6.10.2 Usage of HTTP 477

6.10.2.1 General 477

6.10.2.2 HTTP standard headers 477

6.10.2.2.1 General 477

6.10.2.2.2 Content type 477

6.10.2.3 HTTP custom headers 477

6.10.2.3.1 General 477

6.10.3 Resources 477

6.10.3.1 Overview 477

6.10.4 Custom Operations without associated resources 478

6.10.4.1 Overview 478

6.10.4.2 Operation: Deconceal 478

6.10.4.2.1 Description 478

6.10.4.2.2 Operation Definition 478

6.10.5 Notifications 479

6.10.6 Data Model 479

6.10.6.1 General 479

6.10.6.2 Structured data types 479

6.10.6.2.1 Introduction 479

6.10.6.2.2 Type: DeconcealReqData 480

6.10.6.2.3 Type: DeconcealRspData 480

6.10.6.3 Simple data types and enumerations 480

6.10.6.3.1 Introduction 480

6.10.7 Error Handling 480

6.10.7.1 General 480

6.10.7.2 Protocol Errors 480

6.10.7.3 Application Errors 480

6.10.8 Feature Negotiation 480

6.10.9 Security 481

Annex A (normative): OpenAPI specification 481

A.1 General 481

A.2 Nudm_SDM API 481

A.3 Nudm_UECM API 558

A.4 Nudm_UEAU API 605

A.5 Nudm_EE API 618

A.6 Nudm_PP API 633

A.7 Nudm_NIDDAU API 650

A.8 Nudm_MT API 653

A.9 Nudm_SSAU API 656

A.10 Nudm_ReportSMDeliveryStatus API 661

A.11 Nudm_UEIdentifier API 662

Annex B (informative): Stateless UDMs 663

Annex C (informative): SUCI encoding 667

Annex D (informative): Change history 671
