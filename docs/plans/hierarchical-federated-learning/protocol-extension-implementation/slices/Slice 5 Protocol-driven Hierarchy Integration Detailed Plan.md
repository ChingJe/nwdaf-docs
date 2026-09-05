# Slice 5 — Protocol-driven Hierarchy Integration Detailed Plan

日期：2026-09-04

狀態：Draft／production implementation尚未開始；等待Slice 4 commit完成後進行開工前確認

相關文件：

- [Protocol Extension Implementation Plan](../Hierarchical%20NWDAF%20FL%20Protocol%20Extension%20Implementation%20Plan.md)
- [Protocol Extension Implementation Slice Map](../Protocol%20Extension%20Implementation%20Slice%20Map.md)
- [Protocol Implementation Current-State Inventory](../Protocol%20Implementation%20Current-State%20Inventory.md)
- [Model Bundle Metadata to Protocol Schema Mapping](../Model%20Bundle%20Metadata%20to%20Protocol%20Schema%20Mapping.md)
- [Protocol Resource Lifecycle and Wire Integration Mapping](../Protocol%20Resource%20Lifecycle%20and%20Wire%20Integration%20Mapping.md)
- [Protocol Conformance Case Ownership Mapping](../Protocol%20Conformance%20Case%20Ownership%20Mapping.md)
- [Protocol Extension Implementation Review Ledger](../Protocol%20Extension%20Implementation%20Review%20Ledger.md)
- [Slice 1 Detailed Plan](./Slice%201%20Wire%20Contract%20and%20Resource%20Lifecycle%20Foundation%20Detailed%20Plan.md)
- [Slice 2 Detailed Plan](./Slice%202%20Candidate%20Pool%20Policy%20and%20Local%20Contract%20Execution%20Detailed%20Plan.md)
- [Slice 4 Detailed Plan](./Slice%204%20Controlled%20Local%20Training%20Workload%20Detailed%20Plan.md)
- [Slice 4A Detailed Plan](./Slice%204A%20Digest%20Simplification%20and%20Contract%20Cleanup%20Detailed%20Plan.md)
- [Candidate OpenAPI Schema](../../../../design/hierarchical-federated-learning/candidate_openapi_schema.md)
- [Candidate OpenAPI artifact](../../../../design/hierarchical-federated-learning/candidate_openapi.yaml)
- [Topology、Policy 與 Strategy 細節設計](../../../../design/hierarchical-federated-learning/topology_policy_design.md)
- [Protocol Conformance Matrix](../../../../design/hierarchical-federated-learning/protocol_conformance_matrix.md)

---

## 1. Slice 結果

本slice的production implementation以Slice 4A digest cleanup與Slice 4 controlled
workload完成為前置條件。

本 slice 完成後，protocol mode 應能以 production `Nnwdaf_MLModelTraining` flow 跑通
Root→Branch→Leaf hierarchical FL，而不是只解析 candidate fields或在單一process內
模擬candidate behavior。完整結果包括：

- Root產生hierarchy-wide UUID `mlCorreId`，並將每個direct child應收到的subtree放入
  `x-flTopology`；
- Root、Branch與Leaf逐edge建立model-free preparation subscription，獨立協商
  feature 3；
- 需要local training的Client在preparation不讀取local dataset，也不啟動traffic data
  collection；真正開始training時才由Slice 4 loader讀取deployment掛載的shard；
- Branch依Slice 2 candidate pool執行explicit、delegated或hybrid child
  establishment，並以`x-flTopologyReport`逐級回報realized subtree；
- Root只在realized topology達到readiness後建立該輪global-model ADRF record；
- Root與Branch使用標準`roundInd`及`mLModelInfos`啟動training，Root global model以
  `mLModelAdrf`逐級傳遞，Leaf local result及Branch aggregate以產生節點的暫存
  `mLFileAddr`向上回報；
- Branch需要多個lower-tier rounds時，只有第一個lower round使用Root ADRF
  reference；後續round使用Branch自己的local aggregate URL；
- PUT／PATCH topology update、disabled-child DELETE、topology-only Notify與feature
  mismatch進入真實message path；
- `x-retainedResultReq`仍可被wire contract解析，但runtime明確以`403`拒絕，不建立
  retained-result state，也不沿用舊Branch未送達的結果；
- legacy model-bundle mode在Slice 6決定移除前仍可由明確selector執行與回歸。

完成本slice只代表protocol-driven hierarchy integration與local real-process evidence
成立，不代表Branch failure detection、replacement policy、retained-result recovery、
ADRF caller authorization enforcement或正式testbed experiment完成。

---

## 2. 基準、規格與直接證據

### 2.1 盤點版本

| Repository | Revision | Slice 5 角色 |
| --- | --- | --- |
| `NWDAF/` | `302762a6af677f5ccfb5a3f9d0253fb3dd39bf62` | Model Training SBI owner、backend route、ADRF／NRF private proxy |
| `PyMTLF/` | `0e87ef13622cba0ddf740aa8cedce8ad76705815` | Root／Branch／Leaf hierarchy execution與candidate state owner |
| `nwdaf-resources/` | `83473770e958623f052097349173f56af1e25953` | Multi-process deployment、scenario與evidence owner |
| `adrf/` | `905f059` | ADRF runtime dependency；目前不預期修改 |
| `nwdaf-docs/` | `c8d04d212d3ec6d8e889d041dba7cc82f8ca84b0` 加上本次unstaged plan | Canonical plan、design與review evidence |

Production implementation開始前必須重查所有affected repositories的HEAD與工作樹。
若Slice 1 route contract、Slice 2 candidate pool、ADRF private proxy或real-process harness
已改變，先更新本計畫的exact-file mapping與驗證基準。

### 2.2 Release 18 Model Training evidence

依本workspace收錄的Release 18 TS 29.520與
`TS29520_Nnwdaf_MLModelTraining.yaml`：

- Create request的required properties是`mLEventSubscs`、`notifUri`與`notifCorreId`；
  `mLModelInfos`、`mlCorreId`、`mLPreFlag`與`roundInd`皆為optional；
- Create成功使用`201 Created`並回傳resource body與`Location`；
- PUT與JSON Merge PATCH可更新subscription，成功可為`200`或`204`；
- Notify由FL Client向subscription中的`notifUri`送出POST，成功回`204`；
- `MLEventNotif`的model transport使用`mLFileAddr`或`mLModelAdrf`其中之一；
- `MLModelAdrf`可帶`adrfId`／`adrfSetId`與`storTransId`；本project profile的
  global-model path要求exact `adrfId`與`storTransId`；
