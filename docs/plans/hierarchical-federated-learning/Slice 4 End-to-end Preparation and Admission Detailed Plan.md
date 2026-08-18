# Slice 4：End-to-end Preparation 與 Admission 詳細計畫

日期：2026-08-18

狀態：Ready for implementation；local evidence 與 team review 已完成，production
implementation 尚未開始

上層計畫：

- [Hierarchical NWDAF Federated Learning Implementation Plan](./Hierarchical%20NWDAF%20Federated%20Learning%20Implementation%20Plan.md)

前置 contracts：

- [Slice 0 Baseline Audit and Contract Freeze Detailed Plan](./Slice%200%20Baseline%20Audit%20and%20Contract%20Freeze%20Detailed%20Plan.md)
- [Slice 1 Hierarchy Bundle Contract and Artifact Primitives Detailed Plan](./Slice%201%20Hierarchy%20Bundle%20Contract%20and%20Artifact%20Primitives%20Detailed%20Plan.md)
- [Slice 2 Capability and Process-scoped Role Foundation Detailed Plan](./Slice%202%20Capability%20and%20Process-scoped%20Role%20Foundation%20Detailed%20Plan.md)
- [Slice 3 Root Initiation, Static Topology and Assignment Detailed Plan](./Slice%203%20Root%20Initiation%2C%20Static%20Topology%20and%20Assignment%20Detailed%20Plan.md)

相關文件：

- [Hierarchical NWDAF Federated Learning Proposal Draft](../../proposals/nwdaf/hierarchical-federated-learning/proposal_draft.md)
- [NWDAF Development Policy](../../development_policy.md)

---

## 1. 目的與完成邊界

Slice 4 要把 Slice 3 停在 PREPARATION_WAITING 的 Root–Branch resources，串成可執行的
Root–Branch–Leaf asynchronous preparation walking skeleton。Branch PyMTLF 必須下載並
驗證 Root assignment、依 assignment 重新解析 Leaves、發布 Leaf-specific bundles、使用
同一 instance 的 FL Server engine 建立 lower-tier preparation resources，收集所有 Leaf
outcomes，最後以 preparation-result bundle 回報 Root。

Root 收到所有 Branch outcomes 或 preparation deadline 後，必須驗證 result bundles，套用
第一版 complete_required policy，對整棵 topology 做唯一一次 admission decision。

成功流程固定為：

    Root PREPARATION_WAITING
      -> Branch downloads and validates BRANCH_ASSIGNMENT
      -> Branch binds upper Client resource as BRANCH
      -> Branch republishes one LEAF_ASSIGNMENT per assigned Leaf
      -> Branch creates lower-tier standard preparation resources
      -> Leaves download assignments and prepare local data
      -> Leaves notify Branch with assignment model URLs
      -> Branch classifies prepared, failed and timed-out Leaves
      -> Branch publishes PREPARATION_RESULT
      -> Branch notifies Root with result bundle URL
      -> Root validates every Branch result
      -> complete_required admission succeeds
      -> Root enters ADMITTED

本 Slice 的成功終點是 ADMITTED。上下層 standard subscriptions、process correlations、
validated assignments、participant snapshot 與必要 artifacts 都要保留，供 Slice 5 發送
第一輪 training。

本 Slice 不會：

- 發送第一輪 training PATCH；
- 實作 FedProx local objective；
- 執行 Branch 或 Root aggregation；
- 接受 partial admission、minimum clients、補選、重新分組或替換 Leaf；
- 加入 automatic preparation retry 或 topology retry；
- persistence、resume 或 restart recovery；
- 新增永久 Root／Branch／Leaf role config；
- 將 planId 寫入 Go standard route state；
- 增加新的 public/private peer SBI；
- 修改 Release 18 OpenAPI YAML 或 generated free5GC OpenAPI module；
- 將同 vendor hierarchy metadata 描述成 3GPP 定義的 hierarchical FL procedure。

Slice 5 負責 rounds 與 aggregation；Slice 6 負責完整 terminal lifecycle closure 與 restart
hardening。Slice 4 仍必須完成本次 preparation failure 所直接需要的 bounded cleanup，
不能以 Slice 6 為理由留下 active subscriptions、correlations 或 process-local slot。

---

## 2. 已確認且不得重新決策的事項

本 Slice 直接繼承以下 decisions：

- Root、Branch、Leaf 都是標準 NWDAF，不新增 NF type；
- deployment 只配置 FL Server／Client capabilities，不配置 hierarchy role；
- Branch／Leaf role 由已驗證 assignment 在單一 planId 期間決定；
- Root 必須具備 FL Server；Branch 必須具備 FL Server 與 FL Client；Leaf 至少具備
  FL Client；
- topology 只存在 Root config；Branch 只收到自己的 assigned subtree；
- Branch 必須下載、處理並重新發布 bundle，不 transparent proxy Root URL；
- model bytes 由發布該 artifact 的 PyMTLF serving；Go 只傳遞 mLModelUrl；
- 第一版 strategy 固定為 FedProx、all、all、sample_weighted；
- 第一版 admission 唯一合法值為 complete_required；
- 任一 required Branch／Leaf failure 或 timeout 都拒絕完整 attempt；
- 失敗不自動重試、補選或恢復；operator 排除問題後啟動新 plan；
- process restart 建立全新 memory state；
- 每個 PyMTLF process 同時間只有一個 top-level experiment；
- standard mlCorreId、subscription ID、notifCorreId 與 hierarchy planId 保持分離；
- Go 不解析 hierarchy bundle、不保存 planId，也不決定 topology admission；
- private Root initiation API 與 standard Training service 的責任保持分離；
- 第一版不新增 expected Root ID、caller identity header、artifact signature 或 hierarchy
  trust handshake；
- statusReport 是 optional training status，不是 preparation completion proof；
- successful preparation 必須有 mLModelInfos；
- preparation callback 不帶 roundInd；
- Branch 若已形成完整 failure result，可以同時回 mLModelInfos 與 termTrainReq；
- Root 是整棵 topology admission 的唯一 owner；
- ADMITTED 只表示 participant snapshot 已凍結，不表示 round 0 已開始。

若 implementation 需要變更以上任一項，必須先更新 canonical plan。不得透過新增 role
config、讓 Go 保存 planId、接受 statusReport-only callback、把 Root URL 直接交給 Leaf，
或在 Slice 4 偷跑 round 0 來繞過本計畫。

---

## 3. Repository、branch 與責任歸屬

### 3.1 Baseline

| Repository | Branch | Revision | 狀態 |
| --- | --- | --- | --- |
| PyMTLF/ | feat/r18-hierarchical-federated-learning | 8e6d7d4 | clean；Slice 3 completed |
| NWDAF/ | feat/r18-hierarchical-federated-learning | b045423 | clean |
| nwdaf-docs/ | main | 5fbf4cd | clean；local commits 尚未 push |

