# R3 Model Identity, Generation And Provision Correctness

Date: 2026-07-14

Status: Detailed plan drafted; implementation not started

Parent plan:

- `Behavioral Parity Remediation Plan.md`

Audit and decision record:

- `Behavioral Parity Audit.md`

Historical design plans:

- `Phase 2 Model Lifecycle Migration.md`
- `Phase 4 Accuracy Workflow Migration.md`

Affected repositories:

1. `PyAnLF/`
2. `NWDAF/`
3. `nwdaf-docs/`

---

## 1. Purpose

R3修正AnLF model lifecycle責任搬到PyAnLF後，model identity、generation、artifact reference、
shared runtime與provision event之間的不一致。

R3不是R1、R2那種純粹將historical Go algorithm逐步移植到Python的behavioral parity stage。
舊Go實作本身以`modelUrl`作為shared model與monitor identity；該設計依賴「新模型通常有新URL、
URL內容不原地更新」的隱含假設。PyAnLF搬移後已正式接收`ModelIdentity`與generation，但registry
仍只以URL為key，使contract與runtime state互相矛盾。

因此R3同時處理三類工作：

1. 保留historical Go已有意義的shared loading、fallback與release invariants
2. 完成model lifecycle ownership搬移後必要的identity/generation adaptation
3. 加入cross-process provision所需的atomicity與process-local idempotency

R3完成後，artifact reference只代表模型取得位置，不再決定logical model identity或active
generation。

---

## 2. Goals

1. 分離logical model identity、loaded generation與artifact locator
2. 不同identity即使使用相同URL，也不共享錯誤的identity/generation state
3. 同identity的新provision event即使沿用相同URL，也建立並載入新generation
4. model replacement在所有affected runtimes上採prepare、validate、atomic commit
5. candidate準備失敗或snapshot stale時，所有active runtimes維持原模型
6. 同一internal provision event的retry只執行一次並回傳相同response
7. 不同event ID即使payload相同，也可代表新的generation event
8. identified artifact cache以model identity、generation與完整reference分區，不再以URL最後一段作key
9. old model只在沒有任何runtime reference後才從memory unload
10. generation commit後才重設accuracy generation與舊pending predictions
11. 維持Go只負責event normalization與forwarding，model選擇和fan-out仍由PyAnLF負責
12. 以unit、concurrency、HTTP contract與cross-repository regression tests固定上述語意

---

## 3. Scope

### 3.1 Included

PyAnLF：

1. identified與anonymous model runtime registry分離
2. process-local current generation registry
3. provision event completed/in-flight registry
4. candidate prepare、stale validation與atomic metadata commit
5. deterministic lock ordering與concurrent mutation tests
6. generation-aware artifact cache key與cache concurrency protection
7. accuracy generation transition與runtime commit協調
8. provision API validation、conflict與preparation failure mapping
9. config中的bounded provision-event dedup capacity
10. model lifecycle、cache、API與concurrency tests

NWDAF：

1. internal `ModelProvisionEvent`新增required `event_id`
2. external MTLF callback normalization時建立一次UUID event ID
3. Daisy completion由stable training task ID衍生event ID
4. forwarding同一event object時保留event ID
5. contract、coordinator、MTLF、client與live-contract tests

文件：

1. 更新parent remediation status
2. 更新BP-19至BP-23 closure evidence
3. 記錄實際commits、verification與environment-level gaps

### 3.2 Excluded

1. R1 analytics shaping與R2 accuracy measurement formula
2. `maxReportNbr`、`monDur` completion callback與Go inactive transition
3. SMF/UPF collection cleanup
4. durable event registry、database-backed generation或restart recovery
5. artifact digest新增到Go/PyAnLF wire contract
6. cryptographic artifact authenticity或signature verification
7. expected-source completeness與observation transport worker重構
8. MTLF retrain policy、threshold或baseline algorithm調整
9. external 3GPP `MlEventNotif` schema新增private event ID
10. PyMTLF backend boundary
11. one-shot `Nnwdaf_AnalyticsInfo`
12. R4 reporting scheduler exception recovery

R3若需要修改上述boundary才能完成，必須依development policy停止並提出replan，不得把R4或future
work混入本stage。

---

## 4. Planning Basis

### 4.1 Current Baselines

本文件以2026-07-14 repository狀態為基準：

1. `NWDAF@195e130`
2. `PyAnLF@82f9941`
3. `nwdaf-docs@47a95f1`

### 4.2 Required Local Sources

1. workspace `AGENTS.md`
2. `nwdaf-docs/docs/development_policy.md`
3. `free5gc-dev-skill/SKILL.md`
4. `free5gc-dev-skill/references/concurrency-lifecycle.md`
5. `free5gc-dev-skill/references/testing.md`
6. current `NWDAF/`與`PyAnLF/` source/tests
7. `Behavioral Parity Audit.md` BP-19至BP-23
8. `Behavioral Parity Remediation Plan.md` R3 decisions
9. historical Go commits與tests

