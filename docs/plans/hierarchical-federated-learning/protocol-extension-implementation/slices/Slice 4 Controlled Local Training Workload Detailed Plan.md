# Slice 4 — Controlled Local Training Workload Detailed Plan

日期：2026-09-04

狀態：Draft／review延後至Slice 4A完成後；production implementation尚未開始

相關文件：

- [Protocol Extension Implementation Plan](../Hierarchical%20NWDAF%20FL%20Protocol%20Extension%20Implementation%20Plan.md)
- [Protocol Extension Implementation Slice Map](../Protocol%20Extension%20Implementation%20Slice%20Map.md)
- [Protocol Implementation Current-State Inventory](../Protocol%20Implementation%20Current-State%20Inventory.md)
- [Slice 2 Detailed Plan](./Slice%202%20Candidate%20Pool%20Policy%20and%20Local%20Contract%20Execution%20Detailed%20Plan.md)
- [Slice 4A Detailed Plan](./Slice%204A%20Digest%20Simplification%20and%20Contract%20Cleanup%20Detailed%20Plan.md)
- [Slice 5 Detailed Plan](./Slice%205%20Protocol-driven%20Hierarchy%20Integration%20Detailed%20Plan.md)
- [NWDAF Development Policy](../../../../development_policy.md)

---

## 1. Slice 結果

本 slice 在 protocol-driven hierarchy integration 前，先建立明確的known-workload
boundary。Workload profile決定task、preprocessing、training與metric語意；data source則
決定資料由既有Model Training collection取得，或從部署前放置的local shard讀取。
既有標準`UE_COMMUNICATION` forecasting流程維持可用；controlled experiment可由local
config選擇MNIST或CIFAR-10 image classification，不把dataset寫死在FL execution內。
Root／Branch仍重用既有FL aggregation owner，最後再以各workload對應的held-out資料評估
final global model。

完成後應具備：

- typed workload與data-source configuration；
- 每個Client直接讀取部署時掛載的local shard，不執行per-instance dataset preparation，
  也不在runtime下載dataset；
- `ue_communication_forecasting`與`image_classification`兩種known workload contract；
- `image_classification`依dataset設定選擇MNIST或CIFAR-10的資料讀取、shape與
  normalization，並與received model bundle的input／class contract對齊；
- `UE_COMMUNICATION`可繼續使用既有`consumer_subscription`標準資料取得流程；既有
  `private_api`模式亦不被破壞；
- model bundle與FL round artifacts可依workload profile驗證所需component，不再強迫
  image classification假裝具有traffic `scaler.pkl`；
- 既有sample-weighted aggregation與FedProx penalty可跨known workload重用；
- final model可在training結束後，以不參與local training的held-out test set離線計算
  workload-specific metric；
- 一組不依賴UPF、MongoDB、ADRF Data Management或runtime外部下載的local smoke
  evidence，供Slice 5直接重用。

MNIST與CIFAR-10在本計畫中是controlled FL workloads，不是NWDAF analytics資料或新的
3GPP information element。它們用來降低實驗資料路徑干擾，驗證hierarchy／protocol造成
的系統差異；不得將classification accuracy解讀成NWDAF traffic analytics成效。

---

## 2. 為何獨立成 Slice 4

老師提出的實驗情境偏向使用MNIST等簡單資料集，並將資料預先放在每個Client本地；
CIFAR-10則保留為較不易快速飽和的同類controlled workload選項。
這和目前PyMTLF的production baseline有實質差異：現有training dataset、bundle、loss與
final validation都以traffic time-series forecasting為中心。若直接塞進Slice 5，protocol
wiring、ADRF model distribution與training workload改造會混成一個難以review的跨界變更。

因此本slice只完成「本地controlled workload能被現有FL execution使用」；Slice 5才負責
將它接上Root→Branch→Leaf subscription／Notify與model transport。兩者的界線如下：

本slice的production implementation以Slice 4A完成為前置條件，直接使用其簡化後的
whole-artifact key contract，不建立任何暫時性的component、model、weights或dataset
digest。

| 問題 | Slice 4 | Slice 5 |
| --- | --- | --- |
| Client訓練資料從哪裡來 | 部署時掛載的local shard | 重用Slice 4 local loader |
| 使用何種task／dataset／loss／metric | 由known profile與local config選擇 | 不重新定義 |
| local training與aggregation是否可執行 | 是 | 接入protocol resource lifecycle |
| `x-flTopology`／policy／feature 3 | 不處理 | 處理 |
| Root global model ADRF distribution | 不處理 | 處理 |
| topology Notify與PATCH | 不處理 | 處理 |

