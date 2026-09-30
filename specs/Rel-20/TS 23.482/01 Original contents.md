---
spec: TS 23.482
version: 20.3.0
release: '20'
clause: contents
title: Contents
source_archive: 23482-k30.zip
source_document: 23482-k30.docx
content_origin: 3gpp-source
---

# Contents

Foreword 15

Introduction 16

1 Scope 17

2 References 17

3 Definitions of terms, symbols and abbreviations 18

3.1 Terms 18

3.2 Symbols 19

3.3 Abbreviations 19

4 Architectural requirements 19

4.1 General requirements 19

4.2 AIML capability related requirements 19

5 Application architecture for enabling AI/ML services 20

5.1 General 20

5.2 Application enablement architecture 20

5.2.1 On-Network AIML Enablement (AIMLE) Functional Architecture 20

5.2.1.1 Service-based AIMLE architecture representation 21

5.2.2 Off-Network AIMLE Functional Architecture 22

5.2.2a Enhanced AIMLE Functional Architecture for multi-operator services 22

5.2.2b AIMLE Functional Architecture for roaming scenarios 24

5.2.3 Functional Entities Description 25

5.2.3.1 General 25

5.2.3.2 AIMLE client 25

5.2.3.3 AIMLE server 25

5.2.3.4 ML repository 25

5.2.4 Reference Points Description 25

5.2.4.1 General 25

5.2.4.2 AIML-UU 25

5.2.4.3 AIML-S 26

5.2.4.4 AIML-C 26

5.2.4.5 AIML-R 26

5.2.4.6 AIML-E 26

5.2.4.7 AIML-PC5 26

6 AIMLE Functional Description 26

6.1 Support for ML model retrieval 26

6.2 Support for ML model training 26

6.3 Support for FL member registration 26

6.4 Support for FL events subscription and notification 27

6.5 Support AI/ML task transfer 27

6.6 Support for AIMLE client registration 27

6.7 Support for AIMLE client discovery 27

6.8 Support for AIMLE client selection 27

6.9 Support for AIMLE client participation 27

6.10 Support for ML model management 27

6.11 Support HFL training 28

6.12 Support AIMLE client selection subscription and notification 28

6.13 Support for Split AI/ML Operation 28

6.14 Support data management assistance 28

6.15 Support for Transfer Learning enablement 29

6.16 Support for FL member grouping 29

6.17 Support vertical federated learning 29

6.18 Support for ML model training capability evaluation 29

6.19 Support AIML service operations control and management 29

6.20 Support for ML model update 29

6.21 Support for ML model performance monitoring 29

6.22 Support for AIMLE assisted ML model selection 29

6.23 Support for AIMLE context transfer in edge data networks 30

6.24 Support for Assisting Hierarchical Computing 30

6.25 Support for ML model evaluation information management 30

6.26 Support for AIMLE server registration procedures in hierarchical AIMLE deployments 30

6.27 Support for ML model training in hierarchical AIMLE deployments 30

6.29 Support for ML model maintenance 30

6.30 Support for ML model performance evaluation with information collection from ML model consumer 31

6.31 Support for AIMLE service migration in UE roaming scenarios 31

6.32 Support for federated AIML service enablement 31

6.33 Support for AIMLE server discovery 31

6.34 Support for ML model inference 31

6.35 Support for AIML Server discovery for ML inference service 31

6.36 Support for AI inference service performance management and feedback 32

7 Identities and commonly used values 32

7.1 General 32

7.2 AIMLE server ID 32

7.3 AIMLE client ID 32

7.4 ML repository ID 32

7.5 ML model ID 32

7.6 FL member ID 32

7.7 AIMLE service area 32

7.8 ML model profile ID 32

7.9 AIMLE client set ID 33

8 Procedures and information flows 33

8.1 General 33

8.2 ML model retrieval 33

8.2.1 General 33

8.2.2 Procedure 33

8.2.2.1 General 33

8.2.2.2 ML model retrieval 33

8.2.2.3 ML model retrieval subscription 34

8.2.2.3.1 General 34

8.2.2.3.2 Subscribe 34

8.2.2.3.3 Notify 35

8.2.2.3.4 Subscription update 36

8.2.2.3.5 Unsubscribe 36

8.2.3 Information flows 37

8.2.3.1 ML model retrieval request 37

8.2.3.2 ML model retrieval response 37

