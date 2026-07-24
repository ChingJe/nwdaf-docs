# Phase 5 Dataset Selection And Direct Retrieval

Date: 2026-07-24

Status: Implemented and verified across NWDAF, PyAnLF, PyMTLF, and nwdaf-resources; pinned free5GC NRF ADRF-discovery limitation recorded

Parent plan:

- `MTLF Backend Transition Plan.md`

Previous phase:

- `Phase 4 ML Model Monitoring And Accuracy Policy.md`

---

## 1. Purpose

本phase把Phase 4產生的retrain intent接到可供後續local training使用的raw dataset，但不在本phase訓練模型。
完成後的責任邊界為：

1. PyAnLF依固定ADRF-first政策決定新training data實際寫入ADRF、MongoDB或暫時不可用，並透過Go同步
   `trainingDataSource`；PyMTLF遵循該current source，不另做preferred/effective negotiation。
2. PyMTLF以Phase 4的triggering scope及同一model的active scope inventory決定dataset範圍。
3. ADRF的standard retrieval subscribe與unsubscribe仍由Go以NWDAF identity送出；request body由PyMTLF依
   Release 18 schema建立，Go只做validation、callback URI internalization、routing及standard error mapping。
4. ADRF callback先到Go；Go只在完整notification已由PyMTLF接受並保存後回204。
5. PyMTLF取得完整`FetchInstruction`後，以其中的`fetchCorrIds`直接呼叫ADRF
   `Nadrf_DataManagement_RetrievalRequest`；dataset bytes不經Go。
6. `trainingDataSource=mongodb`時，PyMTLF以和ADRF相同的`smfDataSub + timePeriod`語意直接read-only
   query由PyAnLF寫入的MongoDB collection；Go不提供Mongo query API。Job開始後不跨source fallback或merge。
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
    -> snapshot Go-synced trainingDataSource

ADRF path:
    PyMTLF -> generic NRF discovery through Go, or configured endpoint
    PyMTLF -> standard-shaped RetrievalSubscribe -> Go -> ADRF
    ADRF -> complete RetrievalNotify -> Go -> PyMTLF
    PyMTLF -> direct GET using FetchInstruction -> ADRF
    PyMTLF -> standard-shaped RetrievalUnsubscribe -> Go -> ADRF

Mongo path:
    PyMTLF -> same smfDataSub + TimeWindow query -> PyAnLF Mongo collection

Both paths:
    raw records -> scope-partitioned DatasetSnapshot
    -> READY for Phase 6, or FAILED with explicit source/cleanup outcome
```

### 2.2 Repositories

| Repository | Responsibility |
|---|---|
| `PyMTLF/` | ADRF discovery/configured resolution、training source projection、scope resolution、retrieval job、ADRF notification state、direct fetch、Mongo query、raw dataset snapshot |
| `NWDAF/` | generic NRF discovery/cache、standard ADRF subscribe/unsubscribe client、callback handler、private standard-shaped routing、process-local route mirror、sync projection |
| `PyAnLF/` | ADRF discovery/configured resolution、ADRF-first/Mongo fallback state、`trainingDataSource`回報及ADRF-aligned local Mongo record contract |
| `nwdaf-resources/` | 真實Go/Python/free5GC NRF/ADRF/Mongo跨process harness與操作說明 |
| `nwdaf-docs/` | canonical design、規格證據、decision、review與completion record |

### 2.3 Included

1. Go generic NRF discovery proxy、shared `SearchResult` cache及現有SMF caller regression。
2. PyAnLF/PyMTLF獨立ADRF `nrf`/`configured` config、candidate selection與local endpoint cache。
3. PyAnLF ADRF-first/Mongo fallback、ADRF recovery polling及`trainingDataSource` sync。
4. Retrain intent的single-consumer handoff及簡單job lifecycle。
5. Triggering/active monitor scope到Events Subscription、SMF resource與accepted `smfDataSub`的解析。
6. Go送出完整`NadrfDataRetrievalSubscription`及保存ADRF resource Location。
7. Go接受完整`NadrfDataRetrievalNotification`，並同步交付PyMTLF。
8. PyMTLF直接處理`FetchInstruction`及direct `GET /data-store-records`。
9. PyMTLF以`smfDataSub + timePeriod`直接read-only query MongoDB records。
10. 同一固定historical window、scope partition、dedup、completeness及source-cutover loss rules。
11. Backend reconnect、PyMTLF restart及cleanup-only reconciliation。
12. Unit、contract及真實process-level retrieval tests。
13. API/config/runtime文件及parent plan進度更新。

### 2.4 Explicitly Deferred

- Local trainer、feature extraction、training tensor、model fit及validation。
- New generation、artifact publication及updated/re-trained Model Provision notify；由Phase 6完成。
- 從retrieval dataset重新計算Phase 4 WAPE、baseline或degradation。
- Dataset persistence、job history database、distributed queue或cross-Go-restart recovery。
- Production UDM user-consent retrieval/subscription enforcement；current simulated experiment treats all
  involved UEs as consented and the backend transition does not add a consent mode/config surface。
- `DataSetTag`、ADRF model storage、DCCF及Nnwdaf_DataManagement alternative。
- Multiple-NWDAF FL、AGG、AoI routing及leaf dataset federation。
- Production TLS、OAuth delegation與Python backend NF identity。
- Go legacy retrieval/Daisy code的最終刪除；無production caller後由Phase 7處理。
- MongoDB fallback-period data回填ADRF、cross-source query/merge、source-history routing及historical continuity。
- 同一實驗期間SMF data subscription semantic profile改變或重建後的跨profile歷史查詢；第一版假設其保持一致。

---

## 3. Evidence Baseline

### 3.1 Normative Procedure Evidence

| Behavior | Normative evidence | Consequence |
|---|---|---|
| Generic NF discovery | TS 29.510 clause 5.3.2.2.2；`TS29510_Nnrf_NFDiscovery.yaml` `/nf-instances` | SMF與ADRF使用同一GET operation及完整`SearchResult`，不新增ADRF-specific discovery endpoint |
| Discovery result reuse | TS 29.510 clause 5.3.2.2.1；TS 23.501 clause 6.3.1 | 相同query且validity未到期時應重用；Go作為NRF caller建立backend-neutral shared cache並合併concurrent misses |
| Search result validity | TS 29.510 clause 6.2.6.2.2；`TS29510_Nnrf_NFDiscovery.yaml` `SearchResult` | `validityPeriod`與`nfInstances` mandatory；空result也可cache，validity與`Cache-Control: max-age`一致 |
| ADRF Data Management discovery | `TS29510_Nnrf_NFDiscovery.yaml` query parameters；`TS29510_Nnrf_NFManagement.yaml` `ServiceName` | 使用`target-nf-type=ADRF`及`service-names=nadrf-datamanagement`；第一版不依賴current free5GC尚未完整支援的ADRF capability filters |
| MTLF可在degradation後另行取得retraining資料 | TS 23.288 clause 6.2E.3.3 step 9 | 不要求monitor notification承載private dataset context |
| MTLF可從ADRF取得historical data | TS 23.288 clause 6.2E.2 steps 8至10 | ADRF是合法來源，dataset選擇留給MTLF local logic |
| ADRF raw storage record保留subscription與notification | TS 29.575 clauses 5.1.6.2.2、5.1.6.2.8、5.1.6.2.9；`TS29575_Nadrf_DataManagement.yaml` `NadrfDataStoreRecord`、`DataSubscription`、`DataNotification` | 本資料路徑使用`dataSub.smfDataSub + dataNotif`；Mongo採相同record語意是local fallback design，不宣稱為標準Mongo schema |
| ADRF retrieval以data specification及時間選取 | TS 29.575 clause 5.1.6.2.4；`TS29575_Nadrf_DataManagement.yaml` `NadrfDataRetrievalSubscription` | 本資料路徑使用完整`smfDataSub + timePeriod`作共同domain query；workspace ADRF V0與Mongo adapter第一版都從其中解析SUPI/time執行實際查詢 |
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
- `nwdaf-docs/specs/TS 29.510/5 Services Offered by the NRF/5.3 Nnrf_NFDiscovery Service/5.3.2 Service Operations/5.3.2.2 NFDiscover.md`
- `nwdaf-docs/specs/TS 29.510/6 API Definitions/6.2 Nnrf_NFDiscovery Service API/6.2.6 Data Model/6.2.6.2 Structured data types/6.2.6.2.2 Type SearchResult.md`
- `nwdaf-docs/specs/TS 23.501/6 Network Functions/6.3 Principles for Network Function and Network Function Service discovery and selection/6.3.1 General.md`
- `nwdaf-docs/specs/TS 29.576/4 Services offered by the MFAF.md`
- `nwdaf-docs/specs/TS 29.576/5 API Definitions/5.2 Nmfaf_3caDataManagement Service API.md`
- `nwdaf-docs/specs/openapi/TS29122_CommonData.yaml`
- `nwdaf-docs/specs/openapi/TS29575_Nadrf_DataManagement.yaml`
- `nwdaf-docs/specs/openapi/TS29576_Nmfaf_3caDataManagement.yaml`
- `nwdaf-docs/specs/openapi/TS29510_Nnrf_NFDiscovery.yaml`
- `nwdaf-docs/specs/openapi/TS29510_Nnrf_NFManagement.yaml`

### 3.2 Generic NRF Discovery And Cache Contract

Before the retrieval-specific contract, the phase also extends the existing private
NRF discovery boundary. The route remains:

```http
GET /internal/v1/nrf/nf-instances
```

The current path name is generic but its handler/processor/consumer only accepts and emits the fixed SMF query.
This phase replaces that implementation with a typed generic request:

```text
SMF:
  target-nf-type=SMF
  requester-nf-type=NWDAF
  service-names=nsmf-event-exposure

