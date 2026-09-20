# Paper-first implementation audit: HFL over NWDAF

Date: 2026-09-20
Paper anchor: `HFL-NWDAF-free5GC-paper.tex` (latest workspace revision)
Implementation inspected: `Intelligent-Systems-Lab/PyMTLF`, commit `bdbd2a9591e47d59b690f0031dfebc77f757bb91` on `feat/hierarchical-fl-protocol-extension`

## Executive conclusion

The implementation is a substantial and internally consistent prototype of recursive hierarchical FL preparation, per-edge subscription management, hierarchical aggregation, and same-level intermediate-branch replacement. Its test suite passes (`661 passed, 2 skipped`) and lint passes.

However, the paper is ahead of the implementation in four material areas:

1. The paper's candidate Stage-3 contract and the runtime wire contract are different protocol designs. The paper uses `flTopology`/`flTopologyReport`, topology versions, explicit reparenting instructions, and edge-state reports. The code uses `x-flTopology`/`x-flTopologyReport` and a recursive node/status representation without those fields.
2. E1 (A fails and A* adopts A1/A2) is implemented, but E2a (A1/A2 attach directly to Root) and E2b (only A1 attaches to Root and A2 is reported unrealized) are not implemented end-to-end.
3. The repository does not contain the paper's complete free5GC multi-container deployment, A/B/C/A* experiment topology, data-partition generator, failure-injection controller, five-seed runner, raw results, or analysis scripts. The current numerical E0/E1 results therefore cannot be reproduced from this ZIP alone.
4. The experiment recorder captures several useful events, but not enough to calculate every protocol metric required by the paper or to prove topology-versioned requested/realized/accepted state.

The recommended direction is to retain the paper's protocol design as the target and bring the implementation and experimental artifacts into conformance. The paper should not be weakened to describe only the current E1 code path.

## Verification performed

- Read the complete current manuscript source and its appendices.
- Inspected the feature branch's wire models, topology planner, candidate pool, Root, Branch, Client, Server, NRF discovery, experiment recorder, configuration, tests, and Git history.
- Ran the full test suite with environment proxy inheritance disabled: `661 passed, 2 skipped, 16 warnings`.
- Ran Ruff: all checks passed.
- Confirmed the working tree remained unchanged.

The two skipped tests and warnings do not alter the conclusions below. Most protocol recovery tests are component tests with mocked peer/network boundaries; they do not replace a real multi-container free5GC experiment.

## Paper-to-implementation traceability matrix