- `modelUniqueId`是non-negative integer，不是UUID。Hierarchy-wide procedure identity
  使用UUID `mlCorreId`，兩者不可混用。

因此model-free preparation、後續以PUT／PATCH補入global model，以及各tier共用
`mlCorreId`都不需要改變標準HTTP method。Candidate topology、report與feature 3才是本
project extension。

### 2.3 Release 18 ADRF evidence

依TS 29.575與`TS29575_Nadrf_MLModelManagement.yaml`：

- collection POST建立ML model store record，成功為`201`並回`Location`與record；
- collection GET可依`store-trans-id`或model identity查詢record；
- individual PUT以`storeTransId`更新record，成功為`200`或`204`；
- individual DELETE以`storeTransId`刪除record，成功為`200`或`204`；
- `MLModelInfo`需要`modelUniqueId`、`mlFileAddr`與`mlStorageSize`；
- `allowConsumerList`在標準schema為optional，但本slice的Root global-model profile將它
  視為required implementation requirement，避免建立未記錄consumer scope的record。

`adrf/`已具有上述POST／GET／PUT／DELETE resources。`NWDAF/`目前只向PyMTLF提供
store與retrieve private proxy；Topology update所需的allowlist update及terminal
cleanup尚無PyMTLF可呼叫的Go transport，因此Slice 5必須補NWDAF update／delete
private routes、processor與consumer methods。這是既有標準ADRF procedure的代理補齊，
不是新增ADRF API。

### 2.4 現有 production baseline

- Slice 1已完成Go／Python candidate wire models、Create／PUT／PATCH／DELETE／Notify
  preservation、resource atomicity與per-resource feature state。
- PyMTLF目前在candidate Create上故意不接受feature 3；candidate resource保持idle，
  以避免尚未有execution consumer時宣告可執行。
- Slice 2已完成effective node contract、candidate pool、NRF discovery freshness、
  establishment／cleanup intents、readiness／selection／completion gates、local work與
  realized report snapshot。
- Slice 2 primitives尚未送出真實downstream subscription、DELETE或topology Notify；
  Slice 5必須把intent completion與實際HTTP response關聯，不能把intent產生當作side
  effect成功。
- Existing Root／Branch hierarchy仍以assignment與preparation-result bundles傳遞
  orchestration metadata。Slice 2已修正同一assignment的duplicate GET，但protocol
  mode不得再讀取這些metadata。
- Existing `FLServerEngine`已擁有participant resource、callback correlation、round
  dispatch、wait、result validation、aggregation與DELETE；Slice 5應延伸這個owner，
  不建立第二套subscription client或aggregation engine。
- Existing `FLClientEngine` preparation會先下載model並使用manifest建立trainable
  dataset；Slice 4A先移除多層digest contract，protocol mode再新增model-free path，
  legacy path保持不變。
- Existing final-model `PublicationCoordinator`是durable catalog publication owner；
  每輪temporary ADRF record不得透過它reserve／commit catalog version。

### 2.5 free5GC exemplar boundary

本slice仍以本workspace的NWDAF route與既有tests作為primary implementation truth。
外部structure只比對以下bounded concerns：

- free5GC NF callback／consumer pattern用於確認SBI request、response與error ownership；
- free5GC subscription resource pattern用於確認resource URI與DELETE lifecycle；
- 不複製其他NF的business state，也不以exemplar取代TS 29.520／29.575 contract。

實作review須記錄實際採用的exemplar檔案與採用／未採用的邊界，不得只宣稱
「free5GC-aligned」。

---

## 3. Slice 邊界

### 3.1 納入

- Root-owned protocol／legacy hierarchy selector。
- Hierarchy-wide UUID `mlCorreId`與per-edge `notifCorreId`／resource identity。
- Feature 3逐edge offer、accepted-feature verification與mismatch cleanup。
- Model-free preparation與topology-only report；dataset不在此階段處理。
- Direct-child subscription establishment、callback consumption與realized subtree
  composition。
- Explicit、delegated與hybrid topology的production execution。
- Candidate PATCH reconciliation、serialized downstream mutations與disabled-child DELETE。
- Root per-round ADRF record store、allowlist update、retrieval reference、mapping與cleanup。
- Recipient依exact ADRF identity與transaction ID解析record並下載model payload。
- Branch first lower-round ADRF reference reuse、subsequent local aggregate distribution及
  upward result reuse。
- Unsupported retained-result instruction的403 gate與atomicity。
- Focused unit／boundary tests及local real-process scenarios。
- Legacy hierarchy回歸。

### 3.2 不納入

- 自動偵測Branch failure及選擇replacement Branch的完整owner。
- Retained-result index、保存期限、lookup、handoff或acceptance policy。
- Root在restart後恢復round-to-ADRF mapping；mapping不存在時procedure失效。
- ADRF caller authentication或`allowConsumerList` enforcement實作。
- 修改NRF schema、在NRF保存hierarchy ownership或新增ranking algorithm。
- 最終移除legacy hierarchy artifact roles；由Slice 6處理。
- 正式testbed performance experiment、traffic instrumentation與paper result。
- 新增或修改workload、dataset／trainer／artifact profile；這些由Slice 4先完成。

若實作發現完成本slice必須修改`adrf/`、`nrf/`或candidate design semantics，先回到
boundary review，不把跨repository contract expansion隱藏在production patch中。

### 3.3 Existing model-bundle flow處置

本slice延伸的canonical baseline是目前已跑通的static／model-bundle hierarchy flow。
各stage處置如下，未列出的baseline behavior不得被默認移除：

