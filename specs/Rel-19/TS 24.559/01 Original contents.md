---
spec: TS 24.559
version: 19.4.1
release: '19'
clause: contents
title: Contents
source_archive: 24559-j41.zip
source_document: 24559-j41.docx
content_origin: 3gpp-source
---

# Contents

Foreword 7

1 Scope 9

2 References 9

3 Definitions of terms, symbols and abbreviations 10

3.1 Terms 10

3.2 Abbreviations 10

4 General description 10

4.1 Overview 10

5 Functional entities 11

5.1 Application data analytics enablement server (ADAES) 11

5.2 Application data analytics enablement client (ADAEC) 11

6 Application data analytics enablement service API 11

6.1 General 11

6.2 Application performance analytics 11

6.2.1 Service description 11

6.2.1.1 Overview 11

6.2.2 Service Operations 11

6.2.2.1 Introduction 11

6.2.2.2 Subscribe_VAL_Performance_Analytics 11

6.2.2.2.1 General 11

6.2.2.2.2 Subscribing to VAL performance analytics event using Subscribe_VAL_Performance_Analytics service operation 12

6.2.2.3 Notify_VAL_Performance_Analytics 12

6.2.2.3.1 General 12

6.2.2.3.2 Notifying VAL performance analytics event using Notify_VAL_Performance_Analytics service operation 12

6.2.2.4 Unsubscribe_VAL_Performance_Analytics 12

6.2.2.4.1 General 12

6.2.2.4.2 Unsubscribing from VAL performance analytics event using Unsubscribe_VAL_Performance_Analytics service operation 12

6.3 UE-to-UE session performance analytics 13

6.3.1 Service description 13

6.3.1.1 Overview 13

6.3.2 Service Operations 13

6.3.2.1 Introduction 13

6.3.2.2 Fetch_UE2UE_Session_Performance_Analytics 13

6.3.2.2.1 General 13

6.3.2.2.2 Obtaining UE-to-UE session performance analytics using Fetch_UE2UE_Session_Performance_Analytics service operation 13

6.4 Edge load data collection 14

6.4.1 Service description 14

6.4.2 Service Operations 14

6.4.2.1 Introduction 14

6.4.2.2 Subscribe_Edge_Load_Data_Collection 14

6.4.2.2.1 General 14

6.4.2.2.2 Subscribing to edge load data collection event using Subscribe_Edge_Load_Data_Collection service operation 14

6.4.2.3 Notify_Edge_Load_Data_Collection 15

6.4.2.3.1 General 15

6.4.2.3.2 Notifying edge load data collection event using Notify_Edge_Load_Data_Collection service operation 15

6.4.2.4 Unsubscribe_Edge_Load_Data_Collection 15

6.4.2.4.1 General 15

6.4.2.4.2 Unsubscribing from edge load data collection event using Unsubscribe_Edge_Load_Data_Collection service operation 15

6.5 Service experience performance analytics 15

6.5.1 General 15

6.5.2 Service Operations 16

6.5.2.1 Introduction 16

6.5.2.2 Configure_Triggers_Service_Information_Experience_Report 16

6.5.2.2.1 General 16

6.5.2.2.2 Configuring service experience information reporting using Configure_Triggers_Service_Information_Experience_Report service operation 16

6.5.2.3 Void 17

6.5.2.4 Push_Service_Experience_Information_Report 17

6.5.2.4.1 General 17

6.5.2.4.2 Pushing service experience information report using Push_Service_Experience_Information_Report service operation 17

6.5.2.5 Pull_Service_Experience_Information_Report 17

6.5.2.5.1 General 17

6.5.2.5.2 Pulling service experience information report using Pull_Service_Experience_Information_Report service operation 17

6.6 Collision detection analytics 18

6.6.1 Service description 18

6.6.1.1 Overview 18

6.6.2 Service operations 18

6.6.2.1 Introduction 18

6.6.2.2 Subscribe_Collision_Detection 18

6.6.2.2.1 General 18

6.6.2.2.2 Subscribing to collision detection analytics using Subscribe_Collision_Detection service operation 18

6.6.2.3 Notify_Collision_Detection 19

6.6.2.3.1 General 19

6.6.2.3.2 Notifying collision detection analytics using Notify_Collision_Detection service operation 19

6.6.2.4 Unsubscribe_Collision_Detection 19

6.6.2.4.1 General 19

6.6.2.4.2 Unsubscribing from collision detection analytics using Unsubscribe_Collision_Detection service operation 19

6.7 Location-related UE Group Analytics 20

6.7.1 Service description 20

6.7.1.1 Overview 20

6.7.2 Service operations 20

6.7.2.1 Introduction 20

6.7.2.2 Subscribe_UE_Group_Location 20

6.7.2.2.1 General 20

6.7.2.2.2 Obtaining location-related UE group analytics using Subscribe_UE_Group_Location service operation 20

6.7.2.3 Notify_UE_Group_Location 21

6.7.2.3.1 General 21

6.7.2.3.2 Notifying location-related UE group analytics event using Notify_UE_Group_Location service operation 21

6.7.2.4 Unsubscribe_UE_Group_Location 21

6.7.2.4.1 General 21

6.7.2.4.2 Unsubscribing from location-related UE group analytics event using Unsubscribe_UE_Group_Location service operation 21

7 API Definitions 21

