# Slice 1：Explicit Orchestration 與 Static／Manual Training 詳細計畫

日期：2026-08-25

狀態：Implementation Verified；Mandatory Review Complete；User Review Confirmed；Commit Approved；testbed validation pending

相關文件：

- [Explicit Flat and Hierarchical Orchestration Configuration Detailed Plan](./Explicit%20Flat%20and%20Hierarchical%20Orchestration%20Configuration%20Detailed%20Plan.md)
- Slice 2：MTLF-Triggered UPF Data Collection 詳細計畫（review deferred；不作為本 Slice implementation input）
- [Hierarchical NWDAF Federated Learning Implementation Plan](../hierarchical-fl-model-bundle-edition/Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)
- [Slice 3 Root Initiation, Static Topology and Assignment Detailed Plan](../hierarchical-fl-model-bundle-edition/Slice%203%20Root%20Initiation%2C%20Static%20Topology%20and%20Assignment%20Detailed%20Plan.md)
- [Slice 6 Hierarchical Final Validation and Publication Detailed Plan](../hierarchical-fl-model-bundle-edition/Slice%206%20Hierarchical%20Final%20Validation%20and%20Publication%20Detailed%20Plan.md)
- [Slice 7 Lifecycle Closure and Fresh-state Restart Detailed Plan](../hierarchical-fl-model-bundle-edition/Slice%207%20Lifecycle%20Closure%20and%20Fresh-state%20Restart%20Detailed%20Plan.md)
- [NWDAF Development Policy](../../../development_policy.md)
- [Release 18 Nnwdaf ML Model Training OpenAPI](../../../../specs/openapi/TS29520_Nnwdaf_MLModelTraining.yaml)

---

## 1. Slice 目標與完成邊界

本 Slice 完成 explicit orchestration configuration、static flat participant planning、flat／HFL 共用 manual
training API 與既有 execution lifecycle 的整合。它移除 topology-presence inference，建立唯一 selected
top-level coordinator，並讓 static flat 在 matching training data 已存在時可完整執行 preparation、rounds、
validation、publication 與 cleanup。

本 Slice 完成後：

1. `server`／`client` 只決定 engines 是否存在；`orchestration` 才決定 autonomous top-level owner。
2. 每個 PyMTLF process 最多一個 selected coordinator；Assigned Branch 不自主發起 flat 或 HFL training。
3. Production flat degradation、既有 HFL degradation／manual 與 local retraining 經 explicit settings 後
   維持原 lifecycle。
4. Static flat manual request 不依賴 Monitor-derived participants，並 exact resolve topology 中的 Clients。
5. FL Client／Leaf 顯式設定 `training_data.collection_trigger: consumer_subscription`，並完整保留現有
   descriptor＋ADRF／MongoDB collected snapshot semantics。
6. Flat 與 HFL 共用 training request／status contract，但 Flat／Root lifecycle 分別由各 coordinator
   擁有，不移到 API layer。
7. 所有 intended changes 保持 unstaged、uncommitted，完成測試與 mandatory review 後先交使用者在 IDE
   審查；commit 另需提案與核准。

本 Slice 不實作 `private_api` data collection、不新增 UPF callback／storage、不支援 production
`local_file`，也不執行 paired numerical comparison。Static manual success 的明確 precondition 是
data-owning participant 已透過現有 `consumer_subscription` path 取得 matching descriptor 與 records；
「沒有 analytics consumer 也能主動蒐集」由 Slice 2 關閉。

## 2. 基準版本、repository 與責任

### 2.1 計畫基準

