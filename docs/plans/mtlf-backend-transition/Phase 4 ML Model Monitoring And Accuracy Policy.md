# Phase 4 Initial Model Provision, ML Model Monitoring And Accuracy Policy

Date: 2026-07-23

Status: Initial model provision and monitoring slice implemented, verified, and committed

Parent plan:

- `MTLF Backend Transition Plan.md`

Previous phase:

- `Phase 3 Analytics Subscription Routing.md`

Current review ledger:

- `code-reviews/Initial Model Provision And Monitoring Review Ledger.md`

Historical behavior references:

- `../anlf-backend-transition/Phase 4 Accuracy Workflow Migration.md`
- `../anlf-backend-transition/R2 Accuracy Measurement Parity.md`
- `../anlf-backend-transition/R3 Model Identity Generation And Provision Correctness.md`
- `../daisy/general_improvement/nwdaf-daisy-improvement-plan.md`

---

## 1. Purpose

本phase先完成initial trained model provision，再建立完整的`Nnwdaf_MLModelMonitor` resource與notification
routing，並把目前仍在Go內且經本計畫保留的MTLF degradation/retrain decision mechanics移植到PyMTLF。這個範圍
對應TS 23.288 clause 6.2E.2 steps 1至7：analytics subscription觸發AnLF取得model，model成功載入後AnLF才
register其使用與monitoring capability，MTLF再subscribe accuracy notification。

完成後的責任邊界為：

1. Go NWDAF是唯一對外提供`nnwdaf-mlmodelprovision`與`nnwdaf-mlmodelmonitor`服務的標準NF identity。
2. ML Model Provision subscription resource、initial seed model catalog與artifact URL由MTLF backend擁有；
   AnLF backend擁有model demand、download/load與runtime binding。
3. ML Model Monitor `registrations` resource由MTLF backend擁有；`subscriptions` resource與accuracy measurement由
   AnLF backend擁有。
4. Go負責HTTP/OpenAPI validation、standard status/Location、backend routing、callback delivery、
   最小process-local resource mirror及backend reconnect sync。
5. PyAnLF保留既有model artifact validation/cache/load/atomic activation，以及已驗證的prediction、
   ground-truth matching、confidence readiness與scope isolation；accuracy wire只回報WAPE。
6. PyMTLF保留既有immutable artifact repository，並承接目前Go MTLF中仍適用的baseline、degradation reference、
   z-score、N-in-M、scope isolation、retrain-in-flight與lifecycle mechanics。
7. PyAnLF與PyMTLF不直接呼叫彼此；同一NWDAF內需要交互的provision/monitor operation仍透過Go，body使用
   Release 18 standard shape。
8. 現有custom model provision coordination與`POST /model-accuracy-reports`退出production path；既有正確的
   bundle、download、load與runtime swap行為保留在Python owner內。

本phase不訓練seed model，也不完成retrain後的新generation publication。PyMTLF由config載入已存在且符合既有
bundle contract的seed artifact，提供initial provision。Policy判定之後所需dataset retrieval由Phase 5完成；
local training、new artifact publication及updated/re-trained model reprovision由Phase 6完成。

---

## 2. Active Implementation Slice

### 2.1 Vertical Flow

本phase涵蓋以下vertical flow：

```text
Consumer creates an analytics subscription
    -> PyAnLF accepts it and resolves a compatible loaded model
    -> if absent, PyAnLF requests standard-shaped Model Provision through Go
    -> PyMTLF returns or notifies a configured seed model URL
    -> PyAnLF downloads, validates, loads and binds the model
    -> PyAnLF sends standard-shaped Monitor Register through Go
    -> PyMTLF owns registration
    -> PyMTLF local policy requests standard-shaped Subscribe through Go
    -> PyAnLF owns monitoring subscription
    -> PyAnLF measures prediction against ground truth
    -> PyAnLF sends MLModelMonitorNotify to Go
    -> Go routes the standard notification to PyMTLF
    -> PyMTLF updates policy state and may create a retrain intent
```

同時也包含external consumer透過Go建立／更新／刪除Model Provision subscription或ML Model Monitor
registration/subscription，以及Go將backend notification送往original `notifUri`／`notificationUri`的標準
resource行為。

### 2.2 Repositories

| Repository | Responsibility in this phase |
|---|---|
| `NWDAF/` | Model Provision/Monitor public SBI、private standard-shaped routing、resource mirror、sync、NRF advertisement、error mapping |
| `PyAnLF/` | Model demand/resolution/download/load、monitoring subscription、measurement binding、standard notification producer |
| `PyMTLF/` | Provision subscription/seed catalog、monitoring registration、monitor subscription intent、accuracy/retrain policy state |
| `nwdaf-docs/` | Canonical plan、evidence、decision與completion record |

### 2.3 Required Acceptance Levels

1. Model Provision與Monitor的standard method/path/body/status/Location tests。
2. Go與兩個Python backend的initial provision→monitor process-level round trip。
3. PyAnLF既有artifact/model lifecycle與measurement regression suite。
4. PyMTLF preserved-mechanics fixtures、approved old/new boundary tests與deterministic state-transition tests。
5. Backend unavailable、restart、sync及callback failure tests。
6. NWDAF build/lint/full test、PyAnLF lint/full test、PyMTLF lint/full test。

### 2.4 Explicitly Deferred

- Phase 5 ADRF subscription/fetch instruction/direct fetch與Mongo direct read。
- Phase 5使用ADRF fetch instruction/direct fetch或Mongo direct read取得retrain/training dataset；不再負責補足
  Phase 4 accuracy policy input。
- Phase 6 local training、generation completion、new artifact publication與updated/re-trained model reprovision。
- Analytics Feedback (`anaFeedbacks`)的domain decision；wire model可接受及轉送，但本phase不以它觸發新policy。
- Multiple-NWDAF FL/AGG、AoI routing/leaf data collection、DCCF及UDM Group retrieval；本phase只保留
  multi-AnLF-safe identity/context，不實作future topology。
- TLS、OAuth delegation、backend NF registration及跨Go restart持久化。
- External AnLF registration後由local PyMTLF discovery並主動訂閱remote AnLF；本phase只完成local backend
  orchestration與對外resource producer behavior。

---

## 3. Standard Evidence

### 3.1 Service Roles

TS 29.520 clause 4.5與TS 23.288 clauses 6.2A.1、6.2A.2規定：

- AnLF-side NWDAF可依Analytics ID、filter與target向MTLF-side NWDAF建立、修改或刪除ML Model Provision
  subscription。
- MTLF-side NWDAF判斷既有trained model是否適用；可用時以`Nnwdaf_MLModelProvision_Notify`提供unique model
  identifier與model file address。
- Model Provision subscription可要求immediate reporting；已有report時可在create/update response的
  `mLEventNotifs`提供，不得為了同步response等待future training。
- TS 23.288 clause 6.2E.2 steps 1至5明確把analytics subscription、Model Provision、AnLF register與MTLF
  monitor subscription串成同一程序。

TS 29.520 clause 4.7.1.1與4.7.1.3規定：

- AnLF-side NWDAF向MTLF-side NWDAF register/deregister其model使用與accuracy monitoring capability。
- MTLF-side NWDAF向AnLF-side NWDAF subscribe/update/unsubscribe accuracy monitoring event。
- AnLF-side NWDAF向subscription consumer送出`MLModelMonitorNotify`。

TS 23.288 clause 6.2E.3.2進一步說明AnLF開始使用且能監控model時register，不再使用時deregister；
clause 6.2E.3.3說明MTLF可依registration及local policy建立monitor subscription，AnLF再回報accuracy，
MTLF的degradation判斷是internal procedure。

Future topology引用需保持章節分工：TS 23.288 clause 6.1A是analytics aggregation，clause 6.1B是analytics
context/subscription transfer；Model Provision本身是clause 6.2A，MTLF-based monitoring是clause 6.2E。Current
phase不實作6.1A/6.1B，但`consumerId`與完整context不得妨礙future transfer/multiple-AnLF correlation。

因此本phase不把所有resource交給同一backend：Model Provision producer與Monitor registration producer是
PyMTLF；Model Provision consumer以及Monitor subscription/notification producer是PyAnLF。Go只持有NWDAF
standard boundary與routing state。

### 3.2 Resource And Method Matrix

依`nwdaf-docs/specs/openapi/TS29520_Nnwdaf_MLModelProvision.yaml`、
`TS29520_Nnwdaf_MLModelMonitor.yaml`與TS 29.520 clauses 5.4.3、5.4.5、5.6.3、5.6.5：

| Resource/operation | Method and path | Success |
|---|---|---|
| Create provision subscription | `POST /nnwdaf-mlmodelprovision/v1/subscriptions` | `201` + body + mandatory `Location` |
| Replace provision subscription | `PUT /nnwdaf-mlmodelprovision/v1/subscriptions/{subscriptionId}` | `200` + body or `204` |
| Delete provision subscription | `DELETE /nnwdaf-mlmodelprovision/v1/subscriptions/{subscriptionId}` | `204` |
| Provision notify | `POST {notifUri}` with non-empty array of `NwdafMLModelProvNotif` | `204` |
| Create registration | `POST /nnwdaf-mlmodelmonitor/v1/registrations` | `201` + body + mandatory `Location` |
| Delete registration | `DELETE /nnwdaf-mlmodelmonitor/v1/registrations/{registrationId}` | `204` |
| Create subscription | `POST /nnwdaf-mlmodelmonitor/v1/subscriptions` | `201` + body + mandatory `Location` |
| Replace subscription | `PUT /nnwdaf-mlmodelmonitor/v1/subscriptions/{subscriptionId}` | `200` + body or `204` |
| Delete subscription | `DELETE /nnwdaf-mlmodelmonitor/v1/subscriptions/{subscriptionId}` | `204` |
| Notify | `POST {notificationUri}` with `MLModelMonitorNotify` | `204` |

Create不使用private PUT upsert。Resource ID由owner backend產生UUIDv4，Go沿用同一ID並建立external
Location；restart replay只透過sync，不改變POST語意。

### 3.3 Standard Data Shapes

TS 29.520 clause 5.4.6與Release 18 Model Provision YAML定義：

- `NwdafMLModelProvSubsc`
  - required: non-empty `mLEventSubscs`, `notifUri`
  - optional: `notifCorreId`, `eventReq`, response-only `mLEventNotifs`, `failEventReports`, `suppFeats`
- `MLEventSubscription`
  - required: `mLEvent`, `mLEventFilter`
  - optional: `tgtUe`, target/expiry time, reporting condition, interoperability/use-case/input-data information與`modelId`
- `NwdafMLModelProvNotif`
  - required: non-empty `eventNotifs`, `subscriptionId`
- `MLEventNotif`
  - required: `event` and one of `mLFileAddr` or `mLModelAdrf`
  - initial local flow additionally requires`modelUniqueId`及HTTP(S) `mLFileAddr.mLModelUrl`；ADRF model retrieval
    不是本phase範圍
  - optional filter/target/validity資訊用來判定model applicability，不能被Go proxy丟棄

TS 29.520 clause 5.6.6與Release 18 YAML定義：