這條API是NWDAF Go process與AnLF backend之間的private contract，不是free5GC NF-to-NF SBI。
free5GC guidance用於NWDAF package、client、lifecycle、error與test discipline，不作為PyAnLF model
identity語意的來源。PyAnLF語意以current NWDAF contract、historical Go intent與本計畫為主。

### 4.3 Historical Oracle And Intent

| Source | Preserved intent |
| --- | --- |
| `NWDAF@c52ddcb` | 多subscription可共享同一loaded model，最後reference離開後才釋放 |
| `NWDAF@f2a8b9f` | concurrent first load只能有一個owner，其餘等待並reuse相同結果 |
| `NWDAF@f4e8dbd` | hot-swap先成功load新模型；load失敗時不破壞active model |
| `NWDAF@62e2f9f` | lifecycle ownership下沉到backend，Go不再直接執行load/unload fan-out |
| `PyAnLF@7a2ebc4` | subscription-oriented apply/release與failed replacement fallback |
| `PyAnLF@60994c3` | identity/generation進入accuracy與provision contract |

Historical Go URL-key registry只作為問題來源與相容性背景，不是R3 target key設計的oracle。

---

## 5. Change Classification

R3 implementation與review必須使用以下分類，不得把全部結果描述成historical parity：

| Behavior | Classification | R3 treatment |
| --- | --- | --- |
| Concurrent same-model first load dedup | Historical invariant | 保留並改由runtime key lock固定 |
| Replacement load failure keeps old model | Historical invariant | 保留並擴大為all-affected-runtimes guarantee |
| Last reference unloads model | Historical invariant | 保留 |
| URL作為logical identity | Historical assumption | 不保留 |
| Identity與generation分離 | Migration adaptation | 正式完成 |
| Same URL new generation reload | Migration adaptation | 正式完成 |
| Multi-runtime atomic commit | New correctness guarantee | 新增 |
| Explicit event ID與retry dedup | New cross-process guarantee | 新增 |
| Identity-aware full-reference generation cache key | Pre-existing risk correction | 新增 |

Implementation summary應分別報告這三類結果，不使用「完全按照舊Go」概括R3。

---

## 6. Current State And Confirmed Problems

### 6.1 URL-only Shared Registry

Current PyAnLF：

```text
_shared_models: dict[model_reference, SharedModelRuntime]
```

`SharedModelRuntime`雖保存`identity`與`generation`，lookup仍先由URL決定。結果是different identity
加same URL會取得第一個identity建立的runtime，污染subscription runtime與accuracy report。

### 6.2 Same-URL Update Returns False Success

`apply_provision_event()`先計算next generation，但`_get_or_load_shared_model()`發現URL已存在就
直接回傳舊runtime。response可能回`APPLIED`，實際model ID與generation沒有更新。

### 6.3 Per-runtime Mutation Is Not Atomic

Current provision loop逐一取得subscription lock並修改runtime。reporting thread或runtime query可能在
loop中間看見部分subscription已切換、部分仍使用舊generation。

### 6.4 Retry Has No Contract Identity

Current `ModelProvisionEvent`沒有event ID。相同HTTP operation重送時，backend無法區分retry與新的
same-content generation event。

### 6.5 Artifact Cache Collides

Current `ModelManager`使用URL最後一段作`artifacts/<tid>/`。不同host的相同path tail會碰撞，same URL
new generation也會直接命中舊cache。

---

## 7. Confirmed Decisions

以下方向已由parent plan與audit確認，R3實作不重新選擇：

1. identified runtime key為`(provider_id, model_unique_id, generation)`
2. artifact reference只作取得位置，不作logical identity
3. same URL update必須被支援，不能要求每generation一定有新URL
4. model digest不是本輪required contract欄位
5. provision event增加required `event_id`，不保留長期optional fallback
6. Daisy event ID由training task ID穩定衍生
7. external MTLF callback ingress產生一次UUID；外部重新送來的新HTTP callback不保證被視為duplicate
8. 同一Go forwarding operation重送時必須重用原event ID
9. idempotency只保證process lifetime，不新增persistent store
10. failed event不寫入completed registry，可使用相同event ID重試
11. candidate load失敗時所有runtime繼續使用舊模型
12. model download/load不得在global state lock內執行
13. R3不提前處理R4 scheduler completion
14. Go client對transport error、`STALE_RUNTIME_STATE`與`5xx`作bounded retry，所有attempt重用同一
    `event_id`與request body
15. identified provision event更新所有正在使用相同model identity的subscriptions；correlation另外用於
    納入尚未具有identity的matched runtime

本細部計畫補充以下implementation-level decisions：

