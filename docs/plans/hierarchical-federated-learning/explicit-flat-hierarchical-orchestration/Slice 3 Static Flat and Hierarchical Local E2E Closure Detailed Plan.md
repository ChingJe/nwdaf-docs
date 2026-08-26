# Slice 3：Static Flat 與 Hierarchical 本地 E2E 收尾詳細計畫

日期：2026-08-26

狀態：Implementation Verified；Mandatory Review Complete；User Review Confirmed；Commit Pending Approval；testbed validation pending

相關文件：

- [Explicit Flat and Hierarchical Orchestration Configuration Detailed Plan](./Explicit%20Flat%20and%20Hierarchical%20Orchestration%20Configuration%20Detailed%20Plan.md)
- [Slice 1：Explicit Orchestration 與 Static／Manual Training 詳細計畫](./Slice%201%20Explicit%20Orchestration%20and%20Static%20Manual%20Product%20Flow%20Detailed%20Plan.md)
- [Slice 2：MTLF-Triggered UPF Data Collection 詳細計畫](./Slice%202%20MTLF-Triggered%20UPF%20Data%20Collection%20Detailed%20Plan.md)
- [Slice 5 Hierarchical Rounds and Aggregation Detailed Plan](../Slice%205%20Hierarchical%20Rounds%20and%20Aggregation%20Detailed%20Plan.md)
- [Slice 8 Multi-process E2E and Regression Closure Detailed Plan](../Slice%208%20Multi-process%20E2E%20and%20Regression%20Closure%20Detailed%20Plan.md)
- [NWDAF Development Policy](../../../development_policy.md)
- [Release 18 Nnwdaf ML Model Training OpenAPI](../../../../specs/openapi/TS29520_Nnwdaf_MLModelTraining.yaml)
- [Release 18 NRF NF Management OpenAPI](../../../../specs/openapi/TS29510_Nnrf_NFManagement.yaml)

---

## 1. Slice 目標與完成邊界

本 Slice 在 Slice 1 的 explicit static／manual orchestration 與 Slice 2 的 MTLF-triggered collected dataset
上，分別完成兩條可重複執行的本地 real-process E2E：

1. static flat＋既有 FedAvg；
2. hierarchical＋既有 FedProx。

兩條流程使用各自既有的 algorithm contract，分開啟動、分開保存 evidence，也分開判定成功或失敗。本
Slice 不要求兩者使用相同資料、產生相同 local weights、得到相同 global model 或通過跨 topology 數值
tolerance。兩邊的結果可保存供後續觀察，但不構成本 Slice 的 equivalence claim。

本 Slice 完成後：

1. Private collection 移除 minimum observation gate 與 `READY`；caller 明確 POST 後持續蒐集，直到明確
   DELETE，collection 不自動停止或觸發 training。DELETE cleanup後，有durable stored data進
   `RETAINED`，無data進`TERMINATED`。
2. Top-level Server 維持既有 current-time preparation window 與 `minNumSamples: 1`，不新增 lag、sample
   threshold config 或 request override；DatasetCoordinator 接受 descriptor 與 requested window 的部分交集，
   並只使用 interval 內實際 records。
3. Static flat 保留 topology／NRF／Training TAI；每個 Client 的 local collection profile 使用相同 TAI，
   交給 SMF gating group members。Production flat `monitor_scopes` 的 area semantics 保持不變。
4. Static flat scenario 以四個 Clients、private collection、retained descriptor 與既有 FedAvg 完成完整
   manual training lifecycle。
5. Hierarchical topology、assignment、resolver 與 Training Subscribe 維持 no-TAI；四個 Leaves 的 local
   collection profiles 可各自配置 non-overlapping TAI，只作 SMF ingress gating，再以既有 FedProx 完成
   preparation、two-tier rounds、validation、publication 與 cleanup。
6. PyMTLF local training artifact 保存 deterministic dataset evidence，證明每個 scenario 實際使用的
   observations、training tensors、validation tensors 與 sample counts。
7. Runner 可在各 scenario 內獨立重算 flat one-tier 或 HFL Branch／Root two-tier aggregation；重算只驗證
   該 topology 自身的 production result。
8. 既有 production flat isolated flow、既有 HFL FedProx flow及受影響 lifecycle tests 完成 regression。
9. Local evidence 明確區分 production owners 與 support UDM／SMF／callback replay；本 Slice 完成後仍是
   `Testbed Validation Pending`。

本 Slice 不新增 algorithm variant，不修改 public SBI、NF profile schema、NRF role、standard TAI wire
schema／SMF gating semantics、private training request body 或 private collection request body，也不實作
Root／Server collection fan-out、dynamic topology、跨 topology numerical equivalence 或 model-quality
benchmark。Transport security 不在本計畫的變更與驗證範圍，也不列為 testbed acceptance gate。

## 2. 基準版本與 repository 責任

### 2.1 計畫基準

| 儲存庫 | 版本 | 本 Slice 定位 |
| --- | --- | --- |
| `PyMTLF/` | `aa65204387b38162d87b1c478d64c724548a10c3` | collection lifecycle／TAI、window admission、dataset evidence 與 direct tests |
| `nwdaf-resources/` | `0d839d167dedb6235b8bfbe97e346a95dc53580d` | 兩條獨立 process graphs、private collection replay 與 aggregation evidence |
| `nwdaf-docs/` | `03b7822f2c981e5a760b5c415cf18a0a626f2bb4` | canonical plan、review 與 implementation record |
| `NWDAF/` | `6aed268d6528f8be6c729cbd45b59d067e5e80dc` | existing Training SBI 與 Slice 2 MTLF-private relays；verification-only baseline |
| `PyAnLF/` | `6a4d94ad3cc6f66dac55ea921772d731e4b71371` | production flat `consumer_subscription` regression baseline |
| `nrf/` | `0dd4024d4ab75b6630e04901968228b9b9718cf5` | exact-instance discovery baseline |
| `adrf/` | `905f0599f68fe389bba14ed56db0ef9abeab5ccd` | real local storage／retrieval／model publication baseline |

