# Slice 1 — Wire Contract and Resource Lifecycle Foundation Detailed Plan

日期：2026-09-03

狀態：Committed／production implementation、mandatory initial review、計畫要求驗證
與 `NWDAF`／`PyMTLF` 收尾 commits 已完成

相關文件：

- [Protocol Extension Implementation Plan](../Hierarchical%20NWDAF%20FL%20Protocol%20Extension%20Implementation%20Plan.md)
- [Protocol Extension Implementation Slice Map](../Protocol%20Extension%20Implementation%20Slice%20Map.md)
- [Protocol Implementation Current-State Inventory](../Protocol%20Implementation%20Current-State%20Inventory.md)
- [Protocol Resource Lifecycle and Wire Integration Mapping](../Protocol%20Resource%20Lifecycle%20and%20Wire%20Integration%20Mapping.md)
- [Protocol Conformance Case Ownership Mapping](../Protocol%20Conformance%20Case%20Ownership%20Mapping.md)
- [Candidate OpenAPI Schema](../../../../design/hierarchical-federated-learning/candidate_openapi_schema.md)
- [Candidate OpenAPI artifact](../../../../design/hierarchical-federated-learning/candidate_openapi.yaml)
- [Protocol Conformance Matrix](../../../../design/hierarchical-federated-learning/protocol_conformance_matrix.md)
- [Protocol Extension Implementation Review Ledger](../Protocol%20Extension%20Implementation%20Review%20Ledger.md)

---

## 1. Slice 結果

本 slice 完成後，`NWDAF` 與 `PyMTLF` 應具有同一份 typed candidate message
contract，並能在既有 Create／PUT／PATCH／DELETE／Notify lifecycle 中：

- 解析與重新編碼 `x-flTopology`、`x-flTopologyReport`、
  `x-retainedResultReq` 與 `x-retainedResultStatus`；
- 保留 persistent topology contract，抽離一次性 retained-result instruction；
- 驗證 recursive identity、cross-field rule、receiver／callback identity 與
  `mlCorreId` correlation；
- 在 local／peer route，以及 destination 回 `200`／`204` 的情況下，維持相同的
  effective persistent representation；
- 保存每個 subscription resource 的 offered／negotiated feature state，並以該 state
  gate 後續 candidate operations；
- 對 schema／message 錯誤回傳含 extension path 的 `400 ProblemDetails`，對合法但
  receiver 無法履行的 contract 回傳 `403 ML_MODEL_TRAINING_REQS_NOT_MET`。

本 slice 不執行 candidate selection、policy、strategy、topology dispatch 或 retained
result lookup，也不宣稱 protocol-driven HFL 已可執行。

---

## 2. 基準與直接證據

### 2.1 盤點版本

| 儲存庫 | 版本 | Slice 1 角色 |
| --- | --- | --- |
| `NWDAF/` | `6aed268d6528f8be6c729cbd45b59d067e5e80dc` | External／peer SBI、Go→PyMTLF gateway、route state 與 primary receiver validation |
| `PyMTLF/` | `747962971b63f0a53031d52a1eb7e047ae776998` | Private backend typed mirror、subscription representation 與 execution-entry gate |
| `nwdaf-docs/` | `2964021c6858d1c6e55269898326082cd71177ff` 加上本次 unstaged 文件 | Canonical plan、candidate contract 與 conformance evidence |

開始 production implementation 前應再次確認上述兩個 production repositories 的
HEAD 與 clean working tree；若 revision 或 owner 已改變，先更新本計畫的 exact-file
mapping。

### 2.2 現有 production 基準

- `NWDAF/internal/compat/mlmodeltraining` 已提供 Release 18 compatibility models、
  parse、FL requirements validation、PATCH application 與 Notify correlation。
- `NWDAF/internal/sbi/processor/ml_model_training.go` 已處理 local／remote
  Create、PUT、PATCH、DELETE、Notify、callback URI rewrite、route mutation revision
  與失敗 rollback。
- `NWDAF/internal/context/ml_model_training_routes.go` 已保存 accepted／backend
  representation、peer target、correlation 與 expected `roundInd`。
- `PyMTLF/src/py_mtlf/wire/ml_model_training.py` 已提供 standard-shaped Pydantic
  models；top-level model 目前以 `extra="allow"` 保存未知 Release fields。
- `PyMTLF/src/py_mtlf/core/fl_client.py` 已擁有 subscription resource、replace／patch
  rollback、operation dispatch、generation reset 與 DELETE cleanup。
- `PyMTLF/src/py_mtlf/api/ml_model_training.py` 已將 requirements failure 映射成
  `ProblemDetails`；`app.py` 已將 standard-shaped private API 的 Pydantic request
  validation 映射為 `400`。

