# AGG NWDAF 訂閱轉發：無 AoI Routing 規格分析（§6.1A.3.2）

**規格來源**：TS 23.288 §6.1A、§6.2.2；TS 23.502 §4.15.4.5

---

## 背景與動機

### 目前的部署情境

目前環境中只有一個 NWDAF 實例，5G 網路有兩條流量 path，各由一個 UPF 負責。Consumer 直接對這個唯一的 NWDAF 訂閱兩次——每次針對一個 path 的 group（Internal Group ID）請求 `UE_COMMUNICATION` 分析。AnLF 推論與 MTLF 訓練管線都在這個 NWDAF 內部完成，Daisy FL 也只和這一個 NWDAF 互動。

```
Consumer ──→ NWDAF（唯一實例）
               └── SMF（單一，管理兩個 UPF）
                     ├── UPF-A（path-1）
                     └── UPF-B（path-2）
```

> **注意**：目前只有一個 SMF，這在後續分散式改造時會有 routing 問題，詳見 Step 2。

### 想改造的目標情境

下一階段希望將 NWDAF 以分散式的形式部署於 5GC，讓不同的 NWDAF 實例各自專責特定的流量路徑，並由一個 Aggregator NWDAF（AGG）統一對外服務 consumer 的訂閱請求。**Consumer 不需要感知內部有多少個 NWDAF，只需訂閱一次即可取得完整的 group 分析。**

```
Consumer ──→ NWDAF-AGG（對外門面）
               ├──→ NWDAF-A（專責 path-1 / UPF-A，含 AnLF / MTLF）
               │         └── 向 SMF-A 訂閱
               └──→ NWDAF-B（專責 path-2 / UPF-B，含 AnLF / MTLF）
                         └── 向 SMF-B 訂閱

SMF-A ──→ UPF-A（path-1）
SMF-B ──→ UPF-B（path-2）
```

這個架構同時服務兩個目的：

- **分攤運算資源**：資料蒐集、推論、模型訓練的工作量由各 leaf NWDAF 分擔，而非集中在單一實例。每個 leaf 只需處理自己 path 的資料量，有利於水平擴展。
- **為 FL 合規做準備**：各 leaf NWDAF 在資料層面天然隔離，符合 HFL 中每個 FL client 持有本地資料的前提。AGG 日後可作為 FL server 協調各 leaf 的訓練，同時各 leaf 各自負責監控自己 path 的 model accuracy，當效能下降時觸發 retrain 決策（MTLF）。整體架構在規格精神上對應到 TS 23.288 §6.1A（analytics aggregation）及未來的分散式 model provisioning 機制。

初期（Step 1）只實作訂閱的轉發與通知的 relay；後續再逐步補上各 leaf 的 model accuracy monitoring（效能偵測與 retrain 決策）、跨 NWDAF 的 ML Model Provisioning（TS 23.288 §6.1B 的 context transfer），以及分散式 FL 的協調機制。

### 為什麼需要提前釐清規格

核心動機是為了在分散式部署下讓每個 path 的 NWDAF 各自握有自己 path 的資料，作為後續 FL 訓練的獨立資料節點，同時確保整體架構符合 3GPP 規格的精神。

**分散式資料邊界是 FL 的前提**

HFL（Horizontal FL）訓練要有意義，每個 leaf NWDAF 的訓練資料必須嚴格對應其服務的 path。這個邊界不是事後劃定的，而是在訂閱層面就已決定——leaf NWDAF 訂閱的 target（group ID 或 SUPI 子集）決定了它能看到哪些 SMF/UPF 事件。若訂閱 target 的 cascade 語意不正確，資料邊界就會混淆，FL 訓練也將失去意義。

**Leaf NWDAF 的獨立運作能力**

既然已是分散式環境，每個 path-specialized NWDAF 就應該能夠獨立完成完整的分析閉環：資料蒐集（訂閱 SMF/UPF event）、推論（AnLF）、模型監控（accuracy monitor）、retrain 決策（MTLF），以及計算資源的分配。這些都是 leaf 自己的職責，AGG 不應介入 leaf 的日常運作。

**AGG NWDAF 作為統一入口**

Consumer 不需要感知內部有幾個 NWDAF。AGG 在兩個層面提供統一入口：

- **推論訂閱層面**：Consumer 訂閱一次，AGG 負責將訂閱轉發給對應的 leaf，並將 leaf 的通知 relay 回 consumer
- **FL 協調層面**：AGG 可以作為 FL server，協調各 leaf NWDAF（FL client）進行聯合訓練，Consumer 若需要觸發 retrain 也透過 AGG 進行

