# Slice 7 Lifecycle Closure and Fresh-state Restart Detailed Plan

日期：2026-08-20

狀態：Completed；production implementation、mandatory review、in-scope remediation、full
verification 與 repository-separated commits 已完成。PyMTLF implementation commit：`c7c66b9`；
NWDAF implementation commit：`3279891`

上層計畫：

- [Hierarchical NWDAF Federated Learning Implementation Plan](./Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)

前置 slices：

- [Slice 0 Baseline Audit and Contract Freeze Detailed Plan](./Slice%200%20Baseline%20Audit%20and%20Contract%20Freeze%20Detailed%20Plan.md)
- [Slice 1 Hierarchy Bundle Contract and Artifact Primitives Detailed Plan](./Slice%201%20Hierarchy%20Bundle%20Contract%20and%20Artifact%20Primitives%20Detailed%20Plan.md)
- [Slice 2 Capability and Process-scoped Role Foundation Detailed Plan](./Slice%202%20Capability%20and%20Process-scoped%20Role%20Foundation%20Detailed%20Plan.md)
- [Slice 3 Root Initiation, Static Topology and Assignment Detailed Plan](./Slice%203%20Root%20Initiation%2C%20Static%20Topology%20and%20Assignment%20Detailed%20Plan.md)
- [Slice 4 End-to-end Preparation and Admission Detailed Plan](./Slice%204%20End-to-end%20Preparation%20and%20Admission%20Detailed%20Plan.md)
- [Slice 5 Hierarchical Rounds and Aggregation Detailed Plan](./Slice%205%20Hierarchical%20Rounds%20and%20Aggregation%20Detailed%20Plan.md)
- [Slice 6 Hierarchical Final Validation and Publication Detailed Plan](./Slice%206%20Hierarchical%20Final%20Validation%20and%20Publication%20Detailed%20Plan.md)

開發規則：

- [NWDAF Development Policy](../../development_policy.md)

---

## 1. 目的與 slice 終點

Slice 7 不新增 training algorithm、topology strategy 或 promotion flow。它負責把 Slice 1–6
已建立的 hierarchy resources 收斂成完整 lifecycle，並證明任何單一 Go NWDAF 或 PyMTLF
process 重啟後都不會恢復、接續或重建舊 experiment。

本 slice 的成功終點是：

1. Root final validation rejection、publication完成、必要cutover完成或technical failure後，
   experiment依實際outcome進入terminal closure；`CANDIDATE_READY`與`PUBLISHING`都不是
   cleanup terminal；
2. Root、Branch 與 Leaf 的標準 Training subscriptions、callback correlations、upper／lower
   mappings、worker state 與 experiment reservation 全部依 ownership bounded cleanup；
3. Root candidate只保留到active final validation與same-process publication不再讀取；之後立即
   清除，正式模型只透過既有`FINAL_MODEL`／catalog／Model Provision lifecycle提供；
4. failure、graceful shutdown、Go restart 與 PyMTLF restart 都不恢復 active HFL experiment或
   尚未完成的publication動作；restart後只重新載入已完成catalog commit的model result，不重建
   Root、Branch、Leaf、Training subscription、publication retry或adoption tracking；
5. stale callback、PATCH、DELETE、round result 與 artifact 不會建立新 process 或污染新的
   experiment；
6. hierarchy runtime terminal records、retired plan identities、tombstones與artifact directories
   不會在長時間process中無限成長。

Slice 7 完成後才進入 Slice 8 real multi-process E2E。Slice 8 驗證真實部署互動；不得把本
slice 可用 deterministic unit／component tests 關閉的 lifecycle 缺口延後到 E2E 才發現。

---

## 2. 已固定的 product semantics

以下是上層計畫既有決策，不在 Slice 7 重新選擇：

- active hierarchy state 只存在 memory；
- 不 persistence、resume、replay 或 reconcile 舊 active experiment；
- 任一必要 participant failure 使完整 experiment terminal；
- 不自動 retry；operator 修復後明確建立全新 request 與 `planId`；
- 同一 PyMTLF instance 同時間最多一個 active top-level training；
- Root、Branch、Leaf 是 plan-scoped role，不是 deployment role；
- Go 不保存 hierarchy business state，也不 serving／proxy artifact bytes；
- 最後Root aggregate只是待驗證candidate；`CANDIDATE_READY`只在final validation accepted後
  成立，之後由`PUBLISHING`、`CUTOVER_PENDING`與`COMPLETE`區分promotion lifecycle；
- `PublicationCoordinator`的journal、ADRF／catalog commit與cutover維持既有同一process內的
  publication contract；第一版不把journal作為cross-restart action recovery來源；
- restart後只重新載入已完成catalog commit的current／latest model result；未完成publication一律視為
  abandoned，不呼叫`PublicationCoordinator.resume()`；
- candidate沒有terminal retention consumer；synchronous publication call返回、terminal failure或
  graceful shutdown時立即清除；restart直接清空由該PyMTLF獨占的FL scratch workspace；
- crash 前無法刪除的 remote resources 不新增 recovery protocol，只由 peer timeout、peer
  cleanup 或既有 garbage-collection policy 收斂。

### 2.1 本 slice 明確不做

- active experiment persistence；
- WAL、database、distributed lease 或 leader election；
- restart 後查詢 peers 並重建 subscription mapping；
- automatic training retry 或 topology retry；
- final validation、`FINAL_MODEL`、ADRF store、catalog promotion 或 cutover的新功能；這些由
  Slice 6實作，本Slice只關閉其resource lifecycle；
- 新增 public SBI 或修改 Release 18 OpenAPI schema；
- OAuth／mTLS requester identity 與 artifact origin 的額外安全綁定；
- real multi-process NRF／OAuth／TLS deployment acceptance；該驗證仍屬 Slice 8。

---

## 3. Baseline evidence 與已確認缺口

### 3.1 Slice 6 後 Root outcome 尚未關閉 resource lifecycle

現行`FLRootCoordinator._run()`已呼叫`FLServerEngine.finalize_hierarchy_candidate()`，並投影
`VALIDATION_REJECTED`、`CANDIDATE_READY`、`PUBLISHING`、`CUTOVER_PENDING`與`COMPLETE`。
但該呼叫正常return後沒有success／rejection cleanup，也沒有清除`_active_request_id`。

現行outcome因此是：

- `VALIDATION_REJECTED`仍保留upper-tier Training resources、Server process、Root active ID與
  shared `FLExperimentRegistry` reservation；
- 無active cutover scopes的`COMPLETE`仍保留相同owner；
- `CUTOVER_PENDING`需要保留publication／policy ownership，但Training resources與callback
  correlations已不再需要，現行仍全部保留；
- adoption完成時`FLServerEngine.mark_scope_adopted()`雖把hierarchy process投影成`COMPLETE`，
  仍未完成Root與registry release。

Failure path已有`_cleanup_attempt()`，但它固定把registry terminal outcome寫成`FAILED`，且直接
`release_plan()`；不能原樣套到validation rejection或success。
Slice 7必須抽出outcome-aware共同primitive，不得另建互不一致的success-only orchestrator。

### 3.2 Hierarchy cleanup 已存在，但不是完整 terminal lifecycle

目前已具備：

- `FLServerEngine._cleanup_hierarchy_process()` bounded DELETE participants 並清 correlation；
- Branch `cancel(plan_id)` 取消 lower process 與清除 upper／lower round mapping；
- FL Client hierarchy-aware DELETE cascade；
- `FLWorkspace.release_plan(plan_id)` exact UUIDv4、idempotent deletion；
- shared experiment registry 的 terminal／cleaning／release state；
- Root、Branch、Server 與 Client 的 shutdown fences。

但目前各 engine 仍保留 unbounded terminal state：

- Root `_records`；
- Server `_processes`；
- Branch `_cancelled_plan_ids` 與完成後的 execution／round bookkeeping；
- Client `_cancelled_hierarchy_resources`；
- experiment registry `_retired_plan_ids`。

### 3.3 `release_plan(plan_id)` 無法覆蓋所有 round artifacts

Assignment、Leaf assignment 與 preparation-result artifacts 以 `planId` 作為 workspace 第一層
directory；但 Root／Branch／Leaf round artifacts 依各層 `mlCorreId`／Server process ID 發布。
現有 `release_plan(plan_id)` 只刪除同名 plan directory，因此不能保證移除該 plan 擁有、但
位於其他 process directory 的 `ROUND_INPUT`、`ROUND_LOCAL` 與 `ROUND_GLOBAL`。

