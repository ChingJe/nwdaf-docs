# Phase 6 本地訓練與更新模型重新提供

日期：2026-07-25

狀態：詳細計畫已撰寫，尚未開始實作

上層計畫：

- `MTLF Backend Transition Plan.md`

前一階段：

- `Phase 5 Dataset Selection And Direct Retrieval.md`

---

## 1. 目的

本階段把 Phase 5 已完成的 `DatasetSnapshot` 接到 PyMTLF 擁有的本地訓練流程，產生新的 immutable
model bundle，並沿用 Phase 4 已建立的標準 Model Provision subscription／notification 路徑，將更新後的模型
重新提供給 PyAnLF。

完成後的主要流程為：

```text
任一 monitoring scope 發生 degradation
    -> PyMTLF 建立 model-level retrain intent
    -> Phase 5 蒐集同一模型所有 active scopes 的資料
    -> READY DatasetSnapshot 被 Phase 6 原子 claim
    -> PyMTLF 建立較早 reference validation／較新 training dataset 並本地訓練 candidate
    -> 分 scope 與整體驗證 candidate
    -> 建立、驗證並發布 immutable model bundle
    -> 更新 PyMTLF current model catalog
    -> 透過既有標準 Model Provision notification 通知 PyAnLF
    -> PyAnLF 下載並完整準備 candidate
    -> 所有相依 runtime 原子切換到新模型
    -> 重設相關 accuracy window，開始回報新模型表現
```

本階段不重新設計初始模型提供、Model Provision resource、accuracy monitor、ADRF/Mongo retrieval 或 Go
backend lifecycle。這些已由 Phase 2 至 Phase 5 建立，本階段只補上「READY dataset 到更新模型正式套用」
之間的缺口。

---

## 2. 核心責任邊界

### 2.1 PyMTLF

PyMTLF 擁有：

1. 原子 claim READY dataset，避免相同 snapshot 被重複訓練。
2. raw ADRF-aligned record 到訓練 sample 的轉換。
3. chronological reference-validation/training split。
4. local trainer、candidate evaluation 與 acceptance policy。
5. model generation、bundle packaging、digest 驗證與 immutable artifact publication。
6. current model catalog 與同一模型的 single retrain-in-flight。
7. Model Provision update notification 的 desired state 與 retry reconciliation。
8. terminal training outcome 後釋放 Phase 4/5 的 retrain-in-flight。

### 2.2 PyAnLF

PyAnLF 擁有：

1. 接收標準 Model Provision notification。
2. 直接由 `mLFileAddr.mLModelUrl` 下載 candidate bundle。
3. bundle 安全檢查、digest 驗證、模型與 scaler 完整載入。
4. 找出所有使用同一 model identity 的 runtime。
5. candidate-first、all-or-nothing 的 atomic runtime swap。
6. candidate 準備或切換失敗時保留舊模型。
7. 更新等待期間的 accuracy report gate，以及成功切換後的 prediction/accuracy window reset。

### 2.3 Go NWDAF

Go 僅保留標準 SBI 與 backend routing 責任：

1. 驗證、mirror 並轉送既有 Model Provision standard-shaped payload。
2. 維護 Model Provision subscription callback route。
3. 依 Release 18 OpenAPI 定義處理 method、status、header 與 `ProblemDetails`。
4. backend 不可用時依既有 lifecycle 與 standard API response matrix 回應。

Go 不會：

- 接收或轉送 dataset bytes。
- 執行 feature extraction、training、validation 或 generation 決策。
- 建立另一套 model coordinator。
- 下載、儲存或代理 model artifact。
- 解讀 private generation 欄位。

### 2.4 nwdaf-resources 與 nwdaf-docs

- `nwdaf-resources/`：放置跨 Go、PyAnLF、PyMTLF process 的完整測試 harness 與操作文件。
- `nwdaf-docs/`：保存 canonical design、規格依據、決策、review ledger 與完成紀錄。

---

## 3. 規格依據與邊界

### 3.1 Degradation 後由 MTLF 重新訓練並提供更新模型

TS 23.288 clause 6.2E.3.3 step 9 描述：MTLF 判定模型需要重新訓練時，可取得必要資料、重新訓練或選擇
更新後模型，接著透過 ML Model Provision Notify 將更新模型提供給 AnLF。

因此本階段採用：

```text
Phase 4 accuracy degradation decision
    -> Phase 5 retraining data retrieval
    -> Phase 6 local retraining
    -> existing Model Provision notification
```

這不是 private update protocol，也不需要 Go 另外協調 training。

### 3.2 本階段不實作 Nnwdaf_MLModelTraining

TS 23.288 clause 6.2F 與 TS 29.520 clause 4.6 的 `Nnwdaf_MLModelTraining` 用於 MTLF 向另一個具備 training
能力的 MTLF 請求訓練，以及 delegated／federated learning 類型的跨 NWDAF procedure。

目前 training 在同一個 PyMTLF backend 內完成，不存在另一個標準 MTLF service producer。因此：

- 不新增 `Nnwdaf_MLModelTraining` API。
- 不讓 PyMTLF 成為獨立標準 NF。
- future multiple-NWDAF FL 另行設計，不混入本階段。

### 3.3 更新模型沿用標準 Model Provision notification

TS 23.288 clause 6.2A.1、6.2A.2 與 TS 29.520 clause 4.5.2.4.2 定義 Model Provision
subscription/notification。Release 18 OpenAPI
`TS29520_Nnwdaf_MLModelProvision.yaml` 定義 callback body 為至少一筆
`NwdafMLModelProvNotif` 的 JSON array。每筆 notification 必須有：

