# ASP.NET Core MVC 筆記：Routing 設定 — 慣例路由、屬性路由（Attribute Routing）

## 一、什麼是 Routing（路由）？

Routing 的工作就是 **把網址（URL）對應到 Controller 的某個 Action 方法**。

當使用者在瀏覽器輸入 `https://localhost:7142/Home/Index`，ASP.NET Core 必須知道：「這個網址要呼叫哪一個 Controller 的哪一個 Action？」這就是 Routing 在做的事。

ASP.NET Core 提供兩種 Routing 方式：

1. **慣例路由（Conventional Routing）** — 在 `Program.cs` 統一設定
2. **屬性路由（Attribute Routing）** — 直接寫在 Controller / Action 上面

---

## 二、慣例路由（Conventional Routing）

### 預設設定

在 `Program.cs` 裡會看到這一段：

```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}")
    .WithStaticAssets();
```

關鍵在 `pattern` 這個樣板：

```
{controller=Home}/{action=Index}/{id?}
```

### Pattern 的三個關鍵符號

| 符號 | 意思 | 範例 |
|---|---|---|
| `{xxx}` | 路由參數位置 | `{controller}` |
| `{xxx=值}` | 預設值（網址沒提供時用） | `{controller=Home}` |
| `{xxx?}` | 可選參數（可以不帶） | `{id?}` |

### 對應範例

| 網址 | Controller | Action | id |
|---|---|---|---|
| `/Product/Detail/5` | Product | Detail | 5 |
| `/Product/List` | Product | List | null（沒帶值） |
| `/` | Home（預設值） | Index（預設值） | null |

### 慣例路由的限制

慣例路由很方便，但**格式是固定的** — 永遠都是 `Controller/Action/id` 三段式結構。

如果想要做這種網址，慣例路由就做不到了：
- `/products/2024/laptop`
- `/blog/2025/04/my-first-post`
- `/users/amber/posts`

這時候就要用 **屬性路由**。

---

## 三、屬性路由（Attribute Routing）

### 基本概念

屬性路由是用 `[Route("...")]` 屬性（Attribute）**直接標註在 Controller class 或 Action 方法上面**，告訴系統「這個 Controller / Action 對應到哪個網址」。

### `[Route("XXX")]` 寫在哪裡會怎樣？

| 寫在哪 | 代表什麼 |
|---|---|
| Controller class 上面 | 整個 Controller 的網址前綴（prefix） |
| Action 方法上面 | 該 Action 自己的網址 |

兩個會被 ASP.NET Core **串接**起來。

### 範例

```csharp
[Route("blog")]                            // ← Controller 上：前綴 blog
public class BlogController : Controller
{
    [Route("")]                            // ← Action 上：空字串
    public IActionResult Index()           // 完整網址：/blog
    {
        return Content("Index()");
    }

    [Route("post/{id}")]                   // ← Action 上：post/{id}
    public IActionResult Post(int id)      // 完整網址：/blog/post/{id}
    {
        return Content("Post()");
    }

    [Route("{year}/{month}")]              // ← Action 上：{year}/{month}
    public IActionResult Archive(int year, int month)  // 完整網址：/blog/{year}/{month}
    {
        return Content("Archive()");
    }
}
```

### 對應網址

| 網址 | 對應 Action | 參數值 |
|---|---|---|
| `/blog` | `Index()` | — |
| `/blog/post/123` | `Post(int id)` | id = 123 |
| `/blog/2025/04` | `Archive(int year, int month)` | year = 2025, month = 4 |

---

## 四、字面值 vs 參數位置（重要觀念）

這是很容易搞混的地方：

| 寫法 | 是什麼 | 行為 |
|---|---|---|
| `post` | **字面值** | 網址裡要真的有 `post` 這個字 |
| `{id}` | **參數位置** | 接收網址這個位置的值，傳給方法的 `id` 參數 |

### 範例對照

```csharp
[Route("post/{id}")]
public IActionResult Post(int id)
```

對應網址 `/blog/post/123`：
- `post` → 字面值，網址裡要有「post」這個字
- `{id}` → 參數位置，接收 `123`，傳給方法的 `id`

---

## 五、為什麼字面值很重要？— 避免路由衝突

如果 `Post` 和 `Archive` 都不加字面值：

```csharp
[Route("{id}")]              // /blog/123
public IActionResult Post(int id)

[Route("{year}/{month}")]    // /blog/2025/04
public IActionResult Archive(int year, int month)
```

雖然網址段數不同（一段 vs 兩段），現在不會衝突，但**這種設計很危險**：
- 一旦未來新增其他 Action，網址結構可能會打架
- 路由規則靠「猜長度」來區分，閱讀時不直觀

所以加上 `post/` 這個 **識別標記** 是好習慣：

```csharp
[Route("post/{id}")]         // /blog/post/123 — 一看就知道是 Post
[Route("{year}/{month}")]    // /blog/2025/04 — 一看就知道是 Archive
```

---

## 六、慣例路由 vs 屬性路由 比較

| | 慣例路由（Conventional） | 屬性路由（Attribute） |
|---|---|---|
| **設定位置** | `Program.cs` 統一設定 | 直接寫在 Controller / Action 上面 |
| **格式** | 固定（`Controller/Action/id`） | 自由設計 |
| **適合場景** | 標準的 CRUD 頁面 | 需要客製化網址（RESTful API、SEO 友善網址） |
| **可讀性** | 規則統一，容易管理 | 規則分散在各 Action，但每個 Action 自己看就懂 |

實務上 **兩種可以同時使用** — 大部分頁面用慣例路由，少數需要特殊網址的就用屬性路由覆蓋。

---

## 七、本次練習完整程式碼

```csharp
[Route("blog")]
public class BlogController : Controller
{
    [Route("")]
    public IActionResult Index()
    {
        return Content("Index()");
    }

    [Route("post/{id}")]
    public IActionResult Post(int id)
    {
        return Content("Post()");
    }

    [Route("{year}/{month}")]
    public IActionResult Archive(int year, int month)
    {
        return Content("Archive()");
    }
}
```

### 測試結果

| 測試網址 | 顯示內容 |
|---|---|
| `https://localhost:7142/blog` | `Index()` ✅ |
| `https://localhost:7142/blog/post/123` | `Post()` ✅ |
| `https://localhost:7142/blog/2025/04` | `Archive()` ✅ |

---

## 八、學習重點摘要

1. **Routing 的目的**：把網址對應到 Controller 的某個 Action
2. **慣例路由**：在 `Program.cs` 統一設定，格式固定，適合標準頁面
3. **屬性路由**：用 `[Route("...")]` 寫在 Controller / Action 上，網址完全自由設計
4. **`{ }` 大括號是參數位置**，沒有大括號的就是字面值
5. **Controller 上的 `[Route]`** 是整個 Controller 的網址前綴
6. **Action 上的 `[Route]`** 是該 Action 的網址
7. **兩個會串接起來**，組成完整網址
8. **加字面值是好習慣**，可以避免路由衝突、提升可讀性

---

## 九、過程中犯的錯與學到的觀念

1. **`{Index}` 加大括號變成參數位置** — 大括號不是用來「框住」名字的，是用來宣告「這裡是參數」
2. **`[Route("blog/")]` 多了斜線** — Controller 已經有 `blog`，Action 上若再加 `blog/` 會變成 `/blog/blog/`
3. **字面值 vs 占位符的混淆** — 任務裡的 `/blog/post/123` 中 `post` 是字面值（要照打），不是要替換的東西
