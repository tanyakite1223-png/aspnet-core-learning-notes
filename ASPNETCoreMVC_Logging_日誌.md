# ASP.NET Core MVC 筆記:Logging(日誌)

## 一、為什麼需要 Logging?

Logging(日誌)就像程式的**行車記錄器** 📹

當程式上線後出問題時,使用者可能只回報「網站壞了」這種模糊的訊息。如果程式平常有「自動寫下發生的事」,工程師就能事後翻紀錄,快速定位:

- 什麼時候發生的?
- 是哪個功能出錯?
- 出錯時的狀況是什麼?

**Logging 是程式正式上線後最重要的「除錯依據」。**

---

## 二、Log Level(日誌等級)

### 為什麼要分等級?

如果什麼都記,會有三個問題:
1. **效能問題**:每寫一筆紀錄都花時間
2. **儲存空間問題**:紀錄檔越來越大
3. **找問題變難**:幾百萬筆裡找一筆,大海撈針

所以日誌要**分等級**,就像醫院急診室分輕重症一樣。

### 6 種等級對照表

| 等級 | 名稱 | 白話解釋 | 範例 |
|------|------|---------|------|
| 0 | **Trace** | 「我經過這裡」 | 進入 GetUser 方法 |
| 1 | **Debug** | 「這個變數現在是 X」 | user.Id = 123 |
| 2 | **Information** | 「發生了一件值得記錄的事」 | 使用者 amber 登入成功 |
| 3 | **Warning** | 「有點怪,但還沒壞」 | API 即將被廢棄、SMTP 重試後成功 |
| 4 | **Error** | 「壞了!但只壞一個功能」 | 信用卡扣款失敗 |
| 5 | **Critical** | 「全部都壞了!」 | 資料庫整個掛掉 |

### 判斷訣竅

| 問自己 | 答案 | Log Level |
|--------|------|-----------|
| 詳細到「跑到哪一行」嗎? | 是 | Trace |
| 變數值之類的除錯細節? | 是 | Debug |
| 有意義的業務事件?(老闆會想知道) | 是 | Information |
| 怪怪的但還能跑? | 是 | Warning |
| 涉及安全性或異常行為? | 是 | Warning |
| 功能失敗了? | 是 | Error |
| 整個系統完蛋? | 是 | Critical |
| 正常的使用者操作(例如表單驗證) | — | **不要記** |

---

## 三、ASP.NET Core 預設就有 Logging

### 重點觀念

打開新建立的 ASP.NET Core MVC 專案,**Program.cs 看不到任何 Logging 相關程式碼**。

但這**不代表沒有 Logging** — 因為 `WebApplication.CreateBuilder(args)` 這一行,**已經幫妳預設裝好 Logging 了**!

預設做了:
- ✅ 註冊 `ILogger<T>` 到 DI 容器
- ✅ 加上輸出來源(Console、Debug 視窗、EventSource)
- ✅ 讀取 `appsettings.json` 的 Logging 設定

換句話說:**妳什麼都不做,Logging 已經就緒**。

---

## 四、在 Controller 中使用 Logger

### 標準寫法

```csharp
public class HomeController : Controller
{
    // 1. 宣告 private readonly 欄位
    private readonly ILogger<HomeController> _logger;

    // 2. 透過建構式注入 (DI)
    public HomeController(ILogger<HomeController> logger)
    {
        _logger = logger;
    }

    public IActionResult Index()
    {
        // 3. 使用 _logger 寫日誌
        _logger.LogInformation("使用者進入首頁");
        return View();
    }
}
```

### 6 種 Log 方法

```csharp
_logger.LogTrace("這是 Trace 訊息");
_logger.LogDebug("這是 Debug 訊息");
_logger.LogInformation("這是 Information 訊息");
_logger.LogWarning("這是 Warning 訊息");
_logger.LogError("這是 Error 訊息");
_logger.LogCritical("這是 Critical 訊息");
```

---

## 五、為什麼要寫 `ILogger<T>`?(關鍵概念)

### ❌ 錯誤寫法

```csharp
private readonly ILogger _logger;

public HomeController(ILogger logger)
{
    _logger = logger;
}
```

執行會出現 **HTTP 500 錯誤**:
```
System.InvalidOperationException: Unable to resolve service for type 
'Microsoft.Extensions.Logging.ILogger' while attempting to activate 
'MyMvcApp.Controllers.HomeController'.
```

### ✅ 正確寫法

```csharp
private readonly ILogger<HomeController> _logger;

public HomeController(ILogger<HomeController> logger)
{
    _logger = logger;
}
```

### 為什麼?

ASP.NET Core 註冊到 DI 容器的是 **`ILogger<T>`(泛型版本)**,**不是** `ILogger`。

### `<T>` 的好處:自動加上「分類標籤」

```
trce: MyMvcApp.Controllers.HomeController[0]   ← 這個就是 <T> 帶來的
      這是一條 Trace 級別的日誌
```

當網站有很多 Controller 時,Console 訊息可以一眼辨識來源:

```
info: MyMvcApp.Controllers.HomeController[0]
      使用者進入首頁
fail: MyMvcApp.Controllers.PaymentController[0]
      信用卡扣款失敗      ← 一眼看出是 Payment 出事!
```

