# Gin 框架使用指南 - NWDAF 專案導讀

Gin 是一個高效能的 Go HTTP 框架，我們用它來建立 NWDAF 的 REST API。

---

## 目錄

1. [Gin 是什麼？](#1-gin-是什麼)
2. [建立 Router](#2-建立-router)
3. [註冊路由](#3-註冊路由)
4. [gin.Context 詳解](#4-gincontext-詳解)
5. [請求處理流程](#5-請求處理流程)
6. [我們專案中的使用](#6-我們專案中的使用)

---

## 1. Gin 是什麼？

Gin 是一個 HTTP Web 框架，幫你處理：
- URL 路由（哪個網址對應哪個函數）
- 請求解析（讀取 JSON body、URL 參數）
- 回應發送（送出 JSON、設定 Header）

**類比**：
- Python → Flask / FastAPI
- Node.js → Express
- Go → **Gin**

---

## 2. 建立 Router

```go
import "github.com/gin-gonic/gin"

// 建立一個 Gin 引擎（router）
router := gin.New()

// 或使用預設設定（含日誌和錯誤恢復）
router := gin.Default()
```

**router** 是整個 HTTP 服務器的核心，負責：
1. 接收所有 HTTP 請求
2. 根據 URL 找到對應的處理函數
3. 執行處理函數

---

## 3. 註冊路由

### 基本語法

```go
router.GET("/path", handlerFunction)
router.POST("/path", handlerFunction)
router.PUT("/path", handlerFunction)
router.DELETE("/path", handlerFunction)
```

### 帶參數的路由

```go
// :subscriptionId 是 URL 參數
router.DELETE("/subscriptions/:subscriptionId", handleDelete)

// 在處理函數中取得參數
func handleDelete(c *gin.Context) {
    id := c.Param("subscriptionId")  // 取得 URL 中的 subscriptionId
}
```

### 路由群組

```go
// 建立一個群組，共享相同的前綴
group := router.Group("/nnwdaf-eventssubscription/v1")

// 註冊到群組
group.POST("/subscriptions", handleCreate)      // 完整路徑：/nnwdaf-eventssubscription/v1/subscriptions
group.DELETE("/subscriptions/:id", handleDelete) // 完整路徑：/nnwdaf-eventssubscription/v1/subscriptions/:id
```

---

## 4. gin.Context 詳解

`gin.Context` 是 Gin 中最重要的型別，包含了**一個 HTTP 請求的所有資訊**。

### 取得請求資料

```go
func handler(c *gin.Context) {
    // 1. 取得 URL 參數
    id := c.Param("subscriptionId")
    // 例：/subscriptions/abc123 → id = "abc123"
    
    // 2. 取得 Query 參數
    name := c.Query("name")
    // 例：/users?name=john → name = "john"
    
    // 3. 解析 JSON Body
    var req models.NnwdafEventsSubscription
    err := c.ShouldBindJSON(&req)
    // 會把 request body 的 JSON 解析到 req 變數
}
```

### 發送回應

```go
func handler(c *gin.Context) {
    // 1. 回傳 JSON
    c.JSON(http.StatusOK, gin.H{
        "message": "success",
    })
    // 回應：{"message": "success"}
    
    // 2. 回傳自定義 struct
    c.JSON(http.StatusCreated, response)
    
    // 3. 只回傳狀態碼（無 body）
    c.Status(http.StatusNoContent)  // 204
    
    // 4. 設定 Header
    c.Header("Location", "http://example.com/resource/123")
}
```

### gin.H 是什麼？

```go
// gin.H 只是 map[string]interface{} 的別名
gin.H{
    "key": "value",
    "number": 123,
}
// 等同於
map[string]interface{}{
    "key": "value",
    "number": 123,
}
```

---

## 5. 請求處理流程

當收到 HTTP 請求時：

```
Client
  │
  ▼ POST /nnwdaf-eventssubscription/v1/subscriptions
  │
┌─┴─────────────────────────────────────────────────┐
│                    Gin Router                      │
│                                                    │
│  1. 解析 URL                                       │
│  2. 找到匹配的路由：POST /subscriptions            │
│  3. 建立 gin.Context（包含 request 資訊）          │
│  4. 呼叫對應的處理函數                             │
└─┬─────────────────────────────────────────────────┘
  │
  ▼
┌─────────────────────────────────────────────────┐
│  HandleCreateSubscription(c *gin.Context)       │
│                                                  │
│  1. c.ShouldBindJSON(&req) - 解析 JSON body     │
│  2. 處理業務邏輯                                 │
│  3. c.JSON(201, response) - 回傳結果            │
└──────────────────────────────────────────────────┘
```

---

## 6. 我們專案中的使用

### server.go 中的路由設定

```go
func NewServer(nwdaf nwdafApp) (*Server, error) {
    s := &Server{
        router: gin.New(),  // 建立 router
    }
    
    // 設定中間件
    s.router.Use(gin.Recovery())  // 遇到 panic 時自動恢復
    
    // 註冊路由群組
    group := s.router.Group("/nnwdaf-eventssubscription/v1")
    
    // 註冊各個端點
    group.POST("/subscriptions", s.HandleCreateSubscription)
    group.PUT("/subscriptions/:subscriptionId", s.HandleUpdateSubscription)
    group.DELETE("/subscriptions/:subscriptionId", s.HandleDeleteSubscription)
    
    return s, nil
}
```

### api_eventssubscription.go 中的處理函數

```go
// 這是一個 Server 的 method（屬於 Server）
func (s *Server) HandleCreateSubscription(c *gin.Context) {
    
    // 1. 解析請求 body 的 JSON
    var req models.NnwdafEventsSubscription
    if err := c.ShouldBindJSON(&req); err != nil {
        // 解析失敗，回傳 400 錯誤
        c.JSON(http.StatusBadRequest, models.ProblemDetails{
            Status: http.StatusBadRequest,
            Cause:  "INVALID_JSON",
            Detail: err.Error(),
        })
        return  // 重要：要 return，不然會繼續執行
    }
    
    // 2. 處理業務邏輯（呼叫 Processor）
    response, subscriptionId, problemDetails := s.Processor().HandleCreateSubscription(&req)
    
    // 3. 檢查是否有錯誤
    if problemDetails != nil {
        c.JSON(int(problemDetails.Status), problemDetails)
        return
    }
    
    // 4. 成功，設定 Header 並回傳 201
    c.Header("Location", locationUri)
    c.JSON(http.StatusCreated, response)
}
```

---

## 常用 HTTP 狀態碼

| 常數 | 數值 | 意義 |
|------|------|------|
| `http.StatusOK` | 200 | 成功 |
| `http.StatusCreated` | 201 | 已建立新資源 |
| `http.StatusNoContent` | 204 | 成功但無回傳內容 |
| `http.StatusBadRequest` | 400 | 請求格式錯誤 |
| `http.StatusNotFound` | 404 | 找不到資源 |
| `http.StatusInternalServerError` | 500 | 伺服器內部錯誤 |

---

## 小結

記住這三點就能理解 Gin：

1. **Router** 負責 URL → Handler 的對應
2. **gin.Context** 包含請求的所有資訊，也用來發送回應
3. **Handler function** 接收 `*gin.Context`，處理請求並回應
