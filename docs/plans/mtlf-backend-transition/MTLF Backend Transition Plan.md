# MTLF Backend Transition Plan

Date: 2026-07-17

Status: Replanned after team architecture review; implementation phases pending

Related records:

- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 1 PyMTLF Foundation And Backend Boundary.md`
- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 2 Backend Connectivity And Standard Contract Foundation.md`

---

## 1. Purpose

這份文件是 MTLF backend transition 的 canonical 主計畫。

工作目標是讓 Go NWDAF 收斂成標準 SBI 與 routing layer，並依 feature 將 AnLF、MTLF domain
behavior 分別交給 AnLF backend 與 MTLF backend。這不是只修改 `NWDAF/internal/mtlf/` 的工作；
同一個標準 feature 涉及 Go、PyAnLF 與 PyMTLF 時，必須在同一 phase 內完成端到端調整。

主要成果為：

1. Go 處理標準 SBI、OpenAPI validation、NRF/OAuth、callback、resource routing 與錯誤回應
2. PyAnLF 擁有 analytics runtime、prediction、accuracy measurement 與 AnLF-side standard behavior
3. PyMTLF 擁有 accuracy policy、dataset selection、ADRF/Mongo retrieval、local training 與 model production
4. 標準功能在 Go 與 backend 間沿用 Release 18 OpenAPI payload shape，不建立平行 domain contract
5. PyMTLF 收到 `FetchInstruction` 後直接向 ADRF fetch，或直接以 read-only access 查詢 MongoDB
6. MTLF backend 以標準 ML model provision shape，經 Go 將模型 URL 提供給 AnLF backend
7. `NWDAF/`、`PyAnLF/` 與 `PyMTLF/` 最終不保留 Daisy-specific runtime、API、config 或命名

舊計畫中的 Go-owned dataset provider、normalized chunk push、provider attempt state machine、custom
generation CAS 與跨程序 durable reconciliation 不再是 target architecture。

---

## 2. Planning Rule: Organize By Feature

後續 phase 以 feature 為單位，不以 repository 或 logical function 為單位。

例如 `Nnwdaf_MLModelMonitor` phase 必須同時處理：

- Go standard SBI route、validation、resource URI 與 backend routing
- PyAnLF monitoring subscription、measurement 與 notification behavior
- PyMTLF monitoring registration、subscription request與 accuracy policy input
- 三個 repository 間的 contract、failure 與 integration tests

不得先建立一套 MTLF-only custom API，再於後續 phase 另做 AnLF translation。若 feature 的標準流程
同時涉及兩個 backend，該 phase 應一次完成可驗證的端到端 vertical slice。

---

## 3. Confirmed Architecture Decisions

以下決策已確認，實作不得無聲改回舊方向。

### 3.1 Logical Function Boundary

1. PyAnLF 與 PyMTLF 都是同一個 NWDAF 的 internal backend，不是獨立標準 NF
2. backend 不註冊 NRF，也不以自己的身分 advertise 3GPP service
3. Go-side naming 只使用 `anlfBackend`、`mtlfBackend` 與對應 Go-style identifiers
4. Go code、config、logs、tests 與 route naming 不使用 `PyAnLF` 或 `PyMTLF` implementation name
5. Python repository/package name可以使用 `PyAnLF`、`PyMTLF`、`py_anlf` 與 `py_mtlf`

### 3.2 Standard And Private Contracts

1. 有標準 OpenAPI schema 的 feature，backend API body沿用相同 JSON field、required/optional與 enum semantics
2. Go 不為 accuracy monitor、model provision、events subscription 或 ADRF retrieval 建立平行 business DTO
3. backend route仍是 private deployment boundary，不宣稱 backend 自身是標準 NF
4. routing correlation、readiness、storage handshake 等規格沒有描述的資訊可以使用小型 private contract
5. private metadata不得擴充或污染標準 body
6. current `github.com/free5gc/openapi v1.2.3`與 local Release 18 YAML 的差異必須在 feature phase明確處理

### 3.3 Training Data Boundary

