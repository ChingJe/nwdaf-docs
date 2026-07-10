# Phase 2 Model Lifecycle Migration

Date: 2026-07-10

Status: Completed

Implementation commits:

- `NWDAF/`: `62e2f9f` (`refactor(anlf): delegate model lifecycle to backend`)
- `PyAnLF/`: `7a2ebc4` (`feat(runtime): manage subscription model lifecycle`)

Parent plan:

- `nwdaf-docs/docs/plans/anlf-backend-transition/AnLF Backend Transition Plan.md`

---

## 1. Purpose

這份文件定義 `AnLF Backend Transition` 的 `Phase 2` 細部規劃。

本 phase 的目的，是把 `AnLF` 的 model lifecycle ownership 從 `NWDAF/`
Go 端逐步下沉到 `PyAnLF/` backend。

這裡所說的 lifecycle，不只是單純的 `load / unload` 呼叫，而是包含：

1. model provision 後的 activation
2. 已使用模型的 replacement
3. model usage 與 subscription 關聯
4. replacement 失敗時的保底策略
5. model reference 與 active runtime 之間的取得、準備、切換過程

本 phase 的重點是先搬移 model lifecycle 的主責任，不提前把 analytics
runtime、prediction data path、accuracy workflow 一起混進來。

---

## 2. Scope

本 phase 處理以下事項：

1. 重新定義 `NWDAF/` 與 `PyAnLF/` 之間的 model lifecycle contract
2. 將 `MTLF` provision callback 後的 activation/replacement 主要責任下沉到
   `PyAnLF/`
3. 將 shared model runtime state 的主要 owner 轉為 `PyAnLF/`
4. 讓 Go 端改以 apply/release oriented lifecycle API 與 backend 互動，而不是直接下
   low-level `load / unload / swap` 命令
5. 定義 replacement 失敗時保留舊模型的行為語意
6. 將 `Predict` 的 key 從 `modelId` 改為 `subscriptionId`
7. 規劃本 phase 需要的 Go tests、PyAnLF tests、以及 cross-repo interaction
   tests

本 phase 不處理以下事項：

1. prediction runtime ownership 下沉
2. historical data shaping 與 prediction request shaping 搬移
3. Go 端向 `SMF`、`UPF` 等 5GC components 收資料的流程改造
4. analytics result formatting 搬移
5. accuracy monitor、ground truth matching、retrain input logic 搬移
6. 以多語言 backend 抽象化為目的的新插件框架

---

## 3. Planning Basis

本文件基於以下本地來源整理：

1. `AGENTS.md`
2. `nwdaf-docs/docs/development_policy.md`
3. `free5gc-dev-skill/SKILL.md`
4. `NWDAF/` current tree on 2026-07-10
5. `PyAnLF/` current tree on 2026-07-10
6. `AnLF Backend Transition Plan.md`
7. local 3GPP specs under `nwdaf-docs/specs/`

---

## 4. Agreed Decisions

本 phase 已先確認以下方向：

1. `AnlfBackend` 採 subscription-oriented API，不採 command-oriented API
2. backend contract 欄位盡量貼近 NWDAF subscription 與 provision 的既有語意
3. Go 端仍保留 5GC-facing data collection 與 orchestration
4. Go 端不需要持有實際 active model identity 作為主要 runtime state
5. 若 Go 端日後需要觀察 backend runtime state，應透過 query API 取回，而不是重新把 state 放回 Go context
6. 新模型 replacement 失敗時，應保留舊模型繼續服務
7. replacement failure 應在 `NWDAF/` 與 `PyAnLF/` 兩端都留下清楚 log
8. 本 phase 僅處理 model lifecycle ownership，不先改 prediction data path
9. `Predict` 名稱保留，但 key 由 `modelId` 改為 `subscriptionId`
10. lifecycle 核心操作先收斂成 `ApplySubscriptionRuntime`、
    `ReleaseSubscriptionRuntime`、`Predict` 三個主語意；runtime query 可作為 optional
    輔助能力
11. static/default model selection 屬於 `PyAnLF` 本地設定，不屬於 Go-side contract
12. Daisy retrain completion 在 `NWDAF` 內部轉譯後，應與 `MTLF` provision callback
    收斂成同一條 `ApplySubscriptionRuntime` 更新入口

---

## 5. Pre-implementation State

目前 `NWDAF/` 雖然已經在 Phase 1 收斂為 `AnlfBackend` 命名，但 model lifecycle
主責任仍大多在 Go 端。

### 5.1 Current Go-side Responsibilities

目前仍主要留在 Go 端的 lifecycle 相關責任包括：

1. 解析 `MTLF` callback 後的 activation actions
2. 判斷 model URL 是否已存在於 shared model registry
3. 決定是否要進行 load、reuse、或 replacement
4. 維護 `MlModelInfo` 與 `SharedModelInfo`
5. 直接呼叫 backend 的 `LoadModel` / `UnloadModel`
6. 在 replacement 路徑中處理舊模型 detachment 與 monitor 啟停

### 5.2 Current Model Sources

目前模型相關材料至少有三種來源，但它們不應全部成為 Go-side contract 的獨立語意：

1. static/default model selection
   - 應屬於 `PyAnLF` 本地設定與啟動策略
   - 不應由 Go 端主動下發一個 static URL 作為 apply API
2. external `MTLF` model provision
   - subscription 建立後先向 `MTLF` 訂閱
   - 之後由 callback 帶回新的 model reference
