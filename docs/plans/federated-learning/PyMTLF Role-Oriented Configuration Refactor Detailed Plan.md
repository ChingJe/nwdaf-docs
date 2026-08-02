# PyMTLF Role-Oriented Configuration Refactor Detailed Plan

日期：2026-08-02

狀態：role-oriented config與model identity namespace removal均已完成實作與驗證

相關文件：

- [Distributed NWDAF Federated Learning Implementation Plan](Distributed%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)
- [Phase 1 Role-Aware Deployment And NRF Foundation Detailed Plan](Phase%201%20Role-Aware%20Deployment%20And%20NRF%20Foundation%20Detailed%20Plan.md)
- [Phase 3 And 4 Federated Training Execution Detailed Plan](Phase%203%20And%204%20Federated%20Training%20Execution%20Detailed%20Plan.md)
- [Phase 5 Final Validation ADRF Publication And Reprovision Detailed Plan](Phase%205%20Final%20Validation%20ADRF%20Publication%20And%20Reprovision%20Detailed%20Plan.md)
- [PyAnLF Typed Configuration And Annotation Refactor Detailed Plan](PyAnLF%20Typed%20Configuration%20And%20Annotation%20Refactor%20Detailed%20Plan.md)
- [Distributed NWDAF Model Monitoring And Federated Retraining Architecture](../../design/Distributed%20NWDAF%20Model%20Monitoring%20And%20Federated%20Retraining%20Architecture.md)
- [NWDAF Development Policy](../../development_policy.md)

---

## 1. 目的

目前 PyMTLF 已透過 `runtime.mode` 正確分離 `local`、`fl_server` 與
`fl_client` lifecycle，但 config schema 仍是依功能成長歷史平鋪：

- `federated_learning` 同時保存 common transport、Server orchestration
  與 Client execution 設定；
- `training` 同時保存 standalone local fitting、FL Client fitting 與
  FL Server final validation gate；
- 單一 `config/config.yaml` 同時展示三種角色的所有欄位，使用者無法從
  部署設定直接判斷哪些欄位在目前 mode 會生效。

本 checkpoint 將 config interface 改成角色導向，讓每個部署檔只展示
該 process 真正使用的設定，同時保持已完成 Phase 1–5 的 runtime、HTTP、
training、FedAvg、validation、publication 與 cutover 行為不變。

這是 config contract refactor，不是 FL algorithm 或 SBI refactor。

---

## 2. Active slice

### 2.1 Behavior

完成以下 vertical slice：

```text
role-specific YAML
  -> strict Pydantic validation
  -> role-specific application wiring
  -> unchanged local / FL Server / FL Client behavior
  -> role-specific integration profiles
```

### 2.2 Repository ownership

| Repository | Responsibility |
| --- | --- |
| `PyMTLF/` | config models、loading、role-specific example files、application wiring、tests、README |
| `nwdaf-resources/` | distributed E2E 的 C Server 與 A/B Client config migration |
| `nwdaf-docs/` | canonical plan、supersession notes、architecture-level ownership |

Slice C1–C4不改`NWDAF/`、`PyAnLF/`、`nrf/` 與 `adrf/`，也不改
Go ↔ PyMTLF wire contract、NRF profile、public SBI 與standard-shaped
private API。第10節的identity cleanup會另外同步修改
`PyAnLF/`與`nwdaf-resources/`的private bundle／identity contract，但不改
3GPP wire schema。

### 2.3 Acceptance boundary

- 三個 role configs 都能被 strict schema 載入；
- 每份 config 只要求並展示 active role 的欄位；
- `local` 仍只走 local retraining；
- `fl_server` 仍負責 monitor policy、round orchestration、FedAvg、final
  validation、publication 與 reprovision；
- `fl_client` 仍只接受 Training resources、準備 ADRF dataset、local
  fitting、提供暫存 artifact 與回傳 validation；
- Phase 5 portable E2E migration 後保持通過。

---

## 3. 已確認的設計決策

### 3.1 Role 是 config 的第一層閱讀入口

`runtime.mode` 仍保留：

```yaml
runtime:
  mode: "local"  # local | fl_server | fl_client
```

但使用者不再需要從一個混合設定檔判斷哪些 flat fields 對目前 role 有效。
Repository 提供三份可直接執行的 profile：

```text
config/
├── local.yaml
├── fl-server.yaml
└── fl-client.yaml
```

原本泛稱的 `config/config.yaml` 不再作為三種角色的混合 reference。
README、啟動命令與 integration assets 必須顯式選擇其中一份。

### 3.2 Common FL transport 與 role policy 分離

