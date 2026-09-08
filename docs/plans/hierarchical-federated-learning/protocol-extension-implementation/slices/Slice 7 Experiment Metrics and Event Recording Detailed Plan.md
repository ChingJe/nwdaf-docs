# Slice 7 — Experiment Metrics and Event Recording Detailed Plan

日期：2026-09-08

狀態：User Review Confirmed／Commit Approval Pending；尚未進入實作

相關文件：

- [Protocol Extension Implementation Plan](../Hierarchical%20NWDAF%20FL%20Protocol%20Extension%20Implementation%20Plan.md)
- [Protocol Extension Implementation Slice Map](../Protocol%20Extension%20Implementation%20Slice%20Map.md)
- [Slice 3 Branch Replacement Detailed Plan](./Slice%203%20Branch%20Replacement%20without%20Retained-result%20Recovery%20Detailed%20Plan.md)
- [Slice 4 Controlled Local Training Workload Detailed Plan](./Slice%204%20Controlled%20Local%20Training%20Workload%20Detailed%20Plan.md)
- [Slice 5 Protocol-driven Hierarchy Integration Detailed Plan](./Slice%205%20Protocol-driven%20Hierarchy%20Integration%20Detailed%20Plan.md)
- [NWDAF Development Policy](../../../../development_policy.md)

---

## 1. Slice 目的

Slice 3 已提供單一 Branch 於 training 中途失效、Root 以剩餘 Branches 繼續
training、替代 Branch 完成 preparation 後重新加入後續 round 的執行能力。現有
real-process runner 可以從一般文字 log 與 terminal state 確認流程跑通，但尚未產生可
直接用於實驗分析的逐輪 learning curve 與明確 lifecycle event record。

本 slice 建立 node-local experiment recording，使正式 testbed 可以回答：

1. 從相同隨機初始化開始，Root global model 的 validation loss 與 accuracy 如何隨
   accepted global rounds 變化；
2. Branch 被停止、Root 偵測 failure、replacement ready，以及替代 Branch 首次重新
   貢獻 result 分別發生在何時；
3. failure window 期間 Root 實際以哪些 Branch results 完成 degraded aggregation；
4. 若 Branch 或 Leaf 配置自己的 local validation dataset，其 local／domain model 的
   validation結果如何保存，供後續離線分析。

此功能只產生原始紀錄，不在 PyMTLF 內建圖表、統計推論或論文 claim。正式實驗可以在
事後收集各節點檔案，自行選擇繪圖與分析方式。

---

## 2. 已確認的現況

### 2.1 Repository baseline

| Repository | 盤點 revision | 本 slice 角色 |
| --- | --- | --- |
| `PyMTLF/` | `90f1f622be9ad7a3f69c34b2d93d40e5211c415f` | local validation、Root／Branch／Leaf observation 與 JSONL owner |
| `nwdaf-resources/` | `da9b848174164c9a956e0b241d5068cd202347b6` | deterministic dataset placement、fault-injection timestamp、record collection 與 real-process evidence |
| `NWDAF/` | `be3fa576a22c6b787c9ce578c79621b486641e6d` | containing NF transport；預設 read-only |
| `nwdaf-docs/` | `a39418cafbdbc639b8686d41e40a41c4b113989c` 加上本 plan diff | plan owner |

實作開始前必須重新確認各 repository HEAD 與 working tree，並保留任何無關的既有
修改。

### 2.2 現有資料與 evaluator

- `LocalImageTrainingDataSettings` 只有 `dataset` 與 `shard_path`；`shard_path` 是 Leaf
  local training input，不是 validation input。
- `FittingRuntimeSettings.validation_ratio` 目前只作用於 UE Communication training-data
  builder，不會自動切分 image shard。
- `ImageDatasetLoader` 已能以相同 typed contract 載入 MNIST／CIFAR-10 `.npz`。
- `ImageClassificationEvaluator` 目前只回傳 accuracy；尚未計算 validation loss。
- `FederatedTrainer.final_loss` 是 local training 最後一個 batch 的 training loss，不能
  當成完整 validation set 的 loss，也不能直接用於 Root learning curve。
- 現有 hierarchical runner 只建立各 Leaf training shard及單一 `held-out.npz`，training
  完成後再呼叫 `tools/evaluate_image_model.py` 計算一次 final accuracy。

### 2.3 現有 production 掛點

- Root 在接受 top-level request 時產生 UUIDv4 `plan_id`，並將它作為整棵 hierarchy
  共用的 `mlCorreId`。
