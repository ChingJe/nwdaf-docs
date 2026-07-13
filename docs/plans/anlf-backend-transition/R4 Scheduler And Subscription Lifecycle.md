# R4 Scheduler And Subscription Lifecycle

Date: 2026-07-14

Status: Repository-level implementation and R5 cross-process lifecycle verified; full 5GC environment E2E unverified

Parent plan:

- `Behavioral Parity Remediation Plan.md`

Audit and decision record:

- `Behavioral Parity Audit.md`

Paired compatibility framework:

- `R0 Compatibility Oracle Framework.md`

Affected implementation repositories:

1. `PyAnLF/`
2. `NWDAF/`

---

## 1. Purpose

R4補完report scheduler自然結束後的subscription lifecycle。Current PyAnLF scheduler在
`maxReportNbr`或`monDur`到期時只會結束thread，沒有release local runtime，也不會通知Go；Go端因此
可能繼續把subscription視為active。Scheduler遇到非inference generation exception時也會永久退出，
但兩端都沒有明確的failed或completed state transition。

本stage不撤銷PyAnLF對report scheduling、model runtime、accuracy monitor與observation binding的
ownership。目標是讓PyAnLF先完成local runtime cleanup，再以一條private internal callback讓Go安全、
idempotent地把對應subscription revision設為inactive。

R4完成後應建立以下保證：

1. `maxReportNbr`與`monDur`自然到期會release PyAnLF local runtime
2. Go最終會收到stable completion event並將同revision subscription設為inactive
3. Callback timeout、transport error或non-`204` response不會靜默遺失completion
4. Old completion不會關閉newer subscription runtime revision
5. Explicit delete、update replacement與shutdown不會誤送normal completion
6. Transient report generation error不會永久終止scheduler
7. 已沒有active backend consumer的source不再持續enqueue至PyAnLF
8. SMF/UPF collection與Go traffic/ADRF storage維持本輪既定boundary

---

## 2. Goals

1. 建立Go與PyAnLF之間的runtime completion private contract
2. 將normal completion、explicit cancellation、stale runtime與transient error分成明確結果
3. 在PyAnLF建立manager-owned completion finalization與process-local tombstone delivery
4. 在Go建立API -> processor -> coordinator -> context的completion handling flow
5. 以runtime revision與subscription state提供idempotency，不把correctness只綁在completion ID cache
6. 維持historical report-count與completion-condition語意
7. 保留current `immRep`修正行為
8. 加入R0 Reporting slice的historical provenance、failure evidence與compatibility tests
9. 補齊normal completion、retry、update race、shutdown與observation forwarding tests

---

## 3. Scope

### 3.1 Included

#### PyAnLF

1. `ApplySubscriptionRuntimeRequest`新增required `runtime_completion_callback_uri`
2. 新增`RuntimeCompletionEvent`與completion reason enum
3. Scheduler回報typed normal completion result
4. Scheduler transient generation/send exception recovery
5. Scheduler cancellation與stale runtime不產生completion
6. Manager-owned completion signal queue與finalizer worker
7. Local runtime、model、accuracy與observation binding cleanup
8. Process-local completion tombstone registry與retry worker
9. Dedicated `runtime_completion_delivery` config
10. Process-local per-subscription runtime revision high-water mark
11. Completion delivery stats與structured logs
12. Scheduler、runtime manager、delivery與HTTP contract tests

#### NWDAF

1. `ApplySubscriptionRuntimeRequest`新增required completion callback URI
2. Coordinator依AnLF server advertised URI建立callback URI
3. `anlf.Server`新增runtime completion route與API handler
4. `anlf/processor`新增completion procedure boundary
5. `anlf/coordinator`新增revision-aware completion handling
6. `context.Subscription`新增atomic completion transition
7. Missing、duplicate、stale與future revision的明確HTTP semantics
8. UPF observation enqueue前的active backend consumer gate
9. Handler、processor/coordinator、context、producer與race tests
10. Optional local live callback contract test

#### Documentation And Verification

1. R0 Reporting slice provenance與initial failure evidence
2. Cross-repository contract matrix
3. Exact verification commands與environment-only gaps
4. Parent plan、audit與R0 status更新

### 3.2 Excluded

1. SMF/UPF unsubscribe或collection reference cleanup
2. Go traffic storage與ADRF buffer cleanup
3. Single global observation delivery worker重構
4. Per-source workers、durable observation outbox或backfill
5. Accuracy callback改成outbox；accuracy仍維持finite retry後drop
6. Completion tombstone跨PyAnLF restart持久化
7. Persistent runtime revision store
8. Runtime-failed callback或unrecoverable failure state machine
9. One-shot `Nnwdaf_AnalyticsInfo`
10. THRESHOLD或ON_EVENT_DETECTION scheduler
11. PyMTLF backend migration
12. Standardized 3GPP SBI schema變更
13. Consumer-facing subscription DELETE或notification contract變更
14. `immRep`回退成historical unconditional immediate report

若實作需要上述項目才能安全完成，必須先依development policy提出blocker與replan，不得在R4中
靜默擴張scope。

---

## 4. Planning Basis

### 4.1 Required Local Sources

實作與review依以下順序使用：

1. Current `PyAnLF/src/py_anlf/core/reporting.py`
2. Current `PyAnLF/src/py_anlf/core/runtime_manager.py`
3. Current `NWDAF/internal/anlf/`、`internal/context/`與`internal/sbi/processor/`
4. `NWDAF@0db9584:internal/notifier/notifier.go`
5. `NWDAF@0db9584:internal/notifier/notifier_test.go`
6. `NWDAF@0db9584:internal/sbi/processor/eventssubscription.go`
7. `NWDAF@62e2f9f:internal/sbi/processor/eventssubscription.go`
8. Local TS 29.520與`TS29520_Nnwdaf_EventsSubscription.yaml`
9. `nwdaf-docs/docs/development_policy.md`
10. `free5gc-dev-skill/SKILL.md`
11. `free5gc-dev-skill/references/config-and-lifecycle.md`
12. `free5gc-dev-skill/references/concurrency-lifecycle.md`
13. `free5gc-dev-skill/references/sbi-development.md`
14. `free5gc-dev-skill/references/testing.md`
15. Local free5GC PCF callback boundary作為Go HTTP edge shape參考

