# R5 Cross-language Verification And Closure

Date: 2026-07-14

Status: Implemented; V1 repository and V2 cross-process verification complete, full 5GC V3 environment unavailable

Parent plan:

- `Behavioral Parity Remediation Plan.md`

Evidence and decision record:

- `Behavioral Parity Audit.md`
- `R0 Compatibility Oracle Framework.md`

Preceding implementation plans:

- `R1 Analytics Shaping Parity.md`
- `R2 Accuracy Measurement Parity.md`
- `R3 Model Identity Generation And Provision Correctness.md`
- `R4 Scheduler And Subscription Lifecycle.md`

Affected repositories:

1. `PyAnLF/`
2. `NWDAF/`
3. `nwdaf-docs/`

---

## 1. Purpose

R1至R4已分別完成analytics shaping、accuracy measurement、model identity/generation與scheduler
lifecycle修復，也已在各repository執行focused與full verification。R5不新增另一套domain behavior，
而是把分散在不同stage的證據組成完整、可重複、可稽核的closure chain。

本階段要回答下列問題：

1. R0 fixtures是否仍可追溯到搬移前Go implementation與tests
2. PyAnLF是否對相同輸入產生historical-compatible analytics與accuracy結果
3. NWDAF與PyAnLF的HTTP contracts是否能由真實雙程序互通，而不只是各自mock成功
4. PyAnLF accuracy output進入Go後，MTLF收到的metric與traffic scale units是否正確
5. R3 provision與R4 completion在retry、duplicate、stale與update/delete race下是否維持既定語意
6. audit內每個BP finding是否有implementation、test或explicit accepted-risk closure
7. parent Phase 3與Phase 4可以恢復到哪一層完成狀態

R5的核心產物不是新production feature，而是verification evidence、必要的test-only補強、完整code
review與文件狀態收斂。若驗證發現production defect，必須先依本文件的failure triage處理，不能修改
fixture或降低驗證標準來讓測試通過。

---

## 2. Goals

R5完成時應達到：

1. R0四個compatibility slices都有可追溯fixture、expected result與green test
2. PyAnLF full suite、NWDAF full suite、targeted race、lint與build全部有本階段執行紀錄
3. 真實PyAnLF process與Go AnLF callback server至少完成核心runtime lifecycle HTTP flow
4. cross-repository contract matrix逐項標記unit、contract、live-process或environment E2E證據層級
5. 所有BP findings逐項標記為fixed、approved difference或accepted future risk
6. 沒有未處理P0/P1 finding、data semantic regression或未核准的lifecycle leak
7. environment限制、未執行項目與不能主張的驗證層級明確記錄
8. parent plan、audit、R0與R1至R4文件的狀態互相一致

---

## 3. Scope

### 3.1 Included

#### PyAnLF

1. 執行全部Python tests
2. 逐一執行analytics、accuracy、model與reporting compatibility slices
3. 補足R5 traceability需要但目前缺少的test assertions或test helpers
4. 驗證API serialization、HTTP status與callback delivery contract
5. 驗證scheduler、accuracy worker、completion finalizer與shutdown concurrency
6. 執行真實FastAPI/uvicorn process供NWDAF live contract tests使用

#### NWDAF

1. 執行full Go tests、targeted race、lint與build
2. 驗證AnLF API -> processor -> coordinator -> context boundary
3. 驗證analytics callback到external notification path
4. 驗證accuracy callback到MTLF decision input的欄位與單位
5. 驗證model provision event identity、retry與generation contract
6. 驗證runtime completion duplicate、stale、future、missing與update/delete race semantics
7. 擴充既有`backend_live_test.go`，只補目前R5 matrix缺少且能由雙程序穩定驗證的cases

#### Documentation

1. 建立BP finding closure matrix
2. 記錄每個verification command、commit、結果與skip原因
3. 記錄accepted risks沒有被本階段意外改動
4. 更新parent、audit與各stage status

### 3.2 Excluded

R5不包含：

1. 改寫analytics、accuracy、model lifecycle或scheduler演算法
2. 重新校正MTLF thresholds或baseline policy
3. expected-source completeness與timeout-based ground-truth policy
4. accuracy durable outbox或process-restart delivery durability
5. observation delivery per-source workers、durable queue、replay或backfill
6. scheduler completion後自動取消SMF/UPF data subscription
7. 移除Go端保留的`MlModelInfo`、collection refs、traffic或ADRF state
8. PyMTLF、one-shot AnalyticsInfo、future model selection API
9. 修改local free5GC reference trees或3GPP/OpenAPI source artifacts

若review認為上述任一項已成為P0/P1 correctness blocker，必須停止並replan，不能把它偷偷納入R5。

---

## 4. Governing Sources And Frozen Baseline

### 4.1 Required Local Sources

R5依下列順序判定語意與驗證要求：

