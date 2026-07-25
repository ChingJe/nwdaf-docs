# Phase 7：Deployment Integration And Legacy Closure

Date: 2026-07-25

Status: Completed and verified

Parent plan:

- `MTLF Backend Transition Plan.md`

Previous phase:

- `Phase 6 Local Training And Model Update Reprovision.md`

---

## 1. 目的

Phase 1 至 Phase 6 已經完成 backend responsibility、analytics subscription、model monitoring、
dataset retrieval、local retraining 與 updated model reprovision 的主要 vertical slices。本階段要做的不是再新增
一套 MTLF business flow，而是完成兩件收尾工作：

1. 把目前只在 fixture／局部 process 測試中驗證的流程，擴展為真實 ADRF integration，以及不依賴
   gtp5g、UE registration或active PDU session即可執行的portable application E2E。這條E2E使用
   team SMF的顯式static session resolution mode與team go-upf的standalone dataset replay entrypoint，
   但Nsmf、Nupf及callback wire contract仍維持Release 18標準形式。
2. 移除已被 PyAnLF／PyMTLF 取代的 Go legacy runtime，依既有 AnLF backend transition所確立的
   domain package邊界整理`internal/anlf`與`internal/mtlf`，並集中處理目前散落的手寫3GPP wire types。

本階段完成後：

- Go NWDAF 仍是唯一標準 NF 與 external SBI owner。
- PyAnLF／PyMTLF 仍是同一 NWDAF 的 internal backends，不註冊 NRF。
- Go 不再保有 Daisy、Go-side retraining policy、dataset retrieval coordinator 或 model training runtime。
- AnLF與MTLF各自保有明確的Go-side auxiliary HTTP edge；共用能力只下沉到`internal/backend`或
  `internal/sbi/consumer`，不建立neutral mega-gateway。
- fixed `externalMtlf` selection/config與舊runtime wiring會移除；標準ML Model Provision consumer則
  保持為「NWDAF呼叫另一個具MTLF capability之NWDAF」的薄standard consumer，不把MTLF描述成獨立標準NF。
- `internal/anlf/coordinator`的過渡責任完成搬移或刪除；MTLF不為外觀對稱而新增coordinator。
- 無法由current free5GC generated dependency提供的3GPP wire types，統一收斂到
  `internal/compat/<service>`，且不混入business state或workflow。
- portable E2E會使用真實team SMF、team go-upf、ADRF與MongoDB，且完整經過
  Nsmf create/delete、Nupf create/delete及UPF direct notification。
- NRF discovery是portable E2E的第二個可選scenario；主要acceptance先以configured endpoints隔離資料管線。
- 具備active PDU session、PFCP、gtp5g與UE/RAN的full-core驗證保留為後續選配，不再阻擋Phase 7完成。

---

## 2. 已核對的證據

### 2.1 3GPP OpenAPI 與 TS

| 行為 | 本地證據 | 本階段約束 |
|---|---|---|
| SMF Event Exposure create/update/delete | `specs/openapi/TS29508_Nsmf_EventExposure.yaml`；TS 29.508 clause 4.2.3 | create 維持 `POST`／`201 + Location`；delete 維持 `DELETE`，不得為測試建立平行 custom operation |
| R18經SMF訂閱、UPF直接通知consumer | TS 29.508 V18.11.0 clause 4.2.3.2 Note 2；TS 23.502 V18.14.0 clauses 4.15.4.5.1、4.15.4.5.2 steps 3–5 | `UPF_EVENT`允許SMF做third-party Nupf subscription；SMF把Nsmf的consumer address/correlation傳給UPF，UPF直接通知PyAnLF |
| SMF notification callback | 同一 OpenAPI callback；TS 29.508 clause 4.2.2.2 | consumer 成功接受 notification 回 `204` |
| UPF Event Exposure create/delete/callback | `specs/openapi/TS29564_Nupf_EventExposure.yaml`；TS 29.564 clauses 5.2.2.2.2、6.1.5.2 | SMF以Nupf `eventNotifyUri`及`notifyCorrelationId`傳遞實際consumer資訊；UPF callback成功回`204` |
| ADRF storage | `specs/openapi/TS29575_Nadrf_DataManagement.yaml`；TS 29.575 clause 4.2.2.2 | `POST /data-store-records` 成功回 `201 + Location` |
| ADRF retrieval subscription | 同一 OpenAPI；TS 29.575 clause 4.2.2.6 | create 成功回 `201 + Location`，resource Location 必須可供後續 delete |
| ADRF fetch notification | 同一 OpenAPI；TS 29.575 clause 4.2.2.8 | Go 只在成功交付 MTLF backend 後回 `204`；backend unavailable 不得假裝成功 |
| ADRF direct retrieval | 同一 OpenAPI；TS 29.575 clause 4.2.2.5 | PyMTLF 依 fetch instruction 直接 `GET`；`200` 有資料、`204` 無資料 |
| NRF discovery/cache | `specs/openapi/TS29510_Nnrf_NFDiscovery.yaml`；TS 29.510 clauses 5.3.2.2.1、5.3.2.2.2、6.2.6.2.2 | 使用標準 query 與 `SearchResult`；有效期內可共用 Go cache |
| ML Model Provision/Monitor | `TS29520_Nnwdaf_MLModelProvision.yaml`、`TS29520_Nnwdaf_MLModelMonitor.yaml`；TS 29.520 clauses 4.5、4.7 | package 整理不得改變既有標準 payload、method、status 或 Location semantics |

`nwdaf-docs/specs/openapi/README.md` 已記錄 Release 18 corpus 與尚未收錄的 external `$ref`。因此本階段
不得在未驗證 dependency closure 的前提下，把「可從單一 YAML 看到 schema」等同於「可以安全替換整個
free5GC OpenAPI module」。

### 2.2 團隊元件

以下是撰寫計畫時實際核對過的 team-owned revisions。它們是目前的 tested baseline，不代表分支名稱永久不變；
實作時必須在 `nwdaf-resources` 留下可重現的 lock manifest。

| 元件 | Remote | Branch | Tested commit | 已確認能力 |
|---|---|---|---|---|
| SMF implementation base | `Intelligent-Systems-Lab/smf-nwdaf-ext` | `feat/smf-event-exposure` | `d35e0c1a5c2a` | SUPI/PDU session到selected UPF resolution、Nsmf/Nupf create/delete、完整focused tests；portable application E2E前需完成R18 wire修正 |
| SMF local-wire reference | 同一remote | `ees-only` | `8668372` | Nsmf/Nupf request types與raw HTTP client位於SMF repository內；不需要額外OpenAPI fork，focused repository tests可通過 |
| UPF implementation base | `Intelligent-Systems-Lab/go-upf` | `dev/ees-next-version` | `76e4149e068f` | Nupf Event Exposure endpoint、subscription store與notification |
| UPF pseudo replay reference | 同一remote | `test-EES-with-pseudodriver` | `9a4d95c` | Parquet shared reader、subscription target filtering、batching與paced replay；仍耦合正式UPF startup及URR barrier |
| ADRF | `Intelligent-Systems-Lab/adrf` | `with-mlmodelmanagement` | `7f947db66292` | Data Management、Mongo persistence、retrieval callback、NRF registration/heartbeat |

### 2.3 實際 preflight 結果

已確認下列限制，計畫與驗證宣告必須如實反映：

1. `feat/smf-event-exposure`目前直接使用其`go.mod`會因upstream
   `github.com/free5gc/openapi v1.2.4`不含Nupf Event Exposure package而無法編譯；本階段不以額外
   OpenAPI fork解決，而是改用SMF-local R18 wire types與raw HTTP client。
2. `ees-only`證明local-wire方向可行且focused repository tests可通過，但不可直接當作最終實作：
   它的Nsmf decoder要求Release 19 BERMS的`bundledEventNotifyUri`，且缺少新版branch的focused
   Event Exposure tests。實作只參考其wire/client做法，不回退整個SMF branch。
