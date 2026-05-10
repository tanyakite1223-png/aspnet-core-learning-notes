# 資料存取筆記:連線設定 — Connection String

## Connection String 是什麼？

Connection String（連線字串）告訴 EF Core：
- 要連到哪台伺服器
- 資料庫叫什麼名字
- 用什麼方式驗證

## 放在哪裡？

放在 `appsettings.json` 的 `ConnectionStrings` 區塊，與 `Logging`、`AllowedHosts` 平行：

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\MSSQLLocalDB;Database=ExpenseSystem;Trusted_Connection=True;"
  },
  "AllowedHosts": "*"
}
```

⚠️ JSON 字串裡的反斜線 `\` 必須寫成 `\\`，否則會出現「逸出字元無效」錯誤。

## 在 Program.cs 註冊 DbContext

```csharp
using ExpenseSystem.Data;
using Microsoft.EntityFrameworkCore;

// 放在 var app = builder.Build(); 之前
builder.Services.AddDbContext<ExpenseDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

- `AddDbContext<ExpenseDbContext>` — 把 DbContext 註冊進 DI（依賴注入）容器
- `GetConnectionString("DefaultConnection")` — 去 `appsettings.json` 的 `ConnectionStrings` 區塊找名叫 `DefaultConnection` 的連線字串
- `UseSqlServer(...)` — 指定使用 SQL Server（含 LocalDB）

⚠️ 記得加正確的 `using`，否則會出現「找不到型別」錯誤。
