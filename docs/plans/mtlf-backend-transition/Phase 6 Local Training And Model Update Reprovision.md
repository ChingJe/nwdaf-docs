# Phase 6 本地訓練與更新模型重新提供

日期：2026-07-25

狀態：FamilyKey、per-artifact model identity、local training/reprovision及
registration-driven activation lifecycle已完成本地實作與驗證

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
    -> PyMTLF 建立 family-level retrain intent
    -> Phase 5 蒐集同一 logical model family 所有 active scopes 的資料
    -> READY DatasetSnapshot 被 Phase 6 原子 claim
    -> PyMTLF 建立較早 reference validation／較新 training dataset 並本地訓練 candidate
    -> 分 scope 與整體驗證 candidate
    -> 為 promoted artifact 配發新的 modelUniqueId
    -> 建立、驗證並發布包含新 identity 的 immutable model bundle
    -> 更新 PyMTLF logical model family 的 current version
    -> 透過既有標準 Model Provision notification 通知 PyAnLF
    -> PyAnLF 下載並完整準備 candidate
    -> 所有相依 runtime 原子切換到新 model identity
    -> PyAnLF 以新 modelUniqueId 建立 Model Monitor registration
    -> PyMTLF 建立新的 Monitor subscription/correlation
    -> 重設相關 accuracy window，開始回報明確屬於新 modelUniqueId 的表現
```

本階段不重建初始 Model Provision resource、ADRF/Mongo retrieval 或 Go backend lifecycle。Phase 2 至
Phase 5 建立的標準 resource 與 routing 繼續沿用；但原始實作把穩定 `modelUniqueId` 同時當成 logical model
family 與 artifact version，必須在本階段拆開，才能讓更新模型的 activation 與 accuracy report 有可靠的
標準身分分界。

---

## 2. 核心責任邊界

### 2.1 PyMTLF

PyMTLF 擁有：

1. 原子 claim READY dataset，避免相同 snapshot 被重複訓練。
2. raw ADRF-aligned record 到訓練 sample 的轉換。
3. chronological reference-validation/training split。
4. local trainer、candidate evaluation 與 acceptance policy。
5. logical model family、model version identity、generation、bundle packaging、digest 驗證與 immutable
   artifact publication。
6. current model catalog、wire model ID allocation/index 與同一 family 的 single retrain-in-flight。
7. Model Provision update notification 的 desired state 與 retry reconciliation。
8. 依 Model Monitor registration lifecycle 記錄各 scope 採用哪個 model version。
9. terminal training outcome 後釋放 Phase 4/5 的 retrain-in-flight。

### 2.2 PyAnLF

PyAnLF 擁有：

1. 接收標準 Model Provision notification。
2. 直接由 `mLFileAddr.mLModelUrl` 下載 candidate bundle。
3. bundle 安全檢查、digest 驗證、模型與 scaler 完整載入。
4. 依 provision resource 與 canonical applicability 找出同一 logical model slot 的所有 runtime。
5. candidate-first、all-or-nothing 的 atomic runtime identity/artifact swap。
6. candidate 準備或切換失敗時保留舊模型。
7. 成功切換後重新產生 Model Monitor desired registrations，讓新 registration 成為「已開始使用新模型」的
   標準證據。
8. 切換時停止舊 model ID 的 measurement，並重設 prediction/accuracy windows。

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
- 對應 logical model family 與各代 model ID。
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

TS 23.288 clause 6.2A.1允許 MTLF 對既有 subscription 提供 new 或 re-trained ML Model；clause 6.2A.2
以 unique ML Model identifier 與 model information 組成提供結果。規格沒有要求 re-trained artifact
必須沿用舊 identifier，也沒有定義 artifact generation 欄位。因此，本計畫採用「每個 promoted artifact
使用新的 `modelUniqueId`」作為本地 identity policy。這是規格允許、但不是規格強制的選擇。

### 3.4 Model Monitor lifecycle 是 activation 證據

TS 23.288 clause 6.2E.3.2 明確描述：

- AnLF 開始使用某個 ML Model 並具備監控能力時，向負責的 MTLF 執行
  `Nnwdaf_MLModelMonitor_Register`。
- AnLF 不再使用或監控該模型時執行 `Nnwdaf_MLModelMonitor_Deregister`。

Clause 6.2E.3.3 接著由 MTLF 因 registration 建立 Model Monitor subscription。TS 29.520 Release 18
OpenAPI 也要求 registration 具有 `modelId`，monitor subscription 具有 `modelIds` 與 `notifCorrId`，
accuracy info 則具有 `modelId`。

本階段據此採用下列可驗證分界：

```text
old model M1 + registration R1 + monitor subscription/correlation S1/C1
    -> Model Provision 提供 candidate M2
    -> PyAnLF 完成下載、驗證與 atomic activation
    -> PyAnLF desired registration 改為 M2
    -> 建立 R2，PyMTLF 再建立 S2/C2
    -> 只有屬於 M2 且由 active C2 route 收到的 WAPE 才是新模型 observation
    -> R1/S1 依既有 reconciler 刪除
