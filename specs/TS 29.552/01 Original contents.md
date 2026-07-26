---
spec: TS 29.552
version: 18.7.0
release: '18'
clause: contents
title: Contents
source_archive: 29552-i70.zip
source_document: 29552-i70.docx
source_archive_sha256: 3b3ac7f50c83fc234294ba8224344053f512207c1fc13ce6502230d2a8af7464
source_document_sha256: 6c12a650b021585e94301121de56e86f3435eb485e484fcb4c9410d7741a1add
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Contents

Foreword 5

1 Scope 7

2 References 7

3 Definitions of terms, symbols and abbreviations 9

3.1 Terms 9

3.2 Symbols 9

3.3 Abbreviations 9

4 Reference Architecture for Data Analytics 9

4.1 General 9

4.2 Data Collection 10

4.3 Analytics Exposure 11

4.4 Data Storage and Retrieval 12

4.5 Roaming architecture for data collection and analytics exposure 12

5 Signalling Flows for the Network Data Analytics Framework 12

5.1 General 12

5.2 Analytics Exposure Procedures 13

5.2.1 General 13

5.2.2 Network data analytics Subscribe/Unsubscribe/Notify 13

5.2.2.1 Analytics Subscribe/Unsubscribe/Notify initiated by 5GC NFs, OAM or AFs 13

5.2.2.2 Analytics Subscribe/Unsubscribe/Notify initiated by AFs via the NEF 14

5.2.3 Network data analytics information request 15

5.2.3.1 Analytics information request initiated by 5GC NFs, OAM or AFs 15

5.2.3.2 Analytics information request initiated by AFs via the NEF 15

5.2.4 Analytics Exposure via DCCF 16

5.2.5 Analytics Exposure via DCCF and MFAF 20

5.2.6 Procedure for Analytics Exposure in Roaming Case 25

5.2.6.1 Analytics Exposure from HPLMN to VPLMN for inbound roaming users 25

5.2.6.2 Analytics Exposure from VPLMN to HPLMN for outbound roaming users 26

5.3 Analytics Aggregation from Multiple NWDAFs 28

5.3.1 General 28

5.3.2 Analytics aggregation with provisioning of Area of Interest 28

5.3.3 Analytics aggregation without provisioning of Area of Interest 30

5.4 Procedures for Analytics Transferring 31

5.4.1 Analytics context transfer initiated by target NWDAF selected by the NWDAF service consumer 31

5.4.2 Analytics Subscription Transfer initiated by source NWDAF 33

5.4.3 Prepared analytics subscription transfer 36

5.5 Data Collection 38

5.5.1 Procedure for Data Collection from NFs 38

5.5.1.1 Data Collection from NFs 38

5.5.2 Data collection profile registration 40

5.5.3 Procedure for Data Collection using DCCF 42

5.5.3.1 Data Collection via DCCF 42

5.5.3.2 Data Collection via Messaging Framework 46

5.5.4 Procedure for Data Collection in Roaming Case 51

5.5.4.1 Data Collection by H-RE-NWDAF from V-RE-NWDAF for outbound roaming users 51

5.5.4.2 Data Collection by V-RE-NWDAF from H-RE-NWDAF for inbound roaming users 53

5.6 ML Model provisioning procedures 54

5.6.1 General 54

5.6.2 ML Model Subscribe/Unsubscribe/Notify procedure 54

5.6A ML Model Training procedures 55

5.6A.1 General 55

5.6A.2 ML Model Training Subscribe/Unsubscribe/Notify procedure 55

5.7 Procedures for Specific Network Data Analytics 56

5.7.1 General 56

5.7.2 Network Slice (Instance) load level Analytics 56

5.7.3 Observed Service Experience Analytics 59

5.7.4 NF load Analytics 62

5.7.5 Network Performance Analytics 65

5.7.6 UE Mobility Analytics 67

5.7.7 UE Communication Analytics 70

5.7.8 Expected UE behavioural Analytics 73

5.7.9 Abnormal UE behavioural Analytics 75

5.7.10 User Data Congestion Analytics 76

5.7.11 QoS Sustainability Analytics 79

5.7.12 Dispersion Analytics 81

5.7.13 WLAN Performance Analytics 84

5.7.14 Session Management Congestion Control Experience Analytics 86

5.7.15 Redundant Transmission Experience Analytics 87

5.7.16 DN Performance Analytics 90

5.7.17 PFD Determination Analytics 92

5.7.18 E2E data volume transfer time analytics 94

5.7.19 PDU Session Traffic Analytics 97

5.7.20 Relative Proximity Analytics 99

5.7.21 Movement Behaviour Analytics 102

5.7.22 Location Accuracy Analytics 103

5.8 Procedures for NWDAF Discovery and Selection 106

5.8.1 General 106

5.8.2 Procedures related to NRF 106

5.8.2.1 General 106

5.8.2.2 NWDAF (De-)Registration in NRF 106

5.8.2.3 Consumer discovery and selection of NWDAF in NRF 106

5.8.3 Procedures related to UDM 106

5.8.3.1 General 106

5.8.3.2 NWDAF containing AnLF Registration/Deregistration in UDM 106

5.8.3.2.1 NWDAF containing AnLF Registration in UDM 106

5.8.3.2.2 NWDAF containing AnLF Update of Registration in UDM 107

5.8.3.2.3 NWDAF containing AnLF De-Registration in UDM 108

5.8.3.3 Consumer discovery and selection of NWDAF containing AnLF in UDM 108

5.8.4 Procedures for PCF learning NWDAF IDs for served UEs 109

5.9 Analytics Data Repository procedures 110

5.9.1 General 110

5.9.2 Historical Data and Analytics Storage/Retrieval/Deletion procedure 110

5.9.3 Historical Data and Analytics Storage via Notifications 111

5.10 Federated Learning among Multiple NWDAFs 113

5.10.1 General 113

5.10.2 Procedures related to Federated Learning 113

5.10.2.1 General Procedure for Federated Learning among Multiple NWDAF Instances 113

5.10.2.2 Preparation Procedure for Federated Learning 117

5.10.2.3 Procedure for Maintenance of Federated Learning Process 117

5.11 Analytics Accuracy Monitoring Procedures 119

5.11.1 General 119

5.12 ML Model Accuracy Monitoring Procedures 119

5.12.1 General 119

5.13 DCCF Change Procedures 119

5.13.1 DCCF (re-)selection initiated by the data consumer 119

Annex A (informative): Change history 121
