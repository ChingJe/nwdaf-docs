# MTLF Backend Transition Plan

Date: 2026-07-22

Status: Phase 1 and Phase 2 foundations implemented; Phase 3 analytics subscription, collection, and raw-storage cutover implemented and verified; Phase 4 detailed planning includes initial model provision plus WAPE-only monitoring, with D1 resolved

Related records:

- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 1 PyMTLF Foundation And Backend Boundary.md`
- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 2 Backend Connectivity And Standard Contract Foundation.md`
- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 3 Analytics Subscription Routing.md`
- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 4 ML Model Monitoring And Accuracy Policy.md`
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
4. MTLF backend擁有accuracy/retrain policy、dataset selection、ADRF/Mongo retrieval、local training、
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
| NRF NF discovery | TS 29.510 clause 5.3.2.2.2；`TS29510_Nnrf_NFDiscovery.yaml` `/nf-instances` | 使用GET query；成功回200 `SearchResult`；不得縮成custom endpoint list |
| NWDAF event subscription CRUD | TS 29.520 clauses 4.2.2.2、4.2.2.3；`TS29520_Nnwdaf_EventsSubscription.yaml` | create POST/201+Location、update PUT/200或204、delete DELETE/204 |
| SMF event subscription | TS 29.508 clause 4.2.3；`TS29508_Nsmf_EventExposure.yaml` | create POST/201+Location、replace PUT、delete DELETE |
| SMF notification | TS 29.508 clause 4.2.2.2；same OpenAPI callback | SMF POST `NsmfEventExposureNotification`到`notifUri`；成功處理回204 |
| UPF direct notification | TS 29.564 clauses 5.2.1.2與6.1.5.2；`TS29564_Nupf_EventExposure.yaml` | 經SMF建立subscription後，UPF可直接通知NF consumer提供的URI；成功回204 |
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
- NRF registration與NF discovery consumer
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
- dataset selection
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
  - standard NF consumers                              |
  - routing and minimal snapshot                       |
      |                                                |
      +--> AnLF backend: analytics policy/runtime      |
      `--> MTLF backend: accuracy/training/model       |
                                                       |
PyAnLF -- standard-shaped request --> Go --> NRF/SMF/ADRF
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
| NRF SMF discovery criteria | AnLF backend | Go calls NRF | Go returns standard `SearchResult` |
| SMF candidate filtering/fan-out | AnLF backend | Go subscribes selected SMFs | n/a |
| Group ID membership/expansion | AnLF backend config/runtime | n/a | PyAnLF preserves original group provenance |
| SMF event subscription and UPF callback setup | AnLF backend prepares request | Go calls SMF；SMF依procedure設定UPF | SMF或UPF notification直接到AnLF backend |
| SMF event subscription cleanup | AnLF backend維護refcount並決定last-reference delete | Go依AnLF backend指定的target與resource呼叫SMF DELETE | PyAnLF擁有完整peer identity；Go保留target-aware mirror直到cleanup完成 |
| Raw Mongo storage | AnLF backend | n/a | AnLF backend writes Mongo |
| ADRF storage | AnLF backend prepares record | Go calls ADRF | Go receives ADRF response |
| Initial ML model demand/subscription | AnLF backend | Go routes standard Model Provision operation | MTLF backend owns resource/seed catalog |
| `Nnwdaf_MLModelMonitor` registration/accuracy policy | MTLF backend | Go routes registration/notification | MTLF backend owns registration/policy state |
| `Nnwdaf_MLModelMonitor` subscription/measurement | AnLF backend | Go routes subscription/callback | AnLF backend owns resource/measurement state |
| ADRF retrieval subscribe/unsubscribe | MTLF backend prepares request | Go calls ADRF | ADRF callback first reaches Go |
| ADRF data fetch | MTLF backend | MTLF backend direct HTTP | MTLF backend |
| Mongo training query | MTLF backend | n/a | MTLF backend read-only |
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

### 5.2 NRF Discovery And SMF Selection

AnLF backend與Go之間的discovery interaction模仿`Nnrf_NFDiscovery`：

```text
PyAnLF -> GET private discovery route
          target-nf-type=SMF
          requester-nf-type=NWDAF
          service-names=nsmf-event-exposure

