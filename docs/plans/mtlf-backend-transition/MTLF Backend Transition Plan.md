# MTLF Backend Transition Plan

Date: 2026-07-25

Status: Phase 1 through Phase 6 complete; Phase 7 detailed implementation plan ready

Related records:

- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 1 PyMTLF Foundation And Backend Boundary.md`
- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 2 Backend Connectivity And Standard Contract Foundation.md`
- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 3 Analytics Subscription Routing.md`
- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 4 ML Model Monitoring And Accuracy Policy.md`
- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 5 Dataset Selection And Direct Retrieval.md`
- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 6 Local Training And Model Update Reprovision.md`
- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 7 Deployment Integration And Legacy Closure.md`
- `nwdaf-docs/docs/plans/mtlf-backend-transition/code-reviews/Initial Model Provision And Monitoring Review Ledger.md`
- `nwdaf-docs/docs/plans/anlf-backend-transition/AnLF Backend Transition Plan.md`
- `nwdaf-docs/docs/plans/daisy/general_improvement/nwdaf-daisy-improvement-plan.md`

---

## 1. Purpose

這份文件是 MTLF backend transition 的 canonical 主計畫。

transition 的目標不是把現有 Go MTLF package逐檔翻譯成Python，而是重新固定同一個NWDAF內的
Go SBI layer、AnLF backend與MTLF backend責任：

1. Go NWDAF是唯一向NRF註冊及對外呈現的標準NF。
2. Go主要負責標準SBI、OpenAPI wire validation、NRF與其他NF的標準通訊、外部resource routing、
   最小correlation state及規格定義的HTTP response。
3. AnLF backend擁有analytics subscription policy/runtime、資料蒐集需求與SMF選擇、analytics inference、
   notification shaping及收到資料後的本地/ADRF storage policy execution。
4. MTLF backend擁有accuracy/retrain policy、dataset scope/window/completeness、ADRF/Mongo retrieval、local training、
   model identity及model artifact publication。
5. 有標準OpenAPI contract的feature在Go/backend boundary沿用相同method語意、field name、required/optional
   semantics、response shape與`ProblemDetails`；不得建立平行business DTO。
6. `NWDAF/`、`PyAnLF/`與`PyMTLF/`最終不保留Daisy-specific runtime、API、config或命名。
7. backend transition只改變責任歸屬時，既有且有意義的business algorithm、state transition、config default
   與edge-case behavior必須等價移植；transition本身不是重新設計演算法的授權。只有文件明列old/new behavior、
   impact與tests，且經使用者明確核准的項目可以作為例外；Phase 4 WAPE-only policy就是此類已核准例外。

Phase依feature組織。同一feature需要Go、PyAnLF與PyMTLF共同調整時，必須在同一phase完成vertical slice；
不得因repository邊界把同一標準procedure拆成互不相容的暫時contract。

---

## 2. Evidence And Contract Policy

### 2.1 Source Order

標準behavior與wire contract依下列順序核對：

1. `NWDAF/`目前實際dependency與generated models
2. `nwdaf-docs/specs/openapi/`中的Release 18 OpenAPI attachment
3. `nwdaf-docs/specs/TS 29.510`、`TS 29.520`、`TS 29.508`、`TS 29.564`、`TS 29.575`
4. `nwdaf-docs/specs/TS 23.288`
5. local free5GC reference implementation

每份detailed plan只要描述標準method、path、request/response、callback或業務status code，就必須在相鄰段落
引用exact OpenAPI file及TS clause。free5GC source只能作為handler/processor/consumer、lifecycle及persistence
結構證據，不能取代3GPP規格。

### 2.2 Core Standard Evidence

| Behavior | Evidence | Consequence |
|---|---|---|
| NRF NF discovery and caching | TS 29.510 clauses 5.3.2.2.1、5.3.2.2.2及6.2.6.2.2；TS 23.501 clause 6.3.1；`TS29510_Nnrf_NFDiscovery.yaml` `/nf-instances` | 使用GET query；成功回200且`SearchResult.validityPeriod` mandatory；相同query在有效期內應重用，Go不得縮成custom endpoint list |
| ADRF discovery profile | `TS29510_Nnrf_NFDiscovery.yaml` `target-nf-type`/`service-names`；`TS29510_Nnrf_NFManagement.yaml` `ServiceName`及`AdrfInfo` | Data Management以`target-nf-type=ADRF`及`service-names=nadrf-datamanagement`發現；Release 18 capability filters存在，但current free5GC compatibility profile第一版不依賴它們 |
| NWDAF event subscription CRUD | TS 29.520 clauses 4.2.2.2、4.2.2.3；`TS29520_Nnwdaf_EventsSubscription.yaml` | create POST/201+Location、update PUT/200或204、delete DELETE/204 |
| SMF event subscription | TS 29.508 clause 4.2.3；`TS29508_Nsmf_EventExposure.yaml` | create POST/201+Location、replace PUT、delete DELETE |
| SMF notification | TS 29.508 clause 4.2.2.2；same OpenAPI callback | SMF POST `NsmfEventExposureNotification`到`notifUri`；成功處理回204 |
| UPF direct notification | TS 29.508 clause 4.2.3.2 Note 2；TS 23.502 clauses 4.15.4.5.1、4.15.4.5.2 steps 3–5；TS 29.564 clauses 5.2.2.2.2、6.1.5.2；`TS29564_Nupf_EventExposure.yaml` | 對明確的`UPF_EVENT` subscription，SMF將Nsmf consumer address/correlation傳給UPF，UPF直接通知該consumer；成功回204 |
| ADRF storage | TS 29.575 clause 4.2.2.2；`TS29575_Nadrf_DataManagement.yaml` | POST `NadrfDataStoreRecord`到`/data-store-records`；成功201+Location |
| ADRF retrieval subscription | TS 29.575 clause 4.2.2.6；same OpenAPI | POST `NadrfDataRetrievalSubscription`；成功201+Location |
| ADRF fetch instruction | TS 29.575 clause 4.2.2.8 | callback body可含`fetchInstruct`；consumer成功存下notification後回204 |
| ADRF direct retrieval | TS 29.575 clause 4.2.2.5；same OpenAPI GET operation | `GET /data-store-records`使用`fetch-correlation-ids`；200有資料、204無資料 |
| ML model monitoring | TS 29.520 clause 4.7；`TS29520_Nnwdaf_MLModelMonitor.yaml` | registration/subscription/notification沿用standard resource及`MLModelMonitorNotify`，不建立custom accuracy envelope |
| ML model provision | TS 29.520 clause 4.5；TS 23.288 clauses 6.2A、6.2E.2；`TS29520_Nnwdaf_MLModelProvision.yaml` | AnLF以standard subscription取得model；notification使用`NwdafMLModelProvNotif`，`MLModelAddr.mLModelUrl`可提供model file URL |

### 2.3 Generated Type Gaps

current `github.com/free5gc/openapi` dependency與local Release 18 YAML不完全一致。feature phase開始前必須先
確認generated type是否足夠：

1. type存在時直接使用generated model。
2. type不存在時優先從local YAML做scoped reproducible generation。
3. 若external `$ref`缺失阻擋generation，才建立isolated compatibility wire package並記錄exact schema來源。
4. 不使用`map[string]interface{}`或散落的手寫struct取代標準schema。
5. 不以`testdata/*.json` fixture建立另一套跨語言contract慣例；contract由typed model與request/response tests驗證。

### 2.4 Existing Behavior Preservation

3GPP規格與OpenAPI決定standard procedure及wire behavior；對規格沒有指定的內部business algorithm，現有
production implementation、config default及tests是移植的behavioral source of truth。ownership從Go移到
Python時必須遵守：

1. detailed plan先建立behavior inventory，將每個既有行為分類為`preserve`、`explicitly replace`或
   `remove as obsolete`。沒有列入後兩類的行為一律視為`preserve`。
2. `preserve`不只代表最後結果大致相同，也包括formula、evaluation ordering、state partition key、buffer/window
   semantics、warm-up gate、threshold/default、skip condition、retry/reset/GC及concurrency guard等可觀察語意。
3. Python可以使用idiomatic language structure，但不得因library default、資料型別、rounding、iteration order或
   簡化實作而改變上述語意。
4. 移除舊Go production path前，必須先保留並執行原Go characterization tests，再把相同scenario與edge case
   移植到Python；相同輸入的decision、state transition與side-effect trigger必須等價。
5. 測試不得只證明happy path。既有Go test覆蓋的skip、boundary、multi-scope、in-flight、reset與failure case都
   必須在Python parity suite有對應案例。
6. 若發現既有行為與規格衝突、確認為bug或確實需要改善，必須先記錄old/new behavior、影響與測試，並經使用者
   明確決策；不得把behavior change混入port或cleanup。

這項約束不要求保留已明確supersede的ownership、Daisy integration、custom DTO或Go proxy flow；它保護的是
被保留下來之feature的business semantics。

這項約束也不要求為分配到後續phase的feature建立temporary compatibility bridge。若某個既有Go
side effect或downstream consumer已明確由後續phase取代，前置phase可在cutover後暫時不提供該功能；但
detailed plan必須記錄暫時不可用的範圍、最終owner與完成phase，且最終phase acceptance必須重新驗證完整
end-to-end behavior。當前phase已宣告完成的feature仍須在該phase內保持可用，不能以future work延後其核心
correctness。

accuracy path是本計畫的明確基準案例：

- AnLF accuracy measurement/alignment的historical behavior以`PyAnLF/src/py_anlf/core/accuracy/metrics.py`、
  `monitor.py`及其現有tests為準。
- MTLF degradation/retrain decision的historical evidence來自`NWDAF/internal/mtlf/trigger.go`、
  `state_store.go`、`NWDAF/pkg/factory/config.go`及其tests；Phase 4 detailed plan逐項決定保留或替換範圍。
- 使用者已明確核准Phase 4 accuracy redesign：production只用WAPE與degradation path。保留per-model/per-scope
  isolation、baseline warm-up、healthy reference、fixed WAPE floor、population std/min-std、strict z-score、N-in-M、
  model-level in-flight/reset、generation reset、scope TTL GC及terminal cleanup。
- Primary metric selection、traffic eligibility/input、chronic path、low-traffic path與consecutive-breach fallback明確
  移除。這是已記錄old/new behavior及tests的例外，不代表其他feature可以繞過preservation policy。
- PyAnLF保留target-slot matching、confidence readiness、scope與observation semantics；production measurement改成
  WAPE-only，並採用30秒ground-truth poll、90秒report window與minimum two matched pairs。Approved zero
  denominator與window semantics以Phase 4 plan為準。

---

## 3. Confirmed Architecture Decisions

### 3.1 Logical NF And Naming Boundary

1. PyAnLF與PyMTLF是同一個NWDAF的internal backend，不是獨立標準NF。
2. backend不向NRF註冊，也不以自己的身分advertise 3GPP service。
3. Go擁有唯一NWDAF `nfInstanceId`、NF profile及external SBI identity。
4. Go source、config、logs、tests與route naming只使用`anlfBackend`、`mtlfBackend`或backend-neutral名稱，
   不使用Python implementation name。
5. Python repository/package可以保留`PyAnLF`、`PyMTLF`、`py_anlf`與`py_mtlf`名稱。

### 3.2 Go Responsibility

Go保留：

- external standard path、method、content type及status response
- OpenAPI/wire-level parsing與required-field validation
- external Location與最小resource routing/correlation record
- process-local Events Subscription與SMF peer Location routing table，供backend reconnect sync使用
- NRF registration、generic NF discovery consumer及依`validityPeriod`管理的shared `SearchResult` cache
- NRF、SMF、ADRF及external consumer的standard client
- backend availability polling與sync lifecycle
- standard payload routing及必要的identity/URI injection

Go不再擁有：

- analytics supported-event policy
- SMF candidate selection或fan-out policy
- SMF configured endpoint、collection profile sharing或reference counting
- analytics scheduling/inference/output shaping
- raw notification preprocessing或training feature shaping
- accuracy/retrain policy
- dataset scope、window、query或completeness policy
- backend-specific ADRF discovery mode、candidate selection及selected endpoint
- training job、model generation或artifact production

### 3.3 Standard-shaped Private Boundary

backend path是private deployment path，但只要對應標準operation：

1. request與response使用相同OpenAPI JSON shape。
2. create維持POST語意，不為restart replay刻意改成private PUT upsert。
3. update維持PUT或該標準operation指定的方法；delete維持DELETE。
4. standard `ProblemDetails`可直接跨boundary傳遞，Go只移除不應外露的private detail。
5. private metadata不得加入standard body；真正必要的routing information優先使用path、query或小型header。
6. 第一版不建立general RPC framework、distributed transaction、custom idempotency system或business wrapper。

允許的非標準contract限於：

- `GET /health/live`
- `GET /health/ready`
- readiness後的backend sync/snapshot
- 無法由standard identifier表達的最小routing correlation

### 3.4 Plain HTTP Scope

目前所有新路徑先使用普通HTTP：

- Go ↔ AnLF backend
- Go ↔ MTLF backend
- SMF/UPF → AnLF backend callback
- MTLF backend → ADRF direct retrieval

TLS、mTLS、OAuth token delegation、certificate/key distribution及Python backend security integration只記為未來
可能補強事項，不列入目前phase、acceptance criteria或blocker。backend仍不註冊NRF。

---

## 4. Target Architecture

### 4.1 Control And Data Paths

```text
External consumer
      |
      | Nnwdaf standard SBI
      v
NWDAF Go ----------------------------------------------+
  - NF identity / NRF registration                     |
  - OpenAPI validation                                 |
  - standard NF consumers / shared NRF discovery cache |
  - routing and minimal snapshot                       |
      |                                                |
      +--> AnLF backend: analytics policy/runtime      |
      `--> MTLF backend: accuracy/training/model       |
                                                       |
PyAnLF -- generic discovery request --> Go --> NRF
PyMTLF -- generic discovery request --> Go --> NRF
PyAnLF -- standard-shaped request --> Go --> SMF/ADRF
   ^                                                   |
   |                                                   |
   +--------------- SMF/UPF notification -------------+

PyMTLF -- standard-shaped retrieval request --> Go --> ADRF subscription
PyMTLF <------------ FetchInstruction ---------- Go <-- ADRF callback
PyMTLF ---------------- direct GET -------------------> ADRF
PyMTLF ---------------- direct read ------------------> MongoDB
```

Python processes不成為external NF，但可以是同一NWDAF內承接資料的endpoint或發出direct data fetch的component。
標準control operation仍由Go執行；大量資料不經Go多一次轉送。

### 4.2 Operation Routing Matrix

| Operation | Decision owner | Standard communication owner | Data endpoint |
|---|---|---|---|
| `Nnwdaf_EventsSubscription` | AnLF backend | Go | Go external SBI |
| Generic NRF discovery and valid-result reuse | requesting backend supplies criteria；Go owns canonical cache | Go calls NRF | Go returns complete standard `SearchResult` |
| NRF SMF discovery criteria | AnLF backend | Go calls NRF | Go returns standard `SearchResult` |
| SMF candidate filtering/fan-out | AnLF backend | Go subscribes selected SMFs | n/a |
| Group ID membership/expansion | AnLF backend config/runtime | n/a | PyAnLF preserves original group provenance |
| SMF event subscription and UPF callback setup | AnLF backend prepares request | Go calls SMF；SMF依procedure設定UPF | SMF或UPF notification直接到AnLF backend |
| SMF event subscription cleanup | AnLF backend維護refcount並決定last-reference delete | Go依AnLF backend指定的target與resource呼叫SMF DELETE | PyAnLF擁有完整peer identity；Go保留target-aware mirror直到cleanup完成 |
| Training datasource state | AnLF backend owns ADRF-first/Mongo fallback state | Go relays `trainingDataSource` through sync | MTLF backend follows current source |
| ADRF discovery and endpoint selection | AnLF and MTLF backends independently choose `configured` or `nrf` | Go calls NRF only for `nrf` mode | each backend stores its selected apiRoot |
| Local Mongo storage | AnLF backend when ADRF unavailable | n/a | AnLF backend stores ADRF-aligned `dataSub + dataNotif` records |
| ADRF storage | AnLF backend selects target and prepares record | Go calls selected ADRF | Go receives ADRF response |
| Initial ML model demand/subscription | AnLF backend | Go routes standard Model Provision operation | MTLF backend owns resource/seed catalog |
| `Nnwdaf_MLModelMonitor` registration/accuracy policy | MTLF backend | Go routes registration/notification | MTLF backend owns registration/policy state |
| `Nnwdaf_MLModelMonitor` subscription/measurement | AnLF backend | Go routes subscription/callback | AnLF backend owns resource/measurement state |
| ADRF retrieval subscribe/unsubscribe | MTLF backend selects target and prepares request | Go calls selected ADRF | ADRF callback first reaches Go |
| ADRF data fetch | MTLF backend | MTLF backend direct HTTP | MTLF backend |
| Mongo training query | MTLF backend supplies the same `smfDataSub + timePeriod` semantics as ADRF | n/a | MTLF backend read-only；adapter uses SUPI/time |
| local training/artifact | MTLF backend | n/a | MTLF backend |
| Model artifact publication | MTLF backend | Go routes standard metadata/notification | AnLF backend downloads directly from MTLF-owned URL |

---

## 5. Analytics And Data Collection Flow

### 5.1 Events Subscription

```text
NF consumer -> Go POST NnwdafEventsSubscription
Go -> validate standard wire contract
Go -> AnLF backend POST standard-shaped resource
AnLF backend -> decide accepted events/features and create runtime
Go <- standard representation
Go -> 201 + Location + representation
```

AnLF backend擁有accepted subscription、reporting precedence、runtime、model association及完整standard
notification shaping。Go只保存external resource routing與backend reconnect所需的最小process-local snapshot。

### 5.2 Generic NRF Discovery, Shared Cache And Backend Selection

Go/backend之間保留單一generic discovery route；SMF與ADRF使用同一條private
`Nnrf_NFDiscovery`-shaped boundary：

```text
PyAnLF -> GET /internal/v1/nrf/nf-instances
          target-nf-type=SMF
          requester-nf-type=NWDAF
          service-names=nsmf-event-exposure

PyAnLF/PyMTLF -> GET /internal/v1/nrf/nf-instances
                 target-nf-type=ADRF
                 requester-nf-type=NWDAF
                 service-names=nadrf-datamanagement

Go -> GET {nrfApiRoot}/nnrf-disc/v1/nf-instances with typed standard query
Go <- 200 SearchResult
backend <- same complete standard SearchResult
```

TS 29.510 clause 5.3.2.2.1要求service consumer在送出新查詢前先考慮reuse，且相同input query在
`validityPeriod`未過期時應重用先前結果並避免同時送出多筆相同query。TS 23.501 clause 6.3.1也允許
同一discovery criteria的後續selection使用有效cache。因此Go作為實際呼叫NRF的NWDAF NF service consumer，
建立backend-neutral shared `SearchResult` cache：

1. cache key是Go實際送往NRF之完整typed query的canonical representation；array排序/去重且不受URL parameter
   順序影響。第一版只重用完全相同query，不實作TS 29.510列為operator-policy的related-query superset reuse。
2. `SearchResult.validityPeriod`依TS 29.510 clause 6.2.6.2.2及Release 18 OpenAPI為mandatory，單位為秒，
   並應與HTTP `Cache-Control: max-age`一致。positive value到期後失效；zero不保存；missing、unparseable或
   negative response視為malformed NRF success並映射502。
3. `nfInstances`為空的successful result也依`validityPeriod`cache。backend polling在有效期內取得同一空結果；
   到期後第一個相同request才重新查NRF。
4. Cache hit回傳result copy，將`validityPeriod`改成原absolute expiry的剩餘整秒並設定相同
   `Cache-Control: max-age`；不得把原始full TTL從backend收到時間重新起算而延長NRF結果。
5. 相同key的concurrent misses合併為一次outbound query；cache只在Go memory，Go restart即遺失，不持久化也不
   放進backend sync。
6. Go只cache完整raw standard result，不選擇peer endpoint。PyAnLF/PyMTLF可各自cache自己的selected candidate，
   因此相同ADRF query可共用Go result但仍保持backend config與selection獨立。
7. internal path保留standard query parameter name、encoding、response及error shape；Go固定或驗證
   `requester-nf-type=NWDAF`並注入containing NWDAF `requester-nf-instance-id`。未知或current generated
   contract不支援的query不得盲目forward。
8. 400、403、500及3xx/Location依TS 29.510 clause 5.3.2.2.2保留，不建立SMF/ADRF專用private error。

SMF既有behavior保留：AnLF backend從完整result選擇所有符合`nsmf-event-exposure`的candidate，每個Events
Subscription首次selection後保存自己的candidate set；cache refresh不自動增加或遷移既有SMF resources。
reconciliation只處理已綁定resource，已有usable binding時不重新discovery。AoI filtering仍是future selection
policy。

ADRF discovery第一版使用`target-nf-type=ADRF`及`service-names=nadrf-datamanagement`。Release 18另定義
`ml-model-storage-ind`、`data-storage-ind`與NF profile `adrfInfoList`/`AdrfInfo`，但current free5GC OpenAPI
dependency及NRF尚未完整支援該filter path；本phase依exact service name發現，不依賴non-standard
`customInfo`或capability flag。PyAnLF與PyMTLF各自提供`configured`/`nrf`模式、各自選擇並保存endpoint；
一邊configured、一邊nrf是合法部署。兩者選到不同ADRF造成資料查不到時視為實驗設定錯誤，不建立endpoint
agreement或Go-side selection。

### 5.3 SMF/UPF Subscription And Direct Callback

```text
AnLF backend -> Go: POST standard NsmfEventExposure
Go -> selected SMF: POST /nsmf-event-exposure/v1/subscriptions
Go <- 201 + Location + NsmfEventExposure
AnLF backend <- same standard response

SMF/UPF ---------------- POST standard notification ----------------> AnLF backend
```

AnLF backend在Nsmf subscription的top-level `notifUri`填入自己維護的callback endpoint，並以
top-level `notifId`提供correlation。依TS 29.508 clause 4.2.3.2 Note 2與TS 23.502 clauses
4.15.4.5.1、4.15.4.5.2 steps 3–5，明確的`UPF_EVENT` subscription可由SMF代consumer建立Nupf
subscription，再由UPF直接通知consumer。SMF因此把Nsmf `notifUri/notifId`分別映射成TS 29.564
clauses 5.2.2.2.2、6.1.5.2及`TS29564_Nupf_EventExposure.yaml`定義的Nupf
`eventNotifyUri/notifyCorrelationId`。本流程不依賴Release 19 BERMS的
`bundledEventNotifyUri`。

因此target architecture不再包含：

- Go-owned `/collector/notify`作為SMF/UPF資料必經入口
- collection requirements GET
- observation binding API
- Go將notification轉成custom observation再POST給AnLF backend
- Go-owned analytics data scheduler

AnLF backend收到SMF `NsmfEventExposureNotification`或UPF `NotificationData`後，在schema與
correlation驗證成功且最新notification已進入bounded ingestion buffer後回204。buffer滿載時採
drop-oldest，不因capacity回503；這是明確接受的best-effort新資料優先政策，必須以drop metric與
structured warning暴露資料遺失。格式、correlation或request-size錯誤仍依對應OpenAPI回應。

PyAnLF保存target apiRoot、peer subscription ID、peer Location、correlation及collection profile。後續
PUT/DELETE維持原`Target-Api-Root`；Go只保存同一tuple的process-local routing/reconnect mirror，不建立
額外route UUID，也不假設peer subId跨SMF全域唯一。

### 5.4 Consumer Delete Versus Collection Cleanup

consumer取消Events Subscription與取消底層SMF collection resource是兩個分離的lifecycle：

1. Go先將external DELETE路由到AnLF backend。
2. AnLF backend停runtime、移除consumer subscription、更新自己維護的collection refcount，
   並記錄cleanup intent後回204。
3. Go收到backend 204後移除external route並向consumer回204；不等待所有SMF DELETE。
4. PyAnLF-owned cleanup worker只在最後一個reference消失時，向Go發送標準形狀的
   `DELETE /internal/v1/smf-event-exposure/subscriptions/{subId}` private proxy request。
5. PyAnLF在DELETE提供原`Target-Api-Root`與peer resource identity；Go依target-aware process-local mirror
   呼叫標準DELETE，不重新discovery或selection，也不假設peer subId跨SMF全域唯一。
6. SMF cleanup失敗時保留Location與cleanup intent並bounded-backoff retry；404可由AnLF backend解讀為
   resource已不存在的terminal cleanup result，但Go/backend wire仍保留standard 404。

SMF collection profile sharing、refcount key、stop-before-cleanup及last-reference delete從現有Go實作等價
移植到PyAnLF；現有失敗後只記log且遺失tracking的best-effort cleanup則明確取代為
可retry/reconcile的新行為。

### 5.5 Subscription Admission Versus Collection

TS 29.520 clause 4.2.2.2.2把EventsSubscription create與後續資料蒐集視為不同procedure，並允許partial
acceptance；只有「要求過去statistics但必要資料不存在」明確要求500 `UNAVAILABLE_DATA`。因此target direction：

1. AnLF backend可接受需要未來資料的analytics subscription，不以同步完成SMF subscription及後續
   SMF/UPF delivery setup作為201的必要條件。
2. historical request若已知必要資料不存在，依規格回500及`UNAVAILABLE_DATA`。
3. unsupported event或feature依標準`failEventReports`/error semantics處理。
4. runtime data source暫時失敗時保留subscription並由AnLF backend重試蒐集；若未來要通知consumer
   failure/termination，必須先確認已negotiated feature與標準`failNotifyCode`/`termCause`適用條件，
   不自行發明callback。

---

## 6. Storage And Retrieval

### 6.1 Raw Notification Storage Ownership

AnLF backend直接收到SMF/UPF notification後，依固定ADRF-first政策只寫一個current training datasource：

- ADRF可用：AnLF backend建立標準`NadrfDataStoreRecord`，附上自己選定的`Target-Api-Root`交由Go呼叫ADRF；
  不同時寫MongoDB。
- ADRF不可用且MongoDB可用：AnLF backend直接寫MongoDB並將`trainingDataSource`切成`mongodb`。
- 兩者皆不可用：`trainingDataSource = unavailable`，bounded ingestion/storage buffer仍遵守drop-oldest policy。

PyAnLF是`trainingDataSource` owner，因為它最先知道新資料實際寫入哪個sink。ADRF transport、429或5xx等
availability failure時，同一筆notification嘗試寫入MongoDB；成功後切換source並經Go喚醒PyMTLF sync。
其他semantic 4xx只拒絕該record，不把整個ADRF標為unavailable。第一版不提供跨sink transaction或
durable delivery保證，也不為正常ADRF可用情況支付dual-write成本。

ADRF recovery分兩種：

1. NRF query沒有matching ADRF時，PyAnLF以bounded polling再次呼叫generic discovery route；Go在空結果的
   `validityPeriod`內重用cache，到期後才真正再查NRF。
2. 已發現endpoint但連線失敗時，PyAnLF保留該endpoint並以capped backoff重試同一endpoint，不因每次transport
   failure立即重查NRF；discovery validity到期後可重新discovery以取得被替換/更新的endpoint。

PyAnLF只有在一次真實standard storage request成功後才把`trainingDataSource`切回`adrf`；切換前資料繼續只寫
MongoDB。詳細recovery probe不得依賴自創standard NF health API，可使用有新notification時的bounded real
storage attempt。失敗與切換必須有structured log/metric。

Mongo contract第一版：

1. 保存和ADRF `NadrfDataStoreRecord`一致語意的完整`dataSub + dataNotif`，不轉成training feature record。
2. 額外保存denormalized SUPI、received time、measurement time及source作local query/provenance metadata；
   `source`不是3GPP欄位，只供operational inspection。
3. 以`(supi, measurementTime)`作training query index；collection correlation仍可供callback routing及
   provenance使用，但不是training-data lookup key。
4. MTLF backend對ADRF與Mongo使用相同`smfDataSub + timePeriod` domain input；Mongo adapter第一版和
   workspace ADRF V0一致，從完整subscription取SUPI並依SUPI/time查詢；traffic-training record以standard
   `dataNotif.upfEventNotifs` alternative選取，不使用top-level local `source`作contract。
5. 第一版假設同一實驗期間SMF subscription semantic profile保持一致，不加入full-document exact match、
   canonical hash、schema version或migration framework，並以乾淨collection開始。
6. MTLF backend使用read-only credential直接查詢。
7. Go不寫Mongo、不提供Mongo query API，也不取得dataset bytes。

ADRF storage依TS 29.575 clause 4.2.2.2使用：

```http
POST /nadrf-datamanagement/v1/data-store-records
Content-Type: application/json

NadrfDataStoreRecord
```

成功必須處理201、Location及response representation；error使用該API的`ProblemDetails`。

### 6.2 Backend Sync And Training Datasource

Phase 2的MTLF-only strict storage handshake被本節取代。兩個backend都採：

```text
POLLING -> READY -> SYNCING -> USABLE
```

sync目的不是交換security key，而是讓三個process對目前runtime state有共同認知：

- Go目前可執行的standard peer capabilities
- AnLF backend目前實際寫入的`trainingDataSource`
- active resource/routing snapshot

Go是central sync hub，AnLF與MTLF backend不建立直接sync channel。兩個backend每次process
啟動都產生新的`processInstanceId` UUID並由health回應回報；Go假如在兩次成功probe之間
看到UUID變更，仍必須將該backend視為內存已重置的新process並完整resync。

PyAnLF在sync response回報`trainingDataSource = adrf|mongodb|unavailable`；Go只保存這個current
process-local value並立即喚醒PyMTLF refresh，再於MTLF sync request傳遞。PyMTLF不再做preferred/effective
source negotiation，而是遵循PyAnLF實際storage落點；若自身無法使用指定source，dataset job暫時不可用或失敗，
不得自行切換到PyAnLF沒有在寫的alternate。

ADRF endpoint、Mongo URI、backend discovery mode、NRF `SearchResult`及candidate cache都不是sync資料。
PyAnLF/PyMTLF依各自config解析endpoint；Go shared NRF cache由backend重新呼叫generic route時透明重用。
每個backend只收到自己有production consumer的typed section，不做Python-to-Python direct sync或盲目廣播
相同payload。
initial/reconnect sync在成功前會擋住USABLE；USABLE後的periodic refresh不在每次request期間
反覆將state切成SYNCING，但refresh失敗會轉UNAVAILABLE。Go-owned snapshot或backend回報的typed
observation改變時可立即喚醒受影響backend的refresh；只有語意值真正改變才轉發，避免
AnLF與MTLF更新形成無限循環。

bootstrap規則：

1. PyAnLF先依自己config執行ADRF discovery/configured resolution；ADRF尚不可用時直接使用MongoDB，不進入
   provisional dual-write。
2. Go尚未取得PyAnLF回報前使用`trainingDataSource = unavailable`；取得語意值變化後refresh PyMTLF。
3. backend restart或reconnect後重新sync並由PyAnLF重新回報current source。
4. sync contract保持小型，只傳current source enum，不傳availability matrix或preference。
5. Go的routing/snapshot table只存於memory；Go process停止或重啟時所有訂閱與routing state
   視為遺失，實驗重跑並由consumer重新訂閱，目前不做持久化或雙向authority recovery。

source切換只定義未來資料落點，不保證歷史連續。若ADRF在`T0-T1`不可用、資料存於MongoDB，`T1`恢復後
`trainingDataSource`切回ADRF，PyMTLF向ADRF查詢跨越`T0-T1`的window可能缺少該段資料；本phase接受遺失，
不做Mongo-to-ADRF backfill、cross-source merge、time-range routing或因ADRF回合法empty result而回查Mongo。

### 6.3 ADRF Retrieval

1. MTLF backend決定資料、時間範圍及query時機，並遵循PyAnLF經sync回報的`trainingDataSource`。
2. MTLF backend建立標準`NadrfDataRetrievalSubscription`並交由Go。
3. Go呼叫ADRF POST；成功處理201、Location與standard representation。
4. ADRF向Go-owned callback POST `NadrfDataRetrievalNotification`。
5. Go成功驗證並保存/交付notification後回204，再將完整notification與`fetchInstruct`轉給MTLF backend。
6. Retrieval request設`consTrigNotif = true`，依TS 29.575選擇只接收`fetchInstruct`的consumer-triggered
   profile；其他standard callback alternative仍完整保存，但視為本次profile mismatch。
7. MTLF backend使用自己configured或NRF-discovered的ADRF Data Management apiRoot及`fetchCorrIds`直接執行
   standard RetrievalRequest GET。`fetchUri`完整保留，但依TS 29.575可能是任意值，只有與selected ADRF standard
   endpoint一致時才採用；不誤用TS 29.576的MFAF Fetch POST。
8. ADRF回200時解析`NadrfDataStoreRecord`；204表示沒有matching data。
9. MTLF backend完成或不再需要資料時，要求Go執行standard retrieval unsubscribe。

Go不代理dataset bytes，不做chunking、normalization、backpressure或dataset completion state machine。

---

## 7. Availability And Failure Boundary

1. Go啟動不要求backend process已存在。
2. Go對AnLF與MTLF backend分別polling `/health/ready`。
3. readiness成功後必須sync才usable；兩個backend使用相同lifecycle概念。
4. backend configured但unavailable時，需要該backend的new operation回該standard API允許的
   `503 ProblemDetails`。
5. backend disabled時，不advertise依賴它且已實作的capability。
6. advertisement依configuration與implemented capability，不因瞬時readiness反覆更新NRF profile。
7. live backend transport failure將該backend標記unavailable並立即喚醒polling。
8. MTLF unavailable不應中止AnLF使用既有模型提供analytics；AnLF unavailable也不應阻止獨立的MTLF工作。
9. backend斷線期間直接送往該backend的SMF/UPF notification可能遺失，第一版接受此限制；backend恢復後
   依Go snapshot重建未來的data collection。
10. polling在USABLE後仍持續；process UUID變更、health/probe failure或live transport failure都會導致
    UNAVAILABLE→READY→SYNCING→USABLE完整循環，不只處理startup ordering。

external HTTP status不得依local convenience決定。每個feature detailed plan要先列OpenAPI response matrix，
再定義backend unavailable、peer NF error與domain rejection如何映射。

---

## 8. Accuracy, Training And Model Provision

1. Accuracy policy、threshold、retrain trigger與training job都由MTLF backend擁有。
2. MTLF backend使用簡單local Python trainer；multiple-NWDAF FL不屬於本transition。
3. MTLF backend擁有model identity及artifact publication。
4. model artifact由MTLF-backend-owned HTTP URL提供。
5. Initial Model Provision納入Phase 4：AnLF backend收到analytics subscription後先尋找compatible loaded
   model；沒有時透過Go建立standard Model Provision subscription，MTLF backend提供configured seed model。
6. Version-controlled initial seed bundle及其import/package流程由MTLF backend repository擁有；AnLF backend
   不保存initial model source，只保存下載後的runtime cache。
7. model provision body沿用`NwdafMLModelProvSubsc`、`NwdafMLModelProvNotif`、`MLEventNotif`與
   `mLFileAddr.mLModelUrl` standard semantics。
8. Go只驗證、mirror並route標準payload；artifact binary由AnLF backend直接向MTLF backend URL下載。
9. AnLF backend下載、檢查package、完整載入並atomically bind後才register monitoring capability。
10. compatible analytics subscriptions共用provision resource與loaded model；必要時以standard PUT更新同一resource
   的active-demand union，不建立第二次POST/download。不同group/filter/target仍建立獨立monitoring scope，任一
   scope可依MTLF policy觸發一次model-level retrain。
11. model package沿用`nwdaf-daisy-improvement-plan.md`及current PyMTLF/PyAnLF已驗證的bundle、immutable
    repository、safe download/cache/load概念，但production命名與依賴
   不保留Daisy。
12. Phase 6只接local training、new artifact publication與updated/re-trained model reprovision，不重建Phase 4
    resource/download path。
13. 不建立custom `ModelReady`、base/target generation CAS、多狀態apply-result或active-generation
   reconciliation API。
14. Future multiple-AnLF以standard `consumerId`、event/filter/target隔離monitor scope；目前不實作AGG、AoI
    routing或FL，但不得以single AnLF/single scope作model policy identity。
15. Accuracy notification以standard `MLModelAccuracyInfo.deviation`承載WAPE ratio；不填percentage
    `mlModelAcc`。Data sufficient的period才提供`deviation`，不足時仍送合法periodic notify但不更新PyMTLF policy。
16. Phase 4 accuracy policy只保留degradation path；Phase 5 dataset retrieval只服務retraining/training，不再補足
    monitoring policy input。

---

## 9. Feature-oriented Migration Phases

### Phase 1: Foundation Baseline

狀態：完成。

- PyMTLF service/package/config/test基礎
- MTLF backend naming與artifact boundary
- Python lint/test baseline
- legacy plan re-evaluation

### Phase 2: Backend Connectivity Foundation

狀態：foundation完成；unified readiness/sync correction已由Phase 3完成；dataset retrieval與final
`trainingDataSource` cutover仍由Phase 5完成。

保留：

- AnLF/MTLF independent polling
- cached availability
- backoff、wake-up與shutdown ownership
- liveness/readiness routes
- disabled/configured/unavailable區分

後續phase責任：

- Phase 3已將MTLF-only handshake改成兩個backend都使用的readiness後sync，並移除Go作為future Mongo
  writer的assumption。
- Phase 5以PyAnLF-owned ADRF-first/Mongo fallback取代`preferredSource` negotiation，並將current
  `trainingDataSource`同步給PyMTLF。

### Phase 3: Analytics Subscription, Collection And Raw Storage

詳細計畫：`Phase 3 Analytics Subscription Routing.md`

狀態：完成並通過module-level lint、test、race與build驗證。Closing implementation commits：

- NWDAF `3ced6d6`
- PyAnLF `88088c4`
- PyMTLF `c0cb13a`

- `Nnwdaf_EventsSubscription` standard-shaped routing到AnLF backend
- private create仍使用POST，update PUT，delete DELETE
- PyAnLF-owned analytics resource/runtime
- standard `Nnrf_NFDiscovery`-shaped Go/backend interaction
- SMF candidate filtering與fan-out policy移到PyAnLF
- 實際`groupId -> SUPI list`設定、group expansion與original group provenance移到PyAnLF；static mapping明確
  標示為TS 23.502 UDM discovery/`Nudm_SDM_Get`完整流程的過渡替代
- SMF mode/configured endpoints、per-subscription candidate binding、collection sharing與refcount移到PyAnLF
- NRF `SearchResult`依`validityPeriod`提供future subscription使用；既有subscription不因後續discovery
  result變動而自動遷移或增加SMF
- Go對PyAnLF以`Target-Api-Root`指定的SMF執行standard subscription，並由該procedure建立SMF/UPF delivery
- PyAnLF保存完整peer identity；Go只保存同一份process-local routing/reconnect mirror，不產生另一個route UUID
- PyAnLF在last reference消失後透過Go執行standard SMF DELETE
- SMF/UPF callback直接到PyAnLF
- UPF raw storage與analytics conversion分離；保留SUPI/DNN/S-NSSAI/address identity，單一malformed
  measurement只降級該欄位
- PyAnLF direct Mongo ADRF-aligned `dataSub + dataNotif` raw write
- PyAnLF standard-shaped ADRF storage request經Go送ADRF
- 現有Mongo/ADRF independent queues及bootstrap dual-write將由Phase 5依single-active
  `trainingDataSource`政策調整；drop-oldest bounded ingestion/analytics/storage buffers與startup validation保留
- unified AnLF/MTLF continuous reconnect/sync foundation與process incarnation UUID

### Phase 4: Initial Model Provision, ML Model Monitoring And Accuracy Policy

詳細計畫：`Phase 4 ML Model Monitoring And Accuracy Policy.md`

狀態：initial provision、standard monitoring、WAPE measurement與degradation-only policy已切換至backend
owner；code-review ledger中的current-slice defects及required三process integration gate已於2026-07-24
關閉。跨repository process harness位於
`nwdaf-resources/tests/mtlf_model_monitor/`，不納入任何單一NF runtime repository。

- standard `Nnwdaf_MLModelProvision` subscription/notification routes for configured seed model
- PyAnLF non-blocking model demand、compatible-model reuse、direct artifact download/load及runtime binding
- PyMTLF-owned provision resource、seed catalog與existing immutable artifact URL
- standard `Nnwdaf_MLModelMonitor` routes after model activation
- same model/different group形成independent monitor scopes；same canonical context使用refcount共用
- AnLF backend保留ground-truth alignment、confidence readiness與scope mechanics，改以report-window內WAPE產生
  standard accuracy measurement；資料不足時省略`deviation`
- MTLF backend擁有monitor registration與accuracy/retrain policy
- 將Go `trigger.go`、`state_store.go`中經確認保留的degradation mechanics移植到PyMTLF；multi-metric、traffic、
  chronic與low-traffic行為按已核准redesign移除
- 先以既有Go tests建立behavior matrix，保留mechanics做parity tests，approved changes做explicit old/new tests
- Go只做standard validation、resource/callback routing與error mapping
- 移除被standard flow取代的custom model provision coordination、custom accuracy envelope及Go accuracy policy

### Phase 5: Dataset Selection And Direct Retrieval

詳細計畫：

- `Phase 5 Dataset Selection And Direct Retrieval.md`

- generic Go NRF discovery proxy與shared validity cache
- PyAnLF/PyMTLF各自的ADRF `configured`/`nrf` config與endpoint selection
- PyAnLF-owned ADRF-first/Mongo fallback及`trainingDataSource` sync
- consume Phase 4 retrain intent中的triggering scope與active scope inventory；任一scope觸發後收集同一model
  所有active scopes的資料，全部納入training dataset候選集合；trigger只作cause metadata
- standard ADRF retrieval subscribe/callback/unsubscribe經Go
- 完整`FetchInstruction`交付MTLF backend
- MTLF direct ADRF fetch
- MTLF以相同`smfDataSub + timePeriod`語意read-only query Mongo；adapter使用SUPI/time與standard
  `dataNotif.upfEventNotifs`，不以correlation ID或top-level local `source`作dataset key
- 模擬實驗中的UE固定視為已同意analytics/model-training data use；不新增consent config或UDM enforcement，
  且不宣稱production compliance
- Dataset READY後保持canonical logical model family的`retrain-in-flight`，直到Phase 6 terminal
  training/model-update outcome
  才release，避免同一model重複retrieval
- 真實free5GC NRF/ADRF/Mongo process-level discovery與retrieval tests
- 不從dataset重建Phase 4 accuracy observation，也不承擔policy input enrichment

### Phase 6: Local Training And Model Update/Reprovision

詳細計畫：

- `Phase 6 Local Training And Model Update Reprovision.md`

狀態：model family／per-artifact identity、local training/reprovision與registration-driven
activation lifecycle已完成本地實作與驗證。

- local trainer與job lifecycle
- 產生new artifact package並publish到既有MTLF-owned URL repository
- logical model family、每代artifact的新`modelUniqueId`與generation/update completion
- snapshot建立時對所有active scopes建立較新80% training與較早20% reference validation；符合資料資格的
  scopes參與單次training/evaluation，永遠記錄per-scope/aggregate WAPE，並由config決定performance
  regression是否阻擋promotion
- triggering scope資料不足時job失敗；其他scope依training/evaluation資料資格降級加入或排除並告警，
  不拖垮整個retraining
- training期間scope新增/刪除只記錄warning，不浪費已完成candidate；base family/version/generation stale仍阻擋
- 透過Phase 4既有standard Model Provision subscription通知updated/re-trained model
- AnLF candidate download/load、failure保留old model與atomic runtime identity/artifact swap
- PyAnLF successful swap後重設相關inference/accuracy windows，接著以新model ID建立Model Monitor
  registration；PyMTLF以新registration/subscription/correlation接受新WAPE，不再用no-deviation
  liveness推測activation
- 移除remaining custom generation/apply assumptions

### Phase 7: Deployment Integration And Legacy Closure

詳細計畫：

- `Phase 7 Deployment Integration And Legacy Closure.md`

狀態：runtime、team component及portable deployment實作已完成，configured-endpoint
`portable application E2E`已通過。SMF/go-upf已建立本地實驗commit並固定於component lock；
NWDAF、PyAnLF、PyMTLF與support-tooling均已按repository及責任scope完成closing commits。

- 在`nwdaf-resources`加入真實team SMF、team go-upf EES replay、ADRF與Mongo組成的portable
  application E2E，驗證Nsmf/Nupf create/delete、UPF direct notification、storage、retrieval
  callback/direct fetch、Mongo fallback與local retraining/model activation
- team SMF加入預設關閉的static session resolution，以config將SUPI解析為UE IP與Nupf apiRoot；
  team go-upf加入不初始化PFCP/gtp5g的standalone dataset replay entrypoint。兩者都維持標準wire API，
  normal production behavior不變
- configured SMF/ADRF endpoint是主要acceptance；NRF作為獨立選配scenario驗證SMF discovery與Go shared
  validity cache。NWDAF加入預設enabled的NRF registration switch，configured scenario顯式關閉；
  current pinned NRF仍無法查詢ADRF時，ADRF繼續使用configured mode
- active PDU session、UE/RAN、PFCP、gtp5g與完整free5GC core的privileged full-core E2E延後為選配
  deployment validation，不阻擋本階段完成，也不要求修改目前主機網路
- portable application E2E前先在team SMF完成Release 18 third-party subscription修正：使用repository-local Nupf wire
  types/client，接受PyAnLF現有Nsmf shape，並將top-level`notifUri/notifId`映射到Nupf
  `eventNotifyUri/notifyCorrelationId`；不依賴額外OpenAPI fork，也不要求PyAnLF加入Release 19 BERMS欄位
- 延續AnLF backend transition的domain package慣例：AnLF與MTLF各自保有薄的Go-side auxiliary HTTP
  edge／processor／client，啟用既有`8090`／`8091`server config；共用能力只位於
  `internal/backend`或`internal/sbi`，不建立neutral mega-gateway；兩個backend的sync分別提供對應
  internal callback base URI
- 依production call graph移除Go legacy MTLF scheduler、dataset/model coordinator、Daisy、fixed ADRF flow、
  obsolete traffic state與custom APIs；同時拆解並移除`internal/anlf/coordinator`，且不為MTLF新增
  coordinator
- 移除fixed `externalMtlf` selection/config與舊runtime wiring，但不把標準ML Model Provision consumer
  誤認為獨立MTLF NF client；保留或重構為未來另一個NWDAF所使用的薄standard consumer，目前不接入
  production selection flow
- 移除PyAnLF未掛載的legacy/Daisy routes與tests，完成三個runtime repositories的naming/config audit
- 建立所有手寫3GPP wire types inventory；對ML Model、ADRF及NWDAF Nsmf `UPF_EVENT`等schema family
  分別做scoped generation feasibility，成功則使用generated types，否則集中到
  `internal/compat/<service>`；清除NWDAF legacy Release 19 BERMS bundling欄位，不同repository各自
  保存自己的compat package
- current free5GC NRF可接受ADRF registration但拒絕ADRF target discovery；本階段portable E2E先使用backend
  configured ADRF endpoint，並保留這項pinned compatibility limitation
- final build/lint/race/process verification與文件closure

---

## 10. Verification Strategy

每個feature phase至少驗證：

1. standard method/path/status matrix
2. Go與Python對相同OpenAPI payload的round-trip
3. unsupported Release 18 field不被silent drop
4. backend unavailable/reconnect/sync
5. standard peer error與`ProblemDetails` mapping
6. process-level callback與direct data path
7. 每個移植feature的`preserve`／`explicitly replace`／`remove as obsolete` behavior inventory
8. old Go characterization tests與new Python parity tests對相同scenario的decision/state-transition equivalence

關鍵integration matrix：

- consumer POST/PUT/DELETE → Go → AnLF backend
- PyAnLF discovery request → Go → NRF `SearchResult` → PyAnLF selection
- identical PyAnLF/PyMTLF ADRF discovery → Go shared valid `SearchResult` → independent backend selection
- PyAnLF standard SMF request → Go → all selected SMFs
- SMF/UPF → PyAnLF direct notification → selected ADRF，或ADRF unavailable時Mongo
- PyAnLF `trainingDataSource` change → Go sync → PyMTLF selected-source retrieval
- first analytics demand → PyAnLF → Go → PyMTLF seed provision → Go callback → PyAnLF direct download/load
- same model + different groups → independent monitor notifications and policy scopes
- ADRF retrieval callback → Go → MTLF → direct ADRF fetch
- Mongo `dataSub + dataNotif` raw record → MTLF以`smfDataSub + timePeriod` direct query
- accuracy notify → retrain → model provision → AnLF download/load
- real ADRF storage → retrieval subscription/callback → PyMTLF direct fetch → local retraining
- configured endpoint → team SMF static resolution → team go-upf standalone replay
  subscription/notification/delete → PyAnLF
- optional real NRF SMF discovery → Go shared cache → 同一portable application data path

Go verification依`NWDAF/Makefile`執行`make test`、`make lint`、`make build`；Python repositories執行各自
formatter/linter/test。需要Mongo、NRF、SMF、UPF或ADRF的check必須分開標示unit、process-level及environment
integration，不得以mock test宣稱完整驗證。Phase 7進一步區分fixture process E2E、使用真實team
SMF/go-upf但以static resolution/dataset replay運作的portable application E2E，以及未來需要
gtp5g/PFCP/PDU session的privileged full-core E2E。portable application可以證明標準Nsmf/Nupf
HTTP/resource lifecycle，但只有full-core可以宣稱active-session resolution與真實user-plane通過。

---

## 11. Explicitly Deferred

- multiple-NWDAF federated learning
- AoI-based SMF candidate filtering
- UDM discovery與`Nudm_SDM_Get` Group ID member retrieval
- durable cross-Go-restart subscription recovery
- message broker或distributed transaction
- Mongo schema version/migration framework
- production TLS、mTLS、OAuth delegation及certificate management
- Python backend獨立NF registration
- artifact signing或untrusted third-party model execution

deferred項目不得被加入目前acceptance criteria，也不得成為目前plain-HTTP implementation的blocker。

---

## 12. Future Decision Gates

以下不是已確認架構的翻案點，而是留到有對應feature consumer時才鎖定：

1. runtime data collection持續失敗時，何時使用已negotiated standard failure notification或termination；
   不支援對應feature時只能維持local retry/observability，不能發明callback。

遇到OpenAPI、TS、current generated type或實作假設衝突時，必須先更新計畫並請求決策，不得無聲改回
Go-owned collection、dataset proxy、strict handshake或custom generation framework。

同樣地，移植過程若要改變既有business algorithm、config default、threshold、state/window semantics或edge-case
處理，屬於新的behavior decision gate；即使修改者認為新作法較簡單或較合理，也不得直接隨port實作。

---

## 13. Superseded Designs

下列舊方向不再是target architecture：

- Go-owned normalized dataset provider/chunk/completion API
- Go-owned SMF/UPF notification ingress及observation forwarding
- collection-requirements GET與observation-binding control plane
- Go-owned raw Mongo writer或Mongo query proxy
- MTLF-only strict storage handshake及no-fallback mode
- configurable preferred/effective datasource negotiation與bootstrap/steady-state dual-write
- Go-owned fixed ADRF endpoint或ADRF discovery/selection
- private PUT create/upsert取代standard POST
- custom callback token/idempotency cache作為第一版必要條件
- custom accuracy report envelope
- Go-owned custom model provision binding/event coordination
- custom `ModelReady`與generation CAS
- Go/PyMTLF active-generation reconciliation

historical detailed plans仍可說明已實作過什麼，但新的implementation與review一律以本文件及最新feature
detailed plan為準。
