# Hierarchical NWDAF FL Protocol Extension Implementation Slice Map

日期：2026-09-07

狀態：Slice 1、2、3、4A、4、5、6 Committed；Slice 7 plan review confirmed／commit pending；
Formal Testbed Validation Pending；completed sequence為Slice 1、2、4A、4、5、6、3

相關文件：

- [Protocol Extension Implementation Plan](./Hierarchical%20NWDAF%20FL%20Protocol%20Extension%20Implementation%20Plan.md)
- [Protocol Implementation Current-State Inventory](./Protocol%20Implementation%20Current-State%20Inventory.md)
- [Protocol Resource Lifecycle and Wire Integration Mapping](./Protocol%20Resource%20Lifecycle%20and%20Wire%20Integration%20Mapping.md)
- [Protocol Conformance Case Ownership Mapping](./Protocol%20Conformance%20Case%20Ownership%20Mapping.md)

---

## 1. 拆分原則

- 每個 slice 只承擔可獨立 review 與驗證的 owner boundary。
- Feature 3 只在本階段承諾的 topology／policy／strategy／report behavior 可執行後
  才對 production flow宣告成功；保留但未採用的 retained-result instruction 必須
  明確拒絕，不得被靜默忽略。
- Legacy model-bundle HFL在 protocol-driven E2E成立前保持可回歸。
- 同一 execution只能使用 legacy bundle或 protocol contract其中一個 authority。
- 每個 slice完成後先保留 unstaged diff供 user review，再另行提出 commit proposal。

---

## 2. Slice 1 — Wire Contract and Resource Lifecycle Foundation

### 行為

建立 Go／Python typed candidate contract，完成 Create／PUT／PATCH／DELETE／Notify 的
field preservation、cross-field validation、persistent／operation-scoped state分離與
per-resource feature state。

### 涉及的 repositories

- `NWDAF/`
- `PyMTLF/`
- `nwdaf-docs/`：更新 conformance／review evidence

### 納入範圍

- 延伸 `internal/compat/mlmodeltraining`，不新增 generated OpenAPI module。
- PyMTLF wire models改為 explicit candidate fields。
- Go accepted／backend representation在 local／peer、`200`／`204` 下保持一致。
- PATCH完整 subtree replacement；一次性 retained instructions不進 persistent
  representation。
- Candidate request／Notify validation與 `ProblemDetails.invalidParams`。
- 一般 training result保留 expected-round equality；retained `FOUND`只驗證其 local
  `roundInd`與 lookup／artifact correlation，不與 replacement resource expected round
  比較。
- Route保存 offered／negotiated feature state，後續 operation可 gate。
- Feature尚未連到 execution consumer時，receiver明確不接受為可執行 hierarchical
  resource；不得靜默忽略 candidate fields。

### 驗收條件

- Conformance `REQ-*`、`TOP-01`–`TOP-09`、`NOT-02`–`NOT-07`、`NOT-09`、
  `RET-01`–`RET-04` 的 wire／receiver部分通過。
- `PATCH-01`–`PATCH-03`、`NOT-08`、`RET-05`–`RET-08` 所需的 Go
  effective-representation atomicity、typed transport、operation gate與 route
  serialization prerequisites通過；retained procedure／state consumer暫不納入active
  slices。
- Create／PUT／PATCH response與 stored representation round-trip全部 candidate fields。
- Operation-scoped fields在 destination執行入口可見，但 accepted representation中
  不存在。
- 不含 candidate fields的既有 ML Model Training tests維持通過。

### 延後項目

- Topology dispatch、policy execution與feature success E2E。
- Retained-result lookup runtime不排入目前active slices。

---

## 3. Slice 2 — Candidate Pool, Policy and Local Contract Execution

### 行為

在 PyMTLF 建立不依賴 legacy assignment bundle 的 local orchestration primitives，讓
node能解析 explicit subtree、補充 delegated candidates、套用 policy／strategy／
`reportAfter`，並產生 realized status snapshot。

### 涉及的 repositories

- `PyMTLF/`
- `nwdaf-docs/`：更新 owner／review evidence

### 納入範圍

- Candidate pool保存 `upstream-assigned`／`locally-discovered` provenance、enabled、
  priority與 relationship state。
