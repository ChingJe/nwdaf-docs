---
spec: TR 23.700-80
version: 18.0.0
release: '18'
clause: contents
title: Contents
source_archive: 23700-80-i00.zip
source_document: 23700-80-i00.docx
content_origin: 3gpp-source
---

# Contents

Foreword 9

1 Scope 11

2 References 11

3 Definitions of terms and abbreviations 12

3.1 Terms 12

3.2 Abbreviations 12

4 Architectural Requirements and Assumptions 12

4.1 Architectural Requirements 12

4.2 Architectural Assumptions 12

4.3 Reference Architecture 13

5 Key Issues 13

5.1 Key Issue \#1: Monitoring of network resource utilization for support of Application AI/ML operations 13

5.2 Key Issue \#2: 5GC information exposure to UE 13

5.3 Key Issue \#3: 5GC Information Exposure to authorized 3rd party for Application Layer AI / ML Operation 14

5.4 Key Issue \#4: Enhancing External Parameter Provisioning 14

5.5 Key Issue \#5: 5GC Enhancements to enable Application AI/ML Traffic Transport 14

5.6 Key Issue \#6: QoS and Policy enhancements 15

5.7 Key Issue \#7: 5GS Assistance to Federated Learning Operation 15

6 Solutions 17

6.0 Mapping Solutions to Key Issues 17

6.1 Solution \#1: 5GS Monitoring Capabilities for AI/ML-based Services 19

6.1.1 Description 19

6.1.2 Procedures 20

6.1.2.1 Procedure for QoS monitoring 20

6.1.2.2 Procedure for monitoring of session inactivity time and traffic volume events 22

6.1.3 Impacts on services, entities and interfaces 23

6.2 Solution \#2: User Plane solution for 5GC information exposure to UE 23

6.2.1 Description 23

6.2.2 Procedures 24

6.2.2.1 General 24

6.2.2.2 UE ID retrieval 25

6.2.2.2.1 IEAF based solution 25

6.2.2.2.2 NEF/NWDAF based solution 28

6.2.2.3 Network consent and Multiple IEAF 30

6.2.2.4 Using DCCF to deliver analytics for UEs 32

6.2.3 Impacts on services, entities and interfaces 33

6.3 Solution \#3: 5GC information exposure to UE by SMF 34

6.3.1 Description 34

6.3.2 Procedures 36

6.3.2.1 Procedure for enabling analytics information exposure to UE 36

6.3.2.2 Procedure for disabling analytics information exposure to UE 39

6.3.3 Impacts on services, entities and interfaces 40

6.4 Solution \#4: 5GC Analytics exposure to UE 40

6.4.1 Description 40

6.4.2 Procedures 41

6.4.3 Impacts on services, entities and interfaces 41

6.5 Solution \#5: 5GC information exposure to UE 42

6.5.1 Description 42

6.5.1.1 New capabilities to support UE System Performance and Predictions requests 42

6.5.2 Procedures 44

6.5.3 Impacts on services, entities and interfaces 45

6.6 Solution \#6: Federated learning analytics as assistance to AI/ML application server 46

6.6.1 Description 46

6.6.2 Procedures 47

6.6.3 Impacts on services, entities and interfaces 48

6.7 Solution \#7: 5GS information exposure for AI/ML operation 49

6.7.1 Description 49

6.7.2 Procedures 49

6.7.2.1 Data rate monitoring scheme based on AF session with required QoS 49

6.7.2.2 Data rate monitoring scheme based on service information provision 51

6.7.3 Impacts on services, entities and interfaces 51

6.8 Solution \#8: 5GC information/analytics notification to the AF and the UE 51

6.8.1 Introduction 51

6.8.2 Description 52

6.8.3 Procedures 53

6.8.3.1 AF Subscription for 5GC information or analytics 53

6.8.3.2 5GC information or analytics notification to the AF and the UE 55

6.8.4 Impacts on services, entities and interfaces 56

6.9 Solution \#9: AF influence on AI/ML operations related policies 57

6.9.1 Description 57

6.9.2 Procedures 57

6.9.2.1 Procedure of AF influence on area-based AI/ML operations related policies 57

6.9.3 Impacts on services, entities and interfaces 60

6.10 Solution \#10: Application Data Transfer to support planned and event driven AI/ML traffic transport 60

6.10.1 Description 60

6.10.2 Solutions 61

