---
spec: TS 29.510
version: 18.11.0
release: '18'
clause: contents
title: Contents
source_archive: 29510-ib0.zip
source_document: 29510-ib0.docx
source_archive_sha256: 84e5ccfacace52d488cdca8f0ee1cb3b6817d390ee9903dc5e52105d58c6a1e6
source_document_sha256: 01abf6c9240ec101eef77e3ffc4bab5aca41dfd0d6d02c7515e02364cea887d9
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Contents


Foreword 18

1 Scope 19

2 References 19

3 Definitions and abbreviations 21

3.1 Definitions 21

3.2 Abbreviations 21

4 Overview 22

5 Services Offered by the NRF 23

5.1 Introduction 23

5.2 Nnrf_NFManagement Service 23

5.2.1 Service Description 23

5.2.2 Service Operations 24

5.2.2.1 Introduction 24

5.2.2.2 NFRegister 26

5.2.2.2.1 General 26

5.2.2.2.2 NF (other than NRF) registration to NRF 26

5.2.2.2.3 NRF registration to another NRF 27

5.2.2.2.4 Shared Data registration to NRF 28

5.2.2.3 NFUpdate 29

5.2.2.3.1 General 29

5.2.2.3.1A NF Profile Complete replacement 29

5.2.2.3.1B NF Profile Partial Update 29

5.2.2.3.1C Shared Data Complete replacement 30

5.2.2.3.1D Shared data Partial Update 31

5.2.2.3.2 NF Heart-Beat 32

5.2.2.4 NFDeregister 33

5.2.2.4.1 General 33

5.2.2.4.2 NF Instance Deregistration 33

5.2.2.4.3 Shared Data Deregistration 34

5.2.2.5 NFStatusSubscribe 34

5.2.2.5.1 General 34

5.2.2.5.2 Subscription to NF Instances in the same PLMN 35

5.2.2.5.3 Subscription to NF Instances in a different PLMN 36

5.2.2.5.4 Subscription to NF Instances with intermediate forwarding NRF 37

5.2.2.5.5 Subscription to NF Instances with intermediate redirecting NRF 38

5.2.2.5.6 Update of Subscription to NF Instances 39

5.2.2.5.7 Update of Subscription to NF Instances in a different PLMN 40

5.2.2.6 NFStatusNotify 41

5.2.2.6.1 General 41

5.2.2.6.2 Notification from NRF in the same PLMN 41

5.2.2.6.3 Notification from NRF in a different PLMN 43

5.2.2.6.4 Notification for subscription via intermediate NRF 44

5.2.2.7 NFStatusUnSubscribe 44

5.2.2.7.1 General 44

5.2.2.7.2 Subscription removal in the same PLMN 44

5.2.2.7.3 Subscription removal in a different PLMN 45

5.2.2.8 NFListRetrieval 45

5.2.2.8.1 General 45

5.2.2.9 NFProfileRetrieval 46

5.2.2.9.1 General 46

5.2.2.10 SharedDataRetrieval 47

5.2.2.11 SharedDataListRetrieval 47

5.3 Nnrf_NFDiscovery Service 48

5.3.1 Service Description 48

5.3.2 Service Operations 48

5.3.2.1 Introduction 48

5.3.2.2 NFDiscover 49

5.3.2.2.1 General 49

5.3.2.2.2 Service Discovery in the same PLMN 50

5.3.2.2.3 Service Discovery in a different PLMN 51

5.3.2.2.4 Service Discovery with intermediate redirecting NRF 52

5.3.2.2.5 Service Discovery with intermediate forwarding NRF 53

5.3.2.2.6 Service Discovery with resolution of the target PLMN 55

5.3.2.3 SCPDomainRoutingInfoGet 55

5.3.2.4 SCPDomainRoutingInfoSubscribe 56

5.3.2.5 SCPDomainRoutingInfoNotify 57

5.3.2.6 SCPDomainRoutingInfoUnSubscribe 58