- Hierarchy resolver新增 list-discovery，重用現有 Go internal NRF proxy與標準
  training/profile criteria。
- `minAvailableNodes`、`fractionTrain`、`minTrainNodes`、`acceptFailures`、
  `minCompletionRate`。
- Explicit／delegated hybrid selection與 priority ordering。
- Disabled-child排除、downstream DELETE intent與 `INACTIVE` status production。
- FedProx／sample-weighted strategy resolution與 node-local `reportAfter`。
- 重用現有 Leaf local training、FedProx計算與各層 aggregation engine；本 slice只將
  protocol strategy／policy／`reportAfter`接到既有 execution owner，並新增 selection、
  readiness與 completion gate，不重寫 FL algorithm。
- Unknown forward-compatible values可保存，但無 known executor時拒絕明確 contract。
- 修正legacy hierarchy assignment preparation對同一URL的duplicate GET，以單次
  transport在同一份bytes上完成typed validation與plan-owned adoption；不改變其他
  model transport或Slice 5的model-free preparation方向。

### 驗收條件

- Conformance `POL-*`、`PATCH-01`–`PATCH-03`、`SCOPE-01` 與 `NOT-08` 的
  PyMTLF coordinator／execution-consumer部分通過 deterministic unit tests；其 Go
  wire／route prerequisites由 Slice 1驗證。
- 未達 readiness或 completion threshold不得 dispatch／aggregate。
- 既有 local training與 aggregation regression維持相同 model／result semantics；
  protocol-driven input不得建立第二套 trainer或 aggregator。
- Local candidate pool不因 upstream array replacement被清空；prohibited identity不會
  透過 discovery靜默加入。
- Branch與Leaf每個logical hierarchy assignment各只發出一次HTTP GET，且strict
  digest／role／recipient／plan validation、artifact ownership與cleanup regression仍通過。
- Slice尚不宣稱跨 NWDAF protocol E2E完成。

### 延後項目

- Root／Branch message wiring、feature success negotiation與 real-process evidence。

---

## 4. Slice 3 — Branch Replacement without Retained-result Recovery

原Slice 3的retained-result runtime目標已取消並維持暫緩；本編號重新用於完成老師提出的
mid-training Branch replacement情境，但明確不取回舊結果。

### 行為

一個已admitted direct Branch在training round中因transport failure、termination或
response deadline timeout失效時，Root從該Branch group的direct-child candidates
選出新Branch，經fresh NRF exact-ID resolve後重建同一Leaf candidate set。Root先按
topology root policy判斷當輪completion；符合threshold便只聚合成功Branches，否則才拒絕
attempt。剩餘active Branches符合readiness時可在replacement期間繼續training，new Branch
只加入尚未dispatch的下一輪。

### 涉及的 repositories

- `PyMTLF/`
- `NWDAF/`：新增backend-initiated inbound training-route retirement internal lifecycle
  operation
- `nwdaf-resources/`
- `nwdaf-docs/`：更新plan與review evidence
- `nrf/`、`adrf/`：runtime dependency，預設read-only

### 納入範圍

- topology root policy、`branch_groups -> branches／policy／leaves` static config、Branch
  與Leaf priority，以及global identity validation；Leaf set每組只宣告一次，不以重複
  subtree比對推導group。
- Root選Branch與Branch選Leaf共用direct-child candidate pool／policy semantics；本slice
  只實作及驗證Branch replacement，Leaf replacement維持延後。
- Initial Branch selection與mid-training replacement使用同一group candidate pool；獨立
  initial-preparation failure injection scenario不列為本slice必要evidence。
- 可恢復direct Branch availability failure和non-recoverable validation／aggregation／
  ADRF／Root internal failure分類。
- Per-group `BRANCH_REPLACING` progress、bounded candidate loop與fresh exact discovery；
  request可在其他active representatives符合Root policy時繼續round。
- Failed participant與舊`notifCorreId` local retirement；remote DELETE為best effort且留下
  cleanup evidence。
- Same `mlCorreId`、new per-edge resource identity與same subtree的model-free preparation。
- Leaf same-procedure rebind、舊resource／pending callback fencing與experiment lifecycle。
- Leaf PyMTLF依backend resource identity／generation要求containing Go NWDAF淘汰舊inbound
  public route；operation idempotent且不遞迴呼叫backend DELETE。
