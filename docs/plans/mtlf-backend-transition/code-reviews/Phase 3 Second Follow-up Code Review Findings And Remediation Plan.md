# Phase 3 Second Follow-up Code Review Findings And Remediation Plan

Date: 2026-07-22

Status: Historical review record; current gate is tracked in `Phase 3 Review Ledger.md`

Parent documents:

- `../MTLF Backend Transition Plan.md`
- `../Phase 3 Analytics Subscription Routing.md`
- `Phase 3 Code Review Findings And Remediation Plan.md`
- `Phase 3 Follow-up Code Review Findings And Remediation Plan.md`

Canonical current ledger:

- `Phase 3 Review Ledger.md`

This document preserves the second follow-up findings and remediation history.
Its individual acceptance gate is superseded by the canonical ledger.

Reviewed implementation repositories:

- `NWDAF/`
- `PyAnLF/`
- `PyMTLF/`

---

## 1. Purpose And Overall Conclusion

本文件記錄第二次Phase 3 follow-up review的完整發現與修正方案。前一份follow-up文件的
implementation record保存了當時完成的修改與測試結果，但本次重新檢查production behavior後，不能再把
該批修正視為已關閉。

目前結論是：主要happy path及既有unit/module tests仍可通過，但Phase 3尚未達到主計畫與前一份remediation
文件所定義的完成條件。問題分成四類：

1. 前一輪要求的atomic sync與SMF create compensation只有部分完成。
2. 前一輪修正另外引入runtime fallback及SMF create redirect的新回歸。
3. direct callback與ADRF standard response仍有未補齊的wire contract。
4. 已由PyAnLF接管的SMF discovery/subscription ownership仍留下沒有production caller的舊Go路徑。

本輪確認八項問題，其中三項為P1、五項為P2。另有一項MongoDB operation timeout風險尚未完成受控重現，
獨立列在section 12，不把它當成已確認bug。

本文件不改變既有high-level ownership：

- Go仍是standard SBI、peer routing及central sync coordinator。
- PyAnLF仍擁有Events Subscription domain、runtime、collection policy、SMF resource association及raw storage。
- PyMTLF本輪沒有確認新的implementation defect。

需要修改的是既有owner內部的transaction、cleanup ownership及standard boundary完整性，不新增另一套API或
第二個business owner。

---

## 2. Review Basis

### 2.1 Policy And Plan Sources

本review依下列workspace-local文件執行：

- `nwdaf-docs/docs/development_policy.md`
- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/source-orientation.md`
- `free5gc-dev-skill/references/exemplar-alignment.md`
- `free5gc-dev-skill/references/nf-architecture.md`
- `free5gc-dev-skill/references/sbi-development.md`
- `free5gc-dev-skill/references/openapi-contract.md`
- `free5gc-dev-skill/references/concurrency-lifecycle.md`
- `free5gc-dev-skill/references/testing.md`
- `free5gc-dev-skill/references/review-checklist.md`
- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 3 Analytics Subscription Routing.md`
- 前兩份Phase 3 code review/remediation文件。

Phase 3已固定且與本輪直接相關的要求包括：

- rejected full sync必須對active local state為zero-side-effect。
- SMF回201後，只要peer identity可建立，就必須有active route或pending cleanup owner。
- callback request需在有界memory內驗證，成功204不代表durable storage。
- standard request/response method、status、Location、representation及`ProblemDetails`依operation-specific
  OpenAPI contract處理。
- Go不再擁有SMF candidate selection、fan-out、discovery cache或legacy subscribe/unsubscribe business flow。
- migration完成時沒有production caller的舊private/ownership path必須移除，而不是保留為未指定用途。

### 2.2 Standard Sources

本輪使用下列workspace-local Release 18 corpus：

- `nwdaf-docs/specs/openapi/TS29508_Nsmf_EventExposure.yaml`
  - create為`POST /subscriptions`，成功201、mandatory Location及`NsmfEventExposure` representation。
  - create response沒有宣告307/308。
  - individual GET/PUT/DELETE才宣告307/308。
  - SMF notification callback request只宣告`application/json`，error allowlist包含413及415。
- `nwdaf-docs/specs/openapi/TS29564_Nupf_EventExposure.yaml`
  - UPF notification callback request只宣告`application/json`，成功204，error allowlist包含413及415。
- `nwdaf-docs/specs/openapi/TS29575_Nadrf_DataManagement.yaml`
  - storage create成功201、mandatory Location及`NadrfDataStoreRecord` representation。
  - Phase 3使用的data notification branch要求`dataSub`與`dataNotif`同時存在。
- `nwdaf-docs/specs/openapi/TS29571_CommonData.yaml`
  - common error responses使用`application/problem+json`及`ProblemDetails`。

### 2.3 free5GC Exemplar Scope

本輪延續Phase 3選定的exemplar：