3. current PyAnLF依Release 18送出top-level`nfId`、`notifId`、`notifUri`與`eventSubs[].upfEvents`。
   current SMF strict decoder會拒絕`nfId`，並反過來要求R19
   `eventSubs[].bundledEventNotifyUri`；因此portable application E2E前必須先修正SMF contract。
4. Release 18已明確定義third-party subscription：SMF應將Nsmf`notifUri`映射至Nupf
   `eventNotifyUri`，將Nsmf`notifId`映射至Nupf`notifyCorrelationId`，UPF再直接通知PyAnLF。
5. go-upf `internal/ees`可以編譯，但目前沒有package tests。
6. go-upf 正式 application startup 仍會初始化 gtp5g／PFCP；非特權環境會得到
   `operation not permitted`。pseudo EES driver 不等於整個 UPF binary 可無特權啟動，因此本階段新增
   只啟動Nupf Event Exposure、subscription store、notifier與dataset replay的獨立entrypoint；正常UPF
   binary的startup不因測試模式被改寫。
7. ADRF 的 SBI、store 與 factory focused suites可通過，也可在 `nrfUri` 留空時以 standalone 模式運行。
8. 目前採用的 free5GC NRF 允許 ADRF registration，但 discovery query validation 的 target NF type map
   沒有 ADRF；因此 `target-nf-type=ADRF` 會回 `400`。這是 pinned environment compatibility limitation，
   不是 3GPP 不允許 ADRF discovery。
9. SMF production resolver要建立Nupf subscription時，必須已有包含SUPI、UE IP與selected UPF的active
   PDU session。portable E2E改由顯式啟用且預設關閉的static session resolution config提供同一組解析結果；
   它不得改變Nsmf/Nupf payload或production resolver。
10. NWDAF目前依賴`github.com/free5gc/openapi v1.2.3`；SMF local-wire修正不得連帶升級NWDAF dependency。
11. pseudo replay branch在現狀下會等待實際URR ID 2作為network clock anchor，最長60秒才fallback。
    portable replay不得依賴此barrier；收到有效Nupf subscription後即可用設定的anchor／speed開始播放。
12. NWDAF current config要求非空`nrfUri`，且startup會同步註冊NRF，失敗即終止啟動。要讓configured
    endpoint scenario真正不啟動NRF，本階段必須加入預設保持啟用的明確registration switch；不能只在
    harness隱藏fake NRF。

---

## 3. 範圍

### 3.1 納入

本階段包含：

1. 真實 ADRF process-level E2E。
2. team SMF static session resolution、team go-upf standalone dataset replay、ADRF與MongoDB所組成的
   portable application E2E harness。
3. team SMF的R18 Nsmf→Nupf mapping修正、local Nupf wire types/client及contract tests。
4. team go-upf standalone replay entrypoint、dataset target filtering、create/delete與notification tests。
5. team component branch/commit/dependency lock 與 preflight。
6. NWDAF加入預設為啟用的NRF registration config；portable configured-endpoint scenario可顯式關閉，
   normal deployment的register/heartbeat/deregister行為保持不變。
7. 依AnLF既有package慣例整理AnLF／MTLF各自的private backend HTTP edge、processor、client與config。
8. Go legacy MTLF、Daisy、固定 ADRF runtime 與已無 production caller 的 traffic-data state清除。
9. `internal/anlf/coordinator`的production call graph拆解、責任搬移與package移除。
10. PyAnLF 已不再掛載的 legacy analytics/Daisy routes與tests清除。
11. 所有手寫3GPP wire types的inventory、scoped generation audit與
   `internal/compat/<service>`收斂。
12. config、docs、API docs、naming、dead-code與final regression audit。

### 3.2 不納入

- Multiple NWDAF、AGG、AoI routing或HFL。
- TLS、mTLS、OAuth delegation與Python backend NF identity。
- 擴展SMF/UPF到R19 BERMS bundling、group/any-UE、GET/PUT或新的Event Exposure business semantics。
- 在normal production mode跳過PDU session resolution，或放寬標準required fields。portable mode只能以
  顯式config啟用，不新增custom Nsmf/Nupf operation，也不得改變normal resolver結果。
- 讓normal NWDAF deployment預設不向NRF註冊；registration switch省略時必須保持現有enabled behavior。
- 把active PDU session、PFCP、gtp5g、UE/RAN與完整free5GC core列為Phase 7 completion prerequisite；
  privileged full-core只保留為後續選配驗證。
- ADRF 不可用期間寫入 Mongo、恢復後再回填 ADRF 的歷史 gap merge。
- 讓 `go test ./...` 自動要求 root、gtp5g、network namespace 或完整 core。
- 修復 pinned free5GC NRF 的 ADRF discovery validator。可另案升級／修正，但不是本階段 completion blocker。
- 大範圍升級整個 free5GC OpenAPI dependency，只為了刪除少量 compatibility types。
- ADRF與其他未列入source-change範圍的reference working trees維持pinned external components。
  SMF R18 contract/static resolution及go-upf standalone replay是本階段明列的team-component source
  changes，必須分別在各自repository的branch、tests與commit完成，不混入NWDAF或
  `nwdaf-resources` commit。

---

## 4. Target Architecture

### 4.1 Runtime

```text
External consumer
        |
        | Nnwdaf standard SBI
        v
+---------------------------- NWDAF Go -----------------------------+
| external SBI / NF identity / NRF registration                    |
| standard NF consumers / shared NRF discovery cache               |
| backend availability, sync, routing, minimal correlation state   |
|                                                                   |
| AnLF auxiliary edge (:8090)     MTLF auxiliary edge (:8091)      |
| - AnLF-originated requests      - MTLF-originated requests        |
| - AnLF sync/associations        - MTLF callbacks                  |
| - thin AnLF processor           - thin MTLF processor             |
|              \\                  /                                |
|               +-- shared SBI consumers / context repositories ---+
+---------------+------------------------------------+--------------+
                |                                    |
                v                                    v
           AnLF backend                         MTLF backend
         policy/inference                    policy/retrieval/train
                |                                    |
         UPF callback direct                    ADRF direct GET
                |                                    |
                +---------- ADRF / Mongo ------------+
```

這個形狀延續AnLF backend transition已確立的邊界：domain根package是該backend對Go的auxiliary HTTP
edge，標準peer communication則由SBI consumer實作。兩個edge可以使用相同的standard-shaped operation，
但只掛載各自backend實際需要的route；共用NRF cache、SMF／ADRF client與resource repository以窄介面注入，
不複製standard client實作。

本階段明確採用兩個實體listener，而不只是在package層做邏輯分離：AnLF edge沿用`8090`，MTLF edge使用
`8091`。這是為了讓兩個backend各自從sync取得唯一且正確的callback base URI、避免MTLF-originated route
繼續寄居在AnLF server，並讓backend-specific route ownership可由實際socket boundary驗證。這項選擇不表示
AnLF或MTLF是獨立NF；兩個listener都仍屬於同一個Go NWDAF process的private backend interface。

`internal/backend`只保留真正跨backend的health、sync、availability與通用HTTP helper，不擁有listener、
route table或application processor。`internal/anlf/coordinator`也不是永久architecture component；
長期resource state歸Go context repository，單次use case歸processor，backend HTTP transport歸client。

### 4.2 Portable application E2E data path

```text
Consumer
   |
   | Nnwdaf Events Subscription
   v
NWDAF Go -> PyAnLF
              |
              | generic NF discovery request
              v
          NWDAF Go -> NRF -> SMF profile(s)
              ^
              |
PyAnLF -- Nsmf Event Exposure request --> NWDAF Go --> team SMF
                                                       |
                                                       | explicit static session resolution
                                                       | (SUPI -> UE IP + Nupf apiRoot)
                                                       | Nupf Event Exposure
                                                       v
                                             team go-upf EES replay
                                                       |
                                                       | dataset-backed direct notification
                                                       v
                                                    PyAnLF
                                                       |
                                  ADRF available: standard storage via Go
                                                       v
                                                      ADRF
                                                       ^
                                                       | direct data GET
                                                    PyMTLF
```