Slice 7 必須補process-local per-plan artifact ownership index，再宣稱 cleanup 或 URL invalidation
完整。不得以掃描manifest後猜測ownership，亦不得把arbitrary path交給release primitive。

### 3.4 Artifact cleanup 目前只有粗粒度 startup TTL

`FLWorkspace.open()`目前只依`workspace_ttl_seconds`清理stale directories與staging files，且
artifact serving route只檢查檔案是否存在。既然第一版不恢復任何FL action，Slice 7改為要求
每個PyMTLF獨占自己的`workspace_root`，startup在接受request前清空整個FL scratch workspace。
Normal lifecycle則依memory ownership index立即清理；不新增durable marker或periodic workspace
worker。

### 3.5 PyMTLF restart -> Go 已有 generation fence

PyMTLF `/health/ready` 每次 process startup 產生新的 `processInstanceId`。Go NWDAF 的
`AvailabilityMonitor` 已以此值做 backend generation fence；generation 改變時，
`ResetMLModelBackendGeneration()` 會：

- 移除依賴舊 PyMTLF generation 的 model routes；
- 對 consumer-owned peer Training resource做一次 best-effort DELETE；
- 建立 deletion tombstone；
- 不建立 recovery worker 或 replay state。

Slice 7 應補完整 Training route matrix 與 lifecycle tests，不重寫既有 monitor 架構。

### 3.6 Go restart -> PyMTLF 尚無反向 generation fence

Go NWDAF restart 後，其 in-memory Training routes 全部自然清空；但若同一節點的 PyMTLF
沒有同時 restart，PyMTLF 仍可能保有：

- prepared FL Client resources；
- Root／Branch Server process；
- callback outbox；
- single-active experiment reservation；
- old containing-Go callback URI correlations。

目前 private `/internal/v1/nwdaf-context` 只有 stable `nfInstanceId` 與 base URIs，沒有可辨識
Go process generation 的欄位。只比對 `nfInstanceId` 無法發現同一 NWDAF instance 的 Go
process 已重啟。

本 plan 建議在這個既有 private Go-to-PyMTLF context boundary 增加
`processInstanceId`。它只用於 lifecycle generation fencing，不是 hierarchy role、Root
identity、artifact trust attestation 或新的 standard field。

### 3.7 Dead restart helpers 不構成 recovery protocol

`FLClientEngine.restore_after_restart()` 與 `FLServerEngine.discard_restored_routes()` 目前沒有
production call site。它們會讓程式表面上看似存在 restart restoration flow，但實際 app
startup 並未載入舊 resources。

Slice 7 應移除這兩個 unreachable recovery-shaped helpers及其孤立 tests，或在 code review
提供仍有 active caller 的直接證據。第一版不得接通它們。

### 3.8 Existing-flow lifecycle baseline map

Canonical baseline是flat FL production lifecycle；Slice 6提供HFL validation／publication
outcomes，本Slice只調整hierarchy resource graph與fresh-state restart：

| Baseline stage | HFL disposition |
| --- | --- |
| active Training resources remain through final validation | reused；Root／Branch／Leaf resources不得在candidate形成時提前cleanup |
| `CANDIDATE_READY` followed by `PUBLISHING` | reused；兩者都不是terminal cleanup signal |
| validation rejection／technical failure cleanup | adapted；Root DELETE需cascade Branch lower resources |
| publication handoff | reused within one process；synchronous call返回前candidate仍由active call stack擁有 |
| `CUTOVER_PENDING` retains retrain ownership | reused within one process；可刪Training subscriptions但不得釋放仍required的publication／policy reservation |
| `COMPLETE` releases retrain owner | adapted；同時釋放hierarchy registry、mappings與plan-owned artifacts |
| participant DELETE and correlation removal | adapted為Root–Branch–Leaf bounded cascade |
| terminal record retention | adapted為Root status TTL；non-queryable engine state immediate removal；duplicate markers使用tombstone TTL |
| backend restart generation fence | reused for PyMTLF-to-Go；增加Go-to-PyMTLF reverse generation |
| active experiment crash recovery | explicitly not applicable；本版本discard-and-start-fresh，不resume／reconcile |
| unfinished publication crash recovery | explicitly not applicable；pending action abandoned，只有已commit catalog result會重新載入 |

同名state、artifact與terminal outcome的前置／後置條件必須維持Slice 6與flat baseline語意。

### 3.9 Recovery helper不是第一版production contract

現行`PublicationCoordinator`雖將pending publication寫入`DurableModelStateRepository`，並提供
`resume()`與相關tests，但`src/py_mtlf/app.py`只呼叫`publication.open()`，production沒有呼叫
`publication.resume()`。第一版維持這個fresh-state邊界，不接通該helper。

必須區分兩種資料：

- **已完成的model result**：catalog已commit的revision／`latestModelId`可由既有
  `seed_catalog.restore()`重新載入，之後仍可提供給新的consumer；
- **尚未完成的publication action**：ADRF store、catalog promotion、cutover或adoption tracking
  只要在process退出時尚未完成，就不在restart後繼續執行。

Startup不處理或改寫non-terminal publication journal，只是不呼叫`resume()`、不建立recovery
worker、publication admission fence或舊Root private status。各crash point的結果為：

| Restart前最後狀態 | Restart後規則 |
| --- | --- |
| `CANDIDATE_READY`／`RESERVED`／`FINAL_BUNDLE_READY`／`STORE_IN_FLIGHT` | abandon舊candidate與publication；既有latest model不變 |
| `STORE_ACCEPTED`但catalog尚未commit | 不自動promotion；既有latest model不變；ADRF可能留下orphan model |
| `CATALOG_COMMITTED`／`CUTOVER_PENDING` | 已commit revision仍是current／latest result；不恢復舊cutover retry、scope adoption tracking或admission fence |
| `COMPLETE` | 依既有catalog restore載入完成結果，沒有action需要恢復 |

Journal原樣保留作audit／model-ID high-water資訊；startup不terminalize、compact或重新排入worker，
也不能把`CATALOG_COMMITTED`／`CUTOVER_PENDING`誤標成舊adoption已完成。
ADRF store成功但catalog尚未commit所留下的orphan model，是第一版明確接受的crash limitation；
自動query、delete或reconciliation另列future work。

### 3.10 現行shutdown ordering可能卡在publication retry

現行FastAPI shutdown先呼叫`fl_root.close()`並等待Root executor，之後才
`publication.close()`。若Root executor正在`PublicationCoordinator.publish()`的recoverable
retry loop，publication尚未收到closing signal，Root close可能無法bounded完成。

Slice 7必須把「停止publication retry並abandon本process未完成action」與「等待Root worker退出」
的順序拆開；不得用無限join或先關HTTP client造成in-flight publication使用已關閉client。
下次startup不接續這次被停止的publication。

---

## 4. Standard、private 與 implementation-only boundary

### 4.1 Release 18 standard behavior

Release 18 `Nnwdaf_MLModelTraining` OpenAPI 已定義：

- Individual Training subscription `DELETE` success 為 `204 No Content`；
- unknown individual resource 可回 `404`；
- notification success response 為 `204`；
- callback 也宣告 `404` 等 error responses。

Slice 7 不新增 standard field。Parent DELETE 仍是 normal success／cancellation cleanup 的主要
跨 NWDAF mechanism。

### 4.2 Standard-shaped private behavior

Go 對 PyMTLF 的 Training CRUD／notification relay 繼續使用 standard payload semantics；
Go route generation、backend resource ID 與 tombstone 是 private routing state。

`/internal/v1/nwdaf-context.processInstanceId` 是純 private lifecycle metadata：

- producer／owner：`pkg/service.NwdafApp`，每次startup產生新的UUIDv4；
- carrier：既有 private context response；
- consumer：PyMTLF containing-NWDAF generation monitor；
- lifetime：單次 Go process lifetime；
- absence／malformed：PyMTLF readiness 不成立，不能接受新的 FL work；
- change：舊 experiment terminal cleanup，不 resume，之後才重新開 admission。

PyMTLF model／log中的名稱使用`containing_nwdaf_process_instance_id`等明確字樣，避免與
PyMTLF自身`/health/ready.processInstanceId`混淆；wire response欄位仍是既有private naming
style的`processInstanceId`。