- BSF subscription CRUD：handler/processor separation、resource method及Location handling。
- SMF consumer/callback：outbound peer call放在consumer boundary，callback與standard model分離。
- UDR persistence：processor與repository/persistence boundary。

目前Go的handler、processor、consumer大方向仍符合這些boundary；本輪沒有要求重寫package layout。atomic
Python sync與Go/Python cross-process provisional cleanup沒有free5GC直接precedent，因此相關方案是依本專案已
確認ownership、standard peer identity及lifecycle要求推導，不宣稱是free5GC既有procedure。

### 2.4 Verification Evidence

前一輪implementation record曾完成完整local checks：

```text
NWDAF:  make lint                         PASS
NWDAF:  go test ./...                     PASS
NWDAF:  go test -race ./internal/backend ./internal/context ./internal/anlf/... ./internal/sbi/...  PASS
NWDAF:  make build                        PASS
PyAnLF: uv run ruff check .               PASS
PyAnLF: uv run pytest -q                  195 passed, 1 skipped
PyMTLF: uv run ruff check .               PASS
PyMTLF: uv run pytest -q                  32 passed
```

本次re-review另外執行：

```text
PyAnLF: uv run pytest -q tests/test_sync_api.py tests/test_collection.py tests/test_callbacks.py
         26 passed
NWDAF:  go test ./internal/sbi/consumer ./internal/anlf ./internal/sbi
         PASS
NWDAF, PyAnLF, PyMTLF, nwdaf-docs: git diff --check
         PASS
```

本次也完成兩個direct minimum reproduction：

1. 先送合法sync，再強制runtime commit failure。第二次sync回503，但stored containing NWDAF identity已改成
   rejected snapshot的值，subscription map也被部分清空。
2. 對SMF callback送出合法JSON但使用`Content-Type: text/plain`。目前回400及`application/json`，不是
   OpenAPI允許的415及`application/problem+json`。

全綠tests因此只代表現有assertion通過，不能證明本輪列出的failure window不存在。本輪仍未執行real NRF、
SMF、UPF、ADRF或MongoDB cross-process integration。

---

## 3. Finding Summary And Origin Classification

| ID | Priority | Area | Origin | Confirmed consequence |
|---|---|---|---|---|
| P3-SR-01 | P1 | Atomic sync | P3-FR-01只部分完成 | sync回409/503後仍可留下projection、runtime或subscription partial mutation |
| P3-SR-02 | P1 | Go SMF create compensation | P3-FR-03的Go側只做one-shot compensation | DELETE失敗後remote peer resource沒有可恢復cleanup owner |
| P3-SR-03 | P1 | Callback body limit | callback boundary remediation未完整涵蓋streaming body | 缺少可信Content-Length時，整個oversized body會先配置進memory |
| P3-SR-04 | P2 | Runtime fallback | 前一輪修改造成的新回歸 | replacement失敗但舊runtime仍可用時，service反而刪除舊runtime |
| P3-SR-05 | P2 | Callback media/error contract | 原Phase 3 standard boundary遺漏 | unsupported media type回400，且ProblemDetails media type錯誤 |
| P3-SR-06 | P2 | SMF create redirect | 前一輪redirect修正過度擴張 | create錯誤地pass-through規格未宣告的307/308 |
| P3-SR-07 | P2 | ADRF 201 validation | 原Phase 3 response contract遺漏 | empty或malformed 201 body被視為成功並傳回PyAnLF |
| P3-SR-08 | P2 | Legacy Go ownership cleanup | Phase 3 cutover未完成 | endpoint-only discovery及舊subscribe/unsubscribe policy path仍留在production interfaces/code |

P3-SR-01至P3-SR-03必須先關閉，才能把Phase 3稱為具有安全restart及peer lifecycle。P2項目應在同一批
remediation完成，因為它們都位於這次已修改的runtime或standard boundary，繼續留著會使Phase 3 acceptance與
實際行為不一致。

---

## 4. P3-SR-01: Sync Commit Is Still Fallible And Non-atomic

### 4.1 Expected Behavior

P3-FR-01要求sync有三個清楚階段：

```text
prepare all local state
  -> commit an already validated local transaction
  -> enqueue post-commit convergence
```

任何409/503都必須保留sync前的projection、canonical subscriptions、runtime、peer resources及queue state。
這不要求distributed transaction，只要求單一PyAnLF process內的full snapshot不被部分採用。

### 4.2 Current Behavior

`PyAnLF/src/py_anlf/sbi/routers/sync.py`目前先prepare projection、collection restore及subscription domain
representation，接著在`subscription_service.transaction()`內依序：

1. replace sync projection；
2. commit collection restore；
3. 呼叫`SubscriptionService.reconcile()`；
4. enqueue collection plan；
5. finalize restore。

