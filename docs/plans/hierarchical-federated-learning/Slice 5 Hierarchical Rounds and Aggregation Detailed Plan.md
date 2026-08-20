# Slice 5：Hierarchical Rounds 與 Aggregation 詳細計畫

日期：2026-08-19

狀態：Completed；production implementation、review、remediation、full verification與
repository-separated commit已完成。PyMTLF implementation commit：`0d1b529`。2026-08-20
確認的`CANDIDATE_READY` semantic defect已由Slice 6 commit `cebfe90`修正

上層計畫：

- [Hierarchical NWDAF Federated Learning Implementation Plan](./Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)

前置 contracts：

- [Slice 0 Baseline Audit and Contract Freeze Detailed Plan](./Slice%200%20Baseline%20Audit%20and%20Contract%20Freeze%20Detailed%20Plan.md)
- [Slice 1 Hierarchy Bundle Contract and Artifact Primitives Detailed Plan](./Slice%201%20Hierarchy%20Bundle%20Contract%20and%20Artifact%20Primitives%20Detailed%20Plan.md)
- [Slice 2 Capability and Process-scoped Role Foundation Detailed Plan](./Slice%202%20Capability%20and%20Process-scoped%20Role%20Foundation%20Detailed%20Plan.md)
- [Slice 3 Root Initiation, Static Topology and Assignment Detailed Plan](./Slice%203%20Root%20Initiation%2C%20Static%20Topology%20and%20Assignment%20Detailed%20Plan.md)
- [Slice 4 End-to-end Preparation and Admission Detailed Plan](./Slice%204%20End-to-end%20Preparation%20and%20Admission%20Detailed%20Plan.md)

相關文件：

- [Hierarchical NWDAF Federated Learning Proposal Draft](../../proposals/nwdaf/hierarchical-federated-learning/proposal_draft.md)
- [Phase 3 And 4 Federated Training Execution Detailed Plan](../federated-learning/Phase%203%20And%204%20Federated%20Training%20Execution%20Detailed%20Plan.md)
- [NWDAF Development Policy](../../development_policy.md)

---

## 0. 2026-08-20 semantic erratum

本文件在Slice 5實作時把「configured rounds完成、final Root `ROUND_GLOBAL`形成」稱為
`CANDIDATE_READY`。這個名稱沿用了當時不完整的phase文字，但違反現有flat FL production
invariant：`CANDIDATE_READY`只有在final validation evidence完整且gate accepted後成立。

因此本文件後續所有「Slice 5進入／停在`CANDIDATE_READY`」應視為歷史implementation
checkpoint描述，其正確語意是：

```text
final Root ROUND_GLOBAL formed
-> candidate pending final validation
```

不修改已完成multi-round／aggregation的scope與commit歷史；production state remediation、
hierarchical final validation與publication由：

- [Slice 6 Hierarchical Final Validation and Publication Detailed Plan](./Slice%206%20Hierarchical%20Final%20Validation%20and%20Publication%20Detailed%20Plan.md)

負責。Slice 6完成前，不得以本文件原有的`CANDIDATE_READY` completion claim宣稱HFL產品流程
已完成。

---

## 1. 目的與完成邊界

Slice 5 從 Slice 4 的 `ADMITTED` state 接手，讓已凍結的 Root–Branch–Leaf participant
snapshot 實際完成 fixed-count synchronous hierarchical training rounds：Root 先對 Branches
發 upper-tier round；Branch 以明確的 upper／lower mapping 將同一模型重新發布並對 Leaves
發 lower-tier round；Root 在每輪 input 指定 Client `epochs`，Leaves 使用該指示與 FedProx
執行 local training；Branch 先做 lower-tier
`sample_weighted` aggregation，再以具有 subordinate provenance 與 effective sample count 的
upper-tier interim result 回報 Root；Root 最後再做一次 `sample_weighted` aggregation。

本 Slice 的成功終點是 `CANDIDATE_READY`：Root 已完成 configured rounds，最後一個
Root `ROUND_GLOBAL` artifact 是尚未 promotion 的 hierarchical candidate。成功時不在本 Slice
執行 hierarchy-aware final validation、`FINAL_MODEL` publication、ADRF store、catalog
promotion、Model Provision notification 或 cutover。

成功流程固定為：

```text
Root ADMITTED
  -> Root publishes upper ROUND_INPUT(r)
  -> Root PATCHes every admitted Branch subscription
  -> Branch validates upper command and maps it to one lower round
  -> Branch republishes lower ROUND_INPUT(lower-r)
  -> Branch PATCHes every admitted Leaf subscription
  -> Leaves execute FedProx from the immutable round input
  -> Leaves publish ROUND_LOCAL(TRAINING)
  -> Branch waits for all terminal Leaf outcomes or deadline
  -> Branch validates and sample-weights all Leaf models
  -> Branch publishes lower ROUND_GLOBAL(lower-r)
  -> Branch publishes upper ROUND_LOCAL(HIERARCHY_AGGREGATE)
  -> Branch notifies Root with upper round identity
  -> Root waits for all terminal Branch outcomes or deadline
  -> Root validates Branch provenance and effective weights
  -> Root sample-weights all Branch interim models
  -> Root publishes upper ROUND_GLOBAL(r)
  -> repeat until configured round_count is complete
  -> Root enters CANDIDATE_READY
```

本 Slice 不會：

- 重新執行 topology discovery、preparation 或 admission；
- 在 round 中補選、替換、增加或移除 Branch／Leaf；
- 支援 partial aggregation、`minimum_results` 或 failed-participant skip；
- 支援 FedAvg、FedAsync、staleness weighting 或 per-tier strategy override；
- 讓 Branch 使用自己的 local dataset 參與 training；
- 讓 Leaf 直接下載 Root round URL；
- 讓 Go 解析 hierarchy／round artifact 或執行 aggregation；
- 新增 public SBI、private round-control API 或 Go hierarchy state；
- persistence、resume、automatic round retry 或 restart recovery；
- hierarchy-aware final validation、durable model publication 或 cutover；
- 修改 Release 18 OpenAPI YAML 或 generated free5GC OpenAPI code。

Hierarchy-aware final validation／publication 若要成為第一版產品完成條件，必須另立
production slice；不得在 Slice 5 implementation 中順便接上既有 flat final-validation
flow，因為 Branch 沒有 local dataset，不能合法冒充一般 FL Client accuracy-check result。

---

## 2. 已確認且不得重新決策的事項

本 Slice 直接繼承以下 decisions：

- Root、Branch、Leaf 都是標準 NWDAF，不新增 NF type 或 permanent role config；
- role 由已驗證 assignment 與 active `planId` 決定；
- Root 是 upper FL Server；Branch 是 upper FL Client 與 lower FL Server；Leaf 是 lower
  FL Client；
- Root／Branch／Leaf 只沿用 Slice 4 admitted resources，不為每輪建立新 subscription；
- Root 控制 configured round 數量與整體 experiment 成敗；
- flat FL 由 flat Server 決定 Client epochs，hierarchical FL 只由 Root 決定；
- Branch 不得以本地 config 覆寫 Root 的 epochs，Leaf 不得使用 local epochs fallback；
- 第一版 algorithm 唯一合法值是 typed FedProx，`proximal_mu` 必須是 finite positive；
- participant selection 唯一合法值是 `all`；
- waiting policy 唯一合法值是 `all`；
- aggregation 唯一合法值是 `sample_weighted`；
- Branch effective sample count 是實際納入 lower aggregation 的 Leaf sample count 總和；
- 任一 required Branch／Leaf failure 或 timeout 都使完整 experiment terminal；
- 不自動 retry round、替換 participant 或建立新 plan；
- process restart 建立全新 memory state，不恢復舊 round；
- upper／lower `mlCorreId`、subscription、notification correlation、round state 與 deadline
  各自獨立；