`federated_learning` 只在 distributed modes 使用，並保留少量雙方都需要
理解的 workspace／transport fields；role-specific fields 分別進入
`server` 與 `client`：

```yaml
federated_learning:
  workspace_root: "data/fl-workspaces"
  workspace_ttl_seconds: 3600
  public_base_url: "http://127.0.0.1:9092"
  request_timeout_seconds: 300
  artifact_download:
    allowed_origins:
      - "http://127.0.0.1:9093"
    timeout_seconds: 300

  server:
    # only present in fl-server.yaml

  client:
    # only present in fl-client.yaml
```

不新增名稱含糊的 `common` wrapper；直接位於
`federated_learning` 的欄位就是模型交換雙方共用的 transport／workspace
設定。

### 3.3 Server 擁有 round 與 final validation policy

`fl_server` profile 集中：

```yaml
federated_learning:
  server:
    callback_uri: "http://127.0.0.1:9092/internal/v1/ml-model-training/notifications"
    preparation_timeout_seconds: 300
    round_timeout_seconds: 300
    round_count: 2
    max_active_processes: 1

    delay_policy:
      max_extensions: 1
      max_extension_seconds: 300

    cleanup:
      max_attempts: 3
      retry_backoff_seconds: 0.2

    final_validation:
      enforce_performance_gate: false
      max_scope_wape_regression: 0.02
```

語意：

- `round_count` 只計算 training rounds；final validation 是額外一輪；
- preparation／round deadlines 由 Server 產生並送進 Training resource；
- delay acceptance、remote resource cleanup 與 global promotion gate 都是
  Server policy；
- Server 不顯示 `epochs`、`batch_size` 或 `learning_rate`，因為 Server
  不執行 Client local fitting。

### 3.4 Client 擁有 local fitting policy

`fl_client` profile 集中：

```yaml
federated_learning:
  client:
    callback_deadline_margin_seconds: 5
    callback_queue_size: 256
    max_concurrent_jobs: 2
    model_interoperability_ids:
      - "001122"

    fallback_deadlines:
      preparation_timeout_seconds: 300
      round_timeout_seconds: 300

    training:
      enabled: true
      device: "cpu"
      batch_size: 32
      learning_rate: 0.001
      epochs: 18
      validation_ratio: 0.20
      random_seed: 42
```

語意：

- `epochs` 是每個 Client、每個 training round 的本地完整資料 passes；
- Client 不設定 `round_count`；收到 Server 的 `roundInd` 才執行該輪；
- `callback_queue_size` 是 Client outbound callback outbox 的容量，不是
  Server inbound notification queue；
- `fallback_deadlines` 只在 peer request 沒有可用 deadline 時提供
  Client safety fallback，不取代 Server 在正常流程送出的 `maxResTime`；
- 每個 Client 可以有自己的 hyperparameters；第一版實驗的 A/B profiles
  必須使用相同值，避免把不同 local policy 混入 baseline 結果。

### 3.5 Local training 不再借用 FL Client schema

`local` profile 使用獨立區段：

```yaml
local_training:
  enabled: true
  device: "cpu"
  batch_size: 32
  learning_rate: 0.001
  epochs: 18
  validation_ratio: 0.20
  random_seed: 42
  max_concurrent_jobs: 1
  max_queue_size: 16

  validation:
    enforce_performance_gate: false
    max_scope_wape_regression: 0.02
```

Python implementation可以共用 immutable `FittingSettings` 與
`ValidationSettings` model，但 YAML ownership 必須分開。這樣未來調整
FL Client epochs 不會不小心改變 standalone local retraining。

### 3.6 其他 feature config 維持原 owner

下列 top-level sections 不因本 refactor 重新設計：

- `server`、`runtime`；
- `storage`、`model_state`、`artifact`；
- `model_provision`、`model_monitor`、`accuracy_policy`；
- `adrf`、`dataset`、`publication`、`cutover`；
- `notification`、`log`。

Role-specific YAML 可省略 inactive feature sections；Pydantic defaults 只為
application construction 提供安全值，不得因此啟動 inactive coordinator。

### 3.7 不保留舊 flat alias

本 workstream 尚位於 feature branch，且 config 由本 workspace 的
deployment assets 管理。為避免同一設定同時存在兩個來源：

- 不同時接受舊 `training.*` 與新 role-specific path；
- 不為舊 flat `federated_learning.round_count` 等欄位建立 silent alias；
- 舊 config 由 migration 一次改完；
- strict `extra="forbid"` 對舊欄位 fail fast，錯誤訊息應能指出新的 role
  section。

這是刻意的 config migration，不影響 HTTP backward compatibility。

---

## 4. Current-to-target field map

