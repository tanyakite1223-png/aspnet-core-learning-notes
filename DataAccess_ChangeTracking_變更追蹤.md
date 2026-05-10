# 資料存取筆記：變更追蹤（Change Tracking）概念

## 什麼是 Change Tracking（變更追蹤）？

EF Core 透過 DbContext 自動追蹤物件狀態的機制。

- 物件從資料庫撈出來時，DbContext 會**記住原始值（Snapshot）**
- 呼叫 `SaveChanges()` 時，EF Core 比對「現在的值」vs「原始值」
- **有差異** → 產生對應的 SQL（`INSERT`、`UPDATE`、`DELETE`）
- **沒差異** → 不做事

這表示 EF Core 不會笨笨地把所有撈出來的資料都 UPDATE 一遍，只會針對**真正有變動的物件**產生 SQL，提升效率。

---

## 5 種 EntityState（實體狀態）

| 狀態 | 說明 | 對應的 SQL |
|------|------|-----------|
| **Detached** | DbContext 不認識這個物件（例如剛 `new` 出來的） | 無 |
| **Added** | 透過 `Add()` 加入，準備新增 | `INSERT` |
| **Unchanged** | 從資料庫撈出來，尚未修改；或 `SaveChanges()` 後的狀態 | 無 |
| **Modified** | 撈出來後有屬性被改了 | `UPDATE` |
| **Deleted** | 透過 `Remove()` 標記刪除 | `DELETE` |

---

## 狀態轉換流程

```
new Expense()                          → Detached（DbContext 不認識）
    ↓ dbContext.Expenses.Add()
Added（準備 INSERT）
    ↓ SaveChanges()
Unchanged（已存入，沒有新的變動）
    ↓ expense.Amount = 500
Modified（偵測到值改了，準備 UPDATE）
    ↓ SaveChanges()
Unchanged（存完，現在的值變成新的「原始值」）
    ↓ dbContext.Expenses.Remove()
Deleted（準備 DELETE）
    ↓ SaveChanges()
Detached（資料庫裡沒了，不再追蹤）
```

**關鍵規律**：每次 `SaveChanges()` 之後，還留在追蹤裡的物件都會回到 **Unchanged**，就像「存檔」一樣。

---

## AsNoTracking() — 不追蹤查詢

當查詢結果只需要「讀取顯示、不會修改」時，可以加上 `.AsNoTracking()` 跳過追蹤：

```csharp
var expenses = dbContext.Expenses.AsNoTracking().ToList();
```

- 查出來的物件狀態是 **Detached**，DbContext 不認識它們
- **好處**：不需要記錄原始值、不需要比對，節省記憶體和提升查詢速度
- **適用場景**：列表頁、報表顯示等「只讀不改」的情境

---

## 重要觀念

- Change Tracking 是 **EF Core（DbContext）跟資料庫之間**的機制，跟 Controller 或檔案結構無關
- `Add()` 之後、`SaveChanges()` 之前，資料還**不在資料庫裡**（只是狀態標記為 Added）
- 把值設成跟原本一樣（例如 `expense.Amount = expense.Amount`），預設的 Snapshot 比對會發現值沒變，**不會產生 UPDATE**