- `subscriptionId`
- 非空 `eventNotifs`

每筆 `MLEventNotif` 必須有 `event`，並至少有 `mLFileAddr` 或 `mLModelAdrf` 其中之一；本階段使用
`mLFileAddr.mLModelUrl`。它也可攜帶：

- `notifCorreId`
- `modelUniqueId`
- 選填的 applicability／additional model information

本階段沿用 Phase 4 已存在的 subscription resource 與 callback，不另建 update endpoint。

### 3.4 generation 只屬於 PyMTLF 內部狀態

TS 23.288 的程序文字提到 optional ML Model provide indicator，但目前 workspace 採用的
Release 18 V18.13 `TS29520_Nnwdaf_MLModelProvision.yaml` 並沒有 `modelUpdateInd` 欄位。現有 schema
具有 `modelUniqueId`、`mLFileAddr`、`mLModelAdrf`、`addModelInfo` 與 `mlDegradInd`，但
`mlDegradInd` 不是 model update flag。

因此本階段：

- 不自行發明或傳送 `modelUpdateInd`。
- 不把 private generation 放入標準 payload。
- 使用相同穩定 `modelUniqueId` 加上新的 immutable `mLModelUrl` 表示同一模型的新 artifact。
- generation 只用於 PyMTLF process 內的順序、stale check、catalog promotion 與 retry coalescing。

### 3.5 HTTP 行為

外部 SBI method、success status、error status、Location 與 `ProblemDetails` 全部沿用 Phase 4 已按
`TS29520_Nnwdaf_MLModelProvision.yaml` 建立的 response matrix。本階段不得因 local training convenience
增加或改變標準 HTTP 行為。

本階段實際新增行為只會重複使用 Model Provision Notify：

| 行為 | Method／URI | 成功 | OpenAPI 宣告的其他 response |
|---|---|---|---|
| 通知更新模型 | `POST {notifUri}` | `204 No Content` | `307, 308, 400, 401, 403, 404, 411, 413, 415, 429, 500, 502, 503, default` |

TS 29.520 clause 4.5.2.4.2 要求 consumer 成功處理並保存 notification 後回 `204`。Malformed JSON／mandatory
field failure 回 `400`，unsupported media type 回 `415`，body limit 回 `413`，不存在或已刪除的
subscription correlation 回 `404`；configured backend unavailable 回 `503 ProblemDetails`，backend
transport 成功但回傳不合法 response 則由 Go 依既有 Phase 4 contract 映射為 `502 ProblemDetails`。不得以
`202`、private success body 或 training-specific status code 取代。

PyAnLF callback 在標準 notification 通過同步 schema/domain validation、保存 resource 最新 desired
notification 並成功排入 bounded worker 後回 `204 No Content`。`204` 只表示 notification 已被接受，
不表示 candidate 已完成下載或已成為 active model。

### 3.6 本地規格來源

- `specs/TS 23.288/6 Procedures to Support Network Data Analytics/6.2A Procedure for ML Model Provisioning.md`
- `specs/TS 23.288/6 Procedures to Support Network Data Analytics/6.2E MTLF-based ML Model Accuracy Monitoring.md`
- `specs/TS 23.288/6 Procedures to Support Network Data Analytics/6.2F Procedure for ML Model Training.md`
- `specs/TS 29.520/4 Services offered by the NWDAF/4.5 Nnwdaf_MLModelProvision Service.md`
- `specs/TS 29.520/4 Services offered by the NWDAF/4.6 Nnwdaf_MLModelTraining Service.md`
- `specs/openapi/TS29520_Nnwdaf_MLModelProvision.yaml`
- `specs/openapi/TS29520_Nnwdaf_MLModelTraining.yaml`

---

## 4. 與現有實作的銜接

### 4.1 Phase 4 已有能力

Phase 4 已完成：

- PyAnLF 在沒有 compatible model 時建立 Model Provision demand。
- Go 路由標準 Model Provision subscription/notification。
- PyMTLF 擁有 provision resource 與 configured seed model。
- PyAnLF 直接下載、驗證、載入與綁定 seed bundle。
- 相同 canonical model context 共用模型，不同 group/filter/target 建立獨立 monitoring scope。
- PyAnLF 以 `MLModelAccuracyInfo.deviation` 回報 WAPE。
- PyMTLF 以 degradation-only policy 產生 model-level retrain intent。

本階段不得重建上述 resource 或改變既有 WAPE/degradation 邏輯，除非是為了接上「模型更新等待」與
「成功切換後重設 window」而做的必要延伸。

### 4.2 Phase 5 已有能力

Phase 5 的 terminal READY snapshot 至少包含：

- canonical model key
- triggering scope
- required active scope snapshot
- 固定 historical time window
- 固定 `trainingDataSource`
- 依 scope 分區的 raw records
- scope record count 與 retrieval provenance

目前 READY snapshot 的讀取仍偏向 read-only inspection。本階段需增加 atomic single-consumer claim，
但不新增 external dataset API、SQLite、persistent queue 或 distributed job service。

### 4.3 歷史 Daisy 設計的使用方式

`nwdaf-daisy-improvement-plan.md` 與 `daisy_model_bundle_requirements.md` 只作為以下既有行為的參考：

- 十個 UE communication features 的定義。
- timestamp aggregation 方式。
- TCN bundle 內容與安全載入概念。
- training/validation preprocessing。

新 production code、config、package name 與 runtime dependency 不得引入 Daisy。歷史規劃與 transition
record 可以說明其來源，但 Phase 6 runtime 使用單純的 PyMTLF local Python trainer。

---

## 5. Dataset 到訓練 sample 的轉換

### 5.1 輸入資料

