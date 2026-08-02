---
spec: TS 29.502
version: 18.11.0
release: '18'
source_archive: 29502-ib0.zip
source_document: 29502-ib0.docx
source_archive_sha256: c9132a7cf5493d033470dbbfe714121e0707138e99674c8aaf12bdab4841b264
source_document_sha256: 261580abdd73406068efdbaa9682cbdc3dbd5d31d88244d50c06d6a69c12945c
clause: Annex B
title: 'Annex B (Informative): HTTP Multipart Messages'
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex B (Informative): HTTP Multipart Messages

## B.1 Example of HTTP multipart message

### B.1.1 General

This clause provides a (partial) example of HTTP multipart message. The example does not aim to be a complete representation of the HTTP message, e.g. additional information or headers can be included.

This Annex is informative and the normative descriptions in this specification prevail over the description in this Annex if there is any difference.

### B.1.2 Example HTTP multipart message with N1 SM Message binary data

POST /example.com/nsmf-pdusession/v1/sm-contexts HTTP/2

Content-Type: multipart/related; type="application/json"; boundary=----Boundary

Content-Length: xyz

------Boundary

Content-Type: application/json

{

"supi": "imsi-\<IMSI\>",

"pduSessionId": 235,

"dnn": "\<DNN\>",

"sNssai": {

"sst": 0

},

"servingNfId": "\<AMF Identifier\>",

"n1SmMsg": {

"contentId": "n1msg"

},

"anType": "3GPP_ACCESS",

"smContextStatusUri": "\<URI\>"

}

------Boundary

Content-Type: application/vnd.3gpp.5gnas

Content-Id: n1msg

{ … N1 SM Message binary data …}

------Boundary
