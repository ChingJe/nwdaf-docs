---
spec: TS 29.574
version: 18.12.0
release: '18'
clause: contents
title: Contents
source_archive: 29574-ic0.zip
source_document: 29574-ic0.docx
source_archive_sha256: 998a8efc37e75c103b3e3dbd24941467f1b2ebe283ad8e840937a2f001283e90
source_document_sha256: 41675685db51eca2cded1d9e3ad86a6e0a08c3b6fecbdc89210de19332efafa6
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Contents

Foreword 7

1 Scope 9

2 References 9

3 Definitions, symbols and abbreviations 10

3.1 Definitions 10

3.2 Symbols 10

3.3 Abbreviations 10

4 Services offered by the DCCF 11

4.1 Introduction 11

4.2 Ndccf_DataManagement Service 11

4.2.1 Service Description 11

4.2.1.1 Overview 11

4.2.1.2 Service Architecture 12

4.2.1.3 Network Functions 13

4.2.1.3.1 Data Collection Coordination Function (DCCF) 13

4.2.1.3.2 NF Service Consumers 13

4.2.2 Service Operations 13

4.2.2.1 Introduction 13

4.2.2.2 Ndccf_DataManagement_Subscribe service operation 14

4.2.2.2.1 General 14

4.2.2.2.2 Subscription for analytics notifications 14

4.2.2.2.3 Update subscription for analytic notifications 16

4.2.2.2.4 Subscription for data notifications 18

4.2.2.2.5 Update subscription for data notifications 20

4.2.2.3 Ndccf_DataManagement_Unsubscribe service operation 22

4.2.2.3.1 General 22

4.2.2.3.2 Unsubscribe from analytics notifications 22

4.2.2.3.3 Unsubscribe from data notifications 23

4.2.2.4 Ndccf_DataManagement_Notify service operation 23

4.2.2.4.1 General 23

4.2.2.4.2 Notification about subscribed analytics 23

4.2.2.4.3 Notification about subscribed data event 25

4.2.2.5 Ndccf_DataManagement_Fetch service operation 26

4.2.2.5.1 General 26

4.2.2.5.2 Retrieve notified analytics and data 26

4.2.2.6 Ndccf_DataManagement_Transfer service operation 27

4.2.2.6.1 General 27

4.2.2.6.2 Request for UE data subscription context transfer 27

4.2.2.6.3 Void 28

4.2.2.6.4 Void 28

4.3 Ndccf_ContextManagement Service 28

4.3.1 Service Description 28

4.3.1.1 Overview 28

4.3.1.2 Service Architecture 28

4.3.1.3 Network Functions 29

4.3.1.3.1 Data Collection Coordination Function (DCCF) 29

4.3.1.3.2 NF Service Consumers 29

4.3.2 Service Operations 29

4.3.2.1 Introduction 29

4.3.2.2 Ndccf_ContextManagement_Register service operation 29

4.3.2.2.1 General 29

4.3.2.2.2 Register data collection profile to DCCF 29

4.3.2.3 Ndccf_ContextManagement_Update service operation 31

4.3.2.3.1 General 31

4.3.2.3.2 Update registered data collection profile 31

4.3.2.4 Ndccf_ContextManagement_Deregister service operation 32

4.3.2.4.1 General 32

4.3.2.4.2 Deregister Data collection profile 32

5 API Definitions 32

5.1 Ndccf_DataManagement Service API 32

5.1.1 Introduction 32

5.1.2 Usage of HTTP 33

5.1.2.1 General 33

5.1.2.2 HTTP standard headers 33

5.1.2.2.1 General 33

5.1.2.2.2 Content type 33

5.1.2.3 HTTP custom headers 33

5.1.3 Resources 33

5.1.3.1 Overview 33

5.1.3.2 Resource: DCCF Analytics Subscriptions 34

5.1.3.2.1 Description 34

5.1.3.2.2 Resource Definition 34

5.1.3.2.3 Resource Standard Methods 35

5.1.3.2.4 Resource Custom Operations 35

5.1.3.3 Resource: Individual DCCF Analytics Subscription 35

5.1.3.3.1 Description 35

5.1.3.3.2 Resource Definition 35

5.1.3.3.3 Resource Standard Methods 36

5.1.3.3.4 Resource Custom Operations 38

5.1.3.4 Resource: DCCF Data Subscriptions 38

5.1.3.4.1 Description 38

5.1.3.4.2 Resource Definition 38

5.1.3.4.3 Resource Standard Methods 39

5.1.3.4.4 Resource Custom Operations 39

5.1.3.5 Resource: Individual DCCF Data Subscription 40

5.1.3.5.1 Description 40

5.1.3.5.2 Resource Definition 40

5.1.3.5.3 Resource Standard Methods 40

