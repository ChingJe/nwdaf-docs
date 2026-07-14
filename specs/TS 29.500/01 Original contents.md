---
spec: TS 29.500
version: 18.10.0
release: '18'
clause: contents
title: Contents
source_archive: 29500-ia0.zip
source_document: 29500-ia0.docx
source_archive_sha256: 3e0cbb6bc6ceec4dabb597410fb4b55ba03ae82e9b242972f441cd926971f5f2
source_document_sha256: 8cd4994735a1718db905b0aee4ceba4fb479cc7193d82d051a68450f33eba52b
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Contents


Foreword 8

1 Scope 9

2 References 9

3 Definitions and abbreviations 11

3.1 Definitions 11

3.2 Abbreviations 12

3.3 Special characters, operators and delimiters 12

3.3.1 General 12

3.3.2 ABNF operators 12

3.3.3 URI – reserved and special characters 12

3.3.4 SBI specific usage of delimiters 12

4 Service Based Architecture Overview 13

4.1 NF Services 13

4.2 Service Based Interfaces 13

4.3 NF Service Framework 13

4.3.1 General 13

4.3.2 NF Service Advertisement URI 13

5 Protocols Over Service Based Interfaces 14

5.1 Protocol Stack Overview 14

5.2 HTTP/2 Protocol 14

5.2.1 General 14

5.2.2 HTTP standard headers 14

5.2.2.1 General 14

5.2.2.2 Mandatory to support HTTP standard headers 15

5.2.3 HTTP custom headers 18

5.2.3.1 General 18

5.2.3.2 Mandatory to support custom headers 19

5.2.3.2.1 General 19

5.2.3.2.2 3gpp-Sbi-Message-Priority 24

5.2.3.2.3 3gpp-Sbi-Callback 24

5.2.3.2.4 3gpp-Sbi-Target-apiRoot 25

5.2.3.2.5 3gpp-Sbi-Routing-Binding 25

5.2.3.2.6 3gpp-Sbi-Binding 26

5.2.3.2.7 3gpp-Sbi-Discovery 30

5.2.3.2.8 3gpp-Sbi-Producer-Id 31

5.2.3.2.9 3gpp-Sbi-Oci 32

5.2.3.2.10 3gpp-Sbi-Lci 34

5.2.3.2.11 3gpp-Sbi-Client-Credentials 36

5.2.3.2.12 3gpp-Sbi-Nrf-Uri 37

5.2.3.2.13 3gpp-Sbi-Target-Nf-Id 38

5.2.3.2.14 3gpp-Sbi-Max-Forward-Hops 38

5.2.3.2.15 3gpp-Sbi-Originating-Network-Id 39

5.2.3.2.16 3gpp-Sbi-Access-Scope 39

5.2.3.2.17 3gpp-Sbi-Access-Token 40

5.2.3.2.18 Void 40

5.2.3.2.19 3gpp-Sbi-Target-Nf-Group-Id 40

5.2.3.2.20 3gpp-Sbi-Nrf-Uri-Callback 40

5.2.3.2.21 3gpp-Sbi-NF-Peer-Info 41

5.2.3.2.22 3gpp-Sbi-Source-NF-Client-Credentials 42

5.2.3.3 Optional to support custom headers 42

5.2.3.3.1 General 42

5.2.3.3.2 3gpp-Sbi-Sender-Timestamp 45

5.2.3.3.3 3gpp-Sbi-Max-Rsp-Time 45

5.2.3.3.4 3gpp-Sbi-Correlation-Info 45

5.2.3.3.5 3gpp-Sbi-Alternate-Chf-Id 47

5.2.3.3.6 3gpp-Sbi-Notif-Accepted-Encoding 47

5.2.3.3.7 3gpp-Sbi-Consumer-Info 47

5.2.3.3.8 3gpp-Sbi-Response-Info 49

5.2.3.3.9 Void 50

5.2.3.3.10 3gpp-Sbi-Selection-Info 50

5.2.3.3.11 3gpp-Sbi-Interplmn-Purpose 51

5.2.3.3.12 3gpp-Sbi-Request-Info 52

5.2.3.3.13 3gpp-Sbi-Retry-Info 53

5.2.3.3.14 3gpp-Sbi-Other-Access-Scopes 54

5.2.4 HTTP error handling 54

5.2.5 HTTP/2 server push 54

5.2.6 HTTP/2 connection management 55

5.2.7 HTTP status codes 55

5.2.7.1 General 55

5.2.7.2 NF as HTTP Server 56

5.2.7.3 NF as HTTP Client 61

5.2.7.4 SCP/SEPP 62

5.2.8 HTTP/2 request retries 65

