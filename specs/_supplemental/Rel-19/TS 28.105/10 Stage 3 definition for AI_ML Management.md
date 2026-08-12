---
spec: TS 28.105
version: 19.6.0
release: '19'
supplemental: true
applicability: cross-release-reference-not-rel18-requirement
clause: '10'
title: 10 Stage 3 definition for AI/ML Management
source_archive: 28105-j60.zip
source_document: 28105-j60.docx
source_archive_sha256: 089e1ce07dccc54d8a24698a0bba1a4ce98bbeaf15e0e61591e76cd979363c65
source_document_sha256: b19d56076f9a7e55838d64c44d15840ebdec46c773546fa3d89d232640897564
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
build_id: r18-core-20260812-rel19-ts28105-supplemental
---

# 10 Stage 3 definition for AI/ML Management

## 10.1 RESTful HTTP-based solution set

The RESTful HTTP-based solution set for generic provisioning management service is defined in clause 12.1.1 in 3GPP TS 28.532 \[11\]. Corresponding className is ML model training, ML model testing, AI/ML inference emulation, ML model deployment, and AI/ML inference.

### 10.1.1 ML model training 

Table 10.1.1-1describes the SS to support ML model training request management based on Table 12.1.1.1.1-1 in TS 28.532 \[11\].

Table 10.1.1-1: SS to support ML model training request management

<table>
<colgroup>
<col style="width: 14%" />
<col style="width: 17%" />
<col style="width: 7%" />
<col style="width: 60%" />
</colgroup>
<thead>
<tr class="header">
<th>ML model training request management</th>
<th>IS operation</th>
<th>HTTP Method</th>
<th>Resource URI</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Create an ML model training request</td>
<td>createMOI operation</td>
<td>PUT</td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLTrainingRequest}={id}</td>
</tr>
<tr class="even">
<td>Modify an ML model training request</td>
<td>modifyMOIAttributes operation</td>
<td><p>PUT</p>
<p>PATCH</p></td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLTrainingRequest}={id}</td>
</tr>
<tr class="odd">
<td>Create, Delete, and Update ML training request</td>
<td>changeMOIs operation</td>
<td>PATCH</td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLTrainingRequest}={id}</td>
</tr>
</tbody>
</table>

Table 10.1.1-2 describes the SS to support ML model training report management based on Table 12.1.1.1.1-1 in TS 28.532 \[11\].

**Table 10.1.1-2: SS to support ML model training report management**

| ML model training report management | IS operation               | HTTP Method | Resource URI                                                                |
|-------------------------------------|----------------------------|-------------|-----------------------------------------------------------------------------|
| Query an ML model training report   | getMOIAttributes operation | GET         | {MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLTrainingReport}={id} |

Table 10.1.1-3 describes the SS to support ML model training process based on Table 12.1.1.1.1-1 in TS 28.532 \[11\].

**Table 10.1.1-3: SS to support ML model training process management**

<table>
<colgroup>
<col style="width: 11%" />
<col style="width: 27%" />
<col style="width: 7%" />
<col style="width: 53%" />
</colgroup>
<thead>
<tr class="header">
<th>ML model training process management</th>
<th>IS operation</th>
<th>HTTP Method</th>
<th>Resource URI</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Modify an ML model training process</td>
<td>modifyMOIAttributes operation</td>
<td><p>PUT</p>
<p>PATCH</p></td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLTrainingProcess}={id}</td>
</tr>
<tr class="even">
<td>Query an ML model training process</td>
<td>getMOIAttributes operation</td>
<td>GET</td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLTrainingProcess}={id}</td>
</tr>
</tbody>
</table>

### 10.1.2 ML model testing 

Table 10.1.2-1 describes the SS to support ML model testing management based on Table 12.1.1.1.1-1in TS 28.532 \[11\].

Table 10.1.2-1: SS to support ML model testing management

<table>
<colgroup>
<col style="width: 14%" />
<col style="width: 17%" />
<col style="width: 7%" />
<col style="width: 60%" />
</colgroup>
<thead>
<tr class="header">
<th>ML model testing request management</th>
<th>IS operation</th>
<th>HTTP Method</th>
<th>Resource URI</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Create an ML model testing request</td>
<td>createMOI operation</td>
<td>PUT</td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLTestingRequest}={id}</td>
</tr>
<tr class="even">
<td>Modify an ML model testing request</td>
<td>modifyMOIAttributes operation</td>
<td><p>PUT</p>
<p>PATCH</p></td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLTestingRequest}={id}</td>
</tr>
<tr class="odd">
<td>Create, Delete, and Update ML testing request</td>
<td>changeMOIs operation</td>
<td>PATCH</td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLTestingRequest}={id}</td>
</tr>
</tbody>
</table>