- Root 在進入 round loop 前已持有 initial model；每個 accepted Root round 後已持有
  `ROUND_GLOBAL` aggregate。
- Branch 每完成一次 lower-tier aggregation後已持有 domain `ROUND_GLOBAL` model，最後
  再發布 upper-tier `HIERARCHY_AGGREGATE`。
- Leaf local training 完成後已持有 local model及該 edge 的 `roundInd`。
- Root 已在同一 coordinator 中知道 round selected／successful／failed participant
  identities、round acceptance、failed Branch identity與 replacement Branch identity。
- Branch process 被停止的精確時間只有 test controller 知道；PyMTLF 只能記錄它實際
  偵測 failure 的時間，不能把 fault injection time 推測成自身事件。

---

## 3. 已固定的設計決策

### 3.1 這是 node-local experiment facility

- Dataset path、record directory及是否啟用 local validation只放在各 PyMTLF 的 local
  config。
- 不新增或修改 `Nnwdaf_MLModelTraining` request、PATCH、Notify 或 model artifact
  schema。
- 不經由 Root 將 validation dataset identity／path 下發給其他 nodes。
- 不將 validation結果向上逐級回傳；各 node只保存自己的 raw record。
- 未配置此功能的既有 hierarchical／flat／distributed FL 行為不得改變。

### 3.2 每次 procedure 使用獨立資料夾

每個 PyMTLF node 使用：

```text
<configured-record-directory>/
└── <mlCorreId>/
    └── observations.jsonl
```

`mlCorreId` 直接使用 Root 已產生並逐級下發的 UUIDv4，不新增 `runId`、plan version或
其他只為命名存在的識別碼。每筆 JSON record 仍保存 `mlCorreId` 與本地
`nfInstanceId`，使檔案離開原目錄後仍能辨識 procedure 與 producer。
建立directory前必須重用現有UUIDv4 normalization，不能把未驗證的wire string直接
拼接成filesystem path。

Record directory 不得放在會由 `FLWorkspace.release_plan()` 或 generation reset 刪除的
`workspace_root` 下。Terminal cleanup只清理training workspace與暫存artifact，不刪除
experiment record。Record retention與事後刪除由實驗執行者處理，本 slice不新增自動
retention policy。

每個PyMTLF process必須配置自己的node-private record directory。Process-local lock只
保護同一process的writers，不把多個PyMTLF指向同一個`observations.jsonl`。

### 3.3 JSONL 是唯一 structured evidence

- 每筆 observation 是一行完整 JSON object。
- Recorder 以 process-local lock確保同一 PyMTLF 內的 concurrent callbacks不交錯寫入。
- 每筆資料以單次 append寫入並 flush Python buffer；不要求每筆 `fsync`。
- 一般文字 log繼續供診斷使用，但 loss／accuracy、round cohort與replacement timing
  不再靠 regex解析文字 log取得。
- 不新增 dataset、config或record hash。既有 artifact repository key不因此改變。

### 3.4 Validation語意

- `validationLoss` 是整份 configured validation dataset 上的 mean cross-entropy loss。
- `validationAccuracy` 是 correct predictions除以 validation sample count，範圍為
  `[0, 1]`。
- Evaluator以 `model.eval()`與 `torch.no_grad()`執行，不更新 model state。
- Validation dataset必須與 received model bundle的image dataset contract相容。
- Local training的最後一個 batch loss若日後需要，可以另以 training record保存；本
  slice不把它混入 validation curve，也不以它替代 `validationLoss`。
- Validation只用於觀測，不作為 aggregation、participant selection、Branch
  replacement或training completion gate。

### 3.5 各角色的必要程度

- Root：本實驗必要。記錄 initial model，以及每個 accepted Root global aggregate的
  validation loss／accuracy。
- Branch：local config有 validation block時啟用。在每個 accepted lower-tier domain
  aggregate後記錄；沒有配置時不評估。
- Leaf：local config有 validation block時啟用。在 local model完成後記錄；沒有配置時
  不評估。

主要Branch failure／replacement實驗預設只啟用Root validation，避免Branch／Leaf額外
evaluation時間進入其response deadline而改變failure window。Branch／Leaf validation是
刻意開啟的diagnostic profile；比較兩次run時必須使用相同的local validation設定。

同一套 recorder／evaluator可被不同角色使用，但每個 observation必須用
`evaluationStage` 明確表示被評估的 model 在該 node execution中的位置，不能以固定
NF role推測。