最小scenario不啟動NRF：PyAnLF使用configured SMF endpoint，PyAnLF與PyMTLF各自使用configured ADRF
endpoint。第二個scenario才啟動NRF並讓PyAnLF透過Go的generic discovery route與shared cache取得SMF；
若current pinned NRF仍無法查詢ADRF，ADRF繼續使用configured mode。兩個scenario使用完全相同的
Nsmf/Nupf request、callback及resource lifecycle，NRF只改變endpoint discovery方式。

SMF static resolution只是portable harness的session-context替代來源：

```yaml
eventExposure:
  staticSessionResolution:
    enabled: true
    targets:
      - supi: imsi-208930000000001
        ueIpv4Addr: 10.10.0.1
        nupfEeApiRoot: http://127.0.0.1:19090
```

欄位名稱可依team SMF既有config結構作機械式調整，但語意固定。`enabled`預設為`false`；相同SUPI不得有
多個ambiguous mapping；target不存在時走和production resolver找不到active session相同的error
classification。它不保存PyAnLF callback URI，callback仍只取自每次Nsmf request。

go-upf以repository內獨立entrypoint（例如`cmd/ees-replay`）啟動Nupf router、subscription store、
notifier與dataset reader，不初始化PFCP、gtp5g或GTP-U forwarding。收到有效Nupf create後依request內的
target/UE IP篩選dataset，以config指定的speed與batching播放；delete必須立即停止該resource的後續通知。
這個entrypoint重用production Nupf handler與wire contract，不建立test-only HTTP API。

### 4.3 Deferred full-core substitution

未來要執行privileged full-core時，只替換兩個資料來源：

- SMF關閉`staticSessionResolution`，改由active PDU session取得UE IP與selected UPF。
- go-upf改用normal application startup，由PFCP/URR與真實user-plane data驅動notification。

NWDAF、PyAnLF、PyMTLF、ADRF、Nsmf/Nupf payload與callback URI都不應因此改變。這是後續deployment
validation，不是Phase 7目前的完成條件。

---

## 5. Go Package And Config Closure

### 5.1 目標 package 形狀

建議整理為：

```text
internal/
  backend/
    contract.go
    availability_monitor.go
    standard_http.go
  compat/
    mlmodel/           # 只有scoped generation仍不可行時保留
    adrf/              # 只有scoped generation仍不可行時保留
    nsmf/              # NWDAF的R18 UPF_EVENT缺口；只有scoped generation仍不可行時保留
  anlf/
    server.go
    api_*.go
    http_contract.go
    client/
    contract/        # 只有仍存在的private DTO才保留
    processor/
  mtlf/
    server.go
    api_*.go
    http_contract.go
    client/
    contract/        # 只有仍存在的private DTO才保留
    processor/
  sbi/
    consumer/
    processor/
    notifier/
```

package move遵守：

- `internal/anlf`與`internal/mtlf`根package各自只做HTTP decoding、contract validation、response writing與
  listener lifecycle，並呼叫自己的processor。
- AnLF／MTLF processor只處理該backend發起的一次use case、status/error mapping與窄介面delegation，
  不持有long-running workflow或standard peer HTTP client。
- NRF、SMF與ADRF standard clients、shared NRF cache及external notification仍位於`internal/sbi`。
- 標準ML Model Provision consumer若contract正確，保留或重構為NWDAF-to-NWDAF standard consumer；
  本階段只移除fixed endpoint selection與舊runtime caller，不因local backend routing暫時無caller就把它
  判定成「獨立MTLF NF」legacy。若現有`NmtlfService`命名會暗示獨立MTLF NF，優先改為
  `MLModelProvisionService`／`ml_model_provision_service.go`等service語意命名。
- `internal/backend`不擴張成gateway、processor或service locator。
- `internal/anlf/coordinator`移除；MTLF不新增coordinator。跨request所需的最小routing state位於
  Go context repository，backend restart sync由`pkg/service`組合snapshot並透過backend client送出。
- `anlf/client`與`mtlf/client`只代表Go到各自backend的HTTP transport。
- `contract/`不為了維持目錄對稱而保留；只保存無法用generated／compat標準type表示的private DTO。
- package relocation不得更改既有path、method、body、header、Location或status。
- MTLF backend收到的`internalCallbackBaseUri`由歷史共用的AnLF `8090`改成MTLF `8091`是本次唯一刻意的
  private base-URI變更；PyMTLF、E2E config與tests必須同時更新，不建立fallback alias。
- Go source/config/log不出現`PyAnLF`或`PyMTLF`命名。

目前已掛載的route必須先按呼叫來源分配到正確edge，再mechanical preserve；同一standard-shaped operation
若兩個backend都需要，可在兩個listener掛載相同path，但共用同一個SBI consumer／repository實作：

| Route group | Target edge | Current private path |
|---|---|---|
| Events Subscription notification relay | AnLF | `POST /internal/v1/events-subscription-notifications` |
| Generic NRF discovery | AnLF與MTLF各自需要時掛載 | `GET /internal/v1/nrf/nf-instances` |
| SMF Event Exposure proxy | AnLF | `/internal/v1/smf-event-exposure/subscriptions...` |
| ADRF storage proxy | AnLF | `/internal/v1/adrf-data-management/data-store-records...` |
| ADRF retrieval subscription proxy | MTLF | `/internal/v1/adrf-data-management/data-retrieval-subscriptions...` |
| SMF resource-association sync | AnLF | `PUT /internal/v1/sync/anlf/smf-resource-associations` |
| ML Model Provision subscription CRUD | AnLF；由AnLF backend發起、Go轉送MTLF backend | `/internal/v1/ml-model-provision/subscriptions...` |
| ML Model Provision notification | MTLF；由MTLF backend發起、Go轉送AnLF backend | `/internal/v1/ml-model-provision/notifications`及resource callback path |
| ML Model Monitor registration CRUD | AnLF；由AnLF backend向MTLF backend註冊 | `/internal/v1/ml-model-monitor/registrations...` |
| ML Model Monitor subscription CRUD | MTLF；由MTLF backend向AnLF backend訂閱 | `/internal/v1/ml-model-monitor/subscriptions...` |
| ML Model Monitor notification | AnLF；由AnLF backend發起、Go轉送MTLF backend | `/internal/v1/ml-model-monitor/notifications`及resource callback path |

目前`api_analytics_report.go`、`api_runtime_completion.go`、`api_model_accuracy_report.go`與
`api_mlmodel_notify.go`的handlers並未由current server掛載；其對應舊callback URI/builders及runtime
coordinator是legacy audit candidate，不可因檔案存在就當作current edge contract搬移。

### 5.2 Config migration

現有factory schema/default已具備`anlf.server`的`8090`與`mtlf.server`的`8091`，但current sample只列出
AnLF，且`mtlf.server`仍被包在Daisy-era `MtlfConfig`。目標sample與精簡後schema為：

```yaml
configuration:
  anlf:
    server:
      bindingIPv4: 127.0.0.1
      registerIPv4: 127.0.0.1
      port: 8090
  mtlf:
    server:
      bindingIPv4: 127.0.0.1
      registerIPv4: 127.0.0.1
      port: 8091
  nrfRegistrationEnabled: true
  nrfUri: http://127.0.0.10:8000
```

遷移規則：

1. AnLF edge沿用既有`8090`；MTLF edge啟用既有config/default `8091`，不另造第三套gateway config。
2. AnLF sync的`internalCallbackBaseUri`使用AnLF server URI；MTLF sync使用MTLF server URI。
3. `mtlf.server`不再代表Daisy callback server；移除Daisy route後改為current MTLF backend-facing edge。
4. 移除`externalMtlf`、fixed Go `adrf`及`mtlf`下的`enabled`、endpoint、trigger、task、
   accuracy policy、model provider等Daisy／Go MTLF business設定；`mtlf`只保留server edge設定。
5. `anlfBackend`、`mtlfBackend`、NRF、external SBI設定保留。
6. ADRF configured/discovery policy仍分別位於PyAnLF與PyMTLF；Go的standard ADRF proxy以每次請求提供的
   target API root建立／重用client。
