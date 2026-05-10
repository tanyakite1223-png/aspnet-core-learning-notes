# 資料存取筆記:子查詢(Subquery)

## 一、什麼是子查詢

**子查詢(Subquery)**:把一個 SELECT 的結果,拿來給另一個 SELECT 當作條件值或來源。

### 核心觀念

- 一句 SQL 中,巢狀地包含另一句 SELECT(用括號 `()` 包起來)
- **執行順序**:先算內層,得到結果,再丟給外層使用
- 內層 SELECT 跟外層 SELECT 在資料庫眼中是**兩個獨立的查詢**,各自要宣告自己的 FROM

### 為什麼需要子查詢

當問題涉及「**兩個不同層次的事**」時,單一句 SELECT 沒辦法同時做。

範例情境:**「找出價格高於全部書籍平均價格的書」**

- 「全部書籍的平均價」是一個聚合值(整張表算出一個數字)
- 「每一本書的價格」是逐筆資料
- 這兩件事不能在同一個 SELECT 裡同時做 → 需要子查詢

---

## 二、子查詢的三種基本用法

### 用法 1:子查詢回傳「一個聚合值」,搭配比較運算子(`>`、`<`、`=`)

```sql
-- 找出價格高於平均的書
SELECT *
FROM Books
WHERE Price > (SELECT AVG(Price) FROM Books)
```

執行流程:
```
內層:SELECT AVG(Price) FROM Books → 算出 494
外層:WHERE Price > 494 → 找出 Price > 494 的書
```

其他常見聚合函數搭配:
- `WHERE Price = (SELECT MAX(Price) FROM Books)` — 找最高價的書
- `WHERE Price < (SELECT AVG(Price) FROM Books)` — 找低於平均的書

### 用法 2:子查詢回傳「一個 ID」,可用 `=` 或 `IN`

```sql
-- 找出分類名稱叫「程式設計」的所有書(假設此分類只對應一個 CategoryId)
SELECT *
FROM Books
WHERE CategoryId = (SELECT CategoryId FROM Categories WHERE CategoryName = N'程式設計')
```

當子查詢只回傳一筆時,`=` 跟 `IN` 都可以,結果一樣。

### 用法 3:子查詢回傳「多個值」,**必須用 `IN`**

```sql
-- 找出分類 ID 在 1 或 2 的所有書
SELECT *
FROM Books
WHERE CategoryId IN (SELECT CategoryId FROM Categories WHERE CategoryId <= 2)
```

如果在這個情境下硬用 `=`,會看到 SQL Server 的錯誤訊息:

```
Subquery returned more than 1 value.
This is not permitted when the subquery follows =, !=, <, <=, >, >=
or when the subquery is used as an expression.
```

(子查詢回傳超過 1 個值,不允許接在 `=`、`!=`、`<`、`<=`、`>`、`>=` 後面)

---

## 三、`=` vs `IN` 對照表

| 子查詢回傳 | 用 `=` | 用 `IN` |
|-----------|-------|--------|
| 一個值(例:聚合函數結果) | ✅ 可以 | ✅ 可以(但少用) |
| 多個值(例:符合條件的多個 ID) | ❌ **報錯** | ✅ 可以 |
| 零個值(找不到符合條件的) | ⚠️ 結果為空 | ⚠️ 結果為空 |

### 安全準則

> **不確定子查詢會回傳幾個值時,預設用 `IN`。**
> **`=` 只用在「百分之百確定子查詢只回傳一個值」的情況**,
> 最典型的就是聚合函數(AVG、MAX、MIN、SUM、COUNT)。

---

## 四、`NOT IN`:反向篩選

`NOT IN` 表示「**不在...裡面**」,結構跟 `IN` 完全一樣,只是反過來篩。

```sql
-- 找出分類「不是」程式設計的所有書
SELECT *
FROM Books
WHERE CategoryId NOT IN (SELECT CategoryId FROM Categories WHERE CategoryName = N'程式設計')
```

---

## 五、本次練習中遇到的觀念補充

### 1. 內層 SELECT 也要寫 FROM

直覺上會想:「外層已經 `FROM Books`,內層應該就不用再寫了吧?」

**錯。** 內層 SELECT 跟外層 SELECT 在資料庫眼中是兩個獨立查詢,各自要自己宣告 FROM。

```sql
-- ❌ 錯誤:內層缺 FROM
WHERE Price > (SELECT AVG(Price))

-- ✅ 正確
WHERE Price > (SELECT AVG(Price) FROM Books)
```

唯一例外:如果 SELECT 只是算常數或函數(如 `SELECT 1+1`、`SELECT GETDATE()`),可以不需要 FROM。但只要涉及表的欄位,一定要 FROM。

### 2. WHERE 不能用聚合函數

聚合函數(`SUM`、`AVG`、`COUNT` 等)**只能用在 SELECT、HAVING、ORDER BY**,不能用在 WHERE。

```sql
-- ❌ WHERE 裡不能用聚合函數
SELECT * FROM Books WHERE Price > AVG(Price)
```

如果想做「逐筆比對 vs 整體聚合值」這種事,就要用子查詢。

### 3. SQL 條件要表達「真正的意圖」,不靠資料巧合

