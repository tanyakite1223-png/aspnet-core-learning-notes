# 資料存取筆記：LINQ Join 與 GroupBy

## GroupBy — 分組統計

### 概念

- 對應 SQL 的 `GROUP BY`
- 將集合依照某個欄位分成好幾組，再對每組做統計（Sum、Count 等）

### 語法

```csharp
expenses
    .GroupBy(e => e.ExpenseDate.Month)
    .Select(g => new { Month = g.Key, Total = g.Sum(e => e.Amount) });
```

### 重點

- `.GroupBy()` 回傳的不是一筆一筆的資料，而是**一組一組**的資料
- `g` = 一整組（group），裡面包含該組所有資料
- `g.Key` = 分組依據的值（例如月份數字 1, 2, 3...）
- `g` 既是群組標籤（透過 `g.Key`），又是集合（可以對它做 `.Sum()`、`.Count()` 等）

### SQL 對照

| SQL | LINQ |
|-----|------|
| `GROUP BY MONTH(ExpenseDate)` | `.GroupBy(e => e.ExpenseDate.Month)` |
| `MONTH(ExpenseDate)` | `e.ExpenseDate.Month`（DateTime 的屬性） |
| `SUM(Amount)` | `g.Sum(e => e.Amount)` |
| `SELECT ... SUM(...)` | `.Select(g => new { ..., Total = g.Sum(...) })` |

---

## Join — 關聯查詢

### 概念

- 對應 SQL 的 `INNER JOIN`
- 將兩個集合透過共同欄位配對起來

### 語法（固定四個參數）

```csharp
expenses.Join(
    categories,                        // 1. 要 Join 的對象
    e => e.CategoryId,                 // 2. 左邊的 Key
    c => c.CategoryId,                 // 3. 右邊的 Key
    (e, c) => new { e.Title, c.Name }  // 4. 配對成功後產生什麼結果
);
```

### 重點

- 四個參數缺一不可，這是 Join 的固定寫法
- 參數順序跟著 `左邊.Join(右邊` 來的：左邊先、右邊後
- **第四個參數就是在做 Select 的事**（指定結果的形狀），所以 Join 後面不需要再接 `.Select()`
- `(e, c) =>` 是兩個參數的 Lambda Expression，因為配對成功後同時拿到左右兩邊的資料

### SQL 對照

| SQL | LINQ |
|-----|------|
| `FROM expenses` | `expenses.Join(` |
| `INNER JOIN categories` | 第 1 個參數：`categories` |
| `ON expenses.CategoryId =` | 第 2 個參數：`e => e.CategoryId` |
| `= categories.CategoryId` | 第 3 個參數：`c => c.CategoryId` |
| `SELECT Title, Name` | 第 4 個參數：`(e, c) => new { e.Title, c.Name }` |

---

## Join + GroupBy 結合使用

### 情境

查詢每個費用類別的總花費（需要先 Join 取得類別名稱，再分組統計）

### 串接順序

**Join → GroupBy → Select**

```csharp
expenses.Join(
    categories,
    e => e.CategoryId,
    c => c.CategoryId,
    (e, c) => new { c.Name, e.Amount }   // 先取出需要的欄位
)
.GroupBy(x => x.Name)                     // 依類別名稱分組
.Select(g => new
{
    Name = g.Key,
    TotalAmount = g.Sum(e => e.Amount)    // 每組加總
});
```

---

## 補充觀念

### Anonymous Type（匿名型別）

- `new { Name = g.Key, Total = g.Sum(...) }` — 不用事先定義 class，臨時組一個資料包
- 適合 LINQ 查詢中只需要暫時使用的結果

### Lambda 參數命名慣例

- `e` = 單筆資料（entity / expense）
- `g` = 一整組（group）
- 不同層次用不同名稱，避免混淆：`g.Sum(e => e.Amount)` ✅ vs `g.Sum(g => g.Amount)` ❌（能跑但不好讀）

### 強制轉型 vs 方法呼叫

- `(decimal)e.Amount` — 強制轉型（Cast），放在值的前面
- `Convert.ToDecimal(e.Amount)` — 方法呼叫（Method）
- `.ToDecimal()` 不存在，不要跟轉型搞混
