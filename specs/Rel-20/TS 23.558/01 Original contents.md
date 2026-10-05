---
spec: TS 23.558
version: 20.3.0
release: '20'
clause: contents
title: Contents
source_archive: 23558-k30.zip
source_document: 23558-k30.docx
content_origin: 3gpp-source
---

# Contents

Foreword 18

Introduction 18

1 Scope 19

2 References 19

3 Definitions of terms, symbols and abbreviations 20

3.1 Terms 20

3.2 Symbols 21

3.3 Abbreviations 21

4 Overview 22

4.1 General 22

4.2 Service provisioning 23

4.3 Registration 23

4.4 EAS discovery 23

4.5 Capability exposure to EAS and EEC 23

4.6 Support for service continuity 23

4.7 Security 23

4.8 Dynamic EAS instantiation triggering 24

4.9 Charging 24

4.10 Common EAS discovery 24

4.11 Bundle EAS 24

4.11.1 Direct bundle 24

4.11.2 Proxy bundle 24

4.12 Federation and Roaming 24

4.13 EAS content synchronization and Application context 24

5 Architectural requirements 25

5.1 General 25

5.2 Architectural requirements 25

5.2.1 General requirements 25

5.2.1.1 General 25

5.2.1.2 Requirements 25

5.2.2 Edge configuration data 25

5.2.2.1 General 25

5.2.2.2 Requirements 25

5.2.3 Registration 25

5.2.3.1 General 25

5.2.3.2 EEC registration 25

5.2.3.3 EAS registration 25

5.2.3.4 EES registration 26

5.2.4 EAS discovery 26

5.2.4.1 General 26

5.2.4.2 Requirements 26

5.2.5 Capability exposure to EASs 26

5.2.5.1 General 26

5.2.5.2 Requirements 26

5.2.6 Security 26

5.2.6.1 General 26

5.2.6.2 Requirements 27

5.2.7 Subscription service 27

5.2.7.1 General 27

5.2.7.2 Requirements 27

5.2.8 Traffic management 27

5.2.8.1 General 27

5.2.8.2 Requirements 28

5.2.9 Lifecycle management 28

5.2.9.1 General 28

5.2.9.2 Requirements 28

5.2.10 Edge application KPIs 28

5.2.10.1 General 28

5.2.10.2 Requirements 28

5.2.11 Service continuity 28

5.2.11.1 General 28

5.2.11.2 Requirements 28

6 Application layer architecture 28

6.1 General 28

6.2 Architecture 29

6.2a Architecture for Roaming support 31

6.2a.1 General 31

6.2a.2 Local breakout roaming architecture: Local breakout to access H-ECS 31

6.2a.3 Home-routed EDGE-4 access to H-ECS 32

6.2b Architecture for Federation support 33

6.2b.1 General 33

6.2b.2 Architecture 33

6.2c Architecture for enabling cloud applications with edge applications 34

6.2d Architecture for enabling cloud applications with edge applications, with CES support 35

6.3 Functional entities 36

6.3.1 General 36

6.3.2 Edge Enabler Server (EES) 36

6.3.3 Edge Enabler Client (EEC) 37

6.3.4 Edge Configuration Server (ECS) 37

6.3.5 Application Client (AC) 38

6.3.6 Edge Application Server (EAS) 38

6.3.7 Notification management client 38

6.3.8 Notification management server 38

6.3.9 Cloud Enabler Server (CES) 38

6.3.10 Cloud application server (CAS) 39

6.4 Service-based interfaces 39

6.5 Reference Points 39

6.5.1 General 39

6.5.2 EDGE-1 39

6.5.3 EDGE-2 39

6.5.4 EDGE-3 39

6.5.5 EDGE-4 40

6.5.6 EDGE-5 40

6.5.7 EDGE-6 40

6.5.8 EDGE-7 40

6.5.9 EDGE-8 40

6.5.10 EDGE-9 41

6.5.11 NM-UU 41

6.5.12 NM-S 41

6.5.13 NM-C 42

6.5.14 ECI-1 42

6.5.15 ECI-2 42

6.5.16 ECI-3 42

6.5.17 ECI-4 42

6.5.18 CLOUD-1 42

6.5.19 CLOUD-2 42

6.5.20 CLOUD-3 43

6.5.21 CLOUD-4 43

6.5.22 EDGE-10 43

6.6 Cardinality rules 43

6.6.1 General 43

6.6.2 Functional Entity Cardinality 43

6.6.2.1 General 43

6.6.2.2 AC 43

6.6.2.3 EEC 43

6.6.2.4 ECS 43

6.6.2.5 EES 44

6.6.2.6 EAS 44

6.6.2.7 CES 44

6.6.2.8 CAS 44

6.6.3 Reference Point Cardinality 44

6.6.3.1 General 44

6.6.3.2 EDGE-1 (Between EEC and EES) 44

6.6.3.3 EDGE-3 (Between EAS and EES) 44

6.6.3.4 EDGE-4 (Between EEC and ECS) 44

6.6.3.5 EDGE-5 (Between AC and EEC) 45

6.6.3.6 EDGE-6 (Between EES and ECS) 45

6.6.3.7 EDGE-9 (Between EES and EES) 45

6.6.3.8 EDGE-10 (Between ECS and ECS) 45

