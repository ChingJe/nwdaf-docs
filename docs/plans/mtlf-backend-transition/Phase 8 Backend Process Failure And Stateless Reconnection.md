# Phase 8：Backend Process Failure And Stateless Reconnection

Date: 2026-08-10

Status: Planned

Related records:

- `MTLF Backend Transition Plan.md`
- `Phase 2 Backend Connectivity And Standard Contract Foundation.md`
- `Phase 3 Analytics Subscription Routing.md`
- `Phase 4 ML Model Monitoring And Accuracy Policy.md`
- `Phase 6 Local Training And Model Update Reprovision.md`
- `../../design/Current AnLF MTLF Feature Architecture.md`

---

## 1. 目的

現有Go NWDAF、PyAnLF與PyMTLF使用full-state sync處理backend晚啟動、restart及route recovery。實際整合顯示，
這讓三個process同時維護可恢復state，並在health probe、sync、CRUD、callback及process replacement之間產生大量
時序競態。

本階段改採較適合目前實驗環境的失效模型：

1. backend process失效後，不恢復該process建立的舊runtime resource。
2. Go清除舊generation state，回到等待backend連線的狀態。
3. 新process ready後只接受新resource，不接收舊snapshot。
4. 能由標準consumer操作取消的relationship，以standard DELETE／Deregister做best-effort cleanup。
5. 無標準producer termination procedure的relationship，清除local state並由consumer timeout、lease或後續DELETE
   收斂。
6. AnLF失效會讓舊analytics停止；MTLF失效不會中止AnLF使用已載入模型繼續推論。

這是backend process lifecycle的project architecture，不是3GPP定義的獨立NF failover procedure。所有對外
payload、feature negotiation、HTTP method與status仍依Release 18 API；內部process失效如何映射到這些標準
operation，由本文件明確固定。

---

## 2. 已確認決策

1. 移除Go、PyAnLF與PyMTLF的`POST /internal/v1/sync`、full snapshot、restart replay及reconciliation。
2. 移除`SYNCING` availability state。
3. 移除PyAnLF／PyMTLF的`GET /health/live`；Go、deployment runner及tests只使用`GET /health/ready`。
4. 每次Python process啟動仍產生新的UUID `processInstanceId`，並在ready 200或503 response中提供。
5. Go不要求兩個backend同時存在；AnLF-only、MTLF-only及兩者啟用皆合法。
6. backend失效後不永久latch failure。cleanup完成後回`WAITING`，新process可以成為新的`USABLE` generation。
7. 舊resource不自動恢復；若後續仍有需求，由consumer重新POST建立新resource。
8. Go restart不做durable recovery；所有process-local state視為遺失，實驗重新建立。
9. AnLF process loss對符合feature negotiation的analytics subscription各嘗試一次
   `failNotifyCode=UNAVAILABLE_DATA` notification；不使用`termCause`，不週期重送。
10. MTLF process loss不終止analytics，AnLF保留已載入模型；provision、monitor、accuracy、retraining及FL
    runtime則失效並清理。
11. Model Provision使用finite `expiryTime`及PUT refresh辨識provider resource是否仍存在。
12. Periodic Model Monitor使用`repPeriod` watchdog；預設容許兩個missed reports再加grace period。
13. process loss cleanup是best effort且不做durable retry queue；normal consumer DELETE及正常runtime cleanup仍保留
    各feature既有、已驗證的retry語意。
14. NRF profile依configured capability維持，不因backend短暫不ready反覆更新；需要該backend的新operation回503。

---

## 3. 規格依據與限制

本節使用3GPP TS 29.520 V18.14.0（Release 18）文字。對應OpenAPI attachment檔案的`externalDocs`標示
TS 29.520 V18.13.0；本階段使用的API document versions分別為Events Subscription 1.3.3、ML Model Provision
1.1.4及ML Model Monitor 1.0.2。引用時不把TS文字版本、OpenAPI document version與free5GC generated model
混為同一來源。

