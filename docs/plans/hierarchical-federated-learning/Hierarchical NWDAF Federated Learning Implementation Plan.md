# Hierarchical NWDAF Federated Learning Implementation Plan

日期：2026-08-17

最後更新：2026-08-18

狀態：Draft；canonical 主計畫，後續依團隊討論持續調整

相關文件：

- [Hierarchical NWDAF Federated Learning Proposal Draft](../../proposals/nwdaf/hierarchical-federated-learning/proposal_draft.md)
- [Phase 0 Release 18 Contract Foundation Detailed Plan](../federated-learning/Phase%200%20Release%2018%20Contract%20Foundation%20Detailed%20Plan.md)
- [Distributed NWDAF Federated Learning Implementation Plan](../federated-learning/Distributed%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)
- [Phase 3 And 4 Federated Training Execution Detailed Plan](../federated-learning/Phase%203%20And%204%20Federated%20Training%20Execution%20Detailed%20Plan.md)
- [NWDAF Development Policy](../../development_policy.md)

---

## 1. 計畫目的

本文件是 Hierarchical NWDAF Federated Learning 的 canonical 主計畫。

計畫目標是在既有 distributed NWDAF FL 實作上，加入可由同一種 NWDAF
implementation 依單次 topology assignment 動態扮演 Root、Branch 或 Leaf 的三層
hierarchical FL。此功能不新增 NF type，也不把 Root、Branch、Leaf 固定為部署身分。

本計畫先固定已確認的架構與 contract 決策，再將實作拆成可獨立驗收的 vertical
slices。尚未討論完成的 package、schema 與 private API 細節不得因本文件存在而被視為
已核准；它們必須在對應 phase 的 detailed plan 或本主計畫後續 revision 中定案。

---

## 2. 與既有 Federated Learning 的關係

既有 distributed FL 已具備以下 baseline：

- Go NWDAF 對外提供 `Nnwdaf_MLModelTraining` subscription resource 與 callback
  routing；
- Go NWDAF 可透過 NRF discovery 解析 peer NWDAF，並建立、更新及刪除 remote ML
  Model Training resource；
- PyMTLF 已具備 FL Client preparation／local round execution，以及 FL Server
  participant selection／round orchestration／FedAvg；
- model bundle、temporary artifact、round correlation 與跨 process E2E 已有可延伸的
  implementation baseline。

Hierarchical FL 不重做上述基礎，而是在其上增加：

1. Root 載入 static topology template，並透過 NRF 驗證其中指定的 NWDAFs；
2. Branch 同時承擔 upper-tier FL Client 與 lower-tier FL Server 的 process
   responsibilities；
3. topology assignment 與 subordinate preparation result 的 model bundle contract；
4. upper／lower process mapping；
5. Branch 先完成 lower-tier aggregation，再以上層 interim local model 回報 Root；
6. Root 對整棵 topology 的 admission、重規劃與 lifecycle ownership。

本計畫是新的 hierarchical FL workstream，不回寫或重新開啟已完成的 distributed FL
phases。若實作發現既有 baseline defect，依 development policy 判斷為目前 slice 的直接
blocker、future-phase handoff、legacy cleanup、optional hardening、integration
verification gap 或 unconfirmed risk。

既有 preparation notification contract 是本次 workstream 的直接 blocker，因此是上述
原則的明確例外修正範圍。早期 Phase 0 contract foundation 已正確記錄：依 TS 29.520
§5.5.6.2.8 NOTE 1，notification 至少須包含 `delayEventNotif`、`mLModelInfos` 或
`termTrainReq` 其中之一，`statusReport` 只能作為補充資訊，不能獨自滿足該條件。後續
distributed FL 主計畫與 Phase 3／4 execution plan 卻把 `statusReport` 誤列為可獨立成立的
結果，並據此要求 preparation 接受 `statusReport`-only callback；現有 Go／PyMTLF
validator、PyMTLF preparation sender 與 FL Server stage handling 也實作了這個後期決策。

本計畫明確 supersede 該項後期決策。這次 HFL 實作必須同步修正既有 contract regression，
不能只在新的 hierarchy orchestrator 外圍繞過：

- wire validator 不再把 `statusReport` 單獨存在視為滿足 notification 最低結果集合；
- preparation success sender 必須提供符合本計畫 profile 的 `mLModelInfos`；
- preparation-stage receiver 必須能依 active stage 解讀 `mLModelInfos`，不得再要求
  `statusReport` 才能判定成功；
- `statusReport` 若存在，只提供附加的標準 training status，不作為 preparation completed
  latch；不得再以固定 `trainInDataInfo.samplRatio: 100` 代替完成結果；
- non-hierarchical distributed FL regression tests 必須一併更新，以證明修正沒有破壞既有
  subscription、round 與 final-result behavior。

---

## 3. 已確認的架構基線

### 3.1 Topology profile

第一版支援可擴充的三層 topology：

```text
Root NWDAF
└─ FL Server
   ├─ Branch NWDAF
   │  ├─ upper tier: FL Client
   │  └─ lower tier: FL Server
   │     ├─ Leaf NWDAF
   │     └─ ...
   └─ ...
```

固定原則：

- 每棵 topology 只有一個 Root；
- Branch 與 Leaf 數量不固定；
- 所有節點都是標準 NWDAF，不註冊新的 NF type 或 hierarchy role；
- Root 是主動建立 plan 的 NWDAF；Branch／Leaf 是 Root 在該 plan 中指派的相對位置；
- 第一版不宣稱支援任意深度 recursive hierarchy；
- 每一層仍是獨立的標準 FL process，不建立跨整棵樹的單一 subscription。

標準 FL capability 決定 NWDAF 是否有資格被指派，Root assignment 才決定其在特定
`planId` 中的實際角色：

| Topology position | Minimum registered capability |
| --- | --- |
| Root | `FL_SERVER` 或 `FL_SERVER_AND_CLIENT` |
| Branch | `FL_SERVER_AND_CLIENT` |
| Leaf | `FL_CLIENT` 或 `FL_SERVER_AND_CLIENT` |

`FL_SERVER_AND_CLIENT` NWDAF 可以在一個 plan 中被指派為 Branch，也可以在另一個 plan
中只被指派為 Leaf。Capability 不等於 role，NF profile 不增加 `ROOT`、`BRANCH` 或
`LEAF` 欄位：

```text
standard registered capability
+ Root Branch/Leaf assignment
+ planId / process context
= topology role
```

Root 是主動建立該 topology plan 的 FL Server。Branch 是收到指向自己的 hierarchy
assignment，並為 assigned Leaves 建立 lower-tier process 的 NWDAF。Leaf 則只在該
process 中執行 FL Client behavior。Role 在 plan 結束時隨 process state 清除，不改變
NRF 中註冊的標準 capability。

### 3.2 Process boundary

