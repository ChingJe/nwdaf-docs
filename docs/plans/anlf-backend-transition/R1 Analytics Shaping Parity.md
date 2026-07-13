# R1 Analytics Shaping Parity

Date: 2026-07-13

Status: Ready for implementation

Parent plan:

- `Behavioral Parity Remediation Plan.md`

Test framework:

- `R0 Compatibility Oracle Framework.md`

Audit record:

- `Behavioral Parity Audit.md`

Implementation repository:

- `PyAnLF/`

---

## 1. Purpose

R1恢復Phase 3搬移前Go analytics shaping的domain behavior，使PyAnLF在相同source observations、
sampling interval、input window與inference time下，產生與historical Go相同的model input sequence
及first prediction target timestamp。

本stage與R0 Analytics slice一起進行：先建立historical golden fixtures並證明current mismatch，再
修正PyAnLF。R1不把analytics logic搬回Go，也不重開Phase 3已完成的process boundary與callback
architecture。

---

## 2. Goals

1. Observation snapshot在shaping前保留`observation_source_id`
2. 不同source的相同session metadata不再互相覆寫
3. 恢復per-source/session anchor-relative alignment
4. 恢復same-session same-slot last-wins
5. 使用與Go `math.Round`等價的half-away-from-zero
6. 恢復以inference-time `snapped_now`作global origin
7. 跨source/session在global slot相加所有traffic features
8. 恢復contributing snapped centers的mean timestamp
9. 恢復continuous range與internal zero-padding
10. 維持input-window cap與predictor既有leading padding contract
11. Prediction timestamp由最後一個shaped slot加一個sampling interval推導

---

## 3. Scope

### 3.1 Included

主要修改：

1. `PyAnLF/src/py_anlf/core/observation_store.py`
2. `PyAnLF/src/py_anlf/core/analytics_runtime.py`
3. 新增`PyAnLF/src/py_anlf/core/alignment.py`
4. 必要的clock dependency wiring in `core/runtime_manager.py`
5. `core/accuracy/monitor.py`對source-aware snapshot的mechanical data-shape adaptation
6. R0 Analytics fixtures與`test_analytics_parity.py`
7. Existing analytics/store tests的回歸補強

### 3.2 Excluded

1. Accuracy prediction records、ground-truth matching、cadence與metrics的behavior change
2. Accuracy periodic scheduler與warmup
3. Model identity、generation、cache與provision event
4. Reporting scheduler completion callback
5. Go observation delivery worker、queue、retry與batch contract
6. Go到PyAnLF observation HTTP payload變更
7. Source retention policy與batch dedup policy改寫
8. Predictor preprocessing、model architecture與confidence policy改寫
9. MTLF threshold或retrain policy調整
10. Non-`UE_COMMUNICATION` analytics

如果R1實作發現必須改動上述boundary才能恢復shaping parity，應停止並依development policy提出
blocker，不得在R1隱含擴張scope。

---

## 4. Historical Oracle

### 4.1 Primary Baseline

| Behavior | Historical source |
| --- | --- |
| Shaping implementation | `NWDAF@0db9584:internal/anlf/analytics.go` |
| Shaping tests | `NWDAF@0db9584:internal/anlf/analytics_test.go` |
| Two-step alignment intent | commit `2e2cca4` |
| Zero-padding intent | commit `3936e7d` |
| Prediction target derivation | commit `e8c8978` and `NWDAF@0db9584` |

### 4.2 Exact Historical Details

實作時以production code而非comment中的近似描述為準：

1. Session anchor是該source/session保留資料中的第一筆ingest observation，不是重新排序後的最小
   timestamp
2. Go先用`time.Time.Unix()`取得整秒，再執行`math.Round`
3. Same-slot last-wins由同一source/session的ingest order決定
4. `snapped_now = floor(inference_unix / sampling_interval) * sampling_interval`
5. Non-empty output timestamp是integer session centers的mean，再使用Go-compatible round到整秒
6. Output range以最新non-empty global bucket為`end_idx`
7. `start_idx = max(earliest_non_empty_idx, end_idx - input_window + 1)`
8. 只填補`start_idx..end_idx`內部缺口，不在第一筆真實資料前補leading zeros
9. Predictor仍負責不足fixed model input window時的leading padding
10. 第一個prediction target是最後一個shaped observation timestamp加一個sampling interval

Historical test comments若與production code細節不同，以可執行production code與assertions為準，
並在fixture provenance註記差異。

---

## 5. Current Mismatch

| Audit finding | Current behavior | Required behavior |
| --- | --- | --- |
| BP-01 | 只迭代existing buckets | 建立continuous range並補internal zero slots |
| BP-02 | store flatten後遺失source ID | snapshot保留source ID，source先獨立alignment |
| BP-03 | global origin使用latest raw observation | 使用inference-time snapped grid |
| BP-04 | timestamp取contributors最大raw time | 使用snapped centers的rounded mean |
| BP-05 | 使用Python builtin `round()` | 使用Go-compatible half-away-from-zero |
| BP-06 | 缺少historical parity tests | 建立R0 Analytics fixture suite |