輸入為 Phase 5 保存的 ADRF-aligned raw record，其主要資料位於：

```text
dataSub.smfDataSub
dataNotif.upfEventNotifs
```

Mongo fallback 也保存相同 domain shape，因此 ADRF 與 Mongo 來源在進入 trainer 前使用同一個 converter。
不得因 source 不同產生兩套 feature semantics。

### 5.2 Feature schema

沿用已驗證的十個 feature：

1. `total_vol`
2. `ul_vol`
3. `dl_vol`
4. `total_nb_pkts`
5. `ul_nb_pkts`
6. `dl_nb_pkts`
7. `ul_thr`
8. `dl_thr`
9. `ul_pkt_thr`
10. `dl_pkt_thr`

feature 名稱、順序與 shape 必須和 PyAnLF runtime 及 model bundle `config.json` 一致。任一欄位缺失、
非有限數值或 shape 不一致時，不得默默改變 feature order；應依既有 raw conversion 降級規則處理單筆
measurement，若有效樣本不足則 training job 明確失敗。

### 5.3 Prediction target

不得把十個 input features 全部直接當成固定 output。trainer 必須從 current bundle 的
`inference.output_fields` 與 `inference.out_seq_len` 建立 target；目前 seed bundle 的實際設定為：

```text
output_fields = [ul_vol, dl_vol]
out_seq_len = 1
```

也就是使用前 `seq_length=30` 個十維 observations，預測下一個時間點的 UL/DL volume。若未來 trusted
bundle 改變 output fields 或 horizon，trainer 必須先驗證 current model class、`model.output_size`、
target tensor 與 PyAnLF inference 都能以同一 manifest 表達；不得只改 trainer 造成 bundle consumer
語意分歧。

### 5.4 聚合規則

同一 scope、同一 timestamp 的 UPF observations：

- volume 與 packet count 採加總。
- throughput 與 packet throughput 採有效 observations 的平均。
- 保留 scope identity，不在 feature aggregation 階段把不同 group/SUPI scope 混成一條時間序列。
- 最終 trainer 可以合併各 scope 的 training samples，但 split、window 與 validation metric 仍按 scope 計算。

這些規則是既有實驗與 Go/Python 遷移時已確認的業務語意，不得在移植時任意改成另一種 aggregation。

### 5.5 排序、重複與時間範圍

- 只接受 snapshot 固定 `TimeWindow` 內的 records。
- 依 scope 與 event timestamp 排序。
- 使用 Phase 5 已建立的 provenance/dedup identity 排除重複 notification。
- 不以 callback arrival order 當作 time-series order。
- 不跨不同 `trainingDataSource` 或不同 retrieval window 補資料。

---

## 6. Training 與 reference validation dataset

### 6.1 所有具備足夠資料的 active scopes 都參與訓練

任一 scope degradation 只代表觸發原因，不代表只使用該 scope 的資料。對同一 model：

- snapshot 建立當下的所有 required active scopes 都先嘗試建立 training/reference-validation samples。
- 每個 scope 都保留自己的 reference validation 區段。
- 不把整個「正常 scope」固定當 validation-only。
- 不把整個 triggering scope 固定當 training-only。

這保留目前「其中一個 group 表現下降，就以同一模型涵蓋的完整資料重新訓練」的實驗思路，也避免模型只修復
觸發 group 卻破壞其他正常 group。

### 6.2 Chronological split

每個 scope 分別依時間排序後切分：

```text
較早 20% -> reference validation
較晚 80% -> training
```

預設 `validation_ratio` 為 `0.20`。candidate 只訓練一次，完成評估後直接成為可 promotion 的 candidate；
不做第二次 final refit、rolling validation 或 cross-validation。

這個 reference validation 用來比較 current model 與 candidate 對既有 pattern 的相容性，不宣稱能估計
candidate 的未來表現。較新的資料優先進入 training，是為了讓 degradation 後的 pattern change 能較快被
模型吸收。

不得隨機打散 time series。為避免相鄰 sliding windows 共用 observation，validation 與 training 邊界必須
purge 至少 `seq_length + out_seq_len - 1` 個 observations；任何 input 或 target observation 都不得同時出現在
兩側。

### 6.3 Preprocessing

1. 對 feature 使用 `log1p`。
2. 只用所有 scope 較新 training 區段的 samples fit 一個 global `StandardScaler`。
3. validation 只做 transform，不得重新 fit scaler。
4. target 使用 `output_fields` 對應的 scaler columns，讓 model loss 在和 PyAnLF inference 相同的
   transformed space 計算。
5. candidate WAPE 驗證前，prediction 與 target 都必須用相同 columns 做 inverse scale，再經 `expm1`
   回到原始 volume 單位。
6. candidate bundle 內保存新 scaler。
7. PyAnLF inference 必須使用 candidate bundle 同一份 scaler、feature order 與 output fields。
8. 評估 current model 時使用 current bundle 自己的 scaler/config；評估 candidate 時使用 candidate 的
   scaler/config。兩者最後都還原到相同原始 volume 單位後才比較 WAPE。

### 6.4 最低資料量

實作時以 model context length、prediction horizon、validation ratio、purge gap 與 batch 建立 startup validation 及
per-scope minimum window 計算。

Triggering scope 必須同時具有：

- 足以完成 configured training 的 training targets。
- 足以計算 WAPE 的 reference validation targets。
- 全部為 finite 的 training 與 validation tensors。

任一條不成立，整個 job 失敗並釋放 retrain-in-flight，因為無法確認本次 degradation 已被有效訓練與評估。

其他 snapshot scopes 採降級處理：