問題在於`prepare_reconcile()`只做subscription/policy validation，沒有prepare runtime transition。
`reconcile()`在所謂commit階段仍逐筆呼叫`_apply_runtime()`；該方法會載入模型、替換runtime並可能失敗。
前面的projection、collection及stale runtime release沒有rollback。

### 4.3 Confirmed Reproduction

direct reproduction sequence：

```text
sync valid snapshot                           -> 200
sync replacement with forced runtime failure -> 503
stored containing NWDAF identity              -> rejected snapshot value
stored subscriptions                          -> partially cleared
```

因此HTTP response聲稱snapshot未接受，但active state已不是原合法snapshot。

### 4.4 Root Cause

前一輪把validation移到prepare method，卻沒有把runtime manager納入transaction design。lock只能阻止另一個
request交錯，不能讓已發生的mutation自動rollback。使用同一個lock不等於atomic commit。

### 4.5 Remediation Design

推薦新增runtime-aware batch prepare/commit seam，不採用「失敗後逐筆猜測rollback」：

1. `SubscriptionService.prepare_reconcile()`除canonical representation與policy外，建立每個resource的
   `PreparedRuntimeTransition`。
2. runtime prepare可完成model lookup/load及所有可能失敗的validation，但不得：
   - replace active runtime map；
   - release previous runtime；
   - 啟動或停止scheduler；
   - enqueue collection task。
3. provisional model handle或reference在prepare abort時必須釋放；不可因rejected sync累積model reference。
4. `CollectionManager.prepare_restore()`保持pure candidate construction；`commit_restore()`不得再做domain
   validation或peer I/O。
5. 所有prepare成功後取得shared transaction lock，一次提交：
   - sync projection；
   - canonical subscription map；
   - prepared runtime map/scheduler ownership；
   - restored peer resource map。
6. commit API必須只做已準備完成的local state swap。若scheduler activation仍可能失敗，需在prepare建立
   inactive candidate並使activation具明確abort semantics；不能把fallible model preparation留在commit中。
7. stale runtime、old scheduler及old model reference只在new state已成功publish後釋放。
8. collection reconcile/release/finalize tasks只在完整commit後enqueue。
9. external POST/PUT/DELETE與sync commit繼續共用同一serialization boundary，但lock不包NRF、SMF、ADRF或
   Mongo network I/O。

不推薦以snapshot後呼叫現有`apply()`再rollback，因為現有apply/release會改scheduler、runtime revision及model
reference；補償本身也可能失敗或無法重建原thread state。

### 4.6 Required Tests

- runtime prepare failure：projection、subscriptions、runtime revisions、peer resources及queue depth完全不變。
- 第二筆runtime transition failure：第一筆既有runtime不被release或replace。
- collection restore failure：subscription/runtime map不變。
- rejected sync不啟動或停止任何scheduler。
- rejected sync不增加model reference或留下provisional runtime。
- successful sync在commit後才enqueue collection tasks。
- sync與external PUT同時到達：結果對應完整serial order，不出現混合snapshot。
- sync A prepare後被sync B超越：A不能在B後commit或執行peer side effect。

### 4.7 Completion Criteria

- 所有rejected snapshot為zero active-state side effect。
- sync route能清楚辨識prepare、commit、post-commit enqueue。
- commit path不再呼叫目前會失敗且會mutation的逐筆`runtime_manager.apply()`。
- direct reproduction固定為regression test。

---

## 5. P3-SR-02: Go Loses Cleanup Ownership After Failed SMF Create Finalization

### 5.1 Expected Behavior

SMF create回201後，remote resource已存在。只要Go能從Location或representation建立完整peer identity，後續
local parse、merge或route insertion失敗都不能把resource變成無owner orphan。

Phase 3 ownership仍是：

- Go保存target-aware peer route/mirror並把pending state放進AnLF sync snapshot。
- PyAnLF擁有cleanup intent及retry decision。
- 不新增Go與PyAnLF兩套長期cleanup policy。

### 5.2 Current Behavior

`NWDAF/internal/sbi/consumer/smf_standard_proxy.go`在SMF 201後，遇到下列local failure會呼叫一次
`compensateCreatedSmfResource()`：

- accepted representation無法decode；
- request/response representation merge失敗；
- merged representation無法decode；
- context route insertion失敗。

`compensateCreatedSmfResource()`只送一次DELETE。DELETE若為transport error或回非204/404，caller只記log並
返回原create error。此時沒有pending Go route，也沒有PyAnLF `PeerCleanupTask`。

### 5.3 Consequence

remote SMF resource可能持續向callback URI送notification，但Go/PyAnLF都沒有可追蹤的association。後續sync也
不一定能恢復，因為Go根本沒有保存這筆resource。這正是P3-FR-03要求消除的201-to-local-owner failure window。

### 5.4 Remediation Design

1. 201及Location完成基本resolve後，立即建立provisional Go route；不要等accepted representation merge全部
   完成才取得local ownership。
