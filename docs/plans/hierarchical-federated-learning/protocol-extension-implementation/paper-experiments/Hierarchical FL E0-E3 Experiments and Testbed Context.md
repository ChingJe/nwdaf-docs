# Hierarchical FL E0–E3 實驗情境與 Testbed 對照

狀態：文件內容已確認；E0–E3 的五 seed 結果尚未執行或確認。

本文整合既有論文第四章草稿的 testbed 對照，以及後續提出的 E0–E3 實驗要求。前者是**已執行的單次配對觀測**，後者是**尚待執行的多 seed 實驗情境**；兩者不可混成同一批結果。本文描述研究問題、條件、預期觀測與證據，不規劃 PyMTLF／NWDAF／testbed 如何實作，也不改寫當時的 scenario 或 run metadata。

## 1. 既有論文草稿選用的配對

既有第四章草稿比較兩種工作負載：MNIST 與 CIFAR-10；每種各有無故障 baseline，以及 Area A Intermediate 故障並由預部署替代節點接手的 treatment。兩組資料都採相同拓樸與各自配對內相同的資料切分、初始模型和訓練設定。每個條件只有一次有效執行；這些舊結果不能當作後續五個 seeds 的結果，也不能據此宣稱統計顯著或任意拓樸下的恢復保證。

| 論文工作負載 | 實際 scenario 配對（`experiments/protocol-hierarchical/` 以下） | 有效 run 配對 | 當時的 `kind` |
| --- | --- | --- | --- |
| MNIST，五類／Leaf | `mnist/formal-baseline.yaml`、`mnist/formal-replacement.yaml` | `mnist-formal-baseline-20260913-a`、`mnist-formal-replacement-20260913-b` | `formal-comparison` |
| CIFAR-10，每個 Leaf 均有十類但比例不同 | `cifar10/all-class-skew-baseline.yaml`、`cifar10/all-class-skew-replacement.yaml` | `cifar10-skew-baseline-20260914-a`、`cifar10-skew-replacement-20260914-a` | `diagnostic` |

CIFAR-10 這組執行時標為 `diagnostic`，是**歷史執行標記**；目前論文草稿已選它作為 CIFAR-10 的主要配對。這項定位變更不表示原始 metadata 應被重寫。另有 `cifar10/formal-baseline.yaml`／`formal-replacement.yaml` 的舊五類／Leaf 配對，其 final test accuracy 為 38.62%／38.74%；它**不是**目前論文表格中 58.06%／58.12% 的 CIFAR-10 配對。

## 2. 既有論文術語、角色與實際部署

論文的 *Intermediate node* 對應部署中的 `branch`，不是另一種 NF type；*MTLF Python backend* 對應每個 NWDAF 的 Host-side PyMTLF container，而跨 NWDAF 的控制訊息由 Guest 中的 NWDAF 透過 SBI 傳遞。

| 論文名稱／術語 | Testbed 名稱與位置 | 實際角色 |
| --- | --- | --- |
| Root | `nwdaf-root`，`core` VM；`pymtlf-root` 在 Host | 上層 FL server；接收 Branch 更新並執行 Root aggregation、validation |
| Intermediate A | `nwdaf-branch-a-primary`，`path-a` VM；對應 `pymtlf-branch-a-primary` | Area A 原本的 Branch，管理 A1／A2；故障注入目標 |
| Intermediate A* | `nwdaf-branch-a-replacement`，同在 `path-a`；對應 `pymtlf-branch-a-replacement` | 預部署、預註冊的候選 Branch；A 故障後接手 A1／A2 |
| Intermediate B／C | `nwdaf-branch-b`／`nwdaf-branch-c`，分別在 `path-b`／`path-c` | 兩個未受故障注入的 Branch；degraded rounds 仍可貢獻 |
| Leaves A1／A2、B1／B2、C1／C2 | 各 Area 對應的 `nwdaf-leaf-*` 與 `pymtlf-leaf-*`，分布於 `path-a`／`path-b`／`path-c` | 六個本地訓練參與者；替換 A 時 A1／A2 身分與資料不變 |
| Supporting services | MongoDB、NRF、ADRF 位於 `core` VM | 支援發現、資料與模型流程；不是額外的訓練參與者 |