5.4 Nnrf_AccessToken Service 58

5.4.1 Service Description 58

5.4.2 Service Operations 58

5.4.2.1 Introduction 58

5.4.2.2 Get (Access Token Request) 58

5.4.2.2.1 General 58

5.4.2.2.2 Access Token request with intermediate forwarding NRF 60

5.4.2.2.3 Access Token request with intermediate redirecting NRF 61

5.5 Nnrf_Bootstrapping Service 62

5.5.1 Service Description 62

5.5.2 Service Operations 62

5.5.2.1 Introduction 62

5.5.2.2 Get 62

5.5.2.2.1 General 62

6 API Definitions 64

6.1 Nnrf_NFManagement Service API 64

6.1.1 API URI 64

6.1.2 Usage of HTTP 64

6.1.2.1 General 64

6.1.2.2 HTTP standard headers 64

6.1.2.2.1 General 64

6.1.2.2.2 Content type 64

6.1.2.2.3 Accept-Encoding 64

6.1.2.2.4 ETag 64

6.1.2.2.5 If-Match 65

6.1.2.3 HTTP custom headers 65

6.1.2.3.1 General 65

6.1.3 Resources 65

6.1.3.1 Overview 65

6.1.3.2 Resource: nf-instances (Store) 66

6.1.3.2.1 Description 66

6.1.3.2.2 Resource Definition 67

6.1.3.2.3 Resource Standard Methods 67

6.1.3.2.4 Resource Custom Operations 71

6.1.3.3 Resource: nf-instance (Document) 71

6.1.3.3.1 Description 71

6.1.3.3.2 Resource Definition 71

6.1.3.3.3 Resource Standard Methods 72

6.1.3.4 Resource: subscriptions (Collection) 78

6.1.3.4.1 Description 78

6.1.3.4.2 Resource Definition 78

6.1.3.4.3 Resource Standard Methods 78

6.1.3.5 Resource: subscription (Document) 80

6.1.3.5.1 Description 80

6.1.3.5.2 Resource Definition 80

6.1.3.5.3 Resource Standard Methods 80

6.1.3.6 Resource: shared-data (Document) 82

6.1.3.6.1 Description 82

6.1.3.6.2 Resource Definition 82

6.1.3.6.3 Resource Standard Methods 83

6.1.3.7 Resource: shared-data-store (Store) 89

6.1.3.7.1 Description 89

6.1.3.7.2 Resource Definition 89

6.1.3.7.3 Resource Standard Methods 89

6.1.4 Custom Operations without associated resources 89

6.1.5 Notifications 90

6.1.5.1 General 90

6.1.5.2 NF Instance Status Notification 90

6.1.5.2.1 Description 90

6.1.5.2.2 Notification Definition 90

6.1.6 Data Model 91

6.1.6.1 General 91

6.1.6.2 Structured data types 97

6.1.6.2.1 Introduction 97

6.1.6.2.2 Type: NFProfile 98

6.1.6.2.3 Type: NFService 110

6.1.6.2.4 Type: DefaultNotificationSubscription 119

6.1.6.2.5 Type: IpEndPoint 121

6.1.6.2.6 Type: UdrInfo 122

6.1.6.2.7 Type: UdmInfo 123

6.1.6.2.8 Type: AusfInfo 124

6.1.6.2.9 Type: SupiRange 125

6.1.6.2.10 Type: IdentityRange 125

6.1.6.2.11 Type: AmfInfo 126

6.1.6.2.12 Type: SmfInfo 128

6.1.6.2.13 Type: UpfInfo 130

6.1.6.2.14 Type: SnssaiUpfInfoItem 132

6.1.6.2.15 Type: DnnUpfInfoItem 133

6.1.6.2.16 Type: SubscriptionData 134

6.1.6.2.17 Type: NotificationData 139

6.1.6.2.18 Void 141

6.1.6.2.19 Type: NFServiceVersion 141