**規格合規的必要性**

我們希望在滿足上述情境的同時，也符合 3GPP 規格中其他機制的精神，包括：

- **Model Provisioning**（TS 23.288 §6.1B）：跨 NWDAF 的 analytics context transfer，確保 model 和訂閱狀態可以在 NWDAF 之間正確移交
- **Accuracy Monitoring**：leaf NWDAF 各自回報 accuracy，AGG 需要能夠彙整或 relay 這些資訊
- **Subscription Cascade**：Consumer → AGG → leaf 的訂閱鏈，每一層的 `TargetUeInformation`、`Analytics Filter` 應如何傳遞，哪些欄位需要調整，都需要對應到規格定義

本文件的目的是釐清 AGG 轉發訂閱這個最基礎的動作在規格上應該如何進行，作為後續獨立推論、FL 協調、model provisioning 等功能設計的共同基礎。

---

## AGG 的核心任務

Consumer 送來：

```
POST /nwdaf-eventssubscription/v1/subscriptions
{
  event: UE_COMMUNICATION,
  tgtUe: { intGroupIds: ["group-A"] },
  notificationURI: "http://consumer/cb"
}
```

AGG 要做的事只有一件：**找到能服務 group-A 的 UE_COMMUNICATION 分析的 leaf NWDAF，把訂閱轉發過去，並在收到回應後轉給 Consumer。**

問題在於：AGG 怎麼知道要找哪個 NWDAF？

---

## 前提：UE_COMMUNICATION 用 NF Serving Area 做 Routing

規格的 NWDAF routing 最終都依賴 TAI，但 **TAI 的來源**取決於 Analytics ID 的性質。對於需要 UE 位置資訊的 Analytics（如 UE Mobility），AGG 會先從 AMF/UDM 取得 UE 當前所在的 TAI 再去找 NWDAF（§6.1A.3.2 step 3a）。`UE_COMMUNICATION` 則不同——它不需要知道 UE 在哪，而是改用**負責蒐集資料的 NF（SMF/UPF）的 serving area（TAI）**來找對應的 NWDAF（§6.1A.3.2 step 3b）。

**TS 23.288 §6.1A.3.2 step 3b 原文：**

> "If the data analytics does **not require to collect UE location information**, e.g. for the Analytics IDs **'Service Experience', 'NF load information', or 'UE Communication'**, the Aggregator NWDAF1 can determine the **NFs to be contacted for data collection** as specified in clause 6.2.2.1 and then it can retrieve **NF service area** for each of the data source NFs from NRF."

`UE_COMMUNICATION` 的 routing key 是**負責收集資料的 NF（SMF/UPF）的 serving area**，不是 UE 的地理位置。

---

## 推導 Leaf NWDAF：Group ID → TAI → NWDAF

目標是找到「服務範圍（serving area）涵蓋 group-A 資料來源的那個 NWDAF」。規格用來表示服務範圍的單位是 **TAI（Tracking Area Identity）**——5G 網路將地理區域劃分成若干 Tracking Area，每個 TA 以一個 TAI 識別。NWDAF 在 NRF 的 NF Profile 中會登錄自己負責哪些 TAI，AGG 透過 TAI 過濾就能找到服務對應區域的 leaf NWDAF。

AGG 拿到 `group-A` 後，規格給出的標準推導鏈是：

```
group-A
  │
  ▼ [NRF 找 UDM，Nudm_SDM_Get]                    ← TS 23.288 §6.2.2.1
  SUPI list: [supi-1, supi-2, supi-3, ...]
  │
  ▼ [Nudm_UECM_Get，每個 SUPI 找 SMF]              ← TS 23.502 §4.15.4.5.2
  SMF list
  │
  ▼ [Nnrf_NFManagement，查 SMF NF Profile 取 TAI]  ← TS 29.510 §6.1.6.2.3
  TAI list（SMF 的服務範圍）
  │
  ▼ [Nnrf_NFDiscovery，用 TAI 過濾找 leaf NWDAF]   ← TS 23.288 §6.1A.3.2 step 4
  leaf NWDAF（serving area 涵蓋該 TAI 的 NWDAF）
```

**TS 23.288 §6.1A.3.2 step 4 原文：**

