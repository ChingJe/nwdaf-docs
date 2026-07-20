# Phase 3 Analytics Subscription, Data Collection And Raw Storage

Date: 2026-07-21

Status: Revised detailed plan after team architecture review; Phase 3 contract decisions confirmed

Parent plan:

- `MTLF Backend Transition Plan.md`

Previous phase:

- `Phase 2 Backend Connectivity And Standard Contract Foundation.md`

Related historical record:

- `../anlf-backend-transition/Phase 3 Analytics Runtime Migration.md`

---

## 1. Purpose

Phase 3完成`Nnwdaf_EventsSubscription`、analytics data collection及raw notification storage的第一個
端到端vertical slice，並修正Phase 2不再適用的backend sync假設。

本phase的目標ownership：

1. Go保留唯一NWDAF standard SBI、NRF/SMF/UPF/ADRF consumer、external resource routing及standard
   HTTP response。
2. AnLF backend擁有Events Subscription domain state、analytics runtime、資料需求、SMF candidate
   selection、fan-out policy、notification processing及raw storage execution。
3. NRF discovery與SMF/UPF/ADRF control request仍由Go發出，但Go/backend boundary模仿對應標準
   method、query、body、response與`ProblemDetails`。
4. SMF/UPF notification直接送到AnLF backend，不再經Go轉成custom observation。
5. AnLF backend直接寫Mongo；需要ADRF storage時將標準`NadrfDataStoreRecord`交給Go。
6. AnLF與MTLF backend都在readiness後進行sync，取代MTLF-only storage handshake。
7. AnLF backend用UUIDv4產生Events Subscription resource ID；Go沿用同一ID並保留
   process-local routing/sync snapshot。
8. SMF configured endpoint、candidate selection、collection sharing/refcount與cleanup intent都由AnLF backend擁有。
9. Go作為central sync coordinator，持續poll兩個backend，並以process incarnation UUID偵測快速重啟。

第一版仍只驗證現有`UE_COMMUNICATION` periodic analytics，不在ownership cutover期間擴張新的analytics
event或reporting algorithm。

---

## 2. Planning Basis

### 2.1 Standard Sources

本計畫使用workspace-local Release 18 corpus：

- `nwdaf-docs/specs/openapi/TS29520_Nnwdaf_EventsSubscription.yaml`
  - API attachment version `1.3.3`
- `nwdaf-docs/specs/TS 29.520/4 Services offered by the NWDAF/4.2 Nnwdaf_EventsSubscription Service/`
  - `4.2.2 Service Operations/4.2.2.2 Nnwdaf_EventsSubscription_Subscribe service operation/4.2.2.2.2 Subscription for event notifications.md`
  - `4.2.2 Service Operations/4.2.2.2 Nnwdaf_EventsSubscription_Subscribe service operation/4.2.2.2.3 Update subscription for event notifications.md`
  - `4.2.2 Service Operations/4.2.2.3 Nnwdaf_EventsSubscription_Unsubscribe service operation.md`
  - `4.2.2 Service Operations/4.2.2.4 Nnwdaf_EventsSubscription_Notify service operation.md`
- `nwdaf-docs/specs/openapi/TS29510_Nnrf_NFDiscovery.yaml`
  - API attachment version `1.3.4`
- `nwdaf-docs/specs/TS 29.510/5 Services Offered by the NRF/5.3 Nnrf_NFDiscovery Service/`
  - `5.3.2 Service Operations/5.3.2.2 NFDiscover.md`
- `nwdaf-docs/specs/openapi/TS29508_Nsmf_EventExposure.yaml`
- `nwdaf-docs/specs/TS 29.508/4 Session Management Event Exposure Service/`
  - `4.2 Service Operations/4.2.2 Nsmf_EventExposure_Notify Service Operation.md`
  - `4.2 Service Operations/4.2.3 Nsmf_EventExposure_Subscribe Service Operation.md`
  - `4.2 Service Operations/4.2.4 Nsmf_EventExposure_UnSubscribe Service Operation.md`
- `nwdaf-docs/specs/openapi/TS29564_Nupf_EventExposure.yaml`
- `nwdaf-docs/specs/TS 29.564/5 Services offered by the UPF/5.2 Nupf_EventExposure Service.md`
- `nwdaf-docs/specs/TS 29.564/6 API Definitions/6.1 Nupf_EventExposure Service API/`
- `nwdaf-docs/specs/openapi/TS29575_Nadrf_DataManagement.yaml`
- `nwdaf-docs/specs/TS 29.575/4 Services offered by the ADRF/4.2 Nadrf_DataManagement Service/`
  - `4.2.2 Service Operations/4.2.2.2 Nadrf_DataManagement_StorageRequest service operation.md`
- `nwdaf-docs/specs/TS 23.288/6 Procedures to Support Network Data Analytics/6.2 Procedures for Data Collection/`
  - `6.2.2 Data Collection from NFs/6.2.2.2 Procedure for Data Collection from NFs.md`

### 2.2 Current Implementation Sources

Go現況：

- `NWDAF/internal/sbi/api_eventssubscription.go`
- `NWDAF/internal/sbi/processor/eventssubscription.go`
- `NWDAF/internal/sbi/processor/data_collection.go`
- `NWDAF/internal/sbi/consumer/consumer.go`
- `NWDAF/internal/sbi/consumer/smf_service.go`
- `NWDAF/internal/context/traffic_data.go`
- `NWDAF/internal/anlf/client/`
- `NWDAF/internal/anlf/coordinator/`
- `NWDAF/internal/sbi/api_collector.go`
- `NWDAF/internal/context/db_models.go`
- `NWDAF/internal/context/db_query.go`
- `NWDAF/internal/sbi/processor/upf_notify.go`
- `NWDAF/pkg/factory/config.go`

PyAnLF現況：

- `PyAnLF/src/py_anlf/models.py`
- `PyAnLF/src/py_anlf/sbi/routers/analytics.py`
- `PyAnLF/src/py_anlf/core/runtime_manager.py`
- `PyAnLF/src/py_anlf/core/reporting.py`
- `PyAnLF/src/py_anlf/core/analytics_runtime.py`
- `PyAnLF/src/py_anlf/core/observation_store.py`

PyMTLF現況：

- `PyMTLF/src/py_mtlf/api/health.py`
- `PyMTLF/src/py_mtlf/api/data_source.py`
- `PyMTLF/src/py_mtlf/config.py`

### 2.3 free5GC Exemplar Evidence

primary exemplar：