### 套用範圍

**每一支 Controller、每一個 Service** 想用 logger,都要各自寫 `ILogger<自己>`:

```csharp
// HomeController
private readonly ILogger<HomeController> _logger;

// ProductController
private readonly ILogger<ProductController> _logger;

// NotificationService
private readonly ILogger<NotificationService> _logger;
```

雖然看起來重複,但這就是「**有意義的樣板程式碼**」 — 每個類別都有自己的身份。

---

## 六、Log Level 篩選門檻

### 預設行為

ASP.NET Core 預設**只印 Information 以上**的訊息。

意思是 — 即使妳寫了 `_logger.LogTrace(...)`,**Console 看不到**。因為被預設門檻過濾掉了。

### 在哪裡設定門檻?

`appsettings.json`:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  }
}
```

- `Default`:整體預設門檻
- `Microsoft.AspNetCore`:框架本身訊息的門檻(設高一點避免雜訊)

### 門檻篩選邏輯

```
門檻設成 "Warning"
↓
等級 < Warning 的:被過濾(Trace、Debug、Information ❌)
等級 ≥ Warning 的:會印出(Warning、Error、Critical ✅)
```

### 完整對照表

| 門檻設定 | Trace | Debug | Information | Warning | Error | Critical |
|---------|:-----:|:-----:|:-----------:|:-------:|:-----:|:--------:|
| `Trace` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| `Debug` | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **`Information`(預設)** | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| `Warning` | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| `Error` | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| `Critical` | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

---

## 七、結合 Configuration 的環境覆寫機制

回想 Configuration 章節學過的「**環境檔覆寫**」:

| 檔案 | 作用 |
|------|------|
| `appsettings.json` | 基底設定,所有環境共用 |
| `appsettings.Development.json` | 開發環境覆寫 |
| `appsettings.Production.json` | 正式環境覆寫 |

### 應用到 Logging:不同環境用不同門檻

**`appsettings.Development.json`**(開發環境 — 看細節):
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Trace"
    }
  }
}
```

**`appsettings.Production.json`**(正式環境 — 只記重要的):
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  }
}
```

### 真實世界的建議門檻

| 環境 | 建議門檻 | 為什麼 |
|------|---------|--------|
| **Development(開發)** | `Trace` 或 `Debug` | 看越多細節越好,方便除錯 |
| **Staging(測試)** | `Information` | 看主要事件流程 |
| **Production(正式)** | `Warning` 或 `Error` | 只關心問題,避免日誌爆量 |

---

## 八、Console 視窗的視覺辨識

ASP.NET Core 在 Console 顯示 Log 時,會用**標籤縮寫**和**顏色**幫忙辨識:

| 等級 | 標籤 | 顏色 |
|------|------|------|
| Trace | `trce:` | 灰色 |
| Debug | `dbug:` | 灰色 |
| Information | `info:` | 綠色 |
| Warning | `warn:` | 黃色 |
| Error | `fail:` | 紅底 |
| Critical | `crit:` | 紅底加亮 |

掃一眼 Console 就能找到問題 — 紅色那行通常就是出事的地方!

---

## 九、常見錯誤(Common Errors)

### 錯誤 1:把 LogXXX 跟「畫面」搞混

❌ **誤解**:`_logger.LogTrace("...")` 會印在瀏覽器網頁上
✅ **正確**:Log 訊息只會出現在 **Console 視窗**、**Visual Studio 輸出視窗**、**檔案** 等地方,**永遠不會出現在使用者看到的瀏覽器頁面**

| | 印在「畫面」上 | 印在「Log」上 |
|---|---|---|
| 誰看得到 | 使用者(瀏覽器) | 工程師、維運人員 |
| 怎麼寫 | `return View()`、`ViewBag.X = ...` | `_logger.LogXXX(...)` |

### 錯誤 2:注入 `ILogger`(沒加 `<T>`)

❌ **錯誤寫法**:
```csharp
public HomeController(ILogger logger)  // 會 500 錯誤
```

✅ **正確寫法**:
```csharp
public HomeController(ILogger<HomeController> logger)
```

**口訣**:用在哪個類別,`<T>` 就填哪個類別。

### 錯誤 3:改了 appsettings 沒重啟專案

❌ **狀況**:把 `"Default": "Trace"` 改完後,Trace 訊息還是沒印出來
✅ **原因**:appsettings 是程式啟動時讀取的,改完要**停止 → Ctrl+F5 重啟**

### 錯誤 4:把所有正常事件都記成 log

❌ **不該記**:
- 使用者輸入密碼長度不足(表單驗證,在前端就處理)
- 每次點按鈕都記一筆

✅ **該記**:
- 登入失敗(可能是攻擊)→ Warning
- 訂單建立成功(業務事件)→ Information
- 系統錯誤 → Error / Critical

**判斷原則**:**「這件事我之後會回頭翻嗎?」** 會 → 記。不會 → 不記。

### 錯誤 5:改錯 appsettings 檔案

例如想調 Development 環境的門檻,卻改到了 `appsettings.json`(全環境基底)。

✅ 確認一下 — 如果是要**只改開發環境**,要改 `appsettings.Development.json`。
