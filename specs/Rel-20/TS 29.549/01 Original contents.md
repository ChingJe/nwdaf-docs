---
spec: TS 29.549
version: 20.1.0
release: '20'
clause: contents
title: Contents
source_archive: 29549-k10.zip
source_document: '29549-k10_0_cover.docx, 29549-k10_1_Main-Body_s00_s06.docx, 29549-k10_2_Main-Body_s07_s09.docx, 29549-k10_3_Annexes_sA_sHistory.docx'
content_origin: 3gpp-source
---

# Contents

Foreword 23

1 Scope 25

2 References 25

3 Definitions of terms and abbreviations 27

3.1 Terms 27

3.2 Abbreviations 27

4 Overview 28

5 Services offered by the SEAL servers 28

5.1 Introduction of SEAL services 28

5.2 Location management APIs 37

5.2.1 SS_LocationReporting API 37

5.2.1.1 Service Description 37

5.2.1.1.1 Overview 37

5.2.1.2 Service Operations 37

5.2.1.2.1 Introduction 37

5.2.1.2.2 Create_Trigger_Location_Reporting 38

5.2.1.2.3 Fetch_Location_Report_Trigger 38

5.2.1.2.4 Update_Trigger_Location_Reporting 39

5.2.1.2.5 Cancel_Trigger_Location_Reporting 39

5.2.1.2.6 Notify_Trigger_Location_Reporting 40

5.2.1.2.7 Notify_Adaptive_Configuration 40

5.2.2 SS_LocationInfoEvent API 41

5.2.3 SS_LocationInfoRetrieval API 41

5.2.4 SS_LocationAreaInfoRetrieval API 41

5.2.4.1 Service Description 41

5.2.4.1.1 Overview 41

5.2.4.2 Service Operations 41

5.2.4.2.1 Introduction 41

5.2.4.2.2 Obtain_UEs_Info 41

5.2.5 SS_LocationMonitoring API 42

5.2.6 SS_LocationAreaMonitoring API 43

5.2.7 SS_VALServiceAreaConfiguration API 43

5.2.7.1 Service Description 43

5.2.7.1.1 Overview 43

5.2.7.2 Service Operations 43

5.2.7.2.1 Introduction 43

5.2.7.2.2 Configure_VAL_Service_Area 43

5.2.7.2.3 Obtain_VAL_Service_Area 44

5.2.7.2.4 Update_VAL_Service_Area 44

5.2.7.2.5 Delete_VAL_Service_Area 45

5.2.7.2.6 Subscribe_VAL_Service_Area_Change_Event 45

5.2.7.2.7 Update_Subscription_VAL_Service_Area_Change_Event 46

5.2.7.2.8 Unsubscribe_VAL_Service_Area_Change_Event 46

5.2.7.2.9 Notify_VAL_Service_Area_Change_Event 47

5.2.8 SS_LocationHistoryInfoEvent Service 47

5.2.8.1 Service Description 47

5.2.8.2 Service Operations 47

5.2.8.2.1 Introduction 47

5.2.8.2.2 SS_LocationHistoryInfoEvent_Create 47

5.2.8.2.3 SS_LocationHistoryInfoEvent_Update 48

5.2.8.2.4 SS_LocationHistoryInfoEvent_Delete 49

5.2.8.2.5 SS_LocationHistoryInfoEvent_Query 49

5.2.9 SS_ConfirmLocation Service 49

5.2.9.1 Service Description 49

5.2.9.2 Service Operations 50

5.2.9.2.1 Introduction 50

5.2.9.2.2 SS_ConfirmLocation_Subscribe 50

5.2.9.2.3 SS_ConfirmLocation_Notify 51

5.2.10 SS_SLPositioningManagement API 52

5.2.10.1 Service Description 52

5.2.10.2 Service Operations 52

5.2.10.2.1 Introduction 52

5.2.10.2.2 SS_SLPositioningManagement_Subscribe 52

5.2.10.2.3 SS_SLPositioningManagement_Notify 53

5.2.10.2.4 SS_SLPositioningManagement_SRInfoRequest 54

5.3 Group management APIs 54

5.3.1 SS_GroupManagement API 54

5.3.1.1 Service Description 54

5.3.1.1.1 Overview 54

5.3.1.2 Service Operations 54

5.3.1.2.1 Introduction 54

5.3.1.2.2 Query_Group_Info 55

5.3.1.2.3 Update_Group_Info 56

5.3.1.2.4 Create_Group 56

5.3.1.2.5 Delete_Group 57

5.3.2 SS_GroupManagementEvent API 57

5.4 Configuration management APIs 58

5.4.1 SS_UserProfileRetrieval API 58

5.4.1.1 Service Description 58

5.4.1.1.1 Overview 58

5.4.1.2 Service Operations 58

5.4.1.2.1 Introduction 58

5.4.1.2.2 Obtain_User_Profile 58

5.4.2 SS_UserProfileEvent API 59

5.4.3 SS_VALServiceData API 59

5.4.3.1 Service Description 59

5.4.3.1.1 Overview 59

5.4.3.2 Service Operations 59

5.4.3.2.1 Introduction 59

5.4.3.2.2 Obtain_VAL_Service_Data 59

5.4.4 SS_ASCAIInfoRetrieval API 60

5.4.4.1 Service Description 60

5.4.4.1.1 Overview 60

5.4.4.2 Service Operations 60

5.4.4.2.1 Introduction 60

5.4.4.2.2 Obtain_ASCAI_Info 60

5.5 Network resource management APIs 60

5.5.1 SS_NetworkResourceAdaptation API 60

5.5.1.1 Service Description 60

5.5.1.1.1 Overview 60

5.5.1.2 Service Operations 61

5.5.1.2.1 Introduction 61

5.5.1.2.2 Void 62

5.5.1.2.3 Reserve_Network_Resource 62

5.5.1.2.4 Reserve_Network_Resource_Modify 62

5.5.1.2.5 Request_Multicast_Resource 62

5.5.1.2.6 Notify_UP_Delivery_Mode 63

5.5.1.2.7 Create_TSC_Stream 63

5.5.1.2.8 Delete_TSC_Stream 64

5.5.1.2.9 Discover_TSC_Stream_Availability 65

5.5.1.2.10 Create_MBS_Resource 65

5.5.1.2.11 Update_MBS_Resource 66

5.5.1.2.12 Delete_MBS_Resource 66

5.5.1.2.13 Activate_MBS_Resource 67

5.5.1.2.14 Deactivate_MBS_Resource 67

5.5.1.2.15 BDT_Configuration_Request 68

5.5.1.2.16 BDT_Negotiation_Notification 68

5.5.1.2.17 BDT_Configuration_Get 68

5.5.1.2.18 BDT_Configuration_Update 69

5.5.1.2.19 BDT_Configuration_Delete 69

5.5.1.2.20 Subscribe_Unified_Traffic_Pattern_and_Monitoring_Management 70

5.5.1.2.21 Update_Unified_Traffic_Pattern_and_Monitoring_Management_Subscription 70

5.5.1.2.22 Unsubscribe_Unified_Traffic_Pattern_and_Monitoring_Management 71