5.2.9 Handling of unsupported query parameters 66

5.2.10 URL Encoding of data 67

5.2.10.1 General 67

5.2.10.2 URL Encoding of URI components 67

5.2.10.3 URL Encoding of HTTP/2 request bodies 68

5.3 Transport Protocol 68

5.4 Serialization Protocol 69

5.5 Interface Definition Language 69

6 General Functionalities in Service Based Architecture 69

6.1 Routing Mechanisms 69

6.1.1 General 69

6.1.2 Identifying a target resource 69

6.1.3 Connecting inbound 69

6.1.4 Pseudo-header setting 70

6.1.4.1 General 70

6.1.4.2 Routing within a PLMN 70

6.1.4.3 Routing across PLMN 70

6.1.4.3.1 General 70

6.1.4.3.2 Use of telescopic FQDN between NFs and SEPP within a PLMN 71

6.1.4.3.3 Use of 3gpp-Sbi-Target-apiRoot between NFs and SEPP within a PLMN 71

6.1.4.3.4 Routing between SEPPs 72

6.1.5 Host header 72

6.1.6 Message forwarding 72

6.2 Server-Initiated Communication 73

6.2.1 General 73

6.2.2 Subscription on behalf of NF Service Consumer 73

6.2.3 Notification error handling 74

6.3 Load Control 74

6.3.1 General 74

6.3.2 Load Control based on load signalled via the NRF 74

6.3.3 Load Control based on LCI Header 75

6.3.3.1 General 75

6.3.3.2 Conveyance of Load Control Information 75

6.3.3.3 Frequency of Conveyance 76

6.3.3.4 Load Control Information 76

6.3.3.4.1 General Description 76

6.3.3.4.2 Load Control Timestamp 76

6.3.3.4.3 Load Metric 77

6.3.3.4.4 Scope of LCI 77

6.3.3.4.5 S-NSSAI/DNN Relative Capacity 79

6.3.3.5 LC-H feature support 79

6.3.3.5.1 Indicating supports for the LC-H feature 79

6.3.3.5.2 Utilizing the LC-H feature support indication 79

6.4 Overload Control 79

6.4.1 General 79

6.4.2 Overload Control based on HTTP status codes 80

6.4.2.1 General 80

6.4.2.2 HTTP Status Code "503 Service Unavailable" 80

6.4.2.3 HTTP Status Code "429 Too Many Requests" 81

6.4.2.4 HTTP Status Code "307 Temporary Redirect" 81

6.4.3 Overload Control based on OCI Header 81

6.4.3.1 General 81

6.4.3.2 Conveyance of Overload Control Information 82

6.4.3.3 Frequency of Conveyance 82

6.4.3.4 Overload Control Information 82

6.4.3.4.1 General Description 82

6.4.3.4.2 Overload Control Timestamp 83

6.4.3.4.3 Overload Reduction Metric 84

6.4.3.4.4 Overload Control Period of Validity 84

6.4.3.4.5 Scope of OCI 84

6.4.3.5 Overload Control Enforcement 88

6.4.3.5.1 Message Throttling 88

6.4.3.5.2 Loss Algorithm 89

6.4.3.6 OLC-H feature support 89

6.4.3.6.1 Indicating supports for the OLC-H feature 89

6.5 Support of Stateless NFs 89

6.5.1 General 89

6.5.2 Stateless AMFs 89

6.5.2.1 General 89

6.5.2.2 AMF as service consumer 89

6.5.2.3 AMF as service producer 90

6.5.3 Stateless NFs (for any 5GC NF type) 91

6.5.3.1 General 91

6.5.3.2 Stateless NF as service consumer 91

6.5.3.3 Stateless NF as service producer 93

6.6 Extensibility Mechanisms 94

6.6.1 General 94

6.6.2 Feature negotiation 94

6.6.3 Vendor-specific extensions 95

6.6.4 Extensibility for Query parameters 96

6.7 Security Mechanisms 97

6.7.1 General 97

6.7.2 Transport layer security protection of messages 97

6.7.3 Authorization of NF service access 97

6.7.4 Application layer security across PLMN 98

6.7.4.1 General 98

6.7.4.2 N32 Procedures 98

6.7.5 Client credentials assertion and authentication 99

6.7.5.1 General 99

6.7.5.2 Authorization of NF Service Consumers for data access via DCCF 99

6.7.5.3 Authorization of requesting ML models on behalf of another ML model consumer 100

6.8 SBI Message Priority Mechanism 100

6.8.1 General 100

6.8.2 Message level priority 100

6.8.3 Stream priority 101

6.8.4 Recommendations when defining SBI Message Priorities 101

