---
spec: TR 23.700-82
version: 19.1.0
release: '19'
clause: contents
title: Contents
source_archive: 23700-82-j10.zip
source_document: 23700-82-j10.docx
content_origin: 3gpp-source
---

# Contents

Foreword 9

Introduction 10

1 Scope 11

2 References 11

3 Definitions of terms, symbols and abbreviations 12

3.1 Terms 12

3.2 Symbols 12

3.3 Abbreviations 12

4 Analysis of AI/ML support in 3GPP 13

4.1 AI/ML related use cases and requirements in 3GPP 13

4.1.1 Description 13

4.1.2 Analysis 13

4.2 AI/ML related stage 2 work in 3GPP 15

4.2.1 Description 15

4.2.1.1 3GPP SA2 15

4.2.1.2 3GPP SA4 15

4.2.1.2 3GPP SA5 15

4.2.2 Analysis 16

5 Key issues 16

5.1 Key issue #1: Support of Architecture Enhancement and Functions for Application Layer AI/ML Services 16

5.2 Key issue #2: AI/ML-enhanced ADAES 17

5.3 Key issue #3: Support for federated learning 18

5.4 Key issue #4: Key issue on supporting Vertical FL at enablement layer 18

5.5 Key Issue #5: Support for of AI/ML operation splitting between AI/ML endpoints and in-time transfer of AI/ML models 19

5.6 Key issue #6: Support for transfer learning 20

5.7 Key issue #7: Discovery or Support of Member Selection and Maintenance for Application Layer AIML Service 21

6 Application enablement architecture requirements 22

6.1 General requirements 22

6.2 ADAE capability related requirements 22

6.3 AIMLE capability related requirements 22

7 Application architecture for enabling AI/ML services 22

7.1 General 22

7.2 Application enablement architecture 23

7.2.1 On-Network AIML Enablement (AIMLE) Functional Architecture 23

7.2.1.1 Service-based AIMLE architecture representation 23

7.2.2 Off-Network AIML Enablement Functional Architecture 25

7.2.3 Architecture representation for supporting ADAE analytics 25

7.2.4 Functional Entities Description 26

7.2.4.1 General 26

7.2.4.2 AIMLE client 26

7.2.4.3 AIMLE server 26

7.2.4.4 ML repository 26

7.2.5 Reference Points Description 26

7.2.5.1 General 26

7.2.5.2 AIML-UU 26

7.2.5.3 AIML-S 26

7.2.5.4 AIML-C 26

7.2.5.5 AIML-R 26

7.2.5.6 AIML-E 27

7.2.5.7 ADAE-S 27

7.2.5.8 SEAL-X 27

8 Solutions 27

8.0 Mapping of solutions to key issues 27

8.1 Solution #1: ADAE Functional Architecture Enhancement to Support Application Layer AI/ML Services 28

8.1.2 Functional Architecture 28

8.1.2.1 General 28

8.1.2.2 On-Network MTME-enhanced ADAE Functional Architecture 28

8.1.2.3 On-Network AIML Enablement Layer Functional Architecture 29

8.1.2.4 Coexistence of MTME-enhanced ADAE and AIML enabler 29

8.1.3 Functional Entities Description 30

8.1.3.1 General 30

8.1.3.2 ADAE client 30

8.1.3.3 ADAE server 30

8.1.3.4 AIML Enablement client 30

8.1.3.5 AIML Enablement server 30

8.1.4 Reference Points Description 30

8.1.4.1 General 30

8.1.4.2 ADAE-UU 30

8.1.4.3 ADAE-S 30

8.1.4.4 ADAE-C 31

8.1.4.5 AIML-UU 31

8.1.4.6 AIML-S 31

8.1.4.7 AIML-C 31

8.1.4.8 AIML-E 31

8.1.5 Architecture Impacts 31

8.1.6 Corresponding APIs 31

8.1.7 Solution evaluation 31

8.2 Solution #2: ML client information retrieval 32

8.2.1 General 32

