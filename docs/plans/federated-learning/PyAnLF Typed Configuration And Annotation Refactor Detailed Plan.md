# PyAnLF Typed Configuration And Annotation Refactor Detailed Plan

日期：2026-08-02

狀態：待實作；與 PyMTLF role-oriented config refactor 同屬 Phase 6 前的
Python backend configuration checkpoint

相關文件：

- [Distributed NWDAF Federated Learning Implementation Plan](Distributed%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)
- [PyMTLF Role-Oriented Configuration Refactor Detailed Plan](PyMTLF%20Role-Oriented%20Configuration%20Refactor%20Detailed%20Plan.md)
- [Distributed NWDAF Model Monitoring And Federated Retraining Architecture](../../design/Distributed%20NWDAF%20Model%20Monitoring%20And%20Federated%20Retraining%20Architecture.md)
- [Current AnLF MTLF Feature Architecture](../../design/Current%20AnLF%20MTLF%20Feature%20Architecture.md)
- [NWDAF Development Policy](../../development_policy.md)

---

## 1. 目的

PyAnLF只有一種AnLF backend role，不需要複製PyMTLF的`local`／
`fl_server`／`fl_client`結構；但目前config interface仍有下列問題：

- `run.py`直接`yaml.safe_load()`，沒有集中式typed settings model；
- managers各自從root dictionary使用`.get()`並重複宣告defaults；
- unknown key不一定fail fast，拼字錯誤可能靜默回到default；
- repository YAML沒有展示`runtime_completion_delivery`等已支援設定；
- seconds、milliseconds、bytes、sample counts與queue capacities的unit
  naming不完全一致；
- `model_monitor`與`accuracy_monitor`對不熟悉流程的使用者難以分辨前者
  是outbound registration control，後者是AnLF提供的accuracy monitoring
  runtime；
- `max_retries`在analytics report與accuracy report delivery中分別代表
  「額外重試次數」與「總嘗試次數」，相同key卻有不同語意。

本計畫建立單一typed config source of truth、統一明顯不一致的欄位語意，
並把repository config改成完整且有就地註解的可執行reference。既有
analytics、collection、model demand、monitor、ingestion、ADRF-first
storage與MongoDB fallback行為保持不變。

---

## 2. Active slice

### 2.1 Behavior

```text
annotated YAML
  -> strict typed Settings
  -> owner-specific settings passed to managers
  -> unchanged PyAnLF runtime behavior
```

### 2.2 Repositories

| Repository | Responsibility |
| --- | --- |
| `PyAnLF/` | typed schema、loader、field migration、annotated config、constructor wiring、tests、README |
| `nwdaf-resources/` | local與distributed A/B PyAnLF config migration |
| `nwdaf-docs/` | canonical field semantics與checkpoint integration |

不改`NWDAF/`、PyMTLF、NRF、ADRF、SMF或UPF。Go ↔ PyAnLF HTTP payload、
standard-shaped schema、routes、status code與sync snapshot都不改。

### 2.3 Acceptance boundary

- repository config完整展示current supported fields；
- 每個非直觀field都有purpose、unit、owner或lifecycle註解；
- unknown／invalid config在startup fail fast；
- defaults只由typed settings定義，不再依賴各manager各自猜測；
- 現有PyAnLF lint／tests與portable E2E保持通過。

---

## 3. 設計決策

### 3.1 一份AnLF profile，不製造不存在的role

Repository仍提供一份主要profile：

```text
config/config.yaml
```

PyAnLF-A與PyAnLF-B使用相同schema，但在deployment assets中提供不同的
port、callback URI、allowed origins、AoI／group及peer endpoint值。C在
目前架構沒有PyAnLF process，因此不建立`anlf-server`之類的人造mode。

### 3.2 Typed schema是唯一default source

新增：

```text
src/py_anlf/config.py
```

使用與PyMTLF一致的Pydantic v2原則：

- immutable settings；
- `extra="forbid"`；
- paths、URLs、ports、positive intervals、queue bounds與retry ordering
  集中驗證；
- `load_settings(path)`負責YAML parsing與validation；
- `run.py`不再直接操作untyped dictionary。

各manager constructor接收其owner-specific settings，而不是root dict。
如需降低一次改動風險，可以先讓`SBIServer`完成typed composition，再依
vertical owner逐一更換constructor；checkpoint完成前不得留下production
manager仍以散落`.get(..., default)`作主要default source。

### 3.3 保留feature-oriented top-level structure

