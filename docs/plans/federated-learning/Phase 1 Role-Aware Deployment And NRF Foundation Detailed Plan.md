# Phase 1 Role-Aware Deployment And NRF Foundation Detailed Plan

日期：2026-07-28

狀態：詳細計畫完成，待實作

上層計畫：

- [Distributed NWDAF Federated Learning Implementation Plan](./Distributed%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)

相關設計、規格解讀與政策：

- [Distributed NWDAF Model Monitoring and Federated Retraining Architecture](../../design/Distributed%20NWDAF%20Model%20Monitoring%20And%20Federated%20Retraining%20Architecture.md)
- [NWDAF AnLF／MTLF Current Feature Architecture](../../design/Current%20AnLF%20MTLF%20Feature%20Architecture.md)
- [NWDAF Federated Learning Release 18 規格解讀](../../specification-guides/NWDAF%20Federated%20Learning%20Release%2018%20規格解讀.md)
- [Phase 0 Release 18 Contract Foundation Detailed Plan](./Phase%200%20Release%2018%20Contract%20Foundation%20Detailed%20Plan.md)
- [NWDAF Development Policy](../../development_policy.md)

實作 repositories：

- `NWDAF/`
- `PyAnLF/`
- `PyMTLF/`
- `nrf/`
- `nwdaf-resources/`
- `nwdaf-docs/`

---

## 1. 目的

Phase 1 要把目前「單一 NWDAF、能力主要由 backend enable 狀態推導」的
部署基線，改成可用相同 binary 啟動 A、B、C 三個角色，並讓 NRF 能
lossless 保存、回傳及篩選 Release 18 NWDAF model／FL capability。

本 Phase 完成後必須能證明：

1. A、B、C 使用各自獨立且穩定的 `nfInstanceId` 向 NRF 註冊；
2. A、B 只宣告自己真正提供的 analytics／monitor service，並宣告
   `FL_CLIENT`、TAI 與 local training data capability；
3. C 不宣告 Events Subscription，僅宣告目前已實作的 Model Provision
   與 Model Monitor service，並宣告 `FL_SERVER`；
4. Release 18 `mlAnalyticsList` 從 NWDAF 送出、經 NRF 保存、再由
   NFDiscovery 回傳時不丟欄位；
5. NRF 依同一個 `MlAnalyticsInfo` entry 的 Analytics ID、TAI、
   interoperability、FL capability 與 data-source fields 正確 matching；
6. Go NWDAF 提供一個共用、standard-shaped 的 NRF discovery auxiliary
   edge，PyAnLF／PyMTLF 不各自建立另一套 NRF API；
7. discovery cache 的 key、expiry、singleflight 與 raw Release 18 response
   preservation 對完整 query 正確；
8. `nwdaf-resources` 能啟動 NRF 與三個角色，並以黑箱查詢驗證 profile
   與 discovery assertions。

Phase 1 不建立跨 NWDAF Model Provision resource、不建立 Training public
route，也不執行 FL round。它提供後續 Phase 2 與 Phase 3 可直接使用的
角色、profile、discovery 與部署基礎。

---

## 2. Branch、baseline 與 repository 邊界

### 2.1 Branch strategy

本 federated-learning workstream 不再為每個 Phase 建立短期 branch。
所有 FL implementation 在各 repository 的長期 feature branch 上累積：

| Repository | Branch | 用途 |
| --- | --- | --- |
| `NWDAF/` | `feat/r18-federated-learning` | Go NWDAF profile、discovery gateway 與後續 FL SBI |
| `PyAnLF/` | `feat/r18-federated-learning` | AnLF role config、provider discovery 與後續 remote model flow |
| `PyMTLF/` | `feat/r18-federated-learning` | MTLF execution mode 與後續 FL server／client |
| `nwdaf-resources/` | `feat/r18-federated-learning` | multi-instance deployment、contract assertions 與 E2E |
| `nrf/` | `feat/r18-nwdaf-discovery` | NRF Release 18 NWDAF／ADRF registration and discovery extension |
| `nwdaf-docs/` | `main` | canonical plans、design、specification guides 與進度記錄 |

約束：

- 不建立 `contract-foundation`、`phase1` 或其他以 Phase 命名的 implementation
  branch；
- implementation commit 仍使用 `<type>(<scope>): <summary>`，summary
  不放 Phase、review iteration 或 finding ID；
- 每個 repository 獨立 commit、獨立驗證，不跨 repository 混合 commit；
- `nrf/` 是可修改的 team fork；`resources/references/free5gc-main/NFs/nrf/`
  仍只是 read-only reference；
- 本 Phase 不修改 `resources/` 內的 mirror、ADRF、SMF、UPF 或 UDM。

### 2.2 Current baseline

| Repository | Branch | Baseline |
| --- | --- | --- |
| `NWDAF/` | `feat/r18-federated-learning` | `8842d49` |
| `PyAnLF/` | `feat/r18-federated-learning` | `78fb431` |
| `PyMTLF/` | `feat/r18-federated-learning` | `d102aa7` |
| `nwdaf-resources/` | `feat/r18-federated-learning` | `57db274` |
| `nrf/` | `feat/r18-nwdaf-discovery` | `8e567d4` |
| `nwdaf-docs/` | `main` | `751b8a2` |

計畫撰寫時上述 worktree 均為 clean。開始實作前仍須在各 repository
重新確認 status，保留所有非本工作單元的使用者改動。

### 2.3 Primary and secondary exemplars

本 Phase 的 Go/free5GC shape 依下列優先順序：

1. primary：可修改的 `nrf/` 現有 handler、processor、context、MongoDB
   與 service lifecycle；
2. primary：`NWDAF/` 現有 NF registration、discovery cache、auxiliary
   servers 與 config validation；
3. secondary：`resources/references/free5gc-main/NFs/udm/` 的 outbound
   NRF consumer 與 config shape；
4. normative contract：local Release 18 OpenAPI／TS；
5. generated type reference：`resources/openapi/openapi/`。

不因 upstream exemplar 使用 Release 17 generated type，就容許 Release 18
欄位在本工作流中被無聲丟棄。

---

## 3. Normative evidence

### 3.1 NF Profile and capability

主要 Stage 3 證據：

- [TS 29.510 Nnrf NFManagement OpenAPI](../../../specs/openapi/TS29510_Nnrf_NFManagement.yaml)
- [TS 29.510 NwdafInfo](../../../specs/TS%2029.510/6%20API%20Definitions/6.1%20Nnrf_NFManagement%20Service%20API/6.1.6%20Data%20Model/6.1.6.2%20Structured%20data%20types/6.1.6.2.45%20Type%20NwdafInfo.md)
- [TS 29.510 MlAnalyticsInfo](../../../specs/TS%2029.510/6%20API%20Definitions/6.1%20Nnrf_NFManagement%20Service%20API/6.1.6%20Data%20Model/6.1.6.2%20Structured%20data%20types/6.1.6.2.84%20Type%20MlAnalyticsInfo.md)
- [TS 29.510 FlCapabilityType](../../../specs/TS%2029.510/6%20API%20Definitions/6.1%20Nnrf_NFManagement%20Service%20API/6.1.6%20Data%20Model/6.1.6.3%20Simple%20data%20types%20and%20enumerations/6.1.6.3.19%20Enumeration%20FlCapabilityType.md)
- [TS 29.510 AdrfInfo](../../../specs/TS%2029.510/6%20API%20Definitions/6.1%20Nnrf_NFManagement%20Service%20API/6.1.6%20Data%20Model/6.1.6.2%20Structured%20data%20types/6.1.6.2.122%20Type%20AdrfInfo.md)

