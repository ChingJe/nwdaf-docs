# Hierarchical NWDAF FL Protocol Extension Implementation Slice Map

日期：2026-09-04

狀態：Slice 2 detailed plan Approved for Implementation／active sequence為
Slice 1、2、4、5；Slice 3暫緩

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
  model transport或Slice 4的model-free preparation方向。

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

## 4. Slice 3 — Retained Result State and Lookup（暫緩）

Slice 3保留原編號，避免改寫Slice 1的既有計畫、測試紀錄與conformance case引用；
本階段不建立其detailed plan，也不進入production implementation。

`x-retainedResultReq`與`x-retainedResultStatus`仍保留在candidate schema及Slice 1
完成的wire／validation contract中，但目前不實作：

- latest-completed result index；
- artifact retention handle、保存期限與cleanup；
- outstanding lookup state及`FOUND`／`NOT_FOUND`／`FAILED` outcome producer；
- replacement Branch沿用舊Leaf／Branch計算結果。

Production receiver若收到 retained-result instruction，必須回覆明確的requirements／
capability rejection，不得靜默忽略，也不得改成使用request中其他model資訊重新訓練。
未來若重新啟用此slice，需先重新確認artifact owner、retention期限、procedure
correlation與cleanup contract。

---

## 5. Slice 4 — Protocol-driven Hierarchy Integration

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
- FL Server preparation subscription builder 送出標準必要／training requirement fields、
  `mLPreFlag: true` 與 topology contract，但不附 `mLModelInfos`；Branch／Leaf
  preparation 不下載或驗證 model artifact。
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

## 6. Slice 5 — Migration and Regression Closure

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
- 保留 model／round／aggregate／validation artifact與 provenance。
- 更新 fixtures、real-process scenarios與操作文件。
- 執行 flat、distributed FL、legacy migration checkpoint與 protocol HFL regression；
  legacy path是否最終刪除依 Slice 4 review結果提出明確 commit proposal。

### 驗收條件

- 每個 protocol execution 只以 message contract 作為 hierarchy orchestration
  authority；legacy bundle 不參與該 execution。
- 不存在 bundle與 message同時控制 topology、policy、strategy或 report status的路徑。
- Go／PyMTLF full tests、target builds與 selected real-process scenarios通過。
- 未執行的 testbed／external validation明確標成 remaining gap，不以 local tests代替。

---

## 7. 執行順序

```text
Slice 1: wire／resource lifecycle
  -> Slice 2: policy／candidate execution
  -> Slice 4: protocol-driven E2E integration
  -> Slice 5: migration closure

Slice 3: retained-result state（暫緩，不在目前active dependency chain）
```

Slice 3的編號與原始邊界只為保留追溯性；目前完成Slice 2後直接準備Slice 4。各active
slice仍依workspace review規則逐一完成、驗證與交付，不同時累積成一個大型
working-tree diff。

Slice 1已完成，Slice 2 detailed plan已確認。現在的production implementation work
unit為Slice 2。
