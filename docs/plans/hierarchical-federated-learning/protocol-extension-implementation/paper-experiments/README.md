# Hierarchical FL 論文實驗

狀態：Slice 1 已完成本地實作；第 2 項起沿用既有候選 schema 的文件調整待使用者確認；E0–E2b 五 seed 實驗尚未執行。

本分類後續以既有候選 `x-flTopology`／`x-flTopologyReport` schema 為實作基準；新版論文附錄 B 是另一種設計，供差異比較與論文敘述對齊，不是本批 wire migration 目標。既有 MNIST／CIFAR-10 單次配對仍是歷史觀測，不是 E0–E2b 五 seed 實驗結果。

- [E0–E2b 實驗情境與 Testbed 對照](./Hierarchical%20FL%20E0-E2b%20Experiments%20and%20Testbed%20Context.md)：整理論文術語、部署對照、各情境與證據需求。
- [後續實驗能力與證據盤點](./Hierarchical%20FL%20Experiment%20Capability%20and%20Evidence%20Inventory.md)：盤點現有 Go／PyMTLF 能力、訂閱識別修正與仍需補足的紀錄。
- [論文實驗能力設計討論](./Hierarchical%20FL%20Experiment%20Capability%20Design.md)：集中記錄各項初步設計、已確認方向與待決策問題。
- [實作 Slice 詳細計畫](./slices/README.md)：逐一規劃本批論文實驗能力；Slice 1 的本地實作已完成，testbed 驗證仍待進行。
- [E1 傳遞 Schema 兩版對照](./Hierarchical%20FL%20E1%20Wire%20Schema%20Flow%20Comparison.md)：以同一接手流程對照既有候選 schema 與新版論文附錄 B；前者是目前實作基準，後者不是現行 wire format 或本批實作指令。
- [老師的 paper-first implementation audit](./HFL-NWDAF-paper-first-implementation-audit.md)：保留老師當時的審查原文；其中遷移至附錄 B 的建議已由後續會議決定取代。其 repository 缺件判斷以受檢的 PyMTLF 為範圍，不直接代表整個 workspace 或 testbed。