2. provisional route至少保存：
   - normalized target API root；
   - resolved resource Location；
   - peer subscription ID；
   - request correlation；
   - `pendingCleanup=true`；
   - 可用時保存raw/typed accepted representation。
3. accepted representation及route finalization成功後，將provisional route轉為normal active route。
4. local finalization失敗時可先立即嘗試一次compensating DELETE：
   - 204/404：移除provisional route；
   - transient failure：保留provisional route並立即喚醒AnLF sync/refresh，使PyAnLF建立typed cleanup task；
   - permanent response：保留明確failure state與observability，不能log後丟失identity。
5. Go snapshot需能表示「沒有active NWDAF reference但pending cleanup」的route。PyAnLF restore後沿用既有
   target-aware cleanup executor，不建立fake subscription reference。
6. 若route collision發生，先以`targetApiRoot + peerSubscriptionId + resolved Location`確認是否為同一resource。
   不得覆蓋一筆仍active的route再DELETE；若目前context model無法無歧義保存collision，實作應停下並先調整
   typed route store，而不是退回log-only compensation。
7. 如果201沒有Location且body也無法建立peer identity，Go無法安全DELETE；此情境至少回502並產生明確
   orphan-risk metric/log。它不是保留one-shot behavior的理由。

### 5.5 Required Tests

- 201 + malformed representation + compensating DELETE 503：Go保留pending route。
- 上述pending route進入AnLF sync後，PyAnLF建立cleanup task並retry至204。
- 201 + route insertion failure + DELETE transport error：peer identity仍可從snapshot恢復。
- immediate DELETE 204/404：不留下route或association。
- two targets回相同peer subId：pending route與cleanup互不碰撞。
- collision不覆蓋active route、不誤刪另一個resource。
- 201完全缺少可用peer identity：502及orphan-risk observability。

### 5.6 Completion Criteria

- 每個可辨識的SMF 201都對應active route或pending cleanup route。
- compensation failure不再只是log。
- cleanup仍由PyAnLF executor決策及retry，Go只保存routing/restart mirror。

---

## 6. P3-SR-03: Callback Body Limit Does Not Bound Memory Allocation

### 6.1 Expected Behavior

callback body limit需在request body完整載入application memory之前生效。`Content-Length`只能作early reject，
不能當唯一可信邊界，因為header可能缺少、錯誤或使用chunked transfer。

### 6.2 Current Behavior

`PyAnLF/src/py_anlf/sbi/server.py`的callback middleware：

1. 若有`Content-Length`且超過limit，提早回413；
2. 否則呼叫`await request.body()`；
3. 完整body載入後才檢查`len(body)`。

因此沒有header、header小於實際body或chunked body時，limit只能控制後續是否接受，無法限制記憶體配置。

### 6.3 Remediation Design

以小型route-scoped ASGI body-limit middleware取代目前的full-read middleware：

1. 包住ASGI `receive`，逐chunk累計byte count。
2. 最多buffer configured limit；讀到第`limit + 1` byte立即形成413，不再把其餘body附加到memory。
3. 在limit內的body可replay給FastAPI/Pydantic parsing，並將同一份decoded raw JSON放入
   `request.state.raw_notification`；不得再讀第二份完整body。
4. `Content-Length > limit`保留為early reject，但實際stream count仍是authority。
5. middleware只套用`/callbacks/`，不改health/internal sync等route。
6. 413使用`application/problem+json`。
7. malformed或中途disconnect需回適用的400/transport outcome，不得被誤報為500。

若使用Starlette request stream後replay，實作必須有明確single-consumer test；不要依賴沒有contract保證的
private request cache行為。

### 6.4 Required Tests

- no Content-Length、body超過limit：413且buffer從未超過limit加一個chunk的bounded值。
- forged short Content-Length、實際body超限：413。
- chunked multi-part body跨越limit：413。
- body恰好等於limit：可進入schema parsing。
- malformed JSON在limit內：400。
- successful callback只解析一次body，raw notification仍保存原JSON shape。

### 6.5 Completion Criteria

- callback memory使用不隨untrusted body大小無界成長。
- 413在FastAPI/Pydantic建立完整payload之前發生。
- request limit與existing drop-oldest ingestion policy保持獨立。

---

## 7. P3-SR-04: FAILED_USING_PREVIOUS Deletes The Usable Previous Runtime

### 7.1 Expected Behavior

`SubscriptionRuntimeManager.apply()`的`FAILED_USING_PREVIOUS`明確表示新model/runtime preparation失敗，但
舊runtime仍為READY並繼續使用。external replace可以回503表示更新未採用，但不能同時刪掉fallback runtime。

`FAILED_NO_PREVIOUS`才表示沒有可用fallback，失敗placeholder可被release。

### 7.2 Current Behavior

