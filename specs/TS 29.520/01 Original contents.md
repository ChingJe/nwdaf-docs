---
spec: TS 29.520
version: 18.14.0
release: '18'
clause: contents
title: Contents
source_archive: 29520-ie0.zip
source_document: 29520-ie0.doc
source_archive_sha256: 4937d4ffcdc8760cac3dd990b10d5c0beb722f9eab26d94ce4b1f296287057c1
source_document_sha256: 9c9beb10bbac50abf869d1cc575a55184a8a97008a072a839b3da2ebe4aaeb69
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Contents


Foreword 15

1 Scope 16

2 References 16

3 Definitions and abbreviations 17

3.1 Definitions 17

3.2 Abbreviations 17

4 Services offered by the NWDAF 18

4.1 Introduction 18

4.2 Nnwdaf_EventsSubscription Service 21

4.2.1 Service Description 21

4.2.1.1 Overview 21

4.2.1.2 Service Architecture 22

4.2.1.3 Network Functions 23

4.2.1.3.1 Network Data Analytics Function (NWDAF) 23

4.2.1.3.2 NF Service Consumers 23

4.2.2 Service Operations 25

4.2.2.1 Introduction 25

4.2.2.2 Nnwdaf_EventsSubscription_Subscribe service operation 26

4.2.2.2.1 General 26

4.2.2.2.2 Subscription for event notifications 26

4.2.2.2.3 Update subscription for event notifications 35

4.2.2.3 Nnwdaf_EventsSubscription_Unsubscribe service operation 37

4.2.2.3.1 General 37

4.2.2.3.2 Unsubscribe from event notifications 37

4.2.2.4 Nnwdaf_EventsSubscription_Notify service operation 38

4.2.2.4.1 General 38

4.2.2.4.2 Notification about subscribed event 38

4.2.2.5 Nnwdaf_EventsSubscription_Transfer service operation 40

4.2.2.5.1 General 40

4.2.2.5.2 Creation of request for analytics subscription transfer 40

4.2.2.5.3 Update a request for analytics subscription transfer 42

4.2.2.5.4 Cancel a request for analytics subscription transfer 43

4.3 Nnwdaf_AnalyticsInfo Service 44

4.3.1 Service Description 44

4.3.1.1 Overview 44

4.3.1.2 Service Architecture 44

4.3.1.3 Network Functions 45

4.3.1.3.1 Network Data Analytics Function (NWDAF) 45

4.3.1.3.2 NF Service Consumers 45

4.3.2 Service Operations 47

4.3.2.1 Introduction 47

4.3.2.2 Nnwdaf_AnalyticsInfo_Request service operation 47

4.3.2.2.1 General 47

4.3.2.2.2 Request and get from NWDAF Analytics information 47

4.3.2.3 Nnwdaf_AnalyticsInfo_ContextTransfer service operation 56

4.3.2.3.1 General 56

4.3.2.3.2 Request and get from NWDAF context of a subscription 56

4.4 Nnwdaf_DataManagement Service 58

4.4.1 Service Description 58

4.4.1.1 Overview 58

4.4.1.2 Service Architecture 58

4.4.1.3 Network Functions 59

4.4.1.3.1 Network Data Analytics Function (NWDAF) 59

4.4.1.3.2 NF Service Consumers 59

4.4.2 Service Operations 59

4.4.2.1 Introduction 59

4.4.2.2 Nnwdaf_DataManagement_Subscribe service operation 59

4.4.2.2.1 General 59

4.4.2.2.2 Subscription for data notifications 60

4.4.2.2.3 Update subscription for data notifications 62

4.4.2.3 Nnwdaf_DataManagement_Unsubscribe service operation 63

4.4.2.3.1 General 63

4.4.2.3.2 Unsubscribe from data notifications 64

4.4.2.4 Nnwdaf_DataManagement_Notify service operation 64

4.4.2.4.1 General 64

4.4.2.4.2 Notification about subscribed data 64

4.4.2.5 Nnwdaf_DataManagement_Fetch service operation 66

4.4.2.5.1 General 66

4.4.2.5.2 Retrieve data from the NWDAF 66

4.5 Nnwdaf_MLModelProvision Service 66

4.5.1 Service Description 66

4.5.1.1 Overview 66

4.5.1.2 Service Architecture 67

4.5.1.3 Network Functions 67

4.5.1.3.1 Network Data Analytics Function (NWDAF) 67

4.5.1.3.2 NF Service Consumers 68

4.5.2 Service Operations 68

4.5.2.1 Introduction 68

4.5.2.2 Nnwdaf_MLModelProvision_Subscribe service operation 68

4.5.2.2.1 General 68

