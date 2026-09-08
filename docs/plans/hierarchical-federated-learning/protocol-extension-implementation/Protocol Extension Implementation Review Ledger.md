# Hierarchical NWDAF FL Protocol Extension Implementation Review Ledger

日期：2026-09-08

狀態：Slice 1、2、3、4A、4、5、6 Committed；Formal Testbed Validation Pending

相關文件：

- [Implementation Plan](./Hierarchical%20NWDAF%20FL%20Protocol%20Extension%20Implementation%20Plan.md)
- [Slice Map](./Protocol%20Extension%20Implementation%20Slice%20Map.md)
- [Slice 1 Detailed Plan](./slices/Slice%201%20Wire%20Contract%20and%20Resource%20Lifecycle%20Foundation%20Detailed%20Plan.md)
- [Slice 2 Detailed Plan](./slices/Slice%202%20Candidate%20Pool%20Policy%20and%20Local%20Contract%20Execution%20Detailed%20Plan.md)
- [Slice 3 Detailed Plan](./slices/Slice%203%20Branch%20Replacement%20without%20Retained-result%20Recovery%20Detailed%20Plan.md)
- [Slice 4A Detailed Plan](./slices/Slice%204A%20Digest%20Simplification%20and%20Contract%20Cleanup%20Detailed%20Plan.md)
- [Slice 4 Detailed Plan](./slices/Slice%204%20Controlled%20Local%20Training%20Workload%20Detailed%20Plan.md)
- [Slice 5 Detailed Plan](./slices/Slice%205%20Protocol-driven%20Hierarchy%20Integration%20Detailed%20Plan.md)
- [Slice 6 Detailed Plan](./slices/Slice%206%20Migration%20and%20Regression%20Closure%20Detailed%20Plan.md)

---

## 1. 紀錄範圍

本文件是 protocol extension implementation phase 的單一 review ledger。每個 slice
在此追加審查發現、修正、驗證與 closing commit，不為同一 slice 的每輪修正建立
另一份完整 review 文件。

Slice 1 的 production baselines：

| 儲存庫 | Branch | 基準 |
| --- | --- | --- |
| `NWDAF/` | `feat/hierarchical-fl-protocol-extension` | `6aed268d6528f8be6c729cbd45b59d067e5e80dc` |
| `PyMTLF/` | `feat/hierarchical-fl-protocol-extension` | `747962971b63f0a53031d52a1eb7e047ae776998` |
| `nwdaf-docs/` | `main` | `2964021c6858d1c6e55269898326082cd71177ff` 加上本次 unstaged evidence |

---

## 2. Slice 1 審查結果

### 2.1 計畫符合性

| 要求群組 | 狀態 | 直接證據 |
| --- | --- | --- |
| Typed candidate contract | 已滿足 | Go `internal/compat/mlmodeltraining` 與 Python `wire/ml_model_training.py` 的 typed aliases、closed nested validation、recursive bounds 與 round-trip tests |
| Receiver／participant identity | 已滿足 | Local `NWDAFContext.NfId`、peer `SelectedTarget.NFInstanceID`、route-bound participant state與 local／remote／callback tests |
| Persistent／operation separation | 已滿足 | Create／PUT／PATCH extraction、write-only success rejection、Go route與PyMTLF resource non-persistence tests |
| Feature state與gate | 已滿足 | Route offered／negotiated state、subset validation、unnegotiated operation rejection與PyMTLF feature-disabled `READY` tests |
| CRUD atomicity | 已滿足 | Local／peer `200`／`204`、destination failure、malformed success、stale revision、DELETE與generation reset tests |
| Structured errors | 已滿足 | Public handler、private gateway、processor與FastAPI alias-path `400` tests；capability `403` tests |
| Legacy regression | 已滿足 | Candidate-free focused tests與兩個repository的full test／lint／build gates |

Slice 1 沒有修改generated OpenAPI code、沒有新增Go package，也沒有把candidate
request接到legacy model-bundle execution。PyMTLF仍不advertise feature 3。

### 2.2 發現與修正

| ID | 狀態 | 確認證據 | 修正 | 驗證 | 收尾 commit |
| --- | --- | --- | --- | --- | --- |
| `S1-R1` | 已關閉 | Go handler／processor的candidate parse error曾可能失去extension path | 所有training pre-parser與processor統一使用candidate-aware `ProblemDetails` mapping | Public／private handler與processor structured-path tests | `NWDAF` `302762a` |
| `S1-R2` | 已關閉 | PyMTLF Pydantic validation與receiver mismatch未完整映射為alias-path `400` | 新增validation-location converter與`InvalidMessageError` API mapping | `tests/test_ml_model_training_api.py` | `PyMTLF` `6c27d15` |
| `S1-R3` | 已關閉 | Python `datetime.fromisoformat`與部分string checks比Go接受更寬鬆的輸入 | 對report timestamp使用strict RFC3339 gate，並拒絕空白aggregation、unit、status與retained outcome | `tests/test_ml_model_training_wire.py` | `PyMTLF` `6c27d15` |
| `S1-R4` | 已關閉 | Candidate PUT在feature gate前取得receiver context，可能回傳錯誤分類 | 先以resource negotiated state拒絕operation，再做receiver lookup | `test_unnegotiated_candidate_put_is_gated_before_context_lookup` | `NWDAF` `302762a` |
| `S1-R5` | 已關閉 | Candidate mutation的`200` immediate report曾使用mutation前的expected round驗證 | 改以PUT／PATCH的effective representation建立response identity | `TestMLModelTrainingCandidateMutationResponseUsesEffectiveRound` | `NWDAF` `302762a` |
| `S1-R6` | 已關閉 | Candidate typed round-trip tests 只抽查部分欄位，不能直接證明完整 extension contract 不會在 parse／encode 時遺失 | Go 與 Python 改以完整 JSON 結構相等比較驗證 Subscription、PATCH 與 Notify 的所有 candidate 欄位 | Go `TestCandidate*RoundTrip`；Python `test_candidate_*_round_trip_preserves_complete_extension_contract` | `NWDAF` `302762a`；`PyMTLF` `6c27d15` |

