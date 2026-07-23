# Phase 5 Dataset Selection And Direct Retrieval

Date: 2026-07-24

Status: Detailed plan drafted; implementation has not started

Parent plan:

- `MTLF Backend Transition Plan.md`

Previous phase:

- `Phase 4 ML Model Monitoring And Accuracy Policy.md`

---

## 1. Purpose

本phase把Phase 4產生的retrain intent接到可供後續local training使用的raw dataset，但不在本phase訓練模型。
完成後的責任邊界為：

1. PyMTLF依config preference、Go同步的live capability與本身實際read capability，選擇ADRF或MongoDB。
2. PyMTLF以Phase 4的triggering scope及同一model的active scope inventory決定dataset範圍。
3. ADRF的standard retrieval subscribe與unsubscribe仍由Go以NWDAF identity送出；request body由PyMTLF依
   Release 18 schema建立，Go只做validation、callback URI internalization、routing及standard error mapping。
4. ADRF callback先到Go；Go只在完整notification已由PyMTLF接受並保存後回204。
5. PyMTLF取得完整`FetchInstruction`後，以其中的`fetchCorrIds`直接呼叫ADRF
   `Nadrf_DataManagement_RetrievalRequest`；dataset bytes不經Go。
6. ADRF不存在或不能完成本次dataset時，PyMTLF可依明確規則直接read-only query由PyAnLF寫入的MongoDB raw
   notification collection；Go不提供Mongo query API。
7. 同一model由任一group觸發retrain時，dataset收集所有active scopes的資料；triggering scope只保留
   audit/observability語意，不預先指定training-only或validation-only角色。
8. 現有Go ADRF retrieval mechanics中仍有意義的window、watchdog、callback convergence、dedup及cleanup語意
   會等價移植；Go-owned dataset、Daisy與normalized Mongo record不保留。

TS 23.288 clause 6.2E.3.3 step 9明確允許MTLF在monitor notification沒有攜帶network data時，向AnLF、ADRF
或其他5GS entities取得retraining資料。TS 23.288 clause 6.2E.2 steps 8至10也把ADRF historical retrieval列為
MTLF可用資料來源。因此這個phase是Phase 4 degradation decision與Phase 6 local training之間的資料邊界，
不是accuracy policy的第二套input path。

---

## 2. Active Implementation Slice

### 2.1 Vertical Flow

```text
PyMTLF accuracy policy creates one model-level RetrainIntent
    -> retrieval coordinator snapshots one historical TimeWindow
    -> resolve triggering scope and all active scopes to accepted Events/SMF resources
    -> select preferred live source

ADRF path:
    PyMTLF -> standard-shaped RetrievalSubscribe -> Go -> ADRF
    ADRF -> complete RetrievalNotify -> Go -> PyMTLF
    PyMTLF -> direct GET using FetchInstruction -> ADRF
    PyMTLF -> standard-shaped RetrievalUnsubscribe -> Go -> ADRF

Mongo path:
    PyMTLF -> direct read-only query -> PyAnLF raw notification collection

Both paths:
    raw records -> scope-partitioned DatasetSnapshot
    -> READY for Phase 6, or FAILED with explicit fallback/cleanup outcome
```

### 2.2 Repositories

| Repository | Responsibility |
|---|---|
| `PyMTLF/` | source selection、scope resolution、retrieval job、ADRF notification state、direct fetch、Mongo query、raw dataset snapshot |
| `NWDAF/` | standard ADRF subscribe/unsubscribe client、callback handler、private standard-shaped routing、process-local route mirror、sync projection |
| `PyAnLF/` | 維持既有raw Mongo write contract及依effective source的storage behavior；只在contract有實際缺口時調整 |
| `nwdaf-resources/` | 真實Go/Python/ADRF/Mongo跨process harness與操作說明 |
| `nwdaf-docs/` | canonical design、規格證據、decision、review與completion record |

### 2.3 Included

1. PyMTLF dataset source config、live source selection與bounded fallback。
2. Retrain intent的single-consumer handoff及簡單job lifecycle。
3. Triggering/active monitor scope到Events Subscription、SMF resource與Mongo correlation的解析。
4. Go送出完整`NadrfDataRetrievalSubscription`及保存ADRF resource Location。
5. Go接受完整`NadrfDataRetrievalNotification`，並同步交付PyMTLF。
6. PyMTLF直接處理`FetchInstruction`及direct `GET /data-store-records`。
7. PyMTLF直接read-only query MongoDB raw notifications。
8. 同一固定historical window、scope partition、dedup、completeness及fallback rules。
9. Backend reconnect、PyMTLF restart及cleanup-only reconciliation。
10. Unit、contract及真實process-level retrieval tests。
11. API/config/runtime文件及parent plan進度更新。

### 2.4 Explicitly Deferred

- Local trainer、feature extraction、training tensor、model fit及validation。
- New generation、artifact publication及updated/re-trained Model Provision notify；由Phase 6完成。
- 從retrieval dataset重新計算Phase 4 WAPE、baseline或degradation。
- Dataset persistence、job history database、distributed queue或cross-Go-restart recovery。
- ADRF NRF discovery；第一版沿用Go固定ADRF endpoint。
- UDM user-consent retrieval/subscription implementation。
- `DataSetTag`、ADRF model storage、DCCF及Nnwdaf_DataManagement alternative。
- Multiple-NWDAF FL、AGG、AoI routing及leaf dataset federation。
- Production TLS、OAuth delegation與Python backend NF identity。
- Go legacy retrieval/Daisy code的最終刪除；無production caller後由Phase 7處理。

---

## 3. Evidence Baseline

### 3.1 Normative Procedure Evidence

