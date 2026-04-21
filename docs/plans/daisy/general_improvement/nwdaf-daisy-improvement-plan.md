# NWDAF × Daisy 整合問題與改進方向

---

## 概述

NWDAF（Go）、Daisy（FL 訓練框架）、ML Service（Python 推論引擎）三者之間的整合目前存在幾個根本性問題：重訓期間 HTTP 阻塞、模型傳遞方式 hardcode、ML Service 無法動態載入模型定義、以及元件職責邊界不清晰。以下說明各問題現狀與改進方向。

---

## 1. HTTP 阻塞（重訓期間）

### 問題

NWDAF 向 Daisy 發送 POST `/publish_task` 後，會一直等待 200 回應才繼續執行。FL 訓練可能需要數分鐘，整個重訓邏輯（trigger → swap → restart monitor）在這段時間內完全被卡住。

### 改進方向

改為 async callback 模式：

- Daisy 收到 task 後立即回 `202 Accepted`，在 background thread 執行訓練
- 訓練完成後，Daisy 主動 POST `callback_url`（由 task payload 帶入），body 帶 `{task_id, model_url}`
- NWDAF 新增 `POST /mtlf/training-complete` endpoint，收到 callback 後繼續換版流程
- NWDAF 維護 in-flight 狀態表（`sync.Map`）：`task_id → {oldModelUrl, store, mtlfCfg}`，callback 收到後從中取出對應上下文

需同時確認：`IsRetraining()` guard 在 async 架構下仍能防止重複觸發。Flask 預設 single-thread，`app.run()` 須加 `threaded=True` 才能在訓練中同時處理 callback 以外的請求。

---

## 2. 模型傳遞（目前 hardcode file path）

### 問題

目前 NWDAF 從 task config 讀取 `MODEL_PATH`（Daisy 本地 file path），直接傳給 ML Service；ML Service 從該路徑讀取模型檔案。實際上 ML Service 讀的是預先放好的固定模型，Daisy 訓練完存下的 `model.npy` 並未被真正使用，是 hardcode 的形式。

### 改進方向

改為 HTTP URL 傳遞模式：

- Daisy 訓練完後，將模型打包成 tar.gz，透過 HTTP endpoint serve
- callback body 帶上 `model_url`（指向 tar.gz 下載連結）
- NWDAF 收到後將 `model_url` 轉發給 ML Service
- ML Service 自行 GET 下載、解壓、載入

`MODEL_PATH`（Daisy 本地路徑）維持不變，FL 框架仍用它做 warm start 和存訓練結果。打包和 serve 發生在訓練迴圈結束後，不影響訓練行為本身。

職責因此清楚分明：Daisy 管訓練與存模型；NWDAF 只做協調，不碰模型本身；ML Service 管下載、載入、推論。

---

## 3. 模型打包（三件套 + Scaler）

### 問題

ML Service 的模型載入目前是 hardcode 的：模型結構直接寫死在 Python code 裡，無法動態切換架構。若要換架構，需要改 ML Service 本身的程式碼，部署成本高，且無法通用。

### 改進方向

將「一個模型」定義為四個檔案的組合，打包成 tar.gz 一起傳遞：

| 檔案 | 用途 |
|------|------|
| `model.py` | 模型架構定義（PyTorch class），ML Service 直接 `from model import Model` |
| `config.yaml` | 超參數（建構 Model class 用）+ 推論介面（feature names、seq_length、output fields 等） |
| `model.npy` | 訓練後的模型權重 |
| `scaler.pkl` | 訓練時用的資料 Scaler（均值 / 標準差） |

ML Service 收到 `model_url` 後：GET 下載 tar.gz → 解壓到暫存目錄 → 從 `config.yaml` 讀超參數建構 Model class → 載入 `model.npy` 權重 → 載入 `scaler.pkl`。全部成功才視為載入完成（原子性），失敗直接 error，不進入半套狀態。

Daisy 打包時：`model.npy` / `scaler.pkl` 是訓練輸出，`model.py` / `config.yaml` 從 task payload 指定的路徑讀取（`MODEL_DEF`、`MODEL_CONFIG`），放在 `arch/<version>/` 目錄下，以 git 版本控制，不透過 FL 協議下發給 client。