- `MLModelMonitorReg`
  - required: `modelId`
  - one of `consumerId` or `consumerSetId`
  - optional in V18.13 YAML: `modelAccuInd`, `mLEvent`, `mLEventFilter`, `tgtUe`
  - `modelAccuInd`依TS 29.520 clause 5.6.6.2.2是monitoring transfer indication；normal local registration
    省略，只有TS 23.288 clause 6.2E.3.2 analytics transfer情境才依程序使用
- `MLModelMonitorSub`
  - required: non-empty `modelIds`, `notificationUri`, `notifCorrId`
  - optional: `modelMetric`, `accuThreshold`, `eventReportReq`, response-only `immReport`,
    `mLEvent`, `mLEventFilter`, `tgtUe`, `suppFeat`
- `MLModelMonitorNotify`
  - required: `notifCorrId`
  - at least one of non-empty `modelAccuInfos` or `anaFeedbacks`
  - optional: `accuMeetInd`, `mLEvent`, `mLEventFilter`, `tgtUe`
- `MLModelAccuracyInfo`
  - required: `modelId`
  - optional: `mlModelAcc`, `deviation`, `inferenceNum`, `adrfId` xor `adrfSetId`,
    `dataSetTag`, `modelMetric`, `monitorInterval`

Corpus內存在一個minor version差異：OpenAPI attachment是TS 29.520 V18.13.0，而converted TS text是
V18.14.0。V18.14 clause 5.6.6.2.2在`MLModelMonitorReg`增加conditional `suppFeat`，V18.13 YAML沒有該
property；`MLModelMonitorSub`兩者都有`suppFeat`。本phase以V18.13 YAML作wire schema baseline，同時靠Go
raw JSON preservation及Python `extra="allow"`確保V18.14 registration的`suppFeat`不被丟棄，但不在沒有
對應attachment時自行宣稱完成新版本feature negotiation。

`MLModelMetric`依`TS29520_Nnwdaf_MLModelProvision.yaml`目前只定義`ACCURACY`；forward-compatible string
分支不得拿來私自編碼WAPE、MAE或traffic scale。本計畫以`modelMetric = ACCURACY`表示被監控的是model
accuracy，以`MLModelAccuracyInfo.deviation`承載本地定義的WAPE error ratio；不是新增enum value。

`MLModelAccuracyInfo`只有`modelId`必填，`deviation`、`mlModelAcc`、`modelMetric`、`inferenceNum`與
`monitorInterval`均為optional。外層`MLModelMonitorNotify`必須有`notifCorrId`，且至少提供非空
`modelAccuInfos`或`anaFeedbacks`；本accuracy flow使用非空`modelAccuInfos`。因此資料不足的period仍可送出只含
`modelId`、`inferenceNum`與`monitorInterval`的合法accuracy info，而不偽造`deviation`。

`mlModelAcc`依TS 29.520 clause 5.6.6.2.5是0至100的accuracy percentage，WAPE則是可大於1的error ratio；兩者
不可無損互換，所以本phase不填`mlModelAcc`。`accuMeetInd`是optional threshold outcome；local automatic
subscription不協商`accuThreshold`，因此也省略`accuMeetInd`。PyMTLF不得在未協商threshold時把省略或schema
default的false解讀成degradation。

TS 23.288 clause 5C.1保留prediction correctness與degradation判斷的implementation freedom；因此WAPE公式、
sample sufficiency及degradation policy是本地設計。Section 7會逐項標示哪些既有mechanics保留、哪些經使用者
決策明確替換或移除，不能把這次簡化誤當成無意間的port drift。

### 3.4 HTTP And Error Evidence

TS 29.520 clauses 5.4.2與5.6.2要求JSON使用`application/json`，Problem Details使用
`application/problem+json`，HTTP行為依TS 29.500。TS 29.520 clauses 5.4.7與5.6.7沒有定義額外的
Model Provision/Monitor application errors，因此不得發明Phase-specific status code。

Release 18 OpenAPI所列status：

| Operation | Declared error/redirect status |
|---|---|
| Provision subscription POST | `400, 401, 403, 404, 411, 413, 415, 429, 500, 502, 503, default` |
| Provision subscription PUT | `307, 308, 400, 401, 403, 404, 411, 413, 415, 429, 500, 502, 503, default` |
| Provision subscription DELETE | `307, 308, 400, 401, 403, 404, 429, 500, 502, 503, default` |
| Provision notification POST | `307, 308, 400, 401, 403, 404, 411, 413, 415, 429, 500, 502, 503, default` |
| Registration POST | `400, 401, 403, 404, 411, 413, 415, 429, 500, 502, 503, default` |
| Registration DELETE | `307, 308, 400, 401, 403, 404, 429, 500, 502, 503, default` |
| Subscription POST | `400, 401, 403, 404, 411, 413, 415, 429, 500, 502, 503, default` |
| Subscription PUT | `307, 308, 400, 401, 403, 404, 411, 413, 415, 429, 500, 501, 502, 503, default` |
| Subscription DELETE | `307, 308, 400, 401, 403, 404, 429, 500, 501, 502, 503, default` |
| Notification POST | `307, 308, 400, 401, 403, 404, 411, 413, 415, 429, 500, 502, 503, default` |

Backend configured但unusable時回`503 ProblemDetails`；backend transport成功但回傳無法解析或違反成功
contract的response時回`502 ProblemDetails`。Malformed JSON／mandatory field failure回`400`，unsupported
media type回`415`，body limit回`413`，missing resource回`404`。Go只轉送OpenAPI允許且well-formed的
peer `ProblemDetails`；private error body不得直接外露。

### 3.5 Local OpenAPI Gap

`NWDAF/go.mod`目前使用`github.com/free5gc/openapi v1.2.3`。該版本已有`nwdaf/MLModelProvision` generated
client及相關models，現有`internal/sbi/consumer/mtlf_service.go`也已使用它；implementation應先逐欄確認其
Release 18 coverage並重用可用generated types。該版本沒有`Nnwdaf_MLModelMonitor` generated client/models/
service-name constant。Local Release 18 Model Provision/Monitor YAML又引用本corpus未包含的
`TS29523_Npcf_EventExposure.yaml`、`TS29122_CommonData.yaml`、`TS29554_Npcf_BDTPolicyControl.yaml`與
`TS29574_Ndccf_DataManagement.yaml`，完整直接generation目前不可重現。

Monitor與generated Model Provision未涵蓋的Release 18欄位採scoped compatibility wire package：

1. 欄位、alias、required/optional、array cardinality及oneOf/anyOf直接依Release 18 YAML。
2. 可重用的common types優先使用current free5GC generated models。
3. 缺失的nested type只在該package內建最小typed compatibility representation並註明source `$ref`。
4. Go proxy保留original JSON bytes，typed parse只做已知mandatory/cardinality validation，避免未知R18欄位
   被decode/re-encode時silent drop。
5. Python wire models使用`extra="allow"`並以alias輸出，resource representation round trip保留未知欄位。
6. 不建立`map[string]interface{}` business DTO或`testdata/*.json`跨語言contract。

### 3.6 free5GC Exemplar Alignment

本phase以local snapshot `resources/references/free5gc-main@f64135d2`為結構參考：

- Primary CRUD exemplar: BSF
  - `NFs/bsf/internal/sbi/api_management.go`
  - `NFs/bsf/internal/sbi/processor/subscriptions.go`
  - direct evidence: POST collection、PUT/DELETE document、201+Location、200 representation、204及404分流。
- Callback/notification exemplar: PCF
  - `NFs/pcf/internal/sbi/processor/notifier.go`
  - `NFs/pcf/internal/sbi/consumer/notification.go`
  - direct evidence: callback handler與outbound notification consumer分離，成功處理後回204。

沒有local free5GC NF實作`Nnwdaf_MLModelProvision` producer或`Nnwdaf_MLModelMonitor`，所以resource ownership與
payload semantics來自3GPP，不是從BSF/PCF類推；exemplar只決定Go的router／handler／processor／consumer
分層。Current NWDAF Model Provision consumer只作target-repository characterization evidence。

---

## 4. Current State

### 4.1 Current Initial Provision Flow

Current repositories已具備大部分model artifact/runtime primitives，但ownership與wire仍不是target flow：

1. Go `internal/sbi/consumer/mtlf_service.go`使用free5GC generated ML Model Provision client建立/刪除external
   MTLF subscription；request目前由Go依analytics subscription組裝，沒有把model demand business ownership交給
   PyAnLF，且current create未填Release 18 required `mLEventFilter`。
2. Go `/mlmodel-notify`解析standard-like callback後，`internal/anlf/coordinator/model_provision.go`轉成custom
   `ModelProvisionEvent`／`ModelProvisionBinding`再送PyAnLF；Go仍解析provider、identity與runtime correlation。
3. PyAnLF已有pending provision runtime、stable `ModelIdentity`、generation/reference separation、safe artifact
   download/cache/load、shared model refcount及atomic activation，但model demand與standard provision resource不由
   PyAnLF擁有。
4. PyMTLF已有validated immutable `ArtifactRepository`及download endpoint，尚未擁有standard Model Provision
   subscription resource或configured seed applicability catalog。
5. PyAnLF sample/default model reference可讓runtime直接載入local initial bundle；target flow改由PyMTLF seed
   provision提供production initial model，bundle/loader behavior本身不改。

因此本phase不是從零實作model artifact，而是移除Go business coordination、補齊standard resource ownership，
將existing PyMTLF artifact producer與PyAnLF model consumer串成initial provision vertical slice。

### 4.2 Current Accuracy Flow

```text
PyAnLF model runtime automatically attaches one accuracy monitor
    -> periodic prediction/ground-truth match
    -> five local metrics + traffic scales
    -> custom ModelAccuracyReport
    -> Go auxiliary POST /model-accuracy-reports
    -> Go MTLF generation/dedup adapter
    -> Go trigger.go/state_store.go
    -> legacy ADRF/Daisy retrain path
```

現有matching與confidence readiness已由AnLF transition parity work校正；這次有意改變的是accuracy輸出與
report-window語意，而不是重新設計prediction/ground-truth alignment。Current問題是：

1. 沒有standard registration/subscription lifecycle。
2. PyAnLF是否監控由global config與model attach決定，不是由monitor subscription resource決定。
3. Wire body是custom envelope。
4. MTLF decision與config仍在Go。
5. Go仍透過custom report保存generation/dedup/retrain context。
6. PyMTLF目前只有artifact、health與sync foundation，尚無monitor/policy runtime。

### 4.3 Existing Sync Boundary

Phase 3已建立兩個backend共用的：

```text
GET /health/ready -> POST /internal/v1/sync -> USABLE
```

Go持續poll、用`processInstanceId`辨識backend restart，並在reconnect後送process-local snapshot。本phase
延伸typed snapshot，不建立第二套handshake，也不把policy history放到Go。

### 4.4 Resolved Standard Wire And Policy Boundary

目前custom report帶有：

- 五種named metrics：sMAPE、MAE、MSE、WAPE、NRMSE
- actual/predicted traffic scale
- sample count
- generation與report ID
- private scope ID
- analytics subscription／observation source retrain context