1. PyMTLF決定需要的資料、時間範圍、來源與 query/fetch時機
2. ADRF retrieval subscription由 PyMTLF準備標準-shaped request，再由 Go呼叫 ADRF
3. `notificationURI`由 Go注入或重寫成 Go-owned callback；`notifCorrId`由 Go驗證 ownership與唯一性
4. ADRF callback先到 Go，Go驗證後將完整 `NadrfDataRetrievalNotification`與 `FetchInstruction`交給 PyMTLF
5. PyMTLF直接使用 `fetchUri`與 `fetchCorrIds`向 ADRF執行 `RetrievalRequest`
6. Go不代理 dataset bytes，不做 normalization、chunking、backpressure或 dataset completion coordination
7. retrieval subscription unsubscribe仍由 PyMTLF要求 Go執行，保持 Go-owned subscription lifecycle完整
8. ADRF不存在時，Go保存原始 UPF notification到 MongoDB，PyMTLF使用 read-only credential直接查詢
9. MongoDB不是 ADRF，也不對外 advertise standard service

### 3.4 Storage Mode And Bootstrap

1. supported storage modes為 `adrf`、`mongodb`與 `dual`
2. Go連上 MTLF backend後做一次簡單 requirements handshake，由 backend依 Go可用 sources選擇 mode
3. backend要求的 source不可用時，handshake失敗；Go不得靜默換成其他 source
4. handshake前只有一個 source可用時先寫該 source；兩者都可用時暫時 dual-write
5. backend重新 ready或任一 process restart後重新執行 handshake
6. transient ADRF failure不自動切換 MongoDB；需要 runtime fallback的 deployment必須使用 `dual`
7. PyMTLF第一版sample/default mode為`mongodb`；`adrf`或`dual`由deployment明確設定
8. 每次MTLF handshake前以bounded live ping重新判定Mongo availability，使Mongo可在不重啟Go下恢復
9. ADRF沒有額外non-standard health probe；valid config與已建立standard client代表available capability

### 3.5 Mongo Storage Shape

1. MongoDB保存原始 UPF notification item，不轉成 training feature record
2. 每個 notification item獨立一筆 document，保留原始 JSON內容
3. 外層只保存 query/index所需的最小 metadata，例如 correlation ID、received time與 measurement time
4. 第一版不加入 schema version欄位
5. PyMTLF負責 parsing、deduplication、dataset shaping與 preprocessing
6. Go擁有 collection建立、write path與 indexes；PyMTLF只取得 read-only access

### 3.6 Model Training And Provision

1. PyMTLF使用簡單 local Python trainer；multiple-NWDAF FL不屬於本 transition
2. PyMTLF擁有 model production與 `modelUniqueId`配置
3. model artifact以 MTLF-backend-owned private URL提供
4. provision flow使用標準 `NwdafMLModelProvNotif`、`MLEventNotif`與 `mLFileAddr.mLModelUrl` semantics
5. PyAnLF下載、驗證並成功載入後同步回成功；失敗使用 HTTP error/`ProblemDetails`
6. 第一版不建立 custom `ModelReady`、base/target generation CAS或多狀態 apply-result protocol
7. training job identity與 retry state留在 PyMTLF內部，不穿透 Go boundary

### 3.7 Backend Availability

1. Go不因 backend尚未啟動而停止整個 NWDAF
2. Go對 AnLF backend與 MTLF backend分別 polling `/health/ready`
3. readiness成功後仍須完成必要 handshake，才將 backend標為 usable
4. backend configured但 unavailable時，需要該 backend的新 operation回標準 `503 ProblemDetails`
5. backend disabled時，不 advertise依賴該 backend的 service/capability
6. MTLF backend unavailable不應中止已由 AnLF backend和現有模型提供的 analytics

---

## 4. Standards And Local Sources

contract與 procedure的優先來源為：

