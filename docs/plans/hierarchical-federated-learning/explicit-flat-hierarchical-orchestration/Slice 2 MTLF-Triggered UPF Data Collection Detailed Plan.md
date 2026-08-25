# Slice 2：MTLF-Triggered UPF Data Collection 詳細計畫

日期：2026-08-26

狀態：Revision Complete；User Review Confirmed；Ready for Implementation

相關文件：

- [Explicit Flat and Hierarchical Orchestration Configuration Detailed Plan](./Explicit%20Flat%20and%20Hierarchical%20Orchestration%20Configuration%20Detailed%20Plan.md)
- [Slice 1：Explicit Orchestration 與 Static／Manual Training 詳細計畫](./Slice%201%20Explicit%20Orchestration%20and%20Static%20Manual%20Product%20Flow%20Detailed%20Plan.md)
- [NWDAF Development Policy](../../../development_policy.md)
- [Release 18 Nsmf Event Exposure OpenAPI](../../../../specs/openapi/TS29508_Nsmf_EventExposure.yaml)
- [Release 18 Nupf Event Exposure OpenAPI](../../../../specs/openapi/TS29564_Nupf_EventExposure.yaml)
- [Release 18 Nadrf Data Management OpenAPI](../../../../specs/openapi/TS29575_Nadrf_DataManagement.yaml)
- [Release 18 Nudm SDM OpenAPI](../../../../specs/openapi/TS29503_Nudm_SDM.yaml)
- [Release 18 Nudm UECM OpenAPI](../../../../specs/openapi/TS29503_Nudm_UECM.yaml)
- [TS 23.288 §6.2.2.2 Data Collection from NFs](../../../../specs/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2%20Procedures%20for%20Data%20Collection/6.2.2%20Data%20Collection%20from%20NFs/6.2.2.2%20Procedure%20for%20Data%20Collection%20from%20NFs.md)

---

## 1. Slice 目標與完成邊界

本 Slice 讓 data-owning PyMTLF 可以在沒有 analytics consumer subscription 的情況下，由 private API
主動建立、觀察與停止 training-data collection。資料仍由 SMF／UPF Event Exposure 進入，寫入 ADRF 或
MongoDB，再以 local `TrainingDataDescriptor` 交給既有 `DatasetCoordinator`。它不是 local fixture loader，
也不是另一種 fitting dataset source。

本 Slice 完成後：

1. FL Client／HFL Leaf 可明確設定 `training_data.collection_trigger: private_api`。
2. Operator／runner 可逐一呼叫 data-owning PyMTLF 的 collection API；Root／Server 不代為 fan out。
3. PyMTLF 以 configured profile 解析 Internal Group ID、serving SMF 與 exact Event Exposure endpoint，
   建立 Nsmf subscription 並接收 direct UPF callback。
4. Notification 經 strict correlation／schema admission 並寫入 restricted durable inbox 後才回 `204`，
   再寫入 ADRF 或 existing Mongo fallback；storage 成功才更新 descriptor 與 readiness。
5. 兩種觸發來源的 descriptor 使用 existing wire shape，但 PyMTLF 以 trusted internal metadata 將
   `private_api` 與 `consumer_subscription` descriptor 分開。
6. DatasetCoordinator仍以Training Subscribe中的event、target、absolute time window與`minNumSamples`
   freeze snapshot；不re-anchor、不fallback到另一個collection trigger。
7. SMF peer resources、callback correlations、request status與cleanup state有durable ledger；restart不留下
   無owner的subscription，也不自動resume未知狀態的collection。
8. Containing NWDAF的MTLF-private server提供必要UDM、SMF與ADRF relay，並重用existing
   handler→processor→consumer structure。
9. Local cross-process verification使用support SMF與callback replay，並明確保留real SMF／UPF testbed gap。
10. 所有changes保持unstaged／uncommitted供user review；commit approval為獨立gate。

本 Slice 不實作Root collection fan-out、不執行flat／HFL numerical comparison、不新增HFL FedAvg，也不
修改 SMF、UPF、UDM、UDR、NRF、ADRF 或 PyAnLF production repositories。這些預期不修改的 component
只作 integration peer 或 baseline。

## 2. 基準版本與 repository 責任

### 2.1 計畫基準

| 儲存庫 | 版本 | 本 Slice 定位 |
| --- | --- | --- |
| `PyMTLF/` | `2f15c4732aacbd2c4658a0af12e7612a1f076728` | Slice 1 committed orchestration baseline；collection owner、callback、storage、descriptor與tests |
| `NWDAF/` | `3279891689dd9b54737ffe08dc18b9db72ec57b4` | MTLF-private UDM／SMF／ADRF relays |
| `nwdaf-resources/` | `ac90df6388501db3fa74d2f20218076583af2c54` | Slice 1 committed runners；support SMF、collection caller、callback replay與evidence |
| `PyAnLF/` | `6a4d94ad3cc6f66dac55ea921772d731e4b71371` | canonical consumer-triggered collection baseline |
| `smf-nwdaf-ext/` | `128b0ec6157238efe4203e2060415728599ada04` | real Nsmf→Nupf peer baseline |
| `udm/` | `b8db49ee8c2fdae8f686562ae6a941a232e80ad3` | group membership／serving-SMF resolution baseline |
| `udr/` | `bf447744345fe238092588f105a57d2484f5f1b7` | group data／serving-SMF persistence baseline |
| `nrf/` | `0dd4024d4ab75b6630e04901968228b9b9718cf5` | exact NF／service discovery baseline |
| `adrf/` | `905f0599f68fe389bba14ed56db0ef9abeab5ccd` | data-store／retrieval baseline |

