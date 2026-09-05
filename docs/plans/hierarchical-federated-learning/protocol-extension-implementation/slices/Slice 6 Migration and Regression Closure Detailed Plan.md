# Slice 6 — Migration and Regression Closure Detailed Plan

日期：2026-09-06

狀態：Draft／Review Pending；已完成現況盤點，尚未進入實作

相關文件：

- [Protocol Extension Implementation Plan](../Hierarchical%20NWDAF%20FL%20Protocol%20Extension%20Implementation%20Plan.md)
- [Protocol Extension Implementation Slice Map](../Protocol%20Extension%20Implementation%20Slice%20Map.md)
- [Protocol Implementation Current-State Inventory](../Protocol%20Implementation%20Current-State%20Inventory.md)
- [Model Bundle Metadata to Protocol Schema Mapping](../Model%20Bundle%20Metadata%20to%20Protocol%20Schema%20Mapping.md)
- [Protocol Extension Implementation Review Ledger](../Protocol%20Extension%20Implementation%20Review%20Ledger.md)
- [Slice 5 Detailed Plan](./Slice%205%20Protocol-driven%20Hierarchy%20Integration%20Detailed%20Plan.md)
- [NWDAF Development Policy](../../../../development_policy.md)

---

## 1. Slice 結果

Slice 5已證明production `Nnwdaf_MLModelTraining` message path可完成model-free
Root→Branch→Leaf preparation、逐edge feature negotiation、recursive topology report、
ADRF global-model distribution、Branch lower-tier aggregation與held-out evaluation。

Slice 6不新增hierarchical FL behavior，而是完成authority cutover：

- `orchestration.mode: hierarchical`只啟動protocol-driven hierarchy；
- topology、policy、strategy與preparation status只來自subscription／PUT／PATCH／Notify；
- model bundle只保存model、round result與必要training／evaluation evidence；
- 移除舊assignment／preparation-result bundle schema、producer、consumer及deployment
  scenario；
- 保留standard flat／distributed FL既有model-based preparation與training behavior；
- 保留`HIERARCHY_AGGREGATE`等真正描述model result的artifact contract。

本slice完成後不再有`model_bundle`／`protocol`雙模式selector，也不保留讀取舊
hierarchy-only payload的compatibility fallback。舊experimental bundle與本機暫存狀態
直接重新產生，不提供migration reader。

---

## 2. 盤點基準與證據

### 2.1 Repository版本

| Repository | 盤點revision | 本slice角色 |
| --- | --- | --- |
| `PyMTLF/` | `554c96de76331065f2c8c0d536c80c657f1fe8ba` | legacy／protocol execution authority與artifact owner；主要implementation target |
| `nwdaf-resources/` | `33729a1880cfff63766fca7f6c18969fc8cb5cdb` | legacy及protocol real-process harness；migration scenario target |
| `NWDAF/` | `256349f3f2d459339bc1f4e33d6b5c8a17e2d1e6` | standard-shaped Model Training與ADRF transport；預設read-only |
| `nwdaf-docs/` | `5b23ce465c1d3bd3735d5ce146f3c20518104e3d`加上本plan diff | plan、slice map與review evidence owner |
| `adrf/` | `905f0599f68fe389bba14ed56db0ef9abeab5ccd` | protocol runner runtime dependency；不修改 |
| `nrf/` | `0dd4024d4ab75b6630e04901968228b9b9718cf5` | discovery runtime dependency；不修改 |

實作開始前必須重新確認上述repositories的HEAD與working tree。若Slice 5 contract或
runner在本plan review期間改變，先更新本文件的精確檔案對應。

### 2.2 已確認的protocol baseline

Slice 5已有下列直接證據：

- Root-only UUID `mlCorreId`與recursive `x-flTopology`；
- Branch／Leaf model-free preparation；
- `x-flTopologyReport`逐級回傳realized subtree；
- explicit、delegated及hybrid candidate selection；
- topology PATCH與disabled-child DELETE；
- Root ADRF store、Branch／Leaf retrieval及terminal DELETE；
- Branch local aggregate以`mLFileAddr`執行後續lower-tier round；
- unsupported retained-result instruction以`403`拒絕；
- controlled MNIST production tensor／trainer／aggregator path與held-out evaluation。

