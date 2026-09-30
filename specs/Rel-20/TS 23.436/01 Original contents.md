---
spec: TS 23.436
version: 20.2.0
release: '20'
clause: contents
title: Contents
source_archive: 23436-k20.zip
source_document: 23436-k20.docx
content_origin: 3gpp-source
---

# Contents

Foreword 12

Introduction 13

1 Scope 14

2 References 14

3 Definitions of terms and abbreviations 15

3.1 Terms 15

3.2 Abbreviations 15

4 Architectural requirements 15

4.1 General Description 15

4.2 General Requirements 15

4.3 ADAE internal architecture requirements 16

4.4 ADAE capability related requirements 16

5 Application architecture for ADAES 16

5.1 General 16

5.2 Functional architecture 16

5.2.1 General 16

5.2.2 On-network Functional Architecture 17

5.2.3 Off-network Functional Architecture 18

5.2.4 Functional Architecture for supporting interactions with SEAL AIMLE 19

5.3 ADAE internal architecture 19

5.4 Functional entities description 20

5.4.1 General 20

5.4.2 Application Data Analytics Enablement client 20

5.4.3 Application Data Analytics Enablement server 21

5.5 Reference points description 21

5.5.1 General 21

5.5.2 ADAE-UU 21

5.5.3 ADAE-PC5 21

5.5.4 ADAE-C 22

5.5.5 ADAE-S 22

5.5.4 ADAE-X 22

5.5.5 ADAE-Y 22

5.5.6 ADCCF-1 22

5.5.7 AADRF-1 22

5.5.8 SEAL-X 22

5.5.9 AIML-X 22

6 ADAE layer Functional Description 22

6.1 Support for application performance analytics 22

6.2 Support for slice-specific application performance analytics 23

6.3 Support for UE-to-UE application performance analytics 23

6.4 Support for location accuracy analytics 23

6.5 Support for service API analytics 23

6.6 Slice usage pattern analytics 23

6.7 Support for edge load analytics 23

6.8 Edge computing preparation analytics 24

6.9 Support for server-to-server performance analytics 24

6.10 Support for collision detection analytics 24

6.11 Support for location-related UE group analytics 24

6.12 Support for Application Layer AI/ML Member Capability Analytics 24

6.13 Support for VAL performance analytics for tethered UEs 24

6.14 Support for DN Energy Analytics 24

6.15 Support for ML Model Performance Degradation Detection 25

6.16 Support for monitoring ML-enabled analytics correctness 25

6.17 Support for Energy information Analytics 25

6.18 Support for AI/ML energy consumption analytics 25

6.19 Support for AIMLE client energy sustainability analytics 25

7 Identities and commonly used values 25

7.1 General 25

7.2 ADAE Server ID 25

7.3 ADAE client ID 25

7.4 A-ADRF ID 26

7.5 A-DCCF ID 26

7.6 Data Producer ID 26

7.7 ADAE service area 26

7.8 Analytics ID 26

8 Procedures and information flows 26

8.1 General 26

8.2 Procedure on support for application performance analytics 26

8.2.1 General 26

8.2.2 Procedure on VAL server performance analytics 26

8.2.3 Procedure on VAL session performance analytics 29

8.2.4 Information flows 31

8.2.4.1 General 31

8.2.4.2 VAL performance analytics subscription request 32

8.2.4.3 VAL performance analytics subscription response 32

8.2.4.4 Data collection subscription request 32

8.2.4.5 Data collection subscription response 33

8.2.4.6 Data Notification 33

8.2.4.7 Analytics Notification 34

8.2.4.8 Data producer profile 36

8.3 Procedure on support for slice-specific application performance analytics 36

8.3.1 General 36

8.3.2 Procedure 36

8.3.3 Information flows 38

8.3.3.1 General 38

8.3.3.2 Slice-specific performance analytics subscription request 38

8.3.3.3 Slice-specific performance analytics subscription response 39

8.3.3.4 Slice-specific performance analytics notification 39

8.4 Procedure on support for UE-to-UE application performance analytics 39

8.4.1 General 39

8.4.2 Procedure 39

8.4.3 Information flows 41

8.4.3.1 General 41

8.4.3.2 UE-to-UE session performance analytics subscription request 41

8.4.3.3 UE-to-UE session performance analytics subscription response 42

8.4.3.4 UE-to-UE analytics request 42

8.4.3.5 UE-to-UE analytics response 43