1. `APPLIED`與`NO_MATCH`都是completed event result
2. same event ID搭配不同payload回`409 EVENT_ID_CONFLICT`，不執行第二份payload
3. payload digest只用於偵測event ID collision，不用於將不同event ID判定為duplicate
4. stale snapshot回`409 STALE_RUNTIME_STATE`，event不進completed registry
5. candidate preparation failure回non-success response，event不進completed registry
6. completed registry只淘汰最舊completed entries，不淘汰in-flight entries
7. current generation為process-local monotonic state；backend restart durability不在R3
8. `NO_MATCH`是terminal completed result，不因未來新增runtime重新套用相同event ID
9. completed event registry capacity固定預設為2048
10. identified artifact cache key包含model identity、generation與full artifact reference
11. R3不自動刪除舊generation artifact cache；disk GC另列future work
12. Go retry使用單一120秒operation budget、最多3個attempt及1秒/2秒backoff

---

## 8. Internal Contract Cutover

### 8.1 Request Shape

Go與Python的private contract同步改成：

```text
ModelProvisionEvent {
  event_id
  source
  model_identity
  model_update_ind
  artifact
  analytics_event
  notification_correlation
  training_task_id
}
```

`event_id`：

1. required non-empty string
2. 使用snake_case JSON field，與current internal payload一致
3. 不加入3GPP `MlEventNotif`
4. 不由artifact URL或整份payload fingerprint產生

Response維持：

```text
ModelProvisionEventResponse {
  status
  affected_runtime_count
  generation
}
```

成功status只有`APPLIED`與`NO_MATCH`。duplicate success回傳首次response的完整內容，包括原generation與
affected count。

### 8.2 Event ID Sources

| Source | Event ID rule |
| --- | --- |
| Daisy retrain completion | `daisy:<training_task_id>` |
| External MTLF callback normalized event | `mtlf:<uuid>`，在`PlanModelProvisionActions`建立action時產生一次 |
| Same in-memory action retry | 重用action內既有`event_id` |
| New external callback HTTP request | 新UUID；不保證cross-request dedup |

一個callback body若有多個event notifications，每個normalized `ModelProvisionEvent`取得不同event ID。

### 8.3 Validation And HTTP Mapping

1. missing/empty `event_id`：request validation failure，不進runtime manager
2. missing identity或invalid model ID：request validation failure
3. missing artifact reference：`400`
4. same event ID/different payload：`409 EVENT_ID_CONFLICT`
5. stale runtime set或generation：`409 STALE_RUNTIME_STATE`
6. artifact download/model load failure：`503 MODEL_PREPARATION_FAILED`
7. `APPLIED`與`NO_MATCH`：`200`

Go client保持所有non-`200`為error的既有boundary，但error text需保留status與operation context。R3不新增
Go-side fallback fan-out。Go contract/client在送出HTTP前也必須拒絕空`event_id`，避免把已知無效的
internal event送到backend。

---

## 9. Target Runtime State

本文件中的registry都是PyAnLF process內由lock保護的in-memory map，不是NRF、外部registry service或
database。它們的用途與lifetime如下：

| Registry | Key | Purpose | Lifetime/capacity |
| --- | --- | --- | --- |
| `shared_models` | model identity + generation | 找到已載入model與使用它的subscriptions | 無reference時移除model entry |
| `current_generation` | model identity | 記住該logical model最新成功generation | process lifetime |
| `anonymous_models` | full artifact reference | 支援缺少identity的legacy inference model sharing | 無reference時移除model entry |
| completed provision events | event ID | duplicate retry回原response，避免重複換model | process lifetime，最多2048筆completed entries |
| in-flight provision events | event ID | concurrent duplicate只讓一個caller執行，其餘等待 | operation完成後移除，不受2048限制 |

只有completed provision event registry使用`2048`容量限制。這個數字不限制loaded models、active
subscriptions或generation數量。

### 9.1 Identified Models

Internal immutable keys：

```text
ModelIdentityKey = (provider_id, model_unique_id)
ModelRuntimeKey  = (provider_id, model_unique_id, generation)
ArtifactLocator  = model_reference
```

PyAnLF state：

```text
current_generation: dict[ModelIdentityKey, int]
shared_models: dict[ModelRuntimeKey, SharedModelRuntime]
```

`SharedModelRuntime`保存：

1. `runtime_key`
2. `model_reference`
3. process-local `model_id`
4. `subscription_ids`
5. temporary reservation/preparation state only if implementation needs it

`SubscriptionRuntime`保存：

1. `model_identity`
2. `model_generation`
3. `model_reference`
4. `model_id`

runtime必須能由identity/generation直接找到shared entry；不得再用reference作主要lookup。

### 9.2 Anonymous Artifact Runtimes