1. current `NWDAF/` implementation與實際 dependency version
2. `nwdaf-docs/specs/openapi/TS29520_Nnwdaf_EventsSubscription.yaml`
3. `nwdaf-docs/specs/openapi/TS29520_Nnwdaf_MLModelMonitor.yaml`
4. `nwdaf-docs/specs/openapi/TS29520_Nnwdaf_MLModelProvision.yaml`
5. `nwdaf-docs/specs/openapi/TS29520_Nnwdaf_MLModelTraining.yaml`
6. `nwdaf-docs/specs/openapi/TS29575_Nadrf_DataManagement.yaml`
7. local TS 29.520、TS 29.575與 TS 29.500 text
8. current `PyAnLF/`與 `PyMTLF/` implementation
9. local free5GC reference tree

local OpenAPI corpus仍缺部分 external `$ref`，包括 TS 29.576的 exact `FetchInstruction` attachment。
feature phase開始前必須確認 required generated types是否已存在。若 current free5GC OpenAPI module缺少
Release 18 type，應優先採可重現的 scoped generation；若 generation被缺失依賴阻擋，必須記錄 exact
schema來源與 isolated compatibility type，不能散落手寫 struct或改用 untyped map。

free5GC exemplar使用原則：

- NRF/UDM：Go app、service、consumer、lifecycle與 injected client shape
- PCF/UDR：callback、subscription與 persistence-coupled notification shape
- UDR：Mongo ownership、indexes與 document persistence boundary

free5GC沒有 Python backend或 PyMTLF direct ADRF/Mongo data-path exemplar；這些部分是本專案明確
architecture decision，不宣稱為 upstream precedent。

---

## 5. Target Architecture

### 5.1 Standard SBI Routing

```text
External NF / ADRF
        |
        | standard SBI
        v
NWDAF Go
  - parse and validate OpenAPI payload
  - authentication, NRF, OAuth and TLS
  - resource/callback correlation
  - route operation to logical backend
  - rewrite external Location/callback URI
        |
        +------> AnLF backend
        `------> MTLF backend
```

Go不是完全 stateless reverse proxy。它仍擁有 NF identity、external resource URI、callback ingress、
standard client與最小 routing state；但不再執行 AnLF/MTLF policy、training或 analytics business logic。

### 5.2 Operation Routing Matrix

| Standard operation | Domain owner / routing target |
|---|---|
| `Nnwdaf_EventsSubscription` analytics behavior | AnLF backend |
| `Nnwdaf_MLModelMonitor /registrations` | MTLF backend |
| `Nnwdaf_MLModelMonitor /subscriptions` | AnLF backend |
| `Nnwdaf_MLModelMonitor_Notify` | MTLF backend |
| `Nnwdaf_MLModelProvision /subscriptions` | MTLF backend |
| ML model provision notification callback | AnLF backend |
| `Nnwdaf_MLModelTraining` | MTLF backend |
| ADRF retrieval subscribe/unsubscribe | Go standard client, requested by MTLF backend |
| ADRF retrieval notification | Go callback ingress, forwarded to MTLF backend |
| ADRF data-store-record fetch | MTLF backend direct to ADRF |

`Nnwdaf_MLModelMonitor`同一 service的 operation會依 AnLF/MTLF role routing到不同 backend；不得以
service name直接假設單一 owner。

### 5.3 ADRF Direct Fetch Flow

```text
PyMTLF -- NadrfDataRetrievalSubscription --> Go -- POST subscription --> ADRF

ADRF -- NadrfDataRetrievalNotification/FetchInstruction --> Go --> PyMTLF

PyMTLF -- GET fetchUri?fetch-correlation-ids=... ---------------------> ADRF
PyMTLF <--------------------------- NadrfDataStoreRecord ------------- ADRF

PyMTLF -- retrieval complete --> Go -- DELETE subscription ----------> ADRF
```

PyMTLF在 wire上是 containing NWDAF的 internal component，不是另一個 NF。若 deployment啟用 OAuth，
direct fetch必須使用代表同一 NWDAF的短期 access token；第一版 local deployment可使用 OpenAPI允許的
unauthenticated mode，但必須標記 deployment limitation。

### 5.4 Mongo Direct Read Flow

```text
UPF notification --> Go --> MongoDB raw notification collection
                                  ^
                                  |
                          direct read-only query
                                  |
                               PyMTLF
