# R0 Compatibility Oracle Framework

Date: 2026-07-13

Status: Analytics slice completed with R1; Accuracy, Model, and Reporting slices planned

Parent plan:

- `Behavioral Parity Remediation Plan.md`

Related records:

- `Behavioral Parity Audit.md`
- `R1 Analytics Shaping Parity.md`
- `R2 Accuracy Measurement Parity.md`

Affected repositories:

1. `PyAnLF/`
2. `NWDAF/`
3. `nwdaf-docs/`

---

## 1. Purpose

R0建立behavioral parity修復共用的compatibility oracle、fixture格式、failure evidence與驗證
流程。它不擁有analytics、accuracy、model lifecycle或reporting lifecycle演算法，也不是一個
必須先完整結束才可進入R1的獨立implementation stage。

R0以slice方式伴隨R1至R4落地：

```text
R0 common framework
├── R0 Analytics slice  + R1
├── R0 Accuracy slice   + R2
├── R0 Model slice      + R3
└── R0 Reporting slice  + R4
```

每個slice先用historical sources建立不依賴current PyAnLF output的expected behavior，確認修復前
會失敗，再與該stage的implementation fix一起commit。R0的目標是防止migration以「新程式自身
測試通過」取代「原有domain behavior已保留」的證明。

---

## 2. Scope

### 2.1 Included

1. Historical source、commit與test provenance規則
2. Golden fixture共同格式與目錄配置
3. Frozen/injected clock與deterministic execution規則
4. Initial failure evidence的保存方式
5. Historical invariant與approved behavior change的區分
6. R1至R4各slice的test inventory與traceability
7. Test-first但正式commit保持green的cutover規則
8. Environment-only scenario與accepted risk的明確紀錄

### 2.2 Excluded

1. 直接修正任何domain algorithm
2. 把歷史Go implementation重新放回production runtime
3. 建立Go與Python雙寫或runtime differential mode
4. 重新定義已確認的historical invariants
5. 將current PyAnLF output反向寫成golden expected value
6. 為了測試方便降低cross-repo contract或lifecycle驗收標準

---

## 3. Oracle Hierarchy

每個fixture依以下順序決定expected behavior：

1. `Behavioral Parity Remediation Plan.md`內已核准的decision
2. 搬移前NWDAF production code
3. 搬移前NWDAF tests與獨立修正commit
4. Phase 2至Phase 4已核准且明確記錄的新語意
5. Current cross-repo contract中不改變domain meaning的transport shape

若historical code、historical test與approved decision互相衝突，該fixture不得自行選邊。必須在
對應stage文件記錄contradiction，依development policy提出decision gate後才可固定expected。

Current PyAnLF implementation只能用來：

1. 找到目前實際差異
2. 建立修復前failure evidence
3. 確認既有非parity behavior沒有被修復破壞

它不能作為historical expected value的來源。

---

## 4. Historical Baselines

| Slice | Primary production baseline | Primary test baseline |
| --- | --- | --- |
| Analytics | `NWDAF@0db9584:internal/anlf/analytics.go` | `NWDAF@0db9584:internal/anlf/analytics_test.go` |
| Accuracy | `NWDAF@a7d0693:internal/anlf/accuracy/` | `NWDAF@a7d0693:internal/anlf/accuracy/*_test.go` |
| Accuracy state | `NWDAF@a7d0693:internal/context/accuracy_store.go` | corresponding context tests |
| Model lifecycle | Phase 2 Go/PyAnLF cutover history與current contracts | lifecycle/runtime manager tests |
| Reporting completion | `NWDAF@0db9584`與`NWDAF@62e2f9f` | notifier/processor tests |
| Current ownership | `NWDAF@64fdf0a`、`PyAnLF@60994c3` | current Go/Python tests |

對單一fixture若有更精確的feature commit，應額外記錄。例如analytics使用：

1. `2e2cca4`：two-step timestamp bucketing
2. `3936e7d`：continuous range與zero-padding
3. `e8c8978`：prediction target slot

不得用單一commit覆蓋所有area，因為不同責任的最後pre-migration baseline不同。

---

## 5. Fixture Organization

### 5.1 Target Layout

```text
PyAnLF/tests/
├── fixtures/
│   └── behavioral_parity/
│       ├── README.md
│       ├── analytics_*.json
│       ├── accuracy_*.json
│       └── lifecycle_*.json
├── test_analytics_parity.py
├── test_accuracy_parity.py
├── test_model_lifecycle_parity.py
└── test_reporting_lifecycle.py
```