| 資料狀況 | 行為 |
|---|---|
| training 與 validation 都足夠 | 加入 training，並納入 per-scope/aggregate performance evaluation |
| training 足夠但 validation 不足 | 加入 training，不納入 performance gate；記錄 structured warning |
| training 不足 | 不加入 training 或 performance gate；記錄 structured warning |

不得以重複複製樣本、跨 source/window 補資料或把 reference validation 資料放回 training 的方式硬做。
Candidate manifest 必須記錄每個 snapshot scope 的資料資格、實際 training/validation sample count 與排除原因。

---

## 7. Local trainer

### 7.1 Model 與 warm start

trainer 從 current trusted bundle 載入：

- `model.py` 中的 `Model`
- current weights
- authoritative `config.json`

candidate 預設以 current weights warm start，再使用新 dataset fit 新 scaler 與更新 weights。這保留目前
已運作模型的知識，也較符合 degradation 後微調的第一版需求。

不得從下載來的任意 Python source 建立 trainer。`model.py` 必須來自 PyMTLF 已接受並持有的 trusted
current bundle；candidate packaging 再複製該受信任檔案。

### 7.2 訓練預設值

第一版建議預設：

| 設定 | 預設值 |
|---|---:|
| device | CPU |
| optimizer | Adam |
| loss | Huber loss |
| batch size | 32 |
| learning rate | 0.001 |
| epochs | 18 |
| reference validation ratio | 0.20 |
| random seed | 42 |
| maximum concurrent training jobs | 1 |

設定可由 PyMTLF config 調整，但 startup 必須拒絕無效範圍，例如非正數 batch/epoch、超出 `(0,1)` 的
validation ratio 或小於一個的 concurrency。

### 7.3 Runtime dependencies

PyMTLF runtime 明確加入本地訓練所需 dependency：

- NumPy
- PyTorch
- scikit-learn
- joblib

不得透過 nwdaf-resources 或 PyAnLF 的環境隱式取得這些套件。

---

## 8. Candidate 驗證與接受政策

### 8.1 Metric

candidate 與 current model 都在完全相同的 per-scope reference validation targets 上計算 WAPE：

```text
A = sum(abs(actual))
E = sum(abs(actual - predicted))

A > 0           -> WAPE = E / A
A = 0 and E = 0 -> WAPE = 0
A = 0 and E > 0 -> WAPE = 1
```

每個具有有效 reference validation 的 scope 先將時間點與 `output_fields` 維度攤平成同一組
aligned actual/prediction values，再計算
`E_s` 與 `A_s`。整體 WAPE 直接以所有 evaluation-eligible scopes 的 error/actual sums 計算：

```text
aggregate WAPE = sum(E_s) / sum(A_s)
```

Aggregate 只包含具有有效 reference validation 的 scopes；triggering scope 永遠必須包含在內。整體分母為零
時使用相同的 zero-denominator 規則。不得先算各 scope WAPE 再做未加權平均，避免小 scope 對整體結果產生
不成比例的影響。

這和 Phase 4 accuracy policy 的 zero-denominator 定義一致。

### 8.2 永遠執行的評估

無論 performance gate 是否開啟，都必須在相同 per-scope reference validation targets 上計算並保存：

1. current 與 candidate 的 triggering-scope WAPE。
2. current 與 candidate 的每個其他 evaluation-eligible snapshot scope WAPE。
3. current 與 candidate 的 aggregate WAPE。
4. 每個 scope 與 aggregate 的 absolute delta。
5. performance gate 判定結果與拒絕原因。
6. 未納入 training 或 evaluation 的 scopes、sample count 與排除原因。

結果寫入 structured log 與 candidate manifest summary。即使 `enforce_performance_gate=false`，評估也不能
跳過；表現退步時必須輸出 warning，讓實驗使用者能觀察。

### 8.3 Performance gate

`enforce_performance_gate=true` 時，candidate 必須同時滿足：

1. triggering scope 的 WAPE 優於 current model。
2. aggregate WAPE 優於 current model。
3. 每個其他 evaluation-eligible snapshot scope 的 absolute WAPE regression 不超過 `0.02`。

任一條不成立，candidate 即 rejected，不發布為 current model。

`0.02` 是 absolute WAPE ratio，例如 `0.10 -> 0.12`，不是相對增加 2%。

`enforce_performance_gate=false` 時，上述結果只作 observability，不阻擋 promotion。這個開關不能關閉以下
技術正確性檢查：

- triggering scope 的最低 training/reference-validation 資料量。
- 每個實際納入 training/evaluation scope 的 tensor 與 sample eligibility。
- input、target、loss、weights、prediction 與 metric 均為 finite。
- model output shape、feature order、output fields 與 scaler 相容。
- bundle digest、safe package validation 與完整 reload。
- ModelKey、base generation 與 artifact identity 的 stale check。

### 8.4 不以單一 aggregate 掩蓋 group regression

只看整體平均可能出現「大量正常 group 改善，掩蓋單一 group 嚴重退步」。因此 per-scope metric 是
永遠產生的必要結果；performance gate 開啟時也是 acceptance 的必要輸入。

---

## 9. Model identity、generation 與 artifact

### 9.1 三層 identity

本階段明確區分：

| Identity | 用途 |
|---|---|
| `ModelKey` | PyMTLF canonical policy/catalog key；包含 provider namespace 與穩定 model identity |
| `modelUniqueId` | 標準 Model Provision wire identity；同一模型更新時保持不變 |
| `ArtifactKey` | bundle bytes 的 SHA-256；每個 immutable artifact 唯一 |

不得把 callback URL、scope ID、training job ID 或 transient generation 當成標準模型 identity。