8.2.2 Procedures 32

8.2.3 Architecture Impacts 32

8.2.4 Corresponding APIs 33

8.2.5 Solution evaluation 33

8.3 Solution #3: Provision of ML clients to support AI/ML at the application layer 33

8.3.1 General 33

8.3.2 Procedures 34

8.3.3 Architecture Impacts 34

8.3.4 Corresponding APIs 34

8.3.5 Solution evaluation 34

8.4 Solution #4: Support for ML-enabled ADAE analytics 35

8.4.1 Solution description 35

8.4.2 Architecture Impacts 37

8.4.3 Corresponding APIs / information 37

8.4.4 Solution evaluation 37

8.5 Solution #5: AI/ML model management 37

8.5.1 General 37

8.5.2 AI/ML model information storage procedure 38

8.5.3 AI/ML model information discovery procedure 38

8.5.4 Architecture Impacts 39

8.5.5 Corresponding APIs 40

8.5.5.1 AI/ML model information storage request 40

8.5.5.2 AI/ML model information storage response 41

8.5.5.3 AI/ML model information discovery request 41

8.5.5.4 AI/ML model information discovery response 41

8.5.6 Solution evaluation 42

8.6 Solution #6: AIML enablement client selection 42

8.6.1 Solution description 42

8.6.1.1 AIML Enablement Client Selection with SEAL and 5GC support 42

8.6.1.2 Dynamic AIML Enablement Client selection and Monitoring Subscription 43

8.6.2 Architecture Impacts 45

8.6.3 Corresponding APIs 45

8.6.4 Solution evaluation 46

8.7 Solution #7: AIML enablement client discovery 46

8.7.1 Solution description 46

8.7.2 Architecture Impacts 48

8.7.3 Corresponding APIs 48

8.7.4 Solution evaluation 49

8.8 Solution #8: AIML enablement client registration 49

8.8.1 Solution description 49

8.8.2 Architecture Impacts 50

8.8.3 Corresponding APIs 50

8.8.4 Solution evaluation 51

8.9 Solution #9: Support for FL member registration 51

8.9.1 Solution description 51

8.9.1.1 Procedure for FL member registration 51

8.9.1.2 Procedure for FL member registration update 52

8.9.2 Architecture Impacts 53

8.9.3 Corresponding APIs 53

8.9.4 Solution evaluation 53

8.10 Solution #10: AI/ML member participation configurations provisioning and management 53

8.10.1 General 53

8.10.2 Procedures 54

8.10.2.1 AI/ML member participation configurations provisioning and management 54

8.10.3 Information flows 54

8.10.3.1 AI/ML member participation configurations provisioning and management request 54

8.10.3.2 AI/ML member participation configurations provisioning and management response 55

8.10.4 Architecture Impacts 55

8.10.5 Corresponding APIs 55

8.10.6 Solution evaluation 55

8.11 Solution #11: AIML service lifecycle management procedure 56

8.11.1 General 56

8.11.2 AIML service lifecycle management procedure 56

8.11.3 Architecture Impacts 57

8.11.4 Corresponding APIs 57

8.11.5 Solution evaluation 57

8.12 Solution #12: AI/ML model lifecycle management 58

8.12.1 Solution Description on AI/ML Model Lifecycle Management 58

8.12.2 Consumer-based ML Model Performance Degradation Detection 59

8.12.2 Architecture Impacts 60

8.12.3 Corresponding APIs 60

8.12.4 Solution evaluation 61

8.13 Solution #13: Analytics and Assistance Information Collection for Supporting FL Member (Re)Selection with the ADAES capabilities 61

8.13.1 General 61

8.13.2 Procedure on Analytics/Assistance Information Collection for FL member (re)selection with the ADAES capabilities 61

8.13.3 Architecture Impacts 62

8.13.4 Corresponding APIs 62

8.13.5 Solution evaluation 62

8.14 Solution #14: AI/ML policies provisioning and management 62

8.14.1 General 62

8.14.2 Procedures 63