`README.md`記錄fixture schema、時間表示、numeric comparison與provenance規則。Tests負責載入
fixture並對public/internal stable seam執行；fixture不得包含Python-specific serialized object。

### 5.2 Required Metadata

每個fixture至少包含：

```json
{
  "fixture_id": "analytics-missing-middle-slot",
  "purpose": "preserve an internal missing slot as a zero observation",
  "classification": "historical_invariant",
  "provenance": {
    "repository": "NWDAF",
    "commit": "0db9584",
    "source_path": "internal/anlf/analytics.go",
    "test_path": "internal/anlf/analytics_test.go",
    "test_name": "TestAggregateInMemory_ZeroPadding"
  },
  "clock": {},
  "config": {},
  "input": {},
  "expected": {}
}
```

Required semantics：

1. `fixture_id`在全部slices內唯一且穩定
2. `classification`只能是`historical_invariant`或`approved_change`
3. `provenance`必須能用local `git show`重新取得
4. `clock`明確提供所有影響結果的logical time
5. `config`只放該case真正生效的設定
6. `input`使用language-neutral JSON shape
7. `expected`逐欄固定，不使用snapshot自動接受current output

### 5.3 Numeric And Time Rules

1. Timestamp輸入使用RFC 3339 UTC或明確Unix seconds
2. Expected timestamp必須逐值比較，不只比較sequence length
3. Integer volume/packet fields精確比較
4. Throughput與metrics只在歷史公式涉及floating point時使用明確tolerance
5. Tolerance必須寫在test helper，不得由fixture隱含放寬
6. Slot identity、slot index與rounding結果一律精確比較
7. Collection iteration順序不得影響expected output

---

## 6. Deterministic Test Seams

### 6.1 Clock

任何依賴以下時間的class都必須能注入clock或直接接收logical time：

1. inference/report generation time
2. observation retention cutoff
3. periodic accuracy check time
4. scheduler tick與completion time
5. retry `next_attempt_at`

Production預設仍使用UTC wall clock。Unit/parity tests不得使用`sleep`等待正確性，也不得使用
2099之類日期逃避retention；fixture應完整控制logical time。

### 6.2 Ordering

Tests必須重複或打亂不具語意的container insertion order，證明：

1. source registry的`set`/map iteration不影響結果
2. 可交換的cross-source sum不依賴winner order
3. last-wins只由同一source/session的ingest order決定
4. concurrency tests使用barrier/event控制，不依賴scheduler timing運氣

### 6.3 Test Layers

1. Pure domain tests：alignment、metrics、identity等純函式
2. Component tests：store、runtime manager、scheduler state transition
3. HTTP contract tests：PyAnLF router與Go internal server的request/response
4. Cross-repo tests：只用於必須同時證明兩端contract的項目
5. Environment E2E：需要實際5GC/testbed時獨立記錄，不用unit test結果取代

---

## 7. Slice Mapping

| R0 slice | Paired stage | Main fixture families | Completion meaning |
| --- | --- | --- | --- |
| Analytics | R1 | alignment、source identity、padding、target timestamp | Model input shaping恢復歷史語意 |
| Accuracy | R2 | slot matching、UL/DL metrics、cadence、confidence readiness、scope、per-session ring cap | MTLF measurement input恢復歷史語意 |
| Model | R3 | identity、generation、same URL、dedup、atomic replacement | Provision/lifecycle state正確 |
| Reporting | R4 | max reports、monitoring duration、completion retry | Subscription completion可收斂 |

某一slice完成不代表整體R0完成。R0整體完成必須等四個slice都具有provenance、initial failure
evidence、green compatibility tests與remaining environment gap紀錄。

---

## 8. Failure Evidence Workflow

每個slice依以下順序執行：

1. 從historical sources手工建立fixture expected values
2. Review fixture provenance與approved-change classification
3. 在未套用fix的current implementation執行selected tests
4. 記錄failed fixture ID、assertion差異與對應audit finding
5. 確認failure原因與audit reproduction一致
6. 實作paired stage fix
7. 重跑selected tests、slice tests與existing full tests
8. 在paired stage文件進度紀錄initial failure與final pass結果

Failure evidence至少記錄：