### 4.3 PyMTLF-only state

下列資訊不進 standard request：

- terminal timestamp／retention deadline；
- process-local plan-to-path artifact ownership index；
- retired plan／resource tombstones；
- Root request retention and garbage-collection metadata。

---

## 5. Target lifecycle model

### 5.1 兩層 closure，不再把所有owner視為同一個slot

```text
ACTIVE
  -> validation/publication lifecycle owned by Slice 6
  -> Training procedure closure
       - VALIDATION_REJECTED / FAILED
       - or publication returns CUTOVER_PENDING / COMPLETE in the same process
  -> remote Training resource cleanup attempted with bounded retries
  -> local Training correlation / worker / mapping cleanup
  -> top-level ownership closure
       - VALIDATION_REJECTED / FAILED / COMPLETE: release
       - CUTOVER_PENDING: retain until adoption COMPLETE
  -> candidate deleted when synchronous publication returns and open readers drain
  -> terminal status/tombstone retained when required
  -> retention deadline
  -> garbage collected
```

`Training procedure closure`表示standard Training subscriptions與round callbacks已不再需要；
`top-level ownership closure`表示single-active admission fence可以釋放。兩者在
`CUTOVER_PENDING`刻意不同，實作與tests不得只用一個boolean同時代表兩者。

`CUTOVER_PENDING`阻擋的是下一場top-level training，不是同一場training的下一個round；進入
此state時training rounds與final validation都已完成。第一版必須在同一process內保留fence，
因為`AccuracyPolicy`每個model family只有一組adoption generation，新的
`begin_generation()`會覆寫尚未完成的previous／current version與scope tracking；同時
`FLExperimentRegistry`本來就只允許一個process-local top-level experiment。若要在cutover期間
並行下一場training，必須先把adoption改為以`(family, modelId)`保存多個generations並調整
single-active contract，不屬於本slice。Process restart仍採fresh state，不重建此fence。

Network I/O、artifact deletion 與 worker waits 不得在 registry lock、Root condition、Branch
condition、Client resource lock 或 Server process condition 內執行。

### 5.2 Outcome and ownership transitions

| Outcome | Root status | Remote subscriptions | Root candidate | Active slot |
| --- | --- | --- | --- | --- |
| validation accepted | `CANDIDATE_READY -> PUBLISHING` | still owned；不得因candidate ready cleanup | active synchronous publication call owns candidate until return | retain |
| publication awaiting adoption | `CUTOVER_PENDING` | bounded cleanup；publication adoption不依賴Training resource | publication不再讀candidate後立即刪除 | retain same-process publication fence |
| publication／cutover complete | `COMPLETE` | bounded cleanup（若尚未完成） | immediate release | release after cleanup |
| enforced gate rejection | `VALIDATION_REJECTED` | bounded cleanup | immediate release | release after cleanup |
| preparation／round failure | `FAILED` | bounded cleanup | none；release immediately | release after cleanup |
| validation technical failure | `FAILED` | bounded cleanup | formed candidate immediate release | release after cleanup |
| graceful shutdown | active HFL status變`FAILED`／shutdown cause | bounded best effort | stop publication、wait call return，再依plan ownership清除candidate | release local active／publication ownership |
| Go generation change | active HFL terminal local failure | best effort where routes remain | active call返回後依plan ownership清除candidate | release local active／publication ownership |
| PyMTLF process crash | HFL memory lost | Go generation reset handles known routes | startup在admission前清空獨占FL workspace | new HFL／publication memory fresh；只載入已commit model result |

`CANDIDATE_READY`只有在Slice 6完成final validation後才可見，但它不是cleanup signal。
`PUBLISHING`期間candidate可能仍是same-process synchronous publication call的必要input，也不得
cleanup。
`CUTOVER_PENDING`只允許關閉Training procedure，不允許釋放publication admission fence；
`COMPLETE`才表示publication與必要cutover完成。

### 5.3 Cleanup error semantics

- cleanup error 不得把 terminal experiment 退回 active；
- remote DELETE `204` 與 `404` 都視為 cleanup accepted；
- transport／5xx cleanup 依既有 bounded retry 設定執行，耗盡後記錄
  `cleanup_failure`，仍完成 local release；
- 後續 duplicate DELETE 必須 idempotent；
- cleanup failure 不觸發 training retry 或 recovery worker。

---

## 6. Role-specific terminal cleanup

### 6.1 Root terminal closure

收到Slice 6產生的outcome後，Root必須依序：

1. freeze terminal state、candidate digest、published model identity（若有）、completed rounds與
   terminal timestamp；
2. fence new round publication、callback acceptance 與 state mutation；
3. 對所有 upper-tier Branch Training subscriptions執行 bounded DELETE；
4. 移除 Server correlation 與 participant resource ownership；
5. detach Server process；
6. 若outcome是`CUTOVER_PENDING`，保留publication admission fence與Root／Server完成投影所需的
   最小mapping；若是`VALIDATION_REJECTED`、`COMPLETE`或`FAILED`，將experiment registry走完
   `TERMINAL -> CLEANING -> release`；
7. candidate在synchronous publication call返回前仍屬於active plan，不先執行`release_plan()`；
   publication返回、失敗或shutdown令call退出後，依memory ownership index立即刪除candidate與
   其他plan artifacts；
8. 只在top-level ownership closure後清除`_active_request_id`；
9. 最後publish immutable terminal status snapshot。`CUTOVER_PENDING`仍是active snapshot，
   adoption完成投影`COMPLETE`後再執行一次idempotent final closure。

Root request status依`terminal_status_ttl_seconds`維持queryable。Status只保留candidate digest，
不提供candidate URL；candidate artifact不是terminal status的一部分，不做terminal retention。

### 6.2 Root failure

沿用 Slice 4／5 fail-fast control decision，但 cleanup 仍需 bounded 收斂：

- terminal cause 只 freeze 一次；
- 已建立的 upper resources 全部 cleanup；
- candidate不論是否已形成，都在active reader釋放後隨terminal cleanup立即release，不因journal
  存在而跨restart保護；
- active slot release；
- failure request snapshot retained until TTL；
- degradation failure latch behavior不在本 slice 改變。

### 6.3 Branch

Branch upper resource被 Root DELETE、Go generation fence觸發或 local shutdown時：

1. fence upper Client revision及尚未發布的 callback；
2. cancel Branch execution與所有 lower round waits；
3. bounded DELETE lower-tier Leaf resources；
4. 清除 upper／lower process mapping、round mapping與next-round counter；
5. release Branch-owned assignment、download、result與round artifact directories；
6. 移除 upper Client resource；
7. 只有 upper resources與lower Server process都不存在後才 release shared experiment slot。

同一 cleanup request replay 不得重送已完成的 lower work，也不得因 mapping 已被移除而變成
500。

### 6.4 Leaf

Leaf parent DELETE、Go generation fence或 shutdown時：

- increment／retire resource revision，讓 preparation、training與outbox worker結果變 stale；
- cancel delay timer；
- release callback／training capacity恰好一次；
- release該 plan的download與local round artifacts；
- remove Client resource並留下 bounded tombstone供 duplicate DELETE；
- release experiment slot。

Leaf 不需要知道完整 topology，也不與 Root 做額外 reconciliation。

### 6.5 Distributed artifact cleanup cascade

每個PyMTLF只刪除自己的local workspace；Root不得直接操作Branch／Leaf filesystem。正常cleanup
固定沿standard Training resource ownership cascade：

```text
Root cleanup
  -> Root DELETEs Branch Training resources
     -> each Branch DELETEs its Leaf Training resources
        -> each Leaf releases its downloads, ROUND_LOCAL artifacts and workers
     -> Branch releases its assignments, downloads and lower/upper round artifacts
  -> Root releases its assignments, downloads and Root round artifacts
```

`owner_plan_id`必須隨assignment與local process ownership一路帶到Root、Branch與Leaf，使每個節點
的`release_plan(plan_id)`都能找到本機以`planId`或不同`mlCorreId`為directory key的完整artifact
集合。Peer unreachable時只做bounded DELETE；呼叫端仍完成local release，peer上的殘留由peer
自己的next workspace operation重試，或在restart時由startup workspace clear清除。

---

## 7. Artifact ownership、cleanup 與 URL invalidation

### 7.1 Process-local per-plan ownership