```

Go不提供 Mongo query API，PyMTLF也不寫入或管理 collection。Mongo schema是兩個 repository間的
storage contract，但第一版不增加 schema version或 migration framework。

### 5.5 Accuracy And Model Flow

```text
PyAnLF -- MLModelMonitorNotify --> Go --> PyMTLF accuracy policy

PyMTLF -- direct ADRF/Mongo data --> local training --> publish model URL

PyMTLF -- NwdafMLModelProvNotif --> Go --> PyAnLF
PyAnLF -- download and load --> HTTP success/problem --> Go --> PyMTLF
```

第一版以標準 body與同步 HTTP結果完成流程，不建立額外 distributed transaction protocol。

---

## 6. Minimal Backend Contracts

### 6.1 Standard-shaped Feature Contracts

下列 payload應直接對應 local Release 18 OpenAPI：

- `NnwdafEventsSubscription`
- `MLModelMonitorReg`
- `MLModelMonitorSub`
- `MLModelMonitorNotify`
- `MLModelAccuracyInfo`
- `NwdafMLModelProvSubsc`
- `NwdafMLModelProvNotif`
- `MLEventNotif`
- `NadrfDataRetrievalSubscription`
- `NadrfDataRetrievalNotification`
- `FetchInstruction`
- `NadrfDataStoreRecord`
- `ProblemDetails`

private backend path可加 internal prefix，但 request/response body不得重新命名標準欄位。

### 6.2 Required Private Contracts

第一版只保留：

1. liveness/readiness endpoint
2. backend connection/storage requirements handshake
3. transport-level routing correlation，僅在無法由 standard resource ID表達時使用

建議 handshake語意：

```json
{
  "availableDataSources": ["adrf", "mongodb"]
}
```

```json
{
  "storageMode": "mongodb"
}
```

exact path與 error shape由對應 feature detailed plan固定；不預先建立通用 RPC framework。

### 6.3 Removed Contract Families

下列舊設計不再實作：

- custom accuracy report envelope與 parallel model identity
- dataset request/chunk/completion API
- provider attempt ID與 fallback chunk protocol
- Go-owned normalized training observation
- custom `ModelReady`
- base/target generation CAS
- `APPLIED`、`FAILED`、`STALE`、`NO_MATCH`、`CONFLICT` apply result
- Go/PyMTLF active-generation reconciliation API

若後續實測證明標準 contract無法表達必要 behavior，必須帶具體 blocker重新決策，不能先保留這些
複雜 contract作為預防性 abstraction。

---

## 7. Backend Polling And Handshake

每個 backend獨立維護：

```text
UNKNOWN -> POLLING -> READY_CHECK_PASSED -> HANDSHAKING -> USABLE
                    ^                                  |
                    `----------- UNAVAILABLE <---------'
```

要求：

1. Go啟動後立即 probe一次
2. failure採 bounded exponential backoff，例如 1、2、5、10、30 seconds並加入 jitter
3. readiness成功不等於 usable；MTLF backend還要完成 storage handshake
4. live operation transport failure立即將對應 backend標為 unavailable
5. polling與 shutdown使用 app-owned context，不留下 detached goroutine
6. state transition留下結構化 log；第一版不要求額外 distributed health store
7. request path讀取 cached readiness state，不在每次 consumer request同步做 health network call

Phase 2只協商並快取MTLF storage mode；尚未有data-dependent MTLF feature，因此不切換既有UPF writer。
Phase 5接入raw-notification writer後，Go必須先完成write sink安全切換，再讓data-dependent operation使用
新的mode。

---

## 8. Mongo Raw Notification Contract

第一版 document以一個原始 notification item為單位，概念形狀如下：

```json
{
  "correlationId": "...",
  "receivedAt": "...",
  "measurementTime": "...",
  "notificationItem": {}
}
```

規則：