7. sample config、schema、URI helper、sync tests、startup/shutdown與deployment docs必須同時覆蓋兩個edge。
8. `nrfRegistrationEnabled`省略時視為`true`，保持現有normal startup。設為`false`時允許省略
   `nrfUri`，並跳過register、heartbeat與deregister；generic discovery route回明確`503`，不以空URI
   建立client。
9. registration switch不自動改寫PyAnLF/PyMTLF的configured/discovery mode。部署設定若要求NRF discovery
   卻關閉registration／無`nrfUri`，必須在startup preflight被拒絕。

命名語意固定如下：

- `anlf.server`與`mtlf.server`只代表Go NWDAF自己擁有的private auxiliary listeners，不是Python process
  的listen設定，也不表示AnLF／MTLF是獨立NF。
- `anlfBackend`與`mtlfBackend`只代表Go呼叫各backend所使用的target endpoint、polling及availability設定。
- Go source、config與log不得出現`PyAnLF`或`PyMTLF`；需要區分時使用AnLF backend、MTLF backend或
  auxiliary edge。
- 本階段不為了命名外觀重新巢狀化config schema；只有上述語意仍無法在schema description與deployment
  文件中清楚表達時，才提出獨立config migration，不在實作中無聲改名。

### 5.3 Legacy removal gate

下列項目列為candidate，不得僅因檔名看似legacy就刪除：

| Candidate | 目前判斷 | 刪除前證據 |
|---|---|---|
| `internal/mtlf`下的scheduler、trigger、training、ADRF retrieval、Daisy server/processor | 已由PyMTLF取代 | production call graph只剩無效 lifecycle wiring；backend callback tests已覆蓋新路徑 |
| `internal/mtlf/client/daisy.go` | legacy external MTLF/Daisy client | production config與consumer interface無caller |
| `internal/sbi/consumer/mtlf_service.go` | 標準NWDAF-to-NWDAF ML Model Provision consumer；不是獨立MTLF NF client | 驗證Release 18 contract後保留，並優先改成`ml_model_provision_service.go`／`MLModelProvisionService`等service語意命名；移除fixed `externalMtlf` caller/config，但不因current local routing無caller而直接刪除 |
| fixed `Consumer.Adrf`與舊typed ADRF helpers | 舊Go-owned dataset flow | standard target-root proxy仍保留且有contract tests |
| `context.AdrfSmfInfo`、traffic bucket、old Mongo query | 舊Go storage/retrain state | 非test production caller歸零；PyAnLF/PyMTLF storage tests已存在 |
| `Processor.mtlf *MtlfService`與scheduler lifecycle methods | 僅遺留WaitGroup wiring | processor可在nil legacy service下完整啟停並通過race/lifecycle tests |
| Go AnLF custom runtime apply/release、observation binding及未掛載callback handlers | 已由standard Events Subscription backend resource與backend-owned runtime取代 | current mounted route、model provision與consumer notification flow不依賴舊island |
| `internal/anlf/coordinator` | 遷移期cross-domain facade，不是永久標準邊界 | Events Subscription改由SBI processor透過窄client介面呼叫；SMF association改由context repository保存；availability與sync各有明確owner；其餘runtime／observation／model fan-out caller歸零 |
| PyAnLF `legacy_router`與`DAISY_RETRAIN` tests | production app已不掛載 | current API docs與replacement tests完整 |

每批刪除必須先：

1. 以非test call graph列出所有caller。
2. 將caller分類為current production、test-only或same legacy island。
3. current production caller未歸零時停止刪除並修正計畫／實作。
4. 確認替代路徑已有contract、failure與process-level tests。
5. 刪除後執行affected package tests與repository-wide regression。

---

## 6. Standard Wire Compatibility Types Consolidation

current free5GC generated dependency與local Release 18 corpus的缺口不只ML Model。已知NWDAF內至少有
ML Model Provision／Monitor、ADRF Data Management，以及Nsmf `UPF_EVENT`／nested UPF event fields的
手寫wire types；team SMF另有Nsmf `UPF_EVENT`與Nupf Event Exposure缺口。本階段先建立完整inventory，
不把單一`internal/mlmodel/wire`誤當成全部問題。NWDAF現有手寫Nsmf shape中的
`bundlingAllowed`／`bundledEventNotifyUri`屬於本R18流程不採用的Release 19 BERMS方向，必須在call graph
確認後移除，不得被搬進R18 compat package。

### 6.1 Inventory

| Standard schema family | Repository | Current gap | Target |
|---|---|---|---|
| Nnwdaf ML Model Provision／Monitor | NWDAF | `internal/mlmodel/wire`與current generated models並存 | generated type或`internal/compat/mlmodel` |
| Nadrf Data Management | NWDAF | request／response／callback types散落在SBI API與consumer | generated type或`internal/compat/adrf` |
| Nsmf `UPF_EVENT` | NWDAF | current generated Nsmf model缺少R18 `nfId`、`eventSubs[].upfEvents`等欄位；手寫shape混入R19 bundling欄位 | generated type或`internal/compat/nsmf`；移除R19 bundling欄位 |
| Nsmf `UPF_EVENT`／Nupf Event Exposure | team SMF | current generated module缺少完整cross-spec types/package | SMF repository自己的`internal/compat/nsmf`與`internal/compat/nupf` |

inventory需繼續掃描所有以TS/OpenAPI名稱手寫、帶標準JSON tag或在standard handler／consumer邊界使用的
struct。private backend health/sync DTO、Mongo document與application state不屬於standard wire compat。

### 6.2 Per-service gate

每個schema family分別執行：

1. 先確認current generated type是否已完整可用。
2. 不完整時以對應local Release 18 YAML做scoped reproducible generation，記錄generator版本、命令、
   external `$ref`來源與輸出diff。
3. 驗證JSON tags、required/optional、enum、nullable、unknown-field forwarding及lossless round trip。
4. 只有generation仍受dependency closure或version mismatch阻擋時，才建立
   `internal/compat/<service>`。
5. 不為其中一個schema family大幅升級整個workspace OpenAPI module。

### 6.3 Compat package rules

`internal/compat`是直接位於`internal/`下的統一父資料夾，但不是單一flat Go package：

```text
internal/compat/
  mlmodel/
  adrf/
  nsmf/
```

每個子package：

- 只放源自3GPP OpenAPI的typed wire models、enums與必要codec。
- 在檔案與type上記錄exact YAML schema及TS provenance。
- 不放HTTP client、processor、resource repository、goroutine、Mongo record或backend private contract。
- 不反向依賴API、processor或consumer；由這些上層package單向import。
- generated type補齊後可獨立刪除，不影響其他schema family。
- R18 compat不得包含`bundlingAllowed`、`bundleId`或`bundledEventNotifyUri`；若未來實作R19 BERMS，
  必須另立feature plan與capability negotiation。

完成後不得留下頂層`internal/mlmodel` business-looking package，也不得讓ADRF標準wire struct繼續散落在
handler與consumer；NWDAF Nsmf `UPF_EVENT`也不得繼續以不完整或混有R19欄位的local struct散落。
不同repository各自保存自己的internal compat；SMF不得引用NWDAF的internal code。

---

## 7. E2E Strategy

E2E分為「主要portable application E2E」與「後續privileged full-core substitution」。前者是Phase 7
acceptance；後者只說明如何換回真實session/data-plane，不得阻擋目前工作。兩者均不得混入
repository-wide `go test ./...`。

### 7.1 Portable application E2E

Owner：`nwdaf-resources/`

建議建立獨立目錄：

```text
nwdaf-resources/
  deployments/
    portable_event_exposure/
      README.md
      components.yaml
      configs/
      datasets/
      scripts/
      checks/
```

它不是NWDAF、SMF、UPF或ADRF repository內的test資料夾；跨process orchestration只在support-tooling
repository維護。最低啟動元件：