預期 production repositories：

- PyMTLF/：hierarchy preparation business state、artifact processing、Branch orchestration、
  Leaf preparation、Root admission；
- NWDAF/：Release 18 notification conditional validation 與 relay regression。

Documentation repository：

- nwdaf-docs/。

NWDAF/ 與 PyMTLF/ 必須使用相同 feature branch 名稱
feat/r18-hierarchical-federated-learning，各自獨立 commit。nwdaf-docs/ 維持 main。

### 3.2 Responsibility boundary

Root PyMTLF：

- 延續 Slice 3 active plan；
- 等待並驗證 Branch preparation results；
- 建立 immutable admitted participant snapshot；
- 決定 ADMITTED 或 FAILED；
- rejection 時發動 upper-tier cleanup。

Branch PyMTLF：

- 驗證 Root assignment；
- 將 upper Client reservation bind 為 BRANCH；
- re-resolve assigned Leaves；
- republish Leaf assignments；
- 將 lower Server process attach 同一 experiment；
- 收集 lower outcomes；
- 建立 result bundle；
- 回報 upper Root；
- 失敗時清理 lower resources。

Leaf PyMTLF：

- 驗證 Branch assignment；
- 將 upper Client reservation bind 為 LEAF；
- 驗證模型與本地資料可用性；
- 成功時以 assignment bundle URL 回報 mLModelInfos；
- 失敗時以 termTrainReq 回報。

PyMTLF FL Client engine：

- 擁有 inbound standard subscription resource；
- 擁有 download／local preparation worker、delay timer 與 callback outbox；
- 根據已驗證 artifact contract 將 Branch work交給 hierarchy coordinator；
- 不自行規劃 topology。

PyMTLF FL Server engine：

- 擁有 outbound standard resources、correlation map、deadline waiting 與 cleanup；
- 收集每一 participant 的 standard callback；
- 不解讀 static topology，也不做整棵 topology admission。

Go NWDAF：

- 維持 existing public/private standard-shaped Training routing；
- 依 Release 18 schema驗證 callback shape與standard correlations；
- byte-preserving relay callback；
- 不解析 result bundle、不判斷 hierarchy role、不做 admission。

### 3.3 free5GC exemplar alignment

本計畫以 target NWDAF repository 的既有 ML Model Training handler、processor、route store
與 compat models 為第一直接證據。

free5GC local snapshot 的 PCF callback boundary只作 implementation-shape exemplar：

- NFs/pcf/internal/sbi/api_httpcallback.go：handler解析standard model後交給processor；
- NFs/pcf/internal/sbi/processor/notifier.go：processor擁有procedure decision；
- NFs/pcf/internal/sbi/consumer/notification.go：outbound callback URI與client call留在
  consumer boundary。

由此採用的結論只有「HTTP shape validation、procedure logic與outbound delivery維持分層」。
PCF 沒有 hierarchy FL contract，不能用來證明 result-bundle semantics；後者來自本專案
confirmed design。

---

## 4. Standards、OpenAPI 與實作基線

### 4.1 Release 18 notification contract

本地 official attachment TS29520_Nnwdaf_MLModelTraining.yaml 為 V18.14.0。
NwdafMLModelTrainNotif 只有 notifCorreId 為 schema-required field；其他欄位依 TS
conditional rule決定。

TS 29.520 V18.14.0 clause 5.5.6.2.8 NOTE 1 的直接規則是：

- delayEventNotif、mLModelInfos、termTrainReq 至少提供一項；
- delayEventNotif 與 mLModelInfos 互斥；
- delayEventNotif 與 termTrainReq 互斥；
- statusReport 是 optional，不計入上述至少一項；
- NOTE 1 沒有禁止 statusReport 與其他合法 outcome field 共存；
- mLModelInfos present 時至少一項；
- FL notification 的 mlCorreId 必須存在；
- notifCorreId 必須等於 subscription 中的 correlation。

NOTE 1 沒有逐字寫成「preparation success shall include mLModelInfos」。但結合 TS 23.288
step 9 的 join decision 語意，對成功且不延後、不終止的 preparation outcome 而言，
delayEventNotif 與 termTrainReq 都不符合結果，statusReport又不能單獨滿足最低條件，因此
mLModelInfos是剩下的標準結果載體。Slice 4將此視為規格推論；vendor-specific的部分是
bundle內的 hierarchy metadata，而不是 mLModelInfos作為成功 outcome本身。

因此下列 shape 必須成立：

| Payload shape | Wire result | Slice 4 interpretation |
| --- | --- | --- |
| statusReport only | reject | 不構成 preparation outcome |
| mLModelInfos | accept | stage-aware success/result |
| mLModelInfos + statusReport | accept | status不改變outcome |
| termTrainReq | accept | failure before valid result |
| mLModelInfos + termTrainReq | accept | record result first, then terminal failure |
| delayEventNotif | accept | bounded extension request |
| delayEventNotif + statusReport | accept | optional status不改變delay semantics |
| delayEventNotif + mLModelInfos | reject | TS mutual exclusion |
| delayEventNotif + termTrainReq | reject | TS mutual exclusion |

成功 callback response 仍是 204 No Content。Slice 4 不新增 callback path 或 status code。

### 4.2 Existing NWDAF gap

NWDAF/internal/compat/mlmodeltraining 是 pinned generated dependency 缺少 Release 18
MLModelTraining schema 時建立的 isolated compatibility boundary；本 Slice 繼續修改此
handwritten compat validator，不手改 generated code。

目前 validateNotificationShape 有兩個與 V18.14.0 不一致的行為：

1. 將 statusReport 計入「至少一項」，因此接受 statusReport-only；
2. 將 delayEventNotif 與 statusReport 也視為互斥，限制比 NOTE 1 更嚴格。

既有 processor test 明確characterize statusReport-only relay。Slice 4 要以新的標準證據
取代這個歷史行為，並證明：

- invalid shape在relay前回 standard error；
- mLModelInfos callback完整relay；
- mLModelInfos + termTrainReq完整relay，不丟任一欄位；
- generation lease、route lookup與byte-preserving forwarding沒有 regression。

### 4.3 Existing PyMTLF wire gap

PyMTLF wire validator有相同兩項 gap：接受 statusReport-only，並禁止
delayEventNotif + statusReport。

FLClientEngine 的 plain preparation success目前產生 statusReport-only callback；
FLServerEngine.receive_notification 也把 statusReport視為 preparation completion，並把
沒有 roundInd 的 mLModelInfos誤判為 round callback。