ADRF Data Management:
  target-nf-type=ADRF
  requester-nf-type=NWDAF
  service-names=nadrf-datamanagement
```

Go validates the containing identity, injects `requester-nf-instance-id`, forwards every supported query field
through the generated NFDiscovery client and returns the complete `SearchResult`. It does not create
`/adrf-discovery`, return a private endpoint array, or select an ADRF for either backend.
The route is mounted once on the containing NWDAF internal server advertised by
`containingNwdaf.internalCallbackBaseUri`; its handler/processor interface becomes backend-neutral instead of
remaining an AnLF/SMF-named capability, so PyMTLF can call the same URI without a parallel server route.

The Go cache key is the canonical form of the effective outbound query after defaults and identity injection.
Identical queries from PyAnLF and PyMTLF therefore share one entry; a configured-mode backend never calls this
route. Cache entries contain the complete standard result and absolute expiry. `validityPeriod = 0` is returned
but not stored. Missing, negative or malformed validity is a malformed successful NRF response and maps to
`502`; empty `nfInstances` is a valid cacheable result. Same-key concurrent misses use one outbound NRF call.
Related-but-not-identical query reuse, NRF NF-status subscriptions, persistent cache and cache sync are deferred.
Transport errors, redirects and non-200 `ProblemDetails` are never cached; only a contract-valid 200
`SearchResult`, including a valid empty result and `noProfileMatchInfo`, can create an entry.

On a cache hit, Go returns a copy whose `validityPeriod` is the floored remaining seconds until the stored
absolute expiry and sets `Cache-Control: max-age` to the same value. It never restarts the original full TTL at
the time a backend happens to query. If no positive whole second remains, the entry is expired and the request
performs a new NRF discovery. This keeps the existing PyAnLF/backend-local candidate cache within the NRF
validity boundary. A fresh result also sets the private response `Cache-Control: max-age` from the validated body
value, so backend callers always receive body/header values with the same lifetime even if the generated NRF
client does not expose the upstream header.

The current generated Go scalar represents an absent `validityPeriod` as zero, although zero is a distinct valid
value. The implementation must retain enough raw-response/presence information to distinguish missing from
explicit zero; it may not silently reinterpret a missing mandatory field as a zero-TTL success. Cache storage is
bounded by a first-version internal limit of `256`; insertion first removes expired entries and then evicts the
least-recently-used entry. This does not add another config surface. The bound affects reuse only and never
changes the standard discovery response.

The cache/handler must also preserve the complete Release 18 JSON envelope, including fields unknown to the
current free5GC generated model such as future profile extensions or `ignoredQueryParams`. A scoped
presence-aware compatibility envelope may wrap the generated types; converting the response into a reduced Go
model and re-marshalling it is not acceptable because it would silently drop fields before either backend
performs selection.

The Release 18 discovery API also defines `ml-model-storage-ind` and `data-storage-ind`, and the NF profile
defines `adrfInfoList`. The inspected workspace ADRF branch can register `NFType=ADRF` and the exact
`nadrf-datamanagement` service, while the current free5GC OpenAPI/NRF path does not fully implement those newer
capability filters. The first compatibility profile therefore discovers by NF type and exact service name and
does not treat the branch's non-standard `customInfo` as normative evidence.

Process verification found a stricter pinned-environment limitation: the local free5GC NRF under
`resources/references/free5gc-main/NFs/nrf` accepts the ADRF NF profile through NF Management, but its older
NF Discovery request validator rejects `target-nf-type=ADRF` with `400` before profile filtering. Therefore the
Release 18 NRF-mode contract is implemented and covered with standard-shaped NRF contract tests, while the
real-process data path uses the independently supported `configured` ADRF mode. No non-standard fallback through
NF Management listing is added. A real local NRF-mode process proof requires upgrading the pinned NRF schema;
this environment limitation is not hidden as a successful discovery.

### 3.3 Retrieval Wire Contract

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
   observability，再標記為peer/profile mismatch並終止current job；不得把它當成正常dataset捷徑或改查另一source。
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

1. PyMTLF使用自己的configured或NRF-discovered ADRF Data Management apiRoot；它是direct access endpoint，
   不是credential或sync資料。
2. PyMTLF以selected apiRoot建立standard `/nadrf-datamanagement/v1/data-store-records` GET。
3. `fetchUri`仍完整保存；只有當它normalize後與同一selected ADRF standard endpoint一致時才可直接採用。
4. 不因`fetchUri`指向MFAF-style callback就改送POST，也不跟隨它到任意host。

Current Go OpenAPI dependency仍沒有完整generated Nadrf service/types，且full corpus尚有其他NF API的unresolved
external refs。缺少generated type時可建立isolated compatibility wire package，但欄位與required/cardinality
必須直接來自上述exact attachments，不再把`FetchInstruction`或`TimeWindow`標示為schema gap：

- Go保留original JSON bytes，typed validation依exact Release 18 constraints。
- Python Pydantic wire models使用aliases及`extra="allow"`，避免future extension被drop。
- 不以untyped business map取代contract。

`resources/openapi/openapi/models/model_fetch_instruction.go`與`model_time_window.go`現在只作舊版
interoperability characterization，不再作這兩個schema的設計依據。

### 3.4 HTTP Status Matrix

依`TS29575_Nadrf_DataManagement.yaml`：

| Operation | Success | Declared error/redirect status |
|---|---|---|
| NRF NF discovery GET | `200` + mandatory-validity `SearchResult` | TS 29.510: `400, 403, 500, 3xx`; OpenAPI additionally defines applicable `ProblemDetails` responses |
| Retrieval subscription POST | `201` + JSON representation + mandatory `Location` | `400, 401, 403, 404, 411, 413, 415, 429, 500, 502, 503, default` |
| Retrieval notification POST | `204` | `307, 308, 400, 401, 403, 404, 411, 413, 415, 429, 500, 502, 503, default` |
| Data store records GET | `200` + `NadrfDataStoreRecord`, or `204` | `307, 308, 400, 401, 403, 404, 406, 429, 500, 502, 503, default` |
| Retrieval subscription DELETE | `204` | `307, 308, 400, 401, 403, 404, 429, 500, 502, 503, default` |

Mapping rules:

1. NRF `200`缺少mandatory `validityPeriod`/`nfInstances`或帶negative validity時，Go視為malformed peer
   success並對private caller回`502 ProblemDetails`；合法empty result仍回200並可cache。
2. Malformed JSON、mandatory/cardinality failure或semantically impossible callback回`400 ProblemDetails`。
3. Well-formed但`notifCorrId`沒有active route回404。
4. Unsupported media type回415；request too large回413。
5. MTLF backend configured但unavailable、sync未完成或無法保存notification時回503。
6. ADRF transport failure或ADRF成功status但missing/invalid required response contract，Go private response映射502。
7. ADRF well-formed declared error保留原status及`ProblemDetails`；private backend error body不可直接外露。
8. Direct GET由PyMTLF處理200、204、307及308；redirect有bounded limit，並維持本phaseplain HTTP boundary。
9. 429及5xx為retryable；明確4xx除redirect外視為permanent for current attempt。
10. DELETE 404表示peer resource已不存在，可視為cleanup terminal outcome；wire仍保留standard 404，不由Go
   無條件改寫204。

### 3.5 User Consent Boundary

TS 29.575 clauses 4.2.2.5.1及4.2.2.6.1指出，若資料以SUPI/GPSI為target，consumer在retrieval前需要向UDM
確認user consent。TS 23.288 clause 6.2.9也把SUPI、group與ML Model training納入依local policy/regulation
執行的consent scope。

目前workspace沒有完成這條UDM consent flow。本phase不得宣稱full consent compliance，也不得在Go/Python之間
發明一個boolean代替UDM程序。已確認的實驗環境前提為：

- 所有模擬UE視為已同意其資料用於analytics及ML model training；
- Phase 5不執行UDM consent query，也不新增`consent_mode`、enforcement toggle或其他config；
- 文件與completion record必須標示這是受控模擬實驗假設，不能宣稱production consent compliance；
- production group-to-SUPI consent resolution、consent change subscription及enforcement不屬於目前
  backend transition scope，若未來需要須另立設計。

這項實驗假設已確認，不再是implementation decision gate。

### 3.6 free5GC Structural Alignment

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
3. 以單一`trainingDataSource` enum取代`dataSourceAvailability`、`mtlfSourceSelection`、
   `mongodbAvailable`及`sourceSelection` negotiation欄位。
4. PyAnLF sync response回報current `trainingDataSource`；Go只在值改變時喚醒PyMTLF refresh並於其
   sync request傳遞。
5. PyMTLF遵循current source；endpoint/credential與local read health不回傳sync。

Current Go discovery private path已是`GET /internal/v1/nrf/nf-instances`，但validation、processor interface及
consumer request固定為SMF/`nsmf-event-exposure`。Go只cache generated client，不cache`SearchResult`；
PyAnLF目前自行依`validityPeriod`cache SMF candidates。Phase 5保留PyAnLF candidate cache並在Go下方新增
generic shared raw-result cache，讓PyAnLF與PyMTLF相同ADRF query可重用。

### 4.3 Mongo Contract Migration

PyAnLF已直接寫入configurable Mongo database/collection，sample default為：

```text
database: free5gc
collection: nwdaf_raw_notifications
```

目前每個document保存一筆完整raw notification：

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

PyAnLF的Mongo與ADRF write queues目前互相獨立、bounded且drop-oldest；selection為空時會向所有可用sink
dual-write，selection存在後只寫effective source。Phase 5明確取代這項bootstrap/selection behavior：
PyAnLF從啟動起即採ADRF-first，只有ADRF不可用時寫Mongo，不再dual-write。

Phase 5也取代correlation-based training query。PyAnLF對同一筆notification只建立一次
`NadrfDataStoreRecord`語意的`dataSub + dataNotif` envelope，ADRF path將它交給Go送出，Mongo path則連同
最小local query/provenance metadata直接保存：

```json
{
  "supi": "<denormalized smfDataSub.supi>",
  "source": "smf-or-upf",
  "receivedAt": "<DateTime>",
  "measurementTime": "<DateTime>",
  "dataSub": [
    {
      "smfDataSub": {}
    }
  ],
  "dataNotif": {}
}
```

`dataSub`及`dataNotif`保留完整standard payload；top-level `supi`與`measurementTime`只是Mongo query/index
metadata，不是另一套dataset contract。Target indexes為：

- `receivedAt`
- `(supi, measurementTime)`

第一版不建立schema version、migration、canonical hash或`dataSubKey`，也不相容讀取舊
`correlationId + notification` documents；實驗以乾淨collection開始。PyMTLF不寫Mongo或建立index。
Existing bounded、drop-oldest storage semantics不變。Top-level `source`只保留為local provenance與
operational inspection metadata；它不是3GPP欄位、不作training query contract，也不建立source-based
training index。UPF/SMF notification alternative以standard `dataNotif.upfEventNotifs`或
`dataNotif.smfEventNotifs`判斷。

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

- inspected `with-mlmodelmanagement` branch initially registers `NFType=ADRF` and the exact
  `nadrf-datamanagement`/`nadrf-mlmodelmanagement` services with NRF, which is sufficient for the short-lived
  discovery integration profile;
- retrieval `dataSub`只支援`smfDataSub.supi`；
- 要求`consTrigNotif = true`；
- create時snapshot符合SUPI、measurement `startTime`及ingestion cutoff的records；
- callback可batch fetch IDs，最後以`terminationReq = true`結束，即使零資料仍送termination；
- direct GET一次只接受一個fetch ID；
- DELETE對不存在resource也回204；
- callback delivery目前沒有retry。
- zero-data snapshot會送`terminationReq=true`且`fetchInstruct.fetchCorrIds=[]`。這不符合Release 18
  `FetchInstruction`的1..N cardinality；Go與PyMTLF只對這個terminal empty-list形狀作scoped compatibility
  acceptance，使workflow立即形成zero-data incomplete/FAILED，而不是等待watchdog。非terminal empty list仍
  以400拒絕，且本相容行為不得描述為normative contract。

The branch's heartbeat PATCH and failed-initial-register retry are not yet production-complete and its capability
flags are carried in non-standard `customInfo`. Phase 5 does not modify the ADRF repository or claim long-lived
registration reliability; the process harness must prove actual initial registration/discovery and distinguish
that result from the deferred ADRF lifecycle-hardening gap.

Production wire仍依Release 18 standard；以下只作V0 compatibility constraint：

- PyMTLF第一版每個GET送一個fetch ID。
- 每個distinct accepted SMF data subscription建立一個ADRF retrieval subscription。
- process-level test必須覆蓋zero-data termination及callback沒有retry時的接收可靠性。

`terminationReq`在Release 18 notification中是optional；規格沒有為所有ADRF implementation提供通用的
「這個historical subscription已送完全部records」marker。Current workspace ADRF會在最後一個callback設
`terminationReq = true`。本phase第一版以這個integration profile作完整性條件：沒有termination時，
watchdog只能判定本次資料集合無法證明完整，必須FAILED，不能因已收到部分records便READY或改查另一source。

---

## 5. Confirmed Architecture And Decisions

### 5.1 Already Confirmed

1. Python backend不是獨立標準NF；Go不使用`PyMTLF`命名。
2. Go現有private NRF discovery route泛化為generic typed proxy，不為ADRF建立另一個endpoint；Go擁有
   standard NRF transport與shared `SearchResult` cache，backend擁有criteria、candidate selection及endpoint。
3. PyAnLF與PyMTLF各自配置ADRF `nrf`或`configured` mode，各自保存endpoint；設定不一致造成不同ADRF或
   資料不存在是實驗配置錯誤，不建立endpoint agreement。
4. Standard ADRF subscribe/unsubscribe由Go送；PyMTLF建立完整standard-shaped request並指定target。
5. ADRF callback先到Go，再把完整notification交給MTLF backend。
6. PyMTLF直接向ADRF fetch；Go不取得dataset bytes。
7. PyMTLF直接read-only query Mongo；Go不提供query proxy。
8. Mongo保存PyAnLF建立的ADRF-aligned完整`dataSub + dataNotif` raw record及最小SUPI/time metadata，
   不使用legacy normalized Go records。
9. PyAnLF owns `trainingDataSource`，固定ADRF-first、Mongo fallback；正常ADRF可用時不dual-write。
10. Source state沿用unified health/sync；endpoint、credential、discovery cache不進sync。
11. Go state只在memory；Go restart視為實驗重跑，不做durable recovery。
12. Plain HTTP；本phase不加入internal或external security機制。
13. Phase 4 accuracy logic、WAPE report及degradation decision不因dataset retrieval而改寫。
14. 任一scope觸發model-level retrain後，所有active scopes的資料都納入同一training dataset候選集合；
    Phase 5只保留scope partition及trigger metadata，不決定scope-specific weighting或train/validation split。
15. 每個active scope都是required；current source中任一scope完全沒有valid record時job FAILED，不靜默使用
    缺少部分active scopes的dataset，也不跨source補資料。

### 5.2 ADRF Configuration And Selection

PyAnLF與PyMTLF各自使用與PyAnLF現有SMF collection相同的mode概念，但不共享值：

```yaml
adrf:
  mode: nrf
  configured_endpoint: ""
  discovery_timeout_seconds: 30
  request_timeout_seconds: 120
  retry_initial_backoff_seconds: 2
  retry_max_backoff_seconds: 30
