---
spec: TS 29.500
version: 18.10.0
release: '18'
clause: '3'
title: 3 Definitions and abbreviations
source_archive: 29500-ia0.zip
source_document: 29500-ia0.docx
source_archive_sha256: 3e0cbb6bc6ceec4dabb597410fb4b55ba03ae82e9b242972f441cd926971f5f2
source_document_sha256: 8cd4994735a1718db905b0aee4ceba4fb479cc7193d82d051a68450f33eba52b
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# 3 Definitions and abbreviations


## 3.1 Definitions


For the purposes of the present document, the terms and definitions given in 3GPP TR 21.905 \[1\], 3GPP TS 23.501 \[3\] and the following apply. A term defined in the present document takes precedence over the definition of the same term, if any, in 3GPP TR 21.905 \[1\].

**Binding indication (consumer):** Binding can be used by the NF Service Consumer to indicate suitable NF Service Consumer instance(s) for notification target instance selection, reselection and routing of subsequent notification requests associated with a specific notification subscription. Binding indication needs to be stored by the NF Service Producer. Binding indication may also be used later if the NF Service Consumer starts acting as NF Service Producer, so that service requests can be sent to this NF Service Producer. See clauses 3.1 and 6.3.1.0 in 3GPP TS 23.501 \[3\]. See also Routing binding indication.

**Binding indication (producer):** Binding can be used to indicate suitable target NF Service Producer instance(s) for an NF service instance selection, reselection and routing of subsequent requests associated with a specific NF Service Producer resource (context) and NF service. Binding allows the NF Service Producer to indicate to the NF Service Consumer if a particular context should be bound to an NF service instance, NF instance, NF service set or NF set. Binding indication needs to be stored by the NF Service Consumer. See clauses 3.1 and 6.3.1.0 in 3GPP TS 23.501 \[3\]. See also Routing binding indication.

**Binding entity:** Either of the following identifiers: NF Service Instance, NF Service Set, NF Instance or an NF Set. The relation between these are explained below.

**Binding entity ID:** An identification of a binding entity, i.e. NF Service Instance ID, NF Service Set ID, NF Instance ID or an NF set ID.

**Binding level:** A parameter (bl) in "3gpp-Sbi-Routing-Binding" and "3gpp-Sbi-Binding" HTTP custom headers, which indicates the binding entity towards which a preferred binding exists (i.e. either to NF Service Instance, NF Service Set, NF Instance or an NF Set). Other binding entities in these headers, which do not correspond to the binding level indicate alternative binding entities that can be reselected and that share the same resource contexts (see Table 6.3.1.0-1 in 3GPP TS 23.501 \[3\]).

**Callback URI:** URI to be used by an NF Service Producer to send notification or callback requests.

**Endpoint address:** An address in the format of an IP address, transport and port information, or FQDN, which is used to determine the host/authority part of the target URI. This Target URI is used to access an NF service (i.e. to invoke service operations) of an NF service producer or for notifications to an NF service consumer. See clauses 3.1 and 6.3.1.0 of 3GPP TS 23.501 \[3\].

**NF Instance:** An identifiable instance of the NF. An NF Instance may provide services offered by one or more NF Service instances.

**NF Service Instance:** An identifiable instance of the NF service.

**NF Service Set:** A group of interchangeable NF service instances of the same service type within an NF instance. The NF service instances in the same NF Service Set have access to the same context data.

**NF Set:** A group of interchangeable NF instances of the same type, supporting the same services and the same Network Slice(s). The NF instances in the same NF Set may be geographically distributed but have access to the same context data.

**Notification endpoint:** Notification endpoint is a destination URI of the network entity where the notification is sent. See clause 6.3.1.0 in 3GPP TS 23.501 \[3\].

**Routing binding indication:** Information included in a request or notification and that can be used by the SCP for discovery and associated selection to of a suitable target. See clauses 3.1, 6.3.1.0 and 7.1.2 in 3GPP TS 23.501 \[3\]. Routing binding indication has similar syntax as a binding indication, but it has different purpose. Routing binding indication provides the receiver (i.e. SCP) with information enabling to route an HTTP request to an HTTP server that can serve the request. Routing binding indication is not stored by the receiver.

## 3.2 Abbreviations


For the purposes of the present document, the abbreviations given in 3GPP TR 21.905 \[1\] and the following apply. An abbreviation defined in the present document takes precedence over the definition of the same abbreviation, if any, in 3GPP TR 21.905 \[1\].

GZIP GNU ZIP

LC-H Load Control based on LCI Header

LCI Load Control Information

MCX Mission Critical Service

MPS Multimedia Priority Service

OCI Overload Control Information

OLC-H Overload Control based on OCI Header

SCP Service Communication Proxy

SEPP Security and Edge Protection Proxy

SMP SBI Message Priority

## 3.3 Special characters, operators and delimiters


### 3.3.1 General


A number of characters have special meaning and are used as delimiters in this document and also in other stage 3 SBI specifications. Below clauses specify the usage of a selected set of the special characters. Full set of these special characters are specified in the respective IETF specifications.

### 3.3.2 ABNF operators


/ Operator. The forward slash character separates alternatives. See clause 3.2 in IETF RFC 5234 \[43\].

\# Operator. The number sign character allows for compact definition of comma-separated lists, similar to the "\*" operator. See clause 2.1 in IETF RFC 9110 \[11\].

= Special character. The equal sign character separates an ABNF rule name from the rule elements. See clause 2.2 in IETF RFC 5234 \[43\].

\[ \] Operator. The square bracket characters enclose an optional element sequence. See clause 3.8 in IETF RFC 5234 \[43\].

\< \> Special characters. The angle bracket characters typically enclose an ABNF rule element (they are optional). See clause 2.1 in IETF RFC 5234 \[43\].

\* Operator. The star character precedes an element and indicates the elements repetition. See clause 3.6 in IETF RFC 5234 \[43\].

; Operator. Semicolon character indicates the start of a comment that continues to the end of line. See clause 3.9 in IETF RFC 5234 \[43\].

NOTE: The same characters, like "/", "#", etc. lead to different processing in ABNF and URI grammars. For instance, in URI syntax, ";" character separates parameter and its value, while in ABNF ";" starts a comment. Besides, unlike URI syntax, neither "?", nor ":" operators are specified for ABNF.

### 3.3.3 URI – reserved and special characters


Special characters that are used as delimiters in URI syntax have somewhat different purpose from the same characters when used by ABNF syntax. See clause 3.3.3 in 3GPP TS 29.501 \[5\].

### 3.3.4 SBI specific usage of delimiters


See clause 3.3.4 in 3GPP TS 29.501 \[5\].
