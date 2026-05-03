# 資料存取筆記:JOIN(INNER JOIN)

## 一、為什麼需要 JOIN?

當資料分散在多張表(例如 Products 和 Categories),要把它們合在一起查詢時,就需要 JOIN。

範例情境:列出「商品名稱、價格、分類名稱」——商品資訊在 Products 表,分類名稱在 Categories 表。

---

## 二、舊式 JOIN(隱式 JOIN / Implicit Join)

SQL-89 標準的寫法,FROM 後面用逗號列出多張表,然後用 WHERE 寫關聯條件:

```sql
SELECT Products.ProductName, Products.Price, Categories.CategoryName
FROM Products, Categories
WHERE Products.CategoryId = Categories.CategoryId
```

### 運作原理(內部觀念)

1. **Step 1:笛卡兒積(Cartesian Product)**——把兩張表的每一筆兩兩配對。
   - Products 5 筆 × Categories 3 筆 = 15 筆配對結果
2. **Step 2:WHERE 過濾**——只留下 `CategoryId` 相等的配對。

### 問題

| 問題 | 說明 |
|------|------|
| ⚠️ 忘記寫 WHERE 會炸 | 兩張各 10 萬筆的表 → 100 億筆笛卡兒積,可能拖垮資料庫 |
| 維護困難 | 多張表 JOIN 時,WHERE 子句會變得超級長 |
| 可讀性差 | 「關聯條件」和「過濾條件」混在 WHERE 裡,看不出哪段在做什麼 |

**結論**:現代寫法不用這個,改用 INNER JOIN。

---

## 三、INNER JOIN(SQL-92 標準寫法)

```sql
SELECT Products.ProductName, Products.Price, Categories.CategoryName
FROM Products
INNER JOIN Categories ON Products.CategoryId = Categories.CategoryId
```

### 語法重點

- **`INNER JOIN`**:指定要 JOIN 的另一張表
- **`ON`**:關聯條件——告訴資料庫「兩張表怎麼配對」
- **`WHERE`**:過濾條件——篩選最終結果

### 「INNER」的意思

> **雙邊都要「配對得上」的資料,才會出現在結果裡。**

- 如果 Categories 有「3C 用品」但 Products 沒有任何商品的 CategoryId 是它 → 「3C 用品」不會出現
- 如果 Products 有「神秘商品」但它的 CategoryId 在 Categories 找不到對應 → 「神秘商品」不會出現

⚠️ **重要觀念**:INNER JOIN **沒有誰主誰副**,是**雙向的**。FROM 哪一張表寫前面,結果都一樣。

---

## 四、ON 與 WHERE 的差別

| 子句 | 用途 |
|------|------|
| **ON** | 關聯條件——兩張表「怎麼配對」 |
| **WHERE** | 過濾條件——「哪些資料」要保留 |

範例:列出價格 >= 20 的商品

```sql
SELECT Products.ProductName, Products.Price, Categories.CategoryName
FROM Products
INNER JOIN Categories ON Products.CategoryId = Categories.CategoryId
WHERE Products.Price >= 20
```

- `ON Products.CategoryId = Categories.CategoryId` → 關聯條件
- `WHERE Products.Price >= 20` → 過濾條件

雖然把 `Price >= 20` 放到 ON 裡也跑得起來,但**語意就不對了**——它不是「配對的條件」,而是「過濾的條件」。實務上要分開寫。

---

## 五、本次練習用的表結構

### Categories(商品分類)

```sql
CREATE TABLE Categories(
    CategoryId INT PRIMARY KEY IDENTITY(1,1),
    CategoryName NVARCHAR(50) NOT NULL
)
```

### Products(商品)

```sql
CREATE TABLE Products(
    ProductId INT PRIMARY KEY IDENTITY(100,1),
    ProductName NVARCHAR(50) NOT NULL,
    Price DECIMAL(10,2) NOT NULL,
    CategoryId INT FOREIGN KEY REFERENCES Categories(CategoryId)
)
```

### CREATE TABLE 學到的細節

- **FOREIGN KEY 語法**:`欄位 INT FOREIGN KEY REFERENCES 表名(欄位名)`
  - `REFERENCES` 後面要用**小括號**指定欄位,不是用「點」
  - 為什麼用括號?因為 FOREIGN KEY 可以是多欄位的複合外鍵:`REFERENCES SomeTable(Col1, Col2)`