Go -> GET {nrfApiRoot}/nnrf-disc/v1/nf-instances with standard query
Go <- 200 SearchResult
PyAnLF <- same standard SearchResult
```

規則：

1. internal path可使用private prefix，但method、query parameter name、encoding、response及error shape對齊
   `TS29510_Nnrf_NFDiscovery.yaml`。
2. Go固定或驗證`requester-nf-type=NWDAF`及containing NWDAF identity。
3. Go不得將`SearchResult`預先縮減成`[]string` endpoint。
4. AnLF backend從`nfInstances`/`nfServices`選擇SMF。
5. 第一版沿用現況：選擇全部符合條件且提供`nsmf-event-exposure`的候選SMF。
6. 未來AoI-based filtering屬AnLF selection policy；目前只記錄為後續目標，不實作。
7. `configured`模式的endpoint選擇也由AnLF backend決定，不留在Go processor。
8. `nrf`與`configured`模式都由AnLF backend將已選定的apiRoot放入private
   `Target-Api-Root` header交給Go；Go不再根據自己的SMF config重新選擇target。
9. `SearchResult`依`validityPeriod`cache供新的Events Subscription使用；每個subscription首次selection後
   保存自己的candidate set，後續cache refresh不自動增加或遷移其既有SMF resources。
10. reconciliation只處理該subscription已綁定resources的create、refcount、release、retry與restore；已有
    usable binding時不重新discovery。只有尚未取得candidate或全部initial create失敗時才重新查詢。
11. SMF collection是必要功能，config只選擇`nrf`或`configured`，不提供enable/disable switch。

TS 29.510 clause 5.3.2.2.2要求成功response為200並帶可cache的validity period及匹配NF profiles；
400表示query input錯誤、403表示不允許discover、500表示NRF internal error。Go/backend不得以private
error code取代這些standard semantics。

### 5.3 SMF/UPF Subscription And Direct Callback

```text
AnLF backend -> Go: POST standard NsmfEventExposure
Go -> selected SMF: POST /nsmf-event-exposure/v1/subscriptions
Go <- 201 + Location + NsmfEventExposure
AnLF backend <- same standard response

SMF/UPF ---------------- POST standard notification ----------------> AnLF backend
```

AnLF backend在subscription的`notifUri`或UPF event notification URI填入自己維護的callback endpoint。
TS 29.508 clause 4.2.3.2說明`UPF_EVENT`/`QOS_MON`明確subscription會造成UPF direct notification；
TS 29.564 clause 5.2.1.2也列出經SMF建立subscription後由UPF直接通知NF service consumer的流程。

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

AnLF backend直接收到SMF/UPF notification後，依目前source agreement寫入可用storage：

- MongoDB：AnLF backend直接寫。
- ADRF：AnLF backend建立標準`NadrfDataStoreRecord`，交由Go呼叫ADRF。

callback ingestion後分別進入independent bounded Mongo與ADRF retry queues。兩個worker互不阻塞；
network/timeout、Mongo connection failure及ADRF 5xx使用capped exponential backoff，ADRF明確4xx或local
invalid record記錄後丟棄。每個queue滿載時drop oldest並保留newest，第一版不提供跨sink transaction或
durable delivery保證。

Mongo contract第一版：

1. 保存原始standard notification JSON，不轉成training feature record。
2. 不加入schema version或migration framework。
3. 只增加query/index所需的最小metadata，例如received time、measurement time、source、
   notification/subscription correlation。
4. MTLF backend使用read-only credential直接查詢。
5. Go不寫Mongo、不提供Mongo query API，也不取得dataset bytes。

ADRF storage依TS 29.575 clause 4.2.2.2使用：

```http
POST /nadrf-datamanagement/v1/data-store-records
Content-Type: application/json