| Baseline stage | Slice 5處置 | 主要差異或保留條件 |
| --- | --- | --- |
| Root trigger與top-level request registry | Reused without semantic change | 仍由現有private／configured trigger建立一筆Root request |
| Static topology planning | Adapted | Root仍可讀既有static topology作initial requested tree，但protocol message成為execution authority |
| Hierarchy `mlCorreId`建立 | Adapted | 改由Root建立UUID並逐edge共用，不使用bundle plan ID代表procedure |
| Branch／Leaf discovery | Reused and extended | Exact與list discovery仍經containing Go NWDAF；candidate freshness與selection使用Slice 2 owner |
| Preparation subscription creation | Explicitly replaced in protocol mode | 不發布assignment bundle；改送model-free `x-flTopology` request並逐edge協商feature 3 |
| Preparation model download／validation | Explicitly replaced in protocol mode | Preparation不取model；model-dependent validation移至第一輪 |
| Dataset collection與snapshot | Explicitly replaced for controlled workload | Preparation不讀dataset；local training開始時由Slice 4 loader讀取deployment掛載的shard，不啟動traffic DatasetCoordinator |
| Preparation result handoff | Explicitly replaced in protocol mode | 不發布preparation-result bundle；改用topology-only `x-flTopologyReport` Notify |
| Root readiness decision | Adapted | 使用Slice 2 realized candidate state，不再解析preparation-result artifact |
| Global round artifact creation | Adapted | Model bytes與manifest語意保留；Root temporary artifact另建立per-round ADRF record |
| Root→Branch round dispatch | Adapted | `mLFileAddr`改為`mLModelAdrf`，既有PUT／PATCH resource lifecycle保留 |
| Branch→Leaf first lower-round dispatch | Adapted | 轉送同一Root ADRF reference，不republish Root artifact |
| Branch subsequent lower-round dispatch | Reused with explicit transport boundary | 使用Branch local aggregate `mLFileAddr` |
| Leaf local training | Reused from Slice 4 | 第一輪讀取configured shard並確認received model profile相容後，進入既有FL execution owner |
| Branch selected-set wait與aggregation | Reused with Slice 2 gates | 仍使用既有server／aggregator，cohort與completion由effective policy決定 |
| Leaf／Branch result Notify | Reused with protocol authority | 正常result仍以`roundInd`及producer `mLFileAddr`回報 |
| Final validation與final-model handoff | Reused from Slice 4 | Controlled workload使用held-out accuracy evidence；不把temporary round ADRF owner併入final artifact handoff |
| Successful completion | Adapted | 除既有request terminal外，先完成round ADRF cleanup evidence |
| Failure／timeout | Adapted | 停止仍會取用record的resources，再cleanup ADRF；不做retained recovery |
| Process restart | Approved for deferral beyond invalidation | 遺失in-memory protocol／round mapping即使procedure失效，不恢復execution |
| Legacy mode | Reused for regression | 在Slice 6 decision前由`model_bundle` selector保留 |
| Legacy hierarchy metadata removal | Future-phase handoff | Slice 6才移除active runtime payload與dead compatibility code |

Protocol mode與legacy mode共享FL lifecycle、aggregator、workspace安全下載及result
validation owners；workload-specific trainer與final evaluation由Slice 4 profile contract
選擇。任何共用name／artifact role若在implementation中
需要改變意義，必須先更新本表與相關design，不可只靠新branch condition重新解讀。

---

## 4. 固定 execution contract

### 4.1 Mode selection與authority

Root orchestration config新增明確的hierarchy contract selector。計畫採用：

```yaml
federated_learning:
  orchestration:
    mode: hierarchical
    participant_source: static
    hierarchy_contract: protocol
```

`hierarchy_contract`只接受`model_bundle`或`protocol`，預設`model_bundle`以保留現有部署。
Flat mode不得設定此欄位。只有Root autonomous coordinator讀取selector；Branch與Leaf依
收到的subscription contract決定執行路徑，不依各自deployment config猜測上游模式。

同一resource若同時收到`x-flTopology`與legacy hierarchy assignment artifact，屬
ambiguous authority，PyMTLF以requirements error拒絕，不自行挑一個來源。Protocol
resource全程只以message fields控制topology、policy、strategy及reporting。

### 4.2 Procedure與edge identity

- Root在每次hierarchical FL procedure開始時產生UUID字串`mlCorreId`。
- Root→Branch及Branch→child subscriptions逐級傳遞同一`mlCorreId`。
- 每個subscription仍有自己的resource URI與`notifCorreId`；callback先以
  `notifCorreId`找到direct-child resource，再驗證`mlCorreId`一致。
- `roundInd`屬各local FL operation的進度，不被用作整棵hierarchy的唯一state key。
- Root global-model record mapping以`(mlCorreId, upper roundInd)`為key；Branch lower
  round另外保存local round state，不要求與upper round相等。
- `modelUniqueId`使用random non-negative signed-63-bit integer，並對目前active
  mappings做collision retry；不借用final-model catalog version，也不宣稱跨restart
  全域唯一。

### 4.3 Per-edge feature negotiation

每個parent建立direct-child subscription時：

1. request的`suppFeats` offer包含feature 3；
2. child Go route將offer交給PyMTLF；只有protocol execution prerequisites成立且request
   不含unsupported retained instruction時，PyMTLF accepted representation才包含
   feature 3；
3. parent只根據該Create response的accepted features決定這一條edge，不能沿用自己與
   上游edge的結果；
4. 若feature 3未被接受，parent對已建立resource執行best-effort DELETE，candidate標為
   `FAILED`並使用`FEATURE_NOT_SUPPORTED` cause；
5. 這項failure進入本node readiness policy與逐級topology report。

Create成功但feature不符時，resource cleanup response完成後才能回填final candidate
state。若DELETE失敗，保留cleanup pending evidence，不把resource視為已消失。

### 4.4 Model-free preparation

Protocol preparation request包含建立resource所需的標準fields、`mlCorreId`、
`mLPreFlag: true`與接收node的`x-flTopology`，不附`mLModelInfos`。接收端不得在此階段：

- 下載global model；
- 呼叫ADRF model retrieval；
- 載入或驗證model／preprocessing contract；
- 建立legacy hierarchy assignment plan。

Leaf在preparation階段不讀取MNIST或CIFAR-10 shard；它只保存既有local config所選擇的
workload。Dataset與shard path不從request取得。本scenario不以
`dataAvReq`或`mLModelTrainInfos`表示local filesystem selection，也不啟動traffic
collection／ADRF Data Management retrieval。

第一輪收到global model後，Leaf才讀取deployment掛載的shard、完成profile compatibility，
並保存：

- workload profile與dataset selection；
- model／preprocessing contract identity；
- 實際training sample count。

若第一輪才發現model-dependent requirement不成立，該operation以requirements failure
終止並回報，不改讀其他shard，也不回頭在preparation偷偷抓model。Legacy mode仍保留
現有traffic model-aware preparation behavior。

