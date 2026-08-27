# Hierarchical NWDAF Federated Learning 實作證據包

日期：2026-08-27

最後更新：2026-08-28

狀態：草稿；HFL testbed functional E2E 已確認；正式 Flat／HFL experiment 待執行

## 1. 文件目的

本文件集中整理论文／實驗提案所需的實作與整合證據，回答三組問題：

1. 現有 Flat／Hierarchical FL 實作已完成並驗證到哪個層級；
2. 論文指標能否從現有component與event boundary可靠取得；
3. HFL 與 3GPP NWDAF 機制整合時，哪些現象屬於 standard boundary、
   free5GC-specific issue、our design choice，或 misunderstanding／non-issue。

本文件是 [Proposal 初稿](proposal_draft.md) 的supporting evidence，不取代canonical
implementation plan、testbed repository的raw run record或正式experiment specification。

## 2. 證據政策與狀態標記

### 2.1 證據層級

| 證據類型 | 本文件中的用途 |
| --- | --- |
| TS 條文 | 判斷procedure intent、角色、責任與允許的組合方式 |
| Release 18 OpenAPI YAML | 判斷 path、method、status、header、field 與 schema constraint |
| 現行系統行為 | 說明各角色實際如何協作、資料如何流動，以及成功與失敗時如何收斂 |
| Deterministic test／本機 E2E | 證明本機contract、process composition與multi-process behavior |
| Testbed 證據 | 證明指定revisions在real NRF／NF／VM environment的執行結果 |
| 設計推論 | 由規格與實作證據推導，但不是3GPP直接定義的結論 |

### 2.2 狀態標記

| 狀態 | 定義 |
| --- | --- |
| `Confirmed` | 已有直接來源或執行證據，且目前未發現反證 |
| `Reported` | 由團隊或執行者回報，尚未取得可重查的紀錄 |
| `In progress` | 執行owner已直接確認正在執行，但結果與可重查紀錄尚未完成 |
| `Pending verification` | 已有候選結論，但仍需完成指定查證 |
| `Open` | 尚未形成足夠結論 |
| `Not applicable` | 經查證後不適用於目前範圍 |

同一事項可以在本機為 `Confirmed`，但在testbed仍為
`Pending verification`。本文件不得以 unit／mock／local multi-process evidence 取代
real NRF、跨 VM、SMF／UPF、UE 或 data-plane evidence。

## 3. 實作事實與 E2E 狀態

### 3.1 目前結論

| 項目 | 本機證據 | Testbed 證據 | 狀態／待補資料 |
| --- | --- | --- | --- |
| 同一套實作支援 Flat／HFL | Explicit orchestration config與本機real-process scenarios已完成 | 同一PyMTLF revision已分別跑通Static HFL與fresh Static Flat regression | 本機與testbed functional flows皆`Confirmed` |
| Branch upper-client／lower-server dual role | 本機HFL實作、tests與multi-process E2E已完成 | 兩個Branches各管理兩個Leaves，完成lower process、upper回報與6-resource cleanup | 本機與testbed functional E2E皆`Confirmed` |
| Same global-round semantics | 本機流程已驗證每次lower-tier aggregation後形成upper-tier update | 兩輪均完成4個Leaf updates、2個Branch aggregates與1個Root aggregate | 本機與testbed functional E2E皆`Confirmed` |
| HFL testbed E2E | 不適用 | real three-VM Static HFL已完成collection、training、validation、publication與cleanup | `Confirmed`；只代表bounded functional E2E |
| Flat／HFL正式實驗就緒度 | 本機isolated scenarios已存在，但不是正式論文實驗結果 | Static HFL與Static Flat functional runs已跑通；paper instrumentation與dataset contract尚未完成 | `Open`；不得把functional runs當正式比較結果 |

目前canonical plan的decision log仍將「explicit flat／hierarchical orchestration」標為
implementation pending，但testbed使用的`PyMTLF@7479629`已包含對應實作：執行時會明確選擇
Flat或Hierarchical orchestration，Flat可以固定participant topology直接啟動。故本文件將
implementation與functional E2E狀態列為`Confirmed`，同時保留canonical plan ledger需要另行
reconcile的文件差異。

本機的canonical status仍見
[Hierarchical NWDAF Federated Learning Implementation Plan](../../../plans/hierarchical-federated-learning/Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)；
testbed摘要見
[NWDAF Testbed 整合進度摘要](../../../progress/testbed_integration_status.md)。

### 3.2 Flat／HFL 切換與公平性邊界

