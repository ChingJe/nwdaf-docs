---
spec: TS 24.546
version: 19.5.0
release: '19'
clause: Annex B
title: 'Annex B (normative): Parameters for notifications'
source_archive: 24546-j50.zip
source_document: 24546-j50.docx
content_origin: 3gpp-source
---

# Annex B (normative): Parameters for notifications


## B.1 General

The information in this annex provides a normative description of the parameters which will be sent by SCM-S while sending different types of notification

## B.2 Configuration update notification

The SCM-S shall convey the following parameters while sending configuration notification to SCM-C.

Table B.2-1: Parameters for configuration update notification

| Parameter | Description                                                                                                                |
|-----------|----------------------------------------------------------------------------------------------------------------------------|
| Identity  | REQUIRED. A unique string representing notification channel identity.                                                      |
| Event     | REQUIRED. Shall be set to one of the event as specified in table A.1.2-2 based on which configuration document is updated. |