初始完整diff審查與每項修正的targeted follow-up review均已完成；目前沒有未關閉的
Slice 1 code finding。

---

## 3. 最終驗證

### 3.1 `NWDAF/`

| 命令 | 結果 |
| --- | --- |
| `go test ./internal/compat/mlmodeltraining` | Pass |
| `go test ./internal/context -run MLModelTraining` | Pass |
| `go test ./internal/sbi -run MLModelTraining` | Pass |
| `go test ./internal/mtlf -run MLModel` | Pass |
| `go test ./internal/sbi/processor -run MLModelTraining` | Pass |
| `go test ./internal/sbi/consumer -run MLModelTraining` | Pass |
| `make test` | Pass；environment-gated live backend tests維持既有skip行為 |
| `make lint` | Pass；`0 issues` |
| `make build` | Pass |

### 3.2 `PyMTLF/`

| 命令 | 結果 |
| --- | --- |
| `uv run pytest -q tests/test_ml_model_training_wire.py` | Pass；46 tests |
| `uv run pytest -q tests/test_ml_model_training_api.py` | Pass；7 tests，1個既有Starlette deprecation warning |
| `uv run pytest -q tests/test_fl_client.py` | Pass；45 tests |
| Focused `uv run ruff check ...` | Pass |
| `uv run pytest -q` | Pass；631 passed、2 CUDA-dependent tests skipped、46 dependency deprecation warnings |
| `uv run ruff check .` | Pass |

---

## 4. 明確延後的證據

下列項目依 Slice 1 detailed plan 明確延後，不由上述unit／fake-destination結果宣稱已
完成：

- `future-phase handoff`：candidate selection、policy／strategy execution與downstream
  subscription dispatch（Slice 2）；
- `future-phase handoff`：retained-result index、lookup與outcome producer（當時規劃為
  Slice 3；2026-09-07後由本文件§11取代，retained runtime維持暫緩）；
- `future-phase handoff`：Root／Branch protocol-mode orchestration、feature 3 production
  advertisement、ADRF global-model distribution與sender cleanup（Slice 5）；
- `future-phase handoff`：除whole-artifact key外的既有digest contract清理（Slice 4A）；
- `future-phase handoff`：controlled MNIST local workload與held-out evaluation（Slice 4）；
- `future-phase handoff`：legacy model-bundle cutover與移除（Slice 6）；
- `integration verification gap`：controlled local workload、real NRF、ADRF、multi-NWDAF testbed與
  protocol-driven HFL E2E。

這些項目不改變 Slice 1 僅完成wire contract與resource lifecycle foundation的邊界。

---

## 5. Slice 2 審查結果

### 5.1 計畫符合性

| 要求群組 | 狀態 | 直接證據 |
| --- | --- | --- |
| Effective contract與local defaults | 已滿足 | Typed `policy`／`strategy`／`reportAfter` resolver；protocol值不能被local default擴張；unsupported method／unit拒絕 tests |
| Candidate pool與hybrid provenance | 已滿足 | Explicit／locally-discovered provenance、priority、relationship state、revision fence、idempotent rediscovery與PATCH tests |
| Delegated discovery | 已滿足local boundary | 經containing Go internal NRF proxy的list query；event／interoperability／TAI抽取、profile／service revalidation、dedupe、`validityPeriod`保存、使用前freshness gate與successful snapshot reconciliation tests |
| Readiness與selection | 已滿足 | `minAvailableNodes`、`minTrainNodes`、fraction、priority／seeded-random selection及frozen selected-set tests |
| Completion與aggregation | 已滿足 | All-terminal wait、deadline failure、completion rate、failure policy、nonselected／late callback fence與真實sample-weighted aggregation tests |
| Leaf／Intermediate local work | 已滿足 | Leaf epoch／FedProx arguments進入既有trainer；Intermediate依序執行lower rounds並只回傳最終aggregate |
| Topology update與report | 已滿足local boundary | Upstream replacement、authority revocation／re-enable、DELETE retry intent、stable snapshot與unknown descendant preservation tests |
| Legacy assignment ingress | 已滿足 | Branch／Leaf透過真實`FLWorkspace`與`httpx.MockTransport`各只發一次GET；strict validation、adoption與cleanup regressions |
| Legacy regression | 已滿足 | Candidate-free behavior、full PyMTLF suite、ruff及未修改Go discovery boundary checks |

