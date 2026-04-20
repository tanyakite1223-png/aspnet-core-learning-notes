# ASP.NET Core MVC 筆記：服務註冊 — Transient、Scoped、Singleton

## 一、服務註冊是什麼

**服務註冊（Service Registration）** 就是告訴 IoC 容器（IoC Container）：

> 「嘿,以後有人需要 XXX,你就給他一個 YYY 的實例。」

這是 DI（Dependency Injection,依賴注入）運作的**起點**。沒有註冊,容器就不知道怎麼建立服務。

### 服務註冊的位置

所有服務都註冊在 `Program.cs` 裡,透過 `builder.Services` 這個**服務容器**來操作。

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllersWithViews();  // ← MVC 本身的服務
builder.Services.AddScoped<IShoppingCartService, ShoppingCartService>();  // ← 自訂服務
```

- `builder.Services` 的型別是 `IServiceCollection`
- 它就是 **IoC 容器**,負責記錄所有服務與其對應的實作、生命週期

---

## 二、註冊的兩種寫法

### ❌ 寫法 A：只有具體類別(不推薦)

```csharp
builder.Services.AddScoped<ShoppingCartService>();
```

**問題**：
- Controller 的建構子會被迫寫 `HomeController(ShoppingCartService cartService)`
- **直接依賴具體類別**,沒有解決 DI 最初要解決的問題(緊耦合、難測試)

### ✅ 寫法 B：介面對應實作(標準做法)

```csharp
builder.Services.AddScoped<IShoppingCartService, ShoppingCartService>();
//                         ↑ 介面              ↑ 實作類別
```

**翻譯成白話**：「容器啊,以後有人要 `IShoppingCartService`,你就給他一個 `ShoppingCartService` 的實例。」

**優點**：
1. **好測試**：測試時可以塞假的實作(例如 `FakeShoppingCartService`)進去
2. **好替換**：哪天要換成新的實作,只要改 `Program.cs` 一行,Controller 完全不用動
3. **真正鬆耦合**:Controller 只跟「合約(介面)」打交道,不跟「實作」打交道

---

## 三、三種生命週期(Lifetime)

| 註冊方法 | 中文 | 何時產生新實例 |
|---------|------|--------------|
| `AddTransient<T>()` | 暫時性 | 每次被要求就 new 一個全新的 |
| `AddScoped<T>()` | 範圍內 | 每個 HTTP Request 共用同一個 |
| `AddSingleton<T>()` | 單例 | 整個應用程式只有一個 |

### 🛒 購物網站情境對照

- **Transient(暫時性)** = **訂單編號產生器**:每次呼叫都要產生全新的、獨立的編號,不可共用
- **Scoped(範圍內)** = **購物車服務**:同一次 Request 內的多個步驟(確認庫存 → 計算金額 → 寫入購物車)共用同一個購物車物件;不同 Request 之間則是全新的
- **Singleton(單例)** = **匯率查詢服務**:網站啟動時載入一次,之後所有使用者共用同一份匯率資料,直到網站重啟

---

## 四、三種生命週期 — 實驗總結

### 實驗設計

讓 `ShoppingCartService` 在建構子中產生唯一 `Guid`,觀察不同生命週期下的編號行為:

```csharp
public class ShoppingCartService : IShoppingCartService
{
    private readonly Guid _id;

    public ShoppingCartService()
    {
        _id = Guid.NewGuid();
    }

