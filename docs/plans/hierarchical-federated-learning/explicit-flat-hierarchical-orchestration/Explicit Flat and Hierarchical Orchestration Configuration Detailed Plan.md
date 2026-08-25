# Explicit Flat 與 Hierarchical Orchestration 設定詳細計畫

日期：2026-08-25

狀態：Requirements Reviewed；Slice 1 Plan Approved；Slice 2 Review Deferred；implementation 尚未開始

相關文件：

- [Slice 1：Explicit Orchestration 與 Static／Manual Training 詳細計畫](./Slice%201%20Explicit%20Orchestration%20and%20Static%20Manual%20Product%20Flow%20Detailed%20Plan.md)
- Slice 2：MTLF-Triggered UPF Data Collection 詳細計畫（review deferred；尚未納入本次 commit）
- [Hierarchical NWDAF Federated Learning Implementation Plan](../Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)
- [Slice 8 Multi-process E2E and Regression Closure Detailed Plan](../Slice%208%20Multi-process%20E2E%20and%20Regression%20Closure%20Detailed%20Plan.md)
- [NWDAF Development Policy](../../../development_policy.md)
- [Release 18 Nsmf Event Exposure OpenAPI](../../../../specs/openapi/TS29508_Nsmf_EventExposure.yaml)
- [Release 18 Nupf Event Exposure OpenAPI](../../../../specs/openapi/TS29564_Nupf_EventExposure.yaml)
- [Release 18 Nnwdaf ML Model Training OpenAPI](../../../../specs/openapi/TS29520_Nnwdaf_MLModelTraining.yaml)
- [TS 23.288 §6.2.2.2 Data Collection from NFs](../../../../specs/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2%20Procedures%20for%20Data%20Collection/6.2.2%20Data%20Collection%20from%20NFs/6.2.2.2%20Procedure%20for%20Data%20Collection%20from%20NFs.md)

---

## 1. 目的與重新設計結論

本計畫定義 PyMTLF 如何以顯式 configuration 選擇 autonomous top-level flat 或 hierarchical
federated learning orchestration，並讓資料擁有端 FL Client／HFL Leaf 可在沒有 analytics consumer
subscription 的情況下，由 private API 主動建立 UPF data collection。

重新審查後採用下列結論：

1. 保留 production flat 的 Model Provision、Model Monitor、accuracy degradation、TAI scope 與 NRF
   exact discovery 閉環。
2. 保留既有 HFL Root static topology、Branch／Leaf assignment 與 lifecycle；HFL participant selection、
   discovery、assignment 與 training data matching 不新增 TAI requirement。
3. 新增不依賴 Monitor scopes 的 static flat participant selection，以及 flat／HFL 共用的 manual
   training private API。
4. `server`／`client` engine presence 不代表 autonomous top-level ownership。Assigned Branch 可同時有
   Server 與 Client engines，但不因此成為 flat coordinator 或 Root。
5. 移除原計畫的 production `local_file` training-data source、SHA-256 file contract、requested-span
   re-anchoring 與 parallel dataset adapter。所有 local fitting 都使用既有 collected dataset snapshot。
6. 資料蒐集改以觸發來源命名：`consumer_subscription` 表示現有 analytics consumer subscription
   間接觸發 AnLF collection；`private_api` 表示操作者或測試工具直接要求資料擁有端 PyMTLF 蒐集。
7. `private_api` 不是新的 dataset source。它產生與現有流程相同語意的 SMF subscription、UPF
   notifications、ADRF／MongoDB records 與 `TrainingDataDescriptor`，再交給同一
   `DatasetCoordinator` 依絕對 `timeWindows` freeze snapshot。
8. Private collection 第一版由操作者逐一呼叫 data-owning FL Client／HFL Leaf。Root／top-level Server
   不 fan out collection control，因為現有 cross-NWDAF Training contract 沒有 collection profile、
   collection endpoint 或 request ownership 欄位。
9. 受控 flat／HFL comparison 不再繞過 production ingest。`nwdaf-resources` 以同一組 pinned
   observation fixtures 經 support SMF 與 PyMTLF UPF callback replay，兩邊最終都從 collected snapshot
   fitting；testbed 則以真實 SMF／UPF notification 驗證。
10. 第一個 topology-only numerical comparison 仍要求 flat／HFL 都使用 FedAvg、相同 trainer inputs、
    split、seed、epochs、rounds 與 sample weighting；既有 HFL FedProx profile 保留。