### 4.1 FL common

| Current | Target |
| --- | --- |
| `federated_learning.workspace_root` | unchanged |
| `federated_learning.workspace_ttl_seconds` | unchanged |
| `federated_learning.public_base_url` | unchanged |
| `federated_learning.request_timeout_seconds` | unchanged |
| `federated_learning.artifact_download.*` | unchanged |

### 4.2 FL Server

| Current | Target |
| --- | --- |
| `federated_learning.callback_uri` | `federated_learning.server.callback_uri` |
| `federated_learning.preparation_timeout_seconds` | `federated_learning.server.preparation_timeout_seconds` |
| `federated_learning.round_timeout_seconds` | `federated_learning.server.round_timeout_seconds` |
| `federated_learning.round_count` | `federated_learning.server.round_count` |
| `federated_learning.max_active_server_processes` | `federated_learning.server.max_active_processes` |
| `federated_learning.max_delay_extensions` | `federated_learning.server.delay_policy.max_extensions` |
| `federated_learning.max_delay_extension_seconds` | `federated_learning.server.delay_policy.max_extension_seconds` |
| `federated_learning.cleanup_max_attempts` | `federated_learning.server.cleanup.max_attempts` |
| `federated_learning.cleanup_retry_backoff_seconds` | `federated_learning.server.cleanup.retry_backoff_seconds` |
| `training.enforce_performance_gate` | `federated_learning.server.final_validation.enforce_performance_gate` |
| `training.max_scope_wape_regression` | `federated_learning.server.final_validation.max_scope_wape_regression` |

### 4.3 FL Client

| Current | Target |
| --- | --- |
| `federated_learning.callback_deadline_margin_seconds` | `federated_learning.client.callback_deadline_margin_seconds` |
| `federated_learning.callback_queue_size` | `federated_learning.client.callback_queue_size` |
| `federated_learning.max_concurrent_client_jobs` | `federated_learning.client.max_concurrent_jobs` |
| `federated_learning.model_interoperability_ids` | `federated_learning.client.model_interoperability_ids` |
| Client fallback use of `preparation_timeout_seconds` | `federated_learning.client.fallback_deadlines.preparation_timeout_seconds` |
| Client fallback use of `round_timeout_seconds` | `federated_learning.client.fallback_deadlines.round_timeout_seconds` |
| `training.enabled` | `federated_learning.client.training.enabled` |
| `training.device` | `federated_learning.client.training.device` |
| `training.batch_size` | `federated_learning.client.training.batch_size` |
| `training.learning_rate` | `federated_learning.client.training.learning_rate` |
| `training.epochs` | `federated_learning.client.training.epochs` |
| `training.validation_ratio` | `federated_learning.client.training.validation_ratio` |
| `training.random_seed` | `federated_learning.client.training.random_seed` |

### 4.4 Local

| Current | Target |
| --- | --- |
| `training.enabled` | `local_training.enabled` |
| `training.device` | `local_training.device` |
| `training.batch_size` | `local_training.batch_size` |
| `training.learning_rate` | `local_training.learning_rate` |
| `training.epochs` | `local_training.epochs` |
| `training.validation_ratio` | `local_training.validation_ratio` |
| `training.random_seed` | `local_training.random_seed` |
| `training.max_concurrent_jobs` | `local_training.max_concurrent_jobs` |
| `training.max_queue_size` | `local_training.max_queue_size` |
| `training.enforce_performance_gate` | `local_training.validation.enforce_performance_gate` |
| `training.max_scope_wape_regression` | `local_training.validation.max_scope_wape_regression` |

---

## 5. Implementation slices

### Slice C1：Schema ownership

在 `PyMTLF/src/py_mtlf/config.py`：

1. 抽出可共用但不直接暴露成另一個 top-level family 的
   `FittingSettings`；
2. 建立 `LocalTrainingSettings`；
3. 建立 `FLServerSettings`、`FLClientSettings`；
4. 建立 `DelayPolicySettings`、`CleanupSettings`、
   `FinalValidationSettings` 與 `FallbackDeadlineSettings`；
5. `FederatedLearningSettings` 只持有 common fields 及 optional
   `server`／`client` branches；
6. mode-aware validation 確認 active role branch存在，並對矛盾設定
   fail fast；
7. 保留目前所有 bounds、URL normalization、retry ordering 與 deadline
   safety validation。

不得以 untyped dictionary 或 runtime `getattr` 取代明確 Pydantic models。

### Slice C2：Application wiring

更新 `create_app()` 與直接 consumers：

- local coordinator只收到 `local_training`；
- FL Client service只收到 `federated_learning.client` fitting／capacity／
  fallback settings 與 common transport；