### 3.6 Event的authoritative producer

- `nwdaf-resources` test controller：只有它可以記錄實際停止 Branch processes 的時間。
- Root PyMTLF：記錄 Root何時由 round outcome偵測到 Branch failure，以及何時完成
  replacement preparation。
- Root round outcome：記錄每次 attempt實際 selected、successful與failed direct
  participants；replacement第一次重新貢獻不新增另一個猜測事件，而是由第一筆包含
  replacement `nfInstanceId` 的successful outcome直接判定。

不新增沒有現行 topology／runtime來源的 `branchGroup`、area label或其他 convenience
identifier。Replacement關係直接使用 failed與replacement Branch的 `nfInstanceId`。

---

## 4. Local configuration

Candidate local config如下：

```yaml
federated_learning:
  experiment_recording:
    directory: "/var/lib/pymtlf/experiment-records"
    validation:
      dataset: "mnist"
      path: "/datasets/validation.npz"
      device: "cpu"
      batch_size: 128
```

欄位語意：

| 欄位 | 必要性 | 語意 |
| --- | --- | --- |
| `experiment_recording` | optional | 整個block不存在時，不建立實驗紀錄 |
| `directory` | block內required | 本node持久保存各 `mlCorreId` 資料夾的根目錄 |
| `validation` | optional | 存在時啟用本node model validation；Root實驗profile必須配置 |
| `dataset` | validation內required | `mnist`或`cifar10`，並與model bundle contract交叉驗證 |
| `path` | validation內required | 本node預先配置的read-only validation `.npz` |
| `device` | optional，預設`cpu` | 執行validation的PyTorch device |
| `batch_size` | optional，預設`128` | validation inference batch size；不改變training batch size |

Relative `directory`與`path`依主config位置解析成absolute path，沿用現有 local path
handling。`validation_ratio`不參與image dataset切分；training與validation檔案必須在
deployment前準備好。

若後續仍要保留final held-out test evaluation，該test file必須和每輪使用的validation
file分離；不能在每輪反覆觀察同一份資料後仍將它描述為未使用過的final test set。正式
train／validation／test partition由dataset工作決定，不在本slice硬編比例。

---

## 5. Record contract

### 5.1 所有 PyMTLF records的共同欄位

| 欄位 | 來源 | 語意 |
| --- | --- | --- |
| `recordedAt` | model ready或state transition成立時取得的UTC時間 | 對齊learning curve與lifecycle event，不以稍後完成file append的時間取代 |
| `recordType` | 產生該record的production hook | 決定其後續欄位與語意 |
| `mlCorreId` | active Model Training resource／Root initiation | 此record所屬hierarchical FL procedure |
| `nfInstanceId` | containing NWDAF context | 產生此record的NWDAF instance |

`recordedAt` 使用含timezone的RFC 3339表示。正式multi-host testbed必須使用NTP／chrony
同步各host clock；檔案行順序只保證單一node內的發生順序，不能取代跨host clock
alignment。

### 5.2 `MODEL_EVALUATION`

額外欄位：

| 欄位 | 語意 |
| --- | --- |
| `evaluationStage` | `ROOT_INITIAL`、`ROOT_GLOBAL`、`BRANCH_DOMAIN`或`LEAF_LOCAL` |
| `roundInd` | 產生該model的local process round；`ROOT_INITIAL`不提供 |
| `dataset` | 實際使用的configured validation dataset |
| `sampleCount` | 此次evaluation涵蓋的validation samples |
| `validationLoss` | 完整validation set的mean cross-entropy |
| `validationAccuracy` | 完整validation set的classification accuracy |

Root accepted round範例：

```json
{"recordedAt":"2026-09-08T10:30:12.345Z","recordType":"MODEL_EVALUATION","mlCorreId":"550e8400-e29b-41d4-a716-446655440000","nfInstanceId":"10000000-0000-4000-8000-000000000001","evaluationStage":"ROOT_GLOBAL","roundInd":3,"dataset":"mnist","sampleCount":128,"validationLoss":0.812,"validationAccuracy":0.741}
```

### 5.3 `ROOT_ROUND_OUTCOME`

Root在每個round attempt結束後記錄，包含accepted與rejected attempts：

