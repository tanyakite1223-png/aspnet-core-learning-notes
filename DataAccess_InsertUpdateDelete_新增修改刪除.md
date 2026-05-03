# 資料存取筆記:INSERT / UPDATE / DELETE

## 一、本次學習的核心

SQL 的資料操作語言(DML, Data Manipulation Language)三大指令:

| 指令 | 用途 |
|---|---|
| `INSERT` | 新增資料 |
| `UPDATE` | 修改資料 |
| `DELETE` | 刪除資料 |

加上前一篇學的 `SELECT`(查詢),合稱資料庫操作的基本四大功能 — **CRUD**(Create / Read / Update / Delete)。

---

## 二、INSERT — 新增資料

### 1. 基本語法

```sql
INSERT INTO 資料表名 (欄位1, 欄位2, 欄位3)
VALUES (值1, 值2, 值3);
```

### 2. 範例:單筆 INSERT

```sql
INSERT INTO Categories(CategoryName) VALUES(N'3C 用品');
```

幾個重點:
- `CategoryId` 沒寫,因為它是 `IDENTITY` 自動編號欄位,由資料庫給值
- 中文字串前面記得加 `N` 前綴(沿用前一篇學到的鐵則)

### 3. 多筆 INSERT

```sql
INSERT INTO Categories(CategoryName) VALUES
(N'文具用品'),
(N'居家清潔'),
(N'美妝保養');
```

一次插入多筆,用逗號分隔每組 `()`。

### 4. INSERT 帶外來鍵

```sql
INSERT INTO Products(ProductName, Price, CategoryId)
VALUES(N'藍芽耳機', 1500, 4);
```

⚠️ 要插入的是 `CategoryId`(整數),不是 `CategoryName`(文字)。實務上需要先查 Categories 表才會知道對應的 ID。

> 📌 實務模式:**SELECT 跟 INSERT 永遠是配套使用**。寫入之前,先 SELECT 找到目標 ID。

---

## 三、IDENTITY 的關鍵特性 ⭐

### 1. 單向流水號,不補洞

| 行為 | 結果 |
|---|---|
| 插入新資料 | 編號 +1 |
| 刪除某一筆 | **空洞不會補回來** |
| 再插入新資料 | 接續上次的編號繼續 +1 |

### 2. 實際觀察

```
插入 1, 2, 3, 4 →  1, 2, 3, 4
刪除 4         →  1, 2, 3
再插入新的     →  1, 2, 3, 5  ← 跳過 4,直接給 5
```

### 3. 為什麼這樣設計?

因為 `Id` 在實務上會被**其他表參照**(例如 `Products.CategoryId = 5`)。如果資料庫補洞把新資料塞回 5 號,可能錯誤地把舊產品連到新分類 — 這會破壞**資料完整性**。

> 💡 一句話記住:**IDENTITY 是流水號,不是序號。**(流水號只往前流,序號可以重排)

---

## 四、UPDATE — 修改資料

### 1. 基本語法

```sql
UPDATE 資料表名
SET 欄位1 = 新值1, 欄位2 = 新值2
WHERE 條件;
```

### 2. 範例:單欄位修改

```sql
UPDATE Products SET Price = 18 WHERE ProductId = 100;
```

### 3. 多欄位同時改

```sql
UPDATE Products
SET Price = 12, CategoryId = 7
WHERE ProductId = 102;
```

`SET` 後面用逗號分隔多個欄位。

### 4. 用欄位本身做運算 ⭐

```sql
-- 飲料分類所有商品打 9 折
UPDATE Products SET Price = Price * 0.9 WHERE CategoryId = 2;
```

實務上超常用的模式:
- 商品全面降價:`Price = Price * 0.8`
- 庫存增加:`Stock = Stock + 100`
- 計數累加:`ViewCount = ViewCount + 1`

### 5. ⚠️ 沒寫 WHERE 的災難

```sql
-- 😱 災難級錯誤
UPDATE Products SET Price = 0;
```

**會把整張表所有商品價格都改成 0,而且不會報錯。** 比 DELETE 還可怕,因為錯了可能要等客訴才發現。

---

## 五、DELETE — 刪除資料

### 1. 基本語法

```sql
DELETE FROM 資料表名
WHERE 條件;
```

### 2. 範例:依主鍵刪除

```sql
DELETE FROM Products WHERE ProductId = 107;
```

### 3. 範例:依條件批次刪除

```sql
DELETE FROM Products WHERE CategoryId = 2;
```

### 4. ⚠️ 沒寫 WHERE 的災難

```sql
-- 😱 整張表清空,不會報錯
DELETE FROM Products;
```