5.1.3.5.4 Resource Custom Operations 42

5.1.3.6 Void 43

5.1.3.7 Void 43

5.1.4 Custom Operations without associated resources 43

5.1.4.1 Overview 43

5.1.4.2 Operation: transfer-data-sub 43

5.1.4.2.1 Description 43

5.1.4.2.2 Operation Definition 43

5.1.5 Notifications 44

5.1.5.1 General 44

5.1.5.2 Analytics Notification 44

5.1.5.2.1 Description 44

5.1.5.2.2 Target URI 44

5.1.5.2.3 Standard Methods 44

5.1.5.3 Data Notification 45

5.1.5.3.1 Description 45

5.1.5.3.2 Target URI 45

5.1.5.3.3 Standard Methods 46

5.1.5.4 Fetch Notification 47

5.1.5.4.1 Description 47

5.1.5.4.2 Target URI 47

5.1.5.4.3 Standard Methods 47

5.1.6 Data Model 48

5.1.6.1 General 48

5.1.6.2 Structured data types 51

5.1.6.2.1 Introduction 51

5.1.6.2.2 Type NdccfAnalyticsSubscription 52

5.1.6.2.3 Type NdccfDataSubscription 55

5.1.6.2.4 Type NdccfAnalyticsSubscriptionNotification 58

5.1.6.2.5 Type NdccfDataSubscriptionNotification 59

5.1.6.2.6 Type FormattingInstruction 61

5.1.6.2.7 Type ProcessingInstruction 62

5.1.6.2.8 Type ParameterProcessingInstruction 63

5.1.6.2.9 Type NotifSummaryReport 63

5.1.6.2.10 Type EventParamReport 64

5.1.6.2.11 Type ReportingOptions 66

5.1.6.2.12 Void 67

5.1.6.2.13 Type DccfEvent 67

5.1.6.2.14 Type NotifyEndpoint 67

5.1.6.2.15 Type: StorageHandlingInformation 67

5.1.6.2.16 Type: DeletionAlert 68

5.1.6.2.17 Type: NotifResponse 68

5.1.6.2.18 Void 68

5.1.6.2.19 Type: DataTransferResp 68

5.1.6.3 Simple data types and enumerations 68

5.1.6.3.1 Introduction 68

5.1.6.3.2 Simple data types 68

5.1.6.3.3 Enumeration: SummarizationAttribute 69

5.1.6.3.4 Enumeration: AggregationLevel 69

5.1.6.3.5 Enumeration: DataCollectionPurpose 69

5.1.6.3.6 Enumeration: TermCause 69

5.1.6.4 Data types describing alternative data types or combinations of data types 69

5.1.6.5 Binary data 70

5.1.7 Error Handling 70

5.1.7.1 General 70

5.1.7.2 Protocol Errors 70

5.1.7.3 Application Errors 70

5.1.8 Feature negotiation 70

5.1.9 Security 71

5.2 Ndccf_ContextManagement Service API 71

5.2.1 Introduction 71

5.2.2 Usage of HTTP 72

5.2.2.1 General 72

5.2.2.2 HTTP standard headers 72

5.2.2.2.1 General 72

5.2.2.2.2 Content type 72

5.2.2.3 HTTP custom headers 72

5.2.3 Resources 72

5.2.3.1 Overview 72

5.2.3.2 Resource: DCCF Data Collection Profiles 73

5.2.3.2.1 Description 73

5.2.3.2.2 Resource Definition 73

5.2.3.2.3 Resource Standard Methods 73

5.2.3.2.4 Resource Custom Operations 74

5.2.3.3 Resource: Individual DCCF Data Collection Profile 74

5.2.3.3.1 Description 74

5.2.3.3.2 Resource Definition 74

5.2.3.3.3 Resource Standard Methods 74

5.2.3.3.4 Resource Custom Operations 77

5.2.4 Custom Operations without associated resources 77

5.2.5 Notifications 77

5.2.6 Data Model 77

5.2.6.1 General 77

5.2.6.2 Structured data types 77

5.2.6.2.1 Introduction 77

5.2.6.2.2 Type: NdccfDataCollectionProfile 78

5.2.6.3 Simple data types and enumerations 78

5.2.6.3.1 Introduction 78

5.2.6.3.2 Simple data types 78

5.2.6.4 Data types describing alternative data types or combinations of data types 79

5.2.6.5 Binary data 79

5.2.7 Error Handling 79

5.2.7.1 General 79

5.2.7.2 Protocol Errors 79

5.2.7.3 Application Errors 79

5.2.8 Feature negotiation 79

5.2.9 Security 79

Annex A (normative): OpenAPI specification 80

A.1 General 80

A.2 Ndccf_DataManagement API 80

A.3 Ndccf_ContextManagement API 95

Annex B (informative): Change history 99
