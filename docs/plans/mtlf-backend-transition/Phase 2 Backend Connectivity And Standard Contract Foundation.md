# Phase 2 Backend Connectivity And Standard Contract Foundation

Date: 2026-07-18

Status: Draft for review; implementation not started

Parent plan:

- `nwdaf-docs/docs/plans/mtlf-backend-transition/MTLF Backend Transition Plan.md`

Previous phase record:

- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 1 PyMTLF Foundation And Backend Boundary.md`

---

## 1. Purpose

Phase 2 建立 Go NWDAF 與 AnLF backend、MTLF backend 之間可長期使用的連線與 contract foundation。
本階段要讓 NWDAF 可以先啟動、獨立追蹤兩個 backend、在 backend 晚啟動或重啟後自動恢復，並讓
後續 feature phase 能以一致方式判斷 backend 是否可用。

本階段同時固定兩項後續不能再各自發明的基礎規則：

1. 標準 feature payload 直接沿用 Release 18 OpenAPI JSON shape，不建立平行 business DTO
2. MTLF backend ready 後，以一個最小 private handshake 選擇 `adrf`、`mongodb` 或 `dual`

Phase 2 不遷移 analytics subscription、accuracy policy、ADRF retrieval、raw Mongo storage 或 model
provision procedure。這些功能仍由 Phase 3 至 Phase 6 依 feature 完成 vertical slice。

---

## 2. Entry Baseline

### 2.1 Repository Baseline

撰寫本計畫時，四個 repository worktree 均無未提交變更：

| Repository | Baseline commit | Branch state |
|---|---|---|
| `NWDAF/` | `4b5e114f25b718003e3d248551f2d52fa86a6cf6` | `refactor/free5gc-alignment`, ahead of remote by 1 |
| `PyAnLF/` | `fac8fab52a116e413a36abbb03e69c81b954e4ef` | `master`, ahead of remote by 1 |
| `PyMTLF/` | `8c55a965352ff3a70c31adce4a2e8640d8c8b4a8` | `main`, no upstream shown locally |
| `nwdaf-docs/` | `d88737c8aeaa892ef1dce8c64e346994ff1960cd` | `main`, ahead of remote by 2 |

各 repository 必須維持獨立 commit。本 phase 不得把跨 repository 變更合併成單一 commit。

### 2.2 Existing Implementation

目前已存在：

- Go `anlfBackend` config與既有 private AnLF client，但沒有 health client或 polling lifecycle
- Go `mtlfBackend` config與單次 `GET /health/ready` client，但尚未接入 `NwdafApp`
- PyAnLF private runtime API，default listener為 `127.0.0.1:9093`，但沒有 health endpoint
- PyMTLF `GET /health/live`、`GET /health/ready`與 artifact endpoint
- Go啟動時只在 startup ping MongoDB，沒有 backend availability manager
- NRF profile目前固定只宣告 `Nnwdaf_EventsSubscription`
- current Go MTLF/Daisy flow仍是 production path，新 MTLF backend尚未接管 feature

### 2.3 Existing Storage Behavior

現有 UPF notification path同時包含：

- 將每個 notification item送入 ADRF buffer
- 將每個 measurement轉成 `UpfTrafficRecord`後寫入 MongoDB time-series collection

這不是 target raw-notification Mongo contract。Phase 2 不修改此 write path，原因是 storage document、
index、dual-write與 PyMTLF direct query必須在 Phase 5一起 cut over，避免先用 negotiated mode切換到仍不相容
的舊 Mongo schema。

### 2.4 free5GC Exemplar Evidence

Phase 2以local free5GC reference tree中的NRF作為primary exemplar、UDM作為secondary exemplar：

- `resources/references/free5gc-main/NFs/nrf/pkg/service/init.go`
  - 直接證據：app-owned `context.CancelFunc`、`sync.WaitGroup`、server startup與shutdown ownership
- `resources/references/free5gc-main/NFs/nrf/pkg/app/app.go`
  - 直接證據：service與internal package之間維持窄app interface
- `resources/references/free5gc-main/NFs/udm/pkg/service/init.go`
  - 直接證據：consumer、processor、server由service組裝，external call不在handler任意建立
- `resources/references/free5gc-main/NFs/udm/internal/sbi/consumer/consumer.go`
  - 直接證據：outbound service client集中在consumer boundary

free5GC reference沒有internal Python backend、backend readiness poller或storage handshake precedent。因此
polling state machine、MTLF handshake與Go/Python boundary是本專案architecture decision；只借用app lifecycle、
consumer ownership與shutdown discipline，不宣稱exact upstream pattern。

---

## 3. Phase Outcomes

Phase 2完成後應具備：

1. PyAnLF與PyMTLF都有一致的 liveness/readiness boundary
2. Go以 app-owned worker獨立poll兩個backend
3. request path只讀cached availability，不同步執行health call
4. backend晚啟動、短暫故障或重啟後可自動恢復
5. MTLF每次成功readiness probe後完成storage-mode handshake
6. Go可區分disabled、configured-but-unavailable與usable
7. 需要backend的既有operation可產生標準`503 ProblemDetails`
8. NRF advertisement以config capability決定，不以瞬時readiness動態抖動
9. Release 18與current free5GC OpenAPI type gap有明確處理策略
10. PyMTLF不再包含舊SQLite generation journal、reconciliation或其專用state models
11. 後續feature可直接使用同一availability與transport boundary，不必再建立第二套poller

---

## 4. Non-goals

Phase 2不包含：

- 將`Nnwdaf_EventsSubscription` body與owner遷移到PyAnLF
- 實作`Nnwdaf_MLModelMonitor`、`Nnwdaf_MLModelProvision`或`Nnwdaf_MLModelTraining`route
- 將accuracy policy移到PyMTLF
- 由PyMTLF建立ADRF retrieval subscription
- forwarding `NadrfDataRetrievalNotification`或`FetchInstruction`
- PyMTLF direct ADRF fetch或Mongo query
- 將Mongo schema改成raw UPF notification item
- 依handshake實際切換UPF data write sink
- 移除Daisy、舊Go MTLF scheduler或舊ADRF retrieval flow
- 建立production OAuth delegation或TLS integration
- 產生尚無production consumer的完整Release 18 client/server套件

---

## 5. Confirmed Design Decisions

### 5.1 Process And Naming Boundary

1. PyAnLF與PyMTLF仍是containing NWDAF的internal backend
2. backend不向NRF註冊，也不advertise標準service
3. Go code只出現`anlfBackend`、`mtlfBackend`或backend-neutral命名
4. Go source、config、logs與tests不得出現Python implementation name
5. Python app title、package與repository可以保留`PyAnLF`、`PyMTLF`命名

### 5.2 Availability Is Local Runtime State

backend connection state屬於Go app-owned runtime，不寫入MongoDB、不放入distributed health store，也不
放進factory config。monitor自己以mutex保護snapshot；processor與coordinator只透過窄介面讀取狀態。

### 5.3 Readiness And Usability Are Different

- AnLF backend：readiness成功即為usable
- MTLF backend：readiness成功後還必須完成storage handshake才是usable
- PyMTLF自身readiness不依賴Go是否完成handshake，避免互相等待
- request path不得以一次臨時health request取代cached state

### 5.4 Configured But Unavailable Does Not Stop NWDAF

Go在backend尚未啟動時仍完成Mongo initialization、NRF registration與SBI listener startup。需要該
backend的operation回`503 ProblemDetails`；不需要該backend的operation繼續服務。

### 5.5 Advertisement Follows Configuration, Not Transient Health

- backend disabled：不advertise依賴該backend且已實作完成的service/capability
- backend configured但暫時unavailable：仍advertise，request進入後回503，讓late-start recovery成立
- backend readiness每次轉換不觸發NRF profile update
- 尚未在Go完成標準route的MTLF service，即使`mtlfBackend.enabled=true`也不在Phase 2提前advertise

Phase 2只需使現有`Nnwdaf_EventsSubscription` advertisement受`anlfBackend.enabled`控制。ML Model
Monitor、Provision與Training由各自feature phase在route完成時加入profile。

---

## 6. Backend Connection State Machine

### 6.1 States

每個configured backend使用獨立monitor：

```text
UNKNOWN
   |
   v