| Behavior | Normative evidence | Consequence |
|---|---|---|
| MTLF可在degradation後另行取得retraining資料 | TS 23.288 clause 6.2E.3.3 step 9 | 不要求monitor notification承載private dataset context |
| MTLF可從ADRF取得historical data | TS 23.288 clause 6.2E.2 steps 8至10 | ADRF是合法來源，dataset選擇留給MTLF local logic |
| Retrieval create是POST | TS 29.575 clause 4.2.2.6.2；`TS29575_Nadrf_DataManagement.yaml` | 不使用private PUT假裝standard create |
| Create成功回representation及Location | TS 29.575 clause 4.2.2.6.2 | Go必須驗證201、body與mandatory Location |
| Callback先保存再回204 | TS 29.575 clause 4.2.2.8.2 | Go不能只抽出fetch IDs後提早回204 |
| Fetch instruction可驅動後續GET | TS 29.575 clauses 4.2.2.8.2、4.2.2.5.2 | PyMTLF直接使用完整instruction；Go不代理bytes |
| Consumer-triggered notification只送fetch instruction | TS 29.575 clause 5.1.6.2.4 `consTrigNotif`；`TS29575_Nadrf_DataManagement.yaml` | 本phase設為true，因此`dataNotif`/`anaNotifications`不是所選profile的正常結果 |
| Fetch instruction欄位與cardinality | TS 29.576 clause 5.2.6.2.3；`TS29576_Nmfaf_3caDataManagement.yaml` | required `fetchUri`及1..N `fetchCorrIds`，`expiry` optional |
| Retrieval time window欄位 | `TS29122_CommonData.yaml` `TimeWindow` | `startTime`及`stopTime`皆required |
| Direct GET有200與204 | TS 29.575 clause 4.2.2.5.2 | 204是成功但無matching data，不是transport failure |
| Unsubscribe是DELETE document resource | TS 29.575 clause 4.2.2.7.2 | 必須保存peer Location/resource identity以供cleanup |

Relevant local normative sources:

- `nwdaf-docs/specs/TS 23.288/6 Procedures to Support Network Data Analytics/6.2E MTLF-based ML Model Accuracy Monitoring.md`
- `nwdaf-docs/specs/TS 23.288/6 Procedures to Support Network Data Analytics/6.2 Procedures for Data Collection/6.2.9 User consent for analytics.md`
- `nwdaf-docs/specs/TS 29.575/4 Services offered by the ADRF/4.2 Nadrf_DataManagement Service/4.2.2 Service Operations/`
- `nwdaf-docs/specs/TS 29.575/5 API Definitions/5.1 Nadrf_DataManagement Service API/5.1.6 Data Model.md`
- `nwdaf-docs/specs/TS 29.576/4 Services offered by the MFAF.md`
- `nwdaf-docs/specs/TS 29.576/5 API Definitions/5.2 Nmfaf_3caDataManagement Service API.md`
- `nwdaf-docs/specs/openapi/TS29122_CommonData.yaml`
- `nwdaf-docs/specs/openapi/TS29575_Nadrf_DataManagement.yaml`
- `nwdaf-docs/specs/openapi/TS29576_Nmfaf_3caDataManagement.yaml`

### 3.2 Retrieval Wire Contract

Release 18 `NadrfDataRetrievalSubscription` requires:

- `notifCorrId`
- exactly one of `anaSub`, `dataSub`, `dataSetId`
- `notificationURI`
- `timePeriod`
- optional `consTrigNotif`

本phase只建立`dataSub`，且其內容是PyAnLF先前向SMF建立collection時被接受的standard
`NsmfEventExposure` subscription representation。每個ADRF retrieval subscription使用獨立UUIDv4
`notifCorrId`；不得把同一job ID重複用於多個subscription，因TS 29.575 data model要求同一consumer內的
notification correlation可唯一識別subscription。

Release 18 `NadrfDataRetrievalNotification` requires:

- `notifCorrId`
- `timeStamp`
- exactly one of non-empty `anaNotifications`, `dataNotif`, `fetchInstruct`
- optional `terminationReq`

本phase的唯一正常profile為`fetchInstruct`，因request明確設`consTrigNotif = true`。TS 29.575
clause 5.1.6.2.4及OpenAPI description都規定，true表示ADRF緩衝notification data，只向consumer送
fetch instruction，直到consumer使用Nadrf_DataManagement要求delivery；該欄位省略時default為false。

Go wire validator仍要保留三種standard schema alternative的完整raw JSON；不得因Go只負責routing，就把
callback contract私自縮成只有fetch IDs。PyMTLF coordinator依建立該subscription時保存的profile處理：

1. 消費`fetchInstruct`。
2. 對`consTrigNotif = true`的subscription收到`dataNotif`或`anaNotifications`時，先保存完整callback作
   observability，再標記為peer/profile mismatch並終止/fallback；不得把它當成正常dataset捷徑。
3. 只要PyMTLF已成功保存並記錄上述outcome，Go仍依TS 29.575 clause 4.2.2.8.2回204；若無法保存才回
   declared error。這區分standard body shape與本次subscription選定的procedure semantics。

`FetchInstruction`的authoritative `$ref`指向
`nwdaf-docs/specs/openapi/TS29576_Nmfaf_3caDataManagement.yaml`；`timePeriod`指向
`nwdaf-docs/specs/openapi/TS29122_CommonData.yaml#/components/schemas/TimeWindow`。兩份exact Release 18
attachments目前都已納入local corpus，正式約束為：

- `FetchInstruction` required `fetchUri`及non-empty `fetchCorrIds`，optional `expiry`。
- `TimeWindow` required `startTime`及`stopTime`；本phase兩者都使用同一workflow snapshot的UTC DateTime。

TS 29.576 clause 4.3.2.2.2所描述的`POST {fetchUri}`是MFAF
`Nmfaf_3caDataManagement_Fetch`程序；本phase不是MFAF consumer，不套用該POST。ADRF callback依
TS 29.575 clause 4.2.2.8.2使用相同`FetchInstruction`資料型別，但後續資料取得是TS 29.575 clause 4.2.2.5
的`Nadrf_DataManagement_RetrievalRequest` GET，query使用`fetch-correlation-ids`。TS 29.575亦明確註記
ADRF情境中的`fetchUri`預期是standard data-store-records URI，但可能是任意值，因consumer可使用
fetch correlation IDs與已知ADRF service endpoint。因此：

1. Go sync提供configured ADRF Data Management apiRoot給PyMTLF；它是direct access endpoint，不是credential。
2. PyMTLF以該apiRoot建立standard `/nadrf-datamanagement/v1/data-store-records` GET。
3. `fetchUri`仍完整保存；只有當它normalize後與同一configured ADRF standard endpoint一致時才可直接採用。
4. 不因`fetchUri`指向MFAF-style callback就改送POST，也不跟隨它到任意host。

