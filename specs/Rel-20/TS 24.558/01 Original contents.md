---
spec: TS 24.558
version: 20.0.0
release: '20'
clause: contents
title: Contents
source_archive: 24558-k00.zip
source_document: 24558-k00.docx
content_origin: 3gpp-source
---

# Contents

Foreword 10

1 Scope 12

2 References 12

3 Definitions of terms, symbols and abbreviations 13

3.1 Terms 13

3.2 Symbols 13

3.3 Abbreviations 13

4 Overview 13

4.0 General 13

4.1 Information applicable to APIs over EDGE-1 and EDGE-4 14

5 Services offered by Edge Enabler Server 14

5.1 Introduction 14

5.2 Eees_EECRegistration Service 15

5.2.1 Service Description 15

5.2.2 Service Operations 15

5.2.2.1 Introduction 15

5.2.2.2 Eees_EECRegistration_Request 15

5.2.2.2.1 General 15

5.2.2.2.2 EEC registering to EES using Eees_EECRegistration_Request operation 15

5.2.2.3 Eees_EECRegistration_Update 17

5.2.2.3.1 General 17

5.2.2.3.2 EEC updating registration information using Eees_EECRegistration_Update operation 17

5.2.2.4 Eees_EECRegistration_Deregister 18

5.2.2.4.1 General 18

5.2.2.4.2 EEC deregistering from EES using Eees_EECRegistration_Deregister operation 18

5.3 Eees_EASDiscovery service 19

5.3.1 Service Description 19

5.3.2 Service Operations 19

5.3.2.1 Introduction 19

5.3.2.2 Eees_EASDiscovery_EasDiscRequest 19

5.3.2.2.1 General 19

5.3.2.2.2 EEC requesting EAS discovery information using Eees_EASDiscovery_EasDiscRequest operation 19

5.3.2.3 Eees_EASDiscovery_Subscribe 22

5.3.2.3.1 General 22

5.3.2.3.2 EEC subscribing to EAS discovery information from EES using Eees_EASDiscovery_Subscribe operation 22

5.3.2.4 Eees_EASDiscovery_Notify 22

5.3.2.4.1 General 22

5.3.2.4.2 EES notifying the EAS discovery information to EEC using Eees_EASDiscovery_Notify operation 23

5.3.2.5 Eees_EASDiscovery_UpdateSubscription 23

5.3.2.5.1 General 23

5.3.2.5.2 EEC updating EAS discovery information subscription at EES using Eees_EASDiscovery_UpdateSubscription operation 23

5.3.2.6 Eees_EASDiscovery_Unsubscribe 24

5.3.2.6.1 General 24

5.3.2.6.2 EEC unsubscribing to EAS discovery subscription from EES using Eees_EASDiscovery_Unsubscribe operation 24

5.4 Eees_ACREvents Service 24

5.4.1 Service Description 24

5.4.2 Service Operations 24

5.4.2.1 Introduction 24

5.4.2.2 Eees_ACREvents_Subscribe 25

5.4.2.2.1 General 25

5.4.2.2.2 EEC subscribing to ACR information from EES using Eees_ACREvents_Subscribe operation 25

5.4.2.3 Eees_ACREvents_Notify 25

5.4.2.3.1 General 25

5.4.2.3.2 EES notifying the ACR information to EEC using Eees_ACREvents_Notify operation 26

5.4.2.4 Eees_ACREvents_UpdateSubscription 26

5.4.2.4.1 General 26

5.4.2.4.3 EEC updating ACR information subscription at EES using Eees_ACREvents_UpdateSubscription operation 26

5.4.2.5 Eees_ACREvents_Unsubscribe 27

5.4.2.5.1 General 27

5.4.2.5.2 EEC unsubscribing to service provisioning subscription from EES using Eees_ACREvents_Unsubscribe operation 27

5.5 Eees_AppContextRelocation Service 27

5.5.1 Service Description 27

5.5.2 Service Operations 27

5.5.2.1 Introduction 27

5.5.2.2 Eees_AppContextRelocation_Determine 28

5.5.2.2.1 General 28

5.5.2.2.2 ACR Determination 28

5.5.2.3 Eees_AppContextRelocation_Initiate 28

5.5.2.3.1 General 28

5.5.2.3.2 ACR Initiation 28

5.6 Eees_UEIdentifier Service 29

5.6.1 Service Description 29

5.6.2 Service Operations 30

5.6.2.1 Introduction 30

5.6.2.2 Eees_UEIdentifier_Get 30

5.6.2.2.1 General 30

5.6.2.2.2 Retrieve UE identifier 30