目前 `provider namespace` 是 PyMTLF config 中固定的內部模型提供者名稱：

```text
model_provision.provider_namespace = "local-mtlf"
ModelKey = ("local-mtlf", modelUniqueId)
```

它不屬於 3GPP Model Provision wire，也不傳給 Go 或 PyAnLF；用途只是避免未來不同 model provider 使用相同
數字 `modelUniqueId` 時在 PyMTLF 內部碰撞。current single-PyMTLF deployment 不需要動態協調或發現這個值。

`UE_COMMUNICATION` 則是 seed/current model 的 applicability。PyMTLF 先以 Model Provision demand 中的
`mLEvent`、filter、target 等資訊解析相容模型，選定後才使用 ModelKey 管理該模型的 policy、artifact、
generation 與 retraining。`UE_COMMUNICATION` 不是 ModelKey 的組成欄位；目前 config 已要求同一 provider
內的 model IDs 唯一。

### 9.2 Generation

- configured seed bundle 匯入後為 generation 1。
- candidate generation 為 current generation + 1。
- generation 只存在 PyMTLF process memory、catalog metadata 與 local bundle manifest。
- 不透過標準 Model Provision payload傳送 generation。
- 不在 Go 建立 generation mirror 或 CAS protocol。

### 9.3 Bundle 格式

沿用目前已由 PyMTLF 與 PyAnLF 驗證的四檔 bundle：

```text
config.json
model.py
model.npy
scaler.pkl
```

`config.json` 是 current authoritative manifest，不恢復歷史規劃中的 `config.yaml`。

builder 必須：

1. 從 current trusted bundle 複製受信任 `model.py`。
2. 寫入 candidate weights 與新 scaler。
3. 寫入 feature order、model shape 與必要 runtime config。
4. 計算各檔 digest 及整體 artifact SHA-256。
5. 在 publish 前使用和 consumer 相容的 validator 完整重載一次。
6. 以 artifact digest 建立 immutable URL，不覆寫既有 bundle。

local manifest 可保存：

- `model_generation`
- `parent_artifact_key`
- training source
- training time window
- 每個 scope 的 sample count
- current/candidate validation summary

manifest 不得保存：

- SUPI 清單
- raw targets 或完整 training records
- callback credential
- NRF/ADRF endpoint credential

### 9.4 Seed 與 retrained model 使用同一 catalog

目前只處理 seed 的 `SeedCatalog` 要泛化為 current model catalog：

- configured seed 是 generation 1 baseline。
- 每個 ModelKey 有 current generation、artifact、applicability 與 bundle metadata。
- candidate 先 staged，通過驗證與 stale check 後才 atomic promote。
- Model Provision create 與 update notification 都在送出當下解析 catalog current artifact，不固定保存 seed tuple。

第一版 catalog 仍為 process-local。PyMTLF restart 後回到 configured seed baseline；磁碟上的 candidate
artifacts可以保留，但不得因檔案存在就自動視為 current model。

---

## 10. Stale snapshot 與 promotion

### 10.1 Promotion 前重新確認

長時間訓練期間，provision demand 或 active monitoring scopes 可能改變。candidate promotion 前至少檢查：

1. catalog current generation/artifact 仍是 training 開始時的 base。
2. ModelKey 仍存在。
3. triggering scope 沒有被錯誤改綁到另一個 model identity。

若 base 已被其他更新取代，candidate 視為 stale，不得覆蓋較新的 current model。

訓練期間新增或刪除 scope 不阻擋 promotion。例如 snapshot 建立時只有 group A/B，訓練期間又收到共用
相同 ModelKey 的 group C subscription，candidate 仍可 promotion；C 沒有參與本次訓練/評估的事實必須記錄
warning。已刪除的 scope 也只記錄差異，不要求浪費已完成的訓練。

第一版 stale policy 為：

| 變化 | Promotion 行為 |
|---|---|
| base generation/artifact 已被另一更新取代 | 阻擋 |
| ModelKey 已不存在或已無任何有效 demand | 取消 |
| triggering scope 改綁到另一 ModelKey | 阻擋 |
| active scope 新增或刪除 | 記錄 structured warning，繼續 |

這是為了優先縮短 degradation recovery time。其限制是新加入 scope 可能套用一個未使用該 scope 資料驗證的
updated model，current experiment 明確接受此限制。

### 10.2 建議 promotion 順序

```text
build candidate bundle
    -> reload/validate bundle
    -> calculate per-scope acceptance
    -> publish immutable artifact
    -> recheck base generation/model identity and record scope drift
    -> atomically promote current catalog
    -> establish notification desired state
    -> advance/reset PyMTLF accuracy-policy generation
    -> release retrain-in-flight
```

artifact publish 在 promote 前完成；publish 成功但 stale check 失敗時，該 artifact 只是未被引用的 immutable
檔案，不得成為 current。

---

## 11. 更新模型的 Model Provision reconciliation

### 11.1 Existing provision resource

Phase 4 建立的 Model Provision subscription resource 繼續有效。PyMTLF 不為每次 retrain 建立新 resource；
對每個相容 resource 產生新的 desired notification：

- 相同 `modelUniqueId`
- 新的 immutable `mLModelUrl`
- resource 所需的 event/filter/target applicability
- resource 原有 `notifCorreId`

不放入 private generation、provider ID、training task ID 或 `modelUpdateInd`。

### 11.2 Desired state，不是無界 event queue

每個 provision resource 保存：

- current desired generation/artifact
- last delivered generation/artifact
- retry attempt 與 next retry time
- resource mutation revision

若 generation N 尚未成功送達時 generation N+1 已成為 current，只需要送最新 desired state；舊通知可被
coalesce。不得建立無界 notification history queue。

