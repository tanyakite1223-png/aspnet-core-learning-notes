# C# 筆記：Constructor 與 Property 綜合運用

---

## Field 和 Property 的比較

| 比較 | Field | Property |
|------|-------|----------|
| 寫法 | `public string Name;` | `public string Name { get; set; }` |
| 存取控制 | ❌ 無 | ✅ 有（可限制讀/寫、加驗證） |
| 建議使用 | 少用 | ✅ 優先使用 |

### 大小寫慣例

| 種類 | 大小寫 | 範例 |
|---|---|---|
| Field（欄位） | 小寫開頭 | `private int year` |
| Property（屬性） | 大寫開頭 | `public int Year` |
| 參數 | 小寫開頭 | `string brand, int year` |
| Auto Property | 大寫開頭 | `public string Brand { get; set; }` |

**規則只有一個：Property 大寫，其他都小寫。**

---

## 為什麼 Field 和 Property 不能同名？

```csharp
private int Year;   // Field

public int Year     // ❌ 報錯！同一個 class 不能有兩個東西同名
{
    get { return Year; }
}
```

解決方法：Field 用小寫，Property 用大寫，名字就不衝突了。

```csharp
private int year;   // ✅ Field 小寫

public int Year     // ✅ Property 大寫
{
    get { return year; }
}
```

---

## Constructor（建構子）規則複習

1. **名稱與 Class 相同**
2. **不加 `void`**
3. **用 `new` 觸發**
4. **參數放在 Constructor 的括號內**，不是 Class 名稱後面

```csharp
// ❌ 錯誤：參數放在 class 上
class Product(string name, decimal price) { }

// ✅ 正確：參數放在 Constructor 上
class Product
{
    public Product(string name, decimal price) { }
}
```

---

## `this` 關鍵字

`this` 代表「這個物件本身」，用來明確指向 class 自己的 Field 或 Property。

### 使用時機

當參數名稱和 Field / Property 名稱相同時，用 `this` 區分：

```csharp
public Car(string brand, int year)
{
    this.year = year;   // this.year → Field；year → 參數
}
```

---

## 完整 Property vs Auto Property

| | 寫法 | 適合情況 |
|---|---|---|
| **Auto Property** | `{ get; set; }` | 不需要驗證，任何值都接受 |
| **完整 Property** | `get { } set { }` | 需要加驗證邏輯 |

---

## Constructor 搭配 Property 基本範例

```csharp
class Product
{
    // Property（屬性）
    public string Name { get; set; }
    public decimal Price { get; set; }

    // Constructor（建構子）
    public Product(string name, decimal price)
    {
        Name = name;      // 將參數值指派給 Property
        Price = price;
    }
}
```

### 建立實體（Instance）

```csharp
var p = new Product("手機", 9999m);
Console.WriteLine(p.Name);  // 輸出：手機
```

### 運作流程

```
new Product("手機", 9999m)
        ↓
Constructor 接收參數 name = "手機", price = 9999
        ↓
Name = name  →  Name Property 存入 "手機"
Price = price →  Price Property 存入 9999
        ↓
實體 p 建立完成
```

---

## 完整範例：Car 類別（含驗證）

```csharp
var car = new Car("Toyota", 2008);
Console.WriteLine(car.Brand);   // Toyota
Console.WriteLine(car.Year);    // 2008

class Car
{
    // Auto Property：Brand 不需要驗證
    public string Brand { get; set; }

    // Private Field：配合完整 Property 使用
    private int year;

    // 完整 Property：Year 需要驗證（只接受 2000 以後）
    public int Year
    {
        get { return year; }        // 回傳 Field

        set
        {
            if (value >= 2000)      // 驗證邏輯
            {
                year = value;       // 存進 Field
            }
        }
    }

    // Constructor：接收參數，指派給 Property / Field
    public Car(string brand, int year)
    {
        Brand = brand;          // 直接指派給 Auto Property
        this.year = year;       // this.year = Field；year = 參數
    }
}
```

### 程式流程說明

```
new Car("Toyota", 2008)
        ↓
呼叫 Constructor
        ↓
Brand = "Toyota"（Auto Property）
this.year = 2008（Private Field）
        ↓
car.Year 透過 get 回傳 year
        ↓
印出 2008
```

---

## 什麼時候用 `private` Field？

**判斷只有一個問題：「這個資料需要驗證嗎？」**

```
需要驗證 → 完整 Property → 需要 private Field
不需要驗證 → Auto Property → 不需要 private Field
```

### `private` 的用意

`private` = 外面看不到，強迫外面只能透過 Property 來存取資料。

```csharp
// ❌ 如果 Field 是 public，外面可以繞過驗證直接改
car.year = -999;   // 驗證邏輯白寫了！

// ✅ Field 是 private，外面只能走 Property 這條路
car.year = -999;   // 報錯，外面碰不到
car.Year = -999;   // 只能這樣，會經過驗證被攔截
```

---

## 實務上怎麼選擇寫法？

**從 Auto Property 開始，需要驗證時再改成完整 Property。**

| 情況 | 寫法 |
|---|---|
| 不需要驗證 | Auto Property `{ get; set; }` |
| 需要驗證 | 完整 Property + `private` Field |
| 直接存資料（不需要保護） | Field（現代 C# 很少用） |

**Field 在現代 C# 幾乎很少直接用，大部分都用 Property 取代。**

---

## 常見錯誤整理

| 錯誤 | 說明 |
|------|------|
| `public void Product(...)` | Constructor 不能有 `void` |
| `class Product(string name, ...)` | 參數不能放在 Class 名稱後面 |
| `public decimal Price;` | 這是 Field，應改為 Property |
| `deciaml` | `decimal` 的拼字錯誤 |

---

## LINQPad vs Visual Studio 差異

| | LINQPad | Visual Studio |
|--|---------|---------------|
| 輸出值 | `p.Name.Dump()` | `Console.WriteLine(p.Name)` |
| 模式設定 | 需選 **C# Program** | 不需要 |
| 進入點 | `void Main()` | 現代寫法不需要 `Main()` |

---

## 重點整理

- **Field** 配合完整 Property 使用，負責實際存資料
- **Property** 負責控制存取，可以加驗證邏輯
- **Auto Property** 背後 C# 自動建立 Field，不需要自己寫
- **`this`** 用來區分「物件的 Field」和「參數」
- **Constructor** 讓外面建立實體時可以直接傳值進去
- **`private`** 的用意：強迫外面只能透過 Property 存取，驗證邏輯才有意義