4.5.2.2.2 Subscription for event notifications 68

4.5.2.2.3 Update subscription for event notifications 70

4.5.2.3 Nnwdaf_MLModelProvision_Unsubscribe service operation 71

4.5.2.3.1 General 71

4.5.2.3.2 Unsubscribe from event notifications 71

4.5.2.4 Nnwdaf_MLModelProvision_Notify service operation 71

4.5.2.4.1 General 71

4.5.2.4.2 Notification about subscribed event 71

4.6 Nnwdaf_MLModelTraining Service 72

4.6.1 Service Description 72

4.6.1.1 Overview 72

4.6.1.2 Service Architecture 73

4.6.1.3 Network Functions 73

4.6.1.3.1 Network Data Analytics Function (NWDAF) 73

4.6.1.3.2 NF Service Consumers 73

4.6.2 Service Operations 74

4.6.2.1 Introduction 74

4.6.2.2 Nnwdaf_MLModelTraining_Subscribe service operation 74

4.6.2.2.1 General 74

4.6.2.2.2 Subscription for event notifications 74

4.6.2.2.3 Update subscription for event notifications 75

4.6.2.2.4 Partial update subscription for event notifications 77

4.6.2.3 Nnwdaf_MLModelTraining_Unsubscribe service operation 78

4.6.2.3.1 General 78

4.6.2.3.2 Unsubscribe from event notifications 78

4.6.2.4 Nnwdaf_MLModelTraining_Notify service operation 78

4.6.2.4.1 General 78

4.6.2.4.2 Notification about subscribed event 78

4.7 Nnwdaf_MLModelMonitor Service 79

4.7.1 Service Description 79

4.7.1.1 Overview 79

4.7.1.2 Service Architecture 80

4.7.1.3 Network Functions 80

4.7.1.3.1 Network Data Analytics Function (NWDAF) 80

4.7.1.3.2 NF Service Consumers 81

4.7.2 Service Operations 81

4.7.2.1 Introduction 81

4.7.2.2 Nnwdaf_MLModelMonitor_Register service operation 81

4.7.2.2.1 General 81

4.7.2.2.2 Registering the monitoring of the analytics accuracy of an ML Model 81

4.7.2.3 Nnwdaf_MLModelMonitor_Deregister service operation 82

4.7.2.3.1 General 82

4.7.2.3.2 Deregistering the monitoring of the analytics accuracy of an ML Model 83

4.7.2.4 Nnwdaf_MLModelMonitor_Subscribe service operation 83

4.7.2.4.1 General 83

4.7.2.4.2 Subscription for monitoring notifications 83

4.7.2.4.3 Update of subscription for monitoring notifications 84

4.7.2.5 Nnwdaf_MLModelMonitor_Unsubscribe service operation 85

4.7.2.5.1 General 85

4.7.2.5.2 Unsubscribe from monitoring notifications 85

4.7.2.6 Nnwdaf_MLModelMonitor_Notify service operation 86

4.7.2.6.1 General 86

4.7.2.6.2 Notification about subscribed event 86

4.8 Nnwdaf_RoamingData Service 87

4.8.1 Service Description 87

4.8.1.1 Overview 87

4.8.1.2 Service Architecture 87

4.8.1.3 Network Functions 88

4.8.1.3.1 Network Data Analytics Function (NWDAF) 88

4.8.1.3.2 NF Service Consumers 88

4.8.2 Service Operations 88

4.8.2.1 Introduction 88

4.8.2.2 Nnwdaf_RoamingData_Subscribe service operation 89

4.8.2.2.1 General 89

4.8.2.2.2 Subscription for event notifications 89

4.8.2.2.3 Update of subscription for event notifications 90

4.8.2.3 Nnwdaf_RoamingData_Unsubscribe service operation 91

4.8.2.3.1 General 91

4.8.2.3.2 Unsubscribe from event notifications 91

4.8.2.4 Nnwdaf_RoamingData_Notify service operation 91

4.8.2.4.1 General 91

4.8.2.4.2 Notification about subscribed event 91

4.9.1.2 Service Architecture 92

4.9.1.3 Network Functions 93

4.9.1.3.1 Network Data Analytics Function (NWDAF) 93

4.9.1.3.2 NF Service Consumers 93

4.9.2 Service Operations 94

4.9.2.1 Introduction 94

4.9.2.2 Nnwdaf_RoamingAnalytics_Subscribe service operation 94

4.9.2.2.1 General 94

4.9.2.2.2 Subscription for event notifications 94

4.9.2.4 Nnwdaf_RoamingAnalytics_Notify service operation 98

4.9.2.4.1 General 98