6.6.3.9 ECI-1 (Between CAS and EES) 45

6.6.3.10 ECI-2 (Between CAS and ECS) 45

6.6.3.11 ECI-3 (Between CES and ECS) 45

6.6.3.12 ECI-4 (Between CES and EES) 45

6.6.3.13 CLOUD-1 (Between CAS and CES) 46

6.6.3.14 CLOUD-2 (Between CES and CES) 46

6.7 Capability exposure for enabling edge applications 46

6.7.1 General 46

6.7.2 APIs provided by the Edge Enabler Layer 46

6.8 Capability exposure for enabling cloud applications with edge applications, with CES support 48

6.8.1 General 48

6.9 Capability exposure for enabling cloud applications with edge applications 49

6.9.1 General 49

7 Identities and commonly used values 50

7.1 General 50

7.2 Identities 50

7.2.1 General 50

7.2.2 Edge Enabler Client ID (EECID) 50

7.2.3 Edge Enabler Server ID (EESID) 50

7.2.4 Edge Application Server ID (EASID) 50

7.2.5 Application Client ID (ACID) 50

7.2.6 UE ID 50

7.2.7 UE Group ID 50

7.2.8 EEC Context ID 51

7.2.9 Edge UE ID 51

7.2.10 EAS bundle information 51

7.2.11 Application Group ID 51

7.3 Commonly used values 52

7.3.1 General 52

7.3.2 UE location 52

7.3.3 Service areas 52

7.3.3.1 General 52

7.3.3.2 Topological Service Area 52

7.3.3.3 Geographical Service Area 52

7.3.3.4 EDN service area 52

7.3.3.5 EES Service Area 53

7.3.3.6 EAS service area 53

8 Procedures and information flows 53

8.1 General 53

8.2 Common Information Elements 53

8.2.1 General 53

8.2.2 AC Profile 53

8.2.3 AC Service KPIs 54

8.2.4 EAS Profile 55

8.2.5 EAS Service KPIs 57

8.2.6 EES Profile 57

8.2.7 Topological Service Area 58

8.2.8 EEC Context 59

8.2.9 Geographical Service Area 59

8.2.10 EAS bundle requirements 60

8.2.11 Application Group profile 60

8.2.12 ECS Profile 60

8.3 ECS Discovery and Service provisioning 61

8.3.1 General 61

8.3.2 ECS Discovery 63

8.3.2.1 General 63

8.3.2.2 Procedures 64

8.3.2.2.1 General 64

8.3.2.3 Information flows 64

8.3.2.3.1 General 64

8.3.2.4 APIs 64

8.3.2.4.1 General 64

8.3.3 Service provisioning 64

8.3.3.1 General 64

8.3.3.2 Procedures 64

8.3.3.2.1 General 64

8.3.3.2.2 Request-response model 65

8.3.3.2.3 Subscribe-notify model 68

8.3.3.2.3.1 General 68

8.3.3.2.3.2 Subscribe 68

8.3.3.2.3.3 Notify 69

8.3.3.2.3.4 Subscription update 71

8.3.3.2.3.5 Unsubscribe 72

8.3.3.3 Information flows 72

8.3.3.3.1 General 72

8.3.3.3.2 Service provisioning request 73

8.3.3.3.3 Service provisioning response 73

8.3.3.3.4 Service provisioning subscription request 76

8.3.3.3.5 Service provisioning subscription response 76

8.3.3.3.6 Service provisioning notification 76

8.3.3.3.7 Service provisioning subscription update request 77

8.3.3.3.8 Service provisioning subscription update response 77

8.3.3.3.9 Service provisioning unsubscribe request 78

8.3.3.3.10 Service provisioning unsubscribe response 78

8.3.3.4 APIs 78

8.3.3.4.1 General 78

8.3.3.4.2 Eecs_ServiceProvisioning_Request operation 78

8.3.3.4.3 Eecs_ServiceProvisioning_Subscribe operation 78

8.3.3.4.4 Eecs_ServiceProvisioning_Notify operation 79

8.3.3.4.5 Eecs_ServiceProvisioning_UpdateSubscription operation 79

8.3.3.4.6 Eecs_ServiceProvisioning_Unsubscribe operation 79

8.4 Registration 79

8.4.1 General 79

8.4.2 EEC Registration 80

8.4.2.1 General 80

8.4.2.2 Procedures 80

8.4.2.2.1 General 80

8.4.2.2.2 EEC registration 80

8.4.2.2.3 EEC registration update 82

8.4.2.2.4 EEC de-registration 83

8.4.2.3 Information flows 83

8.4.2.3.1 General 83

8.4.2.3.2 EEC registration request 83

8.4.2.3.3 EEC registration response 84

8.4.2.3.4 EEC registration update request 85

8.4.2.3.5 EEC registration update response 85

8.4.2.3.6 EEC de-registration request 86

8.4.2.3.7 EEC de-registration response 86

8.4.2.4 APIs 86

8.4.2.4.1 General 86

8.4.2.4.2 Eees_EECRegistration_Request operation 87

8.4.2.4.3 Eees_EECRegistration_Update operation 87

8.4.2.4.4 Eees_EECRegistration_Deregister operation 87

8.4.3 EAS Registration 87

8.4.3.1 General 87

8.4.3.2 Procedures 88