---

## 3. 基準與直接證據

### 3.1 盤點版本

| Repository | Revision | Slice 4角色 |
| --- | --- | --- |
| `PyMTLF/` | `0e87ef13622cba0ddf740aa8cedce8ad76705815` | Workload、dataset、trainer、artifact與aggregation owner |
| `nwdaf-docs/` | 本計畫所在working tree | Canonical plan與review evidence |
| `NWDAF/` | `302762a6af677f5ccfb5a3f9d0253fb3dd39bf62` | Read-only runtime dependency；本slice不預期修改 |

Production implementation開始前需再次確認affected repositories的HEAD與working tree。
若PyMTLF trainer、artifact contract或Slice 2 execution owner已改變，先更新本計畫的
exact-file mapping與baseline disposition。

### 3.2 現有 PyMTLF 限制

目前實作不是更換一個dataset path就能在traffic與image classification之間切換。
本slice實作前由Slice 4A先移除多層digest contract；下列清單中的digest限制是
pre-Slice-4A baseline，不是本slice應延伸的新要求：

- `core/artifacts.py`要求bundle精確包含`config.json`、`model.py`、`model.npy`與
  `scaler.pkl`；
- `TrustedBundleLoader`固定載入`StandardScaler`；
- `TrainingDataset`、`LocalTrainer`與`FederatedTrainer.train()`以traffic sequence、
  log transformation、Huber loss及WAPE設計；
- FL artifact的`file_digests`也固定要求相同traffic bundle components；
- final validation與publication gate依賴traffic WAPE語意；
- `FederatedTrainer.aggregate()`本身只聚合PyTorch state dict並依sample count加權，
  可跨workload重用。

因此本slice需要新增受控且明確的workload profile與data-source boundary，不能以identity
scaler、假的traffic fields或測試mock掩蓋語意差異，也不能讓config指向任意未註冊的
trainer或preprocessor。

### 3.3 Dependency boundary

PyMTLF目前已有`numpy`與`torch`，沒有`torchvision`。本計畫不新增runtime
`torchvision` dependency，也不允許Client啟動後向Internet下載MNIST或CIFAR-10。
Dataset取得與license／source record是部署前置作業；runtime只消費已準備好的本地
shard。既有`UE_COMMUNICATION`則繼續由已存在的training-data collection owner取得資料，
不轉成local image dataset path。

---

## 4. Slice 邊界

### 4.1 納入

- 已知workload profile：`ue_communication_forecasting`與`image_classification`。
- 已知data source：既有`consumer_subscription`、`private_api`與controlled `local`。
- Profile-specific bundle component contract與trusted loading。
- MNIST／CIFAR-10 local shard loader與最小資料格式contract。
- Image classification的dataset-specific loading／normalization、cross-entropy training與
  accuracy evaluation。
- Existing FedProx penalty與sample-weighted aggregation對known workloads的重用。
- Local dataset config與read-only deployment mount。
- Branch-only aggregation不要求local dataset。
- Final global model的offline held-out evaluation。
- Traffic workload完整regression。

### 4.2 不納入

- `x-flTopology`、`x-flTopologyReport`、feature 3或standard-shaped message變更。
- Root→Branch→Leaf protocol wiring與resource lifecycle。
- Root global model ADRF record、allowlist、distribution或cleanup。
- NRF discovery、candidate pool或participant policy變更。
- UPF、DCCF、ADRF Data Management、MongoDB或live traffic ingestion。
- Runtime下載MNIST／CIFAR-10或在production dependency加入`torchvision`。
- Dataset產生、partition、IID／non-IID選擇與train／validation／test切分工具；這些由
  dataset／experiment工作提供，本slice只消費其輸出。
- 正式4／8／16 participant experiment、統計分析或paper result。
- 將MNIST／CIFAR-10註冊為標準NWDAF analytics event。
- Retained-result recovery。

---

## 5. Workload 與 data-source contract

### 5.1 Profile identity

Bundle manifest與local execution state新增內部discriminator `workload_profile`，語意為
「這個model與dataset要由哪一個已知training profile解讀」。第一版只接受：

- `ue_communication_forecasting`：保留現有traffic sequence、`StandardScaler`、Huber
  loss與WAPE；
- `image_classification`：使用cross-entropy與accuracy；dataset設定再選擇MNIST或
  CIFAR-10的input與preprocessing contract。

