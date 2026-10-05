---
spec: TS 24.547
version: 19.0.0
release: '19'
clause: Annex B
title: 'Annex B (normative): CoAP entities'
source_archive: 24547-j00.zip
source_document: 24547-j00.docx
content_origin: 3gpp-source
---

# Annex B (normative): CoAP entities


## B.1 Scope

This annex describes the functionality expected from the CoAP entities (i.e. the CoAP client, the CoAP proxy and the CoAP server) defined by RFC 7252 \[17\] and 3GPP TS 23.434 \[2\].

## B.2 General

When the VAL UE is authenticating directly to the SEAL/VAL server without proxies, then the DTLS profile of IETF RFC 9202 \[20\] may be used. In order to authorize clients and protect communication across proxies, the OSCORE profile of IETF RFC 9203 \[21\] shall be used.

The client shall support UDP transport defined in IETF RFC 7252 \[17\] and should support TCP transport defined in IETF RFC 8323 \[18\]:

a\) when UDP transport and OSCORE profile of IETF RFC 9203 \[21\] are used, datagram transport layer security (DTLS) may be used;

b\) when TCP transport and OSCORE profile of IETF RFC 9203 \[21\] are used, transport layer security (TLS) may be used;

c\) when UDP transport and DTLS profile of IETF RFC 9202 \[20\] are used, datagram transport layer security (DTLS) shall be used; and

d\) when TCP transport and DTLS profile of IETF RFC 9202 \[20\] are used, transport layer security (TLS) shall be used.

Proof-of-Possession token type is used with IETF RFC 9200 \[19\].

## B.3 Procedures


### B.3.1 CoAP client

The CoAP client in the UE shall support the client role defined in IETF RFC 7252 \[17\].

If the communication is via proxies, the CoAP client in the UE:

a\) shall be configured with a home CoAP proxy FQDN parameter;

b\) shall be configured with a home CoAP proxy port parameter; and

c\) may be configured with one of the following (D)TLS tunnel authentication method along with its parameters as specified in 3GPP TS 33.434 \[7\]:

1\) one-way authentication of the CoAP proxy based on the server certificate;

2\) mutual authentication based on certificates, along with (D)TLS tunnel authentication based on X.509 certificate; and

3\) mutual authentication based on pre-shared key, along with (D)TLS tunnel authentication based on pre-shared key.

### B.3.2 CoAP proxy


#### B.3.2.1 General

The CoAP proxy shall support CoAP-to-CoAP, CoAP-to-HTTP proxy and HTTP-to-CoAP roles defined in IETF RFC 7252 \[17\].

CoAP proxy shall support UDP transport in IETF RFC 7252 \[17\] and shall support TCP transport defined in IETF RFC 8323 \[18\].

#### B.3.2.2 CoAP request method from CoAP client in UE

The CoAP proxy shall support the server role defined in IETF RFC 7252 \[17\].

The CoAP proxy may support datagram transport layer security (DTLS) or transport layer security (TLS) as specified in clause 6 of 3GPP TS 33.434 \[7\].

The CoAP proxy is configured with the following CoAP proxy parameters:

a\) an FQDN of an CoAP proxy for UEs; and

b\) a port of an CoAP proxy for UEs.

The CoAP proxy may support establishing transport connections on the FQDN of CoAP proxy for UEs and the port of CoAP proxy for UEs. The CoAP proxy shall support establishing a (D)TLS tunnel via each such transport connection as specified in 3GPP TS 33.434 \[7\]. When establishing the (D)TLS tunnel, the CoAP proxy shall act as the (D)TLS server.

#### B.3.2.3 CoAP request method from CoAP client in network entity within trust domain

The CoAP proxy is configured with the following parameters:

a\) a FQDN of an CoAP proxy for trusted entities; and

b\) a port of an CoAP proxy for trusted entities.

Upon receiving an CoAP request method via a transport connection established on the FQDN of CoAP proxy for UEs and the port of CoAP proxy for UEs, if the transport connection is between network elements within trusted domain as specified in 3GPP TS 33.434 \[7\], then:

a\) if the CoAP request contains a CoAP URI identifying a resource in a partner's VAL service provider, the CoAP proxy shall forward the CoAP request according to the CoAP URI; and

b\) if an CoAP request contains CoAP URI identifying a resource in own VAL service provider, the CoAP proxy shall act as reverse proxy for the CoAP request and shall forward the CoAP request according to VAL service provider's policy.

### B.4.2 CoAP server

The CoAP server shall support the server role defined in IETF RFC 7252 \[17\].

Upon reception of an ACE-OAuth Token Provisioning Request message containing an access token, the CoAP server:

a\) shall verify the integrity of the access token; and

b\) shall verify that the key included in the access token belongs to the authenticated requesting party.

Upon reception of a resource request, the CoAP server:

a\) shall verify that the requesting party is authorized according to the access token as specified in the corresponding ACE-OAuth profile; the DTLS profile of IETF RFC 9202 \[20\] or the OSCORE profile of ACE- IETF RFC 9203 \[21\].