4.9.2.4.2 Notification about subscribed event 98

5 API Definitions 99

5.1 Nnwdaf_EventsSubscription Service API 99

5.1.1 Introduction 99

5.1.2 Usage of HTTP 99

5.1.2.1 General 99

5.1.2.2 HTTP standard headers 99

5.1.2.2.1 General 99

5.1.2.2.2 Content type 99

5.1.2.3 HTTP custom headers 100

5.1.3 Resources 100

5.1.3.1 Resource Structure 100

5.1.3.2 Resource: NWDAF Events Subscriptions 101

5.1.3.2.1 Description 101

5.1.3.2.2 Resource definition 101

5.1.3.2.3 Resource Standard Methods 101

5.1.3.2.4 Resource Custom Operations 102

5.1.3.3 Resource: Individual NWDAF Event Subscription 102

5.1.3.3.1 Description 102

5.1.3.3.2 Resource definition 102

5.1.3.3.3 Resource Standard Methods 103

5.1.3.3.4 Resource Custom Operations 105

5.1.3.4 Resource: NWDAF Event Subscription Transfers 105

5.1.3.4.1 Description 105

5.1.3.4.2 Resource definition 105

5.1.3.4.3 Resource Standard Methods 105

5.1.3.4.4 Resource Custom Operations 106

5.1.3.5 Resource: Individual NWDAF Event Subscription Transfer 106

5.1.3.5.1 Description 106

5.1.3.5.2 Resource definition 106

5.1.3.5.3 Resource Standard Methods 107

5.1.3.5.4 Resource Custom Operations 109

5.1.4 Custom Operations without associated resources 109

5.1.5 Notifications 109

5.1.5.1 General 109

5.1.5.2 Event Notification 109

5.1.5.2.1 Description 109

5.1.5.2.2 Operation Definition 110

5.1.6 Data Model 111

5.1.6.1 General 111

5.1.6.2 Structured data types 121

5.1.6.2.1 Introduction 121

5.1.6.2.2 Type NnwdafEventsSubscription 122

5.1.6.2.3 Type EventSubscription 124

5.1.6.2.4 Type NnwdafEventsSubscriptionNotification 131

5.1.6.2.5 Type EventNotification 132

5.1.6.2.6 Type SliceLoadLevelInformation 135

5.1.6.2.7 Type EventReportingRequirement 136

5.1.6.2.8 Type TargetUeInformation 138

5.1.6.2.9 Void 139

5.1.6.2.10 Type UeMobility 139

5.1.6.2.11 Type LocationInfo 140

5.1.6.2.12 Void 141

5.1.6.2.13 Type UeCommunication 141

5.1.6.2.14 Type TrafficCharacterization 143

5.1.6.2.15 Type AbnormalBehaviour 144

5.1.6.2.16 Type Exception 144

5.1.6.2.17 Type UserDataCongestionInfo 145

5.1.6.2.18 Type CongestionInfo 145

5.1.6.2.19 Type QosSustainabilityInfo 146

5.1.6.2.20 Type QosRequirement 147

5.1.6.2.21 Type RetainabilityThreshold 147

5.1.6.2.22 Type NetworkPerfRequirement 148

5.1.6.2.23 Type NetworkPerfInfo 149

5.1.6.2.24 Type ServiceExperienceInfo 150

5.1.6.2.25 Type BwRequirement 152

5.1.6.2.26 Type AdditionalMeasurement 153

5.1.6.2.27 Type IpEthFlowDescription 153

5.1.6.2.28 Type AddressList 154

5.1.6.2.29 Type CircumstanceDescription 154

5.1.6.2.30 Type ThresholdLevel 155

5.1.6.2.31 Type NfLoadLevelInformation 157

5.1.6.2.32 Type NfStatus 157

5.1.6.2.33 Type NsiIdInfo 158

5.1.6.2.34 Type NsiLoadLevelInfo 159

5.1.6.2.35 Type FailureEventInfo 160

5.1.6.2.36 Type AnalyticsMetadataIndication 160

5.1.6.2.37 Type AnalyticsMetadataInfo 161

5.1.6.2.38 Type NumberAverage 161

5.1.6.2.39 Type TopApplication 161

5.1.6.2.40 Type AnalyticsSubscriptionsTransfer 161

5.1.6.2.41 Type SubscriptionTransferInfo 162

5.1.6.2.42 Type ModelInfo 162

5.1.6.2.43 Type AnalyticsContextIdentifier 162

5.1.6.2.44 Type UeAnalyticsContextDescriptor 163

5.1.6.2.45 Type DnPerfInfo 163

5.1.6.2.46 Type DnPerf 164