- Branch 必須下載並處理 Root round artifact，再由 Branch PyMTLF serving lower artifact；
- model bytes 由發布 artifact 的 PyMTLF serving；Go 只傳遞 standard `mLModelUrl`；
- `statusReport` 是 optional supplemental status，不是 round completion proof；
- round success callback 必須有 `mLModelInfos` 與 `roundInd`；
- standard `termTrainReq` 表示該 participant 不再提供本次 training result；
- Go 只做 standard validation、routing、callback relay 與 peer error mapping；
- 不新增 expected Root ID、caller identity header、artifact signature 或 trust handshake。

Slice 4 已確認 preparation「收到第一個 failure 後仍收齊其他 participant terminal outcomes
或等到 bounded deadline」的診斷原則；既有 decision 尚未明文擴及 training round。本 draft
建議 Slice 5 hierarchy round 沿用相同原則：failure 會禁止 aggregation，但不立即丟棄其他已
dispatch participants 的 outcome。此項列在第 21 節供 team review；Flat FL 既有 fail-fast
behavior 不因本 Slice 改變。

---

## 3. Repository、branch 與 ownership

### 3.1 Baseline（2026-08-19）

| Repository | Branch | Revision | Slice 5 disposition |
| --- | --- | --- | --- |
| PyMTLF/ | feat/r18-hierarchical-federated-learning | 096c401 | production owner |
| NWDAF/ | feat/r18-hierarchical-federated-learning | 8885cd9 | verify-only expected |
| nwdaf-docs/ | main | 2d961cd | canonical plan |
| PyAnLF/ | existing branch | unchanged | out of scope |
| nrf/ | existing branch | unchanged | verify-only dependency |
| nwdaf-resources/ | existing branch | unchanged | Slice 7 E2E handoff |

Working trees 在本計畫開始前皆為 clean。Production implementation 必須維持 repository
separation；若 focused Go regression 證明既有 relay contract 有缺口，才在 NWDAF 的相同
feature branch 建立獨立 implementation commit。

### 3.2 PyMTLF owners

`FLRootCoordinator`：

- 只在 admission snapshot 已固定後開始 upper rounds；
- 決定 upper round sequence 與 configured round count；
- 保存 Root candidate 與 request-level observable state；
- 任一 upper round 失敗時終止整個 plan。

`FLBranchCoordinator`（由現有 `FLBranchPreparationCoordinator` 擴充／重新命名）：

- 保存 Slice 4 `BranchPreparationExecution`；
- 驗證 upper round command 與 parent assignment；
- 建立明確 `BranchRoundExecution` upper／lower mapping；
- 重新發布 lower round input；
- 呼叫 lower `FLServerEngine` 執行一輪；
- 將 lower aggregate 轉成 upper interim result；
- 不執行 Branch-local dataset training。

`FLServerEngine`：

- 繼續擁有 per-tier participants、standard PATCH、callback correlation、deadline、delay、
  round outcome collection 與 aggregation；
- 新增可由 Root／Branch coordinator 呼叫的「existing hierarchy process 執行一輪」入口；
- flat `_run()` flow 保持既有行為。

`FLClientEngine`：

- Leaf assignment-bound resource 執行 FedProx local round；
- Branch assignment-bound resource 將 upper round交給 Branch coordinator；
- plain resource 繼續走既有 flat local round；
- 繼續擁有 callback outbox、revision fence、delay timer 與 DELETE handling。

`FederatedTrainer`：

- 保留既有 local fitting 與 sample-weighted aggregation；
- 以明確參數接收 Server-supplied positive integer `epochs`；
- 以明確參數加入 FedProx proximal objective；
- flat call path 不傳 FedProx contract，維持既有 local objective。

`FLWorkspace`／`fl_artifacts`：

- 擁有 `ROUND_INPUT` 與 hierarchy interim artifact contract；
- 保持 archive safety、component digest、model／preprocessing contract 與 weights digest
  validation；
- 不執行 topology admission 或 callback routing。

### 3.3 NWDAF ownership

NWDAF 預期不需 production change。既有 target repository 已有：

- public `PATCH /nnwdaf-mlmodeltraining/v1/subscriptions/{subscriptionId}`；
- `application/merge-patch+json` parsing；
- local／remote route state 與 callback URI rewrite；
- peer PATCH 轉送與 `200`／`204` preservation；
- notification correlation 與 relay；
- standard `ProblemDetails` mapping。

若 implementation 不新增標準欄位，Go 不得加入 `planId`、hierarchy role、subordinate list、
effective sample count 或 aggregation state。

---

## 4. 現有證據與 gap audit

### 4.1 Release 18 standard evidence

直接規格證據：

- TS 29.520 V18.14.0 §4.6.2.2.4 與 OpenAPI
  `TS29520_Nnwdaf_MLModelTraining.yaml` 定義以 `PATCH` partial update existing
  subscription；request content type 是 `application/merge-patch+json`，成功 response 是
  `200 OK` 或 `204 No Content`。
- `NwdafMLModelTrainSubscPatch` 已有 `mLModelInfos`、`mLPreFlag`、`mLTrainRepInfo`、
  `roundInd`、`mLAccChkFlg` 與 `skipFlInd`，不需要 hierarchy-specific public field。
- TS 29.520 §5.5.6.2.3 將 `roundInd` 定義為 multi-round training 的 round number。
- TS 29.520 §5.5.6.2.8 要求 notification 至少提供 `delayEventNotif`、`mLModelInfos` 或
  `termTrainReq` 其中之一；`statusReport` 不在最低結果集合。
- TS 23.288 §6.2C.2.2 step 4 定義 FL Client 以 local data further train Server 提供的
  model，並向 Server 回報 interim local model information。
- TS 23.288 §6.2C.2.2 step 5 定義 FL Server 聚合 local model information 更新 global
  model；step 7 允許以 aggregated model information 進入 next round。

規格只定義每一個 FL Server／Client process。Branch 將 lower aggregate 組成 upper interim
result 是本專案的同 vendor hierarchical composition，不宣稱是 Release 18 定義的單一
hierarchy procedure。

### 4.2 Existing Go evidence

`NWDAF/internal/sbi` 已沿著既有 free5GC-style boundary 實作：

```text
server/router
  -> api_ml_model_training.go
  -> processor/ml_model_training.go
  -> backend or consumer/ml_model_peer_service.go
  -> containing or peer PyMTLF
```

`HandlePatchMLModelTraining` 會 parse patch、對 existing representation 套用 merge、驗證
immutable correlation，再將原 patch 送往 local backend 或 peer；consumer 已使用標準 PATCH
content type並接受 `200`／`204`。因此 Slice 5 不建立新 handler、processor、consumer、
context field 或 Go package。

free5GC skill 在本 Slice 的作用限於確認 SBI responsibility 與 OpenAPI priority。由於沒有
新的 Go endpoint、package 或 lifecycle shape，計畫不以外部 exemplar 推導新的 production
結構，也不宣稱 unit tests 等於 real NRF／OAuth／TLS integration。

