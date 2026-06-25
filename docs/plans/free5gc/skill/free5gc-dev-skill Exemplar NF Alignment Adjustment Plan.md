# free5GC Development Skill Exemplar NF Alignment Adjustment Plan

---

## 0. 目標

補強現有 `free5gc-dev-skill/`，讓 agent 在涉及實作、重構、測試設計、
boundary 調整、consumer/SBI 行為變更時，不只依賴課程筆記整理出的
workflow 與 checklist，還必須主動對照實際 free5GC NF 的實作慣例。

本計畫是既有
`nwdaf-docs/docs/plans/free5gc/skill/free5gc-dev-skill-plan.md`
的後續調整文件，聚焦 skill 行為約束與 reference routing 的補強，不重寫
整份原始 skill 設計。

---

## 1. 背景與問題

目前的 `free5gc-dev-skill` 已具備以下能力：

- 能先做 repo orientation，要求 agent 先看目標 repo 的 `README.md`、
  `Makefile`、`go.mod`、CI workflow
- 能依任務型態導向不同 reference，例如 SBI、OpenAPI、testing、
  config/lifecycle、UPF、review
- 能提醒 agent 以 target repo 的當前真實狀態優先，而不是套用記憶中的版本
  或課程內容

但在實際拿它支援 NWDAF free5GC alignment 重構時，仍暴露一個明確缺口：

1. skill 雖然是依據課程筆記與部份 free5GC 參考原始碼整理而成，
   但沒有把「實作前必須核對實際 free5GC NF 慣例」寫成明確流程
2. agent 若只讀 skill 現有 reference，容易停在抽象分層與一般性 checklist，
   卻沒有主動去確認某個具體模式在 free5GC 哪些 NF 中如何落地
3. 因此在實作過程中，仍需要額外提示 agent 去讀多個 free5GC NF，
   才能較穩定地對齊 `pkg/app`、`pkg/service`、consumer construction、
   lifecycle、test seam、notifier、generated-model usage 等實際慣例
4. 這代表 skill 的知識來源與 skill 的執行約束之間仍有落差

核心問題不是 skill 缺少「知道 free5GC 有哪些 NF」的背景資訊，而是缺少：

- 何時必須查 exemplar NF
- 要查哪些 exemplar NF
- 若本地沒有 free5GC reference tree 時要怎麼 fallback
- planning、implementation、review 三個階段各自要交代哪些 alignment evidence

---

## 2. 調整目標

本次調整希望 skill 達成以下結果：

1. 只要工作涉及 free5GC-style 實作，不再允許 agent 只靠課程整理版 workflow
   就開始設計或動手
2. agent 在 planning 前就要選出 1 到 3 個對應的 exemplar NF，
   並用它們作為設計參考
3. agent 在 implementation 過程中，仍要持續回看對應 exemplar，
   而不是只在一開始象徵性查看
4. skill 不能硬編碼某個使用者本地路徑，因此要定義可移植的 source discovery
   與 fallback 規則
5. agent 若宣稱「對齊 free5GC」，必須能說明是對齊哪些 NF、哪些檔案、
   哪些具體結構或測試樣式

---

## 3. 非目標

本次調整不處理以下事項：

- 不把所有 free5GC NF 的實作細節全文塞進 skill
- 不把本工作區 `resources/references/free5gc-main/` 的本地路徑硬寫進 skill
- 不要求所有任務都一定上網抓取最新 upstream free5GC
- 不把 skill 改成只能服務 NWDAF 工作區
- 不取代 target repository 自己的 source of truth

---

## 4. 新增的核心約束

### 4.1 何時必須做 exemplar alignment

以下任務型態應被視為必須做 exemplar alignment：

- 新增或修改 SBI endpoint、handler、processor、consumer
- 調整 `pkg/app`、`pkg/service`、`pkg/factory`、`internal/context`、
  `internal/sbi` 等主要邊界
- 調整 config、lifecycle、goroutine ownership、shutdown behavior
- 新增或重構 outbound SBI flow、NRF discovery、OAuth、callback、
  notifier、subscription 管理
- 設計或重構測試 seam、mock boundary、consumer tests、handler tests
- 處理 PFCP、SMF-UPF coupling、UPF data-plane 路徑

以下情境可不強制做 exemplar alignment，但仍可視需要進行：