6.1.6.2.20 Type: PcfInfo 142

6.1.6.2.21 Type: BsfInfo 143

6.1.6.2.22 Type: Ipv4AddressRange 144

6.1.6.2.23 Type: Ipv6PrefixRange 144

6.1.6.2.24 Type: InterfaceUpfInfoItem 144

6.1.6.2.25 Type: UriList 145

6.1.6.2.26 Type: N2InterfaceAmfInfo 145

6.1.6.2.27 Type: TaiRange 145

6.1.6.2.28 Type: TacRange 146

6.1.6.2.29 Type: SnssaiSmfInfoItem 146

6.1.6.2.30 Type: DnnSmfInfoItem 147

6.1.6.2.31 Type: NrfInfo 148

6.1.6.2.32 Type: ChfInfo 151

6.1.6.2.33 Void 152

6.1.6.2.34 Type: PlmnRange 152

6.1.6.2.35 Type: SubscrCond 153

6.1.6.2.36 Type: NfInstanceIdCond 153

6.1.6.2.37 Type: NfTypeCond 154

6.1.6.2.38 Type: ServiceNameCond 154

6.1.6.2.39 Type: AmfCond 154

6.1.6.2.40 Type: GuamiListCond 154

6.1.6.2.41 Type: NetworkSliceCond 154

6.1.6.2.42 Type: NfGroupCond 155

6.1.6.2.43 Type: NotifCondition 155

6.1.6.2.44 Type: PlmnSnssai 155

6.1.6.2.45 Type: NwdafInfo 156

6.1.6.2.46 Type: LmfInfo 157

6.1.6.2.47 Type: GmlcInfo 158

6.1.6.2.48 Type: NefInfo 159

6.1.6.2.49 Type: PfdData 160

6.1.6.2.50 Type: AfEventExposureData 160

6.1.6.2.51 Type: WAgfInfo 160

6.1.6.2.52 Type: TngfInfo 161

6.1.6.2.53 Type: PcscfInfo 161

6.1.6.2.54 Type: NfSetCond 162

6.1.6.2.55 Type: NfServiceSetCond 162

6.1.6.2.56 Type: NfInfo 162

6.1.6.2.57 Type: HssInfo 163

6.1.6.2.58 Type: ImsiRange 163

6.1.6.2.59 Type: InternalGroupIdRange 164

6.1.6.2.60 Type: UpfCond 164

6.1.6.2.61 Type: TwifInfo 164

6.1.6.2.62 Type: VendorSpecificFeature 165

6.1.6.2.63 Type: UdsfInfo 165

6.1.6.2.64 Type: NfInstanceIdListCond 165

6.1.6.2.65 Type: ScpInfo 166

6.1.6.2.66 Type: ScpDomainInfo 168

6.1.6.2.67 Type: ScpDomainCond 168

6.1.6.2.68 Type: OptionsResponse 169

6.1.6.2.69 Type: NwdafCond 169

6.1.6.2.70 Type: NefCond 170

6.1.6.2.71 Type: SuciInfo 170

6.1.6.2.72 Type: SeppInfo 171

6.1.6.2.73 Type: AanfInfo 171

6.1.6.2.74 Type: 5GDdnmfInfo 171

6.1.6.2.75 Type: MfafInfo 172

6.1.6.2.76 Type: NwdafCapability 173

6.1.6.2.77 Type: EasdfInfo 174

6.1.6.2.78 Type: SnssaiEasdfInfoItem 174

6.1.6.2.79 Type: DnnEasdfInfoItem 175

6.1.6.2.80 Type: DccfInfo 175

6.1.6.2.81 Type: NsacfInfo 176

6.1.6.2.82 Type: NsacfCapability 178

6.1.6.2.83 Type: DccfCond 179

6.1.6.2.84 Type: MlAnalyticsInfo 180

6.1.6.2.85 Type: MbSmfInfo 181

6.1.6.2.86 Type: TmgiRange 181

