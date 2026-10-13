# SEAL 與應用層使能的相關文獻

這個資料夾收錄 2026-10-13 一次文獻調查的結果：學術界與產業界拿 3GPP SEAL 及其相鄰概念做了什麼，以及哪裡能和 NWDAF 接上。內容包含十四份文獻的精讀介紹與本頁的總覽。

SEAL 本身的規格解讀在[SEAL 系列][seal-series]，本資料夾的用詞與它一致。

標示「解讀」的段落是我們的整理與推論，不是文獻的主張。

## 1. 結論

- **直接研究 SEAL 的論文很少。** 標題或摘要寫明 SEAL 的約八篇，集中在三個作者群：Samsung 的 SA6 標準代表、Ericsson Hungary 的 Géza Szabó、University of Antwerp–imec。
- **沒有找到把 SEAL 與 NWDAF 真正接起來的論文。** 同時提到兩者的論文有，但都只是並列點名（第 03 篇），或把 NWDAF 當成背景元件（第 12、14 篇）。
- **建立在 SEAL 上的服務幾乎沒有學術論文。** ADAE 只找到一篇（第 12 篇），沒有實驗數字。AIMLE 沒有專題論文，討論最多的第 14 篇也只有一段文字加一張圖。VAE、UAE、SEALDD、NSCE 都沒有找到專題論文。
- **十四份精讀的結論都是「選讀」，沒有一份是必讀。** 每份都有一兩個可用之處，但沒有一份同時做到對題、有實作、有扎實的評估。
- **真正呼叫 SEAL API 的只有兩篇**（第 01、02 篇）。其餘談 SEAL 的論文都是標準說明或架構討論。
- **搜尋階段的印象普遍高估了相關性。** 關鍵字出現次數多，不代表文獻真的處理了那個主題。第 03、07、10、13 篇讀完全文後都比預期弱。

## 2. 十四份精讀

每份介紹的結構相同：一分鐘版本、問題、背景、方法、實驗、限制、與 SEAL 的關係、與 NWDAF 的關係、對研究的用處、值得追的文獻、閱讀建議、查證紀錄。

| # | 文獻 | 類型 | 與 SEAL | 與 NWDAF | 最有用的地方 |
| --- | --- | --- | --- | --- | --- |
| [01][n01] | Charpentier 等，VTC 2024 | 實作與實測 | 實作三支 SEAL API | 沒有提到 | 少數有實測數字的 SEAL 實作；指出通知管理 API 的缺口 |
| [02][n02] | Szabó，IEEE Access 2023 | 實作與實測 | 實作網路資源管理 | 只有一句 | SEAL 接上真實 5G 專網的公開實作，程式碼開源 |
| [03][n03] | Gómez-Jerez 等，ETFA 2024 | 資料模型 | 只有點名 | 只有點名 | 工業端期待從 5G 取得哪些資料的清單 |
| [04][n04] | Pateromichelakis 等，IEEE Access 2019 | 架構提案 | 沒有提到，指向 SA6 | 架構的一部分 | 「在 NWDAF 之外放應用層分析實體」的早期來源 |
| [05][n05] | Milani 等，OJCOMS 2024 | 實作與實驗 | 沒有提到 | 期刊版只提兩次 | 電信商與應用業者之間交換量最小的協作學習設計 |
| [06][n06] | Schwarzmann 等，TNSM 2022 | 模擬 | 沒有提到 | 主題 | 特徵在電信商、標籤在應用業者的問題設定 |
| [07][n07] | Taksande 等，arXiv 2025 | 綜述 | 沒有用這些名稱 | 涵蓋 | Release 19 工作項目的索引 |
| [08][n08] | Kousaridas 等，IEEE Commun. Mag. 2021 | 模擬 | 沒有提到 | 主題 | NWDAF 預測經 NEF 送到應用的期刊引用來源 |
| [09][n09] | Silva 等，SBRC 2026 | 實作與實測 | 沒有提到 | 沒有提到 | free5GC 上 AF 經 NEF 影響路由的實測時間 |
| [10][n10] | 5GAA 技術報告，2020 | 技術報告 | 只出現四處 | 主題 | 車用產業對 QoS Sustainability analytics 的需求落差清單 |
| [11][n11] | Shah 等，5GWF 2020 | 標準說明 | 主題 | 沒有提到 | SEAL 設計動機的出處：源自 MCPTT 的輔助服務 |
| [12][n12] | Pateromichelakis 等，IEEE IoT Magazine 2022 | 架構說明與原型 | ADAE 是主題 | 只出現三次 | ADAES 唯一的同儕審查出處 |
| [13][n13] | Fragkos 等，MeditCom 2021 | 架構討論 | 主題之一 | 沒有提到 | 垂直應用、產業 enabler、SEAL、核心網路的分層圖 |
| [14][n14] | Saimler 等，PIMRC 2025 | 概念性立場論文 | 比較的三個選項之一 | 背景元件 | 少數把 AIMLE 與 NWDAF 寫在同一段的論文 |

