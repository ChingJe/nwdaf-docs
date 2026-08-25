# Explicit Flat and Hierarchical Orchestration Configuration Detailed Plan

日期：2026-08-25

狀態：Requirements Confirmed；implementation 尚未開始

相關文件：

- [Hierarchical NWDAF Federated Learning Implementation Plan](./Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)
- [Slice 3 Root Initiation, Static Topology and Assignment Detailed Plan](./Slice%203%20Root%20Initiation%2C%20Static%20Topology%20and%20Assignment%20Detailed%20Plan.md)
- [Slice 8 Multi-process E2E and Regression Closure Detailed Plan](./Slice%208%20Multi-process%20E2E%20and%20Regression%20Closure%20Detailed%20Plan.md)
- [NWDAF Development Policy](../../development_policy.md)

---

## 1. 目的

本計畫定義 PyMTLF 如何以顯式 configuration 區分 flat 與 hierarchical federated
learning orchestration，並為兩種 mode 提供可重現的 static participant topology。

目標是同時保留：

- 現有由 Model Provision、Model Monitor registration、accuracy degradation 與 NRF
  discovery 組成的 production-like flat FL 閉環；
- 現有由 Root static topology 建立 Branch／Leaf assignment 的 hierarchical FL；
- 不依賴 Model Provision／Model Monitor subscription chain 的 controlled flat FL；
- 不依賴 degradation 的 manual flat／hierarchical experiments。

這是既有 flat 與 hierarchical production flows 的 orchestration-selection extension，
不新增 NF type、不修改 Release 18 public SBI schema，也不把實驗 topology role 寫入 NRF
profile。

## 2. 現有行為與問題

目前 PyMTLF 以 `federated_learning.topology` 是否存在隱式決定 automatic retraining intent
的 owner：

- 沒有 topology 時由 flat `FLServerEngine` 接收 intent；
- 有 topology 時建立 `FLRootCoordinator`，並優先接收同一個 degradation intent；
- HFL private training-request API 也只在 topology 存在且 config enabled 時掛載。

現有 flat participant set 來自 Model Monitor active scopes。每個 scope 的 `consumerId` 再透過
NRF exact-instance discovery 對應到提供 `nnwdaf-mlmodeltraining`、符合 TAI、Analytics ID、
model interoperability 與 `FL_CLIENT` capability 的 NWDAF。

這個行為適合 production-like closed loop，但不適合只控制 topology shape 的比較實驗：

- deployment config 無法直接表達本次是 flat 或 hierarchical；
- flat FL 無法固定一組 participants 而完全略過 Model Provision／Monitor chain；
- topology source 與 trigger source 被間接綁在一起；
- flat 與 HFL 的 manual initiation 沒有共用 contract；
- 實驗紀錄無法僅從 resolved config 判斷實際 orchestration mode 與 participant source。

## 3. 術語與角色

Configuration、API、log 與文件使用下列語意：

| 概念 | Flat FL | Hierarchical FL |
| --- | --- | --- |
| Top-level coordinator | FL Server | Root |
| 中間 aggregation node | 不適用 | Branch |
| Data-owning trainer | FL Client | Leaf |
| 共用抽象名稱 | Coordinator／Participant／Node | Coordinator／Participant／Node |

固定規則：

- flat 文件與 runtime evidence 不把 Server 稱為 Root，也不把 Client 稱為 Leaf；
- Root／Branch／Leaf 只表示 hierarchical plan 中的 topology position；
- `FL_SERVER`、`FL_CLIENT`、`FL_SERVER_AND_CLIENT` 仍是 registered capability；
- Branch 在 process responsibility 上同時是 upper-tier Client 與 lower-tier Server；
- topology position 只存在於單次 training request／`planId`，不是永久 deployment role。

## 4. Target configuration contract

### 4.1 Orchestration selection

PyMTLF main config 新增明確的 orchestration block：

```yaml
federated_learning:
  orchestration:
    mode: flat
    participant_source: monitor_scopes
    topology_file: null
```

第一版合法值：

- `mode`: `flat` 或 `hierarchical`；
- `participant_source`: `monitor_scopes` 或 `static`；
- `topology_file`: static topology file path；相對路徑仍以 main config 所在目錄解析。

合法組合：

| Mode | Participant source | Topology file | 用途 |
| --- | --- | --- | --- |
| `flat` | `monitor_scopes` | 不使用 | 保留現有 production-like flat flow |
| `flat` | `static` | 必填 | Controlled flat experiment |
| `hierarchical` | `static` | 必填 | 現有 static HFL 與 controlled HFL experiment |

第一版不支援 `hierarchical + monitor_scopes` 自動分組。下列組合必須在 startup deterministic
failure：

- `hierarchical` 缺少 static topology；
- `flat + static` 缺少 static topology；
- `flat + monitor_scopes` 同時提供 topology file；
- 未知 mode、participant source、topology version 或 mode 不相符的 topology shape。

### 4.2 Trigger selection

Topology source 與 trigger source 必須分開設定。Target contract：

```yaml
federated_learning:
  training_trigger:
    degradation:
      enabled: true
    private_api:
      enabled: false
```