8.4.3.2.1 General 88

8.4.3.2.2 EAS registration 88

8.4.3.2.3 EAS registration update 88

8.4.3.2.4 EAS de-registration 89

8.4.3.3 Information flows 90

8.4.3.3.1 General 90

8.4.3.3.2 EAS registration request 90

8.4.3.3.3 EAS registration response 90

8.4.3.3.4 EAS registration update request 91

8.4.3.3.5 EAS registration update response 91

8.4.3.3.6 EAS de-registration request 91

8.4.3.3.7 EAS de-registration response 92

8.4.3.4 APIs 92

8.4.3.4.1 General 92

8.4.3.4.2 Eees_EASRegistration_Request operation 92

8.4.3.4.3 Eees_EASRegistration_Update operation 92

8.4.3.4.4 Eees_EASRegistration_Deregister operation 92

8.4.4 EES Registration 93

8.4.4.1 General 93

8.4.4.2 Procedures 93

8.4.4.2.1 General 93

8.4.4.2.2 EES registration 93

8.4.4.2.3 EES registration update 93

8.4.4.2.4 EES de-registration 94

8.4.4.3 Information elements 95

8.4.4.3.1 General 95

8.4.4.3.2 EES registration request 95

8.4.4.3.3 EES registration response 95

8.4.4.3.4 EES registration update request 95

8.4.4.3.5 EES registration update response 95

8.4.4.3.6 EES de-registration request 96

8.4.4.3.7 EES de-registration response 96

8.4.4.4 APIs 96

8.4.4.4.1 General 96

8.4.4.4.2 Eecs_EESRegistration_Request operation 96

8.4.4.4.3 Eecs_EESRegistration_Update operation 97

8.4.4.4.4 Eecs_EESRegistration_Deregister operation 97

8.5 EAS discovery 97

8.5.1 General 97

8.5.2 Procedures 98

8.5.2.1 General 98

8.5.2.2 Request-response model 98

8.5.2.3 Subscribe-notify model 102

8.5.2.3.1 General 102

8.5.2.3.2 Subscribe 103

8.5.2.3.3 Notify 104

8.5.2.3.4 Subscription update 106

8.5.2.3.5 Unsubscribe 106

8.5.3 Information flows 107

8.5.3.1 General 107

8.5.3.2 EAS discovery request 107

8.5.3.3 EAS discovery response 109

8.5.3.4 EAS discovery subscription request 110

8.5.3.5 EAS discovery subscription response 112

8.5.3.6 EAS discovery notification 112

8.5.3.7 EAS discovery subscription update request 113

8.5.3.8 EAS discovery subscription update response 114

8.5.3.9 EAS discovery unsubscribe request 114

8.5.3.10 EAS discovery unsubscribe response 114

8.5.4 APIs 114

8.5.4.1 General 114

8.5.4.2 Eees_EASDiscovery_Request operation 115

8.5.4.3 Eees_EASDiscovery_Subscribe operation 115

8.5.4.4 Eees_EASDiscovery_Notify operation 115

8.5.4.5 Eees_EASDiscovery_UpdateSubscription operation 115

8.5.4.6 Eees_EASDiscovery_Unsubscribe operation 115

8.6 EES capability exposure to EAS and EEC 116

8.6.1 General 116

8.6.2 UE location API 116

8.6.2.1 General 116

8.6.2.2 Procedures 116

8.6.2.2.1 General 116

8.6.2.2.2 Request-response model 116

8.6.2.2.3 Subscribe-notify model 117

8.6.2.2.3.1 General 117

8.6.2.2.3.2 Subscribe 117

8.6.2.2.3.3 Notify 118

8.6.2.2.3.4 Subscription update 119

8.6.2.2.3.5 Unsubscribe 120

8.6.2.3 Information flows 120

8.6.2.3.1 General 120

8.6.2.3.2 UE location request 121

8.6.2.3.3 UE location response 121

8.6.2.3.4 UE location subscribe request 121

8.6.2.3.5 UE location subscribe response 121

8.6.2.3.6 UE location notification 122

8.6.2.3.7 UE location subscription update request 122

8.6.2.3.8 UE location subscription update response 122

8.6.2.3.9 UE location unsubscribe request 122

8.6.2.3.10 UE location unsubscribe response 123

8.6.2.4 APIs 123

8.6.2.4.1 General 123

8.6.2.4.2 Eees_UELocation_Get operation 123

8.6.2.4.3 Eees_UELocation_Subscribe operation 123

8.6.2.4.4 Eees_UELocation_Notify operation 123

8.6.2.4.5 Eees_UELocation_UpdateSubscription operation 124

8.6.2.4.6 Eees_UELocation_Unsubscribe operation 124

8.6.3 ACR management events 124

8.6.3.1 General 124

8.6.3.2 Procedures 125

8.6.3.2.1 General 125

8.6.3.2.2 Subscribe 125

8.6.3.2.3 Notify 126

8.6.3.2.4 Subscription update 128

8.6.3.2.5 Unsubscribe 129

8.6.3.3 Information flows 130

8.6.3.3.1 General 130

8.6.3.3.2 ACR management event subscribe request 130

8.6.3.3.3 ACR management event subscribe response 133

8.6.3.3.4 ACR management event notification 133