Current Go OpenAPI dependency仍沒有完整generated Nadrf service/types，且full corpus尚有其他NF API的unresolved
external refs。缺少generated type時可建立isolated compatibility wire package，但欄位與required/cardinality
必須直接來自上述exact attachments，不再把`FetchInstruction`或`TimeWindow`標示為schema gap：

- Go保留original JSON bytes，typed validation依exact Release 18 constraints。
- Python Pydantic wire models使用aliases及`extra="allow"`，避免future extension被drop。
- 不以untyped business map取代contract。

`resources/openapi/openapi/models/model_fetch_instruction.go`與`model_time_window.go`現在只作舊版
interoperability characterization，不再作這兩個schema的設計依據。

### 3.3 HTTP Status Matrix

依`TS29575_Nadrf_DataManagement.yaml`：

| Operation | Success | Declared error/redirect status |
|---|---|---|
| Retrieval subscription POST | `201` + JSON representation + mandatory `Location` | `400, 401, 403, 404, 411, 413, 415, 429, 500, 502, 503, default` |
| Retrieval notification POST | `204` | `307, 308, 400, 401, 403, 404, 411, 413, 415, 429, 500, 502, 503, default` |
| Data store records GET | `200` + `NadrfDataStoreRecord`, or `204` | `307, 308, 400, 401, 403, 404, 406, 429, 500, 502, 503, default` |
| Retrieval subscription DELETE | `204` | `307, 308, 400, 401, 403, 404, 429, 500, 502, 503, default` |

Mapping rules:

1. Malformed JSON、mandatory/cardinality failure或semantically impossible callback回`400 ProblemDetails`。
2. Well-formed但`notifCorrId`沒有active route回404。
3. Unsupported media type回415；request too large回413。
4. MTLF backend configured但unavailable、sync未完成或無法保存notification時回503。
5. ADRF transport failure或ADRF成功status但missing/invalid required response contract，Go private response映射502。
6. ADRF well-formed declared error保留原status及`ProblemDetails`；private backend error body不可直接外露。
7. Direct GET由PyMTLF處理200、204、307及308；redirect有bounded limit，並維持本phaseplain HTTP boundary。
8. 429及5xx為retryable；明確4xx除redirect外視為permanent for current attempt。
9. DELETE 404表示peer resource已不存在，可視為cleanup terminal outcome；wire仍保留standard 404，不由Go
   無條件改寫204。

### 3.4 User Consent Boundary

TS 29.575 clauses 4.2.2.5.1及4.2.2.6.1指出，若資料以SUPI/GPSI為target，consumer在retrieval前需要向UDM
確認user consent。TS 23.288 clause 6.2.9也把SUPI、group與ML Model training納入依local policy/regulation
執行的consent scope。

目前workspace沒有完成這條UDM consent flow。本phase不得宣稱full consent compliance，也不得在Go/Python之間
發明一個boolean代替UDM程序。第一版需要明確採用以下實驗環境前提：

- local policy設定為此受控實驗不要求user-consent enforcement；
- config與啟動log必須明確揭露該模式；
- 若未來啟用consent-required模式但UDM integration尚未完成，dataset retrieval fail closed，不得開始
  ADRF subscription或Mongo read；
- 真正的group-to-SUPI consent resolution與consent change subscription另立future plan。

這是implementation前仍需使用者確認的decision gate，見Section 14。

### 3.5 free5GC Structural Alignment

本phase依`free5gc-dev-skill`的最小適用範圍讀取NF architecture、SBI、OpenAPI contract、testing與
build/run references。Local free5GC snapshot沒有ADRF consumer exemplar，因此：

- Go outbound client ownership/cache/error boundary以
  `resources/references/free5gc-main/NFs/udm/internal/sbi/consumer/`作結構參考。
- Callback handler與processor分離、成功處理後回204，以
  `resources/references/free5gc-main/NFs/pcf/internal/sbi/`的callback/notifier pattern作結構參考。
- ADRF method、payload、Location及status完全依TS 29.575/OpenAPI，不從UDM/PCF behavior類推。

Go仍遵守`router -> handler -> processor -> consumer/context`邊界。PyMTLF不因direct ADRF GET而成為獨立NF；
它是同一NWDAF中的MTLF backend，而Go保留standard subscribe/unsubscribe與callback ingress。

---

## 4. Current State

### 4.1 Phase 4 Retrain Intent

PyMTLF目前在policy hit時建立：

```text
RetrainIntent(
    model_key,
    triggering_scope_key,
    active_scope_keys,
    created_at
)
```

同一model只有一個retrain-in-flight；trigger後該model各scope的decision hit window會reset。Intent目前只存在
in-memory list，沒有production consumer。Canonical scope key含provider/model、AnLF consumer identity、
ML event、filter及target，因此不同group可獨立監控，但仍共享model-level retrain。

缺口是scope key本身不是data query。Phase 5需要在intent產生時取得stable structured scope snapshot，
再以Go sync中的Events Subscription與SMF resource snapshot解析出實際collection resources；不能從JSON key
字串反向猜測。

### 4.2 Existing Unified Sync

Go、PyAnLF與PyMTLF已共用：

```text
POLLING -> READY -> SYNCING -> USABLE
```

`SyncRequest`已定義`eventsSubscriptions`、`smfResources`、`dataSourceAvailability`及
`mtlfSourceSelection`。Go目前只把Events/SMF snapshots送給AnLF；PyMTLF雖可parse欄位，實際收到空list。
`SmfResourceSnapshot`的Go contract已有accepted standard subscription raw JSON，但PyMTLF model尚未保留
`subscription`。PyMTLF sync response目前也固定回`mongodbAvailable = false`與空selection。

本phase延伸既有sync，不建立新的collection-requirements或dataset-control API：

1. Go把PyMTLF真正需要的Events/SMF snapshot送給MTLF backend。
2. PyMTLF `SmfResourceSnapshot`保留accepted standard subscription。
3. PyMTLF回報其Mongo read probe與source selection。
4. Go把selection同步給PyAnLF，沿用現有storage sink selection。

### 4.3 Existing Mongo Contract

PyAnLF已直接寫入configurable Mongo database/collection，sample default為：

```text
database: free5gc
collection: nwdaf_raw_notifications
```

每個document保存一筆完整raw notification：

```json
{
  "source": "smf-or-upf",
  "receivedAt": "<DateTime>",
  "measurementTime": "<DateTime>",
  "correlationId": "<collection correlation>",
  "notification": {}
}
```