```

`R2` 的出現代表 AnLF 已經開始使用 M2，而不是只收到 M2 的 URL。這是依 registration 的規格語意建立的
implementation inference；規格並沒有另外定義 `ModelActivated` operation。

### 3.5 generation 仍只屬於 PyMTLF 內部狀態

TS 23.288 的程序文字提到 optional ML Model provide indicator，但目前 workspace 採用的
Release 18 V18.13 `TS29520_Nnwdaf_MLModelProvision.yaml` 並沒有 `modelUpdateInd` 欄位。現有 schema
具有 `modelUniqueId`、`mLFileAddr`、`mLModelAdrf`、`addModelInfo` 與 `mlDegradInd`，但
`mlDegradInd` 不是 model update flag。

因此本階段：

- 不自行發明或傳送 `modelUpdateInd`。
- 不把 private generation 放入標準 payload。
- 每個 promoted artifact 使用新的 `modelUniqueId` 與新的 immutable `mLModelUrl`。
- logical model family只存在PyMTLF的FamilyKey；PyAnLF以本地ModelSlotKey建立對應，兩者都不加入
  標準payload，也不需要交換private family ID。
- generation 只用於 PyMTLF process 內的順序、stale check、catalog promotion 與 retry coalescing。

`MLModelProvision` callback 的 `204` 只表示 notification 已成功送達／被接受，不表示 consumer 已下載或
啟用模型。`validityPeriod` 表示模型資訊的適用期間，`monitorInterval` 表示 accuracy measurement window，
兩者也不是 activation acknowledgement。沒有 `deviation` 的合法 periodic accuracy report 仍可依 Phase 4
作為資料不足時的 liveness/observability，但不得再作為 model generation cutover barrier。

### 3.6 HTTP 行為

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

### 3.7 本地規格來源

- `specs/TS 23.288/6 Procedures to Support Network Data Analytics/6.2A Procedure for ML Model Provisioning.md`
- `specs/TS 23.288/6 Procedures to Support Network Data Analytics/6.2E MTLF-based ML Model Accuracy Monitoring.md`
- `specs/TS 23.288/7 Nnwdaf Services Description/7.9 Nnwdaf_MLModelMonitor Service.md`
- `specs/TS 23.288/6 Procedures to Support Network Data Analytics/6.2F Procedure for ML Model Training.md`
- `specs/TS 29.520/4 Services offered by the NWDAF/4.5 Nnwdaf_MLModelProvision Service.md`
- `specs/TS 29.520/4 Services offered by the NWDAF/4.6 Nnwdaf_MLModelTraining Service.md`
- `specs/openapi/TS29520_Nnwdaf_MLModelProvision.yaml`
- `specs/openapi/TS29520_Nnwdaf_MLModelMonitor.yaml`
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
- PyMTLF 以 degradation-only policy 產生 family-level retrain intent。

本階段不得重建上述 resource 或改變既有 WAPE 計算、baseline、z-score、hit window 與 degradation
threshold 邏輯。允許且必須調整的是 identity indexing：policy 從 `(provider, modelUniqueId)` 改綁
logical family，stable scope key 不再包含每代 `modelId`，並在新 model registration 啟用該 scope 時重設
該 scope 的 baseline/window。這是身分遷移，不是 accuracy policy redesign。

### 4.2 Phase 5 已有能力

Phase 5 的 terminal READY snapshot 至少包含：

- canonical logical model family key
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

任一 scope degradation 只代表觸發原因，不代表只使用該 scope 的資料。對同一 logical model family：

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

第一版已確認預設：

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
- FamilyKey、base ModelVersionKey、generation 與 artifact identity 的 stale check。

### 8.4 不以單一 aggregate 掩蓋 group regression

只看整體平均可能出現「大量正常 group 改善，掩蓋單一 group 嚴重退步」。因此 per-scope metric 是
永遠產生的必要結果；performance gate 開啟時也是 acceptance 的必要輸入。

---

## 9. Model identity、generation 與 artifact

### 9.1 四層 identity

本階段明確區分：

| Identity | 用途 |
|---|---|
| `FamilyKey` | PyMTLF canonical policy/catalog key；在多代 retraining 間保持穩定 |
| `ModelVersionKey` | `(provider namespace, modelUniqueId)`；識別可實際 provision／monitor 的一代模型 |
| `modelUniqueId` | 標準 wire identity；每個 promoted artifact 配發新值 |
| `ArtifactKey` | bundle bytes 的 SHA-256；每個 immutable artifact 唯一 |

不得把 callback URL、monitor scope ID、training job ID、family ID 或 transient generation 放進標準
`modelUniqueId` 欄位。

PyMTLF config 為每個 configured seed 明確指定內部 `family_id`。已確認的形狀為：

```yaml
model_provision:
  provider_namespace: "local-mtlf"
  seed_models:
    - family_id: "ue-communication-default"
      model_id: 1
      artifact_key: "<sha256>"
      event: "UE_COMMUNICATION"
      event_filter: {}
```

```text
FamilyKey = ("local-mtlf", "ue-communication-default")
ModelVersionKey(generation 1) = ("local-mtlf", 1)
ModelVersionKey(generation 2) = ("local-mtlf", 2)
```

`provider_namespace` 與 `family_id` 都是 PyMTLF 本地 catalog identity，不加入 3GPP payload。`family_id`
使用明確 config 而不直接 hash 整份 filter/target，避免 canonical applicability 因需求擴張而意外改變
retraining ownership。

FamilyKey本身只由`provider_namespace + family_id`組成。Applicability descriptor另外保存在family catalog：

```text
FamilyCatalogEntry
  family_key
  applicability:
    event
    event_filter
    target_ue
    model_interoperability
    use_case_context
  current_model_version