8.6.3.3.5 ACR management event subscription update request 134

8.6.3.3.6 ACR management event subscription update response 135

8.6.3.3.7 ACR management event unsubscribe request 135

8.6.3.3.8 ACR management event unsubscribe response 136

8.6.3.4 APIs 136

8.6.3.4.1 General 136

8.6.3.4.2 Eees_ACRManagementEvent_Subscribe operation 136

8.6.3.4.3 Eees_ACRManagementEvent_Notify operation 136

8.6.3.4.4 Eees_ACRManagementEvent_UpdateSubscription operation 136

8.6.3.4.5 Eees_ACRManagementEvent_Unsubscribe operation 137

8.6.4 AC information exposure API 137

8.6.4.1 General 137

8.6.4.2 Procedures 137

8.6.4.2.1 General 137

8.6.4.2.2 Subscribe 137

8.6.4.2.3 Notify 138

8.6.4.2.4 Subscription update 138

8.6.4.2.5 Unsubscribe 139

8.6.4.3 Information flows 139

8.6.4.3.1 General 139

8.6.4.3.2 AC information subscription request 139

8.6.4.3.3 AC information subscription response 141

8.6.4.3.4 AC information notification 142

8.6.4.3.5 AC information subscription update request 142

8.6.4.3.6 AC information subscription update response 142

8.6.4.3.7 AC information unsubscribe request 143

8.6.4.3.8 AC information unsubscribe response 143

8.6.4.4 APIs 143

8.6.4.4.1 General 143

8.6.4.4.2 Eees_AppClientInformation_Subscribe operation 143

8.6.4.4.3 Eees_AppClientInformation_Notify operation 143

8.6.4.4.4 Eees_AppClientInformation_UpdateSubscription operation 144

8.6.4.4.5 Eees_AppClientInformation_Unsubscribe operation 144

8.6.5 UE Identifier API 144

8.6.5.1 General 144

8.6.5.2 Procedure 144

8.6.5.3 Information flows 145

8.6.5.3.1 General 145

8.6.5.3.2 UE Identifier API request 145

8.6.5.3.3 UE Identifier API response 146

8.6.5.4 APIs 146

8.6.5.4.1 General 146

8.6.5.4.2 Eees_UEIdentifier_Get operation 146

8.6.6 Session with QoS API 147

8.6.6.1 General 147

8.6.6.2 Procedures 147

8.6.6.2.1 General 147

8.6.6.2.2 Create a session 147

8.6.6.2.3 Update a session 149

8.6.6.2.4 Revoke a session 150

8.6.6.2.5 Notify 150

8.6.6.3 Information flows 151

8.6.6.3.1 General 151

8.6.6.3.2 Session with QoS create request 151

8.6.6.3.3 Session with QoS create response 152

8.6.6.3.4 Session with QoS update request 153

8.6.6.3.5 Session with QoS update response 153

8.6.6.3.6 Session with QoS revoke request 153

8.6.6.3.7 Session with QoS revoke response 154

8.6.6.3.8 Session with QoS event notification 154

8.6.6.4 APIs 154

8.6.6.4.1 General 154

8.6.6.4.2 Eees_SessionWithQoS_Create operation 154

8.6.6.4.3 Eees_SessionWithQoS_Update operation 155

8.6.6.4.4 Eees_SessionWithQoS_Revoke operation 155

8.6.6.4.5 Eees_SessionWithQoS_Notify operation 155

8.6.7 Application traffic influence trigger from EAS 155

8.6.7.1 General 155

8.6.7.2 Procedure 155

8.6.7.2.1 Procedure of application traffic influence trigger from EAS 155

8.6.7.2.2 Procedure of application traffic influence update trigger from EAS 156

8.6.7.2.3 Procedure of application traffic influence cancellation trigger from EAS 157

8.6.7.3 Information flows 157

8.6.7.3.1 General 157

8.6.7.3.2 Application traffic influence trigger from EAS request 158

8.6.7.3.3 Application traffic influence trigger from EAS response 158

8.6.7.3.4 Application traffic influence update trigger from EAS request 158

8.6.7.3.5 Application traffic influence update trigger from EAS response 158

8.6.7.3.6 Application traffic influence cancellation trigger from EAS request 158

8.6.7.3.7 Application traffic influence cancellation trigger from EAS response 159

8.6.7.4 APIs 159

8.6.7.4.1 General 159

8.6.7.4.2 Eees_TrafficInfluenceEAS_Create operation 159

8.6.7.4.3 Eees_TrafficInfluenceEAS_Update operation 159

8.6.7.4.4 Eees_TrafficInfluenceEAS_Cancellation operation 159

8.7 Network capability exposure to EAS 160

8.7.1 General 160

8.7.2 Direct network capability exposure 160

8.7.3 Network capability exposure via EES 160

8.8 Service continuity 160

8.8.1 General 160

8.8.1.1 High level overview 160

8.8.1.1A UE movement trigger for ACR 162

8.8.1.2 ACR with service continuity planning 163

8.8.1.3 Unused contexts handling during ACR including service continuity planning 163

8.8.1.4 Modification of ACR parameters during ACR for service continuity planning 163

8.8.1.5 Service continuity between CAS and EAS 164

8.8.1.6 Service continuity for EAS bundle 164