5.5.1.2.23 Notify_Unified_Traffic_Pattern_Update 71

5.5.1.2.24 Reliable_Transmission_Request 72

5.5.2 SS_EventsMonitoring API 72

5.5.3 SS_NetworkResourceMonitoring API 72

5.5.3.1 Service Description 72

5.5.3.1.1 Overview 72

5.5.3.2 Service Operations 72

5.5.3.2.1 Introduction 72

5.5.3.2.2 Subscribe_Unicast_QoS_Monitoring 73

5.5.3.2.3 Unsubscribe_Unicast_QoS_Monitoring 74

5.5.3.2.4 Notify_Unicast_QoS_Monitoring 74

5.5.3.2.5 Obtain_Unicast_QoS_Monitoring_Data 75

5.5.3.2.6 Update_Unicast_QoS_Monitoring_Subscription 75

5.5.4 SS_SatelliteSFInfoEvent API 76

5.5.5 SS_ValUeConfiguration API 76

5.5.5.2 Service Operations 76

5.5.5.2.2 SS_ValUeConfiguration_PowerSavingAssist 76

5.5.6 SS_MMetaConnectivityRequirements Service 77

5.5.6.1 Service Description 77

5.5.6.2 Service Operations 77

5.5.6.2.1 Introduction 77

5.5.6.2.2 Subscribe 77

5.5.6.2.3 Notify 79

5.6 Events APIs 80

5.6.1 SS_Events API 80

5.6.1.1 Service Description 80

5.6.1.1.1 Overview 80

5.6.1.2 Service Operations 80

5.6.1.2.1 Introduction 80

5.6.1.2.2 Subscribe_Event 81

5.6.1.2.3 Notify_Event 81

5.6.1.2.4 Unsubscribe_Event 82

5.6.1.2.5 Update_Subscription 82

5.7 Key management APIs 83

5.7.1 SS_KeyInfoRetrieval API 83

5.7.1.1 Service Description 83

5.7.1.1.1 Overview 83

5.7.1.2 Service Operations 83

5.7.1.2.1 Introduction 83

5.7.1.2.2 Obtain_Key_Info 83

5.7.2 SS_KMParametersProvisioning API 84

5.7.2.1 Service Description 84

5.7.2.1.1 Overview 84

5.7.2.2 Service Operations 84

5.7.2.2.1 Introduction 84

5.7.2.2.2 Provide_Configuration 84

5.8 Network Slice Capability Enablement APIs 84

5.9 Identity Management APIs 85

5.9.1 SS_IdmParameterProvisioning API 85

5.9.1.1 Service Description 85

5.9.1.1.1 Overview 85

5.9.1.2 Service Operations 85

5.9.1.2.1 Introduction 85

5.9.1.2.2 Provide_Configuration 85

5.9.1.2.3 Get_Configuration 86

5.9.1.2.4 Update_Configuration 86

5.9.1.2.5 Delete_Configuration 87

5.10 Data Delivery APIs 87

5.11 Application data analytics enablement service configuration APIs 87

5.11.1 SS_ADAE_VALPerformanceAnalytics API 87

5.11.1.1 Service Description 87

5.11.1.1.1 Overview 87

5.11.1.2 Service Operations 87

5.11.1.2.1 Introduction 87

5.11.1.2.2 Subscribe_VAL_Performance_Analytics 88

5.11.1.2.3 Notify_VAL_Performance_Analytics 88

5.11.1.2.4 Unsubscribe_VAL_Performance_Analytics 89

5.11.2 SS_ADAE_SlicePerformanceAnalytics API 89

5.11.2.1 Service Description 89

5.11.2.1.1 Overview 89

5.11.2.2 Service Operations 89

5.11.2.2.1 Introduction 89

5.11.2.2.2 Subscribe_Slice_Performance_Analytics 90

5.11.2.2.3 Notify_Slice_Performance_Analytics 90

5.11.2.2.4 Unsubscribe_Slice_Performance_Analytics 91

5.11.3 SS_ADAE_Ue2UePerformanceAnalytics API 91

5.11.3.1 Service Description 91

5.11.3.1.1 Overview 91

5.11.3.2 Service Operations 91

5.11.3.2.1 Introduction 91

5.11.3.2.2 UE-to-UE_Performance_Analytics_Subscribe 92

5.11.3.2.3 UE-to-UE_Performance_Analytics_Notify 92

5.11.3.2.4 UE-to-UE_Performance_Analytics_Unsubscribe 93

5.11.4 SS_ADAE_LocationAccuracyAnalytics API 93

5.11.4.1 Service Description 93

5.11.4.1.1 Overview 93

5.11.4.2 Service Operations 93

5.11.4.2.1 Introduction 93

5.11.4.2.2 Subscribe_Location_Accuracy_Analytics 94

5.11.4.2.3 Notify_Location_Accuracy_Analytics 94

5.11.4.2.4 Unsubscribe_Location_Accuracy_Analytics 94

5.11.5 SS_ADAE_ServiceApiAnalytics API 95

5.11.5.1 Service Description 95

5.11.5.1.1 Overview 95

5.11.5.2 Service Operations 95

5.11.5.2.1 Introduction 95

5.11.5.2.2 Subscribe_Service_API_Analytics 95

5.11.5.2.3 Notify_Service_API_Analytics 96

5.11.5.2.4 Unsubscribe_Service_API_Analytics 96

5.11.6 SS_ADAE_SliceUsagePatternAnalytics API 97

5.11.6.1 Service Description 97

5.11.6.1.1 Overview 97

5.11.6.2 Service Operations 97

5.11.6.2.1 Introduction 97

5.11.6.2.2 Subscribe_Slice_Usage_Pattern_Analytics 97

5.11.6.2.3 Notify_Slice_Usage_Pattern_Analytics 98

5.11.6.2.4 Unsubscribe_Slice_Usage_Pattern_Analytics 98

5.11.6.2.5 Get_Slice_Usage_Stats 98

5.11.7 SS_ADAE_EdgeLoadAnalytics API 99

5.11.7.1 Service Description 99

5.11.7.1.1 Overview 99

5.11.7.2 Service Operations 99

5.11.7.2.1 Introduction 99

5.11.7.2.2 Subscribe_Edge_Load 99

5.11.7.2.3 Notify_Edge_Load 100

5.11.7.2.4 Unsubscribe_Edge_Load 100

5.11.7.2.5 Get_Edge_Load_Data 100

5.11.8 SS_AADRF_DataManagement API 101

5.11.8.1 Service Description 101

5.11.8.2 Service Operations 101

5.11.8.2.2 SS_AADRF_DataManagement_Subscribe service operation 101

5.11.8.2.3 SS_AADRF_DataManagement_Notify service operation 102

5.11.8.2.4 SS_AADRF_DataManagement_Unsubscribe 102

5.11.8.2.5 Data_Storage service operation 102

5.11.8.2.6 Data_Removal service operation 103

5.11.9 SS_ADAE_LocationRelatedUeGroupAnalytics API 103

5.11.9.1 Service Description 103

5.11.9.2 Service Operations 103

5.11.9.2.1 Introduction 103