> client 各自機器上需有對應版本的 `model.py`，架構改版時須透過部署腳本同步。

---

## 4. Daisy 初始化流程

### 問題

目前 Daisy 透過 `--init_model` flag 在部署時初始化模型，需要人工觸發，且與 task 裡的 `MODEL_PATH` 各自獨立，容易不一致。初始化邏輯與 task 觸發邏輯分離，也增加了部署步驟的複雜度。

### 改進方向

移除 `--init_model`，改為按需初始化：task 進來時若 `MODEL_PATH` 不存在，則自動呼叫初始化邏輯（`get_model()` + `get_dataloaders()`）。初始化邏輯以 callback 方式由 `master.py` 注入，保持 example-specific 的彈性，不把特定模型邏輯耦合進框架本身。

---

## 5. MTLF / AnLF 職責分離

### 問題

目前 `swapModelAfterRetrain`（`training.go:104`）直接呼叫 `mlClient.InitializeModel` / `mlClient.UnloadModel`，也就是 MTLF 直接操作 ML Service。這與設計原則（AnLF 負責所有 ML Service 互動）不符，且與 Phase 1 初始化路徑不一致。

同樣的問題存在於 `runDelayedTraining`（startup trigger）：啟動時直接呼叫 ML Service 載入初始模型，繞過了 AnLF。

### 改進方向

- `swapModelAfterRetrain` 只負責更新 SharedModelRegistry 和訂閱狀態，不直接操作 ML Service
- 透過 `onModelSwapReady(newModelUrl, oldModelId string)` callback 通知 AnLF，由 AnLF 執行 load new → update registry → unload old 的完整換版流程
- `runDelayedTraining` 改為通知 AnLF 載入 static_model_url，與 Phase 1 初始化路徑對齊

解耦方式：在 `MtlfService` 初始化時注入 `onModelSwapReady` callback，由 AnLF 實作。

---

## 6. SharedModelRegistry Key 設計（後續）

### 問題

目前 `SharedModelRegistry` 以 model URL 為 key，無法支援多版本並存（active / standby），也就無法實作 rollback。

### 改進方向

改以 **series ID**（例如 `"ue-comm-group1"`）為 key，版本記錄（URL、status、model_id）下放到 value 內：

```
key: series_id（穩定，不隨重訓改變）
value:
  versions:
    - version: 1, url: "http://daisy/.../v1.tar.gz", status: standby, model_id: "uuid-1"
    - version: 2, url: "http://daisy/.../v2.tar.gz", status: active,  model_id: "uuid-2"
```

影響範圍：`NWDAFContext.sharedModels`、`SharedModelInfo` 結構、`MlModelInfo.ModelUrl` → `SeriesId`、`training.go` / `trigger.go` / `monitor.go`。

此項優先級較低，建議前五項完成後再處理。

---

## 實作優先順序

| # | 項目 | 元件 | 說明 | 狀態 |
|---|------|------|------|------|
| 1 | HTTP Blocking → async callback | Daisy | 後續所有改動的基礎 | ✅ 完成（E2E 驗證 2026-03-22） |
| 2 | 模型打包 + serve tar.gz | Daisy | `server_api_handler.py` | ✅ 完成（Daisy 2026-03-20，config 格式待改進） |
| 3 | ML Service URL 下載 + cache | ML Service (Python) | HTTP URL → 下載 → artifacts cache | ✅ 完成（2026-03-24，動態 import Phase C1 完成） |
| 4 | NWDAF callback endpoint | NWDAF (Go) | 接收 Daisy 通知，驅動換版流程 | ✅ 完成 |
| 5 | MTLF/AnLF 職責分離 | NWDAF (Go) | 解耦 ML Service 直接呼叫 | ✅ 完成（2026-03-24） |
| 6 | Daisy 初始化流程 | Daisy | 移除 `--init_model` | 待規劃 |
| 7 | SharedModelRegistry Key | NWDAF (Go) | 後續架構優化 | 待實作 |

---

## 分階段實作計畫

> 責任歸屬：**我們**負責 NWDAF（Go）與 ML Service（Python）；**Daisy 團隊**負責 Daisy FL framework。