| 欄位 | 來源／語意 |
| --- | --- |
| `roundInd` | Root此次dispatch使用的attempt round |
| `accepted` | Root completion policy是否接受此attempt |
| `selectedNfInstanceIds` | 此attempt實際selected direct Branches |
| `successfulNfInstanceIds` | 在deadline內提供有效result的selected Branches |
| `failedNfInstanceIds` | 此attempt被判定availability failure的selected Branches |

這筆record用來區分normal、degraded與replacement恢復後的round。Rejected attempt沒有新
global model，因此不得產生對應的`ROOT_GLOBAL` evaluation record。

### 5.4 Branch lifecycle records

Root只記錄有直接production evidence的兩個事件：

- `BRANCH_FAILURE_DETECTED`
  - `roundInd`
  - `failedBranchNfInstanceId`
- `BRANCH_REPLACEMENT_READY`
  - `failedBranchNfInstanceId`
  - `replacementBranchNfInstanceId`

`BRANCH_REPLACEMENT_READY`代表新Branch已完成fresh resolve、subscription與downstream
preparation，且已成為該topology assignment的active Branch；不代表它已在某個Root
round成功回傳。第一次成功貢獻以`ROOT_ROUND_OUTCOME.successfulNfInstanceIds`為準。

### 5.5 Controller record

Test controller另外建立`controller-events.jsonl`。它不是NWDAF，不填
`nfInstanceId`。目前只需要：

```json
{"recordedAt":"2026-09-08T10:31:00.000Z","recordType":"BRANCH_PROCESS_STOPPED","mlCorreId":"550e8400-e29b-41d4-a716-446655440000","targetNfInstanceId":"10000000-0000-4000-8000-000000000011"}
```

此時間必須在controller確認目標NWDAF與PyMTLF processes均已停止後記錄，不能只表示
「準備送出stop command」。

---

## 6. End-to-end flow

### 6.1 啟動與資料準備

1. Dataset工作預先建立Leaf training files與各node所需的validation files。
2. Deployment將每個node自己的paths寫入local PyMTLF config；不透過protocol傳遞。
3. PyMTLF startup解析experiment block、確認record directory可建立，並以
   `ImageDatasetLoader`載入／驗證configured validation file。
4. Root收到manual training request後產生UUIDv4 `mlCorreId`，recorder於本地建立對應
   procedure directory。

### 6.2 Normal round recording

1. Root載入initial random model後，以Root local validation dataset產生
   `ROOT_INITIAL` record。
2. Root每次收到round outcome時先產生`ROOT_ROUND_OUTCOME`。
3. Outcome accepted時，Root先保存model-ready timestamp，再對剛形成的global
   aggregate執行validation並產生`ROOT_GLOBAL` record；rejected時不產生model
   evaluation。
4. Branch若有local validation config，則在每個accepted lower-tier aggregate完成後產生
   `BRANCH_DOMAIN` record。
5. Leaf若有local validation config，則在local training完成且model publish前產生
   `LEAF_LOCAL` record。

### 6.3 Branch failure與replacement

1. Test controller停止指定Branch的NWDAF與PyMTLF processes，確認退出後記錄
   `BRANCH_PROCESS_STOPPED`。
2. Root round收到failure outcome時記錄`ROOT_ROUND_OUTCOME`及
   `BRANCH_FAILURE_DETECTED`。
3. Root若依policy接受該degraded attempt，仍對degraded aggregate產生`ROOT_GLOBAL`
   validation record。
4. Replacement preparation完成後，Root記錄`BRANCH_REPLACEMENT_READY`。
5. Replacement首次出現在後續accepted outcome的`successfulNfInstanceIds`時，即為它
   實際恢復貢獻的時間點。

### 6.4 收集

Canonical local runner以terminal response中的`planId`取得同一`mlCorreId`，將records
整理為：

```text
<scenario-evidence>/
└── <mlCorreId>/
    ├── nodes/
    │   ├── <root-nfInstanceId>/observations.jsonl
    │   ├── <branch-nfInstanceId>/observations.jsonl
    │   └── <leaf-nfInstanceId>/observations.jsonl
    └── controller-events.jsonl
```

收集時保留原始JSON lines，不改寫timestamp、round或metric。正式multi-host testbed可
使用相同layout，但檔案搬運方式由testbed deployment決定。

---

## 7. Canonical baseline disposition

