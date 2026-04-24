# ASP.NET Core MVC 筆記:DI(依賴注入)

---

## 一、為什麼需要 DI?

### ❌ 不用 DI 的寫法(反例)

```csharp
public class ProductController : Controller
{
    public IActionResult Index()
    {
        IProductService service = new ProductService();  // 直接 new
        var products = service.GetAll();
        return View(products);
    }
}
```

### 三個問題

#### 問題 1:緊耦合(Tight Coupling)
Controller 寫死了要用 `ProductService`。將來若要換成 `ProductServiceV2`,或測試時要換成 `FakeProductService`,都得改 Controller 程式碼。

#### 問題 2:無法單元測試
想讓 `GetAll()` 回傳假資料,但 Controller 自己 new 出真實的 `ProductService`,會真的去連資料庫。測試變慢、不穩定、還要準備測試資料庫。

#### 問題 3:依賴鏈爆炸
若 `ProductService` 需要 `ILogger`、`DbContext`、`IEmailSender`:

```csharp
var logger = new Logger(...);
var db = new AppDbContext(...);
var email = new EmailSender(...);
var service = new ProductService(logger, db, email);
```

Controller 得知道所有下游服務怎麼 new。DI 容器能幫妳處理整條依賴鏈。

---

## 二、建構式注入(Constructor Injection)完整寫法

### 固定公式(背下來!)

```csharp
public class [我的類別名] : [我實作的介面]
{
    private readonly [要注入的介面] [欄位名];       // 第 1 步:宣告欄位

    public [我的類別名]([要注入的介面] [參數名])    // 第 2 步:建構式接收
    {
        [欄位名] = [參數名];                        // 第 3 步:存起來
    }
}
```

### 完整範例

```csharp
public class OrderService : IOrderService
{
    private readonly IProductService _productService;

    public OrderService(IProductService productService)
    {
        _productService = productService;
    }

    public void CreateOrder(int productId)
    {
        var product = _productService.GetById(productId);
        // ... 建立訂單的邏輯
    }
}
```

### 五個關鍵規則

1. `public class XxxService : IXxxService` — **冒號只接介面**,不接 `Service` 或其他
2. `private readonly` — 欄位修飾詞,小寫
3. **欄位名前面加底線 `_`,參數名不加**(慣例)
4. **建構式名稱 = 類別名稱**,沒有 `class` 關鍵字
5. 建構式裡 `_欄位 = 參數`

### 注入多個依賴

```csharp
public class PaymentService : IPaymentService
{
    private readonly IEmailSender _emailSender;
    private readonly ILogger _logger;

    public PaymentService(IEmailSender emailSender, ILogger logger)
    {
        _emailSender = emailSender;
        _logger = logger;
    }

    public void ProcessPayment(decimal amount, string userEmail)
    {
        _logger.Log($"處理付款 {amount}");
        _emailSender.Send(userEmail, "付款完成", "已成功扣款");
    }
}
```

---

## 三、在 Program.cs 註冊服務

```csharp
builder.Services.AddScoped<IProductService, ProductService>();
//                          ↑               ↑
//                       服務型別(介面)   實作型別(類別)
//                       別人要求什麼      我實際給他什麼
```

### 兩個泛型參數的意義

| 位置 | 角色 | 意義 |
|------|------|------|
| 第一個 `IProductService` | **服務型別**(Service Type) | Controller 建構子裡寫的型別 |
| 第二個 `ProductService` | **實作型別**(Implementation Type) | DI 容器實際 new 出來的類別 |

### 經典錯誤:兩個都寫介面

```csharp
// ❌ 錯誤
builder.Services.AddScoped<IProductService, IProductService>();
```

**錯誤訊息:**
```
System.ArgumentException: Cannot instantiate implementation type 
'IProductService' for service type 'IProductService'.
```

**原因**:介面沒有實作,**不能被 new**。DI 容器需要一個可以實際建立的類別,不能是介面。