`FLWorkspace.publish()` 與 hierarchy-specific wrappers 增加 explicit `owner_plan_id` input。
Hierarchy caller 必須從已驗證 assignment、Root request或active experiment取得此值，不能從
caller-provided path反推。

`FLWorkspace`只在memory保存canonical `plan_id -> owned process directories` index；每次
hierarchy publish／download成功後，以workspace自行產生且已驗證為`workspace_root` descendant的
directory註冊ownership。Index不寫入bundle或filesystem，也不跨restart保存。

`release_plan(plan_id)` 必須：

- 只接受 canonical UUIDv4；
- 刪除exact plan directory與memory index明確屬於該plan的所有process directories；
- 只能在該plan的active publication call已返回後執行；
- 對不存在的 plan維持成功；
- 不接受 path、glob或workspace root；
- 不刪除 sibling plan或unowned flat process；
- 刪除成功後移除memory index；刪除失敗保留exact path供後續workspace operation重試。

`workspace_root`是單一PyMTLF process獨占的scratch directory，不能與另一個live PyMTLF共享，
也不能與`storage.artifact_root`、`model_state.directory`或`publication.directory`重疊。Startup
在artifact route與request admission開啟前清空其contents並建立空memory index；清空失敗則
readiness不成立，不得部分沿用舊workspace。Durable-root overlap由config validation拒絕；不同
live processes共用同一路徑列為unsupported deployment misconfiguration，第一版不新增filesystem
lease或distributed lock。Resolved `workspace_root`不得是filesystem root、process working
directory、repository root，亦不得是任何durable root的ancestor；cleanup primitive只逐一刪除
驗證後workspace的direct children，不刪除workspace root本身，也不接受glob或unresolved path。

Publication不新增`PUBLICATION_PINNED`state。`FLServerEngine.finalize_hierarchy_candidate()`同步
呼叫`PublicationCoordinator.publish()`；call返回前Root lifecycle不得執行`release_plan()`。
Final validation consumers已在`CANDIDATE_READY`前完成，candidate不是publication期間的external
download contract。`FINAL_BUNDLE_READY`後正式bundle已複製進`ArtifactRepository`；publication
call返回後Root立即清除candidate。正式`FINAL_MODEL`有獨立artifact lifecycle，不依賴FL
workspace。

### 7.2 Lifecycle retention config

第一版只為仍有consumer的terminal metadata保留分離設定：

```yaml
federated_learning:
  workspace_ttl_seconds: 3600
  lifecycle:
    terminal_status_ttl_seconds: 3600
    tombstone_ttl_seconds: 3600
```

- `workspace_ttl_seconds`：同一process內staging與recorded cleanup failure的opportunistic retry
  threshold；startup仍無條件清空整個workspace；
- `terminal_status_ttl_seconds`：Root private terminal status snapshots；
- `tombstone_ttl_seconds`：retired plan、cancelled resource與duplicate cleanup markers；
- Root、Branch、Leaf normal parent cleanup：各自立即`release_plan()`；
- active candidate由synchronous validation／publication call ownership保護，不依賴TTL；
- publication返回、failure或shutdown令call退出後立即release candidate；restart無條件清空。

### 7.3 Live cleanup and URL invalidation

不新增periodic maintenance worker。Workspace publish／download／release等既有操作
opportunistically重試recorded local cleanup failures；Root status GET／submit與resource
GET／DELETE／admission各自在既有lock內lazy-prune對應的過期terminal record或tombstone。
Artifact serving route必須只resolve仍存在且未被lifecycle owner釋放的artifact：

- active owner存在：`200`、digest／ETag不變；
- lifecycle cleanup完成或檔案missing：`404 ARTIFACT_NOT_FOUND`；
- concurrent resolve與cleanup使用open-file handle、process-local reader count或等價的簡單
  primitive協調；cleanup不得讓response讀到partial bytes，結果只能是完整bytes或`404`；
- active plan不得只因總執行時間超過 TTL而被清除。

### 7.4 Missing artifact behavior

| Stage | Missing／expired artifact result |
| --- | --- |
| Branch下載Root assignment | Branch preparation failure；不得dispatch Leaves |
| Leaf下載Branch assignment | Leaf preparation termination |
| Root下載Branch result | Root attempt failure |
| Leaf下載round input | round failure |
| Branch／Root下載local result | tier／experiment failure；不得partial aggregate |
| Branch／Leaf在active final validation下載candidate | active owner存在時不得正常expiry；missing使experiment failure |
| internal candidate URL在terminal cleanup後被重試 | `404`；不重建或重新發布；不是management contract |

沒有 artifact retry worker、alternate origin或由 Go proxy補救。

---

## 8. Go／PyMTLF restart generation boundary

### 8.1 Go process generation producer

`NWDAF/pkg/service.NwdafApp`每次startup建立新的UUIDv4 `processInstanceId`。它與stable
`nfInstanceId`分離：

```text
stable nfInstanceId
+ one-boot processInstanceId
= containing NWDAF generation observed by PyMTLF
```

Ownership與傳遞固定為：

```text
pkg/service.NwdafApp.processInstanceID
  -> constructor injection
  -> internal/mtlf.Server.processInstanceID
  -> internal/backend.NwdafContextResponse.processInstanceId
```

不得把這個project-specific generation放進`internal/context`、factory config、`pkg/app`
interface、NRF profile、NF service schema或public SBI。它只存在於既有private
`/internal/v1/nwdaf-context` response。

### 8.2 PyMTLF generation monitor

PyMTLF startup在開始接受 FL requests前取得並保存 containing Go generation。App-owned
monitor以同一private context boundary做bounded periodic refresh；所有會建立新top-level
training的入口只讀取monitor維護的cached readiness／generation fence，不再各自同步發出context
request。Monitor尚未取得generation或正在reset時，admission保持關閉。

偵測 generation change時：

1. close new experiment admission；
2. freeze目前Root／Branch／Leaf state為terminal local failure；
3. wake所有waiters並fence worker publication；
4. bounded cleanup仍可識別的local／remote resources；route已遺失的`404`視為完成；
5. release所有active reservations與old-generation local artifacts；
6. 清除old-generation Root status、Branch mappings、Client resources與只服務old generation的
   tombstones；
7. bind新 generation；active experiment cleanup完成後重新開top-level training admission。

這是active experiment的discard-and-start-fresh，不是hierarchy recovery。Generation reset
不得把舊request轉成新`planId`，也不得保留prepared snapshot、publication worker、cutover
tracking或admission fence。只有已commit的catalog result依既有restore載入。

若 containing Go context暫時不可用：

- readiness為not ready；
- 不接受新的 FL work；
- active work進入bounded terminal cleanup，不無限等待Go恢復；
- Go恢復後只有明確的新 request可建立新experiment。

### 8.3 PyMTLF restart

沿用既有方向：

1. 新 PyMTLF產生新 backend `processInstanceId`；
2. Go availability monitor關閉舊 generation admission並等待in-flight leases；
3. Go移除舊 Training routes、建立tombstone並best-effort cleanup consumer-owned peer resource；
4. startup在admission前清空獨占FL workspace；新PyMTLF registry、Root records、Client／Server
   engines與memory ownership index皆為fresh；
5. 不呼叫FL restore helper、不重播舊callback、不重新attach舊route；
6. startup不呼叫`publication.resume()`，也不改寫non-terminal publication journal；不建立
   recovery worker或admission fence；
7. 已完成catalog commit的current／latest model result依既有catalog restore載入。

### 8.4 Restart matrix

| Restart target | Expected local result | Peer-visible result | New experiment |
| --- | --- | --- | --- |
| Root PyMTLF | Root memory fresh；Go discards old routes | Branch old resources收到best-effort DELETE或timeout | explicit new request only |
| Branch PyMTLF | upper／lower state fresh；Go discards old routes | Root current attempt fails／times out | no Branch self-retry |
| Leaf PyMTLF | Client state fresh；Go discards old route | Branch round/preparation fails | no Leaf self-retry |
| Root Go only | PyMTLF generation reset discards old Root experiment | old callbacks/routes stale | explicit new request only |
| Branch Go only | PyMTLF discards paired upper／lower state | Root sees 404／failure／timeout | no automatic recreation |
| Leaf Go only | PyMTLF discards old Client state | Branch sees failure／timeout | no automatic recreation |
| Go + PyMTLF | both memory stores fresh | old interactions stale | explicit new request only |

