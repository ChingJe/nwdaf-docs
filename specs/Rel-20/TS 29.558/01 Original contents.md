---
spec: TS 29.558
version: 20.0.0
release: '20'
clause: contents
title: Contents
source_archive: 29558-k00.zip
source_document: 29558-k00.docx
content_origin: 3gpp-source
---

# Contents

Foreword 21

1 Scope 23

2 References 23

3 Definitions of terms, symbols and abbreviations 24

3.1 Terms 24

3.2 Symbols 24

3.3 Abbreviations 24

4 Overview 25

5 Services offered by Edge Enabler Server 25

5.1 Introduction 25

5.2 Eees_EASRegistration Service 27

5.2.1 Service Description 27

5.2.2 Service Operations 28

5.2.2.1 Introduction 28

5.2.2.2 Eees_EASRegistration_Request 28

5.2.2.2.1 General 28

5.2.2.2.2 Service consumer registering to EES using Eees_EASRegistration_Request operation 28

5.2.2.3 Eees_EASRegistration_Update 28

5.2.2.3.1 General 28

5.2.2.3.2 Service consumer updating registration information using Eees_EASRegistration_Update operation 29

5.2.2.4 Eees_EASRegistration_Deregister 29

5.2.2.4.1 General 29

5.2.2.4.2 Service consumer deregistering from EES using Eees_EASRegistration_Deregister operation 29

5.3 Eees_UELocation Service 30

5.3.1 Service Description 30

5.3.2 Service Operations 30

5.3.2.1 Introduction 30

5.3.2.2 Eees_UELocation_Get 30

5.3.2.2.1 General 30

5.3.2.2.2 Service consumer obtaining UE location information from EES using Eees_UELocation_Get operation 30

5.3.2.2.3 User consent management 31

5.3.2.3 Eees_UELocation_Subscribe 31

5.3.2.3.1 General 31

5.3.2.3.2 Service consumer subscribing to continuous UE(s) location reporting from EES using Eees_UELocation_Subscribe operation 31

5.3.2.3.3 User consent management 32

5.3.2.4 Eees_UELocation_Notify 33

5.3.2.4.1 General 33

5.3.2.4.2 EES notifying the UE(s) location reporting to service consumer using Eees_UELocation_Notify operation 33

5.3.2.4.3 EES notifying the service consumer about user consent revocation using Eees_UELocation_Notify operation 33

5.3.2.5 Eees_UELocation_UpdateSubscription 34

5.3.2.5.1 General 34

5.3.2.5.2 Service consumer updating continuous UE(s) location reporting subscription at EES using Eees_UELocation_UpdateSubscribe operation 34

5.3.2.5.3 User consent management 34

5.3.2.6 Eees_UELocation_Unsubscribe 35

5.3.2.6.1 General 35

5.3.2.6.2 Service consumer unsubscribing to continuous UE(s) location reporting from EES using Eees_UELocation_Unsubscribe operation 35

5.4 Eees_UEIdentifier Service 36

5.4.1 Service Description 36

5.4.2 Service Operations 36

5.4.2.1 Introduction 36

5.4.2.2 Eees_UEIdentifier_Get 36

5.4.2.2.1 General 36

5.4.2.2.1A Service consumer obtaining UE Identifier Information using the "Get" custom operation 36

5.4.2.2.2 EAS obtaining UE identifier from EES using Eees_UEIdentifier_Fetch custom operation 37

5.5 Eees_AppClientInformation Service 37

5.5.1 Service Description 37

5.5.2 Service Operations 38

5.5.2.1 Introduction 38

5.5.2.2 Eees_AppClientInformation_Subscribe 38

5.5.2.2.1 General 38

5.5.2.2.2 Service consumer subscribing to AC information reporting from the EES using the Eees_AppClientInformation_Subscribe operation 38

5.5.2.3 Eees_AppClientInformation_Notify 39

5.5.2.3.1 General 39

5.5.2.3.2 EES notifying the AC information to the service consumer using Eees_AppClientInformation_Notify operation 39

5.5.2.4 Eees_AppClientInformation_UpdateSubscription 39

5.5.2.4.1 General 39

5.5.2.4.2 Service consumer updating AC information reporting subscription at the EES using Eees_AppClientInformation_UpdateSubscribe operation 39

5.5.2.5 Eees_AppClientInformation_Unsubscribe 40

5.5.2.5.1 General 40

5.5.2.5.2 Service consumer unsubscribing to AC information reporting from the EES using Eees_AppClientInformation_Unsubscribe operation 40

5.6 Eees_SessionWithQoS Service 41

5.6.1 Service Description 41

5.6.2 Service Operations 41

5.6.2.1 Introduction 41

5.6.2.2 Eees_SessionWithQoS_Create 41

5.6.2.2.1 General 41

5.6.2.2.2 Service consumer requesting reservation of resources for a data session between AC and service consumer with specific QoS using Eees_SessionWithQoS operation 41

5.6.2.3 Eees_SessionWithQoS_Update 42

5.6.2.3.1 General 42

5.6.2.3.2 Service consumer updating QoS of a data session between AC and service consumer using Eees_SessionWithQoS_Update operation 42

5.6.2.4 Eees_SessionWithQoS_Revoke 43

5.6.2.4.1 General 43

5.6.2.4.2 Service consumer revoking QoS of a data session between AC and service consumer using Eees_SessionWithQoS_Revoke operation 43

5.6.2.5 Eees_SessionWithQoS_Notify 44

5.6.2.5.1 General 44

5.6.2.5.2 EES notifying QoS of a data session between AC and service consumer using Eees_SessionWithQoS_Notify operation 44

5.7 Eees_EASDiscovery Service 44

5.7.1 Service Description 44

5.7.2 Service Operations 44

5.7.2.1 Introduction 44

5.7.2.2 Eees_EASDiscovery_TEasDiscRequest 45

5.7.2.2.1 General 45

5.7.2.2.2 Service consumer requesting T-EAS discovery information using Eees_EASDiscovery_TEasDiscRequest operation 45

5.8 Eees_ACRManagementEvent Service 45

5.8.1 Service Description 45

5.8.2 Service Operations 45

5.8.2.1 Introduction 45

5.8.2.2 Eees_ACRManagementEvent_Subscribe 46

5.8.2.2.1 General 46

5.8.2.2.2 Service consumer requesting to get notifications of ACR management events using Eees_ACRManagementEvent_Subscribe service operation 46

