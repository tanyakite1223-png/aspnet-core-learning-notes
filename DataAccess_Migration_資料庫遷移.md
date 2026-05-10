# 資料存取筆記:Migration — 建立與更新資料庫結構

---

## Migration（資料庫遷移）是什麼？

Model（模型）寫好後，資料庫裡並不會自動出現對應的資料表。Migration 是 EF Core 用來**把 C# Model 的結構同步到資料庫**的機制。

---

## Code First vs Database First

| 方向 | 做法 | 適合情境 |
|------|------|----------|
| **Code First** | 先寫 C# Model → 用 Migration 建資料庫 | 新專案，從零開始 |
| **Database First** | 先有資料庫 → 用 Scaffold 產生 Model | 接手舊系統，DB 已存在 |

本主題使用 **Code First**。

---

## 為什麼分兩步？

Migration 流程分兩個指令，原因是：若只有「直接建資料庫」一個指令，每次改 Model 都必須砍掉重建，**資料庫裡的資料會全部消失**。

分兩步的設計讓每次只套用**新的變更**，不動已有的資料。

```
你的 Model (C#)
     ↓
dotnet ef migrations add {名稱}   ← 產生「變更描述檔」，不動資料庫
     ↓
dotnet ef database update         ← 把描述檔套用到資料庫
     ↓
資料庫結構更新，原有資料保留
```

---

## 指令說明

### 第一步：產生 Migration 描述檔

```powershell
dotnet ef migrations add InitialCreate
```

- `InitialCreate` 是自訂名稱，描述這次變更的內容
- 名稱用英文，例如 `FixAmountPrecision`、`AddUserTable`
- 執行後會在專案內產生 `Migrations\` 資料夾

### 第二步：套用到資料庫

```powershell
dotnet ef database update
```

- 把尚未套用的 Migration 描述檔執行到資料庫
- 若資料庫不存在，會自動建立

---

## Migration 檔案結構

`Migrations\` 資料夾內每個檔案都有**時間戳記**前綴，例如：

```
Migrations/
  20260507150539_InitialCreate.cs
  20260508_FixAmountPrecision.cs
  ExpenseDbContextModelSnapshot.cs
```

每個 Migration 檔案包含兩個方法：

| 方法 | 說明 |
|------|------|
| `Up()` | 套用這次變更（例如建立資料表、新增欄位） |
| `Down()` | 回滾（rollback）這次變更（例如刪除資料表） |

> 📌 Migration 檔案是資料庫結構的**版本歷史**，必須納入 git 版本控制，不能 gitignore。

---

## `__EFMigrationsHistory` 資料表

EF Core 在資料庫中自動建立這個資料表，用來追蹤「哪些 Migration 已經執行過」。

每次 `database update` 成功後，會在此表新增一筆記錄。下次執行時，EF Core 會跳過已執行的，只套用新的。

---

## decimal 欄位精確度設定

`decimal` 型別若未指定精確度，EF Core 會產生 warning。金額欄位慣例用 `decimal(18, 2)`（最多 18 位數，小數點後 2 位）。

用 Data Annotation 設定方式：

```csharp
using System.ComponentModel.DataAnnotations.Schema;

[Column(TypeName = "decimal(18,2)")]
public decimal Amount { get; set; }
```

設定後需重新跑一次 Migration：

```powershell
dotnet ef migrations add FixAmountPrecision
dotnet ef database update
```

---

## 本次實作的 Expense Model

```csharp
// Models/Expense.cs
public class Expense
{
    public int ExpenseId { get; set; }

    [Required(ErrorMessage = "Title必填")]
    public string Title { get; set; } = string.Empty;

    [Column(TypeName = "decimal(18,2)")]
    public decimal Amount { get; set; }

    public DateTime ExpenseDate { get; set; }

    public string? Description { get; set; }
}
```

---

## 本次實作的 ExpenseDbContext

```csharp
// Data/ExpenseDbContext.cs
public class ExpenseDbContext : DbContext
{
    public ExpenseDbContext(DbContextOptions<ExpenseDbContext> options) : base(options) { }

    public DbSet<Expense> Expenses { get; set; }
}
```

---

## 本次實作的 Program.cs（新增部分）

```csharp
builder.Services.AddDbContext<ExpenseDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

---

## 排錯記錄：EF Core 套件版本衝突

本次實作遇到 `Microsoft.EntityFrameworkCore.Design` 版本停在 8.0.0，與其他套件的 10.0.7 不一致，導致 `dotnet ef` 指令失敗。

解法：

```powershell
# 升級 Design 套件
dotnet add package Microsoft.EntityFrameworkCore.Design --version 10.0.7

# 升級全域 dotnet-ef 工具
dotnet tool update --global dotnet-ef
```

診斷指令（查看所有套件含相依版本）：

```powershell
dotnet list package --include-transitive
```

---

## 重點整理

- Migration **不是**直接建資料庫，而是先產生描述檔，再套用
- 每次修改 Model 後，都要重跑 `migrations add` + `database update`
- Migration 檔案要 commit 進 git
- `__EFMigrationsHistory` 是 EF Core 自己管理的，不要手動改它
