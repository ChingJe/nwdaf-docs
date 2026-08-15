# Hierarchical NWDAF Federated Learning 文件索引

## 1. 文件目的

本目錄重新整理 **Hierarchical NWDAF Federated Learning** 的通用設計。
討論重點不再從特定 Analytics、Model Provision 或 Model Monitor 情境往回建立
FL topology，而是先回答以下核心問題：

1. FL Server 如何透過標準機制感知環境中的 NWDAF 與其 FL 能力？
2. FL Server 如何選擇同時具備 FL Server 與 FL Client 能力的 NWDAF 作為 Branch？
3. 如何使用多個標準 preparation procedures 逐層建立 topology？
4. Branch 如何非同步回報實際準備成功與失敗的 Leaf participants？
5. Root 如何依完整或部分成功的結果，接受、補選或重建 topology？

目前主文件為：

- [Proposal 初稿](proposal_draft.md)

先前以 Analytics Aggregator、Model Provision、Model Monitor 與特定 UE
Communication scenario 為主線的版本，保留在：

- [Previous draft](../hierarchical-federated-learning-previous-draft/README.md)

## 2. 目前核心定位

新的設計將問題分成三層：

```text
Capability discovery
        ↓
Hierarchical topology preparation
        ↓
FL execution strategy
```

其中：

- NRF registration／discovery 提供候選 NWDAF 與標準 FL capability。
- Preparation 確認候選者是否能參與本次 FL，並逐層建立 topology。
- Model file 的 serialization format 不由 3GPP 指定；本設計在同 vendor 前提下，
  以仍含有效模型的 bundle 附加 hierarchy manifest。這是 vendor-specific contract，
  不是標準定義的 hierarchy 欄位。
- 本設計希望透過 Notify 中的 model bundle URL 回傳 per-Leaf preparation result。
  這不是 `mLModelUrl` 最自然的欄位語意，但規格沒有明文禁止在有效模型 bundle 中
  附加同 vendor metadata；其內容與解析方式仍須標示為 vendor-specific contract。
- Root 負責 topology planning；Branch 負責執行 Leaf preparation 並回報結果。
- 完整成功與部分成功都是可選 policy，不在框架中寫死。

## 3. 證據分類

文件會明確區分：

| 分類 | 定義 |
| --- | --- |
| **3GPP-defined** | 規格直接定義的 capability、欄位、service operation 或 procedure |
| **Implemented baseline** | 現有 NWDAF／PyMTLF 已完成並驗證的行為 |
| **Proposed design** | 以標準介面組合出的 hierarchical topology establishment 方法 |
| **Vendor-specific contract** | 透過標準 model file address 承載，但內容格式由同 vendor 實作約定 |
| **Open decision** | 尚待比較或選擇的 policy 與 lifecycle 細節 |

## 4. 文件狀態

本目錄是新的討論主線。現階段先收斂 topology establishment，不先綁定：

- 誰觸發 FL。
- Model degradation 或 Model Monitor。
- Model Provision chain。
- 特定 Analytics Consumer 或 UE scenario。
- 固定的 participant admission policy。
- 固定的 aggregation、wait 或 async strategy。
