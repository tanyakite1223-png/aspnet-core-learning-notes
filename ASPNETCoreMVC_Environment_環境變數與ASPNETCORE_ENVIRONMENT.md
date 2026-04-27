# ASP.NET Core MVC 筆記:環境變數與 ASPNETCORE_ENVIRONMENT

## 一、核心觀念

### 1. 什麼是 `ASPNETCORE_ENVIRONMENT`?

`ASPNETCORE_ENVIRONMENT` 是一個**環境變數(Environment Variable)**,用來告訴 ASP.NET Core 應用程式:「**現在是在什麼環境執行?**」。

常見的三個值:

| 值 | 用途 | 對應檔案 |
|------|------|----------|
| `Development` | 開發環境(本機開發中) | `appsettings.Development.json` |
| `Staging` | 預備環境(上線前測試) | `appsettings.Staging.json` |
| `Production` | 正式環境(已上線) | `appsettings.Production.json` |

> ⚠️ **大小寫敏感**:`Production` 和 `production` 是不同的,檔名也要對應。

---

### 2. 設定檔是「層層疊加」的,不是「替換」

ASP.NET Core 啟動時讀取設定檔的順序:

```
第 1 步:讀 appsettings.json         (基礎設定,所有環境都讀)
第 2 步:讀 appsettings.{Env}.json   (只讀當前環境的)
最後:後讀的「覆蓋」先讀的同名設定值
```

**舉例**:

```json
// appsettings.json
{
  "MailSettings": {
    "MailHost": "smtp.gmail.com",
    "MailPort": 587,
    "SenderName": "Amber 的購物網站"
  }
}

// appsettings.Production.json
{
  "MailSettings": {
    "SenderName": "Amber 的購物網站(正式環境)"
  }
}
```

當 `ASPNETCORE_ENVIRONMENT=Production` 時,實際生效的設定:

| 項目 | 值 | 來源 |
|------|------|------|
| MailHost | `smtp.gmail.com` | appsettings.json(Production 沒寫,沿用基礎) |
| MailPort | `587` | 同上 |
| SenderName | `Amber 的購物網站(正式環境)` | **被 Production 覆蓋** ✅ |

> 💡 **設計意義**:共用設定寫一次,只在環境檔寫「不同的部分」,避免重複。

---

## 二、`ASPNETCORE_ENVIRONMENT` 從哪裡來?

ASP.NET Core 啟動時,依以下**優先順序**尋找(數字越小優先級越高):

| 順序 | 來源 | 使用情境 |
|------|------|----------|
| 1 | `launchSettings.json` 的 `environmentVariables` | **VS / dotnet run 開發時** |
| 2 | 作業系統的環境變數 | **正式機部署時最常用** |
| 3 | IIS 網站設定 / `web.config` | IIS 部署時 |
| 4 | 預設值 = `Production` | 都沒設時的後備 |

**核心邏輯**:**找到就停、找不到才往下找**(這就是 fallback chain / 後備鏈設計模式)

---

### 1. launchSettings.json(開發專用)

位置:`Properties/launchSettings.json`

```json
{
  "profiles": {
    "http": {
      "commandName": "Project",
      "applicationUrl": "http://localhost:5272",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "https": {
      "commandName": "Project",
      "applicationUrl": "https://localhost:7142;http://localhost:5272",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

**重要特性**:

- ⚠️ **只有 VS / dotnet run 啟動時會讀取**
- ⚠️ **部署到正式機(IIS)後完全不會讀**
- ✅ 改完直接 F5 即可生效,**不需要重啟 VS**

---

### 2. 系統環境變數(正式機常用)

設定方式(Windows):

```
Win + R → 輸入 sysdm.cpl → 進階 → 環境變數
   ↓
在「使用者變數」或「系統變數」新增:
   變數名稱:ASPNETCORE_ENVIRONMENT
   變數值:Staging(或 Production)
```

**重要特性**:

- ⚠️ **必須完全關閉 VS 再重開**,環境變數才會被新 process 讀到
- ✅ 部署到正式機後,**不會被 launchSettings.json 干擾**(因為 launchSettings 不會被讀)

---

## 三、在程式碼裡判斷環境

### 1. 注入 `IWebHostEnvironment`

```csharp
public class HomeController : Controller
{
    private readonly IWebHostEnvironment _webHostEnvironment;

    public HomeController(IWebHostEnvironment webHostEnvironment)
    {
        _webHostEnvironment = webHostEnvironment;
    }

