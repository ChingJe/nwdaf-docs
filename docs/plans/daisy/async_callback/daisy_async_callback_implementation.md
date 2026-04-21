# Daisy 非同步 Callback 實作需求

**狀態**：NWDAF 端 ✅ 完成；Daisy 端待確認
**對象**：Daisy 開發者
**背景**：NWDAF 已完成 async callback 接收端實作，本文說明 Daisy 這邊需要對應補充的實作內容。

---

## 問題現況

目前 `server_api_handler.py` 的 `publish_task` 邏輯如下：

```python
@self.app.route("/publish_task", methods=["POST"])
def publish_task():
    js = request.get_json()
    _, report = self._task_manager.receive_task(task_config=js)  # 阻塞直到訓練完成
    self.get_metrics(report.config[TID])
    return js, 200
```

`receive_task` 會一直阻塞到整個 FL 訓練結束（可能數分鐘），期間 NWDAF 的 goroutine 被卡住，無法處理其他請求。

---

## 需要的改動

### 1. `publish_task` 改為非同步模式

改為立即回 `202 Accepted`，在 background thread 執行訓練，訓練完成後主動 POST callback 到 payload 中的 `CALLBACK_URL`。

```python
import threading
import requests as req_lib

@self.app.route("/publish_task", methods=["POST"])
def publish_task():
    js = request.get_json()
    callback_url = js.get("CALLBACK_URL", "")

    def run_and_notify():
        try:
            _, report = self._task_manager.receive_task(task_config=js)
            self.get_metrics(report.config[TID])
            payload = {
                "task_id": js.get(TID, ""),
                "model_url": js.get("MODEL_PATH", ""),
                "status": "success",
            }
        except Exception as e:
            payload = {
                "task_id": js.get(TID, ""),
                "status": "failure",
                "error": str(e),
            }
        if callback_url:
            try:
                req_lib.post(callback_url, json=payload, timeout=10)
            except Exception as e:
                log(WARNING, f"Callback to {callback_url} failed: {e}")

    t = threading.Thread(target=run_and_notify, daemon=True)
    t.start()
    return {"accepted": True, "task_id": js.get(TID, "")}, 202
```

---

### 2. Flask 啟動時加 `threaded=True`

Flask 預設 single-thread，background thread 執行訓練期間需要能同時處理其他請求（例如 `/metrics`）。

在 `ServerListener.run()` 加上：

```python
def run(self):
    self.app.run(host=self._ip, port=self._port, threaded=True)
```

---

## NWDAF 送出的 task payload 格式

```json
{
  "TID": "7f988bc8-0e7d-44ba-ab99-42e2477c5419",
  "NUM_ROUNDS": 2,
  "EVALUATE_INTERVAL": 1,
  "EVALUATE_INIT_MODEL_MASTER": true,
  "MIN_WAITING_TIME_MASTER": 5,
  "MODEL_PATH": "model/model.npy",
  "CALLBACK_URL": "http://127.0.0.1:8080/mtlf/training-complete"
}
```

- `TID`：每次都是新產生的 UUID，用來配對 callback
- `CALLBACK_URL`：NWDAF 的 callback endpoint，Daisy 訓練完後 POST 到這裡
- 其餘欄位沿用現有 `task.json` 內容

---

## Daisy 送回的 callback payload 格式

POST 到 `CALLBACK_URL`，body 為 JSON：

**成功時：**
```json
{
  "task_id": "7f988bc8-0e7d-44ba-ab99-42e2477c5419",
  "model_url": "model/model.npy",
  "status": "success"
}
```

**失敗時：**
```json
{
  "task_id": "7f988bc8-0e7d-44ba-ab99-42e2477c5419",
  "status": "failure",
  "error": "some error message"
}
```

欄位說明：

| 欄位 | 必填 | 說明 |
|------|------|------|
| `task_id` | ✅ | 必須和 NWDAF 送出的 `TID` 相同，NWDAF 用這個配對 in-flight 記錄 |
| `status` | ✅ | `"success"` 或 `"failure"` |
| `model_url` | success 時必填 | 訓練完成的模型路徑，NWDAF 會傳給 ML Service 載入 |
| `error` | failure 時選填 | 錯誤描述 |

---

## 目前的 `model_url`

目前填入 `MODEL_PATH`（例如 `model/model.npy`）即可，ML Service 會從本地路徑讀取。

未來若實作模型 HTTP serving（tar.gz），`model_url` 改為 HTTP URL，ML Service 那邊也會對應更新，這個階段不需要處理。

---

## 改動範圍摘要

| 檔案 | 改動內容 |
|------|---------|
| `daisyfl/master/server_api_handler.py` | `publish_task` 改為 async（202 + background thread + callback），`run()` 加 `threaded=True` |

其他 example 檔案（`master.py`、`task.json`、`client.py` 等）不需要改動。

---

## 測試方式

用 curl 直接驗證新行為：

```bash
# 預期立即回 202
curl -s -o /dev/null -w "%{http_code}" \
  -X POST http://127.0.0.1:9887/publish_task \
  -H "Content-Type: application/json" \
  -d '{"TID":"test-001","MODEL_PATH":"model/model.npy","CALLBACK_URL":"http://127.0.0.1:8080/mtlf/training-complete","NUM_ROUNDS":2}'
# 預期輸出: 202
```

NWDAF 這邊也提供了 `test/fake_daisy/fake_daisy_server.py`，可以反向參考其 callback 實作方式。