5.8.2.3 Eees_ACRManagementEvent_UpdateSubscription 46

5.8.2.3.1 General 46

5.8.2.3.2 Service consumer updating an existing Individual ACR Management Events Subscription using Eees_ACRManagementEvent_UpdateSubscription service operation 47

5.8.2.4 Eees_ACRManagementEvent_Unsubscribe 47

5.8.2.4.1 General 47

5.8.2.4.2 Service consumer deleting an existing Individual ACR Management Events Subscription using Eees_ACRManagementEvent_Unsubscribe service operation 47

5.8.2.5 Eees_ACRManagementEvent_Notify 48

5.8.2.5.1 General 48

5.8.2.5.2 EES notifying ACR management events using Eees_ACRManagementEvent_Notify operation 48

5.8.2.5.3 EES notifying the availability of user path management events monitoring via the 3GPP 5GC network using Eees_ACRManagementEvent_Notify operation 48

5.9 Eees_AppContextRelocation Service 49

5.9.1 Service Description 49

5.9.2 Service Operations 49

5.9.2.1 Introduction 49

5.9.2.2 Eees_AppContextRelocation_SelectedTargetEAS_Declare 49

5.9.2.2.1 General 49

5.9.2.2.2 S-EAS informing the S-EES about the selected T-EAS using Eees_AppContextRelocation_SelectedTargetEAS_Declare operation 49

5.9.2.3 Eees_AppContextRelocation_ACRDetermination_Request 50

5.9.2.3.1 General 50

5.9.2.3.2 Request the S-EES to determine the ACR using Eees_AppContextRelocation_ACRDetermination_Request operation 50

5.10 Eees_EECContextRelocation Service 50

5.10.1 Service Description 50

5.10.2 Service Operations 50

5.10.2.1 Introduction 50

5.10.2.2 Eees_EECContextRelocation_Pull 51

5.10.2.2.1 General 51

5.10.2.2.2 Service consumer pulling the EEC context information from the EES using the Eees_EECContextRelocation_Pull operation 51

5.10.2.3 Eees_EECContextRelocation_Push 51

5.10.2.3.1 General 51

5.10.2.3.2 Service consumer pushing the EEC context information to the EES using the Eees_EECContextRelocation_Push operation 51

5.11 Eees_EELManagedACR Service 52

5.11.1 Service Description 52

5.11.2 Service Operations 52

5.11.2.1 Introduction 52

5.11.2.2 Eees_EELManagedACR_Request 52

5.11.2.2.1 General 52

5.11.2.2.2 EEL Managed ACR Request 52

5.11.2.3 Eees_EELManagedACR_Subscribe 53

5.11.2.3.1 General 53

5.11.2.3.2 Subscribe to ACT status information reporting 53

5.11.2.4 Eees_EELManagedACR_Notify 53

5.11.2.4.1 General 53

5.11.2.4.2 ACT Status Notification 53

5.12 Eees_ACRStatusUpdate Service 54

5.12.1 Service Description 54

5.12.2 Service Operations 54

5.12.2.1 Introduction 54

5.12.2.2 Eees_ACRStatusUpdate_Request 54

5.12.2.2.1 General 54

5.12.2.2.2 ACR Status Update Request 54

5.13 Eees_ACRParameterInformation Service 55

5.13.1 Service Description 55

5.13.2 Service Operations 55

5.13.2.1 Introduction 55

5.13.2.2 Eees_ACRParameterInformation_Request 55

5.13.2.2.1 General 55

5.13.2.2.2 ACR Parameters Information Request 55

5.14 Eees_CommonEASAnnouncement Service 55

5.14.1 Service Description 55

5.14.2 Service Operations 56

5.14.2.1 Introduction 56

5.14.2.2 Eees_CommonEASAnnouncement_Declare 56

5.14.2.2.1 General 56

5.14.2.2.2 Common EAS Information Declaration 56

5.15 Eees_TrafficInfluenceEAS Service 56

5.15.1 Service Description 56

5.15.2 Service Operations 57

5.15.2.1 Introduction 57

5.15.2.2 Eees_TrafficInfluenceEAS_Manage 57

5.15.2.2.1 General 57

5.15.2.2.2 Application Traffic Influence Initiation 57

5.15.2.2.3 Application Traffic Influence Update 57

5.15.2.2.4 Application Traffic Influence Cancellation 58

5.15.2.2.5 Application Traffic Influence Retrieval 58

5.16 Eees_EASInformationProvisioning Service 59

5.16.1 Service Description 59

5.16.2 Service Operations 59

5.16.2.1 Introduction 59

5.16.2.2 Eees_EASInformationProvisioning_Declare 59

5.16.2.2.1 General 59

5.16.2.2.2 EAS Information Provisioning Information Declaration 59

6 Services offered by Edge Configuration Server 60

6.1 Introduction 60

6.2 Eecs_EESRegistration Service 61

6.2.1 Service Description 61

6.2.2 Service Operations 61

6.2.2.1 Introduction 61

6.2.2.2 Eecs_EESRegistration_Request 61

6.2.2.2.1 General 61

6.2.2.2.2 EES registering to ECS using Eecs_EESRegistration_Request operation 61

6.2.2.3 Eecs_EESRegistration_Update 62

6.2.2.3.1 General 62

6.2.2.3.2 EES updating registration information using Eecs_EESRegistration_Update operation 62

6.2.2.4 Eecs_EESRegistration_Deregister 62

6.2.2.4.1 General 62

6.2.2.4.2 EES deregistering from ECS using Eecs_EESRegistration_Deregister operation 63

6.3 Eecs_TargetEESDiscovery Service 63

6.3.1 Service Description 63

6.3.2 Service Operations 63

6.3.2.1 Introduction 63

6.3.2.2 Eecs_TargetEESDiscovery_Request 63

6.3.2.2.1 General 63

6.3.2.2.2 Service consumer fetching the target Enabler Server information from the ECS using Eecs_TargetEESDiscovery_Request operation 63

6.4 Eecs_EASInfoManagement Service 64

6.4.1 Service Description 64

6.4.2 Service Operations 64

6.4.2.1 Introduction 64

6.4.2.2 Eecs_EASInfoManagement_Get 65

6.4.2.2.1 General 65

6.4.2.2.2 Common EAS Binding Information Retrieval 65