```

Applicability負責將Model Provision demand解析到family，但不構成FamilyKey，也不隨每代artifact重複建立。
Configured seed descriptor是第一版authoritative來源；candidate/retrained version繼承相同descriptor。
Bundle manifest可以保存descriptor snapshot供驗證與追蹤，但不能取代catalog selection state。

第一版retraining不得順便改變family applicability。若event、interoperability或其他適用條件需要形成不同
模型選擇語意，必須建立新的configured family，而不是在promotion時靜默修改既有family。

`UE_COMMUNICATION` 則是 seed/current model 的 applicability。PyMTLF 先以 Model Provision demand 中的
`mLEvent`、filter、target、use-case context 與 interoperability requirement 解析相容 family，選定後才用
`FamilyKey` 管理 policy、artifact、generation 與 retraining。若 demand 帶有已知舊 `modelId`，PyMTLF 先透過
version index 找到 family，再提供該 family 的 current version；未知 ID 不可被錯誤視為另一個新 family。

### 9.2 Model ID allocation 與 index

第一版由單一 PyMTLF provider 擁有 wire model ID allocation：

1. configured seed 的 `model_id` 是 generation 1 的 wire ID。
2. catalog startup 驗證所有 configured seed IDs 在 provider 內唯一。
3. allocator等待initial sync完成，先收集configured/current IDs，以及sync snapshot內既有monitor
   registrations/subscriptions引用的IDs；未知但仍被resource引用的ID只作tombstone reservation，不綁到family。
4. allocator從所有configured/current/reserved/tombstoned IDs的最大值加一開始。
5. candidate 通過 evaluation 後、bundle build 前，在catalog lock內reserve一個從未使用的非負ID。
6. stale/rejected promotion 可以留下未使用的 reserved ID；不得回收給另一個 artifact。
7. candidate manifest、Model Provision notification、Model Monitor registration/subscription/report 全部使用
   這個新 ID。
8. allocator超出implementation支援的integer range時promotion必須安全失敗，不得wrap或重用舊ID。

Catalog 至少維護：

```text
current_by_family[FamilyKey] -> CatalogVersion
version_index[ModelVersionKey] -> FamilyKey + generation + artifact
reserved_model_ids -> candidate/job
tombstoned_model_ids -> restored resource reference only
```

同一 provider process lifetime 內不得讓兩個不同 artifact 共用 ID。第一版 catalog 仍為 process-local，
所以完整 PyMTLF restart 視為實驗重跑；跨 restart 永久 allocator 與 catalog journal 留待後續 durability
設計，不在這次偷偷加入 SQLite。

### 9.3 Generation

- configured seed bundle 匯入後為 generation 1。
- candidate generation 為 current generation + 1。
- generation 只存在 PyMTLF process memory、catalog metadata 與 local bundle manifest。
- 不透過標準 Model Provision payload傳送 generation。
- 不在 Go 建立 generation mirror 或 CAS protocol。

Generation 是 family 內的順序；`modelUniqueId` 是標準 wire 上可觀察的 version identity。兩者不能再是同一個
概念，也不能用 `modelUniqueId + 1` 推導 generation。

### 9.4 Bundle 格式

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

- `model_family_id`
- 新的 `model_identity.model_unique_id`
- `model_generation`
- `parent_model_unique_id`
- `parent_artifact_key`
- training source
- training time window
- 每個 scope 的 sample count
- current/candidate validation summary

Configured seed 的FamilyKey以config為authoritative；不要求只為加入`model_family_id`重新打包既有seed。
若seed manifest已帶該欄位則必須與config一致；所有新candidate manifest都必須寫入family與parent identity。

manifest 不得保存：

- SUPI 清單
- raw targets 或完整 training records
- callback credential
- NRF/ADRF endpoint credential

### 9.5 Seed 與 retrained model 使用同一 catalog

目前只處理 seed 的 `SeedCatalog` 要泛化為 current model catalog：

- configured seed 是 generation 1 baseline。
- 每個 FamilyKey 有 current ModelVersionKey、generation、artifact、applicability 與 bundle metadata。
- 每個歷史 ModelVersionKey 可反查 FamilyKey；舊 version 保留為 retired metadata，不能再成為新 demand 的
  current selection。
- candidate 先 staged，通過驗證與 stale check 後才 atomic promote。
- Model Provision create 與 update notification 都在送出當下解析 family current version，不固定保存 seed
  tuple 或舊 model ID。

第一版 catalog 仍為 process-local。PyMTLF restart 後回到 configured seed baseline；磁碟上的 candidate
artifacts可以保留，但不得因檔案存在就自動視為 current model。

### 9.6 Stable monitoring scope identity

Accuracy scope 要在 M1 到 M2 的更新之間保持同一業務語意，因此 canonical scope key 固定由下列內容組成：

```text
consumerId or consumerSetId
+ mLEvent
+ canonical mLEventFilter
+ canonical tgtUe
```

`modelId`、registration ID、monitor subscription ID 與 `notifCorrId` 都不得放入 stable scope key。這些是某次
model version／resource lifecycle 的身分。Policy state 以 `(FamilyKey, StableScopeKey)` 管理，另存該 scope
目前採用的 `ModelVersionKey`。

---

## 10. Stale snapshot 與 promotion

### 10.1 Promotion 前重新確認

長時間訓練期間，provision demand 或 active monitoring scopes 可能改變。candidate promotion 前至少檢查：

1. family 的 catalog current ModelVersionKey/generation/artifact 仍是 training 開始時的 base。
2. FamilyKey 仍存在。
3. triggering scope 沒有被錯誤改綁到另一個 FamilyKey。

若 base 已被其他更新取代，candidate 視為 stale，不得覆蓋較新的 current model。

訓練期間新增或刪除 scope 不阻擋 promotion。例如 snapshot 建立時只有 group A/B，訓練期間又收到共用
相同 FamilyKey 的 group C subscription，candidate 仍可 promotion；C 沒有參與本次訓練/評估的事實必須記錄
warning。已刪除的 scope 也只記錄差異，不要求浪費已完成的訓練。

第一版 stale policy 為：

| 變化 | Promotion 行為 |
|---|---|
| base generation/artifact 已被另一更新取代 | 阻擋 |
| FamilyKey 已不存在或已無任何有效 demand | 取消 |
| triggering scope 改綁到另一 FamilyKey | 阻擋 |
| active scope 新增或刪除 | 記錄 structured warning，繼續 |

這是為了優先縮短 degradation recovery time。其限制是新加入 scope 可能套用一個未使用該 scope 資料驗證的
updated model，current experiment 明確接受此限制。

### 10.2 已確認 promotion 順序

```text
train candidate weights
    -> calculate per-scope acceptance
    -> reserve new modelUniqueId
    -> build bundle with new model identity/parent identity
    -> reload/validate bundle
    -> publish immutable artifact
    -> recheck base family/version/generation and record scope drift
    -> atomically promote family current version
    -> establish notification desired state
    -> release retrain-in-flight
