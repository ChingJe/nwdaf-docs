# Phase 3 Code Review Findings And Remediation Plan

Date: 2026-07-22

Status: Historical review record; current gate is tracked in `Phase 3 Review Ledger.md`

Parent documents:

- `../MTLF Backend Transition Plan.md`
- `../Phase 3 Analytics Subscription Routing.md`

Follow-up review:

- `Phase 3 Follow-up Code Review Findings And Remediation Plan.md`

Canonical current ledger:

- `Phase 3 Review Ledger.md`

This document preserves the original findings and remediation history. Its
individual acceptance gate is superseded by the canonical ledger and must not
be used to reopen already reclassified work.

Reviewed implementation repositories:

- `NWDAF/`
- `PyAnLF/`
- `PyMTLF/`

---

## 1. Purpose

本文件記錄Phase 3 implementation code review確認的問題、重現證據、影響範圍及建議修正方案。
它不是取代Phase 3主計畫，而是Phase 3進入完成狀態前必須關閉的remediation work list。

本輪確認六個問題：

1. event-level periodic reporting設定被忽略，合法standard subscription被錯誤拒絕。
2. backend restart後，已刪除subscription的stale SMF reference可能阻止cleanup。
3. sync request同步執行NRF/SMF network convergence，超過Go backend timeout。
4. 單一collection retry會阻塞其他subscription的reconciliation。
5. 不同SMF使用相同peer subscription ID時，orphan cleanup identity碰撞。
6. shutdown timeout短於outbound operation timeout，可能留下detached worker。

這些finding均為已從目前程式碼或最小重現確認的問題，不是尚未證實的推測。

---

## 2. Review Basis

### 2.1 Standard And Plan Sources

本review使用下列local Release 18 sources：