6.4.2.3 Eecs_EASInfoManagement_Store 65

6.4.2.3.1 General 65

6.4.2.3.2 Common EAS Binding Information Storage 65

6.4.2.4 Eecs_EASInfoManagement_Update 66

6.4.2.4.1 General 66

6.4.2.4.2 Common EAS Binding Information Update 66

6.4.2.5 Eecs_EASInfoManagement_Delete 66

6.4.2.5.1 General 66

6.4.2.5.2 Common EAS Binding information Deletion 67

6.5 Eecs_ECSServiceProvisioning 67

6.5.1 Service Description 67

6.5.2.1 Introduction 67

6.5.2.2 Eecs_ECSServiceProvisioning_Request 68

6.5.2.2.1 General 68

6.5.2.2.2 Service Provisioning Information Retrieval Request 68

6.5.2.3 Eecs_ECSServiceProvisioning_Subscribe 68

6.5.2.3.1 General 68

6.5.2.3.2 Service Provisioning Subscription Creation 68

6.5.2.4 Eecs_ECSServiceProvisioning_UpdateSubscription 68

6.5.2.4.1 General 68

6.5.2.4.2 Service Provisioning Subscription Update 69

6.5.2.5 Eecs_ECSServiceProvisioning_Unsubscribe 69

6.5.2.5.1 General 69

6.5.2.5.2 Service Provisioning Subscription Deletion 69

6.5.2.6 Eecs_ECSServiceProvisioning_Notify 69

6.5.2.6.1 General 69

6.5.2.6.2 Service Provisioning Notification 70

6.6 Eecs_ECSDiscovery Service 70

6.6.1 Service Description 70

6.6.2 Service Operations 70

6.6.2.1 Introduction 70

6.6.2.2 Eecs_ECSDiscovery_Request 70

6.6.2.2.1 General 70

6.6.2.2.2 Service consumer fetching partner ECS information from the ECS using the Eecs_ECSDiscovery_Request operation 70

6.6.2.3 Eecs_ECSDiscovery_Request 71

6.6.2.3.1 General 71

6.6.2.3.2 ECS notifying the ECS information to the service consumer using Eecs_ECSDiscovery_Notification operation 71

6.7 Eecs_ACREvents 71

6.7.1 Service Description 71

6.7.2 Service Operations 71

6.7.2.1 Introduction 71

6.7.2.2 Eecs_ACREvents_Subscribe 72

6.7.2.2.1 General 72

6.7.2.2.2 ACR Events Subscription Creation 72

6.7.2.3 Eecs_ACREvents_Update 72

6.7.2.3.1 General 72

6.7.2.3.2 ACR Events Subscription Update 72

6.7.2.4 Eecs_ACREvents_Unsubscribe 73

6.7.2.4.1 General 73

6.7.2.4.2 ACR Events Subscription Deletion 73

6.7.2.5 Eecs_ACREvents_Notify 73

6.7.2.5.1 General 73

6.7.2.5.2 ACR Events Notification 74

6A Services offered by the Cloud Application Server (CAS) 74

6A.1 Introduction 74

6A.2 Ecas_SelectedEES Service 74

6A.2.1 Service Description 74

6A.2.2 Service Operations 75

6A.2.2.1 Introduction 75

6A.2.2.2 Ecas_SelectedEES_Request 75

6A.2.2.2.1 General 75

6A.2.2.2.2 Service consumer informing the CAS of the selected EES using Ecas_SelectedEES_Declare operation 75

6B Services offered by the Cloud Enabler Server (CES) 75

6B.1 Introduction 75

7 Information applicable to several APIs 76

7.1 General 76

7.2 Data Types 76

7.2.1 General 76

7.2.2 Referenced structured data types 77

7.2.3 Referenced simple data types and enumerations 77

7.3 Usage of HTTP 77

7.4 Content type 77

7.5 URI structure 77

7.5.1 Resource URI structure 77

7.5.2 Custom operations URI structure 78

7.6 Notifications 78

7.7 Error handling 78

7.8 Feature negotiation 78

7.9 HTTP headers 78

7.10 Conventions for Open API specification files 79

8 Edge Enabler Server API Definitions 79

8.1 Eees_EASRegistration API 79

8.1.1 Introduction 79

8.1.2 Resources 79

8.1.2.1 Overview 79

8.1.2.2 Resource: EAS Registrations 80

8.1.2.2.1 Description 80

8.1.2.2.2 Resource Definition 80

8.1.2.2.3 Resource Standard Methods 80

8.1.2.2.4 Resource Custom Operations 81

8.1.2.3 Resource: Individual EAS Registration 81

8.1.2.3.1 Description 81

8.1.2.3.2 Resource Definition 81

8.1.2.3.3 Resource Standard Methods 81

8.1.2.3.4 Resource Custom Operations 85

8.1.3 Custom Operations without associated resources 85

8.1.4 Notifications 86

8.1.5 Data Model 86

8.1.5.1 General 86

8.1.5.2 Structured data types 87

8.1.5.2.1 Introduction 87

8.1.5.2.2 Type: EASRegistration 87

8.1.5.2.3 Type: EASProfile 88

8.1.5.2.4 Type: EASServiceKPI 91

8.1.5.2.5 Type: EndPoint 91

8.1.5.2.6 Type: EASRegistrationPatch 92

8.1.5.2.7 Type: TransContSuppDetails 92

8.1.5.2.8 Type: EASBundleInfo 92

8.1.5.2.9 Type: EASBdlReqs 93

8.1.5.2.10 Type: CoordinatedAcrReqs 93

8.1.5.2.11 Type: AssociatedDevice 94

8.1.5.3 Simple data types and enumerations 94

8.1.5.3.1 Introduction 94

8.1.5.3.2 Simple data types 94

8.1.5.3.3 Enumeration: PermissionLevel 94

8.1.5.3.4 Enumeration: EASCategory 94

8.1.5.3.5 Enumeration: TransportProtocol 95

8.1.5.3.6 Enumeration: BdlType 95

8.1.5.3.7 Enumeration: Affinity 95

8.1.5.3.8 Enumeration: FailureAction 95

8.1.5.3.9 Enumeration: EASStatus 95

8.1.5.3.10 Enumeration: DeviceType 96

8.1.6 Error Handling 96

8.1.6.1 General 96

8.1.6.2 Protocol Errors 96

8.1.6.3 Application Errors 96