Branch的preparation負責執行direct-child establishment。純aggregation Branch不保存
local workload selection；只有該node同時被明確配置為local trainer時才需要該state。
Branch只有在candidate pool達到
`minAvailableNodes`且自身service／contract requirements成立後，才能向parent回報ready
subtree；未嘗試的candidate保持`UNCONFIRMED`。

### 4.5 Downstream establishment與topology report

每個Server node只處理自己的direct children：

1. 將received node contract交給Slice 2 resolver與candidate pool；
2. 必要時經containing Go NWDAF執行NRF list-discovery；
3. 取得`next_establishment_intents()`，按priority及deterministic tie-break逐一建立
   downstream preparation subscriptions；
4. 每個intent只在HTTP response與feature check完成後回填candidate state；
5. child後續以topology-only Notify回報自己的subtree；parent驗證identity與correlation
   後attach到該candidate；
6. readiness達標時產生本node的`x-flTopologyReport`，逐級送給direct parent。

Preparation Notify不需要`mLModelInfos`。Report root必須是sending node自己的
`nfInstanceId`；children包含該node實際知道的direct-child status與child reports，不能
由Root直接偽造lower-tier結果。

Root收到每個direct Branch report後，以同一套candidate readiness邏輯建立realized
topology view。Root在required direct-child reports尚未達標前不得建立第一輪ADRF
record或dispatch round。

### 4.6 Mutation serialization與PATCH behavior

- 同一downstream resource同時最多一個Create／PUT／PATCH／DELETE in flight。
- Parent收到上一個HTTP response並以matching candidate revision完成state transition後，
  才能對該resource送下一個mutation。
- Late response只能完成建立它的revision；PATCH或generation reset後的stale completion
  不得覆蓋新state。
- JSON Merge Patch中的`children` array取代upstream-assigned children set；
  locally-discovered provenance不因此被清空。
- `enabled: false`產生DELETE intent。DELETE `200`／`204`或已確認不存在的`404`可完成
  local relationship cleanup；transport failure保留pending／failure state並回報。
- PATCH加入candidate時，先完成resource establishment與realized report；只有進入
  ACTIVE且被選入round的consumer才需要取得global model。
- Policy／strategy／`reportAfter`更新在effective resource revision生效，不能修改已
  frozen的current round cohort；下一輪才使用新contract。

若async side effect在上游PATCH已被Go route接受後失敗，persistent representation不
rollback；PyMTLF以topology report回報realized failure。Wire atomicity仍保證invalid
request不會部分更新resource，runtime establishment failure則是合法accepted request的
execution outcome。

### 4.7 Root global-model ADRF lifecycle

Root每個upper round使用獨立的round distribution owner，不重用durable final-model
publication state。State至少保存：

- `mlCorreId`與upper `roundInd`；
- integer `modelUniqueId`；
- selected ADRF `adrfId`；
- `storeTransId`與normalized `Location`；
- immutable artifact URL、size及whole-artifact repository key；
- effective `allowConsumerList`；
- record state、in-flight dispatch／retry references及cleanup state。

建立順序固定如下：

1. Freeze該upper round的selected Root direct participants及其realized ACTIVE subtree。
2. 將global model payload發布為Root local immutable temporary artifact。
3. 由Root PyMTLF呼叫containing Go NWDAF的ADRF store private route。
4. Go consumer執行標準collection POST。
5. Root驗證`201`、`Location`、response record、`modelUniqueId`、`mlFileAddr`、size及
   `allowConsumerList`與request一致。
6. 保存round mapping後，才對selected Branch resources送PUT／PATCH並帶
   `mLPreFlag:false`、`roundInd`與`mLModelAdrf`。

`allowConsumerList`包含本輪可能直接向ADRF取得Root global model的realized NWDAF
identities：selected Branches，以及這些Branches之下所有可能在本次upper operation的
一個或多個lower rounds被選到的`ACTIVE` descendants。Root不需要預先知道Branch每個
lower round最後選出的精確cohort，但只能使用Branch已回報的realized ACTIVE subtree，
不能從requested tree直接推測，也不能包含`FAILED`、`INACTIVE`或`UNCONFIRMED` nodes。

Topology update新增consumer時，Root必須先透過NWDAF ADRF update route提交完整最新
record並確認`200`／`204`，才可向新增consumer下發既有reference；否則等待下一輪新
record。ADRF array update使用完整replacement representation，不在Root與ADRF之間維護
隱含merge語意。

Cleanup順序：

1. 等待selected Root direct participants回覆terminal result／failure；Branch回覆表示
   它所選lower-tier work已terminal；
2. 若procedure被取消或timeout，先best-effort停止／刪除會繼續取用該record的child
   resources；
3. 確認無local dispatch／retrieval retry仍引用record；
4. 由Root經containing Go NWDAF執行ADRF individual DELETE；
5. 記錄cleanup outcome並釋放Root local temporary artifact。

Process restart若遺失round mapping，active procedure標為invalid並停止後續round；本slice
只做best-effort known-record cleanup，不從ADRF反向猜測ownership。

### 4.8 Recipient model retrieval

Branch／Leaf收到`mLModelAdrf`時：

1. 要求`adrfId`、`storTransId`與matching `modelUniqueId`；
2. `AdrfResolver`必須解析指定`adrfId`，不得改用任意first match；
3. 由各自containing Go NWDAF呼叫標準collection GET proxy；
4. 驗證回傳record的transaction ID、model ID、artifact URL、size與consumer profile；
5. 透過既有workspace安全下載URL，驗證實際bytes符合URL中的whole-artifact key，並
   驗證artifact role、
   `mlCorreId`與該local operation所需round metadata；
6. 同一logical model reference在該operation只取得一次，再由typed loader與trainer共用
   同一份owned artifact。

ADRF store／GET或payload download失敗時不得dispatch local training。Testbed目前只能
驗證Root確實產生及ADRF保存allowlist，不能因為不同NWDAF都成功GET就宣稱ADRF已強制
驗證caller identity。

### 4.9 Lower-tier model與result transport

Branch收到Root global model後：

- 第一個lower-tier round對selected children轉送同一`mLModelAdrf`，不重新store或
  republish Root model；
- Branch本身也以該reference取得model並作為aggregation base；
- 若`reportAfter(round).count > 1`，第一個lower aggregate由Branch發布到自己的
  temporary workspace，下一個lower round以該`mLFileAddr`下發；