- `nwdaf-docs/specs/openapi/TS29520_Nnwdaf_EventsSubscription.yaml`
- `nwdaf-docs/specs/TS 29.520/4 Services offered by the NWDAF/4.2 Nnwdaf_EventsSubscription Service/`
- `nwdaf-docs/specs/openapi/TS29508_Nsmf_EventExposure.yaml`
- `nwdaf-docs/specs/openapi/TS29564_Nupf_EventExposure.yaml`
- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 3 Analytics Subscription Routing.md`

Phase 3已固定的相關要求包括：

- `evtReq`與event-level reporting設定必須保留standard precedence。
- Events Subscription POST的HTTP status必須符合Release 18 OpenAPI。
- consumer resource deletion與底層SMF cleanup分離，但cleanup intent必須在restart後恢復。
- peer identity為`targetApiRoot + peerSubscriptionId`，不可假設subId跨SMF全域唯一。
- sync完成local state recovery後才可usable，但不應把慢速peer network convergence綁在private
  handshake request內。
- reconciliation需per-subscription serializable，且一個subscription的retry不得無限阻塞其他
  subscription。
- worker、timer及outbound operation不得在shutdown後脫離owner繼續使用舊state。

### 2.2 free5GC Exemplar Evidence

本輪延續Phase 3原計畫使用的exemplar：

- BSF subscription CRUD：
  - `resources/references/free5gc-main/NFs/bsf/internal/sbi/api_management.go`
  - `resources/references/free5gc-main/NFs/bsf/internal/sbi/processor/subscriptions.go`
  - direct evidence：POST create、PUT replace、DELETE、Location及handler/processor分層。
- SMF outbound/callback：
  - `resources/references/free5gc-main/NFs/smf/internal/sbi/consumer/`
  - `resources/references/free5gc-main/NFs/smf/internal/sbi/processor/notifier.go`
  - direct evidence：standard model、peer request、callback成功204及consumer boundary。
- UDR persistence/callback：
  - `resources/references/free5gc-main/NFs/udr/internal/sbi/processor/`
  - direct evidence：processor與Mongo persistence boundary、resource CRUD與callback responsibility。

free5GC沒有「Go containing NWDAF + Python AnLF/MTLF backend」及其cross-process restart
reconciliation的直接precedent。因此本文件的sync、worker lane及orphan recovery方案是依本專案已確認的
ownership、bounded lifecycle及standard peer identity推導，不宣稱是直接複製free5GC implementation。

### 2.3 Verification Baseline

Review時執行：

```text
NWDAF:  go test ./...          PASS
PyAnLF: uv run pytest -q       181 passed, 1 skipped
PyMTLF: uv run pytest -q       32 passed
```

現有tests通過只代表目前已覆蓋的unit/contract behavior沒有失敗；下列六個情境未被既有test完整涵蓋。
本輪沒有執行real NRF/SMF/UPF/Mongo cross-process integration environment。

---

## 3. Finding Summary

| ID | Priority | Area | Confirmed consequence |
|---|---|---|---|
| P3-CR-01 | P1 | Events Subscription reporting | 合法event-level PERIODIC create被回501拒絕 |
| P3-CR-02 | P1 | Restart reconciliation | stale reference使peer SMF resource永遠不進cleanup |
| P3-CR-03 | P1 | Backend sync | 5秒Go sync timeout包住最長30秒且可重複多次的peer request |
| P3-CR-04 | P2 | Collection concurrency | 一個retry task在backoff期間停止所有其他subscription collection |
| P3-CR-05 | P2 | Peer identity | 跨SMF相同subId使兩個orphan共用一個cleanup identity |
| P3-CR-06 | P2 | Shutdown lifecycle | join返回後daemon worker仍可能使用已關閉repository或舊manager |

P1 finding應在Phase 3繼續擴張功能前修正。P2 finding應與其相依的P1修正同批完成，避免先建立另一套
temporary lifecycle。

---

## 4. P3-CR-01: Event-level Reporting Policy Is Ignored

### 4.1 Expected Behavior

TS 29.520 subscription procedure說明：

- `evtReq.notifMethod`若有提供，覆蓋event-level `notificationMethod`。
- `evtReq.repPeriod`若有提供，覆蓋event-level `repetitionPeriod`。
- 若top-level override未提供，event-level設定仍是有效standard input。

Phase 3目前只實作`UE_COMMUNICATION` periodic analytics，但仍應接受兩種等價的periodic representation：

```json
{
  "eventSubscriptions": [{
    "event": "UE_COMMUNICATION",
    "tgtUe": {"supis": ["imsi-1"]},
    "notificationMethod": "PERIODIC",
    "repetitionPeriod": 30
  }]
}
```

或：

```json
{
  "eventSubscriptions": [{
    "event": "UE_COMMUNICATION",
    "tgtUe": {"supis": ["imsi-1"]}
  }],
  "evtReq": {
    "notifMethod": "PERIODIC",
    "repPeriod": 30
  }
}
```

### 4.2 Current Behavior

`PyAnLF/src/py_anlf/core/subscription_service.py`只讀取`subscription.evt_req`：

- 沒有`evtReq.notifMethod`就直接當作default THRESHOLD。
- 不檢查`eventSubscriptions[].notificationMethod`。
- periodic period只讀`evtReq.repPeriod`。

`PyAnLF/src/py_anlf/core/runtime_manager.py`建立scheduler時也只讀`evtReq.repPeriod`，所以只修改API
validation仍不足以修正runtime behavior。

最小重現使用event-level `PERIODIC + repetitionPeriod=30`且不提供`evtReq`，實際結果為：

```text
status=501
cause=THRESHOLD_NOT_IMPLEMENTED
detail=notificationMethod defaults to THRESHOLD, which is not implemented
```

Release 18 Events Subscription POST明列201、400、401、403、404、411、413、415、429、500、502、503及
default response；Phase 3 status matrix沒有把501列為create behavior。此結果同時造成valid standard form被
拒絕及status contract divergence。

### 4.3 Root Cause

舊runtime request以單一`evt_req` dictionary驅動scheduler。ownership搬到PyAnLF時，event-level fields雖已
加入Pydantic model，卻沒有建立一個共用的effective reporting policy resolution step。validation和runtime
各自直接讀top-level欄位，導致precedence遺失。

### 4.4 Remediation Design

新增一個PyAnLF internal pure helper，例如`resolve_reporting_policy(...)`，但不新增HTTP API或第二套runtime
state。helper輸出至少包含：

```text
effective method
effective repetition period
immediate report flag
max report number
monitoring duration
accepted event indexes
per-event failures
```

第一版resolution規則：

1. 先取`evtReq.notifMethod`；只有非空時才覆蓋event-level method。
2. 先取`evtReq.repPeriod`；只有有效正值時才覆蓋event-level repetition period。
3. 沒有top-level override時，對每個accepted event讀取自己的`notificationMethod`及
   `repetitionPeriod`。
4. method完全省略時才使用規格default THRESHOLD。
5. Phase 3只把有效`PERIODIC + period > 0`交給runtime。
6. THRESHOLD、ONE_TIME或其他尚未實作的event使用standard `FailureEventInfo`，第一版
   `failureCode=OTHER`；若至少一個event可接受，create仍可201並回`failEventReports`。
7. 若所有event都不可接受，沿用Phase 3既有all-events-failed behavior回OpenAPI已定義的400
   `ProblemDetails`，不再回create contract未列出的501。
8. 同一subscription內多個accepted events若沒有top-level override且period彼此衝突，第一版回400，不能
   無聲選第一個period；未來若要支援per-event scheduler需另行規劃。
9. Subscription service與runtime manager必須使用同一份resolved policy，不能各自重新推導。

這個修正不實作THRESHOLD或ONE_TIME scheduler，只修正standard precedence、partial acceptance與目前已承諾
的periodic slice。

### 4.5 Required Tests

- event-level PERIODIC且無`evtReq`：201並啟動正確period scheduler。
- `evtReq` PERIODIC覆蓋不同event-level設定。
- top-level method存在但top-level period省略時，依規格定義確認是否使用event-level period；test固定最後
  採用的resolution rule。
- 多個accepted event-level periods一致：成功。
- 多個accepted event-level periods衝突：400且不建立runtime。
- THRESHOLD與PERIODIC混合：201加`failEventReports`，只為periodic event建立runtime。
- 全部method不支援：400，不回501。
- update與create使用相同resolver。
- runtime scheduler使用resolved period，而不是再次只讀raw `evtReq`。

### 4.6 Completion Criteria

- valid event-level periodic request不再被拒絕。
- create path不再產生`THRESHOLD_NOT_IMPLEMENTED` 501。
- validation、scheduler及response partial failure使用同一effective policy。
- TS 29.520 precedence有direct unit tests。

---

## 5. P3-CR-02: Restart Can Preserve Stale SMF References Forever

### 5.1 Expected Behavior

Go保存process-local external subscription routes及SMF peer mirror，供PyAnLF process restart recovery。
PyAnLF恢復後應：

1. 恢復仍被active Events Subscription引用的peer resource。
2. 對沒有有效consumer reference的peer resource建立cleanup intent。
3. 對Go已標記`pendingCleanup`的resource重新嘗試DELETE。
4. cleanup成功前保留target、peer ID及Location。

### 5.2 Current Behavior

`CollectionManager.restore()`直接把snapshot的`nwdafSubscriptionIds`轉成`resource.references`。
它沒有：

- 與同一份sync payload的active `eventsSubscriptions`交叉檢查。
- 移除已不存在的subscription ID。
- 使用`pendingCleanup`決定restore後的cleanup task。

`finalize_restore()`只處理`references`為空的resource。因此只要Go mirror還帶著一個已刪除的舊ID，該peer
resource就永遠不被視為orphan。

確認的failure sequence：

```text
consumer DELETE Events Subscription
  -> PyAnLF接受local delete並排入SMF cleanup
  -> Go刪除external route
  -> PyAnLF在peer DELETE前或DELETE失敗後crash
  -> Go仍保存peer route及舊association reference
  -> 新PyAnLF收到sync
  -> restore保留已刪除ID
  -> finalize_restore不排cleanup
  -> peer SMF subscription永久殘留