6.1.6.2.87 Type: MbsSession 182

6.1.6.2.88 Type: SnssaiMbSmfInfoItem 182

6.1.6.2.89 Type: DnnMbSmfInfoItem 182

6.1.6.2.90 Void 183

6.1.6.2.91 Type: TsctsfInfo 183

6.1.6.2.92 Type: SnssaiTsctsfInfoItem 183

6.1.6.2.93 Type: DnnTsctsfInfoItem 183

6.1.6.2.94 Type: MbUpfInfo 184

6.1.6.2.95 Type: UnTrustAfInfo 185

6.1.6.2.96 Type: TrustAfInfo 185

6.1.6.2.97 Type: SnssaiInfoItem 186

6.1.6.2.98 Type: DnnInfoItem 186

6.1.6.2.99 Type: CollocatedNfInstance 186

6.1.6.2.100 Type: ServiceNameListCond 186

6.1.6.2.101 Type: NfGroupListCond 187

6.1.6.2.102 Type: PlmnOauth2 187

6.1.6.2.103 Type: V2xCapability 187

6.1.6.2.104 Type: NssaafInfo 188

6.1.6.2.105 Type: ProSeCapability 189

6.1.6.2.106 Type: SharedDataIdRange 190

6.1.6.2.107 Type: SubscriptionContext 190

6.1.6.2.108 Type: IwmscInfo 190

6.1.6.2.109 Type: MnpfInfo 191

6.1.6.2.110 Type: DefSubServiceInfo 191

6.1.6.2.111 Type: LocalityDescriptionItem 191

6.1.6.2.112 Type: LocalityDescription 191

6.1.6.2.113 Type: SmsfInfo 192

6.1.6.2.114 Type: DcsfInfo 192

6.1.6.2.115 Type: MlModelInterInfo 192

6.1.6.2.116 Type: PruExistenceInfo 193

6.1.6.2.117 Type: MrfInfo 193

6.1.6.2.118 Type: MrfpInfo 193

6.1.6.2.119 Type: MfInfo 193

6.1.6.2.120 Type: A2xCapability 194

6.1.6.2.121 Type: RuleSet 195

6.1.6.2.122 Type: AdrfInfo 196

6.1.6.2.123 Type: SelectionConditions 196

6.1.6.2.124 Type: ConditionItem 197

6.1.6.2.125 Type: ConditionGroup 198

6.1.6.2.126 Type: EpdgInfo 198

6.1.6.2.127 Type: CallbackUriPrefixItem 198

6.1.6.2.128 Type: SharedData 199

6.1.6.2.129 Type: NFProfileRegistrationError 199

6.1.6.2.130 Type: SharedDataIdList 200

6.1.6.2.131 Type: PortRange 200

6.1.6.2.132 Type: SharedScope 200

6.1.6.3 Simple data types and enumerations 200

6.1.6.3.1 Introduction 200

6.1.6.3.2 Simple data types 200

6.1.6.3.3 Enumeration: NFType 201

6.1.6.3.4 Enumeration: NotificationType 204

6.1.6.3.5 Enumeration: TransportProtocol 206

6.1.6.3.6 Enumeration: NotificationEventType 206

6.1.6.3.7 Enumeration: NFStatus 206

6.1.6.3.8 Enumeration: DataSetId 206

6.1.6.3.9 Enumeration: UPInterfaceType 208

6.1.6.3.10 Relation Types 208

6.1.6.3.11 Enumeration: ServiceName 209

6.1.6.3.12 Enumeration: NFServiceStatus 212

6.1.6.3.13 Enumeration: AnNodeType 212

6.1.6.3.14 Enumeration: ConditionEventType 212

6.1.6.3.15 Enumeration: IpReachability 213

6.1.6.3.16 Enumeration: ScpCapability 213

6.1.6.3.17 Enumeration: CollocatedNfType 213

6.1.6.3.18 Enumeration: LocalityType 213