此外 receive_notification 現在用互斥 elif順序處理 outcomes。當 callback 同時具有
mLModelInfos與termTrainReq時，會先進 termination branch而沒有保留model result。Slice 4
必須改為「先分類完整shape，再按active stage處理」，不能只更換sender payload。

### 4.4 Existing Slice 1–3 primitives

已存在且應重用：

- BranchAssignmentMetadata、LeafAssignmentMetadata、PreparationResultMetadata；
- strict hierarchy artifact roles與manifest validation；
- digest-addressed tar.gz publication與download；
- Branch republish Leaf assignment primitive；
- Branch publish preparation result primitive；
- exact-instance HierarchyNodeResolver；
- process-scoped FLExperimentRegistry；
- Root assignment publication；
- FLServerEngine.start_hierarchy_preparation；
- Root PREPARATION_WAITING state與private status API；
- partial dispatch rollback與bounded cleanup。

尚未接上 production orchestration：

- FLClient background worker尚未識別hierarchy assignment；
- registry.bind_plan尚無production caller；
- Branch尚未建立lower Server process；
- hierarchy Server process尚未等待或分類callbacks；
- Root尚未下載Branch result或做admission；
- hierarchy-aware parent DELETE／terminal cleanup尚未串起；
- flat preparation sender／receiver仍使用舊statusReport semantics。

---

## 5. Identity 與跨 boundary data flow

### 5.1 Identity ownership

| Value | Producer | Crossing boundary | Receiver use |
| --- | --- | --- | --- |
| planId | Root PyMTLF | assignment/result manifest | bind同一hierarchy attempt |
| upper mlCorreId | Root FL Server | Root→Branch standard subscription | upper process correlation |
| lower mlCorreId | Branch FL Server | Branch→Leaf standard subscription | lower process correlation |
| subscription ID | receiving Go/PyMTLF resource | Location／private route | CRUD ownership |
| notifCorreId | sending FL Server | standard subscription／callback | exact participant lookup |
| Root NF instance ID | Root context | Branch assignment publisher field | logical plan parent |
| Branch NF instance ID | Root topology／Branch context | assignment recipient／result publisher | exact subtree identity |
| Leaf NF instance ID | Root topology／Branch resolver | assignment recipient／result partition | exact participant identity |
| reservation ID | local PyMTLF registry | never crosses process | upper/lower ownership link |
| artifact digest | publishing PyMTLF | URL + X-Artifact-SHA256 | integrity validation |

Go route state不得新增 planId或reservation ID。Cross-tier relationship只存在PyMTLF registry、
validated artifacts與engine process maps。

### 5.2 Upper publisher identity limitation

Branch從standard inbound preparation request本身無法取得經驗證的actual requester NF
instance ID，而且第一版已決定不新增caller identity header或expected Root config。

因此 Branch assignment ingress 要做：

- archive／URL／digest／file-set validation；
- contract version與message type validation；
- intended recipient必須等於containing NWDAF自己的NF instance ID；
- planId必須是canonical UUIDv4且未retired；
- publisher、recipient與assigned Leaves必須符合manifest invariants；
- model event與interoperability必須符合standard request。

publisher_nf_instance_id 在此是同 vendor plan宣告的logical parent，不是實際HTTP caller的
cryptographic proof。Implementation、test與文件不得把它描述成caller authentication。

Root收到Branch result時則已有Slice 3 immutable topology snapshot與expected Branch identity，
因此可以獨立驗證result publisher與recipient；這是plan consistency check，仍不是transport
caller attestation。

### 5.3 State creation order

Inbound Client create維持：

1. Go建立standard route並把internalized request交給PyMTLF；
2. PyMTLF完成standard synchronous validation；
3. FLClient產生subscription ID；
4. registry以subscription ID + upper mlCorreId建立provisional reservation；
5. 回201 Created；
6. background worker下載artifact；
7. 驗證hierarchy contract後才bind planId與BRANCH／LEAF role；
8. Branch role才允許attach lower Server process。

任何artifact I/O、NRF call、HTTP dispatch或condition wait都不得在registry lock內執行。

### 5.4 Root attempt snapshot補強

Slice 3目前只在Root worker local variables保留topology與published assignments；worker進入
PREPARATION_WAITING後，這些值沒有全部留在request record。Slice 4 admission需要獨立於
Branch result取得expected values，因此Root record必須在dispatch成功前保存immutable：

- Root containing NF instance ID；
- base artifact key與model/preprocessing contract digests；
- canonical topology snapshot；
- 每個Branch的expected Leaf IDs；
- 每個Branch assignment URL與digest；
- upper Server process ID。

這些值由Root在Slice 3 validation／publication階段產生，不從Branch callback或result bundle
反推。Snapshot只存在process memory，失敗cleanup後可保留最小queryable status，但不得用來
restart recovery。

---

## 6. Assignment ingress 與角色判定

### 6.1 Download entrypoint

FLWorkspace應增加一個assignment ingress seam，接收：

- assignment URL；
- containing NWDAF NF instance ID；
- optional expected planId，僅用於已知下游流程；
- expected event與model interoperability。

它必須回傳ValidatedHierarchyArtifact與typed
BranchAssignmentMetadata／LeafAssignmentMetadata，不只回傳raw manifest。

此 seam 必須：

- 沿用allowed origins、redirect policy、size limits與digest header validation；
- 將downloaded bytes先放staging；
- 完整驗證後才移到plan workspace；
- failure時不留下partially admitted artifact；
- 不從URL、HTTP header或request body推測hierarchy role；
- 只從validated message_type判定BRANCH_ASSIGNMENT或LEAF_ASSIGNMENT。

現有download_hierarchy仍可供已知publisher／recipient的result path使用；若需generalize，
應以清楚區分「declared publisher」與「independently expected publisher」的method contract
完成，不以傳入manifest自己的publisher來假裝獨立驗證。

### 6.2 Branch assignment validation

BRANCH_ASSIGNMENT 必須確認：

- local PyMTLF同時配置FL Client與FL Server engines；
- containing NWDAF context仍廣告SERVER_AND_CLIENT；
- intended recipient等於local nfInstanceId；
- assigned Leaf list非空、sorted、unique；
- Root、Branch、Leaves identities互不衝突；
- admission mode精確為complete_required；
- strategy精確符合Slice 1 typed first-version contract；
- bundle analytics event／model interoperability與standard subscription一致；
- upper subscription仍是current revision且registry reservation仍active。

成功後，以reservation ID bind planId與ExperimentRole.BRANCH。

### 6.3 Leaf assignment validation

LEAF_ASSIGNMENT 必須確認：

