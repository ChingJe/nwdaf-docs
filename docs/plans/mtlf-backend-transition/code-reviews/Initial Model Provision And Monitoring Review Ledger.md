# Initial Model Provision And Monitoring Review Ledger

Date: 2026-07-23

Status: Closed, verified, and committed on 2026-07-24

Parent documents:

- `../MTLF Backend Transition Plan.md`
- `../Phase 4 ML Model Monitoring And Accuracy Policy.md`

Affected implementation repositories:

- `NWDAF/`
- `PyAnLF/`
- `PyMTLF/`

---

## 1. Purpose

本文件是初始model provision、ML model monitoring及accuracy policy實作的
canonical review/remediation ledger。它記錄本次完整slice review已確認的問題、根因、修正邊界與驗收方式，
供後續實作逐項關閉。

本文件不代表重新設計已確認的architecture，也不建立另一個無限延伸的review cycle。後續修正結果應直接
回填本ledger；除非architecture、product scope或parent plan發生變更，不再為同一批問題建立新的
follow-up review文件。

本輪整體結論如下：

- Go的standard SBI ingress、processor及backend client分層大致符合既定ownership。
- PyAnLF發起初始模型需求、載入模型、建立monitor registration及回報WAPE的happy path已存在。
- PyMTLF接受初始需求、提供model artifact、維護monitor subscription及執行WAPE degradation policy的
  happy path已存在。
- legacy private route目前沒有重新註冊，Go production source也沒有使用`PyAnLF`或`PyMTLF`作為命名。
- 但是sync transaction、monitor lifecycle、model reuse resolver、standard resource semantics及Go
  standard boundary仍有六項確認問題。
- parent plan明定的實際三process round-trip尚未執行，因此即使unit/module tests全綠，仍不能把本slice
  標示為完成。

---

## 2. Review Basis

### 2.1 Policy And Architecture Sources

本review依下列workspace-local文件執行：

- `nwdaf-docs/docs/development_policy.md`
- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/source-orientation.md`
- `free5gc-dev-skill/references/exemplar-alignment.md`
- `free5gc-dev-skill/references/nf-architecture.md`
- `free5gc-dev-skill/references/sbi-development.md`
- `free5gc-dev-skill/references/openapi-contract.md`
- `free5gc-dev-skill/references/config-and-lifecycle.md`
- `free5gc-dev-skill/references/concurrency-lifecycle.md`
- `free5gc-dev-skill/references/testing.md`
- `free5gc-dev-skill/references/review-checklist.md`
- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 4 ML Model Monitoring And Accuracy Policy.md`

直接相關的parent-plan要求包括：

- 初始模型不存在時，由PyAnLF向MTLF backend取得模型；可相容模型存在時必須複用。
- model compatibility不能只比較Analytics ID，還必須考慮scope、validity與runtime interoperability。
- PyAnLF在產生穩定prediction/ground truth pair後才回報WAPE。
- PyMTLF只保留WAPE degradation policy，並以同一model的連續有效report判斷degradation。
- standard resource保留Release 18 representation，不以private shadow schema取代。
- private sync不能留下partial active state；失敗時backend不得被Go視為usable。
- 實際Go、PyAnLF及PyMTLF process-level round-trip是必要驗收層級，不是選配測試。

### 2.2 Release 18 OpenAPI Sources

本輪以workspace-local Release 18 YAML為wire contract來源：

- `nwdaf-docs/specs/openapi/TS29520_Nnwdaf_MLModelProvision.yaml`
- `nwdaf-docs/specs/openapi/TS29520_Nnwdaf_MLModelMonitor.yaml`
- `nwdaf-docs/specs/openapi/TS29520_Nnwdaf_EventsSubscription.yaml`
- `nwdaf-docs/specs/openapi/TS29571_CommonData.yaml`

其中與本輪問題直接相關的語意包括：

- ML model provision與monitor各operation有各自的success status、declared error status、
  `Location`及representation要求。
- monitor subscription的`modelIds`可包含多個model。
- `mLEvent`、analytics filter及target context是optional matching information，不能因為request省略它們就
  自動變成永遠不可能匹配。
- `eventReportReq`控制個別monitor resource的reporting behavior。
- `immReport`與provider產生的model notification/report屬於resource執行結果，不能直接信任並回顯
  consumer送入的同名欄位。
- standard error response使用`ProblemDetails`及`application/problem+json`，且允許的HTTP status依
  operation定義，不是所有4xx/5xx都可原樣穿透。

### 2.3 free5GC Exemplar Scope

本輪對照：

- BSF subscription CRUD的handler/processor separation、resource identity及Location處理。
- PCF consumer/notifier的outbound peer call及callback ownership。