### 4.3 Existing flat PyMTLF round baseline

現有 flat flow 已提供：

- `FLServerEngine._patch_round()`：以既有 subscription 送 `roundInd`、model URL 與
  `maxResTime`；
- `FLClientEngine._run_round()`：下載模型、驗證 model／preprocessing contract、使用 frozen
  dataset training、發布 `ROUND_LOCAL(TRAINING)` 並 callback；
- `FLServerEngine.receive_notification()`：驗證 `notifCorreId`、`mlCorreId`、expected
  `roundInd`、duplicate 與 termination；
- `FLServerEngine._aggregate_round()`：驗證 participant、scope、input digest、sample count，
  再以 `FederatedTrainer.aggregate()` 做 weighted average；
- `ROUND_LOCAL`／`ROUND_GLOBAL` typed artifact、component digest 與 immutable workspace；
- delay notification、bounded extension、callback retry、revision fence 與 cleanup。

Flat baseline 的 local trainer 目前只有 task loss；method 名稱與 error text仍使用 FedAvg
語意。Slice 5 只能泛化共用 primitive，不得讓 flat flow意外套用 positive proximal term。
現有 flat Client 也從本地 `federated_learning.client.training.epochs` 取得輪內次數；
這是本 Slice 明確要取代的舊 contract，不是必須保留的 compatibility behavior。

### 4.4 Slice 4 handoff evidence

Root 成功 handoff 已保留：

- `RootAdmissionSnapshot.plan_id`；
- canonical admitted Branch IDs 與各 Branch prepared Leaf IDs；
- each Branch upper `resource_location`；
- Branch preparation-result digest；
- attached upper `FLServerEngine` process；
- current base model與 strategy；
- active experiment registry reservation。

Branch 成功 handoff 已保留：

- validated parent assignment；
- resolved Leaf nodes；
- Branch-published Leaf assignments；
- attached lower Server process ID；
- lower participant subscription locations與correlations；
- same plan-bound experiment reservation。

Leaf 成功 handoff 已保留：

- validated Leaf assignment；
- frozen dataset snapshot；
- prepared training sample count；
- expected model／preprocessing contract digest；
- same long-lived upper Client subscription。

### 4.5 Confirmed production gaps

1. Root worker 在 `ADMITTED` 結束，沒有 upper round loop。
2. Branch upper Client 收到 round PATCH 會進現有 `_run_round()`，但 Branch 沒有 local
   dataset，正確行為應是驅動 lower round。
3. `FLBranchPreparationCoordinator` 尚未擁有 upper／lower round mapping。
4. hierarchy Server process 只有 preparation collection，沒有 callable one-round execution。
5. hierarchy round termination 目前會直接設 global process failure，不能先收齊其他 terminal
   outcomes。
6. local trainer 未實作 FedProx proximal term。
7. 現有 artifact 沒有明確的 round command role；round 0 base model也不能假裝是已有
   participants 的 `ROUND_GLOBAL`。
8. 現有 `ROUND_LOCAL(TRAINING)` 無法表達 Branch aggregate 的 subordinate provenance 與
   effective sample count。
9. Root request state沒有 round與candidate observability。
10. flat／hierarchy Client epochs仍由Client-local config決定，Server無法對round input指定。
11. 尚無 hierarchical round success、wrong-round、failure、timeout、weight propagation 與
    flat regression tests。

---

## 5. Standard wire contract

### 5.1 Upper-tier Root → Branch PATCH

每個 admitted Branch 沿用 Slice 4 `upper_resource_location`。Round `r` patch 固定為：

| Field | V1 value |
| --- | --- |
| `mLPreFlag` | `false` |
| `roundInd` | upper round `r` |
| `mLModelInfos` | exactly one event + Root-published upper `ROUND_INPUT(r)` URL |
| `mLTrainRepInfo.maxResTime` | Root upper round budget |
| `mLAccChkFlg` | absent |
| `skipFlInd` | absent |

HTTP request 使用 `application/merge-patch+json`；`200` 與 `204` 都表示 command accepted，
不是表示 Branch 已完成 lower aggregation。

### 5.2 Lower-tier Branch → Leaf PATCH

每個 admitted Leaf 沿用 Slice 4 lower `resource_location`。Branch 先完成 upper input download
與 validation，再發布 lower `ROUND_INPUT(lower-r)`，之後發：

| Field | V1 value |
| --- | --- |
| `mLPreFlag` | `false` |
| `roundInd` | explicit lower round ID |
| `mLModelInfos` | exactly one event + Branch-published lower input URL |
| `mLTrainRepInfo.maxResTime` | derived bounded lower budget |
| `mLAccChkFlg` | absent |
| `skipFlInd` | absent |

Leaf 不接收 Root URL。Branch 可以讓同一 lower process 的所有 Leaves 使用同一個
Branch-published input URL；recipient binding仍由各自的 standard subscription與correlation
負責，不為每個 Leaf 複製相同 round bundle。

### 5.3 Leaf → Branch notification

成功 callback 固定包含：

- existing lower `notifCorreId`；
- existing lower `mlCorreId`；
- exact lower `roundInd`；
- one `mLModelInfos` entry pointing to Leaf `ROUND_LOCAL(TRAINING)`；
- no required `statusReport`。

失敗 callback 使用相同 correlations與round，並帶 `termTrainReq`。Delay callback沿用既有
`delayEventNotif`；不得同時帶 model或termination。

### 5.4 Branch → Root notification

Branch 只有在全部 Leaves successful、lower aggregate 已發布且 upper interim artifact
完成後，才回：

- existing upper `notifCorreId`；
- existing upper `mlCorreId`；
- exact upper `roundInd`；
- one `mLModelInfos` entry pointing to Branch
  `ROUND_LOCAL(HIERARCHY_AGGREGATE)`。

若任一 required Leaf failure／timeout或Branch自身處理失敗，Branch等待其餘已dispatch Leaf
terminal outcomes或lower deadline後，以 upper `termTrainReq` 回報；不得提供部分 aggregate。

### 5.5 Standard／vendor boundary

Standard SBI不增加：

- `planId`；
- hierarchy role；
- parent／child IDs；
- `proximal_mu`；
- effective sample count；
- subordinate artifact digests。

上述 hierarchy／strategy／provenance全部由 PyMTLF process state與vendor model bundle
contract承載。Go relay不讀取或改寫 bundle。

---

## 6. Round artifact contract

### 6.1 New `ROUND_INPUT`

新增 `ArtifactRole.ROUND_INPUT`，明確表示「Server 已接受並準備送給本 tier Clients 的本輪
immutable global input」。它不是 aggregation result，也不要求虛構 participants。

`RoundInputMetadata` 至少包含：

```yaml
contract_version: "1.0"
ml_corre_id: <current tier process>
round_ind: <current tier round>
model_contract_digest: <sha256>
preprocessing_contract_digest: <sha256>
weights_digest: <sha256>
client_training:
  epochs: <positive integer>
```

Archive仍使用 exact `model.py`、`model.npy`、`scaler.pkl` component digests。Validation必須
確認 metadata weights digest等於實際 loaded weights，model／preprocessing contract與
preparation已凍結值一致。

`ROUND_INPUT` 不新增 caller authentication、publisher signature 或 intended-recipient
field。合法 command 由 existing subscription、route、resource revision、bound plan與
expected round共同決定；不得從 artifact 自己提供的 identity 推導一個假 expected value。

