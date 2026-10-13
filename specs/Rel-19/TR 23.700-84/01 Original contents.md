---
spec: TR 23.700-84
version: 19.0.0
release: '19'
clause: contents
title: Contents
source_archive: 23700-84-j00.zip
source_document: 23700-84-j00.docx
content_origin: 3gpp-source
---

# Contents

Foreword 8

1 Scope 10

2 References 10

3 Definitions of terms and abbreviations 11

3.1 Terms 11

3.2 Abbreviations 11

4 Architectural Assumptions and Requirements 11

5 Use Cases, Motivations and Key Issues 12

5.1 Use Cases 12

5.1.0 Guidelines 12

5.1.1 Use Case \#1: NWDAF-assisted QoS enhancement 12

5.1.2 Use Case \#2: Enhancements to QoS Determination with NWDAF Assistance 12

5.1.3 Use Case \#3: NWDAF assistance in device signalling storm prevention and mitigation 13

5.1.4 Use Case \#4: Motivation and Support for VFL in 5GC 13

5.1.5 Use Case \#5: NWDAF support for observed service experience analytics based on VFL 14

5.1.6 Use Case \#6: Analytics-assisted prevention of abnormal NF behaviour causing signalling storm and mitigation of its impact in the network 14

5.2 Key Issues 15

5.2.0 Mapping of Key Issues to Use Cases 15

5.2.1 Key Issue \#1: Enhancements to LCS to support Direct AI/ML based Positioning 15

5.2.2 Key Issue \#2: 5GC Support for Vertical Federated Learning 15

5.2.3 Key Issue \#3: NWDAF-assisted policy control and QoS enhancement 16

5.2.4 Key Issue \#4: NWDAF enhancements to support network abnormal behaviours (i.e. signalling storm) mitigation and prevention 16

6 Solutions 17

6.0 Mapping of Solutions to Key Issues 17

6.1 Solution \#1: Direct AI/ML based Positioning for case 2b/3b 17

6.1.1 Description 17

6.1.2 Procedures 19

6.1.2.1 Direct AI/ML based Positioning in LMF 19

6.1.2.2 Training procedure for Direct AI/ML positioning model 20

6.1.2.3 Procedure for ML model training for Direct AI/ML based positioning and data collection 21

6.1.3 Impacts on services, entities and interfaces 23

6.2 Solution \#2: Support for AI/ML Direct Positioning Training, Inference and Data Collection with LMF-side models 23

6.2.1 Description 23

6.2.2 Procedures 24

6.2.2.1 Training procedure for AI/ML Direct Positioning with LMF-side models 24

6.2.2.2 Inference procedure for AI/ML Direct Positioning with LMF-side models 25

6.2.3 Impacts on services, entities and interfaces 26

6.3 Solution \#3: Training of the AI/ML positioning model 26

6.3.1 Functional Description 26

6.3.1.1 Input of model training 26

6.3.2 Procedures 26

6.3.3 Impacts on existing services, entities and interfaces 27

6.4 Solution 4: Data Collection Framework for Direct AI/ML positioning 27

6.4.1 Description 27

6.4.2 Procedures 29

6.4.3 Impacts on services, entities and interfaces 30

6.5 Solution \#5: LMF selection to support the LMF-sided direct AI/ML positioning 30

6.5.1 Description 30

6.5.2 Procedures 31

6.5.3 Impacts on services, entities and interfaces 31

6.6 Solution \#6: LMF based ML model training and Inference 31

6.6.1 Description 31

6.6.2 Procedures 31

6.6.3 Impacts on services, entities and interfaces 32

6.7 Solution \#7 Training of LMF-side Model to determine location 32

6.7.1 Description 32

6.7.2 Procedures 34

6.7.2.1 Training an LMF-side model for NG-RAN assisted positioning 34

6.7.2.2 Training an LMF-side model for UE assisted positioning 36

6.7.2.3 Selecting UEs for reporting locations based on PEI 37

6.7.2.4 Inference of location for NG-RAN assisted positioning 38

6.7.2.5 Inference of location for UE assisted positioning 39