開始 implementation 前重新確認所有 target revision 與 working tree。Revision 若移動，先更新受影響的
characterization 與 conformance record，不能直接沿用本節證據。

### 2.2 Slice 1 實際交接介面

Slice 2 從已提交的 Slice 1 contract 延伸，不再以原始草稿介面為假設：

- `FLTrainingDataSettings` 目前只接受 `collection_trigger: consumer_subscription`；Slice 2 在同一 typed
  setting 中加入 `private_api` variant 與其 discriminated fields，不建立平行 data-source setting。
- `DatasetCoordinator` 目前以 `dict[str, TrainingDataDescriptor]` 按 `correlationId` 保存 descriptor，AnLF
  private route 直接呼叫 `put_training_data_descriptor()`／`delete_training_data_descriptor()`。Slice 2 必須
  顯式遷移這個 owner，不能假設 origin 或 collection group 已存在。
- `create_app()` 目前由同一 lifecycle callback reset `fl_coordinator`、`fl_client` 與 `fl_server`，並在
  shutdown 後段關閉 `DatasetCoordinator`。Collection manager 是新的獨立 lifecycle owner，必須在既有
  app construction、generation reset 與 shutdown order 中有明確位置。
- `/internal/v1/federated-learning/training-requests` 是 Slice 1 已提交的 top-level training contract；本
  Slice 的 `/internal/v1/training-data-collections` 維持獨立 request ID、status 與 lifecycle。Collection
  success 不自動觸發 training，training request 也不攜帶 collection profile。
- `NwdafContextClient` 目前只擁有 containing-NWDAF identity／base URI 與 generation context。Slice 2 不把
  UDM／SMF／ADRF procedure 塞入這個 context client；focused relay client 使用其 resolved
  `internal_api_root`，但自行擁有 request／response contract 與 transport lifecycle。

### 2.3 受影響的 repositories

- `PyMTLF/`：production owner。
- `NWDAF/`：containing-NWDAF private relay owner。
- `nwdaf-resources/`：local real-process evidence owner。
- `nwdaf-docs/`：plan與implementation evidence。

每個 repository 各自 review 與 commit。任一 repository 未完成 direct contract verification 時，Slice
不能因其他 repository suite green 而宣稱 complete。

### 2.4 預期不修改的 repositories

- `PyAnLF/`：保留`consumer_subscription` flow，作characterization與regression。
- `smf-nwdaf-ext/`、`udm/`、`udr/`、`nrf/`、`adrf/`：重用existing public／standard-shaped behavior。
- `resources/`：read-only references／free5GC exemplars。

若 deterministic boundary test 證明 peer 有 current-Slice contract defect，依 development policy 的
decision gate 先更新 plan 與 owner，不由 E2E failure 自動擴大 scope。

## 3. 既有 AnLF 基準與採用範圍

### 3.1 現有 `consumer_subscription` 流程

既有基準流程如下：

1. Analytics consumer建立Events Subscription。
2. PyAnLF `SubscriptionService`建立／更新runtime subscription，呼叫`CollectionManager.reconcile()`。
3. Collection manager從event subscription取得target UE／Internal Group ID、DNN、S-NSSAI與network area。
4. Standard resolver經containing NWDAF AnLF-private NRF／UDM relays取得group members、serving SMF與exact
   `nsmf-event-exposure` endpoint。
5. PyAnLF經AnLF-private SMF relay建立subscription；callback URI指向PyAnLF。
6. UPF notifications 進入 PyAnLF callback，ingestion 寫入 ADRF 或 MongoDB。
7. Collection resource以accepted SMF subscription與stored window建立`TrainingDataDescriptor`。
8. Descriptor經PyAnLF→Go AnLF server→Go MTLF client→PyMTLF relay。
9. Analytics subscription刪除時release references、刪peer resource並保留retained descriptor到TTL。

### 3.2 階段處置

| 基準階段 | Slice 2 處置 |
| --- | --- |
| Analytics consumer subscription owner | replaced：private collection request resource |
| Target/profile production | replaced：immutable PyMTLF collection profile；request只選profile ID |
| Reconcile／deduplicated peer resources | adapted：PyMTLF collection manager owns request references |
| UDM／NRF／SMF resolution | reused semantically；transport改走MTLF-private relay |
| SMF subscription shape／callback correlation | reused without semantic change |
| UPF callback admission | adapted to PyMTLF router／collection owner |
| ADRF record construction／Mongo fallback | reused semantically in PyMTLF ingestion owner |
| Descriptor shape | reused without wire schema change；origin為PyMTLF internal metadata |
| Descriptor delivery | replaced：direct local DatasetCoordinator registration |
| Analytics runtime／reporting | not applicable；不移植 |
| Release／retention／cleanup | adapted：加入 durable ledger 與 restart cleanup |

### 3.3 明確不從 PyAnLF 移植的項目

不移植analytics subscription CRUD、analytics runtime、observation ring、report scheduler、accuracy report、
model demand、consumer notification或AnLF reporting state。PyMTLF只實作model-training data collection所需
的 profile、resource、callback、storage、descriptor 與 cleanup vertical flow。

Python code 不得直接 copy 整個 PyAnLF module 造成兩套 analytics lifecycle。實作可參考其已驗證
algorithm 與 wire semantics，但需依 PyMTLF package owners 重建最小 typed components 與 tests。

## 4. 設定契約

### 4.1 Client 設定

