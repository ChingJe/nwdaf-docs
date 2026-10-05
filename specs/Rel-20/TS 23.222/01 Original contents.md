---
spec: TS 23.222
version: 20.1.0
release: '20'
clause: contents
title: Contents
source_archive: 23222-k10.zip
source_document: 23222-k10.docx
content_origin: 3gpp-source
---

# Contents

Foreword 12

Introduction 12

1 Scope 13

2 References 13

3 Definitions and abbreviations 14

3.1 Definitions 14

3.2 Abbreviations 15

4 Architectural requirements 16

4.1 General 16

4.1.1 Introduction 16

4.1.2 Requirements 16

4.1.3 Requirements for supporting 3<sup>rd</sup> party API providers 16

4.2 Service API publish and discover 16

4.2.1 Introduction 16

4.2.2 Requirements 17

4.2.3 Requirements for 3<sup>rd</sup> party API providers 17

4.3 Security 17

4.3.1 Introduction 17

4.3.2 Requirements 17

4.3.3 Additional requirements for 3<sup>rd</sup> party API provider 18

4.4 Charging 18

4.4.1 Introduction 18

4.4.2 Requirements 18

4.4.3 Requirements for 3<sup>rd</sup> party API providers 18

4.5 Operations, Administration and Maintenance 18

4.5.1 Introduction 18

4.5.2 Requirements 18

4.5.3 Requirements for 3<sup>rd</sup> party API providers 19

4.6 Service API invocation monitoring 19

4.6.1 Introduction 19

4.6.2 Requirements 19

4.7 Logging 19

4.7.1 Introduction 19

4.7.2 Logging events related to service API invocations 19

4.7.3 Logging events related to API invoker onboarding 19

4.7.4 Logging events related to API invoker interaction with the CAPIF 20

4.8 Auditing service API invocation 20

4.8.1 Introduction 20

4.8.2 Requirements 20

4.9 Onboarding API invoker 20

4.9.1 Introduction 20

4.9.2 Requirements 20

4.10 Policy configuration 20

4.10.1 Introduction 20

4.10.2 Requirements 20

4.11 Protocol design 21

4.11.1 Introduction 21

4.11.2 Requirements 21

4.12 Interconnection between the CAPIF providers 21

4.12.1 Introduction 21

4.12.2 Requirements 21

4.13 Identities 22

4.13.1 Introduction 22

4.13.2 Requirements 22

4.14 API provider domain interactions 22

4.14.1 Introduction 22

4.14.2 Requirements 22

4.15 Dynamic routing of service API invocation 22

4.15.1 Introduction 22

4.15.2 Requirements 22

4.16 Registering API provider domain functions 22

4.16.1 Introduction 22

4.16.2 Requirements 22

4.17 Resource owner-aware northbound API invocation 23

4.17.1 Introduction 23

4.17.2 Requirements 23

4.18 API Analytics 23

4.18.1 Introduction 23

4.18.2 Requirements 23

5 Involved business relationships 23

5.1 Basic CAPIF business relationships 23

5.2 CAPIF business relationships for RNAA 24

6 Functional model 25

6.1 General 25

6.2 Functional model description 26

6.2.0 Functional model description for the CAPIF 26

6.2.1 Functional model description to support 3<sup>rd</sup> party API providers 28

6.2.2 Functional model description to support CAPIF interconnection 29

6.2.3 Functional model description to support RNAA 31

6.3 Functional entities description 33

6.3.1 General 33

6.3.2 API invoker 33

6.3.3 CAPIF core function 33

6.3.4 API exposing function 34

6.3.5 API publishing function 34

6.3.6 API management function 34

6.3.7 Authorization function 35

6.3.8 Resource owner function 35

6.4 Reference points 35

6.4.1 General 35

6.4.2 Reference point CAPIF-1 (between the API invoker and the CAPIF core function) 35

6.4.3 Reference point CAPIF-1e (between the API invoker and the CAPIF core function) 36

6.4.4 Reference point CAPIF-2 (between the API invoker and the API exposing function) 36

6.4.5 Reference point CAPIF-2e (between the API invoker and the API exposing function) 36

6.4.6 Reference point CAPIF-3 (between the API exposing function and the CAPIF core function) 36

6.4.7 Reference point CAPIF-4 (between the API publishing function and the CAPIF core function) 37

6.4.8 Reference point CAPIF-5 (between the API management function and the CAPIF core function) 37

6.4.9 Reference point CAPIF-3e (between the API exposing function and the CAPIF core function) 37

6.4.10 Reference point CAPIF-4e (between the API publishing function and the CAPIF core function) 38