Root 每輪將 current Root global model投影成 upper `ROUND_INPUT`。Branch下載後，以相同 model
component bytes和weights重新發布 lower `ROUND_INPUT`，但改用 lower `ml_corre_id` 與lower
`round_ind`；`client_training` 必須原樣保留。這是 process translation，不是模型轉換或
transparent proxy。

`epochs` 的 authoritative source 是主動編排訓練的 Server config：flat flow 是 flat
Server，hierarchy flow 是 Root。它放在每輪 `ROUND_INPUT` 而不是 hierarchy
assignment，因為 flat 與 hierarchy 必須共用同一套明確 contract，且未來可允許
Server 依 round 選擇不同值。第一版雖從 config 固定取值，仍每輪明確攜帶。
Missing、zero、negative、non-integer 或 Client-local override 均使 round 失敗。

### 6.2 Leaf `ROUND_LOCAL(TRAINING)`

沿用既有 contract：

- lower `ml_corre_id`／`round_ind`；
- Leaf `participant_nf_instance_id`；
- frozen `scope_digest`；
- `input_global_weights_digest`；
- model／preprocessing／base／output weights digest；
- exact positive `training_sample_count`。

Branch必須下載並strict-load每個 local artifact後才可 aggregate。Notification內的URL或
participant自報sample count本身都不是 aggregation input，typed artifact validation成功後的
metadata才是。

### 6.3 Branch `ROUND_LOCAL(HIERARCHY_AGGREGATE)`

在既有 `RoundLocalResultType` 新增 `HIERARCHY_AGGREGATE`，用來表示 Branch在upper tier
扮演一個FL Client時回報的interim local model。它仍是可訓練完整模型bundle，不是只含
statistics的sidecar。

Metadata至少包含：

```yaml
contract_version: "1.0"
ml_corre_id: <upper process>
round_ind: <upper round>
participant_nf_instance_id: <Branch NF instance ID>
scope_digest: <upper subscription frozen scope>
input_global_weights_digest: <Root input weights digest>
model_contract_digest: <sha256>
preprocessing_contract_digest: <sha256>
base_weights_digest: <Root input weights digest>
weights_digest: <lower aggregate weights digest>
training_sample_count: <effective subordinate sample sum>
lower_round_ind: <mapped lower round>
lower_global_artifact_digest: <Branch ROUND_GLOBAL archive digest>
subordinate_participants:
  - participant_nf_instance_id: <Leaf ID>
    training_sample_count: <positive integer>
    local_artifact_digest: <Leaf ROUND_LOCAL archive digest>
```

Contract validation固定要求：

- subordinate IDs canonical sorted且unique；
- subordinate set等於Slice 4 admitted Leaves；
- `training_sample_count`等於全部subordinate sample count總和；
- `weights_digest`等於 lower `ROUND_GLOBAL` weights digest；
- input digest、model contract與preprocessing contract等於upper input；
- Branch ID、upper process、upper round與scope符合Root已知subscription state。

Root不依賴Branch local config推測effective weight，也不把每個Branch視為相同權重。

### 6.4 Per-tier `ROUND_GLOBAL`

沿用既有 `ROUND_GLOBAL`：

- Branch lower global的participants是Leaves；
- Root upper global的participants是Branches；
- 每個participant record保存其accepted local artifact digest與training sample count；
- `aggregated_training_sample_count`必須等於participants sample count總和；
- canonical ordering、model contract、preprocessing contract與input digest全部驗證。

Root round `r`輸出的 `ROUND_GLOBAL(r)` 是下一輪upper `ROUND_INPUT(r+1)`的模型來源。Final
configured round的Root `ROUND_GLOBAL`則成為unpromoted candidate。

### 6.5 Backward compatibility

- existing flat `ROUND_LOCAL(TRAINING)`與`ROUND_GLOBAL` JSON projection不變；
- flat Server 也必須先發布 `ROUND_INPUT`，不再直接把 seed／previous
  `ROUND_GLOBAL` URL 當成本輪 training input；
- `federated_learning.client.training.epochs` 從 Client config 移除；舊 config 必須明確
  migration，不得安靜地忽略或當成 fallback；
- new hierarchy result type 只在 hierarchy-bound production path 產生，`ROUND_INPUT` 則為
  flat／hierarchy 共用；
- legacy archive若沒有new fields仍依原discriminator驗證；
- unknown role／result type維持reject；
- 不修改OpenAPI或generated model。

---

## 7. Identity、correlation 與 authoritative source

| Value | Authoritative producer | Crossing | Receiver use |
| --- | --- | --- | --- |
| `planId` | Root initiation | assignment bundle | bind process；不放standard round payload |
| admitted Branch／Leaf set | Root topology + Slice 4 results | Root/Branch retained snapshots | exact participant set |
| upper `mlCorreId` | Root Server process | existing subscription／PATCH／callback | Root–Branch process correlation |
| lower `mlCorreId` | Branch Server process | existing subscription／PATCH／callback | Branch–Leaf process correlation |
| upper `roundInd` | Root coordinator | upper PATCH／input／callback | Root round state |
| lower `roundInd` | Branch coordinator | lower PATCH／input／callback | lower round state |
| upper/lower mapping | Branch coordinator | process-local immutable record | translate one upper command to one lower execution |
| strategy | Root config | validated assignment bundle | Branch validation與Leaf FedProx |
| client `epochs` | initiating Server config | each `ROUND_INPUT.client_training` | Leaf fitting loop；Branch only preserves |
| global weights | Root current model／previous Root aggregate | Root input，再由Branch republish | local-training reference |
| Leaf sample count | Leaf frozen dataset result | typed local artifact | Branch weight |
| Branch effective count | Branch validated lower collection | hierarchy aggregate artifact | Root weight |

Upper／lower round values在第一版正常流程可能同為`0..N-1`，但implementation不得利用數值
相等推導mapping。`BranchRoundExecution`至少保存：

```text
plan_id
upper_client_subscription_id
upper_resource_revision
upper_ml_corre_id
upper_round_ind
upper_input_artifact_digest
lower_server_process_id
lower_ml_corre_id
lower_round_ind
lower_input_artifact_digest
state
```

同一upper command digest重送時回到同一mapping與result；同一upper round但不同command
digest必須reject並使experiment失敗。Stale worker在revision、round或mapping不current時不得
發布callback或改動next-round state。

---

## 8. State model

### 8.1 Root request

在現有state後增加：

```text
ADMITTED
  -> ROUND_DISPATCH
  -> ROUND_WAITING
  -> AGGREGATING
  -> ROUND_DISPATCH ...
  -> CANDIDATE_READY

ADMITTED / ROUND_DISPATCH / ROUND_WAITING / AGGREGATING
  -> FAILED
```

`RootRequestSnapshot`應能觀察current upper round與candidate URL／digest，但不得暴露mutable
internal participant objects。Slice 3 private status API沿用同一resource，不新增start-round
route。

### 8.2 Hierarchy Server process

Slice 4 `READY`是每輪入口：

```text
READY
  -> ROUND_DISPATCH
  -> ROUND_WAITING
  -> ROUND_EVALUATING
  -> AGGREGATING
  -> READY

READY -> ... -> FAILED
```

Root final configured round成功後，由Root coordinator將upper process標為
`CANDIDATE_READY`。Branch lower process每輪完成後回到`READY`，等待下一個upper command。

