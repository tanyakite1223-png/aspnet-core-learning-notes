# Web API 筆記：RESTful 設計原則（HTTP Method 對應 CRUD）

## 核心觀念：前後端分離的資料流

### MVC vs Web API 的差異

| 項目 | MVC | Web API |
|------|-----|---------|
| 前後端 | 合在一起（Controller 產生 View） | 分開（API 只管資料） |
| 回傳內容 | HTML 畫面 | JSON 資料 |
| HTTP Method | 主要用 GET + POST | GET、POST、PUT、DELETE 各有語意 |

### 資料流

```
前端 App（React / 手機 App）
    ↓ 發送 HTTP 請求（帶 JSON 資料）
後端 Web API（接收請求）
    ↓ 處理 + 存取資料庫
後端 Web API（回傳回應）
    ↓ HTTP 狀態碼 + JSON 資料
前端 App（收到結果，更新畫面）
```

重點：**前端主動發請求，後端被動接收**。API 只負責「收到什麼資料、回傳什麼資料」，畫面由前端自己處理。

---

## REST 的 CRUD 對應

HTTP Method 的語意是 HTTP 協定規格書定義的，不是 ASP.NET 發明的：

| CRUD 操作 | HTTP Method | 路由範例 | 說明 |
|-----------|------------|---------|------|
| Create（新增） | POST | `POST /api/expenses` | 建立新資源 |
| Read（查全部） | GET | `GET /api/expenses` | 取得全部資源 |
| Read（查單筆） | GET | `GET /api/expenses/3` | 取得指定資源 |
| Update（修改） | PUT | `PUT /api/expenses/3` | 整筆替換（全部欄位都要傳） |
| Delete（刪除） | DELETE | `DELETE /api/expenses/3` | 刪除指定資源 |

### 路由規律

- **不需要 id**：POST（新增，還沒有 id）、GET 全部（讀全部不需指定）
- **需要 id**：GET 單筆、PUT、DELETE — 都需要告訴 API「操作哪一筆」

### PUT vs PATCH

- **PUT** = 整筆替換（全部欄位都要傳，不管有沒有改）
- **PATCH** = 部分更新（只傳要改的欄位）
- 實務上大部分 API 先用 PUT 就好

---

## HTTP 狀態碼對應

| 操作 | 成功狀態碼 | 說明 |
|------|-----------|------|
| GET | 200 OK | 成功，回傳資料 |
| POST | **201 Created** | 成功，有新資源被建立，回傳新資料 + Location header |
| PUT | **204 No Content** | 成功，不需回傳資料（前端已知道改了什麼） |
| DELETE | **204 No Content** | 成功，不需回傳資料（東西已刪除） |

### 為什麼 POST 回 201、PUT/DELETE 回 204？

- **201**：新增後前端需要知道新資源的 Id（資料庫自動產生），所以後端要把完整資料送回去
- **204**：修改/刪除時前端已經知道是哪筆資料，不需要後端再回傳

### 錯誤狀態碼

- **400 Bad Request**：請求有問題（例如 PUT 時路由 id 與 body 的 ExpenseId 不一致）
- **404 Not Found**：請求合法但資源不存在

---

## 實作程式碼

### POST（Create）

```csharp
[HttpPost]
public async Task<IActionResult> Create(Expense expense)
{
    _context.Expenses.Add(expense);
    await _context.SaveChangesAsync();
    return CreatedAtAction("GetById", new { id = expense.ExpenseId }, expense);
}
```

**CreatedAtAction 三個參數**：
- `"GetById"` — 告訴呼叫端可以用哪個 Action 查詢新資源
- `new { id = expense.ExpenseId }` — 組出查詢路由的參數
- `expense` — 新建的資料放在 response body

組合效果：回傳 201 + Location header（如 `/api/expenses/10`）+ 資料

### PUT（Update）

```csharp
[HttpPut("{id}")]
public async Task<IActionResult> Update(int id, Expense expense)
{
    if (id != expense.ExpenseId)
        return BadRequest();

    _context.Entry(expense).State = EntityState.Modified;

    try
    {
        await _context.SaveChangesAsync();
        return NoContent();
    }
    catch (DbUpdateConcurrencyException)
    {
        bool exists = _context.Expenses.Any(e => e.ExpenseId == id);
        if (!exists)
            return NotFound();
        throw;
    }
}
```

**重點**：
- 兩個參數來源不同：`id` 從路由、`expense` 從 body（JSON）
- 防禦性檢查：路由 id 與 body 的 ExpenseId 必須一致
- `EntityState.Modified`：告訴 EF Core「整個物件被改過，全部欄位都更新」
- `DbUpdateConcurrencyException`：處理並行修改衝突

### DELETE

```csharp
[HttpDelete("{id}")]
public async Task<IActionResult> Delete(int id)
{
    var expense = await _context.Expenses.FindAsync(id);
    if (expense is null)
        return NotFound();

    _context.Expenses.Remove(expense);
    await _context.SaveChangesAsync();
    return NoContent();
}
```

**重點**：
- 先查再刪 — 不能直接刪不存在的東西
- `FindAsync` 會先查 EF Core 快取，比 `FirstOrDefaultAsync` 效率更好

---

## API 參數來源

| 參數來源 | 說明 | 範例 |
|---------|------|------|
| 路由（Route） | 從 URL 路徑取得 | `/api/expenses/3` → `int id = 3` |
| Request Body | 從 JSON 取得（複雜型別） | `{ "title": "午餐", "amount": 150 }` → `Expense expense` |

`[ApiController]` 會自動判斷：簡單型別（int、string）從路由綁定，複雜型別（Expense）從 body 綁定。

---

## async / await 規則

- 方法名稱結尾是 `Async` 的就加 `await`（如 `SaveChangesAsync()`、`FindAsync()`）
- 加了 `await` 的方法，其外層方法簽名要改成 `async Task<回傳型別>`

---

## 測試工具

- **Scalar UI**：本 session 安裝了 `Scalar.AspNetCore` 套件
- 存取網址：`http://localhost:5242/scalar/v1`
- 可直接在瀏覽器中測試各個 endpoint（選 HTTP Method → Try it out → 填 JSON → Execute）

---

## 本次學習心得

- MVC 被瀏覽器限制只能用 GET/POST；Web API 的呼叫端是程式碼，可以自由選擇 HTTP Method
- REST 就是「按照 HTTP 原本的語意來用」，看到 Method 就知道要做什麼操作
- API 思考方式：只想「收到什麼資料、回傳什麼結果」，不用管畫面
