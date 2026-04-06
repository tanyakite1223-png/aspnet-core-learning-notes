# C# 筆記：Interface（介面）

---

## 一、為什麼需要 Interface？

C# 只允許**單一繼承**（一個 class 只能繼承一個父類別）。

當不同類別已經有各自的父類別，但又需要共同的能力時，就需要 Interface。

例如：
- `Bird` 繼承 `Animal`
- `Airplane` 繼承 `Vehicle`
- 兩者都需要 `Fly()` 的能力 → 用 Interface `IFlyable`

---

## 二、Interface 是什麼？

> Interface 是一種**契約（Contract）**，規定類別必須實作哪些方法，但不提供實作內容。

---

## 三、基本語法

### 定義 Interface

```csharp
interface IFlyable
{
    void Fly();  // 不需要 public，不需要 abstract，沒有方法主體
}
```

### 實作 Interface

```csharp
class Bird : IFlyable
{
    public void Fly()  // 不需要 override
    {
        Console.WriteLine("鳥在飛");
    }
}
```

### 同時實作多個 Interface

```csharp
class Bird : IFlyable, ISwimmable
{
    public void Fly()
    {
        Console.WriteLine("鳥在飛");
    }

    public void Swim()
    {
        Console.WriteLine("牠會游泳");
    }
}
```

### 同時繼承 Abstract Class + 實作 Interface

```csharp
class Bird : Animal, IFlyable, ISwimmable
{
    // 繼承的 abstract method → 用 override
    public override void MakeSound()
    {
        Console.WriteLine("啾啾啾");
    }

    // 實作 Interface → 不用 override
    public void Fly()
    {
        Console.WriteLine("鳥在飛");
    }

    public void Swim()
    {
        Console.WriteLine("牠會游泳");
    }
}
```

> ⚠️ 注意順序：繼承的 class 寫在前面，Interface 寫在後面，用逗號隔開。

---

## 四、命名慣例（Naming Convention）

Interface 名稱前面加上大寫 **`I`**，讓人一眼就知道這是 Interface。

例如：`IFlyable`、`ISwimmable`、`IPrintable`

---

## 五、Interface vs Abstract Class 比較

| | Abstract Class | Interface |
|---|---|---|
| 關係 | 是什麼（is-a） | 能做什麼（can-do） |
| 繼承/實作數量 | 只能繼承 **1 個** | 可以實作 **多個** |
| 方法內容 | 可以有實作的方法 | 只有定義，沒有實作 |
| 實作時的關鍵字 | `override` | 不需要 `override` |
| 命名慣例 | 無特殊前綴 | 前面加大寫 `I` |

**判斷原則：**
- 描述「**你是什麼**」→ 用 Abstract Class
- 描述「**你能做什麼**」→ 用 Interface

---

## 六、Interface 搭配多型

Interface 可以當作型別來使用，實現多型：

```csharp
IFlyable[] flyable = new IFlyable[] { new Bird(), new Airplane() };
foreach (var item in flyable)
{
    item.Fly();  // 每個物件各自執行自己的 Fly()
}
```

> 不管是 `Bird` 還是 `Airplane`，只要實作了 `IFlyable`，就可以放進同一個 `IFlyable[]` 陣列。

---

## 七、型別判斷與轉型（Casting）

在 `IFlyable[]` 裡面，`item` 只認識 `Fly()`。如果要呼叫其他方法，需要判斷型別並轉型：

### 基本寫法

```csharp
if (item is Bird)
{
    Bird bird = (Bird)item;
    bird.Swim();
    bird.MakeSound();
}
```

### 簡潔寫法（Pattern Matching 模式匹配）

```csharp
if (item is Bird bird)
{
    bird.Swim();
    bird.MakeSound();
}
```

> 這一行同時做了：判斷型別 → 轉型 → 存到變數。

---

## 八、常見錯誤

### ❌ Interface 關鍵字大寫
```csharp
public Interface IFlyable { }  // 錯！Interface 要小寫
```

### ✅ 正確寫法
```csharp
interface IFlyable { }
```

---

### ❌ Interface 裡面的方法加 `public`
```csharp
interface IFlyable
{
    public void Fly();  // 不需要 public
}
```

### ✅ 正確寫法
```csharp
interface IFlyable
{
    void Fly();
}
```

---

### ❌ 直接 new Interface
```csharp
IFlyable flyable = new IFlyable();  // 錯！Interface 不能被實體化
```

### ✅ 正確寫法
```csharp
IFlyable flyable = new Bird();  // new 的是實作 Interface 的 class
```

---

### ❌ 實作 Interface 的方法加 `override`
```csharp
class Bird : IFlyable
{
    public override void Fly() { }  // 錯！override 是給 abstract/virtual 用的
}
```

### ✅ 正確寫法
```csharp
class Bird : IFlyable
{
    public void Fly() { }  // 不需要 override
}
```