Slice 2 只建立local orchestration primitives與既有executor integration，未將candidate
contract接上Root→Branch→Leaf production message flow，也未advertise feature 3。

### 5.2 發現與修正

| ID | 狀態 | 確認證據 | 修正 | 驗證 |
| --- | --- | --- | --- | --- |
| `S2-R1` | 已關閉 | Upstream subtree省略identity時，原reconcile可能一併停用仍具local provenance的candidate | 只移除upstream authority；仍具local provenance且未被禁止者維持可用 | Candidate provenance／omission tests |
| `S2-R2` | 已關閉 | Delegated selection由啟用改為停用再啟用時，本地authority與candidate狀態可能無法正確恢復 | 明確處理authority revocation與re-enable，並保留upstream precedence | Policy reconfiguration tests |
| `S2-R3` | 已關閉 | DELETE失敗若直接清除intent，無法安全重試 | 保留revision-bound retry intent；成功或stale completion才結束對應工作 | DELETE retry／stale revision tests |
| `S2-R4` | 已關閉 | 相同discovery結果重複套用曾不必要提高relationship revision，使in-flight completion失效 | Idempotent rediscovery不改revision或timestamp | Fake-clock idempotence tests |
| `S2-R5` | 已關閉 | Single-fetch artifact在resource stale或plan bind失敗的pre-bind path可能遺留adopted directory | 將adopted artifact納入明確release owner並補齊失敗清理 | Artifact ownership／stale revision／bind failure tests |
| `S2-R6` | 已關閉 | 初始GET count與completion gate tests曾以mock helper／mock wait取代關鍵production behavior | 改以真實`FLClientEngine`、`FLWorkspace`、callback collection、deadline與real small-model aggregation驗證 | Branch／Leaf exact-one-GET、server callback／partial aggregate tests |
| `S2-R7` | 已關閉 | Delegated discovery最初只接受手動criteria，未直接證明標準request fields能供應必要輸入 | 從實際`NwdafMLModelTrainSubsc`抽取event、model interoperability與TAIs，缺少bounded scope時結構化拒絕 | Subscription requirement extraction tests |
| `S2-R8` | 已關閉 | `HierarchyNodeResolver`曾丟棄mandatory `SearchResult.validityPeriod`，`CandidatePool.add_discovered()`也只有增量加入；因此無法在使用NRF-derived candidate前判斷資料是否過期，也無法依fresh complete snapshot處理candidate增減 | Resolver回傳包含normalized scope、receipt time、`validityPeriod`、`validUntil`與result completeness的snapshot；pool以scope／refresh revision fence late result，過期或scope不符時阻擋local establishment，並依完整／部分snapshot及relationship status reconcile | Resolver 17 tests；candidate orchestration 34 tests；focused 246 passed、2 skipped；full 688 passed、2 skipped；ruff pass |

初始完整diff review、`S2-R1`至`S2-R7`的targeted remediation review，以及user review後
新增`S2-R8`的test-first remediation與targeted follow-up review均已完成。目前沒有未關閉的
Slice 2 code finding。

Slice 2 production closing commit：`PyMTLF` `0e87ef1`。

---

## 6. Slice 2 最終驗證

### 6.1 `PyMTLF/`

| 命令 | 結果 |
| --- | --- |
| `uv run pytest -q tests/test_fl_candidate_orchestration.py` | Pass；34 tests |
| `uv run pytest -q tests/test_fl_hierarchy_discovery.py` | Pass；17 tests |
| `uv run pytest -q tests/test_fl_hierarchy_artifacts.py` | Pass；46 tests，45個dependency deprecation warnings |
| `uv run pytest -q tests/test_fl_server.py` | Pass；67 tests |
| `uv run pytest -q tests/test_federated_trainer.py` | Pass；3 tests |
| `uv run pytest -q tests/test_local_trainer.py` | Pass；14 passed、2 skipped |
| `uv run pytest -q tests/test_fl_branch.py tests/test_fl_client.py` | Pass；65 tests |
| Focused `uv run ruff check ...` | Pass |
| Combined Slice 2 focused test set | Pass；246 passed、2 skipped、45 dependency deprecation warnings |
| `uv run pytest -q` | Pass；688 passed、2 skipped、55 dependency deprecation warnings |
| `uv run ruff check .` | Pass |

### 6.2 `NWDAF/` unchanged boundary

| 命令 | 結果 |
| --- | --- |
| `go test ./internal/backend ./internal/mtlf ./internal/sbi/consumer -run 'NFDiscovery\|Discovery'` | Pass；`internal/mtlf`沒有符合filter的tests，其餘packages通過 |

`NWDAF/` working tree維持clean；Slice 2不需變更Go internal NRF proxy contract。

---

## 7. Slice 2 明確延後與review gate

下列項目不是Slice 2已完成的證據：