### 3.1 Events failure notification

Release 18 TS 29.520：

- clause 4.2.2.4.2定義`Nnwdaf_EventsSubscription_Notify`，consumer成功接收notification回204。
- clause 5.1.6.2.4定義`NnwdafEventsSubscriptionNotification`；`subscriptionId`必填，
  `eventNotifications`與analytics transfer alternative二擇一。
- clause 5.1.6.2.5定義`EventNotification.failNotifyCode`。
- clause 5.1.8 feature 11 `EneNA`及feature 53 `StatisticsFailure`控制statistics failure report；
  `UNAVAILABLE_DATA`作為`failNotifyCode`只在`StatisticsFailure`支援時適用。
- clause 5.1.8 feature 26 `TermRequest`控制`termCause`；本階段不支援也不advertise此feature。

對應OpenAPI是`TS29520_Nnwdaf_EventsSubscription.yaml`：

- `NnwdafEventsSubscriptionNotification.eventNotifications[]`
- `EventNotification.event`
- `EventNotification.failNotifyCode`
- `NwdafFailureCode.UNAVAILABLE_DATA`

3GPP沒有定義「internal Python process掛掉」這種deployment事件。本階段把AnLF process loss視為該
subscription後續所需資料／分析runtime已不可用，使用上述standard-shaped failure notification。這是有規格
schema支撐的project mapping，不宣稱TS明文規定backend crash必須採此處理。

只有subscription已協商`EneNA`與`StatisticsFailure`時才送`UNAVAILABLE_DATA`。未協商者不得夾帶該欄位；Go
仍清理舊resource並依賴consumer timeout及DELETE。

### 3.2 Model Provision

Release 18 TS 29.520 clauses 4.5.2.2.2、4.5.2.2.3與4.5.2.3.2，以及
`TS29520_Nnwdaf_MLModelProvision.yaml`定義：

- POST `/subscriptions`成功為201及Location。
- PUT `/subscriptions/{subscriptionId}`成功為200或204。
- DELETE成功為204；不存在為404；service unavailable可回503。
- `NwdafMLModelProvSubsc.mLEventSubscs[].expiryTime`可表示subscription event到期時間。
- Model Provision notification沒有producer termination/failure callback欄位。

因此provider不能用一筆標準notification主動宣告「我這個process已失效」。AnLF作為consumer使用finite expiry及
到期前PUT refresh，才有標準resource operation可判斷舊subscription是否仍存在。

### 3.3 Model Monitor

Release 18 TS 29.520 clauses 4.7.2.2.2、4.7.2.3.2、4.7.2.4.2、4.7.2.5.2及4.7.2.6.2，以及
`TS29520_Nnwdaf_MLModelMonitor.yaml`定義：

- registration：POST `/registrations`為201；DELETE `/registrations/{registrationId}`為204。
- subscription：POST `/subscriptions`為201；PUT為200或204；DELETE為204。
- `MLModelMonitorSub.eventReportReq`使用standard `ReportingInformation`。
- `ReportingInformation.repPeriod`定義periodic report interval。
- `MLModelMonitorNotify`沒有termination欄位。

因此periodic monitor的consumer可以用已協商`repPeriod`建立local watchdog，但event-triggered subscription不能單靠
沉默判斷失效。本階段production WAPE monitor本來就是periodic，因此只對periodic path實作timeout。

### 3.4 HTTP status matrix

| 情境 | External／standard-shaped result |
|---|---|
| configured backend尚未USABLE，建立或更新需要該backend的resource | 該API定義的503 `ProblemDetails` |
| active resource正常DELETE | backend／peer成功時204 |
| 已知resource已因backend loss清除，consumer延遲DELETE | 204；backend unavailable或404不推翻cleanup成功 |
| 從未存在於active route或deletion record的ID | 404 |
| failed resource的GET／PUT | 404 |
| Events failure callback被consumer接受 | 204 |
| Events failure callback失敗 | 記錄一次結果，不轉為週期重送或恢復舊resource |
| cleanup DELETE對peer得到404 | 視為resource已不存在，terminal cleanup |