5.11.9.2.2 SS_ADAE_LocationRelatedUeGroupAnalytics_Subscribe 104

5.11.9.2.3 SS_ADAE_LocationRelatedUeGroupAnalytics_Notify 105

5.11.10 SS_ADAE_CollisionDetectionAnalytics API 105

5.11.10.1 Service Description 105

5.11.10.2 Service Operations 106

5.11.10.2.1 Introduction 106

5.11.10.2.2 Subscribe 106

5.11.10.2.3 Notify 107

5.11.11 SS_ADAE_AIMLMemberCapabilityAnalytics API 108

5.11.11.1 Service Description 108

5.11.11.2 Service Operations 108

5.11.11.2.1 Introduction 108

5.11.11.2.2 SS_ADAE_AIML_MemberCapabilityAnalytics_Subscribe 108

5.11.11.2.3 SS_ADAE_AIML_MemberCapabilityAnalytics_Unsubscribe 109

5.11.11.2.4 SS_ADAE_AIML_MemberCapabilityAnalytics_Notify 110

5.11.11.2.5 SS_ADAE_AIML_MemberCapabilityAnalytics_Update 111

5.11.12 SS_ADCCF_DataCollection API 112

5.11.12.1 Service Description 112

5.11.12.2 Service Operations 112

5.11.12.2.1 Introduction 112

5.11.12.2.2 Subscribe 112

5.11.12.2.3 Notify 113

5.11.13 SS_ADAE_ServerToServerPerformanceAnalytics API 113

5.11.13.1 Service Description 113

5.11.13.2 Service Operations 113

5.11.13.2.1 Introduction 113

5.11.13.2.2 Subscribe 114

5.11.13.2.3 Notify 115

5.11.14 SS_ADAE_UeRatConnectivityAnalytics API 115

5.11.14.1 Service Description 115

5.11.14.2 Service Operations 116

5.11.14.2.1 Introduction 116

5.11.14.2.2 SS_ADAE_UeRatConnectivityAnalytics_Subscribe 116

5.11.14.2.3 SS_ADAE_UeRatConnectivityAnalytics_Update 117

5.11.14.2.4 SS_ADAE_UeRatConnectivityAnalytics_Unsubscribe 118

5.11.14.2.5 SS_ADAE_UeRatConnectivityAnalytics_Notify 118

5.11.15 SS_ADAE_DN_energy_analytics Service 119

5.11.15.1 Service Description 119

5.11.15.2 Service Operations 119

5.11.15.2.1 Introduction 119

5.11.15.2.2 SS_ADAE_DN_energy_analytics_Request 119

5.12 AIML Enablement APIs 123

5.13 Metaverse Enablement APIs 123

5.14 Digital Asset APIs 124

5.14.1 SS_DAProfileManagement API 124

5.14.1.1 Service Description 124

5.14.1.2 Service Operations 124

5.14.1.2.1 Introduction 124

5.14.1.2.2 SS_DAProfileManagement_Create 124

5.14.1.2.3 SS_DAProfileManagement_Retrieve 125

5.14.1.2.4 SS_DAProfileManagement_Update 125

5.14.1.2.5 SS_DAProfileManagement_Delete 126

5.14.2 SS_DADiscovery API 126

5.14.2.1 Service Description 126

5.14.2.2 Service Operations 126

5.14.2.2.1 Introduction 126

5.14.2.2.2 Discovery 126

5.14.3 SS_DAMediaManagement API 127

5.14.3.1 Service Description 127

5.14.3.2 Service Operations 127

5.14.3.2.1 Introduction 127

5.14.3.2.2 Upload 127

5.14.3.2.3 Download 128

5.14.3.2.4 Update 128

5.14.3.2.5 Delete 129

5.14.4 SS_DAUsageReport API 129

5.14.4.1 Service Description 129

5.14.4.2 Service Operations 129

5.14.4.2.2 SS_DAUsageReport_Create 130

5.14.4.2.3 SS_DAUsageReport_Update 131

5.14.4.2.4 SS_DAUsageReport_Delete 132

5.14.4.2.5 SS_DAUsageReport_Notify 133

6 SEAL Design Aspects Common for All APIs 133

6.1 General 133

6.2 Data Types 134

6.2.1 General 134

6.2.2 Referenced structured data types 134

6.2.3 Referenced Simple data types and enumerations 134

6.3 Usage of HTTP 134

6.4 Content type 135

6.5 URI structure 135

6.5.1 Resource URI structure 135

6.5.2 Custom operations URI structure 135

6.6 Notifications 135

6.7 Error Handling 136

6.8 Feature negotiation 136

6.9 HTTP headers 136

6.10 Conventions for Open API specification files 136

7 SEAL API Definitions 137

7.0 General 137

7.1 Location management APIs 137

7.1.1 SS_LocationReporting API 137

7.1.1.1 API URI 137

7.1.1.2 Resources 137

7.1.1.2.1 Overview 137

7.1.1.2.2 Resource: SEAL Location Reporting Configurations 138

7.1.1.2.3 Resource: Individual SEAL Location Reporting Configuration 139

7.1.1.3 Notifications 143

7.1.1.3.1 General 143

7.1.1.3.2 Location Trigger Event Notification 144

7.1.1.3.3 Adaptive Location Configuration Notification 145

7.1.1.4 Data Model 146

7.1.1.4.1 General 146

7.1.1.4.2 Structured data types 148

7.1.1.4.3 Simple data types and enumerations 150

7.1.1.5 Error Handling 152

7.1.1.5.1 General 152

7.1.1.5.2 Protocol Errors 152

7.1.1.5.3 Application Errors 152

7.1.2 SS_LocationAreaInfoRetrieval API 153

7.1.2.1 API URI 153

7.1.2.2 Resources 153

7.1.2.2.1 Overview 153

7.1.2.2.2 Resource: Location Information 153

7.1.2.3 Notifications 155

7.1.2.4 Data Model 155

7.1.2.4.1 General 155

7.1.2.4.2 Structured Data Types 155

7.1.2.4.3 Simple data types and enumerations 155

7.1.2.5 Error Handling 156

7.1.2.5.1 General 156

7.1.2.5.2 Protocol Errors 156

7.1.2.5.3 Application Errors 156

7.1.2.6 Feature Negotiation 156

7.1.3 SS_VALServiceAreaConfiguration API 156

7.1.3.1 API URI 156

7.1.3.1A Usage of HTTP 157

7.1.3.2 Resources 157

7.1.3.2.1 Overview 157

7.1.3.2.2 Resource: VAL Service Areas 158

7.1.3.2.3 Resource: VAL Service Area Change Subscriptions 162

7.1.3.2.4 Resource: Individual VAL Service Area Change Subscription 164

7.1.3.3 Notifications 168

7.1.3.3.1 General 168

7.1.3.4 Data Model 170

7.1.3.4.1 General 170

7.1.3.4.2 Structured data types 171

7.1.3.4.3 Simple data types and enumerations 173

7.1.3.5 Error Handling 173

7.1.3.5.1 General 173

7.1.3.5.2 Protocol Errors 173

7.1.3.5.3 Application Errors 173