現有 Go typed reconstruction 會遺失未知 `x-*` properties；PyMTLF 雖可保存未知
properties，卻不會驗證或執行其語意。這兩項是本 slice 的直接起點。

### 2.3 標準與 candidate contract 證據

- Release 18 OpenAPI `TS29520_Nnwdaf_MLModelTraining.yaml` 定義 POST `201`＋
  `Location`、PUT／PATCH `200` 或 `204`、PATCH media type
  `application/merge-patch+json`，以及 Notify `204`。
- Release 18 `NwdafMLModelTrainSubsc` 已含 `suppFeats`、`mlCorreId`、
  `mLPreFlag` 與 `roundInd`；candidate extension 不重建這些欄位。
- TS 29.520 Release 18 §5.5.8 將 ML Model Training optional features交由
  TS 29.500 §6.6 extensibility mechanism 協商；candidate 使用 feature 3
  `HierarchicalFLOrch`，bitmask 為 `"4"`。
- Candidate OpenAPI 將 topology／report nested objects 設為
  `additionalProperties: false`；message top level仍保留既有 Release model 的
  forward compatibility。
- Candidate Stage 3 rules將 persistent topology與一次性 retained-result command
  分開，並要求 feature 3 在每一條 parent-to-child resource edge獨立協商。

### 2.4 free5GC 實作結構

本計畫以 target `NWDAF` 現有 handler→processor→context／consumer flow作主要
baseline，並對照 local free5GC snapshot：

- BSF subscription CRUD：確認 request parsing、resource ownership、`Location` 與
  `ProblemDetails` 留在 SBI／processor boundary；
- PCF callback route：確認 callback 的 HTTP parsing／validation 與 procedure owner
  分離。

這些 exemplar只支持 package responsibility與 HTTP boundary；candidate schema、
atomicity與 Go→PyMTLF contract仍以 target repository、Release 18 OpenAPI和本計畫為
準，不從 exemplar推論 protocol semantics。

---

## 3. Slice 邊界

### 3.1 納入

- Go／Python typed candidate fields與 JSON aliases。
- Nested candidate object closed-property validation。
- Recursive topology／report validation與 implementation safety bounds。
- Create／PUT／PATCH／Notify cross-field validation。
- Receiver identity、direct participant callback identity與 correlation validation。
- Persistent contract與 operation-scoped command分離。
- Go route offered／negotiated feature state與後續 operation gate。
- PyMTLF feature-disabled execution-entry behavior。
- Local／peer、`200`／`204` representation與 rollback tests。
- 不含 candidate fields的既有 ML Model Training regression。

### 3.2 不納入

- Candidate pool、NRF list-discovery、policy threshold或 participant selection。
- `strategy`／`reportAfter`的實際 trainer／aggregator binding。
- Topology status producer、recursive realized view或 downstream subscription dispatch。
- Latest-completed result index、lookup outcome producer或 outstanding lookup lifecycle。
- Feature mismatch後的 sender-side DELETE與 `FEATURE_NOT_SUPPORTED` topology report。
- Model-free preparation、ADRF global-model distribution或 Branch aggregate delivery。
- Legacy model-bundle移除與 real-process E2E。

上述項目分別由 Slice 2、Slice 3、Slice 4與 Slice 5負責；本 slice不得為了讓
candidate request「看起來能跑」而先接入舊 bundle execution。

---

## 4. 固定實作契約

### 4.1 型別整合

Go 繼續延伸 `internal/compat/mlmodeltraining`，不修改 pinned free5GC generated
models，也不建立新的 generated OpenAPI module。Candidate OpenAPI artifact是 JSON
contract與 fixture來源，不是 production runtime dependency。

Go nested candidate types放在既有 package的新檔案，subscription／patch／Notify
top-level structs只增加下列 typed properties：

| 訊息 | 欄位 |
| --- | --- |
| `NwdafMLModelTrainSubsc` | `x-flTopology`、`x-retainedResultReq` |
| `NwdafMLModelTrainSubscPatch` | `x-flTopology`、`x-retainedResultReq` |
| `NwdafMLModelTrainNotif` | `x-flTopologyReport`、`x-retainedResultStatus` |

PyMTLF 在現有 `wire/ml_model_training.py`加入對應 typed Pydantic models；不再依賴
unknown-property passthrough承載這四個已知 candidate properties。Top-level
standard-shaped models仍維持 `extra="allow"`；candidate nested objects使用
`extra="forbid"`，對應 OpenAPI `additionalProperties: false`。

`FlSelectionMethod`、`FlAggregation`、`FlReportUnit`、status、cause與 retained
outcome接受未知 string供 forward compatibility；`FlStrategy.method`維持 closed
discriminator，本版本只接受 `fedProx`，且要求 `aggregation`與
`methodParameters.proximalMu`。

