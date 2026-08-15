# Hierarchical NWDAF Federated Learning 文件索引

## 1. 文件目的

本目錄整理以 **Hierarchical NWDAF Federated Learning** 為主軸的情境證據、
設計資料與proposal初稿。現階段先收斂完整scenario、component behavior、spec
evidence、implementation gap與Daisy reuse。

目前主線是：

> 在維持 3GPP Release 18 NRF discovery 與
> standard SBI boundary 的前提下，以 Analytics Aggregator 建立
> Global–Regional–Local operational hierarchy，再以多個標準 NWDAF FL
> processes 組成 hierarchical FL，並以 strategy-independent engine 支援跨層
> participant、wait 與 aggregation policy。

## 2. 文件結構

| 文件 | 內容 |
| --- | --- |
| [Proposal初稿](proposal_draft.md) | 先讀的主文件；整合完整故事、標準依據、架構、component gap、assumptions與待決選項 |
| [情境、元件與落差分析](scenario_components_and_gap_analysis.md) | 支援文件；展開5GC component responsibilities、procedure traces、3GPP／free5GC／現有實作比較，以及assumption relaxation |
| [Daisy 重用與階層式設計](daisy_reuse_and_hierarchical_design.md) | 支援文件；從high level說明Daisy現有能力、可重用邊界、Global／Regional／Local composition與PyMTLF gap |

建議閱讀順序為：

```text
proposal_draft.md
  -> scenario_components_and_gap_analysis.md
  -> daisy_reuse_and_hierarchical_design.md
```

## 3. 狀態分類

各文件使用以下分類，避免把已完成能力、標準行為與候選設計混在一起：

| 分類 | 定義 |
| --- | --- |
| **Implemented baseline** | 已由現有 source、tests 或 full-core E2E 證明 |
| **3GPP-defined** | Release 18 明確定義的角色、procedure 或 service contract |
| **Standards-compatible design inference** | 可由標準角色與 service 組成，但 3GPP 沒有定義完整 procedure |
| **Candidate extension** | 尚未完成、可在核心情境後加入的實作方向 |
| **Assumption** | 為界定目前scenario而固定的條件 |
| **Out of scope** | 已知可延伸，但不納入目前scenario主線 |

## 4. 已確認的基線

- 現有 flat distributed NWDAF FL full-core E2E 已完成驗證。
- NWDAF-C 是 FL Server；NWDAF-A／B 是 FL Clients。
- 目前使用 synchronous、sample-count-weighted FedAvg。
- participant set 在 preparation 後固定，round 需要全部 selected clients 回覆。
- WAPE degradation、final validation、ADRF publication、model reprovision 與
  monitor cutover 已包含在 E2E 中。
- UE registration、PDU Session、UDM／UDR group resolution、serving-SMF
  resolution、SMF AoI gate 與 UPF Event Exposure 走 full-core procedure。

主要驗證紀錄：

- [Phase 7 Full-Core Federated Learning Business E2E Detailed Plan](../../../plans/federated-learning/Phase%207%20Full-Core%20Federated%20Learning%20Business%20E2E%20Detailed%20Plan.md)
- [Distributed NWDAF Model Monitoring and Federated Retraining Architecture](../../../design/Distributed%20NWDAF%20Model%20Monitoring%20And%20Federated%20Retraining%20Architecture.md)

## 5. 已確認的設計邊界

- Hierarchical NWDAF FL 是主要proposal方向。
- Regional 是覆蓋兩個 TAI 的 Analytics Aggregator，不是只在 FL 時出現的
  transparent proxy。
- Model Provision 採 Global→Regional→Local；Monitor 採 Local→Regional→Global。
- 每層FL Server以active monitor owners作為候選來源，再經NRF validation與
  preparation建立participant inventory；Global因此選到Regional，Regional則選到Local。
- Regional 先以 `max(child deviations)` 回報一份 worst-scope deviation；此值不
  稱為 regional WAPE，且 cross-NWDAF monitor 只使用標準欄位。
- Regional 在 FL 中是 pure model aggregator，不加入自己的 dataset contribution。
- Validation 經 Global→Regional→Local 傳遞並逐層聚合，Global作final decision。
- Strategy-independent architecture 是 hierarchy 的核心 enabling design，
  不是另一個互不相關的題目。
- Daisy 是主要 reuse reference。
- Daisy 的 FL engine concepts 可重用，但 Daisy gRPC topology、private discovery
  與 `parent_address` 不取代 3GPP SBI。
- Mobility、dynamic group membership、SSC Mode 2／3、I-SMF 與 roaming 不作為
  hierarchical FL 的必要前提。
- Mobility只保留assumption-relaxation、實際handover流程與改動成本分析。

## 6. 尚待收斂的scenario選項

目前先收斂具體story所需的部署與決策資訊：

1. 參考scenario先採8個UE、4個gNB、4個TAIs、1個AMF、1個SMF、2個UPFs、
   4個Locals、2個Regionals與1個Global；此數量可再scale out。
2. 完整故事由外部AF透過NEF要求Group G的`UE_COMMUNICATION` predictions，
   以分區安排非即時firmware／batch delivery；實驗從Consumer/Nnwdaf邊界開始。
3. Consumer選擇恰好涵蓋requested AoI的Regional Aggregator；Local依model scope
   與相容條件選擇provider，不以Global／Regional／Local標籤做discovery。
4. stable-to-degraded stimulus發生在哪個scope，以及monitor如何沿hierarchy觸發
   retraining。
5. 參考部署如何表達為可增加UE、TAI、Local與Regional數量的scale-out pattern。
6. Strategy第二選項、inner／outer rounds與validation metrics留到scenario成立後
   再討論。

## 7. 證據判讀順序

若不同文件的描述不一致，依下列順序判讀：

1. 現有 implementation 與 tests。
2. Phase 7 verified result 與已完成的 active plans。
3. 本地 Release 18 OpenAPI 與 TS corpus。
4. 現有 architecture／notes。
5. 本地 upstream mirrors；mirror revision 不等於目前 upstream HEAD。

本目錄的證據盤點日期為 **2026-08-10**。
