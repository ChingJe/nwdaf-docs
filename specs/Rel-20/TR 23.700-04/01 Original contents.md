---
spec: TR 23.700-04
version: 20.0.0
release: '20'
clause: contents
title: Contents
source_archive: 23700-04-k00.zip
source_document: 23700-04-k00.docx
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

4.1 Architectural Assumptions 11

4.2 Architectural Requirements 11

5 Use Cases and Key Issues 12

5.1 Use cases 12

5.1.1 Use Case \#1: AI/ML-assisted user plane traffic pattern and behaviour analysis to support efficient performance of User Plane 12

5.1.2 Use Case \#2: User plane performance optimization with assistance of NWDAF 12

5.2 Key Issues 13

5.2.1 Key Issue \#1: Transfer of data over UP for UE data collection 13

5.2.1.1 Description 13

5.2.2 Key Issue \#2: AI/ML-assisted user plane traffic pattern and behaviour analysis to support efficient performance of User Plane 13

5.2.2.1 Description 13

6 Solutions 14

6.0 Mapping of Solutions to Key Issues and Use Cases 14

6.1 Solution \#1: UE data collection via DCCF in CN over UP 14

6.1.1 High level principles 14

6.1.2 Description 15

6.1.3 Procedures 16

6.1.3.1 UE data collection via DCCF in CN over UP 16

6.1.4 Impacts on services, entities and interfaces 18

6.2 Solution \#2: Generalized UE data collection over user plane 18

6.2.1 High level principles 18

6.2.2 Description 18

6.2.3 Procedures 19

6.2.4 Impacts on services, entities and interfaces 20

6.3 Solution \#3: General framework for supporting transfer of standardised data from the UE via user plane 20

6.3.1 High level principles 20

6.3.2 Description 21

6.3.3 Procedures 22

6.3.4 Impacts on services, entities and interfaces 24

6.4 Solution \#4: UP Building for UE data collection 24

6.4.1 High level principles 24

6.4.2 Description 24

6.4.3 Procedures 25

6.4.3.1 The UP building procedure 25

6.4.3.2 UE training data collection triggered by trust AF 26

6.4.3.3 UE training data collection triggered by untrust AF 27

6.4.3.4 NF profile registration and update procedure 28

6.4.3.5 URSP generation procedure 28

6.4.3.6 UE policy distribution 31

6.4.4 Impacts on services, entities and interface 32

6.5 Solution \#5: Establish the UP connection between UE and 5GC to support the standardized UE data collection 32

6.5.1 High level principles 32

6.5.2 Description 33

6.5.3 Procedures 33

6.5.3.1 UE Data Collection Mapping Table 33

6.5.3.2 Data collection application information and UE data collection configuration provision 34

6.5.3.3 UP connection establishment and UE data collection request between the UE and DCF to support the UE data collection 35

6.5.3.4 Discover and selection of data collection UE 38

6.5.4 Impacts on services, entities and interfaces 39

6.6 Solution \#6: Hybrid solution using UDCF to support UP based UE data collection for NR AIML air interface operation 40

6.6.1 High level principles 40

6.6.2 Description 40

6.6.3 Procedures 41

6.6.3.1 General call flow about data collection 41

6.6.3.2 Traffic differentiation 43

6.6.3.3 Format of data reporting parameters send to UDCF 44

6.6.3.4 Data collection reject and cancellation 45

6.6.4 Impacts on services, entities and interfaces 46

6.7 Solution \#7: AF-Triggered UE Data Collection with QoS-Assured Data Transfer 46

6.7.1 High level principles 46

6.7.2 Description 47

6.7.2.1 Negotiation of UE capability and network support for AI/ML enabled feature 48

6.7.2.2 UE policy configuration enhancement for UP Data Collection and Transfer 49

6.7.2.3 Control of UE Data Transfer over User Plane 49

6.7.3 Procedures 50

6.7.4 Impacts on services, entities and interfaces 53

6.8 Solution \#8: AF provisioning of UE Data collection requirements with NEF support 55

6.8.1 High-level solution principles 55

6.8.2 Description 55

6.8.3 Procedure for collecting UE data and reporting to the UE side training server 56

6.8.4 Impacts on services, entities and interfaces 58

6.9 Solution \#9: High-level UE data collection procedure 58

6.9.0 High-level solution Principles 58

6.9.1 Description 58

6.9.2 Procedures 59

6.9.2.1 Initiating UE data collection 59

6.9.3 Impacts on services, entities and interface 60

6.10 Solution \#10: RAN assist UE data collection 60