```

Rules：

1. `mode`只允許`nrf`或`configured`。
2. `configured`要求一個normalized absolute ADRF apiRoot且完全不查NRF。
3. `nrf`呼叫Go generic discovery route，criteria為`ADRF`/`nadrf-datamanagement`，並從完整
   `SearchResult`選擇registered matching service。
4. PyAnLF/PyMTLF各自保存selected apiRoot；Go只cache raw SearchResult，不cache或sync backend selection。
5. NRF回多個candidate時，兩個backend都只使用mandatory registration/service state篩選可用candidate，
   將`nfServiceList`的map key作為缺少inline `serviceInstanceId`時的service identity fallback，再依
   `(nfInstanceId, serviceInstanceId, normalized apiRoot)`升冪排序、按apiRoot去重並選第一個。
   `priority`、`capacity`、`load`及`locality`都是optional selection inputs；第一版刻意不使用這些欄位，
   也不提供load balancing或multi-ADRF failover。此選擇追求簡單、可重現的實驗行為，不表示排序第一的ADRF
   具有較高服務品質。兩個backend仍可能因不同mode、不同query time或config而選到不同ADRF，該部署責任由
   實驗設定承擔。
6. Go原`configuration.adrf.url`不再是target source of truth；其storage/retrieval window等仍有active
   consumer的欄位分別搬到PyAnLF或PyMTLF，obsolete legacy-only fields由Phase 7移除。
7. Go cache使用fixed 256-entry internal bound；Cache TTL不由local config覆寫，只使用NRF mandatory
   `validityPeriod`。未有實際容量需求前不新增cache tuning config。

PyMTLF dataset config移除`preferred_source`，只保留retrieval mechanics與Mongo direct-read設定：

```yaml
dataset:
  retrieval_window_seconds: 1800
  watchdog_timeout_seconds: 120
  fetch_timeout_seconds: 120
  max_redirects: 3
  max_retry_attempts: 3
  retry_initial_backoff_seconds: 1
  retry_max_backoff_seconds: 30
  max_concurrent_jobs: 2
  max_records_per_job: 100000
  mongodb:
    url: mongodb://127.0.0.1:27017
    database: free5gc
    collection: nwdaf_raw_notifications
    connect_timeout_ms: 5000
    read_timeout_ms: 30000