> "With the data obtained in step 3, the Aggregator NWDAF1 queries the NRF for discovering the required NWDAF, by sending an NF discovery request which may include **UE location (e.g. TAI-1) or NF serving area (e.g. TAI list-1)** as a filter to NRF."

換句話說：AGG 不是拿著 group ID 去問 NRF「誰能服務這個 group」，而是先把 group 翻譯成 SMF，再把 SMF 的 TAI 拿去找 serving area 涵蓋該 TAI 的 NWDAF。**Group ID 本身不是 NRF 的 NWDAF 發現維度。**

### Single-SMF 拓撲的 Routing Gap

若部署環境中只有一個 SMF 管理兩個 UPF，上述推導鏈在 SMF 這一層就喪失了區分能力：

```
group-A → SUPI list → 全部指向同一個 SMF → 同一個 TAI → 無法區分兩條 path
group-B → SUPI list → 全部指向同一個 SMF → 同一個 TAI → 無法區分兩條 path
```

規格的 TAI-based routing 假設不同 path 有不同的 SMF serving area，這個前提在 Single-SMF 拓撲下不成立。**規格沒有針對此情境提供 routing 解法。**

### 正確解法：兩個 SMF

讓每條 path 各有一個 SMF（可依 DNN 或 S-NSSAI 區分），routing chain 就能恢復完整：

```
group-A → SUPI list → SMF-A (TAI-A) → NRF → NWDAF-A
group-B → SUPI list → SMF-B (TAI-B) → NRF → NWDAF-B
```

這不只解決 AGG routing 問題，leaf NWDAF 的資料蒐集邊界也自然由 SMF 劃定——NWDAF-A 只訂閱 SMF-A，天然只收到 UPF-A 的資料，不需要任何額外 filter。**這是與 TS 23.288 §6.1A.3.2 step 3b 完全對齊的部署方式。**

> **Note：No-NRF / No-UDM 的簡化做法**
> 若 PoC 環境中沒有 NRF/UDM，兩個 SMF 確立後，推導鏈可以用 config 靜態對應替代：
> `group → known SMF → known NWDAF`
> 語意完全等價於規格的動態查詢路徑。

> **⚠️ 待討論：兩個 SMF 的合理性**
> 兩個 SMF 是規格 TAI-based routing 的合規解法，但實務上有疑慮：5G 架構中 SMF 本來就設計成可以管理多個 UPF、涵蓋大範圍服務區域，強制每個 UPF 各配一個 SMF 有點違反這個設計初衷。
> 更根本的問題是：規格的 TAI routing 是針對**地理上的服務區域區分**設計的，但我們的兩條 path 是**邏輯上的區分**（同一地理位置，以 DNN 或 S-NSSAI 區分），兩者並不在同一個維度。硬套 TAI routing 只是因為規格沒有提供其他路徑，而非這個設計本身合理。
> 這個部署選擇是否真的適合我們的情境，需要進一步討論。

---

> **另一種思路**：若改由 consumer 主動帶 AoI，AGG 走的是完全不同的規格程序（TS 23.288 §6.1A.3.1），不再需要 NF 查詢推導鏈。詳見 [agg_routing_with_aoi.md](agg_routing_with_aoi.md)。

---

## 轉發目標數量：單一 NWDAF vs 多個 NWDAF

推導結果會有兩種情形：

### 情形 A：group-A 的所有 SUPI 都在同一個 NWDAF 的服務範圍

**→ TS 23.288 §6.1A.3.2 step 5a：single-target redirect**

> "If a single target NWDAF (e.g. NWDAF2) can provide the requested analytics data, the Aggregator NWDAF (e.g. NWDAF1) can **redirect** the `Nnwdaf_AnalyticsInfo_Request` to that target NWDAF or request an analytics subscription transfer to that target NWDAF."

AGG 把訂閱轉發給 NWDAF-A，收到通知後直接轉給 Consumer。這個情形下 AGG 退化成 proxy，不需要做真正的聚合。

### 情形 B：group-A 的 SUPI 分散在不同 NWDAF 的服務範圍

**→ TS 23.288 §6.1A.3.2 step 5b：進入完整聚合流程（TS 23.288 §6.1A.3.1 steps 4-9）**

AGG 同時向多個 leaf NWDAF 發出訂閱，各 leaf 各自回傳自己服務的那批 SUPI 的分析結果，AGG 把這些結果合成一份完整的 group-A 分析，再通知 Consumer。

這是 Aggregator 模式真正發揮作用的情境：**Consumer 拿到的是整個 group 的統一分析視圖，而 SUPI 分散的細節對 Consumer 透明。**