目前Go package大方向不需要重寫。需要修正的是既有owner內的transaction、resource lifecycle與
operation-specific contract，不新增另一套API framework。

### 2.4 Current Implementation Evidence Map

下列位置是本輪finding的主要證據與預計修改入口。路徑用來協助實作定位；問題判斷仍以各finding描述的完整
runtime scenario為準，不應只修某一行表面症狀。

| Finding | Repository locations | Current responsibility involved |
|---|---|---|
| IMR-01 | `PyAnLF/src/py_anlf/sbi/routers/sync.py`; `PyMTLF/src/py_mtlf/api/sync.py` | unified sync prepare、publish及accepted response |
| IMR-02 | `PyMTLF/src/py_mtlf/core/monitor_reconciler.py`; `PyMTLF/src/py_mtlf/api/ml_model_monitor.py` | registration/subscription association、restart reconciliation及report acceptance |
| IMR-03 | `PyAnLF/src/py_anlf/core/model_demand.py` | Events Subscription demand extraction及active model compatibility lookup |
| IMR-04 | `PyAnLF/src/py_anlf/core/accuracy/monitor.py`; `PyAnLF/src/py_anlf/core/monitor_subscription.py` | report scheduling、model binding、optional context及immediate report |
| IMR-05 | `PyMTLF/src/py_mtlf/api/ml_model_provision.py`; `PyAnLF/src/py_anlf/core/monitor_subscription.py` | request storage與provider-generated response representation |
| IMR-06 | `NWDAF/internal/backend/standard_http.go`; `NWDAF/internal/sbi/processor/ml_model.go` | backend response validation、operation status matrix及redirect/Location ownership |

修正時可依實際dependency調整同repo的supporting files及tests，但不得因此改變owner或擴張到section 12的
non-goals。

### 2.5 Verification Evidence

本次review以前，現有local verification曾通過：

```text
NWDAF:  Go unit tests, lint, build and selected race tests
PyAnLF: 223 passed, 1 skipped
PyMTLF: 48 passed
```

這些結果證明現有assertion沒有失敗，但沒有覆蓋本ledger中的failure scenarios。本次review為唯讀review，
沒有用新增測試改變production state。

尚未完成的關鍵證據是：

```text
real Go process
  + real PyAnLF process
  + real PyMTLF process
  + initial model artifact HTTP download/load
  + actual standard-shaped resource round-trip
```

因此process-level integration列為獨立completion gate，而不是把「目前沒有測到」描述成已確認production
bug。

---

## 3. Current Gate

| ID | Priority | Area | Classification | Required outcome |
|---|---|---|---|---|
| IMR-01 | P1 | Unified sync | Closed | Rejected sync對所有active domain state為zero-side-effect |
| IMR-02 | P1 | Monitor lifecycle | Closed | restart後的orphan monitor resource會被清除，且不能再驅動policy |
| IMR-03 | P2 | Model reuse | Closed | resolver能從active catalog找回真正相容的model，不依賴單一latest pointer |
| IMR-04 | P2 | Monitor semantics | Closed | per-resource cadence、multiple modelIds、optional context及immReport依contract處理 |
| IMR-05 | P2 | Representation ownership | Closed | consumer不能注入並回顯provider-owned response fields |
| IMR-06 | P2 | Go standard boundary | Closed | error/redirect依operation allowlist處理，private backend URI不外洩 |
| IMR-V01 | Blocking gate | Cross-process integration | Closed | 實際三process initial model、monitor、recovery及restart round-trip通過 |

IMR-01至IMR-06均位於本次已實作的scope，應在同一批remediation中關閉。IMR-V01必須在production fixes及
各repo regression通過後執行。

---

## 4. IMR-01: Rejected Sync Can Publish Partial State

Status: Closed on 2026-07-24.

### 4.1 Expected Behavior

private unified sync的語意是：

```text
prepare and validate every local domain
  -> atomically publish one accepted snapshot
  -> perform retryable post-commit side effects
```

只要任何domain無法接受snapshot，route就應回409或503，且所有active state維持sync前狀態。Go只有在
backend明確接受完整snapshot後才能把它標記為usable。

### 4.2 Current Behavior

PyAnLF目前會先提交Events Subscription相關狀態，再處理model restore。model restore失敗時，route會捕捉
錯誤，但仍可能回傳`snapshotAccepted=true`，使Go與PyAnLF同時相信完整snapshot已接受，實際上model domain
並未恢復。

PyMTLF目前也依序replace projection、registration、monitor subscription及policy store。後段失敗時，前段
已replace的manager沒有共同rollback或單次aggregate commit。

### 4.3 Failure Scenario In Plain Terms

以PyAnLF為例：