6.10.2.1 ADT Mechanism to support Application AI/ML data transfer 62

6.10.2.2 ADT Negotiation for future PDU session to support Application AI/ML data transfer 63

6.10.2.3 Activation or Updating/Removing ADT Transfer to a PDU Session 65

6.10.2.4 NWDAF Analytics support for ADT warning notification 66

6.10.3 Impacts on services, entities and interfaces 67

6.11 Solution \#11: Traffic routing enhancements for Application AIML Traffic Transport 67

6.11.1 Description 67

6.11.2 Procedures 68

6.11.3 Impacts on services, entities and interfaces 69

6.12 Solution \#12: Enhanced charging for Application AIML traffic transport 69

6.12.1 Description 69

6.12.2 Procedures 69

6.12.3 Impacts on services, entities and interfaces 71

6.13 Solution \#13: Support Application AI/ML Traffic Transport 72

6.13.1 Description 72

6.13.2 Procedures 73

6.13.3 Impacts on services, entities and interfaces 74

6.14 Solution \#14: Flock QoS based Federated Learning Operation 74

6.15 Solution \#15: QoS monitoring for AIML application 75

6.15.1 Description 75

6.15.2 Procedures 76

6.15.3 Impacts on services, entities and interfaces 77

6.16 Solution \#16: Solution for supporting the aggregated QoS parameters based on the Group-MBR 77

6.16.1 Description 77

6.16.2 Procedures 80

6.16.2.1 Defining Group-MBR Traffic Monitoring Parameter 80

6.16.2.2 Extensions to AF Influence Information Element in AF request to monitor and report aggregate bit rate among the specific group of PDU sessions 80

6.16.2.3 Extensions to AF Influence Network Function Service Procedure to enable the support for Group-MBR monitoring 81

6.16.2.4 Extensions to AF Influence on traffic routing procedure for Session not identified by an UE address to enable the support for Group-MBR monitoring 82

6.16.2.5 The procedure of Group-MBR monitoring 83

6.16.2.6 Extension to Nnef_EventExposure Services for Event Monitoring to support Group-MBR monitoring 84

6.16.3 Impacts on services, entities and interfaces 85

6.17 Solution \#17: 5GS assistance for FL member selection using QoS monitoring 85

6.17.1 Description 85

6.17.2 Procedures 86

6.17.3 Impacts on services, entities and interfaces 87

6.18 Solution \#18: 5GS assistance for FL member selection based on UE's visited AOI information 87

6.18.1 Description 88

6.18.2 Procedures 90

6.18.3 Impacts on services, entities and interfaces 91

6.19 Solution \#19: 5G assisted FL member selection 91

6.19.1 Description 91

6.19.2 Procedures 92

6.19.3 Impacts on services, entities and interfaces 93

6.20 Solution \#20: 5GS assistance to reduce the latency divergence 93

6.20.1 Description 93

6.20.2 Procedures 95

6.20.3 Impacts on services, entities and interfaces 96

6.21 Solution \#21: Exposure of UE performance or a group of UEs performance 96

6.21.1 Description 96

6.21.2 Procedures 97

6.21.3 Impacts on services, entities and interfaces 98

6.22 Solution \#22: 5GC-assisted Federated Learning over 5G VN group 98

6.22.1 Description 98

6.22.2 Procedures 98

6.22.2.1 Procedure of VN group establishment by AF 98

6.22.3 Impacts on services, entities and interfaces 100

6.23 Solution \#23: Federated Learning Assistance Function assisting on federated learning members selection 100

6.23.1 Description 100

6.23.2 Procedures 101

6.23.3 Impacts on services, entities and interfaces 103

6.24 Solution \#24: 5GS Assistance to Federated Learning 104

6.24.1 Description 104

6.24.2 Procedures 104

6.24.3 Impacts on services, entities and interfaces 105

6.25 Solution \#25: Providing 5GS Assistance Information to AF for Federated Learning Operation 105

6.25.1 Description 105

6.25.2 Procedures 106

6.25.2.1 Procedure for Providing 5GS Assistance Information for Federated Learning Operation 106

6.25.3 Impacts on services, entities and interfaces 109

6.26 Solution \#26: FL operation with 5GS coordination for UE selection 110

6.26.1 Description 110

6.26.2 Procedures 110

6.26.3 Impacts on Existing Nodes and Functionality 111

6.27 Solution \#27: Assistance to selection of UEs for federated learning operation 111

