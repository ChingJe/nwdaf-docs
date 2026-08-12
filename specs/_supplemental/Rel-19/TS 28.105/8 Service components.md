---
spec: TS 28.105
version: 19.6.0
release: '19'
supplemental: true
applicability: cross-release-reference-not-rel18-requirement
clause: '8'
title: 8 Service components
source_archive: 28105-j60.zip
source_document: 28105-j60.docx
source_archive_sha256: 089e1ce07dccc54d8a24698a0bba1a4ce98bbeaf15e0e61591e76cd979363c65
source_document_sha256: b19d56076f9a7e55838d64c44d15840ebdec46c773546fa3d89d232640897564
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
build_id: r18-core-20260812-rel19-ts28105-supplemental
---

# 8 Service components

## 8.0 General

The operations for generic provisioning management service refer to clause 11.1.1 of TS 28.532 \[11\]. For notifications, see clause 7.6.

## 8.1 Lifecycle management operations for AI/ML management MnS

The components for ML model training MnS are listed in table 8.1-1.

Table 8.1-1: Components for ML model training

| ML model training management capability      | Management service component type A | Management service component type B | Management service component type C |
|----------------------------------------------|-------------------------------------|-------------------------------------|-------------------------------------|
| Create an ML model training request          | createMOI operation                 | MLTrainingRequest                   | N/A                                 |
| Modify an ML model training request          | modifyMOIAttributes operation       |                                     |                                     |
| Query an ML model training report            | getMOIAttributes operation          | MLTrainingReport                    |                                     |
| Modify an ML model training process          | modifyMOIAttributes operation       | MLTrainingProcess                   |                                     |
| Query an ML model training process           | getMOIAttributes operation          |                                     |                                     |
| Create, Delete, and Update ML model training | changeMOIs operation                | MLTrainingRequest                   |                                     |

The components for ML model testing are listed in table 8.1-2.

Table 8.1-2: Components for ML model testing

| ML model testing management capability        | Management service component type A | Management service component type B | Management service component type C |
|-----------------------------------------------|-------------------------------------|-------------------------------------|-------------------------------------|
| Create an ML model testing request            | createMOI operation                 | MLTestingRequest                    | N/A                                 |
| Modify an ML model testing request            | modifyMOIAttributes operation       |                                     |                                     |
| Query an ML model testing report              | getMOIAttributes operation          | MLTestingReport                     |                                     |
| Subscribe an ML model testing report          | createMOI operation                 |                                     |                                     |
| Unsubscribe an ML model testing report        | deleteMOI operation                 |                                     |                                     |
| Query an ML model testing report subscription | getMOIAttributes operation          |                                     |                                     |
| Create, Delete, and Update ML model testing   | changeMOIs operation                | MLTestingRequest                    |                                     |

The components for ML model deployment are listed in table 8.1-3

Table 8.1-3: Components for ML model deployment

<table>
<colgroup>
<col style="width: 30%" />
<col style="width: 27%" />
<col style="width: 29%" />
<col style="width: 12%" />
</colgroup>
<thead>
<tr class="header">
<th>ML model deployment management capability</th>
<th>Management service component type A</th>
<th>Management service component type B</th>
<th>Management service component type C</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Create an ML model loading request</td>
<td>createMOI operation</td>
<td rowspan="2">MLModelLoadingRequest</td>
<td rowspan="9">N/A</td>
</tr>
<tr class="even">
<td>Modify an ML model loading request</td>
<td>modifyMOIAttributes operation</td>
</tr>
<tr class="odd">
<td>Create an ML model loading policy</td>
<td>createMOI operation</td>
<td rowspan="4">MLModelLoadingPolicy</td>
</tr>
<tr class="even">
<td>Delete an ML model loading policy</td>
<td>deleteMOI operation</td>
</tr>
<tr class="odd">
<td>Modify an ML model loading policy</td>
<td>modifyMOIAttributes operation</td>
</tr>
<tr class="even">
<td>Query an ML model loading policy</td>
<td>getMOIAttributes operation</td>
</tr>
<tr class="odd">
<td>Modify an ML model loading process</td>
<td>modifyMOIAttributes operation</td>
<td rowspan="2">MLModelLoadingProcess</td>
</tr>
<tr class="even">
<td>Query an ML model loading process</td>
<td>getMOIAttributes operation</td>
</tr>
<tr class="odd">
<td>Create, Delete, and Update ML model Loading</td>
<td>changeMOIs operation</td>
<td><p>MLModelLoadingRequest</p>
<p>MLModelLoadingPolicy</p></td>
</tr>
</tbody>
</table>

The components for ML model inference are listed in table 8.1-4

Table 8.1-4: Components for AI/ML inference

| AI/ML Inference management capability                            | Management service component type A | Management service component type B | Management service component type C |
|------------------------------------------------------------------|-------------------------------------|-------------------------------------|-------------------------------------|
| Create an ML model update request                                | createMOI operation                 | MLUpdateRequest                     | N/A                                 |
| Modify an ML model update request                                | modifyMOIAttributes operation       |                                     |                                     |
| Modify an ML model update process                                | modifyMOIAttributes operation       | MLUpdateProcess                     |                                     |
| Query an ML model update process                                 | getMOIAttributes operation          |                                     |                                     |
| Query an ML model update report                                  | getMOIAttributes operation          | MLUpdateReport                      |                                     |
| Query an AI/ML inference report                                  | getMOIAttributes operation          | AIMLInferenceReport                 |                                     |
| Create, Delete, and Update ML model Update and ML update request | changeMOIs operation                | MLUpdateRequest                     |                                     |

The components for AI/ML inference emulation are listed in table 8.1-5.

Table 8.1-5: Components for AI/ML inference emulation MnS

| ML model emulation management capability  | Management service component type A | Management service component type B | Management service component type C |
|-------------------------------------------|-------------------------------------|-------------------------------------|-------------------------------------|
| Query an AI/ML inference emulation report | getMOIAttributes operation          | AIMLInferenceReport                 | N/A                                 |
