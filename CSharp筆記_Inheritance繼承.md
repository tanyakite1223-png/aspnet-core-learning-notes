# C# 筆記：Inheritance（繼承）

## 什麼是 Inheritance（繼承）？

當多個 Class 有**共同的 Property 和 Method** 時，把共同的部分抽出來放到一個 **Parent Class（父類別）**，讓其他 Class 去繼承，避免重複寫相同的程式碼。

---

## 基本觀念

| 名稱 | 說明 |
|------|------|
| Parent Class（父類別） | 又叫 Base Class（基底類別），放共同的東西，較通用 |
| Child Class（子類別） | 又叫 Derived Class（衍生類別），繼承 Parent，較特定 |

**判斷誰是 Parent：「誰比較大、比較通用？」→ 那個就是 Parent Class**

- `Animal`（動物）→ Parent Class
- `Dog`（狗）→ Child Class

---

## 繼承語法：用 `:` 冒號

```csharp
class Animal {
    public string? Name { get; set; }
    public int Age { get; set; }

    public void Eat() { }
}

// Dog 繼承 Animal，用 : 表示
class Dog : Animal { }
```

`Dog` 雖然裡面是空的，但因為繼承了 `Animal`，自動擁有 `Name`、`Age`、`Eat()`。

---

## 子類別可以加自己獨有的東西

```csharp
class Dog : Animal {
    public string Bark() {
        return $"{Name}：汪汪！";
    }
}

class Cat : Animal {
    public string Meow() {
        return $"{Name}：喵喵！";
    }
}
```

- `Bark()` 只有 `Dog` 能用，`Cat` 不能用
- `Meow()` 只有 `Cat` 能用，`Dog` 不能用
- **子類別之間互相獨立，不能使用對方的東西**

---

## Constructor 與繼承：`base` 關鍵字

當 Parent Class 有 Constructor 時，Child Class 必須用 **`base`** 把參數傳給 Parent 的 Constructor。

```csharp
class Animal {
    public string? Name { get; set; }
    public int Age { get; set; }

    public Animal(string name, int age) {
        Name = name;
        Age = age;
    }
}

class Dog : Animal {
    // 用 : base(name, age) 把參數傳給 Animal 的 Constructor
    public Dog(string name, int age) : base(name, age) { }
}
```

### 建立 Instance

```csharp
Dog dog = new Dog("小白", 2);
Console.WriteLine(dog.Name);   // 小白
Console.WriteLine(dog.Age);    // 2
```

---

## Constructor 執行順序：先 Parent，再 Child

```csharp
public Animal(string name, int age) {
    Console.WriteLine("Animal Constructor 執行了");  // ← 先執行
    Name = name;
    Age = age;
}

public Dog(string name, int age) : base(name, age) {
    Console.WriteLine("Dog Constructor 執行了");     // ← 後執行
}
```

輸出結果：
```
Animal Constructor 執行了
Dog Constructor 執行了
```

**要先有「動物」的基本資料，才能再處理「狗」獨有的東西。**

---

## 完整練習範例：Vehicle 繼承架構

```csharp
Car car = new Car("TOYOTA", 2020);
Motorcycle motorcycle = new Motorcycle("光陽", 1998);

Console.WriteLine(car.Start());          // 引擎啟動！
Console.WriteLine(car.OpenTrunk());      // TOYOTA 後車廂已開啟！
Console.WriteLine(motorcycle.Start());   // 引擎啟動！
Console.WriteLine(motorcycle.Wheelie()); // 光陽 翹孤輪！

class Vehicle {
    public string Brand { get; set; }
    public int Year { get; set; }

    public string Start() {
        return "引擎啟動！";
    }

    public Vehicle(string brand, int year) {
        Brand = brand;
        Year = year;
    }
}

class Car : Vehicle {
    public string OpenTrunk() {
        return $"{Brand} 後車廂已開啟！";
    }

    public Car(string brand, int year) : base(brand, year) { }
}

class Motorcycle : Vehicle {
    public string Wheelie() {
        return $"{Brand} 翹孤輪！";
    }

    public Motorcycle(string brand, int year) : base(brand, year) { }
}
```

---

## 重點整理

1. **共同的東西**放在 Parent Class，子類別用 `:` 繼承
2. **子類別自動擁有** Parent 的 Property 和 Method
3. **子類別可以加自己獨有的東西**（如 `Bark()`、`Meow()`）
4. **子類別之間互相獨立**，不能用對方的東西
5. Parent 有 Constructor 時，子類別用 **`base()`** 把參數傳給 Parent
6. **Constructor 執行順序**：先 Parent，再 Child

---

## 下一個主題

**Polymorphism（多型）** — 與 Inheritance 密切相關的 OOP 重要觀念