- Root configured readiness／selection／completion execution、accepted degraded aggregate、
  rejected-attempt cleanup、higher `roundInd`與last committed model continuation。
- 三區域、四個Branch候選與多Leaves的real-process process-termination scenario。

### 驗收條件

- 有candidate時，單一mid-training Branch failure不立即終止Root request。
- Accepted degraded round只聚合successful results並增加`completedRounds`；rejected
  attempt不產生aggregate，下一attempt的`roundInd`嚴格增加。
- Replacement使用前通過fresh NRF exact-ID validation，以same `mlCorreId`建立新
  subscriptions，並使原Leaves安全切到新upper edge。
- 新ADRF allowlist納入replacement並排除failed Branch，terminal record count回到零。
- Candidate exhaustion依Root readiness決定degraded continuation或bounded terminal
  cleanup；simultaneous Branch failures、non-recoverable errors與shutdown維持terminal。
- Request／Notify不使用retained-result fields；既有unsupported execution gate維持通過。
- Existing hierarchy與distributed／flat regressions維持通過。

### 延後項目

- Retained-result index、lookup、artifact retention與舊計算結果接續。
- Leaf replacement、multi-Branch simultaneous recovery、dynamic NRF list discovery、Root
  restart recovery與一般化re-parent authorization。
- 正式multi-host testbed performance experiment。

詳細內容見
[Slice 3 Detailed Plan](./slices/Slice%203%20Branch%20Replacement%20without%20Retained-result%20Recovery%20Detailed%20Plan.md)。

---

## 5. Slice 4A — Digest Simplification and Contract Cleanup

### 行為

在controlled workload與protocol E2E integration前簡化PyMTLF既有hash contract。只保留
完整壓縮artifact bytes的SHA-256 repository key及URL/body verification，移除bundle
component、model／weights、scope／dataset／tensor、Notify body、topology與
training-data collection的content digest。

### 涉及的 repositories

- `PyMTLF/`
- `nwdaf-docs/`：更新plan與review evidence
- `nwdaf-resources/`：只有fixtures或scenario仍依賴已移除欄位時才納入
- `NWDAF/`：預設read-only；只有private artifact header contract實際需要同步時才納入

### 納入範圍

- 保留`ArtifactMetadata.key`與artifact URL中的whole-artifact SHA-256；下載者只將實際
  bytes與URL key比較。
- 移除`X-Artifact-SHA256` duplicate header requirement與`file_digests`。
- 移除model／preprocessing／weights lineage digests，改用artifact key、typed model／
  workload identity、process／round state及state-dict compatibility。
- 移除scope、dataset、tensor、topology、request／profile／collection與record content
  hashes，改用現有明確identity與typed state。
- FL Notify改為resource／stage state idempotency：第一個terminal outcome生效，後續同
  stage terminal retry不再套用。
- 將persisted training-data ledger收斂為單一現行格式，更新fixtures與flat／distributed／
  legacy HFL regression；舊experimental state直接重建，不保留migration reader。

### 驗收條件

- Production code只有whole-artifact repository key及URL/body verification仍使用
  cryptographic hash。
- 新產生的bundle、round／result metadata、training evidence與local state不包含其他
  digest contract。
- Local shard不需要manifest／hash；既有FL與training-data collection流程可回歸。
- Slice 4與Slice 5不再建立於已移除digest之上。

詳細內容見
[Slice 4A Detailed Plan](./slices/Slice%204A%20Digest%20Simplification%20and%20Contract%20Cleanup%20Detailed%20Plan.md)。

---

## 6. Slice 4 — Controlled Local Training Workload

### 行為

在protocol wiring前建立known-workload boundary：既有`UE_COMMUNICATION`可繼續走
標準`consumer_subscription`資料路徑；controlled experiment則由local config選擇MNIST
或CIFAR-10 shard及相應image-classification preprocessing。既有FL Server重用
sample-weighted aggregation，final global model再由獨立held-out test set計算accuracy。

### 涉及的 repositories

- `PyMTLF/`
- `nwdaf-docs/`：更新plan與review evidence

`NWDAF/`是read-only runtime dependency；本slice不改protocol、NRF、ADRF或Go SBI。

