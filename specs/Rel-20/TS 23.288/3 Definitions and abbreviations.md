---
spec: TS 23.288
version: 20.2.0
release: '20'
clause: 3
title: 3 Definitions and abbreviations
source_archive: 23288-k20.zip
source_document: 23288-k20.docx
content_origin: 3gpp-source
---

# 3 Definitions and abbreviations


## 3.1 Definitions

For the purposes of the present document, the terms and definitions given in TR 21.905 \[1\], TS 23.501 \[2\] and TS 23.503 \[4\]. A term defined in the present document takes precedence over the definition of the same term, if any, in TR 21.905 \[1\].

**Analytics Accuracy Information:** Represent a performance measure of an Analytics ID provided by an NWDAF containing AnLF, which includes accuracy value or loss value and its compute method of the Analytics ID and optionally the corresponding number of samples. The accuracy value is computed as the number of correct predictions divided by the total number of predictions. The loss value is computed by taking the differences between actual values and predicted values of continuous analytics output. Refer to clause 5C.1 for more information.

**Analytics Feedback Information:** Indicates that the consumer NF has taken action(s) influenced by the previously provided analytics, which may or may not affect the ground truth data.

**Label:** A label is the training objective in supervised machine learning.

**ML Model Accuracy Information:** Represent a performance measure of a ML Model provided by an NWDAF containing MTLF, which includes accuracy value or loss value and its compute method of the ML Model and optionally the corresponding number of samples. The accuracy value is computed as the number of correct predictions divided by the total number of predictions. The loss value is computed by taking the differences between actual values and predicted values of continuous analytics output calculated by the model. Refer to clause 5C.1 for more information.

**Vertical Federated Learning (VFL):** A federated learning technique without exchanging/sharing local data set, wherein the local data set in different VFL Participant for local model training have different feature spaces for the same samples (e.g. UE IDs).

## 3.2 Abbreviations

For the purposes of the present document, the abbreviations given in TR 21.905 \[1\], TS 23.501 \[2\] and TS 23.503 \[4\] and the following apply. An abbreviation defined in the present document takes precedence over the definition of the same abbreviation, if any, in TR 21.905 \[1\].

AI/ML Artificial Intelligence/Machine Learning

DCAF Data Collection Application Function

FL Federated Learning

HFL Horizontal Federated Learning

RE-NWDAF Roaming Exchange Network Data Analytics Function

VFL Vertical Federated Learning
