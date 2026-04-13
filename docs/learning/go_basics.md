# Go 語言基礎概念 - NWDAF 專案導讀

本文件解釋在 NWDAF 專案中使用的 Go 語言基礎概念。

---

## 目錄

1. [Package 與 Import](#1-package-與-import)
2. [Struct（結構體）](#2-struct結構體)
3. [Methods 與 Receivers（方法接收者）](#3-methods-與-receivers方法接收者)
4. [Interface（介面）](#4-interface介面)
5. [Pointer（指標）](#5-pointer指標)
6. [Error Handling（錯誤處理）](#6-error-handling錯誤處理)

---

## 1. Package 與 Import

### Package（套件）宣告

每個 Go 檔案開頭都必須宣告它屬於哪個 package：

```go
package sbi  // 這個檔案屬於 "sbi" 套件
```

**規則**：
- 同一個資料夾內的所有 `.go` 檔案必須是同一個 package
- `main` package 是特殊的，它是程式進入點

### Import（引入）

```go
import (
    "fmt"           // 標準庫：格式化輸出
    "net/http"      // 標準庫：HTTP 相關常數與功能
    
    "github.com/gin-gonic/gin"           // 第三方套件：HTTP 框架
    "github.com/free5gc/openapi/models"  // free5gc 的資料模型
    
    "github.com/free5gc/nwdaf/internal/logger"  // 我們自己寫的套件
)
```

**引入後的使用**：
```go
http.StatusOK        // 使用 "net/http" 套件的 StatusOK 常數 (200)
gin.Context          // 使用 "gin" 套件的 Context 型別
models.ProblemDetails // 使用 "models" 套件的 ProblemDetails 型別
logger.SBILog        // 使用我們的 logger 套件
```

---

## 2. Struct（結構體）

Struct 是 Go 中組織資料的主要方式（類似其他語言的 class，但沒有繼承）。

### 定義 Struct

```go
// 定義一個 Server 結構體
type Server struct {
    httpServer *http.Server   // 欄位：HTTP 伺服器（指標）
    router     *gin.Engine    // 欄位：Gin 路由器（指標）
}
```

### 建立 Struct 實例

```go
// 方法 1：使用 & 建立並取得指標
s := &Server{
    router: gin.New(),
}

// 方法 2：先建立，再賦值
var s Server
s.router = gin.New()
```

### Struct Tags（結構標籤）

```go
type Sbi struct {
    Scheme      string `yaml:"scheme"`       // 這個標籤告訴 yaml 解析器對應的欄位名
    BindingIPv4 string `yaml:"bindingIPv4"`
    Port        int    `yaml:"port"`
}
```

YAML 檔案：
```yaml
scheme: http
bindingIPv4: 127.0.0.1
port: 8080
```

解析後會自動填入對應的 struct 欄位。

---

## 3. Methods 與 Receivers（方法接收者）

這是你問的 `func (s *Server) HandleCreateSubscription` 的關鍵概念！

### 什麼是 Method Receiver？

```go
func (s *Server) HandleCreateSubscription(c *gin.Context) {
//    ^^^^^^^^^^
//    這是 "receiver"，表示這個函數屬於 Server 型別
```

**解讀**：
- `s` 是變數名稱（習慣上用型別名稱的小寫首字母）
- `*Server` 表示這個方法屬於 `Server` 型別（的指標）
- 這讓你可以用 `s.xxx` 存取 Server 的欄位和其他方法

### 對比：普通函數 vs Method

```go
// 普通函數
func HandleRequest(server *Server, c *gin.Context) {
    // 需要明確傳入 server
}
// 呼叫：HandleRequest(myServer, ctx)

// Method（有 receiver）
func (s *Server) HandleRequest(c *gin.Context) {
    // s 自動是 Server 實例
}
// 呼叫：myServer.HandleRequest(ctx)
```

### 實際例子

```go
// server.go 中定義 Server struct
type Server struct {
    router *gin.Engine
}

// 這個 method 屬於 Server
func (s *Server) HandleCreateSubscription(c *gin.Context) {
    // 可以直接用 s 存取 Server 的東西
    s.Processor()   // 呼叫 Server 的另一個 method
    s.Config()      // 呼叫 Server 的 Config method
}
```

### 為什麼用 `*Server` 而不是 `Server`？

```go
func (s *Server) Method()   // 指標 receiver - 可以修改 s 的內容
func (s Server) Method()    // 值 receiver - 只能讀取，不能修改
```

大部分情況用指標 receiver，因為：
1. 可以修改 struct 的內容
2. 不需要複製整個 struct（效能較好）

---

## 4. Interface（介面）

Interface 定義了「能做什麼」，而不是「是什麼」。

```go
// 定義介面
type NwdafApp interface {
    CancelContext() context.Context
}

// 任何有 CancelContext() 方法的 struct 都「實作」了這個介面
type Processor struct {
    nwdaf NwdafApp  // 可以放任何實作 NwdafApp 的東西
}
```

**隱式實作**：Go 不需要明確宣告「我實作了這個介面」，只要有對應的方法就自動實作。

---

## 5. Pointer（指標）

### 基本概念

```go
var x int = 10

var p *int = &x   // p 是「指向 x 的指標」
                  // & 取得變數的記憶體位置

fmt.Println(*p)   // *p 取得指標指向的值，印出 10
```

### 在 Struct 中的使用

```go
// 定義
type Server struct {
    router *gin.Engine  // router 是「指向 gin.Engine 的指標」
}

// 使用
s := &Server{}          // &Server{} 建立 Server 並回傳其指標
s.router = gin.New()    // gin.New() 回傳 *gin.Engine
```

### 為什麼到處都是指標？

1. **避免複製大量資料**：傳指標只傳記憶體位址（8 bytes）
2. **共享狀態**：多處程式碼可以修改同一個物件
3. **可以是 nil**：用來表示「沒有值」

---

## 6. Error Handling（錯誤處理）

Go 沒有 try-catch，而是明確回傳 error：

```go
// 函數回傳值包含 error
func ReadConfig(path string) (*Config, error) {
    data, err := os.ReadFile(path)
    
    if err != nil {
        // 發生錯誤，回傳 nil 和錯誤訊息
        return nil, err
    }
    
    // 成功，回傳結果和 nil error
    return cfg, nil
}

// 呼叫方必須處理錯誤
cfg, err := ReadConfig("config.yaml")
if err != nil {
    log.Fatal(err)  // 處理錯誤
}
// 繼續使用 cfg...
```

---

## 快速對照表

| Go 語法 | 意義 | 範例 |
|---------|------|------|
| `*Type` | Type 的指標型別 | `*Server` |
| `&x` | 取得 x 的指標 | `&Server{}` |
| `*p` | 取得指標 p 指向的值 | `*p = 10` |
| `(s *Server)` | Method receiver | `func (s *Server) Foo()` |
| `s.Field` | 存取欄位/方法 | `s.router.GET(...)` |
| `:=` | 宣告並賦值 | `x := 10` |
| `var x Type` | 宣告變數 | `var s Server` |