`NwdafInfo.nwdafEvents` 表達 Events Subscription analytics capability；
`NwdafInfo.mlAnalyticsList` 表達 Model Provision／FL 所使用的 ML
Analytics capability。兩者不得因為都包含 `UE_COMMUNICATION` 就合併成
同一個 config flag。

`MlAnalyticsInfo` 的 relevant fields 為：

| Field | Phase 1 use |
| --- | --- |
| `mlAnalyticsIds` | 支援的 model／FL Analytics ID |
| `snssaiList` | optional model／training scope |
| `trackingAreaList` | A、B 各自可服務／訓練的 TAI |
| `mlModelInterInfo` | model interoperability matching |
| `flCapabilityType` | `FL_SERVER`、`FL_CLIENT` 或 combined |
| `flTimeInterval` | optional FL availability window；第一版不主動設定 |
| `nfTypeList` | local training data 的 NF type；第一版使用 `UPF` |
| `nfSetIdList` | optional local training data NF Set；第一版不主動設定 |

TS 29.510 明確要求：同一 `MlAnalyticsInfo` 內的 Analytics IDs 應共享相同
attribute values。若 attribute 不同，必須拆成另一個 entry。因此 config
與 matcher 都以「entry」為不可拆散的 matching unit。

### 3.2 NF registration

`PUT /nnrf-nfm/v1/nf-instances/{nfInstanceID}`：

| Situation | Required behavior |
| --- | --- |
| new profile | `201` + required `Location` + full `NFProfile` |
| profile replacement | `200` + full `NFProfile` |
| malformed／invalid profile | declared `4xx` `ProblemDetails` |
| temporary／permanent redirect | `307`／`308` + required `Location` |
| NRF failure／unavailable | declared `5xx` `ProblemDetails` |

Path `nfInstanceID` 必須與 body `nfInstanceId` 相同。Phase 1 保留 NWDAF
既有 startup registration retry、OAuth-aware lifecycle、shutdown
deregistration 與 listener ordering；只更換 lossless profile contract，
不得繞過既有 lifecycle。

### 3.3 NF discovery

主要證據：

- [TS 29.510 Nnrf NFDiscovery OpenAPI](../../../specs/openapi/TS29510_Nnrf_NFDiscovery.yaml)
- [TS 29.510 Search NF Instances GET](../../../specs/TS%2029.510/6%20API%20Definitions/6.2%20Nnrf_NFDiscovery%20Service%20API/6.2.3%20Resources/6.2.3.2%20Resource%20nf-instances%20(Store)/6.2.3.2.3.1%20GET.md)
- [TS 29.510 NFDiscovery supported features](../../../specs/TS%2029.510/6%20API%20Definitions/6.2%20Nnrf_NFDiscovery%20Service%20API/6.2.9%20Features%20supported%20by%20the%20NFDiscovery%20service.md)

`GET /nnrf-disc/v1/nf-instances`：

- required：`target-nf-type`、`requester-nf-type`；
- optional：`requester-nf-instance-id`、`service-names`、
  `target-nf-instance-id`、`nwdaf-event-list`、
  `ml-analytics-info-list`、`internal-group-identity`、
  `ml-model-storage-ind`、`data-storage-ind` 等；
- success：`200 application/json`，body `SearchResult`；
- `SearchResult.validityPeriod` 與 `nfInstances` required；
- redirect：`307`／`308` + required `Location`；
- malformed query 使用 declared `4xx` `ProblemDetails`；
- system／availability failure 使用 declared `5xx` `ProblemDetails`。

Encoding：

- `service-names`、`nwdaf-event-list` 為 `style=form, explode=false`；
- `ml-analytics-info-list` 使用 JSON query content；
- complex object／array 不可用逗號拆字串模擬。

Matching：

- query list 的多個 `MlAnalyticsInfo` 是 OR，profile 支援其中至少一個
  query entry 即可；
- 同一 query entry 內出現的多個 attributes 是 AND；
- 每個 array attribute 使用 overlap semantics；
- 同一個 profile `MlAnalyticsInfo` entry 必須同時滿足該 query entry
  的所有 attributes，不得由兩個不同 profile entries 拼湊。

Feature evidence：

- feature 17 `Query-eNA-PH2` 涵蓋 `ml-analytics-info-list`；
- feature 36 `Query-eNA-PH3` 涵蓋 `ml-model-storage-ind` 與
  `data-storage-ind`；
- NRF 透過 `SearchResult.nrfSupportedFeatures` 表達支援；
- implementation 使用 TS 29.510 feature numbering 產生 bitmask，不使用
  自訂字串。

---

## 4. 現況盤點與直接缺口

### 4.1 Go NWDAF

已具備：

- startup registration／deregistration；
- generated NFManagement／NFDiscovery clients；
- OAuth-aware NRF calls；
- discovery validity cache、singleflight 與 bounded eviction；
- AnLF／MTLF 共用 path
  `GET /internal/v1/nrf/nf-instances`；
- raw `SearchResult` capture，避免 unknown Release 18 response subtree 被
  generated decoder 丟棄；
- Phase 0 `internal/compat/nrf` profile wrapper。

缺口：

- `ConfigureNFManagement` 仍回傳 Release 17 generated profile；
- registration client 仍以 generated request body送出 profile；
- service advertisement 仍由 AnLF／MTLF backend enable 組合推導，
  尚未像其他 free5GC NF 一樣由 `serviceNameList`明確控制；
- `nfInstanceId` 每次 process start 隨機產生，三角色 deployment 無法
  在 config 固定 identity；
- `NFDiscoveryQuery` 只包含 target、requester 與 service names；
- cache key 只包含上述少數欄位；
- auxiliary validation 只允許 SMF／ADRF；
- AnLF 與 MTLF 的相同 validation logic 有重複；
- current profile 尚不能宣告完整 FL fields。

### 4.2 NRF

已具備：

- free5GC-style handler／processor／MongoDB persistence；
- registration create／replacement；
- GET、PATCH、DELETE profile lifecycle；
- existing target type、requester type、service name、exact instance 與多種
  legacy query filters；
- `validityPeriod=100` 的 basic `SearchResult`。

缺口：

- registration ingress 先 decode 成 Release 17 profile，後續 normalized
  profile 再寫入 MongoDB，Release 18 FL fields 因此遺失；
- discovery response decode 成 Release 17 discovery profile，再次遺失
  fields；
- target／requester NF type validation allowlist 尚無 `ADRF`；
- `nwdaf-event-list` 雖存在於舊 query struct，沒有完成 matching；
- 沒有解析或 matching `ml-analytics-info-list`；
- 沒有 `ml-model-storage-ind`／`data-storage-ind` matching；
- malformed complex query 多處只記 log，不一定回標準 `400`；
- response 未宣告 `nrfSupportedFeatures`；
- `SearchNFInstances` 仍以單一大型 `buildFilter` 混合 parsing 與 Mongo
  predicate construction，不適合直接塞入同-entry FL matching。

### 4.3 PyAnLF

已具備：

- containing Go sync projection；
- SMF／ADRF 的 NRF discovery client；
- fixed endpoint／NRF mode；
- validity-based local candidate cache；
- model demand 與現有 containing-Go Model Provision auxiliary calls。

缺口：

- 尚無 remote model provider resolver；
- discovery request 尚未使用完整 generic query contract；
- config 尚未表達 A／B role、remote provider mode、provider matching
  scope 與 cross-instance artifact origin；
- provider discovery 結果尚無明確 selected-provider state。

Phase 1 只建立 resolver／selection foundation；把 Model Provision create
真正送往 remote C 是 Phase 2。

### 4.4 PyMTLF

已具備：

- current local model provider／monitor／retraining pipeline；
- ADRF discovery；
- health／sync；
- config validation、artifact storage 與 training policy。

缺口：