Host 為 Ubuntu 20.04、Intel Core i9-12900K、64 GiB 記憶體及單張 NVIDIA RTX 3080 10 GiB GPU；四台 Vagrant／VirtualBox Guest 為 Ubuntu 22.04。這些 Host 規格來自實驗資料整理中的 2026-09-14 inventory，**不是**每個 run 自身的硬體量測。部署共 1 Root、3 個正常使用的 Branch、6 Leaves、1 個替代 Branch，合計 11 組 NWDAF／PyMTLF。Root 與 Leaves 的後端共用 Host GPU，四個 Branch 的後端使用 CPU；「Host-side」不表示各節點有獨立 GPU。故障注入停止的是 A 的 NWDAF service 與配對後端，**不是**停止 `path-a` VM，因此 A* 與 A1／A2 仍可用。

Testbed 的 `protocolTopology` 把 Area A 的 A 設為 priority 100、A* 設為 50；A* 雖已部署與註冊，baseline 不會取代健康的 A。Root policy 的 `min_available_nodes=2`、`min_train_nodes=2`、`min_completion_rate=0.66`、`accept_failures=true`，讓三個 Branch 中有兩個成功時仍可接受 round。各 Area Branch 的 `min_available_nodes=2`、`min_train_nodes=2`、`min_completion_rate=1.0`、`accept_failures=false`，要求自己的兩個 Leaves。Root 與 Branch 均設 `fedProx`、`proximal_mu=0.01`、`sampleWeighted`；每次 Branch 聚合後即上報 Root，沒有額外的多輪 Branch-local aggregation 才上報設定。`roundTimeoutSeconds` 為 300 秒。

## 3. 既有論文資料與訓練設定如何落到 scenario

兩種工作負載都是六個 Leaves 各有 8,000 筆訓練資料，總數 48,000。Root validation 固定使用從官方 training set 留出的 2,000 筆、每類 200 筆，與 Leaf training samples 不重疊；final test 則使用獨立的官方 10,000 筆 test set。Root 在初始模型與每個 accepted round 後量測 validation accuracy／cross-entropy loss；final test 只在模型完成後另做一次。影像轉為 float32 並除以 255。兩者使用同一類 SmallCNN：32-channel 與 64-channel 的兩層 3×3 convolution、ReLU、第一層後的 2×2 max pooling、adaptive average pooling 到 1×1，最後輸出十類。MNIST／CIFAR-10 的輸入 channel 與影像尺寸不同。

| 項目 | MNIST | CIFAR-10（論文選用的 all-class-skew） |
| --- | --- | --- |
| 每個 Leaf 的類別數量 | A1／A2／B1：0–4 各類 1,600；B2／C1／C2：5–9 各類 1,600 | A1／A2：0–4 各 1,300、5–9 各 300；B1／B2：十類各 800；C1／C2：0–4 各 300、5–9 各 1,300 |
| `partition.datasetId` | `mnist-formal` | `cifar10-all-class-skew` |
| Accepted Root rounds | 24 | 40 |
| 每輪 Leaf local epochs | 4 | 5 |
| 故障觸發點 | 第 12 個 accepted Root round 後 | 第 20 個 accepted Root round 後 |
| 初始模型 | 同配對共用 `seedModelId=1001` 的 artifact | 同配對共用 `seedModelId=1002` 的 artifact |

兩組均設 batch size 16、learning rate 0.001、partition／training seed 42，使用 Adam、cross-entropy 與 FedProx。上述 sample counts、類別配額、初始 artifact 與訓練參數均可在對應 scenario 與 testbed 實驗整理中查到；執行當時的最終配置仍以各 run 的 `run.json.scenario` snapshot 為準。

## 4. 既有故障接手觀測與論文圖表用語