7.1 ADAE_ServiceConfiguration API 21

7.1.1 Introduction 21

7.1.2 Usage of HTTP 22

7.1.2.1 General 22

7.1.2.2 Content type 22

7.1.3 Resources 22

7.1.3.1 Overview 22

7.1.3.2 Resource: Application performance event subscription 24

7.1.3.2.1 Description 24

7.1.3.2.2 Resource definition 24

7.1.3.2.3 Resource standard methods 25

7.1.3.2.3.1 POST 25

7.1.3.2.4 Resource custom operations 25

7.1.3.3 Resource: Individual application performance event subscription 25

7.1.3.3.1 Description 25

7.1.3.3.2 Resource Definition 25

7.1.3.3.3 Resource Standard Methods 26

7.1.3.3.3.1 DELETE 26

7.1.3.3.4 Resource Custom Operations 27

7.1.3.4 Resource: UE-to-UE session performance analytics 27

7.1.3.4.1 Description 27

7.1.3.4.2 Resource definition 27

7.1.3.4.3 Resource standard methods 27

7.1.3.4.4 Resource custom operations 27

7.1.3.4.4.1 Overview 27

7.1.3.4.4.2 Fetch 27

7.1.3.5 Resource: Edge load data collection event subscription 28

7.1.3.5.1 Description 28

7.1.3.5.2 Resource definition 28

7.1.3.5.3 Resource standard methods 28

7.1.3.5.3.1 POST 28

7.1.3.5.4 Resource custom operations 29

7.1.3.6 Resource: Individual edge load event subscription 29

7.1.3.6.1 Description 29

7.1.3.6.2 Resource Definition 29

7.1.3.6.3 Resource Standard Methods 29

7.1.3.6.3.1 DELETE 29

7.1.3.6.4 Resource Custom Operations 30

7.1.3.7 Resource: Service experience information 30

7.1.3.7.1 Description 30

7.1.3.7.3.1 Void 31

7.1.3.7.4 Resource custom operations 31

7.1.3.7.4.1 Overview 31

7.1.3.7.4.2 Void 31

7.1.3.7.4.3 Operation: PULL Service Experience Information 31

7.1.3.8 Void 32

7.1.3.9 Resource: Collision detection analytics subscriptions 32

7.1.3.9.1 Description 32

7.1.3.9.2 Resource definition 32

7.1.3.9.3 Resource standard methods 32

7.1.3.9.3.1 POST 32

7.1.3.9.4 Resource custom operations 33

7.1.3.10 Resource: Individual collision detection analytics subscription 33

7.1.3.10.1 Description 33

7.1.3.10.2 Resource Definition 33

7.1.3.10.3 Resource Standard Methods 33

7.1.3.10.3.1 DELETE 33

7.1.3.10.4 Resource Custom Operations 34

7.1.3.11 Resource: Location-related UE group analytics subscriptions 34

7.1.3.11.1 Description 34

7.1.3.11.2 Resource definition 34

7.1.3.11.3 Resource standard methods 34

7.1.3.11.3.1 POST 34

7.1.3.11.4 Resource custom operations 35

7.1.3.12 Resource: Individual location-related UE group analytics subscription 35

7.1.3.12.1 Description 35

7.1.3.12.2 Resource Definition 35

7.1.3.12.3 Resource Standard Methods 35

7.1.3.12.3.1 DELETE 35

7.1.3.12.4 Resource Custom Operations 36

7.1.4 Notifications 37

7.1.4.1 General 37

7.1.4.2 Application performance event notification 37

7.1.4.2.1 Description 37

7.1.4.2.2 Notification definition 37

7.1.4.3 Edge load event notification 38

7.1.4.3.1 Description 38

7.1.4.3.2 Notification definition 38

7.1.4.4 Service experience information report event notification 39

7.1.4.4.1 Description 39

7.1.4.4.2 Notification definition 39

7.1.4.5 Collision detection analytics notification 40

7.1.4.5.1 Description 40

7.1.4.5.2 Notification definition 40

7.1.4.6 Location-related UE group analytics notification 41

7.1.4.6.1 Description 41

7.1.4.6.2 Notification definition 42

7.1.5 Data model 43

7.1.5.1 General 43

7.1.5.2 Structured data types 44

7.1.5.2.1 Introduction 44

7.1.5.2.2 Type: Ue2UePerfReq 44

7.1.5.2.3 Type: Ue2UePerfResp 44

7.1.5.2.4 Void 45

7.1.5.2.5 Void 45

7.1.5.2.6 Type: PullSrvExpInfo 45

7.1.5.2.7 Type: SrvExpInfoRep 45

7.1.5.2.8 Type: Ue2UeRepThreshold 45

7.1.5.2.9 Type: DataCollectReq 45

7.1.5.3 Simple data types and enumerations 46

7.1.5.3.1 Introduction 46

7.1.5.3.2 Simple data types 46

7.1.5.3.3 Void 46

7.1.6 Error Handling 46

7.1.6.1 General 46

7.1.6.2 Protocol Errors 46

7.1.6.3 Application Errors 46

7.1.7 Feature Negotiation 46

8 Usage of common API framework 46

8.1 General 46

9 Security 47

9.1 General 47

Annex A (normative): OpenAPI specification 48

A.1 General 48

A.2 ADAE_ServiceConfiguration API 48

Annex B (informative): Change history 59