Current `_shape_observations()`另外先依timestamp排序session。R1必須改成保留source內ingest order，避免
anchor與last-wins semantics在out-of-order或duplicate資料下被重新定義。

---

## 6. Target Package Shape

```text
PyAnLF/src/py_anlf/core/
├── observation_store.py
├── alignment.py
├── analytics_runtime.py
└── predictor.py

PyAnLF/tests/
├── fixtures/behavioral_parity/
│   ├── README.md
│   └── analytics_*.json
├── test_analytics_parity.py
└── test_analytics_runtime.py
```

Responsibility：

1. `observation_store.py`擁有source buffers、binding、retention、batch dedup與immutable snapshot
2. `alignment.py`擁有純deterministic shaping algorithm，不知道runtime、HTTP或model
3. `analytics_runtime.py`協調shaping、inference、fallback與analytics-domain report
4. `predictor.py`維持模型preprocessing、leading padding與inference，不吸收source alignment

不新增空package。`alignment.py`存在的理由是讓historical algorithm可在固定clock/input下獨立驗證，
並避免`AnalyticsRuntime`同時承擔store identity與複雜時間序列演算法。

---

## 7. Internal Data Design

### 7.1 Source-aware Snapshot

新增internal immutable shape，名稱可依實作微調，但語意固定：

```text
StoredObservation {
  observation_source_id
  observation
}
```

Requirements：

1. `ObservationStore.observations_for_subscription()`回傳source-aware records
2. Source IDs以排序順序建立snapshot，消除`set` iteration nondeterminism
3. 每個source內observation維持ingest order
4. Snapshot建立時複製record list，離開lock後不暴露mutable source buffer
5. 同一source被多subscriptions引用時仍只保存一份observations
6. `source_id`已由ingest route path提供，不修改`SourceObservation` HTTP body
7. Analytics與Accuracy caller都接收同一source-aware snapshot，避免維護兩種store truth

如果型別放在`observation_store.py`，`alignment.py`只能依賴其read-only value shape，不得回頭操作
store lifecycle。

Current Accuracy monitor也使用`observations_for_subscription()`。R1只把它機械式調整為從
`StoredObservation.observation`讀取原欄位，並保留source ID供R2使用；不得在這一步順便修正
tolerance、immediate ingest processing、UL/DL collapse或report bookkeeping。這能避免建立一個
R1-only flattened legacy query，也不會提前改變R2的domain behavior。

Confirmed decision（2026-07-13）：採用單一source-aware snapshot。Analytics與Accuracy都接收相同
record shape；不建立給Accuracy使用的temporary flattened query。R1對Accuracy的修改只限於讀取
`StoredObservation.observation`，其matching、cadence、metrics與state lifecycle仍由R2處理。

### 7.2 Session Key

最低session identity：

```text
(
  observation_source_id,
  ipv4_address,
  supi,
  dnn,
  snssai.sst,
  snssai.sd,
)
```

`observation_source_id + ipv4_address`對應historical Go的`correlation ID + IP traffic data`邊界；
其餘metadata防止單一source內metadata改變時錯誤覆寫。不同session只在global bucket aggregation
階段相加。

### 7.3 Clock

`AnalyticsRuntime`接受production-default UTC clock，例如`Callable[[], datetime]`。同一次
`generate_report()`只讀取一次clock，並把同一logical `inference_time`傳給alignment和
`generated_at`/fallback path，避免跨boundary tick造成不一致。

`ObservationStore` retention clock亦應可注入，讓store component tests不依賴wall clock。兩者可由
`SubscriptionRuntimeManager`取得同一clock dependency，但不得新增config欄位或改變production
default。

---

## 8. Alignment Algorithm

### 8.1 Timestamp Conversion And Rounding

新增明確helper：

```text
go_round(x):
  x >= 0 -> floor(x + 0.5)
  x < 0  -> ceil(x - 0.5)
```

規則：

1. 不使用Python builtin `round()`建立session step、global index或mean timestamp
2. Observation timestamp先轉為Go `Time.Unix()`等價的UTC Unix whole second
3. Tests固定`+0.5`與`-0.5`結果分別為`+1`與`-1`
4. Sampling interval必須大於0；invalid config沿用或補上existing validation，不在algorithm靜默修正

### 8.2 Per-session Alignment

對每個session：

1. 按snapshot中的ingest order收集records
2. 第一筆record的Unix second作`anchor`
3. 對每筆計算`step = go_round((observed_unix - anchor) / sampling_interval)`
4. 計算`center = anchor + step * sampling_interval`
5. 以`center`作session-local key
6. 同center後到record覆寫先到record
7. 不跨source/session作dedup

