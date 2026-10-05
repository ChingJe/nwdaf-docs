---
spec: TS 29.222
version: 20.1.0
release: '20'
clause: contents
title: Contents
source_archive: 29222-k10.zip
source_document: 29222-k10.docx
content_origin: 3gpp-source
---

# Contents

Foreword 14

1 Scope 15

2 References 15

3 Definitions and abbreviations 16

3.1 Definitions 16

3.2 Abbreviations 16

4 Overview 17

4.1 Introduction 17

4.2 Service Architecture 17

4.3 Functional Entities 17

4.3.1 API invoker 17

4.3.2 CAPIF core function 17

4.3.3 API exposing function 17

4.3.4 API publishing function 17

4.3.5 API management function 17

5 Services offered by the CAPIF Core Function 17

5.1 Introduction of Services 17

5.2 CAPIF_Discover_Service_API 19

5.2.1 Service Description 19

5.2.1.1 Overview 19

5.2.2 Service Operations 19

5.2.2.1 Introduction 19

5.2.2.2 Discover_Service_API 20

5.2.2.2.1 General 20

5.2.2.2.2 Consumer discovering service API using Discover_Service_API service operation 20

5.3 CAPIF_Publish_Service_API 20

5.3.1 Service Description 20

5.3.1.1 Overview 20

5.3.2 Service Operations 21

5.3.2.1 Introduction 21

5.3.2.2 Publish_Service_API 21

5.3.2.2.1 General 21

5.3.2.2.2 API publishing function publishing service APIs on CAPIF core function using Publish_Service_API service operation 21

5.3.2.2.3 CAPIF core function publishing service APIs on other CAPIF core function using Publish_Service_API service operation 22

5.3.2.3 Unpublish_Service_API 23

5.3.2.3.1 General 23

5.3.2.3.2 Consumer un-publishing service APIs from CAPIF core function using Unpublish_Service_API service operation 23

5.3.2.4 Get_Service_API 23

5.3.2.4.1 General 23

5.3.2.4.2 Consumer retrieving service APIs from CAPIF core function using Get_Service_API service operation 23

5.3.2.5 Update_Service_API 24

5.3.2.5.1 General 24

5.3.2.5.2 Consumer updating published service APIs on CAPIF core function using Update_Service_API service operation 24

5.4 CAPIF_Events_API 25

5.4.1 Service Description 25

5.4.1.1 Overview 25

5.4.2 Service Operations 25

5.4.2.1 Introduction 25

5.4.2.2 Subscribe_Event 25

5.4.2.2.1 General 25

5.4.2.2.2 Subscribing to CAPIF events using Subscribe_Event service operation 25

5.4.2.3 Unsubscribe_Event 27

5.4.2.3.1 General 27

5.4.2.3.2 Unsubscribing from CAPIF events using Unsubscribe_Event service operation 27

5.4.2.4 Notify_Event 27

5.4.2.4.1 General 27

5.4.2.4.2 Notifying CAPIF events using Notify_Event service operation 27

5.4.2.5 Update_Event_Subscription 28

5.4.2.5.1 General 28

5.4.2.5.2 Update Subscription to CAPIF events using Update_Event_Subscription service operation 28

5.5 CAPIF_API_Invoker_Management_API 28

5.5.1 Service Description 28

5.5.1.1 Overview 28

5.5.2 Service Operations 28

5.5.2.1 Introduction 28

5.5.2.2 Onboard_API_Invoker 29

5.5.2.2.1 General 29

5.5.2.2.2 API Invoker on-boarding itself as a recognized user of CAPIF using the Onboard_API_Invoker service operation 29

5.5.2.3 Offboard_API_Invoker 30

5.5.2.3.1 General 30

5.5.2.3.2 API Invoker off-boarding itself from being a recognized user of CAPIF using the Offboard_API_Invoker service operation 30

5.5.2.4 Notify_Onboarding_Completion 30

5.5.2.4.1 General 30

5.5.2.4.2 Notifying API Invoker's onboarding creation/update completion using Notify_Onboarding_Completion service operation 31

5.5.2.5 Update_API_Invoker_Details 31

5.5.2.5.1 General 31

5.5.2.5.2 API Invoker updating its details on CAPIF using Update_API_Invoker_Details service operation 31