### Phase A — 打通 E2E Async 流程
**負責方**：Daisy（主）、我們（config 調整）
**前置條件**：無
**狀態**：✅ 完整完成（E2E 驗證 2026-03-22）

- ✅ NWDAF：callback endpoint、async 模式、in-flight 狀態表（2026-03-19）
- ✅ Daisy：async `publish_task` + background thread + callback POST（2026-03-20，commit `9a0ed988`）
- ✅ Daisy：訓練完成後打包 tar.gz 並提供 `/download/<tid>` HTTP endpoint
- ✅ ML Service：URL-based artifact cache，收到 HTTP model_url 可下載解壓並載入
- ✅ NWDAF：`asyncMode: true`，E2E 全程驗證通過（callback → hot-swap → accuracy monitor 重啟）

---

### Phase B — MTLF/AnLF 職責分離
**負責方**：我們（NWDAF）
**前置條件**：無（可與 Phase A 並行）
**狀態**：✅ 完成（2026-03-24）

- ✅ `AnlfService.SwapModel(newModelUrl, oldModelId)` — AnLF 負責 load new + unload old
- ✅ `MtlfService` 新增 `onModelSwapReady` callback（Wire 2），`swapModelAfterRetrain` 改呼叫 callback 取得 newModelId，不再直接操作 ML Service
- ✅ `processor.go` 串接 Wire 2：`anlf.SwapModel`
- ✅ MTLF 繼續負責 SharedModelRegistry + subscription 狀態更新

---

### Phase C — 模型 Bundle 格式定義（設計先行）
**負責方**：我們（定義規格後同步給 Daisy）
**前置條件**：無（可立即討論）
**可立即開始**：✅

這是 C1/C2 的共同前置，格式一旦確定，雙方才能獨立實作。

#### Bundle 內容

| 檔案 | 來源 | 說明 |
|------|------|------|
| `model.py` | Daisy example（git 版控） | PyTorch 模型架構定義，ML Service 固定 `from model import Model` |
| `config.yaml` | Daisy example（git 版控） | 超參數（建構 Model 用）+ 推論介面（feature names、seq_length 等） |
| `model.npy` | 訓練輸出 | 模型權重 |
| `scaler.pkl` | 訓練輸出 | 資料 Scaler（均值 / 標準差） |

#### `model.py` 格式

目前 Daisy `model.py` 與 ML Service `tcn.py` 架構完全相同，只差 class 名稱為 `TCNModel`。
為讓 ML Service 能用統一的 `from model import Model` 動態載入任意架構，提議 Daisy 在現有 `model.py` 最末端加一行 alias：

```python
# ... 現有的 Chomp1d, TemporalBlock, TemporalConvNet, TCNModel 完全不動 ...

# 統一 import 介面，ML Service 固定 from model import Model
Model = TCNModel
```

**緣由**：ML Service 不應為每種架構修改 import 路徑；約定 `Model` 為固定入口點，可讓架構自由替換而不影響 ML Service code。

#### `config.yaml` 格式

```yaml
model:
  input_size: 10           # 特徵數（= len(inference.feature_order)）
  output_size: 2           # = out_seq_len × len(inference.output_fields)
  num_channels: [32, 64, 64, 64]
  kernel_size: 2
  dropout: 0.2

inference:
  seq_length: 30           # 輸入時間步數（= dataset.py 的 seq_length）
  out_seq_len: 1           # 預測步數
  feature_order:           # 對應 dataset.py 特徵提取順序，ML Service 用此組裝輸入張量
    - total_vol
    - ul_vol
    - dl_vol
    - total_nb_pkts
    - ul_nb_pkts
    - dl_nb_pkts
    - ul_thr
    - dl_thr
    - ul_pkt_thr
    - dl_pkt_thr
  output_fields:           # 預測目標欄位（順序對應 model 輸出的 column）
    - ul_vol
    - dl_vol
  preprocessing: log1p_standard_scaler
```

**緣由**：目前 ML Service `predictor.py` 的 `_FEATURE_ORDER`、`input_window` 與 Daisy `dataset.py` 的特徵順序、`seq_length` 靠人工維持一致，任一方改動都會靜默地造成推論錯誤。改由 bundle 內的 `config.yaml` 統一聲明，ML Service 從中讀取，消除這個隱性耦合。