Upper tier 與 lower tier 分別維護：

- `mlCorreId`；
- subscription resources；
- notification correlation；
- participant state；
- round state 與 deadline；
- artifact／bundle ownership；
- cleanup 與 terminal state。

Branch 必須維護明確的 upper process 與 lower process mapping。不得僅以相同
`roundInd`、subscription ID 或 callback URI 推測兩層關係。

### 3.3 Topology ownership

Root 擁有整棵 topology 的主要管理權：

- 載入 static topology template，並驗證 configured NWDAF identities；
- 指派 Branch 與 Leaf candidates；
- 決定 admission policy；
- 接收 subordinate preparation results；
- 接受或拒絕 topology；拒絕後只在 operator 明確重新啟動實驗時建立全新 attempt；
- 控制整體 training 的開始、推進與終止。

Branch 執行 Root 委派的 subtree 工作。第一版不得自行補選、重新分組或替換 Leaf。
Leaf 只需要理解自己所屬 lower-tier process，不需要取得完整 topology。

---

## 4. 已確認的 implementation-profile 決策

### 4.1 Model bundle 承載 hierarchy metadata

`mLModelUrl` 指向的 model file 採同 vendor model bundle。Bundle 必須包含有效且可訓練
的 model artifact；hierarchy manifest 是附加 metadata，不取代模型，也不改變 SBI
欄位型別。

標準 SBI 繼續表達：

- Analytics ID；
- training requirements；
- model interoperability；
- `mlCorreId`；
- `mLPreFlag`；
- notification URI 與 correlation。

Bundle 不應重複或覆寫上述標準欄位的 normative semantics。

### 4.2 Preparation result 使用 result bundle

團隊已接受以下 implementation profile：

1. Branch 在 upper-tier preparation outcome notification 中提供
   `mLModelInfos.mLFileAddr.mLModelUrl`；
2. URL 指向包含有效 model artifact 與 subordinate preparation result manifest 的
   result bundle；
3. result manifest 至少能表達 plan identity、prepared、failed 與 timed-out Leaf
   participants，以及可判斷 admission 的結果；
4. Branch 不宣稱 preparation 產生了新的 trained model；
5. Root 下載並驗證 result bundle 後，依實際 subordinate result 接受或拒絕整個
   attempt。

這不是 `mLModelUrl` 最自然的 procedure semantics，但在不新增 private external SBI、
不修改 Release 18 OpenAPI schema 的前提下，是組合現有
`Nnwdaf_MLModelTraining` mechanism 實現 hierarchical FL 最直接的方式。

此行為必須明確標示為同 vendor implementation contract，不得宣稱為 3GPP 已定義的
hierarchical FL procedure。後續實作與 review 不再把是否採用 result bundle 當作 open
decision；只有具體 schema、完整性、lifecycle 與 error behavior 仍待定案。

#### Preparation success notification contract

第一版固定以下成功通知：

- Leaf preparation success 提供 `mLModelInfos`，URL 指回該 Leaf 已下載並驗證的
  Branch-published assignment bundle；
- Branch preparation success 提供 `mLModelInfos`，URL 指向 Branch-published
  preparation-result bundle；
- 兩者都可另外包含 `statusReport`，但 receiver 不得以其存在與否決定 preparation 是否
  完成；
- preparation notification 不帶 standard `roundInd`，接收端以 active process stage、
  notification correlation 與 artifact contract 判斷其用途；
- 在尚未形成有效 result artifact 前失敗，使用 `termTrainReq`；若 Branch 已形成完整
  failure result bundle，可同時提供 `mLModelInfos` 與 `termTrainReq`，供 Root 先記錄結果
  partition，再將 attempt 標記為 terminal failure。

Leaf 回報輸入 assignment bundle 並不是最自然的 preparation 語意，但這是團隊已接受的
Release 18 implementation profile：它保留有效 model artifact、符合 notification 的最低
結果條件，也不虛構 preparation 已產生新 trained model。這項 semantic tension 不得再以
`statusReport`-only callback 規避。

### 4.3 Bundle publication and Branch republishing

Hierarchy 使用既有 PyMTLF temporary-artifact serving boundary：

- model bundle、assignment bundle、result bundle 與 round artifact 的 bytes 由發布該
  artifact 的 PyMTLF 直接提供下載；
- Go NWDAF 只透過標準 SBI 傳遞 `mLModelUrl` 與相關 model information，不 relay 或
  proxy artifact bytes；
- URL、digest、retention、expiry 與 cleanup 由發布 artifact 的 PyMTLF 擁有。

Branch 不能把 Root URL 原樣轉交 Leaves。它必須：

1. 從 Root PyMTLF 下載並驗證 upper-tier bundle；
2. 解析上層 assignment 與 model artifact；
3. 依 assigned subtree 建立只包含 lower-tier 所需內容的新 Leaf assignment bundle；
4. 由 Branch PyMTLF 發布新 bundle，並把 Branch-owned URL 傳給 Leaves。

因此這是必要的 download／process／republish flow，不是 transparent proxy，也不是第一版
可選 strategy。Root 只直接服務 Branch，Leaf 從所屬 Branch 下載。Branch 建立的
preparation result 與 lower-tier round artifacts 同樣由 Branch PyMTLF 發布。

#### First-version artifact trust boundary

第一版部署假設 Root、Branch、Leaf NWDAF 及各自的 PyMTLF 都屬於本專案控制的同一套
實驗系統與同一 vendor trust domain。Hierarchy manifest 的
`publisher_nf_instance_id` 是 process／topology 使用的 logical identity，不是經由
signature、OAuth requester identity 或 mTLS client identity 所形成的 cryptographic
attestation。

第一版不新增 `expected_artifact_origin`、requester identity private header，也不為
hierarchy artifact 額外修改 OAuth／mTLS identity propagation。下載端仍執行 configured
allowed-origin、URL／header／archive digest、publisher、recipient、message type 與
`planId` 等既有一致性驗證，但不得將這些檢查描述為已證明實際 HTTP caller 的 NF
instance identity。

跨 vendor、跨管理域或不可信網路環境所需的 requester NF identity 與 artifact origin
強綁定，列為未來 security hardening，不是第一版 decision gate，也不阻擋後續 Slice。

### 4.4 Asynchronous preparation

第一版沿用既有 implementation profile：

```text
201 Created
= subscription resource 已建立，preparation 非同步開始

prepared notification
= 本層 preparation 完成

termination notification
= 本層無法完成或維持此次 preparation
```

Branch 接受 upper-tier preparation request 後不得阻塞原始 HTTP request 等待全部
Leaves。Lower-tier outcomes 由 Branch 的 background orchestration 收集，再形成
upper-tier notification。

### 4.5 Admission policy