實務上有人因為這樣丟了工作。`DELETE` 最重要的安全意識:**永遠檢查 WHERE 條件**。

---

## 六、Foreign Key 對 DELETE 的保護機制 ⭐

### 1. 觀念

當建表時這樣寫:

```sql
CategoryId INT FOREIGN KEY REFERENCES Categories(CategoryId)
```

這個 FK 不只是「標記關聯」,還會**強制保護資料完整性**:

> **不允許刪除「正在被別人參照」的資料。**

### 2. 實際錯誤訊息

當執行 `DELETE FROM Categories WHERE CategoryId = 1;` 而 Products 表裡有書還指著它,會看到:

```
The DELETE statement conflicted with the REFERENCE constraint
"FK__Products__Catego__534D60F1".
The conflict occurred in database "BookstoreDemo",
table "dbo.Products", column 'CategoryId'.
The statement has been terminated.
```

### 3. 訊息解讀

| 訊息片段 | 意思 |
|---|---|
| `conflicted with the REFERENCE constraint` | 牴觸了參照約束(FK) |
| `FK__Products__Catego__534D60F1` | 衝突的 FK 名稱(SQL Server 自動產生) |
| `table "dbo.Products", column 'CategoryId'` | Products 表的 CategoryId 正在參照妳要刪的資料 |
| `The statement has been terminated.` | 整個語句已被中止,沒有刪成 |

### 4. 白話翻譯

> 「妳要刪 Categories 1 號?**不行,Products 表裡有商品還指著它**。」

FK 在保護妳 — 不讓妳製造**孤兒資料**(指向不存在資料的紀錄)。

---

## 七、硬刪除 vs 軟刪除 ⭐⭐

### 1. 兩種「刪除」的對比

| 項目 | 硬刪除(DELETE) | 軟刪除(UPDATE IsActive = 0) |
|---|---|---|
| 資料還在嗎? | ❌ 真的不見了 | ✅ 還在,只是被標記 |
| 能恢復嗎? | ❌ 除非有備份 | ✅ 改回 `IsActive = 1` 就行 |
| 會被 FK 擋下嗎? | ✅ 會(常常做不到) | ❌ 不會(本來就沒在動參照關係) |
| 歷史紀錄會斷嗎? | ✅ 會(JOIN 不到) | ❌ 不會 |
| SELECT 要改寫嗎? | ❌ 不用 | ✅ 要加 `WHERE IsActive = 1` |

### 2. 實務上的選擇

| 情境 | 做法 |
|---|---|
| 商品下架 | 軟刪除 |
| 會員註銷 | 軟刪除 |
| 訂單取消 | 軟刪除(常用 `Status` 欄位標記) |
| 真的要清掉測試資料 | 硬刪除 |

### 3. 軟刪除的實作步驟

#### Step 1:用 ALTER TABLE 加欄位

```sql
ALTER TABLE Books
ADD IsActive BIT NOT NULL DEFAULT 1;
```

幾個重點:
- `BIT` 是 SQL Server 的布林型別(0 或 1)
- `NOT NULL` 表示這欄不能是 NULL
- `DEFAULT 1` 表示**現有資料自動填 1(啟用)**;以後新增的也預設 1

#### Step 2:用 UPDATE 進行軟刪除

```sql
UPDATE Books SET IsActive = 0 WHERE BookId = 8;
```

#### Step 3:後續 SELECT 要改寫

```sql
-- 給客人看的書籍列表
SELECT Title, Author, Price, Stock
FROM Books
WHERE IsActive = 1
ORDER BY Price ASC;
```

⚠️ **軟刪除的代價**:資料庫裡所有「對使用者展示」的 SELECT,都要記得加 `WHERE IsActive = 1`。少寫了會發生什麼事?**下架的書會跑去前台被客人看到** — 這是實務常見的 bug。

---

## 八、SSMS 寫入操作安全 SOP ⭐

### 為什麼需要 SOP?

實際操作中踩過的雷:**重複按 F5 造成 INSERT 兩次**、**沒選取就執行造成跑錯段落**。寫入操作出錯通常**不會報錯,只會默默地多/少/改了資料**。

### 安全 SOP 六步驟

| 步驟 | 動作 |
|---|---|
| 1️⃣ | 先用 SELECT 確認「目前狀態」 |
| 2️⃣ | 寫好 INSERT / UPDATE / DELETE,**先別執行** |
| 3️⃣ | 用滑鼠**反白**只要執行的那段 SQL |
| 4️⃣ | 看清楚 `WHERE` 條件**是不是妳要的範圍** |
| 5️⃣ | 按 F5 執行 |
| 6️⃣ | 立刻 SELECT 驗證結果 |