開始實作前重新確認 revisions 與 working trees。Revision 若移動，先更新受影響的 characterization 與
conformance map，不直接沿用本節證據。

### 2.2 預計修改

- `PyMTLF/`：移除 minimum observation／`READY` contract、加入 collection network area、調整 descriptor
  matching／window overlap admission、加入 dataset evidence digest、artifact metadata／validation，以及
  direct regression tests。
- `nwdaf-resources/`：四-Client static flat scenario、Root／Branches／四-Leaf hierarchical scenario、support
  collection peers、pinned observations、evidence schema、各 topology 的 independent recomputation 與 checks。
- `nwdaf-docs/`：本 Slice 狀態、conformance map、implementation／verification evidence 與父計畫狀態。

### 2.3 預期不修改

- `NWDAF/`：existing generic training API、Training relay 與 MTLF-private collection relays 足以承載本
  Slice；只執行受影響的 read-only regression 或 real-process usage。
- `PyAnLF/`：兩條新 scenario 都不啟動 PyAnLF；existing production flat runner 仍用它作 regression。
- `nrf/`、`adrf/`：使用現有 production processes，不改 contract。
- `smf-nwdaf-ext/`、`udm/`、`udr/`：local runner 以 support peers 補足，real peers 留待 testbed。
- `resources/`：維持 read-only。

若 deterministic test 證明預期不修改的 repository 有 current-Slice contract defect，先依 development
policy 進 decision gate；runner failure 不自動授權跨 repository 修改。

## 3. 已確認的現況與缺口

### 3.1 可直接重用的 production 行為

| 現有行為 | 本 Slice 結論 |
| --- | --- |
| Flat algorithm | Flat Client local round 已使用 task loss、`proximal_mu=None` 與 sample-weighted FedAvg。 |
| HFL algorithm | Root assignment、Branch delegation與Leaf fitting已使用FedProx；Branch／Root仍按effective sample counts聚合。 |
| Dataset split | `TrainingDatasetBuilder` 使用 purged chronological split，不是隨機切分；`random_seed` 仍固定 local fitting。 |
| Collected snapshot | Slice 2 已以 descriptor origin／group、event、target 與 absolute window freeze ADRF／Mongo records。 |
| Collection lifecycle | Private collection 已支援 DELETE、RETAINED、late-callback rejection 與 restart cleanup-only recovery；現行 minimum observation／`READY` 將在本 Slice 移除。 |
| Static initiation | Flat 與 HFL 已共用 `/internal/v1/federated-learning/training-requests`，request 不可覆寫 topology、strategy 或 window。 |
| Artifact lineage | Local、Branch aggregate、global 與 final bundles 已保存 weights digest、participant identities、sample counts 與 round lineage。 |

### 3.2 本 Slice 必須關閉的 production 缺口

1. Private descriptor 的 available window 只涵蓋 actual stored measurements；DatasetCoordinator 現行要求
   descriptor 完整覆蓋 Training Subscribe 的 requested window。Server window 截止於 preparation 當下，
   因此 caller 先停止 collection 再觸發 training 時必然缺少最後一段 coverage。正確語意是 descriptor 與
   requested window 有交集，retrieval 只取 interval 內 records，再由 sample construction 判定是否足夠。
2. Slice 2 private collection profile 禁止 TAI，無法沿用既有 SMF network-area gating。結果是每個
   data-owning NWDAF 解析同一 group 後都會收集所有 SUPI，也使 static flat Training Subscribe 的
   topology-derived `networkArea` 無法匹配 private descriptor。
3. Slice 2 的 `minimum_observation_count` 只把成功保存的 notification items 累計到 `READY`，不停止
   collection、不觸發 training，也不證明 requested interval 或 training samples 足夠；在 caller-owned
   explicit create／delete lifecycle 中沒有獨立必要性。
4. Existing local artifact 只有 scope digest 與 training sample count，尚無足以直接證明 collected records
   確實形成該次 local fitting input 的 dataset evidence。
5. Existing hierarchy runner 使用 PyAnLF consumer-triggered collection；Slice 2 focused runner只有單一
   Client collection。目前沒有四-Client static flat private-collection E2E，也沒有四-Leaf hierarchical
   private-collection E2E。

HFL FedAvg 不屬於本 Slice 缺口。Flat FedProx 亦不在本 Slice 實作，因為 flat Client 現在沒有接收
algorithm／`proximal_mu` 的 contract；該能力留待未來另行設計。

正常collection state path固定為`PENDING → RESOLVING → SUBSCRIBING → COLLECTING`。Caller從任一
nonterminal state DELETE後進`STOPPING`；cleanup完成時依是否已有durable stored data分別進`RETAINED`或
`TERMINATED`。Failure與restart cleanup-only paths維持Slice 2既有語意。

## 4. Requested window admission 契約

### 4.1 保留既有 Server 設定

Top-level Server 維持既有設定：

```yaml
federated_learning:
  server:
    preparation_data_window_seconds: 1800
```

固定規則：

