# Web API 筆記：Web API 與 MVC 的差異

## MVC 與 Web API 的核心差異

| 比較項目 | MVC | Web API |
|----------|-----|---------|
| 回傳內容 | HTML 頁面（透過 Razor View） | JSON 資料 |
| 回傳方式 | `return View(data);` | `return Ok(data);` |
| 誰畫畫面 | Server 端（Razor 產生 HTML） | Client 端自己畫（App、JS 等） |
| 適合對象 | 瀏覽器 | 任何 Client（手機 App、LINE Bot、前端 JS、其他系統） |

## 為什麼需要 Web API？

MVC 的 Controller 回傳完整的 HTML 頁面，只有瀏覽器能直接使用。如果手機 App 也需要存取同一份資料，它不需要 HTML（有自己的畫面元件），硬是收到 HTML 反而要自己「挖資料」，不合理。

Web API 只回傳純粹的資料（JSON 格式），任何 Client 都能輕鬆接收和解析，各自用自己的方式呈現。

## JSON 格式

JSON（JavaScript Object Notation）是一種用文字表達資料的格式：

```json
{
  "name": "午餐",
  "amount": 150,
  "date": "2026-05-11"
}
```

- 用 `"key": value` 的結構組成
- 多筆資料用 `[ ]` 包起來
- key 通常用英文（對應 Model 的屬性名稱）
- 文字值用雙引號包住，數字不用

多筆範例：

```json
[
  { "name": "午餐", "amount": 150 },
  { "name": "計程車", "amount": 280 }
]
```

## 如何選擇？

- **只有瀏覽器使用** → MVC 比較單純（Server 直接產生完整頁面，前端不用寫程式）
- **多種 Client 需要資料**（App、Bot、前端 JS）→ Web API
- **兩者可以並存**：同一個 ASP.NET Core 專案裡可以同時有 MVC Controller 和 Web API Controller（例如後台管理用 MVC 頁面，同時開放 API 給手機 App）

## 程式碼對比

```csharp
// MVC Controller — 回傳 HTML 頁面
public IActionResult Index()
{
    var expenses = _context.Expenses.ToList();
    return View(expenses);  // 資料交給 Razor View 產生 HTML
}

// Web API Controller — 回傳 JSON 資料
public IActionResult GetAll()
{
    var expenses = _context.Expenses.ToList();
    return Ok(expenses);    // ASP.NET Core 自動將資料轉成 JSON
}
```

資料來源（`_context.Expenses.ToList()`）完全一樣，差別只在回傳方式。
