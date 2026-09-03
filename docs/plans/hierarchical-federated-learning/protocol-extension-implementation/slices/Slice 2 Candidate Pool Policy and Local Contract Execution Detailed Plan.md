# Slice 2 — Candidate Pool, Policy and Local Contract Execution Detailed Plan

日期：2026-09-04

狀態：Committed／`PyMTLF` production commit `0e87ef1`，review evidence已收尾

相關文件：

- [Protocol Extension Implementation Plan](../Hierarchical%20NWDAF%20FL%20Protocol%20Extension%20Implementation%20Plan.md)
- [Protocol Extension Implementation Slice Map](../Protocol%20Extension%20Implementation%20Slice%20Map.md)
- [Protocol Implementation Current-State Inventory](../Protocol%20Implementation%20Current-State%20Inventory.md)
- [Model Bundle Metadata to Protocol Schema Mapping](../Model%20Bundle%20Metadata%20to%20Protocol%20Schema%20Mapping.md)
- [Protocol Resource Lifecycle and Wire Integration Mapping](../Protocol%20Resource%20Lifecycle%20and%20Wire%20Integration%20Mapping.md)
- [Protocol Conformance Case Ownership Mapping](../Protocol%20Conformance%20Case%20Ownership%20Mapping.md)
- [Candidate OpenAPI Schema](../../../../design/hierarchical-federated-learning/candidate_openapi_schema.md)
- [Candidate OpenAPI artifact](../../../../design/hierarchical-federated-learning/candidate_openapi.yaml)
- [Topology、Policy 與 Strategy 細節設計](../../../../design/hierarchical-federated-learning/topology_policy_design.md)
- [Protocol Conformance Matrix](../../../../design/hierarchical-federated-learning/protocol_conformance_matrix.md)
- [Protocol Extension Implementation Review Ledger](../Protocol%20Extension%20Implementation%20Review%20Ledger.md)
- [Legacy Model-bundle Slice 8](../../hierarchical-fl-model-bundle-edition/Slice%208%20Multi-process%20E2E%20and%20Regression%20Closure%20Detailed%20Plan.md#188-late-code-review-findinghierarchy-assignment-duplicate-get)

---

## 1. Slice 結果

本 slice 完成後，`PyMTLF` 應具有一組不依賴 legacy hierarchy assignment bundle 的
local orchestration primitives，能夠：

- 將接收 node 的 `x-flTopology` 解析成 direct-child candidate pool；
- 分開保存 `upstream-assigned` 與 `locally-discovered` provenance；
- 使用既有 training requirements 與 Go internal NRF proxy 補充 delegated
  candidates；
- 保存NRF discovery result的有效期，並在使用過期的NRF-derived candidate前要求
  refresh；
- 解析實際採用的 node-local `policy`、`strategy` 與 `reportAfter`；
- 依 readiness、每輪 selection 與 completion policy 決定可否進入既有 FL
  execution；
- 將 selected participants 交給既有 `FLServerEngine`、`FederatedTrainer` 與
  sample-weighted aggregation path，不建立第二套 trainer 或 aggregator；
- 對 topology update 產生 establishment／DELETE intents，並維護 direct-child
  relationship status；
- 產生可供 Slice 4 放入 `x-flTopologyReport` 的 realized status snapshot；
- 修正 legacy hierarchy assignment preparation 的單次取得與 artifact ownership，
  使同一個 logical assignment 不再因 generic 與 typed validation 各自擁有
  transport 而下載兩次。

本 slice 不把這些 primitives 接上 Root→Branch→Leaf 的 production message flow，也不
開啟 feature 3。完成本 slice只表示 local execution behavior已有 deterministic
contract與tests，不表示 protocol-driven hierarchical FL已可跨NWDAF執行。

---

## 2. 基準與直接證據

### 2.1 盤點版本

| Repository | Revision | Slice 2 角色 |
| --- | --- | --- |
| `PyMTLF/` | `6c27d1581bbee91397aead12b2fa49c750f4ea3a` | Candidate pool、policy executor、NRF resolver與既有local FL execution owner |
| `NWDAF/` | `302762a6af677f5ccfb5a3f9d0253fb3dd39bf62` | Read-only dependency；提供既有Go internal NRF discovery proxy |
| `nwdaf-docs/` | `871a3846fd3fbb190474afdce5b8cbd9b6208e62` 加上本次unstaged evidence | Canonical design、slice plan與review evidence |

Production implementation開始前必須再次確認`PyMTLF/`與`NWDAF/`的HEAD及工作樹。
若resolver、server或Slice 1 resource contract已改變，先更新本計畫的exact-file與
boundary mapping。

### 2.2 現有 PyMTLF production baseline

- `wire/ml_model_training.py` 已由Slice 1提供typed `FlTopologyNode`、`FlPolicy`、
  `FlStrategy`、`FlReportAfter`與`FlTopologyReport` models及cross-field validation。
- `FLClientEngine` 已保存candidate persistent contract；feature 3尚未協商時，resource
  停在idle `READY`且不進入legacy artifact execution。
- `HierarchyNodeResolver.resolve()` 只能依explicit `nfInstanceId`解析單一Branch或Leaf；
  尚無省略exact identity的list-discovery。
- 目前Slice 2 working-tree implementation雖已新增list-discovery，但
  `discover()`／`discover_for_subscription()`只回傳resolved candidates，未保留
  `SearchResult.validityPeriod`，candidate pool也無法判斷NRF-derived資訊是否已過期。
- `FLBranchPreparationCoordinator`仍從legacy assignment bundle取得完整Leaf list，先
  resolve全部Leaves，再一次建立所有lower-tier subscriptions。任一Leaf在
  discovery或assignment publication失敗時，整組會立即失敗；尚未嘗試的其他
  Leaves也會被報成`NOT_AVAILABLE_ML_TRAIN`。這不能直接沿用為新protocol
  mode的per-node status，必須改由candidate lifecycle區分`UNCONFIRMED`、
  `DEPLOYING`與已確認的failure。
- Legacy review finding `S8-R5` 仍未修正；`FLClientEngine._run_preparation()` 在
  hierarchy assignment path 先以
  `FLWorkspace.download()` 取得並以 `inspect_artifact()` 辨識 archive，接著對同一
  URL 呼叫 `download_assignment()`；後者在 `_download_hierarchy()` 再發出一次
  HTTP GET。這會讓 Root→Branch 與 Branch→Leaf 的 assignment bytes 被實作
  額外放大，Flat preparation 不走此分支。
- 同一路徑會重複驗證 archive，並在刪除第一份下載檔後，將舊
  `ArtifactMetadata` 傳給 `claim_artifact()`來登記該空目錄的cleanup；真正被
  使用的typed artifact則已由`_download_hierarchy()`另行登記。這個補償式
  bookkeeping是duplicate ingress造成的冗餘，不應在single-fetch路徑中保留。
- 現有 tests 將 `download()`、`inspect_artifact()` 與 `download_assignment()` 分別
  mock，只驗證 helper 被呼叫，沒有對真實 HTTP transport 計數，因此未抓到
  duplicate GET。
- `FLServerEngine`的hierarchy preparation與round state固定管理一份participant list；
  round dispatch、wait與aggregation目前都要求全部participants完成。
- Legacy Branch從upper input bundle的`client_training.epochs`取得lower-tier local
  work指示，Leaf則從hierarchy assignment metadata取得FedProx參數。Protocol mode
  不得繼續以artifact metadata作為這些決策的authority；本slice的effective
  `strategy`／`reportAfter`需要直接供給既有executor。
- `FederatedTrainer.train()`已接受epochs與FedProx `proximal_mu`，
  `FederatedTrainer.aggregate()`已依sample count做weighted aggregation。
- Legacy hierarchy將upper/lower process、round、artifact與cleanup完整串起；本slice
  必須保留該回歸，不能用candidate primitive取代尚未完成的Slice 4 wiring。

### 2.3 Go→NRF boundary evidence

`NWDAF/internal/backend/nrf_discovery.go`與`internal/mtlf/api_nf_discovery.go`已確認：

- `/internal/v1/nrf/nf-instances`的NWDAF query可省略
  `target-nf-instance-id`；
- query可帶`service-names=nnwdaf-mlmodeltraining`與
  `ml-analytics-info-list`；
- `ml-analytics-info-list`可表達analytics ID、FL capability、model
  interoperability與tracking area；
- Go consumer會將這些條件傳給NRF並回傳SearchResult；
- Release 18 `SearchResult.validityPeriod`是必填秒數，表示discovery result可被NF
  Service Consumer視為有效並快取的期間；HTTP `Cache-Control: max-age`應使用相同值；
- 相同query在有效期內應重用先前結果。現有Go consumer已按normalized query key快取，
  cache hit時回傳剩餘`validityPeriod`，過期後才重新向NRF查詢；
- 既有proxy負責query shape與standard-shaped error，不負責hierarchy policy、candidate
  provenance或selection。

因此Slice 2只延伸PyMTLF resolver並增加boundary characterization tests，不修改
`NWDAF/`或`nrf/`。若實作時發現必要query無法穿越既有proxy，必須回到boundary
decision，不可在PyMTLF繞過containing Go NWDAF直接連NRF。

### 2.4 Design與conformance evidence

- `allowAdditionalCandidates`省略時為`false`；
  `additionalCandidatePriority`省略時為`0`。
- 其餘未指定policy fields由接收node的local decision補足，並在report回報實際值。
- `policy`與`strategy`只管理該node作為Server時的direct-child local FL process；
  `reportAfter`只管理該node對direct parent的回報，不向descendants自動繼承。
- Readiness、selection與completion分別由`minAvailableNodes`、
  `fractionTrain`／`minTrainNodes`、`acceptFailures`／`minCompletionRate`決定。
- PATCH的`children` array是JSON Merge Patch replacement；只取代
  upstream-assigned set，不清除locally-discovered set。
- `enabled: false`是明確prohibition；`priority: 0`不是刪除指令。
- Report-side未知status／cause可保存與逐級轉送，但不得被推測為`ACTIVE`或觸發已知
  recovery action。

### 2.5 Candidate、participant與round cohort語意

- `candidate`是可能成為direct child的NWDAF。它可能由上層明確指定或由本node從NRF
  discovery取得，但尚不表示downstream subscription／preparation已成功；candidate
  record可處於`UNCONFIRMED`、`DEPLOYING`、`FAILED`或`INACTIVE`。
- `participant`是已完成downstream subscription／preparation、relationship狀態為
  `ACTIVE`的direct child。
- `selected participant`或`round cohort`是從active participant set中被選入特定一輪的
  frozen identity set。Active但本輪未被選中的node仍是participant。
- 單輪training failure只影響該round outcome；除非local coordinator另外確認
  subscription／relationship失效，不得直接把participant relationship改成`FAILED`。
- NRF `validityPeriod`管理的是discovery-derived candidate／endpoint資訊的新鮮度，不是
  已建立之active participant relationship的存活期限。

---

## 3. Slice 邊界

### 3.1 納入

- PyMTLF local candidate record、pool、provenance與relationship state。
- Effective node contract resolution與local-default injection boundary。
- Explicit、delegated與hybrid candidate pool。
- Hierarchy list-discovery、profile／service eligibility filtering、result validity與
  successful snapshot reconciliation。
- Priority／random establishment及round selection。
- Readiness與completion gates。
- Disabled／omitted child reconciliation、DELETE intent及late-result fencing state。
- Known FedProx／sample-weighted／`reportAfter` execution mapping。
- Existing `FLServerEngine` selected-set dispatch、wait與aggregation support。
- Realized topology snapshot producer。
- Legacy hierarchy assignment 單次 HTTP 取得、同一份 bytes 的 typed validation、
  正確 plan ownership 與真實 transport-count regression。
- Slice 2所屬`POL-*`、`PATCH-*`、`SCOPE-01`與`NOT-08` deterministic tests。

### 3.2 不納入

- Root產生第一棵protocol tree或hierarchy-wide UUID `mlCorreId`。
- Candidate resource在production flow協商feature 3。
- Root／Intermediate實際建立下一段protocol subscription。
- Topology-only Notify的實際callback送出、逐級接收與Root view更新。
- Feature negotiation失敗後的real downstream DELETE flow。
- Retained-result index、lookup與replacement Branch recovery；candidate fields與Slice 1
  wire validation保留，但runtime不納入目前implementation slices。
- Preparation model-free wiring、ADRF global-model distribution與一般 model transport cutover。
- Legacy assignment／preparation-result bundle移除。
- Multi-NWDAF real-process或testbed evidence。

Topology wiring、ADRF flow與legacy migration由Slice 4–5負責；retained-result runtime
暫緩。Slice 2可以產生transport intents與report values，但不能把未送出或只在unit
test內執行的intent描述為protocol side effect。

本 slice 對 model transport 的唯一例外，是修正已確認的 legacy hierarchy
assignment duplicate GET。這不改變 assignment bundle 的 wire contract，也不引入
ADRF 或 protocol-mode preparation；其他 model 與 result transport 保持不變。

---

## 4. 固定 local execution contract

### 4.1 Candidate record與pool ownership

新增一個PyMTLF-owned、in-memory local candidate pool。每個direct-child identity只有一筆
record；同一identity可同時具有兩種provenance，因此provenance使用set而非單一enum：

- `UPSTREAM_ASSIGNED`：目前effective `x-flTopology.children`明確列出；
- `LOCALLY_DISCOVERED`：本node獲得additional-candidate authority後由NRF找到。

Candidate record至少保存：

- canonical `nfInstanceId`；
- provenance set；
- upstream child instruction，若存在；
- resolved Training service target，若已解析；
- effective `enabled`與priority；
- `UNCONFIRMED`／`DEPLOYING`／`ACTIVE`／`FAILED`／`INACTIVE` relationship state；
- status timestamp、known或opaque cause；
- downstream subscription resource location，若已建立；
- 最新child subtree report，若direct child已回報；
- generation／revision token，用於拒絕stale establishment、result或cleanup completion。

Pool另外保存目前effective normalized discovery scope最近一次成功snapshot的`observedAt`、
剩餘`validityPeriod`、計算後的`validUntil`與snapshot revision。Discovery scope至少包含
event、model interoperability、TAIs、所需role與containing NWDAF identity；scope改變時
舊snapshot不得證明新query仍有效。Snapshot也要保存raw matched／returned count所需資訊；
若`numNfInstComplete`表示NRF找到的總數大於實際回傳數量，該response不是完整membership
snapshot，不得以缺席作pruning依據。

Pool由擁有direct-child FL process的local coordinator保存。Go route、NRF與model artifact
都不是candidate pool owner。Slice 2 pool不持久化；backend generation reset後失效，
Slice 4不得從stale report或NRF response自動恢復舊ownership。

Pool的reconcile、status update、round selection、intent completion與snapshot必須由同一
lock保護；外部HTTP或training operation不得持有該lock。每個輸出intent攜帶pool
revision，side effect完成後只有matching revision可以回填，避免PATCH、DELETE或reset
期間的late completion覆蓋新state。

### 4.2 Effective node contract resolution

新增單一resolver將wire node與caller提供的local defaults解析成immutable effective
contract。Resolution順序固定如下：

1. Protocol-defined defaults：`allowAdditionalCandidates=false`、
   `additionalCandidatePriority=0`；這兩項不能由local config擴大授權。
2. 上層在該node明確指定的policy／strategy／`reportAfter`。
3. 只有wire省略且protocol沒有default的項目，才使用local owner傳入的defaults。
4. 解析後重新驗證effective cross-field與role capability；無法形成完整known executor
   時回`RequirementsError`，不得靜默改成legacy value。

Slice 2不新增production config key。Local resolver constructor接受typed defaults；
Slice 4在啟用protocol mode前，必須將實際configuration映射到此boundary。這避免本slice
預先固定migration selector或改動Leaf／Branch部署設定，同時讓所有省略欄位都有
獨立於request的authoritative source。

`policy` effective value必須包含：

- `selectionMethod`；
- `minAvailableNodes`；
- `fractionTrain`；
- `minTrainNodes`；
- `acceptFailures`；
- `minCompletionRate`，僅在`acceptFailures=true`時影響decision。

`strategy` known executor只接受`method=fedProx`、
`aggregation=sampleWeighted`與typed `proximalMu >= 0`。Wire camelCase values在resolver
boundary轉為現有Python execution arguments，不改寫wire或artifact spelling。

`reportAfter` role mapping：

- local Client work只接受`unit=epoch`，`count`成為既有trainer的epochs；
- local Intermediate work只接受`unit=round`，`count`表示一次upper-tier report前需依序
  完成的lower-tier rounds；
- 上層指定的值不得被local owner覆寫；省略時可由local defaults補足；
- 不符合接收node角色的known unit屬合法shape但local無法履行，回
  `403 ML_MODEL_TRAINING_REQS_NOT_MET`，不是`400`。

Intermediate的`reportAfter(round)`使用目前upper input啟動第一個lower round；後續每個
lower round以前一輪lower aggregate作為新input，依序完成指定count後才向direct parent
回報最後一份aggregate。任一lower round未通過readiness、completion或artifact
validation時，本次upper operation失敗，不用較早的partial aggregate冒充完成結果。

### 4.3 Explicit、delegated與hybrid candidates

Initial reconciliation先載入所有explicit children，包括disabled entries。Explicit
candidate的child instruction、priority、`reportAfter`與subtree永遠優先於同identity
的locally-discovered資訊。

只有effective policy允許additional candidates時才執行list-discovery。Delegated
discovery使用以下authoritative inputs：

- 接收resource的`mLEventSubscs[0].mLEvent`；
- `modelInterInfo`；
- `mlEventFilter.networkArea.tais`中可轉成NRF `trackingAreaList`的範圍；
- direct-child所需FL capability；
- `nnwdaf-mlmodeltraining` service requirement；
- containing NWDAF identity，用來排除self。

第一版不新增generic resource criteria。若`allowAdditionalCandidates=true`但request沒有
可形成bounded candidate domain的tracking areas，也沒有本slice已確認的其他local
scope source，local resolver應以requirements-not-met拒絕此contract，不得只用event、
vendor及capability搜尋整個NRF registry。

Explicit node若帶有`children`或node `policy`，表示該direct child還需作為下一層Server，
resolution要求`FL_SERVER_AND_CLIENT`；沒有downstream orchestration instruction的node
要求`FL_CLIENT`，並允許`FL_SERVER_AND_CLIENT`履行Client角色。Locally discovered
additional candidate第一版只加入direct FL Clients；不替它憑空建立unknown subtree。

List-discovery結果必須：

- 只接受`REGISTERED` profile與registered Training service；
- 重新在PyMTLF驗證event、capability、model interoperability及required TAIs；
- 排除self、explicitly disabled identities與duplicate identities；
- 對同一NF無法唯一選出Training service時不加入pool；
- 依canonical NF identity與service identity排序後才交給pool，避免NRF response order
  影響deterministic state。

Malformed SearchResult或Go／NRF error使本次discovery失敗，但不清空既有pool。NRF找到的
所有profiles不直接出現在report；只有已納入candidate pool、準備嘗試或已嘗試者才回報。

### 4.3.1 Discovery freshness與snapshot reconciliation

Refresh trigger不是「candidate數量不足」，而是local coordinator即將使用
NRF-derived資訊建立新的participant relationship時，該資訊是否仍在有效期內：

```text
need to establish an NRF-derived candidate
  -> matching discovery snapshot still valid?
       yes -> reuse current resolved candidate
       no  -> request discovery through containing Go NWDAF
  -> Go consumer reuses valid cache or queries NRF after cache expiry
  -> successful SearchResult atomically reconciles the local candidate pool
```

Candidate pool不直接連NRF，也不複製Go discovery cache。PyMTLF只保存足以約束local
candidate使用的snapshot freshness；實際是否命中cache或送出NRF request仍由Go consumer
依normalized query與`validityPeriod`決定。從Go internal response收到
`validityPeriod`時，以local receipt time計算`validUntil`；cache hit所帶的剩餘秒數不得被
重設成原始完整TTL。

固定規則如下：

- 無matching successful snapshot、snapshot已過期或discovery scope已改變時，只有
  `LOCALLY_DISCOVERED` provenance的`UNCONFIRMED` candidate不得產生establishment
  intent；pool改為回報需要refresh。是否已有足夠candidate不是freshness判斷條件；
- successful refresh可加入／更新實際回傳的eligible identities；只有完整successful
  snapshot才是同一scope的authoritative candidate-membership observation。新出現者加入為
  `UNCONFIRMED`；仍存在者更新last-seen與resolved service target，但不任意改寫
  relationship status；
- 只具`LOCALLY_DISCOVERED` provenance、仍為`UNCONFIRMED`且未出現在新snapshot的
  identity，只有在新snapshot完整時才可立即從pool移除；partial result不得根據缺席
  prune；
- `DEPLOYING`或`ACTIVE` relationship不因一次refresh缺席而移除。前者等待in-flight
  establishment完成，後者以實際subscription／communication lifecycle為準；
- `FAILED`保留status／cause供topology snapshot回報；`INACTIVE`至少保留到downstream
  cleanup完成。兩者都不具round eligibility，也不因refresh重新出現就自動恢復；
- 具有`UPSTREAM_ASSIGNED` provenance的identity不因list-discovery缺席而移除。若其
  NRF-derived service target已過期，重新建立relationship前仍需由exact resolver取得
  fresh target；
- malformed response、Go／NRF error、timeout或cancel都不是successful snapshot，不得
  prune現有records，也不得延長舊snapshot的`validUntil`；
- concurrent或late refresh result必須以scope與snapshot revision fencing，不能覆蓋較新
  contract或較新discovery observation。

Slice 2完成snapshot value、freshness gate、reconciliation與deterministic intents；Slice 4
才負責在production coordinator中於需要建立新relationship時呼叫refresh並執行後續HTTP
subscription。Failed／inactive record在report delivery後的最終history pruning也留給
Slice 4，因為Slice 2沒有Notify delivery acknowledgement。

### 4.4 Establishment與relationship lifecycle

Candidate pool本身不發HTTP request，而是產生typed local intents；Slice 4將intent接到
既有FL Server subscription client。Transition固定如下：

```text
UNCONFIRMED
  -> DEPLOYING       establishment intent開始執行
  -> FAILED          resolve／admission／timeout／communication失敗

DEPLOYING
  -> ACTIVE          downstream subscription與preparation均成功
  -> FAILED          subscription拒絕、timeout或communication失敗
  -> INACTIVE        上層停用或authority在執行中被撤回

ACTIVE
  -> INACTIVE        participant退出、上層停用或additional authority撤回
```

Legacy `BranchPreDispatchError` 的「單一failure導致未嘗試siblings也報
`NOT_AVAILABLE_ML_TRAIN`」不得成為protocol-mode transition。只有實際開始該
candidate intent後才可進入`DEPLOYING`；尚未嘗試者保持`UNCONFIRMED`，而
discovery、subscription或timeout失敗只更新對應identity。

每次status或cause真正改變才更新UTC timestamp；重複相同outcome保持idempotent。Known
failure映射使用既有candidate causes；無法安全分類使用`OTHER`。Unknown descendant
status／cause只保存於nested report，不改變direct relationship的known lifecycle。

Candidate establishment順序：

- `selectionMethod=priority`時，較高priority先嘗試；同順位使用可注入的random source
  打散；
- `selectionMethod=random`時，從eligible `UNCONFIRMED` pool均勻選取；
- `enabled=false`永遠不進入eligible set；
- 達到`minAvailableNodes`後可先產生ready snapshot，但不強制停止確認其餘候選；
- concurrent intent必須受local capacity限制，capacity值由caller提供，不新增wire欄位。

### 4.5 Readiness與per-round selection

只有已完成subscription／preparation的`ACTIVE` direct children計入`activeCount`。
New round readiness為：

```text
activeCount >= minAvailableNodes
and
activeCount >= minTrainNodes
```

不滿足時不得dispatch新round；coordinator可繼續establishment。滿足時selected count為：

```text
min(activeCount, max(floor(activeCount * fractionTrain), minTrainNodes))
```

`priority` round selection取最高順位，boundary tie以注入random source決定；`random`
selection從active pool均勻抽取。每輪selected identity set在dispatch前凍結：中途新增的
candidate不加入本輪，中途離開的selected participant算本輪failure，不以新candidate
偷換本輪identity。`FLServerEngine`在callback consumption時必須先驗證participant屬於
目前frozen set；非selected或舊round callback不得修改本輪terminal count。

### 4.6 Completion gate與既有aggregation

本slice將selected-set概念接入現有`FLServerEngine.execute_hierarchy_round()`與
`_aggregate_round()`，但保留legacy caller的all-participants預設。Protocol caller需
明確傳入該輪selected identities及effective completion policy。

一輪仍等到所有selected participants進入terminal outcome，或existing round deadline
將未完成者標成timeout後，才評估completion gate：

- `acceptFailures=false`：所有selected participants都必須有合法success result；
- `acceptFailures=true`：`successful / selected >= minCompletionRate`才可聚合；
- failed、terminated、timeout、missing／invalid result都不進aggregation input；
- gate不通過時不得呼叫`FederatedTrainer.aggregate()`；
- gate通過時只把successful selected results交給現有sample-weighted aggregation；
- late或非selected result不得進入該輪aggregate。

現有artifact identity、scope digest、input weights digest、sample count與model contract
validation全部保留。Policy只決定哪些已驗證結果可交給aggregator，不放寬artifact
validation，也不改變weighted aggregation公式。

### 4.7 PATCH、disabled child與cleanup intent

Candidate pool只接收Slice 1已計算完成的old／new effective topology；它不重新實作JSON
Merge Patch。

Reconciliation規則：

- 新`children`完整重建upstream-assigned provenance；locally-discovered provenance與
  local observation保留；
- 同identity兩種provenance重疊時，new explicit instruction立即成為authority；
- omitted identity失去upstream provenance及先前explicit prohibition；只有
  `allowAdditionalCandidates=true`且仍符合discovery條件時才可作為local candidate；
- explicit `enabled=false`建立prohibition，即使NRF再回傳也不得重新加入eligible set；
- `priority: 0`只表示最低順位，不觸發cleanup；
- explicit重新設為enabled會解除該identity prohibition並回到可resolve／establish狀態。

對`DEPLOYING`或`ACTIVE` candidate停用時，pool立即從eligible／selected-future set排除，
產生帶resource location與revision的DELETE intent並將relationship設為`INACTIVE`。Slice 4
執行DELETE後回填：`204`／`404`使用`REMOVED_BY_POLICY`；timeout或communication error
分別保存`RESPONSE_TIMEOUT`／`COMMUNICATION_FAILURE`並保留cleanup retry intent。任何
舊revision的late result皆不得使candidate回到`ACTIVE`或進入aggregation。

若policy PATCH將`allowAdditionalCandidates`由true改成false，所有只有
locally-discovered provenance的active relationships同樣立即失去authority、產生DELETE
intent並成為`INACTIVE`；pool record可保留供report與cleanup，但不再eligible。

### 4.8 Realized topology snapshot

Snapshot builder產生一個`FlTopologyReport` domain value：

- wrapper `nfInstanceId`來自containing NWDAF context，不從instruction自我推論；
- wrapper可帶本node實際採用的effective policy／strategy／`reportAfter`；
- direct children包含upstream明確提供及已納入local pool的additional candidates；
- 每個child保存parent-owned status、原始status timestamp與cause；
- 已收到的child subtree report可巢狀附在該relationship下，descendant timestamp不改寫；
- output依canonical identity排序，使同一state產生stable representation；
- 未知descendant status／cause原樣保存，但未知status不計入activeCount。

Snapshot不帶descendant `roundInd`，也不負責發Notify。Slice 4才會把snapshot放入
`x-flTopologyReport`並交給既有callback lifecycle。

### 4.9 Legacy hierarchy assignment ingress cleanup

Hierarchy assignment preparation 改為一次取得、一次解析，再根據解析結果完成
typed admission：

```text
one HTTP GET
  -> unique staging artifact + response integrity evidence
  -> archive parse / role detection
  -> hierarchy URL, header, archive digest and identity validation
  -> move the same artifact into the plan-owned workspace
```

固定規則：

- 不得為了從 generic artifact 切換到 typed hierarchy artifact 而對同一 URL
  發出第二次 GET；
- hierarchy artifact 仍必須驗證 allowed origin、digest-bearing URL、單一
  `X-Artifact-SHA256`、response body digest、archive contract、artifact role、
  intended recipient，並完成publisher／`planId` schema validation；當caller具有獨立
  expected publisher或plan identity時仍必須比對。單次取得不得降低現有
  strict checks，也不得把artifact自帶的值當成獨立expected value；
- 當 plan identity 驗證完成後，只將最終採用的 artifact 目錄登記給該
  plan；不再 unlink 第一份檔案後對舊 metadata 呼叫
  `claim_artifact()`；
- validation 失敗、resource revision 過期或 plan bind 失敗時，單一 staging／
  adopted artifact 仍依既有 cleanup owner 回收；
- Flat preparation 與 non-hierarchy artifact 的現有行為保持不變。

這項修正先作為 Slice 2 的 baseline remediation，因為本 slice 會繼續延伸
`FLClientEngine`、而 protocol path 成立前仍需保留 legacy HFL regression。它不改變
Slice 4 將 preparation 改為 model-free 的方向。

---

## 5. Baseline stages處置

| Baseline stage | 處置 | Slice 2處理 |
| --- | --- | --- |
| Trigger／initial tree production | 核准延後 | Root protocol producer留給Slice 4 |
| Candidate typed parse與resource storage | 沿用且不改變語意 | 重用Slice 1 typed models與persistent representation |
| Legacy assignment ingress | 修正 | 同一logical assignment只取得一次；在同一份bytes上完成typed validation與plan adoption |
| Node contract resolution | 調整 | 加入effective policy／strategy／`reportAfter`與local-default boundary |
| Explicit identity discovery | 沿用並抽取共用filter | 保留exact resolver semantics |
| Delegated discovery | 調整 | 新增省略target ID的list-discovery，保存`validityPeriod`／`validUntil`並reconcile successful snapshot；不改Go／NRF schema |
| Candidate establishment | 調整但transport延後 | 建立pool、ordering、state與intents；NRF-derived資訊過期時不得建立relationship；HTTP wiring留給Slice 4 |
| Preparation readiness | 調整 | 由complete-required改為`minAvailableNodes`＋`minTrainNodes` gate |
| Round participant selection | 調整 | 由all participants改為凍結selected set；legacy預設仍為all |
| Leaf local training | 沿用executor並增加binding | `reportAfter(epoch)`與`proximalMu`轉為既有trainer arguments |
| Intermediate local rounds | 調整local scheduler contract | `reportAfter(round)`決定upper report前lower-round次數；wire loop留給Slice 4 |
| Result validation | 沿用且不改變語意 | 所有artifact／round／scope／digest validation保留 |
| Completion／aggregation | 調整 | selected terminal outcomes通過policy後才呼叫既有aggregator |
| Model/result publication | 沿用且不改變語意 | 不改workspace artifact內容與sample-weighted output |
| Topology update | 調整local state | Reconcile upstream replacement並保留local provenance |
| DELETE／cleanup | 調整但transport延後 | 產生DELETE intent、INACTIVE state與late-result fence；HTTP留給Slice 4 |
| Topology Notify | 調整producer、transport延後 | 產生snapshot；callback wiring留給Slice 4 |
| Failure／timeout | 沿用deadline並增加policy分類 | Missing selected result在deadline後才成failure；discovery失敗不清pool |
| Restart／generation reset | 沿用失效語意 | Candidate pool與in-flight intents隨PyMTLF generation失效，不做恢復 |
| Final validation／cutover | 沿用legacy，protocol核准延後 | Slice 4／5處理protocol integration與migration closure |

---

## 6. 方向性資料流與owner

### 6.1 Contract到local execution

```text
Slice 1 PyMTLF subscription resource
  -> accepted x-flTopology root node
  -> local contract resolver + independent local defaults
  -> effective node contract
  -> candidate pool / policy executor / local-work scheduler
```

Wire instruction的authoritative producer仍是direct parent；persistent copy由
`FLClientResource`擁有；effective local decision與candidate pool則由該node的local
coordinator擁有。Slice 2不把effective values寫回persistent subscription request。

### 6.2 Delegated discovery

```text
accepted Model Training requirements + containing NWDAF identity
  -> PyMTLF HierarchyNodeResolver list query
  -> containing Go /internal/v1/nrf/nf-instances
  -> Go cache hit or NRF SearchResult after expiry
  -> PyMTLF profile/service revalidation + validity capture
  -> successful discovery snapshot
  -> candidate pool freshness gate + atomic reconciliation
```

Training request是event／interoperability／TAI的authoritative source；NRF profile是
identity、status、capability與service endpoint的authoritative source；candidate
provenance與priority由local pool擁有。Go consumer擁有真正的NRF discovery cache；
PyMTLF local coordinator擁有candidate使用時的freshness decision，但不自行判斷是否需要
bypass Go cache。

### 6.3 Round execution

```text
active candidate pool + effective policy
  -> readiness gate
  -> frozen selected identities
  -> existing FLServerEngine participant resources
  -> existing round PATCH / callback wait / artifact validation
  -> completion gate
  -> existing FederatedTrainer.aggregate(successful selected results)
```

除了§4.9的legacy assignment ingress修正外，本slice不改model transport。Round
input與result URL仍由既有execution owner提供；ADRF global-model path是Slice 4工作。

### 6.4 Legacy assignment ingress

```text
Root or Branch assignment URL
  -> receiving PyMTLF FLWorkspace single-fetch ingress
  -> typed hierarchy validation over the fetched bytes
  -> plan-owned adopted artifact
  -> FLClientEngine preparation / Branch coordinator
```

Publisher 仍是 artifact URL 與 content 的 authoritative producer；receiving
`FLWorkspace` 擁有 transport、integrity validation 與 staging；`FLClientEngine` 擁有
resource revision、role/capability check 與 plan bind。任一 layer 都不得為了重新獲得已經
在前一 layer 取得的資訊而再發一次 HTTP GET。

### 6.5 Topology update與report

```text
old effective subtree + new effective subtree
  -> candidate pool reconcile
  -> establishment / DELETE intents + relationship revisions
  -> direct-child outcomes
  -> stable FlTopologyReport snapshot
  -> Slice 4 Notify producer
```

Pool只產生intent與snapshot。實際peer route、HTTP outcome及Notify delivery分別仍由既有
FL Server／Go route／FL Client callback owners負責。

---

## 7. Exact-file implementation plan

### 7.1 `PyMTLF/` production files

| File | 預計變更 |
| --- | --- |
| `src/py_mtlf/core/fl_candidate_orchestration.py`（new） | Candidate record／pool、provenance、relationship transition、effective contract、discovery snapshot freshness／reconciliation、selection／completion gates、PATCH reconciliation、intents與report snapshot |
| `src/py_mtlf/core/fl_hierarchy_discovery.py` | 抽取共用profile／service eligibility，保留exact `resolve()`語意並讓exact／list結果帶有效期資訊；list snapshot另保存判斷membership completeness所需的raw count |
| `src/py_mtlf/core/fl_workspace.py` | 拆分單次transport與typed hierarchy admission，保留strict integrity／identity checks並正確adopt同一artifact |
| `src/py_mtlf/core/fl_server.py` | Hierarchy round接受frozen selected set與completion policy；wait／aggregate只使用selected participants，legacy default維持all／complete-required |
| `src/py_mtlf/core/fl_client.py` | 修正hierarchy preparation單次取得／adoption；增加resolved local-work argument boundary，使protocol `reportAfter(epoch)`與FedProx strategy使用既有round worker／trainer；feature-disabled resource gate不變 |
| `src/py_mtlf/core/fl_branch.py` | 增加resolved Intermediate local-work argument boundary，以既有lower-round server依序執行`reportAfter(round)`次數；legacy預設維持一次 |
| `src/py_mtlf/core/federated_trainer.py` | 對齊candidate schema，允許finite `proximal_mu >= 0`；aggregation implementation不變 |

`fl_candidate_orchestration.py`是新的Python internal module，不是新NF、HTTP API或wire
schema。它具有獨立穩定責任：保存direct-child local orchestration state並作純
deterministic policy decision。Transport、training與artifact lifecycle仍留在既有owner，
避免把state塞進wire model或再次複製`FLServerEngine`。

預期不修改`wire/ml_model_training.py`；若Slice 2發現Slice 1 typed contract缺陷，先記錄
為cross-slice finding並確認是否需修正，不能在policy implementation中靜默改schema。

### 7.2 `PyMTLF/` test files

| File | 預期證據 |
| --- | --- |
| `tests/test_fl_candidate_orchestration.py`（new） | Provenance、effective defaults、hybrid pool、discovery expiry gate、successful snapshot pruning、ordering、readiness、selection、completion、PATCH、disabled cleanup、status snapshot與unknown report preservation |
| `tests/test_fl_hierarchy_discovery.py` | Exact resolver regression與validity、list query shape、`validityPeriod`／completeness propagation、TAI／capability／service filtering、self／duplicate／ambiguous排除與failure preservation |
| `tests/test_fl_hierarchy_artifacts.py` | Typed hierarchy validation／adoption／cleanup regression；同一assignment的真實HTTP GET count |
| `tests/test_fl_server.py` | Frozen selected-set dispatch、all-terminal wait、partial-success aggregation、gate rejection不呼叫aggregator、late／nonselected result排除及legacy all regression |
| `tests/test_fl_client.py` | Branch／Leaf assignment透過真實workspace／MockTransport各只發一次GET；candidate local-work arguments真正進入既有trainer；上層指定值、local fallback、role mismatch與legacy artifact behavior |
| `tests/test_fl_branch.py` | `reportAfter(round)`依序執行指定lower rounds、前輪aggregate成為後輪input、失敗不提前回報及legacy count 1 regression |
| `tests/test_federated_trainer.py` | Real small-model FedProx `mu=0`／positive behavior與sample-weighted aggregation regression |
| `tests/test_local_trainer.py` | 既有local trainer接受FedProx `mu=0`的boundary regression |

若現有repository沒有`tests/test_federated_trainer.py`，建立該檔案；不要為了測試而從
另一個large integration file間接呼叫private implementation。

### 7.3 `NWDAF/` read-only dependency checks

本slice不修改Go files。Implementation verification需重跑既有NRF query parsing、MTLF
internal route及consumer encoding tests，以證明PyMTLF新增list query仍在現有
standard-shaped boundary內。若必須修改Go query contract，Slice 2停止並重新提出
affected-repository plan，不把`NWDAF/`變更混入原核准範圍。

### 7.4 `nwdaf-docs/`

Implementation期間只更新本plan status、同一phase review ledger及必要evidence links。
不修改design語意。若production evidence推翻本plan的criteria source、owner或lifecycle，
先回到user decision gate。

---

## 8. 實作順序

### 8.1 Characterization與domain primitives

1. 先以真實 `FLWorkspace` 與 `httpx.MockTransport` 建立Branch／Leaf assignment
   transport-count characterization，證明現行路徑的兩次GET。
2. 將assignment ingress改為單次取得／同一artifact typed admission，移除stale
   unlink／claim路徑，並重跑strict validation、cleanup與Flat regression。
3. 為現有all-participants preparation／round／aggregation建立focused regression。
4. 建立effective contract與candidate pool failing tests。
5. 實作provenance、state transition、revision fence與stable snapshot。
6. 完成policy arithmetic及seeded selection table tests。

### 8.2 Delegated discovery

1. 為省略`target-nf-instance-id`的query建立MockTransport boundary test。
2. 從standard training fields抽取event、interoperability與TAIs。
3. 重用exact resolver的profile／service parsing，新增list filtering，並保留
   `validityPeriod`、receipt time與normalized discovery scope。
4. 以fake clock證明未過期snapshot可重用，過期或scope改變時local-only
   `UNCONFIRMED` candidate不能產生establishment intent並要求refresh。
5. 對successful refresh實作atomic snapshot reconciliation，涵蓋新增、仍存在、完整結果
   中local-only `UNCONFIRMED`缺席、partial result不prune，以及
   `DEPLOYING`／`ACTIVE`／`FAILED`／`INACTIVE`保留。
6. 驗證malformed／error response不清除既有pool，也不延長舊snapshot validity。

### 8.3 Existing execution integration

1. 讓`FLServerEngine`接受並凍結selected identities，legacy caller預設仍為全部。
2. Wait與deadline只對selected set收斂，但所有selected都要terminal後才evaluate。
3. Completion gate通過後，只將successful selected results交給既有aggregator。
4. 在`FLClientEngine`將resolved FedProx／epochs接到既有round worker；在
   `FLBranchPreparationCoordinator`將resolved lower-round count接到既有
   `FLServerEngine`，不改model/result artifact contract。
5. 驗證`proximalMu=0`與schema一致，並保留positive FedProx regression。

### 8.4 PATCH、cleanup intent與report

1. 使用old／new effective subtree測試upstream replacement。
2. 實作disabled／authority-revoked transition、DELETE intent與stale revision fence。
3. 產生包含resolved values及nested child report的stable snapshot。
4. 驗證unknown descendant values只保存，不影響readiness或known action。

### 8.5 Final verification與review

1. 執行PyMTLF focused及full gates。
2. 執行unchanged Go NRF boundary checks。
3. 對照Slice 2 conformance table逐項建立production-path→test evidence map。
4. 依Final Completion Re-read Gate重新讀development policy與本plan。
5. 保持working tree unstaged／uncommitted，交由user review。

---

## 9. Slice 2 conformance責任

| Cases | Slice 2 direct evidence | 明確延後 |
| --- | --- | --- |
| `POL-01`, `POL-02` | Active count readiness table；不達門檻無round dispatch | Real subscription establishment由Slice 4 |
| `POL-03` | Formula、priority／random selection與frozen set tests | 跨NWDAF round wiring由Slice 4 |
| `POL-04`, `POL-05` | All-terminal outcome、failure policy、rate boundary與no-aggregate tests | Peer callback E2E由Slice 4 |
| `PATCH-01` | Upstream replacement保留local provenance | Go effective PATCH已由Slice 1；real resource wiring由Slice 4 |
| `PATCH-02` | Disabled active child立即排除、INACTIVE及DELETE intent | HTTP DELETE outcome由Slice 4 |
| `PATCH-03` | Omission、prohibition解除與authorized rediscovery state tests | Cross-edge PATCH scenario由Slice 4 |
| `SCOPE-01` | Node-local policy／strategy／`reportAfter` resolver與role tests | Recursive message forwarding由Slice 4 |
| `NOT-08` procedure portion | Unknown descendant status／cause保存、轉送snapshot且不計ACTIVE | Real Notify relay由Slice 4 |
| `TOP-09` executor closure | Known forward-compatible fields有executor；未知selection／aggregation／unit拒絕且無fallback | Wire decode prerequisite已由Slice 1 |
| Slice 2 discovery freshness | `validityPeriod` capture、expiry／scope gate、successful snapshot reconciliation與failure preservation tests | Production refresh invocation由Slice 4 coordinator完成 |

Slice 2不得將只有intent或snapshot test的case標為完整protocol conformance。Review ledger
需清楚標示local procedure evidence與Slice 4 E2E evidence的界線。

---

## 10. Test design與有效性要求

### 10.1 Candidate／policy tests

- 使用真實`FlTopologyNode`／`FlPolicy`／`FlStrategy` instances，不用自製dict繞過
  Slice 1 contract。
- 對formula與state machine使用table-driven cases，涵蓋boundary等號、fraction floor、
  active cap、zero success與all success。
- Random behavior注入seeded RNG；斷言identity set與可重現性，不mock被測selection
  function的回傳值。
- Fake clock只替換時間來源，用來斷言timestamp及idempotent transition；不mock
  candidate state transition。
- 同一fake clock也用來跨越`validUntil`；直接證明expired local-only candidate不能產生
  establishment intent，而不是mock freshness decision。
- DELETE transport以recording fake作外部boundary；pool必須真實產生intent、revision與
  INACTIVE state。

### 10.2 Assignment ingress tests

- Branch 與 Leaf preparation 均以真實 `FLWorkspace` 及 `httpx.MockTransport`
  執行，對 assignment URL 的 request counter 必須精確為一；不得分別 mock
  `download()` 與 `download_assignment()` 後以 helper call count 代替 transport
  evidence。
- 保留missing／duplicate digest header、URL/body digest mismatch、wrong role、recipient、
  caller具有獨立expected value時的publisher／plan identity、size limit、stale
  revision與cleanup cases。
- 驗證最終resource持有的 `ArtifactMetadata.path` 實際存在於正確plan-owned
  directory，且cleanup不依賴已刪除的generic path。
- Flat preparation 仍只取得一次並維持原有validation／dataset behavior。

### 10.3 Discovery tests

- 只在Go internal HTTP boundary使用`httpx.MockTransport`；profile filtering、dedupe與
  service selection執行真實production code。
- 同一test斷言request未帶`target-nf-instance-id`，並帶正確event、TAIs、capability與
  interoperability。
- 解析真實SearchResult shape並斷言`validityPeriod`、receipt time、`validUntil`與
  normalized scope都進入production snapshot；不得只在test自行補TTL。
- 覆蓋self、explicit duplicate、disabled identity、suspended profile、capability
  mismatch、ambiguous service、malformed SearchResult與503。
- Successful refresh table需覆蓋candidate增加、完整結果減少與partial result不prune；
  error refresh需證明pool和舊`validUntil`均未變。

### 10.4 Execution tests

- `FLServerEngine` tests使用真實participant state、callback collection與workspace
  artifacts；只以fake HTTP peer隔離跨process transport。
- Gate rejection test以spy確認existing aggregator完全未被呼叫，但partial-success
  acceptance test必須使用真實small-model aggregation並驗證sample-weighted結果。
- 非selected與late callback需通過現有correlation path進入，再證明它不污染frozen
  round；不能直接從test刪除該result假裝被忽略。
- Trainer tests使用最小真實tensor／dataset完成至少一個optimization step，不能mock
  loss或optimizer後只檢查參數傳遞。
- Legacy hierarchy focused tests保持原有bundle execution，證明optional selected-policy
  arguments未改變all／complete-required baseline。

---

## 11. Verification commands

Production implementation完成後至少執行：

### 11.1 `PyMTLF/`

Focused：

```bash
uv run pytest -q tests/test_fl_candidate_orchestration.py
uv run pytest -q tests/test_fl_hierarchy_discovery.py
uv run pytest -q tests/test_fl_hierarchy_artifacts.py
uv run pytest -q tests/test_fl_server.py
uv run pytest -q tests/test_federated_trainer.py
uv run pytest -q tests/test_local_trainer.py
uv run pytest -q tests/test_fl_branch.py tests/test_fl_client.py
uv run ruff check src/py_mtlf/core/fl_candidate_orchestration.py src/py_mtlf/core/fl_hierarchy_discovery.py src/py_mtlf/core/fl_workspace.py src/py_mtlf/core/fl_server.py src/py_mtlf/core/fl_client.py src/py_mtlf/core/fl_branch.py src/py_mtlf/core/federated_trainer.py tests/test_fl_candidate_orchestration.py tests/test_fl_hierarchy_discovery.py tests/test_fl_hierarchy_artifacts.py tests/test_fl_server.py tests/test_fl_client.py tests/test_fl_branch.py tests/test_federated_trainer.py tests/test_local_trainer.py
```

Full gate：

```bash
uv run pytest -q
uv run ruff check .
```

### 11.2 `NWDAF/` unchanged boundary

```bash
go test ./internal/backend ./internal/mtlf ./internal/sbi/consumer -run 'NFDiscovery|Discovery'
```

若最終file list改變，focused commands必須同步更新。所有未執行、環境阻擋或已知flaky
checks需在review handoff逐項列出，不能以full suite總數取代focused evidence。

---

## 12. Repository review與commit boundary

- Production change只在`PyMTLF/`；`NWDAF/`是read-only verified dependency。
- `nwdaf-docs/`另行保存plan status與review evidence，不與production commit混合。
- Implementation完成後先保留unstaged／uncommitted diff，回報affected files、diff
  summary、focused／full verification與remaining gaps。
- User確認review後才提出完整commit proposal；review確認不等於commit或push授權。
- 若實作需要修改`NWDAF/`、candidate schema或production config contract，原commit
  split與slice boundary失效，必須先更新計畫並取得user direction。

預期commit split為：

```text
PyMTLF candidate/policy local execution
  -> user review
  -> PyMTLF commit proposal

nwdaf-docs Slice 2 evidence/status
  -> separate user review
  -> separate documentation commit proposal
```

---

## 13. Review checklist

### 13.1 Ownership與boundary

- [x] Candidate pool只由direct-child local coordinator擁有。
- [x] Discovery仍經containing Go NWDAF，不由PyMTLF直連NRF。
- [x] Request／profile／local defaults各自有獨立authoritative source。
- [x] Pool只產生intents與snapshot，不假裝完成Slice 4 transport。
- [x] Feature 3在本slice後仍未於production flow啟用。
- [x] Go consumer繼續單獨擁有NRF cache；PyMTLF只保存candidate freshness metadata，
  沒有建立第二套cache或直連NRF。

### 13.2 Policy與state

- [x] Protocol defaults與local defaults優先順序固定且可測。
- [x] Explicit與local provenance可同時存在且explicit instruction優先。
- [x] Disabled identity、omitted identity與priority 0語意沒有混用。
- [x] Readiness、selection與completion三個threshold階段分開。
- [x] Selected set每輪凍結；late／nonselected result不進aggregate。
- [x] Unknown report values不被推測為known state或action。
- [x] Candidate、participant與selected participant的語意在state與tests中分開。
- [x] Expired或scope-mismatched discovery snapshot不能授權新的local-only
  establishment intent。
- [x] 只有完整successful refresh才立即prune消失的local-only `UNCONFIRMED` records；
  partial result不prune，其他status依relationship／cleanup／report lifecycle保留。
- [x] Failed refresh不改pool，也不延長舊snapshot validity。

### 13.3 Existing execution reuse

- [x] Branch／Leaf每個logical hierarchy assignment都只發出一次HTTP GET。
- [x] 單次取得仍保留全部strict hierarchy integrity／identity validation。
- [x] Resource只保存實際adopt且可由plan cleanup的artifact，不登記已刪除的
  generic artifact。
- [x] 沒有第二套trainer、aggregator、artifact validator或round workspace。
- [x] FedProx `proximalMu`與candidate schema的`>= 0`一致。
- [x] `sampleWeighted`只映射既有sample-count weighted aggregation。
- [x] Leaf `epoch`與Intermediate `round`的`reportAfter` scope正確。
- [x] Legacy all／complete-required hierarchy regression沒有behavior change。

### 13.4 Verification與scope

- [x] Slice 2所有direct conformance cases都有production path及deterministic test evidence。
- [x] Mock只隔離HTTP、clock或scheduler boundary，沒有mock掉被驗證的policy／aggregation。
- [x] Go NRF boundarycheck通過且沒有Go diff。
- [x] 新增freshness／reconciliation tests後，重新執行full PyMTLF tests與ruff。
- [x] 沒有新增retained-result、ADRF、feature negotiation或E2E scope。

---

## 14. 完成條件

Slice 2只有在下列條件全部成立後才能進入implementation `Ready for User Review`：

1. `PyMTLF/` final diff只涵蓋§7列出的legacy assignment ingress remediation、
   local orchestration與existing-executor changes；
2. §9屬於Slice 2的procedure／state responsibilities都有direct tests；
3. `POL-02`與completion rejection確實證明沒有dispatch或aggregate side effect；
4. Explicit／local provenance、disabled prohibition、PATCH replacement及late-result fence
   都有state transition evidence；
5. Branch／Leaf assignment單次GET、strict validation、artifact ownership／cleanup、existing
   trainer／aggregator與legacy hierarchy regressions通過；
6. §5每個baseline stage disposition與final diff一致；
7. §11 focused、full及unchanged-boundary checks完成，或gap明確分類；
8. Mandatory initial review與所有in-scope remediation完成；
9. 依Final Completion Re-read Gate重新讀取current development policy與本plan，完成
   final conformance map；
10. Intended changes保持unstaged／uncommitted並交由user review。
11. NRF discovery snapshot的`validityPeriod`／`validUntil`、scope、freshness gate與
    successful refresh reconciliation均已實作並具direct tests；過期的NRF-derived
    `UNCONFIRMED` candidate不得用於建立新relationship。

即使以上條件全部成立，Root／Branch message wiring、feature 3 success與real-process
protocol evidence仍未完成；retained-result runtime亦不在目前active scope。不得把
Slice 2描述為hierarchical protocol E2E。

---

## 15. 實作審查檢查點

2026-09-04 user review後新增discovery freshness requirement：先前implementation沒有保存
`SearchResult.validityPeriod`，`add_discovered()`也只有增量加入，尚不能在使用
NRF-derived candidate前判斷snapshot是否過期或依fresh successful snapshot處理消失的
local-only `UNCONFIRMED` records。因此本slice已從`Ready for User Review`退回
`Implementation Adjustment Required`。下列結果保留為調整前baseline evidence，不是目前
final review closure。

先前Slice 2 production implementation保留為 `PyMTLF/` unstaged／uncommitted
working-tree diff。已實作範圍包含：

- direct-child candidate pool、provenance、relationship revision、policy arithmetic、
  PATCH reconciliation、DELETE／establishment intents與stable report snapshot；
- bounded delegated discovery，以及由既有Model Training subscription抽取event、model
  interoperability與TAI requirements；
- selected-set freeze、all-terminal completion gate、partial-success acceptance與既有
  sample-weighted aggregator重用；
- Leaf `reportAfter(epoch)`／FedProx local work與Intermediate
  `reportAfter(round)` lower-round scheduling；
- legacy hierarchy assignment的single-fetch typed admission、artifact adoption與失敗清理。

初始diff review辨識並修正了provenance omission、policy re-enable、DELETE retry、重複
discovery revision、pre-bind artifact cleanup，以及以mock取代關鍵production behavior的
測試缺口。詳細finding與direct evidence記錄於同一phase的
[Protocol Extension Implementation Review Ledger](../Protocol%20Extension%20Implementation%20Review%20Ledger.md)。

調整前驗證結果：focused test groups全數通過；`uv run pytest -q`為679 passed、2 skipped；
`uv run ruff check .`通過；未修改的`NWDAF/` discovery boundary tests通過。

本checkpoint只完成local procedure／state責任。Root／Branch protocol message wiring、
feature 3 production advertisement、ADRF global-model distribution、real peer callback E2E
與retained-result runtime仍明確延後，不列入Slice 2完成範圍。

本次調整已關閉discovery freshness缺口：`HierarchyNodeResolver`現在保留normalized scope、
receipt time、`validityPeriod`、`validUntil`、raw returned count與
`numNfInstComplete`所表示的completeness；candidate pool以scope與獨立refresh revision
fence late result，並只讓目前successful snapshot中實際出現且仍有效的local-only
`UNCONFIRMED` candidate產生establishment intent。完整snapshot可移除缺席的local-only
`UNCONFIRMED` record；partial snapshot不以缺席prune，且`DEPLOYING`、`ACTIVE`、`FAILED`、
`INACTIVE` relationship維持既有lifecycle。

最終驗證結果：Slice 2 focused test set為246 passed、2 skipped；其中candidate
orchestration為34 passed，hierarchy discovery為17 passed。`uv run pytest -q`為688
passed、2 skipped；`uv run ruff check .`通過；未修改的`NWDAF/` discovery boundary tests
通過。現有warnings均為既有Starlette／NumPy dependency deprecation warnings。