這項重新設計不新增 NF type、不修改 Release 18 public SBI schema，也不把實驗 topology role 寫入 NRF
profile。它新增的是 PyMTLF private collection management API 與 containing NWDAF 的 MTLF-private relay。

## 2. 現況證據與設計基準

### 2.1 審查版本

| 儲存庫 | 版本 | 用途 |
| --- | --- | --- |
| `PyMTLF/` | `e9aa2235b6b4adc1d9d778b6cfdf23645fc622ec` | orchestration、dataset、training lifecycle 與 tests |
| `nwdaf-resources/` | `39ced284561e542c50da5fa7e83830aae4517821` | flat／HFL multi-process、fixture replay 與 evidence |
| `NWDAF/` | `3279891689dd9b54737ffe08dc18b9db72ec57b4` | containing-NWDAF private relays 與 Training SBI |
| `PyAnLF/` | `6a4d94ad3cc6f66dac55ea921772d731e4b71371` | consumer-triggered collection baseline |
| `nrf/` | `0dd4024d4ab75b6630e04901968228b9b9718cf5` | exact-instance discovery baseline |
| `adrf/` | `905f0599f68fe389bba14ed56db0ef9abeab5ccd` | data storage／retrieval 與 model publication baseline |

開始每個 Slice 前重新確認 target revision 與 working tree。Revision 若已移動，先重跑受影響的
characterization，不把本節記錄當成新 revision 的直接證據。

### 2.2 已確認的實作行為

| 現有行為 | 證據與結論 |
| --- | --- |
| Top-level owner inference | `create_app()` 看到 hierarchy topology 就建立 `FLRootCoordinator`；沒有 explicit mode。 |
| Flat participant selection | `FLServerEngine` 從 degradation `RetrainIntent.active_scopes` 取得 participants；manual static set 尚不存在。 |
| Manual API | 只有 `/internal/v1/hierarchical-fl/training-requests`，且直接要求 `app.state.fl_root`。 |
| HFL lifecycle | Root 已有 idempotency、single-active、failure latch、terminal TTL、generation reset 與 cleanup。 |
| Dataset preparation | `FLClientEngine` 由 Training Subscribe 建立 scope 與 absolute window；`DatasetCoordinator` 匹配本地 descriptor，再以 ADRF／MongoDB records freeze snapshot。 |
| Training metadata | Subscribe 已攜帶 `mLEventSubscs`、`tgtRepUe`、`dataAvReq.inpEvents`、`minNumSamples` 與 `timeWindows`；不攜帶 private collection profile ID。 |
| AnLF collection trigger | PyAnLF `SubscriptionService.create／replace` 呼叫 `CollectionManager.reconcile()`；desired targets 來自 analytics consumer event subscription。 |
| SMF／UPF path | PyAnLF 經 containing NWDAF AnLF-private proxy 建立 Nsmf Event Exposure；SMF 再建立 Nupf subscription，UPF callback 直接送到 PyAnLF。 |
| Storage／descriptor | PyAnLF 將 callback 轉成 `NadrfDataStoreRecord`，寫 ADRF 或 MongoDB，再經 Go relay 發布 `TrainingDataDescriptor` 給 PyMTLF。 |
| MTLF Go relay gap | `NWDAF/internal/mtlf/server.go` 已有 NRF discovery、ADRF retrieval／ML model 與 Training relay，但沒有 UDM collection、SMF Event Exposure 或 ADRF data-store relay。 |
| AnLF Go baseline | `NWDAF/internal/anlf` 已有相同 UDM、SMF、ADRF storage handler／processor boundary，並重用 `internal/sbi/consumer`。 |
| HFL TAI | `HierarchyNodeResolver` 以 exact identity、service、capability、event 與 interoperability resolve，不使用 `trackingAreaList`。 |
| Split 與 fitting | Dataset builder 使用 purged chronological split；`random_seed` 固定 NumPy、PyTorch、deterministic algorithms 與 DataLoader shuffle。 |

### 2.3 規範與已知限制

- TS 23.288 §6.2.2.2 允許 NWDAF 為 model training 使用 NF Event Exposure 收集資料；procedure
  並未要求必須先有 analytics consumer subscription。
- UE-related collection 是否需要 user consent 由 local policy／regulation 決定。第一版 private collection
  只允許 deployment 明確宣告 `not_required_by_local_policy`；若部署要求 consent，功能必須 fail closed，
  不得把既有 AnLF 行為當作 consent evidence。
- 現行 standard collection resolver 只完整支援 Internal Group ID→UDM group members→serving SMF→
  exact SMF discovery。第一版 `private_api` collection profile 因此只支援 `intGroupIds`，不宣稱支援
  `anyUe` 或 caller-provided direct SUPI collection。