Existing indexes：

- `receivedAt`
- `(correlationId, measurementTime)`
- `(source, measurementTime)`

PyAnLF的Mongo與ADRF write queues互相獨立、bounded且drop-oldest。MTLF selection為空時，PyAnLF向所有可用
sink寫；selection存在後只寫effective source。Phase 5不改raw schema、不增加schema version，也不讓PyMTLF
寫Mongo或建立migration/index。

### 4.4 Existing Go ADRF Retrieval

Go legacy flow已具備以下可作characterization的mechanics：

- 一次workflow固定historical window，default 1800秒。
- callback後reset watchdog，default 120秒。
- callback可能一次含多個fetch IDs。
- fetch ID dedup、termination與queue drain共同決定completion。
- direct GET 204代表no data。
- workflow完成/失敗後best-effort unsubscribe。

但current ownership與target不再正確：

- Go建立retrieval request、只抽出fetch IDs、直接fetch dataset。
- callback在backend真正保存前便回204。
- 同一task ID被用作多個subscription correlation。
- query context來自legacy observation sources，不是Phase 4 active monitor scopes。
- 結果送往Daisy/legacy normalized dataset。

本phase保留前述mechanics，明確取代後述ownership及Daisy path。

### 4.5 Local ADRF Interoperability Target

`resources/adrf`是read-only integration target，不是規格source。Current V0 behavior：

- retrieval `dataSub`只支援`smfDataSub.supi`；
- 要求`consTrigNotif = true`；
- create時snapshot符合SUPI、measurement `startTime`及ingestion cutoff的records；
- callback可batch fetch IDs，最後以`terminationReq = true`結束，即使零資料仍送termination；
- direct GET一次只接受一個fetch ID；
- DELETE對不存在resource也回204；
- callback delivery目前沒有retry。

Production wire仍依Release 18 standard；以下只作V0 compatibility constraint：

- PyMTLF第一版每個GET送一個fetch ID。
- 每個distinct accepted SMF data subscription建立一個ADRF retrieval subscription。
- process-level test必須覆蓋zero-data termination及callback沒有retry時的接收可靠性。

`terminationReq`在Release 18 notification中是optional；規格沒有為所有ADRF implementation提供通用的
「這個historical subscription已送完全部records」marker。Current workspace ADRF會在最後一個callback設
`terminationReq = true`。本phase第一版以這個integration profile作完整性條件：沒有termination時，
watchdog只能判定本次資料集合無法證明完整，必須fallback/FAILED，不能因已收到部分records便READY。

---

## 5. Confirmed Architecture And Proposed Decisions

### 5.1 Already Confirmed

1. Python backend不是獨立標準NF；Go不使用`PyMTLF`命名。
2. Standard ADRF subscribe/unsubscribe由Go送；PyMTLF建立完整standard-shaped request。
3. ADRF callback先到Go，再把完整notification交給MTLF backend。
4. PyMTLF直接向ADRF fetch；Go不取得dataset bytes。
5. PyMTLF直接read-only query Mongo；Go不提供query proxy。
6. Mongo保存PyAnLF收到的完整raw notification，不使用legacy normalized Go records。
7. Source agreement沿用unified health/sync；不是額外fancy management API。
8. Go state只在memory；Go restart視為實驗重跑，不做durable recovery。
9. Plain HTTP；本phase不加入internal或external security機制。
10. Phase 4 accuracy logic、WAPE report及degradation decision不因dataset retrieval而改寫。
11. 任一scope觸發model-level retrain後，所有active scopes的資料都納入同一training dataset候選集合；
    Phase 5只保留scope partition及trigger metadata，不決定scope-specific weighting或train/validation split。
12. 每個active scope都是required；任一scope完全沒有valid record時先fallback，alternate仍不足則job FAILED，
    不靜默使用缺少部分active scopes的dataset。

### 5.2 Proposed Source Configuration

第一版只提供single preference，不提供`dual` retrieval mode：

```yaml
dataset:
  preferred_source: adrf
  retrieval_window_seconds: 1800
  watchdog_timeout_seconds: 120
  fetch_timeout_seconds: 120
  max_redirects: 3
  max_retry_attempts: 3
  retry_initial_backoff_seconds: 1
  retry_max_backoff_seconds: 30
  max_concurrent_jobs: 2
  max_records_per_job: 100000
  consent_mode: experimental-no-enforcement
  mongodb:
    url: mongodb://127.0.0.1:27017
    database: free5gc
    collection: nwdaf_raw_notifications
    connect_timeout_ms: 5000
    read_timeout_ms: 30000
```

Rules：

1. `preferred_source`只允許`adrf`或`mongodb`。
2. Preferred read capability與Go宣告的write/capability都可用時選preferred。
3. Preferred unavailable時選alternate。
4. 不同source的raw records不在同一job內merge，以免重複與coverage不一致。
5. 每次job最多一次source fallback；禁止在兩個source間無限切換。
6. `fetchBatchSize`不成為general config；因local ADRF V0限制，direct GET adapter固定一次一個ID，待peer支援
   standard multi-ID後才另立decision。

此設計保留實驗可調性，同時避免`preferred/dual/strict/fallback`形成沒有consumer的複雜mode matrix。

### 5.3 Fallback And Storage Coverage

Selection完成後PyAnLF目前只寫effective source。因此alternate source可能沒有本次完整historical window。
Fallback不得只因查到幾筆資料就宣稱dataset完成：

1. Job在開始時固定`startTime`、`stopTime`及required scope/resource set。
2. Preferred source失敗或required scope為空時，可嘗試alternate。
3. Alternate只有在每個required scope都有有效records時才成為READY。
4. Alternate不完整時current job FAILED；PyMTLF可把alternate設為新的effective source，讓PyAnLF未來改寫，
   但不能補造已遺失的歷史資料。
5. 若兩個source都可用但當時尚未建立selection，bootstrap dual-write資料可自然讓兩邊都具備coverage。

本phase不默默把既有協議改成永遠dual-write。如未來要求high-availability historical fallback，需要另行決定
持續dual-write的storage cost與consistency policy。

### 5.4 Dataset Completeness

Proposed first-version rule：