5.5.2.6 Notify_Update_Completion 32

5.5.2.6.1 General 32

5.6 CAPIF_Security_API 32

5.6.1 Service Description 32

5.6.1.1 Overview 32

5.6.2 Service Operations 32

5.6.2.1 Introduction 32

5.6.2.2 Obtain_Security_Method 33

5.6.2.2.1 General 33

5.6.2.2.2 Request service API security method from CAPIF using Obtain_Security_Method service operation 33

5.6.2.3 Obtain_Authorization 33

5.6.2.3.1 General 33

5.6.2.3.2 Obtain authorization using Obtain_Authorization service operation 34

5.6.2.3.3 Void 34

5.6.2.4 Obtain_API_Invoker_Info 34

5.6.2.4.1 General 34

5.6.2.4.2 Obtain API invoker's security information using Obtain_API_Invoker_Info service operation 35

5.6.2.5 Revoke_Authorization 35

5.6.2.5.1 General 35

5.6.2.5.2 Invalidate authorization using Revoke_Authorization service operation 35

5.7 CAPIF_Monitoring_API 35

5.8 CAPIF_Logging_API_Invocation_API 36

5.8.1 Service Description 36

5.8.1.1 Overview 36

5.8.2 Service Operations 36

5.8.2.1 Introduction 36

5.8.2.2 Log_API_Invocation 36

5.8.2.2.1 General 36

5.8.2.2.2 Logging service API invocations using Log_API_Invocation service operation 36

5.9 CAPIF_Auditing_API 37

5.9.1 Service Description 37

5.9.1.1 Overview 37

5.9.2 Service Operations 37

5.9.2.1 Introduction 37

5.9.2.2 Query_API_Invocation_Log 37

5.9.2.2.1 General 37

5.9.2.2.2 Query API invocation information logs using Query_API_Invocation_Log service operation 37

5.10 CAPIF_Access_Control_Policy_API 38

5.10.1 Service Description 38

5.10.1.1 Overview 38

5.10.2 Service Operations 38

5.10.2.1 Introduction 38

5.10.2.2 Obtain_Access_Control_Policy 38

5.10.2.2.1 General 38

5.10.2.2.2 API exposing function obtaining access control policy from the CAPIF core function using Obtain_Access_Control_Policy service operation 38

5.10.3 Related Events 38

5.11 CAPIF_API_Provider_Management_API 39

5.11.1 Service Description 39

5.11.1.1 Overview 39

5.11.2 Service Operations 39

5.11.2.1 Introduction 39

5.11.2.2 Register_API_Provider 39

5.11.2.2.1 General 39

5.11.2.2.2 API provider domain functions registering as a recognized API provider domain function of CAPIF using Register_API_Provider service operation 39

5.11.2.3 Update_API_Provider 40

5.11.2.3.1 General 40

5.11.2.3.2 API management function updating API provider domain function details on CAPIF using Update_API_Provider service operation 40

5.11.2.4 Deregister_API_Provider 40

5.11.2.4.1 General 40

5.11.2.4.2 API provider domain functions deregistering as a recognized API provider domain function of CAPIF using Deregister_API_Provider service operation 41

5.11.2.5 Disenroll_Service_APIs 41

5.11.2.5.1 General 41

5.11.2.5.2 AMF requesting removal of enrolled service APIs using Disenroll_Service_APIs service operation 41

5.12 CAPIF_Routing_Info_API 42

5.12.1 Service Description 42

5.12.1.1 Overview 42

5.12.2 Service Operations 42

5.12.2.1 Introduction 42

5.12.2.2 Obtain_Routing_Info 42

5.12.2.2.1 General 42

5.12.2.2.2 API exposing function obtaining API routing information from the CAPIF core function using Obtain_Routing_Info service operation 42

5.13 CAPIF_Open_Discover_Service_API 42

5.13.1 Service Description 42

5.13.2 Service Operations 43

5.13.2.1 Introduction 43

5.13.2.2 Open_Discover_Service_API 43

5.13.2.2.1 General 43

5.13.2.2.2 Open Service API(s) Discovery 43

6 Services offered by the API exposing function 43

6.1 Introduction of Services 43

6.2 AEF_Security_API 44

6.2.1 Service Description 44

6.2.1.1 Overview 44

