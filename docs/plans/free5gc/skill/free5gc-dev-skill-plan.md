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

---

## 11. Coverage Gap Audit（2026-06-17）

### 11.1 背景

已建立第一版 `free5gc-dev-skill`，目前 skill repo 位置：

```text
free5gc-dev-skill/
├── SKILL.md
├── agents/openai.yaml
├── references/
│   ├── source-orientation.md
│   ├── nf-architecture.md
│   ├── sbi-development.md
│   ├── openapi-contract.md
│   ├── testing.md
│   ├── debugging.md
│   ├── review-checklist.md
│   └── publishing-and-licensing.md
├── scripts/inspect_free5gc_repo.sh
├── NOTICE
└── LICENSE
```

第一版已能覆蓋：

- repo orientation
- NF 目錄責任分層
- control-plane NF 與 UPF/data-plane 的基本區分
- SBI `server/router/handler/processor/consumer` flow
- OpenAPI generated model 的使用原則
- gock/gomock/httptest/H2C test pattern
- runtime debug 分層
- review checklist
- public publishing/licensing 注意事項

但這一版仍是「minimal usable skill」，有多處只停留在原則層，未完整轉成可執行 workflow。OpenAPI Generator 就是典型案例：目前 skill 只說「prefer regenerating from the correct OpenAPI source」，但沒有指導 agent 如何判斷何時要 regenerate、如何選 generator、如何驗證 output、如何避免混入 handwritten code。

### 11.2 Coverage Gap 分類

以下缺口按優先順序排列。

| 優先級 | 缺口 | 現況 | 風險 | 補強方向 |
|--------|------|------|------|----------|
| P0 | OpenAPI Generator workflow | 只涵蓋 contract 原則，缺少 CLI/Docker/JAR/generator option/驗證流程 | Agent 可能手改 generated code、手刻 models、或不知道何時 regenerate | 擴充 `openapi-contract.md`，必要時新增 `openapi-generation.md` |
| P0 | Config/factory lifecycle | 只在 architecture/debug/review 中點到 `pkg/factory` | Agent 修改 config 時可能漏掉 defaults、validation、sample config、CLI config path、tests | 新增 `config-and-lifecycle.md` |
| P0 | NRF registration/discovery/OAuth/cert flow | 只列為 checklist | Cross-NF 行為最容易壞，agent 缺少註冊、發現、OAuth、certificate 的追蹤方法 | 新增 `nf-registration-discovery.md` |
| P1 | Build/run/environment workflow | 目前只說讀 Makefile/CI，缺少 build/run prerequisites 與環境限制 | Agent 可能用錯 Go/MongoDB/gtp5g/kernel/NAT/UERANSIM 假設 | 新增 `build-run-environment.md` |
| P1 | UPF/data-plane implementation workflow | debug 有工具，architecture 有分類，但缺少 PDR/FAR/QER/URR/PFCP/netlink 變更檢查流程 | Agent 可能只跑 Go tests 就宣稱 data-plane 正確 | 新增或擴充 `data-plane-upf.md` |
| P1 | Concurrency and service lifecycle | 只提 goroutine lifecycle，缺少 context cancellation、WaitGroup、shutdown、server goroutine 管理規則 | Agent 可能新增 goroutine leak、shutdown hang、race | 新增 `concurrency-lifecycle.md` |
| P1 | Metrics/logging/observability | 只提 logging boundary 與 metrics server，缺少 free5GC metrics/context/logging conventions | Agent 可能漏 metrics、亂用 logger、或破壞 request metrics | 新增 `observability.md` 或併入 lifecycle |
| P2 | Interface/protocol-specific NF guidance | AMF/N3IWF/TNGF/SMF/CHF 僅概念區分 | Agent 可能把 AMF NAS/NGAP、SMF PFCP、CHF CDR 等都當一般 SBI | 新增 `nf-specific-routing.md` |
| P2 | CI/release/contribution workflow | 只讀 CI workflow，缺少 regression/integration/unit testing 與 PR contribution expectations | 若 skill 公開給貢獻者，對提 PR 的 guidance 不足 | 新增 `ci-release-contribution.md` |
| P2 | Webconsole / frontend/backend boundary | 目前幾乎未覆蓋 | Agent 處理 webconsole 時會套用 NF 規則 | 新增 webconsole routing，或在 `source-orientation.md` 標記不適用範圍 |
| P2 | Script capability | `inspect_free5gc_repo.sh` 只列 shape，無風險檢查或 machine-readable output | Forward testing 時仍需 agent 自己重組很多資訊 | 擴充 script summary 與 flags |
| P3 | Public packaging / install instructions | 有 licensing，但沒有 release/install layout 決策 | 開放給他人使用時不清楚要 clone 到哪裡、如何安裝到 Codex | 依 skill 發佈方式再補，不急於第一輪 |

