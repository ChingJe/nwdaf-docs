---
spec: TS 29.502
version: 18.11.0
release: '18'
source_archive: 29502-ib0.zip
source_document: 29502-ib0.docx
source_archive_sha256: c9132a7cf5493d033470dbbfe714121e0707138e99674c8aaf12bdab4841b264
source_document_sha256: 261580abdd73406068efdbaa9682cbdc3dbd5d31d88244d50c06d6a69c12945c
clause: Annex D
title: 'Annex D (Informative): Charging Identifier Handling'
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex D (Informative): Charging Identifier Handling

## D.1 Usage of Charging ID and SMF Charging ID

### D.1.1 General

The Charging ID has been defined (from Rel-15 onwards) as a Uint32 value which is unique within one SMF (the V-SMF or H-SMF that assigns it). To avoid possible charging identifiers' collision in H-SMF for Home Routed PDU sessions, an SMF Charging ID has been defined (from Rel-18 onwards) as a string which contains the Uint32 value and the SMF NF Instance ID (see 3GPP TS 29.571 \[13\]).

This clause summarizes the principles on using Charging ID and SMF Charging ID for HR PDU sessions, especially when not all the V-SMF(s) and H-SMF of a PDU session support the SMF Charging ID.

This Annex is informative and the normative requirements in this specification or in other 3GPP specifications prevail over the description in this Annex if there is any difference.

### D.1.2 HPLMN supporting the SMF Charging ID

An SMF, CHF and PCF complying with Rel-18 onwards support the SMF Charging ID.

If the HPLMN (H-SMF, CHF and PCF) supports the SMF Charging ID, the H-SMF indicates support of the SCID (String based Charging Identifier) feature (in its NF Profile in NRF and towards the V-SMF).

The HPLMN uses the SMF Charging ID as the charging identifier for the PDU session.

If the VPLMN supports the SMF Charging ID:

\- The VPLMN uses the SMF Charging ID as the charging identifier for the PDU session.

If the VPLMN does not support the SMF Charging ID (e.g. the V-SMF complies with a release earlier than Rel-18):

\- The VPLMN uses the Charging ID as the charging identifier for the PDU session.

\- The VPLMN only provides the Charging ID to the H-SMF during the HR PDU session establishment. The H-SMF derives the SMF Charging ID using the received Charging ID and V-SMF Instance ID.

### D.1.3 HPLMN not supporting the SMF Charging ID

When the HPLMN does not support the SMF Charging ID (e.g. the H-SMF complies with a release earlier than Rel-18), the charging identifier used by both the V-SMF and the H-SMF is the Charging ID.

NOTE: This applies even if the V-SMF supports the SCID feature.

### D.1.4 Transfer of (SMF) Charging ID between SMFs

The following principles applies to support mobility scenarios where the source and/or target (V-) SMF may or may not support the SCID feature:

\- During a HR PDU session establishment:

\- the V-SMF provides the Charging ID to the H-SMF; and

\- the V-SMF additionally provides the SMF Charging ID if both the V-SMF and H-SMF support the SCID feature.

\- During a V-SMF change or insertion:

\- the Charging ID is passed from the source (V-)SMF to the target V-SMF;

> \- the SMF Charging ID is also passed from the source (V-)SMF to the target V-SMF, if it is available at the source (V-)SMF and both the source and target (V-)SMFs support the SCID feature; and
>
> \- the H-SMF provides the SMF Charging ID to the new V-SMF if both the H-SMF and the new V-SMF support the SCID feature.

NOTE: This enables the SMF Charging ID to be used by the new V-SMF and H-SMF, when both the new V-SMF and H-SMF support the SCID feature but the source V-SMF does not support the SCID feature.

\- During EPS to 5GS mobility:

> \- the H-SMF provides the Charging ID to the new V-SMF; and
>
> \- the H-SMF additionally provides the SMF Charging ID to the new V-SMF if both the H-SMF and the V-SMF support the SCID feature.