1. `notificationItem`保存收到的原始 standard-oriented JSON
2. `measurementTime`優先使用 notification item的 measurement/start time，fallback必須明列
3. indexed metadata不得演變成第二份 normalized feature record
4. Mongo write failure記錄並暴露 health/metric，不阻止 Go完成已獨立成功的其他 sink write
5. PyMTLF query使用 bounded batch/cursor，不一次無限制 materialize整個 collection
6. retention由 deployment/config設定；第一版不建立 automatic schema migration
7. Go read-write與 PyMTLF read-only credential分離

minimum indexes應依實際 query plan在 detailed phase確認，至少評估：

- `measurementTime`
- `correlationId`
- `notificationItem.supi`
- `notificationItem.dnn`

---

## 9. Direct ADRF Fetch Controls

PyMTLF direct fetch至少必須：

1. 驗證 `fetchUri`為允許的 HTTP(S) URI；production預設 HTTPS
2. 驗證 origin符合 Go傳入或 deployment設定的 ADRF allowlist
3. 拒絕導向非信任 origin的 redirect
4. 檢查 `FetchInstruction.expiry`
5. 只使用 instruction提供的 `fetchCorrIds`
6. 使用 bounded response size、request timeout與 retry
7. 正確處理 `200 NadrfDataStoreRecord`、`204`與 standard `ProblemDetails`
8. 不將 fetch URI或 credential寫入一般 log

OAuth token delegation不是第一版 local completion條件，但 client boundary不得把 long-lived NWDAF private
credential寫進 PyMTLF config。啟用 production OAuth前需建立短期 token取得方式與 integration test。

---

## 10. Simplified Model Lifecycle

第一版只要求：

1. PyMTLF完成 local training並產生 PyAnLF-compatible bundle
2. artifact原子發布到 MTLF backend private artifact endpoint
3. PyMTLF配置 `modelUniqueId`
4. PyMTLF建立標準 `MLEventNotif`，以 `mLFileAddr.mLModelUrl`指向 artifact
5. Go驗證並 routing notification到 AnLF backend
6. PyAnLF同步下載、檢查 bundle限制、完整載入後才切換 runtime model
7. PyAnLF成功回 HTTP success，失敗回 `ProblemDetails`

保留 content-addressed artifact storage與 PyAnLF安全下載限制。artifact digest可存在 URL identity、ETag或
bundle manifest，不要求另建非標準 provision欄位。

existing SQLite generation journal與reconciliation primitives只服務已取消的generation/apply protocol，
且沒有新的production consumer。已決定在Phase 2連同專用models、config與tests移除。未來若local training
需要job persistence，必須依實際job semantics建立新的小型內部儲存，不復用舊state machine。

---

## 11. Feature-oriented Migration Phases

每個 phase開始前建立 detailed plan，列出 exact routes、OpenAPI types、repository changes、cutover與 tests。

### Phase 1: Foundation Baseline And Replan Record

狀態：已建立並驗證 baseline；新架構下只選擇性保留。

保留：

- PyMTLF repository、Python package、config、logging、health與 lifecycle
- MTLF backend artifact repository與 PyAnLF bundle consumer hardening
- Go `mtlfBackend` config與 readiness client基礎
- PyAnLF/PyMTLF Ruff與 tests

不再作為 target requirement：

- durable generation/apply state machine
- startup reconciliation protocol
- 舊 future custom contract inventory

詳細紀錄見 Phase 1文件。

### Phase 2: Backend Connectivity And Standard Contract Foundation

Feature goal：Go可靠連接兩個 backend，建立 polling、MTLF storage handshake與 Release 18 contract basis。

包含：

- AnLF/MTLF backend polling、cached state、backoff與 shutdown
- MTLF available-source handshake、source inventory與 negotiated storage mode cache
- 移除PyMTLF舊SQLite generation journal、reconciliation與專用code/tests/config
- configured/disabled/unavailable service behavior與 `503 ProblemDetails`
- Release 18 OpenAPI type gap audit與 scoped generation/compatibility strategy
- standard body不經parallel DTO translation的transport policy與後續feature test requirements
- Go naming boundary與 backend-independent client package ownership

完成條件：backend restart後 Go可恢復 polling、重新 handshake並正確 gate已接入backend的operation；
raw storage sink切換仍由Phase 5完成。