6.27.1 Description 111

6.27.2 Procedures 112

6.27.3 Impacts on services, entities and interfaces 113

6.28 Solution \#28: Application AI/ML Assistance Services 113

6.28.1 Description 113

6.28.2 AaaML Service Architecture 116

6.28.3 AaaML Services 117

6.28.3 Procedures for AaaML Services 119

6.28.3.1 AaaML Service Profile support for AaaML Service Procedures 119

6.28.3 Impacts on services, entities and interfaces 120

6.29 Solution \#29: 5GC assistance to UE on performing AI/ML task 120

6.29.1 Description 120

6.29.2 Procedures 121

6.29.2.1 Procedure for AF provisioning information to 5GC 121

6.29.2.2 Procedure for 5GC exposing information to UE 121

6.29.2.3 Procedure for 5GC exposing information to UE via UE policy 122

6.29.3 Impacts on Existing Nodes and Functionality 124

6.30 Solution \#30: AI/ML Translator (AI/ML-T) for 5GC Information Exposure to UE 125

6.30.1 Description 125

6.30.2 Procedures 126

6.30.3 Impacts on Existing Nodes and Functionality 127

6.31 Solution \#31: Network information exposure to UE 127

6.31.1 Description 127

6.31.2 Procedures 128

6.31.3 Impacts on Existing Nodes and Functionality 129

6.32 Solution \#32: 5GC Information Exposure to Authorized 3rd Party for Assisting AI/ML Operation Splitting between AI/ML Endpoints 129

6.32.1 Description 129

6.32.2 Procedures 131

6.32.3 Impacts on services, entities and interfaces 132

6.33 Solution \#33: 5GC Exposure of Network Authorization for UE to participate in the Application AI/ML operation 133

6.33.1 Description 133

6.33.2 Procedure 134

6.33.2.1 Procedure for 5GC Exposure of Network Authorization Status of UE for Application AI/ML operation 134

6.33.2.2 New Subscription Data of Authorized Application AI/ML services 135

6.33.2.2.1 General Descriptions 135

6.33.2.2.2 Application AI/ML subscription data in UDM 135

6.33.2.2.3 Extension to Nnef_EventExposure Services for New Event Monitoring for 5G system authorization for UE to support Application AI/ML operation 136

6.33.3 Impacts on existing services, entities and interfaces 136

6.34 Solution \#34: Expose AI/ML traffic transmission status to the authorized 3rd party for Application Layer AI/ML Operation 137

6.34.1 Description 137

6.34.2 Procedures 137

6.34.3 Impacts on services, entities and interfaces 139

6.35 Solution \#35: External parameter provisioning by AF for AI/ML data transport 139

6.35.1 Description 139

6.35.2 Procedures 140

6.35.3 Impacts on services, entities and interfaces 140

6.36 Solution \#36: Enhancing External Parameter Provisioning 140

6.36.1 Description 140

6.36.2 Procedures 141

6.36.3 Impacts on services, entities and interfaces 143

6.37 Solution \#37: Solution for 5GC Assistance to support Group-MBR Monitoring 143

6.37.1 Description 143

6.37.2 Procedures 145

6.37.2.1 Defining Group-MBR Traffic Monitoring Parameter 145

6.37.2.2 Extensions to AF Influence Information Element in AF request to monitor and report aggregate bit rate among the specific group of PDU sessions 146

6.37.2.3 Extensions to AF Influence Network Function Service Procedure to enable the support for Group-MBR monitoring 146

6.37.2.4 Extensions to UPF Services to support Group-MBR monitoring 147

6.37.2.5 Extensions to AF Influence on traffic routing negotiation procedure to enable the group service provisioning and the activation for Group-MBR monitoring 148

6.37.2.6 The procedure of Group-MBR monitoring 150

6.37.3 Impacts on services, entities and interfaces 151

6.38 Solution \#38: Time dependent QoS for ML model distribution in federated learning 151

6.38.1 Description 151

6.38.2 Procedures 152

6.38.2.1 Procedure based on setting up an AF session for QoS 152

6.38.2.2 Procedure for UEs' staggering with notification to the AI/ML application server 154

6.38.3 Impacts on services, entities and interfaces 156

6.39 Solution \#39: 5GC information assistance to FL members' selection 156

6.39.1 Description 156

6.39.2 Procedures 158

