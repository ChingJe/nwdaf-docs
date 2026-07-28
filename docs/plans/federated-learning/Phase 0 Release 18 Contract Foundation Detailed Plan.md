# Phase 0 Release 18 Contract Foundation Detailed Plan

日期：2026-07-28

狀態：實作與驗證完成（包含 final validation contract correction）

上層計畫：

- [Distributed NWDAF Federated Learning Implementation Plan](./Distributed%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)

相關設計與規格解讀：

- [Distributed NWDAF Model Monitoring and Federated Retraining Architecture](../../design/Distributed%20NWDAF%20Model%20Monitoring%20And%20Federated%20Retraining%20Architecture.md)
- [NWDAF AnLF／MTLF Current Feature Architecture](../../design/Current%20AnLF%20MTLF%20Feature%20Architecture.md)
- [NWDAF Federated Learning Release 18 規格解讀](../../specification-guides/NWDAF%20Federated%20Learning%20Release%2018%20規格解讀.md)
- [NWDAF Development Policy](../../development_policy.md)

實作 repositories：

- `NWDAF/`
- `PyAnLF/`
- `PyMTLF/`
- `nwdaf-resources/`
- `nwdaf-docs/`

---

## 1. 目的

Phase 0 建立後續 distributed NWDAF FL phases 共用的 contract
foundation，讓 Go、PyAnLF、PyMTLF 與整合資源對下列內容使用同一套
語意：

1. Release 18 `Nnwdaf_MLModelTraining` wire models；
2. Release 18 `ReportingInformation`；
3. NRF `MlAnalyticsInfo` 的 FL capability fields；
4. Training request 的 scope descriptor；
5. FL 暫存與 final model artifact roles；
6. completed model catalog 與 pending publication journal schema；
7. path、method、media type、status code、header 與 conditional validation
   matrix。

這個 Phase 的核心成果是「可被後續 runtime 直接使用的 typed contract」，
不是一個能執行 FedAvg 的半成品。

完成後應能回答：

- Go 與 PyMTLF 對同一個 Training request 是否得到相同欄位與驗證結果；
- 哪些欄位來自 Release 18，哪些是 project-private state；
- 哪些暫存 artifact 不得取得正式 `modelUniqueId`；
- catalog 與 ADRF publication 的 durable state 長什麼樣；
- 後續 Phase 1、3、4、5 應依哪一個 schema 實作，而不需要再次發明
  contract。

---

## 2. 實作基線與證據邊界

### 2.1 Repository baseline

| Repository | Baseline |
| --- | --- |
| `NWDAF/` | `2bdb01e` |
| `PyAnLF/` | `694e3d8` |
| `PyMTLF/` | `11b3199` |
| `nwdaf-resources/` | `4937b20` |
| `nwdaf-docs/` | `467c5b0` |

計畫撰寫時各 repository worktree 均為 clean。

### 2.2 Normative evidence

Stage 3 contract 以 local Release 18 OpenAPI 為主：

- [TS 29.520 Nnwdaf ML Model Training OpenAPI](../../../specs/openapi/TS29520_Nnwdaf_MLModelTraining.yaml)
- [TS 29.520 Nnwdaf ML Model Provision OpenAPI](../../../specs/openapi/TS29520_Nnwdaf_MLModelProvision.yaml)
- [TS 29.520 Nnwdaf ML Model Monitor OpenAPI](../../../specs/openapi/TS29520_Nnwdaf_MLModelMonitor.yaml)
- [TS 29.523 ReportingInformation source OpenAPI](../../../specs/openapi/TS29523_Npcf_EventExposure.yaml)
- [TS 29.510 NRF NFManagement OpenAPI](../../../specs/openapi/TS29510_Nnrf_NFManagement.yaml)
- [TS 29.571 Common Data OpenAPI](../../../specs/openapi/TS29571_CommonData.yaml)
- [TS 29.520 ML Model Training Data Model](../../../specs/TS%2029.520/5%20API%20Definitions/5.5%20Nnwdaf_MLModelTraining%20Service%20API/5.5.6%20Data%20Model.md)
- [TS 29.520 ML Model Training Error Handling](../../../specs/TS%2029.520/5%20API%20Definitions/5.5%20Nnwdaf_MLModelTraining%20Service%20API/5.5.7%20Error%20handling.md)
- [TS 29.510 MlAnalyticsInfo](../../../specs/TS%2029.510/6%20API%20Definitions/6.1%20Nnrf_NFManagement%20Service%20API/6.1.6%20Data%20Model/6.1.6.2%20Structured%20data%20types/6.1.6.2.84%20Type%20MlAnalyticsInfo.md)
- [TS 29.510 FlCapabilityType](../../../specs/TS%2029.510/6%20API%20Definitions/6.1%20Nnrf_NFManagement%20Service%20API/6.1.6%20Data%20Model/6.1.6.3%20Simple%20data%20types%20and%20enumerations/6.1.6.3.19%20Enumeration%20FlCapabilityType.md)

OpenAPI 決定：

- JSON property name；
- required／optional；
- array cardinality；
- path、method 與 media type；
- success status；
- `Location` header；
- declared common errors。

TS table 另決定只靠 OpenAPI `required` 無法表達的 conditional
requirements 與 service-specific cause。

### 2.3 Implementation-shape evidence

本 Phase 不新增 public Training handler，但 contract package 形狀仍遵循：

- `NWDAF/internal/compat/mlmodel/` 的既有 compatibility ownership；
- `NWDAF/internal/sbi/api_ml_model.go` 的 raw-body forwarding 與標準
  response boundary；
- pinned free5GC generated model 優先、缺欄才以 isolated compatibility
  type 補齊的原則；
- free5GC UDM 的 generated-client／generated-model ownership；
- free5GC PCF／UDM callback API 的 handler、processor 分層，供 Phase 3
  真正建立 Training resource 時使用。

本 Phase 不因為規格 YAML 已存在，就把整份 OpenAPI 重新生成到 fork。

---

## 3. 現況盤點

### 3.1 Go NWDAF

目前：

- `go.mod` 使用 `github.com/free5gc/openapi v1.2.3`；
- `internal/compat/mlmodel/` 已包含 Provision／Monitor 的部分 Release 18
  wire models；
- Provision／Monitor public SBI 與 backend gateway 會保留原始 JSON body，
  並用 compatibility parser 做必要驗證；
- 尚無 `internal/compat/mlmodeltraining/`；
- 尚無 Training public route、processor、consumer、callback route 或
  resource state；
- generated `models.ReportingInformation` 缺少 Release 18
  `notifFlagInstruct` 與 `mutingSetting`；
- generated `models.MlAnalyticsInfo` 只有：
  `mlAnalyticsIds`、`snssaiList`、`trackingAreaList`；
