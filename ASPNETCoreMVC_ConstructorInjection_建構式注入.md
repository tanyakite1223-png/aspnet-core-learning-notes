# ASP.NET Core MVC 筆記：Constructor Injection（建構式注入）

## 一、是什麼

**Constructor Injection（建構式注入）** 是 DI（依賴注入）最常用、也是 ASP.NET Core 官方推薦的注入方式。

意思是：當 DI 容器要建立一個 class 的實例時，會透過它的 **建構子（Constructor）** 把這個 class 需要的服務（依賴項）「送進來」。

---

## 二、標準寫法（三步驟）

### 步驟 1：宣告 private readonly field 存放服務

```csharp
public class HomeController : Controller
{
    private readonly INotificationService _notificationService;
}
```

- `private`：只有這個 class 內部能用
- `readonly`：建構子賦值之後不能再改，避免被誤改
- 命名慣例：底線 `_` 開頭，例如 `_notificationService`

### 步驟 2：在 constructor 接收服務並賦值給 field

```csharp
public HomeController(INotificationService notificationService)
{
    _notificationService = notificationService;
}
```

- constructor 的參數列宣告需要的服務型別
- 把參數賦值給 field，之後整個 class 都能用

### 步驟 3：在 Action 中直接使用 field

```csharp
public IActionResult Notify(string x)
{
    return Content(_notificationService.Send(x));
}
```

- **不要在 Action 的參數列重複宣告 `_notificationService`**
- Action 參數列裡的東西會被 Model Binding 從網址拿值，跟注入無關

---

## 三、Field 與 Parameter 的作用範圍（Scope）

| 種類 | 宣告位置 | 作用範圍 |
|------|---------|---------|
| **Field（欄位）** | class 大括號內、Method 之外 | 整個 class 都能用 |
| **Parameter（參數）** | Method 的 `()` 裡面 | 只有該 Method 內部能用 |

**踩雷案例**：

```csharp
private readonly NotificationService _notificationService;  // ① class 的 field

public IActionResult Notify(string _notificationService)    // ② Method 的 parameter
{
    return Content("...");
}
```

雖然名字一樣，但 ① 跟 ② 是**完全不同的兩個東西**。② 只活在 Notify 方法內部、型別是 `string`，跟注入毫無關係。

---

## 四、為什麼要用 DI + Interface（核心精神）

### 沒有 Interface 時：緊耦合

```csharp
// Controller
private readonly NotificationService _notificationService;

public HomeController(NotificationService notificationService) { ... }

// Program.cs
builder.Services.AddScoped<NotificationService>();
```

**問題**：Controller 跟具體 class `NotificationService` 綁死。如果要換成 `LineNotificationService`，**Controller 跟 Program.cs 都要改**。

### 有 Interface 時：鬆耦合

```csharp
// Controller
private readonly INotificationService _notificationService;

public HomeController(INotificationService notificationService) { ... }

// Program.cs
builder.Services.AddScoped<INotificationService, LineNotificationService>();
```

**好處**：要換通知方式，**只改 Program.cs 一行**，Controller 完全不動。

### Interface 的角色：制定規範

> Interface 制定規範 → 多個實作遵守規範 → Controller 只認規範，不認實作 → 所以實作可以隨時換，Controller 不用動

---

## 五、完整範例

### 1. 介面

`Services/INotificationService.cs`

```csharp
namespace MyMvcApp.Services
{
    public interface INotificationService
    {
        string Send(string message);
    }
}
```

📌 **命名慣例**：C# interface 習慣以大寫 `I` 開頭。

### 2. 實作 A

`Services/NotificationService.cs`

```csharp
namespace MyMvcApp.Services
{
    public class NotificationService : INotificationService
    {
        public string Send(string message)
        {
            return $"[通知已發送] {message}";
        }
    }
}
```

### 3. 實作 B

`Services/LineNotificationService.cs`