進入`ROUND_EVALUATING`後，本輪terminal partition凍結；late callback只能作exact duplicate
ack或reject，不能改動aggregate input。

### 8.3 Branch upper Client resource

```text
READY
  -> ROUND_RUNNING
  -> RESULT_PENDING
  -> READY

READY / ROUND_RUNNING / RESULT_PENDING
  -> FAILED
```

`ROUND_RUNNING`在Branch代表lower orchestration，不代表Branch local fitting。Outbox取得Root
`204`後才回`READY`。Parent DELETE依Slice 4 hierarchy cancellation path向下取消lower process。

### 8.4 Leaf Client resource

沿用既有：

```text
READY
  -> ROUND_RUNNING
  -> RESULT_PENDING
  -> READY
```

只有assignment role是LEAF時使用FedProx；plain flat Client resource維持現有training。

---

## 9. Root round orchestration

Root只有在下列全部成立後開始：

- request state是`ADMITTED`；
- admission snapshot與active experiment reservation仍current；
- attached upper Server process仍是Slice 4 process；
- catalog current artifact仍是本attempt base lineage；
- strategy仍是assignment內已發布的first-version contract；
- 所有admitted upper resources仍存在。

每輪`r`：

1. 從Root `server.client_training.epochs`取得positive integer，以current Root global model建立
   immutable upper `ROUND_INPUT(r)`；
2. 對每個admitted Branch重設expected outcome、digest、delay與deadline state；
3. PATCH全部existing upper subscriptions；
4. 即使一個Branch先回termination，仍等待其他已dispatch Branch terminal outcomes或deadline；
5. 若任何Branch failed／timed out，不下載partial success做aggregation；
6. success時strict-download每個`HIERARCHY_AGGREGATE`；
7. 驗證Branch ID、upper correlation／round、admitted subordinate set、input digest與effective
   count；
8. 以Branch effective sample count執行Root `sample_weighted` aggregation；
9. 發布Root `ROUND_GLOBAL(r)`；
10. 若還有下一輪，將其model投影成`ROUND_INPUT(r+1)`；否則凍結candidate。

Root不得直接下載Leaf local results作aggregation，也不得以topology中的Leaf數量推算Branch
weight。

---

## 10. Branch round orchestration

Branch收到upper PATCH後先由`FLClientEngine`完成standard resource／revision validation，再依
stored assignment role分流：

- role `LEAF`：local FedProx；
- role `BRANCH`：Branch coordinator；
- no hierarchy assignment：existing flat local round。

Branch coordinator每次upper round：

1. 確認plan、parent assignment、upper subscription與lower process仍current；
2. strict-download upper `ROUND_INPUT`；
3. 驗證upper `mlCorreId`、round、event、model contract、preprocessing contract與loaded
   weights digest，並驗證`client_training.epochs`為positive integer；
4. 建立immutable `BranchRoundExecution`；
5. 以upper model重新發布lower `ROUND_INPUT`，改寫lower process／round metadata，並
   原樣保留`client_training`；
6. 依derived lower budget PATCH所有admitted Leaves；
7. 收集全部terminal outcomes或deadline；
8. success時驗證Leaf local artifacts並publish lower `ROUND_GLOBAL`；
9. 以lower aggregate model發布upper `HIERARCHY_AGGREGATE`；
10. 透過upper Client callback outbox回報Root。

Branch不得：

- 呼叫dataset builder；
- 用Root URL直接PATCH Leaves；
- 用自己的`server.round_count`自行多跑lower rounds；
- 在沒有新upper command時推進lower round；
- 將partial Leaf results聚合；
- 修改Root傳來的strategy或`client_training`。

Branch local `server.round_count`在hierarchy lower flow不具控制權；每個upper command只建立
一個lower execution。Lower next-round只由下一個upper PATCH觸發。

---

## 11. Leaf FedProx objective

### 11.1 Source of `proximal_mu`

Leaf只從已驗證Leaf assignment的typed strategy讀取：

```yaml
algorithm:
  name: fedprox
  proximal_mu: <finite positive number>
```

不得從Leaf local fitting config、round PATCH、untyped parameters map或environment override
取得`proximal_mu`。Round PATCH不重複傳strategy；resource-bound assignment是authoritative
contract。

### 11.2 Source of `epochs`

Leaf只從當輪已strict-loaded且與subscription／round綁定的`ROUND_INPUT` 讀取：

```yaml
client_training:
  epochs: <positive integer>
```

Flat Server與Root從各自`federated_learning.server.client_training.epochs`產生此欄位。
Branch只做process translation並原樣保留。Leaf不從本地Client config、environment、
assignment或PATCH取值；不支援missing-field fallback。

`epochs`是Server對本輪local fitting的明確指示，但不改變PATCH才是round command
的wire semantics。`batch_size`、`learning_rate`、device、validation split與random seed
仍由Client runtime config提供，不在此Slice一併下發。

### 11.3 Objective

對每個local batch：

```text
total_loss(w) = task_loss(w) + (proximal_mu / 2) * Σ ||w - w_global||²
```

其中`w_global`：

- 來自本輪strict-loaded `ROUND_INPUT`；
- 在local optimizer開始前clone、detach並凍結；
- 不因optimizer step改動；
- 只比較對應trainable parameters；
- device與dtype明確對齊；
- 不包含不存在於model contract的parameter。

Task loss沿用現有Huber loss與deterministic data order／seed。Task、proximal與total loss任一
非finite都使Leaf round失敗並回termination。

Local optimizer必須恰好執行`ROUND_INPUT.client_training.epochs`個full-dataset
passes。訓練artifact無須回傳epochs；Server以已綁定的round input作為expected
directive，Client若無法執行就回報termination，不自行降低或替換值。

### 11.4 Flat compatibility

`FederatedTrainer.train()`應採typed／explicit strategy input。Plain flat flow未綁hierarchy
assignment時，繼續使用既有local objective；不得用`proximal_mu=0`偷偷把flat path改名成
first-version FedAvg config，也不得放寬hierarchy config接受0。Flat flow仍必須改由
flat Server發布含`client_training.epochs`的`ROUND_INPUT`，並使Client用該值訓練。

---

## 12. Aggregation與effective weight

### 12.1 Branch lower aggregation

對Leaves `i`：

```text
branch_weights = Σ(local_weights_i * samples_i) / Σ(samples_i)
branch_effective_samples = Σ(samples_i)
```

只有floating tensors加權；non-floating tensors必須與input global一致。Tensor name、shape、
dtype與model contract任一不相容即整輪失敗。

### 12.2 Root upper aggregation

對Branches `j`：

```text
root_weights = Σ(branch_weights_j * effective_samples_j)
             / Σ(effective_samples_j)
```

Root weights使用Branch artifact內經typed validation的effective sample count。例如Branch A
代表100 samples、Branch B代表900 samples，Root weighting是10%／90%，不是50%／50%。

### 12.3 Determinism

- participants依canonical NF instance ID排序；
- tensor依base state contract順序處理；
- accumulation使用明確高精度dtype後cast回base dtype；
- 同一組validated artifacts產生相同weights digest；
- aggregation開始前participant snapshot immutable；
- aggregate完成後late callback不會改變candidate。

---

## 13. Deadline、delay與waiting policy

### 13.1 Root budget

Root upper round budget來自Root `FLServerSettings.round_timeout_seconds`，並透過
`mLTrainRepInfo.maxResTime`下送。Existing bounded delay extension policy仍可處理participant
在deadline前送出的`delayEventNotif`。