5.1.6.2.47 Type PerfData 165

5.1.6.2.48 Type ResourceUsage 166

5.1.6.2.49 Type ConsumerNfInformation 166

5.1.6.2.50 Type DispersionRequirement 166

5.1.6.2.51 Type ClassCriterion 167

5.1.6.2.52 Type RankingCriterion 167

5.1.6.2.53 Type DispersionInfo 167

5.1.6.2.54 Type DispersionCollection 168

5.1.6.2.55 Type ApplicationVolume 169

5.1.6.2.56 Type RedundantTransmissionExpReq 169

5.1.6.2.57 Type RedundantTransmissionExpInfo 170

5.1.6.2.58 Type RedundantTransmissionExpPerTS 170

5.1.6.2.59 Type WlanPerformanceReq 171

5.1.6.2.60 Type WlanPerformanceInfo 171

5.1.6.2.61 Type WlanPerSsIdPerformanceInfo 171

5.1.6.2.62 Type WlanPerTsPerformanceInfo 172

5.1.6.2.63 Type TrafficInformation 172

5.1.6.2.64 Type AppListForUeComm 173

5.1.6.2.65 Type SessInactTimerForUeComm 173

5.1.6.2.66 Type DnPerformanceReq 173

5.1.6.2.67 Type: RatFreqInformation 174

5.1.6.2.68 Type PrevSubInfo 174

5.1.6.2.69 Type MLModelInfo 175

5.1.6.2.70 Type ObservedRedundantTransExp 176

5.1.6.2.71 Type UeMobilityReq 178

5.1.6.2.72 Type UeCommReq 178

5.1.6.2.73 Type PfdDeterminationInfo 178

5.1.6.2.74 Type PduSessionInfo 179

5.1.6.2.75 Type DirectionInfo 179

5.1.6.2.76 Type GeoDistributionInfo 179

5.1.6.2.77 Type PduSesTrafficInfo 180

5.1.6.2.78 Type TdTraffic 180

5.1.6.2.79 Type PduSesTrafficReq 181

5.1.6.2.80 Type WlanPerUeIdPerformanceInfo 181

5.1.6.2.81 Type ResourceUsageRequirement 181

5.1.6.2.82 Type E2eDataVolTransTimeReq 182

5.1.6.2.83 Type E2eDataVolTransTimeInfo 183

5.1.6.2.84 Type E2eDataVolTransTimePerTS 183

5.1.6.2.85 Type DataVolume 183

5.1.6.2.86 Type E2eDataVolTransTimePerUe 184

5.1.6.2.87 Type E2eDataVolTransTimeUeList 185

5.1.6.2.88 Type AccuracyReq 186

5.1.6.2.89 Type AccuracyInfo 186

5.1.6.2.90 Type DataVolumeTransferTime 187

5.1.6.2.91 Type MovBehavReq 187

5.1.6.2.92 Type MovBehavInfo 187

5.1.6.2.93 Type MovBehav 188

5.1.6.2.94 Type SpeedThresholdInfo 188

5.1.6.2.95 Type GeoLocation 189

5.1.6.2.96 Type LocAccuracyReq 190

5.1.6.2.97 Type LocAccuracyInfo 191

5.1.6.2.98 Type LocAccuracyPerMethod 192

5.1.6.2.99 Type RelProxReq 192

5.1.6.2.100 Type RelProxInfo 193

5.1.6.2.101 Type UeProximity 194

5.1.6.2.102 Type UeTrajectory 194

5.1.6.2.103 Type TimestampedLocation 194

5.1.6.2.104 Type TimeToCollisionInfo 195

5.1.6.2.105 Type AnalyticsFeedbackInfo 195

5.1.6.2.106 Type RoamingInfo 196

5.1.6.2.107 Type SuggestedPfdInfo 197

5.1.6.3 Simple data types and enumerations 197

5.1.6.3.1 Introduction 197

5.1.6.3.2 Simple data types 197

5.1.6.3.3 Enumeration: NotificationMethod 198

5.1.6.3.4 Enumeration: NwdafEvent 199

5.1.6.3.5 Enumeration: Accuracy 199

5.1.6.3.6 Enumeration: ExceptionId 200

5.1.6.3.7 Enumeration: ExceptionTrend 200

5.1.6.3.8 Enumeration: CongestionType 200

5.1.6.3.9 Enumeration: TimeUnit 200

5.1.6.3.10 Enumeration: NetworkPerfType 201

5.1.6.3.11 Enumeration: ExpectedAnalyticsType 201

5.1.6.3.12 Enumeration: MatchingDirection 201

5.1.6.3.14 Enumeration: AnalyticsMetadata 202