Flat與HFL使用同一套NWDAF／PyMTLF implementation；執行時由selected configuration明確選擇
orchestration mode與static participant topology，不是依Server／Client engines是否存在來推論模式。
兩種模式的差異如下：

| 項目 | Flat | HFL |
| --- | --- | --- |
| Top-level coordinator | Root／FL Server | Root／FL Server |
| Participant topology | Root直接指定所有FL Clients | Root指定Branches，並為每個Branch指定Leaves |
| Training resources | Root直接建立Client subscriptions | Root建立upper-tier Branch subscriptions；各Branch再建立lower-tier Leaf subscriptions |
| Intermediate role | 不適用 | Branch對上為FL Client、對下為FL Server |
| Round result flow | Client update直接回Root | Leaf updates先由Branch聚合，再由Branch回傳單一upper-tier update |

因此「同一套實作」不表示Flat與HFL使用完全相同的config file；topology-specific configuration必然
不同。正式paper experiment的fairness contract應固定相同implementation revision、model、
initialization、local training algorithm／epochs、participant datasets、global rounds與offline evaluator，
只改變mode與topology-specific assignment。現有testbed中的Static HFL與Static Flat regression使用
相同PyMTLF revision，足以證明兩種模式都能執行；但兩個functional runs的algorithm與目的不同，
不能直接作為正式Flat／HFL比較結果。

### 3.3 Testbed functional E2E 證據

本次Static HFL functional E2E的權威摘要由
`ChingJe/testbed-docs@89f0c4f`中的
`static-hierarchical-controlled-flow-validation-2026-08-28.md`保存；執行所用Infrastructure
source為`feat/r18-hierarchical-federated-learning@9a7bc914`。已固定的主要identity如下：

| 項目 | 已確認內容 |
| --- | --- |
| Component revisions | NWDAF `6aed268`、PyMTLF `7479629`、PyAnLF `6a4d94a`、ADRF `905f059` |
| Runtime placement | `core`、`path-a`、`path-b`三台VM；1 Root、2 Branches、4 Leaves、7個NWDAF／PyMTLF instances |
| Run identity | `84708477-5dca-499f-a8a0-640b25a30c8c` |
| Training topology | exact 2 Branches × 2 Leaves；兩輪Leaf FedProx與two-tier sample-count-weighted aggregation |
| Data path | 四個Leaves各取得real SMF／UPF collection evidence並保留各自local snapshot |
| Completion path | four-Leaf final validation、ADRF／Root catalog publication、2 upper＋4 lower resources cleanup |
| Regression | 同一Phase另以fresh Static Flat run確認shared operator／renderer／status lifecycle未被破壞 |

這份證據足以支持「Root–Branch–Leaf HFL已在real testbed跑通」的C2 implementation／testbed
realization；它不是正式Flat／HFL performance experiment。該run使用real collection path與bounded
smoke configuration，而paper主實驗仍採固定local datasets、共同held-out test set與另外實作的
communication／timing instrumentation。

Raw logs、generated configs、run artifacts及完整操作紀錄由
`5G_NWDAF_Infrastructure` 保存；本文件只保存可供 proposal 引用的摘要與精確來源。
目前workspace中的唯讀reference已抓取本次feature branch；`main@7d0a36c`仍是舊版，不能取代
前述`9a7bc914`作為本次HFL source identity。

## 4. 指標與 instrumentation 可行性

### 4.1 初始可行性矩陣

下表整合第一輪production path trace；取值點可由現行source確認，但正式experiment logging
尚未實作。本次functional testbed observability只證明流程閉合，不等同paper metric stream。