`PyAnLF/src/py_anlf/core/subscription_service.py`的`_apply_runtime()`把
`FAILED_NO_PREVIOUS`與`FAILED_USING_PREVIOUS`放在同一branch，兩者都呼叫
`runtime_manager.release(subscription_id)`再回503。

這直接反轉`runtime_manager.py`保留previous runtime的設計。正常Phase 3 happy path不一定觸發model replacement，
但後續model provision或任何replacement failure一旦發生，就會把本來可服務的runtime刪除。

### 7.3 Remediation Design

1. `FAILED_USING_PREVIOUS`：
   - 不呼叫release；
   - stored canonical subscription保持原值；
   - previous runtime、scheduler、model reference及revision保持可用；
   - external/sync operation回503表示requested transition未commit。
2. `FAILED_NO_PREVIOUS`：
   - 清除failed placeholder及其資源；
   - create/update回503；
   - 不建立canonical resource或collection intent。
3. 將result handling移入P3-SR-01的prepared runtime transition，避免service與runtime manager再次對同一result
   各自做相反處理。

### 7.4 Required Tests

- replacement load failure + previous READY：回503但previous runtime仍存在並繼續report。
- 上述failure不增加collection revision、不改stored subscription。
- initial load failure：沒有active runtime/resource殘留。
- successful later retry可從previous runtime正常replace，不因前次failure找不到state。

### 7.5 Completion Criteria

- `FAILED_USING_PREVIOUS`不再呼叫release。
- runtime manager result semantics在service及tests中一致。
- 此修正納入atomic sync/replace transaction，而不是另外加temporary compatibility branch。

---

## 8. P3-SR-05: Callback Media Type And ProblemDetails Contract Are Incomplete

### 8.1 Expected Behavior

SMF及UPF callback OpenAPI只宣告`application/json` request body，並允許415。common error response使用
`application/problem+json`。

第一版政策應與Go external ingress一致：

- `application/json`及合法parameters可接受；
- missing或unsupported Content-Type回415；
- malformed JSON/schema回400；
- standard error body使用`ProblemDetails`及`application/problem+json`。

### 8.2 Current Behavior

callback route沒有檢查request Content-Type。FastAPI會嘗試解析payload，而custom validation handler及
`callback_problem()`都使用`JSONResponse`預設的`application/json`。

direct reproduction：合法JSON搭配`text/plain`目前回400及`application/json`，不是415及
`application/problem+json`。

### 8.3 Remediation Design

1. 在P3-SR-03 body-limit middleware開始讀body前，以標準media parser解析Content-Type。
2. 只接受`application/json`及合法parameters，例如`charset=utf-8`。
3. missing、malformed或unsupported media type回415；handler/Pydantic不得執行。
4. 建立單一small callback `ProblemDetails` response helper，明確設定
   `media_type="application/problem+json"`。
5. callback body limit、validation exception、correlation failure及route-level error都使用同一helper。
6. 保持standard status差異：malformed syntax/schema為400、unknown correlation為404、oversize為413、
   media type為415。

### 8.4 Required Tests

- SMF與UPF callback各自測`application/json`成功204。
- `application/json; charset=utf-8`成功。
- `text/plain`加合法JSON回415且ingestion未被呼叫。
- missing Content-Type回415。
- malformed JSON回400。
- 400/404/413/415均回`application/problem+json`及matching `ProblemDetails.status`。

### 8.5 Completion Criteria

- callback contract與TS29508/TS29564 status及media type一致。
- direct reproduction固定為regression test。

---

## 9. P3-SR-06: Redirect Fix Incorrectly Applies To SMF Create

### 9.1 Expected Behavior

`TS29508_Nsmf_EventExposure.yaml`的operation-specific response matrix不同：

- collection POST create沒有307/308；
- individual GET/PUT/DELETE有307/308。

因此Go需停用automatic follow，但只對OpenAPI宣告redirect的individual operations原樣pass-through。

### 9.2 Current Behavior

前一輪正確停用SMF client automatic redirect，也讓response writer保留Location；但
`NWDAF/internal/anlf/api_smf_event_exposure.go`的四個operation共用
`writeSmfEventExposureResponse()`，而該helper對任何307/308都直接pass-through。

因此SMF create若回307/308，Go會把規格未宣告的redirect當成合法create response交給PyAnLF。這是修正
individual redirect時過度擴張到collection POST的新回歸。

### 9.3 Remediation Design

1. response writer接收明確operation/method kind，或分成create與individual resource兩個small helper。
2. create：
   - 只接受201及該operation error allowlist；
   - peer回307/308視為undeclared upstream response，映射成create允許的502 `ProblemDetails`；
   - 不把peer Location當成backend應follow的redirect Location。
3. GET/PUT/DELETE：保留307/308、Location、body及peer Content-Type，不automatic follow。
4. handler不能以一張union allowlist處理所有SMF methods。
5. Go route mirror不因任何redirect自動改target；是否採用新target仍由PyAnLF決定。