Unknown profile必須在trusted bundle validation或execution preparation被拒絕，不做
dynamic import、vendor plugin或fallback到traffic trainer。

既有`bundle_schema_version: "1.0"` traffic bundles為相容性例外：若省略
`workload_profile`，只有在它仍符合完整legacy traffic component與manifest contract時，
才解讀為`ue_communication_forecasting`。所有新產生的bundle與FL artifacts都必須明確
寫入`workload_profile`；image classification省略此欄位一律拒絕。這保留已存在的traffic
artifacts，但不讓任意缺欄位bundle被猜成某個profile。

Profile是PyMTLF內部artifact／execution contract，不是本次proposed protocol extension
field。Slice 5收到model後，可由已驗證bundle取得profile；dataset local path不放入
`Nnwdaf_MLModelTraining` message。

### 5.2 Workload 與 data source 的設定

設定必須把「做什麼任務」和「資料從哪裡來」分開。沿用現有config owner時，概念上等同
下列兩種設定；exact key nesting可在implementation對照`config.py`後確定，但不得把
dataset名稱變成trainer dynamic import或protocol field。

標準`UE_COMMUNICATION`範例：

```yaml
federated_learning:
  client:
    workload:
      profile: ue_communication_forecasting
    training_data:
      collection_trigger: consumer_subscription
```

這個組合重用既有Model Training consumer subscription、`DatasetCoordinator`、traffic
dataset builder、`StandardScaler`、Huber loss與WAPE。既有`private_api`可作為同一workload
的另一個data source，不因本slice被移除或重新解讀。

Controlled image classification範例：

```yaml
federated_learning:
  client:
    workload:
      profile: image_classification
    training_data:
      collection_trigger: local
      dataset: cifar10
      shard_path: /var/lib/py-mtlf/datasets/client-01-train.npz
```

`dataset`第一版接受`mnist`或`cifar10`。它選擇已註冊的shape、channel、normalization與
dataset loading；received model bundle仍是model architecture與weights的來源，且其
input shape／class count必須和configured dataset contract相容。Config只在已實作的路徑
之間選擇，不允許任意程式碼、trainer class或preprocessing function。

部署者只需選擇dataset並指定`shard_path`。大量部署時，各PyMTLF可以使用同一份config
template及相同的容器內路徑，例如全部使用`/data/train.npz`，由deployment mapping把不同
Client的shard以read-only方式掛載到該位置。每個PyMTLF不需要另外執行dataset preparation
command，也不需要維護manifest或hash欄位。

### 5.3 Local image shard representation

每個Client使用一個read-only `.npz` training shard。最小representation為：

```text
images: uint8 [N, C, H, W]
labels: integer [N]
```

MNIST使用`[N, 1, 28, 28]`；CIFAR-10使用`[N, 3, 32, 32]`。Loader進入training前依dataset
contract轉成normalized `float32` tensor。Labels轉成`int64`且介於0到9；images與labels的
第一維必須一致。這些是loader完成training所需的基本格式處理，不建立額外dataset
certification或完整性驗證流程。

下列語意不可改變：

- path是node-local deployment configuration；
- request sender無法透過protocol覆寫任意filesystem path；
- Branch若只聚合children results，不需要配置local training shard；
- Leaf或同時執行local training的node必須配置shard，training時才能讀取資料。

### 5.4 Dataset 與實驗責任邊界

本slice不負責產生或切分dataset。Dataset／experiment工作一次產出所有per-Client shards
及獨立held-out test set；deployment再把每個shard掛載給對應PyMTLF。相同participant
scale的Flat與HFL使用相同shards，train／validation／test切分及IID／non-IID規則由該工作
記錄，不轉成PyMTLF runtime validation。

Held-out test set不掛載到Client local training path，也不被local update使用；只提供給
run完成後的offline evaluator。

### 5.5 Image-classification model與training semantics

固定語意：

- dataset profile決定input shape及normalization；
- model輸出`[batch, 10]` logits；
- loss使用`CrossEntropyLoss`；
- labels使用class indices，不做one-hot；
- model architecture與initial weights來自trusted bundle，不由dataset config動態生成；
- local epochs由effective `reportAfter.epochs`控制；省略時沿用node local default；
- optimizer先重用目前可配置的Adam及learning rate owner，不額外新增scheduler；
- `strategy.method=fedProx`時，在classification loss上加入既有proximal penalty；
- `aggregation.method=sampleWeighted`時，繼續以實際local sample count加權；
- reproducibility evidence保存model initialization seed、data order seed與run ID。