上表的「memory fresh」同時適用HFL／Training owner與未完成publication action。Restart不繼續
ADRF／catalog／cutover動作；只有restart前已完成catalog commit的model result會重新載入。

---

## 9. Stale interaction rules

| Interaction | Rejection key | Required effect |
| --- | --- | --- |
| callback to old Go route | route miss／tombstone／generation mismatch | `404`／terminal error；no route creation |
| PATCH old subscription | route or backend resource missing | `404`；no PyMTLF resource creation |
| duplicate DELETE | tombstone／cancelled resource marker | first accepted cleanup remains idempotent；later `404` allowed |
| old Branch result | no active Root correlation or wrong plan | reject；no request record creation |
| old Leaf result | no active lower correlation or wrong round | reject；no aggregation |
| old assignment CREATE replay in same generation | retired `planId` | conflict／failure；no second experiment |
| old artifact URL after lifecycle cleanup | inactive／missing ownership or missing file | `404`；no republish |
| worker completes after reset | generation + revision + plan fence | drop result；no callback／artifact publication |

Stale rejection必須以direct active lookup、generation、revision、correlation與plan ownership完成，
不得只比對round number或log message。

Restart 後不持久化retired set，因此不宣稱能辨識任意外部惡意重播。第一版受控系統中，舊
artifact本身不會建立process；必須同時有新的standard CREATE。正常system flow在restart後
不會自動重送該CREATE，且Go／Py generation fences會終止舊attempt。跨管理域replay
protection仍是future security hardening。

---

## 10. Retention and garbage collection

### 10.1 Retained objects

| Object | Retention start | Retention | GC owner |
| --- | --- | --- | --- |
| Root terminal request snapshot | terminal cleanup complete | terminal status TTL | Root coordinator |
| Root candidate artifacts | synchronous validation／publication call返回 | immediate release after open response／reader drains | FL workspace |
| Server process | Training cleanup complete；`CUTOVER_PENDING`只保留完成投影所需最小mapping | immediate after terminal／adoption completion | Server engine |
| retired plan IDs | registry release | tombstone TTL | experiment registry |
| cancelled Client resource IDs | local cleanup complete | tombstone TTL | Client engine |
| cancelled Branch plan IDs | local cleanup complete | tombstone TTL | Branch coordinator |
| failed／cancelled non-candidate plan artifacts | n/a | immediate release | lifecycle owner |
| FL workspace from previous process | PyMTLF startup | unconditional cleanup before admission | FL workspace |
| same-process failed local deletion | deletion failure | opportunistic retry on later workspace operation | FL workspace |

Terminal status與tombstone TTL可獨立調整。所有time-based state保存absolute deadline或
injected clock可比較值，不以collection size作為唯一GC條件，也不以一種object的TTL代替另一種
owner語意。Candidate沒有terminal retention clock。

Server、Branch execution／round mapping與Client active resource不是queryable terminal status，
完成cleanup後立即移除；只有Root private status需要`terminal_status_ttl_seconds`。這避免為
第一版新增沒有consumer的多份terminal snapshot。

### 10.2 Lazy GC behavior

- Root status GET／submit與resource GET／DELETE／admission只lazy-prune各自owner的terminal／retired
  objects，不碰active reservation；
- Root private GET在record過期後回`404`；
- GC與duplicate GET／DELETE並行時結果只能是完整snapshot／accepted cleanup／`404`，不得
  產生partial object；
- cleanup與GC都idempotent；
- local deletion failure記錄exact path並在後續workspace operation重試，不中止process；
- 不新增maintenance thread、timer、stop event或shutdown join。

---

## 11. Shutdown ordering

PyMTLF graceful shutdown順序固定為：

1. `runtime.accepting_requests = false`；
2. stop generation monitor and wake its wait；
3. 對`PublicationCoordinator`執行non-blocking stop signal，讓retry退出並把本process未完成的
   publication call返回，但不改寫journal，且尚不關閉可能仍被in-flight worker使用的HTTP client；
4. Root coordinator fence new requests、等待worker退出並關閉active Training lifecycle；
5. Branch coordinator fence new lower dispatch；
6. Client engine cancel resources／timers／outbox publication；
7. Server engine wake waits and bounded cleanup participant resources；
8. release shared experiment reservation；
9. 確認Root executor中的publication call已返回且local／open response reader歸零，立即刪除
   candidate與其餘plan-owned artifacts；
10. 關閉owned HTTP clients；
11. close workspace and remaining application services。

Cleanup I/O有既有request timeout與retry bound；shutdown不得因peer unreachable永久卡住。
Shutdown後不由下一個process依journal接續publication。已完成catalog commit的model result
仍由catalog restore重新載入；未完成action不恢復。

NWDAF維持app-owned cancellation／WaitGroup shape：availability workers綁定app context，server
shutdown與NRF deregistration保持既有順序。本 slice不新增Go background goroutine；Go
`processInstanceId`由`pkg/service.NwdafApp`初始化並constructor-inject到MTLF private server；
private handler只投影既有app-owned值，不自行產生generation。

free5GC alignment basis：

- primary exemplar：local free5GC NRF `pkg/service/init.go` 與app-ownedlifecycle shape；
- secondary exemplar：local free5GC UDM `pkg/service/init.go`；
- direct evidence：app owns context cancellation、WaitGroup、server stop與runtime context；workspace
  exemplars沒有通用`processInstanceId` convention；
- project-specific inference：containing Go／PyMTLF雙process generation fence沒有exact free5GC
  exemplar，因此沿用本 workspace既有backend `AvailabilityMonitor` ownership，不宣稱是
  upstream free5GC protocol。

---

## 12. Concurrency and lock discipline

### 12.1 Lock order

沿用 Slice 4／5 established rule：

```text
generation/admission fence
-> experiment registry
-> Root / Branch coordinator condition
-> Server / Client engine state
-> no lock during network or filesystem I/O
```

不得反向由 worker持有engine lock後進入generation reset。Reset先freeze identities，釋放lock
後執行cleanup，再以generation／revision fence確認是否仍可commit結果。

### 12.2 Deterministic races to test

- final aggregate完成與Root shutdown同時發生；
- success cleanup與late Branch callback同時發生；
- Go generation change發生在Branch lower PATCH fan-out途中；
- generation reset發生在Leaf local artifact write前／後；
- artifact GET與terminal／restart cleanup同時發生；
- duplicate DELETE與GC tombstone同時發生；
- Root terminal record GC與status GET同時發生。

使用Event、barrier、injected clock與fault-injection HTTP client；fixed sleep不得作為唯一證據。

---

## 13. Repository impact

### 13.1 `PyMTLF/`

預期變更：

- `src/py_mtlf/core/fl_workspace.py`
  - process-local plan-to-path ownership index；
  - startup clear of the exclusively owned scratch workspace；
  - complete `release_plan()`；
  - immediate lifecycle release／open-reader-safe deletion；
  - opportunistic retry of recorded local cleanup failures。
- `src/py_mtlf/core/fl_experiment.py`
  - timestamped retired identities；
  - generation reset／terminal GC primitives。
- `src/py_mtlf/core/fl_root.py`
  - `VALIDATION_REJECTED`／`COMPLETE`／`FAILED` terminal cleanup；
  - active-slot release；
  - terminal request retention／GC；
  - generation-reset abort。
- `src/py_mtlf/core/fl_branch.py`
  - idempotent complete plan cleanup；
  - bounded retired bookkeeping；
  - generation-reset abort。
- `src/py_mtlf/core/fl_client.py`
  - hierarchy resource abort／tombstone GC；
  - capacity release exactly once；
  - remove unused restart restoration helper。
- `src/py_mtlf/core/fl_server.py`
  - validation／publication terminal cleanup reuse；
  - Training cleanup與publication ownership分離；terminal process immediate removal；
  - generation-reset abort；
  - remove unused restored-route helper。
- `src/py_mtlf/core/publication.py`
  - production `open()`不啟動recovery worker；
  - split retry stop signal from final client close。
- `src/py_mtlf/core/nwdaf_context.py`
  - parse and expose containing `processInstanceId`；
  - no identity／trust overclaim。
- `src/py_mtlf/app.py`
  - app-owned containing-Go generation monitor；
  - explicitly keep `publication.resume()` outside production startup；
  - restore completed catalog results only；
  - corrected shutdown ordering。
