# Slice 1：Hierarchy Bundle Contract 與 Artifact Primitives 詳細計畫

日期：2026-08-18

狀態：Implementation complete；已 commit，待 team review 與 push

上層計畫：

- [Hierarchical NWDAF Federated Learning Implementation Plan](./Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)

前置 contract：

- [Slice 0 Baseline Audit and Contract Freeze Detailed Plan](./Slice%200%20Baseline%20Audit%20and%20Contract%20Freeze%20Detailed%20Plan.md)

相關文件：

- [Hierarchical NWDAF Federated Learning Proposal Draft](../../proposals/nwdaf/hierarchical-federated-learning/proposal_draft.md)
- [NWDAF Development Policy](../../development_policy.md)

---

## 1. 目的與完成邊界

Slice 1 在 `PyMTLF/` 建立 hierarchical FL 第一版共用的 typed bundle contract 與
artifact primitives。完成後，後續 slices 可以直接產生、發布、下載及驗證 Root–Branch
assignment、Branch–Leaf assignment 與 Branch preparation result，而不再以 raw
dictionary 或未驗證的 `config.json` 推測 hierarchy semantics。

本 Slice 必須完成：

1. 擴充既有 FL artifact discriminated union，加入 hierarchy assignment 與 preparation
   result roles；
2. 建立 versioned hierarchy metadata、FedProx strategy、assignment 與 result typed
   models；
3. 實作 Root assignment publish、Branch assignment republish 與 preparation result
   publish 所需的 artifact primitives；
4. 補齊 hierarchy artifact download 的 URL／response／archive digest binding；
5. 建立 per-plan release 與既有 TTL cleanup 的明確 lifecycle；
6. 以 isolated positive／negative tests 固定 schema、integrity、recipient isolation 與
   content-addressed behavior。

本 Slice 不建立完整 Root、Branch 或 Leaf orchestration，不建立標準 Training resources，
也不開始 FedProx training。所有 primitives 必須可以在沒有 Go NWDAF、NRF、PyAnLF 或多個
running PyMTLF instances 的單元／整合測試中驗證。

---

## 2. 已確認且不得在本 Slice 重新決策的事項

Slice 1 直接繼承下列 Slice 0 contracts：

- hierarchy metadata 放在仍含有效模型的既有四檔案 bundle `config.json`；
- 不修改 Release 18 OpenAPI，也不增加 hierarchy-specific standard SBI fields；
- artifact bytes 由發布者 PyMTLF 提供，Go 不 proxy bytes；
- Branch 必須 download／validate／process Root bundle，再發布 Branch-owned Leaf bundle；
- 第一版 artifact roles 為 `HIERARCHY_ASSIGNMENT` 與
  `HIERARCHY_PREPARATION_RESULT`；
- hierarchy contract version 與 bundle schema version 第一版皆為 `1.0`；
- `planId` 是 vendor UUIDv4，不取代每層標準 `mlCorreId`；
- Root 對每個 Branch 發布 recipient-specific bundle，Branch 對每個 Leaf 重新發布
  recipient-specific bundle；
- 第一版 admission 只接受 `complete_required`；
- 第一版 strategy 只接受 FedProx／all participants／wait for all／sample-weighted
  aggregation；
- result 必須完整 partition assigned clients 為 prepared、failed 或 timed out；
- 第一版所有 NWDAF／PyMTLF 位於同一受控 vendor trust domain；publisher identity 只作
  logical process／topology validation；
- artifact signature、requester NF identity 與 artifact origin 強綁定、跨 trust domain
  authentication，以及 restart recovery 不在第一版；
- 第一版不新增 `expected_artifact_origin`、requester identity private header，亦不為此
  修改 OAuth／mTLS identity propagation。

若實作需要改變上述任一點，必須回到 canonical 主計畫與 Slice 0 review，不得在 code
review 中以 implementation convenience 默默改變 contract。

---

## 3. Repository、branch 與責任歸屬

### 3.1 受影響的 repository

唯一 production repository：

- `PyMTLF/`

實作 branch：

- `feat/r18-hierarchical-federated-learning`

撰寫本計畫時，已稽核的 PyMTLF distributed-FL baseline 是乾淨的
`feat/r18-federated-learning` branch，revision 為 `7e8ab7f`。若開始實作時該 workstream
尚未合併，hierarchy branch 必須從當時已 review 的 distributed-FL branch 分出，並在實作
checkpoint 記錄確切 base revision；不得從缺少可重用 FL workspace 與 artifact contract
的舊 `main` 分支。

本 Slice 不修改：

- `NWDAF/`；
- `PyAnLF/`；
- `nwdaf-resources/`；
- `resources/`；
- Release 18 OpenAPI YAML。

`nwdaf-docs/` 只保存本 detailed plan 與 review 後必要的 canonical clarification，不能與
`PyMTLF/` production commit 混在同一 repository commit。

### 3.2 責任邊界