8.1.7 Feature negotiation 96

8.2 Eees_UELocation API 97

8.2.1 Introduction 97

8.2.2 Resources 97

8.2.2.1 Overview 97

8.2.2.2 Resource: Location Information Subscriptions 98

8.2.2.2.1 Description 98

8.2.2.2.2 Resource Definition 98

8.2.2.2.3 Resource Standard Methods 99

8.2.2.2.4 Resource Custom Operations 99

8.2.2.3 Resource: Individual Location Information Subscription 100

8.2.2.3.1 Description 100

8.2.2.3.2 Resource Definition 100

8.2.2.3.3 Resource Standard Methods 100

8.2.2.3.4 Resource Custom Operations 104

8.2.3 Custom Operations without associated resources 104

8.2.3.1 Overview 104

8.2.3.2 Operation: Fetch 105

8.2.3.2.1 Description 105

8.2.3.2.2 Operation Definition 105

8.2.4 Notifications 106

8.2.4.1 General 106

8.2.4.2 Location Information Notification 107

8.2.4.2.1 Description 107

8.2.4.2.2 Target URI 107

8.2.4.2.3 Standard Methods 107

8.2.4.3 User Consent Revocation Notification 108

8.2.4.3.1 Description 108

8.2.4.3.2 Target URI 108

8.2.4.3.3 Standard Methods 108

8.2.5 Data Model 109

8.2.5.1 General 109

8.2.5.2 Structured data types 111

8.2.5.2.1 Introduction 111

8.2.5.2.2 Type: LocationSubscription 111

8.2.5.2.3 Type: LocationSubscriptionPatch 113

8.2.5.2.4 Type: LocationNotification 113

8.2.5.2.5 Type: LocationEvent 113

8.2.5.2.6 Type: LocationRequest 114

8.2.5.2.7 Type: LocationResponse 114

8.2.5.2.8 Type: ConsentRevocNotif 114

8.2.5.2.9 Type: ConsentRevoked 115

8.2.5.3 Simple data types and enumerations 115

8.2.6 Error Handling 115

8.2.6.1 General 115

8.2.6.2 Protocol Errors 115

8.2.6.3 Application Errors 115

8.2.7 Feature negotiation 115

8.3 Eees_UEIdentifier API 116

8.3.1 Introduction 116

8.3.2 Resources 116

8.3.3 Custom Operations without associated resources 116

8.3.3.1 Overview 116

8.3.3.2 Operation: Fetch 117

8.3.3.2.1 Description 117

8.3.3.2.2 Operation Definition 117

8.3.3.3 Operation: Get 118

8.3.3.3.1 Description 118

8.3.3.3.2 Operation Definition 118

8.3.4 Notifications 119

8.3.5 Data Model 119

8.3.5.1 General 119

8.3.5.2 Structured data types 120

8.3.5.2.1 Introduction 120

8.3.5.2.2 Type: UserInformation 120

8.3.5.2.3 Type: UserInfo 121

8.3.5.2.4 Type: UeIdInfo 121

8.3.5.2.5 Type: UeId 122

8.3.5.3 Simple data types and enumerations 122

8.3.5.3.1 Introduction 122

8.3.5.3.2 Simple data types 122

8.3.6 Error Handling 122

8.3.6.1 General 122

8.3.6.2 Protocol Errors 122

8.3.6.3 Application Errors 122

8.3.7 Feature negotiation 123

8.4 Eees_AppClientInformation API 123

8.4.1 Introduction 123

8.4.2 Resources 123

8.4.2.1 Overview 123

8.4.2.2 Resource: Application Client Information Subscriptions 124

8.4.2.2.1 Description 124

8.4.2.2.2 Resource Definition 124

8.4.2.2.3 Resource Standard Methods 125

8.4.2.2.4 Resource Custom Operations 125

8.4.2.3 Resource: Individual Application Client Information Subscription 125

8.4.2.3.1 Description 125

8.4.2.3.2 Resource Definition 126

8.4.2.3.3 Resource Standard Methods 126

8.4.2.3.4 Resource Custom Operations 130

8.4.3 Custom Operations without associated resources 130

8.4.4 Notifications 131

8.4.4.1 General 131

8.4.4.2 AC Information Notification 131

8.4.4.2.1 Description 131

8.4.4.2.2 Target URI 131

8.4.4.2.3 Standard Methods 131

8.4.5 Data Model 132

8.4.5.1 General 132

8.4.5.2 Structured data types 135

8.4.5.2.1 Introduction 135

8.4.5.2.2 Type: ACInfoSubscription 135

8.4.5.2.3 Type: ACInfoSubscriptionPatch 136

8.4.5.2.4 Type: ACFilters 136

8.4.5.2.5 Type: ACInfoNotification 137

8.4.5.2.6 Type: ACInformation 137

8.4.5.2.7 Type: EASBdlInd 138

8.4.5.3 Simple data types and enumerations 138

8.4.5.3.1 Introduction 138

8.4.5.3.2 Simple data types 138

8.4.5.3.3 Enumeration: TrigCondParams 139

8.4.6 Error Handling 139

8.4.6.1 General 139

8.4.6.2 Protocol Errors 139

8.4.6.3 Application Errors 139

8.4.7 Feature negotiation 139

8.5 Eees_SessionWithQoS API 140

8.5.1 Introduction 140

8.5.2 Resources 140

8.5.2.1 Overview 140

8.5.2.2 Resource: Sessions with QoS 141

8.5.2.2.1 Description 141

8.5.2.2.2 Resource Definition 141

8.5.2.2.3 Resource Standard Methods 141

8.5.2.2.4 Resource Custom Operations 143

8.5.2.3 Resource: Individual Session with QoS 143

8.5.2.3.1 Description 143

8.5.2.3.2 Resource Definition 143

8.5.2.3.3 Resource Standard Methods 144

8.5.2.3.4 Resource Custom Operations 148

8.5.3 Custom Operations without associated resources 148

8.5.4 Notifications 148

8.5.4.1 General 148

8.5.4.2 User Plane Event Notification 148

8.5.4.2.1 Description 148

8.5.4.2.2 TargetURI 148

8.5.4.2.3 Standard Methods 148

8.5.5 Data Model 149

8.5.5.1 General 149

8.5.5.2 Structured data types 151

8.5.5.2.1 Introduction 151

8.5.5.2.2 Type: SessionWithQoS 151

