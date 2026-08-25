# Explicit Flat and Hierarchical Orchestration Configuration Detailed Plan

日期：2026-08-25

狀態：Requirements Reviewed；Plan Approved；implementation 尚未開始

相關文件：

- [Hierarchical NWDAF Federated Learning Implementation Plan](../Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)
- [Slice 3 Root Initiation, Static Topology and Assignment Detailed Plan](../Slice%203%20Root%20Initiation%2C%20Static%20Topology%20and%20Assignment%20Detailed%20Plan.md)
- [Slice 8 Multi-process E2E and Regression Closure Detailed Plan](../Slice%208%20Multi-process%20E2E%20and%20Regression%20Closure%20Detailed%20Plan.md)
- [NWDAF Development Policy](../../../development_policy.md)
- [Release 18 Nnrf NF Management OpenAPI](../../../../specs/openapi/TS29510_Nnrf_NFManagement.yaml)
- [Release 18 Nupf Event Exposure OpenAPI](../../../../specs/openapi/TS29564_Nupf_EventExposure.yaml)
- [Release 18 Nnwdaf ML Model Training OpenAPI](../../../../specs/openapi/TS29520_Nnwdaf_MLModelTraining.yaml)

---

## 1. 目的與審查結論

本計畫定義 PyMTLF 如何以顯式 configuration 選擇 autonomous top-level flat 或
hierarchical federated learning orchestration，並為受控實驗提供可重現的 static
participant topology。

需求審查確認原始方向成立：

- 保留由 Model Provision、Model Monitor registration、accuracy degradation 與 NRF exact
  discovery 組成的 production-like flat FL 閉環；
- 保留既有 Root static topology、Branch／Leaf assignment 與 hierarchical lifecycle；
- 新增不依賴 Monitor scopes 選擇 participants 的 controlled static flat FL；
- 新增不依賴 degradation 的 manual static flat／hierarchical experiments；
- 讓 resolved configuration 與 run evidence 明確記錄 mode、participant source、trigger source
  與 topology identity。

依本 workspace 的實作與實驗資源審查後，計畫另確認八項必要修正：

1. `server` engine presence 不等於 autonomous top-level ownership。Branch PyMTLF 同時具有
   upper-tier Client 與 lower-tier Server engines，但不得因此被迫選擇 flat 或建立 Root。
2. 現行 flat local fitting 是 FedAvg；既有 HFL profile 透過 assignment 使用 FedProx。第一個
   topology-only comparison 必須讓兩邊都使用 FedAvg，否則不能宣稱相同 Leaf updates 或數值等價。
3. Static participant scopes 與 Model Monitor adoption scopes 是不同狀態。不得把 static
   participants 塞進既有 `RetrainIntent.active_scope_keys` 後誤判成 required cutover consumers。
4. `nwdaf-resources` 已有 four-Leaf／two-Branch aggregation profile、真實 NRF／ADRF 與多程序
   evidence harness，因此它是 paired comparison 的正式 owner，不只是事後驗證工具。
5. Static topology 只保證 participant selection，不保證 data-owning FL Client／Leaf 已有
   PyAnLF、Training Data Descriptor 或 ADRF records。Manual static training 必須可由各 Client
   明確選擇本地 UPF notification dataset，否則仍無法在沒有 collection chain 的 testbed profile
   完整執行。
6. Release 18 `DataAvReq.timeWindows` 表示資料樣本的絕對時間區間；現行 FL Server 會在 preparation
   Subscribe 中指定一個區間。Pinned historical local file 不能假裝滿足 wall-clock 絕對區間，必須把
   該區間的長度明確 re-anchor 到每個 Client 自身資料集的最新 observation，並在 evidence 區分
   requested 與 effective window。HFL Leaf 的直接發送者是 assigned Branch 的 lower-tier FL Server，
   因此 paired profile 必須固定每個 direct Server 的 window length，而不能只設定 Root。
7. 現行 `TrainingDatasetBuilder` 已使用 purged chronological training／validation split，而非隨機
   split；`random_seed` 則控制 NumPy、PyTorch 與 training DataLoader shuffle。Controlled comparison
   必須顯式固定 split settings 與 seed，不能只依賴 defaults。
8. Flat 與 HFL 不共用 TAI orchestration contract。Production／static flat 繼續以 TAI 建立
   participant scope、執行 NRF discovery 並形成 Training Subscribe；HFL 維持 explicit
   Branch／Leaf identities、capability 與 interoperability resolution，不新增 per-node TAI topology、
   discovery requirement、filter synthesis 或 local dataset TAI validation。

這是既有 production flows 的 orchestration-selection extension。它不新增 NF type、不修改
Release 18 public SBI schema，也不把實驗 topology role 寫入 NRF profile。

## 2. 審查基準與現況證據

### 2.1 審查版本

本次需求審查使用下列乾淨 working trees：

| Repository | Revision | 用途 |
| --- | --- | --- |
| `PyMTLF/` | `e9aa2235b6b4adc1d9d778b6cfdf23645fc622ec` | production config、orchestration、lifecycle 與 tests |
| `nwdaf-resources/` | `39ced284561e542c50da5fa7e83830aae4517821` | flat／HFL 多程序 fixtures 與 evidence |
| `NWDAF/` | `3279891689dd9b54737ffe08dc18b9db72ec57b4` | containing-NWDAF capability 與 private relay baseline |
| `PyAnLF/` | `6a4d94ad3cc6f66dac55ea921772d731e4b71371` | production Model Provision／Monitor regression baseline |
| `nrf/` | `0dd4024d4ab75b6630e04901968228b9b9718cf5` | exact-instance discovery baseline |
| `adrf/` | `905f0599f68fe389bba14ed56db0ef9abeab5ccd` | data retrieval 與 publication baseline |

### 2.2 已確認的實作行為

| 現有行為 | 證據與結論 |
| --- | --- |
| Topology presence 選擇 Root | `create_app()` 只要看到 `federated_learning.topology` 就建立 `FLRootCoordinator`；沒有 explicit mode。 |
| Flat participant selection | `FLServerEngine` 從 `RetrainIntent.active_scopes` 逐一以 `consumerId` 與 TAI 執行 NRF exact-instance discovery。 |
| Degradation intent owner | Monitor notification 有 trigger 時，依 `fl_root` 是否存在選 Root，否則選 `fl_server`。 |
| Manual API | 只有 `/internal/v1/hierarchical-fl/training-requests`，router 直接呼叫 `app.state.fl_root`。 |
| HFL lifecycle | `FLRootCoordinator` 已有 idempotency、single-active、failure latch、terminal status TTL、generation reset 與 cleanup。 |
| Shared single-active slot | `FLExperimentRegistry` 已仲裁 Root、flat Server、Branch 與 Leaf 的 process-local experiment ownership。 |
| Zero-scope publication | `PublicationCoordinator` 已在 `required_cutover_scopes` 為空時直接完成 catalog commit；HFL manual flow 已使用這條路徑。 |
| Static HFL profile | `nwdaf-resources/deployments/hierarchical_fl` 已有 `aggregation` profile：一個 Root、兩個 Branches、四個 Leaves。 |
| HFL NRF resolution | `HierarchyNodeResolver` 以 exact NF instance ID、Training service、analytics event、FL capability 與 model interoperability resolve Branch／Leaf；query 與 local requirement check 都不使用 `trackingAreaList`。現有 runner 的 Leaf TAI 是 collection-role fixture metadata，不是 hierarchy selection input。 |
| Current comparison algorithm | Flat Client 在沒有 hierarchy assignment 時以 `proximal_mu=None` 訓練；HFL Leaf 從 assignment 取得 positive FedProx `proximal_mu`。 |
| FL Client training data | `FLClientEngine` preparation 固定呼叫同一個 `DatasetCoordinator`；現行 coordinator 必須先匹配本地 `TrainingDataDescriptor`，再從 ADRF 或 MongoDB freeze snapshot。Static topology 與 private trigger 都不會自行產生 descriptor 或 records。 |
| Training data time window | 每個 `FLServerEngine._create_preparation()` 以自身 `preparation_data_window_seconds` 在 `mLModelTrainInfos[].dataAvReq.timeWindows` 送出一個 `[startTime, stopTime]`；flat Client 的 direct sender 是 top-level FL Server，HFL Leaf 的 direct sender 是 assigned Branch 的 lower-tier FL Server。`FLClientEngine` 目前取第一個區間，Release 18 將它定義為資料樣本的絕對時間區間。 |
| Training／validation split | `TrainingDatasetBuilder` 先依 observation time 排序、建立 sliding windows，再以最早的 `validation_ratio` 作 validation、保留 purge gap、其餘較新的 samples 作 training；split 本身不使用 random seed。 |
| Fitting randomness | `FederatedTrainer` 以 client `random_seed` 設定 NumPy、PyTorch、deterministic algorithms 與 DataLoader generator；現有 committed Client profiles 明確使用 `42`。 |