8.4.3.6 ADAE Analytics Notification 43

8.5 Procedure on support for location accuracy analytics 44

8.5.1 General 44

8.5.2 Procedure 44

8.5.3 Information flows 46

8.5.3.1 General 46

8.5.3.2 Location accuracy analytics subscription request 46

8.5.3.3 Location accuracy analytics subscription response 46

8.5.3.4 Location accuracy data request 46

8.5.3.5 Location accuracy data response 47

8.5.3.6 Location accuracy analytics notification 47

8.6 Procedure for supporting service API analytics 48

8.6.1 General 48

8.6.2 Procedure 48

8.6.3 Information flows 50

8.6.3.1 General 50

8.6.3.2 Service API event subscription request 50

8.6.3.3 Service API event subscription response 50

8.6.3.4 Historical service API logs request 51

8.6.3.5 Historical service API logs response 51

8.6.3.6 Service API analytics notification 52

8.7 Slice usage pattern analytics 52

8.7.1 General 52

8.7.2 Procedure on slice usage pattern analytics 52

8.7.3 Procedure on retrieving slice usage statistics data 54

8.7.4 Information flows 54

8.7.4.1 General 54

8.7.4.2 Network slice usage pattern analytics subscription request 54

8.7.4.3 Network slice usage pattern analytics subscription response 55

8.7.4.4 Network slice usage pattern analytics notification 55

8.7.4.5 Network slice data retrieval request 56

8.7.4.6 Network slice data retrieval response 56

8.7.4.7 Slice usage statistics data request 57

8.7.4.8 Slice usage statistics data response 58

8.8 Procedure for supporting edge load analytics 58

8.8.1 General 58

8.8.2 Procedure 58

8.8.2.1 Subscribe-notify model 58

8.8.2.2 Request-response model 60

8.8.3 Information flows 61

8.8.3.1 General 61

8.8.3.2 Edge analytics subscription request 61

8.8.3.3 Edge analytics subscription response 61

8.8.3.4 Edge data collection subscription request 62

8.8.3.5 Edge data collection subscription response 62

8.8.3.6 Data Notification 62

8.8.3.7 Edge analytics Notification 63

8.8.3.8 Get analytics data request 64

8.8.3.9 Get analytics data response 64

8.9 Procedure on Service experience to support application performance analytics 65

8.9.1 General 65

8.9.2 Procedure 65

8.9.2.1 Push service experience information 65

8.9.2.2 Pull service experience information 66

8.9.2.3 Service experience information based on triggers 67

8.9.3 Information flows 67

8.9.3.1 Push service experience information request 67

8.9.3.2 Push service experience information response 68

8.9.3.3 Pull service experience information request 68

8.9.3.4 Pull service experience information response 68

8.9.3.5 Configure service experience report trigger request 69

8.9.3.6 Configure service experience report trigger response 69

8.10 Procedure on support for data storage 69

8.10.1 General 69

8.10.2 Procedure 69

8.10.2.1 Notification based data storage 69

8.10.2.2 Direct data storage 70

8.10.2.3 Data removal from an A-ADRF 71

8.10.3 Information flows 72

8.10.3.1 General 72

8.10.3.2 Data storage subscription request 72

8.10.3.3 Data storage subscription response 73

8.10.3.4 Data storage request 73

8.10.3.5 Data storage response 74

8.10.3.6 Data deletion notification 74

8.10.3.7 Data deletion request 74

8.10.3.8 Data deletion response 75

8.11 Procedure for edge computing preparation analytics 75

8.11.1 General 75

8.11.2 Procedure 75

8.11.2.1 Subscribe-notify model 75

8.11.2.2 Request-response model 76

8.11.3 Information flows 77

8.11.3.1 General 77

8.11.3.2 Edge computing preparation analytics subscription request 77

8.11.3.3 Edge computing preparation analytics subscription response 78

8.11.3.4 Edge computing preparation analytics notification 78

8.11.3.5 Edge computing preparation data request 79

8.11.3.6 Edge computing preparation data response 79

8.11.3.7 Edge computing preparation analytics retrieval request 79

8.11.3.8 Edge computing preparation analytics retrieval response 80

8.12 Procedure for supporting data collection to A-DCCF 80

8.12.1 General 80

8.12.2 Procedure 81

8.12.2.1 Subscribe-notify model 81

8.12.2.2 Request-response model 82

8.12.3 Information flows 83

