---
spec: TS 23.434
version: 20.1.0
release: '20'
clause: Annex B
title: 'Annex B (informative): SEAL functional model mapping with Common functional architecture (CFA)'
source_archive: 23434-k10.zip
source_document: 23434-k10.docx
content_origin: 3gpp-source
---

# Annex B (informative): SEAL functional model mapping with Common functional architecture (CFA)

The table B-1 shows the mapping between the SEAL functional model and the Common functional architecture (CFA). The details of CFA functional entities and reference points are specified in 3GPP TS 23.280 \[4\].

Table B-1: SEAL functional model mapping with CFA

| SEAL service                                                                      | Aspects           | SEAL                               | CFA                             |
|-----------------------------------------------------------------------------------|-------------------|------------------------------------|---------------------------------|
| Location management                                                               | Functional entity | Location management client         | Location management client      |
|                                                                                   |                   | Location management server         | Location management server      |
|                                                                                   | Reference points  | LM-UU                              | CSC-14                          |
|                                                                                   |                   | LM-S                               | CSC-15                          |
|                                                                                   |                   | LM-C                               | Not defined                     |
|                                                                                   |                   | LM-E                               | Not defined                     |
|                                                                                   |                   | LM-PC5                             | Not defined                     |
| Group management                                                                  | Functional entity | Group management client            | Group management client         |
|                                                                                   |                   | Group management server            | Group management server         |
|                                                                                   | Reference points  | GM-UU                              | CSC-2                           |
|                                                                                   |                   | GM-S                               | CSC-3                           |
|                                                                                   |                   | GM-C                               | Not defined                     |
|                                                                                   |                   | GM-E                               | CSC-16                          |
|                                                                                   |                   | GM-PC5                             | CSC-12                          |
| Configuration management                                                          | Functional entity | Configuration management client    | Configuration management client |
|                                                                                   |                   | Configuration management server    | Configuration management server |
|                                                                                   | Reference points  | CM-UU                              | CSC-4                           |
|                                                                                   |                   | CM-S                               | CSC-5                           |
|                                                                                   |                   | CM-C                               | Not defined                     |
|                                                                                   |                   | CM-E                               | CSC-17                          |
|                                                                                   |                   | CM-PC5                             | CSC-11                          |
| Identity management                                                               | Functional entity | Identity management client         | Identity management client      |
|                                                                                   |                   | Identity management server         | Identity management server      |
|                                                                                   | Reference points  | IM-UU                              | CSC-1                           |
|                                                                                   |                   | IM-S                               | Not defined                     |
|                                                                                   |                   | IM-C                               | Not defined                     |
|                                                                                   |                   | IM-E                               | Not defined                     |
|                                                                                   |                   | IM-PC5                             | Not defined                     |
| Key management                                                                    | Functional entity | Key management client              | Key management client           |
|                                                                                   |                   | Key management server              | Key management server           |
|                                                                                   | Reference points  | KM-UU                              | CSC-8                           |
|                                                                                   |                   | KM-S                               | CSC-9                           |
|                                                                                   |                   | KM-PC5                             | Not defined                     |
| Network resource management                                                       | Functional entity | Network resource management client | Not defined (see NOTE)          |
|                                                                                   |                   | Network resource management server | Not defined (see NOTE)          |
|                                                                                   | Reference points  | NRM-UU                             | Not defined (see NOTE)          |
|                                                                                   |                   | NRM-S                              | Not defined                     |
|                                                                                   |                   | NRM-C                              | Not defined                     |
|                                                                                   |                   | NRM-E                              | Not defined                     |
|                                                                                   |                   | NRM-PC5                            | Not defined                     |
| NOTE: Defined in the application layer for Mission Critical service (e.g. MCPTT). |                   |                                    |                                 |