- FL Server orchestrator只收到 `federated_learning.server` orchestration／
  final validation settings與 common transport；
- FL workspace只收到 common workspace／artifact settings；
- 不改 route exposure、state machine、round numbering、FedAvg、callback、
  publication 或 cutover behavior。

每個 settings consumer 要以 owner-specific typed object 作 constructor
dependency，避免再次把整個 root settings 傳入後任意取值。

### Slice C3：Role profiles and documentation

建立三份 annotated YAML：

- `local.yaml`：只展示 standalone model ownership、monitor、dataset 與
  local training；
- `fl-server.yaml`：展示 C 的 provision／monitor／orchestration／final
  validation／publication；
- `fl-client.yaml`：展示 A/B 的 training provider、ADRF dataset、local
  fitting與 callback delivery。

每個非直觀欄位的旁邊註明：

- purpose；
- seconds／milliseconds／bytes 等 unit；
- Server／Client owner；
- 和 round、epoch、validation、retry 或 artifact lifecycle 的關係。

README 的啟動範例必須顯式選擇 role profile，並用一個短表說明哪份檔案
對應 A、B、C 與 legacy local deployment。

### Slice C4：Integration migration

在 `nwdaf-resources`：

- A/B PyMTLF 使用 `fl-client` profile；
- C PyMTLF 使用 `fl-server` profile；
- local portable regression 使用 `local` profile；
- 每個 process 使用獨立 port、workspace、public base URL、callback URL
  與 allowed peer origins；
- 不以環境變數偷偷覆寫新 schema 中沒有出現在 role YAML 的必要值。

若 runner 會 materialize temporary config，template source仍需符合新
schema並在測試中載入驗證。

---

## 6. Verification

### 6.1 Config tests

- 三份 repository configs 都可載入；
- 每個 role 的 effective values與 YAML 一致；
- unknown mode拒絕；
- active branch缺失拒絕；
- old flat FL Server／Client／training paths拒絕；
- invalid timeout、deadline margin、retry ordering、queue size、URL、
  artifact origin與 model interoperability拒絕；
- A/B Client hyperparameters可以獨立載入，但 experiment fixtures確認
  baseline profiles相同。

### 6.2 Lifecycle tests

- `local` 啟動 local coordinator，沒有 Training routes；
- `fl_server` 啟動 provision／monitor／Server orchestration，不啟動 local
  trainer；
- `fl_client` 啟動 Training provider，不掛載 provision／monitor routes；
- health與sync仍回報正確 runtime mode；
- shutdown不留下新增 worker或 workspace handle。

### 6.3 Behavior regression

- Client一個 training round仍依 Client profile的 `epochs` fitting；
- Server仍執行設定的 training `round_count`，再執行額外 final
  validation-only round；
- sample count仍是 unique local training windows，不乘 epochs；
- Server gate disabled時仍記錄 evidence並允許 promotion；enabled時沿用
  既有 WAPE gate；
- local mode candidate validation與現況一致。

### 6.4 Integration

- PyMTLF full lint／pytest；
- `nwdaf-resources` config／contract checks；
- existing portable Phase 5 cross-process E2E；
- real NRF、ADRF、SMF、UPF integration claim不因 config unit tests自動
  擴張。

---

## 7. Non-goals

- 不新增 per-round convergence／early stopping；
- 不由 FL Server指定 Client epochs；
- 不改 fixed participant、synchronous FedAvg或all-Clients-required policy；
- 不改 Model Training／Provision／Monitor wire schema；
- 不改 NRF capability profile；
- 不新增 dynamic config reload；
- 不新增 secrets、TLS或OAuth config；
- Slice C1–C4不重構 dataset、ADRF publication或model catalog實作；
  第10節僅移除catalog identity中的provider namespace，不改選模、
  training、publication或cutover business logic；
- 不為 feature-branch舊 config保留永久 compatibility alias。

---

## 8. Completion criteria

本 checkpoint 完成需同時滿足：

1. config schema的欄位位置能直接表達 runtime owner；
2. 三份 role YAML能獨立作為可執行且自我說明的設定；
3. Server config不再出現Client local fitting欄位；
4. Client config不再出現Server round／promotion policy；
5. local config不再展示distributed FL orchestration；
6. old flat config在所有owned assets完成一次性migration；
7. Phase 1–5既有behavior與E2E保持通過；
8. 主計畫與舊 detailed plans清楚記錄supersession，不留下兩個都宣稱
   canonical的config shape。

---

## 9. Implementation record

- repository profile已拆成`config/local.yaml`、`config/fl-server.yaml`與
  `config/fl-client.yaml`，預設CLI使用local profile；