- BSF subscription CRUD
  - `resources/references/free5gc-main/NFs/bsf/internal/sbi/api_management.go`
  - `resources/references/free5gc-main/NFs/bsf/internal/sbi/processor/subscriptions.go`
  - evidence：handler/processor separation、POST/PUT/DELETE、Location、generated models

secondary exemplars：

- SMF consumer/callback
  - `resources/references/free5gc-main/NFs/smf/internal/sbi/consumer/`
  - `resources/references/free5gc-main/NFs/smf/internal/sbi/processor/notifier.go`
- UDR persistence boundary
  - `resources/references/free5gc-main/NFs/udr/internal/sbi/processor/`

free5GC沒有同一個NWDAF拆成Go與Python backend、Python direct callback或cross-process raw storage的直接
precedent。這些是本專案已確認的architecture decision；exemplar只用於package/consumer/test shape。

---

## 3. Standard Contract Audit

### 3.1 NWDAF Events Subscription

依`TS29520_Nnwdaf_EventsSubscription.yaml`及TS 29.520 clauses 4.2.2.2至4.2.2.4：

| Operation | Method/path | Success | Required plan behavior |
|---|---|---|---|
| Create | `POST /nnwdaf-eventssubscription/v1/subscriptions` | 201 + mandatory Location + representation | Go external ingress；AnLF domain create |
| Update | `PUT /nnwdaf-eventssubscription/v1/subscriptions/{subscriptionId}` | 200 representation或204 | 第一版固定200 representation |
| Delete | `DELETE .../{subscriptionId}` | 204 | Go route與AnLF resource一致刪除 |
| Notify | `POST {notificationURI}` | consumer 204 | AnLF產生standard array；Go執行external delivery |

Create request使用`NnwdafEventsSubscription`。TS 29.520 clause 4.2.2.2.2要求成功create建立並保存
subscription、assign ID、回201 representation及Location；partial acceptance可使用`failEventReports`。

Update不是PATCH。TS 29.520 clause 4.2.2.2.3允許200 representation或204；為使Go externalization與contract
test deterministic，第一版固定200。

Delete依OpenAPI成功204；不存在resource依standard 404 `ProblemDetails`。

Notify body依OpenAPI為：

```text
array<NnwdafEventsSubscriptionNotification>, minItems=1
```

Go不得將AnLF backend產生的standard notification轉回custom `AnalyticsReport`後重建。

### 3.2 Admission And Data Availability

TS 29.520 clause 4.2.2.2.2的直接證據：

- Events Subscription成功時先建立resource並回201。
- immediate report只在資料available時包含。
- partial event acceptance可透過`failEventReports`表達。
- 只有要求past statistics且必要資料不存在時，明確要求500並使用cause `UNAVAILABLE_DATA`。

TS 23.288 clause 6.2.2.2另外描述NWDAF在資料蒐集procedure中再向data source
`Nnf_EventExposure_Subscribe`。

本phase據此固定：

1. future/ongoing analytics subscription不必等待SMF subscription及後續SMF/UPF delivery setup完成才回201。
2. AnLF backend必須先接受standard resource並建立collection intent。
3. collection worker在response後進行discovery/selection/subscription。
4. historical analytics若已知必要資料不存在，回500 `UNAVAILABLE_DATA`。
5. unsupported event/feature由AnLF backend在create response的`failEventReports`或standard error表達。
6. collection暫時失敗不刪除已接受的external resource；保留retry與observability。
7. 若未來要向consumer發failure/termination notification，只能使用已negotiated且適用的
   `failNotifyCode`或`termCause`；Phase 3不自行發明failure callback。

### 3.3 NRF NF Discovery

TS 29.510 clause 5.3.2.2.2與`TS29510_Nnrf_NFDiscovery.yaml`定義：

```http
GET {nrfApiRoot}/nnrf-disc/v1/nf-instances
    ?target-nf-type=SMF
    &requester-nf-type=NWDAF
    &service-names=nsmf-event-exposure
```

- `target-nf-type`與`requester-nf-type`required。
- filter criteria放query parameter。
- 成功200，body為`SearchResult`。
- `SearchResult`至少含`validityPeriod`與`nfInstances`。
- no match仍可200並回empty array/no-profile information。
- invalid query回400 `ProblemDetails`。
- requester不允許discover回403。
- NRF internal error回500。
- redirection依規格使用3xx與Location。

Go/backend interaction必須保留這個standard shape：

- private path可以不同。
- method仍是GET。
- query parameter name、encoding及semantics相同。
- Go注入/驗證NWDAF requester identity。
- response保留完整`SearchResult`，不得轉成custom endpoint array。
- NRF的standard error/status回給AnLF backend，不包成private error code。

### 3.4 SMF Subscription

TS 29.508 clause 4.2.3及`TS29508_Nsmf_EventExposure.yaml`：

| Operation | Method/path | Success |
|---|---|---|
| Create | `POST /nsmf-event-exposure/v1/subscriptions` | 201 + Location + `NsmfEventExposure` |
| Read | `GET .../subscriptions/{subId}` | 200 representation |
| Replace | `PUT .../subscriptions/{subId}` | 200 representation或204 |
| Delete | `DELETE .../subscriptions/{subId}` | 204 |

Create body包含`notifUri`、`notifId`及`eventSubs`等procedure-required資訊。對`UPF_EVENT`/`QOS_MON`
的explicit subscription，TS 29.508 clause 4.2.3.2明確說明會造成UPF direct notification。

AnLF backend選定SMF後，交給Go的body必須是完整`NsmfEventExposure`。target SMF correlation可用private
path/query/header表達，但不得包住或重新命名standard body。

TS 29.508 clause 4.2.4.2要求NF service consumer對已建立resource URI發送HTTP DELETE，SMF
成功移除resource時回204。因此AnLF backend擁有collection refcount與cleanup decision，並向Go發送
標準形狀DELETE；Go只依已保存Location執行SMF operation與保留standard status。

### 3.5 SMF And UPF Notifications

TS 29.508 clause 4.2.2.2：

- SMF POST `NsmfEventExposureNotification`到subscription的`notifUri`。
- notified NF successful processing後回204。
- processing error依Nsmf error handling回應。

TS 29.564 clauses 5.2.1.2與6.1.5.2：

- subscription可以經SMF建立。
- UPF直接POST `NotificationData`到NF service consumer提供的event notification URI。
- successful notification回204。

因此PyAnLF提供兩個typed callback boundaries；exact URI由PyAnLF config維護並放進standard subscription：

