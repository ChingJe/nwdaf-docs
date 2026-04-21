# 分散式 NWDAF 部署：概覽摘要

> 詳細規格分析：
> - 無 AoI 方案（AGG 自行推導 NF serving area）：[agg_routing_without_aoi.md](agg_routing_without_aoi.md)
> - 有 AoI 方案（Consumer 帶 TAI，AGG 直接查 NRF）：[agg_routing_with_aoi.md](agg_routing_with_aoi.md)

---

## 目前的情況

環境裡只有一個 NWDAF。5G 網路有兩條流量 path，各自由一個 UPF 負責，共用同一個 SMF。Consumer 訂閱兩次，每次訂閱一條 path 的 UE 群組（用 Internal Group ID 區分）的 `UE_COMMUNICATION` 分析。推論、模型訓練、資料蒐集全都在這一個 NWDAF 裡面完成。

問題是：所有東西都壓在同一個實例上，運算資源集中，而且沒辦法為未來的分散式 FL 訓練做準備。

---

## 想改成怎樣

把 NWDAF 拆成三個實例：

- **NWDAF-AGG**（Aggregator，負責聚合與轉發的 NWDAF）：對外的統一入口。Consumer 只跟它說話，不需要知道後面有幾個 NWDAF。
- **NWDAF-A**：專門負責 path-1 的資料蒐集、推論、模型訓練。
- **NWDAF-B**：專門負責 path-2 的資料蒐集、推論、模型訓練。

Consumer 訂閱一次，AGG 自動把訂閱轉發給對應的 leaf NWDAF，leaf 做完推論後把結果回傳給 AGG，AGG 再轉給 Consumer。Consumer 感受不到背後有多個 NWDAF 存在。

---

## 為什麼要這樣做

主要有兩個原因：

**1. 分攤運算資源**
資料蒐集、推論、模型訓練的工作由各 leaf NWDAF 分擔，不再集中在一個地方。隨著 path 數量增加，可以水平擴展，加 NWDAF 就好。

**2. 為 FL 訓練做準備**
未來要做 Federated Learning 時，每個 leaf NWDAF 手上要有自己 path 的資料，這樣才能作為獨立的 FL client 參與訓練。AGG 則可以作為 FL server 協調整個訓練過程。各 leaf 也各自負責監控自己 path 的模型效能，當效能下降時自己決定要不要 retrain——這些都符合 3GPP 規格中 Model Provisioning 和 Accuracy Monitoring 的精神。

---

## 大致上怎麼運作

整體流程不論哪種方案都一樣：Consumer 訂閱一次，AGG 找到對應的 leaf NWDAF 轉發，leaf 收集資料做推論後回傳，AGG 再轉給 Consumer。差別在於 AGG **怎麼找到對應的 leaf NWDAF**，這取決於部署拓撲和 Consumer 怎麼訂閱（詳見下節）。

以 AoI（Area of Interest，感興趣的地理範圍）方案為例（Consumer 帶 TAI-A，TAI 即 Tracking Area Identity，5G 的地理服務範圍單位）：

1. Consumer 跟 AGG 說：「我要訂閱 group-A 在 TAI-A 區域的 UE 通訊分析」
2. AGG 直接用 TAI-A 去查 NRF，找到服務 TAI-A 的 NWDAF-A
3. AGG 把訂閱轉發給 NWDAF-A
4. NWDAF-A 先向 AMF 取得 TAI-A 範圍內的 UE 清單，與 Consumer 訂閱的 group-A SUPI 清單取交集，得出最終目標 UE 清單，再 per-UE 向 SMF 訂閱 UPF 事件，收集資料，跑推論，算出結果
5. NWDAF-A 把結果通知給 AGG
6. AGG 把結果轉給 Consumer

以無 AoI 方案為例（Consumer 只給 group ID）：

1. Consumer 跟 AGG 說：「我要訂閱 group-A 的 UE 通訊分析」
2. AGG 把 group-A 解析成 SUPI 列表，找到服務這些 UE 的 SMF
3. 從 SMF 的服務範圍（TAI）推算出應該找哪個 leaf NWDAF
4. NWDAF-A 向 UDM 解析 group-A 取得 SUPI 清單，per-SUPI 向 SMF 訂閱 UPF 事件，收集資料，跑推論，算出結果
5–6. 後續同上

