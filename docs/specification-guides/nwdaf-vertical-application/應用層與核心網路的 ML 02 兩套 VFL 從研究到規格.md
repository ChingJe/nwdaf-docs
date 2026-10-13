# 應用層與核心網路的 ML 02 兩套 VFL 從研究到規格

[第 01 篇][ml-01]指出標準裡有兩套垂直式聯邦學習（VFL）：核心網路的工作組（SA2）定義了 NWDAF 與 AF 的 VFL，應用層的工作組（SA6）在 AIMLE 裡定義了另一套，兩份規格互不引用。

本篇回答一個問題：**這兩套 VFL 在研究階段各自考慮過什麼，哪些做法進了規格、哪些沒有，以及為什麼兩邊沒有對齊。**

**本篇的定位：查閱用的對照紀錄，不需要通讀。** 它記的是「研究階段的哪個做法、出自哪個方案、有沒有進規格」，用途是選定 VFL 相關的題目之後回來查出處。只想知道結論的話，讀下面「先講結論」與第 5 節的比較表就夠了。

依據是兩份研究報告與它們對應的規格：

| | 核心網路這一側 | 應用層這一側 |
| --- | --- | --- |
| 研究報告 | TR 23.700-84 的議題二 | TR 23.700-82 的議題四 |
| 對應的規格 | TS 23.288 §5.4、§5.5、§6.2H | TS 23.482 §8.18 |

先講結論：

- **兩個研究同一年平行進行，互相沒有提到對方，而且各自把對方的範圍排除在外。**
- 核心網路這一側有十三個候選方案，但研究報告沒有任何評估文字，看不到取捨的理由。
- 研究階段出現過、最後沒有進規格的做法，整理出來有 28 項。其中和隱私、標籤、模型有關的幾項，正好是規格現在的留白。
- 研究結論寫了、規格卻沒有落實的只有兩件事：**AF 如何把標籤提供給 NWDAF**，以及**初始模型如何提供**。
- 應用層這一側在研究階段只有一個方案，對象是 UE。用 GPSI 對齊樣本的做法是 Release 20 才加的，採用了「中介者」的設計；核心網路這一側在研究時有方案提過類似的設計，沒有被採納。

研究報告不是規範。本篇說「沒有進規格」時，比對的是本地的 TS 23.288 V20.2.0 與 TS 23.482 V20.3.0，查證方式與限制見第 9 節。

## 1. 兩個研究的時間與範圍

| | TR 23.700-84 議題二（SA2） | TR 23.700-82 議題四（SA6） |
| --- | --- | --- |
| 議題名稱 | 5GC Support for Vertical Federated Learning | Supporting Vertical FL at enablement layer |
| 變更歷史的第一筆 | 2024-01 | 2023-10 |
| 本地版本 | V19.0.0（2024-12）。變更歷史註明 2024-06 的 V1.0.0 之後內容沒有變動 | V19.1.0（2024-09） |
| 候選方案數 | 13 個（方案 #13 至 #25） | 1 個（方案 #18） |
| 評估 | 沒有。§7 只有一則 Editor's note | 有，§8.18.4 與 §11.5 |
| 參與者 | NWDAF 與 AF | UE 上的 AIMLE client |

兩份報告互相沒有提到對方。全文搜尋的結果：TR 23.700-82 沒有出現 TR 23.700-84 或 SA2 的 VFL；TR 23.700-84 沒有出現 SA6、SEAL、AIMLE 或 TR 23.700-82。

兩邊也各自把對方的範圍排除在外：

- SA2 的議題描述寫：「Application layer-based VFL requiring communication between AFs and/or UEs application client, is out of scope.」（TR 23.700-84 §5.2.2 NOTE 1）
- SA6 對自己方案的評估寫：「This solution is feasible and doesn't introduce any dependency to 3GPP network systems.」（TR 23.700-82 §8.18.4）

