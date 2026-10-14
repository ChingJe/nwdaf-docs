# 應用層與核心網路的 ML 03 Daisy 與 AIMLE 的對照

本篇回答一個問題：**把實驗室的聯邦學習框架 Daisy 做成符合 AIMLE 的樣子，哪些對得上、哪些要新增、Daisy 的特色會不會遺失、擴充出來的功能能不能稱為 AIMLE，以及核心網路的資訊能插在哪裡。**

問題來自 [2026-10-08 的會議][mtg]：經理提出 Daisy 更像 SEAL 裡 AIMLE 的一種變形。該次紀錄只查了規格這一側，本篇補上 Daisy 的程式碼這一側。

**本篇的定位：要讀的文件。** 正文只留判斷與依據。逐項的對照放在文末兩個附錄，需要時再查。

| 比對的對象 | 版本 |
| --- | --- |
| Daisy | `open-source` 分支，commit `ce4f3763`（2025-03-19） |
| AIMLE 的流程規格（Stage 2） | TS 23.482 V20.3.0 |
| AIMLE 的介面規格（Stage 3） | TS 29.482、TS 24.560 V20.1.0 與對應的 OpenAPI 檔 |

Daisy 另有一個給 NWDAF 的 MTLF 使用的 `NWDAF-daisy` 分支。它是舊版的串接，不在本篇範圍。

先講結論：

- **角色對得上。** Master、Zone、Client 分別對應中央 AIMLE server、邊緣 AIMLE server、AIMLE client。
- **Daisy 也可以看成使用 AIMLE 的應用（VAL）。** 規格的 Role#3 由 VAL 提供訓練流程，AIMLE 只管成員。這樣看，Daisy 自有的功能是應用邏輯，不在規格範圍內。兩種看法可以並存。
- **Daisy 缺的是管理面。** AIMLE 要求的 client 描述、查詢、徵詢同意、模型倉庫，Daisy 都沒有。
- **Daisy 的特色不會遺失。** 階層、非同步、安全聚合、演算法抽換，都落在 AIMLE 沒有規定的範圍。但這些特色在 AIMLE 的介面上也看不出來。
- **真正的衝突只有連線方向。** Daisy 是 client 連 server，AIMLE 是 server 連 client。
- **AIMLE 的介面規格還沒寫完模型怎麼傳。** 流程規格要求 client 回傳模型參數，介面規格沒有欄位可放。Daisy 現成的傳輸可以補這一段。
- **擴充出來的功能不能稱為 AIMLE。** 實作者能做的擴充只有在訊息裡多帶欄位，而標準的 AIMLE 元件會忽略這些欄位。判斷以功能為單位：照規格做的是 AIMLE，規格沒有的是 Daisy 自己的擴充。
- **核心網路的資訊有地方插，但規格幾乎沒寫。** 點名 NWDAF 的只有選成員時的一句。Daisy 這一側最具體的缺口是移動：機制有了，判斷依據是亂數。

## 1. 角色對應

| Daisy | AIMLE | 說明 |
| --- | --- | --- |
| 發布任務的人（對 Master 發 HTTP） | VAL server | 可以和 AIMLE server 合設在同一個實體 |
| Master | 中央 AIMLE server | 收任務、選成員、聚合 |
| Zone | 邊緣 AIMLE server | 規格的階層只有中央與邊緣兩層 |
| Client | AIMLE client | 可以併入 UE 上的應用程式 |
| 沒有對應 | ML repository | Daisy 的模型只是 Master 本機的一個檔案 |

上表是「Daisy 本身就是 AIMLE」的看法，也是 10 月 8 日會議中經理的說法。規格還容許另一種看法，見 1.1。

### 1.1 另一種看法：Daisy 當成使用 AIMLE 的應用

TS 23.482 Annex C 列出 AIMLE 在 ML 生命週期中的三種角色：

| 角色 | 規格的描述 | 對應的程序 |
| --- | --- | --- |
| Role#1 | 使用者完全交給 AIMLE，AIMLE 執行整個流程，決定要交付的模型 | §8.20 |
| Role#2 | 使用者自己做一部分，其餘由 AIMLE 協助，例如訓練由 AIMLE 處理 | §8.3、§8.12 等 |
| Role#3 | AIMLE 不執行流程，流程由 VAL 提供。AIMLE 負責找出、註冊、儲存、分組、選擇執行 ML 操作的對象 | §8.4、§8.7 至 §8.10 等 |