| 項目 | Slice 1 owner |
| --- | --- |
| Hierarchy metadata schema | PyMTLF typed domain contract |
| Bundle file set 與 component digest validation | 既有 PyMTLF FL workspace |
| Assignment／result construction | 新增的 PyMTLF hierarchy artifact primitives |
| Artifact publication 與 serving | 既有 PyMTLF FL workspace／artifact API |
| Download origin 與 digest binding | PyMTLF FL workspace |
| Per-plan artifact release／TTL | PyMTLF FL workspace |
| Standard `mLModelUrl` transport | 延至 Slice 3／4 的 Go 與 PyMTLF orchestration |
| NRF capability validation | 延至 Slice 3／4 |
| Topology admission | 延至 Slice 4 |

---

## 4. 現有實作基線

### 4.1 可重用的 artifact contract

`src/py_mtlf/core/fl_artifacts.py` 目前提供：

- 使用 `extra="forbid"` 與 frozen instances 的 strict Pydantic models；
- 含 `ROUND_LOCAL`、`ROUND_GLOBAL`、`FINAL_MODEL` 的 `ArtifactRole`；
- 由 `artifact_role` discriminate 的外層 `FLArtifactContract`；
- exact component digest inventory validation；
- 既有 participant metadata 的 normalized NF instance IDs；
- fail-closed schema version 與 role validation。

Slice 1 必須擴充這條 validation path，不得建立另一個接受不同 hierarchy 值集合的 parser。

### 4.2 可重用的 workspace 與 serving path

`src/py_mtlf/core/fl_workspace.py` 目前提供：

- 對 `config.json`、`model.py`、`model.npy`、`scaler.pkl` 的 fixed archive entry validation；
- compressed／extracted／single-file／entry-count limits；
- 拒絕 unsafe path、duplicate entry、digest mismatch 與 incompatible model contract；
- 關閉 redirect 的 allowed-origin download；
- deterministic gzip／tar creation；
- content-addressed workspace paths 與 URLs；
- publisher-owned artifact resolution；
- startup TTL cleanup。

`src/py_mtlf/api/artifacts.py` 已提供 FL artifact serving，response 包含：

- `ETag: "sha256:<digest>"`；
- `X-Artifact-SHA256: <digest>`；
- immutable cache semantics；
- `nosniff` response protection。

### 4.3 Slice 1 必須補齊的差距

現有實作尚未提供：

- hierarchy artifact roles 或 typed hierarchy metadata；
- Branch／Leaf assignment 區分；
- typed first-version strategy；
- preparation result partition validation；
- recipient-specific hierarchy construction helpers；
- URL final digest、response header 與 downloaded archive digest 的三方比對；
- explicit per-plan artifact release；
- 證明 Branch output 是新發布的 Branch-owned artifact，而非 Root URL passthrough 的測試。

現有 `FLWorkspace.publish()` 假設使用 round-oriented `fl_metadata`，並為 training artifact
序列化 model state。Slice 1 必須將 role-aware projection 泛化，重用同一個安全 archive
writer，且不得弱化既有 round／final validation。

---

## 5. 目標 hierarchy contract

### 5.1 Bundle 檔案 contract

所有 hierarchy artifact 保持以下確切 archive entry set：

```text
config.json
model.py
model.npy
scaler.pkl
```

規則：

- 不增加第五個 manifest、signature、topology 或 result file；
- `config.json` 仍是唯一 manifest，並承載 hierarchy projection；
- `file_digests` 必須確切涵蓋三個非 manifest 檔案；
- `model.py`、`model.npy` 與 `scaler.pkl` 必須維持為可載入且彼此相容的 model artifact；
- hierarchy metadata 不取代既有 analytics event、model interoperability、runtime、model 或
  inference contracts；
- republish 可以產生不同的 archive digest，但載入後的 model state、model contract 與
  preprocessing contract 必須和已驗證的 input assignment 相同。

### 5.2 Artifact role 擴充

`ArtifactRole` 只新增：

```text
HIERARCHY_ASSIGNMENT
HIERARCHY_PREPARATION_RESULT
```

外層 artifact union 擴充為：

```text
ROUND_LOCAL
ROUND_GLOBAL
FINAL_MODEL
HIERARCHY_ASSIGNMENT
HIERARCHY_PREPARATION_RESULT
```

未知 role 仍然無效。沒有 hierarchy metadata 的既有 artifact 繼續依原本 role-specific
contract 驗證。

### 5.3 共用 hierarchy metadata

每個 hierarchy artifact 都包含：

```json
{
  "contract_version": "1.0",
  "message_type": "BRANCH_ASSIGNMENT",
  "plan_id": "11111111-1111-4111-8111-111111111111",
  "publisher_nf_instance_id": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
  "intended_recipient_nf_instance_id": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb"
}
```

共用驗證規則：