- `DataAvReq.timeWindows` 保持 Release 18 絕對時間區間語意。移除 `local_file` 後不再有 source-specific
  re-anchoring；資料若不在 requested interval 內，preparation 直接失敗。
- Private API route 是 Go-to-backend／operator-to-backend private boundary，不是 public SBI；SMF、UPF、
  UDM、NRF 與 ADRF 方向仍使用 standard-shaped payload 與既有 consumer transport。

## 3. 術語與責任邊界

### 3.1 聯邦式協調

| 概念 | Flat FL | Hierarchical FL |
| --- | --- | --- |
| 自主頂層協調器 | Flat coordinator／FL Server deployment | Root coordinator |
| 中間 aggregation node | 不適用 | Branch |
| Data-owning trainer | FL Client | Leaf |

固定規則：

- flat runtime evidence 不把 Server 稱為 Root，也不把 Client 稱為 Leaf；
- Root／Branch／Leaf 只表示單次 hierarchical execution position；
- `FL_SERVER`、`FL_CLIENT`、`FL_SERVER_AND_CLIENT` 仍是 NRF capability；
- `federated_learning.server`／`client` 決定 engines，`orchestration` 決定 autonomous owner；
- application process 建立零或一個 selected top-level coordinator。

### 3.2 訓練資料

| 名詞 | 語意 |
| --- | --- |
| `consumer_subscription` | Analytics consumer subscription 觸發 PyAnLF collection；descriptor 經 AnLF→Go→PyMTLF relay。 |
| `private_api` | 操作者直接呼叫 data-owning PyMTLF；該 PyMTLF 自己管理 SMF subscription、callback、storage 與 descriptor。 |
| collected snapshot | 不論觸發來源，DatasetCoordinator 匹配 descriptor 並從 ADRF／MongoDB freeze 的 immutable training dataset。 |
| retrieval transport | ADRF 或 MongoDB；它不是 collection trigger，也不是另一種 dataset source。 |

`collection_trigger` 是 deployment-level collection ownership 選擇，不是 top-level training trigger。
`training_trigger.private_api` 建立 federated training request；`training_data.collection_trigger: private_api`
建立 training data collection request。兩個 API、request ID、status resource 與 lifecycle 必須分開。

## 4. 設定契約

### 4.1 協調與訓練觸發

只有可自主開始 federated training 的 profile 配置：

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

第一版值：

- `mode`: `flat | hierarchical`；
- `participant_source`: `monitor_scopes | static`；
- flat `monitor_scopes` 保留 production degradation flow；
- flat static 與 hierarchical static 可使用 training private API；
- Assigned Branch／Client-only 不得配置 autonomous orchestration 或 training trigger。

### 4.2 訓練資料蒐集觸發

每個可能執行 fitting 的 FL Client engine 明確配置：

```yaml
federated_learning:
  client:
    training_data:
      collection_trigger: consumer_subscription
```

或：

```yaml
federated_learning:
  client:
    training_data:
      collection_trigger: private_api
      callback_base_uri: http://10.0.0.21:9093
      state_directory: data/training-data-collections
      consent:
        purpose: model_training
        policy: not_required_by_local_policy
      collection_profiles:
        - profile_id: ue-communication-default
          ml_event: UE_COMMUNICATION
          ml_event_filter: {}
          target_ue:
            intGroupIds:
              - group-a.example
          dnns:
            - internet
          snssais:
            - sst: 1
              sd: "010203"
          sampling_interval_seconds: 2
          minimum_observation_count: 32
```

固定規則：

- `collection_trigger` 沒有 default；所有 committed FL Client／Branch profiles 一次遷移；
- `consumer_subscription` 禁止 private collection profiles 與 consent settings；
- `private_api` 必須有至少一個 unique `profile_id`、explicit consent policy 與 callback／storage settings；
- collection request 只選 `profile_id`，不得覆寫 target、DNN、S-NSSAI、sampling、callback URI、peer URL
  或 storage endpoint；
- `ml_event` 第一版只接受 builder 已支援的 `UE_COMMUNICATION`，required measurements 由 PyMTLF supported
  model contract 固定推導，不讓 request 或 profile 任意縮減；
- `target_ue` 第一版要求至少一個 Internal Group ID，禁止 `anyUe` 與 direct SUPI；
- private collection profile 不含 TAI／`networkArea`。Flat topology TAI 仍只負責 participant scope 與 NRF
  Training discovery；HFL 持續沒有 TAI contract；