### `AddScoped` 這行做了什麼?

**重要觀念**:這行**不會**真的 new 一個 `ProductService` 物件。

它只是向 IoC 容器**登記合約**:
> 「將來如果有人要 `IProductService`,請 new 一個 `ProductService` 給他。」

真正 new 物件的時機是**收到 request 時**,DI 容器才會按照合約 new 出實體注入進去。

---

## 四、三種生命週期(統一完整版)

### 定義

| 生命週期 | 多久 new 一次 |
|---------|--------------|
| **Transient** | 每次被注入都 new 一個新的 |
| **Scoped** | 每個 HTTP request new 一個,request 結束就丟 |
| **Singleton** | 整個網站啟動後只 new 一次,直到網站關掉 |

### 行為對照表

| 情境 | Transient | Scoped ⭐ | Singleton |
|------|-----------|----------|-----------|
| 同一 request 內多處注入 | 每處都不同 | **同一個** | **同一個** |
| 不同 request 之間 | 不同 | 不同 | **同一個** |
| 網站重啟後 | 不同 | 不同 | **不同** |
| 不同使用者之間 | 不同 | 不同 | **全部共用** ⚠️ |

### 記憶比喻

- **Transient**:衛生紙一次性 — 用完就丟
- **Scoped** ⭐:專員陪辦事 — 妳這次辦事專員全程陪,辦完下班
- **Singleton**:7-11 的櫃檯 — 開店到打烊都是同一個

### Singleton 的生命期範圍

> **網站程序的整個生命週期**,不是「使用者的操作期間」

- 開瀏覽器分頁、關瀏覽器再開 → Singleton **不變**
- VS 按 ⏹ 停掉偵錯 / Console 視窗關閉 → 網站程序結束 → Singleton 消失
- F5 重啟 → 網站程序重生 → Singleton 重新 new

---

## 五、生命週期判斷流程(統一版)

### 🔍 步驟 1:有沒有注入依賴?

**注入依賴 = 建構式有參數,且參數是另一個服務**(通常是 I 開頭的介面)

- **有注入依賴** → 跳到步驟 2
- **沒有注入依賴** → 跳到步驟 3

### 🔍 步驟 2:有注入依賴時,看注入的是什麼生命週期

- 注入了 **Scoped** 依賴(例如 `IDbContext`、`IEmailSender`、大部分業務服務)→ 自己必須是 **Scoped**
- 注入了 **Singleton** 依賴 → 可繼續看步驟 3
- 注入了 **Transient** 依賴 → 可以 Scoped 或 Transient

**實務 90% 的情況**:注入的是 Scoped → 直接選 **Scoped**。

### 🔍 步驟 3:看欄位會不會被寫入

「**會被寫入的欄位**」= class 裡有宣告欄位,且**至少有一個方法會改動它的值**。

- **有任何方法會寫入欄位** → 選 **Scoped**
- **所有欄位都不會被寫入**(或沒欄位) → **Singleton** ✅

### 簡化口訣

**兩個問題就決定:**

1. **建構式有沒有注入 Scoped 依賴?** 有 → **Scoped**
2. 沒有的話,**有沒有方法會改欄位?** 有 → **Scoped**;沒有 → **Singleton**

Transient 很少用,只有特殊需求(每次都要全新物件、不能共用)才選它。

---

## 六、「會被寫入的欄位」詳解

### 🟢 不會被寫入的欄位(安全)

```csharp
private const decimal TaxRate = 0.05m;        // const:編譯時固定,永遠不能改
private readonly decimal _rate = 31.5m;       // readonly + 直接賦值:之後不能改
private readonly IDbContext _db;              // readonly + 建構式賦值:之後不能改
public string SiteName => "Amber 購物網";      // 運算式屬性,每次都是同樣的值
```

### 🔴 會被寫入的欄位(危險)