8.14.2.1 AI/ML policies provisioning and management 63

8.14.3 Information flows 63

8.14.3.1 AI/ML service request with policies provisioning and management information 63

8.14.3.2 AI/ML service response with policies provisioning and management information 66

8.14.4 Architecture Impacts 66

8.14.5 Corresponding APIs 66

8.14.6 Solution evaluation 66

8.15 Solution #15: ADAES support for AI-enabled DN Energy Analytics 67

8.15.1 Solution description 67

8.15.2 Architecture Impacts 68

8.15.3 Corresponding APIs 68

8.15.4 Solution evaluation 68

8.16 Solution #16: Support for FL event notifications 69

8.16.1 Solution description 69

8.16.1.1 Procedure on subscription for FL related events 69

8.16.1.2 Procedure on FL related event notification 69

8.16.1.3 List of FL-related Events 70

8.16.2 Architecture Impacts 71

8.16.3 Corresponding APIs 71

8.16.4 Solution evaluation 71

8.17 Solution #17: AIML operational management 72

8.17.1 Solution description 72

8.17.1.1 AIML operational management procedure 72

8.17.1.2 AIML enablement client task triggering 74

8.17.2 Architecture Impacts 75

8.17.3 Corresponding AIML-S APIs 75

8.17.5 Solution evaluation 78

8.18 Solution #18: Supporting VFL in Enablement Layer 78

8.18.1 Solution description 78

8.18.2 Architecture Impacts 80

8.18.3 Corresponding APIs 80

8.18.4 Solution evaluation 80

8.19 Solution #19: Support for AIML operation splitting 80

8.19.1 Solution description 80

8.19.2 Procedure for AIML operation splitting discovery 81

8.19.2.1 Procedure for subscribe-notify for split operation pipeline events 83

8.19.2.2 Procedure for VAL server AIML split operation registration 83

8.19.2.3 Procedure for client creating pipeline 84

8.19.3 Architecture Impacts 84

8.19.4 Corresponding APIs 85

8.19.4.1 API Overview 85

8.19.4.2 Information flows 85

8.19.5 Solution evaluation 88

8.20 Solution #20: AIML Enabler support for Transfer Learning 88

8.20.1 Solution description 88

8.20.2 Architecture Impacts 89

8.20.3 Corresponding APIs 90

8.20.4 Solution evaluation 90

8.21 Solution #21: AIML data management procedure 90

8.21.1 Solution description 90

8.21.2 Architecture Impacts 92

8.21.3 Corresponding APIs 92

8.21.4 Solution evaluation 93

8.22 Solution #22: Horizontal Federated Learning training 93

8.22.1 Solution description 93

8.22.2 Architecture Impacts 95

8.22.3 Corresponding APIs 95

8.22.4 Solution evaluation 95

8.23 Solution #23: AIML services in edge 96

8.23.1 Solution description 96

8.23.2 Architecture Impacts 98

8.23.3 Corresponding AIML-S APIs 98

8.23.4 Corresponding AIML-UU APIs 99

8.23.5 Solution evaluation 99

8.24 Solution #24: Dynamic ML model distribution 99

8.24.1 Solution description 99

8.24.2 Architecture Impacts 101

8.24.3 Corresponding APIs 101

8.24.4 Solution evaluation 103

8.25 Solution #25: Support for AIML model distribution 103

8.25.1 Solution description 103

8.25.2 Procedures 103

8.25.2.1 Subscription for AIML model updates 103

8.25.2.2 Notification for AIML model updates 105

8.25.3 Architecture Impacts 105

8.25.4 Corresponding APIs 106

8.25.4.1 API Overview 106

8.25.5 Solution evaluation 106

8.26 Solution #26: Support for FL member grouping 106

8.26.1 Solution description 106

8.26.2 Architecture Impacts 107

8.26.3 Corresponding APIs 108

8.26.4 Solution evaluation 108

8.27 Solution #27: New ADAE Analytics for Supporting FL Member (re-) selection 108