### 4.2 遞迴防護上限

本 implementation使用相同的 local safety bounds，避免合法 JSON造成無界 recursive
walk；這些 bounds不是新的 wire field：

- topology／report root計為 depth 1；最大 depth為 16；
- 單一 instruction或report subtree最多 1024個 nodes；
- depth或node count超限使用 `400 INVALID_MSG_FORMAT`，`invalidParams`指向
  `x-flTopology`或`x-flTopologyReport`。

Go與PyMTLF各自以 named constants實作並使用相同 table tests。Body size仍由現有 Go
transport limit處理，不在本 slice改變。

### 4.3 Persistent 與 operation-scoped representation

Persistent properties：

- `x-flTopology`；
- node中的 `enabled`、`priority`、`policy`、`strategy`、`reportAfter`與
  `children`；
- negotiated `suppFeats` resource view。

Operation-scoped properties：

- top-level `x-retainedResultReq`；
- 任一 topology node的 `retainedResultReq`。

Operation-scoped properties必須在當次 request validation後形成 transient operation
descriptor，並在進入 destination execution entry前可見；它們不得出現在：

- destination accepted persistent representation；
- Go route accepted／backend representation；
- 後續 GET-equivalent resource view、PUT base或PATCH base；
- PyMTLF `FLClientResource.representation`。

若 destination成功 response錯誤地回傳 write-only instruction，Go將其視為 invalid
success representation，Create執行既有 compensation，PUT／PATCH執行既有 rollback，
不得自行把錯誤 response存入 route。

### 4.4 驗證責任與錯誤映射

Go是 external／peer SBI的primary receiver validator：

- JSON shape、required／range、closed nested object與closed strategy discriminator；
- duplicate identity、ancestor repetition、depth與node count；
- `mlCorreId`、policy numeric relation、status／cause、retained outcome與既有 Notify
  mutual exclusion；
- request receiver identity與callback-bound direct participant identity；
- negotiated feature gate與既有 resource identity。

PyMTLF mirror所有在private backend入口仍需成立的rules，並負責判斷本地 execution
capability。Go不得判斷 policy或strategy能不能執行；PyMTLF不得用較寬鬆的 validation
接受 Go會拒絕的 candidate message。

錯誤分類固定如下：

| 類型 | HTTP 結果 | 說明 |
| --- | --- | --- |
| JSON／schema／cross-field／identity錯誤 | `400 Bad Request`＋`INVALID_MSG_FORMAT` | `invalidParams`使用JSON alias path，例如 `x-flTopology.children[0].priority` |
| 合法 forward-compatible instruction但本地沒有executor | `403 Forbidden`＋`ML_MODEL_TRAINING_REQS_NOT_MET` | 不改寫成已知值或local default |
| Destination成功response違反candidate contract | `502 Bad Gateway` | Create compensation或mutation rollback後不保存 |
| Resource未協商feature 3卻收到後續candidate operation | `403 Forbidden`＋`ML_MODEL_TRAINING_REQS_NOT_MET` | request不得進destination或修改resource |

Go public handler、Go private gateway與processor必須共用同一個 structured candidate
error→`ProblemDetails` mapping，避免handler pre-parse先把 path information丟失。
PyMTLF的 FastAPI `RequestValidationError` mapping則需將 Pydantic alias location轉為
同樣格式的 `invalidParams`，不能只回 generic `400`。

### 4.5 接收者與 callback identity 來源

Request端 `x-flTopology.nfInstanceId` 的 authoritative expected identity：

| 方向 | 產生者／來源 | 驗證點 |
| --- | --- | --- |
| External／peer→local Go | `NWDAFContext.NfId` | local Go processor；PyMTLF再以 `NwdafContextClient.get().nf_instance_id` mirror |
| Root PyMTLF→remote peer | private `SelectedTarget.NFInstanceID` | sender Go在送出前驗證；receiver Go再以自己的 `NWDAFContext.NfId`驗證 |
| PUT／PATCH existing route | Create時綁定的destination NF identity | Go從route state取得，不重新以request內容推論 |

Create成功時，Go route保存 `BoundParticipantNFInstanceID`。對 outbound peer route取自
`SelectedTarget.NFInstanceID`；對 local destination取自 containing
`NWDAFContext.NfId`。Notify中的 `x-flTopologyReport.nfInstanceId`必須等於這個綁定
identity，不從report本身或當下重新discovery得到 expected value。

`mlCorreId`與`notifCorreId`沿用既有 route identity；Slice 1不新增 hierarchy process
identifier。

### 4.6 Feature 協商與 feature-disabled 行為

