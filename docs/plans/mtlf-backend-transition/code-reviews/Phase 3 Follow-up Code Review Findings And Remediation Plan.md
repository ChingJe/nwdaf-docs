# Phase 3 Follow-up Code Review Findings And Remediation Plan

Date: 2026-07-22

Status: Historical review record; current gate is tracked in `Phase 3 Review Ledger.md`

Parent documents:

- `../MTLF Backend Transition Plan.md`
- `../Phase 3 Analytics Subscription Routing.md`
- `Phase 3 Code Review Findings And Remediation Plan.md`

Second follow-up review:

- `Phase 3 Second Follow-up Code Review Findings And Remediation Plan.md`

Canonical current ledger:

- `Phase 3 Review Ledger.md`

This document preserves the follow-up findings and remediation history. Its
individual acceptance gate is superseded by the canonical ledger.

Reviewed implementation repositories:

- `NWDAF/`
- `PyAnLF/`
- `PyMTLF/`

---

## 1. Purpose

本文件記錄第一輪Phase 3 remediation完成後的follow-up code review結果。它不取代第一輪review文件，
而是把修正後仍存在或新暴露的問題、來源分類、最小重現、影響、修正方案及驗收條件保存為第二輪
remediation work list。

本輪確認六個問題：

1. sync不是atomic transaction；被拒絕的snapshot仍可部分修改active state。
2. PyAnLF建立的SMF subscription有固定expiry，但沒有renewal或expiry reconciliation。
3. stale revision發生於SMF POST成功後時，補償DELETE失敗會遺失local cleanup intent。
4. partial acceptance的request與accepted representation不一致，首次full sync會不必要地重建runtime。
5. external NWDAF Events Subscription ingress缺少request body上限與實際Content-Type驗證。
6. SMF 307/308 redirect的Location及原始redirect語意沒有完整保留。

本輪沒有確認package boundary或Go handler/processor/consumer ownership產生新的結構性問題。主要缺口集中在：

- cross-manager local state consistency；
- SMF peer resource lifecycle；
- canonical standard representation；
- standard HTTP wire behavior。

---

## 2. Review Basis

### 2.1 Policy And Plan Sources

本review依下列workspace-local文件執行：

- `nwdaf-docs/docs/development_policy.md`
- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/exemplar-alignment.md`
- `free5gc-dev-skill/references/review-checklist.md`
- `free5gc-dev-skill/references/sbi-development.md`
- `free5gc-dev-skill/references/openapi-contract.md`
- `free5gc-dev-skill/references/config-and-lifecycle.md`
- `free5gc-dev-skill/references/concurrency-lifecycle.md`
- `free5gc-dev-skill/references/testing.md`
- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 3 Analytics Subscription Routing.md`
- `nwdaf-docs/docs/plans/mtlf-backend-transition/code-reviews/Phase 3 Code Review Findings And Remediation Plan.md`

Phase 3與第一輪remediation已固定的相關要求包括：

- sync先完整validate/stage local snapshot，再commit local state並enqueue peer convergence；不得在被拒絕後留下
  partial mutation。
- `snapshotAccepted=true`只代表local intent已完整恢復，不代表NRF/SMF已同步收斂。
- Go保存accepted standard Events Subscription snapshot，供backend restart/full refresh使用。
- stale revision不得留下沒有cleanup intent的peer side effect。
- SMF peer identity固定為`targetApiRoot + peerSubscriptionId`。
- Go是standard SBI wire boundary，需依Release 18 OpenAPI保留status、Location、representation及
  `ProblemDetails`。
- worker與retry均由app lifecycle擁有；本輪沒有再確認detached worker問題，但新增peer cleanup intent必須沿用
  已修正的executor lifecycle。

### 2.2 Standard Sources

本輪使用下列workspace-local Release 18 corpus：

- `nwdaf-docs/specs/openapi/TS29520_Nnwdaf_EventsSubscription.yaml`
  - POST request只宣告`application/json`。
  - POST error responses包含413及415。
  - POST success為201、mandatory Location及`NnwdafEventsSubscription` representation。
- `nwdaf-docs/specs/openapi/TS29508_Nsmf_EventExposure.yaml`
  - `expiry`是optional `DateTime`。
  - create success為201、mandatory Location及`NsmfEventExposure` representation。
  - individual resource GET/PUT/DELETE宣告307/308 redirect responses。
- `nwdaf-docs/specs/TS 29.508/4 Session Management Event Exposure Service/4.2 Service Operations/4.2.3 Nsmf_EventExposure_Subscribe Service Operation.md`
  - request含expiry時，SMF可選擇小於或等於request的accepted expiry。
- `nwdaf-docs/specs/TS 29.508/5 Nsmf_EventExposure API/5.6 Data Model/5.6.2 Structured data types/5.6.2.2 Type NsmfEventExposure.md`
  - expiry之後SMF不再送notification且subscription失效。
  - response沒有expiry時，consumer不得自行假設expiry。