8.2.3.3 ML model retrieval subscribe request 37

8.2.3.4 ML model retrieval subscription response 38

8.2.3.5 ML model retrieval notification 38

8.2.3.6 ML model retrieval subscription update request 38

8.2.3.7 ML model retrieval subscription update response 39

8.2.3.8 ML model retrieval unsubscribe request 39

8.2.3.9 ML model retrieval unsubscribe response 39

8.3 ML model training 40

8.3.1 General 40

8.3.2 Procedure for ML model training 40

8.3.3 Information flows 40

8.3.3.1 ML model training request 40

8.3.3.2 ML model training response 42

8.3.3.3 ML model training notification 42

8.4 Support for FL member registration 42

8.4.1 General 42

8.4.2 Procedure on FL member registration information storage 43

8.4.3 Procedure on FL member registration information storage update 43

8.4.3a Procedure on registration information storage update for FL member deregistration 44

8.4.3b Procedure on VAL server registration to AIMLE server 45

8.4.3c Procedure on VAL server registration update to AIMLE server 46

8.4.3d Procedure on VAL server deregistration to AIMLE server 46

8.4.3e Procedure for querying VAL server registration for the AIMLE server 47

8.4.4.1 General 47

8.4.4.2 FL member registration information storage request 47

8.4.4.3 FL member registration information storage response 49

8.4.4.4 FL member registration information storage update request 49

8.4.4.5 FL member registration update response 49

8.4.4.6 FL member deregistration request 50

8.4.4.7 FL member deregistration response 50

8.4.4.8 VAL server as FL member registration request 50

8.4.4.9 VAL server as FL member registration response 51

8.4.4.10 VAL server as FL member registration update request 52

8.4.4.11 VAL server as FL member registration update response 52

8.4.4.12 VAL server as FL member deregistration request 52

8.4.4.13 VAL server as FL member deregistration response 52

8.4.4.14 FL member registration query request 53

8.4.4.15 FL member registration query response 53

8.5 Support for FL events subscription and notification 53

8.5.1 General 53

8.5.2 Procedure on subscription for FL related events 53

8.5.3a Procedure on FL related event notification 54

8.5.3b Procedure on FL member notifying a change on renewable energy status 55

8.5.4 Definition of FL-related Events 56

8.5.5 Information flows 57

8.5.5.1 General 57

8.5.5.2 FL-related event subscription request 57

8.5.5.3 FL-related event subscription response 58

8.5.5.4 FL related event notification 59

8.5.5.5 FL-related event notification acknowledgement 59

8.6 Support AI/ML Task Transfer 60

8.6.1 General 60

8.6.2 Procedures for AI/ML task transfer 60

8.6.2.1 Request AIMLE Server to assist AI/ML task transfer 60

8.6.2.2 Request target AI/ML member for AI/ML task transfer 62

8.6.2.3 Direct AI/ML task transfer 62

8.6.2.4 AIMLE server-controlled AI/ML task transfer 63

8.6.3 Information flows 64

8.6.3.1 General 64

8.6.3.2 AI/ML task transfer assist request 64

8.6.3.3 AI/ML task transfer assist response 65

8.6.3.4 AI/ML task transfer request 65

8.6.3.5 AI/ML task transfer response 66

8.6.3.6 Direct AI/ML task transfer request 66

8.6.3.7 Direct AI/ML task transfer Response 66

8.6.3.8 AIMLE server-controlled AI/ML task transfer request 67

8.6.3.9 AIMLE server-controlled AI/ML task transfer Response 67

8.7 AIMLE client registration 67

8.7.1 General 67

8.7.2 Procedures 67

8.7.2.1 General 67

8.7.2.2 AIMLE client registration 68

8.7.2.3 AIMLE client registration update 69

8.7.2.4 AIMLE client de-registration 69

8.7.3 Information flows 70

8.7.3.1 General 70

8.7.3.2 AIMLE client registration request 70

8.7.3.3 AIMLE client registration response 74

8.7.3.4 AIMLE client registration update request 74

8.7.3.5 AIMLE client registration update response 75

8.7.3.6 AIMLE client de-registration request 75

8.7.3.7 AIMLE client de-registration response 76

8.8 AIMLE client discovery 76

8.8.1 General 76

8.8.2 Procedure 76

8.8.2.1 AIMLE client discovery 76