- 每個後續lower aggregate覆蓋的是local round input state，不建立ADRF record；
- 最後一份Branch aggregate可使用同一temporary artifact URL向Root回報；
- Leaf local result始終由Leaf temporary workspace以`mLFileAddr`回報Branch；
- Root收到的Branch aggregate也使用`mLFileAddr`，不使用`mLModelAdrf`。

這個transport分界保持ADRF只承擔Root global-model fan-out。由下往上的model update與
區域內Branch aggregate不增加ADRF lifecycle。

### 4.10 Retained-result capability gate

Create、PUT或PATCH只要在operation scope帶有`x-retainedResultReq`或nested
`retainedResultReq`，PyMTLF在任何candidate pool、resource mutation或side effect之前
回`403 ML_MODEL_TRAINING_REQS_NOT_MET`。Error應指出retained-result execution尚未支援。

不允許：

- 忽略欄位後繼續建立resource；
- 改成使用request中的global model重新訓練；
- 建立空lookup state；
- 部分套用同一PATCH的topology變更。

`x-retainedResultStatus`仍可在wire/report parser保存；本slice不產生`FOUND`或
`NOT_FOUND` runtime outcome。

---

## 5. Directional end-to-end flow

### 5.1 Preparation

```text
Root PyMTLF
  -> Root Go NWDAF: create Branch preparation subscriptions
  -> Branch Go NWDAF: validate/store route, forward to Branch PyMTLF
  -> Branch PyMTLF: accept feature 3, resolve candidate intents
  -> Branch Go NWDAF: create child preparation subscriptions
  -> Leaf Go NWDAF: validate/store route, forward to Leaf PyMTLF
  -> Leaf PyMTLF: keep configured workload selection, build x-flTopologyReport
  -> Leaf Go NWDAF callback: topology-only Notify
  -> Branch Go NWDAF -> Branch PyMTLF: consume child report
  -> Branch PyMTLF: compose realized subtree
  -> Branch Go NWDAF callback: topology-only Notify
  -> Root Go NWDAF -> Root PyMTLF: consume Branch report and evaluate readiness
```

每個箭頭只傳遞sender當下確實擁有的資料。Root不直接建立Leaf resource；Branch不假設
Root知道其locally discovered children；Go route不決定selection policy。

### 5.2 Normal round

```text
Root PyMTLF
  -> Root local temporary artifact
  -> Root Go NWDAF -> ADRF: POST round record + allowConsumerList
  <- ADRF: 201 + Location + record
  -> Branch subscription: PUT/PATCH roundInd + mLModelAdrf
  -> Branch Go NWDAF -> ADRF: GET record
  -> Branch PyMTLF: download Root artifact once
  -> Leaf subscription: same round input mLModelAdrf
  -> Leaf Go NWDAF -> ADRF: GET record
  -> Leaf PyMTLF: download/train, publish local mLFileAddr
  -> Branch: validate/aggregate, publish Branch local mLFileAddr
  -> Root: validate/aggregate
  -> Root Go NWDAF -> ADRF: DELETE terminal round record
```

若Branch執行多個lower rounds，中間段改為：

```text
Branch aggregate N
  -> Branch temporary mLFileAddr
  -> selected Leaves for lower round N+1
```

不再經過ADRF。

### 5.3 Topology PATCH

```text
Parent receives effective PATCH
  -> candidate pool reconcile
  -> serialize direct-child Create/PATCH/DELETE intent
  -> wait for each HTTP response
  -> update matching revision state
  -> wait for child report when establishment succeeds
  -> emit updated x-flTopologyReport to direct parent
```

若新增node將參與仍在使用的Root model record，Root先更新allowlist再dispatch；若不參與
current frozen cohort，則從下一輪record開始納入。

---

## 6. State ownership與failure behavior

| State／data | Authoritative owner | Transport／consumer | Failure behavior |
| --- | --- | --- | --- |
| Requested subscription representation | Containing Go NWDAF route | Go↔PyMTLF private Model Training API | Invalid request原子拒絕 |
| Effective node contract | Receiving PyMTLF resource | Candidate pool／trainer／aggregator | Unknown executor回403 |
| Direct-child candidate lifecycle | Local Server PyMTLF candidate pool | Topology report | Stale completion依revision丟棄 |
| Direct-child resource URI | Parent PyMTLF `FLServerEngine` participant state | Subsequent PUT／PATCH／DELETE | Missing URI不得猜測 |
| Hierarchy-wide `mlCorreId` | Root PyMTLF procedure | 每條subscription／Notify | Mismatch callback拒絕 |
| Per-edge accepted feature | Parent Go route及parent PyMTLF participant state | Edge capability gate | 未接受則DELETE並FAILED |
| Configured local workload selection | 實際執行local training的PyMTLF Client resource | First及subsequent local rounds | Generation reset即失效；deployment-mounted shard本身不刪除 |
| Realized subtree report | Reporting PyMTLF node | Notify逐級傳遞 | Unknown values保存但不推測成功 |
| Root round ADRF mapping | Root PyMTLF round distribution owner | Root dispatch／update／cleanup | Restart遺失即procedure invalid |
| ADRF record | Selected ADRF | Consumers經各自Go proxy GET | Store／GET／update failure gate training |
| Root global artifact | Root temporary workspace | ADRF record `mlFileAddr` | Record cleanup後再release |
| Branch／Leaf result artifact | Producing node temporary workspace | Direct parent GET | Parent terminal／cleanup後release |

所有external HTTP在owner lock之外執行；state change以revision／generation fence回填。
Shutdown與containing-NWDAF generation reset必須先阻止新dispatch，再cancel local work、
best-effort cleanup known resources，最後使procedure terminal。

---

## 7. Exact-file implementation plan

### 7.1 `NWDAF/` production files

- `internal/mtlf/api_adrf_mlmodel.go`
  - 新增PyMTLF可呼叫的individual record PUT／DELETE private routes；
  - 驗證target API root、`storeTransId`與request body；
  - 只負責transport-shaped validation，不解讀hierarchy policy。
- `internal/mtlf/processor/processor.go`
  - 暴露update／delete processor methods並保持standard ProblemDetails mapping。
- `internal/sbi/consumer/consumer.go`
  - 延伸ADRF consumer interface，加入individual update／delete operations。