7.1.3.6 Feature negotiation 174

7.1.4 SS_LocationHistoryInfoEvent API 174

7.1.4.1 Introduction 174

7.1.4.2 Usage of HTTP and common API related aspects 174

7.1.4.3 Resources 174

7.1.4.3.1 Overview 174

7.1.4.3.2 Resource: Location Tracing Configurations 175

7.1.4.3.3 Resource: Individual Location Tracing Configuration 176

7.1.4.3.4 Resource: Location History Reports 181

7.1.4.4 Custom Operations without associated resources 183

7.1.4.5 Notifications 183

7.1.4.6 Data Model 183

7.1.4.6.1 General 183

7.1.4.6.2 Structured data types 184

7.1.4.6.3 Simple data types and enumerations 186

7.1.4.6.4 Data types describing alternative data types or combinations of data types 186

7.1.4.6.5 Binary data 186

7.1.4.7 Error Handling 186

7.1.4.7.1 General 186

7.1.4.7.2 Protocol Errors 186

7.1.4.7.3 Application Errors 187

7.1.4.8 Feature negotiation 187

7.1.4.9 Security 187

7.1.5 SS_ConfirmLocation Service API 187

7.1.5.1 Introduction 187

7.1.5.2 Usage of HTTP and common API related aspects 187

7.1.5.3 Resources 187

7.1.5.3.1 Overview 187

7.1.5.3.2 Resource: Location Confirmation Service Subscriptions 188

7.1.5.3.3 Resource: Individual Location Confirmation Service Subscription 189

7.1.5.4 Custom Operations without associated resources 194

7.1.5.5 Notifications 194

7.1.5.5.1 General 194

7.1.5.5.2 Location Confirmation Service Usage Notification 195

7.1.5.6 Data Model 196

7.1.5.6.1 General 196

7.1.5.6.2 Structured data types 197

7.1.5.6.3 Simple data types and enumerations 198

7.1.5.6.4 Data types describing alternative data types or combinations of data types 199

7.1.5.6.5 Binary data 199

7.1.5.7 Error Handling 199

7.1.5.7.1 General 199

7.1.5.7.2 Protocol Errors 199

7.1.5.7.3 Application Errors 199

7.1.5.8 Feature negotiation 199

7.1.5.9 Security 199

7.1.6 SS_SLPositioningManagement Service API 200

7.1.6.1 Introduction 200

7.1.6.2 Usage of HTTP and common API related aspects 200

7.1.6.3 Resources 200

7.1.6.3.1 Overview 200

7.1.6.3.2 Resource: SL Positioning Management Subscriptions 201

7.1.6.3.3 Resource: Individual SL Positioning Management Subscription 202

7.1.6.4 Custom Operations without associated resources 207

7.1.6.4.1 Overview 207

7.1.6.4.2 Operation: SR Positioning Information Request 207

7.1.6.5 Notifications 208

7.1.6.5.1 General 208

7.1.6.5.2 SL Positioning Management Event Notification 209

7.1.6.6 Data Model 210

7.1.6.6.1 General 210

7.1.6.6.2 Structured data types 211

7.1.6.6.3 Simple data types and enumerations 215

7.1.6.6.4 Data types describing alternative data types or combinations of data types 216

7.1.6.6.5 Binary data 216

7.1.6.7 Error Handling 216

7.1.6.7.1 General 216

7.1.6.7.2 Protocol Errors 216

7.1.6.7.3 Application Errors 217

7.1.6.8 Feature negotiation 217

7.1.6.9 Security 217

7.2 Group management APIs 217

7.2.1 SS_GroupManagement API 217

7.2.1.1 API URI 217

7.2.1.2 Resources 217

7.2.1.2.1 Overview 217

7.2.1.2.2 Resource: VAL Group Documents 218

7.2.1.2.3 Resource: Individual VAL Group Document 221

7.2.1.3 Notifications 225

7.2.1.4 Data Model 225

7.2.1.4.1 General 225

7.2.1.4.2 Structured data types 227

7.2.1.4.3 Simple data types and enumerations 228

7.2.1.5 Error Handling 228

7.2.1.5.1 General 228

7.2.1.5.2 Protocol Errors 228

7.2.1.5.3 Application Errors 228

7.2.1.6 Feature negotiation 229

7.3 Configuration management APIs 229

7.3.1 SS_UserProfileRetrieval API 229

7.3.1.1 API URI 229

7.3.1.2 Resources 229

7.3.1.2.1 Overview 229

7.3.1.2.2 Resource: VAL Services 230

7.3.1.3 Notifications 231

7.3.1.4 Data Model 231

7.3.1.4.1 General 231

7.3.1.4.2 Structured data types 232

7.3.1.4.3 Simple data types and enumerations 232

7.3.1.5 Error Handling 232

7.3.1.5.1 General 232

7.3.1.5.2 Protocol Errors 233

7.3.1.5.3 Application Errors 233

7.3.1.6 Feature negotiation 233

7.3.2 SS_VALServiceData API 233

7.3.2.1 API URI 233

7.3.2.1A Usage of HTTP 233

7.3.2.2 Resources 233

7.3.2.2.1 Overview 233

7.3.2.2.2 Resource: VAL Service Data Sets 234

7.3.2.3 Custom Operations without associated resources 236

7.3.2.4 Notifications 236

7.3.2.5 Data Model 236

7.3.2.5.1 General 236

7.3.2.5.2 Structured data types 236

7.3.2.5.3 Simple data types and enumerations 237

7.3.2.5.4 Data types describing alternative data types or combinations of data types 237

7.3.2.5.5 Binary data 237

7.3.2.6 Error Handling 237

7.3.2.6.1 General 237

7.3.2.6.2 Protocol Errors 237

7.3.2.6.3 Application Errors 238

7.3.2.7 Feature negotiation 238

7.3.3 SS_ASCAIInfoRetrieval API 238

7.3.3.1 API URI 238

7.3.3.2 Resources 238

7.3.3.3 Custom Operations without associated resources 238

7.3.3.3.1 Overview 238

7.3.3.3.2 Operation: RequestASCAIInfo 239

7.3.3.4 Notifications 240

7.3.3.5 Data Model 240

7.3.3.5.1 General 240

7.3.3.5.2 Structured data types 241

7.3.3.5.3 Simple data types and enumerations 242

7.3.3.5.4 Data types describing alternative data types or combinations of data types 243

7.3.3.5.5 Binary data 243

7.3.3.6 Error Handling 243

7.3.3.6.1 General 243

7.3.3.6.2 Protocol Errors 243

7.3.3.6.3 Application Errors 243

7.3.3.7 Feature negotiation 243

7.4 Network resource management APIs 244

7.4.1 SS_NetworkResourceAdaptation API 244

7.4.1.1 API URI 244

7.4.1.1A Usage of HTTP 244

7.4.1.2 Resources 244

7.4.1.2.1 Overview 244

7.4.1.2.2 Resource: Multicast Subscriptions 249

7.4.1.2.3 Resource: Individual Multicast Subscription 250

7.4.1.2.4 Resource: Unicast Subscriptions 252

7.4.1.2.5 Resource: Individual Unicast Subscription 253