8.8.3 Information flows 78

8.8.3.1 AIMLE client discovery request 78

8.8.3.2 AIMLE client discovery response 81

8.9 AIMLE client selection 81

8.9.1 General 81

8.9.2 Procedure 81

8.9.2.1 AIMLE client selection 81

8.9.3 Information flows 83

8.9.3.1 AIMLE client selection request 83

8.9.3.2 AIMLE client selection response 84

8.10 AIMLE client participation 84

8.10.1 General 84

8.10.2 Procedure 85

8.10.2.1 AIMLE client participation 85

8.10.3 Information flows 85

8.10.3.1 AIMLE client participation request 85

8.10.3.2 AIMLE client participation response 85

8.11 ML model management 86

8.11.1 General 86

8.11.2 ML model information storage 86

8.11.2.1 AIMLE server-initiated ML model information storage 86

8.11.2.2 AIMLE consumer-initiated ML model information storage 87

8.11.3 ML model information discovery 87

8.11.4 Information flows 88

8.11.4.1 ML model information storage request 88

8.11.4.2 ML model information storage response 92

8.11.4.3 ML model information discovery request 92

8.11.4.4 ML model information discovery response 93

8.12 HFL training 93

8.12.1 General 93

8.12.2 Procedure 93

8.12.2.1 HFL training subscription and notification 93

8.12.2.2 HFL training subscription update 95

8.12.2.3 HFL training unsubscription 95

8.12.3 Information flows 96

8.12.3.1 HFL training subscription request 96

8.12.3.2 HFL training subscription response 96

8.12.3.3 HFL training subscription notification 97

8.13 AIMLE client selection subscription and notification 97

8.13.1 General 97

8.13.2 Procedures 97

8.13.2.1 General 97

8.13.2.2 AIMLE client selection subscription and notification 98

8.13.2.3 AIMLE client selection subscription update 99

8.13.2.4 AIMLE client selection unsubscribe 100

8.13.3 Information flows 100

8.13.3.1 General 100

8.13.3.2 AIMLE client selection subscription request 100

8.13.3.3 AIMLE client selection subscription response 101

8.13.3.4 AIMLE client selection update notification 101

8.13.3.5 AIMLE client subscription update request 101

8.13.3.6 AIMLE client selection subscription update response 102

8.13.3.7 AIMLE client selection unsubscribe request 102

8.13.3.8 AIMLE client selection unsubscribe response 102

8.14 Support for Split AI/ML Operation 103

8.14.1 General 103

8.14.2 Procedure 103

8.14.2.1 General 103

8.14.2.2 Split operation pipeline discovery 104

8.14.2.3 Split operation pipeline creation 104

8.14.2.4 Split operation node registration 105

8.14.2.4.1 General 105

8.14.2.4.2 Split operation node registration 105

8.14.2.4.3 Split operation node registration update 106

8.14.2.4.4 Split operation node de-registration 107

8.14.2.5 Split operation event subscription 107

8.14.2.5.1 General 107

8.14.2.5.2 Subscribe 107

8.14.2.5.3 Notify 109

8.14.2.5.4 Subscription update 109

8.14.2.5.5 Unsubscribe 110

8.14.2.6 Split operation pipeline update 110

8.14.2.7 Split operation pipeline delete 111

8.14.2.8 Split operation pipeline join 112

8.14.2.9 Energy-aware split operation pipeline update 113

8.14.3 Information flows 114

8.14.3.1 General 114

8.14.3.2 Split operation pipeline discovery request 114

8.14.3.3 Split operation pipeline discovery response 115

8.14.3.4 Split operation pipeline create request 116

8.14.3.5 Split operation pipeline create response 117

8.14.3.6 Split operation node register request 117

8.14.3.7 Split operation node register response 118

8.14.3.8 Split operation node registration update request 118

8.14.3.9 Split operation node registration update response 119

8.14.3.10 Split operation node de-register request 119

8.14.3.11 Split operation node de-register response 119

8.14.3.12 Split operation subscribe request 119

8.14.3.13 Split operation subscription response 120

8.14.3.14 Split operation notification 120

8.14.3.15 Split operation subscribe update request 121

8.14.3.16 Split operation subscribe update response 121

8.14.3.17 Split operation unsubscribe request 122

8.14.3.18 Split operation unsubscribe response 122

8.14.3.19 Split operation pipeline update request 122