因此舊bundle path已不再是hierarchical execution的必要fallback。正式testbed尚未執行，
但這是integration verification gap，不是保留第二套runtime authority的理由。

### 2.3 free5GC／standard boundary

本slice不改變external SBI path、HTTP method、status code、standard field或Go
handler／processor／consumer責任。`NWDAF/`目前沒有
`BranchAssignmentMetadata`、`LeafAssignmentMetadata`或
`PreparationResultMetadata` reader；舊hierarchy-only payload完全由PyMTLF model
artifact承載。

因此本slice的production change預期集中在PyMTLF internal runtime與deployment
harness。若implementation盤點發現Go存在只服務舊payload的dead compatibility code，
必須先提出直接caller evidence後才可把`NWDAF/`納入修改；不能因為做cleanup而順便改
standard-shaped Model Training transport。

---

## 3. 已固定的migration決策

### 3.1 Hierarchical mode只有一個authority

`OrchestrationSettings.mode == "hierarchical"`直接代表使用Slice 5 protocol path。
移除`hierarchy_contract`欄位及下列值：

```yaml
hierarchy_contract: model_bundle
hierarchy_contract: protocol
```

不將`protocol`設成新default，也不接受後忽略舊欄位。Experimental config使用
`extra="forbid"`規則，因此仍提供`hierarchy_contract`的設定必須明確validation fail，
促使部署設定一次性更新。

### 3.2 不影響standard flat／distributed FL

本slice只移除hierarchy-specific bundle authority。沒有candidate extension fields的
standard Model Training resource仍可在flat／distributed FL中：

- preparation以`mLModelInfos[].mLFileAddr`取得base model；
- Client準備local dataset並回傳既有preparation notification；
- round及final validation沿用原本model／result artifact contract。

不可把「hierarchical mode一律使用protocol」誤作「所有FL preparation都必須
model-free」。

### 3.3 Artifact保留邊界

移除的是orchestration payload：

- `HIERARCHY_ASSIGNMENT`；
- `HIERARCHY_PREPARATION_RESULT`；
- nested `hierarchy_metadata`；
- branch／leaf assignment與preparation-result message types。

保留的是training／result payload：

- `ROUND_INPUT`、`ROUND_LOCAL`、`ROUND_GLOBAL`及`FINAL_MODEL`；
- `HIERARCHY_AGGREGATE` result type；
- process／round／participant／sample-count provenance；
- model compatibility、validation summary與held-out evaluation evidence；
- whole-artifact repository key及URL/body verification。

不得在cleanup過程恢復Slice 4A已移除的component、model、topology或request digest。

### 3.4 Retained-result不隨本slice擴張

`x-retainedResultReq`與`x-retainedResultStatus`繼續存在於candidate wire schema；runtime
仍明確拒絕尚未支援的instruction。Slice 6不加入result retention、lookup、handoff或
replacement recovery。

### 3.5 舊runner不作長期compatibility suite

舊hierarchical real-process runner只在改碼前作一次migration characterization。完成
cutover後移除舊runner與其專屬scenario，不要求它繼續通過。Flat／distributed FL由
現有`deployments/distributed_fl/` regression證明，不藉由保留舊hierarchy bundle
runner來證明。

---

## 4. 現有legacy path盤點

