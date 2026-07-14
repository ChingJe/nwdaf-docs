---
spec: TS 29.510
version: 18.11.0
release: '18'
clause: Annex D
title: 'Annex D (normative): Support of "Canary Release" testing in the NRF'
source_archive: 29510-ib0.zip
source_document: 29510-ib0.docx
source_archive_sha256: 84e5ccfacace52d488cdca8f0ee1cb3b6817d390ee9903dc5e52105d58c6a1e6
source_document_sha256: 01abf6c9240ec101eef77e3ffc4bab5aca41dfd0d6d02c7515e02364cea887d9
content_origin: 3gpp-source
conversion: deterministic-pandoc-structure
---

# Annex D (normative): Support of "Canary Release" testing in the NRF


This feature allows a network operator to deploy new software features in a Network Function (NF) in a controlled manner, by isolating the NF (Service) Instances that implement the new features, and steering solely a part of the traffic that, otherwise, would be sent towards such NF (Service) Instance producer.

This is achieved by means of the following mechanisms:

\- Setting the NF (Service) Instance in Canary Release condition. This can be done by:

\- setting the NFStatus, or NFServiceStatus, of the NF service producer, to the value "CANARY_RELEASE", or

\- setting the "canaryRelase" attribute to true in the NFProfile or NFService, while keeping the NF(Service)Status as "REGISTERED"

\- Defining in the NFProfile or NFService of the NF service producer a set of selection conditions, that will be evaluated by an NF service consumer when attempting to select a candidate producer

The set of conditions may include, e.g.:

\- The NF type of the consumer

\- A feature number that is required by the consumer to select (and send traffic to) a producer

\- A set of specific UEs (e.g. based on SUPI ranges, GPSI ranges, IMPU/IMPI ranges, list of PEIs)

\- Any UE camping on a TAI within a set of TAI ranges

\- A list of DNNs

In order to allow flexibility in the definition of the selection conditions, the above conditions can be combined by means of "and" / "or" logical operators.

EXAMPLE: An SMF is deployed with a new software feature, and the operator wishes to test it by carrying traffic to it with the following conditions:

\- The SMF shall ony be selected by an AMF, a NEF, or an NWDAF

\- The SMF shall only be selected by the AMF when:

\- its selection requires the support of the "ATSSS" feature (which corresponds with feature number \#2 in the "nsmf-pdusesssion" service)

\- the UE belongs to a certain range of SUPIs

\- the UE is camping on a TAI belonging to a range of TAIs

\- When the SMF is selected by the NEF, it shall only do it for a certain DNN

\- When the SMF is selected by the NWDAF, it shall onlly do it when the UE is camping on a TAI belonging to a range of TAIs

> An example of the "selectionConditions" attribute could be as follows (note that there might be different logical expressions to encode the same logic):
>
> "selectionConditions": {
>
> "or": \[
>
> {
>
> "and": \[
>
> { "consumerNfType": \[ "AMF" \] },
>
> { "serviceFeature": 2 },
>
> { "supiRange": { "start": "1234511111", "end": "1234599999" } },
>
> { "taiRange": {
>
> "plmnId": { "mcc": "123", "mnc": "45" },
>
> "tacRangeList": \[
>
> { "start": "000011", "end": "0000ff" }
>
> \]
>
> }
>
> }
>
> \]
>
> },
>
> {
>
> "and": \[
>
> { "consumerNfType": \[ "NEF" \] },
>
> { "dnnList": \[ "internet.operator.com" \] }
>
> \]
>
> },
>
> {
>
> "and": \[
>
> { "consumerNfType": \[ "NWDAF" \] },
>
> { "taiRange": {
>
> "plmnId": { "mcc": "123", "mnc": "45" },
>
> "tacRangeList": \[
>
> { "start": "000011", "end": "000022" }
>
> \]
>
> }
>
> }
>
> \]
>
> }
>
> \]
>
> }

or, alternatively, with a more simplified encoding, based on the fact that the individual conditions (attributes) inside ConditionItem (see clause 6.1.6.2.124) are evaluated following the "AND" logical operator:

> "selectionConditions": {
>
> "or": \[
>
> {
>
> "consumerNfType": \[ "AMF" \],
>
> "serviceFeature": 2,
>
> "supiRange": { "start": "1234511111", "end": "1234599999" },
>
> "taiRange": {
>
> "plmnId": { "mcc": "123", "mnc": "45" },
>
> "tacRangeList": \[
>
> { "start": "000011", "end": "0000ff" }
>
> \]
>
> }
>
> },
>
> {
>
> "consumerNfType": \[ "NEF" \],
>
> "dnnList": \[ "internet.operator.com" \]
>
> },
>
> {
>
> "consumerNfType": \[ "NWDAF" \],
>
> "taiRange": {
>
> "plmnId": { "mcc": "123", "mnc": "45" },
>
> "tacRangeList": \[
>
> { "start": "000011", "end": "000022" }
>
> \]
>
> }
>
> }
>
> \]
>
> }

Once the producer has defined the selection conditions in its NFProfile, registered at the NRF, the sequence of steps for selection an NF after it is deployed with a "canary" software release, would be as follows:

1\. NF Service Instance to be software-upgraded changes its status to "UNDISCOVERABLE", and potential consumers are notified

2\. NF Service Instance gets progressively emptied of existing sessions, until no sessions remain

3\. NF Service Instance may optionally change its status to "SUSPENDED", to ensure that no residual traffic is sent to the NF Service Instance, and potential consumers are notified

4\. NF Service Instance is software-upgraded

5\. NF Service Instance with new software registers in NRF indicating its Canary Release condition, by setting its NF(Service)Status to "CANARY_RELEASE" or by setting the "canaryRelease" attribute to true while keeping its NF(Service)Status as "REGISTERED", and includes the set of conditions for selection (e.g. a SUPI range)

6\. Consumer NF sends discovery to NRF with usual discovery parameters and gets matching NF Instances containing NF Services with status "REGISTERED" or "CANARY_RELEASE"

7\. Consumer performs selection from the list of candidate NFs and, if the producer NF is in Canary Release condition, such NF shall only be selected if the selection conditions match (e.g. it shall only be selected if the SUPI matches the SUPI range indicated by the producer NF).

The selection of the candidate producers shall take into account the attributes of the discovered NFProfiles, and in addition, the consumer shall evaluate the attributes in the selection conditions, which shall take precedence over the attributes of the NFProfile of the producer, for those NF (Service) Instances in Canary Release condition.

If multiple candidate producers are available with NF (Service) status set to "REGISTERED" or "CANARY_RELEASE", the consumer shall preferably select a producer in "CANARY_RELEASE" status except if the "exclusiveCanaryReleaseSelection" attribute is present and set to true; in this case, the consumer shall only select producers in Canary Release condition (for which the selection conditions match), and shall not select those producers that are not in Canary Release condition.

NOTE: In the scenario of exclusive selection of Canary NF producers, the operator typically conducts very restricted Canary tests where the NF consumer only considers as valid candidate NF producer those instances in Canary condition, and discards all other NF producers (in non-Canary condition); the benefit of these tests is to check if the traffic case (for the selection conditions of the Canary nodes) actually end up in success or failure, rather than, e.g., ensuring that the traffic cases complete successfully after the consumer re-selects a non-Canary candidate producer. In these test scenarios, the operator need to ensure that the "exclusiveCanaryReleaseSelection" flag is consistently set in all the Canary NF producers; otherwise, the consumer can send traffic exclusively to Canary NF producers as long as there is one instance among the candidate producers having the "exclusiveCanaryReleaseSelection" set to true.

For the case of Indirect Communication, the selection of a candidate producer may be done by the SCP. In that case, the SCP needs to be able to evaluate the selection conditions for those producers in Canary Release condition. Since the SCP does not count with this information at its disposal (e.g. the different identities of the UE for which a service request is invoked via the SCP), it shall be provided by the consumer, e.g. by including the "3gpp-Sbi-Correlation-Info" HTTP header or by including the corresponding discovery headers ("3ggp-Sbi-Discovery") containing the UE identities involved in the specific traffic case.

In certain cases, the operator may want to maintain the NF Instance fully operative (so it keeps serving traffic to any consumer), at the same time as testing the new software features; in such case, the operator may deploy distinct service instances, some of which may keep the old software version and keep the "REGISTERED" NFServiceStatus, while other service instances may be deployed with the new software version, and set the NFServiceStatus to "CANARY_RELEASE", to ensure that it is only selected by consumers when the desired conditions are met.

Another alternative approach to maintain the node fully operative may be achieved by setting the "canaryRelease" indication to true in the NFProfile or NFService of the NF (Service) instance, while keeping both the NFStatus and NFServiceStatus as "REGISTERED". This way, all consumers that do not support the Canary Release feature will keep discovering and potentially selecting such NF (Service) instance, so it will keep serving traffic normally for legacy consumers; however, if the consumer supports the Canary Release feature, it shall check the "canaryRelease" flag and, if set to true, it shall evaluate the selection conditions, similarly as if the NF(Service)Status would be set to "CANARY_RELEASE".

An SCP or SEPP Network Entities may be set under Canary Release condition and define a set of selection conditions at NFProfile level to indicate to potential consumers that such Network Entity (SCP or SEPP) should only be selected if the selection conditions match.

Given that there are no services defined for an SCP, it is not possible to have an SCP "operative" (i.e. to be able to serve traffic to any consumer) when the status is set to "CANARY_RELEASE" and the consumers might not support the "CANARY_RELEASE" feature. However, it is possible to keep the SCP operative (and serve traffic to consumers not supporting such feature) by setting the "canaryRelease" attribute to true in the NFProfile of the SCP, and keep the NFStatus as "REGISTERED".

For a SEPP, the same considerations as with SCP apply, since there are no SEPP services per-se, in relation to the selection of SEPP instances, and the invocation of its N32-related message transfer capabilities.

However, a SEPP Network Entity may set the NFServiceStatus of a "nsepp-telescopic" service instance (while keeping the NFStatus of the SEPP set to "REGISTERED"). In this case, it is possible to have several instances of this service deployed, where some of them have the NFServiceStatus set to "REGISTERED" (and any consumer may select them) and others set to "CANARY_RELEASE", so consumers should only select such service instance when the selection conditions match.
