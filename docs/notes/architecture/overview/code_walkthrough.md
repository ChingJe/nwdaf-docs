# NWDAF EventSubscription 程式碼詳細解說

本文件詳細說明 NWDAF (Network Data Analytics Function) 專案的每個程式碼檔案，包含其用途、運作原理及與 3GPP 規格的對應關係。

---

## 目錄

1. [專案架構總覽](#專案架構總覽)
2. [進入點與啟動流程](#進入點與啟動流程)
3. [配置系統](#配置系統)
4. [日誌系統](#日誌系統)
5. [Context 與訂閱管理](#context-與訂閱管理)
6. [SBI Server 與路由](#sbi-server-與路由)
7. [API Handler](#api-handler)
8. [業務邏輯 (Processor)](#業務邏輯-processor)
9. [3GPP 規格對應](#3gpp-規格對應)

---

## 專案架構總覽

```
nwdaf/
├── cmd/                      # 程式進入點
│   └── main.go
├── config/                   # 配置檔
│   └── nwdafcfg.yaml
├── internal/                 # 內部套件（不對外暴露）
│   ├── context/              # 全域狀態管理
│   │   └── context.go
│   ├── logger/               # 日誌系統
│   │   └── logger.go
│   └── sbi/                  # Service Based Interface (HTTP API)
│       ├── server.go         # HTTP Server
│       ├── api_eventssubscription.go  # API 處理器
│       └── processor/        # 業務邏輯
│           ├── processor.go
│           └── eventssubscription.go
├── pkg/                      # 公開套件
│   ├── app/                  # App 介面定義
│   │   └── app.go
│   ├── factory/              # 配置工廠
│   │   └── config.go
│   └── service/              # 主服務
│       └── init.go
├── go.mod                    # Go 模組定義
└── Makefile                  # 建構腳本
```

### 設計原則

這個架構參考了 **free5gc** 的 NF (Network Function) 設計模式：

1. **分層設計**：`cmd` → `pkg/service` → `internal/sbi` → `internal/context`
2. **關注點分離**：HTTP 處理 (`api_*.go`) 與業務邏輯 (`processor/`) 分開
3. **依賴注入**：透過介面 (`app.App`) 解耦各模組

---

## 進入點與啟動流程

### `cmd/main.go`

```go
package main

import (
    "context"
    "github.com/urfave/cli/v2"
    // ...
)

func main() {
    app := cli.NewApp()
    app.Name = "nwdaf"
    app.Action = action
    app.Flags = []cli.Flag{
        &cli.StringFlag{Name: "config", Aliases: []string{"c"}},
        &cli.StringSliceFlag{Name: "log", Aliases: []string{"l"}},
    }
    app.Run(os.Args)
}

func action(cliCtx *cli.Context) error {
    // 1. 讀取配置
    cfg, err := factory.ReadConfig(cliCtx.String("config"))
    
    // 2. 建立 Context 與 Cancel 機制（用於 Graceful Shutdown）
    ctx, cancel := context.WithCancel(context.Background())
    
    // 3. 監聽系統信號 (SIGINT, SIGTERM)
    signal.Notify(sigCh, os.Interrupt, syscall.SIGTERM)
    go func() {
        <-sigCh
        cancel()  // 觸發關閉流程
    }()
    
    // 4. 建立並啟動 NWDAF App
    nwdaf, _ := service.NewApp(ctx, cfg)
    nwdaf.Start()
    
    return nil
}
```

**重點說明**：
- 使用 `urfave/cli` 處理命令列參數
- 實作 **Graceful Shutdown**：收到 SIGTERM 時，透過 `context.Cancel()` 通知所有 goroutine 停止

---

## 配置系統

### `pkg/factory/config.go`

```go
type Config struct {
    Info          *Info          `yaml:"info"`
    Configuration *Configuration `yaml:"configuration"`
    Logger        *Logger        `yaml:"logger,omitempty"`
}

type Configuration struct {
    NwdafName          string   `yaml:"nwdafName,omitempty"`
    Sbi                *Sbi     `yaml:"sbi,omitempty"`
    NrfUri             string   `yaml:"nrfUri,omitempty"`
    SupportedAnalytics []string `yaml:"supportedAnalytics,omitempty"`
}

type Sbi struct {
    Scheme       string `yaml:"scheme"`       // http 或 https
    RegisterIPv4 string `yaml:"registerIPv4"` // 向 NRF 註冊用的 IP
    BindingIPv4  string `yaml:"bindingIPv4"`  // 綁定監聽的 IP
    Port         int    `yaml:"port"`
}
```

**重點說明**：
- `NwdafEventsSubResUriPrefix = "/nnwdaf-eventssubscription/v1"` 定義 API 路徑前綴
- 若配置檔缺少某些欄位，會自動填入預設值

### `config/nwdafcfg.yaml`

```yaml
info:
  version: 1.0.0
  description: NWDAF initial configuration

configuration:
  nwdafName: NWDAF
  sbi:
    scheme: http
    registerIPv4: 127.0.0.1
    bindingIPv4: 127.0.0.1
    port: 8080
  supportedAnalytics:
    - ABNORMAL_BEHAVIOUR
```

---

## 日誌系統

### `internal/logger/logger.go`

```go
var (
    Log      *logrus.Logger
    MainLog  *logrus.Entry  // 主程式日誌
    InitLog  *logrus.Entry  // 初始化日誌
    SBILog   *logrus.Entry  // HTTP API 日誌
    ProcLog  *logrus.Entry  // 業務邏輯日誌
    CtxLog   *logrus.Entry  // Context 日誌
    // ...
)

func init() {
    Log = logrus.New()
    Log.SetFormatter(&logrus.TextFormatter{
        FullTimestamp:   true,
        TimestampFormat: "2006-01-02T15:04:05.000Z07:00",
    })
    
    MainLog = Log.WithField("component", "NWDAF").WithField("category", "Main")
    // ...
}
```

**重點說明**：
- 使用 `logrus` 套件，支援結構化日誌
- 每個模組有獨立的 `Entry`，方便追蹤問題來源
- 輸出範例：
  ```
  INFO[2025-12-30T18:42:08.924+08:00] Handle CreateSubscription  category=SBI component=NWDAF
  ```

---

## Context 與訂閱管理

### `internal/context/context.go`

這是 NWDAF 的**全域狀態容器**，負責儲存所有活躍的訂閱。

```go
type NWDAFContext struct {
    NfId      string  // NF Instance ID (UUID)
    NwdafName string
    
    mu            sync.RWMutex           // 讀寫鎖（併發安全）
    subscriptions map[string]*Subscription
}

type Subscription struct {
    ID              string
    NotificationURI string  // 消費者的回呼 URL
    NotifCorrId     string  // 關聯 ID
    EventSubs       []models.NwdafEventsSubscriptionEventSubscription
    EvtReq          *models.ReportingInformation
    CreatedAt       time.Time
    UpdatedAt       time.Time
}
```

**關鍵方法**：

| 方法 | 說明 |
|------|------|
| `Init()` | 初始化 Context，生成 NF Instance ID |
| `GetSelf()` | 取得單例 Context |
| `AddSubscription(sub)` | 新增訂閱（thread-safe） |
| `GetSubscription(id)` | 查詢訂閱 |
| `UpdateSubscription(sub)` | 更新訂閱 |
| `DeleteSubscription(id)` | 刪除訂閱 |

**併發安全設計**：
```go
func (c *NWDAFContext) AddSubscription(sub *Subscription) {
    c.mu.Lock()         // 取得寫鎖
    defer c.mu.Unlock() // 確保解鎖
    
    sub.CreatedAt = time.Now()
    c.subscriptions[sub.ID] = sub
}
```

---

## SBI Server 與路由

### `internal/sbi/server.go`

使用 **Gin** 框架建立 HTTP Server。

```go
type Server struct {
    nwdafApp   // 嵌入介面，可存取 Config、Context 等
    httpServer *http.Server
    router     *gin.Engine
}

func NewServer(nwdaf nwdafApp) (*Server, error) {
    s := &Server{
        nwdafApp: nwdaf,
        router:   gin.New(),
    }
    
    // 設定中間件
    s.router.Use(gin.Recovery())  // Panic 恢復
    s.router.Use(gin.Logger())    // 請求日誌
    
    // 註冊路由群組
    eventsSubRoutes := s.getEventsSubscriptionRoutes()
    eventsSubGroup := s.router.Group("/nnwdaf-eventssubscription/v1")
    applyRoutes(eventsSubGroup, eventsSubRoutes)
    
    // 建立 HTTP Server
    bindAddr := fmt.Sprintf("%s:%d", cfg.Sbi.BindingIPv4, cfg.Sbi.Port)
    s.httpServer = &http.Server{Addr: bindAddr, Handler: s.router}
    
    return s, nil
}

func (s *Server) getEventsSubscriptionRoutes() []Route {
    return []Route{
        {Name: "CreateSubscription", Method: "POST", Pattern: "/subscriptions", 
         APIFunc: s.HandleCreateSubscription},
        {Name: "UpdateSubscription", Method: "PUT", Pattern: "/subscriptions/:subscriptionId", 
         APIFunc: s.HandleUpdateSubscription},
        {Name: "DeleteSubscription", Method: "DELETE", Pattern: "/subscriptions/:subscriptionId", 
         APIFunc: s.HandleDeleteSubscription},
    }
}
```

**路由對應表**：

| HTTP Method | Path | Handler |
|-------------|------|---------|
| POST | `/nnwdaf-eventssubscription/v1/subscriptions` | `HandleCreateSubscription` |
| PUT | `/nnwdaf-eventssubscription/v1/subscriptions/:subscriptionId` | `HandleUpdateSubscription` |
| DELETE | `/nnwdaf-eventssubscription/v1/subscriptions/:subscriptionId` | `HandleDeleteSubscription` |

---

## API Handler

### `internal/sbi/api_eventssubscription.go`

處理 HTTP 請求的入口點，負責：
1. 解析 JSON 請求
2. 呼叫 Processor 處理業務邏輯
3. 回傳 HTTP 回應

```go
func (s *Server) HandleCreateSubscription(c *gin.Context) {
    // 1. 解析 JSON
    var req models.NnwdafEventsSubscription
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, models.ProblemDetails{
            Status: http.StatusBadRequest,
            Cause:  "INVALID_JSON",
            Detail: err.Error(),
        })
        return
    }
    
    // 2. 呼叫 Processor 處理
    response, subscriptionId, problemDetails := s.Processor().HandleCreateSubscription(&req)
    if problemDetails != nil {
        c.JSON(int(problemDetails.Status), problemDetails)
        return
    }
    
    // 3. 回傳成功回應
    locationUri := fmt.Sprintf("%s://%s:%d%s/subscriptions/%s",
        cfg.Sbi.Scheme, cfg.Sbi.RegisterIPv4, cfg.Sbi.Port,
        factory.NwdafEventsSubResUriPrefix, subscriptionId)
    
    c.Header("Location", locationUri)  // 標準要求的 Location header
    c.JSON(http.StatusCreated, response)
}
```

**回應格式**：

成功 (201 Created)：
```json
{
  "eventSubscriptions": [...],
  "notificationURI": "http://..."
}
```
Header: `Location: http://127.0.0.1:8080/nnwdaf-eventssubscription/v1/subscriptions/{id}`

錯誤 (4xx/5xx)：
```json
{
  "status": 400,
  "cause": "INVALID_REQUEST",
  "detail": "eventSubscriptions is required"
}
```

---

## 業務邏輯 (Processor)

### `internal/sbi/processor/eventssubscription.go`

核心業務邏輯，包含驗證與資料處理。

#### 訂閱建立流程

```go
func (p *Processor) HandleCreateSubscription(
    req *models.NnwdafEventsSubscription,
) (*models.NnwdafEventsSubscription, string, *models.ProblemDetails) {
    
    // 1. 驗證請求
    if problemDetails := p.validateSubscription(req); problemDetails != nil {
        return nil, "", problemDetails
    }
    
    // 2. 生成 Subscription ID
    subscriptionId := nwdaf_context.NewSubscriptionId()
    
    // 3. 建立 Subscription 物件
    subscription := &nwdaf_context.Subscription{
        ID:              subscriptionId,
        NotificationURI: req.NotificationURI,
        EventSubs:       req.EventSubscriptions,
        // ...
    }
    
    // 4. 存入 Context
    ctx := nwdaf_context.GetSelf()
    ctx.AddSubscription(subscription)
    
    // 5. 準備回應
    return &models.NnwdafEventsSubscription{
        EventSubscriptions: req.EventSubscriptions,
        NotificationURI:    req.NotificationURI,
    }, subscriptionId, nil
}
```

#### ABNORMAL_BEHAVIOUR 驗證

根據 3GPP TS 29.520 規格，ABNORMAL_BEHAVIOUR 事件有特定的驗證規則：

```go
func (p *Processor) validateAbnormalBehaviour(
    eventSub *models.NwdafEventsSubscriptionEventSubscription,
) *models.ProblemDetails {
    
    // 規則 1: 必須有 tgtUe
    if eventSub.TgtUe == nil {
        return &models.ProblemDetails{
            Status: http.StatusBadRequest,
            Cause:  "INVALID_REQUEST",
            Detail: "tgtUe is required for ABNORMAL_BEHAVIOUR",
        }
    }
    
    // 規則 2: tgtUe 必須包含 supis、intGroupIds 或 anyUe=true
    hasTarget := len(eventSub.TgtUe.Supis) > 0 ||
                 len(eventSub.TgtUe.IntGroupIds) > 0 ||
                 eventSub.TgtUe.AnyUe
    if !hasTarget {
        return &models.ProblemDetails{...}
    }
    
    // 規則 3: 必須有 excepRequs 或 exptAnaType
    hasExcepRequs := len(eventSub.ExcepRequs) > 0
    hasExptAnaType := eventSub.ExptAnaType != ""
    if !hasExcepRequs && !hasExptAnaType {
        return &models.ProblemDetails{...}
    }
    
    return nil
}
```

---

## 主服務結構

### `pkg/service/init.go`

整合所有元件的主結構。

```go
type NwdafApp struct {
    cfg       *factory.Config
    nwdafCtx  *nwdaf_context.NWDAFContext
    ctx       context.Context
    cancel    context.CancelFunc
    processor *processor.Processor
    sbiServer *sbi.Server
    wg        sync.WaitGroup
}

func NewApp(ctx context.Context, cfg *factory.Config) (*NwdafApp, error) {
    nwdaf := &NwdafApp{cfg: cfg}
    
    // 1. Context 初始化
    nwdaf_context.Init()
    nwdaf.nwdafCtx = nwdaf_context.GetSelf()
    
    // 2. Processor 初始化
    nwdaf.processor = processor.NewProcessor(nwdaf)
    
    // 3. SBI Server 初始化
    nwdaf.sbiServer, _ = sbi.NewServer(nwdaf)
    
    return nwdaf, nil
}

func (a *NwdafApp) Start() {
    // 啟動 Shutdown 監聽
    go a.listenShutdownEvent()
    
    // 啟動 HTTP Server
    a.sbiServer.Run(context.Background(), &a.wg)
    
    // 等待所有 goroutine 結束
    a.wg.Wait()
}
```

---

## 3GPP 規格對應

| 程式碼元件 | 3GPP 規格參考 |
|-----------|--------------|
| `POST /subscriptions` | TS 29.520 Clause 5.1.3.2.3.1 |
| `PUT /subscriptions/{id}` | TS 29.520 Clause 5.1.3.3.3.2 |
| `DELETE /subscriptions/{id}` | TS 29.520 Clause 5.1.3.3.3.1 |
| `NnwdafEventsSubscription` 資料結構 | TS 29.520 Clause 5.1.6.2.2 |
| `EventSubscription` 資料結構 | TS 29.520 Clause 5.1.6.2.3 |
| ABNORMAL_BEHAVIOUR 驗證規則 | TS 29.520 Clause 4.2.2.2.2 (Table 4.2.2.2.2-1) |
| `SUSPICION_OF_DDOS_ATTACK` | TS 29.520 ExceptionId enumeration |

---

## 執行與測試

### 啟動服務

```bash
# 編譯
make build

# 執行
./bin/nwdaf --config config/nwdafcfg.yaml
```

### 測試 API

**建立訂閱**：
```bash
curl -X POST http://127.0.0.1:8080/nnwdaf-eventssubscription/v1/subscriptions \
  -H "Content-Type: application/json" \
  -d '{
    "eventSubscriptions": [{
      "event": "ABNORMAL_BEHAVIOUR",
      "tgtUe": {"anyUe": true},
      "excepRequs": [{"excepId": "SUSPICION_OF_DDOS_ATTACK"}]
    }],
    "notificationURI": "http://localhost:9090/callback"
  }'
```

**刪除訂閱**：
```bash
curl -X DELETE http://127.0.0.1:8080/nnwdaf-eventssubscription/v1/subscriptions/{subscriptionId}
```

---

## 下一步開發

1. **Phase 3: NRF 整合** - 向 NRF 註冊 NWDAF，讓其他 NF 可以發現
2. **Phase 4: Notification** - 實作主動推送通知到 `notificationURI`
3. **分析引擎** - 整合 DDoS 偵測模型