```

`fetchBatchSize`不成為general config；因local ADRF V0限制，direct GET adapter固定一次一個ID，待peer支援
standard multi-ID後才另立decision。

### 5.3 Training Datasource State And Recovery

第一版固定：

```text
ADRF candidate resolves and is not marked failed -> trainingDataSource = adrf
ADRF unresolved/failed + Mongo write path is available -> trainingDataSource = mongodb
neither sink usable -> trainingDataSource = unavailable
```

Rules：

1. PyAnLF從bootstrap即執行上述政策；ADRF可用時不寫Mongo，沒有selection-empty dual-write period。
2. NRF一開始找不到ADRF時，PyAnLF使用Mongo並定時再呼叫generic discovery。Go cache合法empty result直到
   `validityPeriod`到期，之後第一個poll才重新查NRF。
3. NRF已回endpoint但target不可連時，PyAnLF保留並bounded-backoff重試同一endpoint；不因每次transport error
   立即重查NRF。validity到期後可重新discovery處理replacement。
4. Failed ADRF record當下嘗試寫Mongo；成功後才將source切到Mongo。Configured mode或NRF成功發現endpoint的
   bootstrap可先把source設為ADRF，但一旦標記failed，恢復必須有一次真實standard write
   success，該筆及後續新資料寫ADRF，才將source切回ADRF。
5. PyAnLF在sync response回報current source；Go轉送給PyMTLF。Go尚無值時使用`unavailable`。
6. PyMTLF job開始時snapshot current source；`adrf`走ADRF path，`mongodb`走Mongo path，`unavailable`
   bounded wait/fail。Job不自行選alternate，也不在執行中混合兩個source。
7. PyMTLF的ADRF retrieval收到合法204或沒有usable matching record的valid result，是data outcome而不是
   ADRF unavailability，不能因此fallback Mongo；這可暴露兩backend選到不同ADRF等實驗配置錯誤。

source切換不保證歷史coverage。若ADRF在`T0-T1`不可用，該段只存在Mongo；`T1`恢復後PyAnLF只寫ADRF，
PyMTLF也改查ADRF。跨越`T0-T1`的ADRF query缺少該段資料視為accepted loss。本phase不backfill、不查詢
source history、不merge Mongo+ADRF，也不因缺段回頭查Mongo。

### 5.4 PyAnLF Storage Routing And Cutover

PyAnLF既有private storage route保持standard-shaped body與POST語意：

```http
POST /internal/v1/adrf-data-management/data-store-records
Target-Api-Root: <PyAnLF selected ADRF apiRoot>
Content-Type: application/json