- local PyMTLF配置FL Client engine；
- containing NWDAF context仍廣告FL_CLIENT或SERVER_AND_CLIENT；
- intended recipient等於local nfInstanceId；
- parent_branch_nf_instance_id等於publisher_nf_instance_id；
- strategy、event、interoperability與subscription一致；
- planId未retired且resource revision仍current。

成功後，以reservation ID bind planId與ExperimentRole.LEAF。

若一個combined-capability NWDAF收到LEAF_ASSIGNMENT，它仍是本plan的Leaf，不會因為有Server
engine自行展開下層。Role由message type決定，不由deployment capability自動升級。

### 6.4 Plain FL preservation

沒有hierarchy artifact_role的completed model bundle繼續走既有flat preparation：

- 不bind planId；
- 不建立Branch coordinator；
- 執行local dataset preparation；
- 成功callback改為mLModelInfos指回已下載的input model URL；
- 後續flat rounds維持原本subscription resource。

Unknown或malformed hierarchy role必須fail closed，不可fallback成plain FL。

---

## 7. Branch lower-tier orchestration

### 7.1 Owner

預計新增PyMTLF internal FLBranchPreparationCoordinator，作為FL Client與FL Server engines之間
的business coordinator。名稱可在implementation review依現有module naming微調，但責任
不得散入FastAPI handler或Go route。

Coordinator擁有：

- one Branch plan execution；
- assigned Leaf immutable list；
- exact Leaf resolution snapshot；
- Leaf assignment publication mapping；
- lower Server process ID；
- lower preparation result classification；
- result bundle publication；
- failure cleanup orchestration。

Coordinator不擁有：

- standard subscription maps；
- callback correlation map；
- static topology planning；
- model training round state；
- Root admission；
- HTTP route parsing。

### 7.2 App wiring

App assembly應只在同一PyMTLF同時存在FL Client與FL Server engines時建立Branch
coordinator。這不是role config；它只代表runtime具備被assignment指派為Branch的能力。

建構順序必須讓：

- FLExperimentRegistry只有一個shared instance；
- FLServerEngine先可供coordinator使用；
- FLClientEngine取得optional coordinator dependency；
- hierarchy resolver與artifact service有單一明確owner與close path；
- Root coordinator與Branch coordinator若共用resolver，不double close；
- shutdown先關閉Root／Branch admission，再喚醒waiters，最後關閉engines與HTTP clients。

### 7.3 Leaf re-resolution

Branch不得只相信Root已做過的discovery snapshot。對每個assigned Leaf，Branch在lower
dispatch前重新呼叫existing exact-instance resolver，要求：

- returned nfInstanceId精確相等；
- nfStatus為REGISTERED；
- FL Client或SERVER_AND_CLIENT capability；
- matching analytics event；
- matching model interoperability；
- exactly one eligible registered ML Model Training service；
- endpoint來自resolved service，不來自topology或bundle。

所有Leaves必須先resolve成功，才開始publish／dispatch。任一resolution failure直接形成
Branch FAILED result；不得部分fan-out後再假裝完整preparation。

### 7.4 Republish

對每個Leaf，以Slice 1 primitive從validated Branch assignment建立LEAF_ASSIGNMENT：

- publisher = containing Branch NF instance ID；
- recipient = target Leaf NF instance ID；
- parent Branch = same publisher；
- planId與strategy保持不變；
- model.py、model.npy、scaler.pkl與input contract保持一致；
- URL由Branch PyMTLF public_base_url發布；
- participant path與digest各自獨立。

Branch不可把assigned_leaf_nf_instance_ids完整清單放入Leaf bundle，也不可把Root URL原樣
轉交。

### 7.5 Lower dispatch

本文件中的 lower dispatch，是 Branch 開始透過 FL Server engine 向 Leaves 建立 standard
preparation subscription 的分界。`pre-dispatch` 精確指此分界之前：Branch 雖已收到 Root
assignment，但尚未向任何 Leaf 建立 lower preparation resource。若 Branch 在 assignment
下載／驗證、plan bind、Leaf resolution、Leaf bundle publication 或 lower process 建立前
自行失敗，便不啟動 lower-tier preparation，直接走 Branch failure result／termination
path；此時不存在可等待的 Leaf callback。

FLServerEngine.start_hierarchy_preparation應generalize成tier-neutral internal primitive：

- Root傳入Branch targets；
- Branch傳入Leaf targets；
- target identity、resolved SelectedTarget與assignment URL必須一致；
- participants canonical sorted、unique；
- process使用新lower mlCorreId；
-每個Leaf使用新notifCorreId與standard resource；
- process透過existing registry.attach_server接到same planId reservation；
- outbound request仍由Go private standard-shaped route送到peer public SBI。

Internal target field不得永久命名成branch-only；建議改為participant_nf_instance_id。
這是private refactor，不改wire contract。

### 7.6 Branch不做local preparation

收到BRANCH_ASSIGNMENT時，Branch upper FL Client代表整個subtree，不是額外local training
participant。因此：

- 不要求Branch本機建立training dataset；
- 不把Branch自己的sample count加入Leaf集合；
- preparation成功取決於lower Leaves；
- Slice 5的Branch輸出將來是Leaf aggregate，不是Branch local model。

收到LEAF_ASSIGNMENT或plain model的Client才執行local data availability preparation。

---

## 8. Leaf preparation 與 callback

Leaf完成assignment validation與plan bind後，沿用既有DatasetCoordinator、
TrainingDatasetBuilder與deadline timer：

1. 從standard request建立scope與time window；
2. 驗證event／model interoperability；
3. 準備local training dataset；
4. 確認minNumSamples；
5. 凍結dataset snapshot與model/preprocessing contract digests；
6. 成功時將resource設為PREPARATION_RESULT_PENDING；
7. 透過existing outbox送successful callback；
8. callback 204後設為PREPARED。

成功callback固定：

    {
      "notifCorreId": "<lower-notif-correlation>",
      "mlCorreId": "<branch-lower-process-id>",
      "mLModelInfos": [
        {
          "event": "UE_COMMUNICATION",
          "mLFileAddr": {
            "mLModelUrl": "<branch-published-leaf-assignment-url>"
          }
        }
      ]
    }

不得提供roundInd。statusReport可附加但第一版sender不需要產生，避免再次形成兩套success
contract。

Leaf在有效result產生前失敗時固定送：

    {
      "notifCorreId": "<lower-notif-correlation>",
      "mlCorreId": "<branch-lower-process-id>",
      "termTrainReq": "NOT_AVAILABLE_ML_TRAIN"
    }

Plain FL Client preparation success使用同一mLModelInfos contract，URL指回它已下載並驗證的
plain input model。這是本Slice必須覆蓋的non-hierarchical regression。

---