Role#3 的原文是：「the AIMLE is not performing part of the ML operational workflow; however it serves as a platform to enable the AI/ML apps to utilize ML operational workflows which are provided by VAL.」

**解讀：** Role#3 幾乎就是「Daisy 自己訓練，AIMLE 幫它管成員」。依照這個角色，Daisy 位在應用層（VAL），Daisy 自有的功能是應用邏輯。TS 23.434 §6.2 NOTE 2 寫明「The functionalities and reference points of the vertical application layer are out of scope」，所以這些功能不受 SEAL 規格約束，第 6 節「能不能稱為 AIMLE」的問題也就不存在。

這個看法有一個前提。VAL 的定義都綁在某個產業的某項服務上（TS 23.434 §3.1）：

- Vertical application：「An application catering to a specific vertical」
- VAL server：「the server application function of a specific VAL service」
- VAL system：「The collection of applications, services, and enabling capabilities required to support a VAL service」

**解讀：** Daisy 是通用的 FL 框架，單獨看時不是 VAL。它被部署給某個應用使用時，例如替巡檢 App 訓練模型，就成為那個應用的 VAL server 端的一部分，或者說是那個 VAL system 裡的一項能力。

兩種看法的差別：

| | Daisy 當 AIMLE | Daisy 當 VAL |
| --- | --- | --- |
| Daisy 的位置 | AIMLE server | 應用層，呼叫 AIMLE |
| 對應的角色 | Role#1 或 Role#2：AIMLE 執行流程 | Role#3：流程由 VAL 提供 |
| Daisy 要實作的 AIMLE 部分 | AIMLE server 的 API（提供方） | 呼叫 AIMLE API 的一方（使用方），經 AIML-S |
| AIMLE server 由誰提供 | Daisy 自己 | 另一個 AIMLE 實作，或和 Daisy 放在一起的模組 |
| Daisy 自有的功能 | AIMLE 的擴充，不能稱為 AIMLE（第 6 節） | 應用邏輯，不在規格範圍內 |

兩種看法不互斥。Annex A.5 寫明「The AIMLE server can be co-located with the VAL server (e.g. AI server) in a single physical entity or deployed in different physical entities.」**解讀：** 同一套軟體可以一部分扮演 AIMLE server，另一部分扮演使用它的 VAL。採用哪一種看法，要由經理決定。

依據：[TS 23.482 §5][a5]、[Annex A][aA]、[Annex C][aC]；[TS 23.434 §3.1][s434d]、[§6.2][s434m]。

## 2. Daisy 要新增的東西

AIMLE 最核心的流程是：client 註冊、查詢 client、選擇 client 並徵詢同意、發布訓練、每輪訓練與回報。逐項比對如下。

| AIMLE 的要求 | Daisy 的現況 | 要做的事 |
| --- | --- | --- |
| client 註冊時交 profile：能力、資料集、位置、可用時段等（§8.7） | 只帶一個自己取的識別碼。名單只含此刻連著的 client，不持久保存 | 新增 profile 與持久的成員名冊 |
| 對外提供依條件查詢 client 的 API（§8.8） | 沒有對外介面 | 新增 |
| 依條件或名單選 client，逐一徵詢同意，人數不足則失敗（§8.9、§8.10） | 每輪從可用的 client 隨機抽，不徵詢，client 不能拒絕 | 新增條件篩選與徵詢步驟 |
| 訓練請求帶模型識別碼與位址、資料集識別碼、每輪樣本數、排程（§8.3、§8.12） | 任務設定只有輪數、演算法模組的路徑、模型檔路徑。資料集寫死在 client 的程式裡 | 補上這些欄位的對應 |
| 模型與成員資訊存在 ML repository（§8.11） | 沒有 | 新增，或以既有儲存對應 |

兩個補充：

- **Daisy 內建的選成員幾乎沒有彈性。** 內建的 `FedAvg` 每輪固定抽 2 個、等 2 份結果，任務設定改不了。篩選用的 `Criterion` 介面存在，但唯一的實作是依識別碼名單，沒有依屬性篩選的實作。
- **AIMLE 沒有規定必須實作哪些 API。** 規格沒有必選清單，也沒有符合性等級。**解讀：** 所謂符合 AIMLE，只能理解成「實作的那幾個 API 各自符合條文」。

