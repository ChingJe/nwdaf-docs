# Slice 4A — Digest Simplification and Contract Cleanup Detailed Plan

日期：2026-09-05

狀態：Approved for Implementation／production implementation尚未開始

相關文件：

- [Protocol Extension Implementation Plan](../Hierarchical%20NWDAF%20FL%20Protocol%20Extension%20Implementation%20Plan.md)
- [Protocol Extension Implementation Slice Map](../Protocol%20Extension%20Implementation%20Slice%20Map.md)
- [Protocol Implementation Current-State Inventory](../Protocol%20Implementation%20Current-State%20Inventory.md)
- [Slice 4 Detailed Plan](./Slice%204%20Controlled%20Local%20Training%20Workload%20Detailed%20Plan.md)
- [Slice 5 Detailed Plan](./Slice%205%20Protocol-driven%20Hierarchy%20Integration%20Detailed%20Plan.md)
- [NWDAF Development Policy](../../../../development_policy.md)

---

## 1. Slice 結果

本 slice 在controlled workload與protocol-driven E2E integration前，移除PyMTLF中除「完整artifact
內容識別」以外的 hash／digest contract。完成後只保留一個 SHA-256 用途：發布者對
完整壓縮 artifact bytes 計算 repository key，下載者以 URL 中的同一 key 驗證實際
下載 bytes。

以下不再作為 runtime contract：

- bundle component `file_digests`；
- model／preprocessing contract digest及weights digest；
- scope、observation、training tensor與validation tensor digest；
- ML Model Training Notify body digest；
- static topology digest與以digest產生的scope identity；
- training-data callback、request、profile、collection與record的content digest。

這是既有跨flat／distributed／hierarchical FL與training-data collection的contract
cleanup，不是Slice 4 dataset功能，也不改變Slice 1／2已完成工作的歷史證據。Slice 4
與Slice 5都必須建立在本slice完成後的簡化contract上，不得重新引入上述digest。

---

## 2. 保留的唯一 hash boundary

### 2.1 Canonical artifact identity

保留下列單一機制：

```text
SHA-256(完整壓縮artifact bytes)
    -> ArtifactMetadata.key
    -> /internal/v1/artifacts/{key}
```

規則如下：

- repository publish只對完整壓縮檔計算一次content-addressed key；
- repository儲存路徑與artifact URL以該key識別內容；
- downloader由URL取得expected key，串流下載時同時計算實際bytes，兩者不符即拒絕；
- `artifactDigest`、`candidateDigest`、`resultDigest`等欄位若承載的是上述repository key，
  仍可保留，但不得再改指向weights、component或semantic-contract digest；
- `X-Artifact-SHA256`不再作為第二個必須一致的來源；移除producer、consumer與測試對此
  duplicate header的要求；
- 不在config、sidecar、model metadata或operator input再保存一份expected artifact hash。

Archive origin、size、path traversal、entry set與safe extraction仍是必要的transport／parser
安全檢查；它們不是hash機制，不因本slice移除。

---

## 3. 既有機制與處理方式

| 現有機制 | 現有用途 | Slice 4A處理 |
| --- | --- | --- |
| `file_digests` | 逐一驗證bundle components | Builder停止產生，loader停止要求與比對；component set仍依workload profile驗證 |
| model／preprocessing digest | 判斷code、scaler與contract是否相同 | 改用artifact key、typed workload profile、model identity及實際loader compatibility checks |
| base／input／candidate／final weights digest | 建立model lineage及aggregation前後一致性 | 從metadata與validator移除；改用process、round、artifact identity及state-dict key／shape compatibility |
| scope／dataset／tensor digest | 保存training evidence及scope比較 | 移除digest欄位；保留明確scope資料、sample count與實際metric evidence |
| FL Notify body digest | 重複callback與conflict判斷 | 改為per-resource stage state；同一stage第一個terminal outcome生效，後續terminal retry不再重新套用 |
| topology digest | static topology與scope fingerprint | 移除；使用明確topology內容、`mlCorreId`、resource revision與runtime assignment state |
| collection request／profile digest | persisted state重啟時比較config | 持久化並比較typed request／profile representation，不建立摘要欄位 |
| data callback／record content digest | inbox檔名與內容去重 | inbox改用operation UUID；優先使用來源提供的native identity，沒有native identity時不再宣稱content-level deduplication |

`upstream-assigned`／`locally-discovered` provenance不是cryptographic digest，不在本slice
移除。一般Git commit hash與第三方cache key也不屬於production runtime contract。

---

## 4. 替代語意

### 4.1 Model與artifact compatibility