---

## 4. Current State 與問題來源

### 4.1 Go NWDAF

目前`internal/backend`使用`UNKNOWN/POLLING/SYNCING/UNAVAILABLE/USABLE`，AnLF及MTLF probe會先呼叫
`/health/ready`，再POST `/internal/v1/sync`後才標記USABLE。Go還保存Events、SMF、Model Provision、Monitor、
Training及ADRF retrieval等route，部分route帶`processGeneration`。

主要缺陷是：health worker與業務CRUD可以同時改變generation；sync又會把Go snapshot送回Python並觸發
reconciliation，導致restart途中可能重建舊resource、覆蓋新state或和in-flight request互相競爭。

### 4.2 Python backends

兩個backend都有`/health/live`、`/health/ready`及`/internal/v1/sync`。大量component從`SyncProjection`取得：

- containing NWDAF `nfInstanceId`
- Go internal callback base URI及NWDAF apiRoot
- resource snapshot
- training datasource或其他cross-process runtime observation

因此不能只刪route；必須先把真正的immutable context與incremental domain event移出sync projection。

### 4.3 Support tooling

`nwdaf-resources`中仍有runner先等`/health/live`再等`/health/ready`，以及assert sync access log／restart replay的
process tests。這些測試目前驗證的是將被移除的behavior，Phase 8要改成驗證process-generation reset與
stateless reconnection。

---

## 5. Target Availability Lifecycle

### 5.1 Backend-independent state

每個configured backend獨立維護下列state；一邊變化不應把另一邊一起標成unavailable：

```text
DISABLED                         configuration does not enable this backend

WAITING -> USABLE(G1) -> NOT_READY -> USABLE(G1)
                    \-> RESETTING -> WAITING -> USABLE(G2)
```

- `WAITING`：configured，但尚未有ready process。
- `USABLE(Gn)`：ready回200，且已保存該`processInstanceId` generation。
- `NOT_READY`：process可達但ready回503，或只有第一個尚未確認的transport failure；new operation回503，舊route
  暫時保留。
- `RESETTING`：已確認process replacement/loss，正在drain及cleanup；new operation回503。
- `DISABLED`：不polling、不advertise依賴該backend的capability。

不再有`SYNCING`。ready成功本身就足以讓一個已完成local startup的backend進入USABLE。

### 5.2 Ready response

兩個backend都保留同一最小shape：

```json
{
  "status": "ready",
  "processInstanceId": "3a581cf5-4e7f-4b62-82da-becdb55b0f0a"
}
```

not-ready response仍應包含本次process ID：

```json
{
  "status": "not_ready",
  "processInstanceId": "3a581cf5-4e7f-4b62-82da-becdb55b0f0a"
}
```

PyMTLF可繼續在ready representation附帶`runtimeMode`或artifact readiness detail；Go只依status、HTTP status及
`processInstanceId`做lifecycle決策。

### 5.3 Probe判定

1. 200且ID與目前相同：保持／回到同generation USABLE，transport failure counter歸零。
2. 200且ID不同：立即確認replacement，不需再等第二次probe。
3. 503且ID相同：NOT_READY，保留舊route，不reset；之後相同ID回200可恢復。
4. 503且ID不同：已確認replacement，進入RESETTING；新process即使尚未ready也不能取得舊state。
5. 第一次timeout／connection refused：NOT_READY並block new operation，保留舊route，立即縮短下次probe間隔。
6. 連續第二次readiness transport failure：確認loss並進入RESETTING。
7. 一般backend CRUD 4xx不改health；5xx或transport error只記為suspect並立即喚醒ready probe，不單憑一次業務
   request直接清空generation。