### 納入範圍

- `ue_communication_forecasting`與`image_classification` known workload profiles。
- `consumer_subscription`、`private_api`與`local` data-source選擇及合法組合validation。
- 修正seed import tool仍產生Slice 4A已移除private version／digest欄位的carry-in defect，
  並使其重用profile-specific artifact component contract。
- MNIST／CIFAR-10 local loader；local config只保存dataset及shard path。
- MNIST／CIFAR-10 controlled initial model bundles經現行artifact／trusted-loader boundary
  載入，但不註冊成標準NWDAF analytics model。
- 共用config template與per-Client read-only mount mapping；不執行per-instance dataset
  preparation。
- Workspace-local raw cache只在開發／實驗準備階段一次性轉成smoke `.npz`；runtime不
  下載dataset，也不讀取IDX或Parquet。
- Dataset-specific shape／normalization、cross-entropy與accuracy。
- Profile-specific model bundle／FL artifact contract；traffic profile保持原有語意。
- Existing FedProx local penalty與sample-weighted state-dict aggregation重用。
- Branch-only aggregation不要求local dataset。
- 獨立held-out evaluator與不依賴UPF／ADRF Data Management／runtime下載的local smoke。

### 驗收條件

- 真實tensor／loss／optimizer／aggregation path完成至少兩個Clients的controlled FL smoke。
- Image-classification bundle不需要假的`scaler.pkl`，UE Communication bundle contract與
  `consumer_subscription` regression保持通過。
- Dataset與shard path只來自local config，不進入`Nnwdaf_MLModelTraining` message；
  不要求manifest或digest設定。
- Final model可在獨立held-out test set產生可追溯accuracy evidence。

### 延後項目

- Protocol resource wiring、feature 3、topology Notify與ADRF global-model distribution。
- 正式Flat／HFL participant-scale experiment與non-IID維度。

---

## 7. Slice 5 — Protocol-driven Hierarchy Integration

### 行為

把前兩個active slices接入現有 Root／Intermediate／Client production flow，讓 protocol mode
真正以 `x-flTopology`取代 assignment bundle，以 `x-flTopologyReport`取代
preparation-result bundle，並在每條 edge完成 feature negotiation。

### 涉及的 repositories

- `NWDAF/`
- `PyMTLF/`
- `nwdaf-resources/`
- `nwdaf-docs/`：更新 E2E與 review evidence

`adrf/`是本 slice的 runtime dependency與 real-process evidence component；依目前
production trace不預期修改其 repository。若 focused evidence證明既有 store／GET／
DELETE behavior無法支援本 flow，需先更新 slice boundary再修改。

### 納入範圍

- Root PyMTLF產生 UUID hierarchy-wide `mlCorreId`與每個 direct target subtree。
- FL Server preparation subscription builder 送出標準必要fields、
  `mLPreFlag: true` 與 topology contract，但不附 `mLModelInfos`；Branch／Leaf
  preparation 不下載或驗證 model artifact，也不讀取local dataset；真正開始local
  training時才由Slice 4 loader讀取deployment掛載的shard。
- Intermediate逐級建立 model-free downstream preparation subscriptions，並回傳 realized
  report；Root 只有在 report 滿足 topology readiness 後才開始 model distribution。
- Root 每輪發布 model-payload immutable temporary global-model artifact，依 realized
  topology 產生 `allowConsumerList`，經 containing Go NWDAF 建立 ADRF record，驗證
  `201`／`Location`／response 並保存 round-to-record／allowed-consumer mapping；每輪
  使用不重複的 `modelUniqueId` 與 `storTransId`。
- Root 以第一輪／後續 PUT／PATCH 下發 `mLPreFlag: false`、`roundInd` 與 global model
  entry；該 entry 使用 matching `modelUniqueId` 與包含 `adrfId`／`storTransId` 的
  `mLModelAdrf`。上行 local／aggregate result 使用產生節點的暫存 `mLFileAddr`。
- Branch／Leaf依 `adrfId`與 `storTransId`經各自 containing Go NWDAF執行 standard
  collection GET，驗證 record後下載其 `mLFileAddr`；Intermediate只轉傳同一
  `mLModelAdrf` 來啟動收到 Root global model 後的 lower-tier work。
