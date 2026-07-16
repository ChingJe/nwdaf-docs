---
spec: TS 29.508
version: 18.11.0
release: '18'
clause: contents
title: Contents
source_archive: 29508-ib0.zip
source_document: 29508-ib0.doc
source_archive_sha256: daf27642c0ec0e9c9253524bf6c70b78a6b540ce49abd07b28f730c3b6f0a667
source_document_sha256: 8e4c066907e4b69f9c3747664365b2d145ba149649f162eb8dd40601f5ad332e
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Contents

Foreword 5

1 Scope 6

2 References 6

3 Definitions and abbreviations 7

3.1 Definitions 7

3.2 Abbreviations 7

4 Session Management Event Exposure Service 8

4.1 Service Description 8

4.1.1 Overview 8

4.1.2 Service Architecture 9

4.1.3 Network Functions 10

4.1.3.1 Session Management Function (SMF) 10

4.1.3.2 NF Service Consumers 10

4.2 Service Operations 11

4.2.1 Introduction 11

4.2.2 Nsmf_EventExposure_Notify Service Operation 11

4.2.2.1 General 11

4.2.2.2 Notification about subscribed events 11

4.2.3 Nsmf_EventExposure_Subscribe Service Operation 17

4.2.3.1 General 17

4.2.3.2 Creating a new subscription 17

4.2.3.3 Modifying an existing subscription 20

4.2.4 Nsmf_EventExposure_UnSubscribe Service Operation 22

4.2.4.1 General 22

4.2.4.2 Unsubscription from event notifications 22

4.2.5 Nsmf_EventExposure_AppRelocationInfo Service Operation 23

4.2.5.1 General 23

4.2.5.2 Acknowledgement of Notification about subscribed events 23

5 Nsmf_EventExposure API 24

5.1 Introduction 24

5.2 Usage of HTTP 24

5.2.1 General 24

5.2.2 HTTP standard headers 24

5.2.2.1 General 24

5.2.2.2 Content type 24

5.2.3 HTTP custom headers 25

5.3 Resources 25

5.3.1 Resource Structure 25

5.3.2 Resource: SMF Notification Subscriptions 25

5.3.2.1 Description 25

5.3.2.2 Resource definition 25

5.3.2.3 Resource Standard Methods 26

5.3.2.3.1 POST 26

5.3.2.4 Resource Custom Operations 26

5.3.3 Resource: Individual SMF Notification Subscription 26

5.3.3.1 Description 26

5.3.3.2 Resource definition 27

5.3.3.3 Resource Standard Methods 27

5.3.3.3.1 GET 27

5.3.3.3.2 PUT 28

5.3.3.3.3 DELETE 29

5.3.3.4 Resource Custom Operations 31

5.4 Custom Operations without associated resources 31

5.5 Notifications 31

5.5.1 General 31

5.5.2 Event Notification 31

5.5.2.1 Description 31

5.5.2.2 Target URI 31

5.5.2.3 Standard Methods 31

5.5.2.3.1 POST 31

5.5.3 Acknowledgement of event notification 33

5.5.3.1 Description 33

5.5.3.2 Target URI 33

5.5.3.3 Standard Methods 33

5.5.3.3.1 POST 33

5.6 Data Model 34

5.6.1 General 34

5.6.2 Structured data types 37

5.6.2.1 Introduction 37

5.6.2.2 Type NsmfEventExposure 38

5.6.2.3 Type NsmfEventExposureNotification 41

5.6.2.4 Type EventSubscription 42

5.6.2.5 Type EventNotification 43

5.6.2.6 void. 48

5.6.2.7 Type AckOfNotify 48

5.6.2.8 Type SmNasFromUe 48

5.6.2.9 Type SmNasFromSmf 48

5.6.2.10 Type TransactionInfo 49

5.6.2.11 Type PduSessionInformation 49

5.6.2.12 Type PduSessionInfo 49

5.6.2.13 Type UpfInformation 49

5.6.2.14 Type: TrafficCorrelationNotification 50

5.6.3 Simple data types and enumerations 50

5.6.3.1 Introduction 50

5.6.3.2 Simple data types 50

5.6.3.3 Enumeration: SmfEvent 51

5.6.3.4 Enumeration: NotificationMethod 51

5.6.3.5 void. 52

5.6.3.6 Enumeration: AppliedSmccType 52

5.6.3.7 Enumeration: TransactionMetric 52

5.6.3.8 Enumeration: PduSessionStatus 52

5.7 Error handling 52

5.7.1 General 52

5.7.2 Protocol Errors 52

5.7.3 Application Errors 52

5.8 Feature negotiation 53

5.9 Security 55

Annex A (normative): OpenAPI specification 56

A.1 General 56

A.2 Nsmf_EventExposure API 56

Annex B (informative): Change history 70