- `internal/sbi/consumer/adrf_service.go`
  - 使用TS 29.575 individual resource path送出PUT／DELETE；
  - 接受規格允許的`200`／`204`，保留非成功response body供error mapping；
  - 不在consumer內建立round ownership或allowlist policy。

若route registration不在上述API檔內，依現有`internal/mtlf` router pattern更新同一owner
檔案。不得讓PyMTLF直接繞過Go NF連ADRF。

### 7.2 `NWDAF/` tests

- `internal/sbi/consumer/adrf_service_test.go`
  - PUT／DELETE method、path、headers、body與target-root boundary；
  - `200`／`204`、standard error及malformed response。
- `internal/sbi/consumer/consumer_adrf_test.go`
  - public consumer delegation與error preservation。
- `internal/mtlf/api_adrf_mlmodel_test.go`或既有同package test
  - private route input validation、response/status propagation及invalid transaction ID。
- 既有`internal/sbi/api_ml_model_training_test.go`與processor tests
  - rerun feature、topology-only Notify、PUT／PATCH atomicity與callback preservation。

若現有test seam可在同一檔案完整證明，不為形式新增空泛test file；但transport test必須
觀察真實HTTP method／path／payload，不能只assert mocked helper被呼叫。

### 7.3 `PyMTLF/` production files

- `src/py_mtlf/config.py`
  - 新增`hierarchy_contract` selector與flat／hierarchical cross-field validation；
  - legacy default保持明確。
- `src/py_mtlf/app.py`
  - 只在Root hierarchical orchestration建立對應legacy或protocol coordinator path；
  - 將existing candidate defaults、resolver、round distribution owner注入正確component；
  - generation reset納入new owner cleanup。
- Slice 4A simplified artifact contract與Slice 4 workload modules
  - 本slice只使用whole-artifact repository key及URL/body check，不恢復component、model、
    weights、dataset或tensor digest；
  - 只注入與呼叫已完成的local workload provider、profile validator與held-out handoff；
    不在protocol integration中重新定義local image loader、model或metric。
- `src/py_mtlf/wire/ml_model_training.py`
  - 原則上不新增candidate schema；只在integration發現既有typed model無法表達已確認
    contract時修正，且必須同步candidate OpenAPI evidence。
- `src/py_mtlf/core/fl_client.py`
  - candidate Create真正協商feature 3；
  - 加入protocol model-free preparation、Slice 4 local workload readiness、first-round
    profile binding、
    `mLModelAdrf` input及retained instruction gate；
  - protocol mode不讀legacy hierarchy bundle；
  - Branch／Leaf output保持temporary `mLFileAddr`。
- `src/py_mtlf/core/fl_server.py`
  - 建立protocol preparation／round requests；
  - 保存per-edge accepted feature與candidate revision；
  - 將topology-only callback導向candidate report consumer；
  - 執行serialized Create／PUT／PATCH／DELETE intents；
  - 允許selected input transport為`mLModelAdrf`或Branch local `mLFileAddr`，但每個
    model entry仍遵守exactly-one contract。
- `src/py_mtlf/core/fl_root.py`
  - protocol Root flow、UUID `mlCorreId`、realized readiness、round cohort freeze、
    ADRF store-before-dispatch與terminal cleanup；
  - legacy flow保留並由selector選擇。
- `src/py_mtlf/core/fl_branch.py`
  - 以candidate intents取代protocol mode的Leaf assignment publication；
  - child report composition、first lower-round Root reference reuse、後續local
    aggregate rounds與upstream result reuse。
- `src/py_mtlf/core/fl_candidate_orchestration.py`
  - 原則上只補integration所需的typed completion／snapshot hook；不得把HTTP放入pool。
- `src/py_mtlf/core/adrf_discovery.py`
  - 支援required exact `adrfId` resolution；不允許model reference解析成其他ADRF。
- `src/py_mtlf/core/publication.py`
  - 只重用wire／HTTP conventions；不得把temporary round record加入durable final-model
    catalog lifecycle。
- 新增`src/py_mtlf/core/fl_round_model_distribution.py`：
  - Root-only per-round ADRF record state、int63 model ID allocation、store response
    validation、allowlist update、reference creation與DELETE cleanup；
  - 檔案名稱可在implementation review依現有module style微調，但owner boundary不可與
    final publication混合。

### 7.4 `PyMTLF/` tests

- `tests/test_runtime_modes.py`：selector default、protocol selection及invalid config。
- `tests/test_fl_client.py`：model-free preparation、local workload freeze、first-round
  profile binding、ADRF input、
  retained 403與legacy regression。
- `tests/test_fl_server.py`：protocol request builder、per-edge feature、serialized
  mutation、topology callback、ADRF／local URL dispatch。
- `tests/test_fl_root.py`：UUID、readiness-before-store、allowlist mapping、round lifecycle、
  store／update／delete failure gate及restart invalidation。
- `tests/test_fl_branch.py`：recursive establishment、report composition、same Root reference、
  multi-lower-round local artifact與upstream reuse。
- `tests/test_adrf_discovery.py`：required ADRF identity、no-match／ambiguous／stale result。
- 新增或擴充round distribution tests：record response validation、int63 collision retry、
  complete allowlist replacement、cleanup ordering與in-flight fence。
- `tests/test_ml_model_training_api.py`及`tests/test_ml_model_training_wire.py`：只補實際
  integration暴露的boundary case，不重複Slice 1已充分覆蓋的parser tests。

### 7.5 `nwdaf-resources/`

- 在`deployments/hierarchical_fl/`保留legacy scenario，另加入protocol mode config與
  runner selection。
- Protocol scenario直接掛載Slice 4 pre-generated per-Client shards與matching initial
  model；run期間不下載dataset，也不啟動UPF／traffic collection path。
- Real-process evidence至少提供：
  - explicit Root→Branch→Leaf success；
  - delegated／hybrid candidate establishment；
  - topology PATCH與disabled-child cleanup；
  - child拒絕feature 3；
  - preparation無model／無ADRF GET；
  - 同一Root ADRF reference由至少兩個不同containing NWDAFs取得；
  - Branch multiple lower rounds使用local artifact；
  - unsupported retained request的403與state unchanged。
- Scenario輸出需保留request／response／callback摘要、participant identities、ADRF
  record lifecycle及artifact key evidence；不能只以最終process exit code代表成功。

### 7.6 `adrf/`

目前只作read-only dependency與real-process component。先用既有tests／scenario證明
POST／GET／PUT／DELETE及record persistence。只有既有resource無法履行TS 29.575
contract時才另提repository change proposal。