| Hierarchical baseline階段 | Slice 7處置 |
| --- | --- |
| Manual trigger與UUID `mlCorreId` | 原樣重用；不新增experiment ID |
| Model-free preparation與topology readiness | 原樣重用；不在preparation執行model validation |
| Root ADRF global-model distribution | 原樣重用 |
| Leaf local training | Adapted：model完成後可做non-gating local validation與recording |
| Branch lower-tier aggregation | Adapted：accepted domain aggregate後可做non-gating validation與recording |
| Root round selection／completion | Adapted：將既有outcome寫成structured record，不改policy decision |
| Root global aggregation | Adapted：initial及每個accepted aggregate執行non-gating validation |
| Branch replacement | Adapted：在既有confirmed state transitions產生failure／ready records |
| Final handoff／publication | Reused without semantic change；最後一筆Root evaluation即為final curve point |
| Resource／ADRF／workspace cleanup | Reused；experiment record獨立保存，不隨workspace清除 |
| Shutdown／generation reset | Existing training cleanup維持不變；已flush records保留 |

本slice不重新引入已移除的model-bundle topology metadata，也不建立平行trainer、
aggregator或replacement state machine。

---

## 8. Repository與檔案實作計畫

### 8.1 `PyMTLF/`

| File／area | 計畫變更 |
| --- | --- |
| `src/py_mtlf/config.py` | 新增optional experiment recording與validation settings；解析relative record／dataset paths並驗證不與ephemeral workspace重疊 |
| `src/py_mtlf/core/image_classification.py` | 擴充evaluator，以完整dataset同時計算mean cross-entropy與accuracy |
| `src/py_mtlf/core/experiment_recording.py` | 新增single-owner recorder、typed record constructors、procedure directory與locked JSONL append |
| `src/py_mtlf/app.py` | 建立／open recorder與optional local validator，注入Root、Branch、Leaf owners並在shutdown關閉 |
| `src/py_mtlf/core/fl_root.py` | 記錄initial／accepted global evaluation、Root round outcome、Branch failure detected及replacement ready |
| `src/py_mtlf/core/fl_branch.py` | 在每個accepted lower-tier aggregate後記錄optional domain validation |
| `src/py_mtlf/core/fl_client.py` | 在Leaf local model完成後記錄optional local validation；不使用training final batch loss取代validation |
| `config/fl-server-hierarchy.yaml`與relevant README | 補上experiment recording示例及local-only語意 |

`experiment_recording.py` 是 backend-owned local experiment facility，不是external SBI、
standard-shaped private API或protocol module。

### 8.2 `nwdaf-resources/`

| File／area | 計畫變更 |
| --- | --- |
| `deployments/hierarchical_fl/scripts/support.py` | Canonical config builder為各node設定獨立record directory與optional validation path |
| `deployments/hierarchical_fl/scripts/run.py` | 預先配置training／validation files、記錄controller stop event、按terminal `planId`收集node JSONL，並以structured records驗證normal／degraded／restored windows |
| `deployments/hierarchical_fl/checks/test_support.py` | 驗證generated configs、record paths與collection layout |
| `deployments/hierarchical_fl/README.md` | 記錄raw evidence位置與每種record的用途 |

現有`tools/evaluate_image_model.py`可保留為獨立offline utility，但canonical hierarchical
runner不再以它的一次性final accuracy作為唯一learning evidence。Runner也不再以一般
文字log解析Root round cohort作為主要證據。

### 8.3 預設不修改

- `NWDAF/`：不新增experiment API、欄位或Go-side metrics owner。
- `adrf/`：不改model storage／authorization／cleanup。
- `nrf/`：不改discovery或profile。
- Protocol candidate OpenAPI與conformance matrix：本slice沒有wire contract change。

---

## 9. Failure與lifecycle規則

- 配置experiment recording時，invalid path、unreadable validation file、dataset shape／
  label mismatch或unsafe record location在startup／preflight明確失敗，避免training開始後
  才發現整批資料不可用。
- Model bundle與configured validation dataset不相容時，該node action失敗並留下明確
  error log；不得改用另一份dataset或跳過該required observation。
- Recorder建立procedure directory或append失敗時，configured experiment run視為失敗；
  不以一般log假裝補足structured evidence。
- 已成功append並flush的records不因Branch process被fail-stop而刪除。
- 同一node若因process restart再次看到相同`mlCorreId`，只append既有檔案，不truncate；
  本slice不宣稱Root restart recovery。
- Replacement Branch使用同一`mlCorreId`，但寫入自己的local record directory；不覆寫
  failed Branch既有檔案。

---

## 10. 驗證計畫