- team SMF，啟用static session resolution。
- team go-upf EES replay entrypoint。
- NWDAF Go。
- PyAnLF。
- PyMTLF。
- ADRF。
- MongoDB；ADRF store與PyAnLF fallback使用不同database。
- test consumer。
- NRF只在discovery scenario啟動，不是configured-endpoint scenario的前提。

主要流程：

```text
Consumer Events Subscription
  -> NWDAF Go -> PyAnLF
  -> PyAnLF standard Nsmf request -> NWDAF Go -> real team SMF
  -> static SUPI resolution -> standard Nupf create -> real team go-upf EES replay
  -> dataset-backed standard UPF notifications -> PyAnLF
  -> inference + standard ADRF storage via Go -> real ADRF/Mongo
  -> accuracy degradation -> PyMTLF ADRF retrieval subscription via Go
  -> ADRF callback -> Go -> complete fetch instruction -> PyMTLF direct ADRF GET
  -> dataset READY -> local retrain/promotion
  -> standard Model Provision notification -> PyAnLF download/atomic activation
  -> Consumer delete -> PyAnLF Nsmf delete -> SMF Nupf delete -> replay stops
```

必要assertions：

1. PyAnLF送出的current Release 18 Nsmf request可由team SMF接受。
2. SMF static mapping只取代session lookup；Nupf body與normal resolver產生相同target、UE IP、
   callback及correlation語意。
3. Nupf create回`201 + Location`，SMF保存peer Location；Nsmf delete會cascade成正確Nupf delete。
4. go-upf replay不啟動PFCP、gtp5g或GTP-U；收到有效subscription後在bounded時間內開始播放，不等待
   URR ID 2或60秒fallback。
5. dataset target須與SMF static mapping的UE IP/SUPI相符；不匹配時明確得到no matching data，而不是
   偷送其他UE資料。
6. UPF notification直接送到Nsmf top-level`notifUri`所指定的PyAnLF callback；PyAnLF成功處理回`204`。
7. ADRF可用時PyAnLF只寫ADRF，不同時寫fallback Mongo。
8. ADRF record保留standard `dataSub`與`dataNotif`；retrieval Location被保存並在terminal path delete。
9. Callback中的fetch instruction不被Go改寫成custom dataset DTO；PyMTLF direct GET取得對應
   SUPI/time window資料。
10. `204 no data`、malformed fetch instruction、ADRF/SMF/UPF `5xx`、callback failure與retry有明確結果。
11. Mongo fallback scenario仍可完成local training；ADRF恢復後不merge先前Mongo gap。
12. delete完成後同一UPF resource不得繼續送notification；所有child process及test database可清理。

### 7.2 Endpoint discovery scenarios

同一套portable harness依序執行：

1. **Configured endpoint acceptance**：NWDAF config顯式關閉NRF registration，不啟動fake或real NRF；
   PyAnLF直接使用configured SMF endpoint，PyAnLF/PyMTLF各自使用configured ADRF endpoint。這是最小且
   必須通過的E2E。registration關閉時若backend誤呼叫generic NRF discovery route，Go應回明確
   unavailable error，不嘗試空apiRoot。
2. **Optional NRF discovery**：啟動NRF並讓PyAnLF經Go generic discovery route取得SMF endpoint，驗證
   `SearchResult`及Go shared validity cache。若pinned NRF仍拒絕ADRF target discovery，ADRF保持
   configured mode；不得為了測試修改backend API或假裝ADRF discovery已通過。

NRF scenario失敗只影響discovery coverage，不應模糊已通過的configured data/model pipeline結果。反之，
configured scenario通過也不能宣稱NRF/cache已驗證。

### 7.3 Deferred privileged full-core substitution

未來可在同一harness旁建立`full_core_event_exposure/`，加入NRF、Mongo、必要free5GC core、subscriber
provisioning、UE/RAN simulator、gtp5g與network setup。該scenario關閉SMF static resolution並使用normal
go-upf application，驗證active PDU session、PFCP/URR及真實user-plane。它應另外使用
`privileged full-core E2E`標籤，但不屬於Phase 7完成條件，也不要求本階段修改主機網路。

### 7.4 Component lock與preflight

`components.yaml`至少記錄：

```yaml
components:
  smf:
    remote: git@github.com:Intelligent-Systems-Lab/smf-nwdaf-ext.git
    branch: feat/smf-event-exposure
    implementationBase: d35e0c1a5c2a
    commit: <r18-and-static-resolution-commit>
    contract: r18-direct-upf-notification
  go_upf:
    remote: git@github.com:Intelligent-Systems-Lab/go-upf.git
    implementationBase: 76e4149e068f
    replayReference: 9a4d95c
    commit: <standalone-replay-commit>
  adrf:
    remote: git@github.com:Intelligent-Systems-Lab/adrf.git
    branch: with-mlmodelmanagement
    commit: 7f947db66292
```

placeholder只表示計畫撰寫時尚未產生的team-component commits；portable E2E第一次被宣告可執行前必須
替換為真實commit。test runner不得自動`git pull`、切branch或修改sibling repositories。preflight只檢查：

- repository、required files及dataset存在。
- HEAD是否符合lock；不同時顯示expected/actual。
- SMF不依賴team OpenAPI fork，且R18/static-resolution contract tests可重現。
- go-upf replay entrypoint可在無gtp5g環境啟動。
- required binaries、ports、Mongo與可選NRF狀態。
- dataset中的target identity與SMF static mapping一致。
- selected scenario的NWDAF NRF registration mode與endpoint discovery mode一致。

### 7.5 SMF R18 contract與static resolution closure

portable harness開始前，`feat/smf-event-exposure`必須完成SMF repository-local修正。以新版branch作為
implementation base，僅參考`ees-only`的local types/raw HTTP方式，不回退整個branch：

1. 移除`github.com/free5gc/openapi/upf/EventExposure`及只存在於team fork的root model dependency。
2. 在SMF repository自己的`internal/compat/nsmf`與`internal/compat/nupf`建立隔離且附有schema
   provenance的local wire compatibility packages，不得把手寫shape散落在handler、processor或tests，
   也不得引用NWDAF的`internal/compat`。Nsmf package只補current generated model缺少且本流程需要的
   R18 request fields；Nupf package只定義R18流程實際需要的typed wire types：
   `CreateEventSubscription`、`UpfEventSubscription`、`UpfEvent`、`UpfEventMode`及成功/error response
   所需的最小shape。
3. 使用bounded `http.Client`實作Nupf create/delete，保留redirect rejection、OAuth context、
   `ProblemDetails`分類、`201 + Location`驗證與target-root client reuse等新版branch行為。
4. Nsmf decoder接受current PyAnLF送出的R18欄位：
   `supi`、`nfId`、`notifId`、`notifUri`、`eventSubs[].upfEvents`、`notifMethod`與`repPeriod`。
5. 保留第一版single-SUPI、single matching PDU session、`PERIODIC`、
   `USER_DATA_USAGE_MEASURES`及`PER_SESSION`限制；unsupported標準feature仍以清楚`ProblemDetails`拒絕，
   不能因local struct而silent ignore。
6. 依Release 18 third-party subscription固定映射：

```text
NsmfEventExposure.notifUri
  -> UpfEventSubscription.eventNotifyUri

NsmfEventExposure.notifId
  -> UpfEventSubscription.notifyCorrelationId
```

7. Nupf `nfId`使用實際建立Nupf subscription的SMF NF instance ID；Nsmf收到的NWDAF `nfId`仍須被
   正確decode/validate，不得因`DisallowUnknownFields`誤回`400`。
8. 不要求或注入Release 19 BERMS欄位`bundlingAllowed`、`bundleId`或
   `bundledEventNotifyUri`。未來若採R19 bundling，另立feature plan與capability negotiation。
9. 保留Nsmf create `201 + Location`、delete `204`及SMF→UPF terminal delete cascade。
10. 在Event Exposure config加入預設關閉的static session resolution；每筆mapping至少含SUPI、UE IPv4
    address與Nupf apiRoot，並通過既有request builder產生Nupf body。
11. static mode只替換active-session lookup，不能建立新的HTTP operation、hard-code callback、繞過
    request validation或改變normal production resolver。