> 💡 工程師界的玩笑話:「**沒驗證等於沒做。**」

### SSMS 操作補充

- `USE BookstoreDemo;` 可以指定資料庫(配合 `GO` 使用)
- 工具列的資料庫下拉選單也可以切換 DB
- 新開 Query 視窗,**第一行就先 `USE 資料庫名稱;`** 是好習慣

---

## 九、ALTER TABLE 補充

本次首次接觸 `ALTER TABLE`,先記得「**加欄位**」這個用法:

```sql
ALTER TABLE 資料表名
ADD 欄位名 資料型別 [NOT NULL] [DEFAULT 值];
```

> 📌 補充參考(本次未深入):`ALTER TABLE` 還能改欄位型別、刪欄位、加 / 刪 FK 等,實務上需要時再查語法。

---

## 十、進階觀念(本次未實作,先有印象)

### 1. View(視圖)

可以建一個視圖,內建 `WHERE IsActive = 1`,程式只查視圖,就不會忘記過濾條件:

```sql
CREATE VIEW ActiveBooks AS
SELECT * FROM Books WHERE IsActive = 1;
```

### 2. EF Core 全域查詢過濾器

進入 EF Core 章節時會學 — 設定一次,所有查詢自動加上 `IsActive = 1`,徹底避免漏寫。

### 3. Transaction(交易)

重要的寫入會用 Transaction 包起來,失敗可以還原:

```sql
BEGIN TRANSACTION;
-- 多個寫入操作
COMMIT;  -- 或 ROLLBACK;
```

這些都是進階主題,先有印象即可。

---

## 十一、本次練習題彙整

```sql
-- 第 1 題:單筆 INSERT
INSERT INTO Categories(CategoryName) VALUES(N'3C 用品');

-- 第 2 題:多筆 INSERT
INSERT INTO Categories(CategoryName) VALUES
(N'文具用品'),
(N'居家清潔'),
(N'美妝保養');

-- 第 3 題:INSERT 帶 FK
INSERT INTO Products(ProductName, Price, CategoryId)
VALUES(N'藍芽耳機', 1500, 4);

-- 第 4 題:單欄位 UPDATE
UPDATE Products SET Price = 18 WHERE ProductId = 100;

-- 第 5 題:多欄位 UPDATE
UPDATE Products SET Price = 12, CategoryId = 7 WHERE ProductId = 102;

-- 第 6 題:用欄位本身做運算
UPDATE Products SET Price = Price * 0.9 WHERE CategoryId = 2;

-- 第 7 題:DELETE 依條件批次刪
DELETE FROM Products WHERE CategoryId = 2;

-- 第 8 題:ALTER TABLE 加欄位(軟刪除前置作業)
ALTER TABLE Books
ADD IsActive BIT NOT NULL DEFAULT 1;

-- 第 9 題:軟刪除
UPDATE Books SET IsActive = 0 WHERE BookId = 8;

-- 第 10 題:軟刪除後的查詢(過濾下架資料)
SELECT Title, Author, Price, Stock
FROM Books
WHERE IsActive = 1
ORDER BY Price ASC;
```

---

## 十二、重點口訣

1. **INSERT 不寫 IDENTITY 欄位** — 資料庫自動給,人工別管
2. **IDENTITY 是流水號,不是序號** — 只往前走,不補洞
3. **UPDATE / DELETE 沒寫 WHERE,整張表炸給妳看** — 還不會報錯
4. **FK 是資料的保護傘** — 防止製造孤兒資料
5. **軟刪除 = UPDATE IsActive = 0** — 資料還在,只是「下架」
6. **軟刪除的代價** — 所有 SELECT 都要記得加 `WHERE IsActive = 1`
7. **寫入操作 SOP** — 反白選取、看清 WHERE、寫入後立刻驗證
8. **沒驗證等於沒做**

---

## 十三、SQL vs 實務對應(預告)

未來進入 ASP.NET Core + EF Core 後,妳會用 LINQ 而不是直接寫 SQL。但底層還是 SQL — 觀念對應如下:

| SQL 操作 | LINQ / EF Core 對應 |
|---|---|
| `INSERT` | `context.Books.Add(book); context.SaveChanges();` |
| `UPDATE` | 改物件的屬性,然後 `SaveChanges()` |
| `DELETE` | `context.Books.Remove(book); context.SaveChanges();` |
| `WHERE IsActive = 1` | LINQ 的 `.Where(b => b.IsActive)` 或全域過濾器 |

> 📌 補充參考(本次未實作):這些等學到 EF Core 章節會深入。