1. `Behavioral Parity Audit.md`與其confirmed decisions
2. `R0 Compatibility Oracle Framework.md`與language-neutral fixtures
3. 搬移前NWDAF source與tests
4. current `PyAnLF/`與`NWDAF/` source/tests
5. `Behavioral Parity Remediation Plan.md`及R1至R4 implementation records
6. `nwdaf-docs/docs/development_policy.md`
7. `free5gc-dev-skill/SKILL.md`
8. `free5gc-dev-skill/references/testing.md`
9. `free5gc-dev-skill/references/concurrency-lifecycle.md`
10. `free5gc-dev-skill/references/review-checklist.md`
11. local 3GPP TS與OpenAPI YAML，只用於確認standard-facing meaning
12. local free5GC NF references，只用於Go callback edge、lifecycle與test shape

PyAnLF不是free5GC NF alignment target。其domain expected behavior以historical NWDAF與本輪confirmed
decisions為準；Go HTTP edge則維持current NWDAF的API/processor/coordinator/context boundary。

### 4.2 Planning Baseline

撰寫本文件時的clean repository baseline為：

| Repository | Branch | HEAD |
| --- | --- | --- |
| `NWDAF/` | `refactor/free5gc-alignment` | `68717f8 feat(anlf): handle backend runtime completion` |
| `PyAnLF/` | `master` | `bfd611c feat(runtime): complete subscription reporting lifecycle` |
| `nwdaf-docs/` | `main` | `cf91bb3 docs: record R4 scheduler lifecycle completion` |

R5執行前需重新記錄實際HEAD。若HEAD已前進，必須確認新增commits是否改動R1至R4相關路徑；若有，
將其納入review與verification matrix，不能沿用本表直接宣稱已驗證。

### 4.3 Historical Oracle Pins

| Area | Historical source/test baseline |
| --- | --- |
| Analytics shaping | `NWDAF@0db9584:internal/anlf/analytics.go`與`analytics_test.go` |
| Accuracy workflow | `NWDAF@a7d0693:internal/anlf/accuracy/`與其tests |
| Accuracy state | `NWDAF@a7d0693:internal/context/accuracy_store.go`與tests |
| Reporting lifecycle | `NWDAF@0db9584`與`NWDAF@62e2f9f` notifier/processor tests |
| Current Go boundary | `NWDAF@64fdf0a`與current tests |
| Pre-remediation PyAnLF | `PyAnLF@60994c3` |

Historical production code只作oracle，不得恢復成Go端第二條runtime implementation。

---

## 5. Confirmed Verification Policy

下列原則已由parent plan與前置決策確定，R5不重新選擇：

1. 以最大程度保留搬移前Go behavior為compatibility原則
2. `immRep=false`先等待一個`repPeriod`是approved behavior change
3. confidence-zero prediction不進accuracy baseline
4. accuracy callback維持finite retry後drop，不新增outbox
5. completion callback使用process-local tombstone retry至`204`或shutdown
6. scheduler natural completion只讓NWDAF runtime inactive，不刪除analytics subscription
7. R4不取消SMF/UPF collection，這是explicit lifecycle boundary
8. single global observation delivery worker與HOL risk本輪不修
9. source send failure、queue overflow、restart durability維持documented accepted risk
10. PyAnLF是`min_matched_predictions`唯一sample-count report gate；Go不重複拒絕

R5不得把approved difference寫成parity failure，也不得把accepted risk寫成已修復。

---

## 6. Verification Levels And Claim Rules

### 6.1 Level V1: Deterministic Unit And Golden Verification

V1在單一repository、單一process或pure helper boundary執行，包含：

1. historical fixtures
2. unit/API tests
3. fake sender、fake clock與in-memory HTTP tests
4. Go handler/processor/coordinator/context tests

V1可證明演算法、state transition與serialization，但不能聲稱真實Go/Python process互通。

### 6.2 Level V2: Cross-process HTTP Contract Verification

V2必須同時使用：

1. 真實執行中的PyAnLF FastAPI/uvicorn process
2. 真實Go AnLF callback HTTP server
3. 實際TCP listener與JSON serialization

允許external analytics consumer使用`httptest.Server`，因為其目的只是驗證Go notifier輸出；不得以
mock PyAnLF client或in-process FastAPI client取代真實PyAnLF process並聲稱V2通過。

### 6.3 Level V3: Full Environment End-to-end Verification

V3需要可運作的5GC與外部依賴，至少包含相關的：

1. NRF/NWDAF/SMF/UPF
2. MongoDB與必要network setup
3. UE traffic source或等價integration driver
4. MTLF callback path
5. ADRF與Daisy，若要驗證retrain flow

V3才可證明真實SMF/UPF periodic collection、multi-source arrival、ADRF data retrieval與Daisy retrain
completion。V1/V2不能取代V3。

### 6.4 Status Claim Rules

R5使用分層狀態：

1. `repository verification complete`：V1全部通過且code review無未處理P0/P1
2. `cross-process contract verified`：必要V2 cases全部通過
3. `environment E2E verified`：必要V3 scenarios全部通過
4. `environment E2E unverified`：環境不存在或前置條件不完整，需明確列出