上述expiry evidence表示：一旦PyAnLF主動送出expiry，就必須以SMF accepted response作為真實期限，並在期限前
renew/recreate；不能只保存本地request並永久視為active。

### 2.3 free5GC Exemplar Scope

本輪延續Phase 3既有exemplar：

- BSF subscription CRUD：handler/processor separation、POST/PUT/DELETE及Location。
- SMF consumer/callback：outbound peer call位於consumer boundary，callback與standard model處理分離。
- UDR persistence：processor與repository/persistence boundary。

目前Go code大致維持external handler、processor、consumer及context mirror的分層。本輪finding不是要求重寫
package layout。free5GC沒有「Go containing NWDAF + Python backend atomic restart snapshot」的直接precedent，
因此atomic sync與cross-process cleanup方案是依本專案ownership及lifecycle要求推導，屬inference而非直接
複製free5GC implementation。

### 2.4 Verification Baseline

Follow-up review時執行：

```text
PyAnLF: uv run ruff check .
         PASS
PyAnLF: uv run pytest -q
         191 passed, 1 skipped

NWDAF:  make lint
         PASS, 0 issues
NWDAF:  go test ./...
         PASS
NWDAF:  go test -race ./internal/backend ./internal/context ./internal/anlf/... ./internal/sbi/...
         PASS
NWDAF:  make build
         PASS

PyMTLF: uv run ruff check .
         PASS
PyMTLF: uv run pytest -q
         32 passed, 1 dependency deprecation warning
```

本輪沒有執行real NRF、SMF、UPF、ADRF或Mongo cross-process integration。現有checks全綠只證明已覆蓋的
unit/contract behavior通過；下列failure window沒有被既有tests完整涵蓋。

---

## 3. Finding Summary And Origin Classification

| ID | Priority | Area | Origin | Confirmed consequence |
|---|---|---|---|---|
| P3-FR-01 | P1 | Atomic sync | 原問題在第一輪修正後仍未完整關閉 | sync回409/503後仍可留下新projection或部分runtime mutation |
| P3-FR-02 | P1 | SMF expiry | 最初Phase 3 collection migration引入 | 約一小時或更早後subscription失效且不會恢復collection |
| P3-FR-03 | P1 | Stale create cleanup | 第一輪stale-revision補償修正的未處理edge case | POST已建立的peer在DELETE暫時失敗後沒有local retry intent |
| P3-FR-04 | P2 | Canonical subscription state | 原始Phase 3 create/sync representation不一致 | partial acceptance後首次full sync不必要地增加revision並重排collection |
| P3-FR-05 | P2 | External wire validation | Phase 3之前已存在，Phase 3 acceptance尚未補齊 | oversized或錯誤media type request無法依contract回413/415 |
| P3-FR-06 | P2 | SMF redirect | 原始Phase 3 standard proxy缺口 | 307/308可能被Go自動follow，或error path遺失Location |

只有P3-FR-03可直接分類為最新修正衍生的新edge case。P3-FR-01是原本明確要求修正但目前只完成部分；
其餘是follow-up review額外找到的原始Phase 3或更早缺口。

---

## 4. P3-FR-01: Rejected Sync Can Partially Mutate Active State

### 4.1 Expected Behavior

第一輪remediation要求sync只執行local atomic restore：

1. 完整驗證sync envelope及所有cross-resource reference。
2. 建立temporary validated restore representation。
3. 一次commit projection、subscription/runtime intent與peer metadata。
4. commit後才enqueue collection create/release/cleanup工作。
5. 任一步驗證失敗時，回409或503且active state完全不變。

這裡的atomic不是要求Mongo transaction或distributed transaction；只要求單一PyAnLF process內同一份full
snapshot不能被部分採用。

### 4.2 Current Behavior

`PyAnLF/src/py_anlf/sbi/routers/sync.py`的`sync_backend()`目前依序呼叫：

```text
sync_projection.replace(payload)
collection_manager.restore(...)
subscription_service.reconcile(...)
collection_manager.finalize_restore()
```

第一步已修改active projection，之後的collection或subscription validation才可能拋出`ValueError`或其他
exception。exception只影響HTTP response，不會rollback前面已完成的mutation。

`SubscriptionService.reconcile()`也會逐筆處理：

- 先release stale subscription runtime及collection reference；
- 再逐筆validate/apply desired snapshot；
- 某一筆後段失敗時，前面已release或apply的resource不會恢復。

因此問題不只存在projection，也存在多subscription snapshot的partial application。

### 4.3 Confirmed Reproduction

最小診斷先送出合法snapshot，再送出：

- 不同的containing NWDAF NF instance ID；
- 含invalid `targetApiRoot`的SMF resource。

第二次sync結果：

```text
HTTP status:       409
stored nfInstanceId after rejection: nwdaf-rejected
```

也就是Go收到「snapshot未接受」，但PyAnLF的projection已採用被拒絕的NF identity。

### 4.4 Root Cause

第一輪修正完成了：