| 儲存庫 | 版本 | 本 Slice 定位 |
| --- | --- | --- |
| `PyMTLF/` | `e9aa2235b6b4adc1d9d778b6cfdf23645fc622ec` | production implementation 與主要 tests |
| `nwdaf-resources/` | `39ced284561e542c50da5fa7e83830aae4517821` | config generators、preflight 與 training API caller migration |
| `NWDAF/` | `3279891689dd9b54737ffe08dc18b9db72ec57b4` | containing context、NRF relay 與 Training SBI read-only baseline |
| `PyAnLF/` | `6a4d94ad3cc6f66dac55ea921772d731e4b71371` | `consumer_subscription` collection regression baseline |
| `nrf/` | `0dd4024d4ab75b6630e04901968228b9b9718cf5` | exact-instance discovery baseline |
| `adrf/` | `905f0599f68fe389bba14ed56db0ef9abeab5ccd` | collected retrieval 與 final publication baseline |

開始 implementation 前重新確認 revisions 與 working trees。若 target moved，先更新 characterization 與
conformance map，不把舊 revision 結論直接套用。

### 2.2 預計修改

- `PyMTLF/`：config、app construction、top-level orchestration、static flat topology、FL Server／Root
  adaptation、generic training API、lifecycle 與 tests。
- `nwdaf-resources/`：production flat／HFL generated configs、preflight assertions 與 manual training
  caller path。
- `nwdaf-docs/`：本 Slice 狀態與 implementation evidence。

`NWDAF/`、`PyAnLF/`、`nrf/` 與 `adrf/` 在本 Slice read-only。若 deterministic test 證明其 current
contract 阻擋本 Slice，依 development policy 進 decision gate，不直接擴大 repository scope。

### 2.3 標準邊界

- 本 Slice 不新增 public SBI endpoint 或 Go package。
- Static flat NRF query 仍經 containing NWDAF private relay，並驗證 exact ID、service、capability、event、
  interoperability 與 TAI。
- HFL hierarchy resolution 不送出或檢查 TAI。
- Training Subscribe 的 `DataAvReq.timeWindows` 保持 absolute interval。DatasetCoordinator 不 re-anchor，
  descriptor／records 無法涵蓋 requested interval 時 preparation 失敗。

## 3. 固定架構決策

### 3.1 Top-level owner 與 execution engine 分離

新增 `FlatFLCoordinator` 作 flat autonomous initiation owner；`FLServerEngine` 保留 preparation、round、
aggregation、validation、publication 與 cleanup execution owner。這項分離避免 Assigned Branch 因持有
lower-tier Server engine 而變成 autonomous flat owner。

`create_app()` 依 resolved mode 建立：

- `flat`：一個 `FlatFLCoordinator`，持有 existing `FLServerEngine`；
- `hierarchical`：一個 `FLRootCoordinator`；
- 無 `orchestration`：不建立 top-level coordinator。

共同入口為 `app.state.fl_coordinator`。`app.state.fl_server`、`fl_client`、`fl_branch` 表示 engine
presence，router 與 monitor dispatcher 不再依 object presence 猜測 owner。

### 3.2 共用 contract，不建立中央狀態機

`src/py_mtlf/core/fl_orchestration.py` 或等價 existing-owner file 只擁有：

- `TopLevelCoordinator` protocol；
- `TopLevelInitiation`、`TriggerSource` 與 discriminated `ParticipantSelection`；
- `TopLevelRequestSnapshot`；
- common request domain errors 與 terminal／failure envelope。

Flat request records 由 `FlatFLCoordinator` 擁有；HFL request／plan records 仍由 `FLRootCoordinator`
擁有。API router 只做 wire validation、domain invocation 與 response mapping。

### 3.3 Participant、cutover 與 trigger 分離

`TopLevelInitiation` 至少保存：

- `model_family_id`；
- `trigger_source: degradation | private_api`；
- `participant_source: monitor_scopes | static`；
- discriminated `participant_selection`；
- `required_cutover_scope_keys`；
- `triggering_scope_key: str | None`；
- `request_id: UUID | None`；
- config／topology identity 與 process generation。