```yaml
federated_learning:
  client:
    training_data:
      collection_trigger: private_api
      callback_base_uri: http://10.0.0.21:9093
      state_directory: data/training-data-collections
      request_timeout_seconds: 30
      retry_initial_backoff_seconds: 1
      retry_max_backoff_seconds: 30
      worker_count: 2
      queue_capacity: 256
      descriptor_retention_seconds: 3600
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

`callback_base_uri`是UPF可達的PyMTLF origin，不由binding host或artifact URL推導。`state_directory`以main
config所在目錄解析，但status不回傳resolved path。

### 4.2 嚴格驗證

- `collection_trigger`無default；Slice 2加入`private_api` legal variant。
- `private_api`要求callback URI、state directory、explicit consent與至少一個profile。
- Profile ID non-empty／unique；request只引用profile ID。
- 第一版`ml_event`只接受`UE_COMMUNICATION`。
- `target_ue.intGroupIds` non-empty／unique；禁止`anyUe`、SUPI、GPSI與extra target fields。
- DNN normalized／unique；S-NSSAI strict、unique；可省略表示不以該欄篩選serving registration。
- `ml_event_filter`第一版禁止area／TAI fields；HFL與private collection都不建立TAI contract。
- Sampling、minimum count、timeout、workers、queue、retention全部positive且bounded。
- Required measurements由PyMTLF supported `UE_COMMUNICATION` builder contract固定，不由config／request
  任意縮減。
- `consumer_subscription` 禁止所有 private-only fields；`private_api` 缺任一 required setting 時 startup fail。
- Assigned Branch有children時不因local collection存在而加入samples；若它同時服務其他direct flat
  assignment，仍以該execution的Client semantics處理，不從hierarchy role推測。

### 4.3 Consent 邊界

TS 23.288指出UE-related model-training collection是否需要consent取決於local policy／regulation。V1只接受：

```yaml
consent:
  purpose: model_training
  policy: not_required_by_local_policy
```

這是deployment operator的explicit policy assertion，不是PyMTLF自行判斷已取得consent。若環境要求
consent，config不得啟用private collection；`required`或unknown policy deterministic startup failure。
UDM user-consent retrieval／change subscription 屬 future phase，不能在 review 時把此 assertion 描述為
production consent enforcement。

## 5. Private 管理 API

### 5.1 Routes 與 request

只有`collection_trigger: private_api`且Client engine存在時掛載：

```http
POST   /internal/v1/training-data-collections
GET    /internal/v1/training-data-collections/{requestId}
DELETE /internal/v1/training-data-collections/{requestId}
```

POST body：

```json
{
  "requestId": "canonical-UUIDv4",
  "collectionProfileId": "ue-communication-default"
}
```

禁止target、DNN、S-NSSAI、sampling、minimum count、network area、peer、callback、storage、descriptor或
window override。Request ID為caller idempotency key；same ID＋same body回same resource，不同body為`409`。

### 5.2 HTTP 語意

- POST accepted：`202 Accepted`＋`Location`＋current snapshot；
- GET found：`200 OK`；unknown／expired：`404`；
- DELETE accepted／replay：`202 Accepted`直到cleanup terminal；已terminal可回current snapshot；
- malformed／unsupported：`400 ProblemDetails`；
- unknown profile：`404`；
- collection key 尚在 pending cleanup 或 same ID different body：`409`；
- containing generation unavailable、manager closing、queue exhausted：`503`；
- internal peer error映射成bounded cause／detail，不把peer body、URL或credential回給caller。

API router不保存domain state，不直接call Go relay，不在request task中同步等待整個collection建立完成。

### 5.3 Status schema

Status 至少包含：

- `requestId`、`collectionProfileId`、`collectionTrigger: private_api`；
- state、created／updated time與process generation；
- intended group count、resolved UE／SMF target count；
- active／pending-cleanup peer resource count；
- storage transport：`adrf | mongodb | unavailable`；
- stored absolute `startTime／stopTime`；
- record／observation counts與profile minimum；
- descriptor state：`NONE | ACTIVE | RETAINED`；
- bounded failure cause／detail與cleanup pending indicator。

Status 不得包含 SUPIs、group membership contents、SMF／ADRF API roots、subscription IDs、callback correlation、
raw notifications、Mongo URI、filesystem path或credentials。Test evidence可在runner-controlled protected
artifacts保存必要peer identities，但不改private API disclosure contract。

## 6. Collection state、ledger 與 concurrency

### 6.1 Request states

穩定的 request states：

```text
PENDING -> RESOLVING -> SUBSCRIBING -> COLLECTING -> READY
                                      |              |
                                      +----> STOPPING +----> RETAINED
                                                  \-------> TERMINATED