6.4.11 Reference point CAPIF-5e (between the API management function and the CAPIF core function) 38

6.4.12 Reference point CAPIF-7 (between the API exposing functions) 38

6.4.13 Reference point CAPIF-7e (between the API exposing functions) 38

6.4.14 Reference point CAPIF-6 (between the CAPIF core functions of the same CAPIF provider) 38

6.4.15 Reference point CAPIF-6e (between the CAPIF core functions of different CAPIF providers) 39

6.4.16 Reference point CAPIF-8 (between the CAPIF core function and the resource owner function) 39

6.5 Service-based interfaces 39

7 Application of functional model to deployments 39

7.1 General 39

7.2 Centralized deployment 39

7.3 Distributed deployment 40

7.4 Multiple CCFs deployment 44

7.5 RNAA deployments 45

8 Procedures and information flows 45

8.1 Onboarding the API invoker to the CAPIF 45

8.1.1 General 45

8.1.2 Information flows 45

8.1.2.1 Onboard API invoker request 45

8.1.2.2 Onboard API invoker response 46

8.1.3 Procedure 46

8.2 Offboarding the API invoker from the CAPIF 47

8.2.1 General 47

8.2.2 Information flows 47

8.2.2.1 Offboard API invoker request 47

8.2.2.2 Offboard API invoker response 47

8.2.3 Procedure 48

8.3 Publish service APIs 48

8.3.1 General 48

8.3.2 Information flows 49

8.3.2.1 Service API publish request 49

8.3.2.2 Service API publish response 50

8.3.3 Procedure 50

8.4 Unpublish service APIs 51

8.4.1 General 51

8.4.2 Information flows 51

8.4.2.1 Service API unpublish request 51

8.4.2.2 Service API unpublish response 52

8.4.3 Procedure 52

8.5 Retrieve service APIs 52

8.5.1 General 52

8.5.2 Information flows 53

8.5.2.1 Service API get request 53

8.5.2.2 Service API get response 53

8.5.3 Procedure 53

8.6 Update service APIs 54

8.6.1 General 54

8.6.2 Information flows 54

8.6.2.1 Service API update request 54

8.6.2.2 Service API update response 54

8.6.3 Procedure 55

8.7 Discover service APIs 55

8.7.1 General 55

8.7.2 Information flows 56

8.7.2.1 Service API discover request 56

8.7.2.2 Service API discover response 56

8.7.3 Procedure 57

8.8 Subscription, unsubscription and notifications for the CAPIF events 58

8.8.1 General 58

8.8.2 Information flows 58

8.8.2.1 Event subscription request 58

8.8.2.2 Event subscription response 59

8.8.2.3 Event notification 59

8.8.2.4 Event notification acknowledgement 59

8.8.2.5 Event unsubscription request 60

8.8.2.6 Event unsubscription response 60

8.8.2.7 Event subscription update request 60

8.8.2.8 Event subscription update response 60

8.8.3 Procedure for CAPIF event subscription 61

8.8.4 Procedure for CAPIF event notifications 61

8.8.5 Procedure for CAPIF event unsubscription 62

8.8.5a Procedure for CAPIF event subscription update 63

8.8.6 List of CAPIF events 63

8.9 Revoking subscription of the CAPIF events 64

8.9.1 General 64

8.9.2 Information flows 64

8.9.2.1 Subscription revoke notification 64

8.9.2.2 Subscription revoke notification acknowledgement 65

8.9.3 Procedure 65

8.10 Authentication between the API invoker and the CAPIF core function 65

8.10.1 General 65

8.10.2 Information flows 66

8.10.3 Procedure 66

8.10A API Invoker obtaining security method to access service API 66

8.10A.1 General 66

8.10A.2 Information flows 66

8.10A.3 Procedure 66

8.11 API invoker obtaining authorization to access service API 67

8.11.1 General 67

8.11.2 Information flows 67

8.11.3 Procedure 67

8.12 AEF obtaining service API access control policy 68

8.12.1 General 68

8.12.2 Information flows 68

8.12.2.1 Obtain access control policy request 68

8.12.2.2 Obtain access control policy response 68

8.12.3 Procedure 68

8.13 Topology hiding 69

8.13.1 General 69

8.13.2 Information flows 69

8.13.2.1 Service API invocation request (API invoker – AEF-1) 69

8.13.2.2 Service API invocation request (AEF-1 – AEF-2) 69

8.13.2.3 Service API invocation response (AEF-2 – AEF-1) 70

8.13.2.4 Service API invocation response (AEF-1 – API invoker) 70