PyAnLF不像PyMTLF有互斥runtime roles，因此不做無意義的server／client
nesting。下列feature sections保留：

```text
server
log
model
model_provider
model_provision
model_monitor_registration
model_cutover
analytics
collection
ingestion
adrf
mongodb
accuracy_monitor
runtime_completion_delivery
```

其中只對容易誤解或unit不明確的key做一次性rename。

### 3.4 Model-related sections的責任

- `model`：本機model runtime fallback、bundle下載與architecture contract；
- `model_provider`：configured／NRF模式下選擇哪個remote MTLF NWDAF；
- `model_provision`：PyAnLF向selected provider建立／維護Model Provision
  demand所需設定；
- `model_monitor_registration`：新模型啟用後，PyAnLF向MTLF建立model-use
  registration及其retry policy；
- `accuracy_monitor`：PyAnLF接受MTLF monitor subscription、匹配prediction
  與ground truth，並向MTLF回報WAPE的provider runtime；
- `model_cutover`：new-before-old切換期間保留舊runtime的grace policy。

原`model_monitor`rename為`model_monitor_registration`，是為了避免它和
`accuracy_monitor`被誤認為同一組worker。

### 3.5 Delivery語意統一

所有duration fields使用`_seconds`；所有attempt count使用
`max_attempts`，並明確包含第一次傳送。

```yaml
analytics:
  report_delivery:
    request_timeout_seconds: 5
    max_attempts: 4
    retry_interval_seconds: 1
    worker_stop_timeout_seconds: 10

runtime_completion_delivery:
  request_timeout_seconds: 5
  retry_interval_seconds: 1

accuracy_monitor:
  report_delivery:
    request_timeout_seconds: 5
    max_attempts: 3
    retry_interval_seconds: 1
```

Migration保持現在的有效行為：

- current analytics `max_retries: 3`實際執行第一次加三次retry，因此target
  是`max_attempts: 4`；
- current accuracy `max_retries: 3`實際最多執行三次，因此target是
  `max_attempts: 3`；
- runtime completion目前無固定attempt ceiling，仍持續到`204`或shutdown，
  本計畫不偷偷新增上限。

### 3.6 Units與名稱

至少完成下列rename：

| Current | Target |
| --- | --- |
| `server.host` | `server.binding_host` |
| `analytics.ue_communication.sampling_interval` | `analytics.ue_communication.sampling_interval_seconds` |
| top-level `report_delivery` | `analytics.report_delivery` |
| `report_delivery.request_timeout` | `analytics.report_delivery.request_timeout_seconds` |
| `report_delivery.max_retries` | `analytics.report_delivery.max_attempts` |
| `report_delivery.retry_interval` | `analytics.report_delivery.retry_interval_seconds` |
| `report_delivery.worker_stop_timeout` | `analytics.report_delivery.worker_stop_timeout_seconds` |
| `runtime_completion_delivery.request_timeout` | `runtime_completion_delivery.request_timeout_seconds` |
| `runtime_completion_delivery.retry_interval` | `runtime_completion_delivery.retry_interval_seconds` |
| `model_monitor` | `model_monitor_registration` |
| `accuracy_monitor.ground_truth_check_interval` | `accuracy_monitor.ground_truth_check_interval_seconds` |
| `accuracy_monitor.report_period` | `accuracy_monitor.report_period_seconds` |
| `accuracy_monitor.report_delivery.request_timeout` | `accuracy_monitor.report_delivery.request_timeout_seconds` |
| `accuracy_monitor.report_delivery.max_retries` | `accuracy_monitor.report_delivery.max_attempts` |
| `accuracy_monitor.report_delivery.retry_interval` | `accuracy_monitor.report_delivery.retry_interval_seconds` |

`input_window`、`output_window`、`ring_buffer_size`不是duration：它們是window
steps／buffer entries，因此不加`_seconds`，但YAML必須就地註明。

### 3.7 不保留silent alias

和PyMTLF checkpoint一致：

- owned feature-branch configs一次migration；
- old keys不與new keys同時生效；
- unknown old key由strict schema拒絕；
- 錯誤訊息指出target field；
- HTTP及stored data不受config key migration影響。

---

## 4. Annotated config coverage

Repository YAML至少完整說明：

### 4.1 Process and model

- bind address／port；
- log level；
- default model reference與formal identity何時只作fallback；
- provision event dedup capacity；
- artifact trusted origins、download timeout與archive limits；
- input／output dimensions、TCN channel layout與bundle contract關係。

