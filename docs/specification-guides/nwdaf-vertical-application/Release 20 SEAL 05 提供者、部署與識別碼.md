# Release 20 SEAL 05 提供者、部署與識別碼

本篇回答一個問題：**SEAL 由誰提供、放在哪裡、提供時要做到多少、用什麼名字稱呼使用者和終端。** 前幾篇談的是功能，本篇談功能落到實際組織與網域時的樣子。

本文依據本地 TS 23.434 V20.1.0。前置閱讀是[第 01 篇][seal-01]與[第 03 篇][seal-03]。

## 1. 有哪些組織

§5.1 描述支援一個 VAL user 所需的商業關係：

| 角色 | 是什麼 |
| --- | --- |
| VAL user | 使用產業應用服務的使用者，與 VAL service provider 有服務協議 |
| VAL service provider | 提供產業應用服務的組織 |
| SEAL provider | 提供 SEAL 服務的組織（§3.1） |
| Home PLMN operator | 提供網路的營運者 |
| Visited PLMN operator | 漫遊時的受訪營運者，與 Home PLMN operator 有漫遊協議 |

這些角色之間的關係相當寬鬆：

- VAL service provider 可以和多個 SEAL provider 合作；SEAL provider 也可以和多個營運者合作。
- **SEAL provider 可以和 VAL service provider 或營運者是同一個組織。** 此時兩者之間的商業關係屬於組織內部。
- VAL service provider 與營運者也可以是同一個組織。

§5.3 進一步把 SEAL provider 拆成兩種：提供 SEAL server 的一方和提供 SEAL client 的一方。兩者可以是同一個提供者，也可以不同。不同時，VAL service provider 要和 SEAL client 提供者簽約，SEAL client 提供者再和 SEAL server 提供者簽約。

## 2. SEAL server 放在哪裡

§8.2 列出三種位置，並說明一個 SEAL server 可以支援多個 VAL server，也可以連接一個或多個營運者的網路。

| 位置 | 說明 | 規格 |
| --- | --- | --- |
| 營運者網域 | SEAL server 在營運者網域，VAL server 在 VAL service provider 網域 | §8.2.1 |
| VAL service provider 網域 | SEAL server 和 VAL server 都在 VAL service provider 網域 | §8.2.2 |
| 獨立的 SEAL provider 網域 | SEAL server 同時在前兩者之外 | §8.2.3 |

多個 SEAL server 之間可以互連，也可以不互連。互連時使用 SEAL-E，例如 §8.2.2 圖 8.2.2-3 的情境：兩個 SEAL server 各自連接一個營運者的網路，彼此經 SEAL-E 互連，共同服務經由兩個營運者連線的終端。

這些部署方式對應 §4.2.2 的需求：SEAL 要支援只部署在 PLMN 內、只部署在 PLMN 外，以及兩者並存。

## 3. SEAL client 放在哪裡

§8.3 的說明很短：

- SEAL provider 可以把 SEAL client 做成 UE SDK，放在終端作業系統之上（§8.3.1）。
- SEAL client 可以和 SEAL server 屬於同一個提供者網域（§8.3.2）。
- SEAL client 也可以由終端製造商提供，放在作業系統內或作業系統外（§8.3.3）。

## 4. 部署位置與信任域

**解讀：** [第 03 篇][seal-03]提到，NRM server 在信任域內直接呼叫 PCF，在信任域外則經 NEF。把這件事和本篇的部署位置放在一起，可以得到一個直覺的對應：

| SEAL server 位置 | 與核心網路互動的可能方式 |
| --- | --- |
| 營運者網域 | 可以直接使用核心網路的服務 |
| VAL service provider 網域或獨立 SEAL provider 網域 | 經 NEF |

這是推論，不是規格條文。規格的程序文字是以「是否在營運者信任域內」作為條件，沒有把信任域直接等同於第 2 節的部署網域。部署在營運者網域外但被營運者視為可信的情況，本文未找到條文說明如何處理。

## 5. 識別碼

第 7 章定義了 SEAL 使用的識別碼。它們都屬於應用層，和核心網路的識別碼是不同的體系。

| 識別碼 | 代表什麼 | 重點 |
| --- | --- | --- |
| User ID | 使用者向身分管理 server 認證時出示的身分 | 身分管理是共用服務，這組憑證不一定綁定單一 VAL service（§7.1） |
| VAL user ID | 使用者在某項 VAL service 內的唯一識別 | 可以用來存取任何 SEAL 服務；同一使用者在不同 VAL service provider 會有不同的 VAL user ID（§7.2） |
| VAL UE ID | 終端在某項 VAL service 內的唯一識別 | 依服務協議，可以使用 GPSI 作為 VAL UE ID（§7.3） |
| VAL service ID | 一項 VAL service | 可用於政策對應與 QoS 處理（§7.4） |
| VAL group ID | 一組使用者或終端 | 成員可以屬於不同的 VAL service provider（§7.5） |
| VAL system ID | 一個 VAL system | 全域唯一（§7.6） |
| VAL stream ID | 一條時間敏感通訊串流 | 由 VAL server 使用（§7.7） |

**應用後端用 VAL 識別碼說話，核心網路用自己的識別碼說話，中間的轉換是 SEAL server 的工作之一。** 例如[第 03 篇][seal-03]的資源調整程序中，VAL server 的請求以 VAL UE ID 或 VAL user ID 指明對象，NRM server 則要取得終端的 IP 位址才能向核心網路提出請求。

規格只在 §7.3 明確提到 VAL UE ID 可以使用 GPSI。其他識別碼如何對應到 SUPI、GPSI 或內部群組識別碼，要看各服務的程序。