Go route新增下列 immutable resource-level facts：

- Create request的 offered `suppFeats`；
- Create成功response的 negotiated `suppFeats`；
- feature 3是否 negotiated；
- bound participant NF identity。

Destination response宣告的 bits不得超出request offered bits。Create完成後，後續
PUT／PATCH request或response不得重新啟用一個Create時未協商的feature；operation gate
只使用route保存的 negotiated state，不從當次body重新推論。

Slice 1期間，PyMTLF雖能解析與保存candidate persistent fields，但其production
supported-feature mask仍不包含feature 3，因為topology／policy／retained-result
execution consumer尚未完成。Initial Create仍可依TS 29.500 extensibility semantics
攜帶 `suppFeats: "4"`與candidate fields，並完成shape／identity validation；accepted
representation不宣告feature 3，candidate operation不得啟動：

- PyMTLF resource保留已驗證的persistent contract，抽離write-only instructions；
- resource進入既有idle `READY`狀態，不啟動preparation、training或lookup；
- 本次取得的work／callback capacity立即依既有idle path釋放；
- response中的 `suppFeats`為local supported mask與offered mask的intersection；
- sender若把hierarchy視為必要條件，應刪除該resource；該cleanup與status production在
  Slice 4實作。

這個狀態不表示receiver已支援hierarchy。Lossless representation只用於建立wire／
resource foundation；`suppFeats`才是procedure applicability的authority。Slice 2／3
完成local consumers後，仍不得單獨打開feature 3；直到Slice 4串接真實producer、
consumer與cleanup flow後才啟用production advertised bit。

不含candidate fields的既有Release 18／legacy requests完全沿用現有operation dispatch，
不受此gate影響。

### 4.7 Create／PUT／PATCH atomicity

- Create reservation、destination `201`、`Location`、success representation與
  compensation沿用既有flow。
- PUT是完整representation replacement；PUT省略 `x-flTopology`代表新persistent
  representation不含topology。
- PATCH沿用JSON Merge Patch：省略 `x-flTopology`保留舊值；提供object時依object
  merge；提供某node的`children` array時完整取代該array。
- Go先從accepted persistent representation計算effective state；operation-scoped
  fields不進入base或effective persistent result。
- Destination `200`以其合法representation為authority；`204`使用Go／PyMTLF各自
  計算的同一effective representation。兩條路徑必須產生相同persistent semantics。
- Destination error、malformed success、stale operation revision或backend generation
  reset均不得留下部分updated representation或feature state。
- Accepted與backend representation只有callback URI可不同；candidate persistent
  fields、negotiated features與operation stripping結果必須相同。

### 4.8 DELETE與restart

本 slice沿用既有DELETE route serialization、destination `204`／`404`收斂、PyMTLF
resource cleanup與tombstone behavior。它只確保新增的persistent candidate contract
與feature state隨route／resource刪除，不新增downstream child cleanup。

Go backend generation reset繼續使in-memory routes失效；PyMTLF
`abort_generation()`繼續清除resources。Slice 1不新增persistence或restart recovery，
也不得從stale request重建negotiated feature state。

---

## 5. 基準階段處置

| 基準階段 | 處置 | Slice 1處理 |
| --- | --- | --- |
| Trigger／instruction production | 核准延後 | Root coordinator與protocol-mode producer留給Slice 4 |
| Public／private body read與media type | 沿用且不改變語意 | 保留size／media-type gate；只增加structured candidate error mapping |
| Subscription／patch／Notify typed parse | 調整 | 加入candidate types、aliases與validation |
| Callback URI rewrite | 沿用且不改變語意 | 仍以raw JSON object只改 `notifUri`，candidate fields不得遺失 |
| Local backend／peer transport | 沿用且不改變語意 | 仍傳raw body；新增lossless candidate boundary tests |
| Destination resource admission | 調整 | 保存persistent contract、抽離commands、feature-disabled時不dispatch |
| Create success／Location | 沿用並延伸representation | HTTP semantics不變；accepted body與route新增candidate／feature state |
| PUT／PATCH mutation | 調整 | persistent／operation分離、feature gate、`200`／`204`一致與atomic rollback |
| Normal training execution | 沿用legacy／non-candidate flow | Candidate execution明確deferred，不接入舊bundle path |
| Notify parse／route correlation | 調整 | candidate detailed information、identity與retained cross-field validation |
| Notify procedure consumption | 核准延後 | topology view與retained outcome consumer留給Slice 2／3／4 |
| DELETE | 沿用並清理新增狀態 | 清除新增route／resource fields；不做child cascade |
| Timeout／retry | 沿用既有operations | 不新增lookup timeout或callback procedure |
| Restart／generation reset | 沿用但不提供恢復 | 新增state與resource一起失效，不從body推論恢復 |
| Migration cleanup | 核准延後 | Legacy bundle removal留給Slice 5 |

