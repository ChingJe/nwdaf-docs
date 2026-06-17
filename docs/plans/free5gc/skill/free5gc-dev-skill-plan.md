# free5GC Development Skill 撰寫計畫

---

## 0. 目標

建立一套可公開分享、可獨立版本控管的 Codex skill，用於協助 AI agent 在開發 free5GC 或 free5GC 風格的 5G Core 元件時，穩定對齊 free5GC 官方專案的程式結構、開發流程、測試方式與除錯習慣。

本計畫只規劃 skill 的內容與製作流程，不直接建立 skill、不初始化獨立 git repo、不搬移課程內容。

預定 skill 名稱：

```text
free5gc-dev
```

預定用途：

- 審查 free5GC network function 或延伸元件是否符合 free5GC 慣例
- 指導 agent 在既有 free5GC repo 中修改 NF、SBI API、processor、consumer、factory、config、test
- 協助 agent 追蹤 free5GC API flow、NF 間互動、OpenAPI-generated model 的使用邊界
- 協助 agent 選擇合適的 build/test/debug 路徑
- 作為未來公開給其他 free5GC 開發者使用的通用 skill

非目標：

- 不把 3GPP specification 全文或特定 NF spec 內容內建進 skill
- 不把 `nwdaf-docs/docs/free5gc` 課程筆記完整複製進 skill
- 不把 NWDAF 專案特有設計放進 skill
- 不取代 free5GC 官方文件、官方 repo、3GPP OpenAPI 或 3GPP TS 文件

---

## 1. 已閱讀與對照的資料來源

### 1.1 課程筆記

已完整閱讀：

- `nwdaf-docs/docs/free5gc/NOTICE-free5gc-course-materials.md`
- `nwdaf-docs/docs/free5gc/1. Course Introduction.md`
- `nwdaf-docs/docs/free5gc/2. Foundations of free5GC.md`
- `nwdaf-docs/docs/free5gc/3. free5GC Architecture_ Control and Data Planes.md`
- `nwdaf-docs/docs/free5gc/4. Building and Running free5GC.md`
- `nwdaf-docs/docs/free5gc/5. Exploring the free5GC Source Code.md`
- `nwdaf-docs/docs/free5gc/6. free5GC Development Basics.md`
- `nwdaf-docs/docs/free5gc/7. Debugging free5GC.md`
- `nwdaf-docs/docs/free5gc/8. Summary and Next Steps.md`

課程可轉成 skill 的核心資訊：

- free5GC 是以 3GPP Release 15 and beyond 為目標的開源 5GC 參考實作
- control plane 與 data plane 應分開判斷，不能用同一套審查邏輯處理所有 NF
- control-plane NF 多數透過 SBI、HTTP/2、JSON 與 OpenAPI-generated models 互動
- UPF/data plane 涉及 PFCP、GTP-U、gtp5g、netlink、kernel-level packet processing
- NF 原始碼應依 `cmd`、`internal`、`pkg`、`factory`、`service`、`context`、`logger` 等責任分層理解
- SBI API 應沿著 `router -> handler(api_*.go) -> processor` 追蹤
- handler 負責 HTTP-level parsing/validation/response，processor 負責核心 business logic
- OpenAPI-generated models 與 interfaces 是 3GPP contract 的主要落點，手寫邏輯應圍繞它們實作
- error 應包裝並往上傳，在適合的邊界集中 log，避免低層重複 log
- 測試可用 `gock` mock 外部 SBI HTTP、`gomock` mock internal app/context、`httptest` 與 Gin context 執行 processor
- debug 應依層選工具：Go/SBI 用 logs 與 Delve；UPF/data plane 用 `gtp5g-tunnel`、`dmesg`、eBPF

### 1.2 授權來源

`NOTICE-free5gc-course-materials.md` 指出 `docs/free5gc/` 內容包含 Linux Foundation free5GC course materials 的摘要、改寫或翻譯，授權為 CC BY-SA 4.0。

對 skill 的影響：

