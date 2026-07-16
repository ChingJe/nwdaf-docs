---
spec: TS 29.575
version: 18.11.0
release: '18'
clause: contents
title: Contents
source_archive: 29575-ib0(1).zip
source_document: 29575-ib0.docx
source_archive_sha256: 8dd2c6efa5761d0f11035b07a1b3001be1a46d8a776e8999bcb8577be5a4ab08
source_document_sha256: dfbffaac1d4691129031cf34a08ba815f46f2cbb5b646d9694cf88b3383a714a
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Contents

Foreword 7

Introduction 8

1 Scope 9

2 References 9

3 Definitions, symbols and abbreviations 10

3.1 Definitions 10

3.2 Symbols 10

3.3 Abbreviations 10

4 Services offered by the ADRF 11

4.1 Introduction 11

4.2 Nadrf_DataManagement Service 12

4.2.1 Service Description 12

4.2.1.1 Overview 12

4.2.1.2 Service Architecture 12

4.2.1.3 Network Functions 13

4.2.1.3.1 Analytics Data Repository Function (ADRF) 13

4.2.1.3.2 NF Service Consumers 13

4.2.2 Service Operations 13

4.2.2.1 Introduction 13

4.2.2.2 Nadrf_DataManagement_StorageRequest service operation 14

4.2.2.2.1 General 14

4.2.2.2.2 Request Storage of data or analytics 14

4.2.2.3 Nadrf_DataManagement_StorageSubscriptionRequest service operation 15

4.2.2.3.1 General 15

4.2.2.3.2 Requesting subscription and storage of data or analytics 15

4.2.2.4 Nadrf_DataManagement_StorageSubscriptionRemoval service operation 16

4.2.2.4.1 General 16

4.2.2.4.2 Requesting removal of subscription of data or analytics 17

4.2.2.5 Nadrf_DataManagement_RetrievalRequest service operation 17

4.2.2.5.1 General 17

4.2.2.5.2 Request and get stored data or analytics from ADRF Data Store 17

4.2.2.6 Nadrf_DataManagement_RetrievalSubscribe service operation 19

4.2.2.6.1 General 19

4.2.2.6.2 Requesting retrieval and subscription of data or analytics 19

4.2.2.7 Nadrf_DataManagement_RetrievalUnsubscribe service operation 20

4.2.2.7.1 General 20

4.2.2.7.2 Requesting removal of retrieval subscription for data or analytics 20

4.2.2.8 Nadrf_DataManagement_RetrievalNotify service operation 21

4.2.2.8.1 General 21

4.2.2.8.2 Notification about subscribed data or analytics 21

4.2.2.8.3 Notification about data or analytics that are about to be deleted 22

4.2.2.9 Nadrf_DataManagement_Delete service operation 23

4.2.2.9.1 General 23

4.2.2.9.2 Requesting removal of stored data or analytics 23

4.2.2.9.3 Requesting removal of stored data or analytics using data or analytics specification 23

4.3 Nadrf\_ MLModelManagement Service 24

4.3.1 Service Description 24

4.3.1.1 Overview 24

4.3.1.2 Service Architecture 25

4.3.1.3 Network Functions 25

4.3.1.3.1 Analytics Data Repository Function (ADRF) 25

4.3.1.3.2 NF Service Consumers 25

4.3.2 Service Operations 26

4.3.2.1 Introduction 26

4.3.2.2 Nadrf_MLModelManagement_StorageRequest service operation 26

4.3.2.2.1 General 26

4.3.2.2.2 Request Storage of ML model(s) 26

4.3.2.2.3 Update Storage of ML model(s) 27

4.3.2.3 Nadrf_MLModelManagement_RetrievalRequest service operation 28

4.3.2.3.1 General 28

4.3.2.3.2 Request and get stored ML model(s) from ADRF ML Model Store 28

4.3.2.4 Nadrf_MLModelManagement_Delete service operation 29

4.3.2.4.1 General 29

4.3.2.4.2 Requesting removal of stored ML model(s) 29

4.3.2.4.3 Requesting removal of stored ML model(s) using unique ML model identifier 30

5 API Definitions 31

5.1 Nadrf_DataManagement Service API 31

5.1.1 Introduction 31

5.1.2 Usage of HTTP 31

5.1.2.1 General 31

5.1.2.2 HTTP standard headers 31

5.1.2.2.1 General 31

5.1.2.2.2 Content type 31

5.1.2.3 HTTP custom headers 31

5.1.3 Resources 32

5.1.3.1 Overview 32

5.1.3.2 Resource: ADRF Data Store Records 32

5.1.3.2.1 Description 32

5.1.3.2.2 Resource Definition 32

5.1.3.2.3 Resource Standard Methods 33

5.1.3.2.4 Resource Custom Operations 35

5.1.3.3 Resource: Individual ADRF Data Store Record 35

5.1.3.3.1 Description 35

5.1.3.3.2 Resource Definition 35

5.1.3.3.3 Resource Standard Methods 35

5.1.3.3.4 Resource Custom Operations 36

5.1.3.4 Resource: ADRF Data Retrieval Subscriptions 36