- lifespan 沒有明確 `local`／`fl_server`／`fl_client` mode；
- C 收到 degradation 時仍可能進 current local trainer；
- A／B 的 client-only process 尚無清楚的 mode boundary；
- FL workspace、public artifact origin 與 cleanup config 尚未建立。

Phase 1 只建立 execution mode 與 config/lifecycle boundary；Training
resource、preparation worker 與 round worker 分別在 Phase 3、4 實作。

### 4.5 nwdaf-resources

已具備 single-NWDAF portable deployment，尚無：

- 三份 Go NWDAF config；
- A／B／C backend configs；
- team NRF component pin；
- isolated ports、data directories 與 logs；
- registration／discovery assertion runner。

---

## 5. Phase 1 scope

### 5.1 必須完成

- role-aware Go config 與 validation；
- configurable stable `nfInstanceId`；
- A／B／C NF Profile builder；
- lossless Release 18 NF registration request／response；
- NRF lossless registration／persistence／read／discovery response；
- standard `MlAnalyticsInfo` same-entry matcher；
- generic Go NRF discovery query、serialization、cache key；
- AnLF／MTLF 共用 auxiliary validation／forwarding；
- PyAnLF provider discovery foundation；
- PyMTLF execution modes；
- A／B／C deployment configs與 Phase 1 black-box assertions；
- config、API、design／plan progress documentation。

### 5.2 明確不做

- 不建立 `nnwdaf-mlmodeltraining` public route；
- 不在 NRF profile 廣告尚未實作的 Model Training service；
- 不建立跨 NWDAF Provision／Monitor resource route；
- 不開始 preparation、local FL training、FedAvg 或 final validation；
- 不修改 WAPE、degradation 或既有 local training算法；
- 不建立 ADRF model publication；
- 不建立 UDM group resolution 或 SMF AoI gating；
- 不新增 NRF NF status subscription；
- 不把 discovery cache 放進 backend sync；
- 不持久化 Go discovery cache；
- 不實作 NRF priority／capacity selection；
- 不加入 TLS／OAuth delegation 給 Python；
- 不修改 ADRF、SMF、UPF、UDM 或 `resources/` mirrors。

### 5.3 Truthful-advertisement correction

Phase 0 尚未建立 Training route；完整 Training SBI 在 Phase 3 才實作。
因此 Phase 1：

- A／B profile 會宣告 `FL_CLIENT`；
- C 可透過 `ml-analytics-info-list` 發現 A／B；
- A／B **不**先廣告 `nnwdaf-mlmodeltraining`；
- Phase 3 在 handler、processor、backend routing 與 resource semantics
  可用後，才把該 service 加入 A／B profile；
- Phase 3 之後的 FL Client discovery 才同時要求
  `FL_CLIENT` capability 與 registered Training service。

這避免 NRF 回傳一個實際呼叫只能得到 route-not-found／placeholder error
的虛假 service。

---

## 6. Target roles and profiles

### 6.1 Deployment role matrix

| Capability | NWDAF-A | NWDAF-B | NWDAF-C |
| --- | --- | --- | --- |
| Events Subscription provider | yes | yes | no |
| Model Provision provider | no | no | yes |
| Model Monitor reporting side | yes | yes | no |
| Model Monitor coordination side | no | no | yes |
| FL Client | yes | yes | no |
| FL Server | no | no | yes |
| PyAnLF process | yes | yes | no |
| PyMTLF process | yes, `fl_client` | yes, `fl_client` | yes, `fl_server` |
| TAI | TAI-A | TAI-B | omitted／any |
| local training data NF type | `UPF` | `UPF` | omitted |

`local` PyMTLF mode 保留給既有 single-NWDAF regression，不屬於 A／B／C
第一版 topology。

### 6.2 Phase 1 advertised services

| Instance | Registered services in Phase 1 |
| --- | --- |
| A | `nnwdaf-eventssubscription`, `nnwdaf-mlmodelmonitor` |
| B | `nnwdaf-eventssubscription`, `nnwdaf-mlmodelmonitor` |
| C | `nnwdaf-mlmodelprovision`, `nnwdaf-mlmodelmonitor` |

`nnwdaf-mlmodeltraining` 在 Phase 3 才加入 A／B。

Model Monitor 的 registration 與 subscription 是同一個標準 service。
NRF 的 `NFService` profile只能宣告 service name、version與 endpoint，
不能分別宣告只支援 `/registrations`或 `/subscriptions`。第一版
deployment接受這項 service-granularity限制：

- A／B 的 PyAnLF owner 處理 C 建立的 monitor subscription；
- C 的 PyMTLF owner 處理 A／B 建立的 monitor registration；
- C 不為未使用的 subscription方向額外啟動 PyAnLF；
- 若 request送到該 instance沒有 runtime owner的 operation，依
  ML Model Monitor OpenAPI已宣告的 `503`回應及既有 backend
  availability policy處理；
- 這是第一版刻意接受的限制，不代表 NRF能表達 operation-level
  capability；
- Go 不替缺少的 backend 實作 monitor business logic。

### 6.3 Profile semantics

A：

```json
{
  "nwdafEvents": ["UE_COMMUNICATION"],
  "mlAnalyticsList": [{
    "mlAnalyticsIds": ["UE_COMMUNICATION"],
    "trackingAreaList": ["TAI-A"],
    "mlModelInterInfo": {"vendorList": ["<configured-vendor-id>"]},
    "flCapabilityType": "FL_CLIENT",
    "nfTypeList": ["UPF"]
  }]
}
```

B 與 A 相同，`trackingAreaList` 改為 TAI-B。

C：

```json
{
  "mlAnalyticsList": [{
    "mlAnalyticsIds": ["UE_COMMUNICATION"],
    "mlModelInterInfo": {"vendorList": ["<same-vendor-id>"]},
    "flCapabilityType": "FL_SERVER"
  }]
}
```

上例只顯示語意，不取代正式 `NFProfile` JSON。實際 TAI 使用標準
`Tai {plmnId, tac}`，Vendor ID 依 TS 29.510 使用六位數字字串。

C 不含 `nwdafEvents`，因此不應出現在
`nwdaf-event-list=UE_COMMUNICATION` 的 analytics-provider discovery。

### 6.4 NF identity

Go config 新增 optional `nfInstanceId`：

- 有設定時必須是 UUIDv4；
- A／B／C deployment config 必須設定三個不同 UUIDv4；
- 未設定時保留現況，由 process 啟動時產生 UUIDv4，維持 local
  development backward compatibility；
- identity 不由 PyAnLF／PyMTLF 產生，也不透過 handshake 覆蓋；
- sync snapshot 繼續由 Go 把 containing NWDAF identity 投影給 backend；
- Phase 1 不額外持久化 Go state，但同一份 deployment config 重啟時會
  使用相同 NF identity。

---

## 7. Configuration contract

### 7.1 Go NWDAF

對齊 AMF、SMF、NSSF 等 free5GC NF 的 config 形狀：

- `serviceNameList` 是本 instance 實際提供的標準 SBI service清單；
- NF-specific capability直接使用 `nwdafInfo`表達；
- `anlfBackend`／`mtlfBackend`只描述本地 dependency與連線方式；
- 啟動時由同一份 config同時建立 routes與 NRF `NFProfile`，避免廣告與
  實際服務分離。

A 的設定範例：

