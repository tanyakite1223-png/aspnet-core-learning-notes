# 資料存取筆記:資料關聯 — 一對多關聯

## 一、核心觀念

### Primary Key(主鍵,PK)

> 一張表中,**唯一識別每一筆資料**的欄位

特性:
- 值不能重複
- 不能是 NULL
- 一張表通常只有一個 PK

### Foreign Key(外鍵,FK)

> 一張表中,**指向另一張表 PK** 的欄位

作用:
- 建立兩張表之間的「關聯」
- 強制資料完整性(不能指向不存在的資料)
- 防止亂刪資料(資料還被其他表參照時無法刪除)

---

## 二、一對多關聯設計原則

### 範例情境

一位作者可以寫很多本書,但每一本書只屬於一位作者。

```
Authors 表 (1)              Books 表 (多)
┌────────────┐              ┌────────────────┐
│ Id    (PK) │◄─────────────│ Id        (PK) │
│ Name       │              │ Title          │
│ Country    │              │ ReleasedDate   │
└────────────┘              │ Price          │
                            │ AuthorId  (FK) │
                            └────────────────┘
```

### 關鍵原則:**FK 放在「多」的那一邊**

如果反過來把 FK 放在「一」那邊,會造成:
- 資料重複(同一個作者出現多次)
- 一格塞多個值(違反第一正規化)
- 資料難以維護

---

## 三、命名慣例(EF Core 標準)

| 項目 | 規則 | 範例 |
|------|------|------|
| 表名 | 複數、PascalCase | `Authors`、`Books`、`Orders` |
| PK | 一律叫 `Id` | `Id` |
| FK | `{指向的類別名}Id` | `AuthorId`、`CustomerId` |
| 一般欄位 | PascalCase,不重複表名 | `Name`、`Title`、`Price` |
| 複合單字 | 每個單字大寫,沒有底線 | `ReleasedDate`、`PhoneNumber` |

### 業界慣用時間欄位命名

| 欄位 | 意義 |
|------|------|
| `CreatedAt` | 資料建立時間 |
| `UpdatedAt` | 資料最後更新時間 |
| `DeletedAt` | 資料刪除時間(軟刪除用) |
| `CreatedBy` | 建立者 |
| `UpdatedBy` | 更新者 |

> **At** + 時間 → 「在什麼時間發生」
> **By** + 人 → 「由誰執行」

### 補充:軟刪除 vs 硬刪除(本次未實作)

| 類型 | 做法 | 結果 |
|------|------|------|
| 硬刪除 | 真的把資料從資料庫移除 | 資料消失,救不回來 |
| 軟刪除 | 在 `DeletedAt` 欄位填上時間 | 資料還在,只是「標記為已刪除」 |

軟刪除好處:可還原、保留歷史記錄、避免誤刪災難。

---

## 四、SQL 實作流程

### 1. 建立資料庫

```sql
CREATE DATABASE LearningDb;
```

### 2. 切換資料庫

```sql
USE LearningDb;
```

### 3. 建立 Authors 表

```sql
CREATE TABLE Authors (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Name NVARCHAR(50) NOT NULL,
    Country NVARCHAR(50)
);
```

### 4. 建立 Books 表(含 FK)

```sql
CREATE TABLE Books (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Title NVARCHAR(100) NOT NULL,
    ReleasedDate DATE,
    Price DECIMAL(10, 2),
    AuthorId INT NOT NULL,
    FOREIGN KEY (AuthorId) REFERENCES Authors(Id)
);
```

### 5. 新增資料

```sql
-- 新增作者
INSERT INTO Authors (Name, Country) VALUES (N'村上春樹', N'日本');
INSERT INTO Authors (Name, Country) VALUES (N'J.K. Rowling', N'英國');
INSERT INTO Authors (Name, Country) VALUES (N'吳曉樂', N'台灣');

-- 新增書籍(指定 AuthorId 對應到作者)
INSERT INTO Books (Title, ReleasedDate, Price, AuthorId) 
VALUES (N'挪威的森林', '1987-09-04', 320, 1);

INSERT INTO Books (Title, ReleasedDate, Price, AuthorId) 
VALUES (N'1Q84', '2009-05-29', 450, 1);
```

### 6. 查詢(含 JOIN,本次體驗)

```sql
SELECT 
    Books.Title AS 書名,
    Books.Price AS 價格,
    Authors.Name AS 作者,
    Authors.Country AS 國籍
FROM Books
INNER JOIN Authors ON Books.AuthorId = Authors.Id;
```

---

## 五、SQL 語法重點

### 資料型別

| 型別 | 用途 | 範例 |
|------|------|------|
| `INT` | 整數 | `1`、`100` |
| `NVARCHAR(n)` | 中英文都能存的可變長度文字 | `N'村上春樹'` |
| `DATE` | 只有日期 | `'2024-01-15'` |
| `DECIMAL(p, s)` | 精準的小數(總共 p 位、小數 s 位) | `DECIMAL(10, 2)` 可存 `12345678.99` |

> ⚠️ **金額一律用 DECIMAL,絕對不要用 FLOAT/DOUBLE**(浮點數會有誤差)