`MLModelMonitorNotify`無法原樣承載全部legacy input。D1已透過明確的product decision解決：production accuracy
contract只使用`deviation = WAPE`，PyMTLF只保留degradation path。Multi-metric selection、actual/predicted
traffic scale、chronic path、low-traffic overprediction path與minimum-traffic gate均是明確移除／替換，不要求
Phase 5再補資料以重建舊policy。

仍禁止把traffic scale塞進`deviation`、用future enum私自編碼metric、在standard body加入private
`metrics`／`generation`／`retrainContext`，或以缺省值偽造optional input。Section 14記錄D1的最終決策與
被取代方案。

---

## 5. Scope

### 5.1 Included

1. Public `nnwdaf-mlmodelprovision/v1` subscription與`nnwdaf-mlmodelmonitor/v1`
   registration/subscription routes。
2. Go/backend standard-shaped private routes與兩個service的standard notification callback route。
3. Release 18 generated/compatibility wire models與raw JSON preservation。
4. PyMTLF Model Provision subscription resource store、configured seed model catalog、existing immutable artifact
   URL publication及sync restore。
5. PyAnLF model demand resolver、provision subscription reconciliation、existing artifact download/validation/cache/load
   與analytics runtime binding。
6. PyMTLF Monitor registration resource store、UUID、create/delete及sync restore。
7. PyAnLF Monitor subscription resource store、UUID、create/replace/delete及sync restore。
8. PyAnLF model-use registration refcount/reconciliation。
9. PyMTLF registration-driven local monitor subscription intent/reconciliation。
10. Go callback routing到AnLF backend、MTLF backend或external notification URI。
11. PyAnLF monitoring由accepted monitor subscription控制，保留既有alignment/confidence/scope mechanics，
    並依本計畫改為report-window內的WAPE measurement。
12. Same model在不同group/context建立獨立monitor scope；相同canonical context以refcount共用。
13. Go MTLF degradation mechanics到PyMTLF的scoped port，以及本計畫明確簡化後的WAPE-only config/tests。
14. Go process-local provision/registration/subscription/callback mirror與unified sync extension。
15. NRF profile在完整service configuration下advertise `nnwdaf-mlmodelprovision`與
    `nnwdaf-mlmodelmonitor`。
16. 移除被final standard flow取代的custom model provision coordination、custom accuracy route production
    wiring及Go policy active caller/config ownership。

### 5.2 Excluded

1. 改寫PyAnLF target-slot matching、rounding、confidence gate、scope identity、observation ring或measurement
   concurrency；WAPE zero-denominator與report-window語意是本計畫明確例外。
2. 改寫被保留的MTLF population-std、strict comparison、reference isolation、N-in-M、GC或in-flight semantics。
3. 以custom wire補標準欄位不足。
4. Go保存policy baseline/history或Python process state persistence。
5. Go proxy dataset bytes。
6. Seed model training、training job、new generation transition、new artifact publication及updated/re-trained
   model reprovision；domain seam可保留測試，但production hookup屬Phase 6。
7. 移除全部legacy Daisy/training code；無production caller的殘留在Phase 7統一清除。
8. NRF-based external MTLF selection、remote AnLF orchestration、AGG/AoI/leaf NWDAF routing與FL。

---

## 6. Confirmed Decisions

1. Python backend不是獨立標準NF；Go持有唯一NWDAF identity。
2. Go source/config不使用`PyAnLF`／`PyMTLF`命名，只使用`anlfBackend`／`mtlfBackend`。
3. Internal security維持plain HTTP，不在本phase加入token/key/certificate exchange。
4. Model Provision subscription與Monitor registration由MTLF backend擁有；Monitor subscription、model runtime
   binding與measurement由AnLF backend擁有。
5. Standard create使用POST，replace使用PUT，delete使用DELETE。
6. Resource ID使用UUIDv4；Go保存process-local mirror，Go restart後不復原。
7. PyAnLF與PyMTLF不直連；standard-shaped provision/monitor operation經Go routing；artifact binary依標準URL由
   PyAnLF直接向PyMTLF下載，Go不proxy bytes。
8. Provision `notifUri`與Monitor `notificationUri`由Go internalize成per-subscription callback；Go保存original
   destination並負責external、AnLF-backend或MTLF-backend delivery。
9. Event subscription admission不等待initial provision、model download或monitor chain完成；PyAnLF先保存
   subscription並持續資料蒐集，model-backed inference在model成功載入後開始。
10. Registration DELETE成功與其衍生monitor subscription cleanup分離：PyMTLF先刪local registration並回204，
    再由worker完成standard subscription DELETE。
11. MTLF policy不在Go與PyMTLF雙跑；cutover後Go不再更新policy state或觸發retrain。
12. Local automatic monitor subscription省略`accuThreshold`並使用periodic reporting；PyMTLF直接以收到的WAPE
    執行local degradation policy。External request若帶`accuThreshold`，在percentage mapping未定義前回400
    ProblemDetails，不能接受後忽略。
13. Initial provision只提供PyMTLF啟動時已存在的configured seed model；本phase不因model demand啟動training。
14. PyAnLF只有在artifact驗證、load與runtime binding全部成功，且matching measurement capability存在後，才
    register monitoring capability。沒有stable model identity時不得建立monitor resource。
15. Provision subscription是model-demand level的long-lived resource；相容analytics subscription共用同一
    provision subscription與loaded model refcount，不因每個group重複下載。
16. Monitoring是context level：不同`mLEvent`／filter／`tgtUe`（例如group A與group B）分開register、回報及
    維護policy window；完全相同context才以refcount共用。
17. PyMTLF收到每次periodic monitor notification時event-driven evaluation；獨立timer只可做staleness/
    reconciliation，不重複計算同一report。
18. Any monitoring scope可觸發model-level retrain；同一model同時只存在一個retrain-in-flight，觸發後reset該
    model的全部scope windows。
19. Standard notification沒有report ID時不新增private idempotency token；spec未保證的duplicate delivery只記錄
    observability，不偽裝成標準能力。
20. Future multiple-AnLF相容性由`consumerId`與standard context保存；current phase不實作AGG/AoI routing，但
    不得以single AnLF或single scope作model policy key。
21. Accuracy policy只有degradation path且唯一metric為WAPE；chronic、low-traffic、multi-metric與traffic gate
    不進入PyMTLF production config/runtime。
22. 「穩定後才回報」定義為measurement data sufficient，不要求WAPE數值先收斂；跨report的穩定性由PyMTLF
    baseline、z-score與N-in-M判斷。

---

## 7. Behavioral Inventory

### 7.1 Existing Model Artifact And Runtime Lifecycle: Preserve

Initial Model Provision不是重寫model loader。以下既有行為是本phase的preserve contract：

| Behavior | Current source of truth |
|---|---|
| Four-file bundle (`config.json`, `model.py`, `model.npy`, `scaler.pkl`)及bundle-driven inference config | `PyMTLF/src/py_mtlf/core/artifacts.py`, `PyAnLF/src/py_anlf/core/model_manager.py`, `nwdaf-daisy-improvement-plan.md` |
| PyMTLF immutable content-addressed publication、safe archive validation及integrity response headers | `PyMTLF/src/py_mtlf/core/artifacts.py`, artifact API tests |
| PyAnLF HTTP(S) origin allowlist、300-second default timeout、streaming size limits、no redirect與safe extraction | `PyAnLF/src/py_anlf/core/model_manager.py`及tests |
| Identity/generation/reference-aware cache、per-key concurrency protection及verified re-download | `PyAnLF/src/py_anlf/core/model_manager.py`及tests |
| Candidate先prepare/load，成功後atomic runtime commit；失敗時保留仍有效的old runtime | `PyAnLF/src/py_anlf/core/runtime_manager.py`及model lifecycle tests |
| Same loaded model shared by subscription runtimes並以refcount/reservation管理 | `PyAnLF/src/py_anlf/core/runtime_manager.py` |
| Stable model identity、loaded generation與artifact locator分離 | `../anlf-backend-transition/R3 Model Identity Generation And Provision Correctness.md`及current tests |

本phase只把「誰發起provision、誰擁有subscription、standard notification如何到達PyAnLF」切到final owner；不得
趁機改bundle schema、feature order、dynamic model loading、cache validation或atomic activation semantics。
Existing Go `internal/sbi/consumer/mtlf_service.go`的generated client使用與POST/DELETE interop tests可作
characterization，但Go不再擁有model demand或runtime selection business state。

### 7.2 PyAnLF Measurement: Preserve And Explicitly Refine

以下以current PyAnLF production code與tests為source of truth；表內未另行標示的行為均保留：

| Behavior | Source |
|---|---|
| Per-model monitor與per-scope state | `core/accuracy/monitor.py` |
| Exact target-slot matching及Go-compatible rounding | `core/accuracy/monitor.py`, `core/alignment.py` |
| Source/IP last-wins與cross-source aggregation | accuracy parity fixtures |
| UL/DL separate samples | `core/accuracy/metrics.py` |
| confidence-zero與unresolved scope skip | `core/accuracy/monitor.py` |
| periodic snapshot/evaluate/commit與overlap protection | `core/accuracy/monitor.py` |
| derived prediction miss count與pending prediction retry/expiry | `core/accuracy/monitor.py` |
| generation replacement清除old pending state | `core/accuracy/monitor.py` |
| canonical target/filter/scope | `core/accuracy/identity.py` |
| per-source/IP bounded observation rings | `core/observation_store.py` |

本phase有意進行以下measurement contract refinement：

1. Production accuracy output只計算WAPE，不再輸出sMAPE、MAE、MSE、NRMSE或traffic scale給policy。
2. 對同一report window中的matched pairs，令
   `A = sum(abs(actual_i))`、`E = sum(abs(predicted_i - actual_i))`：
   - `A > 0`時，`WAPE = E / A`，不cap到1；
   - `A = 0`且`E = 0`時，`WAPE = 0`；
   - `A = 0`且`E > 0`時，`WAPE = 1`。
3. Prediction與ground truth是integer traffic values，不加入epsilon/tolerance config。這明確取代current
   PyAnLF與historical Go在`A = 0`時一律回0的行為。
4. `ground_truth_check_interval`與standard reporting period分離。Local defaults為30秒poll ground truth、90秒
   report window、`min_matched_predictions = 2`。
5. Matched pairs可跨同一report window內多次ground-truth poll累積，但不得跨report window保留。這取代current
   「每次check不足就丟棄，且`check_interval`同時是check/report cadence」的round定義。
6. Window結束時，matched pairs達門檻才計算並回報WAPE；不足時仍送standard periodic notification，但省略
   `deviation`。Insufficient window不更新PyMTLF baseline/reference/hit window。
7. `inferenceNum`依TS 29.520表示該interval內執行的inference數，不是matched pair數；matched門檻只留在
   PyAnLF local readiness state。

本phase不恢復historical Go startup-only 120秒warm-up。Confidence為0的prediction不進accuracy matching並記錄
observability；resolved scope、exact target-slot ground truth match及pending retry/expiry仍是必要條件。「可靠」只
代表該window資料足夠，不要求WAPE變異先收斂，避免和PyMTLF的temporal degradation decision重複。