- generated `models.MlAnalyticsInfo` 缺少：
  `mlModelInterInfo`、`flCapabilityType`、`flTimeInterval`、
  `nfTypeList`、`nfSetIdList`。

### 3.2 PyMTLF

目前：

- `src/py_mtlf/wire/ml_model.py` 與
  `src/py_mtlf/wire/ml_model_monitor.py` 只涵蓋 Provision／Monitor；
- `ReportingInformation` 只含 `immRep`；
- 尚無 Training Pydantic models；
- completed bundle 固定為
  `config.json + model.py + model.npy + scaler.pkl`；
- `ArtifactRepository` 只接受含正式 model identity 的 completed bundle；
- current `ModelCatalog` 是由 seed config 建立的 process-memory state；
- current local training candidate 在 build 時已先配置正式 model ID；
- 尚無 completed catalog file 或 pending publication journal。

### 3.3 PyAnLF

目前：

- Provision、Monitor 與 Events Subscription wire models 集中在
  `src/py_anlf/models.py`；
- `ReportingInformation`、`MLModelProvisionReporting` 與
  `MonitorReportingRequirement` 各自只表達部分欄位；
- 本 Phase 不需要 PyAnLF 理解 Training API；
- PyAnLF 只需要對齊它會收發的 Provision、Monitor 與共用 reporting
  contract。

### 3.4 nwdaf-resources

目前：

- 沒有 `contracts/`；
- cross-process tests 已集中於此 repository；
- 不應把共享 contract JSON 放回各 NF repository 的 root `testdata/`。

---

## 4. Phase 0 範圍

### 4.1 必須完成

- Go Release 18 typed compatibility models；
- Go parser 與 pure validation functions；
- PyMTLF equivalent Training wire models 與 validation；
- PyAnLF／PyMTLF shared `ReportingInformation` alignment；
- NRF R18 FL capability representation；
- canonical `TrainingScopeDescriptor`；
- shared supported-profile JSON examples；
- FL artifact role manifest contract；
- completed catalog／publication journal contract；
- code-native unit tests；
- contract ownership與規格證據文件。

### 4.2 明確不做

- 不註冊新的 NRF capability；
- 不修改 NRF matching；
- 不建立 `nnwdaf-mlmodeltraining` listener routes；
- 不建立 outbound Training consumer；
- 不建立 Training subscription state 或 callback routing；
- 不啟動 preparation、local training、FedAvg 或 validation worker；
- 不向 ADRF store／retrieve final model；
- 不替 current local trainer 改成 FL trainer；
- 不把 current in-memory `ModelCatalog` 切換成新 persistent catalog；
- 不修改 PyAnLF analytics、WAPE 或 model activation behavior；
- 不建立 multi-NWDAF E2E；
- 不建立 OpenAPI fork、`replace` 或額外 Go module dependency。

### 4.3 Phase 0 與後續 Phase 的邊界

```text
Phase 0
  typed contracts + pure validation + schema fixtures
       |
       +-> Phase 1: NRF registration/discovery runtime
       +-> Phase 3: Training SBI/resources/preparation
       +-> Phase 4: rounds/FedAvg/artifact production
       +-> Phase 5: validation/ADRF publication/catalog promotion
```

Phase 0 可以建立後續要 import 的 types 與 pure helpers，但不得註冊沒有
runtime owner 的 route，也不得新增永遠回 `501`／`503` 的 placeholder
endpoint。

---

## 5. Contract ownership strategy

### 5.1 Go ownership

| Concern | Owner |
| --- | --- |
| 可完整表達的 R17 common／NWDAF nested type | pinned `free5gc/openapi/models` |
| Release 18 ReportingInformation additions | `internal/compat/mlmodel` |
| Release 18 Training service models | `internal/compat/mlmodeltraining` |
| Release 18 NRF FL capability additions | `internal/compat/nrf` |
| public HTTP handler/resource state | Phase 3 `internal/sbi`，不屬於本 Phase |
| FL policy／worker | PyMTLF，Go 不擁有 |

不得：

- 複製已足夠的 generated type；
- 修改 Go module cache 或 upstream generated files；
- 在 handler 內宣告一次性 request struct；
- 用 `map[string]any` 取代已確定的 standardized schema；
- 把 project-private `sampleCount`、WAPE sums 或 artifact role 加入
  standard Training body。

### 5.2 Compatibility wrapper pattern

NRF profile 仍應重用 generated profile 的既有欄位，只覆蓋無法表達的
Release 18 subtree：

```text
compat/nrf.NFProfile
  embeds generated models.NrfNfManagementNfProfile
  replaces nwdafInfo with compat/nrf.NwdafInfo

compat/nrf.NwdafInfo
  embeds generated models.NwdafInfo
  replaces mlAnalyticsList with []compat/nrf.MlAnalyticsInfo

compat/nrf.MlAnalyticsInfo
  embeds/reuses the generated R17 fields
  adds the Release 18 fields
```

實作必須以 marshal／unmarshal tests 確認：

- JSON 中沒有重複 `nwdafInfo` 或 `mlAnalyticsList`；
- R17 base fields 沒有遺失；
- R18 extension fields 沒有被丟棄；
- generated and compatibility ownership 沒有形成兩套互相矛盾的值。

Phase 1 若 generated NRF client 無法接受 compatibility wrapper，才在
isolated NRF consumer 內使用同一個 typed wrapper 編碼 request；不得因此
建立完整 OpenAPI fork。

### 5.3 Python ownership

- PyMTLF：
  - 完整 Training wire model；
  - FL-specific conditional validation；
  - scope descriptor；
  - artifact／catalog／journal private schemas。
- PyAnLF：
  - Provision／Monitor／ReportingInformation；
  - 不新增 Training types。
- standard wire model 保留 `extra="allow"`，以便 forward-compatible
  extension 不會在 parse／dump 時消失；
- private durable schema 使用 `extra="forbid"`，避免 state file
  typo 被靜默接受。

### 5.4 Complex nested standard subtrees

`EventFilter` 等跨檔案 schema 很大，且 pinned generated module 沒有完整
Release 18 type。Phase 0 採下列界線：

- resource-level Training／Provision／Monitor models 必須 typed；
- 本計畫會用到且目前 generated module 已可表達的 nested type直接重用；
- 本計畫 supported profile 需要判斷的 nested fields 建立 typed projection；
- 其餘完整標準 subtree 在 Go 使用 `json.RawMessage`、Python 使用
  `dict[str, JsonValue]`，並要求 JSON object、保留原始內容；
- 不用 `map[string]any` 承擔 resource-level contract；
- 不因 Phase 0 複製整份 `EventFilter` 及其所有跨規格 dependencies。