- SMF notification：`NsmfEventExposureNotification`
- UPF notification：`NotificationData`

兩者不得先送Go collector。Go只保存/管理Events Subscription routing與SMF subscription Location。
成功callback在schema/correlation驗證且最新notification已進入bounded ingestion buffer後回204。
capacity滿載時以drop-oldest接納新資料，不以queue full回503；這不取代OpenAPI對malformed、
oversized或無法correlate request的standard error response。

### 3.6 ADRF Storage

TS 29.575 clause 4.2.2.2及`TS29575_Nadrf_DataManagement.yaml`：

```http
POST {adrfApiRoot}/nadrf-datamanagement/v1/data-store-records
Content-Type: application/json

NadrfDataStoreRecord
```

- request bodyrequired。
- 成功201。
- response含created `NadrfDataStoreRecord` representation。
- Location mandatory且含`storeTransId`。
- error依Nadrf clause 5.1.7/OpenAPI `ProblemDetails`。

AnLF backend建立完整standard record；Go只驗證、選擇configured ADRF client並forward。Go/backend response
也保留201、Location及representation，不改成boolean storage result。

### 3.7 HTTP Status Matrix

下表直接取自本phase引用的Release 18 OpenAPI。實作與contract tests必須以此為allowlist；不得因private
transport方便而把standard peer response統一改成200、409或自訂error code。`default` response依共同
OpenAPI definition處理，能形成`ProblemDetails`時保留其standard fields。

| Boundary operation | Success | Redirect/error responses declared by OpenAPI |
|---|---|---|
| NWDAF Events create | 201 + Location + representation | 400、401、403、404、411、413、415、429、500、502、503、default |
| NWDAF Events update | 200 + representation或204 | 307、308、400、401、403、404、411、413、415、429、500、501、502、503、default |
| NWDAF Events delete | 204 | 307、308、400、401、403、404、429、500、501、502、503、default |
| NWDAF Events notify | 204 | 307、308、400、401、403、404、411、413、415、429、500、502、503、default |
| NRF NF discovery | 200 `SearchResult` | 307、308、400、401、403、404、406、411、413、415、429、500、501、503、default |
| SMF subscription create | 201 + Location + representation | 400、401、403、404、411、413、415、429、500、502、503、default |
| SMF subscription read | 200 + representation | 307、308、400、401、403、404、406、429、500、502、503、default |
| SMF subscription replace | 200 + representation或204 | 307、308、400、401、403、404、411、413、415、429、500、502、503、default |
| SMF subscription delete | 204 | 307、308、400、401、403、404、429、500、502、503、default |
| SMF notification callback | 204 | 307、308、400、401、403、404、411、413、415、429、500、502、503、default |
| UPF notification callback | 204 | 307、308、400、401、403、404、411、413、415、429、500、502、503、default |
| ADRF data-store create | 201 + Location + representation | 400、401、403、404、411、413、415、429、500、502、503、default |

目前ordinary HTTP決策只表示本phase不實作OAuth/TLS，不表示從standard contract刪除401/403等response。
Go與backend不得主動偽造未發生的security failure，但standard peer真的回傳時仍須依該operation規格處理。

---

## 4. Current Ownership And Required Deletions

### 4.1 Current Go Behavior

目前`internal/sbi/processor/eventssubscription.go`仍擁有：

- supported-event decision
- subscription business state
- AnLF runtime apply/release
- collection requirements
- SMF endpoint resolution
- 遍歷所有discovered endpoints並自動訂閱
- SMF/UPF callback URI
- observation source/binding
- notification processing與storage routing

`internal/sbi/processor/data_collection.go`目前：

- 依Go config選擇`nrf`或`configured`
- NRF discovery只抽出endpoint
- Go自動向所有endpoint訂閱
- 產生Go-owned collector URI
- 管理collection profile/refcount

### 4.2 Current Private Contracts To Remove

target cutover後不再使用：

- `SubscriptionRuntimeContext`
- custom `AnalyticsReport`
- `PUT /subscriptions/{id}/runtime`作為business create/update
- collection-requirements GET
- observation-bindings API
- observation-sources ingestion API
- Go collector到PyAnLF observation forwarding
- runtime-completion callback只為Go釋放collection
- private callback token與custom `Idempotency-Key` cache

若legacy route需短期共存以完成atomic cutover，必須在同一phase結束前移除production caller與dead code；
不得留到未指定的future cleanup。

### 4.3 Current Phase 2 Contract To Replace

- AnLF readiness success立即usable
- MTLF-only `POST /internal/v1/data-source-selection`
- Go-owned Mongo availability
- strict source mismatch 409

會被unified readiness→sync→usable lifecycle取代。

---

## 5. Phase Outcomes

Phase 3完成後：

1. external Events Subscription POST/PUT/DELETE符合TS 29.520 method/body/status。
2. backend create使用POST，不使用private PUT upsert。
3. AnLF backend擁有standard subscription representation及analytics runtime。
4. AnLF backend使用UUIDv4產生subscription ID；Go直接沿用為external resource ID。
5. Go只保存external routing與backend restart sync所需的最小process-local snapshot。
6. AnLF backend產生完整standard analytics notification；Go只delivery。
7. AnLF backend以standard NFDiscovery query向Go要求discovery並收到完整`SearchResult`。
8. SMF mode/configured endpoints、candidate filtering與fan-out policy在AnLF backend。
9. 第一版選擇所有符合條件且提供`nsmf-event-exposure`的SMF。
10. Go只向AnLF backend以`Target-Api-Root` header指定的SMF執行standard subscription。
11. collection sharing/refcount在AnLF backend；last reference消失後由AnLF backend要求Go執行standard SMF DELETE。
12. SMF/UPF notification直接到AnLF backend。
13. callback ingestion、analytics ring及ADRF outbound/retry buffer有bounded drop-oldest與startup validation。
14. AnLF backend直接寫raw notification到Mongo。
15. AnLF backend以standard `NadrfDataStoreRecord`要求Go寫ADRF。
16. AnLF與MTLF backend都經readiness後sync才usable。
17. backend reconnect或`processInstanceId`變更後，Go將active resource snapshot同步回backend。
18. direct callback outage或buffer eviction的資料遺失限制被測試並明確記錄。
19. 不新增parallel DTO、untyped map或`testdata` JSON contract fixture。

---

## 6. Scope Constraints

### 6.1 Non-goals

Phase 3不包含：