callback transport failure 使用 capped exponential backoff。resource 被更新或刪除後，舊 revision 的 retry
必須取消，避免將過期 applicability 送到 consumer。

### 11.3 Completion boundary

建議 PyMTLF 在以下條件成立時，將 training job 視為完成：

1. candidate 已驗證並成為 catalog current。
2. 所有當下相容 provision resources 都已建立最新 desired notification reconciliation。

不等待 PyAnLF 完成下載與 activation，因為標準 callback 的 `204` 不具有 activation acknowledgement 語意。
callback 暫時不可達時，notification reconciliation 繼續 retry，但不 rollback 已成為 current 的 PyMTLF
artifact。

---

## 12. PyAnLF candidate-first atomic replacement

### 12.1 Notification acceptance

PyAnLF 收到更新 notification 時：

1. 進行標準 schema 與 resource correlation 驗證。
2. 保存該 resource 的最新 desired model URL。
3. 以 bounded、coalescing worker 排入 model preparation。
4. 回 `204 No Content`。

下載、digest 驗證與 load 不在 callback request critical path 執行。

### 12.2 Prepare once，commit together

同一 model identity 可能被多個 analytics subscriptions／monitor scopes 共用。更新時：

1. 解析所有仍指向該 model identity 的 runtime。
2. 在 runtime locks 外只下載並完整 prepare candidate 一次。
3. snapshot 各 runtime 的 expected revision/current artifact。
4. 以 deterministic order 取得必要 locks。
5. 再次檢查 resource/model/runtime 沒有 stale。
6. 一次將所有相依 runtime 切換到 candidate。
7. commit 成功後才 release old model references。

若任一 runtime 不相容、resource 已變更、candidate 載入失敗或 stale check 失敗，全部 runtime 保持舊模型；
不得只切一半 scopes。

### 12.3 失敗保留舊模型

以下情況都不得破壞 current inference：

- download timeout
- HTTP 非成功
- bundle size/path 安全檢查失敗
- digest 不一致
- `config.json`、feature order 或 shape 不相容
- model/scaler deserialize 失敗
- runtime revision 已變更

worker 依既有 retry/backoff 政策重新處理最新 desired artifact。使用者的 analytics subscription 不因一次
model update 失敗而被刪除。

---

## 13. Accuracy report 與 generation 切換

### 13.1 為何需要 update-pending gate

標準 accuracy notification 沒有 private generation 欄位。若 PyMTLF promotion 後 PyAnLF 仍在下載
candidate，PyAnLF 繼續送舊模型的 WAPE，PyMTLF 可能把舊表現誤認為新 generation 的 degradation。

### 13.2 採用的第一版語意

當 PyAnLF 對某 model identity 有 replacement pending：

- 在回應 Model Provision callback 前，先原子標記所有相依 scopes 為 pending，停止產生新的舊模型 WAPE。
- 清除尚未送出的舊 report；若已有 report 正在傳送，先讓該次傳送完成，再回 Model Provision callback。
- 仍依標準週期送出合法 monitor notification，維持 subscription liveness。
- 暫時省略 optional `MLModelAccuracyInfo.deviation`。
- PyMTLF 依 Phase 4 規則將沒有 `deviation` 的 notification 視為 liveness-only，不更新 degradation policy。

candidate 成功 atomic swap 後：

- 清除每個相依 runtime 的 prediction/ground-truth alignment window。
- 清除舊 report-period accumulator。
- 每個 scope 的 ordered accuracy delivery lane 至少成功送出一次不含 `deviation` 的 pending-period
  notification 後，才允許後續新模型 WAPE 通過；這個標準 notification 是 local policy 的 cutover barrier，
  不是新增 wire 欄位。
- 從新模型產生足夠穩定 prediction 後，再恢復帶 `deviation` 的 report。
- PyMTLF promotion 後先將相關 scopes 設為 waiting-for-cutover；收到 Model Provision callback 的 `204`，
  再收到該 scope ordered lane 的 liveness-only barrier 後，才使用已 advance 的 local policy generation
  接受後續 observation。

這避免新增 custom activation acknowledgement 或在標準 payload 夾帶 generation。

---

## 14. Job 與狀態模型

### 14.1 Dataset handoff

```text
READY
  -> CLAIMED
      -> terminal success/failure
```

claim 必須是 atomic。相同 DatasetSnapshot 只能被一個 training job 消費。

### 14.2 Training job

對外觀察只保留簡單狀態：

```text
PENDING
  -> RUNNING
      -> REPROVISIONING
          -> COMPLETED

PENDING/RUNNING/REPROVISIONING
  -> FAILED
  -> CANCELLED
```

詳細 stage 可放在 log/diagnostic record，例如：

- `BUILDING_DATASET`
- `FITTING`
- `VALIDATING`
- `PACKAGING`
- `PROMOTING`
- `NOTIFYING`

但不把每個 stage 變成 external API 或複雜 persistent workflow。

### 14.3 Concurrency

- 同一 ModelKey 永遠只允許一個 retrain-in-flight。
- 第一版整個 PyMTLF 預設最多一個 concurrent training job。
- scheduler 使用 bounded queue；queue 滿時不得無聲覆蓋 job。
- Phase 4 對同一 model 的後續 degradation 在 in-flight 期間只更新必要 observability，不建立重複 dataset
  retrieval/training。

---

## 15. 失敗、重試與 restart

### 15.1 Promotion 前失敗

dataset conversion、training、validation、packaging 或 stale check 失敗時：