第 10 份不是同儕審查的論文。把它列入精讀是因為找不到以 VAE 為主題的論文，而它有專節描述 VAE server 傳遞 QoS 預測。

第 11 到 14 份是第二批。它們只開放摘要，取得全文後補做精讀。

## 3. 建議的閱讀順序

**解讀：** 依目的分三條路線，每條都從最值得讀的開始。

| 目的 | 順序 | 說明 |
| --- | --- | --- |
| 了解 SEAL 為什麼存在 | 11 | 動機與沿革的出處，約 20 分鐘 |
| 看 SEAL 實際被怎麼用 | 02 → 01 | 兩篇都真的呼叫了 SEAL API。02 的對照表可以看出應用需求如何變成 SEAL 請求 |
| 看 NWDAF 的分析如何到達應用 | 08 → 10 | 08 是標準路徑的簡明說明；10 接著指出這條路徑對車用不夠的地方 |
| 找和 NWDAF federated learning 相關的題目 | 06 → 05 → 14 → 07 | 06 提供問題設定，05 提供一種不交換梯度與標籤的做法，14 可當 AIMLE 與 NWDAF 並列討論的引用，07 只需看 VFL 一段並留意它寫錯的地方 |
| 看應用層分析這條線 | 04 → 12 | 同一位第一作者 2019 年的提案與 2022 年的 ADAES 說明 |
| 需要時再查 | 09、13、03 | 09 用於 free5GC 上的 NEF 實作參考；13 只需看分層圖；03 主要當引用來源 |

## 4. 搜尋確認的缺口

**解讀：** 以下是六個方向的搜尋都沒有找到的東西。沒有找到不等於不存在，搜尋的限制見第 8 節。

| 缺口 | 依據 | 相關規格 |
| --- | --- | --- |
| NWDAF 與 AF 之間的 vertical federated learning | 找到的兩篇 VFL 論文都只處理 NWDAF 之間，沒有 AF | TS 23.288 §6.2H |
| 在開源 5GC 上實作「AF 把應用層資料交給 NWDAF」 | 第 06 篇是模擬，且明講不模擬資料收集的信令 | TS 23.288 §6.2.8 |
| 實作「NWDAF 的分析經 NEF 開放給第三方 AF」 | 以 AnalyticsExposure 搜尋只得到規格與專利 | TS 23.288 §6.1 |
| SEAL 的服務實際使用 NWDAF 的分析 | 第 02 篇只說 QoS 監測的資料來源可以是 NWDAF，沒有實作 | TS 23.434 §14.3.3.4 |
| 應用側資料作為特徵能讓分析變準多少 | 第 06 篇把應用側資料只當標籤，沒有做這個比較 | 無 |
| 應用業者與電信商之間的資料同意與信任邊界 | 學術論文幾乎空白，內容都在 3GPP 文件與專利 | 無 |
| 應用層的 federated learning（AIMLE）與 NWDAF 的 federated learning 如何分工或協作 | 第 14 篇把 federated learning 歸在 AIMLE 一側，沒有和 NWDAF 自己的 federated learning 連在一起；沒有找到其他討論 | TS 23.482 §8.9、TS 23.288 §6.2C |
| ADAES 接上真實或開源的 5GC 與 NWDAF | 第 12 篇的原型是五台虛擬機，沒有 5GC、NEF、CAPIF、NWDAF，也沒有結果數字 | TS 23.436 第 8 章 |

第一項和第二項可以合成一個題目：第 06 篇的資料分佈（網路 KPI 在電信商、QoE 標籤在應用業者）正好是 vertical federated learning 的情境，而第 05 篇提供了一個不交換梯度的對照組。這個連結是我們的推論，三篇文獻都沒有這樣寫。

另一個接點來自規格而不是文獻：TS 23.436 §8.14 的 Collision Detection Analytics 把 NWDAF 列為資料來源之一，情境與第 01 篇相同。第 01 篇引用的規格版本早於這項功能。

## 5. 可以重用的程式與資料

以下網址照文獻或搜尋結果抄錄，沒有逐一開啟確認內容與授權。