8.5.5.2.3 Type: SessionWithQoSPatch 154

8.5.5.2.4 Type: UserPlaneEventNotification 154

8.5.5.3 Simple data types and enumerations 155

8.5.6 Error Handling 155

8.5.6.1 General 155

8.5.6.2 Protocol Errors 155

8.5.6.3 Application Errors 155

8.5.7 Feature negotiation 155

8.6 Eees_ACRManagementEvent API 155

8.6.1 Introduction 155

8.6.2 Resources 156

8.6.2.1 Overview 156

8.6.2.2 Resource: ACR Management Events Subscriptions 157

8.6.2.2.1 Description 157

8.6.2.2.2 Resource Definition 157

8.6.2.2.3 Resource Standard Methods 157

8.6.2.2.4 Resource Custom Operations 159

8.6.2.3 Resource: Individual ACR Management Events Subscription 159

8.6.2.3.1 Description 159

8.6.2.3.2 Resource Definition 159

8.6.2.3.3 Resource Standard Methods 159

8.6.2.3.4 Resource Custom Operations 163

8.6.3 Custom Operations without associated resources 163

8.6.4 Notifications 164

8.6.4.1 General 164

8.6.4.2 ACR Management Events Notification 164

8.6.4.2.1 Description 164

8.6.4.2.2 Notification definition 164

8.6.4.3 User Plane Path Change Availability Notification 165

8.6.4.3.1 Description 165

8.6.4.3.2 Target URI 165

8.6.4.3.3 Standard Methods 166

8.6.5 Data Model 166

8.6.5.1 General 166

8.6.5.2 Structured data types 169

8.6.5.2.1 Introduction 169

8.6.5.2.2 Type: AcrMgntEventsSubscription 169

8.6.5.2.3 Type: AcrMgntEventSubsc 172

8.6.5.2.4 Type: AcrMgntEventsSubscriptionPatch 174

8.6.5.2.5 Type: AcrMgntEventsNotification 175

8.6.5.2.6 Type: AcrMgntEventReport 176

8.6.5.2.7 Type: FailureAcrMgntEventInfo 178

8.6.5.2.8 Type: TargetUeIdentification 179

8.6.5.2.9 Type: UpPathChangeInfo 179

8.6.5.2.10 Type: IndUeIdentification 179

8.6.5.2.11 Type: AvailabilityNotif 180

8.6.5.2.12 Type: SelectedACRScenarios 180

8.6.5.2.13 Type: ACRParameters 180

8.6.5.2.14 Type: TrafficFilterInfo 180

8.6.5.2.15 Type: EasAckInformation 181

8.6.5.2.16 Type: EasInBundleInfo 181

8.6.5.3 Simple data types and enumerations 181

8.6.5.3.1 Introduction 181

8.6.5.3.2 Simple data types 181

8.6.5.3.3 Enumeration: AcrMgntEvent 181

8.6.5.3.4 Enumeration: AcrMgntEventFilter 182

8.6.5.3.5 Enumeration: ActStatus 182

8.6.5.3.6 Enumeration: AcrMgntEventFailureCode 182

8.6.5.3.7 Enumeration: AvailabilityStatus 182

8.6.5.3.8 Enumeration: ResultCode 183

8.6.5.3.9 Enumeration: InOutArea 183

8.6.6 Error Handling 183

8.6.6.1 General 183

8.6.6.2 Protocol Errors 183

8.6.6.3 Application Errors 183

8.6.7 Feature negotiation 183

8.7 Eees_EECContextRelocation API 184

8.7.1 API URI 184

8.7.1A Usage of HTTP 184

8.7.2 Resources 184

8.7.2.1 Overview 184

8.7.2.2 Resource: EEC Contexts 185

8.7.2.2.1 Description 185

8.7.2.2.2 Resource Definition 185

8.7.2.2.3 Resource Standard Methods 185

8.7.2.2.4 Resource Custom Operations 187

8.7.3 Custom Operations without associated resources 187

8.7.4 Notifications 187

8.7.5 Data Model 187

8.7.5.1 General 187

8.7.5.2 Structured data types 188

8.7.5.2.1 Introduction 188

8.7.5.2.2 Type: SessionContexts 188

8.7.5.2.3 Type: IndividualSessionContext 188

8.7.5.2.4 Type: EECContextPush 189

8.7.5.2.5 Type: EECContext 190

8.7.5.2.6 Type: EECContextPushRes 190

8.7.5.2.7 Type: ImplicitRegDetails 191

8.7.5.2.8 Type: EECSrvContinuitySupport 191

8.7.5.3 Simple data types and enumerations 191

8.7.5.3.1 Introduction 191

8.7.5.3.2 Simple data types 191

8.7.5.4 Data types describing alternative data types or combinations of data types 191

8.7.5.5 Binary data 192

8.7.5.5.1 Binary Data Types 192

8.7.6 Error Handling 192

8.7.6.1 General 192

8.7.6.2 Protocol Errors 192

8.7.6.3 Application Errors 192

8.7.7 Feature negotiation 192

8.8 Eees_EELManagedACR API 193

8.8.1 Introduction 193

8.8.2 Usage of HTTP 193

8.8.3 Resources 193

8.8.3.1 Overview 193

8.8.3.2 Resource: ACT Status Subscriptions 194

8.8.3.2.1 Description 194

8.8.3.2.2 Resource Definition 194

8.8.3.2.3 Resource Standard Methods 194

8.8.3.2.4 Resource Custom Operations 196

8.8.3.3 Resource: Individual ACT Status Subscription 196

8.8.3.3.1 Description 196

8.8.3.3.2 Resource Definition 196

8.8.3.3.3 Resource Standard Methods 196

8.8.3.3.4 Resource Custom Operations 197

8.8.4 Custom Operations without associated resources 198

8.8.4.1 Overview 198

8.8.4.2 Operation: RequestEELManagedACR 198

8.8.4.2.1 Description 198

8.8.4.2.2 Operation Definition 198

8.8.5 Notifications 199

8.8.5.1 General 199

8.8.5.2 ACT Status Notification 199

8.8.5.2.1 Description 199

8.8.5.2.2 Target URI 199

8.8.5.2.3 Standard Methods 200

8.8.6 Data Model 201

8.8.6.1 General 201

8.8.6.2 Structured data types 201

8.8.6.2.1 Introduction 201

8.8.6.2.2 Type: EELACRReq 201