8. 既有successful probe interval保留30秒；failure backoff保留1、2、5、10、30秒上限及jitter。

### 5.4 Generation fence與in-flight request

process replacement可能發生在兩次probe之間。Phase 8不在每次CRUD response後再多打一個health request，而採
generation fence：

1. 每個送往backend的operation進入時取得目前generation及in-flight token。
2. RESETTING先關閉new admission，再等待已取得token的operation退出。
3. cleanup只處理該舊generation的route。
4. 若G2在下一次health probe前成功接到一筆operation，probe看到新ID後仍會進入RESETTING；該operation所建立的
   G2 resource也會由best-effort DELETE清除，不被誤認為可保留的舊resource。
5. cleanup完成後才允許G2成為USABLE並接受新的business operation。

這個設計不承諾跨process exactly-once；它承諾reset完成後沒有任何舊generation resource被自動恢復。

---

## 6. Route Ledger 與 Deletion Record

Go保留的是cleanup所需ledger，不是可重播snapshot。

### 6.1 Active route最小資訊

每筆route依feature只保存：

- external resource ID與resource kind
- owner backend及`processInstanceId`
- relationship direction：backend是standard consumer或provider
- peer `apiRoot`、Location／resource ID或callback URI
- consumer correlation及failure callback所需的原始event identity
- cleanup是否已開始／完成

Go在自己轉送標準POST／PUT／DELETE時直接更新ledger。SMF resource不再需要PyAnLF另送full association snapshot；
Go由既有SMF proxy request、201 Location與delete result取得cleanup資料。

Events route還必須保存consumer request／provider response協商後的`EneNA`與`StatisticsFailure`結果；Go不得只看
自己支援的feature就送failure callback，也不得在reset時重新猜測negotiation。

### 6.2 Deletion record

backend loss後只保留：

- resource ID與kind
- old owner generation
- cleanup attempted/result
- 必要的consumer correlation

不保存完整subscription body、model、dataset、policy state或Python runtime。record只存在memory，直到consumer
DELETE或Go restart；第一版沒有TTL、database或background GC。

### 6.3 DELETE rule

```text
consumer DELETE
  -> active route exists: normal delete path
  -> deletion record exists: optional one-shot backend delete
       -> 204 / 404 / backend unavailable: remove record, return 204
  -> neither exists: return 404
```

這使consumer可以在收到failure或自己的timeout後正常完成取消訂閱，不會因新backend不認識舊ID而得到錯誤。

---

## 7. AnLF Process Loss

### 7.1 影響範圍

AnLF process memory擁有analytics subscription、inference window、SMF collection refcount、loaded runtime binding、
monitor provider resource及raw notification buffers。process失效後這些runtime全部視為無效，不嘗試恢復。

### 7.2 Events failure notification

Go對每筆已協商`EneNA`與`StatisticsFailure`的active Events subscription，向原`notificationURI`嘗試一次：

```http
POST {notificationURI}
Content-Type: application/json

{
  "eventNotifications": [
    {
      "event": "UE_COMMUNICATION",
      "failNotifyCode": "UNAVAILABLE_DATA"
    }
  ],
  "subscriptionId": "f447ae68-5b71-42f4-8272-57a507a065cb",
  "notifCorrId": "consumer-correlation-1"
}
```

- `event`使用該subscription實際event，不固定成`UE_COMMUNICATION`；範例只呈現目前實驗事件。
- 不填`termCause`或`rvWaitTime`。
- 每個受影響subscription只做一次HTTP delivery attempt；成功204或任何失敗都不建立週期worker。
- delivery失敗只記structured log及metric。
- notification後不再產生該舊subscription的analytics notification。

### 7.3 Relationship cleanup

按dependency執行一次best-effort cleanup：