- active subscription ID intersection；
- restore順序調整；
- peer I/O離開sync request；

但「validate/stage/commit」仍只是文件順序，沒有形成production transaction boundary。各manager仍各自
validate並立即mutation，因此cross-manager failure無法保持all-or-nothing。

### 4.5 Remediation Design

新增一個PyAnLF local sync transaction coordinator；不新增HTTP API、wire revision或distributed lock。

建議流程：

```text
parse Pydantic envelope
  -> prepare projection candidate
  -> derive and validate unique active subscription IDs
  -> prepare canonical accepted subscription/runtime plans
  -> prepare restored peer resources and cleanup plans
  -> cross-check every reference and duplicate peer tuple
  -> acquire sync commit lock
  -> replace local projection/resource/runtime intent maps
  -> release commit lock
  -> enqueue latest collection and cleanup tasks
  -> return 200 snapshotAccepted=true
```

具體規則：

1. `SyncProjection`提供pure validation/candidate construction；prepare階段不得寫入`_snapshot`。
2. `SubscriptionService`提供prepare reconcile結果，至少包含：
   - canonical accepted subscription map；
   - resolved reporting policy/runtime configuration；
   - stale ID set；
   - collection intents。
3. `CollectionManager`提供prepare restore結果，至少包含：
   - target-aware restored resource map；
   - active reference intersection；
   - explicit cleanup tasks；
   - association snapshot。
4. prepare階段不得啟動scheduler、修改runtime、enqueue task或呼叫NRF/SMF。
5. commit階段只能執行已驗證且不應再產生domain error的local map/state replacement。
6. 若runtime manager目前只能逐筆create/release，需新增full local restore/replace seam或可rollback transaction；
   不得繼續以「失敗機率低」接受partial state。
7. commit與正常external create/update/delete需共用明確serialization。sync commit期間不得有另一個request插入
   一半狀態；但lock不得包住peer network I/O。
8. queue enqueue只在commit後執行。enqueue使用latest-state/coalescing，不會讓舊snapshot task覆蓋新snapshot。
9. prepare或commit失敗回409/503時，projection、subscriptions、runtime revisions、peer resources及queue depth
   都保持原值。

### 4.6 Required Tests

- invalid peer target使sync回409，containing NWDAF projection完全不變。
- 第二筆subscription invalid時，第一筆既有runtime不被release或replace。
- duplicate subscription ID或duplicate composite peer tuple在mutation前被拒絕。
- collection restore validation失敗時subscription/runtime/association都不變。
- commit後才enqueue；被拒絕snapshot不增加ready/delayed task數。
- sync與external PUT同時到達時，結果對應完整serial order，不出現混合snapshot。
- sync A prepare中被sync B超越時，A不得在B後commit或執行peer side effect。
- sync成功後peer convergence仍為background，latency不受NRF/SMF timeout影響。

### 4.7 Completion Criteria

- 所有rejected snapshot都具有zero local side effect。
- `snapshotAccepted=true`只在完整local commit後回傳。
- sync code中能清楚辨識prepare、commit及post-commit enqueue三個階段。
- regression test直接固定本輪`409但NF identity已改變`的重現情境。

---

## 5. P3-FR-02: SMF Subscription Expires Without Renewal

### 5.1 Expected Behavior

TS 29.508允許consumer在request提供optional expiry。若提供：

- SMF可回覆等於或早於request的expiry；
- accepted response中的expiry才是consumer可依賴的期限；
- expiry後SMF不再送notification，subscription成為invalid。

因此合法實作只能選擇：

1. 不要求expiry，讓consumer不建立本地expiry假設；或
2. 要求expiry，保存accepted expiry並完整實作renewal/recreate lifecycle。

不能送出expiry後永久把resource視為active。

### 5.2 Current Behavior

`PyAnLF/src/py_anlf/core/collection.py`目前：

- 從`collection.subscription_duration_seconds`讀取期限，預設3600秒；
- create request固定設定`expiry = now + duration`；
- SMF回201後保存的是本地request加peer subId，不是SMF accepted representation；
- 沒有renewal timer、PUT replace、expiry reconciliation或到期後recreate；
- existing resource只要仍存在於local map，就會阻止新的create。

### 5.3 Confirmed Consequence

這是由code path與TS 29.508 expiry semantics直接確認的lifecycle failure：

```text
PyAnLF POST with expiry=now+3600
SMF accepts expiry <= requested value
time reaches accepted expiry
SMF stops notifications and resource becomes invalid
PyAnLF still sees CollectionResource in _resources
normal reconcile reuses the stale local resource
data collection does not recover
```

實際停止時間可能少於一小時，因為SMF可回覆較早expiry，而目前PyAnLF沒有保存該accepted value。

### 5.4 Origin

舊Go collection path沒有實際送出SMF expiry。這個問題是collection ownership搬到PyAnLF時新增
`subscription_duration_seconds`及request expiry，卻沒有一併設計renewal所造成；不是第一輪restart/worker
remediation引入。

