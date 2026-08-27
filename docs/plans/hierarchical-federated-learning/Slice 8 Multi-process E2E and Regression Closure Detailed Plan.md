# Slice 8 多程序端到端驗證與回歸收尾詳細計畫

日期：2026-08-22

狀態：Remediation Pending／Testbed Validation Pending；本機 E2E與前兩批 remediation已完成，
但 2026-08-27 late code review 確認hierarchy preparation assignment會對同一URL重複
HTTP GET。現行testbed run可保留為functional diagnostics，但不能作為最終可驗收
revision；修正、本機驗證、使用者review與commit後，必須重跑受影響的HFL testbed
scenario。在此之前本 Slice保持開放。

技術驗證完成日期：2026-08-25

上層計畫：

- [Hierarchical NWDAF Federated Learning Implementation Plan](./Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)

必要後續 remediation：

- [Flat FL Owned Artifact Self-download Remediation Detailed Plan](./Flat%20FL%20Owned%20Artifact%20Self-download%20Remediation%20Detailed%20Plan.md)
- 本文 §18.8：Hierarchy assignment duplicate GET late code-review finding

前置 slices：

- [Slice 0 Baseline Audit and Contract Freeze Detailed Plan](./Slice%200%20Baseline%20Audit%20and%20Contract%20Freeze%20Detailed%20Plan.md)
- [Slice 1 Hierarchy Bundle Contract and Artifact Primitives Detailed Plan](./Slice%201%20Hierarchy%20Bundle%20Contract%20and%20Artifact%20Primitives%20Detailed%20Plan.md)
- [Slice 2 Capability and Process-scoped Role Foundation Detailed Plan](./Slice%202%20Capability%20and%20Process-scoped%20Role%20Foundation%20Detailed%20Plan.md)
- [Slice 3 Root Initiation, Static Topology and Assignment Detailed Plan](./Slice%203%20Root%20Initiation%2C%20Static%20Topology%20and%20Assignment%20Detailed%20Plan.md)
- [Slice 4 End-to-end Preparation and Admission Detailed Plan](./Slice%204%20End-to-end%20Preparation%20and%20Admission%20Detailed%20Plan.md)
- [Slice 5 Hierarchical Rounds and Aggregation Detailed Plan](./Slice%205%20Hierarchical%20Rounds%20and%20Aggregation%20Detailed%20Plan.md)
- [Slice 6 Hierarchical Final Validation and Publication Detailed Plan](./Slice%206%20Hierarchical%20Final%20Validation%20and%20Publication%20Detailed%20Plan.md)
- [Slice 7 Lifecycle Closure and Fresh-state Restart Detailed Plan](./Slice%207%20Lifecycle%20Closure%20and%20Fresh-state%20Restart%20Detailed%20Plan.md)

開發規則：

- [NWDAF Development Policy](../../development_policy.md)

---

## 1. 目的與完成邊界

Slice 8 不新增 hierarchy algorithm、topology strategy、artifact role 或 public SBI。它把
Slice 1–7 已完成的 production owners 放入真實的多程序部署，關閉過去只能由 unit、mock 或
本機 component tests 支持的 integration claims。

本 Slice 的成功終點是：

1. 一組真實 NRF 與 ADRF，以及多個真實 Go NWDAF、PyMTLF 與必要的 PyAnLF processes，
   可從空白 runtime 啟動並完成 hierarchical FL；
2. smoke topology 與 two-Branch aggregation topology 都使用 NRF 中實際註冊的 Release 18
   NWDAF capabilities，不配置永久 Root、Branch 或 Leaf 角色；
3. 完整流程經過 assignment、preparation、多輪 FedProx、Branch 與 Root 兩層聚合、final
   validation、`FINAL_MODEL` publication、ADRF store、catalog commit 與 Model Provision
   cutover；
4. failure、timeout、single-active conflict 與兩個方向的 process-generation restart，至少
   各有一個真實 process 情境；
5. terminal 後可由 operator 明確建立新的 experiment，且不會有仍被占用的 Training
   capacity 或 scratch artifact 阻止下一次執行；
6. 既有 non-hierarchical distributed FL deployment 更新至現行 config contract 並再次通過；
7. runner 輸出可重現的 evidence summary，記錄 cross-process、real NRF、real ADRF 與實際
   transport profile。

不能只因所有 repository unit tests 通過就宣稱 E2E 完成，也不能用 fake NRF、fake ADRF、
直接呼叫 PyMTLF 內部 coordinator 或手工寫入最終 artifact 取代 production flow。

---

## 2. 既有流程基準與現況證據

### 2.1 既有部署流程

E2E 基準是 `nwdaf-resources/deployments/distributed_fl/` 的 isolated runner：

- runtime 產生暫時性的 configs、MongoDB databases、logs、workspaces 與 binaries；
- 啟動真實的團隊版本 NRF、ADRF，以及三組 NWDAF、PyMTLF 與 PyAnLF processes；
- 透過真實 NRF registration／discovery、ADRF storage／retrieval、standard-shaped Go relay、
  local fitting、final validation、publication 與 cutover 完成 flat FL；
- support-only fake SMF 只負責建立 deterministic data-collection callback correlation，不取代
  NRF、ADRF、NWDAF、PyMTLF、PyAnLF 或 FL procedure；
- 失敗時保留暫存執行目錄，成功時清除；不 checkout、修改或清理其他 repositories。

Slice 8 沿用這套 runner ownership 與 evidence style，不建立第二套部署框架。Hierarchy runner
放在獨立的 `deployments/hierarchical_fl/`，共用現有且穩定的 process 與暫存檔案系統 helpers；
不得把七節點 topology 硬塞進既有 flat runner。

### 2.2 既有流程延伸對照

| 基準階段 | 既有 flat deployment | Slice 8 處理方式 |
| --- | --- | --- |
| preflight | 檢查 branch、revision、binary、venv 與必要檔案 | 調整：加入 hierarchy commits、topology profiles、獨立 process roots 與現行 config validation |
| config generation | 三組 NWDAF 與互斥的舊 FL modes | hierarchy 另行產生；flat generator 遷移至 capability-based `runtime.mode: federated` |
| process startup | MongoDB → NRF／ADRF → backends → NWDAFs | 沿用並擴充成 1R／1–2B／2–4L，保留有時限的 readiness |
| NRF registration | 真實 NF profile persistence | 沿用並確認 FL Server、Server+Client、Client capabilities 與 services 完全符合預期 |
| data production | PyAnLF 產生固定資料並存入真實 ADRF | Leaves 沿用；Root 與 Branches 不建立假的 local dataset |
| trigger | flat Server 的 deterministic degradation | 調整：smoke 使用 Root PyMTLF private API；aggregation 使用 degradation；另驗證停用的 route |
| preparation | 一個 Server 對 flat Clients | 調整成 Root–Branch–Leaf 與固定 topology admission |
| rounds | flat FedAvg rounds | 調整成 Root 決定 epochs、Leaf FedProx、Branch lower aggregate、Root upper aggregate |
| final validation | Server 直接對 Clients | 調整成 Root–Branch–Leaf validation-only flow 與完整 subordinate evidence |
| publication | `FINAL_MODEL` → ADRF → catalog → PyAnLF cutover | 沿用並增加 hierarchy provenance assertions |
| failure／timeout | process 與 callback failure checks | 調整成必要 subordinate failure 使整個 hierarchy terminal，且不自動 retry |
| restart | component tests 中 replacement 以空白狀態啟動 | 調整成真實 PyMTLF-to-Go 與 Go-to-PyMTLF generation changes |
| cleanup | runner shutdown 與 production DELETE | 調整成第二次 experiment admission、workspace cleanup、stale URL／request assertions |
| regression | flat isolated 與 optional full-core profiles | 沿用；flat isolated runner 必須在 config migration 後再次通過 |

### 2.3 已確認的現況缺口

1. `nwdaf-resources` 尚無 hierarchical deployment directory、topology profile、runner 或
   summary schema；
2. 既有 flat runner 仍產生 `runtime.mode: fl_server|fl_client`，但現行 PyMTLF 只接受
   `local|federated`，並由 `server`／`client` sections 決定 engine presence；
3. 既有 flat runner 仍把 `epochs` 放在 Client training config，現行 contract 要求 Server
   透過 `client_training.epochs` 發布於每個 `ROUND_INPUT`；
4. 既有 runner 只有一個 Server 與兩個 Clients，無法證明 Branch lower aggregation、兩個
   Branch results 的 Root weighting 或 hierarchy validation provenance；
