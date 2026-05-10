# 資料存取筆記:GROUP BY + 聚合函數

---

## 核心觀念

一般的 `SELECT + WHERE` 只能過濾資料，無法做「分組統計」。  
要達到「每個分類各有幾筆」這類需求，需要兩個動作：

1. **先分組** → `GROUP BY`
2. **再統計** → 聚合函數（Aggregate Functions）

---

## 聚合函數（Aggregate Functions）

| 函數 | 用途 |
|------|------|
| `COUNT(*)` | 計算筆數 |
| `SUM(欄位)` | 計算加總 |
| `AVG(欄位)` | 計算平均（補充參考）|
| `MAX(欄位)` | 最大值（補充參考）|
| `MIN(欄位)` | 最小值（補充參考）|

---

## GROUP BY 基本語法

```sql
SELECT CategoryId, COUNT(*) AS 筆數
FROM Books
GROUP BY CategoryId;
```

- `GROUP BY CategoryId` → 按 CategoryId 分組
- `COUNT(*)` → 計算每組的筆數
- `AS 筆數` → 幫結果欄位取別名

---

## SUM 範例

```sql
SELECT CategoryId, SUM(Price) AS 總價
FROM Books
GROUP BY CategoryId;
```

---

## HAVING — 分組後的條件過濾

`WHERE` 是在分組**之前**過濾原始資料，看不到聚合函數的結果。  
要過濾分組**之後**的統計結果，必須用 `HAVING`。

```sql
SELECT CategoryId, SUM(Price) AS 總價
FROM Books
GROUP BY CategoryId
HAVING SUM(Price) > 600;
```

> 記住：條件裡有 `COUNT`、`SUM` 等聚合函數 → 用 `HAVING`，不是 `WHERE`

---

## SQL 子句執行順序（固定）

```
SELECT
FROM
WHERE      ← 分組前過濾（針對原始資料）
GROUP BY
HAVING     ← 分組後過濾（針對統計結果）
ORDER BY
```

HAVING 一定在 GROUP BY **之後**，順序不能對調。

---

## 綜合範例

> 統計每個分類的書籍數量，只顯示數量大於 1 的分類，並按數量由多到少排序

```sql
SELECT CategoryId, COUNT(*) AS 總量
FROM Books
GROUP BY CategoryId
HAVING COUNT(*) > 1
ORDER BY 總量 DESC;
```

---

## 常見錯誤

| 錯誤 | 原因 | 修正 |
|------|------|------|
| `GROUP BY COUNT(CategoryId)` | GROUP BY 後面要接欄位，不是函數 | `GROUP BY CategoryId` |
| `WHERE SUM(Price) > 600` | WHERE 無法過濾聚合結果 | 改用 `HAVING` |
| `HAVING` 放在 `GROUP BY` 前 | 順序錯誤 | HAVING 必須在 GROUP BY 之後 |