    public string AddItem(string productName)
    {
        return $"[實例編號:{_id}] 已將 {productName} 加入購物車";
    }
}
```

Controller 注入**兩次**同樣的服務,看容器給的是同一個還是不同的實例:

```csharp
public HomeController(
    IShoppingCartService cartService1,
    IShoppingCartService cartService2)
{
    _cartService1 = cartService1;
    _cartService2 = cartService2;
}
```

### 預期行為對照表

| 生命週期 | 同一次 Request 內 | 不同 Request 之間 | 何時產生新實例 |
|---------|-----------------|-----------------|---------------|
| **Transient** | 不同 | 不同 | 每次被要求就 new |
| **Scoped** | 相同 | 不同 | 每個 Request 一個 |
| **Singleton** | 相同 | 相同 | 整個程式只有一個 |

### 🔬 實驗紀錄(親手驗證的編號)

**[Transient] `builder.Services.AddTransient<IShoppingCartService, ShoppingCartService>();`**
- 實例 1:`c8ec31ab-9904-4547-aa8b-5b6ff63f2ab7`
- 實例 2:`119f9daa-1354-4b8c-8e07-64da06b66b92`
- 結論:同一次 Request 內的兩個編號就**不一樣**,按 F5 後更是完全換新 → 符合定義 ✅

**[Scoped] `builder.Services.AddScoped<IShoppingCartService, ShoppingCartService>();`**
- 同 Request 實例 1:`8a7301f5-1c09-423e-bbe0-3743f2e0e153`
- 同 Request 實例 2:`8a7301f5-1c09-423e-bbe0-3743f2e0e153`
- 結論:同一次 Request 內兩編號**相同**,按 F5 後變成新的編號 → 符合定義 ✅

**[Singleton] `builder.Services.AddSingleton<IShoppingCartService, ShoppingCartService>();`**
- 同 Request 實例 1:`cde90145-a509-4548-b72a-0e346c46614f`
- 同 Request 實例 2:`cde90145-a509-4548-b72a-0e346c46614f`
- 按 F5 後:編號**完全不變**
- 結論:整個應用程式從啟動到關閉都是同一個實例 → 符合定義 ✅

---

## 五、重要觀念澄清:Scope 的範圍是什麼?

**Scope = 一次 HTTP Request**,不是「不同頁面之間」。

- 使用者打開頁面 A → 第 1 次 Request → 建立新實例
- 使用者點連結到頁面 B → 第 2 次 Request → 建立**另一個新的**實例
- 使用者再點回頁面 A → 第 3 次 Request → 再建立**一個新的**實例

### Scoped 真正的「共用」是在這裡

同一次 Request 內,**從 Controller 到 Service 層到 View**:

- Controller 拿到 → `CurrentUserService` 實例 X
- Service 拿到 → **同一個**實例 X
- View 拿到 → **同一個**實例 X
- → 只查一次資料庫 ✅

### 不同 Request 之間要怎麼共用資料?

這就是**另一個主題**了 — 例如 Session、Cookie、資料庫。DI 的生命週期**只管服務實例的建立與回收**,不管「跨 Request 的使用者狀態」。

---

## 六、實務選擇指南

### 什麼情境用 Singleton?

- 服務**無狀態**或**狀態全站共用**
- 資料量大、初始化成本高、且整個應用程式週期內資料不變
- **範例**:商品目錄快取、匯率查詢服務、Email 範本快取、全站設定

⚠️ **禁區**:絕對不要用於「**與特定使用者有關的狀態**」,例如 `CurrentUserService`

> **災難案例**:`CurrentUserService` 若註冊成 Singleton,User A 登入時記錄 `UserId = A`,同時 User B 登入會覆蓋成 `UserId = B`,這時 User A 看到的就是 **User B 的資料** → **資料洩漏(data leakage)** 😱

### 什麼情境用 Scoped?

- 服務的狀態**只在同一次 Request 內有意義**
- 同一次 Request 內的多個步驟需要**共用同一個狀態**,但不同 Request 絕不能互相影響
- **範例**:`CurrentUserService`、`ShoppingCartService`、`OrderProcessingService`、`DbContext`

### 什麼情境用 Transient?

- 每次呼叫都要是**獨立、全新**的實例
- 物件**輕量**、無需共用狀態
- **範例**:`RandomCouponGenerator`、`GuidGenerator`、無狀態的工具類別

---

## 七、建構式注入(Constructor Injection)

這是 ASP.NET Core **最常用、最推薦**的注入方式。

```csharp
public class HomeController : Controller
{
    private readonly IShoppingCartService _cartService;

    // ↓ 建構子參數,容器會自動傳入服務實例
    public HomeController(IShoppingCartService cartService)
    {
        _cartService = cartService;
    }

    public IActionResult Index()
    {
        var result = _cartService.AddItem("蘋果");
        return Content(result, "text/plain", System.Text.Encoding.UTF8);
    }
}
```

### 完整流程

1. **在 `Program.cs` 註冊** → 告訴 IoC 容器「`IShoppingCartService` 對應到 `ShoppingCartService`」
2. **有 Request 進來,ASP.NET Core 要建立 `HomeController`** → 發現建構子需要 `IShoppingCartService`
3. **ASP.NET Core 去問 IoC 容器** → 「我需要一個 `IShoppingCartService`,你有嗎?」
4. **容器按註冊規則建立實例並交出** → 依照生命週期決定是給現有的或 new 一個新的
5. **呼叫 `new HomeController(那個實例)`** → Controller 建立完成,`_cartService` 有值

---

## 八、本次學到的重點整理

- ✅ 服務註冊的語法:`builder.Services.AddXxx<介面, 實作類別>()`
- ✅ **介面對應實作**才是標準做法,才能發揮 DI 的真正價值(易測試、易替換)
- ✅ 三種生命週期的定義與親手驗證:
  - Transient → 每次 new
  - Scoped → 每個 Request 共用一個
  - Singleton → 整個應用程式共用一個
- ✅ **Scope = 一次 HTTP Request**,不是「頁面之間」
- ✅ 建構式注入是 ASP.NET Core 最常用的注入方式
- ✅ 不當選擇生命週期會造成嚴重問題(例如 Singleton + 使用者資料 = 資料洩漏)

---

## 九、中英術語對照

| 英文 | 中文 |
|------|------|
| Service Registration | 服務註冊 |
| IoC Container | IoC 容器(服務容器) |
| Service Collection | 服務集合 |
| Lifetime | 生命週期 |
| Transient | 暫時性 |
| Scoped | 範圍內(Request 範圍) |
| Singleton | 單例 |
| Constructor Injection | 建構式注入 |
| Loose Coupling | 鬆耦合 |
| Tight Coupling | 緊耦合 |
| Data Leakage | 資料洩漏 |
