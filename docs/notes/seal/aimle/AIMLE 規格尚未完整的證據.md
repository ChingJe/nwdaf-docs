# AIMLE 規格尚未完整的證據

本筆記整理 AIMLE 規格尚未定義完整的證據，以及從證據推出的判斷。3GPP 刻意不定義的部分，例如聚合演算法、訓練邏輯，和 NWDAF 的情況相同，不在本筆記範圍。

依據：

- AIMLE 本身的規格：TS 23.482 V20.3.0、TS 29.482 V20.1.0、TS 24.560 V20.1.0。
- 對照用的 NWDAF 規格：TS 23.288 V18.13.0、TS 29.520 V18.14.0。
- OpenAPI 的最新狀態：2026-10-15 從 3GPP forge 的 REL-19、REL-20 分支取得。

標示「解讀」的段落是推論，不是規格原文。

## 結論

1. **Stage 3 凍結不到一年，HFL 相關的 API 每一季都還在更正。**
2. **規格裡還留著 Editor's Note、FFS、TBD，也有寫明「this release」不支援的項目。**
3. **Stage 2 和 Stage 3 對不上。** 最明顯的是訓練結果：Stage 2 規定要回報模型參數，Stage 3 的欄位卻只能放評估指標，從 client 到 VAL server 都沒有能指出模型在哪裡的欄位。

## 1. 規格的層次

| 層次 | 定義什麼 | AIMLE 的規格 | 工作組 |
| --- | --- | --- | --- |
| Stage 2 | 架構、流程、每個訊息帶哪些資訊 | TS 23.482 | SA6 |
| Stage 3 | API 路徑、HTTP 方法、欄位名稱與型別（OpenAPI） | TS 24.560：AIMLE client 與 server 之間 | CT1 |
| | | TS 29.482：AIMLE server 對 VAL server、其他 AIMLE server、ML repository | CT3 |

實作時依據的是 Stage 3。Stage 2 寫「要帶什麼」，Stage 3 才決定「實際放在哪個欄位」。

## 2. Stage 3 很新，而且一直在更正

**時間線：**

| 時間 | 事件 |
| --- | --- |
| 2024-12 | TS 23.482（Stage 2）凍結為 V19.0.0 |
| 2025-10 | TS 24.560 加入 HFL 訓練服務的說明與 API（CT1#157） |
| 2025-11 | TS 24.560 加入 HFL 訓練的 OpenAPI（CT1#158） |
| 2025-12 | TS 24.560、TS 29.482 凍結為 V19.0.0 |

HFL 訓練 API 從加入到凍結大約兩個月。

規格凍結後，任何修改都要提 CR（Change Request），並標明類別。F 類是更正錯誤（essential correction），B 類是新增功能，C 類是修改功能，D 類是編輯上的修改。

**凍結之後和 HFL 訓練有關的 F 類 CR：**

| 時間 | 規格 | CR | 標題 |
| --- | --- | --- | --- |
| 2026-01 | TS 23.482 | 0056 | HFL training completion |
| 2026-03 | TS 29.482 | 0016 | Corrections on the AIMLES_MLModelTraining API |
| 2026-03 | TS 24.560 | 0005 | Corrections on the outcome of the ML model training |
| 2026-03 | TS 24.560 | 0011 | Corrections on the uri of the HFL training subscription |
| 2026-06 | TS 29.482 | 0053 | Correction of AIMLES_MLModelTraining_Notify Initiated by |
| 2026-06 | TS 29.482 | 0054 | Corrections to the apiName component of the AIMLES_MLModelTraining API URI |
| 2026-06 | TS 24.560 | 0018 | Corrections on the Aimlec_HFLTraining API |
| 2026-09 | TS 23.482 | 0135 | Correction to the information flow of the ML model training request |
| 2026-09 | TS 24.560 | 0033 | Correction to Aimlec HFL Training API |

**推理：** F 類 CR 代表規格寫錯了或漏寫了，不是新增功能。HFL 是 AIMLE 的主要流程之一，但從 Stage 2 到兩份 Stage 3，凍結後每一季都還有更正。可見這部分還沒有穩定下來。

## 3. 規格自己標出的未完成項目

3GPP 用 Editor's Note、FFS（for further study）、TBD 標記還沒決定的內容。條文寫「not supported in this release」或「out of scope for this release」，代表這一版刻意不處理，留給之後的版本。

| 位置 | 原文 |
| --- | --- |
| TS 23.482 §8.18.2 NOTE | 「How AIMLE Server coordinates the selected VFL members to perform VFL training is out of scope for this release.」 |
| TS 23.482 §8.18.3 | 「Editor's Note: Details about sample binding criteria is FFS.」 |
| TS 23.482 §8.31 | 「Editor's Note: How the AIMLE server can trigger A-DCCF data collection service will be further discussed during normative phase.」 |
| TS 23.482 §8.34 | 「Editor's Note: The usage of ML repository to discover the AIMLE servers is FFS.」 |
| TS 23.482 §5.2.2a NOTE | 「any cross-operator ML repository interaction is not supported in this release」 |
| TS 29.482 §5.2.20 | 錯誤處理寫「as specified in clause TBD」 |
| TS 29.482 §6.2.5 | 「Editor's Note: The definition of type SvrDscResp is FFS.」 |
| TS 24.560 §5.1 | `Aimles_MLModelEvalInfo` 的 OpenAPI 檔名與版本都寫「TBD」 |
| TS 24.560 §6.15.3 | 「Editor's note: … The figure depicting the resource URIs structure is FFS.」 |