第一版只實作 `COMPLETE_REQUIRED`：任一 assigned Branch 或必要 Leaf preparation 失敗或
逾時，都不接受目前 topology attempt。`admission.mode` 欄位保留為未來擴充點，但第一版
唯一合法值是 `complete_required`；`PARTIAL_ALLOWED` 與 `minimumClients` 不實作。

Branch 必須回報實際 prepared、failed 與 timed-out lists；最終 topology admission 只由
Root 決定。

### 4.6 Preparation failure 的第一版 Root handling

Branch 的 result bundle 必須讓 Root 區分：

- `preparedClients`；
- `failedClients` 與各自的 failure cause；
- `timedOutClients`。

Root 先驗證每份 bundle 的 version／integrity、`planId`、Branch identity 與目前
attempt，並等待所有 assigned Branch outcomes 或 preparation deadline。Root 將各 Branch
results 組成實際 subordinate topology 後，對整個 attempt 做下列決定：

| Policy | Accept condition | Otherwise |
| --- | --- | --- |
| `COMPLETE_REQUIRED` | 所有 assigned Clients 都在 `preparedClients` | reject 整個 attempt |

若 Root 接受：

1. 將各 Branch 的 `preparedClients` 凍結為本次實際 participant snapshot；
2. 更新各 accepted Branch 的既有 upper-tier subscription 並送出第一輪 training，作為
   topology 已被接受的 execution transition。

若 Root 不接受：

1. 刪除該 attempt 建立的所有 upper-tier subscriptions；
2. 各 Branch 清理對應 lower-tier subscriptions、process mapping 與 temporary bundles；
3. 不安排自動重試；operator 修復問題並明確重新啟動後，Root 才建立新的 `planId` 與
   preparation attempt。

第一版不跨 attempt 保留先前成功的 Leaves、不做增量補選或重新分組，也不允許 Branch
自行替換失敗 Leaf。這些能力可在未來加入；第一版失敗後停在 terminal state，完成可行的
cleanup。後續 operator-initiated experiment 使用全新 attempt，避免 stale participant
snapshot、subscription 與 callback correlation。

### 4.7 First-version strategy profile

Daisy 只提供 selection、waiting、aggregation 與固定 orchestration flow 可分離的設計
參考。本計畫不搬入 Daisy dynamic module loading、gRPC `ClientProxy`、完整 strategy
hierarchy 或所有既有 algorithms。

第一版只實作完整主情境，同時保留穩定 config 欄位供未來擴充：

```yaml
federated_learning:
  strategy:
    algorithm:
      name: fedprox
      proximal_mu: 0.01
    participant_selection: all
    waiting_policy: all
    aggregation: sample_weighted
```

#### Algorithm

- `algorithm.name: fedprox`：第一版唯一合法 algorithm。Leaf local training objective 在
  task loss 外加入 proximal term，限制 local weights 不要偏離本輪 global weights；
  `proximal_mu` 必須為有效的 finite positive value。

FedProx 的主要差異在 Client local objective，並不表示 Server 端改用另一種 aggregation
公式。FedAvg 可在未來視為 `proximal_mu: 0` 的 baseline，但不列入第一版合法選項。

#### Participant selection

- `all`：第一版唯一合法值；本輪選擇所有 prepared participants。

#### Waiting policy

- `all`：第一版唯一合法值；必須收到所有 selected participants 的成功結果才可
  aggregation。任一必要 participant 失敗或 deadline 到期時，本輪失敗。

#### Aggregation

- `sample_weighted`：依各 participant 的有效 training sample count 加權平均模型。Branch
  回報 Root 時，其 effective sample count 是該 Branch 實際納入 aggregation 的 Leaves
  sample count 總和。

`proximal_mu` 是 FedProx-specific parameter，因此放在 typed `algorithm` block 內，不與
generic strategy fields 並列，也不放進 Leaf-local `federated_learning.client.training`
settings。第一版不使用無型別的 generic `parameters` dictionary；implementation 應以
discriminated／typed algorithm settings 驗證 algorithm-specific fields。

Strategy config fields 都保留為未來擴充點，但第一版各只有一個合法值。Config 使用明確
enum／built-in registry；不得允許任意 Python module 或 class name。Top-level
`proximal_mu`、`fedavg`、`fixed_count`、`minimum_results`、其他 aggregation，以及任何
未知或不相容的值都必須在 validation 階段被拒絕。

Root 將已解析的 strategy contract 放入 model bundle，Branch／Leaf 驗證後執行，避免各
節點只依本地 config 推測本次 FL semantics。第一版先使用同一份 strategy contract 貫穿
topology；per-tier override 保留為未來擴充，不在第一版實作。

### 4.8 Root-only static topology configuration

第一版 topology establishment strategy 固定為 `static`。只有主動建立 topology 的 Root
PyMTLF 在主 config 引用獨立 topology file；Branch 與 Leaf 不保存完整 topology，也不
配置 hierarchy role。

Root main config：

```yaml
federated_learning:
  topology:
    strategy: static
    config_file: "./topology/hierarchical-topology.yaml"
  training_trigger:
    private_api:
      enabled: true
```

Referenced topology file：

```yaml
version: 1

admission:
  mode: complete_required

branches:
  - nf_instance_id: "22222222-2222-4222-8222-222222222222"
    leaves:
      - nf_instance_id: "33333333-3333-4333-8333-333333333333"
      - nf_instance_id: "44444444-4444-4444-8444-444444444444"
  - nf_instance_id: "55555555-5555-4555-8555-555555555555"
    leaves:
      - nf_instance_id: "66666666-6666-4666-8666-666666666666"
      - nf_instance_id: "77777777-7777-4777-8777-777777777777"
```

Config rules：

- `config_file` 相對路徑以主 config 所在目錄解析，不依賴 process working directory；
- `training_trigger.private_api.enabled` 預設為 `false`；只有要接受管理操作觸發的 Root
  PyMTLF 才明確設為 `true`；
- private API 啟用時必須同時具備 FL Server capability 與有效的 Root topology config，
  否則 startup validation failure；Branch／Leaf 不因共用 PyMTLF implementation 而自動
  暴露此 endpoint；
- topology file 是 assignment template，不保存 endpoint、`planId`、subscription ID、
  callback URI 或 `mlCorreId`；
- Root 每次 preparation attempt 從已驗證的 config snapshot 產生新的 `planId`；
- Root 依 topology file 中的 `nf_instance_id` 透過 NRF 做 exact-instance resolution，並
  驗證 Branch／Leaf 的 registered FL capability 與 service endpoint；
- Branch 收到 assignment bundle 後，仍須再次透過 NRF 驗證 assigned Leaves；
- file version、空 Branch、重複 identity、Branch／Leaf identity 衝突、admission fields
  與 capability mismatch 都必須 deterministic validation failure；
- 第一版不支援 hot reload；修改 topology file 後重新啟動 Root；
- `strategy: static` 的 Root config 缺少、無法讀取或無法驗證指定 topology file 時，
  Root PyMTLF startup failure。