12. missing／duplicate mapping、invalid UE IP／apiRoot及UPF unavailable均有明確failure classification。

不接受：

- 以hidden `go.work`或test runner臨時改寫`go.mod`注入OpenAPI fork。
- 要求PyAnLF加入R19欄位才能通過R18流程。
- 以SMF fixed config保存PyAnLF callback URI。
- static resolution在未開啟時影響normal PDU-session path。
- build或contract失敗時改用mock SMF並仍宣稱portable application E2E通過。

### 7.6 go-upf standalone dataset replay closure

以`dev/ees-next-version`為implementation base，參考`test-EES-with-pseudodriver`的Parquet reader、
target filtering、batching與paced replay，但不直接沿用其full-app/URR coupling：

1. 建立獨立EES replay entrypoint，只組裝Nupf router、subscription repository、notifier與dataset
   reader；不得初始化PFCP、gtp5g、GTP-U driver或修改host network。
2. 重用normal Nupf handler、create/delete status、Location與callback payload；不建立`/test`、
   `/replay`等平行HTTP控制面。
3. 收到Nupf create後，以subscription target/UE IP選取dataset rows；可設定dataset path、speed、
   batch interval與clock anchor，預設值必須適合本地E2E。
4. standalone mode不等待URR ID 2。production pseudo mode若仍需要URR anchor，其行為保持不變。
5. 每個subscription有獨立cancelable replay lifecycle；delete、shutdown與terminal error都停止後續通知。
6. 相同dataset的shared read可以保留，但不得跨subscription洩漏其他target資料。
7. 加入handler、target filtering、start/delete、callback `204`/failure、concurrency與graceful shutdown
   tests；`nwdaf-resources`再做真實process black-box驗證。

---

## 8. Implementation Slices

本階段以可驗證的slice工作，但不要求每個slice單獨commit。全部完成後再做整體review與按repository整理commit。

### Slice 1：freeze baseline與component preflight

Repository：`nwdaf-resources/`。

- 加入component lock與preflight。
- 記錄configured endpoint、optional NRF及deferred full-core環境能力矩陣。
- 將SMF R18/static-resolution與go-upf standalone replay tests/final commits列為portable E2E precondition。
- 保存Phase 6現有三process/Mongo E2E baseline。

### Slice 2：SMF R18 direct-notification與static resolution closure

Repository：`smf-nwdaf-ext`，以`feat/smf-event-exposure`為implementation base並獨立commit。

- 依第7.5節建立local Nupf wire types/raw HTTP client，移除team OpenAPI fork需求。
- Nsmf decoder接受PyAnLF現有R18 shape與`nfId`。
- 將top-level`notifUri/notifId`映射到Nupf`eventNotifyUri/notifyCorrelationId`。
- 保留新版resolver、resource repository、failure classification、OAuth與delete cascade。
- 加入預設關閉的SUPI→UE IP/Nupf apiRoot static resolution，且重用normal Nupf request builder。
- 新增request decode、static/normal resolver、captured Nupf body、create/delete、Location、peer failure及
  concurrency tests。
- 完成後把真實SMF commit補入component lock。

### Slice 3：go-upf standalone dataset replay

Repository：`go-upf`，以`dev/ees-next-version`為implementation base並獨立commit。

- 依第7.6節建立無PFCP/gtp5g startup的EES replay entrypoint。
- 從pseudo branch移植必要Parquet reader、target filtering、batching與paced replay，不移植URR barrier。
- 重用normal Nupf router、subscription store、Location、delete及notification wire contract。
- 新增target isolation、bounded start、cancel/delete、callback failure、concurrency及shutdown tests。
- 完成後把真實go-upf commit補入component lock。

### Slice 4：portable application E2E與real ADRF

Repository：`nwdaf-resources/`。

- 建立`deployments/portable_event_exposure/`，只啟動SMF、UPF EES replay、NWDAF、PyAnLF、PyMTLF、
  ADRF、Mongo與consumer。
- 建立動態ADRF config與process lifecycle。
- 使用真實ADRF storage、retrieval subscription、callback、direct GET與delete。
- 完整驗證Nsmf→SMF→Nupf→UPF direct callback與terminal delete。
- configured endpoint scenario為必須；NRF SMF discovery/cache為獨立選配scenario。
- 保留Mongo fallback scenario。
- 新增failure-path assertions與完整README命令／log判讀。
- 若查出ADRF owner-side contract bug，保存最小reproduction並停止該case；另開repository-specific修正，
  不由harness臨時patch。

### Slice 5：AnLF／MTLF auxiliary edge consolidation

Repositories：`NWDAF/`、`PyMTLF/`、`nwdaf-resources/`。

- 保留`internal/anlf`既有HTTP edge／processor／client責任形狀，不搬入neutral gateway。
- 將`internal/mtlf/server.go`從Daisy callback server改成current MTLF backend-facing edge，並建立必要
  `api_*.go`、`http_contract.go`與processor。
- 依第5.1節把現有routes按backend caller分配；standard peer client與shared NRF cache仍由
  `internal/sbi`提供。
- 啟動AnLF `8090`與MTLF `8091`兩個edge；sync snapshot分別帶入對應
  `internalCallbackBaseUri`。
- 將`MtlfConfig`縮成server-only auxiliary edge config，更新schema/default/sample、startup/shutdown與
  contract tests。
- 加入預設enabled的NWDAF NRF registration switch；disabled mode只供configured-endpoint
  portable deployment，並驗證不會啟動register/heartbeat/deregister。
- 更新PyMTLF sync/API tests與E2E templates，使所有MTLF-originated operation使用sync提供的`8091`
  base URI；PyAnLF繼續使用`8090`。
- 驗證caller-specific route只掛載於正確edge；錯誤listener不應意外接受另一backend專屬operation。
- relocation前後執行相同method/path/body/header/Location/status tests，禁止趁機重新設計API。

### Slice 6：Go legacy island與AnLF coordinator removal

Repository：`NWDAF/`

- 先完成production call graph ledger。
- 移除old `MtlfService`建構與WaitGroup依賴。
- 移除Daisy、fixed `externalMtlf` selection/config與舊runtime wiring、old fixed ADRF coordinator、
  old traffic/dataset state。
- 驗證`internal/sbi/consumer/mtlf_service.go`的Release 18 method/path/status/Location contract；正確則
  保留並優先按NWDAF ML Model Provision service語意改名，作為未來external NWDAF peer consumer，
  目前不接入local backend selection。
- 移除已被standard Events Subscription flow取代的Go AnLF custom runtime/observation island與未掛載handlers；
  保留current AnLF events backend client及notification relay。
- 將Events Subscription直接轉發所需窄介面放到SBI processor；SMF association state改由context repository
  接受AnLF processor更新；availability／sync繼續使用既有明確owner。
- 移除`internal/anlf/coordinator`，不得把相同facade改名後搬到其他package。
- 縮小`internal/mtlf`為HTTP edge、processor與backend client boundary。
- 每批刪除執行focused tests，最後執行full test/lint/build/race。

### Slice 7：Python legacy cleanup

Repositories：`PyAnLF/`、`PyMTLF/`

- 移除PyAnLF未掛載的legacy router與`DAISY_RETRAIN` test vocabulary。
- 確認docs只描述current APIs。
- 全repo搜尋Daisy／legacy custom endpoint／obsolete config。
- PyMTLF只做實際發現的dead-code cleanup，不因對稱性硬改現有internal design。

### Slice 8：standard wire compatibility types consolidation

Repository：`NWDAF/`

- 建立所有手寫3GPP wire types inventory。
- 對ML Model、ADRF及NWDAF Nsmf `UPF_EVENT` schema family分別執行scoped generation feasibility gate。
- 依第6節結果使用generated types或搬入`internal/compat/<service>`。
- 刪除`internal/mlmodel/wire`，並移除SBI API／consumer中散落的重複ADRF wire structs與混有R19 BERMS
  欄位的Nsmf local shape。
- 保留lossless forwarding、required/optional、enum與standard status/header tests。