- collection profile 是資料取得 intent，不是 Training Subscribe 的替代品。Fitting 前仍以 Subscribe 中的
  event 與 explicit filter／target constraints，以及 absolute window 匹配 descriptor；request 未指定
  target／filter 時沿用既有 wildcard semantics，但若因此匹配多個 private collection request groups，
  preparation 必須以 ambiguity 失敗，不得任意合併。

### 4.3 合法 profile matrix

| Profile | 自主訓練 | 參與者來源 | 蒐集觸發來源 |
| --- | --- | --- | --- |
| Local | 無 federated owner | 不適用 | 現有 local flow，不由本欄改變 |
| Client-only | 無 | upper Server 決定 | `consumer_subscription` 或 `private_api` |
| Assigned Branch | 無；保留 assigned execution | upper assignment | 有 local fitting 時同上；有 children 時只 delegation |
| Production flat Server | flat degradation | `monitor_scopes` | Server 本身不擁有 participant 資料 |
| Static flat Server | flat manual／optional degradation | static topology | Server 本身不擁有 participant 資料 |
| HFL Root | hierarchical manual／degradation | static hierarchy | Root 本身不擁有 Leaf 資料 |

Private collection 是 data-owning process 能力，不因 Root／Server 選擇 training mode 而自動啟用。第一版
不支援 Root collection fan-out，也不允許 top-level training request 帶 collection settings。

### 4.4 一次性 migration

- 不保留 topology-presence orchestration inference；
- 不保留 `local_file` alias、path／digest 欄位、loader 或 re-anchoring semantics；
- 不把 `consumer_subscription`／`private_api` 隱藏成 production／controlled mode；實際 deployment 語意由
  orchestration、participant source、training trigger 與 collection trigger 的組合形成；
- 更新 PyMTLF committed configs、fixtures 與 nwdaf-resources generators；strict validators 對 legacy 或
  矛盾設定 fail fast。

## 5. 靜態拓樸、TAI 與參與者契約

### 5.1 靜態 flat

Static flat topology 保存至少兩個 Client 的 exact NF instance ID 與 TAI：

```yaml
version: 1
clients:
  - nf_instance_id: 11111111-1111-4111-8111-111111111111
    scope:
      tracking_areas:
        - plmn_id: {mcc: "466", mnc: "92"}
          tac: "001101"
```

Parser 依 explicit mode 選擇 flat／hierarchy schema，拒絕少於兩個 Clients、duplicate ID／TAI、self target、
unknown fields 與 mode／shape mismatch，並以 canonical order 計算 topology SHA-256。

Static flat participant event、target UE、non-area filter 與 interoperability 來自 current model family；Client
identity 與 TAI 只來自 topology。若 family filter 已有 area source 且無唯一 composition 語意，request 在
discovery 前失敗。NRF resolver 必須驗證 exact identity、REGISTERED Training service、FL Client
capability、event、interoperability 與 TAI，不以其他 eligible candidate 替代。

### 5.2 HFL 無 TAI 邊界

- hierarchy topology 與 assignment 只攜帶 Branch／Leaf identities；
- resolver 只檢查 exact identity、Training service、role capability、event 與 interoperability；
- controlled HFL profiles 不配置 `trackingAreaList`；
- private collection profile 也不以 TAI 篩選、驗證或選擇 records；
- production flat 的 TAI-based AnLF collection 與 NRF discovery regression 保留，但不得作為 HFL success
  evidence。

### 5.3 參與者、切換與觸發分離

Internal `TopLevelInitiation` 使用 discriminated participant selection，並分開保存：

- `participant_selection`；
- `required_cutover_scope_keys`；
- `triggering_scope_key: str | None`；
- `trigger_source: degradation | private_api`；
- optional manual `request_id`。

Static manual participants 來自 topology，required cutover scopes 為空，triggering scope 為 `None`。
Degradation flow 的 adoption scopes 仍來自 Monitor intent。不得把 static participants 偽裝成 active
consumers。

## 6. 訓練管理契約

### 6.1 選定的協調器

`create_app()` 建立零或一個 `app.state.fl_coordinator`：flat 使用 `FlatFLCoordinator`，hierarchical 使用
`FLRootCoordinator`，non-owner 不建立。`FLServerEngine` 仍負責 preparation、round、aggregation、
validation、publication 與 cleanup；Assigned Branch 持有 Server engine 不代表可自主開始 training。

### 6.2 共用訓練 private API

```http
POST /internal/v1/federated-learning/training-requests
GET  /internal/v1/federated-learning/training-requests/{requestId}
```