若V1與V2通過但V3無法執行，文件可關閉repository與contract工作，但不得把parent狀態寫成完整
environment-verified。若V2因本地依賴缺失無法執行，R5不得標記cross-process complete。

### 6.5 Confirmed Current Environment Boundary

Confirmed environment constraint（2026-07-14）：目前工作區沒有完整5GC環境，因此本輪R5不執行
V3，也不把V3列為V1/V2 closure的blocker。

本輪實際目標固定為：

1. 完成V1 deterministic repository verification
2. 完成V2真實Go/PyAnLF cross-process HTTP contract verification
3. 將第13節V3 scenarios保留為未來environment verification checklist
4. final status明確保留`full 5GC environment E2E unverified`

不得使用mocked SMF/UPF、synthetic local driver或V2 live contract結果替代V3並宣稱完整5GC E2E通過。

---

## 7. R0 Oracle Closure Matrix

### 7.1 Analytics Slice

Fixtures：

- `PyAnLF/tests/fixtures/behavioral_parity/analytics_cases.json`

Tests：

- `PyAnLF/tests/test_analytics_parity.py`

最低驗證cases：

1. clean single-source sequence
2. missing middle slot zero-padding
3. same-session duplicate last-wins
4. same metadata from different sources不互相覆寫
5. multi-IP aggregation
6. different-anchor global alignment
7. mean contributing-center timestamp
8. positive/negative half-slot Go-compatible rounding
9. input-window cap

每個expected output需保留historical source/test provenance。若current refactor只改資料結構，expected
數值仍不得由current PyAnLF output反向產生。

### 7.2 Accuracy Metric Slice

Fixtures：

- `accuracy_metric_cases.json`

Tests：

- `test_accuracy_parity.py`

最低驗證cases：

1. UL/DL分離計算
2. MAE、MSE、sMAPE、WAPE與NRMSE historical formulas
3. zero actual traffic behavior
4. actual/predicted traffic scale使用UL/DL samples mean
5. confidence-zero prediction不進baseline
6. `min_matched_predictions` gate

### 7.3 Accuracy Matching And Scope Slice

Fixtures：

- `accuracy_matching_cases.json`
- `accuracy_scope_cases.json`

最低驗證cases：

1. exact target slot match
2. adjacent slot rejection
3. periodic best-effort multi-source aggregation
4. staggered source arrival不在ingest時提前消耗prediction
5. per-source/IP duplicate last-wins
6. UE Communication event selection不受event ordering影響
7. SUPI/group list trim、sort與dedup
8. unresolved/`AnyUe`與tracked-resource fallback既定語意

### 7.4 Model And Reporting Slice

Fixtures：

- `reporting_lifecycle_cases.json`

Model tests：

- `test_model_manager.py`
- `test_provision_events.py`
- `test_runtime_manager.py`

Reporting tests：

- `test_reporting.py`
- `test_runtime_completion.py`

最低驗證cases：

1. identity + generation registry key
2. same URL new generation reload
3. duplicate provision event回傳stable response且不重套
4. candidate load failure保留舊runtime/model
5. max-before-monDur precedence
6. `maxReportNbr=0` unlimited
7. `immRep` approved behavior
8. transient report generation/send failure後繼續下一period
9. natural completion callback stable payload與retry
10. explicit delete/update/shutdown不誤發natural completion

---

## 8. BP Finding Closure Matrix

R5需在implementation record逐項填入實際test name與結果。最低分類如下：

| Findings | Expected closure | Primary evidence |
| --- | --- | --- |
| BP-01至BP-06 | fixed by R1 | analytics fixtures與`test_analytics_parity.py` |
| BP-07至BP-18 | fixed/approved by R2 | accuracy metric、matching、scope與reporting tests |
| BP-19至BP-23 | fixed by R3 | model manager、provision event與runtime manager tests |
| BP-24至BP-25 | fixed by R4 | reporting、completion、Go lifecycle與live contract tests |
| BP-26 | approved difference | `immRep=false` fixture/test與decision record |
| BP-27 | accepted future risk | observation worker/HOL documentation保持不變 |
| BP-28至BP-31 | fixed/approved by R2 | prediction bookkeeping、confidence、config與retention tests |

每一列還需確認：

1. production path仍使用被測實作，不是dead helper
2. test assertion驗證behavior，不只驗證沒有exception
3. expected value可追溯historical oracle或confirmed decision
4. accepted risk具有目前影響、觀測方式與future boundary

---

## 9. Cross-repository Contract Matrix

### 9.1 Runtime Apply

驗證：

1. Go request包含subscription context
2. `report_callback_uri`與`runtime_completion_callback_uri`皆存在且可達
3. PyAnLF回傳positive runtime revision與collection requirements
4. Go只有在binding resync完成後啟用runtime
5. request/response field names與required semantics在雙方一致

證據：Go client contract tests、PyAnLF lifecycle API tests與V2 live apply。

