# C# 筆記：Static（靜態）

## 核心觀念

- 加了 `static` 的成員（方法、屬性），**屬於類別本身**，不屬於任何物件
- 使用時用**類別名稱**呼叫，不需要 `new`
- 一般成員屬於物件，`static` 成員屬於類別

| | 一般成員 | static 成員 |
|---|---|---|
| 屬於 | 物件（Object） | 類別（Class）本身 |
| 使用方式 | 先 `new`，再用物件呼叫 | 直接用類別名稱呼叫 |

---

## Static Method（靜態方法）

適合用在**不需要依賴物件資料（物件的屬性值）**的方法——只靠傳入的參數就能完成工作。

```csharp
class Calculator
{
    public static int Add(int x, int y)
    {
        return x + y;
    }
}

// 呼叫方式：直接用類別名稱
Console.WriteLine(Calculator.Add(4, 8));  // 12
```

**判斷標準：** 這個方法需不需要用到物件的屬性值？不需要 → 適合 `static`。

---

## Static Property（靜態屬性）

適合用在**所有物件共用**的資料，例如「總數量」。

```csharp
class Dog
{
    public string Name { get; set; }          // 一般屬性：每隻狗各自的
    public static int Count { get; set; }     // 靜態屬性：所有狗共用的

    public Dog(string name)
    {
        Name = name;
        Count += 1;   // 每次 new 就自動加 1
    }
}

// 使用
Dog[] dogs = new Dog[] { new Dog("小花"), new Dog("小豹"), new Dog("小喵") };
Console.WriteLine(Dog.Count);  // 3
```

---

## 重要規則

### 1. static 成員只能用類別名稱呼叫，不能用物件呼叫

```csharp
// ✅ 正確
Dog.Count

// ❌ 錯誤：不能用物件呼叫 static 成員
dog1.Count
```

### 2. static 方法裡面不能使用一般的屬性或欄位

```csharp
class Dog
{
    public string Name { get; set; }
    public static int Count { get; set; }

    public static void ShowCount()
    {
        Console.WriteLine(Name);   // ❌ 錯誤：static 方法不知道要讀哪個物件的 Name
        Console.WriteLine(Count);  // ✅ 正確：Count 也是 static
    }
}
```

**原因：** `static` 方法屬於類別，不屬於任何物件，所以它無法知道要讀「哪一個物件」的資料。

---

## 補充：`is` 判斷 + 轉型

當你有一個父類別陣列（例如 `Animal[]`），想判斷裡面的物件有沒有實作某個 Interface，可以用 `is`：

```csharp
if (item is IAdoptable adoptable)
{
    Console.WriteLine(adoptable.GetAdoptInfo());
}
```

這一行做了兩件事：
1. 判斷 `item` 有沒有實作 `IAdoptable`
2. 如果有，自動轉型並存到 `adoptable` 變數裡（變數名稱可以自己取）

---

## 常見錯誤

### ❌ 方法宣告忘記寫回傳型別

```csharp
public static Add(int x, int y)  // ❌ 少了 int
```

### ✅ 正確寫法

```csharp
public static int Add(int x, int y)  // ✅ public static 回傳型別 方法名稱
```

### ❌ abstract 方法寫了方法體

```csharp
public abstract string Speak()    // ❌ abstract 方法不能有 { }
{
    return "未知聲音";
}
```

### ✅ 正確寫法

```csharp
public abstract string Speak();   // ✅ 只寫簽名，用分號結尾
```

### ❌ Constructor 名稱寫成屬性名稱

```csharp
public Name(string name) { ... }  // ❌ Constructor 名稱必須跟類別名稱一樣
```

### ✅ 正確寫法

```csharp
public Dog(string name) { ... }   // ✅ 類別叫 Dog，Constructor 就叫 Dog
```

---

## 常見的 static 範例

`Console.WriteLine()` —— `WriteLine` 就是 static method，所以可以直接用 `Console.` 呼叫，不需要 `new Console()`。

---

## 綜合練習：寵物收容所

結合 Abstract Class、Interface、Polymorphism、Static 的完整範例：

```csharp
Animal[] animals = new Animal[] { new Dog("小白"), new Dog("小黑"), new Cat("咪咪") };

foreach (var item in animals)
{
    Console.WriteLine(item.Speak());

    if (item is IAdoptable adoptable)
    {
        Console.WriteLine(adoptable.GetAdoptInfo());
    }
}

Console.WriteLine($"Dog 總數：{Dog.Count}");
Console.WriteLine($"Cat 總數：{Cat.Count}");

abstract class Animal
{
    public string? Name { get; set; }

    public abstract string Speak();
}

interface IAdoptable
{
    string GetAdoptInfo();
}

class Dog : Animal, IAdoptable
{
    public static int Count { get; set; }

    public Dog(string name)
    {
        Name = name;
        Count += 1;
    }

    public override string Speak()
    {
        return "汪汪！";
    }

    public string GetAdoptInfo()
    {
        return $"我是{Name}，歡迎領養我！";
    }
}

class Cat : Animal, IAdoptable
{
    public static int Count { get; set; }

    public Cat(string name)
    {
        Name = name;
        Count += 1;
    }

    public override string Speak()
    {
        return "喵喵～";
    }

    public string GetAdoptInfo()
    {
        return $"我是{Name}，歡迎領養我！";
    }
}
```

**輸出結果：**
```
汪汪！
我是小白，歡迎領養我！
汪汪！
我是小黑，歡迎領養我！
喵喵～
我是咪咪，歡迎領養我！
Dog 總數：2
Cat 總數：1
```