| Mode／trigger | 參與者 | 必要 cutover scopes | 觸發 scope |
| --- | --- | --- | --- |
| Flat monitor degradation | Monitor scopes 轉成 typed flat selection | Monitor active scope keys | degradation scope |
| Flat static degradation | current family＋static topology | Monitor active scope keys | degradation scope |
| Flat static manual | current family＋static topology | 空集合 | `None` |
| HFL static degradation | immutable hierarchy plan | Monitor active scope keys | degradation scope |
| HFL static manual | immutable hierarchy plan | 空集合 | `None` |

Static participants 不寫回 `RetrainIntent.active_scopes`；HFL identities 不偽裝成 flat TAI scopes。

## 4. 既有基準階段處置

| 階段 | Production flat degradation | 既有 HFL | Static／manual extension |
| --- | --- | --- | --- |
| Trigger | explicit degradation gate 送 selected flat coordinator | selected Root 接收 degradation／manual | generic training API；manual 不讀 Monitor scopes |
| Base model | reuse current catalog | reuse | reuse current family＋artifact |
| Participant planning | reuse Monitor snapshot | reuse static hierarchy planner | new static flat planner |
| NRF resolution | reuse exact Client＋TAI | reuse exact identity，無 TAI | static flat 由 topology TAI 驅動，禁止 decoy substitute |
| Collection | reuse PyAnLF consumer flow | reuse | approved precondition：matching collected data已存在 |
| Preparation | reuse descriptor＋ADRF／Mongo＋absolute window | reuse | reuse without source branch／re-anchor |
| Local fitting | reuse builder／seed／purged split | reuse Leaf fitting／Branch delegation | reuse |
| Rounds／aggregation | reuse flat FedAvg | reuse HFL FedProx | static flat reuse FedAvg；HFL FedAvg deferred |
| Final validation | reuse degradation checks | reuse hierarchy finalization | manual `triggering_scope_key=None` |
| Publication／cutover | reuse active consumer adoption | reuse | zero required scopes仍publish／catalog commit |
| Status／idempotency | new Flat request record | adapt Root request record | common API envelope |
| Failure／timeout | reuse Server cleanup | reuse Root／Branch／Leaf cleanup | partial dispatch bounded cleanup |
| Restart／shutdown | adapt selected coordinator fence | reuse | old request GET 404；abort／close idempotent |

每個 `reuse` 都需要 direct regression test；existing full-suite green 不自動滿足延伸流程。

## 5. 設定契約

### 5.1 型別化設定

Autonomous profile：

```yaml
federated_learning:
  orchestration:
    mode: flat
    participant_source: static
  topology:
    strategy: static
    config_file: ./topology/flat.yaml
  training_trigger:
    degradation:
      enabled: false
    private_api:
      enabled: true
```

Data-owning Client：

```yaml
federated_learning:
  client:
    training_data:
      collection_trigger: consumer_subscription
```

本 Slice 的 strict settings 只接受 `consumer_subscription`。`private_api` collection trigger 與其 profiles
由 Slice 2 在完整 runtime semantics、API、callback、storage 與 cleanup 同時可用時加入；Slice 1 不先接受
尚未實作的 configuration。

### 5.2 Profile 矩陣

| Profile | 必要設定 | 禁止設定 |
| --- | --- | --- |
| Local | `runtime.mode: local` | federated orchestration／topology／trigger |
| Client-only | Client＋`consumer_subscription` | orchestration／topology／training trigger |
| Assigned Branch | Server＋Client＋`consumer_subscription` | autonomous orchestration／main topology／trigger |
| Production flat | `flat + monitor_scopes`、degradation enabled | topology、hierarchy strategy、training private API |
| Static flat | `flat + static`、topology、training private API | hierarchy strategy |
| HFL Root | `hierarchical + static`、topology／strategy、至少一種 trigger | `monitor_scopes` |

另外拒絕 orchestration 沒有 Server、trigger 沒有 orchestration、all triggers disabled、unknown enum、
topology shape／mode mismatch、static mode 缺 topology、hierarchical 缺 strategy，以及 legacy
topology-presence settings。

### 5.3 一次性遷移