NadrfDataStoreRecord
```

Go不再從`configuration.adrf.url`選target。它驗證`Target-Api-Root`、使用target-keyed shared transport client，
送出TS 29.575 clause 4.2.2.2的standard POST，並把201/Location/representation或declared peer error送回
PyAnLF。

Storage result分類：

1. `201`且mandatory response contract有效：ADRF write success。
2. Transport/timeout、`429`或`5xx`：ADRF current availability failure；同一raw notification嘗試Mongo，
   Mongo成功後切`trainingDataSource=mongodb`。
3. Redirect依standard client bounded處理；成功後保存normalized final target/route information。
4. 其他well-formed `4xx`表示current record/request被拒絕，不證明ADRF endpoint unavailable；記錄並丟棄該
   record，不切整體source。
5. ADRF可能已接收但response遺失時，Mongo fallback可能造成跨store duplicate；因本phase不merge source，
   接受此best-effort ambiguity，不建立distributed idempotency protocol。

Mongo active期間，每筆正常資料只寫Mongo。PyAnLF按backoff schedule把「下一筆到達且可形成合法
`NadrfDataStoreRecord`的notification」作為real ADRF recovery attempt；沒有新資料時不送synthetic record或
依賴非標準health endpoint。成功的probe record開始新的ADRF period並切source。source change與Go→PyMTLF
refresh是eventually consistent，不提供跨process atomic cutover；切換邊界附近缺資料屬本phase已接受限制。

### 5.5 Dataset Completeness

Confirmed first-version rule：

- Triggering scope與intent建立當下的所有active scopes都是required。
- Dataset按scope分區，不將不同group混成無標記record list。
- 每個required scope至少一筆valid raw record，整個snapshot才是READY。
- Triggering scope只保留為retrain cause及observability metadata，不取得特殊sample role或weight。
- 所有active scopes的records都進入Phase 6可使用的training dataset候選集合；沒有scope被預先限制為
  validation-only。
- Phase 5不進行ML train/validation split。Phase 6可再從所有scope的資料分別切出training及held-out samples，
  但不能把某個healthy scope整體排除在training dataset之外。
- 重複record以source-native stable identity優先；缺少identity時以canonical `dataSub + dataNotif` payload
  及measurement time計算local dedup key。
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
5. 從每個accepted `NsmfEventExposure`取出完整`smfDataSub`；ADRF與Mongo path共用這份query input及同一
   fixed `timePeriod`。
6. ADRF path為每個distinct accepted SMF resource建立retrieval subscription；local ADRF V0實際以
   `smfDataSub.supi + timePeriod`建立snapshot。
7. Mongo path從相同`smfDataSub`解析SUPI，以所有required SUPIs及相同window執行一個bounded query；
   不使用collection `correlationId`作training-data lookup。
8. 一個collection resource可被多個analytics scopes refcount共用；record在dataset中可refer到多個scope，
   但physical raw payload只保存一次。

Resolution ambiguity不得用「取第一個」掩蓋。零match、同一scope出現conflicting collection profile或accepted
subscription無法依Release 18 compatibility model解析，皆形成explicit incomplete reason並使current job失敗。

### 6.3 Sync Changes

Go送給MTLF backend的sync增加既有typed sections的實際內容，不新增另一條API：

- active Events Subscription accepted representation
- SMF resource correlation、target apiRoot、resource Location、refcounted NWDAF subscription IDs、
  pending cleanup及accepted standard subscription representation
- existing `containingNwdaf.apiBaseUri`，供PyMTLF建立Go-owned ADRF callback URL
- `trainingDataSource`，值為`adrf`、`mongodb`或`unavailable`

PyAnLF sync response回報`trainingDataSource`；Go保存並只在值真正改變時喚醒PyMTLF refresh，避免sync update
loop。PyMTLF不回報source preference/selection，也不接收ADRF endpoint或Mongo credential。MTLF backend在
完整sync成功前不能consume retrain intent；backend reconnect後以最新snapshot解析新job。

Exact directional shape：

PyAnLF → Go `BackendSyncResponse`：

```json
{
  "trainingDataSource": "adrf"
}
```

Go → PyMTLF `BackendSyncRequest`：

```json
{
  "trainingDataSource": "adrf"
}
```

The field uses the enum `adrf | mongodb | unavailable`. It is optional in the shared wire model only because
the opposite backend direction has no producer/consumer: Go omits it from AnLF sync requests, PyMTLF omits it
from sync responses, and Go treats an unexpected PyMTLF value as a malformed role-specific sync response.
`unavailable` is the explicit bootstrap value, not an omitted/empty-string selection.

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
- snapshotted `trainingDataSource`
- per-scope resource IDs及record counts
- raw record dedup index
- ADRF subscription/correlation/termination/fetch state
- failure/source/cleanup outcome

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
不在每個scope或retry時重新計算`now`，避免coverage drift。

Mongo writer保留PyAnLF現有measurement-time推導：一個UPF callback取所有notification items中最早的
`startTime`，item缺少`startTime`時使用其`timeStamp`。第一版把同一callback視為一個時間接近的raw batch，
不新增per-item min/max range schema；這是local query approximation，不能宣稱與任意ADRF implementation的
internal time matching完全相同。

### 7.4 Concurrency

- 同一model延續Phase 4 one-in-flight guarantee。
- 不同model可並行，但有configurable small global worker limit，避免同時建立無界ADRF subscriptions/DB cursors。
- Callback handler只保存notification及enqueue fetch work，不在HTTP request內等待全部GET。
- Fetch IDs全job dedup；同一callback replay不會重複append record。
- `terminationReq`是per-subscription terminal flag，不用callback數量猜測完成。
- ADRF job只有在所有subscription termination已收到且所有accepted fetch IDs已處理時才能進入completeness
  evaluation；terminal failure/timeout只能FAILED，不能改查Mongo或以partial records進入READY。

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
3. PyMTLF以private standard-shaped POST及selected ADRF `Target-Api-Root`交給Go。
4. Go驗證body、target apiRoot與callback URI確實是自己的configured public callback，再呼叫selected ADRF
   `POST /nadrf-datamanagement/v1/data-retrieval-subscriptions`。
5. Go只在ADRF回201、valid JSON representation及mandatory Location時視為成功。
6. Go把201、Location與完整representation交回PyMTLF，並保存process-local peer route mirror。

ADRF endpoint config搬到PyMTLF；NWDAF external callback base仍由Go擁有並透過containing identity sync。
Private API path使用`mtlfBackend`/`adrf-data-management`命名，不出現`pymtlf`。

Internal route matrix：

| Caller | Method and path | Success |
|---|---|---|
| MTLF backend -> Go | `POST /internal/v1/adrf-data-management/data-retrieval-subscriptions` + `Target-Api-Root` | `201` + JSON + `Location` |
| MTLF backend -> Go | `DELETE /internal/v1/adrf-data-management/data-retrieval-subscriptions/{subscriptionId}` | `204`；Go mirror保存original target/Location |
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

1. 使用PyMTLF自己selected ADRF Data Management apiRoot建立standard data-store-records URL；received
   `fetchUri`若normalize後是同一endpoint才可直接採用。
2. 以標準query name `fetch-correlation-ids`正確URL encode。
3. 第一版一次一個ID，以符合local ADRF V0；domain interface保留未來multi-ID能力。
4. 200時驗證完整`NadrfDataStoreRecord`，只接受和本job `dataSub`/scope相符的raw data。
5. 204標記該fetch ID completed-with-no-data。
6. optional `expiry`已過期時不fetch，形成explicit incomplete reason。
7. 307/308 bounded follow；429/5xx及network timeout capped retry；permanent 4xx停止該ID。
8. 每次有效callback/成功fetch progress reset 120秒default watchdog。

Selected ADRF apiRoot與可採用的`fetchUri`都必須是absolute HTTP(S) URI且不得帶userinfo、query或fragment。
Redirect第一版只允許selected ADRF same-origin，避免callback把backend導向任意新host。這是HTTP client
validation，不導入TLS、OAuth或key exchange。TS 29.576的MFAF Fetch POST不在這條direct ADRF path中。

PyMTLF不得透過Go抓資料，也不得讓Go做record normalization、chunk completion或dataset merge。

### 8.4 Completion And Cleanup

- 收到某subscription的`terminationReq = true`後不再期待新instruction，但已接受的fetch IDs仍要drain。
- 全部required subscriptions terminal且fetch queue drained後評估scope completeness。
- Job成功、失敗或shutdown皆發出cleanup intent。
- PyMTLF以Go private standard-shaped DELETE要求刪除peer Location。
- Go依保存的exact target/Location呼叫ADRF，不重新discovery或自行重組錯誤apiRoot。
- Cleanup retry有上限並保留observability；cleanup failure不把已READY dataset改成資料內容失敗，但job需記錄
  leaked-resource risk。

---

## 9. Mongo Path

### 9.1 Ownership And Dependency

PyMTLF新增`pymongo` runtime dependency與read-only repository。Connection settings屬PyMTLF local config，
sync只交換current `trainingDataSource`，不交換credential/key。

Repository startup/refresh probe：

- parse config
- connect with bounded server-selection/connect/read timeout
- `ping`
- verify configured database/collection可read
- 不create index、不write probe document

### 9.2 Query

PyMTLF Mongo repository的domain input和ADRF adapter相同：

```text
accepted smfDataSub + fixed TimeWindow
```

第一版假設同一實驗期間SMF subscription semantic profile保持一致。Mongo adapter從每個required
`smfDataSub`解析SUPI，將distinct SUPIs合併成一個bounded query：

```javascript
{
  "supi": {"$in": ["imsi-...", "imsi-..."]},
  "measurementTime": {
    "$gte": "<fixed startTime>",
    "$lte": "<fixed stopTime>"
  },
  "dataNotif.upfEventNotifs.0": {
    "$exists": true
  }
}
```

完整`smfDataSub`是兩條source path的共同查詢描述；Mongo不對整份embedded JSON做exact equality，也不建立
canonical hash。這和workspace ADRF V0目前從完整request取出SUPI再依時間建立snapshot的能力一致。
Training query不使用top-level local `source`欄位；UPF record selection以standard
`dataNotif.upfEventNotifs` alternative為準。

若一個group展開為三個SUPI，ADRF path在current V0 profile建立三個retrieval subscriptions；Mongo path可用
同一個`$in` query取得三個SUPI的候選records，再依scope-to-dataSub/SUPI mapping分區。Query結果：

1. 驗證`supi`、`measurementTime`、`dataSub`及`dataNotif`。
2. `dataSub`必須包含本profile唯一一個可解析的`smfDataSub`，且top-level SUPI必須與
   `smfDataSub.supi`一致；不對完整accepted subscription JSON做exact equality。
3. `dataNotif`必須只選擇non-empty `upfEventNotifs` standard alternative；`smfEventNotifs`可被Mongo保存，
   但不進入本phase traffic-training dataset。
4. 確認record SUPI屬於本job要求的accepted subscriptions，再依scope mapping分區。
5. 保留完整`dataSub + dataNotif`及最小provenance，不把它轉成training feature。ADRF與Mongo只統一外層
   `DatasetRecord` provenance/identity/scope envelope，不改寫內層standard payload。
6. Deterministic sort與dedup。
7. Cursor/record數有safety limit；超限形成explicit failure，不靜默截斷後宣稱完整。

Connection/query failure使current job失敗，不跨source查ADRF。單筆malformed document只skip並計數；
required scope因此為空時整個dataset不READY。

### 9.3 Selected Source Failure

PyAnLF回報`trainingDataSource=mongodb`只證明storage side已切到Mongo，不保證PyMTLF credential、network或
collection read一定成功。PyMTLF仍在job開始前做bounded direct-read probe；失敗時記錄selected-source
unavailable。Connection/query依dataset retry config完成bounded retry後仍失敗，current job進入FAILED；
cleanup完成後解除model in-flight，等待未來accuracy report重新觸發，不可自行改查ADRF，因為PyAnLF在該
期間沒有寫ADRF。

同理，`trainingDataSource=adrf`時PyMTLF自己的configured/discovered endpoint仍可能不可連，或和PyAnLF
選到不同ADRF。這些情況是runtime/experiment configuration failure，不改變PyAnLF current source，也不觸發
Mongo fallback。只有PyAnLF的實際storage path負責切換environment `trainingDataSource`。

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
| Go normalized Mongo traffic records | PyAnLF ADRF-aligned `dataSub + dataNotif` raw documents |
| Dataset records混成單一group label | scope-partitioned raw snapshot |
| Go fixed ADRF URL決定所有storage/retrieval target | PyAnLF/PyMTLF各自configured或經generic NRF discovery選target |
| Go只cache NRF client、PyAnLF單獨cache SMF result | Go generic shared `SearchResult` cache + backend-local selected candidate |
| Preferred/effective source negotiation | PyAnLF-owned fixed ADRF-first/Mongo fallback及single `trainingDataSource` |
| Selection空時dual-write | bootstrap即single-active sink；ADRF可用只寫ADRF，失敗才寫Mongo |
| Partial successful fetch可繼續legacy training | required scope completeness or FAILED |
| Watchdog close queue後仍提交legacy training | timeout永遠FAILED，不以partial records READY或跨source補資料 |
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

### Slice A: Generic NRF Discovery And ADRF Resolution

1. 將Go existing SMF-only handler/processor/consumer泛化為typed generic NF discovery。
2. 建立canonical query key、mandatory validity validation、empty-result caching、expiry cleanup及same-key
   concurrent request coalescing與bounded LRU eviction。
3. 保留existing PyAnLF SMF request/response與candidate behavior regression。
4. 在PyAnLF/PyMTLF新增獨立ADRF `nrf`/`configured` config、validation、candidate parsing及local selection cache。
5. 將Go fixed ADRF endpoint ownership移除，ADRF private standard-shaped request由backend提供
   `Target-Api-Root`。

Acceptance：

- identical PyAnLF/PyMTLF ADRF query在validity內只造成一次NRF call；
- valid empty result被cache、到期後重查；zero validity不cache；malformed validity回502；
- late cache hit只回remaining validity且body/header一致，不延長NRF expiry；
- explicit zero與missing mandatory field可區分；capacity eviction不改變caller response；
- existing SMF discovery contract與per-subscription binding不變；
- configured mode不查NRF，兩backend selection不進Go sync。

### Slice B: Training Datasource Cutover And Sync

1. PyAnLF storage path改為ADRF-first single sink，failed ADRF record改寫Mongo。
2. 實作no-result rediscovery與known-endpoint capped retry/recovery。
3. 以`trainingDataSource`取代availability/preference/selection sync fields。
4. Go在PyAnLF source change時refresh PyMTLF；PyMTLF snapshot並遵循source。
5. 新增source transition、restart與no-refresh-loop tests。

Acceptance：

- ADRF success不寫Mongo；
- ADRF write failure後同筆資料寫Mongo並切`mongodb`；
- ADRF真實write恢復後只影響新資料並切回`adrf`；
- PyMTLF不自行跨source fallback；
- switch前historical gap明確保留，不backfill/merge。

### Slice C: Intent Handoff And Scope Resolution

1. 將RetrainIntent接到single-consumer coordinator。
2. 建立structured scope snapshot，不改policy algorithm。
3. 實作scope -> Events -> SMF resource解析。
4. 建立fixed TimeWindow及scope-partitioned job。
5. 覆蓋shared resource、不同SMF相同sub ID、pending cleanup、zero/ambiguous match。

Acceptance：

- group A trigger時job包含A及同model active group B；
- peer identity使用target + Location；
- 沒有從canonical JSON key反向猜資料resource。

### Slice D: Standard ADRF Control Plane Through Go

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

### Slice E: Callback And Direct Fetch

1. Go callback改為完整notification validation/forwarding。
2. PyMTLF先註冊correlation再create，消除immediate callback race。
3. PyMTLF保存fetch instruction、dedup、expiry、termination及watchdog state；對
   `consTrigNotif = true`卻收到direct notification alternative的情況記錄profile mismatch並使job失敗。
4. PyMTLF direct GET 200/204/redirect/retry。
5. Completion與cleanup convergence。

Acceptance：

- Go只有backend接受後回204；
- zero-data termination可正常完成為incomplete/FAILED，而非timeout；
- batch/replay/out-of-order callback不重複資料；
- dataset bytes不出現在Go API或state。

### Slice F: Mongo Direct Read

1. PyMTLF read-only Mongo repository。
2. PyAnLF將Mongo writer改成保存ADRF-aligned`dataSub + dataNotif` envelope與top-level SUPI/time metadata，
   並建立`(supi, measurementTime)` index。
3. PyMTLF以和ADRF相同的`smfDataSub + TimeWindow` domain input建立SUPI/window query、validation、sort、
   dedup與scope partition。
4. Mongo training query以standard `dataNotif.upfEventNotifs` alternative選取UPF records，不依賴top-level
   local `source` metadata。
5. Selected-source connectivity、coverage gap與malformed-record outcomes。

Acceptance：

- direct Mongo path可生成和ADRF path相同domain shape的raw DatasetSnapshot；
- PyMTLF不建立index、不write；PyAnLF只實作本phase核准的local schema/index replacement；
- training-data lookup不使用collection correlation ID；
- training-data lookup不使用top-level local `source`，且SMF notification不被誤當traffic-training sample；
- incomplete Mongo result不被誤標READY，且不回查ADRF。

### Slice G: Process-Level Verification And Documentation

1. 在`nwdaf-resources/tests/mtlf_dataset_retrieval/`建立跨process harness。
2. 啟動Go NWDAF、PyAnLF、PyMTLF、free5GC NRF、workspace ADRF及MongoDB。
3. 驗證ADRF initial registration、兩backend相同query共用cache、empty-result validity expiry及configured bypass。
4. 透過PyAnLF callback/storage path寫入真實raw notifications。
5. 觸發Phase 4 policy intent並分別驗證ADRF、ADRF-unavailable→Mongo cutover與ADRF recovery。
6. 驗證Go/backend restart boundary、callback 204 timing與cleanup。
7. 更新各repo API/config/runbook與parent plan。

Harness不放在`NWDAF/`、`PyAnLF/`或`PyMTLF/`內；單一repo只保留unit/contract tests。

---

## 13. Verification Matrix

### 13.1 PyMTLF

- ADRF configured/nrf config、invalid mode/URI/timeout validation及candidate parsing。
- `trainingDataSource` projection、unavailable gating及source snapshot。
- Intent single consumption與one-model-in-flight。
- Scope resolution：multi-group、shared resource、multi-SMF collision、pending cleanup、missing accepted body。
- Fixed TimeWindow、`smfDataSub` input、inclusive SUPI/window query及standard
  `dataNotif.upfEventNotifs` selection。
- Raw record validation/dedup/scope completeness。
- ADRF create race、callback replay、batch IDs、expiry、200/204、termination、watchdog。
- `consTrigNotif = true`只以`fetchInstruct`進入正常fetch path；其他合法schema alternative被完整保存並形成
  可觀察的profile mismatch。
- 429/5xx retry、4xx terminal、redirect bound。
- Selected ADRF/Mongo failure不跨source，historical gap不觸發merge/fallback。
- Shutdown cleanup及READY/FAILED policy handoff。
- `uv run pytest -q`
- `uv run ruff check .`
- `uv run ruff format --check .`

### 13.2 NWDAF

- Generic discovery query forwarding、requester identity injection及SMF/ADRF response round trip。
- Canonical-key cache、validity expiry、empty result、zero/malformed validity及concurrent miss coalescing。
- Late cache hit remaining TTL與`Cache-Control: max-age`一致，不重新開始original validity。
- Unknown Release 18 SearchResult/NFProfile fields及`ignoredQueryParams`經fresh/cache-hit response保留。
- ADRF POST/DELETE exact method/path/query/header/body。
- 201 representation與mandatory Location。
- Full callback round trip及unknown Release 18 field preservation。
- 204 only after backend acceptance。
- 400/413/415/429/5xx/502/503 mapping。
- Backend unavailable/reconnect/process UUID change。
- MTLF sync收到Events/SMF snapshots且AnLF sync沒有regression。
- `trainingDataSource` change只refresh受影響backend且沒有loop。
- No production Go direct fetch/dataset caller。
- `make test`
- `make lint`
- `make build`

### 13.3 PyAnLF

- ADRF configured/nrf resolution、no-result rediscovery及known-endpoint retry。
- ADRF success single-write、failed record Mongo fallback、recovery cutover及`trainingDataSource`回報。
- ADRF-aligned Mongo `dataSub + dataNotif` contract、SUPI/time metadata、standard notification
  alternative selection與indexes regression。
- Source sync refresh不形成loop。
- Existing full lint/test suite。

除非本phase真的修改PyAnLF，否則不為了測試方便新增production API。

### 13.4 Process-Level Versus Environment-Level

| Level | Required proof |
|---|---|
| Unit/contract | fake peer、deterministic clock、status/body/state transition |
| Workspace process-level | real Go/Python processes、real free5GC NRF registration/compatibility check、real Mongo、real workspace ADRF、configured-mode callback/direct fetch |
| Full 5GC environment | complete NRF registration lifecycle plus SMF/UPF-generated data；本phase可記錄但不是completion gate |

不得以mock callback測試宣稱ADRF e2e完成，也不得把workspace process-level稱為完整5GC integration。

---

## 14. Confirmed Final Decisions Before Implementation

Dataset composition、strict active-scope completeness、generic NRF cache、independent ADRF config、
ADRF-first/Mongo fallback、single-active storage、source-cutover historical loss及Mongo
`smfDataSub + timePeriod` query semantics均已確認，不再是decision gate。最後兩項也已確認：

1. **Simulated user consent**：所有模擬UE視為已同意；不新增consent config或UDM integration，且不宣稱
   production compliance。
2. **READY handoff**：Phase 5完成dataset後保持canonical model identity的`retrain-in-flight = true`，
   直到Phase 6 training/model update取得terminal success或failure才release；因此同一model即使有多個scope
   再次degrade，也不會在既有workflow完成前建立重複retrieval job。

目前沒有剩餘high-level implementation decision gate。

這些decision之外，implementation不得自行改變Phase 4 accuracy policy、本phase核准的PyAnLF Mongo
`dataSub + dataNotif` schema、既有`trainingDataSource` agreement或Go/Python ownership。

---

## 15. Completion Criteria

本phase只有同時滿足以下條件才算完成：

1. Existing private NRF discovery route可typed地服務SMF與ADRF，並保持SMF behavior regression。
2. Go依canonical query與mandatory `validityPeriod`共用完整`SearchResult`，包含empty result、expiry及
   concurrent miss convergence。
3. PyAnLF/PyMTLF各自的ADRF configured/nrf mode可獨立工作，endpoint不進sync。
4. PyAnLF ADRF success只寫ADRF；ADRF unavailable切Mongo並回報`trainingDataSource`；recovery只切未來資料。
5. Phase 4 retrain intent能自動且只被一個retrieval job消費。
6. Same model/different group形成一個model-level job內的獨立scope partitions。
7. ADRF subscribe/callback/unsubscribe符合Release 18 method/body/status/Location。
8. Go收到完整callback並在PyMTLF接受後才回204。
9. PyMTLF以`fetchCorrIds`直接執行ADRF RetrievalRequest GET，Go完全不接觸dataset bytes。
10. PyMTLF可用和ADRF相同的`smfDataSub + timePeriod`語意read-only query PyAnLF Mongo collection；
    source adapter實際使用SUPI/time與`dataNotif.upfEventNotifs`，且不依賴correlation ID或top-level
    local `source`。
11. ADRF-first/Mongo cutover、endpoint recovery與incomplete history behavior可觀察、可測且不merge source。
12. Every required scope有資料才產生READY DatasetSnapshot。
13. Failure、timeout、restart與shutdown有bounded cleanup/retry，不留下失控worker。
14. Workspace real-process path通過：ADRF向real NRF註冊，pinned R17 NRF對ADRF discovery的400限制被明確
    驗證，configured-mode ADRF/Mongo storage與retrieval通過；不得以mock或NF Management listing fallback
    宣稱Release 18 NRF-mode process discovery已完成。
15. NWDAF、PyAnLF、PyMTLF各自full lint/test/build通過。
16. 文件清楚區分normative spec、generated/local compatibility、workspace ADRF behavior及deferred limitation。

---

## 16. Implementation Record

Implemented on 2026-07-24:

- Go unified sync projects the accepted Events/SMF resource inventory to MTLF and carries the single
  `trainingDataSource` state from PyAnLF to PyMTLF.
- The private NRF discovery path accepts SMF Event Exposure and ADRF Data Management profiles. Go owns a
  query-keyed validity cache; each backend still owns deterministic candidate selection.
- PyAnLF owns independent ADRF resolution, ADRF-first single-sink routing, same-record Mongo fallback,
  future-record recovery cutover, and the ADRF-aligned Mongo document/index contract.
- Go accepts backend-selected `Target-Api-Root`, forwards ADRF storage and retrieval POST/DELETE, stores
  process-local peer Locations, and forwards complete retrieval callbacks synchronously before returning 204.
- PyMTLF consumes structured retrain intents, resolves every active scope through synced Events/SMF resources,
  snapshots one window/source, performs direct ADRF fetch or read-only Mongo retrieval, and publishes READY only
  when every required scope has a valid UPF record.
- READY retains the Phase 4 model-level in-flight flag for the local trainer handoff; FAILED releases it after
  bounded retrieval and cleanup.
- The consolidated review on 2026-07-25 corrected source cutover timing, capped ADRF recovery, selected-source
  retries, malformed Mongo record isolation, common dataset bounds, standard callback validation, idempotent
  cleanup/shutdown, Go ADRF transport reuse, same-origin peer Location validation, and deterministic
  identity-first ADRF candidate ordering.

Verified locally:

- `NWDAF`: `go test ./...`, `make lint`, and `make build`.
- `PyAnLF`: `uv run ruff check .` and `uv run pytest -q` (`241 passed, 1 skipped`).
- `PyMTLF`: `uv run ruff check .`, changed-file `uv run ruff format --check`, and `uv run pytest -q`
  (`77 passed`, one dependency deprecation warning), including config/discovery, scope resolution, partition,
  dedup, completeness, fetch URI, retry, malformed Mongo record, callback bound, validation, and cleanup tests.
- `nwdaf-resources/tests/mtlf_dataset_retrieval/run.py`: real Mongo, free5GC NRF, workspace ADRF, NWDAF,
  PyAnLF, and PyMTLF process run passed. It verified ADRF registration, the pinned NRF's explicit ADRF discovery
  rejection, configured-mode ADRF storage/retrieval, Mongo cutover/direct retrieval, and ADRF future-write
  recovery.
- `nwdaf-resources/tests/mtlf_model_monitor`: remains the companion process-restart/sync proof from Phase 4;
  Phase 5 does not duplicate that harness.

The full PyMTLF repository-wide formatter check still identifies ten pre-existing files outside this phase.
They were not mechanically reformatted because the development policy forbids mixing unrelated refactors into
feature work; every Python file changed by Phase 5 passes `ruff format --check`.

The PyAnLF repository-wide formatter baseline also predates this phase. All newly added or substantively edited
Phase 5 implementation and test files pass `ruff format --check`; four previously modified integration/test files
would require broad mechanical rewrites beyond the source-field removals made here, so they remain lint-clean but
are intentionally not reformatted in this change.
