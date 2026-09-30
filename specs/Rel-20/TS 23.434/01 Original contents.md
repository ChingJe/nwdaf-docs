---
spec: TS 23.434
version: 20.1.0
release: '20'
clause: contents
title: Contents
source_archive: 23434-k10.zip
source_document: 23434-k10.docx
content_origin: 3gpp-source
---

# Contents

Foreword 21

Introduction 21

1 Scope 22

2 References 22

3 Definitions, symbols and abbreviations 24

3.1 Definitions 24

3.2 Abbreviations 25

4 Architectural requirements 26

4.1 General 26

4.1.1 Description 26

4.1.2 Requirements 26

4.2 Deployment models 26

4.2.1 Description 26

4.2.2 Requirements 26

4.3 Location management 27

4.3.1 Description 27

4.3.2 On-network functional model requirements 27

4.3.3 Off-network functional model requirements 27

4.4 Group management 27

4.4.1 Description 27

4.4.2 Requirements 27

4.5 Configuration management 28

4.5.1 Description 28

4.5.2 Requirements 28

4.6 Key management 28

4.6.1 Description 28

4.6.2 Requirements 28

4.7 Identity management 28

4.7.1 Description 28

4.7.2 Requirements 28

4.8 Network resource management 28

4.8.1 Description 28

4.8.2 Requirements 28

5 Involved business relationships 29

5.1 Business relationships for VAL services 29

5.2 Business relationships for VAL services with satellite connectivity 30

5.3 Business relationships for VAL services involving SEAL client provider 31

6 Generic functional model for SEAL services 33

6.1 General 33

6.2 On-network functional model description 34

6.3 Off-network functional model description 37

6.4 Functional entities description 37

6.4.1 General 37

6.4.2 Application plane 37

6.4.2.1 General 37

6.4.2.2 VAL client 37

6.4.2.3 VAL server 37

6.4.2.4 SEAL client 38

6.4.2.5 SEAL server 38

6.4.2.6 VAL user database 38

6.4.3 Signalling control plane 38

6.4.3.1 SIP entities 38

6.4.3.1.1 Signalling user agent 38

6.4.3.1.2 SIP AS 38

6.4.3.1.3 SIP core 38

6.4.3.1.3.1 General 38

6.4.3.1.3.2 Local inbound / outbound proxy 39

6.4.3.1.3.3 Registrar finder 39

6.4.3.1.3.4 Registrar / application service selection 39

6.4.3.1.4 Diameter proxy 40

6.4.3.2 SIP database 40

6.4.3.2.1 General 40

6.4.3.2.2 SIP database logical functions 40

6.4.3.3 HTTP entities 41

6.4.3.3.1 HTTP client 41

6.4.3.3.2 HTTP proxy 41

6.4.3.3.3 HTTP server 41

6.4.3.4 LWP entities 42

6.4.3.4.1 LWP client 42

6.4.3.4.2 LWP proxy 42

6.4.3.4.3 LWP server 42

6.4.3.5 LWP usage 42

6.5 Reference points description 42

6.5.1 General reference point principle 42

6.5.2 Application plane 43

6.5.2.1 General 43

6.5.2.2 VAL-UU 43

6.5.2.3 VAL-PC5 43

6.5.2.4 SEAL-UU 43

6.5.2.5 SEAL-PC5 43

6.5.2.6 SEAL-C 43

6.5.2.7 SEAL-S 43

6.5.2.8 SEAL-E 43

6.5.2.9 SEAL-X 44

6.5.2.9.1 General 44

6.5.2.9.2 Reference point SEAL-X1 (between the key management server and the group management server) 44

6.5.2.9.3 Reference point SEAL-X2 (between the group management server and the location management server) 44

6.5.2.9.4 Reference point SEAL-X3 (between the group management server and the configuration management server) 44

6.5.2.10 Reference point VAL-UDB (between the VAL user database and the SEAL server) 44

6.5.3 Signalling control plane 44

6.5.3.1 General 44

6.5.3.2 Reference point SIP-1(between the signalling user agent and the SIP core) 45

6.5.3.3 Reference point SIP-2 (between the SIP core and the SIP AS) 45

6.5.3.4 Reference point SIP-3 (between the SIP core and SIP core) 45

6.5.3.5 Reference point HTTP-1 (between the HTTP client and the HTTP proxy) 45

6.5.3.6 Reference point HTTP-2 (between the HTTP proxy and the HTTP server) 45

6.5.3.7 Reference point HTTP-3 (between the HTTP proxy and HTTP proxy) 46

6.5.3.8 Reference point AAA-1 (between the SIP database and the SIP core) 46

6.5.3.9 Reference point AAA-2 (between the SIP core and Diameter proxy) 46

6.5.3.10 Reference point LWP-1 (between the LWP client and the LWP proxy) 46

6.5.3.11 Reference point LWP-2 (between the LWP proxy and the LWP server) 46

6.5.3.12 Reference point LWP-3 (between the LWP proxy and LWP proxy) 46

6.5.3.13 Reference point LWP-HTTP-2 (between the LWP proxy and the HTTP server) 46

6.5.3.14 Reference point LWP-HTTP-3 (between the LWP proxy and the HTTP proxy) 46

7 Identities 46

7.1 User identity (User ID) 46

7.2 VAL user identity (VAL user ID) 47

7.3 VAL UE identity (VAL UE ID) 47

7.4 VAL service identity (VAL service ID) 47

7.5 VAL group identity (VAL group ID) 47

7.6 VAL system identity (VAL system ID) 47

7.7 VAL Stream ID 47

8 Application of functional model to deployments 47

8.1 General 47

8.2 Deployment of SEAL server(s) 47

8.2.1 SEAL server(s) deployment in PLMN operator domain 48

8.2.2 SEAL server(s) deployment in VAL service provider domain 49

8.2.3 SEAL server(s) deployment outside of VAL service provider domain and PLMN operator domain 51

8.3 Deployment of SEAL client(s) 51

8.3.1 General description 51

8.3.2 Deployment of SEAL Client(s) in the same provider domain of SEAL server(s) 52

8.3.3 Deployment of SEAL Client(s) in UE vendor domain 52

9 Location management 54

9.1 General 54

9.2 Functional model for location management 54

9.2.1 General 54

9.2.2 On-network functional model description 54

9.2.3 Off-network functional model description 56

9.2.4 Functional entities description 57

9.2.4.1 General 57