| Profile／generator | 遷移內容 |
| --- | --- |
| `PyMTLF/config/fl-server.yaml` | explicit production flat＋degradation |
| `PyMTLF/config/fl-server-hierarchy.yaml` | explicit hierarchical static＋FedProx |
| `PyMTLF/config/fl-server-client.yaml` | assigned engine settings＋`consumer_subscription`；無 autonomous config |
| `PyMTLF/config/fl-client.yaml` | Client＋`consumer_subscription`；無 autonomous config |
| distributed FL generator | explicit flat Server／Client settings |
| HFL generator | Root explicit owner；Branch／Leaf non-owner＋`consumer_subscription` |

不保留 implicit inference、aliases 或 dual semantics。Workspace 中不加入 `local_file` setting、path、digest
或 loader。

## 6. 靜態 flat 拓樸與 discovery

### 6.1 拓樸 parser

```yaml
version: 1
clients:
  - nf_instance_id: 11111111-1111-4111-8111-111111111111
    scope:
      tracking_areas:
        - plmn_id: {mcc: "466", mnc: "92"}
          tac: "001101"
```

Flat topology 要求至少兩個 Clients、unique UUIDv4 identities、每個 Client 至少一個合法 unique TAI、
non-self target 與 no unknown fields。Canonical form 依 NF ID／PLMN／TAC 排序並計算 stable SHA-256。
Product schema 不寫死四個 Clients，也不要求不同 Clients 的 TAI globally disjoint。

### 6.2 參與者 scope 組合

| 值 | 權威來源 | 使用者 |
| --- | --- | --- |
| family event、target、non-area filter、interoperability | current model catalog | flat planner／Training request |
| Client ID、TAI | immutable topology snapshot | planner／NRF resolver |
| containing NWDAF ID | runtime context | self-target validation |
| service／capability requirement | PyMTLF FL contract | Go NRF relay／returned profile validation |

若 family filter 已有 area source且無唯一 composition 語意，request 在 discovery 前失敗。Participant scope
key 包含 family semantics、exact Client identity 與 topology TAI，但不代表 active Monitor consumer，也不加入
`required_cutover_scope_keys`。

### 6.3 Exact discovery 與 dispatch fencing

每個 configured Client 必須驗證 exact returned ID、non-self、REGISTERED Training service、FL Client
capability、event、interoperability與TAI。Missing、duplicate、mismatch或只有NRF decoy都失敗，不 substitute
其他 candidate。所有 participants 完成 planning／discovery 後才 dispatch；mid-dispatch failure 使用
existing cleanup 回收 subscriptions、workspaces、callbacks 與 registry reservation。

HFL resolver 保持 exact identity、service、capability、event、interoperability，且 query／profile check
不使用TAI。

## 7. 已蒐集資料集的前置條件

`DatasetCoordinator` 繼續擁有 external preparation job、queue、callback、timeout、cancel、generation fence
與 snapshot retention。本 Slice 不新增 dataset source adapter。

Static manual preparation 與 production flows 共用：

1. FL Client 從 Training Subscribe 取得 event、filter、target、single absolute window與`minNumSamples`。
2. DatasetCoordinator 只匹配現有 `TrainingDataDescriptor`；本 Slice 的 descriptor 由
   `consumer_subscription` path 發布。
3. ADRF retrieval 成功時 freeze returned records；ADRF unavailable 時依既有 policy 使用 MongoDB。
4. Record event／scope／target與requested absolute window必須相符；不得移動window、讀local file或造假
   descriptor／ADRF record。
5. Frozen `DatasetSnapshot` 供所有 rounds 與 final validation 重用；preparation 後不重新查詢可變資料。
6. Assigned Branch 有 children 時走 delegation，local dataset provider 不得被呼叫。

Missing descriptor、records、window coverage 或 sample count 是 data-owning Client／Leaf preparation failure；
它不改變 participant planning 結果，也不授權 fallback 到其他來源。Slice 1 tests 必須直接證明 static
manual flow 在 prepared collected fixture 成功，並在 missing collected precondition 時 bounded
failure／cleanup。