固定規則：

- `degradation.enabled: true` 保留既有 accuracy-policy trigger；
- `private_api.enabled: true` 掛載 manual training request／status API；
- 兩者可同時啟用，但仍受 single-active-training invariant 約束；
- controlled experiment 可關閉 degradation、只啟用 private API；
- 至少必須有一種 initiation source，否則 startup failure；
- static participant selection 不得因 config 中仍存在 Model Provision／Monitor settings，
  就偷偷改回 monitor-derived participants。

### 4.3 Backward compatibility

Implementation 必須先盤點所有 committed config、fixtures 與 downstream callers，再決定 migration
mechanism。第一個 slice 的最低相容性要求為：

- 所有 repository-owned configs 更新為 explicit orchestration configuration；
- 現有 flat production behavior 與 HFL behavior 不改變；
- 若暫時接受 legacy config，resolved mode 必須 deterministic，並輸出 deprecation warning；
- 不得長期同時維護兩套互相矛盾的 implicit／explicit selection semantics；
- implementation plan／review 必須明確記錄 legacy inference 的移除條件。

## 5. Static flat topology contract

Static flat topology 固定一個 FL Server 與一組 data-owning FL Clients。Server identity 來自
containing NWDAF context，不在 topology file 重複設定。

Target shape：

```yaml
version: 1

clients:
  - nf_instance_id: "11111111-1111-4111-8111-111111111111"
    scope:
      tracking_areas:
        - plmn_id: {mcc: "001", mnc: "01"}
          tac: "000001"
  - nf_instance_id: "22222222-2222-4222-8222-222222222222"
    scope:
      tracking_areas:
        - plmn_id: {mcc: "001", mnc: "01"}
          tac: "000002"
```

Detailed schema 可在 implementation review 中依既有 wire／internal models 調整命名，但必須滿足
以下 semantic requirements：

- participant identity 與 data scope 都是 explicit、stable、可 hash 的 experiment input；
- model family 決定 Analytics ID、model interoperability、event filter 與 target UE 的 authoritative
  source，不在每個 Client entry 任意覆寫；
- topology file 提供足以建立 training preparation 與 final validation 的 scope，不依賴既有
  Model Monitor registration 重建；
- Server 仍透過 NRF exact-instance resolution 驗證 Client identity、registered status、
  `nnwdaf-mlmodeltraining` service、`FL_CLIENT` eligibility 與 model interoperability；
- topology 不保存 endpoint、subscription ID、callback URI、`mlCorreId` 或其他 per-run identity；
- empty clients、duplicate identities、Server self-reference、invalid scope 與 NRF mismatch 都是
  deterministic failure；
- 第一版維持 `all` participant selection、`all` waiting 與 complete-required semantics。

## 6. Static hierarchical topology contract

既有 HFL topology shape 繼續使用 `branches`／`leaves`，不得為了和 flat 共用 schema 而將
flat Client 改稱 Leaf：

```yaml
version: 1

admission:
  mode: complete_required

branches:
  - nf_instance_id: "33333333-3333-4333-8333-333333333333"
    leaves:
      - nf_instance_id: "11111111-1111-4111-8111-111111111111"
      - nf_instance_id: "22222222-2222-4222-8222-222222222222"
```

Root、Branch、Leaf eligibility 與 exact NRF validation 維持既有 contract。新的 orchestration
block 取代「只因 topology 欄位存在就推斷 HFL」的 selection semantics，但不改變 assignment、
bundle、upper／lower process mapping、aggregation、validation、publication 或 cleanup contracts。

## 7. Manual initiation contract

Flat static experiment 不能依賴 degradation intent，因此需要和 HFL 共用的 config-gated manual
training request contract。

Target private API：

```http
POST /internal/v1/federated-learning/training-requests
Content-Type: application/json

{
  "requestId": "<canonical UUIDv4>",
  "modelFamilyId": "ue-communication-default"
}
```

固定語意：

- request 不允許覆寫 deployment 的 orchestration mode 或 topology file；
- request 只選擇已存在的 current model family，不能在 request 內注入 model URL；
- response 維持 asynchronous request／status resource semantics；
- single-active、idempotent retry、conflicting request、failure latch、restart 與 tombstone contract
  沿用既有 HFL private API；
- endpoint 仍是 PyMTLF-private management boundary，不經 Go、不宣稱為 Release 18 public SBI；
- 現有 HFL-specific private route 的保留、alias 或 migration 必須在 implementation 前定案，
  不得留下兩個可建立不同 top-level ownership 的入口。

## 8. Model lifecycle without Provision／Monitor chain

`flat + static + private_api` 必須能在沒有 active Model Provision／Monitor resources 時完成受控
training experiment，但這不代表整個 model lifecycle 可以省略語意：

- Root／Server 仍需從 seed／durable catalog 取得 current base model；
- preparation、round aggregation 與 participant final validation 仍須執行；
- static participants 不得被誤當成已存在的 model-consumer adoption scopes；
- 沒有 monitor-derived adoption scopes 時，publication 可以保存／commit accepted model，但
  completion evidence 必須明確記錄 `required_cutover_scopes = 0`；