6.2.2 Service Operations 44

6.2.2.1 Introduction 44

6.2.2.2 Initiate_Authentication 44

6.2.2.2.1 General 44

6.2.2.2.2 API invoker initiating authentication using Initiate_Authentication service operation 45

6.2.2.3 Revoke_Authorization 45

6.2.2.3.1 General 45

6.2.2.3.2 CAPIF core function initiating revocation using Revoke_Authorization service operation 45

7 CAPIF Design Aspects Common for All APIs 45

7.1 General 45

7.2 Data Types 46

7.2.1 General 46

7.2.2 Void 46

7.2.3 Void 46

7.3 Usage of HTTP 46

7.4 Content type 46

7.5 URI structure 46

7.5.1 Resource URI structure 46

7.5.2 Custom operations URI structure 46

7.6 Notifications 46

7.7 Error handling 47

7.8 Feature negotiation 47

7.9 HTTP custom headers 47

7.10 Conventions for Open API specification files 47

7.11 CAPIF vendor-specific extensions 47

8 CAPIF Core Function API Definition 47

8.1 CAPIF_Discover_Service_API 47

8.1.1 API URI 47

8.1.2 Resources 48

8.1.2.1 Overview 48

8.1.2.2 Resource: All published service APIs 48

8.1.2.2.1 Description 48

8.1.2.2.2 Resource Definition 48

8.1.2.2.3 Resource Standard Methods 49

8.1.2.2.4 Resource Custom Operations 53

8.1.2A Custom Operations without associated resources 53

8.1.3 Notifications 53

8.1.4 Data Model 53

8.1.4.1 General 53

8.1.4.2 Structured data types 54

8.1.4.2.1 Introduction 54

8.1.4.2.2 Type: DiscoveredAPIs 54

8.1.4.2.3 Void 55

8.1.4.2.4 Type: IpAddrInfo 55

8.1.4.2.5 Type: ResOperInfo 55

8.1.4.3 Simple data types and enumerations 55

8.1.4.3.1 Introduction 55

8.1.4.3.2 Simple data types 55

8.1.4.4 Data types describing alternative data types or combinations of data types 56

8.1.5 Error Handling 56

8.1.5.1 General 56

8.1.5.2 Protocol Errors 56

8.1.5.3 Application Errors 56

8.1.6 Feature negotiation 56

8.2 CAPIF_Publish_Service_API 57

8.2.1 API URI 57

8.2.2 Resources 57

8.2.2.1 Overview 57

8.2.2.2 Resource: APF published APIs 58

8.2.2.2.1 Description 58

8.2.2.2.2 Resource Definition 58

8.2.2.2.3 Resource Standard Methods 59

8.2.2.2.4 Resource Custom Operations 60

8.2.2.3 Resource: Individual APF published API 61

8.2.2.3.1 Description 61

8.2.2.3.2 Resource Definition 61

8.2.2.3.3 Resource Standard Methods 61

8.2.2.3.4 Resource Custom Operations 65

8.2.2A Custom Operations without associated resources 65

8.2.3 Notifications 65

8.2.4 Data Model 66

8.2.4.1 General 66

8.2.4.2 Structured data types 67

8.2.4.2.1 Introduction 67

8.2.4.2.2 Type: ServiceAPIDescription 68

8.2.4.2.3 Type: InterfaceDescription 70

8.2.4.2.4 Type: AefProfile 71

8.2.4.2.5 Type: Version 72

8.2.4.2.6 Type: Resource 72

8.2.4.2.7 Type: CustomOperation 73

8.2.4.2.8 Type: ShareableInformation 73

8.2.4.2.9 Type: PublishedApiPath 73

8.2.4.2.10 Type: AefLocation 74

8.2.4.2.11 Type: ServiceAPIDescriptionPatch 75

8.2.4.2.12 Type: ApiStatus 75

8.2.4.2.13 Type: ServiceKpis 76

8.2.4.2.14 Type: IpAddrRange 78

8.2.4.2.15 Type: UsageInfo 78

8.2.4.3 Simple data types and enumerations 78

8.2.4.3.1 Introduction 78

8.2.4.3.2 Simple data types 78

8.2.4.3.3 Enumeration: Protocol 79

8.2.4.3.4 Enumeration: DataFormat 79

