# 資料存取筆記：LINQ 與 EF Core 的結合使用

## IQueryable vs IEnumerable — 最核心的差異

| 項目 | IQueryable\<T\> | IEnumerable\<T\> |
|------|----------------|-------------------|
| 使用場景 | EF Core 的 `DbSet<T>` | `List<T>` 等記憶體集合 |
| LINQ 執行位置 | 翻譯成 **SQL**，在**資料庫端**執行 | 在 **C# 記憶體**中執行 |
| 效能影響 | 只回傳符合條件的資料 | 資料必須先全部載入記憶體 |

重點：`context.Expenses` 回傳的是 `IQueryable<Expense>`，所以對它寫 LINQ，EF Core 會翻譯成 SQL。

## 延遲執行（Deferred Execution）

對 `IQueryable` 寫 `.Where()`、`.OrderBy()`、`.Select()` 時，**不會立刻查資料庫**，只是把條件記下來。要等到「需要具體結果」的動作出現，才會真正發出 SQL 查詢。

### 不觸發查詢的方法（只記錄條件）

- `.Where()` — 過濾條件
- `.OrderBy()` / `.OrderByDescending()` — 排序
- `.Select()` — 選擇欄位

### 會觸發查詢的方法（立即執行）

- `.ToList()` — 轉成 List，取回所有符合條件的資料
- `.FirstOrDefault()` — 取第一筆（SQL: `SELECT TOP(1)`）
- `.Count()` — 計算筆數
- `.Sum()` — 加總
- `.Any()` — 是否有資料（回傳 bool）
- `foreach` — 迭代時觸發

## .ToList() 的位置很重要

```csharp
// ❌ 版本 A：先 ToList() 再 Where() — 全撈再過濾
var result = context.Expenses.ToList().Where(e => e.Amount > 200);
// SQL: SELECT * FROM Expenses（全部撈回記憶體，再用 C# 過濾）

// ✅ 版本 B：先 Where() 再 ToList() — 資料庫過濾
var result = context.Expenses.Where(e => e.Amount > 200).ToList();
// SQL: SELECT ... FROM Expenses WHERE Amount > 200（只回傳符合條件的）
```

原則：**`.ToList()` 放在 LINQ 鏈的最後面**，讓所有條件都先組好，EF Core 才能把它們全部翻成 SQL。

## EF Core 的 SQL 翻譯對照

### Where + Select + OrderBy

```csharp
context.Expenses
    .Where(e => e.ExpenseDate >= new DateTime(2026, 5, 5))
    .OrderByDescending(e => e.Amount)
    .Select(e => new { e.Title, e.Amount, e.ExpenseDate });
```

翻譯成：

```sql
SELECT [e].[Title], [e].[Amount], [e].[ExpenseDate]
FROM [Expenses] AS [e]
WHERE [e].[ExpenseDate] >= '2026-05-05T00:00:00.0000000'
ORDER BY [e].[Amount] DESC
```

### Sum（聚合函數）

```csharp
context.Expenses.Sum(e => e.Amount);
```

翻譯成：

```sql
SELECT COALESCE(SUM([e].[Amount]), 0.0) FROM [Expenses] AS [e]
```

`COALESCE` 是 EF Core 自動加的保護，資料表為空時 `SUM` 回傳 `NULL`，`COALESCE` 將其轉為 `0.0`。

### OrderByDescending + FirstOrDefault

```csharp
context.Expenses
    .OrderByDescending(e => e.Amount)
    .Select(e => new { e.Title, e.Amount })
    .FirstOrDefault();
```

翻譯成：

```sql
SELECT TOP(1) [e].[Title], [e].[Amount]
FROM [Expenses] AS [e]
ORDER BY [e].[Amount] DESC
```

### Contains（字串搜尋）

```csharp
context.Expenses.Where(e => e.Description.Contains("客戶"));
```

翻譯成：

```sql
WHERE [e].[Description] LIKE N'%客戶%'
```

類似的字串方法：
- `.StartsWith("拜訪")` → `LIKE N'拜訪%'`
- `.EndsWith("停車")` → `LIKE N'%停車'`

## 日期的寫法

C# 中建立日期要用 `DateTime` 建構子：

```csharp
new DateTime(2026, 5, 5)   // ✅ 正確：年, 月, 日 用逗號分隔
// 2026/05/05              // ❌ 錯誤：C# 會當成除法運算
// '2026/05/05'            // ❌ 錯誤：單引號是 char，不能表示日期
```

## 查看 EF Core 產生的 SQL

在 Development 環境下，ASP.NET Core 預設會在 Console 視窗輸出 EF Core 產生的 SQL。如果需要手動啟用：

```csharp
builder.Services.AddDbContext<ExpenseDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection"))
           .LogTo(Console.WriteLine, LogLevel.Information));
```

## 使用 FirstOrDefault() 的 null 檢查

`FirstOrDefault()` 找不到資料時回傳 `null`，使用前應加檢查：

```csharp
var item = context.Expenses
    .OrderByDescending(e => e.Amount)
    .FirstOrDefault();

if (item != null)
{
    Console.WriteLine($"項目：{item.Title}  金額：{item.Amount}元");
}
```