9.2.4.2 Location management client 57

9.2.4.3 Location management server 57

9.2.5 Reference points description 57

9.2.5.1 General 57

9.2.5.2 LM-UU 58

9.2.5.3 LM-PC5 58

9.2.5.4 LM-C 58

9.2.5.5 LM-S 58

9.2.5.6 LM-E 58

9.2.5.7 T8 58

9.2.5.8 Le 58

9.2.5.9 LM-3P 58

9.2.5.10 N33 58

9.3 Procedures and information flows for Location management (on-network) 59

9.3.1 General 59

9.3.2 Information flows for location information 59

9.3.2.0 Location reporting configuration request 59

9.3.2.1 Location reporting configuration response 59

9.3.2.2 Location information report 60

9.3.2.3 Location information request 60

9.3.2.4 Location reporting trigger 61

9.3.2.5 Location information subscription request 63

9.3.2.6 Location information subscription response 64

9.3.2.7 Location information notification 64

9.3.2.7a Void 65

9.3.2.8 Location reporting configuration cancel request 65

9.3.2.9 Get UE(s) information request 65

9.3.2.10 Get UE(s) information response 66

9.3.2.11 Monitor Location Subscription Request 66

9.3.2.12 Monitor Location Subscription Response 66

9.3.2.12a Monitor location subscription update procedure 67

9.3.2.13 Notify Location Monitoring Event 67

9.3.2.14 Location area monitoring subscription request 68

9.3.2.15 Location area monitoring subscription response 69

9.3.2.16 Location area monitoring notification 69

9.3.2.17 Location area monitoring subscription modify request 70

9.3.2.18 Location area monitoring subscription modify response 71

9.3.2.19 Location area monitoring unsubscribe request 71

9.3.2.20 Location area monitoring unsubscribe response 71

9.3.2.21 VAL service area configuration request 72

9.3.2.22 VAL service area configuration response 72

9.3.2.23 Location service registration request 72

9.3.2.24 Location service registration response 73

9.3.2.25 Location reporting configuration cancel response 73

9.3.2.26 VAL service area obtain request 73

9.3.2.27 VAL service area obtain response 74

9.3.2.28 VAL service area update request 74

9.3.2.29 VAL service area update response 74

9.3.2.30 VAL service area delete request 74

9.3.2.31 VAL service area delete response 75

9.3.2.32 Location information unsubscribe request 75

9.3.2.33 Location information unsubscribe response 75

9.3.2.34 Monitor location unsubscribe request 75

9.3.2.35 Monitor location unsubscribe response 75

9.3.2.36 Location service registration update request 76

9.3.2.37 Location service registration update response 76

9.3.2.38 Location service deregistration request 76

9.3.2.39 Location service deregistration response 76

9.3.2.40 Location reporting configuration update request 77

9.3.2.41 Location reporting configuration update response 77

9.3.2.42 VAL service area subscription request 77

9.3.2.43 VAL service area subscription response 78

9.3.2.44 VAL service area notification 78

9.3.2.45 VAL service area unsubscribe request 78

9.3.2.46 VAL service area unsubscribe response 79

9.3.2.47 Adaptive location reporting configuration provisioning notification request 79

9.3.2.48 Adaptive location reporting configuration provisioning notification response 79

9.3.2.49 Location tracing configuration request 79

9.3.2.50 Location tracing configuration response 80

9.3.2.51 Location history request 80

9.3.2.52 Location history response 80

9.3.2.53 Location tracing configuration cancel request 80

9.3.2.54 Location tracing configuration cancel response 81

9.3.2.55 Surrounding UE retrieval request 81

9.3.2.56 Surrounding UE retrieval response 81

9.3.2.57 Verify location sharing request 81

9.3.2.58 Verify location sharing response 81

9.3.2.59 Location reuse request 82

9.3.2.60 Location reuse response 82

9.3.2.61 SL positioning management subscription request 82

9.3.2.62 SL positioning management subscription response 83

9.3.2.63 SL positioning management notification 83

9.3.2.64 SL positioning configuration command 83

9.3.2.65 Short-Range based positioning information request 84

9.3.2.66 Short-Range based positioning information response 84

9.3.2.67 Location subscription request 85

9.3.2.68 Location subscription response 85

9.3.2.69 Location notification 86

9.3.2.70 Off-network location positioning configuration request 86

9.3.2.71 Off-network location positioning configuration response 86

9.3.2.72 Off-network history location result report 87

9.3.2.73 Confirm location service subscription request 87

9.3.2.74 Confirm location service subscription response 87

9.3.2.75 Confirm location request 87

9.3.2.76 Confirm location report 87

9.3.2.77 Confirm location service usage notification 88

9.3.3 Event-triggered location reporting procedure 88

9.3.3.1 General 88

9.3.3.2 Fetching location reporting configuration 88

9.3.3.3 Location reporting 89

9.3.3.4 Update Location reporting configuration 90

9.3.4 On-demand location reporting procedure 90

9.3.5 Client-triggered or VAL server-triggered location reporting procedure 91

9.3.6 Location reporting triggers configuration cancel 93

9.3.7 Location information subscription procedure 94

9.3.7a Location information subscription update procedure 96

9.3.8 Event-trigger location information notification procedure 96

9.3.9 On-demand usage of location information procedure 98

9.3.10 Obtaining UE(s) information at a location 99

9.3.11 Monitoring Location Deviation 99

9.3.11.1 General 99

9.3.11.2 Monitoring Location Deviation procedure 99

9.3.12 Location area monitoring information procedure 104

9.3.12.1 Location area monitoring subscription procedure 104

9.3.12.2 Location area monitoring subscription modify procedure 104

9.3.12.3 Location area monitoring unsubscribe procedure 105

9.3.12.4 Location area monitoring notification procedure 105

9.3.13 VAL Service Area configuration 106

9.3.13.1 General 106

9.3.13.2 Configure VAL service area identifier procedure 106

9.3.13.3 Obtain VAL service area identifier procedure 107

9.3.13.4 Update VAL service area identifier procedure 107

9.3.13.5 Delete VAL service area identifier procedure 108

9.3.13.6 VAL service area identifier subscribe procedure 109

9.3.13.7 VAL service area identifier notify procedure 109

9.3.13.8 VAL service area identifier unsubscribe procedure 110

9.3.13.9 VAL service area identifier subscription update procedure 110

9.3.14 Location profiling for supporting location service enablement 111