**解讀：** 這兩句話劃出了一條界線。SA2 只管「至少有一個 NWDAF 參與」的 VFL，SA6 只管「完全在應用層」的 VFL。界線兩邊各自完整，但跨過界線的組合（例如應用層的服務以 AF 的身分加入 NWDAF 的 VFL）沒有人研究。

依據：[TR 23.700-84 §5][r84-5]、[Annex A][r84-A]；[TR 23.700-82 §8.18][r82-818]、[Annex A][r82-A]。

## 2. 核心網路這一側：研究了什麼

### 2.1. 動機

研究報告給了兩個使用情境：

- **情境四：NWDAF 之間的 VFL。** 各個 NWDAF 對同一個 Analytics ID 收集到的輸入資料不同。動機之一是多廠商環境下，NWDAF 之間不必分享模型就能合作（§5.1.4）。
- **情境五：用 VFL 做服務體驗分析。** NWDAF 持有網路側的資料，AF 持有應用側的資料，因為隱私政策不能直接交換。報告舉的例子是 NWDAF 的 access speed、network access delay，與 AF 的 stall time、frame rate（§5.1.5）。

情境五明寫了兩個限制：AF 這一端怎麼收集資料不在範圍內；AF 的模型訓練怎麼做也不在範圍內（§5.1.5 與其 NOTE 1）。

### 2.2. 要回答的問題

§5.2.2 列出的研究問題：

1. 哪些情境、哪些條件下，用 VFL 訓練是合理的。
2. NF 的發現與選擇要怎麼強化。
3. 訓練與推論的程序要怎麼強化。
4. VFL 訓練出來的模型怎麼監控效能。
5. **要不要、以及如何把模型提供給訓練的參與者。**
6. **怎麼支援樣本與特徵的對齊。**

安全交給 SA WG3（§5.2.2 NOTE 2），RAN 與 UE 不在範圍內（NOTE 3）。

### 2.3. 十三個方案

| 方案 | 主張 | 誰當 server |
| --- | --- | --- |
| #13 | AF 握有標籤並當 server，經 NEF 與 NWDAF 做樣本與特徵對齊、訓練、推論 | AF |
| #14 | NWDAF 當 server 發起訓練與推論 | NWDAF |
| #15 | 把 server、握有標籤的一方、沒有標籤的一方定為三種邏輯功能；引入 contribution weights | MTLF |
| #16 | 用新的資料對齊服務做樣本與特徵對齊；讓 client 之間傳中間結果 | NWDAF |
| #17 | 只談註冊與發現 | NWDAF |
| #18 | NWDAF 或 AF 都可當 server；**由 NEF 比對樣本**，並用暫時的識別碼隱藏 NWDAF | 兩者皆可 |
| #19 | 只談 AF 發起的推論；推論的成員可以和訓練時不同 | AF |
| #20 | 只談推論，為推論另外定義能力與服務 | NWDAF |
| #21 | 把模型切成 bottom model 與 top model，用於應用層的 QoE | 兩者皆可 |
| #22 | 沿用 MTLF、AnLF、ADRF 的分工；模型存在 ADRF；**對齊時檢查 user consent** | NWDAF |
| #23 | 跨網域的 VFL；涵蓋**標籤的分享**、由 client 計算 loss、user consent | 兩者皆可 |
| #24 | AF 當 server，請 NEF 找 client，也可以由 NEF 做對齊 | AF |
| #25 | 只談註冊、發現與準備 | 兩者皆可 |

「主張」一欄是歸納，不是原文。十三個方案彼此重複，也互相矛盾，這在研究報告裡是正常的；結論只有 §8.2。

**十三個方案都沒有評估的小節，§7 的整體評估也是空的。** 所以研究報告沒有留下「某個做法為什麼不採納」的文字。要知道理由，得看會議文件，本地沒有。

依據：[TR 23.700-84 §6][r84-6]、[§7][r84-7]。

## 3. 核心網路這一側：什麼進了規格，什麼沒有

### 3.1. 結論大多落實了

§8.2 有 25 條原則與 14 則 NOTE。逐條對照 TS 23.288 V20.2.0 的結果：