本slice不追求模型準確率調參。選定的MNIST／CIFAR-10 model bundle與training budget只需
能證明真實gradient update、跨Client aggregation與held-out evaluation可執行。

---

## 6. Execution 與 ownership

### 6.1 Known-profile adapter

PyMTLF新增一個小型、封閉的workload adapter／registry boundary，由profile identity選擇
已知implementation：

- `ue_communication_forecasting` adapter包住既有dataset build、scaler、training與WAPE
  behavior，並依既有`training_data.collection_trigger`選擇`consumer_subscription`或
  `private_api`；
- `image_classification` adapter負責local shard loading、依dataset選擇tensor
  preparation、classification training與accuracy evaluation。

這個boundary的目的，是把workload-specific語意從FL orchestration中分離，而不是建立
任意plugin framework。FL lifecycle owner仍是既有`FLClientEngine`／`FLServerEngine`；
adapter不建立第二套subscription、round state或aggregation engine。

### 6.2 Local execution flow

Slice 4的controlled local image execution流程：

1. PyMTLF讀取local workload config。
2. 驗證workload profile、data source與dataset組合是已知且相容的值。
3. Training開始時由`shard_path`讀取部署已掛載的shard，轉成dataset-specific tensors。
4. 載入匹配profile的initial model bundle，確認input shape與class count可供該dataset使用。
5. 進入既有local training與aggregation flow。

Slice 5的model-free protocol preparation不讀取或處理dataset。第一輪global model到達並
真正開始training時，才讀取configured shard並確認model可供該workload使用。

`ue_communication_forecasting`不走上述local shard流程；它保持既有
`consumer_subscription`或`private_api` collection、dataset snapshot與traffic model
validation流程。本slice只在共同workload boundary接入它並以regression鎖定既有語意。

### 6.3 Training與aggregation flow

```text
pre-staged local shard
  -> known workload adapter loads and prepares tensors
  -> FLClientEngine supplies received global state and effective local work
  -> existing FederatedTrainer ownership performs real local update
  -> local result records actual sample count
  -> existing FLServerEngine freezes selected cohort
  -> existing sample-weighted aggregate combines state dicts
  -> final global state handed to offline evaluator
```

Branch如果只扮演lower-tier FL Server，不得因image-classification profile而被要求建立假的local dataset。
若同一node同時需要本地參與上層training，該local-client process與lower-server process各自
依既有process state取得所需input，不共用未經識別的mutable dataset state。

### 6.4 Artifact contract

Bundle與round artifact必須依profile決定合法component inventory：

| Profile | Required model bundle components | Preprocessing evidence |
| --- | --- | --- |
| `ue_communication_forecasting` | `config.json`、`model.py`、`model.npy`、`scaler.pkl` | Scaler與feature contract |
| `image_classification` | `config.json`、`model.py`、`model.npy` | Dataset-specific normalization與input shape／class count |

Component set仍需精確符合該profile要求，不能變成接受任意檔案；不產生或驗證
`file_digests`。Round local、aggregate與final artifacts沿用同一profile及完整artifact
identity；image classification不得攜帶假的`scaler.pkl`來通過validator。

Existing archive path traversal、size、whole-artifact URL/body identity、trusted `model.py`
loading與workspace cleanup invariants全部保留。若目前publication catalog強制依賴
traffic-only final validation，
本slice先提供明確的controlled final artifact handoff與offline evaluation evidence，不在
未釐清catalog語意前把image-classification結果冒充已接受的traffic model version。

### 6.5 Evaluation output

Offline evaluator輸入：

- final global image-classification bundle或model state；
- held-out test `.npz`；
- matching model／preprocessing profile。

輸出至少包含：

- run ID；
- model artifact key；
- test dataset path／dataset identity；
- test sample count；
- correct count；
- accuracy；
- evaluation timestamp。

它不回寫Client training state，也不使用distributed WAPE acceptance gate。是否在後續正式
experiment增加confusion matrix或per-class accuracy，不是本slice完成條件。

---

## 7. Existing-flow disposition