supported-profile typed projection 至少涵蓋：

- `networkArea`／TAI；
- `snssais`；
- `appIds`；
- `dnns`；
- `tgtUe`；
- `mLTargetPeriod`；
- `modelInterInfo`。

Go public SBI 原始 body 仍是 lossless forwarding 的 authoritative payload；
typed projection負責驗證與後續 policy input。

### 5.5 Standard and private identity

`modelProviderId` 不是 Release 18 `MLEventNotif` field。Phase 0 必須：

- 不再把它宣告在 standard wire model；
- 將 containing-provider identity 保留於 PyAnLF private demand／provision
  context；
- 保持目前單一 NWDAF 的 provider selection behavior；
- 不提前實作 Phase 2 的 remote `SelectedTarget`。

同理，下列只屬於 project-private contract：

- exact `sampleCount`；
- WAPE component sums；
- artifact role；
- catalog revision；
- publication journal state。

---

## 6. Shared Release 18 ReportingInformation

### 6.1 Exact fields

`ReportingInformation` 由 TS 29.523 OpenAPI 定義：

| JSON field | Type | Presence |
| --- | --- | --- |
| `immRep` | boolean | optional |
| `notifMethod` | extensible notification method string | optional |
| `maxReportNbr` | unsigned integer | optional |
| `monDur` | timezone-aware date-time | optional |
| `repPeriod` | duration seconds | optional |
| `sampRatio` | sampling ratio | optional |
| `partitionCriteria` | non-empty array | optional |
| `grpRepTime` | duration seconds | optional |
| `notifFlag` | extensible notification flag string | optional |
| `notifFlagInstruct` | `MutingExceptionInstructions` | optional |
| `mutingSetting` | `MutingNotificationsSettings` | optional |

`MutingExceptionInstructions`：

- `bufferedNotifs`：
  `SEND_ALL`／`DISCARD_ALL`／`DROP_OLD` 或 future extension；
- `subscription`：
  `CLOSE`／`CONTINUE_WITH_MUTING`／`CONTINUE_WITHOUT_MUTING` 或 future
  extension。

`MutingNotificationsSettings`：

- `maxNoOfNotif`；
- `durationBufferedNotif`。

### 6.2 Presence semantics

optional scalar 必須能區分：

- 欄位不存在；
- 欄位存在且為 `false`；
- 欄位存在且為 `0`。

因此：

- Go 使用 pointer scalar；
- Python 使用 `T | None`，default 為 `None`；
- 不再用 `false`／`0` 當 wire model 的 presence default；
- business logic 可以在 wire parse 後另行套用 default，不改寫原始
  representation。

### 6.3 Reuse points

同一個 typed contract 由下列欄位共用：

- Model Provision `eventReq`；
- Model Monitor `eventReportReq`；
- Model Training `eventReq`。

PyAnLF 與 PyMTLF 不再維護三種欄位集合不同的 reporting DTO。

第一版 FL flow 不主動設定 one-time reporting，但合法的 one-time／muting
欄位仍必須可 parse、validate 與 dump。

---

## 7. ML Model Training wire contract

### 7.1 Resource operations

Base path：

```text
/nnwdaf-mlmodeltraining/v1
```

| Operation | Request media type | Success |
| --- | --- | --- |
| `POST /subscriptions` | `application/json` | `201` + representation + required `Location` |
| `PUT /subscriptions/{subscriptionId}` | `application/json` | `200` + representation，或 `204` |
| `PATCH /subscriptions/{subscriptionId}` | `application/merge-patch+json` | `200` + representation，或 `204` |
| `DELETE /subscriptions/{subscriptionId}` | none | `204` |
| `POST {notifUri}` callback | `application/json` | `204` |

POST declared errors：

```text
400 401 403 404 411 413 415 429 500 502 503 default
```

PUT／PATCH declared errors：

```text
307 308 400 401 403 404 411 413 415 429 500 502 503 default
```

DELETE declared errors：

```text
307 308 400 401 403 404 429 500 502 503 default
```

Callback declared errors：

```text
307 308 400 401 403 404 411 413 415 429 500 502 503 default
```

這份 matrix 在 Phase 0 作為 contract evidence；Phase 3 才建立 handler
與 response writer。

### 7.2 Subscription model

`NwdafMLModelTrainSubsc` OpenAPI required fields：

- `mLEventSubscs`，至少一個；
- `notifUri`；
- `notifCorreId`。

optional fields：

- `suppFeats`；
- `eventReq`；
- `failEventReports`；
- `mlCorreId`；
- `mLModelInfos`；
- `immReport`；
- `mLModelTrainInfos`；
- `mLPreFlag`；
- `mLAccChkFlg`；
- `mLTrainRepInfo`；
- `roundInd`；
- `tgtRepUe`；
- `skipFlInd`。

PATCH 只允許：

- `notifUri`；
- `eventReq`；
- `mLModelInfos`；
- `mLModelTrainInfos`；
- `mLPreFlag`；
- `mLAccChkFlg`；
- `mLTrainRepInfo`；
- `roundInd`；
- `tgtRepUe`；
- `skipFlInd`。

PATCH model 不得自行加入 `mlCorreId`。

### 7.3 Notification model

`NwdafMLModelTrainNotif` 的 OpenAPI required field 只有
`notifCorreId`，但 TS conditional rule 另要求至少存在：

- `delayEventNotif`；
- `mLModelInfos`；
- `termTrainReq`；

三者之一。

合法組合：

| delay | model info | terminate | Valid |
| --- | --- | --- | --- |
| yes | no | no | yes |
| no | yes | no | yes |
| no | no | yes | yes |
| no | yes | yes | yes |
| yes | yes/no | yes/no | no |
| no | no | no | no |

`statusReport` 可以伴隨可接受的結果，但不能獨自滿足上述
at-least-one rule。

### 7.4 FL-specific conditional validation

wire shape validation 與 FL procedure validation 分開：

```text
Parse / model_validate
  -> OpenAPI shape

ValidateFLRequest(existing state)
  -> TS conditional requirements
  -> project supported profile
```

FL procedure validation：

1. FL request 必須有非空 `mlCorreId`；
2. 每個 `mLEventSubscs` element 必須有非空 `modelInterInfo`；
3. `mLPreFlag=true` 時，每個 `MLModelTrainInfo` 同時有：
   - `dataAvReq`；
   - `timeAvReq`；
4. `dataAvReq.inpEvents` 至少一個；
5. `mLTrainRepInfo` 只有在
   `eventReq.notifMethod=ON_EVENT_DETECTION` 時可使用；
