# NWDAF 跨節點 ML Model Route 併發修正計畫

日期：2026-08-13

狀態：完成；程式實作、repository 驗證與 full-core E2E 均已通過

上層文件：

- [Distributed NWDAF Federated Learning Implementation Plan](../Distributed%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)
- [Phase 2 Cross-NWDAF Model Provision And Monitoring Detailed Plan](../Phase%202%20Cross-NWDAF%20Model%20Provision%20And%20Monitoring%20Detailed%20Plan.md)
- [Phase 7 Full-Core Federated Learning Business E2E Detailed Plan](../Phase%207%20Full-Core%20Federated%20Learning%20Business%20E2E%20Detailed%20Plan.md)
- [Phase 8 Backend Process Failure And Stateless Reconnection](../../mtlf-backend-transition/Phase%208%20Backend%20Process%20Failure%20And%20Stateless%20Reconnection.md)

涉及 repositories：

- `NWDAF/`
- `nwdaf-docs/`

實際程式修改範圍只包含 `NWDAF/`。PyAnLF 與 PyMTLF 只作為問題證據來源與
regression 驗證對象，不是本次修正對象。

## 1. 目的

本計畫處理 Phase 7 full-core E2E teardown 刪除跨 NWDAF Model Provision 與
Model Monitor 資源時發現的 distributed lock inversion。

修正後必須達成：

- 不同 NWDAF 上互不相干的 ML model 資源可以同時刪除；
- NWDAF 等待 peer NWDAF、PyAnLF、PyMTLF 或 callback HTTP I/O 時，不持有
  ML model route mutex；
- 舊 HTTP response 不得提交到已被 replace、reset 或更新的 route；
- duplicate operation 不得產生重複 peer 或 backend side effect；
- backend process generation fence 必須繼續有效；
- Release 18 request、response、`Location`、callback 與 `ProblemDetails`
  contract 不變；
- 不重新加入 external full-state sync 或 process recovery 功能。

本修正不是新的 feature phase，而是補齊 Phase 2 peer-route 設計和目前 NWDAF
實作間的 concurrency 缺口。

## 2. 證據與基線

full-core 問題報告使用以下 clean revisions：

| Repository | Revision |
| --- | --- |
| `NWDAF/` | `318ac19d8b027373f4468660394da1ec3338268e` |
| `PyAnLF/` | `08798f15c3693027e00bc60dd53f74ebaa26c3a1` |
| `PyMTLF/` | `7e8ab7f23bf5d6398eb1cd5f053dd8bda9439a87` |

問題拓撲為：

```text
NWDAF-A / PyAnLF-A
  outbound Model Provision route -> NWDAF-C / PyMTLF-C

NWDAF-C / PyMTLF-C
  outbound Model Monitor route   -> NWDAF-A / PyAnLF-A
```

這兩筆 route 是方向相反但彼此獨立的資源。它們沒有共用 subscription ID，
也不是雙方同時刪除同一筆資源。

### 2.1 已確認的等待關係

目前 `Processor` 使用一把 process-local `mlModelMu` 保護所有 ML model
resource kind。一般 DELETE handler 持有這把鎖並呼叫 peer 或 backend HTTP：

```text
NWDAF-A goroutine
  lock A.mlModelMu
  DELETE NWDAF-C 上的 provision resource
  等待 NWDAF-C 回應

NWDAF-C goroutine
  lock C.mlModelMu
  DELETE NWDAF-A 上的 monitor resource
  等待 NWDAF-A 回應

NWDAF-C 收到的反向 request
  等待 C.mlModelMu

NWDAF-A 收到的反向 request
  等待 A.mlModelMu
```

兩端 mutex 並非跨 process 共用；是同步 HTTP call 把兩個本地 lock wait 串成
distributed cycle。

### 2.2 為何結果是 503，而不是永久卡死

peer consumer 使用固定 30 秒 request timeout。timeout 會成為 transport error，
processor 再將 peer transport error 映射成：

```text
503 Service Unavailable
requested ML model service capability is temporarily unavailable
```

timeout 發生後，本地 mutex 會被釋放。PyAnLF 與 PyMTLF 接著使用相同的
1、2、4、8、16 秒 exponential backoff，再次以接近相同的時間重試，因此會
重建同一個 cycle。這是被 timeout 打斷的 distributed deadlock，加上後續 retry
livelock，不是永遠不會解開的 Go runtime deadlock。

### 2.3 PyMTLF 的次要 head-of-line blocking

PyMTLF Model Monitor reconciler 使用單一序列 worker。第一筆 delete 持續失敗時，
它仍會被選為下一個 action，第二筆 retired monitor resource 因此沒有機會執行。

這是實際存在的次要影響，但反覆失敗的根因仍是 NWDAF lock cycle。本次不能藉由
調整 PyMTLF retry、ordering 或 shutdown 行為來掩蓋 NWDAF 缺陷。

## 3. 既有設計與規格邊界

### 3.1 Phase 2 仍具權威性的既有決策

Phase 2 已經要求：

