# ASP.NET Core MVC 筆記：Model Binding（模型繫結）

---

## 什麼是 Model Binding？

Model Binding 是 ASP.NET Core 的內建機制，負責：

> **把 HTTP Request 帶來的資料（表單、Route 參數、Query String），自動對應到 Action 方法的參數上。**

開發者不需要自己寫程式去解析 Request 資料，Model Binding 會自動完成配對與轉換。

---

## 配對原則：靠名稱匹配

Model Binding 的核心是**名稱匹配** —— 資料的 key 要跟 Action 參數名稱（或 Model 屬性名稱）一致，才能配對成功。

### 範例

表單送出的資料：
```
Name=Amber&Email=test@test.com&Message=Hello
```

Action 方法：
```csharp
[HttpPost]
public IActionResult Contact(ContactForm model)
{
    // model.Name = "Amber"
    // model.Email = "test@test.com"
    // model.Message = "Hello"
}
```

Model：
```csharp
public class ContactForm
{
    public string? Name { get; set; }
    public string? Email { get; set; }
    public string? Message { get; set; }
}
```

表單欄位的 `name` 屬性（由 `asp-for` 自動產生）和 Model 的屬性名稱一致，所以 Model Binding 能自動配對。

---

## 資料來源與優先順序

Model Binding 會依序從以下三個來源尋找資料：

| 優先順序 | 來源 | 說明 | 範例 |
|---------|------|------|------|
| 1 | **Form（表單）** | POST 送出的表單欄位 | `<input asp-for="Name" />` |
| 2 | **Route（路由參數）** | 網址路徑中的參數 | `/Product/Details/5` → `{id}` = 5 |
| 3 | **Query String** | 網址 `?` 後面的參數 | `?category=Electronics` |

### Route 和 Query String 可以併用

```csharp
public IActionResult Details(int id, string category)
```

網址：`/Product/Details/5?category=Electronics`
- `id` = 5 → 從 **Route** 來
- `category` = "Electronics" → 從 **Query String** 來

Model Binding 會自動從不同來源分別抓取對應的值。

---

## `asp-for` 的作用

在 View 中使用 `asp-for` Tag Helper：

```html
<input asp-for="Name" />
```

會自動產生帶有正確 `name` 屬性的 HTML：

```html
<input type="text" id="Name" name="Name" value="" />
```

這個 `name="Name"` 就是 Model Binding 用來配對 Model 屬性的依據。

---

## 完整實作範例

### Model（ContactForm.cs）

```csharp
using System.ComponentModel.DataAnnotations;

public class ContactForm
{
    [Display(Name = "姓名")]
    public string? Name { get; set; }

    [Display(Name = "信箱")]
    public string? Email { get; set; }

    [Display(Name = "訊息")]
    public string? Message { get; set; }
}
```

### Controller（ContactController.cs）

```csharp
using Microsoft.AspNetCore.Mvc;
using MyMvcApp.Models;

public class ContactController : Controller
{
    // GET: 顯示表單
    public IActionResult Contact()
    {
        return View();
    }

    // POST: 接收表單資料（Model Binding 自動將表單資料對應到 ContactForm）
    [HttpPost]
    public IActionResult Contact(ContactForm model)
    {
        return Content($"姓名：{model.Name}，信箱：{model.Email}，訊息：{model.Message}");
    }
}
```

### View（Contact.cshtml）

```html
@model MyMvcApp.Models.ContactForm

<form asp-controller="Contact" asp-action="Contact" method="post">
    <label asp-for="Name"></label>
    <input asp-for="Name" />

    <label asp-for="Email"></label>
    <input asp-for="Email" />

    <label asp-for="Message"></label>
    <input asp-for="Message" />

    <button type="submit">送出</button>
</form>
```

---

## 常見錯誤

### ❌ 表單欄位 name 與 Model 屬性名稱不一致

```html
<!-- 錯誤：name 是 "UserName"，但 Model 屬性叫 "Name" -->
<input name="UserName" />
```
→ Model Binding 配對失敗，`model.Name` 會是 **null**

### ✅ 使用 asp-for 自動產生正確的 name

```html
<input asp-for="Name" />
```
→ 自動產生 `name="Name"`，與 Model 屬性一致，配對成功

---

### ❌ 把 `asp-for` 和 `asp-action-for` 搞混

```html
<!-- 錯誤：沒有 asp-action-for 這個 Tag Helper -->
<input asp-action-for="Message" />
```

### ✅ input 綁定 Model 屬性用 `asp-for`

```html
<input asp-for="Message" />
```

> 💡 `asp-action` 是用在 `<form>` 上指定要送到哪個 Action，`asp-for` 是用在 `<input>` / `<label>` / `<span>` 上綁定 Model 屬性，兩者不要混淆。

---

## 重點整理

1. **Model Binding** 是 ASP.NET Core 自動把 Request 資料對應到 Action 參數的機制
2. 配對靠**名稱匹配**，名稱不一致就繫結不到（值為 null）
3. 資料來源優先順序：**Form → Route → Query String**
4. 三種來源可以**併用**，Model Binding 會分別從不同來源抓值
5. 使用 `asp-for` Tag Helper 可自動產生正確的 `name` 屬性，避免手動寫錯名稱