5.1.6.3.15 Enumeration: DatasetStatisticalProperty 202

5.1.6.3.16 Enumeration: OutputStrategy 203

5.1.6.3.17 Enumeration: TransferRequestType 203

5.1.6.3.18 Enumeration: AnalyticsSubset 204

5.1.6.3.19 Enumeration: DispersionType 206

5.1.6.3.20 Enumeration: DispersionClass 207

5.1.6.3.21 Enumeration: DispersionOrderingCriterion 207

5.1.6.3.22 Enumeration: RedTransExpOrderingCriterion 207

5.1.6.3.23 Enumeration: WlanOrderingCriterion 207

5.1.6.3.24 Enumeration: ServiceExperienceType 208

5.1.6.3.25 Enumeration: DnPerfOrderingCriterion 208

5.1.6.3.26 Enumeration: TermCause 208

5.1.6.3.27 Enumeration: UserDataConOrderCrit 208

5.1.6.3.28 Enumeration: UeMobilityOrderCriterion 208

5.1.6.3.29 Enumeration: UeCommOrderCriterion 209

5.1.6.3.30 Enumeration: NetworkPerfOrderCriterion 209

5.1.6.3.31 Enumeration: DeviceType 209

5.1.6.3.32 Enumeration: LocInfoGranularity 209

5.1.6.3.33 Enumeration: TrafficDirection 209

5.1.6.3.34 Enumeration: ValueExpression 210

5.1.6.3.35 Enumeration: E2eDataVolTransTimeCriterion 210

5.1.6.3.36 Void 210

5.1.6.3.37 Enumeration: AnalyticsAccuracyIndication 210

5.1.6.3.38 Enumeration: LocationOrientation 210

5.1.6.3.39 Enumeration: Direction 210

5.1.6.3.40 Enumeration: ProximityCriterion 211

5.1.7 Error handling 211

5.1.7.1 General 211

5.1.7.2 Protocol Errors 211

5.1.8 Feature negotiation 212

5.1.9 Security 215

5.2 Nnwdaf_AnalyticsInfo Service API 216

5.2.1 Introduction 216

5.2.2 Usage of HTTP 216

5.2.2.1 General 216

5.2.2.2 HTTP standard headers 216

5.2.2.2.1 General 216

5.2.2.2.2 Content type 216

5.2.2.3 HTTP custom headers 217

5.2.3 Resources 217

5.2.3.1 Resource Structure 217

5.2.3.2 Resource: NWDAF Analytics 217

5.2.3.2.1 Description 217

5.2.3.2.2 Resource definition 217

5.2.3.2.3 Resource Standard Methods 218

5.2.3.2.4 Resource Custom Operations 218

5.2.3.3 Resource: NWDAF Context 219

5.2.3.3.1 Description 219

5.2.3.3.2 Resource definition 219

5.2.3.3.3 Resource Standard Methods 219

5.2.4 Custom Operations without associated resources 220

5.2.5 Notifications 220

5.2.6 Data Model 220

5.2.6.1 General 220

5.2.6.2 Structured data types 226

5.2.6.2.1 Introduction 226

5.2.6.2.2 Type AnalyticsData 227

5.2.6.2.3 Type EventFilter 230

5.2.6.2.4 Void 235

5.2.6.2.5 Type AdditionInfoAnalyticsInfoRequest 235

5.2.6.2.6 Type ContextData 235

5.2.6.2.7 Type ContextElement 236

5.2.6.2.8 Type ContextIdList 237

5.2.6.2.9 Type HistoricalData 237

5.2.6.2.10 Type SpecificAnalyticsSubscription 238

5.2.6.2.11 Type RequestedContext 238

5.2.6.2.12 Type SmcceInfo 238

5.2.6.2.13 Type SmcceUeList 239

5.2.6.2.14 Type SpecificDataSubscription 239

5.2.6.2.15 Type UserDataCongestReq 240

5.2.6.2.16 Type NetworkPerfReq 240

5.2.6.2.17 Type ResourceUsageRequPerNwPerfType 240

5.2.6.2.18 Type AnalyticsAccuracyInfo 241

5.2.6.2.19 Type GroundTruthInfo 241

5.2.6.2.20 Type MlModelAccuracyInfo 241

5.2.6.3 Simple data types and enumerations 242

5.2.6.3.1 Introduction 242

5.2.6.3.2 Simple data types 242

5.2.6.3.3 Enumeration: EventId 243

5.2.6.3.4 Enumeration: ContextType 244

5.2.6.3.5 Enumeration: AdrfDataType 244

5.2.6.4 Data types describing alternative data types or combinations of data types 244