5.7 Eees_EASInformationProvisioning Service 30

5.7.1 Service Description 30

5.7.2 Service Operations 31

5.7.2.1 Introduction 31

5.7.2.2 Eees_EASInformationProvisioning_Declare 31

5.7.2.2.1 General 31

5.7.2.2.2 EEC exchanging EAS information in EES using Eees_EASInformationProvisioning_Declare operation 31

6 Edge Enabler Server API Definitions 32

6.1 Void 32

6.2 Eees_EECRegistration API 32

6.2.1 API URI 32

6.2.2 Resources 33

6.2.2.1 Overview 33

6.2.2.2 Resource: EEC Registrations 33

6.2.2.2.1 Description 33

6.2.2.2.2 Resource Definition 33

6.2.2.2.3 Resource Standard Methods 34

6.2.2.2.4 Resource Custom Operations 34

6.2.2.3 Resource: Individual EEC registration 34

6.2.2.3.1 Description 34

6.2.2.3.2 Resource Definition 34

6.2.2.3.3 Resource Standard Methods 35

6.2.2.3.4 Resource Custom Operations 40

6.2.3 Custom Operations without associated resources 40

6.2.4 Notifications 40

6.2.5 Data Model 40

6.2.5.1 General 40

6.2.5.2 Structured data types 41

6.2.5.2.1 Introduction 41

6.2.5.2.2 Type: EECRegistration 42

6.2.5.2.3 Type: ACProfile 44

6.2.5.2.4 Type: EasDetail 44

6.2.5.2.5 Type: ACServiceKPIs 45

6.2.5.2.6 Type: EECRegistrationPatch 45

6.2.5.2.7 Type: UnfulfilledAcProfile 45

6.2.5.3 Simple data types and enumerations 46

6.2.5.3.1 Introduction 46

6.2.5.3.2 Simple data types 46

6.2.5.3.3 Enumeration: UnfulfillACProfRsn 46

6.2.5.3.4 Enumeration: DeviceType 46

6.2.6 Error Handling 46

6.2.6.0 General 46

6.2.6.1 Application Errors 46

6.2.7 Feature negotiation 47

6.3 Eees_EASDiscovery API 47

6.3.1 API URI 47

6.3.2 Resources 48

6.3.2.1 Overview 48

6.3.2.2 Resource: EAS Discovery Subscriptions 49

6.3.2.2.1 Description 49

6.3.2.2.2 Resource Definition 49

6.3.2.2.3 Resource Standard Methods 49

6.3.2.2.4 Resource Custom Operations 50

6.3.2.3 Resource: Individual EAS Discovery Subscription 50

6.3.2.3.1 Description 50

6.3.2.3.2 Resource Definition 50

6.3.2.3.3 Resource Standard Methods 50

6.3.2.3.4 Resource Custom Operations 53

6.3.2.4 Resource: EAS Profiles 53

6.3.2.4.1 Description 53

6.3.2.4.2 Resource Definition 53

6.3.2.4.3 Resource Standard Methods 54

6.3.2.4.4 Resource Custom Operations 54

6.3.3 Custom operations without associated resources 55

6.3.4 Notifications 55

6.3.4.1 General 55

6.3.4.2 EAS Discovery Notification 55

6.3.4.2.1 Description 55

6.3.4.2.2 Target URI 55

6.3.4.2.3 Standard Methods 56

6.3.5 Data Model 56

6.3.5.1 General 56

6.3.5.2 Structured data types 58

6.3.5.2.1 Introduction 58

6.3.5.2.2 Type: EasDiscoveryReq 59

6.3.5.2.3 Type: EasDiscoveryResp 61

6.3.5.2.4 Type: EasDiscoverySubscription 62

6.3.5.2.5 Type: EasDiscoveryNotification 64

6.3.5.2.6 Type: EasDiscoveryFilter 64

6.3.5.2.7 Type: EasCharacteristics 65

6.3.5.2.8 Type: DiscoveredEas 66

6.3.5.2.9 Type: EasDynamicInfoFilter 66

6.3.5.2.10 Type: EasDynamicInfoFilterData 67

6.3.5.2.11 Type: ACCharacteristics 69

6.3.5.2.12 Type: EasDiscoverySubscriptionPatch 69

6.3.5.2.13 Type: RequestorId 69

6.3.5.2.14 Type: EdgeLoadAnalytic 70

6.3.5.2.15 Type: PredictiveData 70

6.3.5.2.16 Type: StatisticalData 70

6.3.5.3 Simple data types and enumerations 70