- Branch 完成 domain aggregation 後，若在下一次 upstream update 前繼續 lower-tier
  round，則將 aggregate 發布在 Branch 自己的暫存 workspace，並以 `mLFileAddr` 下發
  給 selected children；該下行路徑不使用 ADRF。同一暫存 aggregate 可向 direct
  parent 回報。
- Topology update 若加入新的 Root global-model consumer，Root 必須先更新現有 record 的
  `allowConsumerList`，或等下一輪 record 納入該 identity，再下發 model reference。
- Testbed store request 仍提供 `allowConsumerList`；目前 plain-HTTP ADRF 沒有 caller
  authorization enforcement，因此只能驗證清單產生與保存，不宣稱 access-control
  enforcement 已完成。
- Root在該輪 selected subtree terminal且無 in-flight retry，或 procedure terminal後
  刪除 ADRF record；restart無法恢復 mapping時使 procedure失效。
- Protocol mode支援 explicit、delegated與 hybrid topology。
- Normal round沿用 standard `roundInd`／model result，不重送 topology；global model
  的 ADRF reference逐級下發，Leaf local result與 Branch aggregate不進 ADRF。
- PUT／PATCH topology update與disabled-child cleanup接入真實 route；replacement Branch
  只重建新的downstream resources，不查詢或沿用舊路徑未送達的result。
- Create／PUT／PATCH若帶有`x-retainedResultReq`或nested `retainedResultReq`，PyMTLF
  execution capability gate回覆明確的`403` requirements error；resource與topology
  state不得因該operation部分更新。
- Feature 3逐 edge negotiation；必要 feature未接受時清除 resource並回報 failure。
- Legacy／protocol execution selector由 Root orchestration明確控制。
- Slice 4的controlled local image workload是主要protocol E2E資料路徑；不把
  `dataAvReq`誤寫成MNIST／CIFAR-10 local filesystem selection contract。

### 驗收條件

- Conformance `NOT-01`、`FEAT-01`–`FEAT-04` 與前述 procedure cases完成跨 component
  integration tests。
- Unsupported retained-result instruction的`403`與resource atomicity由boundary test
  證明，不以欄位被parse或丟棄視為成功。
- Real-process evidence至少涵蓋 explicit topology、delegated／hybrid selection、
  topology PATCH、topology-only Notify與feature mismatch。
- Preparation evidence 證明各層 Create 不帶 `mLModelInfos`、不觸發 ADRF GET，且 Root
  在 realized topology ready 前不建立該輪 model record。
- ADRF evidence涵蓋 realized-topology-to-`allowConsumerList` mapping、一次 store、至少
  兩個不同 containing NWDAFs 以同一 reference retrieval、無 Intermediate republish、
  store／retrieval failure gate與 terminal cleanup；另記錄 testbed 尚未驗證 caller
  authorization enforcement。
- Multi-lower-round evidence 證明 Branch domain aggregate 以 Branch 暫存
  `mLFileAddr` 下發 selected children，不建立 ADRF record，且可用同一 artifact 向
  direct parent 回報。
- 同一 protocol execution不讀取 assignment／preparation-result bundle metadata。
- Legacy mode仍可跑既有 static HFL regression。

### 延後項目

- 完整 Branch failure detector、replacement selector與fencing。
- Retained-result persistence、lookup與acceptance policy。

---

## 8. Slice 6 — Migration and Regression Closure

### 行為

完成 protocol path的回歸與移除舊 hierarchy-only runtime payload，確保 artifact只保存
model／result／evidence，不再是第二套 orchestration source。

### 涉及的 repositories

- `PyMTLF/`
- `nwdaf-resources/`
- `nwdaf-docs/`
- `NWDAF/`：只有 closure test或已確認的 dead compatibility code需要修改時才納入

### 納入範圍

- 移除 `BranchAssignmentMetadata`、`LeafAssignmentMetadata`、
  `PreparationResultMetadata` 與 hierarchy-only artifact roles 的 active runtime path。
- 刪除 Root preparation-result download／validation與 Branch Leaf-assignment republish。
- 保留 model／round／aggregate／validation artifact與必要的明確process／round／sample
  provenance；不得恢復Slice 4A已移除的digest contract。
