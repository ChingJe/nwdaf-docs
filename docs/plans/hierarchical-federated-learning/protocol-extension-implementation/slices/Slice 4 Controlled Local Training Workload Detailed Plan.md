# Slice 4 — Controlled Local Training Workload Detailed Plan

日期：2026-09-04（2026-09-05依Slice 4A後baseline校正）

狀態：Review Confirmed／Commit Approval Pending；production implementation、review、
in-scope remediation與required verification已完成

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
- MNIST與CIFAR-10各有可由現行trusted loader讀取的controlled initial model bundle
  source，供local smoke提供model architecture與initial weights；
- `UE_COMMUNICATION`可繼續使用既有`consumer_subscription`標準資料取得流程；既有
  `private_api`模式亦不被破壞；
- model bundle與FL round artifacts可依workload profile驗證所需component，不再強迫
  image classification假裝具有traffic `scaler.pkl`；
- 既有sample-weighted aggregation與FedProx penalty可跨known workload重用；
- final model可在training結束後，以不參與local training的held-out test set離線計算
  workload-specific metric；
- 一組不依賴UPF、MongoDB、ADRF Data Management或runtime外部下載的local smoke
  evidence；該smoke使用一次性準備的real-dataset `.npz` shards與held-out set，供
  Slice 5直接重用。

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

Slice 4A已由`PyMTLF` commit `899bf23`完成並通過review。本slice直接使用其簡化後的
whole-artifact key contract，不建立任何暫時性的component、model、weights或dataset
digest，也不恢復舊schema compatibility path。

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
| `PyMTLF/` | `899bf23b44da3699591ea30e7ae6eccdbbab0802` | Workload、dataset、trainer、artifact與aggregation owner |
| `nwdaf-docs/` | `35d000ff6549fd4ac50ab38a6f5431bac50062d7`後的本計畫working tree | Canonical plan與review evidence |
| `NWDAF/` | `302762a6af677f5ccfb5a3f9d0253fb3dd39bf62` | Read-only runtime dependency；本slice不預期修改 |

2026-09-05已在上述baseline上完成Slice 4 implementation與review；本次變更保持
unstaged、uncommitted，供user直接檢查working-tree diff。

### 3.2 現有 PyMTLF 限制

目前實作不是更換一個dataset path就能在traffic與image classification之間切換。
Slice 4A已移除多層digest與舊schema reader；目前剩餘的限制如下：

- `core/artifacts.py`與`FLArtifactReader`仍要求bundle精確包含`config.json`、`model.py`、
  `model.npy`與`scaler.pkl`；
- `TrustedBundleLoader`固定載入`StandardScaler`，model constructor參數亦以現有traffic
  model shape為主；
- `tools/import_seed_model.py`仍固定讀取全域`REQUIRED_BUNDLE_FILES`，並重新產生Slice 4A
  已移除的`bundle_schema_version`與`file_digests`；它目前產生的bundle會被現行
  `ArtifactRepository`拒絕，且無法建立不含`scaler.pkl`的classification bundle；
- `TrainingDataset`、`LocalTrainer`與`FederatedTrainer.train()`以traffic sequence、
  log transformation、Huber loss及WAPE設計；
- final validation與publication gate依賴traffic WAPE語意；
- `FederatedTrainer.aggregate()`本身只聚合PyTorch state dict並依sample count加權，
  可跨workload重用。

因此本slice需要新增受控且明確的workload profile與data-source boundary，不能以identity
scaler、假的traffic fields或測試mock掩蓋語意差異，也不能讓config指向任意未註冊的
trainer或preprocessor。

### 3.3 Dependency boundary

