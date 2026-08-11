# Phase 7 PyAnLF Collection Runtime Revision Race Remediation Plan

Date: 2026-08-11

Status: Implemented; full-core E2E verification pending

Parent documents:

- `../Phase 7 Full-Core Federated Learning Business E2E Detailed Plan.md`
- `../../mtlf-backend-transition/Phase 8 Backend Process Failure And Stateless Reconnection.md`

Affected repositories:

- `PyAnLF/`
- `nwdaf-docs/`

## 1. Purpose

本文件記錄 full-core federated-learning E2E 在 initial model activation 時暴露的
PyAnLF internal collection/runtime revision race，並定義一個範圍受限的修正。

本修正不重開既有架構，也不改變以下行為：

- analytics subscription 建立後可在等待模型期間先建立資料蒐集資源；
- initial model provision 使 pending runtime 進入可推論狀態；
- collection resource 依既有 key 與 reference ownership 複用；
- model-only activation 不應取消並重建需求未變的 SMF／UPF subscription；
- 後續 completed model cutover 使用 model generation，不因本修正改成重建
  analytics runtime；
- Go、PyAnLF 與 PyMTLF 間不重新加入已移除的 full-state external sync。

## 2. Evidence And Reproduction

問題在 2026-08-11 的 full-core E2E 重跑中於 NWDAF-A 與 NWDAF-B 各出現一次。

測試版本：

```text
Repository: PyAnLF
Revision: 5c305c7b69a50e9356bcfca8229f1a3cffd11a9a
Working tree: clean
Infrastructure commit: 75cae80
```

兩側共同時序為：

```text
analytics subscription 建立 runtime revision N
  -> collection reconcile N 開始建立或取得 bindings
  -> initial model provision 啟用模型
  -> SubscriptionRuntimeManager 發布 runtime revision N+1
  -> reconcile N 嘗試 sync_bindings(N)
  -> RuntimeManager 拒絕舊 revision
  -> StaleRuntimeRevisionError 被 generic worker handler 記成 ERROR
```

E2E 後續仍完成 UPF callbacks、analytics、accuracy reports、FL training、final
publication 與 reprovision。這證明問題是可恢復的 concurrency defect，不是整體
collection 或 FL failure；成功結果不代表該 race 可以忽略。

## 3. Current Revision Semantics

本問題中的 `revision` 是單一 analytics subscription runtime 的 process-local
版本號，不是 Model Monitor resource revision，也不是 completed model revision。

目前正常產生新 runtime revision 的 production 路徑只有：

1. 建立 analytics subscription；
2. consumer replace analytics subscription；
3. 沒有 active model 的 runtime 首次啟用 provisioned model。

下列行為不增加此 revision：

- observation ingestion 與 analytics reporting；
- collection／provision binding 更新；
- accuracy report；
- 已有 active model 後的 model generation cutover；
- subscription delete。Delete 直接移除 runtime，CollectionManager 另以自己的
  invalidation revision 讓未完成 task 失效。

RuntimeManager 的 revision fence 是必要的 correctness boundary：舊 task 不得把
binding 覆蓋到新的 runtime。此次要修的是 stale work 的結束與接手機制，不是
移除 fence。

## 4. Confirmed Root Cause

CollectionManager 與 SubscriptionRuntimeManager 目前有兩個不同的 current-state
view：

- CollectionManager 的 `_latest_revisions` 只在 `reconcile()`／`release()` 時更新；
- RuntimeManager 的 active runtime revision 在 `apply()` publication 時立即更新。

initial model activation 的順序是先發布 N+1，再安排 collection reconcile N+1。
因此兩者之間存在合法但短暫的 window：

```text
RuntimeManager current revision = N+1
CollectionManager latest revision = N
```

舊 task 在這個 window 中會通過 CollectionManager 自己的 `_is_current()`，直到
`sync_bindings()` 才被 RuntimeManager 的 authoritative fence 拒絕。

worker 又把所有 exception 都視為一般失敗，所以：

- 輸出 `Collection task failed` ERROR traceback；
- 建立相同 revision、只增加 attempt 的 retry task。