---

## 轉發訂閱的 Target 設計

AGG 向 leaf NWDAF 發出的訂閱，`tgtUe` 應該帶什麼？規格沒有明文規定，但有兩種合理做法：

**做法 A：帶原始 group ID**

```json
{ "tgtUe": { "intGroupIds": ["group-A"] } }
```

Leaf NWDAF 收到後自己解析 group-A → SUPI list → 找 SMF → 只對自己能建立訂閱的 SMF 建立訂閱，找不到的自然跳過。規格沒有禁止，leaf 行為上是 graceful degradation。

**做法 B：帶 SUPI 子集（推薦）**

```json
NWDAF-A: { "tgtUe": { "supis": ["supi-1", "supi-2"] } }
NWDAF-B: { "tgtUe": { "supis": ["supi-3", "supi-4"] } }
```

AGG 在推導 Leaf NWDAF 時已解析出哪些 SUPI 屬於哪個 NWDAF，直接分配子集。每個 leaf 只收到自己負責的 SUPI，不需要自行做過濾。這更接近 TS 23.288 §6.1A.2 中 AoI 被切分成子 AoI 的精神。

`TargetUeInformation` schema 同時支援 `supis` 和 `intGroupIds`，兩種做法在 API 層面都合法。

---

## Leaf NWDAF 的資料蒐集

Leaf NWDAF 收到訂閱（無論是 group ID 或 SUPI list）後，按 **TS 23.502 §4.15.4.5.2** 建立 UPF 資料蒐集管線：

> "In the case of a group of UEs, the UPF event consumer (e.g. NWDAF) first issues an `Nnrf_NFDiscovery_Request` to find the UDM... Then, NWDAF obtains the list of SUPIs that correspond to the Group ID from UDM using `Nudm_SDM_Get`.
> Then, **for each SUPI**:
> 1. invoke `Nudm_UECM_Get` to retrieve the appropriate SMF...
> 3. The UPF event consumer sends the `Nsmf_EventExposure Subscription` request to the SMF..."

若收到的是 SUPI list（做法 B），leaf 可跳過 group 解析，直接進行 per-SUPI SMF 訂閱。

> **Note：Group ID 與 AMF/SMF 直接訂閱**
> TS 23.288 §6.2.2.1 說明 NWDAF 可以「直接以 group ID 為 target 訂閱所有 AMF/SMF 實例」，這條路適用於 AMF/SMF 事件的一般訂閱，**不是** via SMF 訂閱 UPF 事件的路徑。via SMF 訂閱 UPF 仍必須 per-SUPI（TS 23.502 §4.15.4.5.2）。

---

## 通知 Relay 與聚合

AGG 向各 leaf 訂閱時將自己的 callback endpoint 填入 `notificationURI`，並記錄 leaf sub ID → consumer sub ID 的對應關係。

- **情形 A（single-target）**：leaf 通知 AGG，AGG 直接轉給 Consumer。
- **情形 B（multi-target）**：每個 leaf 回傳的 `EventNotification` 包含 `anaMetaInfo`（numSamples、dataWindow、accuracy 等），AGG 依此合併成完整 group 分析再通知 Consumer。

`anaMetaInfo` 欄位（定義於 TS 23.288 §6.1.3）的存在目的正是讓 AGG 能夠做**有意義的合併**，而不是盲目相加。

---

## 完整流程概覽

```
Consumer
  │  POST /subscriptions
  │  { event: UE_COMMUNICATION, tgtUe: {intGroupIds:[group-A]} }
  ▼
NWDAF-AGG
  │
  ├─[NWDAF 發現] group-A → SUPI list → SMF list → TAI → leaf NWDAF
  │              （NRF/UDM 查詢，或 config 靜態對應）
  │
  ├─[轉發判斷] 情形 A 或 B
  │
  │ ┌─[情形 A: single NWDAF]──────────────────────────────────────┐
  │ │  POST /subscriptions → NWDAF-A                             │
  │ │  { tgtUe: {supis:[supi-1,supi-2]}, notifUri: agg-cb }      │
  │ │        ↓                                                   │
  │ │  NWDAF-A → POST agg-cb { EventNotification }               │
  │ │        ↓                                                   │
  │ │  AGG → POST consumer-cb（直接轉發）                          │
  │ └────────────────────────────────────────────────────────────┘
  │
  │ ┌─[情形 B: multi NWDAF]──────────────────────────────────────┐
  │ │  POST /subscriptions → NWDAF-A { supis:[1,2] }            │
  │ │  POST /subscriptions → NWDAF-B { supis:[3,4] }            │
  │ │        ↓ ↓ (各自回傳)                                      │
  │ │  AGG 合併 anaMetaInfo + 聚合分析結果                         │
  │ │        ↓                                                  │
  │ │  AGG → POST consumer-cb（聚合後結果）                        │
  │ └───────────────────────────────────────────────────────────┘
  ▼
Consumer
```