```

artifact publish 在 promote 前完成；publish 成功但 stale check 失敗時，該 artifact 只是未被引用的 immutable
檔案，不得成為 current。

---

## 11. 更新模型的 Model Provision reconciliation

### 11.1 Existing provision resource

Phase 4 建立的 Model Provision subscription resource 繼續有效。PyMTLF 不為每次 retrain 建立新 resource；
對每個相容 resource 產生新的 desired notification：

- family current 的新 `modelUniqueId`
- 新的 immutable `mLModelUrl`
- resource 所需的 event/filter/target applicability
- resource 原有 `notifCorreId`

不放入 private generation、provider ID、training task ID 或 `modelUpdateInd`。

### 11.2 Desired state，不是無界 event queue

每個 provision resource 保存：

- resolved FamilyKey
- current desired ModelVersionKey/generation/artifact
- last delivered ModelVersionKey/generation/artifact
- retry attempt 與 next retry time
- resource mutation revision

若 generation N 尚未成功送達時 generation N+1 已成為 current，只需要送最新 desired state；舊通知可被
coalesce。不得建立無界 notification history queue。

callback transport failure 使用 capped exponential backoff。resource 被更新或刪除後，舊 revision 的 retry
必須取消，避免將過期 applicability 送到 consumer。

### 11.3 Completion boundary

PyMTLF 在以下條件成立時，將 training job 視為完成：

1. candidate 已驗證並成為 catalog current。
2. 所有當下相容 provision resources 都已建立最新 desired notification reconciliation。

不等待 PyAnLF 完成下載與 activation，因為標準 callback 的 `204` 不具有 activation acknowledgement 語意。
callback 暫時不可達時，notification reconciliation 繼續 retry，但不 rollback 已成為 current 的 PyMTLF
artifact。

Training completion 與 model adoption 是兩個不同狀態。PyMTLF 可在 job 完成後另以 family adoption state
觀察哪些 stable scopes 已透過新 `modelId` registration 採用 current version；adoption 不回寫成 training
job failure，也不需要新增 private acknowledgement。

---

## 12. PyAnLF candidate-first atomic replacement

### 12.1 Notification acceptance

PyAnLF 收到更新 notification 時：

1. 進行標準 schema 與 resource correlation 驗證。
2. 以 provision subscription、provider 與 canonical event/filter/target/use-case 建立本地
   `ModelSlotKey`。
3. 保存該 slot 最新 desired `modelUniqueId + model URL`。
4. 同一 slot 的較新 notification 可 coalesce 尚未 activation 的舊 candidate。
5. 以 bounded、coalescing worker 排入 model preparation。
6. 回 `204 No Content`。

下載、digest 驗證與 load 不在 callback request critical path 執行。

Current first version 每個 canonical slot只選用一個模型，不啟用 Model Provision 的 multiple-model optional
behavior。不同 `modelUniqueId` 但屬於同一 slot 時，視為同一 logical family 的候選更新，而不是第二個可並行
使用的模型。

### 12.2 Prepare once，commit together

同一 logical model slot 可能被多個 analytics subscriptions／monitor scopes 共用。更新時：

1. 依 slot/applicability 解析所有仍指向 old ModelVersionKey 的 runtime。
2. 在 runtime locks 外只下載並完整 prepare candidate 一次。
3. snapshot 各 runtime 的 expected revision/current ModelVersionKey/current artifact。
4. 以 deterministic order 取得必要 locks。
5. 再次檢查 resource/model/runtime 沒有 stale。
6. 一次將所有相依 runtime 的 model identity、artifact reference、model/scaler 切換到 candidate。
7. commit 成功後才 release old model references。
8. 清除舊 model ID 的 prediction、ground-truth、cached measurement 與 report-period state。
9. 重新計算 Model Monitor desired registrations；只有此時才會產生 candidate model ID 的 registration。

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

只要 candidate 尚未 atomic activation：

- runtimes 仍明確使用 old model ID；
- old Model Monitor registrations/subscriptions 繼續代表 old model；
- 若 PyMTLF 已將 family current promote 到 candidate，old report 只作 observability，不再觸發下一輪
  degradation；
- candidate registration 不得提前建立。

---

## 13. Accuracy report 與 activation cutover

### 13.1 不再使用 liveness barrier

標準 Model Monitor accuracy notification 沒有 artifact URL 或 generation。沒有 `deviation` 的 report 在
Phase 4 只表示該 periodic window 沒有足夠資料可計算 WAPE；它與 model activation 沒有規格關聯。Model
Provision callback 的 `204` 也只表示 notification 被接受。

因此下列舊語意明確廢止：

- pending 期間用 no-deviation report 表示 model update。
- 把第一筆 post-swap liveness 當 cutover barrier。
- PyMTLF 等待 `204 + liveness` 後才推測新 generation 已啟用。

資料不足時仍可照 Phase 4 傳送 no-deviation report，但只能更新一般 liveness/observability。

### 13.2 Registration-driven adoption

PyMTLF promotion 時建立 process-local adoption state：

```text
FamilyAdoptionState
  previous_model_id
  current_model_id
  expected_stable_scopes
  adopted_stable_scopes
  promoted_generation