3. Daisy retrain completion
   - retrain 完成後拿到新的 model reference
   - 應先在 `NWDAF` 內部轉譯，再收斂成和 `MTLF` provision 相同語意的更新材料

因此，對 `NWDAF -> PyAnLF` contract 來說：

1. static/default model selection 不應成為獨立 apply source
2. 真正需要進 API 的，是外部模型材料更新事件

### 5.3 Current Backend Responsibilities

目前 `PyAnLF/` 或現有 backend contract 所暴露的能力，仍偏向 low-level executor：

1. `LoadModel`
2. `UnloadModel`
3. `Predict`

也就是說，backend 目前知道如何做某些單一步驟，但不知道整體 subscription /
provision lifecycle。

### 5.4 Current Structural Problem

這種切分的主要問題是：

1. lifecycle policy 仍由 Go 端主導，backend 只是被動執行
2. shared model usage state 分散在 Go 端，未來很難讓 backend 自主處理
3. `SwapModel` 這類步驟化語意讓 Go 端仍知道太多 backend 細節
4. `Predict(modelId, ...)` 綁死 Go 端必須知道 active model
5. 若日後 prediction runtime 也要下沉，現在的低階 contract 會變成阻礙

---

## 6. Target Boundary

### 6.1 Goal

本 phase 完成後，`NWDAF/` 應只表達：

1. 哪個 subscription 存在
2. 該 subscription 對應到什麼 analytics / event / provision 資訊
3. 某個 subscription 的 lifecycle 發生了什麼狀態變化

而 `PyAnLF/` 應負責根據這些狀態，決定：

1. 是否要取得模型
2. 是否要載入模型
3. 是否可重用既有模型 runtime
4. 是否需要進行 replacement
5. 哪些 subscription 仍在使用某模型
6. 哪個 replacement 失敗後要保留舊模型

### 6.2 Ownership Split

本 phase 的責任切分目標如下：

`NWDAF/` 保留：

1. 5GC-facing subscription handling
2. 與 `SMF`、`UPF` 等資料來源互動
3. `MTLF` callback ingress
4. Daisy retrain completion ingress 與其到 provision-style update 的轉譯
5. NWDAF-local correlation 與 orchestration
6. 對 backend 發送 lifecycle apply/release requests

`PyAnLF/` 接手：

1. model activation runtime
2. model replacement runtime
3. model acquisition from provisioned model reference
4. model usage tracking
5. model release timing
6. lifecycle failure handling 與 fallback

### 6.3 Explicit Boundary Rule

本 phase 明確不要求：

1. Go 端停止呼叫 `Predict`
2. backend 直接接手 5GC data collection
3. Go 端在這個 phase 失去所有與 analytics 有關的流程

換句話說，本 phase 的成果應是「lifecycle ownership 已下沉，但 prediction data
path 暫時還是舊路徑」。

---

## 7. API Design Direction

### 7.1 Design Principle

本 phase 不應繼續把 `LoadModel`、`UnloadModel`、`SwapModel` 當成長期主介面，也不應
直接從 3GPP 規格照抄一組外部 SBI endpoint 到 backend contract。

原因是：

1. 這些名稱描述的是 backend 內部步驟，而不是 Go 端真正要表達的 intent
2. 一旦 lifecycle policy 留在 Go 端，ownership 實際上就沒有完成下沉
3. 後續若有多模型選擇、subscription-level policy、或更完整的 runtime state，
   這種 low-level API 會讓 Go 端被迫知道更多細節
4. 這條 contract 的本質是 `NWDAF(go) -> AnLF backend` 的內部 API，不是對外 3GPP
   service API

因此，本 phase 要先定的是：

1. 核心操作語意
2. 何時呼叫這些操作
3. 每個操作需要什麼 spec-aligned 資料

而不是先把 path 寫死成某種唯一風格。

### 7.2 Contract Style Direction

Go 端對 backend 的主要輸入，應以 subscription / provision context 為中心，而不是
以 model operation step 或抽象 generic state 為中心。

語意上應偏向以下方向：

1. backend 接收某個 subscription 的 runtime apply request
2. backend 根據該 subscription 的 analytics type、granularity、event 與
   provisioned model reference，自行決定實際 lifecycle 操作
3. subscription 釋放時，Go 端發出明確的 release request
4. Go 端若需要知道目前狀態，再透過 query API 取回

### 7.3 Input Shape Direction

contract 欄位應盡量貼近現有 subscription 與 provision 語意，例如：

1. subscription identity
2. notification correlation
3. analytics event / analytics type
4. granularity 或其他與模型選擇可能相關的規格欄位
5. model provision callback 交付的 model reference 與相關 metadata
6. retrain completion 轉譯後的模型更新資訊

是否完全重用現有 OpenAPI model，或只擷取其中必要欄位形成 backend request
struct，應在實作時以可維護性決定；但語意上應維持與規格欄位接近，不應先轉為
一套過度 backend-specific 的命名。

### 7.4 Query Direction

本 phase 可允許預留 query-style API，用於日後觀察 backend runtime state，例如：

1. 某 subscription 當前 lifecycle state
2. 某 provision 是否已成功生效
3. 某 replacement 是否 fallback 到舊模型

但 query API 不是本 phase 的主要成果。若實作上沒有立即需要，可先只定介面方向。

### 7.5 Evidence From Current Code And Local Specs

在進一步定 API 之前，先明確目前有哪些資訊來源可用。

#### 7.5.1 Current NWDAF-side Subscription State