- `src/py_mtlf/api/artifacts.py`
  - lifecycle-aware resolve與cleanup race handling。
- focused tests、config comments與full regression。
- `src/py_mtlf/config.py`
  - typed terminal-status／tombstone retention settings；
  - reject broad／unsafe `workspace_root` and overlap with durable artifact／model-state／publication
    roots。

實作可依現有ownership合併helper；不得新增workspace／status／tombstone maintenance timer。

### 13.2 `NWDAF/`

預期變更：

- `pkg/service.NwdafApp`-owned boot UUIDv4 `processInstanceId`；
- constructor injection into `internal/mtlf.Server`；
- private `NwdafContextResponse` projection；
- service／private handler tests；
- MTLF generation reset Training route matrix、late callback／PATCH／DELETE tests；
- startup／shutdown regression。

不新增public route、不修改generated OpenAPI model、不增加hierarchy state。

### 13.3 `nwdaf-docs/`

- 本 detailed plan；
- 上層計畫 status／decision／implementation record回填；
- 若implementation review改變private generation contract，先更新本文件再改code。

### 13.4 `PyAnLF/`

預期不修改。它不擁有Training subscription、hierarchy state或artifact workspace。只有直接
code evidence顯示shared lifecycle regression時才重新評估。

---

## 14. Implementation slices and checkpoints

### Checkpoint 1：Characterization and conformance map

- 建立本文件每項normative requirement到production path／test／verification的working map；
- 以failing tests固定Root success占slot、incomplete plan release、cleanup後URL仍可用與unbounded
  terminal record baseline；
- 固定`CUTOVER_PENDING`的Training cleanup／publication fence分離；
- 固定production不呼叫`publication.resume()`為第一版contract，並固定現行shutdown retry hang的
  baseline；
- 補Go／Py restart matrix中已有behavior的characterization tests。

Checkpoint驗收：每個confirmed gap都有deterministic evidence，不從prose直接做大範圍重構。

### Checkpoint 2：Plan-owned artifact lifecycle

- explicit owner propagation；
- process-local ownership index與exact multi-directory release；
- startup unconditional scratch-workspace cleanup與directory exclusivity validation；
- Root candidate immediate release after synchronous validation／publication returns；
- active plan immunity；
- open-reader-safe deletion與opportunistic cleanup retry。

Checkpoint驗收：同一plan跨多個process IDs的scratch artifacts可exact release；synchronous
publication call返回後candidate立即清除；restart在admission前清空全部FL scratch artifacts；
sibling與flat artifacts在同一process的targeted cleanup中不受影響。

### Checkpoint 3：Terminal outcome closure

- Root `VALIDATION_REJECTED`／`COMPLETE`／`FAILED` cleanup ordering；
- `CANDIDATE_READY`／`PUBLISHING`不得觸發terminal cleanup；
- `CUTOVER_PENDING`清除Training resources但保留publication admission fence；
- adoption完成後從Server observer觸發Root final closure；
- upper-to-lower DELETE cascade；
- active slot release；
- terminal snapshot retention；
- Branch／Leaf idempotent cleanup；
- terminal bookkeeping lazy pruning。

Checkpoint驗收：第一個terminal experiment完成cleanup後，第二個明確request可立即取得新
`planId`；`CUTOVER_PENDING`期間則仍拒絕第二個request。Candidate在publication call返回且open
reader釋放後立即刪除；terminal status與tombstone各依自己的TTL收斂。

### Checkpoint 4：PyMTLF restart -> Go closure

- 完整backend generation reset Training route tests；
- stale callback／PATCH／DELETE；
- tombstone exactly-once behavior；
- no restored route/helper path。

Checkpoint驗收：新PyMTLF generation不繼承舊Go route，old interaction不能觸達新backend
resource。

### Checkpoint 5：Go restart -> PyMTLF closure

- Go boot generation private projection；
- PyMTLF generation monitor；
- production不呼叫`resume()`且startup不改寫non-terminal publication journal；
- completed catalog result restore不建立publication worker或admission fence；
- Root／Branch／Leaf active reset matrix；
- idle prepared resource reset；
- worker publication fence；
- new admission只在old cleanup後reopen。

Checkpoint驗收：只重啟Go也不會讓PyMTLF舊experiment永久占slot，且不會resume old round或
publication；只重啟PyMTLF時不恢復hierarchy／publication action，但已完成catalog result仍可用。

### Checkpoint 6：Shutdown、GC and regression

- deterministic shutdown races；
- publication retry shutdown先signal、後等待Root call返回，之後立即清除candidate；
- lazy terminal／tombstone pruning與opportunistic workspace retry；
- full PyMTLF tests／Ruff；
- NWDAF `make test`、`make build`、`make lint`；
- flat FL regression；
- mandatory initial review、in-scope remediation、targeted recheck與full rerun。

Checkpoint驗收：plan-conformance map全部closed後才可標示Slice 7 Completed並準備
repository-separated commits。

Checkpoint 1–3先在`PyMTLF/`形成可review的implementation work unit；Checkpoint 4先完成
既有Go backend-generation matrix；Checkpoint 5的private response producer在`NWDAF/`獨立
commit，consumer／monitor再於`PyMTLF/`獨立commit；Checkpoint 6只做收尾與regression。
不得把兩個repositories累積成一個不可分離的大diff，也不得在Checkpoint 3尚未closed前開始
cross-repository generation實作。

---

## 15. Verification matrix

### 15.1 PyMTLF focused tests

#### Root terminal outcomes and slot

- `CANDIDATE_READY`／`PUBLISHING`不cleanup active publication owner；
- `VALIDATION_REJECTED`／`COMPLETE`／`FAILED`後cleanup all upper Training resources；
- `CUTOVER_PENDING` cleanup Training resources但registry／publication fence仍阻擋第二個request；
- adoption投影`COMPLETE`後Root active ID與registry恰好release一次；
- terminal outcome／adoption completion後shared experiment registry為empty；
- second manual request obtains new request／plan；
- no automatic request created；
- terminal status依TTL收斂；candidate在publication call返回且open reader釋放後立即刪除。

#### Failure and idempotency

- preparation／round／shutdown failure全部release slot；
- cleanup `204`／`404` accepted；
- transport failure bounded且local release仍完成；
- duplicate Root／Branch／Leaf cleanup不重複capacity release；
- no partial aggregate or new callback after terminal fence。

#### Artifact ownership and cleanup

- one plan ownsassignment + Root process rounds + Branch process rounds + Leaf process rounds；
- exact release removes all owned scratch dirs；
- sibling plan與flat process保留；
- ownership index只接受workspace產生的normalized descendant paths；arbitrary path／root rejected；
- startup無條件清空獨占FL workspace且建立空ownership index；cleanup失敗時readiness不成立；
- active plan超過TTL不被opportunistic cleanup刪除；
- publication call返回／terminal cleanup後candidate GET `404`；
- GET／cleanup race只產生complete bytes或`404`；
- startup在admission前清空所有舊FL scratch artifacts與staging files。

#### Retention GC

- Root records、retired plans、Branch cancelled plans與Client tombstones到期清除；
- terminal Server／Branch／Client active records在cleanup後立即移除，不等待status TTL；
- active objects不清除；
- Root status after TTL `404`；
- lazy pruning不需要background worker且不影響active object。

#### Restart generation

- fresh PyMTLF app has new process ID and empty hierarchy state；
- pending publication不恢復Root／Server／Branch／Leaf state，不resume publication，也不重建
  top-level training admission fence；
- completed catalog result可重新載入並提供給新的consumer；
- containing Go generation unchanged不reset；
- Go generation changed時Root／Branch／Leaf各自terminal cleanup；
- generation reset duringdispatch／training／callback publication drops stale work；
- Go context unavailable closes admission and does notwait forever；
- no call site for removed restore helpers。

#### Publication and shutdown

- app production lifecycle明確不呼叫`PublicationCoordinator.resume()`；
- restart於`RESERVED`／`FINAL_BUNDLE_READY`／`STORE_IN_FLIGHT`時abandon舊publication，既有latest
  model不變；
- restart於`STORE_ACCEPTED`且catalog未commit時不promotion，接受ADRF留下orphan model的第一版
  limitation；
- restart於`CATALOG_COMMITTED`／`CUTOVER_PENDING`時保留已commit current／latest result，但不
  恢復cutover retry、scope adoption tracking或publication admission fence；
