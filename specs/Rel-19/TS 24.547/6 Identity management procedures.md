---
spec: TS 24.547
version: 19.0.0
release: '19'
clause: 6
title: 6 Identity management procedures
source_archive: 24547-j00.zip
source_document: 24547-j00.docx
content_origin: 3gpp-source
---

# 6 Identity management procedures


## 6.1 General

## 6.2 On-network procedures


### 6.2.1 General

### 6.2.2 User authentication procedure


#### 6.2.2.1 SIM-C procedure


##### 6.2.2.1.1 HTTP based procedure

Upon receiving a request from VAL user to initiate authentication for VAL services, the SIM-C shall:

a\) establish a TLS tunnel to the authorisation endpoint of the SIM-S as specified in 3GPP TS 33.434 \[7\] using the URL of authorisation endpoint of the SIM-S as provided by the specific VAL service; and

b\) send an OIDC Authentication Request message as specified in the OpenID Connect 1.0 \[11\] and IETF RFC 6749 \[9\] using an HTTP GET request method towards the SIM-S according to IETF RFC 9110 \[16\]. The SIM-C shall include the following parameters as specified in 3GPP TS 33.434 \[7\] in the query component of the authorization endpoint's URI using the "application/x-www-form-urlencoded" format as specified in W3C.REC-html401-19991224 \[6\]:

\- response_type;

\- client_id;

\- scope;

\- redirect_uri;

\- state;

\- acr_values;

\- code_challenge; and

\- code_challenge_method.

Upon receiving an HTTP 200 (OK) response from the SIM-S, the SIM-C shall:

a\) prompt the VAL service user for their username and password;

b\) generate an HTTP POST request method containing the VAL service user's username and password; and

c\) send the HTTP POST request method towards the SIM-S.

Upon receiving an OIDC Authentication Response message, the SIM-C shall:

a\) establish a TLS tunnel to the token endpoint of the SIM-S as specified in 3GPP TS 33.434 \[7\]; and

b\) send an OIDC Token Request message as specified in OpenID Connect 1.0 \[11\] and IETF RFC 6749 \[9\] using an HTTP POST request method towards the SIM-S according to IETF RFC 9110 \[16\]. The SIM-C shall include the following parameters in the entity body of the HTTP POST request using the "application/x-www-form-urlencoded" format as specified in W3C.REC-html401-19991224 \[6\] as specified in 3GPP TS 33.434 \[7\]:

\- grant_type;

\- code;

\- client_id;

\- redirect_uri; and

\- code_verifier.

Upon receiving an OIDC Token response message from the SIM-S, the SIM-C shall:

a\) validate the id_token, access_token and refresh token in the received OIDC Token Response message as specified in the OpenID Connect 1.0 \[11\] specification; and

b\) provide the id_token and access_token in the received OIDC Token Response message to the VAL user.

The SIM-C may repeat the entire procedure in this clause as needed to obtain the necessary authorisation tokens for the VAL service clients, depending on the scope parameter in the Authentication Request message as specified in 3GPP TS 33.434 \[7\].

##### 6.2.2.1.2 CoAP based procedure

Upon receiving a request from VAL user to initiate authentication for VAL services, the SIM-C:

a\) may establish a (D)TLS tunnel to the token endpoint of the SIM-S as specified in 3GPP TS 33.434 \[7\] using the URL of token endpoint of the SIM-S as provided by the specific VAL service; and

b\) shall send an ACE-OAUTH Token Request message with client credentials grant type as specified in IETF RFC 9200 \[19\] using a CoAP POST request towards the SIM-S. The SIM-C shall use the "application/ace+cbor" format and:

a\) shall include grant type parameter;

b\) shall include scope parameter;

c\) may include req_cnf parameter; and

d\) may include ace_profile parameter,

in the message payload as specified in IETF RFC 9200 \[19\].

Upon receiving an CoAP 2.01 (Created) response from the SIM-S, the SIM-C shall:

a\) validate the access token as specified in IETF RFC 9200 \[19\]; and

b\) provide the access token in the received ACE-OAUTH Token Response message to the VAL user.

The SIM-C may repeat the entire procedure in this clause as needed to obtain the necessary access tokens for the VAL service clients, depending on the scope parameter in the Token Request message as specified in 3GPP TS 33.434 \[7\].

#### 6.2.2.2 SIM-S procedure


##### 6.2.2.2.1 HTTP based procedure

Upon receiving an OIDC Authentication Request message as specified in the OpenID Connect 1.0 \[11\] and IETF RFC 6749 \[9\] via a secure TLS tunnel between the SIM-C and the authorisation endpoint of the SIM-S, the SIM-S shall:

a\) validate the received OIDC Authentication Request message as specified in the OpenID Connect 1.0 \[11\] and IETF RFC 6749 \[9\];

b\) generate an HTTP 200 (OK) response according to IETF RFC 9110 \[16\] including form data to prompt the VAL service user for their username and password credentials; and