PyAnLF不是free5GC NF implementation target。Scheduler與runtime semantics以historical NWDAF behavior
和目前PyAnLF ownership為主要依據；PCF只用於確認callback route、handler與processor分層外型，不作為
private completion contract的domain oracle。

### 4.2 Historical Oracle

`NWDAF@0db9584`證明以下行為：

1. `maxReportNbr == 0`代表unlimited
2. report count在每次send attempt前increment
3. `reportCount >= maxReportNbr`時停止
4. `monDur`到期時停止
5. completion callback把subscription設為inactive
6. explicit scheduler cancellation不建立normal completion
7. completion callback由scheduler非同步觸發，避免blocking report loop
8. 舊callback本身不清理SMF/UPF collection

`NWDAF@62e2f9f`在上述基礎上增加AnLF runtime release。R4保留這個責任結果，但因runtime已搬到
PyAnLF，release順序改為PyAnLF先完成local cleanup，再通知Go inactive；Go callback不得反向呼叫
PyAnLF DELETE。

Historical Go scheduler沒有正確使用`immRep`，會無條件立即送第一份report。Current PyAnLF已明確
修正為：

1. `immRep=true`立即嘗試第一份report
2. `immRep=false`先等待一個`repPeriod`

這是approved behavior change，不納入parity rollback。

### 4.3 Specification Boundary

TS 29.520的`evtReq`包含`maxReportNbr`、`monDur`、`repPeriod`、`immRep`等consumer-facing reporting
control。R4只實作這些欄位在既有PERIODIC subscription中的lifecycle結果。

`RuntimeCompletionEvent`與`runtime_completion_callback_uri`是NWDAF Go與PyAnLF之間的private
implementation contract，不是3GPP Nnwdaf service schema，不得加入free5GC generated OpenAPI models。

### 4.4 Free5GC Shape Evidence

Go callback edge維持目前NWDAF AnLF server的外型：

```text
anlf.Server route
    -> api_runtime_completion.go
    -> anlf/processor
    -> anlf/coordinator
    -> internal/context
```

Local free5GC PCF的`internal/sbi/server.go`與`api_httpcallback.go`證明callback URI可由server route接收，
handler負責HTTP parsing/response、processor負責procedure dispatch。R4不把private callback放進SBI
server，也不讓handler直接修改global context。

---

## 5. Change Classification

R4是cross-repository private contract change、subscription lifecycle change與concurrency change。

它不是：

1. 3GPP external API feature
2. free5GC SBI service addition
3. analytics或accuracy algorithm migration
4. model provision lifecycle extension
5. observation transport redesign

因此implementation必須同時滿足：

1. Go保持既有API -> processor -> coordinator分層
2. PyAnLF lifecycle worker有明確owner、stop condition與shutdown順序
3. Contract在兩個repository同步cut over
4. 每個repository獨立commit，commit pair作為部署單位
5. 不以background thread掩蓋缺少的state transition

---

## 6. Current State And Confirmed Problems

### 6.1 Normal Scheduler Exit Leaves Runtime Active

Current `ReportingScheduler._run()`在以下條件直接`return`：

1. current time到達`monitoring_end`
2. `sequence >= max_reports`

Runtime仍留在`SubscriptionRuntimeManager._subscriptions`，model reference、accuracy membership與
observation bindings也未release。Go subscription保持active。

### 6.2 Generation Exception Permanently Stops Reporting

Current scheduler捕捉`generate()` exception後log並直接`return`。Inference exception已有
confidence-zero fallback，但timestamp parsing、runtime snapshot或其他transient exception仍可能永久終止
thread。這種錯誤不應被誤判為normal completion。

### 6.3 Scheduler-thread Cleanup Can Deadlock With Update

Current update/apply path會在per-subscription lock內呼叫`previous.scheduler.stop()`；`stop()`會join scheduler
thread。如果scheduler thread在normal completion callback中同步取得同一subscription lock做release，會形成：

```text
update thread: holds subscription lock -> stop() -> join scheduler
scheduler thread: completion callback -> waits subscription lock
```

因此scheduler thread只能投遞completion signal，不得同步執行需要subscription lock的runtime cleanup，
也不得等待completion HTTP callback。

### 6.4 Runtime Revision Can Be Reused After Natural Completion

Current `_next_revision(previous)`在`previous is None`時回傳`1`。R4自然完成會移除runtime；若consumer之後
更新同一Go subscription ID，PyAnLF可能再次配置revision `1`。尚未ack的old revision-1 completion會因此
誤關閉new revision-1 runtime。

R4必須保留process-local per-subscription revision high-water mark。只要PyAnLF process仍存活，同一
subscription ID的新runtime revision必須strictly increase，即使previous runtime已完成或explicitly release。

### 6.5 Completion Delivery Needs Stronger Guarantee Than Reports

Analytics與accuracy report可依既定policy有限retry後drop；subscription completion若drop，Go會永久保留
錯誤active state。因此completion必須使用process-local tombstone持續retry至Go回`204`或PyAnLF shutdown。

### 6.6 Post-completion Observations Continue To Backend

依approved boundary，completion不會刪除SMF/UPF subscription。Current UPF notification仍會無條件
enqueue observation batch至PyAnLF；local runtime/binding已release後，這會持續造成backend `404`與retry。

Producer必須在enqueue前確認該correlation source至少仍有一個active NWDAF subscription。Traffic storage
與ADRF forwarding不受此gate影響。

---

## 7. Confirmed Decisions

Items 1至13直接承接parent plan與audit的confirmed decisions。Items 14至17原為根據historical check
order與current concurrency/state structure推導出的implementation safety rules，已於2026-07-14確認採用。
若後續要改變其中任一項，必須在implementation前先修改本計畫與對應tests。