Create body 只接受：

```json
{"requestId":"canonical-UUIDv4","modelFamilyId":"ue-communication-default"}
```

Request 不得覆寫 mode、participants、topology、strategy、collection trigger、profile、window、URL 或
artifact。Create 回 `202 Accepted` 與 `Location`；same ID＋same body replay 同一 resource；unknown family
為 `404`；conflict 為 `409`；generation unavailable 為 `503`。舊 HFL-specific route 移除且 active callers
同步遷移。

Manual initiation 沒有 fake triggering scope，但仍執行 aggregate／per-participant validation、ADRF final
model publication 與 zero-cutover catalog commit。

## 7. Private API 資料蒐集契約

### 7.1 本地負責元件與 API

只有 `collection_trigger: private_api` 的 data-owning PyMTLF 掛載：

```http
POST   /internal/v1/training-data-collections
GET    /internal/v1/training-data-collections/{requestId}
DELETE /internal/v1/training-data-collections/{requestId}
```

Create body 只接受：

```json
{"requestId":"canonical-UUIDv4","collectionProfileId":"ue-communication-default"}
```

PyMTLF collection manager 擁有 request records、profile snapshot、resolved targets、SMF resources、callback
correlations、storage progress、descriptor origin、retry、cleanup 與 restart recovery。Training coordinator、
DatasetCoordinator 與 API router 不得各自建立第二套 collection state。

### 7.2 方向性資料流

1. Operator／runner 呼叫每個 data-owning FL Client／Leaf 的 private collection API。
2. PyMTLF 從 immutable configured profile 取得 event、target Internal Group IDs、DNN、S-NSSAI、sampling 與
   minimum observation gate。
3. PyMTLF 經 containing NWDAF MTLF-private NRF／UDM relays 解析 group members、serving-SMF registrations 與
   exact `nsmf-event-exposure` endpoints。
4. PyMTLF 經 MTLF-private SMF relay 建立 Nsmf Event Exposure resource；SMF 建立 Nupf subscription，
   accepted representation 與 `Location` 形成 peer resource identity。
5. UPF notification 直接送到 PyMTLF `/callbacks/upf-event-exposure`；callback 以 server-generated
   correlation 驗證 active resource，將 accepted payload 寫入 restricted durable inbox 後才回 `204`。
6. PyMTLF 將 notification 包成 standard-shaped `NadrfDataStoreRecord`，經 containing NWDAF MTLF-private
   ADRF relay 寫入 ADRF；transient failure 依既有 policy fallback 到 MongoDB。
7. Durable storage 成功後，PyMTLF 以 accepted SMF subscription、stored absolute window、source SMF、ADRF
   identity 與 retention 建立 local `TrainingDataDescriptor`，直接註冊到 DatasetCoordinator。Internal key
   與 provenance 保存 origin、collection request／profile ID；不修改 wire schema，也不經
   PyAnLF→Go→PyMTLF descriptor round trip。
8. Operator 從 GET 確認 stored window／counts 達到 profile readiness 後，另行呼叫 top-level training API。
9. Training Subscribe 到達 Client／Leaf 後，DatasetCoordinator 只選與 configured collection trigger
   origin、event、filter、target 與 absolute requested window 相符的 descriptor，freeze 既有 immutable
   snapshot。
10. DELETE 移除 request reference；最後一個 reference 釋放時刪除 SMF resource。已保存 descriptor 轉為
    `RETAINED` 直到 TTL，讓剛停止 collection 的 training 仍可取得資料。

### 7.3 狀態與失敗

Collection status 至少包含 request／profile ID、state、resolved target counts、peer resource counts、storage
transport、stored time window、record／observation counts、descriptor state 與 bounded failure／cleanup
detail；不得回傳 peer URL、callback secret、credential 或 raw records。

`READY` 只表示 profile 的 minimum observation gate 與至少一次 durable storage 成功，不取代 Training
Subscribe 的 event／scope／`minNumSamples`／absolute window admission。Training preparation 仍可能因
requested window 或 sample construction 不足而失敗。

Collection ledger 必須 durable。Restart／generation change 時不自動 resume 舊 peer subscription；先從
ledger 進入 `RECOVERING` 並重試 exact peer cleanup，再保留可用 retained descriptor 或標記 bounded
interruption。Pending cleanup 不得被新 request 當成 fresh resource。Unknown／retired correlation callback
回 `404`；shutdown 停止 admission、完成已接受 callback 的 durable inbox ownership、cleanup peers、
persist final state，所有 close 操作 idempotent。Deadline 內未完成 storage 的 inbox items 繼續保留，已回覆
`204` 的資料不得因 shutdown 被丟棄。