- Server 固定產生 `[now - duration, now]`；authoritative clock 與 duration 仍由 Server config 擁有；
- generic training request 與 collection request 都不能覆寫 window；
- 不新增 `preparation_data_lag_seconds` 或其他固定 delay；
- `minNumSamples` 維持現行 `1`，本 Slice 不新增 sample-threshold config 或自動推算；
- DatasetCoordinator 要求每個 selected descriptor 的 available window 與 requested window 至少有交集；
- retrieval 與 dataset builder 仍只接受 requested interval 內的 records，不 re-anchor、不把 interval 外資料
  搬入；
- interval 內無 eligible records、scope sample construction 不足或現行 `minNumSamples` admission 不通過時，
  preparation 失敗。

### 4.2 每個 scenario 的資料時間

每條 scenario 獨立建立 historical fixture anchor，並在觸發 top-level training 前完成 pinned observations
replay與manual DELETE。`preparation_data_window_seconds`必須大於該scenario的observation跨度加上從earliest
fixture到training request的bounded execution budget，讓actual requested interval包含足夠records。

Runner 不製造只為滿足 descriptor coverage 的 boundary records，也不修改 process clock。Summary 保存
actual requested window、descriptor available window、兩者交集與實際 sample counts；若 runtime 超出配置的
window budget或interval內samples不足，scenario failed，不動態改寫window。

## 5. Topology TAI 與 collection TAI 分層

### 5.1 Collection network area

每個 data-owning Client／Leaf 的 private collection profile 加入 typed local network area：

```yaml
network_area:
  tais:
    - plmn_id:
        mcc: "466"
        mnc: "92"
      tac: "001101"
```

- controlled static profile 至少包含一個 canonical TAI，duplicate／invalid values 在 startup fail；
- collection request 仍只選 `collectionProfileId`，不得覆寫 network area；
- PyMTLF 將 local TAI 映射到 Nsmf `eventSubs[].networkArea`；accepted representation 必須保留相同值；
- canonical network area 納入profile digest、request snapshot與peer `CollectionKey`；不同area不得share同一
  peer resource或callback correlation；
- 每個 data owner 仍解析同一 Internal Group 的全部 SUPI。SMF 以各 SUPI current TAI gating，只有 matching
  UE 建立 Nupf subscription並產生data；out-of-area waiting resource不計為collected data；
- descriptor 的 `mLEventFilter.networkArea` 保存 accepted collection provenance。DatasetCoordinator 不從
  notification row 反推 TAI，也不在 ADRF／Mongo records 上二次 area filter。

本 Slice 與 testbed 假設 UE 在 scenario 期間保持 static TAI；不實作 mobility 後的 subscription
reconciliation。各 data owner 的 local TAI 必須 non-overlapping，確保同一 SUPI 只由一個 NWDAF 收到資料。
本地runner以同一份static inventory定義group membership、每個SUPI的current TAI與唯一expected owner；
support SMF依收到的`networkArea`計算downstream-eligible SUPIs，讓本地evidence能直接檢查group雖相同，
每個SUPI仍恰好只屬於一個data owner。這是support boundary驗證，不取代testbed的real SMF／Nupf證據。

### 5.2 Static flat

- Static topology 繼續保存 exact Client ID 與 tracking areas；
- topology TAI 繼續供 NRF exact discovery、profile validation 與既有 participant
  `ml_event_filter.networkArea`／Training Subscribe scope；
- 每個 Client local collection TAI 是獨立 local config owner。Root 無法透過既有 Training contract讀取它；
  runner／testbed config generator 必須由同一 inventory 產生 topology 與 Client profile，preflight 驗證
  exact match與cross-Client non-overlap；
- production flat `monitor_scopes` 繼續沿用 Monitor／AnLF area filter，不因 private collection 改變。

### 5.3 Hierarchical

- HFL topology、assignment、resolver 與 Training Subscribe 繼續只使用 Root／Branch／Leaf identities、
  service、capability、event與interoperability，不加入TAI；
- Leaf local collection TAI只供SMF ingress gating，不成為HFL participant selection或training scope；
- 當 Training request 未指定 area 時，descriptor 多出的 local `networkArea` 是允許的 provenance；request
  明確提供的其他 filter fields仍必須匹配。這項 matching 不得放寬 explicit area constraint的既有
  production flat行為。

Local Release 18 OpenAPI 將兩個 contract 分開：NRF profile 的 `MlAnalyticsInfo.trackingAreaList` 屬 discovery
capability，Training Subscribe 的 `MLEventSubscription.mLEventFilter` 與 Nsmf
`eventSubs[].networkArea`又是不同方向的fields。本Slice只修正既有local owners間的value propagation，
不修改generated model或OpenAPI schema。

## 6. Dataset evidence 契約

### 6.1 Evidence 目的

Runner 送出 fixture 只證明 ingress intent；collection、storage、descriptor matching、window admission、
timestamp aggregation 與 purged split 都可能改變實際 fitting input。因此 PyMTLF 必須在建立
`TrainingDataset` 後產生不含 raw records 的 deterministic evidence。

Evidence 用來證明每條 scenario 自己實際使用的 dataset，不用來要求 flat 與 HFL 相同。

### 6.2 Evidence 格式

每個 `ROUND_LOCAL(TRAINING)` artifact 至少保存：

```yaml
dataset_evidence:
  contract_digest: <sha256>
  observation_digest: <sha256>
  training_tensor_digest: <sha256>
  validation_tensor_digest: <sha256>
  observation_count: <positive integer>
  training_sample_count: <positive integer>
  validation_sample_count: <positive integer>
```

固定 canonicalization：