Table 10.1.2-2 describes the SS to support ML model testing report management based on Table 12.1.1.1.1-1 in TS 28.532 \[11\].

Table 10.1.2-2: SS to support ML model testing report management

| ML model testing report management            | IS operation               | HTTP Method | Resource URI                                                               |
|-----------------------------------------------|----------------------------|-------------|----------------------------------------------------------------------------|
| Query an ML model testing report              | getMOIAttributes operation | GET         | {MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLTestingreport}={id} |
| Subscribe an ML model testing report          | createMOI operation        | PUT         | {MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLTestingreport}={id} |
| Unsubscribe an ML model testing report        | deleteMOI operation        | DELETE      | {MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLTestingreport}={id} |
| Query an ML model testing report subscription | getMOIAttributes operation | GET         | {MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLTestingreport}={id} |

### 10.1.3 AI/ML inference emulation 

Table 10.1.3-1 describes the SS to support AI/ML inference emulation report management based on Table 12.1.1.1.1-1 in TS 28.532 \[11\].

Table 10.1.3-1: SS to support AI/ML inference emulation report management

| AI/ML inference emulation report management | IS operation               | HTTP Method | Resource URI                                                                   |
|---------------------------------------------|----------------------------|-------------|--------------------------------------------------------------------------------|
| Query an AI/ML inference emulation report   | getMOIAttributes operation | GET         | {MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{AIMLInferenceReport}={id} |

### 10.1.4 ML model deployment 

Table 10.1.4-1 describes the SS to support ML model loading request management based on Table 12.1.1.1.1-1 in TS 28.532 \[11\].

Table 10.1.4-1: SS to support ML model loading request management

<table>
<colgroup>
<col style="width: 14%" />
<col style="width: 17%" />
<col style="width: 7%" />
<col style="width: 60%" />
</colgroup>
<thead>
<tr class="header">
<th>ML model loading management</th>
<th>IS operation</th>
<th>HTTP Method</th>
<th>Resource URI</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Create an ML model loading request</td>
<td>createMOI operation</td>
<td>PUT</td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLModelLoadingRequest}={id}</td>
</tr>
<tr class="even">
<td>Modify an ML model loading request</td>
<td>modifyMOIAttributes operation</td>
<td><p>PUT</p>
<p>PATCH</p></td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLModelLoadingRequest}={id}</td>
</tr>
<tr class="odd">
<td>Create, Delete, and Update ML model Loading request</td>
<td>changeMOIs operation</td>
<td>PATCH</td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLModelLoadingRequest}={id}</td>
</tr>
</tbody>
</table>

Table 10.1.4-2 describes the SS to support ML model loading policy based on Table 12.1.1.1.1-1 in TS 28.532 \[11\].

Table 10.1.4-2: SS to support ML model loading policy management

<table>
<colgroup>
<col style="width: 11%" />
<col style="width: 27%" />
<col style="width: 7%" />
<col style="width: 53%" />
</colgroup>
<thead>
<tr class="header">
<th>ML model loading policy management</th>
<th>IS operation</th>
<th>HTTP Method</th>
<th>Resource URI</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Create an ML model loading policy</td>
<td>createMOI operation</td>
<td><p>PUT</p>
<p>PATCH</p></td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLModelLoadingPolicy}={id}</td>
</tr>
<tr class="even">
<td>Delete an ML model loading policy</td>
<td>deleteMOI operation</td>
<td>DELETE</td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLModelLoadingPolicy}={id}</td>
</tr>
<tr class="odd">
<td>Modify an ML model loading policy</td>
<td>modifyMOIAttributes operation</td>
<td><p>PUT</p>
<p>PATCH</p></td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLModelLoadingPolicy}={id}</td>
</tr>
<tr class="even">
<td>Query an ML model loading policy</td>
<td>getMOIAttributes operation</td>
<td>GET</td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLModelLoadingPolicy}={id}</td>
</tr>
<tr class="odd">
<td>Create, Delete, and Update ML model Loading policy</td>
<td>changeMOIs operation</td>
<td>PATCH</td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLModelLoadingPolicy}={id}</td>
</tr>
</tbody>
</table>