### 7.3 Go MTLF Policy: Preserve Selected Mechanics In PyMTLF

以下以`NWDAF/internal/mtlf/trigger.go`、`state_store.go`、`pkg/factory/config.go`及tests為historical oracle；只有
列為preserve的mechanics要求parity：

1. WAPE observation存在時才evaluation；省略`deviation`的insufficient report不更新policy state。
2. Per-model/per-scope isolated state。
3. Recent observation ring與separate healthy degradation reference ring。
4. Baseline sample gate。
5. Fixed WAPE floor。
6. Population standard deviation與`max(std, minStd)` z-score denominator。
7. Strict `zscore > threshold`及`currentWape > fixedWapeFloor`比較。
8. 異常樣本不污染degradation reference；baseline未滿時仍建立reference。
9. Degradation path使用N-in-M hit window；達標才觸發。
10. Any scope可觸發model-level retrain，trigger後reset該model所有scope decision windows。
11. Retrain-in-flight時跳過所有new reports。
12. Scope TTL GC及empty model removal。
13. Generation advance resets model policy state；stale generation behavior保留為domain transition seam，
    production source於Phase 6接回。
14. Retrain terminal success/failure清除in-flight的domain behavior保留，production source於Phase 6接回。

### 7.4 WAPE-only Policy Defaults

PyMTLF第一版採用目前實驗已調整的WAPE defaults；它們是可調參數，不是3GPP wire contract：

| Setting | Default |
|---|---:|
| recent buffer size | `12` |
| minimum baseline reports | `5` |
| minimum std | `0.14` |
| fixed WAPE floor | `0.05` |
| z-score threshold | `1.3` |
| decision window size | `5` |
| required hits | `3`, capped to window size |
| scope TTL | `600s` |

PyMTLF active config不包含`primaryMetric`、`degradationPolicy.minDecisionTrafficScale`、chronic、low-traffic或
`consecutiveBreaches` fallback。Go sample config移除`mtlf.accuracyPolicy` active ownership；historical Go config
只作被保留mechanics與old/new test evidence，不控制production。

### 7.5 Explicitly Replaced

- Global config-only accuracy activation -> accepted `MLModelMonitorSub` activation。
- Go-owned per-analytics ML Model Provision request/correlation -> PyAnLF model-demand registry與PyMTLF-owned
  standard provision subscription。
- Per-subscription duplicate model download -> compatible model resolution、long-lived provision subscription與
  shared loaded-model refcount。
- Custom provider field in provision body -> routed MTLF provider namespace。Local Release 18 Model Provision YAML
  沒有`modelProviderId` property，不把existing private field偽裝成standard schema。
- Custom `ModelAccuracyReport` wire -> `MLModelMonitorNotify`。
- Multi-metric report -> `deviation = WAPE`；`modelMetric = ACCURACY`只標示standard monitored metric category。
- Historical WAPE `A = 0 -> 0` -> section 7.2明確的zero-denominator三分支。
- Combined measurement `check_interval` -> 30秒ground-truth polling與90秒standard report window。
- Per-check round-local discard -> same-report-window accumulation；report-window boundary後仍discard。
- Private `scope_id` wire -> canonical key由MTLF provider namespace、`modelId`、`mLEvent`、
  AnLF `consumerId`、`mLEventFilter`與`tgtUe`在PyMTLF內建立；monitor subscription ID只作resource/correlation metadata，
  不拆散語意相同scope的policy history。
- Private `report_id` dedup -> no custom token；standard callback outcome與observability。
- Private generation in notify -> PyMTLF-owned current model state；Phase 6 model update event重置policy。
- Private retrain context -> PyMTLF retrain intent的triggering/active scope inventory；Phase 5再由backend-owned
  dataset selection取得training data，不要求monitor notification承載dataset context。
- Go policy config/runtime -> PyMTLF config/runtime。
- Legacy primary metric selection、traffic eligibility、chronic與low-traffic paths -> intentionally removed from
  production policy；這是使用者核准的redesign，不受general port-parity要求約束。

### 7.6 Remove As Obsolete From Production

- Go-owned model selection/provision correlation與custom `ModelProvisionEvent` forwarding path，於standard
  initial provision與PyAnLF-owned binding完成後退出production。
- Go auxiliary `POST /model-accuracy-reports` route。
- `internal/anlf/contract.ModelAccuracyReport` production transport。
- Go processor `HandleModelAccuracyReport` production wiring。
- Go MTLF接受custom report後做legacy adapter的active path。
- PyAnLF `accuracy_monitor.report_delivery.callback_uri` custom endpoint設定。

若`trigger.go`、`state_store.go`或相關legacy training code因Phase 5/6直接dependency暫時必須留在tree，必須無
production caller、標記`legacy cleanup`並交由Phase 7；不能繼續從Go config啟用。

---

## 8. Target Resource Architecture

### 8.1 Public Routing

```text
POST/PUT/DELETE Model Provision subscriptions
    External consumer or AnLF backend -> Go SBI -> MTLF backend

NwdafMLModelProvNotif[]
    MTLF backend -> Go callback router -> AnLF backend or external notifUri

POST/DELETE registrations
    External consumer -> Go SBI -> MTLF backend

POST/PUT/DELETE subscriptions
    External consumer -> Go SBI -> AnLF backend

MLModelMonitorNotify
    AnLF backend -> Go callback router -> MTLF backend or external notificationUri
```

Go handler只處理HTTP concern；processor選backend、resource mirror與error mapping；backend client封裝outbound
private HTTP。不得在handler加入model selection、download/load或monitor policy。

### 8.2 Initial Provision And Monitoring Loop

1. Consumer建立analytics subscription。PyAnLF先commit subscription/runtime intent；不等待model取得或monitor
   setup才回覆Events Subscription create。
2. PyAnLF把standard `mLEvent`、canonical filter、`tgtUe`及runtime requirements轉成model demand，先在loaded
   model catalog尋找仍有效且applicability涵蓋該demand的model。不得只以Analytics ID相同就判定compatible。
3. 若沒有compatible model，PyAnLF建立或共用一個long-lived `NwdafMLModelProvSubsc` desired state；concurrent
   equivalent demand coalesce成一個POST intent。Request保留`mLEvent`、filter、target、`notifCorreId`及
   immediate reporting requirement。
4. PyAnLF透過Go internal standard-shaped route POST；Go internalize `notifUri`後route到PyMTLF。PyMTLF建立
   UUID subscription、保存standard representation並回201+Location。
5. 若matching configured seed model已存在，PyMTLF可依standard immediate-report規則在response的
   `mLEventNotifs`提供；否則經Go callback送出non-empty `NwdafMLModelProvNotif` array。兩條delivery path進入
   同一PyAnLF prepare/commit model activation procedure。
6. Notification至少提供stable `modelUniqueId`與HTTP(S) `mLFileAddr.mLModelUrl`。Provider namespace由routed
   MTLF target決定；local Release 18 wire不加入private `providerId`。
7. PyAnLF直接下載PyMTLF artifact URL，沿用既有bundle validation、cache與candidate load。只有validation、load
   與atomic runtime binding全部成功後model才成為`READY`；initial failure保持runtime pending並由provision
   reconciliation retry，不建立monitor resource。
8. Compatible second analytics subscription重用loaded model與provision subscription，只增加model-use refcount；
   不重新POST或download。若existing provision representation尚未涵蓋new demand，PyAnLF以standard PUT full
   replacement更新active demand union；其monitoring context若不同，仍進入下一步建立獨立monitor scope。
9. PyAnLF model-use registry對`model key + monitoring context`增加refcount。First reference建立
   `MLModelMonitorReg`：
   - `consumerId` = sync中的containing NWDAF `nfInstanceId`
   - `modelId` = active logical model ID
   - normal local flow省略`modelAccuInd`；它是analytics transfer indication，不是monitor-capability flag
   - `mLEvent`／`mLEventFilter`／`tgtUe` = canonical runtime context
10. PyAnLF透過Go POST；PyMTLF建立UUID registration並回201+Location，不同步等待monitor subscription chain。
11. PyMTLF reconciliation worker依local policy建立一個standard `MLModelMonitorSub`；第一版一個
   registration scope對一個subscription，避免model array與scope correlation模糊。
12. PyMTLF透過Go POST；Go internalize `notificationUri`；PyAnLF建立UUID subscription並啟動matching
    measurement binding。
13. PyAnLF依reporting period分scope送出`MLModelMonitorNotify`。Sample sufficient時`deviation`帶WAPE並由
    PyMTLF立即更新該scope policy；sample insufficient時仍送notification但省略`deviation`，PyMTLF只更新
    liveness/observability。不另以timer重複evaluate相同report。
14. Last monitoring-context reference消失時，PyAnLF DELETE registration；PyMTLF先commit delete並回204，再
    enqueue對應monitor subscription DELETE。
15. Last compatible model-demand reference消失時，PyAnLF DELETE provision subscription。Model可保留在validated
    disk cache，但不再視為active binding；cleanup不延遲consumer Events Subscription DELETE response。

Standard wire不加入`providerId`。`modelId`由target MTLF namespace界定；current local flow的target是唯一
MTLF backend，所以PyMTLF以自己的stable provider namespace加`modelId`形成internal model key。未來
external MTLF target selection必須由Go routing/NRF結果決定，不能把provider identity塞進standard body。

Control-plane reconciliation intent不使用drop-oldest queue。相同logical key coalesce，只保存latest desired
state；transport/503使用capped exponential backoff直到成功、intent取消或process shutdown。

### 8.3 Model Reuse And Scope Isolation

Identity分成三層，不能互相替代：

```text
ModelKey
  routed provider namespace + modelUniqueId

ModelDemandKey
  provider target + canonical requested event/filter/target/interoperability requirements

MonitoringScopeKey
  ModelKey + AnLF consumerId + canonical mLEvent/filter/tgtUe
```

- `ModelDemandKey`用於coalesce provision intent；returned model applicability可涵蓋多個demand。Model catalog
  必須以returned/requested filter、target、validity與runtime compatibility判斷coverage，不能要求group字串完全
  相等才能共享，也不能只因event相同就盲目共享。
- Group A與Group B若都被同一generic `UE_COMMUNICATION` seed model涵蓋，兩個analytics runtime共用
  `ModelKey`及provision resource。Generic provision entry已涵蓋兩者時不需wire mutation；否則PyAnLF以PUT
  更新同一resource的`mLEventSubscs` active-demand union，不建立第二個POST/download。Monitoring仍因`tgtUe`
  不同形成兩個registration/subscription/correlation。
- 兩個完全相同的consumer subscriptions若canonical context相同，共用monitoring scope並增加refcount，避免
  duplicate report污染policy window。
- PyMTLF policy state以`MonitoringScopeKey`隔離baseline/window，但retrain intent與in-flight以`ModelKey`
  聚合。任一scope達標即可觸發一次model-level retrain並reset該model所有scope decision windows。
- Future多個AnLF即使model/event/target相同，也由不同`consumerId`形成不同scope；這符合TS 23.288 clause
  6.2E.3.3 step 8允許MTLF考量來自一個或多個AnLF的multiple accuracy information。