- `contract_version` 只接受 literal `1.0`；
- `plan_id` 必須是 normalized UUIDv4；
- publisher 與 recipient 都是 normalized UUID，且兩者不得相同；
- unknown fields 必須 fail closed；
- caller 提供的 expected publisher 與 containing NWDAF identity 必須符合解析結果；
- 完成 metadata parsing 不代表已建立標準 process identity 或完成 participant admission；
- `mlCorreId`、subscription ID、callback URI 與 standard `roundInd` 不得出現在
  `hierarchy_metadata`。

typed implementation 在既有外層 `artifact_role` union 之下，使用由 `message_type`
discriminate 的 nested union。完成驗證後，assignment 與 result data 不得再透過 untyped
dictionary lookup 存取。

### 5.4 第一版 strategy model

Branch 與 Leaf assignment 承載的 normalized strategy 如下：

```json
{
  "algorithm": {
    "name": "fedprox",
    "proximal_mu": 0.01
  },
  "participant_selection": "all",
  "waiting_policy": "all",
  "aggregation": "sample_weighted"
}
```

驗證規則：

- `algorithm.name` 只接受 `fedprox`；
- `proximal_mu` 必須是 finite 且嚴格大於零；
- participant selection 只接受 `all`；
- waiting policy 只接受 `all`；
- aggregation 只接受 `sample_weighted`；
- `fedavg`、zero／negative／NaN／infinite `proximal_mu`、`fixed_count`、
  `minimum_results`、generic parameter dictionaries 與 unknown fields 必須 fail closed；
- serialization 必須 canonical，且不得插入 per-node default；
- Branch 必須將已驗證的 normalized strategy 原樣放入每個 Leaf assignment，不提供 per-tier
  override。

Slice 1 將其實作為可重用的 domain contract。接入 Root main config 屬於 Slice 3；執行
proximal objective 屬於 Slice 5。

### 5.5 Branch assignment

Root 對每個 Branch 發布一份 bundle：

```json
{
  "bundle_schema_version": "1.0",
  "artifact_role": "HIERARCHY_ASSIGNMENT",
  "hierarchy_metadata": {
    "contract_version": "1.0",
    "message_type": "BRANCH_ASSIGNMENT",
    "plan_id": "11111111-1111-4111-8111-111111111111",
    "publisher_nf_instance_id": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
    "intended_recipient_nf_instance_id": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb",
    "assigned_leaf_nf_instance_ids": [
      "cccccccc-cccc-4ccc-8ccc-cccccccccccc",
      "dddddddd-dddd-4ddd-8ddd-dddddddddddd"
    ],
    "admission": {"mode": "complete_required"},
    "strategy": {
      "algorithm": {"name": "fedprox", "proximal_mu": 0.01},
      "participant_selection": "all",
      "waiting_policy": "all",
      "aggregation": "sample_weighted"
    }
  }
}
```

不變條件：

- assigned Leaf list 不得為空，且在 UUID normalization 後必須唯一並按 lexical order 排序；
- Root publisher、recipient Branch 與 assigned Leaves 必須互不相同；
- bundle 只能包含 recipient Branch 的 subtree；
- admission 只接受 `complete_required`；
- strategy 必須是已驗證的 normalized strategy object；
- publisher helper 要求明確提供 Root identity、Branch identity、assigned Leaves 與
  strategy；它不讀取 static topology，也不執行 NRF discovery。

### 5.6 Leaf assignment

Branch 對每個 assigned Leaf 發布一份新 bundle：

```json
{
  "bundle_schema_version": "1.0",
  "artifact_role": "HIERARCHY_ASSIGNMENT",
  "hierarchy_metadata": {
    "contract_version": "1.0",
    "message_type": "LEAF_ASSIGNMENT",
    "plan_id": "11111111-1111-4111-8111-111111111111",
    "publisher_nf_instance_id": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb",
    "intended_recipient_nf_instance_id": "cccccccc-cccc-4ccc-8ccc-cccccccccccc",
    "parent_branch_nf_instance_id": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb",
    "strategy": {
      "algorithm": {"name": "fedprox", "proximal_mu": 0.01},
      "participant_selection": "all",
      "waiting_policy": "all",
      "aggregation": "sample_weighted"
    }
  }
}
```

不變條件：

- publisher 必須等於 `parent_branch_nf_instance_id`；
- recipient Leaf 必須和 Branch 不同；
- recipient 必須存在於傳入 republish primitive、且已驗證的 parent Branch assignment；
- `plan_id` 與 strategy 必須和 parent Branch assignment 完全一致；
- Leaf bundle 不包含 sibling Leaf list，也不包含 Root／其他 Branch topology；
- output URL 使用 Branch `public_base_url` 與新發布的 Branch workspace path；
- typed republish API 不可能回傳 parent Root URL；
- output model state、model contract 與 preprocessing contract 必須符合 parent assignment。

### 5.7 Preparation result

assigned clients 都有 terminal preparation outcome 後，Branch 發布一份 result bundle：

