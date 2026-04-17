# ASP.NET Core MVC 筆記：Model 定義與 Data Annotation（驗證屬性）

## 什麼是 Data Annotation（資料註解）

Data Annotation 是一種用**方括號 `[ ]` 標記在 Model 屬性上方**的驗證規則，也稱為驗證屬性（Validation Attributes）。

核心概念：**驗證規則跟著 Model 走，而不是散落在 Controller 裡面**。

### 傳統做法 vs Data Annotation

```csharp
// ❌ 傳統做法：在 Controller 裡用 if 一個一個檢查
if (string.IsNullOrEmpty(name))
{
    // 錯誤處理...
}
if (price < 0)
{
    // 錯誤處理...
}
// 欄位越多，程式越冗長、越難維護
```

```csharp
// ✅ Data Annotation：規則直接標記在 Model 上
[Required(ErrorMessage = "產品名稱為必填")]
[StringLength(50, ErrorMessage = "名稱不能超過 50 個字")]
public string? Name { get; set; }
```

---

## 使用前置作業

在 Model 檔案最上方加上：

```csharp
using System.ComponentModel.DataAnnotations;
```

### 關於 namespace

Model 檔案建議加上 `namespace`，例如 `namespace MyMvcApp.Models`。

```csharp
using System.ComponentModel.DataAnnotations;

namespace MyMvcApp.Models  // 建議加上
{
    public class Product
    {
        // ...
    }
}
```

原因：
1. **避免類別名稱衝突** — 不加 namespace 的話，所有類別都在全域命名空間，如果有兩個類別同名就無法區分
2. **其他檔案引用時更清楚** — 例如 View 裡的 `@model MyMvcApp.Models.Product` 就是靠 namespace 定位的
3. **符合業界慣例** — 實務專案幾乎都會用 namespace

> 💡 Visual Studio 建立新檔案時，會自動根據資料夾路徑產生 namespace（例如檔案放在 `Models` 資料夾就會自動帶 `namespace MyMvcApp.Models`），通常不需要自己手打。

---

## 常用的 Data Annotation

### 驗證類

| Attribute | 用途 | 範例 |
|---|---|---|
| `[Required]` | 必填欄位 | `[Required(ErrorMessage = "必填")]` |
| `[StringLength]` | 限制字串長度 | `[StringLength(50, ErrorMessage = "最多 50 字")]` |
| `[Range]` | 限制數值範圍 | `[Range(1, 99999, ErrorMessage = "1~99999")]` |
| `[EmailAddress]` | 驗證 Email 格式 | `[EmailAddress(ErrorMessage = "Email 格式錯誤")]` |
| `[Phone]` | 驗證電話格式 | `[Phone(ErrorMessage = "電話格式錯誤")]` |
| `[RegularExpression]` | 自訂正則表達式 | `[RegularExpression(@"^[A-Z]...", ErrorMessage = "格式錯誤")]` |
| `[Compare]` | 比對兩個欄位是否一致 | `[Compare("Password", ErrorMessage = "密碼不一致")]` |

### 顯示類

| Attribute | 用途 | 範例 |
|---|---|---|
| `[Display(Name = "...")]` | 自訂 label 顯示名稱 | `[Display(Name = "產品名稱")]` |

> ⚠️ `[Display(Name = "...")]` 裡面的 `Name` 是 Display 這個 Attribute 的固定參數名稱，意思是「顯示名稱」，不是指 Model 的 Name 屬性。不管放在哪個屬性上面，都是寫 `Name = "..."`。

---

## 完整 Model 範例

```csharp
using System.ComponentModel.DataAnnotations;

namespace MyMvcApp.Models
{
    public class Product
    {
        public int Id { get; set; }  // 不需要驗證，由資料庫自動產生

        [Required(ErrorMessage = "產品名稱為必填")]
        [StringLength(50, ErrorMessage = "名稱不能超過 50 個字")]
        [Display(Name = "產品名稱")]
        public string? Name { get; set; }

        [Required(ErrorMessage = "價格為必填")]
        [Range(1, 99999, ErrorMessage = "價格必須在 1 到 99999 之間")]
        [Display(Name = "價格")]
        public decimal? Price { get; set; }

        [StringLength(200, ErrorMessage = "描述不能超過 200 個字")]
        [Display(Name = "描述")]
        public string? Description { get; set; }
    }
}
```

---

## 驗證流程：Model → Controller → View

### 1. Controller — 用 `ModelState.IsValid` 檢查

```csharp
[HttpGet]
public IActionResult Create()
{
    return View();  // 顯示空白表單
}

[HttpPost]
public IActionResult Create(Product product)
{
    if (ModelState.IsValid)
    {
        // 驗證通過，處理資料（例如存到資料庫）
        return RedirectToAction("Index");
    }

    // 驗證沒通過，把資料和錯誤訊息帶回表單
    return View(product);
}
```