### 9.2 Observation To Analytics Notification

V2流程：

1. Go建立runtime
2. Go同步observation binding
3. Go經HTTP送出source observation batch
4. PyAnLF完成shaping/inference/fallback report
5. PyAnLF callback Go AnLF server
6. Go notifier轉送external analytics consumer

至少assert subscription ID、notification correlation ID、runtime revision與report sequence相關穩定欄位。
模型數值不在V2使用隨機artifact做exact assertion；精確shaping數值由V1 golden fixtures負責。

### 9.3 Accuracy Report To MTLF

驗證：

1. PyAnLF contract保留UL/DL-derived historical metrics
2. `actual_traffic_scale`與`predicted_traffic_scale`單位為sample mean
3. Go handler/processor不重算metrics
4. Go不以第二個`minSamples`重複拒絕
5. MTLF收到primary metric、sample count、traffic scales、model identity/generation與scope
6. threshold/baseline input沒有因JSON conversion改變數值

這一項至少需要language-neutral accuracy fixture經PyAnLF serialization後，進入Go handler/processor/MTLF
test seam。若未由真實PyAnLF process送出，必須標為cross-repo fixture contract，不得標為V2 live flow。

### 9.4 Model Provision

驗證：

1. Go對同一logical retry重用stable `event_id`
2. PyAnLF duplicate event不重增generation
3. same identity + same URL + new event建立新generation並重新prepare artifact
4. different identity + same URL不共用錯誤runtime identity
5. candidate failure不替換任何affected runtime
6. Go callback、processor與coordinator維持目前package boundary

V2至少保留duplicate no-match contract；涉及artifact load與atomic runtime swap的精確結果由deterministic
PyAnLF tests提供，除非R5建立穩定的local artifact live fixture。

### 9.5 Runtime Completion

驗證：

1. `maxReportNbr` normal completion送出stable completion event
2. `monDur` normal completion送出正確reason
3. callback暫時失敗後使用同一event ID與payload retry
4. Go valid/duplicate/stale/missing回`204`
5. Go future revision回`409`
6. invalid body/path mismatch回`400`
7. current active revision轉inactive
8. stale completion不關閉new revision
9. explicit delete、replacement與shutdown不發natural completion
10. completion後completed-only source不繼續送backend observation

V2核心case至少包含apply、analytics callback、`maxReportNbr` completion與Go inactive transition。
`monDur`、retry與race若live test不穩定，使用deterministic unit/HTTP tests並清楚標示層級。

---

## 10. Planned Test Changes

R5開始時先做coverage inventory。只有matrix無現有證據時才新增test，不為了stage名稱重寫已存在tests。

### 10.1 PyAnLF Candidate Changes

可能修改：

1. `tests/test_analytics_parity.py`
2. `tests/test_accuracy_parity.py`
3. `tests/test_provision_events.py`
4. `tests/test_runtime_manager.py`
5. `tests/test_reporting.py`
6. `tests/test_runtime_completion.py`
7. `tests/fixtures/behavioral_parity/README.md`

允許的改動只有：

1. 補缺少case或assertion
2. 補fixture provenance
3. 補stable test helper
4. 修正R5驗證發現的production defect及其regression test

不允許僅為current implementation調整historical expected values。

### 10.2 NWDAF Candidate Changes

可能修改：

1. `internal/anlf/client/backend_live_test.go`
2. `internal/anlf/client/backend_test.go`
3. `internal/anlf/client/model_provision_test.go`
4. `internal/anlf/api_model_accuracy_report_test.go`
5. `internal/anlf/api_runtime_completion_test.go`
6. `internal/anlf/processor/analytics_report_test.go`
7. `internal/anlf/processor/runtime_completion_test.go`
8. `internal/anlf/coordinator/model_provision_test.go`
9. `internal/anlf/coordinator/runtime_completion_test.go`
10. `internal/mtlf/model_accuracy_report_test.go`

新增test seam前先使用現有client、API/processor/coordinator interfaces與`httptest`模式，不新增production
exported API只為方便測試。

### 10.3 Production Fix Boundary

若R5發現production defect：

1. 先新增或確認failing regression test
2. 追溯historical oracle或confirmed decision
3. 判定是否仍在既定R1至R4scope
4. 在正確repository做最小完整修正
5. 重新執行該slice、full suite與受影響cross-repo cases
6. 文件記錄failure、root cause、fix commit與verification

若修正會改API、architecture ownership、MTLF policy或accepted risk，必須先replan並取得決策。

---

## 11. Execution Sequence

### Step 1: Preflight And Baseline Freeze

1. 確認三個repository worktree與branch
2. 記錄實際HEAD、Go/Python/tool versions
3. 確認R0 fixtures與R1至R4 implementation commits存在
4. 確認本地artifact與PyAnLF dependencies可用
5. 確認V2所需ports與callback address可用
6. 記錄目前沒有完整5GC V3 environment的confirmed constraint

