# Slice 6 Hierarchical Final Validation and Publication Detailed Plan

日期：2026-08-20

狀態：Plan ready；flat FL baseline、HFL production gap、target data flow 與 evidence
contract 已確認，可進入 production implementation

上層計畫：

- [Hierarchical NWDAF Federated Learning Implementation Plan](./Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)

前置 slices：

- [Slice 4 End-to-end Preparation and Admission Detailed Plan](./Slice%204%20End-to-end%20Preparation%20and%20Admission%20Detailed%20Plan.md)
- [Slice 5 Hierarchical Rounds and Aggregation Detailed Plan](./Slice%205%20Hierarchical%20Rounds%20and%20Aggregation%20Detailed%20Plan.md)

後續 slice：

- Slice 7 lifecycle closure and fresh-state restart；待 Slice 6 implementation、code review
  與針對性修正完成後再檢視與定案，現有草稿不是本 Slice 的 normative input。

---

## 1. 目的與修正範圍

Slice 5 已完成 Root–Branch–Leaf multi-round training，但把最後 configured Root
`ROUND_GLOBAL` 形成後的狀態直接標成 `CANDIDATE_READY`。這與現有 flat FL production
flow 不一致：flat FL 只有在 final validation 完成且 validation gate 接受後才進入
`CANDIDATE_READY`。

本 Slice 修正這項 semantic defect，並將已驗證 candidate 接回既有 durable publication
flow：

```text
final Root ROUND_GLOBAL
  -> FINAL_VALIDATION_DISPATCH
  -> FINAL_VALIDATION_WAITING
  -> FINAL_VALIDATION_EVALUATING
  -> VALIDATION_REJECTED
     or
  -> CANDIDATE_READY
  -> PUBLISHING
  -> CUTOVER_PENDING / COMPLETE
```

本 Slice 完成時：

1. 最後 Root `ROUND_GLOBAL` 是待驗證 candidate，不是已 ready candidate；
2. candidate URL 有明確 recipient：Root 給 assigned Branch，Branch 下載、驗證並重新發布
   Branch-owned URL 給 assigned Leaves；
3. Leaves 使用 preparation 時凍結的 dataset 與 base model 執行 validation-only job；
4. Branch 收集並驗證所有 assigned Leaf evidence，再以 Branch upper validation result 回報
   Root；
5. Root 驗證完整 topology evidence、計算 global gate，只有接受後才進入
   `CANDIDATE_READY`；
6. accepted candidate 由既有 `PublicationCoordinator` 建立 `FINAL_MODEL`、store ADRF、commit
   catalog 並進入既有 cutover；
7. Training resource 的 terminal cleanup 交由 Slice 7，且不得在 final validation 前刪除。

---

## 2. Existing-flow extension baseline map

Canonical baseline 是目前 `PyMTLF/src/py_mtlf/core/fl_server.py` 的 flat FL production
flow，而不是舊計畫中把 `CANDIDATE_READY` 放在 validation 前的歷史文字。

| Baseline stage | Flat FL production meaning | HFL disposition |
| --- | --- | --- |
| trigger／single-active admission | degradation intent 建立一個 active retrain | Slice 3 已調整為 Root plan；直接沿用 single-active invariant |
| preparation | Server 建立 participant Training resources，Client freeze dataset／base | Slice 4 已調整為 Root–Branch–Leaf preparation |
| training rounds | Server dispatch `ROUND_INPUT`，Clients回 `ROUND_LOCAL`，Server聚合 `ROUND_GLOBAL` | Slice 5 已調整成 lower／upper rounds |
| final aggregate | 最後 `ROUND_GLOBAL` 成為未驗證 candidate | 直接沿用語意；撤銷 Slice 5 對 `CANDIDATE_READY` 的提早使用 |
| validation dispatch | 相同 Training resources 收到 validation-only PATCH | Root 對 Branches dispatch；Branches 對 Leaves dispatch |
| candidate transfer | Server-published candidate URL 供 selected Clients下載 | Root URL只給Branches；Branches republish後由Leaves下載 |
| validation execution | Clients比較 frozen base與candidate，不 fitting | Leaves直接沿用；Branch改為subtree coordinator，不執行Branch-local validation |
| validation evidence | 每個direct participant回 `ROUND_LOCAL/ACCURACY_CHECK` | Leaves回 lower evidence；Branch聚合並附完整 subordinate evidence後回 upper result |
| validation gate | Server驗證evidence並計算aggregate/per-scope gate | Root驗證exact Branch／Leaf topology並對所有Leaf evidence計算gate |
| `CANDIDATE_READY` | validation accepted、可交給publication | 完整沿用，不得再表示「rounds complete」 |
| publication | reserve model ID、建立`FINAL_MODEL`、ADRF store、catalog commit | 重用既有`PublicationCoordinator`，只增加hierarchy provenance input |
| cutover | desired provision／monitor adoption完成後`COMPLETE` | 重用既有publication與policy owner；HFL status投影相同lifecycle |
| terminal cleanup | validation／publication outcome後刪Training resources並釋放owner | 調整到Slice 7，不得在candidate形成時提前執行 |
| failure | technical failure為`FAILED`；enforced gate rejection不publish | HFL任一required Branch／Leaf failure使完整experiment terminal，不partial validation |