PyMTLF目前已有`numpy`與`torch`，沒有`torchvision`或Parquet reader。本計畫不新增
runtime `torchvision`／Parquet dependency，也不允許Client啟動後向Internet下載MNIST或
CIFAR-10。Workspace-local raw cache已備妥MNIST IDX gzip與CIFAR-10 train／test Parquet；
它們不屬於任何repository，也不是runtime input。Local smoke開始前，僅在開發／實驗準備
環境一次性轉成`.npz` shards與held-out set；PyMTLF runtime只消費轉換完成的檔案。
既有`UE_COMMUNICATION`則繼續由已存在的training-data collection owner取得資料，不轉成
local image dataset path。

---

## 4. Slice 邊界

### 4.1 納入

- 已知workload profile：`ue_communication_forecasting`與`image_classification`。
- 已知data source：既有`consumer_subscription`、`private_api`與controlled `local`。
- Profile-specific bundle component contract與trusted loading。
- 修正`tools/import_seed_model.py`和Slice 4A後的現行artifact contract不一致問題，讓它
  重用同一profile-specific component owner，且不恢復private version／digest欄位。
- MNIST／CIFAR-10 controlled initial model bundle sources與profile-aware loading；這些
  bundles只作為local controlled workload輸入，不註冊成標準NWDAF analytics model。
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
- 正式dataset generator、4／8／16 participant partition、IID／non-IID選擇及paper用
  train／validation／test切分工具；這些由dataset／experiment工作提供。本slice只允許為
  local smoke一次性準備少量deterministic `.npz` inputs，不把該步驟做成runtime能力。
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

所有Slice 4建立或消費的bundle與FL artifacts都必須明確寫入`workload_profile`；省略
此欄位一律拒絕，不依舊格式推測profile。既有experimental traffic artifacts應以目前
contract重新產生，不保留舊bundle reader或fallback。

Profile是PyMTLF內部artifact／execution contract，不是本次proposed protocol extension
field。Slice 5收到model後，可由已驗證bundle取得profile；dataset local path不放入
`Nnwdaf_MLModelTraining` message。

Classification initial model在本slice以controlled source bundle進入同一
`ArtifactRepository`／trusted-loader boundary，但不加入現行`model_provision.seed_models`
或對外Model Provisioning通知。現行seed descriptor的`event`會進入標準形狀的model
notification；在沒有定義合適NWDAF analytics event前，不得把MNIST／CIFAR-10假裝成
`UE_COMMUNICATION` model。Slice 5若需要Root-side durable family selection，必須另行確認
該model source如何進入hierarchical run，不能在本slice靜默借用traffic catalog identity。
Classification source manifest以`workload_profile`及image input／class contract表達用途；
不得由import tool補上假的`analytics_event: UE_COMMUNICATION`。Traffic source則繼續保留
真正的`analytics_event`語意。

### 5.2 Workload 與 data source 的設定

設定必須把「做什麼任務」和「資料從哪裡來」分開。沿用現有`FLClientSettings` owner，
本slice固定使用下列nesting；不得把dataset名稱變成trainer dynamic import或protocol
field。

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

MNIST使用`[N, 1, 28, 28]`；CIFAR-10使用`[N, 3, 32, 32]`。Loader進入training前將
`uint8` images轉成`float32`並除以`255.0`，不套用traffic workload的`StandardScaler`。
Labels轉成`int64`且介於0到9；images與labels的第一維必須一致。這些是loader完成training
所需的基本格式處理，不建立額外dataset certification或完整性驗證流程。

下列語意不可改變：

- path是node-local deployment configuration；
- request sender無法透過protocol覆寫任意filesystem path；
- Branch若只聚合children results，不需要配置local training shard；
- Leaf或同時執行local training的node必須配置shard，training時才能讀取資料。

### 5.4 Dataset 與實驗責任邊界

本slice不負責正式實驗的dataset generator或partition policy。為完成local smoke，可由
workspace-local raw cache一次性產生少量deterministic per-Client `.npz` shards及獨立
held-out test set；該轉換不進入PyMTLF startup／runtime，也不新增manifest、hash或
per-instance preparation。正式實驗仍由Dataset／experiment工作一次產出所有per-Client
shards，deployment再把每個shard掛載給對應PyMTLF。相同participant scale的Flat與HFL
使用相同shards，train／validation／test切分及IID／non-IID規則由該工作記錄，不轉成
PyMTLF runtime validation。