任一repo有非本階段未commit changes時，先判定是否衝突，不得覆寫。

### Step 2: Build Traceability Inventory

1. 將BP-01至BP-31對應到fixture/test/accepted risk
2. 將R1至R4 completion criteria對應到current test name
3. 標出只有implementation self-consistency、沒有historical oracle的case
4. 標出只有unit evidence、沒有HTTP evidence的contract
5. 建立缺口清單後才決定要新增哪些tests

### Step 3: Close Deterministic Test Gaps

依Analytics、Accuracy、Model、Reporting順序補測，避免同時跨四個domain修改。每個slice：

1. 先執行focused tests建立green/failing baseline
2. 補failing evidence或缺少assertion
3. 必要時修正production defect
4. 重跑focused tests
5. 更新traceability matrix

### Step 4: Run Full PyAnLF Verification

執行fixture slices、all tests與project正式提供的quality tools。若`pyproject.toml`沒有formatter、lint或type
checker entrypoint，不臨時以不同工具替代，也不將其列為通過。

### Step 5: Run Full NWDAF Verification

先focused AnLF/MTLF tests，再執行full tests、targeted race、lint與build。任何full race限制必須列出
失敗package與targeted evidence，不可靜默省略。

### Step 6: Run V2 Cross-process HTTP Verification

1. 啟動真實PyAnLF service
2. 等待health/readiness可接受連線
3. 設定`PYANLF_LIVE_ENDPOINT`
4. 執行NWDAF live contract tests
5. 收集兩邊必要logs
6. 正常停止PyAnLF並確認沒有殘留worker/process
7. 對每個matrix item標記V2 pass、V1-only或blocked

V2不能依賴未提交的手動source改動或未知local state。若需要test config，應提交明確、無secret且只供
test使用的config或documented invocation。

### Step 7: Record V3 As Environment-unverified

目前沒有完整5GC環境，因此本輪不執行第13節scenarios。Implementation record需：

1. 明確標記V3未執行
2. 記錄缺少完整5GC environment，而不是將V1/V2 failure或skip混入同一分類
3. 保留第13節scenario matrix供未來具備環境時使用
4. 不以V1/V2結果填寫任何V3 pass

### Step 8: Final Code Review

Review範圍為R1至R4實際commits與R5 test/fix diff，不只review最後一個commit。 findings依severity列出，
P0/P1未關閉時不得進入status restoration。

### Step 9: Documentation And Status Closure

1. 更新本文件implementation record
2. 更新R0 oracle status
3. 更新R1至R4 residual risk與verification status
4. 更新Behavioral Parity Audit closure matrix
5. 更新parent remediation plan status
6. 更新AnLF Backend Transition parent status
7. 明確保留future work references

---

## 12. Verification Commands

依workspace規則，實際script/test/service execution需使用elevated permissions。

### 12.1 PyAnLF Focused

```bash
uv run pytest -q tests/test_analytics_parity.py
uv run pytest -q tests/test_accuracy_parity.py
uv run pytest -q tests/test_model_manager.py tests/test_provision_events.py tests/test_runtime_manager.py
uv run pytest -q tests/test_reporting.py tests/test_runtime_completion.py
```

### 12.2 PyAnLF Full

```bash
uv run pytest -q
```

目前`pyproject.toml`沒有正式formatter、lint或type-checker entrypoint，因此本計畫不臨時指定替代工具。
若R5開始前project已新增正式entrypoint，需一併執行並記錄。

### 12.3 NWDAF Focused

```bash
go test ./internal/anlf/...
go test ./internal/mtlf/...
go test ./internal/sbi/processor/...
go test -race ./internal/anlf/... ./internal/sbi/processor/... ./internal/mtlf/...
```

### 12.4 NWDAF Full

```bash
make test
make lint
make build
```

### 12.5 V2 Cross-process

PyAnLF terminal：

```bash
uv run python run.py
```

NWDAF terminal：

```bash
PYANLF_LIVE_ENDPOINT=http://127.0.0.1:9090 go test ./internal/anlf/client -run '^TestLivePyAnLF' -count=1 -v
```

不得平行執行會共用global NWDAF context或固定PyAnLF runtime IDs的live cases，除非test已明確做到
isolation。

### 12.6 Repository Hygiene

三個repository分別執行：

```bash
git status --short --branch
git diff --check
```

文件修改不要求執行implementation tests；R5 implementation與closure時才執行本節完整matrix。

---

## 13. Environment-level Scenario Matrix

本節不屬於目前R5 execution scope；它保留未來取得完整5GC環境後所需的V3驗證項目。目前所有
scenarios一律維持`environment E2E unverified`。