1. Go送入包含consumer subscriptions及model state的snapshot。
2. PyAnLF成功換掉subscription state。
3. 載入snapshot指定的model時失敗。
4. HTTP response仍表示snapshot已接受，或錯誤發生時前面狀態沒有退回。
5. PyAnLF開始用「新subscription、舊或缺失model」的混合狀態工作。

PyMTLF也可能得到「新projection、舊registration或policy」的混合狀態。

### 4.4 Root Cause

目前manager各自有prepare或replace能力，但route沒有一個涵蓋所有current-slice domain的transaction
boundary。依序持有lock並呼叫多個fallible commit不等於atomic commit；捕捉exception也不會還原已發生的
mutation。

### 4.5 Remediation Design

PyAnLF與PyMTLF都採相同原則：

1. 為每個sync domain建立不修改active state的prepared candidate。
2. schema、duplicate、association、model artifact/loadability及policy validation全部在prepare完成。
3. prepare期間建立的temporary model/runtime reference若abort，必須可明確釋放。
4. 所有candidate成功後，在同一個sync/CRUD serialization boundary內：
   - 再確認state token或revision未被較新的CRUD改變；
   - 一次publish aggregate snapshot，或以明確rollback保證所有manager恢復原狀。
5. HTTP、callback delivery、scheduler activation、peer cleanup及其他可能失敗的I/O放在accepted commit後，
   以intent與retry state追蹤。
6. 任一prepare或stale-token失敗：
   - 回409或503；
   - `snapshotAccepted`不得為true；
   - backend維持unusable；
   - 所有active domain保持原值。

推薦優先使用aggregate immutable state swap，不採用「失敗後逐manager猜測如何倒帶」。

### 4.6 Required Tests

- PyAnLF在Events candidate準備後、model prepare失敗：所有active state不變。
- PyAnLF model準備成功但commit前發生較新CRUD：stale sync回409且不覆蓋新CRUD。
- PyMTLF在每一個domain prepare失敗：projection、registration、monitor及policy皆不變。
- commit後scheduler或peer reconciliation失敗：snapshot仍是accepted，intent保留並可重試。
- response為409/503時，`snapshotAccepted`永遠不是true。
- successful sync一次發布全部domain，沒有可觀察的mixed generation。

並行測試使用barrier或controllable fake，不用固定sleep作為唯一證據。

### 4.7 Completion Criteria

- fault injection涵蓋每個prepare/commit boundary。
- PyAnLF與PyMTLF focused sync tests通過。
- full Python regression與lint通過。
- process restart測試沒有partial snapshot。

---

## 5. IMR-02: Restart Can Leave An Orphan Monitor Subscription Active

Status: Closed on 2026-07-24.

### 5.1 Expected Behavior

一個PyMTLF monitor subscription只有在它仍能對應有效monitor registration時才可以：

- 接收PyAnLF report；
- 更新WAPE consecutive-degradation state；
- 觸發後續training intent。

registration刪除後，對應的downstream subscription必須進入active cleanup ownership。即使process在DELETE
完成前掛掉，restart/sync也要辨識orphan並繼續清理。

### 5.2 Current Behavior

目前可能發生：

1. registration已刪除；
2. 對PyAnLF的monitor subscription cleanup尚未完成；
3. PyMTLF process在兩者之間中斷；
4. restart sync恢復了monitor subscription，但沒有對應registration。

`MonitorSubscriptionReconciler.restore()`只重建能匹配的association；沒有匹配到的subscription仍留在
projection/store，沒有轉成pending cleanup。notification handler仍可能接受這個resource的report，並在
找不到registration時以空consumer context執行policy。

### 5.3 Consequence

這不是單純的memory leak。orphan resource仍可能：

- 接受已不應存在的accuracy notification；
- 累積degradation counter；
- 產生無法追溯到consumer/model agreement的retrain intent；
- restart後永久留存，因為沒有owner再次送DELETE。

### 5.4 Root Cause

目前association被視為reconciler能推導出的暫時結果，卻沒有把「這個downstream resource由哪個
registration擁有」及「失去owner後由誰清理」納入可恢復state。

### 5.5 Remediation Design

1. monitor subscription state保存明確owner identity：
   - local registration resource ID；
   - downstream resource ID與URI；
   - lifecycle state，例如active或pending-delete。
2. sync prepare驗證association；無法匹配registration的downstream resource不得恢復為active。
3. orphan轉成pending cleanup，由reconciler重試標準DELETE。
4. DELETE成功或peer回可視為已不存在的operation-specific status後，移除local orphan record。
5. notification path在更新policy前再次確認：
   - monitor subscription仍active；
   - owner registration仍active；
   - report的model與registration agreement一致。