### 2.3 已確認的需求與非需求

| 項目 | 審查結果 |
| --- | --- |
| Explicit flat／hierarchical selection | 必要；不得再從 topology presence 推論。 |
| Static flat participant set | 必要；是 controlled comparison 的主要 production 缺口。 |
| Local-file training data source | 必要；讓 static manual flat Client／HFL Leaf 不依賴 PyAnLF collection 與 ADRF training-data retrieval。 |
| Local-file window re-anchoring | 必要；只對 `local_file` 把單一 requested interval 的長度 re-anchor 到本地 `Tmax`，`collected` 仍使用絕對區間。 |
| Random training／validation split | 不需要；保留現行 purged chronological split，paired profiles 顯式固定 `validation_ratio` 與 `random_seed`。 |
| Generic manual management API | 必要；但必須只有一個 selected top-level coordinator owner。 |
| 新 publication state machine | 不需要；重用既有 publication 與零 cutover-scope completion。 |
| 新 Go hierarchy API | 不需要；現有 Training relay 與 containing-NWDAF context 足夠。 |
| 新 NRF topology role | 不需要；仍使用 standard FL capability 與 exact-instance discovery。 |
| HFL per-node TAI contract | 不需要；HFL topology、assignment、NRF resolution 與 `local_file` preparation 都不新增 TAI requirement。 |
| 四個 Clients 寫死在 product schema | 不需要；四個是第一個 comparison fixture，product contract 支援至少兩個 static Clients。 |
| 暫時保留 implicit config inference | 不需要；configs 與已知 active callers 都由 workspace repositories 管理，採一次性 migration。 |

## 3. 術語與責任邊界

Configuration、API、logs 與 evidence 使用下列語意：

| 概念 | Flat FL | Hierarchical FL |
| --- | --- | --- |
| Autonomous top-level coordinator | FL Server | Root |
| 中間 aggregation node | 不適用 | Branch |
| Data-owning trainer | FL Client | Leaf |
| 共用抽象名稱 | Coordinator／Participant／Node | Coordinator／Participant／Node |

固定規則：

- flat 文件與 runtime evidence 不把 Server 稱為 Root，也不把 Client 稱為 Leaf；
- Root／Branch／Leaf 只表示 hierarchical execution 的 topology position；
- `FL_SERVER`、`FL_CLIENT`、`FL_SERVER_AND_CLIENT` 仍是 registered capability；
- Branch 在 process responsibility 上同時是 upper-tier Client 與 lower-tier Server，但不是
  autonomous top-level coordinator；
- `federated_learning.server`／`client` 決定 engines 是否存在，`orchestration` 才決定此
  PyMTLF 是否可自主建立 top-level training；
- topology position 只存在於單次 request／plan，不是永久 deployment role。

## 4. 顯式 configuration contract

### 4.1 自主頂層選擇

只有可自主開始 training 的 PyMTLF profile 配置 `orchestration`：

```yaml
federated_learning:
  orchestration:
    mode: flat
    participant_source: monitor_scopes
```

第一版合法值：

- `mode`: `flat` 或 `hierarchical`；
- `participant_source`: `monitor_scopes` 或 `static`。

`topology` 保留既有 owner 與 path-resolution semantics，不把 path 搬進
`orchestration`：

```yaml
federated_learning:
  topology:
    strategy: static
    config_file: "./topology/flat-topology.yaml"
```

相對路徑繼續由 main config 所在目錄解析。這可重用現有 loader、tracked HFL configs 與
generated runtime configs，避免同一 topology path 有兩個 configuration sources。

### 4.2 合法 profile matrix

| Profile | Orchestration | Participant source | Topology | Strategy | Trigger |
| --- | --- | --- | --- | --- | --- |
| Local | 無 | 不適用 | 無 | 無 | federated trigger 不適用；既有 local retraining 不變 |
| Client-only | 無 | 不適用 | 無 | 無 | federated trigger 全部關閉 |
| Assigned Branch | 無 | 由 upper assignment 決定 | main config 無 | 由 assignment 決定 | 全部關閉 |
| Production flat Server | `flat` | `monitor_scopes` | 無 | 現行 flat FedAvg | degradation |
| Controlled flat Server | `flat` | `static` | static flat，必填 | 第一版 FedAvg | private API；可另啟 degradation |
| Hierarchical Root | `hierarchical` | `static` | static hierarchy，必填 | 必填 | degradation、private API 或兩者 |

第一版不支援：

- `hierarchical + monitor_scopes` dynamic grouping；
- `flat + monitor_scopes + private_api`。Manual flat V1 固定使用 static participants，避免在
  request 時另定義 Monitor snapshot semantics；
- autonomous orchestration 沒有 `server` engine；
- Assigned Branch 或 Client-only profile 啟用 federated training trigger。

下列情形必須 deterministic startup failure：

- `hierarchical` 或 `flat + static` 缺少 topology；
- `flat + monitor_scopes` 同時配置 topology 或 hierarchy strategy；
- topology shape 與 explicit mode 不符；
- `hierarchical` 缺少 strategy；
- 未知 mode、participant source、topology version 或 strategy；
- `training_trigger` 啟用但沒有 autonomous `orchestration`；
- autonomous `orchestration` 的所有 initiation sources 都關閉。

### 4.3 Trigger selection

Topology source 與 trigger source 分開設定：

```yaml
federated_learning:
  training_trigger:
    degradation:
      enabled: true
    private_api:
      enabled: false
```

固定規則：

- `federated_learning.training_trigger` 只管理 federated autonomous top-level initiation，不得改變
  local mode 既有 AccuracyPolicy／DatasetCoordinator retraining；
- `degradation.enabled: true` 才允許 AccuracyPolicy 的 degradation result 進入 federated
  autonomous coordinator；
- 關閉 federated degradation 時不得建立或留下之後可能被其他 request 誤取出的 FL retraining
  intent；實作可在 federated enqueue／dispatch boundary gate，但不得關閉 local policy evaluation；
- `private_api.enabled: true` 才掛載 manual request／status API；
- static profile 可同時啟用兩種 trigger，但仍由同一 selected top-level coordinator 與 shared
  registry 維持 single-active；
- manual initiation 沒有 Monitor adoption scopes；
- degradation initiation 可攜帶 Monitor adoption scopes，但 static participant selection 仍固定來自
  topology，不得改回 monitor-derived participants。

### 4.4 一次性 config migration

本 workstream 位於協作 feature branches，已知 configs 與 active callers 都可同步更新。沿用先前
role-oriented config refactor 的 strict migration 決策：

- 不保留 topology-presence legacy inference；
- 不建立 silent alias、deprecation fallback 或同時接受兩套 selection semantics；
- 更新 `PyMTLF/config/*.yaml`、PyMTLF config fixtures、`nwdaf-resources` config generators 與
  preflight tests；