### 5.5 Confirmed Phase 3 Remediation

Phase 3只要求維持local experiment的continuous collection，主計畫沒有要求finite SMF lease或renewal feature。
已確認採用最小且行為保真的方案：

1. create request不再設定`expiry`。
2. 移除只為該欄位存在的`collection.subscription_duration_seconds` config、validation及sample value，避免留下
   無production effect的dead config。
3. 仍保存完整SMF accepted representation，不能以request覆蓋SMF實際接受的欄位。
4. Phase 3不實作finite-lease renewal。若目標SMF依operator policy在request未提供expiry時仍回覆expiry，
   PyAnLF不得把它保存為永久active resource；第一版將該peer binding視為unsupported finite lease，建立
   target-aware cleanup intent、保留原collection intent並輸出明確log/metric。這個controlled-environment
   limitation需保留在交付紀錄，不能靜默等到expiry後停止資料。

### 5.6 Deferred Alternative: Full Renewal Lifecycle

若未來需要支援會強制回覆expiry的SMF，需另行規劃完整finite-lease lifecycle：

1. 保存SMF 201/200 accepted representation及accepted expiry。
2. 依accepted expiry排入typed renewal task，不能依request duration自行推算。
3. 在期限前以原`targetApiRoot + peerSubscriptionId`執行standard PUT replace；是否重新POST只能在resource已
   不存在或standard response要求時進行。
4. renewal使用既有bounded executor、target-aware identity、stale revision check及retry policy。
5. renewal暫時失敗不得阻塞其他subscription；若超過expiry，local resource需標記expired並recreate，不能繼續
   當作usable binding。
6. update、last-reference delete、restart restore與renewal同一peer resource必須serializable。
7. Go mirror與sync snapshot需保存accepted representation中的expiry，不能只保存原request。

此方案明顯擴張lifecycle與tests，已確認不納入目前Phase 3 remediation。

### 5.7 Confirmed Decision

Phase 3移除request expiry及`subscription_duration_seconds` config，不實作renewal。完整renewal只在未來實驗
需要finite SMF lease時另行規劃，不得留下未使用config或半套timer。

### 5.8 Required Tests

- SMF create request不含expiry。
- accepted response原樣保存，不以request覆蓋SMF欄位。
- sample config與startup validation不再出現dead subscription duration。
- repeated reconcile不因本地虛構期限重建resource。
- SMF主動回覆expiry時，不建立永久active binding；保留collection intent、建立cleanup intent並增加明確
  unsupported finite-lease observability。

### 5.9 Completion Criteria

- PyAnLF不主動要求finite expiry，且不存在「本地仍active但SMF已因expiry失效」的狀態。
- PyAnLF保存SMF accepted representation，而非以request假裝accepted state。
- config、runtime behavior與tests都固定為no-request-expiry/no-renewal policy。

---

## 6. P3-FR-03: Failed Compensation Loses Cleanup Intent

### 6.1 Expected Behavior

Phase 3要求stale revision不得留下peer side effect。由於SMF POST一旦送出就無法在本地取消，revision可能在
request in flight時變更，因此POST成功後必須再次檢查current revision。若已stale：

- 不得把resource加入舊revision的active reference；
- 必須建立target-aware cleanup intent；
- DELETE暫時失敗時必須retry，直到204、terminal 404或明確的permanent outcome。

### 6.2 Current Behavior

`CollectionManager._create_or_share()`目前：

1. POST SMF。
2. 解析201 response、peer ID及Location。
3. 建立local `CollectionResource` object。
4. 再次檢查task revision。
5. 若stale，立即呼叫一次`_delete_peer(resource)`。
6. DELETE失敗只記error log，之後raise stale reconciliation error。
7. resource直到revision check通過後才加入`_resources`。

因此DELETE失敗時，resource不在authoritative table，也沒有`PeerCleanupTask`。worker看到task已stale後不會為
它安排下一次retry。

### 6.3 Confirmed Reproduction

最小診斷使用mock SMF：

- POST回201並在回覆前讓subscription revision變stale；
- DELETE回503。

結果：

```text
retry scheduled:       false
tracked resources:     0
DELETE attempts:       1
```

Go可能仍保存create route，未來full sync也可能把orphan帶回，但local correctness不能依賴「之後剛好又有一次
sync」。若association publish/sync延遲，peer會在該期間永久無owner。

### 6.4 Root Cause

第一輪修正正確加入post-POST revision check與compensating DELETE，但把補償當成one-shot best effort。
production resource只有在revision仍current時才進入tracking，造成「remote resource已存在」與「local owner尚未
建立」之間的failure window。

### 6.5 Remediation Design

規則改為：SMF回201且能取得完整peer identity後，remote resource就必須立刻由某個local typed state擁有。

推薦使用既有explicit peer cleanup lane：

