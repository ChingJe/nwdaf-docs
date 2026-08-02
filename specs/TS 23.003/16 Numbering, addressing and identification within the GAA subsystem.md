---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: '16'
title: 16 Numbering, addressing and identification within the GAA subsystem
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 16 Numbering, addressing and identification within the GAA subsystem

## 16.1 Introduction

This clause describes the format of the parameters needed to access the GAA system. For further information on the use of the parameters see TS 33.221 \[58\]. For more information on the ".3gppnetwork.org" domain name and its applicability, see Annex D of the present document.

## 16.2 BSF address

The Bootstrapping Server Function (BSF) address is in the form of a Fully Qualified Domain Name as defined in IETF RFC 1035 \[19\] and IETF RFC 1123 \[20\]. The BSF address consists of several labels. Each label shall consist of the alphabetic characters (A-Z and a-z), digits (0-9) and the hyphen (-) in accordance with IETF RFC 1035 \[19\]. Each label shall begin and end with either an alphabetic character or a digit in accordance with IETF RFC 1123 \[20\]. The case of alphabetic characters is not significant.

For 3GPP systems, the UE shall discover the BSF address from the identity information related to the UICC application that is used during the bootstrapping procedure i.e. IMSI for USIM, or IMPI for ISIM, in the following way:

\- In the case where the USIM is used in bootstrapping, the BSF address shall be derived as follows:

1\. take the first 5 or 6 digits, depending on whether a 2 or 3 digit MNC is used (see TS 31.102 \[27\]) and separate them into MCC and MNC; if the MNC is 2 digits then a zero shall be added at the beginning;

2\. use the MCC and MNC derived in step 1 to create the "mnc\<MNC\>.mcc\<MCC\>.pub.3gppnetwork.org" domain name;

3\. add the label "bsf." to the beginning of the domain.

Example 1: If IMSI in use is "234150999999999", where MCC=234, MNC=15, and MSIN=0999999999, the BSF address would be "bsf.mnc015.mcc234.pub.3gppnetwork.org".

\- In the case where ISIM is used in bootstrapping, the BSF address shall be derived as follows:

1\. extract the domain name from the IMPI;

2\. if the last two labels of the domain name extracted from the IMPI are "3gppnetwork.org":

a\. the first label is "bsf";

b\. the next labels are all labels of the domain name extracted from the IMPI apart from the last two labels; and

c\. the last three labels are "pub.3gppnetwork.org";

Example 2: If the IMPI in use is "234150999999999@ims.mnc015.mcc234.3gppnetwork.org", the BSF address would be "bsf.ims.mnc015.mcc234.pub.3gppnetwork.org".

3\. if the last two labels of the domain name extracted from the IMPI are other than the "3gppnetwork.org":

a\. add the label "bsf." to the beginning of the domain.

Example 3: If the IMPI in use is "user@operator.com", the BSF address would be "bsf.operator.com ".