目前 `NWDAF/` 會在本地 context 中保存：

1. `subscriptionId`
2. `notifCorrId`
3. `eventSubs`
4. `evtReq`

也就是說，Go 端已經持有一份接近 `NnwdafEventsSubscription` 的 runtime snapshot，
不需要再從別處回推這些資料。

#### 7.5.2 Current MTLF Subscription Request

目前 `NWDAF/` 對 `MTLF` 發出的 `NwdafMLModelProvSubsc`，實際只帶：

1. `mLEventSubscs[].mLEvent`
2. `mLEventSubscs[].tgtUe`
3. `notifUri`
4. `notifCorreId`

因此，若 backend 在 lifecycle 決策時需要更多 analytics context，不能假設它會從
`MTLF` callback 完整拿回原始 consumer subscription，而應由 `NWDAF/` 主動在內部
contract 中補上必要的 subscription snapshot。

#### 7.5.3 Current MTLF Callback Payload

目前 `AnLF` callback 實際吃到的 `NwdafMLModelProvNotif` / `MLEventNotif` 主要欄位包括：

1. top-level `subscriptionId`
2. per-event `notifCorreId`
3. per-event `event`
4. per-event `mLFileAddr.mLModelUrl`

本地 YAML 額外還定義了對 lifecycle 可能有用，但目前尚未實際使用的欄位，例如：

1. `modelUpdateInd`
2. `useCaseCxt`
3. `mLEventFilter`
4. `tgtUe`
5. `modelUniqueId`
6. `modelProviderId`
7. `validityPeriod`
8. `spatialValidity`

這代表 Phase 2 的 backend contract 不需要被現況綁死在只有 `modelUrl` 的極簡型態，
而可以保留 spec-aligned 欄位空間。

#### 7.5.4 Current Daisy Callback Payload

目前 Daisy callback 實際只提供：

1. `task_id`
2. `model_url`
3. `status`
4. `error`

這些資訊足以支援 `MTLF` 端的 retrain completion handling，但不足以單獨構成
`AnLF` lifecycle contract。

因此，Phase 2 的 AnLF backend API 應以：

1. `NWDAF` 原始 subscription snapshot
2. `MTLF` model provision notification
3. retrain completion 轉譯後的模型更新資訊

作為主要語意來源，而不是直接以 Daisy callback shape 為基礎。

### 7.6 Model Reference Principle

本 phase 應明確把 model reference 視為 lifecycle 的核心概念之一。

在目前 spec 與現有程式裡，最重要的 provisioned model reference 是：

1. `MLEventNotif.mLFileAddr.mLModelUrl`

它的語意不是「模型已經可直接拿來推論」，而是：

1. `MTLF` 或 provider 告知 `AnLF` 模型可從哪裡取得
2. backend 根據這個 reference 自行下載、準備、載入、替換
3. 只有在 backend 完成這些步驟後，active runtime 才真正更新

因此本 phase 應區分兩種不同語意：

1. provisioned model reference
   - 例如 `mLModelUrl`
   - 描述模型取得位置
2. active runtime state
   - backend 內部真正正在服務的模型狀態
   - 不應再作為 Go-side 主語意

這個區分會直接影響 replacement/fallback：

1. 新的 `mLModelUrl` 到來，不代表可立即切換
2. backend 需要先驗證其可取得、可載入、可生效
3. 若失敗，必須保留舊 active runtime

### 7.7 Core Operations

基於目前討論，本 phase 先收斂三個核心操作與一個 optional 操作。

必要操作：

1. `ApplySubscriptionRuntime`
2. `ReleaseSubscriptionRuntime`
3. `Predict`

optional 操作：

1. `GetSubscriptionRuntimeState`

這些是操作語意，不等於 path 已經最終定案。

例如實作上可以長成：

1. REST-like resource paths
2. action-style endpoints
3. 其他等價 internal API 形狀

但不管 path 最後怎麼命名，語意上都應對齊這三個核心操作。

### 7.8 Event To Operation Mapping

本 phase 文件應以「什麼時候會呼叫 backend API」來理解，而不是只看 endpoint 名稱。

目前至少有以下事件需要被承接：

1. `SubscriptionRegistered`
   - consumer subscription 剛建立
   - backend 需要知道這個 subscription runtime 存在
2. `ProvisionMaterialUpdated`
   - `MTLF` callback 帶來新的 `mLModelUrl`
   - Daisy retrain completion 經 `NWDAF` 轉譯後，帶來新的 model reference
3. `SubscriptionReleased`
   - subscription delete
   - 或 update 前清除舊 runtime
4. `PredictionRequested`
   - Go 端已收集 historical data，準備請 backend 推論

對應關係建議如下：

1. `SubscriptionRegistered`
   - backend 需要知道 subscription runtime 已存在
   - 是否在這個事件就呼叫 `ApplySubscriptionRuntime`，取決於實作時是否要讓 backend
     先建立空 runtime；至少不應在這個事件下發 static model material
2. `ProvisionMaterialUpdated`
   - 呼叫 `ApplySubscriptionRuntime`
   - 由 backend 決定 activation / reuse / replacement
3. `SubscriptionReleased`
   - 呼叫 `ReleaseSubscriptionRuntime`
4. `PredictionRequested`
   - 呼叫 `Predict`

其中最重要的收斂是：

1. `MTLF` provision callback
2. Daisy retrain completion

雖然來源不同，但在 backend contract 的語意上，都應收斂成同一類
`ProvisionMaterialUpdated` 事件。