移除digest後仍需保留下列必要判斷：

- artifact URL key和實際下載bytes一致；
- bundle符合workload profile要求的檔案集合與typed metadata；
- request／artifact的`mlCorreId`、`roundInd`、artifact role與participant identity符合目前
  operation；
- model identity、input shape、output shape與class／feature contract相容；
- aggregation前確認所有state dict具有相同parameter keys、shape與dtype；
- sample count保持positive並與aggregation weighting input一致。

不得以新的version fingerprint、config hash或tensor checksum取代被移除欄位。

### 4.2 FL Notify idempotency

FL Server以既有correlation與stage state處理callback：

1. `notifCorreId`定位participant resource；
2. `mlCorreId`與`roundInd`確認procedure及round；
3. preparation、round、delay與terminal state確認callback是否仍可接受；
4. 每個participant／stage的第一個terminal outcome生效；
5. 相同stage後續terminal callback視為retry並不再套用；已進入其他stage的callback依
   現有late／stale規則拒絕。

這個slice不再比較callback JSON內容，也不嘗試判斷兩個terminal callback的bytes是否
相同。Delay extension仍由明確的expected completion time與stage state處理。

### 4.3 Training-data collection與record identity

- durable inbox檔名使用新產生的UUID，不使用callback內容摘要；
- callback仍先依standard correlation找到active collection resources；
- 若資料來源提供native record ID，繼續使用該ID；
- 沒有native ID時，使用local UUID識別該次已接受delivery；若peer因未收到HTTP response
  而重送相同payload，系統不再宣稱可以跨delivery辨識內容重複；
- persisted request與profile直接保存typed representation，restart時作typed equality
  comparison；不保存`requestDigest`或`profileDigest`；
- 不以另一套canonical JSON、measurement composite fingerprint或checksum補回content
  deduplication。

### 4.4 Scope與topology identity

- `TrainingScopeDescriptor`保留實際event subscription、target與time windows，不再產生
  `scopeDigest`；
- local collection／training job以既有resource／request ID及明確scope key關聯；
- static flat participant scope使用可讀、typed的local assignment identity，不對整份
  topology或scope JSON計hash；
- protocol hierarchy使用`mlCorreId`、subscription resource、candidate revision與
  explicit subtree，不建立topology fingerprint。

---

## 5. Compatibility與migration boundary

- 新產生的bundle與round／result metadata不得包含被移除的digest欄位。
- 既有durable model bundle若包含`file_digests`，migration reader可在明確的legacy
  compatibility path讀取但不驗證；新writer不得再輸出。該legacy欄位接受路徑由
  Slice 6隨舊bundle migration一併移除。
- Temporary FL round artifacts不提供跨版本resume；active process在deployment升級時
  依既有restart boundary失效，不為舊digest metadata新增轉換器。
- Training-data persisted ledger升級為不含request／profile digests的representation；
  若舊record已保存完整typed request／profile，loader由這些欄位完成migration，不因
  缺少或不符舊digest拒絕啟動。
- External 3GPP SBI schema沒有新增或移除hash欄位；本slice處理的是project-private
  artifact與PyMTLF local state。

---

## 6. Repository與檔案範圍

### 6.1 `PyMTLF/`

主要production owners：

- `core/artifacts.py`、`api/artifacts.py`：保留whole-artifact key，移除component digest
  與duplicate response header；
- `core/bundle_builder.py`、`core/fl_workspace.py`、`core/fl_artifacts.py`：簡化artifact
  metadata、builder、loader與validation；
- `core/fl_client.py`、`core/fl_server.py`、`core/fl_root.py`、
  `core/fl_hierarchy_artifacts.py`：改用explicit process／round／artifact／model state；
- `core/training_scope.py`、`core/training_data.py`、`core/trainer.py`、
  `core/training_jobs.py`：移除scope及dataset evidence digests；
- `core/training_data_collection.py`、`core/dataset.py`：移除callback、request、profile、
  collection與record content hashes；
- `core/fl_topology.py`、`core/fl_flat.py`、`core/fl_orchestration.py`：移除topology與static
  scope hashes。

實作前以`rg`重新盤點所有production `hashlib`／SHA／digest usage；只有完整artifact
repository key及其URL/body check可留在allowlist。欄位名稱雖含`digest`但明確承載完整
artifact key者需逐一證明，不可僅依名稱批次刪除。

### 6.2 其他repositories

- `NWDAF/`：目前沒有production SHA／digest validation；預設read-only，只有實際private
  artifact header contract需要同步移除時才納入。