8.8.2 Scenarios 164

8.8.2.1 General 164

8.8.2.2 Initiation by EEC using regular EAS Discovery 165

8.8.2.3 EEC executed ACR via S-EES 168

8.8.2.4 S-EAS decided ACR scenario 171

8.8.2.5 S-EES executed ACR 173

8.8.2.6 EEC executed ACR via T-EES 177

8.8.2.7 ACR for direct EAS bundle, executed by EEC 179

8.8.2.8 ACR for EAS bundle, executed by S-EAS 179

8.8.2.9 ACR for EAS bundle, executed by S-EES 181

8.8.2A Scenarios for ACR between EAS and CAS 182

8.8.2A.1 General 182

8.8.2A.2 Enabling ACR with CAS - Initiation by EEC using regular EAS Discovery 182

8.8.2A.3 Enabling ACR with CAS - EEC executed ACR via S-EES 184

8.8.2A.4 Enabling ACR with CAS - S-EAS decided ACR 185

8.8.2A.5 Enabling ACR with CAS - S-EES executed ACR 187

8.8.2A.6 CAS decided ACR scenario via last S-EES 188

8.8.2B Scenarios for ACR between EAS and CAS with CES 190

8.8.2B.1 General 190

8.8.2B.2 ACR from edge to cloud 190

8.8.2B.2.1 General 190

8.8.2B.2.2 Initiation by EEC using regular EAS Discovery 190

8.8.2B.2.3 EEC executed ACR via S-EES 190

8.8.2B.2.4 S-EAS decided ACR 191

8.8.2B.2.5 S-EES executed ACR 191

8.8.2B.3 ACR from cloud to edge 191

8.8.2B.3.1 General 191

8.8.2B.3.2 Initiation by EEC using regular EAS Discovery 191

8.8.2B.3.3 EEC executed ACR via CES 191

8.8.2B.3.4 CAS decided ACR 191

8.8.2B.3.5 CES executed ACR 191

8.8.3 Procedures 192

8.8.3.1 General 192

8.8.3.2 Discover T-EAS 192

8.8.3.3 Retrieve T-EES procedure 194

8.8.3.4 ACR launching procedure 196

8.8.3.5 ACR information subscription 198

8.8.3.5.1 General 198

8.8.3.5.2 Subscribe 198

8.8.3.5.3 Notify 199

8.8.3.5.4 Subscription update 200

8.8.3.5.5 Unsubscribe 200

8.8.3.6 EELManagedACR procedure 201

8.8.3.6.1 General 201

8.8.3.6.2 Procedure 201

8.8.3.6.2.1 General 201

8.8.3.6.2.2 ACR request 201

8.8.3.6.2.3 ACT status subscription 202

8.8.3.6.2.4 ACT status notification 202

8.8.3.7 Selected T-EAS declaration 203

8.8.3.8 ACR status update procedure 204

8.8.3.9 ACR parameter information procedure 204

8.8.3.10 Selected EES declaration 205

8.8.3.11 ACR event subscription and notification procedure 206

8.8.4 Information flows 207

8.8.4.1 General 207

8.8.4.2 EAS discovery request 207

8.8.4.3 EAS discovery response 207

8.8.4.4 ACR request 207

8.8.4.5 ACR response 209

8.8.4.6 Retrieve EES request 209

8.8.4.7 Retrieve EES response 210

8.8.4.8 ACR information subscription request 210

8.8.4.9 ACR information subscription response 211

8.8.4.10 ACR information notification 211

8.8.4.11 ACR information subscription update request 212

8.8.4.12 ACR information subscription update response 213

8.8.4.13 ACR information unsubscribe request 213

8.8.4.14 ACR information unsubscribe response 213

8.8.4.15 EELManagedACR service request 213

8.8.4.16 EELManagedACR service response 214

8.8.4.17 Selected target EAS declaration request 214

8.8.4.18 Selected target EAS declaration response 214

8.8.4.19 ACR status update request 214

8.8.4.20 ACR status update response 215

8.8.4.21 ACT status subscription request 215

8.8.4.22 ACT status subscription response 215

8.8.4.23 ACT status notification 216

8.8.4.24 ACR parameter information request 216

8.8.4.25 ACR parameter information response 216

8.8.4.26 Selected EES declaration request 216

8.8.4.27 Selected EES declaration response 217

8.8.4.28 ACR event subscription request 217

8.8.4.29 ACR event subscription response 217

8.8.4.30 ACR event notification 218

8.8.4.31 ACR event subscription update request 218

8.8.4.32 ACR event subscription update response 218

8.8.4.33 ACR event unsubscribe request 218

8.8.4.34 ACR event unsubscribe response 219

8.8.5 APIs 219

8.8.5.1 General 219

8.8.5.2 Eees_EASDiscovery API 219

8.8.5.2.1 Void 220

8.8.5.2.2 Void 220

8.8.5.3 Eees_AppContextRelocation API 220

8.8.5.3.1 General 220

8.8.5.3.2 Eees_AppContextRelocation_Request operation 220

8.8.5.4 Eecs_TargetEESDiscovery API 220

8.8.5.4.1 General 220

8.8.5.4.2 Eecs_TargetEESDiscovery_Request operation 220

8.8.5.5 Eees_ACREvents API 220

8.8.5.5.1 General 220