Root-only topology config 只允許該 deployment 主動建立 static plan，不代表它向 NRF
註冊為特殊 Root。未來可新增 `strategy: dynamic`，以 NRF candidate inventory 自動建立
assignment，但不改變後續 bundle、preparation 與 training contracts。

### 4.9 Fresh-state restart semantics

第一版不實作 active hierarchy state persistence、restart recovery 或 reconciliation。
Root、Branch、Leaf 的 Go 或 PyMTLF process 重新啟動後，都從全新 runtime state 開始，
不恢復舊 topology、subscription mapping、prepared participant snapshot 或 round state。

Restart 前的 callback、bundle 或 round result 若之後送達，必須因找不到 active process、
correlation 不相符或 artifact 已過期而被拒絕；不得隱式重建舊 process。正常流程仍須實作
terminal cleanup、artifact retention／expiry 與 garbage collection，但不為 crash 後的舊
remote resources 增加 recovery protocol。

### 4.10 Training initiation boundary

Hierarchical training 有兩個第一版入口：

1. PyMTLF 偵測到 model degradation 後，由 training producer 主動建立新 training plan；
2. operator／management function 透過 private internal management API 提交 training
   request，由 Root PyMTLF 非同步接受或拒絕並建立新 plan。

Private management API 是 config-gated PyMTLF endpoint，預設不掛載。只有
`training_trigger.private_api.enabled: true` 的 PyMTLF instance 才註冊 route；request
直接由 Root PyMTLF 處理，不經過 Go NWDAF，也不屬於 NWDAF public SBI。第一版使用簡單
private asynchronous request resource：建立成功回傳 `202 Accepted`、`requestId` 與
`planId`，並提供相對應的 status query。Exact path、payload、authentication 與 error body
在 detailed plan 依既有 PyMTLF private API conventions 定案。

這兩個入口都發生在 FL process 建立之前。`Nnwdaf_MLModelTraining` 用於已進入 training
procedure 後的 NWDAF 間 preparation、round control 與結果通知，不得把它誤作 OAM 或
degradation trigger API。

Release 19 TS 28.105 的 `MLTrainingRequest` 與 generic Provisioning MnS REST mapping 可作為
management-plane semantics 參考；其中 consumer 建立 request，producer 接受後自行決定
實際開始時間，而且 producer-initiated training 不要求存在 request。然而本專案第一版
仍以 Release 18 為標準基線，因此 private API 不宣稱符合 TS 28.105，也不在本 slice 實作
完整 TS 28.532 generic Provisioning MnS。

### 4.11 Single-active-training and failure termination

第一版沿用 PyMTLF 既有「同時間只能有一個 training」限制；此限制作用於整個 PyMTLF
instance，而不是每個 model 各自允許一個 active plan：

現有 `FLServerSettings.max_active_processes` 雖預設為 `1`，但 schema 允許設定更大的值。
Hierarchy 不能只依賴該預設；HFL-enabled Root 必須拒絕大於 `1` 的設定，runtime 也必須
建立跨 FL Server／Client engines 的 instance-level active-experiment guard。Branch 的
upper Client process 與 lower Server process 是同一 experiment 的配對 resources，不算成
兩個可平行 experiments。

- degradation event 或 private request 在已有 active training 時，不得建立第二個 plan；
- private API 對不同的新 request 回傳 conflict；相同 idempotency identity 的重送回傳
  已建立的 request／plan identity；
- degradation detector 不平行啟動另一個 experiment。

任一必要 participant 在 preparation 或 training round 失敗／逾時時，整個 experiment
進入 terminal failure。Root／Branch 執行可完成的 termination 與 cleanup，但不自動 retry
preparation、round 或 topology，也不因原 degradation condition 仍存在而自動建立新
`planId`。團隊先停止並檢查失敗原因；修復後由 operator 以明確操作重新啟動全新
experiment。

---

## 5. 標準邊界與證據政策

### 5.1 Evidence order

實作決策依下列順序核對：

1. `NWDAF/` 與 `PyMTLF/` 現有 implementation／tests；
2. 本主計畫、proposal 與已確認 decision；
3. Release 18 OpenAPI YAML；
4. TS 23.288 與 TS 29.520 procedure／data model；
5. specifically applicable free5GC exemplar；
6. local generated free5GC OpenAPI code與 reference implementation。

OpenAPI 決定 method、path、schema、required fields、status、header 與 declared errors；
TS text 決定 procedure intent 與 role boundary；free5GC exemplar 只決定 implementation
shape，不得覆寫 NWDAF contract。

核心規格證據：

- [TS 23.288 §6.2C Federated Learning among Multiple NWDAFs](../../../specs/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2C%20Federated%20Learning%20among%20Multiple%20NWDAFs.md)
- [TS 23.288 §6.2F Procedure for ML Model Training](../../../specs/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2F%20Procedure%20for%20ML%20Model%20Training.md)
- [TS 29.520 §4.6 Nnwdaf_MLModelTraining Service](../../../specs/TS%2029.520/4%20Services%20offered%20by%20the%20NWDAF/4.6%20Nnwdaf_MLModelTraining%20Service.md)
- [Release 18 Nnwdaf_MLModelTraining OpenAPI](../../../specs/openapi/TS29520_Nnwdaf_MLModelTraining.yaml)
- [Release 18 Nnwdaf_MLModelProvision OpenAPI](../../../specs/openapi/TS29520_Nnwdaf_MLModelProvision.yaml)
- [Release 18 Nnrf_NFDiscovery OpenAPI](../../../specs/openapi/TS29510_Nnrf_NFDiscovery.yaml)
- [Release 18 Nnrf_NFManagement OpenAPI](../../../specs/openapi/TS29510_Nnrf_NFManagement.yaml)

### 5.2 free5GC implementation-shape evidence

本工作流沿用既有 distributed FL plan 已選定並重新確認的 local exemplars：

- BSF `internal/sbi/api_management.go` 與
  `internal/sbi/processor/subscriptions.go`：subscription collection、individual
  resource、`Location` 與 CRUD shape；
- PCF `internal/sbi/api_httpcallback.go` 與
  `internal/sbi/consumer/notification.go`：callback handler 與 outbound notification
  separation；
- UDM `internal/sbi/consumer/nrf_service.go`：outbound consumer 與 NRF discovery client
  ownership。

Hierarchical topology、cross-process mapping 與 result bundle 沒有直接 free5GC
exemplar；其設計來自 3GPP processes 的同 vendor composition，屬本專案 proposed
design，而不是 upstream implementation behavior。

---

## 6. Repository 與 component ownership

### 6.1 `NWDAF/`

Go NWDAF 繼續作為唯一標準 NF boundary，預期負責：