### 關鍵字與符號

| 關鍵字 | 意義 |
|--------|------|
| `CREATE DATABASE` | 建立資料庫 |
| `CREATE TABLE` | 建立資料表 |
| `USE` | 切換到指定資料庫 |
| `PRIMARY KEY` | 設定主鍵 |
| `IDENTITY(1,1)` | 自動編號(從 1 開始,每次 +1) |
| `NOT NULL` | 必填(不允許 NULL) |
| `FOREIGN KEY (...) REFERENCES 表(欄位)` | 設定外鍵 |
| `INSERT INTO` | 新增資料 |
| `SELECT * FROM` | 查詢所有欄位 |
| `INNER JOIN ... ON ...` | 內部連接(體驗用,後續主題深入) |

### 字串前綴 `N`

存中文一定要用 `NVARCHAR` 型別,且值前面要加 `N`:

```sql
N'村上春樹'   ✅ 正確
'村上春樹'    ❌ 可能變成 ???
```

### 業界寫作慣例

- 關鍵字全部大寫:`SELECT`、`FROM`、`WHERE`
- 表名與欄位名:PascalCase
- 每行 SQL 結尾加 `;`

---

## 六、實驗驗證:FK 真的會擋掉爛資料

### 故意塞錯誤資料

```sql
-- Authors 表只有 Id = 1, 2, 3
INSERT INTO Books (Title, ReleasedDate, Price, AuthorId) 
VALUES (N'幽靈作者的書', '2024-01-01', 500, 999);
-- ↑ AuthorId = 999 不存在
```

### 系統回應

```
訊息 547,層級 16,狀態 0
The INSERT statement conflicted with the FOREIGN KEY constraint 
"FK_Books_AuthorId_xxx".
The statement has been terminated.
```

### 解讀

- **錯誤代碼 547** → 專門代表「外鍵約束違反」,以後看到這個編號就知道是 FK 問題
- **層級 16** → 一般錯誤,可由使用者修正
- **效果** → 該筆爛資料**沒有寫入**,Books 表保持乾淨

---

## 七、SSMS 操作小技巧

### 確認 PK 與 FK 是否設定成功

在物件總管中:
1. 展開資料表 → 展開「索引鍵」資料夾
2. 看到 `PK_xxx` → 主鍵存在
3. 看到 `FK_xxx` → 外鍵存在

### 欄位圖示意義

| 圖示 | 意義 |
|------|------|
| 🔑 鑰匙 | Primary Key |
| 🔗 鎖鏈 | Foreign Key |

### 查詢視窗操作

- **F5** 或上方「執行(X)」按鈕 → 執行 SQL
- **下方「結果」標籤** → 顯示查詢結果(成功時)
- **下方「訊息」標籤** → 顯示執行訊息或錯誤(失敗時)
- **左上角下拉選單** → 切換目前操作的資料庫

---

## 八、本次練習用的完整資料

### Authors 表

| Id | Name | Country |
|----|------|---------|
| 1 | 村上春樹 | 日本 |
| 2 | J.K. Rowling | 英國 |
| 3 | 吳曉樂 | 台灣 |

### Books 表

| Id | Title | ReleasedDate | Price | AuthorId |
|----|-------|--------------|-------|----------|
| 1 | 挪威的森林 | 1987-09-04 | 320.00 | 1 |
| 2 | 1Q84 | 2009-05-29 | 450.00 | 1 |
| 3 | 哈利波特:神秘的魔法石 | 1997-06-26 | 380.00 | 2 |
| 4 | 哈利波特:消失的密室 | 1998-07-02 | 380.00 | 2 |
| 5 | 你的孩子不是你的孩子 | 2014-05-10 | 280.00 | 3 |

### JOIN 查詢結果

| 書名 | 價格 | 作者 | 國籍 |
|------|------|------|------|
| 挪威的森林 | 320.00 | 村上春樹 | 日本 |
| 1Q84 | 450.00 | 村上春樹 | 日本 |
| 哈利波特:神秘的魔法石 | 380.00 | J.K. Rowling | 英國 |
| 哈利波特:消失的密室 | 380.00 | J.K. Rowling | 英國 |
| 你的孩子不是你的孩子 | 280.00 | 吳曉樂 | 台灣 |

---

## 九、學習環境

- **資料庫引擎**:LocalDB(`(localdb)\MSSQLLocalDB`,SQL Server 2025 17.0.1000.7)
- **管理工具**:SQL Server Management Studio (SSMS)
- **練習資料庫**:`LearningDb`

---

## 📌 補充說明:本次涉及但未深入的主題

以下指令在本次「體驗式實作」中有用到,但**不是本主題的學習目標**,後面會有專門主題深入學:

- `SELECT` + `WHERE` + `ORDER BY` → 下個主題
- `INSERT` / `UPDATE` / `DELETE` → 後續主題
- `JOIN`(INNER JOIN) → 後續主題

本次目的是讓「一對多關聯」的概念**從紙上設計變成親手實作的具體經驗**,後續學每個指令時會更紮實。