### 8.3 Global Alignment

先計算：

```text
snapped_now = floor(inference_unix / sampling_interval) * sampling_interval
```

再對每個session-local center：

1. `global_index = go_round((center - snapped_now) / sampling_interval)`
2. 所有numeric traffic features加到global bucket
3. 每個session-local contribution增加一次`center_sum`與`center_count`
4. Cross-source/session aggregation只作sum，不使用last-wins

Global origin不得使用latest raw timestamp。Source/session traversal採stable ordering；即使numeric sum
理論上可交換，fixture仍要求相同input和clock重複執行得到完全相同輸出。

### 8.4 Output Range And Timestamp

若沒有bucket，回傳empty historical sequence與existing DNN fallback。

否則：

1. 排序non-empty global indexes
2. `end_idx`取最大index
3. `start_idx = end_idx - input_window + 1`
4. 若earliest non-empty index較晚，將`start_idx`clamp至earliest
5. 逐一產生`start_idx..end_idx`
6. Non-empty bucket timestamp為`go_round(center_sum / center_count)`
7. Empty bucket timestamp為`snapped_now + index * sampling_interval`
8. Empty bucket所有十個features為0
9. 不向earliest real bucket以前補leading zeros
10. 不因padding新增`end_idx`以後的future slot

如input含真實future observation，R1不另行丟棄；本stage只禁止padding自行超出最新non-empty bucket。

### 8.5 DNN

R1 fixtures假設同一subscription綁定sources具有相容collection profile與相同DNN。Snapshot以stable
source/ingest order解析existing DNN fallback，避免`set` iteration造成不穩定。不同DNN source的
policy不是本stage新議題；若實作發現production允許衝突DNN且結果會影響analytics contract，應停止
並提出decision gate。

---

## 9. Runtime Integration

`AnalyticsRuntime.generate_report()`流程調整為：

1. 讀取一次`inference_time`
2. 將source-aware snapshot與logical time交給alignment
3. 無model或無historical data時維持confidence-zero fallback
4. 有資料時將shaped sequence交給既有predictor
5. Prediction成功時，以最後一個shaped timestamp加sampling interval建立first target timestamp
6. 保留現有UL/DL predicted steps聚合、average confidence與communication duration behavior
7. 不在R1建立accuracy prediction record；此責任留給R2

若`output_window > 1`而predictor回傳多步，external report仍維持現有聚合shape。Per-step target
bookkeeping在R2處理，但R1 helper應能由first target與step index穩定推導後續slots。

---

## 10. R0 Analytics Fixtures

| Fixture ID | Historical oracle | Primary assertion | Current expected status |
| --- | --- | --- | --- |
| `analytics-clean-single-source` | `TestAggregateInMemory_SingleIP_Basic` | values與timestamps依序一致 | may pass after seam adaptation |
| `analytics-missing-middle-slot` | `TestAggregateInMemory_ZeroPadding` | middle slot存在且十個features為0 | fail |
| `analytics-anchor-drift` | `TestAggregateInMemory_Drift` | drifted report進入正確session slot | verify current mismatch |
| `analytics-duplicate-last-wins` | `TestAggregateInMemory_Dedup` | same source/session/center後者覆寫 | may pass |
| `analytics-two-sources-same-anchor` | `TestAggregateInMemory_MultiIP_SameAnchor` | 每個slot跨source加總 | pass unless metadata collides |
| `analytics-two-sources-different-anchor` | `TestAggregateInMemory_MultiIP_DifferentAnchor` | late joiner同slot聚合 | fail timestamp/origin |
| `analytics-same-metadata-different-source` | historical corr-ID separation invariant | 不覆寫，traffic相加 | fail |
| `analytics-input-window-cap` | `TestAggregateInMemory_InputWindowCap` | 只保留latest N continuous slots | verify |
| `analytics-mean-output-timestamp` | different-anchor test | centers 25與27輸出26 | fail |
| `analytics-round-positive-half` | Go `math.Round` | `+0.5 -> +1` | fail for Python builtin |
| `analytics-round-negative-half` | Go `math.Round` | `-0.5 -> -1` | fail for Python builtin |

Fixture需包含全部十個traffic features，至少一個aggregation case使用非零packet/throughput欄位，避免
修復只對UL/DL正確。

### 10.1 Prediction Target Assertion

至少一個fixture經`generate_report()`而非只測pure alignment，確認：

1. predictor收到的sequence逐欄等於expected
2. report timestamp等於最後shaped timestamp加sampling interval
3. generated/fallback time由injected clock控制

### 10.2 Ordering Assertion