1. 201 response完成validation後建立provisional `CollectionResource`或直接建立`PeerCleanupTask`。
2. task identity使用完整`CollectionKey`，至少包含target API root、peer subscription ID及resource Location。
3. revision仍current時，provisional resource轉成normal active resource並加入正確references。
4. revision已stale時，不加入fake subscription reference，直接enqueue cleanup task。
5. cleanup 204或terminal 404後移除provisional resource及Go mirror。
6. cleanup transient failure保留resource/task並依bounded delayed retry重排。
7. cleanup intent在association full replacement中以`pendingCleanup`或等價typed snapshot發布給Go，確保backend
   restart可恢復。
8. 同一composite peer的active/release/cleanup task需coalesce；不得同時POST duplicate與DELETE同一resource。

Go的`CreateSmfEventExposure()`也需檢查相同failure family：

- SMF已回201後，若response body malformed、Location無法resolve或local route insertion collision，Go不能只回
  error並遺留remote resource。
- 能取得Location時應執行bounded compensating DELETE或保留pending route供PyAnLF cleanup；不能生成第二套
  cleanup owner。
- 若連完整peer identity都無法建立，需記錄明確orphan-risk metric/log，因為此時無法安全定位DELETE target。

### 6.6 Required Tests

- POST成功、revision stale、DELETE 503：建立target-aware cleanup task並retry。
- 上述情境在PyAnLF restart後，由Go snapshot恢復並完成DELETE。
- POST成功、revision stale、DELETE 204：不留下resource或association。
- POST成功、revision stale、DELETE 404：視為terminal cleanup success。
- cleanup retry期間新revision需要相同collection key：行為serializable，不誤刪新active binding。
- 兩個SMF回相同subId時，各自擁有獨立provisional/cleanup task。
- Go在201後遇到malformed representation或route collision時不靜默leak peer。

### 6.7 Completion Criteria

- 任一201 response只要能取得peer identity，就必定對應active resource或pending cleanup intent。
- compensation failure不再是log-only。
- stale revision、restart及target collision tests均不留下unreachable peer resource。

---

## 7. P3-FR-04: Partial Acceptance Is Not Canonical Across Create And Sync

### 7.1 Expected Behavior

Phase 3要求：

- PyAnLF擁有standard Events Subscription domain resource。
- Go保存PyAnLF回傳的accepted standard snapshot。
- full sync送回同一份accepted representation時，PyAnLF應判斷為no-op。
- `failEventReports`是partial acceptance response的一部分，不代表consumer又更新subscription。

### 7.2 Current Behavior

`SubscriptionService.create()`及`replace()`目前：

- `_validate_and_build_response()`建立帶`failEventReports`的response；
- runtime apply使用request及resolved policy；
- `_resources`保存原始request，而不是response。

Go在backend create成功後保存accepted response，full sync也送accepted response。`reconcile()`使用完整Pydantic
model equality比較current resource與snapshot；因此partial acceptance時：

```text
current PyAnLF resource: original request, no failEventReports
Go snapshot:             accepted response, has failEventReports
comparison:              different
```

PyAnLF便重新apply runtime、增加revision並再次reconcile collection。

### 7.3 Confirmed Reproduction

最小診斷建立同時包含：

- accepted `UE_COMMUNICATION` periodic event；
- unsupported event並產生`failEventReports`。

結果：

```text
create status:           201
runtime revision:        1
Go-style accepted sync:  200
runtime revision:        2
```

consumer沒有更新subscription，但首次full sync被視為replace。第一次sync後PyAnLF已保存accepted snapshot，
所以後續完全相同的periodic refresh通常會no-op；問題主要出現在每次partial-accept create/replace之後的首次sync。

### 7.4 Consequences

- scheduler/runtime被不必要地重建。
- 已排入的collection task因revision變更成為stale。
- create後立即觸發Go refresh時，可能與initial collection create交錯。
- standard accepted resource在Go與PyAnLF之間沒有單一canonical representation。

### 7.5 Remediation Design

推薦讓PyAnLF domain resource保存canonical accepted representation：

1. create/replace validation產生`accepted_subscription`及`EffectiveReportingPolicy`。
2. runtime仍只依resolved policy與accepted events運作，不從`failEventReports`重新推導business decision。
3. `_resources`保存`accepted_subscription`，與回給Go的response完全相同。
4. `get()`、sync snapshot比較及replace current-state comparison都使用同一canonical form。
5. `failEventReports`由當次request重新計算，不接受consumer提供的舊response-only值成為authority。
6. reconcile收到Go snapshot時先做canonical validation/normalization；如果semantic domain input及recomputed
   acceptance相同，視為no-op。
7. 不以單純「比較時忽略所有failEventReports」掩蓋真正domain change。若supported feature/config改變使
   accepted set不同，應形成明確reconcile結果，而不是永遠忽略。

此修正應與P3-FR-01 prepare stage整合：sync prepare先把每個snapshot轉成canonical accepted representation，
再決定是否需要commit runtime change。

### 7.6 Required Tests

