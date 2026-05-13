# 資料存取筆記：CRUD 實作 — 完整的新增 / 查詢 / 修改 / 刪除功能實作（整合 MVC + EF Core）

## 主題概述

CRUD 是資料操作的四個基本動作：
- **C**reate（新增）
- **R**ead（查詢 / 列表）
- **U**pdate（修改）
- **D**elete（刪除）

本主題將之前學過的 MVC（Controller、View、Model Binding、Tag Helpers）、EF Core（DbContext、Migration）、LINQ 全部整合，在 ExpenseSystem 實作完整的 CRUD 功能。

---

## Program.cs 設定：API 模式 vs MVC 模式

原本 ExpenseSystem 使用 API 模式設定，進入 CRUD 實作後需改為 MVC 模式：

| 項目 | API 模式 | MVC 模式 |
|------|----------|----------|
| 服務註冊 | `builder.Services.AddControllers()` | `builder.Services.AddControllersWithViews()` |
| 路由設定 | `app.MapControllers()` | `app.MapControllerRoute(name: "default", pattern: "{controller=Home}/{action=Index}/{id?}")` |
| View 支援 | ❌ 不支援 | ✅ 支援 Razor View |
| 適用場景 | Web API（只回傳 JSON） | MVC（回傳 HTML 頁面） |

`AddControllersWithViews()` 是全域一次性設定，所有 Controller 和 View 都會生效。

---

## Controller 基本結構：建構式注入 DbContext

```csharp
public class ExpensesController : Controller
{
    private readonly ExpenseDbContext _context;

    public ExpensesController(ExpenseDbContext context)
    {
        _context = context;
    }
}
```

重點：
- Controller 繼承 `Controller`（MVC 的類別），不是 `DbContext`（EF Core 的類別）
- DbContext 透過**建構式注入（Constructor Injection）** 取得，不要自己 `new`
- DbContext 的生命週期是 **Scoped**（每個 HTTP Request 一個實例），避免多個使用者共用同一實例導致資料互相干擾

---

## View 的對應規則

ASP.NET Core MVC 的 View 靠**資料夾結構**對應 Controller：

```
Views/
  Expenses/          ← 對應 ExpensesController
    Index.cshtml     ← 對應 Index() Action
    Create.cshtml    ← 對應 Create() Action
    Edit.cshtml      ← 對應 Edit() Action
    Delete.cshtml    ← 對應 Delete() Action
```

規則：`Views/{Controller名稱}/{Action名稱}.cshtml`

---

## `_ViewImports.cshtml` — Tag Helper 啟用設定

```cshtml
@using ExpenseSystem
@using ExpenseSystem.Models
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

- `@addTagHelper` 告訴框架啟用 Tag Helpers，沒有它 `asp-for`、`asp-action` 等都不會作用
- 放在 `Views/` 根目錄下，所有 View 都會生效
- 建一次即可

---

## R（Read）— 列表頁

### Controller Action

```csharp
public IActionResult Index()
{
    var expenses = _context.Expenses.ToList();
    return View(expenses);
}
```

- `_context.Expenses` 是 `DbSet<Expense>`，對應資料庫的 Expenses 資料表
- `.ToList()` 執行查詢，結果是 `List<Expense>`
- `return View(expenses)` 把資料傳給 View 顯示

### View（Index.cshtml）

```cshtml
@model List<Expense>

<h2>報銷列表</h2>
<a asp-action="Create">新增</a>

<table class="table">
    <thead>
        <tr>
            <th>Id</th><th>Title</th><th>金額</th><th>日期</th><th>說明</th><th>操作</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var item in Model)
        {
            <tr>
                <td>@item.ExpenseId</td>
                <td>@item.Title</td>
                <td>@item.Amount</td>
                <td>@item.ExpenseDate</td>
                <td>@item.Description</td>
                <td>
                    <a asp-action="Edit" asp-route-id="@item.ExpenseId">修改</a>
                    <a asp-action="Delete" asp-route-id="@item.ExpenseId">刪除</a>
                </td>
            </tr>
        }
    </tbody>