## 9. Stage-aware callback collection

### 9.1 Wire validation與stage interpretation分離

Go與Py wire models只驗證TS shape。FLServerEngine依active process stage解讀：

- PREPARATION_CREATING／PREPARATION_WAITING：
  - mLModelInfos without roundInd = preparation artifact／acknowledgement；
  - termTrainReq = participant preparation failure；
  - mLModelInfos + termTrainReq = result artifact available, then failure；
  - delayEventNotif = preparation extension request；
- ROUND_*／FINAL_VALIDATION_*：
  - mLModelInfos必須帶matching roundInd；
  - round behavior留給existing flat flow與Slice 5 hierarchy extension；
- callback不能因為有statusReport就改變stage或完成predicate。

Preparation callback帶roundInd要reject。Round callback缺roundInd或round不符也要reject。

### 9.2 Locking

receive_notification在engine lock／process condition內只做：

- correlation lookup；
- identity/stage validation；
- canonical payload digest；
- duplicate／conflict classification；
- 保存deep-copied notification；
- 更新per-participant outcome marker；
- notify waiters。

Artifact download、manifest parsing、NRF、callback delivery、DELETE cleanup不得在lock內執行。

### 9.3 Duplicate與late callbacks

- exact same payload digest replay：idempotent 204；
- same participant conflicting second terminal payload：400並fail current attempt；
- unknown notifCorreId：404；
- callback correlation matching但process已terminal／cleaning：reject，不復活state；
- callback plan只能由下載後artifact驗證，不由notification body推測；
- callback arrival與deadline race以condition lock下第一個terminal transition為準；
- accepted attempt的latepreparation callback不得改participant snapshot。

### 9.4 Collection result

FLServerEngine提供immutable hierarchy preparation collection result，至少包含：

- process ID與planId；
- expected participant IDs；
- per-participant received notification或absence；
- participant termination cause；
- requested／granted delay extensions；
- deadline-expired participant IDs；
- cleanup handles仍由engine持有。

Engine不把prepared判斷硬編成單一boolean；Root與Branch coordinator依tier-specific artifact
contract判斷。

---

## 10. Deadline、failure classification 與 result bundle

### 10.1 Waiting rule

第一版waiting_policy固定all。此 waiting rule 只在 lower dispatch 已成功、participant
callbacks 已進入等待狀態後適用；它不要求 pre-dispatch failure 啟動 Leaves，也不要求
Branch 等待尚未建立的 callback。

進入等待狀態後，Branch與Root都等待：

- 所有assigned participants收到terminal preparation outcome；或
- effective preparation deadline到期；或
- local shutdown／unrecoverable process failure。

收到單一termTrainReq後不得立即結束collection；Branch仍在bounded deadline內收集其他
participants，使result partition一次反映實際prepared、failed與timed-out集合。這不代表
partial admission；最終仍reject。

Existing delay policy可延長effective deadline，但：

- extension次數與總秒數受config限制；
- invalid或超budget request不產生unbounded wait；
- transport callback retry與cleanup retry是bounded delivery mechanics，不是topology retry。

### 10.2 Leaf classification

Branch對每個Leaf分類：

- prepared：收到合法mLModelInfos、URL精確指回該Leaf被發布的assignment，且沒有
  termTrainReq；
- failed：收到termTrainReq、conflicting callback、invalid acknowledged artifact，或
  local processing error；
- timed out：deadline前沒有合法terminal outcome。

Failure cause映射到existing bounded enum：

| Evidence | PreparationFailureCause |
| --- | --- |
| exact NRF resolution failure | DISCOVERY_FAILED |
| capability／event／interoperability mismatch | CAPABILITY_MISMATCH |
| assignment identity／plan mismatch | INVALID_ASSIGNMENT |
| archive／digest／model contract invalid | INVALID_BUNDLE |
| local data／min samples不符合 | REQUIREMENTS_NOT_MET |
| standard termTrainReq NOT_AVAILABLE_ML_TRAIN | NOT_AVAILABLE_ML_TRAIN |
| local coordinator／delivery invariant failure | INTERNAL_ERROR |

Result bundle不放exception、stack、peer response body或任意log text。

若Branch在lower dispatch前自行失敗，因為尚未建立lower preparation resources，所以不套用
上述waiting rule，也不等待Leaf callbacks。Branch仍應在parent assignment有效且result
publication可行時形成完整partition：

- 直接發生resolution／capability／publication failure的Leaf使用對應精確cause；
- 因complete_required fail-fast而沒有啟動的其他Leaves標為
  NOT_AVAILABLE_ML_TRAIN；
- 沒有真的等到deadline的Leaf不得標為timed out；
- 如果連valid result bundle都無法形成，才退回termTrainReq-only。

### 10.3 Result construction

Branch使用Slice 1 publish_preparation_result primitive。Result必須：

- publisher = containing Branch；
- intended recipient = Root assignment publisher；
- planId與parent assignment一致；
- assigned_client_nf_instance_ids精確等於assigned Leaves；
- prepared、failed、timed-out互斥且union完整；
- READY只允許全部Leaves prepared；
- 其他情況一律FAILED；
- model／preprocessing／weights contract與Branch input assignment一致。

### 10.4 Upper callback

READY：

    {
      "notifCorreId": "<upper-notif-correlation>",
      "mlCorreId": "<root-upper-process-id>",
      "mLModelInfos": [
        {
          "event": "UE_COMMUNICATION",
          "mLFileAddr": {
            "mLModelUrl": "<branch-preparation-result-url>"
          }
        }
      ]
    }

FAILED但已形成完整result：

    {
      "notifCorreId": "<upper-notif-correlation>",
      "mlCorreId": "<root-upper-process-id>",
      "mLModelInfos": [
        {
          "event": "UE_COMMUNICATION",
          "mLFileAddr": {
            "mLModelUrl": "<branch-preparation-result-url>"
          }
        }
      ],
      "termTrainReq": "NOT_AVAILABLE_ML_TRAIN"
    }

Branch在形成valid result前失敗則只送termTrainReq。Root收到models + termination時，必須先
下載／驗證／記錄result partition，再將attempt標為failure。

---

## 11. Root admission

### 11.1 Root states

RootRequestState增加：

- PREPARATION_EVALUATING：已取得terminal upper outcomes，正在驗證result bundles；
- ADMITTED：完整topology通過complete_required，participant snapshot已凍結。

FAILED沿用terminal state，並增加bounded failure causes：

- PREPARATION_FAILED；
- PREPARATION_TIMEOUT；
- RESULT_VALIDATION_FAILED；
- ADMISSION_REJECTED。

Public private-status detail仍只提供stable phase-level wording，不回傳peer response或內部
exception。

### 11.2 Result validation