**ML Service 使用方式**：
- `model` 區塊 → `Model(**cfg['model'])` 建構模型
- `inference.feature_order` → 取代 `predictor.py` 的 `_FEATURE_ORDER`
- `inference.seq_length` → 取代 `ModelManager._input_window`
- `inference.output_fields` → 配合 `feature_order` 動態計算 scaler inverse transform 的 column index（取代 hardcode 的 `scaler.scale_[1:3]`）

#### Daisy 打包提議

提議 Daisy 在 example 目錄下新增 `arch/v1/` 存放架構檔案（git 版控，不透過 FL 協議下發）：

```
examples/07_MTLF_training/
└── arch/
    └── v1/
        ├── model.py      ← 現有 model.py + Model = TCNModel alias
        └── config.yaml   ← 上述格式
```

訓練結束後，打包邏輯將 `arch/v1/model.py`、`arch/v1/config.yaml` 與訓練輸出的 `model/model.npy`、`model/scaler.pkl` 合併成一個 tar.gz，透過 HTTP endpoint serve，callback 的 `model_url` 指向此 tar.gz。

task payload 新增兩個 key 讓 NWDAF 指定架構版本路徑（提議）：

```json
{
  "MODEL_DEF":    "arch/v1/model.py",
  "MODEL_CONFIG": "arch/v1/config.yaml"
}
```

**緣由**：架構版本與訓練結果分開管理，`model.py`/`config.yaml` 走 git，`model.npy`/`scaler.pkl` 走訓練流程，互不干擾。未來升版只需新增 `arch/v2/`，不影響現有訓練流程。

---

### Phase C1 — ML Service 動態載入
**負責方**：我們（ML Service）
**前置條件**：Phase C 格式確認
**狀態**：✅ 完成（2026-03-24）

#### 完成項目
- `model_manager.py` 改為 URL-based artifact cache 架構（`artifacts/<tid>/`）
- 接收 HTTP `model_url` → GET 下載 tar.gz → 解壓 → cache hit 時直接重用
- 讀 Daisy 的 `config.json`（`MODEL_PATH`、`SCALER_PATH` 檔名）→ 載入 `model.npy` + `scaler.pkl`
- `artifacts/initial/` 預填初始模型四件套（含 `model.py` + `config.json`）
- `model_manager.py` 從 bundle `config.json` 讀 `model`/`inference` 區塊初始化模型（利用 Daisy MODEL_META）
- `model_manager.py` 用 `importlib` 動態載入 bundle 內的 `model.py`，取出 `Model` class（fallback 到 `TCNModel`）
- `predictor.py` 從 model dict 讀 `feature_order`、`output_fields`；scaler inverse 改用動態 column index，移除 hardcode `scale_[1:3]`
- `artifacts/initial/model.py` 改回舊 `weight_norm` API，與 Daisy（Python 3.8）對齊
- Daisy `model.py` 新增 `Model = TCNModel` alias（commit `0db2fe78`）

#### MODEL_META 格式（Daisy commit `647f7bcc`，2026-03-24）

bundle `config.json` 格式（Daisy 從 task payload 的 `MODEL_META` merge 而來）：

```json
{
  "TID": "abc123",
  "MODEL_PATH": "model.npy",
  "SCALER_PATH": "scaler.pkl",
  "MODEL_SCRIPT": "model.py",
  "model": { "input_size": 10, "output_size": 2, "num_channels": [32,64,64,64], ... },
  "inference": { "seq_length": 30, "feature_order": [...], "output_fields": [...], ... }
}
```

> **注意**：格式是 `config.json`（含超參數），非原規劃的獨立 `config.yaml`。功能等效。

---

### Phase C2 — Daisy 模型打包 + HTTP Serve
**負責方**：Daisy
**前置條件**：Phase A + Phase C 格式確認
**狀態**：✅ 完成（2026-03-24）

- ✅ 訓練結束後將四個檔案打包為 tar.gz
- ✅ 透過 HTTP endpoint serve（供 ML Service 下載）
- ✅ callback payload 的 `model_url` 改為 HTTP URL
- ✅ `MODEL_META` 支援：task payload 帶入超參數與推論設定，merge 進 bundle `config.json`（commit `647f7bcc`）