5. Slice 1–7 的 restart、cleanup 與 generation tests 都不是真實 multi-process supervisor run；
6. 既有 HTTP profile 使用 `oauth: false` 與 loopback endpoints，符合本 Slice 的本機
   cross-process acceptance 範圍。

### 2.4 free5GC 對齊證據

本 Slice 的 Go-facing deployment 與測試形狀使用：

- primary exemplar：workspace `nrf/` 與本機 free5GC NRF 的 `pkg/service/init.go`、
  `internal/sbi/server.go` 及 NF Management／Discovery processor boundary，用於確認
  registration、discovery、startup 與 shutdown 層級；
- secondary exemplar：本機 free5GC UDM 的 `internal/sbi/consumer/nrf_service.go` 與
  `nf_management_test.go`，只用於確認 outbound NRF consumer 與 test seam 慣例；
- deployment baseline：`nwdaf-resources/deployments/distributed_fl/`，它是本 workspace 的
  直接 runner 證據，不宣稱為 free5GC 通用慣例。

如果 E2E 暴露需要修改 NWDAF handler、processor、consumer、factory 或 service 的
production defect，實作前必須重新沿相同 exemplar boundary 確認 package placement 與 tests。
Support runner 維持 Python deployment tooling 結構，不套用 Go NF package layout。

### 2.5 各 repository 基準（2026-08-22）

| 儲存庫 | 分支 | 基準 HEAD | 初始處理方式 |
| --- | --- | --- | --- |
| `nwdaf-docs/` | `main` | `ffc888b739fb` | 計畫 owner |
| `nwdaf-resources/` | `feat/r18-federated-learning` | `8a7619ec9c74` | 主要實作 owner；實作前建立 hierarchy branch |
| `NWDAF/` | `feat/r18-hierarchical-federated-learning` | `3279891689dd` | 只驗證；除非 E2E 證實 production defect |
| `PyMTLF/` | `feat/r18-hierarchical-federated-learning` | `c7c66b9c0993` | 只驗證；除非 E2E 證實 production defect |
| `PyAnLF/` | `feat/r18-federated-learning` | `08798f15c369` | 現有 Provision／cutover dependency；只驗證 |
| `nrf/` | `feat/r18-nwdaf-discovery` | `0dd4024d4ab7` | 真實 registration／discovery dependency；只驗證 |
| `adrf/` | `feat/r18-federated-learning` | `905f0599f68f` | 真實 data／model persistence dependency；只驗證 |

以上 repositories 在 Slice 8 planning 開始前均為 clean。`components.yaml` 必須保存 required
branch、minimum ancestor 與 required files；不得要求 exact HEAD，以免合法的後續修正無法執行
E2E。

---

## 3. 已固定的產品與驗證語意

- Root、Branch、Leaf 都是標準 NWDAF；capability 由各自 NRF profile 決定，實際 topology
  position 只由 Root assignment 與 active `planId` 決定；
- Root 具備 FL Server capability，Branch 具備 FL Server 與 Client capabilities，Leaf 具備
  FL Client capability；
- 只有 Root PyMTLF 配置 static topology 與 strategy；Branch／Leaf 不配置 hierarchy role；
- Root 的 topology file 必須由主要 PyMTLF config 以相對路徑引用；
- 第一版 strategy 只有 FedProx、正數且有限的 `proximal_mu`、`all` selection、`all` waiting、
  `sample_weighted` aggregation 與 `complete_required` admission；
- Root 決定 round count 與 epochs；Branch 只傳遞 epochs，Leaf 不使用 local fallback；
- Branch 不執行 local fitting 或 Branch-local final validation；
- PyMTLF 直接提供 artifacts；Go 只傳遞 URLs 與 standard messages，不代理 bytes；
- candidate 在 final validation 前仍是 `ROUND_GLOBAL`；private status 不提供 `candidateUrl`；
- 任一必要 participant failure 使 experiment terminal；不執行 partial aggregate 或 automatic retry；
- restart 後不恢復 active hierarchy 或 unfinished publication；
- cleanup 由 standard subscription DELETE 與各 PyMTLF local ownership 完成，不由 Root 新增
  filesystem maintenance command；
- E2E runner 不以固定 sleep 作為唯一完成判斷；所有等待都必須有 bounded deadline、process
  liveness diagnostics 與 observable predicate。

---

## 4. 部署拓樸設定

### 4.1 固定的邏輯識別

Profiles 使用固定 canonical UUIDv4 NF instance IDs，使 topology files 與 summary 可以比較；
ports、暫存目錄、MongoDB names 與 process generations 每次執行時動態產生。

| 邏輯節點 | NF instance ID | NRF 能力 | PyMTLF 引擎 | PyAnLF |
| --- | --- | --- | --- | --- |
| Root | `10000000-0000-4000-8000-000000000001` | `FL_SERVER` | Server | 停用 |
| Branch 1 | `10000000-0000-4000-8000-000000000101` | `FL_SERVER_AND_CLIENT` | Server + Client | 停用 |
| Branch 2 | `10000000-0000-4000-8000-000000000102` | `FL_SERVER_AND_CLIENT` | Server + Client | 停用 |
| Leaf 1.1 | `10000000-0000-4000-8000-000000001101` | `FL_CLIENT` | Client | 啟用 |
| Leaf 1.2 | `10000000-0000-4000-8000-000000001102` | `FL_CLIENT` | Client | 啟用 |
| Leaf 2.1 | `10000000-0000-4000-8000-000000001201` | `FL_CLIENT` | Client | 啟用 |
| Leaf 2.2 | `10000000-0000-4000-8000-000000001202` | `FL_CLIENT` | Client | 啟用 |

每個 NWDAF 未來仍可在另一份 topology 中被指派不同位置；表格只描述本 Slice 的 deployment
profile，不是新增 production role config。

### 4.2 最小 smoke profile

```text
Root
└─ Branch 1
   ├─ Leaf 1.1
   └─ Leaf 1.2
```

目的：以最少的 processes 完成 manual-trigger full flow、Branch republish、two-Leaf lower
aggregate、hierarchy final validation、publication、cutover 與 terminal cleanup。

### 4.3 聚合驗收 profile

```text
Root
├─ Branch 1
│  ├─ Leaf 1.1
│  └─ Leaf 1.2
└─ Branch 2
   ├─ Leaf 2.1
   └─ Leaf 2.2
```

四個 Leaves 使用 deterministic 但不相等的 trainable sample counts。Fixture 不能只讓 raw
record 數量不同；runner 必須從 `ROUND_LOCAL` manifests 取得實際 training sample counts，確認：

```text
branch_effective_count = sum(leaf_training_sample_count)
root_total_count = sum(branch_effective_count)
```

Runner 並從 bundle 的 `model.npy` 獨立重算每個 Branch 與 Root 的 sample-weighted weights，
使用明確的 numeric tolerance 比對 production aggregate。只檢查 metadata 或只看到兩個
callbacks，不算完成 aggregation acceptance。

### 4.4 負向情境變體

負向情境使用 smoke topology 的最小變體，不建立第三套長期 profile：

- capability mismatch：設定的 Branch 或 Leaf 在 NRF 註冊的 FL capability 不符合 assignment；
- preparation failure：一個 Leaf 沒有可用 training data，但其他 Leaves 仍完成 terminal outcome；
- round timeout：在已 admitted 的 round 中停止一個 Leaf PyMTLF 回報，等待 bounded deadline；
- generation restart：只重啟指定的 Go 或 PyMTLF process，其他 processes 保持運行。

---

## 5. 執行期程序圖與設定產生

### 5.1 必須使用的真實元件

核心 E2E 啟動：

- 一個 MongoDB process，使用隔離的 NRF 與 ADRF databases；
- 一個團隊版本的 NRF binary；
- 一個團隊版本的 ADRF binary；
- smoke profile 有四組、aggregation profile 有七組 logical NWDAF processes；每組各有：
  - 一個真實 Go NWDAF；
  - 一個真實 PyMTLF；
  - 只有 Leaf 啟動一個真實 PyAnLF；
- 一個 support-only fake SMF，用來建立 deterministic collection subscription 與 callbacks。

Summary 必須標明 fake SMF 的存在。它只建立 Leaf training data，不參與 NRF discovery、
ML Model Training、hierarchy artifacts、aggregation、validation、publication 或 cleanup。

### 5.2 啟動順序