8.8.5.5.2 Eees_ACREvents_Subscribe operation 220

8.8.5.5.3 Eees_ACREvents_Notify operation 220

8.8.5.5.4 Eees_ACREvents_UpdateSubscription operation 221

8.8.5.5.5 Eees_ACREvents_Unsubscribe operation 221

8.8.5.6 Eees_EELManagedACR API 221

8.8.5.6.1 General 221

8.8.5.6.2 Eees_EELManagedACR_Request operation 221

8.8.5.6.3 Eees_EELManagedACR_Subscribe operation 221

8.8.5.6.4 Eees_EELManagedACR_Notify operation 222

8.8.5.7 Eees_SelectedTargetEAS API 222

8.8.5.7.1 General 222

8.8.5.7.2 Eees_SelectedTargetEAS_Declare operation 222

8.8.5.8 Eees_ACRStatusUpdate API 222

8.8.5.8.1 General 222

8.8.5.8.2 Eees_ACRStatusUpdate_Request operation 222

8.8.5.9 Eees_ACRParameterInformation API 222

8.8.5.9.1 General 222

8.8.5.9.2 Eees_ACRParameterInformation Request operation 222

8.8.5.10 Ecas_SelectedEES API 223

8.8.5.10.1 General 223

8.8.5.10.2 Ecas_SelectedEES_Declare operation 223

8.8.5.11 Eecs_ACREvents API 223

8.8.5.11.1 General 223

8.8.5.11.2 Eecs_ACREvents_Subscribe operation 223

8.8.5.11.3 Eecs_ACREvents_Notify operation 223

8.8.5.11.4 Eecs_ACREvents_UpdateSubscription operation 223

8.8.5.11.5 Eecs_ACREvents_Unsubscribe operation 224

8.9 EEC Context and EEC Context relocation 224

8.9.1 General 224

8.9.1.1 EEC Context handling at EEC registration 224

8.9.1.2 EEC Context handling at EEC registration update 224

8.9.1.3 EEC Context handling at EEC de-registration 224

8.9.1.4 EEC Context handling at Application Context Relocation 225

8.9.1.5 Other EEC Context handling 225

8.9.2 Procedures 225

8.9.2.1 General 225

8.9.2.2 EEC Context Pull relocation 225

8.9.2.3 EEC Context Push relocation 226

8.9.3 Information flows 227

8.9.3.1 General 227

8.9.3.2 EEC Context Pull request 227

8.9.3.3 EEC Context Pull response 227

8.9.3.4 EEC Context Push request 228

8.9.3.5 EEC Context Push response 228

8.9.4 APIs 228

8.9.4.1 General 228

8.9.4.2 Eees_EECContextPull API 228

8.9.4.2.1 General 228

8.9.4.2.2 Eees_EECContextPull_Request operation 228

8.9.4.3 Eees_EECContextPush API 229

8.9.4.3.1 General 229

8.9.4.3.2 Eees_EECContextPush_Request operation 229

8.10 Utilizing 3GPP core network capabilities 229

8.10.1 General 229

8.10.2 Capabilities utilized by ECS 229

8.10.3 Capabilities utilized by EES and CES 229

8.11 EEC Authentication/Authorization 230

8.11.1 General 230

8.12 Dynamic EAS instantiation triggering 230

8.12.1 General 230

8.13 Charging 231

8.14 EDGE-5 APIs 232

8.14.1 General 232

8.14.2 Procedures 232

8.14.2.1 General 232

8.14.2.2 Registration 232

8.14.2.2.1 General 232

8.14.2.2.2 AC registration 232

8.14.2.2.3 AC registration update 233

8.14.2.2.4 AC deregistration 234

8.14.2.3 EAS discovery 235

8.14.2.4 ACR trigger request 235

8.14.2.5 EEC services subscription 236

8.14.2.5.1 General 236

8.14.2.5.2 Subscribe 236

8.14.2.5.3 EEC services notification 237

8.14.2.5.4 EEC services subscription update 238

8.14.2.5.5 Unsubscribe 239

8.14.2.6 UE ID request 239

8.14.3 Information flows 240

8.14.3.1 General 240

8.14.3.2 AC registration request 240

8.14.3.3 AC registration response 240

8.14.3.4 AC registration update request 241

8.14.3.5 AC registration update response 241

8.14.3.6 AC deregistration request 241

8.14.3.7 AC deregistration response 242

8.14.3.8 EAS discovery request 242

8.14.3.9 EAS discovery response 242

8.14.3.10 ACR trigger request 242

8.14.3.11 ACR trigger response 243

8.14.3.12 EEC services subscription request 243

8.14.3.13 EEC services subscription response 243

8.14.3.14 EEC services notification 244

8.14.3.15 EEC services subscription update request 245

8.14.3.16 EEC services subscription update response 246

8.14.3.17 EEC services unsubscribe request 246

8.14.3.18 EEC services unsubscribe response 247

8.14.3.19 UE ID request 247

8.14.3.20 UE ID response 247

8.14.4 APIs 248

8.14.4.1 General 248

8.14.4.2 Eeec_ACRegistration API 248

8.14.4.2.1 Eeec_ACRegistration_Request operation 248

8.14.4.2.2 Eeec_ACRegistration_Update operation 248