1. 無故障 baseline：A、B、C 全程作為 Root 的正常貢獻者；A* 已部署但不接手。
2. Treatment 前段：A、B、C 正常訓練。Controller 在預定的 accepted Root round 後，先對 A 的 Guest NWDAF 與 Host-side PyMTLF 發 `SIGSTOP`，再終止目標服務；A*、同台 VM 與 A1／A2 不受停止操作影響。
3. Root 等不到 A 的回覆並在約 300 秒的 round timeout 後辨識故障。B、C 的結果仍滿足 Root policy，因此可產生 **degraded accepted round**；實際兩組各觀察到兩輪，這不是事先保證的固定數量。
4. Root 選用次順位 A*；A* 與原 A1／A2 建立新的下層 subscriptions，取得**後續**訓練結果並上報 Root。現行情境沒有使用 retained-result lookup，也不宣稱取回 A 故障前 Leaf 手上的未送達結果。
5. **Replacement ready** 表示替代關係已建立；**first accepted A* contribution** 則是 Root 首次接受含 A* 成功貢獻的 outcome，兩者不可混用。MNIST 的首次 accepted A* contribution 是第 15 輪，CIFAR-10 是第 23 輪。

| 論文圖表／敘述 | MNIST treatment | CIFAR-10 treatment |
| --- | --- | --- |
| Normal accepted rounds | 1–12（A、B、C） | 1–20（A、B、C） |
| Degraded accepted rounds | 13–14（B、C） | 21–22（B、C） |
| Restored participation | 15–24（A*、B、C） | 23–40（A*、B、C） |
| Final official test accuracy：baseline／treatment | 88.84%／88.99% | 58.06%／58.12% |

論文中的 accepted Root round 從 **1** 起算；`roundInd` 在底層記錄中從 **0** 起算；圖上的 round 0 是初始模型 validation，並非一次 accepted aggregation。`mlCorreId` 用於同一 run 中的 procedure correlation；它不是 subscription resource ID。A* 為 A1／A2 建立的是**新訂閱**，不是沿用 A 的下層訂閱。論文提及的 A→A1／A2 與 A*→A1／A2 subscription ID prefixes 是特定 CIFAR-10 treatment 的追蹤例子，不能當成固定配置值。

既有第四章草稿將 *accuracy recovery* 定義為：替換後某次 Root validation accuracy 達到至少故障注入前一輪的值；它不等同於替換關係就緒，也不代表 treatment 的整體學習品質優於 baseline。MNIST 第 12 輪為 79.60%，第 13／14 輪為 60.60%／58.75%，A* 首次貢獻的第 15 輪為 82.20%；CIFAR-10 第 20 輪為 54.10%，第 21／22 輪為 51.30%／51.75%，第 23 輪為 55.55%。兩組皆在首次含 A* 的 accepted round 達到此舊門檻，之後繼續完成預定輪數。後續 E0–E3 改採第 7 節的 paired-baseline／95% CI 判定；舊單次結果不得套用新定義而宣稱已恢復。

## 5. 後續實驗的共同條件與比較關係

本文沿用論文第三章對形成／修復過程的三個名稱：

- **Initial topology intention／instruction**：Root 下發的拓樸建立意圖，可包含候選者及選擇要求；候選者尚不等於已確認的參與者。
- **Realized topology**：完成必要 preparation 並確認參與後，實際形成的 parent–child 關係；尚未確認或失敗的候選者可一併回報，但不算已形成的關係。
- **Accepted realized topology**：Root 判定 realized topology 滿足適用的 formation requirements 後接受的拓樸；符合條件時可以是部分或深度不一致的樹。

這三者描述拓樸形成或修復的不同時點；某一訓練輪實際成功貢獻的 participant set 另行紀錄，不等於 accepted realized topology。

以下 E0–E3 是接下來要跑的實驗，**尚不是上節單次 run 的結果**。E0 是同一工作負載、同一 seed 的無故障對照；E1、E2a、E2b 以 E0 為配對基準。每個選用的工作負載與情境使用同一組五個 seeds；同一 seed 固定資料分割、模型初始化、local epochs、FedProx／aggregation 設定與總 accepted rounds。E0／E1 的接受門檻維持一致；E2／E3 若因 direct cohort 或存活者數量改變而需不同的接受條件，必須事先寫清楚並在分析時標示，不能假稱只改了故障類型。預期值是待驗證主張，不能寫成已達成的結果。