```json
{
  "bundle_schema_version": "1.0",
  "artifact_role": "HIERARCHY_PREPARATION_RESULT",
  "hierarchy_metadata": {
    "contract_version": "1.0",
    "message_type": "PREPARATION_RESULT",
    "plan_id": "11111111-1111-4111-8111-111111111111",
    "publisher_nf_instance_id": "bbbbbbbb-bbbb-4bbb-8bbb-bbbbbbbbbbbb",
    "intended_recipient_nf_instance_id": "aaaaaaaa-aaaa-4aaa-8aaa-aaaaaaaaaaaa",
    "outcome": "FAILED",
    "assigned_client_nf_instance_ids": [
      "cccccccc-cccc-4ccc-8ccc-cccccccccccc",
      "dddddddd-dddd-4ddd-8ddd-dddddddddddd"
    ],
    "prepared_clients": [
      {"nf_instance_id": "cccccccc-cccc-4ccc-8ccc-cccccccccccc"}
    ],
    "failed_clients": [
      {
        "nf_instance_id": "dddddddd-dddd-4ddd-8ddd-dddddddddddd",
        "cause": "REQUIREMENTS_NOT_MET"
      }
    ],
    "timed_out_client_nf_instance_ids": []
  }
}
```

允許的 outcome：

- `READY`；
- `FAILED`。

第一版允許的 failure cause：

- `DISCOVERY_FAILED`；
- `CAPABILITY_MISMATCH`；
- `INVALID_ASSIGNMENT`；
- `INVALID_BUNDLE`；
- `REQUIREMENTS_NOT_MET`；
- `NOT_AVAILABLE_ML_TRAIN`；
- `INTERNAL_ERROR`。

Result 不變條件：

- assigned、prepared、failed 與 timed-out lists 必須 canonical sort 且不得重複；
- prepared、failed 與 timed-out sets 必須 pairwise disjoint；
- 三者 union 必須精確等於 assigned client set；
- 每個 failed client 必須恰好具有一個 bounded cause；
- `READY` 要求所有 assigned clients 都已 prepared，且兩個 failure lists 都為空；
- 其他有效 partition 必須使用 `FAILED`；
- 禁止任意 error text、traceback、HTTP body 或 nested exception details；
- publisher Branch、recipient Root、`plan_id` 與 assigned set 必須符合傳入 result builder、
  且已驗證的 parent Branch assignment；
- output model state 與 model／preprocessing contracts 必須符合 parent assignment。

builder 接受已完成分類的 outcome values，不負責 discovery、等待 participants、判定 timeout
或套用 Root admission policy。

---

## 6. Artifact primitives

### 6.1 Typed validation result

Archive validation 必須向 caller 回傳 typed role-specific contract，而不只是 success／
failure。目標結果包含：

- 已下載或發布的 `ArtifactMetadata`；
- 已驗證的 manifest projection；
- typed `FLArtifactContract` instance；
- caller 要求 model loading 時的 loaded model bundle。

只需要 `ArtifactMetadata` 的既有 caller 可以保留 compatibility wrapper，但所有新的
hierarchy code 都必須使用 typed result。Publish 與 download paths 必須共用唯一的
authoritative projection builder。

### 6.2 Publish primitives

提供明確的 role-specific operations，不公開 generic metadata dictionary：

- 發布 Branch assignment；
- 從已驗證的 Branch assignment 重新發布 Leaf assignment；
- 從已驗證的 Branch assignment 發布 preparation result。

每個 operation 都必須：

1. 在 filesystem write 前驗證 typed inputs；
2. 保留有效的四檔案 model bundle；
3. 在內部建立 hierarchy metadata；
4. 透過共用 artifact adapter 驗證完整 manifest；
5. 寫出 deterministic archive bytes；
6. 計算完整 archive SHA-256；
7. 將檔案保存於
   `<planId>/<counterpartyNfInstanceId>/0/<artifactRole>/<digest>.tar.gz`，並透過既有 route
   提供 final URL segment 為 `<digest>` 的 URL；
8. 回傳 immutable artifact metadata 與 typed contract。

`round_indicator = 0` path component 只是 private preparation storage coordinate，永遠不投影為
standard `roundInd`。

將相同完整內容發布到相同 coordinate 必須 idempotent。不同內容會產生不同 digest URL，
不得取代既有 URL 的 bytes。

### 6.3 Download integrity binding

對 hierarchy download 而言，只有 HTTP 成功與 archive validation 並不足夠。Primitive
必須：

1. 依 configured allowed origins 驗證 URL origin；
2. 拒絕 credentials、fragment、unsupported scheme 與 redirect；
3. 要求 final URL path segment 恰好是一個 lowercase SHA-256 digest；
4. 要求恰好一個有效的 `X-Artifact-SHA256` response header；
5. 在既有 size limits 內串流並計算 compressed response hash；
6. 要求 URL digest = response header digest = downloaded archive digest；
7. 驗證 archive safety、entry set 與 component digests；
8. 驗證 role-specific typed contract；
9. 比對 expected role、message type、publisher、recipient，以及已知時的 `plan_id`；
10. 所有檢查通過後，才以 atomic operation 暴露 local file。