### 2.1 Shared semantic invariants

- `CANDIDATE_READY` 的必要前置條件永遠是 final validation evidence 完整且 gate accepted；
- `ROUND_GLOBAL`、`ROUND_LOCAL`、`FINAL_MODEL` 的既有 artifact role 意義不變；
- candidate 是 final Root `ROUND_GLOBAL` 在 lifecycle 中的位置，不是新 artifact role；
- `PUBLISHING` 才表示 durable promotion 已開始；
- `COMPLETE` 才表示 publication 與必要 cutover 完成；
- `VALIDATION_REJECTED` 不建立 `FINAL_MODEL`，old latest 不變；
- passing tests 不得取代 baseline-to-plan 與 plan-to-implementation conformance。

---

## 3. Current production evidence 與 gap

### 3.1 Flat FL 已有完整 finalization owner

`FLServerEngine` 已有：

- `FINAL_VALIDATION_DISPATCH`、`FINAL_VALIDATION_WAITING`、
  `FINAL_VALIDATION_EVALUATING`；
- validation-only PATCH：`mLAccChkFlg: true`、`skipFlInd: true`、下一個
  `roundInd`、一個 `mLModelInfos` candidate URL；
- Client frozen dataset／base contract validation；
- `ROUND_LOCAL/result_type=ACCURACY_CHECK` evidence；
- component-sum WAPE evaluation與optional enforcement；
- `CANDIDATE_READY -> PUBLISHING`；
- `PublicationCoordinator.publish(ValidatedCandidate)`；
- durable model ID、`FINAL_MODEL`、ADRF、catalog、Provision notification與cutover。

這些 owner 應抽取或擴充成 hierarchy-capable reusable operations，不在 Root coordinator
另寫一套 parallel publication implementation。

### 3.2 HFL Root 提早使用 `CANDIDATE_READY`

目前 `FLRootCoordinator._run()` 在最後 configured round後：

1. 將 aggregate URL／digest寫入Root record；
2. 將Server process與Root request直接設為`CANDIDATE_READY`；
3. 沒有dispatch final validation；
4. 沒有建立`ValidatedCandidate`；
5. 沒有呼叫`PublicationCoordinator`。

因此目前的 `candidateUrl` 沒有 training procedure recipient，只能由 private status caller
看到。這不是有效的candidate lifecycle。

### 3.3 Branch validation 尚未 delegation

目前 `FLClientEngine._run_validation()` 對所有 Client resource 都執行 local validation。
Branch resource雖有`BranchAssignmentMetadata`，validation path尚未像round path一樣delegate
至`FLBranchCoordinator`。若直接啟用Root validation PATCH，Branch會錯誤地只驗證Branch
本地dataset，而不會讓assigned Leaves驗證。

### 3.4 Candidate serving route 已存在

`FLWorkspace.publish()` 已產生：

```text
/internal/v1/fl-artifacts/{process_id}/{participant_id}/{round}/{role}/{digest}
```

Artifact route由PyMTLF直接serving。Slice 6不新增Go proxy，也不新增public SBI。缺少的是
recipient與lifecycle，不是URL生成能力。

### 3.5 Publication service 已由 app 建立

`PublicationCoordinator`、ADRF resolver、catalog、durable model state與Provision notifier
已由PyMTLF app建立。HFL Root目前只持有`FLServerEngine`，因此應透過Server-owned reusable
finalization method接入publication，不把publication state複製到Root coordinator。

---

## 4. Target end-to-end flow

### 4.1 Root candidate formation

最後configured training round完成後，Root：

