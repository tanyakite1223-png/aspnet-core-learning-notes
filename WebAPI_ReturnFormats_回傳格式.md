# Web API 筆記：回傳格式 — JSON、IActionResult

## 1. IActionResult — 統一回傳型別

`IActionResult` 是一個 Interface（介面），讓 Action 方法可以在不同情況下回傳不同的結果。

常用的回傳方法：

| 方法 | 狀態碼 | 意思 |
|------|--------|------|
| `Ok()` | 200 | 成功，有回傳資料 |
| `NoContent()` | 204 | 成功，但不回傳資料 |
| `CreatedAtAction()` | 201 | 成功建立新資源 |
| `BadRequest()` | 400 | 用戶端的請求有問題 |
| `NotFound()` | 404 | 找不到指定的資源 |

這些方法回傳的都是不同的類別（`OkObjectResult`、`NotFoundResult` 等），但都實作了 `IActionResult`，這就是 C# 多型（Polymorphism）的實際應用。

## 2. ActionResult\<T\> — 帶型別資訊的回傳

`ActionResult<T>` 與 `IActionResult` 在程式執行上結果一樣，差別在於 **API 文件**。

| | `IActionResult` | `ActionResult<T>` |
|---|---|---|
| 執行結果 | 一樣 | 一樣 |
| API 文件 | 不知道回傳什麼型別 | 自動列出欄位結構（schema） |
| 適用場景 | MVC（回傳 HTML 頁面，不需要 API 文件） | Web API（前端工程師需要看文件串接） |

範例：

```csharp
// 回傳單筆
public ActionResult<Expense> GetById(int id) { ... }

// 回傳多筆
public ActionResult<List<Expense>> GetAll() { ... }
```

注意：`ActionResult`（沒有 `<T>`）跟 `IActionResult` 效果一樣，不會產生型別資訊。一定要寫 `ActionResult<T>` 才有效果。

## 3. JSON 序列化與反序列化

- **序列化（Serialization）**：C# 物件 → JSON 字串（回傳資料給 client 時）
- **反序列化（Deserialization）**：JSON 字串 → C# 物件（接收 client 傳來的資料時）

ASP.NET Core 自動處理，不需要手動寫轉換程式碼。

### 命名慣例自動轉換

- C# 慣例：**PascalCase**（`ExpenseDate`）
- JavaScript/JSON 慣例：**camelCase**（`expenseDate`）

ASP.NET Core 預設自動將 PascalCase 轉成 camelCase，因為 JSON 主要給前端 JavaScript 使用。

### 五個 Action 的序列化方向

| Action | 序列化（物件→JSON） | 反序列化（JSON→物件） |
|--------|:---:|:---:|
| GetAll | ✔ | |
| GetById | ✔ | |
| Create | ✔（回傳新建資料） | ✔（接收 JSON body） |
| Update | | ✔（接收 JSON body） |
| Delete | | |

## 4. ProducesResponseType — 標註可能的回傳狀態碼

讓 API 文件顯示 Action 所有可能的回傳狀態碼：

```csharp
[ProducesResponseType(StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
[HttpGet("{id}")]
public ActionResult<Expense> GetById(int id)
{
    var expense = _context.Expenses.Find(id);
    if (expense == null)
    {
        return NotFound();
    }
    return Ok(expense);
}
```

沒加 `[ProducesResponseType]` 時，API 文件只會列出 200；加了之後，404 也會出現在文件中。

404 回傳的 schema 是 `ProblemDetails` — ASP.NET Core 內建的標準錯誤回傳格式。

## 5. openapi/v1.json

ASP.NET Core 自動產生的 API 規格檔，可在瀏覽器直接存取：

```
http://localhost:5242/openapi/v1.json
```

這個 JSON 檔案描述了所有 API endpoint 的路徑、參數、回傳型別等資訊，Scalar UI 就是根據這個檔案來顯示 API 文件的。
