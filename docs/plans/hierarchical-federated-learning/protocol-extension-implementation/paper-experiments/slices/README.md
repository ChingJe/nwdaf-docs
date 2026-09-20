# 論文實驗能力 Slice 盤點與計畫

本分類放本批論文實驗能力的證據盤點與實作詳細計畫；不沿用上層 protocol extension 已完成的 Slice 1–7 編號語意。各文件的盤點與實作狀態以文件本身為準。

- [Slice 1 — ML Model Training 訂閱資源識別與生命週期](./Slice%201%20ML%20Model%20Training%20Subscription%20Resource%20Identity%20and%20Lifecycle%20Detailed%20Plan.md)：讓接收端 Go／PyMTLF 共用正式資源 ID，讓發起端以對端資源 ID 管理訂閱；callback ID 保留於正式路由供通知查找，不另建索引表。本地實作與審查已確認，正式多節點驗證待執行。
- [Slice 2 — 逐節點協定與拓樸證據實作計畫](./Slice%202%20Per-Node%20Protocol%20and%20Topology%20Evidence%20Detailed%20Plan.md)：本地程式修改、測試與原始紀錄已供使用者確認，待提交核准；部署腳本仍使用舊事件欄位，自動驗收與正式實驗待完成。
- [Slice 3 — 直接重掛 Leaf 與混合深度訓練](./Slice%203%20Direct%20Leaf%20Reparenting%20and%20Mixed-Depth%20Training%20Detailed%20Plan.md)：詳細計畫與候選 payload 欄位改名已確認，待實作；Go／PyMTLF 合約同步、實作與驗證尚未開始。

逐節點事件紀錄與 E2a／E2b mixed-depth 能力不屬於 Slice 1；後續沿用候選 `flTopology`／`flTopologyReport` 的遞迴語意，但實作仍需同步新欄位名稱。E2a／E2b 執行能力以 Slice 3 計畫為準，不預設遷移至論文附錄 B 的其他欄位。