8.2.4.3.5 Enumeration: CommunicationType 79

8.2.4.3.6 Enumeration: SecurityMethod 79

8.2.4.3.7 Enumeration: Operation 80

8.2.4.3.8 Enumeration: UsageValue 80

8.2.5 Error Handling 80

8.2.5.1 General 80

8.2.5.2 Protocol Errors 80

8.2.5.3 Application Errors 80

8.2.6 Feature negotiation 80

8.3 CAPIF_Events_API 81

8.3.1 API URI 81

8.3.2 Resources 82

8.3.2.1 Overview 82

8.3.2.2 Resource: CAPIF Events Subscriptions 83

8.3.2.2.1 Description 83

8.3.2.2.2 Resource Definition 83

8.3.2.2.3 Resource Standard Methods 83

8.3.2.2.4 Resource Custom Operations 84

8.3.2.3 Resource: Individual CAPIF Events Subscription 84

8.3.2.3.1 Description 84

8.3.2.3.2 Resource Definition 84

8.3.2.3.3 Resource Standard Methods 84

8.3.2.3.4 Resource Custom Operations 87

8.3.2A Custom Operations without associated resources 87

8.3.3 Notifications 87

8.3.3.1 General 87

8.3.3.2 Event Notification 88

8.3.3.2.1 Description 88

8.3.3.2.2 Notification definition 88

8.3.4 Data Model 89

8.3.4.1 General 89

8.3.4.2 Structured data types 90

8.3.4.2.1 Introduction 90

8.3.4.2.2 Type: EventSubscription 91

8.3.4.2.3 Type: EventNotification 92

8.3.4.2.4 Type: CAPIFEventFilter 92

8.3.4.2.5 Type: CAPIFEventDetail 93

8.3.4.2.6 Type: AccessControlPolicyListExt 93

8.3.4.2.7 Type: TopologyHiding 93

8.3.4.2.8 Type: EventSubscriptionPatch 93

8.3.4.2.9 Type: ApiInvokerCount 94

8.3.4.2.10 Type: DiscoveryCount 94

8.3.4.2.11 Type: CAPIFRepInfoExt 94

8.3.4.3 Simple data types and enumerations 94

8.3.4.3.1 Introduction 94

8.3.4.3.2 Simple data types 94

8.3.4.3.3 Enumeration: CAPIFEvent 95

8.3.4.4 Data types describing alternative data types or combinations of data types 96

8.3.4.5 Binary data 96

8.3.4.5.1 Binary Data Types 96

8.3.5 Error Handling 96

8.3.5.1 General 96

8.3.5.2 Protocol Errors 96

8.3.5.3 Application Errors 96

8.3.6 Feature negotiation 96

8.4 CAPIF_API_Invoker_Management_API 97

8.4.1 API URI 97

8.4.2 Resources 97

8.4.2.1 Overview 97

8.4.2.2 Resource: On-boarded API Invokers 98

8.4.2.2.1 Description 98

8.4.2.2.2 Resource Definition 98

8.4.2.2.3 Resource Standard Methods 99

8.4.2.2.4 Resource Custom Operations 99

8.4.2.3 Resource: Individual On-boarded API Invoker 99

8.4.2.3.1 Description 99

8.4.2.3.2 Resource Definition 99

8.4.2.3.3 Resource Standard Methods 100

8.4.2.3.4 Resource Custom Operations 103

8.4.2A Custom Operations without associated resources 103

8.4.3 Notifications 103

8.4.3.1 General 103

8.4.3.2 Notify_Onboarding_Completion 104

8.4.3.2.1 Description 104

8.4.3.2.2 Notification definition 104

8.4.3.3 Void 105

8.4.4 Data Model 105

8.4.4.1 General 105

8.4.4.2 Structured data types 106

8.4.4.2.1 Introduction 106

8.4.4.2.2 Type: APIInvokerEnrolmentDetails 107

8.4.4.2.3 Type: Void 108

8.4.4.2.4 Type: APIList 108

8.4.4.2.5 Type: OnboardingInformation 108

8.4.4.2.6 Type: Void 109

8.4.4.2.7 Type: OnboardingNotification 109

8.4.4.2.8 Type: APIInvokerEnrolmentDetailsPatch 109