上述 publisher comparison 是同一受控系統內的 logical contract validation。標準 inbound
`NwdafMLModelTrainSubsc` body 不提供 requester NF instance ID；因此第一版不宣稱此欄位
或 allowed-origin validation 能證明實際 HTTP caller identity。Caller 已有 process route
或 orchestration expectation 時仍應提供 expected publisher，但 Slice 1 不增加獨立的
artifact-origin identity binding。

對 inbound preparation request 而言，receiver 在完成 bundle 驗證前並不知道 `plan_id`。
因此 hierarchy downloader 先串流寫入 workspace-owned temporary staging file，解析並驗證
identity contract，再將檔案 atomically commit 到已驗證的 `plan_id` directory。它不得要求
caller 從 URL 猜測 plan ID。Caller 已知 expected plan ID 時，同一 primitive 必須在 commit
前完成比對。驗證失敗或衝突時刪除 staging bytes，不留下 caller 命名的 provisional
directory。

缺少／malformed digest header、uppercase 或 noncanonical URL digest、digest 不一致、
unexpected peer identity 或錯誤 recipient 都必須 fail closed，並刪除 temporary file。

本 Slice 不修改 general artifact repository download contract。這項 binding 適用於
`/internal/v1/fl-artifacts/...` hierarchy artifacts；只有在另有 regression evidence 後，
才可將其重用於既有 round artifacts。

### 6.4 Branch republish contract

republish primitive 接受：

- 一份已驗證的 `BRANCH_ASSIGNMENT` artifact；
- containing Branch identity；
- 從該 assignment 選出的 intended Leaf identity。

它產生新的 `LEAF_ASSIGNMENT` artifact，不接受 raw Root URL 或 caller-provided Leaf
strategy。這個 API shape 保證：

- Root assignment 已先完成下載與驗證；
- recipient membership 來自已驗證的 parent assignment；
- Branch identity 成為 publisher 與 parent；
- strategy 與 `plan_id` 不會 drift；
- sibling 資訊會被移除；
- Branch 擁有輸出 URL 與 lifecycle。

NRF eligibility check 由 Slice 4 在呼叫此 primitive 前完成，不嵌入 artifact code。

### 6.5 Retention 與 release

新增 idempotent exact-plan release primitive：

```text
release_plan(plan_id)
```

規則：

- target 只能是 normalized UUIDv4 plan directory；
- release 只刪除該確切 plan directory 內的 artifacts 與 downloads；
- download staging files 不屬於 retained plan artifacts，每次失敗都必須刪除；crash 後
  abandoned staging files 可由 startup cleanup 清除；
- directory 不存在仍視為成功；
- 不接受 caller-provided arbitrary path、glob 或 recursive workspace root target；
- normal orchestration 只在 plan terminal cleanup 開始時呼叫 release；
- callback `204 No Content` 本身不會釋放 result bundle；
- startup TTL cleanup 繼續負責 crash leftovers；
- restart 後不恢復任何 artifact 或 active process。

Slice 1 直接測試此 primitive；由 Slice 4／6 決定 orchestration 的呼叫時機。

### 6.6 Error boundary

Hierarchy contract 與 artifact failure 使用 bounded internal exception categories，讓後續
orchestration 不必比對 log text 即可完成 mapping：

- schema／unsupported version；
- identity／recipient mismatch；
- strategy／admission mismatch；
- partition invariant violation；
- origin／transport digest mismatch；
- archive／component integrity failure；
- model／preprocessing compatibility failure；
- artifact missing／expired；
- workspace I/O failure。

Exception 可以保留安全的 operator-facing message，但 bundle metadata 不得包含 raw
exception details。SBI ProblemDetails／termination mapping 延至 Slice 4。

---

## 7. PyMTLF 預計變更範圍

### 7.1 新增 domain contract module

新增 `src/py_mtlf/core/fl_hierarchy.py`，負責 pure typed hierarchy contracts：

- contract／message／outcome／failure enums；
- normalized UUIDv4 與 NF instance ID validation；
- admission model；
- FedProx strategy model；
- Branch／Leaf assignment metadata；
- preparation result metadata 與 partition validators；
- 回傳 immutable validated values 的小型 construction functions。

此 module 不得 import FastAPI、HTTP client、NRF discovery、workspace path 或 process
orchestrator。

### 7.2 擴充既有 artifact contract

更新 `src/py_mtlf/core/fl_artifacts.py`：

- 擴充 `ArtifactRole`；
- 新增 hierarchy assignment／result outer artifact models；
- 將 nested hierarchy metadata union 接入 `FLArtifactContract`；
- 保留唯一的 `validate_fl_artifact()` entry point；
- 讓既有 round／final types 與 tests 維持 source-compatible。

