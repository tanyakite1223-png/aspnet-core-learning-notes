# ASP.NET Core MVC 筆記：appsettings.json 與環境設定檔

---

## 一、appsettings.json 是什麼？

`appsettings.json` 是 ASP.NET Core 專案的**組態設定檔（Configuration File）**，用來存放各種設定值，例如：
- 資料庫連線字串
- Email 設定
- 網站名稱
- 第三方服務的 API 金鑰

---

## 二、環境設定檔

專案裡預設有兩個設定檔：

| 檔案 | 用途 |
|------|------|
| `appsettings.json` | 所有環境共用的基本設定 |
| `appsettings.Development.json` | 開發環境專用，會**覆蓋**主檔的同名設定 |

正式上線時可新增 `appsettings.Production.json`，放正式環境的設定。

### 運作邏輯

```
ASPNETCORE_ENVIRONMENT = "Development"
         ↓
自動載入 appsettings.json
         +
自動載入 appsettings.Development.json（覆蓋同名設定）
```

---

## 三、ASPNETCORE_ENVIRONMENT 環境變數

ASP.NET Core 靠 **`ASPNETCORE_ENVIRONMENT`** 這個環境變數（Environment Variable）判斷目前的執行環境。

設定位置：`Properties/launchSettings.json`

```json
"environmentVariables": {
  "ASPNETCORE_ENVIRONMENT": "Development"
}
```

> ⚠️ 注意：用 `.csproj` 直接開啟專案執行，不會套用 `launchSettings.json`，環境會預設變成 `Production`。要用 `.slnx`（方案檔）開啟，才能選擇正確的啟動設定。

---

## 四、IConfiguration — 讀取設定值

### 注入方式

`IConfiguration` 由 ASP.NET Core 自動註冊，直接用**建構式注入（Constructor Injection）**取得：

```csharp
private readonly IConfiguration _configuration;

public HomeController(IConfiguration configuration)
{
    _configuration = configuration;
}
```

### 讀取一般設定值

```csharp
var siteName = _configuration["SiteName"];
```

### 讀取巢狀設定值

用 `:` 分隔層級：

```csharp
var logLevel = _configuration["Logging:LogLevel:Default"];
// 回傳 "Information"
```

對應的 `appsettings.json` 結構：

```json
"Logging": {
  "LogLevel": {
    "Default": "Information"
  }
}
```

---

## 五、Options Pattern — 強型別設定

### 為什麼需要 Options Pattern？

用 `IConfiguration` 一個一個讀取的缺點：
- 程式碼冗長，不易維護
- 回傳值都是**字串**，數字還要自己轉型
- key 打錯也不會在編譯時報錯

### 實作步驟

**Step 1：建立對應的 Class**

```csharp
// Models/MailSettings.cs
public class MailSettings
{
    public string Host { get; set; }
    public int Port { get; set; }
    public string SenderName { get; set; }
}
```

**Step 2：在 appsettings.json 加入設定**

```json
"MailSettings": {
  "Host": "smtp.gmail.com",
  "Port": 587,
  "SenderName": "Amber 的購物網站"
}
```

**Step 3：在 Program.cs 註冊**

```csharp
builder.Services.Configure<MailSettings>(
    builder.Configuration.GetSection("MailSettings"));
```

**Step 4：在 Controller 注入使用**

```csharp
using Microsoft.Extensions.Options;

private readonly MailSettings _mailSettings;

public HomeController(IOptions<MailSettings> mailOptions)
{
    _mailSettings = mailOptions.Value;  // .Value 取出實際物件
}

public IActionResult Index()
{
    ViewBag.Host = _mailSettings.Host;
    ViewBag.Port = _mailSettings.Port;        // 直接是 int，不用轉型
    ViewBag.SenderName = _mailSettings.SenderName;
    return View();
}
```

### 兩種寫法比較

| | IConfiguration | Options Pattern |
|---|---|---|
| **寫法** | `_configuration["MailSettings:Port"]` | `_mailSettings.Port` |
| **回傳型別** | 字串 `"587"` | 數字 `587` |
| **IntelliSense** | 無提示 | ✅ 有提示 |
| **適合情境** | 讀取單一零散設定值 | 一組相關設定 |

---

## 六、常見錯誤

### ❌ 建構式有兩個，造成 DI 衝突
```csharp
// 錯誤：兩個建構式
public HomeController(IConfiguration configuration) { ... }
public HomeController(INotificationService service) { ... }
```
✅ 正確：所有參數放在同一個建構式
```csharp
public HomeController(IConfiguration configuration, INotificationService service)
{
    _configuration = configuration;
    _notificationService = service;
}
```

### ❌ 用錯讀取方法
```csharp
// 錯誤：GetConnectionString 是專門給連線字串用的
_configuration.GetConnectionString("SiteName");
```
✅ 正確：讀取一般設定值用索引器
```csharp
_configuration["SiteName"];
```

### ❌ ViewBag 名稱不一致
```csharp
// Controller 設定
ViewBag.Host = "smtp.gmail.com";
```
```html
<!-- View 裡打錯名稱，什麼都不會顯示 -->
@ViewBag.MailHost
```
✅ 正確：Controller 和 View 的名稱必須完全一致
```html
@ViewBag.Host
```

### ❌ Options Pattern 注入後忘記 .Value
```csharp
// 錯誤：_mailSettings 型別是 IOptions<MailSettings>，不是 MailSettings
private readonly IOptions<MailSettings> _mailSettings;
_mailSettings = mailOptions;
```
✅ 正確：用 .Value 取出實際物件
```csharp
private readonly MailSettings _mailSettings;
_mailSettings = mailOptions.Value;
```