8.4.4.2.9 Type: OnboardingCriteria 110

8.4.4.2.10 Type: RelatedCriteria 110

8.4.4.2.11 Type: ApiInfo 110

8.4.4.2.12 Type: EnrolFailReason 110

8.4.4.3 Simple data types and enumerations 111

8.4.4.3.1 Introduction 111

8.4.4.3.2 Simple data types 111

8.4.4.3.3 Enumeration: EnrolFailCause 111

8.4.4.3.4 Enumeration: OnboardingFailReason 111

8.4.4.4 Data types describing alternative data types or combinations of data types 111

8.4.4.5 Binary data 112

8.4.4.5.1 Binary Data Types 112

8.4.5 Error Handling 112

8.4.5.1 General 112

8.4.5.2 Protocol Errors 112

8.4.5.3 Application Errors 112

8.4.6 Feature negotiation 112

8.5 CAPIF_Security_API 113

8.5.1 API URI 113

8.5.2 Resources 113

8.5.2.1 Overview 113

8.5.2.2 Resource: Trusted API invokers 114

8.5.2.2.1 Description 114

8.5.2.2.2 Resource Definition 115

8.5.2.2.3 Resource Standard Methods 115

8.5.2.2.4 Resource Custom Operations 115

8.5.2.3 Resource: Individual trusted API invokers 115

8.5.2.3.1 Description 115

8.5.2.3.2 Resource Definition 115

8.5.2.3.3 Resource Standard Methods 115

8.5.2.3.4 Resource Custom Operations 118

8.5.2A Custom Operations without associated resources 122

8.5.3 Notifications 122

8.5.3.1 General 122

8.5.3.2 Authorization revoked notification 123

8.5.3.2.1 Description 123

8.5.3.2.2 Notification definition 123

8.5.4 Data Model 124

8.5.4.1 General 124

8.5.4.2 Structured data types 126

8.5.4.2.1 Introduction 126

8.5.4.2.2 Type: ServiceSecurity 126

8.5.4.2.3 Type: SecurityInformation 127

8.5.4.2.4 Void 127

8.5.4.2.5 Type: SecurityNotification 127

8.5.4.2.6 Type: AccessTokenReq 128

8.5.4.2.7 Type: AccessTokenRsp 132

8.5.4.2.8 Type: AccessTokenClaims 136

8.5.4.2.9 Type: AccessTokenErr 139

8.5.4.2.10 Void 140

8.5.4.2.11 Type: ResOwnerId 140

8.5.4.3 Simple data types and enumerations 140

8.5.4.3.1 Introduction 140

8.5.4.3.2 Simple data types 140

8.5.4.3.3 Enumeration: Cause 140

8.5.4.3.4 Enumeration: OAuthGrantType 141

8.5.5 Error Handling 141

8.5.5.1 General 141

8.5.5.2 Protocol Errors 141

8.5.5.3 Application Errors 141

8.5.6 Feature negotiation 141

8.6 CAPIF_Access_Control_Policy_API 142

8.6.1 API URI 142

8.6.2 Resources 142

8.6.2.1 Overview 142

8.6.2.2 Resource: Access Control Policy List 143

8.6.2.2.1 Description 143

8.6.2.2.2 Resource Definition 143

8.6.2.2.3 Resource Standard Methods 144

8.6.2.2.4 Resource Custom Operations 145

8.6.2A Custom Operations without associated resources 145

8.6.3 Notifications 145

8.6.4 Data Model 145

8.6.4.1 General 145

8.6.4.2 Structured data types 145

8.6.4.2.1 Introduction 145

8.6.4.2.2 Type: AccessControlPolicyList 146

8.6.4.2.3 Type: ApiInvokerPolicy 146

8.6.4.2.4 Type: TimeRangeList 146

8.6.4.3 Simple data types and enumerations 146

8.6.5 Error Handling 146

8.6.5.1 General 146

8.6.5.2 Protocol Errors 146

8.6.5.3 Application Errors 146

8.6.6 Feature negotiation 147

8.7 CAPIF_Logging_API_Invocation_API 147

8.7.1 API URI 147

8.7.2 Resources 147

8.7.2.1 Overview 147

8.7.2.2 Resource: Logs 148

8.7.2.2.1 Description 148