### 13.2 Branch lower budget

Branch的parent upper resource已驗證`maxResTime`大於local
`callback_deadline_margin_seconds`。Lower budget固定為：

```text
min(
  Branch FLServerSettings.round_timeout_seconds,
  parent maximum response time - Branch callback deadline margin
)
```

它必須為positive。這個margin保留給Branch aggregation、artifact publication與upper callback。
若實際處理仍無法在parent budget內完成，Branch依existing Client timer在deadline前送delay；
Root只可在configured extension budget內grant。

### 13.3 `all` outcome collection

Hierarchy tier在dispatch完成後等待：

- 每個selected participant產生一個terminal success／failure outcome；或
- bounded deadline到期。

第一個failure會將本輪標為不可aggregation，但不提前結束collection。Deadline後未回報者
標為timed out。只有所有participants均為validated success才進aggregation。

Delay不是terminal outcome；exact duplicate delay idempotent，conflicting／exhausted extension
使該round failure。

---

## 14. Failure、cancellation與cleanup

### 14.1 Failure propagation

| Failure point | Local action | Upstream action |
| --- | --- | --- |
| Root input publication／PATCH | freeze Root round failed | cancel all upper resources |
| Branch upper input validation | no lower dispatch | upper termination callback |
| Branch lower dispatch | cancel already-created lower work | wait terminal cleanup then upper termination |
| Leaf local training／artifact | record Leaf failed | Branch round fails；no partial aggregate |
| Branch lower aggregation／publication | preserve no success interim | upper termination callback |
| Root Branch-result validation／aggregation | no Root aggregate | complete experiment fails |
| tier deadline | mark missing participants timed out | complete experiment fails |

Branch或Root在round failure後不得推進next round。Root負責對all Branches發parent DELETE；
Branch hierarchy DELETE path向下取消Leaves。Cleanup遵守experiment registry
`ACTIVE -> TERMINAL -> CLEANING -> release`順序。

### 14.2 Callback combinations

- models only：candidate success outcome，仍需artifact validation；
- termination only：failure；
- models + termination：記錄terminal failure，不進aggregation；
- delay only：pending；
- delay + models／termination：wire invalid；
- status only：wire invalid；
- models + optional status：success candidate；
- termination + optional status：failure。

### 14.3 Parent DELETE與shutdown

- hierarchy parent DELETE在round中仍可cancel，且process-locally idempotent；
- cancellation先fence new publication／dispatch，再wake waiters，最後做bounded cleanup；
- shutdown順序維持Root、Branch coordinator、registry admission fence、Client／Server engines；
- stale worker不得在resource revision或plan retired後enqueue callback；
- flat in-progress DELETE仍維持既有`ML_TRAINING_NOT_COMPLETE` behavior。

### 14.4 Success retention

`CANDIDATE_READY`時保留：

- Root candidate `ROUND_GLOBAL` URL／digest；
- Root admission snapshot；
- upper／lower subscriptions與process mappings；
- required round artifacts；
- active experiment reservation。

本 Slice不自行promote或release successful experiment。後續finalization／lifecycle slice接手；
failure所需cleanup則不得延後。

---

## 15. Detailed implementation scope

### 15.1 PyMTLF expected files

Production candidates：

- `src/py_mtlf/config.py`
  - Server-owned `client_training.epochs`、Client-local epochs removal與strict validation；
- `config/fl-server.yaml`、`config/fl-server-hierarchy.yaml`、`config/fl-server-client.yaml`與
  `config/fl-client.yaml`
  - initiating Server epochs examples與Client-local epochs migration；
- `src/py_mtlf/core/fl_root.py`
  - Root round states、loop、candidate snapshot與failure propagation；
- `src/py_mtlf/core/fl_branch.py`
  - generalize coordinator、round mapping、lower execution與upper interim publication；
- `src/py_mtlf/core/fl_server.py`
  - hierarchy one-round API、stage-aware terminal collection與typed aggregation mode；
- `src/py_mtlf/core/fl_client.py`
  - Branch／Leaf／flat round dispatch、FedProx input與stale fences；
- `src/py_mtlf/core/federated_trainer.py`
  - FedProx objective與generic sample-weighted wording；
- `src/py_mtlf/core/fl_artifacts.py`
  - `ROUND_INPUT`、`HIERARCHY_AGGREGATE` contracts；
- `src/py_mtlf/core/fl_workspace.py`
  - publish／strict-download round input helpers；
- `src/py_mtlf/core/fl_experiment.py`
  - only if candidate／round ownership needs an existing-record extension；
- `src/py_mtlf/wire/ml_model_training.py`
  - only if focused tests reveal missing standard round correlation validation；
- `src/py_mtlf/app.py`
  - only for coordinator rename／wiring，不新增route。

不預先建立新的Python package。若現有owner files無法容納distinct stable responsibility，必須
先回到plan說明ownership與dependency direction。

### 15.2 NWDAF expected scope

Verify-only focused files：

- `internal/compat/mlmodeltraining/validation_test.go`；
- `internal/sbi/processor/ml_model_training_test.go`；
- existing consumer tests if needed。

只有deterministic regression證明PATCH／notification relay不符current OpenAPI或本Slice標準
payload時，才修改production Go。不得新增Go package或hierarchy-aware route state。

### 15.3 Explicitly untouched

- `PyAnLF/`；
- `nrf/`；
- Release 18 OpenAPI YAML；
- generated free5GC OpenAPI；
- `resources/`；
- `nwdaf-resources/`（留給Slice 8 profiles）。

---

## 16. Implementation order與checkpoints

### Checkpoint 1：Round artifact contract

先以isolated tests定義：

- valid／invalid `ROUND_INPUT`；
- required positive integer `client_training.epochs`；
- `HIERARCHY_AGGREGATE` subordinate canonicalization與sample sum；
- component／weights／model／preprocessing digest mismatch；
- flat artifacts backward compatibility；
- unknown role／result type rejection。

完成後才修改orchestration。

### Checkpoint 2：Server-controlled epochs與FedProx local objective

- flat Server／Root config產生round epochs directive；
- Branch原樣轉傳，Leaf local config無法覆寫；
- trainer恰好執行Server指定epochs；
- missing／zero／negative／non-integer epochs reject；
- pure proximal penalty test；
- global-reference clone保持immutable；
- positive `proximal_mu`影響optimizer objective；
- nonfinite loss／parameter failure；
- hierarchy Leaf uses assignment value；
- flat local trainer regression unchanged。

### Checkpoint 3：Reusable hierarchy one-round Server primitive

- existing hierarchy process從READY進一輪再回READY；
- PATCH all admitted participants；
- exact duplicate／conflicting duplicate／wrong round；
- all terminal outcome collection；
- delay extension與deadline；
- `ROUND_EVALUATING` freeze；
- success typed aggregate；
- failure不產生partial aggregate。

### Checkpoint 4：Branch upper／lower translation

- upper command strict validation；
- immutable explicit mapping；
- Root URL不轉交Leaf；
- lower budget derivation；
- no Branch dataset builder call；
- lower global與upper interim weight／digest一致；
- exact retry returns same result；
- cancellation fences dispatch／publication。

### Checkpoint 5：Root multi-round loop