---

## 6. 方向性資料流

### 6.1 Create request與accepted response

```text
FL Server／external consumer
  -> containing Go handler／private gateway
  -> Go typed parse + receiver validation
  -> callback URI rewrite
  -> local PyMTLF or peer NWDAF
  -> PyMTLF typed mirror + persistent/operation split
  -> accepted representation + negotiated suppFeats
  -> Go validates success representation
  -> Go route stores accepted/backend views + bound identity + feature state
  -> 201 + Location + accepted representation
```

Authoritative owners：

- Request instruction：caller；
- Expected receiver identity：sender selected target與receiver containing NWDAF context；
- Persistent contract：recipient PyMTLF resource，以及containing Go route的wire views；
- Offered／negotiated features：Create request／destination response，Go route保存resolved
  state；
- Operation descriptor：recipient PyMTLF當次call stack，離開operation後不保存。

### 6.2 PUT／PATCH

```text
caller operation
  -> Go loads route identity + negotiated feature
  -> validate body/patch and candidate gate
  -> build persistent effective representation + transient operation descriptor
  -> destination mutation
  -> validate 200 representation or accept 204 effective state
  -> atomically replace Go/PyMTLF persistent state
```

任何前置validation、destination或success-response失敗都保留原route與resource。

### 6.3 Notify

```text
direct child callback producer
  -> parent NWDAF public callback endpoint
  -> parent containing Go route lookup
  -> typed shape + correlation + bound participant validation
  -> raw candidate body relay
  -> parent PyMTLF typed mirror
```

Slice 1只保證合法body到達parent PyMTLF入口；realized topology或retained outcome不在
本slice套入coordinator state。

### 6.4 DELETE／reset

DELETE與generation reset沿用既有owner清除Go route與PyMTLF resource。本slice沒有
其他durable source可重建offered／negotiated features或operation command，因此reset後
舊callback與mutation維持fenced。

---

## 7. 確切檔案計畫

### 7.1 `NWDAF/` production 檔案

| 檔案 | 預計變更 |
| --- | --- |
| `internal/compat/mlmodeltraining/models.go` | 將四個top-level candidate properties接入現有subscription／patch／Notify structs，延伸resource identity input |
| `internal/compat/mlmodeltraining/candidate.go`（new） | 定義recursive topology／report、policy、strategy、report-after、status／cause與retained outcome types及feature constants |
| `internal/compat/mlmodeltraining/validation.go` | 接入candidate parse／cross-field validation、operation stripping與PATCH effective semantics |
| `internal/compat/mlmodeltraining/candidate_validation.go`（new） | 實作recursive walk、closed nested properties、identity uniqueness、limits、status／cause與structured path errors |
| `internal/context/ml_model_training_routes.go` | 保存／clone offered features、negotiated features與bound participant identity |
| `internal/sbi/api_ml_model.go` | 為training-specific pre-parser提供可注入的structured error mapper，不改其他ML Model API結果 |
| `internal/sbi/api_ml_model_training.go` | Subscription／PUT／PATCH handler使用candidate-aware error mapping |
| `internal/sbi/api_ml_model_callback.go` | Training callback pre-parser保留candidate invalid path |
| `internal/mtlf/api_ml_model_gateway.go` | Go private gateway對training request／Notify使用同一candidate error mapping |
| `internal/sbi/processor/ml_model_training.go` | Receiver identity、feature state、operation gate、success-response validation與persistent representation更新 |

不預期修改route registration、factory、config或新增Go package。若implementation需要
新的package而非上述existing owners，必須先通過development policy的New Go Package
Gate並更新本計畫。

`internal/mtlf/client/ml_model.go`與
`internal/sbi/consumer/ml_model_peer_service.go`現況已傳遞raw body；預期只新增tests，
除非characterization test證明training operation仍有lossy rewrite。

### 7.2 `NWDAF/` 測試檔案

| 檔案 | 預期證據 |
| --- | --- |
| `internal/compat/mlmodeltraining/models_test.go` | Candidate JSON alias與exact round trip |
| `internal/compat/mlmodeltraining/validation_test.go` | REQ／TOP／NOT／RET table-driven validation與PATCH semantics |
| `internal/compat/mlmodeltraining/candidate_validation_test.go`（new） | Recursive duplicates／cycle、limits、closed properties、forward-compatible enums與operation stripping |
| `internal/context/ml_model_training_routes_test.go` | Feature／bound identity clone與mutation isolation |
| `internal/sbi/api_ml_model_training_test.go`（new） | Public Create／PUT／PATCH structured `400 ProblemDetails.invalidParams`與media type regression |
| `internal/sbi/api_ml_model_callback_test.go`（new） | Topology／retained Notify pre-parser error path與valid relay entry |
| `internal/mtlf/api_ml_model_gateway_test.go` | Private gateway structured errors與raw candidate preservation |
| `internal/sbi/processor/ml_model_training_test.go` | Local／peer Create、PUT／PATCH `200`／`204`、feature gate、identity、atomic rollback與Notify validation |
| `internal/mtlf/client/ml_model_test.go` | Training CRUD／Notify raw candidate body與ProblemDetails preservation |
| `internal/sbi/consumer/ml_model_peer_service_test.go` | Peer training CRUD body、private-header isolation與candidate response preservation |