| 階段 | 現有producer／state | 現有consumer | Slice 6處置 |
| --- | --- | --- | --- |
| Root mode selection | `config.py`的`hierarchy_contract`，預設`model_bundle` | `app.py`、`FLRootCoordinator` | 移除selector；hierarchical直接建立protocol coordinator |
| Root assignment | `FLRootCoordinator._run()`發布Branch assignment bundle | Branch `FLClientEngine._run_preparation()` | 移除producer與整段legacy branch |
| Branch assignment metadata | `BranchAssignmentMetadata` | Branch coordinator | 移除schema與tests |
| Leaf assignment republish | `HierarchyArtifactService.republish_leaf_assignment()` | Leaf preparation ingress | 移除publisher、download與admission |
| Preparation result | Branch發布`PreparationResultMetadata` bundle並在Notify回URL | Root下載、驗證並作admission | 改以既有`x-flTopologyReport` collection作唯一來源 |
| Legacy lower process | `FLBranchPreparationCoordinator.dispatch()`／`prepare()` | `FLServerEngine.start_hierarchy_preparation()` | 移除assignment URL path；保留`prepare_protocol()` |
| Legacy round | Branch `execute_round()`依assignment state執行 | FL Client依`hierarchy_assignment`分流 | 移除；保留`execute_protocol_round()` |
| Legacy hierarchical validation | Branch `execute_validation()`及Root `finalize_hierarchy_candidate()` | 舊WAPE hierarchy path | 移除active caller；保留一般flat validation與model/evidence types |
| Workspace special admission | `ValidatedHierarchyArtifact`、`download_hierarchy()`、`admit_assignment()` | legacy Client／Root | 移除；保留generic archive/model download與plan cleanup |
| Artifact discriminator | `ArtifactRole.HIERARCHY_*`及strict projection | legacy publisher／reader | 移除；舊artifact role變成unsupported，而非普通model |
| Deployment selector | protocol builder在legacy config上覆寫`hierarchy_contract` | Root PyMTLF config | 收斂成單一canonical hierarchy builder |
| Deployment runners | `run.py`執行legacy hierarchy；`run_protocol.py`借用其helper | checks／README／static collection wrapper | protocol runner成為canonical runner；抽出通用helper後移除legacy runner |

### 4.1 仍需保留的同名hierarchy元件

下列元件名稱包含`hierarchy`，但不是舊orchestration payload，不應因字串搜尋而刪除：

- static topology planner與Root explicit topology input；
- candidate pool、policy、strategy及report resolution；
- `FLServerEngine.execute_hierarchy_round()`等共用多tier aggregation lifecycle；
- `HierarchyArtifactService.publish_round_input()`與
  `publish_hierarchy_aggregate()`；
- `FLWorkspace.release_plan()`與hierarchy-wide UUID ownership；
- `RoundLocalResultType.HIERARCHY_AGGREGATE`及其sample provenance。

---

## 5. 目標執行流程

### 5.1 Root

1. Root依static topology planner與NRF resolver取得Branch／Leaf targets。
2. `FLRootCoordinator`無selector，直接建立protocol preparation targets。
3. Root等待`x-flTopologyReport`達成readiness。
4. Root建立ADRF global-model record並下發標準round fields。
5. Root收集Branch aggregate，完成held-out handoff與ADRF cleanup。

Root不再發布Branch assignment bundle，也不再下載preparation-result bundle。

### 5.2 Branch

1. Branch從自身subscription的`x-flTopology`取得direct-child contract。
2. Branch透過candidate pool處理explicit／delegated selection。
3. Branch逐級建立model-free subscriptions並回傳realized report。
4. Branch依protocol resource state執行lower rounds與upper aggregate。

Branch不再複製Root model bytes來產生Leaf assignment，也不再產生preparation-result
artifact。

### 5.3 Leaf

Leaf只從message contract取得strategy與`reportAfter`，在真正round開始時才取得model並
讀取local shard。任何帶舊`HIERARCHY_ASSIGNMENT` artifact role的bundle都必須在strict
artifact validation中成為unsupported input，不能被當作普通base model使用。

### 5.4 Cleanup

Protocol resource DELETE、failure與generation reset仍依hierarchy-wide `mlCorreId`釋放
Branch／Leaf resource、workspace與ADRF mapping。移除`hierarchy_assignment` resource
field後，hierarchy ownership由candidate contract、experiment binding及resource
`mlCorreId`判斷；不能因刪除舊artifact object而讓protocol DELETE退回一般
`ML_TRAINING_NOT_COMPLETE`路徑。