6.3.5.3.1 Introduction 70

6.3.5.3.2 Simple data types 70

6.3.5.3.3 Enumeration: EASDiscEventIDs 71

6.3.6 Error Handling 71

6.3.6.1 General 71

6.3.6.2 Protocol Errors 71

6.3.6.3 Application Errors 71

6.3.7 Feature negotiation 71

6.4 Eees_ACREvents API 72

6.4.1 API URI 72

6.4.2 Resources 73

6.4.2.1 Overview 73

6.4.2.2 Resource: ACR events subscriptions 73

6.4.2.2.1 Description 73

6.4.2.2.2 Resource Definition 73

6.4.2.2.3 Resource Standard Methods 74

6.4.2.2.4 Resource Custom Operations 74

6.4.2.3 Resource: Individual ACR events subscription 74

6.4.2.3.1 Description 74

6.4.2.3.2 Resource Definition 74

6.4.2.3.3 Resource Standard Methods 75

6.4.2.3.4 Resource Custom Operations 80

6.4.3 Custom operations without associated resources 80

6.4.4 Notifications 80

6.4.4.1 General 80

6.4.4.2 ACR Information Notification 80

6.4.4.2.1 Description 80

6.4.4.2.2 Target URI 80

6.4.4.2.3 Standard Methods 80

6.4.5 Data Model 81

6.4.5.1 General 81

6.4.5.2 Structured data types 82

6.4.5.2.1 Introduction 82

6.4.5.2.2 Type: ACREventsSubscription 83

6.4.5.2.3 Type: ACRInfoNotification 84

6.4.5.2.4 Type: TargetInfo 84

6.4.5.2.5 Type: ACREventsSubscriptionPatch 85

6.4.5.2.6 Type: EecCtxtRelocStatus 85

6.4.5.2.7 Type: ACRCompleteEventInfo 85

6.4.5.3 Simple data types and enumerations 85

6.4.5.3.1 Introduction 85

6.4.5.3.2 Simple data types 85

6.4.5.3.3 Enumeration: ACREventIDs 86

6.4.6 Error Handling 86

6.4.7 Feature negotiation 86

6.5 Eees_AppContextRelocation API 86

6.5.1 Introduction 86

6.5.3 Custom Operations without associated resources 87

6.5.3.1 Overview 87

6.5.3.2 Operation: Determine 88

6.5.3.2.1 Description 88

6.5.3.2.2 Operation Definition 88

6.5.3.3 Operation: Initiate 88

6.5.3.3.1 Description 88

6.5.3.3.2 Operation Definition 88

6.5.3.4 Operation: Declare 89

6.5.3.4.1 Description 89

6.5.3.4.2 Operation Definition 89

6.5.4 Notifications 90

6.5.5 Data Model 90

6.5.5.1 General 90

6.5.5.2 Structured data types 91

6.5.5.2.1 Introduction 91

6.5.5.2.2 Type: AcrDetermReq 91

6.5.5.2.3 Type: AcrInitReq 92

6.5.5.2.4 Type: AcrDecReq 94

6.5.5.2.5 Type: EecCtxtReloc 95

6.5.5.2.6 Type: ExpectedLocationArea 95

6.5.5.2.7 Type: AcrParameters 95

6.5.5.2.8 Type: AcrModificationParams 96

6.5.5.3 Simple data types and enumerations 96

6.5.5.3.1 Introduction 96

6.5.5.3.2 Simple data types 96

6.5.6 Error Handling 96

6.5.7 Feature negotiation 96

6.6 Eees_EASInformationProvisioning API 97

6.6.1 Introduction 97

6.6.2 Resources 97

6.6.3 Custom operations without associated resources 97

6.6.3.1 Overview 97

6.6.3.2 Operation: Declare 98

6.6.3.2.1 Description 98

6.6.3.2.2 Operation Definition 98

6.6.3.3 Void 99

6.6.3.4 Void 99

6.6.4 Notifications 99

6.6.5 Data Model 99

6.6.5.1 General 99

6.6.5.2 Structured data types 99

6.6.5.2.1 Introduction 99

6.6.5.2.2 Type: EASInfoProvReq 100

6.6.5.2.3 Type: EASInfoProvResp 101

6.6.5.2.4 Type: InstantiatedEASInfo 101

6.6.5.2.5 Void 101

6.6.5.2.6 Void 101

6.6.5.2.7 Void 101

6.6.5.3 Simple data types and enumerations 101

6.6.5.3.1 Introduction 101

6.6.5.3.2 Simple data types 101

6.6.5.3.3 Enumeration: EasInfoProvReqType 102