Root對每個expected Branch確認：

- callback由current upper Server process correlation取得；
- exactly one mLModelInfos item；
- event與plan model event一致；
- download URL與digest通過workspace validation；
- artifact role為HIERARCHY_PREPARATION_RESULT；
- message type為PREPARATION_RESULT；
- planId等於active plan；
- publisher等於expected Branch；
- intended recipient等於Root containing NWDAF；
- assigned set等於Root immutable topology中該Branch Leaves；
- result partition invariants成立；
- model／preprocessing／input weights contract等於Root assignment；
- duplicate Branch result bytes idempotent，conflicting result fail closed。

### 11.3 complete_required decision

Accept condition只有：

- 每個configured Branch都提供valid READY result；
- 每個Branch的prepared set精確等於assigned Leaves；
- 沒有failed或timed-out Branch／Leaf；
- Root base model仍是Slice 3啟動時的artifact；
- experiment registry與Server process仍是同一plan。

成功後Root建立immutable snapshot：

    planId
      -> Branch NF instance ID
           -> prepared Leaf NF instance IDs
           -> upper subscription identity
           -> Branch result digest

Snapshot存在PyMTLF process memory，不寫入Go route、config或durable state。

Root與FL Server process進入ADMITTED／READY-for-round狀態，但不修改subscription、
不設定roundInd、不送round 0。Slice 5接手這些active resources。

### 11.4 Rejection

任一條件不成立：

1. Root將request設為FAILED並設定failure latch；
2. Server engine刪除已建立的upper Branch subscriptions；
3. Branch收到parent DELETE後清理lower Leaf subscriptions；
4. Root釋放plan artifacts與registry reservation；
5. 不自動建立新plan。

Root若沒有收到valid Branch result，不能用Root在Slice 3做過的Leaf discovery推測prepared
狀態。

---

## 12. Lifecycle 與 cleanup

### 12.1 Success retention

ADMITTED時不得清理：

- Root→Branch subscriptions；
- Branch→Leaf subscriptions；
- Root／Branch Server processes；
- Branch／Leaf upper Client resources；
- assignment/result bundles；
- registry active reservation；
- correlation maps；
- prepared Leaf dataset snapshots。

它們是Slice 5 round execution的輸入。

### 12.2 Failure cleanup order

Root failure：

1. Root Server停止接受新的preparation callback；
2. DELETE upper Branch resources；
3. 每個Branch取消／清理lower Server process；
4. Branch DELETE lower Leaf resources；
5. 各engine移除correlations、timers、worker references；
6. release hierarchy workspace plan；
7. registry ACTIVE -> TERMINAL -> CLEANING -> released；
8. Root request保留queryable FAILED snapshot與failure latch。

Branch local failure：

1. 能形成result時先發布result；
2. 透過upper callback送result + termination；
3. cleanup lower resources；
4. 等待Root DELETE或bounded local terminal cleanup完成；
5. 不啟動replacement Leaves。

Leaf local failure：

1. 送termTrainReq；
2. callback delivery達bounded終點；
3. 等待Branch DELETE或執行local terminal cleanup；
4. 不重新執行dataset preparation。

### 12.3 Parent DELETE

Hierarchy-bound Client resource收到parent DELETE時是experiment cancellation signal：

- PREPARING中的stale worker結果不得再publish callback；
- timer取消；
- Branch coordinator被通知並cleanup lower process；
- Leaf dataset work以revision／resource-existence fence失效；
- callback outbox不再建立新的delivery；
- bound resource依registry lifecycle順序移除；
- DELETE保持idempotent。

Existing flat FL「operation in progress時拒絕DELETE」behavior不得無條件改掉。Hierarchy-aware
cancellation只套用已bind BRANCH／LEAF的resource，並需要focused regression。

### 12.4 Callback delivery failure

Outbox維持existing bounded retry。若terminal callback一直無法delivery：

- local resource記錄stable failure；
- hierarchy coordinator執行可行cleanup；
- 不自動重建subscription或plan；
- parent最後依deadline將participant分類timeout並清理；
- external Go route可能暫時存在，但後續DELETE／Go restart要能idempotently收斂。

### 12.5 Shutdown與restart

- shutdown先關閉new admission；
- 喚醒Root／Branch／Server waiters；
- 停止timers與executors；
- bounded cleanup已知remote resources；
- 關閉HTTP clients；
- 不讀workspace重建active plan；
- new process registry與Root failure latch都是fresh state；
- old callback得到route miss／stale rejection，不建立新experiment。

---

## 13. 預計 production 變更範圍

### 13.1 PyMTLF/

預計修改：

- src/py_mtlf/wire/ml_model_training.py
  - 修正NOTE 1 conditional validation；
  - statusReport不再構成terminal outcome。
- src/py_mtlf/core/fl_workspace.py
  - 增加assignment ingress validation seam；
  - 保留publisher expectation語意區分。
- src/py_mtlf/core/fl_client.py
  - hierarchy assignment detection；
  - Branch／Leaf plan bind；
  - plain／Leaf success mLModelInfos sender；
  - hierarchy cancellation與terminal callback hooks。
- src/py_mtlf/core/fl_server.py
  - tier-neutral hierarchy targets；
  - stage-aware callback classification；
  - hierarchy wait／deadline collection；
  - models + termination先記錄result；
  - admitted retention與failure cleanup。
- src/py_mtlf/core/fl_root.py
  - 保存Root-owned immutable topology／assignment／base contract snapshot；
  - wait／evaluate Branch results；
  - ADMITTED state與participant snapshot；
  - rejection cause與cleanup。
- src/py_mtlf/core/fl_hierarchy_artifacts.py
  - 只在existing primitive不足時補typed helper；不得搬入orchestration。
- src/py_mtlf/core/fl_experiment.py
  - 只補hierarchy cancellation／cleanup直接需要的atomic lifecycle seam。
- src/py_mtlf/app.py
  - wiring與shutdown ownership。
- 新增或等價的 src/py_mtlf/core/fl_branch.py
  - Branch lower-tier coordinator。

預計新增／修改tests：

- tests/test_ml_model_training_wire.py；
- tests/test_fl_client.py；
- tests/test_fl_server.py；
- tests/test_fl_root.py；
- tests/test_fl_workspace.py或test_fl_hierarchy_artifacts.py；
- tests/test_fl_experiment.py；
- tests/test_runtime_modes.py；
- 新增tests/test_fl_branch.py；
- 必要的API integration tests。

### 13.2 NWDAF/

預計修改：

- internal/compat/mlmodeltraining/validation.go；
- internal/compat/mlmodeltraining/validation_test.go；
- internal/compat/mlmodeltraining/models_test.go；
- internal/sbi/processor/ml_model_training_test.go。