Held-out test set不掛載到Client local training path，也不被local update使用；只提供給
run完成後的offline evaluator。

### 5.5 Image-classification model與training semantics

固定語意：

- dataset profile決定input shape、input channel及normalization；
- MNIST與CIFAR-10共用同一個parameterized small-CNN architecture family，只由
  `input_channels`區分MNIST的1 channel與CIFAR-10的3 channels：

  ```text
  Conv2d(input_channels, 32, kernel_size=3, padding=1)
  -> ReLU
  -> MaxPool2d(2)
  -> Conv2d(32, 64, kernel_size=3, padding=1)
  -> ReLU
  -> AdaptiveAvgPool2d(1)
  -> Flatten
  -> Linear(64, 10)
  ```

- 此CNN不使用BatchNorm，避免各Client的running statistics在FL aggregation時引入額外
  state語意；
- model輸出`[batch, 10]` logits；
- loss使用`CrossEntropyLoss`；
- labels使用class indices，不做one-hot；
- model architecture與initial weights來自trusted bundle，不由dataset config動態生成；
- MNIST與CIFAR-10各自使用明確的controlled source bundle；bundle manifest記錄matching
  input shape、channel count與class count，並分別保存各自的initial weights；兩者不共用
  同一份initial weights。Local config只選dataset，不生成或修改模型；
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
| Initial model artifact | Adapted | Local smoke使用受版本控制的classification source bundle並經現行artifact／trusted-loader boundary；不冒充標準NWDAF model family |
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
- `seed_models/image_classification/mnist/`與`seed_models/image_classification/cifar10/`
  controlled source bundles
  - MNIST／CIFAR-10各有可重現的model source、initial weights與manifest；
  - local smoke必須經`ArtifactRepository`與trusted loader，不直接注入任意
    in-memory model；
  - 不把classification source註冊到標準Model Provisioning event catalog。
- `tools/import_seed_model.py`
  - 移除仍在產生的`bundle_schema_version`與`file_digests`；
  - 從source manifest取得`workload_profile`，並重用artifact owner的profile-specific
    component contract，不自行維護第二份file list；
  - 只有traffic profile處理真正的`analytics_event`，classification profile不合成假的
    `UE_COMMUNICATION` event。
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
- Image pixels由`uint8`轉成`float32`並除以`255.0`，不讀取或建立`scaler.pkl`。
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
- Import tool產生的traffic與classification bundles都能由現行`ArtifactRepository`重新
  載入，且manifest不含`bundle_schema_version`或`file_digests`；classification bundle
  不含假的`analytics_event`。
- MNIST與CIFAR-10 bundle載入相同CNN architecture family、各自匹配1／3 input channels，
  並保存彼此獨立的initial weights；model不含BatchNorm state。

### 9.3 Real training tests

- 使用真實tiny MNIST-shaped及CIFAR-10-shaped tensors執行optimizer step，證明已選擇正確
  preprocessing path、weights改變且loss有限。
- `fedProx.mu=0`與無proximal penalty一致；正值會納入已知global state距離。
- 兩個不同sample count的真實local results經existing aggregator得到預期加權state。
- Branch-only aggregation不建立或讀取local shard。
- Offline evaluator以已知predictions／labels計算正確accuracy與sample count。
- 至少一組real MNIST local smoke shards與held-out set由workspace-local raw cache一次性
  轉換；CIFAR-10 loader／training path以tiny CIFAR-shaped fixture驗證，若同時準備real
  CIFAR-10 smoke input則沿用相同`.npz` contract。

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
   現有behavior，並固定重現current import tool和Slice 4A後artifact contract不一致問題。
2. 建立typed workload／data-source config與合法組合validation。
3. 建立profile-specific bundle inventory與known-profile discriminator。
4. 讓seed import tool重用profile-specific artifact contract，移除殘留private version／
   digest輸出。