---

## 6. 精確檔案實作計畫

### 6.1 `PyMTLF/` production files

| File | 計畫變更 |
| --- | --- |
| `src/py_mtlf/config.py` | 移除`OrchestrationSettings.hierarchy_contract`及其flat cross-field branch；hierarchical mode本身即為protocol hierarchy |
| `src/py_mtlf/app.py` | 不再向Root coordinator注入selector；仍注入Slice 5 `RoundModelDistribution` |
| `src/py_mtlf/core/fl_root.py` | 移除legacy constructor branch、assignment publication、preparation-result download／evaluation、legacy round／finalization path與legacy-only snapshot欄位；`_run()`直接進入protocol hierarchy |
| `src/py_mtlf/core/fl_client.py` | 移除`hierarchy_assignment` state、legacy bundle detection／admission、Branch／Leaf metadata分流及legacy Branch round／validation callback；保留standard non-candidate preparation與candidate protocol path |
| `src/py_mtlf/core/fl_branch.py` | 移除legacy preparation dataclasses、`dispatch()`、`prepare()`、`execute_round()`、`execute_validation()`與assignment-based classify；保留protocol candidate／round／cleanup owner |
| `src/py_mtlf/core/fl_server.py` | 移除`HierarchyPreparationTarget`、legacy `start_hierarchy_preparation()`及preparation outcome中的`assignment_url`；保留flat `_create_preparation()`與protocol preparation／hierarchy-round engine |
| `src/py_mtlf/core/fl_hierarchy.py` | 移除assignment、preparation-result、failure／admission metadata models；保留protocol仍使用的UUID normalization helper |
| `src/py_mtlf/core/fl_hierarchy_artifacts.py` | 移除assignment／result publish及republish helpers；保留round input、hierarchy aggregate與仍有active caller的model/evidence helpers |
| `src/py_mtlf/core/fl_artifacts.py` | 移除兩個hierarchy-only artifact roles、typed contracts及`hierarchy_metadata` projection；保留`HIERARCHY_AGGREGATE`與training/evaluation models |
| `src/py_mtlf/core/fl_workspace.py` | 移除`ValidatedHierarchyArtifact`與special hierarchy download／admission path；保留generic download、ADRF download、publication、reader與plan cleanup |
| `config/fl-server-hierarchy.yaml` | 原地改成現行protocol hierarchy示例；不加`v2`檔名或compatibility欄位，且不再預載只能由legacy path執行的UE Communication hierarchy seed |
| `README.md` | 將hierarchical profile說明改為單一protocol path，說明Root image seed需先import並填入現行config |

不預先改名`FLRootCoordinator`、`FLBranchPreparationCoordinator`、
`HierarchyArtifactService`或`execute_hierarchy_round()`；名稱仍描述有效owner或multi-tier
behavior，改名只會擴大diff而不增加authority清晰度。

### 6.2 `PyMTLF/` tests

| File | 計畫變更 |
| --- | --- |
| `tests/test_config.py` | 移除雙模式default測試；新增hierarchical直接採protocol與舊`hierarchy_contract`被拒絕 |
| `tests/test_fl_root.py` | 刪除legacy assignment／result／WAPE hierarchy cases；fixture改成protocol-only，保留Root lifecycle、failure、ADRF與cleanup cases |
| `tests/test_fl_client.py` | 刪除legacy assignment admission、metadata role routing與legacy Branch validation cases；保留flat preparation及完整protocol Branch／Leaf cases |
| `tests/test_fl_branch.py` | 刪除legacy assignment publication、preparation result、round與validation cases；保留candidate pool、report、multi-lower-round與cleanup cases |
| `tests/test_fl_server.py` | 刪除assignment URL preparation及legacy hierarchy finalization cases；保留flat Server與protocol hierarchy round／collection cases |
| `tests/test_fl_hierarchy.py` | 刪除整份legacy metadata schema tests；UUID normalization coverage搬至discovery／candidate consumer tests後移除檔案 |
| `tests/test_fl_hierarchy_artifacts.py` | 移除assignment／preparation-result publish、download與identity tests；原檔保留round／aggregate／workspace ownership tests |
| `tests/test_fl_artifacts.py` | 移除hierarchy-only role/discriminator cases；增加舊role為unsupported的fail-closed case，保留aggregate／validation provenance |
| `tests/test_health_and_artifact_api.py` | 將只為`HIERARCHY_ASSIGNMENT`建立的artifact serving fixture改用仍支援的artifact role |

