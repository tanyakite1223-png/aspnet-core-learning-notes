# 資料存取筆記：Scaffold-DbContext（反向工程，從資料庫產生 Model）

## 兩種方向：Code First vs Database First

| 方向 | 做法 | 適用情境 |
|------|------|----------|
| **Code First** | 先寫 C# Model → 用 Migration 產生資料庫 | 全新專案，資料庫還不存在 |
| **Database First**（反向工程） | 資料庫已存在 → 用 Scaffold 產生 C# Model | 接手舊系統、公司已有資料庫 |

---

## Scaffold-DbContext 指令

### 基本語法

```
dotnet ef dbcontext scaffold "連線字串" Microsoft.EntityFrameworkCore.SqlServer
```

- 第一個參數：連線字串（Connection String）
- 第二個參數：資料庫 Provider（SQL Server 就是 `Microsoft.EntityFrameworkCore.SqlServer`）

### 常用參數

| 參數 | 用途 | 範例 |
|------|------|------|
| `--output-dir` | 指定 Model 檔案輸出目錄 | `--output-dir Models` |
| `--context-dir` | 指定 DbContext 檔案輸出目錄 | `--context-dir Data` |
| `--table` | 只產生指定的表（可重複使用） | `--table Categories --table Products` |

### 實際執行範例

```
dotnet ef dbcontext scaffold "Server=(localdb)\MSSQLLocalDB;Database=BookstoreDemo;Trusted_Connection=True;TrustServerCertificate=True" Microsoft.EntityFrameworkCore.SqlServer --output-dir Models --context-dir Data
```

### 必要的 NuGet 套件

- `Microsoft.EntityFrameworkCore.SqlServer` — SQL Server Provider
- `Microsoft.EntityFrameworkCore.Design` — 提供 `dotnet ef` 設計階段工具支援

---

## 自動產生的程式碼特徵

### Model 類別

Scaffold 根據資料表結構自動產生對應的 C# 類別：

- 欄位名稱、型別自動對應（例如 `nvarchar` → `string`、`int` → `int`）
- 有 FK constraint 的欄位會產生**導覽屬性（Navigation Property）**

### 導覽屬性（Navigation Property）

以 Categories 和 Products 的一對多關聯為例：

```csharp
// Category 端（一的那端）
public virtual ICollection<Product> Products { get; set; }

// Product 端（多的那端）
public int? CategoryId { get; set; }
public virtual Category? Category { get; set; }
```

- `ICollection<Product>` — 一個 Category 底下有多個 Products
- `Category?` — 一個 Product 屬於哪個 Category

**重要**：Scaffold 只能根據資料庫裡**實際存在的 FK constraint** 來產生導覽屬性。如果欄位取名叫 CategoryId 但沒有設 FK，Scaffold 不會知道它們有關聯。

### DbContext — OnModelCreating

Scaffold 預設使用 **Fluent API** 在 `OnModelCreating` 方法中描述資料表結構：

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Product>(entity =>
    {
        entity.HasKey(e => e.ProductId);
        entity.Property(e => e.ProductName).HasMaxLength(100);
        // FK 關聯設定
        entity.HasOne(d => d.Category)
              .WithMany(p => p.Products)
              .HasForeignKey(d => d.CategoryId);
    });
}
```

這跟手寫 Model 時用 Data Annotation（如 `[Required]`、`[MaxLength]`）是不同的寫法，但效果相同。

### 連線字串警告

Scaffold 會在 `OnConfiguring` 中直接寫死連線字串，並產生警告註解：

```csharp
// 自動產生，但不安全 — 應該移到 appsettings.json
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder.UseSqlServer("Server=...");
```

在實際專案中，應該把連線字串移到 `appsettings.json`，用 DI 注入 DbContext。

---

## 實作練習紀錄

- 在 `D:\Workshops\Amber\Sandbox\ScaffoldDemo\` 建立 Console 專案
- 對 BookstoreDemo 資料庫執行 Scaffold，產生：
  - `Data/BookstoreDemoContext.cs` — DbContext
  - `Models/Book.cs` — Book 實體
  - `Models/Category.cs` — Category 實體（含 `ICollection<Product>` 導覽屬性）
  - `Models/Product.cs` — Product 實體（含 `Category?` 導覽屬性和可為 null 的 FK）
- 觀察到 Book 沒有導覽屬性 — 因為當初建表時 CategoryId 未設 FK constraint

---

## 重點整理

1. **Scaffold-DbContext = 從資料庫反向產生 Model**，適合接手已有資料庫的專案
2. 產生的程式碼是「起點」，通常還需要調整（移連線字串、調整命名等）
3. 用 `--table` 參數可以只產生需要的表，不用全部產生
4. 導覽屬性的產生**依賴資料庫的 FK constraint**，沒設 FK 就不會產生
5. Scaffold 預設用 Fluent API，而非 Data Annotation