- scopes 依 `scope_digest` 排序；
- field／tensor order、shape 與 dtype 納入 hash；
- floating values 轉為明確 little-endian IEEE-754 representation；
- timestamps 轉為 UTC canonical representation；
- counts 與實際 arrays及artifact `training_sample_count`一致；
- 不納入request ID、subscription ID、callback correlation、descriptor correlation、peer URL、delivery
  order、process path或archive metadata；
- 只保存digests與counts，不保存raw UE／measurement data。

### 6.3 Artifact role 規則

- `ROUND_LOCAL(TRAINING)` 必須有 `dataset_evidence`；
- `ROUND_LOCAL(HIERARCHY_AGGREGATE)` 沒有 local dataset，必須禁止 `dataset_evidence`；
- `ROUND_LOCAL(ACCURACY_CHECK)` 沿用 existing evaluation contract；
- existing flat FedAvg 與 HFL FedProx local artifacts 使用同一 evidence contract；
- Server驗證typed evidence與sample count一致，但不從caller digest反向推導expected value。

目前 `RoundLocalHierarchyAggregateMetadata` 繼承 training metadata。若 required evidence 會迫使 Branch 宣告
不存在的 local dataset，需將 shared round／participant／count fields 下移到 common owner，讓 training 與
hierarchy aggregate 成為明確 siblings；不得把 `dataset_evidence` 變成所有 result types 都可任意省略。

## 7. 兩條獨立 scenario

### 7.1 Static flat＋FedAvg

```text
Flat Server
  ├─ Client A1
  ├─ Client A2
  ├─ Client B1
  └─ Client B2
```

- topology 列出四個 exact Client IDs 與各自 TAI；
- resolver 驗證 identity、Training service、FL Client capability、event、interoperability 與 TAI；
- Training Subscribe 保留 topology-derived `networkArea`；
- 每個 Client 使用 `collection_trigger: private_api`，local collection profile配置同一TAI，由runner分別
  觸發；preflight驗證topology／profile exact match與四個Clients間non-overlap；
- 四個 Clients 的 profile 使用同一 Internal Group ID；support inventory 將每個 group member 指派到唯一
  static TAI／Client，並驗證其他三個 Clients 對該 SUPI 都是 out-of-area；
- local fitting 維持 `proximal_mu=None`；
- Flat Server 按 actual training sample counts 執行 one-tier FedAvg；
- 至少兩個 Clients 的 sample counts 不同，避免等權測試掩蓋 weighting defect。

### 7.2 Hierarchical＋FedProx

```text
Root
  ├─ Branch A
  │   ├─ Leaf A1
  │   └─ Leaf A2
  └─ Branch B
      ├─ Leaf B1
      └─ Leaf B2
```

- 使用既有 static hierarchy topology 與 FedProx strategy；
- Root 將 immutable strategy 與 positive `proximal_mu` 傳給 Branches／Leaves；
- HFL topology、assignment、resolver 與 Training Subscribe 不配置或驗證 TAI；
- 四個 Leaves 使用 `collection_trigger: private_api`，local collection profiles各自配置non-overlapping TAI，
  由runner分別觸發；這些TAI不進入hierarchy assignment或training scope；
- 四個 Leaves 的 profile 使用同一 Internal Group ID；support inventory 將每個 group member 指派到唯一
  static TAI／Leaf，並驗證其他三個 Leaves 對該 SUPI 都是 out-of-area；
- Branches 不配置 local collection profile、不執行 local fitting、不加入 local samples；
- Branch effective count 等於 subordinate Leaf counts 總和；Root 使用 Branch effective counts；
- 至少兩個 Leaves 的 sample counts 不同。

### 7.3 Scenario 隔離

兩條 scenario 不共用 runtime、request IDs、storage namespace 或成功狀態。可以重用 support helpers 與 fixture
生成器，但每條 scenario 必須：

- 使用 fresh temporary root 與 isolated ADRF／Mongo namespace；
- 完整啟動、驗證 readiness、執行、保存 evidence、teardown 與 deregistration；
- 產生自己的 `summary.json`；
- 可單獨執行與單獨失敗，不以另一條成功補足；
- 不宣稱 trainer identities、dataset digests、weights 或metrics跨scenario相同。

## 8. 每條 scenario 的 collection→training 流程

1. 啟動 production NRF、ADRF、MongoDB、Go NWDAF 與 PyMTLF process graph，以及 support UDM／SMF。
2. 確認 data owners ready、NRF registrations 與 topology-specific discovery constraints；support UDM 對所有
   owners回覆同一group membership，support SMF載入同一份SUPI current-TAI inventory。
3. Runner 逐一 POST `/internal/v1/training-data-collections`；body只含`requestId`與`collectionProfileId`。
4. 等待 requests 進入 `COLLECTING`，取得 server-generated callback correlations；驗證每個Nsmf request的
   `networkArea`、support SMF計算的downstream-eligible SUPIs，以及SUPI→唯一expected owner partition。
5. 只為matching owner的eligible SUPI重播pinned observations到production
   `/callbacks/upf-event-exposure`；禁止future timestamp或對out-of-area owner注入fixture。
6. Requests保持`COLLECTING`；等待已知fixtures成功保存，驗證counts、stored window、storage transport與
   descriptor state。這是runner對自身fixture的確認，不是production minimum gate。
7. Runner作為explicit caller DELETE collections，等待`RETAINED`，驗證exact SMF resources已刪除；late
   callback必須`404`。
8. POST generic training request 到 Flat Server 或 Root；request 不攜帶 collection profile、window、strategy
   或 target override。