8.14.3.20 Split operation pipeline update response 123

8.14.3.21 Split operation pipeline delete request 123

8.14.3.22 Split operation pipeline delete response 123

8.14.3.23 Split operation pipeline join request 123

8.14.3.24 Split operation pipeline join response 124

8.14.3.25 Energy credit/usage report 124

8.14.3.26 Energy-aware split operation pipeline update trigger request 124

8.14.3.27 Energy-aware split operation pipeline update trigger response 125

8.15 AIMLE data management assistance 125

8.15.1 General 125

8.15.2 Procedure 125

8.15.3 Information flows 127

8.15.3.1 AIMLE data management assistance subscription request 127

8.15.3.2 AIMLE data management assistance subscription response 128

8.15.3.3 AIMLE data management assistance notify 128

8.15.3.4 Client data processing trigger request 129

8.15.3.5 Client data processing trigger response 129

8.16 Support for Transfer Learning enablement 130

8.16.1 General 130

8.16.2 Procedure for server-triggered transfer learning enablement 130

8.16.3 Procedure for client-triggered transfer learning enablement 131

8.16.4 Information flows 132

8.16.4.1 Transfer learning model selection assistance request 132

8.16.4.2 Transfer learning model selection assistance response 132

8.16.4.3 UE transfer learning model selection assistance request 133

8.16.4.4 UE transfer learning model selection assistance response 133

8.17 Support for FL member grouping 134

8.17.1 General 134

8.17.2 Procedure 134

8.17.3 Information flows 136

8.17.3.1 FL member grouping support request 136

8.17.3.2 FL member grouping support response 137

8.17.3.3 FL grouping indication 138

8.17.3.4 FL grouping indication acknowledge 139

8.18 Support Vertical FL 139

8.18.1 General 139

8.18.2 Procedure for supporting VFL 139

8.18.2b Procedure for sample alignment and VFL member initiation 142

8.18.3 Information flows 143

8.18.3.1 VFL task initiation request 143

8.18.3.2 VFL client training/inference initiation request 144

8.18.3.3 VFL client training/inference initiation response 144

8.18.3.4 VFL task initiation response 144

8.18.3.5 Client feature selection request 145

8.18.3.6 Client feature selection response 145

8.18.3.7 Sample binding request 146

8.18.3.8 Sample binding response 146

8.19 ML Model Training Capability Evaluation 146

8.19.1 General 146

8.19.2 Procedure for ML model training capability evaluation 146

8.19.3 Information flows 147

8.19.3.1 ML model training capability evaluation request 147

8.19.3.2 ML model training capability evaluation response 148

8.20 AIML service operations control and management procedure 148

8.20.1 General 148

8.20.2 AIML service operations control and management procedure 148

8.20.3 Information flows 149

8.20.3.1 AIML service operations control and management request 149

8.20.3.2 AIML service operations control and management response 150

8.20.3.3 AIML Enablement client service operation request 150

8.20.3.4 AIML Enablement client service operation response 150

8.21 ML model update 151

8.21.1 General 151

8.21.2 Procedure 151

8.21.3 Information flows 152

8.21.3.1 ML model update request 152

8.21.3.1 ML model update response 152

8.22 ML model performance monitoring 152

8.22.1 General 152

8.22.2 Procedure 153

8.22.3 Information flows 154

8.22.3.1 ML model performance monitoring subscription request 154

8.22.3.2 ML model performance monitoring subscription response 155

8.22.3.3 ML model performance monitoring notify 155

8.23 AIMLE assisted ML model selection 155

8.23.1 General 155

8.23.2 Procedure 155

8.23.3 Information flows 157

8.23.3.1 AIMLE assisted ML model selection subscription request 157

8.23.3.2 ML model selection subscription response 157

8.23.3.3 ML model selection notification 158

8.24 AIMLE context transfer 158

8.24.1 General 158

8.24.2 Procedure 158

8.24.3 Information flows 159

8.24.3.1 AIMLE context transfer request 159

8.24.3.2 AIMLE context transfer response 160

8.25 Support of AIML Services for Assisting Hierarchical Computing 161

8.25.1 General 161

8.25.2 Procedure for assisting hierarchical computing process 161

8.25.3 Information flows 163

8.25.3.1 Hiearchitical computing assistance request 163

8.25.3.2 Assist hierarchical computing response 164

