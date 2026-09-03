# Flat FL 本機 Artifact Self-download 修正詳細計畫

日期：2026-08-25

狀態：Locally Completed；production remediation、mandatory review、必要本機驗證、使用者
IDE review 與第二批 repository-separated commits 均已完成，進入 `Testbed Validation Pending`

上層計畫：

- [Hierarchical NWDAF Federated Learning Implementation Plan](./Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)
- [Slice 8 Multi-process E2E and Regression Closure Detailed Plan](./Slice%208%20Multi-process%20E2E%20and%20Regression%20Closure%20Detailed%20Plan.md)

既有 flat FL 設計：

- [Distributed NWDAF Federated Learning Implementation Plan](../../federated-learning/Distributed%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)
- [Phase 3 And 4 Federated Training Execution Detailed Plan](../../federated-learning/Phase%203%20And%204%20Federated%20Training%20Execution%20Detailed%20Plan.md)

開發規則：

- [NWDAF Development Policy](../../../development_policy.md)

第一批已建立 commits：

- PyMTLF：`579b86e`、`3628068`
- PyAnLF：`6a4d94a`
- nwdaf-resources：`213d031`、`39ced28`
- nwdaf-docs：`f5c6186`、`4a5aaad`

第二批已建立 commits：

- PyMTLF：`e9aa223`
- nwdaf-docs：`f2d0175`

---

## 1. 目的與修正終點

本計畫修正 flat `FLServerEngine` 在同一個 PyMTLF process 內，將自己剛發布且仍持有的 artifact
URL 交回 `FLWorkspace.download()` 的三條 production paths：

1. 第二輪以後重新下載上一輪的 `ROUND_GLOBAL`；
2. 每輪聚合時重新下載當輪 Server 自己發布的 `ROUND_INPUT`；
3. final validation evaluation 重新下載最後一輪的本機 candidate。

修正終點是：Server 在本機 state 中持續保存 authoritative artifact handle，URL 只作為傳給 peer
participants 的 wire projection 與 diagnostics。只有 peer-owned `ROUND_LOCAL` 與 validation result
URLs 維持 origin-checked remote download。

本計畫不改變 flat FL algorithm、round identity、artifact schema、public SBI、NRF discovery、final
validation gate、publication 或 cleanup semantics。

---

## 2. Baseline 與直接證據

### 2.1 計畫基準 revisions

| Repository | Branch | Baseline revision | 角色 |
| --- | --- | --- | --- |
| `PyMTLF/` | `feat/r18-hierarchical-federated-learning` | `3628068` | 唯一預定 production owner |
| `nwdaf-resources/` | `feat/r18-hierarchical-federated-learning` | `39ced28` | 既有 E2E harness；預設只驗證 |
| `nwdaf-docs/` | `main` | `4a5aaad` | 計畫與驗證紀錄 owner |

### 2.2 既有 flat flow 的正確語意

Phase 3／4 與 distributed FL 主計畫要求：

- round 0 使用 current catalog base；
- round `r > 0` 使用 `ROUND_GLOBAL(r-1)`；
- Server 發布 `ROUND_INPUT(r)` 給 Clients；
- Clients 下載 input、訓練並發布 peer-owned `ROUND_LOCAL(r)`；
- Server 下載並驗證各 participant result，再發布 `ROUND_GLOBAL(r)`；
- 最後一輪 `ROUND_GLOBAL` 成為未 promotion 的 candidate，送給 Clients 執行 validation-only；
- Server 收集 peer-owned validation results，通過 gate 後才進入 publication。

這些設計只要求跨 process 邊界以 immutable URL 傳遞 artifact，沒有要求 artifact producer 經 HTTP
下載自己持有的 bytes。修正因此是 process-local ownership correction，不是 wire contract 變更。

### 2.3 現行三條 self-download 路徑

| 階段 | 現行 authoritative state | 現行下載 | 問題 |
| --- | --- | --- | --- |
| 下一輪 source | `process.current_global_url` | `_run()` 呼叫 `workspace.download()` | 上一輪 `publish()` 已回傳本機 artifact，卻只保留 URL |
| 當輪 aggregation base | `round_input.url` | `_aggregate_round()` 的 optional fallback | `publish_round_input()` 的 artifact handle 已在 caller 手上 |
| final candidate evaluation | `process.candidate_url` | `_evaluate_final_validation()` 呼叫 `workspace.download()` | 最後一輪 aggregate 已由同一 Server 發布並持有 |

