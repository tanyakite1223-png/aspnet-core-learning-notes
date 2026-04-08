# MVC 筆記：Program.cs 啟動流程

## Program.cs 是什麼？

整個 ASP.NET Core MVC 專案的**啟動檔案**，所有東西都從這裡開始。

---

## 兩大階段

### 第一階段：builder（準備階段）

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();
var app = builder.Build();
```

- `WebApplication.CreateBuilder(args)` → 建立一個準備工具（WebApplicationBuilder）
- `builder.Services.AddControllersWithViews()` → 註冊 MVC 服務（告訴網站要使用 Controller + View）
- `builder.Build()` → 準備完成，產生 `app`（WebApplication）

> 比喻：開餐廳前的準備工作——決定菜單、請廚師、買餐具

### 第二階段：app（營業階段）

```csharp
app.UseHttpsRedirection();
app.UseRouting();
app.UseAuthorization();
app.MapStaticAssets();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

- `app.UseXxx()` → 設定 Request（請求）的處理流程（之後會詳細學 Middleware）
- `app.MapControllerRoute(...)` → 設定網址對應規則（Routing）
- `app.Run()` → 正式啟動網站

> 比喻：餐廳正式開門營業，設定客人進來後的服務流程

---

## Routing（路由）— 網址對應規則

### 預設路由 Pattern

```
{controller=Home}/{action=Index}/{id?}
```

- `controller=Home` → 如果網址沒有指定 Controller，預設用 **HomeController**
- `action=Index` → 如果網址沒有指定 Action，預設呼叫 **Index()** 方法
- `id?` → 選填參數（之後會學到）

### 網址對應範例

| 瀏覽器網址 | 對應的 Controller | 對應的 Action（方法） |
|---|---|---|
| `/` | HomeController | Index() |
| `/Home/Index` | HomeController | Index() |
| `/Home/Privacy` | HomeController | Privacy() |
| `/Home/About` | HomeController | About() |

---

## MVC 完整運作流程

1. 瀏覽器輸入網址，例如 `/Home/About`
2. Routing 解析網址 → 找到 **HomeController** 的 **About()** 方法
3. `About()` 執行 `return View()`
4. 自動去 **Views/Home/About.cshtml** 拿畫面
5. 把畫面回傳給瀏覽器顯示

---

## View 檔案位置規則

`return View()` 會自動對應到以下路徑：

```
Views / Controller名稱 / 方法名稱.cshtml
```

例如：
- HomeController 的 `Index()` → `Views/Home/Index.cshtml`
- HomeController 的 `Privacy()` → `Views/Home/Privacy.cshtml`
- HomeController 的 `About()` → `Views/Home/About.cshtml`

---

## 容易搞錯的地方

### ❌ Controller 檔案名稱少了 Controller 後綴

```
Product.cs        ← ❌ 錯誤
ProductController.cs  ← ✅ 正確（一定要加 Controller）
```

### ❌ HTML 標籤打錯

```html
<hi>標題</hi>     ← ❌ 不存在的標籤
<h1>標題</h1>     ← ✅ 是數字 1，不是字母 i
```

---

## 練習紀錄

成功在 HomeController 新增 `About()` 方法，搭配 `Views/Home/About.cshtml`，
透過瀏覽器 `/Home/About` 顯示自訂頁面。
