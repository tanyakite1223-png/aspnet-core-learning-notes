# C# 筆記：Basics（基本語法）

---

## 1. Method（方法）

Method（方法）是一段可重複使用的程式碼，可以傳入資料、執行動作、回傳結果。

### 基本結構

```csharp
int Add(int a, int b)
{
    return a + b;
}
int result = Add(3, 5); // result = 8
```

### 重要概念

| 名稱 | 說明 |
|------|------|
| **parameter（參數）** | 傳進去的值，例如 `int a, int b` |
| **return value（回傳值）** | 回傳出來的結果，例如 `return a + b` |
| **void（無回傳）** | Method 沒有回傳值時，必須寫 `void` |

### void 範例

```csharp
void SayHello(string name)
{
    Console.WriteLine("Hello, " + name);
}
SayHello("Will"); // 印出：Hello, Will
```

### 回傳型別規則

| 情況 | 寫法 |
|------|------|
| 有回傳值 | 寫回傳的型別，例如 `int`、`string` |
| 沒有回傳值 | 必須寫 `void` |
| Constructor（建構子） | 什麼都不寫（連 `void` 也不用） |

---

## 2. Class（類別）

Class（類別）是一個**範本（設計圖）**，用來描述某種物件應該有什麼資料和行為。

### Class 標準結構

```csharp
class ClassName
{
    // field（欄位）...

    // Constructor（建構子）...

    // Method（方法）...
}
```

### 基本結構

```csharp
class Dog
{
    public string Name;   // field（欄位）
    public int Age;       // field（欄位）

    public void Bark()    // method（方法）
    {
        Console.WriteLine(Name + " 說：汪汪！");
    }
}
```

### 建立 Instance（實體）

```csharp
Dog myDog = new Dog();
myDog.Name = "小黑";
myDog.Age = 3;
myDog.Bark(); // 印出：小黑 說：汪汪！
```

### 重要概念

| 名稱 | 說明 |
|------|------|
| **class（類別）** | 範本、設計圖 |
| **instance（實體）** | 用 `new` 建立出來的真實物件 |
| **field（欄位）** | class 裡儲存資料的變數，跟著 instance 一直存在 |
| **method（方法）** | class 裡執行動作的函式 |

> 一個 class（類別）可以建立任意多個 instance（實體），每個 instance 的資料互不影響。

---

## 3. Constructor（建構子）

Constructor（建構子）是**建立 instance（實體）時自動執行**的特殊 method（方法）。

### 規則

- 名稱必須跟 class（類別）**完全相同**
- **不能寫任何回傳型別**（連 `void` 也不用寫）
- 使用 `new` 時自動觸發

### 基本結構

```csharp
class Dog
{
    public string Name;
    public int Age;

    public Dog(string name, int age)  // Constructor（建構子）
    {
        Name = name;  // 把 parameter（參數）存進 field（欄位）
        Age = age;
    }
}
Dog myDog = new Dog("小黑", 3); // 建立時直接傳入資料
```

### 有無 Constructor 的比較

```csharp
// 沒有 Constructor（建構子）
Dog myDog = new Dog();
myDog.Name = "小黑";
myDog.Age = 3;

// 有 Constructor（建構子）
Dog myDog = new Dog("小黑", 3); // 更簡潔，且強迫填入必要資料
```

### 沒有 Constructor 時的預設值

| 型別 | 預設值 |
|------|------|
| `string` | `null`（不是 `""`） |
| `int` | `0` |
| `bool` | `false` |

> `string` 是 reference type（參考型別），預設是 `null`。用 `+` 串接時，`null` 會被當成 `""` 處理，所以不會出錯，但內容是空的。
>
> **實驗驗證：**
> ```csharp
> class Dog
> {
>     public string Name;
>     public void Bark()
>     {
>         Console.WriteLine(Name + " 說：汪！");
>     }
> }
> Dog myDog = new Dog();
> myDog.Bark(); // 印出：「 說：汪！」（Name 是 null，串接後變空白）
> ```

### Constructor（建構子）的用途

- 建立 instance（實體）時，**強迫提供必要資料**
- 沒填入必要 parameter（參數），C# 編譯時直接擋掉
- 注意：Constructor 無法阻止傳入 `""` 或 `null`，若要防止需在內部加驗證邏輯（之後會學到）

### 設定 field（欄位）的兩種方式

```csharp
// 方式一：製造後再設定（有忘記設定的風險）
Dog myDog = new Dog();
myDog.Name = "小白";

// 方式二：透過 Constructor 傳入（推薦）
Dog myDog = new Dog("小白");
```

> **為什麼推薦方式二？**
> - 寫法較簡潔
> - 強迫使用者在建立時就填入資料，物件一出生就是完整狀態
> - 忘記填入 → 編譯就會直接報錯，不會等到執行時才發現問題

### 沒有 Constructor 時，如何正確設定 field

```csharp
// ❌ 錯誤寫法（用 class 名稱而不是 instance 變數名稱）
Dog.Name = "小白";

// ✅ 正確寫法（先建立 instance，再用變數名稱設定）
Dog myDog = new Dog();
myDog.Name = "小白";
```

---

## 練習題解答：Car Class（類別）

```csharp
class Car
{
    public string Brand;  // field（欄位）
    public int Speed;     // field（欄位）

    public Car(string brand, int speed)  // Constructor（建構子）
    {
        Brand = brand;
        Speed = speed;
    }

    public void Drive()  // method（方法），void = 不回傳值
    {
        Console.WriteLine($"{Brand}正在以{Speed}公里行駛");
    }
}

Car record = new Car("Toyota", 120);
record.Drive(); // 印出：Toyota 正在以120公里行駛
```

---

## 注意事項

- C# 大小寫敏感：`Console` ✅　`console` ❌
- `class` 小寫 ✅　`Class` ❌
- Constructor（建構子）的 parameter（參數）執行完就消失，資料要存進 field（欄位）才能被其他 method（方法）使用
- `string` 預設值是 `null`，不是 `""`
- 設定 field 時，要用 **instance 的變數名稱**（例如 `myDog.Name`），不能用 class 名稱（`Dog.Name` 是錯的）