### 8.4 External Subscription Notification

Go收到external Model Provision或Monitor subscription create時：

1. 保存external `notifUri`／`notificationUri`及correlation ID。
2. 以per-resource Go callback URI替換送給owner backend的callback URI。
3. External response body恢復consumer原本URI，不洩漏private callback URI。
4. Owner backend POST standard notify到Go callback。
5. Go依route mirror POST原始standard body到external URI。
6. External peer回204時Go回204；redirect或standard error依OpenAPI處理／轉送。

Internal PyMTLF subscription使用同一callback route，但destination kind是`MTLF_BACKEND`，Go把body轉給
MTLF backend，不做external network call。

Internal PyAnLF Model Provision subscription的destination kind是`ANLF_BACKEND`，Go把PyMTLF notification
轉給AnLF backend，不讓PyMTLF直連PyAnLF。

### 8.5 NRF Advertisement

`nnwdaf-mlmodelprovision`及`nnwdaf-mlmodelmonitor`只有在所需backend configured/enabled且本build實作各自
完整resource surface時加入NF profile。瞬時readiness不造成NRF profile反覆更新；runtime unavailable時該
operation回503。Current local PyAnLF不透過NRF選local PyMTLF；它只呼叫Go，Go依configured backend route。

Go context需要為不同Nnwdaf service保存不同`serviceInstanceId`，不能把目前Events Subscription的單一ID
重用成所有service identity。Current generated model沒有service-name constant時使用typed local constant
`models.ServiceName("nnwdaf-mlmodelmonitor")`，不修改generated code。

---

## 9. Go/Backend Private Contracts

Private path可與public path不同，但method、body、success representation與ProblemDetails semantics相同：

### 9.1 MTLF Backend Producer Routes

```text
POST   /internal/v1/ml-model-provision/subscriptions
PUT    /internal/v1/ml-model-provision/subscriptions/{subscriptionId}
DELETE /internal/v1/ml-model-provision/subscriptions/{subscriptionId}
POST   /internal/v1/ml-model-monitor/registrations
DELETE /internal/v1/ml-model-monitor/registrations/{registrationId}
POST   /internal/v1/ml-model-monitor/notifications
```

### 9.2 AnLF Backend Producer Routes

```text
POST   /internal/v1/ml-model-provision/subscriptions/{subscriptionId}/notifications
POST   /internal/v1/ml-model-monitor/subscriptions
PUT    /internal/v1/ml-model-monitor/subscriptions/{subscriptionId}
DELETE /internal/v1/ml-model-monitor/subscriptions/{subscriptionId}
```

### 9.3 Backend-Initiated Go Routes

Existing Go internal callback base增加相同standard-shaped operation：

```text
POST   /internal/v1/ml-model-provision/subscriptions
PUT    /internal/v1/ml-model-provision/subscriptions/{subscriptionId}
DELETE /internal/v1/ml-model-provision/subscriptions/{subscriptionId}
POST   /internal/v1/ml-model-provision/subscriptions/{subscriptionId}/notifications
POST   /internal/v1/ml-model-monitor/registrations
DELETE /internal/v1/ml-model-monitor/registrations/{registrationId}
POST   /internal/v1/ml-model-monitor/subscriptions
PUT    /internal/v1/ml-model-monitor/subscriptions/{subscriptionId}
DELETE /internal/v1/ml-model-monitor/subscriptions/{subscriptionId}
POST   /internal/v1/ml-model-monitor/subscriptions/{subscriptionId}/notifications
```

Request origin由已知listener/route決定，不加入body field。這些route是少數允許的standard-shaped private
boundary，不建立general RPC envelope。

---

## 10. Resource State And Sync

### 10.1 Go Mirror

Go context新增process-local typed records：

```text
MLModelProvisionSubscriptionRoute
  subscriptionId
  accepted external standard representation
  backend representation with internal callback URI
  initiator kind: ANLF_BACKEND | EXTERNAL
  destination kind: ANLF_BACKEND | EXTERNAL
  destination notification URI
  notification correlation ID

MLModelMonitorRegistrationRoute
  registrationId
  accepted standard representation
  initiator kind: ANLF_BACKEND | EXTERNAL

MLModelMonitorSubscriptionRoute
  subscriptionId
  accepted external standard representation
  backend representation with internal callback URI
  destination kind: MTLF_BACKEND | EXTERNAL
  destination notification URI
  notifCorrId
```

Go mirror只支援routing、Location、callback及reconnect sync，不保存model catalog、download/cache state、runtime
binding、measurement sample、baseline、hit window、policy history或retrain decision。

### 10.2 Sync Extension

Unified `BackendSyncRequest`增加typed sections：

- `mlModelProvisionSubscriptions`
- `mlModelMonitorRegistrations`
- `mlModelMonitorSubscriptions`

投影規則：

1. PyMTLF收到全部active provision subscriptions、monitor registrations及destination為MTLF backend的monitor
   subscription correlation。
2. PyAnLF收到initiator為AnLF backend的provision subscription correlation、全部active monitor subscriptions及
   initiator為AnLF backend的monitor registration correlation。
3. PyAnLF restore順序為Events Subscription/model demand、provision correlation與model activation先，monitor
   registration/subscription intent後。沒有READY model不得先恢復measurement worker。
4. PyMTLF atomically replace provision/registration/subscription projection；對active provision subscription
   reconciliation current configured seed notification，讓restarted PyAnLF可重新驗證cache或下載，不由Go保存
   last model payload。
5. PyMTLF為restored monitor scope建立fresh empty policy windows；Go不復原
   lost baseline。
6. Sync失敗不得留下partial resource mutation；backend維持unusable並由polling retry。
7. Go restart後mirror消失，實驗重跑；不做database persistence。

### 10.3 Restart Convergence

- PyAnLF restart：Go replay Events/provision/monitor resources；PyMTLF re-notify current seed，PyAnLF以既有
  validated cache或重新下載恢復model binding，之後才恢復monitor worker。Pending prediction與未送notification
  可遺失。
- PyMTLF restart：Go replay provision subscriptions、registrations與owned monitor subscription correlation；
  seed catalog由config/artifact repository重建，policy baseline/window重建，已完成但未送出的retrain intent可
  遺失。
- Backend process UUID變更觸發完整READY→SYNCING→USABLE。
- Live transport failure將對應backend標記unavailable並立即喚醒polling。

---

## 11. Initial Model Provision Design

### 11.1 PyMTLF Seed Model Catalog

PyMTLF啟動時從config載入至少一個用於planned `UE_COMMUNICATION` experiment的seed model descriptor。Descriptor
保存stable model identity、artifact key、public URL與standard applicability metadata；artifact本體必須已由
existing `ArtifactRepository`驗證及publish。本phase不新增另一套artifact store或讓Go讀取artifact directory。

Current first seed來源是PyAnLF `artifacts/initial`的既有four-file bundle；implementation以現有bundle bytes/config
作characterization，將它package/import到PyMTLF `ArtifactRepository`，保持model identity與inference metadata。
Cutover後PyAnLF sample config不再以`default_model_reference`直接擁有production initial model。PyMTLF runtime
`data/`繼續gitignore；reproducible local seed import/setup寫入README或repository-native helper，不commit mutable
artifact repository state。

Config validation至少檢查：

1. model ID合法且在同一provider namespace內唯一；
2. artifact存在、metadata identity與descriptor一致；
3. public base URL為允許的HTTP(S) origin且可形成`mLModelUrl`；
4. event/filter/target applicability可canonicalize；
5. planned local experiment至少有一個descriptor可涵蓋所需`UE_COMMUNICATION` demand。

無matching model的standard subscription仍可成為resource，但PyMTLF不得假裝已提供model；使用
`failEventReports`或保持等待future availability必須依request/reporting semantics處理。本phase acceptance以
configured matching seed存在為前提，不因miss觸發training。

### 11.2 PyMTLF Provision Subscription Service

PyMTLF新增獨立resource store，保存UUID、accepted standard representation、canonical demand及current delivered
seed correlation。POST/PUT/DELETE採與monitor resource相同的prepare-commit-finalize及standard status：

1. validate mandatory fields、array cardinality與supported local artifact delivery mode；
2. canonicalize each `MLEventSubscription`並resolve matching seed；
3. atomically commit resource；
4. `eventReq.immRep=true`且report available時才在response加入`mLEventNotifs`；
5. 非immediate或後續reconciliation透過standard callback送non-empty notification array；
6. PUT是full replacement，missing resource回404；DELETE commit後回204並停止future delivery。

Callback delivery沿用bounded retry分類：transport/timeout/5xx可重試，well-formed 4xx不重試。Standard payload沒有
private event ID；initial duplicate notification以`subscriptionId + model identity + artifact reference + accepted
resource revision`做local reconciliation，不能把相同initial model誤判成new generation。Updated model的正式
generation/replay語意由Phase 6完成。

### 11.3 PyAnLF Model Demand And Activation

PyAnLF在Events Subscription runtime之外維護model-demand registry及loaded-model catalog：

```text
NO_MODEL
  -> PROVISION_PENDING
  -> DOWNLOADING
  -> LOADING
  -> READY

PROVISION_PENDING / DOWNLOADING / LOADING
  -> RETRY_PENDING on retryable failure
```

- Events Subscription create先commit；model未READY時仍可進行collection，但不得啟動model-backed inference或
  accuracy monitor。Existing fallback output是否送出維持current characterized Events behavior，不由本phase
  重新定義。
- Download/load使用current `ModelManager`，不建立第二個HTTP downloader或新的bundle DTO。
- Initial load失敗時沒有active model可回退，runtime保持pending；replacement失敗保留old model的能力屬Phase 6，
  但existing prepare/commit primitive不得移除。
- Compatible model resolution以standard request/response applicability及runtime interoperability為依據。Second
  demand命中existing model時直接attach shared runtime；若相同model的provision subscription仍active則增加refcount。
- Last model user離開時停止active binding並reconcile provision DELETE；validated disk cache可保留，disk GC不是
  本phase範圍。

### 11.4 Initial Provision Boundary With Monitoring

Model loaded不等於自動回報accuracy。PyAnLF只有在以下全部成立才建立monitor registration intent：

1. stable routed provider namespace及`modelUniqueId`已知；
2. artifact已驗證、model已成功load並atomically bind到analytics runtime；
3. concrete `mLEvent`／filter／target context已知；
4. PyAnLF對該context具備prediction/ground-truth measurement capability。

Provision與Monitor resource各自有refcount與retry lifecycle；不得在provision POST response內同步等待MTLF反向
monitor subscription。這避免Consumer Events Subscription、Model Provision與Monitor形成跨process request
deadlock。

---

## 12. PyAnLF Monitoring Subscription Design

### 12.1 Resource Store

PyAnLF新增獨立的ML Model Monitor subscription service，不與NWDAF Events Subscription resource store混用。
每個resource保存原始standard representation、resource UUID、canonical model/scope bindings及reporting state。

Create/replace採prepare-commit-finalize：

1. Validate standard body與supported reporting behavior。
2. Resolve active model ID及matching analytics context。
3. Prepare monitor binding與optional immediate cached report。
4. Atomically commit resource representation與binding。
5. Commit後才停止old worker/binding或送side effect。