- outbound create 前先建立 `CREATING` reservation；
- route 具有 `ACTIVE`、`REPLACING`、`DELETING` 與
  `PENDING_CLEANUP` 狀態；
- network call 不得長時間持有 store-wide route mutex；
- response commit 前必須驗證 generation 或 revision；
- stale create response 必須執行 compensation；
- peer DELETE timeout 要保留可重試的 route；
- peer `404` 代表 cleanup 已完成；
- duplicate delete 必須保持 side-effect safe。

目前 context 雖然已定義上述 lifecycle state，但一般 DELETE 沒有真正進入
`DELETING`，`DELETING` 與 `REPLACING` 也尚未成為預期中的 operation fence。
因此本修正是實作已記錄的設計，而不是用另一套架構取代它。

Phase 2 failure matrix 所寫的「DELETE timeout 保留 deleting route」在這裡進一步
釐清為：刪除意圖仍由 backend reconciler 保留，但 Go route 的 `DELETING` 只代表
一次正在執行的 HTTP operation。若該次 operation 暫時性失敗，Go route 必須回到
`ACTIVE`，讓 backend 下一次 retry 能重新取得 operation ownership；否則 route 會
永遠卡在 `DELETING` 並持續回 503。

### 3.2 Release 18 HTTP surface

TS 29.520 Release 18 OpenAPI 為以下資源定義 DELETE：

- `Nnwdaf_MLModelProvision` subscription；
- `Nnwdaf_MLModelMonitor` registration；
- `Nnwdaf_MLModelMonitor` subscription；
- `Nnwdaf_MLModelTraining` subscription。

各 operation 宣告的 response surface 都包含 `204`、`404` 與 `503`，以及該
operation 引用的其他 common `ProblemDetails` response。規格不定義 internal
mutex、operation token 或 route store；這些屬於實作內部的 concurrency control，
不得出現在 wire schema。

因此本計畫不新增 public/private HTTP field、header、method 或 endpoint。暫時無法
取得 route mutation ownership 時使用既有 `503 ProblemDetails`；資源已不存在時
沿用既有 `204`／`404` 語意。

## 4. 修正範圍

### 4.1 必須修正的範圍

直接重現問題的路徑為：

1. Model Provision subscription DELETE；
2. Model Monitor subscription DELETE。

同一個 correctness rule 必須套用到四種 route family：

1. Model Provision subscription；
2. Model Monitor registration；
3. Model Monitor subscription；
4. Model Training subscription。

實作時也必須掃描並移除以下路徑中的 lock-held external I/O：

- create 與 failed-create compensation；
- replace 與 Training PATCH；
- notification 與 callback forwarding；
- pending peer-cleanup reconciliation；
- local backend CRUD call；
- peer NWDAF CRUD call。

若只修報告中的兩個 DELETE branch，其他 peer call 仍會和同一把 global mutex
綁在一起，無法真正關閉這類缺陷。

### 4.2 不在本次範圍

本次不會：

- 修改 PyAnLF 或 PyMTLF reconciler；
- 加入 distributed lock 或 distributed transaction protocol；
- 恢復已移除的 Go/Python full-state sync；
- 持久化 Go route state；
- 在 Go 或 backend process loss 後恢復原實驗；
- 修改 Model Provision、WAPE、degradation、FL round、FedAvg 或
  publication policy；
- 為一般 consumer DELETE 增加 per-peer retry worker；
- 重設計 NRF discovery 或 selected-target cache；
- 為此次修正建立泛用的跨 service resource framework 或不必要的新 package。

## 5. 併發不變條件

只有以下條件全部成立，才算完成修正：

1. `mlModelMu` 只保護短時間的 in-memory validation 與 state transition commit。
2. 持有 `mlModelMu` 時，不執行 peer、backend、callback、DNS、redirect、body read
   或其他可能阻塞的 HTTP operation。
3. context route-store lock 同樣不得跨 external I/O。
4. 同一時間最多只有一個 mutating operation 擁有某筆 route。
5. response 只能提交到當初發出它的同一筆 route operation。
6. stale response 不得刪除或覆蓋 newer route。
7. 不同 route ID 的 operation 不因彼此的 network round trip 而互相等待。
8. 已獲准的 local-backend request 在 backend I/O 期間繼續持有既有
   generation lease，但不持有 route mutex。
9. backend reset 可以在 peer operation in-flight 時移除並 tombstone route；舊
   completion 對 route state 必須成為 no-op。
10. callback 若在 route 還是 `ACTIVE` 時已被准入，可以在 delete 開始後完成；
    route 進入 `DELETING` 後不得再准入新的 callback。
11. consumer-visible method 與 status code 必須維持在 Release 18 operation
    contract 內。
12. concurrency test 不得依靠固定 sleep 製造 race。

## 6. 目標內部設計

### 6.1 維持現有責任邊界

processor 繼續負責 route business transition；context 繼續保存各 resource kind 的
typed store；consumer 繼續負責 outbound HTTP。

不需要新建 package。小型 operation helper 應放在既有 processor package，必要時
可新增聚焦的 `ml_model_route_operations.go`；各資源的 representation 邏輯仍留在
既有 Model Provision、Monitor 與 Training 檔案。