7.4.1.2.6 Resource: TSC Stream Availability 258

7.4.1.2.7 Resource: TSC streams 259

7.4.1.2.8 Resource: Individual TSC Stream 261

7.4.1.2.9 Resource: MBS Resources 264

7.4.1.2.10 Resource: Individual MBS Resource 265

7.4.1.2.11 Resource: BDT Policy Configurations 272

7.4.1.2.12 Resource: Individual BDT Policy Configuration 273

7.4.1.2.13 Resource: Unified Traffic Pattern Subscriptions 277

7.4.1.2.14 Resource: Individual Unified Traffic Pattern Subscription 278

7.4.1.2A Custom Operations without associated resources 284

7.4.1.2A.1 Overview 284

7.4.1.2A.2 Operation: RelTransRequest 284

7.4.1.3 Notifications 285

7.4.1.3.1 General 285

7.4.1.3.2 Notify_UP_Delivery_Mode 286

7.4.1.3.3 BDT_Negotiation_Notification 287

7.4.1.3.4 Unified_Traffic_Pattern_Notification 288

7.4.1.4 Data Model 289

7.4.1.4.1 General 289

7.4.1.4.2 Structured data types 293

7.4.1.4.3 Simple data types and enumerations 312

7.4.1.5 Error Handling 314

7.4.1.5.1 General 314

7.4.1.5.2 Protocol Errors 314

7.4.1.5.3 Application Errors 314

7.4.1.6 Feature negotiation 316

7.4.2 SS_NetworkResourceMonitoring API 316

7.4.2.1 API URI 316

7.4.2.2 Resources 317

7.4.2.2.1 Overview 317

7.4.2.2.2 Resource: Unicast Monitoring Subscriptions 317

7.4.2.2.3 Resource: Individual Unicast Monitoring Subscription 318

7.4.2.3 Notifications 323

7.4.2.3.1 General 323

7.4.2.3.2 Individual Unicast Monitoring Notification 323

7.4.2.4 Data Model 324

7.4.2.4.1 General 324

7.4.2.4.2 Structured data types 326

7.4.2.4.3 Simple data types and enumerations 331

7.4.2.5 Error Handling 333

7.4.2.5.1 General 333

7.4.2.5.2 Protocol Errors 333

7.4.2.5.3 Application Errors 333

7.4.2.6 Feature negotiation 333

7.4.3 SS_ValUeConfiguration API 334

7.4.3.1 Introduction 334

7.4.3.2 Usage of HTTP 334

7.4.3.3 Resources 334

7.4.3.4 Custom Operations without associated resources 334

7.4.3.4.1 Overview 334

7.4.3.4.2 Operation: PowerSavingAssistRequest 335

7.4.3.5 Notifications 336

7.4.3.6 Data Model 336

7.4.3.6.2 Structured data types 336

7.4.3.6.3 Simple data types and enumerations 338

7.4.3.6.4 Data types describing alternative data types or combinations of data types 338

7.4.3.7 Error Handling 338

7.4.3.7.1 General 338

7.4.3.7.2 Protocol Errors 339

7.4.3.9 Security 339

7.4.4 SS_MMetaConnectivityRequirements API 339

7.4.4.1 Introduction 339

7.4.4.2 Usage of HTTP and common API related aspects 339

7.4.4.3 Resources 340

7.4.4.3.1 Overview 340

7.4.4.3.2 Resource: Connectivity Requirements Subscriptions 340

7.4.4.3.3 Resource: Individual Connectivity Requirements Subscription 342

7.4.4.4 Custom Operations without associated resources 346

7.4.4.5 Notifications 347

7.4.4.5.1 General 347

7.4.4.5.2 Individual Connectivity Requirements Notification 347

7.4.4.6 Data Model 348

7.4.4.6.1 General 348

7.4.4.6.2 Structured data types 349

7.4.4.6.3 Simple data types and enumerations 351

7.4.4.6.4 Data types describing alternative data types or combinations of data types 351

7.4.4.6.5 Binary data 351

7.4.4.7 Error Handling 351

7.4.4.7.1 General 351

7.4.4.7.2 Protocol Errors 351

7.4.4.7.3 Application Errors 352

7.4.4.8 Feature negotiation 352

7.4.4.9 Security 352

7.5 Event APIs 352

7.5.1 SS_Events API 352

7.5.1.1 API URI 352

7.5.1.2 Resources 352

7.5.1.2.1 Overview 352

7.5.1.2.2 Resource: SEAL Events Subscriptions 353

7.5.1.2.3 Resource: Individual SEAL Events Subscription 354

7.5.1.3 Notifications 357

7.5.1.3.1 General 357

7.5.1.3.2 SEAL Event Notification 358

7.5.1.4 Data Model 359

7.5.1.4.1 General 359

7.5.1.4.2 Structured data types 365

7.5.1.4.3 Simple data types and enumerations 381

7.5.1.5 Error Handling 383

7.5.1.5.1 General 383

7.5.1.5.2 Protocol Errors 383

7.5.1.5.3 Application Errors 383

7.5.1.6 Feature Negotiation 384

7.6 Key management APIs 387

7.6.1 SS_KeyInfoRetrieval API 387

7.6.1.1 API URI 387

7.6.1.2 Resources 387

7.6.1.2.1 Overview 387

7.6.1.2.2 Resource: Key Records 388

7.6.1.3 Notifications 389

7.6.1.4 Data Model 389

7.6.1.4.1 General 389

7.6.1.4.2 Structured Data Types 390

7.6.1.4.3 Simple data types and enumerations 390

7.6.1.5 Error Handling 390

7.6.1.5.1 General 390

7.6.1.5.2 Protocol Errors 390

7.6.1.5.3 Application Errors 391

7.6.1.6 Feature Negotiation 391

7.6.2 SS_KMParametersProvisioning API 391

7.6.2.1 Introduction 391

7.6.2.3 Resources 391

7.6.2.4 Custom operations without associated resources 391

7.6.2.4.1 Overview 391

7.6.2.4.2 Operation: Request 392

7.6.2.5 Notifications 393

7.6.2.6 Data Model 393

7.6.2.6.1 General 393

7.6.2.6.2 Structured data types 394

7.6.2.6.3 Simple data types and enumerations 395

7.6.2.6.4 Data types describing alternative data types or combinations of data types 395

7.6.2.6.5 Binary data 395

7.6.2.7 Error Handling 396

7.6.2.7.1 General 396

7.6.2.7.2 Protocol Errors 396

7.6.2.7.3 Application Errors 396

7.6.2.8 Feature negotiation 396

7.7 Network Slice Capability Enablement APIs 396

7.8 Identity management APIs 396

7.8.1 SS_IdmParameterProvisioning API 396

7.8.1.1 API URI 396

7.8.1.2 Resources 397

7.8.1.2.1 Overview 397

7.8.1.2.2 Resource: VAL Services Configurations 398

7.8.1.2.3 Resource: Individual VAL Services Configuration 400

7.8.1.3 Custom operations without associated resources 404

7.8.1.4 Notifications 404

7.8.1.5 Data Model 405

7.8.1.5.1 General 405

