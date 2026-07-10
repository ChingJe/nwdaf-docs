# Phase 3.5 Go Package Boundary Consolidation

Date: 2026-07-11

Status: Planned

Parent plan:

- `nwdaf-docs/docs/plans/anlf-backend-transition/AnLF Backend Transition Plan.md`

Implementation baseline:

1. `NWDAF/`: `6b3442f` (`refactor(anlf): delegate analytics runtime to backend`)
2. `PyAnLF/`: `6a0c1f4` (`feat(runtime): own analytics reporting workflow`)
3. `nwdaf-docs/`: `5c6ba9a` (`docs: record analytics runtime migration completion`)

---

## 1. Purpose

Phase 3 已把 analytics generation、observation buffering、model execution 與 reporting
lifecycle 搬到 `PyAnLF`，但 Go 端 `internal/anlf` 的 package shape 仍保留遷移前的
多重責任。

目前 `internal/anlf` 根 package 同時包含：

1. auxiliary HTTP server 與 API adapter
2. NWDAF-to-AnLF Backend JSON contract
3. subscription runtime 與 model provision coordination
4. observation delivery queue、retry 與 worker lifecycle
5. Phase 4 尚未遷移的 accuracy workflow
6. callback processor 所需的共用型別與 error semantics

這使根 package 不像已對齊的 `internal/sbi` HTTP edge，也讓 `sbi/processor`、
`anlf/processor`、backend client、notifier 與 accuracy workflow 都依賴過度寬廣的
`anlf.AnlfService` 或根 package 型別。

本 phase 的目的，是在不改變 Phase 3 行為的前提下，重新建立可說明且無循環的
package boundary。

---

## 2. Classification

這是一個純 Go-side、behavior-preserving 的 package refactor。

允許改動：

1. package path、檔案位置與 import
2. internal type 所在 package
3. narrow interface 的定義位置
4. constructor 與 dependency injection wiring
5. app-owned component 的直接 lifecycle wiring
6. 測試檔案位置、fixtures 與 mocks

不得改動：

1. AnLF server HTTP paths、methods 與 status mapping
2. NWDAF-to-AnLF Backend JSON field、enum 與 error contract
3. subscription create、update、delete、rollback 與 runtime revision semantics
4. observation queue capacity、timeout、retry、stable batch ID 與 drop semantics
5. analytics report validation、dedup、in-flight 與 stale-report semantics
6. external Nnwdaf notification payload 與 delivery behavior
7. model provision callback 與 retrain completion flow
8. config field、default value與 validation behavior
9. Phase 4 transitional accuracy behavior
10. `PyAnLF/` implementation

若整理過程發現必須改變上述 observable behavior 才能完成 package 拆分，應停止實作，
先回到 plan 討論，不得把行為修正混入本 phase。

---

## 3. Reference Basis

本 phase 以以下本地來源為依據：

1. `NWDAF/internal/anlf/` Phase 3 baseline
2. `NWDAF/internal/sbi/` 現有 server、API、processor 與 consumer shape
3. free5GC PCF `internal/sbi` 的 server/API/processor 分層
4. free5GC NEF `internal/sbi/notifier` 的獨立 notifier package
5. free5GC AMF、PCF、SMF 與 UDM 的 notification/callback placement
6. `nwdaf-docs/docs/development_policy.md`
7. `free5gc-dev-skill/SKILL.md` 與其 NF architecture、SBI、testing、lifecycle guidance

free5GC 並沒有唯一的 notifier placement。PCF 多把 notification procedure 放在
processor 或 consumer；NEF 則明確保留 `internal/sbi/notifier`。NWDAF analytics
notification 同時具有 subscription state、delivery policy、dedup 與 outbound callback
責任，因此本 phase 採用 NEF-style 的獨立 SBI notifier，而不是消除 notifier abstraction。

---

## 4. Current Responsibility Map

Phase 3 baseline 的主要責任分布如下：