### 7.9 Concrete Contract Proposal

本 phase 建議以「subscription runtime」作為 backend 端的主要資源語意，但不要求現在
把 path 寫成唯一固定形式。

#### 7.9.1 ApplySubscriptionRuntime

這是本 phase 最核心的一個操作。

它需要能覆蓋：

1. subscription 剛建立，但還沒有模型
2. `MTLF` callback 後拿到第一個模型下載 URL
3. retrain completion 經轉譯後拿到新的模型 reference
4. backend 收到外部模型材料更新後重新 reconcile runtime
5. subscription update 後重新 reconcile runtime

也就是說，不應因為外部模型材料來源不同，就拆成不同 API。

#### 7.9.2 ReleaseSubscriptionRuntime

這個操作負責：

1. backend 移除該 subscription 的 runtime 關聯
2. backend 判斷模型是否仍被其他 subscriptions 使用
3. 只有在不再被使用時，才真正釋放模型 runtime

#### 7.9.3 Predict

這個操作保留名稱，但 key 改為 `subscriptionId`。

Go 端仍負責：

1. historical data collection
2. historical data shaping

backend 則負責：

1. 依 `subscriptionId` 找出 active runtime
2. 以當前 active runtime 執行 prediction
3. 若新模型替換失敗但舊模型仍可用，則自動落回舊模型

#### 7.9.4 GetSubscriptionRuntimeState

這是 optional 操作，用於：

1. debug
2. 觀察 backend 是否 ready
3. 檢查是否 fallback 到舊模型

它不是本 phase 最小完成條件，但文件應保留這個方向。

### 7.10 Apply Request Shape

`ApplySubscriptionRuntime` 建議 request 不直接重用完整 `NnwdafEventsSubscription` 或
完整 `NwdafMLModelProvNotif`，而是定義一組 spec-aligned、但經過收斂的 request struct。

理由是：

1. backend 不需要 consumer-facing `notificationURI`
2. backend 不需要 NWDAF 回應 consumer 時才會用到的 `eventNotifications`
3. backend 需要的是 lifecycle decision 所需的最小充分資訊

建議主 request 結構如下：

```json
{
  "subscription": {
    "subscription_id": "sub-123",
    "notif_corr_id": "sub-123",
    "evt_req": {},
    "event_subscriptions": []
  },
  "provision_context": {
    "source": "MTLF_PROVISION",
    "mtlf_subscription_id": "mtlf-sub-1",
    "notif_subscription_id": "mtlf-sub-1",
    "ml_event_notif": {}
  }
}
```

這裡不再要求額外的 generic `desiredState`。

原因是：

1. 本 phase 的 lifecycle 操作已經明確拆成 `apply` 與 `release`
2. `apply` 本身就已經是明確的意圖
3. 加上一個抽象 state 只會讓 API 更難理解，卻沒有帶來相稱的設計收益

### 7.11 Minimum Required Fields

為了避免 scope 失控，本 phase 應區分最低必要欄位與預留欄位。

#### 7.11.1 Minimum Fields For Phase 2

`subscription` 區塊最低建議包含：

1. `subscription_id`
2. `notif_corr_id`
3. `evt_req`
4. `event_subscriptions`

其中 `event_subscriptions` 最低優先保留：

1. `event`
2. `tgtUe`
3. `exptAnaType`
4. `temporalGranSize`
5. `repetitionPeriod`
6. `snssaia`
7. `dnns`
8. `appIds`
9. `networkArea`

`provision_context` 區塊最低建議包含：

1. `source`
   - `MTLF_PROVISION`
2. `mtlf_subscription_id`
3. `notif_subscription_id`
4. `ml_event_notif`

其中 `ml_event_notif` 最低優先保留：

1. `event`
2. `notifCorreId`
3. `mLFileAddr`
4. `modelUpdateInd`

#### 7.11.2 Reserved Spec-aligned Fields

以下欄位可在 Phase 2 先保留語意方向，不強制一開始全部用上：

1. `useCaseCxt`
2. `mLEventFilter`
3. `tgtUe` in `MLEventNotif`
4. `modelUniqueId`
5. `modelProviderId`
6. `validityPeriod`
7. `spatialValidity`

這些欄位仍值得保留在設計裡，因為它們未來可能影響：

1. 多模型選擇
2. runtime traceability
3. replacement policy

### 7.12 Backend-local Default Model Policy

static/default model selection 不屬於 `NWDAF -> PyAnLF` contract 的責任。

它應屬於 `PyAnLF` 自己的本地設定，例如：

1. 本地模型路徑
2. 預設模型名稱或 alias
3. backend 自己可解析的模型來源設定

因此：

1. Go 端不應再持有 `staticModelUrl` 這類模型管理設定作為長期設計
2. Go 端不應特地發一條 `static model apply` API
3. 若 backend 啟動時要先載入預設模型，應由 backend 自行決定與管理

### 7.13 Response And Error Direction

`ApplySubscriptionRuntime` 的 response 應回傳最小但足夠記錄的結果。

建議回傳：

1. `subscription_id`
2. `runtime_state`
3. `result`
4. `fallback_applied`
5. `message`

其中 `result` 可先定義為：

1. `ACTIVATED`
2. `REUSED`
3. `REPLACED`
4. `FAILED_USING_PREVIOUS`
5. `FAILED_NO_PREVIOUS`