### 6.2 Route 操作版本

在 common peer-route metadata 加入一個 process-local operation revision：

```text
MLModelPeerRoute
  ...existing fields...
  LifecycleState
  OperationRevision uint64
```

processor 擁有一個由 `mlModelMu` 保護的 monotonically increasing counter。
每次開始 state-changing operation 時取得下一個值，並寫入 route。

此欄位：

- 只在 Go process 內使用；
- 不是標準 resource version；
- 不是 model revision；
- 不是 backend process generation；
- 不傳給 PyAnLF、PyMTLF、NRF 或另一個 NWDAF；
- 不跨 Go restart 持久化。

它只回答一個問題：「這個 response 是否仍屬於目前擁有此 local route 的
operation？」

使用 process-global counter，而不是每次從單一 route 的 1 開始。即使同一個
route ID 在同一個 process 內被意外重新建立，舊 response 也不會剛好對上新
operation。

### 6.3 不可變 operation snapshot

processor 在 mutex 內完成 transition 後，建立只包含該次 external call 所需資料的
immutable snapshot：

```text
route kind
local resource ID
operation revision
expected lifecycle state
direction
selected target identity（若存在）
peer Location（若存在）
backend resource ID（若存在）
process generation
related backend and generation
request 或 callback destination
本次 operation 所需的 request representation
```

snapshot 不是第二份 route，也不寫入另一個 store，只是該次函式呼叫的 local data。
釋放 `mlModelMu` 後，external I/O 只使用這份 snapshot。

### 6.4 條件式完成

external call 完成後重新取得 `mlModelMu`、讀取 typed route，只有以下條件都成立才
能 commit：

```text
resource kind 相同
and local resource ID 相同
and OperationRevision 相同
and LifecycleState 等於該 operation 預期的 in-flight state
and 相關 backend generation 仍相同
```

completion 會得到三種 ownership 結果：

| 結果 | 涵義 | 處理 |
| --- | --- | --- |
| current | 此 operation 仍擁有 route | commit success 或 rollback |
| absent／tombstoned | reset 或其他 terminal delete 已移除 route | 不得重建 route |
| superseded | 同 ID 已由 newer route／operation 擁有 | 不得修改；除非刪除已是 terminal，否則回暫時性失敗 |

route store 繼續保持 typed。不可為所有 schema 建立 reflection-based 或
`interface{}`-based 的巨大 generic CRUD engine。shared helper 只能針對四種已知
resource kind 共用 lifecycle metadata 與 ownership validation。

### 6.5 鎖取得順序

固定 lock ordering 為：

```text
processor mlModelMu
  -> 短時間操作 context route store
  -> 釋放 context lock
  -> 釋放 mlModelMu
  -> external I/O
  -> 重新取得 mlModelMu 做 conditional completion
```

不得由 outbound consumer callback 反向取得 `mlModelMu`。consumer layer 不負責
修改 route state。

availability `GenerationLease` 可以在 local backend HTTP request 期間繼續持有，
因為它是 backend-generation admission fence；但它不能隱含持有 `mlModelMu`。
Availability reset 本來就會等 admitted lease 完成，再 reset 該 backend generation。

### 6.6 鎖實際保護的範圍

`mlModelMu` 不再保護完整的 Go 通訊過程，只保護通訊前後兩段很短的
process-local 記憶體操作：

```text
通訊前：
  lock
  -> 找到 route
  -> 驗證 current lifecycle
  -> 標記 in-flight state
  -> 配置 operation revision
  -> 複製 external call 所需 snapshot
  -> unlock

通訊期間：
  peer NWDAF／Python backend HTTP
  -> 不持有 mlModelMu

通訊後：
  lock
  -> 重新讀取 route
  -> 驗證 lifecycle、operation revision、generation
  -> commit success 或 rollback
  -> unlock
```

因此另一個 NWDAF 的反向 request 可以在 HTTP 執行期間取得本機的
`mlModelMu`。mutex 不再成為跨節點 wait graph 的一部分。

### 6.7 為何解鎖執行 HTTP 仍然可靠

可靠性不建立在「解鎖期間 route 不會改變」這個假設上，而是明確允許 route
改變，再用三層 fence 防止舊結果錯誤提交：

1. `LifecycleState` 是同一筆 route 的 operation admission fence。第一個 DELETE
   把 route 從 `ACTIVE` 改成 `DELETING` 後，後來的 DELETE、REPLACE 或 PATCH
   不能再送出第二筆 conflicting external call。
2. `OperationRevision` 是 process-local optimistic concurrency fence。HTTP
   response 只有在 revision 仍等於發出 request 時的值，才有權修改 route；reset、
   replacement 或 newer operation 產生的 route 不會被舊 response 覆蓋或刪除。
3. `GenerationLease` 是 Python backend process-generation fence。已 admitted 的
   local backend request 在 I/O 期間保留 lease；availability reset 會等舊
   generation 的 request 結束，新的 Python process 不會繼承舊 request 的結果。

三者分工如下：