| 指標 | 論文語意 | 候選measurement boundary | 目前狀態 | 待確認事項 |
| --- | --- | --- | --- | --- |
| Artifact body bytes（Primary） | `mLModelUrl`後續HTTP GET實際收到的compressed model／update bundle body | 各接收節點的artifact download response stream | source取值點`Confirmed`；instrumentation待實作 | 同一URL被多個receiver下載時逐次計算；partial transfer也保存實收bytes |
| SBI body bytes（Supporting） | Public Training request／response／PATCH／callback的JSON body | Go NWDAF outbound standard transaction boundary | source取值點`Confirmed`；instrumentation待實作 | 排除Go NWDAF ↔ PyMTLF `/internal/v1` relay與HTTP headers |
| Root-facing bytes | Flat：Client ↔ Root；HFL：Branch ↔ Root | Root-facing artifact body與SBI body events離線加總 | source取值點`Confirmed`；instrumentation待實作 | 依frozen topology與artifact-origin mapping分類 |
| Lower-tier bytes | HFL：Leaf ↔ Branch | Lower-tier artifact body與SBI body events離線加總 | source取值點`Confirmed`；instrumentation待實作 | Flat為N/A，不填0以免混淆 |
| Total FL bytes | 全run的FL application body bytes | `artifact_body_bytes + sbi_body_bytes`，再依全部tier加總 | 計算方式`Confirmed`；instrumentation待實作 | 同一HTTP body只選一個authoritative observation，避免sender／receiver重複計算 |
| Artifact share | Artifact body bytes占Total FL bytes比例 | 前述raw events離線計算 | 計算方式`Confirmed`；正式結果待實驗 | 用來驗證「model／update transfer是主要負荷」，不預先把假設寫成結果 |
| Message count | 成功與失敗的FL-related application messages | 與bytes共用event schema | source取值點`Confirmed`；instrumentation待實作 | request、response、callback與每次retry各自一筆attempt |
| E2E training time | Root接受trigger至最後global round aggregation完成 | Root top-level coordinator acceptance至最後一次aggregate publication | lifecycle boundary `Confirmed`；timestamp待實作 | 不以較晚的final validation／ADRF publication terminal state作終點 |
| Per-global-round latency | 同一global round的Root dispatch至Root aggregate | Root `ROUND_DISPATCH`至該round aggregate publication | lifecycle boundary `Confirmed`；timestamp待實作 | 全程使用Root process monotonic clock；另存wall-clock timestamp供trace |
| Root／Branch aggregation time | 單次aggregation owner的執行時間 | Root或Branch進入aggregation至發布聚合結果 | lifecycle boundary `Confirmed`；timestamp待實作 | raw event分開保存artifact fetch／validation、aggregate compute與publication |
| Final model performance | 每次run結束後，以Root final model在同一份held-out test set上計算WAPE | Root final model artifact與獨立offline evaluator | 評估方式`Confirmed`；dataset split與identity待補 | 與FL流程內的distributed final validation分開，不以local validation shards代替paper test set |
| Process CPU／peak RSS | 個別Root／Branch process resource usage | External process-level sampler | `Open／Optional` | shared host干擾與採樣成本 |

### 4.2 紀錄結構契約

Flat／HFL應使用同一套raw event schema。每筆FL／SBI event至少需要：

- timestamp與clock source；
- sender、receiver與containing NWDAF identity；
- topology mode與tier；
- experiment／Run ID、FL process／correlation ID與round ID；
- message type、direction與success／failure；
- transfer kind（`SBI_BODY`／`ARTIFACT_BODY`）、application payload bytes與model artifact
  bytes；
- artifact URL、digest、role、HTTP attempt與actual received／sent body bytes；
- retry／attempt identity，以及是否納入paper metric。

本節只固定觀測資料需求，不預先決定必須使用application log、structured event、HTTP
middleware、artifact metadata、runner observation或外部process sampler。選擇實作方式前，
必須先確認每個值的authoritative producer、transport path與double-counting規則。

### 4.3 Artifact data path查核

現行architecture不是由Go NWDAF代理model bytes。Go NWDAF只在public Training SBI中傳遞
`mLModelUrl`；收到URL後，各NWDAF所屬PyMTLF再直接對artifact publisher的PyMTLF送出HTTP
GET。因此SBI JSON只包含control reference，不能代表model／update的實際傳輸量。

#### HFL artifact transfer inventory

下表固定正式實驗中每個logical recipient應有的network transfer semantics；同一PyMTLF
process內的load、copy、aggregate與republish不列為network transfer。實際HTTP attempts仍由
receiver-side raw events逐筆保存；正式run開始前必須確認實作與本logical transfer
contract一致。