```

最小重現結果：resource保留`deleted-sub`，queued cleanup task數為0。

### 5.3 Root Cause

目前把Go的association mirror當成完整authoritative reference state，但Phase 3 ownership其實是：

- Go active Events Subscription routes是external resource存在性的current snapshot。
- Go peer association只是PyAnLF crash recovery metadata，不是第二個refcount owner。
- PyAnLF在restart後仍需把mirror與active resource intent做reconciliation。

此外`pendingCleanup`已存在wire contract，卻沒有production consumer。

### 5.4 Remediation Design

調整sync restore順序，先建立本次snapshot的`active_subscription_ids`，再restore peer resources：

```text
validate complete sync envelope
replace Go-owned projection
derive active Events Subscription ID set
restore peer resources against active ID set
restore/apply subscription runtimes locally
enqueue reconciliation and orphan cleanup
return sync success
```

每個SMF snapshot的effective references計算：

```text
effectiveReferences = snapshot.nwdafSubscriptionIds ∩ activeSubscriptionIds
```

規則：

1. 不存在於active set的reference一律視為stale，不得阻止cleanup。
2. `effectiveReferences`非空時保留resource；active subscription永遠優先於矛盾的cleanup hint，避免刪除仍被
   使用的peer resource。
3. `effectiveReferences`為空時，不論`pendingCleanup`為true或false，都建立orphan cleanup intent。
4. `pendingCleanup=true`且effective references非空時記錄structured warning，保留resource並由PyAnLF重新
   發布校正後的association mirror；不能直接DELETE。
5. restore完成後發布的full association replacement只能包含active IDs。
6. peer DELETE失敗仍保留resource與cleanup task，依bounded backoff retry。
7. SMF 404依既有決策視為cleanup terminal success，但wire response仍保持404。

建議讓`CollectionManager.restore()`明確接收active IDs或已驗證的restore context，不從全域service state
隱式查詢，以便unit test完整覆蓋。

### 5.5 Required Tests

- snapshot peer reference指向不存在的Events Subscription：排入cleanup。
- `pendingCleanup=true`且零reference：排入cleanup。
- `pendingCleanup=false`但零effective reference：仍排入cleanup。
- stale與active reference混合：只保留active reference。
- `pendingCleanup=true`但仍有active reference：不DELETE，重新發布校正association。
- cleanup DELETE暫時失敗、PyAnLF再次restart：仍會retry。
- cleanup成功後Go route與PyAnLF resource都移除。

### 5.6 Completion Criteria

- backend在DELETE流程任何時間點crash都不會永久遺失cleanup intent。
- restart後association mirror不含已不存在的external subscription ID。
- `pendingCleanup`有明確production behavior及tests。

---

## 6. P3-CR-03: Sync Performs Slow Peer Network Convergence Inline

### 6.1 Expected Behavior

Private sync的責任是：

- 驗證完整snapshot。
- 恢復local domain/runtime state。
- 建立需要收斂的collection與cleanup intent。
- 回覆snapshot已理解並接受。

NRF discovery、SMF POST/DELETE及association retry是背景convergence，不應使private sync request等待多個
peer network round trip。

### 6.2 Current Behavior

`POST /internal/v1/sync`依序呼叫：

```text
sync_projection.replace
collection_manager.restore
subscription_service.reconcile
collection_manager.finalize_restore
```

`subscription_service.reconcile()`對新或變更subscription呼叫`collection_manager.reconcile_now()`。
`reconcile_now()`直接執行`_reconcile()`；若沒有existing binding，會同步執行NRF discovery與SMF create。

目前timeout關係：

```text
Go anlfBackend.requestTimeout:               5 seconds
PyAnLF collection.discovery_timeout_seconds: 30 seconds
PyAnLF collection.request_timeout_seconds:   30 seconds
```

多SUPI乘多SMF時，network calls仍可依序重複。因此sync不只可能超過5秒，也可能遠超單一30秒timeout。

### 6.3 Consequences

- Go先取消sync並把backend轉為UNAVAILABLE。
- Python request thread可能仍在執行無法由Go cancellation中止的`requests` call。
- 下一次poll可能和前一次尚未完成的sync重疊。
- backend可能在background side effect已發生後才被Go視為sync failure。
- slow/unavailable NRF或SMF不必要地阻止backend恢復usable，即使local snapshot本身有效。

### 6.4 Remediation Design

將`reconcile_now()`從sync request path移除。sync只做local atomic restore並enqueue latest intent：

1. 完整驗證payload及duplicate IDs。
2. 建立temporary validated restore representation。
3. 在local locks內replace projection、subscription resources、runtime intent及restored peer metadata。
4. 為需要create/update/delete的subscription呼叫非阻塞`reconcile(...)`。
5. 為orphan建立非阻塞cleanup task。
6. 確認所有local state已建立後回`200 snapshotAccepted=true`。
7. background collection workers進行NRF/SMF convergence。

`snapshotAccepted=true`表示backend完整理解並恢復local intent，不表示所有外部SMF已在該HTTP request內完成
收斂。這與Phase 3「future data collection failure不rollback accepted analytics resource」的既有決策一致。

若local runtime apply本身可能載入模型或做其他慢速I/O，也應在實作時確認是否仍超過Go timeout；本finding
不授權把必要local validation省略，只要求peer network side effect離開sync request。

### 6.5 Required Tests

- mock NRF request阻塞超過Go timeout：sync仍快速成功且只enqueue task。
- mock SMF create阻塞：sync response不等待create。
- sync handler內不得呼叫`requests.get/post/delete`，用mock call count固定。
- 連續相同full sync不建立duplicate peer create。
- sync A後立刻sync B：stale A reconciliation不得執行peer side effect。
- Go availability monitor只在sync HTTP成功後標記USABLE。

### 6.6 Completion Criteria

- sync latency不再隨NRF/SMF數量或peer timeout成長。
- Go的5秒backend timeout足以涵蓋local restore。
- peer convergence失敗由collection observability呈現，不偽裝為snapshot parse failure。

---

## 7. P3-CR-04: Retry Sleeps The Only Collection Worker

### 7.1 Expected Behavior

Phase 3要求：

- 同一subscription的快速update需serialized且latest revision wins。
- 不同subscription彼此獨立。
- retry使用bounded exponential backoff。
- unresolved Group ID只影響相關subscription，不應暫停其他consumer的collection。

### 7.2 Current Behavior

所有`ReconcileTask`共用一個thread與一個FIFO queue。task要求retry後，worker直接呼叫：

```text
closing.wait(backoff)
```

等待完成後才把task重新放回queue。這段wait發生在唯一worker內，所以queue後面的所有task都停止處理。

unknown Group ID會讓`_desired_keys()`每次都回retry；隨attempt增加，單次全域停頓最長30秒。慢速NRF/SMF
request也會在同一worker內阻塞所有其他subscription。

最小重現中，先放入一個retrying task，再放入independent task；200ms後只執行前者，後者尚未開始。

### 7.3 Root Cause

目前queue同時承擔：

- ready work queue
- delayed retry scheduler
- per-subscription serialization

但只有一個worker，且backoff以worker sleep實作，導致三種責任互相耦合。

### 7.4 Remediation Design

建立bounded、app-owned collection executor，但不建立unbounded thread-per-subscription：

1. 使用小型固定worker pool；預設值在實作時固定並做startup validation，不需要為第一版增加複雜的
   dynamic scaling。
2. 保留`_reconcile_locks[subscription_id]`或等價lane state，確保同一subscription最多一個task in flight。
3. dispatcher只提交ready task；retry以`next_attempt_at`放入delayed priority queue或timer scheduler，worker
   不sleep等待backoff。
4. 同一subscription的新revision取代尚未執行的舊task；舊revision開始或完成side effect前仍檢查
   `_is_current()`。
5. retry task到期時只重新排該subscription，不阻塞其他lane。
6. worker pool及retry scheduler都由`SBIServer`/manager lifecycle擁有並可完整shutdown。
7. queue保持bounded；若coalescing後仍達capacity，依collection control-task語意保留latest per subscription，
   不能用raw-data drop-oldest直接丟掉唯一cleanup intent。

這裡的control queue與callback raw-data buffer不同。raw notifications可以依已確認policy drop-oldest；resource
create/delete intent必須透過coalescing及bounded latest-state representation保證最終收斂。

### 7.5 Required Tests

- subscription A retry backoff期間，subscription B可立即完成。
- subscription A慢速network call時，B仍可由另一worker處理。
- 同一subscription兩個快速revision不並行。
- stale revision不create/delete peer resource。
- unknown Group ID只增加該subscription retry/metric。
- retry delay達max interval後不再增長。
- executor capacity下只保留每個subscription latest intent。
- shutdown取消delayed retries且所有workers退出。

### 7.6 Completion Criteria

- 一個subscription無法讓所有collection停頓。
- per-subscription serializability及latest-revision semantics維持不變。
- retry scheduler不建立detached timer/thread。

---

## 8. P3-CR-05: Orphan Identity Ignores Target SMF

### 8.1 Expected Behavior

Phase 3已明確規定peer subscription identity為：

```text
targetApiRoot + peerSubscriptionId
```

不同SMF可合法回傳相同`subId`。所有routing、restore、association及cleanup都必須使用完整peer tuple。

### 8.2 Current Behavior

`finalize_restore()`為orphan產生：

```text
orphan:{peerSubscriptionId}
```

接著把該字串當成`_subscription_keys` key。如果SMF-A與SMF-B都回`same-id`：

1. 第一個orphan寫入`_subscription_keys["orphan:same-id"]`。
2. 第二個orphan覆蓋相同entry。
3. queue內雖有兩個task，但兩者查到同一份最後寫入的key set。
4. 一個peer resource被清理，另一個沒有任何可到達的cleanup mapping。

最小重現得到兩個orphan resources、兩個queued tasks，但synthetic mapping只保存一個resource key。

### 8.3 Root Cause

Go routing已改成target-aware composite key，但PyAnLF orphan cleanup仍沿用「peer subId全域唯一」的舊
假設，且把orphan假裝成subscription reference來重用release path。

### 8.4 Remediation Design

推薦移除synthetic subscriber ID，新增明確的peer cleanup task：

```text
PeerCleanupTask {
    collectionKey
    targetApiRoot
    peerSubscriptionId
    attempt
}
```

規則：

1. cleanup task以`CollectionKey`或`targetApiRoot + peerSubscriptionId`辨識resource。
2. orphan不加入`resource.references`，避免fake reference影響association mirror及refcount reasoning。
3. 同一peer tuple的cleanup taskcoalesce；不同target即使subId相同仍是不同task。
4. DELETE仍由既有`_delete_peer()`使用原target與peer ID執行。
5. cleanup成功後移除resource並發布full association mirror。
6. cleanup暫時失敗只retry相同composite peer。

若實作希望先做較小修改，synthetic ID至少必須由完整composite tuple生成；但長期建議使用explicit cleanup
task，因為orphan本質不是consumer subscription。

### 8.5 Required Tests

- SMF-A/same-id與SMF-B/same-id各自DELETE一次。
- 一個DELETE成功、一個失敗：只retry失敗peer。
- association update不包含synthetic orphan reference。
- 相同target與subId的duplicate snapshot被validation或restore dedup明確處理。
- target URI normalization後identity仍一致。

### 8.6 Completion Criteria

- codebase內沒有以peer subId單獨辨識SMF resource或cleanup task的production path。
- 跨SMF相同subId不再造成mapping overwrite或resource leak。

---

## 9. P3-CR-06: Shutdown Can Leave Detached Workers

### 9.1 Expected Behavior

所有worker需滿足：

- manager/server擁有明確stop signal。
- shutdown停止接受新資料。
- pending wait與retry可被喚醒。
- current outbound operation有有限timeout。
- owner在關閉repository及runtime dependency前確認worker已退出。
- repeated startup/shutdown不留下操作舊manager的thread。

### 9.2 Current Behavior

目前timeout關係：

```text
Collection/ADRF request timeout: up to 30 seconds
Collection worker join timeout:  5 seconds
ADRF worker join timeout:        5 seconds
Mongo worker join timeout:       5 seconds
```

`IngestionPipeline.shutdown()`在join timeout後仍會呼叫`repository.close()`。如果Mongo worker仍卡在insert或
ADRF/collection worker仍在network request：

- shutdown method已返回，但daemon thread仍存活。
- thread可能繼續操作已close repository。
- FastAPI app再次startup時會建立新manager及新threads，舊thread仍持有舊runtime/reference。
- process exit依賴daemon thread被強制終止，而不是正常lifecycle completion。

此外`DropOldestBuffer.close()`目前仍允許worker把已存在的items逐一pop；大型queue在shutdown時可能進行
長時間drain，使5秒join更容易失敗。

### 9.3 Root Cause

shutdown採固定短join timeout，但outbound calls使用更長timeout，也沒有定義「graceful drain」或「立即丟棄
pending best-effort data」的明確策略。repository close order沒有以worker termination為前置條件。

### 9.4 Remediation Design

Phase 3 raw notification storage是best-effort，建議shutdown採bounded discard而非無限drain：

1. server先停止接受新callback與subscription request。
2. 設定所有manager closing event。
3. ingestion/control buffers關閉時停止接收並清除尚未開始的pending items；增加
   `dropped_on_shutdown_total`或等價log/metric。
4. retry wait必須被closing event立即喚醒，不再requeue。
5. current network/database operation仍由既有finite timeout結束。
6. join必須等待worker實際退出；不可在join timeout後繼續close其dependency。
7. 所有Mongo/ADRF/collection worker退出後才close repository、runtime manager及model manager。
8. 若需要全域shutdown deadline，deadline必須至少覆蓋current operation timeout；到期視為明確fatal
   lifecycle failure並記錄，不可靜默留下thread後宣稱shutdown complete。
9. repeated FastAPI lifespan startup時必須確認舊worker count為0再初始化新manager。

Python `requests`無法可靠以另一個thread的Event中斷已送出的blocking call，因此最小安全修正是保留有限
connect/read timeout並讓owner等到current call返回。後續若要更快取消，可改用支援structured async
cancellation的client，但不應為本finding額外重寫整個SBI stack。

### 9.5 Required Tests

- ADRF request正在等待時shutdown：shutdown等待operation結束，最後worker不存活。
- Mongo insert正在等待時shutdown：repository只在worker退出後close。
- retry backoff中shutdown：立即喚醒，不等完整backoff。
- 大量pending raw items：依discard policy結束，不逐筆做network drain。
- repeated TestClient/startup-shutdown：沒有舊thread操作新manager或closed repository。
- collection association publisher同樣可完整停止。
- shutdown後enqueue不接受資料且有明確counter/log。

### 9.6 Completion Criteria

- 所有manager shutdown返回時，其owned threads均已退出。
- repository/model/runtime dependency不會在使用者thread仍存活時被close。
- repeated startup/shutdown test驗證沒有detached workers。

---

## 10. Cross-finding Remediation Sequence

建議依下列順序實作，避免重複修改相同lifecycle：

### Step 1: Fix Standard Reporting Resolution

- 完成P3-CR-01 resolver、status及tests。
- 不同時實作THRESHOLD/ONE_TIME scheduler。

### Step 2: Make Sync Local-state-only

- 完成P3-CR-03。
- 移除sync path的`reconcile_now()` peer I/O。
- 固定snapshot accepted語意。

### Step 3: Correct Restart Reference And Peer Identity

- P3-CR-02與P3-CR-05同批修改。
- active ID intersection、`pendingCleanup`及explicit composite cleanup task一起完成。
- 不保留synthetic orphan subscriber作temporary dual path。

### Step 4: Replace The Global Blocking Worker

- 完成P3-CR-04 bounded executor、delayed retry及latest revision coalescing。
- 讓create、release及orphan cleanup共用同一個清楚的executor lifecycle，但使用不同typed task。

### Step 5: Close Lifecycle Ownership

- 完成P3-CR-06。
- 在新executor基礎上統一stop、buffer close、worker join及dependency close order。

### Step 6: Full Verification And Re-review

- 執行unit、race、lint、build及可行的cross-process smoke。
- 對照本文件每個completion criteria。
- 再做一次focused code review，確認沒有以temporary compatibility path迴避finding。

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

- sync timeout/usable transition tests。
- target-aware peer snapshot/association tests。
- backend restart與pending cleanup contract tests。

### 11.2 PyAnLF

```bash
uv run ruff check .
uv run pytest -q
```

需新增focused suites：

- reporting precedence/status matrix。
- restore active/stale/pending reference matrix。
- duplicate peer subId across targets。
- cross-subscription retry isolation。
- sync no-peer-I/O assertion。
- shutdown/repeated lifecycle tests。

涉及thread race的test不能只依固定`sleep`判斷；應使用Event/barrier控制worker進度並assert bounded outcome。

### 11.3 PyMTLF

```bash
uv run ruff check .
uv run pytest -q
```

本輪沒有確認PyMTLF sync seam的blocking finding，但unified sync contract若調整欄位或accepted semantics，
PyMTLF contract tests仍需同步驗證，不能讓AnLF/MTLF形成不同handshake格式。

### 11.4 Integration Scope

環境允許時至少驗證：

1. Go + PyAnLF + PyMTLF startup，兩個backend完成readiness→sync→usable。
2. consumer以event-level PERIODIC建立subscription。
3. PyAnLF透過Go discovery並對mock/real SMF建立resource。
4. PyAnLF在SMF cleanup pending時crash/restart，恢復後完成DELETE。
5. 兩個SMF回相同peer subId仍各自route及cleanup。
6. 一個SMF長時間失敗時，另一個subscription仍能建立collection。
7. shutdown時所有Python worker正常退出。

real NRF/SMF/UPF/Mongo未執行時，交付說明必須明確標示unit-level verification，不得稱為完整end-to-end
validation。

---

## 12. Non-goals And Decision Boundary

本remediation不包含：

- THRESHOLD或ONE_TIME scheduler實作。
- AoI-based SMF discovery。
- UDM Group ID resolution。
- MTLF dataset retrieval。
- accuracy/retrain或model provision migration。
- Go process durable persistence。
- TLS/OAuth delegation。
- message broker或distributed queue。

目前六項修正均可依既有Phase 3 ownership與標準contract進行，不需要新增high-level architecture決策。
若實作時發現下列情況，必須先更新計畫並要求決策：

1. Release 18 procedure要求的reporting precedence無法映射到目前單一scheduler，而需要per-event scheduler。
2. Go snapshot不能提供足以辨識active/stale association的資料，必須擴張sync contract。
3. bounded worker executor需要改變已確認的resource acceptance或cleanup guarantee。
4. Python HTTP client必須全面改為async才能滿足shutdown，而不再是局部lifecycle修正。

---

## 13. Final Acceptance Gate

只有下列條件全部成立，才能將本code review標記resolved：

1. P3-CR-01至P3-CR-06各自有對應production fix及regression test。
2. event-level periodic standard request成功，create不再回不合計畫status matrix的501。
3. backend crash不會使stale SMF reference或pending cleanup永久遺失。
4. sync handler不執行NRF/SMF peer network convergence。
5. collection retry在subscription之間隔離，同一subscription仍serialized/latest-wins。
6. 所有peer identity與cleanup path都target-aware。
7. shutdown完成時沒有owned worker存活。
8. NWDAF、PyAnLF及PyMTLF既有tests仍通過。
9. lint、race及build結果被記錄；未執行integration checks有具體原因。
10. focused re-review沒有新的P0/P1 finding。