---

## 規格原始程序對照（TS 23.288 §6.1A.3.2）

以下為規格 §6.1A.3.2「Analytics Aggregation without Provision of Area of Interest」的完整 step 順序，以及各 step 對應本文件哪個分析段落。

**TS 23.288 §6.1A.3.2 — Aggregation without Area of Interest**

| Step | 規格描述摘要 | 本文對應段落 |
|------|-------------|-------------|
| 1 | Consumer 做 NWDAF discovery，選擇一個具備 aggregation capability 的 NWDAF（AGG） | — （Consumer 側行為，本文不展開） |
| 2 | Consumer 向 AGG 送出 analytics request / subscription，含 Analytics ID 等參數 | AGG 的核心任務 |
| 3a | 若該 Analytics ID 需要 UE 位置資訊（如 UE Mobility、Abnormal Behaviour），AGG 透過 UDM 或 AMF 取得 UE 位置 | — （`UE_COMMUNICATION` 不走此路徑） |
| 3b | 若該 Analytics ID **不需要** UE 位置資訊（如 **UE Communication**、Service Experience、NF load），AGG 改由 §6.2.2.1 決定要聯繫的 NF（SMF/UPF），再從 NRF 取得這些 NF 的 service area | 前提：UE_COMMUNICATION 用 NF Serving Area 做 Routing |
| 4 | AGG 用 step 3 取得的 UE 位置（TAI）或 NF serving area（TAI list）向 NRF 做 NWDAF discovery | 推導 Leaf NWDAF：Group ID → TAI → NWDAF |
| 5a | 若 NRF 只回傳一個目標 NWDAF，AGG 可以直接 redirect 訂閱給該 NWDAF | 轉發目標數量：情形 A |
| 5b | 若需要多個 NWDAF，進入 §6.1A.3.1 steps 4–9 的完整聚合流程（見下表） | 轉發目標數量：情形 B |

**TS 23.288 §6.1A.3.1 steps 4–9 — 多目標聚合子流程**（由 §6.1A.3.2 step 5b 觸發）

| Step | 規格描述摘要 | 本文對應段落 |
|------|-------------|-------------|
| 4–5 | AGG 向各目標 NWDAF 發出訂閱，可附帶 analytics metadata request；各 NWDAF 各自對應子範圍 | 轉發訂閱的 Target 設計 |
| 6–7 | 各目標 NWDAF 回傳分析結果（含 `anaMetaInfo`） | Leaf NWDAF 的資料蒐集 |
| 8 | AGG 將各 NWDAF 的結果聚合為單一輸出 | 通知 Relay 與聚合 |
| 9 | AGG 將聚合結果通知 Consumer | 通知 Relay 與聚合 |

---

## 規格章節對應速查

| 流程步驟 | 規格章節 |
|----------|---------|
| `UE_COMMUNICATION` 不需 AoI，改用 NF serving area 路由 | TS 23.288 §6.1A.3.2 step 3b |
| group ID → SUPI → SMF → TAI 推導鏈 | TS 23.288 §6.2.2.1；TS 23.502 §4.15.4.5.2 |
| TAI → leaf NWDAF NRF 查詢 | TS 23.288 §6.1A.3.2 step 4 |
| Single-target redirect（情形 A） | TS 23.288 §6.1A.3.2 step 5a |
| Multi-target 聚合（情形 B） | TS 23.288 §6.1A.3.2 step 5b → §6.1A.3.1 steps 4-9 |
| Leaf per-SUPI SMF 訂閱（via SMF 到 UPF） | TS 23.502 §4.15.4.5.2 |
| Single-SMF routing gap → 需要兩個 SMF | TS 23.288 §6.1A.3.2 step 3b 隱含前提（各 path 有各自 SMF） |
| `anaMetaInfo` 欄位（聚合用 metadata） | TS 23.288 §6.1.3；TS 23.288 §6.1A.3.1 step 4-5 |