- public `Nnwdaf_MLModelTraining` wire validation 與 resource lifecycle；
- callback URI internalization／externalization；
- NRF registration、capability advertisement 與 discovery gateway；
- peer NWDAF service endpoint resolution；
- standard subscription create／replace／patch／delete routing；
- notification relay、OAuth／transport 與 `ProblemDetails` mapping；
- containing PyMTLF private boundary 的最小 routing／correlation state。

Go 不應擁有 topology selection、admission policy、FL aggregation algorithm 或 bundle
manifest business semantics。Private training-trigger API 不由 Go expose、route 或 proxy；
request 直接進入 PyMTLF。

### 6.2 `PyMTLF/`

PyMTLF 預期擁有：

- Root-only static topology config loading／validation 與 topology planner；
- FL Server／Client engines，以及 capability-aligned process dispatch；
- Branch upper／lower process mapping；
- hierarchy assignment／result manifest parsing、validation 與 construction；
- subordinate preparation orchestration；
- topology admission policy；
- per-tier participant、deadline、round 與 terminal state；
- Leaf local training；
- Branch lower-tier aggregation；
- Root upper-tier aggregation；
- bundle／artifact publication、download validation、retention、expiry 與 cleanup；
- degradation-driven trigger 與 config-gated private internal training-request API；
- single-active-training guard 與 terminal-failure state。

PyMTLF 不是獨立標準 NF，不向 NRF 註冊，也不直接取代 Go 對外提供
`Nnwdaf_MLModelTraining`。

### 6.3 `nrf/`

既有 Release 18 NWDAF profile persistence 與 FL capability discovery 是本計畫
dependency。第一版先將 `nrf/` 視為 verify-only repository；只有 deterministic
evidence 證明目前 exact-instance resolution、capability matching 或 profile
persistence 無法支援 hierarchical flow，才建立獨立 NRF implementation slice。

### 6.4 `nwdaf-resources/`

若 hierarchical multi-process E2E 需要新的 config generator、fixture、bundle fixture
或 launcher，放在 `nwdaf-resources/` 的獨立 implementation slice。不得把 support
tooling 與 NWDAF／PyMTLF production changes 混在同一 commit。

### 6.5 `resources/`

`resources/` 只作 Daisy、free5GC generated code 與 exemplar reference，不在本計畫
修改。

### 6.6 Implementation branch workflow

本 canonical plan 可直接維護於 `nwdaf-docs/main`。開始 production implementation 前，
`NWDAF/`、`PyMTLF/`，以及確實需要修改時的 `PyAnLF/`，各自在自己的 repository 建立
`feat/r18-hierarchical-federated-learning`，沿用既有 `feat/r18-federated-learning`
workstream 的命名格式。新 branch 以包含既有 distributed FL baseline 的 branch／commit
為基底；若開始實作時該 baseline 尚未合併，便從 `feat/r18-federated-learning` 分出。
各 repository 必須獨立 commit、測試及 push，不跨 repository 混合提交；若 `PyAnLF/`
只需驗證既有行為，則維持 verify-only，不為了名稱一致而建立無實際修改的 branch。

---

## 7. Target control flow

### 7.1 Topology preparation

```text
Root PyMTLF
  -> degradation trigger or enabled PyMTLF-private management request
  -> load Root-only static topology config
  -> Go NRF discovery gateway
  -> exact-instance capability/endpoint verification
  -> Root PyMTLF publishes upper-tier assignment bundle
  -> Go peer ML Model Training consumer
  -> Branch Go public Training SBI
  -> Branch PyMTLF downloads and validates Root bundle
  -> Branch PyMTLF publishes lower-tier assignment bundle
  -> Branch PyMTLF background preparation
  -> Branch Go NRF discovery gateway
  -> Leaf Go public Training SBI
  -> Leaf PyMTLF preparation outcomes
  -> Branch PyMTLF publishes result bundle
  -> Root preparation notification
  -> Root accepts or rejects the complete attempt
```

Static topology file 決定 Root 想建立的 assignment，但不能證明目標 NWDAF 可參與。
Root 必須先透過 NRF 驗證 configured identities。Branch 從 assignment manifest 取得的
`nfInstanceId` 仍只是 candidate hint；Branch 必須經 NRF 重新解析並確認目前 NF
profile、FL capability 與 service endpoint，不能直接把 manifest 視為 admission proof。

### 7.2 Hierarchical training round

```text
Root updates upper-tier subscriptions
  -> Branch updates lower-tier subscriptions
  -> Leaves perform local training
  -> Leaves notify Branch with local model results
  -> Branch aggregates lower-tier results
  -> Branch notifies Root with aggregated interim local model
  -> Root aggregates Branch results
```

第一版使用 synchronous rounds，預設以 FedProx 執行 Leaf local training，再以
`sample_weighted` 聚合；participant selection 與 waiting policy 都固定為 `all`。
FedAvg、`fixed_count`、`minimum_results`、FedAsync、staleness weighting、per-tier
strategy override 或任意深度 recursive scheduling 不作為第一版 implementation 的
必要條件。

---

## 8. 初始 implementation slices

每個 slice 開始前都必須建立或更新 detailed plan，明列 affected repositories、contract、
acceptance tests、deferred behavior 與 checkpoint。

### Slice 0：Baseline audit and contract freeze

目標：確認既有 distributed FL implementation 可重用邊界，避免平行重建。

Detailed plan：

- [Slice 0 Baseline Audit and Contract Freeze Detailed Plan](./Slice%200%20Baseline%20Audit%20and%20Contract%20Freeze%20Detailed%20Plan.md)

預期產出：

- current Go Training route／callback／NRF discovery evidence map；
- current PyMTLF FL Client／Server state and persistence evidence map；
- existing bundle schema and artifact lifecycle gap list；
- hierarchy assignment manifest 與 result manifest 的 versioned schema；
- standard field 與 vendor metadata ownership table；
- first-version typed strategy config／bundle contract；
- static topology config contract；
- capability-oriented runtime 與 process-scoped role migration map；
- degradation trigger 與 config-gated PyMTLF-private management request contract；
- single-active-training、idempotent retry 與 terminal-failure contract。

### Slice 1：Hierarchy bundle contract and artifact primitives

目標：提供可驗證、可版本化且不覆寫標準欄位的 assignment／result bundles。

狀態：Implementation 與 code review 完成；PyMTLF checkpoint 已 commit。

Detailed plan：

- [Slice 1：Hierarchy Bundle Contract 與 Artifact Primitives 詳細計畫](./Slice%201%20Hierarchy%20Bundle%20Contract%20and%20Artifact%20Primitives%20Detailed%20Plan.md)

至少包含：