### Slice 9：final closure

Repositories：所有實際受影響repositories，分開commit。

- config/API/docs同步。
- Go/Python source、logs、configs搜尋Daisy與錯誤backend命名。
- NRF advertisement不宣稱backend是NF。
- repository status與untracked artifacts清理。
- 執行完整verification matrix並補回實際結果。
- 將privileged full-core需要的替換點與未驗證範圍留在README，但不安裝gtp5g、不修改host network，
  也不把該選配scenario列為未完成工作。

---

## 9. Verification Matrix

### 9.1 NWDAF

- `go test ./...`
- `golangci-lint run ./...`
- `go build -o bin/nwdaf ./cmd/`
- affected concurrency packages執行`go test -race`
- config validation、startup/shutdown、backend reconnect/sync
- AnLF／MTLF兩個auxiliary listener與各自sync callback base URI
- PyAnLF使用AnLF `8090`、PyMTLF使用MTLF `8091`，且backend-specific routes不跨listener誤掛載
- all moved private routes的method/path/body/header/status regression
- nontest call graph確認legacy symbols及`internal/anlf/coordinator`caller歸零
- standard wire compat inventory完整；standard wire structs不再散落於handler／consumer
- NWDAF Nsmf `UPF_EVENT`可lossless decode/forward R18 `nfId`、`notifId`、`notifUri`與
  `eventSubs[].upfEvents`，且不產生R19 BERMS bundling欄位
- 標準ML Model Provision consumer被驗證為NWDAF-to-NWDAF contract；fixed `externalMtlf` runtime/config
  已移除，但consumer不因local backend routing而被誤刪

### 9.2 PyAnLF

- `ruff check`
- full `pytest`
- current API docs與production router一致
- ADRF-first/Mongo fallback behavior
- SMF create/delete與direct UPF callback
- legacy router及Daisy vocabulary歸零

### 9.3 PyMTLF

- `ruff check`
- full `pytest`
- ADRF callback/direct fetch、Mongo fallback
- dataset READY→training→promotion→provision regression
- source change、no-data、retry、terminal cleanup

### 9.4 ADRF/team components

- ADRF focused Go tests。
- team SMF不搭配額外OpenAPI fork即可通過full build與Event Exposure/context/factory focused tests。
- team SMF contract tests至少驗證：
  - current PyAnLF R18 request shape可被接受；
  - Nupf captured body的`eventNotifyUri`等於Nsmf top-level`notifUri`；
  - `notifyCorrelationId`等於Nsmf`notifId`；
  - Nupf`nfId`為SMF NF instance ID；
  - static mapping與normal session resolution使用同一request builder；
  - static mode預設關閉、invalid/duplicate/missing mapping rejection；
  - Nsmf create/delete status、Location、UPF delete cascade及failure mapping。
- go-upf standalone replay entrypoint可在沒有gtp5g、PFCP及network privilege時啟動。
- go-upf focused tests驗證Nupf create/delete/Location、dataset target isolation、bounded replay start、
  callback failure、concurrency與graceful shutdown。
- `nwdaf-resources`以真實team SMF/go-upf process做black-box lifecycle驗證，不以fixture取代後仍宣稱
  portable application E2E。
- normal go-upf application只在deferred privileged environment驗證；本階段不以未執行normal dataplane
  否定standalone EES replay結果，也不反過來宣稱PFCP/gtp5g已通過。

### 9.5 E2E completion labels

結果必須使用下列精確標籤：

- `unit/contract`: 單process或in-memory peer。
- `portable fixture process E2E`: 真實NWDAF/Python/ADRF/Mongo，SMF/UPF可為fixture。
- `portable application E2E`: 真實team SMF、team go-upf EES replay、NWDAF/Python/ADRF/Mongo；
  SMF session resolution與UPF data source是顯式實驗模式。
- `portable application E2E + NRF`: 上一項再加上真實SMF discovery與Go shared NRF cache。
- `privileged full-core E2E`: 真實NRF/core/SMF/UPF、active PDU session、PFCP/URR與gtp5g data-plane。

`portable application E2E`可以宣稱Nsmf→team SMF→Nupf→team go-upf→PyAnLF的真實HTTP/resource
lifecycle通過，但必須同時揭露static resolution與dataset replay。只有`privileged full-core E2E`可以宣稱
active-session resolution、PFCP/URR、gtp5g及真實free5GC user-plane通過。

---

## 10. Failure And Cleanup Policy

1. 每個child process必須有bounded startup timeout、captured log與graceful shutdown；失敗後不得留下process。
2. portable harness不得修改host route、iptables/nftables、network namespace、kernel module或sysctl。
3. ADRF/Mongo使用test-specific database名稱；成功或失敗後皆可選擇保留供diagnosis，但預設清除。
4. fixture failure、component unavailable、contract mismatch與environment prerequisite missing要使用不同錯誤訊息。
5. prerequisite missing應標記skipped/not-runnable，不能偽裝passed。
6. peer回傳非預期standard status時保留status、ProblemDetails與component log。
7. test不得自動pull、checkout、commit或修補team repositories。
8. 未來full-core cleanup只能刪除harness建立的namespace/interface/config/artifact，不動使用者既有環境。

---

## 11. 完成條件

本階段只有在下列條件全部滿足時才算完成：

1. Portable application E2E以真實team SMF、team go-upf EES replay、NWDAF、PyAnLF、PyMTLF、ADRF與
   Mongo完整涵蓋Nsmf/Nupf create、direct notification、storage、retrieval、local retraining/model
   activation及terminal delete。
2. Mongo fallback E2E仍通過，且ADRF可用時不重複寫Mongo。
3. Configured endpoint scenario不需要NRF、gtp5g、UE/RAN、active PDU session或host network修改即可重現；
   optional NRF scenario的結果獨立標示。
4. Team SMF使用repository-local R18 Nupf wire/client，static resolution預設關閉且不影響normal resolver，
   通過Nsmf→Nupf mapping/lifecycle tests；component lock記錄實際commit。
5. Team go-upf提供不初始化PFCP/gtp5g的standalone EES replay entrypoint，通過target isolation、
   create/delete、callback、concurrency及shutdown tests；component lock記錄實際commit。
6. SMF與go-upf都不依賴額外OpenAPI fork、hidden workspace state或custom test-only wire operation。
7. Go AnLF／MTLF各自擁有薄的auxiliary HTTP edge；`internal/backend`沒有擴張成gateway或processor，
   對外與private wire contract沒有改變。
8. Go legacy MTLF/Daisy/dataset coordinator、fixed ADRF flow及無caller traffic state已依call graph安全移除。
9. `internal/anlf/coordinator`已移除；必要責任分別位於SBI processor、context repository、
   backend availability/sync與backend client，沒有改名後的broad facade。
10. PyAnLF production與tests不再保留legacy/Daisy API vocabulary；PyMTLF沒有Daisy residue。
11. 所有手寫3GPP wire types已有inventory；可生成者使用generated types，其餘位於
   `internal/compat/<service>`。`internal/mlmodel`、散落的ADRF wire definitions及混有R19 BERMS欄位的
   NWDAF Nsmf local shape已移除或替換。
12. 所有受影響repository的unit、lint、build與contract tests通過。
13. 文件記錄實際component revisions、commands、environment limitations與驗證結果。
14. Go／Python config、logs、route naming只呈現AnLF backend、MTLF backend或backend-neutral概念。
15. 沒有把fixture、static resolution、dataset replay或unprivileged compile誤稱為full-core E2E。
16. fixed `externalMtlf` selection/config與舊runtime caller已移除；標準ML Model Provision consumer已通過
    Release 18 contract tests，並以NWDAF-to-NWDAF service語意保留。

---

## 12. 已固定決策與不需再決策項目

本計畫已依現有共識固定：

1. AnLF與MTLF各自保有Go-side auxiliary HTTP edge；啟用既有`8090`／`8091`server config，不建立
   neutral mega-gateway。這是兩個實體private listeners，而非僅有logical package separation。