Training／validation split保留purged chronological semantics；`random_seed`只固定fitting與shuffle，不改成
random split。

## 8. Flat、HFL 與 manual 生命週期

### 8.1 Flat 協調器

`FlatFLCoordinator`負責trigger mapping、static／monitor participant planning、manual request idempotency、
single-active、failure latch、terminal TTL、execution invocation與snapshot projection。

`FLServerEngine` input 從隱含 `RetrainIntent` consumption 改成 explicit execution request。既有 degradation
入口由 Flat coordinator 轉接；Server engine 不再自行讀 AccuracyPolicy queue。

### 8.2 HFL adapter

`FLRootCoordinator`接受common initiation／manual methods，保留Root request、plan、assignment、failure
latch、TTL與hierarchy execution ownership。Root只由explicit hierarchical config建立。

Manual HFL的triggering scope為`None`、required cutover scopes為空；existing zero-scope publication完成
catalog commit。本 Slice保留FedProx assignment semantics；FedAvg variant deferred to Slice 3。

### 8.3 共用訓練 API

```http
POST /internal/v1/federated-learning/training-requests
GET  /internal/v1/federated-learning/training-requests/{requestId}
```

POST body：

```json
{"requestId":"canonical-UUIDv4","modelFamilyId":"ue-communication-default"}
```

禁止mode、participant、topology、strategy、collection trigger、profile、window、URL或artifact override。
Create 回 `202 Accepted`＋`Location`；same ID／body replay；unknown family／request `404`；body／active conflict
`409`；generation unavailable／closing `503`；malformed body `400 ProblemDetails`。

Status 保存 request／family、mode、participant source、trigger source、state、progress、candidate digest 與 bounded
failure fields；不得洩漏artifact URL、peer endpoint或credential。舊
`/internal/v1/hierarchical-fl/training-requests`直接移除；PyMTLF tests與resources caller同步遷移。

### 8.4 Degradation gate、restart 與 cleanup

Federated degradation disabled時，AccuracyPolicy需atomic discard／complete queued intent與family in-flight
marker，避免stale suppression race；local mode policy evaluation不變。

Generation reset／shutdown順序：停止new initiation、abort coordinator request、abort participant engines／
callbacks、fence publication、release registry、close owned clients。Abort／close idempotent，restart後old
training request GET回`404`。

## 9. 預計修改檔案

### 9.1 `PyMTLF/`

- `src/py_mtlf/config.py`：orchestration／training trigger、`consumer_subscription` marker與strict matrix；
- `src/py_mtlf/app.py`：engine construction、zero／one selected coordinator、route gating與shutdown；
- `src/py_mtlf/core/fl_orchestration.py`：common initiation／selection／snapshot contracts；
- `src/py_mtlf/core/fl_topology.py`：mode-aware parser、static flat planner、canonical hash；
- `src/py_mtlf/core/fl_server.py`：explicit execution input、typed participants與manual validation；
- `src/py_mtlf/core/fl_root.py`：common adapter；
- `src/py_mtlf/core/accuracy_policy.py`、`ml_model_monitor.py`：degradation gate與atomic discard；
- `src/py_mtlf/core/publication.py`：cutover terminology與zero-scope regressions；
- `src/py_mtlf/api/federated_learning.py`、`wire/federated_learning.py`：generic private boundary；
- remove old HFL-specific API／wire implementation與tests；
- committed configs與direct tests。

`dataset.py`、`training_data.py`與`fl_client.py`只在participant／cutover terminology或direct regression需要時
最小修改；本 Slice不加入source selector、local payload discriminator或collection manager。

### 9.2 `nwdaf-resources/`

- distributed／hierarchical generators：explicit owner、trigger與`consumer_subscription` settings；
- preflight／support checks：new Settings 與 topology parser；
- hierarchical runner：generic training endpoint／status fields；
- active README／config examples不再產生legacy settings。