- bundle type／version；
- topology plan identity；
- intended recipient identity；
- assigned Leaf candidate identities；
- `complete_required` admission policy；
- normalized typed strategy／algorithm contract；
- prepared／failed／timed-out result；
- model artifact reference and digest；
- manifest integrity and compatibility validation；
- Root-to-Branch download 與 Branch-to-Leaf process／republish contract；
- PyMTLF-owned publish、download、retention 與 cleanup lifecycle；
- contract／artifact primitives 的 isolated round-trip tests。

### Slice 2：Capability and process-scoped role foundation

目標：讓同一 NWDAF implementation 可在不同 FL processes 中承擔不同角色，並使
Branch 安全地連結 upper／lower resources。

至少完成：

- deployment capability 與 per-topology role 分離；
- 同一 runtime 依 registered capability 啟用 FL Server／Client engines；
- Root assignment 驅動 Branch／Leaf process behavior，不增加 role config；
- upper process／lower process identity model；
- fresh-state restart、duplicate、stale callback 與 cleanup semantics；
- bounded concurrency and ownership；
- Go route state 與 PyMTLF business state 的責任切分；
- 現有純 FL Server 與純 FL Client profiles 的 regression checkpoint；
- Server + Client capabilities 同時啟用時的 route／engine availability tests；
- instance-level single-active-training guard foundation。現有
  `FLServerSettings.max_active_processes` 雖預設為 `1`，schema 允許更大值，因此不能只
  依賴預設；HFL-enabled Root 若沿用此欄位，必須 deterministic validation 其值為 `1`，
  並由 instance-level guard 作為最終限制。Branch 的 upper／lower resources 視為同一
  experiment 內的配對 processes。

### Slice 3：Root initiation, static topology and assignment

目標：完成 Root control plane：由 degradation 或已啟用的 private API 建立單一 active
plan，載入 static topology、透過 NRF 驗證 configured assignment，並送出 upper-tier
preparation。

至少完成：

- main-config-relative topology file loading；
- version、identity、tree shape 與 `complete_required` admission validation；
- exact-instance FL Server／Client capability query；
- configured Branch／Leaf eligibility validation；
- `StaticTopologyPlanner` interface／implementation；
- degradation-driven plan initiation；
- config-gated PyMTLF-private request／status API，不經過 Go；
- idempotent request replay 與 conflicting-request rejection；
- instance-level single-active-plan enforcement；
- assignment bundle publication；
- upper-tier preparation resource lifecycle；
- duplicate plan 與 stale discovery result handling。

### Slice 4：End-to-end preparation and admission

目標：完成 Root–Branch–Leaf 的 preparation walking skeleton；Branch 非同步完成
lower-tier preparation，以 result bundle 回報 Root，並由 Root 做整棵 topology admission。

至少完成：

- assignment bundle validation；
- Root-to-Branch bundle download；
- Branch process／republish and Leaf bundle download；
- assigned Leaf NRF re-resolution；
- lower-tier preparation fan-out；
- per-Leaf outcome／deadline tracking；
- `complete_required` evaluation input；
- result bundle construction and notification；
- Root accept／reject decision and rejected-attempt cleanup；
- Branch／Leaf failure 與 timeout tests；
- preparation failure 使整個 experiment terminal；
- no automatic preparation／topology retry。

### Slice 5：Hierarchical rounds and aggregation

目標：完成 Root–Branch–Leaf 的 multi-round hierarchical aggregation。

至少完成：

- upper／lower round dispatch；
- per-tier round correlation；
- Leaf local result validation；
- FedProx local objective、global-reference snapshot 與 `proximal_mu` validation；
- `all` participant selection 與 `all` waiting；
- Branch aggregation and upper-tier interim result；
- Root aggregation；
- duplicate、late、wrong-round 與 incompatible artifact rejection；
- round deadline、termination and cleanup；
- any required-participant failure terminates the complete experiment；
- no automatic round retry。

### Slice 6：Lifecycle closure and fresh-state restart behavior

目標：收斂前述 slices 已隨流程實作的 terminal／cleanup semantics，並驗證 process restart
後不恢復舊 state 的行為。基本 cleanup 不得延後到此 slice 才首次實作。

至少完成：

- Root、Branch、Leaf individual restart behavior；
- Go restart 與 PyMTLF restart boundary；
- bundle URL expiry／missing artifact behavior；
- restart 後不 resume 或 reconcile 舊 active process；
- stale callback／result rejection；
- idempotent cleanup；
- terminal process retention and garbage collection。

### Slice 7：Multi-process E2E and regression closure

目標：用兩個層級的 topology profiles 驗證完整 business flow：

- smoke profile：一個 Root、一個 Branch、兩個 Leaves；
- aggregation acceptance profile：一個 Root、至少兩個 Branches，且至少一個 Branch
  具有兩個 Leaves；推薦兩個 Branches 各有兩個 Leaves，以完整驗證兩層 weighting。

至少驗證：

- NRF capability discovery；
- Root-only static topology loading and assignment；
- capability／assignment mismatch rejection；
- complete preparation；
- rejected preparation cleanup and operator-initiated new experiment；
- multi-round lower／upper aggregation；
- 至少兩個 Branch results 的 Root upper-tier aggregation；
- 不同 subordinate sample counts 的 two-tier effective-weight propagation；
- FedProx、`all` selection／waiting 與 `sample_weighted` aggregation；
- Branch bundle process／republish 與 Leaf 從 Branch 下載；
- subordinate failure and timeout；
- failure 後 terminal stop，且不自動 retry；
- degradation trigger 與 config-enabled PyMTLF-private management request trigger；
- private API disabled 時 route 不存在，且 trigger 不經過 Go；
- 同一 PyMTLF 同時間只存在一個 active training；
- restart 後 fresh state 與 stale interaction rejection；
- graceful completion 後沒有 orphan subscriptions／artifacts；crash 前的 remote resource
  只依既有 expiry／garbage-collection policy 收斂；
- regression of existing non-hierarchical FL profile。

---

## 9. Architecture decisions

### Capability-oriented runtime and process-scoped role

現況：PyMTLF 的 `runtime.mode` 為 `local`、`fl_server` 或 `fl_client`，三者互斥；proposal
要求同一 Branch 在不同 process boundary 同時作為 FL Client 與 FL Server。

已確認方向：runtime 不新增永久 `root`、`branch`、`leaf` 或 `fl_branch` role。相同 NWDAF
implementation 依標準 registered FL capability 啟用對應 FL Server／Client engines；Root
assignment 與 `planId` process state 決定本次實際行為。Branch 所需的雙向 behavior 由同一
runtime 中可同時使用的 Server／Client capabilities 組成。

Slice 0 仍須提出既有互斥 `runtime.mode` 的 migration detail、Go advertised capability 與
PyMTLF enabled engines 的一致性 validation，以及 restart／lifecycle ownership；但不再重新
選擇 role-oriented runtime 方案。

### Bundle publication owner