- admission before first dispatch；
- upper `all` dispatch／waiting；
- at least twoBranch effective-weight aggregation；
- prior Root global feeds next upper input；
- final configured round becomescandidate；
- no final validation／publication／cutover；
- any failure terminatescomplete plan。

### Checkpoint 6：Contract relay regression

- Go/Py PATCH accepts standard round fields；
- callback relay preserves `mlCorreId`／`notifCorreId`／`roundInd`／model URL；
- `200`／`204` accepted；
- status-only still rejected；
- models + optional status accepted；
- noGo hierarchy field orbundle parsing。

### Checkpoint 7：Full verification、mandatory review與remediation

Implementation與full verification完成後，不中斷進行mandatory initial review。Admitted
in-scope findings依development policy test-first修正並做targeted follow-up review，直到closed
或真正decision gate。最後再建立repository-separated commits。

---

## 17. Acceptance test matrix

### 17.1 Artifact與strategy

- `ROUND_INPUT` round／process／digest round trip；
- `client_training.epochs` required且只接受positive integer；
- first input可由assignment base產生，不虛構participants；
- Branch lower input component bytes與upper input一致；
- Leaf只接受prepared contract相容input；
- typed FedProx positive finite value；
- `fedavg`、mu 0、top-level mu、unknown strategy仍reject；
- hierarchy aggregate subordinate partition與effective count exact。

### 17.2 Leaf

- prepared Leaf完成一輪FedProx並callback；
- Leaf恰好執行Root-supplied epochs，本地舊epochs值無法覆寫；
- global reference未被optimizer修改；
- frozen dataset sample count change使round fail；
- wrong event／model contract／preprocessing contract／weights digest reject；
- exact duplicate PATCH不重訓；
- conflicting duplicate／stale round fail；
- termination保留round identity。

### 17.3 Branch

- upper Branch resource不呼叫local dataset builder／trainer；
- one upper command只dispatch one lower round；
- upper epochs原樣出現在lower `ROUND_INPUT`，Branch local config不影響；
- explicit mapping在upper／lower round數值不同時仍正確；
- every Leaf收到Branch URL而非Root URL；
- first Leaf failure後仍收其他terminal outcomes或等deadline；
- any Leaf failure／timeout產生upper termination且沒有aggregate；
- all Leaves success產生lower global與upper hierarchy aggregate；
- effective sample count是subordinate sum；
- parent DELETE與shutdown取消waiting worker；
- stale worker不能回報next plan。

### 17.4 Root

- 未ADMITTED不得dispatch；
- admission snapshot之外的Branch result reject；
- first Branch failure後仍完成bounded terminal collection；
- any Branch failure／timeout使experiment failed；
- wrong upper round／correlation／Branch ID／subordinate set reject；
- two Branches不同effective counts得到正確weighted result；
- round 0 input是admitted base；
- round r+1 input weights等於Root `ROUND_GLOBAL(r)`；
- configured rounds完成後`CANDIDATE_READY`；
- candidate未promotion且沒有round之外的PATCH；
- failure cleanup cascades upper to lower。

### 17.5 Flat regression

- flat Server從`server.client_training.epochs`發布`ROUND_INPUT`；
- flat Client使用Server-supplied epochs，不使用Client-local fallback；
- old Client config中的`training.epochs`有明確migration，不被安靜忽略；
- flat Client仍使用existing local objective；
- flat Server round loop／FedAvg aggregation結果不變；
- flat termination仍使process fail而不繼續next round；
- flat in-progress DELETE仍reject；
- flat final validation／publication path不因new artifact discriminator破壞。

### 17.6 Concurrency

- callback arriving duringdispatch；
- exact duplicate before／after collection freeze；
- conflicting duplicate after first accepted result；
- cancellation betweenupper validation and lower publication；
- cancellation duringlower PATCH fan-out；
- deadline simultaneous withlast callback；
- Root failure whileBranch callback pending；
- shutdown wakesRoot andBranch waiters。

Concurrency tests使用Event、barrier、controllable clock或fault injection；固定sleep不得作為
唯一proof。

---

## 18. Verification commands

### 18.1 PyMTLF focused

實際test names依implementation落點調整，但至少執行：

```bash
.venv/bin/pytest -q \
  tests/test_fl_artifacts.py \
  tests/test_fl_hierarchy_artifacts.py \
  tests/test_local_trainer.py \
  tests/test_fl_client.py \
  tests/test_fl_server.py \
  tests/test_fl_branch.py \
  tests/test_fl_root.py \
  tests/test_ml_model_training_wire.py \
  tests/test_runtime_modes.py
```

### 18.2 PyMTLF full

```bash
.venv/bin/pytest -q
.venv/bin/ruff check .
```

### 18.3 NWDAF

若verify-only tests未要求production delta，執行focused processor／compat regressions後仍做：

```bash
make test
make build
make lint
```

### 18.4 Document與diff

```bash
git diff --check
```

### 18.5 Claims boundary

Unit／mock verification只能證明process-local orchestration與mocked HTTP contract。Real NRF、
OAuth、TLS、multi-process deployment與cross-host artifact download留給Slice 8，不得在Slice 5
完成報告中宣稱已驗證。

---

## 19. Risks與controls

### 19.1 Branch誤走local training

風險：現有`FLClientEngine`只看round fields，Branch會嘗試使用不存在的dataset。

控制：先依validated assignment role分流；Branch path只能呼叫Branch coordinator，test assert
dataset builder與local trainer未被呼叫。

### 19.2 Cross-tier identity collision

風險：upper／lower round可能同為相同整數，實作誤用一組correlation。

控制：immutable mapping保存兩組process／subscription／revision／round；unit test刻意使用不同
round值。

### 19.3 Incorrect Root weighting

風險：Root平均Branches或相信未驗證的自報count，造成兩層權重錯誤。

控制：typed hierarchy aggregate保存subordinate list與exact sum；Root以artifact validation後
effective count作weight。

### 19.4 Parent deadline被lower work吃完

風險：Branch等到upper deadline才開始aggregate，無時間callback。

控制：lower budget受parent maximum與Branch callback margin限制；delay必須在parent deadline
前送出且extension bounded。

### 19.5 Partial aggregate誤用

風險：一個Leaf失敗後仍將success子集合aggregate並回Root。

控制：terminal partition先freeze；只有all-success進aggregation；failure test assert沒有global
artifact與model callback。

### 19.6 Flat behavior regression

風險：共用Server collection或trainer泛化改變既有flat semantics。

控制：flat local objective與result contract保持，但round input改為Server-published
`ROUND_INPUT`且epochs改由Server控制；full flat suite必須更新並通過，不得以
Client-local fallback保持舊行為。

### 19.7 Candidate被誤當正式模型

風險：最後Root `ROUND_GLOBAL`被寫入catalog或觸發Provision notification。

控制：Slice 5終點明確是unpromoted `CANDIDATE_READY`；tests assert no model ID、ADRF store、
catalog update或cutover。

---

## 20. Deferred work classification

### Future-phase handoff

- hierarchy-aware final validation；
- `FINAL_MODEL` publication、ADRF store、catalog promotion、Provision notification與cutover；
- successful candidate terminal unsubscribe與complete cleanup；
- restart／stale interaction／artifact expiry完整matrix（Slice 6）；
- real multi-process topologies與two-Branch E2E（Slice 7）；
- dynamic participant maintenance、reselection與partial admission；
- FedAvg、fixed-count selection、minimum-results waiting與其他aggregation。

### Optional hardening