如果 UE 分散在兩個 leaf NWDAF 的服務範圍，AGG 就同時轉發給兩個 leaf，等兩個都回來再合併結果，一起通知 Consumer。

---

## 拓撲與 Routing 方案選擇

兩個關鍵概念：
- **TAI**（Tracking Area Identity）：5G 用來識別地理服務範圍的單位，每個 Tracking Area 對應一塊地理區域。NWDAF 在 NRF 登錄自己負責哪些 TAI，Consumer 或 AGG 用 TAI 來找對應的 NWDAF。
- **AoI**（Area of Interest）：Consumer 訂閱分析時可以附帶的地理範圍參數，以 TAI 表示，告訴 NWDAF「我只想要這個區域的分析結果」。

AGG 要找到對應的 leaf NWDAF，核心依據就是 TAI。兩條 path 能不能被區分，取決於它們是否對應到不同的 TAI。目前有兩種方向：

---

### 方案一：兩個 SMF，各自管一個 UPF（無 AoI）

讓每條 path 各有一個 SMF（以 DNN 或 S-NSSAI 區分），各自有獨立的服務區域：

- SMF-A 管 UPF-A（TAI-A）→ AGG routing 到 NWDAF-A
- SMF-B 管 UPF-B（TAI-B）→ AGG routing 到 NWDAF-B

Consumer 只給 group ID，AGG 自行推導：group → SUPI → SMF → TAI → NWDAF，不需要 Consumer 知道任何 TAI 資訊。資料邊界也自然由 SMF 劃定，NWDAF-A 只訂閱 SMF-A，天然只收到 UPF-A 的資料。

詳細規格流程見 [agg_routing_without_aoi.md](agg_routing_without_aoi.md)。

> **⚠️ 待討論**：5G 本來就設計成一個 SMF 可以管理很多個 UPF，強制一個 SMF 只管一個 UPF 感覺違反設計初衷。更根本的問題是規格的 TAI routing 是針對地理上的區分設計的，而我們的兩條 path 是邏輯上的區分（同一個地方，不同的 DNN 或 S-NSSAI），這兩個維度本來就對不上。

---

### 方案二：單一 SMF，兩個 UPF 在不同 TAI（有 AoI）

保留一個 SMF，但讓兩個 UPF 分別部署在不同的地理區域（UPF-A 在 TAI-A，UPF-B 在 TAI-B）。單一 SMF 管理兩個 UPF、service area 跨兩個 TAI，是完全正常的多區域覆蓋部署。

Consumer 訂閱時帶上 AoI（指定 TAI-A 或 TAI-B），AGG 直接用 AoI 查 NRF 找到對應的 leaf NWDAF，不需要任何 NF 推導鏈，流程更簡潔。

詳細規格流程見 [agg_routing_with_aoi.md](agg_routing_with_aoi.md)。

> **待討論**：Consumer 需要事先知道各 path 對應的 TAI，對 Consumer 的使用門檻較高。不過若 Consumer 本來就清楚自己想監控哪個地理區域，由它帶 AoI 反而是合理的——Consumer 知道自己關心什麼，不需要 AGG 被動推導。

---

### 兩種方案比較

| | 方案一（兩個 SMF） | 方案二（單一 SMF + AoI） |
|---|---|---|
| SMF 數量 | 2 | 1 |
| Consumer 需提供 | group ID | group ID + AoI（TAI） |
| AGG routing 邏輯 | 推導鏈（group → SUPI → SMF → TAI） | 直接用 AoI 查 NRF |
| 拓撲合理性 | 有疑慮（一 SMF 只管一 UPF） | 自然（SMF 跨多 TAI 是正常部署） |
| Leaf 資料蒐集機制 | per-SUPI SMF 訂閱（§4.15.4.5.2） | AMF 取 UE list → 與 group 取交集 → per-UE SMF 訂閱（§4.15.4.5.4） |
| 規格對應程序 | TS 23.288 §6.1A.3.2 | TS 23.288 §6.1A.3.1 |