```text
preflight 與 config validation
  -> 將 NRF／NWDAF／ADRF binaries 建置到暫存目錄
  -> 啟動 MongoDB
  -> 啟動 NRF 並等待 SBI ready
  -> 啟動 ADRF 並等待真實 NRF registration
  -> 匯入 Root seed bundle
  -> 啟動 PyMTLF／PyAnLF processes
  -> 啟動各 backend 所屬的 Go NWDAFs
  -> 等待所有 Go registration、backend readiness 與 generation binding
  -> 驗證 NRF profiles 與 discovery queries
  -> 建立 Leaf analytics、Model Provision 與 Model Monitor resources
  -> 寫入 deterministic observations 並等待 ADRF records
  -> 執行指定情境
  -> 保存 summary 並執行 bounded production cleanup
  -> 依 ownership 的反向順序停止 processes
```

Readiness 必須同時確認 process 仍存活、HTTP status、backend `processInstanceId`、containing-Go
generation binding 與 NRF registration；只確認 TCP port 已開啟，不足以證明可以開始 training。

### 5.3 產生的 NWDAF configs

- 每個 Go process 都有獨立的 SBI、AnLF-private 與 MTLF-private bind／register endpoint；
- Root advertises `FL_SERVER` 與 Model Provision／Monitor services；
- Branch advertises `FL_SERVER_AND_CLIENT` 與 ML Model Training service；
- Leaf advertises `FL_CLIENT`、ML Model Training、Model Monitor，以及既有 PyAnLF
  collection／provision flow 所需的 services；
- 所有 profiles 向同一個真實 NRF 註冊，並使用各自固定且不重複的 `nfInstanceId`；
- backend endpoints 只指向 containing process 自己的 PyMTLF／PyAnLF；
- Root／Branch／Leaf 標籤只出現在 deployment file names、logs 與 summary，不加入 NRF schema。

### 5.4 產生的 PyMTLF configs

- 全部使用 `runtime.mode: federated`；
- Root 具有 `server`、`strategy`、`topology` 與選定的 trigger config；
- Branch 具有 `server` 與 `client`，但沒有 `strategy` 或 `topology`；
- Leaf 只有 `client`；
- Root 擁有 `server.client_training.epochs`；Branch 保留並向下傳遞該值，Leaves 沒有
  `training.epochs` 欄位；
- 每個 process 都有獨立的 `workspace_root`、`artifact_root`、`model_state.directory` 與
  `publication.directory`；
- allowed artifact origins 只包含該 profile 實際需要的 peer PyMTLF origins；
- topology path 相對於產生出的 Root main config 解析；
- timeouts 必須能穩定完成 E2E，且大於量測到的正常流程時間；失敗情境只覆寫受影響的 deadline。

### 5.5 Artifact 來源拓樸

每個 PyMTLF 使用不同 origin。Runner 必須記錄：

- Root upper assignment、round 與 candidate URLs；
- 每個 Branch lower assignment、round 與 validation republish URLs；
- 每個 Leaf result URLs；
- 每一次傳遞的 archive 與 weights digests。

Round acceptance 必須以 Leaf result 的 input artifact identity 連回 Branch lower
`ROUND_INPUT`，不能只比較 weights digest。Validation candidate 是 byte-identical republish，
因此另以 Root／Branch artifact access record 與 origin 證明 Leaf 從 Branch URL 下載。
Loopback 上的不同 origins 只作為 cross-process routing 證據。

---

## 6. 固定資料與模型設定

### 6.1 種子模型

Root 使用現有 PyMTLF seed import path 建立正式 base model 與 family
`ue-communication-default`。Runner 不得直接把 seed 複製到 Branches／Leaves 的 catalog；base
model 只能沿 production assignment／Training flow 到達下層。

### 6.2 Leaf 資料集

沿用既有 isolated runner 的 deterministic UE Communication observations 與 support SMF callback
correlation，使每個 Leaf PyAnLF 透過 containing Go 把 raw records 存入真實 ADRF。每個 Leaf
使用不同的 scope／TAI 與設定的 record count；ADRF retrieval、fetch instruction、dataset freeze
與 split 都由 production PyMTLF path 完成。

Runner 必須在 training 前保存：

- 預期的 Leaf scope 與 NF instance ID；
- ADRF 原始 record 數量；
- preparation 後實際的 training／validation sample counts；
- model／preprocessing contract digests。

不能用直接建立 `ROUND_LOCAL` bundle 或 mock trainer 的方式控制 weights。

### 6.3 可重現性

- 使用 CPU training；
- 使用固定 random seed；
- 固定 batch size、learning rate、validation ratio、epochs、round count 與 FedProx mu；
- 產生的 config 與 topology copies 必須放入失敗時保留的目錄；
- summary 記錄 repository HEADs、dirty-state marker、binary SHA-256 與 backend process
  generations；
- numeric assertions 使用已記錄的 tolerance；獨立聚合的 floating-point arrays 不要求完全相同的
  bytes。

---

## 7. 成功情境

### 7.1 手動觸發的最小成功情境

1. 啟用 Root private API；
2. 對 `/internal/v1/hierarchical-fl/training-requests` POST 一個 canonical UUIDv4 request，
   必須取得 `202` 與 `Location`；
3. 完全相同的 request 重送時，必須回到相同 request／plan；另一個不同的 concurrent request
   必須回 `409`，且不能建立第二個 plan；
4. 只透過回傳的 status resource 追蹤 preparation、rounds、final validation、publication 與
   cutover states；
5. response 不得包含 `candidateUrl`；candidate digest 只在 candidate 形成後出現；
6. downstream adoption 完成後，final state 到達 `COMPLETE`；
7. 兩個 Leaves 透過既有 Model Provision／Monitor cutover 啟用同一個新發布的 model ID；
8. terminal cleanup 後，candidate URL 回 `404`、plan workspaces 為空，第二個明確 request
   取得新的 `planId`。

### 7.2 效能劣化觸發的聚合情境

1. 停用 Root private API，route 必須回 `404`；Go 不得提供 trigger route；
2. 四個 Leaf analytics flows 建立穩定的 WAPE reports；
3. 一個 deterministic degradation intent 透過既有 Model Monitor／AccuracyPolicy production path
   到達 Root PyMTLF；
4. 只能建立一個 Root plan；
5. 兩個設定的 Branches 與四個 Leaves 全部 admitted；
6. 至少完成兩個設定的 training rounds；
7. 通過獨立的兩層 weight recomputation；
8. final validation evidence 涵蓋正確的四個 Leaves；
9. `FINAL_MODEL.hierarchy_validation` 保存 canonical two-Branch／four-Leaf evidence；
10. ADRF 與 Root catalog 指向同一個新 model，Leaf PyAnLF consumers 完成 cutover。

### 7.3 發布驗證項目

接受 publication 前，必須分別證明：

- final candidate 在 validation 前仍是 `ROUND_GLOBAL`；
- validation accepted 後才進入 `CANDIDATE_READY`；
- 新的 immutable `FINAL_MODEL` 使用新的 formal model ID；
- direct `participants` 是 Branches，且其 training sample counts 是 Leaves 的有效總和；
- direct validation summaries 是 Branches；
- `hierarchy_validation` 涵蓋正確的 Branch-to-Leaf admission，沒有重複或遺漏 NF；
- ADRF store response／record、catalog current record 與 final bundle digest 一致；
- private management status 提供 digest／state，但不提供 download URL；
- publication call 返回後 candidate scratch artifact 已移除，durable final artifact 則繼續透過
  既有 Model Provision lifecycle 提供下載。

---

## 8. 失敗、逾時、衝突與重啟情境

### 8.1 指令下發前的 capability mismatch

- 啟動所有真實 processes，但讓一個設定節點註冊不符合 assignment 的 capability；
- Root exact-instance discovery 必須在發布 assignment 或建立 upper Training resource 前失敗；
- status 為 `FAILED`，並提供 bounded cause；
- 不建立 Branch 或 Leaf hierarchy workspace；
- 修正 profile 後必須明確啟動新的 run；不得自動 topology retry。

### 8.2 下層 fan-out 後的 preparation failure

- 讓一個 Leaf 沒有 trainable records，其他 sibling Leaves 保持有效；
- Branch 必須收集所有 Leaf terminal outcomes 或等到 deadline，不能在第一個 failure 時返回；
- preparation result 必須把每個 assigned Leaf 分類且只出現一次；
- Root 拒絕 `complete_required`，執行 bounded cascade cleanup 並釋放 top-level admission；
- operator 修正資料後，新的明確 request 使用新 `planId` 並可成功完成。

### 8.3 訓練輪次逾時與中斷注入