8.13.3 Procedure 70

8.14 Authentication between the API invoker and the AEF prior to service API invocation 71

8.14.1 General 71

8.14.2 Information flows 71

8.14.3 Procedure 71

8.15 Authentication between the API invoker and the AEF upon the service API invocation 72

8.15.1 General 72

8.15.2 Information flows 72

8.15.2.1 Service API invocation request with authentication information 72

8.15.2.2 Service API invocation response 72

8.15.3 Procedure 72

8.16 Service API invocation with AEF authorization 73

8.16.1 General 73

8.16.2 Information flows 74

8.16.2.1 Service API invocation request 74

8.16.2.2 Service API invocation response 74

8.16.2.3 Obtain API invoker information request 74

8.16.2.4 Obtain API invoker information response 74

8.16.3 Procedure 75

8.17 CAPIF access control 76

8.17.1 General 76

8.17.2 Information flows 76

8.17.2.1 Service API invocation request 76

8.17.2.2 Service API invocation response 76

8.17.3 Procedure 76

8.18 CAPIF access control with cascaded AEFs 77

8.18.1 General 77

8.18.2 Information flows 77

8.18.2.1 Service API invocation request 77

8.18.2.2 Service API invocation response 77

8.18.3 Procedure 78

8.19 Logging service API invocations 78

8.19.1 General 78

8.19.2 Information flows 79

8.19.2.1 API invocation log request 79

8.19.2.2 API invocation log response 79

8.19.3 Procedure 79

8.20 Charging the invocation of service APIs 80

8.20.1 General 80

8.20.2 Information flows 80

8.20.3 Procedure 80

8.21 Monitoring service API invocation 80

8.21.1 General 80

8.21.2 Information flows 81

8.21.2.1 Monitoring service API event notification 81

8.21.2.2 Monitoring service API event notification acknowledgement 81

8.21.3 Procedure 81

8.22 Auditing service API invocation 81

8.22.1 General 81

8.22.2 Information flows 82

8.22.2.1 Query service API log request 82

8.22.2.2 Query service API log response 82

8.22.3 Procedure 82

8.23 CAPIF revoking API invoker authorization 83

8.23.1 General 83

8.23.2 Information flows 83

8.23.2.1 Revoke API invoker authorization request 83

8.23.2.2 Revoke API invoker authorization response 84

8.23.2.3 Revoke API invoker authorization notify 84

8.23.2.4 Revoke API invoker authorization notify acknowledgement 84

8.23.3 Procedure for CAPIF revoking API invoker authorization initiated by AEF 85

8.23.4 Procedure for CAPIF revoking API invoker authorization initiated by CAPIF core function 86

8.24 API topology hiding management 87

8.24.1 General 87

8.24.2 Information flows 87

8.24.2.1 API topology hiding notify 87

8.24.3 Procedure 87

8.25 Support for CAPIF interconnection 88

8.25.1 General 88

8.25.2 Information flows 88

8.25.2.1 Interconnection API publish request 88

8.25.2.2 Interconnection API publish response 88

8.25.2.3 Interconnection service API discover request 89

8.25.2.4 Interconnection service API discover response 89

8.25.2.5 Interconnection API unpublish request 89

8.25.2.6 Interconnection API unpublish response 90

8.25.2.7 Interconnection get service API request 90

8.25.2.8 Interconnection get service API response 90

8.25.2.9 Interconnection update service API request 91

8.25.2.10 Interconnection update service API response 91

8.25.2.11 Interconnection revoke API invoker authorization request 91

8.25.2.12 Interconnection revoke API invoker authorization response 92

8.25.2.13 Interconnection revoke API invoker authorization notify 92

8.25.2.14 Interconnection obtain access control policy request 93

8.25.2.15 Interconnection obtain access control policy response 93

8.25.2.16 Interconnection API invoker obtaining authorization to access service API 93

8.25.2.17 Interconnection Authentication and authorization between the API invoker and the AEF 93

8.25.3 Procedure 93

8.25.3.1 Service API publish for CAPIF interconnection 93

8.25.3.2 Service API discovery involving multiple CCFs 94

8.25.3.3 Service API discovery for CAPIF interconnection 95

8.25.3.4 Service API unpublish for CAPIF interconnection 96

8.25.3.5 Retrieve service APIs for CAPIF interconnection 97

8.25.3.6 Update service APIs for CAPIF interconnection 98

8.25.3.7 API invoker obtaining authorization for service API access in CAPIF interconnection 99

8.25.3.8 Procedure for CAPIF revoking API invoker authorization in CAPIF interconnection 100