Current contract仍允許沒有stable model identity的legacy provision/default activation。已確認行為是可執行
inference，但不啟動accuracy monitoring。

R3將這條路徑隔離：

```text
anonymous_models: dict[full_model_reference, SharedModelRuntime]
```

規則：

1. anonymous runtime可維持historical same-reference sharing
2. anonymous runtime沒有formal generation，不寫入`current_generation`
3. anonymous runtime不參與identity-based accuracy monitor
4. identified provision event可依correlation將anonymous runtime升級到identified runtime
5. anonymous registry不得回傳或覆寫identified runtime的identity
6. local default config已有identity時必須走identified registry，不走anonymous fallback

這是legacy compatibility boundary，不是第二套model identity制度。未來若contract要求所有模型都有identity，
可另行移除anonymous path。

### 9.3 Generation Rules

1. identity第一次成功activation使用generation 1
2. 每個新的成功provision event使用`current_generation + 1`
3. `model_update_ind`不作唯一generation gate；event identity才區分retry與新事件
4. failed prepare、stale commit與event ID conflict不增加generation
5. duplicate completed event回原generation，不增加generation
6. same content/different event ID可建立下一generation
7. current generation在最後runtime release後仍保留到process結束，避免同process重用舊generation number
8. backend restart後generation durability與runtime reconcile不在R3保證範圍

如果identified runtime key已存在但artifact reference不同，視為internal state conflict，不得靜默reuse。

---

## 10. Provision Event State Machine

### 10.1 Phase A: Idempotency Admission

1. validate event ID與payload
2. 計算不含`event_id`的canonical payload digest，只用於collision detection
3. completed registry命中且digest相同：立即回原response
4. completed registry命中但digest不同：回event ID conflict
5. in-flight registry命中且digest相同：等待owner完成並取得相同result/error
6. in-flight registry命中但digest不同：立即回event ID conflict
7. 未命中：註冊為in-flight owner

### 10.2 Phase B: Resolve Snapshot

在短暫`_state_lock`內：

1. 計算affected runtime IDs，其集合為「所有identity等於event identity的runtimes」聯集「binding
   correlation明確匹配的runtimes」
2. 保存每個runtime的object/revision/model assignment snapshot
3. 保存affected ID set
4. 保存identity current generation
5. 計算candidate generation

若沒有affected runtime，建立`NO_MATCH` response並完成event，不下載模型。

### 10.3 Phase C: Prepare Candidate

在global state lock與subscription locks之外：

1. 以identity、candidate generation與artifact reference建立candidate descriptor
2. 透過ModelManager取得generation-specific artifact cache
3. 載入新process-local model instance
4. 不修改active runtime、current generation、shared registry或accuracy monitor

same URL new generation仍必須執行新的load attempt。prepare失敗時卸載任何不完整candidate，active state
不變，event可用同一ID重試。

### 10.4 Phase D: Validate And Atomic Commit

1. 將affected subscription IDs映射到striped lock index
2. 去重後依固定index由小到大取得locks
3. 取得`_state_lock`
4. 重新計算目前affected ID set
5. 驗證affected set與snapshot完全相同
6. 驗證每個runtime object/revision/model assignment仍符合snapshot
7. 驗證identity current generation仍等於snapshot
8. 任一驗證失敗則不修改任何runtime，釋放並unload candidate，回stale conflict
9. 全部通過後一次更新所有runtime model assignment
10. 將candidate加入`shared_models[ModelRuntimeKey]`
11. 更新`current_generation`
12. 從old shared entries移除affected subscription references
13. 原子切換new identity accuracy generation與runtime membership

Public runtime revision不因model provision自動增加。該revision目前用於observation binding resync；R3只更新
model assignment snapshot，不讓model replacement錯誤觸發collection resync。

### 10.5 Phase E: Post-commit Cleanup

在state lock之外：

1. 建立`APPLIED` response
2. 將response寫入completed event registry
3. 喚醒等待同event ID的concurrent callers
4. detach不再使用的old identity accuracy memberships
5. stop任何已無runtime的old accuracy worker
6. unload沒有subscription reference的old model IDs
7. 記錄structured success log

Event completion必須先於可能失敗的old-model cleanup。commit後的cleanup failure只記錄error並保留
`APPLIED`結果，不得讓相同event ID重做generation commit。

old artifact cache目錄可保留；R3只保證memory model依reference count unload。

---

## 11. Locking And Concurrency Model

### 11.1 Lock Order

需要同時取得多個locks時固定使用：

```text
subscription stripe locks in ascending index
  -> runtime manager _state_lock
    -> accuracy registry metadata lock for non-blocking generation commit only
```

禁止：

1. 持有`_state_lock`下載或載入model
2. 持有`_state_lock`執行HTTP callback
3. 持有`_state_lock`等待accuracy worker join
4. 以subscription ID字串排序但忽略striped lock collision
5. 取得高index stripe後再取得低index stripe