## 6. 回到「一定要用 SEAL 嗎」

10 月 1 日會議的第 1 題問：宣稱自己是 VAL 是否帶有使用 SEAL 的義務。從本系列已查到的內容整理如下：

- VAL client、VAL server 的定義沒有把「使用 SEAL」當成條件（§3.1）。
- SEAL 是一組各自獨立的服務，應用可以只用其中幾項。
- VAL server 甚至可以自己兼任某項 SEAL server 的角色，例如 NRM（§14.2.4.3）。
- 一旦 VAL server 與 SEAL server 有 API 互動，Stage 2 要求符合 CAPIF（§4.1.2-e）；Stage 3 的落差見[第 04 篇][seal-04]第 4 節。

會議當時的討論與尚未解開的部分，見[2026-10-01 會議議題與後續探索][meeting]第 1 節。

## 7. 提供 SEAL 服務要做到多少

第 6 節是從使用者的角度問。本節換成提供者的角度：自己實作一個 SEAL server，要做到多少才算數。

規格沒有「必須做滿哪些」的清單。能確定的只有下列四點。

| 問題 | 規格怎麼寫 | 依據 |
| --- | --- | --- |
| 要不要做齊所有服務 | 不用。每項服務各有自己的 server 與 client，服務之間透過 SEAL-X 互相使用 | TS 23.434 §6.2 |
| 第 4 章的需求是不是必做清單 | 不是（解讀，見下方）。條文的主詞是「The SEAL」，不是「the SEAL server」 | TS 23.434 §4.1.2、§4.2.2 |
| 一項服務裡要實作哪些 API | 沒有規定。TS 29.549 第 5 章只列出各服務的 API 與操作，整章沒有任何 shall support 或 may support 的句子 | TS 29.549 §5 |
| 一定要做到的 | API 的存取保護。使用 CAPIF 時，要支援 CAPIF 規定的功能與至少一種認證方式；不使用 CAPIF 時，要支援 TLS 雙向認證與 OAuth 2.0 | TS 29.549 §8.1、§9.2 |

最後一列在 Stage 2 寫成一律要符合 CAPIF（§4.1.2-f），和 Stage 3 的寫法有落差，見[第 04 篇][seal-04]第 4 節。

**解讀：** 第 4 章包含「The SEAL shall leverage the satellite connectivity」（§4.1.2-g），也包含只在電信網路內、只在網路外、兩者同時三種部署（§4.2.2）。這些不可能每個部署都做到，所以應該看成整份規格要涵蓋的範圍，不是每個實作的必做清單。

**解讀：** 程序裡的 shall 是條件式的。實作了某個程序，就要照該程序與資料模型的規定；沒有實作的程序，不受它的條文約束。AIMLE 的情況相同，見[應用層與核心網路的 ML 03][ml-03]第 2 節與第 6 節。綜合上面幾點，照標準方式提供某項服務的功能，就算提供了這項 SEAL 服務，而且可以只提供這項服務的一部分 API。

**未能確認：** 3GPP 是否有 SEAL 的符合性測試規格，本地沒有。各服務另外的協定規格（TS 24.5xx 系列）只有部分在本地，沒有逐一檢查。

## 規格來源

TS 23.434 V20.1.0：[§3.1][s3]、[§4.1.2 與 §4.2.2][s4]、[§5][s5]、[§6.2][s62]、[§7][s7]、[§8][s8]、[§14.2.4.3][s142]、[§14.3.3][s1433]。TS 29.549 V20.1.0：[§5][p5]、[§8][p8]、[§9][p9]。

[seal-01]: Release%2020%20SEAL%2001%20定位與通用模型.md
[seal-03]: Release%2020%20SEAL%2003%20與核心網路的互動.md
[seal-04]: Release%2020%20SEAL%2004%20服務%20API%20與%20CAPIF.md
[meeting]: 2026-10-01%20會議議題與後續探索.md
[ml-03]: 應用層與核心網路的%20ML%2003%20Daisy%20與%20AIMLE%20的對照.md
[s3]: ../../../specs/Rel-20/TS%2023.434/3%20Definitions,%20symbols%20and%20abbreviations.md
[s4]: ../../../specs/Rel-20/TS%2023.434/4%20Architectural%20requirements.md
[s5]: ../../../specs/Rel-20/TS%2023.434/5%20Involved%20business%20relationships.md
[s7]: ../../../specs/Rel-20/TS%2023.434/7%20Identities.md
[s8]: ../../../specs/Rel-20/TS%2023.434/8%20Application%20of%20functional%20model%20to%20deployments.md
[s142]: ../../../specs/Rel-20/TS%2023.434/14%20Network%20resource%20management/14.2%20Functional%20model%20for%20network%20resource%20management.md
[s1433]: ../../../specs/Rel-20/TS%2023.434/14%20Network%20resource%20management/14.3%20Procedures%20and%20information%20flows%20for%20network%20resource%20management/14.3.3%20Unicast%20resource%20management.md
[s62]: ../../../specs/Rel-20/TS%2023.434/6%20Generic%20functional%20model%20for%20SEAL%20services/6.2%20On-network%20functional%20model%20description.md
[p5]: ../../../specs/Rel-20/TS%2029.549/5%20Services%20offered%20by%20the%20SEAL%20servers/README.md
[p8]: ../../../specs/Rel-20/TS%2029.549/8%20Using%20Common%20API%20Framework.md
[p9]: ../../../specs/Rel-20/TS%2029.549/9%20Security.md