8.8.6.2.3 Type: EELACRResp 202

8.8.6.2.4 Type: ACTStatusSubsc 202

8.8.6.2.5 Type: ACTStatusNotif 202

8.8.6.3 Simple data types and enumerations 202

8.8.6.3.1 Introduction 202

8.8.6.3.2 Simple data types 202

8.8.6.4 Data types describing alternative data types or combinations of data types 203

8.8.6.5 Binary data 203

8.8.6.5.1 Binary Data Types 203

8.8.7 Error Handling 203

8.8.7.1 General 203

8.8.7.2 Protocol Errors 203

8.8.7.3 Application Errors 203

8.8.8 Feature negotiation 203

8.9 Eees_ACRStatusUpdate API 203

8.9.1 Introduction 203

8.9.2 Usage of HTTP 204

8.9.3 Resources 204

8.9.4 Custom Operations without associated resources 204

8.9.4.1 Overview 204

8.9.4.2 Operation: RequestACRUpdate 205

8.9.4.2.1 Description 205

8.9.4.2.2 Operation Definition 205

8.9.5 Notifications 205

8.9.6 Data Model 206

8.9.6.1 General 206

8.9.6.2 Structured data types 206

8.9.6.2.1 Introduction 206

8.9.6.2.2 Type: ACRUpdateData 207

8.9.6.2.3 Type: ACRDataStatus 207

8.9.6.2.4 Type: ACTResultInfo 208

8.9.6.2.5 Type: AppGroupInfo 208

8.9.6.3 Simple data types and enumerations 208

8.9.6.3.1 Introduction 208

8.9.6.3.2 Simple data types 208

8.9.6.3.3 Enumeration: ACTResult 208

8.9.6.3.4 Enumeration: E3SubscsStatus 208

8.9.6.3.5 Enumeration: ACTFailureCause 209

8.9.6.4 Data types describing alternative data types or combinations of data types 209

8.9.6.5 Binary data 209

8.9.6.5.1 Binary Data Types 209

8.9.7 Error Handling 209

8.9.7.1 General 209

8.9.7.2 Protocol Errors 209

8.9.7.3 Application Errors 209

8.9.8 Feature negotiation 210

8.10 Eees_ACRParameterInformation API 210

8.10.1 Introduction 210

8.10.2 Usage of HTTP 210

8.10.3 Resources 210

8.10.4 Custom Operations without associated resources 211

8.10.4.1 Overview 211

8.10.4.2 Operation: Request 211

8.10.4.2.1 Description 211

8.10.4.2.2 Operation Definition 211

8.10.5 Notifications 212

8.10.6 Data Model 212

8.10.6.1 General 212

8.10.6.2 Structured data types 213

8.10.6.2.1 Introduction 213

8.10.6.2.2 Type: ACRParamsInfo 213

8.10.6.3 Simple data types and enumerations 213

8.10.6.3.1 Introduction 213

8.10.6.3.2 Simple data types 213

8.10.6.4 Data types describing alternative data types or combinations of data types 213

8.10.6.5 Binary data 214

8.10.6.5.1 Binary Data Types 214

8.10.7 Error Handling 214

8.10.7.1 General 214

8.10.7.2 Protocol Errors 214

8.10.7.3 Application Errors 214

8.10.8 Feature negotiation 214

8.11 Eees_CommonEASAnnouncement API 214

8.11.1 Introduction 214

8.11.2 Usage of HTTP 215

8.11.3 Resources 215

8.11.4 Custom Operations without associated resources 215

8.11.4.1 Overview 215

8.11.4.2 Operation: Declare 215

8.11.4.2.1 Description 215

8.11.4.2.2 Operation Definition 216

8.11.5 Notifications 216

8.11.6 Data Model 216

8.11.6.1 General 216

8.11.6.2 Structured data types 217

8.11.6.2.1 Introduction 217

8.11.6.2.2 Type: CommonEASInfo 217

8.11.6.2.3 Type: CommonEASInfoDecResp 218

8.11.6.2.4 Type: GrpConnInfo 218

8.11.6.3 Simple data types and enumerations 218

8.11.6.3.1 Introduction 218

8.11.6.3.2 Simple data types 218

8.11.6.4 Data types describing alternative data types or combinations of data types 218

8.11.6.5 Binary data 219

8.11.6.5.1 Binary Data Types 219

8.11.7 Error Handling 219

8.11.7.1 General 219

8.11.7.2 Protocol Errors 219

8.11.7.3 Application Errors 219

8.11.8 Feature negotiation 219

8.12 Eees_TrafficInfluenceEAS API 219

8.12.1 Introduction 219

8.12.2 Usage of HTTP 220

8.12.3 Resources 220

8.12.3.1 Overview 220

8.12.3.2 Resource: Application Traffic Influence 221

8.12.3.2.1 Description 221

8.12.3.2.2 Resource Definition 221

8.12.3.2.3 Resource Standard Methods 221

8.12.3.2.4 Resource Custom Operations 222

8.12.3.3 Resource: Individual Application Traffic Influence 222

8.12.3.3.1 Description 222

8.12.3.3.2 Resource Definition 222

8.12.3.3.3 Resource Standard Methods 222

8.12.3.3.4 Resource Custom Operations 226

8.12.4 Notifications 227

8.12.5 Data Model 227

8.12.5.1 General 227

8.12.5.2 Structured data types 227

8.12.5.2.1 Introduction 227

8.12.5.2.2 Type: AppTrafficInfluence 228

8.12.5.2.3 Type: AppTrafficInfluencePatch 229

8.12.5.3 Simple data types and enumerations 229

8.12.5.3.1 Introduction 229

8.12.5.3.2 Simple data types 229

8.12.5.4 Data types describing alternative data types or combinations of data types 229

8.12.5.5 Binary data 229

8.12.5.5.1 Binary Data Types 229

8.12.6 Error Handling 230

8.12.6.1 General 230

8.12.6.2 Protocol Errors 230

8.12.6.3 Application Errors 230

8.12.7 Feature negotiation 230

8A CAS API Definitions 230

8A.1 Ecas_SelectedEES API 230

8A.1.1 Introduction 230

8A.1.2 Usage of HTTP 231

8A.1.3 Resources 231

8A.1.4 Custom Operations without associated resources 231

8A.1.4.1 Overview 231

8A.1.4.2 Operation: Declare 231

8A.1.4.2.1 Description 231