- 新analytics event implementation
- THRESHOLD/ONE_TIME behavior擴充
- Events Subscription transfer resource
- AoI-based SMF selection
- ADRF NRF discovery
- MTLF ADRF retrieval subscription/direct fetch
- MTLF Mongo query
- accuracy/retrain policy migration
- local training/model provision
- durable Go-process-restart recovery，包含routing/subscription snapshot persistence或backend-to-Go
  authority reconstruction
- TLS、mTLS、OAuth delegation或certificate管理
- Python backend NRF registration
- message broker、distributed transaction或Mongo schema migration framework

PyMTLF只因unified sync contract受到小範圍修改，不接入dataset或training feature。

### 6.2 Migration Fidelity Gate

本phase雖然明確改變collection ownership與data path，仍不得把未被本計畫取代的既有business behavior一併
改寫。開始移植前必須逐項盤點current Go implementation及tests，標記：

- `preserve`：例如all-matching SMF selection結果、subscription correlation、refcount、成功cleanup
  ordering、reporting precedence及runtime transition semantics；owner移到PyAnLF不代表這些語意可以改變。
- `explicitly replace`：例如Go-owned collector ingress、observation forwarding、Go Mongo writer，以及SMF
  selection/fan-out的owner與call path。
- `remove as obsolete`：沒有新production caller的custom contract及compatibility path。

只有後兩類的architecture/flow本身可以改變或刪除；其中若夾帶仍有意義的business semantics，仍須另外列為
`preserve`。`preserve`項目必須先保留Go characterization test，再於PyAnLF建立等價scenario；
old/new path的decision、state transition與side effect應相同。若實作時發現需要改變`preserve`行為，必須先
更新plan並取得決策，不能以Python重寫或cleanup為理由直接修改。

本phase已明確將現有SMF cleanup中「peer DELETE失敗後只記log，且local tracking已先移除」
分類為`explicitly replace`；新行為是Go保留peer Location、PyAnLF保留cleanup intent，並在
reconnect sync後retry/reconcile。refcount key、sharing result、stop-before-cleanup及last-reference delete仍為
`preserve`。現有SMF client將DELETE 200視為成功也分類為`explicitly replace`；Release 18
`TS29508_Nsmf_EventExposure.yaml`只定義204 success，新contract不再無聲接受peer 200。

---

## 7. Target Ownership

### 7.1 Go

Go擁有：

- external Nnwdaf route與NWDAF identity
- standard request parsing與wire validation
- external Location
- minimal process-local mapping：external subscription ID、external notification URI、accepted standard snapshot
- SMF create回應的peer resource Location與target routing metadata，直到cleanup成功
- backend availability/sync orchestration
- NRF discovery client
- SMF subscription client
- ADRF storage client
- external analytics notification client
- SMF resource Location及unsubscribe routing

Go不擁有：

- reporting policy/default
- SMF candidate selection
- fan-out count
- collection requirement derivation
- SMF configured endpoint或selection mode
- collection profile sharing或reference counting
- raw notification transformation
- storage source preference
- analytics scheduling/inference/shaping

### 7.2 AnLF Backend

AnLF backend擁有：

- standard Events Subscription domain resource
- UUIDv4 subscription ID generation
- supported features/events與partial failure
- reporting precedence/runtime
- data collection intent
- discovery/configured mode selection
- configured SMF endpoint ownership
- SMF candidate filtering
- standard SMF subscription construction，包括direct UPF notification所需欄位
- callback URI
- direct SMF/UPF notification processing
- raw Mongo storage
- standard ADRF storage record construction
- analytics inference與standard notification production
- collection retry與cleanup intent
- collection sharing/refcount及last-reference SMF delete decision

### 7.3 MTLF Backend

本phase只擁有：

- readiness後sync participation
- 對available data sources的preference/effective selection contract foundation

Phase 5才加入direct data retrieval。

---

## 8. Proposed Private Routes

這些path是deployment-private，但standard operation必須保留相同method/body/response。

### 8.1 Events Subscription Routing

```text
POST   /internal/v1/events-subscriptions
PUT    /internal/v1/events-subscriptions/{subscriptionId}
DELETE /internal/v1/events-subscriptions/{subscriptionId}
POST   /internal/v1/events-subscription-notifications
```

- POST/PUT body：`NnwdafEventsSubscription`
- POST success：201 + Location + `NnwdafEventsSubscription`
- PUT success：200 + representation
- DELETE success：204
- notification body：`array<NnwdafEventsSubscriptionNotification>`
- notification success：Go完成external delivery並收到consumer 204後，回204給AnLF backend

POST由AnLF backend產生UUIDv4 `subscriptionId`。Go沿用同一ID建立external Location，並在確認
backend create成功後才寫入自己的memory routing table；不建立Go/backend雙ID mapping。

Go送backend前把external `notificationURI`換成generic Go internal notification URI；body仍是standard shape。
Go在external response還原原consumer URI。routing依notification item的`subscriptionId`，不新增opaque callback
token或custom report envelope。

### 8.2 NRF Discovery Proxy

```text
GET /internal/v1/nrf/nf-instances
```

query/response完全模仿`TS29510_Nnrf_NFDiscovery.yaml`。Go至少：

- 強制`target-nf-type=SMF`
- 強制/驗證`requester-nf-type=NWDAF`
- 強制/驗證`service-names`含`nsmf-event-exposure`
- 注入containing NWDAF `requester-nf-instance-id`（若current client支援）
- forward NRF 200 `SearchResult`
- forward/normalize standard 3xx、400、403及5xx `ProblemDetails`

AnLF backend保存`validityPeriod`並避免在相同query仍valid時重複discovery；TS 29.510 clause 5.3.2.2.1
建議reuse相同query的有效結果。

### 8.3 SMF Event Exposure Proxy

概念routes：

```text
POST   /internal/v1/smf-event-exposure/subscriptions
PUT    /internal/v1/smf-event-exposure/subscriptions/{subId}
DELETE /internal/v1/smf-event-exposure/subscriptions/{subId}
```

body/response使用`NsmfEventExposure`。因standard body不包含target apiRoot，private transport使用
`Target-Api-Root` header，不建立JSON wrapper。NRF與configured模式都由
AnLF backend選擇target；Go只驗證URI shape、執行standard request並保存create回應Location。

Go回AnLF backend時保留SMF的standard success status、Location、representation及`ProblemDetails`。
PUT/DELETE依create時保存的Location routing，不因NRF cache expiry或configured endpoint改變而重新選擇
resource。DELETE沒有body，成功只接受OpenAPI定義的204。

### 8.4 ADRF Storage Proxy

```text
POST /internal/v1/adrf-data-management/data-store-records
```