| 情境 | 重要性 | 相對 E0 的主要變動 | 主要要證明的事 |
| --- | --- | --- | --- |
| E0：無故障 baseline | 必要 | 不注入故障 | 正常完整參與的學習軌跡與五 seed 變異 |
| E1：A 故障、A* 接手 | 核心、必要 | A 停止，由預部署的 A* 接回 A1／A2 | 同一 procedure 下替換受影響 Branch，B／C 關係保持 |
| E2a：A 故障、A1／A2 直掛 Root | 核心、強烈建議 | 不使用 A*；Area A 兩個 Leaves 上移 | 改變 hierarchy depth 且保留 Area A 全部資料 |
| E2b：僅 A1 直掛 Root | 建議的 ablation | A 永久停止，A1 上移、A2 不可用 | 部分修復與資料覆蓋減少時的 policy 決定 |
| E3：Leaf A1 消失 | 次要、可選 | A 保留，只失去 A1 | descendant-edge loss 的逐級回報及 degraded subtree 接受 |

E0／E1 沿用第 2–3 節的 Root–A/B/C–Leaves 部署、資料與工作負載，E1 的故障觸發點沿用 MNIST 第 12 輪、CIFAR-10 第 20 輪後。E2a／E2b 的永久 A 故障及 E3 的 A1 故障，須在執行前固定與 E0／E1 可比較的觸發輪次；目前提供的 E2／E3 敘述沒有另定精確輪次。E3 只需選一種資料異質性較明顯的 partition，不預設兩種工作負載都跑。若 E2 使用兩種工作負載，各自仍須有同 seed 的 E0 配對。

比較單位是同工作負載、同 seed、同 accepted Root round 的結果；記錄 seed 與 run 的對應，避免把不同初始化或分割造成的差異誤作拓樸效果。E2a／E2b 改變 Root 的 direct-child 數量時，accepted participant／completion 條件必須以該情境實際 cohort 清楚表述，不能不加說明地把原三個 Branch 的比例套到新 cohort。這是實驗條件的定義，不是本文要決定的程式修改。

## 6. 各實驗情境與預期觀測

### E0：無故障 baseline

Root 持續與 A、B、C 訓練；各 Branch 維持原本兩個 Leaves，A* 即使預部署也不接手健康的 A。與故障組使用同一組五個 seeds、資料分割、初始模型、訓練預算與接受門檻。預期所有 accepted rounds 都有完整的 A／B／C 貢獻；初始拓樸建立後，沒有 topology-version change 或新增訂閱。逐輪 validation accuracy／loss 的平均軌跡與 95% CI，是後續 paired comparison、accuracy AUC、endpoint 與 recovery 判定的基準。若 E0 自身出現缺席或異常，也須照實保留紀錄，不能先假定每輪完整。

### E1：A 失效，A* 接管 A1／A2

MNIST 在第 12 個、CIFAR-10 在第 20 個 accepted Root round 後停止 A 的服務；A* 在故障前已部署，但只在 A 失效後建立 A*→A1／A2 新訂閱。Root→B／C 及 B／C 的下層關係不應因 Area A 的修復而重建。檢查同一 run 的 `mlCorreId` 是否延續、受影響的舊／新訂閱 ID、拓樸修復 instruction、realized topology 與 Root 接受後的 accepted realized topology，以及每輪成功貢獻者。預期先出現只由 B／C 支撐的 degraded rounds，再恢復 A*、B、C 的完整參與；「五個 seeds 都成功」是目標，不是事先確定的觀測。

因 A1／A2 的資料與原邏輯分組均保留，修復後 accuracy／loss 可與同 seed 的 E0 接近；但故障期間的缺席、聚合順序與 timing 可能造成差異，不預設逐輪完全重合、零損失或一定恢復。Replacement ready 與 A* 首次出現在 accepted Root outcome 中，必須分別記錄。

### E2a：A 永久消失，A1／A2 直接改掛 Root