| Paper requirement or claim | Status | Implementation evidence | Required action |
|---|---|---|---|
| Extend the existing ML model training lifecycle rather than define a parallel service | Implemented in PyMTLF | Existing subscription create/PUT/PATCH/DELETE and callback paths are reused in `api/ml_model_training.py`; Root and Branch create direct-child training subscriptions in `fl_server.py`. | Retain. Verify the containing Go NWDAF exposes the corresponding public Release-18/19 resource paths and representations. |
| Use one `mlCorreId` throughout the hierarchy | Implemented | Root uses the plan ID as process/correlation ID; Branch forwards the same ID; validation rejects missing or changed IDs. | Retain. Add a run-level assertion and structured trace record proving one value across every edge and event. |
| Independent subscription resource for every parent-child edge | Implemented in PyMTLF | Root-to-Branch and Branch-to-child subscriptions receive separate notification correlations and resource locations. | Retain. Record all resource IDs/locations in the experiment evidence and verify the Go routing ledger. |
| Recursive, role-neutral topology instruction | Partially implemented | `FlTopologyNode.children` is recursive and bounded to depth 16 / 1024 nodes. Intermediate/leaf role is inferred from the received node. The Root's static planner, however, is fixed to Root -> Branch group -> Leaf and cannot express an arbitrary graph or direct Root-to-Leaf repair. | Generalize the Root topology state and repair planner. Add an end-to-end depth-greater-than-three test only if arbitrary recursion remains a headline claim. |
| Requested, realized, and accepted topology are distinct | Partially implemented | Requested state exists as static assignment/instructions; realized state exists as recursive child status reports; Root keeps an internal admission snapshot. There is no wire-level topology version or explicit accepted-topology object/report. | Add versioned topology state and persist/request/report/accept transitions. Define whether acceptance is an internal Root decision or an explicit confirmation sent to participants. |
| `flTopology` and `flTopologyReport` wire properties | Not implemented; direct mismatch | Code and tests use `x-flTopology` and `x-flTopologyReport`. | Rename the wire aliases and all tests/config/examples to the paper's standardized-candidate names. A compatibility parser may temporarily accept old names, but emitted payloads should use the paper contract. |
| Paper Appendix-B `FLTopologyInstruction` schema | Not implemented | Runtime uses `nfInstanceId`, policy, strategy, report-after, enabled/priority, retained-result request, and recursive children. It lacks `topologyVersion`, `addressedNodeId`, `allowNrfDiscovery`, `minDirectChildren`, `maxDepthBelow`, `candidates`, and `reparentInstruction`. | Implement the paper schema or formally revise Appendix B after a deliberate design review. Do not claim that the current code implements Appendix B. |
| Paper Appendix-B `FLTopologyReport` and direct-edge states | Not implemented | Runtime report contains recursive nodes with status/timestamp/cause; it lacks topology version, reporting node state, direct edge records, child edge state, and explicit failure cause per edge in the paper's form. | Implement versioned node/edge reporting and map every direct subscription resource to an edge report. |
| Feature negotiation using `suppFeats` | Partially implemented | Code assigns hierarchical orchestration to feature number 3 and requires negotiated support. The paper correctly says a final CR must assign a feature number. | Treat bit 3 as a prototype-local allocation, document it, and avoid implying 3GPP assignment. Rebase and assign the final bit only through the proposed CR. |
| Initial hierarchy formation and recursive confirmation | Implemented for the current recursive-node contract | Root sends topology-bearing preparation; Branch creates downstream subscriptions; child reports are composed upward; Root validates minimum participation before admission. | Port this behavior to the paper's versioned instruction/report schema and add a real multi-container formation trace. |
| Same-level branch replacement E1: A* adopts A1/A2 | Implemented | Root detects one failed direct Branch during a round, retires it, selects the next Branch candidate, sends the original descendant assignment to A*, and continues while replacement runs asynchronously when Root policy permits. Leaf rebind/supersession logic is present. | Add real integration tests and five-seed experiments. Record new Root-A* and A*-A1/A2 resources plus unchanged B/C resources. |
| Localized repair preserves unaffected B/C relationships | Implemented by control flow; incompletely instrumented | Root removes only the failed participant and retains the existing Server process and remaining participant objects. Tests verify cohort preservation. | Add structured before/after edge snapshots and immutable resource IDs so the experiment can prove preservation without relying on prose logs. |
| Explicit descendant reparenting semantics | Partially implemented for E1 only | A* receives a complete subtree instruction containing A1/A2. There is no explicit `failedParentNfId`/`adoptChildNfIds` reparent instruction as defined in the paper. | Implement `ReparentInstruction` and validate authority, stale versions, duplicate parents, and cycles. |
| Cross-level reparenting E2a: Root directly adopts A1/A2 | Not implemented | Root resolves replacement candidates specifically as Branches; static branch groups require leaves; Root rounds expect `HIERARCHY_AGGREGATE` results from top-level participants. Direct leaf results use a different artifact role. | Generalize top-level participants to Branch or Leaf, create Root-A1/A2 subscriptions, accept both leaf training results and branch aggregates in one Root round, and weight by represented sample count. |
| Partial flattening E2b: Root adopts A1, reports A2 unrealized, and applies acceptance policy | Not implemented end-to-end | Candidate reports can contain failed/inactive nodes, and completion thresholds exist, but Root cannot perform direct leaf adoption and there is no paper-style requested/realized/accepted versioned topology. | Build on E2a, add explicit unrealized-edge reporting, an acceptance decision with reason, and class-wise evaluation. |
| Continue accepted rounds during a Branch outage | Implemented when policy permits | Root can aggregate only successful selected branches when `acceptFailures` and completion-rate policy allow it. Replacement occurs asynchronously. | Ensure experiment configuration is committed with Root minimum 2 of 3 and the intended completion rate. Do not imply exactly two degraded rounds; that remains an observation. |
| Detection is separate from repair | Partially implemented | A failure becomes a typed round participant failure after PATCH transport/HTTP failure or timeout; Root then invokes repair. There is no pluggable detector or Root use of NRF status notifications/liveness for this path. | Keep the design principle, but describe the evaluated detector as round-operation timeout. Optionally add an interchangeable detector interface and NRF notification/liveness implementation. |
| Failure-to-detection near 298 seconds under a 300-second timeout | Plausible but not reproducible from this ZIP | Default round timeout is 300 seconds; the recorder logs detection time but not injection time. No raw run is committed. | Record `FAILURE_INJECTED` with monotonic and UTC timestamps, exact command/signal, process/container ID, and corresponding detection event. Commit raw evidence. |
| At most one intermediate Branch failure per round | Implemented limitation | Root explicitly terminates recovery when more than one direct Branch fails in a round. | Keep this limitation in the paper. Add a negative test/experiment only if space permits; do not imply arbitrary simultaneous-failure tolerance. |
| Root failure, host failure, and network partition are covered | Not implemented and not claimed by the current paper | E1 is service/process failure with live Root, leaves, and replacement. | Retain the current limitation. |
| NRF-based capability discovery | Implemented on the PyMTLF side | PyMTLF asks the containing Go NWDAF's internal NRF proxy for NWDAFs advertising the requested training service/capability/model interoperability. | Obtain and archive the Go NWDAF/free5GC repositories and commits; run an integration trace against the actual NRF. |
| free5GC-based multi-NWDAF realization | Partially evidenced by this repository | PyMTLF is explicitly a private MTLF backend and delegates public SBI/NRF routing to a containing Go NWDAF. The ZIP contains neither that Go implementation nor the multi-container deployment. | Publish/pin the Go NWDAF/free5GC side, Compose/Kubernetes manifests, NF profiles, and exact commits. The paper's careful term “free5GC-based” is appropriate. |
| OAM bootstraps deployment/policy | Not implemented as an interface | Equivalent policy is local YAML; the paper already says OAM provisioning is not evaluated. | No implementation is required for the current contribution. Keep it architectural context and do not mark it as evaluated. |
| OpenAPI specification and generated-code provenance | Missing | FastAPI can emit runtime OpenAPI, but no versioned Stage-3 YAML patch, generated client/server code, or 3GPP toolchain result is committed. | Add a standalone candidate OpenAPI fragment/full patch, validation job, generated artifacts if used, and provenance/commit metadata. |