| 名稱 | 網址 | 來源 | 內容 |
| --- | --- | --- | --- |
| ros2-3gppSA6-mapper | `https://github.com/Ericsson/ros2-3gppSA6-mapper` | 第 02 篇 | ROS2 設定對應到 SEAL 請求的節點 |
| C2N | `https://github.com/LABORA-INF-UFG/C2N` | 第 09 篇 | CAMARA Traffic Influence 轉 NEF |
| 5G-AAS | `https://github.com/uwicore/5G-AAS` | 第 03 篇 | 5G UE 與網路的 AAS 模型 |
| seal-server | `https://github.com/nuk-icslab/seal-server` | 搜尋結果 | SEAL 的身分管理與群組管理，連結 OpenCAPIF |
| OpenCAPIF | `https://ocf.etsi.org/` | 搜尋結果 | ETSI 的 CAPIF 開源實作 |
| NEF_emulator | `https://github.com/medianetlab/NEF_emulator` | 搜尋結果 | 可設定的 NEF API 模擬環境 |
| 5G-MAG Data Collection | `https://www.5g-mag.com/reference-tools/data-collection/` | 搜尋結果 | Data Collection AF 的參考實作，未實作 NWDAF 端 |

## 6. 引用前要注意的原文問題

精讀時在原文發現的不一致。除第 10 篇以外，下列各項都在 PDF 中找到了原句；第 10 篇只確認了內文那一句，表格在 PDF 中不是文字，表格一側依賴撰寫者的閱讀。詳細內容與其餘較小的問題見各篇的「限制」與「查證紀錄」。

「VAL server 作為 API exposing function」這個說法在第 01 篇與第 13 篇都出現。TS 23.434 §6.4.2.3 寫的是 VAL server 作為 API invoker，引用這兩篇對 CAPIF 角色的描述時要留意。

| 文獻 | 問題 |
| --- | --- |
| 01 | 寫 VAL server 作為 API exposing function；TS 23.434 §6.4 規定的是 SEAL server |
| 01 | 與 ITS-G5、C-V2X 的比較引用自作者的前作，不是同場量測，且比的是單程延遲 |
| 02 | 寫 NF 把服務註冊到 NEF，應為 NRF；引用 NWDAF 時給的規格編號是 TS 33.521 |
| 05 | 摘要寫效能最多提升 37.7%，內文寫 37.3% |
| 07 | 寫 VFL server 把模型合併成最終模型；TS 23.288 §5.4 的機制是交換中間結果，各方保留本地模型 |
| 07 | 寫 VFL 可支援與其他營運商網路協作；本地 Release 20 的 TS 23.288 §5.4 限於單一 PLMN |
| 09 | 寫 N4 迴圈占非同步延遲的 87%；依論文自己的數字，87% 是占總時間的比例 |
| 10 | §5.6.2 的選項編號在表格與內文之間互相矛盾 |
| 11 | 兩處寫 SEAL-X、SEAL-E「not shown in Fig. 1」，依撰寫者判讀應為圖 3 |
| 12 | 參考文獻 [14] 寫成 TS 23.533，依標題應為 TS 28.533；[15] 把 TR 23.700-36 寫成 TS |
| 13 | 寫支援 CAPIF 時 VAL server 作為 API exposing function |
| 13 | 表 I 的星等與內文的評語矛盾，星等也沒有定義 |
| 14 | 三個選項都假設應用可以取用 NWDAF 的 MLOps 工具，並以現在式書寫；論文自己在另一處說這是未來可能的演進 |

第 07 篇是預印本，尚未經同儕審查。

## 7. 沒有精讀的文獻

### 7.1. 只能取得摘要

依摘要判斷與 NWDAF 輔助 federated learning 相關，但出版社只開放摘要：

- Chen 等，5GS Assistance for Federated Learning Member Selection in Trajectory Prediction Scenarios，ICCC Workshops 2022，`10.1109/ICCCWorkshops55477.2022.9896691`
- Zhang 等，5GC Enabled Lightweight Federated Learning under Communication Quality Constraint，ICCC 2024，`10.1109/ICCC62479.2024.10681875`

原本列在這裡的四篇已取得全文並完成精讀，即第 11 到 14 篇。

### 7.2. 有全文但這次沒有選

