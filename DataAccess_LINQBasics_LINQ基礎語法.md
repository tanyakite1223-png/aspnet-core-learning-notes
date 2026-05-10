# 資料存取筆記：LINQ 基礎語法 — Where、Select、OrderBy

## LINQ 是什麼？

LINQ（Language Integrated Query，語言整合查詢）讓你在 C# 程式碼中對集合（Collection）進行篩選、轉換、排序等操作，概念類似 SQL 的 `WHERE`、`SELECT`、`ORDER BY`，但對象是 C# 的 `List<T>` 等集合。

## 兩種寫法

| 寫法 | 名稱 | 特徵 |
|------|------|------|
| Query Syntax（查詢語法） | `from e in list where ... select ...` | 長得像 SQL |
| Method Syntax（方法語法） | `list.Where(...).Select(...)` | 長得像一般 C# 程式碼 |

**實務上以 Method Syntax 為主**，因為風格一致、可自由串接（chaining）。

## Lambda Expression（Lambda 運算式）

Method Syntax 的核心語法：

```csharp
e => e.Amount > 500
```

- `e` → 參數（代表集合裡的每一筆資料）
- `=>` → 箭頭，讀作「goes to」
- `e.Amount > 500` → 要做的事

參數名稱自己取，慣例用**型別名稱的第一個小寫字母**（`Expense` → `e`）。

## 三個基本方法

### `.Where()` — 篩選（條件判斷）

篩出符合條件的資料，數量可能變少。

```csharp
// 篩出金額大於 200 的報銷
var result = expenses.Where(e => e.Amount > 200m);
```

對應 SQL：`WHERE Amount > 200`

### `.Select()` — 轉換（決定輸出的形狀）

把每筆資料轉換成另一個形式，數量不變。

```csharp
// 只取出 Title
var titles = expenses.Select(e => e.Title);

// 轉成大寫
var upper = fruits.Select(f => f.ToUpper());
```

對應 SQL：`SELECT Title FROM ...`

> 📌 `.Select()` 不只是「選欄位」，更精確地說是**轉換**——可以改變資料的形狀。

### `.OrderBy()` / `.OrderByDescending()` — 排序

```csharp
// 按金額由小到大（ASC）
var result = expenses.OrderBy(e => e.Amount);

// 按金額由大到小（DESC）
var result = expenses.OrderByDescending(e => e.Amount);
```

對應 SQL：`ORDER BY Amount ASC` / `ORDER BY Amount DESC`

> 📌 排序物件集合時，必須指定**排序依據的屬性**（例如 `e.Amount`），不能直接寫 `e => e`（物件本身無法比大小）。字串、數字等基本型別才可以 `f => f`。

## 串接（Chaining）

三個方法可以用 `.` 串在一起：

```csharp
// 篩出金額 > 200 → 按金額排序 → 只取 Title
var result = expenses
    .Where(e => e.Amount > 200m)
    .OrderBy(e => e.Amount)
    .Select(e => e.Title);
```

**串接順序沒有強制規定**，根據需求決定。但要注意：
- 實務習慣：**先篩選（Where）再排序或轉換**，減少後續處理量
- 順序會影響後面能用的屬性：如果先 `.Select(e => e.Title)` 只取了 Title，後面就拿不到 `Amount` 來排序了

## 常用輔助方法

### `.Count()` — 計算筆數

```csharp
// 可以直接放條件，不需要先 .Where()
var count = expenses.Count(e => e.Amount > 200m);
```

### `.Any()` — 是否有任何一筆符合（回傳 bool）

```csharp
var hasExpensive = expenses.Any(e => e.Amount > 1000);
// 回傳 true 或 false
```

### `.FirstOrDefault()` — 取第一筆符合的資料

```csharp
var first = expenses.FirstOrDefault(e => e.Amount > 200);
```

找不到時的回傳值取決於型別：
- 參考型別（如 `Expense` 物件）→ `null`
- 值型別（如 `int`）→ 該型別的預設值（`int` → `0`）

### Null-conditional Operator（`?.`）

搭配 `.FirstOrDefault()` 使用，避免結果為 `null` 時存取屬性爆錯：

```csharp
var first = expenses.FirstOrDefault(e => e.Amount > 200);
Console.WriteLine(first?.Title);   // 如果 first 是 null，整個結果變 null，不會爆錯
Console.WriteLine(first?.Amount);
```

## SQL 與 LINQ 對照表

| SQL | LINQ Method Syntax |
|-----|-------------------|
| `WHERE Amount > 200` | `.Where(e => e.Amount > 200)` |
| `SELECT Title` | `.Select(e => e.Title)` |
| `ORDER BY Amount` | `.OrderBy(e => e.Amount)` |
| `ORDER BY Amount DESC` | `.OrderByDescending(e => e.Amount)` |
| `COUNT(*)` | `.Count()` |
| `TOP 1` / `LIMIT 1` | `.FirstOrDefault()` |
| `EXISTS` | `.Any()` |