| 階段 | Direction／tier | Artifact | Logical transfer semantics | 計量處理 |
| --- | --- | --- | --- | --- |
| Preparation | Root → Branch／Root-facing | Branch assignment bundle | 每個Branch一次 | 計入實際HTTP response body bytes |
| Preparation | Branch → Leaf／Lower-tier | Leaf assignment bundle | 每個Leaf一次 | 計入實際HTTP response body bytes |
| Preparation | Leaf → Branch／Lower-tier | Leaf preparation acknowledgement中的assignment URL | 不產生artifact transfer | 只計SBI body，不以URL反推artifact bytes |
| Preparation | Branch → Root／Root-facing | Preparation-result bundle | 每個Branch一次 | 計入實際HTTP response body bytes |
| 每個training round | Root → Branch／Root-facing | Upper `ROUND_INPUT` | 每個Branch一次 | 計入實際HTTP response body bytes |
| 每個training round | Branch → Leaf／Lower-tier | Republished lower `ROUND_INPUT` | 每個Leaf一次 | 計入實際HTTP response body bytes |
| 每個training round | Leaf → Branch／Lower-tier | Leaf `ROUND_LOCAL` | 每個Leaf一次 | 計入實際HTTP response body bytes |
| 每個training round | Branch → Root／Root-facing | Branch `HIERARCHY_AGGREGATE`／`ROUND_LOCAL` | 每個Branch一次 | 計入實際HTTP response body bytes |
| Final validation | Root → Branch／Root-facing | Final `ROUND_GLOBAL` candidate | 每個Branch一次 | 計入實際HTTP response body bytes |
| Final validation | Branch → Leaf／Lower-tier | Republished candidate | 每個Leaf一次 | 計入實際HTTP response body bytes |
| Final validation | Leaf → Branch／Lower-tier | Leaf `ACCURACY_CHECK`／`ROUND_LOCAL` | 每個Leaf一次 | 計入實際HTTP response body bytes |
| Final validation | Branch → Root／Root-facing | Hierarchical `ACCURACY_CHECK`／`ROUND_LOCAL` | 每個Branch一次 | 計入實際HTTP response body bytes |

Preparation assignment、preparation result、round input／result與validation result都不是只有小型
manifest；發布節點會將model artifact與metadata一併封裝進compressed bundle。故model size很可能
主導application communication，但這仍是待正式量測驗證的systems hypothesis。Paper應同時呈現
artifact bytes、SBI bytes與artifact share，不能只呈現SBI payload或message count。

#### Flat control的artifact inventory

Flat flow的對應network transfers為：preparation時Root → 每個Client的base model一次；每個
round為Root → Client的`ROUND_INPUT`一次及Client → Root的`ROUND_LOCAL`一次；final
validation為Root → Client的candidate一次及Client → Root的`ACCURACY_CHECK`一次。Preparation
success callback會回傳原base-model info，但Flat Server不因此再次下載該URL。

這份inventory也固定以下counting rules：

- artifact大小必須按**每次HTTP GET attempt**計算；一個URL被`N`個participants成功下載時，
  是`N`次transfer，不是一份artifact；
- primary counter採artifact receiver實際從response stream讀到的bytes。此boundary可取得local
  receiver identity，也能在exception前保存partial bytes；
- sender identity由本次run凍結的`artifact public_base_url／origin → nfInstanceId` mapping
  解析。該mapping必須納入experiment metadata，不能只靠runtime source IP猜測；
- 成功完整下載時，receiver累計值應等於publisher端artifact file size，可用`size_bytes`作
  consistency check；`size_bytes`本身不能辨認multiple GETs、partial transfer或retry；
- artifact sender-side stream counter可作diagnostic cross-check，但paper加總只採receiver-side
  authoritative event，避免同一body重複計算；
- run window內所有實際讀到的artifact body bytes皆納入primary metric，包括failed attempt與
  retry。另保存success-only logical bytes作diagnostic breakdown；
- 現行artifact download沒有application-level automatic retry；`attempt_id`仍保留
  給未來retry、人工重跑與failed-transfer correlation；
- HTTP headers、TCP／IP／Ethernet overhead與NRF background traffic維持不計。

### 4.4 SBI與timing production path查核

Communication control path可量，但目前尚未存在符合§4.2的完整structured measurement stream：

- Go NWDAF在公開Training SBI的inbound與outbound transaction boundary都持有完整JSON
  body，因此application payload bytes可在不重編碼JSON的情況下量測；
- 上述public NWDAF-to-NWDAF traffic必須與Go NWDAF ↔ PyMTLF的`/internal/v1` relay分開，
  後者不納入paper的Root-facing或lower-tier communication；
- SBI raw metric建議由outbound standard transaction boundary保存一次：request／callback body
  是initiator實際送出的bytes，response body是同一initiator實際收到的bytes。Inbound handler
  若另記錄，只作cross-check，不再加入paper metric；
- artifact與SBI events以sender／receiver NF identity及frozen topology離線分類為Root-facing或
  lower-tier，不需要把`UPPER`／`LOWER`擴充為public SBI field。

Timing path也可量，但現行snapshot只保存state、round與terminal retention time，尚未保存
paper所需的起訖timestamps：

- Flat與HFL皆由top-level coordinator接受manual trigger，並共用相同的Server round
  lifecycle；E2E start應記在request首次被接受且active reservation建立完成時；
- E2E end應記在最後一次`ROUND_GLOBAL`完成並發布後。現行flow隨後還會執行final
  validation與publication，因此不能直接把`COMPLETE`／`CUTOVER_PENDING`時間當成論文定義的
  E2E終點；