A 永久不可用，且此情境沒有 A* 接手。Root 讓仍存活的 A1、A2 成為 direct FL clients；修復後的邏輯拓樸為 `Root→A1/A2`、`Root→B→B1/B2`、`Root→C→C1/C2`。要確認 Root→A1／A2 的新訂閱、B／C 原關係未改、整個 run 的 `mlCorreId` 延續，以及拓樸修復 instruction、realized topology 與 accepted realized topology 均可表達不同深度的分支。這是驗證 **uneven-depth topology restructuring** 的核心情境，不是另一個同層 Branch replacement。

六個 Leaves 的資料仍可參與，但 Root 同時收到直接 Leaf 與 Branch aggregate，aggregation order 與時序會改變。預期 endpoint performance 可與 E0／E1 比較，不保證曲線完全相同；若沒有接受新拓樸或無法繼續 accepted rounds，須記為情境失敗，而非只展示成功的種子。

### E2b：A 永久消失，只有 A1 直接改掛 Root

A 不回來，A1 存活並直接回報 Root，A2 保持不可用。此 ablation 特別觀察 Root 的拓樸修復 instruction 與實際形成的 realized topology 不一致時，是否依 policy 接受 `Root→A1`、`Root→B→B1/B2`、`Root→C→C1/C2`。應同時保存 instruction 中的 A1／A2 候選、realized topology 中實際建立的 A1 關係，以及 Root 的接受／拒絕結果；接受後才稱為 accepted realized topology，避免只留下「修復成功」布林值。

若仍能持續 accepted rounds，A2 的資料卻已永久缺席。因此 accuracy／loss 或 class-specific performance 偏離 E0／E1／E2a，不能直接歸因於 protocol failure；須把 topology outcome 與資料覆蓋改變分開討論。若 Root 拒絕此部分修復，則應按實際 decision 報告，不能把它併入「已恢復」統計。

### E3：Leaf A1 消失，A 維持運作

在指定 accepted round 後永久停止 A1；A、A2、B、C 與其餘 Leaves 仍存活。觀察 A 是否回報自己 realized subtree 中的 A1 edge loss，Root 是否依 policy 接受只剩 A2 的 degraded Area A；Root→A／B／C 不應重建。此情境只驗證 HFL／NWDAF 的 descendant-edge report 與接受邊界，不用來主張新的 FL convergence theory。

Root 的 aggregation threshold 維持 E0 設定。第 2 節所述 Branch policy 卻要求兩個 Leaves 都成功，**不足以讓 A 只憑 A2 正常完成下層聚合**。E3 若要驗證「A 保留且以 A2 繼續」，其實驗條件必須明定 A 對單一存活 Leaf 的接受方式；不能宣稱原設定不變即可跑出預期結果。模型影響取決於 A1 所持資料，可能是整體 accuracy 下降、特定類別受影響，也可能很小。若 Root 無法接受 degraded subtree，應保留拒絕結果，不從分析中移除。

## 7. 跨情境證據與分析口徑

### 7.1 Protocol evidence 為主要結果

每個 run 都要保留是否完成重配置；跨五個 seeds 可報告成功數，例如 `5/5` 或實際較低值。證據至少能辨認：故障前後的 topology version 或等效變更標記、Root 的 initial topology intention／後續修復 instruction、實際建立的 realized topology、Root 的接受／拒絕結果及接受後的 accepted realized topology、`mlCorreId`、舊／新 subscription IDs、未受影響的 Root→B／C edges、每輪 selected／successful／failed participant set，以及控制面 message／API-call 數量。這些是**新實驗欲蒐集的證據**，不表示既有單次 run 已保存全部欄位，也不預設一定存在名為 `topologyVersion` 的既有欄位。

時間線分開量測 `failure→detection`、`detection→reparent instruction`、`instruction→new subscriptions ready`、`ready→first accepted contribution`，另給 `failure→first post-reconfiguration contribution` 的整體耗時。故障注入、就緒與首次 accepted contribution 是不同事件；對 E3 或被拒絕的拓樸，應明確標示不適用或未達成，不能填入虛構時間。若某 run 沒有恢復，報為 `not recovered`，並保留其事件與學習曲線。

### 7.2 Model metrics 為輔助結果