### 11.2 Regular Apply And Release

Existing single-subscription apply/release維持subscription stripe後state lock的順序。identified first-load
dedup改用`ModelRuntimeKey` striped lock；anonymous first-load使用full reference striped lock。

Concurrent same-key load應保留historical `WaitLoaded`語意：只有一個loader，其他caller等待後reuse成功結果或
取得相同failure，不可建立兩個active model instances。

### 11.3 Concurrent Different Provision Events

不同event ID不以payload dedup。若兩個event同時針對同identity：

1. 兩者可在lock外prepare candidate
2. 第一個通過snapshot validation者commit
3. 第二個因current generation/runtime snapshot stale而整體失敗
4. 第二個使用相同event ID重試時重新resolve並取得下一generation

此策略允許stale candidate產生一次額外load，但避免下載期間持有global lock，也避免partial commit。

### 11.4 Accuracy Transition

Accuracy registry需提供不執行I/O、不join worker的atomic metadata operation，完成：

1. new identity generation reset
2. old generation pending prediction discard
3. committed subscription membership更新

runtime manager在model metadata commit後才能公開新generation。若old identity需要停止worker，join放到
post-commit cleanup，不在runtime state lock內執行。

---

## 12. Provision Event Registry

### 12.1 State

建議新增`core/provision_events.py`：

```text
completed[event_id] = {
  payload_digest,
  response
}

in_flight[event_id] = {
  payload_digest,
  completion signal,
  result or error
}
```

### 12.2 Capacity

PyAnLF config新增：

```yaml
model:
  provision_event_dedup_capacity: 2048
```

規則：

1. 必須大於0
2. 只限制completed registry
3. 超過容量時淘汰最舊completed entry
4. in-flight entry不因capacity被淘汰
5. eviction後舊event重送可能被視為新事件，屬bounded process-local guarantee

### 12.3 Failure Semantics

1. success與`NO_MATCH`保存response
2. validation、prepare、stale與internal failure不進completed registry
3. 同時等待的duplicate caller取得owner相同failure
4. owner完成後in-flight entry一定清除
5. unexpected exception不能讓waiter永久阻塞
6. shutdown不新增durable drain語意

Registry不得使用artifact URL或payload digest跨event ID去重，避免same-content new generation被誤判為retry。

---

## 13. Artifact Cache Design

### 13.1 HTTP/HTTPS Cache Key

Identified model使用：

```text
cache_key = sha256(canonical_model_identity + generation + full_artifact_reference)
cache_dir = artifacts/<cache_key>/
```

Hash input必須使用無歧義的length-prefix或canonical serialization，不可直接以未分隔字串串接。

結果：

1. 不同host但相同path tail不碰撞
2. same URL generation 1與generation 2不共用舊cache
3. different identity即使使用same URL與相同generation number也不共用cache
4. duplicate同identity、同generation load可reuse同cache
5. cache directory不再依賴Daisy TID一定是URL最後一段

### 13.2 Anonymous Cache Key

Anonymous runtime沒有formal generation，使用full-reference hash加固定anonymous namespace。same reference
維持existing cache reuse；anonymous path不提供same-URL content update保證。

### 13.3 Local References

`file://`與無scheme local directory仍直接resolve本地目錄，不複製到HTTP artifact cache。identified new
generation仍建立新的in-memory model instance並重新讀取該目錄。

### 13.4 Cache Concurrency

ModelManager需依cache key序列化download/extract。download使用temporary file/directory；失敗只清理該key
的不完整內容，不刪除其他generation cache。

R3不刪除舊cache、不實作disk quota或digest verification。這些屬artifact lifecycle future work。

---

## 14. NWDAF Event Normalization

### 14.1 External MTLF Callback

`internal/anlf/coordinator/model_provision.go`在將有model identity的`MlEventNotif`轉為
`ModelProvisionEvent`時：

1. 產生`mtlf:<uuid.NewString()>`
2. 保存於`ModelProvisionAction.Event`
3. `ExecuteModelProvisionActions`只轉送既有event，不重新產生ID
4. 同一action若由caller重試，ID保持不變

沒有model identity的legacy callback仍走subscription runtime apply path，不強行建立identified provision
event。

### 14.2 Daisy Completion

`internal/mtlf/training.go`建立：

```text
event_id = "daisy:" + taskID
```

同一training completion forwarding retry必須重用該值。`training_task_id`欄位仍保留作correlation，不用來
取代required `event_id`。

### 14.3 Client Boundary

`internal/anlf/client/model_provision.go`：