- per-round latency可在Root同一process內以monotonic clock量測，避免跨VM clock skew；
  Branch lower-tier latency與upper／lower mapping可作supporting breakdown；
- aggregation raw timestamps應至少區分`AGGREGATING` stage、
  `FederatedTrainer.aggregate()` compute與output artifact publication，之後再離線決定paper圖表
  採用stage time或compute-only time。

Learning outcome採用獨立的post-run evaluation。每次Flat／HFL run結束後，保存Root產出的
final global model artifact，再由offline evaluator以同一份凍結的held-out test set計算WAPE。
這份結果作為paper比較Flat／HFL learning outcome的主要指標；現行由各Flat Client或Leaf在
local validation shard上執行的distributed final validation仍保留，用於FL lifecycle完成與
功能驗證，但不代替paper的held-out test result。

Dataset應先拆成`train`、`validation`與`test`，再進行participant sharding：`train`供各Leaf
local training，`validation`供訓練期間的validation或model selection，`test`則不得參與
training、validation、participant selection或參數調整。Flat／HFL以及4／8／16 participants
都必須使用同一份test set；只對`train`／`validation`依participant count與partition rule產生
local shards。實際split比例、random seed、test-set identity與offline evaluator input contract
由Dataset／Generator Design Packet另行確認。

## 5. HFL 整合觀察稽核

### 5.1 統一查核格式

每項觀察使用以下欄位：

1. **Requirement**：hierarchical execution需要解決的精確問題；
2. **Spec evidence**：TS text與OpenAPI分開列出；
3. **目前實作邏輯**：用角色分工、執行順序、資料／結果流向及失敗處理說明系統現在如何運作；
4. **Consequence**：若沒有額外處理，實際procedure或interoperability會發生什麼；
5. **Our treatment**：目前實作如何處理，以及是否為第一版限制；
6. **Classification**：`standard boundary`、`free5GC-specific issue`、
   `our design choice`或`misunderstanding／non-issue`；
7. **Status**：`Confirmed`、`Pending verification`、`Open`或`Not applicable`。

本輪稽核使用 Release 18 TS 23.288 v18.13.0、TS 29.520 v18.14.0 與本地
Release 18 OpenAPI corpus。現行實作快照為 `NWDAF@6aed268`、
`PyMTLF@7479629` 與 `nrf@0dd4024`。Source／deterministic conclusions與§3.3的functional
testbed record分開標示；testbed evidence只支持實際跑通的static successful flow，不擴張成
dynamic topology或performance claim。

| 證據入口 | 本輪用途 |
| --- | --- |
| [TS 23.288 §6.2C](<../../../../specs/TS 23.288/6 Procedures to Support Network Data Analytics/6.2C Federated Learning among Multiple NWDAFs.md>) | FL registration／discovery、preparation、round與maintenance procedure |
| [TS 29.520 §4.6](<../../../../specs/TS 29.520/4 Services offered by the NWDAF/4.6 Nnwdaf_MLModelTraining Service.md>) | Training subscription／notification HTTP behavior |
| [Nnwdaf ML Model Training OpenAPI](../../../../specs/openapi/TS29520_Nnwdaf_MLModelTraining.yaml) | `mLPreFlag`、`mLModelInfos`、`statusReport`與`termTrainReq` schema |
| [Nnwdaf ML Model Provision OpenAPI](../../../../specs/openapi/TS29520_Nnwdaf_MLModelProvision.yaml) | `MLEventNotif`與`MLModelAddr.mLModelUrl` schema |
| [NRF NF Management OpenAPI](../../../../specs/openapi/TS29510_Nnrf_NFManagement.yaml) | `NwdafInfo`、`MlAnalyticsInfo`與`FlCapabilityType` |
| [NRF NF Discovery OpenAPI](../../../../specs/openapi/TS29510_Nnrf_NFDiscovery.yaml) | exact-instance、service與ML analytics discovery query |

### 5.2 P1：Branch dual role／upper-lower FL process composition

- **Requirement**：確認Branch如何對Root作為FL Client、對Leaves作為FL Server，並維持兩個
  process的identity、round、callback與lifecycle mapping。
- **Spec evidence**：TS 23.288 §6.2C.2.1允許NWDAF以FL Server、FL Client或兩者能力
  註冊，並由FL Server發現、選擇及邀請FL Clients；§6.2C.2.2則定義單一
  Server–Clients process中的訓練與聚合。Release 18 `FlCapabilityType`也包含
  `FL_SERVER_AND_CLIENT`。上述證據允許同一NWDAF承擔雙能力，但未定義Branch角色、兩個
  subscriptions的parent–child關係或cross-tier round mapping。