9. 等待 preparation、configured rounds、final validation、publication 與 terminal status。
10. 保存 artifacts、actual requested window、dataset evidence、aggregation evidence與cleanup結果。
11. 完整停止 processes，確認 deregistration、ports 與 runtime ownership 收斂。

這個 ordering 直接驗證caller-owned create／delete、retained descriptor與後續training，並避免training期間
的新callback改變snapshot。Descriptor retention必須長於單一scenario的bounded deadline；runner不得修改
ledger、偽造collection state或直接注入descriptor。Production PyMTLF不自動DELETE或觸發training。

## 9. 各 topology 的獨立重算

### 9.1 Flat round

對每個 round `r`：

1. 從四個 `ROUND_LOCAL(TRAINING)` bundles 讀取 weights、dataset evidence 與 actual sample counts；
2. 驗證每個 local output 的 base weights、round lineage 與sample count；
3. Runner 獨立計算 one-tier sample-weighted average；
4. 與 Flat Server 發布的 `ROUND_GLOBAL(r)` 逐 tensor 比較。

### 9.2 HFL round

對每個 round `r`：

1. 從 A1／A2 與 B1／B2 local bundles 驗證 FedProx local lineage 與 actual counts；
2. 分別重算 Branch A／B lower aggregates；
3. 驗證 Branch artifacts 的 subordinate identities、local digests 與 effective counts；
4. 以兩個 Branch aggregates 及 effective counts 重算 Root aggregate；
5. 與 Root `ROUND_GLOBAL(r)` 逐 tensor 比較。

### 9.3 數值規則

每條 scenario 內的 independent recomputation 預設使用 `rtol=1e-5`、`atol=1e-7`。Tolerance 集中定義並
寫入各自 summary；若失敗，保存 first divergent round／tensor／max absolute 與 relative error，不得只為
green run 放寬 threshold。

這個 tolerance 只比較「runner 重算結果」與「同 topology production result」。Flat 與 HFL 之間不執行
local／global／final model tolerance assertion。

## 10. Runner 與 evidence 責任

### 10.1 執行入口

在 `nwdaf-resources` 建立可重用 support layer與兩個獨立scenario入口。可由同一script以不同scenario選擇，
但不得有隱含的paired success。建議命令：

```bash
PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/scripts/run_static_collection_e2e.py \
  --scenario flat-fedavg

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/scripts/run_static_collection_e2e.py \
  --scenario hierarchical-fedprox
```

實際 filename 可在 implementation 時依 existing owner 小幅調整。Runner 可重用 hierarchy `support.py`、
distributed FL observation builders、Slice 2 private-collection helpers與existing two-tier recomputation
helpers；不得建立另一套 production orchestrator 或直接改寫 repository configs。

### 10.2 每條 summary

每次執行至少保存：

- schema version、scenario name、run ID、開始／結束時間；
- exact repository revisions 與 production binary digests；
- NF ID→topology position mapping；
- Internal Group membership、SUPI→static TAI→expected owner mapping、每個owner的requested area與support
  SMF downstream-gating結果；
- fixture spec、absolute anchor、canonical fixture digests 與 counts；
- actual requested windows、collection status、retained descriptor 與 late callback 結果；
- per-trainer dataset evidence 與 actual sample counts；
- algorithm、epochs、rounds 與 fitting settings；
- per-round local／Branch／global artifact lineage；
- 該 topology 的 independent recomputation 與 tolerance；
- final validation、ADRF publication、catalog state 與 cleanup；
- verification levels 的 `passed | failed | not_run` 與 reason；
- production／support／not-claimed boundary。

任何階段失敗仍保存 partial summary、configs、logs、bundles與first divergence diagnostics。Cleanup只處理本次
temporary root與本次啟動的processes；不得編輯、checkout或刪除repositories。

## 11. 失敗、重啟與 regression 邊界

- 任一 fixture 未在 collection status 的 counts／stored window 中出現、retention失敗、dataset evidence不一致
  於自身artifact、sample count mismatch或topology-internal recomputation mismatch，都使該scenario failed；
- flat failure 不影響 hierarchical scenario 的可執行性，反向亦同；
- process timeout 使用 bounded wait 並輸出 process logs；不得無限等待；
- teardown failure 與 training／aggregation failure 分開記錄；
- existing failure、timeout、late callback、generation reset 與 cleanup semantics 不因 runner 擴充而改變；
- production flat isolated runner 必須重新執行，證明 static changes 未破壞 `monitor_scopes`／
  `consumer_subscription` flow；
- existing HFL FedProx aggregation runner 必須重新執行，證明 private-collection scenario 未改變既有 HFL
  lifecycle。

## 12. 實作檢查點

1. 從本文件建立 normative conformance map，確認 revisions、working trees與active artifact callers。
2. 先為explicit create／delete lifecycle加入failing tests，移除minimum observation config／status與
   `READY` transition；保留counts、descriptor growth、DELETE／retention／restart cleanup semantics。
3. 為collection network area加入typed config、Nsmf payload／accepted representation、descriptor provenance
   與SMF gating boundary tests；確認static flat topology／Training TAI保留，HFL topology／Training仍無TAI。
4. 加入descriptor／requested window partial-overlap admission tests，確認no-overlap rejection、interval內
   record filtering、no re-anchor與現行`minNumSamples: 1`不變，再修正DatasetCoordinator coverage gate。
5. 定義 dataset evidence canonicalization 與 direct digest tests，包含 dict／scope order、endianness、
   timestamp、single-value change、split change 與 no-raw-data assertions。
6. 將 typed dataset evidence 加入 local training artifact，更新 producer、validator、Server acceptance 與
   existing flat／HFL artifact tests。
