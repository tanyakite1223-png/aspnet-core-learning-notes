# ASP.NET Core MVC 筆記：Partial View（部分檢視）

---

## Partial View 是什麼？

Partial View 是頁面中**可重複使用的 HTML 區塊**，抽出來變成獨立的 `.cshtml` 檔案，可以在不同頁面中嵌入使用。

**用途：**
- 避免重複撰寫相同的 HTML
- 同一個區塊可以在不同頁面顯示不同的資料

**與 Layout 的差異：**

| | Layout | Partial View |
|---|---|---|
| 用途 | 整個頁面的外框（導覽列、頁尾） | 頁面內某個可重複使用的區塊 |
| 位置 | `Views/Shared/_Layout.cshtml` | `Views/Shared/_Xxx.cshtml` |

---

## 命名慣例

檔名用**底線 `_` 開頭**，代表這是一個「片段」而不是完整頁面。

```
_ProductCard.cshtml   ✅
ProductCard.cshtml    ❌（沒有底線）
```

---

## 建立 Partial View

存放位置：`Views/Shared/` 資料夾下。

Partial View 只包含 HTML 片段，**不能有完整頁面結構**。

```html
<!-- ✅ 正確：只有 HTML 片段 -->
<div style="border: 1px solid #ccc; padding: 10px;">
    <h3>商品名稱</h3>
    <p>價格：$100</p>
    <button>加入購物車</button>
</div>
```

---

## 使用 Partial View

在其他 View 中用 `<partial>` Tag Helper 嵌入：

```html
<!-- 不傳資料 -->
<partial name="_ProductCard" />

<!-- 傳資料 -->
<partial name="_ProductCard" model="@Model" />
```

---

## 傳資料給 Partial View

### 完整資料流

```
Controller → return View(product)
    ↓
View（@model 宣告型別，<partial model="@Model" /> 往下傳）
    ↓
Partial View（@model 宣告型別，@Model.屬性 顯示資料）
```

### 1. Controller

```csharp
public IActionResult Index()
{
    var product = new Product
    {
        Name = "藍牙耳機",
        Price = 1500
    };
    return View(product);
}
```

### 2. View（Index.cshtml）

```cshtml
@model MyMvcApp.Models.Product
@{
    ViewData["Title"] = "Home Page";
}

<partial name="_ProductCard" model="@Model" />
```

### 3. Partial View（_ProductCard.cshtml）

```cshtml
@model MyMvcApp.Models.Product
<div style="border: 1px solid #ccc; padding: 10px; margin: 10px;">
    <h3>@Model.Name</h3>
    <p>價格：@Model.Price</p>
    <button>加入購物車</button>
</div>
```

---

## @model 與 @Model 的差異

| | 寫法 | 意思 |
|---|---|---|
| `@model`（小寫 m）| `@model MyMvcApp.Models.Product` | **宣告**這個 View 接收的資料型別 |
| `@Model`（大寫 M）| `@Model.Name` | **使用**Controller 傳進來的實際物件 |

> 💡 只要寫了 `@model`，ASP.NET Core 就自動準備好 `@Model` 給你用，名字是固定的，不是自己取的。

---

## 常見錯誤

### ❌ Partial View 包含完整頁面結構

```cshtml
<!-- ❌ 錯誤：不應該有 html / head / body / @RenderBody() -->
<!DOCTYPE html>
<html>
<head>...</head>
<body>
    <div>商品內容</div>
    <div>@RenderBody()</div>
</body>
</html>
```

```cshtml
<!-- ✅ 正確：只有 HTML 片段 -->
<div>
    <h3>@Model.Name</h3>
    <p>價格：@Model.Price</p>
</div>
```

---

### ❌ @model 後面加分號

```cshtml
<!-- ❌ 錯誤：@model 是 Razor 特殊指令，不是 C# 程式碼，不加分號 -->
@model MyMvcApp.Models.Product;
```

```cshtml
<!-- ✅ 正確 -->
@model MyMvcApp.Models.Product
```

> 💡 Razor 特殊指令（`@model`、`@using`、`@inject` 等）不加分號。
> `@{ }` 裡面的 C# 程式碼才需要加分號。

---

### ❌ @model 放在 @{ } 裡面

```cshtml
<!-- ❌ 錯誤 -->
@{
    @model MyMvcApp.Models.Product
    ViewData["Title"] = "Home Page";
}
```

```cshtml
<!-- ✅ 正確：@model 必須放在第一行，在任何程式碼之前 -->
@model MyMvcApp.Models.Product
@{
    ViewData["Title"] = "Home Page";
}
```

---

### ❌ @model 後面加分號

```cshtml
<!-- ❌ 錯誤：@model 不需要分號 -->
@model MyMvcApp.Models.Product;
```

```cshtml
<!-- ✅ 正確 -->
@model MyMvcApp.Models.Product
```