1. Normal completion只由`maxReportNbr`與`monDur`建立
2. Explicit DELETE、update replacement、rollback、stale runtime與shutdown不建立completion event
3. PyAnLF先完成local runtime cleanup，再callback Go
4. Go只將matching revision subscription設為inactive，不呼叫PyAnLF DELETE
5. 本輪不清理SMF/UPF collection references、traffic storage或ADRF state
6. Completion delivery使用process-local tombstone，持續retry至`204`或shutdown
7. Tombstone不跨restart保存
8. Accuracy與analytics report delivery policy不因R4改變
9. Single global observation delivery worker保持不變
10. Source沒有active consumer時停止backend enqueue；shared source仍有active consumer時繼續
11. `immRep`保留current behavior，不恢復historical unconditional immediate report
12. Report sequence/count保留increment-before-generation/send-attempt語意
13. Transient generation exception消耗該次sequence，log後於下一個`repPeriod`繼續
14. 同一boundary同時滿足max report與monitoring duration時，以historical check order讓
    `MAX_REPORTS_REACHED`優先
15. Runtime revision high-water mark只保證process lifetime；restart不保證monotonicity
16. Completion ID只作event identity與observability；正確性必須由subscription state/revision判斷
17. Go自然完成callback不刪除`MlModelInfo`，避免與subscription update重疊時破壞新runtime apply；這項
    local metadata cleanup與完整collection cleanup一起保留為future lifecycle work

Confirmed refinement（2026-07-14）：`monDur`維持historical scheduler tick判斷，不新增獨立expiry
timer；自然完成後保留Go `MlModelInfo`。Runtime revision high-water、required completion callback URI、
attempt-based sequence、manager-owned finalization與retry-until-`204`等安全規則一併依本計畫採用。

---

## 8. Internal Contract Cutover

### 8.1 Apply Runtime Request

Go與Python contract同步改為：

```text
ApplySubscriptionRuntimeRequest {
  subscription
  provision_context
  report_callback_uri
  runtime_completion_callback_uri
}
```

`runtime_completion_callback_uri`為required non-empty internal field。Go coordinator依AnLF server advertised
URI與subscription ID建立：

```text
{anlfServerURI}/subscriptions/{subscriptionId}/runtime-completions
```

PyAnLF不得由`report_callback_uri`字串替換或推導completion endpoint。Apply contract缺少或空白URI時，
request必須在HTTP/model boundary被拒絕；不保留old-Go fallback。

### 8.2 Completion Event

```text
RuntimeCompletionEvent {
  completion_id: string
  subscription_id: string
  runtime_revision: integer > 0
  reason: RuntimeCompletionReason
  completed_at: RFC3339 DateTime
  last_report_sequence: integer >= 0
}
```

Allowed reasons：

```text
MAX_REPORTS_REACHED
MONITORING_DURATION_EXPIRED
```

Rules：

1. `completion_id`在local completion commit時生成一次UUID
2. 同一tombstone所有retry重用完全相同payload與ID
3. `subscription_id`必須與path一致
4. `runtime_revision`來自完成的PyAnLF runtime
5. `completed_at`使用UTC aware timestamp
6. `last_report_sequence`表示已開始的最後一次report attempt；沒有attempt時為`0`
7. 不加入`FAILED`、`CANCELED`或`RELEASED` reason

### 8.3 Go Route

```text
POST /subscriptions/{subscriptionId}/runtime-completions
```

Response matrix：

| Condition | Status | Semantics |
| --- | --- | --- |
| Valid current revision, active | `204` | Atomically set inactive |
| Same revision already inactive | `204` | Idempotent duplicate/consumed |
| Missing/deleted subscription | `204` | Already absent; stop retry |
| Event revision older than current | `204` | Stale event consumed; do not touch new runtime |
| Event revision newer than current | `409` | Go state not yet converged; retry later |
| Malformed JSON | `400` | Contract error |
| Path/body subscription mismatch | `400` | Contract error |
| Invalid ID/revision/reason/time/sequence | `400` | Contract error |

PyAnLF只把`204`視為ack。Timeout、transport error、`4xx`、`5xx`及其他status都保留tombstone。`409`
預期可在Go完成runtime state更新後收斂；`400`代表程式contract defect，需持續error log但不得busy-loop。

### 8.4 Contract Ownership

Go types放在`internal/anlf/contract/`；Python types放在`py_anlf.models`或一個不造成circular import的
private contract module。不得修改3GPP generated model或external Nnwdaf payload。

---

## 9. Scheduler State Machine

### 9.1 States

```text
CREATED
  -> RUNNING
  -> NORMAL_COMPLETION_PENDING
  -> STOPPED

RUNNING
  -> CANCELED
  -> STOPPED

RUNNING
  -> transient error
  -> WAITING_NEXT_TICK
  -> RUNNING
```

只有`NORMAL_COMPLETION_PENDING`產生completion signal。

### 9.2 Boundary Check

每次report attempt前依序檢查：

1. stop/cancel event
2. `max_reports > 0 && sequence >= max_reports`
3. `monitoring_end != nil && now >= monitoring_end`

若2與3同時成立，reason為`MAX_REPORTS_REACHED`。`max_reports == 0`維持unlimited。

R4不新增獨立monitoring-duration timer。`monDur`在initial wait或兩個periodic attempts之間到期時，於下一個
scheduler wake-up判斷，保留historical ticker boundary，最多延遲一個`repPeriod`。

### 9.3 Immediate Reporting

1. `immRep=true`：進入loop後立即執行boundary check與第一個attempt
2. `immRep=false`：先等待一個`repPeriod`，再執行boundary check
3. Initial wait期間cancel：直接停止，不產生completion
4. Initial wait期間`monDur`到期：wake-up後產生`MONITORING_DURATION_EXPIRED`，sequence為0

### 9.4 Sequence Semantics

1. Sequence從0開始
2. 每次allowed attempt先increment，再呼叫generate
3. Generate failure仍消耗sequence
4. Send failure仍消耗sequence
5. `maxReportNbr`限制attempt數，不只計算成功delivery數
6. Max limit在下一個scheduler boundary被觀察並產生completion

這保留historical increment-before-send與current PyAnLF scheduler behavior。

### 9.5 Error Classification