| Baseline stage | Slice 4處置 | 說明 |
| --- | --- | --- |
| Root trigger／request registry | Reused without semantic change | 本slice可由local smoke入口啟動；不改SBI trigger |
| Participant／candidate selection | Reused without semantic change | 重用Slice 2 selected-set contract |
| Traffic data collection | Reused without semantic change for UE Communication | `consumer_subscription`與`private_api`維持既有owner；local image profile不啟動traffic collection |
| Dataset loading | Adapted | 由workload與data-source config選擇traffic builder或local image loader |
| Initial model artifact | Adapted | Profile-specific component contract；安全檢查保留 |
| Local training | Adapted | Image classification使用cross entropy；UE Communication使用既有Huber／WAPE path |
| FedProx local penalty | Reused without semantic change | 套用在profile-specific base loss之上 |
| Round result | Adapted | 保留sample count與model state；metric payload依profile解讀 |
| Aggregation | Reused without semantic change | 既有state-dict sample weighting |
| Round dispatch／wait／timeout | Reused without semantic change | 本slice不改protocol transport |
| Distributed traffic final validation | Not applicable to image classification | 不以WAPE判定classification result |
| Final evaluation | Adapted | Image classification使用獨立held-out accuracy evaluator |
| Durable catalog publication | Approved for deferral if traffic-specific | 不影響controlled workload與Slice 5 transport驗證 |
| Successful completion | Adapted | 需產生final artifact與offline evaluation evidence |
| Failure／cleanup | Reused and extended | Training tensor、workspace與temporary artifact需可清理；deployment-mounted shard不由PyMTLF刪除 |
| Restart | Reused without semantic change | Pre-staged data保留；in-memory run state不恢復 |

---

## 8. Repository 工作拆分

### 8.1 `PyMTLF/`

預期修改的owner區域，exact files在implementation開始前依HEAD確認：

- `src/py_mtlf/config.py`
  - 加入typed local workload configuration；
  - 驗證profile、data source、dataset與shard path語意。
- `src/py_mtlf/core/artifacts.py`
  - 將固定traffic inventory改為known profile-specific inventory；
  - 保留exact-file與whole-artifact key validation，不恢復component digest。
- `src/py_mtlf/core/trainer.py`、`core/federated_trainer.py`
  - 抽出known-profile execution seam；
  - 保留UE Communication behavior並加入image-classification training。
- 新增或擴充既有dataset owner下的local image loader
  - 不為payload shape另建不必要package；
  - owner應接近既有dataset／training code，而非SBI package。
- `src/py_mtlf/core/fl_artifacts.py`、`core/fl_workspace.py`
  - 讓round／aggregate／final artifacts保存profile並驗證正確component set。
- `src/py_mtlf/core/fl_client.py`、`core/fl_server.py`
  - 只做選擇workload adapter與保存profile-aware result所需的最小修改；
  - 不改candidate或protocol wire contract。
- Offline evaluator入口
  - 重用trusted bundle loader與image-classification adapter；
  - 不建立另一套model parser。

### 8.2 `nwdaf-docs/`

- 更新本plan、slice map與review ledger；
- implementation完成後記錄exact revisions、verification與remaining gaps；
- 不把正式experiment result寫進implementation plan。

### 8.3 不預期修改

- `NWDAF/`：本slice沒有新的Go-facing或standard boundary；
- `nwdaf-resources/`：dataset generator與部署scenario不是本slice implementation owner；
- `adrf/`：本slice不走ADRF model management；
- `nrf/`：candidate discovery已由Slice 2處理；
- `PyAnLF/`、`smf-nwdaf-ext/`、`udm/`、`udr/`：不在資料或training flow內。

若implementation證明需要上述repository修改，先補完整producer→transport→consumer flow
並重新確認slice boundary，不直接擴大工作樹。

---

## 9. 測試與驗證計畫

### 9.1 Dataset loader tests

- MNIST與CIFAR-10 fixtures可由configured `shard_path`讀取並轉成正確tensor。
- 缺檔或無法形成training tensor的資料回報清楚的load／training failure。
- Config path不能由protocol payload覆寫。

### 9.2 Bundle與artifact tests

- Traffic bundle原有exact component contract保持通過。
- Image-classification bundle缺`model.py`／`model.npy`或多出未宣告component時被拒絕。
- Image classification不需要`scaler.pkl`，UE Communication仍必須有。
- Unknown profile／data source／dataset、非法組合、profile mismatch、input shape／class count
  mismatch被拒絕。
- Round local→aggregate→final artifact保留一致profile與model source identity。
- Archive、whole-artifact key與workspace safety regressions維持通過。

### 9.3 Real training tests

- 使用真實tiny MNIST-shaped及CIFAR-10-shaped tensors執行optimizer step，證明已選擇正確
  preprocessing path、weights改變且loss有限。