1. 不產生或改寫event ID
2. marshal caller提供的完整event
3. 整個forwarding operation共用caller context與120秒operation budget
4. 最多3個attempt，即initial attempt加2次retry；backoff為1秒、2秒並可被context取消
5. transport error、response detail為`STALE_RUNTIME_STATE`的`409`、以及`5xx`可retry
6. validation failure、`EVENT_ID_CONFLICT`與其他永久性`4xx`不retry
7. 所有attempt重用相同serialized request body與event ID
8. non-success error保留HTTP status與backend detail

---

## 15. Planned Code Changes

### 15.1 PyAnLF

| File | Planned change |
| --- | --- |
| `src/py_anlf/models.py` | `ModelProvisionEvent.event_id` required validation；必要的status typing |
| `src/py_anlf/core/provision_events.py` | bounded completed/in-flight registry、collision detection與waiter coordination |
| `src/py_anlf/core/runtime_manager.py` | identified/anonymous registries、generation state、snapshot/prepare/atomic commit與cleanup |
| `src/py_anlf/core/model_manager.py` | identity-aware full-reference generation cache key、cache locking與new load signature |
| `src/py_anlf/core/accuracy/monitor.py` | non-blocking atomic generation/membership transition seam |
| `src/py_anlf/sbi/routers/analytics.py` | provision conflict與preparation failure HTTP mapping |
| `config/config.yaml` | `provision_event_dedup_capacity` default |
| `tests/test_provision_events.py` | sequential/concurrent duplicate、collision、failure、eviction |
| `tests/test_runtime_manager.py` | identity/generation、atomic replacement、fallback、refcount與concurrency |
| `tests/test_model_manager.py` | cache collision、same-URL generation與partial download cleanup |
| `tests/test_lifecycle_api.py` | required event ID、duplicate response與HTTP errors |

若`runtime_manager.py`仍可讀，R3不為每個dataclass建立額外module。只有idempotency registry因具有獨立
concurrency lifecycle與tests，規劃為獨立`provision_events.py`。

### 15.2 NWDAF

| File | Planned change |
| --- | --- |
| `internal/anlf/contract/model_provision.go` | required `EventID` JSON field與internal validation helper |
| `internal/anlf/coordinator/model_provision.go` | external MTLF normalized event UUID |
| `internal/anlf/coordinator/model_provision_test.go` | event ID presence、uniqueness與action reuse |
| `internal/mtlf/training.go` | Daisy deterministic event ID |
| `internal/mtlf/training_test.go` | task ID derivation與retry stability |
| `internal/anlf/client/model_provision.go` | 送出前驗證event ID；同body bounded retry與backend error classification |
| `internal/anlf/client/backend_live_test.go` | real HTTP duplicate event response/generation check |
| Existing fake clients/tests | 補required event ID並驗證payload不被改寫 |

R3不新增Go factory config，不把PyAnLF dedup capacity搬到NWDAF config。

---

## 16. Implementation Sequence

### Step 1: Establish R0/R3 Failure Evidence

1. 新增BP-19至BP-23最小reproduction tests
2. 證明current different identity/same URL污染
3. 證明current same identity/same URL不reload
4. 證明current duplicate event增加或錯誤沿用generation
5. 證明current per-runtime fan-out沒有atomic stale guard
6. 保存initial failure evidence；正式commit保持green

### Step 2: Cut Over Internal Event Contract

1. Go `ModelProvisionEvent`加入required event ID
2. external MTLF與Daisy建立stable per-operation IDs
3. Python Pydantic contract同步required field
4. Go client加入同event/body bounded retry與error classification
5. 更新兩端fixtures與API tests
6. 不保留missing-ID fallback

此step是cross-repository deployment pair；不能只部署其中一端。

### Step 3: Add Process-local Idempotency

1. 建立completed/in-flight registry
2. 實作same-ID same-payload replay
3. 實作same-ID different-payload conflict
4. 實作concurrent duplicate waiter
5. 實作failure removal與bounded eviction
6. 將registry包在provision operation最外層

### Step 4: Separate Runtime Identity

1. 新增identity/runtime key helpers
2. 建立`current_generation`
3. 改identified shared registry key
4. 隔離anonymous artifact registry
5. 修改regular apply/release/detach lookup
6. 保留concurrent first-load dedup與last-reference unload

### Step 5: Implement Candidate Transaction

1. snapshot affected runtimes與generation
2. lock外load candidate
3. 固定stripe順序取得affected locks
4. re-resolve affected set並完整validate
5. 一次commit runtime/shared/generation metadata
6. commit後處理accuracy cleanup與old unload
7. stale/failure路徑卸載candidate並保持old state

### Step 6: Fix Artifact Cache

1. 改full-reference generation key
2. 加cache-key concurrency lock
3. 保留local directory behavior
4. 補host collision與same-URL new generation tests
5. 確認failed extraction只清理candidate directory

### Step 7: Cross-repository Regression And Review

