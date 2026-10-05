---
spec: TS 23.558
version: 20.3.0
release: '20'
clause: 9
title: 9 Usage of SEAL services
source_archive: 23558-k30.zip
source_document: 23558-k30.docx
content_origin: 3gpp-source
---

# 9 Usage of SEAL services


## 9.1 Notification management service


### 9.1.1 General

The notification management is a SEAL service that offers the notification functionality. This service enables EEC to subscribe and receive notifications from the EES and ECS, and thereby offloading the complexity of delivery and reception of notifications to the edge enabler layer.

### 9.1.2 Information flows

The following information flows of notification management service of SEAL as specified in 3GPP TS 23.434 \[13\] are applicable for the EEL:

\- Create notification channel request specified in clause 17.3.2.1;

\- Create notification channel response specified in clause 17.3.2.2;

\- Open notification channel specified in clause 17.3.2.3;

\- Notification message specified in clause 17.3.2.4.

The usage of the above information flows are clarified as below:

\- The Callback URL is the address (e.g. Notification Target Address) where the notifications destined for the EEC;

\- VAL Application ID is EECID;

\- VAL Service ID is ECS ID or EES ID;

\- VAL client is the EEC;

\- VAL server is the EES or ECS.

### 9.1.3 Procedures

The following procedures of notification management service of SEAL as specified in 3GPP TS 23.434 \[13\] are applicable for the edge enabler layer:

\- Procedure for creating notification channel to receive notifications, specified in clause 17.3.3.