- 等到 topology 進入 `ADMITTED` 且第一個 lower round 已可觀察；
- 暫停或終止一個 Leaf PyMTLF，使 production callback 無法送達；
- sibling results 仍收集到 bounded deadline；
- Branch 與 Root 不得聚合 partial set；
- 完整 experiment 到達 `FAILED`，且不自動 retry 或建立第二個 plan；
- runner 在 bounded teardown 中恢復或回收被注入故障的 process。

不能使用固定 sleep 作為注入觸發條件。只有 observable state、artifact 或 access event 證明目標
階段已到達後，才能開始注入。

如果停止 process 被現有 generation monitor 立即轉成 technical failure，該 run 只能算
subordinate failure，不能算 round-timeout acceptance。Runner 必須使用能讓已接受 round 保持
無回報直到 deadline 的 deterministic injection；如果現有 production boundary 無法做到，必須
將 timeout 情境標記為 blocked 並請求決策，不能改名或以 failure 結果代替。

### 8.4 真實程序世代重啟矩陣

代表性情境必須涵蓋兩個方向：

1. **PyMTLF → Go**：Root Go 保持運行並重啟 Root PyMTLF。新的 backend
   `processInstanceId` 使 Go 移除舊 generation 的 Training routes；新的 PyMTLF 以空白 hierarchy
   state／scratch workspace 啟動，且不恢復 publication；
2. **Go → PyMTLF**：Branch PyMTLF 保持運行並重啟該 Branch Go。新的 containing-Go
   `processInstanceId` 使 PyMTLF 關閉 admission、丟棄舊 Branch／Client／Server state，完成
   cleanup 後才重新綁定。

每個情境都必須確認：

- process generation 確實改變，固定的 NF instance ID 不變；
- 舊 request status、callback、PATCH 或 artifact 無法修改新 generation；
- 舊 experiment 依 fresh-state semantics terminal 或消失；
- 不恢復 journal-driven round、validation 或 publication action；
- cleanup 後，由 operator 發出的新 request 可使用新 `planId` 開始。

Leaf-specific 與所有 race permutations 繼續由 Slice 7 deterministic tests 涵蓋。除非真實代表性
run 與既有 invariants 衝突，Slice 8 不擴張 deployment matrix。

### 8.5 清理驗收

E2E 若要直接檢查 private Go maps，就必須增加 test-only production API，因此 cleanup
acceptance 改由下列 observable production effects 組成：

- parent terminal flow 傳送 standard DELETE，所有 peer responses 在 bounded time 內收斂；
- reader drain 後 candidate 與所有 owned scratch workspaces 都不存在；
- stale artifact GET 回 `404`；
- stale callback 或 resource operation 無法重建 process；
- 第二次完整 experiment 成功，證明 Root／Branch／Leaf training capacity 已釋放；
- 正常流程中的 processes 不需 forced kill 即可停止；
- forced kill、cleanup error 與保留的 remote resource 分開記錄，不能描述為 graceful cleanup
  success。

Graceful terminal path 必須完成 production DELETE。Crash path 不要求 replacement process 恢復
cleanup，也不宣稱 remote resources 立即消失；crash 前已建立的 remote subscriptions／artifacts
只依既有 expiry／garbage-collection policy 收斂，summary 必須分開記錄這個 verification level。

Exact in-memory route deletion 繼續由 Slice 7 direct tests 證明；runner 不得只為檢查
implementation internals 而新增 debug endpoint。

---

## 9. 驗證摘要契約

每次執行都必須在 cleanup 前寫入 `summary.json`。最低欄位如下：

```json
{
  "schemaVersion": 1,
  "profile": "smoke|aggregation",
  "scenario": "manual_success|degradation_success|capability_mismatch|preparation_failure|round_timeout|restart",
  "transport": {
    "sbiScheme": "http",
    "oauth": false,
    "hostModel": "single-host-distinct-origins"
  },
  "repositories": {},
  "processes": [],
  "topology": {},
  "request": {},
  "preparation": {},
  "rounds": [],
  "validation": {},
  "publication": {},
  "cleanup": {},
  "verificationLevels": {}
}
```

Summary 必須保存由 production responses、bundles、ADRF／catalog records 與 process
generations 取得的事實。Logs 可協助 diagnostics，但不能成為 model weights、topology identities、
publication 或 state outcomes 的唯一證據。

`verificationLevels` 至少記錄：

- repository config／unit checks；
- binary 建置；
- process 啟動；
- cross-process flow；
- 真實 NRF registration／discovery；
- 真實 ADRF storage／retrieval；
- 使用的 fake／support components。

任何未執行層級都必須記為 `not_run` 並附原因，不能默認為通過。

---

## 10. 各儲存庫影響範圍

### 10.1 `nwdaf-resources/`：主要實作 owner

實作前先依既有命名建立 `feat/r18-hierarchical-federated-learning` branch。

預期新增檔案：

```text
deployments/hierarchical_fl/
├─ README.md
├─ components.yaml
├─ checks/
│  ├─ preflight.py
│  └─ test_support.py
├─ configs/
│  └─ topology/
│     ├─ smoke.yaml
│     └─ aggregation.yaml
└─ scripts/
   ├─ run.py
   └─ support.py
```

`support.py` 可匯入 `portable_event_exposure/scripts/support.py` 的穩定 helpers，並重用 isolated
distributed FL fixture logic。不得複製整份舊 runner，也不得建立通用 orchestration framework。

同時依需要更新 `deployments/distributed_fl/scripts/run.py`、文件與 preflight：

- 產生現行 `runtime.mode: federated` configs；
- 以 `server`／`client` sections 表達 FL engine presence；
- 將 flat epochs 移到 `server.client_training.epochs`；
- 保留既有 flat flow 與 evidence assertions。

Hierarchy tooling 與 flat regression changes 位於同一個 `nwdaf-resources` repository，但在 review
與 commit description 中必須能依目錄清楚區分。

### 10.2 `NWDAF/`、`PyMTLF/`、`PyAnLF/`、`nrf/`、`adrf/`

初始處理方式都是只驗證。不能只因 E2E assertion 難以觀察，就預先授權 production change。

如果 deterministic real-process run 暴露 production defect：

1. 依本 Slice 的明確 acceptance criterion 分類；
2. 在實際 production owner 的 repository 增加會失敗的直接測試；
3. 實作最小修正；
4. 執行 mandatory review 與 targeted recheck；
5. 重跑受影響的 E2E 情境與該 repository 的完整必要檢查；
6. 保留 unstaged working-tree diff 供使用者審查，待 review 結果確認後再提出
   repository-separated commit proposal。

如果修正會改變 architecture、public／standard-shaped contract、ownership、dependency 或固定的
Slice scope，必須在修改 production 前停在 decision gate。

#### E2E 前置審視發現：PyAnLF static collection scope 不完整

Hierarchy isolated runner 沿用既有 flat portable fixture，使用 PyAnLF
`collection.resolution_mode: static`、configured SMF endpoints 與 configured group
memberships。Events Subscription 雖然包含 `networkArea`，但既有 static resolution 在建立
`CollectionKey` 時沒有保存該欄位；因此後續共用流程建立的 SMF Event Exposure subscription
與 training-data descriptor 都缺少原始 area scope。既有 full-core flat profile 使用
`resolution_mode: standard`，該路徑原本就會保存 `network_area_json`，不受此 defect 影響。

這個缺口不是 hierarchy algorithm 造成。Git history 顯示 static collection path 自
`88088c4` 起即未保存 area；`a35f689` 加入 network-aware standard resolution 與 descriptor
後，static path 沒有同步補齊。舊 portable flat E2E 的 seed model 使用空
`event_filter: {}`，support fake SMF 也不執行真實 TAI gating，因此「能取得資料」沒有證明
descriptor scope 正確，問題直到本 Slice 的 scope 檢查才被發現。

本 Slice 的最小 production remediation 是：

- static resolution 以 canonical JSON 保存每個 Events Subscription 的 `networkArea`；
- resource identity 同時包含 target 與 area，避免同一 UE 的不同 area scope 被錯誤共用；
- 共用 downstream path 將相同 area 放入 SMF subscription 與 training-data descriptor；
- PyAnLF direct test 驗證 configured group resolution 保留 area 與 group scope；
- 重跑 flat regression 與受影響的 hierarchy E2E。

結構上，static 與 standard resolution 目前各自建立 `CollectionKey`，只在後續 acquire、
ingestion、ADRF 與 descriptor lifecycle 共用實作。這種 duplicated construction 是本次漏欄位
的直接維護風險。較完整的未來方向是讓兩種 resolver 都先產生同一個 normalized resolved-target
representation，再由單一 owner 建立 `CollectionKey`。Mandatory initial review 必須記錄此項；
除非 review 證實目前最小修正仍無法維持本 Slice 的 scope correctness，否則不在 Slice 8
擴張成 PyAnLF resolver architecture refactor。