1. 驗證final aggregate仍是目前plan／Server process的`ROUND_GLOBAL`；
2. 驗證model／preprocessing contract與base一致；
3. freeze candidate artifact path、URL、archive digest與weights digest；
4. 不設定`CANDIDATE_READY`；
5. 使用`validation_round = server.round_count`進入
   `FINAL_VALIDATION_DISPATCH`。

Candidate bundle維持：

```json
{
  "artifact_role": "ROUND_GLOBAL",
  "fl_metadata": {
    "round_ind": 1,
    "weights_digest": "..."
  }
}
```

`round_ind`仍是產生它的最後training round；validation command的`roundInd`則使用下一個
保留值。不得改寫candidate bundle來假裝它是validation output。

### 4.2 Root-to-Branch validation command

Root沿用已建立且仍為active的upper Training subscriptions，對每個admitted Branch送：

```json
{
  "mLAccChkFlg": true,
  "skipFlInd": true,
  "roundInd": 2,
  "mLModelInfos": [
    {
      "mLFileAddr": {
        "mLModelUrl": "http://root-pymtlf/internal/v1/fl-artifacts/.../ROUND_GLOBAL/..."
      }
    }
  ],
  "mLTrainRepInfo": {
    "maxResTime": 300
  }
}
```

Root URL只傳給assigned Branch，不放入private management status，也不作為operator model
download contract。

### 4.3 Branch download and republish

Branch收到upper validation command後：

1. 驗證resource仍綁定同一plan、assignment、upper subscription與expected validation round；
2. 下載Root candidate；
3. 驗證archive digest、`ROUND_GLOBAL` contract、model／preprocessing digest與frozen base；
4. 將已驗證candidate以byte-identical artifact重新發布到Branch PyMTLF；
5. Branch-owned URL不同，但archive digest與manifest不變；
6. 將Branch URL放入lower validation PATCH給assigned Leaves。

Byte-identical republish不改寫candidate manifest中的upper `ml_corre_id`。Branch lower Server以
active lower subscription、validation round與expected candidate digest做command correlation；
Leaf validation result則使用lower Training resource的`mlCorreId`。不得為了讓兩層correlation
看起來相同而修改immutable candidate archive。

Branch不得：

- 將Root URL原樣轉交Leaf；
- 修改candidate weights；
- 把candidate重新標成`ROUND_INPUT`或`FINAL_MODEL`；
- 以Branch本地candidate替換Root candidate；
- 由Go relay或proxy artifact bytes。

`FLWorkspace`需新增plan-owned validated-artifact republish primitive，使下載、atomic publish、
resolve、retention與cleanup仍由Branch PyMTLF workspace擁有。

### 4.4 Leaf validation

Leaf沿用flat Client validation-only behavior：

- 使用preparation freeze的dataset snapshot、validation split與base artifact；
- 不執行optimizer、backpropagation或local fitting；
- 比較base與candidate WAPE components；
- candidate weights與input digest保持不變；
- 發布`ROUND_LOCAL/result_type=ACCURACY_CHECK`；
- callback使用既有Training notification path；
- technical failure／timeout回terminal result，完整experiment不得partial accept。

### 4.5 Branch evidence aggregation

Branch lower Server等待所有assigned Leaves成功回報或bounded deadline。成功時：

1. 驗證Leaf IDs與admitted subordinate set完全一致；
2. 驗證每個result是`ROUND_LOCAL/ACCURACY_CHECK`；
3. 驗證lower process、validation round、participant、scope、base／candidate digest；
4. 將base／candidate error sums、actual sums與evaluation sample counts分別加總；
5. 不平均各Leaf WAPE；
6. 發布Branch upper `ROUND_LOCAL/ACCURACY_CHECK`，participant identity是Branch；
7. 附上canonical ordered `subordinate_validation_summaries`。

`RoundLocalAccuracyCheckMetadata`新增optional vendor field：

```json
{
  "subordinate_validation_summaries": [
    {
      "participant_nf_instance_id": "leaf-id",
      "scope_digest": "...",
      "evaluation_sample_count": 100,
      "start_time": "2026-08-20T00:00:00Z",
      "end_time": "2026-08-20T01:00:00Z",
      "base_model_weights_digest": "...",
      "candidate_weights_digest": "...",
      "base": {
        "absolute_error_sum": 10.0,
        "absolute_actual_sum": 100.0
      },
      "candidate": {
        "absolute_error_sum": 8.0,
        "absolute_actual_sum": 100.0
      }
    }
  ]
}
```