| 判定 | 數量 | 例子 |
| --- | --- | --- |
| 原則被採納 | 22 條 | server 與 client 的定義、單一 PLMN、server 取樣本的交集、NEF 做識別碼對應 |
| 原則被部分採納 | 2 條 | P#2.4.2（標籤）、P#2.4.5（client 之間互傳） |
| 原則被別的做法取代 | 1 條 | P#2.1.4（AF 當 server 時綁什麼） |
| NOTE 留待規範階段，後來有決定 | 12 則 | 樣本對齊併入準備階段；模型各自存在本地 |
| NOTE 留待規範階段，**規格仍然沒有規定** | 2 則 | NOTE 6（標籤）、NOTE 8（初始模型） |

進了規格的做法，可以追到來源的方案：

| 規格裡的做法 | 來自 |
| --- | --- |
| 用暫時的識別碼對 untrusted AF 隱藏 NWDAF（規格稱 external NWDAF ID） | 方案 #13、#18 |
| 不標準化的 feature ID，註冊在 NRF | 方案 #22 |
| contribution weights | 方案 #15 |
| 推論可以只用部分 client，或 server 自己推論 | 方案 #23 |
| 全域的模型指標與 VFL 狀態回報 | 方案 #14 |

### 3.2. 結論寫了，規格沒有落實的

**標籤。** P#2.4.2 寫：「When the NWDAF acts as VFL Server, the NWDAF can receive labels from an AF.」NOTE 6 把怎麼提供留到規範階段。TS 23.288 V20.2.0 的條文只寫 server 用標籤來計算，沒有寫標籤從哪裡來。所有 VFL 服務操作的參數清單（§6.2H.3、§6.2H.4、§7.12、§7.13、第 11、12 章）裡，`label` 是零筆。

**初始模型。** NOTE 8 把初始模型如何提供給 client 留到規範階段。規格的 VFL 程序沒有下發初始模型的步驟，只在 NOTE 的舉例裡出現「preconfigured initial model」。

**被取代的一條。** P#2.1.4 寫 AF 當 server 時，VFL 關聯到「internal AF process」。規格改成一律以 Analytics ID 為參數，另外允許 AF 使用非標準的值（§6.2H.2.1.2 NOTE 2）。「internal AF process」這個詞只出現在結論，十三個方案都沒有。

**有限度支援的一條。** P#2.4.5 寫 client 可以把中間結果給其他 client。唯一提出這件事的方案 #16，是讓 NWDAF client 經 NEF 把中間結果傳給 AF client。規格裡的樣子不同：只有 untrusted AF 當 server 時，NEF 可以挑一個 NWDAF 替其他 NWDAF 收攏結果，AF 只看得到這一個（§6.2H.2.1.2 步驟 8）。

### 3.3. 研究階段出現過，規格沒有的做法

事實表整理出 28 項。下面依主題列出和研究題目最有關係的。「研究報告自己的保留」是指該方案或結論對這個做法留下的但書。

**樣本對齊**

| 做法 | 方案 | 研究報告自己的保留 |
| --- | --- | --- |
| **由 NEF 比對雙方的 sample ID、決定交集** | #18 | §8.2 NOTE 5 留待討論。規格最後由 server 決定交集，NEF 只做識別碼對應 |
| 由 NEF 代 AF 做樣本與特徵對齊 | #24 | 方案自己註明也可以由其他 NF 做 |
| **把隱私保護的對齊方法做成服務的參數** | #16 | 無 |
| 對齊時向 UDM 查 user consent，不同意的 UE 從樣本清單移除 | #22 | 無 |
| 對齊時查 UE 位置，訓練中監看 UE 是否可達 | #22 | 無 |

方案 #18 的原文是：common sample ID「are determined by the NEF comparing sample ID(s) from NWDAF and AF(s)」。

方案 #16 是十三個方案中唯一提到隱私保護對齊方法的。它把「alignment technique」列為對齊服務的必要輸入，原文是「such as Private Set Intersection (PSI), feature hashing, or other techniques that ensure data privacy and integrity」。但同一個方案的程序又寫 client 把 sample ID 交給 server 取交集，兩者沒有協調。