這樣 `NWDAF/` 才能在不重新理解 backend 內部步驟的情況下，正確記錄 log 與 error
outcome。

### 7.14 Predict Contract Transition

本 phase 雖然不搬 prediction data path，但 `Predict` contract 本身需要同步調整，
否則 lifecycle owner 下沉後，Go 端仍會被迫持有 active `modelId`。

因此本 phase 明確採以下方向：

1. `Predict` 名稱保留
2. `Predict` 的主鍵由 `modelId` 改為 `subscriptionId`
3. active model resolution 改由 backend 內部處理

實作上可以是：

1. `POST /subscriptions/{subscriptionId}/predict`
2. 或其他等價的 internal API 形狀

但 request 語意應接近：

```json
{
  "historical_data": []
}
```

而不是：

```json
{
  "model_id": "model-123",
  "historical_data": []
}
```

這代表：

1. Go 端仍負責 historical data collection 與 shaping
2. backend 根據 `subscriptionId` 找到目前 active runtime
3. 若 replacement 失敗但舊模型仍有效，`Predict` 應自動落到舊模型

至少應定義以下錯誤語意：

1. `SUBSCRIPTION_NOT_FOUND`
2. `MODEL_NOT_READY`
3. `NO_ACTIVE_MODEL`

### 7.15 API Examples

以下例子只是在說明語意，不代表 path 命名已經最終定案。

#### 7.15.1 MTLF Provision Apply

```json
{
  "subscription": {
    "subscription_id": "sub-123",
    "notif_corr_id": "sub-123",
    "evt_req": {
      "notifMethod": "PERIODIC",
      "repPeriod": 10
    },
    "event_subscriptions": [
      {
        "event": "UE_COMMUNICATION",
        "tgtUe": {
          "supis": ["imsi-001010000000001"]
        },
        "exptAnaType": "COMMUN"
      }
    ]
  },
  "provision_context": {
    "source": "MTLF_PROVISION",
    "mtlf_subscription_id": "mtlf-sub-1",
    "notif_subscription_id": "mtlf-sub-1",
    "ml_event_notif": {
      "event": "UE_COMMUNICATION",
      "notifCorreId": "sub-123",
      "mLFileAddr": {
        "mLModelUrl": "http://mtlf.example/models/model-a.onnx"
      },
      "modelUpdateInd": false
    }
  }
}
```

#### 7.15.2 Daisy Retrain Completion Translated To Provision-style Apply

這個例子要表達的是：Daisy callback 本身不直接成為另一套 backend API，而是先在
`NWDAF` 內部被轉譯成與 `MLModelProvision` 同語意的更新材料，再呼叫同一條
`ApplySubscriptionRuntime`。

```json
{
  "subscription": {
    "subscription_id": "sub-123",
    "notif_corr_id": "sub-123",
    "event_subscriptions": [
      {
        "event": "UE_COMMUNICATION"
      }
    ]
  },
  "provision_context": {
    "source": "MTLF_PROVISION",
    "ml_event_notif": {
      "event": "UE_COMMUNICATION",
      "mLFileAddr": {
        "mLModelUrl": "http://daisy.example/models/model-b.onnx"
      },
      "modelUpdateInd": true
    }
  }
}
```

#### 7.15.3 Release

`ReleaseSubscriptionRuntime` 的最小語意是：

1. `sub-123` 不再需要 active runtime
2. backend 解除 usage 關聯
3. 若模型仍被其他 subscriptions 使用，則不卸載

#### 7.15.4 Predict

```json
{
  "historical_data": []
}
```

---

## 8. Lifecycle Semantics

### 8.1 Activation

當 `MTLF` provision callback 或 Daisy retrain completion 轉譯後帶來新的模型
reference 時，Go 端應：

1. 完成 ingress parsing 與基本 correlation
2. 將 subscription/provision context 同步給 backend
3. 不再自行主導 `load / reuse / swap` 細節

backend 應：

1. 根據該 subscription 目前的 runtime state 判斷是否首次 activation
2. 根據 provisioned model reference 取得模型
3. 決定是否共用既有模型 runtime 或建立新模型 runtime
4. 在成功後更新 backend 內部 usage state
5. 在需要時回傳可供 NWDAF 記錄的 activation outcome

### 8.2 Replacement

當既有 subscription 已有 active runtime，而新的 provision 指向不同模型時：

1. Go 端只負責通知 provision material 已更新
2. backend 決定是否需要 replacement
3. backend 應在 replacement 完成前保有舊模型的可服務能力
4. replacement 的具體步驟不再作為 Go-side API 語意暴露

### 8.3 Failure Handling

replacement 失敗時，正式語意如下：

1. 舊模型應繼續保留並維持服務
2. 新模型不得在 failure path 中取代舊模型成為 active runtime
3. backend 應清楚記錄 replacement failure 與 fallback 結果
4. `NWDAF/` 端也應收到可記錄的 failure outcome，以便留下對應 log

### 8.4 Release

當 subscription 結束、取消、或不再需要某 analytics runtime 時：

1. Go 端應同步 release-oriented request 給 backend
2. backend 根據其內部 usage tracking 判斷某模型是否仍被其他 subscriptions 使用
3. 只有在不再被使用時，backend 才真正釋放該模型 runtime

---

## 9. Planned Implementation Areas

### 9.1 NWDAF Repository

預期修改區域包括：

1. `internal/anlf/`
   - 將現有 lifecycle entrypoints 改為 apply/release oriented backend invocation
   - 收斂 `SwapModel` 與 low-level model operation 的對外語意
   - 將 `Predict` 的 key 從 `modelId` 轉為 `subscriptionId`
