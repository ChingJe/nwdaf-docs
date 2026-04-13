# ADRF V0 系統設計與整合計畫 (Phase 2 LIVE DEMO)
## **0. 本版範圍**

### **0.1 In Scope**

1. `store` 接點：`POST /data-store-records`。
2. NWDAF 收到 UPF notify 後，將資料寫入 ADRF（payload 走 `NadrfDataStoreRecord.dataSub + dataNotif`）。
3. retrain / FL 路徑使用 `POST /data-retrieval-subscriptions` + `fetch` 模式（`consTrigNotif=true`）。
4. `RetrievalNotify` callback 設計（`fetchInstruct`、`terminationReq`、去重與收斂）。
5. `correlationId` 與 `group` 的分流設計（client1/path1、client2/path2）。
6. 依 `config/5GC`（UPF 5 秒回報等）評估大時間窗資料拉取可行性。

### **0.2 Out of Scope（本版先不做）**

1. OAuth2 正式串接（先 stub）。
2. accuracy monitor 經 ADRF retrieval（明確不做；保留現行 Mongo + memory）。
3. `data-set-id`（EnhDataMgmt）策略。
4. ADRF delete 流程細節（先不納入 V0）。

---

## **1. 當前系統事實（作為設計前提）**

1. NWDAF 目前 UPF 入口為 `POST /collector/upf-notify`。
2. Group 訂閱在 NWDAF 內會先做 `groupId -> SUPI[]` 展開，再逐 SUPI 建立資料收集流。
3. `correlationId` 在現行程式語意是「訂閱流識別」，不是「group 識別」。
4. 現行 AnLF accuracy path：
    - 即時推論讀 memory ring buffer。
    - accuracy ground truth 主要讀 MongoDB（失敗才 fallback memory）。
5. 這一版 ADRF 的目的：
    - 線上資料持久化到 ADRF。
    - retrain 時可從 ADRF 拉歷史資料。
    - 不改動 accuracy 熱路徑。

---

## **2. Store 接點（沿用你們確認版 payload）**

## **2.1 規格最小集合（必須）**

規格關鍵：`NadrfDataStoreRecord` 需滿足 oneOf；本案走 `dataSub + dataNotif`。

### **建議請求體（具體範例，貼近 TS 29.564 的 UPF 訂閱 + 通知格式）**

```json
{
  "dataSub": [
    {
      "smfDataSub": {
        "supi": "imsi-208930000000001",
        "notifUri": "http://192.168.107.5:8080/collector/notify",
        "notifId": "corr-session-001",
        "notifMethod": "PERIODIC",
        "repPeriod": 5,
        "eventSubs": [
          {
            "event": "UPF_EVENT",
            "upfEvents": [
              {
                "type": "USER_DATA_USAGE_MEASURES",
                "measurementTypes": [
                  "VOLUME_MEASUREMENT",
                  "THROUGHPUT_MEASUREMENT"
                ],
                "granularityOfMeasurement": "PER_SESSION"
              }
            ],
            "bundlingAllowed": true,
            "bundledEventNotifyUri": "http://192.168.107.5:8080/collector/upf-notify"
          }
        ]
      }
    }
  ],
  "dataNotif": {
    "upfEventNotifs": [
      {
        "correlationId": "corr-session-001",
        "notificationItems": [
          {
            "eventType": "USER_DATA_USAGE_MEASURES",
            "timeStamp": "2026-03-20T10:00:00Z",
            "startTime": "2026-03-20T10:00:00Z",
            "ueIpv4Addr": "10.10.0.1",
            "userDataUsageMeasurements": [
              {
                "volumeMeasurement": {
                  "totalVolume": 6800,
                  "ulVolume": 1200,
                  "dlVolume": 5600,
                  "totalNbOfPackets": 96,
                  "ulNbOfPackets": 30,
                  "dlNbOfPackets": 66
                },
                "throughputMeasurement": {
                  "ulThroughput": "16 Kbps",
                  "dlThroughput": "64 Kbps",
                  "ulPacketThroughput": "12 pps",
                  "dlPacketThroughput": "25 pps"
                }
              }
            ]
          }
        ]
      }
    ]
  }
}
```

### **欄位閱讀重點**