9.3.14.1 Location profiling 111

9.3.14.2 Procedure of Location profiling for location service 111

9.3.15 Location service registration procedure 112

9.3.16 Location information unsubscribe procedure 113

9.3.17 Monitor location unsubscribe procedure 113

9.3.18 Location service registration update procedure 114

9.3.19 Location service deregistration procedure 114

9.3.20 SEAL location management server provides adaptive configuration 115

9.3.21 Location history querying procedure 116

9.3.21.1 Location tracing configuration create procedure 116

9.3.21.2 Location history request procedure 116

9.3.21.3 Location tracing configuration cancel procedure 117

9.3.21.4 Off-network history location result retrieve 118

9.3.21.4.1 Off-network location positioning configuration 118

9.3.21.4.2 Off-network history location result report 118

9.3.22 Surrounding UEs retrieval procedure 119

9.3.23 Optimization of location service for multiple UEs sharing the same location 120

9.3.23.1 General 120

9.3.23.2 Procedure of LM Server identifying the UEs sharing the same location 120

9.3.23.3 Location reuse request 121

9.3.24 Support for sidelink positioning / ranging management 122

9.3.24.1 Procedure of SL positioning /ranging management service 122

9.3.25 Short-Range based positioning information 124

9.3.26 Verify UE location procedure 125

9.3.26.1 General 125

9.3.26.2 Confirm location service subscription 125

9.3.26.3 Confirm location verification 125

9.4 SEAL APIs for location management 126

9.4.1 General 126

9.4.2 SS_LocationReporting API 127

9.4.2.1 General 127

9.4.2.2 Create_Trigger_Location_Reporting operation 127

9.4.2.3 Update_Trigger_Location_Reporting operation 128

9.4.2.4 Cancel_Trigger_Location_Reporting operation 128

9.4.2.5 Notify_Trigger_Location_Reporting operation 128

9.4.2.6 Notify_Adaptive_Configuration operation 128

9.4.3 SS_LocationInfoEvent API 128

9.4.3.1 General 128

9.4.3.2 Subscribe_Location_Info operation 129

9.4.3.3 Notify_Location_Info operation 129

9.4.3.4 Unsubscribe_Location_Info operation 129

9.4.3.5 Update_Location_Info_Subscription 129

9.4.4 SS_LocationInfoRetrieval API 129

9.4.4.1 General 129

9.4.4.2 Obtain_Location_Info operation 130

9.4.5 SS_LocationAreaInfoRetrieval API 130

9.4.5.1 General 130

9.4.5.2 Obtain_UEs_Info operation 130

9.4.6 SS_LocationMonitoring API 130

9.4.6.1 General 130

9.4.6.2 Subscribe_Location_Monitoring operation 130

9.4.6.3 Notify_Location_Monitoring_Events operation 130

9.4.6.4 Unsubscribe_Location\_ Monitoring operation 131

9.4.6.5 Update_Location_Monitoring_Subscription 131

9.4.7 SS_LocationAreaMonitoring API 131

9.4.7.1 General 131

9.4.7.2 Subscribe_Location_Area_Monitoring 131

9.4.7.3 Notify_Location_Area_Monitoring_Events 131

9.4.7.4 Update_Location_Area_Monitoring_Subscribe 132

9.4.7.5 Unsubscribe_Location_Area_Monitoring 132

9.4.8 SS_VALServiceAreaConfiguration API 132

9.4.8.1 General 132

9.4.8.2 Configure_VAL_Service_Area 132

9.4.8.3 Obtain_VAL_Service_Area 132

9.4.8.4 Update_VAL_Service_Area 133

9.4.8.5 Delete_VAL_Service_Area 133

9.4.8.6 Subscribe_VAL_Service_Area_Change_Event 133

9.4.8.7 Notify_VAL_Service_Area_Change_Event 133

9.4.8.8 Unsubscribe_VAL_Service_Area_Change_Event 133

9.4.8.9 Update_Subscription_VAL_Service_Area_Change_Event 134

9.4.9 SS_LocationHistoryInfoEvent API 134

9.4.9.1 General 134

9.4.9.2 Create_Location_Tracing_Configuration operation 134

9.4.9.3 Update_Location_Tracing_Configuration operation 134

9.4.9.4 Cancel_Location_Tracing_Configuration operation 134

9.4.9.5 Obtain_history_Location_Info operation 135

9.4.10 SS_SLPositioningManagement API 135

9.4.10.1 General 135

9.4.10.2 Subscribe\_ SLPositioningManagement 135

9.4.10.3 Notify_SLPositioningManagement 135

9.4.11 SS\_ SRPositioningInformation API 135

9.4.11.1 General 135

9.4.11.2 SR_Positioning_Information operation 136

9.4.12 SS_ConfirmLocation API 136

9.4.12.1 General 136

9.4.12.2 Void 136

9.4.12.3 Confirm Location Service Subscription 136

9.4.12.4 Notify Confirm Location Service 136

9.5 Procedures and information flows for location management (Off-network) 136

9.5.1 General 136

9.5.2 Information flows for off network location management 137

9.5.2.1 Off-network location reporting trigger configuration 137

9.5.2.2 Off-network location reporting trigger configuration response 137

9.5.2.3 Off-network location management ack 137

9.5.2.4 Off-network location report 137

9.5.2.5 Off-network location reporting trigger cancel 138

9.5.2.6 Off-network location reporting trigger cancel response 138

9.5.2.7 Off-network location request 138

9.5.2.8 Off-network location response 138

9.5.3 Event-triggered location reporting procedure 138

9.5.3.1 Location reporting trigger configuration 138

9.5.3.2 Location reporting 139

9.5.3.3 Location reporting trigger cancel 140

9.5.4 On-demand location reporting procedure 141

10 Group management 142

10.1 General 142

10.2 Functional model for group management 142

10.2.1 General 142

10.2.2 On-network functional model description 142

10.2.3 Off-network functional model description 143

10.2.4 Functional entities description 144

10.2.4.1 General 144

10.2.4.2 Group management client 144

10.2.4.3 Group management server 144

10.2.5 Reference points description 144

10.2.5.1 General 144

10.2.5.2 GM-UU 145

10.2.5.3 GM-PC5 145

10.2.5.4 GM-C 145

10.2.5.5 GM-S 145

10.2.5.6 GM-E 145

10.2.5.7 N33 145

10.3 Procedures and information flows for group management 145

