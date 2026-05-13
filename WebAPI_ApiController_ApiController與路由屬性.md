# Web API 筆記：ApiController 與路由屬性

## ControllerBase vs Controller

| 項目 | `ControllerBase` | `Controller` |
|------|-----------------|--------------|
| 用途 | Web API | MVC |
| 提供的方法 | `Ok()`、`NotFound()`、`BadRequest()` 等 | 繼承 ControllerBase 的所有方法，再加上 `View()`、`ViewBag` 等 |
| 處理 View | ❌ 不處理 | ✅ 可以回傳 View |

`Controller` 繼承 `ControllerBase`，所以 MVC Controller 也能用 `Ok()`、`NotFound()` 這些方法，但 Web API 不需要 View 功能，用 `ControllerBase` 就夠了，比較輕量。

---

## `[ApiController]` 屬性

加在 class 上面，開啟 Web API 專用的自動化行為：

- **自動驗證 Model**：前端送來的資料不合格時，自動回傳 400 Bad Request，不用自己寫 `if (!ModelState.IsValid)` 判斷
- **自動推斷參數來源**：系統自動判斷參數該從 URL、Query String 還是 Request Body 取得

這些在 MVC Controller 裡要手動處理，加了 `[ApiController]` 就自動做了。

---

## 屬性路由（Attribute Routing）

### Class 層級：`[Route]`

```csharp
[Route("api/[controller]")]
public class ApiExpensesController : ControllerBase
```

- `[controller]` 是佔位符（placeholder），取 class 名稱去掉 `Controller` 後綴
- `ApiExpensesController` → 去掉 `Controller` → 基底路徑為 `/api/ApiExpenses`
- 系統看的是 **class 名稱**，不是檔案名稱

### Action 層級：`[HttpGet]`、`[HttpGet("{id}")]`

```csharp
[HttpGet]                    // GET /api/ApiExpenses
public IActionResult GetAll() { ... }

[HttpGet("{id}")]            // GET /api/ApiExpenses/3
public IActionResult GetById(int id) { ... }
```

- Action 上的路由片段會跟 class 的 `[Route]` **自動組合**
- `[HttpGet("{id}")]` 中的 `{id}` 會對應到方法參數 `int id`

### 屬性路由 vs 慣例路由

| 項目 | 屬性路由（Attribute Routing） | 慣例路由（Conventional Routing） |
|------|-------------------------------|--------------------------------|
| 定義位置 | Controller / Action 上方的 Attribute | Program.cs 的 `MapControllerRoute` |
| 常用於 | Web API | MVC |
| 範例 | `[Route("api/[controller]")]` | `{controller=Home}/{action=Index}/{id?}` |

兩種路由可以**同時並存**。ExpenseSystem 目前的狀態：MVC 的 `ExpensesController` 走慣例路由，API 的 `ApiExpensesController` 走屬性路由。

---

## HTTP 狀態碼（Status Code）

| 狀態碼 | 意義 | 對應方法 | 使用情境 |
|--------|------|----------|----------|
| 200 OK | 成功 | `Ok()` | 正常回傳資料 |
| 400 Bad Request | 請求有問題 | `BadRequest()` | 格式不對、驗證失敗 |
| 404 Not Found | 資源不存在 | `NotFound()` | 請求格式正確，但找不到資料 |

**400 vs 404 的區分重點**：
- 400 = **你的請求有問題**（例如 `/api/ApiExpenses/abc`，id 應該是 int 卻送了字串）
- 404 = **你的請求沒問題，但東西不存在**（例如 `/api/ApiExpenses/999`，格式正確但沒有這筆資料）

---

## 完整程式碼

```csharp
[ApiController]
[Route("api/[controller]")]
public class ApiExpensesController : ControllerBase
{
    private readonly ExpenseDbContext _context;

    public ApiExpensesController(ExpenseDbContext context)
    {
        _context = context;
    }

    [HttpGet]
    public IActionResult GetAll()
    {
        var expenses = _context.Expenses.ToList();
        return Ok(expenses);
    }

    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var expense = _context.Expenses.Find(id);
        if (expense == null)
        {
            return NotFound();
        }
        return Ok(expense);
    }
}
```

---

## 本次實作結果

- `GET /api/ApiExpenses` → 回傳所有 Expenses（JSON 陣列）
- `GET /api/ApiExpenses/1` → 回傳單筆 Expense（JSON 物件）
- `GET /api/ApiExpenses/999` → 回傳 404 Not Found