另外還有編輯上的痕跡：TS 24.560 §5.1 把 `Aimles_IntermediateNode` 的 OpenAPI 檔名寫成「TS25560_…」，規格編號打錯了。

**推理：** 這些標記都是規格自己承認「還沒定」。其中 VFL 的訓練過程整個不在這一版的範圍內，所以 AIMLE 的 VFL 目前只能做到選成員和對齊樣本。

## 4. Stage 2 和 Stage 3 對不上

### 4.1 訓練結果沒有模型

一次 HFL 有兩個通知會帶訓練結果：

- 每一輪 client 回報給 AIMLE server：TS 24.560 `Aimlec_HFLTraining_Notify`，對應 Stage 2 §8.12 步驟 8。
- AIMLE server 通知 VAL server：TS 29.482 `AIMLES_MLModelTraining` 的通知，對應 Stage 2 §8.3 步驟 4（也就是 §8.12 步驟 10）。

| | client → AIMLE server | AIMLE server → VAL server |
| --- | --- | --- |
| Stage 2 的規定 | Table 8.12.3.3-1：HFL training output，必填（M），「ML model parameters from HFL training」 | Table 8.3.3.3-1：Training output，「Output of training, e.g., ML model parameters for the training」 |
| Stage 3 的欄位 | `HflTrngNotify.hflTrngOut` | `MlModelTrainNotif.trainOut` |
| Stage 3 欄位的說明 | 「Identifies the ML model parameters from the HFL training」 | 型別說明照抄 Stage 2：「Represents the output of training, e.g., ML model parameters for the training」 |
| Stage 3 欄位的型別 | `PerfParams` | `PerfParams` |
| `PerfParams` 實際的內容 | accuracy、precision、recall、F1、MSE、MAE | 同左 |

`PerfParams` 定義在 TS 29.482，TS 24.560 直接引用它，所以兩段的問題其實是同一個。2026-10-15 從 forge 取得的最新 OpenAPI 仍然是這樣，還沒有修正。

**對照同一套規格裡其他傳模型的地方，用的都是位置參照：**

- **AIMLE 的下發方向**：`HflTrngSub.aimlMdlInfo` 的型別是 `MlModelInfo`，帶模型識別碼和位置 `mlModelLoc`。
- **NWDAF 的 HFL 回報**：TS 29.520 的 `Nnwdaf_MLModelTraining_Notify` 帶 `mlFile` 或 `mLFileAddr`（URL 或 FQDN）。

**解讀：** 3GPP 的慣例是用位置參照傳模型，不在訊息裡夾帶權重。AIMLE 下發時遵守了這個慣例，回報時卻沒有任何放位置的欄位。欄位說明寫「模型參數」，型別卻是效能指標，最可能的原因是 Stage 3 把「ML model parameters」理解成了效能方面的參數。也可能是刻意設計，例如 client 把模型放到約定的位置，再由 server 去取，但規格裡沒有任何文字這樣寫。

**影響：** 照 Stage 3 實作時，從 client 回報到通知 VAL server，整條路上都沒有能指出訓練後模型在哪裡的欄位。VAL server 收到的通知只帶 `trainingID`，沒有模型識別碼，它要怎麼取得最終模型，Stage 3 也沒有交代。

### 4.2 引用了空的章節

TS 23.482 §8.12 步驟 2 寫「The AIMLE server retrieves the indicated ML model using the ML model retrieval procedure as described in clause 8.1」。

但 §8.1 只有標題「General」，沒有任何內容。名稱叫 ML model retrieval 的是 §8.2，而 §8.2 描述的是 client 或 VAL server 向 AIMLE server 取模型，方向剛好相反。所以 AIMLE server 從哪裡取得要訓練的模型，條文沒有交代。

### 4.3 Release 20 新增的 API 還沒有 Stage 3

TS 23.482 第 9 章在 Release 20 新增了一批 API。下列幾個，用 API 名稱和關鍵字（hierarchical、feedback、edge、collaborative、federated AIML）搜尋 TS 29.482 和 TS 24.560 的全文，都找不到對應：

| Stage 2 條號 | API | Stage 2 相關的 CR |
| --- | --- | --- |
| §9.2.19 | AIMLE server registration（`Aimles_HierarchicalServer`） | 2026-03（CR0065） |
| §9.2.20 | Hierarchical ML model training | 2026-03（CR0066） |
| §9.2.26 | Federated AIML service config | 2026-07（CR0103） |
| §9.2.28 | ML model inference service performance feedback | 2026-07（CR0127） |
| §9.2.29 | Edge ML model inference | 2026-07（CR0128） |
| §9.4.9 | ML model collaborative inference | 2026-07（CR0130） |

同一批裡也有已經補上 Stage 3 的：

- §9.2.25 intermediate node discovery：TS 24.560 的 `Aimles_IntermediateNode`，2026-09。
- §9.2.27 server change：TS 29.482 與 TS 24.560 的 server change 服務。
- §9.2.30 ML model inference：TS 29.482 的 `AIMLES_MLModelInference`。

**推理：** Stage 3 通常落後 Stage 2 幾季，所以這一項本身不算錯誤，而是說明 Release 20 的 AIMLE 還在擴充中。這裡只用名稱和關鍵字比對，如果 Stage 3 用了完全不同的名稱，可能會漏掉。