6. `mLTrainRepInfo.maxResTime` 若存在必須符合 `DurationSec`；
7. `notifCorreId` 必須與 resource 內接受的值相同；
8. FL notification 必須帶相同 `mlCorreId`；
9. PUT body 的 `mlCorreId` 必須與 existing resource 相同；
10. PATCH 不含 `mlCorreId`，由 path 對應 existing resource 取得；
11. `roundInd` 若存在必須為非負整數；
12. `modelUniqueId` 保留規格的 optional semantics；
13. enum 允許 future string extension，不以 closed enum 拒絕未來合法值。

Phase 0 pure validator 接受一個最小 existing-resource view：

```text
TrainingResourceIdentity
  subscriptionId
  mlCorreId
  notifCorreId
  expectedRoundInd (optional)
```

它不是 resource repository；Phase 3 的 state owner 再提供這些值。

### 7.5 Final validation profile

final training round aggregation 完成後，Server 不直接 promotion。第一版
使用一個獨立 validation-only round：

```text
C -> A/B Training resource update
     mLAccChkFlg=true
     skipFlInd=true
     roundInd=final training round + 1
     mLModelInfos=final candidate

A/B -> download candidate
     -> use frozen local validation data
     -> do not perform local fitting
     -> publish ROUND_LOCAL/result_type=ACCURACY_CHECK

A/B -> C notifUri
     NwdafMLModelTrainNotif
     mLModelInfos=Client-hosted accuracy-check bundle
     statusReport.mlModelAcc (when true ACCURACY is available)

C -> verify candidate digest and WAPE evidence
  -> one global promotion gate
  -> FINAL_MODEL
```

約束：

- `mLAccChkFlg=true` 表示要求 Client 驗證 global model，本身不表示停止
  training；
- `skipFlInd=true` 才表示本 round 不執行 local fitting；
- `statusReport` 不能獨自滿足 Training notification 的 at-least-one
  condition，因此正常成功 callback 同時提供 `mLModelInfos`；
- `mLModelInfos.mLFileAddr` 指向合法 model bundle；bundle 包含未修改的
  candidate model，並由 project model interoperability contract 加入
  WAPE components 等 private metadata；
- accuracy-check bundle 的 output weights digest 必須等於 request 中
  candidate digest；
- `statusReport.mlModelAcc` 只用於真正的 0–100 accuracy，不把 WAPE
  重新命名後塞入該欄位；
- WAPE exact sample count、error sum、actual-value sum 與 time window
  只存在 project bundle metadata，不新增標準 notification field。

規格依據：

- TS 23.288 §6.2F 將 Model Accuracy Check Flag 與 skip current FL round
  indication列為獨立 optional inputs；
- TS 29.520 §5.5.6 與
  `TS29520_Nnwdaf_MLModelTraining.yaml` 定義 `mLAccChkFlg`、
  `skipFlInd`、notification `statusReport` 與 `mLModelInfos` conditional
  rule。

### 7.6 Service-specific errors

Phase 0 定義 typed cause constants 與 validation-to-cause mapping：

| HTTP | Cause |
| --- | --- |
| `403` | `ML_MODEL_TRAINING_REQS_NOT_MET` |
| `403` | `ML_TRAINING_NOT_COMPLETE` |
| `403` | `OVERLOAD` |
| `403` | `NOT_AVAILABLE_FOR_FL_PROCESS_ANYMORE` |
| `500` | `UNAVAILABLE_ML_MODEL_TRAINING_FOR_ALLEVENTS` |

`ML_MODEL_TRAINING_REQS_NOT_MET` 的 `ProblemDetails.invalidParams` 應列出
無法滿足的 attribute。Phase 0 只建立可產生 invalid parameter paths 的
validation result；HTTP `ProblemDetails` writer 在 Phase 3 接入。

---

## 8. TrainingScopeDescriptor

### 8.1 目的

`TrainingScopeDescriptor` 是 PyMTLF private canonical representation，
用來：

- 固定 preparation 時接受的 training scope；
- 計算 stable `scopeDigest`；
- 對照 Client 的 ADRF descriptor inventory；
- 讓 round、evaluation 與 final manifest 指向同一 scope。

它不是新的標準 API body。

### 8.2 保留的標準資訊

每個 `mLEventSubscs[]` 轉換後保留：

- `mLEvent`；
- 完整 `mLEventFilter` object；
- element-level `tgtUe`；
- `mLTargetPeriod`；
- `modelInterInfo`；
- `nfConsumerInfo`；
- `modelProvExt`；
- `useCaseCxt`；
- `inferDataForModel`；
- `modelId`。

subscription-level 另保留：

- `tgtRepUe`；
- preparation `dataAvReq.timeWindows`；
- `mlCorreId` 不進入 scope digest，只作 process identity。

Phase 0 不自行推導 element-level `tgtUe` 與 `tgtRepUe` 的覆蓋關係；
兩者都保存。Phase 3 依 procedure context 決定 effective target。

### 8.3 Canonicalization

```text
1. 使用 wire aliases 輸出標準 JSON property names
2. 排除值為 null 的 optional fields
3. object keys lexicographically sorted
4. arrays 保持原始順序
5. UTF-8、無額外 whitespace
6. SHA-256 lowercase hex -> scopeDigest
```

不得：

- 排序 `supis`、TAI 或其他規格上可能具有順序語意的 arrays；
- 把 `mlCorreId`、`notifCorreId`、`roundInd` 納入 scope digest；
- 只取 TAI 而丟棄 DNN、S-NSSAI、Application ID 或 target；
- 以 SMF subscription ID 取代 training scope。

---

## 9. NRF Release 18 FL capability contract

### 9.1 MlAnalyticsInfo

compatibility type 必須表達：

| Field | Contract |
| --- | --- |
| `mlAnalyticsIds` | non-empty when present |
| `snssaiList` | non-empty when present |
| `trackingAreaList` | non-empty when present |
| `mlModelInterInfo.vendorList` | each Vendor ID is exactly six digits |
| `flCapabilityType` | extensible string enum |
| `flTimeInterval` | `TimeWindow` |
| `nfTypeList` | non-empty when present |
| `nfSetIdList` | non-empty when present |

第一版認得：

- `FL_SERVER`；
- `FL_CLIENT`；
- `FL_SERVER_AND_CLIENT`。

parser 必須保存 future extension string；Phase 1 的 role-matching policy
只把已知三值視為可執行能力。

### 9.2 NwdafInfo／NFProfile wrapper

Phase 0 測試以下 round trips：

1. 只有現有 analytics provider fields；
2. 只有 model provider `mlAnalyticsList`；
3. FL Server capability；
4. FL Client capability + TAI；
5. 同一 instance 同時具有 Server 與 Client capability；
6. unknown `flCapabilityType` 能保存，但不被 supported-profile validator
   接受為已知 runtime role。