- Root／Branch protocol-mode message wiring與recursive subscription forwarding；
- feature 3 production advertisement／negotiation success；
- Controlled local workload、ADRF global-model distribution及real NRF／ADRF／multi-NWDAF E2E；
- 除whole-artifact repository key外的bundle、model、training evidence、Notify、topology與
  collection digest cleanup；
- HTTP establishment／DELETE outcome、real Notify relay與peer callback E2E；
- retained-result index、lookup、retention lifecycle與recovery runtime。

`PyMTLF/` production與test changes已於`0e87ef1`收尾；`nwdaf-docs/`本次status／
evidence changes由獨立文件commit保存。Slice 2 local procedure與state requirements均具
direct evidence，但不得據此描述成hierarchical protocol E2E。

---

## 8. Slice 4 審查結果

### 8.1 計畫符合性

| 要求群組 | 狀態 | 直接證據 |
| --- | --- | --- |
| Known workload與config boundary | 已滿足 | Closed traffic／image profiles、MNIST／CIFAR-10 datasets、合法data-source組合與config tests |
| Profile-specific artifact contract | 已滿足 | Traffic保留scaler；image只接受model config／code／weights；workspace與import tests覆蓋exact inventory |
| Controlled model source | 已滿足 | MNIST／CIFAR-10 reproducible bundles、matching channels、獨立weights、無BatchNorm及trusted loading tests |
| Local image execution | 已滿足 | Local `.npz` loader、normalization、CrossEntropy、真實optimizer step與production FL Client round test |
| Generic FL reuse | 已滿足 | FedProx、actual sample count與production FL Server sample-weighted aggregation tests |
| Aggregation-only Branch | 已滿足 | Image profile不配置local shard仍可建立與執行lower-server aggregation path |
| Held-out evaluation | 已滿足 | Offline evaluator接受durable artifact key或workspace artifact path，輸出sample count與accuracy |
| Existing traffic behavior | 已滿足 | Traffic configuration、trainer、artifact、FL Client／Server及full-suite regressions通過 |
| Scope boundary | 已滿足 | 未修改`NWDAF/`、protocol schema、NRF或ADRF contract；protocol E2E保留給Slice 5 |

### 8.2 發現與修正

| ID | 狀態 | 確認證據 | 修正 | 驗證 |
| --- | --- | --- | --- | --- |
| `S4-R1` | 已關閉 | 初版image config要求所有nodes配置local shard，與aggregation-only Branch requirement衝突 | 只在真正執行local training時要求shard；aggregation-only Branch可省略training data | Config與Branch production-path tests |
| `S4-R2` | 已關閉 | 初版offline evaluator只接受durable repository key，無法評估尚未進traffic catalog的final FL workspace artifact | 增加互斥的artifact key／artifact path輸入，兩者重用同一trusted loader | Known-result與CLI workspace-artifact tests |
| `S4-R3` | 已關閉 | 初版關鍵tests只直接呼叫training／aggregation helpers，對production owners的連接證據不足 | 新增真實`FLClientEngine._run_round`與`FLServerEngine._aggregate_round` tests，只mock transport／callback boundary | Image Client round與two-client aggregation tests |

Initial full-diff review與每項修正的targeted follow-up review均已完成；目前沒有未關閉的
Slice 4 code finding。

### 8.3 最終驗證

| 命令／證據 | 結果 |
| --- | --- |
| `.venv/bin/pytest -q` | Pass；710 passed、2 skipped、55個dependency deprecation warnings |
| `.venv/bin/ruff check .` | Pass |
| `git diff --check` | Pass |
| Real MNIST local smoke | Pass；2 Clients各64 training samples，128 held-out samples，accuracy `0.078125` |

Smoke從workspace-local raw IDX cache一次性建立temporary `.npz` shards，通過真實local
training、sample-weighted aggregation與held-out evaluator後清理。Accuracy僅作為execution
evidence，不作為模型品質或正式實驗結果。

### 8.4 明確延後與review gate

- `future-phase handoff`：Root→Branch→Leaf protocol wiring、model-free preparation、
  feature 3、ADRF global-model distribution、topology Notify／PATCH及multi-lower-round
  artifact transport由Slice 5處理。
- `integration verification gap`：real multi-process／multi-NWDAF testbed protocol E2E尚未執行。
- `future-phase handoff`：正式participant partition、repeated runs、statistical analysis與paper
  evaluation不屬於Slice 4。
- `approved deferral`：Image final artifact不進入traffic-specific durable catalog；本slice
  以workspace artifact handoff及offline evaluator完成驗證。

Slice 4 production與test changes已由`PyMTLF/` `e71f1d5`收尾，status／review evidence
已由`nwdaf-docs/` `a56d986`收尾。

---

## 9. Slice 5 審查結果

### 9.1 計畫符合性