- cryptographic artifact signatures；
- cross-vendor hierarchy contract；
- Byzantine robustness、secure aggregation、differential privacy；
- bandwidth-awaredelta update或model compression。

### Integration verification gap

- real NRF exact-instance discovery duringrounds；
- OAuth／TLS peer PATCH與callbacks；
- cross-host artifact serving throughput；
- real multi-process deadline behavior。

以上不阻擋Slice 5 unit／mock acceptance。

---

## 21. Decision gates

目前沒有會阻止plan review的architecture contradiction。下列邊界已在本draft明確提出，team
review時必須確認，確認後implementation不得自行更改：

| Item | Recommended Slice 5 decision |
| --- | --- |
| Success endpoint | `CANDIDATE_READY`，不含hierarchy final validation／promotion |
| Round input | new vendor `ROUND_INPUT`，不把base假裝成`ROUND_GLOBAL` |
| Branch upper result | `ROUND_LOCAL(HIERARCHY_AGGREGATE)`，帶subordinate provenance與effective count |
| Strategy source | only validated assignment；round PATCH不重複strategy |
| Client epochs | flat由flat Server、hierarchy由Root決定；`ROUND_INPUT` every round；Branch preserves；no Client fallback |
| Branch local training | never；Branch只做lower orchestration與aggregation |
| Failure waiting | hierarchy tier等待all terminal outcomes或deadline；failure禁止aggregate |
| Lower timeout | parent budget minusBranch callback margin，並受local server timeout cap |
| Go production | verify-only expected；不加hierarchy state／field／API |

若team要求Slice 5同時完成hierarchy-aware final validation與durable model promotion，這會擴大
behavior、artifact contract、Branch responsibility與acceptance matrix，必須先更新canonical
plan；不得視為implementation-local追加。

---

## 22. Review checklist

### Contract

- [x] standard PATCH fields、content type與`200`／`204`符合OpenAPI
- [x] success／delay／termination callback combinations符合TS 29.520
- [x] no public/private round API added
- [x] no Go hierarchy state or OpenAPI/generated change
- [x] vendor fields只存在artifact／PyMTLF state
- [x] `ROUND_INPUT.client_training.epochs` required、typed且不進入standard PATCH field

### State與ownership

- [x] Root只在ADMITTED後dispatch
- [x] Branch upper／lowermapping explicit且immutable
- [x] Branch不執行local dataset training
- [x] Leaf strategy來自validated assignment
- [x] Root／flat Server是epochs authoritative source
- [x] Branch原樣轉傳epochs，Leaf無local fallback
- [x] successful subscriptions retained throughcandidate handoff
- [x] failure cleanup followsregistry lifecycle

### Training與aggregation

- [x] FedProx proximal reference immutable
- [x] positive finite mu enforced
- [x] trainer恰好執行Server-supplied positive integer epochs
- [x] all／all semantics enforced
- [x] Branch effective count equalsLeaf sum
- [x] Root usesBranch effective counts
- [x] no partial aggregate onfailure／timeout
- [x] deterministic participant／tensor ordering

### Concurrency與failure

- [x] duplicate／late／wrong-round behavior deterministic
- [x] collection frozen beforeartifact I/O
- [x] stale workers fenced byrevision／round／plan
- [x] cancellation wakesall waiters
- [x] first failure still yieldsbounded terminal collection
- [x] no automatic retry ornew plan

### Verification

- [x] focused PyMTLF tests pass
- [x] PyMTLF full pytest／Ruff pass
- [x] NWDAF focused／full build-test-lint pass
- [x] flat FL regressions pass
- [x] mandatory review與targeted remediation complete
- [x] repository-separated commits recorded

### Implementation record（2026-08-19）

實作 repositories：

- `PyMTLF` commit `0d1b529`：完成Root multi-round orchestration、Branch upper／lower
  mapping、Branch lower aggregation、Root effective-sample aggregation、Leaf FedProx、
  Server-controlled epochs、typed artifacts、observable status與hierarchy failure cleanup；
- `NWDAF`：production未修改，維持standard validation／relay boundary；驗證基準commit
  `8885cd9`；
- `nwdaf-docs`：回填本文件與上層計畫的完成狀態、實作與驗證紀錄；
- `PyAnLF`：未修改。

Verification：

- PyMTLF focused matrix：`163 passed, 2 skipped`；review remediation affected suites：
  `77 passed`；
- PyMTLF full：`403 passed, 2 skipped`；`ruff check .`通過；
- NWDAF：`make test`、`make build`、`make lint`通過，lint為`0 issues`；
- `git diff --check`於PyMTLF通過；NWDAF working tree維持clean。

Mandatory review與targeted remediation修正下列不改變architecture decision的問題：

1. Branch mapping原先在lower dispatch後才固定，且upper／lower round index仍依賴數值相同；
2. hierarchy failure可能在lower resources清理前先回到上層；
3. Root request未完整投影dispatch、waiting、aggregation與candidate progress；
4. parent在lower PATCH fan-out途中取消後，仍可能dispatch後續participants；
5. multi-round lineage、two-Branch effective weighting、deadline、duplicate、late callback、
   cancellation與shutdown缺少direct deterministic evidence。

修正後已完成focused recheck、PyMTLF full verification與plan-to-implementation follow-up
review。驗證範圍是unit／mock與local build；real NRF、OAuth、TLS、multi-process deployment與
cross-host artifact download仍依計畫留給Slice 8，不在本Slice宣稱已驗證。

---

## 23. Completion criteria

Slice 5只有在下列條件全部成立後才能標為Completed：

1. Root從Slice 4 `ADMITTED`開始且不重做preparation；
2. Root使用existing upper subscriptions完成configured rounds；
3. Branch將每個upper command映射為exactly one lower round；
4. Branch下載Root input並由Branch PyMTLF重新發布lower input；
5. Root在每輪`ROUND_INPUT`指定positive integer epochs，Branch原樣轉傳，Leaf依該值
   與validated assignment執行FedProx；
6. upper／lowerprocess、subscription、round與deadline彼此隔離；
7. Leaves以typed local artifacts回報exact positive sample count；
8. Branch只在all Leaves success時sample-weighted aggregate；
9. Branch upper interim artifact保存subordinate provenance與effective count；
10. Root以至少兩個Branch effective weights驗證upper weighted aggregation；
11. next round input來自previous Root global artifact；
12. duplicate、late、wrong-round、incompatible artifact與timeout全部有deterministic tests；
13. 任一requiredparticipant failure使complete experiment terminal且無partial aggregate／retry；
14. failure cleanup從Root向Branch／Leaf bounded cascade；
15. final configured Root aggregate形成unpromoted `ROUND_GLOBAL`並等待Slice 6 final
    validation；不得在validation前標成`CANDIDATE_READY`；
16. flat FL也改由Server發布`ROUND_INPUT`並決定epochs，其餘objective、result與
    lifecycle behavior沒有regression；
17. NWDAF保持standard relay boundary，無hierarchy-awarebusiness state；
18. PyMTLF與NWDAF完成required verification；
19. implementation後不中斷完成mandatory review、in-scope remediation與targeted recheck；
20. production changes依repository分開commit並回填implementation record。

後續切分已確認：Slice 6完成hierarchical final validation與publication，Slice 7完成lifecycle
closure與fresh-state restart，Slice 8負責real multi-process E2E。本段取代早期把candidate
finalization、lifecycle closure與E2E合併考量的草案敘述。