### 7.3 Workspace primitives

更新 `src/py_mtlf/core/fl_workspace.py`：

- 依 role 集中處理 manifest projection extraction；
- 為新的 hierarchy operations 回傳 typed validation results；
- 新增 URL／header／archive digest binding；
- inbound hierarchy download 維持 staging，直到 validated manifest 提供 `plan_id`，再 commit
  至該確切 plan directory；
- 新增不使用 raw metadata dictionary 的 role-specific hierarchy publish／republish 支援；
- 新增 exact `release_plan()`；
- 保留既有 archive safety、deterministic writer 與 round artifact behavior。

若 role-specific construction 使 `FLWorkspace` 責任過寬，可用小型
`HierarchyArtifactService` 組合 `FLWorkspace`；它仍須使用同一個 validator 與 writer，
不得建立平行的 storage layout。

### 7.4 Artifact serving API

`src/py_mtlf/api/artifacts.py` 不應需要新 route。既有 FL artifact route 維持唯一 serving
path；測試必須確認新的 hierarchy roles 能正確 resolve，並繼續輸出宣告的 digest header。

### 7.5 Tests 與 fixtures

新增或擴充：

- `tests/test_fl_hierarchy.py`；
- `tests/test_fl_artifacts.py`；
- `tests/test_fl_workspace.py`；
- `tests/test_health_and_artifact_api.py`；
- `tests/conftest.py` 只放置可重用的 valid bundle builders。

不得只在 test fixture helper 中實作 production schema behavior。

---

## 8. 實作順序與 checkpoints

### Checkpoint 1：Typed hierarchy domain contract

實作 models 與 focused validation tests：

- UUID normalization 與 UUIDv4 enforcement；
- strict FedProx strategy；
- Branch 與 Leaf assignments；
- result partition 與 bounded causes；
- unknown field／version／enum rejection。

Checkpoint 驗收：所有 hierarchy model tests 在不使用 filesystem 或 network fixture 的情況
下通過。

### Checkpoint 2：Artifact union 與 archive validation

擴充 outer roles 與 common projection handling：

- hierarchy artifacts 透過 `validate_fl_artifact()` 驗證；
- archive validation 回傳 typed role；
- 既有 round／global／final artifacts 維持相同行為；
- invalid role／metadata combination 必須 fail closed。

Checkpoint 驗收：focused artifact contract 與既有 FL artifact regression tests 通過。

### Checkpoint 3：Publish 與 republish primitives

實作：

- Root Branch assignment publication；
- Branch Leaf assignment republishing；
- Branch preparation result publication；
- deterministic content addressing 與 recipient isolation。

Checkpoint 驗收：不建立 orchestrator 的情況下，in-process Root → Branch → Leaf 與
Branch → Root artifact round trips 通過。

### Checkpoint 4：Download binding 與 serving

實作並驗證：

- allowed origin；
- URL/header/body digest binding；
- atomic local exposure；
- hierarchy role／publisher／recipient expectations；
- 既有 serving headers 與 missing artifact behavior。

Checkpoint 驗收：tampered URL、header、body、manifest 與 peer identity fixtures 全部 fail
closed，且不留下 partial file。

### Checkpoint 5：Release、TTL 與完整 regression

實作 exact-plan release，並執行：

- per-plan isolation／idempotency tests；
- startup TTL cleanup tests；
- focused hierarchy suite；
- 完整 PyMTLF test suite；
- 針對 changed Python files 執行 Ruff check。

Checkpoint 驗收：workspace cleanup 無法刪除 sibling plans 或 workspace root，且所有既有
non-hierarchical tests 通過。

所有 checkpoints 都在同一個 Slice 1 implementation branch 上進行。Commit boundary 可以
依 checkpoints 切分，但完整 acceptance matrix 通過前，Slice 1 不算完成。

---

## 9. 驗收測試矩陣

### 9.1 正向 contracts

- valid Branch assignment 會 normalize identities 與 strategy；
- valid Leaf assignment 不包含 sibling topology；
- valid all-prepared result 產生 `READY`；
- valid failed／timed-out partition 產生 `FAILED`；
- Root 為不同 recipients 發布各自的 recipient-specific Branch bundle；
- Branch 從一份已驗證的 parent assignment，為不同 Leaves 重新發布各自的 bundle；
- result bundle 維持為有效且可載入的四檔案 model bundle；
- 重複發布相同內容必須 idempotent；
- serving response digest header 必須符合 URL 與實際 bytes；
- per-plan release 必須 idempotent；
- TTL 清除 expired plan directories，並保留未過期目錄。

### 9.2 Schema 拒絕案例

