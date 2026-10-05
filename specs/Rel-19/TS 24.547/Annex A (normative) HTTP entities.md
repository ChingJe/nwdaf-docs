---
spec: TS 24.547
version: 19.0.0
release: '19'
clause: Annex A
title: 'Annex A (normative): HTTP entities'
source_archive: 24547-j00.zip
source_document: 24547-j00.docx
content_origin: 3gpp-source
---

# Annex A (normative): HTTP entities


## A.1 Scope

This annex describes the functionality expected from the HTTP entities (i.e. the HTTP client, the HTTP proxy and the HTTP server) defined by 3GPP TS 23.434 \[2\].

## A.2 Procedures


### A.2.1 HTTP client


#### A.2.1.1 General

The HTTP client shall support the client role defined in IETF RFC 9112 \[15\].

#### A.2.1.2 HTTP client in UE

The HTTP client in the U E shall support the client role defined in IETF RFC 9110 \[16\].

The HTTP client in the UE shall support transport layer security (TLS) as specified in clause 6 of 3GPP TS 33.434 \[7\].

The HTTP client in the UE is configured with the following parameters:

a\) a home HTTP proxy FQDN;

b\) a home HTTP proxy port;

c\) One of the following TLS tunnel authentication method along with its parameters as specified in 3GPP TS 33.434 \[7\]:

1\) one-way authentication of the HTTP proxy based on the server certificate;

2\) mutual authentication based on certificates, along with TLS tunnel authentication based on X.509 certificate; and

3\) mutual authentication based on pre-shared key, along with TLS tunnel authentication based on pre-shared key;

The HTTP client in the UE shall establish a TCP connection towards the home HTTP proxy FQDN and the home HTTP proxy port.

The HTTP client in the UE shall establish a TLS tunnel via the TCP connection as specified in 3GPP TS 33.434 \[7\]. When establishing the TLS tunnel, the HTTP client in the UE shall act as a TLS client and the UE shall perform the TLS tunnel authentication using the TLS authentication method indicated by the TLS tunnel authentication method parameter according to 3GPP TS 33.434 \[7\]. In order to prevent man-in-the-middle attacks, the HTTP client in the UE shall check the home HTTP proxy FQDN against the server's identity as presented in the received server's certificate message if the TCP connection terminates on the HTTP proxy. The HTTP client in the UE shall check the portion of dereferenced HTTP URL against the server's identity as presented in the received server's certificate message only if the TCP connection terminates on the SIM-S.

NOTE: The TLS tunnel can be terminated in the HTTP proxy (rather than in the HTTP server providing the dereferenced HTTP URL).

The HTTP client in the UE shall send and receive all HTTP messages via the TLS tunnel.

If the HTTP client in the UE has an access token of the "bearer" token type as specified in IETF RFC 6750 \[13\], the HTTP client in the UE shall include an Authorization header field with the "Bearer" authentication scheme as specified in IETF RFC 6750 \[13\] in HTTP requests.

#### A.2.1.3 HTTP client in network entity

The HTTP client in the network entity is configured with the following parameters:

a\) a home HTTP proxy FQDN; and

b\) a home HTTP proxy port.

The HTTP client in the network entity shall send and receive all HTTP messages via the home HTTP proxy.

The HTTP client in the network entity shall insert an X-3GPP-Asserted-Identity header field as specified in 3GPP TS 24.109 \[14\] in the HTTP request and shall set X-3GPP-Asserted-Identity header field to the identity of the HTTP client in the network entity. The identity of the HTTP client in the network entity can be a public service identity, a VAL group ID, or a VAL service ID.

### A.2.2 HTTP proxy


#### A.2.2.1 General

The HTTP proxy shall support proxy role defined in .

#### A.2.2.2 HTTP request method from HTTP client in UE

The HTTP proxy shall support the server role defined in IETF RFC 9112 \[15\] \[5\], and in IETF RFC 9110 \[16\].

The HTTP proxy shall support transport layer security (TLS) as specified in 3GPP TS 33.434 \[7\].

The HTTP proxy is configured with the following HTTP proxy parameters:

a\) an FQDN of an HTTP proxy for UEs; and

b\) a TCP port of an HTTP proxy for UEs.

The HTTP proxy shall support establishing TCP connections on the FQDN of HTTP proxy for UEs and the TCP port of HTTP proxy for UEs. The HTTP proxy shall support establishing a TLS tunnel via each such TCP connection as specified in 3GPP TS 33.434 \[7\]. When establishing the TLS tunnel, the HTTP proxy shall act as the TLS server.

Upon reception of an HTTP request method via a TLS tunnel:

a\) if the HTTP request method contains an X-3GPP-Asserted-Identity header field as specified in 3GPP TS 24.109 \[14\], the HTTP proxy shall reject the HTTP request method with an HTTP 403 (Forbidden) response and shall not continue with the below steps;

b\) if the HTTP request method contains a Request-URI identifying a resource in a partner's VAL service provider, the HTTP proxy shall forward the HTTP request method according to the Request-URI; and

c\) if the HTTP request method contains a Request-URI identifying a resource in its own VAL service provider, the HTTP proxy shall act as a reverse proxy for the HTTP request method and shall forward the HTTP request method according to the VAL service provider's policy.

#### A.2.2.3 HTTP request method from HTTP client in network entity within trust domain

The HTTP proxy is configured with the following parameters:

a\) a FQDN of an HTTP proxy for trusted entities; and

b\) a TCP port of an HTTP proxy for trusted entities.

Upon receiving an HTTP request method via a TCP connection established on the FQDN of HTTP proxy for UEs and the TCP port of HTTP proxy for UEs, if the TCP connection is between network elements within trusted domain as specified in 3GPP TS 33.434 \[7\], then:

a\) if the HTTP request method contains a Request-URI identifying a resource in a partner's VAL service provider, the HTTP proxy shall forward the HTTP request method according to the Request-URI; and

b\) if an HTTP request method contains Request-URI identifying a resource in own VAL service provider, the HTTP proxy shall act as reverse proxy for the HTTP request method and shall forward the HTTP request method according to VAL service provider's policy.

### A.2.3 HTTP server

The HTTP server shall support the server role defined in IETF RFC 9112 \[15\].

Upon reception of an HTTP request:

a\) if the received HTTP request does not contain an Authorization header field with the "Bearer" authentication scheme and a bearer access token as specified in IETF RFC 6750 \[13\] and the received HTTP request does not contain an X-3GPP-Asserted-Identity header field as specified in 3GPP TS 24.109 \[14\], the HTTP server shall reject the request with HTTP 403 (Forbidden) response;

b\) if the received HTTP request contains an Authorization header field with the "Bearer" authentication scheme and a bearer access token as specified in IETF RFC 6750 \[13\];

a\) the HTTP server shall validate the bearer access token as specified in IETF RFC 6750 \[13\]; and

b\) the HTTP server shall consider the VAL service ID derived from the bearer access token as the identity of the sender of the HTTP request; and

c\) if the received HTTP request does not contain an Authorization header field with the "Bearer" authentication scheme and a bearer access token as specified in IETF RFC 6750 \[13\] and the received HTTP request contains an X-3GPP-Asserted-Identity header field as specified in 3GPP TS 24.109 \[14\], the HTTP server shall consider the URI in the X-3GPP-Asserted-Identity header field as the identity of the sender of the HTTP request.
