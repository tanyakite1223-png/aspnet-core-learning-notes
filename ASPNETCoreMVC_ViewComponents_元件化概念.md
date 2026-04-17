# ASP.NET Core MVC 筆記：View Components（元件化概念）

---

## View Component 是什麼？

View Component 是一種「**迷你的 Controller + View 組合**」，可以嵌入任何頁面。

它解決了 Partial View 的限制：Partial View 只能顯示外部傳入的資料，無法自己取得資料。View Component 則可以**自己執行邏輯、自己取得資料**，不依賴外層的 Controller。

---

## Partial View vs View Component

| | Partial View | View Component |
|---|---|---|
| 自己取資料 | ❌ 不行 | ✅ 可以 |
| 有自己的邏輯 | ❌ 不行 | ✅ 可以 |
| 適合用在 | 純版面拆分、重複 HTML | 獨立的功能型元件 |

---

## 建立 View Component 的三個步驟

### 步驟一：建立 View Component 類別

在專案中新增 `ViewComponents` 資料夾，在裡面建立類別檔案。

類別必須繼承 `ViewComponent`，並實作 `Invoke()` 方法。

```csharp
using Microsoft.AspNetCore.Mvc;

public class AnnouncementViewComponent : ViewComponent
{
    public IViewComponentResult Invoke()
    {
        var count = 5; // 可以是從 DB 取得的資料
        return View(count);
    }
}
```

### 步驟二：建立對應的 View 檔案

路徑規則（固定）：
```
Views/Shared/Components/{元件名稱}/Default.cshtml
```

- 資料夾名稱 = 類別名稱去掉 `ViewComponent`
- 檔案名稱固定為 `Default.cshtml`

範例：`AnnouncementViewComponent` → 資料夾名稱為 `Announcement`

```
Views/Shared/Components/Announcement/Default.cshtml
```

View 檔案內容範例：

```html
@model int

<div>
    📢 最新消息：共 @Model 則
</div>
```

### 步驟三：在 Razor View 中呼叫

在任何想顯示元件的 `.cshtml` 頁面加上：

```html
@await Component.InvokeAsync("Announcement")
```

引號內填入**元件名稱**（去掉 `ViewComponent`）。

#### `@await Component.InvokeAsync` 是什麼意思？

- **Invoke** = 呼叫、執行
- **Async** = Asynchronous（非同步）的縮寫
- **await** = 等它執行完畢

「非同步」的概念：當程式需要等待某件事（例如查資料庫），非同步的方式讓程式在等待期間可以去做其他事，不會卡住整個網站。就像去早餐店點完餐，不用站在櫃檯發呆，可以先去找位子坐。

> 💡 看到方法名稱結尾是 `Async`，前面通常要加 `await`。

---

## 重要優點

同一個 View Component 可以放在任何頁面，**不需要修改任何 Controller**。元件自己負責取資料，與外層完全獨立。

---

## 常見錯誤

### ❌ View 檔案路徑或檔名錯誤

```
// 錯誤：檔名用元件名稱
Views/Shared/Components/Announcement/Announcement.cshtml

// 錯誤：少了元件名稱資料夾
Views/Shared/Components/Default.cshtml
```

```
// ✅ 正確
Views/Shared/Components/Announcement/Default.cshtml
```

### ❌ 類別名稱資料夾沒有去掉 `ViewComponent`

```
// 錯誤
Views/Shared/Components/AnnouncementViewComponent/Default.cshtml

// ✅ 正確
Views/Shared/Components/Announcement/Default.cshtml
```
