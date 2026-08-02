---
spec: TS 29.502
version: 18.11.0
release: '18'
source_archive: 29502-ib0.zip
source_document: 29502-ib0.docx
source_archive_sha256: c9132a7cf5493d033470dbbfe714121e0707138e99674c8aaf12bdab4841b264
source_document_sha256: 261580abdd73406068efdbaa9682cbdc3dbd5d31d88244d50c06d6a69c12945c
clause: Annex C
title: 'Annex C (Normative): ABNF grammar for 3GPP SBI HTTP custom headers'
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex C (Normative): ABNF grammar for 3GPP SBI HTTP custom headers

## C.1 General

This Annex contains a self-contained set of ABNF rules, comprising the re-used rules from IETF RFCs, and the rules defined by the 3GPP custom headers defined in this specification (see clause 6.1.2.3).

This grammar may be used as input to existing tools to help implementations to parse 3GPP custom headers.

## C.2 ABNF definitions (Filename: "TS29502_CustomHeaders.abnf")

; ----------------------------------------

; RFC 5234

; ----------------------------------------

HTAB = %x09 ; horizontal tab

SP = %x20

DIGIT = %x30-39 ; 0-9

; ----------------------------------------

; RFC 9110

; ----------------------------------------

OWS = \*( SP / HTAB )

date1 = day SP month SP year ; e.g., 02 Jun 1982

day = 2DIGIT

month = %x4A.61.6E ; "Jan", case-sensitive

/ %x46.65.62 ; "Feb", case-sensitive

/ %x4D.61.72 ; "Mar", case-sensitive

/ %x41.70.72 ; "Apr", case-sensitive

/ %x4D.61.79 ; "May", case-sensitive

/ %x4A.75.6E ; "Jun", case-sensitive

/ %x4A.75.6C ; "Jul", case-sensitive

/ %x41.75.67 ; "Aug", case-sensitive

/ %x53.65.70 ; "Sep", case-sensitive

/ %x4F.63.74 ; "Oct", case-sensitive

/ %x4E.6F.76 ; "Nov", case-sensitive

/ %x44.65.63 ; "Dec", case-sensitive

year = 4DIGIT

day-name = %x4D.6F.6E ; Mon

/ %x54.75.65 ; Tue

/ %x57.65.64 ; Wed

/ %x54.68.75 ; Thu

/ %x46.72.69 ; Fri

/ %x53.61.74 ; Sat

/ %x53.75.6E ; Sun

time-of-day = hour ":" minute ":" second

hour = 2DIGIT

minute = 2DIGIT

second = 2DIGIT

; ----------------------------------------

; 3GPP TS 29.502

;

; Version: 18.5.0 (December 2023)

;

; (c) 2023, 3GPP Organizational Partners (ARIB, ATIS, CCSA, ETSI, TSDSI, TTA, TTC).

; ----------------------------------------

;

; Header: 3gpp-Sbi-Origination-Timestamp

;

Sbi-Origination-Timestamp-Header = "3gpp-Sbi-Origination-Timestamp:" OWS day-name ","

SP date1 SP time-of-day "." milliseconds SP "GMT" OWS

milliseconds = 3DIGIT
