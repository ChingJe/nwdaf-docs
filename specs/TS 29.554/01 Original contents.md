---
spec: TS 29.554
version: 18.3.0
release: '18'
clause: contents
title: Contents
source_archive: 29554-i30.zip
source_document: 29554-i30.doc
source_archive_sha256: 6c4b6afbda7aac02b47c89a700c2dd7c4166f3101a06fd9f3a540f586d287f15
source_document_sha256: 06837a9c2fc89dc80b80bbf8ee033acd765c9ddade475b2ddd9610099ccf6367
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

4 Background Data Transfer Policy Control Service 8

4.1 Service Description 8

4.1.1 Overview 8

4.1.2 Service Architecture 8

4.1.3 Network Functions 9

4.1.3.1 Policy Control Function (PCF) 9

4.1.3.2 NF Service Consumers 9

4.2 Service Operations 9

4.2.1 Introduction 9

4.2.2 Npcf_BDTPolicyControl_Create service operation 10

4.2.2.1 General 10

4.2.2.2 Creation of a BDT Policy 10

4.2.3 Npcf_BDTPolicyControl_Update service operation 12

4.2.3.1 General 12

4.2.3.2 Indication about selected transfer policy 12

4.2.3.3 Modification of BDT warning notification request indication 13

4.2.4 Npcf_BDTPolicyControl_Notify service operation 13

4.2.4.1 General 13

4.2.4.2 Sending the BDT warning notification 14

5 Npcf_BDTPolicyControl API 15

5.1 Introduction 15

5.2 Usage of HTTP 15

5.2.1 General 15

5.2.2 HTTP standard headers 15

5.2.2.1 General 15

5.2.2.2 Content type 16

5.2.3 HTTP custom headers 16

5.3 Resources 16

5.3.1 Resource Structure 16

5.3.2 Resource: BDT policies (Collection) 17

5.3.2.1 Description 17

5.3.2.2 Resource definition 17

5.3.2.3 Resource Standard Methods 17

5.3.2.3.1 POST 17

5.3.2.4 Resource Custom Operations 18

5.3.3 Resource: Individual BDT policy (Document) 18

5.3.3.1 Description 18

5.3.3.2 Resource definition 18

5.3.3.3 Resource Standard Methods 18

5.3.3.3.1 GET 18

5.3.3.3.2 PATCH 19

5.4 Custom Operations without associated resources 20

5.5 Notifications 20

5.5.1 General 20

5.5.2 BDT Notification 21

5.5.2.1 Description 21

5.5.2.2 Target URI 21

5.5.2.3 Standard Methods 21

5.5.2.3.1 POST 21

5.6 Data Model 22

5.6.1 General 22

5.6.2 Structured data types 23

5.6.2.1 Introduction 23

5.6.2.2 Type BdtPolicy 24

5.6.2.3 Type BdtReqData 25

5.6.2.4 Type BdtPolicyData 26

5.6.2.5 Type TransferPolicy 26

5.6.2.6 Type BdtPolicyDataPatch 26

5.6.2.7 Void 27

5.6.2.8 Type NetworkAreaInfo 27

5.6.2.9 Void 27

5.6.2.10 Type Notification 27

5.6.2.11 Type PatchBdtPolicy 27

5.6.2.12 Type BdtReqDataPatch 28

5.6.3 Simple data types and enumerations 28

5.6.3.1 Introduction 28

5.6.3.2 Simple data types 28

5.7 Error handling 28

5.7.1 General 28

5.7.2 Protocol Errors 28

5.7.3 Application Errors 28

5.8 Feature negotiation 29

5.9 Security 29

Annex A (normative): OpenAPI specification 30

A.1 General 30

A.2 Npcf_BDTPolicyControl API 30

Annex B (informative): Change history 37
