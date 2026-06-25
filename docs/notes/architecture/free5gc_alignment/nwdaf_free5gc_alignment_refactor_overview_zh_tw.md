# NWDAF free5GC Alignment 重構總覽與設計說明

日期：2026-06-26

---

## 1. 文件目的

這份文件的目的不是重述每一份 plan，而是把這一輪
`project_scan -> remediation batches -> free5gc-alignment plans -> NWDAF code`
之間的關係串起來，幫助理解：

1. 為什麼當初會掃出這些問題
2. 這些問題背後真正的架構矛盾是什麼
3. 已完成的重構到底改了什麼邏輯
4. 為什麼這樣改比較接近 free5GC control-plane NF 的形狀
5. 目前還剩哪些殘留問題與未開工項目

這份文件同時覆蓋兩個層次：

1. 架構與 ownership 的整體方向
2. 各個 priority / round / commit 的脈絡與理由

---

## 2. 先講結論

這一輪重構的核心，不是單純把一些 bug 修掉，而是把 NWDAF 往較一致的
free5GC-style control-plane NF 邊界收斂。

收斂方向可以簡化成一句話：

> 讓 `service` 擁有 lifecycle，讓 `app` 成為共享依賴邊界，讓
> `handler / processor / consumer / context` 各自只做自己的事，並讓測試、
> config、error contract、外部 client ownership 都沿著同一條邊界走。

如果用更具體的方式描述，這幾輪重構其實在處理五件事：

1. 修正明確錯誤：例如 subscription `PUT` 只改本地資料卻沒有重建外部收集狀態。
2. 把長生命週期工作收回 app 擁有：例如 notifier scheduler、retrain workflow、ADRF retrieval loop。
3. 讓測試對準真正邊界：handler 測 handler、processor 測程序邏輯、consumer 測 outbound call、app seam 用 `gomock`。
4. 建立共享 runtime boundary：`pkg/app` 不再只是名義上的 package，而是 service、server、processor、AnLF、MTLF、consumer 共用的根 contract。
5. 把 runtime truth 從零散全域讀取收斂到 app/config seam：避免已經有 app seam 的地方又偷讀 `factory.NwdafConfig`。

---

## 3. 這輪重構前後，NWDAF 的骨架應該怎麼看

### 3.1 目前應該用這張圖理解 NWDAF

```text
cmd/main.go
  -> pkg/factory.ReadConfig(...)
  -> pkg/service.NewApp(ctx, cfg)
      -> internal/context.Init()
      -> internal/sbi/consumer.NewConsumer(app)
      -> internal/sbi/processor.NewProcessor(app)
          -> internal/anlf.NewAnlfService(app, mlClient)
          -> internal/mtlf.NewMtlfService(app, daisyClient, adrfClient)
      -> internal/sbi.NewServer(app)
  -> app.Start()
      -> Mongo init
      -> shutdown watcher
      -> MTLF scheduler
      -> SBI server
```

### 3.2 每一層的責任

依照目前 `NWDAF/` 的實作與 free5GC 參考 NF，應該這樣理解：

- `pkg/factory`
  - 負責 config struct、default、validation、getter。
- `pkg/service`
  - 負責組裝 app、建立 runtime context、初始化 consumer / processor / server、
    啟動與終止流程、waitgroup、MongoDB 初始化。
- `pkg/app`
  - 提供共享 root contract，讓不同層在不綁死具體 `NwdafApp` struct 的前提下共享
    `Config()` 與 `Context()` 等能力。
- `internal/sbi/api_*.go`
  - 負責 HTTP 邊界：讀 body、parse、回 status code、回 `ProblemDetails`。
- `internal/sbi/processor`
  - 負責程序與狀態轉移：subscription create/update/delete、SMF / UPF notify、
    AnLF / MTLF 之間的 wiring。
- `internal/sbi/consumer`
  - 負責所有 outbound client ownership 與對外呼叫。
- `internal/context`
  - 是 runtime truth，包含 subscriptions、SMF target mapping、traffic data、
    ML model info、shared model registry、ADRF state。
- `internal/anlf`
  - 做推論、模型載入、accuracy monitoring。
- `internal/mtlf`
  - 做 retrain 決策、Daisy training workflow、ADRF-assisted retrain workflow。

### 3.3 為什麼這個分層重要

如果這些責任沒有切開，後果會很直接：