- partial-accept create後立即full sync，runtime revision保持不變。
- partial-accept replace後立即full sync，revision只因replace增加一次。
- full accepted resource round-trip保持`failEventReports`及external/internal notification URI rewrite。
- consumer偽造`failEventReports`不改變PyAnLF acceptance decision。
- supported event/config真正改變時，canonical comparison能辨識需要reconcile。
- repeated identical full sync不重建scheduler、不重新discovery、不增加collection task。

### 7.7 Completion Criteria

- create/replace response、PyAnLF stored resource及Go snapshot使用同一canonical accepted representation。
- partial acceptance後首次sync為no-op。
- response-only metadata不再造成虛假的consumer update。

---

## 8. P3-FR-05: External Events Subscription Ingress Lacks Wire Limits

### 8.1 Expected Behavior

Go是唯一external NWDAF SBI boundary，負責standard HTTP parsing及wire validation。Release 18
`TS29520_Nnwdaf_EventsSubscription.yaml`對create/update：

- request content宣告`application/json`；
- oversized request可回413；
- unsupported media type可回415；
- malformed JSON回standard `ProblemDetails`。

private AnLF routes已使用`http.MaxBytesReader`及明確body limits，external standard boundary至少應有同等保護。

### 8.2 Current Behavior

`NWDAF/internal/sbi/api_eventssubscription.go`的POST與PUT目前：

- 直接呼叫Gin `GetRawData()`讀取完整body；
- 沒有`http.MaxBytesReader`或`io.LimitReader`；
- 不讀取實際request `Content-Type`；
- 固定呼叫`openapi.Deserialize(..., "application/json")`。

因此錯誤media type只要body碰巧是JSON仍可能被接受；大型body則會在application memory中完整配置後才進入
deserialize。此程式在Phase 3之前已存在，本次migration沒有直接造成，但Phase 3 acceptance明確把standard
wire validation及413/415納入Go責任。

### 8.3 Remediation Design

1. 為external Events Subscription request定義有界limit。第一版建議與Go↔PyAnLF Events Subscription client
   的1 MiB上限一致，避免external接受後private transport無法轉送。
2. POST與PUT在任何body read之前使用`http.MaxBytesReader`。
3. 使用`mime.ParseMediaType`解析實際Content-Type；只接受OpenAPI宣告的`application/json`及合法parameters，
   例如`application/json; charset=utf-8`。
4. missing或unsupported Content-Type回415 `ProblemDetails`；若要允許missing header，必須先更新plan並記錄
   compatibility理由，不能默認固定成JSON。
5. `*http.MaxBytesError`回413，不能落入500 system failure。
6. malformed JSON維持400 malformed syntax。
7. successful body解析仍使用generated `models.NnwdafEventsSubscription`及既有processor，不把domain validation
   移入handler。
8. POST與PUT共用小型wire helper或一致實作，避免兩者status drift；不要建立新的business API abstraction。
9. response `Content-Type`與`ProblemDetails`使用現有`util.GinProblemJson`及free5GC-style helper。

### 8.4 Required Tests

- POST/PUT `application/json`成功進入processor。
- `application/json; charset=utf-8`可接受。
- `text/plain`加合法JSON回415，processor未被呼叫。
- missing Content-Type依確認政策回415。
- body恰好等於limit可解析；超過limit一byte回413。
- oversized malformed body仍優先穩定回413，不配置無界memory。
- malformed JSON回400且Content-Type為`application/problem+json`。
- 413/415 response為standard `ProblemDetails`，status/body一致。

### 8.5 Completion Criteria

- external create/update不再無界讀取body。
- 實際Content-Type決定是否接受request，不再hard-code JSON假設。
- 413及415均有handler-level regression tests。

---

## 9. P3-FR-06: SMF Redirect Semantics And Location Are Not Preserved

### 9.1 Expected Behavior

Release 18 Nsmf Event Exposure individual resource GET/PUT/DELETE宣告307及308。Go作為standard communication
owner時應把peer response的：

- status；
- Location；
- representation或`ProblemDetails`；

保留給PyAnLF。Go不能在PyAnLF不知情的情況下自動改變target，也不能在error mapping時丟失Location。

### 9.2 Current Behavior

`NsmfService.ExecuteStandardRequest()`使用的`http.Client`沒有`CheckRedirect`，因此Go預設會自動follow
307/308。若redirect成功，PyAnLF只看到最終response，無法知道resource target已改變。

即使test client禁止automatic redirect，使consumer取得307/308：

- `StandardSmfResponse`確實保存Location；
- 307/308同時被轉成`StandardSmfError`；
- `writeSmfEventExposureResponse()`只有在`err == nil`時才複製response Location；
- standard error path只輸出`ProblemDetails`，Location被丟棄。

因此目前存在兩個failure layer：automatic follow及handler error path header loss。

### 9.3 Remediation Design