Flat local validation result不帶此欄位；Branch hierarchy result必須帶且cover exact admitted
Leaves。Branch-level `evaluation` 必須等於subordinate summaries的component-wise sum，避免
Root只相信不可核對的彙總數字；其`evaluation_sample_count`是Leaf sample counts總和，
`start_time`／`end_time`分別是subordinate windows的minimum／maximum。

### 4.6 Root validation and gate

Root等待所有admitted Branches成功回報或deadline，並驗證：

- direct Branch set與admission一致；
- 每個Branch result identity、round、scope與candidate digest正確；
- 每個Branch的subordinate set與static topology admission一致；
- 所有Leaf只出現一次；
- Branch aggregate components等於其Leaf summaries總和；
- base／candidate model與preprocessing contract一致；
- denominators與sample counts有效。

HFL gate沿用`federated_learning.server.final_validation` config，但將flat的single triggering
participant判斷調整為整棵admitted topology：

- aggregate candidate WAPE必須優於aggregate base WAPE；
- 每個Leaf candidate regression不得超過`max_scope_wape_regression`；
- enforcement disabled時仍完整計算、保存evidence與`gate_would_accept`，但允許技術上有效的
  candidate進入publication；
- enforcement enabled且不接受時進入`VALIDATION_REJECTED`，不建立`FINAL_MODEL`。

### 4.7 Candidate ready and publication

Validation accepted後才依序：

```text
CANDIDATE_READY
  -> PUBLISHING
  -> CUTOVER_PENDING / COMPLETE
```

`CANDIDATE_READY`是明確checkpoint，但當publication owner存在時通常立即轉入
`PUBLISHING`，不是operator approval等待狀態。

Publication必須重用既有`PublicationCoordinator`：

- family與base revision來自Root initiation record／Server hierarchy process；
- direct participants是Branches；
- Branch training sample count是其Leaves的effective sample總和；
- direct validation summaries是Branch aggregate summaries；
- hierarchy provenance另保存canonical Branch-to-Leaf validation evidence；
- reserve正式model ID；
- 建立新的immutable `FINAL_MODEL`，不原地修改candidate；
- store ADRF、commit catalog、啟動Provision／cutover；
- stale base、ADRF retry、catalog commit與cutover沿用既有durable semantics。

`FinalModelMetadata`新增optional hierarchy provenance projection；flat `FINAL_MODEL`不帶此
欄位，HFL `FINAL_MODEL`必須cover exact admitted Branch／Leaf topology。既有
`validation_summary`仍以direct participant（Branch）為單位，維持它與`participants`集合
一致的invariant。

Hierarchy provenance至少使用以下typed shape：

```json
{
  "hierarchy_validation": {
    "plan_id": "...",
    "branches": [
      {
        "branch_nf_instance_id": "...",
        "subordinate_validation_summaries": []
      }
    ]
  }
}
```

Branches與Leaves都使用canonical NF instance ID排序且不得重複；branch集合必須等於direct
`participants`，每個subordinate集合必須等於admission snapshot。此欄位是model bundle內的
同vendor provenance，不修改Release 18 public OpenAPI schema。

已確認第一版不只保存Branch aggregate：每個Leaf的validation summary必須作為
Branch upper result的typed evidence一路回傳Root，並按exact admitted topology寫入
`FINAL_MODEL.hierarchy_validation`。這是durable validation provenance，不只是運行期
diagnostic status。

---

## 5. State and status contract

### 5.1 Root request states

`RootRequestState`補齊：

- `FINAL_VALIDATION_DISPATCH`；
- `FINAL_VALIDATION_WAITING`；
- `FINAL_VALIDATION_EVALUATING`；
- `VALIDATION_REJECTED`；
- `CANDIDATE_READY`；
- `PUBLISHING`；
- `CUTOVER_PENDING`；
- `COMPLETE`；
- `FAILED`。

相同名稱必須與`FLServerState`維持相同進入條件，不得只做近似status projection。

### 5.2 Private management status

Private asynchronous status保留：

- request／plan／family identity；
- state；
- current／completed training rounds；
- candidate archive digest（candidate形成後）；
- failure cause／detail。

移除`candidateUrl`：

- candidate URL是Training procedure transport input；
- operator不透過此status取得未發布模型；
- 正式模型透過既有catalog／Model Provision lifecycle提供；
- status移除URL不刪除artifact serving route。

### 5.3 Failure classification

