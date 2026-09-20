# 論文實驗能力 Slice 詳細計畫

本分類只放本批論文實驗能力的實作詳細計畫；不沿用上層 protocol extension 已完成的 Slice 1–7 編號語意。每份文件界定一個可獨立審查的工作單位，實作狀態以各文件為準。

- [Slice 1 — ML Model Training 訂閱資源識別與生命週期](./Slice%201%20ML%20Model%20Training%20Subscription%20Resource%20Identity%20and%20Lifecycle%20Detailed%20Plan.md)：讓接收端 Go／PyMTLF 共用正式資源 ID，讓發起端以對端資源 ID 管理訂閱；callback ID 保留於正式路由供通知查找，不另建索引表。本地實作與審查已確認，正式多節點驗證待執行。

後續逐節點事件紀錄與 E2a／E2b mixed-depth 能力不預先塞入本 slice；第 2 項起沿用既有 `x-flTopology`／`x-flTopologyReport` 候選 schema，待各自設計邊界確定後再規劃。不預設遷移至論文附錄 B 欄位。