**標籤**

| 做法 | 方案 | 研究報告自己的保留 |
| --- | --- | --- |
| **AF 當 client 時，把標籤的值交給當 server 的 NWDAF** | #23 | 方案註明需要再討論。§8.2 採納了原則，沒有定義做法 |
| server 沒有標籤時，由 client 代算 loss 再回傳 | #23 | 同上。每一輪多一次來回 |
| AF 當 server 時，反過來把標籤或它的二元值交給 NWDAF | #23 | 寫「as agreed in handshake」 |
| client 也握有標籤 | #16 | 方案註明需要再討論 |

標籤的傳遞只有方案 #23 一個方案寫過。它預期 AF 提供的標籤「is either a digital representation or an aggregated value to protect privacy」。它也說明了反方向的用途：讓 NWDAF 學到哪些特徵影響應用的 QoE，之後不依賴 AF 的資料也能推論。

方案 #18 與 #22 各有完整的訓練程序，卻完全沒有提到標籤。

**模型**

| 做法 | 方案 | 研究報告自己的保留 |
| --- | --- | --- |
| server 把初始模型或參數下發給 client | #13、#14、#24 | 方案 #14 列為待研究並交給 SA3；§8.2 NOTE 8 留待討論 |
| 下發 bottom model、模型識別碼或切分點 | #21 | 方案列為待研究（必要性與安全） |
| VFL 的模型存到 ADRF | #15、#22 | 方案 #15 列為待研究；方案 #16、#23 排除 |
| 推論的成員可以和訓練時不同；模型在 NF 之間移動 | #19、#25、#20、#22 | 方案 #14 列為待研究。規格最後要求訓練與推論是同一個 NF（§5.4） |

**中間結果的保護**

| 做法 | 方案 | 研究報告自己的保留 |
| --- | --- | --- |
| 傳送中間結果前，確認 user consent 或套用隱私保護方法 | #23 | 方案註明方法由 SA WG3 定義 |
| 保護中間結果的方法 | #18、#23 的 NOTE | 交給 SA WG3 |

`encrypt`、`homomorphic`、`differential`、`secure` 這幾個字，在十三個方案、§8.2 與 TS 23.288 的 VFL 條文中都是零筆。§8.2 對安全、授權、user consent 沒有任何文字。

**參與者與架構**

| 做法 | 方案 | 研究報告自己的保留 |
| --- | --- | --- |
| 把協調者與握有標籤的一方分開 | #15、#22 的 NOTE | 方案 #15 列為待研究 |
| 由 NEF 聚合多個 NWDAF 的中間結果 | #13 | 方案註明留到規範階段。規格改由一個 NWDAF 聚合 |
| 沿用 `Nnef_MemberUESelectionAssistance` 來做對齊 | #13 的 NOTE 提問 | 規格用新的服務 |

AF 對 AF、UE 或 RAN 參與，研究範圍一開始就排除了，沒有任何方案處理。跨 PLMN 沒有被明文排除，但也沒有方案處理。

### 3.4. 規格新增，研究報告沒有的

這些出現在 TS 23.288 V20.2.0，但 §8.2 與十三個方案都沒有：

- VFL interoperability indicator 與 VFL Interoperability Information 的兩層設計。前者註冊在 NRF，後者在準備階段協商。
- NWDAF 請 AF 當 server 來訓練（`Naf_Training`、`Nnef_Training`）。
- 訓練的迭代編號、checkpoint、delta list、離開請求、重新開始的時間。
- 對齊的附加條件：資料樣本的時間窗、最少樣本數。

**解讀：** 規範階段補上的多半是讓程序跑得起來的管理細節。研究階段爭論過的實質問題（標籤、初始模型、隱私保護）沒有在規範階段得到答案，而是被「內容不標準化」與「靠 interoperability indicator 事先約定」吸收了。