| Scenario | Required evidence | Pass condition |
| --- | --- | --- |
| SMF/UPF periodic collection | NWDAF、SMF/UPF logs與observations | sampling interval內持續收到正確source data |
| Multi-source subscription | 至少兩個source/correlation | shaped input與report不跨source覆寫，global slot聚合正確 |
| Analytics notification | external consumer payload | subscription/correlation與UE Communication report正確 |
| Accuracy round | PyAnLF monitor與Go MTLF logs | periodic match後units與sample count正確進入MTLF |
| Model provision | MTLF -> Go -> PyAnLF flow | stable event ID、identity/generation與runtime swap正確 |
| Daisy retrain completion | Daisy callback與redirect | provision event進入同一model provision path |
| ADRF retrieval | retrieval session/data logs | retrain input可取得且completion state收斂 |
| `maxReportNbr` | completion callback與Go state | report上限後runtime inactive，subscription未刪除 |
| `monDur` | completion callback與Go state | duration到期後runtime inactive，延遲不超既定tick語意 |

V3結果需註明使用的config、NF endpoints、model artifact與traffic scenario；不得只寫「E2E passed」。

---

## 14. Final Review Checklist

### 14.1 Analytics And Accuracy

1. source identity在flatten、dedup與aggregation過程沒有遺失
2. internal gaps仍zero-pad
3. global origin、mean timestamp與Go rounding一致
4. UL/DL沒有重新壓成single scalar
5. adjacent slot不進ground truth
6. periodic best-effort不因第一個source到達就consume prediction
7. metric formulas、zero-traffic與traffic scale units一致
8. confidence-zero與sample-count gate維持confirmed semantics
9. scope canonicalization不受event/list ordering影響

### 14.2 Model Lifecycle

1. runtime key使用identity + generation
2. cache key不只依URL basename
3. same URL new generation確實reload
4. duplicate event idempotent
5. candidate load/commit failure保留old model
6. concurrent provision不產生partial runtime replacement

### 14.3 Scheduler And Completion

1. max/monDur precedence與attempt count一致
2. transient error不永久停止scheduler
3. natural completion不在scheduler thread執行recursive release
4. completion tombstone payload/event ID在retry間穩定
5. shutdown可停止所有workers且network waits有timeout
6. stale completion不關閉new runtime revision
7. explicit release/update/shutdown不誤發completion
8. completed-only source不繼續造成backend 404/retry

### 14.4 Go Boundary

1. API只負責HTTP parse/validation/response
2. processor負責procedure orchestration
3. coordinator負責cross-domain workflow
4. context transition是revision-aware atomic operation
5. outbound backend calls仍在AnLF client
6. external analytics notification仍由notifier處理
7. MTLF沒有重新擁有PyAnLF accuracy measurement

### 14.5 Accepted Risks

1. accuracy finite retry後drop仍有error log/counter
2. completion tombstone不跨process restart持久化
3. single observation worker仍可能HOL blocking
4. queue overflow/source send failure沒有replay/backfill
5. natural completion不取消SMF/UPF collection
6. Go保留local subscription、traffic、ADRF與model metadata

---

## 15. Failure Triage And Replan Rules

### 15.1 Classification

任何failure先分類：

1. `implementation defect`：current code違反historical invariant或confirmed decision
2. `fixture defect`：expected/provenance抄錄錯誤，且可由historical source證明
3. `test defect`：race、clock、port或global state造成不穩定，但production semantics未錯
4. `contract defect`：Go與PyAnLF schema、status或lifecycle interpretation不一致
5. `environment blocker`：必要NF、network、artifact或external service不可用
6. `approved difference`：已明確決策不恢復historical behavior
7. `accepted future risk`：已明確排除且不阻擋本輪correctness

### 15.2 Stop Conditions

發生下列情況必須停止並向使用者提出決策：

1. historical Go tests與confirmed plan互相衝突
2. 修正需要改Go/PyAnLF API contract
3. 修正需要移動domain ownership
4. 修正需要改MTLF thresholds或baseline policy
5. accepted risk實際成為P0/P1資料錯誤或resource leak
6. V2需要新增production-only test seam或不安全debug endpoint
7. 計畫要求的dependency/environment不存在，且只能用較弱替代驗證

### 15.3 Fixture Change Rule

修改golden expected前必須在commit或implementation record附上：

1. historical file與commit
2. 對應舊test或演算法推導
3. 為何原fixture錯誤
4. 為何不是current output反向定義expected

---

## 16. Evidence Record Format

R5 implementation record對每個command至少記錄：

| Field | Required content |
| --- | --- |
| Repository | repo name與branch |
| Commit | tested HEAD |
| Command | exact command |
| Level | V1、V2或V3 |
| Result | pass/fail/skip與test count |
| Environment | relevant endpoint/config/dependency |
| Gap | skipped或partial原因 |

不需要提交完整verbose logs；保留足以重現與判斷結果的summary。若failure是本輪修復依據，需保存
關鍵error與failing test name。

---

## 17. Commit Strategy

R5不以單一跨repo commit處理：

1. `PyAnLF/`：fixture/test補強與必要production fix各自形成語意清楚的commit
2. `NWDAF/`：contract/live tests與必要production fix各自形成語意清楚的commit
3. `nwdaf-docs/`：progress、verification evidence與status closure獨立commit