| 防護 | 防止的問題 | 不代表什麼 |
| --- | --- | --- |
| `LifecycleState` | 同一 route 同時啟動互相衝突的操作 | 不代表 model generation |
| `OperationRevision` | 舊 HTTP response 提交到 newer route／operation | 不傳到 wire，不是標準 revision |
| `GenerationLease` | 舊 Python process request 被當成新 process 的操作 | 不取代 route ownership |

典型競態會得到下列結果：

| 競態 | 防護結果 |
| --- | --- |
| 兩個 DELETE 同時進來 | 只有第一個取得 `DELETING` ownership；第二個不送 external DELETE |
| DELETE 期間收到 REPLACE | REPLACE 發現 `DELETING`，回 declared `503` |
| HTTP 等待期間 backend reset | old route 被移除／tombstone；舊 completion 不得重建它 |
| 相同 local ID 被重新使用 | newer route 使用不同 operation revision；舊 response 不得修改它 |
| HTTP timeout 後又發生 newer operation | timeout 只有在 revision 仍 matching 時才能 rollback 成 `ACTIVE` |

這是 optimistic concurrency control：先記錄本次操作擁有的版本，在鎖外執行慢速
工作，回來後確認 ownership 仍未改變，只有 current operation 才能 commit。相較於
持有 mutex 等待跨 process HTTP，它能同時避免 distributed lock inversion 與 stale
response corruption。

此設計可靠的必要前提是：所有 route mutation 都遵循相同的
prepare／execute／complete boundary。任何繞過 lifecycle transition 或 operation
revision check 的 create、replace、delete、callback 或 cleanup 路徑，都會破壞上述
保證。因此本計畫的 exit criterion 是完整掃描所有 ML model external I/O，而不是
只修正本次重現的兩個 DELETE handler。

## 7. 各 operation 語意

### 7.1 DELETE

DELETE 使用以下 state sequence：

```text
ACTIVE
  -> DELETING + new operation revision
  -> release mutex
  -> peer 或 backend DELETE
  -> conditional completion
```

completion 規則：

| External result | Current route 處理 | Caller result |
| --- | --- | --- |
| `204` | 移除 route | `204` |
| peer/backend `404` | 移除 route；遠端不存在視為 terminal | `204` |
| declared error，例如 `429` 或 `503` | 恢復 `ACTIVE` | 傳遞標準 `ProblemDetails` |
| transport timeout | 恢復 `ACTIVE` | 沿用既有 `503 ProblemDetails` mapping |
| malformed response | 恢復 `ACTIVE` | 沿用既有 `502 ProblemDetails` mapping |
| route 已 absent／tombstoned | 不修改 state，也不消耗 tombstone | `204` |
| 同 ID 已由 newer revision 擁有 | 不修改 state | `503 ProblemDetails` |

一般成功 DELETE 不新增 durable tombstone。route 消失後，之後另一個獨立 DELETE
回標準 `404`。tombstone 仍只用於已設計的 backend-generation loss late-delete
acknowledgement。

若另一個 request 遇到已是 `DELETING` 的 route：

- 不送出第二次 peer/backend DELETE；
- 立即回 declared `503 ProblemDetails`；
- 不持有 route mutex 等待第一個 request；
- 第一個 DELETE 成功後，後續 retry 因資源已不存在而回 `404`。

此設計保證 final state idempotent，同時不新增 in-memory future registry 或新的
internal API。已擁有 retry 行為的 consumer 可以針對暫時性 `503` 重試。

### 7.2 CREATE

CREATE 沿用 Phase 2 reservation 設計：

```text
reserve local route ID as CREATING + operation revision
  -> release mutex
  -> peer/backend POST
  -> validate status、body、media type、Location
  -> conditional ACTIVE commit
```

若 external create 成功，但 response malformed 或 reservation 已不是 current：

1. 不將 route 發布成 `ACTIVE`；
2. 在 `mlModelMu` 外執行 bounded compensation；
3. compensation 成功或 peer 回 `404` 時移除 reservation；
4. compensation 失敗時，只保留原 reservation，改成 `PENDING_CLEANUP` 並配置
   新 operation revision；
5. stale create response 不得覆蓋 newer route。

現有 backend/domain reconciler 繼續負責在 Go 回傳失敗後重試 desired resource
create；Go 不新增第二套 desired-state manager。

### 7.3 REPLACE 與 Training PATCH

active route 先以新 operation revision 進入 `REPLACING`。route 保留目前已
commit 的 representation 與 callback correlation；proposed representation 只放在
operation snapshot，成功前不寫入 active state。

```text
ACTIVE
  -> REPLACING
  -> release mutex
  -> peer/backend PUT 或 PATCH
  -> conditional completion
```

成功時 commit accepted/effective representation 並回到 `ACTIVE`。transport、peer、
backend 或 validation failure 時，以舊 committed representation 恢復 `ACTIVE`。
stale completion 不得修改 route。

`REPLACING` 期間收到 DELETE 或另一個 REPLACE 時回 declared `503`，不啟動第二個
mutation，也不新增 operation queue。

