# C# 筆記：Polymorphism（多型）

## 什麼是 Polymorphism（多型）？

同一個 Method，根據實際物件不同，自動執行不同的行為。
不需要用 `if` 或 `switch` 判斷型別，C# 會自動找到正確的版本。

---

## 核心語法

### Parent Class 用 `virtual`（虛擬的）

表示「這個 Method **可以**被子類別覆寫」：

```csharp
class Animal
{
    public virtual string MakeSound()
    {
        return "（未知的聲音）";  // 預設版本
    }
}
```

### Child Class 用 `override`（覆寫）

覆寫 Parent 的 `virtual` Method，定義自己的行為：

```csharp
class Dog : Animal
{
    public override string MakeSound()
    {
        return "汪汪！";
    }
}

class Cat : Animal
{
    public override string MakeSound()
    {
        return "喵喵！";
    }
}
```

---

## 重要觀念：C# 看的是「實際物件」

```csharp
Animal myAnimal = new Dog("小白", 2);
Console.WriteLine(myAnimal.MakeSound());  // 印出「汪汪！」
```

- 左邊型別是 `Animal`，但右邊 `new` 的是 `Dog`
- C# 看的是**實際建立的物件（右邊）**，不是變數的型別（左邊）
- 所以會執行 `Dog` 的 `override` 版本

---

## 沒有 override 的情況

如果子類別**沒有寫** `override`，就會用 Parent 的預設版本：

```csharp
// Dog 沒有 override MakeSound()
Dog dog = new Dog("小白", 2);
Console.WriteLine(dog.MakeSound());  // 印出「（未知的聲音）」
```

`virtual` 的意思是「**可以**被覆寫」，不是「**一定要**覆寫」。

---

## 多型最實用的地方：陣列 + foreach

```csharp
Animal[] animals = new Animal[]
{
    new Dog("小白", 2),
    new Cat("起司", 1),
    new Dog("大黃", 3)
};

foreach (Animal a in animals)
{
    Console.WriteLine(a.MakeSound());
}
```

輸出：
```
汪汪！
喵喵！
汪汪！
```

不用寫 `if` 或 `switch` 判斷是 Dog 還是 Cat，C# 自動根據實際物件呼叫正確的 `override` 版本。

---

## 練習題：Shape（形狀）

### 常見錯誤

**1. `base()` 傳了多餘的參數**

```csharp
// ❌ 錯誤：Shape 的 Constructor 只接收 name，不能傳 radius
public Circle(string name, double radius) : base(name, radius) { }

// ✅ 正確：base() 只傳 name，radius 自己存
public Circle(string name, double radius) : base(name)
{
    Radius = radius;
}
```

重點：`base()` 是呼叫 Parent 的 Constructor，要看 Parent 接收幾個參數。子類別自己的參數（如 `radius`）要在 `{}` 裡面自己存起來。

**2. 子類別忘記寫 Constructor**

```csharp
// ❌ 錯誤：Rectangle 沒有 Constructor，無法接收 width 和 height
class Rectangle : Shape
{
    public double Width { get; set; }
    public double Height { get; set; }

    public override double GetArea()
    {
        return Width * Height;
    }
}

// ✅ 正確：要寫 Constructor，用 base(name) 傳給 Parent，自己存 Width 和 Height
public Rectangle(string name, double width, double height) : base(name)
{
    Width = width;
    Height = height;
}
```

**3. 建立 Instance 時參數數量不對**

```csharp
// ❌ 錯誤：Rectangle 需要 name、width、height 三個參數
new Rectangle("Rectangle", 6)

// ✅ 正確：傳入三個參數
new Rectangle("Rectangle", 4, 6)
```

---

### 正確完整版

```csharp
Shape[] shape = new Shape[]
{
    new Circle("Circle", 3),
    new Rectangle("Rectangle", 4, 6)
};

foreach (var item in shape)
{
    Console.WriteLine($"{item.Name}的面積是{item.GetArea()}");
}

class Shape
{
    public string Name { get; set; }

    public Shape(string name)
    {
        Name = name;
    }

    public virtual double GetArea()
    {
        return 0;
    }
}

class Circle : Shape
{
    public double Radius { get; set; }

    public Circle(string name, double radius) : base(name)
    {
        Radius = radius;
    }

    public override double GetArea()
    {
        return 3.14 * Radius * Radius;
    }
}

class Rectangle : Shape
{
    public double Width { get; set; }
    public double Height { get; set; }

    public Rectangle(string name, double width, double height) : base(name)
    {
        Width = width;
        Height = height;
    }

    public override double GetArea()
    {
        return Width * Height;
    }
}
```

輸出：
```
Circle的面積是28.26
Rectangle的面積是24
```

---

## 重點整理

| 關鍵字 | 寫在哪裡 | 意思 |
|--------|----------|------|
| `virtual` | Parent Class | 這個 Method 可以被子類別覆寫 |
| `override` | Child Class | 覆寫 Parent 的 virtual Method |

1. **`virtual`** — Parent Class 定義可覆寫的 Method
2. **`override`** — Child Class 覆寫，決定自己的行為
3. **沒有 override** → 用 Parent 的預設版本
4. **多型核心** → C# 看的是實際建立的物件（右邊），不是變數型別（左邊）
5. **最大好處** → 陣列 + foreach，不用 if/switch，自動執行正確的版本

---

## 下一個主題

**Abstract Class（抽象類別）** — 跟 `virtual` 有關聯，但規則更嚴格。