在本次 initial activation 路徑，稍後排入的 N+1 task 會依 queue coalescing 規則
壓掉 N 的 retry，因此只看到一次錯誤並成功恢復。但若未來某條 runtime publication
路徑沒有安排新版 reconcile，相同 stale revision 將永遠不可能重新有效。

## 5. Safety Assessment Before Remediation

已確認的安全界線：

- `sync_bindings()` 在修改 runtime bindings、observation store 與 scheduler 前先
  驗證 revision，因此舊 task 沒有覆蓋 N+1 runtime；
- seed model activation 不改變 Target UE、AoI、SMF endpoint、sampling interval
  或 required measurements；
- 已建立的 collection resource 仍由現有 key、subscription references 與
  candidate bindings 管理；stale binding failure 本身不送出 SMF DELETE；
- N+1 reconcile 會使用既有 collection resource，不應建立第二份相同 peer
  subscription。

仍需用 deterministic test 證明的事項：

- task 在 acquire 完成後才變 stale 時，不產生 duplicate peer POST；
- stale task 不送 peer DELETE；
- N+1 最終只保留一組 bindings 與 references；
- shutdown、replacement 與 normal collection retry 沒有因 exception 分類調整而
  regression。

## 6. Remediation Decisions

1. RuntimeManager 是 active runtime revision 的唯一 authoritative source。
2. CollectionManager 仍保留 process-local latest-task marker，用於 queue
   coalescing 與 delete invalidation；它不是第二份 runtime revision，也不能單獨
   證明 task 仍有效。
3. `StaleRuntimeRevisionError` 表示 task 已被較新 runtime 取代；
   `SubscriptionRuntimeNotFoundError` 表示 runtime 已被移除。兩者對該 task 都是
   terminal cancellation，不是 retryable collection failure。
4. stale task 不得重試相同 revision，也不得只降低 log level 後繼續原邏輯。
5. 不把舊 task 的 revision 改成最新 revision後重跑。task 內含 deep-copied
   subscription snapshot；若 consumer replace 改變 collection requirements，這會
   把舊 snapshot 錯綁到新 runtime。
6. 每個 active runtime publication 必須由掌握正確 subscription snapshot 的
   producer 安排同 revision reconcile。
7. model activation 沒有改變 collection requirements 時，N+1 必須複用既有
   peer subscription；不得用 delete-and-recreate 迴避 race。
8. 不修改 WAPE、accuracy policy、model provision、inference、report scheduling
   或 collection key 語意。
9. 不加入 external sync、跨 process revision 或 durable recovery。

CollectionManager 的既有 `_latest_revisions` 應改名為能表達實際用途的名稱，例如
`_latest_task_revisions`。這份 map 只能回答「這是否仍是最新排入的 collection
task」，不得用來：

- 推斷 active runtime 是否存在；
- 判斷 runtime 現在是否仍為 task 攜帶的 revision；
- 授權 `sync_bindings()` mutation；
- 自動將舊 task 升級成較新的 runtime revision；
- 對外傳輸、持久化或參與跨 process recovery。

本修正不需要新的產品決策。

## 7. Implementation Plan

### 7.1 Add An Authoritative Revision Query

在 `SubscriptionRuntimeManager` 提供最小 read-only 方法，例如
`is_runtime_revision_current()`，以同一把 state lock 判斷：

```text
subscription 是否仍存在
and active runtime revision 是否等於 task revision
```

既有 reporting scheduler 的 private current-runtime 判斷應共用同一語意，不再讓
CollectionManager 自行讀取或保存 RuntimeManager 內部物件。

此 API 是 PyAnLF process 內部方法，不新增 HTTP endpoint，也不跨 Go boundary。

### 7.2 Separate Task Currency From Runtime Currency

目前含糊的 `_is_current()` 語意應拆成兩個明確判斷：

```text
is_latest_collection_task()
  -> 只讀 CollectionManager latest-task marker

is_runtime_revision_current()
  -> 只由 RuntimeManager 判斷 active runtime revision
```

一般 reconcile task 必須同時滿足：

1. CollectionManager latest-task marker 等於 task 攜帶的 runtime revision；
2. RuntimeManager active revision 等於 task revision。