測試不能以Mock直接回傳預組好的protocol report來取代關鍵production path。Root／Branch／
Leaf focused tests至少要有一條使用real wire model、candidate resolver及artifact loader的
boundary；real-process runner負責證明跨process transport。

### 6.3 `nwdaf-resources/`

| File／area | 計畫變更 |
| --- | --- |
| `deployments/hierarchical_fl/scripts/support.py` | 將protocol config builder收斂為唯一canonical builder；移入protocol runner仍需的port、repository、NRF／ADRF config與readiness helper；移除legacy builder及`hierarchy_contract`注入 |
| `deployments/hierarchical_fl/scripts/run_protocol.py` | 在移除舊`run.py`後成為canonical `run.py`；不再dynamic import legacy hierarchy runner或藉其取得通用helper |
| `deployments/hierarchical_fl/scripts/run.py` | 移除舊assignment-bundle hierarchy runner；檔名由現行protocol runner接替 |
| `deployments/hierarchical_fl/scripts/run_static_collection_e2e.py` | 移除依賴legacy hierarchical runner的wrapper；standard flat／distributed regression改由`deployments/distributed_fl/`負責 |
| `deployments/hierarchical_fl/scripts/static_collection.py` | 已確認repository內只有legacy runner／wrapper使用；連同legacy static hierarchy scenario移除 |
| `deployments/hierarchical_fl/checks/test_static_collection.py` | 隨已移除scenario刪除，不保留指向不存在runner的測試 |
| `deployments/hierarchical_fl/checks/test_support.py` | 移除legacy runner assertions，更新canonical builder／runner名稱，保留protocol evidence與config tests |
| `deployments/hierarchical_fl/checks/preflight.py` | 只驗證canonical protocol hierarchy assets與runtime dependencies |
| `deployments/hierarchical_fl/README.md` | 移除legacy scenario清單；只記錄canonical protocol runner及其evidence scope |
| `deployments/hierarchical_fl/components.yaml` | 移除protocol scenario不再使用的PyAnLF component，更新implementation revisions與required files |

Repository-wide caller盤點未發現`hierarchical_fl/`以外的current caller，因此不保留
dead entrypoint，也不為舊hierarchy mode保留compatibility branch。

### 6.4 `NWDAF/`、`adrf/`與`nrf/`

- `NWDAF/`預設read-only；執行full tests、lint、build及protocol real-process regression。
- `adrf/`與`nrf/`只作real-process dependency，不修改repository。
- 本slice不新增SBI field、private route、discovery criterion或ADRF lifecycle operation。

### 6.5 `nwdaf-docs/`

- 更新上層implementation plan、slice map、slice index及review ledger狀態；
- 不回寫歷史Slice 1／2／4A／4／5 plan來假裝legacy path從未存在；
- Slice 6 review完成後在本文件補實作檔案、驗證結果與remaining gaps。

---

## 7. 實作順序

### 7.1 遷移前特徵確認

1. 重查四個affected repositories狀態與revision。
2. 在刪除前執行既有`run.py --profile smoke --scenario manual-success`，保存結果作
   migration checkpoint；此結果不成為post-cutover acceptance requirement。
3. 執行現行protocol real-process runner，建立cutover前baseline。

### 7.2 設定與Root authority cutover

1. 先加入`hierarchy_contract`不再被接受及hierarchical constructor需要protocol
   distribution owner的測試。