```yaml
configuration:
  nfInstanceId: "11111111-1111-4111-8111-111111111111"

  serviceNameList:
    - nnwdaf-eventssubscription
    - nnwdaf-mlmodelmonitor

  nwdafInfo:
    nwdafEvents:
      - UE_COMMUNICATION
    mlAnalyticsList:
      - mlAnalyticsIds:
          - UE_COMMUNICATION
        trackingAreaList:
          - plmnId:
              mcc: "466"
              mnc: "92"
            tac: "000001"
        mlModelInterInfo:
          vendorList:
            - "001122"
        flCapabilityType: FL_CLIENT
        nfTypeList:
          - UPF

  anlfBackend:
    enabled: true
    endpoint: "http://127.0.0.1:<anlf-port>"

  mtlfBackend:
    enabled: true
    endpoint: "http://127.0.0.1:<mtlf-port>"
```

Structured fields使用與 Release 18 wire schema一致的名稱與形狀。
Factory types可直接使用 Phase 0 compatibility types，或建立只做 YAML
decode的薄 wrapper，但不得再建立
`analyticsProvider.enabled`、`federatedLearning.server`等平行語意。

`serviceNameList`不是單純的 NRF metadata。與其他 free5GC NF一致，它
同時控制：

1. 哪些 public SBI route groups會掛載；
2. 哪些 `NFService`會放進 registration profile；
3. 啟動 validation需要哪些 local backend owner。

Model Monitor 的兩種 operation方向不能由 NRF分開廣告，因此 config也
不建立假的 `reportingSide`／`coordinationSide` profile欄位。A／B 與 C
都列出同一個 `nnwdaf-mlmodelmonitor`；runtime依實際 backend owner
routing，非預期方向依標準宣告的 `503`處理。

### 7.2 Deployment config files

保留 `NWDAF/config/nwdafcfg.yaml` 作既有 single-NWDAF／local regression
設定，並在同一目錄新增：

- `nwdafcfg-a.yaml`：TAI-A Analytics provider、Monitor subscription
  owner、FL Client；
- `nwdafcfg-b.yaml`：TAI-B Analytics provider、Monitor subscription
  owner、FL Client；
- `nwdafcfg-c.yaml`：Model Provision provider、Monitor registration
  owner、FL Server，不啟用 PyAnLF。

三份設定使用相同 factory schema，不建立 A／B／C專用 Go types。不同
instance的 SBI address、backend endpoint、NF instance ID與 TAI由各檔案
提供。範例設定必須可以個別通過 config validation；使用者可複製後只
修改部署位址與實驗範圍。

### 7.3 Go validation

Validation 至少涵蓋：

- `nfInstanceId` UUIDv4；
- `serviceNameList`只接受 current runtime真正實作的 service；
- duplicate service name拒絕；
- `nwdafEvents`只接受 current runtime真正支援的
  `UE_COMMUNICATION`；
- `nnwdaf-eventssubscription` 必須有 enabled AnLF backend及非空
  `nwdafEvents`；
- `nwdafEvents`非空時必須列出 `nnwdaf-eventssubscription`；
- `nnwdaf-mlmodelprovision` 必須有 enabled MTLF backend及非空
  `mlAnalyticsList`；
- `nnwdaf-mlmodelmonitor` 至少有 enabled AnLF或 MTLF backend；
- `FL_CLIENT`、`FL_SERVER` 或 `FL_SERVER_AND_CLIENT` entry必須有
  enabled MTLF backend；
- `mlAnalyticsList` present時每個 entry至少有一個
  `mlAnalyticsIds`；
- TAI 的 MCC、MNC、TAC 使用 standard nested type validation；
- Vendor ID 為六位數字；
- local training data NF types 不得在非 FL entry 中誤用；
- profile 宣告的 Analytics ID 必須由 runtime allowlist支援；
- duplicate equivalent `MlAnalyticsInfo` entries 在 config normalization
  後拒絕，避免 profile ambiguity；
- C 可無 AnLF backend，且不得宣告 Events Subscription；
- registration disabled 的 local test仍可建立 profile，但不送往 NRF。

Defaults：

- legacy config 同時未提供 `serviceNameList`與 `nwdafInfo`時，保留目前
  single-NWDAF capability推導作為 compatibility path，並記一筆
  deprecation warning；
- A／B／C deployment 必須使用 explicit profile，不走 legacy default；
- 後續移除 legacy inference 另列 cleanup，不在 Phase 1 強迫所有既有
  local configs同步重寫。

### 7.4 PyMTLF

新增：

```yaml
runtime:
  mode: "local"  # local | fl_server | fl_client

federated_learning:
  workspace_root: "data/fl-workspaces"
  workspace_ttl_seconds: 3600
  public_base_url: "http://127.0.0.1:<instance-port>"
  artifact_download:
    allowed_origins:
      - "http://127.0.0.1:<peer-port>"
    timeout_seconds: 300
```

Lifecycle：

- `local`：完全保留 current local retraining coordinator；
- `fl_server`：啟動 model provision、monitor policy與未來 server owner；
  degradation 只建立／保留 remote-FL intent，不得呼叫 current local
  trainer；
- `fl_client`：啟動 client role所需 catalog／artifact／dataset foundation，
  不啟動 server degradation coordinator，也不啟動尚未實作的 round
  worker；
- 不建立回 `501` 的 placeholder Training endpoint；
- unknown mode、不可寫 workspace、非 absolute public base URL、
  invalid timeout／TTL 在 startup fail-fast；
- data directory仍由 `.gitignore` 排除。

### 7.5 PyAnLF

新增 model provider discovery config：

```yaml
model_provider:
  mode: "nrf"  # nrf | configured
  configured_endpoint: ""
  discovery_timeout_seconds: 30
  retry_initial_interval_seconds: 2
  retry_max_interval_seconds: 30
  analytics_id: "UE_COMMUNICATION"
  model_interoperability_vendor_ids:
    - "001122"
```

行為：

- `configured` mode只驗證並回傳固定 NWDAF API root；
- `nrf` mode透過 containing Go 的共用 discovery edge 查詢 C；
- candidate 必須同時具有 matching `mlAnalyticsList` 與 registered
  `nnwdaf-mlmodelprovision` service；
- 第一版 deterministic selection 為
  `(nfInstanceId, serviceInstanceId, apiRoot)` 排序後第一個；
- 不使用 priority、capacity 或 load；
- discovery failure不刪除仍在 validity period內的有效 selection；
- expiry 後重新查詢；empty result是「目前沒有 provider」，不是 malformed
  response；
- selected state保存 provider NF instance ID、service instance ID、API root、
  discovery expiry；
- Phase 1 不使用 selected state建立 remote resource；Phase 2 接入。

### 7.6 Timeout ordering

Phase 1 先建立可延伸的由內而外 timeout validation：

```text
NRF outbound request timeout
  < PyAnLF/PyMTLF discovery operation budget
  < Go backend-facing request budget
  < later cross-NWDAF process budget
```

本 Phase 不預先猜完整 FL round timeout，但：

- artifact download default 保留 300 秒，符合既有慢速網路決策；
- NRF discovery default 30 秒；
- config validation要求 retry initial不大於 retry max；
- shutdown wait必須可被 context cancellation中斷；
- 不用永久 blocking retry阻止 backend health endpoint啟動。

---

## 8. Go NF Profile and registration implementation

### 8.1 Profile ownership

`NWDAF/internal/compat/nrf.NFProfile` 成為 Release 18 wire profile owner：

- embed generated `models.NrfNfManagementNfProfile`；
- only replace `nwdafInfo`／`nwdafInfoList` with compatibility types；
- existing generated base fields與 NF services仍直接 reuse；
- 不建立 OpenAPI fork或 local `replace`；
- 不用 `map[string]any` 組 profile。

`NWDAFContext` 保存 compatibility profile，而不是只保存 generated
profile。提供：

- lossless profile snapshot給 registration consumer；
- generated base view給既有只需要 identity／OAuth 的程式；
- deep-copy／immutable snapshot semantics，避免 registration期間被其他
  lifecycle state mutation。

### 8.2 Profile builder