8.12.3.1 General 83

8.12.3.2 Data collection subscription request 83

8.12.3.3 Data collection subscription response 83

8.12.3.4 Data collection notification 84

8.12.3.5 Get Data/Analytics request 84

8.12.3.6 Data collection response 85

8.12.3.7 Data producer profile 86

8.13 Procedure on support for server-to-server performance analytics 86

8.13.1 General 86

8.13.2 Procedure 86

8.13.3 Information flows 88

8.13.3.1 General 88

8.13.3.2 Server-to-server performance analytics subscription request 88

8.13.3.3 Server-to-server performance analytics subscription response 88

8.13.3.4 Server-to-server performance analytics notification 88

8.13.3.5 Inter-server session data request 89

8.13.3.6 Inter-server session data response 89

8.13.3.7 Server-to-server analytics request 90

8.13.3.8 Server-to-server analytics response 90

8.14 Procedure for Collision Detection Analytics 90

8.14.1 General 90

8.14.2 Procedure 90

8.14.2.1 Subscribe-notify model 90

8.14.2.2 Request-response model 92

8.14.3 Information flows 92

8.14.3.1 General 92

8.14.3.2 Collision detection analytics subscription request 93

8.14.3.3 Collision detection analytics subscription response 93

8.14.3.4 Collision detection analytics notification 94

8.14.3.5 Ranging/SL positioning data and location information collection subscription request 94

8.14.3.6 Ranging/SL positioning data and location information collection subscription response 95

8.14.3.7 Data Notification 95

8.14.3.8 Get analytics data request 96

8.14.3.9 Get analytics data response 96

8.15 Procedure for Location-related UE Group Analytics 97

8.15.1 General 97

8.15.2 Procedure 97

8.15.2.1 Subscribe-notify model 97

8.15.2.2 Request-response model 99

8.15.3 Information flows 100

8.15.3.1 General 100

8.15.3.2 Location-related UE group analytics subscription request 100

8.15.3.3 Location-related UE group analytics subscription response 101

8.15.3.4 Location-related UE group analytics notification 101

8.15.3.5 Location information collection subscription request 102

8.15.3.6 Location information collection subscription response 103

8.15.3.7 Data Notification 103

8.15.3.8 Get analytics data request 103

8.15.3.9 Get analytics data response 104

8.16 Procedure for Application Layer AI/ML Member Capability Analytics 105

8.16.1 General 105

8.16.2 Procedure 106

8.16.2.2 Request-response model 107

8.16.3 Information flows 107

8.16.3.1 General 107

8.16.3.2 Application Layer AI/ML Member capability analytics subscription request 108

8.16.3.3 Application Layer AI/ML Member capability analytics subscription response 108

8.16.3.4 Application layer AI/ML Member capability analytics notification 108

8.16.3.5 Application Layer AI/ML Member capability data collection subscription request 109

8.16.3.6 Application Layer AI/ML Member capability data collection subscription response 110

8.16.3.7 Data Notification 110

8.16.3.8 Get analytics data request 111

8.16.3.9 Get analytics data response 111

8.17 Procedure VAL performance analytics for tethered UEs 112

8.17.1 General 112

8.17.2 Procedure 112

8.17.3 Information flows 114

8.17.3.1 General 114

8.17.3.2 Tethered VAL connectivity performance analytics subscription request 114

8.17.3.3 Tethered VAL connectivity performance analytics subscription response 115

8.18 Procedure for supporting DN Energy Efficiency analytics 115

8.18.1 General 115

8.18.2 Procedure 115

8.18.3 Information flows 117

8.18.3.1 General 117

8.18.3.2 DN energy analytics request/subscription request 117

8.18.3.3 DN energy analytics response/notification 118

8.18.3.4 Response to DN energy analytics request 119

8.19 Procedure for ML Model Performance Degradation Detection 120

8.19.1 General 120

8.19.2 Procedure 120

8.20 Procedure for monitoring ML-enabled analytics correctness 121

8.20.1 General 121

8.20.2 Procedure 121

8.20.3 Information flows 122

8.20.3.1 General 122

8.20.3.2 ADAE analytics monitoring subscription request 122

8.20.3.3 ADAE analytics monitoring subscription response 122

8.20.3.4 ADAE analytics monitoring request 123

8.20.3.5 ADAE analytics monitoring response 123

8.20.3.6 ADAE analytics monitoring notify 123