### 7.3 `PyMTLF/` production 檔案

| 檔案 | 預計變更 |
| --- | --- |
| `src/py_mtlf/wire/ml_model_training.py` | Candidate typed models、aliases、nested closed properties、recursive／cross-field validation、transient operation extraction |
| `src/py_mtlf/wire/features.py` | 加入 `HierarchicalFLOrch` feature 3 constant；重用既有hex mask／intersection helpers |
| `src/py_mtlf/core/fl_client.py` | Persistent／operation split、feature-disabled `READY` gate、atomic replace／patch與generation cleanup |
| `src/py_mtlf/api/ml_model_training.py` | 建立accepted negotiated representation，區分 `400` contract error與 `403` capability error |
| `src/py_mtlf/api/problems.py` | 將Pydantic alias location轉成candidate `invalidParams`格式的helper |
| `src/py_mtlf/app.py` | Standard-shaped private route的RequestValidationError使用上述path mapping |

本slice不修改`fl_server.py`、`fl_branch.py`、`fl_root.py`、trainer、aggregator、NRF
resolver或artifact workspace。

### 7.4 `PyMTLF/` 測試檔案

| 檔案 | 預期證據 |
| --- | --- |
| `tests/test_ml_model_training_wire.py` | Typed fixtures、recursive／cross-field mirror、operation extraction與exact JSON names |
| `tests/test_ml_model_training_api.py` | `400`／`403` ProblemDetails、accepted feature intersection、Location與candidate response |
| `tests/test_fl_client.py` | Persistent／transient separation、feature-disabled `READY`、capacity release、PUT／PATCH atomicity與reset／DELETE cleanup |

若`tests/test_fl_client.py`新增內容使單一test file不易review，可新增
`tests/test_fl_client_protocol_contract.py`，但不得建立新的production abstraction只為
測試。

### 7.5 `nwdaf-docs/`

Implementation期間更新本plan的status／evidence與同一phase review ledger；不修改
candidate design語意。若production evidence與既有schema矛盾，先停在decision gate，
不得由implementation靜默改寫design。

---

## 8. 實作順序

### 8.1 先建立契約特性測試

1. 在Go與PyMTLF以candidate OpenAPI examples建立最小合法fixtures。
2. 先證明目前Go typed reconstruction會丟失candidate fields、PyMTLF只會unknown
   passthrough、topology-only Notify會被拒絕。
3. 加入expected-failing tests，分開schema／receiver／route lifecycle evidence。

### 8.2 Go typed contract 與接收者驗證

1. 新增candidate types與structured validation error。
2. 接入subscription／patch／Notify parse與existing FL validation。
3. 實作persistent／operation split與PATCH helper。
4. 更新public／private pre-parser的ProblemDetails mapping。
5. 完成local／remote receiver identity source與tests。

### 8.3 Go route feature state 與原子化 lifecycle

1. Route保存offered／negotiated features與bound participant identity。
2. Create success驗證feature subset、candidate response與write-only absence。
3. PUT／PATCH在destination call前gate，在成功後原子更新persistent views。
4. Notify使用bound identity與route feature state；一般round expected equality不變。
5. DELETE／backend reset清除新增state。

### 8.4 PyMTLF typed mirror 與 resource foundation

1. 建立explicit Pydantic fields與nested strict models。
2. 對應Go的recursive／cross-field rules與structured paths。
3. 將operation-scoped values抽成當次operation descriptor。
4. Accepted response計算feature intersection；Slice 1不advertise feature 3。
5. Candidate resource不啟動舊preparation／training，進入`READY`並釋放capacity。
6. PUT／PATCH／DELETE／generation reset保持atomic與可清理。

### 8.5 跨邊界驗證

1. Go local processor＋fake PyMTLF destination驗證`201`、`200`、`204`與failure。
2. Go peer processor＋fake peer驗證raw request、response、callback URI與feature state。
3. Go backend client／peer consumer tests確認candidate body不被transport改寫。
4. PyMTLF API tests確認direct private caller不能繞過mirror validation。
5. 最後執行兩個production repositories的full test／lint／build gates。