- **NVARCHAR(50)**:可變長度的 Unicode 字串,適合中英文混合
- **DECIMAL(10,2)**:存金額用,共 10 位數、其中 2 位小數(例如 99999999.99)。比 INT 更適合存價格。
- **IDENTITY(起始值, 累加值)**:自動編號
  - `IDENTITY(1,1)` = 從 1 開始,每次 +1
  - `IDENTITY(100,1)` = 從 100 開始,每次 +1
  - 只保證「不重複、遞增」,**不保證連續**(失敗的 INSERT 也會跳號)

---

## 六、INSERT 多筆資料的語法

```sql
-- Categories(IDENTITY 自動產生 CategoryId,所以欄位列表不寫它)
INSERT INTO Categories(CategoryName) VALUES
(N'文具'),
(N'飲料'),
(N'零食')

-- Products
INSERT INTO Products(ProductName, Price, CategoryId) VALUES
(N'原子筆', 15, 1),
(N'紅茶', 25, 2),
(N'鉛筆', 10, 1),
(N'洋芋片', 40, 3),
(N'咖啡', 45, 2)
```

### 重點

- **IDENTITY 欄位不要寫進 INSERT**,資料庫會自己生
- **多筆 INSERT 格式**:`VALUES (...), (...), (...)`——每筆一組小括號,中間用逗號,**不用外層括號**
- **N 字首**:表示 Unicode 字串(NVARCHAR 必須加,中文才不會變亂碼)
- **灌資料順序**:有外鍵關聯時,先灌「被參考」的表(主表),再灌「參考別人」的表(從表)。本例先 Categories 再 Products,否則 FOREIGN KEY 會擋下來——這個叫 **參考完整性(Referential Integrity)**

---

## 七、本次練習的三題

### Q1:列出價格 >= 20 的商品(商品名稱、價格、分類名稱)

```sql
SELECT Products.ProductName, Products.Price, Categories.CategoryName
FROM Products
INNER JOIN Categories ON Products.CategoryId = Categories.CategoryId
WHERE Products.Price >= 20
```

### Q2:列出「飲料」分類下的所有商品

```sql
SELECT Products.ProductName, Products.Price
FROM Products
INNER JOIN Categories ON Products.CategoryId = Categories.CategoryId
WHERE Categories.CategoryName = N'飲料'
```

注意:`CategoryName = '飲料'` 是**過濾條件**,放 WHERE,不放 ON。

### Q3:列出商品名稱與分類名稱,依分類名稱排序

```sql
SELECT Products.ProductName, Categories.CategoryName
FROM Products
INNER JOIN Categories ON Products.CategoryId = Categories.CategoryId
ORDER BY CategoryName ASC
```

---

## 八、常見錯誤與提醒

### ⚠️ 欄位名稱模稜兩可(Ambiguous column)

```sql
SELECT CategoryId FROM Products INNER JOIN Categories ON ...
-- 錯誤:資料行名稱 'CategoryId' 模稜兩可
```

兩張表都有 `CategoryId` 欄位,資料庫不知道要哪一個。

✅ **解法**:永遠加上表名前綴 `Products.CategoryId` 或 `Categories.CategoryId`。

實務上**所有欄位都建議加表名前綴**,即使現在沒歧義,未來表結構改動也不會有問題。

---

## 📌 補充參考(本次未實作)

### 其他 JOIN 類型

INNER JOIN 之外還有其他 JOIN,**本次未實作,只建立認知**:

| JOIN 類型 | 行為 |
|----------|------|
| **LEFT JOIN** | 左邊的表全部留下,右邊配不上的欄位補 NULL |
| **RIGHT JOIN** | 右邊的表全部留下,左邊配不上的欄位補 NULL |
| **FULL JOIN** | 兩邊都全部留下,配不上的補 NULL |
| **CROSS JOIN** | 笛卡兒積(等同舊式 `FROM A, B` 不加 WHERE) |

📌 **何時需要 LEFT JOIN?**

「列出所有分類,以及它底下的商品。**沒有商品的分類也要列出來**」——這時 INNER JOIN 不夠用,因為「3C 用品」這種沒商品的分類會被過濾掉。需要用 LEFT JOIN 才能保留它們。

---

## 九、本次學到的關鍵句

> **INNER JOIN 的核心規則:雙邊都要配對得上,才會出現。**