### 9.4 Required Tests

- create peer 307/308：backend收到502 `ProblemDetails`，不是redirect。
- GET/PUT/DELETE各自307/308：status、Location、body及Content-Type保留。
- normal create 201、GET 200、PUT 200/204、DELETE 204及404不回歸。
- client仍不automatic follow任何SMF redirect。

### 9.5 Completion Criteria

- response handling使用operation-specific OpenAPI allowlist。
- create不再pass-through 307/308。
- individual resource redirect behavior保持前一輪已修正結果。

---

## 10. P3-SR-07: ADRF 201 Representation Is Not Validated

### 10.1 Expected Behavior

ADRF storage create成功需同時具備：

- status 201；
- mandatory Location；
- valid `NadrfDataStoreRecord` representation。

Phase 3的raw SMF/UPF storage path使用`dataSub + dataNotif` branch；Go作為standard communication owner需驗證
peer success contract，再把完整representation傳回PyAnLF。

### 10.2 Current Behavior

`NWDAF/internal/sbi/consumer/adrf_service.go`的`ExecuteStandardStorageRequest()`目前對201只檢查Location，
不驗證response body是否：

- 非空；
- 合法JSON；
- 可形成`NadrfDataStoreRecord`；
- 仍包含本path必要的`dataSub`與`dataNotif`。

因此empty或malformed 201會被當作成功。Go private ADRF handler接著把body以application/json傳回PyAnLF，
PyAnLF sender又只以status判斷成功，錯誤representation不會被任何一側發現。

### 10.3 Remediation Design

1. `StandardAdrfResponse`保存peer Content-Type。
2. 201時要求compatible `application/json` response及非空body。
3. 將bodydecode為目前已存在的typed `consumer.NadrfDataStoreRecord`。
4. 對Phase 3 data notification path驗證：
   - `len(dataSub) > 0`；
   - `dataNotif != nil`；
   - 至少一個supported SMF/UPF notification array非空。
5. Location仍須存在且可作resource URI；body驗證不能取代Location驗證。
6. malformed 201是upstream contract failure，Go private boundary回502 `ProblemDetails`，不得pass-through假成功。
7. valid representation、Location及Content-Type原樣傳回PyAnLF；PyAnLF ADRF worker只有在完整201 contract通過後
   才標記record delivered。

若未來加入analytics notification branch，需擴充typed union validation；本修正不以untyped map先接受所有
未知shape。

### 10.4 Required Tests

- valid 201 + Location + dataSub/dataNotif representation成功。
- 201 empty body回contract failure。
- 201 malformed JSON回contract failure。
- 201缺dataSub或dataNotif回contract failure。
- 201缺Location回contract failure。
- non-201 `ProblemDetails` behavior保持不變。
- PyAnLF不把malformed peer 201計為ADRF write success。

### 10.5 Completion Criteria

- ADRF success只有在status、Location及representation全部合法時成立。
- Go與PyAnLF不再以status-only判斷storage delivery。

---

## 11. P3-SR-08: Legacy Go SMF Ownership Paths Remain After Cutover

### 11.1 Expected Behavior

Phase 3已把下列owner移到PyAnLF：

- NRF discovery cache及candidate selection；
- all-matching fan-out；
- standard SMF subscription construction；
- subscribe/unsubscribe decision及refcount；
- target-aware cleanup intent。

Go只應保留full `SearchResult` discovery proxy、selected-target standard transport、OAuth/peer communication及
process-local route mirror。

### 11.2 Current Behavior

Go的新production path已使用：

- `DiscoverSmfProfiles()`回完整`SearchResult`；
- standard-shaped SMF create/read/replace/delete proxy。

但舊path仍存在：

- `ConsumerAPI.DiscoverSmfEventExposure()`回endpoint array；
- `SubscribeToSmf()`及`UnsubscribeFromSmf()` wrappers；
- `NrfService.DiscoverSmfEventExposure()`內的Go-owned endpoint extraction、validity cache及in-flight coalescing；
- legacy `smf_service.go` subscribe/unsubscribe methods；
- 對應mocks及只驗證舊ownership的tests。

production search只找到新`DiscoverSmfProfiles()`由AnLF proxy使用；舊methods只剩自身interface/wrapper及tests，
沒有現行production caller。

### 11.3 Consequence

這些code目前不一定直接破壞happy path，但造成三個問題：

1. Phase 3 acceptance要求的legacy removal尚未完成。
2. Go仍看起來擁有另一套discovery cache及subscribe policy，未來維護者可能誤用並重新形成雙owner。
3. mocks/tests繼續固定已被計畫取代的API，增加後續Phase維護成本。

### 11.4 Remediation Design

