# ASP.NET Core MVC 筆記：Action 方法與回傳型別（IActionResult、ViewResult）

## IActionResult 是什麼？

- `IActionResult` 是一個 **Interface（介面）**，由 ASP.NET Core 框架定義
- 所有 Action 方法的回傳型別都建議用 `IActionResult`
- 各種具體的 Result 類別都**實作了 `IActionResult` 介面**
- 這就是 Polymorphism（多型）的應用：宣告介面型別，實際回傳具體類別

---

## 四種常用回傳方式

| 方法 | 回傳型別 | 用途 | 需要 .cshtml？ |
|------|----------|------|---------------|
| `View()` | ViewResult | 回傳一個畫面（View） | ✅ 需要 |
| `RedirectToAction("Action名稱")` | RedirectToActionResult | 重新導向到另一個 Action | ❌ 不需要 |
| `Content("文字")` | ContentResult | 回傳純文字給瀏覽器 | ❌ 不需要 |
| `Json(物件)` | JsonResult | 回傳 JSON 格式資料 | ❌ 不需要 |

### 重點：只有 `return View()` 需要對應的 `.cshtml` 檔案，其他三種都不需要。

---

## Redirect 的兩種常見用法

```csharp
// 導向外部網址
return Redirect("https://www.google.com");

// 導向同一個專案裡的另一個 Action（最常用）
return RedirectToAction("Privacy");
```

### ⚠️ 注意：直接呼叫方法 vs 重新導向

```csharp
// ❌ 直接呼叫方法（瀏覽器網址不會變）
return Privacy();

// ✅ 重新導向（瀏覽器會發新請求，網址會改變）
return RedirectToAction("Privacy");
```

---

## 為什麼建議用 IActionResult？

可以寫具體型別（例如 `public ContentResult Greeting()`），也能正常執行。

但如果同一個方法裡面可能回傳不同種類的結果，就**必須**用 `IActionResult`：

```csharp
public IActionResult CheckAge()
{
    int age = 16;

    if (age >= 18)
    {
        return Content("歡迎進入！");  // ContentResult
    }
    else
    {
        return RedirectToAction("Privacy");  // RedirectToActionResult
    }
}
```

如果把回傳型別寫成 `ContentResult`，那 `return RedirectToAction(...)` 就會報錯。

---

## Console.WriteLine vs return Content

- `Console.WriteLine()` → 寫到**主控台**（開發者自己看，在 Visual Studio 輸出視窗）
- `return Content()` → 回傳給**瀏覽器**（使用者看到的網頁內容）

在 ASP.NET Core 中，瀏覽器只看得到 `return` 回來的東西。

---

## 編譯錯誤筆記

### CS0161：不是所有程式碼路徑都有傳回值

```csharp
// ❌ 編譯器認為可能沒有 return
if (age >= 18)
{
    return Content("歡迎進入！");
}
else if (age < 18)  // 編譯器不確定這是否涵蓋所有情況
{
    return RedirectToAction("Privacy");
}

// ✅ 用 else 就能涵蓋所有情況
if (age >= 18)
{
    return Content("歡迎進入！");
}
else
{
    return RedirectToAction("Privacy");
}
```

---

## 補充：JSON 中文顯示

- `Json()` 回傳中文時，預設會轉成 Unicode 編碼（例如 `\u53F0\u5317`）
- 前端（瀏覽器 / JavaScript）會自動還原成中文，資料本身沒有問題
- 若需要 JSON 直接顯示中文，可在 `Program.cs` 加設定（進階，之後再學）

---

## 測試網址格式

```
https://localhost:7142/{Controller名稱}/{Action名稱}
```

例如：
- `https://localhost:7142/Home/Greeting` → 執行 HomeController 的 Greeting()
- `https://localhost:7142/Home/UserInfo` → 執行 HomeController 的 UserInfo()