1. Inference error：沿用analytics confidence-zero fallback，不由scheduler改寫
2. `ReportingRuntimeStopped` typed signal：表示runtime已stale/released，直接停止且不completion
3. Other `generate()` exception：`logger.exception`後等待下一個`repPeriod`
4. Unexpected send callable exception：log後等待下一個`repPeriod`
5. HTTP sender本身回`False`：視為該attempt delivery失敗，但scheduler繼續
6. Repeated failure必須經stop-aware timed wait，不得busy-loop
7. Scheduler outer boundary若意外退出，必須留下error log

Runtime manager用一個scheduler-specific wrapper把`SubscriptionRuntimeNotFoundError`與
`StaleRuntimeRevisionError`轉為`ReportingRuntimeStopped`。Reporting module不得反向import runtime manager
exception，避免circular dependency。

### 9.6 Completion Signal

Scheduler normal completion只建立immutable signal：

```text
SchedulerCompletion {
  subscription_id
  runtime_revision
  reason
  last_report_sequence
}
```

Scheduler callback只把signal放入manager-owned unbounded queue後return，不取得subscription/state lock、
不release runtime、不join thread、不送HTTP。

---

## 10. PyAnLF Completion Architecture

### 10.1 Worker Ownership

`SubscriptionRuntimeManager`擁有：

1. one completion signal queue
2. one local completion finalizer worker
3. one `RuntimeCompletionDelivery` tombstone worker
4. one manager closing event

Workers在manager construction完成後啟動，於`shutdown()`中明確停止並join。不得建立per-completion detached
thread。

### 10.2 Local Finalization

Finalizer取得signal後：

1. 取得subscription stripe lock
2. 取得state lock
3. 重新查詢current runtime
4. 若runtime不存在、revision不同、scheduler已replacement或manager closing，discard signal且不建tombstone
5. 從`_subscriptions`移除matching runtime
6. 從shared model registry detach subscription並取得可能需unload的model ID
7. 清除runtime scheduler reference
8. 釋放state lock與subscription lock
9. 從accuracy monitor detach subscription
10. 從observation store release subscription bindings/source references
11. unload最後一個shared model reference
12. 建立stable completion event
13. 寫入delivery tombstone

Local cleanup failure必須逐項log並繼續可安全執行的cleanup。只有state commit成功的matching runtime可以建立
tombstone；stale signal不得通知Go關閉new revision。

### 10.3 Revision High-water Mark

Manager新增：

```text
_runtime_revision_high_water: dict[subscription_id, int]
```

Allocation rules：

1. 新runtime revision在subscription stripe lock與state lock保護下配置
2. `next = max(previous.revision, high_water.get(id, 0)) + 1`
3. Runtime被normal completion或explicit release後不刪除high-water entry
4. Failed candidate若回用previous runtime，不虛增active revision
5. 建立pending/failed-no-previous runtime時仍配置revision
6. Process shutdown後map消失；不宣稱restart monotonicity

這個map會隨process lifetime內曾見過的subscription IDs成長。R4接受此bounded-by-created-subscriptions成本；
持久化與GC另列future work。

### 10.4 Completion Tombstone

```text
CompletionTombstone {
  event
  callback_uri
  attempt_count
  next_attempt_at
}
```

Registry key使用`(subscription_id, runtime_revision)`。同key重複record不得覆寫既有event ID或payload。
Different revision各自保留，以支援old completion與new runtime並存的network delay。

### 10.5 Delivery Worker

Delivery worker：

1. 以condition variable等待新tombstone、next due time或shutdown
2. 每個due tombstone每輪只嘗試一次
3. HTTP request在registry lock外執行
4. 每次request有bounded timeout
5. `204`後在lock內確認event仍相同，再移除
6. 其他結果增加attempt count並設定`now + retry_interval`
7. 一輪處理所有due tombstones，避免單一永久失敗event完全阻塞其他completion
8. Shutdown停止新attempt、等待目前bounded request返回、記錄放棄數量並清除process-local state

Delivery worker可接受injectable HTTP post callable與clock/wait seam，讓unit tests不依賴real sleep或network。

### 10.6 Explicit Release And Replacement

Existing `release(subscription_id)`維持explicit path：

1. 取得subscription stripe/state lock並移除runtime
2. `scheduler.stop()`屬cancellation
3. release accuracy、observation與model state
4. 不建立completion tombstone

Apply replacement仍stop old scheduler。若old scheduler已enqueue normal completion signal：

1. replacement先取得subscription lock並安裝new revision，finalizer之後看到revision mismatch並discard；或
2. finalizer先移除old runtime並建立tombstone，replacement依high-water配置new revision；old callback到Go後
   會因old revision被視為stale或在update transition中已inactive

兩種interleaving都不得關閉new revision。

### 10.7 Shutdown Order

```text
1. set manager closing
2. explicitly release/stop active runtimes and schedulers
3. stop and join completion finalizer worker
4. stop completion delivery worker; abandon process-local tombstones with log
5. shutdown remaining accuracy registry resources
```

Closing後新scheduler signal只可被discard，不得建立會在shutdown中無限retry的tombstone。所有network wait受
`request_timeout`限制。

---

## 11. Locking And Concurrency Model

### 11.1 Lock Order

沿用R3順序：

```text
subscription stripe lock
  -> runtime manager state lock
    -> release both
      -> accuracy/observation/model cleanup
```

Completion tombstone registry lock獨立於runtime state lock。不得在以下操作時持有runtime state lock：

1. scheduler join
2. accuracy worker join
3. model unload I/O
4. completion HTTP request
5. condition wait

### 11.2 Required Race Outcomes

| Race | Required result |
| --- | --- |
| Normal completion vs explicit DELETE | At most one local cleanup; no callback required after explicit release wins |
| Normal completion vs update replacement | New revision remains active; old event is discarded or Go consumes it as stale |
| Duplicate scheduler signal | One local cleanup and at most one tombstone per revision |
| Callback retry vs Go update | Old revision gets `204` without closing new revision |
| Callback arrives before Go stores future revision | Go returns `409`; tombstone retries |
| Shutdown vs pending callback | Request bounded; tombstone abandoned only at process shutdown |
| Shared model completion vs active peer runtime | Model remains loaded for peer runtime |
| Shared source completion vs active peer subscription | Go continues one source-scoped enqueue |

