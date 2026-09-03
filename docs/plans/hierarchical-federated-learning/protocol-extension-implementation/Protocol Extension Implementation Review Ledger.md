# Hierarchical NWDAF FL Protocol Extension Implementation Review Ledger

日期：2026-09-03

狀態：Slice 1 Committed／implementation、verification 與 production commits 已完成

相關文件：

- [Implementation Plan](./Hierarchical%20NWDAF%20FL%20Protocol%20Extension%20Implementation%20Plan.md)
- [Slice Map](./Protocol%20Extension%20Implementation%20Slice%20Map.md)
- [Slice 1 Detailed Plan](./slices/Slice%201%20Wire%20Contract%20and%20Resource%20Lifecycle%20Foundation%20Detailed%20Plan.md)

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