- missing／invalid candidate、Branch republish failure、Leaf technical failure、timeout、invalid
  evidence與publication terminal error：`FAILED`；
- enforced performance gate rejection：`VALIDATION_REJECTED`；
- gate rejected與technical failure都保留old latest；
- 任一required participant失敗都不得partial validate／publish；
- 不新增automatic validation retry；publication只沿用既有durable retry semantics。

---

## 6. Ownership and repository impact

### 6.1 `PyMTLF/`

預期修改：

- `core/fl_root.py`
  - 修正final aggregate後state；
  - orchestration final validation／publication；
  - status projection與failure classification。
- `core/fl_server.py`
  - 抽取／新增hierarchy-capable final validation與publication operations；
  - 保持publication owner、gate owner與cutover owner單一；
  - validate exact subordinate evidence。
- `core/fl_client.py`
  - Branch-assigned validation delegate至Branch coordinator；
  - Leaf維持existing local validation。
- `core/fl_branch.py`
  - candidate download／republish；
  - lower validation dispatch／wait／aggregate；
  - duplicate、late、wrong-round與cancel fencing。
- `core/fl_artifacts.py`
  - typed subordinate validation evidence；
  - optional HFL `FINAL_MODEL` provenance projection；
  - component-sum與identity validators。
- `core/fl_hierarchy_artifacts.py`
  - Branch validation candidate republish與upper validation result writer。
- `core/fl_workspace.py`
  - byte-identical validated artifact republish；
  - explicit plan ownership供Slice 7 cleanup使用。
- `core/publication.py`
  - 接收optional hierarchy provenance；
  - flat publication behavior不變。
- `wire/hierarchical_fl.py`、`api/hierarchical_fl.py`
  - 補states；
  - 移除`candidateUrl` response field。
- tests
  - Root、Branch、Client、artifact、publication與private API regression。

### 6.2 `NWDAF/`

本Slice不修改Go。Validation仍使用既有Release 18-shaped Training PATCH／notification，Go只
轉送standard-shaped payload與URL，不serving artifact bytes。

### 6.3 `adrf/`、`PyAnLF/`

第一版重用既有publication／provision contracts，不修改ADRF或PyAnLF。若實作證據顯示
existing `FINAL_MODEL` consumer無法忽略optional hierarchy provenance，必須先列為contract
decision，不得在本Slice臨時移除provenance。

### 6.4 `nwdaf-docs/`

- 修正主計畫Slice順序、completion criteria與decision log；
- 在Slice 5保留歷史scope並加semantic erratum；
- 將lifecycle plan順延為Slice 7；
- 將multi-process E2E順延為Slice 8。

---

## 7. Implementation checkpoints

### Checkpoint 1：Baseline characterization and conformance map

- 用tests固定flat final validation／publication state preconditions；
- 用failing test證明HFL目前在validation前進入`CANDIDATE_READY`；
- 建立本文件normative item到production path／test的working map。

### Checkpoint 2：Hierarchy validation artifact contract

- subordinate validation summary schema；
- component-sum validators；
- Branch byte-identical candidate republish；
- flat artifact regression。

### Checkpoint 3：Branch lower validation

- Branch Client delegate；
- candidate download／republish；
- Leaf validation PATCH；
- exact result collection與upper aggregate result；
- failure、timeout、duplicate與late result。

### Checkpoint 4：Root final validation and gate

- Root final validation states；
- exact Branch／Leaf evidence validation；
- aggregate／per-Leaf gate；
- `VALIDATION_REJECTED`與`CANDIDATE_READY` preconditions。

### Checkpoint 5：Publication and status

- reusable Server-owned publication handoff；
- HFL `FINAL_MODEL` lineage／provenance；
- ADRF／catalog／cutover mocked component verification；
- private status移除candidate URL；
- handoff到Slice 7 lifecycle owner。

### Checkpoint 6：Review and regression

- focused tests；
- full PyMTLF lint／pytest；
- mandatory initial code review；
- in-scope findings test-first remediation；
- targeted follow-up review；
- close baseline／plan／implementation conformance map。

---

## 8. Verification matrix

### 8.1 Deterministic focused tests