5.2.6.4.1 Type ProblemDetailsAnalyticsInfoRequest 244

5.2.7 Error handling 244

5.2.7.1 General 244

5.2.7.2 Protocol Errors 245

5.2.8 Feature negotiation 245

5.2.9 Security 248

5.3 Nnwdaf_DataManagement Service API 248

5.3.1 Introduction 248

5.3.2 Usage of HTTP 249

5.3.2.1 General 249

5.3.2.2 HTTP standard headers 249

5.3.2.2.1 General 249

5.3.2.2.2 Content type 249

5.3.2.3 HTTP custom headers 249

5.3.3 Resources 249

5.3.3.1 Resource Structure 249

5.3.3.2 Resource: NWDAF Data Management Subscriptions 250

5.3.3.2.1 Description 250

5.3.3.2.2 Resource Definition 250

5.3.3.2.3 Resource Standard Methods 251

5.3.3.2.4 Resource Custom Operations 251

5.3.3.3 Resource: Individual NWDAF Data Management Subscription 251

5.3.3.3.1 Description 251

5.3.3.3.2 Resource definition 251

5.3.3.3.3 Resource Standard Methods 252

5.3.3.3.4 Resource Custom Operations 254

5.3.4 Custom Operations without associated resources 254

5.3.5 Notifications 255

5.3.5.1 General 255

5.3.5.2 Event Notification 255

5.3.5.2.1 Description 255

5.3.5.2.2 Operation Definition 255

5.3.5.3 Fetch Notification 256

5.3.5.3.1 Description 256

5.3.5.3.2 Target URI 256

5.3.5.3.3 Standard Methods 256

5.3.6 Data Model 257

5.3.6.1 General 257

5.3.6.2 Structured data types 259

5.3.6.2.1 Introduction 259

5.3.6.2.2 Type NnwdafDataManagementSubsc 260

5.3.6.2.3 Type NnwdafDataManagementNotif 263

5.3.6.3 Simple data types and enumerations 263

5.3.6.3.1 Introduction 263

5.3.6.3.2 Simple data types 264

5.3.6.3.3 Enumeration: PendingNotificationCause 264

5.3.7 Error handling 264

5.3.7.1 General 264

5.3.7.2 Protocol Errors 264

5.3.7.3 Application Errors 264

5.3.8 Feature negotiation 265

5.3.9 Security 265

5.4 Nnwdaf_MLModelProvision Service API 265

5.4.1 Introduction 265

5.4.2 Usage of HTTP 266

5.4.2.1 General 266

5.4.2.2 HTTP standard headers 266

5.4.2.2.1 General 266

5.4.2.2.2 Content type 266

5.4.2.3 HTTP custom headers 266

5.4.3 Resources 266

5.4.3.1 Resource Structure 266

5.4.3.2 Resource: NWDAF ML Model Provision Subscriptions 267

5.4.3.2.1 Description 267

5.4.3.2.2 Resource definition 267

5.4.3.2.3 Resource Standard Methods 267

5.4.3.2.4 Resource Custom Operations 268

5.4.3.3 Resource: Individual NWDAF ML Model Provision Subscription 268

5.4.3.3.1 Description 268

5.4.3.3.2 Resource definition 268

5.4.3.3.3 Resource Standard Methods 268

5.4.3.3.4 Resource Custom Operations 271

5.4.4 Custom Operations without associated resources 271

5.4.5 Notifications 271

5.4.5.1 General 271

5.4.5.2 Event Notification 271

5.4.5.2.1 Description 271

5.4.5.2.2 Operation Definition 272

5.4.6 Data Model 273

5.4.6.1 General 273

5.4.6.2 Structured data types 274

5.4.6.2.1 Introduction 274

5.4.6.2.2 Type NwdafMLModelProvSubsc 275

5.4.6.2.3 Type MLEventSubscription 276

5.4.6.2.4 Void 277

5.4.6.2.5 Type NwdafMLModelProvNotif 277

5.4.6.2.6 Type MLEventNotif 278

5.4.6.2.7 Type FailureEventInfoForMLModel 279

5.4.6.2.8 Type MLModelAddr 279

5.4.6.2.9 Void 279

5.4.6.2.10 Void 279

5.4.6.2.11 Type MLRepEventCondition 279

5.4.6.2.12 Type InputDataInfo 280

5.4.6.2.13 Type ModelProvisionParamsExt 281

5.4.6.2.14 Type AdditionalMLModelInformation 282

5.4.6.2.15 Type MLModelAdrf 282

5.4.6.2.16 Type TrainInputDataInfo 283

5.4.6.2.17 Type InferenceDataForModelTrain 283

5.4.6.3 Simple data types and enumerations 283