8.14.4.2.3 Eeec_ACRegistration_Deregister operation 249

8.14.4.3 Eeec_EASDiscovery API 249

8.14.4.3.1 Eeec_EASDiscovery_Request operation 249

8.14.4.4 Eeec_ACRTrigger API 249

8.14.4.4.1 Eeec_ACRTrigger_Request operation 249

8.14.4.5 Eeec_Services API 249

8.14.4.5.1 Eeec_Services_Subscribe operation 249

8.14.4.5.2 Eeec_Services_Notify operation 249

8.14.4.5.3 Eeec_Services_UpdateSubscription operation 250

8.14.4.5.4 Eeec_Services_Unsubscribe operation 250

8.14.4.6 Eeec_UEId API 250

8.14.4.6.1 Eeec_UEId_Request operation 250

8.15 EAS Information provisioning 250

8.15.1 General 250

8.15.2 Procedure 251

8.15.2.1 General 251

8.15.2.2 EAS Information provisioning 251

8.15.3 Information flows 253

8.15.3.1 General 253

8.15.3.2 EAS information provisioning request 253

8.15.3.3 EAS information provisioning response 254

8.15.4 APIs 255

8.15.4.1 General 255

8.15.4.2 Eees_EASInformationProvisioning_Declare operation 255

8.16 EEC triggering service to initiate procedures over EDGE-1 or EDGE-4 256

8.16.1 General 256

8.17 Support for roaming and federation 256

8.17.1 General 256

8.17.2 Procedures 257

8.17.2.1 General 257

8.17.2.2 Registration 257

8.17.2.2.1 General 257

8.17.2.2.2 ECS registration 257

8.17.2.2.3 ECS registration update 258

8.17.2.2.4 ECS de-registration 258

8.17.2.3 ECS discovery via ECS-ER 259

8.17.2.3.1 General 259

8.17.2.3.2 Request-response model 259

8.17.2.3.3 Subscribe-notify model 260

8.17.2.3.3.1 General 260

8.17.2.3.3.2 Subscribe 260

8.17.2.3.3.3 Notify 261

8.17.2.3.3.4 Subscription update 262

8.17.2.3.3.5 Unsubscribe 262

8.17.2.4 Service provisioning information retrieval 263

8.17.2.4.1 General 263

8.17.2.4.2 Procedures 263

8.17.2.4.2.1 General 263

8.17.2.4.2.2 Request-response model 263

8.17.2.4.2.3 Subscribe-notify model 264

8.17.2.4.2.3.1 General 264

8.17.2.4.2.3.2 Subscribe 264

8.17.2.4.2.3.3 Notify 265

8.17.2.4.2.3.4 Subscription update 266

8.17.2.4.2.3.5 Unsubscribe 266

8.17.3 Information flows 267

8.17.3.1 General 267

8.17.3.2 ECS registration request 267

8.17.3.3 ECS registration response 268

8.17.3.4 ECS registration update request 268

8.17.3.5 ECS registration update response 268

8.17.3.6 ECS de-registration request 268

8.17.3.7 ECS de-registration response 269

8.17.3.8 ECS discovery request 269

8.17.3.9 ECS discovery response 269

8.17.3.10 ECS discovery subscription request 269

8.17.3.11 ECS discovery subscription response 270

8.17.3.12 ECS discovery notification 270

8.17.3.13 ECS discovery subscription update request 270

8.17.3.14 ECS discovery subscription update response 271

8.17.3.15 ECS discovery unsubscribe request 271

8.17.3.16 ECS discovery unsubscribe response 271

8.17.3.17 Service provisioning information retrieval request 272

8.17.3.18 Service provisioning information retrieval response 272

8.17.3.19 Service provisioning information subscription request 273

8.17.3.20 Service provisioning information subscription response 273

8.17.3.21 Service provisioning information notification 273

8.17.3.22 Service provisioning information subscription update request 274

8.17.3.23 Service provisioning information subscription update response 274

8.17.3.24 Service provisioning information unsubscribe request 274

8.17.3.25 Service provisioning information unsubscribe response 275

8.17.4 APIs 275

8.17.4.1 General 275

8.17.4.2 Eecs_ECSRegistration API 275

8.17.4.2.1 General 275

8.17.4.2.2 Eecs_ECSRegistration_Request operation 275

8.17.4.2.3 Eecs_ECSRegistration_Update operation 276

8.17.4.2.4 Eecs_ECSRegistration_Deregister operation 276

8.17.4.3 Eecs_ECSDiscovery API 276

8.17.4.3.1 General 276

8.17.4.3.2 Eecs_ECSDiscovery_Request operation 276

8.17.4.3.3 Eecs_ECSDiscovery_Subscribe operation 276

8.17.4.3.4 Eecs_ECSDiscovery_Notify operation 276

8.17.4.3.5 Eecs_ECSDiscovery_UpdateSubscription operation 277

8.17.4.3.6 Eecs_ECSDiscovery_Unsubscribe operation 277

8.17.4.4 Eecs_ECSServiceProvisioning API 277

8.17.4.4.1 General 277

8.17.4.4.2 Eecs_ECSServiceProvisioning_Request operation 277

8.17.4.4.3 Eecs_ECSServiceProvisioning_Subscribe operation 277