8A.1.4.2.2 Operation Definition 231

8A.1.5 Notifications 232

8A.1.6 Data Model 232

8A.1.6.1 General 232

8A.1.6.2 Structured data types 233

8A.1.6.2.1 Introduction 233

8A.1.6.2.2 Type: SelEESDecInfo 233

8A.1.6.3 Simple data types and enumerations 233

8A.1.6.3.1 Introduction 233

8A.1.6.3.2 Simple data types 233

8A.1.6.4 Data types describing alternative data types or combinations of data types 233

8A.1.6.5 Binary data 234

8A.1.6.5.1 Binary Data Types 234

8A.1.7 Error Handling 234

8A.1.7.1 General 234

8A.1.7.2 Protocol Errors 234

8A.1.7.3 Application Errors 234

8A.1.8 Feature negotiation 234

9 Edge Configuration Server API Definitions 234

9.1 Eecs_EESRegistration API 234

9.1.1 Introduction 234

9.1.2 Resources 235

9.1.2.1 Overview 235

9.1.2.2 Resource: EES Registrations 236

9.1.2.2.1 Description 236

9.1.2.2.2 Resource Definition 236

9.1.2.2.3 Resource Standard Methods 236

9.1.2.2.4 Resource Custom Operations 237

9.1.2.3 Resource: Individual EES Registration 237

9.1.2.3.1 Description 237

9.1.2.3.2 Resource Definition 237

9.1.2.3.3 Resource Standard Methods 237

9.1.2.3.4 Resource Custom Operations 241

9.1.3 Custom Operations without associated resources 241

9.1.4 Notifications 241

9.1.5 Data Model 241

9.1.5.1 General 241

9.1.5.2 Structured data types 243

9.1.5.2.1 Introduction 243

9.1.5.2.2 Type: EESRegistration 243

9.1.5.2.3 Type: EESProfile 244

9.1.5.2.4 Type: EESRegistrationPatch 246

9.1.5.2.5 Type: ServiceArea 247

9.1.5.2.6 Type: TopologicalServiceArea 247

9.1.5.2.7 Type: GeographicalServiceArea 247

9.1.5.2.8 Type: EASInstantiationInfo 247

9.1.5.2.9 Type: InstantiationCriteria 248

9.1.5.2.10 Type: EDNInfo 248

9.1.5.2.11 Type: LoadInfo 248

9.1.5.2.12 Type: UeServSatInfo 249

9.1.5.2.13 Type: AppSatCovAvailInfo 249

9.1.5.3 Simple data types and enumerations 249

9.1.5.3.1 Introduction 249

9.1.5.3.2 Simple data types 249

9.1.5.3.3 Enumeration: ACRScenario 249

9.1.5.3.4 Enumeration: InstantiationStatus 250

9.1.6 Error Handling 250

9.1.6.1 General 250

9.1.6.2 Protocol Errors 250

9.1.6.3 Application Errors 250

9.1.7 Feature negotiation 250

9.2 Eecs_TargetEESDiscovery API 251

9.2.1 Introduction 251

9.2.2 Resources 251

9.2.2.1 Overview 251

9.2.2.2 Resource: EES Profiles 252

9.2.2.2.1 Description 252

9.2.2.2.2 Resource Definition 252

9.2.2.2.3 Resource Standard Methods 252

9.2.2.2.4 Resource Custom Operations 254

9.2.3 Custom Operations without associated resources 254

9.2.4 Notifications 254

9.2.5 Data Model 254

9.2.5.1 General 254

9.2.5.2 Structured data types 255

9.2.5.2.1 Introduction 255

9.2.5.2.2 Type: TunnelInfo 255

9.2.5.2.3 Type: CandidateDnaisInfo 256

9.2.5.3 Simple data types and enumerations 256

9.2.6 Error Handling 256

9.2.7 Feature negotiation 256

9.3 Eecs_EASInfoManagement API 257

9.3.1 Introduction 257

9.3.2 Usage of HTTP 257

9.3.3 Resources 258

9.3.3.1 Overview 258

9.3.3.2 Resource: Common EAS Bindings 258

9.3.3.2.1 Description 258

9.3.3.2.2 Resource Definition 258

9.3.3.2.3 Resource Standard Methods 259

9.3.3.2.4 Resource Custom Operations 261

9.3.3.3 Resource: Individual Common EAS Binding 261

9.3.3.3.1 Description 261

9.3.3.3.2 Resource Definition 261

9.3.3.3.3 Resource Standard Methods 261

9.3.4 Custom Operations without associated resources 265

9.3.5 Notifications 265

9.3.6 Data Model 266

9.3.6.1 General 266

9.3.6.2 Structured data types 266

9.3.6.2.1 Introduction 266

9.3.6.2.2 Type: CommonEASBindReq 267

9.3.6.2.3 Type: CommonEASBindResp 267

9.3.6.2.4 Type: CommonEASBinding 268

9.3.6.2.5 Type: CommEasBdlInfo 269

9.3.6.2.6 Type: CommonEASBindingPatch 269

9.3.6.3 Simple data types and enumerations 270

9.3.6.3.1 Introduction 270

9.3.6.3.2 Simple data types 270

9.3.6.4 Data types describing alternative data types or combinations of data types 270

9.3.6.4.1 Type: ProblemDetailsEIMExt 270

9.3.6.5 Binary data 270

9.3.6.5.1 Binary Data Types 270

9.3.7 Error Handling 270

9.3.7.1 General 270

9.3.7.2 Protocol Errors 270

9.3.7.3 Application Errors 270

9.3.8 Feature negotiation 271

9.4 Eecs_ECSServiceProvisioning API 271

9.4.1 Introduction 271

9.4.2 Usage of HTTP 271

9.4.3 Resources 272

9.4.3.1 Overview 272

9.4.3.2 Resource: Service Provisioning Subscriptions 272

9.4.3.2.1 Description 272

9.4.3.2.2 Resource Definition 272

9.4.3.2.3 Resource Standard Methods 273

9.4.3.2.4 Resource Custom Operations 273

9.4.3.3 Resource: Individual Service Provisioning Subscription 273

9.4.3.3.1 Description 273

9.4.3.3.2 Resource Definition 274

9.4.3.3.3 Resource Standard Methods 274

9.4.4 Custom Operations without associated resources 278

9.4.4.1 Overview 278

9.4.4.2 Operation: Request 279

