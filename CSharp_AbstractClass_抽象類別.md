# C# 筆記：Abstract Class（抽象類別）

---

## 一、為什麼需要 Abstract Class？

有些類別（如 `Animal`）只是用來「定義共同規範」，不應該被直接建立物件。

例如：現實中不存在一種動物就叫做「Animal」，只有具體的 Dog、Cat、Bird。

**Abstract Class 解決兩個問題：**

1. **禁止直接 `new` 父類別** → `new Animal()` 會編譯錯誤
2. **強制子類別實作指定方法** → 子類別不寫就編譯失敗（錯誤代碼 CS0534）

---

## 二、基本語法

### 宣告 Abstract Class

```csharp
abstract class Animal
{
    public abstract string MakeSound();  // 沒有方法主體，用分號結尾
}
```

重點：
- class 前面加 `abstract`
- abstract 方法**沒有 `{ }`**，直接用 `;` 結尾
- abstract 方法必須是 `public`（或 `protected`），不能是 `private`

### 子類別實作 Abstract 方法

```csharp
class Dog : Animal
{
    public override string MakeSound()  // 必須加 override
    {
        return "汪汪";
    }
}
```

重點：
- 子類別用 **`override`** 關鍵字來實作父類別的 abstract 方法
- 回傳型別必須跟父類別一致（父類別是 `string`，子類別也要是 `string`）
- **不寫 override 會編譯錯誤**（CS0534）
- 子類別**不需要**加 `abstract`，只有父類別需要

### 繼承語法提醒

繼承和 Constructor 是分開寫的，不要混在一起：

```csharp
// ❌ 錯誤
class Dog : Animal(string name) { }

// ✅ 正確
class Dog : Animal
{
    public Dog(string name)
    {
        Name = name;
    }
}
```

- **`: Animal`** → 宣告繼承關係（我是誰的子類別）
- **`public Dog(string name)`** → Constructor，負責接收參數和初始化

---

## 三、完整範例

```csharp
// 抽象類別 —— 不能被 new
abstract class Animal
{
    public abstract string MakeSound();
}

// 子類別 —— 必須實作 MakeSound()
class Dog : Animal
{
    public override string MakeSound()
    {
        return "汪汪";
    }
}

class Cat : Animal
{
    public override string MakeSound()
    {
        return "喵喵";
    }
}

class Bird : Animal
{
    public override string MakeSound()
    {
        return "啾啾";
    }
}
```

### 測試：用陣列 + foreach

```csharp
Animal[] animals = new Animal[]
{
    new Dog(),
    new Cat(),
    new Bird()
};

foreach (var item in animals)
{
    Console.WriteLine(item.MakeSound());
}

// 輸出：
// 汪汪
// 喵喵
// 啾啾
```

---

## 四、Polymorphism（多型）

Abstract Class 是多型的重要基礎。

```csharp
Animal a1 = new Dog();
Animal a2 = new Cat();

a1.MakeSound();  // 執行 Dog 的 MakeSound() → 汪汪
a2.MakeSound();  // 執行 Cat 的 MakeSound() → 喵喵
```

雖然變數型別都是 `Animal`，但 C# 會根據**物件實際的型別**來決定執行哪個版本的方法。

### 多型的好處：用父類別當參數型別

```csharp
public void LetAnimalSpeak(Animal animal)
{
    Console.WriteLine(animal.MakeSound());
}
```

不管傳入 Dog、Cat 還是 Bird，一個方法就能處理所有子類別。

---

## 五、Abstract Class 可以有一般方法

Abstract class 裡面可以同時有 abstract 方法和一般方法：

```csharp
abstract class Animal
{
    public abstract string MakeSound();  // 子類別「必須」override

    public void Sleep()                   // 子類別「自動繼承」，不用寫就能用
    {
        Console.WriteLine("Zzz...");
    }
}
```

- **abstract 方法** → 子類別**必須** override
- **一般方法** → 子類別**直接繼承**，不需要寫任何東西就能用

```csharp
Dog d = new Dog();
d.Sleep();  // 直接印出 "Zzz..."，Dog 裡面不用寫任何東西
```

---

## 六、void vs return 方法的呼叫方式