7. 擴充resources generators，先建立四-Client static flat private-collection scenario，完成TAI一致性
   preflight與單條black-box success。
8. 加入 flat one-tier independent recomputation、failure summary 與 cleanup checks。
9. 建立 Root／Branches／四-Leaf hierarchical FedProx private-collection scenario，重用已驗證的 collection
   helpers 與 HFL topology owners。
10. 加入 Branch／Root independent recomputation、lineage、effective counts與cleanup checks。
11. 執行兩條新scenario、production flat isolated與existing HFL FedProx regressions。
12. 執行PyMTLF／resources focused與full verification，完成mandatory initial review。
13. 依finding admission gate對in-scope defect作test-first remediation與targeted follow-up review。
14. 重新完整讀取development policy與本文件，重建final conformance，再執行final full verification。
15. 更新implementation record與狀態為`Ready for User Review`，保持所有changes unstaged／uncommitted。

## 13. 驗證矩陣

### 13.1 PyMTLF 聚焦測試

```bash
# PyMTLF/
.venv/bin/pytest -q \
  tests/test_config.py \
  tests/test_training_data_collection.py \
  tests/test_collection_relay.py \
  tests/test_upf_event_exposure.py \
  tests/test_dataset.py \
  tests/test_fl_topology.py \
  tests/test_fl_flat.py \
  tests/test_fl_server.py \
  tests/test_fl_hierarchy.py \
  tests/test_fl_root.py \
  tests/test_fl_branch.py \
  tests/test_fl_client.py \
  tests/test_local_trainer.py \
  tests/test_training_data.py \
  tests/test_fl_artifacts.py \
  tests/test_fl_hierarchy_artifacts.py
```

Focused coverage必須直接包含manual collection lifecycle、collection network area、static flat與HFL TAI分層、
partial-overlap window admission、dataset evidence、flat one-tier、HFL FedProx two-tier、failure、timeout、
generation與cleanup paths。

### 13.2 PyMTLF 完整驗證

```bash
# PyMTLF/
.venv/bin/pytest -q
.venv/bin/ruff check src tests
```

### 13.3 Resources 檢查

```bash
# workspace root
PyMTLF/.venv/bin/pytest -q \
  nwdaf-resources/deployments/hierarchical_fl/checks \
  nwdaf-resources/deployments/distributed_fl/checks

PyMTLF/.venv/bin/ruff check \
  nwdaf-resources/deployments/hierarchical_fl \
  nwdaf-resources/deployments/distributed_fl

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/checks/preflight.py
```

### 13.4 本地 real-process E2E

```bash
# static flat＋private collection＋FedAvg
PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/scripts/run_static_collection_e2e.py \
  --scenario flat-fedavg

# hierarchical＋private collection＋FedProx
PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/scripts/run_static_collection_e2e.py \
  --scenario hierarchical-fedprox

# production flat regression
PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/distributed_fl/scripts/run.py

# existing HFL FedProx regression
PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/scripts/run.py \
  --profile aggregation --scenario degradation-success
```

實際 filename 可在 implementation record 更新，但四種不同 claims 均須有直接 evidence。Support callback
replay 不得描述為 real SMF／UPF data-plane evidence。

### 13.5 NWDAF 條件式驗證

預期 NWDAF 無 production diff。兩條 runner 實際通過 existing Go relays 與 Training SBI 即提供
cross-process regression。只有 NWDAF 出現 working-tree change 時，才額外執行：

```bash
# NWDAF/
make test
make lint
make build
```

所有 code／script commands 依 workspace policy 使用 elevated permission。Documentation-only planning 階段
不執行 code tests，只做 document 與 diff 檢查。

## 14. 規範符合性對照表

| ID | 要求 | Production／evidence owner | 直接證據目標 | 最終狀態 |
| --- | --- | --- | --- | --- |
| S3-LIF-01 | explicit collection維持COLLECTING直到caller DELETE；移除minimum gate／READY／auto-actions | PyMTLF config／manager／wire | config＋state transition＋API tests | Satisfied |
| S3-LIF-02 | DELETE cleanup後依durable stored data進RETAINED或TERMINATED | PyMTLF manager／wire | state transition＋API tests | Satisfied |
| S3-WIN-01 | 保留existing current-time window與`minNumSamples: 1`；無lag或request override | FL Server | characterization＋request tests | Satisfied |
| S3-WIN-02 | descriptor與requested window partial overlap可用；no overlap拒絕且只取interval內records | DatasetCoordinator／runner | dataset tests＋flat／HFL evidence | Satisfied |
| S3-TAI-01 | private collection typed network area進Nsmf gating與descriptor provenance | config／collection manager／relay | config＋payload＋accepted resource tests | Satisfied |
| S3-TAI-02 | static flat保留topology／NRF／Training TAI，且local collection TAI一致 | flat owners／runner | discovery＋Subscribe＋preflight＋E2E | Satisfied |
| S3-TAI-03 | HFL topology／Training無TAI；Leaf local area只作ingress gating | HFL／dataset owners | matching tests＋preflight＋E2E | Satisfied |
| S3-TAI-04 | 同一group的每個SUPI依static current TAI恰好只屬於一個data owner | support SMF／runner | partition checks＋scenario summaries | Satisfied |
| S3-DATA-01 | deterministic dataset evidence | dataset builder／artifact | canonicalization tests | Satisfied |
| S3-DATA-02 | training artifact必帶evidence；Branch aggregate禁止local evidence | producer／validator | artifact tests | Satisfied |
| S3-FLAT-01 | 四-Client private collection→FedAvg E2E | production processes＋support peers | flat scenario summary | Satisfied |
| S3-FLAT-02 | flat one-tier aggregate可逐輪獨立重算 | runner | bundle recomputation | Satisfied |
| S3-HFL-01 | 四-Leaf private collection→existing FedProx E2E | production processes＋support peers | HFL scenario summary | Satisfied |
| S3-HFL-02 | Branch／Root aggregates與effective counts可逐輪獨立重算 | runner | bundle recomputation | Satisfied |
| S3-LIF-03 | failure／timeout／generation／cleanup不regression | PyMTLF lifecycle owners | focused tests | Satisfied |
| S3-REG-01 | production flat isolated flow preserved | existing flat owners | distributed FL runner | Satisfied |
| S3-REG-02 | existing HFL FedProx flow preserved | existing HFL owners | aggregation runner | Satisfied |
| S3-EVD-01 | summaries區分production、support與not-claimed boundaries | resources runner | schema checks | Satisfied |
| S3-VER-01 | §13 required commands完成 | affected repositories | exact results | Satisfied |
| S3-REV-01 | mandatory review、fresh-read conformance與language pass完成 | all diffs／docs | review handoff | Satisfied |