2. `internal/anlf/processor/`
   - 讓 callback 後流程偏向整理 apply request，而不是直接執行本地 model lifecycle
3. `internal/notifier/`
   - 只在必要範圍內調整對 lifecycle state 的假設
4. `internal/context/`
   - 重新界定 `MlModelInfo`、`SharedModelInfo` 哪些欄位仍需保留在 Go 端
5. `internal/mtlf/`
   - 將 retrain completion 後的模型替換轉譯為與 `MTLF` provision 同語意的
     provision material update
6. `pkg/service/` 與相關 wiring
   - 若 backend contract 初始化方式需要擴充，應在這裡同步調整

### 9.2 PyAnLF Repository

預期修改區域包括：

1. lifecycle-oriented API routes / handlers
2. backend-side subscription state store
3. backend-side model acquisition from `mLModelUrl`
4. backend-side model usage tracking
5. replacement with fallback runtime behavior
6. subscription-keyed prediction routing
7. API tests 與 backend domain unit tests

### 9.3 Migration Order

本 phase 內部建議按以下順序落地：

1. 先定 `ApplySubscriptionRuntime` / `ReleaseSubscriptionRuntime` / `Predict`
   contract
2. 將 `Predict` 的 key 從 `modelId` 改為 `subscriptionId`
3. 讓 `MTLF` callback、retrain completion 轉譯後的更新材料走同一條 apply contract
4. 收縮 Go-side shared model runtime state
5. 視需要再補 runtime query API

### 9.4 Contract Migration

現有 `LoadModel`、`UnloadModel`、`Predict` 不應在本 phase 立即粗暴刪除。

較合理的過渡策略應是：

1. 先新增新的 lifecycle contract
2. 讓 Go 端 activation / replacement / release 路徑改走新 contract
3. 將 `Predict` 的 contract key 從 `modelId` 改為 `subscriptionId`
4. 視實作需要，暫時保留 `LoadModel` / `UnloadModel` 作為 backend 內部或過渡用途
5. 等 Phase 3 明確接手 prediction runtime 後，再決定哪些舊 API 可以正式移除

---

## 10. Explicit Non-goals

以下事項即使看起來相關，也不應在本 phase 一起納入：

1. 將 prediction request input 改成直接把歷史資料流推給 backend
2. 將 `models.UeCommunication` 的生成責任搬到 `PyAnLF/`
3. 將 accuracy monitor 與 ground truth matching 一起搬移
4. 重寫 `NWDAF/` 內部所有 analytics handler
5. 引入多 backend runtime selector 或 plugin registry

若實作過程中發現某個 lifecycle 設計必然會影響 prediction data path，應先把影響
控制在 contract 相容層，而不是直接把 Phase 3 內容提前做掉。

---

## 11. Verification

本 phase 需要明確承接主文件列出的三類驗證，但可以分層完成。

### 11.1 NWDAF-side Verification

至少應補齊：

1. Go unit tests
   - 驗證 `MTLF` callback 與 retrain completion 轉譯後的兩種入口形成的 apply
     request 是否正確
   - 驗證 subscription release 時 backend invocation 是否正確
   - 驗證 `Predict` 已改用 `subscriptionId`
   - 驗證 Go context 在 lifecycle owner 下沉後仍保留必要 correlation state

### 11.2 PyAnLF-side Verification

至少應補齊：

1. backend domain unit tests
   - 驗證 activation、reuse、replacement、release 的核心 state transitions
   - 驗證 replacement failure 時保留舊模型
   - 驗證 model acquisition from `mLModelUrl`
2. API-level tests
   - 驗證 lifecycle endpoint request/response contract
   - 驗證錯誤輸入與 failure outcome 的回應語意

### 11.3 Cross-repo Verification

至少應規劃一條最小 interaction test，驗證：

1. `NWDAF/` 可將 provision-driven lifecycle context 正確送到 `PyAnLF/`
2. `PyAnLF/` 可回應成功 activation 或 failure-with-fallback outcome
3. Go 端對應 log / outcome handling 與新語意一致

若當前環境不足以完成完整實機驗證，文件中仍應明確區分：

1. 已完成的 unit/API tests
2. 尚待可運行環境補做的 interaction validation

---

## 12. Completion Criteria

本 phase 完成時，應同時滿足以下條件：

1. `MTLF` provision callback 後的主要 lifecycle policy 已不再由 Go 端逐步拼裝
2. `MTLF` callback 與 retrain completion 轉譯後的更新材料已收斂到同一類 backend
   apply 語意
3. `NWDAF/` 與 `PyAnLF/` 已有可用的 lifecycle contract
4. backend 已成為 model usage tracking 與 replacement fallback 的主要 owner
5. Go 端不再把 active model identity 當成主要本地 runtime state
6. `Predict` 已改為以 `subscriptionId` 為 key
7. prediction data path 仍可工作，且未被這個 phase 粗暴重寫
8. replacement failure 的「保留舊模型」語意已在設計與測試上被明確承接
9. Go unit tests、PyAnLF unit/API tests 至少達到本文件定義的最低覆蓋

---

## 13. Risks

本 phase 的主要風險包括：

