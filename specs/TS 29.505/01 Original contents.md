---
spec: TS 29.505
version: 18.8.0
release: '18'
source_archive: 29505-i80.zip
source_document: 29505-i80.docx
source_archive_sha256: b432c3d6d7a211f0d1eeb72da4364d32ba1d9a3ae51c03df8cab38d2700b777e
source_document_sha256: 46edda85efed924d08a4f02c12acb7970b710feae119f61edd880bb6a2db96a3
clause: contents
title: Contents
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Contents

Foreword [13](#foreword)

1 Scope [14](#scope)

2 References [14](#references)

3 Definitions and abbreviations [15](#definitions-and-abbreviations)

3.1 Definitions [15](#definitions)

3.2 Abbreviations [15](#abbreviations)

4 Overview [15](#overview)

5 Usage of Nudr_DataRepository Service [16](#usage-of-nudr_datarepository-service)

5.1 Introduction [16](#introduction)

5.2 Resources [17](#resources)

5.2.1 Overview [17](#overview-1)

5.2.2 Resource: AuthenticationSubscription [29](#resource-authenticationsubscription)

5.2.2.1 Description [29](#description)

5.2.2.2 Resource Definition [29](#resource-definition)

5.2.2.3 Resource Standard Methods [30](#resource-standard-methods)

5.2.2.3.1 GET [30](#get)

5.2.2.3.2 PATCH [30](#patch)

5.2.3 Resource: AccessAndMobilitySubscriptionData [31](#resource-accessandmobilitysubscriptiondata)

5.2.3.1 Description [31](#description-1)

5.2.3.2 Resource Definition [31](#resource-definition-1)

5.2.3.3 Resource Standard Methods [31](#resource-standard-methods-1)

5.2.3.3.1 GET [31](#get-1)

5.2.4 Resource: SmfSelectionSubscriptionData [32](#resource-smfselectionsubscriptiondata)

5.2.4.1 Description [32](#description-2)

5.2.4.2 Resource Definition [32](#resource-definition-2)

5.2.4.3 Resource Standard Methods [33](#resource-standard-methods-2)

5.2.4.3.1 GET [33](#get-2)

5.2.5 Resource: SessionManagementSubscriptionData [33](#resource-sessionmanagementsubscriptiondata)

5.2.5.1 Description [33](#description-3)

5.2.5.2 Resource Definition [33](#resource-definition-3)

5.2.5.3 Resource Standard Methods [34](#resource-standard-methods-3)

5.2.5.3.1 GET [34](#get-3)

5.2.6 Resource: Amf3GppAccessRegistration [34](#resource-amf3gppaccessregistration)

5.2.6.1 Description [34](#description-4)

5.2.6.2 Resource Definition [34](#resource-definition-4)

5.2.6.3 Resource Standard Methods [35](#resource-standard-methods-4)

5.2.6.3.1 PUT [35](#put)

5.2.6.3.2 PATCH [35](#patch-1)

5.2.6.3.3 GET [36](#get-4)

5.2.7 Resource: AmfNon3GppAccessRegistration [36](#resource-amfnon3gppaccessregistration)

5.2.7.1 Description [36](#description-5)

5.2.7.2 Resource Definition [37](#resource-definition-5)

5.2.7.3 Resource Standard Methods [37](#resource-standard-methods-5)

5.2.7.3.1 PUT [37](#put-1)

5.2.7.3.2 PATCH [37](#patch-2)

5.2.7.3.3 GET [38](#get-5)

5.2.8 Resource: SmfRegistrations [39](#resource-smfregistrations)

5.2.8.1 Description [39](#description-6)

5.2.8.2 Resource Definition [39](#resource-definition-6)

5.2.8.3 Resource Standard Methods [39](#resource-standard-methods-6)

5.2.8.3.1 GET [39](#get-6)

5.2.9 Resource: IndividualSmfRegistration [39](#resource-individualsmfregistration)

5.2.9.1 Description [39](#description-7)

5.2.9.2 Resource Definition [40](#resource-definition-7)

5.2.9.3 Resource Standard Methods [40](#resource-standard-methods-7)

5.2.9.3.1 PUT [40](#put-2)

5.2.9.3.2 DELETE [40](#delete)

5.2.9.3.3 GET [41](#get-7)

5.2.9.3.4 PATCH [41](#patch-3)

5.2.10 Resource: OperatorSpecificData [42](#resource-operatorspecificdata)

5.2.10.1 Description [42](#description-8)

5.2.10.2 Resource Definition [42](#resource-definition-8)

5.2.10.3 Resource Standard Methods [42](#resource-standard-methods-8)

5.2.10.3.1 GET [42](#get-8)

5.2.10.3.2 PATCH [43](#patch-4)

5.2.10.3.3 PUT [44](#_Toc177493781)

5.2.10.3.4 DELETE [44](#delete-1)

5.2.11 Resource: Smsf3GppAccessRegistration [45](#resource-smsf3gppaccessregistration)

5.2.11.1 Description [45](#description-9)

5.2.11.2 Resource Definition [45](#resource-definition-9)

5.2.11.3 Resource Standard Methods [45](#resource-standard-methods-9)

5.2.11.3.1 PUT [45](#put-4)

5.2.11.3.2 DELETE [46](#delete-2)

5.2.11.3.4 GET [46](#get-9)

5.2.12 Resource: SmsfNon3GppAccessRegistration [47](#resource-smsfnon3gppaccessregistration)

5.2.12.1 Description [47](#description-10)

5.2.12.2 Resource Definition [47](#resource-definition-10)

5.2.12.3 Resource Standard Methods [47](#resource-standard-methods-10)

5.2.12.3.1 PUT [47](#put-5)

5.2.12.3.2 DELETE [48](#delete-3)

5.2.12.3.4 GET [48](#get-10)

5.2.12A Resource: IpSmGwRegistration [49](#a-resource-ipsmgwregistration)

5.2.12A.1 Description [49](#a.1-description)

5.2.12A.2 Resource Definition [49](#a.2-resource-definition)

5.2.12A.3 Resource Standard Methods [49](#a.3-resource-standard-methods)

5.2.12A.3.1 PUT [49](#a.3.1-put)

5.2.12A.3.2 DELETE [50](#a.3.2-delete)

5.2.12A.3.3 PATCH [50](#a.3.3-patch)

5.2.12A.3.4 GET [51](#a.3.4-get)

5.2.12B Resource: MessageWaitingData [51](#b-resource-messagewaitingdata)

5.2.12B.1 Description [51](#b.1-description)

5.2.12B.2 Resource Definition [52](#b.2-resource-definition)

5.2.12B.3 Resource Standard Methods [52](#b.3-resource-standard-methods)

5.2.12B.3.1 PUT [52](#b.3.1-put)

5.2.12B.3.2 DELETE [52](#b.3.2-delete)

5.2.12B.3.3 PATCH [53](#b.3.3-patch)

5.2.12B.3.4 GET [53](#b.3.4-get)

5.2.13 Resource: SMSManagementSubscriptionData [54](#resource-smsmanagementsubscriptiondata)

5.2.13.1 Description [54](#description-11)

5.2.13.2 Resource Definition [54](#resource-definition-11)

5.2.13.3 Resource Standard Methods [54](#resource-standard-methods-11)

5.2.13.3.1 GET [54](#get-11)

5.2.14 Resource: ProvisionedParamenterData [55](#resource-provisionedparamenterdata)

5.2.14.1 Description [55](#description-12)

5.2.14.2 Resource Definition [55](#resource-definition-12)

5.2.14.3 Resource Standard Methods [55](#resource-standard-methods-12)

5.2.14.3.1 GET [55](#get-12)

5.2.14.3.2 PATCH [56](#patch-5)

5.2.14A Resource: PpProfileData [57](#a-resource-ppprofiledata)

5.2.14A.1 Description [57](#a.1-description-1)

5.2.14A.2 Resource Definition [57](#a.2-resource-definition-1)

5.2.14A.3 Resource Standard Methods [57](#a.3-resource-standard-methods-1)

5.2.14A.3.1 GET [57](#a.3.1-get)

5.2.14B Resource: ProvisionedParameterDataEntry [57](#b-resource-provisionedparameterdataentry)

5.2.14B.1 Description [57](#b.1-description-1)

5.2.14B.2 Resource Definition [58](#b.2-resource-definition-1)

5.2.14B.3 Resource Standard Methods [58](#b.3-resource-standard-methods-1)

5.2.14B.3.1 PUT [58](#b.3.1-put-1)

5.2.14B.3.2 DELETE [59](#b.3.2-delete-1)

5.2.14B.3.3 GET [59](#b.3.3-get)

5.2.14C Resource: ProvisionedParameterDataEntries [60](#c-resource-provisionedparameterdataentries)

5.2.14C.1 Description [60](#c.1-description)

5.2.14C.2 Resource Definition [60](#c.2-resource-definition)

5.2.14C.3.1 GET [60](#c.3.1-get)

5.2.15 Resource: SMSSubscriptionData [61](#resource-smssubscriptiondata)

5.2.15.1 Description [61](#description-13)

5.2.15.2 Resource Definition [61](#resource-definition-13)

5.2.15.3 Resource Standard Methods [61](#resource-standard-methods-13)

5.2.15.3.1 GET [61](#get-13)

5.2.16 Resource: SdmSubscriptions [62](#resource-sdmsubscriptions)

5.2.16.1 Description [62](#description-14)

5.2.16.2 Resource Definition [62](#resource-definition-14)

5.2.16.3 Resource Standard Methods [62](#resource-standard-methods-14)

5.2.16.3.1 GET [62](#get-14)

5.2.16.3.2 POST [62](#post)

5.2.17 Resource: IndividualSdmSubscription [63](#resource-individualsdmsubscription)

5.2.17.1 Description [63](#description-15)

5.2.17.2 Resource Definition [63](#resource-definition-15)

5.2.17.3 Resource Standard Methods [63](#resource-standard-methods-15)

5.2.17.3.1 PUT [63](#put-6)

5.2.17.3.2 DELETE [64](#delete-4)

5.2.17.3.3 PATCH [64](#patch-6)

5.2.17.3.4 GET [65](#get-15)

5.2.17A Resource: HssSdmSubscriptionInfo [66](#a-resource-hsssdmsubscriptioninfo)

5.2.17A.1 Description [66](#a.1-description-2)

5.2.17A.2 Resource Definition [66](#a.2-resource-definition-2)

5.2.17A.3 Resource Standard Methods [66](#a.3-resource-standard-methods-2)

5.2.17A.3.1 PUT [66](#a.3.1-put-1)

5.2.17A.3.2 DELETE [67](#a.3.2-delete-1)

5.2.17A.3.3 GET [67](#a.3.3-get)

5.2.17A.3.4 PATCH [68](#a.3.4-patch)

5.2.18 Resource: EeSubscriptions [68](#resource-eesubscriptions)

5.2.18.1 Description [68](#description-16)

5.2.18.2 Resource Definition [68](#resource-definition-16)

5.2.18.3 Resource Standard Methods [69](#resource-standard-methods-16)

5.2.18.3.1 GET [69](#get-16)

5.2.18.3.2 POST [69](#post-1)

5.2.19 Resource: IndividualEeSubscription [70](#resource-individualeesubscription)

5.2.19.1 Description [70](#description-17)

5.2.19.2 Resource Definition [70](#resource-definition-17)

5.2.19.3 Resource Standard Methods [70](#resource-standard-methods-17)

5.2.19.3.1 PUT [70](#put-7)

5.2.19.3.2 DELETE [71](#delete-5)

5.2.19.3.3 PATCH [71](#patch-7)

5.2.19.3.4 GET [72](#get-17)

5.2.19A Resource: IndividualEeGroupSubscription [72](#a-resource-individualeegroupsubscription)

5.2.19A.1 Description [72](#a.1-description-3)

5.2.19A.2 Resource Definition [72](#a.2-resource-definition-3)

5.2.19A.3 Resource Standard Methods [73](#a.3-resource-standard-methods-3)

5.2.19A.3.1 PUT [73](#a.3.1-put-2)

5.2.19A.3.2 DELETE [73](#a.3.2-delete-2)

5.2.19A.3.3 PATCH [74](#a.3.3-patch-1)

5.2.19A.3.4 GET [74](#a.3.4-get-1)

5.2.19B Resource: EeGroupProfileData [75](#b-resource-eegroupprofiledata)

5.2.19B.1 Description [75](#b.1-description-2)

5.2.19B.2 Resource Definition [75](#b.2-resource-definition-2)

5.2.19B.3 Resource Standard Methods [75](#b.3-resource-standard-methods-2)

5.2.19B.3.1 GET [75](#b.3.1-get)

5.2.19C Resource: AmfGroupSubscriptionInfo [76](#c-resource-amfgroupsubscriptioninfo)

5.2.19C.1 Description [76](#c.1-description-1)

5.2.19C.2 Resource Definition [76](#c.2-resource-definition-1)

5.2.19C.3 Resource Standard Methods [76](#c.3-resource-standard-methods)

5.2.19C.3.1 PUT [76](#c.3.1-put)

5.2.19C.3.2 DELETE [77](#c.3.2-delete)

5.2.19C.3.3 GET [77](#c.3.3-get)

5.2.31.3.4 PATCH [78](#patch-8)

5.2.19D Resource: SmfGroupSubscriptionInfo [78](#d-resource-smfgroupsubscriptioninfo)

5.2.19D.1 Description [78](#d.1-description)

5.2.19D.2 Resource Definition [79](#d.2-resource-definition)

5.2.19D.3 Resource Standard Methods [79](#d.3-resource-standard-methods)

5.2.19D.3.1 PUT [79](#d.3.1-put)

5.2.19D.3.2 DELETE [79](#d.3.2-delete)

5.2.19D.3.3 GET [80](#d.3.3-get)

5.2.19D.3.4 PATCH [80](#d.3.4-patch)

5.2.19E Resource: HssGroupSubscriptionInfo [81](#e-resource-hssgroupsubscriptioninfo)

5.2.19E.1 Description [81](#e.1-description)

5.2.19E.2 Resource Definition [81](#e.2-resource-definition)

5.2.19E.3 Resource Standard Methods [81](#e.3-resource-standard-methods)

5.2.19E.3.1 PUT [81](#e.3.1-put)

5.2.19E.3.2 DELETE [82](#e.3.2-delete)

5.2.19E.3.3 GET [82](#e.3.3-get)

5.2.19E.3.4 PATCH [83](#e.3.4-patch)

5.2.20 Resource: SubscriptionDataSubscriptions [83](#resource-subscriptiondatasubscriptions)

5.2.20.1 Description [83](#description-18)

5.2.20.2 Resource Definition [84](#resource-definition-18)

5.2.20.3 Resource Standard Methods [84](#resource-standard-methods-18)

5.2.20.3.1 POST [84](#post-2)

5.2.20.3.2 GET [85](#get-18)

5.2.20.3.3 DELETE [85](#delete-6)

5.2.21 Resource: IndividualSubscriptionDataSubscription [86](#resource-individualsubscriptiondatasubscription)

5.2.21.1 Description [86](#description-19)

5.2.21.2 Resource Definition [86](#resource-definition-19)

5.2.21.3 Resource Standard Methods [87](#resource-standard-methods-19)

5.2.21.3.1 DELETE [87](#delete-7)

5.2.21.3.2 PATCH [87](#patch-9)

5.2.21.3.3 GET [88](#get-19)

5.2.22 Resource: TraceData [89](#resource-tracedata)

5.2.22.1 Description [89](#description-20)

5.2.22.2 Resource Definition [89](#resource-definition-20)

5.2.22.3 Resource Standard Methods [89](#resource-standard-methods-20)

5.2.22.3.1 GET [89](#get-20)

5.2.23 Resource: IdentityData [90](#resource-identitydata)

5.2.23.1 Description [90](#description-21)

5.2.23.2 Resource Definition [90](#resource-definition-21)

5.2.23.3 Resource Standard Methods [90](#resource-standard-methods-21)

5.2.23.3.1 GET [90](#get-21)

5.2.24 Resource: AuthenticationStatus [91](#resource-authenticationstatus)

5.2.24.1 Description [91](#description-22)

5.2.24.2 Resource Definition [91](#resource-definition-22)

5.2.24.3 Resource Standard Methods [91](#resource-standard-methods-22)

5.2.24.3.1 PUT [91](#put-8)

5.2.24.3.2 GET [92](#get-22)

5.2.24.3.3 DELETE [92](#delete-8)

5.2.24A Resource: IndividualAuthenticationStatus [93](#a-resource-individualauthenticationstatus)

5.2.24A.1 Description [93](#a.1-description-4)

5.2.24A.2 Resource Definition [93](#a.2-resource-definition-4)

5.2.24A.3 Resource Standard Methods [93](#a.3-resource-standard-methods-4)

5.2.24A.3.1 PUT [93](#a.3.1-put-3)

5.2.24A.3.2 GET [94](#a.3.2-get)

5.2.24A.3.3 DELETE [94](#a.3.3-delete)

5.2.25 Resource: AuthenticationSoR [95](#resource-authenticationsor)

5.2.25.1 Description [95](#description-23)

5.2.25.2 Resource Definition [95](#resource-definition-23)

5.2.25.3 Resource Standard Methods [95](#resource-standard-methods-23)

5.2.25.3.1 PUT [95](#put-9)

5.2.25.3.2 GET [95](#get-23)

5.2.25.3.3 PATCH [96](#patch-10)

5.2.25A Resource: AuthenticationUPU [97](#a-resource-authenticationupu)

5.2.25A.1 Description [97](#a.1-description-5)

5.2.25A.2 Resource Definition [97](#a.2-resource-definition-5)

5.2.25A.3 Resource Standard Methods [97](#a.3-resource-standard-methods-5)

5.2.25A.3.1 PUT [97](#a.3.1-put-4)

5.2.25A.3.2 GET [98](#a.3.2-get-1)

5.2.25B Resource: SubscribedSNSSAIs [98](#b-resource-subscribedsnssais)

5.2.25B.1 Description [98](#b.1-description-3)

5.2.25B.2 Resource Definition [98](#b.2-resource-definition-3)

5.2.25B.3 Resource Standard Methods [99](#b.3-resource-standard-methods-3)

5.2.25B.3.1 PUT [99](#b.3.1-put-2)

5.2.25B.3.2 GET [99](#b.3.2-get)

5.2.25C Resource: SubscribedCAG [100](#c-resource-subscribedcag)

5.2.25C.1 Description [100](#c.1-description-2)

5.2.25C.2 Resource Definition [100](#c.2-resource-definition-2)

5.2.25C.3 Resource Standard Methods [100](#c.3-resource-standard-methods-1)

5.2.25C.3.1 PUT [100](#c.3.1-put-1)

5.2.25C.3.2 GET [100](#c.3.2-get)

5.2.26 Resource: ProvisionedData [101](#resource-provisioneddata)

5.2.26.1 Description [101](#description-24)

5.2.26.2 Resource Definition [101](#resource-definition-24)

5.2.26.3 Resource Standard Methods [101](#resource-standard-methods-24)

5.2.26.3.1 GET [101](#get-24)

5.2.27 Resource: OperatorDeterminedBarringData [102](#resource-operatordeterminedbarringdata)

5.2.27.1 Description [102](#description-25)

5.2.27.2 Resource Definition [103](#resource-definition-25)

5.2.27.3 Resource Standard Methods [103](#resource-standard-methods-25)

5.2.27.3.1 GET [103](#get-25)

5.2.28 Resource: EeProfileData [103](#resource-eeprofiledata)

5.2.28.1 Description [103](#description-26)

5.2.28.2 Resource Definition [103](#resource-definition-26)

5.2.28.3 Resource Standard Methods [104](#resource-standard-methods-26)

5.2.28.3.1 GET [104](#get-26)

5.2.29 Resource: EeGroupSubscriptions [104](#resource-eegroupsubscriptions)

5.2.29.1 Description [104](#description-27)

5.2.29.2 Resource Definition [104](#resource-definition-27)

5.2.29.3 Resource Standard Methods [105](#resource-standard-methods-27)

5.2.29.3.1 GET [105](#get-27)

5.2.29.3.2 POST [105](#post-3)

5.2.30 Resource: SharedData [106](#resource-shareddata)

5.2.30.1 Description [106](#description-28)

5.2.30.2 Resource Definition [106](#resource-definition-28)

5.2.30.3 Resource Standard Methods [106](#resource-standard-methods-28)

5.2.30.3.1 GET [106](#get-28)

5.2.30A Resource: IndividualSharedData [107](#a-resource-individualshareddata)

5.2.30A.1 Description [107](#a.1-description-6)

5.2.30A.2 Resource Definition [107](#a.2-resource-definition-6)

5.2.30A.3 Resource Standard Methods [107](#a.3-resource-standard-methods-6)

5.2.30A.3.1 GET [107](#a.3.1-get-1)

5.2.31 Resource: AmfSubscriptionInfo [107](#resource-amfsubscriptioninfo)

5.2.31.1 Description [107](#description-29)

5.2.31.2 Resource Definition [108](#resource-definition-29)

5.2.31.3 Resource Standard Methods [108](#resource-standard-methods-29)

5.2.31.3.1 PUT [108](#put-10)

5.2.31.3.2 DELETE [108](#delete-9)

5.2.31.3.3 GET [109](#get-29)

5.2.31.3.4 PATCH [109](#patch-11)

5.2.31A Resource: SmfSubscriptionInfo [110](#a-resource-smfsubscriptioninfo)

5.2.31A.1 Description [110](#a.1-description-7)

5.2.31A.2 Resource Definition [110](#a.2-resource-definition-7)

5.2.31A.3 Resource Standard Methods [110](#a.3-resource-standard-methods-7)

5.2.31A.3.1 PUT [110](#a.3.1-put-5)

5.2.31A.3.2 DELETE [111](#a.3.2-delete-3)

5.2.31A.3.3 GET [111](#a.3.3-get-1)

5.2.31A.3.4 PATCH [112](#a.3.4-patch-1)

5.2.31B Resource: HssSubscriptionInfo [113](#b-resource-hsssubscriptioninfo)

5.2.31B.1 Description [113](#b.1-description-4)

5.2.31B.2 Resource Definition [113](#b.2-resource-definition-4)

5.2.31B.3 Resource Standard Methods [113](#b.3-resource-standard-methods-4)

5.2.31B.3.1 PUT [113](#b.3.1-put-3)

5.2.31B.3.2 DELETE [114](#b.3.2-delete-2)

5.2.31B.3.3 GET [114](#b.3.3-get-1)

5.2.31B.3.4 PATCH [114](#b.3.4-patch)

5.2.32 Resource: ContextData [115](#resource-contextdata)

5.2.32.1 Description [115](#description-30)

5.2.32.2 Resource Definition [115](#resource-definition-30)

5.2.32.3 Resource Standard Methods [115](#resource-standard-methods-30)

5.2.32.3.1 GET [115](#get-30)

5.2.33 Resource: GroupIdentifiers [116](#resource-groupidentifiers)

5.2.33.1 Description [116](#description-31)

5.2.33.2 Resource Definition [116](#resource-definition-31)

5.2.33.3 Resource Standard Methods [116](#resource-standard-methods-31)

5.2.33.3.1 GET [116](#get-31)

5.2.34 Resource: 5GVnGroups [117](#resource-5gvngroups)

5.2.34.1 Description [117](#description-32)

5.2.34.2 Resource Definition [117](#resource-definition-32)

5.2.34.3 Resource Standard Methods [117](#resource-standard-methods-32)

5.2.34.3.1 GET [117](#get-32)

5.2.35 Resource: Individual5GVnGroup [118](#resource-individual5gvngroup)

5.2.35.1 Description [118](#description-33)

5.2.35.2 Resource Definition [118](#resource-definition-33)

5.2.35.3 Resource Standard Methods [118](#resource-standard-methods-33)

5.2.35.3.1 PUT [118](#put-11)

5.2.35.3.2 DELETE [119](#delete-10)

5.2.35.3.3 PATCH [120](#patch-12)

5.2.35.3.4 GET [120](#get-33)

5.2.36 Resource: LcsPrivacySubscriptionData [121](#resource-lcsprivacysubscriptiondata)

5.2.36.1 Description [121](#description-34)

5.2.36.2 Resource Definition [121](#resource-definition-34)

5.2.36.3 Resource Standard Methods [121](#resource-standard-methods-34)

5.2.36.3.1 GET [121](#get-34)

5.2.37 Resource: LcsMobileOriginatedSubscriptionData [122](#resource-lcsmobileoriginatedsubscriptiondata)

5.2.37.1 Description [122](#description-35)

5.2.37.2 Resource Definition [122](#resource-definition-35)

5.2.37.3 Resource Standard Methods [122](#resource-standard-methods-35)

5.2.37.3.1 GET [122](#get-35)

5.2.38 Resource: NiddAuthorizationData [123](#resource-niddauthorizationdata)

5.2.38.1 Description [123](#description-36)

5.2.38.2 Resource Definition [123](#resource-definition-36)

5.2.38.3 Resource Standard Methods [123](#resource-standard-methods-36)

5.2.38.3.1 GET [123](#get-36)

5.2.39 Resource: CoverageRestrictionData [124](#resource-coveragerestrictiondata)

5.2.39.1 Description [124](#description-37)

5.2.39.2 Resource Definition [124](#resource-definition-37)

5.2.39.3 Resource Standard Methods [124](#resource-standard-methods-37)

5.2.39.3.1 GET [124](#get-37)

5.2.40 Resource: Location [125](#resource-location)

5.2.40.1 Description [125](#description-38)

5.2.40.2 Resource Definition [125](#resource-definition-38)

5.2.40.3 Resource Standard Methods [125](#resource-standard-methods-38)

5.2.40.3.1 GET [125](#get-38)

5.2.41 Resource: V2xSubscriptionData [126](#resource-v2xsubscriptiondata)

5.2.41.1 Description [126](#description-39)

5.2.41.2 Resource Definition [126](#resource-definition-39)

5.2.41.3 Resource Standard Methods [126](#resource-standard-methods-39)

5.2.41.3.1 GET [126](#get-39)

5.2.41A Resource: ProseSubscriptionData [127](#a-resource-prosesubscriptiondata)

5.2.41A.1 Description [127](#a.1-description-8)

5.2.41A.2 Resource Definition [127](#a.2-resource-definition-8)

5.2.41A.3 Resource Standard Methods [127](#a.3-resource-standard-methods-8)

5.2.41A.3.1 GET [127](#a.3.1-get-2)

5.2.42 Resource: LcsBroadcastAssistanceSubscriptionData [128](#resource-lcsbroadcastassistancesubscriptiondata)

5.2.42.1 Description [128](#description-40)

5.2.42.2 Resource Definition [128](#resource-definition-40)

5.2.42.3 Resource Standard Methods [128](#resource-standard-methods-40)

5.2.42.3.1 GET [128](#get-40)

5.2.43 Resource: 5GVnGroupsInternal [129](#_Toc177494099)

5.2.43.1 Description [129](#description-41)

5.2.43.2 Resource Definition [129](#resource-definition-41)

5.2.43.3 Resource Standard Methods [129](#resource-standard-methods-41)

5.2.43.3.1 GET [129](#get-41)

5.2.44 Resource: Pp5gVnGroupProfileData [130](#resource-pp5gvngroupprofiledata)

5.2.44.1 Description [130](#description-42)

5.2.44.2 Resource Definition [130](#resource-definition-42)

5.2.44.3 Resource Standard Methods [130](#resource-standard-methods-42)

5.2.44.3.1 GET [130](#get-42)

5.2.45 Resource: NiddAuthorizations [131](#resource-niddauthorizations)

5.2.45.1 Description [131](#description-43)

5.2.45.2 Resource Definition [131](#resource-definition-43)

5.2.45.3 Resource Standard Methods [131](#resource-standard-methods-43)

5.2.45.3.1 PUT [131](#put-12)

5.2.45.3.2 DELETE [131](#delete-11)

5.2.45.3.3 GET [132](#get-43)

5.2.45.3.4 PATCH [132](#patch-13)

5.2.46 Resource: 5MBSSubscriptionData [133](#_Toc177494117)

5.2.46.1 Description [133](#description-44)

5.2.46.2 Resource Definition [133](#resource-definition-44)

5.2.46.3 Resource Standard Methods [133](#resource-standard-methods-44)

5.2.46.3.1 GET [133](#get-44)

5.2.47 Resource: UeSubscriptionData [134](#resource-uesubscriptiondata)

5.2.47.1 Description [134](#description-45)

5.2.47.2 Resource Definition [134](#resource-definition-45)

5.2.47.3 Resource Standard Methods [134](#resource-standard-methods-45)

5.2.47.3.1 GET [134](#get-45)

5.2.48 Resource: ServiceSpecificAuthorizationData [136](#resource-servicespecificauthorizationdata)

5.2.48.1 Description [136](#description-46)

5.2.48.2 Resource Definition [136](#resource-definition-46)

5.2.48.3 Resource Standard Methods [136](#resource-standard-methods-46)

5.2.48.3.1 GET [136](#get-46)

5.2.49 Resource: SpecificServiceAuthorizations [137](#resource-specificserviceauthorizations)

5.2.49.1 Description [137](#description-47)

5.2.49.2 Resource Definition [137](#resource-definition-47)

5.2.49.3 Resource Standard Methods [137](#resource-standard-methods-47)

5.2.49.3.1 PUT [137](#put-13)

5.2.49.3.2 DELETE [138](#delete-12)

5.2.49.3.3 GET [138](#get-47)

5.2.49.3.4 PATCH [139](#patch-14)

5.2.50 Resource: RoamingInfo [139](#resource-roaminginfo)

5.2.50.1 Description [139](#description-48)

5.2.50.2 Resource Definition [140](#resource-definition-48)

5.2.50.3 Resource Standard Methods [140](#resource-standard-methods-48)

5.2.50.3.1 PUT [140](#put-14)

5.2.50.3.2 GET [140](#get-48)

5.2.51 Resource: UserConsentData [141](#resource-userconsentdata)

5.2.51.1 Description [141](#description-49)

5.2.51.2 Resource Definition [141](#resource-definition-49)

5.2.51.3 Resource Standard Methods [141](#resource-standard-methods-49)

5.2.51.3.1 GET [141](#get-49)

5.2.52 Resource: PeiInfo [142](#resource-peiinfo)

5.2.52.1 Description [142](#description-50)

5.2.52.2 Resource Definition [142](#resource-definition-50)

5.2.52.3 Resource Standard Methods [142](#resource-standard-methods-50)

5.2.52.3.1 PUT [142](#put-15)

5.2.52.3.2 GET [143](#get-50)

5.2.53 Resource: TimeSyncSubscriptionData [143](#resource-timesyncsubscriptiondata)

5.2.53.1 Description [143](#description-51)

5.2.53.2 Resource Definition [143](#resource-definition-51)

5.2.53.3 Resource Standard Methods [144](#resource-standard-methods-51)

5.2.53.3.1 GET [144](#get-51)

5.2.54 Resource: 5GMbsGroup [144](#resource-5gmbsgroup)

5.2.54.1 Description [144](#description-52)

5.2.54.2 Resource Definition [144](#resource-definition-52)

5.2.54.3 Resource Standard Methods [144](#resource-standard-methods-52)

5.2.54.3.1 GET [144](#get-52)

5.2.55 Resource: Individual5GmbsGroup [145](#resource-individual5gmbsgroup)

5.2.55.1 Description [145](#description-53)

5.2.55.2 Resource Definition [145](#resource-definition-53)

5.2.55.3 Resource Standard Methods [145](#resource-standard-methods-53)

5.2.55.3.1 PUT [145](#put-16)

5.2.55.3.2 DELETE [146](#delete-13)

5.2.55.3.3 PATCH [146](#patch-15)

5.2.55.3.4 GET [147](#get-53)

5.2.56 Resource: 5GMbsGroupsInternal [148](#resource-5gmbsgroupsinternal)

5.2.56.1 Description [148](#description-54)

5.2.56.2 Resource Definition [148](#resource-definition-54)

5.2.56.3 Resource Standard Methods [148](#resource-standard-methods-54)

5.2.56.3.1 GET [148](#get-54)

5.2.57 Resource: Pp5gMbsGroupProfileData [148](#resource-pp5gmbsgroupprofiledata)

5.2.57.1 Description [148](#description-55)

5.2.57.2 Resource Definition [149](#resource-definition-55)

5.2.57.3 Resource Standard Methods [149](#resource-standard-methods-55)

5.2.57.3.1 GET [149](#get-55)

5.2.58 Resource: UeUpdateConfirmation [149](#resource-ueupdateconfirmation)

5.2.58.1 Description [149](#description-56)

5.2.58.2 Resource Definition [149](#resource-definition-56)

5.2.58.3 Resource Standard Methods [150](#resource-standard-methods-56)

5.2.58.3.1 GET [150](#get-56)

5.2.59 Resource: LcsSubscriptionData [150](#resource-lcssubscriptiondata)

5.2.59.1 Description [150](#description-57)

5.2.59.2 Resource Definition [150](#resource-definition-57)

5.2.59.3 Resource Standard Methods [151](#resource-standard-methods-57)

5.2.59.3.1 GET [151](#get-57)

5.2.60 Resource: RangingSlPosSubscriptionData [151](#resource-rangingslpossubscriptiondata)

5.2.60.1 Description [151](#description-58)

5.2.60.2 Resource Definition [151](#resource-definition-58)

5.2.60.3 Resource Standard Methods [152](#resource-standard-methods-58)

5.2.60.3.1 GET [152](#get-58)

5.2.61 Resource: A2xSubscriptionData [152](#resource-a2xsubscriptiondata)

5.2.61.1 Description [152](#description-59)

5.2.61.2 Resource Definition [152](#resource-definition-59)

5.2.61.3 Resource Standard Methods [152](#resource-standard-methods-59)

5.2.62 Resource: RangingSlPrivacySubscriptionData [153](#resource-rangingslprivacysubscriptiondata)

5.2.62.1 Description [153](#description-60)

5.2.62.2 Resource Definition [153](#resource-definition-60)

5.2.62.3 Resource Standard Methods [153](#resource-standard-methods-60)

5.2.62.3.1 GET [153](#get-59)

5.3 Notifications [154](#notifications)

5.3.1 General [154](#general)

5.3.2 Data Change Notification [154](#data-change-notification)

5.3.3 Data Removal Notification [155](#data-removal-notification)

5.3.4 Stale Check Notification [156](#stale-check-notification)

5.4 Data Model [157](#data-model)

5.4.1 General [157](#general-1)

5.4.2 Structured data types [162](#structured-data-types)

5.4.2.1 Introduction [162](#introduction-1)

5.4.2.2 Type: AuthenticationSubscription [163](#type-authenticationsubscription)

5.4.2.3 Type: OperatorSpecificDataContainer [165](#type-operatorspecificdatacontainer)

5.4.2.4 Type: SmfRegList [166](#type-smfreglist)

5.4.2.5 Type: SubscriptionDataSubscriptions [167](#type-subscriptiondatasubscriptions)

5.4.2.6 Type: DataChangeNotify [170](#type-datachangenotify)

5.4.2.7 Type: IdentityData [171](#type-identitydata)

5.4.2.8 Type: ProvisionedDataSets [172](#type-provisioneddatasets)

5.4.2.9 Type: SorData [173](#type-sordata)

5.4.2.9A Type: UpuData [173](#a-type-upudata)

5.4.2.9B Type: NssaiAckData [173](#b-type-nssaiackdata)

5.4.2.9C Type: CagAckData [174](#c-type-cagackdata)

5.4.2.10 Void [174](#void)

5.4.2.11 Void [174](#void-1)

5.4.2.12 Void [174](#void-2)

5.4.2.13 Void [174](#void-3)

5.4.2.14 Void [174](#void-4)

5.4.2.15 Void [174](#void-5)

5.4.2.16 Void [174](#void-6)

5.4.2.17 Void [174](#void-7)

5.4.2.18 Void [174](#void-8)

5.4.2.19 Type: AmfSubscriptionInfo [175](#type-amfsubscriptioninfo)

5.4.2.20 Type: EeProfileData [175](#type-eeprofiledata)

5.4.2.21 Void [175](#void-9)

5.4.2.22 Type: ContextDataSets [176](#type-contextdatasets)

5.4.2.23 Type: SequenceNumber [177](#type-sequencenumber)

5.4.2.24 Type: MessageWaitingData [177](#type-messagewaitingdata)

5.4.2.25 Type: SmscData [178](#type-smscdata)

5.4.2.26 Type: SmfSubscriptionInfo [178](#type-smfsubscriptioninfo)

5.4.2.27 Type: SmfSubscriptionItem [178](#type-smfsubscriptionitem)

5.4.2.28 Type: MtcProvider [178](#type-mtcprovider)

5.4.2.29 Type: HssSubscriptionInfo [178](#type-hsssubscriptioninfo)

5.4.2.30 Type: HssSubscriptionItem [179](#type-hsssubscriptionitem)

5.4.2.31 Type: EeGroupProfileData [179](#type-eegroupprofiledata)

5.4.2.32 Type: Pp5gVnGroupProfileData [180](#type-pp5gvngroupprofiledata)

5.4.2.33 Type: PpProfileData [180](#type-ppprofiledata)

5.4.2.34 Type: AllowedMtcProviderInfo [180](#type-allowedmtcproviderinfo)

5.4.2.35 Type: GroupIdentifiers [181](#type-groupidentifiers)

5.4.2.36 Type: AuthorizationData [181](#type-authorizationdata)

5.4.2.37 Type: NiddAuthorizationInfo [181](#type-niddauthorizationinfo)

5.4.2.38 Type: PpDataEntryList [181](#type-ppdataentrylist)

5.4.2.39 Type: UeSubscribedDataSets [182](#type-uesubscribeddatasets)

5.4.2.40 Type: ServiceSpecificAuthorizationInfo [182](#type-servicespecificauthorizationinfo)

5.4.2.41 Type: NfIdentifier [182](#type-nfidentifier)

5.4.2.42 Type: EeSubscriptionExt [182](#type-eesubscriptionext)

5.4.2.43 Type: AdditionalEeSubsInfo [182](#type-additionaleesubsinfo)

5.4.2.44 Type: ImmediateReport [182](#type-immediatereport)

5.4.2.47 Type: UeUpdConfData [183](#type-ueupdconfdata)

5.4.2.48 Type: AdditionalDataRef [183](#type-additionaldataref)

5.4.3 Simple data types and enumerations [183](#simple-data-types-and-enumerations)

5.4.3.1 Introduction [183](#introduction-2)

5.4.3.2 Simple data types [183](#simple-data-types)

5.4.3.3 Enumeration: AuthMethod [183](#enumeration-authmethod)

5.4.3.4 Enumeration: ProvisionedDataSetName [184](#enumeration-provisioneddatasetname)

5.4.3.5 Void [184](#void-10)

5.4.3.6 Enumeration: ContextDataSetName [184](#enumeration-contextdatasetname)

5.4.3.7 Enumeration: SqnScheme [185](#enumeration-sqnscheme)

5.4.3.8 Enumeration: Sign [185](#enumeration-sign)

5.4.3.9 Enumeration: UeUpdateStatus [185](#enumeration-ueupdatestatus)

5.4.3.10 Enumeration: PpDataType [186](#enumeration-ppdatatype)

5.4.3.11 Enumeration: UeSubscribedDataSetName [186](#enumeration-uesubscribeddatasetname)

5.4.4 Binary data [186](#binary-data)

5.5 Error handling [186](#error-handling)

5.6 Feature negotiation [186](#feature-negotiation)

Annex A (normative): OpenAPI specification [187](#annex-a-normative-openapi-specification)

A.1 General [187](#a.1-general)

A.2 Nudr_DataRepository API for Subscription Data [187](#a.2-nudr_datarepository-api-for-subscription-data)

Annex B (informative): Change history [333](#annex-b-informative-change-history)