- `fl-server.yaml` 明確成為 production flat top-level profile；
- `fl-server-hierarchy.yaml` 明確成為 hierarchical Root profile；
- `fl-client.yaml` 與 assigned `fl-server-client.yaml` 不配置 autonomous orchestration；
- strict `extra="forbid"` 與 cross-field validators 對舊或矛盾 configuration fail fast。

### 4.5 FL Client training data source

每個可能執行 local fitting 的 FL Client engine 明確配置 training data source。此設定屬於
data-owning PyMTLF，不屬於 Root／Server topology，也不能由 private training request 傳入或覆寫：

```yaml
federated_learning:
  client:
    training_data:
      source: local_file
      path: "./datasets/leaf-a1.jsonl"
      sha256: "<lowercase SHA-256>"
```

合法 source：

- `collected`：保留現行 descriptor matching、ADRF／MongoDB retrieval 與 snapshot freeze；
- `local_file`：直接從本 PyMTLF 所在 filesystem 載入指定檔案，不建立假的
  `TrainingDataDescriptor`、`dataSub` 或 ADRF record，也不呼叫 ADRF／MongoDB training-data
  retrieval。

`source` 不設 implicit default；所有 committed FL Client profiles 在 Slice 1 一次遷移。`path` 與
`sha256` 在 `local_file` 時必填，在 `collected` 時禁止，且相對 path 以 main config 所在目錄解析；
`sha256` 是 raw file bytes 的 digest，必須在 startup 與每次 preparation freeze 前驗證，並作為 run
evidence identity。`collected` 與 `local_file` 不得互相 silent fallback；這項限制不移除
`collected` 內既有依 descriptor 與 ADRF availability 在 ADRF／MongoDB 間選擇 retrieval transport
的行為。
`collected` 缺少 descriptor／records，或 `local_file` 缺少、digest 不符、格式不合法或資料不足時，
該 Client／Leaf preparation 明確失敗。

本地檔固定為 UTF-8 JSON Lines；每個 non-empty line 是一筆 Release 18
`Nupf_EventExposure.NotificationData` shape，而不是完整 `NadrfDataStoreRecord`：

```json
{"correlationId":"dataset-a1-000001","notificationItems":[{"eventType":"USER_DATA_USAGE_MEASURES","ueIpv4Addr":"10.0.1.1","timeStamp":"2026-08-25T10:00:00Z","startTime":"2026-08-25T09:59:58Z","userDataUsageMeasurements":[{"volumeMeasurement":{"totalVolume":13000,"ulVolume":5000,"dlVolume":8000,"totalNbOfPackets":20,"ulNbOfPackets":8,"dlNbOfPackets":12},"throughputMeasurement":{"ulThroughput":"2 Mbps","dlThroughput":"4 Mbps","ulPacketThroughput":"300 pps","dlPacketThroughput":"500 pps"}}]}]}
```

本地檔格式由本計畫固定；workspace 中的 loader、fixtures 與 configs 在格式改變時一次遷移。
每筆資料至少必須符合下列 training-specific validation：

- `notificationItems` 非空，`eventType` 固定為 `USER_DATA_USAGE_MEASURES`；
- `timeStamp` 必填且帶 timezone；`startTime` 若存在也必須帶 timezone，並優先作 observation time；
- 每個 item 至少有一種 UE address；
- `userDataUsageMeasurements` 非空，且明確提供目前模型需要的六個 volume／packet counts 與四個
  throughput／packet-throughput fields；數值必須 finite、non-negative，rate unit 必須受支援；
- loader 依 observation time 排序；同 timestamp 的 volume／packet counts 相加、rates 依既有
  `TrainingDatasetBuilder` 語意平均；
- dataset file 不攜帶 scope key、TAI、NF identity、model family、request ID 或 topology role。Local
  adapter 將 frozen records 綁定到 preparation request 已確定的 participant scope；PyMTLF 不得以
  TAI 對 local-file rows 作額外篩選、內容驗證或資料來源選擇；
- Release 18 `timeWindows` 是資料樣本的絕對時間區間。`collected` 完整保留此語意，以 requested
  `[A, B]` 查詢及篩選 records；
- `local_file` 第一版要求 preparation request 恰有一個 timezone-aware interval，且 `B > A`。Client
  計算 requested span `D = B - A`，完整驗證檔案後以最新 observation time `Tmax` 建立 inclusive
  effective window `[Tmax - D, Tmax]`；
- flat Client 從 top-level FL Server 收到 span；HFL Leaf 從 assigned Branch 的 lower-tier FL Server
  收到 span。Root-to-Branch preparation 不直接決定 Leaf 的 span；第一個 paired profile 必須在
  flat Server 與所有 Branch Server 顯式固定相同 `preparation_data_window_seconds: 3600`；
- `local_file` 的 re-anchoring 是明確的 source-specific controlled profile adaptation，不宣稱本地
  samples 實際位於原始絕對 `[A, B]`。Evidence 必須同時記錄 `[A, B]`、`D`、`Tmax`、effective
  window、window 內 observation count 與 training sample count；
- `local_file` 缺少 `timeWindows`、提供多個 interval、區間長度為零、沒有合法 observation、effective
  window 為空或不滿足 `minNumSamples` 時，preparation deterministic failure；
- preparation 讀取、驗證並 freeze 一次 immutable snapshot；後續 fitting rounds 與 final validation
  重用同一 snapshot，不重新讀取可能改變的檔案。

此 source contract 同時適用 flat FL Client 與 HFL Leaf。Assigned Branch 有 Client engine 但在具有
children 的 assignment 中只 delegation／aggregation，不讀取 local dataset。Local-file training
移除的是 PyAnLF collection 與 ADRF training-record dependency；final model ADRF publication、catalog
commit 與有 active consumers 時的 PyAnLF cutover 是獨立 lifecycle，不因 source 改變。

### 4.6 Training／validation split 與 fitting determinism

本 workstream 保留現行 `TrainingDatasetBuilder` 的 purged chronological split，不新增隨機 split：

- observation 先依 timestamp 升冪排序，再產生 sliding-window samples；
- 最早的 `validation_ratio` samples 作 reference validation，至少一筆且必須保留 training samples；
- validation 與 training 間保留 `seq_length + out_seq_len - 1` 的 purge gap，避免重疊 window
  洩漏；其餘較新的 samples 作 training；
- `random_seed` 不參與 split，只固定 NumPy、PyTorch、deterministic fitting 與 DataLoader shuffle；
- production settings 保持可配置；controlled flat／HFL fixtures 必須在 generated config 明確寫出
  相同 `validation_ratio` 與 `random_seed`。第一個 paired profile 固定 `validation_ratio: 0.10`、
  `random_seed: 42` 與 `device: cpu`，不得只依賴 settings defaults；
- 同一 data-owning participant 在 flat 與 HFL run 必須產生相同 effective window、ordered samples
  與 training／validation sample counts。不同 participants 可有不同資料，但兩個 mode 中同一
  participant 的輸入與設定必須相同。

## 5. Static flat topology 與 scope contract

### 5.1 Topology 結構

Static flat topology 固定 containing NWDAF 所擁有的一個 FL Server 與至少兩個 data-owning FL
Clients。Server identity 由 containing-NWDAF context 提供，不在 topology file 重複設定。

第一版 shape：

```yaml
version: 1

clients:
  - nf_instance_id: "11111111-1111-4111-8111-111111111111"
    scope:
      tracking_areas:
        - plmn_id: {mcc: "466", mnc: "92"}
          tac: "001101"
  - nf_instance_id: "22222222-2222-4222-8222-222222222222"
    scope:
      tracking_areas:
        - plmn_id: {mcc: "466", mnc: "92"}
          tac: "001102"
```

Product schema 不寫死四個 Clients；first controlled comparison fixture 才固定 A1／A2／B1／B2。