- final Root aggregate不再直接設定`CANDIDATE_READY`；
- Root validation PATCH使用same Branch subscription與`roundInd=round_count`；
- Branch下載Root candidate並提供不同origin／相同digest的Branch URL；
- Leaf只從Branch URL下載candidate；
- Branch resource不執行Branch-local validation；
- Leaves不進行fitting且candidate weights不變；
- Branch result涵蓋exact admitted Leaves並使用component sums；
- missing、duplicate、wrong Leaf／Branch evidence被拒絕；
- Root gate使用所有Leaf evidence；
- gate disabled仍保存evidence並publish；
- gate enabled rejection進入`VALIDATION_REJECTED`且不publish；
- validation accepted才短暫進入`CANDIDATE_READY`；
- publication建立新的`FINAL_MODEL`與正式model ID；
- `FINAL_MODEL` direct participants、effective sample counts與hierarchy provenance一致；
- stale base不commit latest；
- private status不含`candidateUrl`；
- flat final validation／publication完全regression。

### 8.2 Failure and race tests

- Branch republish失敗前不dispatch Leaves；
- 一個Leaf failure／timeout使完整validation terminal；
- validation callback與parent DELETE race只commit一次terminal outcome；
- late validation result不重建mapping；
- publication開始後duplicate Root status query不重送validation；
- shutdown during validation fences worker publication。

### 8.3 Required commands

Focused：

```bash
.venv/bin/pytest -q tests/test_fl_artifacts.py tests/test_fl_client.py tests/test_fl_branch.py tests/test_fl_root.py tests/test_fl_server.py tests/test_publication.py tests/test_hierarchical_fl_api.py
```

Full：

```bash
.venv/bin/ruff check src tests
.venv/bin/pytest -q
```

本Slice不以unit／mock tests宣稱real ADRF、NRF、OAuth、TLS或multi-process integration；由
Slice 8 E2E關閉。

---

## 9. Explicitly deferred work

- partial validation或minimum-results acceptance；
- dynamic participant replacement；
- alternate candidate origin／retry worker；
- operator download未發布candidate；
- new public candidate API；
- arbitrary-depth recursive validation；
- cross-vendor hierarchy evidence schema；
- production E2E與network partition；
- lifecycle TTL／restart cleanup implementation（Slice 7）。

---

## 10. Review checklist

### Baseline and semantics

- [ ] flat production flow是canonical baseline
- [ ] every baseline stage有explicit disposition
- [ ] `CANDIDATE_READY`只有validation accepted後成立
- [ ] candidate維持`ROUND_GLOBAL`
- [ ] `FINAL_MODEL`是publication建立的新bundle

### Data flow

- [ ] Root candidate URL有唯一明確recipient set
- [ ] Branch download／validate／republish完整
- [ ] Leaf只使用Branch URL
- [ ] Branch與Root都驗證exact subordinate identities
- [ ] evidence producer、carrier、consumer與retention owner完整

### Publication

- [ ] publication logic沒有在Root重複實作
- [ ] effective sample count與direct Branch participant一致
- [ ] hierarchy evidence durable且不弱化flat contract
- [ ] validation rejected不建立正式model ID
- [ ] old latest在ADRF／catalog commit前不變

### Status and lifecycle handoff

- [ ] private status不提供candidate download contract
- [ ] `candidateUrl`已移除
- [ ] Slice 7只在validation／publication允許的stage cleanup
- [ ] no automatic validation retry

### Verification

- [ ] every normative plan item有production／test／approved deferral evidence
- [ ] focused tests通過
- [ ] full ruff與pytest通過
- [ ] initial review與targeted remediation loop完成
- [ ] baseline ↔ plan ↔ implementation conformance closed

---

## 11. Completion criteria

Slice 6 完成必須同時滿足：

1. HFL final aggregate不再直接進入`CANDIDATE_READY`；
2. Root、Branch、Leaf使用既有Training resources完成階層式validation-only flow；
3. Root URL只給Branches，Leaves只使用Branch-republished URL；
4. Branch不做local fitting或Branch-local validation，而是彙整完整Leaf evidence；
5. Root驗證exact topology、digests與component sums；
6. performance gate disabled／enabled behavior與flat config語意一致；
7. validation accepted後才進入`CANDIDATE_READY`；
8. accepted candidate經既有publication owner建立`FINAL_MODEL`並進入ADRF／catalog／cutover；
9. 每個Leaf validation summary經Branch typed evidence回傳Root，並durably記錄在
   `FINAL_MODEL.hierarchy_validation`；
10. enforced gate rejection與technical failure不改變old latest；
11. private status不再暴露candidate URL；
12. flat FL final validation／publication沒有regression；
13. terminal cleanup與restart ownership明確交接Slice 7；
14. plan-conformance map與必要verification全部關閉。