PyMTLF 直接 serving 自己發布的 assignment／result／round artifacts；Go 只傳遞 URL，
不 proxy bytes。Branch 必須下載、驗證及處理 Root bundle，再由 Branch PyMTLF 發布新的
Leaf bundle。每個發布者擁有其 URL、digest、retention、expiry 與 cleanup。

### Fresh-state restart profile

第一版不持久化或恢復 active hierarchy process state。任何 Go／PyMTLF process restart
都建立全新 runtime state；舊 callback、result 與 artifact 不用來重建 process。

### First-version admission mode

第一版只支援 `COMPLETE_REQUIRED`。任一 assigned Branch／Leaf preparation 失敗或逾時即
reject 整個 attempt 並進入 terminal state。完成可行的 cleanup 後不自動重建；operator
確認並修復問題後，才以新 `planId` 啟動全新 experiment。`PARTIAL_ALLOWED`、補選、
重新分組與跨 attempt 保留成功 participants 全部延後。

### Config-gated PyMTLF-private management request API

已確認 private management API 與 degradation trigger 都位於 FL process 建立之前，且不使用
`Nnwdaf_MLModelTraining` 作為起始介面。API owner 固定為 Root PyMTLF，request 不經過
Go；endpoint 由 config 明確啟用且預設關閉。第一版採 asynchronous request／status
semantics，並沿用單一 active training 限制。Exact path、payload 與 authentication 是
detailed-plan implementation detail，不再是 architecture gate。Release 19 TS 28.105 仍只
作語意參考，不是 Release 18 compliance requirement。

---

## 10. Verification policy

### 10.1 Repository checks

`NWDAF/` production slice 至少執行：

```text
focused go test
make test
make build
make lint
```

`PyMTLF/` production slice 至少執行：

```text
focused pytest
full pytest
ruff check
```

涉及 shared state、callback race、deadline 或 cleanup 的 slice 必須加入 deterministic
concurrency tests；不得只以固定 sleep 證明 correctness。

### 10.2 Contract tests

至少涵蓋：

- standard preparation request required fields；
- `mLPreFlag: true` validation；
- assignment／result bundle schema and version rejection；
- static topology file path／version／identity／tree validation；
- registered capability 與 Root-assigned position mismatch；
- Root／Branch／Leaf behavior 由 `planId` assignment 驅動，而非 global role config；
- recipient／plan mismatch；
- artifact digest mismatch；
- Branch 未處理 Root bundle、直接把 Root URL 交給 Leaf 時必須測試失敗；
- Leaf 只從 Branch-published URL 取得 lower-tier bundle；
- prepared／failed／timed-out result interpretation；
- `COMPLETE_REQUIRED` Root decision and rejected-attempt cleanup；
- FedProx proximal loss、`proximal_mu` 與 global-reference consistency；
- typed `algorithm` block validation，並拒絕 top-level `proximal_mu` 或 generic untyped
  algorithm parameters；
- `fedavg`、`fixed_count`、`minimum_results` 與其他 unsupported strategy value rejection；
- `all` participant selection／waiting-policy validation；
- Branch effective sample count propagation；
- 至少兩個 Branch updates 的 Root `sample_weighted` aggregation；
- 不同 Leaf／Branch sample counts 的 end-to-end effective-weight propagation；
- notification `mlCorreId`／`notifCorreId`／subscription route correlation；
- upper／lower `roundInd` independence；
- restart 後不恢復舊 process，並拒絕 stale callback／result；
- degradation 與 private management request 可建立新 plan；
- private API 預設關閉；只有 config enabled 的 Root PyMTLF 掛載 route；
- private trigger request 直接由 PyMTLF 處理，不經過 Go；
- 同一 PyMTLF 的 single-active-training guard、idempotent retry 與 conflicting-request
  rejection；
- preparation／round failure 使整個 experiment terminal，且不自動 retry；
- `Nnwdaf_MLModelTraining` 不被用作 pre-training trigger；
- standard `ProblemDetails` and service-specific cause mapping。

### 10.3 Verification claims

Unit／mock tests 不得宣稱完成 real NRF、OAuth、TLS 或 multi-process integration。E2E
closure 必須分別記錄：build pass、repository test pass、cross-process pass、real NRF
discovery pass，以及未執行的 environment-dependent checks。

---

## 11. 第一版明確不包含

- 任意深度 recursive hierarchy；
- dynamic topology planner／automatic grouping；
- `root`／`branch`／`leaf` permanent runtime or NF-profile role；
- dynamic topology mutation without Root knowledge；
- cross-vendor hierarchy manifest interoperability；
- 新增非標準 public topology-control SBI；
- 修改 Release 18 OpenAPI schema 以加入 hierarchy fields；
- 把 manifest assignment 視為 preparation admission proof；
- FedAsync 或 staleness weighting 作為 mandatory behavior；
- per-tier strategy override；
- arbitrary strategy module／class dynamic loading；
- FedAvg、`fixed_count` participant selection、`minimum_results` waiting 或其他第一版
  strategy profile；
- `PARTIAL_ALLOWED` admission 或 `minimumClients`；
- 跨 preparation attempts 保留成功 participants 或增量補選；
- Root bundle URL 直接轉交 Leaf，或由 Go proxy artifact bytes；
- active hierarchy state persistence、restart recovery 或 subscription reconciliation；
- preparation、round 或 topology 的 automatic retry；
- 在 private API config disabled 的 PyMTLF 上暴露 training-trigger route；
- 由 Go NWDAF relay／proxy private training-trigger request；
- 對 Release 19 TS 28.105／TS 28.532 generic Provisioning MnS 的完整相容實作；
- production-grade Byzantine robustness、secure aggregation 或 differential privacy；
- 自動跨 operator hierarchical FL；
- 未經獨立核准的 Daisy runtime／networking migration。

上述項目若不直接阻擋目前 slice，依 development policy 記為 future-phase handoff 或
optional hardening，不得默默擴張進第一版。

---

## 12. Overall completion criteria

本計畫完成時必須同時滿足：

1. 所有節點都以標準 NWDAF capability 註冊，不增加 hierarchy NF type／profile role；
2. Root-only main config 可引用並驗證獨立 static topology file；
3. Root 只把 capability-eligible NWDAFs 指派為 Branch／Leaf，角色只在對應 `planId`
   process state 中存在；
4. 同一 NWDAF implementation 可依不同 topology assignments 承擔不同角色；
5. Branch 可同時維護 upper-tier Client process 與 lower-tier Server process，且 identity
   與 callback 不互相污染；
6. Root 會透過 NRF exact-instance discovery 驗證 static assignment；
7. assignment manifest 只提供候選提示，Branch 會重新確認 Leaf capability；
8. Branch 以非同步 preparation 建立實際 subordinate topology；
9. result bundle 保留有效 model artifact，並可讓 Root 重建 prepared／failed／timed-out
   participants；