1. gate AnLF-backed new operations並drain in-flight。
2. 發送Events failure notification。
3. 對AnLF作為consumer建立的Model Provision subscription送standard DELETE。
4. 對AnLF作為consumer建立的Model Monitor registration送standard Deregister。
5. 對AnLF建立的SMF collection resource送standard DELETE。
6. 清除AnLF作為provider承接的Model Monitor subscription route；標準schema沒有provider termination callback。
7. 清除analytics/model runtime routes，轉為deletion records。
8. 將AnLF state改回WAITING。

normal consumer DELETE仍可在上述任何時點進入，但只由generation gate中的單一owner完成；不得double-delete或因
peer 404把external結果改成failure。

---

## 8. MTLF Process Loss

### 8.1 不受影響的analytics

MTLF失效不代表已下載到AnLF的model失效。PyAnLF繼續：

- 使用目前loaded model推論
- 回覆既有analytics subscription
- 蒐集及保存training data

不得因MTLF unavailable對analytics consumer送failure或取消Events subscription。

### 8.2 失效的MTLF runtime

清除：

- MTLF作為provider的Model Provision resources
- AnLF向MTLF建立的monitor registrations
- MTLF作為consumer向AnLF建立的monitor subscriptions
- accuracy policy window、retrain-in-flight與dataset job runtime
- in-flight local training／FL process、round、candidate與temporary workspace
- 未完成的retrieval subscription及callback route

保留：

- completed model catalog
- immutable completed model artifacts
- formal latest-model pointer及已完成publication record
- ADRF／Mongo中的資料與模型store record

### 8.3 Cleanup順序

1. gate MTLF-backed new operations並drain in-flight。
2. 對MTLF作為consumer建立的Model Monitor subscriptions送standard DELETE。
3. 對未完成ADRF retrieval及跨NWDAF Model Training relationship送其standard delete／termination operation，
   若已有可辨識resource。
4. 將MTLF provider-side Model Provision及monitor registration route轉為deletion record。
5. 終止local worker並清除temporary workspace；completed artifact不得被刪除。
6. MTLF state回WAITING。

新PyMTLF ready後不會自動re-provision、重新register monitor或接續舊training round。

---

## 9. Model Provision Lease

### 9.1 Consumer behavior

PyAnLF建立Model Provision subscription時，對每個`mLEventSubscs[]`填finite `expiryTime`。第一版config：

```yaml
model_provision:
  lease_duration_seconds: 3600
  renewal_margin_seconds: 300
```

- default lease為60分鐘，在到期前5分鐘開始PUT refresh。
- 同一provision resource同時只有一個renewal in flight。
- multi-event resource以最早`expiryTime`安排refresh。
- PUT body仍是完整standard `NwdafMLModelProvSubsc` representation，只更新必要的expiry／demand union。

### 9.2 Refresh結果

| 結果 | PyAnLF行為 |
|---|---|
| 200／204 | 保存新expiry，relation維持active |
| 404 | 舊provider resource不存在；清除relation，保留loaded model |
| 503／transport failure | peer unavailable；停止舊relation refresh，保留loaded model |
| other 4xx | 記錄contract rejection並清除舊relation，不自行改body重試 |

relation清除後不立即自動POST同一舊需求。後續新的analytics model demand可以重新走NRF discovery／configured
provider selection並建立新的subscription。

---

## 10. Periodic Model Monitor Watchdog

### 10.1 Timeout計算

PyMTLF作為monitor consumer，使用實際subscription中的`eventReportReq.repPeriod`：

```text
deadline = lastAcceptedNotificationAt
         + missedReportLimit * repPeriod
         + gracePeriod
```

第一版config：

```yaml
model_monitor:
  report_period_seconds: 90
  missed_report_limit: 2
  watchdog_grace_seconds: 30
```

因此default timeout為210秒。建立subscription後尚未收到第一筆report時，以201 response acceptance time作起點。
每筆通過correlation及model/scope validation的`MLModelMonitorNotify`都重設deadline；即使資料不足而沒有
`deviation`，仍代表monitor provider活著，因此可重設watchdog，但不更新accuracy policy。