### 5.2 權威輸入與 scope 組合

每個 training input 的 authoritative source 固定如下：

| Value | Authoritative source | Transport／consumer |
| --- | --- | --- |
| FL Server identity | containing-NWDAF context | PyMTLF startup／runtime validation |
| Client NF instance IDs | static topology snapshot | flat planner／NRF exact query |
| Participant TAIs | static topology snapshot | effective event filter／NRF query／Training request scope |
| Analytics event | current model-family descriptor | Training request／NRF requirement |
| Model interoperability | current model-family descriptor | NRF requirement／Training request |
| Non-area event filter、target UE | current model-family descriptor | Training request／dataset scope |
| Base model artifact | seed／durable current catalog | Server preparation |
| Client training data source | 每個 Client／Leaf 的 main config | Client preparation source selection |
| Local dataset bytes／digest | 每個 Client／Leaf 的 configured file／SHA-256 | local loader／frozen snapshot／run evidence |
| Requested training interval／span | 每個 direct FL Server 的 preparation time 與 `preparation_data_window_seconds` | Standard-shaped Training Subscribe `dataAvReq.timeWindows[0]`／其 direct Client preparation；HFL Leaf 由 assigned Branch 發送 |
| Local dataset anchor `Tmax` | 每個 Client／Leaf 驗證後的 configured local file | local adapter／run evidence |
| Effective local window | Client 以 requested span 與本地 `Tmax` 計算 | frozen snapshot／training builder／run evidence |
| Dataset scope binding | Server／Root 建立的 participant scope | Training request／Client local adapter |
| Endpoint 與 service identity | exact NRF result | containing Go Training relay |
| Request／process correlations | current run owner | private status／Training resources |

Static V1 只讓 topology 擁有 participant area partition。Model descriptor 擁有 event、target UE 與
非 area filter。若 descriptor 已帶 `networkArea`／`aoi`，startup 或 request validation 必須拒絕
ambiguous composition；第一版不做 replace、intersection 或 union 推論。

Planner 以 canonical model descriptor 與 topology TAI 組成 internal participant scope，並由
canonical representation 計算 stable scope key／digest。Topology 不接受 caller 自訂 scope key、
model ID、event、interoperability 或 target UE。

### 5.3 驗證與 exact resolution

Static flat 必須：

- 拒絕 empty Clients、少於兩個 Clients、duplicate identities、Server self-reference、invalid／
  duplicate TAI 與 unknown fields；
- canonicalize Client ordering 與每個 Client 的 TAI ordering，再計算 topology SHA-256；
- 以 topology Client ID、effective TAI、model event 與 interoperability 呼叫既有 exact NRF
  discovery；
- 驗證唯一 `REGISTERED` NWDAF profile、唯一 registered `nnwdaf-mlmodeltraining` service、
  `FL_CLIENT`／`FL_SERVER_AND_CLIENT` eligibility 與相符 tracking area；
- 只訓練 topology 中的 configured Clients；NRF 多出的 eligible Clients 不得改變結果；
- 不把 endpoint、subscription ID、callback URI、`mlCorreId`、model URL 或 per-run identity 寫入
  topology。

Comparison fixture 的 flat side 額外要求四個 participant TAI sets 彼此不重疊，以證明 exact
scope／discovery 並避免重複的 Flat area assignment；local dataset partition 仍只由各 Client 的
configured `path + sha256` 決定。這是 fixture acceptance，不是 product topology parser 的全域限制。

### 5.4 Participant scopes 與 adoption scopes 分離

現行 flat `RetrainIntent` 同時攜帶 participant selection 與 publication adoption scopes。Static
manual flow 不能沿用這個隱含等價。

新的 internal initiation contract 必須至少區分：

- `participant_scopes`：用來選擇 Clients、建立 preparation、訓練與 final validation；
- `required_cutover_scope_keys`：只代表現有 Model Monitor consumers 的 adoption requirement；
- `triggering_scope_key`：只在 degradation trigger 存在；
- `trigger_source`：`DEGRADATION` 或 `PRIVATE_API`；
- `request_id`：只在 caller-supplied manual request 存在。

固定語意：

- production flat degradation：participants 與 required cutover scopes 都來自 Monitor snapshot；
- static flat degradation：participants 來自 topology，required cutover scopes 來自 degradation
  intent；
- static flat manual：participants 來自 topology，required cutover scopes 為空；
- hierarchical degradation：Branch／Leaf assignment 來自 topology，required cutover scopes 來自
  degradation intent；
- hierarchical manual：assignment 來自 topology，required cutover scopes 為空。

不得建立語意不一致的 `RetrainIntent`，例如 `active_scopes` 非空但
`active_scope_keys` 為空。Production implementation 應引入明確的 orchestration initiation model
或等價 typed boundary，再由 mode-specific owner 轉成 FL Server execution input。

## 6. Hierarchical topology 與 training strategy

### 6.1 既有 hierarchy contract

既有 HFL topology shape 繼續使用 `branches`／`leaves`：

```yaml
version: 1

admission:
  mode: complete_required

branches:
  - nf_instance_id: "33333333-3333-4333-8333-333333333333"
    leaves:
      - nf_instance_id: "11111111-1111-4111-8111-111111111111"
      - nf_instance_id: "22222222-2222-4222-8222-222222222222"
```

Root、Branch、Leaf eligibility、exact NRF validation、assignment bundle、upper／lower process
mapping、complete-required admission、aggregation、validation、publication 與 cleanup contracts
維持既有語意。Flat Client 不改稱 Leaf，flat topology 不重用 hierarchy assignment bundle。

本 workstream 不把 TAI 納入 HFL contract：

- topology 與 assignment 只攜帶 explicit Branch／Leaf NF instance IDs；
- hierarchy resolver 只要求 exact identity、Training service、analytics event、role capability 與
  model interoperability，不送出或檢查 `trackingAreaList`；
- controlled HFL NRF profiles 與 model descriptor 不提供 area filter，Root／Branch 不合成 per-Leaf
  `networkArea`，Leaf 的 `local_file` loader 也不檢查 TAI；
- Release 18 registration 中選填的 area fields 不因 HFL 而成為必填。Existing generic event-filter
  transport 不在本 workstream 移除，但不得把它描述成 HFL participant selection 或 dataset partition
  contract。

### 6.2 受控比較 strategy

現行實作不能直接作 topology-only numerical comparison：

- flat Client 沒有 hierarchy assignment 時執行 FedAvg local fitting；
- HFL Leaf 從 assignment 讀取 positive FedProx `proximal_mu`。

第一個 controlled comparison 固定兩邊都使用 FedAvg：

- 現有 flat FedAvg execution 不改；
- HFL `strategy.algorithm` 新增明確 `fedavg` variant，且 `proximal_mu` 不得出現；
- 現有 `fedprox + positive proximal_mu` profile 與 regressions 繼續支援；
- paired HFL fixture 改用 `fedavg`，讓相同 seed、資料、epochs、rounds 與 random seeds 產生可比較
  的 Leaf updates；
- 第一版不新增 static flat FedProx。若未來要比較 FedProx，必須另行定義 flat Client 如何從
  common internal contract 取得 algorithm parameters。

HFL assignment contract 增加 `fedavg` legal variant。此 comparison 只支援所有 PyMTLF nodes 使用
同一新 revision，不宣稱新 Root 與舊 Leaf 的 mixed-version interoperability；原有 FedProx payload
仍維持原語意。

## 7. Manual initiation 與 status contract

### 7.1 唯一選定 owner

Application construction 必須建立零或一個 `top_level_coordinator`：

- production flat／static flat 選 flat coordinator；
- hierarchical Root 選 Root coordinator；
- Client-only／assigned Branch 不建立 autonomous coordinator。