`FLWorkspace.download()` 會執行 allowed-origin 檢查、HTTP GET、compressed-size limit、archive
validation 與 download-copy ownership。這些是 remote artifact 的正確 trust boundary，不應用來讀取
同一 workspace 已發布的本機 artifact。

---

## 3. 根因

`FLProcess` 目前對 round progression 只保存 `current_global_url`，而 `_run()` 在聚合後又只從
`FLWorkspaceArtifact` 取 `.url`。因此 producer 已取得的 `path`、digest 與 manifest 沒有延續到
下一個 stage。

同樣地，`_aggregate_round()` 雖已為 hierarchy Root／Branch 支援 caller 傳入
`round_input_artifact`，flat caller 尚未傳入；optional URL fallback 便繼續下載 Server 自己發布的
`ROUND_INPUT`。`_evaluate_final_validation()` 則只有 candidate URL 參數來源，沒有接收本機
candidate metadata。

問題不是 allowed-origin 設定過窄。把 Server 自己的 origin 加入 allowlist 只會掩蓋 ownership
錯誤，增加一次不必要的 HTTP、壓縮檔複製與 failure surface，並讓本機與 peer trust boundary
混在一起。

---

## 4. Ownership 與資料流 invariant

修正後固定下列 invariant：

1. `FLWorkspace.publish*()` 回傳值是 Server-owned artifact 的 authoritative process-local handle；
2. catalog base 使用既有 `ArtifactMetadata`，不為了統一型別複製或下載；
3. `current_global_url` 與 `candidate_url` 可保留作 wire／status projection，但不能單獨授權本機讀取；
4. 本機讀取必須使用已保存的 path 與 digest，且 URL 必須與投影值一致；
5. participant notification 中的 `ROUND_LOCAL` 與 validation URLs 仍是 peer-owned，必須經
   `FLWorkspace.download()` 與既有 identity／contract validation；
6. 本機 artifact 缺失或 URL／handle 不一致時明確失敗，不 fallback 成 self-download；
7. artifact 仍由既有 `owner_plan_id`／process workspace lifecycle cleanup，不新增 durable index、
   HTTP cache 或 recovery state；
8. restart 仍採 fresh-state，不保存或恢復 active flat process artifact handle。

---

## 5. Production 實作計畫

### 5.1 保存 current global artifact

在 `FLProcess` 增加 process-local `current_global_artifact`，以 `ArtifactMetadata | None` 表達可直接
交給 bundle loader 的 source：

- preparation 完成後設為 current catalog artifact；
- 每輪 aggregation 後，把回傳的 `FLWorkspaceArtifact` 正規化為指向同一路徑、digest 與 URL 的
  `ArtifactMetadata`；
- `current_global_url` 同步投影同一 artifact URL；
- 下一輪直接從 `current_global_artifact` load，不呼叫 downloader。

正規化只建立 metadata value，不複製 bytes，也不改變 workspace ownership。

### 5.2 讓 aggregation 必須取得本機 round input

flat `_run()` 呼叫 `_aggregate_round()` 時傳入剛由 `publish_round_input()` 取得的
`FLWorkspaceArtifact`。

重新檢查全部 production callers 後，Root 與 Branch hierarchy paths 也都已持有並傳入本機
round input。故 `_aggregate_round()` 不再保留 URL-only download fallback：到達 aggregation 的
caller 必須同時提供 local artifact，方法繼續驗證 artifact URL 等於 dispatch URL，再以本機 path
load 與驗證 `ROUND_INPUT` identity。

在 timeout、participant failure 或 generation abort 發生於 aggregation 前的路徑，可以維持尚未
使用 artifact 的既有呼叫形狀；但任何真正進入 `_aggregate_round()` 的 production path 都不得缺少
local artifact。

### 5.3 讓 final validation 使用本機 candidate

最後一輪 aggregation 後，以同一份 current global metadata 作為 candidate handle：