### 7.7 `nwdaf-docs/`

- Implementation期間更新本計畫的review checkpoint，不改寫design decision歷史。
- 更新Review Ledger的per-repository findings、verification與remaining gaps。
- Slice 5完成user review後才更新主計畫與Slice Map為implementation complete。
- 若runtime evidence推翻既有candidate schema，先更新design／conformance artifact，
  再修改production code；不能只讓code形成新的隱含規格。

---

## 8. 實作順序

### 8.1 Characterization與transport prerequisites

1. 鎖定各repository revisions及clean worktree。
2. 為NWDAF ADRF update／delete private boundary先寫transport characterization tests。
3. 實作Go consumer、processor及route，完成focused Go tests。
4. 為PyMTLF ADRF exact-resolution與round-record owner建立failure-first tests。
5. 完成store／update／retrieve／delete client與state lifecycle，不接hierarchy flow。

### 8.2 Model-free resource execution

1. 加入Root-only`hierarchy_contract` selector並保持legacy default。
2. 在FLClientEngine加入protocol-mode authority與retained request capability gate。
3. 實作model-free preparation，確認此階段不讀取或處理local dataset。
4. 實作第一輪`mLModelAdrf`取得、model／workload profile binding與single-fetch validation。
5. 驗證legacy preparation及round behavior不變。

### 8.3 Recursive establishment與report

1. 將Slice 2 establishment intents接到FLServerEngine真實Create／DELETE。
2. 實作每條edge feature offer／accepted verification及cleanup。
3. 將topology-only Notify導向candidate pool並組合realized subtree。
4. Root加入readiness gate；Branch加入recursive readiness與report。
5. 接上PUT／PATCH、revision fence與disabled-child cleanup。

### 8.4 Round distribution與execution

1. Root freeze selected cohort及realized consumer set。
2. Root store ADRF round record並只在成功後dispatch。
3. Branch／Leaf以exact ADRF reference取得Root model。
4. Branch將同一reference下發第一個lower round。
5. 接上Leaf local result、Branch aggregation及upstream result。
6. 接上multiple lower rounds的Branch local artifact path。
7. 完成allowlist update與terminal cleanup ordering。

### 8.5 Cross-component closure

1. 跑NWDAF與PyMTLF focused／full tests。
2. 跑protocol real-process explicit、hybrid、PATCH與failure scenarios。
3. 跑legacy HFL regression。
4. 依development policy進行production及test-code review，修正findings後重跑affected
   tests。
5. 更新Review Ledger並保留unstaged diff供user review。

不得先開feature 3再補execution path。Feature acceptance與protocol selector最後接線前，
all required consumers、failure gates及cleanup tests必須已存在。

---

## 9. Conformance responsibility

| Case group | Slice 5責任 | 主要證據 |
| --- | --- | --- |
| `NOT-01` | Topology-only Notify可走完整callback path | Go callback + PyMTLF report integration |
| `FEAT-01` | Parent offer feature 3、child accepted | Create boundary + route state |
| `FEAT-02` | 每條edge獨立協商 | Multi-tier integration test |
| `FEAT-03` | 未接受即cleanup並FAILED | Boundary + coordinator test |
| `FEAT-04` | Legacy／protocol明確分流 | Config + real-process regression |
| `POL-*` | Slice 2 decision進入真實dispatch／aggregation | Coordinator integration tests |
| `PATCH-*` | Replacement semantics、serialized side effects、state report | PUT/PATCH/DELETE boundary tests |
| `SCOPE-01` | Node-local policy／strategy／reportAfter作用於正確local process | Branch／Leaf integration tests |
| `RET-*` runtime | Unsupported instruction明確403且無state mutation | Receiver boundary tests |

Slice 1已完成的shape／wire tests與Slice 2已完成的local policy tests不重寫；Slice 5只補
跨owner、transport、side effect與real-process evidence。

---

## 10. Test design與有效性要求

### 10.1 不可用無效mock替代的行為

- ADRF consumer test必須觀察實際HTTP method、path、query、body與status mapping；
  不只mock consumer method return。
- Model-free preparation test必須證明download／ADRF transport與Slice 4 local dataset
  loader皆未被呼叫；第一輪training test再證明configured shard由真實loader讀取。
- Single-fetch test要由transport request count證明，同一logical model reference只GET
  一次；不能分別mock downloader與loader後推論沒有duplicate GET。
- Feature mismatch test要走Create response的accepted feature，再觀察DELETE及candidate
  state；不能直接呼叫`complete_establishment(success=False)`。
- Topology report test要從child Notify進入parent callback consumer，再觀察composed
  report；不能直接塞入child report object。
- Round aggregation使用Slice 4 real image-classification model／artifact fixtures及真正trainer／aggregator
  path。可mock外部HTTP transport，但不能mock被claim的policy decision或aggregation
  result。
- Real-process scenario必須啟動真實Go NWDAF、PyMTLF、NRF與ADRF，並掛載pre-staged
  local shards；本scenario不需要MongoDB或UPF。若環境
  無法啟動，明列remaining gap，不能以unit test冒充。

### 10.2 主要positive cases

- One Root、two Branches、multiple Leaves的explicit topology完成preparation與一輪training。
- Branch以NRF補充candidate後達到`minAvailableNodes`，未嘗試node維持`UNCONFIRMED`。
- Explicit與local candidates共同形成hybrid selected cohort。
- Parent PATCH停用active child並成功DELETE；下一輪selection不含該child。
- Root global model record包含realized selected identities，兩個不同NWDAFs以相同
  `storTransId`取得相同whole-artifact key。
- Branch `reportAfter(round)>1`時只第一個lower round使用ADRF，其餘使用Branch URL。
- Parent省略某些effective contract值時，child採local default並在report回傳實際值。

### 10.3 主要negative cases

- `mLPreFlag:true`的protocol Create帶legacy assignment authority。
- Candidate Create未接受feature 3。
- Child topology report的root identity、`mlCorreId`或`notifCorreId`不符。
- Root readiness未達卻嘗試store／dispatch model。
- ADRF store非201、Location缺失、response record不一致或allowlist不一致。
- Recipient解析到錯誤ADRF、缺`storTransId`、model ID不符、size不符或下載bytes與
  whole-artifact URL key不符。
