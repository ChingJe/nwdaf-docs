---
spec: TS 29.576
version: 18.8.0
release: '18'
clause: contents
title: Contents
source_archive: 29576-i80.zip
source_document: 29576-i80.docx
source_archive_sha256: 4b70ed06646c84f7f9c47958cd2d64fe6c6ff89448ac6db4679c4b1de6a598c6
source_document_sha256: 38b6e8bd8f4ff1104b4b2ec5d475a7edb65e4330ee6cad4536abd2a24e7943b1
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Contents

Foreword 6

Introduction 7

1 Scope 8

2 References 8

3 Definitions, symbols and abbreviations 9

3.1 Definitions 9

3.2 Symbols 9

3.3 Abbreviations 9

4 Services offered by the MFAF 9

4.1 Introduction 9

4.2 Nmfaf_3daDataManagement Service 10

4.2.1 Service Description 10

4.2.1.1 Overview 10

4.2.1.2 Service Architecture 10

4.2.1.3 Network Functions 11

4.2.1.3.1 Messaging Framework Adaptor Function (MFAF) 11

4.2.1.3.2 NF Service Consumers 11

4.2.2 Service Operations 12

4.2.2.1 Introduction 12

4.2.2.2 Nmfaf_3daDataManagement_Configure service operation 12

4.2.2.2.1 General 12

4.2.2.2.2 Initial configuration for mapping data or analytics 12

4.2.2.2.3 Update the configuration of existing individual mapping data or analytics 13

4.2.2.3 Nmfaf_3daDataManagement_Deconfigure service operation 14

4.2.2.3.1 General 14

4.2.2.3.2 Stop mapping data or analytics 14

4.3 Nmfaf_3caDataManagement Service 15

4.3.1 Service Description 15

4.3.1.1 Overview 15

4.3.1.2 Service Architecture 15

4.3.1.3 Network Functions 16

4.3.1.3.1 Messaging Framework Adaptor Function (MFAF) 16

4.3.1.3.2 NF Service Consumers 16

4.3.2 Service Operations 16

4.3.2.1 Introduction 16

4.3.2.2 Nmfaf_3caDataManagement_Fetch service operation 16

4.3.2.2.1 General 16

4.3.2.2.2 Retrieve data or analytics from the MFAF 17

4.3.2.2A Nmfaf_3caDataManagement_Subscribe service operation 17

4.3.2.3 Nmfaf_3caDataManagement_Notify service operation 17

4.3.2.3.1 General 17

4.3.2.3.2 Notification about the subscribed data or analytics 17

5 API Definitions 19

5.1 Nmfaf_3daDataManagement Service API 19

5.1.1 Introduction 19

5.1.2 Usage of HTTP 19

5.1.2.1 General 19

5.1.2.2 HTTP standard headers 19

5.1.2.2.1 General 19

5.1.2.2.2 Content type 19

5.1.2.3 HTTP custom headers 19

5.1.3 Resources 20

5.1.3.1 Overview 20

5.1.3.2 Resource: MFAF Configurations 20

5.1.3.2.1 Description 20

5.1.3.2.2 Resource Definition 20

5.1.3.2.3 Resource Standard Methods 21

5.1.3.2.4 Resource Custom Operations 21

5.1.3.3 Resource: Individual MFAF Configuration 21

5.1.3.2.1 Description 21

5.1.3.3.2 Resource Definition 21

5.1.3.3.3 Resource Standard Methods 22

5.1.4 Custom Operations without associated resources 24

5.1.5 Notifications 24

5.1.6 Data Model 24

5.1.6.1 General 24

5.1.6.2 Structured data types 25

5.1.6.2.1 Introduction 25

5.1.6.2.2 Type: MfafConfiguration 25

5.1.6.2.3 Type: MessageConfiguration 26

5.1.6.2.4 Type: MfafNotiInfo 26

5.1.6.3 Simple data types and enumerations 27

5.1.6.3.1 Introduction 27

5.1.6.3.2 Simple data types 27

5.1.6.4 Data types describing alternative data types or combinations of data types 27

5.1.6.5 Binary data 27

5.1.7 Error Handling 27

5.1.7.1 General 27

5.1.7.2 Protocol Errors 27

5.1.7.3 Application Errors 27

5.1.8 Feature negotiation 27

5.1.9 Security 28

5.2 Nmfaf_3caDataManagement Service API 28

5.2.1 Introduction 28

5.2.2 Usage of HTTP 28

5.2.2.1 General 28

5.2.2.2 HTTP standard headers 29

5.2.2.2.1 General 29

5.2.2.2.2 Content type 29

5.2.2.3 HTTP custom headers 29

5.2.3 Resources 29

5.2.4 Custom Operations without associated resources 29

5.2.5 Notifications 29

5.2.5.1 General 29

5.2.5.2 MFAF Notification 29

5.2.5.2.1 Description 29

5.2.5.2.2 Target URI 30

5.2.5.2.3 Standard Methods 30

5.2.5.3 Fetch Notification 31

5.2.5.3.1 Description 31

5.2.5.3.2 Target URI 31

5.2.5.3.3 Standard Methods 31

5.2.6 Data Model 32

5.2.6.1 General 32

5.2.6.2 Structured data types 33

5.2.6.2.1 Introduction 33

5.2.6.2.2 Type: NmfafDataRetrievalNotification 34

5.2.6.2.3 Type: FetchInstruction 34

5.2.6.2.4 Type: NmfafDataAnaNotification 35

5.2.6.3 Simple data types and enumerations 35

5.2.6.3.1 Introduction 35

5.2.6.3.2 Simple data types 35

5.2.6.4 Data types describing alternative data types or combinations of data types 35

5.2.6.5 Binary data 35

5.2.7 Error Handling 35

5.2.7.1 General 35

5.2.7.2 Protocol Errors 35

5.2.7.3 Application Errors 35

5.2.8 Feature negotiation 36

5.2.9 Security 36

Annex A (normative): OpenAPI specification 37

A.1 General 37

A.2 Nmfaf_3daDataManagement API 37

A.3 Nmfaf_3caDataManagement API 40

Annex B (informative): Change history 44