### 11.3 課程章節對應缺口

課程中尚未完整轉換成 skill workflow 的內容：

- `5. Exploring the free5GC Source Code.md`
  - `Code Generation Tools`
  - `Code Generation Examples from 3GPP Specifications`
  - `Common Options Explained`
  - `OpenAPI Code Analysis`
  - `Integration with free5GC Architecture`
- `4. Building and Running free5GC.md`
  - VM/network prerequisites
  - Go/MongoDB/kernel/build dependencies
  - gtp5g install/build
  - Webconsole build
  - IP forwarding/NAT
  - UERANSIM connectivity validation
- `6. free5GC Development Basics.md`
  - goroutines/channels/select/WaitGroup 在 control plane 的使用守則
  - netlink family development
  - PDU session establishment flow from control plane to data plane
- `7. Debugging free5GC.md`
  - gtp5g-tunnel 的具體 PDR/FAR/QER 調查流程
  - eBPF attach workflow 與使用前提
  - Delve 常用命令與 breakpoint workflow
- `8. Summary and Next Steps.md`
  - version assignment
  - regression/integration/unit testing 的 release gate 語義
  - contribution workflow

---

## 12. 補充實作計畫

### Phase 6: OpenAPI Generator Workflow 補強（P0）

目標：讓 agent 能處理「需要從 3GPP OpenAPI YAML 產生或更新 Go code」的任務，而不是只知道 generated model 很重要。

動作：

- 擴充 `references/openapi-contract.md`，或新增 `references/openapi-generation.md`
- 在 `SKILL.md` 增加 routing：
  - API contract/model mismatch → `openapi-contract.md`
  - regenerate/update generated code → `openapi-generation.md`
- 補入 workflow：
  1. 判斷是否真的需要 regeneration
  2. 找出 target repo 使用的 `github.com/free5gc/openapi` 版本
  3. 找出對應 3GPP OpenAPI YAML 或 free5GC openapi source
  4. 選擇 Docker 或 JAR generator path
  5. 使用 target repo 指定或相容的 OpenAPI Generator version
  6. 使用 Go generator 與 package options
  7. 檢查 generated models/interfaces/client/server files
  8. 將 handwritten business logic 維持在 processor/service，不混進 generated code
  9. 跑 compile/test，並比較 generated diff 是否只包含預期 contract changes
- 補入常用 option 說明：
  - `-i`
  - `-g go`
  - `-o`
  - `packageName`
  - `withGoCodegenComment`
  - `enumClassPrefix`
  - `isGoSubmodule`

完成條件：

- Agent 能說明何時不該 regenerate
- Agent 能列出 regeneration 前必讀的 repo/spec/version 資訊
- Agent 能避免手改 generated code
- Agent 能在 PR/review 中辨識 generated diff 與 handwritten diff 是否混雜

### Phase 7: Config、Factory、Lifecycle 補強（P0）

目標：讓 agent 修改 NF config 或 startup/shutdown 行為時，能對齊 free5GC 的 `cmd -> factory -> service -> context` 路徑。

動作：

- 新增 `references/config-and-lifecycle.md`
- 補入 workflow：
  - CLI `--config` path 與 default config path
  - `pkg/factory` config struct、validation、default values
  - root `config/*cfg.yaml` 或 NF sample config 的同步
  - `service.NewApp` wiring
  - context initialization
  - server startup
  - graceful shutdown / signal / cancel / WaitGroup
  - metrics server lifecycle