`fl_server`、`fl_client`、`fl_branch` 與 `fl_root` 可繼續作為 mode-specific engines／owners，但
Monitor dispatch 與 private API 只能呼叫 selected `top_level_coordinator`，不得依 object presence
自行猜測 owner。

### 7.2 共用 private route

Target private API：

```http
POST /internal/v1/federated-learning/training-requests
GET  /internal/v1/federated-learning/training-requests/{requestId}
```

Create body：

```json
{
  "requestId": "<canonical UUIDv4>",
  "modelFamilyId": "ue-communication-default"
}
```

固定語意：

- request 不允許覆寫 deployment mode、participant source、topology path、strategy 或 model URL；
- request 不新增 local dataset path、training-data source 或 data-window 欄位；FL Server 繼續從
  preparation settings 產生 standard-shaped `[A, B]`，`local_file` Client 只使用其 span 作 re-anchoring；
- `modelFamilyId` 只選擇 current catalog family；
- `requestId` 是 caller idempotency key；相同 body 重送回同一 resource，不同 body conflict；
- representation 共用 `requestId`、`modelFamilyId`、`orchestrationMode`、`participantSource`、
  `triggerSource`、`state`、round progress、candidate digest 與 bounded failure fields；
- flat 可在 allocation 後提供 `processId`；hierarchical 可提供 mode-specific `planId`；
- status 使用共用 envelope 與穩定 terminal values；中間 progress state 可依 mode 不同，但 flat
  state 不得使用 Root／Branch／Leaf topology terms，caller 必須依 `orchestrationMode` 解讀；
- response 不提供 candidate URL、topology content、peer response body 或 artifact secrets；
- create 回 `202 Accepted` 與 `Location`；unknown family 回 `404`；active conflict 回 `409`；
  coordinator／generation unavailable 回 `503`；private validation error 維持 normalized `400`
  `ProblemDetails`。

現有 `/internal/v1/hierarchical-fl/...` route 直接移除，不保留 alias。已知 active callers 只有
PyMTLF tests 與 `nwdaf-resources/deployments/hierarchical_fl/scripts/run.py`，兩邊在同一 API migration
slice 更新；completed historical plans 保留舊 route 作歷史紀錄。

### 7.3 生命週期與重啟

Flat manual status 必須對齊既有 HFL management semantics：

- process-local request records 與 terminal status TTL；
- single-active reservation；
- idempotent replay、conflicting active request 與 explicit fresh request；
- bounded failure cause／detail；
- failure latch 只阻止 automatic retrigger，operator 的新 manual request 可在 cleanup 後清除 latch；
- containing-Go generation reset 清除舊 request、workspace 與 reservation；
- PyMTLF restart 不恢復舊 request，舊 status URL 回 `404`；
- shutdown、timeout、partial dispatch 與 publication failure 均進入 bounded cleanup。

Manual static final validation 沒有 `triggering_scope_key`。因此不得任意指定第一個 Client 為
triggering scope：

- degradation flow 保留 triggering-scope improvement、aggregate improvement 與 per-scope regression
  checks；
- manual flow 計算 aggregate 與每個 participant validation evidence；performance gate 啟用時使用
  aggregate improvement 與 per-scope regression checks，不套用 triggering-scope check；
- performance gate 關閉時仍記錄 `gate_would_accept` 與 reasons，再沿既有 publication path 完成。

## 8. 既有流程延伸對照

| Baseline stage | Production flat `monitor_scopes` | Static flat manual | Existing／controlled HFL |
| --- | --- | --- | --- |
| Trigger | 適配：增加 explicit degradation gate | 新增 generic private API | 適配：degradation／private API 都交給 selected Root |
| Base model | 重用 current seed／durable catalog | 重用 | 重用 |
| Participant planning | 重用 Monitor snapshot | 新增 static flat planner | 重用 static Branch／Leaf planner |
| NRF resolution | 重用含 TAI 的 exact Client resolver | 重用並由 static TAI scope 驅動 | 重用不含 TAI requirement 的 hierarchy resolver |
| Preparation | 重用 `collected` source 與絕對 window | 適配：每個 Client 明確使用 `collected` 或 re-anchored `local_file` | 適配：Branch 重用 delegation；Leaf 明確使用 `collected` 或 re-anchored `local_file` |
| Round execution | 重用 flat FedAvg | 重用 flat FedAvg | 重用；comparison profile 使用新 FedAvg strategy variant |
| Aggregation | 重用 sample weighting | 重用 | 重用 two-tier effective sample weighting |
| Final validation | 重用 degradation gate | 適配 manual no-trigger gate | 重用 hierarchy validation；manual adoption scopes 為空 |
| Publication | 重用 | 重用既有 zero-scope completion | 重用 |
| Consumer cutover | 重用 Monitor scopes | 不適用，明確記錄零 required scopes | degradation 重用；manual 不宣稱 adoption |
| Failure／timeout | 重用 | 重用 Server cleanup 並補 request status | 重用 Root／Branch／Leaf cleanup |
| Restart／shutdown | 重用 generation fencing | 適配 generic request records | 重用既有 lifecycle |
| Evidence | 補 resolved orchestration identity | 補 topology／trigger／participants | 補相同欄位並保留 hierarchy evidence |

## 9. 端到端資料流

### 9.1 Static manual flat 流程

1. PyMTLF startup 解析 explicit orchestration 與 static topology，保存 canonical snapshot 與 hash。
2. Caller 只傳 `requestId` 與 `modelFamilyId` 到 PyMTLF-private API。
3. Flat coordinator 從 current catalog 取得 event、filter、target UE、interoperability 與 base model。
4. Planner 將 model descriptor 與 topology TAIs 組成 participant scopes；required cutover scopes 為空。
5. Resolver 經 containing Go 的 NRF relay exact-resolve 每個 configured Client；任何 missing、
   capability、service 或 scope mismatch 使整個 request 失敗。
6. 每個 Client 依自身 PyMTLF config 選擇 `collected` 或 `local_file`；前者匹配 descriptor 並取得
   ADRF／MongoDB records 並保留絕對 requested window，後者載入、驗證 local UPF notification JSONL，
   將 requested span re-anchor 到本地 `Tmax` 後 freeze。兩者都產生同一個 internal dataset snapshot
   contract，且不得跨 source fallback。
7. Existing FL Server preparation／round／aggregation／final validation path 對 exact configured set
   執行 all-participant semantics，後續 rounds 重用 preparation 時 frozen 的 snapshot。
8. Accepted candidate 經既有 ADRF publication 與 durable catalog commit；publication 以
   `required_cutover_scopes = 0` 到達 `COMPLETE`。
9. Status 與 run summary 記錄 mode、participant source、trigger source、topology hash、exact Client
   identities、training data source／digest、requested span、`Tmax`、effective window、split policy、
   validation ratio、random seed、sample counts、artifact identity 與 component revisions。

### 9.2 Degradation 觸發的 static topology

當 static profile 同時啟用 degradation：

- AccuracyPolicy 的 intent 只提供 trigger identity 與 adoption scopes；
- configured topology 仍是 participant set 的唯一來源；
- publication required cutover scopes 來自 intent，不來自 topology；
- active request 或 failure latch 存在時不建立第二個 top-level execution。

### 9.3 失敗責任

| Failure point | Owner 與結果 |
| --- | --- |
| Invalid config／topology | PyMTLF startup fail fast；不掛載 management route |
| Missing current family／base artifact | selected coordinator terminal failure；不 dispatch |
| NRF mismatch | flat planner／resolver terminal failure；不替換 configured Client |
| Collected descriptor／records unavailable | data-owning Client／Leaf preparation failure；不改用 local file |
| Local file missing／digest／schema／window／sample failure | data-owning Client／Leaf preparation failure；不改用 collected retrieval |
| Partial preparation create | FL Server bounded cleanup 已建立 resources，request `FAILED` |
| Round／validation timeout | FL Server 收集至 deadline，不聚合 partial set，request `FAILED` |
| Publication failure | Publication owner 保留可診斷 durable state，request bounded failure |
| Containing-Go generation change | registry fence admission、清除 process-local request／workspace |
| PyMTLF restart | fresh coordinator state；舊 status 不恢復 |