### 11.3 No Recursive Release

Go completion handler不得呼叫`ReleaseSubscriptionRuntime()`。PyAnLF已先release local runtime；反向DELETE會
形成不必要的recursive lifecycle，並可能等待正在等待Go callback response的delivery path。

---

## 12. NWDAF Completion Handling

### 12.1 API Handler

新增`internal/anlf/api_runtime_completion.go`：

1. 註冊POST route
2. 取得path subscription ID
3. bind JSON
4. 驗證required identity、revision、reason、timestamp與sequence
5. 驗證path/body ID一致
6. 呼叫processor
7. success/stale/missing/duplicate回`204`
8. future revision回`409`
9. malformed/invalid回`400`
10. unexpected internal error回`500`

Handler不直接讀寫`internal/context`，不呼叫AnLF backend client。

### 12.2 Processor

`anlf/processor`新增narrow interface：

```text
runtimeCompletionWorkflow {
  CompleteSubscriptionRuntime(event) error
}
```

`Processor.HandleRuntimeCompletion`只做procedure dispatch與error normalization。Constructor wiring沿用目前
coordinator dependency，不建立第二個broad app dependency。

### 12.3 Coordinator

Coordinator procedure：

1. 取得NWDAF context
2. subscription不存在時log debug並success
3. 呼叫subscription atomic completion transition
4. current active -> inactive，log info
5. current inactive -> idempotent success
6. old revision -> stale success，log debug
7. future revision -> typed conflict error
8. 不呼叫backend DELETE
9. 不清理SMF/UPF collection
10. 不刪除Go `MlModelInfo`

### 12.4 Atomic Context Transition

`Subscription`新增：

```text
CompleteRuntime(revision) RuntimeCompletionDisposition
```

在單一`runtimeMu` critical section比較`RuntimeRevision`與`IsActive`並完成transition：

```text
revision < current -> STALE
revision > current -> FUTURE
revision == current && !active -> ALREADY_INACTIVE
revision == current && active -> set inactive; COMPLETED
```

不得以分開的`RuntimeSnapshot()`與`SetActive(false)`完成check-then-act，否則update可能在兩次lock之間安裝新
revision。

### 12.5 Update Race

Current update path先把old subscription設inactive，再向PyAnLF apply replacement，最後以new
`Subscription` object與new revision更新context。R4保持此順序。

Old completion可能：

1. 在update前將old revision設inactive；update仍可建立new active revision
2. 在update apply期間看到old revision已inactive並回`204`
3. 在new context commit後看到older revision並回`204`

因PyAnLF high-water防止revision reuse，old event在任何情況都不能match new runtime revision。

---

## 13. Observation Forwarding Gate

### 13.1 Context Query

在`internal/context`提供thread-safe query：

```text
HasActiveNwdafSubscriber(correlationID) bool
```

Implementation：

1. 取得`SmfSubscription`
2. 在SMF subscription lock內copy `NwdafSubIds`
3. release SMF lock
4. 對每個ID查詢NWDAF subscription並讀取atomic runtime snapshot
5. 任一subscription active即回true

不得在持有SMF subscription lock時取得subscription runtime lock，降低跨domain lock order風險。

### 13.2 Producer Placement

Gate只放在`HandleUpfNotification`建立observations後、呼叫`EnqueueObservations`前：

```text
store traffic data
update SMF last-seen
build source observations
if source has active NWDAF subscriber:
    enqueue PyAnLF
else:
    skip backend enqueue and debug log
forward ADRF data
```

不得提前return，否則會破壞traffic storage或ADRF behavior。

### 13.3 Race Semantics

Active check與enqueue不是transaction。Completion剛好發生在check後時，最多允許一個已通過gate或已在queue
中的batch繼續delivery；R4只保證completion收斂後不持續產生新backend enqueue，不宣稱清空既有queue。

---

## 14. Configuration

PyAnLF `config/config.yaml`新增：

```yaml
runtime_completion_delivery:
  request_timeout: 5
  retry_interval: 1
```

Rules：

1. `request_timeout > 0`
2. `retry_interval > 0`
3. 沒有`max_retries`
4. Omitted section使用上述defaults
5. 不共用`report_delivery`或`accuracy_monitor.report_delivery`
6. Invalid explicit value在runtime manager construction時fail fast

NWDAF不新增completion target config。Callback URI由既有`anlf.server.registerIPv4`與port組合；這符合目前
analytics report callback URI的建構方式。

---

## 15. Planned Code Changes

### 15.1 PyAnLF

Expected files：

1. `src/py_anlf/models.py`
   - apply request callback URI
   - completion event/reason models
2. `src/py_anlf/core/reporting.py`
   - scheduler outcome、typed stop、recovery與completion signal
3. `src/py_anlf/core/runtime_completion.py`（new）
   - tombstone registry、HTTP delivery worker與stats
4. `src/py_anlf/core/runtime_manager.py`
   - completion queue/finalizer、revision high-water、local cleanup與shutdown
5. `config/config.yaml`
   - dedicated delivery config
6. `tests/test_reporting.py`或既有scheduler test file
7. `tests/test_runtime_completion.py`（new）
8. `tests/test_runtime_manager.py`
9. `tests/test_lifecycle_api.py`
10. R0 reporting fixtures/test loader under existing compatibility test layout

實作時可依目前test organization微調檔名，但不得把delivery、scheduler與runtime finalization重新混成一個
無法獨立測試的module。

### 15.2 NWDAF

Expected files：

1. `internal/anlf/contract/runtime.go`
   - apply callback URI
2. `internal/anlf/contract/runtime_completion.go`（new）
   - event、reason與validation
3. `internal/anlf/api_runtime_completion.go`（new）
4. `internal/anlf/server.go`
   - processor API與route registration
5. `internal/anlf/processor/processor.go`
6. `internal/anlf/processor/runtime_completion.go`（new）
7. `internal/anlf/coordinator/coordinator.go`
8. `internal/anlf/coordinator/subscription_runtime.go`
   - callback URI builder與completion procedure
9. `internal/context/context.go`
   - atomic completion transition