`REPLACING` 期間收到 callback，只能依最後已 commit 的 representation 與
correlation 驗證；尚未 commit 的新欄位若造成 callback 無法通過，sender 可在 replace
完成後重試。

### 7.4 Notification 與 callback delivery

notification handler 在 `mlModelMu` 內只做：

1. 找到 typed route；
2. 驗證 direction、lifecycle、correlation、model identity 與 generation；
3. 複製 committed destination 與 validation fields。

接著釋放 mutex，再送往 PyAnLF、PyMTLF 或 external callback URI。delivery
completion 不修改 route ownership，因此不需要為了回傳 HTTP result 再取得 mutex。

准入規則：

- `ACTIVE`：通過所有既有驗證後接受；
- `REPLACING`：只依最後 committed representation 接受；
- `CREATING`、`DELETING`、`PENDING_CLEANUP`：依該 callback route 的既有
  temporarily-unavailable／not-found 行為拒絕；
- route 不存在：`404`。

在 DELETE 將 route 改成 `DELETING` 之前已准入的 callback 可以完成；這只是
in-flight delivery，不是 stale route mutation。DELETE 開始後不得准入新的 callback。

### 7.5 待清理 peer 資源

`ReconcilePendingMLModelPeerCleanup` 目前會持有 global mutex，遍歷 route 並呼叫
peer。它必須改成每次對一筆 due route 執行 conditional operation：

1. mutex 內選出一筆 due `PENDING_CLEANUP` route，配置新 operation revision，
   snapshot peer Location；
2. 釋放 mutex；
3. 送出一次 bounded peer DELETE；
4. `204`／`404` 時 conditional remove；失敗時 conditional update attempt 與
   next due time；
5. 重複上述流程，但不得在整個 batch 期間一直持有 mutex。

某筆 cleanup I/O 被阻塞時，互不相干的一般 CRUD 仍要能回應。保留現有 bounded
backoff 與 logging；本計畫不將 cleanup worker 擴充成一般 consumer DELETE 的 retry
owner。

### 7.6 Backend generation 重置

`ResetMLModelBackendGeneration` 已經會在 `mlModelMu` 內收集並移除 matching route，
釋放鎖後才執行 best-effort cleanup；此形狀要保留。

加入 operation revision 後，in-flight call 的處理為：

- reset 移除並 tombstone old route；
- old peer response 之後發現 route absent，不得重建；
- 若同一 local ID 被意外重建，新的 operation revision 不會和舊 response 相同；
- local backend call 仍由 `GenerationLease` fence，reset 會等 lease 結束。

不新增 reset-time replay 或 deferred state restoration。

## 8. HTTP 與錯誤映射

修正會保留既有 standard response parsing 與 redirect handling，只改變 local state
何時能 commit。

| 狀況 | 結果 |
| --- | --- |
| operation 開始前資源不存在 | declared `404 ProblemDetails` |
| route 正在不相容的 transition | declared `503 ProblemDetails` |
| peer/backend DELETE `204` | 移除 local route，回 `204` |
| peer/backend DELETE `404` | 移除 local route，normalize 成 `204` |
| peer 回 well-formed declared error | 恢復 prior state 並傳遞該 error |
| peer/backend transport failure | 恢復 prior state，沿用既有 `503` mapping |
| peer/backend success 違反 contract | 恢復 prior state，沿用既有 `502` mapping |
| completion 已被 newer route supersede | 不修改 state，回 `503` |
| completion 發現 generation-loss tombstone | 不修改 state，回 `204` |

只要 current route 仍是 active 且遠端刪除結果未知，就不能把錯誤轉成成功。

## 9. 檔案層級實作規劃

### 9.1 `internal/context/ml_model_routes.go`

- 在 common route metadata 加入 `OperationRevision`；
- 保留 typed route record 與 clone 行為；
- 不加入 wire serialization；
- 增加 focused tests，證明 clone/update round-trip 不遺失 revision；
- 不把 context lock 暴露給 processor caller。

### 9.2 `internal/sbi/processor/processor.go`

- 將 `mlModelMu` 明確註解為短時間持有的 ML model route-transition mutex；
- 加入 monotonic operation revision counter；
- 不修改 backend 與 peer interface。

### 9.3 Processor operation helpers

在既有 processor package 中：

- 加入配置 revision 與驗證 current operation ownership 的小型 helper；
- resource-specific route lookup/update/delete 維持 typed；
- 集中 lifecycle conflict `503` 建構；
- 不建立新架構 package 或 generic persistent transaction layer。

### 9.4 `internal/sbi/processor/ml_model.go`

將 Provision 與 Monitor 的 create、replace、delete、notification delivery 改成
prepare／execute／complete boundary。保留所有現有 representation rewrite、callback
identity、correlation validation 與 status mapping。

### 9.5 `internal/sbi/processor/ml_model_training.go`

將相同 boundary 套用到 Training POST、PUT、PATCH、DELETE 與 notification
delivery。不得修改 preparation、round、final validation、`skipFlInd` 或
`mLAccChkFlg` business semantics。