- 不支援的 bundle 或 hierarchy contract version；
- 未知 artifact role、message type、strategy、admission、outcome 或 cause；
- 缺少必要 hierarchy field 或出現 extra field；
- 非 UUID 或非 v4 的 `plan_id`；
- duplicate／unsorted NF instance lists；
- publisher 等於 recipient；
- Root／Branch／Leaf identity collision；
- 空的 Branch Leaf assignment；
- Leaf parent 與 publisher 不同；
- zero、negative、NaN 或 infinite `proximal_mu`；
- 將不支援的 future strategy option 當成 active config。

### 9.3 Result partition 拒絕案例

- assigned client 未出現在任何 outcome；
- 出現 unassigned client；
- 同一 client 出現在多個 outcome lists；
- 同一 list 內出現 duplicate client；
- `READY` 同時包含 failure 或 timeout；
- 所有 clients 都 prepared 卻使用 `FAILED`；
- 缺少 failure cause 或使用未知 cause；
- 出現 arbitrary exception text field。

### 9.4 Artifact 與 transport 拒絕案例

- unsafe／extra／missing／duplicate archive entry；
- component size 或 digest 違規；
- 無效的 model 或 preprocessing contract；
- 不允許的 URL origin；
- redirect response；
- missing／duplicate／invalid `X-Artifact-SHA256`；
- URL digest 與 header 不同；
- body digest 與 URL／header 不同；
- unexpected artifact role 或 message type；
- 錯誤的 publisher、recipient 或 expected `plan_id`；
- Leaf republish request 指定 parent assignment 中不存在的 Leaf；
- 嘗試直接 passthrough Root URL；
- expired／missing artifact。

### 9.5 Regression

- 既有 `ROUND_LOCAL` training 與 accuracy-check artifacts；
- 既有 `ROUND_GLOBAL` sample accounting；
- 既有 `FINAL_MODEL` validation；
- 既有 FL Client／Server workspace download 與 publication；
- 既有 public model artifact repository；
- health 與 artifact serving APIs；
- non-HFL bundle loader behavior。

---

## 10. 明確延後的行為

Slice 1 不實作：

- Root static topology loading 或 NRF validation；
- runtime capability／role migration；
- active plan registry 或 upper／lower process mapping；
- degradation 或 private API initiation；
- standard Training subscription create／update／notification／delete；
- preparation scheduling、waiting、timeout classification 或 Root admission；
- `statusReport` notification contract correction；
- FedProx objective、local training 或 hierarchical aggregation；
- sample count propagation；
- final model Provision 或 PyAnLF activation；
- artifact signature、encryption、cross-domain authentication 或 transparent proxy；
- arbitrary-depth hierarchy、partial admission、dynamic selection 或 automatic retry；
- restart 後 active plans 的 persistence 或 recovery。

Schema extension point 只能作為 typed boundary 保留。Deferred enum values 不得被接受成已
實作功能。

---

## 11. 風險與控制

### 11.1 弱化既有 artifact validation

風險：projection logic 泛化後，意外接受不完整的 round／final artifacts。

控制：只使用一個 strict role-discriminated adapter，保留既有 regression fixtures，且不提供
generic `dict[str, Any]` fallback。

### 11.2 將 publisher identity 誤當成 cryptographic proof

風險：manifest publisher field 被誤認為 signed attestation。

控制：明確記錄第一版 publisher binding 只代表 logical same-vendor trust。保留 expected
peer（caller 已知時）、allowed origin 與 digest binding 作一致性檢查，但不新增
`expected_artifact_origin`、requester identity header 或 OAuth／mTLS propagation，也不將
這些檢查描述為 authenticated caller proof。嚴格 requester-to-origin binding 延後到跨
trust domain 的 security hardening。

### 11.3 Branch passthrough 洩漏 topology 或增加 Root 負載

風險：Branch 直接把 Root URL 或完整 subtree manifest 轉交 Leaves。

控制：republish API 要求已驗證的 parent assignment，並產生新的 recipient-specific
Branch-owned artifact；測試拒絕不存在的 recipient，並驗證 Branch URL origin。

### 11.4 Cleanup 刪除使用中或無關 artifacts

風險：過廣的 filesystem deletion 影響 sibling plans 或 non-FL artifacts。

控制：只允許 UUIDv4 exact directory release、idempotent missing behavior、sibling isolation
tests，且不接受 arbitrary path parameter。

### 11.5 過早建立 orchestration

風險：artifact helpers 開始擁有 discovery、participant state 或 admission decision。

控制：builder 只接受已驗證的 identities 與已分類的 results；process ownership 維持延後至
Slice 2–4。

---

## 12. Review checklist

### 12.1 Contract

- [ ] 既有四檔案 bundle contract 維持不變。
- [ ] Hierarchy metadata 必須 strict、versioned 且 role-discriminated。
- [ ] `plan_id` 是 UUIDv4，且不同於 standard process identifiers。
- [ ] Branch 與 Leaf assignments 都是 recipient-specific。
- [ ] 第一版 strategy 只接受已確認的值。
- [ ] Preparation result 精確 partition assigned clients。
- [ ] Failure causes 是 bounded values，且不包含 raw error details。

