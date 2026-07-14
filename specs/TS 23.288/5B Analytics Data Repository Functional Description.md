---
spec: TS 23.288
version: 18.13.0
release: '18'
clause: 5B
title: 5B Analytics Data Repository Functional Description
source_archive: 23288-id0.zip
source_document: 23288-id0.docx
source_archive_sha256: 7116a1f4123dcf949818ab56feb6b242eda8819877346ab045654e1d0d8c36aa
source_document_sha256: 5e4ca6e4e350f62a5d4e4443d36ae93c9df679e1e9aea20ccaa44a4b0eb3e196
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 5B Analytics Data Repository Functional Description


## 5B.1 General


The ADRF offers services that enable a consumer to store, retrieve and delete data and analytics. The analytics are produced by the NWDAF as described in clause 6.1 and data are collected as described in clause 6.2. Multiple instances of ADRF may be deployed in a network, NF consumer selects a target ADRF instance using ADRF discovery mechanism defined in clause 6.3.20 of TS 23.501 \[2\].

Data may be stored in the ADRF by:

\- a consumer sending the ADRF an Nadrf_DataManagement_StorageRequest containing the data or analytics to be stored. The ADRF response provides a result indication.

\- a consumer sending the ADRF an Nadrf_DataManagement_StorageSubscriptionRequest requesting that the ADRF subscribes to receive data or analytics for storage. The ADRF then subscribes to the NWDAF or DCCF for data or analytics, providing ADRF Notification Address (+Notification Correlation ID). Analytics or Data are subsequently provided as notifications using DCCF, NWDAF or MFAF services (Ndccf_DataManagement Nnwdaf_DataManagement or Nmfaf_3caDataManagement service).

Data may be retrieved from the ADRF by:

\- a consumer sending an Nadrf_DataManagement_RetrievalRequest request to the ADRF to retrieve data or analytics for a Storage Transaction Identifier or a Fetch Instructions received from the ADRF in an Nadrf_DataManagement_RetrievalNotify. The ADRF determines the availability of the data or analytics in its repository and sends in the response to the consumer either the data or analytics.

\- a consumer sending an Nadrf_DataManagement_RetrievalSubscribe request to the ADRF to retrieve data or analytics for a specified data or analytics collection time window. If the time window includes the future and the ADRF has subscribed to receive the data or analytics, subsequent notifications received by the ADRF are sent by the ADRF to the notification endpoint.

The ADRF determines the availability of the data or analytics and sends a success/failure indication in the response to the consumer. The ADRF then sends one or more notifications using an Nadrf_DataManagement_RetrievalNotify to the Notification Address (+Notification Correlation ID) specified by the consumer. The notification(s) either provide the data or analytics or provide instructions to the endpoint to fetch the data or analytics using an Nadrf_DataManagement_RetrievalRequest.

Data may be deleted from the ADRF by:

\- a consumer sending an Nadrf_DataManagement_Delete request. The ADRF response provides a result indication.

An ADRF may be configured to register the data it is collecting with a DCCF. The registration uses the Ndccf_ContextManagement service specified in clause 8.3.2. The registration may subsequently be used by the DCCF to obtain data from the ADRF as described in clause 6.2.6.3.6.

The ADRF offers services that enable a consumer to store, delete and retrieve ML Models. The ML Models are trained by NWDAF containing MTLF as described in clause 5.1.

ML Model(s) may be retrieved from the ADRF by:

\- a consumer sending the ADRF an Nadrf_MLModelManagement_RetrievalRequest as described in clause 10.3.4, containing one or more tuples of unique ML Model identifier stored in ADRF or Storage Transaction Identifier. The ADRF response provides one or more tuples of unique ML Model identifiers and ML Model file address of ML Model file stored in ADRF.

ML Model(s) may be stored or updated in the ADRF by:

\- a consumer sending the ADRF an Nadrf_MLModelManagement_StorageRequest as described in clause 10.3.2, containing the ML Model or ML Model address to be stored or updated. The ADRF response provides a result indication.

NOTE: When MTLF updates a model in ADRF, or even in MTLF itself, it could happen that multiple different ML models which are being used in the system have the same model identifier, which may lead to misoperations. It is up to MTLF implementation on how to avoid such situation if that can cause problems. For example, MTLF can only update models that are not provisioned yet.

ML Model(s) may be deleted from the ADRF by:

\- a consumer sending an Nadrf_MLModelManagement_Delete request as described in clause 10.3.3. The ADRF response provides a result indication.

When a consumer requests data or analytics to be stored in an ADRF, it may specify Storage Handling information requesting:

\- a lifetime indicating how long the data or analytics should be stored; and/or

\- that a notification alerting the consumer be sent prior to data deletion from the ADRF.

The ADRF, DCCF or NWDAF may be configured with default operator storage policies. The policies specify how long data or analytics are to be stored and conditions when the default policy supersedes Storage Handling Information provided in a request from the Data or Analytics Consumer. Based on the default operator policy and the Storage Handling information, the ADRF, DCCF or NWDAF determines the Storage Approach that will be applied for the data or analytics.

The Storage Approach is comprised of the lifetime for how long the data or analytics will be stored and the notification requirement to be applied when data or analytics are to be deleted from the ADRF.

When more than one consumer requests a storage lifetime for the same data or analytics, the Storage Approach should be based on the longest requested storage lifetime.

The response to the consumer request for data or analytics includes the Storage Approach.

NOTE: The default operator policy for how long data or analytics are to be stored may be longer or shorter than the lifetime requested by the consumer. A default operator policy may for example accept only consumer requested lifetimes that are shorter or longer than the default policy.

The ADRF, DCCF or NWDAF determines when the data or analytics lifetime has expired. When data or analytics is to be removed from the ADRF, an alert is sent to the Consumer that the data is about to be deleted. The alert contains a Storage Transaction Identifier that can be used by the consumer to retrieve the data or analytics.