Phase 0 不送 NRF request，也不決定 NRF cache key。這些屬於 Phase 1。

---

## 10. FL artifact manifest contract

### 10.1 不改變 completed bundle 的安全邊界

現有 completed bundle 外層格式保持：

```text
bundle.tar.gz
├── config.json
├── model.py
├── model.npy
└── scaler.pkl
```

現有 archive size、entry count、safe path、exact file set 與 digest
檢查不得放寬。

本 Phase 新增獨立的 role-aware Training Workspace validator。不得為了讓
暫存模型通過而把 current completed `ArtifactRepository` 改成接受缺少
正式 identity 的任意 bundle。

### 10.2 Artifact roles

| Role | Meaning | Formal `modelUniqueId` | Durable completed revision |
| --- | --- | --- | --- |
| `ROUND_LOCAL` | Client training 或 accuracy-check result | no | no |
| `ROUND_GLOBAL` | Server aggregated round result | no | no |
| `FINAL_MODEL` | 通過 global gate、待／已 publication 的正式模型 | yes | ADRF store 成功後 |

`FINAL_MODEL` 在送 ADRF 前先配置一個不再重用的正式 ID，並把它寫入
immutable bundle；只有 ADRF store 成功及 catalog commit 後，才成為
completed／latest。

### 10.3 Common FL metadata

`config.json` 延續 `bundle_schema_version`，另加入：

```json
{
  "artifact_role": "ROUND_LOCAL",
  "result_type": "TRAINING",
  "fl_metadata": {
    "contract_version": "1.0",
    "ml_corre_id": "fl-process-001",
    "model_contract_digest": "<sha256>",
    "preprocessing_contract_digest": "<sha256>",
    "base_weights_digest": "<sha256>",
    "weights_digest": "<sha256>"
  }
}
```

`result_type` 只適用於 `ROUND_LOCAL`，並與 `artifact_role` 同層。final
validation response 範例：

```json
{
  "artifact_role": "ROUND_LOCAL",
  "result_type": "ACCURACY_CHECK",
  "fl_metadata": {
    "contract_version": "1.0",
    "ml_corre_id": "fl-process-001",
    "round_ind": 3,
    "participant_nf_instance_id": "<nf-instance-id>",
    "scope_digest": "<scope-sha256>",
    "model_contract_digest": "<sha256>",
    "preprocessing_contract_digest": "<sha256>",
    "base_weights_digest": "<candidate-sha256>",
    "input_global_weights_digest": "<candidate-sha256>",
    "weights_digest": "<same-candidate-sha256>",
    "evaluation": {
      "evaluation_stage": "FINAL_VALIDATION",
      "evaluation_sample_count": 400,
      "start_time": "2026-07-27T00:00:00Z",
      "end_time": "2026-07-28T00:00:00Z",
      "base_model_weights_digest": "<base-model-sha256>",
      "candidate_weights_digest": "<candidate-sha256>",
      "base": {
        "absolute_error_sum": 100.0,
        "absolute_actual_sum": 1000.0
      },
      "candidate": {
        "absolute_error_sum": 50.0,
        "absolute_actual_sum": 1000.0
      }
    }
  }
}
```

所有 role 必須有：

- `contract_version`；
- `ml_corre_id`；
- `model_contract_digest`；
- `preprocessing_contract_digest`；
- `base_weights_digest`；
- current bundle 的 `weights_digest`；
- outer existing `file_digests`。

`model_contract_digest` 涵蓋：

- model architecture identity；
- parameter／buffer name；
- tensor order；
- shape；
- dtype。

`preprocessing_contract_digest` 涵蓋：

- feature order；
- scaler artifact digest；
- sequence length；
- preprocessing version。

### 10.4 Role-specific metadata

#### ROUND_LOCAL

- `result_type`：
  - `TRAINING`；
  - `ACCURACY_CHECK`；
- `round_ind`；
- `participant_nf_instance_id`；
- `scope_digest`；
- `input_global_weights_digest`；
- output `weights_digest`。

`TRAINING`：

- 必須有 `training_sample_count > 0`；
- output weights 可與 input global weights 不同；
- 若同一輪也設定 `mLAccChkFlg=true`，可選擇附帶對 input global model
  的 evaluation metadata。

`ACCURACY_CHECK`：

- 對應 Server request 的 `mLAccChkFlg=true + skipFlInd=true`；
- 不執行 local fitting；
- 不宣告 `training_sample_count`；
- 必須有 evaluation metadata；
- output `weights_digest` 必須等於 `input_global_weights_digest`，證明
  Client 沒有修改被驗證的 candidate；
- bundle 仍包含合法模型內容，evaluation metadata 是 bundle 中的
  project-private metadata。

evaluation metadata：

- evaluation stage；
- evaluation sample count；
- evaluation time window；
- base model digest 與：
  - `absolute_error_sum`；
  - `absolute_actual_sum`；
- candidate model digest 與：
  - `absolute_error_sum`；
  - `absolute_actual_sum`。

#### ROUND_GLOBAL

- `round_ind`；
- canonical ordered participants；
- 每個 participant：
  - NF instance ID；
  - input local artifact digest；
  - training sample count；
- `aggregated_training_sample_count`；
- output `weights_digest`。

participant ordering 固定以 normalized NF instance ID 排序，避免相同輸入
產生不同 manifest。

WAPE 不以各 Client 的百分比再平均。C 從 `ROUND_LOCAL` accuracy-check
metadata 取得 sums 後重建：

```text
WAPE = sum(absolute_error_sum) / sum(absolute_actual_sum)
```

#### FINAL_MODEL

- `model_identity.model_unique_id`；
- `previous_model_unique_id`；
- `ml_corre_id`；
- participants；
- 各 Client training sample count；
- final candidate digest；
- per-Client validation summary：
  - participant NF instance ID；
  - scope digest；
  - evaluation sample count 與 time window；
  - base／candidate weights digests；
  - base／candidate WAPE component sums；
- global gate result；
- creation timestamp。

per-Client summary 只保存 evidence，不包含 `accepted`；是否發布只有
top-level `global_gate_accepted` 一次決策。

`storeTransId` 不放入 bundle，因為它在 ADRF 接受 store request 後才存在。

### 10.5 Tensor compatibility

- FedAvg 只聚合 floating tensors；
- non-floating state 必須與 base model 完全一致；
- scaler 與 preprocessing contract 在整個 process 不變；
- model contract 或 preprocessing digest 不同即拒絕；
- `ROUND_LOCAL/TRAINING.training_sample_count` 必須大於零；
- `ROUND_LOCAL/ACCURACY_CHECK` 必須維持 candidate weights digest；
- temp roles 不得包含正式 `model_identity`；
- `FINAL_MODEL` 必須包含正式 numeric model ID。