- **目前實作邏輯**：Branch收到Root指派後，先驗證自己同時具備FL Client與
  FL Server能力，再將面向Root的upper-tier Client process與面向Leaves的lower-tier
  Server process綁定在同一個hierarchy plan。每輪先向Leaves發布lower-tier input並完成
  下層聚合，再把單一Branch aggregate傳回Root。系統以plan、upper correlation與round
  的組合映射上下層process；取消、restart或shutdown時會一併清除mapping與lower-tier
  work，避免殘留結果回到舊的upper process。
- **Consequence**：若只共用`mlCorreId`或round number，Root round與Branch local round可能
  誤關聯，重送也可能重複執行lower-tier aggregation；若只清除upper resource，則lower
  process可能殘留並送出late callback。
- **Our treatment**：將hierarchy實作為兩個獨立FL processes的同 vendor composition；
  cross-process mapping、idempotency與cascade cleanup由Branch本地runtime負責，不宣稱為
  3GPP直接定義的procedure。
- **Classification**：`standard boundary`（標準未表達cross-process binding）＋
  `our design choice`（Branch composition profile）。
- **Status**：source／deterministic tests與two-Branch／four-Leaf testbed functional flow皆為
  `Confirmed`。

### 5.3 P2：NRF capability discovery與hierarchy／topology establishment

- **Requirement**：區分NRF能證明的candidate capability／endpoint資訊，與Root／operator建立
  parent-child assignment所需的額外資訊。
- **Spec evidence**：TS 23.288 §6.2C.2.1列出NWDAF profile中的Analytics ID、位址、
  service area、FL capability與支援時間，並允許FL Server依這些條件從NRF發現及選擇
  Clients。NRF OpenAPI的`NwdafInfo.mlAnalyticsList`可表達`mlAnalyticsIds`、
  `mlModelInterInfo`與`flCapabilityType`；NF Discovery也提供`target-nf-instance-id`、
  `service-names`與`ml-analytics-info-list`。查核範圍內未找到Root–Branch–Leaf、
  parent–child assignment或topology lifecycle欄位。
- **目前實作邏輯**：Root從只有自己持有的static topology取得Branch／Leaf分組
  intent，但不直接信任設定檔。發布assignment前，Root會透過NRF依NF identity逐一確認
  節點仍已註冊、具備所需FL能力，且提供相容的Training service。Branch收到自己的
  assignment後，也會重新向NRF解析各Leaf當下的endpoint與capability，不沿用Root早先
  觀察到的可能過期資訊。
- **Consequence**：只靠capability search可以得到eligible candidates，卻不能推出哪個Leaf
  屬於哪個Branch；反過來，只信任static file又不能證明目標instance目前仍註冊、具備能力
  或提供可用的Training service。
- **Our treatment**：operator／Root config提供topology intent，NRF只負責當下的identity、
  capability與endpoint resolution。第一版固定`strategy: static`、不做hot reload、runtime
  reconfiguration或Branch自行補選。
- **Classification**：`standard boundary`（discovery不等於hierarchy establishment）＋
  `our design choice`（Root-only static assignment）。
- **Status**：source／deterministic tests為`Confirmed`；testbed functional record確認七個unique
  NWDAF profiles與exact 2×2 admission。

### 5.4 P3：`mLModelUrl`／model bundle與cross-process mapping

- **Requirement**：釐清`mLModelUrl`在preparation／training中的標準語意，以及hierarchy
  metadata與cross-process mapping是否屬於標準內容。
- **Spec evidence**：TS 29.520的`MLEventNotif.mLFileAddr`表示ML model file address；
  Release 18 OpenAPI的`MLModelAddr`要求`mLModelUrl`或`mlFileFqdn`擇一。Training request、
  patch與notification可透過`mLModelInfos`攜帶該資訊，但OpenAPI只定義位址與可用欄位，
  沒有定義檔案內的hierarchy manifest、`planId`、publisher／recipient、assigned Leaves或
  upper／lower process mapping。TS 29.520並明載`mlFile`內容格式不在3GPP範圍內。
- **目前實作邏輯**：Root發布給每個Branch的assignment bundle；Branch下載後會確認
  來源、完整性、plan、recipient與model contract，再依自己被分配的subtree為每個Leaf建立
  Branch-owned assignment。Leaf因此只向所屬Branch取得model bundle，不直接向Root下載。
  Preparation完成後，Branch再發布一份bundle，完整列出該subtree的prepared、failed與
  timed-out Leaves，供Root執行整棵topology admission。各節點以plan ownership管理取得的
  artifacts與terminal cleanup。
