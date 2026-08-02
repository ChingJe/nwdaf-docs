---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: Annex B
title: 'Annex B (normative): IMEI Check Digit computation'
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex B (normative): IMEI Check Digit computation

## B.1 Representation of IMEI

The International Mobile station Equipment Identity and Software Version number (IMEISV), as defined in clause 6, is a 16 digit decimal number composed of three distinct elements:

\- an 8 digit Type Allocation Code (TAC);

\- a 6 digit Serial Number (SNR); and

\- a 2 digit Software Version Number (SVN).

The IMEISV is formed by concatenating these three elements as illustrated below:

|     |     |     |
|-----|-----|-----|
| TAC | SNR | SVN |

Figure A.1: Composition of the IMEISV

The IMEI is complemented by a check digit as defined in clause 3. The Luhn Check Digit (CD) is computed on the 14 most significant digits of the IMEISV, that is on the value obtained by ignoring the SVN digits.

The method for computing the Luhn check is defined in Annex B of the International Standard "Identification cards - Numbering system and registration procedure for issuer identifiers" (ISO/IEC 7812 \[3\]).

In order to specify precisely how the CD is computed for the IMEI, it is necessary to label the individual digits of the IMEISV, excluding the SVN. This is done as follows:

The (14 most significant) digits of the IMEISV are labelled D14, D13 ... D1, where:

\- TAC = D14, D13 ... D7 (with D7 the least significant digit of TAC);

\- SNR = D6, D5 ... D1 (with D1 the least significant digit of SNR).

## B.2 Computation of CD for an IMEI

Computation of CD from the IMEI proceeds as follows:

Step 1: Double the values of the odd labelled digits D1, D3, D5 ... D13 of the IMEI.

Step 2: Add together the individual digits of all the seven numbers obtained in Step 1, and then add this sum to the sum of all the even labelled digits D2, D4, D6 ... D14 of the IMEI.

Step 3: If the number obtained in Step 2 ends in 0, then set CD to be 0. If the number obtained in Step 2 does not end in 0, then set CD to be that number subtracted from the next higher number which does end in 0.

## B.3 Example of computation

IMEI (14 most significant digits):

|                              |                   |
|------------------------------|-------------------|
| TAC                          | SNR               |
| D14 D13 D12 D11 D10 D9 D8 D7 | D6 D5 D4 D3 D2 D1 |
| 2 6 0 5 3 1 7 9              | 3 1 1 3 8 3       |

Step 1:

|                 |             |
|-----------------|-------------|
| 2 6 0 5 3 1 7 9 | 3 1 1 3 8 3 |
| x2 x2 x2 x2     | x2 x2 x2    |
| 12 10 2 18      | 2 6 6       |

Step 2:

2 + 1 + 2 + 0 + 1 + 0 + 3 + 2 + 7 + 1 + 8 + 3 + 2 + 1 + 6 + 8 + 6 = 53

Step 3:

CD = 60 - 53 = 7