- `candidate_url` 仍傳給 participants；
- `_evaluate_final_validation()` 明確接收 candidate `ArtifactMetadata`；
- 先驗證其 URL 與 `process.candidate_url` 相同，再直接 load；
- `process.candidate_artifact` 保存同一 metadata，供既有 publication owner 使用；
- peer validation result URLs 繼續逐一 download。

不得新增「本機失敗就用 candidate URL 下載」的 fallback。

### 5.4 不改變的 owner 與 contracts

- `FLWorkspace.download()` 本身不修改，也不加入 self-origin special case；
- `FLWorkspace.publish()`／`publish_round_input()` schema 與 serving behavior 不修改；
- `ROUND_INPUT`、`ROUND_LOCAL`、`ROUND_GLOBAL`、candidate 與 `FINAL_MODEL` roles 不修改；
- Go NWDAF、PyAnLF、NRF、ADRF 與 standard Training SBI payload 不修改；
- participant download、origin allowlist 與 remote artifact identity validation 不放寬。

---

## 6. 直接回歸測試

在 `PyMTLF/tests/test_fl_server.py` 至少加入或調整三條獨立證據：

1. **下一輪 source**：兩輪 flat execution 保存第一輪 aggregate handle，第二輪 loader 直接使用該
   metadata；`workspace.download()` 不曾收到 Server-owned `ROUND_GLOBAL` URL。
2. **aggregation input**：flat caller 將 `publish_round_input()` 回傳 object 原樣交給
   `_aggregate_round()`；aggregation 不下載該 `ROUND_INPUT`，但仍下載 participant
   `ROUND_LOCAL` URLs。
3. **final candidate**：`_evaluate_final_validation()` 直接 load caller 提供的本機 candidate；
   downloader calls 只能包含 participant validation result URLs，不能包含 candidate URL。

同步更新 `_aggregate_round()` 既有 tests，使合法 aggregation 明確提供 local round-input
artifact。另保留或新增 negative assertions：

- artifact URL 與 dispatch URL 不一致時拒絕；
- local artifact path 缺失或 bundle identity 不符時不進行 fallback download；
- hierarchy Root／Branch owned-input regression 持續通過；
- peer-owned participant artifacts 仍呼叫 downloader。

測試不得只斷言最終 state；必須直接檢查 downloader call list 與傳入 loader 的 artifact identity。

---

## 7. Repository 範圍

### `PyMTLF/`

預定修改：

- `src/py_mtlf/core/fl_server.py`
- `tests/test_fl_server.py`

若 direct evidence 顯示需要調整共用 conversion helper，可在同 repository 最小擴充；不得先行修改
`FLWorkspace.download()` 或其他 engine。

### `nwdaf-resources/`

預設不修改。使用 commits `213d031`、`39ced28` 已建立的 runner 驗證 flat isolated 與 hierarchy
smoke regression。只有 runner 本身暴露可重現 defect 時，才依 finding admission gate 另行納入。

### `nwdaf-docs/`

維護本計畫、Slice 8 狀態、上層 HFL plan 連結與 implementation record。

其他 repositories 不在本 remediation 的預定修改範圍。

---

## 8. 驗證矩陣

### 8.1 聚焦測試

先執行三條 direct regressions與受影響的既有 Server tests；exact node IDs 在實作後回填，不以只跑
整檔取代 direct path evidence。

### 8.2 PyMTLF 完整驗證

```bash
cd PyMTLF
.venv/bin/pytest -q
.venv/bin/ruff check src tests
git diff --check
```

### 8.3 真實程序回歸

從 workspace root 分別執行：

```bash
PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/distributed_fl/scripts/run.py

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/scripts/run.py \
  --profile smoke --scenario manual-success
```

flat isolated 是本修正的主要 E2E；hierarchy smoke 會同時走 Root 與 Branch owned round-input
aggregation，作為共用 `_aggregate_round()` contract 的受影響 regression。不得用 unit tests 取代
這兩個 real-process runs。

若 remediation 修改 `nwdaf-resources`，再額外執行其 support tests、Ruff 與 `git diff --check`；
若沒有修改，只記錄 runner 使用的 exact revision。

---

## 9. 失敗與 lifecycle 行為

- 本機 source 缺失、無法讀取或 contract 不符：目前 process terminal failure；不發 HTTP GET
  嘗試修復；