8.27.1 General 108

8.27.2 Procedure 109

8.27.2.1 Subscribe-notify model 109

8.27.2.2 Request-response model 110

8.27.3 Information flows 110

8.27.3.1 General 110

8.27.3.2 Application Layer AI/ML Member capability analytics subscription request 110

8.27.3.3 Application Layer AI/ML Member capability analytics subscription response 111

8.27.3.4 Application layer AI/ML Member capability analytics notification 111

8.27.3.5 Application Layer AI/ML Member capability data collection subscription request 112

8.27.3.6 Application Layer AI/ML Member capability data collection subscription response 112

8.27.3.7 Data Notification 112

8.27.3.8 Get analytics data request 113

8.27.3.9 Get analytics data response 113

8.27.4 Architecture Impacts 114

8.27.5 Corresponding APIs 114

8.15.6 Solution evaluation 114

8.28 Solution #28: Support AI/ML Splitting Operations in Enablement Layer 114

8.28.1 General 114

8.28.2 Procedures for supporting assistance of split AI/ML operations 115

8.28.2.1 Procedure for subscribe/request assistance of split AI/ML operations 115

8.28.2.2 Procedure for assistance of AI/ML task/model/data delivery/distribution 116

8.28.3 Architecture Impacts 117

8.28.4 Corresponding APIs 117

8.28.5 Solution evaluation 117

8.29 Solution #29: Enhance AIML Services for Assisting Edge Computing 118

8.29.1 General 118

8.29.2 Procedures for assisting edge computing 118

8.29.3 Architecture Impacts 119

8.29.4 Corresponding APIs 119

8.29.5 Solution evaluation 119

8.30 Solution #30: Support Transfer of Intermediate AIML Operation Information 119

8.30.1 General 119

8.30.2 Procedures for intermediate AI/ML information transfer 120

8.30.2.1 AI/ML Enablement Server assist intermediate AI/ML information transfer 120

8.30.2.2 Direct intermediate AI/ML information transfer 121

8.30.3 Architecture Impacts 122

8.30.4 Corresponding APIs 122

8.30.5 Solution evaluation 123

8.31 Solution #31: Supporting AIML inference service in edge 123

8.31.1 Solution description 123

8.31.2 Architecture Impacts 124

8.31.3 Corresponding APIs 124

8.31.4 Solution evaluation 124

9 Deployment scenarios 125

9.1 General 125

9.2 Deployment model #1: Cloud-deployed AIMLE server 125

9.3 Deployment model #2 Edge-deployed AIMLE server 125

9.4 Deployment model #3: Hierarchical AIMLE server deployment 126

10 Business Relationships 127

11 Overall evaluation 127

11.1 Summary of enablement capabilities and APIs 127

11.1.1 Summary of AIMLE services 127

11.1.2 Summary of ADAE analytics enhancements 128

11.2 Key issue #1: Support of Architecture Enhancement and Functions for Application Layer AI/ML Services 129

11.3 Key issue #2: AI/ML-enhanced ADAES 130

11.4 Key issue #3: Support for federated learning 131

11.5 Key issue #4: Supporting Vertical FL at enablement layer 131

11.6 Key Issue #5: Support for of AI/ML operation splitting between AI/ML endpoints and in-time transfer of AI/ML models 132

11.6.1 Split AI/ML operation 132

11.6.2 Transfer of AI/ML models 133

11.7 Key issue #6: Support for transfer learning 133

11.8 Key issue #7: Discovery or Support of Member Selection and Maintenance for Application Layer AIML Service 133

12 Conclusions 134

12.0 General conclusions and recommendations 134

12.1 Conclusions of key issue #1 135

12.2 Conclusions of key issue #2 135

12.3 Conclusions of key issue #3 135

12.4 Conclusions of key issue #4 136

12.5 Conclusions of Key Issue #5 136

12.6 Conclusions of key issue #6 136

12.7 Conclusions of key issue #7 137

Annex A: Change history 138