release task 的 runtime 已可能被移除，因此仍使用 CollectionManager 的 release
invalidation marker，不要求 RuntimeManager 中存在 active runtime。

即使 CollectionManager 尚未收到 N+1 task、latest-task marker 仍是 N，只要
RuntimeManager 已發布 N+1，task N 就必須停止。相反地，CollectionManager 的
marker 已是 N+1但 RuntimeManager 尚未發布 N+1 時，N+1也不得寫入；task marker
永遠不能取代 authoritative runtime check。

重要 side-effect 前保留既有 currency checks；`sync_bindings()` 的 revision check
繼續作為最後一道 atomic write fence，以處理 check 與 write 之間仍可能發生的
更新。

### 7.3 Classify Stale Work Separately

worker 與 synchronous reconcile path 單獨處理 `StaleRuntimeRevisionError` 與
`SubscriptionRuntimeNotFoundError`：

- 記錄 concise debug／info superseded-task message；
- 不輸出一般 failure traceback；
- 回傳 no-retry；
- 不建立相同 revision 的 retry task。

SMF discovery failure、peer create／delete transport failure、finite lease 不可用等
真正暫時性錯誤，繼續使用現有 backoff retry，不能因本修正一起被吞掉。

### 7.4 Guarantee The Replacement Task

審查並測試所有 active runtime publication call sites：

- analytics create；
- analytics replace；
- initial provisioned-model activation。

每條路徑都必須在同一個 subscription-service critical section 中，以該 revision
對應的最新 subscription snapshot 呼叫 `CollectionManager.reconcile()`。Initial
activation 應在 runtime publication 成功後立即保證 N+1 已被安排，不依賴舊 task
自行猜測或升級 revision。

以下路徑不新增 reconcile：

- failed runtime publication 後立即 release，因為沒有 active runtime；
- existing-model generation cutover，因為它不建立新 runtime revision且不改變
  collection requirements；
- delete，沿用明確 `CollectionManager.release()`。

### 7.5 Preserve Resource Reuse

不調整 collection key、reference set、candidate binding 或 peer cleanup 演算法。
N+1 reconcile 仍計算 desired/current 差異：

```text
requirements unchanged
  -> desired key already active
  -> retain existing SMF/UPF subscription
  -> bind the same observation source to runtime N+1
```

只有真正改變 requirements 的 analytics replace 才依現有差異演算法新增或釋放
peer resource。

## 8. Test-First Remediation

依 development policy，先加入 deterministic failing test，再修改 production
code。測試使用 `threading.Event`／barrier 控制順序，不使用 `sleep()` 猜時序。

### 8.1 Required Concurrency Test

強制以下順序：

1. 建立 revision N 並開始 reconcile；
2. 讓 N 完成或取得既有 collection resource，停在 `sync_bindings()` 前；
3. 啟用 initial model，發布 revision N+1並排入 N+1 reconcile；
4. 放行 N；
5. 等待 queue drain 與 N+1 bindings 完成。

必須驗證：

- N 不會更新 N+1 runtime；
- N 不進 retry；
- 沒有 `Collection task failed` ERROR traceback；
- N+1 `bindings_synced=true`；
- peer create 次數保持一；
- peer delete 次數為零；
- 只存在一組 active resource、reference 與 observation binding。

### 8.2 Focused Regression Tests

- local latest 已先更新時，舊 task 在任何 peer side effect 前停止；
- RuntimeManager 已先更新、CollectionManager latest-task marker 仍為 N且尚未
  收到 N+1 task 時，舊 task也停止；
- CollectionManager marker 與 RuntimeManager revision 不一致時，兩者任何一方
  都不能單獨授權 binding mutation；
- check 後、`sync_bindings()` 前才更新 revision 或移除 runtime 時，atomic fence
  被當成 superseded／inactive cancellation；
- normal retryable SMF failure仍依原 backoff 重試；
- consumer replace 使用新 snapshot，不把舊 task自動升級；
- delete／shutdown 正常釋放 peer resource；
- existing-model generation cutover 不造成不必要的 collection reconcile。