1. standard SMF proxy request必須停用automatic redirect。為避免無意改變其他legacy client path，可在
   `ExecuteStandardRequest()`使用共享transport的shallow client copy並設定：

   ```text
   CheckRedirect -> http.ErrUseLastResponse
   ```

   若整個`NsmfService`已只剩standard proxy production caller，也可在constructor統一設定，但需先確認actual
   callers。
2. `writeSmfEventExposureResponse()`只要`response != nil`，就先保存其Location；不能以`err == nil`作header gate。
3. 對307/308直接回原status與Location。若body存在，保留相容的body/content type；若無body則只回status。
4. `StandardSmfError`可增加`RedirectLocation()`或明確redirect response type，沿用NRF proxy已存在的
   redirect-location pattern；不要以generic 502取代。
5. target-aware resource state不因Go自動follow而靜默改變。是否由PyAnLF採用新Location、重新選擇target或
   保持原binding，仍屬AnLF collection decision。
6. create operation的OpenAPI response matrix與individual GET/PUT/DELETE不同；tests需按method/operation宣告的
   allowlist驗證，不要把所有SMF methods一律視為具有相同redirect contract。

### 9.4 Required Tests

- mock SMF對GET/PUT/DELETE回307：Go不follow，PyAnLF收到307及原Location。
- 同上308。
- redirect有`ProblemDetails` body時，status、Location及standard fields都保留。
- redirect無body時不生成錯誤的502或空JSON syntax error。
- normal 200/204/404 behavior不改變。
- redirect target與原target不同時，Go route mirror不被automatic follow偷偷改寫。
- test明確區分create與individual-resource operation的OpenAPI status allowlist。

### 9.5 Completion Criteria

- standard SMF proxy不自動follow 307/308。
- 任何有standard Location的redirect response都能傳回PyAnLF。
- redirect handling不改變AnLF backend的target ownership。

---

## 10. Cross-finding Remediation Sequence

### Step 1: Lock Regression Tests Before Refactoring

先加入能穩定重現的focused tests：

- rejected sync zero-side-effect；
- partial acceptance revision stability；
- POST-success stale cleanup retry；
- external 413/415；
- SMF 307/308 Location。

thread/concurrency tests使用Event/barrier控制，不只依固定`sleep`。

### Step 2: Canonicalize Subscription State And Build Sync Prepare Plans

同批完成P3-FR-01與P3-FR-04：

- create/replace保存canonical accepted representation；
- sync prepare先完成所有subscription policy及peer restore validation；
- 建立single local commit boundary；
- commit後才enqueue collection work。

這兩項若分開修，容易先建立第二套normalization或再次修改`SubscriptionService.reconcile()`。

### Step 3: Close Peer Create And Cleanup Ownership

完成P3-FR-03：

- 201後立即建立active或pending-cleanup ownership；
- compensation failure進入既有bounded executor；
- Go 201後local processing failure也不能log-only leak；
- target-aware association mirror與restart restore保持一致。

### Step 4: Remove Requested SMF Expiry

- 移除request expiry與dead duration config。
- 保存SMF accepted representation。
- SMF主動回覆expiry時進入明確unsupported finite-lease cleanup/observability path，不建立半套renewal。

### Step 5: Complete Standard HTTP Boundaries

同批完成P3-FR-05與P3-FR-06：

- external NWDAF request size/media validation；
- SMF client停用automatic redirect；
- redirect status/Location/body pass-through；
- 以Release 18 operation-specific matrix固定handler/consumer tests。

### Step 6: Full Verification And Focused Re-review

- 執行NWDAF lint、all tests、race及build。
- 執行PyAnLF/PyMTLF lint及all tests。
- 可行時執行Go + PyAnLF + mock/real SMF cross-process scenarios。
- 逐項對照本文件completion criteria及第一輪remediation final gate。
- 再做一次focused review，確認沒有新的P0/P1 finding。

---

## 11. Verification Matrix After Remediation

### 11.1 NWDAF

```bash
make lint
go test ./...
go test -race ./internal/backend ./internal/context ./internal/anlf/... ./internal/sbi/...
make build
```

需新增或更新：

- external Events Subscription body limit/media type handler tests；
- SMF 307/308 no-follow及Location pass-through tests；
- SMF 201後local route/parse failure cleanup tests；
- accepted subscription snapshot round-trip tests。

### 11.2 PyAnLF

```bash
uv run ruff check .
uv run pytest -q
```

需新增focused suites：

- atomic sync rejection matrix；
- sync與external CRUD serialization；
- canonical partial acceptance round-trip；
- provisional peer/create compensation retry；
- selected expiry policy；
- restart後pending cleanup/renewal restore。

### 11.3 PyMTLF

```bash
uv run ruff check .
uv run pytest -q
```

本輪沒有確認PyMTLF-specific regression。若atomic sync shared model或wire schema沒有改變，PyMTLF只需跑完整
regression；若為transaction結果修改shared sync contract，需同步更新contract tests，仍不得讓AnLF/MTLF形成
不同handshake envelope。

### 11.4 Cross-process Scenarios

環境允許時至少驗證：