6.6.6 Error Handling 102

6.6.7 Feature negotiation 102

7 Services offered by Edge Configuration Server 102

7.1 Introduction 102

7.2 Eecs_ServiceProvisioning Service 103

7.2.1 Service Description 103

7.2.2 Service Operations 103

7.2.2.1 Introduction 103

7.2.2.2 Eecs_ServiceProvisioning_Request 103

7.2.2.2.1 General 103

7.2.2.2.2 EEC requesting service provisioning information using Eecs_ServiceProvisioning_Request operation 103

7.2.2.3 Eecs_ServiceProvisioning_Subscribe 105

7.2.2.3.1 General 105

7.2.2.3.2 EEC subscribing to service provisioning information from ECS using Eecs_ServiceProvisioning_Subscribe operation 106

7.2.2.4 Eecs_ServiceProvisioning_Notify 106

7.2.2.4.1 General 106

7.2.2.4.2 ECS notifying the service provisioning information to EEC using Eecs_ServiceProvisioning_Notify operation 106

7.2.2.5 Eecs_ServiceProvisioning_UpdateSubscription 107

7.2.2.5.1 General 107

7.2.2.5.2 EEC updating service provisioning information subscription at ECS using Eecs_ServiceProvisioning_UpdateSubscription operation 107

7.2.2.6 Eecs_ServiceProvisioning_Unsubscribe 108

7.2.2.6.1 General 108

7.2.2.6.2 EEC unsubscribing to service provisioning subscription from ECS using Eecs_ServiceProvisioning_Unsubscribe operation 108

8 Edge Configuration Server API Definitions 108

8.1 Eecs_ServiceProvisioning API 108

8.1.1 Introduction 108

8.1.2 Resources 109

8.1.2.1 Overview 109

8.1.2.3 Resource: Service Provisioning Subscriptions 109

8.1.2.3.1 Description 109

8.1.2.3.2 Resource Definition 110

8.1.2.3.3 Resource Standard Methods 110

8.1.2.3.4 Resource Custom Operations 110

8.1.2.4 Resource: Individual Service Provisioning Subscription 111

8.1.2.4.1 Description 111

8.1.2.4.2 Resource Definition 111

8.1.2.4.3 Resource Standard Methods 111

8.1.3 Custom Operations without associated resources 114

8.1.3.1 Overview 114

8.1.3.2 Operation: Request 115

8.1.3.2.1 Description 115

8.1.3.2.2 Operation Definition 115

8.1.4 Notifications 116

8.1.4.1 General 116

8.1.4.2 Service Provisioning Notification 116

8.1.4.2.1 Description 116

8.1.4.2.2 Notification definition 116

8.1.5 Data Model 117

8.1.5.1 General 117

8.1.5.2 Structured data types 119

8.1.5.2.1 Introduction 119

8.1.5.2.2 Type: ECSServProvReq 119

8.1.5.2.3 Type: ECSServProvResp 119

8.1.5.2.4 Type: ECSServProvSubscription 120

8.1.5.2.5 Type: ConnectivityInfo 122

8.1.5.2.6 Type: ServProvNotification 122

8.1.5.2.7 Type: EDNConfigInfo 122

8.1.5.2.8 Type: EDNConInfo 122

8.1.5.2.9 Type: EESInfo 123

8.1.5.2.10 Type: ECSServProvSubscriptionPatch 124

8.1.5.2.11 Void 124

8.1.5.2.12 Type: ECSRedirectInfo 124

8.1.5.2.13 Type: AppGroupProfile 124

8.1.5.2.14 Type: ApplicationInfo 125

8.1.5.2.15 Type: EASBundleDetail 125

8.1.5.3 Simple data types and enumerations 125

8.1.5.3.1 Introduction 125

8.1.5.3.2 Simple data types 125

8.1.5.3.3 Enumeration: EesAuthMethod 125

8.1.6 Error Handling 126

8.1.7 Feature negotiation 126

9 Security 127

10 SEAL services 127

Annex A (normative): Edge Enabler Server OpenAPI specification 128

A.1 General 128

A.2 Eees_EECRegistration API 128

A.3 Eees_EASDiscovery API 134

A.4 Eees_ACREvents API 144

A.5 Eees_AppContextRelocation API 149

A.6 Eees_EASInformationProvisioning API 154

Annex B (normative): Edge Configuration Server OpenAPI specification 158

B.1 Eecs_ServiceProvisioning 158

Annex C (informative): Protocol options considered for EDGE-4 reference point 168

Annex D(informative): Change history 169