| Current location | Current responsibility | Problem |
| --- | --- | --- |
| `internal/anlf/server.go` | AnLF HTTP server、routes、lifecycle | 合理的 HTTP edge 責任 |
| `internal/anlf/api_*.go` | request parsing、response/status mapping | 合理的 HTTP edge 責任 |
| `internal/anlf/backend.go` | backend interface、wire models、report errors | transport、contract 與 outcome 混在根 package |
| `internal/anlf/anlf.go` | broad service facade、worker lifecycle、accuracy callbacks | 同時成為多個 domain 的 service locator |
| `internal/anlf/model.go` | runtime apply/release/binding、model correlation | coordinator 與 transitional accuracy state 混合 |
| `internal/anlf/mlmodel_notify.go` | provision action planning/execution | application workflow 留在 HTTP 根 package |
| `internal/anlf/observation_delivery.go` | queue、retry、worker lifecycle | outbound workflow 留在 HTTP 根 package |
| `internal/anlf/monitor.go`、`scope.go` | Phase 4 transitional accuracy | 暫時性業務邏輯污染永久 transport boundary |
| `internal/anlf/client/` | backend HTTP client | 邊界合理，但依賴根 package contract |
| `internal/anlf/processor/` | callback use case entry | 邊界合理，但依賴 broad root package types/service |
| `internal/notifier/` | report mapping、dedup、external delivery | 語意屬 SBI notifier，但位置在頂層 |

---

## 5. Target Package Layout

目標 layout：

```text
internal/anlf/
├── server.go
├── api_analytics_report.go
├── api_mlmodel_notify.go
│
├── contract/
│   ├── runtime.go
│   ├── observation.go
│   ├── analytics_report.go
│   └── model_provision.go
│
├── client/
│   ├── client.go
│   ├── runtime.go
│   └── observation.go
│
├── processor/
│   ├── processor.go
│   ├── analytics_report.go
│   ├── mlmodel_notify.go
│   └── errors.go
│
├── coordinator/
│   ├── coordinator.go
│   ├── subscription_runtime.go
│   ├── model_provision.go
│   └── observation_delivery.go
│
└── accuracy/
    ├── monitor.go
    ├── scope.go
    └── report.go

internal/sbi/
└── notifier/
    ├── notifier.go
    ├── analytics_report.go
    ├── models_output.go
    └── analytics_report_test.go
```

檔名可在實作時依實際 method grouping 做小幅調整，但 package ownership 與 dependency
direction 不得改變。

---

## 6. Package Responsibilities

### 6.1 `internal/anlf`

只保留 auxiliary HTTP edge：

1. Gin router 與 route registration
2. request path/body parsing
3. 呼叫 `anlf/processor`
4. 將 processor outcome 映射成 HTTP status/body
5. listener startup、shutdown 與 server lifecycle

沿用目前 NWDAF SBI 與 free5GC PCF 的寫法，不新增獨立 `routes.go`：

1. `server.go` 保存共用 `Route`、`applyRoutes`、server constructor 與 lifecycle
2. `api_analytics_report.go` 保存 analytics report route declaration 與 handler
3. `api_mlmodel_notify.go` 保存 ML model provision notify route declaration 與 handler
4. `NewServer` 組合各 API route group

根 package 不再保存 backend client、observation worker、accuracy monitor 或 runtime
coordination state。

### 6.2 `internal/anlf/contract`

保存 NWDAF 與 AnLF Backend 間穩定的 wire contract：

1. subscription runtime apply request/response
2. provision context
3. observation bindings 與 observation batches
4. analytics report callback payload
5. wire-level enums

這個 package 只保存 protocol data，不持有 HTTP client、context state、goroutine 或
workflow。Outcome errors 不應因方便而全部放入 contract；它們應由實際判斷該 outcome
的 processor/coordinator 擁有。

### 6.3 `internal/anlf/client`

只負責 outbound backend HTTP transport：

1. URL construction
2. JSON serialization/deserialization
3. request timeout context propagation
4. response status/error decoding
5. concrete `http.Client` ownership