1. 先以production call graph確認legacy methods沒有Phase 4至Phase 6的必要caller。
2. 從`ConsumerAPI`、`SmfServiceClient`及mocks移除endpoint-only discovery與legacy subscribe/unsubscribe methods。
3. 移除`NrfService.DiscoverSmfEventExposure()`及只為endpoint-array cache存在的state/helpers/tests。
4. 移除legacy `smf_service.go` high-level subscribe/unsubscribe path；若底層HTTP client、OAuth context或transport仍被
   standard proxy使用，只保留共享低階能力，不為刪檔而重建duplicate client。
5. 保留`DiscoverSmfProfiles()`與standard SMF proxy tests，並新增negative call-graph check或review assertion，
   確認Go沒有candidate selection/fan-out production code。
6. 不為future phase保留沒有current caller的compatibility wrapper；後續若真的需要新standard operation，依當時
   contract新增。

### 11.5 Required Tests And Checks

- `rg`/compile確認production code沒有舊method caller或definition。
- NWDAF全套tests及build通過。
- full SearchResult proxy、OAuth context及standard SMF transport tests仍通過。
- PyAnLF discovery cache/all-matching selection tests仍為唯一business owner evidence。

### 11.6 Completion Criteria

- Go不再暴露endpoint-only SMF discovery或legacy subscribe/unsubscribe business API。
- production path只有「PyAnLF選擇，Go執行standard peer communication」。
- 刪除舊code沒有影響shared OAuth、transport或new proxy behavior。

---

## 12. Unconfirmed Risk: MongoDB Write Timeout And Shutdown Bound

### 12.1 Current Evidence

`PyAnLF/src/py_anlf/core/ingestion.py`建立`MongoClient`時只明確設定
`serverSelectionTimeoutMS`。這限制找不到server時的selection wait，但沒有明確限制已選到server後單次socket
read/write或write concern等待時間。

Phase 3與第一輪remediation要求shutdown不能在Mongo worker仍操作repository時關閉dependency。若
`insert_one()`在network blackhole或half-open connection下長時間不返回，目前shutdown join可能沒有可靠的
operation upper bound。

本輪沒有完成可穩定重現這個情境的Mongo fault injection，因此不把它列為confirmed finding。

### 12.2 Required Diagnostic

在改production config前先建立受控test：

1. 讓Mongo initial selection成功。
2. 在`insert_one()`期間模擬socket不回覆或write concern不完成。
3. 量測operation是否能在目前configured timeout內返回。
4. 同時觸發pipeline shutdown，確認worker與repository close ordering。

### 12.3 Conditional Remediation

若diagnostic確認operation可超過shutdown bound：

- 在Mongo client明確設定connect、server selection及socket timeout；必要時設定write concern timeout。
- timeout值與pipeline shutdown deadline建立可驗證關係。
- timeout視為transient sink failure，進入既有independent bounded retry queue。
- 新增Mongo operation timeout及shutdown regression tests。

不得只把join timeout調大來掩蓋無界I/O，也不得在worker仍存活時先close repository。

---

## 13. Cross-finding Remediation Sequence

### Step 1: Lock Current Failures With Regression Tests

先加入以下focused reproduction：

- rejected sync runtime failure仍mutation；
- Go SMF 201後compensation DELETE failure；
- callback no/false Content-Length oversized body；
- `FAILED_USING_PREVIOUS`被release；
- callback text/plain response；
- SMF create 307/308；
- ADRF malformed 201。

concurrency/lifecycle tests使用Event、barrier或controllable fake dependency，不以固定`sleep`作唯一判斷。

### Step 2: Complete Atomic Runtime-aware Sync

同批完成P3-SR-01及P3-SR-04：

- runtime manager加入prepare/commit/abort seam；
- canonical subscription、runtime及peer restore形成單一local transaction；
- 修正fallback semantics；
- commit後才enqueue collection work。

這兩項不能拆成「先加rollback，之後再整理」，否則會再留下scheduler/model reference不一致。

### Step 3: Close SMF 201 Ownership Window

完成P3-SR-02：

- Go在201後立即保存provisional target-aware route；
- one-shot compensation失敗保留pending state；
- PyAnLF透過sync接管typed cleanup retry；
- collision及restart tests固定完整ownership。

### Step 4: Harden Direct Callback Boundary

同批完成P3-SR-03及P3-SR-05：

- streaming bounded body read；
- actual Content-Type validation；
- 400/404/413/415 `application/problem+json`；
- 保留raw notification single-read及204 enqueue semantics。

### Step 5: Correct Operation-specific Standard Responses

同批完成P3-SR-06及P3-SR-07：

- SMF create與individual operation使用不同status allowlist；
- ADRF 201完整驗證Location及representation；
- 保持Go handler/processor/consumer boundary，不把peer validation放進PyAnLF business layer。

### Step 6: Remove Superseded Go Ownership

完成P3-SR-08：