1. handler 會開始偷偷做 procedure logic
2. processor 會開始自己建立 HTTP client 或偷讀 global config
3. lifecycle 無法知道哪些 goroutine 是誰啟動、誰該停
4. 測試會變成跨很多層的混合測試
5. 文件上的 `pkg/app`、`consumer`、`processor` 邊界會變成名義存在、實際失效

`project_scan` 掃出的大部分問題，本質上都屬於這一類。

---

## 4. 為什麼會有這批 issue

### 4.1 不是不能跑，而是「能跑但邊界不夠可信」

在最初 scan 時，`NWDAF/` 已經可以：

- `go test ./...`
- `make build`
- `make lint`

都通過。

所以掃出來的問題不是「專案完全不能用」，而是：

1. correctness 上有幾個明確錯誤
2. lifecycle ownership 不完整
3. test seam 不對準實際邊界
4. shared app boundary 不夠真實
5. config / contract / docs 與 runtime truth 有漂移

### 4.2 `project_scan` 真正指出的矛盾

從 `NWDAF Main Project free5GC Alignment Scan Findings.md` 與
`NWDAF Main Project Remediation Batches.md` 來看，這批問題可縮成四大類。

#### A. correctness 與 state reconciliation 問題

代表案例是 Priority 1：

- `PUT /subscriptions/{id}` 只更新本地 subscription record
- 但沒有同步清理與重建：
  - SMF subscription
  - correlation mapping
  - traffic bucket
  - ADRF tracking state
  - ML / MTLF 相關狀態

這代表 API 回傳的資源狀態，和實際外部收集圖譜不是同一份 truth。

#### B. lifecycle ownership 問題

代表案例是 Priority 2：

- periodic notifier scheduler 原本不在 app shutdown tree 內
- 一些 background work 用 `context.Background()` 自行長出生命週期
- 終止流程只停 server，不一定停所有 owned workflow

這種問題平常不一定炸，但一到 shutdown、timeout、異常路徑就會暴露。

#### C. boundary / dependency injection 問題

代表案例是 Priority 4：

- `pkg/app` 原本太窄
- 各 package 自己定義近似但不完全一樣的 `NwdafApp` interface
- consumer construction 沒有統一走 app seam
- AnLF / MTLF / processor 等路徑各自持有自己的 runtime truth / client 組裝方式

這會讓整個 repo「看起來像 free5GC」，但真正 dependency boundary 其實是碎裂的。

#### D. contract / config / governance 問題

代表案例是 Priority 5、8、10、11：

- error body 有的回 `ProblemDetails`，有的回 `gin.H{"error": ...}`
- parse path 有的用 free5GC 常見 deserialize，有的還是自定義/重複型別
- config 有 default 與 helper bug
- `nrfUri`、metrics、runtime truth 與 README / sample config 不一致
- 對於 standardized payload 與 local extension payload 缺乏明確治理規則

---

## 5. 已完成的重構主線

這裡按照實際實作完成順序來看，而不是只照 priority 編號。

### 5.1 Round 1：先解 correctness 與最危險的 lifecycle 問題

對應文件：

- `NWDAF Round 1 free5GC Alignment Implementation Plan.md`
- 相關完成狀態已吸收進 `project_scan` / `remediation batches`

對應 code / commit：

- `NWDAF` commit `5e0e271`

#### 原本問題

1. `PUT /subscriptions/{id}` 沒有真正做 full replacement。
2. notifier scheduler 脫離 app lifecycle。

#### 改動重點

1. 把 `PUT` 明確視為完整 replacement。
2. update 路徑重跑 default / notification semantics。
3. update 時重建與清理外部收集狀態，而不是只改本地 stored object。
4. app termination 時會停 active subscription schedulers。

#### 為什麼這樣改

這一輪是在恢復一個最基本的原則：

> 同一個 subscription resource 的「API 表示」與「runtime 實際狀態」必須一致。

如果 update 只改本地 record，整個後續 MTLF / ADRF / SMF collection graph 都會失真。

另一方面，notifier scheduler 若不屬於 app lifecycle，之後任何更大的 app-boundary 重構都會沒有穩定基礎。

#### 這輪之後留下什麼

- notifier 主 ownership 問題關掉了
- 但 broader lifecycle cancellation 與 app-owned I/O context 問題當時還沒收斂完成
- 這一段後來由 2026-06-25 external-client round 與 2026-06-26 remaining Priority 2 round
  接續完成 bounded closure

### 5.2 Priority 3：先把測試安全網建立在真實邊界上

對應文件：

- `NWDAF Priority 3 Test Refactor Detailed Plan.md`
- `NWDAF Priority 3 Test Refactor Phase 1 Summary.md`
- `NWDAF Priority 3 Test Refactor Completion Summary.md`