8.21 Procedure on support for energy information analytics for location services 124

8.21.1 General 124

8.21.2 Procedure 124

8.21.3 Information flows 126

8.21.3.1 General 126

8.21.3.2 Energy information analytics subscription request 126

8.21.3.3 Energy information analytics subscription response 126

8.21.3.4 Energy information analytics notification 127

8.22 AI/ML energy consumption analytics 127

8.22.1 General 127

8.22.2 Procedure 127

8.22.2.1 Subscribe-notify model 127

8.22.2.2 Request-response model 129

8.22.3 Information flows 129

8.22.3.1 General 129

8.22.3.2 AI/ML energy consumption analytics subscription request 129

8.22.3.3 AI/ML energy consumption analytics subscription response 130

8.22.3.4 AI/ML energy consumption analytics notification 130

8.22.3.5 Get AI/ML energy consumption analytics request 131

8.22.3.6 Get AI/ML energy consumption analytics response 132

8.23 ADAES Support for AIMLE client energy sustainability analytics 132

8.23.1 General 132

8.23.2 Procedure 132

8.23.3 Information flows 134

8.23.3.1 General 134

8.23.3.2 AIMLE client energy sustainability analytics request 134

8.23.3.3 AIMLE client energy sustainability analytics response 134

9 ADAE layer APIs 135

9.1 General 135

9.2 ADAE server APIs 135

9.2.1 General 135

9.2.2 ADAE server APIs 135

9.2.3 SS\_ ADAE_VAL_performance_analytics API 137

9.2.3.1 General 137

9.2.3.2 Subscribe 137

9.2.3.3 Notify 137

9.2.4 SS\_ ADAE_slice_performance_analytics API 137

9.2.4.1 General 137

9.2.4.2 Subscribe 137

9.2.4.3 Notify 137

9.2.5 SS\_ ADAE_UE-to-UE_performance_analytics API 138

9.2.5.1 General 138

9.2.5.2 Subscribe 138

9.2.5.3 Notify 138

9.2.6 SS\_ ADAE_location_accuracy_analytics API 138

9.2.6.1 General 138

9.2.6.2 Subscribe 138

9.2.6.3 Notify 138

9.2.7 SS\_ ADAE_service_API_analytics API 139

9.2.7.1 General 139

9.2.7.2 Subscribe 139

9.2.6.3 Notify 139

9.2.8 SS_ADAE_slice_usage_pattern_analytics API 139

9.2.8.1 General 139

9.2.8.2 Subscribe 139

9.2.8.3 Notify 139

9.2.9 SS\_ ADAE_edge_analytics API 140

9.2.9.1 General 140

9.2.9.2 Subscribe 140

9.2.9.3 Notify 140

9.2.9.4 Get 140

9.2.10 SS\_ ADAE_slice_usage_stats 140

9.2.10.1 General 140

9.2.10.2 Get 140

9.2.11 SS_ADAE_edge_preparation_analytics API 141

9.2.11.1 General 141

9.2.11.2 Subscribe 141

9.2.11.3 Notify 141

9.2.11.4 Get 141

9.2.12 SS_ADAE_server-to-server_performance_analytics API 141

9.2.12.1 General 141

9.2.12.2 Subscribe 141

9.2.12.3 Notify 142

9.2.13 SS_ADAE_collision_detection_analytics API 142

9.2.13.1 General 142

9.2.13.2 Subscribe 142

9.2.13.3 Notify 142

9.2.13.4 Get 142

9.2.14 SS_ADAE_location-related_UE_group_analytics API 142

9.2.14.1 General 142

9.2.14.2 Subscribe 143

9.2.14.3 Notify 143

9.2.14.4 Get 143

9.2.15 SS_ADAE_AIML_member_capability_analytics API 143

9.2.15.1 General 143

9.2.15.2 Subscribe 143

9.2.15.3 Notify 143

9.2.15.4 Get 144

9.2.16 SS\_ ADAE_ServiceExp API 144

9.2.16.1 General 144

9.2.16.2 Request 144

9.2.17 SS\_ ADAE_DN_energy_analytics API 144

9.2.17.1 General 144

9.2.17.2 Get DN_energy_analytics 144

9.2.18 SS\_ ADAE_AIML_energy_consumption_analytics API 144

9.2.18.1 General 144

9.2.18.2 Subscribe 145

9.2.18.3 Notify 145

9.2.18.4 Get 145