| 要求群組 | 狀態 | 直接證據 |
| --- | --- | --- |
| Protocol／legacy authority | 已滿足 | Root-only `hierarchy_contract` selector、ambiguous authority rejection、config及legacy real-process regression |
| Model-free preparation | 已滿足 | Protocol Create不帶model；Leaf與Branch preparation tests及real-process ADRF record count證明此階段不取model或讀local shard |
| Recursive topology execution | 已滿足 | Root two-Branch subtree coordinator test、Branch explicit／NRF hybrid establishment、逐edge Create／feature negotiation及topology-only callback tests |
| Controlled image contract | 已滿足 | `X_IMAGE_CLASSIFICATION`、MNIST／CIFAR-10 interoperability mapping、unknown／local mismatch preparation refusal及first-round bundle mismatch tests |
| Round model distribution | 已滿足 | Root per-round ADRF owner的POST／PUT／GET／DELETE、exact ADRF、allowlist、int63 ID、failure及generation cleanup tests |
| Lower-tier execution | 已滿足 | First lower round沿用Root ADRF reference；替代multi-lower policy的第二輪起使用Branch local artifact；Leaf result及Branch aggregate向上使用producer URL |
| Topology PATCH與failure gates | 已滿足 | Candidate revision fence、disabled-child DELETE、feature mismatch cleanup、retained-result 403 atomicity及real-process PATCH evidence |
| Real-process protocol flow | 已滿足local boundary | 真實Go NWDAF、PyMTLF、NRF、ADRF與MongoDB完成一輪controlled MNIST HFL；同一`storTransId`由Branch與兩個Leaves取得，record最終刪除 |
| Legacy regression | 已滿足 | `smoke/manual-success`真實多程序情境及full test suites通過 |
| Scope boundary | 已滿足 | 未修改`adrf/`、`nrf/`、generated OpenAPI或retained recovery；正式testbed與Branch replacement仍延後 |

### 9.2 發現與修正

| ID | 狀態 | 確認證據 | 修正 | 驗證 |
| --- | --- | --- | --- | --- |
| `S5-R1` | 已關閉 | Pure aggregation Branch在第一輪曾被Leaf local image config gate拒絕 | Branch只驗證上游image task／bundle contract，不要求local shard；Leaf維持local dataset gate | `test_protocol_intermediate_accepts_image_round_without_local_training_config`及real-process run |
| `S5-R2` | 已關閉 | Direct scenario可能在第二個Leaf的Go availability monitor尚未接納PyMTLF時提早送request | Direct checks先對兩個Leaves各跑model-free preparation／cleanup，確認每個public Go boundary已ready | Protocol real-process run |
| `S5-R3` | 已關閉 | Evidence regex曾把transaction ID後的換行文字一併擷取 | Transaction ID capture限制為單一non-whitespace token | `test_protocol_evidence_parses_transaction_ids_at_log_line_boundaries` |
| `S5-R4` | 已關閉 | Protocol preparation的`dataAvReq`曾沿用traffic `USER_DATA_USAGE_TRENDS`，與image task矛盾 | 改用與subscription相同的`X_IMAGE_CLASSIFICATION` `nwdafEvent`，不啟動traffic collection | Protocol request builder test及real-process run |
| `S5-R5` | 已關閉 | Unknown／local-incompatible image contract、first-round bundle mismatch與two-Branch protocol flow缺少直接測試 | 補上preparation refusal、bundle validation及two-Branch Root preparation／round tests | Slice 5 focused PyMTLF matrix |
| `S5-R6` | 已關閉 | 新增Go test code有context、bytes comparison、constant與line-length lint findings | 依現有test style修正，不改production behavior | Focused Go tests與`make lint` |

初始production／test-code review及每項修正的targeted follow-up review均已完成；目前沒有
未關閉的Slice 5 current-slice code finding。

### 9.3 驗證

| Repository／命令 | 結果 |
| --- | --- |
| `NWDAF/` focused package tests | Pass |
| `NWDAF/ make test` | Pass；environment-gated live backend tests維持既有skip行為 |
| `NWDAF/ make lint` | Pass；`0 issues` |
| `NWDAF/ make build` | Pass |
| `PyMTLF/` Slice 5 focused matrix | Pass；283 passed，1個dependency deprecation warning |
| `PyMTLF/ .venv/bin/pytest -q` | Pass；746 passed、2 skipped、55個dependency deprecation warnings |
| `PyMTLF/ .venv/bin/ruff check src tests` | Pass |
| `nwdaf-resources/` evidence parser tests | Pass；27 passed |
| `nwdaf-resources/` changed Python lint與preflight | Pass |
| Protocol real-process scenario | Pass；`/tmp/nwdaf-hierarchical-fl-protocol-nuym_1eg/summary.json` |
| Legacy `smoke/manual-success` scenario | Pass；`/tmp/nwdaf-hierarchical-fl-smoke-il6yom5g/summary.json` |
| `adrf/ go test ./internal/sbi/processor` | Pass；repository未修改 |

### 9.4 剩餘缺口與審查狀態

- `integration verification gap`：正式multi-host testbed尚未執行；目前只有local
  real-process evidence。
- `approved scope boundary`：Main protocol scenario為same-global-round fairness，只有一個
  lower round；`reportAfter(round)>1`以production-owner integration test覆蓋。
- `approved scope boundary`：ADRF allowlist representation已驗證，但目前環境不提供caller
  authentication，因此不宣稱ADRF已強制access control。
- `future-phase handoff`：Branch failure detection、replacement、dynamic re-parenting及
  retained-result recovery不屬於Slice 5。