### 10.2 Timeout cleanup

deadline到期後：

1. 將monitor relationship標成inactive。
2. 對AnLF provider送一次standard DELETE。
3. 204或404都完成cleanup；503／transport failure記錄後清除local relationship。
4. 清除對應registration、subscription correlation及尚未terminal的policy／retrain runtime。
5. 不自動resubscribe。

event-triggered monitor沒有可推導的periodic deadline，不套用此watchdog。

### 10.3 Provider callback failure

PyAnLF仍使用Phase 4既有bounded notification retry。若一筆notification最終無法送達，不把它變成Events failure
notification；若Go已確認local MTLF loss，Go會以MTLF consumer身分對PyAnLF執行standard monitor DELETE。對遠端
consumer，callback持續得到terminal 404時停止該monitor；transport／503則由bounded retry及對端watchdog收斂，
不建立永久重送queue。

---

## 11. 移除 SyncProjection 後的最小交互

### 11.1 Stateless containing-NWDAF context

兩個backend config只需要知道Go internal apiRoot。真正由Go擁有且可能由部署產生的NWDAF identity，不在兩份
Python config重複維護，改由單一stateless read取得：

```http
GET /internal/v1/nwdaf-context

200 OK
{
  "nfInstanceId": "2e76b460-ec3f-4cd0-8f27-c180839ccf01",
  "apiRoot": "http://127.0.0.1:8000",
  "internalApiRoot": "http://127.0.0.1:8090"
}
```

這條route分別掛在既有AnLF與MTLF薄internal server；`internalApiRoot`回傳該backend應使用的Go-side root。response
只提供identity及base URI，不含subscription、route、datasource、capability matrix或backend state。
backend可在需要第一個outbound standard operation時讀取並process-local cache；Go restart後重新讀取即可，不能用它
恢復business resource。

### 11.2 Existing sync payload分類

| Current sync內容 | Phase 8處理 |
|---|---|
| containing NWDAF identity/base URI | 改用stateless context read |
| Events／Model／Training active resource snapshot | 移除，不重播 |
| `trainingDataSource` | 移除cross-backend同步；兩邊各自套ADRF-first／Mongo fallback |
| SMF resource associations | 不再由PyAnLF發full list；Go由標準SMF proxy lifecycle建立cleanup ledger |
| training-data descriptors | 保留為domain-specific incremental PUT／DELETE，Go只轉給當下USABLE的PyMTLF，不作restart replay |
| process generation | 只由`/health/ready.processInstanceId`提供 |

training descriptor path從`/internal/v1/sync/anlf/training-data-descriptors`移到
`/internal/v1/anlf/training-data-descriptors/{descriptorId}`。PUT建立／replace current descriptor，DELETE移除；payload
保留目前已驗證的standard-shaped `smfDataSub`及scope metadata，不趁改名重設業務語意。PyMTLF unavailable時回503，
不在Go建立durable queue；PyAnLF記錄並放棄本次descriptor delivery，不阻擋analytics、collection或storage，且不在
PyMTLF後續restart時重播舊descriptor。

### 11.3 Readiness independence

backend local config、artifact root、database client及worker初始化完成即可ready。`nwdaf-context`暫時不可讀不應讓
ready endpoint block；真正需要Go-owned standard communication的operation才依該次lookup結果回503。這避免
Go probe backend、backend又必須先成功call Go才能ready的循環依賴。

---

## 12. Repository Scope

### 12.1 NWDAF

- 將availability monitor改成ready-only、generation-aware state machine。
- 移除sync client、contract、snapshot builder及`trainingDataSource` relay。
- 加入reset gate、in-flight drain與per-generation cleanup coordinator；不建立新的AnLF／MTLF business
  coordinator package。