- peer download、digest、identity 或 contract failure：維持既有 terminal behavior；
- round timeout／participant failure 在 aggregation 前發生：不產生 aggregate，也不要求讀取本機
  input；
- validation rejected：candidate 依既有 lifecycle cleanup；
- publication accepted：同一 candidate metadata 交給既有 publication coordinator；
- graceful shutdown、Go generation change 與 PyMTLF restart：維持既有 cleanup／fresh-state
  semantics，不新增 recovery。

---

## 10. 明確不在本計畫

- 修改 artifact URL 或 HTTP serving contract；
- 將 local path 傳到其他 process；
- 對 peer artifact 增加 shared-filesystem shortcut；
- 放寬 allowed origins 或停用 remote download validation；
- deduplicate 不同 participants 的 remote downloads；
- redesign `ArtifactMetadata`／`FLWorkspaceArtifact` 的全 repository type hierarchy；
- active process persistence、restart resume 或 remote reconciliation；
- OAuth、TLS、cross-host 或 testbed transport 設計。

testbed validation 是第二批 commits 後的獨立 acceptance gate；其 VM topology、transport profile
與 required scenario matrix 仍須在部署前與使用者確認，不能在本計畫中自行假設。

---

## 11. Review 與 finding gate

實作完成後必須不中斷進行 mandatory initial review，至少檢查：

- 所有本機 producer／consumer path 是否都有 handle，沒有第四條 URL-only self-download；
- URL 是否仍正確傳給 peers；
- peer-owned downloads 是否仍受 origin 與 artifact identity validation；
- local handle 是否跨 cleanup、generation 或 process boundary 被錯誤保存；
- `_aggregate_round()` fallback 移除後是否存在未確認的 production caller；
- candidate metadata 是否仍符合 publication owner 的 reader／cleanup lifecycle。

finding 只有在直接阻擋本 remediation invariant 或 acceptance criteria 時才納入；較廣泛 artifact
type refactor、download cache 或 resilience 改善依 policy 分類記錄。

---

## 12. 完成條件

本 remediation 只有在下列條件全部成立後才能進入第二批 commit proposal：

1. 三條 Server-owned self-download production paths 均已移除；
2. flat process state 保存 current global artifact handle，URL 只作 wire／diagnostic projection；
3. `_aggregate_round()` 的所有 production callers 都提供本機 round-input artifact，URL-only
   fallback 沒有合法 production caller且已移除；
4. final validation 使用本機 candidate metadata，publication 取得同一 candidate identity；
5. participant `ROUND_LOCAL` 與 validation results 仍經 remote downloader；
6. local URL／handle mismatch、missing path 與 invalid bundle 都明確失敗且不 self-download；
7. 三條 direct regressions 與相關 negative／hierarchy tests 通過；
8. PyMTLF full pytest、Ruff 與 `git diff --check` 通過；
9. flat isolated real-process E2E 通過；
10. hierarchy smoke manual-success regression 通過；
11. mandatory review、targeted remediation 與 fresh-read plan-conformance gate 完成；
12. implementation record 保存 exact commands、results、revisions、skips 與 remaining gaps；
13. working-tree diff 保持 unstaged、uncommitted並由使用者完成 IDE review；
14. review confirmation 後另行提出第二批 repository-separated commit proposal，取得對該精確
    proposal 的核准後才建立 commits。

第二批 commits 建立後，本 remediation 可標為 locally completed，但 Slice 8 與 HFL 第一版只能
進入 `Testbed Validation Pending`；required testbed matrix 通過且使用者確認 evidence 前不得標為
`Completed`。

---

## 13. 實作紀錄

### 13.1 Production 修改

PyMTLF baseline 為 `3628068`；本次 working tree 只修改：

- `src/py_mtlf/core/fl_server.py`
- `tests/test_fl_server.py`

`FLProcess` 現在同時保存 `current_global_artifact` 與 URL projection。flat preparation 直接保存
catalog artifact；每輪 aggregation 後把 `FLWorkspaceArtifact` 正規化成指向相同 path、digest 與
URL 的 `ArtifactMetadata`，下一輪直接載入該 metadata。

flat caller 會把 `publish_round_input()` 回傳的 local artifact 明確傳給 `_aggregate_round()`；該
方法的 `round_input_artifact` 已改為 required keyword-only argument，並移除 URL-only download
fallback。所有 production callers 均已重新確認會提供本機 artifact。