- Triggering scope與intent建立當下的所有active scopes都是required。
- Dataset按scope分區，不將不同group混成無標記record list。
- 每個required scope至少一筆valid raw record，整個snapshot才是READY。
- Triggering scope只保留為retrain cause及observability metadata，不取得特殊sample role或weight。
- 所有active scopes的records都進入Phase 6可使用的training dataset候選集合；沒有scope被預先限制為
  validation-only。
- Phase 5不進行ML train/validation split。Phase 6可再從所有scope的資料分別切出training及held-out samples，
  但不能把某個healthy scope整體排除在training dataset之外。
- 重複record以source-native stable identity優先；缺少identity時以canonical raw payload +
  correlation + measurement time計算local dedup key。
- Malformed individual record記錄並skip；若因此任何required scope變空，job不能READY。

這裡的「complete」是可實作的operational completeness，不代表能證明storage從未漏失任何measurement：

- ADRF path必須收到所有subscription termination、drain全部accepted fetch IDs且沒有terminal fetch error。
- Mongo path必須讓bounded query完整結束，不能因limit、cursor或connection error提前截斷。
- 兩條path都要求每個required scope至少一筆valid record。
- PyAnLF storage queue本來就是best-effort/drop-oldest；沒有upstream sequence/expected count時，Phase 5無法證明
  window內每個原始notification都曾被保存。Dataset snapshot必須保留drop/skip/count observability，Phase 6可
  依實驗需求再加入更高的minimum sample gate。

這使使用者原有「group A degraded、group B healthy、直接使用A+B全部資料retrain」的實驗語意可以延續到
Phase 6，同時不在Phase 5提前定義feature tensor、scope weighting或train/validation sampling。

---

## 6. Scope To Data-Resource Resolution

### 6.1 Required Snapshot

Retrain intent進入coordinator前，PyMTLF取得immutable structured snapshot：

```text
model identity
triggering monitor scope
all active monitor scopes for that model
accepted MLModelMonitor registration/subscription context
Go-synced Events Subscription resources
Go-synced SMF collection resources
fixed retrieval TimeWindow
```

Policy仍以原canonical scope key維持behavior parity；新增structured scope reference是additive handoff，
不能改變baseline、z-score、N-in-M或model-level in-flight判斷。

### 6.2 Resolution Algorithm

對每個active monitor scope：

1. 以`mLEvent`、`mLEventFilter`及`tgtUe`對照accepted Events Subscription。
2. 使用Events Subscription ID找出`SmfResourceSnapshot.nwdafSubscriptionIds`包含該ID的resources。
3. 排除`pendingCleanup = true`及缺少accepted subscription representation的resource。
4. 以`targetApiRoot + resourceLocation`作peer resource identity，不假設不同SMF回傳的subscription ID全域唯一。
5. 對ADRF path，從accepted `NsmfEventExposure`取出完整`smfDataSub`；local ADRF V0要求可解析SUPI。
6. 對Mongo path，以distinct `correlationId`查raw notifications。
7. 一個collection resource可被多個analytics scopes refcount共用；record在dataset中可refer到多個scope，
   但physical raw payload只保存一次。

Resolution ambiguity不得用「取第一個」掩蓋。零match、同一scope出現conflicting collection profile或accepted
subscription無法依Release 18 compatibility model解析，皆形成explicit incomplete reason並進入fallback/failure。

### 6.3 Sync Changes

Go送給MTLF backend的sync增加既有typed sections的實際內容，不新增另一條API：

- active Events Subscription accepted representation
- SMF resource correlation、target apiRoot、resource Location、refcounted NWDAF subscription IDs、
  pending cleanup及accepted standard subscription representation
- existing `containingNwdaf.apiBaseUri`，供PyMTLF建立Go-owned ADRF callback URL
- ADRF configured capability
- configured ADRF Data Management apiRoot，供PyMTLF直接執行standard RetrievalRequest GET
- PyAnLF回報的Mongo write availability
- current MTLF source selection

PyMTLF回應：

- `mongodbAvailable`: direct read probe結果
- `sourceSelection.preferredSource`
- `sourceSelection.effectiveSource`

Go只在值真正改變時喚醒另一backend refresh，避免sync update loop。MTLF backend在完整sync成功前不能consume
retrain intent；backend reconnect後以最新snapshot解析新job。

---

## 7. Dataset Job Model

### 7.1 Minimal State

第一版使用in-memory、one-model-one-job registry：

```text
PENDING -> RESOLVING -> RETRIEVING -> READY
                                \-> FAILED
```

Job保存：

- UUIDv4 job ID
- model identity
- triggering scope與required active scopes
- fixed UTC start/stop
- attempted/effective source
- per-scope resource IDs及record counts
- raw record dedup index
- ADRF subscription/correlation/termination/fetch state
- failure/fallback/cleanup outcome

不建立external dataset API、不寫dataset file、不建立SQLite/job database。Phase 6直接透過internal domain
interface取得READY snapshot。

### 7.2 Policy Handoff

1. Accuracy policy在既有lock內原子建立intent並設model in-flight。
2. Retrieval coordinator以single-consumer queue取走intent；network/database operation不得在policy lock內執行。
3. Job FAILED且cleanup完成後呼叫既有`complete_retrain(model_key)`，允許未來report重新觸發。
4. Job READY時保持model in-flight，等待Phase 6 consume並在training terminal result完成/失敗後release。
5. Phase 5單獨完成但Phase 6尚未實作期間，READY job會抑制同model的重複retrieval；這是有意的temporary
   handoff boundary，不是stuck bug。
6. PyMTLF shutdown停止accept新intent、取消workers、以獨立cleanup context嘗試DELETE active ADRF resources。

### 7.3 Time Window

Workflow開始時只呼叫一次clock：

```text
stopTime = workflow start UTC
startTime = stopTime - retrievalWindowSeconds
```

同一job的所有ADRF subscriptions與Mongo queries使用完全相同window。Mongo query採inclusive
`measurementTime >= startTime`及`<= stopTime`；結果依`measurementTime`、Mongo `_id`穩定排序。
不在每個scope或fallback時重新計算`now`，避免coverage drift。

### 7.4 Concurrency