至少對same-metadata multi-source fixture使用兩種binding/source insertion order，輸出必須完全相同。
Same-session duplicate fixture則保留observation ingest order，證明last-wins不是timestamp sort或source
iteration副作用。

---

## 11. Implementation Sequence

### Step 1: Establish R0 Analytics Failure Evidence

1. 新增fixture README、loader與Analytics fixtures
2. 先對current code建立最小adapter，不改變shaping semantics
3. 執行selected failing cases
4. 在本文件progress section記錄fixture ID與實際差異

Selected evidence至少包含：

1. missing middle slot
2. same metadata/different source
3. different anchor/mean timestamp
4. positive與negative half rounding

### Step 2: Introduce Deterministic Inputs

1. 加入clock injection
2. 加入source-aware immutable snapshot
3. 對Accuracy caller作data-shape adaptation，但不修改其演算法
4. 保留source refcount、retention與batch dedup tests
5. 確認HTTP observation contract沒有改變

### Step 3: Implement Pure Alignment

1. 新增Go-compatible time helpers
2. 實作per-session alignment與dedup
3. 實作global aggregation與mean timestamp
4. 實作continuous range、padding與window cap
5. 讓`AnalyticsRuntime`委派給alignment helper

### Step 4: Verify Runtime Output

1. 驗證predictor input
2. 驗證first target timestamp與external report shape
3. 驗證confidence-zero fallback未改變
4. 驗證重複執行deterministic

### Step 5: Close Slice

1. 執行focused parity tests
2. 執行existing PyAnLF full tests
3. Review BP-01至BP-06 mapping
4. 更新本文件、R0 framework、audit與parent remediation progress

---

## 12. Verification

最低執行：

```bash
uv run pytest -q tests/test_analytics_parity.py
uv run pytest -q tests/test_analytics_runtime.py
uv run pytest -q
```

如果實作新增正式lint/type-check entrypoint，再依`pyproject.toml`執行；不得臨時以未配置工具宣稱
lint通過。

Verification report必須分開列出：

1. Initial failing fixtures與差異
2. Final Analytics parity tests
3. Existing component/API tests
4. 未執行的cross-repo或environment E2E及原因

R1不修改NWDAF，因此不要求Go full suite作stage gate；若實作意外需要Go contract change，應先停止
並replan，而不是直接擴張verification scope。

---

## 13. Commit Strategy

正式implementation commit建議包含：

1. R0 Analytics fixtures與test loader
2. Source-aware snapshot與clock seam
3. Alignment implementation
4. Runtime integration與regression tests

Commit前所有included tests必須green。不得先commit red fixtures，也不得把R2至R4 fixtures提前放入
R1 implementation commit。

`nwdaf-docs`的initial failure摘要、實際檔案偏差與completion status在implementation完成後另作docs
commit；兩個repository不混合commit。

---

## 14. Risks And Controls

| Risk | Control |
| --- | --- |
| Source-aware return type影響其他caller | 先用`rg`列出全部caller並加入type/component tests |
| R1 data-shape change意外提前改變Accuracy | Accuracy只unwrap observation，existing Accuracy tests固定current behavior，R2再改語意 |
| Clock injection擴散到scheduler | R1只注入manager/runtime/store既有constructor seam，不改scheduler policy |
| Timestamp fractional semantics誤移植 | 以Go `Time.Unix()` production behavior和half-round fixtures固定 |
| Session ordering被sort改變 | Snapshot與alignment明確保留per-source ingest order |
| Zero-padding與predictor leading padding重複 | R1只補earliest到latest間internal gaps，leading shortage仍由predictor處理 |
| Different DNN sources語意不明 | 維持compatible-binding assumption；發現真實衝突時停止決策 |
| 修復shaping同時改變accuracy | R1只作source-aware data-shape adaptation，不修改accuracy domain semantics；R2再建立修復fixtures |
| Current transport failure mode被順手重構 | Single global worker、queue與retry明列non-goal |

---

## 15. Completion Criteria

R1完成必須同時滿足：

1. R0 Analytics 11組fixtures全部通過
2. BP-01至BP-06逐項有test closure
3. Source identity在shaping前不再遺失
4. Same metadata/different source不再互相覆寫
5. Internal gaps產生完整zero-valued slots
6. Global origin、mean timestamp與prediction target符合historical oracle
7. Slot identity不再使用Python builtin `round()`
8. 相同input、clock與不同無語意container order產生相同輸出
9. Existing source refcount、batch dedup、retention、fallback與shutdown tests通過
10. Go observation transport與external API沒有改動
11. Initial failure與final verification已更新回文件

完成R1只代表Analytics shaping parity恢復。Accuracy matching、metrics與prediction bookkeeping仍由R2
處理，Phase 3/4在R2完成前仍維持`behavioral parity validation incomplete`。

---

## 16. Progress

尚未開始implementation。