- body：`NadrfDataStoreRecord`
- success：201 + Location + same standard representation
- failure：Nadrf-defined `ProblemDetails`

第一版使用Go config固定ADRF apiRoot，不做discovery。

### 8.5 Unified Sync

兩個backend使用相同lifecycle：

```text
GET /health/ready -> POST /internal/v1/sync -> USABLE
```

sync body是小型private envelope，但每個resource snapshot保留standard body。第一版禁止free-form map。

health response在200 ready與503 not-ready兩種狀態都回報每次backend process啟動都重新產生的
`processInstanceId` UUID。Go在USABLE後
仍持續poll；假如UUID變更，即使中間沒有觀察到probe failure，也必須將backend視為新
process並重新完整sync。`processInstanceId`不是schema version或持久NF identity。
現有health payload只加入這個欄位，不把snapshot塞進readiness response：

```json
{
  "status": "ready",
  "processInstanceId": "5f5db241-1e7e-42c8-9bb0-35653f8ef6d4"
}
```

AnLF最低snapshot：

- containing NWDAF public runtime identity
- active external subscription ID
- accepted `NnwdafEventsSubscription`
- current external notification URI routing metadata
- existing SMF peer resource Locations（如仍有效）
- pending/orphan SMF cleanup routing records
- configured ADRF capability

MTLF最低snapshot：

- containing NWDAF public runtime identity
- currently observed `adrf`/`mongodb` availability
- current preferred/effective retrieval source（Phase 5啟用前可為空）

sync response最低回報：

- backend `processInstanceId`
- snapshot是否完整理解與恢復
- AnLF-observed Mongo/storage availability
- MTLF source preference/effective selection（Phase 5有consumer後啟用）

Go整合這些typed observation後，只將與對方有關的資訊同步給另一backend。AnLF與MTLF
backend不直接sync。

initial/reconnect sync在成功前會擋住USABLE。USABLE後每個periodic health cycle仍可進行full snapshot
refresh，Go-owned snapshot或backend回報的typed observation改變時也立即喚醒受影響backend。refresh
期間不把正常backend反覆切成SYNCING，但refresh失敗將它轉UNAVAILABLE並回到backoff retry。
Go以typed semantic equality比較是否需要轉發，不在wire上加revision/CAS或形成無限sync loop。

backend回response確認snapshot是否已完整理解與恢復，並回報source observation。第一版每次都以
完整snapshot取代backend中「Go-owned synchronized projection」的舊狀態，不是清空backend-owned
runtime、refcount或pending work；backend依完整snapshot進行reconcile。不加入revision/CAS或partial patch protocol。

Go的active resource與routing snapshot只存於memory。Go process停止或重啟後不復原現有訂閱；
實驗直接重跑並由consumer重新訂閱。本phase不加入持久化、backend-to-Go雙向authority
recovery或state conflict protocol。

---

## 9. End-to-end Procedures

### 9.1 External Create

```text
consumer -> Go POST NnwdafEventsSubscription
Go -> validate body/content type and AnLF usability
Go -> replace notificationURI with generic internal Go callback
Go -> AnLF POST standard resource
AnLF -> validate domain, generate UUIDv4, create resource/runtime, return 201 + Location + representation
Go -> reuse the same UUID and record external URI plus accepted process-local snapshot
Go -> externalize Location and notificationURI
Go -> consumer 201

AnLF worker -> begin discovery/SMF selection/subscription asynchronously
```

collection failure不rollback已成功create的analytics resource。historical data request若在create時已知資料不存在，
AnLF直接回500 `UNAVAILABLE_DATA`，Go保留standard cause。

### 9.2 External Update

```text
consumer -> Go PUT full standard resource
Go -> verify route and AnLF usable
Go -> rewrite notificationURI
Go -> AnLF PUT /{subscriptionId}
AnLF -> update domain resource/runtime, return 200 representation
Go -> atomically update external route snapshot
Go -> consumer 200 externalized representation

AnLF worker -> reconcile collection intent asynchronously
```

backend rejection不修改Go route。Go成功更新route後的新notification使用最新external URI。

### 9.3 External Delete

```text
consumer -> Go DELETE /{subscriptionId}
Go -> AnLF DELETE /{subscriptionId}
AnLF -> stop runtime, remove resource, release local collection references
AnLF -> record cleanup intents for SMF resources whose last reference disappeared
Go <- 204
Go -> remove external route
Go -> consumer 204

PyAnLF collection-cleanup worker -> Go DELETE each no-longer-needed SMF subscription
Go -> DELETE saved SMF resource Location
SMF -> 204, or standard error retained for PyAnLF retry/reconcile
```

不存在resource回404。backend unavailable時不先刪Go route。external 204只等待AnLF backend已接受
本地resource deletion與cleanup intent，不等待所有peer SMF DELETE。SMF cleanup失敗時Go保留Location，
PyAnLF保留intent並以bounded backoff重試；SMF 404在wire上仍為404，但PyAnLF可視為已無
resource的terminal cleanup outcome。

### 9.4 Discovery And Fan-out

```text
AnLF -> Go GET standard-shaped NF discovery query
Go -> NRF GET /nnrf-disc/v1/nf-instances
Go <- SearchResult
AnLF <- SearchResult
AnLF -> filter candidates that expose nsmf-event-exposure
AnLF -> first version selects all candidates
for each selected candidate:
    AnLF -> Go POST NsmfEventExposure + Target-Api-Root header
    Go -> SMF POST /subscriptions
    Go <- 201 + Location
    AnLF <- 201 + Location
```

Go不自行loop全部discovered endpoint。selection與fan-out loop在AnLF backend。

### 9.5 Direct Notification And Storage

```text
SMF/UPF -> AnLF callback standard notification
AnLF -> validate/correlate
AnLF -> append newest notification to bounded ingestion buffer
AnLF -> evict oldest buffered notification if capacity is full
AnLF -> 204

AnLF storage worker:
    -> Mongo raw notification write (direct)
    -> construct NadrfDataStoreRecord
    -> Go ADRF storage proxy
    -> ADRF POST /data-store-records
```

Phase 3 bootstrap在MTLF尚未選擇effective source前，向所有available sinks寫入。單一sink failure不撤銷另一個
已成功sink；failure需有bounded-backoff retry、structured log及health/metric，但不把原始standard
notification改成normalized training record。ingestion與ADRF outbound/retry buffer滿載時都接納最新資料並
淘汰最舊未處理資料；這個best-effort loss以counter、buffer utilization及bounded warning暴露。