2. 移除config selector與app wiring。
3. 將Root `_run()`收斂到單一protocol hierarchy path。
4. 移除assignment publication與preparation-result evaluation。

### 7.3 Branch／Client／Server cutover

1. 移除Client resource中的legacy assignment state及ingress分流。
2. 移除Branch legacy preparation、round及validation入口。
3. 移除FL Server assignment URL target與legacy preparation constructor。
4. 以candidate contract、experiment binding及`mlCorreId`維持DELETE／cleanup ownership。
5. 跑flat與protocol focused tests，避免先刪schema導致大量不可判讀failure。

### 7.4 Artifact contract cleanup

1. 移除hierarchy-only metadata models與artifact discriminator。
2. 移除workspace special download／admission methods。
3. 精簡artifact service，只保留有active model／result caller的方法。
4. 增加舊artifact role fail-closed regression。
5. 以`rg`確認production code不存在舊type、role、selector或metadata field。

### 7.5 部署測試工具收尾

1. 將protocol runner用到的通用helper移至canonical support boundary。
2. 將protocol config builders原地收斂為canonical builders。
3. 由現行protocol runner接替hierarchical deployment的`run.py`。
4. 移除legacy及static-collection hierarchy entrypoints與tests。
5. 更新README、preflight、component inventory及summary assertions。

### 7.6 回歸與審查

1. 跑PyMTLF focused及full suites、ruff。
2. 跑NWDAF full tests、lint及build。
3. 跑nwdaf-resources checks。
4. 跑canonical protocol hierarchy real-process scenario。
5. 跑現有distributed／flat regression；若local runtime prerequisite缺失，
   進入decision gate並交由user決定，不得直接將Slice 6標記為完成。
6. Review production code與test code，確認沒有以刪除assertion取代必要behavior proof。

---

## 8. 測試設計

### 8.1 必要positive cases

- hierarchical config不含selector即可建立Root protocol coordinator；
- Root只發布`x-flTopology` preparation，不建立assignment artifact；
- Branch只從message contract建立children並回傳`x-flTopologyReport`；
- Leaf只從node-scoped strategy／`reportAfter`取得local work；
- protocol round仍完成ADRF retrieval、lower-tier aggregation、upper result及cleanup；
- protocol DELETE／failure仍釋放hierarchy-owned resources與workspace；
- standard flat preparation仍可帶base model並完成local data preparation；
- `HIERARCHY_AGGREGATE` artifact仍保留participants與sample-count evidence。

### 8.2 必要negative cases

- config仍提供`hierarchy_contract`時被明確拒絕；
- 舊`HIERARCHY_ASSIGNMENT`及`HIERARCHY_PREPARATION_RESULT` role被strict artifact
  validation拒絕；
- candidate resource未協商feature 3時仍不得執行；
- protocol preparation帶legacy assignment model時仍拒絕ambiguous authority；
- retained-result instruction仍回`403`且不部分更新state；
- 移除legacy assignment state後，in-flight protocol resource DELETE不可誤走一般
  incomplete-training拒絕。

### 8.3 Regression levels

| Level | 必須證明 |
| --- | --- |
| PyMTLF unit／component | Config、Root、Branch、Client、Server、artifact與workspace behavior |
| NWDAF module | Standard-shaped Model Training及ADRF proxy未受cleanup影響 |
| nwdaf-resources checks | Canonical builder、preflight、evidence parser與runner dependency完整 |
| Protocol real-process | Root→Branch→Leaves message authority、ADRF model flow、training與cleanup |
| Flat／distributed real-process | 非candidate Model Training path仍可工作 |
| External testbed | 不在本slice local completion claim內；未執行時列remaining gap |

---

## 9. 驗證指令

### 9.1 `PyMTLF/`

```bash
.venv/bin/pytest -q tests/test_config.py tests/test_fl_root.py \
  tests/test_fl_branch.py tests/test_fl_client.py tests/test_fl_server.py \
  tests/test_fl_artifacts.py tests/test_fl_hierarchy_artifacts.py \
  tests/test_health_and_artifact_api.py
.venv/bin/pytest -q
.venv/bin/ruff check .
git diff --check
```