6. 找不到active registration時，不得以空consumer context更新policy。
7. cleanup delivery保持有界timeout與retry；restart後由sync/reconciler重新接手，不依賴原process memory。

這項修正只處理目前initial monitor lifecycle，不提前實作後續training或model replacement。

### 5.6 Required Tests

- registration delete後、downstream delete前模擬crash；restart後resource進pending cleanup。
- orphan cleanup第一次失敗、後續成功；期間notification不更新policy。
- restart snapshot同時含有效與orphan subscriptions；有效者恢復，orphan被隔離清理。
- owner registration被較新CRUD重建但identity不同；舊orphan不能錯配給新registration。
- cleanup完成後local projection、association及policy counters都沒有殘留。

### 5.7 Completion Criteria

- 每個downstream monitor resource都有唯一可恢復owner或pending-cleanup owner。
- orphan不會被當成active resource。
- restart與retry tests可重現並關閉原failure window。

---

## 6. IMR-03: Model Reuse Depends On One Latest Model Pointer

Status: Closed on 2026-07-24.

### 6.1 Expected Behavior

PyAnLF收到新的Events Subscription時，應根據該subscription的每一個analytics demand：

1. 從已載入且仍有效的active model catalog尋找candidate；
2. 比較Analytics ID、filter、target scope、validity與runtime compatibility；
3. 找到相容model就複用；
4. 找不到才向MTLF backend發起initial model demand。

「相同Analytics ID」本身不足以證明可複用。

### 6.2 Current Behavior

目前resolver主要依賴單一`_active_notification`，代表最近一次active provision，而不是可查詢的model
catalog。

可重現的邏輯情境：

1. group A需求載入model A。
2. group B需求載入model B，latest pointer改指向B。
3. 新的group A subscription進來。
4. model A其實仍在runtime中，但resolver只看到B，因此判斷沒有相容model。
5. provision aggregate又可能認為既有scope union已涵蓋A，不再發新的PUT。
6. 新group A demand永久停在pending。

此外，current demand extraction只取一個`UE_COMMUNICATION` event；同一subscription含多個event時，其餘
demand沒有獨立解析。相容性比較目前也沒有完整納入validity、spatial validity、use-case及runtime
interoperability。

### 6.3 Root Cause

model provision notification同時被當成：

- last received notification；
- current active model identity；
- compatibility lookup index；
- aggregate provision coverage。

這四個責任不能由一個latest pointer正確表示。

### 6.4 Remediation Design

1. 建立active model catalog，至少以stable model identity及normalized applicability索引。
2. catalog entry保存：
   - model ID及artifact/runtime handle；
   - Analytics ID；
   - normalized filter與target scope；
   - validity time及spatial validity；
   - use-case或interoperability metadata；
   - runtime compatibility/loading status；
   - active references或可安全回收資訊。
3. 每個Events Subscription的每個analytics event建立獨立`ModelDemand`，不可只取第一個
   `UE_COMMUNICATION`。
4. resolver依序過濾：
   - analytics type相容；
   - requested scope被model applicability涵蓋；
   - model仍在validity window及spatial validity內；
   - runtime支援artifact/interface；
   - model不是pending removal或failed runtime。
5. 多個相容candidate時使用deterministic selection，例如最精確scope優先，再以已載入/最新有效者排序。
6. provision aggregate只用來決定是否需要向MTLF backend擴充需求，不得取代catalog lookup。
7. subscription delete/replacement更新model reference，但本slice不要求實作後續model generation或
   replacement transaction。

### 6.5 Required Tests

- A model、B model依序載入後，新A demand仍複用A。
- 相同Analytics ID但group/filter不同時不錯誤複用。
- scope相容但model已過期時不複用並產生initial demand。
- runtime格式不支援時不複用。
- 一個subscription含多個analytics events時，每個event獨立resolve。
- 等價subscription使用normalized ModelKey得到相同結果。
- delete最後一個reference後，catalog lifecycle符合既定cleanup規則。

### 6.6 Completion Criteria

- model reuse不依賴last notification。
- A/B/A、expired與multi-event focused tests通過。
- 現有generic seed happy path不回歸。

---

## 7. IMR-04: Monitor Resource Semantics Are Only Partially Implemented

Status: Closed on 2026-07-24.

### 7.1 Expected Behavior

每一個ML model monitor subscription是獨立standard resource。它自己的request決定：

- 要監控哪些`modelIds`；
- optional context是否縮小matching範圍；
- reporting cadence；
- 是否要求immediate cached report；
- callback URI與correlation identity。

不能用一個PyAnLF global period覆蓋所有resource，也不能在接受multiple modelIds後只實際監控其中一個。

### 7.2 Current Behavior

目前有四個相互關聯的問題：