6.39.3 Impacts on services, entities and interfaces 161

6.40 Solution \#40: Flexible Federated Learning Operation over 5GS 161

6.40.1 Description 161

6.40.1.1 Overview 161

6.40.1.2 Allocation of an AMSID to the UE 162

6.40.2 Procedures 163

6.40.2.1 Procedure for group QoS request for FL by AF 163

6.40.2.2 Procedure for enabling FL operation configuration recommendation by AFLSF 164

6.40.2.3 Procedure for UE-initiated request to join an AI/ML session for FL 166

6.40.2.3.1 Using a session management request 166

6.40.2.3.2 Using an application layer request 167

6.40.2.4 Procedure for multiple PCF discovery 168

6.40.3 Impacts on services, entities and interfaces 168

6.41 Solution \#41: Solution for supporting aggregated UE performance monitoring and exposure for a group of UEs 169

6.41.1 Description 169

6.41.2 Procedures 169

6.41.3 Impacts on services, entities and interfaces 169

6.42 Solution \#42: FL operation support by 5GS based on AF session with required QoS provided by Application Server 170

6.42.1 Description 170

6.42.2 Procedures 171

6.42.3 Impacts on services, entities and interfaces 173

6.43 Solution \#43: FL member selection 173

6.43.1 Description 173

6.43.2 Procedures 173

6.43.2.1 Procedure for AF directly collection information for FL member selection 173

6.43.3 Impacts on services, entities and interfaces 174

6.44 Solution \#44: 5G assisted FL member selection for application request 174

6.44.1 Description 174

6.44.2 Procedures 175

6.44.3 Impacts on services, entities and interfaces 175

6.45 Solution \#45: Improving FL performance by selecting central application server with 5GC's assistance 176

6.45.1 Description 176

6.45.2 Procedures 177

6.45.3 Impacts on services, entities and interfaces 178

6.46 Solution \#46: 5GS assistance for FL member selection based on UE's location and direction 179

6.46.1 Description 179

6.46.2 Procedures 180

6.46.3 Impacts on services, entities and interfaces 181

6.47 Solution \#47: Composite exposure service for 5GS assistance to FL member selection 182

6.47.1 Description 182

6.47.2 Procedures 182

6.47.3 Impacts on Existing Nodes and Functionality 183

7 Evaluation 183

7.1 Key Issue \#1: Monitoring of network resource utilization for support of Application AI/ML operations 183

7.2 Key Issue \#2: 5GC information exposure to UE 188

7.3 Key Issue \#3: 5GC Information Exposure to authorized 3rd party for Application Layer AI/ML Operation 196

7.4 Key Issue \#4: Enhancing External Parameter Provisioning 199

7.5 Key Issue \#5: 5GC Enhancements to enable Application AI/ML Traffic Transport 200

7.6 Key Issue \#6: QoS and Policy enhancements 202

7.6.1 QoS performance measurement assistance to Application AI/ML operation 202

7.6.2 Monitoring Group-MBR 204

7.6.3 Other topics 205

7.6.3.1 QoS request for a group of UEs 205

7.7 Key Issue \#7: 5GS Assistance to Federated Learning Operation 205

7.7.1 Evaluation for solutions based on the Performance monitoring/exposure 208

7.8 Usage of analytics 209

7.8.1 Overview 209

7.8.2 Usage of analytics for assistance to federated learning 214

8 Conclusion 214

8.0 General Architecture Principles 214

8.1 Key Issue \#1: Monitoring of network resource utilization for support of Application AI/ML operations 215

8.2 Key Issue \#2: 5GC information exposure to UE 215

8.3 Key Issue \#3: 5GC Information Exposure to authorized 3rd party for Application Layer AI/ML Operation 216

8.4 Key Issue \#4: Enhancing External Parameter Provisioning 216

8.5 Key Issue \#5: 5GC Enhancements to enable Application AI/ML Traffic Transport 216

8.6 Key Issue \#6: QoS and Policy enhancements 217

8.6.1 QoS performance measurement assistance to Application AI/ML operation 217

8.6.2 QoS request for a group of UEs 218

8.6.3 Performance KPIs mapping to 5GS QoS parameters 218

8.7 Key Issue \#7: 5GS Assistance to Federated Learning Operation 218

8.7.1 Performance monitoring/exposure 219

8.7.2 Analytics to Assist Federated Learning Operation 219

Annex A: Change history 220