## 10. 受控比較 profile

本地 `nwdaf-resources` 已提供下列 HFL aggregation topology，paired flat profile 必須使用相同四個
data-owning trainers：

```text
Flat
FL Server
├─ Client A1
├─ Client A2
├─ Client B1
└─ Client B2

Hierarchical
Root
├─ Branch A
│  ├─ Leaf A1
│  └─ Leaf A2
└─ Branch B
   ├─ Leaf B1
   └─ Leaf B2
```

固定比較規則：

- A1／A2／B1／B2 在兩個 mode 使用相同 NF identities、local UPF notification records、requested
  span、per-participant effective window 與 exact sample counts；
- controlled flat 依 static topology TAI 執行 scope composition、NRF resolution 與 Training
  Subscribe；controlled HFL 不配置或驗證 per-Leaf TAI。TAI 是刻意保留的 mode-specific
  orchestration 差異，不是 paired numerical equivalence 的共同輸入；
- Branch 不加入 local samples，只聚合 assigned Leaves；
- training／validation 保留相同 purged chronological split；不得為 comparison 引入 random split；
- seed artifact、FedAvg、local epochs、round count、device、batch size、learning rate、validation
  policy、`validation_ratio` 與 `random_seed` 相同；generated controlled configs 明確 pin
  `device: cpu`、`validation_ratio: 0.10`、`random_seed: 42`，並在 flat Server 與所有 Branch Server
  pin `preparation_data_window_seconds: 3600`；
- flat 與 HFL 使用相同 sample-count weighting；
- runner 必須從兩次獨立 executions 保存每一輪 Leaf weights／counts，重新計算 flat aggregate 與
  two-tier aggregate，再以既有 tolerance 比較；
- topology-only profile 預期兩者數值等價；若不等價，先視為資料、strategy、round 或 evidence
  mismatch，不能直接歸因於 hierarchy；
- latency、request count、artifact bytes、top-level load、failure isolation、timeout 與 cleanup 才是
  此 profile 的主要 topology cost 指標；
- model-quality study 若要加入 non-IID regrouping、Branch local rounds、partial participation 或其他
  strategy，必須建立另一個清楚標示 mixed variables 的 experiment profile。

Paired harness 應延伸既有 `deployments/hierarchical_fl` support 與 artifact capture，不複製
`distributed_fl` runner，也不另建一套平行 orchestration framework。Controlled flat 與 HFL
paired runs 固定讓 A1／A2／B1／B2 使用同一組 local UPF notification JSONL partitions；該 profile
不得啟動 PyAnLF process、建立 Training Data Descriptor 或存取 ADRF training records。Static
selection 與 training-data independence 必須由 Root／Server 沒有 active Monitor-derived participant
input，以及 Clients／Leaves 沒有 descriptor／ADRF training-data retrieval 的 direct assertions 證明。
Controlled HFL 另須證明所有 Root／Branch／Leaf profiles 在沒有 `trackingAreaList` 時仍完成 exact
resolution 與 training；runner 不得把 Leaf TAI 當成 HFL preflight 或 success evidence。

既有 production PyAnLF-to-ADRF collection fixture 仍作獨立 regression，證明新增 `local_file` 不破壞
`collected` source；不得把 controlled local-file success 當成 production collection flow 的替代證據。
該獨立 fixture 若保留 Leaf `trackingAreaList`，evidence 必須將它標示為 analytics collection prerequisite，
不得據此宣稱 HFL hierarchy discovery、assignment 或 local fitting 使用 TAI。

## 11. 儲存庫責任與預計檔案

### 11.1 `PyMTLF/`：production owner

預計修改：

- `src/py_mtlf/config.py`：orchestration／trigger models、FL Client training-data source、relative local
  path resolution、SHA-256、cross-field validation 與 explicit migration；
- `src/py_mtlf/app.py`：建立唯一 selected top-level coordinator，移除 topology-presence dispatch；
- `src/py_mtlf/core/dataset.py`、`training_data.py` 與 `fl_client.py`：`collected`／`local_file`
  preparation source selection、UPF notification JSONL validation、requested-span re-anchoring、scope
  binding 與 immutable snapshot reuse；
- `src/py_mtlf/core/fl_topology.py`：mode-specific static topology models、canonical snapshot 與 hash；
- `src/py_mtlf/core/fl_server.py`：explicit initiation input、participant／cutover scope 分離、static flat
  execution status 與 manual validation semantics；
- `src/py_mtlf/core/fl_root.py`：適配共用 top-level coordinator／status contract，不改 HFL execution；
- `src/py_mtlf/core/fl_hierarchy.py` 與 assignment artifacts：加入 explicit FedAvg strategy variant；
- 新的 `src/py_mtlf/core/fl_orchestration.py` 或等價 existing-owner file：共用 initiation／snapshot
  protocol；不得把兩套 coordinator state machine 藏在 API layer；
- `src/py_mtlf/api/federated_learning.py` 與對應 wire model：generic private management route；移除
  HFL-specific route implementation；
- `config/fl-server.yaml`、`config/fl-server-hierarchy.yaml`、`config/fl-server-client.yaml`、
  `config/fl-client.yaml` 與相關 tests／fixtures。

預計直接 tests：

- `tests/test_config.py`、`tests/test_runtime_modes.py`；
- `tests/test_fl_topology.py`、`tests/test_fl_server.py`；
- `tests/test_fl_root.py`、`tests/test_fl_experiment.py`；
- `tests/test_dataset.py`、`tests/test_training_data.py`、`tests/test_fl_client.py` 的 local-file source、
  cross-source no-fallback、single-window、requested-span re-anchoring、timestamp、digest、scope binding、
  purged chronological split 與 snapshot reuse tests；
- generic API tests，取代 `tests/test_hierarchical_fl_api.py`；
- `tests/test_publication.py` 的 zero-scope／restart characterization；
- HFL strategy／assignment artifact tests。

### 11.2 `nwdaf-resources/`：comparison 與 real-process evidence owner

預計修改：

- `deployments/hierarchical_fl/scripts/support.py`：mode-aware node sets、configs、origins 與 paired
  topology assets；controlled HFL profiles 不產生 `trackingAreaList`，controlled flat profiles 仍由
  static topology 與 registered capability 提供 TAI evidence；
- `deployments/hierarchical_fl/scripts/run.py`：generic API caller、controlled flat／HFL paired runs、
  exact numerical comparison 與 evidence；
- `deployments/hierarchical_fl/checks/preflight.py`、`checks/test_support.py`：explicit configs、topology
  validation、no-monitor participant selection 與 summary contract；
- `deployments/hierarchical_fl/configs/topology/`：新增 static flat four-Client fixture，保留現有 HFL
  `aggregation.yaml`；
- `deployments/hierarchical_fl/configs/datasets/` 或等價 tracked fixture directory：A1／A2／B1／B2
  的 UPF notification JSONL partitions 與 pinned SHA-256；
- `deployments/hierarchical_fl/README.md`、`components.yaml`；
- `deployments/distributed_fl` 的 generated production flat config 與 regression assertions。

### 11.3 預期不修改的 production repositories

- `NWDAF/`：現有 context、NRF relay、selected-target headers 與 Training SBI routing 重用；
- `PyAnLF/`：現有 Model Provision／Monitor closed loop 作 regression baseline；
- `nrf/`：現有 exact-instance、service、capability 與 interoperability matching 重用；TAI matching
  只供既有／static flat flow 使用，HFL 不新增 area query 或 matching requirement；
- `adrf/`：現有 data retrieval、model storage 與 durable reference behavior 重用。