### 7.4 Go NWDAF 邊界

`NWDAF/internal/mtlf` 在 existing server／processor owners 下加入：

- UDM SDM group identifier read relay；
- UDM UECM serving-SMF registration read relay；
- SMF Event Exposure create／read／replace／delete relay；
- ADRF data-store create relay。

Handlers 只處理 HTTP、header、body bound 與 ProblemDetails；processor 擁有 procedure delegation；outbound
call 重用 `internal/sbi/consumer`。以 `NWDAF/internal/anlf` 現有 route／processor／consumer wiring 作直接
local baseline，不新增 Go package，也不讓 PyMTLF 直接持有 NRF／UDM／SMF／ADRF public credentials。

## 8. 既有基準流程處置

| 階段 | Production flat `monitor_scopes` | Static manual training | 新增的 `private_api` collection |
| --- | --- | --- | --- |
| Trigger | explicit degradation gate | generic training API | per-data-owner collection API |
| Participant planning | Monitor snapshot | static flat／HFL topology | 不適用；profile 只定義 local data target |
| NF resolution | existing exact Client＋TAI | static flat 含 TAI；HFL 無 TAI | UDM group／serving SMF＋exact SMF discovery |
| Collection | existing PyAnLF consumer flow | Slice 1 先以 existing collected precondition | PyMTLF owns SMF／UPF resources |
| Storage／descriptor | PyAnLF→ADRF／Mongo→Go relay | reuse | PyMTLF→ADRF／Mongo→local descriptor |
| Preparation | absolute-window collected snapshot | reuse | reuse，禁止 re-anchor |
| Rounds／aggregation | flat FedAvg | flat FedAvg／existing HFL FedProx | 不適用 |
| Validation／publication | reuse Monitor cutover | manual no-trigger＋zero cutover | 不適用 |
| Failure／timeout | existing | common request status | collection ledger、retry、peer cleanup |
| Restart／shutdown | existing generation fence | adapted generic request records | cleanup-only recovery，不自動 resume |

## 9. 受控比較與資料重播

受控 flat 與 HFL 使用相同 A1／A2／B1／B2 data-owning trainers。所有 Client／Leaf 設定
`collection_trigger: private_api`，不啟動 PyAnLF process，也不建立 analytics consumer subscription。

Runner 會保存 pinned observation templates，在同一 paired run group 建立 common absolute anchor，為每個
trainer 產生相同 measurement sequence，然後依各次 SMF subscription 的 server-generated correlation
包成 NotificationData 並送入 PyMTLF production callback。因 correlation 是 transport identity 且 flat／
HFL run 必然不同，raw notification digest 不作 equivalence input；runner 另計算排除 correlation 與
delivery metadata 後的 canonical training-observation digest，並要求同一 trainer 兩個 mode 完全相同。

固定比較條件：

- trainer identities、canonical observations、absolute observation timestamps、requested window、split、
  sample counts 與 effective weights 必須相同；
- seed artifact、FedAvg、epochs、rounds、device、batch size、learning rate、validation ratio 與 random seed
  必須相同；
- Branch 不加入 local samples，只作 assigned aggregation；
- flat 保留 topology／NRF TAI；HFL 不配置或驗證 TAI；
- runner 保存每輪 trainer weights／counts，獨立重算 flat aggregate 與 two-tier aggregate 後比較 tolerance；
- local replay 只證明 production PyMTLF collection ingress 與 FL business flow，不宣稱真實 SMF／UPF
  integration；testbed 另以 real SMF／UPF evidence 收尾。

## 10. Repository 責任

### 10.1 `PyMTLF/`

- Slice 1：explicit orchestration、static flat planner、Flat coordinator、generic training API、participant／
  cutover separation、legacy config migration。
- Slice 2：collection trigger settings／profiles、private collection API、manager／ledger、UDM／SMF resolve、
  callback ingress、ADRF／Mongo storage、local descriptor origin 與 restart cleanup。
- Slice 3：HFL FedAvg variant、run evidence hooks 與受控 comparison defects。

### 10.2 `NWDAF/`

Slice 2 擴充 existing `internal/mtlf` server／processor wiring，重用既有 SBI consumers。Public SBI、NF
profile、NRF registration 與 standard generated models 不因 private route 而改變。

### 10.3 `nwdaf-resources/`

- Slice 1：config generators、preflight 與 generic training API caller migration；
- Slice 2：support SMF、private collection caller、callback replay、storage／descriptor／cleanup evidence；
- Slice 3：paired flat／HFL profiles、canonical observation digest、numerical comparison 與 local E2E closure。