client 不管理 subscription state、retry queue 或 model correlation。

### 6.4 `internal/anlf/processor`

作為 AnLF callback API 的 use-case entry：

1. ML model provision notification workflow entry
2. analytics report acceptance workflow entry
3. 呼叫 narrow coordinator/notifier interfaces
4. 提供 HTTP edge 所需的 typed outcome errors

processor 不直接擁有 listener、backend HTTP implementation 或 long-running worker。

### 6.5 `internal/anlf/coordinator`

保存 Go 端仍必要的 cross-domain coordination：

1. 建立 subscription runtime request
2. apply、release 與 observation binding synchronization
3. model provision action planning/execution
4. model reference correlation 的 Phase 4 過渡整合
5. observation delivery worker ownership
6. 對 SBI processor 提供 narrow AnLF coordination interface

使用 `coordinator` 而不是 `runtime`，是為了避免暗示 Go 端仍擁有 analytics runtime；
使用 `coordinator` 而不是泛稱 `service`，則可避免和 `pkg/service` bootstrap package
混淆。

### 6.6 `internal/anlf/accuracy`

集中 Phase 4 尚未遷移的 AnLF-side accuracy measurement 與 reporting responsibility：

1. prediction bookkeeping
2. ground-truth lookup/alignment
3. accuracy metric calculation
4. accuracy monitor lifecycle
5. per-scope accuracy information 與 inference count 產生
6. reporting period/threshold 到達時輸出供 MTLF 使用的 report

這個 package 不負責判定 ML Model 是否 degraded，也不負責 baseline、decision window、
retrain/reprovision policy 或 training dispatch。這些屬於 `internal/mtlf`。

本 phase 只搬移程式碼與依賴，不重寫 accuracy algorithm。Phase 4 可基於這個清楚邊界
把 AnLF-side accuracy information generation 搬到 PyAnLF，同時保留 MTLF 的 model
degradation 與 retraining decision ownership。

### 6.7 AnLF And MTLF Accuracy Boundary

依 TS 23.288 clauses 5C.1、6.2D 與 6.2E.3，本計畫採用 AnLF-assisted MTLF ML Model
Accuracy Monitoring 的責任線：

| Responsibility | Owner |
| --- | --- |
| 保存 prediction 並取得對應 ground truth | AnLF / future PyAnLF |
| 比較 prediction 與 ground truth | AnLF / future PyAnLF |
| 計算 accuracy/deviation metrics、sample count 與 inference count | AnLF / future PyAnLF |
| 依 reporting period 或 reporting threshold 產生通知 | AnLF / future PyAnLF |
| 接收一個或多個 AnLF 的 accuracy information | MTLF |
| 維護 baseline、history、decision window 與 multi-report state | MTLF |
| 判斷 ML Model degradation | MTLF |
| 決定 retrain、reprovision 或 model reselection | MTLF |
| 執行 training workflow 並提供更新模型 | MTLF |

AnLF 在規格上可判斷 analytics accuracy 是否低於 reporting threshold，並因此發送
`Nnwdaf_MLModelMonitor_Notify`；這是 report trigger，不等同於 MTLF 對 model degradation
及 retraining necessity 的最終判斷。

目前程式碼大致已遵守這條責任線：

1. `internal/anlf/monitor.go` 負責 prediction/ground-truth matching、metric calculation
   與 report generation
2. `internal/mtlf/trigger.go` 負責 baseline、z-score、chronic/low-traffic policy、
   decision window 與 retrain trigger
3. `internal/mtlf/training.go` 負責 retraining workflow

Phase 3.5 不得把 `mtlf/trigger.go`、`mtlf/state_store.go` 或 training decision 搬進
`anlf/accuracy`。

### 6.8 `internal/sbi/notifier`

保留 notifier abstraction，負責標準 SBI outbound notification：