9.4.4.2.1 Description 279

9.4.4.2.2 Operation Definition 279

9.4.5 Notifications 280

9.4.5.1 General 280

9.4.5.2 Service Provisioning Notification 281

9.4.5.2.1 Description 281

9.4.5.2.2 Target URI 281

9.4.5.2.3 Standard Methods 281

9.4.6 Data Model 282

9.4.6.1 General 282

9.4.6.2 Structured data types 283

9.4.6.2.1 Introduction 283

9.4.6.2.2 Type: ServProvReq 283

9.4.6.2.3 Type: ServProvResp 283

9.4.6.2.4 Type: ServProvSubsc 284

9.4.6.2.5 Type: ServProvSubscPatch 284

9.4.6.2.6 Type: ServProvNotif 285

9.4.6.2.7 Type: FederationInfo 285

9.4.6.2.8 Type: AppInfo 285

9.4.6.2.9 Type: AppGrpProfile 285

9.4.6.3 Simple data types and enumerations 285

9.4.6.3.1 Introduction 285

9.4.6.3.2 Simple data types 285

9.4.6.4 Data types describing alternative data types or combinations of data types 286

9.4.6.5 Binary data 286

9.4.6.5.1 Binary Data Types 286

9.4.7 Error Handling 286

9.4.7.1 General 286

9.4.7.2 Protocol Errors 286

9.4.7.3 Application Errors 286

9.4.8 Feature negotiation 286

9.4.9 Security 287

9.5 Eecs_ECSDiscovery API 287

9.5.1 Introduction 287

9.5.2 Resources 287

9.5.2.1 Overview 287

9.5.2.2 Resource: ECS Information 288

9.5.2.2.1 Description 288

9.5.2.2.2 Resource Definition 288

9.5.2.2.3 Resource Standard Methods 288

9.5.2.2.4 Resource Custom Operations 288

9.5.3 Custom Operations without associated resources 290

9.5.4 Notifications 290

9.5.4.1 General 290

9.5.4.2 ECS Discovery Notification 290

9.5.4.2.1 Description 290

9.5.4.2.2 Target URI 290

9.5.4.2.3 Standard Methods 290

9.5.5 Data Model 291

9.5.5.1 General 291

9.5.5.2 Structured data types 292

9.5.5.2.1 Introduction 292

9.5.5.2.2 Type: EcsInfoDiscoveryReq 293

9.5.5.2.3 Type: EcsInfoDiscoveryResp 293

9.5.5.2.4 Type: EcsInfo 293

9.5.5.2.5 Type: ECSProfile 294

9.5.5.2.6 Type: SupportedPlmn 294

9.5.5.2.7 Type: SupportedEcsp 294

9.5.5.2.8 Type: PduConfiguration 294

9.5.5.2.9 Type: EcsInfoDiscNotif 295

9.5.5.3 Simple data types and enumerations 295

9.5.5.3.1 Introduction 295

9.5.5.3.2 Simple data types 295

9.5.5.4 Data types describing alternative data types or combinations of data types 295

9.5.5.5 Binary data 295

9.5.5.5.1 Binary Data Types 295

9.5.6 Error Handling 295

9.5.6.1 General 295

9.5.6.2 Protocol Errors 295

9.5.6.3 Application Errors 295

9.5.7 Feature negotiation 296

9.6 Eecs_ACREvents API 296

9.6.1 Introduction 296

9.6.2 Usage of HTTP 296

9.6.3 Resources 296

9.6.3.1 Overview 296

9.6.3.2 Resource: ACR Events Subscriptions 297

9.6.3.2.1 Description 297

9.6.3.2.2 Resource Definition 297

9.6.3.2.3 Resource Standard Methods 298

9.6.3.2.4 Resource Custom Operations 298

9.6.3.3 Resource: Individual ACR Events Subscription 298

9.6.3.3.1 Description 298

9.6.3.3.2 Resource Definition 298

9.6.3.3.3 Resource Standard Methods 299

9.6.3.3.4 Resource Custom Operations 303

9.6.4 Custom Operations without associated resources 303

9.6.5 Notifications 303

9.6.5.1 General 303

9.6.5.2 ACR Events Notification 304

9.6.5.2.1 Description 304

9.6.5.2.2 Target URI 304

9.6.5.2.3 Standard Methods 304

9.6.6 Data Model 305

9.6.6.1 General 305

9.6.6.2 Structured data types 306

9.6.6.2.1 Introduction 306

9.6.6.2.2 Type: AcrEventsSubsc 306

9.6.6.2.3 Type: AcrEventsSubscPatch 307

9.6.6.2.4 Type: AcrEventsNotif 307

9.6.6.3 Simple data types and enumerations 307

9.6.6.3.1 Introduction 307

9.6.6.3.2 Simple data types 307

9.6.6.3.3 Enumeration: AcrEvent 307

9.6.6.4 Data types describing alternative data types or combinations of data types 308

9.6.6.5 Binary data 308

9.6.6.5.1 Binary Data Types 308

9.6.7 Error Handling 308

9.6.7.1 General 308

9.6.7.2 Protocol Errors 308

9.6.7.3 Application Errors 308

9.6.8 Feature negotiation 308

9.6.9 Security 309

10 Using Common API Framework 309

10.1 General 309

10.2 Security 309

11 Security 310

Annex A (normative): OpenAPI specification 310

A.1 General 310

A.2 Eees_EASRegistration API 310

A.3 Eees_UELocation API 319

A.4 Eees_UEIdentifier API 327

A.5 Eees_AppClientInformation API 330

A.6 Eees_SessionWithQoS API 336

A.7 Eees_ACRManagementEvent API 343

A.8 Eees_EECContextRelocation API 354

A.9 Eees_EELManagedACR API 358

A.10 Eees_ACRStatusUpdate API 362

A.11 Eecs_EESRegistration API 365

A.12 Eecs_TargetEESDiscovery API 373

A.13 Eees_ACRParameterInformation API 376

A.14 Ecas_SelectedEES API 378

A.15 Eees_CommonEASAnnouncement API 379

A.16 Eecs_EASInfoManagement API 381

A.17 Eees_TrafficInfluenceEAS API 387

A.18 Eecs_ECSServiceProvisioning API 392

A.19 Eecs_ECSDiscovery API 398

A.20 Eecs_ACREvents API 402

Annex B (informative): Change history 408
