# ASP.NET Core MVC 筆記：表單驗證 — Server-side Validation

## 什麼是 Server-side Validation（伺服器端驗證）

使用者送出表單後，資料傳到 Server，由 Server 檢查資料是否合法。如果不合法，就把錯誤訊息帶回表單頁面，告訴使用者哪裡填錯了。

---

## Server-side Validation 的三個關鍵要素

1. **Model**：加上驗證屬性（Data Annotation）
2. **Controller**：用 `ModelState.IsValid` 檢查驗證結果
3. **View**：用 Tag Helper 顯示錯誤訊息

---

## 一、Model — 驗證屬性（Data Annotation）

### 常用驗證屬性

| 驗證屬性 | 用途 | 範例 |
|---------|------|------|
| `[Required]` | 必填 | `[Required(ErrorMessage = "名稱為必填")]` |
| `[StringLength]` | 限制字串長度 | `[StringLength(50, ErrorMessage = "不能超過 50 個字")]` |
| `[Range]` | 限制數值範圍 | `[Range(1, 99999, ErrorMessage = "必須在 1~99999 之間")]` |
| `[RegularExpression]` | 用正規表達式檢查格式 | `[RegularExpression(@"^[a-zA-Z\u4e00-\u9fff]+$", ErrorMessage = "只能包含中文或英文")]` |
| `[Compare]` | 比較兩個欄位是否相同 | `[Compare("Price", ErrorMessage = "兩次輸入不一致")]` |

### 範例：Product Model

```csharp
using System.ComponentModel.DataAnnotations;

public class Product
{
    public int Id { get; set; }

    [Required(ErrorMessage = "產品名稱為必填")]
    [StringLength(50, ErrorMessage = "名稱不能超過 50 個字")]
    [Display(Name = "產品名稱")]
    public string? Name { get; set; }

    [Required(ErrorMessage = "價格為必填")]
    [Range(1, 99999, ErrorMessage = "價格必須在 1 到 99999 之間")]
    [Display(Name = "價格")]
    public decimal? Price { get; set; }

    [Display(Name = "描述")]
    [StringLength(200, ErrorMessage = "描述不能超過 200 個字")]
    public string? Description { get; set; }
}
```

### 驗證屬性的排列順序

驗證屬性的排列順序**不影響功能**，ASP.NET Core 會全部都檢查。但習慣上會把 `[Required]` 放最前面，因為「有沒有填」是最基本的檢查。

---

## 二、Controller — `ModelState.IsValid`

### 基本結構

```csharp
[HttpPost]
public IActionResult Create(Product product)
{
    if (ModelState.IsValid)
    {
        // 驗證通過，處理資料（例如存到資料庫）
        return RedirectToAction("Index");
    }

    // 驗證失敗，帶著原本的資料回到表單
    return View(product);
}
```

### 驗證流程

1. 使用者送出表單 → 資料送到 Server
2. Model Binding 把表單資料對應到 Model 的屬性
3. ASP.NET Core 根據驗證屬性自動做檢查，結果存到 `ModelState`
4. Action 方法用 `ModelState.IsValid` 判斷驗證是否通過
5. 通過 → `RedirectToAction` 導向其他頁面
6. 失敗 → `return View(product)` 帶回表單

### 為什麼驗證失敗時不能用 `RedirectToAction`？

`RedirectToAction` 會讓瀏覽器重新發一個新的 Request，這樣：
- 使用者填過的資料會**消失**
- `ModelState` 裡的錯誤訊息也會**消失**

`return View(product)` 則是直接把目前的資料和錯誤訊息帶回同一個頁面。

---

## 三、手動加入錯誤訊息 — `ModelState.AddModelError`

有些驗證無法靠 Data Annotation 完成（例如檢查資料庫是否有重複），需要在 Controller 裡手動加入錯誤。

### 語法

```csharp
ModelState.AddModelError("參數1", "錯誤訊息");
```

- **參數1 = `""`（空字串）**：錯誤不屬於任何欄位 → 顯示在 `asp-validation-summary` 區域
- **參數1 = `"欄位名稱"`**：錯誤屬於特定欄位 → 顯示在該欄位的 `asp-validation-for` 旁邊

### 範例

```csharp
[HttpPost]
public IActionResult Create(Product product)
{
    if (ModelState.IsValid)
    {
        // 檢查資料庫是否有重複名稱
        if (已經存在相同名稱的產品)
        {
            // 顯示在上方彙總區域
            ModelState.AddModelError("", "此產品名稱已存在！");
            return View(product);
        }

        // 沒有重複，正常處理
        return RedirectToAction("Index");
    }

    return View(product);
}
```

### Data Annotation vs AddModelError 的差別

| | Data Annotation | AddModelError |
|---|---|---|
| 驗證類型 | 格式層面（必填、長度、範圍） | 商業邏輯（資料庫重複、折扣碼是否有效） |
| 寫在哪裡 | Model 屬性上方 | Controller 的 Action 方法裡 |
| 是否需要查資料庫 | 不需要 | 通常需要 |
| 執行方式 | 自動檢查 | 手動撰寫判斷邏輯 |