---

## 9. Slice 1 的 conformance 責任

| 案例 | Slice 1 證據 | 明確延後 |
| --- | --- | --- |
| `REQ-01`–`REQ-03` | Go／PyMTLF decoder、existing-resource correlation與no-partial-update tests | 無 |
| `TOP-01`–`TOP-08` | Shape、root identity、recursive uniqueness、policy relation、disabled＋lookup與FedProx validation | Downstream subscription side effect |
| `TOP-09` | Forward-compatible enum decode；PyMTLF capability gate回`403`且無fallback | Known policy／strategy executor由Slice 2提供 |
| `PATCH-01`–`PATCH-03` prerequisites | JSON Merge Patch、children array replacement、command stripping與resource atomicity | Local discovered pool、DELETE intent與provenance state由Slice 2提供 |
| `NOT-01` Go portion | Existing callback／correlation加上candidate route gate | Parent coordinator consumption由Slice 4提供 |
| `NOT-02`–`NOT-07`, `NOT-09` | Go primary＋PyMTLF mirror validation | Realized topology state update |
| `NOT-08` prerequisite | Unknown status／cause typed parse與raw relay，不推論known action | Coordinator preservation／action由Slice 2／4提供 |
| `RET-01`–`RET-04` | FOUND／NOT_FOUND／FAILED shape、artifact／round combination與normal-round exception | Lookup state／artifact owner由Slice 3提供 |
| `RET-05`, `RET-06` prerequisites | Create／PUT／PATCH operation extraction與non-persistence | Actual lookup與outcome producer由Slice 3提供 |
| `RET-07`, `RET-08` prerequisites | Go operation gate與route serialization structure | Outstanding state與terminal outcome由Slice 3提供 |
| `FEAT-01` | Initial request可帶offered feature＋candidate fields並進正常validation | 無 |
| `FEAT-02` prerequisites | Fake destination `201`的negotiated bit與Go route state一致 | Production PyMTLF啟用feature 3留給Slice 4 |
| `FEAT-03`, `FEAT-04` prerequisites | Unnegotiated route gate與per-resource state | Sender DELETE、status report與multi-edge E2E由Slice 4提供 |

Slice 1不得因某個case只完成wire prerequisite就將整個procedure case標成通過。

---

## 10. 測試矩陣

### 10.1 Go focused 測試

- Valid candidate subscription／patch／Notify exact JSON round trip。
- Unknown nested property、closed strategy method、missing FedProx parameter、negative range。
- Duplicate sibling、duplicate descendant、ancestor repetition、depth 17與node 1025。
- Receiver root match／mismatch，local與selected peer兩種source。
- Priority child缺少`priority`、`minAvailableNodes < minTrainNodes`、disabled＋lookup。
- Topology-only Notify、report identity mismatch、status／cause matrix。
- Retained FOUND／NOT_FOUND／FAILED combinations；FOUND不套normal expected-round
  equality，一般result仍套用。
- Create local／remote `201`、missing／invalid Location、malformed success與compensation。
- PUT／PATCH local／remote `200`／`204` effective representation equivalence。
- Operation command在destination request可見、在success response與route state不存在。
- Offered／negotiated subset、feature 3 gate與route clone isolation。
- Destination failure／stale revision／generation reset不更新candidate state。
- Candidate-free existing preparation、round、Notify與DELETE tests不變。

### 10.2 PyMTLF focused 測試

- Pydantic alias、strict nested properties與forward-compatible enum behavior。
- 與Go相同的recursive／cross-field table與`invalidParams` path。
- Initial candidate request在feature-disabled state不啟動preparation／training／lookup，
  accepted response不宣告feature 3。
- Persistent topology保留；top-level與nested retained command不保存。
- PUT完整replacement、PATCH omission／children replacement、`200` representation。
- Replace／patch failure還原resource、revision、capacity與callback slots。
- DELETE與`abort_generation()`清除新增state。
- Candidate-free legacy preparation仍走現有artifact path。

### 10.3 不在本 slice 執行的測試

- `nwdaf-resources` real-process scenarios；
- real NRF、ADRF、MongoDB或multi-NWDAF testbed；
- policy selection、aggregation、retained lookup與protocol HFL E2E。

這些是後續slice的handoff，不以unit或fake-destination tests宣稱已完成。

---

## 11. 驗證命令

Production implementation完成後至少執行：

### 11.1 `NWDAF/`

Focused：

```bash
go test ./internal/compat/mlmodeltraining
go test ./internal/context -run MLModelTraining
go test ./internal/sbi -run MLModelTraining
go test ./internal/mtlf -run MLModel
go test ./internal/sbi/processor -run MLModelTraining
go test ./internal/sbi/consumer -run MLModelTraining
```

