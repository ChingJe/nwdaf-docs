# Phase 2 Backend Connectivity And Standard Contract Foundation

Date: 2026-07-21

Status: Foundation implemented locally; original MTLF-only storage handshake superseded by the unified backend sync direction

Parent plan:

- `nwdaf-docs/docs/plans/mtlf-backend-transition/MTLF Backend Transition Plan.md`

Previous phase:

- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 1 PyMTLF Foundation And Backend Boundary.md`

Next phase:

- `nwdaf-docs/docs/plans/mtlf-backend-transition/Phase 3 Analytics Subscription Routing.md`

---

## 1. Purpose And Historical Role

Phase 2原始目的，是建立Go NWDAF與AnLF/MTLF backend之間可晚啟動、可重連、可獨立判斷availability
的runtime foundation，並先為MTLF backend加入storage-mode handshake。

本phase的polling、cached availability、health endpoint及shutdown ownership已完成並仍是後續phase基礎。
但team architecture review已改變資料蒐集與storage ownership：

- SMF/UPF notification將直接送到AnLF backend。
- AnLF backend而非Go負責raw Mongo write。
- AnLF backend準備ADRF storage request，由Go執行標準ADRF呼叫。
- MTLF backend依source availability與preference選擇retrieval source。
- AnLF與MTLF backend都需要readiness後sync，不能只有MTLF做storage handshake。

因此本文件現在同時是：

1. 已完成Phase 2 implementation的歷史紀錄。
2. 哪些foundation保留、哪些contract被supersede的replan boundary。

新implementation不得再把原始`POST /internal/v1/data-source-selection`視為長期canonical contract。

---

## 2. Evidence And Alignment

Phase 2 connectivity本身沒有3GPP定義的Python backend precedent；polling及sync是本專案architecture
decision。結構上參考local free5GC：

- `resources/references/free5gc-main/NFs/nrf/pkg/service/init.go`
  - app-owned context、wait group、server startup/shutdown ownership
- `resources/references/free5gc-main/NFs/nrf/pkg/app/app.go`
  - service與internal packages之間的窄app interface
- `resources/references/free5gc-main/NFs/udm/pkg/service/init.go`
  - consumer、processor與server由service組裝
- `resources/references/free5gc-main/NFs/udm/internal/sbi/consumer/consumer.go`
  - outbound service client集中在consumer boundary

這些source只支持lifecycle與package shape；沒有證據支持MTLF-only handshake或Go-owned Mongo writer。

標準feature的payload與HTTP behavior必須由對應OpenAPI/TS決定。Phase 2的private health/sync不得成為
自訂business error或standard feature DTO的來源。

---

## 3. Preserved Foundation

下列Phase 2成果仍是target architecture：

1. Go可在兩個backend都未啟動時完成自己的startup。
2. AnLF與MTLF backend使用獨立availability worker。
3. worker startup立即probe，不先sleep。
4. probe failure使用bounded backoff與jitter。
5. success後持續periodic probe。
6. request path讀cached availability，不同步呼叫health。
7. live operation的network error、timeout及5xx可將backend標為unavailable並喚醒polling。
8. 明確4xx domain response不把整個backend誤判為down。
9. worker、timer及network request受`NwdafApp` context cancellation與wait group管理。
10. backend disabled、configured-but-unavailable與usable是不同狀態。
11. NRF advertisement依configuration及implemented capability，不隨瞬時health抖動。
12. AnLF與MTLF backend都提供：

```text
GET /health/live
GET /health/ready
```

`/health/live`只表示process/event loop可回應；`/health/ready`表示本地初始化完成。沒有active model或尚未
取得資料不代表process not ready。

target correction要求兩個backend每次process啟動產生新的`processInstanceId` UUID並放入
readiness response。這不是schema version或持久identity，只用於讓Go辨識兩次probe之間發生的
快速backend restart。

---

## 4. Implemented State

### 4.1 NWDAF

已實作：

- backend-neutral availability monitor
- independent worker、cached snapshot、bounded backoff、jitter及immediate retry wake-up
- app-owned cancellation與shutdown wait
- AnLF/MTLF bounded readiness client
- current AnLF operation使用cached availability gate
- backend configured但unusable時的generic 503 `ProblemDetails`
- config-based Events Subscription advertisement
- 原始MTLF source inventory與Mongo bounded ping
- 原始MTLF `data-source-selection` handshake client

implementation commit：

- `e85b7ef feat(backend): add availability coordination`

### 4.2 PyAnLF

已實作：

- `/health/live`
- `/health/ready`
- readiness綁定manager initialization、request acceptance與shutdown lifecycle

implementation commit：

- `05a85ec feat: add backend health lifecycle`

### 4.3 PyMTLF

已實作：

- `/health/live`
- `/health/ready`
- `POST /internal/v1/data-source-selection`
- `adrf`、`mongodb`、`dual` strict selection與409 mismatch
- 移除舊SQLite generation journal、reconciliation、專用models/config/tests

implementation commit：

- `a00e8f0 feat: add data source selection handshake`

本節只記錄已存在的code，不代表每一項仍屬長期target。

---

## 5. Superseded Phase 2 Design

下列原始決策已被2026-07-21 architecture review取代：

### 5.1 MTLF-only Usability

舊設計：

```text
AnLF readiness success -> USABLE
MTLF readiness success -> HANDSHAKE -> USABLE
```

新方向：

```text
AnLF: POLLING -> READY -> SYNCING -> USABLE
MTLF: POLLING -> READY -> SYNCING -> USABLE
```

兩者都需要在reconnect後取得足以恢復自己feature state的snapshot。exact sync payload由實際feature
consumer決定，不建立generic state bus。

### 5.2 Strict Storage Requirement

舊設計把`adrf`、`mongodb`、`dual`當成MTLF hard requirement；缺少任何required source即409且不fallback。

新方向：

- MTLF backend表達source preference及effective selection。
- preferred source不可用時可使用另一個available source。
- MTLF尚未完成selection時，AnLF backend向所有可用storage寫入。
- storage availability是Go、AnLF與MTLF sync的一部分，不是單一MTLF handshake response。

因此`DATA_SOURCE_REQUIREMENT_UNSATISFIED`及strict 409不再是長期business contract。

### 5.3 Go-owned Storage Inventory And Writer

舊設計假設Go：

- ping Mongo
- 決定available source inventory
- 根據MTLF mode切換future UPF writer
- 寫raw notification到Mongo

新方向：

- AnLF backend直接收到SMF/UPF notification。
- AnLF backend直接寫Mongo並最能判斷Mongo write availability。
- AnLF backend透過Go執行標準ADRF storage request。
- MTLF backend直接讀Mongo或ADRF。
- Go不再擁有training-data writer或Mongo query API。

### 5.4 Security Acceptance

原始Phase 2把production OAuth/TLS列為deployment pending。現在這些只保留為future note，不屬於目前
phase implementation、acceptance或blocker；現有target使用普通HTTP。

---

## 6. Revised Connection State

backend-neutral state概念調整為：

```text
DISABLED (config only)