8.17.4.4.4 Eecs_ECSServiceProvisioning_Notify operation 277

8.17.4.4.5 Eecs_ECSServiceProvisioning_UpdateSubscription operation 278

8.17.4.4.6 Eecs_ECSServiceProvisioning_Unsubscribe operation 278

8.18 Edge Node Sharing 278

8.18.1 General 278

8.18.2 Procedures 278

8.18.2.1 General 278

8.18.2.2 Application information sharing between ECSPs 278

8.18.2.2.1 General 279

8.18.2.3 EAS discovery for ENS 279

8.18.2.3.1 General 279

8.18.2.3.2 EAS discovery via leading ECSP 279

8.18.2.3.3 EAS discovery via Partner ECSP, EEC triggered 280

8.18.2.3.4 EAS discovery via Partner ECSP, EES triggered 281

8.18.2.4 Service continuity for ENS via leading ECSP 281

8.18.2.4.1 General 281

8.18.2.4.2 ACR launching procedure 281

8.18.2.4.3 Selected T-EAS declaration procedure 281

8.18.2.4.4 ACR status update procedure 282

8.18.2.4.5 EAS Information provisioning procedure 282

8.18.2.4.6 ACR management events 282

8.18.2.4.7 EELManagedACR procedure 282

8.18.3 APIs 282

8.18.3.1 General 282

8.19 Common EAS announcement 282

8.19.1 General 282

8.19.2 Procedure 283

8.19.3 Information flows 283

8.19.3.1 General 283

8.19.3.2 Announce common EAS request 283

8.19.3.3 Announce common EAS response 284

8.19.4 APIs 284

8.19.4.1 General 284

8.19.4.2 Eees_CommonEasAnnouncement_Declare operation 284

8.20 Interaction with ECS with Repository function 285

8.20.1 General 285

8.20.2 Procedure 285

8.20.2.1 General 285

8.20.2.2 Obtain EAS information 285

8.20.2.3 Common EAS information storage 285

8.20.2.4 Common EAS information removal 286

8.20.2.5 Common EAS information update 287

8.20.3 Information flows 288

8.20.3.1 General 288

8.20.3.2 EAS information get request 288

8.20.3.3 EAS information get response 288

8.20.3.4 Common EAS information store request 288

8.20.3.5 Common EAS information store response 289

8.20.3.6 Common EAS information remove request 289

8.20.3.7 Common EAS information remove response 290

8.20.3.8 Common EAS information update request 290

8.20.3.9 Common EAS information update response 290

8.20.4 APIs 290

8.20.4.1 General 290

8.20.4.2 Eecs_EASInfoManagement_Store operation 291

8.20.4.3 Eecs_EASInfoManagement_Get operation 291

8.20.4.4 Eecs_EASInfoManagement_Remove operation 291

8.20.4.5 Eecs_EASInfoManagement_Update operation 291

9 Usage of SEAL services 292

9.1 Notification management service 292

9.1.1 General 292

9.1.2 Information flows 292

9.1.3 Procedures 292

10 Edge service reliability support 292

10.1 General 292

11 Edge Enabler Using Satellite Access 293

11.1 General 293

11.1.1 EES determination based on UE serving satellite ID Procedure 293

11.1.2 EES determination based on UE location and route Procedure 293

11.1.2.1 EES profile 294

11.1.2.2 Service provisioning request 294

11.1.2.3 Service provisioning response 294

11.1.3 Application Server Relocation during satellite access 295

11.1.3.1 General 295

11.1.3.2 Updates to General clauses 4.6, 8.6.3 and 8.8.1 295

Annex A (Informative): Deployment models 297

A.1 General 297

A.2 Deployment models for different DN implementations 297

A.2.1 General 297

A.2.2 Option 1. Use of non-dedicated DN 297

A.2.3 Option 2. Use of Edge-dedicated DN 298

A.2.4 Option 3. Use of LADN 298

A.2.5 Option 4. EDN in GEO/MEO/LEO satellite 299

A.3 ECS deployments in relation to the UE 301

A.3.1 General 301

A.3.2 UE (EEC) served by a single ECS 301

A.3.3 UE (EECs) served by multiple ECSs 301

A.4 Deployment of EES in relation with SEAL services and Application Enabler Services 301

A.4.1 General 301

A.4.2 Deployment of SEAL services 303

A.4.3 Deployment of Application Enabler services 303

A.5 Deployments in relation with CAPIF 303

A.5.1 General 303

A.5.2 Distributed CAPIF core functions 303

A.5.3 Centralized CAPIF core function 305

A.5.4 Supporting Exposure of EAS Service APIs using CAPIF 306

Annex B (Informative): Involved entities and relationships 307

B.1 General 307

B.2 Federation and Roaming 308

B.3 Application Groups 309

B.4 Relationships involved in edge computing service with satellite connectivity 310

Annex C (Informative): Relationship with ETSI MEC architecture 310

C.1 Void 311

C.2 Void 311

Annex D (Informative): Relationship with GSMA OPG 311

D.1 Void 311

D.2 Void 311

Annex E (Informative): Support for common EAS 311

E.1 General 311

E.2 Procedure 311

E.2.1 General 311

E.2.2 Common EAS support without repository function 312

E.2.3 Common EAS support with repository function 313

Annex F (informative): Change history 316