- current catalog 與現有 PyAnLF model 不變。
- job 進入 FAILED。
- Phase 5 snapshot 記錄 terminal outcome。
- 呼叫既有 `complete_retrain` 釋放 model-level in-flight。
- 不自動用完全相同 snapshot 無限重訓；後續 degradation 可用新的 historical window 再觸發。

### 15.2 Promotion 後 callback 暫時失敗

- PyMTLF current model 不 rollback。
- desired notification 保留並 capped backoff retry。
- PyAnLF 繼續使用舊模型。
- PyAnLF update-pending 只在已接受到 notification 後生效；尚未收到時仍可能回報舊模型 accuracy，
  因此 PyMTLF promotion 後也必須忽略該 model 舊 policy generation 的 observation，直到新 generation
  observation window 正式開始。

### 15.3 Restart

第一版明確接受 process-local state：

| Restart | 行為 |
|---|---|
| PyMTLF restart | training/current generation state 遺失，回到 configured seed baseline；實驗重跑 |
| PyAnLF restart | Go sync 恢復 subscription snapshot，PyMTLF 依 current catalog 重新提供模型；PyAnLF cache 可重用或重下載 |
| Go restart | process-local mirror 遺失，依現有 backend reconnect/sync 重建；實驗可重跑 |

本階段不新增 SQLite、journal、cross-process transaction 或 crash recovery database。

---

## 16. Config

建議新增的 PyMTLF config 群組：

```yaml
training:
  enabled: true
  device: cpu
  batch_size: 32
  learning_rate: 0.001
  epochs: 18
  validation_ratio: 0.20
  random_seed: 42
  max_concurrent_jobs: 1
  max_queue_size: <bounded positive value>
  enforce_performance_gate: true
  max_scope_wape_regression: 0.02

# 既有 storage/artifact 群組繼續擁有 artifact root、public base URL 與安全限制
```

實際 key 命名需遵守 PyMTLF 現有 config style；不得建立同義重複欄位。startup validation 必須覆蓋：

- range 與 finite 檢查
- writable artifact root
- public base URL 與現有 artifact server 一致
- bundle size/download timeout 沿用既有較寬鬆網路預設
- queue/concurrency 不得為零或負數

---

## 17. 實作切片

本階段完成後再統一 review，不要求每個切片各自 commit。

### Slice A：READY handoff 與 dataset builder

PyMTLF：

- atomic claim READY snapshot
- raw record converter
- per-scope chronological series
- feature aggregation、dedup 與 minimum-window validation
- triggering/non-triggering scope data eligibility
- training/reference-validation tensors
- unit/characterization tests

### Slice B：Trainer 與 candidate policy

PyMTLF：

- trusted current bundle warm start
- deterministic CPU local trainer
- scaler fit/transform
- single-pass recent-data training with older reference validation
- current/candidate per-scope WAPE
- always-on evaluation 與 optional performance gate
- failure/release semantics

### Slice C：Catalog、bundle 與 artifact

PyMTLF：

- SeedCatalog 泛化為 current model catalog
- generation/staged/promote
- deterministic bundle builder
- digest/reload validation
- immutable artifact publication
- stale base/model check 與 non-blocking scope-drift warning

### Slice D：Updated Model Provision reconciliation

PyMTLF：

- provision resource 在 send-time 解析 current catalog
- desired/delivered generation state
- retry、coalescing 與 resource revision cancellation
- successful promotion 後 accuracy-policy generation advance

NWDAF：

- 以 contract/regression tests 確認 same model ID/new URL payload lossless forwarding
- 確認 standard status 與 callback mapping 未改變
- 除非測試發現既有 routing defect，否則不新增 production business logic

### Slice E：PyAnLF atomic replacement

PyAnLF：

- same identity/new URL 視為 candidate update
- latest desired artifact coalescing
- prepare-once
- multi-runtime deterministic atomic swap
- failure retains old
- replacement-pending accuracy gate
- successful swap 後清除 inference/accuracy windows

### Slice F：Process-level verification 與文件

nwdaf-resources：

- 啟動 Go、PyAnLF、PyMTLF 的真實 process harness
- 建立兩個不同 group/scopes，共用同一 model
- 觸發其中一個 scope degradation
- 驗證 snapshot 建立當下所有 active scope data 進入 dataset
- 驗證 candidate training 與 per-scope gate
- 驗證更新通知、direct download 與 atomic multi-scope switch
- 驗證 callback/download 失敗保留舊模型並可恢復

nwdaf-docs：

- 更新 master plan status
- 記錄最終 config/API/behavior
- 補充實際驗證命令與結果

---

## 18. Verification Matrix

### 18.1 PyMTLF unit/contract

1. READY snapshot 只能成功 claim 一次。
2. ADRF/Mongo 相同 raw domain record 產生相同 features。
3. timestamp aggregation 保留既有 sum/mean 語意。
4. 較早 reference validation 與較新 training 依時間分區，purge gap 防止 sliding-window observation 重疊。
5. scaler 只 fit training-period data。
6. zero-denominator WAPE 與 Phase 4 一致。
7. candidate 只訓練一次，評估後相同 weights 直接進入 packaging。
8. performance gate 開啟時，triggering scope、aggregate 與 other-scope regression 可各自拒絕 candidate。
9. performance gate 關閉時仍計算、保存並 warning 相同 metrics，但允許 technically valid candidate。
10. triggering scope 資料不足時 job 失敗並釋放 in-flight。
11. non-triggering scope training/validation 不足時依資格加入、排除並記錄 warning，不拖垮整個 job。
12. aggregate WAPE 只包含 evaluation-eligible scopes，且 triggering scope 必須存在。
13. non-finite metric、trainer exception 正確失敗並釋放 in-flight。
14. current bundle warm start 與 deterministic seed 可重現。
15. bundle digest、reload、immutable URL 與 catalog promotion。
16. stale base generation 不得 promote；scope 新增/刪除只記錄 warning。
17. notification retry coalesces 到最新 desired artifact。
18. resource update/delete 取消 stale retry。

