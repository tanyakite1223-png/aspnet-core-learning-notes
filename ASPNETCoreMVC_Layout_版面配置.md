# ASP.NET Core MVC 筆記：Layout 與 _ViewStart（版面配置）

---

## 一、為什麼需要 Layout？

網站中很多元素每頁都一樣，例如：
- 導覽列（Navigation Bar）
- 頁尾（Footer）
- 共用的 CSS / JS 引用

如果每個 View 都自己寫一份，不但重複，維護也很麻煩。

**解決方案：** 使用 Layout（版面配置）把共用的部分集中管理。

---

## 二、_ViewStart.cshtml

位置：`Views/_ViewStart.cshtml`

```csharp
@{
    Layout = "_Layout";
}
```

- **每次**任何 View 被執行前，都會先跑這個檔案
- 用來統一設定所有 View 要使用哪個 Layout
- 不需要每個 View 自己宣告 Layout

---

## 三、_Layout.cshtml

位置：`Views/Shared/_Layout.cshtml`

基本結構：

```html
<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"] - MyMvcApp</title>
    <!-- 共用 CSS -->
</head>
<body>
    <header>
        <!-- 導覽列：每頁共用 -->
    </header>

    <div class="container">
        @RenderBody()   <!-- ← 每個 View 自己的內容插入這裡 -->
    </div>

    <footer>
        <!-- 頁尾：每頁共用 -->
    </footer>

    <!-- 共用 JS -->
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

---

## 四、重要語法說明

### @ViewData["Title"]
- Layout 裡的標題佔位符號
- 每個 View 自己設定值：
```csharp
@{
    ViewData["Title"] = "Home Page";
}
```
- 瀏覽器分頁會顯示：`Home Page - MyMvcApp`

### @RenderBody()
- Layout 中的**主要內容佔位符號**
- 每個 View 自己的內容會插入到這個位置
- 每個 Layout 只能有一個 `@RenderBody()`

### @RenderSection / @RenderSectionAsync

```html
@await RenderSectionAsync("Scripts", required: false)
```

- 在 Layout 中預留**選擇性**的插入位置
- `required` 是寫在 **Layout** 裡的設定，不是 View 裡的

| Layout 設定 | View 的義務 |
|-------------|------------|
| `required: false` | 愛填不填，不填也不會出錯 |
| `required: true` | **每個 View 都必須**提供這個區塊，否則執行時報錯 |

**View 中使用 Section 的寫法：**

```html
@section Scripts {
    <script src="~/js/my-page.js"></script>
}
```

**不需要額外 JS 的 View（required: false 時可以完全不寫）：**

```html
@{
    ViewData["Title"] = "Home Page";
}

<div class="text-center">
    <h1>Welcome</h1>
</div>
```

**需要額外 JS 的 View：**

```html
@{
    ViewData["Title"] = "Home Page";
}

<div class="text-center">
    <h1>Welcome</h1>
</div>

@section Scripts {
    <script src="~/js/my-page.js"></script>
}
```

---

## 五、@RenderBody() vs @RenderSection 比較

| | @RenderBody() | @RenderSection |
|---|---|---|
| 用途 | View 的主要內容 | 選擇性的額外插入點 |
| View 是否必填 | 是（每個 View 都有主要內容） | 視 `required` 設定而定 |
| 數量 | Layout 只能有一個 | 可以有多個，名稱不同即可 |
| 常見用途 | 頁面主體 | 頁面專屬的 JS |

---

## 六、整體運作流程

```
使用者請求 /Home/Index
        ↓
_ViewStart.cshtml 先執行 → 設定 Layout = "_Layout"
        ↓
Index.cshtml 執行 → 產生主要內容
        ↓
_Layout.cshtml 執行 → @RenderBody() 插入 Index 的內容
        ↓
瀏覽器收到完整 HTML（含導覽列、內容、頁尾）
```

---

## 七、各檔案位置整理

| 檔案 | 位置 | 用途 |
|------|------|------|
| `_ViewStart.cshtml` | `Views/` | 統一設定 Layout |
| `_Layout.cshtml` | `Views/Shared/` | 共用版面框架 |
| `_ViewImports.cshtml` | `Views/` | 共用 using、Tag Helpers 等 |
| `_ValidationScriptsPartial.cshtml` | `Views/Shared/` | 表單驗證 JS（之後會用到） |

---

## 八、常見錯誤 ❌ vs ✅

### Layout 名稱不一致
```csharp
// _ViewStart.cshtml 寫的是：
Layout = "_Layout";

// 但 Shared 資料夾裡的檔案叫：
_Layout1.cshtml   ❌ 找不到，會出錯

_Layout.cshtml    ✅ 名稱一致才能正確套用
```

### 以為 @RenderBody() 可以省略
```html
<!-- ❌ Layout 裡沒有 @RenderBody()，View 的內容不會出現 -->
<body>
    <header>...</header>
    <footer>...</footer>
</body>

<!-- ✅ 必須有 @RenderBody() -->
<body>
    <header>...</header>
    @RenderBody()
    <footer>...</footer>
</body>
```

### 以為修改導覽列要改每個 View
```
❌ 在 Index.cshtml、Privacy.cshtml、About.cshtml 各自修改導覽列

✅ 只需修改 _Layout.cshtml，所有頁面自動更新
```

### 誤以為 required 是寫在 View 裡
```
❌ 以為每個 View 都要宣告 required: false

✅ required 是 Layout 裡 RenderSectionAsync 的參數
   View 只負責決定要不要提供 @section 內容
```