```csharp
// 情況 1:一般欄位被方法賦值
private int _count = 0;
public void Add() => _count++;                  // 寫入

// 情況 2:一般欄位被方法賦值
private string _theme = "light";
public void SetTheme(string theme) => _theme = theme;  // 寫入

// 情況 3:即使 readonly,集合內容還是可以改!
private readonly List<string> _items = new();
public void AddItem(string x) => _items.Add(x);  // 寫入(改了內容)
```

### ⚠️ readonly 的陷阱

`readonly` 只擋「**整個欄位被換掉**」,**不擋「集合/Dictionary 的內容被改動**」。

```csharp
private readonly List<string> _items = new();

_items = new List<string>();    // ❌ readonly 擋住這個
_items.Add("iPhone");           // ✅ readonly 擋不住!這算寫入
_items.Clear();                 // ✅ readonly 擋不住!這算寫入
```

判斷 Singleton 時,**要看所有方法有沒有改到集合內容**,不是只看 readonly。

### 什麼動作算「寫入」?

| 動作 | 範例 | 算寫入? |
|------|------|--------|
| 賦值 | `_count = 5;` | ✅ |
| 遞增遞減 | `_count++;` / `_count--;` | ✅ |
| 複合賦值 | `_total += 100;` | ✅ |
| List 操作 | `_items.Add(x);` / `_items.Remove(x);` | ✅ |
| Dictionary 賦值 | `_cache["k"] = v;` | ✅ |
| 純讀取 | `return _items;` | ❌ |
| 呼叫算東西 | `return _items.Count;` | ❌ |

---

## 七、實務生命週期對照表

| 服務類型 | 範例 | 生命週期 | 理由 |
|---------|------|---------|------|
| Repository / 資料存取 | `IOrderRepository` | **Scoped** | 注入了 `IDbContext`(Scoped) |
| 業務邏輯服務 | `IProductService`、`IOrderService` | **Scoped** | 通常注入 Repository |
| 使用者相關服務 | `ICartService`、`ICurrentUserService` | **Scoped** | 每個使用者的 request 要獨立 |
| 純計算/格式化工具 | `IPriceFormatter`、`ICurrencyConverter` | **Singleton** | 沒欄位或欄位全 readonly |
| 網站設定(常數) | `ISiteSettingService` | **Singleton** | 全站共用、不變 |
| 純編碼解碼 | `IStringEncoder` | **Singleton** | 無狀態 |
| 訂單編號產生器 | `IOrderNumberGenerator` | Transient | 每次要全新的 |
| 不確定時 | - | **Scoped** | 最安全的預設選擇 |

### Singleton 的鐵律 ⚠️

> **有欄位會記錄「特定使用者的資料」,絕對不能 Singleton。**

例如購物車服務 `ICartService`:
- 使用者 A `AddItem("iPhone")` → `_items = ["iPhone"]`
- 使用者 B `AddItem("MacBook")` → `_items = ["iPhone", "MacBook"]`(加進同一份!)
- 使用者 A 看購物車 → **看到自己沒買的 MacBook** 😱

這叫「**跨使用者資料污染**」,嚴重可能變成資安漏洞。

---

## 八、四大「常見誤解」糾正

### ❌ 誤解 1:「有欄位 = 不能 Singleton」

**錯**。關鍵是「欄位會不會被寫入」,不是「有沒有欄位」。

`readonly` 或 `const` 且無方法修改的欄位,是安全的,可以 Singleton。

### ❌ 誤解 2:「有方法 = 不能 Singleton」

**錯**。每個 class 都有方法,這不是判斷標準。

### ❌ 誤解 3:「有建構式 = 不能 Singleton」

**錯**。建構式本身不影響,重點是建構式**有沒有注入 Scoped 依賴**。

### ❌ 誤解 4:「每次參數不同 = 要 Transient」

**錯**。參數是方法呼叫時傳進來的,跟物件生命週期無關。