- 更新 fixtures、real-process scenarios與操作文件。
- 改碼前執行一次legacy migration checkpoint；cutover後執行flat、distributed FL與
  protocol HFL regression。Slice 5已完成review並確認protocol path可取代legacy
  hierarchy runtime，因此本slice直接移除舊path，不保留雙模式selector。

### 驗收條件

- 每個 protocol execution 只以 message contract 作為 hierarchy orchestration
  authority；legacy bundle 不參與該 execution。
- 不存在 bundle與 message同時控制 topology、policy、strategy或 report status的路徑。
- Go／PyMTLF full tests、target builds與 selected real-process scenarios通過。
- 未執行的 testbed／external validation明確標成 remaining gap，不以 local tests代替。

---

## 9. Slice 7 — Experiment Metrics and Event Recording

### 行為

在不改變Model Training protocol與training decision的前提下，讓各PyMTLF以每次
procedure的UUIDv4 `mlCorreId`建立node-local JSONL record。Root保存initial與每個
accepted global aggregate的validation loss／accuracy、每個round attempt的direct
Branch cohort，以及Branch failure detection與replacement ready時間；Branch／Leaf可
透過local config選擇保存domain／local model validation。Test controller另保存實際
停止Branch processes的時間，canonical runner再收集所有raw evidence。

### 涉及的 repositories

- `PyMTLF/`
- `nwdaf-resources/`
- `nwdaf-docs/`：更新slice與review evidence

`NWDAF/`、`adrf/`與`nrf/`預設read-only；本slice不修改protocol schema或新增
Go-side experiment API。

### 納入範圍

- Optional node-local experiment config、independent validation file與record directory。
- Mean cross-entropy validation loss與accuracy evaluator。
- `<record-directory>/<mlCorreId>/observations.jsonl` locked append與flush。
- Root initial／accepted-global evaluations及每-attempt selected／successful／failed
  participant records。
- Root Branch failure-detected／replacement-ready records，直接使用failed與replacement
  `nfInstanceId`，不新增無runtime來源的group identifier。
- Optional Branch domain與Leaf local validation records。
- Controller fault-injection event、per-node record collection及canonical local
  branch-replacement evidence。

### 驗收條件

- 每次Root request使用自身`mlCorreId`建立獨立資料夾，terminal cleanup不刪除records。
- Root curve包含initial point及所有accepted global rounds；rejected attempts沒有虛構的
  model evaluation。
- Structured outcome可直接辨識normal、degraded與replacement恢復後的participant cohort。
- Fault injection、Root failure detection與replacement ready由各自authoritative producer
  記錄，timestamp可在同步時鐘的testbed上對齊。
- Config不存在時不改變既有FL行為；配置時invalid dataset／record path明確失敗。
- Canonical runner不再依一般文字log或單次external final accuracy作為主要learning
  evidence。

### 延後項目

- 自動畫圖、集中式metrics服務與Prometheus。
- Communication／resource／latency instrumentation。
- Flat對照、dataset partition study與正式multi-host statistical evaluation。

詳細內容見
[Slice 7 Detailed Plan](./slices/Slice%207%20Experiment%20Metrics%20and%20Event%20Recording%20Detailed%20Plan.md)。

---

## 10. 執行順序

```text
Slice 1: wire／resource lifecycle
  -> Slice 2: policy／candidate execution
  -> Slice 4A: digest simplification／contract cleanup
  -> Slice 4: controlled local training workload
  -> Slice 5: protocol-driven E2E integration
  -> Slice 6: migration closure
  -> Slice 3: Branch replacement without retained-result recovery
  -> Slice 7: experiment metrics／event recording
```

Slice 3編號沿用原本暫緩的work unit，但目標已由retained-result lookup改為不使用舊結果
的Branch replacement；先前已commit的Slice編號與歷史紀錄不重寫。Slice 4A編號表示它是
protocol integration前新增的supporting work，不代表數字順序。

Slice 1、2、3、4A、4、5與6已完成審查、驗證並commit。Slice 3的Branch
replacement、degraded training、Leaf rebind與terminal cleanup已有local real-process
evidence；Slice 7計畫已確認且尚未進入實作，正式multi-host testbed尚未進入。