若 deterministic implementation test 證明上述 repository 有 current-slice contract defect，必須先
依 development policy 進入 decision gate，再擴大 owner 與 plan；不能預先把它們列為 production
changes。

### 11.4 `nwdaf-docs/`

本文件保存 canonical requirement、implementation slices 與 verification matrix。Completed HFL
plans 保留歷史 contract；implementation 完成後只補本文件的 evidence 與 open testbed gap。

## 12. 實作切片

本 workstream 拆成三個可獨立驗收的垂直工作單元。Configuration migration、完整 static flat
產品流程，以及 paired comparison／本地 E2E 具有不同的變更邊界與失敗面，因此仍需分段；但
planner、API 與其驗證不得再拆成沒有完整操作入口的半成品，驗證收尾也不另立成實作切片。

### Slice 1：Explicit top-level configuration 與 migration

行為：

- 新增 typed orchestration／degradation trigger settings；
- 新增 FL Client `collected`／`local_file` training-data source schema，並一次遷移 committed Client
  profiles；此 Slice 只建立 config contract，source execution 納入 Slice 2；
- 區分 engine presence 與 autonomous top-level ownership；
- `create_app()` 只建立零或一個 selected coordinator；
- 一次更新 PyMTLF profiles、fixtures 與 nwdaf-resources generators；
- 不保留 implicit topology inference；
- 在尚未加入 static flat execution 前，完整保留 production flat、hierarchical 與 local 基準流程。

驗收測試：合法 matrix、所有 invalid combinations、relative topology／local dataset path、`collected`
禁止 path／SHA-256、`local_file` 必須提供 path／SHA-256、assigned Branch 不建立 Root／flat top-level
owner、federated degradation disabled 不累積 FL stale intent、local retraining 不受 federated trigger
設定影響，以及既有 production flat／HFL initiation 仍由顯式選定的 owner 接收。

延後項目：static flat execution、generic API migration 與 paired comparison。

### Slice 2：Static flat 完整產品流程與共用 manual API

行為：

- 解析／canonicalize static flat topology 與 hash；
- 建立 model descriptor + topology TAI 的 participant scopes；
- exact NRF resolve configured Clients；
- 讓 flat Client／HFL Leaf 依自身 config 明確選擇 `collected` 或 `local_file`，將 local UPF
  notification JSONL 驗證、scope binding、requested-span re-anchoring、effective window 與 immutable
  freeze 接入既有 dataset preparation contract；
- `local_file` loader 不讀取、推導或驗證 TAI，也不以 request area 篩選 rows；Flat 的 TAI 僅留在
  participant selection／request scope，HFL 不新增 TAI scope requirement；
- local file 不建立假的 descriptor／ADRF record、不執行 ADRF／MongoDB training-data retrieval，
  且 `local_file`／`collected` 禁止互相 fallback；`collected` 內既有 ADRF／MongoDB transport
  selection 保留；
- 引入 participant scopes／required cutover scopes 分離的 typed initiation；
- 讓 FL Server 從 explicit participant set 執行既有 preparation、round、aggregation、validation 與
  zero-scope publication；
- 建立 generic private router 與 common snapshot；
- flat static 與 HFL 共用 request body、idempotency、conflict、status retention、failure mapping 與
  generation reset semantics；
- 移除 HFL-specific route 並同步已知 active callers；
- 保持 selected coordinator 為唯一 ownership entry point；
- 完成 manual no-trigger validation、failure、timeout、restart 與 cleanup 的整條可操作流程。

驗收測試：missing／duplicate／self／scope／capability／service mismatch、extra NRF decoy、zero required
cutover scopes、partial dispatch cleanup、route gating、flat／HFL create + GET、same-ID replay、conflicting
body／active request、unknown family、unavailable generation、failure latch、terminal TTL、restart 404、
manual no-trigger validation、no URL disclosure、local-file missing／digest／JSONL／notification／timestamp／
measurement validation、scope binding、missing／multiple／zero-length requested window、`Tmax`、inclusive
effective file window、`minNumSamples`、purged chronological split、snapshot reuse，以及 cross-source
no-fallback failure；另驗證 local-file records 不需要 TAI，且 HFL profile 缺少 `trackingAreaList`
不影響 hierarchy resolution。

延後項目：paired numerical comparison。

### Slice 3：FedAvg paired comparison 與本地 E2E 收尾

行為：

- HFL strategy 增加 FedAvg variant，保留既有 FedProx；
- nwdaf-resources 以相同 four trainers 與 pinned local UPF notification JSONL partitions 執行
  controlled flat 與 HFL，且 controlled profiles 不啟動 PyAnLF process；
- controlled flat 保留 topology／NRF TAI contract；controlled HFL generated profiles、resolver query、
  run summary requirement 與 success assertions 不要求 TAI；
- generated controlled configs 顯式 pin 相同 `device`、`validation_ratio`、`random_seed`、direct
  Server `preparation_data_window_seconds` 與其他 fitting hyperparameters；
- 保存 config／topology／dataset copies、dataset digests、requested windows／spans、`Tmax`、effective
  windows、split policy、resolved identities、records、sample counts、weights、artifact transfers、timings
  與 revisions；
- 獨立重算 flat 與 two-tier aggregates 並比較 tolerance；
- 重跑 production flat isolated E2E；
- 重跑 HFL smoke manual、aggregation degradation、failure／timeout／restart scenarios；
- 執行新的 controlled flat／HFL paired runs，保存完整 local evidence，並維持
  `Testbed Validation Pending`。

驗收測試：strategy contract、generated configs、topology membership、origin allowlists、summary
schema、wrong-weight detection、相同 dataset partition／digest／requested span／effective window／split／
sample count／seed、no-monitor participant dependency、no-descriptor／no-ADRF-training-retrieval
assertions、Flat TAI mismatch rejection、HFL no-TAI resolution、`collected` production regression、
受影響的 lifecycle regressions，以及 §13 規定的 repository 與本地多程序驗證。

延後項目：testbed VM placement 與 full-core experiment conclusions。

Slice 3 不以 runner failure 自動授權其他 production repository changes；任何 confirmed defect
仍依 owner、failing test、review 與 repository-separated commit gates 處理。

## 13. 驗證矩陣

### 13.1 PyMTLF focused 驗證

```bash
.venv/bin/pytest -q \
  tests/test_config.py \
  tests/test_runtime_modes.py \
  tests/test_dataset.py \
  tests/test_training_data.py \
  tests/test_fl_client.py \
  tests/test_fl_topology.py \
  tests/test_fl_server.py \
  tests/test_fl_root.py \
  tests/test_fl_experiment.py \
  tests/test_publication.py \
  tests/test_federated_learning_api.py
```

Focused tests 必須直接覆蓋 selected production owner，不得只 mock 掉 topology planner、NRF resolver、
FL Server initiation 或 publication boundary 後宣稱完成。

### 13.2 儲存庫驗證

```bash
# PyMTLF/
.venv/bin/pytest -q
.venv/bin/ruff check src tests

# workspace root；nwdaf-resources checks
PyMTLF/.venv/bin/pytest -q \
  nwdaf-resources/deployments/hierarchical_fl/checks \
  nwdaf-resources/deployments/distributed_fl/checks
PyMTLF/.venv/bin/ruff check \
  nwdaf-resources/deployments/hierarchical_fl \
  nwdaf-resources/deployments/distributed_fl
```

### 13.3 必要的本地多程序情境

Exact CLI 可在 Slice 3 review 時依既有 runner interface 微調，但 completion evidence 必須分開保存：

```bash
PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/checks/preflight.py

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/distributed_fl/scripts/run.py

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/scripts/run.py \
  --profile smoke --scenario manual-success

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/scripts/run.py \
  --profile aggregation --scenario degradation-success

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/scripts/run.py \
  --profile controlled-flat --scenario manual-success

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/scripts/run.py \
  --profile controlled-hierarchical --scenario manual-success
```