    public IActionResult Index()
    {
        // 取得目前環境名稱(字串)
        ViewBag.EnvironmentName = _webHostEnvironment.EnvironmentName;
        return View();
    }
}
```

### 2. 用 `IsDevelopment()` / `IsStaging()` / `IsProduction()` 判斷

```csharp
if (_webHostEnvironment.IsDevelopment())
{
    ViewBag.Message = "🔧 你正在開發環境,可以開啟詳細錯誤頁面";
}
else if (_webHostEnvironment.IsStaging())
{
    ViewBag.Message = "🧪 你在預備環境(Staging),這裡用來做上線前測試";
}
else if (_webHostEnvironment.IsProduction())
{
    ViewBag.Message = "🚀 你在正式環境(Production),要小心操作";
}
```

### 3. 實務應用(常見情境)

```csharp
// 開發環境顯示詳細錯誤頁面,正式環境不顯示
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Home/Error");
}
```

---

## 四、實驗驗證(優先順序)

### 實驗設定

- 系統環境變數:`ASPNETCORE_ENVIRONMENT = Staging`

### 三種情境結果

| 情境 | launchSettings.json | EnvironmentName 結果 | 原因 |
|------|---------------------|----------------------|------|
| 1 | `Development` | **Development** | 第 1 層找到 → 停止 |
| 2 | `Production` | **Production** | 第 1 層找到 → 停止 |
| 3 | (整個 environmentVariables 區塊刪除) | **Staging** | 第 1 層跳過,第 2 層找到 |

> 🎯 **核心結論**:`launchSettings.json` 的設定**會蓋過**作業系統環境變數,因為它優先級最高。

---

## 五、常見錯誤

### 錯誤 1:以為 `appsettings.{Env}.json` 會「取代」`appsettings.json`

❌ **錯誤觀念**:
```
appsettings.Production.json 沒寫 MailHost → MailHost 不存在
```

✅ **正確觀念**:
```
appsettings.Production.json 沒寫 MailHost
   → 沿用 appsettings.json 的 MailHost
   → MailHost 仍然存在(層層疊加,不是替換)
```

---

### 錯誤 2:把 `ASPNETCORE_ENVIRONMENT` 寫進 `appsettings.json`

❌ **這在邏輯上不通**:

```json
// appsettings.json(錯誤示範)
{
  "ASPNETCORE_ENVIRONMENT": "Production"
}
```

**為什麼錯?**

`ASPNETCORE_ENVIRONMENT` 是用來決定「**要讀哪個 appsettings 檔**」的開關。
如果它寫在 appsettings.json 裡 → 「讀 A 才知道要讀哪個 A」→ 邏輯死循環。

> ✅ **正確做法**:環境變數**必須來自應用程式外部**(OS、IIS、launchSettings),不能來自程式自己的設定檔。

---

### 錯誤 3:設定系統環境變數後沒重啟 VS

❌ 設好 `ASPNETCORE_ENVIRONMENT=Staging` 後直接在現有 VS 按 F5 → **不會生效**

✅ **必須完全關閉 VS 再重開** → 環境變數會在新 process 啟動時被讀進來

> 💡 原因:環境變數在 process 啟動時被讀進去,已經啟動的 process 不會自動感應到新設定。

---

### 錯誤 4:刪除 launchSettings.json 的 environmentVariables 但留下逗號

❌ 錯誤的 JSON 格式:
```json
"http": {
  "applicationUrl": "http://localhost:5272",   ← 後面這個逗號要拿掉
}
```

✅ 正確:
```json
"http": {
  "applicationUrl": "http://localhost:5272"   ← 沒有後續欄位,逗號要去掉
}
```

> 💡 JSON 格式規定:最後一個欄位後面**不能有逗號**。

---

### 錯誤 5:以為「沒設定環境變數」就是 Development

❌ **錯誤觀念**:沒設環境變數就是 Development(開發中)

✅ **正確觀念**:沒設環境變數時,**.NET 預設值是 `Production`**

> 💡 這也是為什麼正式機如果忘記設環境變數,程式還是會「正常」當成正式環境執行 — 不會掉到開發模式。

---

## 六、設計精神總結

```
✦ 程式碼:一份(同一個 publish 包)
✦ 設定檔:多份(appsettings.json + appsettings.Development.json + appsettings.Production.json ...)
✦ 環境變數:決定「現在要讀哪個 appsettings」
   ↓
同一份程式可以部署到多個環境,自動讀取對應設定 ✅
```

**實務情境**:

```
公司有三台伺服器:
├── dev.company.com    ← 系統環境變數 = Development
├── staging.company.com ← 系統環境變數 = Staging
└── www.company.com    ← 系統環境變數 = Production

開發人員寫好同一份程式碼 → 同一個 publish 包 → 部署到三台
   ↓
每台伺服器的環境變數不同 → 自動讀對應的 appsettings 檔案
   ↓
三台伺服器自動有不同的設定行為(資料庫連線、日誌等級、SMTP 設定...)
```

---

## 七、延伸觀念:Fallback Chain(後備鏈)

「**逐層往下找,找到就用**」這個設計模式在很多地方都會看到:

| 情境 | 找哪些地方 |
|------|-----------|
| ASP.NET Core 找環境變數 | launchSettings → OS env → IIS → 預設值 |
| ASP.NET Core 找設定值 | appsettings.json → appsettings.{Env}.json → 環境變數 → 命令列參數 |
| 瀏覽器找 CSS 樣式 | inline style → `<style>` → 外部 CSS → 瀏覽器預設 |
| 程式語言找變數 | 區域變數 → 類別欄位 → 全域變數 |

> 🎯 認識這個觀念,以後遇到「優先順序」相關問題會更直覺地理解。

---

## 八、快速速查表

| 問題 | 答案 |
|------|------|
| 如何在程式取得目前環境名稱? | `IWebHostEnvironment.EnvironmentName` |
| 如何判斷是不是開發環境? | `IWebHostEnvironment.IsDevelopment()` |
| 開發時改環境最簡單的方法? | 改 `Properties/launchSettings.json` 的 `ASPNETCORE_ENVIRONMENT` |
| 正式機改環境的方法? | 設作業系統的 `ASPNETCORE_ENVIRONMENT` 環境變數 |
| 沒設環境變數預設是什麼? | `Production` |
| `appsettings.Production.json` 沒寫的設定怎麼辦? | 自動沿用 `appsettings.json` 的值 |
| 改系統環境變數後 VS 沒反應? | 完全關閉 VS 再重開 |