1. lifecycle 與 prediction 邊界沒切乾淨，導致 scope 膨脹
2. Go 端 state 移除過快，造成既有流程仍隱含依賴但未被發現
3. backend contract 若過度貼近現有 low-level API，只是換名字而未真正改 owner
4. retrain completion 若沒有和 `MTLF` provision 收斂，會留下第二套 lifecycle path
5. fallback 語意若沒有在兩端一致承接，實作結果可能與設計不符
6. 若一開始就過度追求最終版多模型策略，會拖慢本 phase 的落地

對應原則是：

1. 先搬 lifecycle owner，不先搬 prediction runtime
2. 先收斂外部模型材料更新與三個核心操作，再收縮 Go-side shared model state
3. 以可驗證的 fallback semantics 作為 replacement 設計基準

---

## 14. Relation To Later Phases

`Phase 2` 完成後，後續 phase 的預期分工應為：

1. `Phase 3`
   - 將 prediction runtime、historical data shaping、analytics request handling
     逐步搬到 `PyAnLF/`
2. `Phase 4`
   - 將 accuracy bookkeeping、ground truth matching、retrain input logic 搬到
     `PyAnLF/`

也就是說，`Phase 2` 完成後的理想狀態，不是 Go 端已完全退出 `AnLF` 業務邏輯，
而是它已不再主導 model lifecycle policy，為後續 prediction 與 accuracy owner
下沉清出乾淨邊界。

---

## 15. Implementation Record

### 15.1 Completion Summary

`Phase 2` 已在 2026-07-10 完成實作與驗證。

本次完成的核心 ownership 轉移是：

1. `NWDAF/` 不再直接決定 model load、reuse、replacement、unload 的步驟
2. `PyAnLF/` 成為 subscription runtime、shared model usage 與 replacement fallback
   的主要 owner
3. `MTLF` provision callback 與 Daisy retrain completion 已收斂成同一類
   provision-style apply contract
4. prediction call 已改以 `subscriptionId` 為主鍵，Go 端不再需要 backend
   `modelId`
5. static/default model selection 已移到 `PyAnLF/` 本地 config

完成 commits：

1. `NWDAF/` commit `62e2f9f`
   - `refactor(anlf): delegate model lifecycle to backend`
2. `PyAnLF/` commit `7a2ebc4`
   - `feat(runtime): manage subscription model lifecycle`

### 15.2 Final Internal API Shape

Phase 2 最終採用以下 internal HTTP API：

1. `PUT /subscriptions/{subscriptionId}/runtime`
   - 對應 `ApplySubscriptionRuntime`
   - 用於 registration、首次 activation、reuse 與 replacement
2. `DELETE /subscriptions/{subscriptionId}/runtime`
   - 對應 `ReleaseSubscriptionRuntime`
   - backend 解除 subscription usage，最後一個使用者離開時才 unload
3. `POST /subscriptions/{subscriptionId}/predict`
   - `Predict` 名稱保留
   - 主鍵由 `modelId` 改為 `subscriptionId`

optional `GetSubscriptionRuntimeState` 沒有在本 phase 實作，因為目前 Go 端沒有
立即需要主動查詢 backend runtime state 的流程；這不影響 Phase 2 completion。

Apply request 最終包含：

1. `subscription.subscription_id`
2. `subscription.notif_corr_id`
3. `subscription.evt_req`
4. `subscription.event_subscriptions`
5. optional `provision_context.source`
6. optional `provision_context.mtlf_subscription_id`
7. optional `provision_context.notif_subscription_id`
8. optional `provision_context.ml_event_notif`

Apply response 最終支援：

1. `PENDING_PROVISION`
2. `ACTIVATED`
3. `REUSED`
4. `REPLACED`
5. `FAILED_USING_PREVIOUS`
6. `FAILED_NO_PREVIOUS`

response 另外保留 `active_model_reference`。它不是重新把 active model ownership
交還 Go，而是暫時提供尚未搬移的 accuracy workflow 做 model-reference correlation。

### 15.3 NWDAF Implementation

`NWDAF/` 完成以下調整：

1. `internal/anlf/backend.go`
   - `AnlfBackendAPI` 移除 `LoadModel` 與 `UnloadModel`
   - 新增 apply/release oriented methods
   - `Predict` 改用 `subscriptionId`
   - contract 盡量重用 free5GC generated OpenAPI models
2. `internal/anlf/client/backend.go`
   - 改用新的三個 subscription-oriented paths
   - 保留 context、timeout、HTTP client injection 與 error wrapping
3. `internal/anlf/model.go`
   - 移除 `InitializeMlModel` 與 `SwapModel` low-level lifecycle policy
   - 改為建立 subscription snapshot、呼叫 apply/release、處理 backend outcome
   - 新增 retrain completion 到 provision-style apply 的轉譯
4. `internal/anlf/mlmodel_notify.go`
   - callback action 改為攜帶完整 apply request
   - 先用 `notifCorreId` correlation，必要時再以 MTLF subscription ID 回查
   - replacement 前不再先拆除舊 runtime correlation
5. `internal/anlf/analytics.go`
   - prediction 不再讀取 `modelId`
   - 直接以 NWDAF subscription ID 呼叫 backend
6. `internal/context/ml_model.go`
   - 移除 per-subscription 與 shared `ModelId`
   - 移除 Go-side load completion channel 與 wait barrier
   - `SharedModelInfo` 只暫留 accuracy/retrain 所需的 model-reference correlation
7. `internal/mtlf/`
   - 移除 `onModelSwapReady` 與 `onModelSwapped` 兩段 callback
   - Daisy completion 改走單一 `onModelProvisionUpdated`
   - apply failure 會清除 retraining flag，讓舊模型繼續服務並允許後續重試