5.1.3.4.1 Description 36

5.1.3.4.2 Resource Definition 36

5.1.3.4.3 Resource Standard Methods 37

5.1.3.4.4 Resource Custom Operations 37

5.1.3.5 Resource: Individual ADRF Data Retrieval Subscription 38

5.1.3.5.1 Description 38

5.1.3.5.2 Resource Definition 38

5.1.3.5.3 Resource Standard Methods 38

5.1.3.5.4 Resource Custom Operations 39

5.1.4 Custom Operations without associated resources 39

5.1.4.1 Overview 39

5.1.4.2 Operation: request-storage-sub 40

5.1.4.2.1 Description 40

5.1.4.2.2 Operation Definition 40

5.1.4.3 Operation: request-storage-sub-removal 41

5.1.4.3.1 Description 41

5.1.4.3.2 Operation Definition 41

5.1.4.4 Operation: remove-stored-data-analytics 42

5.1.4.4.1 Description 42

5.1.4.4.2 Operation Definition 42

5.1.5 Notifications 43

5.1.5.1 General 43

5.1.5.2 Retrieval Notification 44

5.1.5.2.1 Description 44

5.1.5.2.2 Target URI 44

5.1.5.2.3 Standard Methods 44

5.1.5.3 ADRF Alert Notification 45

5.1.5.3.1 Description 45

5.1.5.3.2 Target URI 45

5.1.5.3.3 Standard Methods 45

5.1.6 Data Model 46

5.1.6.1 General 46

5.1.6.2 Structured data types 50

5.1.6.2.1 Introduction 50

5.1.6.2.2 Type: NadrfDataStoreRecord 50

5.1.6.2.3 Type: NadrfDataStoreSubscription 51

5.1.6.2.4 Type: NadrfDataRetrievalSubscription 52

5.1.6.2.5 Type: NadrfDataRetrievalNotification 53

5.1.6.2.6 Type: NadrfDataStoreSubscriptionRef 53

5.1.6.2.7 Type: NadrfStoredDataSpec 54

5.1.6.2.8 Type: DataSubscription 54

5.1.6.2.9 Type: DataNotification 55

5.1.6.2.10 Type: StorageHandlingInfo 55

5.1.6.2.11 Type: NadrfAlertNotification 56

5.1.6.2.12 Type: NadrfAlertNotificationResponse 56

5.1.6.2.13 Type: DataSetTag 56

5.1.6.3 Simple data types and enumerations 56

5.1.6.4 Data types describing alternative data types or combinations of data types 56

5.1.7 Error Handling 56

5.1.7.1 General 56

5.1.7.2 Protocol Errors 57

5.1.7.3 Application Errors 57

5.1.8 Feature negotiation 57

5.1.9 Security 57

5.2 Nadrf_MLModelManagement Service API 58

5.2.1 Introduction 58

5.2.2 Usage of HTTP 58

5.2.2.1 General 58

5.2.2.2 HTTP standard headers 58

5.2.2.2.1 General 58

5.2.2.2.2 Content type 58

5.2.2.3 HTTP custom headers 58

5.2.3 Resources 59

5.2.3.1 Overview 59

5.2.3.2 Resource: ADRF ML Model Store Records 59

5.2.3.2.1 Description 59

5.2.3.2.2 Resource Definition 59

5.2.3.2.3 Resource Standard Methods 60

5.2.3.2.4 Resource Custom Operations 62

5.2.3.3 Resource: Individual ADRF ML Model Store Record 62

5.2.3.3.1 Description 62

5.2.3.3.2 Resource Definition 62

5.2.3.3.3 Resource Standard Methods 62

5.2.3.3.4 Resource Custom Operations 64

5.2.4 Custom Operations without associated resources 64

5.2.4.1 Overview 64

5.2.4.4 Operation: remove-stored-mlmodel 65

5.2.4.4.1 Description 65

5.2.4.4.2 Operation Definition 65

5.2.5 Notifications 66

5.2.6 Data Model 67

5.2.6.1 General 67

5.2.6.2 Structured data types 67

5.2.6.2.1 Introduction 67

5.2.6.2.2 Type: NadrfMLModelStoreRecord 68

5.2.6.2.3 Type: MLModelInfo 68

5.2.6.2.4 Type: MLModel 68

5.2.6.2.5 Type: MLModelDelResult 68

5.2.6.2.6 Type: AllowedConsumer 69

5.2.6.2.7 Type: ModelStoreResult 69

5.2.6.3 Simple data types and enumerations 69

5.2.6.3.2 Enumeration: StoreResult 69

5.2.6.4 Data types describing alternative data types or combinations of data types 69

5.2.7 Error Handling 69

5.2.7.1 General 69

5.2.7.2 Protocol Errors 70

5.2.7.3 Application Errors 70

5.2.8 Feature negotiation 70

5.2.9 Security 70

Annex A (normative): OpenAPI specification 71

A.1 General 71

A.2 Nadrf_DataManagement API 71

A.3 Nadrf_MLModelManagement API 83

Annex B (informative): Change history 90