### 9.6 `internal/sbi/processor/ml_model_peer.go`

- 將 remote create compensation 與 pending cleanup 改為 lock-free I/O；
- 保留既有 peer Location 與 redirect 行為；
- 以 operation revision conditional commit cleanup result；
- 保留 bounded cleanup logging 與 backoff。

### 9.7 既有 backend reset

必要時因 operation revision 調整 reset tests，但不得重新加入 state replay 或
process-loss retry。

## 10. 實作切片

### Slice A：現況鎖定測試

修改 production code 前先：

1. 用 deterministic barrier 重現 opposite-direction DELETE；
2. 證明目前 unrelated operation 會被 lock-held peer I/O 阻塞；
3. 固定目前 `204`、`404`、timeout 與 route-retention behavior；
4. characterization process-generation reset 與 deletion tombstone。

red test 必須因 concurrency 問題失敗，不得依賴極短 timeout 或 scheduler timing。

### Slice B：Route operation 防護

1. 加入 operation revision metadata 與 counter；
2. 加入 begin/current/complete helper；
3. 加入 lifecycle conflict behavior；
4. 測試 absent、current、superseded 與 tombstoned completion。

在 fence 本身有測試前，不開始搬移 external I/O。

### Slice C：四種 DELETE family

依序遷移：

1. Model Provision subscription DELETE；
2. Model Monitor subscription DELETE；
3. Model Monitor registration DELETE；
4. Model Training subscription DELETE；
5. 上述資源的 local backend DELETE branch。

完成本 Slice 後，報告中的 A/C teardown cycle 必須不再可能發生。

### Slice D：待清理資源

將 peer compensation cleanup I/O 移出 `mlModelMu`，並 conditional commit 每次結果。
驗證 blocked cleanup 不會阻塞 unrelated route。

### Slice E：剩餘 external I/O 掃描

遷移 create、replace、Training PATCH、notification、callback 與 failed-create
compensation。exit criterion 是機械式的：持有 `mlModelMu` 時，不得呼叫 peer
consumer、Python backend client 或 callback delivery helper。

### Slice F：回歸與 E2E 收尾

執行 repository tests、race tests 與相同 A/B/C teardown scenario。驗證完成後才把
實作結果寫回本文件與 Phase 7 E2E 紀錄。

## 11. 確定性測試計畫

### 11.1 方向相反的 peer DELETE

建立兩個 processor/server instance：

- A 有一筆指向 C 的 outbound Provision route；
- C 有對應的 inbound Provision route，後端為 PyMTLF stub；
- C 有一筆指向 A 的 outbound Monitor route；
- A 有對應的 inbound Monitor route，後端為 PyAnLF stub。

所有 route 建立後，以 channel 或 barrier 同時啟動兩個 outer DELETE。驗證：

- 兩個 request 都不需要等待 30 秒 peer timeout；
- 每個 peer/backend resource 正好收到一次 DELETE；
- 兩筆 outer route 都被移除；
- unrelated ML model operation 不會被阻塞到 peer timeout；
- 測試結束後沒有 goroutine 留在 wait。

### 11.2 同一資源的 concurrent DELETE

第一個 external DELETE 在 route 已進入 `DELETING` 後用 barrier 暫停，再送第二個
local DELETE：

- 第二個 request 回 declared `503`，不送第二次 external call；
- 釋放第一個 call 後得到 `204`；
- 之後再 DELETE 得到 `404`；
- external side effect 總數正好為一。

### 11.3 終止性與可重試結果

對四種 route family 做 table test：

- `204` 移除 route；
- `404` 移除 route並回 `204`；
- 該 operation 宣告的 `429`、`500`、`502`、`503` 恢復原本 `ACTIVE` route；
- transport timeout 恢復 `ACTIVE` 並映射為 `503`；
- malformed success 恢復 `ACTIVE` 並映射為 `502`。

### 11.4 I/O 期間發生 replacement 或 reset

用 barrier 暫停 external call，再：

1. reset owning backend generation；
2. 移除 old route，並用相同 ID 建立 synthetic newer route；
3. 允許 old response 完成。

驗證 old response 不能刪除、activate 或覆蓋 newer route；reset tombstone 與
late-delete acknowledgement 行為不變。

### 11.5 Create compensation 測試

強制 peer POST 成功後出現 malformed response 或 stale reservation：

- compensation 不持有 `mlModelMu`；
- compensation `204`／`404` 移除 reservation；
- compensation failure 只產生一筆 `PENDING_CLEANUP` route；
- 後續 cleanup success 只移除 matching operation revision；
- old cleanup 不得移除 newer route。

### 11.6 Replace、PATCH 與 callback race

- old PUT/PATCH completion 不得覆蓋 reset 後的 route；
- concurrent second mutation 回 `503`，且不發 peer call；
- delete 前已准入的 callback 可以完成；
- `DELETING` 後到達的 callback 不轉送；
- `REPLACING` 期間 callback 使用 committed correlation 與 representation；
- notification delivery 被 backend 或 consumer 阻塞時，不得阻塞 unrelated route
  mutation。

### 11.7 Cleanup worker 隔離性