### 10.4 預期不修改

- `PyAnLF/`：保留 `consumer_subscription` canonical baseline 與 regression；
- `smf-nwdaf-ext/`、`udm/`、`udr/`、`nrf/`、`adrf/`：重用現有 standard operations；
- `resources/`：read-only exemplar／reference。

若 direct test 證明預期不修改的 repository 有 current-slice contract defect，先依 development policy 進
decision gate；不得由 runner failure 自動授權跨 repository 修正。

## 11. 實作 Slice

### Slice 1：Explicit orchestration 與 static／manual training

詳細計畫：[Slice 1](./Slice%201%20Explicit%20Orchestration%20and%20Static%20Manual%20Product%20Flow%20Detailed%20Plan.md)

完成 explicit mode／participant source／training trigger、zero／one coordinator、static flat topology 與
exact discovery、participant／cutover separation、Flat coordinator、Root adapter、generic training API 與
active caller migration。FL Client config 先顯式採 `consumer_subscription`，所有 dataset preparation 保持
existing collected absolute-window semantics。

Slice 1 的 static manual success 以「matching descriptor 與 records 已存在」為 precondition；它不宣稱
沒有 analytics consumer 也能蒐集資料。這個 gap 由 Slice 2 明確關閉。

### Slice 2：MTLF-triggered UPF data collection

詳細計畫目前是 working-tree 草稿；review deferred，尚未納入本次 commit。

完成 `private_api` collection trigger、configured profiles、per-data-owner management API、PyMTLF
collection lifecycle、UPF callback、ADRF／Mongo storage、local descriptor、durable cleanup ledger，以及 Go
NWDAF MTLF-private UDM／SMF／ADRF relays。以 support SMF 與 callback replay 完成 local cross-process flow，
保留 real SMF／UPF testbed gap。

### Slice 3：FedAvg paired comparison 與本地 E2E 收尾

完成 HFL FedAvg variant、four-trainer controlled profiles、common anchored observation replay、canonical
observation digests、independent aggregation recomputation、production flat isolated rerun、HFL lifecycle
regressions 與完整 local evidence。完成後仍維持 `Testbed Validation Pending`。

三個 Slice 各自完成 implementation、direct verification、mandatory review 與 user review 後才可提出各
repository commit proposal；不得累積成一個跨 repository 大 commit。

## 12. 驗證矩陣

### 12.1 Slice 1

```bash
# PyMTLF/
.venv/bin/pytest -q \
  tests/test_config.py \
  tests/test_runtime_modes.py \
  tests/test_fl_topology.py \
  tests/test_fl_hierarchy_discovery.py \
  tests/test_fl_server.py \
  tests/test_fl_root.py \
  tests/test_fl_experiment.py \
  tests/test_publication.py \
  tests/test_federated_learning_api.py

.venv/bin/pytest -q
.venv/bin/ruff check src tests
```

### 12.2 Slice 2

```bash
# PyMTLF/
.venv/bin/pytest -q \
  tests/test_config.py \
  tests/test_training_data_collection.py \
  tests/test_upf_event_exposure.py \
  tests/test_dataset.py \
  tests/test_training_data.py \
  tests/test_fl_client.py

.venv/bin/pytest -q
.venv/bin/ruff check src tests

# NWDAF/
go test ./internal/mtlf/...
make test
make lint
make build
```

### 12.3 Slice 3 與 local real-process

```bash
PyMTLF/.venv/bin/pytest -q \
  nwdaf-resources/deployments/hierarchical_fl/checks \
  nwdaf-resources/deployments/distributed_fl/checks

PyMTLF/.venv/bin/ruff check \
  nwdaf-resources/deployments/hierarchical_fl \
  nwdaf-resources/deployments/distributed_fl

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/checks/preflight.py

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/distributed_fl/scripts/run.py

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/scripts/run.py \
  --profile controlled-flat --scenario manual-success

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/scripts/run.py \
  --profile controlled-hierarchical --scenario manual-success
```

實際 filenames／CLI 可在各 Slice 詳細計畫與 implementation 中依 existing owner 小幅調整，但不得省略
等價 direct coverage 或 full repository verification。Code／script commands 依 workspace policy 以
elevated permission 執行。

## 13. 驗收標準