1. scheduler使用PyAnLF global `report_period`，沒有依個別resource的
   `eventReportReq.repPeriod`建立cadence。
2. request可包含多個`modelIds`，service卻只選一個analytics runtime。
3. request省略optional `mLEvent`時，exact-match邏輯可能使它無法匹配任何runtime。
4. `immReport`沒有完整實作既定的cached-only語意。

結果是API表面接受比runtime實際支援更廣的standard representation。

### 7.3 Root Cause

monitor scheduler最初以單一local experimental loop建立，resource parser後來擴充為Release 18形狀，但
runtime binding及clock ownership沒有一起改成per-resource/per-model。

### 7.4 Remediation Design

1. canonical monitor resource保存normalized reporting requirement。
2. 每個resource依`eventReportReq`計算自己的next due time；未提供時才使用documented default。
3. scheduler可以共享單一clock/worker，但due decision必須以resource identity計算。
4. 每個`modelId`建立明確binding：
   - 全部支援時全部監控；
   - 任一model無法解析時，create/update依OpenAPI error contract整體拒絕；
   - 不接受後靜默忽略其餘model。
5. optional context作為constraint：
   - request有提供時必須驗證/match；
   - request省略時不能因`None != runtime context`而拒絕。
6. `immReport=true`時：
   - 有符合resource與model的有效cached report才在response提供；
   - 沒有cache時不為了create同步阻塞等待新prediction window；
   - 不回顯consumer注入的report。
7. report delivery前再次確認resource revision與model binding仍active，避免delete/update後送出late report。

### 7.5 Required Tests

- 兩個resource分別使用30秒與90秒period，在controlled clock下各自到期。
- 一個resource含兩個modelIds，兩者都有獨立有效report。
- 第二個model不存在時，resource create/update不會半接受。
- 省略mLEvent但modelIds可解析時成功；提供不相容mLEvent時依contract拒絕。
- `immReport=true`且有cache時回傳cache；無cache時成功建立但不虛構report。
- resource update改period後不留下舊scheduler ownership。
- delete後即使舊worker剛完成計算，也不再delivery。

### 7.6 Completion Criteria

- accepted representation與runtime behavior一致。
- controlled-clock tests不使用真實長時間sleep。
- multiple-model與optional-context contract tests通過。

---

## 8. IMR-05: Consumer Can Inject Provider-Owned Response Fields

Status: Closed on 2026-07-24.

### 8.1 Expected Behavior

standard request representation可以包含同一schema中的optional欄位，但provider產生的執行結果必須以
provider current state重建。例如：

- PyMTLF回傳的model provision notifications必須來自它實際可提供的model/artifact。
- PyAnLF回傳的immediate accuracy report必須來自它實際已計算的cache。

consumer送入同名欄位不能變成可信server state。

### 8.2 Current Behavior

PyMTLF會接受request中的`mLEventNotifs`。當`immRep=true`但server沒有seed result時，current
representation沒有清除這個request field，因此可能把consumer提供的model URL原樣回傳。

PyAnLF monitor resource也有相同類型風險：request提供的`immReport`可能被保存/回顯，而不是由cached
accuracy state生成。

### 8.3 Consequence

consumer可以讓resource看起來像是provider已經：

- 準備好某個model artifact；
- 驗證某個model URL；
- 產生一筆accuracy report。

後續owner若信任該response，可能下載未經provider選擇的URL或用虛假report更新policy。

### 8.4 Root Cause

目前resource store保存接近原始request的完整dict，response builder再對其做增量修改。當provider沒有新值
可覆蓋時，request中的response-only內容會殘留。

### 8.5 Remediation Design

1. request ingestion明確分離：
   - consumer-owned configuration；
   - provider-generated state。
2. canonical store只保存允許由consumer設定的欄位及必要raw extension。
3. `mLEventNotifs`、generated model URL、calculated accuracy/immediate report等provider-owned欄位在request
   ingestion時剝離。
4. GET/create/update response由canonical configuration加上current provider state重新組裝。
5. 沒有provider result時省略optional result field，不製造placeholder，也不回顯request value。
6. unknown Release 18 extension仍依raw-preservation規則保留，但已知provider-owned field不屬於
   passthrough extension。

### 8.6 Required Tests

- request注入任意`mLEventNotifs` URL，response與store均不含該值。
- server有seed model時，response只包含server生成的artifact URL。
- request注入`immReport`，沒有cache時不回顯。
- 有cache時response內容來自cache，與request注入值無關。
- create、GET與update走相同representation ownership規則。

### 8.7 Completion Criteria

- consumer無法偽造provider execution result。
- raw Release 18 preservation tests仍通過。

---

## 9. IMR-06: Go Error And Redirect Forwarding Is Too Broad