- restart於`COMPLETE`時依既有catalog restore載入完成結果；
- publication call返回且open reader釋放後candidate立即刪除，正式artifact仍可用；
- publication retry期間shutdown可bounded退出，未完成action不由next startup接續；
- untouched non-terminal journal不會建立worker、fence或舊publication ownership。

### 15.2 NWDAF focused tests

- one Go app lifetime returns stable valid `processInstanceId`；
- new app lifetime returns a different generation while configured `nfInstanceId` remains stable；
- private context response includes capabilities andgeneration；
- MTLF backend generation change clears inbound andoutbound Training routes；
- consumer-owned peer route gets at most one best-effort DELETE；
- provider-owned local route is removed without callingthe new backend generation；
- old callback／PATCH cannot reach new backend resource；
- first late DELETE tombstone behavior與second `404`保持既有contract；
- app shutdown monitor goroutine exits。

### 15.3 Full commands

PyMTLF：

```bash
.venv/bin/pytest -q
.venv/bin/ruff check .
```

NWDAF：

```bash
make test
make build
make lint
```

Documentation：

```bash
git diff --check
```

以上是unit／component／local build驗證，不宣稱real multi-process E2E。Slice 8另行執行
real NRF、peer NWDAFs與cross-process artifact flow。

---

## 16. Review checklist

### Contract

- [x] public Release 18 path／method／status未變
- [x] private `processInstanceId` producer／carrier／consumer／lifetime完整
- [x] no hierarchy role added to NF profile
- [x] no active HFL or unfinished publication restore／resume contract introduced
- [x] completed catalog result restore不建立舊publication action
- [x] artifact ownership只存在process-local memory index，不進bundle或filesystem marker

### Lifecycle

- [x] Root terminal outcome releases subscriptions and active slot
- [x] `CANDIDATE_READY`／`PUBLISHING`不被誤當terminal cleanup signal
- [x] `CUTOVER_PENDING`只關閉Training resources，publication admission fence保留到adoption完成
- [x] Branch upper DELETE cascades lower cleanup
- [x] Leaf capacity／outbox slots release exactly once
- [x] cleanup failure cannot reactivate experiment
- [x] graceful shutdown order bounded
- [x] second explicit request can start after prior terminal cleanup

### Restart

- [x] PyMTLF restart is fenced by Go backend generation
- [x] Go restart is fenced by PyMTLF containing generation
- [x] Root／Branch／Leaf individual restart matrix covered
- [x] stale callback／PATCH／DELETE／result cannot create state
- [x] no old plan resume／reconciliation／automatic retry
- [x] non-terminal publication journal不建立worker、admission fence或Root／Branch／Leaf state

### Artifact and GC

- [x] release covers all process directories owned by exact plan
- [x] ownership index完整涵蓋planId與各層mlCorreId directories
- [x] sibling／flat artifacts preserved
- [x] Root candidate在synchronous publication call返回且open reader釋放後立即刪除
- [x] terminal status retained for terminal status TTL
- [x] tombstones retained for tombstone TTL
- [x] lifecycle-cleaned／missing URL returns `404`
- [x] active plan never expires solely because runtime exceeds TTL
- [x] terminal records／tombstones／retired identities collected
- [x] no workspace／status／tombstone maintenance worker introduced
- [x] artifact response持有open handle／reader count時cleanup不產生partial bytes
- [x] config rejects durable-root overlap；deployment config gives each PyMTLF a unique workspace root
- [x] startup clear rejects filesystem／working／repository roots and deletes only validated direct children
- [x] containing-Go generation admission只讀single monitor的cached fence

### Verification and review

- [x] focused deterministic tests pass
- [x] PyMTLF full pytest／Ruff pass
- [x] NWDAF test／build／lint pass
- [x] flat FL regression pass
- [x] mandatory initial review complete
- [x] admitted findings remediated and targeted rechecked
- [x] plan-conformance map closed for production behavior and verification
- [x] repository-separated commits prepared with descriptive bodies

---

## 17. Risks and controls

### 17.1 Candidate被cleanup過早刪除

控制：`finalize_hierarchy_candidate()`到`PublicationCoordinator.publish()`維持同步call ownership；
call返回前不得`release_plan()`。Call返回且open response／local reader歸零後立即刪除candidate。
Shutdown先signal publication、等待Root call返回再cleanup；restart則在admission前清空workspace。

### 17.2 Plan release遺漏round process directories

控制：每次hierarchy publish／download都要求explicit owner plan並立即註冊process-local ownership
index；`release_plan()`只接受index內由workspace建立的normalized descendant path，不依賴manifest
猜測或caller-provided path。Restart不需重建index，直接清空scratch workspace。

### 17.3 Workspace TTL誤刪active長時間training

控制：active plan不依age刪除；正常terminal／shutdown依memory owner立即清理。Startup無條件清空
獨占workspace，不需要generation marker；同一process的failed deletion只在後續workspace operation
opportunistically retry。

### 17.4 Go generation reset與worker commit競爭

控制：generation是每次worker commit的fence之一；reset先close admission／increment revision，
再做I/O cleanup。

### 17.5 Reset deadlock

控制：固定lock order；network／filesystem不在lock內；Event／barrier tests覆蓋dispatch、
callback與GC races。

### 17.6 Cleanup耗盡但slot未釋放

控制：remote cleanup failure只記錄operator-visible failure；local correlation與reservation仍
必須release，不建立unbounded retry worker。

### 17.7 GC刪除status造成late retry語意改變

控制：TTL內request ID replay仍回同一snapshot；TTL後GET回`404`，相同request ID可視為新
operator request。Restart本來就會立即清空此idempotency memory，因此不宣稱durable
idempotency。

### 17.8 ADRF store與catalog commit之間crash

控制：fault injection固定`STORE_ACCEPTED`但catalog尚未commit時的結果：舊latest model不變、
restart不promotion，且不恢復publication action。ADRF可能留下orphan model，第一版透過log／
operator觀察處理，不宣稱自動cleanup。

### 17.9 Shutdown等待publication retry

控制：publication提供separate retry stop signal與final client close；先喚醒synchronous retry loop，
再等待Root executor中的call返回，最後才關owned HTTP client。Production不啟動recovery worker，
因此沒有額外publication thread需要join。Deterministic test以recoverable ADRF failure固定此順序。

---

## 18. Deferred work classification

### Future-phase handoff

- real multi-process restart and network partition E2E（Slice 8）；
- persistent idempotency key或durable terminal history；
- remote resource reconciliation；
- cross-restart publication action recovery；
- ADRF orphan model reconciliation／deletion；
- 只依ADRF重建PyMTLF local artifact／catalog state。

### Optional hardening

- cryptographic replay protection；
- signed generation tokens；
- distributed lease／fencing service；
- cross-vendor restart protocol；
- metrics dashboard for retention／GC volume。

### Integration verification gap

- real supervisor process restart；
- real NRF／OAuth／TLS cleanup；
- cross-host candidate download exactly at lifecycle cleanup boundary；
- peer timeout behavior under network partition。

### Existing legacy cleanup

- `DurableModelState.pending_publications`的completed／failed journal compaction與
  `tombstoned_model_ids`長期壓縮是flat FL既存durable-state維護議題；Slice 7不得為了HFL
  runtime GC順帶改變其model-ID或audit語意。Slice 7只需確保non-terminal journal不被resume、
  不建立worker或admission fence；長期compaction另案處理。

這些不阻擋Slice 7 deterministic lifecycle acceptance，除非實作證據顯示第一版主流程依賴
其中任一項才能正確收斂。

---

## 19. Reviewed and confirmed contracts

先前討論已確認：

1. **Go restart generation**：`pkg/service.NwdafApp`每次boot產生UUIDv4
   `processInstanceId`，constructor-inject到`internal/mtlf.Server`，再由private
   `nwdaf-context` response投影；PyMTLF在變更時discard active FL state；
2. **分離retention clocks（已取代）**：先前曾規劃workspace、candidate artifact、terminal
   status與tombstone分離設定；candidate沒有terminal consumer後已由下方第10項取代；
3. **Candidate lifecycle**：final aggregate必須先經Slice 6 final validation／publication；
   `CANDIDATE_READY`不是terminal cleanup signal，private status不提供candidate URL。

本次review後，使用者於2026-08-20進一步確認：