- 同一model延續Phase 4 one-in-flight guarantee。
- 不同model可並行，但有configurable small global worker limit，避免同時建立無界ADRF subscriptions/DB cursors。
- Callback handler只保存notification及enqueue fetch work，不在HTTP request內等待全部GET。
- Fetch IDs全job dedup；同一callback replay不會重複append record。
- `terminationReq`是per-subscription terminal flag，不用callback數量猜測完成。
- ADRF job只有在所有subscription termination已收到且所有accepted fetch IDs已處理時才能進入completeness
  evaluation；terminal failure/timeout只能fallback或FAILED，不能以partial records進入READY。

---

## 8. ADRF Path

### 8.1 Create

每個distinct accepted SMF collection subscription：

1. PyMTLF先建立UUIDv4 `notifCorrId`並在local route table註冊，避免ADRF在POST response後立即callback時race。
2. PyMTLF建立完整`NadrfDataRetrievalSubscription`：
   - `dataSub.smfDataSub` = accepted SMF subscription
   - `notificationURI` = 由sync的`containingNwdaf.apiBaseUri`建立Go-owned retrieval callback URI
   - `timePeriod` = fixed workflow window
   - `notifCorrId` = per-subscription UUID
   - `consTrigNotif = true`，明確選擇只接收`fetchInstruct`再direct GET的consumer-triggered profile
3. PyMTLF以private standard-shaped POST交給Go。
4. Go驗證body與callback URI確實是自己的configured public callback，再呼叫configured ADRF
   `POST /nadrf-datamanagement/v1/data-retrieval-subscriptions`。
5. Go只在ADRF回201、valid JSON representation及mandatory Location時視為成功。
6. Go把201、Location與完整representation交回PyMTLF，並保存process-local peer route mirror。

Go config不搬到PyMTLF；固定ADRF endpoint及NWDAF external callback base仍由Go擁有。Private API path使用
`mtlfBackend`/`adrf-data-management`命名，不出現`pymtlf`。

Internal route matrix：

| Caller | Method and path | Success |
|---|---|---|
| MTLF backend -> Go | `POST /internal/v1/adrf-data-management/data-retrieval-subscriptions` | `201` + JSON + `Location` |
| MTLF backend -> Go | `DELETE /internal/v1/adrf-data-management/data-retrieval-subscriptions/{subscriptionId}` | `204` |
| ADRF -> Go | `POST /collector/retrieval-notify` | `204` |
| Go -> MTLF backend | `POST /internal/v1/adrf-data-management/retrieval-notifications` | `204` |

Private create仍使用POST；private body保留standard representation。DELETE除了path ID亦需由Go mirror解析原
target apiRoot/Location，MTLF backend不得在request body夾帶另一套custom dataset command。

### 8.2 Callback

Go保留既有public callback URI `/collector/retrieval-notify`，因TS 29.575允許subscription指定任意
`notificationURI`，無需為了視覺一致改path。

Handler flow：

```text
HTTP body limit/content-type
    -> typed + raw JSON validation
    -> processor resolves notifCorrId route
    -> synchronous private POST of complete notification to MTLF backend
    -> backend validates and stores it in its owned in-memory job state
    -> Go returns 204
```

「accepted and stored」在本phase只代表已進入仍存活PyMTLF process的owned job state，不代表disk
persistence。若backend unavailable或queue無法接受，Go回503，讓ADRF知道notification未被接受；不得先回204
再只寫log。

### 8.3 Direct Fetch

PyMTLF對每個新fetch ID：

1. 使用Go sync的configured ADRF Data Management apiRoot建立standard data-store-records URL；received
   `fetchUri`若normalize後是同一endpoint才可直接採用。
2. 以標準query name `fetch-correlation-ids`正確URL encode。
3. 第一版一次一個ID，以符合local ADRF V0；domain interface保留未來multi-ID能力。
4. 200時驗證完整`NadrfDataStoreRecord`，只接受和本job `dataSub`/scope相符的raw data。
5. 204標記該fetch ID completed-with-no-data。
6. optional `expiry`已過期時不fetch，形成explicit incomplete reason。
7. 307/308 bounded follow；429/5xx及network timeout capped retry；permanent 4xx停止該ID。
8. 每次有效callback/成功fetch progress reset 120秒default watchdog。

Configured ADRF apiRoot與可採用的`fetchUri`都必須是absolute HTTP(S) URI且不得帶userinfo、query或fragment。
Redirect第一版只允許configured ADRF same-origin，避免callback把backend導向任意新host。這是HTTP client
validation，不導入TLS、OAuth或key exchange。TS 29.576的MFAF Fetch POST不在這條direct ADRF path中。

PyMTLF不得透過Go抓資料，也不得讓Go做record normalization、chunk completion或dataset merge。

### 8.4 Completion And Cleanup

- 收到某subscription的`terminationReq = true`後不再期待新instruction，但已接受的fetch IDs仍要drain。
- 全部required subscriptions terminal且fetch queue drained後評估scope completeness。
- Job成功、失敗、fallback或shutdown皆發出cleanup intent。
- PyMTLF以Go private standard-shaped DELETE要求刪除peer Location。
- Go依保存的exact target/Location呼叫ADRF，不重新discovery或自行重組錯誤apiRoot。
- Cleanup retry有上限並保留observability；cleanup failure不把已READY dataset改成資料內容失敗，但job需記錄
  leaked-resource risk。

---

## 9. Mongo Path

### 9.1 Ownership And Dependency

PyMTLF新增`pymongo` runtime dependency與read-only repository。Connection settings屬PyMTLF local config，
sync只交換availability與selection，不交換credential/key。

Repository startup/refresh probe：

- parse config
- connect with bounded server-selection/connect/read timeout
- `ping`
- verify configured database/collection可read
- 不create index、不write probe document

### 9.2 Query

每個job先取得所有required scopes的distinct correlation IDs，再執行bounded query：

```javascript
{
  "correlationId": {"$in": ["..."]},
  "measurementTime": {
    "$gte": "<fixed startTime>",
    "$lte": "<fixed stopTime>"
  }
}
```

`source`只在scope明確要求時加入filter；不能假設所有historical documents都使用單一source字串。Query結果：

1. 驗證`correlationId`、`measurementTime`、`notification`。
2. 依correlation-to-scope map分區。
3. 保留原始`notification`及最小provenance，不把它轉成training feature。ADRF與Mongo只統一外層
   `DatasetRecord` provenance/identity/scope envelope，不改寫內層standard notification payload。