Status: Closed on 2026-07-24.

### 9.1 Expected Behavior

Go是external standard SBI owner，也是backend routing boundary。它必須：

- 依目前operation的OpenAPI status matrix決定可穿透的status。
- 驗證`ProblemDetails`及media type。
- 自己處理standard redirect ownership。
- 絕不把private backend URI暴露給external consumer。
- 不讓Python自行follow external NF redirect而繞過Go的standard communication ownership。

### 9.2 Current Behavior

shared standard request helper目前會接受任何`>=400`且可解析為`ProblemDetails`的backend response，而不是
套用operation-specific declared error allowlist。

redirect方面，如果backend回307/308及`Location`，private PyAnLF/PyMTLF URI有機會被直接傳給external
consumer；另一個方向則可能讓Python client follow ADRF或其他NF redirect，繞過Go。

### 9.3 Consequence

- external API可能出現該operation規格沒有宣告的status。
- consumer可能看到只能在local deployment使用的backend URL。
- standard NF peer communication ownership從Go洩漏到Python。
- create/read/update/delete共用過寬allowlist，使其中一個operation合法的redirect/error錯套到另一個
  operation。

### 9.4 Root Cause

目前shared helper只驗證「是不是看起來像standard error」，沒有接收operation contract。redirect也被當成
普通response pass-through，而不是resource/peer routing行為。

### 9.5 Remediation Design

1. 為每個ML model provision/monitor operation建立小型explicit contract：
   - success statuses；
   - declared error statuses；
   - redirect statuses；
   - Location及response representation要求。
2. shared helper接收operation contract，不建立全域「所有4xx/5xx都可穿透」規則。
3. backend回未宣告status、錯誤media type或malformed ProblemDetails時，Go轉為bounded
   `502 Bad Gateway` ProblemDetails。
4. backend回redirect時：
   - 若該operation不允許，視為invalid backend response；
   - 若允許，Go決定是否follow external standard target或重寫成Go-owned public resource URI；
   - 不傳出private backend host/port。
5. outbound Go standard client設定明確redirect policy；Python對外standard-shaped request只送到Go，不自行
   follow external NF redirect。
6. callback URL仍指向實際data owner時，該公開URL必須由config明確提供，不可從private backend response
   自動洩漏。

### 9.6 Required Tests

- 每個operation的所有declared success/error status可正確處理。
- backend回未宣告的418/redirect時，external response為502 ProblemDetails。
- malformed ProblemDetails或錯誤Content-Type不直接穿透。
- private backend `Location`不出現在external response。
- 合法redirect由Go處理後，後續standard request仍由Go送出。
- create與individual resource operation使用不同allowlist。

### 9.7 Completion Criteria

- status matrix直接對照local Release 18 YAML。
- Go focused handler/processor/backend-client tests通過。
- Go full tests、lint、build及相關race tests通過。

---

## 10. IMR-V01: Required Three-Process Round-Trip Has Not Been Run

Status: Closed on 2026-07-24.

### 10.1 Classification

這是blocking verification gap，不是單憑缺少測試就宣稱的production bug。

parent plan的required acceptance levels與final acceptance均要求actual Go、PyAnLF、PyMTLF process-level
round-trip。目前證據只有：

- Python TestClient及mocked transports；
- Go `httptest` backend；
- individual repository unit/module tests。

它們無法證明三個process的config、port、URL、serialization、callback及artifact download實際可以接起來。

### 10.2 Required Harness

建立最小、deterministic、可重跑的integration harness：

1. 使用dynamic free ports或明確隔離的test config啟動Go、PyAnLF及PyMTLF。
2. 等待三者health/sync進入usable，不用固定sleep猜測啟動完成。
3. 向Go建立一個沒有可用model的Events Subscription。
4. 驗證PyAnLF經Go建立initial model provision demand。
5. 驗證PyMTLF提供server-owned model notification及artifact URL。
6. PyAnLF透過HTTP下載artifact並實際完成runtime load。
7. 驗證PyAnLF經Go建立monitor registration。
8. 驗證PyMTLF經Go建立對PyAnLF的monitor subscription。
9. 注入deterministic prediction/ground-truth samples。
10. 驗證PyAnLF只在valid paired window形成後計算並回報WAPE。
11. 驗證PyMTLF收到report並更新同一model/resource的consecutive degradation state。
12. 刪除consumer subscription，驗證monitor resource依ownership清理。
13. orderly shutdown三個process，確認沒有殘留child process或test data。

本harness不需要真實NRF、SMF、UPF、ADRF或MongoDB；這些不屬於本slice的驗收目的。外部sample source可使用
deterministic local fixture，但Go、PyAnLF、PyMTLF及artifact HTTP path必須是真實process/network path。

