# 認證與授權筆記:角色授權（RBAC）

## 什麼是 RBAC

RBAC（Role-Based Access Control，角色型存取控制）：
- 使用者被指派**角色**（例如：Employee、Manager）
- 角色決定能做什麼
- 程式在對的地方檢查角色

`[Authorize]` 只能確認「有沒有登入」，RBAC 進一步判斷「登入的人可以做什麼」。

---

## 實作流程

### 1. 登入時放入角色 Claim

在建立 `ClaimsIdentity` 時，用 `ClaimTypes.Role` 加入角色資訊：

```csharp
// LoginController.cs — 依帳號分流，給予不同角色
List<Claim> claims;
if (model.Username == "admin" && model.Password == "1234")
{
    claims = new List<Claim>
    {
        new Claim(ClaimTypes.Name, model.Username),
        new Claim(ClaimTypes.Role, "Manager")
    };
}
else if (model.Username == "amber" && model.Password == "5678")
{
    claims = new List<Claim>
    {
        new Claim(ClaimTypes.Name, model.Username),
        new Claim(ClaimTypes.Role, "Employee")
    };
}
else
{
    ModelState.AddModelError("", "帳號或密碼錯誤");
    return View(model);
}
```

> 每個角色建立自己的 Claims 清單，不是在同一個清單裡「選」角色。

### 2. Controller 上加角色限制

```csharp
// 整個 Controller：任何登入者皆可進入
[Authorize]
public class ExpensesController : Controller { ... }

// 特定 Action：只有 Manager 可以執行
[Authorize(Roles = "Manager")]
[HttpGet]
public IActionResult Delete(int id) { ... }

[Authorize(Roles = "Manager")]
[HttpPost]
[ActionName("Delete")]
public IActionResult DeleteConfirmed(int id) { ... }
```

允許多個角色時，用逗號隔開：
```csharp
[Authorize(Roles = "Manager,Admin")]
```

### 3. 設定授權失敗的導向頁面

```csharp
// Program.cs
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.LoginPath = "/Login/Index";
        options.AccessDeniedPath = "/Expenses/Index"; // 授權失敗時導向這裡
    });
```

若未設定 `AccessDeniedPath`，Cookie 驗證預設導向 `/Account/AccessDenied`（若該頁面不存在會 404）。

### 4. View 中依角色控制顯示內容

```cshtml
@* Views/Expenses/Index.cshtml *@
@if (User.IsInRole("Manager"))
{
    <a asp-action="Delete" asp-route-id="@item.ExpenseId">刪除</a>
}
```

> `User.IsInRole("Manager")` 回傳 bool，不需要寫 `== true`。

---

## 401 vs 403

| 狀態碼 | 意義 | 情境 |
|--------|------|------|
| 401 Unauthorized | 你是誰？我不認識你 | 未登入就存取需要驗證的頁面 |
| 403 Forbidden | 我認識你，但你不能進來 | 登入了，但權限不夠 |

### Cookie 驗證的行為

Cookie 驗證會自動將 403 轉成 **302 redirect**，導向 `AccessDeniedPath`。
- 優點：使用者體驗較好（導回首頁，不是看到錯誤頁）
- 注意：**Web API 不適合這樣做** — 前端打 API 期待收到 403，不是 redirect

---

## 排錯觀念

授權失敗後看到 404 → **先看 URL**：URL 告訴你它想導去哪裡，再決定要建那個頁面，還是改 `AccessDeniedPath`。

---

## 本次實作結果（ExpenseSystem）

- 兩組測試帳號：`admin/1234`（Manager）、`amber/5678`（Employee）
- Employee 登入後：看不到刪除按鈕；直接存取刪除 URL 會被導回 Index
- Manager 登入後：可正常刪除
- git commit：`2d7e378`（Role Claim 授權）