- `optional hardening`：同一round key的並行double-store防漏；現有Root單一active request
  與sequential upper rounds不會觸發。

Slice 5已分別由`NWDAF/` `256349f`、`PyMTLF/` `554c96d`、
`nwdaf-resources/` `33729a1`及`nwdaf-docs/` `5b23ce4`收尾。正式multi-host
testbed仍為integration verification gap；下一個active work unit為Slice 6 migration
closure。

---

## 10. Slice 6 審查結果

### 10.1 計畫符合性

| 要求群組 | 狀態 | 直接證據 |
| --- | --- | --- |
| Protocol-only authority | 已滿足 | `hierarchy_contract`與legacy Root／Branch／Client branches已移除；hierarchical config直接使用protocol path |
| Legacy artifact removal | 已滿足 | Assignment／preparation-result roles、typed metadata、publisher、reader及special admission已移除；old-role negative tests fail closed |
| Model／result artifact preservation | 已滿足 | `HIERARCHY_AGGREGATE`、round artifacts、sample provenance與flat validation／publication regressions通過 |
| Canonical deployment harness | 已滿足 | Protocol runner接替`run.py`；common runtime helpers收斂到`support.py`；legacy及static-collection entrypoints已移除 |
| Protocol real-process regression | 已滿足local boundary | 真實Go NWDAF、PyMTLF、NRF、ADRF與MongoDB完成protocol hierarchy、ADRF distribution及cleanup |
| Flat／distributed regression | 已滿足 | 既有distributed runner完成training、final validation、ADRF publication與model cutover |
| Go transport boundary | 已滿足 | `go test ./...`、`make lint`與`make build`通過；`NWDAF/`無Slice 6 working-tree change |
| Scope boundary | 已滿足 | Retained-result runtime、Branch replacement與正式testbed未被納入或誤宣稱完成 |

### 10.2 審查發現與處理

| ID | 狀態 | 確認證據 | 處理 | 驗證 |
| --- | --- | --- | --- | --- |
| `S6-R1` | 已關閉 | Canonical protocol runner仍dynamic import舊`run.py`取得port、config及readiness helpers | 將通用helper移至`support.py`，再由protocol runner接替canonical `run.py` | Hierarchy checks、Ruff、preflight及real-process scenario |
| `S6-R2` | 已關閉 | Distributed regression preflight仍要求已被本workstream取代的NWDAF／PyMTLF舊branch名稱 | 只同步manifest的兩個branch prerequisites，不改scenario實作 | Distributed preflight及完整real-process regression |
| `S6-R3` | 已關閉 | Fresh-read conformance檢查發現Root distribution owner prerequisite與candidate preparation夾帶model reference的feature refusal缺少直接測試 | 新增兩個production-entry tests；後者同時確認不下載model或啟動protocol preparation | Targeted tests、focused matrix及full suite |

Production與test-code diff、legacy caller search及targeted fixes均已審查；目前沒有未關閉的
Slice 6 current-slice finding。

### 10.3 最終驗證

| Repository／命令 | 結果 |
| --- | --- |
| `PyMTLF` focused Slice 6 matrix | Pass；259 tests，7個dependency warnings |
| `PyMTLF/.venv/bin/pytest -q` | Pass；636 passed、2 skipped、16個dependency warnings |
| `PyMTLF/.venv/bin/ruff check .` | Pass |
| `nwdaf-resources` hierarchy checks | Pass；10 tests |
| `nwdaf-resources` hierarchy Ruff／preflight | Pass |
| Canonical hierarchy real-process | Pass；`/tmp/nwdaf-hierarchical-fl-protocol-61n8df8p/summary.json` |
| Distributed／flat real-process | Pass；`/tmp/nwdaf-distributed-fl-h3dpak93` |
| `NWDAF/go test ./...` | Pass |
| `NWDAF/make lint` | Pass；`0 issues` |
| `NWDAF/make build` | Pass |
| Changed repositories `git diff --check` | Pass |

### 10.4 Remaining gap與review gate

- `integration verification gap`：正式multi-host external testbed尚未執行；local
  real-process evidence不取代該項驗證。
- `approved deferral`：retained-result persistence／lookup、Branch replacement、fencing與
  runtime topology self-healing維持暫緩。
- `closing commits`：`PyMTLF/` `8a1d6fc`、`nwdaf-resources/` `e0e73c3`與
  `nwdaf-docs/` `021677f`；`NWDAF/`只作回歸驗證，沒有Slice 6變更。

---

## 11. Slice 3 重新界定與計畫狀態

### 11.1 目標調整

2026-09-07確認Branch replacement仍是目前implementation plan需要完成的behavior，但
不採用retained-result recovery。原Slice 3的latest-completed index、lookup、artifact
retention與舊結果接續維持暫緩；Slice 3編號改用於：

- Root偵測單一direct Branch在training round中的availability failure；
- 從該assignment的static candidates選擇replacement並做fresh NRF exact-ID resolve；
- 從topology root與各Branch group解析policy，分別控制Root-to-Branch與
  Branch-to-Leaf的readiness、selection與completion，不保留hard-coded all-required值；