本 Slice 不加入 private collection caller、support SMF changes、callback replay、controlled paired profiles
或 numerical comparison。

## 10. 實作檢查點

1. 建立 characterization tests 與 initial conformance map，固定 production flat、HFL、local、collected dataset
   與publication baselines。
2. 實作typed settings、profile migration與zero／one coordinator construction。
3. 實作common initiation、participant／cutover separation與degradation gate。
4. 實作static flat topology、scope composition、exact discovery與dispatch fencing。
5. 實作Flat coordinator與manual lifecycle，讓Server只接受explicit execution input。
6. 適配Root common contract、generic API、old route removal與active caller migration。
7. 執行 focused／full verification、mandatory initial review、test-first remediation 與 fresh-read conformance
   gate，保留unstaged diff供user review。

每個checkpoint先建立或確認deterministic failing test。若需要提前加入Slice 2 collection behavior、改動Go
NWDAF或改變standard-shaped contract，停止進decision gate。

## 11. 直接驗證矩陣

### 11.1 設定與建構

- legal profile matrix與forbidden combinations；
- mode／topology shape、relative topology path、unknown／extra fields；
- committed profiles 與 resources generators 可由 new Settings load；
- Local、Client-only、Assigned Branch建立zero coordinator；flat／HFL恰一個；
- Branch保留assigned execution但不掛top-level route、不消耗degradation intent；
- `private_api` collection trigger與legacy `local_file`在本Slice deterministic rejected。

### 11.2 拓樸與 discovery

- min two、UUIDv4、duplicate ID／TAI、self、canonical order／hash；
- family semantics＋topology TAI scope composition與area ambiguity rejection；
- exact identity、scope、capability、service、interoperability與NRF decoy rejection；
- all-or-nothing pre-dispatch validation與partial dispatch cleanup；
- HFL profiles沒有`trackingAreaList`仍resolve，query不帶TAI。

### 11.3 已蒐集資料集 regression

- consumer-delivered descriptor＋ADRF records的static manual preparation success；
- missing descriptor、ADRF／Mongo unavailable、window mismatch、samples不足的bounded failure；
- absolute requested window不re-anchor；
- one immutable snapshot跨round／final validation reuse；
- Assigned Branch delegation不讀dataset；
- purged chronological split、validation ratio與random seed semantics不變。

### 11.4 Manual、生命週期與 publication

- flat／HFL route gating、POST＋GET、`202`＋`Location`、same-ID replay；
- conflicting body、single-active、unknown family／request、unavailable generation；
- failure latch、terminal TTL、restart 404與bounded detail；
- request 不能 override deployment settings，status 不洩漏 URL／peer／credential；
- manual no fake trigger，仍保存aggregate／per-participant evidence；
- zero cutover scopes 完成 ADRF publication／catalog commit，不呼叫 consumer adoption；
- degradation保留cutover與trigger-specific validation；
- preparation／round／validation timeout、late callback、abort、cleanup與registry release；
- old HFL-specific route 為 `404`，active resources caller 只使用 generic route。

### 11.5 必要指令

```bash
# PyMTLF/
.venv/bin/pytest -q \
  tests/test_config.py \
  tests/test_runtime_modes.py \
  tests/test_dataset.py \
  tests/test_training_data.py \
  tests/test_fl_client.py \
  tests/test_fl_topology.py \
  tests/test_fl_hierarchy_discovery.py \
  tests/test_fl_server.py \
  tests/test_fl_root.py \
  tests/test_fl_experiment.py \
  tests/test_publication.py \
  tests/test_federated_learning_api.py

.venv/bin/pytest -q
.venv/bin/ruff check src tests

# workspace root
PyMTLF/.venv/bin/pytest -q \
  nwdaf-resources/deployments/hierarchical_fl/checks \
  nwdaf-resources/deployments/distributed_fl/checks

PyMTLF/.venv/bin/ruff check \
  nwdaf-resources/deployments/hierarchical_fl \
  nwdaf-resources/deployments/distributed_fl
```