對應 code / commit：

- `NWDAF` commit `fd5c878`
- `NWDAF` commit `427ac74`

#### 原本問題

scan 指出：

1. 缺少 `api_*_test.go` handler 邊界測試
2. outbound consumer 測試策略不一致
3. processor test 有時跨太多層，甚至直接拉真實 consumer
4. app seam 缺少 consistent `gomock` pattern

#### 改動重點

1. 為所有 `internal/sbi/api_*.go` 建立 handler coverage。
2. consumer tests 收斂到一致的 outbound interception 策略。
3. processor/app seam 改用 `gomock`。
4. ADRF、ML service、Daisy 等 consumer test strategy 被統一。

#### 為什麼 Priority 3 很重要

這一輪不是「補測試數量」而已，而是先決定：

- 真正要保護的 seam 在哪裡
- 之後改 boundary 時要由哪一層的測試來擋 regressions

如果沒有這層安全網，Priority 4 之後所有共享 app boundary 的改動風險都會被放大。

#### documented SMF exception

`NsmfService` 仍保留 raw `http.Client` 路徑。

這不是漏改，而是刻意保留的例外，原因是目前 pinned
`github.com/free5gc/openapi v1.2.3` 對 NWDAF 目前需要的 SMF request shape
還不夠安全，尤其是 `upfEvents`、`bundledEventNotifyUri` 相關欄位。

這個例外被視為：

- Priority 3 已完成
- 但它是之後 Priority 10 OpenAPI / model governance 的一部分

### 5.3 Priority 3 strict follow-up：把 shared mock ownership 收正

對應文件：

- `NWDAF Priority 3 Strict Test Ownership Alignment Plan.md`

對應 code / commit：

- `NWDAF` commit `0768839`

#### 原本問題

即使 Priority 3 主體完成後，strict reassessment 還是發現：

- `internal/sbi` 與 `internal/sbi/processor` 還保留 package-local app mock shape
- shared app seam mock ownership 還不夠像 free5GC survey 的做法

#### 改動重點

1. 新增 `pkg/mockapp/`
2. shared generated app-boundary mock 放到 `pkg/mockapp`
3. `internal/sbi` 與 `internal/sbi/processor` 不再各自維護一份 app mock shape

#### 為什麼這件事獨立成 follow-up

因為它不是 coverage 不夠，而是 ownership 不夠嚴格。

free5GC survey 顯示，shared app seam mock 通常掛在 app / service 相關 package，
而不是讓每個 package 自己長一份「像 app 但其實只是 local 測試方便」的界面。

### 5.4 Priority 5：SBI error contract 與 parse path 對齊

對應文件：

- `NWDAF Priority 5 SBI Error Contract Alignment Plan.md`

對應 code / commit：

- `NWDAF` commit `cb2e066`
- `NWDAF` commit `56afd7a`

#### 原本問題

同一個 SBI server 裡：

- 有的 endpoint 回 `models.ProblemDetails`
- 有的 callback handler 回 `gin.H{"error": ...}`
- media type 與 parse path 也不一致

#### 改動重點

1. 導入 `internal/util/GinProblemJson(...)`
2. 統一用 `application/problem+json`
3. 對 covered scope 轉向 free5GC 常見的
   `GetRawData() + openapi.Deserialize(...)`
4. `api_mlmodelprovision.go` 改用 generated
   `[]models.NwdafMlModelProvNotif`

#### 為什麼這樣改

這一輪不是在追求「所有 endpoint 都一定要同一個 parse path」，
而是回到 corrected baseline：

1. outward contract 要先一致
2. parse/model path 要看目前 dependency snapshot 能否安全支援
3. 對 project-local callback 與缺少精確 generated model 的路徑，不強行假對齊

#### 這輪之後留下什麼

有些 intentionally excluded callback 仍留待後續治理：

- `POST /collector/upf-notify`
- Daisy training callback
- ADRF top-level retrieval callback exact generated model

### 5.5 Priority 8：factory / config / runtime truth hardening

對應文件：

- `NWDAF Priority 8 Factory And Runtime Config Hardening Plan.md`

對應 code / commit：

- `NWDAF` commit `5abb292`

#### 原本問題

1. `ReadConfig(...)` 沒有明確 validation pass
2. `GetSbiBindingAddr()` 有 multi-digit port bug
3. `supportedAnalytics` default 與真正 runtime support 不一致
4. README / sample config / runtime truth 漂移