7.8.1.5.2 Structured data types 405

7.8.1.5.3 Simple data types and enumerations 406

7.8.1.5.4 Data types describing alternative data types or combinations of data types 406

7.8.1.5.5 Binary data 406

7.8.1.6 Error Handling 406

7.8.1.6.1 General 406

7.8.1.6.2 Protocol Errors 406

7.8.1.6.3 Application Errors 406

7.8.1.7 Feature negotiation 407

7.9 Data Delivery APIs 407

7.10 Application data analytics enablement service configuration APIs 407

7.10.1 SS_ADAE_VALPerformanceAnalytics API 407

7.10.1.1 API URI 407

7.10.1.2 Resources 407

7.10.1.2.1 Overview 407

7.10.1.2.2 Resource: Application performance event subscription 408

7.10.1.2.3 Resource: Individual application performance event subscription 409

7.10.1.3 Notifications 411

7.10.1.3.1 General 411

7.10.1.3.2 Application performance event notification 411

7.10.1.4 Data Model 413

7.10.1.4.1 General 413

7.10.1.4.2 Structured data types 414

7.10.1.4.3 Simple data types and enumerations 418

7.10.1.5 Error Handling 419

7.10.1.5.1 General 419

7.10.1.5.2 Protocol Errors 419

7.10.1.5.3 Application Errors 419

7.10.1.6 Feature Negotiation 420

7.10.2 SS_ADAE_SlicePerformanceAnalytics API 420

7.10.2.1 API URI 420

7.10.2.2 Resources 420

7.10.2.2.1 Overview 420

7.10.2.2.2 Resource: Slice specific application performance event subscription 421

7.10.2.2.2.1 Description 421

7.10.2.2.3 Resource: Individual slice specific application performance event subscription 422

7.10.2.2.3.1 Description 422

7.10.2.3 Notifications 424

7.10.2.3.1 General 424

7.10.2.3.2 Slice-specific application performance event notification 424

7.10.2.4 Data Model 426

7.10.2.4.1 General 426

7.10.2.4.2 Structured data types 426

7.10.2.5 Error Handling 428

7.10.2.5.1 General 428

7.10.2.5.2 Protocol Errors 428

7.10.2.5.3 Application Errors 428

7.10.2.6 Feature Negotiation 429

7.10.3 SS_ADAE_Ue2UePerformanceAnalytics API 429

7.10.3.1 API URI 429

7.10.3.2 Resources 429

7.10.3.2.1 Overview 429

7.10.3.2.2 Resource: UE-to-UE session performance event subscription 430

7.10.3.2.3 Resource: Individual UE-to-UE Session Performance Event Subscription 431

7.10.3.3 Notifications 433

7.10.3.3.1 General 433

7.10.3.3.2 UE-to-UE session performance event notification 433

7.10.3.4 Data Model 435

7.10.3.4.1 General 435

7.10.3.4.2 Structured data types 436

7.10.3.4.3 Simple data types and enumerations 438

7.10.3.5 Error Handling 438

7.10.3.5.1 General 438

7.10.3.5.2 Protocol Errors 438

7.10.3.5.3 Application Errors 438

7.10.3.6 Feature Negotiation 438

7.10.4 SS_ADAE_LocationAccuracyAnalytics API 439

7.10.4.1 API URI 439

7.10.4.2 Resources 439

7.10.4.2.1 Overview 439

7.10.4.2.2 Resource: Location accuracy event subscription 440

7.10.4.2.3 Resource: Individual location accuracy event subscription 441

7.10.4.3 Notifications 443

7.10.4.3.1 General 443

7.10.4.3.2 Location accuracy event notification 443

7.10.4.4 Data Model 444

7.10.4.4.1 General 444

7.10.4.4.2 Structured data types 445

7.10.4.4.3 Simple data types and enumerations 448

7.10.4.5 Error Handling 448

7.10.4.5.1 General 448

7.10.4.5.2 Protocol Errors 448

7.10.4.5.3 Application Errors 448

7.10.4.6 Feature Negotiation 449

7.10.5 SS_ADAE_ServiceApiAnalytics API 449

7.10.5.1 API URI 449

7.10.5.2 Resources 449

7.10.5.2.1 Overview 449

7.10.5.2.2 Resource: Service API event subscription 450

7.10.5.2.3 Resource: Individual service API event subscription 451

7.10.5.3 Notifications 453

7.10.5.3.1 General 453

7.10.5.3.2 Service API event notification 453

7.10.5.4 Data Model 455

7.10.5.4.1 General 455

7.10.5.4.2 Structured data types 455

7.10.5.4.3 Simple data types and enumerations 456

7.10.5.5 Error Handling 457

7.10.5.5.1 General 457

7.10.5.5.2 Protocol Errors 457

7.10.5.5.3 Application Errors 457

7.10.5.6 Feature Negotiation 457

7.10.6 SS_ADAE_SliceUsagePatternAnalytics API 457

7.10.6.1 API URI 457

7.10.6.2 Resources 457

7.10.6.2.1 Overview 457

7.10.6.2.2 Resource: Slice usage pattern event subscriptions 458

7.10.6.2.3 Resource: Individual slice usage pattern event subscription 459

7.10.6.3 Notifications 461

7.10.6.3.1 General 461

7.10.6.3.2 Slice usage pattern event notification 462

7.10.6.4 Data Model 463

7.10.6.4.1 General 463

7.10.6.4.2 Structured data types 463

7.10.6.4.3 Simple data types and enumerations 465

7.10.6.5 Error Handling 465

7.10.6.5.1 General 465

7.10.6.5.2 Protocol Errors 465

7.10.6.5.3 Application Errors 465

7.10.6.6 Feature Negotiation 465

7.10.7 SS_ADAE_EdgeLoadAnalytics API 466

7.10.7.1 API URI 466

7.10.7.2 Resources 466

7.10.7.2.1 Overview 466

7.10.7.2.2 Resource: Edge Load Event Subscription 467

7.10.7.2.3 Resource: Individual Edge Load Event Subscription 468

7.10.7.3 Notifications 470

7.10.7.3.1 General 470

7.10.7.3.2 Edge load event notification 470

7.10.7.4 Data Model 472

7.10.7.4.1 General 472

7.10.7.4.2 Structured data types 473

7.10.7.4.3 Simple data types and enumerations 477

7.10.7.5 Error Handling 477

7.10.7.5.1 General 477

7.10.7.5.2 Protocol Errors 477

7.10.7.5.3 Application Errors 478

7.10.7.6 Feature Negotiation 478

7.10.8 SS_AADRF_DataManagement API 478

7.10.8.1 API URI 478

7.10.8.2 Resources 478

7.10.8.2.1 Overview 478

7.10.8.2.2 Resource: A-ADRF Data Management Subscriptions 479

7.10.8.2.3 Resource: Individual A-ADRF Data Management Subscription 480

7.10.8.3 Custom Operations without associated resources 481

7.10.8.3.2 Operation: Store 482

7.10.8.3.3 Operation: Remove 483

7.10.8.4 Notifications 484

7.10.8.4.1 General 484

7.10.8.4.2 Event Notification 484