### 18.2 PyAnLF unit/contract

1. same `modelUniqueId`/new URL 進入 replacement path。
2. callback 在保存與 enqueue 後回 204，不等待下載。
3. candidate 只 prepare 一次並提供給所有相依 runtimes。
4. 多 runtime 全部成功才 commit。
5. 任一 validation/load/stale failure 時全部保留舊模型。
6. replacement pending 時合法通知仍送出，但省略 `deviation`。
7. swap 後 prediction、ground-truth 與 report windows 全部重設。
8. 穩定累積足夠資料後恢復新模型 WAPE report。

### 18.3 NWDAF regression

1. Model Provision notification 的 standard fields lossless round-trip。
2. 相同 model ID/new URL 不被 Go 誤判為 duplicate old notification。
3. backend unavailable、peer error 與 callback error 保持 Phase 4 status mapping。
4. Go 不解析 bundle、generation 或 training metadata。

### 18.4 Process-level E2E

最少涵蓋：

```text
two analytics subscriptions
  -> same model identity
  -> two independent monitor scopes
  -> one scope degradation
  -> one model-level retrain
  -> all eligible snapshot scopes included
  -> local candidate accepted
  -> immutable updated artifact
  -> standard Model Provision notify
  -> PyAnLF direct download
  -> atomic replacement for both scopes
  -> new stable WAPE reports
```

另測：

- rejected candidate 不通知、不切換。
- `enforce_performance_gate=false` 時表現退步仍有完整 log 並可 promotion。
- non-triggering scope 資料不足時仍可完成 retraining，且 manifest/log 清楚記錄其資格與排除原因。
- retraining 期間新增或刪除 scope 不浪費 candidate，且產生 scope-drift warning。
- callback 暫時失敗後 retry。
- artifact download/validation 失敗保留舊模型。
- provision resource 在 retraining 期間刪除或變更。
- PyAnLF restart 後重新取得 current model。

---

## 19. 明確不在本階段處理

- Multiple-NWDAF、AGG、AoI routing、HFL 或 Daisy。
- `Nnwdaf_MLModelTraining`。
- 只使用某一條 path 訓練、另一條 path 專門驗證。
- 跨 ADRF/Mongo 歷史 gap 回填或 merge。
- persistent training database、SQLite、distributed queue。
- custom activation acknowledgement、custom `ModelReady` 或 generation wire protocol。
- TLS/OAuth 與 Python backend 的標準 NF identity。
- Go MTLF package 最終整形。
- `NWDAF/internal/mlmodel/` compatibility code 的 relocation/removal。
- legacy/Daisy dead code 全面清除。

最後三項由 Phase 7 統一處理，避免在功能尚未完整連通前同時做大範圍 package relocation。

---

## 20. 已確認決策

### D1. Candidate acceptance 與 scope stability

1. Candidate 只訓練一次：較早 20% 作 reference validation，較晚 80% 作 training。
2. Performance evaluation 永遠執行並寫入 structured log/manifest。
3. `enforce_performance_gate=true` 時要求 triggering scope 與 aggregate 改善，其他 scope regression
   不超過 `0.02`；關閉時表現結果只告警，不阻擋 technically valid candidate。
4. scope 新增或刪除不阻擋 promotion，只記錄 scope drift；base generation/identity stale 仍必須阻擋。
5. triggering scope 資料不足時 job 失敗；其他 scope 資料不足時依 training/evaluation 資格降級處理並告警，
   不使整個 job 失敗。

### D2. Training completion 邊界

採用 asynchronous callback：

- `204` 只表示 PyAnLF 已接受 notification。
- PyMTLF 在 artifact promotion 並建立 notification desired-state reconciliation 後完成 job。
- 不等待 PyAnLF activation，不新增 private ack。
- PyAnLF 以 pending 時省略 `deviation`、成功 swap 後 reset windows 解決 generation gap。

不新增 private activation acknowledgement。

### D3. Warm start

從 current model weights warm start，並對新 training-period data 重新 fit scaler；第一版不使用 fresh
initialization。

---

## 21. 完成條件

本階段只有在以下條件全部滿足時才算完成：

1. Phase 5 READY dataset 能被單次 claim 並進入 bounded local training。
2. Triggering scope 必須具備完整 training/reference-validation 資料；其他 snapshot scopes 依實際資料資格
   加入單次 training/evaluation，任何排除都有 structured log 與 manifest record。
3. Performance evaluation 永遠產生；是否因退步拒絕 candidate 由 `enforce_performance_gate` 控制，技術正確性
   與 stale base checks 永遠不可關閉。
4. PyMTLF 建立可由 PyAnLF 直接下載的 immutable、可重載、digest 驗證 bundle。
5. updated model 透過既有標準 Model Provision resource 送出，不含自創 wire 欄位。
6. PyAnLF 對所有相依 runtime 做 atomic replacement，任何失敗皆保留舊模型。
7. update gap 不會讓舊模型 WAPE 污染新 PyMTLF policy generation。
8. Go 仍只負責 SBI validation/routing，沒有 training 或 artifact business logic。
9. PyMTLF、PyAnLF、NWDAF 各自 lint/unit/contract tests 通過。
10. nwdaf-resources 的真實三 process E2E 通過並有完整操作說明。
11. 文件記錄實際 config、API、限制、驗證結果與剩餘 Phase 7 工作。