active state after restart -> RECOVERING -> RETAINED | FAILED
any nonterminal unrecoverable error -> STOPPING -> FAILED
```

- `READY`表示至少一次durable storage success且observations達profile minimum；不是training guarantee。
- DELETE可從任何nonterminal state進`STOPPING`。
- 有stored data時successful stop進`RETAINED`；無data進`TERMINATED`。
- `RETAINED` descriptor在TTL內可供training；TTL後request／descriptor移除，GET `404`。
- `FAILED`前必須完成或明確保留pending peer cleanup；不能在orphan resource仍無owner時假裝terminal clean。

### 6.2 Resource identity 與 references

`CollectionKey`至少包含resolved SUPI、exact SMF identity／API root、PDU session ID、DNN、S-NSSAI、sampling
interval、required measurements與canonical non-area filter。SMF identity是provenance；peer resource identity
使用accepted subscription ID／Location與server-generated correlation。

多個 request／profiles 解析到 same key 時可 share 一個 peer resource，resource 保存 request references。
只有最後一個 reference 釋放才 DELETE peer。Different key 不得因 same SMF endpoint 被合併。Concurrent
create 以 per-request 與 per-key locks 序列化；API idempotency、resource dedup 與 worker task
supersession 是三個不同 invariant。

### 6.3 Durable ledger

Collection manager以atomic replace寫入`state_directory`，至少保存：

- request body digest、profile snapshot digest與state；
- resolved collection keys所需的non-secret identity；
- accepted peer subscription ID／Location、target API root、correlation與references；
- storage transport、stored window、counts、ADRF identity與descriptor retention；
- retry attempt、pending cleanup與last bounded failure；
- containing-NWDAF process generation。

Ledger不保存credentials或raw notification bodies。Raw accepted callbacks使用`state_directory/inbox`或
等價durable spool，directory／files必須以process owner-only permissions建立，record成功寫入ADRF／Mongo
後才刪除。Corrupt／unknown-version ledger或inbox使private collection admission fail closed，保留檔案供
diagnosis；不得silent discard後建立可能duplicate的SMF subscriptions。

### 6.4 Restart 與 shutdown

Startup 先 load ledger／inbox 並 block new collection admission。先以 ledger 中的 accepted subscription 處理已
acknowledged inbox records，再讓previous active resource進`RECOVERING`，透過stored exact target執行peer
DELETE；V1不automatic resume。Cleanup成功後，有durable records者重建`RETAINED` local descriptor，無
資料者標`FAILED／INTERRUPTED`。Cleanup transient failure依bounded backoff持續保留owner與status。

Collection manager 是 `app.state.training_data_collection_manager` 或等價明確 state owner，不併入
`fl_coordinator`、`fl_client`、`fl_server` 或 `DatasetCoordinator`。`create_app()` 必須依下列順序整合：

1. `DatasetCoordinator` 與 containing-NWDAF context 建立後才建構 collection manager；
2. lifespan startup 先 open containing context，再由 manager load ledger／inbox、drain acknowledged items
   與完成 cleanup-only recovery；manager 到達可安全 admission 的狀態後，runtime 才接受 collection API；
3. containing-Go generation change 先 fence collection create／callback admission並把已接受 callback
   ownership 固定到 durable inbox，再進 exact peer cleanup；之後才讓既有 FL owners 與 workspace 執行
   generation reset；
4. shutdown 必須先關閉 collection manager，再關閉 `DatasetCoordinator` 與 containing context，避免 manager
   在 dataset repository 或 relay transport 已關閉後仍提交 descriptor／storage work。

Containing-Go generation change停止new work、fence in-flight responses並進同一cleanup path。Manager
shutdown內部順序：

1. stop API admission；
2. invalidate queued／scheduled discovery tasks；
3. stop accepting new callbacks，將已 admitted callback durable spool drain 至 deadline；未完成 storage 的
   inbox items 保留供 restart recovery，不得丟棄；
4. release request references 並 delete peer resources；
5. persist descriptors／terminal／pending cleanup state；
6. close HTTP／Mongo／worker clients。

所有 abort／close／delete operations 都必須 idempotent。Late callback 對 retired correlation 回 `404`，
不得復活 resource。

## 7. Resolution、SMF subscription 與 callback

### 7.1 標準解析流程

每個configured Internal Group ID：

1. 經MTLF-private NRF relay exact discover一個REGISTERED `nudm-sdm` endpoint。
2. 經UDM SDM relay取得group identifiers與UE ID list；returned group ID若不相符，request失敗。
3. Exact discover 同一 UDM 的 REGISTERED `nudm-uecm` endpoint。
4. 對每個SUPI取得serving-SMF registrations，依configured DNN／S-NSSAI篩選。
5. 以registration中的exact SMF instance ID discoverREGISTERED `nsmf-event-exposure` service。
6. Missing、ambiguous、malformed 或 incomplete target 保持 request pending retry 或 bounded failure，不以其他 SMF
   substitute。

Private collection 不從 flat topology TAI 或 HFL assignment 產生 target，也不查 `trackingAreaList`。Collection
profile與Training participant selection是兩條獨立authoritative inputs；training時由descriptor matching
檢查兩者語意是否相符。

### 7.2 Nsmf resource

PyMTLF建立standard-shaped Nsmf Event Exposure subscription，至少包含：

- exact SUPI、PDU session、DNN、S-NSSAI；
- containing NWDAF NF instance ID；
- server-generated `notifId`／correlation；
- PyMTLF callback URI；
- SMF event `UPF_EVENT`，其中`upfEvents`固定要求`USER_DATA_USAGE_MEASURES`、
  `VOLUME_MEASUREMENT`＋`THROUGHPUT_MEASUREMENT`與`PER_SESSION` granularity；
- periodic notification method與configured reporting interval；
- configured non-area filter所能合法映射的fields。

Create 必須取得 `201`、`Location` 與可驗證的 accepted representation。Missing／malformed representation 先追蹤
provisional resource再進cleanup，不遺失peer identity。Peer若回V1不支援的finite expiry，立即標cleanup
pending並失敗；lease renewal deferred。

### 7.3 UPF callback admission

```http
POST /callbacks/upf-event-exposure
```

Callback使用Release 18 `NotificationData` strict supported profile與bounded body：

- `correlationId`必填且屬active resource；unknown／retired回`404`；
- `notificationItems`non-empty，event type為`USER_DATA_USAGE_MEASURES`；
- timezone-aware timestamps、至少一種UE address與non-empty usage measurements；
- builder所需volume／packet／throughput fields完整、finite、non-negative且unit supported；
- body／schema錯誤回`400 ProblemDetails`；
- queue capacity admission失敗回`503`，不接受後silent drop；
- accepted callback以canonical raw payload digest建立durable inbox item後回`204`；exact duplicate delivery
  重用同一inbox identity，不重複建立storage work。

Callback不信任payload提供owner、profile、scope、SMF identity或expected correlation；這些由server-side
resource lookup取得。Transport correlation不得作training dataset equivalence identity。ADRF create若在
peer已commit後發生response loss，V1無法由create API證明exactly-once；private-origin dataset retrieval因此
需以canonical NotificationData content digest去除exact duplicate records，並將此限制保留在private
origin，不改變`consumer_subscription` baseline。

## 8. Storage、descriptor 與 dataset 整合

### 8.1 Record 建構與 storage

PyMTLF以accepted Nsmf subscription建立`NadrfDataStoreRecord.dataSub[].smfDataSub`，將callback放入
`dataNotif.upfEventNotifs`。不得建立與peer resource無關的fake `dataSub`。

Storage policy 沿用 existing deployment semantics：

- ADRF candidate與target由existing PyMTLF ADRF settings／NRF discovery取得；
- 經containing NWDAF MTLF-private ADRF data-store relay POST；
- `201`＋valid `Location`才算ADRF durable success；
- transient／5xx／429依bounded retry並可fallback到configured MongoDB；
- permanent 4xx drop該record並記bounded failure，不標storage success；
- Mongo document shape保持DatasetCoordinator existing retrieval contract與indexes；
- callback received不等於stored，readiness只計durable success。

### 8.2 本地 descriptor

Resource至少在accepted SMF subscription、stored start／stop、source SMF identity與durable transport已知後
才能建立descriptor：

- wire shape保持existing `TrainingDataDescriptor`；
- `storedDataSpec.dataSpec.smfDataSub`來自accepted subscription；
- `timePeriod`來自實際durable records；
- `mlEventSubscription`來自configured profile，不從callback row反推；
- `sourceNfInstanceId`是serving SMF；
- ADRF success填`adrfInstanceId`，Mongo fallback為`None`；
- active resource為`ACTIVE`，stop後為`RETAINED`至configured TTL。

Private-collection descriptor 直接呼叫 DatasetCoordinator owner method，不送 AnLF-private descriptor
route。Slice 2 將目前按 `correlationId` 儲存的 `dict[str, TrainingDataDescriptor]` 遷移為 internal
`DescriptorEntry` repository；entry key 為
`DescriptorKey(origin, correlation_id, collection_group_id)`，value 才包含 unchanged wire
`TrainingDataDescriptor` 與 trusted metadata。

Origin 與 group 的生產邊界固定如下：

- existing AnLF PUT／DELETE route 與 request body 維持不變；router 呼叫 consumer-specific owner method，
  由 server-side code 固定注入 `origin=consumer_subscription`、`collection_group_id=None`，DELETE 也只作用於
  同一 origin key；
- collection manager 呼叫 private-specific owner method，固定注入 `origin=private_api`，並從 immutable
  request／profile snapshot 產生 `collection_group_id`；
- wire caller、Training Subscribe、callback payload 與 generic descriptor body 都不能提交或覆寫 origin／
  group；
- 同一 `correlationId` 跨 origin 或 private collection groups 不得互相覆寫或刪除。

`DescriptorOrigin = consumer_subscription | private_api` 不加入 wire schema。這個 migration 必須先以既有
AnLF PUT／DELETE characterization tests 固定 backward-compatible behavior，再加入 private-origin tests。

### 8.3 Dataset selection 與 absolute window

FL Client持有immutable `collection_trigger` setting。External preparation只選同origin descriptors：

- `consumer_subscription`只選AnLF relay descriptors；
- `private_api`只選local collection manager descriptors；
- 任一 trigger 失敗時不得 fallback 到另一個 trigger；
- Local mode既有policy flow不因federated Client setting而改變。

Training Subscribe 仍是 fitting admission authority。DatasetCoordinator 要求 event 相同；request 明確提供
filter／target時必須normalized match，未提供時沿用existing wildcard semantics。Private descriptors先按
collection request／profile分組；若一個preparation匹配多個groups，必須以ambiguous dataset失敗，不能
union任意profiles。選定group後再驗證SMF data subscription與single absolute requested window，並從
ADRF／Mongo freeze records。`READY`或descriptor存在不表示requested interval一定涵蓋；missing coverage／
`minNumSamples`不足仍preparation failure。

Private-origin retrieval對每筆UPF NotificationData計算排除ADRF storage wrapper的canonical content digest，
去除exact duplicate records後才建立observations。Correlation仍包含於resource ownership，不納入training
content digest；不同timestamps或measurement payload不視為duplicate。

所有 rounds 與 final validation 重用同一 immutable snapshot。不得 re-anchor、讀取 local path、hot reload
dataset 或用 TAI 篩選。

## 9. Go NWDAF MTLF-private 邊界

### 9.1 Route／processor／consumer 責任

在 existing `NWDAF/internal/mtlf` package 新增 route files 與 processor methods，不建立 new package：

```text
GET    /internal/v1/udm-sdm/group-data/group-identifiers
GET    /internal/v1/udm-uecm/{ueId}/registrations/smf-registrations
POST   /internal/v1/smf-event-exposure/subscriptions
GET    /internal/v1/smf-event-exposure/subscriptions/{subscriptionId}
PUT    /internal/v1/smf-event-exposure/subscriptions/{subscriptionId}
DELETE /internal/v1/smf-event-exposure/subscriptions/{subscriptionId}
POST   /internal/v1/adrf-data-management/data-store-records
```

Paths與AnLF-private baseline一致，但掛在MTLF internal server。Handler負責method／path／query／header／body
bound與ProblemDetails；processor接typed domain values並delegate；`internal/sbi/consumer`持有outbound public
SBI clients。`pkg/service/init.go`把existing `nwdaf.consumer`依required interfaces注入MTLF processor。

### 9.2 Standard-shaped 語意

- NRF／UDM／SMF／ADRF payload 與 status 依 local Release 18 OpenAPI；
- `Target-Api-Root`只接受HTTP(S) origin，不接受credentials、query、fragment或unexpected path；
- selected exact peer由PyMTLF resolution產生，Go仍validate target header；
- SMF create／read／replace／delete 轉送 `Location`、representation 與 operation-specific ProblemDetails；
- ADRF create轉送`201`、`Location`與representation；
- body limits 沿用／對齊 AnLF-private routes；
- private route不自動取得public OAuth／TLS語意，但outbound consumer維持existing NRF discovery、access-token
  與transport behavior；
- 不新增 public SBI route、NF registration service 或 OpenAPI generation change。

### 9.3 free5GC／本地 exemplar

Primary direct baseline是`NWDAF/internal/anlf/api_smf_event_exposure.go`、`api_udm_collection.go`、
`api_adrf_storage.go`及`internal/anlf/processor`；shared outbound owner是`internal/sbi/consumer`。Implementation
shape依workspace free5GC guidance維持server→handler→processor→consumer與`pkg/service/init.go` wiring。

Review 必須確認 new MTLF handlers 沒有 copy business logic、processor 沒有 parse raw HTTP、PyMTLF 沒有
繞過 Go consumer 直接 call NRF／UDM／SMF／ADRF public endpoints。

## 10. 本地跨 process 驗證

### 10.1 Support 流程

`nwdaf-resources`以dedicated private-collection runner建立單一data-owning Client／Leaf profile；不改變
hierarchical runner既有`smoke`／`aggregation` profile語意：

1. 啟動containing NWDAF MTLF private server、PyMTLF、ADRF／Mongo support與support NRF／UDM／SMF peers。
2. POST collection request，等待 resolution 與 Nsmf create。
3. Support SMF 保存 accepted subscription 與 callback URI，runner 以 exact correlation 重播 strict
   NotificationData。
4. 驗證 PyMTLF callback→ADRF／Mongo storage→local descriptor→`READY`。
5. 建立 matching Training Subscribe／component preparation，驗證 absolute window snapshot 與 sample count。
6. DELETE collection，驗證 SMF delete、retained descriptor、late callback `404` 與 TTL cleanup。
7. Restart scenario 在 active collection 時 terminate／restart PyMTLF，驗證 ledger recovery、peer cleanup、
   no automatic resume 與 new request 可在 cleanup 後建立。

### 10.2 Evidence 邊界

本地 support flow 直接測 production PyMTLF API、manager、callback、ingestion、descriptor 與
DatasetCoordinator，以及 production Go MTLF private handlers／processors。Support NRF／UDM／SMF 不是
real core；callback 由 runner injection，不宣稱 UPF 或 data-plane 通過。

Slice completion evidence需保存configs、profile digest、request/status transitions、resolved counts、SMF
create／delete calls、callback admission、storage result、descriptor／window／counts、restart cleanup與component
revisions。Raw SUPIs／group members 只保存於 protected temp evidence，不進 normal status 或 committed
fixture。

Real SMF→Nupf→UPF callback、UDM／UDR live data、OAuth／TLS、UE sessions與cross-VM network仍是
`integration verification gap`，留給testbed。

## 11. 預計修改檔案

### 11.1 `PyMTLF/`

- `src/py_mtlf/config.py`：private collection settings、profiles、consent、paths與strict matrix；
- `src/py_mtlf/app.py`：manager／router construction、generation reset與shutdown ordering；
- `src/py_mtlf/core/training_data_collection.py`：request／resource lifecycle、dedup、ledger、durable inbox、
  retry與cleanup；
- `src/py_mtlf/core/dataset.py`：`DescriptorEntry`／`DescriptorKey` owner、兩種trusted registration path、
  origin-aware selection與absolute window；
- `src/py_mtlf/models.py`：保留existing wire `TrainingDataDescriptor`；不加入origin／group欄位；
- `src/py_mtlf/core/fl_client.py`：Client setting傳入dataset admission；Branch delegation regression；
- focused relay client file under existing `src/py_mtlf/core/` package：MTLF-private NRF／UDM／SMF／ADRF
  request／response contract與bounded HTTP lifecycle；`NwdafContextClient`只供應trusted containing identity與
  `internal_api_root`，不擴張成procedure owner；
- `src/py_mtlf/core/ingestion.py`或等價existing owner：bounded callback queue、ADRF／Mongo writers；
- `src/py_mtlf/api/training_data_collection.py`、`wire/training_data_collection.py`：management API；
- `src/py_mtlf/api/callbacks.py`或existing callback router、`wire/upf_event_exposure.py`：strict callback；
- committed Client／Branch configs與direct tests。

若implementation顯示existing owners可更小幅容納behavior，可調整filename；但不得建立第二個manager／ledger
owner 或把 API router 當 state store。任何 new Python package 需先記錄 owner 與 dependency direction。

### 11.2 `NWDAF/`

- `internal/mtlf/server.go`：掛載 new route groups；
- `internal/mtlf/api_udm_collection.go`；
- `internal/mtlf/api_smf_event_exposure.go`；
- `internal/mtlf/api_adrf_storage.go`；
- `internal/mtlf/processor/processor.go`與focused files／tests；
- `pkg/service/init.go`：existing consumer注入；
- handler／processor／consumer contract tests。

不得新增 new Go package；若 existing interfaces 無法乾淨擴充，依 development policy 的 New Go Package
Gate 先進 decision gate。

### 11.3 `nwdaf-resources/`

- 重用distributed FL現有support process primitives、fake SMF與collection fixtures；
- 在`deployments/distributed_fl/scripts/run_private_collection.py`或等價focused script建立單一data-owning
  Client／Leaf flow，不新增hierarchical runner專用collection profile；
- support NRF／UDM／SMF collection behavior、private collection profile與reachable callback URI；
- collection API caller、status waiter、callback replay與cleanup assertions；
- focused checks與README；
- temporary evidence schema，不先加入Slice 3 paired numerical comparison。

## 12. 實作檢查點

1. 建立 AnLF baseline characterization、boundary tests 與 initial conformance map。
2. 實作PyMTLF settings／profiles／consent與API wire validation；尚未有manager時route保持disabled。
3. 實作durable request／resource ledger、idempotency、dedup與restart recovery tests。
4. 擴充Go MTLF UDM／SMF／ADRF routes、processor／consumer wiring與direct contract tests。
5. 實作PyMTLF standard resolution、Nsmf create／delete與provisional cleanup。
6. 實作UPF callback admission、bounded ingestion、ADRF／Mongo storage與descriptor lifecycle。
7. 實作DatasetCoordinator descriptor origin selection、absolute-window fitting與no-fallback regressions。
8. 以獨立private-collection runner擴充nwdaf-resources support flow，並執行local cross-process
   success／failure／restart／cleanup scenarios；hierarchical smoke／aggregation profiles保持Slice 1／3用途。
9. 執行 full repository verification、mandatory initial review、test-first remediation 與 fresh-read final
   conformance gate；保留 unstaged diffs 供 user review。

每checkpoint先建立deterministic failing test。若需要Root fan-out、user-consent UDM procedure、lease renewal、
public SBI或peer repository change，停止進decision gate。

## 13. 直接驗證矩陣

### 13.1 設定／API

- `consumer_subscription`／`private_api` legal matrix與all forbidden field combinations；
- callback／state relative path、consent、profile ID、group、DNN、S-NSSAI、sampling／count bounds；
- route 只在 Client＋private trigger 時啟用；
- POST／GET／DELETE、202／Location、same-ID replay、conflict、unknown profile／request、queue／generation 503；
- body不能overrideprofile；status no-sensitive disclosure。

### 13.2 Resolution／peer resources

- exact UDM SDM／UECM／SMF service selection；wrong group／SMF／service／registration rejection；
- DNN／S-NSSAI filters、incomplete group resolution與retry；
- no TAI query／profile requirement；
- Nsmf create body、Target-Api-Root、201／Location／representation；
- malformed create tracks provisional cleanup；finite expiry cleanup；
- per-key concurrency、reference dedup、last-release delete、delete retry與no substitute。

### 13.3 Callback／storage／descriptor

- unknown／retired correlation 404；invalid body／event／timestamp／UE address／measurements 400；queue full 503；
- accepted callback 204 only after durable inbox ownership；crash／restart drain與exact duplicate delivery；
- exact `NadrfDataStoreRecord` 使用 accepted `smfDataSub`；
- ADRF 201／Location success、transient retry、permanent drop、Mongo fallback與unavailable state；
- descriptor `ACTIVE`／`RETAINED` fields、stored window growth、origin、retention 與 TTL removal；
- AnLF relay descriptor仍由server-side標記consumer origin，wire schema不新增origin field；
- existing AnLF PUT／DELETE route只作用於consumer-origin key，不能覆寫或刪除private-origin entry。

### 13.4 Dataset／生命週期

- `DescriptorKey(origin, correlationId, collectionGroupId)` collision isolation、configured origin-only matching
  與 cross-trigger no-fallback；
- private request／profile grouping、wildcard match ambiguity rejection與private-origin exact-record dedup；
- Training event／filter／target／absolute window／min samples仍admission authority；
- READY但window mismatch或samples不足仍failure；
- immutable snapshot reuse、purged split與fitting seed regression；
- corrupt ledger fail closed；restart `RECOVERING` cleanup；no auto-resume；pending cleanup blocks duplicate；
- shutdown ordering、late callbacks、idempotent close與no orphan task／peer resource。

### 13.5 Go 邊界

- 每條 route 的 method／path／query／body bound；
- valid／invalid Target-Api-Root；
- standard success status／Location／representation forwarding；
- peer ProblemDetails forwarding與unavailable／bad gateway mapping；
- processor delegates toconsumer，generation／service wiring存在；
- existing AnLF routes 與 MTLF retrieval／model routes 不 regression。

### 13.6 必要指令

```bash
# PyMTLF/
.venv/bin/pytest -q \
  tests/test_config.py \
  tests/test_training_data_collection.py \
  tests/test_upf_event_exposure.py \
  tests/test_dataset.py \
  tests/test_training_data.py \
  tests/test_fl_client.py \
  tests/test_runtime_modes.py