### 4.2 Provider, provision, monitor and cutover

- NRF／configured provider selection；
- provision及monitor service instance IDs／API roots；
- Analytics ID與model interoperability filters；
- provision callback、request timeout及retry；
- monitor registration timeout及retry；
- rollback grace period。

### 4.3 Analytics and collection

- sampling interval是seconds；
- input／output window是steps；
- ring buffer及dedup capacities是entry counts；
- SMF selection mode與configured endpoints；
- discovery／request／reconcile／association retry；
- worker count、task queue capacity與callback base URI；
- static group membership是Phase 6 UDM integration前的experiment mapping。

### 4.4 Ingestion and storage

- callback body size與drop-oldest buffer；
- ADRF／Mongo outbound queue capacities；
- ADRF-first行為、NRF／configured resolution及recovery backoff；
- MongoDB是ADRF unavailable時的local fallback，而非distributed FL主路徑；
- Mongo connect timeout使用milliseconds，其他request／retry使用seconds。

### 4.5 Accuracy and delivery

- prediction／ground-truth matching cadence；
- stable report period與minimum matched samples；
- analytics report、accuracy report及runtime completion是三條不同delivery
  lifecycle；
- `max_attempts`是否有上限及是否包含first attempt；
- worker stop timeout的shutdown用途。

---

## 5. Implementation slices

### Slice A1：Typed schema and loader

- 建立所有section settings models；
- 移動current defaults與validation；
- 補上目前YAML未展示的supported fields；
- `run.py`使用`load_settings()`；
- config validation errors在server startup前回報。

### Slice A2：Owner-specific constructor wiring

依序遷移：

1. `SBIServer` composition；
2. model manager／runtime manager；
3. model provider／provision／monitor registration；
4. collection／ingestion／ADRF／Mongo；
5. accuracy monitor／report delivery／runtime completion。

每個slice保留existing behavior tests，不建立第二套parallel config path。

### Slice A3：Annotated config and README

- 更新`config/config.yaml`為完整typed schema example；
- 每個非直觀field加短註解；
- README補feature section索引與常見調整位置；
- 不把完整reference複製到README，避免兩份default drift。

### Slice A4：Integration migration

- 更新`nwdaf-resources`所有PyAnLF config/template；
- A/B使用獨立callback URL、model provider target及collection scope；
- local regression使用同一schema；
- runner生成的temporary YAML也先經PyAnLF loader驗證。

---

## 6. Verification

### 6.1 Config

- repository config可載入；
- unknown／old keys拒絕；
- blank URL、invalid port、negative timeout、retry max小於initial、queue
  capacity非正數、ring buffer小於input window等拒絕；
- analytics與accuracy delivery的`max_attempts`測試證明migration前後總次數
  相同；
- missing optional sections得到唯一typed defaults。

### 6.2 Behavior

- subscription runtime scheduling與report count不變；
- model demand、NRF/configured provider selection與new-before-old cutover不變；
- group membership、SMF collection refcount與reconcile不變；
- ingestion buffer仍drop oldest；
- ADRF-first／Mongo fallback與recovery不變；
- WAPE matching、minimum matched sample gate與periodic delivery不變；
- sync／restart behavior不變。

### 6.3 Repository and integration

- PyAnLF lint；
- PyAnLF full pytest；
- `nwdaf-resources` config checks；
- existing local／distributed portable E2E；
- 不從typed config unit tests宣稱real NRF／SMF／UPF／ADRF integration。

---

## 7. Non-goals

- 不新增PyAnLF runtime role；
- 不改Analytics Subscription、Model Provision或Model Monitor wire schema；
- 不改prediction／WAPE／degradation business logic；
- 不實作Phase 6 UDM group resolution或SMF AoI changes；
- 不更改ADRF-first／Mongo fallback decision；
- 不調整queue capacity defaults或drop-oldest policy；
- 不新增dynamic reload、secrets、TLS或OAuth；
- 不為feature-branch old keys保留永久alias。

---

## 8. Completion criteria

1. PyAnLF所有runtime config由typed schema載入；
2. repository YAML完整、自我說明且沒有隱藏supported field；
3. manager不再各自擁有互相可能漂移的default；
4. seconds／milliseconds／bytes／steps／entries可由key或註解辨識；
5. model provider、provision、monitor registration與accuracy provider責任
   能從config section直接理解；
6. delivery attempt semantics一致且migration前後observable behavior相同；
7. 所有owned configs完成migration；
8. full tests與existing portable E2E通過。