UNKNOWN
   |
   v
POLLING -- probe failure --> UNAVAILABLE
   |
   v
READY
   |
   v
SYNCING -- sync failure --> UNAVAILABLE
   |
   v
USABLE -- transport/probe failure --> UNAVAILABLE
```

要求：

1. 每個backend獨立維護state與last success/failure。
2. READY只表示`/health/ready`成功；尚不能承接依賴snapshot的feature。
3. sync成功後才publish USABLE。
4. backend reconnect或process restart後重新sync。
5. request path只讀immutable cached snapshot。
6. state transition留下bounded structured log，不暴露backend credential或private response body。
7. sync failure回到UNAVAILABLE並依原有backoff重試。
8. shutdown不得留下detached goroutine。
9. USABLE後仍持續periodic probe；`processInstanceId`變更即使中間沒有probe failure也必須
   重新sync。

原始monitor implementation應被延伸，而不是建立第二套`sync monitor`。

---

## 7. Revised Sync Responsibilities

sync是ordinary HTTP private coordination，不是3GPP service，也不處理security key。

共同最低語意：

- Go告知backend目前NWDAF runtime可用的standard peer capabilities。
- Go可提供process-local active resource/routing snapshot。
- backend回報自己的source availability或selected preference。
- Go只快取後續routing與shared sync distribution有production consumer的結果，不保存raw data或
  backend business state。
- Go是central coordinator；AnLF與MTLF backend不建立直接sync channel。

AnLF-specific：

- restore accepted Events Subscription snapshot
- restore需要重新建立的SMF/UPF collection intent
- report Mongo write availability及ADRF storage capability
- receive MTLF backend經Go分發的effective source
- restore existing SMF peer Location與pending/orphan cleanup routing

MTLF-specific：

- receive available ADRF/Mongo source view
- report preferred/effective retrieval source
- restore尚未完成的standard retrieval/model feature state（只在對應phase有實際consumer時加入）

Go可將AnLF backend回報的Mongo availability整合進central snapshot並sync給MTLF backend；MTLF
backend回報的effective source也由Go同步給AnLF backend。sync是per-backend typed snapshot，不是將
相同payload無條件廣播。

initial/reconnect sync在成功前會擋住USABLE。USABLE後仍依periodic health cycle進行snapshot
refresh，並在Go-owned state或backend typed observation真正改變時喚醒相關backend；periodic refresh
期間不將正常backend反覆暴露為SYNCING，但refresh失敗會轉UNAVAILABLE並回到backoff retry。
轉發前以typed semantic equality比較新舊值，不加wire revision/CAS，也不形成AnLF↔MTLF無限回圈。

第一版sync不得預先包含：

- credentials或key
- raw dataset
- training job state
- generation CAS
- provider attempt state
- arbitrary key/value extension map
- schema migration/version framework

Go的active resource/routing snapshot第一版只存在memory。Go process重啟後不做訂閱持久化、
backend-to-Go authority recovery或自動重建；實驗視為重跑並由consumer重新訂閱。

exact route、method及Phase 3有production consumer的payload由`Phase 3 Analytics Subscription Routing.md`
固定；Phase 5再加入MTLF retrieval所需的最小source fields。若一個欄位沒有production
consumer，就不加入。

---

## 8. Availability And Standard Error Boundary

### 8.1 Backend Unavailable

- backend disabled：不advertise依賴它且已實作的service/capability。
- backend configured但unavailable：NWDAF仍啟動並advertise configured implemented capability。
- operation確實需要該backend且backend unusable時，Go使用該standard API允許的503
  `ProblemDetails`。
- 不需要該backend的operation繼續服務。

Phase 2的generic 503只適用「backend本身不可用」。資料來源暫時不可用是feature/domain狀態，不能一律
轉成相同503；Phase 3/5必須依TS/OpenAPI另行決定。

### 8.2 Advertisement

NRF profile不隨每次readiness transition更新。只有：

- config enable/disable
- feature route是否已實作
- advertised standard capability

才決定profile內容。

---

## 9. Verification Record

原始Phase 2 implementation完成時已記錄：

- `PyAnLF/`: `145 passed, 1 skipped`; Ruff通過
- `PyMTLF/`: `42 passed`; Ruff通過
- `NWDAF/`: focused tests、race tests、`make test`、`make build`、`make lint`通過
- real PyAnLF/PyMTLF process-level health/handshake smoke tests通過
- backend停止與重啟時Go availability可失效並恢復
- `git diff --check`通過
- 沒有新增`testdata` JSON contract fixture

這些結果證明polling foundation，不證明新sync、direct callback、Mongo ownership或ADRF data path。

修正後需新增：

1. AnLF與MTLF都經READY→SYNCING→USABLE。
2. backend restart觸發resync。
3. backend在兩次probe間快速restart時，`processInstanceId`變更仍觸發resync。
4. old strict selection endpoint沒有production caller後被移除。
5. source fallback semantics不再回舊409。
6. direct callback期間AnLF unavailable的loss/recovery限制有integration test與文件。
7. plain HTTP deployment smoke test。

---

## 10. Acceptance And Handoff

Phase 2 foundation視為「已實作但部分contract待後續取代」，不是完整transition completion。

可直接重用的acceptance：

- backend缺席時Go可啟動與關閉
- independent polling與cached state
- health semantics
- wake-up/backoff/shutdown
- config-based advertisement
- generic backend-unavailable gate

必須由Phase 3完成：

- unified sync lifecycle
- AnLF snapshot/reconnect
- 移除Go-owned collection assumptions
- 移除MTLF-only strict handshake作為production contract

必須由Phase 5完成：

- MTLF source preference/effective selection
- fallback semantics
- direct ADRF/Mongo retrieval

review時不得以原始Phase 2 acceptance「handshake已通過」認定新的storage/data architecture已完成。