### `void` 方法 — 直接呼叫

```csharp
// 方法定義：void，內部自己印
public void ShowInfo()
{
    Console.WriteLine("這是一種交通工具");  // 印的動作在方法裡面
}
```

```csharp
item.ShowInfo();  // ✅ 正確，直接呼叫就會印出來

Console.WriteLine(item.ShowInfo());  // ❌ 錯誤！void 沒有回傳值，WriteLine 沒東西可以接
```

### `return` 方法 — 外面要自己處理

```csharp
// 方法定義：string，只回傳資料
public override string Move()
{
    return "Car在公路上行駛";  // 只回傳，不印
}
```

```csharp
Console.WriteLine(item.Move());  // ✅ 正確，用 WriteLine 把回傳值印出來

string result = item.Move();     // ✅ 也可以先存起來再用
Console.WriteLine(result);

item.Move();  // ⚠️ 不會報錯，但回傳值沒人接，等於白做了
```

### 兩種方法在 foreach 裡的正確寫法

```csharp
foreach (var item in vehicle)
{
    item.ShowInfo();                      // void → 直接呼叫
    Console.WriteLine(item.Move());       // string → 用 Console.WriteLine() 接
}
```

### 速查表

| 方法類型 | 呼叫方式 | 原因 |
|---------|---------|------|
| `void` 方法 | `item.ShowInfo();` | 方法內部自己印，直接呼叫就好 |
| `string` 方法 | `Console.WriteLine(item.Move());` | 方法只回傳值，外面要自己決定怎麼用 |

### 方法設計建議

建議用 **`return`** 的方式，原因：
1. 回傳值可以印出來、存起來、組合字串、傳給其他方法，更靈活
2. 符合「職責分離」原則：方法負責處理資料，呼叫者決定如何呈現
3. 未來寫 ASP.NET Core 時，資料是顯示在網頁上而非 Console

**一句話總結：`void` 自己做完了，`return` 只給你結果、你自己處理。**

---

## 七、練習題：交通工具系統

```csharp
Vehicle[] vehicle = new Vehicle[]
{
    new Car(),
    new Boat(),
    new Airplane()
};

foreach (var item in vehicle)
{
    item.ShowInfo();
    Console.WriteLine(item.Move());
}

abstract class Vehicle
{
    public abstract string Move();

    public void ShowInfo()
    {
        Console.WriteLine("這是一種交通工具");
    }
}

class Car : Vehicle
{
    public override string Move()
    {
        return "Car在公路上行駛";
    }
}

class Boat : Vehicle
{
    public override string Move()
    {
        return "Boat在水上航行";
    }
}

class Airplane : Vehicle
{
    public override string Move()
    {
        return "Airplane在天空飛行";
    }
}

// 輸出：
// 這是一種交通工具
// Car在公路上行駛
// 這是一種交通工具
// Boat在水上航行
// 這是一種交通工具
// Airplane在天空飛行
```

---

## 八、常見錯誤整理

| 錯誤 | 原因 |
|------|------|
| `new Animal()` 報錯 | abstract class 不能建立物件 |
| 子類別沒寫 `MakeSound()` 報 CS0534 | 必須實作所有 abstract 方法 |
| 子類別忘記加 `override` | 覆寫 abstract 方法必須加 `override` |
| 子類別寫 `abstract class Dog : Animal` | Dog 不是抽象類別，不需要加 `abstract` |
| `Console.WriteLine(item.ShowInfo())` 報錯 | `ShowInfo()` 是 `void`，沒有回傳值不能放進 `WriteLine()` |
| 繼承語法寫成 `class Dog : Animal(string name)` | 繼承和 Constructor 要分開寫 |

---

## 九、關鍵字速查

| 關鍵字 | 用途 |
|--------|------|
| `abstract class` | 宣告抽象類別，不能被 `new` |
| `abstract`（方法） | 宣告抽象方法，沒有實作內容，子類別必須 override |
| `override` | 子類別實作或覆寫父類別的 abstract / virtual 方法 |
| `void` | 方法不回傳值，內部自己處理 |
| `return` | 方法回傳值，讓呼叫者決定怎麼用 |