6.10.1 High level principles 60

6.10.2 Description 61

6.10.3 Procedures 61

6.10.3.1 Procedure of CN informing RAN to initiate UE data collection 61

6.10.4 Impacts on services, entities and interfaces 62

6.11 Solution \#11: UE-Side model training data collection via DCCF and ADRF in CN over UP 62

6.11.1 High level principles 62

6.11.2 Description 63

6.11.3 Procedures 64

6.11.3.1 UE-side model training data collection via DCCF and ADRF in CN over UP 64

6.11.4 Impacts on services, entities and interfaces 65

6.12 Solution \#12: UP-based UE data collection 65

6.12.1 High level principles 65

6.12.2 Description 66

6.12.3 Procedures 67

6.12.4 Impacts on existing services, entities and interfaces 69

6.13 Solution \#13: User Plane solution for Data collection for UE-side model training 70

6.13.1 High-level solution principles 70

6.13.2 Description 70

6.13.3 Procedure for collecting UE data and reporting to the UE training centre 72

6.13.4 Aspects for further consideration 74

6.13.5 Impacts on services, entities and interfaces 74

6.14 Solution \#14: UE data collection over the user plane 74

6.14.1 High level principles 74

6.14.2 Description 74

6.14.3 Procedures 75

6.14.4 Impacts on services, entities and interfaces 76

6.15 Solution \#15: UE data transfer over UP based on guidance for URSP 76

6.15.1 High level principles 76

6.15.2 Description 76

6.15.2.1 Terminology 76

6.15.3 Procedures 77

6.15.4 Impacts on services, entities and interfaces 78

6.16 Solution \#16: Support of standardized data over UP for NR air interface operation with UE-side model training 78

6.16.1 High level principles 78

6.16.2 Description 79

6.16.3 Procedures 80

6.16.3.1 Procedures of DCF initiated UP connection for data transfer 80

6.16.3.2 Procedures of UE initiated UP connection for data transfer 82

6.16.4 Impacts on services, entities and interfaces 83

6.17 Solution \#17: Data transfer over UP connection for UE data collection 84

6.17.1 High level principles 84

6.17.2 Description 84

6.17.3 Procedures 85

6.17.4 Impacts on services, entities and interfaces 86

6.18 Solution \#18: General Framework for UE Data Collection over UP 87

6.18.1 High level principles 87

6.18.2 Description 87

6.18.3 Procedures 90

6.18.3.1 Initiation of UE data collection transfer 90

6.18.3.2 Modification and Termination of UE data collection transfer 92

6.18.4 Impacts on services, entities and interfaces 93

6.19 Solution \#19: Standardized transfer of standardized data over UP for UE-side data collection 94

6.19.1 High-level solution Principles 94

6.19.2 Description 94

6.19.2.1 Data Collection Profile 95

6.19.3 Procedures 96

6.19.3.1 MNO Controlled Procedures for configuration of UE-side Data Collection 96

6.19.3.2 Procedures for MNO Controlled UE-side Data Collection 99

6.19.3.3 Establishing a PDU Session for Data Collection 102

6.19.4 Impacts on services, entities and interfaces 104

6.20 Solution \#20: UE data collection over UP 105

6.20.1 High level principles 105

6.20.2 Description 105

6.20.3 Procedures 106

6.20.3.1 UE data collection over UP 106

6.20.4 Impacts on services, entities and interfaces 107

6.21 Solution \#21: Control of UE data collection and transfer with UE data verification 107

6.21.1 High-level principles 107

6.21.2 Description 108

6.21.3 Procedures 109

6.21.4 Impacts on services, entities and interfaces 111

6.22 Solution \#22: Assistance for User plane performance optimization 112

6.22.1 High level principles 112

6.22.2 Description 112

6.22.3 Procedures 114

6.22.4 Impacts on services, entities and interfaces 115

6.23 Solution \#23: AI/ML-assisted UP traffic pattern and behaviour analysis 115

6.23.1 High level principles 115

6.23.2 Description 116

6.23.3 Procedures 117

6.23.3.1 UP pattern data collection from UPF as Input Data to NWDAF 117

6.23.4 Impacts on services, entities and interfaces 120

6.24 Solution \#24: Collocation of UPF and NWDAF containing AnLF to support efficient performance of User Plane 120

6.24.1 High level principles 120

6.24.2 Description 120

6.24.2.0 General 120

6.24.2.1 Input Data 121

6.24.2.2 Output data 121

6.24.2.3 Example actions 122

