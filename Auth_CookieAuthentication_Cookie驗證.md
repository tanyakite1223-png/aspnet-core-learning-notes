# 認證與授權筆記:Cookie-based 驗證

---

## 為什麼需要 Cookie 驗證？

HTTP 是**無狀態（stateless）**的協定——每一個請求對伺服器來說都是全新的，伺服器不會自動記得「上一個請求是誰發的」。

解決方式：登入成功後，伺服器產生一個憑證存入瀏覽器的 **Cookie**。之後每次請求，瀏覽器會**自動**把 Cookie 帶上去，伺服器看到憑證就知道這是誰、是否已登入。

---

## ClaimsPrincipal / ClaimsIdentity / Claim 三層結構

| 現實比喻 | 程式概念 | 說明 |
|----------|----------|------|
| 身分證上每一欄資料 | **Claim**（聲明） | 例如姓名、角色 |
| 身分證本身 | **ClaimsIdentity**（身分） | 一組 Claim 的集合 |
| 拿著身分證的人 | **ClaimsPrincipal**（當事人） | 可持有多個 ClaimsIdentity |

一個人可以同時持有多張證件（身分證、護照、員工證），所以 `ClaimsPrincipal` 可以包含多個 `ClaimsIdentity`。

**建立順序：先有資料 → 裝進身分 → 交給當事人**

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, model.Username)
};

var identity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);
var principal = new ClaimsPrincipal(identity);

await HttpContext.SignInAsync(CookieAuthenticationDefaults.AuthenticationScheme, principal);
```

### 逐行說明

**第一步：準備資料欄位**

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, model.Username)
};
```

`Claim` 是「類型 + 值」的配對，一條 Claim 就是一個欄位。`ClaimTypes.Name` 是內建常數，代表「姓名」這個類型。如果要存更多資料，就多加幾條：

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, model.Username),
    new Claim(ClaimTypes.Email, "admin@example.com"),
    new Claim(ClaimTypes.Role, "Admin")
};
```

**第二步：把欄位裝進「身分證」**

```csharp
var identity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);
```

需要兩個參數：
- `claims`：剛才準備好的欄位資料
- `AuthenticationScheme`：這張身分證是用哪種方式驗證的（這裡是 Cookie）

指定驗證方式是因為系統可能同時支援多種登入方式（Cookie、Google、JWT…），這個參數讓系統知道這張身分證是哪種方式簽發的。

**第三步：建立「拿著身分證的人」**

```csharp
var principal = new ClaimsPrincipal(identity);
```

把身分證交給當事人。多一層的原因：一個人可以持有多張證件，`ClaimsPrincipal` 可以包含多個 `ClaimsIdentity`。

**第四步：簽進 Cookie**

```csharp
await HttpContext.SignInAsync(CookieAuthenticationDefaults.AuthenticationScheme, principal);
```

`SignInAsync` 會把 `principal` 裡的所有資料加密、序列化，寫入瀏覽器的 Cookie。之後每次請求，ASP.NET Core 自動解密並還原成 `ClaimsPrincipal`，存放在 `HttpContext.User`。

因此在任何 Controller 裡都可以這樣取得登入者姓名：

```csharp
var name = User.Identity?.Name; // 就是當初存進去的 model.Username
```

> 整體流程：**準備資料 → 裝進身分證 → 交給當事人 → 簽進 Cookie → 之後每次請求自動帶著走**

---

## Program.cs 設定

```csharp
// 註冊 Cookie 驗證服務
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.LoginPath = "/Login/Index"; // 未登入時自動導向這裡
    });

// Middleware 順序：Authentication 必須在 Authorization 之前
app.UseAuthentication();  // 先認證（確認是誰）
app.UseAuthorization();   // 再授權（確認有沒有權限）
```

> ⚠️ **順序很重要**：`UseAuthentication()` 必須在 `UseAuthorization()` 之前。
> 理由：要先知道「這個人是誰」，才能判斷「這個人有沒有權限」。順序搞反，授權時根本不知道在授權給誰。

---

## LoginViewModel.cs

```csharp
public class LoginViewModel
{
    [Required]
    public string Username { get; set; } = string.Empty;

    [Required]
    public string Password { get; set; } = string.Empty;
}
```

---

## LoginController.cs

```csharp
public class LoginController : Controller
{
    // GET：顯示登入頁
    public IActionResult Index()
    {
        return View();
    }

    // POST：驗證帳密、簽發 Cookie
    [HttpPost]
    public async Task<IActionResult> Index(LoginViewModel model)
    {
        if (!ModelState.IsValid)
            return View(model);

        if (model.Username != "admin" || model.Password != "1234")
        {
            ModelState.AddModelError("", "帳號或密碼錯誤");
            return View(model);
        }

        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.Name, model.Username)
        };
        var identity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);
        var principal = new ClaimsPrincipal(identity);

        await HttpContext.SignInAsync(CookieAuthenticationDefaults.AuthenticationScheme, principal);

        return RedirectToAction("Index", "Expenses");
    }

    // GET：登出
    public async Task<IActionResult> Logout()
    {
        await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
        return RedirectToAction("Index");
    }
}
```

---

## 保護 Controller：[Authorize]

在 Controller 類別上加 `[Authorize]`，整個 Controller 的所有 Action 都需要登入才能存取：

```csharp
[Authorize]
public class ExpensesController : Controller
{
    // 所有 Action 都受保護
}
```

未登入的使用者存取受保護的頁面時，會被自動導向 `LoginPath` 所設定的路徑。

---

## 常見錯誤

| 錯誤 | 說明 |
|------|------|
| `ModelState.IsValid` 條件方向寫反 | 應該是 `if (!ModelState.IsValid) return View(model)`，驗證**失敗**才回到頁面 |
| ClaimsIdentity 和 ClaimsPrincipal 建立順序搞反 | 要先有 claims → 建 identity → 再建 principal，有依賴順序 |
| `UseAuthentication()` 放在 `UseAuthorization()` 後面 | 必須先認證再授權 |

---

## 本次實作的檔案

| 新增 / 修改 | 檔案 |
|-------------|------|
| 新增 | `Models/LoginViewModel.cs` |
| 新增 | `Controllers/LoginController.cs` |
| 新增 | `Views/Login/Index.cshtml` |
| 修改 | `Program.cs`（加入驗證服務與 Middleware） |
| 修改 | `Controllers/ExpensesController.cs`（加入 `[Authorize]`） |
| 修改 | `Views/Expenses/Index.cshtml`（加入登出連結） |
