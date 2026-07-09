# Phase 1 Backend Boundary Alignment

Date: 2026-07-09

Status: Completed

Parent plan:

- `nwdaf-docs/docs/plans/anlf-backend-transition/AnLF Backend Transition Plan.md`

---

## 1. Purpose

這份文件定義 `AnLF Backend Transition` 的 `Phase 1` 細部規劃。

本 phase 的目的不是搬移 `AnLF` 的主要業務邏輯，而是先把 `NWDAF/` 內目前以
`inference engine` 為中心的接縫，完整收斂成 `AnLF backend` 語意。

這樣後續 `Phase 2` 到 `Phase 4` 在搬移 model lifecycle、analytics runtime、
accuracy workflow 時，就不需要一邊搬邏輯、一邊反覆回頭整理命名與 wiring。

---

## 2. Scope

本 phase 處理以下事項：

1. 將 `NWDAF/` 內部 `inference engine` 相關命名重整為 `AnLF backend`
2. 將 config 欄位從 `inferenceEngine` 收斂為 `anlfBackend`
3. 收斂 Go-side interface、client、constructor、injection point 與註解語意
4. 收斂與 `AnLF backend` 相關的 Go unit tests
5. 建立後續 phase 可沿用的 backend contract 邊界

本 phase 不處理以下事項：

1. model load / unload / swap ownership 的實質下沉
2. analytics request shaping 或 prediction runtime 的實質搬移
3. accuracy monitor、ground truth matching、prediction bookkeeping 的 owner 轉移
4. `PyAnLF/` 內部功能擴張或 API 行為改造
5. 跨 repo 的實機互通測試

---

## 3. Planning Basis

本文件基於以下本地來源整理：

1. `AGENTS.md`
2. `nwdaf-docs/docs/development_policy.md`
3. `free5gc-dev-skill/SKILL.md`
4. `NWDAF/` current tree on 2026-07-09
5. `PyAnLF/` current tree on 2026-07-09
6. `AnLF Backend Transition Plan.md`

---

## 4. Agreed Decisions

本 phase 已先確認以下方向：

1. 命名採積極收斂，包含 config、型別、欄位、檔名、測試名稱與註解
2. config 不保留 `inferenceEngine` 的相容期舊欄位
3. Go 端抽象語意採 `AnlfBackend`
4. package 仍可保留 `client` 風格，但型別與 contract 應反映 `AnLF backend`
5. `analytics` 相關 config 暫時仍保留在 `NWDAF/`，不在本 phase 搬移
6. 本 phase 完成標準採完整版本，不接受留下大批舊語意待後續補改
7. 驗證只要求 Go 端 unit tests，不要求跨 repo 實機互通測試

---

## 5. Implementation Progress

截至 2026-07-09，本 phase 已完成實作，對應 `NWDAF/` commit：

- `0db9584` `refactor(anlf): align backend boundary naming`

本次已完成的範圍包括：

1. `inferenceEngine` config key 收斂為 `anlfBackend`
2. `InferenceEngineConfig`、`InferenceEngineAPI` 收斂為
   `AnlfBackendConfig`、`AnlfBackendAPI`
3. `internal/anlf/inference_engine.go`、`internal/anlf/client/inference_engine.go`
   與對應測試檔完成 rename
4. `AnlfService`、notifier、SBI processor、相關測試與樣板設定完成 backend
   語意對齊
5. 既有 `load / unload / predict` 行為維持不變，只調整接縫命名與 wiring

驗證結果：

1. 已執行 `go test ./...`
2. Go test 全數通過

本 phase 明確未處理：

1. model lifecycle ownership 下沉
2. analytics runtime 搬移
3. accuracy workflow owner 轉移
4. `PyAnLF/` API 擴張

---

## 6. Current Code Boundaries

目前 `NWDAF/` 與下游 Python service 的接縫主要集中在以下位置：

1. `NWDAF/internal/anlf/inference_engine.go`
   - 定義 `InferenceEngineAPI` 與對應 request / response 型別
2. `NWDAF/internal/anlf/client/inference_engine.go`
   - 實作本地 HTTP client，對 `/model/load`、`/model/unload`、`/predict`
     發送請求
3. `NWDAF/internal/anlf/anlf.go`
   - `AnlfService` 直接持有 `inferenceEngine InferenceEngineAPI`
4. `NWDAF/pkg/service/init.go`
   - 建立 client，並注入 `AnlfService`
5. `NWDAF/pkg/factory/config.go`
   - 定義 `InferenceEngineConfig`
6. `NWDAF/config/nwdafcfg.yaml`
   - 暴露 `inferenceEngine` 設定欄位

此外，以下模組會吃到這條接縫，但本 phase 原則上只做命名與 wiring 對齊：

1. `NWDAF/internal/anlf/model.go`
2. `NWDAF/internal/anlf/analytics.go`
3. `NWDAF/internal/notifier/`
4. `NWDAF/internal/mtlf/`

---

## 7. Target Shape

### 7.1 Goal

本 phase 完成後，`NWDAF/` 應能用較中性的 `AnLF backend` 語意看待下游服務，
而不是把它當成僅承接推論的 `inference engine`。

### 7.2 Naming Direction

預期命名方向如下：

1. `InferenceEngineAPI` → `AnlfBackendAPI`
2. `InferenceEngineConfig` → `AnlfBackendConfig`
3. config key `inferenceEngine` → `anlfBackend`
4. `AnlfService` 內部欄位 `inferenceEngine` → `anlfBackend`
5. 建構與 getter 相關參數名稱同步改為 backend 語意

### 7.3 File-Level Direction

預期檔名方向如下：