將目前長參數 `ConfigureNFManagement(...)` 改成 typed configuration input，
避免新增另一組 capability booleans：

```text
factory config
  -> normalized serviceNameList + NwdafInfo
  -> context profile builder
  -> compat/nrf.NFProfile
```

Builder：

1. 建立 NF identity、status、addresses；
2. 依 normalized `serviceNameList`加入 `NFService`；
3. 將 configured `nwdafEvents`放入 `NwdafInfo`；
4. 將 configured `mlAnalyticsList`保留為一或多個完整
   `MLAnalyticsInfo` entries；
5. 同 entry保留 Analytics ID、TAI、interoperability、FL capability、
   data-source fields；
6. 空的 `NwdafInfo` 不輸出；
7. omitted optional array不輸出 `[]`；
8. service instance IDs仍由 context建立並在 process lifetime保持穩定；
9. profile JSON不得同時出現 generated 與 compatibility duplicate fields。

### 8.3 Lossless registration transport

Generated NFManagement request type無法承載完整 R18 profile。Phase 1 在
`NrfService` 內建立隔離的 NFManagement registration transport：

- 沿用現有 HTTP/2 client、TLS verification、metrics transport、timeout、
  retry、redirect policy與 OAuth behavior；
- request method／path／media type完全依 OpenAPI；
- body由 `json.Marshal(compat/nrf.NFProfile)` 產生；
- success body先以 compatibility profile decode，再驗證 identity、type、
  status、heartbeat與 `Location`；
- `201` 要求 valid `Location`；`200` replacement可無 `Location`；
- error body仍 decode standard `ProblemDetails`；
- `307`／`308`不自動跟隨，保留既有 explicit redirect handling；
- 不為了送 R18 body而複製整套 generic HTTP stack。

現有 generated client registration tests改成 transport-level contract tests；
其他仍可完整表達的 NRF operation繼續使用 generated clients。

### 8.4 Lifecycle regression

必須保留：

- registration成功前 public listeners不對外服務；
- retryable server／transport failure使用 bounded exponential backoff；
- cancellation中斷 in-flight request與 retry wait；
- terminal `4xx`不重試；
- malformed success不被當作完整成功；
- remote可能已註冊但 local解析失敗時沿現有 cleanup語意；
- shutdown deregistration；
- backend availability與 sync不因 profile型別變更失效。

Phase 1 不新增 periodic heartbeat PATCH worker；現有 lifecycle能做什麼就
保持什麼，不在本 Phase 誤宣稱完成未存在的 heartbeat功能。

---

## 9. NRF discovery relay contract in NWDAF

本節的「relay」只表示 Python backend透過 containing Go NWDAF執行
`Nnrf_NFDiscovery`。Internal resource path可以由專案定義，但承載的
standard contract不得改名或重新塑形：

- query parameter名稱必須與 TS 29.510 OpenAPI完全相同；
- JSON property名稱、nested structure與 enum lexical value必須完全
  相同；
- standard request不得 flatten、重新分組或包進 project-specific
  request envelope；
- success body直接使用 standard `SearchResult`；
- error body使用 standard `ProblemDetails`；
- Go／Python內部 identifier可遵循各語言慣例，但 serializer tags與實際
  HTTP wire names不得因此改變；
- 不提供 deprecated alias、簡寫欄位或同義欄位，避免兩套 contract並存。

### 9.1 Query model

Go內部擴充 shared typed `NFDiscoveryQuery`，但它只是標準 query的
in-memory representation，不是另一套 wire schema：

```text
target NF type
requester NF type (Go-owned NWDAF)
requester NF instance ID (Go-owned)
target NF instance ID
service names
NWDAF event list
ML analytics info list
internal group identity
ML model storage indicator
data storage indicator
```

`MlAnalyticsInfo` 直接 reuse `internal/compat/nrf.MLAnalyticsInfo`。Query
model不得使用 raw `map[string]any`。

第一版 target matrix：

| Target | Typical filters | Owner |
| --- | --- | --- |
| `NWDAF` analytics provider | `service-names`, `nwdaf-event-list` | later analytics routing |
| `NWDAF` model provider | `service-names`, `ml-analytics-info-list` | PyAnLF |
| `NWDAF` FL Client | `ml-analytics-info-list`；Phase 3後再加 Training service | PyMTLF-C |
| `NWDAF` exact peer | `target-nf-instance-id`, `service-names` | later monitor routing |
| `UDM` | `service-names`, `internal-group-identity` | later group resolution |
| `SMF` | `service-names` | PyAnLF collection |
| `ADRF` data | `service-names`, `data-storage-ind=true` | PyAnLF／PyMTLF |
| `ADRF` model | `service-names`, `ml-model-storage-ind=true` | later PyMTLF publication |

### 9.2 Internal auxiliary edge

AnLF／MTLF兩個 auxiliary listeners保留相同 path：

```http
GET /internal/v1/nrf/nf-instances
```

Boundary rules：

- Python傳入 OpenAPI定義的 exact standard query parameter names；
- `ml-analytics-info-list`內的 property names與 enum values也必須保持
  OpenAPI原值，不接受 project-specific alias；
- Go固定 outbound `requester-nf-type=NWDAF`；
- Go固定 outbound `requester-nf-instance-id`為 containing NWDAF ID；
- migration期間 Python若帶 `requester-nf-type=NWDAF`可接受，但其他值
  回 `400`；Go不得照單轉發可偽造的 requester identity；
- target／filter combination使用共用 validator，AnLF與MTLF server不再
  各複製一份；
- 只允許上表明確支援的 standard query；
- internal API仍使用普通 HTTP，不增加 key、TLS或 OAuth；
- external NRF call仍由 Go執行。

Internal response：

| Result | Auxiliary response |
| --- | --- |
| valid NRF success | `200` + lossless `SearchResult`; `Cache-Control: max-age=<remaining>` |
| invalid internal query | `400` standard `ProblemDetails` |
| NRF redirect | same `307`／`308` + `Location` |
| NRF standard error | preserve status and `ProblemDetails` |
| containing NRF unavailable | `503` |
| transport／malformed upstream success | `502` |

### 9.3 Query serialization

Outbound builder以 `url.Values`／typed helpers明確處理：

- form arrays排序、deduplicate後以逗號分隔；
- `ml-analytics-info-list`先做 canonical typed JSON，再 URL encode；
- TAI、S-NSSAI、Vendor ID、NF type不拆散；
- boolean indicator只在 `true` 時送出；
- omitted與 explicit empty不是同一語意；empty array在 validation時拒絕；
- 不使用 string concatenation拼 URL。

至少保存以下 round-trip fixtures：

1. model provider query；
2. FL Client query含 TAI-A／TAI-B；
3. exact monitor peer query；
4. ADRF data storage query；
5. UDM internal group query。

### 9.4 Canonical cache key

Cache key包含：

- normalized NRF URI；
- requester NF instance ID；
- target／requester NF type；
- sorted unique service names；
- exact target instance ID；
- sorted NWDAF event list；
- canonical JSON `ml-analytics-info-list`；
- internal group identity；
- ADRF storage indicators。

Rules：

- array order不影響等價 query的 key；
- `MlAnalyticsInfo` entry內 object key order不影響 key；
- query entry list的 OR順序不影響 key；
- entry內 array order不影響 key；
- 不合併語意不同的 omitted與present values；
- cache仍以 NRF `validityPeriod`為 TTL；
- hit時回傳剩餘 validity，不回原始完整 TTL；
- `validityPeriod<=0`不 cache；
- singleflight使用同一 canonical key；
- bounded eviction與expired cleanup保留；
- 失敗 response不 cache；
- cache不進 sync，也不在 Go restart後恢復。

### 9.5 Response parsing