10.3.1 General 145

10.3.2 Information flows for group management 146

10.3.2.1 Group creation request 146

10.3.2.2 Group creation response 146

10.3.2.3 Group creation notification 146

10.3.2.4 Group information query request 147

10.3.2.5 Group information query response 147

10.3.2.6 Group membership update request 147

10.3.2.7 Group membership update response 147

10.3.2.8 Group membership notification 148

10.3.2.9 Group deletion request 148

10.3.2.10 Group deletion response 148

10.3.2.11 Group deletion notification 149

10.3.2.12 Void 149

10.3.2.13 Void 149

10.3.2.14 Group information subscribe request 149

10.3.2.15 Group information subscribe response 149

10.3.2.16 Void 149

10.3.2.17 Void 149

10.3.2.18 Store group configuration request 149

10.3.2.19 Store group configuration response 150

10.3.2.20 Get group configuration request 150

10.3.2.21 Get group configuration response 150

10.3.2.22 Subscribe group configuration request 150

10.3.2.23 Subscribe group configuration response 151

10.3.2.24 Notify group configuration request 151

10.3.2.25 Notify group configuration response 151

10.3.2.26 Configure VAL group request 151

10.3.2.27 Configure VAL group response 152

10.3.2.28 Group announcement 152

10.3.2.29 Group registration request 153

10.3.2.30 Group registration response 153

10.3.2.31 Identity list notification 153

10.3.2.32 Group de-registration request 154

10.3.2.33 Group de-registration response 154

10.3.2.34 Location-based group creation request 154

10.3.2.35 Location-based group creation response 155

10.3.2.36 Group list fetch request 155

10.3.2.37 Group list fetch response 155

10.3.2.38 Temporary group formation request 156

10.3.2.39 Temporary group formation response 156

10.3.2.40 Temporary group formation notify 156

10.3.2.41 Temporary group formation notification 156

10.3.2.42 Temporary group formation notification response 157

10.3.2.43 Unsubscribe group configuration request 157

10.3.2.44 Unsubscribe group configuration response 157

10.3.2.45 Group configuration subscription update request 157

10.3.2.46 Group configuration subscription update response 157

10.3.3 Group creation 158

10.3.4 Group information query 159

10.3.4.1 General 159

10.3.4.2 Procedure 159

10.3.5 Group membership 159

10.3.5.0 Group membership subscription 159

10.3.5.1 Group membership notification 160

10.3.5.2 Group membership update by authorized user/UE/VAL server 160

10.3.6 Group configuration management 161

10.3.6.1 Store group configurations at the group management server 161

10.3.6.2 Retrieve group configurations 162

10.3.6.3 Subscription and notification for group configuration data 163

10.3.6.4 Structure of group configuration data 165

10.3.7 Location-based group creation 165

10.3.8 Group announcement and join 166

10.3.8.1 General 166

10.3.8.2 Procedure 166

10.3.9 Group member leave 167

10.3.9.1 General 167

10.3.9.2 Procedure 167

10.3.10 Temporary groups 168

10.3.10.1 Temporary group formation within a VAL system 168

10.3.11 Group List Fetch 170

10.3.12 Location-based group update 170

10.3.13 Group deletion 171

10.4 SEAL APIs for group management 172

10.4.1 General 172

10.4.2 SS_GroupManagement API 172

10.4.2.1 General 172

10.4.2.2 Query_Group_Info operation 172

10.4.2.3 Update_Group_Info operation 172

10.4.2.4 Create_LocationBasedGroup_Info operation 173

10.4.2.5 Create_Group operation 173

10.4.3 Void 173

10.4.3.1 Void 173

10.4.3.2 Void 173

10.4.4 Void 173

10.4.4.1 Void 173

10.4.4.2 Void 173

10.4.5 SS_Group_Management_Event API 173

10.4.5.1 General 173

10.4.5.2 Subscribe\_ Group_Info_Modification operation 173

10.4.5.3 Notify_Group_Info_Modification operation 174

10.4.5.4 Notify_Group_Creation operation 174

10.4.5.5 Notify_TempGroupFormation operation 174

11 Configuration management 174

11.1 General 174

11.2 Functional model for configuration management 175

11.2.1 General 175

11.2.2 On-network functional model description 175

11.2.3 Off-network functional model description 175

11.2.4 Functional entities description 176

11.2.4.1 General 176

11.2.4.2 Configuration management client 176

11.2.4.3 Configuration management server 176

11.2.5 Reference points description 176

11.2.5.1 General 176

11.2.5.2 CM-UU 177

11.2.5.3 CM-PC5 177

11.2.5.4 CM-C 177

11.2.5.5 CM-S 177

11.2.5.6 CM-E 177

11.2.5.7 Reference point CM-VAL-UDB (between the configuration management server and the VAL user database) 177

11.3 Procedures and information flows for configuration management 178

11.3.1 General 178

11.3.2 Information flows 178

11.3.2.1 Get VAL UE configuration request 178

11.3.2.2 Get VAL UE configuration response 178

11.3.2.3 Get VAL user profile request 178

11.3.2.4 Get VAL user profile response 179

11.3.2.5 Notification for VAL user profile data update 179

11.3.2.6 Get updated VAL user profile data request 179

11.3.2.7 Get updated VAL user profile data response 179

11.3.2.8 Update VAL user profile data request 179

11.3.2.9 Update VAL user profile data response 180

11.3.2.10 Updated user profile subscription request 180

11.3.2.11 Updated user profile subscription response 180

11.3.2.12 Updated user profile notification 180

11.3.2.13 Get VAL service request 181

11.3.2.14 Get VAL service response 181

11.3.3 VAL UE configuration data 181

11.3.3.1 General 181

11.3.3.2 Procedures 181

11.3.3.3 Structure of VAL UE configuration data 182

11.3.4 VAL user profile data 182

11.3.4.1 General 182

11.3.4.2 Obtaining the VAL user profile(s) from the network 182

11.3.4.2.1 Obtaining the VAL user profile(s) in primary VAL system 182

11.3.4.2.2 VAL user receiving VAL service from a partner VAL system 183

11.3.4.3 VAL user receives updated VAL user profile data from the network 184

11.3.4.4 VAL user updates VAL user profile data to the network 185

11.3.4.5 Updated user profile subscription procedure 186

11.3.5 VAL service data 187

11.3.5.1 General 187

11.3.5.2 Procedures 187