6.7.2.6 Monitoring accuracy of model for NG-RAN assisted location measurements 40

6.7.3 Impacts on services, entities and interfaces 40

6.8 Solution \#8: MTLF-based model performance monitoring for AI/ML positioning 42

6.8.1 Description 42

6.8.2 Procedures 42

6.8.3 Impacts on services, entities and interfaces 43

6.9 Solution \#9: new solution for KI#1 support monitoring the performance of AI model 43

6.9.1 Description 43

6.9.2 Procedures 44

6.9.2.1 Performance of AI/ML model monitoring based on PRU 44

6.9.2.2 Procedure of PRU information collection for AI/ML model training 45

6.9.3 Impacts on services, entities and interfaces 46

6.10 Solution \#10: Direct AI/ML based positioning with NWDAF assistance 46

6.10.1 Description 46

6.10.1.1 General 46

6.10.1.2 Input data 47

6.10.2 Procedures 49

6.10.3 Impacts on services, entities and interfaces 50

6.11 Solution \#11: Data collection procedure for LMF-side model training 51

6.11.1 Functional Description 51

6.11.2 Procedures 51

6.11.3 Impacts on existing services, entities and interfaces 52

6.12 Solution \#12: new solution for KI#1 support the data collection for AI model training based on authorization 52

6.12.1 Description 52

6.12.2 Procedures 53

6.12.3 Impacts on services, entities and interfaces 53

6.13 Solution \#13: Sample/Feature alignment and general training procedure for the VFL 54

6.13.1 Description 54

6.13.2 Procedures 55

6.13.2.1 Sample and feature alignment procedure 55

6.13.2.2 General training procedure for the VFL between the VFL Active Participant(VFL server) (i.e. AF) and VFL Passive Participant(VFL client) (i.e. NWDAF) 58

6.13.2.3 Inferencing procedure for the VFL between the VFL Active Participant (i.e. AF) and VFL Passive Participant (i.e. NWDAF(s)) 60

6.13.3 Impacts on services, entities and interfaces 61

6.14 Solution \#14: General procedure for NWDAF initiated Vertical Federated Learning between NWDAF(s) and AF(s) 61

6.14.1 Description 61

6.14.2 Procedures 62

6.14.2.1 Training Procedure 62

6.14.2.2 Inference Procedure 64

6.14.3 Impacts on services, entities and interfaces 65

6.14.3.1 Training Impact 65

6.14.3.2 Inference Impact 65

6.15 Solution 15: Support for vertical federated learning: Model Training and Inference 66

6.15.1 Description 66

6.15.2 Procedures 66

6.15.2.1 Sample/Feature alignment 66

6.15.2.2 VFL training procedure 67

6.15.2.3 Distributed Inference 69

6.15.3 Impacts on services, entities and interfaces 71

6.16 Solution \#16: Support for VFL with NWDAF and AF as Participants 71

6.16.1 Description 71

6.16.2 Procedures 71

6.16.3 Impacts on services, entities and interfaces 75

6.17 Solution \#17: NF discovery and selection for VFL 75

6.17.1 Description 75

6.17.2 Procedures 76

6.17.2.1 External AF VFL capability report 76

6.17.2.2 NWDAF VFL capability report 76

6.17.2.3 Client selection process of VFL 77

6.17.3 Impacts on existing services, entities and interfaces 78

6.18 Solution \#18: Vertical Federated Learning between NWDAF and AF 78

6.18.1 Description 78

6.18.2 Procedures 80

6.18.2.1 Discovery and selection of VFL clients 80

6.18.2.1.1 Discovery and selection of AF(s) if NWDAF as the VFL server 80

6.18.2.1.2 Discovery and selection of NWDAF(s) if AF as the VFL server 81

6.18.2.2 VFL training procedure between NWDAF and AF 81

6.18.2.2.1 VFL training procedure if NWDAF acts as the VFL server 82

6.18.2.2.2 VFL training procedure if AF acts as the VFL server 84

6.18.2.3 VFL training procedure among NWDAFs 85

6.18.2.4 VFL inference procedure between NWDAF and AF 88