#### 改動重點

1. `pkg/factory/config.go` 增加 defaults + explicit validation
2. 修正 SBI getter 行為
3. table-driven config tests 補齊
4. README 與 sample config 對齊 current runtime truth

#### 為什麼這樣改

config 在 free5GC NF 不是次要問題，而是 app startup contract 的一部分。

若 config validation 弱，很多問題會從「啟動即失敗」變成
「執行到 consumer / processor / server 某一路徑才爆」。

這一輪的目的就是先把 config 變成可信的 startup boundary，再往更大的 app boundary 改。

### 5.6 Priority 4 Phase 1：重建一個真的 shared app boundary

對應文件：

- `NWDAF Priority 4 Rebuild One Real App Boundary Plan.md`

對應 code / commit：

- `NWDAF` commit `bbf42c3`

#### 原本問題

scan 指出：

1. `pkg/app` 太窄，不是真正 shared boundary
2. server、processor、AnLF、MTLF、consumer 各自宣告自己的 `NwdafApp`
3. consumer construction 沒有 app-driven

#### 改動重點

1. `pkg/app.App` 擴成真正共用的 root contract
2. `pkg/service/init.go` 組裝 `consumer.NewConsumer(nwdaf)`
3. 各層 local interface 開始往 shared app contract 收斂
4. server lifecycle signature 也順手對齊

#### 為什麼這輪是整個重構的核心

因為它處理的是「shared boundary 到底存不存在」。

只要 `pkg/app` 不是真正的 root seam，後面所有事情都會碎掉：

- mock strategy 不穩
- consumer ownership 不穩
- runtime truth 會繼續旁路 app seam
- 任何 free5GC-style comparison 都會只停在資料夾名稱相似

### 5.7 Priority 4 Phase 2：把 root contract 再縮回 free5GC 常見形狀

對應文件：

- `NWDAF Priority 4 Phase 2 Strict free5GC Boundary Alignment Plan.md`

對應 code / commit：

- `NWDAF` commit `6569e02`

#### 原本問題

Phase 1 之後 shared boundary 已存在，但 strict review 發現：

1. `CancelContext()` 還放在 root `pkg/app.App`
2. processor test seam 對 consumer shape 的耦合仍偏大

#### 改動重點

1. `CancelContext()` 從 root app contract 移除
2. 只讓真正需要 cancellation 的 local extension seam 擁有它
3. processor 對 consumer 的依賴縮成較窄的 mockable seam
4. 移除 exported consumer test assembly path

#### 為什麼這樣改

這一輪的重點不是把 root contract 做大，而是做對。

free5GC survey 顯示，root `pkg/app.App` 通常很窄：

- `Start()`
- `Terminate()`
- `Config()`
- `Context()`

若把太多 lifecycle-specific capability 掛回 root contract，就會重新污染共享邊界。

### 5.8 Priority 4 residual runtime truth cleanup：把已經有 app seam 的地方停止偷讀 global config

對應文件：

- `NWDAF Priority 4 Residual Runtime Truth Cleanup Plan.md`

對應 code / commit：

- `NWDAF` commit `a912581`

#### 原本問題

即使 app boundary 已重建，strict reassessment 發現：

- `internal/anlf`
- `internal/mtlf`
- `internal/sbi/processor`

裡一些 runtime path 仍直接讀 `factory.NwdafConfig`。

這表示 app seam 已經存在，但 runtime truth 還是雙軌：

1. app seam
2. package-global config singleton

#### 改動重點

1. 把已觸及 runtime path 的 config 讀取改成經由 app seam
2. 對應測試也改成從 test app seam 提供 config

#### 為什麼這是 Priority 4 的殘留，而不是新議題

因為這不是重新設計 app boundary，而是讓已宣稱的 app-owned runtime truth
真正成立。

### 5.9 Priority 4 follow-up：ML / Daisy / ADRF 等 external client ownership 收回 consumer/app 路徑

對應文件：

- `NWDAF Priority 4 External Client Ownership And Priority 2 Residual Lifecycle Hardening Plan.md`

對應 code / commit：

- `NWDAF` commit `637235b`
- `NWDAF` commit `f026e77`

#### 原本問題

即使主體 Priority 4 完成後，strict reassessment 還是發現：

- AnLF 仍直接建立 ML service client
- MTLF 仍直接建立 Daisy client
- 某些 shutdown / teardown 收斂點仍不夠穩

#### 改動重點

1. ML 與 Daisy client assembly 移到
   `internal/sbi/consumer.NewConsumer(...)`