4. Deterministic sort與dedup。
5. Cursor/record數有safety limit；超限形成explicit failure，不靜默截斷後宣稱完整。

Connection/query failure觸發一次alternate fallback。單筆malformed document只skip並計數；required scope因此
為空時整個dataset不READY。

### 9.3 Read/Write Availability Distinction

Go的`dataSourceAvailability.mongodb`目前來自PyAnLF write availability；PyMTLF也要回報自己的read
availability。Effective source只有在兩者皆true時才能選Mongo：

```text
Mongo usable = AnLF can write AND MTLF can read
ADRF usable = Go has ADRF standard client capability AND MTLF can direct-fetch the advertised URI
```

ADRF direct-fetch capability無法只靠Go config完全證明；第一個job的connectivity/response結果仍可能導致
runtime fallback。Availability是current capability hint，不是資料完整性保證。

---

## 10. Restart And Reconciliation

### 10.1 Same PyMTLF Process Reconnect

Go到同一`processInstanceId`的transport短暫中斷後：

- Go重新poll/sync。
- PyMTLF保留in-memory jobs及notifCorrId routes。
- Go以process-local mirror重新建立callback routing。
- Unfinished workflow可繼續，前提是ADRF callback/fetch state仍在PyMTLF。

### 10.2 PyMTLF Process Restart

新的`processInstanceId`代表job memory已遺失。本phase不假裝能恢復dataset：

1. Go辨識process incarnation改變。
2. 舊ADRF retrieval resources標為cleanup-only。
3. Go把可清理的peer Locations/correlations納入reconnect snapshot，或由Go cleanup worker執行DELETE。
4. 新PyMTLF不把舊callback接到新job；well-formed但沒有active route的`notifCorrId`回404，格式或
   cardinality錯誤回400。
5. Phase 4 policy state與intent也因backend process restart遺失；實驗由後續accuracy reports重新建立。

這是對parent plan「restore unfinished standard retrieval state」的限縮：沒有持久化job body時只能恢復
cleanup/routing知識，不能安全復活dataset completion state。

### 10.3 Go Restart

Go mirror不持久化。Go restart後舊ADRF subscription可能成為orphan，現階段接受實驗重跑與ADRF-side expiry/
manual cleanup。不得把此限制誤寫為durable recovery已完成。

---

## 11. Behavioral Inventory

### 11.1 Preserve

| Behavior | Historical source |
|---|---|
| 一次workflow固定historical window | `NWDAF/internal/mtlf/adrf_retrieval.go` |
| Default retrieval window 1800秒 | `NWDAF/pkg/factory/config.go` |
| Default callback watchdog 120秒且progress時reset | same |
| Fetch ID dedup | legacy Go retrieval job |
| Callback可batch IDs | legacy Go callback/retrieval tests |
| 204 direct GET表示no data | TS 29.575及legacy client tests |
| Termination + accepted fetch queue drain才完成 | legacy Go retrieval convergence |
| Job終止後cleanup subscriptions | legacy Go retrieval cleanup |
| Same model one retrain-in-flight | PyMTLF Phase 4 accuracy policy |
| Triggering scope及active scope inventory | PyMTLF Phase 4 retrain intent |

### 11.2 Explicitly Replace

| Old behavior | Replacement |
|---|---|
| Go決定dataset/query | PyMTLF owns selection and query |
| Go只抽取fetch IDs | complete standard notification forwarded |
| Go直接fetch ADRF bytes | PyMTLF direct fetch |
| Shared task ID作多個notif correlation | UUIDv4 per ADRF subscription |
| Legacy observation source context | Phase 4 scope + unified sync resource resolution |
| Go normalized Mongo traffic records | PyAnLF raw notification documents |
| Dataset records混成單一group label | scope-partitioned raw snapshot |
| Partial successful fetch可繼續legacy training | required scope completeness or fallback/fail |
| Watchdog close queue後仍提交legacy training | timeout永遠fallback/FAILED，不以partial records READY |
| Dataset交給Daisy | READY handoff to Phase 6 local trainer |
| Callback先204再處理 | backend accepted/stored before 204 |

### 11.3 Remove From Production Path

- Go legacy dataset provider/query active caller。
- Go direct ADRF fetch active caller。
- Go retrieval completion後的Daisy upload。
- Legacy normalized `upfTrafficData`作為new dataset source。
- Custom chunk/completion API及Go dataset byte proxy。

若legacy package因Phase 7 cleanup sequencing仍留在tree，必須沒有production caller並標記obsolete；不能讓新舊
retrieval coordinator同時消費同一retrain。

---

## 12. Implementation Slices

### Slice A: Contract And Source Agreement

1. 新增PyMTLF dataset config與validation。
2. 新增Mongo read probe、ADRF capability projection及source selector。
3. Go將Events/SMF snapshots送給MTLF backend。
4. PyMTLF接受SMF `subscription` raw representation。
5. PyMTLF sync回報Mongo read availability與preferred/effective source。
6. Go把selection轉送PyAnLF，驗證沒有refresh loop。

Acceptance：

- availability matrix deterministic；
- preferred unavailable可選alternate；
- 沒有source時backend仍USABLE，但retrain job在bounded retry budget後明確FAILED，不阻止monitoring。

### Slice B: Intent Handoff And Scope Resolution

1. 將RetrainIntent接到single-consumer coordinator。
2. 建立structured scope snapshot，不改policy algorithm。
3. 實作scope -> Events -> SMF resource解析。
4. 建立fixed TimeWindow及scope-partitioned job。
5. 覆蓋shared resource、不同SMF相同sub ID、pending cleanup、zero/ambiguous match。

Acceptance：

- group A trigger時job包含A及同model active group B；
- peer identity使用target + Location；
- 沒有從canonical JSON key反向猜資料resource。

### Slice C: Standard ADRF Control Plane Through Go

1. 建立scoped Release 18 compatibility wire types。
2. PyMTLF建立full RetrievalSubscribe request。
3. Go handler/processor/consumer送ADRF POST並保留201/Location/body。
4. Go mirror保存notif correlation與peer Location。
5. Go DELETE forwarding及status/error mapping。
6. 移除legacy coordinator的production subscription ownership。

Acceptance：

- exact method/path/body/status tests；
- malformed success response映射502；
- standard `ProblemDetails`及Location round trip。

