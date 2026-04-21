# NWDAF 專案架構與元件關係圖

本文件說明各個元件如何連接在一起。

---

## 整體架構圖

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              cmd/main.go                                     │
│                                                                              │
│   main() ──────────► service.NewApp() ──────────► nwdaf.Start()             │
│                              │                         │                     │
└──────────────────────────────┼─────────────────────────┼─────────────────────┘
                               │                         │
                               ▼                         ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                         pkg/service/init.go                                   │
│                                                                               │
│  ┌─────────────────────────────────────────────────────────────────────────┐ │
│  │                          NwdafApp struct                                 │ │
│  │                                                                          │ │
│  │   cfg *factory.Config ◄────────── 配置（從 YAML 讀取）                   │ │
│  │   nwdafCtx *context.NWDAFContext ◄── 狀態存儲（訂閱資料）               │ │
│  │   processor *processor.Processor ◄── 業務邏輯處理器                     │ │
│  │   sbiServer *sbi.Server ◄────────── HTTP 伺服器                        │ │
│  │                                                                          │ │
│  └─────────────────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────────────┘
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
          ▼                    ▼                    ▼
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────────────────────┐
│ pkg/factory/     │ │ internal/context │ │ internal/sbi/                    │
│ config.go        │ │                  │ │                                  │
│                  │ │ NWDAFContext     │ │ ┌────────────────────────────┐  │
│ ┌──────────────┐ │ │                  │ │ │         Server              │  │
│ │   Config     │ │ │ ┌──────────┐    │ │ │                             │  │
│ │              │ │ │ │ NfId     │    │ │ │  router *gin.Engine         │  │
│ │  Sbi         │ │ │ │ subs map │    │ │ │  httpServer *http.Server   │  │
│ │  NwdafName   │ │ │ └──────────┘    │ │ │                             │  │
│ │  NrfUri      │ │ │                  │ │ │  ┌───────────────────────┐ │  │
│ └──────────────┘ │ │ Add/Get/Delete  │ │ │  │ Handler Functions     │ │  │
│                  │ │ Subscription()  │ │ │  │                       │ │  │
└──────────────────┘ └──────────────────┘ │ │  │ HandleCreateSub()    │ │  │
                                          │ │  │ HandleUpdateSub()    │ │  │
                                          │ │  │ HandleDeleteSub()    │ │  │
                                          │ │  └───────────────────────┘ │  │
                                          │ └────────────────────────────┘  │
                                          │               │                  │
                                          │               ▼                  │
                                          │ ┌────────────────────────────┐  │
                                          │ │ processor/eventssubscription│ │
                                          │ │                             │  │
                                          │ │ HandleCreateSubscription()  │  │
                                          │ │ HandleUpdateSubscription()  │  │
                                          │ │ HandleDeleteSubscription()  │  │
                                          │ │ validateSubscription()      │  │
                                          │ └────────────────────────────┘  │
                                          └──────────────────────────────────┘
```

---

## 請求處理流程

```
HTTP Client
     │
     │ POST /nnwdaf-eventssubscription/v1/subscriptions
     │ {"eventSubscriptions": [...], "notificationURI": "..."}
     │
     ▼
┌────────────────────────────────────────────────────────────────────┐
│                      1. internal/sbi/server.go                      │
│                                                                     │
│   gin.Router 接收請求，根據 URL 找到對應的 Handler                  │
│   POST /subscriptions → HandleCreateSubscription                    │
└────────────────────────────────────────────────────────────────────┘
     │
     ▼
┌────────────────────────────────────────────────────────────────────┐
│               2. internal/sbi/api_eventssubscription.go             │
│                                                                     │
│   func (s *Server) HandleCreateSubscription(c *gin.Context) {       │
│       // 解析 JSON                                                  │
│       c.ShouldBindJSON(&req)                                        │
│                                                                     │
│       // 呼叫 Processor 處理業務邏輯                                │
│       response, id, err := s.Processor().HandleCreateSubscription() │
│                                                                     │
│       // 回傳結果                                                   │
│       c.JSON(201, response)                                         │
│   }                                                                 │
└────────────────────────────────────────────────────────────────────┘
     │
     │ s.Processor() 從哪來？
     │ → Server struct 嵌入了 nwdafApp interface
     │ → nwdafApp 有 Processor() method
     │ → 實際上是 NwdafApp.processor
     │
     ▼