Table 10.1.4-3 describes the SS to support ML model loading process management based on Table 12.1.1.1.1-1 in TS 28.532 \[11\].

Table 10.1.4-3: SS to support ML model loading process management

<table>
<colgroup>
<col style="width: 11%" />
<col style="width: 27%" />
<col style="width: 7%" />
<col style="width: 53%" />
</colgroup>
<thead>
<tr class="header">
<th>ML model loading process management</th>
<th>IS operation</th>
<th>HTTP Method</th>
<th>Resource URI</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Modify an ML model loading process</td>
<td>modifyMOIAttributes operation</td>
<td><p>PUT</p>
<p>PATCH</p></td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLModelLoadingProcess}={id}</td>
</tr>
<tr class="even">
<td>Query an ML model loading process</td>
<td>getMOIAttributes operation</td>
<td>GET</td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLModelLoadingProcess}={id}</td>
</tr>
</tbody>
</table>

### 10.1.5 AI/ML inference 

Table 10.1.5-1 describes the SS to support ML model update request management based on Table 12.1.1.1.1-1 in TS 28.532 \[11\].

Table 10.1.5-1: SS to support ML model update request management

<table>
<colgroup>
<col style="width: 14%" />
<col style="width: 17%" />
<col style="width: 7%" />
<col style="width: 60%" />
</colgroup>
<thead>
<tr class="header">
<th>ML model update request management</th>
<th>IS operation</th>
<th>HTTP Method</th>
<th>Resource URI</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Create an ML model update request</td>
<td>createMOI operation</td>
<td>PUT</td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLUpdateRequest}={id}</td>
</tr>
<tr class="even">
<td>Modify an ML model update request</td>
<td>modifyMOIAttributes operation</td>
<td><p>PUT</p>
<p>PATCH</p></td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLUpdateRequest}={id}</td>
</tr>
<tr class="odd">
<td>Create, Delete, and Update ML model Update request</td>
<td>changeMOIs operation</td>
<td>PATCH</td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLUpdateRequest}={id}</td>
</tr>
</tbody>
</table>

Table 10.1.5-2 describes the SS to support ML model update report management based on Table 12.1.1.1.1-1 in TS 28.532 \[11\].

Table 10.1.5-2: SS to support ML model update report management

| ML model update report management | IS operation               | HTTP Method | Resource URI                                                              |
|-----------------------------------|----------------------------|-------------|---------------------------------------------------------------------------|
| Query an ML model update report   | getMOIAttributes operation | GET         | {MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLUpdateReport}={id} |

Table 10.1.5-3 describes the SS to support ML model update process management based on Table 12.1.1.1.1-1 in TS 28.532 \[11\].

Table 10.1.5-3: SS to support ML model update process management

<table>
<colgroup>
<col style="width: 11%" />
<col style="width: 27%" />
<col style="width: 7%" />
<col style="width: 53%" />
</colgroup>
<thead>
<tr class="header">
<th>ML model update process management</th>
<th>IS operation</th>
<th>HTTP Method</th>
<th>Resource URI</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Modify an ML model update process</td>
<td>modifyMOIAttributes operation</td>
<td><p>PUT</p>
<p>PATCH</p></td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLUpdateProcess}={id}</td>
</tr>
<tr class="even">
<td>Query an ML model update process</td>
<td>getMOIAttributes operation</td>
<td>PATCH</td>
<td>{MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{MLUpdateProcess}={id}</td>
</tr>
</tbody>
</table>

Table 10.1.5-4 describes the SS to support AI/ML infernece report management based on Table 12.1.1.1.1-1 in TS 28.532 \[11\].

Table 10.1.5-4: SS to support AI/ML infernece report management

| AI/ML inference report management | IS operation               | HTTP Method | Resource URI                                                                   |
|-----------------------------------|----------------------------|-------------|--------------------------------------------------------------------------------|
| Query an AI/ML inference report   | getMOIAttributes operation | GET         | {MnSRoot}/ProvMnS/{MnSVersion}/{URI-LDN-first-part}/{AIMLInferenceReport}={id} |