### Slice D: Callback And Direct Fetch

1. Go callback改為完整notification validation/forwarding。
2. PyMTLF先註冊correlation再create，消除immediate callback race。
3. PyMTLF保存fetch instruction、dedup、expiry、termination及watchdog state；對
   `consTrigNotif = true`卻收到direct notification alternative的情況記錄profile mismatch並fallback。
4. PyMTLF direct GET 200/204/redirect/retry。
5. Completion與cleanup convergence。

Acceptance：

- Go只有backend接受後回204；
- zero-data termination可正常完成為incomplete/fallback，而非timeout；
- batch/replay/out-of-order callback不重複資料；
- dataset bytes不出現在Go API或state。

### Slice E: Mongo Direct Read And Fallback

1. PyMTLF read-only Mongo repository。
2. Fixed-window/correlation query、validation、sort、dedup與scope partition。
3. Preferred/alternate one-shot fallback。
4. Coverage gap與malformed-record outcomes。

Acceptance：

- direct Mongo path可生成和ADRF path相同domain shape的raw DatasetSnapshot；
- 不建立index、不write、不改PyAnLF schema；
- incomplete alternate不被誤標READY。

### Slice F: Process-Level Verification And Documentation

1. 在`nwdaf-resources/tests/mtlf_dataset_retrieval/`建立跨process harness。
2. 啟動Go NWDAF、PyAnLF、PyMTLF、workspace ADRF及MongoDB。
3. 透過PyAnLF callback/storage path寫入真實raw notifications。
4. 觸發Phase 4 policy intent並分別驗證ADRF、Mongo與fallback。
5. 驗證Go/backend restart boundary、callback 204 timing與cleanup。
6. 更新各repo API/config/runbook與parent plan。

Harness不放在`NWDAF/`、`PyAnLF/`或`PyMTLF/`內；單一repo只保留unit/contract tests。

---

## 13. Verification Matrix

### 13.1 PyMTLF

- Config defaults/invalid enum/timeout validation。
- Source selection完整availability matrix。
- Intent single consumption與one-model-in-flight。
- Scope resolution：multi-group、shared resource、multi-SMF collision、pending cleanup、missing accepted body。
- Fixed TimeWindow與inclusive Mongo query。
- Raw record validation/dedup/scope completeness。
- ADRF create race、callback replay、batch IDs、expiry、200/204、termination、watchdog。
- `consTrigNotif = true`只以`fetchInstruct`進入正常fetch path；其他合法schema alternative被完整保存並形成
  可觀察的profile mismatch。
- 429/5xx retry、4xx terminal、redirect bound。
- Preferred failure -> alternate complete/incomplete。
- Shutdown cleanup及READY/FAILED policy handoff。
- `uv run pytest -q`
- `uv run ruff check .`
- `uv run ruff format --check .`

### 13.2 NWDAF

- ADRF POST/DELETE exact method/path/query/header/body。
- 201 representation與mandatory Location。
- Full callback round trip及unknown Release 18 field preservation。
- 204 only after backend acceptance。
- 400/413/415/429/5xx/502/503 mapping。
- Backend unavailable/reconnect/process UUID change。
- MTLF sync收到Events/SMF snapshots且AnLF sync沒有regression。
- No production Go direct fetch/dataset caller。
- `make test`
- `make lint`
- `make build`

### 13.3 PyAnLF

- Selection空時dual-write、effective ADRF/Mongo時single sink behavior regression。
- Raw Mongo document contract與indexes regression。
- Sync selection refresh不形成loop。
- Existing full lint/test suite。

除非本phase真的修改PyAnLF，否則不為了測試方便新增production API。

### 13.4 Process-Level Versus Environment-Level

| Level | Required proof |
|---|---|
| Unit/contract | fake peer、deterministic clock、status/body/state transition |
| Workspace process-level | real Go/Python processes、real Mongo、real workspace ADRF、real HTTP callbacks/direct fetch |
| Full 5GC environment | NRF/SMF/UPF-generated data；本phase可記錄但不是completion gate |

不得以mock callback測試宣稱ADRF e2e完成，也不得把workspace process-level稱為完整5GC integration。

---

## 14. Decision Gates Before Implementation

Dataset composition及strict active-scope completeness已確認於Section 5.1，不再是decision gate。以下項目已有
recommended default，但仍需使用者確認後才鎖定：

1. **Source config**：只保留`preferred_source = adrf|mongodb`，preferred不可用時自動嘗試alternate；不提供
   explicit `dual` retrieval mode。Recommended: accept。
2. **Historical fallback gap**：selection後不改成永久dual-write；alternate缺完整window時current job失敗，
   只切換未來write selection。Recommended: accept。
3. **User consent**：第一版明確使用`experimental-no-enforcement` local policy；若配置要求consent則fail closed，
   UDM integration另案。Recommended: accept for current controlled experiment。
4. **READY handoff**：Phase 5完成dataset後保持model retrain-in-flight，直到Phase 6 training terminal outcome；
   不因尚未實作trainer而反覆抓同一model資料。Recommended: accept。

這些decision之外，implementation不得自行改變Phase 4 accuracy policy、PyAnLF raw Mongo schema、既有
storage selection agreement或Go/Python ownership。

---

## 15. Completion Criteria

本phase只有同時滿足以下條件才算完成：

1. Phase 4 retrain intent能自動且只被一個retrieval job消費。
2. Same model/different group形成一個model-level job內的獨立scope partitions。
3. ADRF subscribe/callback/unsubscribe符合Release 18 method/body/status/Location。
4. Go收到完整callback並在PyMTLF接受後才回204。
5. PyMTLF以`fetchCorrIds`直接執行ADRF RetrievalRequest GET，Go完全不接觸dataset bytes。
6. PyMTLF可read-only query現有PyAnLF raw Mongo collection。
7. Preferred/fallback與incomplete history behavior可觀察、可測且不merge source。
8. Every required scope有資料才產生READY DatasetSnapshot。
9. Failure、timeout、restart與shutdown有bounded cleanup/retry，不留下失控worker。
10. Workspace ADRF及Mongo兩條real process path都通過。
11. NWDAF、PyAnLF、PyMTLF各自full lint/test/build通過。
12. 文件清楚區分normative spec、generated/local compatibility、workspace ADRF behavior及deferred limitation。