8. `internal/sbi/processor/`
   - 沒有 external MTLF 時，送出不含模型材料的 registration apply
   - 有 external MTLF 時，保留 pending correlation 並等待 callback
   - delete、update cleanup 與 scheduler completion 會送 release
9. `pkg/factory/config.go` 與 `config/nwdafcfg.yaml`
   - 移除 Go-side `mtlf.staticModelUrl`

目前 free5GC OpenAPI v1.2.3 generated `MlEventNotif` 尚未包含本地新版
TS 29.520 YAML 的 `modelUpdateInd`。本次沒有修改 generated code，而是在 internal
contract 以 `MLEventNotification` 包裝 generated model，補上 retrain update 所需欄位。

### 15.4 PyAnLF Implementation

`PyAnLF/` 完成以下調整：

1. 新增 `src/py_anlf/core/runtime_manager.py`
   - `SubscriptionRuntimeManager` 成為 lifecycle owner
   - per-subscription state 由 `SubscriptionRuntime` 保存
   - per-model shared usage 由 `SharedModelRuntime` 保存
2. activation 與 reuse
   - 首次 model acquisition 成功回傳 `ACTIVATED`
   - 相同 reference 已存在時重用 runtime，回傳 `REUSED`
3. replacement 與 fallback
   - candidate model 成功準備後才切換 active runtime
   - candidate 準備失敗時保留舊 runtime
   - 有舊 runtime 時回傳 `FAILED_USING_PREVIOUS`
   - 無舊 runtime 時回傳 `FAILED_NO_PREVIOUS`
4. release
   - subscription release 只解除該 subscription 的 usage
   - 最後一個使用者離開後才真正 unload model
5. concurrency
   - 同一 subscription 的 apply/release 會序列化
   - 同一 model reference 的 concurrent apply 只會 load 一次
   - reservation 避免 candidate 尚未綁定時被提早 unload
   - 使用固定 striped locks，避免 lock registry 隨 subscription 永久增長
6. `src/py_anlf/models.py`
   - 新增 subscription/provision/apply/release/predict schemas
   - 以 aliases 保留 `mLFileAddr`、`mLModelUrl`、`notifCorreId` 等規格語意
   - 允許保留 backend 尚未消費的額外 spec-aligned fields
7. `src/py_anlf/sbi/routers/analytics.py`
   - 新增 lifecycle 與 subscription-keyed predict routes
   - 以 FastAPI `app.state` 完成 dependency injection
8. `src/py_anlf/sbi/server.py`
   - 移除 class-level singleton manager dependency
   - server 建立並持有 `ModelManager` 與 `SubscriptionRuntimeManager`
9. `src/py_anlf/core/model_manager.py`
   - 支援 HTTP、HTTPS、`file://` 與本地目錄 model references
   - model registry 加鎖
   - tar extraction 使用 Python 3.12 data filter
   - 補齊 HTTP response、temporary file 與失敗 artifact cleanup
10. `config/config.yaml`
   - 新增 backend-local `model.default_model_reference`
11. README、manual client 與 test dependencies
   - README 改以 subscription-oriented API 為主要介面
   - `tests/client_test.py` 改走 apply/predict/release
   - `pyproject.toml` 與 `uv.lock` 加入 pytest/httpx 開發依賴

舊 `/model/load`、`/model/unload`、`/predict` endpoints 仍暫時保留，符合本文件
原先的 contract migration 策略；`NWDAF/` 已完全不再依賴它們。正式移除時機留給
後續 phase 決定。

### 15.5 Verification Record

`NWDAF/` 已執行：

```bash
make test
make build
make lint
```

結果：

1. full Go test suite 通過
2. build 通過
3. `golangci-lint` 回報 `0 issues`

`PyAnLF/` 已執行：

```bash
uv run --group dev pytest -q
```

結果：

1. `10 passed`
2. domain tests 覆蓋 activation、reuse、replacement、fallback、release、default
   policy 與 concurrent sharing
3. API tests 覆蓋 apply/release/predict contract 與 invalid request

另外完成一條 local live API interaction：

1. 啟動目前工作樹的真實 PyAnLF service
2. 以 Go contract 等價 payload 呼叫 registration apply
3. default model bundle 實際載入並回傳 `ACTIVATED`
4. subscription-keyed predict 回傳成功
5. release 回傳 `released`

本次 live interaction 驗證了真實 PyAnLF HTTP API 與模型載入；沒有啟動完整
NWDAF、MTLF、SMF、UPF 環境，因此不宣稱完成完整 5GC end-to-end validation。

### 15.6 Completion Assessment

本 phase 的 completion criteria 已滿足：

1. lifecycle policy 已移出 Go
2. MTLF 與 Daisy 更新已收斂
3. backend contract 已可用並有兩端 tests
4. model usage tracking 與 fallback 已由 PyAnLF 主責
5. Go 不再保存 backend `modelId`
6. prediction 已改用 `subscriptionId`
7. historical data shaping 與 analytics result formatting 未被提前搬移
8. fallback 已有明確 implementation 與 tests

仍保留給後續 phase 的項目：

1. Phase 3：prediction request shaping、historical data handling 與 analytics runtime
2. Phase 4：prediction bookkeeping、ground truth matching、accuracy evaluation 與
   retrain input logic
3. optional runtime query API
4. legacy PyAnLF low-level endpoints 的正式移除