6.8.5 HTTP/2 client behaviour 102

6.8.6 HTTP/2 server behaviour 102

6.8.7 HTTP/2 proxy behaviour 102

6.8.8 DSCP marking of messages 103

6.9 Discovering the supported communication options 103

6.9.1 General 103

6.9.2 Discoverable communication options 103

6.9.2.1 Content-encodings supported in HTTP requests 103

6.9.2.2 Content-encodings supported in HTTP responses 104

6.10 Support of Indirect Communication 104

6.10.1 General 104

6.10.2 Routing Mechanisms with SCP Known to the NF 104

6.10.2.1 General 104

6.10.2.2 HTTP/2 connection management 105

6.10.2.3 Connecting inbound 105

6.10.2.4 Pseudo-header setting 105

6.10.2.5 3gpp-Sbi-Target-apiRoot header setting 107

6.10.2.6 Cache key (ck) query parameter 107

6.10.2A Routing Mechanism with SCP Unknown to the NF 108

6.10.2A.1 Connecting inbound 108

6.10.2A.2 HTTP/2 connection management 108

6.10.2A.3 Pseudo-header setting 108

6.10.3 NF Discovery and Selection for indirect communication with Delegated Discovery 108

6.10.3.1 General 108

6.10.3.2 Conveyance of NF Discovery Factors 108

6.10.3.3 Notifications corresponding to default notification subscriptions 110

6.10.3.4 Returning the Producer's NF Instance ID and NF Group ID to the NF Service Consumer 110

6.10.3.5 Returning an Alternate CHF instance ID to the NF Service Consumer 111

6.10.4 Authority and/or deployment-specific string in apiRoot of resource URI 111

6.10.5 NF / NF service instance selection for Indirect Communication without Delegated Discovery 112

6.10.5.1 General 112

6.10.5.2 Notifications corresponding to default notification subscriptions 113

6.10.6 Feature negotiation for Indirect Communication with Delegated Discovery 114

6.10.7 Notification and callback requests sent with Indirect Communication 114

6.10.8 Error Handling 115

6.10.8.1 General 115

6.10.8.2 Requirements for the originator of an HTTP error response 115

6.10.8.3 Requirements for an SCP or SEPP relaying an HTTP error response 116

6.10.9 HTTP redirection 117

6.10.9.1 General 117

6.10.10 Detection and handling of loop path when relaying message with indirect communication 118

6.10.10.1 General 118

6.10.10.2 Message Forwarding Depth Control 118

6.10.10.3 Loop Detection with Via header 118

6.10.11 Authorization of NF service access 119

6.10.11.1 General 119

6.10.11.2 Authorization for indirect communication with delegated discovery 119

6.10.11.2.1 General 119

6.10.11.2.2 Error handling when the SCP fails to obtain an access token 120

6.10.11.2.2A Error handling when the SCP obtains an access token without all the scopes required for access authorization of the incoming service request 121

6.10.11.2.3 Error handling when SCP receives a "401 Unauthorized" response or a "403 Forbidden" response with a "WWW-Authenticate" header 121

6.10.11.3 Authorization for indirect communication without delegated discovery 121

6.11 Detection and handling of late arriving requests 122

6.11.1 General 122

6.11.2 Detection and handling of requests which have timed out at the HTTP client 122

6.11.2.1 General 122

6.11.2.2 Principles 122

6.12 Binding between an NF Service Consumer and an NF Service Resource 122

6.12.1 General 122

6.12.2 Binding created as part of a service response 125

6.12.3 Binding created as part of a service request 125

6.12.4 Binding for explicit or implicit subscription requests 126

6.12.5 Binding for service requests creating a callback resource 128

6.13 SBI messages correlation for network troubleshooting 128

6.13.1 General 128

6.13.2 SBI messages correlation using UE identifier 128

6.13.2.1 General 128

6.13.2.2 Principles 129

6.13.3 SBI messages correlation using NF Peer Information 129

6.13.3.1 General 129

6.13.3.2 Principles 129

6.14 Indicating the purpose of Inter-PLMN signaling 130

6.14.1 General 130

6.14.2 Inclusion of the intended purpose 130

6.14.3 Evaluating the intended purpose 130

Annex A (informative): Client-side Adaptive Throttling for Overload Control 131

Annex B (normative): 3gpp-Sbi-Callback Types 132

Annex C (informative): Internal NF Routing of HTTP Requests 133

Annex D (Normative): ABNF grammar for 3GPP SBI HTTP custom headers 134

D.1 General 134

D.2 ABNF definitions (Filename: "TS29500_CustomHeaders.abnf") 135

Annex E (informative): Change history 145