11.4 SEAL APIs for configuration management 188

11.4.1 General 188

11.4.2 SS_UserProfileRetrieval API 188

11.4.2.1 General 188

11.4.2.2 Obtain_User_Profile operation 188

11.4.3 SS_UserProfileEvent API 188

11.4.3.1 General 188

11.4.3.2 Subscribe_User_Profile_Update operation 188

11.4.3.3 Notify_User_Profile_Update operation 189

11.4.4 SS_VALServiceData API 189

11.4.4.1 General 189

11.4.4.2 Obtain_VAL_Service_Data operation 189

12 Identity management 189

12.1 General 189

12.2 Functional model for identity management 189

12.2.1 General 189

12.2.2 On-network functional model description 190

12.2.3 Off-network functional model description 190

12.2.4 Functional entities description 190

12.2.4.1 General 190

12.2.4.2 Identity management client 191

12.2.4.3 Identity management server 191

12.2.5 Reference points description 191

12.2.5.1 General 191

12.2.5.2 IM-UU 191

12.2.5.3 IM-PC5 191

12.2.5.4 IM-C 191

12.2.5.5 IM-S 191

12.2.5.6 IM-E 191

12.3 Procedures and information flows for identity management 191

12.3.1 General 191

12.3.2 Information flows 191

12.3.2.1 VAL server provisioning request 192

12.3.2.2 VAL server provisioning response 192

12.3.2.3 Update VAL server provisioning request 192

12.3.2.4 Update VAL server provisioning response 192

12.3.2.5 Get VAL server provisioning request 192

12.3.2.6 Get VAL server provisioning response 193

12.3.2.7 Delete VAL server provisioning request 193

12.3.2.8 Delete VAL server provisioning response 193

12.3.3 General user authentication and authorization for VAL services 193

12.3.3.1 General 193

12.3.3.2 Primary VAL system 193

12.3.3.3 Interconnection partner VAL system 194

12.3.4 VAL server provisioning for identity management service 194

12.3.4.1 General 194

12.3.4.2 Procedure 194

12.3.4.3 Update VAL server provisioning procedure 195

12.3.4.4 Get VAL server provisioning procedure 196

12.3.4.5 Delete VAL server provisioning procedure 196

12.4 SEAL APIs for identity management 197

12.4.1 General 197

12.4.2 Void 197

12.4.2.1 Void 197

12.4.2.2 Void 197

12.4.3 SS_IdmParameterProvisioning API 197

12.4.3.1 General 197

12.4.3.2 Provide_Configuration operation 197

12.4.3.3 Update_Configuration operation 197

12.4.3.4 Get_Configuration operation 198

12.4.3.5 Delete_Configuration operation 198

13 Key management 198

13.1 General 198

13.2 Functional model for key management 198

13.2.1 General 198

13.2.2 On-network functional model description 198

13.2.3 Off-network functional model description 199

13.2.4 Functional entities description 199

13.2.4.1 General 199

13.2.4.2 Key management client 199

13.2.4.3 Key management server 200

13.2.5 Reference points description 200

13.2.5.1 General 200

13.2.5.2 KM-UU 200

13.2.5.3 KM-PC5 200

13.2.5.4 KM-C 200

13.2.5.5 KM-S 200

13.2.5.6 KM-E 200

13.2.5.7 SEAL-X1 201

13.3 Procedures and information flows for key management 201

13.3.1 Information flows 201

13.3.1.1 void 201

13.3.1.2 void 201

13.3.2 VAL server provisioning for key management service 201

13.3.2.1 General 201

13.3.2.2 void 201

13.4 SEAL APIs for key management 201

13.4.1 General 201

13.4.2 Void 202

13.4.2.1 Void 202

13.4.2.2 Void 202

13.4.3 void 202

14 Network resource management 202

14.1 General 202

14.2 Functional model for network resource management 202

14.2.1 General 202

14.2.2 On-network functional model description 202

14.2.2.1 Generic on-network functional model for network resource management 202

14.2.2.2 On-network functional model for network resource management for TSN 204

14.2.2.3 On-network functional model for network resource management for 5G TSC 205

14.2.3 Off-network functional model description 206

14.2.4 Functional entities description 206

14.2.4.1 General 206

14.2.4.2 Network resource management client 206

14.2.4.3 Network resource management server 207

14.2.5 Reference points description 207

14.2.5.1 General 207

14.2.5.2 NRM-UU 207

14.2.5.3 NRM-PC5 207

14.2.5.4 NRM-C 207

14.2.5.5 NRM-S 207

14.2.5.6 NRM-E 207

14.2.5.7 MB2-C 207

14.2.5.8 xMB-C 207

14.2.5.9 Rx 207

14.2.5.10 N5 208

14.2.5.11 N33 208

14.2.5.12 Nmb13 208

14.2.5.13 Nmb10 208

14.2.5.14 N6mb 208

14.2.5.15 Nmb8 208

14.2.5.16 N6 208

14.3 Procedures and information flows for network resource management 208

14.3.1 General 208

14.3.2 Information flows 208

14.3.2.1 Network resource adaptation request 208

14.3.2.2 Network resource adaptation response 209

14.3.2.2a Network resource adaptation modification request 209

14.3.2.2b Network resource adaptation modification response 210

14.3.2.3 MBMS bearer announcement 210

14.3.2.4 MBMS listening status report 211

14.3.2.5 MBMS suspension reporting instruction 211

14.3.2.6 Void 212

14.3.2.7 Void 212

14.3.2.8 Void 212

14.3.2.9 Void 212

14.3.2.10 MBMS bearers request 212

14.3.2.11 MBMS bearers response 212

14.3.2.12 User plane delivery mode 213

14.3.2.13 end-to-end QoS management request 213

14.3.2.14 end-to-end QoS management response 214

14.3.2.15 QoS downgrade indication 214

14.3.2.16 Application QoS change notification 214

14.3.2.17 Monitoring Events Subscription Request 215

14.3.2.18 Monitoring Events Subscription Response 215

14.3.2.19 Monitoring Events Notification message 215

14.3.2.20 Unicast QoS monitoring subscription request 216

14.3.2.21 Unicast QoS monitoring subscription response 219

14.3.2.22 Unicast QoS monitoring notification 219

14.3.2.23 TSC stream availability discovery request 220

14.3.2.24 TSC stream availability discovery response 220

14.3.2.25 TSC stream creation request 220

14.3.2.26 TSC stream creation response 220