.venv/bin/pytest -q
.venv/bin/ruff check src tests

# NWDAF/
go test ./internal/mtlf/...
make test
make lint
make build

# workspace root
PyMTLF/.venv/bin/pytest -q \
  nwdaf-resources/deployments/hierarchical_fl/checks \
  nwdaf-resources/deployments/distributed_fl/checks

PyMTLF/.venv/bin/ruff check \
  nwdaf-resources/deployments/hierarchical_fl \
  nwdaf-resources/deployments/distributed_fl

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/distributed_fl/scripts/run_private_collection.py \
  --scenario success

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/distributed_fl/scripts/run_private_collection.py \
  --scenario restart-cleanup
```

實際 focused filenames 與 runner CLI 可依 implementation existing interface 小幅調整，但需在 plan／conformance
map 同步更新，且保留等價 success、failure、restart 與 cleanup evidence。所有 code／script execution 需依
workspace policy 使用 elevated permission。

## 14. 規範符合性對照表

| ID | 要求 | Production owner | 直接證據目標 | 初始狀態 |
| --- | --- | --- | --- | --- |
| S2-CFG-01 | private trigger／profile／consent strict settings | PyMTLF config | config tests | Open |
| S2-CFG-02 | request只選profile；無target／peer override | API／config snapshot | API tests | Open |
| S2-API-01 | POST／GET／DELETE、idempotency、errors、no disclosure | PyMTLF API／manager | route tests | Open |
| S2-OWN-01 | one collection manager ownsrequest／resource／ledger lifecycle | app／manager | construction／ownership tests | Open |
| S2-OWN-02 | manager在existing generation reset與shutdown中先fence admission／durable ownership，且早於DatasetCoordinator／context關閉 | app／manager | lifecycle ordering與fault-injection tests | Open |
| S2-LED-01 | durable atomic ledger／inbox、corrupt fail closed | manager repository | restart／corruption tests | Open |
| S2-LIF-01 | states、READY semantics、retention與TTL | manager | transition tests | Open |
| S2-LIF-02 | restart cleanup-only recovery、no auto-resume | manager／ledger | restart tests | Open |
| S2-RES-01 | Internal Group ID→UDM→serving SMF exact resolution | resolver | boundary tests | Open |
| S2-RES-02 | private collection no-TAI contract | resolver／profile | query／config tests | Open |
| S2-SMF-01 | Nsmf create／accepted identity／provisional cleanup | manager／Go relay | peer tests | Open |
| S2-SMF-02 | key dedup、references、last-release delete／retry | manager | concurrency tests | Open |
| S2-CB-01 | correlation／schema／durable inbox／duplicate admission | callback router／ingestion | callback tests | Open |
| S2-STO-01 | accepted subscription→ADRF record；ADRF／Mongo semantics | ingestion／writers | storage tests | Open |
| S2-DES-01 | correlation-only repository遷移為trusted origin／group key；AnLF wire與PUT／DELETE semantics不變 | manager／dataset repository | descriptor collision與AnLF compatibility tests | Open |
| S2-DATA-01 | origin／request grouping、absolute window、ambiguity／duplicate rejection、no fallback | DatasetCoordinator | dataset／Client tests | Open |
| S2-GO-01 | MTLF-private UDM relay | Go handler／processor／consumer | Go tests | Open |
| S2-GO-02 | MTLF-private SMF relay | Go handler／processor／consumer | Go tests | Open |
| S2-GO-03 | MTLF-private ADRF store relay | Go handler／processor／consumer | Go tests | Open |
| S2-GO-04 | existing package／service wiring；無 public SBI／new package | mtlf／service | structural review＋tests | Open |
| S2-E2E-01 | dedicated runner驗證support SMF callback→storage→descriptor→dataset success | resources／production owners | private-collection local run | Open |
| S2-E2E-02 | dedicated runner驗證delete／late callback／restart cleanup | resources／production owners | private-collection failure runs | Open |
| S2-REG-01 | consumer subscription、existing MTLF／AnLF flows不regression | all baseline owners | focused／full suites | Open |
| S2-VER-01 | PyMTLF／NWDAF／resources required commands通過 | repositories | §13.6 | Open |
| S2-REV-01 | mandatory review、fresh-read conformance、language pass | all diffs／docs | review handoff | Open |

Real SMF／UPF、UDM／UDR live state、UE session、OAuth／TLS與testbed network evidence維持Open integration gap，
不在本Slice conformance map標Satisfied。

## 15. Review 與完成閘門

### 15.1 初次 review

Focused implementation tests 通過後，立即 review 三個 repository 的完整 Slice diff：

- end-to-end每個value有authoritative producer、transport、owner、storage與failure behavior；
- PyMTLF profile與Training Subscribe authority沒有混淆；
- callback correlation不是自我驗證或dataset identity；
- descriptor origin是internal trusted metadata，不由wire caller提供；
- ledger涵蓋restart與pending cleanup，沒有orphan SMF resource；
- Go handlers／processors／consumers 責任分離，new files 全在 existing packages；
- AnLF baseline semantics重用但未copy analytics runtime；
- tests直接跑required owner／entry point，不用mock bypass claimed boundary；
- unchanged peer repositories確實沒有unauthorized changes。

Confirmed in-scope finding依development policy test-first remediation並做targeted follow-up review。需要Root
fan-out、consent retrieval、lease renewal、public contract或peer repository change時進decision gate。

### 15.2 最終閘門

交付user review前：

1. 從disk完整重讀development policy與本Slice；
2. 依 latest text 重建全部 normative conformance map；
3. 執行§13.6 commands，分別記錄PyMTLF、NWDAF、resources結果；
4. 對每個Satisfied item定位production path、direct test與command；
5. 檢查每個affected repository status、完整unstaged diff與unrelated changes；
6. 重新完整讀取本文件並與父計畫／Slice 1／同系列sibling比對，完成繁體中文language pass；
7. 狀態保持`Ready for User Review`，不stage、不commit、不標Completed。

交接內容列出 affected repositories、diff summary、actual tests、support-vs-real integration boundary、open
testbed／consent gaps、conformance state 與 unrelated changes。User review 確認後才準備
repository-separated commit proposal。

## 16. 明確延後項目

- Root／Server collection fan-out與cross-NWDAF collection-control contract；
- `anyUe`、direct SUPI／GPSI private collection；
- UDM user-consent retrieval／change subscription；
- finite lease renewal與active collection automatic restart resume；
- TAI／network-area private collection profile；
- HFL FedAvg與paired numerical comparison；
- real SMF→Nupf→UPF、UE session、data-plane與cross-VM testbed execution；
- hot reload profile、distributed durable collection coordinator與multi-process shared ledger。

Consent-required deployment在future consent slice前不得啟用private collection。Local support E2E完成後仍只
能宣稱`Local Collection E2E Passed；Testbed Validation Pending`。