5.4.6.3.1 Introduction 283

5.4.6.3.2 Simple data types 283

5.4.6.3.3 Enumeration: FailureCode 284

5.4.6.3.4 Enumeration: MLModelMetric 284

5.4.7 Error handling 284

5.4.7.1 General 284

5.4.7.2 Protocol Errors 284

5.4.7.3 Application Errors 284

5.4.8 Feature negotiation 285

5.4.9 Security 285

5.5 Nnwdaf_MLModelTraining Service API 285

5.5.1 Introduction 285

5.5.2 Usage of HTTP 286

5.5.2.1 General 286

5.5.2.2 HTTP standard headers 286

5.5.2.2.1 General 286

5.5.2.2.2 Content type 286

5.5.2.3 HTTP custom headers 286

5.5.3 Resources 286

5.5.3.1 Resource Structure 286

5.5.3.2 Resource: NWDAF ML Model Training Subscriptions 287

5.5.3.2.1 Description 287

5.5.3.2.2 Resource definition 287

5.5.3.2.3 Resource Standard Methods 287

5.5.3.2.4 Resource Custom Operations 288

5.5.3.3 Resource: Individual NWDAF ML Model Training Subscription 288

5.5.3.3.1 Description 288

5.5.3.3.2 Resource definition 288

5.5.3.3.3 Resource Standard Methods 288

5.5.3.3.4 Resource Custom Operations 291

5.5.4 Custom Operations without associated resources 291

5.5.5 Notifications 292

5.5.5.1 General 292

5.5.5.2 Event Notification 292

5.5.5.2.1 Description 292

5.5.5.2.2 Operation Definition 292

5.5.6 Data Model 293

5.5.6.1 General 293

5.5.6.2 Structured data types 295

5.5.6.2.1 Introduction 295

5.5.6.2.2 Type NwdafMLModelTrainSubsc 296

5.5.6.2.3 Type NwdafMLModelTrainSubscPatch 298

5.5.6.2.5 Type MLModelTrainInfo 299

5.5.6.2.6 Type MLTrainReportInfo 299

5.5.6.2.7 Type FailureEventInfoForMLModelTrain 299

5.5.6.2.8 Type NwdafMLModelTrainNotif 300

5.5.6.2.9 Void 301

5.5.6.2.10 Type DataAvReq 301

5.5.6.2.11 Type DelayEventNotif 301

5.5.6.2.12 Type StatusReportInfo 301

5.5.6.2.13 Type TrainDataInfo 302

5.5.6.3 Simple data types and enumerations 302

5.5.6.3.1 Introduction 302

5.5.6.3.2 Simple data types 302

5.5.6.3.3 Enumeration: FailureCodeTrain 302

5.5.6.3.4 Enumeration: TermTrainCause 303

5.5.6.3.5 Enumeration: DelayCause 303

5.5.7 Error handling 303

5.5.7.1 General 303

5.5.7.2 Protocol Errors 303

5.5.7.3 Application Errors 303

5.5.8 Feature negotiation 304

5.5.9 Security 304

5.6 Nnwdaf_MLModelMonitor Service API 304

5.6.1 Introduction 304

5.6.2 Usage of HTTP 305

5.6.2.1 General 305

5.6.2.2 HTTP standard headers 305

5.6.2.2.1 General 305

5.6.2.2.2 Content type 305

5.6.2.3 HTTP custom headers 305

5.6.3 Resources 305

5.6.3.1 Resource Structure 305

5.6.3.2 Resource: NWDAF ML model monitoring registrations 306

5.6.3.2.1 Description 306

5.6.3.2.2 Resource Definition 306

5.6.3.2.3 Resource Standard Methods 307

5.6.3.2.4 Resource Custom Operations 307

5.6.3.3 Resource: Individual NWDAF ML model monitoring registration 307

5.6.3.3.1 Description 307

5.6.3.3.2 Resource definition 308

5.6.3.3.3 Resource Standard Methods 308

5.6.3.3.4 Resource Custom Operations 309

5.6.3.4 Resource: NWDAF ML model monitoring Subscriptions 309

5.6.3.4.1 Description 309

5.6.3.4.2 Resource Definition 309

5.6.3.4.3 Resource Standard Methods 309

5.6.3.4.4 Resource Custom Operations 310

5.6.3.5 Resource: Individual NWDAF ML model monitoring Subscription 310

5.6.3.5.1 Description 310

5.6.3.5.2 Resource definition 310

5.6.3.5.3 Resource Standard Methods 311

5.6.3.5.4 Resource Custom Operations 313

5.6.4 Custom Operations without associated resources 313

5.6.5 Notifications 313