- allowlist update未完成就向新增consumer dispatch。
- DELETE／PATCH response late completion試圖覆寫新revision。
- Retained-result instruction與合法topology update同時送入；整個operation 403且state不變。
- Restart遺失active round mapping後仍嘗試繼續dispatch。

### 10.4 Cleanup與failure injection

- Store成功但Root dispatch失敗：刪除record並release local artifact。
- 部分Branches完成、另一Branch timeout：先終止仍可能取用的resource，再cleanup record。
- Child Create成功但feature mismatch：DELETE response後report failure。
- ADRF DELETE失敗：procedure terminal但保留cleanup failure evidence及record identity。
- Containing Go NWDAF generation reset：阻止新operation、cancel work並使active protocol
  procedure失效。

---

## 11. Verification commands

實作完成後依repository分開執行；所有command須在workspace execution policy允許下運行。

### 11.1 `NWDAF/`

```bash
gofmt -w <changed-go-files>
go test ./internal/compat/mlmodeltraining ./internal/context ./internal/mtlf ./internal/sbi/consumer ./internal/sbi/processor
make test
make lint
make build
git diff --check
```

### 11.2 `PyMTLF/`

```bash
.venv/bin/ruff check src tests
.venv/bin/pytest -q \
  tests/test_runtime_modes.py \
  tests/test_adrf_discovery.py \
  tests/test_fl_client.py \
  tests/test_fl_server.py \
  tests/test_fl_root.py \
  tests/test_fl_branch.py \
  tests/test_fl_candidate_orchestration.py \
  tests/test_ml_model_training_api.py \
  tests/test_ml_model_training_wire.py
.venv/bin/pytest -q
git diff --check
```

### 11.3 `nwdaf-resources/`

使用該repository既有hierarchical deployment runner及preflight執行新增protocol
scenarios。Exact command在scenario建立後記錄於Review Ledger；需保留每個process的
revision、config、run ID與evidence output。

### 11.4 `adrf/`

執行既有ML Model Management focused tests，並由real-process scenario驗證實際
POST／GET／PUT／DELETE。若未修改repository，不建立空白commit。

---

## 12. Repository review與commit boundary

實作預期形成三個獨立production change sets：

1. `NWDAF/`：ADRF update／delete private transport及tests。
2. `PyMTLF/`：protocol hierarchy execution、round distribution及tests。
3. `nwdaf-resources/`：protocol real-process configs、runner與evidence checks。

`nwdaf-docs/`只保存plan、review與verification evidence，另外commit。`adrf/`目前不在
change set。各repository必須分別保留unstaged diff供user review，不能因其中一個通過
就宣稱整個slice完成。

Commit proposal前必須提供：

- 每個repository的diff summary與included files；
- focused／full verification結果；
- real-process evidence與未執行項目；
- production code與test code review findings；
- 完整commit split與messages；
- 排除的pre-existing／unrelated changes。

User確認review後仍需另行批准commit；commit approval不等於push approval。

---

## 13. Review checklist

### 13.1 Contract與authority

- [ ] Protocol resource不讀legacy hierarchy metadata。
- [ ] Legacy default及explicit selector可回歸。
- [ ] Hierarchy-wide `mlCorreId`與per-edge resource／callback identity沒有混淆。
- [ ] Feature 3逐edge協商，未接受時不執行candidate contract。
- [ ] Retained instruction在任何state mutation前403。

### 13.2 Preparation與topology

- [ ] Protocol preparation不帶model、不下載model、不查ADRF。
- [ ] Configured local workload在preparation驗證並凍結，model profile checks在第一輪完成。
- [ ] Candidate intent只有在真實HTTP response後才改變relationship state。
- [ ] Child topology report經callback進入parent並逐級組合。
- [ ] Root在realized readiness前不store／dispatch global model。
- [ ] PATCH mutation serialized且late completion有revision fence。

### 13.3 Model transport與lifecycle

- [ ] Root round artifact與final-model catalog publication分開。
- [ ] ADRF store response、Location、record及allowlist完整驗證。
- [ ] Recipient解析exact `adrfId`／`storTransId`並single-fetch payload。
- [ ] First lower round轉送同一Root ADRF reference。
- [ ] Subsequent lower rounds及upward results使用producer temporary `mLFileAddr`。
- [ ] Allowlist update先於新增consumer dispatch。
- [ ] Terminal／failure／reset cleanup順序不會在consumer仍retry時先刪record。

### 13.4 Testing與scope

- [ ] Tests觀察被claim的真實behavior，不只mock helper return。
- [ ] Go與Pythonfocused／full suites通過。
- [ ] Protocol及legacy real-process scenarios皆有evidence。
- [ ] ADRF access-control enforcement沒有被過度宣稱。
- [ ] 未修改NRF／ADRF schema或實作隱含recovery scope。
- [ ] 所有changed docs完成繁體中文語言一致性檢查。

---

## 14. 完成條件

Slice 5只有在以下條件全部成立後才可標為Ready for User Review：

1. NWDAF ADRF update／delete transport、error mapping與tests完成。
2. Protocol selector、model-free preparation及retained capability gate完成。
3. Root→Branch→Leaf逐edgefeature negotiation與recursive establishment完成。
4. Topology-only Notify逐級回到Root，readiness gate使用realized state。
5. Root per-round ADRF store、mapping、allowlist update、recipient retrieval與cleanup完成。
6. Branch first lower-round reference reuse及subsequent local artifact path完成。
7. Explicit、delegated／hybrid、PATCH、feature mismatch與failure gates通過integration tests。
8. Legacy HFL regression沒有被protocol path破壞。
9. 至少一組local real-process protocol scenario使用Slice 4 controlled local shards，並
   提供跨Go／PyMTLF／NRF／ADRF evidence。
10. Production code、test code、boundary conformance與scope review完成，remaining gaps明列。
11. 所有intended changes保持unstaged／uncommitted供user review。

正式testbed validation若尚未執行，應保留為remaining gap；只要本slice定義的local
real-process evidence成立，可進入user review，但不可把testbed列為已驗證。

---

## 15. 實作審查檢查點

目前尚未開始Slice 5 production implementation。本節在實作完成後記錄：

- affected repository revisions；
- change summary與diff statistics；
- focused／full test結果；
- real-process scenario evidence；
- production及test-code review findings；
- remaining gaps；
- proposed commit split與messages；
- user review與commit approval狀態。

在user確認review前，本計畫維持open state，不標為Completed。