8.26 ML model evaluation information management 164

8.26.1 General 164

8.26.2 Procedure 164

8.26.2.1 General 164

8.26.2.2 ML model evaluation information storage 164

8.26.2.3 ML model evaluation information retrieval 165

8.26.2.4 ML model evaluation information removal 166

8.26.2.5 ML model evaluation information retrieval subscription 167

8.26.2.5.1 General 167

8.26.2.5.2 Subscribe 167

8.26.2.5.3 Notify 168

8.26.2.5.4 Update 168

8.26.2.5.5 Unsubscribe 169

8.26.3 Information flows 170

8.26.3.7 ML model evaluation information retrieval subscribe request 173

8.26.3.8 ML model evaluation information retrieval subscribe response 173

8.26.3.9 ML model evaluation information retrieval notification 173

8.26.3.10 ML model evaluation information retrieval subscription update request 174

8.26.3.11 ML model evaluation information retrieval subscription update response 174

8.26.3.12 ML model evaluation information retrieval unsubscribe request 174

8.26.3.13 ML model evaluation information retrieval unsubscribe response 175

8.27 AIMLE server registration in hierarchical AIMLE deployments 175

8.27.1 General 175

8.27.2 Procedures for supporting hierarchical AIMLE deployments 175

8.27.2.1 AIMLE server registration 175

8.27.2.2 AIMLE server registration update 176

8.27.2.3 AIMLE server de-registration 177

8.27.3 Information flows 178

8.27.3.1 AIMLE server registration request 178

8.27.3.2 AIMLE server registration response 178

8.27.3.3 AIMLE server registration update request 179

8.27.3.4 AIMLE server registration update response 179

8.27.3.5 AIMLE server de-registration request 180

8.27.3.6 AIMLE server de-registration response 180

8.28 ML model training in hierarchical AIMLE deployments 180

8.28.1 General 180

8.28.2 Procedure for Hierarchical ML model training 180

8.28.3 Information flows 182

8.28.3.1 Hierarchical ML model training subscription request 182

8.28.3.2 Hierarchical ML model training subscription response 182

8.28.3.3 Hierarchical ML model training notification 182

8.29 Intermediate Node discovery in ML model split learning operations 183

8.29.1 General 183

8.29.2 Procedure 183

8.29.3 Information flows 185

8.29.3.1 Intermediate node discovery request 185

8.29.3.2 Intermediate node discovery response 185

8.29.3.3 Intermediate node selection request 186

8.29.3.4 Intermediate node selection response 186

8.30 ML Model Maintenance 186

8.30.1 General 186

8.30.2 Procedure 187

8.30.3 Information flows 188

8.30.3.1 ML model maintenance subscription request 188

8.30.3.2 ML model maintenance subscription response 188

8.30.3.3 ML model maintenance notification 188

8.31 ML Model Performance Evaluation with Information Collection from ML Model Consumer 189

8.31.1 General 189

8.31.2 ML model performance evaluation 189

8.31.3 Information flows 190

8.31.3.1 ML model performance evaluation subscription request 190

8.31.3.2 ML model performance evaluation subscription response 190

8.31.3.3 ML model performance evaluation notification 190

8.32 AIMLE service migration in UE roaming scenarios 191

8.32.1 General 191

8.32.2 Procedure 191

8.32.3 Information flows 192

8.32.3.1 Cross-PLMN AIMLE server discovery request 192

8.32.3.2 Cross-PLMN AIMLE server discovery response 193

8.32.3.3 AIMLE server change trigger request 193

8.32.3.4 AIMLE server change trigger response 194

8.32.3.5 AIMLE server change trigger complete 194

8.32.3.6 AIMLE server change notification 195

8.33 Federated AIML service enablement 195

8.33.1 General 195

8.33.2 Procedure 195

8.33.3 Information flows 196

8.33.3.1 Federated AIML service config request 196

8.33.3.2 Federated AIML service config response 197

8.34 AIMLE server discovery 197

8.34.1 General 197

8.34.2 Procedure for AIMLE server discovery 198

8.34.3 Information flows 199

8.34.3.1 AIMLE server discovery request 199

8.34.3.2 AIMLE server discovery response 199

8.35 ML model collaborative inference 200

8.35.1 General 200

8.35.2 Procedure for ML model collaborative inference 200

8.35.3 Information flows 202