8.7.2.2.2 Resource Definition 148

8.7.2.2.3 Resource Standard Methods 148

8.7.2.2.4 Resource Custom Operations 149

8.7.2A Custom Operations without associated resources 149

8.7.3 Notifications 149

8.7.4 Data Model 149

8.7.4.1 General 149

8.7.4.2 Structured data types 150

8.7.4.2.1 Introduction 150

8.7.4.2.2 Type: InvocationLog 150

8.7.4.2.3 Type: Log 151

8.7.4.3 Simple data types and enumerations 151

8.7.4.3.1 Introduction 151

8.7.4.3.2 Simple data types 152

8.7.5 Error Handling 152

8.7.5.1 General 152

8.7.5.2 Protocol Errors 152

8.7.5.3 Application Errors 152

8.7.6 Feature negotiation 152

8.8 CAPIF_Auditing_API 152

8.8.1 API URI 152

8.8.2 Resources 153

8.8.2.1 Overview 153

8.8.2.2 Resource: All service API invocation logs 153

8.8.2.2.1 Description 153

8.8.2.2.2 Resource Definition 153

8.8.2.2.3 Resource Standard Methods 154

8.8.2.2.4 Resource Custom Operations 155

8.8.2A Custom Operations without associated resources 155

8.8.3 Notifications 155

8.8.4 Data Model 155

8.8.4.1 General 155

8.8.4.2 Structured data types 156

8.8.4.2.1 Introduction 156

8.8.4.2.2 Type: InvocationLogs 156

8.8.4.3 Simple data types and enumerations 157

8.8.4.4 Data types describing alternative data types or combinations of data types 157

8.8.4.4.1 Type: InvocationLogsRetrieveRes 157

8.8.5 Error Handling 157

8.8.5.1 General 157

8.8.5.2 Protocol Errors 157

8.8.5.3 Application Errors 157

8.8.6 Feature negotiation 157

8.9 CAPIF_API_Provider_Management_API 158

8.9.1 API URI 158

8.9.2 Resources 158

8.9.2.1 Overview 158

8.9.2.2 Resource: All API Provider Domains Registrations 159

8.9.2.2.1 Description 159

8.9.2.2.2 Resource Definition 159

8.9.2.2.3 Resource Standard Methods 159

8.9.2.2.4 Resource Custom Operations 160

8.9.2.3 Resource: Individual API Provider Domain Registration 160

8.9.2.3.1 Description 160

8.9.2.3.2 Resource Definition 160

8.9.2.3.3 Resource Standard Methods 160

8.9.2.3.4 Resource Custom Operations 163

8.9.2A Custom Operations without associated resources 163

8.9.2A.1 Overview 163

8.9.2A.2 Operation: disenroll 164

8.9.2A.2.1 Description 164

8.9.2A.2.2 Operation Definition 164

8.9.3 Notifications 165

8.9.4 Data Model 165

8.9.4.1 General 165

8.9.4.2 Structured data types 167

8.9.4.2.1 Introduction 167

8.9.4.2.2 Type: APIProviderEnrolmentDetails 167

8.9.4.2.3 Type: APIProviderFunctionDetails 168

8.9.4.2.4 Type: RegistrationInformation 168

8.9.4.2.5 Type: APIProviderEnrolmentDetailsPatch 169

8.9.4.2.6 Type: DisenrollReq 169

8.9.4.2.7 Type: DisenrollRsp 169

8.9.4.3 Simple data types and enumerations 169

8.9.4.3.1 Introduction 169

8.9.4.3.2 Simple data types 169

8.9.4.3.3 Enumeration: ApiProviderFuncRole 170

8.9.5 Error Handling 170

8.9.5.1 General 170

8.9.5.2 Protocol Errors 170

8.9.5.3 Application Errors 170

8.9.6 Feature negotiation 170

8.10 CAPIF_Routing_Info_API 171

8.10.1 API URI 171

8.10.2 Resources 171

8.10.2.1 Overview 171

8.10.2.2 Resource: Individual Service API routing info 172

8.10.2.2.1 Description 172

8.10.2.2.2 Resource Definition 172

8.10.2.2.3 Resource Standard Methods 172

8.10.2.2.4 Resource Custom Operations 173

8.10.2A Custom Operations without associated resources 173