本 Slice 不要求 local multi-process success run；caller migration 仍需 unit／config checks。所有 code／
script commands 依 workspace policy 以 elevated permission 執行。

## 12. 規範符合性對照表

| ID | 要求 | Production owner | 實作與直接證據 | 狀態 |
| --- | --- | --- | --- | --- |
| S1-CFG-01 | explicit mode／participant source／training trigger strict matrix | `config.py` | `test_config.py` profile matrix與`test_runtime_modes.py` construction | 已滿足 |
| S1-CFG-02 | Client marker 固定 `consumer_subscription`；拒絕 private collection 與 local file | `config.py` | `test_config.py` marker與negative cases | 已滿足 |
| S1-CFG-03 | committed profiles／generators一次遷移 | configs／resources | committed config load與兩組resources checks | 已滿足 |
| S1-OWN-01 | engine presence與autonomous owner分離；zero／one coordinator | `app.py` | Local／Client／Branch／flat／HFL construction tests | 已滿足 |
| S1-OWN-02 | Assigned Branch不autonomous但保留assigned execution | app／Branch | route absence與real Branch delegation tests | 已滿足 |
| S1-INT-01 | participant selection、cutover與optional trigger typed separation | orchestration contract | `test_fl_flat.py` manual／degradation mapping | 已滿足 |
| S1-INT-02 | disabled degradation atomic discard；local policy不變 | policy／monitor | `test_accuracy_policy.py` atomic discard與runtime dispatch tests | 已滿足 |
| S1-TOP-01 | static flat schema、validation、canonical hash | topology | `test_fl_topology.py` schema、self、order與digest tests | 已滿足 |
| S1-TOP-02 | family＋topology TAI composition；area ambiguity拒絕 | planner | `test_fl_flat.py` static selection與area conflict tests | 已滿足 |
| S1-DIS-01 | exact NRF resolve且不substitute decoy | resolver | `test_fl_server.py` exact profile、TAI、duplicate與dispatch fence tests | 已滿足 |
| S1-DIS-02 | HFL resolution不使用TAI | hierarchy resolver | `test_fl_hierarchy_discovery.py` no-TAI query／profile tests | 已滿足 |
| S1-DATA-01 | static manual重用consumer-collected absolute-window snapshot | dataset／Client | real Client→DatasetCoordinator collected fixture test | 已滿足 |
| S1-DATA-02 | missing collected precondition bounded failure；無source fallback | dataset／Client | missing descriptor與ADRF／Mongo window fault tests | 已滿足 |
| S1-DATA-03 | snapshot跨round／validation reuse；split semantics不變 | dataset／builder | real round＋validation snapshot reuse與split regressions | 已滿足 |
| S1-FLT-01 | Flat coordinator owns initiation；Server只execution | Flat coordinator／Server | coordinator initiation與Server explicit execution tests | 已滿足 |
| S1-FLT-02 | static manual完成prepare→round→validate→publish→cleanup | Flat coordinator／Server | `test_static_manual_runs_server_publication_and_cleanup_without_cutover` | 已滿足 |
| S1-HFL-01 | Root common adapter且FedProx lifecycle不變 | Root | `test_fl_root.py`與hierarchy Server regressions | 已滿足 |
| S1-API-01 | generic POST／GET、status／errors、no override | API／wire | API contract、runtime override、actual unavailable／restart tests | 已滿足 |
| S1-API-02 | old route移除、active caller遷移 | API／resources | runtime `404`與resources source／runner checks | 已滿足 |
| S1-LIF-01 | idempotency、single-active、latch、TTL、restart fence | coordinators／registry | Flat／Root lifecycle與actual API restart `404` tests | 已滿足 |
| S1-LIF-02 | timeout／partial dispatch／shutdown cleanup | coordinators／engines | Server／Root／Client fault-injection與shutdown tests | 已滿足 |
| S1-VAL-01 | manual no fake trigger；aggregate／per-participant evidence | validation | actual final-validation manual／degradation parameterized test | 已滿足 |
| S1-PUB-01 | zero cutover仍publish／commit；degradation cutover不變 | publication | static manual component flow與publication nonzero regressions | 已滿足 |
| S1-REG-01 | production flat／HFL／local baselines保留 | all owners | focused matrix與520-test full suite | 已滿足 |
| S1-VER-01 | focused／full PyMTLF與resources checks通過 | repositories | §11.5全部五項指令通過 | 已滿足 |
| S1-REV-01 | mandatory review、remediation、fresh-read與language pass | all diffs／docs | full diff review、test-first findings、fresh-read與全文檢查 | 已滿足 |