- 以same `mlCorreId`與same Leaf subtree建立新的model-free subscriptions；
- 由Leaf PyMTLF在新resource成功後透過既有notification gateway發送
  `termTrainReq`；成功時由Branch consumer沿既有unsubscribe path發送標準
  DELETE，peer delivery failure或後續DELETE超時時再做provider-side terminal cleanup，
  不另建backend-ID route retirement API；
- Root policy接受時以successful Branch results完成degraded round；拒絕時才丟棄partial
  results。剩餘cohort符合readiness時與replacement並行training，new Branch只加入尚未
  dispatch的下一輪；
- 完成Leaf same-procedure rebind、old-work fencing、ADRF lifecycle與real-process
  process-termination evidence。

詳細owner、failure classification、round semantics、tests與acceptance criteria見
[Slice 3 Detailed Plan](./slices/Slice%203%20Branch%20Replacement%20without%20Retained-result%20Recovery%20Detailed%20Plan.md)。

### 11.2 Review gate

- `plan status`：`S3-R5` remediation、重新驗證與repository-separated commits已完成；
  closing commits見第12.5節。
- `retained boundary`：既有wire fields與unsupported `403` execution gate保留，不建立
  runtime owner。
- `integration verification gap`：正式multi-host testbed尚未執行；未來local
  real-process replacement evidence不能取代該項驗證。

---

## 12. Slice 3 實作審查結果

### 12.1 計畫符合性

| 要求群組 | 狀態 | 直接證據 |
| --- | --- | --- |
| Static topology與assignment | 已滿足 | `branch_groups`分開各區域的Branch candidate pool與單一Leaf set；Root／group policy、strategy及Branch／Leaf edge instructions均由topology config映射至production runtime |
| Root policy與round accounting | 已滿足 | Root依selected cohort執行readiness、selection及completion；accepted degraded round只聚合成功Branches，dispatch `roundInd`與`completedRounds`分開計數 |
| Branch replacement | 已滿足 | 單一direct Branch availability failure啟動background replacement；remaining cohort符合Root policy時繼續training，replacement只加入下一個尚未dispatch的round |
| Candidate lifecycle | 已滿足 | Initial selection與replacement共用priority／eligibility semantics、fresh NRF exact-ID resolve及single-attempt candidate rule；candidate exhaustion依remaining readiness繼續或終止 |
| Leaf rebind與terminal lifecycle | 已滿足 | 新resource建立後atomic supersede舊resource並fence舊work；`termTrainReq`沿既有gateway轉送，Branch consumer使用標準DELETE，明確delivery failure與missing DELETE由terminal cleanup收尾 |
| ADRF與artifact lifecycle | 已滿足 | 每個attempt使用獨立ADRF record；replacement僅進入後續allowlist；terminal record count為零；Branch aggregate與上行result維持temporary `mLFileAddr` |
| Retained-result boundary | 已滿足 | Production replacement未發送或執行retained-result instruction；既有unsupported `403` gate保留 |
| Regression與real-process evidence | 已滿足local boundary | PyMTLF／NWDAF完整驗證、hierarchy smoke／aggregation、Branch replacement及distributed／flat real-process scenarios均通過 |

### 12.2 審查發現與修正

| ID | 狀態 | 確認證據 | 修正 | 驗證 |
| --- | --- | --- | --- | --- |
| `S3-R1` | 已關閉 | Priority selection的enabled explicit candidate可省略`priority`，與plan要求及deterministic candidate order不一致 | Branch與Leaf在priority mode一律要求explicit non-negative priority；random mode仍允許省略 | 新增Branch／Leaf rejection tests與random-mode regression；focused及full PyMTLF tests通過 |
| `S3-R2` | 已關閉 | Go route retirement先只依backend ID取得任意route，再檢查direction／generation；相同backend ID出現在多個route時可能找不到應淘汰的current inbound route | Context lookup在同一lock內同時匹配backend ID、`DirectionInbound`及current generation，processor直接使用該結果 | Context與processor tests覆蓋shared backend ID、stale generation、outbound route及idempotency；Go full tests通過 |
| `S3-R3` | 已關閉 | Leaf收到新parent preparation ACK後，registry membership切換與舊resource supersede之間可被舊parent DELETE插入，使同一procedure誤進terminal cleanup | 在同一Leaf lock內完成新resource activation、membership轉移與舊resource supersede，再於lock外釋放semaphore slots | Event／barrier concurrency regression先重現失敗，再確認舊DELETE無法終止新resource；PyMTLF full tests通過 |
| `S3-R4` | 已關閉 | Root generation在replacement Create進行中重設時，Create response仍可能被舊generation採納並留下新remote resource | Create返回後再次確認active generation；stale／closing時best-effort移除剛建立的participant resource並拒絕adoption | Deterministic in-flight generation-reset test確認replacement resource被清理；focused及full PyMTLF tests通過 |
| `S3-R5` | 已關閉 | Leaf rebind為清除Go inbound route另新增backend-ID retirement endpoint，繞過既有Model Training Notify／DELETE lifecycle並引入只有單一cleanup情境使用的非標準API | 移除專用endpoint；Leaf以`termTrainReq`通知舊Branch，成功時等Branch經既有unsubscribe發送DELETE，明確peer failure或DELETE grace timeout時由provider本地收尾 | PyMTLF rebind success／explicit failure／local Go transport failure及Branch async unsubscribe tests；Go Notify→DELETE／failure／timeout tests；full regressions與real-process replacement通過 |
| `S3-R6` | 已關閉 | Superseded Leaf resource雖已fence，等待terminal delivery期間仍可能保留dataset、local work與artifact references | Rebind transition立即清除transient training state與舊experiment reservation，只保留terminal notification及標準DELETE所需resource metadata | Leaf rebind test直接注入transient state並確認supersede後全部釋放；focused及full PyMTLF tests通過 |