### 10.1 `PyMTLF/` focused tests

- Config block absent時不啟用recorder，既有profiles維持通過。
- Relative directory／validation path正確依config解析；unsafe workspace overlap與invalid
  validation config被拒絕。
- Known logits dataset可直接驗證mean cross-entropy、accuracy與sample count，不只斷言
  數值finite。
- Recorder在同一`mlCorreId`下append多筆valid JSON lines，existing file不被truncate，
  concurrent writes不產生破損line。
- Root production round path產生一筆initial evaluation、每個attempt一筆outcome、每個
  accepted aggregate一筆evaluation；rejected attempt沒有evaluation。
- Root failure與replacement production path記錄真實failed／replacement
  `nfInstanceId`，不產生`branchGroup`等無來源欄位。
- Branch production aggregation path只在accepted domain aggregate後記錄；Leaf
  production training path只在local model完成後記錄。
- Validation前後model weights相同，證明observation不更新training state。
- Existing hierarchical replacement、flat／distributed FL focused regressions維持通過。

測試必須執行真正的recorder與image evaluator；不能mock掉JSONL writer或直接塞入預組
metric result來宣稱production observation path已驗證。

### 10.2 `nwdaf-resources/` checks

- Dataset helper為每個configured node提供存在且可載入的validation file，並保持Leaf
  training file與validation file分離。
- Config builder為每個node產生獨立local record directory。
- Normal profile收集到Root initial evaluation及所有accepted global round evaluations。
- Branch-replacement profile同時收集controller stop、Root failure detected、replacement
  ready與round outcomes。
- Evidence可直接指出至少一個primary Branch成功round、至少一個degraded accepted round，
  以及至少一個replacement Branch成功round。
- 所有Root loss／accuracy為finite，accuracy位於`[0, 1]`，recorded round identity與
  terminal `completedRounds`一致。
- Successful teardown後仍可讀取records，ADRF terminal record count維持既有零殘留
  assertion。

### 10.3 Full verification

實作與focused review完成後，依當時repository-native command執行：

```text
PyMTLF: full pytest、ruff、compileall
nwdaf-resources: hierarchical checks與canonical real-process normal／branch-replacement profiles
NWDAF: 若無production diff，只執行既有required regression，不建立commit
```

正式multi-host testbed仍是implementation完成後的external integration verification，
不能以local runner代替。

---

## 11. 驗收條件

- 每次Root training request皆以該次UUIDv4 `mlCorreId`建立獨立record directory。
- Root record至少包含initial model及每個accepted global aggregate的validation loss與
  accuracy。
- Root每個round attempt皆有structured cohort outcome，能辨識normal、degraded及
  replacement恢復後的training。
- Fault injection、Root detection與replacement ready的時間分別由正確owner記錄。
- 所有record fields都有已定義producer與analysis用途，不存在方便分組但沒有runtime
  來源的欄位。
- Branch／Leaf local validation可由各自local config獨立開啟，不需要protocol變更。
- Structured recording不改變model、aggregation、policy、replacement及cleanup semantics。
- Canonical real-process runner能收集raw records並驗證內容，不依賴一般文字log或單次
  external final-accuracy command作主要證據。
- Implementation完成後仍保持unstaged供user review；正式testbed未完成前，整體phase
  仍標為external validation pending。

---

## 12. 明確非目標

- Flat vs hierarchical對照實驗。
- Communication bytes、message count、CPU、memory或network header instrumentation。
- AnLF degradation／recovery、UPF replay或traffic WAPE curve。
- Validation結果透過Model Training Notify逐級回傳。
- 以validation結果影響participant selection、aggregation acceptance或Branch replacement。
- 自動繪圖、dashboard、Prometheus exporter或集中式metrics service。
- Dataset partition research、IID／non-IID matrix或正式train／validation／test比例決策。
- 為record、dataset或config新增hash／manifest。
- Retained-result recovery、Leaf replacement、multi-Branch simultaneous recovery或Root
  restart recovery。

---

## 13. 實作順序

1. `PyMTLF` config、validation loss evaluator與recorder primitives。
2. Root initial／round／replacement hooks與focused tests。
3. Branch／Leaf optional validation hooks與focused tests。
4. `nwdaf-resources` dataset／config／controller-event／collection integration。
5. Canonical local normal及Branch-replacement real-process verification。
6. Initial review、test-first remediation、full verification及user review handoff。

本文件通過user review前不進入implementation；review確認也不等同commit approval。
