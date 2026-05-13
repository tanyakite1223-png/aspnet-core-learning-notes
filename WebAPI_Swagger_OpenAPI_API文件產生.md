# Web API 筆記:Swagger / OpenAPI（API 文件產生）

## OpenAPI 是什麼？

OpenAPI 是一種**描述 API 的標準格式**，用 JSON 記錄：
- 有哪些路由（endpoint）
- 每個路由接受什麼參數、request body
- 每個路由可能回傳哪些狀態碼與格式

ASP.NET Core 內建支援 OpenAPI，專案跑起來後可直接瀏覽：
- **JSON 文件**：`/openapi/v1.json`
- **Scalar UI（視覺化介面）**：`/scalar/v1`

---

## OpenAPI 文件給誰看？

| 對象 | 用途 |
|------|------|
| 前端開發者 | 了解有哪些 API、怎麼呼叫、回傳什麼 |
| 後端開發者自己 | 直接在瀏覽器測試 API，不需另裝 Postman |
| 第三方串接方 | 串接系統時參考 API 規格 |

---

## 文件資訊的兩種來源

### 1. 自動推斷
若 Action Method 回傳型別是 `ActionResult<T>`，系統會自動推斷出 **200 OK**，不需要手動標註。

```csharp
public ActionResult<List<Expense>> GetAll()  // 自動產生 200
```

### 2. 手動標註 — `[ProducesResponseType]`
200 以外的狀態碼（404、400、204、201 等）需要手動標註，否則不會出現在文件裡。

```csharp
[ProducesResponseType(StatusCodes.Status200OK)]      // 可省略（自動推斷）
[ProducesResponseType(StatusCodes.Status404NotFound)]
[HttpGet("{id}")]
public ActionResult<Expense> GetById(int id)
```

`[ProducesResponseType]` 寫在 **Action Method 的上面**（與 `[HttpGet]`、`[HttpPost]` 等並列）。

---

## 各 Action 的狀態碼對應

| Action | 狀態碼 | 說明 |
|--------|--------|------|
| GetAll | 200 | 自動推斷 |
| GetById | 200、404 | 找不到回 404 |
| Create | 201、400 | 建立成功回 201；格式錯誤回 400 |
| Update | 204、400、404 | 成功回 204；id 不符回 400；找不到回 404 |
| Delete | 204、404 | 成功回 204；找不到回 404 |

> ⚠️ `NoContent()` 對應的是 **204**，不是 200。

---

## 常用狀態碼速查

| 狀態碼 | 名稱 | 對應方法 | 情境 |
|--------|------|----------|------|
| 200 | OK | `Ok()` | 查詢成功 |
| 201 | Created | `CreatedAtAction()` | 新增成功 |
| 204 | No Content | `NoContent()` | 修改／刪除成功（無回傳內容） |
| 400 | Bad Request | `BadRequest()` | 格式錯誤、條件不符 |
| 404 | Not Found | `NotFound()` | 找不到資料 |

---

## 實作範例（ApiExpensesController.cs）

```csharp
[ProducesResponseType(StatusCodes.Status201Created)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
[HttpPost]
public async Task<IActionResult> Create(Expense expense) { ... }

[ProducesResponseType(StatusCodes.Status204NoContent)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
[HttpPut("{id}")]
public async Task<IActionResult> Update(int id, Expense expense) { ... }

[ProducesResponseType(StatusCodes.Status204NoContent)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
[HttpDelete("{id}")]
public async Task<IActionResult> Delete(int id) { ... }
```

---

## 本次未實作

- **XML 註解**（`/// <summary>`）：可在 Scalar UI 顯示 API 說明文字，但 .NET 10 內建 OpenAPI 的支援設定較繁瑣，本次跳過。