### 9.6 Analytics Notification

```text
AnLF scheduler -> standard notification array
AnLF -> Go POST /events-subscription-notifications
Go -> validate subscriptionId and standard shape
Go -> lookup current external notificationURI
Go -> consumer POST standard array
consumer -> 204
Go -> AnLF 204
```

Go不重算analytics內容。external transport/network error依Nnwdaf callback允許的status與local retry policy返回
AnLF；第一版不加private idempotency key/cache。

### 9.7 Backend Restart

```text
Go polling detects AnLF READY
Go compares processInstanceId with the last successful probe
Go -> AnLF sync active standard resources and routing snapshot
AnLF -> restore resource/runtime
AnLF -> reconcile collection intent and peer resources
AnLF -> sync success
Go marks AnLF USABLE
```

若`processInstanceId`變更，即使Go沒有在兩次probe間看到failure，也必須執行完整sync。若Go仍
持有有效SMF subscriptions且callback URI不變，AnLF不重複create；若peer resource失效，再透過
standard GET/POST/DELETE reconcile。AnLF outage期間direct callback遺失不回補；恢復後只保證future collection。

---

## 10. Raw Mongo Contract

### 10.1 Document Layout

Mongo由AnLF backend建立collection、indexes及write path。第一版document保存一個完整收到的standard
notification，不拆成Go training record：

```json
{
  "source": "smf",
  "receivedAt": "2026-07-21T12:00:00Z",
  "measurementTime": "2026-07-21T11:59:59Z",
  "correlationId": "corr-1",
  "notification": {}
}
```

規則：

1. `notification`保存收到的`NsmfEventExposureNotification`或`NotificationData`原始JSON。
2. 不加入`schemaVersion`。
3. metadata只為bounded time-range/source/correlation query及indexes。
4. 不同standard notification type需可由`source`或typed collection清楚辨識。
5. MTLF backend未來只讀，不寫、不建index、不做migration。
6. raw payload解析、deduplication、feature extraction與training dataset shaping屬MTLF。
7. retention由deployment config設定，不在第一版建automatic migration。
8. `data/`或local database artifact不得進git。

collection name依PyAnLF現有config/repository naming在實作時固定，Phase 3同時建立本phase query與
retention所需的baseline indexes；Phase 5有真實MTLF query後才補其額外index。已確認每個完整notification
一筆document，不得回到每個measurement一筆`UpfTrafficRecord`。

### 10.2 Bounded Buffer Contract

PyAnLF將以下三種state分開，不以同一個「queue」模糊表達：

1. callback ingestion buffer：已通過callback validation、等待storage與analytics processing的raw notification。
2. analytics observation ring：單次推論與accuracy alignment所需的recent observation window。
3. ADRF outbound/retry buffer：尚未成功寫入ADRF或等待retry的standard record。

共同政策：

- capacity由PyAnLF config設定，提供偏大但有界的預設值。
- capacity滿載時drop oldest並保留newest，不因buffer full使callback失敗。
- 每種buffer提供current depth/capacity、drop-oldest counter與bounded warning log。
- startup對non-positive capacity、invalid maximum request size及analytics ring小於input window等明確錯誤
  fail fast。
- startup對理論最大memory或與configured source/window明顯不成比例的設定產生warning；不以
  未有實際負載證據的任意上限拒絕合理的大buffer。

此政策表示callback 204後的舊資料仍可因overload eviction遺失；這是第一版實驗環境已接受
的best-effort語意，不能在文件或metrics中宣稱durable delivery。

---

## 11. Repository Work Plan

### 11.1 NWDAF

#### Standard contract boundary

- 補足Release 18 Events Subscription wire model
- typed standard notification array
- standard ProblemDetails mapping
- POST create、PUT update、DELETE semantics
- internal Location與external Location rewrite

#### NRF discovery consumer

- 將現有`DiscoverSmfEventExposure`從endpoint-only結果改為完整`SearchResult`
- 支援standard query parameter
- 提供private standard-shaped route給AnLF backend
- requester identity由Go固定/驗證
- 不在Go進行candidate selection
- 移除Go中`nrf`/`configured` SMF selection mode與configured endpoint ownership；只保留generic SBI
  transport timeout等通訊設定

#### SMF/UPF consumer

- 接受AnLF選定的`Target-Api-Root` header及完整`NsmfEventExposure`
- standard create/read/replace/delete transport
- 保存peer Location只供routing/reconcile，成功cleanup前不移除
- 移除Go自動遍歷全部discovered SMF的policy
- callback URI改為AnLF-owned endpoint

#### ADRF storage

- typed `NadrfDataStoreRecord` client
- configured ADRF apiRoot
- 201/Location/representation
- standard error pass-through

#### Resource routing/sync

- UUIDv4 external/backend shared subscription ID
- 最小process-local external subscription record
- unified backend sync orchestration
- health `processInstanceId`與快速restart detection
- AnLF reconnect snapshot
- MTLF sync seam
- 移除MTLF-only strict handshake production use

#### Legacy removal

- 移除Go collector/observation forwarding production caller
- 移除collection requirements、bindings及runtime completion seam
- 移除Go Mongo traffic-data writer及舊normalized record path
- 保留仍由其他feature使用的code時，必須列出actual caller，不能只因可能有用而保留

### 11.2 PyAnLF

#### Standard resource

- typed `NnwdafEventsSubscription`
- POST/PUT/DELETE resource service
- backend-generated UUIDv4 subscription ID
- standard representation與ProblemDetails
- existing runtime manager整合，不建立第二套scheduler

#### Discovery/selection

- `nrf`/`configured` mode config移入PyAnLF
- configured SMF endpoints移入PyAnLF
- standard-shaped NF discovery client to Go
- `SearchResult` parsing與validity cache
- filter service status/name/apiRoot
- first version all-matching-SMF selection
- future AoI policy extension seam，不實作AoI

#### Data collection

- construct standard `NsmfEventExposure`
- selected `Target-Api-Root` routing header
- direct callback URI
- selected-target create/update/delete
- collection intent/retry/refcount及last-reference cleanup
- app-owned collection cleanup worker與standard-shaped Go DELETE client
- restart sync restore/reconcile

#### Callback/storage

- typed SMF notification endpoint
- typed UPF notification endpoint
- 204/error behavior
- separately named bounded ingestion、analytics ring及ADRF outbound/retry buffers
- drop-oldest metrics與startup config validation
- callback body-size limit、buffer capacities及memory warning threshold的PyAnLF config/defaults
- raw Mongo repository/index
- standard ADRF storage request through Go
- source bootstrap dual-write