1. 執行PyAnLF focused/full tests
2. 執行NWDAF focused/full tests、race、build與lint
3. 啟動local PyAnLF時執行real HTTP duplicate contract
4. review lock order、stale path、reference counts與failure cleanup
5. 更新audit、parent plan與本文件progress record
6. 三個repository分別commit，不混合history

---

## 17. Test Plan

### 17.1 PyAnLF Identity And Generation

1. different identity/same URL建立不同runtime keys與model instances
2. same identity/same URL/new event建立next generation並再次load
3. same identity/new URL建立next generation
4. same identity/current generation/same reference regular apply可reuse
5. same runtime key/different reference拒絕state conflict
6. generation只在successful commit增加
7. last runtime release unload model但current generation保留
8. anonymous same-reference sharing維持
9. anonymous runtime不啟動accuracy monitor
10. identified event可將correlated anonymous runtime升級
11. event更新所有相同identity runtimes，即使只有其中一個binding直接匹配correlation
12. unrelated identity且correlation不匹配的runtime不受影響

### 17.2 Candidate And Atomicity

1. candidate load failure保留全部old runtimes
2. one affected runtime revision changes during load使整批stale
3. affected runtime新增或release during load使整批stale
4. stale event不更新generation、accuracy或shared references
5. stale candidate model被unload
6. success時所有affected runtimes一次切換
7. old model仍有unaffected reference時不unload
8. old model無reference時只unload一次
9. scheduler/report read看不到partial fan-out state
10. concurrent different events只有一個commit，另一個取得stale conflict

### 17.3 Event Idempotency

1. sequential duplicate只load/commit一次並回相同response
2. concurrent duplicate只由一個owner執行
3. duplicate `NO_MATCH`回相同response且不重新resolve
4. same event ID/different payload回conflict
5. same payload/different event ID可建立下一generation
6. prepare failure不cache，same ID retry可成功
7. stale failure不cache，same ID retry可重新resolve
8. capacity eviction只移除oldest completed
9. in-flight entry不被capacity eviction
10. unexpected owner exception會喚醒waiters並清除in-flight

### 17.4 Artifact Cache

1. different hosts/same path tail使用不同cache directories
2. same URL generation 1與2使用不同cache directories
3. different identity/same URL/same generation number使用不同cache directories
4. same identity/same URL/same generation cache hit
5. concurrent same cache key只download/extract一次
6. failed download/extract清除不完整candidate directory
7. local path/file URL仍直接讀取

### 17.5 NWDAF

1. external identity-bearing callback產生non-empty `mtlf:` event ID
2. callback body內多個events取得不同IDs
3. same planned action execution重用ID
4. Daisy completion產生`daisy:<taskID>`
5. Daisy forwarding failure後使用同一completion event retry時ID不變
6. Go client JSON包含event ID且不改寫
7. Go client拒絕empty event ID且不發HTTP request
8. transport、stale與`5xx` retry重用完全相同event ID/body
9. permanent `4xx`不retry
10. existing model identity、artifact與correlation欄位維持
11. missing identity legacy callback仍走runtime apply path

### 17.6 HTTP Contract

1. missing event ID被拒絕
2. first event回`APPLIED`與generation N
3. same event重送回相同generation N
4. model load call count維持1
5. same payload/new event ID回generation N+1
6. same event ID/different payload回`409`
7. stale commit回`409`且old runtime可繼續report
8. preparation failure回non-success且retry可成功
9. 首次response遺失但backend已commit時，Go retry取得cached original response且不增加generation

---

## 18. Verification Commands

實作時依實際test file與test name微調，但最低執行：

```bash
cd PyAnLF
uv run pytest -q tests/test_provision_events.py
uv run pytest -q tests/test_model_manager.py tests/test_runtime_manager.py tests/test_lifecycle_api.py
uv run pytest -q
```

```bash
cd NWDAF
go test ./internal/anlf/... ./internal/mtlf/...
go test -race ./internal/anlf/... ./internal/mtlf/...
make test
make build
make lint
```

Local cross-process contract：

```bash
cd NWDAF
PYANLF_LIVE_ENDPOINT=http://127.0.0.1:9090 go test ./internal/anlf/client -run Live
```

依workspace policy，test/script與local service execution使用elevated permission。若環境無法啟動PyAnLF，
必須明確記錄`environment-level HTTP E2E unverified`；不得將unit/FastAPI test描述成完整cross-process
驗證。

---

## 19. Observability

PyAnLF最低需記錄：

1. provision accepted：event ID、source、identity、candidate generation、affected count
2. duplicate replay：event ID與original status，不重複記artifact payload
3. event ID conflict：event ID與safe identity summary
4. candidate preparation failure：event ID、identity、generation與error
5. stale commit：event ID與stale原因分類
6. successful commit：event ID、identity、old/new generation、affected count
7. old model unload failure或unexpected missing shared runtime