Full gate：

```bash
make test
make lint
make build
```

### 11.2 `PyMTLF/`

Focused：

```bash
uv run pytest -q tests/test_ml_model_training_wire.py
uv run pytest -q tests/test_ml_model_training_api.py
uv run pytest -q tests/test_fl_client.py
uv run ruff check src/py_mtlf/wire/ml_model_training.py src/py_mtlf/wire/features.py src/py_mtlf/core/fl_client.py src/py_mtlf/api/ml_model_training.py src/py_mtlf/api/problems.py src/py_mtlf/app.py tests/test_ml_model_training_wire.py tests/test_ml_model_training_api.py tests/test_fl_client.py
```

Full gate：

```bash
uv run pytest -q
uv run ruff check .
```

若最終新增focused test file，commands需同步納入。所有未執行或受環境限制的check在
review handoff中逐項列出。

---

## 12. Repository review 與 commit 邊界

- `NWDAF/`與`PyMTLF/`分開review、分開commit；任一repository通過不得宣稱Slice 1
  end-to-end完成。
- Implementation期間先完成一個repository的focused diff與review handoff，再進入
  另一個repository，避免同時累積兩個大型production diff。
- `nwdaf-docs/`只更新plan status、conformance evidence與review ledger，另成文件
  commit。
- Commit message與split需在user確認review後另行提出；本plan不構成commit approval。

建議production順序為：

```text
NWDAF typed contract／route lifecycle
  -> user review checkpoint
  -> PyMTLF typed mirror／resource lifecycle
  -> user review checkpoint
  -> cross-repository conformance reconciliation
```

---

## 13. Review 檢查表

### 13.1 契約與責任

- [x] 沒有修改generated OpenAPI code或新增Go package。
- [x] 四個top-level candidate properties在Go／Python使用相同JSON names。
- [x] Expected receiver／participant identity在consumption point有independent source。
- [x] Go只做wire／route validation；PyMTLF擁有execution capability decision。
- [x] Candidate request沒有被接到legacy model-bundle execution。

### 13.2 狀態與原子性

- [x] Persistent與operation-scoped values在Create／PUT／PATCH均分離。
- [x] Accepted／backend representation除了callback URI外candidate semantics相同。
- [x] `200`與`204`產生相同effective persistent state。
- [x] Failed mutation不改representation、feature state或revision ownership。
- [x] DELETE／reset清除新增state，late callback／mutation仍被fence。

### 13.3 Feature 協商

- [x] Offered／negotiated mask與bound participant保存於每個route。
- [x] Response features不得超出offered mask。
- [x] Unnegotiated route拒絕後續candidate operations。
- [x] Slice 1 PyMTLF不advertise feature 3，也不啟動candidate execution。
- [x] Candidate-free legacy flow無behavior change。

### 13.4 驗證與證據

- [x] Go／PyMTLF recursive limits、path format與cross-field results一致。
- [x] `400`與`403`分類沒有混用。
- [x] Normal result與retained FOUND的`roundInd`規則分開。
- [x] 每個Slice 1 normative item有production path與deterministic test evidence。
- [x] Full test／lint／build results與skipped integration checks完整記錄。

---

## 14. 完成條件

Slice 1只能在下列條件全部成立後進入`Ready for User Review` implementation state：

1. `NWDAF`與`PyMTLF` exact-file changes均在本plan範圍內；
2. §9中屬於Slice 1的wire／receiver／resource prerequisites均有direct tests；
3. §5 baseline每個stage disposition均由final diff保留；
4. §11 focused與full gates完成，或未執行項目明確列為verification gap；
5. Mandatory initial review與所有in-scope remediation完成；
6. Active plan與development policy已依Final Completion Re-read Gate重新讀取並完成
   final conformance map；
7. Intended changes保持unstaged／uncommitted，交由user review。

即使以上條件成立，Slice 1仍只代表wire／resource foundation完成；feature 3 production
advertisement、protocol-driven hierarchy execution與real-process evidence仍屬後續
slices。

---

## 15. 實作證據

Slice 1 的 production paths、deterministic tests、初始審查發現、修正結果、
focused／full verification 與明確延後項目，統一記錄於
[Protocol Extension Implementation Review Ledger](../Protocol%20Extension%20Implementation%20Review%20Ledger.md)。

目前沒有未關閉的 Slice 1 code finding。尚未執行的 real NRF、ADRF、MongoDB、
multi-NWDAF testbed 與 protocol-driven HFL E2E，均為本計畫 §3.2 與 §10.3 已核准的
後續 slice 或 integration verification boundary，不作為本 slice 的完成宣稱。