### 12.1 最終驗證與審查紀錄

2026-08-26 的最終驗證結果如下：

- PyMTLF focused matrix：295 passed；
- PyMTLF full suite：520 passed、2 skipped；
- PyMTLF ruff：通過；
- `nwdaf-resources` hierarchical／distributed checks：39 passed；
- `nwdaf-resources` ruff：通過。

必要審查確認並以deterministic tests修正exact TAI coverage、duplicate NRF candidate、admission／
policy ownership、cutover family lookup、failure-latch queue cleanup、absolute window clipping與executor cleanup等
Slice 1缺失；每項均完成focused regression與targeted follow-up review。`NWDAF/`、`PyAnLF/`、`nrf/`與
`adrf/`維持read-only，未新增Go package、dataset source adapter或Slice 2 collection behavior。

Slice 2 `private_api` collection、Go NWDAF relays、UPF callback、storage／descriptor與local cross-process
evidence不得在本 Slice標為Satisfied。Slice 3 FedAvg comparison與multi-process numerical evidence同樣deferred。

## 13. Review 與完成閘門

### 13.1 初次 review

Focused tests 通過後立即 review 完整 Slice diff：

- production changes全在本Slice範圍；
- baseline每個stage有direct disposition與evidence；
- config不先接受Slice 2 behavior；
- ownership、generation、timeout、cleanup與concurrency有direct tests；
- standard-shaped Training／NRF boundary維持local OpenAPI與existing relay semantics；
- 沒有新增unplanned package或第二套coordinator／dataset state machine；
- `NWDAF/`、`PyAnLF/`、`nrf/`、`adrf/`未被修改。

Confirmed in-scope finding依development policy test-first remediation並做targeted follow-up review。

### 13.2 最終閘門

交付user review前：

1. 從disk完整重讀development policy與本Slice；
2. 依 current text 重建 conformance map；
3. 執行§11.5 commands；
4. 對每個Satisfied item定位production path、direct test與command result；
5. 檢查 affected repository status、完整 unstaged diff 與 unrelated changes；
6. 完整重讀本文件並與父計畫／同系列 sibling比對，完成繁體中文language consistency pass；
7. 保持`Ready for User Review`，不stage、不commit、不標Completed。

交付必須列affected repositories、diff summary、tests、open Slice 2／3與testbed gaps、conformance state與
unrelated changes。User review 確認後才另行準備 repository-separated commit proposal。

## 14. 明確延後項目

- MTLF `private_api` collection與Go NWDAF UDM／SMF／ADRF relays：`future-phase handoff`，Slice 2 owner；
- HFL FedAvg、controlled four-trainer comparison、production flat／HFL multi-process reruns：
  `future-phase handoff`，Slice 3 owner；
- real SMF／UPF、testbed VM／full-core experiment：`integration verification gap`；
- dynamic HFL、flat monitor manual snapshot、Root collection fan-out、arbitrary-depth hierarchy、topology hot
  reload與HFL TAI orchestration：parent non-goals。

Slice 1只有在implementation、direct verification、mandatory review與user review完成後才進commit proposal；
matching collected fixture 成功不代表 Slice 2 private collection 或 testbed integration 已完成。