依據：[TS 23.482 §8.3][a83]、[§8.7][a87]、[§8.8][a88]、[§8.9][a89]、[§8.10][a810]、[§8.11][a811]、[§8.12][a812]。Daisy 這一側：`src/py/daisyfl/common/client_manager.py`、`strategy/fedavg.py`、`strategy/sec_agg.py`、`master/server_api_handler.py`。

## 3. Daisy 的特色會不會遺失

| Daisy 的特色 | Daisy 實際的做法 | AIMLE 怎麼看 | 會不會遺失 |
| --- | --- | --- | --- |
| 演算法可抽換 | 選成員、等待條件、聚合各是外掛模組，由任務設定指定 | 請求只能指定 HFL 或 VFL 與訓練目標。聚合只寫「例如取平均」 | 不會 |
| 非同步 | 換一組模組即可，協定不變 | 三份規格沒有出現這個詞。介面是每個 client 各自回報，不妨礙 | 不會 |
| 安全聚合 | 五個階段，換一組模組即可 | 沒有任何文字 | 不會 |
| 階層式 | Zone 自己和 client 跑幾輪、自己聚合，再把結果交給 Master | 有中央與邊緣的階層式訓練（§8.28），但沒寫邊緣是否自己聚合 | 不會，但對應不完整 |
| 移動中的 client | client 改連另一個 Zone，把模型交給它 | 有邊緣之間的 context transfer（§8.24） | 不會，但做法不同 |
| 斷線恢復 | client 斷線後自己重連，結果不會遺失或重複 | 沒有對應。規格的處理是註冊到期、換人 | 不會 |

三個要注意的地方：

- **不遺失，但也不會更突出。** 前三項在 AIMLE 的介面上完全看不出來。符合 AIMLE 不會讓這些特色變成可以對外宣告的能力。
- **階層式在介面規格沒有 API。** §8.27 與 §8.28 只存在於流程規格。流程規格也沒有「邊緣內跑幾輪」的參數。
- **移動的做法和規格相反。** 規格的例子是新的邊緣 server 把結果**轉回**原本的 server。Daisy 是把模型**留在**新的 Zone，原本的 Zone 只被告知不用再等。

另外，Daisy 的幾項特色比說明文件講的弱，對照時要留意：

- **移動沒有任何位置輸入。** 範例裡誰要換 Zone 是亂數決定的，換去哪裡是寫死的位址。
- **斷線恢復只管 client 自己接回來。** client 在訓練途中消失時，同步的 `FedAvg` 沒有逾時，會一直等下去。

依據：[TS 23.482 §8.12][a812]、[§8.24][a824]、[§8.27][a827]、[§8.28][a828]。Daisy 這一側：`operator/`、`strategy/`、`client/grpc_client/c2z_connection.py`、`common/communicator.py`、`examples/05_proximal_transmission_pytorch/`。

## 4. 唯一的衝突：連線方向

| | Daisy | AIMLE |
| --- | --- | --- |
| 誰發起連線 | 一律是 client（或 Zone）連上層 | 註冊與取模型由 client 發起，其餘都是 server 對 client 發 HTTP |
| client 要不要能被連到 | 不用 | 要。註冊時必須報出自己的位址 |
| 傳輸 | gRPC 串流 | HTTP 請求與回呼 |

AIMLE 的徵詢同意、每輪訓練的下發、能力評估，都是 server 主動連 client。所以 Daisy 的 client 要新增一個 HTTP 服務端。

**未能確認：** UE 在 NAT 或防火牆後面時 server 如何連到它，三份規格都沒有提到。Daisy 由 client 發起連線的做法沒有這個問題。

依據：[TS 24.560][a24]、[OpenAPI][oapi] 中的 `TS24560_Aimles_AIMLEClientRegistration.yaml`、`TS24560_Aimlec_HFLTraining.yaml`。

## 5. 模型參數怎麼傳

流程規格和介面規格說的不一樣。

| 方向 | 流程規格的要求 | 介面規格實際能帶的 | 缺口 |
| --- | --- | --- | --- |
| server 給 client | 模型與模型參數，必填 | 模型的識別碼與位址 | client 用什麼方法去取，沒有規定 |
| client 給 server | 訓練後的模型參數，必填 | 六個效能數值 | **模型參數沒有欄位，位址也沒有** |
| server 給發布者 | 訓練輸出，例如模型參數 | 六個效能數值 | 要另外從模型倉庫取 |