---

### Phase C3 — NWDAF 串接收尾
**負責方**：我們（NWDAF）
**前置條件**：Phase C1 + Phase C2
**狀態**：✅ 完成（隱含，無需額外改動）

HTTP URL 傳遞路徑（`HandleTrainingComplete` → `swapModelAfterRetrain` → `onModelSwapReady` → `anlf.SwapModel`）在 Phase A 時已預留完整，C1/C2 完成後 NWDAF 側無需調整。`staticModelUrl` 是 startup trigger 的初始模型用途，非 fallback，保留不動。

---

### Phase D — SharedModelRegistry Key 重設計
**負責方**：我們（NWDAF）
**前置條件**：Phase B、Phase C 穩定後
**優先級**：最低

目前以 model URL 為 key，無法支援多版本並存與 rollback。改以 series ID（如 `"ue-comm-group1"`）為 key，版本記錄下放至 value 內。影響範圍廣（`NWDAFContext`、`SharedModelInfo`、`training.go`、`trigger.go`、`monitor.go`），最後處理。

---

### 彙整

| Phase | 負責方 | 前置條件 | 狀態 |
|-------|-------|---------|------|
| A | Daisy + 我們 | 無 | ✅ 完成（2026-03-22） |
| B | 我們（NWDAF） | 無 | ✅ 完成（2026-03-24） |
| C（格式定義） | 我們 | 無 | ✅ 規格已定（Daisy 以 MODEL_META in config.json 實作） |
| C1 | 我們（ML Service） | C | ✅ 完成（2026-03-24） |
| C2 | Daisy | A + C | ✅ 完成（2026-03-24，commit `647f7bcc`） |
| C3 | 我們（NWDAF） | C1 + C2 | ✅ 完成（隱含） |
| D | 我們（NWDAF） | B、C | 待實作 |

---

## 完成記錄

### Phase B + Phase C1 完成（2026-03-24 續）

**Phase B — MTLF/AnLF 職責分離**：`swapModelAfterRetrain` 移除直接 ML Service 呼叫，改透過 `onModelSwapReady` callback 委派給 AnLF。AnLF 新增 `SwapModel(newModelUrl, oldModelId)`，負責 load new + unload old。MTLF 繼續管 SharedModelRegistry 和 subscription 狀態更新。

**Phase C1 — ML Service 動態載入**：`model_manager.py` 改用 `importlib` 動態載入 bundle 內的 `model.py`（fallback 到 `TCNModel`）。`predictor.py` 移除 hardcode `_FEATURE_ORDER` 和 scaler index `[1:3]`，改從 bundle `inference` 區塊讀 `feature_order`、`output_fields` 動態計算。`artifacts/initial/model.py` 改回舊 `weight_norm` API 與 Daisy（Python 3.8）對齊。

**雜項修正（ML Service）**：`run.py` 改在 `import torch` 前設 `NNPACK_DISABLE=1`；suppress `weight_norm` FutureWarning；`ModelManager init` log 改為 `default params` 說明。

**相關 commits（NWDAF）**：
- `f4e8dbd` refactor: delegate ML Service ops to AnLF during hot-swap

**相關 commits（ML Service）**：
- `882d4c2` feat: dynamic model.py loading and bundle-driven inference config

**Daisy 更新**：`examples/07_MTLF_training/model.py` 新增 `Model = TCNModel` alias（commit `0db2fe78`）

---

### Sync Mode 移除 + MODEL_META 支援 + Race Fix（2026-03-24）

**Sync mode 移除**：`AsyncMode` config 欄位與全部 sync 分支（`TriggerTraining`、`DaisyDefaultTimeout`、`newModelUrlFromCfg` 等）清除，Daisy 通訊統一為 async-only。

**MODEL_META 支援**（Daisy commit `647f7bcc`）：
- NWDAF `nwdafcfg.yaml` task 加入 `MODEL_META` block（超參數 + 推論介面）
- ML Service `model_manager.py` 從 bundle `config.json` 讀 `model`/`inference` 區塊初始化模型
- `artifacts/initial/config.json` 補上對應欄位，與 Daisy 產出格式一致