只有focused test證明processor production code也需改動時，才修改
internal/sbi/processor/ml_model_training.go。預期既有processor已byte-preserving relay合法
payload，因此主要production delta應限制在compat conditional validator。

不得：

- 修改Release 18 OpenAPI YAML；
- hand-edit generated free5GC openapi；
- 在MLModelTrainingSubscriptionRoute增加planId／role；
- 新增public route；
- 讓Go下載hierarchy artifacts；
- 在Go做Root admission。

### 13.3 nwdaf-docs/

Implementation完成後在本文件補：

- per-repository production commits；
- exact focused／full verification commands與results；
- mandatory code review findings與remediation；
- actual changed-file scope；
- approved plan deviations或none；
- integration verification gaps。

---

## 14. 實作順序與 checkpoints

### Checkpoint 1：Notification contract correction

先以Go與Py wire tests證明：

- statusReport-only從accept改成reject；
- delay + status合法；
- model + termination合法；
- delay + model／termination非法；
- existing round notification仍合法。

接著修改兩邊validators。再把flat FL Client preparation success sender改為mLModelInfos，並
修改FL Server preparation stage interpretation。此checkpoint先不接hierarchy orchestration。

### Checkpoint 2：Assignment ingress與Leaf path

- typed assignment download；
- local recipient／plan／event／interoperability validation；
- registry bind LEAF；
- local data preparation；
- assignment URL success callback；
- failure termination；
- parent DELETE cancellation；
- plain FL regression。

### Checkpoint 3：Branch lower-tier orchestration

- app wiring；
- registry bind BRANCH；
- exact Leaf re-resolution；
- Leaf-specific republish；
- tier-neutral Server targets；
- attach lower process；
- fan-out rollback；
- Branch no-local-dataset proof。

### Checkpoint 4：Outcome collection與Branch result

- stage-aware callback collection；
- deadline／delay handling；
- prepared／failed／timed-out partition；
- result bundle publication；
- READY callback；
- failure result + termination callback；
- duplicate／conflict tests。

### Checkpoint 5：Root admission與cleanup

- Root wait extension；
- result download／identity validation；
- immutable participant snapshot；
- ADMITTED state；
- complete_required rejection；
- upper-to-lower cleanup cascade；
- no round 0 assertion；
- private status state regression。

### Checkpoint 6：Full verification與不中斷review

1. 執行focused tests；
2. 執行PyMTLF full pytest與Ruff；
3. 執行NWDAF make test、make build、make lint；
4. 完整review Slice 4 diff、plan與direct dependencies；
5. admitted finding先寫deterministic failing test；
6. 直接remediate並做targeted follow-up review；
7. 若remediation invalidates full verification，重跑；
8. repository-separated commits；
9. 回填本文件implementation record。

---

## 15. 驗收測試矩陣

### 15.1 Standard wire

- Go/Py拒絕statusReport-only；
- Go/Py接受mLModelInfos-only；
- Go/Py接受mLModelInfos + statusReport；
- Go/Py接受termTrainReq-only；
- Go/Py接受mLModelInfos + termTrainReq；
- Go/Py接受delayEventNotif + statusReport；
- Go/Py拒絕delay + models與delay + termination；
- notifCorreId／mlCorreId mismatch拒絕；
- preparation roundInd拒絕；
- round callback缺少／錯誤roundInd拒絕。

### 15.2 Assignment ingress

- valid Branch assignment bind BRANCH；
- valid Leaf assignment bind LEAF；
- combined node收到Leaf assignment仍為LEAF；
- wrong recipient；
- retired／conflicting planId；
- invalid digest／header／archive；
- unknown artifact role；
- event／interoperability mismatch；
- plain bundle不誤判hierarchy；
- invalid hierarchy不fallback plain。

### 15.3 Branch flow

- Branch不執行local dataset preparation；
- all assigned Leaves重新exact-resolve；
- capability／service mismatch fail；
- pre-dispatch failure不建立任何lower preparation resource，也不等待Leaf callback；
- every Leaf取得recipient-specific Branch URL；
- Root URL不出現在Leaf request；
- lower resources canonical dispatch；
- partial dispatch rollback；
- same reservation attach lower Server process；
- second/conflicting Server attach拒絕。

### 15.4 Leaf outcomes

- all prepared；
- one termTrainReq不提前結束collection，仍收集其他Leaves至全部terminal或deadline；
- one invalid acknowledgement URL；
- one timeout；
- delay extension within budget；
- extension count／budget exceeded；
- exact duplicate callback idempotent；
- conflicting duplicate fail；
- late callback不改terminal result；
- parent DELETE取消in-flight work。

### 15.5 Branch result

- READY partition exact cover；
- FAILED prepared／failed partition exact cover；
- timeout partition exact cover；
- failure cause bounded enum；
- model/preprocessing/weights unchanged；
- result recipient Root；
- result publisher Branch；
- failure result callback包含models + termination；
- pre-result failure只含termination；
- statusReport presence不影響interpretation。

### 15.6 Root admission

- all Branch READY -> ADMITTED；
- any Branch failed -> FAILED；
- any Leaf failed／timeout -> FAILED；
- Branch callback timeout -> FAILED；
- wrong plan／publisher／recipient／assigned set -> FAILED；
- base model changed -> FAILED；
- models + termination先記錄partition再fail；
- participant snapshot immutable；
- ADMITTED沒有round PATCH；
- failure latch阻止automatic degradation retrigger；
- explicit operator attempt遵守existing fresh-attempt rule。

### 15.7 Lifecycle

- failure cleanup刪除upper與lower resources；
- correlation maps清空；
- artifacts按owner釋放；
- registry release順序正確；
- ADMITTED保留resources；
- callback delivery failure bounded；
- shutdown喚醒waiters；
- new process不恢復plan；
- stale callback不建立record；
- flat FL delete／round／final validation behavior無regression。

### 15.8 Verification commands

Planning checkpoint：

    git diff --check

Production implementation至少：

PyMTLF：

    .venv/bin/pytest -q <focused Slice 4 tests>
    .venv/bin/pytest -q
    .venv/bin/ruff check .

NWDAF：

    go test ./internal/compat/mlmodeltraining ./internal/sbi/processor
    make test
    make build
    make lint

Real NRF與多NWDAF deployment若本地環境不可用，記為integration verification gap，不可用
unit mocks宣稱end-to-end deployment通過。

---

## 16. 風險與控制

### 16.1 statusReport歷史行為被誤當相容性要求

控制：以TS 29.520 V18.14.0 NOTE 1與OpenAPI為直接證據，Go/Py同checkpoint修改，並保留flat
FL regression。