8.35.3.1 ML model collaborative inference request 202

8.35.3.2 ML model collaborative inference response 202

8.35.3.3 Intermediate inference output 202

8.35.3.4 ML Model collaborative inference result response 203

8.36 ML model inference 203

8.36.1 General 203

8.36.2 Procedure for ML model inference 203

8.36.3 Information flows 204

8.36.3.1 ML model inference request 204

8.36.3.2 ML model inference response 205

8.37 ML model inference in a hierarchical AIMLE deployment 206

8.37.1 General 206

8.37.2 Procedure for ML model inference in a hierarchical AIMLE deployment 206

8.37.3 Information flows 207

8.37.3.1 Edge ML model inference request 207

8.37.3.2 Edge ML model inference response 208

8.38 ML model inference service performance management and feedback 209

8.38.1 General 209

8.38.2 Procedure for ML model inference service performance management and feedback 209

8.38.3 Information flows 211

8.38.3.1 ML model inference service performance feedback subscription request 211

8.38.3.2 ML model inference service performance feedback subscription response 211

8.38.3.3 ML model inference service performance feedback notification 212

8.38.3.4 ML model inference service performance feedback notify acknowledgement 212

9 AIMLE APIs 213

9.1 General 213

9.2 AIMLE server APIs 213

9.2.1 AIMLE AI/ML Task Transfer API 213

9.2.1.1 General 213

9.2.1.2 Aimles_AIMLTaskTransferAssist_Request operation 214

9.2.1.2.1 General 214

9.2.1.2.2 AIML task transfer assist request operation 214

9.2.1.3 Aimles_AIMLESControlledAIMLTaskTransfer_Request operation 214

9.2.1.3.1 General 214

9.2.1.3.2 AIMLE server-controlled AIML task transfer request operation 214

9.2.2 ML model retrieval API 214

9.2.2.1 General 214

9.2.2.2 Aimles_MLModelRetrieval_Request operation 215

9.2.2.3 Aimles_MLModelRetrieval_Subscribe operation 215

9.2.2.4 Aimles_MLModelRetrieval_Notify operation 215

9.2.2.5 Aimles_MLModelRetrieval_UpdateSubscription operation 215

9.2.2.6 Aimles_MLModelRetrieval_Unsubscribe operation 215

9.2.3 ML model training API 216

9.2.3.1 General 216

9.2.3.2 Aimles_MLModelTraining_Request operation 216

9.2.4 AIMLE TL model selection assistance API 216

9.2.4.1 General 216

9.2.4.2 Aimles_TLModelSelectionAssistance_Request operation 216

9.2.5 FL member grouping support API 217

9.2.5.1 General 217

9.2.5.2 Aimles_FLMemberGroupSupport_Request operation 217

9.2.6 AIMLE client Discovery API 217

9.2.6.1 General 217

9.2.6.2 AIMLE client discovery request operation 217

9.2.7 AIMLE client Selection API 218

9.2.7.1 General 218

9.2.7.2 AIMLE client selection request operation 218

9.2.8 AIMLE client Selection Subscribe API 218

9.2.8.1 General 218

9.2.8.2 Aimles_ClientSelection_subscribe operation 218

9.2.8.3 Aimles_ClientSelection_notify operation 219

9.2.8.4 Aimles_ClientSelection_update operation 219

9.2.8.5 Aimles_ClientSelection_unsubscribe operation 219

9.2.9 AIMLE Data Management API 219

9.2.9.1 General 219

9.2.9.2 Aimles_DataManagement_Subscribe operation 219

9.2.9.3 Aimles_DataManagement_Notify operation 220

9.2.10 AIMLE Service Operations Management API 220

9.2.10.1 General 220

9.2.10.2 Aimles_AIMLEServiceOperationsManagement_Request operation 220

9.2.11 AIMLE Client Registration APIs 220

9.2.11.1 General 220

9.2.11.2 AIMLE client registration request operation 220

9.2.11.3 AIMLE client registration update request operation 221

9.2.11.4 AIMLE client registration delete request operation 221

9.2.12 Split AI/ML Operation API 221

9.2.12.1 General 221

9.2.12.2 Aimles_SplitOpPipeline_Discover operation 221

9.2.12.3 Aimles_SplitOpPipeline_Create operation 222

9.2.12.4 Aimles_SplitOpPipeline_Update operation 222