介面規格裡唯一沾得上邊的欄位在另一個程序：啟動與停止訓練的服務操作（§8.20）有一個選填的聚合位址，說明是「the ML model aggregator URI to send the model updates」。它只給一個位址，沒有規定怎麼送、送什麼格式，也沒有說明和每輪訓練的關係。

**未能確認：** 這是刻意讓模型走另一條通道，還是介面規格尚未完成。三份規格沒有任何 NOTE 說明。HFL 訓練的 OpenAPI 檔版本仍是 alpha。

**解讀：** Daisy 的訊息本來就帶模型參數。把 AIMLE 當成流程的外殼、模型仍走 Daisy 的通道，是最直接的補法，而且聚合位址那個欄位可以用來指向 Daisy 的節點。

依據：[TS 23.482 §8.12][a812]、[§8.20][a820]、[§8.3][a83]；[OpenAPI][oapi] 中的 `TS24560_Aimlec_HFLTraining.yaml`、`TS29482_AIMLES_MLModelTraining.yaml`、`TS24560_Aimlec_AIMLEClientServiceOperations.yaml`。

## 6. 擴充之後還算不算 AIMLE

Daisy 補上 AIMLE 的介面之後，仍會保留 AIMLE 沒有的功能，例如 Zone 之間的 handover。本節回答兩件事：AIMLE 的 API 能怎麼擴充，以及擴充出來的功能能不能稱為 AIMLE。

### 6.1 規格允許的擴充

| 方法 | 規格的規定 | 限制 |
| --- | --- | --- |
| 在訊息裡多帶欄位 | AIMLE 的 OpenAPI 沒有任何物件禁止多帶欄位。接收端遇到不認得的欄位，"shall be ignored"。欄位名稱用 `vendorSpecific-` 加上廠商代碼 | 只有自己的元件看得懂，標準的 AIMLE 元件會直接忽略 |
| 功能協商 | 雙方用 bitmask 交換各自支援的選用功能 | TS 29.482 的 25 個功能表全部是空的。其中 6 個 API 連協商用的欄位都沒有，包含 `AIMLES_MLModelTraining`。功能編號由規格分配，不是給實作者自訂的 |
| 規格明定由廠商決定的內容 | §8.11 的 ML Model Interoperable Information，編碼與格式不規定，由廠商之間約定 | TS 23.482 中明寫 vendor-specific 的只有這一項 |

實作者不能自己做的：

- **升版本或改必填欄位。** `apiVersion` 只在不相容的修改時由 3GPP 調升。
- **在 AIMLE 的 API 名稱下加自己的操作。** **解讀：** 規格沒有明文禁止，但標準元件不會支援，這樣的操作不能算 AIMLE 的 API。

另外開一個自己的 API 是可以的。CAPIF 支援第三方發布自己的 service API。但它只是經 CAPIF 發布的 API，不是 AIMLE，也不是 SEAL 服務，因為 SEAL 服務的種類由 TS 23.434 列舉。

**未能確認：** TS 29.122 §5.2.13.1 說廠商擴充欄位是否適用，由各 API 的規格決定。TS 29.549 與 TS 29.482 都沒有提到，所以 `vendorSpecific-` 的命名規則對 AIMLE 是否正式適用，無法確認。本地的 TS 29.122 是 Release 18 的版本。

### 6.2 擴充出來的功能算不算 AIMLE

判斷的單位是功能，不是整個 Daisy。同一套 Daisy 可以一部分是 AIMLE，一部分是自己的擴充。本節以「Daisy 當 AIMLE」的看法為前提；若採 1.1 的「Daisy 當 VAL」，自有的功能屬於應用邏輯，不需要這樣判斷。

| 情況 | 能不能稱為 AIMLE | Daisy 的例子 |
| --- | --- | --- |
| 規格定義的功能，照規格做 | 可以 | HFL 訓練的訂閱與通知（§8.12） |
| 規格沒規定做法的部分，自己補上 | 功能是 AIMLE，做法不是 | 聚合方式（規格只寫「例如取平均」）；模型參數的傳法（§8.20 的聚合位址沒有規定怎麼送） |
| 規格沒有的功能 | 不能 | Zone 之間的 handover。§8.24 的 context transfer 把結果轉回原本的 server，Daisy 把模型留在新的 Zone，兩者不是同一件事 |

兩個補充：