- 在 `testing.md` 補 config test pattern：
  - table-driven validation
  - invalid/missing config cases
  - config default compatibility
- 在 `review-checklist.md` 增加 config-specific checklist

完成條件：

- Agent 修改 config 時會同步 schema、default、example yaml、validation、tests
- Agent 不會把 startup logic 塞進 handler/processor
- Agent 能指出 goroutine/shutdown 風險

### Phase 8: NRF Registration、Discovery、OAuth、Certificate 補強（P0）

目標：補足 cross-NF 行為中最容易出錯的 service registration/discovery 與授權流程。

動作：

- 新增 `references/nf-registration-discovery.md`
- 內容包含：
  - NF profile 與 `nfInstanceId`
  - NRF registration/deregistration lifecycle
  - discovery request/response trace
  - service consumer / producer roles
  - OAuth enabled/disabled branch
  - access token request/validation path
  - certificate path、root cert、NF cert、URI/ID matching
  - config 中 NRF URI、SBI register/bind address 差異
- 對照 `resources/references/free5gc-main/NFs/nrf/internal/sbi/api_accesstoken.go`
- 對照 `resources/references/free5gc-main/NFs/nrf/internal/sbi/processor/access_token.go`
- 在 `debugging.md` 增加 registration/discovery debug checklist

完成條件：

- Agent 能追蹤一個 NF 從 startup 到 NRF registration 的路徑
- Agent 能辨識 discovery/OAuth/cert 問題應看 config、context、consumer、NRF processor 還是 certificate
- Agent 不會只用 hardcoded peer URI 解決 cross-NF 問題

### Phase 9: Build、Run、Environment 補強（P1）

目標：讓 agent 在建置/執行/驗證 free5GC 時能正確處理環境依賴，不把 unit test、build、runtime、connectivity test 混為一談。

動作：

- 新增 `references/build-run-environment.md`
- 內容包含：
  - Go version 來源優先序：target `go.mod`/CI > course notes
  - MongoDB requirement and AVX caveat
  - build dependencies
  - root `make`, `make <nf>`, `make webconsole`
  - gtp5g build/install for UPF
  - `run.sh`
  - IP forwarding
  - NAT rule
  - firewall caveat
  - UERANSIM validation boundary
  - Webconsole default role as subscriber data management UI
- 明確標記：
  - local build pass 不等於 runtime pass
  - runtime pass 不等於 UE connectivity pass
  - UE connectivity pass 需要 network/kernel/simulator prerequisites

完成條件：

- Agent 會先讀 target repo 的 build docs 和 CI
- Agent 能列出未執行 integration/connectivity test 的環境原因
- Agent 不會把課程中的版本號當硬規則

### Phase 10: UPF/Data Plane Workflow 補強（P1）

目標：把目前 debug 中的 UPF 工具列表提升成可操作的 data-plane implementation/review workflow。

動作：

- 新增 `references/data-plane-upf.md`
- 內容包含：
  - control plane 到 data plane 的 PDU session path
  - SMF policy/PFCP instruction → UPF session/rule translation
  - PDR/FAR/QER/URR 的責任與關聯
  - Rtnetlink vs Generic Netlink
  - go-upf / gtp5g / kernel module boundary
  - runtime update path
  - data-plane review checklist
  - data-plane validation tiers：
    1. Go unit tests
    2. PFCP/session tests
    3. gtp5g rule inspection
    4. packet forwarding/connectivity
    5. kernel tracing
- 在 `SKILL.md` 將 UPF 任務 routing 到 `data-plane-upf.md`

完成條件：

- Agent 能說明 PDR/FAR/QER 的變更影響
- Agent 能區分 PFCP control issue、UPF translation issue、kernel rule issue、routing/NAT issue
- Agent 不會用 SBI handler/processor 規則套 UPF

### Phase 11: Concurrency、Lifecycle、Observability 補強（P1）

目標：補足 control-plane Go runtime 行為，避免 agent 寫出 goroutine leak、shutdown hang 或不可觀測的流程。

動作：