4. **Restart semantic boundary**：Root／Branch／Leaf／Training state與尚未完成的publication action
   都不resume；production不呼叫`PublicationCoordinator.resume()`；
5. **Completed result boundary**：已完成catalog commit的current／latest model result可重新載入，
   但`CUTOVER_PENDING`的cutover retry、scope adoption tracking與admission fence不跨restart重建；
6. **Publication candidate ownership**：不新增candidate pin state；synchronous publication call
   返回前由active plan持有candidate，返回後立即刪除，不轉terminal retention；
7. **Same-process cutover**：`CUTOVER_PENDING`可cleanup Training resources，但在process未重啟的
   前提下，publication admission fence保留到adoption完成；
8. **Shutdown ordering**：先signal並abandon未完成publication，再等待Root executor，最後才join
   並關閉publication HTTP client；next startup不接續該action；
9. **Known V1 limitation**：ADRF store成功但catalog尚未commit時crash，可能留下orphan model；
   第一版不做自動query、delete或reconciliation；
10. **Candidate immediate release**：private status不提供candidate URL，正式`FINAL_MODEL`已有獨立
    artifact lifecycle，因此不新增`candidate_artifact_ttl_seconds`、`PUBLICATION_PINNED`或
    `RETAINED` marker；shutdown在call返回後清除，restart清空整個FL workspace；
11. **Distributed artifact cleanup**：Root DELETE Branch、Branch DELETE Leaves；每個PyMTLF依
    `owner_plan_id`清除自己的local workspace，Leaf downloads／local results也在cleanup範圍內；
12. **Same-process cutover fence rationale**：第一版每個family只有一組adoption generation且整個
    PyMTLF只有一個top-level experiment slot，因此`CUTOVER_PENDING`在同一process內阻擋下一場
    training；restart依fresh-state決策不重建此fence；
13. **Scratch workspace ownership**：每個PyMTLF獨占且只用一個`workspace_root`；它不得與durable
    model／publication directories重疊。Runtime只用memory plan-to-path index，startup在admission
    前無條件清空，不寫durable ownership marker；
14. **No maintenance worker**：workspace cleanup failure採existing operation opportunistic retry，
    terminal status與tombstone採owner-operation lazy pruning；不新增periodic maintenance thread；
15. **Single generation monitor**：只有app-owned monitor refresh containing-Go generation，training
    admission只讀cached readiness／generation，不再同步重查private context。

這些不是額外algorithm或strategy選項，而是Slice 7的fresh-state lifecycle contract。Production
implementation、review、remediation、verification與repository-separated commits均已完成。
Production commits為PyMTLF `c7c66b9`與NWDAF `3279891`。

若implementation evidence迫使改變上述owner、state flow或private contract，必須先更新本
計畫並使用development policy decision gate，不得在code中默默改回舊設計。

---

## 20. Completion criteria

Slice 7只有在下列條件全部成立後才能標為Completed：

1. Root在`VALIDATION_REJECTED`、`FAILED`以及publication handoff完成後執行bounded
   upper Training cleanup；`CANDIDATE_READY`／`PUBLISHING`不提前cleanup；
2. `CUTOVER_PENDING`期間Training resources已清除但publication admission fence仍存在；adoption
   `COMPLETE`後才release top-level ownership，下一個explicit request可建立全新`planId`；
3. candidate URL不作為private management contract；candidate在synchronous publication call
   返回且open reader釋放後立即刪除，只有Root terminal status與tombstone依各自retention設定GC；
4. `release_plan()`依process-local ownership index清除exact plan擁有的所有assignment、download
   與round process directories；publication call返回後立即release candidate；restart在admission
   前清空整個獨占FL workspace；
5. sibling plan與flat FL artifacts不受影響；
6. Branch parent cleanup完整cascade到lower Leaves；
7. Leaf worker、timer、callback與capacity ownership恰好release一次；
8. failure、shutdown與cleanup error都不能留下active reservation；
9. Root terminal status有bounded retention；Server／Branch／Client active bookkeeping在cleanup後
   immediate removal，registry／duplicate-cleanup tombstones有bounded GC；
10. PyMTLF restart使Go移除舊generation Training routes，且不把舊route attach到新backend；
11. Go restart使PyMTLF discard舊Root／Branch／Leaf state，不resume或reconcileactive experiment；
    PyMTLF restart不resume未完成publication，也不重建publication admission fence；
12. 已完成catalog commit的current／latest model result可重新載入；`STORE_ACCEPTED`但catalog未
    commit不promotion，可能留下的ADRF orphan列為已知第一版限制；
13. stale callback、PATCH、DELETE、result與worker completion不能建立或修改新experiment；
14. active-stage missing artifact依stage造成明確failure；lifecycle-cleaned／missing URL回internal
    `404`，不partial aggregate；
15. 不新增durable ownership marker或publication pin；normal completion、shutdown與restart都不
    留下candidate retention；unused FL restart restoration helpers不再存在production surface；
16. graceful shutdown先停止publication retry再等待Root executor，所有worker／timer／monitor
    bounded停止；
17. config拒絕`workspace_root`與durable roots重疊；deployment文件要求每個PyMTLF使用獨立
    workspace，startup clear失敗時不開readiness／admission；
18. terminal status／tombstone lazy pruning與workspace opportunistic retry不需要maintenance worker；
19. containing-Go generation只有單一monitor做network refresh，admission只讀cached fence；
20. flat distributed FL behavior沒有regression；
21. PyMTLF focused／full tests與Ruff通過；
22. NWDAF focused tests、`make test`、`make build`、`make lint`通過；
23. implementation後不中斷完成mandatory review、in-scope remediation與targeted recheck；
24. plan-conformance map逐項closed，production changes依repository分開commit並回填
    implementation record。

---

## 21. Implementation record（2026-08-20–2026-08-21）

### 21.1 Production changes

- PyMTLF完成per-plan scratch artifact ownership、exact release、open-reader-safe serving、startup
  clear與workspace safety validation；terminal status、tombstone與failed cleanup採bounded lazy／
  opportunistic collection，不新增maintenance worker。
- Root、Server、Branch與Client完成outcome-aware cleanup、Branch-to-Leaf cascade、Leaf capacity exact
  release，以及generation／shutdown fences；late CREATE、PATCH、callback、result與worker completion
  不得跨generation寫回新state。
- `CUTOVER_PENDING`只保留same-process publication ownership；candidate在publication call返回後
  release，terminal private status不暴露candidate URL。Production startup不resume未完成
  publication action，completed catalog result仍由既有restore路徑載入。
- NWDAF每次`NwdafApp`生命週期產生獨立UUIDv4 `processInstanceId`，透過既有private
  `nwdaf-context`提供給PyMTLF；它不取代穩定的NF instance ID，也不新增public SBI contract。
- App-owned containing-generation monitor是唯一refresh owner；generation改變或暫時不可用時，
  PyMTLF先關閉admission並清理舊generation state，再接受新工作。

### 21.2 Mandatory review and remediation

Initial review後承認並修正的主要findings包括：workspace root安全邊界、cleanup error local-release、
Leaf capacity ownership、Branch pre-dispatch race、Server late CREATE leak、Root stale worker commit、
terminal state過早可見、final aggregate與shutdown競爭，以及candidate／published-result terminal
bookkeeping。每項修正後均加入或更新針對性deterministic test；最後再依本文件與
`development_policy.md`重新完成plan-conformance檢查。

### 21.3 Verification evidence

| Repository | Command | Result |
|---|---|---|
| PyMTLF | `.venv/bin/pytest -q` | PASS：468 passed、2 skipped |
| PyMTLF | `.venv/bin/ruff check .` | PASS：All checks passed |
| NWDAF | `make test` | PASS |
| NWDAF | `make build` | PASS |
| NWDAF | `make lint` | PASS：0 issues |
| all affected repositories | `git diff --check` | PASS |

2026-08-21再次逐項檢查completion criteria，並以timeout保護重跑先前留下background terminal
的兩組focused tests：分別為26 passed與161 passed，均正常退出；再次完整執行PyMTLF與NWDAF
verification亦通過，沒有殘留pytest process。

上述證據涵蓋completion criteria 1–23；criteria 24由PyMTLF commit `c7c66b9`、NWDAF commit
`3279891`與本implementation record關閉。因此Slice 7標記Completed；這仍不宣稱Slice 8的
real multi-process NRF／OAuth／TLS／cross-host E2E。