#### E2E 前置審視發現：PyMTLF hierarchy lifecycle 缺口

在啟動 real-process E2E 前，依本計畫的成功、失敗與多輪 acceptance criteria 對照現有
PyMTLF direct tests，發現下列會阻斷或弱化 E2E 證據的 production defects：

- Leaf 的 training-data scope 在同步建立 Training subscription 時檢查，使缺少資料的 Leaf
  無法先接受 resource、進入 asynchronous preparation，再以
  `NOT_AVAILABLE_ML_TRAIN` notification 形成 Branch 所需的完整 terminal partition；
- Root 與 Branch 完成 preparation evaluation 後直接操作或沒有更新 Server process state，
  缺少由 `PREPARATION_EVALUATING` 到 `READY` 的單一 admission owner；
- 多輪 hierarchy final candidate 的 `base_weights_digest` 代表最後一輪輸入，不能與實驗最初的
  validation baseline weights digest 比較，否則第二輪以後的合法 candidate 會被拒絕；
- hierarchy publication 的 ADRF `allowConsumerList` 只有 Root／direct participants，沒有納入
  validation tree 中實際需要下載新模型的 Leaves；
- Root coordinator 在 server process identity 尚未綁定前關閉時存在競態，worker 可能稍後才
  進入 preparation collection，導致 shutdown 等待無法結束。

本 Slice 採取最小修正：把 scope resolution 放入既有 asynchronous preparation worker、由
Server engine 提供 guarded admission transition、保留 final validation 對初始 baseline 的模型
評估但不把最後一輪 base 誤認為初始 weights、從 hierarchy validation tree 建立 publication
consumer 集合，並在進入 blocking preparation collection 前重新檢查 Root generation／shutdown。
每一項都由實際 owner repository 的 direct regression test 覆蓋；real-process E2E 仍須另行執行，
不能由這些 direct tests 取代。

### 10.3 `nwdaf-docs/`

- 維護本詳細計畫及後續 implementation record；
- 更新上層計畫狀態與 decision log；
- required local E2E、flat FL self-download remediation、required testbed validation 或使用者
  最終 evidence confirmation 尚未完成時，不得把 Slice 8 或整體計畫標為完成。

### 10.4 明確不受影響的 repositories

`udm/`、`udr/`、`smf-nwdaf-ext/` 與 `go-upf/` 不屬於 isolated hierarchy business profile 的
必要 components。其 full-core collection behavior 繼續由既有 distributed FL profiles 涵蓋；
除非直接證據顯示 hierarchy 依賴它們，Slice 8 不重新執行或複製該工作。

---

## 11. 執行期傳輸範圍

本 Slice 固定使用 HTTP/H2C、`oauth: false` 與 single-host distinct origins。這是本次 HFL 計畫
從一開始採用的 runtime baseline，不是待決策項目。

- 必須驗證真實 NRF／ADRF 與獨立 NWDAF／PyMTLF processes 間的完整 business flow；
- loopback 上不同 origins 只代表 cross-process；
- summary 記錄實際 transport config；其他 transport profile 不納入本 Slice。

---

## 12. 實作檢查點

### 檢查點 1：基準確認與 preflight

- 建立 working plan-conformance map；
- 固定並記錄目前 repository branches／HEADs 與 required files；
- 為舊 flat runner modes／Client epochs 增加會失敗的 config characterization；
- 在不啟動 processes 的情況下驗證 smoke／aggregation topology files 與所有 generated configs；
- 確認 generated transport 固定為 HTTP/H2C 與 `oauth: false`。

驗收：preflight 不修改 repositories、一次回報所有缺少的 dependencies，並區分 unsupported
environment 與 production failure。

### 檢查點 2：現行 flat runner 遷移

- 將 generated PyMTLF configs 遷移至 capability-based runtime；
- 保留 two-round flat aggregation、validation、publication 與 cutover assertions；
- 加入 hierarchy logic 前先執行 isolated flat E2E。

驗收：既有 non-hierarchical flow 在目前 repository HEADs 上通過，且不使用 Client-local epochs。

### 檢查點 3：Hierarchy 啟動器與 smoke profile

- 產生隔離的 configs／topology 並啟動真實 process graph；
- 驗證 NRF profiles、readiness 與 generation binding；
- 建立兩個 Leaf datasets；
- 完成 manual-trigger smoke flow；
- 產生第一份 `summary.json` 與 cleanup evidence。

### 檢查點 4：雙 Branch 聚合與發布

- 加入 aggregation profile 與 unequal sample fixtures；
- 獨立重算 lower／upper weights；
- 驗證 degradation trigger 與 private-disabled route；
- 驗證完整 hierarchy final validation evidence；
- 驗證 ADRF／catalog／PyAnLF cutover。

### 檢查點 5：失敗、逾時與重啟

- dispatch 前 capability mismatch；
- preparation failure 與完整 terminal partition；
- lower round timeout／network-loss injection；
- Root PyMTLF restart 與 Branch Go restart；
- stale interaction rejection 與 operator 啟動的新 experiment。

### 檢查點 6：審查、完整驗證與收尾

- 對完整 Slice diff 執行 mandatory initial review；
- 對納入範圍的 findings 執行 test-first remediation；
- 每次 remediation 後執行 targeted follow-up review；
- 執行 fresh-read policy／plan conformance gate；
- 執行完整 repository commands 與 required E2E scenarios；
- 完成 implementation record，並保留 IDE 可直接審查的 unstaged working-tree diff；
- 等待使用者確認 review 結果後，才更新 completion state 並提出
  repository-separated commit proposal。

---

## 13. 驗證矩陣

### 13.1 支援工具的確定性測試

- topology／config generator 產生正確 capabilities 與 engine sections；
- Root-only topology reference 從 main config directory 正確解析；
- NF IDs、ports、origins 與 workspace roots 不重複；
- unequal sample fixture 經過 dataset split 後仍維持不相等；
- independent sample-weighted recomputation 能抓到錯誤 counts 或 weights；
- summary schema 拒絕缺少 verification levels；
- 失敗時保留的目錄包含 configs、logs 與 partial summary；
- cleanup 不刪除 repositories，也不刪除本次暫存執行根目錄以外的任何路徑。

### 13.2 必須執行的 E2E 指令

實作期間可微調 exact CLI，但 committed interface 應維持如下：

```bash
PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/checks/preflight.py

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/scripts/run.py \
  --profile smoke --scenario manual-success

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/scripts/run.py \
  --profile aggregation --scenario degradation-success

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/scripts/run.py \
  --profile smoke --scenario capability-mismatch

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/scripts/run.py \
  --profile smoke --scenario preparation-failure

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/scripts/run.py \
  --profile smoke --scenario round-timeout

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/hierarchical_fl/scripts/run.py \
  --profile smoke --scenario restart-generations

PyMTLF/.venv/bin/python \
  nwdaf-resources/deployments/distributed_fl/scripts/run.py
```

Runner 可提供 `--suite core` 作為便利入口，但 completion evidence 必須分開保存每個情境結果；
只記錄一個 aggregate exit code 不足以完成驗收。

### 13.3 儲存庫驗證

如果只有 `nwdaf-resources` 變更：

```bash
# Run from the workspace root.
PyMTLF/.venv/bin/pytest -q \
  nwdaf-resources/deployments/hierarchical_fl/checks
PyMTLF/.venv/bin/ruff check \
  nwdaf-resources/deployments/hierarchical_fl \
  nwdaf-resources/deployments/distributed_fl/scripts/run.py
git -C nwdaf-resources diff --check
```

如果確認需要修改 production repositories，另外執行：

```bash
# PyMTLF
.venv/bin/pytest -q
.venv/bin/ruff check src tests

# NWDAF / nrf / adrf as affected
make test
make build
make lint

# PyAnLF if affected
.venv/bin/pytest -q
.venv/bin/ruff check src tests
```

Production repository commands 必須在各自 repository 執行。不能只因某個 unaffected repository
的 binary 在一次 E2E 中啟動，就宣稱該 repository 已完成完整驗證。

### 13.4 驗證層級

最終報告分開記錄：

1. config／support unit checks；
2. repository full tests 與 lint；
3. binary build；
4. process startup；
5. cross-process HFL flow；
6. real NRF registration／discovery；
7. real ADRF storage／retrieval；
8. Model Provision／PyAnLF cutover。