```csharp
namespace MyMvcApp.Services
{
    public class LineNotificationService : INotificationService
    {
        public string Send(string message)
        {
            return $"[LINE 推播] {message}";
        }
    }
}
```

### 4. 註冊（Program.cs）

```csharp
builder.Services.AddScoped<INotificationService, LineNotificationService>();
//                         ↑ 規範             ↑ 實際要給的具體 class
```

### 5. Controller 使用

`Controllers/HomeController.cs`

```csharp
using MyMvcApp.Services;

public class HomeController : Controller
{
    private readonly INotificationService _notificationService;

    public HomeController(INotificationService notificationService)
    {
        _notificationService = notificationService;
    }

    public IActionResult Notify(string x)
    {
        return Content(_notificationService.Send(x));
    }
}
```

訪問 `/Home/Notify?x=訂單已成立` → 顯示 `[LINE 推播] 訂單已成立`

要換回原本的通知服務？只改 Program.cs 那一行：
```csharp
builder.Services.AddScoped<INotificationService, NotificationService>();
```

---

## 六、檔案組織原則

### 一個檔案放一個 class

```
Services/
├─ INotificationService.cs      （介面）
├─ NotificationService.cs       （實作 A）
└─ LineNotificationService.cs   （實作 B）
```

**為什麼不要塞在一起？**

| 原因 | 說明 |
|------|------|
| 方便尋找 | 檔名 = class 名，一眼就知道在哪 |
| Git 衝突少 | 不同人改不同檔案，不會互相干擾 |
| 職責分離 | 單一職責原則（SRP），每個檔案只負責一件事 |
| 業界共識 | 台灣與全世界的 .NET 團隊都這樣做 |

**例外**（合理可放同檔）：Nested class、極小的 Enum/DTO、Partial class（EF Core 自動產生的）

---

## 七、常見錯誤

### 錯誤 1：把 field 跟 parameter 搞混

```csharp
// 錯：在 Action 參數列重新宣告同名變數
public IActionResult Notify(string _notificationService)
{
    return Content("...");  // 沒有用到注入的服務
}
```

**正確做法**：Action 不要在 `()` 寫 `_notificationService`，直接使用 class 的 field。

### 錯誤 2：用「改名 + 改內容」假裝換實作

```csharp
// 錯：把 NotificationService 內部改成 LINE 邏輯，但名字沒變
public class NotificationService : INotificationService
{
    public string Send(string message)
    {
        return $"[LINE 推播] {message}";  // 名字叫 NotificationService，內容卻是 LINE
    }
}
```

**問題**：名字跟內容不一致，後續維護的人會崩潰。**正確做法**是新增一個 `LineNotificationService` class，透過 Program.cs 切換。

### 錯誤 3：忘記在 Program.cs 註冊

服務沒註冊，Controller 啟動時會直接拋錯：
```
Unable to resolve service for type 'XXX' while attempting to activate 'YYY'.
```

**檢查清單**：
1. Program.cs 有沒有 `AddScoped<介面, 實作>()`？
2. Controller 的 constructor 參數型別跟註冊的介面一致嗎？

---

## 八、Debug 心得（這次學到的觀念）

當錯誤訊息指向某個檔案時，**真正的問題往往不在那個檔案本身，而在「餵資料給它的地方」**。

例如 View（`Index.cshtml`）爆 `NullReferenceException`，根因可能是 Controller 沒有把 Model 傳給 View。

---

## 九、本次重點整理

| 觀念 | 重點 |
|------|------|
| Constructor Injection | field + constructor + 賦值，三步驟 |
| Field vs Parameter | field 整個 class 能用，parameter 只在 Method 裡能用 |
| Interface 的角色 | 制定規範，讓 Controller 不依賴具體 class |
| 緊耦合 vs 鬆耦合 | 看「綁的是具體實作還是介面」 |
| DI 的威力 | 換實作只改 Program.cs 一行 |
| 一個檔案一個 class | C# 業界共識，方便維護 |