| 文獻 | 位置 | 沒選的原因 |
| --- | --- | --- |
| Mestoukirdi、Khanfouci，Reliable Vertical Federated Learning in 5G Core Network Architecture | arXiv 2505.15244 | 只處理 NWDAF 之間的 VFL，偏演算法；與 NWDAF federated learning 最接近，值得另外讀 |
| Guo 等，Beyond Per-Request QoS | arXiv 2605.00570 | 應用與網路雙向協調，有測試平台，但 NWDAF 只是背景 |
| Castañeda Garcia 等，A Tutorial on 5G NR V2X Communications | arXiv 2102.04538 | 只需讀 QoS Sustainability Analytics 一節，內容與第 08 篇重疊 |
| Tomaszewski 等，Mobile Networks' Support for Large-Scale UAV Services，Energies 2022 | `10.3390/en15144974` | 同時談到 SEAL 與 NWDAF，但只是概念提案 |
| Gramaglia 等，A unified service-based capability exposure framework，ETT 2022 | `10.1002/ett.4598` | 能力開放的統一框架，有測試平台，與 SEAL 的關係較遠 |
| Piscione 等，CAPIF 加 NEF 的概念驗證 | arXiv 2508.09150 | 只有兩頁，併入第 09 篇的補充一節 |

完整的候選清單（六個方向共約八十筆）沒有收進本資料夾。

## 8. 調查方法與限制

**流程。** 六個方向各由一個 agent 搜尋候選文獻並確認存在，篩選後每份文獻由一個 agent 讀全文並撰寫介紹，最後抽查。六個方向是：SEAL 架構本身、CAPIF 與 NEF 的能力開放、應用層資料分析、應用層 AI/ML 與 federated learning、預測式 QoS 與應用調適、應用資料回饋給網路。

**篩選標準。** 與 SEAL 或 NWDAF 的關聯程度、有無實作或實驗、能否取得全文。只有摘要的文獻不寫精讀介紹。

**抽查做了什麼。**

- 每份介紹挑出關鍵數字與主張，回頭對照 PDF 的文字，兩批共核對五十餘處。PDF 文字中找得到的全部相符。
- 有幾個數值只出現在圖中，無法用文字比對，例如第 04 篇的 517 個樣本與第 08 篇的預測提前量。這些依賴撰寫者讀圖。
- 第 6 節列出的原文問題逐項回到 PDF 找原句，結果見該節開頭的說明。
- 介紹中引用的四處規格條文（TS 23.434 §14.3.3.4、TS 23.436 §8.14、TS 23.288 §5.4、TS 23.434 第 17 章）對照了本地規格。
- 二十三張重繪的圖全部實際渲染，沒有語法錯誤。

**沒有做的事。**

- 沒有逐句核對每份介紹；抽查以外的內容依賴撰寫的 agent 自己的查核。
- 各篇從圖上讀出的近似數值都標為解讀，沒有另外驗證。
- 第 5 節的程式庫沒有開啟確認。

**搜尋的限制。**

- 使用的資料庫大多只索引標題與摘要。SEAL、VAE、ADAE 這類詞常常只出現在內文，所以內文有討論而摘要沒寫的論文很可能漏掉。
- 沒有使用 Google Scholar 與 IEEE Xplore 的全文檢索，也沒有做引用追蹤。
- OpenAlex 的每日額度在搜尋中途用完；Semantic Scholar 的關鍵字搜尋大多被限流。
- 沒有搜尋中文與韓文文獻。
- 專利與 3GPP 貢獻文件的數量遠多於論文，這次沒有整理。

## 9. 全文的位置

十四份文獻的 PDF 放在工作區的 `local-references/papers/seal-application-enablement/`，不在本文件倉庫內。各篇介紹的資訊表列有 DOI 或原始連結。

[seal-series]: ../../../specification-guides/nwdaf-vertical-application/README.md
[n01]: 01%20Charpentier%202024%20SEAL%20作為邊緣應用.md
[n02]: 02%20Szabó%202023%20ROS2%20與%20SEAL%20網路資源管理.md
[n03]: 03%20Gómez-Jerez%202024%205G%20的%20Asset%20Administration%20Shell.md
[n04]: 04%20Pateromichelakis%202019%20端到端資料分析框架.md
[n05]: 05%20Milani%202024%20ATELIER%20有限信任的網路分析.md
[n06]: 06%20Schwarzmann%202022%20以%20AF%20資料估計%20QoE.md
[n07]: 07%20Taksande%202025%20Release%2019%20的%20AI%20ML%20綜述.md
[n08]: 08%20Kousaridas%202021%20QoS%20預測與自動駕駛.md
[n09]: 09%20Silva%202026%20C2N%20CAMARA%20對接%20NEF.md
[n10]: 10%205GAA%202020%20eNESQO%20技術報告.md
[n11]: 11%20Shah%202020%20SEAL%20概觀.md
[n12]: 12%20Pateromichelakis%202022%20ADAES%20與%20NetApps.md
[n13]: 13%20Fragkos%202021%20VAE%20的實作挑戰.md
[n14]: 14%20Saimler%202025%20AI%20as%20a%20Service.md