缺少 MongoDB 或其他本 Slice 必要的 environment dependency 時，記錄為 `not_run`；不能視為
production bug，也不能視為通過。

---

## 14. 審查與一致性檢查清單

### 基準與範圍

- [x] 既有 flat deployment 的每個階段都有明確處理方式
- [x] flat runner current-config regression 在 hierarchy claims 前完成
- [x] support fake SMF boundary 已明確記錄
- [x] 沒有 fake NRF／ADRF／NWDAF／PyMTLF 取代 production owners
- [x] 沒有新增 public SBI 或 hierarchy role config

### 拓樸與資料流

- [x] smoke 與 aggregation topology IDs／capabilities 完全符合 NRF profiles
- [x] 只有 Root 載入 topology 與 strategy
- [x] Branches 使用 combined engines，且不執行 local training
- [x] Leaves 經由 Branches 收到 Root 決定的 epochs
- [x] 每一次傳遞都記錄 artifact producer、URL origin、digest、recipient 與 cleanup owner
- [x] 兩層 weight acceptance 已獨立重算

### 生命週期

- [x] failure 與 timeout 使整個 experiment terminal，且不 partial aggregate 或 retry
- [x] 兩個 process-generation 方向都有真實 restart evidence
- [x] 舊 interaction 無法修改新 generation
- [x] terminal cleanup 後可完成第二個完整 experiment
- [x] 沒有新增 test-only production introspection endpoint

### 發布

- [x] validation accepted 後才進入 `CANDIDATE_READY`
- [x] `FINAL_MODEL` 與 ADRF／catalog identity 一致
- [x] hierarchy provenance 涵蓋正確 topology
- [x] publication reader drain 後移除 candidate
- [x] PyAnLF consumers 啟用已發布 model

### 驗證紀律

- [x] 每個情境都有獨立 summary 與 exit result
- [x] 記錄 exact commands、verification levels 與 skips
- [x] 完成 initial review、remediation 與 targeted follow-up
- [x] 完成 final fresh-read conformance gate
- [x] 完整修改已恢復為 IDE 可直接審查的 unstaged working-tree diff
- [x] 使用者已確認本機 E2E 批次的 review 結果
- [x] 第一批 repository-separated commit proposal 已獲核准並建立 commits
- [x] flat FL self-download remediation 已完成 production change、mandatory review 與本機驗證
- [x] 使用者已確認 flat FL self-download remediation working-tree review 結果
- [x] flat FL self-download remediation 第二批 commits 已獲核准並建立
- [ ] hierarchy preparation assignment duplicate GET 已完成test-first remediation、本機驗證與
  targeted follow-up review
- [ ] 使用者已確認本remediation的working-tree review結果，並核准精確commit proposal
- [ ] 包含本remediation的精確 revisions 已通過 required testbed validation
- [ ] 使用者已確認 testbed evidence 與最終結果

---

## 15. 延後工作分類

### Testbed 前必要 remediation

- flat FL Server 不得透過 remote downloader 重新下載自己持有的 artifact。第二輪以後的
  `ROUND_GLOBAL`、每輪聚合使用的本機 `ROUND_INPUT` 與 final validation candidate 都必須保留
  本機 `FLWorkspaceArtifact`／`ArtifactMetadata` handle；只有 peer-owned URL 才能經過
  origin-checked remote downloader。
- 修正必須分別加入上述三條 production paths 的 direct regression tests，重跑完整 PyMTLF
  tests 與 Ruff、flat isolated E2E，以及受影響的 hierarchy regression。
- 此 remediation 必須在第一批 commits 之後進行，另經使用者 review 與 commit proposal
  核准建立第二批 commits，才可部署到 testbed。
- 2026-08-27 late code review 確認`FLClientEngine._run_preparation()`對hierarchy
  assignment先呼叫generic `FLWorkspace.download()`與`inspect_artifact()`，辨識為
  hierarchy後刪除該暫存檔，再對同一URL呼叫`download_assignment()`。因此
  Root → Branch與Branch → Leaf每個logical assignment都會產生兩次HTTP GET；
  Flat preparation不進入該hierarchy branch，沒有這個重複。
- 此行為不是retry，也不是hierarchical FL的必要communication cost。根因是Slice 4在保留
  原flat generic ingress的同時，新增了會自行取回bytes的typed hierarchy downloader；當時
  tests將`download()`與`download_assignment()`分開mock，只驗證兩個helper都被呼叫，
  沒有對真實HTTP GET次數設上限。
- 修正邊界限於`PyMTLF/`：每assignment只能取回一次，但必須在同一份已接收bytes上
  保留URL、response header digest、archive digest、artifact role、recipient與`planId`的typed
  validation，並將通過驗證的同一份artifact移交給plan-owned workspace。不得單純刪除
  第二次呼叫而一併移除strict hierarchy validation；不修改Go NWDAF、public SBI、
  Release 18 schema或artifact wire contract。
- 必須先以deterministic transport test證明Branch與Leaf各自對assignment只發出一次真實
  HTTP GET，並保留flat單次download、digest／recipient／role／`planId`失敗、cleanup與
  ownership regressions。修正後重跑focused tests、PyMTLF full pytest、Ruff、local HFL
  smoke／aggregation scenario與受影響的testbed HFL scenario。

### 未來階段移交（future-phase handoff）

- 任意深度的 recursive hierarchy；
- dynamic topology 與 participant replacement；
- persistent active experiment recovery 與 remote resource reconciliation；
- hierarchy-specific topology 的 full-core UERANSIM／UPF data production。

### 選用強化（optional hardening）

- container／Kubernetes manifests；
- long-running soak、bandwidth shaping 與 large model throughput；
- packet capture 與 distributed tracing；
- hierarchy artifacts 的 cryptographic signatures；
- bounded process stop／restart scenarios 以外的 chaos framework。

### 整合驗證缺口（integration verification gap）

- 如果只使用 process pause／termination，真實 network partition 仍未驗證。

### 舊有清理（legacy cleanup）

- 一般化重構既有大型 `distributed_fl/scripts/run.py`；
- completed／failed publication journal 的 compaction；
- 重新命名仍具歷史正確性的舊 flat deployment descriptions。

---

## 16. 風險與控制

### 16.1 Runner 變成另一套平行編排框架

控制：只建立一個聚焦的 hierarchy runner、小型 support helpers 與兩個 topology files；重用
現有 temporary-process primitives，不建立 plugin 或 class hierarchy。

### 16.2 E2E 只靠 logs 判定通過

控制：state 來自 private status responses；weights 與 lineage 來自 bundles；durability 來自
ADRF／catalog；generations 來自 readiness／context。Logs 只用於 diagnostics。

### 16.3 訓練結果出現數值不穩定

控制：固定 CPU、seed、dataset、hyperparameters 與 tolerance；驗證 aggregation arithmetic，
performance gate 停用時不要求特定 model quality value。

### 16.4 逾時注入發生在錯誤階段

控制：只能在 observable status／artifact／access event 後注入；使用 bounded deadline 並在
failure 時保留 logs；不能只靠固定 sleep 觸發。

### 16.5 程序太多而隱藏第一個失敗

控制：先用 smoke profile 關閉 launcher；每次等待都檢查所有 processes 是否存活，失敗時輸出
有界的 log tails 並保留完整 runtime directory。

### 16.6 清理驗證需要 private debug API

控制：使用 observable stale URL／operation behavior、workspace state、第二次 experiment
capacity 與既有 direct unit tests；不新增 E2E-only API。

### 16.7 執行期範圍被默默擴張

控制：generated config 與 summary 固定反映第 11 節的 runtime baseline；其他 transport profile
必須另行規劃，不加入本 Slice completion gate。

---

## 17. 完成條件

Slice 8 只有在下列必要項目都有直接證據後才能標為完成：

1. 既有 flat isolated runner 使用現行 capability-based PyMTLF config 與 Server-owned epochs，
   並完整通過；
2. smoke profile 以真實 NRF、ADRF、NWDAF、PyMTLF 與 PyAnLF processes 完成 manual-trigger
   flow；
3. aggregation profile 以一個 Root、兩個 Branches 與四個 Leaves 完成 degradation-trigger flow；
4. 所有 nodes 以 standard NWDAF capabilities 註冊，topology role 只由 Root assignment 產生；
5. Root-only relative topology config、exact-instance discovery 與 re-resolution 都有 real NRF
   evidence；
6. preparation exact topology 與 `complete_required` admission 通過；
7. Root-authored epochs、Leaf FedProx 與至少兩個 rounds 通過；
8. runner 以實際 sample counts 獨立重算每個 Branch lower aggregate 與 Root upper aggregate；
9. Branch republish／Leaf download chain 在 assignment、round 與 validation stages 都有
   cross-process origin／artifact evidence；