- 新增 `references/concurrency-lifecycle.md`
- 視內容量決定是否另開 `references/observability.md`
- 內容包含：
  - goroutine 啟動條件
  - context cancellation
  - channel/select timeout
  - WaitGroup add/done/wait discipline
  - panic recovery and fatal behavior
  - graceful shutdown
  - log boundary and severity
  - metrics server and request metrics
- 對照 free5GC `pkg/service/init.go` 類型的 lifecycle pattern

完成條件：

- Agent 新增 background worker 時會同時設計 cancellation 和 test
- Agent 修改 service lifecycle 時會檢查 shutdown path
- Agent review 時能指出 goroutine leak / duplicate logging / missing observability

### Phase 12: NF-Specific Routing 補強（P2）

目標：讓 skill 對 AMF、SMF、UPF、N3IWF、TNGF、CHF、UDM/UDR 等不同 NF 有基本 routing，不把所有任務都導向 generic SBI。

動作：

- 新增 `references/nf-specific-routing.md`
- 內容不寫完整 spec，只寫「遇到此 NF 應優先看哪些 package 與介面」：
  - AMF: NAS, NGAP, GMM, SCTP, N1/N2
  - SMF: PDU session, PFCP, user plane information, UPF selection
  - UPF: PFCP, GTP-U, forwarder, gtp5g
  - N3IWF/TNGF: non-3GPP access, IKE/IPsec/NGAP path
  - NRF: NF management, discovery, OAuth
  - UDM/UDR: subscription/auth data, MongoDB/data repository path
  - CHF: charging/CDR path
- 在 `source-orientation.md` 增加 NF-specific routing table

完成條件：

- Agent 能在任務開始時判斷「這不是 generic SBI 問題」
- Agent 能選擇正確 package 先讀

### Phase 13: CI、Release、Contribution 補強（P2）

目標：支援公開使用者用 skill 準備 PR 或 review CI failure。

動作：

- 新增 `references/ci-release-contribution.md`
- 內容包含：
  - unit / integration / regression testing 的語義
  - root GitHub workflow 的 build/test pattern
  - OAuth and non-OAuth integration test paths
  - release/version assignment：version、commit hash、build time injection
  - PR preparation checklist
  - issue/feature scope description
- 在 `review-checklist.md` 補 PR readiness checklist

完成條件：

- Agent 能把測試結果對應到 release gate 語義
- Agent 能在 PR summary 中清楚說明測試範圍和未覆蓋風險

### Phase 14: Script Enhancement（P2）

目標：讓 `inspect_free5gc_repo.sh` 不只列檔，也能輔助找出缺口。

動作：

- 新增 flags：
  - `--json` 或 `--summary`
  - `--nf <name>`
  - `--checks`
- 增加檢查：
  - NF 是否有 `cmd/pkg/internal`
  - control-plane NF 是否有 `internal/sbi/processor`
  - `go.mod` Go version 與 CI go-version 是否不一致
  - root Makefile 是否支援目標 NF
  - test files 是否存在
  - UPF 是否有 data-plane packages
- 保持 script 為輔助工具，不讓它取代人工閱讀

完成條件：

- Script 可作為 review 起始輸入
- Script failure 不會阻斷 agent 手動分析

---

## 13. 更新後的建議實作順序

短期應先補 P0：

1. Phase 6: OpenAPI Generator Workflow
2. Phase 7: Config、Factory、Lifecycle
3. Phase 8: NRF Registration、Discovery、OAuth、Certificate

接著補 P1：

4. Phase 9: Build、Run、Environment
5. Phase 10: UPF/Data Plane Workflow
6. Phase 11: Concurrency、Lifecycle、Observability

最後補 P2：

7. Phase 12: NF-Specific Routing
8. Phase 13: CI、Release、Contribution
9. Phase 14: Script Enhancement

每個 phase 完成後都應：

- 更新 `SKILL.md` routing
- 保持 reference 一層直連，不做深層 reference chain
- 跑 `quick_validate.py`
- 用 `resources/references/free5gc-main` 做 smoke test
- 至少做一個 forward-test prompt，確認 agent 會讀到新增 reference