6.24.3 Procedures 123

6.24.3.1 Support of efficient performance of User Plane by collocating UPF and NWDAF containing AnLF 123

6.24.4 Impacts on services, entities and interfaces 124

6.25 Solution 25: NWDAF assisted traffic pattern generation for user plane performance optimization 125

6.25.1 High level principles 125

6.25.2 Description 125

6.25.3 Procedures 125

6.25.4 Impacts on services, entities and interfaces 128

6.26 Solution \#26: User plane analytics derivation with efficient UP data collection 128

6.26.1 High level principles 128

6.26.2 Description 128

6.26.3 Procedures 129

6.26.4 Impacts on services, entities and interfaces 131

6.27 Solution \#27: Support of user plane management analytics 131

6.27.1 High level principles 131

6.27.2 Description 131

6.27.2.1 General Description of UP Management Analytics 131

6.27.2.2 Input Data 132

6.27.2.3 Output Analytics 134

6.27.3 Procedures 136

6.27.4 Impacts on services, entities and interfaces 137

6.28 Solution \#28: Analysis of user plane traffic towards malicious and/or forbidden sites 137

6.28.1 High Level principles 137

6.28.2 Description 137

6.28.2 Procedures 138

6.28.3 Impacts on services, entities and interfaces 140

6.29 Solution \#29: NWDAF-assisted Analytics for Efficient User Plane Performance 140

6.29.1 High level principles 140

6.29.2 Description 140

6.29.2.1 Input Data 141

6.29.2.2 Output Analytics 142

6.29.2 Procedures 143

6.29.3 Impact on services, entities and interfaces 143

6.30 Solution \#30: User plane traffic pattern and anomaly detection and policy update based on NWDAF analysis 144

6.30.1 High-level principles 144

6.30.2 Description 144

6.30.3 Procedures 145

6.30.4 Impacts on services, entities and interfaces 146

6.31 Solution \#31: Analytics on user plane traffic pattern 146

6.31.1 High-level solution principles 146

6.31.2 Description 147

6.31.2 Procedures 149

6.31.3 Impacts on services, entities and interface 150

6.5 Solution \#32: How to authorize a UE request to make use of radio resources for measurements 150

6.5.1 High level principles 150

6.5.2 Description 150

6.5.3 Procedures 151

6.5.4 Impacts on services, entities and interfaces 152

6.33 Solution \#33: Malicious traffic identification and mitigation 152

6.33.1 High level principles 152

6.33.2 Functional Description 152

6.33.2.1 General Description 152

6.33.2.2 Input data of the Analytics 153

6.33.2.2 Output data of the Analytics 153

6.33.3 Procedures 154

6.33.4 Impacts on services, entities and interface 155

6.34 Solution \#34: User plane traffic pattern and anomaly detection and policy update based on NWDAF analysis 155

6.34.1 High-level principles 155

6.34.2 Description 155

6.34.3 Procedures 156

6.34.4 Impacts on services, entities and interfaces 157

6.35 Solution \#35: NWDAF-Driven UPF Function Set Dynamic Adaptation Mechanism 157

6.35.1 High level principles 157

6.35.2 Description 157

6.35.3 Procedures 159

6.35.4 Impacts on services, entities and interfaces 159

6.36 Solution \#36: Analysis of abnormal user plane traffic 160

6.36.1 High level principles 160

6.36.2 Description 160

6.36.2.1 Abnormal user plane traffic AnalyticsID 161

6.36.2.2 Input Data 161

6.36.2.3 Output Analytics 162

6.36.2.4 Procedure for AnalyticsID on abnormal user plane traffic 164

6.36.2.5 Provisioning of N4 rules to take mitigation actions on node level 165

6.36.3 Impacts on services, entities and interfaces 167

7 Interim agreements 167

7.1 Agreed Principles 167

7.1.1 Agreed Principles for KI#1 167

7.1.2 Agreed Principles for KI#2 169

7.1.2.1 General 169

7.1.2.2 Agreed Principles for Use Case \#1 169

7.1.2.3 Agreed Principles for Use Case \#2 170

7.2 Topics for further consideration 171

7.2.1 Topics for further consideration for KI#1 171

7.2.2 Topics for further consideration for KI#2 171

8 Conclusions 171

8.1 Conclusions for KI#1 171

8.2 Conclusions for KI#2 171

8.2.1 General 171

8.2.2 Conclusions for KI#2 Use Case#1 172

8.2.3 Conclusions for KI#2 Use Case#2 173

Annex A: Change history 175