- 純文件整理
- 不改行為的機械性更名
- 明確只做規格閱讀或 OpenAPI/YAML 比對，尚未進入實作設計

### 4.2 planning 前必做的 exemplar selection

agent 在開始設計或提出 implementation plan 前，應先完成：

1. 判斷這次變更屬於哪一類 free5GC 問題
2. 選出 1 到 3 個最相關的 exemplar NF
3. 確認要對照的檔案層級，例如：
   - `pkg/app/app.go`
   - `pkg/service/init.go`
   - `pkg/factory/config.go`
   - `internal/sbi/server.go`
   - `internal/sbi/api_*.go`
   - `internal/sbi/processor/*.go`
   - `internal/sbi/consumer/*.go`
   - 對應測試檔
4. 在 plan 或 reasoning 中明確交代：
   - 選了哪些 exemplar
   - 各自提供什麼參考價值
   - 打算沿用哪些 shape 或 testing pattern

### 4.3 implementation 中的持續對照

agent 在實作過程中，不應只在起手式看過 exemplar 一次就停止。當遇到以下問題
時，應回到 exemplar 重新核對：

- 新的 interface 或 constructor 該放在哪個 package
- consumer 是掛在 app、service，還是 local seam
- 某個測試應該用 `gock`、`gomock`、`httptest` 還是現有 local seam
- 某個 notifier、timer、scheduler 是否應掛入 app lifecycle
- callback、subscription、ProblemDetails、generated models 應如何沿用既有模式

### 4.4 review 與輸出中的 evidence 要求

當 agent 聲稱某個方案或修改是「對齊 free5GC」時，輸出應至少交代：

- 參考的 exemplar NF 名稱
- 參考的檔案或邊界
- 對齊的是哪一種慣例
- 哪些部分找不到直接對應，只能做推論

---

## 5. 建議的 exemplar selection matrix

skill 不應只籠統地說「去看 free5GC NFs」，而應給 agent 一個可操作的預設矩陣。
此外，matrix 應明確區分：

- `primary exemplar`：作為某類問題的首選 canonical shape
- `secondary exemplar`：補充另一個面向，例如測試樣式、callback、persistence、
  protocol-specific behavior

這樣 agent 才不會把所有任務都平均分配到多個 NF，導致參考面太散卻沒有主軸。

### 5.1 通用 control-plane 結構

建議 exemplar：

- `nrf`：primary exemplar
- `udm`：secondary exemplar

理由：

- `nrf` 的 `cmd -> pkg/service -> pkg/app -> internal/sbi -> processor -> consumer`
  骨架非常清楚，適合作為一般 control-plane NF 的 canonical shape
- `nrf` 對 app lifecycle、processor construction、consumer construction、SBI server
  的組裝方式相對乾淨，不容易混入過多 domain-specific noise
- `udm` 也保有相同 control-plane 骨架，但比 `nrf` 多了更實際的 generated-model、
  consumer、notification、測試 seam 使用情境

用途：

- `cmd` / `pkg/app` / `pkg/service` / `pkg/factory` / `internal/sbi`
  的典型 control-plane 形狀
- app/service/consumer 的共享 boundary
- 先用 `nrf` 看結構，再用 `udm` 補「這個骨架在較實際業務流程中如何落地」

### 5.2 協議較重或非純 SBI 導向的 control-plane

建議 exemplar：

- `amf`：primary exemplar
- `smf`：secondary exemplar

理由：

- `amf` 同時掛有 `internal/nas`、`internal/ngap`、`internal/gmm` 與 `internal/sbi`，
  明確說明某些 control-plane NF 不能只用 generic SBI 分層來理解
- `amf` 適合作為「協定層與 SBI 共存」的代表
- `smf` 則適合作為「SBI procedure 會跨到 PFCP、UPF selection、user plane state」
  的代表，補足另一種非純 SBI 的複雜度

用途：

- protocol-specific package 不應被 generic SBI abstraction 硬吃掉
- 跨越 `internal/sbi`、`internal/context`、`internal/pfcp` 等多層路徑的設計

### 5.3 generated models、consumer、outbound SBI tests

建議 exemplar：

- `udm`：primary exemplar
- `smf`：secondary exemplar

理由：

