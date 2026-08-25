# NWDAF Testbed 整合進度摘要

最後更新：2026-08-25

目前狀態：新版階層式聯邦學習組件已加入 testbed integration branch；testbed 架構調整與
實驗驗證尚未開始

## 摘要

目前使用 `5G_NWDAF_Infrastructure` 作為新版 NWDAF full-core testbed 的整合 repository。
Testbed 已從 `main@7d0a36c` 建立 `feat/r18-hierarchical-federated-learning`，並更新下列組件：

| 組件 | Revision | Branch |
| --- | --- | --- |
| Go NWDAF | `3279891` | `feat/r18-hierarchical-federated-learning` |
| PyAnLF | `6a4d94a` | `feat/r18-federated-learning` |
| PyMTLF | `e9aa223` | `feat/r18-hierarchical-federated-learning` |

Parent repository 的 submodule gitlinks、readable component lock 與 ML container source identity
已同步。Repository `make test` 與唯讀 preflight 均通過，16 個 component revisions 一致；這些
結果只證明 source baseline 與既有 repository contracts 一致，未執行 VM、container 或 business
E2E。

## 對組件開發的意義

- Testbed 後續驗證會使用上述精確 revisions；若 NWDAF、PyAnLF 或 PyMTLF 再有必要修正，應先在
  各自 repository 完成獨立 review、verification 與 commit，再更新 testbed pin。
- 現有 testbed 仍以 three-NWDAF flat federated learning topology 與五個 ML containers 為基線，
  尚未配置 HFL Root、Branch 與 Leaves。
- 現階段不可宣稱 HFL 已通過 testbed、full-core、跨 VM、UERANSIM／UPF data production、failure、
  timeout、restart 或 publication／cutover validation。
- 組件端已完成的 local multi-process HFL evidence 仍是 production behavior baseline；testbed
  adaptation 不應反向改變已確認的 NWDAF／PyMTLF／PyAnLF ownership 或 contracts。

## 後續階段

1. 盤點現有 three-NWDAF／three-VM／five-container deployment 與 HFL topology requirements。
2. 設計 Root／Branch／Leaf placement、NF capabilities、process composition、network／port、artifact
   origins、ADRF 與資料來源。
3. 調整 testbed config、renderer、lifecycle、observability 與 validation tooling。
4. 確認實驗 topology、trigger、stimulus、aggregation、failure／timeout／restart 與 evidence matrix。
5. 以 testbed 精確 revisions 執行驗證；完成前維持 `Testbed Validation Pending`。

本頁只提供 NWDAF 組件開發所需的 testbed 狀態，不保存 testbed 操作步驟、詳細實作歷史、環境
設定、run records 或實驗 evidence。