## Experiment-readiness audit

### Configuration conflicts with the paper

The repository's checked-in sample configuration is not the paper's experiment configuration:

| Item | Paper | Checked-in sample | Action |
|---|---:|---:|---|
| Topology | Root + A/B/C + six leaves + A* | One Branch + two leaves; no A/B/C/A* deployment manifest | Commit separate E0/E1/E2a/E2b topology profiles and the actual container deployment. |
| Accepted Root rounds | MNIST 24; CIFAR-10 40 | 2 | Add workload/condition profiles matching the paper. |
| Local epochs | MNIST 4; CIFAR-10 5 | Leaf example uses 18 | Make epochs condition/workload controlled and record the effective value. |
| Batch size | 16 | 32 | Align the implementation configuration or correct the manuscript if 32 was actually used in the original runs. |
| Learning rate | 0.001 | 0.001 | Aligned. |
| FedProx coefficient | 0.01 | 0.01 | Aligned. |
| Aggregation | Sample-count weighted | `sampleWeighted` | Aligned in intent. |
| Root degraded-round policy | At least two of A/B/C | Sample topology requires its only Branch and does not accept failures | Commit the real Root policy: minimum 2 of 3 and a matching completion-rate rule. |
| Round timeout | 300 s | 300 s | Aligned. |
| Five paired seeds | Five seeds, same paired conditions | Runtime default training seed 42; initial image weights have fixed seeds 101/202; no sweep runner | Add a run manifest and seed plumbing for partition, model initialization, data-loader order, candidate selection, and failure timing. |
| Data partitions | Six 8,000-sample non-IID leaf shards; separate validation/test sets | Runtime accepts externally mounted `.npz` shards but no generator/manifests/checksums are committed | Commit deterministic partition generation and a per-seed manifest with class histograms and SHA-256 values. |

The current code resets the training RNG to the configured client seed for each local training call. This is deterministic, but a five-seed study must define whether the seed varies model initialization, partition construction, minibatch ordering, or all three. Each source of randomness should be explicit rather than represented by one ambiguous “seed.”

### What the recorder can support now

The recorder currently writes:

- root/branch/leaf model evaluations with validation loss and accuracy;
- Root round outcome with selected, successful, and failed direct participants;
- Branch failure detected;
- Branch replacement ready;
- final model artifact and digest.

From those events, the implementation can derive accepted participant sets, detection-to-replacement-ready time, replacement-ready-to-first-successful-contribution time, validation trajectories, and final artifact identity.

### Missing evidence required by the paper

Add structured records for:

1. `RUN_MANIFEST`: condition, workload, seed components, commits, image/container digests, hardware allocation, topology policy, training settings, dataset shard hashes, and start time.
2. `TOPOLOGY_REQUESTED`, `TOPOLOGY_REALIZED`, and `TOPOLOGY_ACCEPTED`: topology version, complete direct-edge list, parent/child IDs, edge state, subscription ID/location, and decision reason.
3. `SUBSCRIPTION_CREATED`, `SUBSCRIPTION_DELETED`, and cleanup failure: parent, child, `notifCorreId`, resource ID/location, topology version, and `mlCorreId`.
4. `FAILURE_INJECTED`: UTC and monotonic timestamps, target, process/container generation, signal/command, and injection round.
5. `REPAIR_INSTRUCTION_SENT`, `REPAIR_EDGE_READY`, and `REPAIR_ACCEPTED`: versioned repair phase timing.
6. `FIRST_POST_REPAIR_CONTRIBUTION`: or a deterministic extractor that derives it from Root round outcomes.
7. Control-plane message counters by operation, edge, phase, HTTP outcome, and retry.
8. Per-class test metrics for E2b, because A2's permanent data loss may disproportionately affect its represented classes.