14.3.2.27 TSC stream deletion request 221

14.3.2.28 TSC stream deletion response 221

14.3.2.29 TSN bridge information report 221

14.3.2.30 TSN bridge information confirmation 221

14.3.2.31 TSN bridge configuration request 221

14.3.2.32 TSN bridge configuration response 221

14.3.2.33 Unicast QoS monitoring data request 221

14.3.2.34 Unicast QoS monitoring data response 222

14.3.2.35 Application connectivity request 223

14.3.2.36 Application connectivity response 223

14.3.2.37 Application connectivity notification 223

14.3.2.38 Unicast QoS monitoring subscription update request 224

14.3.2.39 Unicast QoS monitoring subscription update response 224

14.3.2.40 Multicast/broadcast resource request 224

14.3.2.41 Multicast/broadcast resource response 225

14.3.2.42 Multicast/broadcast resource update request 226

14.3.2.43 Multicast/broadcast resource update response 226

14.3.2.44 Multicast/broadcast resource delete request 227

14.3.2.45 Multicast/broadcast resource delete response 227

14.3.2.46 Multicast resource activate request 227

14.3.2.47 Multicast resource activate response 227

14.3.2.48 Multicast resource deactivate request 227

14.3.2.49 Multicast resource deactivate response 228

14.3.2.50 MapVALGroupToSessionStream 228

14.3.2.51 UE session join notification 228

14.3.2.52 MBS listening status report 228

14.3.2.53 UE unified traffic pattern and monitoring management subscription request 229

14.3.2.54 UE unified traffic pattern and monitoring management subscription response 230

14.3.2.55 UE unified traffic pattern update notification 230

14.3.2.56 Get application connectivity context request. 230

14.3.2.57 Get application connectivity context response. 231

14.3.2.58 BDT configuration request 231

14.3.2.59 BDT configuration response 232

14.3.2.60 BDT negotiation notification 232

14.3.2.61 BDT configuration get request 232

14.3.2.62 BDT configuration get response 232

14.3.2.63 BDT configuration update request 233

14.3.2.64 BDT configuration update response 233

14.3.2.65 BDT configuration delete request 234

14.3.2.66 BDT configuration delete response 234

14.3.2.67 Reliable transmission request 234

14.3.2.68 Reliable transmission response 234

14.3.2.69 Device triggering request 235

14.3.2.70 Device triggering response 235

14.3.2.71 MM-specific QoS management request 235

14.3.2.72 MM-specific QoS management response 236

14.3.2.73 MM service QoS configuration notification 236

14.3.2.74 MMeta service requirements request 237

14.3.2.75 MMeta service requirements response 237

14.3.2.76 MMeta service connectivity request 237

14.3.2.77 MMeta service connectivity response 238

14.3.2.78 MMeta service connectivity notification 238

14.3.2.79 Power saving assist request 239

14.3.2.80 Power saving configuration request 239

14.3.2.81 Power saving configuration response 239

14.3.2.82 Power saving assist response 240

14.3.3 Unicast resource management 240

14.3.3.1 General 240

14.3.3.2 Unicast resource management with SIP core 241

14.3.3.2.1 Request for unicast resources at VAL service communication establishment 241

14.3.3.2.1.1 General 241

14.3.3.2.1.2 Procedure 241

14.3.3.2.2 Request for modification of unicast resources 242

14.3.3.2.2.1 General 242

14.3.3.2.2.2 Procedure 242

14.3.3.3 Unicast resource management without SIP core 243

14.3.3.3.1 Network resource adaptation 243

14.3.3.3.1.1 General 243

14.3.3.3.1.2 Procedure 243

14.3.3.3.2 Request for unicast resources at VAL service communication establishment 244

14.3.3.3.2.1 General 244

14.3.3.3.2.2 Procedure 244

14.3.3.3.3 Request for modification of unicast resources 245

14.3.3.3.3.1 General 245

14.3.3.3.3.2 Procedure 245

14.3.3.4 Unicast QoS monitoring 246

14.3.3.4.1 Unicast QoS monitoring subscription procedure 246

14.3.3.4.1.1 General 246

14.3.3.4.1.2 Procedure 246

14.3.3.4.2 Unicast QoS monitoring notification procedure 247

14.3.3.4.2.1 General 247

14.3.3.4.2.2 Procedure 248

14.3.3.4.3 Unicast QoS monitoring subscription termination procedure 248

14.3.3.4.3.1 General 248

14.3.3.4.3.2 Procedure 248

14.3.3.4.4 Unicast QoS monitoring data retrieval procedure 249

14.3.3.4.4.1 General 249

14.3.3.4.4.2 Procedure 249

14.3.3.4.5 Unicast QoS monitoring subscription update procedure 250

14.3.3.4.5.1 General 250

14.3.3.4.5.2 Procedure 250

14.3.4 Multicast resource management for EPS 250

14.3.4.1 General 250

14.3.4.2 Use of pre-established MBMS bearers 251

14.3.4.2.1 General 251

14.3.4.2.2 Procedure 251

14.3.4.3 Use of dynamic MBMS bearer establishment 253

14.3.4.3.1 General 253

14.3.4.3.2 Procedure 253

14.3.4.4 MBMS bearer announcement over MBMS bearer 255

14.3.4.4.1 General 255

14.3.4.4.2 Procedure 255

14.3.4.5 MBMS bearer quality detection 257

14.3.4.5.1 General 257

14.3.4.5.2 Procedure 257

14.3.4.6 Service continuity in MBMS scenarios 258

14.3.4.6.1 General 258

14.3.4.6.2 Service continuity when moving from one MBSFN to another 258

14.3.4.7 MBMS suspension notification 261

14.3.4.7.1 General 261

14.3.4.7.2 Procedure 261

14.3.4.8 MBMS bearer event notification 262

14.3.4.8.1 General 262

14.3.4.8.2 Procedure 262

14.3.4.9 Switching between MBMS bearer and unicast bearer 263

14.3.4.9.1 General 263

14.3.4.9.2 Procedure 263

14.3.4A Multicast resource management for 5GS 264

14.3.4A.1 General 264

14.3.4A.2 MBS session creation and MBS session announcement 266

14.3.4A.2.1 General 266

14.3.4A.2.2 Procedure for pre-created MBS session and MBS session announcement 266

14.3.4A.2.3 Procedure for dynamic MBS sessions 268

14.3.4A.3 MBS resources update 270