2. `processor.NewProcessor(...)` 改為消費 already-owned external clients
3. `MtlfService` 的 retrain / ADRF workflow 更清楚地掛在 app-owned lifecycle 上
4. 同一輪順手補掉了嚴格審查時發現的 shutdown convergence 殘留

#### 為什麼這件事重要

這一輪是在劃清一條很關鍵的線：

> 即使是 non-3GPP local integration，也不能任由 domain package 自行擁有外部 client。

理由是 client ownership 不是「是不是標準 API」的問題，而是：

1. 它是不是 outbound I/O
2. 它是不是 app 應該掌握的依賴
3. 它能不能被統一組裝、測試與關閉

這也是為什麼這一輪同時與 Priority 2 殘留 lifecycle hardening 相鄰。

### 5.10 Priority 2 remaining lifecycle completion：把 bounded lifecycle / ownership scope 正式關閉

對應文件：

- `NWDAF Priority 2 Remaining Lifecycle Completion Plan.md`

對應 code / commit：

- `NWDAF` commit `17f9e17`

#### 原本問題

在 2026-06-25 的 external-client round 之後，Priority 2 剩下的問題已不再是
「哪個 domain package 還自己建 client」，而是較廣泛但有界線的 lifecycle discipline：

1. 某些 runtime consumer method 仍會自己用 `context.Background()` 建 request lifetime
2. MongoDB query helper 與部分 persistence path 還會自己 root context
3. 某些 request-triggered async follow-up 在 shutdown-sensitive flow 裡，仍可能在關閉時擴張新的 runtime work
4. 一些 `context.Background()` 使用其實是合理的 cleanup / shutdown exception，但需要和 accidental runtime-root ownership 明確分開

#### 改動重點

1. 在 `internal/sbi/consumer/` 補上明確的 caller-owned context 規則。
   `timeoutContextFromParent(...)` 現在要求正常 runtime path 必須帶 parent context，
   不再默默 fallback 到 `context.Background()`。
2. `SMF / MTLF / ML / Daisy / ADRF` 的 runtime request path 改成從 caller 傳入 parent
   context，再由 consumer 內部加 bounded timeout。
3. `internal/context/db_query.go` 的 MongoDB query helpers 改成接收 caller-owned context，
   由 helper 從 parent 派生 query timeout。
4. `internal/anlf/monitor.go` 的 ground-truth lookup 現在把 monitor-owned cancellation 傳進 DB query path。
5. `internal/sbi/processor/upf_notify.go` 的 persistence write 改成走 app-owned cancel context，而不是 root background context。
6. subscription data collection、ML model provisioning 等 request-triggered async path 保留為 one-shot
   request-triggered work，但 shutdown 不再允許這些路徑繼續擴張新工作。
7. production code 中剩餘的 `context.Background()` 在這個 bounded Priority 2 scope 內，
   現在被收斂為明確的 cleanup / shutdown exception，例如：
   - bounded SBI shutdown
   - bounded cursor close cleanup
   - bounded ADRF unsubscribe cleanup

#### 為什麼這樣改

這一輪最重要的不是「把所有 `context.Background()` 刪光」，而是把 lifecycle rule
說清楚並落到 code 上：

1. 長生命週期且 app-owned 的工作，要明確掛在 app cancellation / waitgroup 上。
2. 一般 runtime outbound I/O 與 DB access，要由 caller 擁有 lifetime。
3. cleanup / shutdown exception 可以存在，但必須是 owner 明確建立的 bounded timeout，
   不是平常 runtime path 的預設行為。
4. request-triggered async follow-up 不應因為有 goroutine 就被自動升格成 app-owned worker；
   但它們也不能在 shutdown-sensitive flow 中無限制擴張新工作。

#### 完成後的判斷

到 2026-06-26 為止，canonical docs 已把 Priority 2 判定為：

- completed for bounded lifecycle / ownership scope

這代表：

1. Priority 2 本來負責的 lifecycle / ownership inventory 已有明確 closure
2. 之後仍存在的問題，不應再籠統回收成「Priority 2 還沒結束」
3. 必須改歸它們自己的 canonical home，例如 Priority 5、6、7、10、11、12

---

## 6. 已明確保留但不再算 Priority 2 殘留的問題

### 6.1 SMF raw HTTP exception 仍存在

這個例外目前是有意識保留：

- 不視為 unfinished Priority 3
- 但代表 OpenAPI / model governance 還沒結束

### 6.2 bounded scope closure 與 full free5GC parity 不是同一件事