## 9. Verification

PyAnLF focused verification：

```text
tests/test_collection.py
tests/test_runtime_manager.py
tests/test_events_subscription_api.py
tests/test_model_demand.py
```

接著執行：

- complete PyAnLF test suite；
- PyAnLF Ruff check；
- 既有 portable model provision／monitor regression；
- Phase 6 full-core collection regression；
- Phase 7 full-core FL E2E 重跑。

E2E 必須確認 A／B 都不再出現 stale collection ERROR，且 UPF callbacks、analytics、
accuracy、seed provision、monitor、FL、publication 與 reprovision 仍完整成功。

## 10. Completion Criteria

本 finding 只有在下列條件全部成立後才能關閉：

1. stale 或 inactive collection task 是 no-retry terminal cancellation；
2. authoritative runtime revision 與 local latest-task marker 各自履行獨立檢查，
   且 local marker 不被當成 runtime state；
3. 所有 active runtime revision publication 都有正確 snapshot 的 replacement
   reconcile；
4. stale task 不覆蓋 runtime、不重試舊 revision，也不輸出一般 ERROR traceback；
5. requirements 未變時沿用同一 peer collection subscription；
6. requirements 改變時既有 reconcile 差異語意不變；
7. deterministic concurrency tests 證明沒有 duplicate／missing／leaked resources；
8. focused、full PyAnLF、Ruff 與相關 E2E regression 全部通過；
9. Phase 7 follow-up record 回填實際 revision、test counts 與 E2E evidence。

## 11. Non-Goals

本修正不處理：

- external backend process sync 或 restart recovery；
- Go route lifecycle；
- PyMTLF scheduling、training 或 model management；
- model generation／identity redesign；
- collection key、SMF discovery、AoI 或 group-to-SUPI resolution redesign；
- scikit-learn version warning；
- 跨 process exactly-once 或 durable queue。

## 12. Implementation Record（2026-08-11）

### 12.1 Implemented behavior

PyAnLF 已完成以下修改：

- `SubscriptionRuntimeManager.is_runtime_revision_current()` 成為 active runtime
  revision 的 lock-protected read-only query，reporting scheduler 與
  CollectionManager 共用相同語意；
- CollectionManager `_latest_revisions` 改名為 `_latest_task_revisions`，並以註解
  固定它只負責 process-local task supersession 與 release invalidation；
- collection currency 拆成 latest-task marker 與 authoritative runtime revision
  兩項檢查；release task 仍只依 local invalidation marker；
- worker 與 synchronous reconcile 將 stale revision及已移除 runtime 視為
  no-retry terminal cancellation，不再輸出一般 failure traceback；
- create、replace 與 initial model activation 的既有 replacement reconcile 路徑
  由測試固定；existing-model generation cutover 仍不增加 runtime revision；
- collection key、reference、peer create／delete 與 cleanup 演算法沒有改變。

### 12.2 Deterministic evidence

新增的 concurrency coverage 以 `threading.Event` 控制時序，證明：

1. CollectionManager marker 仍為 N、RuntimeManager 已為 N+1 時，N 在 peer side
   effect 前停止；
2. stale 或 missing runtime exception 不產生相同 revision retry；
3. N 已取得一份 SMF collection resource並停在 `sync_bindings(N)` 時，N+1可接手
   同一 resource；peer POST 維持一次、peer DELETE 為零；
4. initial provisioned model activation 會排入 revision N+1 的最新 subscription
   snapshot；
5. authoritative query 在 apply、replacement 與 release 後回報正確結果。

### 12.3 Verification record

- PyAnLF focused collection tests：`20 passed`；
- PyAnLF complete suite：`282 passed, 2 skipped`；
- PyAnLF Ruff：passed；
- `git diff --check`：passed。

尚未在本次 remediation 中重跑 Phase 6 full-core collection 與 Phase 7 full-core
FL E2E，因此本 ledger 保持「implementation complete, E2E pending」，不宣稱第10節
全部完成。完整 E2E 驗收後應補入實際 PyAnLF revision、infrastructure revision、
resource counts 與 A／B log evidence，再將本文件關閉。