POLLING -- readiness failure --------------------> UNAVAILABLE
   |
   +-- AnLF readiness success -------------------> USABLE
   |
   `-- MTLF readiness success --> HANDSHAKING ---+--> USABLE
                                      |
                                      `----------+--> UNAVAILABLE

USABLE -- readiness/handshake/transport failure -> UNAVAILABLE
UNAVAILABLE -- next poll ------------------------> POLLING
```

disabled backend不建立monitor，以`DISABLED`作為config-derived狀態回報；`DISABLED`不是worker state。

### 6.2 Snapshot

monitor至少提供不可變snapshot：

- state
- last successful probe time
- last failure time
- sanitized failure category
- MTLF selected storage mode；AnLF為空

snapshot不得保存response body、backend URL credential或完整network error chain供外部API輸出。

### 6.3 Polling Schedule

1. worker啟動後立即probe，不先sleep
2. failure retry使用`1s, 2s, 5s, 10s, 30s`上限序列
3. 每個delay加入最多正負20%的jitter，避免兩個backend同步重試
4. 成功後每30秒重新probe
5. 任一failure後從1秒retry重新開始
6. probe與handshake使用backend的request timeout
7. sleep、network call與handshake都必須受`NwdafApp` context cancellation控制
8. ticker/timer由worker建立並負責停止
9. worker由`NwdafApp.wg`追蹤，shutdown不得留下goroutine

backoff sequence與jitter source在production使用固定internal default；測試以constructor option注入clock、
delay與random source，不新增大量YAML tuning欄位。

### 6.4 Periodic MTLF Handshake

每次MTLF readiness probe成功後都重新執行handshake，而不是只在Go觀察到backend down/up transition時
執行。這可處理PyMTLF在兩次30秒probe之間快速重啟、使Go沒有觀察到readiness failure的情況。

若handshake回傳的mode與cached mode不同，Phase 2只原子更新connection snapshot。Phase 5接入write
sink後，必須先完成sink切換再publish新mode為usable；Phase 2不得假裝舊write path已受mode控制。

### 6.5 Live Operation Failure

backend client需分類：

- network error、timeout、connection reset與backend 5xx：標記connection unavailable
- request validation、not found、conflict等明確4xx：回傳feature error，不將整個backend判定為down
- parent context cancellation：保持cause，不誤報為backend failure

Phase 2將這個reporting seam接到目前已存在的AnLF operation；後續每個feature phase接入新route時必須
沿用。MTLF新feature尚未啟用，因此本階段只由probe/handshake更新MTLF state。

---

## 7. Private Health Contract

### 7.1 Routes

兩個backend都提供：

| Method | Path | Meaning |
|---|---|---|
| `GET` | `/health/live` | process與HTTP event loop仍可回應 |
| `GET` | `/health/ready` | backend已完成本地初始化，可接受其private feature request |

這些route是internal deployment contract，不是3GPP SBI。

### 7.2 Success Shape

readiness最低成功body為：

```json
{
  "status": "ready"
}
```

liveness成功body為`{"status":"live"}`。

backend可以增加本地component status供診斷，但Go只以HTTP 200判斷readiness，不依賴Python-specific欄位。

### 7.3 Failure Shape

- not ready回HTTP 503
- response body保持bounded且不得暴露exception stack、path、credential或artifact URL
- Go最多讀64 KiB health response
- malformed或oversized response視為probe failure

### 7.4 PyAnLF Readiness

PyAnLF ready代表：

- `ModelManager`成功建立
- `SubscriptionRuntimeManager`成功建立
- app未進入shutdown

沒有active model不代表process not ready；subscription可能合法等待後續model provision。readiness不得為了
檢查模型而執行download或prediction。

### 7.5 PyMTLF Readiness

PyMTLF ready代表：

- artifact repository可open與probe
- app未進入shutdown

既有SQLite generation journal與reconciliation只服務已取消的generation/apply protocol，沒有Phase 2或
後續已確認流程的production consumer，因此本phase直接移除，不保留為readiness diagnostic。PyMTLF
readiness response也不再包含database或reconciliation欄位。

現有code audit確認這些元件的參照只存在於app lifecycle、health、config、專用core modules、README與
tests；accuracy、training、artifact publish與model provision route都沒有將它們當作business dependency。

未來若local training需要job persistence，應以training job identity、retry與artifact publish的實際需求
建立新的小型internal store；不得恢復舊base/target generation與apply reconciliation state machine。

---

## 8. MTLF Storage-mode Handshake

### 8.1 Route

```text
POST /internal/v1/data-source-selection
```

此route只存在於MTLF backend。它是最小private contract，不是標準NF service，也不使用standard
service root。

### 8.2 Request

```json
{
  "availableDataSources": ["adrf", "mongodb"]
}
```

規則：

1. allowed values只有`adrf`與`mongodb`
2. array順序沒有語意；Go固定以`adrf`、`mongodb`順序送出
3. 不允許duplicate或unknown value
4. 空array是valid transport shape，但無法滿足任何mode，應得到409
5. request不包含Mongo URI、credential、ADRF fetch URI或standard payload

### 8.3 Response

```json
{
  "storageMode": "mongodb"
}
```

allowed values為`adrf`、`mongodb`與`dual`。PyMTLF依自己的`data_source.storage_mode`設定選擇；第一版
sample/default設為`mongodb`，符合目前local-training transition。deployment要使用ADRF或runtime
fallback時必須明確設定`adrf`或`dual`，不做silent substitution。

### 8.4 Satisfaction Rules

| PyMTLF configured mode | Required Go sources |
|---|---|
| `adrf` | `adrf` |
| `mongodb` | `mongodb` |
| `dual` | `adrf`及`mongodb` |

required source缺少時：

- PyMTLF回HTTP 409
- private error code為`DATA_SOURCE_REQUIREMENT_UNSATISFIED`
- `retryable`為`true`
- Go保持MTLF connection unavailable並依backoff重試
- Go不得改選其他mode

### 8.5 Source Availability Definition In Phase 2

- `adrf`：`adrf.url`已通過config validation並已建立ADRF consumer client
- `mongodb`：Mongo config存在，且Go在本次handshake前以bounded context執行的live ping成功

ADRF沒有本專案可依賴的standard health operation，因此Phase 2不增加non-standard ADRF probe。`adrf`
在handshake中的available代表Go具備configured standard client，不保證遠端在每次operation都存活。

每次MTLF readiness成功後，Go先重新計算source inventory；Mongo ping使用2秒internal timeout並受app
context cancellation控制，不新增獨立Mongo polling goroutine。Mongo晚啟動或恢復後，下一次handshake即可
重新加入`mongodb`；Mongo失聯時也會從該次request移除。這個檢查只判斷handshake capability，不執行
training query、不切換writer，也不宣告舊Mongo schema已符合target contract。

### 8.6 Bootstrap Storage Policy

主計畫中的bootstrap規則保留為Phase 5 write-path requirement：

- handshake前只有一個source可用時，未來writer寫該source
- 兩個source可用時，未來writer暫時dual-write
- handshake成功後，未來writer依selected mode切換

Phase 2只實作source inventory、selection與cached mode，不修改現有UPF writer。Phase 5必須在raw
notification schema完成後一次實作bootstrap、safe sink switch與dual-write。

---

## 9. Standard Error Boundary

### 9.1 External SBI

當operation需要configured但unusable的backend時，Go回：

```json
{
  "status": 503,
  "title": "Service Unavailable",
  "detail": "requested NWDAF capability is temporarily unavailable"
}
```

response使用current `models.ProblemDetails`與既有Gin problem helper。第一版不在external response加入
`ANLF_BACKEND_UNAVAILABLE`或`MTLF_BACKEND_UNAVAILABLE`等non-standard cause，也不暴露internal
component name；backend identity與failure category只留在structured log。因目前無法可靠預估恢復時間，
第一版不送`Retry-After`。

這裡沿用已確認的503方向，表示containing NWDAF暫時無法提供該capability；internal backend不是另一個
standard NF，因此不是直接relay另一個NF回傳的503。若後續標準feature對特定failure另有明確status/cause，
以該service OpenAPI與TS procedure為準。

### 9.2 Private Backend API

health與handshake可以沿用小型private error：

```json
{
  "code": "DATA_SOURCE_REQUIREMENT_UNSATISFIED",
  "message": "configured storage mode requires unavailable data source",
  "retryable": true
}
```

private error只描述transport/handshake問題。標準feature route在後續phase仍須使用`ProblemDetails`
semantics，不得因這個private error model而建立平行business error family。

---

## 10. Release 18 Contract Gap Audit

### 10.1 Current Go Dependency

`NWDAF/go.mod`目前使用`github.com/free5gc/openapi v1.2.3`。local module可確認：

| Contract family | Current generated model | Local Release 18 status | Phase 2 decision |
|---|---|---|---|
| Events Subscription | `models.NnwdafEventsSubscription`，來源為TS 29.520 V17.10.0 | V18.13.0 YAML存在 | Phase 3逐欄位比對後重用或補compatibility type |
| Problem Details | `models.ProblemDetails` | CommonData V18.12.0 YAML存在 | Phase 2直接重用 |
| ML Model Provision | `NwdafMlModelProvSubsc`、`NwdafMlModelProvNotif`、`MlEventNotif`，來源為V17.7.0 | V18.13.0 YAML存在，Release 18新增欄位 | Phase 6不得直接假設V17 type完整 |
| Fetch Instruction | `models.FetchInstruction`，來源為V17.9.0 | exact TS 29.576 attachment缺失，但29.575引用存在 | Phase 5先核對top-level `fetchUri`、`fetchCorrIds`、`expiry` |
| ML Model Monitor | 無對應generated models | V18.13.0 YAML存在 | Phase 4建立scoped Release 18 types |
| ADRF Retrieval | 無subscription/notification/store-record generated models | V18.11.0 YAML存在 | Phase 5建立scoped Release 18 types |
| ML Model Training | current module無完整Release 18 family | V18.14.0 YAML存在 | 對應feature實作時再建立 |

### 10.2 Local Corpus Limitation

local Release 18 OpenAPI attachments包含主要NWDAF與ADRF YAML，但仍缺多個external `$ref`，包括
TS 29.576 `FetchInstruction` attachment、TS 29.122 CommonData與部分其他NF API。Phase 2不以猜測內容
補齊整個dependency graph，也不從不同Release無紀錄地拼接generated code。

### 10.3 Compatibility Strategy

後續feature遵守：

1. 先比對current generated type與Release 18 YAML的required、optional、enum、oneOf/anyOf與JSON name
2. 完整相容時直接重用current generated model
3. 不完整時優先使用可重現的scoped generation
4. generation受缺失external schema阻擋時，只在一個明確的Release 18 compatibility package放必要type
5. compatibility type必須註明schema、version、缺失dependency與欄位來源
6. 不在handler、client或`internal/mtlf/contract`散落手寫struct
7. Go與Python各自使用typed model，但JSON field name與optionality必須相同
8. standard body不可先轉成另一份backend-specific business DTO
9. test payload使用inline builder或typed fixture；不新增production contract的`testdata/*.json`

Phase 2本身只新增實際被health、handshake與503使用的type，不預先生成Phase 4至Phase 6尚未被route
consume的models。

---

## 11. Repository Work Plan

### 11.1 NWDAF

預計修改：

- `pkg/factory/config.go`
  - 對`anlfBackend.endpoint`套用與MTLF一致的HTTP origin normalization
  - 為AnLF backend補`requestTimeout`，default/sample使用5秒
  - 保持poll backoff為internal default，避免增加不必要config
- `pkg/factory/config_test.go`
  - enabled/disabled、invalid origin、invalid timeout與normalization matrix
- `config/nwdafcfg.yaml`
  - 補AnLF request timeout
  - 移除Phase 1 only註解，改成target backend boundary說明
- `internal/backend/`
  - 新增backend-neutral monitor、state snapshot、backoff與availability error
  - package不得知道AnLF/MTLF business contract
  - tests覆蓋state、retry、jitter、cancellation、concurrent snapshot與shutdown
- `internal/anlf/client/client.go`
  - 使用validated origin與bounded request timeout
  - 新增`CheckReadiness`
  - 保持現有feature method，Phase 3才改standard-shaped route
- `internal/anlf/client/*_test.go`
  - health success、503、oversize、timeout與parent cancellation
- `internal/anlf/coordinator/`
  - 注入窄availability reporter/gate
  - unusable時不發live request；transport/5xx failure回報monitor
  - 保持現有external 503 rollback behavior
- `internal/mtlf/client/backend.go`
  - 保留現有readiness transport hardening
  - 新增`POST /internal/v1/data-source-selection`
  - small handshake request/response留在client package，不建立新的broad contract directory
- `internal/mtlf/client/*_test.go`
  - selection success、invalid mode、409、oversize、timeout與body close
- `pkg/service/init.go`
  - `NewApp`只construct clients/monitors，不做network call
  - `startRuntime`啟動兩個app-owned monitor workers
  - shutdown沿用app context與WaitGroup
  - MTLF source provider從validated ADRF config與每次handshake前的bounded Mongo ping建立
- `pkg/service/init_test.go`
  - backend absent不阻止app lifecycle
  - worker cancellation與WaitGroup completion
  - independent AnLF/MTLF state
- `internal/context/context.go`
  - NF profile construction改為config-capability aware
  - Phase 2只控制Events Subscription advertisement
- `internal/context/nrf_profile_test.go`
  - AnLF disabled/configured matrix
  - configured但unavailable仍advertise
  - MTLF services尚未實作時不得提前advertise
- `internal/sbi/processor/eventssubscription.go`
  - 使用共同且不洩漏internal backend identity的503 `ProblemDetails` helper
  - 不在本phase改subscription ownership或payload

如實作時發現`internal/backend/`只會成為單一client wrapper而無法同時服務兩個backend，必須停止重新
評估，不得為了符合檔名預先建立空泛interface。

### 11.2 PyAnLF

預計修改：

- `src/py_anlf/sbi/server.py`
  - 改用可測試lifespan state管理accepting requests
  - include health router
- `src/py_anlf/sbi/routers/health.py`
  - 新增`/health/live`與`/health/ready`
  - readiness只檢查manager初始化與shutdown state
- `tests/test_health_api.py`
  - startup ready、constructor failure、shutdown not ready與repeated lifecycle
- existing lifecycle tests
  - 確認health route不改既有runtime behavior

Phase 2不重寫PyAnLF models、不移除舊private runtime route，也不開始Events Subscription payload遷移。

### 11.3 PyMTLF

預計修改：

- `src/py_mtlf/config.py`
  - 新增`DataSourceSettings`
  - `storage_mode`只允許`adrf`、`mongodb`、`dual`
  - default為`mongodb`
  - 移除`StorageSettings.database_path`與`ReconciliationSettings`
- `config/config.yaml`
  - 明列`data_source.storage_mode: mongodb`
  - 移除`storage.database_path`與`reconciliation`section
- `src/py_mtlf/api/data_source.py`
  - 實作selection route與satisfaction validation
- `src/py_mtlf/app.py`
  - include route
  - runtime保存目前selected mode
  - handshake可安全重複執行
  - 移除journal/reconciliation construction、startup task、shutdown與dependency injection
- `src/py_mtlf/api/health.py`
  - readiness以artifact與shutdown state判斷
  - response移除database與reconciliation diagnostic欄位
- `src/py_mtlf/core/generation_journal.py`
  - 刪除整個舊generation/apply journal
- `src/py_mtlf/core/reconciliation.py`
  - 刪除整個舊cross-process reconciliation engine
- `src/py_mtlf/models.py`
  - 刪除只由journal/reconciliation使用的`ArtifactRecord`、`ApplyStatus`、`ApplyEvidence`、
    `ObservedModelStatus`與`ObservedModelState`
  - 保留`ModelIdentity`、`PrivateError`、`SHA256_PATTERN`與artifact repository實際仍使用的types
- `tests/test_data_source_api.py`
  - 三種mode、missing source、duplicate/unknown/empty source與repeat handshake
- `tests/test_config.py`
  - default、invalid mode與已移除config key rejection
- `tests/conftest.py`
  - 移除舊SQLite path與reconciliation fixture setup
- `tests/test_health_and_artifact_api.py`
  - health response不再包含journal/reconciliation狀態
- `tests/test_generation_journal.py`
  - 隨production code刪除
- `tests/test_reconciliation.py`
  - 隨production code刪除
- `tests/test_models.py`
  - 移除舊state-machine assertions，保留仍使用model的validation
- `README.md`
  - 修正舊的「Go owns all ADRF communication」敘述
  - 說明Go owns standard subscribe/callback、PyMTLF future direct fetch的邊界
  - 不再將durable generation journal或startup reconciliation列為foundation capability

Phase 2不加入Mongo driver、ADRF HTTP client、training policy或新的standard feature model。

### 11.4 Documentation

- 完成後更新本文件status、實際files、commits與verification evidence
- 若實作結果改變canonical architecture，同步更新parent plan
- 不建立與本文件重複的contract inventory文件

---

## 12. Implementation Sequence

### Step 1: Lock Health Semantics

1. PyAnLF加入health lifecycle
2. PyMTLF移除superseded journal/reconciliation及專用config/models/tests
3. PyMTLF readiness收斂為artifact與app lifecycle
4. 各自完成Python API tests與Ruff

這一步不需要Go即可驗證，先固定poll target。

### Step 2: Build Backend-neutral Monitor

1. 以fake probe建立state/backoff/cancellation tests
2. 完成monitor與snapshot
3. 驗證兩個monitor可獨立運作且無global state
4. 跑Go race tests

### Step 3: Align AnLF And MTLF Clients

1. AnLF補validated endpoint、timeout與readiness
2. MTLF保留既有readiness hardening並加入handshake
3. 統一error classification，但不合併feature-specific clients

### Step 4: Implement MTLF Handshake

1. PyMTLF config與route
2. Go source inventory、bounded Mongo live ping與selection client
3. monitor在每次successful readiness後重新計算sources並handshake
4. mismatch保持unavailable並retry

### Step 5: Wire App Lifecycle And Advertisement

1. `NewApp`construct dependencies
2. Mongo client仍在startup初始化，source inventory則在每次MTLF handshake前以live ping重算
3. listener startup後啟動monitor workers
4. config控制NRF advertisement
5. current AnLF operation使用cached gate與503
6. shutdown取消workers並wait

### Step 6: Cross-process Verification

依第13節矩陣完成late start、restart、conflict與independent failure測試後，才可提交implementation。

---

## 13. Verification Plan

### 13.1 NWDAF Unit And Race Tests

至少覆蓋：

1. immediate first probe
2. exact bounded backoff progression
3. deterministic jitter boundary
4. success後30秒periodic probe
5. failure後reset到1秒retry
6. AnLF ready直接usable
7. MTLF ready但handshake failure仍unavailable
8. 每次MTLF successful probe都重新handshake
9. AnLF與MTLF狀態互不影響
10. snapshot concurrent read/write無race
11. parent cancellation中止sleep與HTTP call
12. app shutdown等待workers退出
13. transport/5xx標記unavailable；4xx不誤判process down
14. unavailable operation回503 ProblemDetails，不洩漏private error或non-standard backend cause
15. NRF profile advertisement config matrix
16. Mongo late start、failure與recovery會改變下一次handshake的source inventory
17. Mongo ping timeout或app cancellation不會阻塞monitor shutdown

planned commands：

```bash
go test ./internal/backend ./internal/anlf/... ./internal/mtlf/client ./pkg/factory ./pkg/service ./internal/context ./internal/sbi/processor
go test -race ./internal/backend ./internal/anlf/... ./internal/mtlf/client ./pkg/service
make test
make build
make lint
```

### 13.2 PyAnLF

```bash
uv run pytest -q
uv run ruff check .
```

tests需證明health route、repeated startup/shutdown與既有runtime API都通過。

### 13.3 PyMTLF

```bash
uv run pytest -q
uv run ruff check .
```

tests需證明handshake matrix、readiness semantics、artifact API與移除舊journal/reconciliation後的config/model
hygiene都通過。

### 13.4 Cross-process Matrix

| Scenario | Expected result |
|---|---|
| Go先啟動，兩個backend都不存在 | NWDAF持續運行；兩者unavailable |
| PyAnLF晚啟動 | AnLF自動轉usable，不需重啟Go |
| PyMTLF晚啟動且mode可滿足 | readiness、handshake後轉usable |
| PyMTLF要求`mongodb`但Go沒有Mongo | MTLF保持unavailable並retry |
| Mongo在Go之後才啟動 | 下一次MTLF probe重新ping，handshake後轉usable |
| Mongo由available轉為unreachable | 下一次handshake移除`mongodb`；需要它的MTLF mode轉unavailable |
| Mongo恢復 | 不重啟Go即可在下一次handshake恢復MTLF usable |
| PyMTLF重啟且Go觀察到downtime | unavailable後重新ready/handshake |
| PyMTLF在兩次probe間快速重啟 | 下一次successful probe仍重新handshake |
| PyAnLF down、PyMTLF up | 只影響AnLF-dependent operation |
| PyMTLF down、PyAnLF up | existing AnLF analytics不因MTLF monitor停止 |
| Go shutdown發生於probe或backoff | call取消、worker退出、WaitGroup完成 |

cross-process harness可以使用local HTTP test process，不要求Phase 2啟動真實ADRF或執行Mongo data query。
若full NWDAF startup受NRF/Mongo環境限制，必須分別回報unit、process-level與未執行的environment-level
驗證，不得把httptest稱為end-to-end。

### 13.5 Hygiene Audit

完成前執行：

- Go tree不含Python implementation name
- 沒有新的Daisy dependency或route
- 沒有`testdata` JSON contract fixture
- 沒有新的dataset proxy、chunk或generation CAS contract
- PyMTLF source、config與tests不再包含generation journal或reconciliation engine
- `git diff --check`
- 四個repository各自`git status`

---

## 14. Acceptance Criteria

Phase 2只有在以下條件全部成立時完成：

1. Go在backend缺席時可正常啟動與關閉
2. 兩個backend由不同worker獨立poll
3. PyAnLF與PyMTLF health semantics已被API tests固定
4. PyMTLF handshake exact path、shape、mode與failure semantics已固定
5. MTLF只有readiness與handshake都成功才usable
6. backend restart後不需重啟Go即可恢復
7. cached availability可供processor/coordinator使用
8. current AnLF-dependent failure可回503 ProblemDetails
9. config-based NRF advertisement有tests
10. MTLF標準services尚未實作前沒有被提前advertise
11. Release 18 type gap與後續compatibility strategy已記錄
12. Mongo source availability可在不重啟Go的情況下隨handshake恢復或失效
13. PyMTLF舊SQLite journal、reconciliation與專用config/models/tests已移除
14. 沒有預先建立unused standard types、broad RPC framework或testdata JSON
15. Go tests/race/build/lint、兩個Python tests/Ruff與cross-process matrix通過
16. 未執行的NRF、OAuth、TLS、Mongo或environment validation明確列出

---

## 15. Decision And Replan Gates

實作時遇到以下情況必須停止並回報：

1. PyAnLF無法在不載入active model的情況定義安全readiness
2. code audit發現journal/reconciliation仍有非測試production consumer，與目前「無consumer」前提衝突
3. current AnLF operation無法區分4xx domain failure與backend transport failure
4. config-based advertisement會破壞目前合法的AnLF-disabled deployment behavior
5. ADRF configured client不足以作為handshake available source，且需要新增non-standard health API
6. current Mongo client無法安全重複ping或在失聯後恢復，必須改變Mongo lifecycle ownership
7. `internal/backend`無法保持backend-neutral而開始吸收feature contract
8. Release 18 type gap迫使Phase 2提前生成大量尚無consumer的code
9. Phase 2若不改現有UPF writer就無法安全表示negotiated mode

blocker report必須列出原計畫、實際矛盾、選項、建議與是否需要修改parent plan。

---

## 16. Handoff To Later Phases

### Phase 3

直接使用AnLF connection gate與503 helper，遷移Events Subscription標準payload與ownership。Phase 3不得
再建立另一個AnLF poller。

### Phase 4

使用同一monitor狀態routing ML Model Monitor operations；Release 18 monitor compatibility types在該phase
依實際route建立。

### Phase 5

接管本phase只協商未套用的storage mode，完成：

- raw UPF notification Mongo document
- real bootstrap writer
- safe mode switch與dual-write
- PyMTLF direct Mongo/ADRF retrieval
- actual write/fetch operation health與retry需求

在Phase 5完成前，`selected storage mode`只代表MTLF backend requirement已被Go確認，不代表current legacy
writer已符合target storage contract。

### Phase 6

使用同一MTLF connection gate與standard-contract strategy完成training與model provision。若實際training
job需要durable retry state，依當時job lifecycle建立新的小型internal store，不恢復已移除的generation/apply
journal與reconciliation。