若inventory確認現有tests已完整，不要求為了R5建立空的implementation commit。不得把production fix、
unrelated refactor與文件closure混在同一repository commit。

---

## 18. Completion Criteria

### 18.1 Repository-level Closure

需全部滿足：

1. R0四個fixture slices可追溯且green
2. PyAnLF full suite通過
3. NWDAF full tests、targeted race、lint與build通過
4. BP-01至BP-31都有fixed、approved difference或accepted-risk closure
5. accuracy到MTLF的units與gate有cross-repo contract evidence
6. code review沒有未處理P0/P1 finding
7. 三個repository hygiene checks通過

### 18.2 Cross-process Closure

需全部滿足：

1. 真實PyAnLF process與Go callback server可互通
2. runtime apply、binding、observation與analytics callback成功
3. `maxReportNbr` completion使Go current runtime inactive
4. duplicate/stale/retry semantics至少由V1 HTTP tests補足
5. explicit release/update不誤發completion
6. live process可正常shutdown且沒有殘留worker造成測試hang

### 18.3 Environment-level Closure

目前工作區沒有完整5GC環境，因此本輪不達成environment-level closure。狀態必須保留：

```text
Repository and cross-process verification complete;
full 5GC environment E2E unverified.
```

### 18.4 Parent Status Restoration

1. V1完成後可移除「behavioral parity implementation incomplete」描述
2. V2完成後可標記R5 repository/cross-process complete
3. V3未完成時，parent Phase 3/4只能標記repository-level behavioral parity verified
4. V3完成後，才可加入full environment E2E verified聲明
5. future risks與non-goals不因status restoration消失

---

## 19. Implementation Record

### 19.1 Actual Baseline And Environment

R5於2026-07-14以下列source開始：

1. `PyAnLF@bfd611c feat(runtime): complete subscription reporting lifecycle`
2. `NWDAF@68717f8 feat(anlf): handle backend runtime completion`
3. `nwdaf-docs@bd2f30d docs: plan AnLF verification closure`
4. Python `3.12.3`
5. `uv 0.8.8`
6. Go `1.25.5`
7. `golangci-lint 2.8.0`

R5 test-only變更已commit為：

1. `PyAnLF@2ab89fd test(accuracy): verify cross-language report contract`
2. `NWDAF@c48d1bb test(anlf): verify backend cross-process contracts`

驗證期間`127.0.0.1:9090`由既有Prometheus process使用，因此V2以相同`config/config.yaml`application
config在`127.0.0.1:19090`啟動真實PyAnLF process，沒有修改production config或使用mock backend。

### 19.2 Coverage Inventory And Added Evidence

Inventory確認R1至R4大部分deterministic evidence已存在，R5只補下列缺口：

1. PyAnLF `test_report_serialization_matches_go_contract`固定accuracy callback的JSON field names、timestamp、
   model identity/generation、scope、metrics、traffic scales與retrain context
2. NWDAF `TestPyAnLFAccuracyJSONReachesMtlfWithoutUnitConversion`以同一canonical JSON經真實AnLF HTTP
   listener、API binding、processor與MTLF workflow，確認數值與units不變
3. 上述Go test使用`sample_count=1`且`minBufferSamples=8`，證明Go不重複套用AnLF matched-sample gate，
   report仍進入MTLF policy observation
4. `TestLivePyAnLFContract`新增backend report ID、sequence與runtime revision assertions
5. 新增`TestLivePyAnLFExplicitReplacementAndReleaseDoNotComplete`，驗證real process replacement與explicit
   release不產生natural completion callback

R5沒有修改production source、API contract、domain ownership、MTLF threshold或accepted-risk boundary。

### 19.3 BP-01 To BP-31 Closure