5.6.5.1 General 313

5.6.5.2 Event Notification 313

5.6.5.2.1 Description 313

5.6.5.2.2 Operation Definition 313

5.6.6 Data Model 315

5.6.6.1 General 315

5.6.6.2 Structured data types 315

5.6.6.2.1 Introduction 315

5.6.6.2.2 Type MLModelMonitorReg 316

5.6.6.2.3 Type MLModelMonitorSub 317

5.6.6.2.4 Type MLModelMonitorNotify 318

5.6.6.2.5 Type MLModelAccuracyInfo 319

5.6.6.2.6 Type AnalyticsFeedback 319

5.6.7 Error handling 320

5.6.7.1 General 320

5.6.7.2 Protocol Errors 320

5.6.7.3 Application Errors 320

5.6.8 Feature negotiation 320

5.6.9 Security 320

5.7 Nnwdaf_RoamingData Service API 320

5.7.1 Introduction 320

5.7.2 Usage of HTTP 321

5.7.2.1 General 321

5.7.2.2 HTTP standard headers 321

5.7.2.2.1 General 321

5.7.2.2.2 Content type 321

5.7.2.3 HTTP custom headers 321

5.7.3 Resources 321

5.7.3.1 Resource Structure 321

5.7.3.2 Resource: NWDAF Roaming Data Subscriptions 322

5.7.3.2.1 Description 322

5.7.3.2.2 Resource Definition 322

5.7.3.2.3 Resource Standard Methods 322

5.7.3.2.4 Resource Custom Operations 323

5.7.3.3 Resource: Individual NWDAF Roaming Data Subscription 323

5.7.3.3.1 Description 323

5.7.3.3.2 Resource definition 323

5.7.3.3.3 Resource Standard Methods 324

5.7.3.3.4 Resource Custom Operations 326

5.7.4 Custom Operations without associated resources 326

5.7.5 Notifications 326

5.7.5.1 General 326

5.7.5.2 Event Notification 326

5.7.5.2.1 Description 326

5.7.5.2.2 Operation Definition 326

5.7.6 Data Model 328

5.7.6.1 General 328

5.7.6.2 Structured data types 328

5.7.6.2.1 Introduction 328

5.7.6.2.2 Type RoamingDataSub 329

5.7.7 Error handling 330

5.7.7.1 General 330

5.7.7.2 Protocol Errors 330

5.7.8 Feature negotiation 330

5.7.9 Security 330

5.8 Nnwdaf_RoamingAnalytics Service API 331

5.8.1 Introduction 331

5.8.2 Usage of HTTP 331

5.8.2.1 General 331

5.8.2.2 HTTP standard headers 331

5.8.2.2.1 General 331

5.8.2.2.2 Content type 331

5.8.2.3 HTTP custom headers 331

5.8.3 Resources 332

5.8.3.1 Resource Structure 332

5.8.3.2 Resource: NWDAF Roaming Analytics Subscriptions 332

5.8.3.2.1 Description 332

5.8.3.2.2 Resource Definition 332

5.8.3.2.3 Resource Standard Methods 333

5.8.3.2.4 Resource Custom Operations 333

5.8.3.3 Resource: Individual NWDAF Roaming Analytics Subscription 334

5.8.3.3.1 Description 334

5.8.3.3.2 Resource definition 334

5.8.3.3.3 Resource Standard Methods 334

5.8.3.3.4 Resource Custom Operations 337

5.8.4 Custom Operations without associated resources 337

5.8.5 Notifications 337

5.8.5.1 General 337

5.8.5.2 Roaming Analytics Notification 337

5.8.5.2.1 Description 337

5.8.5.2.2 Operation Definition 337

5.8.6 Data Model 338

5.8.6.1 General 338

5.8.6.2 Structured data types 339

5.8.6.2.1 Introduction 339

5.8.6.2.2 Type RoamingAnalyticsSubscription 340

5.8.6.2.3 Type RoamingAnalyticsNotification 341

5.8.7 Error handling 341

5.8.7.1 General 341

5.8.7.2 Protocol Errors 341

5.8.8 Feature negotiation 342

5.8.9 Security 342

Annex A (normative): OpenAPI specification 343

A.1 General 343

A.2 Nnwdaf_EventsSubscription API 343

A.3 Nnwdaf_AnalyticsInfo API 395

A.4 Nnwdaf_DataManagement API 408

A.5 Nnwdaf_MLModelProvision API 413

A.6 Nnwdaf_MLModelTraining API 421

A.7 Nnwdaf_MLModelMonitor API 429

A.8 Nnwdaf_RoamingData API 435

Annex B (informative): Change history 443
