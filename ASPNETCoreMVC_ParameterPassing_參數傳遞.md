# ASP.NET Core MVC 筆記：參數傳遞 — Route 參數、Query String

## 一、概念總覽

Controller 的 Action 方法可以透過**參數**接收使用者傳來的資料。主要有兩種方式：

- **Route 參數**：值放在 URL 路徑裡面
- **Query String（查詢字串）**：值放在 `?` 後面

---

## 二、Route 參數

### 路由樣板中的定義

`Program.cs` 預設的路由樣板：

```
{controller=Home}/{action=Index}/{id?}
```

- `{id?}` 就是 Route 參數
- `?` 代表**選填**（optional），可以不給

### 使用方式

URL：`/Product/Details/5`

Action 方法：

```csharp
public IActionResult Details(int? id)
{
    if (id is null)
    {
        return Content("請提供商品編號");
    }
    return Content($"商品編號：{id}");
}
```

### 重點

- 參數名稱必須跟路由樣板中的名稱一致（`id`）。這個 `id` 不是系統內定的，是在 `Program.cs` 路由樣板的 `{id?}` 自己取的名字，只是大家習慣用 `id`，算是一種慣例（convention）
- 使用 `int?`（Nullable<int>）可以在沒給值時得到 `null`，而不是預設值 `0`
- 如果用 `int`（非 Nullable），沒給值時會得到預設值 `0`

---

## 三、Query String（查詢字串）

### 格式

在 URL 後面加 `?`，用 `key=value` 的格式傳值：

```
/Product/Search?keyword=手機
```

### 多個參數

用 `&` 串接多個參數：

```
/Product/Search?keyword=手機&page=2&sort=price
```

### 使用方式

```csharp
public IActionResult Search(string keyword, int page, string sort)
{
    return Content($"搜尋：{keyword}，頁碼：{page}，排序：{sort}");
}
```

### 重點

- Action 方法的**參數名稱**要和 Query String 的 **key 名稱**一致
- ASP.NET Core 的 Model Binding 會自動配對

---

## 四、同一個 Action 可以同時接受兩種方式

`Details(int? id)` 可以透過兩種方式接收值：

- Route 參數：`/Product/Details/8`
- Query String：`/Product/Details?id=8`

兩種都能正確把值傳進 `id` 參數。

---

## 五、Model Binding 優先順序

當 Route 參數和 Query String 同時提供同一個參數的值時，不是按 URL 文字順序決定的，而是有固定的優先順序：

1. **Route 參數**（最優先）
2. **Query String**
3. **Form 表單資料**（之後學表單時會用到）

範例：`/Product/Details/5?id=8` → `id` 會拿到 `5`（Route 參數贏）

---

## 六、Nullable 型別的應用

| 型別 | 沒給值時 | 能否區分「沒給」和「給 0」 |
|------|----------|--------------------------|
| `int` | 得到 `0` | 不能 |
| `int?` | 得到 `null` | 可以 |

建議：當參數是**選填**時，使用 `int?` 搭配 `null` 判斷，更能區分使用者的意圖。