`NFDiscoveryResult`：

- generated `SearchResult`仍供既有 Go SMF caller使用；
- raw envelope仍供 backend pass-through；
- Phase 1額外提供 typed compatibility profile view給 Go內部 selection／test；
- `validityPeriod`與 `nfInstances`缺少或型別錯誤即視為 malformed `502`；
- `nrfSupportedFeatures`存在時解析並記錄 Query-eNA-PH2／PH3；
- NRF未宣告所需 feature時記錄清楚 diagnostic；本 team NRF acceptance要求
  兩個 feature都存在；
- 不因 generated decoder不認得 R18 fields而把 raw response覆寫掉。

---

## 10. NRF Release 18 implementation

### 10.1 Compatibility ownership

在 `nrf/internal/compat/nrf/` 建立 NRF-local typed compatibility models：

- management `NFProfile` wrapper；
- discovery `NFProfile` wrapper；
- `NwdafInfo`；
- `MlAnalyticsInfo`；
- `MlModelInterInfo`；
- `FlCapabilityType`；
- `AdrfInfo` fields若 pinned generated model不足則一併隔離補齊。

原則：

- reuse pinned generated base fields；
- 只補缺少／不完整的 Release 18 subtree；
- 不 import `NWDAF/internal/compat`，Go `internal` boundary不得跨 module；
- 不建立新的 shared OpenAPI fork；
- unknown future JSON field不等於支援；只保存 typed supported fields；
- enum parser對 future string可 round-trip，但 runtime matcher只對已知
  capability values提供明確語意。

### 10.2 Registration ingress and persistence

新的 pipeline：

```text
raw JSON
  -> compat NFProfile decode
  -> generated base validation
  -> R18 extension validation
  -> existing base normalization
  -> overlay validated R18 subtree
  -> typed JSON/BSON persistence
```

Requirements：

- raw body只讀一次；
- path/body identity一致；
- existing UUIDv4、NF type、NF status、address、service validation保留；
- `mlAnalyticsList` present時至少一項；
- 每個 list field present時至少一項；
- TAI、S-NSSAI、Vendor ID、time interval validation對齊 Phase 0 contract；
- `flTimeInterval`、`nfTypeList`、`nfSetIdList`只在有 FL capability時接受；
- combined server/client值接受；
- invalid R18 extension回 `400`，不得只 log後保存；
- Mongo document保留 standard JSON field names；
- `201`／`200` response用 compatibility profile輸出，不重新decode成 R17；
- GET profile round-trip保留 R18 fields；
- PATCH在套用後以完整 compatibility profile重新驗證；
- DELETE、URI list、OAuth certificate與 notification bookkeeping維持既有
  base profile view；
- profile-change notification若含 profile body，使用 lossless R18 body；
  若 existing generated consumer無法承載，建立隔離 transport adapter，
  不降級資料。

### 10.3 Discovery parsing

把 extended query parsing從大型 Mongo builder拆出：

```text
url.Values
  -> required/base validation
  -> typed ExtendedDiscoveryCriteria
  -> legacy/base Mongo candidate filter
  -> typed post-filter for R18 complex semantics
  -> lossless SearchResult
```

採 typed post-filter的理由：

- `ml-analytics-info-list`需要 outer OR、same-entry AND與array overlap；
- profile可使用 `nwdafInfo`或 `nwdafInfoList`；
- nested TAI／S-NSSAI／interoperability不適合用脆弱的 exact BSON
  subdocument equality；
- pure matcher可直接做 table-driven unit tests；
- first-version NRF profile量小，正確性優先於把所有 nested condition塞進
  Mongo query。

Base Mongo filter仍處理：

- target NF type；
- requester allowed NF type；
- service name與 `REGISTERED` status；
- exact target NF instance ID；
-既有已驗證的 legacy filters。

Typed post-filter處理本 Phase承諾的 complex fields。不得先把所有
profile取回後忽略 base filter。

### 10.4 Same-entry ML analytics matcher

Pure matcher：

```text
for each query entry:
  for each profile ML analytics entry:
    if every present query attribute matches this same profile entry:
      profile matches
```

Attribute semantics：

| Attribute | Match |
| --- | --- |
| `mlAnalyticsIds` | at least one common Analytics ID |
| `snssaiList` | at least one exact standard S-NSSAI |
| `trackingAreaList` | at least one exact standard TAI |
| `mlModelInterInfo.vendorList` | at least one common Vendor ID |
| `flCapabilityType=FL_SERVER` | profile is `FL_SERVER` or `FL_SERVER_AND_CLIENT` |
| `flCapabilityType=FL_CLIENT` | profile is `FL_CLIENT` or `FL_SERVER_AND_CLIENT` |
| `flCapabilityType=FL_SERVER_AND_CLIENT` | profile must be combined |
| `flTimeInterval` | Phase 1 profile可 lossless保存；第一版 query不使用 |
| `nfTypeList` | at least one common NF type |
| `nfSetIdList` | at least one common NF Set ID |

若 query attribute omitted，該 attribute不限制 matching。若 profile
attribute omitted但 query有要求，該 profile entry不 match。

Stage 2將 discovery輸入描述為「Time Period of Interest」，但本地
TS 29.510 matching table沒有像 array attributes一樣明確定義
`TimeWindow`究竟採 overlap或 full-containment。第一版 profile與 queries
都不設定 `flTimeInterval`；Phase 1若收到含該 field的 internal discovery
query，回 `400` unsupported query，而不是自行猜測、忽略或做不可靠
matching。未來啟用前須先在規格解讀／design中freeze語意。

Tests必須包含：

- query list outer OR；
- same-entry成功；
- Analytics ID在 entry-1、TAI在 entry-2時不得拼成成功；
- combined capability可滿足 server-only或client-only query；
- client-only不可滿足 server query；
- TAI-A／TAI-B各自只命中對應 profile；
- generic model provider未提供 TAI時，只有未要求 TAI的 query會命中；
- malformed／empty query list回 `400`。

### 10.5 Other extended matchers

- `nwdaf-event-list`只檢查 `NwdafInfo.nwdafEvents`，不可用
  `mlAnalyticsIds`替代；
- `internal-group-identity`依 target NF對應 standard profile subtree；
- `data-storage-ind=true`要求 target ADRF `adrfInfo.dataStorageInd=true`；
- `ml-model-storage-ind=true`要求
  `adrfInfo.mlModelStorageInd=true`；
- service filters仍要求 matching service `REGISTERED`；
- exact instance與其他 filters是 AND，不因指定 instance就跳過 capability
  validation；
- target NF type allowlist加入標準 `ADRF`。

### 10.6 SearchResult and supported features

Success response：

- `200 application/json`；
- `validityPeriod`沿用 config／current default，第一版可保持100秒；
- `nfInstances`為完整 lossless profiles；
- `nrfSupportedFeatures`宣告 feature 17與36；
- 沒有 match時回 `200` + empty `nfInstances`，不回 `404`；
- 保留 future `completeNfInstances`等欄位的現有行為邊界，不在 Phase 1
  假裝實作 stored search；
- response不把 Mongo internal fields外洩。

### 10.7 Error behavior

下列情況回 `400` `ProblemDetails`：

- missing required target／requester type；
- unsupported target／requester type；
- invalid JSON query content；
- empty present array；
- invalid UUID、TAI、S-NSSAI、Vendor ID、FL capability；
- unsupported filter／target combination；
- path/body profile identity mismatch；
- invalid R18 profile extension。

Mongo／internal decode failure回 `500`；不可把系統錯誤偽裝成 empty result。
既有 `ProblemDetails` helper沿用，cause採規格／free5GC既有語意，不新增
project-specific HTTP status。

---

## 11. Required discovery examples