10. `internal/context/traffic_data.go`
    - active subscriber query
11. `internal/sbi/processor/upf_notify.go`
    - enqueue gate
12. Corresponding unit、handler、coordinator、producer與live tests
13. Existing apply request fixtures/mocks更新required field

R4不新增`routes.go`；AnLF server沿用目前每個`api_*.go`提供route list的方法，保持與既有SBI/AnLF外型一致。

---

## 16. Implementation Sequence

### Step 1: Establish R0 Reporting Failure Evidence

1. 從historical Go source/tests手工建立scheduler completion fixtures
2. Fixture metadata記錄commit、source file、symbol、invariant與approved deviation
3. 在current PyAnLF確認max/monDur只停thread、不release、不callback
4. 確認generation exception永久停止
5. 保存focused failing test名稱與failure摘要到implementation record
6. 不commit red tests；正式implementation commit保持green

### Step 2: Add Go Completion Receiver

1. 新增Go completion contract
2. 新增context atomic transition
3. 新增coordinator與processor procedure
4. 新增AnLF route/handler
5. 完成missing、duplicate、stale、future與validation tests

先建立receiver可讓後續PyAnLF callback有明確target；尚未部署新PyAnLF前route不影響existing behavior。

### Step 3: Cut Over Apply Runtime Contract

1. Go apply request加入completion callback URI
2. Coordinator建立URI
3. Python apply request同步required field
4. 更新所有Go/Python fixtures與HTTP tests
5. 不保留optional compatibility fallback

### Step 4: Refactor Scheduler Outcome

1. 定義completion reason與typed stopped signal
2. 保留`immRep`與sequence semantics
3. 正常條件emit signal
4. Transient exception下一tick恢復
5. Explicit stop/stale不emit
6. 以injected clock/wait seam完成deterministic tests

### Step 5: Implement Local Finalization And Revision High-water

1. 加入manager-owned completion queue/worker
2. 加入high-water revision allocation
3. 完成revision-aware local finalization
4. 避免scheduler-thread subscription lock與recursive release
5. 測試shared model、shared source、replacement、delete與shutdown interleavings

### Step 6: Implement Tombstone Delivery

1. 加入dedicated config與validation
2. 建立stable event/tombstone registry
3. 實作retry-until-204 worker
4. 加入stats、logs與shutdown behavior
5. 測試timeout、5xx、400、409、duplicate與event stability

### Step 7: Gate Post-completion Observation Forwarding

1. 建立thread-safe active subscriber query
2. 在UPF producer enqueue boundary套用
3. 保持traffic storage與ADRF path不變
4. 測試inactive-only、shared active與check/enqueue race boundary

### Step 8: Cross-repository Verification And Review

1. Go/Python contract serialization tests
2. Optional real PyAnLF -> Go callback test
3. Focused race tests
4. Full test/build/lint
5. 完整code review
6. 更新parent plan、audit、R0與本文件implementation record

---

## 17. Test Plan

### 17.1 R0 Reporting Compatibility

至少建立：

1. max report not reached
2. max report reached
3. max report zero unlimited
4. monitoring duration future
5. monitoring duration expired
6. both limits exceeded且max reason優先
7. sequence increment-before-failed-generation
8. explicit cancellation no completion
9. historical completion sets inactive（Go-side oracle）
10. approved `immRep=false` deviation

### 17.2 PyAnLF Scheduler

1. `max_reports=1`只嘗試一份report並emit correct reason/sequence
2. monitoring duration expiry emit correct reason
3. expiry before first non-immediate report produces sequence 0
4. both conditions reason precedence
5. explicit stop no signal
6. typed stale runtime stop no signal
7. one transient generate exception後next tick繼續
8. send callable exception後next tick繼續
9. failed attempt仍計入max report sequence
10. repeated exception不busy-loop

Tests不得依賴長時間`time.sleep`；使用injectable clock、event或短controlled wait。

### 17.3 PyAnLF Runtime Finalization

1. max report completion移除runtime
2. monDur completion移除runtime
3. accuracy membership detached
4. observation binding/source references released
5. last shared model subscriber才unload
6. duplicate signal只cleanup一次
7. stale revision signal不影響new runtime
8. explicit release不建tombstone
9. update replacement不deadlock
10. natural completion後same subscription apply取得higher revision
11. release後same subscription apply取得higher revision
12. shutdown停止workers且不產生completion

### 17.4 PyAnLF Tombstone Delivery

1. immediate `204` ack removes tombstone
2. timeout後保留並重試相同event
3. `500`後重試至`204`
4. `409`後重試至`204`
5. `400`保留且依interval重試
6. retry payload與completion ID stable
7. duplicate record不覆寫event
8. multiple due tombstones都能取得attempt
9. request在registry lock外執行
10. shutdown停止retry並記錄abandoned count
11. invalid timeout/interval fail fast

### 17.5 NWDAF Contract And Handler

1. valid current event -> `204`
2. malformed JSON -> `400`
3. path/body mismatch -> `400`
4. empty completion ID -> `400`
5. invalid revision/reason/time/sequence -> `400`
6. missing subscription -> `204`
7. future revision -> `409`
8. route registration與processor dependency wiring

### 17.6 NWDAF State And Race

1. current active revision atomically becomes inactive
2. current inactive duplicate stays inactive
3. old revision does not close new revision
4. future revision does not mutate state
5. concurrent update/completion has no race
6. callback does not call backend DELETE
7. callback does not clean up SMF/UPF references
8. callback does not delete `MlModelInfo`
9. `go test -race` covers context/coordinator/processor affected packages

### 17.7 Observation Forwarding

1. inactive-only source still stores traffic data
2. inactive-only source still forwards ADRF data
3. inactive-only source does not enqueue backend observations
4. shared source with one active subscription still enqueues once
5. missing SMF source mapping skips backend enqueue safely
6. active subscriber query copies IDs before subscription state reads

### 17.8 Cross-repository

1. Real PyAnLF scheduler completion -> real Go AnLF server -> `204`
2. First callback `5xx`或timeout，retry後成功
3. Go update與old completion race
4. Apply request包含兩個不同用途的callback URIs
5. Old Go/new Py或new Go/old Py不作長期compatibility guarantee；deployment使用commit pair