- **Consequence**：若直接轉交Root URL，Root必須跨tier服務Leaves，破壞locality與artifact
  ownership；若URL內容沒有typed correlation，Root無法判斷結果是否屬於目前plan、正確
  Branch與正確participant set。
- **Our treatment**：保留標準`mLModelInfos.mLFileAddr.mLModelUrl`作為傳輸入口，URL指向
  同 vendor model bundle；hierarchy metadata與bundle validation屬本專案contract。Paper應
  稱為利用標準model-address mechanism承載vendor-specific composition metadata，不應稱其為
  標準hierarchy schema。
- **Classification**：`standard boundary`（3GPP未標準化bundle內容）＋`our design choice`
  （typed bundle與download／republish flow）。
- **Status**：source／deterministic tests為`Confirmed`；testbed functional record確認跨節點
  artifact URL／digest與Leaf–Branch–Root two-tier lineage。

### 5.5 P4：Preparation accept／reject／status expressiveness

- **Requirement**：確認ML Model Training preparation的response、notification與status能否表達
  hierarchical subordinate preparation結果。
- **Spec evidence**：TS 23.288 §6.2C.2.1 steps 7–10定義Server發起preparation、Client檢查
  requirements／model download、Client回報是否加入及可選failure reason，最後由Server決定
  participants。TS 29.520允許subscription creation以201／403／500表達同步結果，非同步
  notification則要求至少包含`mLModelInfos`、`delayEventNotif`或`termTrainReq`之一；
  `statusReport`只是optional supplemental field，不能單獨成為合法notification。標準欄位
  沒有prepared／failed／timed-out subordinate lists或整棵topology admission result。
- **目前實作邏輯**：Leaf若已成功取得並驗證assignment，便以`mLModelInfos`回報
  該model address；無法參與時則以`termTrainReq`回報。Branch不會在第一個Leaf失敗時立即
  結束，而是收集所有terminal outcomes或等到deadline，再將完整partition放入
  preparation-result bundle並回報Root。Root確認每個assigned participant都被唯一分類後，
  才以`COMPLETE_REQUIRED`決定整棵topology是否admitted。Go NWDAF只負責標準notification的
  合法性檢查與relay，不解析subtree result或代替Root做admission。
- **Consequence**：若把`statusReport`誤當preparation-completed latch，會產生不符合
  notification schema的訊息，也無法攜帶per-Leaf outcome；若只使用HTTP 201，Root只能知道
  Branch建立了upper subscription，不能知道lower-tier preparation是否完成。
- **Our treatment**：標準notification欄位負責transport-level outcome；完整subordinate
  evidence放在`mLModelUrl`所指的result bundle，Root保有最終admission ownership。第一版不
  增加新的public SBI field。
- **Classification**：`standard boundary`（無hierarchical outcome schema）；將
  `statusReport`視為獨立成功載體屬`misunderstanding／non-issue`。
- **Status**：source／deterministic tests為`Confirmed`；testbed成功路徑確認四個Leaves完成
  preparation並由Root exact admission，負向outcome仍由deterministic evidence涵蓋。

### 5.6 P5：Server-driven selection與Client-initiated participation

- **Requirement**：確認Release 18 NWDAF FL procedure由誰選擇／邀請participants，以及是否存在
  FL Client主動加入既有Server process的標準mechanism。
- **Spec evidence**：TS 23.288 §6.2C.2.1由FL Server發現、選擇並送出preparation request；
  Client可接受或拒絕，Server再決定實際participants。§6.2C.2.3雖以「dynamic join／leave」
  描述maintenance，但新candidate仍由Server透過NRF得知、重選，並重走Server發起的selection
  procedure。現行Client可主動送`termTrainReq`表示離開；查核範圍內沒有Client主動建立或加入
  既有Server-owned process的獨立service operation。
- **目前實作邏輯**：Root先依static topology與NRF查核結果選定Branches／Leaves，再由
  Root FL Server主動建立upper-tier Training subscriptions。Branch收到並驗證assignment後，由
  Branch FL Server主動建立lower-tier subscriptions。Branch／Leaf可接受、拒絕或中止已收到的
  participation，但不會主動向未知Server加入某個plan，也不在runtime自行替換或重新
  分組。
- **Consequence**：若描述成Client主動加入，會顛倒subscription owner與callback correlation
  的建立方向；但若寫成「標準沒有dynamic participation」也不正確，因為Release 18已描述
  Server-driven reselection與Client-initiated leave。