流程說明：
1. 使用者填完表單按送出
2. ASP.NET Core 自動把表單資料對應到 Model 物件（Model Binding）
3. 對應的同時，**自動檢查所有 Data Annotation 的規則**
4. 檢查結果存在 `ModelState` 裡面
5. 在 Action 裡用 `ModelState.IsValid` 判斷是否通過

### 2. View — 用 Tag Helpers 顯示表單和錯誤訊息

```html
@model MyMvcApp.Models.Product

<form asp-controller="Product" asp-action="Create" method="Post">
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

重要的 Tag Helpers：
- `asp-for` — 把 `<input>` 綁定到 Model 的屬性
- `asp-validation-for` — 顯示該欄位的驗證錯誤訊息
- `<label asp-for="...">` — 自動顯示 `[Display(Name)]` 設定的名稱

---

## 常見錯誤

### 1. 值型別不需要 `[Required]`

```csharp
// ❌ int 是值型別，預設值為 0，永遠有值，[Required] 無意義
[Required]
public int Id { get; set; }

// ✅ 不需要加 [Required]
public int Id { get; set; }
```

### 2. 值型別搭配 `[Required]` 要用 nullable

```csharp
// ❌ decimal 是值型別，空白送出時型別轉換失敗，顯示英文系統錯誤
[Required(ErrorMessage = "價格為必填")]
public decimal Price { get; set; }

// ✅ 改成 decimal?，空白可轉為 null，[Required] 才能正常運作
[Required(ErrorMessage = "價格為必填")]
public decimal? Price { get; set; }
```

> 為什麼？當 `Price` 是 `decimal` 時，使用者送出空白 → ASP.NET Core 嘗試把空字串轉成 `decimal` → **型別轉換失敗**（還沒走到 `[Required]` 就先出錯）→ 顯示系統預設的英文錯誤訊息。改成 `decimal?` 後，空白可以轉成 `null` → 轉換成功 → `[Required]` 檢查到 `null` → 顯示妳自訂的中文 ErrorMessage。

### 3. `[StringLength]` 不能用在非字串型別

```csharp
// ❌ int 不是字串，[StringLength] 無作用
[StringLength(8)]
public int Id { get; set; }

// ✅ [StringLength] 只用在 string 屬性上
[StringLength(50)]
public string? Name { get; set; }
```

### 4. `[Display]` 的參數固定是 `Name`

```csharp
// ❌ 不能用屬性名稱當參數
[Display(Price = "價格")]
public decimal? Price { get; set; }

// ✅ 不管放在哪個屬性上，參數都是 Name
[Display(Name = "價格")]
public decimal? Price { get; set; }
```

### 5. 驗證失敗時要把 Model 帶回 View

```csharp
// ❌ 驗證失敗時沒帶 Model 回去，使用者之前填的資料會全部清空
return View();

// ✅ 把 Model 帶回去，使用者只需要修改有錯的欄位
return View(member);
```

### 6. Controller 收到資料後要先驗證再處理

```csharp
// ❌ 沒有檢查就直接存到資料庫，無效資料也會被存入
[HttpPost]
public IActionResult Register(Member member)
{
    _db.Members.Add(member);
    _db.SaveChanges();
    return RedirectToAction("Index");
}

// ✅ 先用 ModelState.IsValid 檢查，通過才處理
[HttpPost]
public IActionResult Register(Member member)
{
    if (ModelState.IsValid)
    {
        _db.Members.Add(member);
        _db.SaveChanges();
        return RedirectToAction("Index");
    }
    return View(member);
}
```

### 7. `[EmailAddress]` 和 `[Required]` 功能不同，可以同時使用

```csharp
// ❌ 只有 [EmailAddress]，不填也能通過（非必填）
[EmailAddress(ErrorMessage = "Email 格式錯誤")]
public string? Email { get; set; }

// ✅ 兩個都加，既必填又要檢查格式
[Required(ErrorMessage = "Email 為必填")]
[EmailAddress(ErrorMessage = "Email 格式錯誤")]
public string? Email { get; set; }
```

---

## Razor 頁面 vs Razor 檢視

在 Visual Studio 新增檔案時要注意選擇正確的範本：

| 範本 | 用途 | 產生的檔案 |
|---|---|---|
| **Razor 檢視（Razor View）** | MVC 的 View ✅ | `Create.cshtml`（單一檔案）|
| **Razor 頁面（Razor Page）** | Page-based 模式 ❌ | `Create.cshtml` + `Create.cshtml.cs`（兩個檔案）|

> ⚠️ Razor 頁面的 `.cshtml` 會包含 `@page` 指令，會繞過 Controller 直接處理請求，與 MVC 架構衝突。如果不小心選錯，刪掉重建即可，不影響專案其他部分。
>
> 辨認方式：如果一個 `.cshtml` 底下還跟著一個同名的 `.cshtml.cs`，就是 Razor Page。