- 移除endpoint-only discovery、legacy subscribe/unsubscribe interfaces/callers/tests；
- 保留new standard proxy所需low-level transport及OAuth behavior；
- 跑production call graph與full regression。

### Step 7: Mongo Diagnostic And Full Re-review

- 完成section 12 fault-injection diagnostic。
- 執行所有repository full checks及real peer integration（環境允許時）。
- 再做focused code review；不能只以test count判定本文件resolved。

---

## 14. Verification Matrix After Remediation

### 14.1 NWDAF

```bash
make lint
go test ./...
go test -race ./internal/backend ./internal/context ./internal/anlf/... ./internal/sbi/...
make build
```

需新增或更新：

- SMF provisional route及compensation failure tests。
- SMF operation-specific redirect allowlist tests。
- ADRF 201 Location/Content-Type/representation validation tests。
- legacy SMF ownership removal後的interface/mock/consumer tests。

### 14.2 PyAnLF

```bash
uv run ruff check .
uv run pytest -q
```

需新增focused suites：

- runtime-aware atomic sync rejection matrix。
- previous-runtime fallback preservation。
- streamed callback body limit及media type matrix。
- Go pending peer route restore/cleanup scenario。
- ADRF malformed success不計為delivered。
- conditional Mongo timeout/shutdown test。

### 14.3 PyMTLF

```bash
uv run ruff check .
uv run pytest -q
```

本輪沒有PyMTLF-specific finding。只要sync envelope不變，執行完整regression即可；若atomic transaction實作需要
修改shared sync schema，必須先回到plan decision gate，不能讓AnLF與MTLF出現不同handshake格式。

### 14.4 Cross-process Scenarios

環境允許時至少驗證：

1. Go + PyAnLF + PyMTLF readiness→sync→usable。
2. invalid/failed runtime sync被拒絕後，原runtime仍繼續report。
3. SMF POST 201後Go local processing失敗、DELETE暫時失敗，PyAnLF最終完成cleanup。
4. SMF/UPF對PyAnLF callback使用正確與錯誤media type、chunked oversized body。
5. SMF create與individual operation回307/308時，各自得到operation-specific結果。
6. ADRF回valid及malformed 201時，Mongo/ADRF independent sink state正確。
7. Mongo write network stall期間shutdown能在documented bound內完成。

real NRF、SMF、UPF、ADRF或MongoDB未執行時，final record需逐項標示unit、mock cross-process與real integration
層級，不得稱為完整end-to-end validation。

---

## 15. Scope And Decision Boundaries

本remediation不包含：

- THRESHOLD或ONE_TIME analytics擴張。
- AoI SMF discovery、automatic SMF migration或UDM Group ID resolution。
- MTLF dataset retrieval、training或model provision。
- Go process durable persistence。
- TLS/OAuth delegation擴張。
- distributed transaction、message broker或跨process CAS。
- finite SMF lease renewal。

目前八項finding都能保持既有Go/PyAnLF ownership與wire contract，不需要新增high-level product decision。

實作遇到下列情況時必須先更新計畫並要求決策：

1. runtime manager無法提供prepare/commit/abort，唯一方案會改成persistent或distributed rollback。
2. Go context無法保存provisional/pending peer route，且修改會改變Go/PyAnLF authority，而不只是typed mirror。
3. callback streaming limit需要改變已確認的204/drop-oldest semantics。
4. ADRF accepted representation無法以現有typed model表達，需要OpenAPI generation或擴張standard schema support。
5. legacy Go SMF path仍有Phase 4至Phase 6的actual production caller，不能直接移除。

---

## 16. Final Acceptance Gate

只有以下條件全部成立，才能把Phase 3 second follow-up標記resolved：

1. P3-SR-01至P3-SR-08各自有production fix及regression test。
2. rejected sync對projection、subscriptions、runtime、scheduler、model references、peer resources及queues為
   zero-side-effect。
3. `FAILED_USING_PREVIOUS`保留舊runtime，`FAILED_NO_PREVIOUS`不留下failed active state。
4. 每個可辨識的SMF 201都有active或pending-cleanup owner，compensation failure可跨restart恢復。
5. callback request body在stream read期間即有界，且400/404/413/415 status/media type符合OpenAPI。
6. SMF create不接受307/308，individual GET/PUT/DELETE仍正確pass-through redirect。
7. ADRF 201只有在Location及typed representation都合法時被視為成功。
8. Go舊endpoint-only discovery與legacy subscribe/unsubscribe ownership path已移除。
9. 第一輪P3-CR及P3-FR regression tests仍全部通過。
10. NWDAF lint、all tests、race及build通過。
11. PyAnLF/PyMTLF lint及all tests通過。
12. Mongo timeout risk已有受控診斷及記錄；若確認，對應fix及test一併完成。
13. 未執行real integration checks有具體環境說明，不誤稱完整end-to-end validation。
14. focused re-review沒有新的P0/P1 finding。