</table>
```

- `@model List<Expense>`：宣告 View 接收的資料型別（強型別 View）
- `asp-route-id`：Tag Helper，把值塞進 URL 路由（產生 `/Expenses/Edit/3` 之類的網址）
- 注意：`asp-route-id` 跟 HTML 的 `id` 屬性完全不同。`id` 是 HTML 原生屬性（給 CSS/JS 用），`asp-route-id` 是 Tag Helper（影響 URL 路由）

---

## C（Create）— 新增

### 兩個 Action 的分工

```csharp
[HttpGet]
public IActionResult Create()
{
    return View();  // 回傳空白表單
}

[HttpPost]
public IActionResult Create(Expense expense)
{
    if (ModelState.IsValid)
    {
        _context.Expenses.Add(expense);
        _context.SaveChanges();
        return RedirectToAction("Index");  // PRG 模式
    }
    return View(expense);  // 驗證失敗，帶資料回表單讓使用者修正
}
```

- **GET**：顯示空白表單，`return View()` 不帶參數
- **POST**：接收表單資料（Model Binding 自動把表單欄位對應到 `Expense` 物件）
- `ModelState.IsValid`：檢查 Data Annotation 驗證是否通過
- `_context.Expenses.Add(expense)`：標記為新增
- `_context.SaveChanges()`：實際寫入資料庫
- POST 參數是 `Expense expense`（單一物件），不是 `List<Expense>`，因為表單一次只送一筆

### View（Create.cshtml）

```cshtml
@model Expense

<h2>新增單據</h2>
<form asp-action="Create" method="post">
    <div>
        <label asp-for="Title"></label>
        <input asp-for="Title" />
        <span asp-validation-for="Title"></span>
    </div>
    <!-- Amount、ExpenseDate、Description 同樣結構 -->
    <button type="submit">新增</button>
</form>
```

- `@model Expense`：單一物件（對比列表頁的 `List<Expense>`）
- `asp-for`：Tag Helper，自動綁定 Model 屬性，如果 Model 有值會自動填入
- `asp-validation-for`：顯示對應欄位的驗證錯誤訊息
- `<input>` 是 void element（自閉合標籤），不能在標籤之間放內容

---

## U（Update）— 修改

### 與 Create 的關鍵差異

| | Create | Edit |
|---|--------|------|
| GET Action | `return View()`（空白表單） | `return View(expense)`（帶現有資料） |
| 靠什麼找資料 | 不需要 | 靠 URL 帶的 `id` 參數 |
| 表單是否需要 hidden field | 不需要 | 需要，放 ExpenseId |
| EF Core 操作 | `_context.Add()` | `_context.Update()` |

### Controller Actions

```csharp
[HttpGet]
public IActionResult Edit(int id)
{
    var expense = _context.Expenses.Find(id);
    if (expense == null) return NotFound();
    return View(expense);
}

[HttpPost]
public IActionResult Edit(Expense expense)
{
    if (ModelState.IsValid)
    {
        _context.Expenses.Update(expense);
        _context.SaveChanges();
        return RedirectToAction("Index");
    }
    return View(expense);
}
```

- `Find(id)`：用主鍵查詢單筆資料
- `null` 檢查：如果找不到回傳 `NotFound()`（HTTP 404）
- `_context.Update()`：標記為修改（對比新增用 `_context.Add()`）

### View（Edit.cshtml）— hidden field

```cshtml
@model Expense

<form asp-action="Edit" method="post">
    <input type="hidden" asp-for="ExpenseId" />
    <!-- Title、Amount、ExpenseDate、Description 表單欄位 -->
    <button type="submit">儲存</button>
</form>
```

- `<input type="hidden" asp-for="ExpenseId" />`：使用者看不到，但表單送出時會把 ExpenseId 一起帶回 Controller
- 如果少了這個，POST 回去的 `expense.ExpenseId` 會是 0，EF Core 不知道要更新哪一筆

---

## D（Delete）— 刪除

### 為什麼需要確認頁

1. **防止誤刪**：使用者看到確認頁可以反悔
2. **GET 不應做寫入操作**：瀏覽器預載、搜尋引擎爬蟲可能自動發 GET 請求，如果 GET 就能刪除資料會被意外觸發

### Controller Actions

```csharp
[HttpGet]
public IActionResult Delete(int id)
{
    var expense = _context.Expenses.Find(id);
    if (expense == null) return NotFound();
    return View(expense);  // 顯示確認頁
}