若local environment不適合啟動兩個process，1至3可標示environment-level unverified，但必須以real HTTP
handler/component tests覆蓋serialization、status mapping與retry，不能只測pure functions。

---

## 18. Verification Commands

### 18.1 PyAnLF

```bash
uv run pytest -q tests/test_reporting.py
uv run pytest -q tests/test_runtime_completion.py
uv run pytest -q tests/test_runtime_manager.py
uv run pytest -q tests/test_lifecycle_api.py
uv run pytest -q
```

實際檔名依implementation決定；若scheduler tests整合進既有檔案，progress record需列出正確commands。

### 18.2 NWDAF

```bash
go test ./internal/anlf/...
go test ./internal/context/...
go test ./internal/sbi/processor/...
go test -race ./internal/anlf/... ./internal/context/... ./internal/sbi/processor/...
make test
make build
make lint
```

### 18.3 Repository Hygiene

```bash
git diff --check
git status --short
```

### 18.4 Environment-level Optional Check

若環境允許：

1. 啟動Go NWDAF AnLF server
2. 啟動PyAnLF並指向該server
3. 建立短`repPeriod`、`maxReportNbr=1`subscription
4. 驗證PyAnLF runtime消失、Go subscription inactive、completion tombstone被ack
5. 注入first-attempt failure驗證retry

不得把unit/component pass描述成完整running-process E2E。

---

## 19. Observability

### 19.1 PyAnLF Logs

至少包含：

1. Scheduler normal completion：subscription、revision、reason、last sequence
2. Scheduler transient failure：subscription、revision、sequence、exception
3. Stale/canceled scheduler stop：debug level
4. Local completion committed：subscription、revision、reason
5. Tombstone attempt：completion ID、subscription、revision、attempt、status/error
6. Tombstone acknowledged：completion ID與attempt count
7. Contract error retry：`400` status與bounded-frequency error log
8. Shutdown abandoned tombstones count
9. Stale completion signal discarded

不得log完整subscription payload、model artifact或subscriber-sensitive observation內容。

### 19.2 PyAnLF Process-local Stats

至少提供testable snapshot：

```text
completion_created_total
completion_acknowledged_total
completion_retry_total
completion_contract_error_total
completion_stale_signal_total
completion_abandoned_total
completion_tombstone_count
```

本stage不要求Prometheus endpoint；stats可沿用R3 process-local counters外型。

### 19.3 NWDAF Logs

1. Current completion applied：info
2. Duplicate/already inactive：debug
3. Old revision consumed：debug
4. Future revision conflict：error或warning
5. Malformed/path mismatch：handler response，不重複多層log
6. Inactive-only source enqueue skipped：debug

---

## 20. Risks And Controls

### 20.1 Completion/update Deadlock

Risk：scheduler callback同步取得subscription lock，而update持鎖join scheduler。

Control：scheduler只enqueue immutable signal；manager finalizer在獨立worker取lock。加入update/completion timeout與race
test。

### 20.2 Old Completion Closes New Runtime

Risk：natural completion後revision reset或Go check-then-act race。

Control：PyAnLF process-local high-water + Go atomic revision transition + old revision回`204`。

### 20.3 Permanent Tombstone On Contract Error

Risk：`400`永遠不會自行收斂，tombstone持續retry。

Control：positive retry interval、bounded request timeout、explicit contract-error counter/log、process shutdown boundary。
R4不finite-drop lifecycle completion。

### 20.4 Process Crash Between Local Cleanup And Ack

Risk：PyAnLF crash會丟失process-local tombstone，Go可能維持active。

Control：已由Decision 9接受；文件與logs不得宣稱restart durability。Durable outbox另列future work。

### 20.5 Revision High-water Growth

Risk：map保留process lifetime內所有subscription IDs。

Control：ID數量受process lifetime建立subscription數量界定；先以正確性優先。Persistent/GC policy另開工作，
不得在R4加入會造成revision reuse的eviction。

### 20.6 Completion Delivery Head-of-line Delay

Risk：single worker的slow request可延遲其他events一個request timeout。

Control：每個request bounded、每輪嘗試所有due tombstones、不在單一event內部無限loop。若實驗顯示completion
量超過此模型，再另開bounded concurrency work。

### 20.7 Residual Queued Observations

Risk：completion前已通過gate或已queued batch仍可能送到已release backend source。

Control：接受最多有限in-flight/queued delivery；R4只停止後續producer enqueue，不修改single worker queue。

### 20.8 Go Local Metadata Retention

Risk：自然完成後`MlModelInfo`與collection references保留。

Control：這是approved cleanup boundary，避免update race與scope expansion；subscription inactive可防止analytics
report與observation consumer繼續使用。完整cleanup另列future lifecycle work。

### 20.9 Monitoring Duration Completion Delay

Risk：`monDur`到期後最多等下一個`repPeriod`才完成。

Control：保留historical ticker semantics並加test/documentation；不在R4增加第二個timer與新的ordering。

---

## 21. Blocker And Replan Conditions

遇到以下任一情況必須停止implementation並回報：

1. Current apply/update ordering無法在不改external subscription procedure下保證revision safety
2. Scheduler signal queue無法避免與existing subscription stripe lock的deadlock
3. Runtime finalization必須同步呼叫Go或Go必須反向DELETE PyAnLF才能清理
4. Required callback URI不能由existing AnLF advertised server config建立
5. Current context無法提供thread-safe atomic revision transition
6. Observation active-consumer query必須改寫SMF reference-count architecture
7. Completion correctness需要durable cross-restart outbox
8. Tests顯示report count實際語意不是attempt count
9. 3GPP spec或current generated model要求不同consumer-facing completion procedure
10. Full fix必須清理SMF/UPF collection，與Decision 3衝突
11. Required race/component tests因dependency或environment無法建立
12. Contract cutover需要長期backward compatibility mode

Blocker report必須包含原假設、實際證據、可行選項、建議與是否需先修改本計畫。

---

## 22. Completion Criteria