c\) send the HTTP 200 (OK) response towards the SIM-C.

Upon receiving an HTTP POST request method from the SIM-C containing the VAL service user's username and password, the SIM-S authenticates the VAL service user and shall:

a\) generate an OIDC Authentication Response message as specified in OpenID Connect 1.0 \[11\] and IETF RFC 6749 \[9\] with the following clarifications:

1\) shall generate an HTTP 302 (FOUND) response according to IETF RFC 9110 \[16\]; and

2\) shall include the following parameters as specified in 3GPP TS 33.434 \[7\]:

\- code; and

\- state,

in the query component of the redirection URI contained in the Location header field of the HTTP FOUND request method using the "application/x-www-form-urlencoded" format as specified in W3C.REC-html401-19991224 \[6\]; and

b\) send the HTTP 302 (FOUND) response towards the SIM-C.

Upon receiving an OIDC Token Request message via a secure TLS tunnel established between the SIM-C and the token endpoint of the SIM-S, the SIM-S shall:

a\) validate the OIDC Token Request message and if valid shall generate an OIDC Token Response message as specified in OpenID Connect 1.0 \[11\] and IETF RFC 6749 \[9\] with the following clarifications:

1\) shall generate an HTTP 200 (OK) response according to IETF RFC 9110 \[16\];

2\) shall based on the received VAL user ID obtained from the received user authentication credentials, determine the VAL service ID of the VAL service user;

3\) shall include the:

\- access_token;

\- token_type; and

\- expires_in.

parameters and may include the:

\- id_token; and

\- refresh_token.

parameters as specified in 3GPP TS 33.434 \[7\]; and

4\) shall include the other required parameters as specified in OpenID Connect 1.0 \[11\] and IETF RFC 6749 \[9\]; and

b\) shall send the HTTP 200 (OK) response towards the SIM-C.

##### 6.2.2.2.2 CoAP based procedure

##### 6.2.2.2.2 CoAP based procedure

Upon receiving an ACE-OAUTH Token Request message with client credentials grant type as specified in IETF RFC 9200 \[19\] optionally via a secure (D)TLS tunnel between the SIM-C and the token endpoint of the SIM-S, the SIM-S shall:

a\) validate the ACE-OAuth Token Request message and if valid shall generate an ACE-OAuth Token Response message as specified in IETF RFC 9200 \[19\] with the following clarifications:

1\) shall generate an COAP 2.01 (Created) response according to IETF RFC 9200 \[19\];

2\) based on the received client credentials, shall determine the VAL user ID, VAL service ID of the VAL service user;

3\) shall include parameters:

\- access_token;

\- expires_in;

\- ace_profile; and

\- rs_cnf; and

4\) shall include the other required parameters as specified in IETF RFC 9200 \[19\]; and

b\) shall send the CoAP 2.01 (Created) response towards the SIM-C.

### 6.2.3 Token exchange procedure


#### 6.2.3.1 SIM-C procedure

Upon receiving a request from the VAL user to acquire a security token for authentication of the VAL services, the SIM-C shall:

a\) establish a TLS tunnel to the token endpoint of the SIM-S; and

b\) send a Token Exchange Request message as specified in 3GPP TS 33.434 \[7\] and IETF RFC 8693 \[8\] using an HTTP POST request method towards the SIM-S according to IETF RFC 9110 \[16\]. The following parameters shall be included in the entity body of the HTTP POST request using the "application/x-www-form-urlencoded" format as specified in W3C.REC-html401-19991224 \[6\]:

\- grant_type;

\- code;

\- client_id;

\- redirect_uri; and

\- code_verifier.

Upon receipt of an HTTP 200 (OK) response from SIM-S, the SIM-C shall extract the security token contained in the access_token parameter of the received Token Exchange Response message as specified in IETF RFC 8693  \[8\] and send it to the VAL user.

#### 6.2.3.2 SIM-S procedure

Upon receiving a Token Exchange Request message as specified in IETF RFC 8693 \[8\] via a secure TLS tunnel between the SIM-C and the token endpoint of the SIM-S, the SIM-S shall:

a\) validate the received Token Exchange Request message as specified in IETF RFC 8693 \[8\]; and

b\) send a Token Exchange Response message as specified in IETF RFC 8693 \[8\] and IETF RFC 6749 \[9\] using an HTTP 200 (OK) response to the SIM-C according to IETF RFC 9110 \[16\]. The following parameters shall be included,

\- access_token;

\- token_type; and

\- expires_in.

and the following parameters may be included,

\- id_token; and

\- refresh_token.

in the HTTP 200 (OK) response and are serialized into a JavaScript Object Notation (JSON) structure as specified in IETF RFC 8693 \[8\] and IETF RFC 7159 \[10\].

## 6.3 Off-network procedures

The off-network procedures are out of scope of the present document in this release of the specification.