1. `dataSub[0].smfDataSub.notifId` 與 `dataNotif.upfEventNotifs[*].correlationId` 應可對上。
2. `notificationItems[*].startTime` 建議作為主要 measurement timestamp。
3. `dataNotif` 建議保留原始 TS 29.564 形狀，不先壓平。

### **本版還不支援的 optional**

1. `storeHandl`
2. `dataSetTag`
3. `dsc`
4. `suppFeat`

原因：先打通最小合法 store 與 retrain 可拉取能力。

## **2.2 Store 流程（新方向）**

1. NWDAF 收到 `/collector/upf-notify`。
2. 轉成 `NadrfDataStoreRecord(dataSub + dataNotif)`。
3. 呼叫 ADRF `POST /data-store-records`。
4. 接收 `201 Created`，除了 `Location(.../{storeTransId})` header，response body 也應包含建立完成的 `NadrfDataStoreRecord`；再記錄操作結果（可選：審計/追蹤）。
5. accuracy 路徑不依賴此回寫結果（避免牽動熱路徑）。

---

## **3. Retrieval 接點（僅 Retrain/FL，採 Subscription + fetch）**

## **3.1 為何本版選擇 fetch 模式**

1. 長時間窗（如過去 30 分鐘）資料量大。
2. `consTrigNotif=true` 可讓 ADRF 先發 `fetchInstruct`，NWDAF 分批拉取，避免 callback body 過大。
3. 這條路徑是 retrain 非即時熱路徑，可接受多一步 GET 拉取。

## **3.2 必要能力最小集合**

1. `POST /data-retrieval-subscriptions`（建立 retrieval 訂閱）。
2. `notificationURI` callback（接 `NadrfDataRetrievalNotification`）。
3. `GET /data-store-records?fetch-correlation-ids=...`（依 fetch instruction 拉資料）。
4. `DELETE /data-retrieval-subscriptions/{subscriptionId}`（收尾清理）。

## **3.3 Retrieval 訂閱請求（建議）**

```json
{
  "notifCorrId": "retrain-job-20260324-001",
  "notificationURI": "http://192.168.107.5:8080/adrf/retrieval-notify",
  "timePeriod": {
    "startTime": "2026-03-24T09:00:00Z",
    "stopTime": "2026-03-24T09:30:00Z"
  },
  "dataSub": {
    "smfDataSub": {
      "supi": "imsi-208930000000001",
      "notifId": "corr-session-001",
      "eventSubs": [
        {
          "event": "UPF_EVENT",
          "upfEvents": [
            {
              "type": "USER_DATA_USAGE_MEASURES",
              "measurementTypes": [
                "VOLUME_MEASUREMENT",
                "THROUGHPUT_MEASUREMENT"
              ],
              "granularityOfMeasurement": "PER_SESSION"
            }
          ]
        }
      ]
    }
  },
  "consTrigNotif": true
}
```

說明：

1. `timePeriod` 使用 `startTime/stopTime`。
2. retrain 用過去時間窗即可，不需要覆蓋未來。
3. `notifCorrId` 建議與 retrain job / Daisy TID 做一對一綁定。
4. retrieval 訂閱的 `dataSub` 是物件（object），與 store record 的 `dataSub`（array）不同。
5. `POST /data-retrieval-subscriptions` 成功回應為 `201 Created`，除了 `Location(.../{subscriptionId})` header，response body 也應包含建立完成的 `NadrfDataRetrievalSubscription`。

---

## **4. RetrievalNotify（notify）設計**

## **4.1 callback 形狀與處理原則**

1. ADRF 對 `notificationURI` 發 `POST`。
2. body 為 `NadrfDataRetrievalNotification`，必有：`notifCorrId`、`timeStamp`。
3. body 三選一：`anaNotifications` / `dataNotif` / `fetchInstruct`。
4. fetch 模式下，預期主要收到 `fetchInstruct`。
5. NWDAF 成功處理後回 `204 No Content`。

## **4.2 `fetch-correlation-ids` 與 `terminationReq` 的關係**

1. `fetch-correlation-ids` 代表「這一批可拉取資料」的鍵，不代表整個訂閱結束。
2. 本版設計採用：`fetchCorrIds` 直接放對應資料的 `storeTransId`（不加 `st:` 或其他前綴）。
3. `terminationReq=true` 代表「此訂閱不會再有後續通知」的收斂訊號。
4. 即使收到 `terminationReq=true`，仍建議 NWDAF 主動 `DELETE` 訂閱資源做明確收尾。