- `udm` 很適合用來看 `consumer.NewConsumer(...)` 的依附邊界，以及
  `gock`、`gomock`、`openapi.InterceptH2CClient()` 如何一起使用
- `udm` 的 consumer test 與 processor test 都能直接展示 mock app、
  generated models、outbound HTTP client interception 的實際組合方式
- `smf` 的價值在於把同樣測試樣式放到更複雜的跨 NF procedure 與 PFCP 關聯情境中

用途：

- `consumer.NewConsumer(...)` 的依附邊界
- `gock`、`gomock`、`openapi.InterceptH2CClient()` 等測試樣式
- generated models 與實際 processor/consumer flow 的結合方式

### 5.4 callback、policy/event、notification fan-out

建議 exemplar：

- `pcf`：primary exemplar
- `udr`：secondary exemplar
- `nef`：optional secondary exemplar

理由：

- `pcf` 有清楚的 HTTP callback route 與 policy/event notification handling，
  適合作為 callback 入口與 policy-driven notification 的主 exemplar
- `udr` 同時具備 subscription data change callback 與資料驅動 notification 路徑，
  適合作為 callback + persistence 耦合的補充 exemplar
- `nef` 比較適合作為 event/subscription-heavy public API exposure 的補充案例，
  但不適合作為所有 callback 問題的通用 primary exemplar

用途：

- callback 路徑
- policy/event notification
- fan-out 到外部 callback URI 的設計與 error path

### 5.5 subscription CRUD resource

建議 exemplar：

- `bsf`：primary exemplar
- `nef`：secondary exemplar

理由：

- `bsf` 的 subscription CRUD 路徑相對直接，適合觀察輕量 subscription resource
  的 request/response shape 與 context-side storage pattern
- `bsf` 更像 subscription resource 本身的 exemplar，而不是複雜 notifier lifecycle
  的 exemplar
- `nef` 可補 public API exposure 與 event exposure 相關的 subscription shape

用途：

- subscription create / replace / delete resource shape
- 輕量 subscription 管理與 in-memory context ownership

### 5.6 subscriber data、repository、persistence

建議 exemplar：

- `udr`：primary exemplar
- `udm`：secondary exemplar

理由：

- `udr` 的 processor 直接依附 database connector，且大量路徑是 document-oriented
  的資料存取與 subscription data 操作，最適合當 repository / persistence primary exemplar
- `udm` 雖然也處理 subscriber data，但更多時候是在 consumer-facing、
  generated-model-driven 的 control-plane 流程中使用這些資料
- 因此 `udm` 較適合作為「資料如何被上層 NF 使用」的 secondary exemplar，
  而不是 persistence primary exemplar

用途：

- subscriber data shape
- repository/document-oriented processor 路徑
- MongoDB 或資料存取邊界
- callback 與 persistence 如何交會

### 5.7 PFCP、N4、UPF data plane

建議 exemplar：

- `smf`：primary exemplar
- `upf`：secondary exemplar

理由：

- `smf` 是 N4 / PFCP control-plane side 的主要設計來源，負責 PFCP rule
  construction、UPF selection、以及與 SBI procedure 的串接
- `upf` 則是 user-plane / forwarding side 的主要設計來源，負責 PFCP server、
  GTP-U、forwarder、driver 與 shutdown ownership
- 兩者組合才能完整覆蓋 N4 與 data-plane 問題；只看 `smf` 或只看 `upf`
  都不夠

用途：

- SMF 的 PFCP rule construction
- UPF 的 PFCP/GTP-U/forwarder/kernel-facing 路徑
- control-plane 與 user-plane 的責任分界

---

## 6. Source Discovery 與 Fallback 規則

因 skill 需要可公開分享、可在不同環境使用，不能把
`resources/references/free5gc-main/` 這類工作區本地路徑寫死。

建議 skill 採用以下 discovery order：

1. 使用者明確提供的 free5GC source tree 或 reference repo
2. 目前工作區中可被辨識出的本地 free5GC checkout 或 mirror
3. 若本地沒有可用 reference，先與使用者確認下一步
4. 經使用者同意後，再採取以下其中一種方式：
   - 瀏覽 upstream free5GC repository
   - clone 一份官方 free5GC repository 到本地暫存位置
   - 使用使用者指定的其他 mirror

這裡的關鍵約束是：