Priority 2、4、5、8 都是針對 current intended standalone-NWDAF alignment level
完成，而不是宣告：

- 已具備完整 NRF registration/deregistration
- 已具備 metrics server parity
- 已達到 full free5GC NF lifecycle parity

這一點非常重要，因為它直接連到後面尚未開工的 Priority 11。

---

## 7. 尚未動工的部分

以下項目在 `remediation batches` 中仍是 open。
這些項目不應再被模糊回收到「Priority 2 還沒結束」。

### 7.1 Priority 6：post-subscription activation 與 late-failure signaling

這個問題不是說現在 async activation 一定錯，而是：

1. subscription resource 已接受成功
2. 下游資料收集在之後才展開
3. 如果後面 activation degraded / failed，現在缺少清楚且一致的對外表達方式

這比較像 operability / design completeness 問題。

### 7.2 Priority 7：logging boundary 與 payload hygiene

目前 logger 分類本身並不差，但還有兩種殘留：

1. 某些 consumer 路徑仍傾向記錄較多 request payload
2. 某些 goroutine / entry path 還保留較硬的 fatal-style control flow

這一輪通常會跟 Priority 6、10 一起看比較合理。

### 7.3 Priority 9：runtime config 與 lab / workflow config 分離

這是 repo 邊界問題的一部分。

現在 config 同時混了：

1. NF runtime config
2. group-resolution fallback data
3. MTLF / Daisy workflow settings
4. ADRF retrain workflow settings
5. lab / replay / local operation 方便性的資訊

Priority 8 只先把現在這個大 config 變可信，還沒有重新切割範圍。

### 7.4 Priority 10：OpenAPI / model governance

這一項特別重要，因為它決定之後如何處理：

1. standardized payload 是否應回歸 generated model
2. local extension model 應該放在哪一層
3. 何時因 dependency snapshot 落後而保留 handwritten struct 是合理的

目前 `api_mlmodelprovision.go` 已往 generated model 靠攏，但這只是治理線的一部分，
不代表整體 model governance 已完成。

### 7.5 Priority 11：決定 NWDAF 想對齊到哪種 free5GC integration level

這是目前最關鍵的策略性未決問題之一。

它實際上在問：

1. NWDAF 要維持現在偏 standalone SBI service 的形狀嗎？
2. 還是要進一步補上更完整的 free5GC NF lifecycle：
   - NRF registration / deregistration
   - metrics server
   - TLS / HTTP2 / auth / cert wiring
   - 更完整的 service truth

這個決定會影響 Priority 9、10、12 的走向。

### 7.6 Priority 12：repo 與 package ownership boundary cleanup

這是最晚處理也最合理的一項。

因為它會牽涉到：

1. test harness 與 main repo 邊界
2. workflow-specific callback HTTP glue 是否仍放在 `internal/sbi`
3. config / tools / notes / fixtures 的 repository placement

在 app boundary、config、governance、integration level 還沒定下來前，太早做這件事只會反覆搬家。

---

## 8. 這些改動背後的設計理由

### 8.1 為什麼 `pkg/app` 要窄，但又要真

這次重構不是把 `pkg/app` 做得越大越好，而是讓它變成：

1. 足夠真，確實是共享 seam
2. 足夠窄，不把所有局部能力都提升到 root contract

所以最後的方向是：

- 根 contract 提供 `Config()`、`Context()`、`Start()`、`Terminate()`
- `CancelContext()` 這種局部能力保留給 local extension seam

這樣才能同時保住：

1. shared boundary 的一致性
2. free5GC 參考 NF 常見的窄 root app 形狀

### 8.2 為什麼 outbound client 應交給 consumer / app 路徑

只要是 outbound I/O，它通常就不應該讓 domain package 自由建立。

把 client ownership 收回 consumer/app 路徑的好處是：

1. 組裝點單一
2. 測試 seam 一致
3. lifecycle 更容易收斂
4. config / endpoint truth 不會散落在各 package

這也是為什麼 Daisy 與 ML service 雖然不是典型 3GPP NF consumer，
仍被收回同一條 ownership 路徑。

### 8.3 為什麼 processor 不該同時扮演 assembler

processor 的職責是 procedure logic。

一旦 processor 開始：

- 建 client
- 決定 transport
- 偷讀 global config
- 自己掌握 shutdown context

它就不再只是 procedure layer，而會變成半個 assembler。

這會讓 unit test、lifecycle、ownership 與 config truth 都變得混亂。

### 8.4 為什麼 runtime truth 要盡量走 app seam，而不是全域 singleton