5. 建立MNIST／CIFAR-10 controlled classification source bundles並通過現有artifact／trusted
   loader boundary。
6. 實作MNIST／CIFAR-10 local shard loader與configuration selection。
7. 實作dataset-specific preprocessing、cross-entropy training與profile adapter。
8. 將existing FL Client／Server最小化接入profile adapter並保留generic aggregation。
9. 讓round／aggregate／final artifact contract變成profile-aware。
10. 實作offline held-out evaluator。
11. 從workspace-local raw cache一次性準備smoke `.npz`，並使用repository內小型fixtures
    執行local smoke。
12. 執行focused tests、full regression、lint與local smoke。
13. 更新review ledger，保留unstaged diff供user review。

每一步若發現既有traffic semantics必須被改寫，先確認是必要adaptation還是可分離的profile
implementation。不得為了讓image classification通過而弱化所有bundle或result validation。

---

## 11. Acceptance checklist

- [x] `ue_communication_forecasting`與`image_classification`具有明確且封閉的execution
  contract。
- [x] 標準`UE_COMMUNICATION + consumer_subscription`流程保持通過。
- [x] Local image data可由config選擇MNIST或CIFAR-10，不寫死於FL execution。
- [x] Local image Client只讀deployment掛載的shard，runtime不下載dataset，也不要求每個
  PyMTLF執行dataset preparation。
- [x] 大量部署可共用相同config template與容器內`shard_path`，只由mount mapping分配
  各Client資料。
- [x] Local config只需提供dataset與shard path，不要求manifest或digest欄位，且不經
  protocol傳遞。
- [x] Image classification不使用假的traffic `scaler.pkl`或traffic fields。
- [x] MNIST／CIFAR-10使用已定義的shared small-CNN family，分別匹配1／3 input channels、
  使用獨立initial weights，且不含BatchNorm。
- [x] MNIST／CIFAR-10 controlled initial bundles可經現行artifact／trusted-loader boundary
  載入，且不被註冊或回報成標準NWDAF analytics model。
- [x] Seed import tool不再產生Slice 4A已移除的private version／digest欄位，並能以同一
  profile-specific component contract建立traffic與classification bundles。
- [x] Unknown／mismatched workload、data source、dataset或model contract在training前被拒絕。
- [x] 真實image-classification optimizer step、FedProx與sample-weighted aggregation測試通過。
- [x] Branch-only aggregation不需要local dataset。
- [x] Final model可由獨立held-out evaluator產生accuracy evidence。
- [x] Traffic workload tests與artifact safety regressions保持通過。
- [x] Local smoke不依賴UPF、MongoDB、ADRF Data Management或外部network download。
- [x] Local smoke的real-data `.npz`由workspace-local raw cache一次性準備；PyMTLF runtime
  不包含download、Parquet／IDX conversion或per-instance preparation。
- [x] 沒有修改protocol schema、NRF或ADRF contract。
- [x] Slice 5能直接消費本slice的local workload，而不重新定義dataset或trainer。
- [x] Affected repositories的diff、verification與remaining gaps已交付user review。

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

## 13. Review evidence

### 13.1 Revision與working tree

| Repository | Baseline HEAD | 狀態 |
| --- | --- | --- |
| `PyMTLF/` | `899bf23b44da3699591ea30e7ae6eccdbbab0802` | Slice 4 production與test changes維持unstaged、uncommitted，等待user review |
| `nwdaf-docs/` | `35d000ff6549fd4ac50ab38a6f5431bac50062d7` | 本plan、slice map與review ledger evidence維持unstaged、uncommitted |
| `NWDAF/` | `302762a6af677f5ccfb5a3f9d0253fb3dd39bf62` | Working tree未因Slice 4修改 |

`PyMTLF/`的production owner變更集中於workload／config、artifact／trusted loading、
FL Client training、generic aggregation與offline evaluation。Tests、controlled source bundles、
sample config及CLI一併更新；沒有修改standard-shaped protocol schema、NRF或ADRF contract。