- **沒有機構來判定。** AIMLE 沒有必選的 API 清單，也沒有符合性等級（見第 2 節）。能不能稱為 AIMLE，是說法正不正確，不是有沒有通過認證。
- **擴充欄位的內容依規格本身就不屬於 3GPP。** TS 29.500 §6.6.3 NOTE 2："The content (value) of those vendor-specific member elements, and their usage, is not to be defined by any of the 3GPP Technical Specifications."

**解讀：** 對 Daisy 的意思和第 5 節相同。和標準 AIMLE 元件互通時，只依靠標準欄位；Daisy 特有的東西（模型參數、Zone、handover）走擴充欄位或 Daisy 自己的通道。

依據：[TS 29.549 §6.5.1、§6.8][s549]；[TS 29.122 §5.2.7][s122f]、[§5.2.13][s122v]；[TS 29.500 §6.6][s500]；[TS 23.482 §8.11][a811]；[TS 23.222 §6.2][s222]；[TS 23.434 §6.1][s434]；TS 29.482 各 API 的 Feature negotiation 小節；[OpenAPI][oapi] 中的 `TS29482_*`、`TS24560_*`。

## 7. 核心網路的資訊能插在哪裡

AIMLE server 有幾個決定需要自己沒有的資訊。規格對資訊來源的寫法分兩種。

| 要做的決定 | 規格寫的來源 | Daisy 對應的位置 |
| --- | --- | --- |
| 選成員 | SEAL 的位置管理、NEF 的事件監看、**NWDAF 的 UE mobility 分析**。用字是 may，沒有程序 | 選成員的模組 |
| 成員是否符合速度、連線品質、掉線率的條件 | 沒有寫 | 選成員的模組 |
| 每個 client 的訓練排程 | 沒有寫 | 沒有對應，任務一進來就開始 |
| 何時聚合、等哪些 client | 沒有寫 | 等待條件的模組 |
| 成員即將不可用的預測 | 沒有寫 | 沒有對應 |
| client 換了服務區，何時移轉 | 邊緣運算的應用移轉程序（TS 23.558） | client 端程式自己決定，目前是亂數 |

三個觀察：

- **規格點名 NWDAF 的只有第一列。** 整份 TS 23.482 的核心流程中，NWDAF 只在選成員出現一次。
- **NWDAF 的角色仍然是提供預測。** 表中每一列都是「拿預測來做決定」，沒有任何一列讓 NWDAF 參與訓練。
- **移動是兩邊都有機制、中間缺判斷的地方。** Daisy 有換 Zone 的機制，AIMLE 有移轉的程序，NWDAF 有 UE mobility 的分析。三者之間沒有任何規格或程式把它們接起來。

依據：[TS 23.482 §8.9][a89]、[§8.12][a812]、[§8.24][a824]、[§8.4][a84]。

## 8. 對選題的意義

以下是**解讀**，不是已選定的方向。

1. **「讓 Daisy 符合 AIMLE」本身是工程工作。** 主要是補管理面的 API，以及在 client 加一個 HTTP 服務端。它不會改變 Daisy 的訓練方式。
2. **研究上可以指出來的東西有兩類。** 一類是規格的缺口與補法，最明確的是模型參數的上行。另一類是用網路的預測來驅動訓練的決定，最具體的是移動。
3. **NWDAF 在這條路上是提供預測的角色。** 這一點規格與程式兩側的結論相同。若題目要求 NWDAF 參與訓練，要走 [第 01 篇][ml-01]的機制三（NWDAF 與 AF 的垂直式 FL），那是另一條路。

## 9. 還沒確認的事

- **Daisy 的現行版本。** 本篇以 `open-source` 分支為基準。它是不是實驗室目前使用的主線，沒有確認。
- **Daisy 的行為是讀程式碼得出的。** 沒有執行。「會一直等下去」這類敘述沒有實測。
- **邊緣 AIMLE server 是否自己聚合。** §8.28 沒有寫。本篇假設它把收到的請求當成一次一般的訓練來處理，這是推論。
- **移轉由誰偵測移動。** §8.24 引用 TS 23.558 的應用移轉程序，本篇沒有往下追。
- **AIMLE 的安全規定。** 規格只有一句指向 TS 33.434，本地只有 Release 19 的版本。
- **廠商擴充欄位對 AIMLE 是否正式適用。** TS 29.549 與 TS 29.482 都沒有提到，見第 6.1 節。

## 附錄 A：AIMLE 流程規格與介面規格的不一致（查閱用）

