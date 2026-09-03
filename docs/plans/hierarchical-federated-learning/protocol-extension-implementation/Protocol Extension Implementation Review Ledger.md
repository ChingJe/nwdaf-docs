# Hierarchical NWDAF FL Protocol Extension Implementation Review Ledger

日期：2026-09-04

狀態：Slice 2 Committed／`PyMTLF` closing commit `0e87ef1`

相關文件：

- [Implementation Plan](./Hierarchical%20NWDAF%20FL%20Protocol%20Extension%20Implementation%20Plan.md)
- [Slice Map](./Protocol%20Extension%20Implementation%20Slice%20Map.md)
- [Slice 1 Detailed Plan](./slices/Slice%201%20Wire%20Contract%20and%20Resource%20Lifecycle%20Foundation%20Detailed%20Plan.md)
- [Slice 2 Detailed Plan](./slices/Slice%202%20Candidate%20Pool%20Policy%20and%20Local%20Contract%20Execution%20Detailed%20Plan.md)

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
- `future-phase handoff`：retained-result index、lookup與outcome producer（Slice 3）；
- `future-phase handoff`：Root／Branch protocol-mode orchestration、feature 3 production
  advertisement、ADRF global-model distribution與sender cleanup（Slice 4）；
- `future-phase handoff`：legacy model-bundle cutover與移除（Slice 5）；
- `integration verification gap`：real NRF、ADRF、MongoDB、multi-NWDAF testbed與
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
- ADRF global-model distribution及real NRF／ADRF／MongoDB／multi-NWDAF E2E；
- HTTP establishment／DELETE outcome、real Notify relay與peer callback E2E；
- retained-result index、lookup、retention lifecycle與recovery runtime。

`PyMTLF/` production與test changes已於`0e87ef1`收尾；`nwdaf-docs/`本次status／
evidence changes由獨立文件commit保存。Slice 2 local procedure與state requirements均具
direct evidence，但不得據此描述成hierarchical protocol E2E。