7.10.8.4.3 Delete Notification 486

7.10.8.5 Data Model 487

7.10.8.5.1 General 487

7.10.8.5.2 Structured data types 488

7.10.8.5.3 Simple data types and enumerations 496

7.10.8.6 Error Handling 498

7.10.8.6.1 General 498

7.10.8.6.2 Protocol Errors 498

7.10.8.6.3 Application Errors 499

7.10.8.7 Feature negotiation 499

7.10.9 SS_ADAE_LocationRelatedUeGroupAnalytics 499

7.10.9.1 Introduction 499

7.10.9.2 Usage of HTTP and common API related aspects 499

7.10.9.3 Resources 500

7.10.9.3.1 Overview 500

7.10.9.3.2 Resource: Location-Related UE Group Analytics Subscriptions 500

7.10.9.3.3 Resource: Individual Location-Related UE Group Analytics Subscription 501

7.10.9.4 Custom Operations without associated resources 506

7.10.9.5 Notifications 506

7.10.9.5.1 General 506

7.10.9.5.2 Location-Related UE Group Analytics Notification 506

7.10.9.6 Data Model 507

7.10.9.6.1 General 507

7.10.9.6.2 Structured data types 508

7.10.9.6.3 Simple data types and enumerations 512

7.10.9.6.4 Data types describing alternative data types or combinations of data types 512

7.10.9.6.5 Binary data 512

7.10.9.7 Error Handling 513

7.10.9.7.1 General 513

7.10.9.7.2 Protocol Errors 513

7.10.9.7.3 Application Errors 513

7.10.9.8 Feature Negotiation 513

7.10.9.9 Security 513

7.10.10 SS_ADAE_CollisionDetectionAnalytics 513

7.10.10.1 Introduction 513

7.10.10.2 Usage of HTTP and common API related aspects 514

7.10.10.3 Resources 514

7.10.10.3.1 Overview 514

7.10.10.3.2 Resource: Collision Detection Analytics Subscriptions 515

7.10.10.3.3 Resource: Individual Collision Detection Analytics Subscription 516

7.10.10.4 Custom Operations without associated resources 520

7.10.10.5 Notifications 520

7.10.10.5.1 General 520

7.10.10.5.2 Collision detection analytics Notification 521

7.10.10.6 Data Model 522

7.10.10.6.1 General 522

7.10.10.6.2 Structured data types 523

7.10.10.6.3 Simple data types and enumerations 527

7.10.10.6.4 Data types describing alternative data types or combinations of data types 527

7.10.10.6.5 Binary data 528

7.10.10.7 Error Handling 528

7.10.10.7.1 General 528

7.10.10.7.2 Protocol Errors 528

7.10.10.7.3 Application Errors 528

7.10.10.8 Feature Negotiation 528

7.10.10.9 Security 528

7.10.11 SS_ADAE_AIMLMemberCapabilityAnalytics API 528

7.10.11.1 Introduction 528

7.10.11.2 Usage of HTTP and common API related aspects 529

7.10.11.3 Resources 529

7.10.11.3.1 Overview 529

7.10.11.3.2 Resource: AIML Member Capability Analytics Subscriptions 530

7.10.11.3.3 Resource: Individual AIML Member Capability Analytics Subscription 531

7.10.11.3A Custom Operations without associated resources 535

7.10.11.4 Notifications 535

7.10.11.4.1 General 535

7.10.11.4.2 AIML Member Capability Analytics Notification 536

7.10.11.5 Data Model 537

7.10.11.5.1 General 537

7.10.11.5.2 Structured data types 538

7.10.11.5.3 Simple data types and enumerations 541

7.10.11.6 Error Handling 542

7.10.11.6.1 General 542

7.10.11.6.2 Protocol Errors 542

7.10.11.6.3 Application Errors 542

7.10.11.7 Feature Negotiation 542

7.10.11.8 Security 542

7.10.12 SS_ADAE_UeRatConnectivityAnalytics API 543

7.10.12.1 Introduction 543

7.10.12.2 Usage of HTTP and common API related aspects 543

7.10.12.3 Resources 543

7.10.12.3.1 Overview 543

7.10.12.3.2 Resource: UE RAT Connectivity Analytics Subscriptions 544

7.10.12.3.3 Resource: Individual UE RAT Connectivity Analytics Subscription 545

7.10.12.4 Custom Operations without associated resources 550

7.10.12.5 Notifications 550

7.10.12.5.1 General 550

7.10.12.5.2 UE RAT Connectivity Analytics Notification 550

7.10.12.6 Data Model 551

7.10.12.6.1 General 551

7.10.12.6.2 Structured data types 552

7.10.12.6.3 Simple data types and enumerations 556

7.10.12.6.4 Data types describing alternative data types or combinations of data types 556

7.10.12.7 Error Handling 557

7.10.12.7.1 General 557

7.10.12.7.2 Protocol Errors 557

7.10.12.7.3 Application Errors 557

7.10.12.8 Feature Negotiation 557

7.10.12.9 Security 557

7.10.13 SS_ADCCF_DataCollection 557

7.10.13.1 Introduction 557

7.10.13.2 Usage of HTTP and common API related aspects 558

7.10.13.3 Resources 558

7.10.13.3.1 Overview 558

7.10.13.3.2 Resource: Data Collection Subscriptions 559

7.10.13.3.3 Resource: Individual Data Collection Subscription 560

7.10.13.4 Custom Operations without associated resources 564

7.10.13.5 Notifications 565

7.10.13.5.1 General 565

7.10.13.5.2 Data Collection Notification 565

7.10.13.6 Data Model 566

7.10.13.6.1 General 566

7.10.13.6.2 Structured data types 567

7.10.13.6.3 Simple data types and enumerations 570

7.10.13.6.4 Data types describing alternative data types or combinations of data types 571

7.10.13.6.5 Binary data 571

7.10.13.7 Error Handling 571

7.10.13.7.1 General 571

7.10.13.7.2 Protocol Errors 571

7.10.13.7.3 Application Errors 571

7.10.13.8 Feature Negotiation 572

7.10.13.9 Security 572

7.10.14 SS_ADAE_ServerToServerPerformanceAnalytics API 572

7.10.14.1 Introduction 572

7.10.14.2 Usage of HTTP and common API related aspects 572

7.10.14.3 Resources 572

7.10.14.3.2 Resource: Server to Server Performance Analytics Subscriptions 573

7.10.14.3.3 Resource: Individual Server to Server Performance Analytics Subscription 574

7.10.14.4 Custom Operations without associated resources 579

7.10.14.5 Notifications 580

7.10.14.5.1 General 580

7.10.14.5.2 Server to Server Performance analytics Notification 580

7.10.14.6 Data Model 581

7.10.14.6.1 General 581

7.10.14.6.2 Structured data types 582

7.10.14.6.3 Simple data types and enumerations 585

7.10.14.6.4 Data types describing alternative data types or combinations of data types 585

7.10.14.6.5 Binary data 585

7.10.14.7 Error Handling 586

7.10.14.7.1 General 586

7.10.14.7.2 Protocol Errors 586

7.10.14.7.3 Application Errors 586