比喻:一台計算機可以算 1+1、2+2、3+3,不用因為數字不同就換計算機。

---

## 九、在 View 中使用 DI

### 為什麼 View 也需要 DI?

90% 情況 View 只該顯示 Controller 傳來的 Model。少數例外:

- 每個頁面側邊欄都要顯示「商品分類清單」
- Layout 想顯示「目前線上人數」、「網站公告」
- 顯示時間格式化、貨幣轉換等純顯示工具

### 語法:`@inject`

```cshtml
@using MyMvcApp.Services
@inject IProductService ProductService
```

**格式**:`@inject 型別 名字`

**必須給它一個名字**,不然 View 裡沒辦法叫它。

### 使用範例

```cshtml
@using MyMvcApp.Services
@inject ICategoryService CategoryService

<aside>
    <h4>商品分類</h4>
    <ul>
        @foreach (var category in CategoryService.GetCategories())
        {
            <li>@category</li>
        }
    </ul>
</aside>
```

### 為什麼 View 沒建構式也能注入?

Razor View 編譯成 C# 類別時,`@inject` 會自動變成**屬性注入**(Property Injection)。Razor 引擎在幕後幫妳處理建構式的細節,妳只要寫 `@inject` 就好。

---

## 十、實測觀察:Singleton 生命週期

### 測試服務

```csharp
public class SiteSettingService : ISiteSettingService
{
    public string SiteName => "Amber 購物網";
    public DateTime ServiceStartedAt { get; } = DateTime.Now;
}
```

註冊:`builder.Services.AddSingleton<ISiteSettingService, SiteSettingService>();`

### 觀察結果

在 Layout 顯示 `ServiceStartedAt`,測試三個動作:

| 動作 | 時間 | 原因 |
|------|------|------|
| 重新整理 10 次 | **沒變** | 同網站程序內只有一個實體 |
| 切到 `/Product` 再回首頁 | **沒變** | 跨 request、跨頁面都共用 |
| 停掉網站重啟 | **有變** | 網站程序重來,Singleton 消失後重 new |

---

## 十一、追問補充:Cookie 與 AuthToken 運作原理

### 問題背景

> 「Scoped 每個 request 重生,網站怎麼知道我是誰?」

### HTTP 是無狀態的

每個 request 獨立:點「會員中心」、點「我的訂單」、點「修改密碼」都是**不同的 request**,網站每次都要重新確認身份。

### AuthToken 是**網站伺服器**產生的

不是瀏覽器、不是使用者,是網站自己在**使用者登入成功那一刻**產生。

### 完整流程

```
Step 1: 使用者送出登入(帳密)
Step 2: 網站驗證帳密 → 正確
Step 3: 網站產生 AuthToken(例如 "abc123xyz789")
Step 4: 網站把 Token 記在伺服器(資料庫/記憶體)
        ┌─────────────┬──────────┐
        │ Token       │ 是誰      │
        ├─────────────┼──────────┤
        │ abc123xyz789│ Amber    │
        └─────────────┴──────────┘
Step 5: Response 回傳 Set-Cookie: AuthToken=abc123xyz789
Step 6: 瀏覽器自動保存 Cookie
```

### 之後每次 request

```
[Request] GET /Member/Profile
          Cookie: AuthToken=abc123xyz789   ← 瀏覽器自動帶!
          ↓
[網站]    查 abc123xyz789 是誰 → Amber ✓
          回傳頁面
```

### 核心觀念

> **使用者「登入狀態」 ≠ 服務「物件實體」**
>
> - 登入狀態存在 Cookie 裡,跨 request 持續存在
> - Scoped 服務物件每個 request 都重生一次
> - 每個 request 進來,服務物件從 Cookie 重新讀取身份

### 兩種 Token 風格(未來會詳學)

- **Session-based**:Token 存伺服器、Cookie 只放識別碼,每次查資料庫驗證
- **JWT Token**:Token 自身打包+簽章,不用查資料庫,驗證簽章即可