6.18.2.4.1 VFL inference procedure if NWDAF acts as the VFL server 88

6.18.2.3.2 VFL inference procedure if AF acts as the coordinator 89

6.18.2.5 VFL inference procedure among NWDAFs 90

6.18.3 Impacts on services, entities and interfaces 91

6.19 Solution \#19: VFL inference procedure between NWDAF and AF 91

6.19.1 Description 91

6.19.2 Procedures 92

6.19.3 Impacts on Existing Nodes and Functionality 92

6.20 Solution \#20: Inference procedure for the Vertical Federated Learning between NWDAF(s) and AF(s) 93

6.20.1 Description 93

6.20.2 Procedures 94

6.20.2.1 Inference Procedure Initiated by the NWDAF 94

6.20.3 Impacts to Services, Entities and Interfaces 95

6.21 Solution \#21: Vertical Federated Learning for support of Application Layer QoE 96

6.21.1 Key Issue mapping 96

6.21.2 Description 96

6.21.2.1 VFL for Application QoE provisioning 96

6.21.2.2 VFL for Network parameters provisioning 97

6.21.3 Procedures 99

6.21.3.1 Application provisioning for customized QoE 99

6.21.3.2 Network parameters provisioning 101

6.21.4 Impacts on services, entities and interfaces 102

6.22 Solution \#22: Vertical Federated learning considering internal NWDAF architecture 103

6.22.1 Description 103

6.22.2 Procedures 105

6.22.2.1 VFL model training and inference involving MTLF, AnLF, ADRF and AF 105

6.22.2.2 VFL sample alignment for UE samples 108

6.22.3 Impacts on services, entities and interfaces 109

6.23 Solution \#23: Cross domain VFL involving NWDAF and AF 110

6.23.1 Description 110

6.23.1.1 Terminology 110

6.23.2 Procedures 112

6.23.2.1 NWDAF acts as FL Server with VFL capabilities 112

6.23.2.2 AF acts as FL Server with VFL capabilities 114

6.23.2.3 VFL capabilities within NF profile 114

6.23.2.4 Training procedure 115

6.23.2.4.1 NWDAF as FL Server with VFL capabilities 115

6.23.2.4.2 AF as FL Server with VFL capabilities 117

6.23.2.5 Inference procedure 119

6.23.2.5.1 NWDAF(AnLF) triggers the inference procedure 119

6.23.2.5.2 AF triggers the inference procedure 120

6.23.3 Impacts on services, entities and interfaces 121

6.24 Solution \#24: How to support Vertical Federated Learning between NWDAF and AF 121

6.24.1 Terminology 121

6.24.1 Description 122

6.24.2 Procedures 122

6.24.2.1 Procedure of NF discovery and selection 122

6.24.3 Impacts on services, entities and interfaces 124

6.25 Solution \#25: NF Registration and Discovery Enhancement for VFL 125

6.25.1 Description 125

6.25.2 Procedures 126

6.25.2.1 Registration and discovery of NWDAF and AF for VFL 126

6.25.2.1.1 NWDAF-initiated registration and discovery of AF for VFL in Scenario 1 126

6.25.2.1.2 AF-initiated registration and discovery of NWDAF for VFL in Scenario 2 127

6.25.3 Impacts on Existing Nodes and Functionality 128

6.26 Solution \#26: NWDAF-assisted policy control with Recommendation logical function 128

6.26.1 Description 128

6.26.1.1 Recommendation logical function 128

6.26.1.2 Policy control enhancement with the assistance of NWDAF (ReLF) 130

6.26.2 Procedures 131

6.26.3 Impacts to Services, Entities and Interfaces 132

6.27 Solution \#27: NWDAF assisted QoS policy generation 132

6.27.1 Description 132

6.27.2 Procedures 133

6.27.3 Impacts on services, entities and interfaces 134

6.28 Solution \#28: QoS Flow Analytics 134

6.28.1 Description 134

6.28.2 Procedures 135

6.28.3 Impacts on existing services, entities and interfaces 139

6.29 Solution \#29: How to evaluate NWDAF-assisted policy control and QoS enhancement 139