依據：[TR 23.700-84 §8][r84-8]、[方案 #13][r84-s13]、[#16][r84-s16]、[#18][r84-s18]、[#21][r84-s21]、[#22][r84-s22]、[#23][r84-s23]；[TS 23.288 §5.4][n54]、[§6.2H.2.1][n62H21]、[§6.2H.2.2][n62H22]、[§6.2H.2.3][n62H23]、[§6.2H.3][n62H3]。

## 4. 應用層這一側：研究了什麼

### 4.1. 議題與方案

TR 23.700-82 的議題四只問兩件事（§5.4）：

1. 怎麼確保所有訓練的一方有對齊的樣本範圍，例如相同的使用者。
2. 怎麼發現不同網域之間有哪些特徵可用。

只有方案 #18 處理這個議題。它的對象是 UE：VFL 的成員是部署在 UE 上的 AIMLE client。流程是：

1. AIMLE client 把自己的 VFL 資料（能收集哪些特徵、屬於哪些資料網域）註冊到 AIMLE server。
2. VAL server 請 AIMLE server 訓練模型，並表明要用 VFL。
3. AIMLE server 對候選成員發出**能力評估**的請求，內含一個測試任務。資料的對齊是測試任務的一部分。VAL server 可以提供標籤。
4. AIMLE server 依回覆決定成員，再協調它們訓練。成員把中間結果送給 AIMLE server，AIMLE server 回傳梯度。

§12.4 的結論只有一句：方案 #18 可以進入規範階段。§11.5 另外註明，UE 之間的樣本與特徵對齊在 Release 19 研究，若有缺漏留給之後的 Release。

### 4.2. 進規格之後的變化

方案 #18 成為 TS 23.482 §8.18.2。有兩處比研究報告退了一步：

- **訓練的協調。** 方案 #18 步驟 7 寫 AIMLE server 協調成員進行訓練。規格在 §8.18.2 步驟 10a 之後加了一則 NOTE：AIMLE server 如何協調成員執行 VFL 訓練，不在這個 Release 的範圍。
- **樣本對齊的條件。** 規格把對齊改寫成「sample binding」：AIMLE server 把 VAL server 定義的樣本集合送給 client，client 把自己的資料對應上去。對應條件的細節標為 FFS（表 8.18.3.7-1 之後的 Editor's Note）。

### 4.3. Release 20 加入的第二個程序

TS 23.482 §8.18.2b 是研究報告裡沒有的。它處理的是**應用後端之間**的 VFL：VFL client 與 VFL server 都是 VAL server，樣本是 UE。

TS 23.482 的變更歷史顯示它是 Release 20 才加入的：

| 日期 | 變更請求的標題 | 版本 |
| --- | --- | --- |
| 2026-03 | Sample Alignment Enablement for VAL Servers in VFL | V20.1.0 |
| 2026-07 | VFL client and server registration | V20.2.0 |
| 2026-07 | Sample alignment in VFL | V20.2.0 |

做法是讓 AIMLE server 當中介者：

1. 各個 VAL server 交出以自己的 UE 識別碼表示的樣本清單。當 client 的在註冊時交，當 server 的在發起任務時交。
2. AIMLE server 經 NEF 的 UE ID 查詢程序，把這些識別碼換成 GPSI。
3. AIMLE server 用 GPSI 找出共同的樣本，為每個共同樣本發一個 correlation ID。
4. 各個 VAL server 只拿到自己的識別碼與 correlation ID。規格要求 GPSI「shall not be shared with VFL client(s) and VFL server」。

之後的訓練與推論由 VAL server 之間自行進行，不在程序範圍內。

**解讀：** 應用層是在核心網路的 VFL 定案之後，才走到「樣本是 UE、要向核心網路查識別碼」這一步。這時它面對的問題和 SA2 當初面對的一樣，但選了不同的答案。

依據：[TR 23.700-82 §5][r82-5]、[§8.18][r82-818]、[§11.5][r82-115]、[§12][r82-12]；[TS 23.482 §8.18][a818]、[Annex D][a-D]。

## 5. 同一個問題，兩邊怎麼回答

| 問題 | 核心網路的 VFL（TS 23.288 §6.2H） | 應用層的 VFL，成員是 UE（§8.18.2） | 應用層的 VFL，成員是應用後端（§8.18.2b） |
| --- | --- | --- | --- |
| 誰參與 | NWDAF 與 AF | UE 上的 AIMLE client | VAL server |
| 樣本是什麼 | UE | VAL server 定義的樣本集合，識別碼沒有定義 | UE |
| 誰算樣本的交集 | VFL server，它是訓練的參與者 | AIMLE server | AIMLE server，它不是訓練的參與者 |
| 參與者看得到彼此的網路識別碼嗎 | 看得到。SUPI 或 GPSI 的清單在 server 與 client 之間交換 | 規格沒有提到 | 看不到，只拿到 correlation ID |
| 標籤 | server 使用，來源沒有規定 | VAL server 可以提供 | 沒有提到 |
| 初始模型 | 沒有規定 | 訓練請求裡的模型資訊，是要分發給成員去訓練的模型 | 沒有提到 |
| 訓練中交換的內容 | 定義了訊息，內容不標準化 | 成員送中間結果，AIMLE server 回傳更新資訊 | 不在範圍 |
| 訓練的協調 | 定義了完整的程序 | 明寫不在這個 Release 的範圍 | 不在範圍 |
| 用到核心網路的地方 | 整個機制都在核心網路的分析框架裡 | 沒有 | NEF 的 UE ID 查詢 |
| user consent | VFL 的條文沒有提到 | 能力評估的 NOTE：收集 UE 資料要考慮 user consent | 轉換識別碼要符合營運者的政策與 consent 要求 |

兩邊強的地方剛好相反：

- **核心網路這一側把訓練與推論的信令定義得完整**，但樣本對齊沒有保護識別碼，標籤與模型怎麼來沒有規定。
- **應用層這一側把樣本對齊的識別碼保護做進了程序**，但訓練怎麼進行幾乎沒有定義。

**解讀：** 核心網路這一側在研究時，方案 #18 提過由 NEF 比對 sample ID，也就是讓一個不參與訓練的 NF 來算交集。這和應用層後來在 §8.18.2b 讓 AIMLE server 當中介者是同一種設計。SA2 沒有採納，研究報告沒有說明原因。

## 6. 為什麼兩邊沒有對齊

以下是**解讀**。研究報告與規格都沒有討論這個問題。

1. **範圍一開始就切開了。** SA2 把應用層的 VFL 排除在外，SA6 把不依賴核心網路當成方案的優點。兩邊都沒有理由去看對方。
2. **研究是同時進行的。** 兩份報告的內容都在 2024 年定案，誰也沒有成品可以引用。
3. **應用層的第二個程序來得晚。** 用 GPSI 對齊樣本是 2026 年才加入 TS 23.482 的。那時 TS 23.288 的 VFL 已經存在，但新條文沒有引用它，而是自己定義了一套。
4. **兩邊對「AF」的看法不同。** 核心網路的規格把 AF 當成一個具備 VFL 能力的參與者，不管它背後是什麼。應用層的規格把自己的 server 當成 AF 來用 NEF，但只用來查資訊與設定 QoS，沒有把它當成 VFL 的參與者。

## 7. 可以追的問題

以下全部是**解讀**，承接[第 01 篇][ml-01]第 12 節，把其中和 VFL 有關的幾條講得更具體。這些不是已選定的研究方向。

1. **中介者式的樣本對齊能不能用在 NWDAF 與 AF 之間。** 研究階段有方案提過由 NEF 算交集，應用層後來用了同樣的設計。可以問的是：由 NEF 算交集時，NWDAF 與 AF 各自少暴露了什麼，NEF 多承擔了什麼，信令多了多少。
2. **標籤怎麼跨過信任邊界。** 研究結論同意 NWDAF 可以從 AF 取得標籤，規格沒有定義。方案 #23 給了三種做法，各自的成本與洩漏不同。這和[文獻筆記第 06 篇][lit-06]的設定（QoE 標籤在應用提供者手上）直接相關。
3. **初始模型與模型怎麼切。** 規格只說「事先設定好」。研究階段有下發模型、指定切分點的做法，被列為待研究後沒有下文。
4. **user consent 放在哪一步。** 方案 #22 放在樣本對齊，方案 #23 放在每次傳送中間結果之前。規格的 VFL 條文沒有提到。這一項要先確認 TS 33.501 有沒有規定。
5. **同一個後端同時是 AIMLE server 與具備 VFL 能力的 AF。** 兩套樣本對齊的結果（AIMLE 的 correlation ID 與 TS 23.288 的 sample ID 清單）能不能互通，要自己定義。

## 8. 還缺的資料

| 缺什麼 | 為什麼需要 |
| --- | --- |
| SA2 的會議文件 | 研究報告沒有評估文字。做法為什麼沒被採納，只能從會議文件與會議紀錄看。方案條文中只出現過兩個文件編號（S2-2403591、S2-2401830），都在方案 #24 |
| TS 33.501 | SA3 有沒有為 VFL 規定授權、user consent 或中間結果的保護。本地沒有這份規格 |
| TS 23.288 的 Release 19 | 「沒有進規格」目前是指 V20.2.0 沒有。無法排除 Release 19 曾經有、後來改掉 |
| SA6 在 Release 20 的研究報告，若有 | §8.18.2b 是 Release 20 加入的，本地只有變更歷史的標題，沒有它的研究過程 |

3GPP 的會議文件是公開的，可以從 3GPP 的檔案伺服器取得。

## 9. 查證方式與限制

- 核心網路這一側：2026-10-13 把 TR 23.700-84 議題二的全部內容與 TS 23.288 的 VFL 條文整理成一份附原文引句的事實表（573 句引句），每一句都以字串比對確認出現在本地規格中。本篇引用的英文原文另外逐句查過。
- 應用層這一側：TR 23.700-82 的 §5.4、§8.18、§11.5、§12.4 與 TS 23.482 的變更歷史是直接閱讀；TS 23.482 §8.18 的內容沿用第 01 篇的事實表。
- 「零筆」是對本地 Markdown 做字串搜尋的結果。
- 「沒有進規格」是缺席的判斷：全文閱讀加上字串搜尋後沒有找到。沒有任何一句條文明說「不做」。
- 第 2.3 節的方案摘要與第 3.1 節的數量是歸納，界線有主觀成分。
- 研究報告的圖只看了三張。圖裡可能有文字沒寫出來的細節。

## 規格來源

| 規格 | 本地版本 | 本篇用到的部分 |
| --- | --- | --- |
| TR 23.700-84 | V19.0.0（Release 19） | §4、§5.1.4、§5.1.5、§5.2.2、§6.13 至 §6.25、§7、§8.2、Annex A |
| TR 23.700-82 | V19.1.0（Release 19） | §5.4、§8.18、§11.5、§12.4、Annex A |
| TS 23.288 | V20.2.0（Release 20） | §5.2、§5.4、§5.5、§6.2A、§6.2H、§7.12、§7.13、第 11、12 章 |
| TS 23.482 | V20.3.0（Release 20） | §8.3、§8.4、§8.18、Annex D |

這些內容是研究用的解讀，不構成目前 Release 18 實作的需求。

[ml-01]: 應用層與核心網路的%20ML%2001%20機制總覽.md
[lit-06]: ../../notes/literature/seal-application-enablement/06%20Schwarzmann%202022%20以%20AF%20資料估計%20QoE.md
[r84-5]: ../../../specs/Rel-19/TR%2023.700-84/5%20Use%20Cases,%20Motivations%20and%20Key%20Issues.md
[r84-6]: ../../../specs/Rel-19/TR%2023.700-84/6%20Solutions/README.md
[r84-7]: ../../../specs/Rel-19/TR%2023.700-84/7%20Overall%20Evaluation.md
[r84-8]: ../../../specs/Rel-19/TR%2023.700-84/8%20Conclusions.md
[r84-A]: ../../../specs/Rel-19/TR%2023.700-84/Annex%20A%20Change%20history.md
[r84-s13]: ../../../specs/Rel-19/TR%2023.700-84/6%20Solutions/6.13%20Solution%20No.13%20Sample%20and%20Feature%20alignment%20and%20general%20training%20procedure%20for%20the%20VFL.md
[r84-s16]: ../../../specs/Rel-19/TR%2023.700-84/6%20Solutions/6.16%20Solution%20No.16%20Support%20for%20VFL%20with%20NWDAF%20and%20AF%20as%20Participants.md
[r84-s18]: ../../../specs/Rel-19/TR%2023.700-84/6%20Solutions/6.18%20Solution%20No.18%20Vertical%20Federated%20Learning%20between%20NWDAF%20and%20AF/README.md
[r84-s21]: ../../../specs/Rel-19/TR%2023.700-84/6%20Solutions/6.21%20Solution%20No.21%20Vertical%20Federated%20Learning%20for%20support%20of%20Application%20Layer%20QoE.md
[r84-s22]: ../../../specs/Rel-19/TR%2023.700-84/6%20Solutions/6.22%20Solution%20No.22%20Vertical%20Federated%20learning%20considering%20internal%20NWDAF%20architecture.md
[r84-s23]: ../../../specs/Rel-19/TR%2023.700-84/6%20Solutions/6.23%20Solution%20No.23%20Cross%20domain%20VFL%20involving%20NWDAF%20and%20AF/README.md
[r82-5]: ../../../specs/Rel-19/TR%2023.700-82/5%20Key%20issues.md
[r82-818]: ../../../specs/Rel-19/TR%2023.700-82/8%20Solutions/8.18%20Solution%20No.18%20Supporting%20VFL%20in%20Enablement%20Layer.md
[r82-115]: ../../../specs/Rel-19/TR%2023.700-82/11%20Overall%20evaluation/11.5%20Key%20issue%20No.4%20Supporting%20Vertical%20FL%20at%20enablement%20layer.md
[r82-12]: ../../../specs/Rel-19/TR%2023.700-82/12%20Conclusions.md
[r82-A]: ../../../specs/Rel-19/TR%2023.700-82/Annex%20A%20Change%20history.md
[n54]: ../../../specs/Rel-20/TS%2023.288/5%20Network%20Data%20Analytics%20Functional%20Description/5.4%20Vertical%20Federated%20Learning%20%28VFL%29.md
[n62H21]: ../../../specs/Rel-20/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2H%20Vertical%20Federated%20Learning%20among%20NWDAFs%20and%20AFs/6.2H.2%20Procedures/6.2H.2.1%20Registration%20and%20Discovery%20procedure%20for%20Vertical%20Federated%20Learning.md
[n62H22]: ../../../specs/Rel-20/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2H%20Vertical%20Federated%20Learning%20among%20NWDAFs%20and%20AFs/6.2H.2%20Procedures/6.2H.2.2%20Preparation%20procedure%20for%20Vertical%20Federated%20Learning.md
[n62H23]: ../../../specs/Rel-20/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2H%20Vertical%20Federated%20Learning%20among%20NWDAFs%20and%20AFs/6.2H.2%20Procedures/6.2H.2.3%20Training%20Procedure%20for%20Vertical%20Federated%20Learning.md
[n62H3]: ../../../specs/Rel-20/TS%2023.288/6%20Procedures%20to%20Support%20Network%20Data%20Analytics/6.2H%20Vertical%20Federated%20Learning%20among%20NWDAFs%20and%20AFs/6.2H.3%20Contents%20of%20ML%20Model%20VFL%20Training%20services%20for%20Vertical%20Federated%20Learning.md
[a818]: ../../../specs/Rel-20/TS%2023.482/8%20Procedures%20and%20information%20flows/8.18%20Support%20Vertical%20FL.md
[a-D]: ../../../specs/Rel-20/TS%2023.482/Annex%20D%20Change%20history.md