10. Root 只依 `COMPLETE_REQUIRED` 接受或拒絕 topology；
11. 被拒絕的 preparation attempt 會進入 terminal failure 並執行可完成的 cleanup；不自動
    retry，operator 修復問題後才以新 `planId` 啟動新 experiment；
12. 第一版只接受 typed FedProx algorithm block、`all` selection、`all` waiting 與
    `sample_weighted` aggregation；`proximal_mu` 只存在於 algorithm block，其他 strategy
    values 在 validation 階段被拒絕；
13. Branch 可聚合 lower-tier updates，並以 subordinate sample count 總和作為 effective
    weight，將 interim local model 回報 Root；Root 會以至少兩個 Branch updates 驗證真正
    的 upper-tier weighted aggregation；
14. multi-round flow 正確隔離 upper／lower correlations、rounds、deadlines 與 cleanup；
15. Branch 下載並處理 Root bundle，再由 Branch PyMTLF 發布 Leaf bundle；Leaf 不直接使用
    Root artifact URL，Go 不 proxy artifact bytes；
16. degradation 與 config-enabled private management request 都可在標準 Training SBI
    procedure 之前建立新 plan；private request 直接由 Root PyMTLF 處理，不經過 Go；
17. private API 預設關閉，未啟用的 PyMTLF 不掛載該 route；
18. 同一 PyMTLF 同時間最多一個 active training；衝突 request 不會建立第二個 plan；
19. 任一必要 participant 的 preparation／round failure 終止整個 experiment，且不自動
    retry；
20. process restart 後不恢復舊 active state，stale callback／result 不會重建 process；
21. failure、timeout、duplicate、late result 與 bundle expiry 均有明確且有測試的 state
    transition；
22. existing non-hierarchical distributed FL behavior 沒有 regression；
23. documentation 明確區分 Release 18 3GPP-defined behavior、同 vendor implementation
    profile，以及僅供參考的 Release 19 management semantics。

---

## 13. Decision log

| Date | Decision | Status |
| --- | --- | --- |
| 2026-08-17 | 第一版採固定三層、非任意深度 hierarchy | Confirmed |
| 2026-08-17 | Root／Branch／Leaf 是 per-topology role，不是新 NF type | Confirmed |
| 2026-08-17 | Hierarchy 由多個獨立標準 FL processes 組成 | Confirmed |
| 2026-08-17 | Assignment metadata 放在仍含有效模型的 vendor model bundle | Confirmed |
| 2026-08-17 | Subordinate preparation detail 使用 `mLModelUrl` result bundle 回報 | Confirmed |
| 2026-08-17 | 接受上述 result bundle 的 procedure semantic tension，並明確標為同 vendor contract | Confirmed |
| 2026-08-18 | PyMTLF 直接 serving artifacts，Go 只傳遞 URL；Branch 必須處理 Root bundle 後重新發布 Leaf bundle | Confirmed |
| 2026-08-18 | 所有 process restart 都從全新 state 開始，不恢復或 reconcile 舊 hierarchy process | Confirmed |
| 2026-08-18 | 第一版 algorithm 只支援 FedProx；FedAvg 與 `proximal_mu: 0` baseline 延後 | Confirmed |
| 2026-08-18 | 第一版 selection 只支援 `all`，waiting 只支援 `all` | Confirmed |
| 2026-08-18 | 第一版 aggregation 只支援 `sample_weighted`，但保留 config 欄位供未來擴充 | Confirmed |
| 2026-08-18 | 不搬入 Daisy dynamic loading／完整 strategy hierarchy，只參考 strategy responsibility separation | Confirmed |
| 2026-08-18 | Preparation attempt 未達 admission condition 時進入 terminal failure；不自動重試，operator 重啟時使用新 `planId` | Confirmed |
| 2026-08-18 | 第一版不跨 attempts 保留成功 Leaves、不增量補選，也不由 Branch 自行替換 Leaf | Confirmed |
| 2026-08-18 | 所有節點都是標準 NWDAF；registered capability 決定 eligibility，Root assignment 決定 plan-scoped Branch／Leaf role | Confirmed |
| 2026-08-18 | Runtime 採 capability-oriented engines，不新增永久 `root`／`branch`／`leaf`／`fl_branch` mode | Confirmed |
| 2026-08-18 | 第一版由 Root PyMTLF main config 引用獨立 static topology file；Branch／Leaf 不保存完整 topology | Confirmed |
| 2026-08-18 | Static topology 只保存 identities／tree／admission；runtime identifiers 與 endpoints 每個 attempt 產生或解析 | Confirmed |
| 2026-08-18 | 第一版 admission 只支援 `COMPLETE_REQUIRED`；不做 partial、補選或跨 attempt 保留 | Confirmed |
| 2026-08-18 | Degradation 可由 producer 主動觸發；另提供 config-gated、預設關閉的 pre-training private management request API | Confirmed |
| 2026-08-18 | `Nnwdaf_MLModelTraining` 只用於 training procedure，不作為 training initiation API | Confirmed |
| 2026-08-18 | TS 28.105 `MLTrainingRequest` 只作 Release 19 management semantics 參考，第一版不宣稱相容 | Confirmed |
| 2026-08-18 | Private management API 直接由 Root PyMTLF 提供，不經 Go；採 asynchronous request／status semantics | Confirmed |
| 2026-08-18 | 同一 PyMTLF 沿用 single-active-training 限制，不平行建立第二個 hierarchy plan | Confirmed |
| 2026-08-18 | Preparation 或 round 任一必要 participant 失敗即終止 experiment；不自動 retry，由 operator 檢查後重新啟動 | Confirmed |
| 2026-08-18 | `proximal_mu` 放在 typed FedProx algorithm block，不作為 generic strategy 或 Leaf-local fitting 欄位 | Confirmed |
| 2026-08-18 | Slice 3 同時實作 Root initiation、single-active guard、static topology 與 assignment | Confirmed |
| 2026-08-18 | Preparation failure policy 歸 Slice 4，round failure policy 歸 Slice 5；Slice 6 只做 lifecycle closure／restart hardening | Confirmed |
| 2026-08-18 | E2E smoke 使用一個 Branch；aggregation acceptance 至少使用兩個 Branches | Confirmed |
| 2026-08-18 | 修正既有 preparation notification contract regression：`statusReport` 不得單獨代表成功，sender／validator／stage-aware receiver 改以 `mLModelInfos` profile 為準 | Confirmed |
| 2026-08-18 | preparation 的 `statusReport` 僅為 optional supplemental status；不再以固定 `samplRatio: 100` 作為 completed latch | Confirmed |
| 2026-08-18 | 第一版假設所有 NWDAF／PyMTLF 位於同一受控 vendor trust domain；publisher 僅為 logical identity，不新增 artifact-origin／requester cryptographic binding | Confirmed |
