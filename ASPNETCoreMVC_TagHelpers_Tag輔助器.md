# ASP.NET Core MVC 筆記：Tag Helpers — 常用的 Form、Anchor、Input 等

## Tag Helpers 是什麼？

Tag Helpers 是 Razor 提供的特殊屬性（以 `asp-` 開頭），讓你在 HTML 標籤上直接跟 ASP.NET Core 的 Controller、Action、Model 綁定。

**核心概念：Tag Helpers 在伺服器端執行，把 `asp-*` 屬性轉換成標準 HTML，再傳給瀏覽器。瀏覽器看不到 `asp-*`，只看到純 HTML。**

---

## 一、Anchor Tag Helper（連結）

### 語法
```html
<a asp-controller="Home" asp-action="Index">回首頁</a>
```

### 產生的 HTML
```html
<a href="/">回首頁</a>
```

### 優點

**1. 有 Route 參數時，網址自動組合，不容易拼錯**

例如要連到某個產品的詳細頁面：

```html
<!-- 寫死：要自己拼網址，容易出錯 -->
<a href="/Product/Detail?id=@product.Id">看詳情</a>

<!-- Tag Helper：自動組合，清楚易讀 -->
<a asp-controller="Product" asp-action="Detail" asp-route-id="@product.Id">看詳情</a>
```

**2. 可讀性更好，看起來像標準 HTML**

Tag Helpers 讓 Razor View 更乾淨易讀，同時也有 Visual Studio 的 IntelliSense 支援。

> ⚠️ 注意：打錯 Controller 或 Action 名稱時，Visual Studio **不會**顯示錯誤或警告，執行後只會產生錯誤的網址，需自行測試確認。

---

## 二、Form Tag Helper（表單）

### 語法
```html
<form asp-controller="Home" asp-action="About" method="post">
    <button type="submit">送出</button>
</form>
```

### 產生的 HTML
```html
<form action="/Home/About" method="post">
    <button type="submit">送出</button>
    <input name="__RequestVerificationToken" type="hidden" value="..." />
</form>
```

### 重點
- `asp-controller` + `asp-action` → 轉換成 `action="/Controller/Action"`
- 自動加上 `__RequestVerificationToken`（隱藏欄位）

### CSRF Token（防跨站請求偽造）
Form Tag Helper 會自動產生一個隱藏的 `__RequestVerificationToken` 欄位。

**作用：** 伺服器驗證這個 Token，確認表單是從自己的網站送出的，防止惡意網站偷偷發出請求。

**不需要自己寫，框架自動處理。**

---

## 三、Input Tag Helper（輸入欄位）

### 前提：View 必須宣告 Model

```razor
@model MyMvcApp.Models.Product
```

### 語法
```html
<input asp-for="Name" />
<input asp-for="Price" />
```

### 產生的 HTML
```html
<input type="text" id="Name" name="Name" value="" />
<input type="number" data-val="true" data-val-required="The Price field is required."
       id="Price" name="Price" value="" />
```

### asp-for 自動幫你做的三件事

| 自動產生 | 說明 |
|---|---|
| `id` 和 `name` | 對應 Model 屬性名稱，表單送出時用於綁定資料 |
| `type` | 根據資料型別自動決定（string → text、int → number） |
| 驗證屬性 | 根據型別是否可為 null 自動加上必填驗證 |

### 型別與 type 對應
| C# 型別 | HTML type |
|---|---|
| `string` | `text` |
| `int` | `number` |

### 可為 null 與驗證的關係
| 宣告方式 | 是否必填 |
|---|---|
| `string?` | ❌ 允許空白，不加驗證 |
| `int` | ✅ 必填，自動加上 `data-val-required` |

---

## 常見錯誤

### ❌ 屬性名稱打錯
```html
<!-- 錯誤：多打了 asp- -->
<a asp-asp-controller="Home" asp-action="Index">回首頁</a>

<!-- 錯誤：拼字錯誤 aps → asp -->
<a asp-controller="Home" aps-action="Index">回首頁</a>
```

### ✅ 正確寫法
```html
<a asp-controller="Home" asp-action="Index">回首頁</a>
```

---

## 總整理

| Tag Helper | 用在哪 | 主要屬性 |
|---|---|---|
| Anchor | `<a>` | `asp-controller`、`asp-action` |
| Form | `<form>` | `asp-controller`、`asp-action` |
| Input | `<input>` | `asp-for` |