Add one analysis program that consumes only committed manifests and JSONL records and produces:

- repair success ratio;
- five observations plus mean and 95% Student-t CI;
- post-failure accuracy AUC over the preregistered common window `K`;
- endpoint accuracy/loss;
- accepted rounds to recovery;
- failure-to-detection, detection-to-instruction, instruction-to-edge-ready, edge-ready-to-contribution, and total failure-to-contribution;
- topology and subscription transition tables;
- message counts;
- confidence-band plots.

## Priority backlog

### P0: align the protocol contract before collecting final data

1. Choose the manuscript Appendix-B schema as the canonical target.
2. Rename emitted wire fields to `flTopology` and `flTopologyReport`.
3. Implement `topologyVersion` and reject stale/out-of-order instructions and reports.
4. Implement explicit candidate and reparenting instructions.
5. Implement direct-edge reports tied to actual subscription resources.
6. Define and implement the Root's topology acceptance decision.
7. Mark the current `suppFeats` bit as prototype-local until a CR assigns it.
8. Add schema, wire-round-trip, presence-rule, cycle, duplicate-parent, stale-version, and authorization tests.

Collecting final experiments before P0 risks producing evidence for a protocol different from the one proposed in the paper.

### P1: implement the two missing protocol claims

1. Replace the fixed “Root participants are Branches” assumption with a role-neutral direct-child abstraction.
2. Support mixed Root inputs in one round: Branch aggregates from B/C and direct leaf updates from A1/A2.
3. Implement E2a repair planning and subscription creation.
4. Implement E2b partial realization and policy acceptance with explicit A2 failure state.
5. Preserve/fence stale A and A2 callbacks and clean up reachable obsolete resources.
6. Add real HTTP integration tests, not only mocked Root/Server tests.

### P2: make the free5GC testbed reproducible

1. Publish the containing Go NWDAF/free5GC changes and pin exact commits.
2. Commit the multi-container deployment for Root, A/B/C, A*, six leaves, NRF, ADRF, and required core NFs.
3. Commit NRF profiles and the mapping between logical roles and containers.
4. Commit deterministic dataset partition generation and manifests.
5. Implement a condition/seed/workload experiment runner and failure injector.
6. Archive raw JSONL, container logs, manifests, and final models per run.

### P3: run the paper's final evaluation

Run E0, E1, E2a, and E2b for both MNIST and CIFAR-10 with five paired seeds. E2b provides the useful data-loss contrast while remaining tied to the proposed hierarchy repair protocol.

## Claims the paper can make today

Subject to retaining its current qualifications, the paper can already say that:

- the prototype implements hierarchical preparation and aggregation using direct ML-training subscription lifecycles;
- one `mlCorreId` is propagated through the hierarchy;
- E1-style same-level replacement logic exists and is well covered by component tests;
- the system can continue Root rounds using unaffected participants when the configured completion policy permits it;
- the code supports MNIST and CIFAR-10 workloads and structured per-round validation recording;
- the implementation is free5GC-oriented/free5GC-based, with PyMTLF intentionally behind a containing Go NWDAF.

The paper should not yet state as an implemented/evaluated fact that:

- Appendix B is the runtime OpenAPI contract;
- E2a or E2b has been realized;
- the protocol has versioned requested/realized/accepted topology state;
- all claimed subscription/topology evidence is available in the artifact;
- the complete testbed and current E0/E1 numerical results are reproducible from the public PyMTLF repository;
- five-seed statistical results exist;
- arbitrary recursive topologies or arbitrary failure patterns have been validated.

## Repository/artifact identifiers available now

The PyMTLF placeholder can eventually use:

- Repository: `https://github.com/Intelligent-Systems-Lab/PyMTLF`
- Audited implementation commit: `bdbd2a9591e47d59b690f0031dfebc77f757bb91`

Still required for the paper's artifact statement:

- containing Go NWDAF repository and commit;
- free5GC base repository/tag/commit;
- multi-container deployment repository/commit;
- candidate OpenAPI file and validation/code-generation provenance;
- experiment-data release and immutable identifier.