1. 將已接受的 backend analytics report 映射為 Nnwdaf notification
2. consumer notification URI delivery
3. delivered/in-flight dedup coordination
4. report sequence completion state
5. 成功 delivery 後的 transitional accuracy recording hook

它不應搬進 `sbi/consumer`，也不應被刪除。AnLF processor 透過 narrow notifier
interface 使用它，不直接依賴 concrete HTTP internals。

---

## 7. Dependency Direction

允許的主要依賴方向：

```text
pkg/service
    -> anlf server
    -> anlf processor
    -> anlf coordinator
    -> anlf client
    -> sbi notifier

anlf server
    -> anlf processor
    -> anlf contract

anlf processor
    -> anlf contract
    -> coordinator narrow interfaces
    -> notifier narrow interfaces

sbi processor
    -> coordinator narrow interfaces
    -> anlf contract

anlf coordinator
    -> anlf contract
    -> backend-client narrow interface
    -> anlf accuracy

anlf client
    -> anlf contract

sbi notifier
    -> anlf contract
    -> NWDAF context
    -> standard OpenAPI models
```

禁止的方向：

1. `anlf/contract` import client、processor、coordinator、context 或 factory
2. `anlf/client` import root `anlf` server package
3. `anlf/coordinator` import `sbi/processor`
4. `sbi/notifier` import concrete `anlf/processor`
5. root `anlf` 被當成共用 domain model package
6. 為了消除 import cycle 而重新引入 broad app/service locator interface

---

## 8. Interface And Wiring Rules

### 8.1 Interfaces Live With Consumers

依 Go 慣例，narrow interface 應定義在使用它的 package，而不是集中建立一個新的
global interfaces package。

至少應拆出：

1. coordinator 使用的 backend runtime client interface
2. observation delivery 使用的 observation sender interface
3. AnLF processor 使用的 model provision workflow interface
4. AnLF processor 使用的 analytics notifier interface
5. SBI processor 使用的 AnLF coordination interface

### 8.2 App Wiring

`pkg/service.NewApp` 保持 composition root，負責建立：

1. concrete AnLF backend HTTP client
2. AnLF accuracy transitional component
3. AnLF coordinator
4. SBI notifier
5. AnLF processor
6. AnLF server
7. SBI processor 與 SBI server

不得讓 server constructor 自行建立 backend client、notifier 或 coordinator。

### 8.3 Lifecycle Ownership

目前 observation delivery 的 Start/Stop 由 `pkg/service` 經過 `sbi/processor` 再轉呼叫
`AnlfService`。這個 passthrough 不屬於 SBI request processing。

整理後應由 `pkg/service` 直接持有並控制 coordinator/observation delivery lifecycle，
但啟停時機保持不變：

1. owned listeners 成功啟動後才開始 observation delivery
2. shutdown 時停止接收新工作並等待 worker 結束
3. 不新增 background goroutine
4. 不改變 queue drain/cancel semantics
5. 不改變 WaitGroup ownership 保證

---

## 9. File Migration Map

建議搬移順序與目標：

| Current file | Target |
| --- | --- |
| `internal/anlf/backend.go` | 按 wire domain 拆到 `internal/anlf/contract/`；consumer-owned interfaces 移到實際使用 package |
| `internal/anlf/anlf.go` | lifecycle/coordinator 部分移到 `anlf/coordinator`；accuracy callback/report 移到 `anlf/accuracy` |
| `internal/anlf/model.go` | runtime/binding 移到 `anlf/coordinator/subscription_runtime.go`；accuracy correlation 移到 `anlf/accuracy` 或 transitional coordinator adapter |
| `internal/anlf/mlmodel_notify.go` | `anlf/coordinator/model_provision.go` |
| `internal/anlf/observation_delivery.go` | `anlf/coordinator/observation_delivery.go` |
| `internal/anlf/monitor.go` | `anlf/accuracy/monitor.go` |
| `internal/anlf/scope.go` | `anlf/accuracy/scope.go` |
| `internal/anlf/client/backend.go` | 依 runtime/observation API 拆分，但保持 `client` package |
| `internal/notifier/*` | `internal/sbi/notifier/*` |
| root `internal/anlf/server.go`、`api_*.go` | 保留在根 package；route declaration 移到對應 `api_*.go`，不建立 `routes.go` |

