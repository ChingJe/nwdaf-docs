# Hierarchical FL 論文實驗

狀態：依新版論文附錄 B 對齊的文件修訂已確認；E0–E2b 五 seed 實驗尚未執行。

本分類以新版論文的候選協定設計為後續實作目標；現行 `x-flTopology`／`x-flTopologyReport` 是既有程式契約，不等同於論文附錄 B。既有 MNIST／CIFAR-10 單次配對仍是歷史觀測，不是 E0–E2b 五 seed 實驗結果。

- [E0–E2b 實驗情境與 Testbed 對照](./Hierarchical%20FL%20E0-E2b%20Experiments%20and%20Testbed%20Context.md)：整理論文術語、部署對照、各情境與證據需求。
- [後續實驗能力與證據盤點](./Hierarchical%20FL%20Experiment%20Capability%20and%20Evidence%20Inventory.md)：盤點現有 Go／PyMTLF 能力、訂閱識別修正與仍需補足的紀錄。
- [論文實驗能力設計討論](./Hierarchical%20FL%20Experiment%20Capability%20Design.md)：集中記錄各項初步設計、已確認方向與待決策問題。
- [E1 傳遞 Schema 兩版對照](./Hierarchical%20FL%20E1%20Wire%20Schema%20Flow%20Comparison.md)：以同一接手流程對照既有程式契約與新版論文附錄 B；後者是後續實作目標，尚非現行 wire format。
- [老師的 paper-first implementation audit](./HFL-NWDAF-paper-first-implementation-audit.md)：收錄老師提供的審查內容，實驗範圍已依本分類調整；其 repository 缺件判斷以受檢的 PyMTLF 為範圍，不直接代表整個 workspace 或 testbed。
