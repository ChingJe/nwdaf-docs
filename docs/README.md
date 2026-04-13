# docs/ — 技術文件目錄

所有設計文件、需求分析、合規報告、程式碼詳解統一放在此目錄，依元件分類。

---

## 進度總覽

| 文件 | 說明 |
|------|------|
| [progress_report.md](progress_report.md) | 各 Phase 進度、技術決策、已知問題（Phase 11 + E2 完成） |

---

## subscription/ — EventSubscription API

| 文件 | 說明 |
|------|------|
| [subscription_requirements.md](subscription/subscription_requirements.md) | 完整功能需求（API 端點、資料結構、錯誤處理） |
| [notification_mechanism.md](subscription/notification_mechanism.md) | 通知機制規範研究（輸入參數、輸出格式、定時規則） |
| [comprehensive_compliance_report.md](subscription/comprehensive_compliance_report.md) | 3GPP TS 29.520 全面合規分析報告 |

---

## smf/ — SMF 資料蒐集

| 文件 | 說明 |
|------|------|
| [data_collection_requirements.md](smf/data_collection_requirements.md) | SMF/UPF 資料蒐集需求（必要欄位、優先級、時間戳記） |
| [smf_event_data_collection_requirements.md](smf/smf_event_data_collection_requirements.md) | SMF Event Exposure 資料蒐集對應分析（TS 29.508 / TS 29.564） |
| [smf_resource_management.md](smf/smf_resource_management.md) | SMF 訂閱資源管理設計（引用計數、多 consumer 去重） |
| [collector_implementation_report.md](smf/collector_implementation_report.md) | Collector 模組實作報告 |
| [smf_upf_data_collection_compliance_report.md](smf/smf_upf_data_collection_compliance_report.md) | SMF/UPF 資料蒐集合規報告（TS 29.508 / TS 29.564） |

---

## upf/ — UPF 通知

| 文件 | 說明 |
|------|------|
| [upf_ees_starttime_issues.md](upf/upf_ees_starttime_issues.md) | UPF EES startTime 欄位準確性問題報告 |
| [upf_ees_starttime_permanent_offset.md](upf/upf_ees_starttime_permanent_offset.md) | UPF EES startTime 永久偏差（76–77s）問題報告 |

---

## anlf/ — Analytics（推論與 UE Communication）

| 文件 | 說明 |
|------|------|
| [ue_communication_workflow.md](anlf/ue_communication_workflow.md) | UE Communication 訂閱 → 資料蒐集 → ML 推論 → 通知流程 |
| [inference_aggregation.md](anlf/inference_aggregation.md) | AnLF 推論聚合 pipeline（ring buffer → 對齊 → 聚合 → ML） |
| [data_query_logic.md](anlf/data_query_logic.md) | MongoDB time series query + in-memory ring buffer 查詢邏輯 |
| [ue_communication_compliance_report.md](anlf/ue_communication_compliance_report.md) | UE Communication 合規報告（輸入驗證、通知輸出、資料蒐集） |

---

## adrf/ — ADRF 整合

| 文件 | 說明 |
|------|------|
| [adrf_integration_design.md](adrf/adrf_integration_design.md) | NWDAF × ADRF × Daisy 整合設計（E2 完成，E3 待實作） |
| [phase_e2_implementation_plan.md](adrf/phase_e2_implementation_plan.md) | Phase E2 StorageRequest 實作報告（✅ 完成，commit 5dbbb13） |

---

## daisy/ — Daisy FL 整合

| 文件 | 說明 |
|------|------|
| [nwdaf-daisy-improvement-plan.md](daisy/nwdaf-daisy-improvement-plan.md) | NWDAF × Daisy 整合改進計畫（Phase A–D，含 ADRF 資料路徑） |
| [daisy_async_callback_implementation.md](daisy/daisy_async_callback_implementation.md) | Daisy 非同步 callback 需求（NWDAF 端 ✅，Daisy 端待確認） |
| [daisy_model_bundle_requirements.md](daisy/daisy_model_bundle_requirements.md) | Daisy model bundle 需新增 `Model` 別名供 ML Service 動態載入 |

---

## architecture/ — 全局架構

| 文件 | 說明 |
|------|------|
| [code_walkthrough.md](architecture/code_walkthrough.md) | 所有原始碼逐檔說明（package 職責、進入點、config、logger） |
| [architecture_diagram.md](architecture/architecture_diagram.md) | 元件架構圖（cmd → service → nwdaf 啟動流程） |
| [traffic_data_refactoring.md](architecture/traffic_data_refactoring.md) | TrafficData 儲存架構重構報告（nested map + SmfSubscriptionResource） |

---

## distributed/ — 分散式 NWDAF

| 文件 | 說明 |
|------|------|
| [distributed_nwdaf_overview.md](distributed/distributed_nwdaf_overview.md) | 分散式 NWDAF 部署概覽摘要（懶人包：情境說明、運行流程、雙 SMF 問題） |
| [agg_routing_without_aoi.md](distributed/agg_routing_without_aoi.md) | 多 NWDAF 分析聚合規格分析（§6.1A.3.2 無 AoI routing 推導、group ID 處理、AGG 轉發決策） |
| [agg_routing_with_aoi.md](distributed/agg_routing_with_aoi.md) | AoI 為基礎的 AGG Routing（§6.1A.3.1 有 AoI 方案，單一 SMF + 兩個 TAI 拓撲） |

---

## learning/ — 教學參考

| 文件 | 說明 |
|------|------|
| [go_basics.md](learning/go_basics.md) | Go 語言基礎（struct、interface、pointer、error handling） |
| [gin_guide.md](learning/gin_guide.md) | Gin 框架使用指南（router、context、請求處理） |