### 10.3 Required Failure Cases

至少另外覆蓋：

- PyMTLF暫時不可用後恢復，Go polling、sync及usable transition完成。
- model artifact download第一次timeout/失敗，retry後成功，且沒有半載入runtime。
- backend回未宣告status或private redirect，Go不外洩。
- PyMTLF在registration delete與downstream cleanup之間restart，orphan得到清理。

### 10.4 Completion Criteria

- harness可由一個documented command重跑。
- success及必要failure scenarios均通過。
- test log記錄resource IDs與state transitions，但不含secret。
- parent plan completion record與`docs/progress/project_progress.md`只在本gate通過後標示本slice完成。

---

## 11. Remediation Execution Order

為降低問題互相遮蔽，建議按以下順序實作：

### Slice A: Transactional Sync

- 關閉IMR-01。
- 先寫fault-injection tests，再改aggregate prepare/commit。
- focused tests通過後再進下一slice。

### Slice B: Recoverable Monitor Ownership

- 關閉IMR-02。
- 明確保存registration與downstream resource association。
- 完成restart/orphan cleanup tests。

### Slice C: Model Catalog And Demand Resolution

- 關閉IMR-03。
- 保留目前已確認的initial-demand flow，不提前加入training/new generation。
- 完成A/B/A與multi-event tests。

### Slice D: Standard Monitor Semantics

- 關閉IMR-04與IMR-05。
- 使用controlled clock及server-owned response state。
- 對所有accepted standard options提供真實行為，不做silent partial support。

### Slice E: Go Wire Boundary

- 關閉IMR-06。
- 直接由Release 18 YAML建立operation matrix tests。
- 不藉此重寫既有handler/processor/client layout。

### Slice F: Unified Verification

1. 各repo focused tests。
2. 各repo full lint/test/build/race gate。
3. IMR-V01三process harness。
4. 唯讀final review只檢查本ledger completion與變更diff。
5. 全部通過後才更新parent plan及project progress的completion record。

依目前workspace policy，每個slice完成後保留可驗證checkpoint即可；除非使用者另行要求，不需要每個slice各自
commit。最終應在所有slice完成、full verification與final review後再準備整體commit。

---

## 12. Explicit Non-goals

本次remediation不得順便擴張到：

- ADRF fetch instructions及dataset retrieval。
- MongoDB training dataset讀取。
- local training、new model generation及retraining execution。
- retrained model replacement、rollback及reprovision transaction。
- multiple NWDAF FL、AGG、AoI routing或leaf topology。
- TLS、OAuth或跨process certificate/key設計。
- Go restart persistence。
- real NRF/SMF/UPF/ADRF integration。
- Phase 7 legacy unreachable code cleanup。

這些項目即使與未來流程相關，也不能用來阻擋本ledger current gate；反之，本ledger也不能以「未來會做」
為理由延後目前initial model及monitor slice已承諾的行為。

---

## 13. Decision Status

本ledger沒有新增需要product-level決策的項目。修正方案都由既有共識直接推導：

- standard representation與HTTP status依Release 18 OpenAPI。
- Go保持standard communication owner。
- PyAnLF保持analytics runtime、model demand及accuracy report owner。
- PyMTLF保持model provision、monitor resource及WAPE degradation policy owner。
- initial provision只處理現有model取得與複用；training/new generation留待後續。

若實作發現必須改變以上ownership、拒絕目前已接受的standard request shape，或需要新增private API，才應依
development policy停在decision gate；不能在修正中自行改約定。

---

## 14. Ledger Update Rules

每個finding關閉時，直接在本文件對應section補充：

- status；
- production behavior change；
- focused test command與結果；
- full regression結果；
- process-level evidence（如適用）；
- closing commit。

所有IMR-01至IMR-06與IMR-V01關閉前：

- parent plan不得標示為fully complete；
- 既有未提交implementation不得因unit tests全綠就直接commit；
- 不建立第五次相同範圍的full review文件。

如果final review只發現本ledger修正直接造成的新regression，新增finding到本ledger並標示origin；如果只是
future-phase缺口，記到相應future plan，不重新打開本slice。

---

## 15. Closure Record

### 15.1 Production Outcomes

2026-07-24已完成本ledger要求的同批remediation：

- PyAnLF與PyMTLF unified sync先prepare所有domain，再在共同serialization boundary一次commit；rejected
  snapshot不發布partial active state。
- Go sync的所有list欄位，包括沒有association的`nwdafSubscriptionIds`，固定序列化為JSON array而非
  `null`，避免跨語言snapshot被Pydantic拒絕。