14.3.4A.3.1 General 270

14.3.4A.3.2 Procedure for updating MBS resources without dynamic PCC rule 270

14.3.4A.3.3 Procedure for updating MBS resources with dynamic PCC rule 272

14.3.4A.4 MBS resource deletion 273

14.3.4A.4.1 General 273

14.3.4A.4.2 Procedure 274

14.3.4A.5 Request to activate / de-activate multicast MBS sessions 275

14.3.4A.5.1 General 275

14.3.4A.5.2 Multicast MBS session activation procedure 275

14.3.4A.5.3 Multicast MBS session de-activation procedure 276

14.3.4A.6 VAL service group media transmissions over 5G MBS sessions 277

14.3.4A.6.1 General 277

14.3.4A.6.2 Procedure 278

14.3.4A.7 Aplication level control signalling over 5G MBS sessions 280

14.3.4A.7.1 Description 280

14.3.4A.7.2 Procedure 280

14.3.4A.8 Service continuity between 5G MBS delivery and unicast delivery 281

14.3.4A.8.1 General 281

14.3.4A.8.2 Service continuity for broadcast MBS session 281

14.3.4A.8.2.1 General 281

14.3.4A.8.2.2 Procedures 282

14.3.4A.8.2.2.1 Service continuity from broadcast to unicast 282

14.3.4A.8.2.2.2 Service continuity from unicast to broadcast 283

14.3.4A.8.3 Service continuity for multicast MBS session 285

14.3.4A.9 Service continuity between 5G MBS delivery and unicast delivery 285

14.3.4A.9.1 General 285

14.3.4A.9.2 Service continuity for broadcast MBS session 285

14.3.4A.9.2.1 General 285

14.3.4A.9.2.2 Procedures 285

14.3.4A.9.2.2.1 Service continuity from broadcast to unicast 285

14.3.4A.9.2.2.2 Service continuity from unicast to broadcast 287

14.3.4A.9.3 Service continuity for multicast MBS session 288

14.3.4A.10 VAL service inter-system switching between 5G and LTE 288

14.3.4A.10.1 General 288

14.3.4A.10.2 Inter-system switching from 5G MBS session to LTE eMBMS bearer 289

14.3.4A.10.3 Inter-system switching from 5G MBS session to LTE unicast bearer 290

14.3.4A.10.4 Inter-system switching from LTE eMBMS to 5G MBS session 292

14.3.5 QoS/resource management for network-assisted UE-to-UE/VN group communications 295

14.3.5.1 General 295

14.3.5.2 QoS/resource management capability initiation in network assisted UE-to-UE communications 295

14.3.5.2.1 Procedure for a single pair of UEs 295

14.3.5.2.2 Procedure for a group of UEs 296

14.3.5.3 Procedure for coordinated QoS provisioning operation in network assisted UE-to-UE communications 297

14.3.5.3.1 Procedure for a single pair of UEs 297

14.3.5.3.2 Procedure for a group of UEs 298

14.3.5.4 Application QoS coordination for Mobile Metaverse Services in distributed VAL servers 299

14.3.5.4.1 General 299

14.3.5.4.2 Procedure on application QoS coordination for Mobile Metaverse Services 300

14.3.6 Event Monitoring 301

14.3.6.1 General 301

14.3.6.2 Monitoring Events Subscription Procedure 301

14.3.6.2.1 General 301

14.3.6.2.2 Procedure 301

14.3.6.3 Monitoring Events Notification Procedure 302

14.3.6.3.1 General 302

14.3.6.3.2 Procedure 302

14.3.7 5G TSC resource management procedures 303

14.3.7.1 General 303

14.3.7.2 TSC stream availability discovery procedure 303

14.3.7.3 TSC stream creation procedure 304

14.3.7.4 TSC stream deletion procedure 305

14.3.8 TSN resource management procedures 306

14.3.8.1 General 306

14.3.8.2 5GS TSN Bridge information reporting 306

14.3.8.3 5GS TSN Bridge configuration procedure 307

14.3.9 Establishing communication with application service requirements 308

14.3.9.1 General 308

14.3.9.2 Procedures 308

14.3.9.2.1 Procedure triggered by correlated source and destination requests 308

14.3.9.2.2 Procedure triggered by source request and coordinated with destination 309

14.3.9.2.3 Procedure for establishing communication in Situational Awareness use case 311

14.3.10 AF influence URSP procedure for reliable transmission 312

14.3.10.1 General 312

14.3.10.2 AF influence URSP procedure for reliable transmission 312

14.3.11 VAL services over 5GS supporting EPS interworking 313

14.3.12 UE unified traffic pattern and monitoring management 313

14.3.12.1 General 313

14.3.12.3 UE unified traffic pattern update notification procedure 315

14.3.12.4 Management and 5GC exposure procedures 315

14.3.12.4.1 General 315

14.3.12.4.2 UE unified traffic pattern management procedure 315

14.3.12.4.3 Network parameter coordination procedure 317

14.3.13 Background Data Transfer configuration 318

14.3.13.1 General 318

14.3.13.2 Request and Select Background Data Transfer Policy 318

14.3.13.3 Reselect Background Data Transfer Policy 319

14.3.13.4 BDT configuration get 320

14.3.13.5 BDT configuration update 320

14.3.13.6 BDT configuration delete 321

14.3.14 Device triggering 322

14.3.14.1 General 322

14.3.14.2 Device Triggering via NRM procedure 322

14.3.15 Power saving configuration 323

14.3.15.1 General 323

14.3.15.2 VAL server triggers power saving configuration 324

14.3.15.3 VAL client triggers power saving configuration 325

14.4 SEAL APIs for network resource management 325

14.4.1 General 325

14.4.2 SS_NetworkResourceAdaptation API 326

14.4.2.1 General 326

14.4.2.2 Reserve_Network_Resource operation 326

14.4.2.2a Reserve_Network_Resource_Modify operation 327

14.4.2.3 Void 327

14.4.2.4 Void 327

14.4.2.5 Request_Multicast_Resource 327

14.4.2.6 Notify_UP_Delivery_Mode 327

14.4.2.7 TSC_Stream\_ Availability_Discovery 327

14.4.2.8 TSC_Stream_Creation 328

14.4.2.9 TSC_Stream_Deletion 328

14.4.2.10 Request_Multicast/Broadcast_Resource 328

14.4.2.11 Update_Multicast/Broadcast_Resource 328