測試應和對應 production responsibility 一起搬移，不要先搬完 production code 再集中修測試。

---

## 10. Implementation Sequence

### Step 1: Extract Wire Contract

1. 建立 `internal/anlf/contract`
2. 搬移 runtime、observation、report 與 provision DTO
3. 更新 client、processor、notifier 與 tests imports
4. 確認 JSON serialization golden behavior 不變

### Step 2: Extract Coordinator

1. 建立 coordinator facade 與 narrow backend dependency
2. 搬移 runtime apply/release/binding workflow
3. 搬移 model provision action workflow
4. 搬移 observation delivery worker
5. 讓 SBI processor 依賴 narrow coordination interface

### Step 3: Isolate Transitional Accuracy

1. 搬移 monitor、scope、report models 與 tests
2. 將 prediction recording hook 收斂成 narrow accuracy interface
3. 保持 MTLF report input 與 metric behavior不變
4. 不提前執行 Phase 4 ownership migration

### Step 4: Relocate Notifier

1. 建立 `internal/sbi/notifier`
2. 搬移 dispatcher、output models 與 tests
3. 讓 AnLF processor 透過 narrow notifier interface 呼叫
4. 保持 status/error mapping、dedup 與 delivery completion semantics

### Step 5: Clean HTTP Root And App Wiring

1. 根 `internal/anlf` 只保留 server/API files，route declaration 由對應 `api_*.go` 管理
2. `pkg/service` 直接 wiring coordinator lifecycle
3. 移除 SBI processor 的 worker Start/Stop passthrough
4. 清除 broad `AnlfService` 與不再需要的 compatibility methods
5. 檢查沒有 package cycle 或反向 dependency

### Step 6: Verification And Review

1. 執行完整 formatting、lint、unit、build 與 targeted race tests
2. 執行 Phase 3 cross-repo live integration test
3. 比較 refactor 前後 API/status/config behavior
4. 進行一次以 regression、lifecycle 與 dependency direction 為主的 code review

---

## 11. Verification Plan

### 11.1 Static And Build Verification

在 `NWDAF/` 執行：

```bash
make lint
make build
go test ./...
```

額外檢查：

1. `git diff --check`
2. `go list`/build 不存在 import cycle
3. root `internal/anlf` 不再包含 coordinator、client、worker 或 accuracy implementation
4. `PyAnLF/` 沒有非預期修改

### 11.2 Race Verification

至少執行：

```bash
go test -race ./internal/anlf/... ./internal/sbi/... ./internal/context ./internal/mtlf/...
```

若完整 `go test -race ./...` 仍受到既有 `pkg/service` global context/Gin test isolation
問題影響，必須保留實際輸出與 targeted packages 結果，不得宣稱完整 race suite 通過。

### 11.3 Behavior Regression Tests

至少保留並通過：

1. subscription create/update/delete 與 rollback tests
2. runtime apply/release/binding tests
3. observation queue full、retry、stable batch ID 與 shutdown tests
4. analytics report invalid/stale/duplicate/in-flight/status mapping tests
5. external notification mapping與 delivery tests
6. model provision callback workflow tests
7. accuracy monitor 與 MTLF retrain policy tests
8. config validation/default tests

### 11.4 Cross-repository Integration

重跑既有 live contract test：

1. 真實 Go backend client 呼叫真實 PyAnLF
2. apply runtime、sync bindings 與 send observations
3. PyAnLF callback Go AnLF server
4. Go AnLF processor 經 SBI notifier 送出 external notification
5. release runtime

這個測試的 endpoint、payload 與 observable sequence 必須和 Phase 3 baseline 相同。

---

## 12. Acceptance Criteria

Phase 3.5 完成時必須同時滿足：