1. `internal/anlf/inference_engine.go` → `internal/anlf/backend.go`
2. `internal/anlf/client/inference_engine.go` → `internal/anlf/client/backend.go`
3. 對應測試檔名同步跟進

若實作時發現需要更明確檔名，例如 `anlf_backend.go`，可在不改變本 phase
語意前提下調整，但不應保留 `inference_engine` 舊檔名。

### 7.4 Contract Principle

本 phase 不要求把所有 request / response 型別設計成最終版，但至少要達成：

1. Go 端 contract 命名不再以 inference-only 語意描述下游
2. 現有 `load / unload / predict` contract 在 backend 語意下仍能成立
3. 後續 phase 能在同一條 contract seam 上擴充能力，而不是另開平行接法

---

## 8. Planned Implementation Areas

### 8.1 Config And Factory

預期修改：

1. `pkg/factory/config.go`
   - 新增或改名為 `AnlfBackendConfig`
   - 驗證規則改為 `anlfBackend.endpoint`
2. `config/nwdafcfg.yaml`
   - 樣板與註解改為 `anlfBackend`
3. `pkg/factory/config_test.go`
   - 測試案例名稱與預期錯誤字串同步改名

### 8.2 AnLF Contract And Service Wiring

預期修改：

1. `internal/anlf/backend.go`
   - 定義 `AnlfBackendAPI`
   - 收斂 shared request / response 型別與註解
2. `internal/anlf/anlf.go`
   - `AnlfService` 改持有 `anlfBackend AnlfBackendAPI`
   - constructor 與 accessor 語意同步
3. `pkg/service/init.go`
   - 以 `anlfBackend` config 建立 client
   - 注入 `AnlfService`

### 8.3 Client Implementation

預期修改：

1. `internal/anlf/client/backend.go`
   - client 註解與錯誤訊息改為 backend 語意
   - `NewClient`、method receiver、測試名稱同步收斂
2. `internal/anlf/client/backend_test.go`
   - 保留既有 HTTP 行為覆蓋
   - 更新型別名稱、測試描述與期望字串

### 8.4 Call Site Alignment

預期修改：

1. `internal/anlf/model.go`
2. `internal/anlf/analytics.go`
3. `internal/notifier/`
4. `internal/mtlf/`
5. 其他直接依賴 `InferenceEngineAPI` 的 Go 檔案

這些位置本 phase 的目標是：

1. 改成依賴 `AnlfBackendAPI`
2. 將局部註解或 log wording 對齊 backend 語意
3. 不改變既有業務責任與控制流程

---

## 9. Explicit Non-goals

以下事項即使在實作過程中很接近，也不應順手納入本 phase：

1. 將 `analytics` config 下沉到 `PyAnLF/`
2. 將 model activation state machine 改成 backend-owned
3. 調整 `MlModelInfo`、`SharedModelInfo`、`ModelAccuracyStore` ownership
4. 重新設計 `PyAnLF` API 路徑或擴充新 endpoint
5. 為了命名收斂而重寫 prediction、monitor、retrain 行為

若實作中發現某個 rename 會直接逼出 ownership 變更，應先停下來重新評估，
而不是讓 `Phase 1` 滑進 `Phase 2` 或 `Phase 3`。

---

## 10. Verification

本 phase 的最低驗證要求限定為 Go 端 unit-level 驗證。

至少應覆蓋：

1. `pkg/factory/config_test.go`
   - 驗證 `anlfBackend` config 的必要欄位與驗證錯誤字串
2. `pkg/service/init_test.go`
   - 驗證 app initialization 與 backend client wiring
3. `internal/anlf/client/backend_test.go`
   - 驗證 load / unload / predict 的 client request behavior
4. 所有因型別名變更而需要同步調整的 `internal/anlf` 與呼叫端 unit tests

本 phase 不要求：

1. `NWDAF/` 與 `PyAnLF/` 的實機互通測試
2. `PyAnLF/` 端新增 unit tests
3. integration-level API smoke test

---

## 11. Completion Criteria

本 phase 完成時，應同時滿足以下條件：

1. `NWDAF/` 內主要下游接縫已不再使用 `inference engine` 作為主要抽象語意
2. config、型別、欄位、檔名、測試與樣板設定已完成 `AnLF backend` 收斂
3. `anlfBackend` 成為唯一有效的 config key
4. 與 backend 接縫相關的主要呼叫點已完成 rename，不留下大批舊名待後續收尾
5. 現有 `load / unload / predict` 行為仍能在新語意下工作
6. Go 端相關 unit tests 已更新並通過
7. 未實質改動 model lifecycle、analytics runtime、accuracy workflow 的 owner

---

## 12. Risks

本 phase 的主要風險包括：

1. rename 範圍擴散，導致 diff 過大且混入業務邏輯調整
2. config rename 造成既有測試、樣板或手動執行方式失效
3. 某些註解與 log wording 若沒有同步更新，會留下語意不一致
4. 若 call site 對齊不完整，後續 phase 仍可能需要回頭修接縫

對應原則是：

1. 只做接縫相關 code 變更
2. 允許完整 rename，但不允許順勢改 owner
3. 以單元測試確認 rename 後的 wiring 與 client 行為仍成立

---

## 13. Relation To Later Phases

`Phase 1` 完成後，後續 phase 的預期分工應為：

1. `Phase 2`
   - 在既有 `AnlfBackend` seam 上搬移 model lifecycle runtime ownership
2. `Phase 3`
   - 在既有 `AnlfBackend` seam 上搬移 analytics runtime 與 request shaping
3. `Phase 4`
   - 在既有 `AnlfBackend` seam 上搬移 accuracy workflow 與 retrain input logic

也就是說，後續 phase 應把這次建立的 backend seam 視為既有基礎，而不是重新命名
或重開另一條呼叫路徑。