`tests/test_fl_hierarchy.py`應在UUID normalization coverage搬至discovery／candidate
consumer tests後刪除，不執行空殼測試。

### 9.2 `NWDAF/`

```bash
go test ./...
make lint
make build
git diff --check
```

### 9.3 `nwdaf-resources/`

```bash
../PyMTLF/.venv/bin/pytest -q deployments/hierarchical_fl/checks
../PyMTLF/.venv/bin/ruff check deployments/hierarchical_fl
../PyMTLF/.venv/bin/python deployments/hierarchical_fl/checks/preflight.py
../PyMTLF/.venv/bin/python deployments/hierarchical_fl/scripts/run.py
git diff --check
```

從`nwdaf-resources/`執行時，Python相對路徑依實際workspace位置校正；不得把錯誤工作
目錄造成的失敗誤判成product defect。

### 9.4 Flat／distributed regression

除PyMTLF full suite中的flat flow外，必須執行現有distributed FL local real-process
regression：

```bash
PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/distributed_fl/scripts/run.py
```

此指令從workspace root執行。若local runtime prerequisite確實無法提供，Slice 6不得直接
標成完成；review handoff需列出阻礙、已完成的替代coverage並交由user決定。

---

## 10. 驗收條件

- [ ] `orchestration.mode: hierarchical`只有protocol-driven implementation。
- [ ] Production config及code不存在`hierarchy_contract` selector。
- [ ] Production code不存在`BranchAssignmentMetadata`、`LeafAssignmentMetadata`、
  `PreparationResultMetadata`。
- [ ] Artifact role不存在`HIERARCHY_ASSIGNMENT`與
  `HIERARCHY_PREPARATION_RESULT`。
- [ ] Root不發布assignment bundle，也不下載preparation-result bundle。
- [ ] Branch不republish Leaf assignment或preparation result。
- [ ] Client不依model bundle判斷Branch／Leaf hierarchy role。
- [ ] Protocol execution只有message contract可控制topology、policy、strategy及status。
- [ ] Flat／distributed non-candidate Model Training behavior維持通過。
- [ ] `HIERARCHY_AGGREGATE`、round result及sample provenance維持通過。
- [ ] Protocol resource failure／DELETE／generation reset仍完成bounded cleanup。
- [ ] Hierarchical deployment runner不再dynamic import legacy runner。
- [ ] Legacy-only runner、static hierarchy wrapper、static collection helper與dead tests
  已移除；不得留下不可執行entrypoint。
- [ ] PyMTLF full tests、ruff、NWDAF tests／lint／build及nwdaf-resources checks通過。
- [ ] Canonical protocol real-process scenario通過。
- [ ] Distributed／flat local real-process regression通過。
- [ ] 正式external testbed若未執行，已明列為remaining gap。

---

## 11. 明確延後

- Branch failure detection、replacement selection與fencing；
- retained-result persistence、lookup、handoff及cleanup policy；
- runtime topology self-healing；
- ADRF caller authorization enforcement；
- 正式4／8／16 participant experiment與統計分析；
- 將experimental candidate schema upstream成正式3GPP proposal。

上述項目不因legacy path移除而成為Slice 6 blocker。

---

## 12. Repository審查與commit邊界

預期implementation commits依repository分開：

1. `PyMTLF/`：protocol-only hierarchy cutover與legacy artifact/runtime removal；
2. `nwdaf-resources/`：canonical protocol harness及legacy scenario removal；
3. `nwdaf-docs/`：Slice 6 plan／review evidence與上層狀態更新；
4. `NWDAF/`：只有出現經確認且在本slice範圍內的dead compatibility code才另提commit。

實作完成後先保持unstaged diff，回報affected repositories、diff summary、測試
結果與remaining gaps，等待user review。Review確認後再提出完整commit proposal；不得
因本plan核准而直接commit或push。