2. Configured SMF/ADRF endpoints是Phase 7主要portable acceptance；NRF是第二個選配discovery scenario。
3. Portable application與privileged full-core E2E分開；後者不是Phase 7 completion prerequisite。
4. team component由`nwdaf-resources`做lock/preflight，不由test runner自動管理Git狀態。
5. `anlf.server`繼續代表AnLF auxiliary edge；`mtlf.server`從Daisy callback用途改為MTLF auxiliary edge。
   `anlfBackend`與`mtlfBackend`則只代表Go呼叫backend的target endpoint／polling設定。
6. 移除`internal/anlf/coordinator`，且不為MTLF新增coordinator。
7. 所有手寫3GPP wire schema統一納入`internal/compat` inventory；每個service以scoped generation
   gate決定generated或`internal/compat/<service>`，不啟動整體OpenAPI升級。
8. Portable application E2E驗證真實Nsmf/Nupf HTTP/resource lifecycle與deterministic retraining；
   full-core只在未來補active-session/PFCP/gtp5g coverage。
9. PyAnLF維持現有Release 18 Nsmf request；SMF負責`notifUri/notifId`到Nupf
   `eventNotifyUri/notifyCorrelationId`的third-party subscription映射。
10. 本階段不啟用Release 19 BERMS bundling fields，也不要求PyAnLF配合team-specific callback欄位。
11. fixed `externalMtlf` selection/config與舊runtime wiring移除；標準ML Model Provision consumer保留或
    依service語意重構成NWDAF-to-NWDAF consumer，目前不接入production peer selection。
12. NWDAF Nsmf `UPF_EVENT`納入`internal/compat` inventory；R18 compat不得保留Release 19 BERMS欄位。
13. SMF static session resolution與go-upf standalone dataset replay都以顯式config／entrypoint隔離，
    預設normal production behavior不變。
14. standalone replay收到有效Nupf subscription後直接播放匹配dataset，不等待URR barrier；delete停止播放。
15. NWDAF NRF registration預設保持enabled；只有configured-endpoint portable scenario顯式關閉，
    且關閉時generic discovery不可使用。

開始實作前沒有新的產品／架構決策 blocker。若scoped generation要求大幅升級OpenAPI dependency，或
SMF/UPF portable contract與Release 18 OpenAPI有實質衝突，才需要停止該slice並提出新的決策。

---

## 13. Implementation And Verification Record

### 13.1 Runtime repositories

Closing implementation commits：

- NWDAF：`2bdb01e`
- PyAnLF：`2b43941`、`d616c06`、`694e3d8`
- PyMTLF：`94f3304`、`11b3199`
- nwdaf-resources：`4937b20`
- team SMF local experiment revision：`08c7402`
- team go-upf local experiment revision：`2a9a004`

實作已完成下列責任收斂：

- NWDAF以`:8090` AnLF auxiliary edge及`:8091` MTLF auxiliary edge分離backend-originated routes；
  external SBI、NRF client/cache、SMF／ADRF標準consumer及process-local resource routing仍由Go負責。
- `internal/anlf/coordinator`、Go-owned Daisy／training／dataset retrieval／traffic Mongo runtime、
  fixed `externalMtlf` selection及被標準流程取代的custom report/model endpoints已移除。
- AnLF、MTLF、ADRF、ML Model及Nsmf流程需要而current generated module缺少的wire types已集中到
  `internal/compat/<service>`；`internal/mlmodel`與散落的重複wire definitions已移除。
- 標準ML Model Provision peer consumer以NWDAF service語意保留；成功create必須驗證
  `201 + Location`，不再以notification ID組出非標準fallback resource URI。
- PyAnLF已移除未掛載的legacy analytics router，並修正model provision與Go sync同時發生時的
  ownership race，以及UPF notification早於SMF association metadata時的ADRF pending retry。
- PyMTLF不再因相同Go sync snapshot重建既有provision revision或重送同一model version；
  sync log明確記錄接受的`trainingDataSource`。

### 13.2 Team SMF、go-upf與portable deployment

- team SMF已使用repository-local Release 18 Nsmf/Nupf compatibility types及bounded raw HTTP client，
  把Nsmf top-level`notifUri/notifId`映射為Nupf
  `eventNotifyUri/notifyCorrelationId`，並保留`201 + Location`、terminal delete cascade與peer
  `ProblemDetails`分類。
- SMF static session resolution預設關閉；啟用時只替代active PDU-session lookup，後續仍經相同
  Nupf request builder、resource repository及delete path。
- team go-upf已有獨立`cmd/ees-replay`，不初始化PFCP、gtp5g或GTP-U；它重用正常Nupf router及
  subscription store，依target UE IP隔離Parquet records，且delete、shutdown與terminal failure會停止replay。
- 跨process orchestration已由舊的單一test script搬到
  `nwdaf-resources/deployments/portable_event_exposure/`；runner不修改任何sibling repository，
  並在temporary directory產生binary、config、Mongo data、artifact與logs。
- `components.yaml`已記錄remote、branch、implementation base、contract、required files與實際本地commit：
  team SMF `08c740203720b33dbe794e49af6df1051f97ee25`、team go-upf
  `2a9a00482fed7013f27077ac8329d14804b87c17`。兩者是本地實驗用魔改版本，不要求push；preflight會要求
  sibling repository的branch及HEAD精確符合lock，不會以dirty worktree或遠端branch名稱代替revision。

### 13.3 Verified portable sequence

2026-07-26已以真實team SMF、team go-upf replay、NWDAF、PyAnLF、PyMTLF、ADRF、Mongo與consumer
通過`portable application E2E`：

1. Consumer建立NWDAF Events Subscription。
2. PyAnLF經Go建立Nsmf resource；SMF static resolver建立真實Nupf resource。
3. go-upf只重播matching SUPI／UE IP資料並直接通知PyAnLF。
4. ADRF可用時PyAnLF只寫ADRF；PyMTLF經retrieval subscription、callback及direct GET取得dataset，
   完成local training及generation 2 activation。
5. ADRF停止後PyAnLF切換Mongo；training datasource經Go sync傳到PyMTLF，完成Mongo-backed training及
   generation 3 activation。
6. ADRF恢復後只接收future writes，不回填Mongo gap。
7. Consumer刪除subscription後，Nsmf delete cascade到Nupf並停止replay。

Runner最終輸出：

```text
PASS: Consumer -> NWDAF -> AnLF backend -> SMF -> go-upf direct notification -> ADRF training -> model activation -> Mongo fallback training -> ADRF future-write recovery -> terminal delete
```

### 13.4 Verification results

| Repository | Result |
|---|---|
| NWDAF | `golangci-lint run ./...` 0 issues；`go test ./...`、affected packages `go test -race`及`go build ./...`通過 |
| PyAnLF | Ruff lint／format及full pytest通過：235 passed、1 skipped |
| PyMTLF | Ruff lint／format及full pytest通過：93 passed；只有既有dependency warnings |
| ADRF | `go test ./...`通過，並由portable runner驗證真實storage/retrieval lifecycle |
| team SMF | Event Exposure、context、consumer、processor、factory focused race tests及build通過；真實binary通過portable E2E |
| team go-upf | `internal/ees` race tests、standalone replay build及new-code lint通過；真實binary通過portable E2E |
| nwdaf-resources | portable scripts Ruff lint／format通過；既有model-monitor regression及portable application E2E通過 |

team SMF的repository-wide test仍包含既有`internal/pfcp/handler` log/index parsing failures，且目前
`golangci-lint` binary以Go 1.25執行時無法載入該module宣告的Go 1.26。go-upf repository-wide test仍包含
既有root test unused import，以及需要gtp5g／network privilege的data-plane tests。這些路徑沒有被本次
修改，affected packages與本階段portable application path均已通過；不得把這些結果寫成full repository
pass，也不得把portable結果誤稱為privileged full-core E2E。

Optional NRF scenario與privileged full-core substitution未執行，依第7.2、7.3及9.5節不阻擋configured
endpoint portable acceptance，也不代表NRF cache、active PDU session、PFCP、URR或gtp5g已被驗證。