### 11.1 A／B discover C model provider

```http
GET /nnrf-disc/v1/nf-instances
  ?target-nf-type=NWDAF
  &requester-nf-type=NWDAF
  &requester-nf-instance-id=<A-or-B>
  &service-names=nnwdaf-mlmodelprovision
  &ml-analytics-info-list=[
    {
      "mlAnalyticsIds":["UE_COMMUNICATION"],
      "mlModelInterInfo":{"vendorList":["001122"]}
    }
  ]
```

Expected：只回 C；A／B沒有 Model Provision service。

### 11.2 C discover eligible FL Clients

Phase 1不要求尚未廣告的 Training service：

```http
GET /nnrf-disc/v1/nf-instances
  ?target-nf-type=NWDAF
  &requester-nf-type=NWDAF
  &requester-nf-instance-id=<C>
  &ml-analytics-info-list=[
    {
      "mlAnalyticsIds":["UE_COMMUNICATION"],
      "trackingAreaList":[TAI-A,TAI-B],
      "mlModelInterInfo":{"vendorList":["001122"]},
      "flCapabilityType":"FL_CLIENT",
      "nfTypeList":["UPF"]
    }
  ]
```

因 array採 overlap semantics，A命中 TAI-A、B命中 TAI-B。C是
`FL_SERVER`，不命中。

Phase 3 route可用後，相同 query加入：

```text
service-names=nnwdaf-mlmodeltraining
```

### 11.3 Analytics provider query

```http
GET /nnrf-disc/v1/nf-instances
  ?target-nf-type=NWDAF
  &requester-nf-type=NWDAF
  &nwdaf-event-list=UE_COMMUNICATION
  &service-names=nnwdaf-eventssubscription
```

Expected：A、B；C不得出現。

### 11.4 Exact peer lookup

```http
GET /nnrf-disc/v1/nf-instances
  ?target-nf-type=NWDAF
  &requester-nf-type=NWDAF
  &target-nf-instance-id=<A>
  &service-names=nnwdaf-mlmodelmonitor
```

Expected：只回 A；若 A存在但 service未 registered，回 empty result。

### 11.5 ADRF capability

Data：

```text
target-nf-type=ADRF
service-names=nadrf-datamanagement
data-storage-ind=true
```

Model：

```text
target-nf-type=ADRF
service-names=nadrf-mlmodelmanagement
ml-model-storage-ind=true
```

Phase 1建立 query／NRF matcher；實際 ADRF model publication在 Phase 5。

---

## 12. nwdaf-resources deployment foundation

新增：

```text
deployments/distributed_fl/
├── README.md
├── components.yaml
├── config/
│   ├── nrf.yaml
│   ├── nwdaf-a.yaml
│   ├── nwdaf-b.yaml
│   ├── nwdaf-c.yaml
│   ├── pyanlf-a.yaml
│   ├── pyanlf-b.yaml
│   ├── pymtlf-a.yaml
│   ├── pymtlf-b.yaml
│   └── pymtlf-c.yaml
├── checks/
│   └── preflight.py
└── scripts/
    └── verify_role_discovery.py
```

實際檔名可依 repository既有 deployment convention微調，但責任不變。

Runner：

1. 驗證 component path、branch與 commit；
2. 建立 isolated temp root；
3. 產生每個 instance獨立 port、artifact root、workspace、log；
4. 啟動 temporary MongoDB；
5. 啟動 team NRF；
6. 啟動 PyMTLF-A／B／C與 PyAnLF-A／B；
7. 啟動 NWDAF-A／B／C；
8. 等待 registration與 backend health／sync；
9. GET NRF profiles，保存 sanitized JSON assertions；
10. 執行 §11 queries；
11. 驗證 cache hit與 expiry至少一個 black-box case；
12. graceful shutdown並確認 NWDAF deregistration。

Phase 1 runner不需要 ADRF、SMF、UPF或 consumer analytics subscription。

`components.yaml` pin：

- `NWDAF`；
- `PyAnLF`；
- `PyMTLF`；
- `nrf`；
- 不在 runner內自動 checkout、pull或修改 sibling repositories。

README明確標示：

- 這是 role/profile/discovery integration，不是 FL E2E；
- 不證明 remote Model Provision、Training或 FedAvg；
- 使用普通 HTTP；
- MongoDB只供 NRF persistence。

---

## 13. Detailed implementation slices

### Slice 1A：Role config and runtime modes

Repositories：

- `NWDAF/`
- `PyAnLF/`
- `PyMTLF/`

順序：

1. characterization current legacy config；
2. Go `nfInstanceId`與 explicit profile config；
3. Go validation與 role-to-service mapping tests；
4. PyMTLF mode parser／lifecycle gating；
5. PyAnLF provider resolver config與 selection state；
6. config docs／examples。

Exit：

- A／B／C config可獨立通過 validation；
- legacy single-NWDAF config仍通過；
- `fl_server`不觸發 local trainer；
- `fl_client`不啟動 server coordinator。

### Slice 1B：Lossless NWDAF registration

Repository：`NWDAF/`

順序：

1. context改存 compatibility profile；
2. typed profile builder；
3. isolated lossless registration transport；
4. success／error parsing；
5. lifecycle regression；
6. profile JSON golden assertions。

Exit：

- outgoing A／B／C body含完整 R18 fields；
- generated base fields與 services不受影響；
- registration retry／shutdown tests通過。

### Slice 1C：NRF profile persistence

Repository：`nrf/`

順序：

1. NRF-local compatibility types；
2. registration decode／validation；
3. typed persistence；
4. create／replace／GET／PATCH round-trip；
5. lossless registration response；
6. profile notification regression。

Exit：

- Mongo round-trip不丟 FL／ADRF fields；
- invalid extension回 `400`；
- existing R17 profile regression通過。

### Slice 1D：NRF discovery relay and cache

Repository：`NWDAF/`

順序：

1. typed query model；
2. shared AnLF／MTLF auxiliary validator；
3. query serializer；
4. complete canonical cache key；
5. typed/raw response parser；
6. Python caller migration。

Exit：

- §11 query可由 backend經 Go正確序列化；
- equivalent query共用 cache；
- semantic-different query不碰撞；
- raw R18 profile仍保留。

### Slice 1E：NRF extended matching

Repository：`nrf/`

順序：

1. typed criteria parser；
2. base Mongo filter boundary；
3. Nwdaf events matcher；
4. same-entry ML analytics matcher；
5. ADRF storage matchers；
6. lossless SearchResult與 feature bits；
7. malformed query matrix。

Exit：

- A／B／C discovery matrix全部通過；
- same-entry negative tests通過；
- `nrfSupportedFeatures`正確。

### Slice 1F：Three-role integration

Repository：`nwdaf-resources/`

順序：

1. component pins與 preflight；
2. isolated configs；
3. process runner；
4. profile assertions；
5. discovery assertions；
6. cache／deregistration assertions；
7. README。

Exit：

- 一個命令可重現 Phase 1黑箱驗證；
- runner不修改 sibling repositories；
- logs與 artifacts在 temp root。

### Slice 1G：Documentation and review

Repositories：

- `NWDAF/`
- `PyAnLF/`
- `PyMTLF/`
- `nrf/`
- `nwdaf-resources/`
- `nwdaf-docs/`

更新：

- config reference；
- internal NRF auxiliary API；
- role/profile mapping；
- NRF R18 support boundary；
- deployment README；
- 主計畫與本文件 implementation result；
- canonical design若實作選擇改變既有架構敘述。

最後進行一次完整 current-slice review，不把 Phase 2／3 deferred work
誤列為 Phase 1 defect。

---

## 14. Repository file map

### 14.1 NWDAF

預期主要變更：