### Phase 3: Analytics Subscription Routing

Feature goal：將 AnLF相關 analytics subscription behavior從 Go domain logic收斂到 AnLF backend。

包含：

- `Nnwdaf_EventsSubscription` create/update/delete的 standard-shaped backend route
- Go保留 external URI、validation、data collection consumer與標準 response
- PyAnLF擁有 runtime creation、report scheduling、analytics shaping與 subscription state
- backend unavailable的 create/update rollback與 `503`
- existing AnLF custom runtime API與 contract依 cutover結果縮減或移除

完成條件：同一份 standard subscription可經 Go routing由 PyAnLF建立、更新、通知與刪除，且無雙重 owner。

### Phase 4: ML Model Monitoring And Accuracy Policy

Feature goal：以 `Nnwdaf_MLModelMonitor` standard shape完成 AnLF accuracy measurement到 MTLF policy。

包含：

- `/registrations` routing到 MTLF backend
- `/subscriptions` routing到 AnLF backend
- PyAnLF產生 `MLModelMonitorNotify`
- Go callback/notification routing
- PyMTLF接收 `MLModelAccuracyInfo`並執行簡單 accuracy/retrain policy
- 移除 custom model-accuracy-report contract與 Go accuracy policy

完成條件：accuracy資訊從 PyAnLF到 PyMTLF全程使用標準欄位，Go不執行 policy。

### Phase 5: Training Data Storage And Direct Retrieval

Feature goal：完成 PyMTLF direct ADRF/Mongo data path。

包含：

- Go raw UPF notification Mongo write與 indexes
- storage mode bootstrap、handshake apply與 dual-write
- PyMTLF read-only Mongo query
- PyMTLF建立 `NadrfDataRetrievalSubscription` request
- Go ADRF subscribe/callback/unsubscribe lifecycle
- 完整 `FetchInstruction` forwarding
- PyMTLF direct ADRF fetch、security limits與 response parsing
- source selection與 retry留在 PyMTLF
- 移除 Go retrain retrieval job、dataset upload與 provider/chunk design

完成條件：ADRF與 Mongo mode各自完成真實 process-level retrieval test；Go不接收 dataset bytes。

### Phase 6: Local Training And ML Model Provision

Feature goal：由 PyMTLF完成 local training並以標準 model provision flow交付 PyAnLF。

包含：

- 簡單 Python trainer與 preprocessing seam
- candidate evaluation最低必要條件
- artifact publish與 `modelUniqueId`
- `NwdafMLModelProvSubsc` resource behavior routing到 MTLF backend
- `NwdafMLModelProvNotif`/`MLEventNotif`經 Go routing到 AnLF backend
- PyAnLF同步 download/load與 HTTP success/`ProblemDetails`
- URL allowlist、size、timeout、safe extraction與 cache integrity
- 移除 custom ModelReady/generation/apply protocol assumptions

完成條件：PyMTLF以 ADRF或 Mongo資料訓練後，PyAnLF可載入模型並用於既有 analytics runtime。

### Phase 7: Legacy Removal And Closure

Feature goal：移除 transition後不再使用的 Go MTLF、Daisy與 custom backend paths。

包含：

- 移除 Daisy client、callback、config、tests與 dependency
- 移除 Go accuracy policy、training scheduler、dataset provider與 model apply coordinator
- 移除被 standard-shaped routes取代的 PyAnLF/PyMTLF private APIs
- NRF service/capability advertisement audit
- full cross-repository、restart、failure與 no-legacy-name audit

完成條件：production dependency graph、config與 source不再包含 Daisy，且每個 feature只有一個 owner。

---

## 12. Verification Strategy

### 12.1 Contract Verification

- request/response JSON對照 local Release 18 OpenAPI required/optional fields
- standard enum、oneOf/anyOf與 `ProblemDetails` tests
- Go/backend round-trip不得重命名或丟失標準欄位
- current free5GC generated model與 scoped compatibility type差異有明確 tests

### 12.2 Go Verification