| Finding | Closure | Primary current evidence |
| --- | --- | --- |
| BP-01 | fixed | `analytics-missing-middle-slot` golden fixture |
| BP-02 | fixed | `analytics-same-metadata-different-source`與source-order test |
| BP-03 | fixed | clean/anchor-drift/different-anchor analytics fixtures |
| BP-04 | fixed | `analytics-mean-output-timestamp` fixture |
| BP-05 | fixed | positive/negative half-slot fixtures與`test_go_round_uses_half_away_from_zero` |
| BP-06 | fixed | 11-case `test_historical_analytics_shaping` matrix |
| BP-07 | fixed | adjacent-slot與positive/negative half accuracy matching fixtures |
| BP-08 | fixed to confirmed periodic best-effort | source-aware matching fixture與periodic monitor tests |
| BP-09 | fixed | `BP-09-directional-error` UL/DL fixture |
| BP-10 | fixed | zero-actual與historical metric formula fixtures |
| BP-11 | fixed | `BP-11-multi-pair-scale` fixture與R5 Go policy-unit test |
| BP-12 | fixed | per-model periodic worker/check tests |
| BP-13 | fixed | `test_matched_samples_below_minimum_do_not_accumulate_across_rounds` |
| BP-14 | approved replacement | confidence readiness tests；fixed-duration warmup config已移除 |
| BP-15 | approved finite retry/drop | delivery retry tests與independent-next-round test |
| BP-16 | fixed | multi-step prediction bookkeeping、pending commit與context tests |
| BP-17 | fixed | scope golden、event selection、sort/dedup與binding fallback tests |
| BP-18 | fixed | accuracy metric/matching/scope golden suites |
| BP-19 | fixed | different-identity same-reference runtime test |
| BP-20 | fixed | same-URL new-generation model manager/runtime tests |
| BP-21 | fixed | full-reference hash與identity/generation cache-key tests |
| BP-22 | fixed | stale snapshot、candidate failure與concurrent provision commit tests |
| BP-23 | fixed | sequential/concurrent duplicate event registry與HTTP replay tests |
| BP-24 | fixed | scheduler completion、tombstone、Go revision semantics與V2 completion flow |
| BP-25 | fixed | generation/send failure recovery與next-tick tests |
| BP-26 | approved difference | `immRep=false` delayed-first-report scheduler behavior |
| BP-27 | accepted future risk | single global Go observation worker保持不變 |
| BP-28 | fixed | each-output-step prediction record與multi-step runtime tests |
| BP-29 | approved behavior | confidence-zero observable but excluded from pending/baseline |
| BP-30 | fixed | PyAnLF-owned accuracy config validation與Go duplicate config removal tests |
| BP-31 | fixed | per-source/IP-session count-bounded ring-buffer test |

所有fixed項目的current tests皆直接使用production implementation；golden expected仍保留historical commit與
source provenance。BP-26、BP-29是confirmed behavior，BP-27維持accepted future risk，沒有偽裝成historical
parity或本輪修復。

### 19.4 Verification Results

PyAnLF V1：

1. focused accuracy slice：`35 passed`
2. full `uv run pytest -q`：`130 passed`
3. `pyproject.toml`沒有正式formatter、lint或type-checker entrypoint，因此沒有以臨時工具取代

NWDAF V1：

1. focused AnLF/MTLF/SBI tests：pass
2. final `make test`：pass；未設定live endpoint時三個`TestLivePyAnLF*`按設計skip，另由V2獨立執行
3. `go test -race ./internal/anlf/coordinator ./internal/anlf/client ./internal/mtlf`：pass
4. `make lint`：`0 issues`
5. `make build`：pass

V2 real-process contract：

```bash
PYTHONPATH=src uv run python -c '<load config/config.yaml and run SBIServer on 127.0.0.1:19090>'
PYANLF_LIVE_ENDPOINT=http://127.0.0.1:19090 \
  go test ./internal/anlf/client -run '^TestLivePyAnLF' -count=1 -v
```

結果為三個cases全部通過：

1. runtime apply、binding sync、observation ingest、analytics callback與external notification
2. `maxReportNbr=1` natural completion使Go current runtime inactive
3. replacement與explicit release不發natural completion
4. duplicate model provision no-match回傳stable response
5. PyAnLF在測試後正常shutdown，沒有殘留test process

Accuracy report到MTLF的精確數值證據屬V1 cross-repository fixture contract，不宣稱是V2 live accuracy
workflow；same-URL artifact swap、stale/retry與`monDur`精確語意也由deterministic V1 tests負責。

### 19.5 Final Review

Review範圍包含`PyAnLF@60994c3..bfd611c`、`NWDAF@64fdf0a..68717f8`與R5 test diff。結論：

1. 沒有未處理P0/P1 finding
2. alignment、accuracy、model lifecycle與completion tests呼叫的都是current production path，不是dead helper
3. Go inbound callback維持API parse/validation、processor dispatch、coordinator/MTLF workflow分層
4. external analytics delivery仍由`internal/sbi/notifier`負責
5. runtime completion compare-and-transition維持revision-aware atomic operation
6. R5沒有把PyAnLF-owned measurement config或accuracy bookkeeping搬回Go

### 19.6 Environment Boundary And Remaining Risks

目前沒有完整5GC、SMF/UPF、Daisy與ADRF整合環境，因此V3未執行且所有第13節scenarios維持
`environment E2E unverified`。V1/V2結果沒有被拿來替代V3。

保留的confirmed risks：

1. accuracy callback有限retry後drop
2. runtime completion tombstone不跨PyAnLF process restart持久化
3. Go observation delivery仍是single worker，可能head-of-line blocking
4. queue overflow/source send failure沒有replay或backfill
5. natural completion不取消SMF/UPF collection，也不刪除Go subscription
6. Go仍保留local subscription、traffic、ADRF與必要model metadata

### 19.7 Final Status

R5已達成repository-level與cross-process closure。正式狀態為：

```text
Repository and cross-process verification complete;
full 5GC environment E2E unverified.
```

Phase 3與Phase 4可恢復為repository-level behavioral parity verified；只有取得完整V3證據後，才能加入
full environment E2E verified聲明。