PUT是full replacement；404不create。DELETE local commit後回204，不等待任何已在transport中的notification。

### 12.2 Activation Rule

Current `accuracy_monitor.enabled`不再代表「所有active model自動送custom report」。它只代表backend具備
measurement capability。真正active reporting必須同時滿足：

1. model有stable `modelId`；
2. matching monitor subscription存在；
3. `mLEvent`／filter／target與runtime context相容；
4. subscription尚未被delete或reporting lifecycle終止。

Prediction bookkeeping只為active monitor binding建立。Registration尚未完成或MTLF尚未subscribe時，analytics
inference照常，accuracy measurement可暫時缺失，不使Events Subscription失敗。

### 12.3 Reporting

- Omitted reporting requirements使用local defaults：`ground_truth_check_interval = 30s`、periodic
  `repPeriod = 90s`、`min_matched_predictions = 2`。Ground-truth poll cadence與report window是不同設定，不能再
  共用模糊的`check_interval`。
- Explicit `PERIODIC` + `repPeriod`控制該monitor subscription的report-window與delivery cadence；ground-truth
  polling仍可在同一window內執行多次。
- `immRep=true`只回傳已存在且仍有效的cached measurement；不得為了HTTP response阻塞等待future ground truth。
- Local automatic subscription省略`accuThreshold`。PyAnLF在sample sufficient時以`deviation`回報WAPE ratio，
  `inferenceNum`回報該interval執行的全部inference數、`monitorInterval`回報measurement/report window，並建議
  填`modelMetric = ACCURACY`；不填`mlModelAcc`或`accuMeetInd`。
- External request若提供`accuThreshold`，在percentage mapping尚未由後續decision定義前以400 ProblemDetails
  拒絕，不能接受後忽略或拿raw deviation直接和0..100 threshold比較。這不阻擋current local periodic flow，
  但不得宣稱已支援threshold-reporting optional behavior。
- 每個report window只累積該window內的matched pairs。Window結束時若matched count至少為2，送出
  `modelId + deviation(WAPE) + inferenceNum + monitorInterval + modelMetric(ACCURACY)`；不足時送出
  `modelId + inferenceNum + monitorInterval`並省略`deviation`與`accuMeetInd`，然後清空window-local samples。
- PyMTLF收到沒有`deviation`的合法periodic report只更新liveness/observability，不更新baseline、reference或
  decision window。不得把absence或`accuMeetInd=false`當成degradation。
- Unsupported reporting mode或自相矛盾欄位在resource create/replace時以400 ProblemDetails拒絕，不先接受再靜默忽略。
- Notify使用subscription的`notifCorrId`，context使用standard `mLEvent`／`mLEventFilter`／`tgtUe`。

Measurement worker與HTTP delivery必須有bounded shutdown；replace/delete後的stale worker不得送notification。

---

## 13. PyMTLF Accuracy Policy Design

### 13.1 Package Boundary

建議保持小型、explicit結構：

```text
src/py_mtlf/
  api/ml_model_monitor.py
  core/monitor_registration.py
  core/monitor_subscription.py
  core/accuracy_policy.py
  core/policy_state.py
  core/retrain_intent.py
  wire/ml_model_monitor.py
```

不建立plugin framework、generic workflow engine或database repository。Policy與state使用plain Python class／
dataclass及lock；FastAPI router只做HTTP boundary。

### 13.2 Policy Observation

Policy domain type與wire model分離，但不是另一個HTTP DTO。它只需要：

```text
model key
scope key
WAPE deviation (optional)
inference count
monitor interval
```

`deviation` absent保持`None`，adapter不得補0。只有WAPE存在時才更新degradation state；ADRF/DataSetTag可以作
standard notification metadata保留，但不屬於Phase 4 policy必要input，也不觸發Phase 5 enrichment。

### 13.3 Deterministic Degradation Port

1. 先從現有`policy_oracle_v1.json`與Go tests抽取被保留的degradation mechanics；fixture必須註明historical
   source與本計畫的old/new boundary，不要求removed policy paths parity。
2. Clock注入policy/state store；TTL tests不使用fixed sleep。
3. Ring與hit window的overwrite order、population std、`minStd`及strict comparisons保持相同。
4. Per-model serialization防止同時notification造成window race或double trigger。
5. Trigger時先reset all model scope windows，再atomically claim model-level in-flight。
6. Phase 4 production sink只建立in-memory retrain intent及observability。Intent保存`ModelKey`、triggering
   `MonitoringScopeKey`與當下active scope keys，讓Phase 5/6能在修正degraded group時同時避免healthy scope
   regression；本phase不決定dataset composition或執行training。
7. Phase 5接dataset，Phase 6接training。
8. `complete_retrain`與`advance_generation`保留domain method及tests，但本phase不偽造production completion。

### 13.4 Policy Fixture Completion

PyMTLF fixtures至少涵蓋：

- decision window retains recent hits
- decision window tolerates misses
- anomalous sample does not pollute degradation reference
- zero history still requires absolute gate
- any scope triggers model-level retrain
- exact WAPE policy defaults and required-hits capping
- buffer overwrite ordering
- simultaneous same-model notification serialization
- notification without deviation does not mutate policy history
- WAPE zero-denominator three-way behavior and WAPE greater than one
- Group A degraded while Group B healthy triggers one model-level intent and resets both decision windows

每個fixture保存historical NWDAF commit、source test name、logical clock、ordered input及每step expected state。

---

## 14. D1 Resolved: Standard WAPE Deviation

### 14.1 Decision

D1採用simplified standard-scalar design，這是使用者明確核准的policy redesign：

1. Standard wire只使用`MLModelMonitorNotify.modelAccuInfos[].deviation`承載WAPE。
2. `modelMetric`可填standard `ACCURACY`；不新增WAPE enum、不填percentage `mlModelAcc`。
3. PyMTLF只實作degradation path；legacy multi-metric selection、traffic inputs/gates、chronic及low-traffic
   path均移除。
4. 不建立temporary private accuracy extension，也不讓Phase 5讀dataset重建Phase 4 policy observation。
5. 因此standard callback到degradation decision與retrain intent可在Phase 4完成；Phase 5只負責retraining所需
   dataset selection與direct retrieval。

### 14.2 WAPE Wire Semantics

- WAPE使用ratio：`0.24`表示24% error，數值越高越差。
- `A > 0`時不cap，故WAPE可以大於1；不能轉成0至100的`mlModelAcc`而假裝等價。
- `A = 0, E = 0`回0；`A = 0, E > 0`回1，避免zero-traffic overprediction被誤報為perfect。
- Reliable periodic report填`modelId`、`deviation`、`inferenceNum`、`monitorInterval`及建議的
  `modelMetric = ACCURACY`。
- Insufficient periodic report仍有non-empty `modelAccuInfos`，但只填`modelId`、`inferenceNum`與
  `monitorInterval`；省略`deviation`、`mlModelAcc`與`accuMeetInd`。

### 14.3 Accepted Limitation

Degradation-only policy無法可靠辨識「model從第一次啟用起就持續很差」：初始bad reports可能形成baseline。
Current experiment接受此限制，因為seed model預期先可用，之後才刻意改變單一group pattern。未來initial model
quality gate應放在Model Provision/activation validation，不因這項限制重新加入chronic path。

被取代的舊選項僅保留為決策背景：temporary private extension違反standard-only boundary；Phase 5
data-backed enrichment會把monitoring decision不必要地耦合到dataset retrieval。兩者都不是target design。

---

## 15. Planned Code Changes

### 15.1 NWDAF

預計觸及：

- `internal/sbi/`
  - ML Model Provision與Monitor public routes/handlers
  - processor routing
  - provision/monitor external notification consumers
  - per-service authorization name
- current generated ML Model Provision models/client先做field coverage characterization；Monitor及missing R18欄位使用
  isolated compatibility wire package
- `internal/anlf/`
  - backend-initiated provision subscription與monitor registration routes
  - provision notification與monitor subscription backend client
  - replace custom model-event/accuracy callback routing
- `internal/mtlf/client/`
  - provision subscription CRUD、monitor registration CRUD與notification forwarding
- `internal/context/`
  - provision/registration/subscription route mirror
  - per-service NRF profile identity
- `internal/backend/contract.go`
  - typed sync snapshots
- `pkg/service/init.go`
  - sync projection及availability wake-up
- `pkg/factory/`與`config/nwdafcfg.yaml`
  - remove active Go accuracy policy ownership
  - advertise/configure ML Model Provision與Monitor capability

Current `internal/sbi/consumer/mtlf_service.go`與tests提供generated client、POST/DELETE及Location parsing的
characterization；final local flow由Go processor route到`mtlfBackend`，不得保留Go-owned per-analytics model
selection/correlation business logic。

### 15.2 PyAnLF

預計觸及：

- standard ML Model Provision wire model/client intent與notification router
- model-demand registry、compatible-model resolver、provision refcount/reconciliation
- existing `ModelManager`及`SubscriptionRuntimeManager`的小範圍final-flow integration；保留artifact與atomic
  activation behavior
- standard ML Model Monitor wire models/router/resource service
- model-use registration reconciliation/refcount
- accuracy monitor subscription bindings及notification sender
- separate ground-truth poll/report-window scheduler及window-local matched-pair accumulator
- runtime manager attach/detach integration
- sync projection/reconcile
- config：移除default local model作production ownership與custom callback URI；保留download safety、measurement
  defaults及Go internal base from sync
- README/API documentation
- existing accuracy tests plus resource/contract/restart tests

### 15.3 PyMTLF

預計新增／調整：

- standard Model Provision/Monitor wire models/router
- configured seed model catalog、artifact descriptor validation及existing repository lookup
- provision subscription resource store、immediate report與notification sender
- registration resource store
- registration-driven subscription reconciler
- standard notification consumer
- accuracy policy config/defaults
- WAPE-only degradation state/reference ring/N-in-M window/GC
- retrain intent/in-flight domain seam
- sync projection/reconcile
- preserved-mechanics fixtures、approved old/new behavior tests與concurrency tests
- README/API documentation

### 15.4 Expected Production Removals

- Go per-analytics direct MTLF subscription ownership、`ModelProvisionBinding`與custom `ModelProvisionEvent`
  forwarding wiring。
- PyAnLF `/subscriptions/{subscriptionId}/model-provision-binding`與`/model-provision-events` custom production
  routes；其正確download/load/runtime primitives保留並由standard notification呼叫。
- PyAnLF custom `ModelAccuracyReport` serialization/sender。
- Go `/model-accuracy-reports` handler/processor path。
- Go active accuracy policy invocation及Go config sample ownership。
- Custom accuracy report cross-language serialization tests，以standard contract round-trip替代。

---

## 16. Implementation Sequence

依development policy拆成可獨立驗證的vertical slices。依本次執行決策，各slice完成後只執行其focused
tests並確認工作樹仍可繼續，不在slice邊界commit、停下或要求review；Slice A至G全部完成後，才統一執行
cross-repository code review、完整測試、lint、build、文件對照與修正。所有修正完成後，再依repository邊界
分別commit。