暫停一筆 `PENDING_CLEANUP` peer DELETE，驗證：

- 另一筆 due cleanup 不會被錯誤修改；
- 一般 CRUD 仍可回應；
- failure 只在 revision 仍 matching 時更新 attempt/backoff；
- success 或 peer `404` 只移除被選中的 route。

### 11.8 Race 與 repository 驗證

必要指令：

```text
go test ./internal/context ./internal/sbi/processor ./internal/sbi/consumer
go test -race ./internal/context ./internal/sbi/processor
make test
make build
make lint
```

若 race target 需要較長 timeout，應明確調整 test runner；不得在 production 或 test
code 加入固定 sleep。

## 12. Full-core E2E 驗收

沿用既有三 NWDAF Phase 7 topology 重跑：

1. A、B 向 C 訂閱 seed model；
2. C 向 A、B 建立 monitor subscription；
3. degradation 觸發 FL 與 final model cutover；
4. old monitor resource retire；
5. consumer analytics subscription 被刪除；
6. PyAnLF 與 PyMTLF reconciler 刪除剩餘跨 NWDAF 資源；
7. processes 正常 shutdown。

驗收證據：

- peer DELETE 不等待 30 秒 NWDAF timeout；
- 不出現帶有 1／2／4／8／16 秒 retry 間隔的重複 cleanup `503`；
- 不再出現由 cleanup lock cycle 造成的
  `requested ML model service capability is temporarily unavailable`；
- 兩筆 retired monitor subscription 都獲得執行機會；
- Provision、Monitor Registration、Monitor Subscription 與 Training route
  inventory 為空，或只包含 scenario 明確保留的資源；
- 同一 operation revision 不產生 duplicate peer/backend DELETE；
- 沒有 leaked downstream subscription；
- analytics、accuracy reporting、FL training、ADRF publication、reprovision 與
  model cutover 仍能完成。

既有 Phase 7 business result 在 follow-up 尚未完成期間仍有效，但 clean teardown
必須等以上 assertions 通過才算關閉。

## 13. 可觀測性

route-operation boundary 的 structured log 應包含：

- operation name；
- resource kind；
- local route ID；
- operation revision；
- old/new lifecycle state；
- direction；
- peer NF instance ID 或 backend kind；
- status/error classification；
- elapsed duration。

不得記錄 model bytes、request body、credential 或可能含敏感 query 的完整 URI。

一般 lifecycle conflict 應使用簡潔 warning 或 info。非預期 stale completion 應以
revision evidence 記錄 warning，不得 panic。transport 與 contract failure 保留現有
error mapping。

## 14. 程式審查檢查表

review 不能只看報告中的兩個 DELETE function，必須確認：

- 每個 `mlModelMu.Lock()` block 都只包含 bounded in-memory work；
- block 內沒有 peer consumer call；
- block 內沒有 PyAnLF/PyMTLF backend call；
- block 內沒有 callback delivery；
- cleanup loop 不跨整個 batch 持有 mutex；
- 每個 state-changing external call 都有 revision-checked completion；
- 每個 failure path 都會恢復 prior state、移除 exact route，或留下唯一明確的
  `PENDING_CLEANUP`；
- generation lease 正好 release 一次；
- state conflict 使用 operation 已宣告的標準 status；
- route ID、peer Location、selected target、callback correlation 與 accepted
  representation 語意不變；
- 沒有新增 package、API、sync payload 或 durable route state。

## 15. 風險與控制

| 風險 | 控制 |
| --- | --- |
| old response 覆蓋 new route | operation revision + lifecycle/generation check |
| concurrent operation 產生 duplicate side effect | 同一 route 只有一個 mutation owner；衝突者回 `503` |
| callback 與 delete race | I/O 前先 admission snapshot；delivery completion 不修改 route |
| reset 與 peer I/O race | reset remove/tombstone；stale completion 不得重建 |
| request 期間 local backend reset | backend I/O 期間保留 `GenerationLease` |
| cleanup worker 阻塞所有 ML model traffic | single-route snapshot + lock-free peer I/O |
| broad refactor 改變 business logic | 保留 wire/parser/representation helper，只遷移 operation boundary |
| operation revision 被誤認成 model identity | internal-only naming，不序列化，加入 focused comments/tests |
| 遺漏相同模式的路徑 | 機械式掃描所有持有 `mlModelMu` 的 peer/backend/callback call |

## 16. 完成條件

以下條件全部成立才算完成：

1. deterministic opposite-direction DELETE 不發生 timeout；
2. 四種 DELETE family 都使用 `DELETING` 與 revision-checked completion；
3. 持有 `mlModelMu` 時沒有 external HTTP call；
4. same-route concurrent mutation 只產生一個 side effect，另一方得到 declared
   transient response；
5. reset/replacement test 證明 stale response 無法修改 newer route；
6. pending cleanup 已 lock-free 且具 revision guard；
7. targeted/full Go tests、`go test -race`、build 與 lint 通過；
8. 既有 full-core business flow 仍通過；
9. teardown 不再出現重複 30 秒 cleanup timeout 或 leaked route；
10. 本文件記錄實作結果與 exact revisions。