- 擴充既有route stores為cleanup ledger及deletion record。
- 在AnLF loss時建立符合negotiation的standard Events failure notification。
- 保留Events `suppFeat` request／response intersection並把已協商`EneNA`／`StatisticsFailure`存入route ledger。
- 調整Events、Model Provision、Model Monitor、Model Training、SMF及ADRF retrieval DELETE semantics。
- 新增薄的stateless containing-NWDAF context handler；位置依既有internal API server/domain ownership決定，不為
  單一feature任意建立新的top-level package。
- 移除`/internal/v1/sync/anlf/*` routes；training descriptor使用domain path，SMF association由proxy lifecycle取得。

### 12.2 PyAnLF

- 移除sync router、models、projection、snapshot reconciliation及tests。
- 移除`/health/live`及相關docs/tests。
- 將discovery、collection、storage、model demand、monitor registration及retrieval改用stateless NWDAF context client。
- Model Provision request加入finite expiry、renewal scheduler及lease failure cleanup。
- AnLF shutdown／loss不做舊analytics restore；正常local shutdown仍釋放worker與file/database resources。
- monitor callback terminal failure停止該舊relationship，但analytics及loaded model保留。
- 保留既有accuracy、collection、storage、model load/cutover演算法；本phase只改lifecycle ownership。

### 12.3 PyMTLF

- 移除sync router、models、projection、snapshot reconciliation及tests。
- 移除`/health/live`及相關docs/tests。
- discovery、ADRF、dataset、publication、monitor、FL server/client改用stateless NWDAF context client。
- 加入periodic monitor watchdog及terminal cleanup。
- process restart只從durable completed model state啟動；不恢復舊subscription、accuracy window、dataset job或FL round。
- 保留既有WAPE、retrain、dataset、training、promotion及model catalog演算法。

### 12.4 nwdaf-resources

- runner只等待`/health/ready`。
- 移除sync access-log assertion及snapshot replay fixture。
- 新增AnLF loss、MTLF loss、process ID replacement、late DELETE與in-flight race的cross-process scenarios。
- 保留現有portable event exposure及distributed FL資料路徑；只替換lifecycle expectation。

### 12.5 nwdaf-docs

- 更新current feature architecture及PyAnLF／PyMTLF API reference。
- historical Phase 2／3文件保留當時實作紀錄，但加上Phase 8 supersession指引，不重寫歷史結果。

---

## 13. Implementation Slices

slice是實作與驗證checkpoint，不要求每個slice各自commit；完成全部scope並review後再依repository分開commit。

### Slice 1：Contract inventory與context extraction

- 列出所有`SyncProjection` consumer及payload field。
- 建立stateless NWDAF context route/client。
- 將immutable context consumer移轉並建立characterization tests。
- 將training descriptor移到domain path；SMF association ledger改由proxy lifecycle維護。

### Slice 2：Go availability與generation reset

- ready-only state machine、兩次transport confirmation及process ID replacement。
- admission gate、in-flight drain、route generation fence。
- cleanup ledger及deletion record。
- AnLF-only、MTLF-only及both-enabled config tests。

### Slice 3：AnLF failure path

- negotiated one-shot Events failure notification。
- Events late DELETE 204及failed GET／PUT 404。
- SMF、Model Provision及Monitor relationship cleanup。
- PyAnLF sync/live removal及provision lease。

### Slice 4：MTLF failure path

- analytics continuity與loaded-model preservation。
- provision/monitor/training/FL cleanup及completed artifact preservation。
- PyMTLF sync/live removal及monitor watchdog。

### Slice 5：Cross-process migration

- support runner與API docs更新。
- process restart、in-flight replacement、callback failure、late DELETE、lease及watchdog E2E。
- 移除所有active runtime對`/health/live`、`/internal/v1/sync`與`SYNCING`的依賴。

---

## 14. Verification Matrix

### 14.1 Go unit/race