這項執行決策只改變checkpoint與review時機，不放寬任何slice的contract、acceptance criteria或focused
verification。若中途遇到development policy定義的真正decision gate，仍須先停下取得決策。

### Slice A: Provision And Monitor Contract/Go Edge

1. Characterize current generated Model Provision coverage，建立missing Release 18及Monitor compatibility wire models。
2. 增加兩個service的public route、method/status/Location/ProblemDetails tests。
3. 增加Go backend client methods與raw payload preservation tests。
4. 增加resource mirror與NRF profile tests。
5. 此slice不切換current model/accuracy production path。

### Slice B: PyMTLF Initial Provision Producer

1. Seed catalog config與existing artifact repository validation。
2. Provision subscription POST/PUT/DELETE store及sync。
3. Immediate report與standard notification production。
4. Go provision callback destination routing。
5. Resource CRUD與callback process tests；尚不切換PyAnLF runtime owner。

### Slice C: PyAnLF Model Demand And Initial Activation

1. Analytics subscription建立model demand；admission不等待provision。
2. Equivalent intent coalescing、compatible loaded-model reuse及refcount。
3. Standard provision create/delete與notification ingestion。
4. Reuseexisting ModelManager/runtime prepare-commit path完成download/load/bind。
5. First subscription obtains seed；second compatible group不重複POST/download的process test。
6. Model尚未READY不得建立monitor registration。

### Slice D: Monitor Registration/Subscription Lifecycle

1. PyMTLF registration CRUD/store/sync。
2. PyAnLF subscription CRUD/store/sync。
3. Go monitor callback gateway與callback destination routing。
4. PyAnLF model-use registration reconciler。
5. PyMTLF local monitor subscription reconciler。
6. Same model/different group建立獨立scope；identical context只增加refcount。
7. Process-level provision→register→subscribe→notify→delete lifecycle。

### Slice E: Measurement Subscription Cutover

1. 將PyAnLF monitor activation綁定standard subscription。
2. 保留matching/confidence/scope fixtures，並為WAPE zero denominator與report-window refinement建立explicit
   old/new tests。
3. 分離30秒ground-truth poll與90秒report window，在window內累積matched pairs且不跨windowcarry。
4. 產生standard `MLModelMonitorNotify`：sufficient window以`deviation`帶WAPE，insufficient window省略它。
5. 移除custom sender/config，但尚不移除Go policy oracle code。

### Slice F: PyMTLF WAPE Degradation Policy

1. 從Go behavior matrix抽取本計畫保留的degradation mechanics，並把removed behavior標示為superseded。
2. 先以Python tests固定WAPE defaults、baseline/reference、z-score、N-in-M與multi-scope result。
3. 實作WAPE-only state store與single degradation path。
4. 補no-deviation skip、clock/concurrency/in-flight/generation seam tests。
5. 確認chronic、low-traffic、traffic-scale與multi-metric config/runtime沒有production path。

### Slice G: Ownership Cutover

1. Standard provision path成為initial model唯一production owner；移除Go custom provision coordination caller。
2. Standard monitor notification production path只進PyMTLF。
3. Go停止更新policy及dispatch legacy retrain。
4. 移除custom provision/accuracy route與contract production wiring。
5. 移除Go sample accuracy config ownership與PyAnLF default local model production ownership。
6. 保留必要historical oracle code或將unreachable residual標記Phase 7 cleanup。
7. 執行full cross-repo verification並更新completion record。

---

## 17. Failure And Concurrency Semantics

### 17.1 Backend Availability

- Provision subscription與Monitor registration operation只依賴MTLF backend；Monitor subscription operation只依賴
  AnLF backend。
- Required backend disabled時不advertise完整service；configured但unusable時回503。
- Transport failure立即MarkUnavailable並喚醒polling。
- MTLF unavailable不使Events Subscription admission失敗：沒有model的runtime保持pending，已有READY model的
  analytics繼續使用既有model。AnLF unavailable不刪PyMTLF provision/registration resource。
- Configured seed artifact在PyMTLF startup validation失敗時artifact capability不得READY；不能回傳指向不存在
  或未驗證bundle的URL。

### 17.2 Resource Mutation

- Concurrent PUT/DELETE對同一resource serializable。
- Stale replace不得在commit後覆蓋newer delete。
- Equivalent PyAnLF model demand及同一artifact concurrent notification以per-key lock/coalescing保證單次POST、
  download及load；不同monitoring context不因此被錯誤合併。
- Backend create成功但Go回應前中斷時可能留下resource；第一版不建立custom idempotency protocol，restart sync及
  resource cleanup提供best effort convergence。
- Go mirror更新與backend accepted response必須同一processor critical section，避免Location已回但callback route
  尚未存在。

### 17.3 Notification

- Go只在destination backend/external consumer接受Model Provision或Monitor notification後回204。
- PyAnLF provision callback在完成整批typed/correlation validation並atomically enqueue notification intent後
  回204；HTTP response不等待最長300秒artifact download。Download/load與runtime activation由bounded owned
  worker執行，失敗進retry state。
- Provision notification retry/replay不得對相同initial model增加generation或重複attach；delete/replace後的
  stale revision在download完成前再次檢查，不得commit已取消model。
- PyAnLF對transport、timeout、502/503等5xx有限retry；well-formed 4xx不重試。
- Go只有在destination已接受standard notification後回204。
- PyMTLF在回204前完成typed validation及atomic policy-state commit；policy計算很小，不增加drop queue。
- External destination failure不被Go改成成功。
- Delete/replace後stale notification worker在send前再次檢查resource revision。

### 17.4 Policy State

- Same model notification以per-model lock serialization。
- 不同model可並行。
- TTL GC與evaluation使用同一injected clock。
- Trigger claim與window reset atomic；兩個scope同時達標只能產生一個in-flight intent。
- Missing `deviation`只記錄liveness/observability，不更新baseline/reference/hit window，也不以0污染history。
- Group A/B等scope各自維護reference與window；任一scope觸發時以model-level lock一次claim intent並reset所有scope
  decision windows。

---

## 18. Verification Plan

### 18.1 NWDAF Focused Tests

- Model Provision/Monitor route method/path與各自authorization scope。
- 兩個service的201 body/Location、PUT 200/204及external/private URI separation。
- PUT 200/204 forwarding；DELETE 204/404。
- 400/413/415/502/503 mapping及ProblemDetails media type。
- Unknown Release 18 fields round-trip不丟失。
- Provision subscription與Monitor registration只route MTLF backend；Monitor subscription只route AnLF backend。
- Provision callback route internal AnLF versus external URI；Monitor callback route internal MTLF versus external URI。
- Backend unusable及live transport failure wake-up。
- Resource mirror/sync snapshot及backend restart。
- NRF profile two-service advertisement與disabled capability。

### 18.2 PyAnLF Tests

- First analytics subscription without model creates one provision intent and remains non-blocking pending。
- Valid seed notification downloads/verifies/loads bundle before monitor registration。
- Invalid URL/origin/digest/archive/load keeps initial runtime pending and never advertises monitoring capability。
- Concurrent equivalent demand and duplicate initial notification perform one POST/download/load。
- Compatible second group reuses model/provision resource but creates independent monitoring scope；PUT is allowed
  only when the active-demand union needs expansion。
- Exact same canonical context increments refcount and does not duplicate monitoring report。
- Last-scope/last-model reference independently drives monitor deregistration and provision DELETE。
- Registration refcount first-create/last-delete/coalescing/retry。
- Subscription POST/PUT/DELETE standard contract。
- Unsupported/missing model context rejection。
- `immReport` cached-only behavior。
- Subscription-driven attach/detach，Events admission不等待accuracy setup。
- Existing matching/confidence/scope/ring/concurrency tests維持；被本計畫明確替換的multi-metric output、WAPE
  zero-denominator及round-local discard tests改成old/new behavior tests。
- WAPE formula涵蓋normal、all-zero correct、all-zero overprediction及ratio greater than one。
- Ground-truth每30秒poll、90秒window內累積、matched threshold 2及window boundary discard。
- Sufficient standard notify帶WAPE `deviation`；insufficient notify省略`deviation`但仍含model/inference/window。
- `inferenceNum`是window內全部inference count，不是matched count。
- Standard notify alias/cardinality/context/monitor interval。
- Retry, stale worker, delete and shutdown。
- Sync replays provision before monitor，reuses valid cache or re-downloads，without duplicate registration/worker。

### 18.3 PyMTLF Tests

- Seed descriptor/artifact identity/applicability/startup validation。
- Provision subscription POST/PUT/DELETE、immediate report、async notification及sync restore。
- Unsupported/no-match request不得產生fake model URL或啟動training。
- Registration CRUD與auto-subscription intent。
- Registration delete與async downstream cleanup。
- Notification required/anyOf/cardinality validation。
- Canonical scope key由standard context建立。
- Preserved degradation-mechanics oracle與approved redesign boundary。
- Deterministic TTL、ring overwrite、baseline warm-up、fixed WAPE floor、population std/minStd、strict z-score、
  N-in-M及reference isolation。
- Same-model concurrency single trigger。
- Group A degraded/Group B healthy仍觸發single model-level intent，保存triggering/active scopes並reset both scope
  windows。
- Missing `deviation` does not synthesize zero or mutate policy windows；missing/false `accuMeetInd` without negotiated
  threshold does not trigger degradation。
- Production config/runtime has no primary-metric, traffic gate, chronic, low-traffic or consecutive-breach fallback。
- Retrain in-flight、reset、terminal及generation domain seams。
- Sync restart resets volatile baseline but restores resource/control intent。

### 18.4 Process Matrix

1. First Group A Events Subscription → PyAnLF pending demand → Go → PyMTLF provision subscription 201。
2. PyMTLF seed notify → Go → PyAnLF 204；PyAnLF direct artifact GET/load → READY。
3. PyAnLF model use → Go → PyMTLF monitor registration 201。
4. PyMTLF → Go → PyAnLF monitor subscription 201。
5. Group B Events Subscription reuses model without another provision POST/download，but creates Monitor B；optional
   PUT only updates the shared active-demand union。
6. PyAnLF Group A/Group B WAPE notify → Go → PyMTLF 204 with independent scope state；insufficient period仍送
   no-deviation notify且不污染state。
7. Group A WAPE degradation + Group B healthy → exactly one model-level retrain intent。
8. External provision/monitor subscription → Go → owner backend → Go callback → external consumer。
9. Registration delete → PyMTLF local 204 → async monitor subscription cleanup；last model demand deletes provision resource。
10. PyAnLF restart → Go sync → provision re-notify/cache restore → monitor resources restored in order。
11. PyMTLF restart → Go sync → seed/provision/registration intent restored, policy baseline empty。
12. Either backend unavailable → dependent standard operation 503, independent accepted Events resource remains usable。

### 18.5 Repository Commands

NWDAF：

```bash
make test
make lint
make build
go test -race ./internal/backend ./internal/context ./internal/anlf/... ./internal/mtlf/... ./internal/sbi/... ./pkg/service
```

PyAnLF：

```bash
uv run ruff check .
uv run pytest -q
```