9.2.19 SS\_ ADAE_AIMLE_Client_Energy_Sustainability_analytics API 145

9.2.19.1 General 145

9.2.19.2 Get 145

9.2.20 SS\_ ADAE_Analytics_monitoring API 145

9.2.20.1 General 145

9.2.20.2 Subscribe 146

9.2.20.3 Notify 146

9.3 A-ADRF APIs 146

9.3.1 General 146

9.3.2 A-ADRF APIs 146

9.3.3 SS_AADRF_Data_Collection API 147

9.3.3.1 General 147

9.3.3.2 Subscribe 147

9.3.3.3 Notify 147

9.3.4 SS\_ AADRF_Historical_serviceAPI_logs API 147

9.3.4.1 General 147

9.3.4.2 Get 147

9.3.5 SS\_ AADRF_NetworkSlice_data API 147

9.3.5.1 General 147

9.3.5.2 Get 147

9.3.6 SS_AADRF_EdgeData_Collection API 148

9.3.6.1 General 148

9.3.6.2 Subscribe 148

9.3.6.3 Notify 148

9.3.7 SS_AADRF_Location_Accuracy API 148

9.3.7.1 General 148

9.3.7.2 Get 148

9.3.8 SS_AADRF_Edge_Preparation_Data API 149

9.3.8.1 General 149

9.3.8.2 Get 149

9.3.9 SS_AADRF_Data_Storage API 149

9.3.9.1 General 149

9.3.9.2 Request Subscripiton 149

9.3.9.3 Store Data 149

9.3.10 SS_AADRF_ServerToServer_Analytics API 149

9.3.10.1 General 149

9.3.10.2 Get 150

9.4 A-DCCF APIs 150

9.4.1 General 150

9.4.2 A-DCCF APIs 150

9.4.3 SS_ADCCF_Data_Collection API 150

9.4.3.1 General 150

9.4.3.2 Subscribe 150

9.4.3.3 Notify 151

9.4.3.4 Request 151

10 Analytics related to satellite access 151

10.1 General 151

10.2 Support for UE RAT connectivity analytics 151

10.2.1 General 151

10.2.2 Procedure 151

10.2.3 Information flows 153

10.2.3.1 UE RAT connectivity analytics subscription request 153

10.2.3.2 UE RAT Connectivity analytics subscription response 153

10.2.3.3 UE RAT Connectivity data retrieval request 153

10.2.3.4 UE RAT Connectivity data retrieval response 154

10.2.3.5 UE RAT Connectivity analytics notification 154

10.2.4 ADAE server APIs 155

10.2.4.1 General 155

10.2.4.2 ADAE server APIs 155

10.2.4.3 SS_ADAE_UE_RAT_connectivity_analytics API 155

10.2.4.3.1 General 155

10.2.4.3.2 Subscribe 155

10.2.4.3.3 Notify 156

10.2.5 A-ADRF APIs 156

10.2.5.1 General 156

10.2.5.2 A-ADRF APIs 156

10.2.5.3 SS_AADRF_UE RAT connectivity analytics API 156

10.2.5.3.1 General 156

10.2.5.3.2 Get 156

10.3.2 Procedure 157

10.3.3 Information flows 158

10.3.4 ADAE server APIs 159

10.3.4.1 General 159

10.3.4.2 ADAE server APIs 159

10.3.4.3 SS_ADAE_Satellite_Communication_ASACI_analytics 159

10.3.4.3.1 General 159

10.3.4.3.2 Subscribe 160

10.3.4.3.3 Notify 160

10.4 Support for QoS analytics for services over satellite access 160

10.4.1 General 160

10.4.2 Procedure 160

10.4.3 Information flows 162

10.4.3.2 Satellite communication QoS analytics subscription response 162

10.4.3.3 Satellite communication QoS analytics notification 162

10.4.4 ADAE server APIs 163

10.4.4.1 General 163

10.4.4.2 ADAE server APIs 163

10.4.4.3 SS_ADAE_Satellite_Communication_QoS_analytics 163

10.4.4.3.1 General 163

10.4.4.3.2 Subscribe 163

10.4.4.3.3 Notify 163

Annex A (informative): Deployment scenarios 163

A.1 General 163

A.2 Deployment model #1: Cloud-deployed ADAES 164

A.3 Deployment model #2 Edge-deployed ADAES 164

A.4 Deployment model #3: Coordinated ADAES deployment 165

Annex B (informative): Change history 167