NadrfDataStoreRecord
```

成功必須處理201、Location及response representation；error使用該API的`ProblemDetails`。

### 6.2 Backend Sync And Source Agreement

Phase 2的MTLF-only strict storage handshake被本節取代。兩個backend都採：

```text
POLLING -> READY -> SYNCING -> USABLE
```

sync目的不是交換security key，而是讓三個process對目前runtime state有共同認知：

- Go目前可執行的standard peer capabilities
- AnLF backend觀察到的Mongo/ADRF write availability
- MTLF backend可直接使用的Mongo/ADRF read capability
- active resource/routing snapshot
- MTLF backend選擇的source preference/effective source

Go是central sync coordinator，AnLF與MTLF backend不建立直接sync channel。兩個backend每次process
啟動都產生新的`processInstanceId` UUID並由health回應回報；Go假如在兩次成功probe之間
看到UUID變更，仍必須將該backend視為內存已重置的新process並完整resync。

Go可將AnLF backend回報的Mongo availability更新到central snapshot後sync給MTLF backend；
MTLF backend回報的effective source也可經Go更新給AnLF backend。每個backend只收到自己有
production consumer的typed section，不做Python-to-Python direct sync或盲目廣播相同payload。
initial/reconnect sync在成功前會擋住USABLE；USABLE後的periodic refresh不在每次request期間
反覆將state切成SYNCING，但refresh失敗會轉UNAVAILABLE。Go-owned snapshot或backend回報的typed
observation改變時可立即喚醒受影響backend的refresh；只有語意值真正改變才轉發，避免
AnLF與MTLF更新形成無限循環。

bootstrap規則：

1. MTLF尚未選擇時，AnLF backend向所有當下可用storage寫入；兩者可用即dual-write。
2. selection完成後，MTLF backend依config preference與live availability選擇source。
3. preferred source失效時可fallback到另一個可用source；不得沿用舊設計把temporary source loss永久視為
   handshake conflict。
4. backend restart或reconnect後重新sync。
5. sync contract保持小型，exact fields由Phase 3/Phase 5按真正consumer固定，不預先建立通用協議。
6. Go的routing/snapshot table只存於memory；Go process停止或重啟時所有訂閱與routing state
   視為遺失，實驗重跑並由consumer重新訂閱，目前不做持久化或雙向authority recovery。

### 6.3 ADRF Retrieval

1. MTLF backend決定資料、時間範圍、來源及query時機。
2. MTLF backend建立標準`NadrfDataRetrievalSubscription`並交由Go。
3. Go呼叫ADRF POST；成功處理201、Location與standard representation。
4. ADRF向Go-owned callback POST `NadrfDataRetrievalNotification`。
5. Go成功驗證並保存/交付notification後回204，再將完整notification與`fetchInstruct`轉給MTLF backend。
6. MTLF backend使用`fetchUri`及`fetchCorrIds`直接GET ADRF。
7. ADRF回200時解析`NadrfDataStoreRecord`；204表示沒有matching data。
8. MTLF backend完成或不再需要資料時，要求Go執行standard retrieval unsubscribe。

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
6. model provision body沿用`NwdafMLModelProvSubsc`、`NwdafMLModelProvNotif`、`MLEventNotif`與
   `mLFileAddr.mLModelUrl` standard semantics。
7. Go只驗證、mirror並route標準payload；artifact binary由AnLF backend直接向MTLF backend URL下載。
8. AnLF backend下載、檢查package、完整載入並atomically bind後才register monitoring capability。
9. compatible analytics subscriptions共用provision resource與loaded model；必要時以standard PUT更新同一resource
   的active-demand union，不建立第二次POST/download。不同group/filter/target仍建立獨立monitoring scope，任一
   scope可依MTLF policy觸發一次model-level retrain。
10. model package沿用`nwdaf-daisy-improvement-plan.md`及current PyMTLF/PyAnLF已驗證的bundle、immutable
    repository、safe download/cache/load概念，但production命名與依賴
   不保留Daisy。
11. Phase 6只接local training、new artifact publication與updated/re-trained model reprovision，不重建Phase 4
    resource/download path。
12. 不建立custom `ModelReady`、base/target generation CAS、多狀態apply-result或active-generation
   reconciliation API。
13. Future multiple-AnLF以standard `consumerId`、event/filter/target隔離monitor scope；目前不實作AGG、AoI
    routing或FL，但不得以single AnLF/single scope作model policy identity。
14. Accuracy notification以standard `MLModelAccuracyInfo.deviation`承載WAPE ratio；不填percentage
    `mlModelAcc`。Data sufficient的period才提供`deviation`，不足時仍送合法periodic notify但不更新PyMTLF policy。
15. Phase 4 accuracy policy只保留degradation path；Phase 5 dataset retrieval只服務retraining/training，不再補足
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

狀態：foundation完成；unified readiness/sync correction已由Phase 3完成；dataset retrieval與final source
selection仍由Phase 5完成。

保留：

- AnLF/MTLF independent polling
- cached availability
- backoff、wake-up與shutdown ownership
- liveness/readiness routes
- disabled/configured/unavailable區分

後續phase責任：

- Phase 3已將MTLF-only handshake改成兩個backend都使用的readiness後sync，並移除Go作為future Mongo
  writer的assumption。
- Phase 5依dataset retrieval實際consumer完成`preferredSource`、live availability與effective source選擇。

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
- PyAnLF direct Mongo raw write
- PyAnLF standard-shaped ADRF storage request經Go送ADRF
- Mongo與ADRF各自使用independent bounded retry queue；drop-oldest bounded ingestion/analytics/storage
  buffers與startup validation
- unified AnLF/MTLF continuous reconnect/sync foundation與process incarnation UUID

### Phase 4: Initial Model Provision, ML Model Monitoring And Accuracy Policy

詳細計畫：`Phase 4 ML Model Monitoring And Accuracy Policy.md`

狀態：draft；initial provision、standard monitoring、WAPE measurement與degradation-only policy decisions已完成，
沒有未解的D1 blocker。

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

- MTLF source preference與fallback
- consume Phase 4 retrain intent中的triggering scope與active scope inventory；exact dataset composition留給Phase 5
  detailed decision
- standard ADRF retrieval subscribe/callback/unsubscribe經Go
- 完整`FetchInstruction`交付MTLF backend
- MTLF direct ADRF fetch
- MTLF read-only Mongo query
- 真實ADRF/Mongo process-level retrieval tests
- 不從dataset重建Phase 4 accuracy observation，也不承擔policy input enrichment

### Phase 6: Local Training And Model Update/Reprovision

- local trainer與job lifecycle
- 產生new artifact package並publish到既有MTLF-owned URL repository
- model generation/update completion
- retrained candidate至少對triggering scope與仍使用該model的healthy scopes執行planned validation，避免只修正
  degraded group卻讓其他group regression
- 透過Phase 4既有standard Model Provision subscription通知updated/re-trained model
- AnLF candidate download/load、failure保留old model與atomic runtime swap
- successful swap後重設相關monitor policy generation/windows
- 移除remaining custom generation/apply assumptions

### Phase 7: Legacy Removal And Closure

- 移除Go legacy MTLF scheduler、dataset provider及model coordinator
- 移除Daisy client/callback/config/test/dependency
- 移除被standard-shaped routes與direct data paths取代的custom APIs
- final NRF advertisement、config、naming與dead-code audit

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
- PyAnLF standard SMF request → Go → all selected SMFs
- SMF/UPF → PyAnLF direct notification → Mongo/ADRF storage
- first analytics demand → PyAnLF → Go → PyMTLF seed provision → Go callback → PyAnLF direct download/load
- same model + different groups → independent monitor notifications and policy scopes
- ADRF retrieval callback → Go → MTLF → direct ADRF fetch
- Mongo raw record → MTLF direct query
- accuracy notify → retrain → model provision → AnLF download/load

Go verification依`NWDAF/Makefile`執行`make test`、`make lint`、`make build`；Python repositories執行各自
formatter/linter/test。需要Mongo、NRF、SMF、UPF或ADRF的check必須分開標示unit、process-level及environment
integration，不得以mock test宣稱完整驗證。

---

## 11. Explicitly Deferred

- multiple-NWDAF federated learning
- AoI-based SMF candidate filtering
- UDM discovery與`Nudm_SDM_Get` Group ID member retrieval
- ADRF NRF discovery
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

1. Phase 5 storage preference/fallback的exact config shape及是否保留explicit dual preference。
2. runtime data collection持續失敗時，何時使用已negotiated standard failure notification或termination；
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
- private PUT create/upsert取代standard POST
- custom callback token/idempotency cache作為第一版必要條件
- custom accuracy report envelope
- Go-owned custom model provision binding/event coordination
- custom `ModelReady`與generation CAS
- Go/PyMTLF active-generation reconciliation

historical detailed plans仍可說明已實作過什麼，但新的implementation與review一律以本文件及最新feature
detailed plan為準。