不得在log輸出完整subscriber payload、credentials或artifact response body。URL可依現有model lifecycle log
policy輸出必要定位資訊，但不重複在每一層記錄相同error。

最低process-local counters：

1. provision event duplicate total
2. provision event conflict total
3. provision prepare failure total
4. provision stale commit total
5. provision success total

若專案尚無metrics exporter，先以runtime stats seam與structured logs供tests/觀察，不在R3引入新metrics server。

---

## 20. Risks And Controls

### 20.1 Deadlock Across Subscription Stripes

Risk：multi-runtime commit與normal apply/release以不同順序取locks。

Control：依實際stripe index排序並去重；加入反向subscription ID與hash collision concurrency tests；禁止在
state lock內等待低階lock。

### 20.2 Candidate Resource Leak

Risk：load成功後snapshot stale或commit exception，candidate model留在memory。

Control：candidate在commit前不進shared registry；所有non-commit exit使用單一cleanup path unload。

### 20.3 Accuracy Generation Window

Risk：runtime已公開新generation但accuracy monitor仍是舊generation，造成短暫prediction drop。

Control：提供non-blocking atomic accuracy metadata transition並納入runtime commit lock hierarchy；worker join與
I/O延後。

### 20.4 Bounded Registry Eviction

Risk：很久以前的event在entry淘汰後重送會再次執行。

Control：清楚定義bounded process-local guarantee、提供capacity config與eviction counter；durable dedup不在本輪。

### 20.5 New External Callback Retry

Risk：external MTLF重送整個3GPP callback時Go建立新UUID，可能形成新generation。

Control：這是已確認限制；不以content fingerprint猜測duplicate，避免壓掉合法same-content update。若部署證明
external callback具stable identifier，再另行擴充normalization。

### 20.6 Cache Growth

Risk：每generation使用新cache directory，disk使用量持續增加。

Control：R3保證正確性但不刪cache；disk quota與artifact garbage collection列future work。

### 20.7 Restart Generation Reset

Risk：PyAnLF restart後process-local generation與dedup state消失。

Control：文件不宣稱durability；完整restart reconcile需要runtime resync/persistent state，另行規劃。

---

## 21. Blocker And Replan Conditions

遇到以下情況必須停止實作並提出decision：

1. current apply/release lock order無法在不重構整個runtime manager下安全加入multi-lock commit
2. model download/load必須持有global state lock才能避免duplicate
3. accuracy generation無法與runtime commit協調而會產生可重現prediction loss
4. external MTLF callback其實已有stable event identifier，與目前UUID決策衝突
5. same URL artifact server無法提供generation-specific內容或cache bypass
6. current ModelManager load API無法安全清理stale candidate
7. anonymous runtime不能在不破壞existing default/legacy activation下隔離
8. required contract change影響3GPP-facing payload或generated OpenAPI model
9. R3實作必須加入R4 completion callback才能避免資料損壞
10. required concurrency/live-contract tests因environment或dependency無法建立

Blocker report必須包含原假設、實際證據、可行選項、建議與plan是否需要先修訂。

---

## 22. Completion Criteria

R3只有在以下全部成立時才能標記repository-level completed：

1. BP-19至BP-23都有對應tests與closure evidence
2. identified shared runtime不再以URL-only registry決定
3. different identity/same URL不污染identity或generation
4. same identity/same URL/new event確實reload並切到next generation
5. duplicate internal event不重複load、commit或增加generation
6. same event ID/different payload被拒絕
7. same payload/different event ID可形成next generation
8. candidate failure與stale snapshot保留全部old runtimes
9. multi-runtime commit不產生partial externally observable state
10. 同identity runtimes一起更新，unrelated identity不受影響
11. old model依reference count正確unload
12. accuracy generation只在successful model commit後切換
13. identity-aware full-reference generation cache collision tests通過
14. anonymous compatibility path與identified registry隔離
15. Go與Python required `event_id`同一cutover完成
16. Go bounded retry重用同一event ID/body且不retry permanent `4xx`
17. external MTLF與Daisy event ID tests通過
18. PyAnLF focused與full tests通過
19. NWDAF focused/full/race/build/lint通過
20. `git diff --check`在三個repository通過
21. 完整code review沒有未處理P0/P1 finding
22. parent plan、audit與本文件progress record已更新

若local cross-process HTTP test未執行，可完成repository-level implementation，但文件必須維持
`environment-level HTTP E2E unverified`，不得宣稱完整部署驗證。

---

## 23. Implementation Record

尚未開始。

完成後至少記錄：

1. NWDAF commit
2. PyAnLF commit
3. docs progress commit
4. initial failure evidence
5. focused/full/race/build/lint結果
6. live HTTP contract結果或environment gap
7. code review findings與closure
8. 任何經核准的deviation或remaining risk