1. `internal/anlf` 根 package 只保留 HTTP server、API adapter 與必要 package doc，且沒有
   獨立 `routes.go`
2. backend wire models 位於 `internal/anlf/contract`
3. Go-side cross-domain orchestration 位於 `internal/anlf/coordinator`
4. backend HTTP transport 仍位於 `internal/anlf/client`
5. callback use cases 位於 `internal/anlf/processor`
6. Phase 4 transitional accuracy 位於 `internal/anlf/accuracy`
7. notifier abstraction 保留並位於 `internal/sbi/notifier`
8. `pkg/service` 直接控制 AnLF-owned worker lifecycle
9. 不存在新的 broad service locator 或 import cycle
10. Phase 3 HTTP、contract、lifecycle、retry、dedup、notification 與 config 行為不變
11. 標準 tests、lint、build、targeted race 與 live integration verification 通過
12. `PyAnLF/` 不需要為 package 整理修改 production code

---

## 13. Risks And Controls

### 13.1 Import Cycle

風險：root AnLF server、processor、coordinator、SBI processor 與 notifier 彼此引用。

控制：先抽取無 behavior 的 contract package；interface 定義在 consumer package；禁止
coordinator 反向 import SBI processor。

### 13.2 Hidden Behavior Change

風險：搬移 lifecycle 或拆 interface 時，改變 worker 啟停、error mapping、rollback 或
dedup 時序。

控制：每一步只做一種責任搬移；保留既有 tests；完成後重跑 live contract test。

### 13.3 Transitional Accuracy Coupling

風險：accuracy 仍依賴 NWDAF context、model correlation 與 MTLF callback，直接搬 package
可能暴露目前的隱性耦合。

控制：Phase 3.5 只建立 adapter/narrow interface，不改 algorithm 或 ownership；若必須重新
設計 accuracy state，停止並留給 Phase 4。

目前 `factory.MtlfConfig.AccuracyMonitor` 同時提供部分 AnLF metric/reporting 參數與 MTLF
decision policy 參數，語意上仍有 ownership 混合。Phase 3.5 為保持 config schema 與行為
不變，不拆分這個設定；Phase 4 設計時必須依 AnLF measurement/reporting 與 MTLF
degradation/retraining decision 重新劃分 config ownership。

### 13.4 Notifier Over-simplification

風險：誤把 notifier 當成普通 HTTP client，導致 delivery state、dedup 或 subscription
notification semantics 散落到 processor。

控制：保留獨立 `internal/sbi/notifier`，以 NEF notifier package 作為結構參考，不強迫
套用 PCF consumer-only shape。

### 13.5 Review Noise

風險：大量 rename 讓真正的 dependency 或 lifecycle 改動難以檢查。

控制：Phase 3 baseline 已獨立 commit；實作時按 contract、coordinator、accuracy、notifier、
wiring 分段，必要時拆成數個純 refactor commits。

---

## 14. Relation To Phase 4

Phase 3.5 不代表 accuracy workflow 已完成遷移。

它只把目前混在 AnLF HTTP 根 package 的 transitional AnLF-side accuracy code 收斂到
明確位置，讓 Phase 4 能直接回答：

1. 哪些 prediction/ground-truth state 應移到 PyAnLF
2. PyAnLF 應如何產生並提供 AnLF-side accuracy information
3. 哪些 accuracy output 應交給 MTLF 的 degradation/retraining decision
4. 哪些 model/subscription correlation 必須留在 Go
5. `internal/anlf/accuracy` 最終應刪除、縮小或轉成 adapter

Phase 4 不應把 `internal/mtlf` 的 baseline、multi-report aggregation、model degradation
判斷或 retraining policy 搬到 PyAnLF；若未來要改變這條規格角色邊界，必須另行設計，
不能視為 AnLF Backend migration 的自然延伸。

因此 Phase 3.5 的完成條件是 package responsibility 清楚且行為不變，不是提前完成
Phase 4 業務邏輯遷移。