1. Go持有active resource，PyAnLF restart後atomic sync成功並開始background convergence。
2. invalid snapshot被拒絕後，再送原合法snapshot仍為no-op且state未被污染。
3. partial-accepted resource在create後立即refresh，不重建runtime。
4. SMF POST in flight期間consumer PUT/DELETE，compensating DELETE暫時失敗後仍完成retry。
5. selected expiry policy在超過一小時或縮短測試期限後仍持續collection。
6. SMF PUT/DELETE回307/308時，PyAnLF收到Location且Go沒有自動follow。
7. oversized或錯誤Content-Type external request在到達PyAnLF前由Go依standard拒絕。

real NRF/SMF/UPF/ADRF/Mongo未執行時，交付紀錄需逐項標示unit、mock cross-process及real integration層級，
不得把unit pass稱為end-to-end validation。

---

## 12. Scope And Decision Boundaries

本remediation不包含：

- THRESHOLD或ONE_TIME analytics擴張。
- AoI SMF discovery或automatic SMF migration。
- UDM Group ID resolution。
- MTLF dataset retrieval、training或model provision。
- Go process durable persistence。
- TLS/OAuth delegation。
- distributed transaction、message broker或跨process CAS protocol。

目前finding均已有實作方向，不再有未決high-level decision。它們可在既有Phase 3 ownership與contract內修正，
不需改變Go/PyAnLF責任邊界。

實作時若發現下列情況，需先更新計畫並要求決策：

1. atomic sync必須改變external resource acceptance或Go/PyAnLF authority，而不只是local transaction。
2. runtime manager無法提供atomic restore，且唯一方案是持久化或distributed rollback。
3. SMF accepted response無法用目前model無損保存，需新增generated model或改wire schema。
4. redirect handling需要Go自行選擇新SMF target，而不是pass-through給PyAnLF。
5. request body limit需大於Go↔PyAnLF private transport limit，導致兩個boundary不一致。

---

## 13. Final Acceptance Gate

Phase 3 follow-up review只有在以下條件全部成立時才可標記resolved：

1. P3-FR-01至P3-FR-06都有production fix及regression test。
2. rejected sync對projection、runtime、subscriptions、peer resources及queues均為zero-side-effect。
3. partial acceptance的create/replace response、PyAnLF stored resource與Go snapshot為同一canonical representation。
4. PyAnLF不送出SMF expiry、移除dead duration config，且SMF主動回覆expiry時不建立虛假的永久active state。
5. SMF 201後任何stale/local failure都保留active resource或target-aware cleanup intent。
6. external NWDAF POST/PUT具body limit、actual media validation及413/415 tests。
7. SMF individual-resource 307/308不被Go自動follow，且Location完整傳回PyAnLF。
8. 第一輪P3-CR-01至P3-CR-06 regression tests仍全部通過。
9. NWDAF lint、tests、race及build通過。
10. PyAnLF/PyMTLF lint及tests通過。
11. 未執行real integration checks有具體環境說明，不誤稱完整end-to-end validation。
12. focused re-review沒有新的P0/P1 finding。

## 14. Implementation Record

> Completion note: this section is a historical record of the first follow-up implementation and its local
> verification. The second follow-up review subsequently confirmed incomplete remediation and new regressions.
> Phase 3 remains open; use `Phase 3 Second Follow-up Code Review Findings And Remediation Plan.md` as the current
> remediation and acceptance source.

2026-07-22已依本文件完成第一版修正：

- PyAnLF加入sync prepare/commit transaction boundary；所有subscription、SMF resource及cross-reference
  validation在local mutation前完成，commit後才排入collection convergence。sync與external Events Subscription
  CRUD共用service transaction lock。
- PyAnLF create/replace/sync改用canonical accepted Events Subscription representation，`failEventReports`不再
  造成首次full sync的虛假runtime revision。
- PyAnLF移除SMF create request的`expiry`與`collection.subscription_duration_seconds`。SMF unsolicited finite
  lease會被標記為unsupported並進入target-aware cleanup/retry，不被視為永久active。
- PyAnLF在SMF POST成功後立即建立provisional resource ownership；stale revision、malformed response及暫時性
  DELETE failure都保留typed cleanup intent並使用既有bounded retry lane。
- Go external Events Subscription POST/PUT加入1 MiB `MaxBytesReader`與實際`Content-Type`解析，依contract回413/415。
- Go standard SMF proxy停用307/308 automatic follow；AnLF proxy保留redirect status、Location、body及peer
  Content-Type。Go在SMF 201後local representation/route failure時執行bounded compensating DELETE。
- 新增對應PyAnLF及NWDAF regression tests；沒有新增testdata JSON contract fixture。

本次已完成的本地驗證：

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

尚未執行real NRF、SMF、UPF、ADRF或MongoDB cross-process integration；因此本記錄只宣告unit、module、mock及
build層級通過，不能替代完整end-to-end驗證。完成real peer integration與最後一次focused re-review後，才可將
本文件Status改為`Resolved`。