#### Analytics output

- full standard notification array
- Go delivery client
- remove custom `AnalyticsReport` production path

### 11.3 PyMTLF

- replace strict data-source-selection handshake participation with unified sync
- generate and report per-process `processInstanceId`
- do not add retrieval/training feature yet
- preserve artifact/health behavior
- remove old endpoint once Go production caller is gone

### 11.4 Documentation

- update sample config ownership
- document current NRF direct SMF discovery as project limitation
- document future AoI selection
- document direct callback outage loss
- document plain HTTP scope
- update Phase 2 implementation/replan record

---

## 12. Implementation Sequence

### Step 1: Record Confirmed Contracts And Behavior Inventory

- lock section 16 confirmed decisions into contract tests
- inventory current Go behavior as preserve/replace/remove
- identify existing characterization tests and missing edge cases

### Step 2: Lock Behavior And Contract Types

- audit current generated models
- scoped generate missing Release 18 types
- add Go/Python request/response tests
- no JSON fixture files

### Step 3: Implement Unified Sync

- extend existing availability monitor
- AnLF and MTLF READY→SYNCING→USABLE
- process incarnation UUID、central typed snapshot fan-out
- snapshot tests、stale sync tests、rapid-restart tests
- keep old strict handshake until new caller works, then remove

### Step 4: Cut Over Events Subscription CRUD

- AnLF POST/PUT/DELETE standard resource
- UUIDv4 ID generation and validation
- Go external routing/Location rewrite
- minimal snapshot
- standard error matrix

### Step 5: Move Discovery And Selection

- Go full SearchResult consumer/proxy
- PyAnLF mode config、cache、candidate filtering
- all-matching selection
- selected target passed back to Go

### Step 6: Move SMF/UPF Collection

- PyAnLF builds standard subscription
- PyAnLF owns SMF config、sharing/refcount及cleanup intent
- Go executes peer calls
- PyAnLF last-reference cleanup calls Go standard-shaped DELETE
- callbacks direct to PyAnLF
- remove Go observation forwarding

### Step 7: Move Raw Storage

- PyAnLF Mongo repository
- ADRF storage proxy
- ingestion/analytics/ADRF buffer config、drop-oldest與startup validation
- bootstrap multi-sink write
- remove Go legacy Mongo writer

### Step 8: Cut Over Analytics Notification

- PyAnLF standard array
- Go delivery only
- remove custom report contract

### Step 9: Cleanup And Cross-process Verification

- remove old private APIs/callers
- run Go/Python unit, race, lint, build
- real Go/PyAnLF/PyMTLF/NRF/SMF/UPF/Mongo smoke matrix where environment permits

---

## 13. Verification Plan

### 13.1 Standard Contract Tests

Events Subscription：

- POST success 201 + Location + representation
- PUT success 200 representation
- DELETE success 204
- malformed/unsupported field does not silent-drop
- 404/503 ProblemDetails
- notification array minItems and subscriptionId

NF discovery：

- standard required query
- full SearchResult round-trip
- validityPeriod preserved
- no-match 200 empty result
- 400/403/500 pass-through
- requester type cannot be overridden

SMF/UPF：

- SMF POST 201 + Location
- SMF PUT 200/204
- SMF DELETE 204
- PyAnLF last-reference decision is the only SMF DELETE trigger
- SMF DELETE failure retains Location and cleanup intent for retry
- SMF DELETE 404 remains standard on the proxy boundary and completes PyAnLF reconciliation
- direct SMF callback typed body and 204
- direct UPF callback typed body and 204
- AnLF chooses all matching candidates; Go does not auto-fan-out

ADRF storage：

- typed POST request
- 201 + Location + representation
- standard error response

### 13.2 Availability/Sync Tests

- both backend late start
- readiness success but sync failure remains unusable
- backend restart triggers resync
- backend rapid restart between successful probes changes `processInstanceId` and triggers resync
- usable backend receives periodic refresh without transient request-gate flapping
- typed state change wakes only affected backend and does not create a sync loop
- active AnLF resource snapshot restore
- peer SMF Location and orphan cleanup restore
- MTLF sync without retrieval feature
- AnLF Mongo observation reaches MTLF source snapshot through Go
- MTLF effective selection reaches AnLF snapshot through Go
- Go restart does not claim subscription recovery
- old strict endpoint no production caller
- shutdown/race

### 13.3 Storage Tests

- raw SMF notification round-trip
- raw UPF notification round-trip
- metadata/index time-range query
- no schemaVersion
- Mongo fail/ADRF success
- ADRF fail/Mongo success
- both available bootstrap dual-write
- ingestion full evicts oldest and accepts newest
- analytics ring full evicts oldest and still covers input window
- ADRF outbound/retry full evicts oldest and increments drop metric
- invalid capacities and undersized analytics ring fail startup validation
- high theoretical memory setting emits bounded startup warning
- no Go Mongo write

### 13.4 Process Matrix

1. consumer → Go → PyAnLF create
2. PyAnLF → Go → NRF discovery → full SearchResult
3. PyAnLF selects all → Go → multiple SMF subscriptions
4. consumer delete → PyAnLF refcount → Go → last-reference SMF DELETE
5. SMF/UPF → PyAnLF direct callback
6. PyAnLF → Mongo and PyAnLF → Go → ADRF storage
7. PyAnLF analytics → Go → consumer notification
8. PyAnLF stop/restart → process ID change → Go polling/sync → future collection restored

### 13.5 Commands

NWDAF：

```bash
make test
make lint
make build
go test -race ./internal/...
```

PyAnLF/PyMTLF：

```bash
ruff check .
pytest
```

script/test execution依workspace policy使用elevated permission。若real NRF/SMF/UPF/ADRF/Mongo environment
不可用，final record必須分別列出未執行項目，不以mock代替。

---

## 14. Acceptance Criteria

Phase 3完成需全部成立：