Phase 0 只實作 model／validator／fixture；builder、download、FedAvg 與 TTL
cleanup 在 Phase 4／5 接入。

---

## 11. Catalog and publication journal contract

### 11.1 Completed catalog

第一版 catalog 只表達線性 revisions：

```text
M1 -> M2 -> M3
            ^
            latestModelId
```

schema：

```text
Catalog
  schemaVersion
  latestModelId
  nextModelId
  revisions[]

CompletedRevision
  modelUniqueId
  previousModelUniqueId
  origin (SEED | FEDERATED)
  artifactKey
  artifactDigest
  createdAt
  mlCorreId (optional for seed)
  participants[]
  validationSummary
  adrfReference (required for FEDERATED)

AdrfReference
  adrfInstanceId
  storeTransId
  resourceLocation
```

invariants：

- revision IDs unique；
- `latestModelId` 必須存在於 revisions；
- `previousModelUniqueId` 必須存在或為 null；
- 不能形成 cycle；
- `nextModelId` 大於所有已完成或已 reserve 的 ID；
- `FEDERATED` revision 必須有 ADRF reference；
- seed 可只有 current local content-addressed artifact；
- `storeTransId` 是 ADRF record locator，不是 model identity。

### 11.2 Pending publication journal

schema：

```text
PendingPublication
  schemaVersion
  publicationId
  state
  mlCorreId
  reservedModelId
  previousModelId
  participantsAndSampleCounts
  validationSummary
  candidatePath
  candidateDigest
  finalBundlePath
  finalBundleDigest
  selectedAdrfTarget
  storeTransId
  resourceLocation
  lastError
  updatedAt
```

states：

- `RESERVED`；
- `FINAL_BUNDLE_READY`；
- `STORE_IN_FLIGHT`；
- `STORE_ACCEPTED`；
- `CATALOG_COMMITTED`；
- `FAILED_TERMINAL`。

invariants：

- reserve model ID 時 `nextModelId` 已前進；
- publication 失敗也不回收 ID；
- terminal 前 candidate／final bundle 必須由 durable publication directory
  pin 住；
- `STORE_ACCEPTED` 必須有 `storeTransId` 與 `resourceLocation`；
- catalog latest 只在 `STORE_ACCEPTED` 後更新；
- `CATALOG_COMMITTED` 的 model ID 必須是 catalog latest 或已知 completed
  revision；
- corrupt file fail-fast，不得默默回 seed；
- unknown schema version fail-fast。

### 11.3 Migration semantics

Phase 0 freeze 以下啟動語意，Phase 5 才接入 application lifespan：

1. catalog 不存在：
   - 驗證 configured seed artifact；
   - 建立第一個 `SEED` revision；
   - `latestModelId=seed ID`；
   - `nextModelId=max(seed IDs)+1`。
2. catalog 存在且合法：
   - catalog 是 source of truth；
   - 不重新從 seed 覆蓋。
3. catalog 存在但損壞：
   - readiness fail；
   - 不自動 reset。
4. journal 存在：
   - 依 state reconcile；
   - ADRF 已成功但 catalog 未 commit 時，不重新配 ID。
5. 實驗重跑：
   - 使用明確 reset procedure；
   - 不以 startup fallback 隱式清除。

Phase 0 實作 Pydantic schema、pure invariant validator 與 current/migration
fixtures；atomic filesystem repository、fsync、recovery worker 與 ADRF probe
在 Phase 5 實作。

### 11.4 不取代 current local path

Phase 0 不把 current `ModelCatalog` 或 local retraining coordinator 切換到
新 catalog。這避免 contract work 影響已可運作的 local training。

Phase 5 導入 persistent catalog 時，才一起移除舊 in-memory promotion
ownership，避免兩個 catalog 同時寫入。

---

## 12. Shared contract examples

在 `nwdaf-resources/contracts/federated_learning/v1/` 建立：

```text
contracts/federated_learning/v1/
├── README.md
├── reporting-information-full.json
├── nrf-fl-client-profile.json
├── nrf-fl-server-profile.json
├── training-preparation-request.json
├── training-preparation-response.json
├── training-round-update.json
├── training-round-result.json
├── training-delay-notification.json
├── training-termination-notification.json
├── training-partial-update.json
├── training-final-validation-result.json
├── artifact-round-local.json
├── artifact-round-local-accuracy-check.json
├── artifact-round-global.json
├── artifact-final-model.json
├── catalog-current.json
├── catalog-seed-migration.json
└── publication-pending.json
```

README 對每個 case 記錄：

- normative or project-private；
- producer；
- consumer；
- expected validation result；
- 對應規格；
- 哪些欄位刻意省略；
- 後續哪個 Phase 會使用。

這些 JSON 是跨 repository 的 review／E2E input，不成為各 implementation
repo unit test 的 runtime dependency。

各 repository 的 unit tests 以相同 case ID 建立 code-native fixtures：

```text
RPT_FULL_R18
NRF_FL_CLIENT
TRAIN_PREPARE_VALID
TRAIN_ROUND_VALID
TRAIN_NOTIFY_DELAY_VALID
TRAIN_NOTIFY_DELAY_CONFLICT
ARTIFACT_ROUND_LOCAL_VALID
ARTIFACT_ROUND_LOCAL_ACCURACY_CHECK_VALID
CATALOG_CURRENT_VALID
```

如此可維持 repository 獨立執行，同時讓 review 能對照相同案例。

---

## 13. Repository-specific implementation plan

### 13.1 NWDAF

預計新增：

```text
internal/compat/mlmodel/reporting.go
internal/compat/mlmodel/reporting_test.go
internal/compat/mlmodeltraining/models.go
internal/compat/mlmodeltraining/validation.go
internal/compat/mlmodeltraining/models_test.go
internal/compat/mlmodeltraining/validation_test.go
internal/compat/nrf/models.go
internal/compat/nrf/models_test.go
```

預計調整：

```text
internal/compat/mlmodel/models.go
internal/compat/mlmodel/models_test.go
```

工作：

1. 把 Provision／Monitor 的 reporting raw field 改為 shared typed
   `ReportingInformation`；
2. 保留 public processor 的 original raw body forwarding；
3. 建立 Training types 與 pure validators；
4. 建立 NRF wrapper；
5. 補 required／optional／cardinality／future enum tests；
6. 補 marshal/unmarshal presence tests；
7. 不修改 `internal/sbi/server.go`、route registration 或 config prefix；
8. 不新增 backend Training methods。

### 13.2 PyMTLF

預計新增：

```text
src/py_mtlf/wire/reporting.py
src/py_mtlf/wire/ml_model_training.py
src/py_mtlf/core/training_scope.py
src/py_mtlf/core/fl_artifacts.py
src/py_mtlf/core/model_records.py
tests/test_ml_model_training_wire.py
tests/test_training_scope.py
tests/test_fl_artifacts.py
tests/test_model_records.py
```