1. ready 200 same ID維持USABLE。
2. ready 503 same ID進NOT_READY但不cleanup。
3. single transport failure不cleanup；second consecutive failure進RESETTING。
4. any response with changed ID立即reset。
5. reset blocks new CRUD並等待in-flight token。
6. G2在probe前建立的resource會被reset cleanup，不殘留。
7. AnLF與MTLFstate完全獨立。
8. cleanup concurrent with consumer DELETE只產生一個terminal result。
9. known deletion record回204；unknown ID回404。
10. `go test -race`覆蓋availability、route store及processor lifecycle。

### 14.2 Events

1. negotiated features產生一次合法`UNAVAILABLE_DATA` notification。
2. 未協商StatisticsFailure不送不合法field。
3. 不含`termCause`且不advertiseTermRequest。
4. callback 204、4xx、5xx及transport failure都不建立週期retry worker。
5. loss後不再從舊subscription送analytics。
6. delayed DELETE在backend unavailable、new backend 404兩種情境都回204。

### 14.3 Model Provision／Monitor

1. provision body的每個event帶finite expiry。
2. renewal 200／204延長lease；404／503／transport清relation但保留model。
3. no-deviation periodic report重設watchdog但不更新accuracy policy。
4. `2 * repPeriod + grace`前不timeout，boundary後只cleanup一次。
5. MTLF loss後AnLF analytics持續。
6. AnLF loss後MTLF monitor／policy／retrain state清除。

### 14.4 Python

1. `/health/ready`在200及503都回valid UUID。
2. `/health/live`為404，沒有router、test或API reference宣稱它存在。
3. 沒有full sync route/model/projection。
4. completed models可跨PyMTLF restart載入；active resource與FL workspace不恢復。
5. PyAnLF既有collection、WAPE及atomic model cutover parity tests保持通過。
6. PyMTLF既有policy、dataset、training及promotion parity tests保持通過。

### 14.5 Process-level

1. Go先啟動、backend後啟動，ready後可建立new resource且沒有sync request。
2. PyAnLF kill：consumer只收到一次failure；MTLF不被一起標失效；restart後舊Events不恢復。
3. PyMTLF kill：analytics繼續使用舊model；monitor/provision/training停止；restart後只接受new relationship。
4. 同一backend快速G1→G2 replacement不殘留任何G1 route。
5. AnLF-only及MTLF-only部署不因另一backend缺席持續報錯或錯誤advertise capability。
6. runner不呼叫`/health/live`或`/internal/v1/sync`。

---

## 15. 完成條件

1. 主runtime不存在full-state sync、restart replay或`SYNCING` state。
2. PyAnLF、PyMTLF及support tooling不再使用`/health/live`。
3. backend process ID replacement與transport-confirmed loss都能安全reset並重新等待ready。
4. reset期間new request回503，in-flight request不與route cleanup產生data race。
5. AnLF loss依feature negotiation送一次standard failure notification，且consumer late DELETE可完成。
6. MTLF loss不停止analytics，completed model state不被刪除。
7. Model Provision lease及periodic Monitor watchdog完成並有boundary tests。
8. 所有outbound standard relationship都有明確consumer-owned cleanup；沒有標準termination的provider resource不
   發明custom callback。
9. stateless context與incremental descriptor path取代sync projection，不形成另一個full-state sync。
10. NWDAF build/test/lint/race、PyAnLF/PyMTLF lint/test及cross-process scenarios全部通過。
11. 相關API、architecture及deployment文件反映實際行為。

---

## 16. 明確不在本階段處理

- durable Go state或Go restart recovery
- backend process crash後自動恢復舊subscription
- exactly-once callback／distributed transaction
- message broker或durable cleanup queue
- NRF profile因瞬時readiness動態變更
- TermRequest／`termCause`
- event-triggered monitor silence detection
- Mongo-to-ADRF backfill或cross-source history merge
- production TLS、OAuth delegation或backend獨立NF identity

上述項目不得成為Phase 8 acceptance blocker，也不得以「未來可能需要」為理由保留full sync。