### 13.2 計畫符合性

| 要求群組 | 狀態 | 直接證據 |
| --- | --- | --- |
| Known workload與config boundary | 已滿足 | `core/workloads.py`的closed profiles／datasets、`config.py`組合validation與config tests |
| Profile-specific artifact contract | 已滿足 | `core/artifacts.py`、`core/fl_workspace.py`與artifact／workspace／import tests |
| Controlled model sources | 已滿足 | MNIST／CIFAR-10 source bundles、reproducible initialization與trusted-loader tests |
| Local image data與training | 已滿足 | `.npz` loader、normalization、CrossEntropy、real optimizer step及production `FLClientEngine` round test |
| Existing FL semantics | 已滿足 | FedProx、actual sample count、real `FLServerEngine` sample-weighted aggregation與traffic regressions |
| Branch-only aggregation | 已滿足 | Image profile不配置local shard仍可執行aggregation-only Branch的production-path test |
| Held-out evaluation | 已滿足 | Offline evaluator支援durable artifact key及FL workspace artifact path，並有known-result與CLI tests |
| Local smoke | 已滿足 | 兩個real MNIST shards完成local update、aggregation與獨立held-out evaluation |

### 13.3 Review發現與修正

| ID | 狀態 | 確認證據 | 修正 | 驗證 |
| --- | --- | --- | --- | --- |
| `S4-R1` | 已關閉 | 初版image config會要求所有image-profile nodes配置local shard，使aggregation-only Branch無法成立 | 將local shard要求移到真正執行local training的Client path；aggregation-only Branch只需profile | Config與Branch production-path tests |
| `S4-R2` | 已關閉 | 初版evaluator只接受durable `ArtifactRepository` key，但Slice 4明確允許未進traffic catalog的final workspace artifact | Evaluator增加互斥的artifact key／artifact path入口，兩者皆經同一trusted loader | Known-result與CLI workspace-artifact tests |
| `S4-R3` | 已關閉 | 初版關鍵證據偏重helper-level training／aggregation，不能直接證明production FL owners已接上 | 新增真實`FLClientEngine._run_round`及`FLServerEngine._aggregate_round` tests；只mock transport／callback boundary | Image production-path與two-client aggregation tests |

Initial full-diff review與上述targeted follow-up review均已完成，目前沒有未關閉的Slice 4
code finding。

### 13.4 Verification結果

| 驗證 | 結果 |
| --- | --- |
| `.venv/bin/pytest -q` | Pass；710 passed、2 skipped、55個dependency deprecation warnings |
| `.venv/bin/ruff check .` | Pass |
| `git diff --check` | Pass |
| Real MNIST local smoke | Pass；2 Clients各64 training samples，128 held-out samples，accuracy `0.078125` |

Real MNIST smoke直接讀取workspace-local raw IDX cache，一次性建立temporary `.npz` shards，
執行兩個Client local updates、sample-weighted aggregation與held-out evaluation；temporary inputs
已清理。Accuracy只證明完整execution可執行，不作為模型品質或paper experiment結果。

### 13.5 Remaining gaps與review gate

- `future-phase handoff`：Root→Branch→Leaf protocol wiring、model-free preparation、feature 3、
  ADRF global-model distribution、topology Notify／PATCH與multi-lower-round transport由Slice 5負責。
- `integration verification gap`：尚未執行real multi-process／multi-NWDAF testbed protocol E2E。
- `future-phase handoff`：正式4／8／16 participant dataset partition、repeated experiments與paper
  evaluation不屬於本slice。
- `approved deferral`：Image final artifact不冒充traffic model進入現行durable catalog；本slice
  使用workspace artifact handoff與offline evaluator完成驗證。

Slice 4的production behavior與required verification已完成，user已確認review結果。
所有變更保持unstaged、uncommitted，等待本次commit proposal取得明確核准。