9.2.12.5 Aimles_SplitOpPipeline_Delete operation 222

9.2.12.6 Aimles_SplitOpNodeRegistration_Request operation 222

9.2.12.7 Aimles_SplitOpNodeRegistration_Update operation 222

9.2.12.8 Aimles_SplitOpNodeRegistration_Deregister operation 223

9.2.12.9 Aimles_SplitOpEvent_Subscribe operation 223

9.2.12.10 Aimles_SplitOpEvent_Notify operation 223

9.2.12.11 Aimles_SplitOpEvent_UpdateSubscription operation 223

9.2.12.12 Aimles_SplitOpEvent_Unsubscribe operation 223

9.2.13 ML model update API 224

9.2.13.1 General 224

9.2.13.2 Aimles_MLModelUpdate_Request operation 224

9.2.14 ML model performance monitoring API 224

9.2.14.1 General 224

9.2.14.2 Subscribe 224

9.2.14.3 Notify 225

9.2.15 AIMLE assisted ML model selection API 225

9.2.15.1 General 225

9.2.15.2 Aimles_AssistedMLModelSelection_Subscribe operation 225

9.2.15.3 Aimles_AssistedMLModelSelection_Notify operation 225

9.2.16 AIMLE context transfer API 225

9.2.16.1 General 225

9.2.16.2 Aimles_ContextTransfer_Request operation 226

9.2.17 AIMLE Assistance of Hierarchical Computing API 226

9.2.17.1 General 226

9.2.17.2 Aimles_HierarchicalComputingAssist_Request operation 226

9.2.18.8 Aimles_MLModelEvalInfoRetrieval_Unsubscribe operation 228

9.2.18.9 Aimles_FLMember_Registration_Query operation 228

9.2.19 AIMLE server registration API 228

9.2.19.1 General 228

9.2.19.2 Aimles_HierarchicalServer_Registration operation 229

9.2.19.3 Aimles_HierarchicalServer_UpdateRegistration operation 229

9.2.19.4 Aimles_HierarchicalServer_De-Registration operation 229

9.2.20 Hierarchical ML model training API 229

9.2.20.1 General 229

9.2.20.2 Aimles_HierarchicalMLModelTraining_Subscribe operation 229

9.2.20.3 Aimles_HierarchicalMLModelTraining_Notify operation 230

9.2.21 VAL Server FL Member API 230

9.2.21.1 General 230

9.2.21.2 Aimles_FLMember_Registration operation 230

9.2.21.3 Aimles_FLMember_Registration_Update operation 230

9.2.21.4 Aimles_FLMember_Deregistration operation 230

9.2.22 ML model maintenance API 231

9.2.22.1 General 231

9.2.22.2 Subscribe 231

9.2.22.3 Notify 231

9.2.23 ML model performance evaluation API 231

9.2.23.1 General 231

9.2.23.2 Subscribe 231

9.2.23.3 Notify 232

9.2.24 AIMLE server discovery API 232

9.2.24.1 General 232

9.2.24.2 Aimles_ServerDiscovery_Request operation 232

9.2.25 Intermediate node discovery and selection API 232

9.2.25.1 General 232

9.2.25.2 Aimles_IntermediateNodeDiscovery operation 233

9.2.25.3 Aimles_IntermediateNodeSelection operation 233

9.2.26 Federated AIML service config API 233

9.2.26.1 General 233

9.2.26.2 Aimles_FederatedAIMLServiceConfig 233

9.2.27 AIMLE Server Change Notify API 234

9.2.27.1 General 234

9.2.27.2 Aimles_AIMLEServerChangeNotify 234

9.2.28 ML model inference service performance feedback API 234

9.2.28.1 General 234

9.2.28.2 Aimles_MLModelInferenceFeedback_Subscribe operation 234

9.2.28.3 Aimles_MLModelInferenceFeedback_Notify operation 234

9.2.29 Edge ML model inference API 235

9.2.29.1 General 235

9.2.29.2 Aimles_EdgeMLModelInference_Request operation 235

9.2.30 ML model inference API 235

9.2.30.1 General 235

9.2.30.2 Aimles_MLModelInference_Request operation 235

9.3 ML repository APIs 236

9.3.1 MLR FL member Registration API 236

9.3.1.1 General 236

9.3.1.2 MLR_FLMemberRegistration_Request operation 236