[HttpPost]
[ActionName("Delete")]
public IActionResult DeleteConfirmed(int id)
{
    var expense = _context.Expenses.Find(id);
    _context.Expenses.Remove(expense);
    _context.SaveChanges();
    return RedirectToAction("Index");
}
```

- `[ActionName("Delete")]`：因為 C# 不允許同一個 class 有兩個同名同參數的方法，所以 POST 方法取名 `DeleteConfirmed`，但用 `[ActionName]` 告訴路由系統「這個方法對應的 Action 名稱還是 Delete」
- `_context.Remove()`：標記為刪除

---

## 核心觀念整理

### GET vs POST 的分工

- **GET 的 Action**：負責顯示畫面（列表、空白表單、帶資料的表單、刪除確認頁）
- **POST 的 Action**：負責處理資料（新增、修改、刪除）

GET 是「要東西看」，POST 是「把資料送過去處理」。

### PRG 模式（Post-Redirect-Get）

POST 的 Action 處理完資料後，使用 `RedirectToAction()` 而非 `return View()`：

- `return View()`：瀏覽器停留在 POST 結果頁，按重新整理會重複送出 POST → 資料重複操作
- `RedirectToAction()`：伺服器回應 302，瀏覽器發新的 GET 請求 → 按重新整理只是重新 GET，不會重複操作

簡單記：
- **GET 的 Action** → `return View()`
- **POST 的 Action**（成功時）→ `RedirectToAction()`

### EF Core 操作對照

| CRUD | EF Core 方法 | 說明 |
|------|-------------|------|
| Create | `_context.Add(entity)` | 標記為新增 |
| Read | `_context.Expenses.ToList()` / `.Find(id)` | 查詢全部 / 單筆 |
| Update | `_context.Update(entity)` | 標記為修改 |
| Delete | `_context.Remove(entity)` | 標記為刪除 |

以上都需要呼叫 `_context.SaveChanges()` 才會實際寫入資料庫。

### asp-route-{參數名} vs HTML 原生屬性

- `asp-route-id="@item.ExpenseId"` → Tag Helper，影響 URL 路由（產生 `/Expenses/Edit/3`）
- `id="something"` → HTML 原生屬性，給 CSS/JavaScript 識別元素用

名字相似但功能完全不同。

---

## 完成的檔案結構

```
ExpenseSystem/
├── Controllers/
│   └── ExpensesController.cs      ← 完整 CRUD Actions
├── Models/
│   └── Expense.cs                 ← 資料模型
├── Data/
│   └── ExpenseDbContext.cs        ← EF Core DbContext
├── Views/
│   ├── _ViewImports.cshtml        ← Tag Helper 全域啟用
│   └── Expenses/
│       ├── Index.cshtml           ← 列表頁（R）
│       ├── Create.cshtml          ← 新增表單（C）
│       ├── Edit.cshtml            ← 修改表單（U）
│       └── Delete.cshtml          ← 刪除確認頁（D）
└── Program.cs                     ← MVC 模式設定
```

---

## 學習過程常見錯誤

1. **Controller 繼承錯對象**：應繼承 `Controller`（MVC），不是 `DbContext`（EF Core）
2. **`@model` 的型別**：列表頁用 `List<Expense>`，單筆頁用 `Expense`。注意是 class 名稱 `Expense`，不是 DbSet 名稱 `Expenses`
3. **Razor 語法**：`@using` 不加分號（不同於 C#）；class 結尾不加分號
4. **Edit 忘記 hidden field**：漏了 `<input type="hidden" asp-for="ExpenseId" />`，導致更新時 id 為 0
5. **`<input>` 是 void element**：不能寫 `<input>內容</input>`，值透過 `asp-for` 綁定
6. **Tag Helper 不生效**：檢查是否有 `_ViewImports.cshtml` 且包含 `@addTagHelper`