1. main plan與Phase 2不再宣稱Go擁有notification ingress/Mongo writer。
2. Events Subscription create/update/delete method/status符合OpenAPI。
3. private create使用POST。
4. PyAnLF產生UUIDv4 subscription ID，Go沿用同一ID。
5. PyAnLF擁有standard resource/runtime。
6. Go只保存process-local routing/sync snapshot，不保存analytics policy或refcount。
7. NF discovery boundary使用standard query與完整SearchResult。
8. SMF mode/config、selection/fan-out及refcount在PyAnLF。
9. 第一版all-matching behavior有tests。
10. Go只對`Target-Api-Root` header指定的target執行standard request。
11. consumer delete與SMF cleanup分離，只有PyAnLF last-reference decision觸發standard SMF DELETE。
12. SMF cleanup失敗保留Location與intent並可在reconnect後retry/reconcile。
13. SMF/UPF notification直接到PyAnLF。
14. callback 204/error behavior符合TS 29.508/29.564與已確認drop-oldest policy。
15. PyAnLF寫raw Mongo；Go不寫。
16. ADRF storage使用standard record、201及Location。
17. AnLF/MTLF都完成readiness後sync，並在`processInstanceId`變更後重新sync。
18. backend restart恢復future collection intent，且USABLE後仍持續polling。
19. buffer startup validation、drop metrics與best-effort loss語意有tests與文件。
20. Go restart不宣稱復原subscription，memory state遺失後由實驗重跑。
21. custom AnalyticsReport、collection requirements、observation forwarding與runtime completion production
    paths被移除。
22. plain HTTP是唯一current security scope。
23. tests/lint/build與未執行environment checks完整記錄。
24. 所有被移植或刪除的Go business behavior都有`preserve`／`explicitly replace`／`remove as obsolete`分類。
25. 所有`preserve`行為在移除Go production path前已有Python parity tests，且沒有未經決策的語意變更。

---

## 15. Risks And Controls

### 15.1 Direct Callback Loss

AnLF backend停機期間SMF/UPF callback可能失敗且資料遺失。第一版接受此限制；control是backend polling、
restart sync、future subscription reconcile及清楚observability，不宣稱durable delivery。

### 15.2 Standard Type Version Gap

V17 generated model可能丟失R18欄位。control是scoped generation/isolated compatibility package及
unknown-field rejection tests。

### 15.3 Duplicate SMF Subscription

PyAnLF restart後若不辨識existing peer Location可能重複subscribe。sync snapshot需包含peer resource
correlation；reconcile先驗證existing resource再create。

### 15.4 Source Availability Divergence

Mongo由AnLF直接使用、ADRF由Go呼叫、MTLF未來直接讀，三者觀察可能短暫不同。sync只傳observed state與
effective selection；runtime operation failure仍更新local availability，不把sync當永久truth。

### 15.5 Premature 204

callback在payload通過schema/correlation驗證且newest item已進入ingestion buffer後回204，不等待
ADRF或analytics completion。buffer full會drop oldest，因此204不是durable storage保證。control是明確
best-effort contract、drop counter、utilization metric、startup validation及failure/overflow tests。

### 15.6 Internal API Re-expansion

target selector與sync容易演變成通用RPC。control是standard body原樣保留、private metadata最小化、每個欄位
必須有production consumer。

### 15.7 Missed Rapid Backend Restart

backend可能在兩次成功health probe間完成crash/restart，如果只比較HTTP status，Go會錯把已清空
memory的新process視為原process。control是每次process啟動產生新的`processInstanceId` UUID；
UUID改變必定導致READY→SYNCING→USABLE。

### 15.8 Go Restart State Loss

Go routing/snapshot只在memory，Go process停止或重啟後無法恢復external consumer URI與SMF resource
routing。這是實驗環境已接受的限制：重跑實驗並由consumer重新訂閱，本phase不實作
持久化、冷啟動reconstruction或雙向authority conflict resolution。

---

## 16. Confirmed Decisions

### 16.1 Resource ID And Go Memory Snapshot

- AnLF backend在private POST create時產生UUIDv4 `subscriptionId`。
- Go沿用同一ID作external resource ID，只rewrite apiRoot/Location。
- Go在memory保留accepted standard resource、external notification routing及SMF peer Location，供backend
  reconnect sync使用；不在Go重複analytics policy或refcount。
- Go process重啟時不復原這些state；實驗重跑並由consumer重新訂閱。

### 16.2 SMF Config, Target And Cleanup Ownership

- `nrf`/`configured` mode、configured SMF endpoints、candidate selection及fan-out都在PyAnLF。
- 兩種mode都由PyAnLF以`Target-Api-Root` private header告知Go已選定target；standard
  `NsmfEventExposure` body不加wrapper。
- PyAnLF擁有collection profile sharing、refcount與cleanup intent。
- consumer Events Subscription delete可在PyAnLF已停runtime、移除resource與記錄cleanup intent後回204，
  不等待SMF cleanup完成。
- 只有last reference消失時，PyAnLF cleanup worker才向Go發送standard-shaped SMF DELETE。
- Go依已保存Location執行DELETE；失敗保留routing供retry/reconcile，不重新選擇target。

### 16.3 Notification 204 And Buffer Policy

- JSON/schema/correlation驗證成功且newest notification進入bounded ingestion buffer後回204。
- 不等待ADRF remote write或analytics completion才回204。
- ingestion、analytics ring及ADRF outbound/retry buffer都接納newest並在滿載時drop oldest；
  queue full本身不回503。
- startup對invalid capacity/request size與ring小於input window fail fast，對過高理論memory產生warning。
- 所有drop都有counter、utilization及bounded warning；204不宣稱durable delivery。

### 16.4 Unified Continuous Sync

- Go是central sync coordinator；PyAnLF與PyMTLF不直接sync。
- 兩個backend共用`GET /health/ready -> POST /internal/v1/sync -> USABLE` lifecycle，但各自
  使用有production consumer的typed section。
- health回應包含每次process啟動重新產生的`processInstanceId` UUID。
- Go在USABLE後仍持續poll；probe/transport failure或process UUID變更都會使backend重新進入
  READY→SYNCING→USABLE，sync failure則回UNAVAILABLE並持續backoff retry。
- USABLE後的periodic refresh與typed state-change wake-up使shared snapshot持續更新；refresh期間不反覆
  關閉request gate，但refresh失敗會轉UNAVAILABLE。
- Go可將PyAnLF回報的Mongo availability分發給PyMTLF，並將PyMTLF的effective source分發給
  PyAnLF；不傳raw data、dataset、model artifact或credential。

### 16.5 Mongo Document Granularity

- 每個完整SMF/UPF notification一筆document。
- 保留原始standard JSON，只加time/source/correlation query metadata與index。
- 不加`schemaVersion`，不拆成每個measurement一筆record。

### 16.6 Deferred Storage Preference Shape

Phase 3先實作「MTLF尚未selection時寫所有available sinks」。exact `preferredSources`、`effectiveSource`及
explicit `dual` config留到Phase 5 direct retrieval plan，不在本phase預先固定。