6.1.6.3.19 Enumeration: FlCapabilityType 213

6.1.6.3.20 Void 214

6.1.6.3.21 Enumeration: RuleSetAction 214

6.1.6.3.22 Enumeration: DnsSecurityProtocol 214

6.1.7 Error Handling 214

6.1.7.1 General 214

6.1.7.2 Protocol Errors 214

6.1.7.3 Application Errors 214

6.1.8 Security 215

6.1.9 Features supported by the NFManagement service 215

6.2 Nnrf_NFDiscovery Service API 216

6.2.1 API URI 216

6.2.2 Usage of HTTP 217

6.2.2.1 General 217

6.2.2.2 HTTP standard headers 217

6.2.2.2.1 General 217

6.2.2.2.2 Content type 217

6.2.2.2.3 Cache-Control 217

6.2.2.2.4 ETag 217

6.2.2.2.5 If-None-Match 217

6.2.2.3 HTTP custom headers 217

6.2.2.3.1 General 217

6.2.3 Resources 218

6.2.3.1 Overview 218

6.2.3.2 Resource: nf-instances (Store) 219

6.2.3.2.1 Description 219

6.2.3.2.2 Resource Definition 219

6.2.3.2.3 Resource Standard Methods 219

6.2.3.2.4 Resource Custom Operations 243

6.2.3.3 Resource: Stored Search (Document) 243

6.2.3.3.1 Description 243

6.2.3.3.2 Resource Definition 243

6.2.3.4 Resource: Complete Stored Search (Document) 244

6.2.3.4.1 Description 244

6.2.3.4.2 Resource Definition 244

6.2.3.5 Resource: SCP Domain Routing Information (Document) 245

6.2.3.5.1 Description 245

6.2.3.5.2 Resource Definition 245

6.2.3.6 Resource: SCP Domain Routing Information Subscriptions (Collection) 246

6.2.3.6.1 Description 246

6.2.3.6.2 Resource Definition 246

6.2.3.6.3 Resource Standard Methods 246

6.2.3.7 Resource: Individual SCP Domain Routing Information Subscription (Document) 247

6.2.3.7.1 Description 247

6.2.3.7.2 Resource Definition 247

6.2.3.7.3 Resource Standard Methods 247

6.2.4 Custom Operations without associated resources 248

6.2.5 Notifications 248

6.2.5.1 General 248

6.2.5.2 SCP Domain Routing Information Change Notification 248

6.2.5.2.1 Description 248

6.2.5.2.2 Notification Definition 248

6.2.6 Data Model 249

6.2.6.1 General 249

6.2.6.2 Structured data types 252

6.2.6.2.1 Introduction 252

6.2.6.2.2 Type: SearchResult 253

6.2.6.2.3 Type: NFProfile 255

6.2.6.2.4 Type: NFService 264

6.2.6.2.5 Type: StoredSearchResult 270

6.2.6.2.6 Type: PreferredSearch 271

6.2.6.2.7 Type: NfInstanceInfo 273

6.2.6.2.8 Type: ScpDomainRoutingInformation 273

6.2.6.2.9 Type: ScpDomainConnectivity 274

6.2.6.2.10 Type: ScpDomainRoutingInfoSubscription 274

6.2.6.2.11 Type: ScpDomainRoutingInfoNotification 275

6.2.6.2.12 Type: NfServiceInstance 275

6.2.6.2.13 Type: NoProfileMatchInfo 275

6.2.6.2.14 Type: QueryParamCombination 275

6.2.6.2.15 Type: QueryParameter 275

6.2.6.2.16 Type: AfData 276

6.2.6.2.17 Type: SearchResultInfo 276

6.2.6.3 Simple data types and enumerations 276

6.2.6.3.1 Introduction 276

6.2.6.3.2 Simple data types 276

6.2.6.3.3 Enumeration: NoProfileMatchReason 276

6.2.7 Error Handling 277