### 16.2 mLModelInfos在preparation被誤判round result

控制：stage + roundInd + correlation共同判斷；statusReport不驅動state。

### 16.3 Branch在下載前不知道Root identity

控制：不新增不可取得的expected Root欄位；驗證local recipient、contract與logical
publisher invariants，並明確不宣稱caller attestation。

### 16.4 Branch重複本地訓練

控制：BRANCH_ASSIGNMENT走subtree coordinator，不進local dataset preparation；測試證明
dataset builder未被呼叫。

### 16.5 callback帶models + termination時result遺失

控制：移除mutually exclusive elif interpretation；先保存／驗證artifact，再套terminal
state。

### 16.6 deadline後未完整partition

控制：collection result明確列出absence，deadline時轉成timed-out set，result union必須完整
cover assigned set。

### 16.7 failure cleanup造成跨engine deadlock

控制：registry lock內無I/O；engine condition內不下載artifact；cleanup順序固定，並以Event／
barrier做deterministic concurrency tests。

### 16.8 parent DELETE無法停止child work

控制：只對已bind hierarchy resource增加cancellation path，以revision／resource existence
fence忽略stale worker；flat behavior保留。

### 16.9 admission後提前清理或提前round

控制：ADMITTED明確保留resources；Slice 4 tests同時assert沒有roundInd PATCH。

### 16.10 result bundle被當作trained model

控制：artifact role與message type明確為HIERARCHY_PREPARATION_RESULT；model bytes保持input
contract，文件不宣稱preparation產生新trained model。

---

## 17. Decision gates

目前沒有待確認的architecture gate。Implementation details先固定：

| Item | Slice 4 decision |
| --- | --- |
| success callback | mLModelInfos required；statusReport optional |
| preparation roundInd | absent |
| Leaf success URL | exact Branch-published Leaf assignment URL |
| Branch success URL | Branch-published preparation-result URL |
| complete result failure | mLModelInfos + termTrainReq |
| failure before result | termTrainReq only |
| admission | complete_required only |
| pre-dispatch Branch failure | do not start lower preparation；report without waiting for Leaf callbacks |
| post-dispatch waiting | collect all terminal outcomes or stop at bounded deadline；one failure does not finish collection |
| accepted state | ADMITTED；no round 0 |
| peer identity | logical manifest identity；no new caller header |
| Go role | standard validation and relay only |
| retry | bounded delivery/cleanup only；no topology retry |

若實作證明下列任一情況，必須停止並回到plan decision：

- standard callback route無法在不改Release 18 contract下傳遞models + termination；
- Branch無法從existing inbound resource取得assignment URL或local recipient identity；
- registry無法讓Branch upper Client與lower Server共享同一reservation而保持single-active；
- parent DELETE cancellation必須改變所有flat FL public semantics；
- complete_required結果無法在existing result bundle表達完整partition；
- ADMITTED resource retention必須提前實作Slice 5 round state；
- required expected peer identity只能靠新增跨process contract取得。

一般internal class名稱、method signature、test fixture placement與private state enum微調不構成
decision gate，只要ownership、wire contract與acceptance criteria不變。

---

## 18. Review checklist

### 18.1 Standard contract

- [ ] statusReport-only rejected in Go and Py
- [ ] mLModelInfos + termTrainReq preserved
- [ ] delay mutual exclusions match TS NOTE 1
- [ ] preparation callback has no roundInd
- [ ] Go route remains hierarchy-unaware
- [ ] no OpenAPI/generated-code change

### 18.2 Assignment and role

- [ ] role comes only from validated assignment
- [ ] no role config
- [ ] local recipient is checked
- [ ] publisher limitation is not described as caller authentication
- [ ] plain FL does not bind hierarchy plan
- [ ] invalid hierarchy does not fallback

### 18.3 Branch and Leaf flow

- [ ] Branch re-resolves every assigned Leaf
- [ ] Branch republishes recipient-specific URLs
- [ ] Branch does not run local dataset preparation
- [ ] pre-dispatch Branch failure creates no lower resources and waits for no Leaf callback
- [ ] Leaf prepares local dataset
- [ ] lower Server process attaches same reservation
- [ ] no Root URL is forwarded to Leaf

### 18.4 Outcome and admission

- [ ] every assigned identity appears exactly once in result partition
- [ ] first Leaf failure does not finish collection before all terminal outcomes or deadline
- [ ] complete_required is the only admission mode
- [ ] models + termination records result before failure
- [ ] Root validates plan/publisher/recipient/assigned set
- [ ] admitted snapshot is immutable
- [ ] no round dispatch occurs

### 18.5 Lifecycle

- [ ] failure cleanup cascades upper to lower
- [ ] parent DELETE cancels bound hierarchy work
- [ ] ADMITTED resources remain active
- [ ] registry release follows terminal/cleaning order
- [ ] shutdown wakes all waiters
- [ ] no automatic topology retry/recovery

### 18.6 Verification

- [ ] focused Go and Py tests pass
- [ ] PyMTLF full pytest and Ruff pass
- [ ] NWDAF make test/build/lint pass
- [ ] flat FL regressions pass
- [ ] mandatory review and targeted remediation complete
- [ ] implementation record and repository commits recorded

---

## 19. 完成條件

Slice 4只有在下列條件全部成立後才能標為Completed：

1. Go與Py notification validators符合TS 29.520 V18.14.0 conditional rules；
2. flat與Leaf preparation success都使用mLModelInfos，不依賴statusReport；
3. Branch／Leaf assignment由background worker完整下載、驗證並bind process-scoped role；
4. Branch重新exact-resolve所有assigned Leaves；
5. Branch為每個Leaf發布recipient-specific assignment並建立lower standard resources；
6. Branch upper Client與lower Server processes attach同一plan registry reservation；
7. FL Server stage-aware收集preparation outcomes、duplicates、delay與deadline；
8. Branch result bundle完整表達prepared、failed與timed-out partition；
9. Branch READY／FAILED callback符合本文件contract；
10. Root驗證所有Branch result bundles並執行complete_required admission；
11. success進入ADMITTED並保留Slice 5需要的resources；
12. failure使完整experiment terminal並執行bounded cleanup；
13. 沒有automatic preparation／topology retry、role config、Go plan state或round 0；
14. flat FL、combined capability、backend generation reset與artifact regressions通過；
15. PyMTLF與NWDAF完成focused／full verification；
16. implementation後不中斷完成mandatory review、in-scope remediation與targeted recheck；
17. production changes依repository分開commit，並在本文件記錄實際結果。

完成後下一個production slice是Slice 5：Root–Branch–Leaf hierarchical rounds、FedProx local
objective、per-tier correlation、Branch aggregation與Root aggregation。
