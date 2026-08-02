---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: Annex A
title: 'Annex A (informative): Colour Codes'
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex A (informative): Colour Codes

## A.1 Utilization of the BSIC

A BSIC is allocated to each cell. A BSIC can take one of 64 values. In each cell the BSIC is broadcast in each burst sent on the SCH, and is then known by all MSs which synchronise with this cell. The BSIC is used by the MS for several purposes, all aiming at avoiding ambiguity or interference which can arise when an MS in a given position can receive signals from two cells *using the same BCCH frequency*.

Some of the uses of the BSIC relate to cases where the MS is attached to one of the cells. Other uses relate to cases where the MS is attached to a third cell, usually somewhere between the two cells in question.

The first category of uses includes:

\- The three least significant bits of the BSIC indicate which of the 8 training sequences is used in the bursts sent on the downlink common channels of the cell. Different training sequences allow for a better transmission if there is interference. The group of the three least significant bits of the BSIC is called the BCC (Base station Colour Code).

\- The BSIC is used to modify the bursts sent by the MSs on the access bursts. This aims to avoid one cell correctly decoding access bursts sent to another cell.

The second category of uses includes:

\- When in connected mode, the MSs measure and report the level they receive on a number of frequencies, corresponding to the BCCH frequencies of neighbouring cells in the same network as the used cell. Along with the measurement result, the MS sends to the network the BSIC which it has received on that frequency. This enables the network to discriminate between several cells which happen to use the same BCCH frequency. Poor discrimination might result in faulty handovers.

\- The content of the measurement report messages is limited to information for 6 neighbour cells. It is therefore useful to limit the reported cells to those to which handovers are accepted. For this purpose, each cell provides a list of the values of the three most significant bits of the BSICs which are allocated to the cells which are useful to consider for handovers (usually excluding cells in other PLMNs). This information enables the MS to discard information for cells with non‑conformant BSICs and not to report them. The group of the three most significant bits of the BSIC is called the NCC (Network Colour Code).

It should be noted that when in idle mode, the MS identifies a cell (for cell selection purposes) according to the cell identity broadcast on the BCCH and *not* by the BSIC.

## A.2 Guidance for planning

From these uses, the following planning rule can be derived:

*If there exist places where MSs can receive signals from two cells, whether in the same PLMN or in different PLMNs, which use the same BCCH frequency, it is highly preferable that these two cells have different BSICs.*

Where the coverage areas of two PLMNs overlap, the rule above is respected if:

1\) The PLMNs use different sets of BCCH frequencies (In particular, this is the case if no frequency is common to the two PLMNs. This usually holds for PLMNs in the same country), or

2\) The PLMNS use different sets of NCCs, or

3\) BSIC and BCCH frequency planning is co-ordinated.

Recognizing that method 3) is more cumbersome than method 2), and that method 1) is too constraining, it is suggested that overlapping PLMNs which use a common part of the spectrum agree on different NCCs to be used in any overlapping areas. As an example, a preliminary NCC allocation for countries in the European region can be found in clause A.3 of this annex.

This example can be used as a basis for bilateral agreements. However, the use of the NCCs allocated in clause A.3 is not compulsory. PLMN operators can agree on different BSIC allocation rules in border areas. The use of BSICs is not constrained in non-overlapping areas, or if ambiguities are resolved by using different sets of BCCH frequencies.

If the PLMNs share one or more cells with other PLMNs, the planning rule above should be applied also when the BCCH frequency is different. The rule should be respected by using different sets of NCCs. In addition to that, the PLMN sharing one or more cells with other PLMNs should use different NCCs for shared and non-shared neighbouring cells.

## A.3 Example of PLMN Colour Codes (NCCs) for the European region

Austria : 0

Belgium : 1

Cyprus : 3

Denmark : 1

Finland : 0

France : 0

Germany : 3

Greece : 0

Iceland : 0

Ireland : 3

Italy : 2

Liechtenstein : 2

Luxembourg : 2

Malta : 1

Monaco : 3 (possibly 0(=France))

Netherlands : 0

Norway : 3

Portugal : 3

San Marino : 0 (possibly 2(= Italy))

Spain : 1

Sweden : 2

Switzerland : 1

Turkey : 2

UK : 2

Vatican : 1 (possibly 2(=Italy)

Yugoslavia : 3

This allows a second operator for each country by allocating the colour codes n (in the table) and n + 4. More than 2 colour codes per country may be used provided that in border areas only the values n and/or n+4 are used.