Flat 與 HFL 的 cross-topology model equality 明確沒有 conformance item。Real SMF／Nupf／UPF、UE session、
cross-VM與testbed network evidence是open integration verification gap，不在local map標為Satisfied。

## 15. Review 檢查清單

### 15.1 架構與 contract

- [x] Flat維持FedAvg；HFL維持FedProx，沒有新增algorithm variant或strategy transport。
- [x] Collection由caller明確create／delete；沒有minimum observation setting、`READY`、auto-stop或auto-training。
- [x] DELETE cleanup後，有durable stored data進`RETAINED`，無data進`TERMINATED`；failure cleanup不被掩蓋。
- [x] 不新增historical lag或sample-threshold config；現行current-time window與`minNumSamples: 1`保留。
- [x] DatasetCoordinator只放寬descriptor coverage為partial overlap；仍只取requested interval內records且不
  re-anchor。
- [x] Static flat topology／NRF／Training TAI保留，local collection TAI一致；production monitor flow不變。
- [x] HFL topology／assignment／Training不帶TAI；Leaf local TAI只進Nsmf collection gating。
- [x] 所有data owners解析同一group；每個SUPI依static current TAI恰好只在一個owner形成fixture／stored data。
- [x] 沒有新增public SBI、request override、Root fan-out或test-only introspection endpoint。
- [x] Dataset evidence是production artifact provenance，不含raw records或transport correlations。

### 15.2 端到端資料流

- [x] 每條scenario各自完成fixture→callback→storage→descriptor→snapshot→training→publication→cleanup。
- [x] Descriptor與actual requested window有交集，window外records不進dataset，interval內samples足夠。
- [x] DELETE後retained descriptor直接支援training，late callback被拒絕。
- [x] 每個local artifact的dataset evidence與自身counts一致。
- [x] HFL Branch沒有local data或sample contribution。

### 15.3 聚合與 evidence

- [x] Flat sample counts來自actual artifacts且不全相等，one-tier aggregate逐輪獨立重算。
- [x] HFL Leaf counts來自actual artifacts且不全相等，Branch／Root aggregates逐輪獨立重算。
- [x] Tolerance只用於同topology production result與runner重算結果。
- [x] 沒有把flat／HFL dataset、weights、metrics或final model equality列為通過條件。
- [x] Partial summary與failure diagnostics在失敗時仍保存。

### 15.4 Regression 與外部邊界

- [x] Production flat isolated與existing HFL FedProx regressions通過。
- [x] 既有failure／timeout／generation／cleanup direct tests通過。
- [x] Support UDM／SMF／callback replay沒有被描述為real SMF／UPF integration。
- [x] Summary保存support SMF的SUPI ownership partition，並將real SMF／Nupf gating保留為testbed gap。
- [x] Testbed validation仍維持open。

## 16. 明確延後項目

- flat FedProx與其Server→Client strategy／`proximal_mu` transport：`future-phase handoff`；
- algorithm×topology或cross-topology numerical comparison：`future-phase handoff`；
- real SMF→Nupf→UPF、UE traffic、cross-VM與testbed experiment：`integration verification gap`；
- production Root／Server collection fan-out：`future-phase handoff`；
- dynamic topology、participant replacement、arbitrary-depth hierarchy：`future-phase handoff`；
- per-tier或per-Branch algorithm override：`future-phase handoff`；
- HFL TAI／network-area participant matching：parent non-goal；Leaf local collection area不屬於participant
  matching；
- UE mobility後的TAI re-evaluation、peer subscription migration與data reassignment：`future-phase handoff`；
- throughput、convergence speed、accuracy superiority或statistical benchmark：另立experiment plan；
- mixed hardware／GPU determinism、mixed-version artifact interoperability與hot reload：future phase。

Local runner完成不關閉real-environment gap；超出本計畫的transport security項目也不會被誤列為未完成或
testbed blocker。

## 17. 完成與交付閘門

### 17.1 可進入實作的閘門

只有在使用者確認本計畫後才開始 production implementation。開始時：

1. 重新讀取 workspace instructions、development policy 與本文件；
2. 確認三個預計修改 repositories clean 或辨識 unrelated user changes；
3. 建立 §14 working conformance map；
4. 重新確認 artifact producers／consumers 與 runner helper ownership；
5. 保持 changes 分 repository，不跨 repo commit。

### 17.2 可交付審查的閘門