## **4.3 去重、容錯與收斂**

1. 去重鍵建議：`notifCorrId + timeStamp + payloadHash`。
2. callback 內容落地成功後才回 `204`。
3. 設計 watchdog：避免極端情況下僅依賴單一 `terminationReq`。

## **4.4 參考範例（fetch 路線）**

### **A. ADRF 回 fetch 通知時（callback 到 NWDAF）**

`expiry` 在 `FetchInstruction` 中是選填欄位，本範例先省略。

```json
{
  "notifCorrId": "retrain-job-20260324-001",
  "timeStamp": "2026-03-24T09:30:05Z",
  "fetchInstruct": {
    "fetchUri": "http://adrf.local/nadrf-datamanagement/v1/data-store-records",
    "fetchCorrIds": [
      "store-trans-20260324-000001",
      "store-trans-20260324-000002"
    ]
  },
  "terminationReq": false
}
```

### **B. NWDAF 實際 fetch 請求**

```
GET /nadrf-datamanagement/v1/data-store-records?fetch-correlation-ids=store-trans-20260324-000001,store-trans-20260324-000002
```

### **C. ADRF fetch 回應（重資料）**

```json
{
  "dataSub": [
    {
      "smfDataSub": {
        "supi": "imsi-208930000000001",
        "notifUri": "http://192.168.107.5:8080/collector/notify",
        "notifId": "corr-session-001",
        "notifMethod": "PERIODIC",
        "repPeriod": 5,
        "eventSubs": [
          {
            "event": "UPF_EVENT",
            "upfEvents": [
              {
                "type": "USER_DATA_USAGE_MEASURES",
                "measurementTypes": [
                  "VOLUME_MEASUREMENT",
                  "THROUGHPUT_MEASUREMENT"
                ],
                "granularityOfMeasurement": "PER_SESSION"
              }
            ],
            "bundlingAllowed": true,
            "bundledEventNotifyUri": "http://192.168.107.5:8080/collector/upf-notify"
          }
        ]
      }
    }
  ],
  "dataNotif": {
    "upfEventNotifs": [
      {
        "correlationId": "corr-session-001",
        "notificationItems": [
          {
            "eventType": "USER_DATA_USAGE_MEASURES",
            "startTime": "2026-03-24T09:00:00Z",
            "timeStamp": "2026-03-24T09:00:05Z",
            "ueIpv4Addr": "10.10.0.1",
            "userDataUsageMeasurements": [
              {
                "volumeMeasurement": {
                  "totalVolume": 6800,
                  "ulVolume": 1200,
                  "dlVolume": 5600
                },
                "throughputMeasurement": {
                  "ulThroughput": "16 Kbps",
                  "dlThroughput": "64 Kbps"
                }
              }
            ]
          },
          {
            "eventType": "USER_DATA_USAGE_MEASURES",
            "startTime": "2026-03-24T09:00:05Z",
            "timeStamp": "2026-03-24T09:00:10Z",
            "ueIpv4Addr": "10.10.0.1",
            "userDataUsageMeasurements": [
              {
                "volumeMeasurement": {
                  "totalVolume": 7100,
                  "ulVolume": 1300,
                  "dlVolume": 5800
                },
                "throughputMeasurement": {
                  "ulThroughput": "17 Kbps",
                  "dlThroughput": "66 Kbps"
                }
              }
            ]
          }
        ]
      }
    ]
  }
}
```

### **D. 最後一包通知（可選）**

```json
{
  "notifCorrId": "retrain-job-20260324-001",
  "timeStamp": "2026-03-24T09:30:10Z",
  "fetchInstruct": {
    "fetchUri": "http://adrf.local/nadrf-datamanagement/v1/data-store-records",
    "fetchCorrIds": [
      "store-trans-20260324-000100"
    ]
  },
  "terminationReq": true
}
```

---

## **5. correlationId 與 group 分流（client1/path1、client2/path2）**

## **5.1 關鍵結論**

1. `correlationId` 可以當流識別鍵，但不是 group 的語意鍵。
2. group 訂閱目前是先展開成 SUPI，再逐 SUPI 形成多條 correlation stream。
3. 因此要做 retrain 分路，建議用「`groupId -> correlationId[]` 映射」而不是把單一 `correlationId` 直接等同 group。