┌────────────────────────────────────────────────────────────────────┐
│             3. internal/sbi/processor/eventssubscription.go         │
│                                                                     │
│   func (p *Processor) HandleCreateSubscription(req) {               │
│       // 驗證請求                                                   │
│       p.validateSubscription(req)                                   │
│       p.validateAbnormalBehaviour(eventSub)                         │
│                                                                     │
│       // 生成 ID                                                    │
│       subscriptionId := context.NewSubscriptionId()                 │
│                                                                     │
│       // 儲存到 Context                                             │
│       context.GetSelf().AddSubscription(subscription)               │
│                                                                     │
│       return response, subscriptionId, nil                          │
│   }                                                                 │
└────────────────────────────────────────────────────────────────────┘
     │
     ▼
┌────────────────────────────────────────────────────────────────────┐
│                   4. internal/context/context.go                    │
│                                                                     │
│   func (c *NWDAFContext) AddSubscription(sub *Subscription) {       │
│       c.mu.Lock()           // 取得鎖（併發安全）                   │
│       defer c.mu.Unlock()   // 函數結束時解鎖                       │
│                                                                     │
│       c.subscriptions[sub.ID] = sub   // 存入 map                   │
│   }                                                                 │
└────────────────────────────────────────────────────────────────────┘
     │
     │ 回傳一路往上
     ▼
HTTP Response: 201 Created + {"eventSubscriptions": [...]}
```

---

## `s.Processor()` 怎麼來的？詳細解釋

### 問題：api_eventssubscription.go 中

```go
func (s *Server) HandleCreateSubscription(c *gin.Context) {
    response, id, _ := s.Processor().HandleCreateSubscription(&req)
    //                  ^^^^^^^^^^^^
    //                  這是從哪來的？
}
```

### 答案：Interface 嵌入

**Step 1：定義 interface**
```go
// server.go
type nwdafApp interface {
    Config() *factory.Config
    Context() *nwdaf_context.NWDAFContext
    Processor() *processor.Processor
    CancelContext() context.Context
}
```

**Step 2：Server 嵌入這個 interface**
```go
// server.go
type Server struct {
    nwdafApp   // 嵌入 interface（沒有欄位名，叫做「嵌入」）
    
    httpServer *http.Server
    router     *gin.Engine
}
```

**Step 3：NewServer 時傳入實作**
```go
// server.go
func NewServer(nwdaf nwdafApp) (*Server, error) {
    s := &Server{
        nwdafApp: nwdaf,   // 這裡傳入了 NwdafApp 實例
        router:   gin.New(),
    }
}
```

**Step 4：NwdafApp 實作了這個 interface**
```go
// pkg/service/init.go
type NwdafApp struct {
    processor *processor.Processor
    // ...
}

func (a *NwdafApp) Processor() *processor.Processor {
    return a.processor
}
```

**結果**：
```go
s.Processor()
// 等同於
s.nwdafApp.Processor()
// 實際執行
NwdafApp.Processor() → 回傳 NwdafApp.processor
```

---

## 檔案職責總覽

| 檔案 | 職責 |
|------|------|
| `cmd/main.go` | 程式入口、CLI 參數、啟動服務 |
| `pkg/factory/config.go` | 讀取 YAML 配置檔 |
| `pkg/service/init.go` | 組裝所有元件、管理生命週期 |
| `internal/context/context.go` | 全域狀態、訂閱資料存儲 |
| `internal/sbi/server.go` | HTTP 伺服器、路由設定 |
| `internal/sbi/api_eventssubscription.go` | HTTP Handler（解析請求、回傳回應）|
| `internal/sbi/processor/eventssubscription.go` | 業務邏輯（驗證、處理）|
