# 資料存取筆記:EF Core 概念 — ORM 是什麼、DbContext

## ORM 是什麼？

ORM（Object-Relational Mapping，物件關聯對應）是一種工具，
自動把資料庫的表格結構和 C# 的 Class 對應起來。

有了 ORM，你不需要自己拼 SQL 字串，改用 C# 物件和 LINQ 操作資料。

### 資料庫 vs C# 的對應關係

| 資料庫世界 | C# 世界 |
|-----------|---------|
| 一張表（Table） | 一個 Class |
| 一個欄位（Column） | 一個 Property |
| 一筆資料列（Row） | 一個物件（Object） |
| 整張表的集合 | `DbSet<T>` |

### ORM 的好處
- 防止 SQL Injection（不用自己拼 SQL 字串）
- 每張表對應一個 Class，結構清楚好維護
- 同一份程式碼，換設定就能切換不同資料庫（SQL Server / PostgreSQL 等）

---

## EF Core 是什麼？

EF Core（Entity Framework Core）是 .NET 世界的 ORM 工具。

不是內建的，需要另外安裝 NuGet 套件：
- `Microsoft.EntityFrameworkCore.SqlServer`（連接 SQL Server 用）
- `Microsoft.EntityFrameworkCore.Tools`（Migration 等工具指令用）

安裝指令（在專案目錄下執行）：
```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

---

## DbContext（資料庫上下文）

DbContext 是 EF Core 的核心類別，是程式跟資料庫之間的「管理員」。

負責：
- 維持資料庫連線
- 追蹤資料的變更
- 把 LINQ 查詢翻譯成 SQL 送出去

### 建立方式

繼承 `DbContext`，建立自訂的 Context 類別：

```csharp
// Data/ExpenseDbContext.cs
using Microsoft.EntityFrameworkCore;
using ExpenseSystem.Models;

namespace ExpenseSystem.Data
{
    public class ExpenseDbContext : DbContext
    {
        public ExpenseDbContext(DbContextOptions<ExpenseDbContext> options) 
            : base(options)
        {
        }

        public DbSet<Expense> Expenses { get; set; }
    }
}
```

- **建構子**：接收 `DbContextOptions`，裡面包含連線資訊（伺服器、資料庫名稱、帳號密碼等），細節在「連線設定」主題說明
- **`DbSet<Expense> Expenses`**：代表資料庫裡的 `Expenses` 這張表

---

## DbSet\<T\>

`DbSet<T>` 代表資料庫中的一張表。
你透過它對該表進行查詢、新增、修改、刪除。

---

## 本次實作：Expense Model

```csharp
// Models/Expense.cs
using System.ComponentModel.DataAnnotations;

namespace ExpenseSystem.Models
{
    public class Expense
    {
        public int ExpenseId { get; set; }

        [Required(ErrorMessage = "Title必填")]
        public string Title { get; set; } = string.Empty;

        public decimal Amount { get; set; }
        public DateTime ExpenseDate { get; set; }
        public string? Description { get; set; }
    }
}
```

### Property 說明

| Property | 型別 | 說明 |
|----------|------|------|
| `ExpenseId` | `int` | Primary Key |
| `Title` | `string` | 報銷標題（必填） |
| `Amount` | `decimal` | 金額 |
| `ExpenseDate` | `DateTime` | 日期 |
| `Description` | `string?` | 備註（可為 null） |

### `[Required]` vs `= string.Empty`

| | 作用時機 | 用途 |
|--|---------|------|
| `= string.Empty` | 編譯時期 | 給初始值，避免編譯器警告「string 可能為 null」 |
| `[Required]` | 執行時期 | 表單送出時驗證使用者是否有填寫此欄位 |

兩者各司其職，可以同時使用。

---

## 📌 補充參考（本次未實作）

- Connection String 設定 → 下一個主題「連線設定」
- 在 `Program.cs` 註冊 DbContext（`builder.Services.AddDbContext`）→ 下一個主題
- Migration（建立資料庫結構）→ 之後的主題