7.10.14.8 Feature Negotiation 586

7.10.14.9 Security 586

7.10.15 SS_ADAE_DN_energy_analytics API 586

7.10.15.1 Introduction 586

7.10.15.2 Usage of HTTP and common API related aspects 587

7.10.15.3 Resources 587

7.10.15.3.1 Overview 587

7.10.15.3.2 Resource: ADAE DN Energy Analytics 587

7.10.15.4 Custom Operations without associated resources 589

7.10.15.5 Notifications 589

7.10.15.6 Data Model 589

7.10.15.6.1 General 589

7.10.15.6.2 Structured data types 589

7.10.15.6.3 Simple data types and enumerations 590

7.10.15.6.4 Data types describing alternative data types or combinations of data types 591

7.10.15.6.5 Binary data 591

7.10.15.7 Error Handling 591

7.10.15.7.1 General 591

7.10.15.7.2 Protocol Errors 591

7.10.15.7.3 Application Errors 591

7.10.15.8 Feature Negotiation 591

7.10.15.9 Security 592

7.10.16 SS_ADAE_AIMLEnergyConsumptionAnalytics 592

7.10.17 SS_ADAE_AIMLEClientEnergySustainabilityAnalytics API 605

7.11 AIML Enablement APIs 611

7.12 Metaverse Enablement APIs 611

7.13 Digital Asset APIs 611

7.13.1 SS_DAProfileManagement Service API 611

7.13.1.1 Introduction 611

7.13.1.3 Resources 612

7.13.1.3.1 Overview 612

7.13.1.3.2 Resource: DA Profiles 612

7.13.1.3.3 Resource: Individual DA profile 613

7.13.1.4 Custom Operations without associated resources 618

7.13.1.5 Notifications 618

7.13.1.6 Data Model 618

7.13.1.6.1 General 618

7.13.1.6.2 Structured data types 619

7.13.1.6.3 Simple data types and enumerations 621

7.13.1.7 Error Handling 621

7.13.1.7.1 General 621

7.13.1.7.2 Protocol Errors 622

7.13.1.7.3 Application Errors 622

7.13.1.8 Feature negotiation 622

7.13.1.9 Security 622

7.13.2 SS_DADiscovery Service API 622

7.13.2.1 Introduction 622

7.13.2.3 Resources 623

7.13.2.3.1 Overview 623

7.13.2.3.2 Resource: Digital Asset 623

7.13.2.4 Custom Operations without associated resources 625

7.13.2.5 Notifications 625

7.13.2.6 Data Model 625

7.13.2.6.1 General 625

7.13.2.6.2 Structured data types 626

7.13.2.6.3 Simple data types and enumerations 627

7.13.2.7 Error Handling 627

7.13.2.7.1 General 627

7.13.2.7.2 Protocol Errors 627

7.13.2.7.3 Application Errors 628

7.13.2.8 Feature negotiation 628

7.13.2.9 Security 628

7.13.3 SS_DAMediaManagement Service API 628

7.13.3.1 Introduction 628

7.13.3.3 Resources 628

7.13.3.3.1 Overview 628

7.13.3.3.2 Resource: DA medias 629

7.13.3.3.3 Resource: Individual DA media 630

7.13.3.4 Custom Operations without associated resources 634

7.13.3.5 Notifications 634

7.13.3.6 Data Model 635

7.13.3.6.1 General 635

7.13.3.6.2 Structured data types 635

7.13.3.6.3 Simple data types and enumerations 636

7.13.3.7 Error Handling 637

7.13.3.7.1 General 637

7.13.3.7.2 Protocol Errors 637

7.13.3.7.3 Application Errors 637

7.13.3.8 Feature negotiation 637

7.13.3.9 Security 637

7.13.4 SS_DAUsageReport API 637

7.13.4.1 Introduction 637

7.13.4.3 Resources 638

7.13.4.3.1 Overview 638

7.13.4.3.2 Resource: DA Usage Report Subscriptions 639

7.13.4.3.3 Resource: Individual DA Usage Report Subscription 640

7.13.4.4 Custom Operations without associated resources 645

7.13.4.5 Notifications 645

7.13.4.5.2 DA Usage Report Notification 645

7.13.4.6 Data Model 646

7.13.4.6.1 General 646

7.13.4.6.2 Structured data types 647

7.13.4.6.3 Simple data types and enumerations 652

7.13.4.6.4 Data types describing alternative data types or combinations of data types 653

7.13.4.6.5 Binary data 653

7.13.4.7 Error Handling 653

7.13.4.7.1 General 653

7.13.4.7.2 Protocol Errors 653

7.13.4.8 Feature negotiation 653

7.13.4.9 Security 653

8 Using Common API Framework 654

8.1 General 654

8.2 Security 654

9 Security 655

9.1 General 655

9.2 SEAL-S security 655

Annex A (normative): OpenAPI specification 656

A.1 General 656

A.2 SS_LocationReporting API 656

A.3 SS_GroupManagement API 664

A.4 SS_UserProfileRetrieval API 669

A.5 SS_NetworkResourceAdaptation API 671

A.6 SS_Events API 704

A.7 SS_KeyInfoRetrieval API 717

A.8 SS_LocationAreaInfoRetrieval API 719

A.9 Void 720

A.10 SS_NetworkResourceMonitoring API 720

A.11 SS_VALServiceData API 729

A.12 SS_VALServiceAreaConfiguration API 731

A.13 SS_IdmParameterProvisioning API 740

A.14 SS_KMParametersProvisioning API 745

A.15 SS_ADAE_VALPerformanceAnalytics API 746

A.16 SS_ADAE_SlicePerformanceAnalytics API 753

A.17 SS_ADAE_Ue2UePerformanceAnalytics API 757

A.18 SS_ADAE_LocationAccuracyAnalytics API 762

A.19 SS_ADAE_ServiceApiAnalytics API 767

A.20 SS_ADAE_SliceUsagePatternAnalytics API 770

A.21 SS_ADAE_EdgeLoadAnalytics API 774

A.22 SS_AADRF_DataManagement API 778

A.23 SS_ADAE_LocationRelatedUeGroupAnalytics API 790

A.24 SS_ADAE_CollisionDetectionAnalytics API 797

A.25 SS_LocationHistoryInfoEvent API 803

A.27 SS_SLPositioningManagement API 815

A.28 SS_ADAE_AIMLMemberCapabilityAnalytics API 823

A.29 SS_ADCCF_DataCollection API 829

A.30 SS_ADAE_ServerToServerPerformanceAnalytics API 836

A.31 SS_ADAE_UeRatConnectivityAnalytics API 843

A.32 SS_ASCAIInfoRetrieval API 849

A.33 SS_DAProfileManagement API 852

A.34 SS_DADiscovery API 858

A.35 SS_DAMediaManagement API 861

A.36 SS_ADAE_DN_energy_analytics API 865

A.40 SS_ADAE_AIMLEClientEnergySustainabilityAnalytics API 883

A.41 SS_MMetaConnectivityRequirement API 885

Annex B (normative): SEAL NRM server support integration with TSN 892

Annex C (informative): Change history 893