6.2.7.1 General 277

6.2.7.2 Protocol Errors 277

6.2.7.3 Application Errors 277

6.2.8 Security 277

6.2.9 Features supported by the NFDiscovery service 278

6.3 Nnrf_AccessToken Service API 283

6.3.1 General 283

6.3.2 API URI 283

6.3.3 Usage of HTTP 283

6.3.3.1 General 283

6.3.3.2 HTTP standard headers 283

6.3.3.2.1 General 283

6.3.3.2.2 Content type 283

6.3.3.3 HTTP custom headers 284

6.3.3.3.1 General 284

6.3.4 Custom Operations without associated resources 284

6.3.4.1 Overview 284

6.3.4.2 Operation: Get (Access Token Request) 284

6.3.4.2.1 Description 284

6.3.4.2.2 Operation Definition 284

6.3.5 Data Model 286

6.3.5.1 General 286

6.3.5.2 Structured data types 286

6.3.5.2.1 Introduction 286

6.3.5.2.2 Type: AccessTokenReq 287

6.3.5.2.3 Type: AccessTokenRsp 291

6.3.5.2.4 Type: AccessTokenClaims 292

6.3.5.2.5 Type: AccessTokenErr 294

6.3.5.2.6 Type: MlModelInterInd 294

6.3.5.3 Simple data types and enumerations 294

6.3.5.3.1 Introduction 294

6.3.5.3.2 Simple data types 294

6.3.5.3.3 Void 294

6.3.5.4 Data types describing alternative data types or combinations of data types 294

6.3.5.4.1 Type: Audience 294

6.3.6 Error Handling 295

6.3.6.1 General 295

6.3.6.2 Protocol Errors 295

6.3.6.3 Application Errors 295

6.4 Nnrf_Bootstrapping Service API 295

6.4.1 API URI 295

6.4.2 Usage of HTTP 295

6.4.2.1 General 295

6.4.2.2 HTTP standard headers 295

6.4.2.2.1 General 295

6.4.2.2.2 Content type 295

6.4.2.2.3 Cache-Control 296

6.4.2.2.4 ETag 296

6.4.2.2.5 If-None-Match 296

6.4.2.3 HTTP custom headers 296

6.4.2.3.1 General 296

6.4.3 Resources 296

6.4.3.1 Overview 296

6.4.3.2 Resource: Bootstrapping (Document) 297

6.4.3.2.1 Description 297

6.4.3.2.2 Resource Definition 297

6.4.3.2.3 Resource Standard Methods 297

6.4.4 Custom Operations without associated resources 298

6.4.5 Notifications 298

6.4.6 Data Model 298

6.4.6.1 General 298

6.4.6.2 Structured data types 298

6.4.6.2.1 Introduction 298

6.4.6.2.2 Type: BootstrappingInfo 299

6.4.6.3 Simple data types and enumerations 299

6.4.6.3.1 Introduction 299

6.4.6.3.2 Enumeration: Status 299

6.4.6.3.3 Relation Types 300

Annex A (normative): OpenAPI specification 301

A.1 General 301

A.2 Nnrf_NFManagement API 301

A.3 Nnrf_NFDiscovery API 378

A.4 Nnrf_AccessToken API (NRF OAuth2 Authorization) 411

A.5 Nnrf_Bootstrapping API 415

Annex B (normative): NF Profile changes in NFRegister and NFUpdate responses 418

B.1 General 418

Annex C (normative): Enhanced Authorization Policy using RuleSets in NF (Service) Profile 419

C.1 General 419

C.2 Examples of NF-Producer profile only using RuleSets (i.e. without AllowedXXX parameters) in NF (Service) Profile 419

C.3 Example of NF-Producer profile using RuleSets and AllowedXXX parameters in NF (Service) Profile 420

C.4 Backward Compatibility 421

Annex D (normative): Support of "Canary Release" testing in the NRF 421

Annex E (informative): Change history 425