---

## 四、View — 顯示錯誤訊息的 Tag Helper

### `asp-validation-for`：顯示個別欄位的錯誤

```html
<label asp-for="Name"></label>
<input asp-for="Name" />
<span asp-validation-for="Name"></span>
```

錯誤訊息會顯示在 `<span>` 的位置，緊跟在欄位旁邊。

### `asp-validation-summary`：顯示錯誤彙總

```html
<div asp-validation-summary="ModelOnly"></div>
```

#### 三種模式

| 模式 | 顯示內容 | 說明 |
|------|---------|------|
| `None` | 不顯示 | 等於沒加 |
| `ModelOnly` | 只顯示不屬於特定欄位的錯誤 | ⭐ 最常用，搭配 `asp-validation-for` 不會重複 |
| `All` | 顯示所有錯誤 | 會跟 `asp-validation-for` 重複顯示 |

### 完整 View 範例

```html
@model MyMvcApp.Models.Product

<form asp-controller="Product" asp-action="Create" method="Post">
    <div asp-validation-summary="ModelOnly"></div>

    <label asp-for="Name"></label>
    <input asp-for="Name" />
    <span asp-validation-for="Name"></span>

    <label asp-for="Price"></label>
    <input asp-for="Price" />
    <span asp-validation-for="Price"></span>

    <label asp-for="Description"></label>
    <input asp-for="Description" />
    <span asp-validation-for="Description"></span>

    <button type="submit">送出</button>
</form>
```

---

## 常見錯誤

### ❌ 驗證失敗時用 RedirectToAction

```csharp
if (!ModelState.IsValid)
{
    return RedirectToAction("Create"); // ❌ 資料和錯誤訊息都會消失
}
```

### ✅ 驗證失敗時用 return View(model)

```csharp
if (!ModelState.IsValid)
{
    return View(product); // ✅ 保留資料和錯誤訊息
}
```

---

### ❌ AddModelError 後繼續往下執行

```csharp
if (ModelState.IsValid)
{
    ModelState.AddModelError("", "此產品名稱已存在！");
    return RedirectToAction("Index"); // ❌ 加了錯誤訊息但直接跳走，使用者看不到
}
```

### ✅ AddModelError 後要 return View

```csharp
if (ModelState.IsValid)
{
    ModelState.AddModelError("", "此產品名稱已存在！");
    return View(product); // ✅ 帶回表單顯示錯誤訊息
}
```

---

### ❌ 搞混 AddModelError 和 Data Annotation 的用途

```csharp
// ❌ 這種格式檢查應該用 [Required]，不需要手動寫
if (string.IsNullOrEmpty(product.Name))
{
    ModelState.AddModelError("Name", "名稱為必填");
}
```

### ✅ AddModelError 用在需要查資料庫的商業邏輯

```csharp
// ✅ 這種需要查資料庫的邏輯才用 AddModelError
if (已經存在相同名稱的產品)
{
    ModelState.AddModelError("", "此產品名稱已存在！");
}
```

---

## 複習題

### Q1：請寫出 Controller 裡 `[HttpPost]` Action 方法驗證的完整結構

```csharp
[HttpPost]
public IActionResult Create(Product product)
{
    if (ModelState.IsValid)
    {
        // 驗證通過，處理資料
        return RedirectToAction("Index");
    }

    // 驗證失敗，帶回表單
    return View(product);
}
```

> 注意：回傳型別是 `IActionResult`，不是 `string`。

### Q2：以下兩行的錯誤訊息分別顯示在哪裡？

```csharp
ModelState.AddModelError("", "系統錯誤！");
ModelState.AddModelError("Name", "名稱有問題！");
```

- `""` → 顯示在 `<div asp-validation-summary>` 所在的位置（整張表單層級的錯誤）
- `"Name"` → 顯示在 Name 欄位旁邊的 `asp-validation-for="Name"` 位置

### Q3：列出五個驗證屬性及用途

| 驗證屬性 | 用途 |
|---------|------|
| `[Required]` | 必填 |
| `[StringLength]` | 限制字串長度 |
| `[Range]` | 限制數值範圍 |
| `[RegularExpression]` | 用正規表達式檢查格式 |
| `[Compare]` | 比較兩個欄位的值是否相同 |

### Q4：`asp-validation-summary` 最常用的模式是哪一種？為什麼？

`ModelOnly` 最常用。因為它只顯示整張表單層級的錯誤（用 `AddModelError("", "...")` 加的），個別欄位的錯誤交給 `asp-validation-for` 處理，各管各的不會重複。

### Q5：驗證失敗時，為什麼不能用 `RedirectToAction`？

- `RedirectToAction` → 瀏覽器會發一個**新的 Request**，使用者填過的資料和 `ModelState` 裡的錯誤訊息都會**消失**
- `return View(model)` → 直接帶回原來的畫面，資料和錯誤訊息都會**保留**