- agent 不可在 source location 不明時直接假裝自己已完成對齊比對
- agent 若要改用 upstream browse 或 clone，必須明確說明這是 fallback
- agent 必須區分本地 snapshot 與 upstream 最新狀態，不可混為一談

---

## 7. 對 skill 檔案結構的建議修改

### 7.1 `SKILL.md`

建議在現有的 orientation 與 repository workflow 之後，新增一個簡短但明確的
主流程段落，例如：

- 若任務涉及實作、重構、測試設計、邊界調整或 lifecycle 變更，
  在 planning 前必須選擇對應的 exemplar NF
- 若無法定位可用的 free5GC reference source，先與使用者確認，必要時再改用
  upstream browse 或 clone
- planning 與 final output 中都要交代 exemplar 與 alignment evidence

此段應保持精簡，只負責下達約束與路由，不承載全部細節。

### 7.2 新增 `references/exemplar-alignment.md`

建議新增獨立 reference，內容包含：

- 哪些任務需要 exemplar alignment
- exemplar selection matrix
- source discovery order
- fallback 規則
- planning / implementation / review 各階段的 evidence 要求

這份 reference 應成為本次調整的主要承載點。

### 7.3 擴充 `references/source-orientation.md`

建議補充：

- 除了辨識 target repo，也要辨識可用的 free5GC reference source
- 說明本地 reference、使用者提供 source、upstream browse、clone 的優先順序
- 提醒 agent 不要把 course notes 當成唯一 alignment 依據

### 7.4 擴充 `references/nf-specific-routing.md`

建議補充：

- 將目前的「first packages to inspect」再往前補成「first exemplar NF to select」
- 依 area 給出預設 exemplar NF
- 指出某些問題需要跨多個 exemplar 對照，而不是只看單一 NF

### 7.5 擴充 `references/review-checklist.md`

建議新增檢查項：

- 若宣稱對齊 free5GC，是否交代 comparison basis
- 是否說明參考了哪些 exemplar NF 與哪些檔案
- 是否將 direct evidence 與推論分開

### 7.6 視需要擴充 `references/testing.md`

可補充：

- 在選擇 test seam 前，先看對應 exemplar NF 的測試樣式
- 對 outbound SBI、consumer、handler、app-boundary mock 的典型例子做 routing

---

## 8. 建議的實作順序

1. 先修改 `SKILL.md`，加入最小但清楚的主流程約束
2. 新增 `references/exemplar-alignment.md`
3. 補強 `source-orientation.md`
4. 補強 `nf-specific-routing.md`
5. 補強 `review-checklist.md`
6. 視內容量決定是否補強 `testing.md`
7. 用實際 prompt 做 forward-test，確認 agent 真的會在 planning 前去找 exemplar

此順序可讓 skill 先具備可見的高層約束，再逐步把細節下沉到 references。

---

## 9. 驗證方式

本次調整完成後，至少應做以下驗證：

### 9.1 正向驗證

- 給一個 control-plane SBI 實作任務，確認 agent 會先選 `nrf`、`udm`、
  `smf` 或其他合理 exemplar，再提出 plan
- 給一個 lifecycle 或 app-boundary 重構任務，確認 agent 會先找
  `pkg/app`、`pkg/service`、consumer construction 的實例
- 給一個 PFCP / UPF 類任務，確認 agent 不會只看 `internal/sbi`

### 9.2 fallback 驗證

- 在沒有本地 free5GC mirror 的環境描述下，確認 agent 會先與使用者溝通，
  而不是直接跳過 exemplar alignment
- 確認 agent 能說明何時改用 upstream browse，何時建議 clone

### 9.3 review 輸出驗證

- 讓 agent 做一次 code review 或 plan review，確認它會在輸出中明示：
  - 參考的 exemplar NF
  - 參考的檔案或邊界
  - direct evidence 與 inference 的界線

---

## 10. 預期效果

若本次調整落實，`free5gc-dev-skill` 將從「以課程整理為主的 free5GC workflow
guide」進一步提升為「會要求 agent 在設計與實作前主動對照真實 free5GC NF
慣例的 alignment workflow」。

這個變更的價值不在於把更多 free5GC 知識堆進 skill，而在於把原本依賴人工額外
提示的步驟，轉成 skill 內建且可檢查的執行約束。