```

它不進入 Go、不放入標準 payload，也不是 training job 的額外 blocking stage。各 scope 的狀態依標準
registration lifecycle 更新：

1. PyAnLF 尚未 activation 時，scope 仍只有 old model registration；old WAPE 可記錄但不得再驅動已
   promotion family 的 degradation policy。
2. PyAnLF atomic activation 後，monitor desired state 產生相同 stable scope、但 `modelId` 為 current
   candidate 的新 registration。
3. PyMTLF 接受新 registration 時，以 `modelId -> FamilyKey` index 驗證它是該 family 的 current version，
   將 scope 標為 adopted，並為新 registration 建立全新的 monitor subscription／`notifCorrId`。
4. 新 subscription 成功建立後，PyAnLF 才會對新 model ID 建立 measurement/report binding。
5. 新 model 的第一筆 WAPE 需等待正常的 minimum matched predictions；不需要先送 liveness barrier。
6. old registration/subscription 隨 desired reconciliation 刪除。任何延遲 old callback即使到達，也因
   retired correlation、old model ID 或非current owner而不得更新 current policy。

Stable scope 的新 WAPE 可在該 scope adopted 後立即建立新 baseline，不必等待其他 scope。當所有仍有效的
expected scopes 都 adopted 或已被刪除，family adoption state 可移除。新加入的 scope若直接使用 current
model，正常建立 current model registration即可，不阻擋既有 adoption。

### 13.3 Resource ownership 與 callback validation

PyMTLF 接受 accuracy notification 前必須同時確認：

- `notifCorrId` 對應 active monitor subscription projection；
- 該 subscription 仍由 active registration 擁有；
- notification `modelId` 在 subscription `modelIds` 內；
- registration、subscription、notification 的 event/filter/target scope一致；
- model ID 可由 catalog version index 對應到預期 FamilyKey；
- 該 stable scope目前採用的 model ID與 notification一致。

任一條不成立時不得把 deviation 送入 WAPE policy。Well-formed但 correlation/resource 已刪除的 callback
依既有標準 route回 `404`；若 race 已穿過 transport validation，PyMTLF business layer仍要 drop stale
observation並記錄 structured log。

Restart sync匯入但catalog無法識別的tombstoned model ID，不是current version、不能算adopted，也不能驅動
新monitor subscription或policy；既有unknown subscription projection依orphan reconciliation清除，等待
PyAnLF經seed reprovision重新建立可識別registration。

### 13.4 Multi-scope atomicity

同一 PyAnLF 內共用 model slot 的 runtimes仍必須原子切換 M1 到 M2；registration POST/DELETE 本身可由
reconciler逐筆收斂。短暫同時存在 R1與R2是可接受的，因 model IDs及correlations不同。已確認由reconciler
先成功建立R2/S2，再清除R1/S1，以避免不必要的monitoring gap；但只有atomic runtime commit後才可建立R2。

### 13.5 Local state machines

PyAnLF model slot只需下列本地狀態，不新增API欄位：

```text
CURRENT(M1)
  -> PREPARING(M2, expected current M1)
      -> CURRENT(M2), then reconcile R2/S2
      -> CURRENT(M1) on validation/load/stale failure, retry latest desired
```

PyMTLF family adoption只需：

```text
CURRENT(M1)
  -> PROMOTED(M2, expected scopes)
      -> scope-by-scope ADOPTED(M2) when R2 arrives
      -> CURRENT_OBSERVED(M2) when valid M2 WAPE begins
```

Model Provision delivery failure不把M2 rollback成M1；PyAnLF activation failure也不虛構R2。這兩個狀態機靠
標準resource的實際存在收斂，而不是靠callback timing猜測。

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

- 同一 FamilyKey 永遠只允許一個 retrain-in-flight。
- 第一版整個 PyMTLF 預設最多一個 concurrent training job。
- scheduler 使用 bounded queue；queue 滿時不得無聲覆蓋 job。
- Phase 4 對同一 family 的後續 degradation 在 in-flight 期間只更新必要 observability，不建立重複 dataset
  retrieval/training。

---

## 15. 失敗、重試與 restart

### 15.1 Promotion 前失敗

dataset conversion、training、validation、packaging 或 stale check 失敗時：

- current catalog 與現有 PyAnLF model 不變。
- job 進入 FAILED。
- Phase 5 snapshot 記錄 terminal outcome。
- 呼叫調整後的 `complete_retrain` 釋放 family-level in-flight。
- 不自動用完全相同 snapshot 無限重訓；後續 degradation 可用新的 historical window 再觸發。

### 15.2 Promotion 後 callback 暫時失敗

- PyMTLF current model 不 rollback。
- desired notification 保留並 capped backoff retry。
- PyAnLF 繼續使用舊模型。
- PyMTLF 以 old `modelId` 明確辨識仍來自舊模型的 accuracy，保留 observability但不再觸發該 family 的
  degradation。
- PyAnLF 成功 activation 並建立新 `modelId` registration 後，PyMTLF 才為該 stable scope 接受新 baseline。

### 15.3 Restart

第一版明確接受 process-local state：

| Restart | 行為 |
|---|---|
| PyMTLF restart | family catalog、allocator、training/current generation state 遺失，回到 configured seed baseline；實驗重跑 |
| PyAnLF restart | Go sync 恢復 subscription snapshot，PyMTLF 依 current catalog 重新提供模型；PyAnLF cache 可重用或重下載 |
| Go restart | process-local mirror 遺失，依現有 backend reconnect/sync 重建；實驗可重跑 |

本階段不新增 SQLite、journal、cross-process transaction 或 crash recovery database。

因 model ID allocator 也是 process-local，不能宣稱 promoted model identity 可跨獨立 PyMTLF restart
延續。完整 restart 測試必須以重跑實驗及 seed reprovision 為預期，不得讓殘留 old registration 被誤認為
restart 後新配發的 model。Initial sync看到的unknown model IDs必須先tombstone reserve，直到對應resources
被reconciliation清除；本process仍不得重新配發這些IDs。永久catalog/allocator是後續durability工作。

---

## 16. Config

已確認的 PyMTLF config target：

```yaml
model_provision:
  provider_namespace: "local-mtlf"
  seed_models:
    - family_id: "ue-communication-default"
      model_id: 1
      artifact_key: "<sha256>"
      event: "UE_COMMUNICATION"
      event_filter: {}

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
- non-empty、provider 內唯一的 `family_id`
- provider 內唯一的 configured seed `model_id`
- writable artifact root
- public base URL 與現有 artifact server 一致
- bundle size/download timeout 沿用既有較寬鬆網路預設
- queue/concurrency 不得為零或負數