| 程序 | 不一致之處 |
| --- | --- |
| client 註冊（§8.7） | 流程規格的 PLMN、能耗能力、漫遊設定，介面規格沒有屬性。介面規格多了必填的 client 位址 |
| client 查詢（§8.8） | 流程規格必填的 Allowed MNO info，介面規格沒有屬性 |
| HFL 訓練（§8.12） | 請求沒有模型參數、目前迭代、效能追蹤的屬性。通知沒有狀態與模型參數的屬性 |
| 模型訓練（§8.3） | 請求沒有輪數。通知的訓練輸出只有效能數值 |
| 訓練型態 | 列舉只有 `TRAIN_VFL` 與 `TRAIN_HFL`，兩者的說明文字互相顛倒 |
| 階層式（§8.27、§8.28） | 介面規格沒有 API |
| context transfer（§8.24） | 結果在介面規格只是一個字串。轉送結果的訊息沒有定義 |
| task transfer（§8.6） | 中間模型的屬性，型別裝不下模型 |
| 由 client 或應用後端請 server 代存模型（§8.11.2.2） | 流程規格與介面規格都沒有 API |
| 版本 | 34 個 OpenAPI 檔中 17 個帶 alpha，包含註冊、查詢、選擇、HFL 訓練 |

## 附錄 B：Daisy 沒有的項目（查閱用）

以下都是在 `open-source` 分支的原始碼與文件中實際搜尋後的結果。

| 項目 | 現況 |
| --- | --- |
| 列出已註冊 client 的介面 | 沒有。對外只有 `/publish_task` 與 `/metrics` |
| client 的能力、資料集、位置描述 | 沒有 |
| client 同意或拒絕 | 沒有 |
| 模型識別碼、版本、倉庫 | 沒有 |
| 資料集識別碼、每輪樣本數 | 沒有 |
| 訓練排程 | 沒有 |
| 認證與授權 | 沒有。只有選用的 server 端 TLS |
| 主動通知發布者 | 沒有。`/publish_task` 等任務跑完才回應 |
| 任務轉給另一個成員 | 沒有 |
| 成員可用性變化的事件 | 沒有 |
| 推論、垂直式 FL、切分式 | 沒有 |
| 任何位置或網路資訊的輸入 | 沒有 |

## 查證方式與限制

- 2026-10-14 由兩個 subagent 分別精讀 Daisy 與 AIMLE，各產出一份附原文引句的事實表。Daisy 572 句引句逐行比對原始碼，AIMLE 393 句引句比對規格與 OpenAPI 檔，全部相符。
- 本篇引用的關鍵事實另外直接回檔案確認過：聚合位址欄位、訓練型態說明顛倒、`FedAvg` 的固定人數、移動結果進入新 Zone 的佇列。
- 第 6 節的事實於 2026-10-14 直接回檔案確認：TS 29.482 的 25 個功能表以程式逐一檢查，全部是空的；34 個 OpenAPI 檔都沒有 `additionalProperties: false`；`TS29482_*` 中有 6 個檔沒有任何 supported features 欄位。
- 「沒有」是搜尋後零筆的判斷。
- AIMLE 的 §8.14 到 §8.16、§8.18、§8.22 以後的多數條文只做了關鍵字搜尋，沒有全文閱讀。
- 事實表不在本倉庫內。

## 規格來源

| 規格 | 本地版本 | 本篇用到的部分 |
| --- | --- | --- |
| TS 23.482 | V20.3.0（Release 20） | 第 4、5 章，§8.3、§8.4、§8.6 至 §8.13、§8.20、§8.24、§8.27、§8.28、Annex A（含 A.5）、Annex C（C.2 至 C.4） |
| TS 29.482、TS 24.560 | V20.1.0（Release 20） | 上列程序對應的 API 與資料模型 |
| OpenAPI | Release 20 | `TS29482_*`、`TS24560_*` 共 34 個檔 |
| TS 29.549 | V20.1.0（Release 20） | §6.5.1、§6.8：SEAL API 的版本與功能協商 |
| TS 29.122、TS 29.500 | V18.10.0（Release 18） | TS 29.122 §5.2.7、§5.2.13，TS 29.500 §6.6：功能協商與廠商擴充 |
| TS 23.222 | V20.1.0（Release 20） | §6.2：第三方的 service API |
| TS 23.434 | V20.1.0（Release 20） | §3.1：VAL 的定義；§6.1：SEAL 服務的列舉；§6.2：VAL 層不在規格範圍 |