1. Resolved configuration 明確輸出 `flat` 或 `hierarchical`，不從 topology presence 推論。
2. Client-only 與 Assigned Branch 不建立 autonomous coordinator；每個 process 最多一個 selected owner。
3. Production flat degradation、local retraining 與既有 HFL degradation／manual lifecycle 不 regression。
4. Static flat 只訓練 configured exact Clients；NRF decoy、scope、service、capability 或
   interoperability mismatch deterministic failure。
5. Static participants、required cutover scopes 與 optional triggering scope 使用不同 typed fields。
6. Generic training API 對 flat／HFL 提供一致 create／GET、idempotency、conflict、failure、TTL、restart 與
   no-disclosure semantics；舊 HFL-specific route 移除且 active callers 遷移。
7. Manual training 沒有 fake trigger，仍完成 validation、ADRF publication 與 zero-cutover catalog commit。
8. FL Client／Leaf 只使用 collected snapshot；workspace 中沒有 production `local_file` setting、loader、
   path、digest、re-anchoring 或 cross-source fallback。
9. `consumer_subscription` 完整保留 PyAnLF→SMF／UPF→ADRF／Mongo→descriptor relay→PyMTLF flow。
10. `private_api` 由 data-owning PyMTLF 建立、觀察、停止並在 restart 後 cleanup exact SMF resources；
    unknown callback correlation 不進 storage。
11. Private collection 只從 configured profile 取得 target／sampling，不接受 request override；第一版只
    支援 Internal Group ID standard resolution，且明確處理 local consent policy。
12. Go MTLF private server 以 existing handler→processor→consumer boundary 提供 UDM、SMF 與 ADRF relays，
    不新增 public SBI 或 Go package。
13. ADRF 成功與 Mongo fallback 都建立正確 local descriptor provenance、stored absolute window 與
    retention；DatasetCoordinator 依 configured trigger origin／request group 匹配，不混用另一條
    collection path，也不任意合併 ambiguous profile groups。
14. `READY` 不被誤解為 training guaranteed；Training Subscribe 的 event、target、absolute window 與
    `minNumSamples` 仍是 final dataset admission。
15. HFL topology／NRF resolution／assignment／collection profile／dataset matching 不新增 TAI；production
    flat TAI flow 保留。
16. Controlled flat／HFL 同一 trainer 使用相同 canonical observations、absolute timestamps、split、sample
    counts、seed、FedAvg 與 hyperparameters；transport correlation 不作 dataset equivalence identity。
17. Paired evidence 可獨立重算 flat 與 two-tier aggregate 並比較 tolerance。
18. Local replay evidence 明確標為 support SMF／callback injection；不得宣稱 real SMF／UPF、UE 或
    data-plane integration 已通過。
19. 每個 Slice 保持 unstaged／uncommitted 供 IDE review，review 確認後另提 repository-separated commit
    proposal。
20. Local completion 後狀態仍為 `Testbed Validation Pending`；只有部署到 testbed 並保存 real SMF／UPF
    與跨 VM evidence 後才能宣稱整體完成。

## 14. 明確延後項目與非目標

- dynamic HFL grouping 或 `hierarchical + monitor_scopes`；
- HFL per-node TAI topology、area discovery 或 filter synthesis；
- Root／top-level Server collection fan-out；
- request-level target、DNN、S-NSSAI、sampling、peer URL、callback URI 或 storage override；
- `anyUe` 與 direct SUPI private collection；
- deployment 要求 user consent 時的 UDM consent retrieval／change subscription；
- collection lease renewal 與 active subscription automatic restart resume；
- arbitrary-depth hierarchy、hot reload topology／collection profile；
- mixed-version Root／Branch／Leaf interoperability；
- static flat FedProx；
- permanent Root／Branch／Leaf NRF role；
- 新增非標準 public NWDAF SBI；
- testbed VM placement、network mapping 與 actual testbed experiment execution。

User-consent integration 是 `future-phase handoff`；在 local policy 要求 consent 的環境中，它是啟用
private collection 的 blocker。Real SMF／UPF 與 testbed execution 是 `integration verification gap`，不能
由 mock、support SMF、callback replay 或 unit tests 關閉。

## 15. 實作交接

開始 production implementation 前：

1. 使用者已確認本次重新設計的父計畫與 Slice 1；Slice 2 在 Slice 1 完成後另行 review 與確認；
2. 每個 Slice 從最新文件建立 normative conformance map；
3. 重新確認 target revisions、working trees 與 active callers；
4. 依 Slice 順序實作、direct verification、mandatory review 與 user review；
5. 每個 repository commit 前另行提出完整 commit proposal 並取得明確核准；
6. local Slice 3 完成後保留 `Testbed Validation Pending`，不提前關閉 external acceptance。