- `pkg/factory/config.go`
- `pkg/factory/config_test.go`
- `config/nwdafcfg.yaml`
- `config/nwdafcfg-a.yaml`
- `config/nwdafcfg-b.yaml`
- `config/nwdafcfg-c.yaml`
- `pkg/service/init.go`
- `pkg/service/init_test.go`
- `internal/context/context.go`
- `internal/context/nrf_profile_test.go`
- `internal/compat/nrf/`
- `internal/sbi/consumer/nrf_service.go`
- `internal/sbi/consumer/nrf_service_test.go`
- `internal/anlf/api_nf_discovery.go`
- `internal/mtlf/api_nf_discovery.go`
- shared discovery validation helper所在 package

不新增：

- `testdata/` JSON directory；
- generated OpenAPI fork；
- placeholder Training handler。

### 14.2 NRF

預期主要變更：

- `internal/compat/nrf/`
- `internal/sbi/api_nfmanagement.go`
- `internal/sbi/processor/nf_management.go`
- `internal/sbi/processor/nf_profile_validation.go`
- `internal/sbi/processor/nf_discovery.go`
- focused registration／discovery tests

若大型 `nf_discovery.go`需要拆分，依 concern拆成：

- query parsing；
- base filter；
- extended matcher；
- response construction。

不建立名為 `coordinator`的空泛 package，也不把 R18欄位塞進 arbitrary
generic compatibility map。

### 14.3 PyAnLF

預期主要變更：

- `config/config.yaml`
- `src/py_anlf/core/discovery.py`或拆出的 model provider resolver
- config validation／startup wiring
- focused discovery/config tests
- `docs/api.md`／config docs

### 14.4 PyMTLF

預期主要變更：

- `config/config.yaml`
- `src/py_mtlf/config.py`
- application lifespan／server wiring
- mode-specific lifecycle tests
- README／config docs

### 14.5 nwdaf-resources

預期新增 `deployments/distributed_fl/`，不把跨 process runner放回
`NWDAF/test/`或 backend repository root。

---

## 15. Test plan

### 15.1 NWDAF focused tests

- explicit `serviceNameList`／`nwdafInfo` config validation；
- A／B／C deployment config load tests；
- service list、mounted routes與 advertised `NFService`一致；
- legacy config characterization；
- fixed／generated NF UUID；
- A／B／C profile JSON；
- no duplicate compatibility fields；
- lossless registration request body；
- `201 Location`與 `200` replacement；
- `4xx` terminal、`5xx` retry、redirect、cancellation；
- query serialization round-trip；
- complete cache key；
- hit、expiry、zero-validity、singleflight、bounded eviction；
- raw R18 SearchResult preservation；
- shared AnLF／MTLF auxiliary validation；
- internal response status mapping。

### 15.2 NRF focused tests

- R17 profile regression；
- R18 NWDAF create／replace／GET／PATCH round-trip；
- R18 ADRF profile round-trip；
- invalid profile extension；
- service status filter；
- exact instance filter；
- `nwdaf-event-list` positive／negative；
- outer-OR and same-entry-AND ML analytics matching；
- TAI／interoperability／FL capability／NF type matching；
- combined FL capability；
- ADRF data／model storage indicators；
- malformed JSON query；
- empty result `200`；
- `nrfSupportedFeatures`；
- Mongo/system error `500`。

### 15.3 PyAnLF focused tests

- configured／NRF provider modes；
- config validation；
- query payload；
- candidate requires registered provision service；
- deterministic selection；
- validity cache／expiry；
- empty result；
- malformed result；
- retry state；
- sync unavailable；
- existing SMF／ADRF discovery regression。

### 15.4 PyMTLF focused tests

- valid／invalid execution modes；
- `local` current coordinator regression；
- `fl_server` local trainer suppression；
- `fl_client` server coordinator suppression；
- workspace／URL／TTL／timeout validation；
- health／sync startup and shutdown regression。

### 15.5 Integration tests

Black-box assertions：

1. three registrations成功；
2. profiles包含正確 service與 R18 fields；
3. C不在 analytics query；
4. A／B不在 model-provider query；
5. C不在 FL Client query；
6. TAI-A query只回 A；
7. TAI-B query只回 B；
8. combined TAI query回 A、B；
9. exact A query只回 A；
10. same-entry negative query回 empty；
11. repeated query有 cache evidence；
12. expiry後重新進 NRF；
13. graceful shutdown後 profile移除。

### 15.6 Required repository verification

實作時依各 repository policy執行：

- formatting；
- focused tests；
- race tests for changed concurrent Go package；
- repository lint；
- repository full tests；
- build；
- Python lint／type checks／full tests；
- resource preflight與 integration runner。

所有 script、test runner、build與 local service command依 workspace policy
使用 elevated execution。

---

## 16. Acceptance criteria

Phase 1只有在下列條件同時成立才完成：

1. explicit A／B／C configs存在且可通過 validation；
2. 三者使用不同且可重啟復用的 UUIDv4；
3. A／B profiles宣告 Events、Monitor、`FL_CLIENT`與各自 TAI；
4. C profile宣告 Provision、Monitor、`FL_SERVER`，且沒有
   `nwdafEvents`；
5. Phase 1沒有廣告尚未存在的 Training service；
6. NWDAF registration body使用 lossless Release 18 profile；
7. NRF Mongo round-trip不丟 `flCapabilityType`、
   `mlModelInterInfo`、TAI或 data-source fields；
8. registration create／replace status與 `Location`符合 OpenAPI；
9. discovery query使用正確 form／JSON encoding；
10. NRF same-entry matching符合 TS 29.510；
11. `nwdaf-event-list`與 `ml-analytics-info-list`語意分離；
12. exact instance、service status與 capability filters共同生效；
13. Query-eNA-PH2／PH3 feature bits回傳；
14. Go cache key涵蓋完整 query且無已知碰撞；
15. PyAnLF可選出 C provider但尚不建立 remote resource；
16. PyMTLF三種 mode lifecycle正確；
17. single-NWDAF local regression仍通過；
18. distributed deployment runner可重現 profile／discovery matrix；
19. 每個 repository required verification都有記錄；
20. 文件沒有把 Phase 2／3／4功能誤寫成已完成。

---

## 17. Deferred work and handoff

Phase 2接收：

- PyAnLF selected model provider；
- exact peer discovery；
- A／B／C stable NF identity；
- lossless remote profiles；
- generic discovery cache。

Phase 3接收：

- A／B `FL_CLIENT` capability；
- PyMTLF `fl_client` mode；
- Go role-aware profile builder。

Phase 3建立完整 Training public route後，才：

1. 擴充 explicit profile config，啟用 A／B Training service provider；
2. 加入 `nnwdaf-mlmodeltraining` NF service；
3. FL Client discovery同時要求 service name與 FL capability。

Phase 5接收：

- ADRF model-storage discovery query；
- Query-eNA-PH3 support。

Phase 6接收：

- UDM／SMF generic discovery query；
- internal group identity與 later serving-SMF filters。

---

## 18. Decisions and blockers

已確定：

- core repositories共用 `feat/r18-federated-learning`；
- NRF使用 `feat/r18-nwdaf-discovery`；
- docs保留 `main`；
- compatibility-first，不建立 OpenAPI fork；
- standard communication留在 Go；
- Python backend不向 NRF註冊；
- A／B／C固定角色；
- deterministic first candidate，暫不實作 priority／capacity；
- discovery cache留在 Go且不進 sync；
- Training service不提前廣告；
- 普通 HTTP，安全機制 deferred。

目前沒有需要使用者先決策的 blocking item。實作若發現 local Release 18
schema與本文 field name／conditional rule不一致，必須先以 OpenAPI／TS
修正文件與 contract，不以本文覆蓋規格。