---

## 17. 實作切片

本階段完成後再統一 review，不要求每個切片各自 commit。

現有實作到target design的主要migration map：

| 現有概念 | Target |
|---|---|
| PyMTLF `ModelKey = (provider, modelId)` | `FamilyKey`與`ModelVersionKey`分離 |
| `CatalogModel.descriptor.model_id`永遠不變 | catalog current version每次promotion換新model ID |
| `ProvisionResource.model_keys` | `family_keys`，send-time解析current version |
| `DatasetSnapshot.model_key`／retrain-in-flight | family key |
| Accuracy scope key包含subscription `modelIds` | stable scope排除model ID，另存active version |
| `CutoverState`等待provision delivery+liveness | `FamilyAdoptionState`觀察new registration |
| PyAnLF active model只以provider/model ID索引 | ModelSlotKey管理family-like applicability，version identity另存 |
| same-ID/new-URL replacement | same-slot/new-ID/new-URL atomic identity replacement |
| replacement pending省略deviation | 移除；old/new model由registration與correlation自然隔離 |

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

- 將既有 ModelKey 拆成 FamilyKey 與 ModelVersionKey
- SeedCatalog 泛化為 family/current-version catalog
- process-local model ID allocator 與 version index
- generation/staged/promote
- deterministic bundle builder
- manifest 的 family/new identity/parent identity
- digest/reload validation
- immutable artifact publication
- stale base/model check 與 non-blocking scope-drift warning

### Slice D：Updated Model Provision reconciliation

PyMTLF：

- provision resource 保存 FamilyKey，並在 send-time 解析 family current version
- desired/delivered ModelVersionKey/generation/artifact state
- retry、coalescing 與 resource revision cancellation
- stable scope key 排除 model ID
- current registration adoption state 與 old model report rejection
- active registration/subscription/correlation ownership validation

NWDAF：

- 以 contract/regression tests 確認 new model ID/new URL payload lossless forwarding
- 確認 standard status 與 callback mapping 未改變
- 除非測試發現既有 routing defect，否則不新增 production business logic

### Slice E：PyAnLF atomic replacement

PyAnLF：

- 以 ModelSlotKey 將 new identity/new URL 視為同一 family 的 candidate update
- latest desired model identity/artifact coalescing
- prepare-once
- multi-runtime deterministic atomic identity/artifact swap
- failure retains old
- successful swap 後清除 inference/accuracy windows
- activation 後才建立 new model ID registration
- new registration/subscription 建立前不產生新 model WAPE；old resource cleanup 可非同步收斂

### Slice F：Process-level verification 與文件

nwdaf-resources：

- 啟動 Go、PyAnLF、PyMTLF 的真實 process harness
- 建立兩個不同 group/scopes，共用同一 logical model family/current artifact
- 觸發其中一個 scope degradation
- 驗證 snapshot 建立當下所有 active scope data 進入 dataset
- 驗證 candidate training 與 per-scope gate
- 驗證更新通知具有新 model ID、direct download 與 atomic multi-scope identity switch
- 驗證新 registrations/subscriptions/correlations 只在 activation 後建立
- 驗證 delayed old model/correlation report 不會進入 current policy
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
16. 每次 successful promotion 配發新的 model ID；同一 family current version 正確前進。
17. version index 可由 seed、current 及 retired model ID 反查同一 FamilyKey。
18. stale base generation/version 不得 promote；scope 新增/刪除只記錄 warning。
19. provision resource 跨 promotion 保持 FamilyKey，notification retry coalesces 到最新 ModelVersionKey。
20. resource update/delete 取消 stale retry。
21. stable scope key 不含 model ID；M1/M2 registrations 對應同一業務 scope。
22. old M1 report、retired correlation 或錯誤 owner 不得更新 current M2 policy。
23. M2 registration 建立後，該 scope 才開始 M2 baseline；不依賴 liveness barrier。

### 18.2 PyAnLF unit/contract

