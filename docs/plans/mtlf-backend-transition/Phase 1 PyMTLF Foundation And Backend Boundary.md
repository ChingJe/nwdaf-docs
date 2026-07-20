# Phase 1 PyMTLF Foundation And Backend Boundary

Date: 2026-07-21

Status: Verified historical foundation; later architecture boundaries are superseded by the canonical parent plan

Parent plan:

- `nwdaf-docs/docs/plans/mtlf-backend-transition/MTLF Backend Transition Plan.md`

---

## 1. Purpose Of This Record

這份文件記錄 2026-07-17 已完成並驗證的 Phase 1 foundation。team architecture review之後，後續
transition已改採 feature-oriented plan，並大幅簡化 private contract與 distributed state design。

因此本文件不再定義 future accuracy、dataset、generation或 model-apply contract。新的 canonical
architecture與 phase順序只以 parent plan為準。

---

## 2. Implemented Baseline

Phase 1建立了：

### PyMTLF

- 獨立 repository與 `src/py_mtlf/` packaging
- FastAPI、Pydantic、Uvicorn、typed config與 logging
- `127.0.0.1:9092` local default
- liveness/readiness與 graceful startup/shutdown
- content-addressed artifact repository與 immutable artifact GET
- SQLite generation journal與 startup reconciliation primitives
- unit/API tests與 Ruff
- 整個 runtime `data/`由 `.gitignore`排除

### PyAnLF

- default port由 `9090`移到 `127.0.0.1:9093`
- remote artifact origin allowlist與 `300 seconds` download timeout
- compressed、extracted、single-file與entry-count limits
- redirect rejection與 safe extraction exact-file validation
- archive digest/size與 cache integrity hardening
- PyAnLF-owned deterministic bundle fixture builder
- Ruff baseline與 behavior-preserving lint remediation

### NWDAF

- disabled-by-default `mtlfBackend` config與 validation
- `internal/mtlf/client/BackendClient` readiness-only transport boundary
- typed backend request errors、timeout/cancellation cause preservation
- Go naming只使用 MTLF backend語意，不含 Python implementation name
- 未將新 backend接入 production MTLF policy或 training path

---

## 3. Baseline Commits

已驗證並分 repository提交：

1. `PyMTLF@8c55a965352ff3a70c31adce4a2e8640d8c8b4a8`
   - `feat: establish MTLF backend foundation`
2. `PyAnLF@fac8fab52a116e413a36abbb03e69c81b954e4ef`
   - `feat(model): harden model artifact consumption`
3. `NWDAF@4b5e114f25b718003e3d248551f2d52fa86a6cf6`
   - `feat(mtlf): add backend readiness boundary`
4. `nwdaf-docs@a6b28b3e8798f1d1d0830fa5ee9df2d14dc5b5a0`
   - `docs(mtlf): record verified backend foundation baseline`

這些 commits是 implementation baseline，不代表舊 target architecture仍有效。

---

## 4. Foundation Retained By The New Plan

後續 feature phases直接保留並擴充：

1. PyMTLF repository、package、config、logging與 lifecycle
2. PyAnLF與 PyMTLF目前的 local port分配
3. Go `mtlfBackend` config與 backend-neutral naming
4. backend readiness endpoint與 client transport基礎
5. MTLF backend content-addressed artifact repository
6. PyAnLF download、cache、archive與safe-extraction hardening
7. PyAnLF/PyMTLF Ruff與既有 test discipline
8. 各 repository獨立 commit與 verification boundary

artifact repository仍支援標準 `mLFileAddr.mLModelUrl` flow。digest、ETag與 cache metadata可以作為
artifact implementation detail，不需要再形成大型 private provision contract。

---

## 5. Superseded Design

team review已取消以下舊方向：