10. final validation exact Branch／Leaf evidence 與 gate calculation 通過；
11. accepted candidate 建立新的 `FINAL_MODEL`，且 real ADRF record、catalog current record 與
    PyAnLF cutover 全部一致；
12. private-enabled manual request、idempotent retry、concurrent conflict 與 private-disabled `404`
    都有 black-box evidence；
13. 一個 capability mismatch 在 dispatch 前失敗，且沒有 lower resources；
14. 一個 preparation failure 回傳完整 Leaf outcome partition 並拒絕 admission；
15. 一個 round timeout／network loss 在不 partial aggregate 或 automatic retry 的情況下 terminal；
16. Root PyMTLF restart 與 Branch Go restart 涵蓋兩個 generation directions 與 fresh-state
    behavior；
17. stale interaction 無法重建或修改新 experiment；
18. success／failure cleanup 後 candidate／scratch artifacts 已移除，且第二個明確 experiment
    可以取得 capacity；
19. 每次執行都產生完整 `summary.json`，並記錄實際 verification levels 與 support fakes；
20. required support tests、repository checks、builds 與 E2E commands 通過；
21. E2E 發現的 production defect 已由 direct repository test 與 mandatory review
    關閉，或由已核准的 decision gate 明確阻擋；
22. final fresh-read plan-conformance map 沒有未完成的 normative item；
23. implementation record 列出 exact commands、results、預計 commit split、environment 與
    未執行層級；
24. 使用者已審查 IDE 中的 unstaged working-tree diff 並確認本機 E2E 批次的 review 結果；
25. flat FL self-download 的三條 production paths 已完成修正、direct regressions、完整 PyMTLF
    verification、flat isolated E2E 與受影響的 hierarchy regression；
26. 第一批與 flat remediation 第二批均以使用者核准的 repository-separated commits 固定可部署的
    精確 revisions；
27. hierarchy preparation assignment對每個remote recipient只發出一次HTTP GET，且不放寬
    typed validation與artifact ownership；本remediation已完成direct regressions、完整PyMTLF
    verification、local HFL E2E、targeted follow-up review、使用者review與核准的commit；
28. 包含第三批remediation的精確 revisions 已部署到 testbed，並通過部署前與使用者確認的
    required scenario matrix；
29. testbed record 已列出 VM topology、environment、各 repository revision、部署與驗證 commands、
    results、skips 及 failure remediation history，且使用者已確認最終 evidence。

第 15 項只有在 deadline-driven timeout 確實被觀察到時才算通過；generation／process technical
failure 不能替代它。Graceful cleanup 與 crash-time expiry／GC 也必須分別報告，不能合併成單一
cleanup pass。

本機 checkpoint 只能描述為「使用 HTTP/H2C 與 `oauth: false` 的真實 NRF／ADRF 本機多程序
HFL E2E 已通過」。只有 criteria 25–29 也完成後，才能描述 Slice 8 與本 HFL 第一版計畫完成。

---

## 18. Implementation record（2026-08-22–2026-08-25）

### 18.1 實作與修正

- `nwdaf-resources` working-tree changes 建立獨立 hierarchy runner、smoke／aggregation topology、
  preflight、deterministic support checks 與 flat runner config migration。
- PyAnLF working-tree changes 保留 static collection 的 canonical `networkArea`，並使 collection
  identity、configured SMF subscription 與 training-data descriptor 使用相同 scope。
- PyMTLF working-tree changes 關閉 asynchronous preparation、Server admission、多輪 candidate、
  ADRF consumer 與 shutdown generation fence 的前置缺口。
- 第一輪真實 smoke 暴露 registration poll 仍固定使用 flat NRF port。`nwdaf-resources`
  先以 `test_registration_wait_uses_supplied_nrf_uri` 穩定重現，再由對應
  working-tree changes 讓 hierarchy runner 使用當次配置的動態 NRF origin。
- 第二輪 smoke 證實兩個 Leaves 都已產生 `ROUND_LOCAL`，但 Branch Server aggregation 會把
  自己剛發布的 `ROUND_INPUT` 當成 peer URL 下載，因此被 peer-only origin allowlist 拒絕。
  PyMTLF 先修改 Server aggregation direct test 使其在修正前失敗，再由對應
  working-tree changes 讓 Root／Branch 直接使用自己持有的 immutable round-input artifact；
  participants 仍取得相同 URL，peer download 與 origin validation 未放寬。
- Final conformance 發現 summary 缺少 criterion 5 所需的 NRF re-resolution direct evidence。
  `test_nrf_reresolution_targets_restarted_node_exact_instance` 先重現缺口，對應
  working-tree changes 再於 Branch Go new generation 完成綁定與 fresh workspace 檢查後，
  保存真實 NRF profile 與 exact-instance discovery 結果。

### 18.2 真實多程序 E2E 證據

所有情境使用 Linux single-host distinct origins、CPU training、HTTP/H2C 與 `oauth: false`，
啟動真實 MongoDB、NRF、ADRF、Go NWDAF、PyMTLF 與 Leaf PyAnLF processes。Configured fake
SMF 只建立 deterministic collection callbacks；不參與 NRF discovery、Training、artifact、
aggregation、validation、publication 或 cleanup。

| 情境 | 結果與主要直接證據 | Summary |
|---|---|---|
| flat isolated regression | PASS；兩輪 flat training、final validation、ADRF publication 與 A/B cutover；final rerun model ID `1787594661767` | `/tmp/nwdaf-distributed-fl-wpyexf5i` |
| smoke `manual-success` | PASS；private API enabled、idempotent replay、concurrent `409`、兩輪、兩個 Leaves cutover、candidate `404`、第二個 plan完成 | `/tmp/nwdaf-hierarchical-fl-smoke-e9h2klxg/summary.json` |
| aggregation `degradation-success` | PASS；private route disabled `404`、兩個 Branches、四個 Leaves、兩輪 independent lower／upper recomputation、publication與四個 Leaves cutover | `/tmp/nwdaf-hierarchical-fl-aggregation-i2zlssda/summary.json` |
| `capability-mismatch` | PASS；Branch以錯誤`FL_CLIENT`註冊，Root在dispatch前以`DISCOVERY_FAILED`終止，lower artifacts／workspaces均為空 | `/tmp/nwdaf-hierarchical-fl-smoke-pqaf533v/summary.json` |
| `preparation-failure` | PASS；Leaf 1.1 prepared、Leaf 1.2精確回`NOT_AVAILABLE_ML_TRAIN`、無timeout誤分類，Root拒絕admission；修復資料後新plan完成 | `/tmp/nwdaf-hierarchical-fl-smoke-au92c8wa/summary.json` |
| `round-timeout` | PASS；在`ROUND_WAITING`與Leaf input artifact可觀察後才`SIGSTOP` Leaf 1.2，實際等待約19.16秒lower deadline；Leaf 1.1結果不作partial aggregate | `/tmp/nwdaf-hierarchical-fl-smoke-xl1yxy1r/summary.json` |
| `restart-generations` | PASS；Root PyMTLF與Branch Go兩個generation方向、fresh state、舊status／artifact `404`、Branch NRF re-resolution與新plan完成 | `/tmp/nwdaf-hierarchical-fl-smoke-46jy_u72/summary.json` |

Aggregation profile 的四個實際 trainable sample counts 為`4／9／14／18`；兩個 Branch effective
counts為`13／32`，Root total為`45`。兩個 rounds 都從bundle weights以`rtol=1e-5`、
`atol=1e-7`獨立重算通過。該情境記錄44次跨程序artifact transfers、580筆真實ADRF training
records，且final bundle、ADRF download、catalog current與四個PyAnLF cutover identity一致。

### 18.3 Exact commands 與 repository verification

| Repository／層級 | Command | Result |
|---|---|---|
| hierarchy preflight | `PyMTLF/.venv/bin/python nwdaf-resources/deployments/hierarchical_fl/checks/preflight.py` | PASS |
| hierarchy support | `PyMTLF/.venv/bin/pytest -q nwdaf-resources/deployments/hierarchical_fl/checks` | PASS：21 passed |
| hierarchy／flat runner lint | `PyMTLF/.venv/bin/ruff check nwdaf-resources/deployments/hierarchical_fl nwdaf-resources/deployments/distributed_fl/scripts/run.py` | PASS：All checks passed |
| PyMTLF | `.venv/bin/pytest -q` | PASS：475 passed、2 skipped、46 warnings |
| PyMTLF | `.venv/bin/ruff check src tests` | PASS：All checks passed |
| PyAnLF | `.venv/bin/pytest -q` | PASS：283 passed、2 skipped |
| PyAnLF | `.venv/bin/ruff check src tests` | PASS：All checks passed |
| affected repositories | `git diff --check` | PASS |