Implementation 與 focused verification 後立即執行 mandatory initial review。所有 in-scope findings 依
policy 完成 test-first remediation 與 targeted follow-up review 後：

1. 從 disk 重新完整讀取 development policy 與本文件；
2. 逐項重建 §14 final conformance；
3. 執行 §13 全部適用 commands；
4. 更新 actual commands、results、summary paths、revisions、support boundaries 與 open gaps；
5. 完整重讀 changed docs 並執行繁體中文 language consistency pass；
6. 狀態改為 `Ready for User Review`，保持所有 intended changes unstaged／uncommitted 供 IDE 檢視。

### 17.3 Commit 與 testbed 閘門

User review確認不代表commit approval。Review確認後另提出`PyMTLF`、`nwdaf-resources`、`nwdaf-docs`
repository-separated commit proposal，取得明確核准後才stage與commit；push另需獨立授權。

Local commits只固定可部署revisions。只有後續將這些exact revisions部署到testbed，完成事前另行確認的
real SMF／UPF／UE／cross-VM scenario matrix、保存record並由使用者確認evidence後，父計畫才能由
`Testbed Validation Pending`進入最終完成狀態。

## 18. 實作紀錄

### 18.1 Baseline 與修改範圍

2026-08-26 implementation 使用下列未提交 baseline：

- `PyMTLF/`：`aa65204387b38162d87b1c478d64c724548a10c3`；
- `nwdaf-resources/`：`0d839d167dedb6235b8bfbe97e346a95dc53580d`；
- `nwdaf-docs/`：`93a5a5c38f40bab89315a35242c81ade4479baee`；
- `NWDAF/`：`6aed268d6528f8be6c729cbd45b59d067e5e80dc`，無production diff。

`PyMTLF/`完成explicit collection lifecycle、typed `networkArea`、descriptor partial-overlap admission、
deterministic dataset evidence與artifact validation。Collection ledger升為schema version 2，並保存實際收到
callback的resource correlations；DELETE後只保留確實寫入資料之resources的descriptor，避免共享ADRF中
out-of-area subscription使其他data owner誤取資料。

`nwdaf-resources/`更新private collection runner，並加入四-Client Flat FedAvg與Root／兩Branch／四-Leaf
Hierarchical FedProx兩條獨立static-collection E2E。Support UDM／SMF提供同一group與TAI ownership gating，
callback replay只向matching owner注入資料；summary保存fixtures、requested windows、dataset evidence、
aggregation重算、publication、cleanup以及production／support／not-claimed boundaries。

### 18.2 Review 與 remediation

Mandatory initial review與後續targeted review關閉下列in-scope findings：

1. 共享ADRF原先會讓每個Leaf保留所有group members的descriptor。先以
   `test_delete_retains_descriptors_only_for_resources_that_stored_data`重現，再以observed correlation
   ownership修正；四個Leaf最終trainable sample counts分別為4、9、14、18。
2. Static collection scenario不啟動PyAnLF，因此不再等待model provision cutover；summary以`not_run`與
   原因明確記錄，而不虛構該verification claim。
3. 新runner entrypoint的module名稱與pytest collection衝突，改為唯一dynamic module名稱並以完整resources
   checks驗證。
4. 新scenario的unequal sample-count assertion一度誤套到既有`degradation-success`。先確認real-process
   regression失敗並加入deterministic unit test，再將條件限於`static_collection`；focused test、ruff與原
   regression重跑均通過。

未確認但不阻擋本 Slice 的風險為舊schema version 1 collection ledger的跨版本migration；目前沒有已部署
v1 ledger必須由此未提交版本原地升級的acceptance requirement，分類為`optional hardening`。Real SMF／
Nupf／UPF、UE session、cross-VM與testbed execution仍為`integration verification gap`。

### 18.3 Final verification

Final working tree的exact結果如下：

- 計畫指定的PyMTLF focused matrix：422 passed、2 skipped、37 warnings；
- `PyMTLF/.venv/bin/pytest -q`：597 passed、2 skipped、46 warnings；
- `PyMTLF/.venv/bin/ruff check src tests`：passed；
- resources checks：53 passed；
- resources ruff：passed；
- hierarchical preflight：passed；
- Flat static-collection FedAvg：passed，summary位於
  `/tmp/nwdaf-hierarchical-fl-flat-7l0cn4z2/summary.json`；
- Hierarchical static-collection FedProx：passed，summary位於
  `/tmp/nwdaf-hierarchical-fl-aggregation-lh5t43e7/summary.json`；
- production Flat isolated regression：passed，temporary root為
  `/tmp/nwdaf-distributed-fl-vxbbhe1m`；
- existing HFL FedProx `degradation-success` regression：passed，summary位於
  `/tmp/nwdaf-hierarchical-fl-aggregation-82xspryq/summary.json`。

兩條static scenarios均保存四個SUPI各自唯一owner、16次SMF subscription create／delete、DELETE後
`RETAINED`、late callback `404`、requested-window overlap、不同sample counts、兩個configured rounds、
topology內aggregation重算、ADRF publication與bounded cleanup。Flat與Hierarchical不作cross-topology
model equality claim；support callback replay也不作real SMF／UPF integration claim。

### 18.4 Conformance 與交付狀態

§14所有本地normative items均為`Satisfied`；mandatory initial review、test-first remediation、fresh-read
conformance與final full verification均完成。使用者已確認review結果；文件狀態為
`Commit Pending Approval`，三個repositories的intended changes維持unstaged／uncommitted，等待使用者核准
repository-separated commit proposal。Testbed validation仍未執行，因此父計畫保持
`Testbed Validation Pending`。