8.10.3 Notifications 173

8.10.4 Data Model 173

8.10.4.1 General 173

8.10.4.2 Structured data types 174

8.10.4.2.1 Introduction 174

8.10.4.2.2 Type: RoutingInfo 174

8.10.4.2.3 Type: RoutingRule 174

8.10.4.2.4 Type: Ipv6AddressRange 174

8.10.4.3 Simple data types and enumerations 175

8.10.5 Error Handling 175

8.10.5.1 General 175

8.10.5.2 Protocol Errors 175

8.10.5.3 Application Errors 175

8.10.6 Feature negotiation 175

8.11 CAPIF_Open_Discover_Service_API 175

8.11.1 Introduction 175

8.11.1A Usage of HTTP 176

8.11.2 Resources 176

8.11.2.1 Overview 176

8.11.2.2 Resource: Service APIs 176

8.11.2.2.1 Description 176

8.11.2.2.2 Resource Definition 176

8.11.2.2.3 Resource Standard Methods 177

8.11.2.2.4 Resource Custom Operations 181

8.11.3 Custom Operations without associated resources 181

8.11.4 Notifications 181

8.11.5 Data Model 181

8.11.5.1 General 181

8.11.5.2 Structured data types 182

8.11.5.2.1 Introduction 182

8.11.5.2.2 Type: OpenDiscoveryResp 182

8.11.5.2.3 Type: OpenAPIDetails 182

8.11.5.2.4 Type: OpenAefProfile 183

8.11.5.3 Simple data types and enumerations 183

8.11.5.3.1 Introduction 183

8.11.5.3.2 Simple data types 183

8.11.5.4 Data types describing alternative data types or combinations of data types 183

8.11.5.5 Binary data 183

8.11.5.5.1 Binary Data Types 183

8.11.6 Error Handling 183

8.11.6.1 General 183

8.11.6.2 Protocol Errors 184

8.11.6.3 Application Errors 184

8.11.7 Feature negotiation 184

8.11.8 Security 184

9 AEF API Definition 184

9.1 AEF_Security_API 184

9.1.1 API URI 184

9.1.2 Resources 184

9.1.2A Custom Operations without associated resources 185

9.1.2A.1 Overview 185

9.1.2A.2 Operation: check-authentication 185

9.1.2A.2.1 Description 185

9.1.2A.2.2 Operation Definition 185

9.1.2A.3 Operation: revoke-authorization 186

9.1.2A.3.1 Description 186

9.1.2A.3.2 Operation Definition 186

9.1.3 Notifications 187

9.1.4 Data Model 187

9.1.4.1 General 187

9.1.4.2 Structured data types 188

9.1.4.2.1 Introduction 188

9.1.4.2.2 Type: CheckAuthenticationReq 188

9.1.4.2.3 Type: CheckAuthenticationRsp 188

9.1.4.2.4 Type: RevokeAuthorizationReq 188

9.1.4.2.5 Type: RevokeAuthorizationRsp 188

9.1.4.3 Simple data types and enumerations 189

9.1.5 Error Handling 189

9.1.5.1 General 189

9.1.5.2 Protocol Errors 189

9.1.5.3 Application Errors 189

9.1.6 Feature negotiation 189

10 Security 189

10.1 General 189

10.2 CAPIF-1/1e security 189

10.3 CAPIF-2/2e security and securely invoking service APIs 190

Annex A (normative): OpenAPI specification 191

A.1 General 191

A.2 CAPIF_Discover_Service_API 191

A.3 CAPIF_Publish_Service_API 194

A.4 CAPIF_Events_API 205

A.5 CAPIF_API_Invoker_Management_API 213

A.6 CAPIF_Security_API 219

A.7 CAPIF_Access_Control_Policy_API 227

A.8 CAPIF_Logging_API_Invocation_API 229

A.9 CAPIF_Auditing_API 232

A.10 AEF_Security_API 234

A.11 CAPIF_API_Provider_Management_API 236

A.12 CAPIF_Routing_Info_API 242

A.13 CAPIF_Open_Discover_Service_API 243

Annex B (informative): IANA registration of 3GPP defined JWT claims 248

B.1 Introduction 248

B.2 "resOwnerId" JWT claim 248

Annex C (informative): Change history 249