- `fedProx.mu=0`與無proximal penalty一致；正值會納入已知global state距離。
- 兩個不同sample count的真實local results經existing aggregator得到預期加權state。
- Branch-only aggregation不建立或讀取local shard。
- Offline evaluator以已知predictions／labels計算正確accuracy與sample count。

關鍵training與aggregation測試不得只mock trainer／aggregator後驗證helper被呼叫。Mock可用於
隔離transport或failure injection，但用來支撐「真實image classification可執行」的證據必須進入
真實tensor、loss、optimizer與state-dict aggregation path。

### 9.4 Regression與smoke

- PyMTLF full test suite與lint。
- 既有`UE_COMMUNICATION` `consumer_subscription`、`private_api`、LocalTrainer、
  FederatedTrainer、FL Client／Server與artifact tests。
- 一組local controlled FL smoke，至少兩個Client使用不同pre-staged shards完成local
  training、sample-weighted aggregation與held-out evaluation。
- Smoke輸出保存profile、participant identities、sample counts與accuracy。
- 不把single-process fake callback當成Slice 5 protocol E2E證據；本slice只主張workload
  vertical flow成立。

---

## 10. 實作順序

1. 以characterization tests鎖定traffic bundle、trainer、aggregation與final validation
   現有behavior。
2. 建立typed workload／data-source config與合法組合validation。
3. 建立profile-specific bundle inventory與known-profile discriminator。
4. 實作MNIST／CIFAR-10 local shard loader與configuration selection。
5. 實作dataset-specific preprocessing、cross-entropy training與profile adapter。
6. 將existing FL Client／Server最小化接入profile adapter並保留generic aggregation。
7. 讓round／aggregate／final artifact contract變成profile-aware。
8. 實作offline held-out evaluator。
9. 使用repository內小型fixtures執行local smoke。
10. 執行focused tests、full regression、lint與local smoke。
11. 更新review ledger，保留unstaged diff供user review。

每一步若發現既有traffic semantics必須被改寫，先確認是必要adaptation還是可分離的profile
implementation。不得為了讓image classification通過而弱化所有bundle或result validation。

---

## 11. Acceptance checklist

- [ ] `ue_communication_forecasting`與`image_classification`具有明確且封閉的execution
  contract。
- [ ] 標準`UE_COMMUNICATION + consumer_subscription`流程保持通過。
- [ ] Local image data可由config選擇MNIST或CIFAR-10，不寫死於FL execution。
- [ ] Local image Client只讀deployment掛載的shard，runtime不下載dataset，也不要求每個
  PyMTLF執行dataset preparation。
- [ ] 大量部署可共用相同config template與容器內`shard_path`，只由mount mapping分配
  各Client資料。
- [ ] Local config只需提供dataset與shard path，不要求manifest或digest欄位，且不經
  protocol傳遞。
- [ ] Image classification不使用假的traffic `scaler.pkl`或traffic fields。
- [ ] Unknown／mismatched workload、data source、dataset或model contract在training前被拒絕。
- [ ] 真實image-classification optimizer step、FedProx與sample-weighted aggregation測試通過。
- [ ] Branch-only aggregation不需要local dataset。
- [ ] Final model可由獨立held-out evaluator產生accuracy evidence。
- [ ] Traffic workload tests與artifact safety regressions保持通過。
- [ ] Local smoke不依賴UPF、MongoDB、ADRF Data Management或外部network download。
- [ ] 沒有修改protocol schema、NRF或ADRF contract。
- [ ] Slice 5能直接消費本slice的local workload，而不重新定義dataset或trainer。
- [ ] Affected repositories的diff、verification與remaining gaps已交付user review。

---

## 12. 完成與 handoff

本slice只有在local dataset loading、profile-aware artifact／training flow、真實local
aggregation、held-out evaluation、traffic regression與local smoke全部完成後，才可進入
`Ready for User Review`。User確認review後仍需另外提出commit proposal；未經commit核准
不得stage或commit。

Slice 4完成後，Slice 5可假設每個需要local training的Client都能依共同config template
讀取其deployment掛載的local shard。Slice 5只需處理protocol resource、topology、model
transport與callback flow，不把dataset產生、切分或per-instance preparation納入自己的
實作範圍。

---

## 13. Review evidence（implementation後填寫）

目前尚未開始Slice 4 production implementation。本節在實作完成後記錄：

- affected repository revisions與working-tree diff；
- exact files與owner boundary；
- focused／full test與lint結果；
- local controlled FL smoke evidence；
- traffic regression結果；
- 未完成的external／testbed validation；
- user review與commit狀態。