```text
Fixture:
Audit finding:
Command:
Current result:
Expected result:
Root mismatch:
Fixed by:
Final verification:
```

不要求提交完整terminal log；文件需保留足以重現的command、fixture ID與差異摘要。如果failure
受到environment限制，必須標記為`environment-only`，不能改寫expected讓它通過。

---

## 9. Commit And Review Rules

### 9.1 Commit Strategy

1. 不建立獨立red-test commit
2. Initial failing tests可在local worktree短暫存在
3. Tests與其直接對應的fix在同一implementation commit保持green
4. 不把R2至R4尚未修正的red tests混入R1 commit
5. Behavior-preserving test infrastructure可獨立commit，但不得掩蓋尚未證明的failure
6. `nwdaf-docs`進度更新維持獨立repository commit

### 9.2 Review Gate

每個paired stage review必須回答：

1. Expected value是否來自historical/approved source，而非current output
2. Historical invariant是否被完整移植，而非只修正單一example
3. Existing current behavior中已核准的新語意是否仍保留
4. 測試是否控制clock與ordering
5. 是否新增未核准的fallback、tolerance或transport guarantee
6. 是否有P0/P1 finding仍缺test mapping

---

## 10. R0 Analytics Slice

第一個落地slice與R1一起進行，詳細內容由`R1 Analytics Shaping Parity.md`定義。

本slice至少建立：

1. clean single-source sequence
2. missing middle slot
3. anchor drift
4. same-session duplicate last-wins
5. two sources with same anchor
6. two sources with different anchors
7. same metadata but different source IDs
8. input-window cap
9. mean output timestamp
10. positive half-slot rounding
11. negative half-slot rounding

Initial failure evidence至少要覆蓋BP-01至BP-05，BP-06由完整fixture/test inventory關閉。

### 10.1 Implementation Result（2026-07-13）

R0 Analytics slice已和R1一起完成：

1. `PyAnLF/tests/fixtures/behavioral_parity/analytics_cases.json`建立11組language-neutral fixtures
2. Fixture逐組記錄historical commit、source path、test path與expected output
3. `test_analytics_parity.py`直接驗證pure alignment，不以current PyAnLF output產生expected values
4. 修復前執行結果為`8 failed, 3 passed`
5. 修復後11組fixtures、正負half-round helper與source ordering assertion全數通過
6. Analytics runtime component test另外驗證predictor input與first prediction target timestamp

Implementation commit：`PyAnLF@fc59df2`。

Initial failure摘要：

| Fixture | Current result | Historical expected | Root mismatch |
| --- | --- | --- | --- |
| `analytics-missing-middle-slot` | `[t0, t10]` | `[t0, zero@t5, t10]` | existing buckets only |
| `analytics-same-metadata-different-source` | UL 20 | UL 30 | source ID在shaping前遺失 |
| `analytics-two-sources-different-anchor` | timestamp 27 | timestamp 26 | maximum raw timestamp取代mean center |
| `analytics-round-positive-half` | 兩筆合併於raw t5 | centers 0與10 | Python half-to-even |
| `analytics-round-negative-half` | 只保留一筆 | centers 0與10 | timestamp sort加Python half-to-even |

完整修復前failure inventory另包含anchor drift timestamp、same-slot representative timestamp與獨立
mean timestamp fixture。正式implementation沒有提交red state；fixtures與fix保持同一個green change。

本slice完成不代表R0整體完成。Accuracy、Model與Reporting slices仍需分別伴隨R2至R4落地。

---

## 11. Completion Criteria

### 11.1 Slice Complete

單一slice完成需要：

1. 所有fixture有可重現provenance
2. Selected fixtures已在修復前證明失敗或明確記錄原本已通過
3. 對應implementation與compatibility tests一起通過
4. Existing tests仍通過
5. 對應audit findings具有test mapping與closure狀態
6. Environment-only gaps與accepted risks已記錄

### 11.2 R0 Complete

1. Analytics、Accuracy、Model、Reporting四個slice都完成
2. 所有P0/P1 findings至少有一個fixture或明確test mapping
3. Approved behavior changes有獨立tests，未被誤標為historical parity
4. Fixture README與test helpers能供後續regression持續使用
5. Parent remediation plan與audit已更新實際verification結果

R0完成只代表compatibility oracle完整；整體remediation仍需通過R5 full verification與code review。
