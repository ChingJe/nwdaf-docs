---
spec: TS 29.503
version: 18.13.0
release: '18'
clause: Annex C
title: 'Annex C (informative): SUCI encoding'
source_archive: 29503-id0.zip
source_document: 29503-id0.docx
source_archive_sha256: 4c773b0d93af7d82fab12329d4c7d06807cc92271da897e23ee94e89a7112afa
source_document_sha256: 6c856fd8095c9d5afe17dbc8d23248638223b9f6c80bf2129176c4a516b50fef
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex C (informative): SUCI encoding

The structure of the Subscription Concealed Identifier (SUCI) is defined in TS 23.003 \[8\].

When SUCI needs to be sent as a character string (e.g. as a string in a JSON content of any of the service operations defined in the APIs defined in this specification), the SUCI is composed as an UTF-8 character string, where the different components are separated by the "minus" character "-" (UTF-8 0x2D).

These components shall be formatted as follows:

1\) SUPI Type: a single decimal digit, from 0 to 7, formatted as a single UTF-8 character (UTF-8 0x30 to 0x37)

2\) Home Network Identifier.

> When the SUPI Type is an IMSI, the Home Network Identifier consists on 2 components: MCC and MNC, separated by the "minus" character; these components are formatted as a string of 3 characters for MCC and a string of 2 or 3 characters for MNC (UTF-8 0x30 to 0x39).
>
> When the SUPI type is a Network Specific Identifier, Global Line Identifier (GLI) or Global Cable Identifier (GCI) the Home Network Identifier consists of a string of characters with a variable length, formatted as an UTF-8 character string.

3\) Routing Indicator, consisting of 1 to 4 decimal digits formatted as a string of 1 to 4 characters (UTF-8 0x30 to 0x39).

4\) Protection Scheme Identifier, consisting in a value in the range of 0 to 15, representing a single hexadecimal digit, formatted as a single UTF-8 character (UTF-8 0x30 to 0x39, or 0x41 to 0x46, or 0x61 to 0x66).

5\) Home Network Public Key Identifier, consisting in a value in the range 0 to 255, formatted as a sequence of 1 to 3 decimal digits, formatted of 1 to 3 UTF-8 characters (UTF-8 0x30 to 0x39).

6\) Scheme Output, consisting of a string of UTF-8 characters with a variable length, or a sequence of hexadecimal digits, dependent on the used protection scheme. It represents the output of a public key protection scheme specified in Annex C of TS 33.501 \[6\] or the output of a protection scheme specified by the HPLMN.

Finally, the above structure is prefixed by the string "suci-", when the SUCI is sent in the "SupiOrSuci" data type definition, as described in TS 29.571 \[7\], Table 5.3.2-1.

For the SUPI types "Network Specific Identifier (NSI)", "Global Line Identifier (GLI)", and "Global Cable Identifier (GCI)", the SUCI sent by the UE to the AMF is always formatted as a NAI (see TS 23.003 \[8\], clauses 28.7.3, 28.15.5 and 28.16.5). In those cases, before sending such SUCI to the UDM or AUSF, the AMF needs to re-encode the received SUCI, and format it according to the structure described above (as an UTF-8 character string, where the different components are separated by the "minus" character "-").

EXAMPLES:

1\) SUPI is IMSI-based; MCC=123, MNC=45, MSIN: 0123456789

SUPI type: 0 (IMSI)

Routing Indicator: 012

Protection Scheme: 0 (NULL scheme)

Home Network Public Key Identifier: 0

Scheme output = MSIN (cleartext)

\- SUCI UTF-8 string, as sent on the UDM APIs:

> "suci-0-123-45-012-0-0-0123456789"

NOTE: According to TS 33.501 \[6\] (see annex C.2) the NULL scheme returns the same output as the input (i.e. MSIN in this example), which can be packed BCD coded. However, when formatted as character string in JSON, the scheme output is expected to be reformatted from packed BCD (5 octets in this example) to a sequence of decimal digits in UTF-8 (10 characters in this example).

2\) SUPI is IMSI-based, MCC=123, MNC=45, MSIN: 9876543210 (coded as 10 hexadecimal digits using 5 octets packed BCD coding: 89, 67, 45, 23, 01)

> SUPI type: 0 (IMSI)
>
> Routing Indicator: 0002
>
> Protection Scheme: 1 (Profile A)
>
> Home Network Public Key Identifier: 17

Scheme output = ECC ephemeral public key (32 octets, first bolded part below) + Encrypted MSIN (where MSIN has 10 digits, i.e. 5 octets coded as hexadecimal digits using packed BCD coding, italic part below) + MAC tag (8 octets, last bolded part below) = 50 octets = 100 hexadecimal characters

(NOTE: the keys and encrypted content below is fictitious).

> \- SUCI UTF-8 string, as sent on the UDM APIs:
>
> "suci‑0‑123‑45‑0002‑1‑17‑**e9b9916c911f448d8792e6b2f387f85d3ecab9040049427d9edbb5431b0bc711***023be6a057***b45d936238aebeb7**"

3\) SUPI is NAI-based, SUPI = alice@example.com

SUPI type = 1 (Network Specific Identifier)

Routing Indicator: 84

Protection Scheme: 2 (Profile B)

Home Network Public Key Identifier: 250

ECC ephemeral public key (33 octets, first bolded part below, in field "ecckey")

Encrypted username of NAI "alice" (5 octets, italic part below, in field "cip")

MAC tag (8 octets, last bolded part below, in field "mac")

(NOTE: the keys and encrypted content below is fictitious)

\- SUCI in NAI format as sent by the UE to the AMF:

> "type1.rid84.schid2.hnkey250.ecckey**e9b9916c911f448d8792e6b2f387f85d3ecab9040049427d9edbb5431b0bc71195**.cip*023be6a057*.mac**b45d936238aebeb7**@example.com"

\- SUCI UTF-8 string, as sent on the UDM APIs:

> "suci‑1‑example.com‑84‑2‑250‑**e9b9916c911f448d8792e6b2f387f85d3ecab9040049427d9edbb5431b0bc71195***023be6a057***b45d936238aebeb7**"

4\) SUPI is NAI-based; SUPI = 00-00-5E-00-53-00@operator.com

SUPI type: 3 (GCI)

Routing Indicator: 012

Protection Scheme: 0 (NULL scheme)

Home Network Public Key Identifier: 0

Username = 00-00-5E-00-53-00 (in field "userid" below)

\- SUCI in NAI format as sent by the UE to the AMF:

> "type3.rid012.schid0.userid00-00-5E-00-53-00@operator.com"

\- SUCI UTF-8 string, as sent on the UDM APIs:

> "suci-3-operator.com-012-0-0-00-00-5E-00-53-00"

5\) Anonymous SUCI

SUPI type: 1 (Network Specific Identifier)

Operator realm = operator.com

Routing Indicator: 3456

Protection Scheme: 0 (NULL scheme)

Home Network Public Key Identifier: 0

\- Alternative with username = anonymous

\- SUCI in NAI format as sent by the UE to the AMF:

> "type1.rid3456.schid0.useridanonymous@operator.com"

\- SUCI UTF-8 string, as sent on the UDM APIs:

> "suci-1-operator.com-3456-0-0-anonymous"

\- Alternative with omitted username

\- SUCI in NAI format as sent by the UE to the AMF:

> "type1.rid3456.schid0.userid@operator.com"

\- SUCI UTF-8 string, as sent on the UDM APIs:

> "suci-1-operator.com-3456-0-0-"