既有 capability mismatch、preparation failure、round timeout 與 restart-generations scenarios 也必須
在受影響時重跑；不能以新的 success runs 取代 lifecycle regressions。

### 13.4 證據契約

每個 controlled run summary 至少保存：

- run-summary schema version、mode、participant source、trigger source；
- main config digest、topology source copy 與 topology SHA-256；
- 每個 data-owning participant 的 training data source；local-file profile 另保存 dataset copy、
  SHA-256、notification count、observation count、requested absolute window、requested span、`Tmax` 與
  effective time window；
- exact NF instance IDs、registered capabilities 與 service identities；Flat run 另保存 topology／
  registered TAIs，HFL run 明確保存 `trackingAreaList` 非必要且 controlled profile 未配置該欄位；
- model family、base／candidate／final artifact identities；
- per-participant split policy、training／validation sample counts 與每輪 effective counts；
- FedAvg strategy、epochs、rounds、direct Server preparation data window、device、batch size、learning
  rate、validation ratio 與 random seeds；
- request／process／plan identities；
- preparation、round、validation、publication 與 cleanup outcomes；
- artifact transfer counts／bytes 與 measured timings；
- repository revisions、dirty markers、binary hashes 與 backend process generations；
- `required_cutover_scopes` count，以及是否執行 consumer adoption verification。

Paired comparison 另外保存 flat aggregate、two-tier aggregate、numeric tolerance 與 equivalence result。

## 14. 驗收條件

1. Autonomous coordinator resolved config 明確輸出 `flat` 或 `hierarchical`，不再從 topology
   presence 推論。
2. Client-only 與 assigned Branch profiles 不建立 autonomous top-level coordinator。
3. 所有合法／非法 mode、participant source、topology、strategy 與 trigger combinations 在 startup
   deterministic validation。
4. `flat + monitor_scopes` 完整保留既有 Model Provision／Monitor degradation flow。
5. Local mode 的 AccuracyPolicy／DatasetCoordinator retraining 不受 federated trigger settings 影響。
6. Static flat product flow 支援至少兩個 configured Clients；four-Client comparison fixture 只訓練
   exact A1／A2／B1／B2，NRF decoy 不改變結果。
7. Static flat participant event、interoperability、target 與 non-area filter 來自 current model family；
   TAI partition 只來自 flat topology，ambiguous area source 被拒絕。HFL 不重用此 TAI composition
   contract。
8. Static participants 與 required cutover scopes 使用不同 typed fields；manual static publication
   記錄 `required_cutover_scopes = 0`。
9. `flat + static + private_api` 在沒有 Monitor-derived participant input 時仍完成 preparation、rounds、
   final validation、ADRF publication 與 catalog commit。
10. Static flat Client 與 HFL Leaf 可明確選擇 `local_file`，在沒有 PyAnLF、Training Data Descriptor
    與 ADRF training records 時，從 pinned UPF notification JSONL freeze dataset 並完成 local fitting；
    private request 不可指定或覆寫任何 local path。
11. `local_file` 嚴格驗證 path、SHA-256、UPF notification shape、timezone、完整 measurements 與
    sample eligibility；恰有一個 positive-length requested interval 時，以其 span 與本地 `Tmax`
    建立 inclusive effective window，將資料綁定到 request participant scope，且所有 rounds／final
    validation 重用同一 immutable snapshot；不得以 TAI 篩選或驗證 local-file rows，任一錯誤都不
    fallback 到 `collected`。
12. `collected` 完整保留 descriptor matching、ADRF／MongoDB retrieval 與現行 data conversion，且
    使用 requested absolute window，不因 local-file profile 存在而改變 source selection 或既有
    ADRF／MongoDB transport fallback。
13. Static HFL 只建立 configured Branch／Leaf assignments；controlled HFL Root／Branch／Leaf profiles
    都不配置 `trackingAreaList`，Branch／Leaf 仍可依 exact identity、service、capability 與
    interoperability 完成 resolution；不建立 per-Leaf TAI scope，且 existing FedProx behavior 不
    regression。
14. Generic manual request 不可覆寫 deployment mode／topology／strategy／training data source，且
    router 只交給 selected coordinator。
15. HFL-specific old route 已移除；已知 PyMTLF tests 與 nwdaf-resources callers 全部遷移。
16. 同一 process 不會讓一個 trigger 同時建立 flat 與 HFL ownership；兩種 trigger 同時啟用仍維持
    single-active。
17. Manual no-trigger validation 不製造假的 triggering scope，且保留 aggregate／per-scope evidence。
18. Missing、duplicate、self、capability、service、scope 與 model-composition mismatch deterministic
    failure，不 substitute 其他 NRF candidate。
19. Flat production、local retraining、HFL degradation、HFL manual、failure、timeout、restart、cleanup 與
    publication regressions 維持。
20. Controlled flat 與 HFL 使用相同 four trainers、local notification records／digests、requested
    span、per-participant effective windows、purged chronological split、`validation_ratio: 0.10`、
    `random_seed: 42`、direct Server `preparation_data_window_seconds: 3600`、FedAvg、其他
    hyperparameters、rounds 與 sample weighting。
21. Paired evidence 能獨立重算並證明相同 Leaf updates 下 flat 與 two-tier aggregate 在 tolerance 內
    等價，或以明確 mismatch evidence 失敗。
22. Run evidence 完整記錄 resolved mode、participant／trigger／training-data sources、topology hash、
    exact identities、dataset digest、requested window／span、`Tmax`、effective window、split policy、
    validation ratio、random seed、algorithm、sample counts、artifact／publication identity 與 component
    revisions。
23. Local verification 通過後狀態仍是 `Testbed Validation Pending`，不得宣稱跨 VM／full-core
    experiment 已完成。

## 15. 明確不在本次需求範圍

- dynamic HFL grouping 或 `hierarchical + monitor_scopes` planner；
- HFL per-Branch／per-Leaf TAI topology、area-based discovery、`networkArea` synthesis、local dataset TAI
  filtering，以及把 NWDAF registration 的選填 area field 升格為 HFL 必填 contract；
- `flat + monitor_scopes` manual snapshot semantics；
- arbitrary-depth hierarchy；
- 在單一 request 覆寫 deployment mode、topology、participant list 或 strategy；
- static flat FedProx parameter transport；
- mixed-version Root／Branch／Leaf interoperability；
- permanent Root／Branch／Leaf NRF role；
- 新增非標準 public NWDAF SBI；
- hot reload topology；
- local dataset hot reload、同一 execution 中切換 source、由 private request 傳入 path，以及同時
  支援多種 local dataset encoding；
- `local_file` multi-window union、依原始絕對時間選取歷史資料或其他 re-anchoring policy；
- 同一 PyMTLF 同時執行 flat 與 hierarchical top-level training；
- 為了製造模型差異而改變 aggregation、round count 或 local policy；
- testbed VM placement、Compose scaling、network mapping 與實際 testbed experiment run。

Testbed execution 是 `integration verification gap`，不是此 production implementation 的隱藏
completion criterion。Component 與 local paired E2E 通過後，仍須另行更新 testbed pins、部署並保存
testbed evidence。

## 16. 實作交接

本次審查已完成原文件要求的 exact revision、config loaders、fixtures、private callers、flat／HFL
lifecycle tests 與本地 deployment resources 盤點。Production implementation 開始前仍須：

1. 使用者確認本次修正版 plan；
2. 以 Slice 1 建立 normative conformance map，列出每個 config profile 與 invalid matrix；
3. 確認當時各 target repository revision 與 working tree 仍符合 §2.1；
4. 逐 Slice 實作、測試、mandatory review，並維持 unstaged／uncommitted diff 供使用者確認；
5. local completion 後保留 `Testbed Validation Pending`，不提前關閉 testbed acceptance。