預計調整：

```text
src/py_mtlf/wire/ml_model.py
src/py_mtlf/wire/ml_model_monitor.py
```

工作：

1. 抽出 shared `ReportingInformation`；
2. 建立完整 Training Pydantic models；
3. 建立 FL conditional validation；
4. 建立 canonical scope descriptor／digest；
5. 建立 role-aware manifest models；
6. 建立 catalog／journal schema與 pure invariants；
7. 保留 current `ArtifactRepository`、`CandidateBundleBuilder`、
   `ModelCatalog` 與 application lifespan 行為；
8. 不新增 FastAPI Training router；
9. 不新增 FL coordinator 或 background worker。

### 13.3 PyAnLF

預計調整：

```text
src/py_anlf/models.py
tests/test_model_demand.py
tests/test_monitor_subscription.py
tests/test_events_subscription_api.py
```

工作：

1. 將共用 `ReportingInformation` 補為 exact R18 fields；
2. Model Provision 與 Monitor 使用同一 reporting type；
3. optional scalar 使用 presence-preserving representation；
4. 將非標準 `modelProviderId` 移出 standard wire model，改由既有 private
   provision context保存；
5. 保留現有 demand matching、activation、monitor reporting 與 WAPE
   behavior；
6. 不新增 Training contract 或 FL state。

### 13.4 nwdaf-resources

新增第 12 節的 cross-language examples 與 README。

不新增：

- NF-root `testdata`；
- process launcher；
- network namespace；
- multi-NWDAF deployment；
- E2E success claim。

### 13.5 nwdaf-docs

實作完成後更新：

- 上層計畫 Phase 0 status；
- compatibility ownership inventory；
- 規格解讀中的實際 contract examples，如實作發現原例與 YAML 不一致；
- verification evidence；
- deferred items。

---

## 14. Detailed implementation order

### Step 1：Go shared and Training contract

1. 建立 shared `ReportingInformation`；
2. 調整既有 Provision／Monitor parser；
3. 建立 Training models；
4. 建立 OpenAPI shape validation；
5. 建立 FL conditional validation；
6. 跑 focused Go tests。

完成條件：

- existing Provision／Monitor behavior tests 不退步；
- full R18 reporting fixture 可 round-trip；
- valid／invalid Training cases有明確結果。

### Step 2：Go NRF compatibility contract

1. 建立 generated-wrapper types；
2. 建立 known／unknown `flCapabilityType`；
3. 驗證 base and R18 fields 共存；
4. 驗證 Vendor ID、non-empty arrays、TimeWindow；
5. 不連 NRF。

### Step 3：PyMTLF wire and scope contract

1. 抽出 reporting type；
2. 建立 Training models；
3. 建立 procedure validation；
4. 建立 scope canonicalization；
5. 對照 Go case IDs。

### Step 4：PyAnLF shared reporting alignment

1. 移除重複的部分 reporting DTO；
2. 更新呼叫端；
3. 確認 existing Events／Provision／Monitor tests；
4. 確認 WAPE 與 model demand behavior不變。

### Step 5：PyMTLF artifact and durable-state schemas

1. 建立三種 artifact roles，並以 `ROUND_LOCAL.result_type` 區分 training
   與 accuracy-check result；
2. 建立 role-specific invariant validator；
3. 建立 catalog／journal models；
4. 建立 seed migration pure function／fixture；
5. 不接 application lifespan。

### Step 6：Shared examples and documentation

1. 建立 `nwdaf-resources/contracts/`；
2. 對照 Go/Python code-native cases；
3. 更新 spec evidence links；
4. 檢查無 standard/private field 混用。

### Step 7：一次完整 review

依 development policy，在所有 slices 完成後做一次完整 review：

- contract fidelity；
- existing behavior regression；
- dead／speculative runtime code；
- repository boundary；
- test claim；
- docs consistency。

review 發現只做 targeted remediation；不重新進行無範圍的整體掃描。

---

## 15. Test plan

### 15.1 NWDAF focused tests

- ReportingInformation absent／false／zero presence；
- complete R18 reporting round-trip；
- Training required fields；
- array `minItems`；
- `mLPreFlag` conditional requirements；
- notification at-least-one and mutual exclusion；
- PUT process identity；
- PATCH omitted-field presence；
- model ID／round non-negative；
- future enum preservation；
- NRF R17 + R18 field round-trip；
- duplicate JSON key rejection／prevention；
- existing Provision／Monitor parser regression。

### 15.2 PyMTLF focused tests

- same valid/invalid wire case IDs as Go；
- aliases and `exclude_none` serialization；
- unknown standard extension preservation；
- timezone-aware date/time；
- TrainingScopeDescriptor deterministic digest；
- object key order does not change digest；
- array order does change digest；
- process／notification IDs do not change scope digest；
- temporary round artifact rejects formal model ID；
- final model artifact requires formal model ID；
- digest mismatch rejection；
- non-floating state mismatch rejection；
- WAPE component validation；
- catalog invariants；
- journal state invariants；
- seed migration round-trip；
- unknown private schema version fail-fast。

### 15.3 PyAnLF regression tests

- full ReportingInformation parse／dump；
- Model Provision body；
- Model Monitor body；
- Events Subscription reporting；
- model demand selection unchanged；
- monitor schedule behavior unchanged；
- WAPE reporting unchanged。

### 15.4 Repository verification

實作完成後預計執行：

#### NWDAF

```text
go test ./internal/compat/...
go test -race ./internal/compat/...
make test
make build
make lint
```

#### PyMTLF

```text
uv run pytest -q
uv run ruff check .
```

#### PyAnLF

```text
uv run pytest -q
uv run ruff check .
```

#### nwdaf-resources

- 所有 JSON 可被標準 JSON parser 讀取；
- README case inventory 與實際檔案一致；
- 不宣稱 process-level E2E。

所有實際執行結果須在完成後記錄，不預先填寫 passed。

---

## 16. Acceptance criteria

Phase 0 只有在以下全部成立時完成：

