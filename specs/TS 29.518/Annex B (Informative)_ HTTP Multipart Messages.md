---
spec: TS 29.518
version: 18.14.0
release: '18'
clause: Annex B
title: 'Annex B (Informative): HTTP Multipart Messages'
source_archive: 29518-ie0.zip
source_document: 29518-ie0.docx
source_archive_sha256: aa2789dac49d4f52e56fcc9d04277a5122a40fdf4c22faf877a7832f844daae1
source_document_sha256: 263e0bf7e06ccfa30ab69e661e2a65e0fedf654b22d2675975add5f47f8acad0
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex B (Informative): HTTP Multipart Messages


## B.1 Example of HTTP multipart message


### B.1.1 General

This clause provides a (partial) example of HTTP multipart message. The example does not aim to be a complete representation of the HTTP message, e.g. additional information or headers can be included.

This Annex is informative and the normative descriptions in this specification prevail over the description in this Annex if there is any difference.


### B.1.2 Example HTTP multipart message with N2 Information binary data

POST /example.com/namf-comm/v1/ue-contexts/{ueContextId}/n1-n2-messages HTTP/2

Content-Type: multipart/related; boundary=----Boundary

Content-Length: xyz

------Boundary

Content-Type: application/json

{

"n2InfoContainer": {

"n2InformationClass": "SM",

"smInfo": {

"pduSessionId": 5,

"n2InfoContent": {

"ngapIeType": "PDU_RES_SETUP_REQ",

"ngapData": {

"contentId": "n2msg"

}

}

}

},

"pduSessionId": 5

}

------Boundary

Content-Type: application/vnd.3gpp.ngap

Content-Id: n2msg

{ … N2 Information binary data …}

------Boundary