- focused processor/consumer/client tests
- polling、backoff、reconnect、handshake與 shutdown race tests
- callback correlation、Location rewrite與 backend routing tests
- Mongo raw persistence/index query tests
- `make test`、`make build`與 `make lint`

### 12.3 Python Verification

- PyAnLF/PyMTLF unit、API與 Ruff
- direct ADRF fetch URI/expiry/redirect/timeout/size tests
- Mongo read-only query/batching tests
- local trainer deterministic smoke
- artifact publish、download與load tests

### 12.4 Cross-process Verification

至少覆蓋：

1. Go先啟動、backend後啟動與 polling recovery
2. backend restart後重新 handshake
3. AnLF-only analytics在 MTLF unavailable時繼續服務
4. ADRF subscribe -> callback -> PyMTLF direct fetch -> unsubscribe
5. Mongo raw write -> PyMTLF direct query
6. accuracy notify -> retrain -> model provision -> PyAnLF load
7. disabled、unavailable、timeout與 malformed standard payload failures

完整 OAuth、NRF discovery與 production TLS若未在本地執行，必須明確列為未驗證，不得以 unit test代替。

---

## 13. Explicitly Deferred

本 transition第一版不包含：

- PyAnLF或 PyMTLF成為獨立 NF
- multiple-NWDAF federated learning
- centralized FL/Daisy compatibility
- automatic ADRF-to-Mongo failover without dual-write
- Mongo schema migration/version framework
- distributed training job transaction與跨程序 exactly-once guarantee
- custom generation CAS與 multi-status apply reconciliation
- artifact signing或 untrusted third-party model execution
- production OAuth token delegation implementation，除非對應 phase另行納入

---

## 14. Decision And Replan Gates

以下情況必須停止並重新確認：

1. local OpenAPI缺失使標準 payload無法可靠實作
2. ADRF要求的 authentication無法安全委派給 PyMTLF direct fetch
3. raw Mongo notification無法支援已確認的 MTLF query scope或 index
4. 標準 model provision同步 HTTP結果不足以保證 PyAnLF安全切換
5. backend polling/handshake與 NRF advertisement產生規格衝突
6. feature cutover需要同時保留兩個 production owner
7. 移除 legacy path會破壞尚未遷移的 analytics feature

遇到上述情況不得以恢復舊的 generic chunk/provider/generation framework作為默認 workaround。

---

## 15. Completion Criteria

整體 transition完成時：

1. Go主要執行 standard SBI、routing、callback、NF identity與必要 storage write
2. AnLF/MTLF feature behavior分別由 PyAnLF/PyMTLF擁有
3. 標準 feature的 backend payload與 Release 18 OpenAPI shape一致
4. PyMTLF可直接從 ADRF或 MongoDB取得 training data
5. Go不代理 dataset bytes或執行 training policy
6. PyMTLF完成 local training並以標準 model provision shape提供模型
7. PyAnLF成功載入模型並繼續提供 analytics
8. backend unavailable時 Go polling並對需要的 operation回正確錯誤
9. 沒有並行的 Go/Python owner、dead custom API或 speculative contract
10. NWDAF、PyAnLF與 PyMTLF沒有 Daisy runtime字樣或相依性
11. PyMTLF沒有舊generation journal、reconciliation engine或其專用state models
12. 三個 implementation repositories各自通過 tests/lint/build與 cross-process verification
13. deferred OAuth/TLS/NRF或環境級驗證限制有明確紀錄

---

## 16. Relationship To Prior Plans

AnLF backend transition已完成的 ownership經驗仍可作為 migration與 test參考，但其現有 private contract
不是永久限制。本計畫允許在對應 standard feature phase修改：

- `PyAnLF/`
- `NWDAF/internal/anlf/`
- `NWDAF/internal/sbi/`
- shared Go wiring、config與 tests

舊 MTLF backend plan中與 Go-owned data provider、chunk delivery、custom generation/apply state machine有關的
內容由本文件取代。Phase 1文件只保留已提交 foundation的歷史與可重用部分，不再作為 future contract規格。