9.3.1.3 MLR_FLMemberRegistrationUpdate_Request operation 236

9.3.1.4 MLR_FLMemberRegistrationFetch_Request operation 236

9.3.1.5 MLR_FLMemberDeregistration_Request operation 236

9.3.2 MLR FL Event API 237

9.3.2.1 General 237

9.3.2.2 Subscribe 237

9.3.2.3 Notify 237

9.3.3 MLR model management API 237

9.3.3.1 General 237

9.3.3.2 MLR_ModelInformationStorage_Request operation 237

9.3.3.3 MLR_ModelInformationDiscovery_Request operation 238

9.3.4.5 MLR_ModelEvalInfoRetrieval_Subscribe operation 239

9.3.4.6 MLR_ModelEvalInfoRetrieval_Notify operation 239

9.3.4.7 MLR_ModelEvalInfoRetrieval_UpdateSubscription operation 239

9.3.4.8 MLR_ModelEvalInfoRetrieval_Unsubscribe operation 239

9.3.5 MLR Cross-PlmnAIMLE server discovery API 240

9.3.5.1 General 240

9.3.5.2 MLR\_ Cross-PlmnAIMLE server_discovery operation 240

9.4 AIMLE client APIs 240

9.4.1 ML model training capability evaluation API 240

9.4.1.1 General 240

9.4.1.2 Aimlec_MLModelTrainingCapabilityEva_Request operation 240

9.4.1.3 Aimlec_MLModelTrainingCapabilityEva_Response operation 240

9.4.2 HFL training API 241

9.4.2.1 General 241

9.4.2.2 Aimlec_HFLTraining_Subscribe operation 241

9.4.2.3 Aimlec_HFLTraining_Notify operation 241

9.4.3 Client data processing API 241

9.4.3.1 General 241

9.4.3.2 Aimlec_ClientDataProcessing_Request operation 241

9.4.4 AIMLE Client Service Operations API 242

9.4.4.1 General 242

9.4.4.2 Aimlec\_ AIMLEClientServiceOperations_Request operation 242

9.4.4.3 Aimlec\_ AIMLEClientServiceOperations_Response operation 242

9.4.5 AIMLE Client Participation API 242

9.4.5.1 General 242

9.4.5.2 Aimlec_AIMLEClientParticipation_Request operation 243

9.4.6 AIMLE AI/ML Task Transfer APIs 243

9.4.6.1 General 243

9.4.6.2 Aimlec_AIMLTaskTransfer_Request operation 243

9.4.6.2.1 General 243

9.4.6.2.2 AIML task transfer request operation 243

9.4.6.3 Aimlec_DirectAIMLTaskTransfer_Request operation 243

9.4.6.3.1 General 243

9.4.6.3.2 Direct AIML task transfer request operation 244

9.4.7 FL grouping indication API 244

9.4.7.1 General 244

9.4.7.2 Aimlec_FLGroupIndication operation 244

9.4.8 AIMLE Server Change Trigger API 244

9.4.8.1 General 244

9.4.8.2 Aimlec\_ AIMLEServerChangeTrigger_Request operation 244

9.4.8.3 Aimlec\_ AIMLEServerChangeTrigger_Complete operation 245

9.4.9 ML model collaborative inference API 245

9.4.9.1 General 245

9.4.9.2 Aimlec_MLModelCollaborativeInference_Request operation 245

9.4.9.3 Aimlec_MLModelCollaborativeInference_SendIntermediateOutput operation 245

Annex A (informative): Deployment scenarios 246

A.1 General 246

A.2 Deployment model #1: Cloud-deployed AIMLE server 246

A.3 Deployment model #2: Edge-deployed AIMLE server 246

A.4 Deployment model #3: Hierarchical AIMLE server deployment 247

A.5 Deployment model #4: Centralized AIMLE server in multi-operator scenarios 248

A.6 Deployment model #5: Distributed AIMLE server in multi-operator scenarios 249

Annex B (informative): Business Scenarios 251

B.1 Business Relationship for federation and roaming 252

Annex C (informative): Role of AIMLE in ML Model Lifecycle 253

C.1 General 253

C.2 Role#1 of AIMLE in ML Model Lifecycle 253

C.3 Role#2 of AIMLE in ML Model Lifecycle 254

C.4 Role#3 of AIMLE in ML Model Lifecycle 254

Annex D: Change history 256