14.4.2.12 Delete_Multicast/Broadcast_Resource 328

14.4.2.13 Activate_Multicast_Resource 329

14.4.2.14 Deactivate_Multicast_Resource 329

14.4.2.15 BDT_Configuration_request 329

14.4.2.16 BDT_Negotiation_notification 329

14.4.2.17 Subscribe_Unified_Traffic_Pattern_and_Monitoring_Management operation 329

14.4.2.18 Notify_Unified_Traffic_Pattern_Update operation 330

14.4.2.19 BDT_Configuration_Get_request 330

14.4.2.20 BDT_Configuration_Update_request 330

14.4.2.21 BDT_Configuration_Delete_request 330

14.4.2.22 Reliable_Transmission_request 330

14.4.2.23 Request_Device_Triggering 331

14.4.3 SS_EventsMonitoring API 331

14.4.3.1 Subscribe_Monitoring_Events 331

14.4.3.2 Notify_Monitoring_Events 331

14.4.4 SS_NetworkResourceMonitoring API 331

14.4.4.1 General 331

14.4.4.2 Subscribe_Unicast_QoS_Monitoring operation 331

14.4.4.3 Notify_Unicast_QoS_Monitoring operation 332

14.4.4.4 Unsubscribe_Unicast_QoS_Monitoring operation 332

14.4.4.5 Obtain_Unicast_QoS_Monitoring_Data operation 332

14.4.4.6 Update_Unicast_QoS_Monitoring_Subscription operation 332

14.4.5 SS_MMetaServiceRequirements API 333

14.4.6 SS_ValUeConfiguration API 334

14.4.6.1 General 334

14.4.6.2 Power_Saving_Assist_Request operation 334

15 Service-based interface representation of the functional model for SEAL services 334

15.1 General 334

15.2 Functional model representation 334

15.3 Service-based interfaces 335

16 Network slice capability enablement 335

16.1 General 335

16.2 Functional model 336

16.2.1 General 336

16.2.2 Void 336

16.2.3 Void 336

16.2.4 Void 336

16.3 Procedures and information flows for network slice capability enablement 336

16.4 SEAL APIs for network slice capability enablement 336

17 Notification Management 336

17.1 General 336

17.2 Functional model 336

17.2.1 General 336

17.2.2 Functional model description 336

17.2.3 Functional entities description 337

17.2.3.1 General 337

17.2.3.2 Notification Management client 337

17.2.3.3 Notification Management server 337

17.2.4 Reference points description 337

17.2.4.1 General 337

17.2.4.2 NM-UU 338

17.2.4.3 NM-C 338

17.2.4.4 NM-S 338

17.3 Procedures and information flows for notification management 338

17.3.1 General 338

17.3.2 Information flows for notification management 338

17.3.2.1 Create notification channel request 338

17.3.2.2 Create notification channel response 338

17.3.2.3 Open notification channel 339

17.3.2.4 Notification message 339

17.3.2.5 Delete notification channel request 339

17.3.2.6 Delete notification channel response 340

17.3.2.7 Update notification channel request 340

17.3.2.8 Update notification channel response 340

17.3.3 Procedure for creating notification channel to receive notifications 340

17.3.4 Procedure for deleting notification channel 342

17.3.5 Procedure for updating notification channel 343

18 Data Delivery 344

18.1 General 344

19 Application Data Analytics Enablement 344

19.1 General 344

20 AIML Enablement 344

20.1 General 344

21 SEAL services over Satellite Access 344

21.1 General 344

21.2 Application satellite coverage availability information (ASCAI) configuration 344

21.2.1 General 344

21.2.2 Procedures 345

21.2.2.1 Procedures of the ASCAI provisioning 345

21.2.2.2 Procedures for obtaining the application satellite coverage availability information (ASCAI) 345

21.2.3 Information flows 346

21.2.3.1 Application satellite coverage availability information (ASCAI) configuration request 346

21.2.3.2 Application Satellite coverage availability information (ASCAI) configuration response 347

21.2.3.3 Get application satellite coverage availability information (ASCAI) request 347

21.2.3.4 Get application satellite coverage availability information (ASCAI) response 347

21.2.4 APIs for application satellite coverage availability information (ASCAI) 347

21.2.4.1 General 347

21.2.4.2 SS_ASCAIInfoRetrieval API 348

21.2.4.2.1 General 348

21.2.4.2.2 Obtain_ASCAI_Info operation 348

21.3 Satellite S&F events information 348

21.3.1 General 348

21.3.2 Procedures 348

21.3.2.1 NRM Server exposing the S&F events to the VAL server procedure 348

21.3.2.2 VAL UE reports the UE satellite information procedure 349

21.3.2.3 NRM server provisioning the S&F configuration procedure 350

21.3.3 Information flows 351

21.3.3.1 Satellite S&F events subscription request 351

21.3.3.2 Satellite S&F events subscription response 352

21.3.3.3 Satellite S&F events notification 352

21.3.3.4 Satellite S&F events unsubscribe request 352

21.3.3.5 Satellite S&F events unsubscribe response 352

21.3.3.6 Report UE satellite information request 353

21.3.3.7 Report UE satellite information response 353

21.3.3.8 Satellite S&F configuration request 353

21.3.3.9 Satellite S&F configuration response 354

21.3.4 APIs for Satellite S&F events information 354

21.3.4.1 General 354

21.3.4.2 SS_SatelliteS&FInfoEvent API 354

21.3.4.2.1 General 354

21.3.4.2.2 Subscribe_S&F_Info operation 354

21.3.4.2.3 Unsubscribe_S&F_Info operation 354

21.3.4.2.4 Update\_ S&F \_Info_Subscription operation 355

21.3.4.2.5 Notify\_ S&F \_Info operation 355

21.4.2 Procedure 355

22 Spatial anchors (SAn) service 357

22.1 General 357

23 Spatial map (SM) service 357

23.1 General 357

24 Digital Asset 357

24.1 General 357

Annex A (informative): SEAL integration with 3GPP network exposure systems 357

Annex B (informative): SEAL functional model mapping with Common functional architecture (CFA) 360

Annex C (normative): Protocol realizations of LWP in the signalling control plane 361

C.1 General 361

C.2 Usage of CoAP as LWP 361

Annex D (informative): Exemplary location profile attributes 362

Annex E (informative): Support for Mobile Metaverse use case 362

Annex F (informative): Change history 364