§13.2列出的七個hierarchy commands與flat isolated command均分開執行且exit code為`0`；上表不以
unit tests替代這些E2E。每個hierarchy summary都記錄repository revisions、binary SHA-256、
process generations、真實NRF／ADRF verification levels、support fake與任何`not_run`層級。

### 18.4 Verification levels 與未執行範圍

- `binaryBuild`、`processStartup`、`crossProcessFlow`、`realNrfRegistrationDiscovery`與適用情境的
  `realAdrfStorageRetrieval`／`modelProvisionCutover`均由真實process summary直接標為passed。
- Capability mismatch未進入資料與cutover階段，因此這些層級依契約標為`not_run`，不視為通過；
  其他負向情境亦只對實際到達的層級作pass宣稱。
- Round-timeout唯一forced kill是刻意注入並回收的Leaf PyMTLF；summary沒有unexpected forced
  kill，且未把technical process failure冒充deadline timeout。
- Crash-time remote reconciliation依fresh-state contract標為`not_run`；crash前remote resources
  仍只由既有expiry／garbage-collection policy收斂。
- 本機 E2E 未執行true cross-host deployment、UERANSIM／UPF data production與真實network
  partition；timeout情境使用可觀察階段後的process pause。這不取代後續testbed gate；testbed的
  VM topology、transport profile與required scenario matrix必須在部署前另行確認並記錄。
- OAuth／TLS依本Slice §11的固定HTTP/H2C、`oauth: false` baseline為計畫外項目，不屬後續
  testbed gate。

### 18.5 Review 與完成結論

Mandatory initial review、前置remediation、每次E2E finding的test-first修正與targeted follow-up
review均已完成。Final fresh-read依`development_policy.md`與本文件重新對照全部normative items；
completion criteria 1–23都有production path、direct test、獨立summary或exact command作為證據，
沒有silent deferral或未關閉的本機技術驗證項目。使用者已於2026-08-25確認criterion 24的
working-tree review結果。

因此目前是 `Remediation Pending／Testbed Validation Pending`；已完成的技術驗證範圍包含
使用HTTP/H2C與`oauth: false`的真實NRF／ADRF本機多程序HFL E2E，以及flat owned-artifact
self-download remediation的direct tests、full PyMTLF verification、flat isolated E2E與hierarchy
smoke regression，且使用者已確認第二批working-tree review與精確commit proposal。criteria
25–26已由PyMTLF `e9aa223`與nwdaf-docs `f2d0175`關閉；criterion 27因2026-08-27的
late code review finding重新開啟為remediation pending，criteria 28–29仍須以修正後的精確
revisions完成testbed validation、保存完整record並由使用者確認evidence。

### 18.6 技術驗證後補充紀錄：flat FL self-download

技術驗證後沿既有 flat FL 設計文件與 PyMTLF history 追查 owned artifact flow，確認
`FLServerEngine` 仍有三條本機 self-download production paths：第二輪以後以
`process.current_global_url` 重新下載上一輪 `ROUND_GLOBAL`、每輪聚合以剛發布的
`ROUND_INPUT` URL 重新下載輸入，以及 final validation 以 `candidate_url` 重新下載本機
candidate。Phase 3／4 原始計畫只要求跨 NWDAF 邊界以 immutable URL 傳遞 artifact，並未要求
producer 經 peer downloader 下載自己持有的檔案；此行為來自 URL-only internal state 與通用
downloader 的實作沿用。

目前 PyMTLF working-tree changes 已讓 hierarchical Root／Branch aggregation 使用本機 owned
round-input artifact，但未修改 flat flow。現行 flat isolated E2E 仍通過，且未發現模型或 lifecycle
錯誤；然而使用者已決定在第一批 commits 後、testbed 部署前把本項提升為必要 remediation。
修正必須讓 flat round transition、aggregation 與 final validation 持續保留本機 artifact handle，
只對 peer-owned URLs 執行 origin-checked download；同時加入三條 direct regressions，重跑 flat
isolated E2E 與受影響的 hierarchy regression，並重新評估 `_aggregate_round()` 的 optional URL
fallback 是否仍有合法 production caller。

上述 remediation 已完成：flat process state 保存current global artifact handle；aggregation必須接收
owned round-input artifact；final evaluator直接接收owned candidate metadata。三條本機
self-download均已移除，peer-owned round與validation artifacts仍使用origin-checked downloader。
direct regressions為`7 passed`，PyMTLF full suite為`479 passed, 2 skipped`，Ruff通過；flat isolated
E2E與hierarchy smoke manual-success regression也均通過。完整commands、skip reasons、暫存summary
paths與review結論記錄於remediation detailed plan §13；使用者已確認IDE review與精確proposal，
第二批commits為PyMTLF `e9aa223`與nwdaf-docs `f2d0175`，目前進入testbed validation gate。

### 18.7 Commit 與 testbed acceptance sequence

1. 使用者已完成第一批 working-tree review；第一批 repository-separated commits 已建立：
   PyMTLF `579b86e`、`3628068`，PyAnLF `6a4d94a`，nwdaf-resources `213d031`、`39ced28`，
   nwdaf-docs `f5c6186`、`4a5aaad`。
2. 第一批 commits 建立後完成 §18.6 的 flat FL self-download remediation；使用者已確認
   working-tree review與精確proposal，第二批commits為PyMTLF `e9aa223`與nwdaf-docs `f2d0175`。
3. 2026-08-27 late code review發現hierarchy preparation assignment duplicate GET，以本文
   §18.8為單一review ledger entry；完成test-first remediation、本機驗證、使用者review與精確
   commit proposal核准前，不得將現行revisions視為最終可驗收版本。
4. 使用者於2026-08-27因現在來不及完成修正後重測，決定先保存finding而不修改
   production code。現行testbed run可保留為functional diagnostics；修正後須以包含第三批
   commits的精確revisions重跑受影響的HFL scenario，並記錄scenario matrix、VM topology與transport
   profile。
5. 若 testbed 失敗，回到 working-tree remediation、重跑受影響的本機 regression、重新 review 與
   commit，再部署新 revisions；不得把失敗 revisions 或僅本機通過視為完成。
6. 只有 required testbed matrix 全部通過、record 完整且使用者確認 evidence 後，Slice 8 與本 HFL
   第一版計畫才可標為 `Completed`。

### 18.8 Late code review finding：hierarchy assignment duplicate GET

| 欄位 | 紀錄 |
| --- | --- |
| ID／狀態 | `S8-R5`／`Open — Deferred until the current testbed run completes` |
| Owner phase | Slice 8 testbed closure；原始production path由Slice 4引入 |
| Confirmed evidence | PyMTLF commit `096c401`的`FLClientEngine._run_preparation()`在hierarchy assignment path先執行`download()`／`inspect_artifact()`，再對同一URL執行`download_assignment()`；`download_assignment()`內部又經`_download_hierarchy()`發出HTTP GET。Root → Branch與Branch → Leaf都走此路徑，Flat preparation不走此分支。 |
| Consequence | HFL preparation的Root-facing與lower-tier artifact bytes被實作額外放大，增加不必要的I/O與failure surface；此overhead不得解釋為hierarchy固有代價，現行revision不適合用於正式communication comparison。 |
| Root cause | Slice 4為保留flat ingress behavior先沿用generic download，再加入具strict hierarchy checks的typed downloader；兩個helper各自擁有transport，tests又分開mock呼叫，因此沒有驗證每個logical assignment的實際GET次數。 |
| Required remediation | 將transport與typed validation拆開；單次取回artifact後，在同一份bytes上完成URL／header／archive digest、role、recipient與`planId`驗證，再將同一份artifact移交給plan-owned workspace。 |
| Verification | 新增真實HTTP transport count tests，證明Branch／Leaf每assignment各一次GET；保留flat與全部strict-validation／cleanup regressions；重跑focused tests、PyMTLF full pytest、Ruff、local HFL smoke／aggregation及受影響的testbed HFL scenario。 |
| Deferral | 2026-08-27使用者決定本次只保存finding，不修改production code，因為當前時間不足以完成必要重測。此決定不關閉finding，也不允許現行revision用於正式communication comparison或最終testbed acceptance。 |
| Closing commit | Pending；尚未修改production code |