- 若 skill 直接吸收課程文字，公開時需保留 attribution、license URL、變更聲明，並採用相容授權
- 更穩妥的做法是只吸收「我們自行歸納出的 workflow、checklist、reference routing」，避免大量重製課程文字
- skill repo 應包含 `NOTICE`，說明課程筆記曾作為設計參考
- skill repo 的 `LICENSE` 需在 Apache-2.0 便利性與 CC BY-SA 相容性之間明確選擇；若 reference 內含課程衍生內容，應傾向 CC BY-SA 4.0 或雙層授權策略

### 1.3 free5GC 原始碼對照

已對照 `resources/references/free5gc-main`：

- `resources/references/free5gc-main/README.md`
- `resources/references/free5gc-main/Makefile`
- `resources/references/free5gc-main/.github/workflows/test.yml`
- `resources/references/free5gc-main/NFs/nrf`
- `resources/references/free5gc-main/NFs/amf`
- `resources/references/free5gc-main/NFs/upf`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/processor/generate_auth_data_test.go`
- `resources/references/free5gc-main/NFs/udm/internal/sbi/consumer/nf_management_test.go`
- `resources/references/free5gc-main/NFs/amf/pkg/factory/config_test.go`

從原始碼確認的規範事實：

- 主 repo 以 `NFs/` 放置各 NF：`amf`、`ausf`、`bsf`、`nrf`、`nssf`、`pcf`、`smf`、`udm`、`udr`、`n3iwf`、`upf`、`chf`、`tngf`、`nef`
- 各 NF 是獨立 Go module，例：`NFs/nrf/go.mod` 使用 `module github.com/free5gc/nrf`
- `NFs/nrf` 符合課程描述的 `cmd`、`pkg/app`、`pkg/service`、`pkg/factory`、`internal/sbi`、`internal/context`、`internal/logger`
- `NFs/amf` 除通用結構外，另有 `internal/nas`、`internal/ngap`、`internal/gmm`，表示 AMF 類 NF 要額外看 N1/N2 專屬協議層
- `NFs/upf` 沒有 control-plane SBI 形狀，主要是 `internal/pfcp`、`internal/gtpv1`、`internal/forwarder`，表示 UPF 審查規則應獨立
- `Makefile` 的 build entrypoint 是 `make` 或 `make <nf>`，會進入 `NFs/<nf>/cmd` build，並注入 version/build metadata
- CI 使用 Go `1.24.5`，而 `NFs/nrf/go.mod` 標示 `go 1.24.0`；課程中出現的 Go `1.25.5` 不應被 skill 當成硬規則，應以目標 repo 的 `go.mod`、CI workflow、release 文件為準
- GitHub workflow 會 build `make` 並執行 `test_ci.sh`、`test_ci_ulcl.sh`，包含 OAuth 與非 OAuth 路徑
- UDM 測試實例證實課程描述的 `gock`、`openapi.InterceptH2CClient()`、`gomock`、`httptest` 測試模式

---

## 2. Skill 設計原則

依 `skill-creator` 指引，skill 應保持 progressive disclosure：

- `SKILL.md` 只放觸發後必要的工作流程與 resource routing
- 細節放在 `references/`
- 不建立額外 README、INSTALLATION_GUIDE、CHANGELOG 等雜訊文件
- 若需要腳本，放在 `scripts/`，但只有在能提供 deterministic 檢查時才加
- 之後真正建立 skill 時，應用 `skill-creator` 的 `init_skill.py` 初始化，而不是手刻 skeleton
- 完成後用 `quick_validate.py` 驗證

skill 必須讓 agent 先做 repo-aware 判斷：

1. 先辨識工作目標是修改既有 free5GC repo、新增 free5GC-style NF、還是審查 third-party extension
2. 先讀目標 repo 的 `README.md`、`Makefile`、`go.mod`、CI workflow、NF 目錄，不用課程中的版本數字硬套
3. 先判斷元件屬於 control plane、data plane、webconsole、test/ci，避免使用錯誤規則
4. 只在需要 spec 細節時導向 3GPP TS/OpenAPI/free5GC 官方 repo，不在 skill 內預置特定 NF spec

---

## 3. 預定 skill repo 位置與結構

使用者希望在本工作區建立新目錄，並以獨立 git 追蹤。實作階段建議建立：

```text
free5gc-dev-skill/
├── .git/
├── SKILL.md
├── agents/
│   └── openai.yaml
├── references/
│   ├── source-orientation.md
│   ├── nf-architecture.md
│   ├── sbi-development.md
│   ├── openapi-contract.md
│   ├── testing.md
│   ├── debugging.md
│   ├── review-checklist.md
│   └── publishing-and-licensing.md
├── scripts/
│   └── inspect_free5gc_repo.sh
├── NOTICE
└── LICENSE
```

說明：

- `SKILL.md`：精簡 workflow，描述何時讀哪些 reference
- `references/`：通用規範與 checklist，不放 NWDAF 專屬內容
- `scripts/inspect_free5gc_repo.sh`：可選，用於列出 NF 目錄、go.mod、Makefile、CI、SBI 結構；若後續發現 `rg/find` 就足夠，則不加 script
- `NOTICE`：記錄參考資料來源與 CC BY-SA 4.0 attribution
- `LICENSE`：公開前決定授權，避免混用課程衍生內容與自寫規範時不清楚

---

## 4. `SKILL.md` 預定內容

### 4.1 Frontmatter

預定：

```yaml
---
name: free5gc-dev
description: Develop, review, and debug free5GC or free5GC-style 5G Core components. Use when working on free5GC network functions, SBI APIs, OpenAPI-generated models, NF registration/discovery flows, Go control-plane code, UPF/data-plane code, build/test/CI workflows, or repository reviews that should align with free5GC project structure and conventions.
---
```

### 4.2 Body 大綱

`SKILL.md` 應控制在精簡長度，內容包含：

1. Start by identifying the task type
   - edit existing NF
   - add new SBI endpoint
   - implement consumer call
   - debug runtime issue
   - review code
   - add tests
   - work on UPF/data plane

2. Orient in the repository before editing
   - read `README.md`, `Makefile`, `go.mod`, workflow files
   - locate `NFs/<nf>` or equivalent
   - identify module path and Go version from repo, not from memory

3. Use source routing
   - control-plane NF: read `nf-architecture.md`, `sbi-development.md`
   - OpenAPI/model issue: read `openapi-contract.md`
   - tests: read `testing.md`
   - UPF/data plane: read `debugging.md` plus data-plane sections
   - review: read `review-checklist.md`

4. Development guardrails
   - keep handler/processor/consumer responsibilities separate
   - use generated models where available
   - preserve existing config/factory/logger patterns
   - prefer existing Makefile/CI/test entrypoints
   - document assumptions when 3GPP/free5GC behavior cannot be verified locally

5. Verification
   - run focused Go tests in the modified module
   - run `go test ./...` where feasible
   - run `make <nf>` or repo build target where relevant
   - do not require full integration test unless the change affects cross-NF behavior

---

## 5. References 設計

### 5.1 `source-orientation.md`

用途：agent 初次進入 free5GC repo 或第三方 free5GC-style repo 時閱讀。

內容：

- 如何辨識 main repo、NF submodule、webconsole、test/ci、deployment repo
- 必讀檔案：`README.md`、`Makefile`、`go.mod`、`.github/workflows/*.yml`
- 版本優先序：
  1. 目標 repo 的 `go.mod`
  2. 目標 repo CI workflow
  3. release/tag documentation
  4. 課程筆記或外部說明
- 對 free5GC main repo 的實際觀察：
  - `NFs/<nf>` independent modules
  - root `Makefile` builds from `NFs/<nf>/cmd`
  - CI runs build plus `test_ci.sh` integration-like tests

### 5.2 `nf-architecture.md`

用途：開發或審查 NF 結構時閱讀。

內容：

- control-plane NF 通用責任分層：
  - `cmd`: CLI、config path、signal handling、app start
  - `pkg/factory`: config schema、default path、validation、read config
  - `pkg/service`: app lifecycle、context init、server start/stop、metrics
  - `pkg/app`: interface boundary for service/processor/consumer tests
  - `internal/context`: runtime NF state
  - `internal/logger`: NF-specific logger
  - `internal/sbi`: service-based interface
  - `internal/util`: NF-private helpers
- AMF 類特殊層：
  - `internal/nas`
  - `internal/ngap`
  - `internal/gmm`
- UPF 類特殊層：
  - `internal/pfcp`
  - `internal/gtpv1`
  - `internal/forwarder`
- 審查問題：
  - 是否把 NF-private implementation 暴露到 `pkg`
  - 是否在 handler 寫入過多 business logic
  - 是否繞過 factory/context/service 直接散落全域狀態
  - 是否將 control-plane 與 data-plane concerns 混在一起

### 5.3 `sbi-development.md`

用途：新增或修改 SBI endpoint、handler、processor、consumer 時閱讀。

內容：

- route 定位：`internal/sbi/router.go` 與 `api_*.go`
- inbound path：HTTP request -> server/router -> handler -> processor -> response
- outbound path：processor/service -> consumer -> generated OpenAPI client or free5GC openapi package
- handler 責任：
  - parse request
  - validate HTTP-level input
  - convert to generated model
  - call processor
  - serialize response or problem details
- processor 責任：
  - implement NF procedure and state transition
  - use context/store/config/consumer
  - return spec-aligned models or problem details
- consumer 責任：
  - call peer NF service
  - respect NRF discovery/config/OAuth where existing code does
- 常見反模式：
  - processor 處理 raw JSON body
  - handler 直接修改 deep business state
  - consumer hardcode peer NF URI when config/discovery exists
  - 手寫 struct 取代既有 generated model

### 5.4 `openapi-contract.md`

用途：處理 3GPP SBI API、models、generated code、schema mismatch 時閱讀。

內容：

- OpenAPI 是 SBI contract 的主要來源
- free5GC 通常透過 `github.com/free5gc/openapi/models` 使用 generated models
- processor 應接收 generated model，而非任意 map 或 ad hoc struct
- 修改 API contract 前要確認：
  - 3GPP OpenAPI YAML
  - free5GC openapi repo/module version
  - target NF current generated model
  - existing caller/consumer code
- generated code 原則：
  - 不隨意手改 generated code
  - 若必須改，標註原因並考慮 regeneration path
- 對外公開的 skill 不內建特定 TS 29.xxx 內容，只指導 agent 去查正確來源

### 5.5 `testing.md`

用途：新增或修改測試時閱讀。

內容：

- test scope selection:
  - pure factory/config validation: table-driven Go tests
  - processor procedure: `httptest` + `gin.CreateTestContext`
  - external SBI dependency: `gock`
  - H2C client: `openapi.InterceptH2CClient()` and restore
  - internal app/context dependency: `gomock`
  - full repo flow: existing `test_ci.sh` when feasible
- free5GC examples to emulate:
  - `NFs/udm/internal/sbi/processor/generate_auth_data_test.go`
  - `NFs/udm/internal/sbi/consumer/nf_management_test.go`
  - `NFs/amf/pkg/factory/config_test.go`
- verification command guidance:
  - inside modified NF module: `go test ./...`
  - root repo build: `make <nf>` or `make`
  - integration changes: consider `test_ci.sh <case>` if environment supports it
- testing caveats:
  - MongoDB, kernel module, network namespace, and OAuth paths may require environment setup
  - do not claim full integration coverage if only unit tests ran

### 5.6 `debugging.md`

用途：debug free5GC runtime、SBI、UPF/data plane 問題時閱讀。

內容：

- Debug routing by layer:
  - SBI request issue: route -> handler -> processor -> consumer
  - NF startup issue: `cmd` -> `factory` -> `service.NewApp` -> context/server start
  - config issue: `pkg/factory` and `config/*.yaml`
  - cross-NF issue: NRF registration/discovery, OAuth, peer URI
  - UPF issue: PFCP, gtp5g rules, netlink, kernel logs
- Go tools:
  - logs
  - Delve breakpoints on `cmd/main.go`, handler, processor
- Data-plane tools:
  - `gtp5g-tunnel list pdr`
  - `dmesg -wT`
  - eBPF/ftrace for kernel function observation
- Output discipline:
  - separate observed evidence from inferred cause
  - record exact command and environment when diagnosing deployment problems

### 5.7 `review-checklist.md`

用途：user 要求 review 或掃描問題時閱讀。

內容：

- Repo orientation
- Module/version consistency
- NF directory responsibility
- SBI flow and separation of concerns
- OpenAPI/model alignment
- Config/factory/context usage
- Error handling/logging
- Test coverage
- Build/CI fit
- Data-plane specific checks
- License/publication checks for generated skill repo

### 5.8 `publishing-and-licensing.md`

用途：準備公開 skill repo 時閱讀。

內容：

- attribution requirements for course-derived planning
- avoid copying long course passages
- free5GC main repo license note: Apache 2.0
- course note license: CC BY-SA 4.0
- recommended public repo metadata:
  - `NOTICE`
  - `LICENSE`
  - source acknowledgements
  - clear statement that skill is unofficial and not endorsed by Linux Foundation/free5GC unless explicit permission exists

---

## 6. 可選 script 設計

### 6.1 `scripts/inspect_free5gc_repo.sh`

目的：讓 agent 快速取得 repo shape，而不是靠人工逐次重寫查詢。

預定輸出：

- root marker:
  - `README.md`
  - `Makefile`
  - `.github/workflows`
- NF list:
  - `NFs/*/go.mod`
  - module path
  - Go version
- per-NF shape:
  - `cmd`
  - `pkg/factory`
  - `pkg/service`
  - `pkg/app`
  - `internal/sbi`
  - `internal/sbi/processor`
  - `internal/sbi/consumer`
  - `internal/context`
  - `internal/logger`
- testing entrypoints:
  - `*_test.go`
  - root workflow test commands

是否實作條件：

- 如果 skill 實測時 agent 常漏讀 repo 結構，加入 script
- 如果 `rg --files` 與 `find` 足夠，先不加 script，避免維護成本

---

## 7. 實作階段計畫

### Phase 1: 建立 skill repo skeleton

動作：

- 在工作區建立 `free5gc-dev-skill/`
- 對該目錄執行 `git init`
- 使用 `skill-creator` 的 `init_skill.py` 建立 skill skeleton
- 建立必要 resource directories：`references`，視需要加 `scripts`
- 產生 `agents/openai.yaml`

完成條件：

- `SKILL.md` frontmatter 合法
- `agents/openai.yaml` 存在
- `quick_validate.py` 通過

### Phase 2: 撰寫 minimal usable skill

動作：

- 撰寫精簡 `SKILL.md`
- 撰寫 `source-orientation.md`
- 撰寫 `nf-architecture.md`
- 撰寫 `sbi-development.md`
- 撰寫 `review-checklist.md`

完成條件：

- skill 能引導 agent 掃描一個 free5GC-style repo
- skill 能引導 agent 判斷 handler/processor/consumer 分責問題
- skill 不包含大量課程原文

### Phase 3: 補足 OpenAPI、testing、debugging

動作：

- 撰寫 `openapi-contract.md`
- 撰寫 `testing.md`
- 撰寫 `debugging.md`
- 視需要撰寫 `inspect_free5gc_repo.sh`

完成條件：

- skill 能針對 API/model mismatch 給出查證路徑
- skill 能建議 free5GC 風格的 unit test/mock strategy
- skill 能把 UPF/data plane debug 與 SBI/Go debug 分開

### Phase 4: 授權與公開準備

動作：

- 撰寫 `NOTICE`
- 選擇 `LICENSE`
- 撰寫 `publishing-and-licensing.md`
- 標明 skill 為 unofficial helper
- 檢查是否有過量重製 Linux Foundation 課程文字

完成條件：

- attribution 清楚
- license strategy 清楚
- repo 可公開，不暴露本專案 NWDAF 私有內容

### Phase 5: Forward testing

動作：

- 用 skill 測試至少三種任務：
  1. Review a control-plane NF change
  2. Plan an SBI endpoint implementation
  3. Debug or review UPF/data-plane issue
- 若可用 subagent，使用獨立上下文測試 skill 是否足以引導 agent，不提供預期答案

完成條件：

- agent 能主動讀正確 references
- agent 不把 NWDAF 專屬規則套到 free5GC 通用開發
- agent 不把課程版本數字硬套到目標 repo
- agent 會根據 repo 現況決定測試命令

---

## 8. 驗證策略

### 8.1 靜態驗證

- `quick_validate.py <skill-folder>`
- 檢查 `SKILL.md` frontmatter 只有 `name` 與 `description`
- 檢查 skill name 使用 lowercase/hyphen
- 檢查 references 都由 `SKILL.md` 直接路由，不深層跳轉

### 8.2 內容驗證

用下列問題檢查 skill 是否達標：

- 是否能讓 agent 在改 code 前先辨識 repo shape?
- 是否能讓 agent 避免把 handler、processor、consumer 寫混?
- 是否能讓 agent 優先使用 generated OpenAPI models?
- 是否能讓 agent 區分 control-plane NF 與 UPF/data-plane?
- 是否能讓 agent 找到正確測試方式，而不是只說「請加測試」?
- 是否能讓 agent 在不確定 spec 時導向官方來源，而不是自行猜測?
- 是否避免重製課程內容?

### 8.3 實務驗證

可用本工作區做後續測試案例：

- 對目前 NWDAF 專案做 free5GC-style review，檢查 NF 結構、SBI flow、OpenAPI/model 對齊
- 對 `resources/references/free5gc-main/NFs/nrf` 做只讀 review，確認 skill 對正統 free5GC repo 不產生誤判
- 對 `resources/references/free5gc-main/NFs/upf` 做只讀 review，確認 skill 不用 control-plane SBI 規則硬套 UPF

---

## 9. 風險與決策

### 9.1 課程內容與官方 repo 版本可能不一致

已觀察到課程建置章節提到 Go `1.25.5`，但 `resources/references/free5gc-main/NFs/nrf/go.mod` 是 Go `1.24.0`，CI 使用 Go `1.24.5`。

決策：

- skill 不固定 Go 版本
- agent 必須從目標 repo 的 `go.mod`、CI workflow、release docs 取得版本

### 9.2 公開授權不清

課程筆記是 CC BY-SA 4.0，free5GC repo 是 Apache 2.0。若 skill reference 包含課程衍生表述，公開授權要謹慎。

決策：

- skill 以自寫 workflow/checklist 為主
- 保留 `NOTICE`
- 公開前檢查是否有長段課程文字

### 9.3 過度通用導致規範無力

如果 skill 只寫「遵守 free5GC 風格」，對 agent 幫助有限。

決策：

- 每個 reference 都要包含可操作檢查項
- review checklist 要能直接用於掃描 repo
- testing reference 要給具體 free5GC 測試樣式

### 9.4 過度依賴本工作區 snapshot

`resources/references/free5gc-main` 是本工作區的 snapshot，不一定等於未來最新 free5GC。

決策：

- skill 只把 snapshot 當作設計樣本
- agent 使用 skill 時必須重新讀目標 repo
- 若使用者要求最新官方行為，必須查官方 free5GC repo/docs

---

## 10. 下一步

待使用者確認本計畫後，下一輪才開始建立獨立 skill repo。

建議下一輪工作順序：

1. 確認 skill repo 目錄名稱：建議 `free5gc-dev-skill`
2. 確認公開授權策略：建議先用 CC BY-SA 4.0，或將課程衍生內容降到最低後改採 Apache-2.0
3. 使用 `skill-creator` 初始化 skill skeleton
4. 先完成 `SKILL.md`、`source-orientation.md`、`nf-architecture.md`、`sbi-development.md`、`review-checklist.md`
5. 跑 validation
6. 用本工作區做一次 review forward test
