# ASP.NET Core MVC 筆記：Middleware（中介軟體）

---

## Middleware 是什麼？

Middleware（中介軟體）是 ASP.NET Core 處理 Request 和 Response 時，**一道道的關卡**。

Request 從瀏覽器進來後，會**依照 `Program.cs` 裡的排列順序**，一關一關通過每個 Middleware，最後到達 Controller。Controller 處理完後，Response 再**一層一層往回走**。

這整個流程就叫做 **Middleware Pipeline（中介軟體管線）**。

### 機場比喻

> 就像去機場搭飛機，從進航廈到登機，要經過好幾道關卡：
> 查護照 → 過安檢 → 驗登機證 → 登機
>
> 每一道關卡檢查一件事，通過了才讓你往下走。

---

## Pipeline 的執行流程

每個 Middleware 會被**經過兩次**：Request 進去一次，Response 回來一次。

```
Request 進來 →  [第一關 進] → [第二關 進] → [第三關 進] → Controller 處理
Response 回來 ← [第一關 出] ← [第二關 出] ← [第三關 出] ← Controller 回傳
```

像剝洋蔥一樣，一層一層包進去，再一層一層剝出來。

---

## `await next()` 的作用

```csharp
app.Use(async (context, next) =>
{
    // ① Request 進來時做的事
    Console.WriteLine("進來了");

    await next();  // ② 交給下一關，等它做完再回來

    // ③ Response 回來時做的事
    Console.WriteLine("出去了");
});
```

- `await next()` 的意思：**交給排在自己下面的那一關去處理，等所有後面的關卡都做完，再回來繼續執行下一行**
- `await` 本身就代表「等它做完，然後自動繼續往下」，不需要額外的判斷
- `next()` 去哪裡，不是寫在程式碼裡，而是由 **Program.cs 裡的排列順序**決定的

### 為什麼需要 `await next()` 這種設計？

因為有些事情需要**在 Request 進來時做一次，在 Response 回來時再做一次**。

例如計時器：

```csharp
app.Use(async (context, next) =>
{
    var 開始時間 = DateTime.Now;        // Request 進來，記錄開始時間
    await next();                        // 讓後面的 Middleware + Controller 去做事
    var 花了多久 = DateTime.Now - 開始時間;  // Response 回來，算出花了多少時間
    Console.WriteLine($"這次 Request 花了 {花了多久.TotalMilliseconds} 毫秒");
});
```

---

## `app.Use` vs `app.Run`

| | `app.Use` | `app.Run` |
|---|---|---|
| 有沒有 `next()` | 有，可以往下傳 | 沒有，到這裡就結束 |
| 用途 | 中間的關卡 | Pipeline 的**終點站** |

```csharp
// app.Run — 沒有 next 參數，Request 停在這裡
app.Run(async context =>
{
    Console.WriteLine("到此為止，不往下走了");
});
```

使用 `app.Run` 或在 `app.Use` 裡面不執行 `await next()`，後面的 Middleware 和 Controller 都**不會被執行**，瀏覽器會看到空白頁面。

---

## 擋下 Request

在 `app.Use` 裡面，可以用 `if` 判斷，**決定要不要放行**：

```csharp
app.Use(async (context, next) =>
{
    if (沒有登入)
    {
        return;         // 不執行 await next()，Request 停在這一關
    }

    await next();       // 有登入，放行
});
```

- 不執行 `await next()` → Request 就停在這一關，不會往下走到 Controller
- 執行 `await next()` → 放行，讓 Request 繼續往下

---

## 內建 Middleware

ASP.NET Core 提供很多內建的 Middleware，每一個負責一件事：

| Middleware | 做什麼 |
|---|---|
| `UseHttpsRedirection()` | 把 http 轉成 https |
| `UseRouting()` | 判斷 Request 要去哪個 Controller / Action |
| `UseAuthorization()` | 檢查有沒有權限存取頁面 |
| `MapStaticAssets()` | 處理靜態檔案（CSS、JS、圖片等） |

---

## 順序很重要

Middleware 的排列順序就是執行順序，**不能亂排**。

正確順序範例：

```csharp
app.UseHttpsRedirection();   // 1. 先確保 HTTPS
app.UseRouting();             // 2. 判斷要去哪裡（拿到登機證）
app.UseAuthorization();       // 3. 檢查權限（安檢）
app.MapStaticAssets();        // 4. 處理靜態檔案
app.MapControllerRoute(...);  // 5. 對應到 Controller
```

例如 `UseRouting` 一定要在 `UseAuthorization` 前面，因為：
- `UseAuthorization` 需要**先知道你要去哪個頁面**，才能判斷那個頁面需不需要權限
- 如果還沒經過 `UseRouting`，根本不知道你要去哪裡，怎麼檢查權限？

> 就像機場安檢人員要看登機證才知道你能不能過，如果你還沒拿到登機證（還沒 Routing），安檢人員沒辦法做事。

---

## 自訂 Middleware

可以用 `app.Use` 寫自己的 Middleware：

```csharp
// 印出每次 Request 的網址路徑
app.Use(async (context, next) =>
{
    Console.WriteLine(context.Request.Path);
    await next();
});
```

- `context.Request.Path` → 取得 Request 的網址路徑
- 瀏覽器開一個頁面，其實會產生**很多個 Request**（主頁面 + CSS + JS 等資源），因為 `_Layout.cshtml` 裡引用的每個檔案都會各發一個 Request

---

## 容易搞錯的地方

### ❌ 以為 `app.Use` 裡的程式碼是全部做完才往下一關

```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine("A");
    await next();           // ← 這裡會先暫停，跳去下一關
    Console.WriteLine("B"); // ← 等下一關做完才回來執行這行
});
```

### ✅ `await next()` 會先暫停，讓下一關去做事，做完再回來繼續

---

### ❌ 以為每個 Middleware 裡面都會執行所有內建 Middleware

每個 `app.UseXxx()` 都是**獨立的一關**，各自負責一件事。Request 經過一關就做那一關的事，然後往下走。

### ✅ 它們是排隊過關，不是巢狀包含

---

### ❌ 以為順序不重要

`UseRouting` 和 `UseAuthorization` 互換會出問題。

### ✅ Program.cs 裡的排列順序就是執行順序，順序有規定，不能隨意調換

---

### ❌ 忘記 `Console.WriteLine` 的輸出位置

`Console.WriteLine` 的訊息不在 Visual Studio 的「輸出」視窗，而是在**獨立的黑色 Console 視窗**（命令提示字元），可能藏在 Visual Studio 後面。

### ✅ 去工作列找黑色的 Console 視窗查看輸出