## **5.2 建議做法**

1. retrain 啟動時先快照分流映射（避免訓練中 membership 漂移）。
2. 形成兩份 partition plan：
    - path1/client1 使用 `corrSetA`
    - path2/client2 使用 `corrSetB`
3. fetch 回來後依 `dataNotif.upfEventNotifs[*].correlationId` 做本地分桶。
4. 輸出兩份 dataset 給 Daisy clients（例如兩個資料目錄或兩份 manifest）。

---

## **6. 長時間窗（例如 30 分鐘）下的 payload 形狀**

## **6.1 先釐清規格表面形狀**

1. Retrieval notify 在 fetch 模式下可以很小（只給 `fetchInstruct`）。
2. 真正重資料在後續 `GET /data-store-records?fetch-correlation-ids=...`。
3. `GET` 回應 schema 是 `NadrfDataStoreRecord`。

## **6.2 會是「疊在一起」還是「個別通知」**

兩者都可行，取決於 ADRF 分批策略；本版採「固定筆數分批」：

1. ADRF 先依 retrieval 條件（`timePeriod + dataSub`）找出符合的 store records。
2. `fetchCorrIds` 直接使用這些 records 的 `storeTransId`（不加前綴）。
3. ADRF 依常數 `n`（config 可調）切成多包 fetch 指令。
4. NWDAF 分批 GET，逐批寫入 retrain staging。

## **6.3 建議的批次策略（V0）**

1. 不做時間切桶，不做 `corrId + timeBucket` 命名。
2. 每個 `fetchCorrId` 對應一筆 store record 的 `storeTransId`。
3. 每次 callback 送出最多 `n` 個 `fetchCorrIds`；最後一包可帶 `terminationReq=true`。
4. NWDAF 若要最簡單可先一筆一筆拉取（每次 GET 帶 1 個 ID），之後再優化成多 ID 併發。
5. 若要兼顧 URL 長度，建議仍保留 `n` 的上限控制。

---

## **7. 與現行 config 的相容性驗證**

以下值來自 `config/5GC`：

1. `UPF-EES.periodSec = 5`。
2. `NWDAF samplingInterval = 5`。
3. `accuracyMonitor.checkInterval = 50`。
4. `accuracyMonitor.warmupDuration = 155`。
5. `SMF urrPeriod = 5`。

推論：

1. 30 分鐘約為 `30*60/5 = 360` 筆/每 corrId。
2. retrain 若涉及多 corrId，資料量會放大。
3. 使用 fetch 分批可控制單次 payload 尺寸與 NWDAF 記憶體壓力。
4. accuracy 熱路徑維持原樣，不受 ADRF retrieval 影響。

---

## **8. 元件資料流（新方向）**

## **8.1 Store 流**

```
UPF-EES (5s)
  -> NWDAF /collector/upf-notify
    -> Build NadrfDataStoreRecord (dataSub + dataNotif)
      -> ADRF POST /data-store-records
         <- 201 + Location(storeTransId)
```

## **8.2 Retrain Retrieval 流（fetch）**

```
NWDAF(MTLF retrain trigger)
  -> POST /data-retrieval-subscriptions (consTrigNotif=true, timePeriod=[past window])
     <- 201 + subscriptionId
  <- ADRF callback POST notificationURI (fetchInstruct, notifCorrId, timeStamp)
  -> GET /data-store-records?fetch-correlation-ids=...
     <- NadrfDataStoreRecord (batch)
  -> repeat until completion (or terminationReq=true)
  -> DELETE /data-retrieval-subscriptions/{subscriptionId}
  -> build per-client dataset (path1/path2)
  -> Daisy /publish_task retrain
```

---

## **9. 本版結論**

1. 新方向已明確：`store` 走 ADRF、`accuracy` 不走 ADRF、`retrain` 走 RetrievalSubscribe(fetch)。
2. `correlationId` 可用於資料分流，但要配合 `groupId -> correlationId[]` 映射使用。
3. 長時間窗 retrieval 應以 fetch 分批，且本版 `fetchCorrIds` 直接使用 `storeTransId`。
4. `terminationReq` 與 `fetch-correlation-ids` 角色不同，兩者都建議保留。