8.25.3.9 Procedure for obtaining access control policy in CAPIF interconnection 101

8.25.3.10 Procedure for obtaining security information in CAPIF interconnection 101

8.26 Update API invoker's API list 102

8.26.1 General 102

8.26.2 Information flows 102

8.26.2.1 Update API invoker API list request 102

8.26.2.2 Update API invoker API list response 102

8.26.3 Procedure 103

8.27 Dynamically routing service API invocation 104

8.27.1 General 104

8.27.2 Information flows 104

8.27.2.1 Obtain routing information request 104

8.27.2.2 Obtain routing information response 104

8.27.3 Procedure 104

8.28 Registering the API provider domain functions on the CAPIF 105

8.28.1 General 105

8.28.2 Information flows 105

8.28.2.1 Registration request 105

8.28.2.2 Registration response 106

8.28.3 Procedure 106

8.29 Update registration information of the API provider domain functions on the CAPIF 107

8.29.1 General 107

8.29.2 Information flows 107

8.29.2.1 Registration update request 107

8.29.2.2 Registration update response 107

8.29.3 Procedure 108

8.30 Deregistering the API provider domain functions on the CAPIF 109

8.30.1 General 109

8.30.2 Information flows 109

8.30.2.1 Deregistration request 109

8.30.2.2 Deregistration response 109

8.30.3 Procedure 109

8.31 API invoker obtaining authorization from resource owner 110

8.31.1 General 110

8.31.2 Information flows 110

8.31.3 Procedure 110

8.32 Reducing authorization information inquiry in a nested API invocation 112

8.32.1 General 112

8.32.2 Information flows 112

8.32.3 Procedure 112

8.34 UE-deployed API invoker accessing other UEs’ resources of a group 115

8.34.1 General 115

8.34.2 Information flows 115

8.34.3 Procedure 116

8.34a API invoker deployed in an application server accessing UE’s resources of a group of UEs 117

8.34a.1 General 117

8.34a.2 Information flows 117

8.34a.3 Procedure 117

8.35 Revoking resource owner authorization 118

8.35.1 General 118

8.35.2 Information flows 118

8.35.3 Procedure 119

8.36 CCF Obtaining Resource Owner Authorization 119

8.36.1 General 119

8.36.2 Information flows 120

8.36.3 Procedure 120

8.37 UE-deployed API invoker accessing other UEs’ resources 120

8.37.1 General 120

8.37.2 Information flows 121

8.37.3 Procedure 121

8.38 Open Discover service APIs 122

8.38.1 General 122

8.38.2 Information flows 122

8.38.2.1 Open Service API discover request 122

8.38.2.2 Open Service API discover response 122

8.38.3 Procedure 122

8.39 API provider domain remove enrolled APIs for API invoker 123

8.39.1 General 123

8.39.2 Information flows 123

8.39.2.1 Remove enrolled service APIs request 123

8.39.2.2 Remove enrolled service APIs response 124

8.39.3 Procedure 124

8.40 API invoker reporting errors 125

8.40.1 General 125

8.40.2 Information flows 125

8.40.2.1 API invocation error reporting request 125

8.40.2.2 API invocation error reporting response 125

8.40.3 Procedure 125

8.41 Service API performance analytics 127

8.41.1 General 127

8.42 AEF Fetch exposure policy 127

8.42.1 General 127

8.42.2 Information flows 127

8.42.2.1 Select exposure policy request 127

8.42.2.2 Select exposure policy response 127

8.42.3 Procedure 128

9 API consistency guidelines 129

9.1 General 129

9.2 Fundamental API Guidelines 129

9.3 Architecture design considerations 130

10 CAPIF core function APIs 130

10.1 General 130

10.2 CAPIF_Discover_Service_API API 132

10.2.1 General 132

10.2.2 Discover_Service_API operation 132

10.2.3 Subscribe_Event operation 132

10.2.4 Notify_Event operation 133

10.2.5 Unsubscribe_Event operation 133

10.2.6 Update_Event_Subscription operation 133

10.3 CAPIF_Publish_Service_API API 133

10.3.1 General 133

10.3.2 Publish_Service_API operation 133

10.3.3 Unpublish_Service_API operation 134

10.3.4 Update_Service_API operation 134

10.3.5 Get_Service_API operation 134

10.3.6 Subscribe_Event operation 134

10.3.7 Notify_Event operation 134

10.3.8 Unsubscribe_Event operation 135

10.3.9 Update_Event_Subscription operation 135

10.4 CAPIF_Events API 135

