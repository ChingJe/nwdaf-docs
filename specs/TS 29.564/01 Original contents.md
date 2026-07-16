---
spec: TS 29.564
version: 18.7.0
release: '18'
clause: contents
title: Contents
source_archive: 29564-i70.zip
source_document: 29564-i70.docx
source_archive_sha256: dda2ab5dd327b415a1a334941e36923100eca4db73cc43a799ed5ccc529323f6
source_document_sha256: 9827ed4ca93c41f8771827e800e3be40fccfd30f2572ac9452c364163a144715
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Contents

Foreword 5

1 Scope 6

2 References 6

3 Definitions, symbols and abbreviations 7

3.1 Definitions 7

3.2 Symbols 7

3.3 Abbreviations 8

4 Overview 8

4.1 Introduction 8

5 Services offered by the UPF 9

5.1 Introduction 9

5.2 Nupf_EventExposure Service 9

5.2.1 Service Description 9

5.2.1.1 Service operations 9

5.2.1.2 Subscription to UPF events 10

5.2.1.3 UPF events supported by the Nupf_EventExposure service 10

5.2.1.3.1 General 10

5.2.1.3.2 QoS Monitoring 11

5.2.1.3.3 User Data Usage Measures 12

5.2.1.3.4 User Data Usage Trends 13

5.2.1.3.5 TSC Management Information 13

5.2.2 Service Operations 13

5.2.2.1 Introduction 13

5.2.2.2 Subscribe 14

5.2.2.2.1 General 14

5.2.2.2.2 Creation of a subscription 14

5.2.2.2.3 Modification of a subscription 16

5.2.2.2A Unsubscribe 17

5.2.2.2A.1 General 17

5.2.2.3 Notify 17

5.2.2.3.1 General 17

5.2.2.3.2 UPF sends notification on subscribed events 18

5.3 Nupf_GetUEPrivateIPaddrAndIdentifiers Service 19

5.3.1 Service Description 19

5.3.2 Service Operations 19

5.3.2.1 Introduction 19

5.3.2.2 Get 19

5.3.2.2.1 General 19

6 API Definitions 20

6.1 Nupf_EventExposure Service API 20

6.1.1 API URI 20

6.1.2 Usage of HTTP 20

6.1.2.1 General 20

6.1.2.2 HTTP standard headers 21

6.1.2.2.1 General 21

6.1.2.2.2 Content type 21

6.1.2.3 HTTP custom headers 21

6.1.3 Resources 21

6.1.3.1 Overview 21

6.1.3.2 Resource: EventExposureSubscriptions 22

6.1.3.2.1 Description 22

6.1.3.2.2 Resource Definition 22

6.1.3.2.3 Resource Standard Methods 22

6.1.3.2.4 Resource Custom Operations 23

6.1.3.3 Resource: Individual subscription 24

6.1.3.3.1 Description 24

6.1.3.3.2 Resource Definition 24

6.1.3.3.3 Resource Standard Methods 24

6.1.3.3.4 Resource Custom Operations 27

6.1.4 void 27

6.1.5 Notifications 27

6.1.5.1 General 27

6.1.5.2 Event Notification 27

6.1.5.2.1 Description 27

6.1.5.2.2 Target URI 27

6.1.6 Data Model 28

6.1.6.1 General 28

6.1.6.2 Structured data types 30

6.1.6.2.1 Introduction 30

6.1.6.2.2 Type: NotificationData 31

6.1.6.2.3 Type: NotificationItem 32

6.1.6.2.4 Type: QosMonitoringMeasurement 33

6.1.6.2.5 Type: UserDataUsageMeasurements 36

6.1.6.2.6 Type: VolumeMeasurement 37

6.1.6.2.7 Type: ThroughputMeasurement 37

6.1.6.2.8 Type: ApplicationRelatedInformation 37

6.1.6.2.9 Type: ThroughputStatisticsMeasurement 38

6.1.6.2.10 Type: DomainInformation 38

6.1.6.2.11 Type: UpfEventSubscription 39

6.1.6.2.12 Type: UpfEventMode 40

6.1.6.2.13 Type: UpfEvent 42

6.1.6.2.14 Type: CreateEventSubscription 43

6.1.6.2.15 Type: CreatedEventSubscription 43

6.1.6.2.16 Type: ReportingSuggestionInformation 43

6.1.6.2.17 Type: TscManagementInfo 43

6.1.6.3 Simple data types and enumerations 43

6.1.6.3.1 Introduction 43

6.1.6.3.2 Simple data types 44

6.1.6.3.3 Enumeration: EventType 44

6.1.6.3.4 Enumeration: UpfEventTrigger 44

6.1.6.3.5 Enumeration: MeasurementType 44

6.1.6.3.6 Enumeration: GranularityOfMeasurement 45

6.1.6.3.7 Enumeration: DnProtocol 45

6.1.6.3.8 Enumeration: ReportingUrgency 45

6.1.7 Error Handling 45

6.1.7.1 General 45

6.1.7.2 Protocol Errors 45

6.1.7.3 Application Errors 45

6.1.8 Feature negotiation 46

6.1.9 Security 46

6.1.10 HTTP redirection 46

6.2 Nupf_GetUEPrivateIPaddrAndIdentifiers Service API 47

6.2.1 Introduction 47

6.2.2 Usage of HTTP 47

6.2.2.1 General 47

6.2.2.2 HTTP standard headers 47

6.2.2.2.1 General 47

6.2.2.2.2 Content type 47

6.2.2.3 HTTP custom headers 47

6.2.3 Resources 48

6.2.3.1 Overview 48

6.2.3.2 Resource: UE IP Address Info 48

6.2.3.2.1 Description 48

6.2.3.2.2 Resource Definition 48

6.2.3.2.3 Resource Standard Methods 48

6.2.3.2.4 Resource Custom Operations 50

6.2.4 Custom Operations without associated resources 50

6.2.5 Notifications 50

6.2.5.1 General 50

6.2.6 Data Model 50

6.2.6.1 General 50

6.2.6.2 Structured data types 50

6.2.6.2.1 Introduction 50

6.2.6.2.2 Type: UeIpInfo 51

6.2.6.3 Simple data types and enumerations 51

6.2.6.3.1 Introduction 51

6.2.7 Error Handling 51

6.2.7.1 General 51

6.2.7.2 Protocol Errors 52

6.2.7.3 Application Errors 52

6.2.8 Feature negotiation 52

6.2.9 Security 52

6.2.10 HTTP redirection 52

Annex A (normative): OpenAPI specification 53

A.1 General 53

A.2 Nupf_EventExposure API 53

A.3 Nupf_GetUEPrivateIPaddrAndIdentifiers API 62

Annex B (informative): Change history 65