- common FL transport／workspace與Server、Client role settings已拆成不同
  typed models，standalone fitting移至`local_training`；
- runtime mode會拒絕missing／conflicting role branch，舊`training`與flat
  FL role keys由`extra="forbid"`拒絕；
- trainer、dataset builder、local coordinator、FL Client及FL Server改為接收
  owner-specific settings；
- `nwdaf-resources`的local與distributed config generator已完成migration，
  並會在啟動process前呼叫backend loader驗證temporary YAML；
- model catalog已改用scalar `familyId`與`modelUniqueId`，seed import、bundle、
  durable state、sync及publication不再保存provider namespace；durable state
  schema升級為`2.0`；
- verification：PyMTLF Ruff與`187 passed`、PyAnLF Ruff與
  `274 passed, 1 skipped`、resources Go module tests、local model lifecycle
  E2E及完整distributed FL／ADRF publication／cutover runner均通過。

---

## 10. Model identity namespace removal extension

### 10.1 原因與邊界

[TS 29.520 §5.4.6](../../../specs/TS%2029.520/5%20API%20Definitions/5.4%20Nnwdaf_MLModelProvision%20Service%20API/5.4.6%20Data%20Model.md)
將`modelUniqueId`定義為`Uinteger`，並要求在5GC scope內唯一。
現有PyMTLF卻同時使用：

```text
FamilyKey       = (provider_namespace, family_id)
ModelVersionKey = (provider_namespace, modelUniqueId)
```

這會讓private namespace看起來像標準模型識別的必要條件，也可能
掩蓋duplicate `modelUniqueId`本身不符合規格的事實。本擴充只統一
identity representation，不重新設計multi-provider ID allocator。

### 10.2 Target identity

```text
FamilyKey       = familyId
ModelVersionKey = modelUniqueId
ArtifactKey     = SHA-256(bundle bytes)
```

- `familyId`仍是PyMTLF private logical family key；
- `modelUniqueId`是正式模型的唯一canonical identity；
- provider NWDAF `nfInstanceId`是route／provenance metadata，不放入
  `FamilyKey`或`ModelVersionKey`；
- bundle不再用generic `provider_id`建立model identity；本擴充不新增
  另一個namespace的替代名稱。

### 10.3 PyMTLF implementation slice

1. 移除`ModelProvisionSettings.provider_namespace`與三份role config中的
   `provider_namespace`；
2. `FamilyKey`改為單純`familyId`，`ModelVersionKey`改為單純
   numeric `modelUniqueId`；
3. catalog、provision resources、accuracy policy、dataset jobs、training jobs、
   notification delivery與publication依新key串接，不改其狀態機與業務
   條件；
4. seed import、seed bundle、completed model bundle與FL artifact contract移除
   `provider_id`；formal completed artifact仍保存`modelUniqueId`；
5. durable model state升級schema，移除`providerNamespace`；新版不讀取
   舊state，實驗啟動前顯式清除舊`data/model-state/`與對應
   artifacts；
6. backend sync移除`providerNamespace`，不從containing Go或PyAnLF
   取得替代namespace。

### 10.4 Allocation assumption and deferral

- 當前情境只有NWDAF-C／PyMTLF-C配發正式model IDs；
- 沿用`max(currentUnixMilliseconds, lastAllocatedModelId + 1)`與
  durable non-reuse；
- 不將`modelUniqueId`改為UUID，因為標準schema是`Uinteger`；
- multi-provider 5GC-wide uniqueness為future-phase handoff，不阻擋本次
  namespace removal。

### 10.5 Verification

- config loader拒絕舊`provider_namespace`；
- catalog、retrain、promotion、restart與notification tests改以單純family／model
  identity驗證；
- seed、round／final bundle contract不再含`provider_id`；
- sync response不再含`providerNamespace`；
- PyMTLF lint／full pytest、PyAnLF contract regression與portable cross-process
  E2E通過；
- `rg` 確認active config、production code與owned fixtures無
  `provider_namespace`／`providerNamespace`／model identity `provider_id`殘留。

### 10.6 Completion record

- `FamilyKey`與`ModelVersionKey`已分別改為scalar `familyId`與numeric
  `modelUniqueId`；
- seed與completed bundle只在`model_identity.model_unique_id`保存正式模型
  identity；round-local FL bundles仍不建立正式identity；
- timestamp加durable monotonic allocator保持現狀，沒有加入UUID或新的private
  namespace；
- 舊namespace config、CLI option、state schema及owned integration fixture已移除；
- actual provider `nfInstanceId`只留在selected target與route log，不參與模型
  equality、cache或monitor worker key。