- 不得宣稱已驗證 downstream consumer adoption 或 production cutover；
- 若 implementation 決定第一個 slice 只完成 candidate／validation 而延後 publication，必須使用
  不會弱化既有 `CANDIDATE_READY`／`FINAL_MODEL` 語意的 distinct terminal result，並先取得核准。

## 9. Controlled comparison profile

第一個比較目標固定相同四個 data-owning trainers：

```text
Flat
FL Server
├─ Client A1
├─ Client A2
├─ Client B1
└─ Client B2

Hierarchical
Root
├─ Branch A
│  ├─ Leaf A1
│  └─ Leaf A2
└─ Branch B
   ├─ Leaf B1
   └─ Leaf B2
```

比較規則：

- A1／A2／B1／B2 使用相同 identities、datasets、training windows 與 sample counts；
- Branch 不加入額外 local training samples，只聚合 assigned Leaves；
- seed model、local epochs、round count、optimizer、validation policy 與 random seeds 必須相同；
- flat 與 HFL 都使用相同 effective sample-count weighting；
- 若每個 upper round 只包含一次相同 Leaf updates，hierarchical weighted result 在數學上應與
  flat weighted result 等價；這個情境優先驗證 numerical equivalence 與 orchestration cost；
- latency、request count、artifact bytes、Root load、failure isolation、timeout 與 cleanup 是「只增加
  Branch」時的主要比較指標；
- 若要比較 model-quality 差異，必須另外標示 Branch local rounds、non-IID grouping、partial
  participation 或 strategy change，不能把混合變因歸因於 Branch 存在。

## 10. Ownership 與 implementation slices

### 10.1 Repository ownership

- PyMTLF：config models、validation、planners、manual initiation、flat static orchestration、status
  與 lifecycle ownership；
- Go NWDAF：維持 standard capability、Training SBI routing 與 containing-NWDAF context；預期不新增
  hierarchy public API 或 permanent role；
- PyAnLF：現有 production Model Provision／Monitor flow不得 regression；static manual experiment
  不要求 PyAnLF 建立相關 subscriptions；
- 文件：記錄 contract、implementation evidence 與 regression outcome。

### 10.2 建議 slices

1. Explicit config models、cross-field validation、resolved configuration identity 與 legacy migration；
2. Static flat topology parser、exact NRF resolution 與 flat process initiation；
3. Unified manual request／status API 與 HFL route migration；
4. Static flat final validation、publication／zero-cutover semantics 與 lifecycle closure；
5. Flat production、HFL、manual static flat 與 paired topology multi-process regressions。

每個 slice 都應保持 reviewable，不能把 config refactor、new flow、API migration 與完整 E2E
一次累積成單一未審查變更。

## 11. Acceptance criteria

1. Resolved config 明確輸出 `flat` 或 `hierarchical`，不再只靠 topology presence 判斷；
2. `hierarchical` 與 `flat + static` 缺少 topology 時 startup failure；
3. `flat + monitor_scopes` 完整保留既有 Model Provision／Monitor degradation flow；
4. `flat + static + private_api` 在沒有 Model Provision／Monitor resources 時可建立固定 participants；
5. Static flat 只訓練 topology 中列出的 exact four Clients，NRF 多出其他 eligible Clients 不改變結果；
6. Static HFL 只建立 configured Branch／Leaf assignment；
7. Flat logs／status 使用 Server／Client，HFL 使用 Root／Branch／Leaf；
8. 同一 config 不會讓一個 automatic intent 同時建立 flat 與 HFL processes；
9. Manual request 不可覆寫 config-selected mode／topology；
10. Static participant flow 不建立或要求 Model Provision／Monitor subscriptions；
11. Missing、duplicate、self、capability mismatch、service mismatch 與 scope mismatch cases deterministic fail；
12. Flat production、HFL degradation、HFL manual 與 restart／cleanup regressions全部維持；
13. Paired four-trainer flat／two-Branch HFL fixture 可證明 participant data與effective sample counts一致；
14. Run evidence 可記錄 resolved mode、participant source、trigger source、topology hash、exact identities
    與 component revisions。

## 12. 明確不在本次需求範圍

- dynamic HFL grouping 或 hierarchical `monitor_scopes` planner；
- arbitrary-depth hierarchy；
- 在單一 training request 中覆寫 deployment mode；
- permanent Root／Branch／Leaf NRF role；
- 新增非標準 public NWDAF SBI；
- hot reload topology；
- 同一 PyMTLF 同時執行 flat 與 hierarchical top-level training；
- 為了製造模型差異而偷偷改變 aggregation、round count 或 local training policy；
- testbed VM placement、Compose scaling 與實際 experiment run。

## 13. Handoff

本文件只固定 requirements 與 implementation boundary。實作者開始修改前仍須：

1. 對照當時 PyMTLF exact revision 與 working tree；
2. 盤點現有 config loaders、fixtures、private API callers 與 flat/HFL lifecycle tests；
3. 為第一個 slice 提出精確 files、contracts、tests 與 deferred items；
4. 取得使用者核准後才開始 production implementation。