PyMTLF：

```bash
uv run ruff check .
uv run pytest -q
```

Real NRF/peer NWDAF/Mongo/ADRF process checks另外列為integration；mock與`TestClient`通過不得宣稱full 5GC E2E。

---

## 19. Acceptance Criteria

Phase 4只有在以下全部成立時可標記completed：

1. D1已依section 14完成：standard `deviation = WAPE`、single degradation path，Phase 5沒有policy enrichment責任。
2. Public Model Provision subscription與Monitor registration/subscription method、path、body、status、callback及
   Location符合Release 18 OpenAPI。
3. Go/backend private operation使用相同standard body與method語意，沒有business wrapper。
4. Go只route/validate/mirror，不選model、不下載artifact、不計算accuracy或degradation。
5. PyMTLF擁有configured seed catalog、provision/registration resource與policy state。
6. PyAnLF擁有model demand/download/load/runtime binding、monitor subscription與measurement state。
7. Local Events demand→initial provision→download/load→register→subscribe→notify chain可在三個process間運作。
8. External subscription callback可經Go送達original notification URI。
9. Backend unavailable回OpenAPI允許的503 ProblemDetails。
10. Unknown R18 fields不被Go proxy silent drop。
11. PyAnLF所有既有artifact/model lifecycle與被保留的measurement parity tests保持通過；approved WAPE與
    report-window changes有explicit old/new tests。
12. PyMTLF fixtures覆蓋被保留的Go degradation mechanics，以及removed behavior不再存在的boundary tests。
13. Python policy的population std、strict comparison、reference、N-in-M、reset、GC與in-flight result對保留範圍
    與Go oracle等價。
14. WAPE依section 7.2計算，sufficient notify帶`deviation`；insufficient notify省略它且不更新policy。
15. Group A與Group B共用同一model但具有獨立monitor windows；A單獨degraded可產生一次model-level intent，
    intent保留A為triggering scope及A/B active scope inventory。
16. Compatible second subscription不重複provision/download；same context不duplicate monitor。
17. Custom model provision event/binding與`/model-accuracy-reports`不再有production caller或route。
18. Go accuracy policy不再有production caller或active config ownership。
19. Provision/registration/subscription mirror可在backend process restart後依序sync restore。
20. Go restart loss、Phase 5 dataset retrieval及Phase 6 training/updated reprovision明確標示未完成。
21. Full repository lint/test/build與planned race tests通過。
22. 實際未執行的NRF/ADRF/Mongo/peer NWDAF integration被精確記錄，不以unit test代替。

---

## 20. Risks And Controls

### 20.1 WAPE Local Semantics May Be Misread

`deviation`沒有在3GPP中指定為WAPE。Control是在wire/API docs、config與tests明確宣告ratio semantics、zero
denominator及higher-is-worse；不填`mlModelAcc`，也不以`accuMeetInd`替代local degradation decision。

### 20.2 Duplicate POST After Ambiguous Timeout

Standard POST沒有本地custom idempotency key。Control是coalesced desired state、Go mirror、Location correlation、
restart sync及observability；不宣稱exactly-once。

### 20.3 Model Compatibility False Reuse

只以Analytics ID重用model可能把group/AoI/filter不適用的artifact綁到runtime。Control是canonical demand、returned
applicability/validity及runtime interoperability coverage check；無法證明compatible時重新建立provision intent，
不得猜測。

### 20.4 Provision Callback Versus Slow Download

Artifact download timeout預設300秒，不得讓standard callback request同步等待。Control是callback先validate並
commit desired revision後回204，owned worker下載；crash/restart由PyMTLF current-seed reconciliation re-notify。

### 20.5 Resource Loop Deadlock

Registration response不等待subscription create；delete response不等待downstream cleanup。兩邊worker只持有
local intent lock，不跨HTTP call持有resource store lock。

### 20.6 Callback URI Leakage

Go分開保存external representation與backend representation；external response/sync不得洩漏private callback base。

### 20.7 Restart Baseline Loss

Policy state只在PyMTLF memory。Restart後registration恢復但baseline重建；這符合已接受的實驗重跑／volatile
state範圍，不加入Go persistence。

### 20.8 Scope Collision

Standard沒有private scope ID。PyMTLF canonical key包含MTLF provider namespace、model ID、AnLF consumer ID、event、filter與
target的canonical JSON；subscription ID只作contributor correlation，不能只用model ID、subscription ID或
target string形成scope。

### 20.9 Partial Service Advertisement

`nnwdaf-mlmodelprovision`在MTLF backend configured且完整producer surface存在時advertise；
`nnwdaf-mlmodelmonitor`需AnLF/MTLF兩邊完整resource surface。Runtime readiness變化不改profile，避免NRF看到
只有registration或只有subscription的半套Monitor service。

### 20.10 Legacy Code Appears To Retain Ownership

Cutover review以production call graph與config ownership為準。為historical oracle暫留的unreachable Go code必須
列入Phase 7 legacy cleanup，不能由startup或handler重新接回。

### 20.11 Degraded From Initial Activation

Degradation-only baseline可能把從第一個window起就很差的seed model視為reference。Current experiment接受此
限制；Control是PyMTLF startup artifact validation與future Model Provision/activation quality gate，不能在本phase
暗中恢復chronic policy。

---

## 21. Decision Gates

### Required Before Implementation

- 無；D1、WAPE zero-denominator、report-window sufficiency與initial provision boundary均已確認。

### Stop During Implementation Only If

1. Scoped compatibility model無法在不使用untyped business map的情況下保留mandatory R18 fields。
2. 被標示為preserve的Go degradation mechanics對同一fixture產生不可解衝突。
3. 除本計畫已核准的WAPE/report-window變更外，PyAnLF matching或confidence semantics必須改變才可產生standard
   notification。
4. Go需要保存policy state或Python backends必須直連才可完成resource lifecycle。
5. Required change會提前改變Phase 5 dataset ownership或Phase 6 training/new-generation reprovision boundary。
6. Repository tooling或dependency無法執行required verification。

普通package placement、class naming、lock implementation及test seam若不改變上述contract與ownership，不是
decision gate，依本計畫與development policy直接完成。

---

## 22. Implementation Baseline Record

### 22.1 Implemented Ownership

- Go新增Release 18 Model Provision/Monitor public SBI與backend gateway，保存external/backend representation、
  callback correlation及resource ID，並將三種ML model resource納入continuous reconnect sync。
- PyMTLF以configured seed catalog及immutable artifact repository提供initial model，擁有provision subscription、
  monitor registration、monitor-subscription intent及WAPE degradation state。
- PyAnLF以analytics demand建立／共用provision resource，沿用既有artifact驗證與atomic runtime activation，
  以PUT收斂未被目前model涵蓋的active-demand union，只在READY後register model use，並擁有monitor
  subscription與accuracy measurement。Provision callback完成整批validation/enqueue後即回204，artifact
  download/load由owned worker執行。
- Backend restart時保留sync恢復的monitor registration，待model runtime恢復後再authoritatively reconcile；
  monitor subscription binding也會在runtime READY後重算。相同canonical scope只選一個穩定代表runtime產生
  measurement，避免duplicate consumer subscription污染WAPE samples。
- Production accuracy path使用30秒ground-truth poll、90秒window與minimum two matched pairs；sufficient window
  以`deviation`回報WAPE，insufficient window省略`deviation`與`modelMetric`。
- `monitorInterval`依TS 29.520引用的`TimeWindow`使用`startTime`與`stopTime`。
- Go sample config不再啟動legacy MTLF policy/training owner；custom provision binding/event、
  `/mlmodel-notify`及`/model-accuracy-reports`不再註冊production route。

### 22.2 Verification Evidence

2026-07-23 local verification：

- NWDAF：`go test ./...`通過；`make lint`為0 issues；`make build`通過。
- NWDAF planned race set：`internal/backend`、`internal/context`、`internal/anlf/...`、
  `internal/mtlf/...`、`internal/sbi/...`及`pkg/service`全部通過。
- PyAnLF：223 passed、1 skipped；`uv run ruff check .`通過。
- PyMTLF：48 passed；`uv run ruff check .`通過。另有FastAPI TestClient對`httpx2`的upstream deprecation
  warning，不影響測試結果。
- Contract tests覆蓋standard method/status/Location、raw Release 18 field preservation、callback URI separation、
  backend unavailable mapping、initial model reuse/union expansion、non-blocking callback enqueue、
  monitor notification routing、restart registration/binding convergence、same-scope coalescing、WAPE windows、
  insufficient liveness report、multi-scope degradation及single in-flight intent。

### 22.3 Remaining Boundaries

- 未執行real NRF/SMF/UPF/ADRF/Mongo及三個實際process共同啟動的5GC integration；不得把上述local
  contract/unit/process-fixture tests宣稱為live 5GC E2E。
- Dataset source selection與ADRF fetch/Mongo direct retrieval由後續dataset工作完成。
- Local training、new generation artifact publication及updated/re-trained model reprovision由後續training工作完成。
- Unreachable historical Go MTLF/Daisy與custom handler implementation留待legacy cleanup；本次已以route、
  startup wiring及sample config確保其不再擁有production behavior。

### 22.4 Review Gate Closure

上述紀錄保存initial implementation及當時local verification的事實。2026-07-23完整slice review確認六項
current-slice defect及一項required integration evidence gap，統一由
`code-reviews/Initial Model Provision And Monitoring Review Ledger.md`追蹤：

- unified sync不得接受partial state；
- restart後的orphan monitor resource必須被隔離並清理；
- compatible-model reuse必須使用active catalog，而不是單一latest pointer；
- monitor resource必須落實per-resource cadence、multiple model、optional context及cached immediate report；
- provider-generated response field不得由consumer request注入；
- Go error/redirect forwarding必須使用operation-specific OpenAPI contract；
- actual Go、PyAnLF及PyMTLF process-level round-trip必須通過。

2026-07-24以上ledger gate已全部關閉。Final local evidence為：

- NWDAF full test、lint、build及selected race tests通過。
- PyAnLF 232 passed、1 skipped，ruff通過。
- PyMTLF 55 passed，ruff通過。
- actual Go、PyAnLF與PyMTLF三程序harness連續兩次通過，包含PyMTLF unavailable→polling/sync recovery、
  artifact首次失敗後原子重試、WAPE round-trip，以及registration delete/downstream cleanup間restart後的
  orphan cleanup。
- 跨repository harness已移至`nwdaf-resources/tests/mtlf_model_monitor/`；搬移並修正fake SMF resource
  identity後，從獨立Go module連續執行三次完整scenario通過，三個runtime repositories只保留各自
  unit／contract tests。

整合過程另修正兩個cross-process才會顯現的契約問題：Go空association list不得序列化為`null`，以及
PyAnLF動態載入`model.py`不得在verified artifact cache產生`__pycache__`而破壞restart integrity。

本phase只宣告local initial-model/monitor slice完成；real NRF/SMF/UPF/ADRF/Mongo E2E、Phase 5 dataset及
Phase 6 training/new generation仍不在本completion claim內。Implementation已依repository boundary提交；
exact commit記錄於current review ledger。
