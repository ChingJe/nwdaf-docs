---
spec: TS 28.105
version: 19.6.0
release: '19'
supplemental: true
applicability: cross-release-reference-not-rel18-requirement
clause: '4'
title: 4 Concepts and overview
source_archive: 28105-j60.zip
source_document: 28105-j60.docx
source_archive_sha256: 089e1ce07dccc54d8a24698a0bba1a4ce98bbeaf15e0e61591e76cd979363c65
source_document_sha256: b19d56076f9a7e55838d64c44d15840ebdec46c773546fa3d89d232640897564
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
build_id: r18-core-20260812-rel19-ts28105-supplemental
---

# 4 Concepts and overview

## 4.1 Overview

The AI/ML techniques and relevant applications are being increasingly adopted by the wider industries and proved to be successful. These are now being applied to telecommunication industry including mobile networks.

Although AI/ML techniques in general are quite mature nowadays, some of the relevant aspects of the technology are still evolving while new complementary techniques are frequently emerging.

The AI/ML techniques can be generally characterized from different perspectives including the followings:

\- **Learning methods**

The learning methods include supervised learning, semi-supervised learning, unsupervised learning and reinforcement learning. Each learning method fits one or more specific category of inference (e.g. prediction), and requires specific type of training data. A brief comparison of these learning methods is provided in table 4.1-1.

Table 4.1-1: Comparison of Learning methods

<table>
<colgroup>
<col style="width: 22%" />
<col style="width: 19%" />
<col style="width: 18%" />
<col style="width: 19%" />
<col style="width: 19%" />
</colgroup>
<thead>
<tr class="header">
<th></th>
<th>Supervised learning</th>
<th>Semi-supervised learning</th>
<th>Unsupervised learning</th>
<th>Reinforcement learning</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>Category of inference</strong></td>
<td>Regression (numeric), classification</td>
<td>Regression (numeric), classification</td>
<td>Association,<br />
Clustering</td>
<td>Reward-based behaviour</td>
</tr>
<tr class="even">
<td><strong>Type of training data</strong></td>
<td>Labelled data (Note)</td>
<td>Labelled data (Note), and unlabelled data</td>
<td>Unlabelled data</td>
<td>Not pre-defined</td>
</tr>
<tr class="odd">
<td colspan="5">NOTE: The labelled data refers to a set of training and testing data that have been assigned with one or more labels in order to add context and meaning.</td>
</tr>
</tbody>
</table>

**- Learning complexity:**

\- As per the learning complexity, there are Machine Learning (i.e. basic learning) and Deep Learning.

**- Learning architecture**

\- Based on the topology and location where the learning tasks take place, the AI/ML can be categorized to centralized learning, distributed learning and federated learning.

**- Learning continuity**

\- From learning continuity perspective, the AI/ML can be offline learning or continual learning.

Artificial Intelligence/Machine Learning (AI/ML) capabilities are used in various domains in 5GS, including management and orchestration (e.g. MDA, see 3GPP TS 28.104 \[2\]) and 5G networks (e.g. NWDAF, see 3GPP TS 23.288 \[3\]).

The AI/ML inference function in the 5GS uses the ML model for inference.

Each AI/ML technique, depending on the adopted specific characteristics as mentioned above, may be suitable for supporting certain type/category of use case(s) in 5GS.

To enable and facilitate the AI/ML capabilities with the suitable AI/ML techniques in 5GS, the ML model and AI/ML inference function need to be managed.

The present document specifies the generic AI/ML management related capabilities and services without specifically taking any of the above-mentioned learning methods into consideration. The AI/ML management capabilities which include the followings:

\- ML model training.

\- ML model testing.

\- AI/ML inference emulation.

\- ML model deployment.

\- AI/ML inference.

## 4.2 Management of AI/ML capabilities for RAN

The management of AI/ML capabilities for the RAN covers scenarios where both the ML model training and AI/ML inference are located in the NG-RAN node, as well as scenarios where the ML model training is located in the management system and the AI/ML inference is located in the NG-RAN node. In either case, the NG-RAN AI/ML-based feature defined in clause 16.20 of TS 38.300 \[16\] can be supported.

## 4.3 Management of AI/ML capabilities for 5GC

The management of AI/ML capabilities for the 5GC covers scenarios where both the ML model training and AI/ML inference functions are located in the 5GC.in this case, the NWDAF feature defined in clause 6 of TS 23.288 \[3\] can be supported.

## 4.4 Management of AI/ML capabilities for MDA

For MDA, the ML training function can be located either inside or outside the MDAF while the AI/ML inference function is located in the MDAF. In this case, the MDA capabilities defined in clause 7.2 of TS 28.104 \[2\] can be supported.