`S3-R1`至`S3-R6`的targeted follow-up review均已完成，目前沒有未關閉的Slice 3
code finding。

### 12.3 Remediation 前驗證基準

| Repository／命令 | 結果 |
| --- | --- |
| `PyMTLF/.venv/bin/pytest -q` | Pass；644 passed、2 skipped、16個dependency warnings |
| `PyMTLF/.venv/bin/ruff check .` | Pass |
| `NWDAF/make test` | Pass |
| `NWDAF/make lint` | Pass；`0 issues` |
| `NWDAF/make build` | Pass |
| `nwdaf-resources` hierarchy support tests | Pass；16 tests |
| `nwdaf-resources` hierarchy Ruff／preflight | Pass |
| Branch replacement real-process | Pass；`/tmp/nwdaf-hierarchical-fl-protocol-wzic9hae/summary.json` |
| Canonical hierarchy smoke | Pass；`/tmp/nwdaf-hierarchical-fl-protocol-pgitlu9e/summary.json` |
| Canonical hierarchy aggregation | Pass；`/tmp/nwdaf-hierarchical-fl-protocol-u6rbxo42/summary.json` |
| Distributed／flat real-process | Pass；`/tmp/nwdaf-distributed-fl-gkzquy6q` |
| Changed repositories `git diff --check`與new topology YAML whitespace check | Pass |

Branch replacement evidence直接呈現四個successful Root aggregations：initial all-active
round、Area A primary失效後的accepted 2／3 degraded round、replacement尚未ready時的
two-Branch round，以及replacement加入後恢復three-Branch round。新舊Branch使用同一
`mlCorreId`及不同`notifCorreId`／Location；兩個Area A Leaf舊inbound routes均已淘汰，
`retainedResultUsed=false`，四個ADRF `storeTransId`彼此不同且terminal record count為零。

上述結果是`S3-R5`修正前的regression baseline，不足以驗收新的
`termTrainReq`／standard DELETE lifecycle；remediation完成後必須重新執行受影響的
focused、full與real-process checks。

### 12.4 Remediation 後驗證

| Repository／命令 | 結果 |
| --- | --- |
| `PyMTLF/.venv/bin/pytest -q` | Pass；647 passed、2 skipped、16個dependency warnings |
| `PyMTLF/.venv/bin/ruff check .` | Pass |
| `go test ./...`（`NWDAF/`） | Pass |
| `golangci-lint run`（`NWDAF/`） | Pass；`0 issues` |
| `make build`（`NWDAF/`） | Pass |
| `nwdaf-resources` hierarchy support tests | Pass；16 tests |
| `nwdaf-resources` hierarchy Ruff／preflight | Pass |
| Branch replacement real-process | Pass；`/tmp/nwdaf-hierarchical-fl-protocol-s8lap18y/summary.json` |
| Canonical hierarchy real-process | Pass；`/tmp/nwdaf-hierarchical-fl-protocol-kqg871u4/summary.json` |
| Distributed／flat real-process | Pass；`/tmp/nwdaf-distributed-fl-cdga2w2r` |
| Changed repositories `git diff --check` | Pass |

Branch replacement重新驗證保留一輪failed-Branch accepted aggregate、一輪two-Branch
degraded continuation及一輪replacement恢復後的three-Branch aggregate；兩個Area A
Leaves均完成舊backend resource與matching route的terminal cleanup，且
`retainedResultUsed=false`。Focused tests另直接覆蓋peer回`204`後的standard DELETE、
明確delivery failure、local Go transport failure bounded retry，以及accepted Notify後未收到
DELETE的grace fallback；明確delivery failure後的late DELETE只消耗一次tombstone，重複
DELETE不會再次清理backend resource。

### 12.5 剩餘缺口與review gate

- `integration verification gap`：正式multi-host external testbed尚未執行；local
  real-process fail-stop、rebind與cleanup evidence不取代實際跨主機網路失敗及timing驗證。
- `approved deferral`：retained-result persistence／lookup／handoff、Leaf replacement
  production transition、simultaneous multi-Branch replacement、Root restart recovery及
  authenticated multi-vendor re-parent不屬於本slice。
- `closing commits`：`NWDAF/` `be3fa57`、`PyMTLF/` `90f1f62`、
  `nwdaf-resources/` `da9b848`、`nwdaf-docs/` `8433b03`。
- `delivery status`：Slice 3已完成repository-separated commits；正式multi-host testbed仍是
  未關閉的external validation，因此整體phase尚未標示為`Completed`。
