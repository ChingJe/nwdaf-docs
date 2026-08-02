---
spec: TS 23.003
version: 18.8.0
release: '18'
source_archive: 23003-i80.zip
source_document: 23003-i80.docx
source_archive_sha256: 41b3214bc0a5303937bdd33f0fb0a77a33885610b99b5b3b3d662eaafddd59d4
source_document_sha256: cea584d8dfd003d4d25119f1353107486ccbaacfe27cedcdeb3674b2ec06bd30
clause: Annex D
title: 'Annex D (informative): Applicability and use of the ".3gppnetwork.org" domain name'
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex D (informative): Applicability and use of the ".3gppnetwork.org" domain name

There currently exists a private IP network between operators to provide connectivity for user transparent services that utilise protocols that rely on IP. This includes (but is not necessarily limited to) such services as GPRS/PS roaming, WLAN roaming, GPRS/PS inter‑PLMN handover and inter‑MMSC MM delivery. This inter‑PLMN IP backbone network consists of indirect connections using brokers (known as GRXs – GPRS Roaming Exchanges) and direct inter‑PLMN connections (e.g. private wire); it is however *not* connected to the Internet. More details can be found in GSMA PRD IR.34 \[57\].

Within this inter‑PLMN IP backbone network, the domain name ".gprs" was originally conceived as the only domain name to be used to enable DNS servers to translate logical names for network nodes to IP addresses (and vice versa). However, after feedback from the Internet Engineering Task Force (IETF) it was identified that use of this domain name has the following drawbacks:

1\. Leakage of DNS requests for the ".gprs" top level domain into the public Internet is inevitable at sometime or other, especially as the number of services (and therefore number of nodes) using the inter‑PLMN IP backbone increases. In the worst case scenario of faulty clients, the performance of the Internet's root DNS servers would be seriously degraded by having to process requests for a top level domain that does not exist.

2\. It would be very difficult for network operators to detect if/when DNS requests for the ".gprs" domain were leaked to the public Internet (and therefore the security policies of the inter‑PLMN IP backbone network were breached), because the Internet's root DNS servers would simply return an error message to the sender of the request only.

To address the above, the IETF recommended using a domain name that is *routable* in the pubic domain but which requests to it are not actually *serviced* in the public domain. The domain name ".3gppnetwork.org" was chosen as the new top level domain name to be used (as far as possible) within the inter‑PLMN IP backbone network.

Originally, only the DNS servers connected to the inter‑PLMN IP backbone network were populated with the correct information needed to service requests for *all* sub‑domains of this domain. However, it was later identified that some new services needed their allocated sub‑domain(s) to be resolvable by the UE and not just inter-PLMN IP network nodes. To address this, additional, higher‑level sub‑domains were created:

\- "pub.3gppnetwork.org", which is to be used for domain names that need to be resolvable by UEs (and possibly network nodes too) that are connected to a local area network that is connected to the Internet; and

\- "ipxuni.3gppnetwork.org", which is to be used for domain names for UNI interfaces that need to be resolvable by UEs that are connected to a local area network that is not connected to the Internet (e.g. local area networks connected to the inter-PLMN IP network of the IPX).

Therefore, DNS requests for the above domain names can be resolved, while requests for all other sub‑domains of "3gppnetwork.org" can simply be configured to return the usual DNS error for unknown hosts (thereby avoiding potential extra, redundant load on the Internet's root DNS servers).

The GSM Association is in charge of allocating new sub‑domains of the ".3gppnetwork.org" domain name. The procedure for requesting new sub‑domains can be found in Annex E.