1. 同 provision slot 的 new `modelUniqueId`/new URL 進入 replacement path。
2. callback 在保存與 enqueue 後回 204，不等待下載。
3. candidate 只 prepare 一次並提供給所有相依 runtimes。
4. 多 runtime 的 identity、artifact、model/scaler 全部成功才 commit。
5. 任一 validation/load/stale failure 時全部保留舊模型。
6. candidate 尚未 activation 時只保留 old model registration，不提前建立 new registration。
7. swap 後 prediction、ground-truth、cached measurement 與 report windows 全部重設。
8. swap 後 desired registration 從 M1 改為 M2，建立新 registration/subscription/correlation。
9. old monitor subscription 不再匹配 M2 runtime，不能送出錯誤 model ID report。
10. M2 穩定累積足夠資料後直接送 WAPE，不要求先送 no-deviation barrier。
11. M2 準備失敗時 M1 inference、registration 與 WAPE 路徑維持可用。
12. 連續收到 M2、M3 時同 slot coalesce 最新 desired candidate，不交錯 commit。

### 18.3 NWDAF regression

1. Model Provision notification 的 standard fields lossless round-trip。
2. 新 model ID/new URL 不被 Go 誤判為 duplicate old notification。
3. backend unavailable、peer error 與 callback error 保持 Phase 4 status mapping。
4. Go 不解析 bundle、generation 或 training metadata。

### 18.4 Process-level E2E

最少涵蓋：