### 12.2 Artifact 行為

- [ ] New roles 重用既有 workspace 與 serving route。
- [ ] Branch republish 無法把 Root URL 當作 Leaf artifact 回傳或暴露。
- [ ] URL、header 與 body archive digests 完成 binding。
- [ ] Model、preprocessing 與 weights 經過 republish／result publication 後維持相容。
- [ ] Artifact URLs 在 retention 期間 immutable。
- [ ] `release_plan()` 只以一個已驗證的 plan directory 為 target。
- [ ] TTL 維持 crash-leftover cleanup mechanism。

### 12.3 Scope

- [ ] 不修改 Go、PyAnLF、`nwdaf-resources` 或 OpenAPI。
- [ ] 不加入 NRF、topology planner、Training SBI 或 orchestration logic。
- [ ] 不執行 FedProx 或 aggregation。
- [ ] 不接受 future strategy value。
- [ ] 既有 non-HFL FL tests 維持通過。

---

## 13. 完成條件

Slice 1 只有在下列條件全部成立時才算完成：

1. hierarchy strategy、assignment 與 result models 都是 immutable 且 fail closed；
2. hierarchy roles 與既有 FL roles 透過同一個 outer artifact adapter 驗證；
3. Root Branch assignment、Branch Leaf republish 與 Branch result publication primitives 能產生
   有效且可載入的四檔案 bundles；
4. hierarchy download 完成 allowed origin、URL digest、response header 與 archive bytes
   binding；
5. publisher、recipient、message type、role 與已知 `plan_id` expectations 都會被強制檢查；
6. result partition invariants 與 bounded failures 具有完整測試；
7. exact-plan release 與 TTL behavior 已測試，且不會影響 sibling plans；
8. focused hierarchy、artifact、workspace 與 serving tests 通過；
9. 完整 PyMTLF regression 與 changed-file lint 通過；
10. implementation 已 commit 在 `PyMTLF/feat/r18-hierarchical-federated-learning`，且沒有
    unrelated repository changes。

上述條件通過後，Slice 2 可以在 typed contracts 上建立 capability-oriented runtime 與
process-scoped role state；Slice 3 可以將 Root topology／config 接入 Branch assignment
publication；Slice 4 可以呼叫 Branch republish／result primitives，並修正 preparation
notification sender／validator／stage-aware receiver contract。

---

## 14. Implementation checkpoint

### 14.1 Branch 與 baseline

- repository：`PyMTLF/`；
- branch：`feat/r18-hierarchical-federated-learning`；
- base branch：`feat/r18-federated-learning`；
- base revision：`7e8ab7f23bf5d6398eb1cd5f053dd8bda9439a87`；
- implementation commit：`fa352d3`（`feat(mtlf): add hierarchical FL artifact primitives`）；
- implementation state：已 commit，尚未 push。

### 14.2 實際變更

新增：

- `src/py_mtlf/core/fl_hierarchy.py`；
- `src/py_mtlf/core/fl_hierarchy_artifacts.py`；
- `tests/test_fl_hierarchy.py`；
- `tests/test_fl_hierarchy_artifacts.py`。

擴充：

- `src/py_mtlf/core/fl_artifacts.py`；
- `src/py_mtlf/core/fl_workspace.py`；
- `tests/test_fl_artifacts.py`；
- `tests/test_health_and_artifact_api.py`。

完成內容：

- FedProx-only typed strategy、UUIDv4 `plan_id`、Branch／Leaf assignment 與 preparation
  result partition contracts；
- `HIERARCHY_ASSIGNMENT` 與 `HIERARCHY_PREPARATION_RESULT` artifact roles；
- 共用 complete-manifest projection 與 role-incompatible field rejection；
- Root Branch assignment publication；
- Branch-owned Leaf assignment republish，typed API 不接受 raw Root URL；
- Branch preparation result publication；
- hierarchy download staging、allowed-origin、URL／header／body digest binding、role／message／
  publisher／recipient／plan validation；
- bounded hierarchy transport／integrity／contract／identity error categories；
- exact UUIDv4 `release_plan()` 與 stale staging／plan TTL cleanup；
- 既有 FL artifact serving route 的 hierarchy role coverage。

### 14.3 Verification evidence

執行：

```text
.venv/bin/pytest -q
.venv/bin/ruff check .
```

結果：

- Pytest：`241 passed, 2 skipped`；
- Ruff：`All checks passed`；
- skipped tests 為既有 suite 的環境條件 skip，沒有新增 failure；
- warnings 為既有 Starlette/httpx deprecation 與 joblib／NumPy deprecation，沒有 Slice 1
  contract failure。

### 14.4 Handoff

Team review 應優先確認 typed schema、Branch republish ownership、digest binding 與
`release_plan()` scope。Review 通過後，Slice 2 可以直接使用這些
immutable contracts 與 artifact primitives 建立 capability-oriented runtime 和
process-scoped role state。