- monitor subscription以private sync metadata保存原始registration identity；相同scope但不同
  registration identity不能在restart後錯配。失去owner的resource隔離為orphan、停止policy processing並由
  reconciler重試標準DELETE。
- PyAnLF使用active model catalog解析每個analytics demand，完成A/B/A reuse、scope、validity與runtime
  interoperability checks。
- monitor subscription落實per-resource cadence、multiple `modelIds`、optional context與cached-only
  `immReport`，且provider-owned response fields不再從consumer request回顯。
- Go使用operation-specific status allowlist驗證backend response；未宣告status及private redirect不會外洩，
  允許的external standard redirect只由Go follow。
- PyAnLF載入artifact中的`model.py`時不再於content-addressed cache產生`__pycache__`；同一已驗證cache可在
  process restart後重新驗證及載入。runtime cache已由`.gitignore`排除，既有`artifacts/initial` fixture保留。

### 15.2 Three-Process Evidence

可重跑命令：

```text
cd nwdaf-resources/tests/mtlf_model_monitor
go test -run TestInitialModelProvisionAndMonitorRoundTrip -v -count=2
```

Harness使用actual NWDAF binary、actual PyAnLF process及actual PyMTLF process，並以dynamic ports提供
deterministic local NRF/SMF/consumer/artifact fixture。連續兩次均通過，每次涵蓋：

1. 在PyMTLF尚未啟動時先接受Events Subscription，觀察PyAnLF經Go取得503。
2. 啟動PyMTLF後，由Go continuous polling、process sync及usable transition自動恢復。
3. artifact第一次GET回503；PyAnLF不發布half-loaded runtime，後續retry成功後才active。
4. initial model activation、monitor registration、owned monitor subscription及30筆input window。
5. analytics callback、兩輪WAPE notification及PyMTLF policy processing。
6. 停止PyAnLF，使registration delete與downstream subscription delete之間留下pending cleanup。
7. 停止並重新啟動PyMTLF及PyAnLF；Go依process instance變更重新sync。
8. 舊monitor subscription因owner identity已不存在而成為orphan，標準DELETE完成後才移除mirror。
9. 新runtime/monitor resources重新建立，最後刪除consumer subscription並完成SMF及monitor cleanup。
10. test cleanup orderly終止所有child processes。

Harness及完整操作／assertion說明保存在
`nwdaf-resources/tests/mtlf_model_monitor/README.md`。它不放在`NWDAF/`、`PyAnLF/`或`PyMTLF/`，避免單一
runtime repository持有跨repository path、Python environment及process orchestration責任。2026-07-24搬移後
已從新位置連續執行三次完整scenario並通過。第一次穩定性重跑曾暴露fake SMF重複使用同一resource ID的
harness競態；fixture改為每次create產生唯一ID，且只公開active callback後，三次連續結果均通過。

未宣告status及private redirect由Go focused contract tests覆蓋；這項failure不需要啟動Python process才能驗證
Go external boundary，且測試直接使用operation-specific contract。

### 15.3 Final Regression

2026-07-24 final local results：

```text
NWDAF
  go test ./...                                                   PASS
  make lint                                                      0 issues
  make build                                                     PASS
  go test -race ./internal/context ./internal/sbi/processor \
    ./pkg/service -count=1                                       PASS

PyAnLF
  uv run ruff check .                                            PASS
  uv run pytest -q                                               232 passed, 1 skipped

PyMTLF
  uv run ruff check .                                            PASS
  uv run pytest -q                                               55 passed
```

PyMTLF測試仍有FastAPI TestClient對`httpx2`的upstream deprecation warning；不影響結果。

### 15.4 Remaining Boundaries

本gate是local three-process integration，不宣稱real NRF、SMF、UPF、ADRF或MongoDB deployment E2E。
Dataset retrieval仍屬Phase 5；training、new generation artifact與updated model reprovision仍屬Phase 6。
Go process restart persistence及TLS/OAuth仍維持既定non-goal。

### 15.5 Closing Commits

| Repository | Commit | Summary |
|---|---|---|
| `NWDAF/` | `eb7f4ee` | Standard Model Provision/Monitor SBI、backend routing、mirror與sync |
| `PyAnLF/` | `923c6cb` | Initial model demand/catalog/runtime activation與accuracy monitoring |
| `PyMTLF/` | `ec8bdf4` | Seed provision、monitor ownership、WAPE policy與orphan reconciliation |
| `nwdaf-resources/` | `16a529d` | Workspace-level three-process failure/restart harness及完整操作文件 |

各runtime repository只提交自己擁有的production與unit／contract tests；跨repository process orchestration已
獨立提交到`nwdaf-resources/`。本文件與parent/progress completion record由後續同一批
`nwdaf-docs/` commit保存。