---

## 十二、附加釐清:C# 基礎觀念

### 1️⃣ 冒號 `:` 的兩種用途

```csharp
// 用途 1:繼承類別(後面接 class)
public class ProductController : Controller      // Controller 是類別

// 用途 2:實作介面(後面接 interface)
public class ProductService : IProductService    // IProductService 是介面

// 也可以同時:先繼承類別,再接介面
public class MyController : Controller, IMyInterface
```

**MVC Controller 必須繼承 `Controller` 基底類別**(不是實作介面),這樣才能用 `View()`、`RedirectToAction()` 等功能。

### 2️⃣ 欄位宣告格式

```
修飾詞    型別    名稱;
  ↓        ↓       ↓
private readonly  IProductService _productService;
```

**只能有一個型別!** 介面本身就是完整型別,不需要再加 `string`、`decimal` 等:

```csharp
// ❌ 錯誤(兩個型別)
private readonly string IProductService _productService;

// ✅ 正確(一個型別)
private readonly IProductService _productService;
```

### 3️⃣ 方法名 vs 回傳型別

```csharp
public List<Book> GetAll()
//      ↑         ↑
//   回傳型別   方法名稱
{
    return _db.Books.ToList();
}
```

**方法名 = 緊貼著 `(` 前面的那個字**,不是前面的型別。

### 4️⃣ `return` 的兩種用法

| 方法宣告 | `return;` | `return 值;` |
|---------|-----------|-------------|
| `void` | ✅ 可以(提早結束) | ❌ 不行 |
| 有回傳型別(`int`/`string`/...) | ❌ 不行 | ✅ **必須** |

```csharp
// void 方法:可以空手 return,用來提早結束
public void SendSms(string phone)
{
    if (string.IsNullOrEmpty(phone))
    {
        return;      // ✅ 空手離開
    }
    _smsApi.Send(phone);
}

// 有回傳型別:必須帶值 return
public int GetAge() 
{ 
    return 25;     // ✅ 帶 int 回來
}
```

### 5️⃣ 運算式主體成員 `=>`

「只有一行程式碼的方法或屬性」的簡寫:

```csharp
// 傳統寫法
public int Add(int a, int b)
{
    return a + b;
}

// 簡寫(完全等價)
public int Add(int a, int b) => a + b;
```

void 方法也可以:

```csharp
public void AddItem(string item) => _items.Add(item);
// 等同於
public void AddItem(string item) { _items.Add(item); }
```

⚠️ **`=>` 在方法/屬性上的意思**:「就是這個,沒別的。」後面放要回傳的值或要執行的動作,編譯器自動補 `return` 或 `{ }`。

### 6️⃣ 欄位 vs 區域變數

| 名稱 | 位置 | 生命期 | 誰看得到 |
|------|------|-------|---------|
| **欄位** | class 裡、方法外 | 跟物件一樣長 | class 所有方法都看得到 |
| **區域變數** | 方法裡面 | 方法結束就消失 | 只有那個方法看得到 |

```csharp
public class Example
{
    private int _count = 0;          // 欄位

    public void MethodA()
    {
        int temp = 10;               // 區域變數,只活在 MethodA
        _count++;                    // 可以用欄位
    }

    public void MethodB()
    {
        // 這裡看不到 temp!
        _count++;                    // 但看得到欄位
    }
}
```

判斷 Singleton 時,**區域變數不算「寫入」**,因為它不是欄位。

### 7️⃣ 類別角色的命名慣例

| 結尾 | 角色 | 範例 | 資料夾 |
|------|------|------|-------|
| `Controller` | 控制器(處理 HTTP) | `ProductController` | `/Controllers` |
| `Service` | 業務服務 | `OrderService` | `/Services` |
| `Repository` | 資料存取 | `ProductRepository` | `/Repositories` |
| **沒特殊字尾** | **資料模型** | `Product`、`Order`、`Cart` | `/Models` |