1. Go-owned ADRF/Mongo training data provider abstraction
2. PyMTLF向 Go提出 custom dataset request
3. Go fetch ADRF records後以 normalized chunks push給 PyMTLF
4. provider attempt ID、chunk acknowledgement、completion與 fallback protocol
5. ADRF與 Mongo輸出共用 Go-normalized observation contract
6. 禁止 PyMTLF直接連 ADRF或 MongoDB
7. custom accuracy report envelope優先於 `MLModelMonitorNotify`
8. custom `ModelReady`與 parallel model provision DTO
9. `baseGeneration`／`targetGeneration` compare-and-swap contract
10. `APPLIED`／`FAILED`／`STALE`／`NO_MATCH`／`CONFLICT` apply-result state machine
11. Go/PyMTLF active-generation reconciliation API
12. 以 repository為單位拆分 AnLF與MTLF改動

後續不應再從舊 git history或本文件恢復上述 contract。

---

## 6. Reclassified Existing Code

SQLite generation journal與reconciliation primitives已實作且通過Phase 1測試，但team後續確認它們只服務
已取消的generation/apply protocol，沒有新的production consumer。現在的決策為：

- 不再是 Go/PyMTLF wire contract
- 不再是第一版 model provision的必要 state machine
- 在Phase 2移除journal、reconciliation、專用models、config與tests
- 不保留為readiness diagnostic或future training placeholder
- 未來若local training需要job persistence，依實際job lifecycle重新設計小型internal store

Phase 1 baseline commit與historical verification evidence仍保持不變；Phase 2會以新的PyMTLF commit執行刪除。
後續不得從Phase 1 git history恢復這套state machine作為新wire或job contract。

---

## 7. Current Canonical Boundary

2026-07-21 review後，Phase 1 foundation適用的current target boundary為：

```text
Go
  - polls AnLF/MTLF backend readiness
  - performs standard NRF/SMF/UPF/ADRF control communication requested by backends
  - owns standard SBI, routing and external resource URIs
  - syncs both backends after readiness

PyAnLF
  - owns analytics runtime, data-collection intent and SMF candidate selection
  - receives SMF/UPF notifications directly
  - writes raw Mongo records and prepares standard ADRF storage requests

PyMTLF
  - owns accuracy policy, source selection, direct ADRF/Mongo retrieval,
    local training and standard-shaped model provision behavior
```

PyMTLF收到 `FetchInstruction`後直接向 ADRF fetch；Mongo mode則使用 read-only credential直接 query。
Go只建立/清理 ADRF retrieval subscription與 forwarding callback instruction，不代理 dataset bytes。

本文件不固定sync、collection或storage contract；exact behavior以parent plan及最新Phase 2/Phase 3文件為準。

---

## 8. Historical Verification Evidence

final Phase 1 baseline verification包含：

### PyMTLF

- `uv run pytest -q`: `62 passed`
- `uv run ruff check .`: passed

### PyAnLF

- `uv run pytest -q`: `141 passed, 1 skipped`
- `uv run ruff check .`: passed

### NWDAF

- affected package tests與 race tests: passed
- `make test`: passed
- `make build`: passed
- `make lint`: passed

### Cross-process

- Go live readiness request to PyMTLF passed
- PyAnLF HTTP download、digest/size validation與bundle load passed
- PyMTLF graceful shutdown/restart與artifact persistence passed
- repository hygiene與 no-Go-Python-implementation-naming checks passed

這些 evidence只證明 foundation commits的品質；新 feature-oriented flow仍須依 parent plan重新驗證。

---

## 9. Historical Handoff

Phase 1後續工作已進入Phase 2與Phase 3。原始entry conditions的結果為：

1. backend polling與app lifecycle已在Phase 2完成。
2. PyMTLF SQLite generation journal/reconciliation已在Phase 2移除。
3. 原始MTLF-only available-source handshake後續被unified backend sync取代。
4. Release 18 OpenAPI gap由每個feature phase繼續audit。
5. Phase 3依新架構同時調整NWDAF、PyAnLF及小範圍PyMTLF sync。
6. 不建立沒有production consumer的broad interface或duplicated JSON contract fixture。

Phase 1不再新增功能。後續變更應進入新的 feature phase與獨立 commit。
