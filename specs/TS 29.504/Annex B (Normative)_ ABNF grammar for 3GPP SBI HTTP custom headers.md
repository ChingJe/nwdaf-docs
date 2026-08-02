---
spec: TS 29.504
version: 18.13.0
release: '18'
source_archive: 29504-id0.zip
source_document: 29504-id0.docx
source_archive_sha256: 08eb3fe98bf504d4516fefd5b25d7d85d32137d58050b96e05c110afd341706b
source_document_sha256: 819017105e6641a94e926626c4b8c28c960eff13fd1c084dc9c76b2ef745ba6a
clause: Annex B
title: 'Annex B (Normative): ABNF grammar for 3GPP SBI HTTP custom headers'
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex B (Normative): ABNF grammar for 3GPP SBI HTTP custom headers

## B.1 General

This Annex contains a self-contained set of ABNF rules, comprising the re-used rules from IETF RFCs, and the rules defined by the 3GPP custom headers defined in this specification (see clause 6.1.2.3).

This grammar may be used as input to existing tools to help implementations to parse 3GPP custom headers.

## B.2 ABNF definitions (Filename: "TS29504_CustomHeaders.abnf")

; ----------------------------------------

; RFC 5234

; ----------------------------------------

HTAB = %x09 ; horizontal tab

LF = %x0A ; linefeed

CR = %x0D ; carriage return

SP = %x20

DQUOTE = %x22 ; " (Double Quote)

DIGIT = %x30-39 ; 0-9

ALPHA = %x41-5A / %x61-7A ; A-Z / a-z

VCHAR = %x21-7E ; visible (printing) characters

WSP = SP / HTAB ; white space

CRLF = CR LF ; Internet standard newline

; ----------------------------------------

; RFC 5322

; ----------------------------------------

obs-FWS = 1\*WSP \*( CRLF 1\*WSP )

FWS = ( \[ \*WSP CRLF \] 1\*WSP ) / obs-FWS

obs-NO-WS-CTL = %d1-8 / %d11 / %d12 / %d14-31 / %d127

obs-ctext = obs-NO-WS-CTL

ctext = %d33-39 / %d42-91 / %d93-126 / obs-ctext

obs-qp = "\\ ( obs-NO-WS-CTL / LF / CR )

quoted-pair = ( "\\ ( VCHAR / WSP ) ) / obs-qp

ccontent = ctext / quoted-pair / comment

comment = "(" \*( \[ FWS \] ccontent ) \[ FWS \] ")"

; ----------------------------------------

; RFC 9110

; ----------------------------------------

OWS = \*( SP / HTAB )

tchar = "!" / "#" / "\$" / "%" / "&" / "'" / "\*" / "+" / "-"

/ "." / "^" / "\_" / "\`" / "\|" / "~" / DIGIT / ALPHA

token = 1\*tchar

obs-text = %x80-FF

entity-tag = \[ weak \] opaque-tag

weak = %x57.2F ; "W/", case-sensitive

opaque-tag = DQUOTE \*etagc DQUOTE

etagc = %x21 / %x23-7E / obs-text ; VCHAR except double quotes, plus obs-text

; ----------------------------------------

; 3GPP TS 29.504

;

; Version: 18.5.0 (March 2024)

;

; (c) 2024, 3GPP Organizational Partners (ARIB, ATIS, CCSA, ETSI, TSDSI, TTA, TTC).

; ----------------------------------------

;

; Header: 3gpp-Sbi-Notification-Correlation

;

Sbi-Notification-Correlation-Header = "3gpp-Sbi-Notification-Correlation:" OWS subscriptionId

\*( OWS "," OWS subscriptionId ) OWS

subscriptionId = token

;

; Header: 3gpp-Sbi-Etags

;

Sbi-Etags-Header = "3gpp-Sbi-Etags:" OWS datasetEtag \*( OWS "," OWS datasetEtag ) OWS

datasetEtag = dataSetName "=" entity-tag

dataSetName = UeSubscribedDataSetName

UeSubscribedDataSetName = 1\*( ALPHA / DIGIT / "\_" )