Daisy 的原始碼在工作區的 `resources/daisy/`，不在本倉庫內，所以只列路徑、不設連結。

這些內容是研究用的解讀，不構成目前 Release 18 實作的需求。

[mtg]: 2026-10-08%20會議%20SEAL%20討論紀錄與查證.md
[ml-01]: 應用層與核心網路的%20ML%2001%20機制總覽.md
[a5]: ../../../specs/Rel-20/TS%2023.482/5%20Application%20architecture%20for%20enabling%20AI%20and%20ML%20services.md
[a83]: ../../../specs/Rel-20/TS%2023.482/8%20Procedures%20and%20information%20flows/8.3%20ML%20model%20training.md
[a84]: ../../../specs/Rel-20/TS%2023.482/8%20Procedures%20and%20information%20flows/8.4%20Support%20for%20FL%20member%20registration/README.md
[a87]: ../../../specs/Rel-20/TS%2023.482/8%20Procedures%20and%20information%20flows/8.7%20AIMLE%20client%20registration.md
[a88]: ../../../specs/Rel-20/TS%2023.482/8%20Procedures%20and%20information%20flows/8.8%20AIMLE%20client%20discovery.md
[a89]: ../../../specs/Rel-20/TS%2023.482/8%20Procedures%20and%20information%20flows/8.9%20AIMLE%20client%20selection.md
[a810]: ../../../specs/Rel-20/TS%2023.482/8%20Procedures%20and%20information%20flows/8.10%20AIMLE%20client%20participation.md
[a811]: ../../../specs/Rel-20/TS%2023.482/8%20Procedures%20and%20information%20flows/8.11%20ML%20model%20management.md
[a812]: ../../../specs/Rel-20/TS%2023.482/8%20Procedures%20and%20information%20flows/8.12%20HFL%20training.md
[a820]: ../../../specs/Rel-20/TS%2023.482/8%20Procedures%20and%20information%20flows/8.20%20AIML%20service%20operations%20control%20and%20management%20procedure.md
[a824]: ../../../specs/Rel-20/TS%2023.482/8%20Procedures%20and%20information%20flows/8.24%20AIMLE%20context%20transfer.md
[a827]: ../../../specs/Rel-20/TS%2023.482/8%20Procedures%20and%20information%20flows/8.27%20AIMLE%20server%20registration%20in%20hierarchical%20AIMLE%20deployments.md
[a828]: ../../../specs/Rel-20/TS%2023.482/8%20Procedures%20and%20information%20flows/8.28%20ML%20model%20training%20in%20hierarchical%20AIMLE%20deployments.md
[aA]: ../../../specs/Rel-20/TS%2023.482/Annex%20A%20%28informative%29%20Deployment%20scenarios.md
[aC]: ../../../specs/Rel-20/TS%2023.482/Annex%20C%20%28informative%29%20Role%20of%20AIMLE%20in%20ML%20Model%20Lifecycle.md
[a24]: ../../../specs/Rel-20/TS%2024.560/README.md
[s549]: ../../../specs/Rel-20/TS%2029.549/6%20SEAL%20Design%20Aspects%20Common%20for%20All%20APIs.md
[s122f]: ../../../specs/Rel-18/TS%2029.122/5%20T8%20APIs/5.2%20Information%20applicable%20to%20several%20APIs/5.2.7%20Feature%20negotiation.md
[s122v]: ../../../specs/Rel-18/TS%2029.122/5%20T8%20APIs/5.2%20Information%20applicable%20to%20several%20APIs/5.2.13%20Vendor-specific%20extensions.md
[s500]: ../../../specs/Rel-18/TS%2029.500/6%20General%20Functionalities%20in%20Service%20Based%20Architecture/6.6%20Extensibility%20Mechanisms.md
[s222]: ../../../specs/Rel-20/TS%2023.222/6%20Functional%20model/6.2%20Functional%20model%20description.md
[s434]: ../../../specs/Rel-20/TS%2023.434/6%20Generic%20functional%20model%20for%20SEAL%20services/6.1%20General.md
[oapi]: ../../../specs/Rel-20/openapi/README.md
[s434d]: ../../../specs/Rel-20/TS%2023.434/3%20Definitions,%20symbols%20and%20abbreviations.md
[s434m]: ../../../specs/Rel-20/TS%2023.434/6%20Generic%20functional%20model%20for%20SEAL%20services/6.2%20On-network%20functional%20model%20description.md