### 16.1 2026-08-13 實作與審查結果

NWDAF 實作已在 `318ac19d8b027373f4468660394da1ec3338268e` 基線上完成，最終
implementation revision 為 `c53f05804c6533ec4c5fa7e230e90048fb219162`。完成內容包括：

- 四種 ML model route family 的 create、replace、PATCH、DELETE、callback 與
  pending cleanup 都改成 prepare／execute／conditional-complete；
- `MLModelPeerRoute.OperationRevision` 與 process-local monotonic counter 已成為
  stale completion fence；
- peer、PyAnLF、PyMTLF 與 external callback I/O 均移出 `mlModelMu`；
- backend generation reset 仍先移除及 tombstone route，再於鎖外 best-effort
  cleanup，沒有加入 replay 或 external sync；
- opposite-direction Provision／Monitor DELETE、same-route concurrent DELETE、
  reset、same-ID replacement、pending cleanup ownership 與 callback-during-replace
  都有 deterministic regression test；
- follow-up review 修正 Training PUT/PATCH 在 peer `404` 時誤刪 local route、
  cleanup 把 `nil` response 誤判成功，以及 failed-create 進入
  `PENDING_CLEANUP` 時未配置新 operation revision 的問題；
- review 也撤回會改變既有 monitor registration／subscription 線性化語意的
  owner recheck，不把 concurrency 修正擴大成新的 parent-child cleanup policy。

已通過：

```text
go test ./...
go test -race ./internal/context ./internal/sbi/processor
golangci-lint run ./...
go build ./...
```

另以機械式掃描確認所有 `mlModelMu` critical section 內沒有 peer consumer、Python
backend client 或 callback HTTP call。

### 16.2 2026-08-15 Full-core E2E 驗收結果

修正後的 full-core A/B/C E2E 已完成重跑。驗收使用的整合版本如下：

| Repository | Revision |
| --- | --- |
| `NWDAF/` | `c53f05804c6533ec4c5fa7e230e90048fb219162` |
| `PyAnLF/` | `08798f15c3693027e00bc60dd53f74ebaa26c3a1` |
| `PyMTLF/` | `7e8ab7f23bf5d6398eb1cd5f053dd8bda9439a87` |
| `nwdaf-resources/` | `8a7619ec9c745f71a8fea42134cefd550ad2c180` |

驗收結果：

- A、C 方向相反的 Model Provision／Model Monitor 資源可以並行清理；
- teardown 不再出現因 cross-node lock inversion 造成的 30 秒 peer timeout；
- DELETE 不再因該互鎖反覆回傳 `503 Service Unavailable`；
- analytics、model provision、model monitoring 與 FL 閉環在修正後仍可完成；
- teardown 可以收斂，未觀察到本問題造成的殘留 route 或 downstream
  subscription。

因此 §12 與 §16 的 full-core 驗收門檻已關閉，本 remediation 完成。

## 17. 決策門檻

進入實作前不需要新的產品或標準行為決策。本計畫修正的是已核准 Phase 2
concurrency semantics 的實作偏差。

只有實作證據顯示必須進行下列任一變更時，才需要回到設計討論：

- 新增 public/private API 或 wire field；
- 加入 durable Go route persistence；
- 改變 PyAnLF/PyMTLF retry ownership；
- 回傳不在對應 Release 18 operation 中宣告的 status；
- 為了未記錄的 dependency 而 serialize unrelated resource；
- 改變 Model Provision、monitoring 或 FL business behavior。

## 18. 證據對照

主要實作證據：

- `NWDAF/internal/sbi/processor/processor.go`：process-global ML model mutex；
- `NWDAF/internal/sbi/processor/ml_model.go`：Provision 與 Monitor operation；
- `NWDAF/internal/sbi/processor/ml_model_training.go`：Training operation；
- `NWDAF/internal/sbi/processor/ml_model_peer.go`：peer create compensation 與
  cleanup worker；
- `NWDAF/internal/sbi/consumer/ml_model_peer_service.go`：30 秒 peer timeout；
- `NWDAF/internal/context/ml_model_routes.go`：typed route store 與 lifecycle；
- `NWDAF/internal/sbi/processor/ml_model_backend_reset.go`：既有的 lock-free
  best-effort reset cleanup 形狀；
- `NWDAF/internal/backend/availability_monitor.go`：generation admission lease。

規格證據：

- TS 29.520 Release 18 OpenAPI `Nnwdaf_MLModelProvision` DELETE operation；
- TS 29.520 Release 18 OpenAPI `Nnwdaf_MLModelMonitor` registration/subscription
  DELETE operation；
- TS 29.520 Release 18 OpenAPI `Nnwdaf_MLModelTraining` DELETE operation。

專案決策證據：

- Phase 2 §7、§16、§17 定義 route lifecycle、retry ownership、lock-free network
  call 與 revision guard；
- backend process-failure 計畫定義 stateless reset 與 no replay；
- Phase 7 記錄 full-core topology 與 teardown verification boundary。