**重複 model ID 修正**：兩個 subscription 同時呼叫 `InitializeMlModel` 時會產生 race condition，導致 ML Service 載入兩份相同模型。新增 `LoadDone()`/`WaitLoaded()` 機制：`isNew=false` 的 goroutine 等待第一個載入完成後直接重用，不再重複呼叫 ML Service。

**相關 commits（NWDAF）**：
- `c5d18c8` refactor: remove sync Daisy mode, async-only
- `38e659e` feat: add MODEL_META to Daisy task payload
- `f2a8b9f` fix: prevent duplicate ML model loading via WaitLoaded

**相關 commits（ML Service）**：
- `9e33375` chore: remove unused legacy artifacts and fix docs
- `2eb5d3e` feat: read model/inference params from bundle config.json

---

### Phase A E2E 完整驗證（2026-03-22）

NWDAF → Daisy → ML Service 完整 async 流程驗證通過。

**驗證結果**（server 環境）：
- Daisy callback 正確 POST 至 `/mtlf/training-complete`
- MTLF 收到 `model_url`（`http://192.168.127.5:9887/download/<tid>`）
- ML Service 下載 tar.gz → 解壓至 `artifacts/<tid>/` → 載入成功（23ms）
- 舊模型 unload、SharedModelRegistry 更新、兩條 subscription 切換至新 model ID
- 舊 accuracy monitor 退出，新 monitor 啟動（warmup 155s）

**相關 commits（ML Service）**：
- `9bfcdfb` feat: URL-based artifact cache + artifacts/initial/ 四件套
- `34aa59a` fix: suppress startup warnings（sklearn / weight_norm / NNPACK）
- `be62f9e` fix: disable NNPACK in run.py before any torch usage

**相關 commits（NWDAF）**：
- `6edb42f` feat: adopt model bundle cache scheme（staticModelUrl、task config、timeout）

---

### #4 NWDAF Async Callback Endpoint（2026-03-19）

對應需求文件 #1 的 NWDAF 側實作 + #4 完整實作。commit: `feat(mtlf): add async Daisy callback mode for non-blocking training`

**新增/修改檔案：**

| 檔案 | 變動內容 |
|------|---------|
| `pkg/factory/config.go` | `MtlfConfig` 新增 `AsyncMode bool` 欄位 |
| `config/nwdafcfg.yaml` | 新增 `asyncMode: false`（附說明註解） |
| `internal/sbi/consumer/daisy_service.go` | 新增 `TriggerTrainingAsync`：產生新 UUID TID、注入 `CALLBACK_URL`、預期 202 回應 |
| `internal/sbi/consumer/daisy_service_test.go` | 新增 3 個 async 測試案例 |
| `internal/mtlf/mtlf.go` | 新增 `inFlight sync.Map` 欄位 |
| `internal/mtlf/training.go` | 新增 `buildCallbackURL`、`triggerTrainingAsync`、`HandleTrainingComplete`；`swapModelAfterRetrain` 改為接受 `newModelUrl` 參數；整體邏輯依 `asyncMode` 分支 |
| `internal/mtlf/training_test.go` | 新增 5 個 `HandleTrainingComplete` 測試案例 |
| `internal/sbi/api_daisy_callback.go` | 新增 `POST /mtlf/training-complete` endpoint |
| `internal/sbi/server.go` | 新增 `/mtlf` route group 註冊 |
| `internal/sbi/processor/processor.go` | 新增 `HandleDaisyCallback` 委派至 `mtlf.HandleTrainingComplete` |
| `test/fake_daisy/fake_daisy_server.py` | 新增 Fake Daisy server（FastAPI）：立即回 202、背景延遲後 POST callback |
| `test/fake_daisy/pyproject.toml` | Fake server 依賴（fastapi、uvicorn、httpx） |

**設計要點：**
- `asyncMode: false`（預設）保留原有阻塞行為，不影響現有流程
- async 路徑中 TID 一律覆寫為新 UUID，防止 `inFlight` key 重複
- `buildCallbackURL` 使用 `RegisterIPv4`（fallback `BindingIPv4`）+ Port 組合，確保 Daisy 能回呼外部可達的位址
- Daisy 開發者需對應實作的項目已整理至 `.agent/docs/daisy_async_callback_implementation.md`