```text
two analytics subscriptions
  -> same logical model family and model M1
  -> two independent monitor scopes
  -> one scope degradation
  -> one family-level retrain
  -> all eligible snapshot scopes included
  -> local candidate accepted
  -> immutable updated artifact with model M2
  -> standard Model Provision notify
  -> PyAnLF direct download
  -> atomic identity/artifact replacement for both scopes
  -> new M2 registrations/subscriptions/correlations
  -> new stable WAPE reports tagged M2
  -> delayed M1 report cannot enter M2 policy
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
4. scope 新增或刪除不阻擋 promotion，只記錄 scope drift；base family/version/generation stale 仍必須阻擋。
5. triggering scope 資料不足時 job 失敗；其他 scope 資料不足時依 training/evaluation 資格降級處理並告警，
   不使整個 job 失敗。

### D2. Training completion 邊界

採用 asynchronous callback：

- `204` 只表示 PyAnLF 已接受 notification。
- PyMTLF 在 artifact promotion 並建立 notification desired-state reconciliation 後完成 job。
- 不等待 PyAnLF activation，不新增 private ack。
- PyAnLF activation 後以標準 Model Monitor Register 表示開始使用新 model ID。
- PyMTLF 以新 model ID、registration ownership 與新 correlation 判斷 observation 歸屬，不以 liveness
  推測 activation。

不新增 private activation acknowledgement。

### D3. Warm start

從 current model weights warm start，並對新 training-period data 重新 fit scaler；第一版不使用 fresh
initialization。

### D4. Model identity 與 family

1. 同一 logical model family 在 PyMTLF 以穩定 FamilyKey 管理。
2. 每個 successful promoted artifact 使用新的標準 `modelUniqueId`；多個 scopes若共用同一 artifact，
   仍使用相同 model ID。
3. Provision resource、retrain intent、dataset snapshot 與 single retrain-in-flight 綁 FamilyKey。
4. Monitor registration/subscription/report 綁當代 ModelVersionKey，stable scope key排除 model ID。
5. no-deviation report 只保留 Phase 4 的資料不足/liveness語意，不再作 cutover barrier。

### D5. FamilyKey 與 applicability ownership

1. `FamilyKey = (provider_namespace, family_id)`；`family_id`由PyMTLF seed config明確指定。
2. 不從event/filter/target自動hash或推導family ID。
3. Applicability descriptor作為family catalog資料另存，至少包含event、event filter、target、
   interoperability與use-case context。
4. Seed config/descriptor是runtime selection的authoritative來源；bundle manifest只保存可選的驗證snapshot。
5. Retrained versions繼承family applicability。第一版若applicability語意不同，建立新family，不在promotion
   中修改既有family。

### D6. Allocator、adoption與第一版限制

1. Model ID allocator維持process-local、provider-wide monotonic reservation；同一process不回收或重用ID。
2. Initial sync引用的unknown model IDs先tombstone reserve；永久catalog/allocator不在本階段實作。
3. PyMTLF restart回到configured seed並視為實驗重跑。
4. Stable scope在自己的current-model registration建立後即可開始新baseline，不等待其他scopes。
5. Promotion後尚未activation的old-model reports只保留observability，不再觸發family degradation。
6. R2/S2先成功建立，再刪除R1/S1。
7. 第一版每個PyAnLF ModelSlot只使用一個current model，不啟用multiple-model optional behavior。

---

## 21. 完成條件

本階段只有在以下條件全部滿足時才算完成：

1. Phase 5 READY dataset 能被單次 claim 並進入 bounded local training。
2. Triggering scope 必須具備完整 training/reference-validation 資料；其他 snapshot scopes 依實際資料資格
   加入單次 training/evaluation，任何排除都有 structured log 與 manifest record。
3. Performance evaluation 永遠產生；是否因退步拒絕 candidate 由 `enforce_performance_gate` 控制，技術正確性
   與 stale base checks 永遠不可關閉。
4. PyMTLF 建立可由 PyAnLF 直接下載的 immutable、可重載、digest 驗證 bundle。
5. updated model 以新的 `modelUniqueId` 透過既有標準 Model Provision resource 送出，不含自創 wire欄位。
6. PyMTLF 可在 FamilyKey、ModelVersionKey與ArtifactKey間正確對應，provision/retrain ownership不因
   model ID更新而斷裂。
7. PyAnLF 對所有相依 runtime 做 atomic identity/artifact replacement，任何失敗皆保留舊模型。
8. 新 Model Monitor registration 只在 activation 後建立；PyMTLF 以新 registration/subscription/correlation
   接受新模型 WAPE。
9. old model ID、retired correlation及delayed callback都不會污染current model policy，且不依賴liveness
   barrier。
10. Go 仍只負責 SBI validation/routing，沒有 training、family 或 artifact business logic。
11. PyMTLF、PyAnLF、NWDAF 各自 lint/unit/contract tests 通過。
12. nwdaf-resources 的真實三 process E2E 通過並有完整操作說明。
13. 文件記錄實際 config、API、限制、驗證結果與剩餘 Phase 7 工作。

---

## 22. Current Implementation Result

2026-07-25 已完成新版 FamilyKey／per-artifact model identity／registration-driven activation重構，
並通過unit、lint、Go routing及真實三process local-training驗證。

### 22.1 PyMTLF

- `SeedCatalog` 已拆成穩定`FamilyKey`、每代`ModelVersionKey`、current-by-family及version index。
  Configured seed為generation 1；candidate在打包前保留provider-wide新model ID，promotion使用base
  generation與artifact digest CAS，Model Provision representation在送出時解析family current version。
- Phase 5 `READY` snapshot 具有 `CLAIMED` 與 terminal state，單一 snapshot 只會被一個 bounded training
  job 消費；terminal success、failure 或 cancellation 都會釋放 family-level retrain-in-flight。
- 新增 ADRF-aligned raw notification 轉換、固定十個 feature 的 timestamp/scope aggregation、較早 20%
  reference validation、較新 80% training、purge gap、training-only scaler fit 與資料資格紀錄。
- 新增 CPU-only deterministic warm-start trainer，使用 Adam、Huber loss，並永遠計算 current/candidate
  per-scope 與 aggregate WAPE。`training.enforce_performance_gate` 控制 regression 是否阻擋 promotion，
  技術驗證與 stale checks 不可關閉。
- Candidate 以既有四檔 bundle contract 打包，manifest 記錄 parent artifact、generation、time window、
  datasource、scope eligibility、loss 與 validation summary；發布前後都會 reload/validate。
- Promotion 與「當下仍有 provision demand」在同一 resource guard 內確認。Promotion 後對每個相容 resource
  建立 latest-desired notification；retry 會 coalesce 新 artifact、採 capped exponential backoff，resource
  revision 更新或刪除會取消舊 delivery。
- Accuracy policy已移除liveness-based cutover state。Retired model ID與舊correlation不更新current
  policy；每個scope只在new model registration建立後採用新version並重設baseline。
- MongoDB client 使用 timezone-aware BSON datetime，避免合法 UTC measurement 在訓練轉換時被誤判為
  naive timestamp。

### 22.2 PyAnLF

- 同一 ModelSlotKey 的new identity/new URL會進入background candidate replacement；slot由provider、
  event、filter、target與use-case context組成，不出現在3GPP payload。
- Candidate 只 prepare 一次，所有選定 runtime 在同一 commit 內all-or-nothing切換identity與artifact；
  download、validation、load 或 stale runtime state 失敗時全部保留舊模型並由既有 worker retry。
- Replacement liveness gate已移除。Activation後desired monitor state改為new model ID；reconciler先建立
  新registration，再刪除舊registration，正常足量WAPE由新subscription/correlation承載。
- 相同 Events Subscription snapshot 的週期性 sync 現在是 runtime no-op；同一 provision resource 下，
  本地已接受但尚未套用的 candidate 不會被 snapshot 中的舊 immediate model 覆蓋。
- Monitor registration create 與較舊 sync snapshot 的競爭以 unconfirmed identity 收斂，避免相同
  canonical scope 被重複 POST 並讓後續 sync 因 duplicate scope 失敗。

### 22.3 Go NWDAF

- Production routing 未增加 training、generation 或 artifact business logic。
- Regression test已涵蓋「新`modelUniqueId`、新`mLModelUrl`」，並確認Go只做lossless forwarding。

### 22.4 Process-level harness

`nwdaf-resources/tests/mtlf_model_monitor/` 新增 MongoDB-backed local-training scenario。它實際啟動：

- Go NWDAF
- PyAnLF
- PyMTLF
- MongoDB

目前驗證兩個 SUPI scope 共用同一family、任一 scope degradation觸發單一family-level retrain、兩個
scope都進入snapshot、generation 2／model ID 2 candidate發布、第一次artifact GET刻意回`503`、舊
runtime保留、retry後兩個runtime原子切換、activation後registration/subscription重建，以及新
correlation以model ID 2建立fresh baseline。

### 22.5 實際驗證結果

- PyMTLF：`ruff check` 通過；`pytest` 為 `92 passed`。
- PyAnLF：`ruff check` 通過；`pytest` 為 `245 passed, 1 skipped`。
- NWDAF：`go test ./...`、`golangci-lint run ./...`、`go build -o bin/nwdaf ./cmd/` 通過；
  `go test -race ./internal/sbi/processor ./internal/mtlf ./internal/anlf/...` 通過。
- nwdaf-resources：
  `TestInitialModelProvisionAndMonitorRoundTrip` 通過；
  `TestLocalTrainingAndUpdatedModelRoundTrip` 通過。

測試使用真實 runtime processes 與 MongoDB，但 NRF、SMF、UPF、ADRF 仍為 fixture 或未啟動，因此不宣稱
real free5GC deployment E2E、ADRF retrieval E2E、TLS/OAuth 或 restart durability。PyMTLF current
family catalog、model ID allocator及generation仍依本文件決策為process-local，restart回到configured
seed baseline。