final validation evaluator 現在明確接收本機 candidate metadata，先驗證 URL projection 一致後
直接載入。participant `ROUND_LOCAL` 與 validation-result URLs 仍使用既有
`FLWorkspace.download()`，其 allowed-origin 與 artifact identity validation 未放寬。

`FLWorkspace.download()`、artifact schema、public SBI、NRF discovery、ADRF publication 與其他
repositories 的 production code 均未修改。`nwdaf-resources` 只使用既有 `39ced28` runner 驗證。

### 13.2 Test-first 與直接回歸證據

先修改兩輪 flat regression，再於 production change 前執行：

```bash
cd PyMTLF
.venv/bin/pytest -q \
  tests/test_fl_server.py::test_flat_server_publishes_round_input_with_server_owned_epochs
```

結果為預期失敗：flat `_run()` 尚未把 `round_input_artifact` 傳給 `_aggregate_round()`。最小
production 修正後，同一 node 通過。

final direct regressions：

```bash
cd PyMTLF
.venv/bin/pytest -q \
  tests/test_fl_server.py::test_flat_server_publishes_round_input_with_server_owned_epochs \
  tests/test_fl_server.py::test_server_aggregation_does_not_download_missing_owned_round_input \
  tests/test_fl_server.py::test_server_aggregation_rejects_owned_round_input_url_mismatch_without_download \
  tests/test_fl_server.py::test_final_validation_rejects_owned_candidate_url_mismatch_without_download \
  tests/test_fl_server.py::test_final_validation_uses_owned_candidate_and_downloads_only_peer_result \
  tests/test_fl_server.py::test_root_aggregation_weights_two_branch_results_by_effective_sample_count \
  tests/test_fl_server.py::test_aggregation_rejects_local_artifact_with_different_model_contract
```

結果：`7 passed in 1.60s`。這些 tests 直接證明第二輪 source、aggregation input 與 final candidate
均使用本機 handle；missing path 或 URL mismatch 不 fallback download；peer-owned round 與
validation artifacts 仍由 downloader 取得。

### 13.3 Repository 完整驗證

```bash
cd PyMTLF
.venv/bin/pytest -q -rs
.venv/bin/ruff check src tests
git diff --check
```

結果：

- pytest：`479 passed, 2 skipped, 46 warnings in 11.29s`
- skips：`tests/test_local_trainer.py:117` 與 `:129`，原因皆為 CUDA runtime unavailable
- Ruff：`All checks passed!`
- `git diff --check`：通過

### 13.4 真實程序回歸

flat isolated：

```bash
PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/distributed_fl/scripts/run.py
```

結果：通過完整 flat control plane、training、final validation、ADRF publication 與 model cutover；
ADRF model ID 為 `1787602712432`，A／B consumers 均採用。暫存根目錄為
`/tmp/nwdaf-distributed-fl-ndt2chdk`。

hierarchy shared-contract regression：

```bash
PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/scripts/run.py \
  --profile smoke --scenario manual-success
```

結果：通過；summary 為
`/tmp/nwdaf-hierarchical-fl-smoke-rwl1tfh8/summary.json`。

### 13.5 Review 與計畫對照結論

Mandatory initial review 重新檢查所有 `current_global_url`、`candidate_url`、
`_aggregate_round()`、`_evaluate_final_validation()` 與 `workspace.download()` call paths，沒有發現
第四條 Server-owned self-download。Root、Branch 與 flat production aggregation callers 都提供
owned round-input artifact；剩餘 downloader calls 只消費 peer notifications。

Initial review 發現 final evaluator 尚缺一條直接同時證明 owned candidate 不下載、peer validation
result 仍下載的測試證據；已新增 focused regression，targeted follow-up 與完整驗證均通過。沒有
新增 contract、schema、config 或 lifecycle owner。

完成條件 1–14 已關閉；使用者已確認 IDE review 與精確 commit proposal，第二批 commits 為
PyMTLF `e9aa223` 與 nwdaf-docs `f2d0175`。本 remediation 已 locally completed；testbed
validation 仍是整體 HFL 計畫的獨立 acceptance gate。