- **Our treatment**：paper只研究固定participant-count與static topology；將流程精確表述為
  Server-driven selection／invitation，並把dynamic reselection、join／leave與topology
  reconfiguration排除在第一版scope外。
- **Classification**：`standard boundary`（沒有Client-initiated join operation）；否認標準
  已有dynamic maintenance或Client leave則屬`misunderstanding／non-issue`。
- **Status**：spec與source flow為`Confirmed`；不需要以testbed churn驗證，churn為
  `Not applicable`。

## 6. 主張與證據交接

| 候選主張 | 必要證據 | 目前狀態 | Proposal處理 |
| --- | --- | --- | --- |
| 多個既有NWDAF FL processes可組成Root–Branch–Leaf execution | Spec composition analysis、production trace、local與testbed E2E | Local與real three-VM functional E2E皆已確認 | 可支持C1／C2 realization claim |
| Branch可維持upper-client／lower-server dual role並完成two-tier training | Process mapping、correlation isolation、round／cleanup evidence | Source、deterministic audit與two-Branch testbed成功流程已確認 | 作為implementation realization，不擴張為標準已定義行為 |
| HFL降低Root-facing communication | Frozen byte semantics與正式Flat／HFL experiment | 尚無正式結果 | 不提前寫成已證實結論 |
| HFL引入lower-tier communication、Branch aggregation與latency cost | Frozen measurement semantics與正式實驗 | 尚無正式結果 | 以research question／hypothesis表述 |
| 3GPP management／FL models未直接表達hierarchical orchestration | Release-specific TS／OpenAPI audit | P1–P5第一輪完成；OAM evidence reconciliation待補 | 暫列practical implication候選 |

## 7. 待辦事項

- [x] 取得本次testbed functional run的Infrastructure／component revisions、topology、Run ID與
  verified result summary。
- [x] 確認real three-VM Static HFL完成collection、two-tier training、validation、publication與
  cleanup，足以支持proposal C2。
- [x] 明確記錄Flat／HFL共用implementation的切換方式、topology-specific config差異與正式
  experiment fairness boundary。
- [ ] Reconcile canonical plan中explicit orchestration仍標示implementation pending的舊狀態。
- [x] 完成Flat／HFL source-level message與artifact transport初查；testbed route trace待補。
- [x] 形成bytes counting、retry、failure與double-counting候選規則；待experiment spec freeze。
- [ ] 後續正式experiment instrumentation階段再實作artifact receiver-side與Go outbound SBI
  transaction structured measurement events；Friday前只回報feasibility，不實作量測code。
- [x] 確認E2E與per-round的authoritative start／end source boundaries；timestamp實作待補。
- [x] 固定final learning-outcome評估方式：每次run後以Root final model對同一held-out test set
  離線計算WAPE；不以FL流程內的distributed validation代替。
- [ ] 由Dataset／Generator Design Packet確認`train／validation／test` split比例、random seed、
  test-set identity與offline evaluator input contract。
- [x] 完成P1–P5第一輪spec、implementation與classification audit，並補入successful-flow
  testbed functional evidence。
- [ ] 將可成立的claim與limitation回填Proposal初稿。

## 8. 更新紀錄

| 日期 | 更新 |
| --- | --- |
| 2026-08-27 | 建立初稿；固定證據層級、testbed收集欄位、instrumentation矩陣、P1–P5模板與主張交接 |
| 2026-08-27 | 完成P1–P5第一輪查核；區分標準procedure、NRF capability、vendor bundle、preparation result與static topology邊界 |
| 2026-08-27 | 完成instrumentation第一輪source trace；確認communication／timing取值點，並辨識shared held-out evaluation尚未成立 |
| 2026-08-27 | 深入追蹤`mLModelUrl`後續artifact GET；固定artifact-first metrics、各階段logical transfer inventory與receiver-side counting rule |
| 2026-08-27 | 修正testbed evidence provenance：由實際執行者直接確認執行中，非團隊二手回報；同時將measurement code明確延後至正式experiment instrumentation階段 |
| 2026-08-27 | 固定learning-outcome contract：每次run後以Root final model對共同held-out test set離線計算WAPE；dataset另拆train／validation／test，實際split由Dataset／Generator Design Packet確認 |
| 2026-08-28 | 回填real three-VM Static HFL verified record；確認1 Root／2 Branches／4 Leaves的functional E2E、two-tier aggregation、publication與cleanup，並與正式Flat／HFL experiment明確分開 |
| 2026-08-28 | 補充Flat／HFL切換契約：兩者共用implementation revision，但使用不同mode與topology-specific configuration；正式比較另固定training與dataset條件 |