1. Go 與 PyMTLF 可表達完整 Release 18 Training supported profile；
2. Go、PyAnLF、PyMTLF 共用相同 ReportingInformation field semantics；
3. absence、`false`、`0` 不會混為一談；
4. Training conditional requirements 有 pure tests；
5. notification 使用 at-least-one，沒有誤寫 exactly-one；
6. PATCH 不出現非規格 `mlCorreId`；
7. NRF R18 FL fields 不會在 JSON round-trip 消失；
8. generated model 與 compatibility model ownership不重疊；
9. `NWDAF/go.mod` 無 OpenAPI fork／local `replace`；
10. supported-profile examples 在各語言的 code-native tests 語意一致；
11. `ROUND_LOCAL`／`ROUND_GLOBAL` 無正式 model ID；
12. `FINAL_MODEL` 有正式 numeric model ID；
13. `storeTransId` 不寫回 immutable final bundle；
14. current completed bundle validator 沒有被放寬；
15. current local training、Provision、Monitor、AnLF WAPE behavior不變；
16. catalog／journal current and migration fixtures 可 round-trip；
17. corrupt private durable schema fail-fast；
18. 沒有新增無 runtime owner 的 public route、worker 或 config；
19. 各 repository verification 完成；
20. 文件只宣稱實際驗證過的結果。

---

## 17. Deferred work

| Deferred item | Target |
| --- | --- |
| explicit A/B/C config roles | Phase 1 |
| NRF registration／discovery／matching | Phase 1 |
| remote Model Provision／Monitor | Phase 2 |
| Training routes／resource mirror／callback | Phase 3 |
| preparation worker／ADRF dataset snapshot | Phase 3 |
| local rounds／temporary artifact serving | Phase 4 |
| weighted FedAvg | Phase 4 |
| final Client validation | Phase 5 |
| persistent filesystem catalog integration | Phase 5 |
| ADRF final store／reconcile／promotion | Phase 5 |
| group→SUPI→serving SMF／AoI collection | Phase 6 |
| multi-NWDAF E2E | Phase 7 |

---

## 18. Blockers and decisions

Phase 0 沒有新的 product decision blocker。

已足以直接採用的決策：

- compatibility-first，不建立 OpenAPI fork；
- standard wire 與 project-private state 分離；
- long-lived Training resource；
- fixed participants、synchronous full-model FedAvg；
- exact local sample count只放 project artifact；
- temporary artifact沒有正式 model ID；
- final model先配置 ID、ADRF 成功後才 catalog promote；
- single latest completed revision；
- Phase 0 不接 runtime。

下列項目不阻擋 Phase 0：

- production model ID 起始區間；
- 實際 data directory；
- external NRF／ADRF fork merge timing；
- Phase 3 resource timeout；
- Phase 4 round count；
- Phase 5 publication retry parameters。

這些值只影響後續 runtime config，不影響本 Phase contract schema。

---

## 19. 實作結果

### 19.1 Compatibility ownership

實際 ownership 與本計畫一致：

- `NWDAF/internal/compat/mlmodel/`
  - shared Release 18 `ReportingInformation`；
  - Provision／Monitor 共用 reporting contract；
  - existing completed model bundle validator 未放寬。
- `NWDAF/internal/compat/mlmodeltraining/`
  - Training subscription、PATCH、notification 與 nested supported-profile
    types；
  - OpenAPI shape validation 與 FL procedure conditional validation；
  - 只提供 pure types／validators，未註冊 public route。
- `NWDAF/internal/compat/nrf/`
  - generated Release 17 profile wrapper；
  - Release 18 `MlAnalyticsInfo` FL fields；
  - model-only、analytics-provider、FL Client、FL Server、combined role 與
    future capability round-trip。
- `PyMTLF/src/py_mtlf/wire/`
  - shared reporting 與完整 Training wire models。
- `PyMTLF/src/py_mtlf/core/`
  - canonical training scope；
  - role-aware artifact contract 與 tensor compatibility validator；
  - completed catalog、pending publication journal、seed migration 與
    cross-record invariants。
- `PyAnLF`
  - Provision／Monitor／Events 使用同一 Release 18 reporting field
    semantics；
  - containing provider identity 改由 private provision context 保存；
  - standard `MLEventNotif` 不再宣告非規格 `modelProviderId`。
- `nwdaf-resources/contracts/federated_learning/v1/`
  - 18 個 standard／project-private JSON examples 與 case inventory。

### 19.2 Preserved runtime boundary

本次沒有：

- 新增 `Nnwdaf_MLModelTraining` route、handler、consumer 或 worker；
- 啟動 FL preparation、round、FedAvg、ADRF publication 或 recovery；
- 將 current in-memory `ModelCatalog` 切換為 persistent catalog；
- 放寬 existing completed bundle 的 archive、file set 或 digest 驗證；
- 修改 `NWDAF/go.mod` 或建立 OpenAPI fork／local `replace`；
- 宣稱真實 NRF、ADRF、SMF、UPF 或三 NWDAF process-level E2E。

### 19.3 Verification evidence

2026-07-28 實際執行：

#### NWDAF

- `go test ./internal/compat/...`：passed；
- `go test -race ./internal/compat/...`：passed；
- `make test`：passed；
- `make build`：passed；
- `make lint`：passed。

#### PyMTLF

- `uv run pytest -q`：128 passed；
- `uv run ruff check .`：passed。

測試涵蓋 Training shape／conditional rules、final validation flags 與
callback、scope digest、三種 artifact roles、`ROUND_LOCAL` result types、
unchanged-candidate digest、完整 validation evidence、tensor
compatibility、WAPE components、linear catalog、journal states、seed
migration、reserved ID 與 catalog cross-record invariants。

#### PyAnLF

- `uv run pytest -q`：239 passed、1 skipped；
- `uv run ruff check .`：passed。

skip 是既有 test suite 的環境性 skip；本次 contract、demand、monitor、
runtime 與 Events regression tests 均通過。

#### nwdaf-resources

- 18 個 JSON examples 全部通過 `jq` JSON parse；
- 15 個 Training／artifact／catalog／journal examples 另由 PyMTLF 的實際
  Pydantic contract parse；
- README inventory 與實際 examples 一致。

PyMTLF full suite 的 10 個 warnings 來自既有 Starlette/httpx 與
joblib/NumPy 相依套件 deprecation，未視為本次 contract failure。

### 19.4 Final validation design correction

後續討論將 initial artifact contract 修正為：

- 移除 `CLIENT_EVALUATION` role；
- `ROUND_LOCAL.result_type` 使用 `TRAINING` 或 `ACCURACY_CHECK`；
- final validation request 使用
  `mLAccChkFlg=true + skipFlInd=true`；
- accuracy-check bundle 保存未修改的 candidate model 及 evaluation
  metadata；
- `PROCESS_FINAL` 改名為 `FINAL_MODEL`。

correction 已同步至 PyMTLF schema／tests、Go Training wire validation、
`nwdaf-resources` examples、主計畫與 architecture design。正式模型的
per-Client validation summary 只保存 scope、data window、model digests
與 WAPE components；不再保存 per-Client `accepted`，promotion 只由
top-level global gate 決定。

重新驗證結果已併入 §19.3，因此 Phase 0 contract foundation 現在完成。