範例:本次練習中,目標是「只留下 CategoryId 1~4」,正確的條件是:

```sql
DELETE FROM Categories WHERE CategoryId > 4;
```

如果寫成 `> 6`,雖然當下資料剛好不會有問題,但**意圖被表達錯了**。
資料一變(例如多了 CategoryId=5 的資料,或 6 變成你不想刪的東西),條件就不再正確。

> **教訓**:條件要表達意圖,不要靠資料的巧合「剛好對」。

### 4. 改資料的反射動作:SELECT → 修改 → SELECT 驗證

直接下 DELETE / UPDATE 之前,先 SELECT 確認影響範圍;
做完之後,再 SELECT 一次驗證結果。

```sql
-- 改之前先看
SELECT * FROM Categories WHERE CategoryId > 4;

-- 執行
DELETE FROM Categories WHERE CategoryId > 4;

-- 改之後驗證
SELECT * FROM Categories;
```

### 5. FK 衝突的正確處理方式

**錯誤處理**:把違反 FK 的紀錄隨便改成另一個合法值,只為了讓 DELETE 能跑通。

```sql
-- ❌ 危險!為了刪除 Categories 中的某筆,把 Products 的 CategoryId 隨便改成 1
UPDATE Products SET CategoryId = 1 WHERE ProductId = 102;
DELETE FROM Categories WHERE ...;
```

**問題**:這種做法會造成**資料污染**——商品被歸到錯誤的分類。
例如「鉛筆」原本屬於文具(假設 CategoryId=8),被改成 CategoryId=1,
而 CategoryId=1 後來被改名成「程式設計」,結果鉛筆變成「程式設計」分類的商品 → 資料完全錯亂。

**正確處理(視情境選一)**:
1. 如果這筆資料不該存在了 → 先 DELETE 子表(Products)那筆,再 DELETE 主表(Categories)
2. 如果應該保留但分類確實要改 → 想清楚它真正屬於哪個分類,UPDATE 成正確的值
3. 如果是練習資料要重來 → 整批清空重做

> **核心觀念**:不要為了滿足技術限制(FK 不能違反)而修改業務資料的正確性。

---

## 六、本次實際執行的 SQL 整理

### 練習 1:聚合值比較

```sql
-- 找出價格高於平均的書
SELECT *
FROM Books
WHERE Price > (SELECT AVG(Price) FROM Books)

-- 找出價格低於平均的書
SELECT *
FROM Books
WHERE Price < (SELECT AVG(Price) FROM Books)

-- 找出價格等於最高價的書
SELECT *
FROM Books
WHERE Price = (SELECT MAX(Price) FROM Books)
```

### 練習 2:用分類名稱查書(子查詢回傳一個 ID)

```sql
SELECT *
FROM Books
WHERE CategoryId = (SELECT CategoryId FROM Categories WHERE CategoryName = N'程式設計')

-- 同樣的事用 IN 寫,結果一樣但更穩健
SELECT *
FROM Books
WHERE CategoryId IN (SELECT CategoryId FROM Categories WHERE CategoryName = N'程式設計')
```

### 練習 3:子查詢回傳多個值,必須用 IN

```sql
-- 故意讓子查詢回傳多筆,觀察 = 報錯
SELECT *
FROM Books
WHERE CategoryId = (SELECT CategoryId FROM Categories WHERE CategoryId <= 2)
-- → 錯誤訊息 512:Subquery returned more than 1 value.

-- 改用 IN 就正常
SELECT *
FROM Books
WHERE CategoryId IN (SELECT CategoryId FROM Categories WHERE CategoryId <= 2)
```

### 練習 4:NOT IN

```sql
-- 找出分類「不是」程式設計的所有書
SELECT *
FROM Books
WHERE CategoryId NOT IN (SELECT CategoryId FROM Categories WHERE CategoryName = N'程式設計')
```

---

## 七、本次主題 SQL 基礎完結

到本次為止,SQL 基礎(三. 1)完成。涵蓋的內容:

- ✅ RDBMS 觀念(Table、Column、Row、PK、FK)
- ✅ 一對多關聯
- ✅ SELECT + WHERE + ORDER BY
- ✅ JOIN(INNER JOIN)
- ✅ INSERT / UPDATE / DELETE
- ✅ GROUP BY + 聚合函數(COUNT、SUM)
- ✅ **子查詢(Subquery)** ← 本次

下一個分類是「三. 1.5 實戰環境準備」,將從 Visual Studio + LocalDB 練習場,
切換到 VS Code + Claude Code CLI 的實戰場,啟動報銷系統(ExpenseSystem)專題。

---

## 📌 補充參考(本次未實作)

以下內容**不在本次練習範圍內**,只是知道有這些東西即可,
等將來需要時再學:

- **`EXISTS` / `NOT EXISTS`**:另一種子查詢用法,用「子查詢有沒有結果」當條件,而不是用值比較。某些情況下比 IN 更有效率。
- **相關子查詢(Correlated Subquery)**:內層子查詢可以參考外層的欄位值,例如「找出每個分類中價格最高的書」。
- **子查詢用在 FROM 之後(衍生資料表)**:把子查詢的結果當作一張臨時表來用。
- **子查詢用在 SELECT 之後**:在輸出欄位中嵌入子查詢結果。