10.4.1 General 135

10.4.2 Subscribe_Event operation 136

10.4.3 Notify_Event operation 136

10.4.4 Unsubscribe_Event operation 136

10.4.5 Update_Event_Subscription operation 136

10.5 CAPIF_API_invoker_management API 137

10.5.1 General 137

10.5.2 Onboard_API_Invoker operation 137

10.5.2a Update_API_Invoker_Details operation 137

10.5.3 Offboard_API_Invoker operation 137

10.5.4 Subscribe_Event operation 137

10.5.5 Notify_Event operation 138

10.5.6 Unsubscribe_Event operation 138

10.5.7 Update_Event_Subscription operation 138

10.6 CAPIF_Security API 138

10.6.1 General 138

10.6.2 Obtain_Security_Method operation 138

10.6.3 Obtain_Authorization operation 139

10.6.4 Obtain_API_Invoker_Info operation 139

10.6.5 Revoke_Authorization operation 139

10.6.6 Revoke_Authorization_Notify operation 139

10.7 CAPIF_Monitoring API 140

10.7.1 General 140

10.7.2 Subscribe_Event operation 140

10.7.3 Notify_Monitoring_Service_Event operation 140

10.7.4 Unsubscribe_Event operation 140

10.7.5 Update_Event_Subscription operation 140

10.8 CAPIF_Logging_API_Invocation API 141

10.8.1 General 141

10.8.2 Log_API_Invocation operation 141

10.9 CAPIF_Auditing API 141

10.9.1 General 141

10.9.2 Query\_ API_Invocation_Log operation 141

10.10 CAPIF_Access_Control_Policy API 141

10.10.1 General 141

10.10.2 Obtain_Access_Control_Policy operation 142

10.11 CAPIF_Routing_Info API 142

10.11.1 General 142

10.11.2 Obtain_Routing_Info operation 142

10.12 CAPIF_API_provider_management API 142

10.12.1 General 142

10.12.2 Register_API_Provider operation 142

10.12.3 Update_API_Provider operation 142

10.12.4 Deregister_API_Provider operation 143

10.12.5 Disenroll_Service_APIs operation 143

10.13 CAPIF_Open_Discover_Service_API API 143

10.13.1 General 143

10.13.2 Open_Discover_Service_API operation 143

10.14 CAPIF_API_Invocation_Error_Reporting API 144

10.14.1 General 144

10.14.2 Report_API_Invocation_Error operation 144

10.15 CAPIF_AEF_Exposure_Policy API 144

10.15.1 General 144

10.15.2 AEF_Exposure_Policy operation 144

11 API exposing function APIs 144

11.1 General 144

11.2 AEF_Security API 145

11.2.1 General 145

11.2.2 Revoke_Authorization operation 145

11.2.3 Initiate_Authentication operation 145

Annex A (informative): Overview of CAPIF operations 146

Annex B (informative): CAPIF relationship with network exposure aspects of 3GPP systems 148

B.0 CAPIF utilization by service API provider 148

B.1 CAPIF relationship with 3GPP EPS network exposure 149

B.1.1 General 149

B.1.2 Deployment models 149

B.1.2.1 General 149

B.1.2.2 SCEF implements the CAPIF architecture 150

B.1.2.3 SCEF implements the service specific aspect compliant with the CAPIF architecture 150

B.1.2.4 Distributed deployment of the SCEF compliant with the CAPIF architecture 151

B.2 CAPIF relationship with 3GPP 5GS network exposure 152

B.2.1 General 152

B.2.2 Deployment models 153

B.2.2.1 General 153

B.2.2.2 NEF implements the CAPIF architecture 153

B.2.2.3 NEF implements the service specific aspect compliant with the CAPIF architecture 154

B.2.2.4 Distributed deployment of the NEF compliant with the CAPIF architecture 155

B.3 Integrated deployment of 3GPP network exposure systems with the CAPIF 156

B.3.1 General 156

B.3.2 Deployment model 157

B.3.2.1 General 157

B.3.2.2 Integrated deployment of the SCEF and the NEF with the CAPIF 157

Annex C (informative): CAPIF role in charging 159

C.1 General 159

C.2 CAPIF role in online charging 159

C.3 CAPIF role in offline charging 159

Annex D (informative): CAPIF relationship with external API frameworks 160

Annex E (normative): Configuration data for CAPIF 161

Annex F (informative): Examples of API invoker roles in CAPIF 163

Annex G (informative): API invoker with a frontend and backend component 164

Annex H (informative): AEF instantiation 165

Annex I (informative): Change history 167