1. R0 Reporting fixtures具有historical provenance與approved deviation標記
2. Initial failure evidence已保存但正式commits保持green
3. `maxReportNbr`與`monDur`各自建立correct stable completion event
4. PyAnLF正常完成後local runtime、binding、accuracy membership與model reference正確release
5. Explicit release/update/shutdown不建立normal completion
6. Transient generation/send exception後下一tick繼續
7. Completion tombstone持續retry至Go `204`
8. Retry重用相同completion ID與payload
9. Natural completion後same subscription runtime revision不重用
10. Go current revision atomically設inactive
11. Missing、duplicate與old revision idempotently回`204`
12. Future revision回`409`且不修改state
13. Go handler不呼叫PyAnLF DELETE
14. SMF/UPF collection、traffic storage與ADRF path未被cleanup
15. Inactive-only source停止new backend enqueue
16. Shared source仍有active subscription時繼續enqueue
17. Callback/update、completion/release與shutdown race tests通過
18. PyAnLF full tests通過
19. NWDAF focused/full tests、targeted race、build與lint通過
20. Cross-repository HTTP contract tests通過，或environment-level gap明確記錄
21. 完整code review沒有未處理P0/P1 finding
22. Parent plan、audit、R0與本文件progress已更新

R4完成只代表scheduler與subscription lifecycle可收斂。整體behavioral parity的R5 V1/V2 verification、
audit closure與status restoration已於2026-07-14完成；完整5GC V3仍未驗證。

---

## 23. Implementation Record

Implementation於2026-07-14完成並commit為`PyAnLF@bfd611c`與`NWDAF@68717f8`。

### 23.1 Initial Failure Evidence

在production code修改前新增`tests/test_reporting.py`並執行：

```text
uv run pytest -q tests/test_reporting.py
```

Test collection因current `py_anlf.core.reporting`不存在`ReportingRuntimeStopped`與completion contract而
失敗。這是預期的第一個R4 boundary failure：current scheduler只會直接return，沒有typed stale stop、normal
completion signal或manager-owned finalization。正式worktree隨後一起加入tests與fix，沒有保留red commit。

R0 Reporting slice另外建立
`tests/fixtures/behavioral_parity/reporting_lifecycle_cases.json`，以`NWDAF@0db9584`的
`internal/notifier/notifier.go`與`notifier_test.go`記錄max-report、monitoring-duration、zero-unlimited與
combined-limit check order。Fixture expected values不是由current PyAnLF output產生。

### 23.2 Implemented PyAnLF Behavior

1. `ReportingScheduler`新增subscription/revision-aware immutable completion signal
2. Max report先於monitoring duration判斷，report sequence維持attempt-before-generation/send
3. `ReportingRuntimeStopped`直接停止且不建立normal completion
4. Transient generation或sender exception等待下一個`repPeriod`後繼續
5. `SubscriptionRuntimeManager`擁有unbounded signal queue與單一finalizer worker
6. Finalizer在subscription/state lock內驗證revision並移除runtime，鎖外清理accuracy、observation與model
7. Process-local revision high-water在normal/explicit release後仍保留，防止same-ID revision reuse
8. `RuntimeCompletionDelivery`保存stable event/tombstone，所有non-`204`結果固定間隔retry
9. Delivery提供created、acknowledged、retry、contract-error、stale-signal、abandoned與tombstone stats
10. Dedicated `runtime_completion_delivery` config驗證timeout與retry interval必須為正數

### 23.3 Implemented NWDAF Behavior

1. Apply runtime contract新增required `runtime_completion_callback_uri`
2. Coordinator依advertised AnLF server URI建立callback address
3. AnLF server新增`POST /subscriptions/{subscriptionId}/runtime-completions`
4. HTTP edge保持API -> processor -> coordinator -> context分層
5. `Subscription.CompleteRuntime`在單一runtime lock內完成revision compare與inactive transition
6. Current/duplicate/missing/stale completion回`204`；future revision回`409`；invalid request回`400`
7. Completion path不呼叫PyAnLF DELETE、不刪除`MlModelInfo`、不清理SMF/UPF references
8. UPF producer只在source仍有active NWDAF subscriber時enqueue PyAnLF；traffic storage與ADRF path不變

### 23.4 Verification

Executed successfully：

```text
cd PyAnLF
uv run pytest -q
# 129 passed

cd NWDAF
make test
go test -race ./internal/anlf/... ./internal/context ./internal/sbi/processor
make build
make lint
# golangci-lint: 0 issues
```

Focused Python tests另驗證scheduler boundaries、transient recovery、stable retry payload、`400` retention、
revision high-water、stale signal、explicit release、shared model/source與scheduler-to-callback component flow。
Focused Go tests驗證route/status matrix、atomic transition、coordinator semantics、active-source gate與shared source。

`TestLivePyAnLFContract`已擴充為同時等待analytics report與runtime completion callback，但本環境未設定
`PYANLF_LIVE_ENDPOINT`，因此`make test`中該running-process test被skip。Repository-level contract tests通過；
不得描述為完整cross-process E2E。

### 23.5 Review And Remaining Risks

完整diff review沒有未處理P0/P1 finding。確認以下為approved remaining risks：

1. Completion tombstone與revision high-water不跨PyAnLF restart持久化
2. Completion delivery使用單一bounded-request worker，可能有一個request-timeout的head-of-line delay
3. Completion前已通過gate或已queued的observation仍可能有限度送達
4. Go保留`MlModelInfo`、SMF/UPF references、traffic與ADRF state
5. `monDur`於下一個scheduler tick判斷，最多延遲一個`repPeriod`
6. Accuracy與analytics report仍採各自既定finite retry policy，不提升為completion tombstone guarantee

R4因此標記為repository-level completed。

### 23.6 R5 Closure Update

2026-07-14 R5以真實PyAnLF process重跑`TestLivePyAnLFContract`，驗證analytics callback與
`maxReportNbr=1` completion；另新增live replacement/explicit release case，確認兩者不誤發natural completion。
R5 full suites、targeted race、lint、build與final review均通過，沒有未處理P0/P1 finding。完整5GC、SMF/UPF、
Daisy與ADRF V3仍因環境不可用而保持unverified。
