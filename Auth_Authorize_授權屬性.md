# 認證與授權筆記：[Authorize] 與 [AllowAnonymous]

## 核心觀念

### 預設行為
ASP.NET Core 的 Controller 和 Action **預設任何人都能進**（包括未登入的訪客）。  
需要主動加上 `[Authorize]` 才會保護。

### 為什麼預設不是「只有登入才能進」？
因為登入頁本身也是一個 Action——如果預設鎖起來，未登入的人連登入頁都進不去，會卡死。

---

## [Authorize]

貼上後，框架在進入 Action 之前會先檢查「這個人有沒有登入？」，未登入會被導向登入頁。

### 貼在 class 上（推薦）
```csharp
[Authorize]
public class ExpensesController : Controller
{
    public IActionResult Index() { ... }   // 需要登入
    public IActionResult Create() { ... }  // 需要登入
}
```
整個 Controller 所有 Action 一次保護，**不容易漏貼**。

### 貼在單一 Action 上
```csharp
public class ExpensesController : Controller
{
    [Authorize]
    public IActionResult Index() { ... }   // 需要登入

    public IActionResult PublicStats() { ... }  // 任何人都能進
}
```
適合只有少數 Action 需要保護的情況，但容易漏貼造成資安問題。

### 設計原則：預設嚴格，例外開放
把 `[Authorize]` 貼在整個 Controller，只有特別需要公開的 Action 才加 `[AllowAnonymous]`。  
比「預設全開、一個一個加 `[Authorize]`」安全得多。

---

## [AllowAnonymous]

在已套用 `[Authorize]` 的 Controller 裡，讓特定 Action 豁免限制、任何人都能進。

```csharp
[Authorize]
public class ExpensesController : Controller
{
    public IActionResult Index() { ... }        // 需要登入

    [AllowAnonymous]
    public IActionResult PublicStats() { ... }  // 任何人都能進（豁免）
}
```

---

## 角色授權（RBAC）

**RBAC = Role-Based Access Control（角色型存取控制）**  
依照使用者的角色給予不同權限。

### 指定角色
```csharp
[Authorize(Roles = "admin")]
public IActionResult Approve() { ... }  // 只有 admin 才能進
```

### 允許多個角色（同一字串，逗號隔開）
```csharp
[Authorize(Roles = "admin,manager")]
public IActionResult Approve() { ... }  // admin 或 manager 都能進
```

> ⚠️ 注意：兩個角色要放在**同一個字串**裡，不能寫成兩個引號：
> ```csharp
> // 錯誤寫法（會出錯）
> [Authorize(Roles = "admin", "manager")]
>
> // 正確寫法
> [Authorize(Roles = "admin,manager")]
> ```

---

## 角色資訊存在哪裡？

### 寫入時機：登入時
角色以 `Claim` 的形式寫進 Cookie：

```csharp
// Controllers/LoginController.cs
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, model.Username),
    new Claim(ClaimTypes.Role, "admin")   // 角色寫進 Claim
};
```

> **一個 Claim = 一件事實**  
> 要表達兩件事，就寫兩個 Claim，不能把兩筆資料塞進同一個 Claim 物件。

### 讀取時機：每次 request
框架從 Cookie 裡讀出 Role Claim，比對 `[Authorize(Roles = "...")]` 指定的角色。  
不需要每次都去查資料庫。

### JWT 的情況
如果用 JWT，角色一樣是在登入時打包進 Token，之後每次 request 在 Header 帶著 Token，框架解開後讀取角色。

---

## ExpenseSystem 目前的實作

```csharp
// Controllers/LoginController.cs — 登入時寫入角色
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, model.Username),
    new Claim(ClaimTypes.Role, "admin")
};

// Controllers/ExpensesController.cs — 限定角色存取
[Authorize(Roles = "admin")]
public class ExpensesController : Controller
{
    // 所有 Action 均需 admin 角色才能存取
}
```

Git commit：`ab873cd` — `add role-based authorization (RBAC)`

---

## 本次學到的意外收穫

### ReturnUrl 機制
未登入的使用者被導向登入頁時，URL 會帶上 `?ReturnUrl=/Expenses`。  
登入成功後，系統會自動跳回原本要去的頁面，不需要自己寫這個邏輯。

### AccessDeniedPath
當使用者已登入但角色不符時（例如 employee 嘗試進入只有 admin 才能進的頁面），  
ASP.NET Core 會導向 `AccessDeniedPath` 設定的路徑。  
目前 ExpenseSystem 尚未自訂此路徑，預設打 `/Account/AccessDenied` 會 404。