`factory.NwdafConfig` 並不是完全不能存在，而是：

> 在已經有 app seam 的 runtime path 裡，再去讀 package-global config，
> 會讓同一個執行路徑同時有兩個 authority。

這會導致：

1. 測試難以建立可信 seam
2. lifecycle 與 config source of truth 分裂
3. constructor 宣稱的依賴與執行時實際依賴不一致

### 8.5 為什麼 Priority 3 必須早於更大規模的 boundary 重構

因為沒有 seam-based tests，就沒有可信的結構重構。

這也是整個 `project_scan` 優先級設計的合理之處：

1. 先修 correctness
2. 先補 lifecycle 基礎
3. 先建立測試安全網
4. 再去動共享 boundary

這個順序不是保守，而是必要。

---

## 9. 目前可以怎麼理解「已完成」與「未完成」

### 9.1 已完成的是什麼

已完成的不是「整個 NWDAF 已完全與 free5GC 等價」。

已完成的是：

1. 一組比較可信的共享 runtime boundary
2. 一組比較一致的 handler / processor / consumer 分工
3. 一組比較可靠的測試 seam
4. 一組比較一致的 config 與 error contract 基礎
5. 一條較合理的 external client ownership 路徑

### 9.2 尚未完成的是什麼

尚未完成的比較偏向：

1. 對外 signaling 與 observability 的成熟度
2. standardized model / local extension 的治理規則
3. runtime config scope 重切
4. NWDAF 最終 free5GC integration level 的策略選擇
5. repository 邊界的最後整理

因此現在的狀態比較適合描述成：

> NWDAF 已經從「表面像 free5GC-style NF，但實際邊界不夠一致」
> 走到「共享 runtime boundary、測試 seam、consumer ownership、config contract、
> bounded lifecycle / ownership discipline 已大幅收斂」，但還沒有走到 full
> integration-level closure。

---

## 10. Git timeline 與重構脈絡

以下是目前最重要的 alignment commit 對照：

| 日期 | Commit | 主題 | 意義 |
| --- | --- | --- | --- |
| 2026-06-22 | `5e0e271` | `fix(subscription): reconcile updates and notifier lifecycle` | 關掉 Priority 1，並完成 Priority 2 的第一段 |
| 2026-06-23 | `fd5c878` | `refactor(test): align SBI seams and remove stale test assets` | Priority 3 phase 1 safety net |
| 2026-06-23 | `427ac74` | `test(sbi): normalize remaining consumer seams and processor mocks` | Priority 3 completion |
| 2026-06-23 | `cb2e066` | `refactor(sbi): align problem details responses` | Priority 5 Phase A |
| 2026-06-23 | `56afd7a` | `refactor(sbi): align request parsing with openapi deserialize` | Priority 5 covered Phase B |
| 2026-06-24 | `5abb292` | `fix(factory): harden runtime config contract` | Priority 8 |
| 2026-06-24 | `bbf42c3` | `refactor(app): rebuild NWDAF runtime app boundary` | Priority 4 Phase 1 |
| 2026-06-24 | `6569e02` | `refactor(app): tighten NWDAF boundary to free5gc seams` | Priority 4 Phase 2 |
| 2026-06-24 | `a912581` | `refactor(app): remove residual runtime config globals` | Priority 4 residual strictness cleanup |
| 2026-06-24 | `0768839` | `test(mockapp): align shared app test ownership` | Priority 3 strict follow-up |
| 2026-06-25 | `637235b` | `refactor(runtime): own external integration clients` | Priority 4 external-client round，第一段 |
| 2026-06-25 | `f026e77` | `refactor(consumer): own external integration clients` | Priority 4 external-client round，第二段與相鄰 shutdown 收斂 |
| 2026-06-26 | `17f9e17` | `fix: propagate caller-owned lifecycle contexts` | Priority 2 bounded remaining lifecycle closure |

這條 timeline 很能說明整體策略：

1. 先修錯
2. 再補測試
3. 再動共享邊界
4. 再處理更嚴格的 ownership 與 runtime truth
5. 再把 non-3GPP external client 也拉回同一條架構規則

---

## 11. 建議如何配合既有文件閱讀

如果之後要繼續深入，我建議按照這個順序看：

1. 先看本文件，建立整體圖像。
2. 再看
   `NWDAF Main Project Remediation Batches.md`
   了解目前 open / closed 狀態。
