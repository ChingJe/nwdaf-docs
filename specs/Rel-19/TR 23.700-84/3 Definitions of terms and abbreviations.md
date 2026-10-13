---
spec: TR 23.700-84
version: 19.0.0
release: '19'
clause: 3
title: 3 Definitions of terms and abbreviations
source_archive: 23700-84-j00.zip
source_document: 23700-84-j00.docx
content_origin: 3gpp-source
---

# 3 Definitions of terms and abbreviations


## 3.1 Terms

For the purposes of the present document, the terms given in TR 21.905 \[1\] and the following apply. A term defined in the present document takes precedence over the definition of the same term, if any, in TR 21.905 \[1\].

**Horizontal Federated Learning (HFL):** a federated learning technique without exchanging/sharing local data set, wherein the local data set in different FL clients for local model training have the same feature space for different samples (e.g. UE IDs).

**Vertical Federated Learning (VFL):** a federated learning technique without exchanging/sharing local data set, wherein the local data set in different VFL Participant for local model training have different feature spaces for the same samples (e.g. UE IDs).

**Label:** A label is the training objective in supervised machine learning.

**VFL Server:** An NWDAF or AF that integrates local training results, computes gradient information or loss information and send them to VFL client(s) for the local ML model update in VFL training process. It also coordinates the VFL training process by discovering and selecting VFL clients. In VFL inference process, The VFL server aggregates local inference results from VFL clients to generate the final VFL inference result and sends the final VFL inference result to the consumer. Only one VFL server may exist for each VFL process.

**VFL Client:** An NWDAF or AF that holds the local dataset and performs local training and inference as asked by VFL Server. There can be multiple VFL Clients in VFL training and inference.

## 3.2 Abbreviations

For the purposes of the present document, the abbreviations given in TR 21.905 \[1\] and the following apply. An abbreviation defined in the present document takes precedence over the definition of the same abbreviation, if any, in TR 21.905 \[1\].

AP Active Participant

HFL Horizontal Federated Learning

PP Passive Participant

VFL Vertical Federated Learning