- `nwdaf-resources/`：只有fixtures／scenario仍產生或驗證已移除欄位時修改。
- `nwdaf-docs/`：更新plan、review evidence與operator-facing artifact說明。

本slice不修改3GPP OpenAPI corpus、candidate schema、NRF或ADRF。

---

## 7. Test design

測試應證明簡化後的production behavior，而不是替被移除機制保留hash mismatch cases。

### 7.1 必要positive cases

- whole-artifact publish取得content-addressed key，下載bytes與URL key一致時成功；
- bundle沒有`file_digests`仍可依known workload profile安全載入；
- local update與aggregate在無weights／contract／dataset digests時完成真實訓練與聚合；
- incompatible state-dict key／shape仍在aggregation boundary被明確拒絕；
- 同一participant／round重送terminal Notify不造成第二次aggregation或state mutation；
- training-data callback使用UUID inbox並可完成durable processing；
- restart可載入不含request／profile digest的新ledger；
- static flat、distributed FL及legacy HFL baseline仍能執行。

### 7.2 必要negative cases

- artifact URL key與實際下載bytes不符；
- bundle component set、typed metadata或workload compatibility不合法；
- callback correlation、round或stage不符；
- state dict keys／shape／dtype不相容；
- persisted typed request／profile和目前resource語意不一致。

不得保留或新增component、weights、tensor、callback-body、topology、request或profile
hash mismatch tests。

---

## 8. 實作順序

1. Characterize whole-artifact key的publisher、URL、downloader與所有carrier fields。
2. 建立hash allowlist，將其餘production SHA／digest用途映射到本文件的替代語意。
3. 先簡化typed artifact metadata、builders與loaders，再調整FL Client／Server／Root。
4. 將Notify改為resource／stage state idempotency。
5. 移除training scope、dataset evidence與training-data collection content hashes。
6. 移除static topology／scope fingerprint並更新flat／hierarchical state。
7. 更新persisted ledger migration、fixtures與tests。
8. 執行focused tests、PyMTLF full suite、ruff及相關real-process regression。
9. 進行production與test-code review，保留unstaged diff供user review。

---

## 9. Verification

實作完成後在`PyMTLF/`執行：

```bash
.venv/bin/ruff check src tests
.venv/bin/pytest -q \
  tests/test_artifact_repository.py \
  tests/test_health_and_artifact_api.py \
  tests/test_fl_workspace.py \
  tests/test_fl_artifacts.py \
  tests/test_fl_hierarchy_artifacts.py \
  tests/test_fl_client.py \
  tests/test_fl_server.py \
  tests/test_fl_root.py \
  tests/test_fl_flat.py \
  tests/test_fl_topology.py \
  tests/test_training_scope.py \
  tests/test_training_data.py \
  tests/test_training_data_collection.py \
  tests/test_collection_relay.py \
  tests/test_dataset.py \
  tests/test_training_jobs.py \
  tests/test_local_trainer.py \
  tests/test_federated_trainer.py
.venv/bin/pytest -q
git diff --check
```

若`NWDAF/`或`nwdaf-resources/`因確認的contract dependency進入change set，需另外依各
repository政策執行focused與full regression；沒有修改時不建立空白commit。

---

## 10. 驗收條件

Slice 4A只有在以下條件全部成立後才可標為Ready for User Review：

1. Production code中只有完整artifact repository key及URL/body verification仍使用
   cryptographic hash。
2. `file_digests`及所有model、weights、scope、dataset、tensor、Notify、topology、
   collection content digest不再被新runtime state產生、要求或驗證。
3. Whole-artifact carrier fields仍一致指向同一repository key，沒有新增第二份expected
   hash設定。
4. Explicit identity、resource state與typed compatibility checks足以完成原有必要流程。
5. Flat、distributed FL、legacy HFL、training-data collection與artifact regression通過。
6. 既有traffic workload regression通過；Slice 4 plan不要求local shard或新
   image-classification bundle提供manifest／hash。
7. Slice 4與Slice 5 plans及fixtures不再依賴已移除digest。
8. Production code、test code與migration compatibility review完成，remaining gaps明列。
9. 所有intended changes保持unstaged／uncommitted供user review。

正式testbed validation不屬於本slice；若local real-process regression尚未執行，必須記為
remaining gap，不以unit tests代替。

---

## 11. 明確延後

- Legacy hierarchy assignment／preparation-result artifact roles的最終移除仍由Slice 6
  處理。
- Retained-result runtime維持暫緩。
- Branch failure detection／replacement selector不因移除digest而納入。
- 不新增signature、HMAC、Merkle tree、dataset certification或其他替代hash方案。