3. 對某條主線有疑問時，再回去看對應 plan：
   - correctness / notifier：Round 1
   - tests：Priority 3 系列
   - app boundary / ownership：Priority 4 系列
   - error contract：Priority 5
   - config：Priority 8
4. 最後回 `NWDAF/` 對照實際 code 與對應 commit。

如果只想抓整體方向，可用一句話記憶：

> 這一輪重構是在把 NWDAF 從「能運作的專案」
> 收斂成「邊界、ownership、測試、config、contract 比較一致的 free5GC-style NF」。

---

## 12. 本文件依據的資源

### 12.1 Issue / status / scan

- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project free5GC Alignment Scan Findings.md`
- `nwdaf-docs/docs/issues/free5gc/project_scan/NWDAF Main Project Remediation Batches.md`

### 12.2 Plans / summaries

- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Round 1 free5GC Alignment Implementation Plan.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 3 Test Refactor Detailed Plan.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 3 Test Refactor Phase 1 Summary.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 3 Test Refactor Completion Summary.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 3 Strict Test Ownership Alignment Plan.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 4 Rebuild One Real App Boundary Plan.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 4 Phase 2 Strict free5GC Boundary Alignment Plan.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 4 Residual Runtime Truth Cleanup Plan.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 4 External Client Ownership And Priority 2 Residual Lifecycle Hardening Plan.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 2 Remaining Lifecycle Completion Plan.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 5 SBI Error Contract Alignment Plan.md`
- `nwdaf-docs/docs/plans/free5gc-alignment/NWDAF Priority 8 Factory And Runtime Config Hardening Plan.md`

### 12.3 Workspace skill 與參考方法

- `free5gc-dev-skill/SKILL.md`
- `free5gc-dev-skill/references/exemplar-alignment.md`
- `free5gc-dev-skill/references/review-checklist.md`
- `free5gc-dev-skill/references/nf-architecture.md`
- `free5gc-dev-skill/references/sbi-development.md`
- `free5gc-dev-skill/references/testing.md`
- `free5gc-dev-skill/references/source-orientation.md`

### 12.4 目前 NWDAF 主程式代表性檔案

- `NWDAF/pkg/app/app.go`
- `NWDAF/pkg/service/init.go`
- `NWDAF/pkg/factory/config.go`
- `NWDAF/pkg/mockapp/`
- `NWDAF/internal/context/context.go`
- `NWDAF/internal/notifier/notifier.go`
- `NWDAF/internal/sbi/server.go`
- `NWDAF/internal/sbi/api_eventssubscription.go`
- `NWDAF/internal/sbi/api_collector.go`
- `NWDAF/internal/sbi/api_mlmodelprovision.go`
- `NWDAF/internal/sbi/api_daisy_callback.go`
- `NWDAF/internal/sbi/api_adrf_notify.go`
- `NWDAF/internal/sbi/consumer/consumer.go`
- `NWDAF/internal/sbi/consumer/context.go`
- `NWDAF/internal/sbi/processor/processor.go`
- `NWDAF/internal/anlf/anlf.go`
- `NWDAF/internal/anlf/monitor.go`
- `NWDAF/internal/anlf/model.go`
- `NWDAF/internal/mtlf/mtlf.go`
- `NWDAF/internal/mtlf/training.go`
- `NWDAF/internal/mtlf/adrf_retrieval.go`
- `NWDAF/internal/context/db_query.go`

### 12.5 local free5GC exemplar references

- `resources/references/free5gc-main/NFs/udr/pkg/app/app.go`
- `resources/references/free5gc-main/NFs/udr/pkg/service/init.go`
- `resources/references/free5gc-main/NFs/udr/internal/sbi/consumer/consumer.go`
- `resources/references/free5gc-main/NFs/udr/internal/util/problem_json.go`
- `resources/references/free5gc-main/NFs/udr/pkg/factory/config.go`
- `resources/references/free5gc-main/NFs/udm/pkg/app/app.go`
- `resources/references/free5gc-main/NFs/udm/pkg/mockapp/mock.go`
- `resources/references/free5gc-main/NFs/nrf/pkg/app/app.go`
- `resources/references/free5gc-main/NFs/nef/internal/sbi/consumer/consumer.go`
- `resources/references/free5gc-main/NFs/smf/pkg/service/mock.go`

### 12.6 Git evidence

本文件的時間線與完成狀態也對照了：

- `git -C NWDAF log --oneline`
- `git -C NWDAF show --stat` 對應 alignment commits
- `git -C nwdaf-docs log --oneline -- docs/issues/free5gc/project_scan docs/plans/free5gc-alignment`