固定 held-out validation set 上，保存初始模型與每個 accepted Root round 的 accuracy／loss；五個 seeds 彙整逐輪 mean 與 95% CI。另比較固定 post-failure accepted-round window 的 accuracy AUC、最後 accepted round 的 validation endpoint accuracy／loss、與同 seed E0 的 paired 差值、rounds-to-recovery，以及 wall-clock failure-to-contribution time。Final official test 可用於 completed model 的獨立評估，不得與逐輪 validation 混為一談。資料缺席的 E2b／E3 必須同時呈現 participant／class coverage，不單靠總 accuracy 判斷協定成敗。

後續實驗採老師提出的 recovery 定義：修復後 Root validation accuracy **首次進入相同工作負載之 E0 五 seed、在對應 accepted round 估計的 95% CI 範圍，且連續維持兩個 accepted rounds**。同 seed 的 E0 仍用於 paired effect comparison。Recovery 是離線分析判定，不是訓練期間的停止或接納條件。對無法修復或無法連續滿足門檻者，明確記為 `not recovered`。這個定義不同於第 4 節舊稿的「回到故障前 accuracy」。

正式統計前尚需固定 95% CI 的估計方法、post-failure AUC window 與 E2／E3 的故障觸發輪次；這些屬分析／實驗規格，不在本文預先設計成 PyMTLF runtime 行為。五個 seeds 的成功率、accuracy 接近程度和 recovery rounds 均須由實際結果決定。

## 8. 既有證據位置與使用界線

下表中的 `5G_NWDAF_Infrastructure/` 與 `testbed-docs/`，在目前 workspace 分別位於 `resources/references/5G_NWDAF_Infrastructure/` 與 `resources/references/testbed-docs/`；兩者都只作為唯讀參考。

| 來源 | 本文用途 |
| --- | --- |
| 本次提供的論文第三章草稿 | 對齊 initial topology intention／instruction、realized topology 與 accepted realized topology 的用語 |
| 本次提供的論文第四章草稿 | 決定目前論文採用的情境、術語、圖表敘事與數值；尚屬草稿 |
| 本次提供的 E0–E3 實驗要求 | 決定後續實驗的研究目的、五 seed 比較、預期證據與 recovery 判定；尚未執行 |
| `5G_NWDAF_Infrastructure/testbed.protocol-hierarchical.yaml` | VM／NWDAF／PyMTLF 對應、GPU／CPU 指派、候選優先級、Root／Branch policy 與 timeout |
| `5G_NWDAF_Infrastructure/experiments/protocol-hierarchical/{mnist,cifar10}/` | 論文選用的兩組 scenario、資料切分、訓練參數與故障觸發設定 |
| `testbed-docs/5g-nwdaf-infrastructure/plans/protocol-driven-hierarchical-fl-experiment/chapter-4-experiment-materials-draft.md` | 已保存 run 的摘要、驗證／測試曲線、故障時間線與來源清單；文件本身仍是草稿 |
| `testbed-docs/5g-nwdaf-infrastructure/plans/protocol-driven-hierarchical-fl-experiment/formal-branch-replacement-comparison-plan.md` | 原始實驗條件、執行狀態與舊 CIFAR-10 配對的區分 |
| `nwdaf-docs/docs/plans/hierarchical-federated-learning/protocol-extension-implementation/slices/Slice 3 Branch Replacement without Retained-result Recovery Detailed Plan.md` | 不使用 retained-result handoff 的實作範圍 |

本次唯讀對照使用的參考版本為 `5G_NWDAF_Infrastructure@b796331` 與 `testbed-docs@8fb75fc`。`runs/protocol-hierarchical/` 原始目錄未包含在目前可讀的 reference clone 中，因此本文對精確 subscription ID prefixes、逐筆 `mlCorreId` 一致性與逐輪事件的描述，依論文草稿及 testbed-docs 轉錄，**未在此重新核對原始 `run.json`／`events.jsonl`**。若要把這些逐筆識別碼寫成最終論文證據，需再對實際 run artifacts 逐筆確認。Host inventory 也不是 per-run snapshot。既有單次配對沒有「A 故障且不進行任何修復」的對照組，不能單獨推論 replacement 相對於完全不修復的因果效果；後續 E2a／E2b 則是不同的修復方式，不屬於完全不修復的對照。