### 8️⃣ 自訂型別

`Product`、`Order`、`User` 等是自己(或團隊)建立的類別,不是 C# 內建的。

```csharp
// Models/Product.cs 定義型別
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

// 任何地方都可以用 Product 當型別
Product p = new Product();                     // 當變數
public void AddProduct(Product p) { }          // 當參數
public Product GetById(int id) { }             // 當回傳型別
List<Product> products = new();                // 當泛型內容
```

### 9️⃣ 服務的三種分類

#### 業務服務(Business Service)— 處理「公司的生意」
- 例如:`IProductService`、`IOrderService`、`ICartService`、`IPromotionService`
- 跟特定業務場景綁定,換行業就完全不同

#### 基礎設施服務(Infrastructure Service)— 處理「技術通用功能」
- 例如:`IDbContext`、`ILogger`、`IEmailSender`、`ISmsService`
- 跟具體業務無關,任何專案都可能用

#### 工具服務(Utility Service)— 處理「小幫手」
- 例如:`IPriceFormatter`、`ICurrencyConverter`、`IStringEncoder`
- 無狀態的純工具類

---

## 十三、常見錯誤與陷阱

### 錯誤 1:註冊時第二個泛型寫成介面

```csharp
// ❌ 錯誤
builder.Services.AddScoped<IProductService, IProductService>();
```

錯誤訊息:`Cannot instantiate implementation type...`

### 錯誤 2:Visual Studio 自動完成選錯型別

輸入 `Date` 時,VS 可能補成 `DataSetDateTime`(屬於 `System.Data`),不是 `DateTime`。

**辨識方法**:看自動完成選項上的小型說明浮窗,確認命名空間。

**我們要的是**:`DateTime`(屬於 `System`,.NET 6+ 隱式引入,不用 using)

### 錯誤 3:Singleton 存使用者資料

絕對不要把「與特定使用者相關」的資料存在 Singleton,會被所有人共用。

### 錯誤 4:void 方法裡寫 `return 值;`

```csharp
// ❌ 錯誤
public void SendSms(string phone)
{
    return _logger.Log(...);   // void 不能 return 值
}
```

### 錯誤 5:建構式寫成 class

```csharp
// ❌ 錯誤
public class OrderService
{
    public class OrderService(IProductService p) { }   // 多了 class 關鍵字
}

// ✅ 正確
public class OrderService
{
    public OrderService(IProductService p) { }         // 沒有 class,名稱=類別名
}
```

### 錯誤 6:void 方法回傳值用 `var` 接

```csharp
// ❌ 錯誤
var x = _logger.Log("...");   // Log 是 void,沒東西可接
```

動作型方法(void)直接呼叫就好,不要接收。

---

## 十四、重點 API 總整理

### 註冊方式

```csharp
builder.Services.AddTransient<IService, Service>();  // 每次都新
builder.Services.AddScoped<IService, Service>();     // 每 request 一個 ⭐
builder.Services.AddSingleton<IService, Service>();  // 整站共用
```

### Controller 注入語法

```csharp
public class MyController : Controller
{
    private readonly IMyService _service;
    
    public MyController(IMyService service)   // 建構式注入
    {
        _service = service;
    }
}
```

### View 注入語法

```cshtml
@using MyApp.Services
@inject IMyService MyService

@MyService.DoSomething()
```

---

## 十五、最終判斷流程總整理

```
拿到一個 Service,要選生命週期:

步驟 1:建構式有沒有注入 Scoped 依賴?
   └─ 有  → Scoped ✅(結束)
   └─ 沒  → 步驟 2

步驟 2:有沒有方法會寫入欄位?
   └─ 有  → Scoped ✅
   └─ 沒  → Singleton ✅

(Transient 只在特殊情況用,例如要求每次都產生不同物件)
```

**不確定時預設 Scoped**,90% 是對的。
