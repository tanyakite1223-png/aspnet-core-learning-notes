# ASP.NET Core 筆記：MVC 基礎概念與實作

---

## 1. Class（類別）與 Object（物件）

| 概念 | 生活比喻 | C# 術語 |
|------|----------|---------|
| 空白表格範本 | 客戶資料卡範本 | `class`（類別） |
| 填好的一張表格 | 一筆客戶資料 | `object`（物件） |
| 建立一張新表格 | 拿範本填一份新的 | `new Customer()` |

- `class` 只是範本，本身不包含實際資料
- 有幾筆資料，就要建立幾個 `object`

### 範例

```csharp
// class 定義（範本）
public class Customer
{
    public string CompanyName { get; set; }
    public string Name { get; set; }
    public string Phone { get; set; }
    public string Email { get; set; }
    public string Address { get; set; }
}

// 建立 object（實際資料）
Customer c = new Customer();
c.Name = "王小明";
c.Phone = "0912345678";
```

---

## 2. MVC 架構

| 餐廳比喻 | 餐廳工作內容 | MVC 角色 | 負責的事 |
|----------|-------------|----------|----------|
| 服務生 | 接受點餐、協調廚房、端菜給客人 | **Controller**（控制器） | 接收請求、協調、回傳結果 |
| 廚房 | 準備食材、做出料理 | **Model**（模型） | 處理資料、商業邏輯 |
| 菜單/擺盤 | 呈現給客人看 | **View**（視圖） | 呈現畫面給使用者 |

### 請求流程

```
使用者（瀏覽器）→ Controller → Model → View → 使用者看到畫面
```

---

## 3. Routing（路由）

網址規則：`/[Controller名稱]/[Action名稱]`

| 網址 | Controller | Action 與對應 View |
|------|------------|--------|
| `/Home/Index` | `HomeController` | `Index()` → `Views/Home/Index.cshtml` |
| `/Home/Privacy` | `HomeController` | `Privacy()` → `Views/Home/Privacy.cshtml` |
| `/Customer/Index` | `CustomerController` | `Index()` → `Views/Customer/Index.cshtml` |

### 重要規則
- 放在 `Controllers` 資料夾下的檔案，命名必須加上 `Controller` 結尾
- ASP.NET Core 會自動去除 `Controller` 字眼來對應網址
- 不需要自己寫這個對應邏輯，框架自動處理

---

## 4. Action 與 View 的對應規則

看到 Action → 就知道 `Views/` 裡面一定有對應的檔案。

**對應路徑公式：`Views/[Controller名稱]/[Action名稱].cshtml`**

`return View()` 預設會去找對應的 `.cshtml` 檔案：

```
CustomerController 的 Index()   → Views/Customer/Index.cshtml
CustomerController 的 Detail()  → Views/Customer/Detail.cshtml
HomeController 的 Privacy()     → Views/Home/Privacy.cshtml
```

**規則：Controller 名稱 → 資料夾名稱，Action 名稱 → 檔案名稱**

### return View() 在做什麼？
去找對應的 `.cshtml` 檔案，把它 render（渲染）成 HTML 畫面，回傳給瀏覽器。

### 潛規則：Convention over Configuration（慣例優於設定）
`return View()` 不需要指定路徑，框架自動依照慣例去找對應檔案。這是 ASP.NET Core 的設計哲學——**照著規則做，框架自動運作，不需要額外設定。**

---

## 5. Action 的結構

```csharp
public IActionResult Index()
{
    return View();
}
```

| 部分 | 說明 |
|------|------|
| `IActionResult` | 回傳值的**型別**，代表這個 Action 會回傳一個結果 |
| `Index` | Action 的**名稱**，對應網址第二段，也對應 View 的檔案名稱 |
| `return View()` | 實際回傳的內容，觸發框架去找對應的 `.cshtml` 檔案 |

---

## 6. 今日實作步驟

1. 建立 ASP.NET Core MVC 專案 `CustomerDemo`
2. 在 `Models` 資料夾新增 `Customer.cs`，定義客戶欄位
3. 在 `Controllers` 資料夾新增 `CustomerController.cs`
4. 在 `Views` 資料夾新增 `Customer` 子資料夾
5. 在 `Views/Customer` 資料夾新增 `Index.cshtml`
6. 執行專案，瀏覽 `/Customer/Index` 確認畫面正常顯示

---

## 7. 下一步

將 Customer 的資料從 Controller 傳到 View 顯示出來。