6.29.1 Description 139

6.29.2 Procedures 140

6.29.3 Impacts on services, entities and interfaces 141

6.30 Solution \#30: NWDAF-assisted PDU Set assistance information 142

6.30.1 Description 142

6.30.2 Procedures 143

6.30.3 Impacts on services, entities and interfaces 143

6.31 Solution \#31: PCF as RL Agent and NWDAF as Interpreter 143

6.31.1 Key Issue mapping 143

6.31.2 Description 143

6.31.3 Procedures 146

6.31.3.1 PCF as RL Agent 146

6.31.4 Impacts on services, entities and interfaces 147

6.32 Solution \#32: NWDAF-assisted optimized QoS policies determination 147

6.32.1 Description 147

6.32.2 Procedures 148

6.32.2.1 Procedure for optimized QoS determination based on combined Analytics 148

6.32.3 Impacts on services, entities, and interfaces 150

6.33 Solution \#33: QoS/policy enhancements assisted by NWDAF 150

6.33.1 Description 150

6.33.2 Procedures 153

6.33.2.1 Procedures for NWDAF assisted QoS and policy determination 153

6.33.2.2 Procedures to deploy NWDAF assisted QoS and policy determination during PDU session establishment 154

6.33.3 Impacts on services, entities and interfaces 155

6.34 Solution \#34: BDT Policy Recommendations 155

6.34.1 Description 155

6.34.2 Procedures 156

6.34.2.1 General Procedure 156

6.34.2.2 Input Data 157

6.34.2.3 Output Data 157

6.34.3 Impacts on services, entities and interfaces 158

6.35 Solution \#35: NWDAF-assisted Network Abnormal Behaviour Mitigation and Prevention 158

6.35.1 Description 158

6.35.2 Procedures 159

6.35.2.1 General 159

6.35.2.2 Input Data 160

6.35.2.3 Output Analytics 163

6.35.2.4 Mitigation or Prevention 163

6.35.3 Impacts on services, entities and interfaces 164

6.36 Solution \#36: Registration Signalling Analytics to support detection and prevention of signalling storm 164

6.36.1 Description 164

6.36.2 Procedures 167

6.36.3 Impacts on services, entities and interfaces 168

6.37 Solution \#37: Signalling storm detection and mitigation based on O&M data 168

6.37.1 Description 168

6.37.2 Procedures 169

6.37.3 Impacts on services, entities and interfaces 169

6.38 Solution \#38: NWDAF-assisted signalling storm analytics, predictions, prevention and mitigation 170

6.38.1 General description 170

6.38.1.1 NWDAF-assisted signalling storm detection 170

6.38.1.1.1 Input Data 171

6.38.1.1.2 Output Data 172

6.38.1.1.3 NWDAF-assisted signalling storm analytics Procedure 173

6.38.1.2 NWDAF-assisted signalling storm prevention and mitigation 174

6.38.1.2.1 Input Data 175

6.38.1.2.2 Output Data 175

6.38.1.2.3 NWDAF-assisted signalling storm prevention and mitigation procedure 176

6.38.2 Impacts on existing services, entities and interfaces 177

6.39 Solution \#39: NWDAF Analytics based Signalling Strom Prediction 177

6.39.1 Description 177

6.39.2 Procedures 180

6.39.2.1 UE registration signalling storm prevention/mitigation using Control Plane Signalling Abnormality Analytics 182

6.39.2.2 Inter-NF Signalling storm prevention/mitigation by SCP using NWDAF Analytics 184

6.39.3 Impacts on existing services, entities and interfaces 185

7 Overall Evaluation 185

8 Conclusions 185

8.1 Conclusions for KI#1: Enhancements to LCS to support Direct AI/ML based Positioning 185

8.2 Conclusions for KI#2: 5GC Support for Vertical Federated Learning 186

8.3 Conclusions for KI#3: NWDAF-assisted policy control and QoS enhancement 188

8.4 Conclusions for KI#4: NWDAF-assisted Network Abnormal Behaviour Mitigation and Prevention 189

Annex A: Change history 191
