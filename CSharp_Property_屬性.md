# C# 筆記：Property（屬性）

---

## 為什麼需要 Property？

封裝的做法需要寫兩個 Method：

```csharp
public void SetBalance(int number) { ... }
public int GetBalance() { ... }
```

每次都要寫兩個，比較囉嗦。Property 是更簡潔的寫法，做的事情一樣，但只需要一個區塊。

---

## 完整 Property 寫法

```csharp
private int balance;

public int Balance
{
    get
    {
        return balance;
    }
    set
    {
        if (value > 0)
        {
            balance = value;
        }
    }
}
```

外面使用：

```csharp
account.Balance = 500;
Console.WriteLine(account.Balance);
```

---

## 重點說明

| 關鍵字 | 說明 |
|--------|------|
| `get` | 讀取值，用 `return` 回傳 private field |
| `set` | 設定值，用 `value` 接收外面傳進來的值 |
| `value` | C# 自動準備的關鍵字，代表外面傳進來的值，不需要自己定義 |

### 注意事項
- `get { }` 和 `set { }` 結尾只用 `}`，**不加分號**
- `get` 要 `return` 的是 **private field**，不是 Property 本身
- `set` 要設定的也是 **private field**，不是 Property 本身

---

## Auto Property（自動屬性）

當 Property **不需要加任何判斷**，可以用更短的寫法：

```csharp
public string Owner { get; set; }
```

等同於：

```csharp
private string owner;

public string Owner
{
    get { return owner; }
    set { owner = value; }
}
```

### 背後發生了什麼事？

寫完整 Property 時，private field **你來寫**，`get` / `set` **你來寫**：

```csharp
private string owner;        // 你自己宣告

public string Owner
{
    get { return owner; }    // 你自己寫
    set { owner = value; }   // 你自己寫
}
```

寫 Auto Property 時，你只寫一行：

```csharp
public string Name { get; set; }
```

你沒有寫 `private string name;`，但 C# 編譯器看到這行，會**在背後自動幫你建立** private field。你看不到它，也不需要去碰它，C# 全包了。

簡單說：
- 完整 Property → private field **你來寫**，`get` / `set` **你來寫**
- Auto Property → private field **C# 幫你寫**，`get` / `set` **C# 幫你寫**

---

## 完整 Property vs Auto Property

| | 完整 Property | Auto Property |
|---|---|---|
| **適合情況** | 需要加 `if` 判斷攔截值 | 不需要判斷，任何值都接受 |
| **需要 private field** | 要自己宣告 | 自動建立，不用寫 |
| **寫法長度** | 較長 | 一行 |

---

## 完整範例

```csharp
Student s = new Student();
s.Name = "Will";
s.Score = 85;
Console.WriteLine(s.Name);   // 印出 Will
Console.WriteLine(s.Score);  // 印出 85

s.Score = -10;
Console.WriteLine(s.Score);  // 印出 85（被攔截，沒有改變）

class Student
{
    public string Name { get; set; }  // Auto Property，任何值都接受

    private int score;

    public int Score
    {
        get
        {
            return score;
        }
        set
        {
            if (value >= 0 && value <= 100)
            {
                score = value;
            }
        }
    }
}
```

---

## 重點整理

- Property 是封裝的進化版，把 `GetXxx` / `SetXxx` 合併成一個區塊
- 外面用起來像直接存取 field（不用加 `()`），裡面還是有保護
- `value` 是 `set` 裡面 C# 自動準備的關鍵字，代表外面傳進來的值
- 需要判斷 → 完整 Property；不需要判斷 → Auto Property
- 判斷「在範圍內」用 `&&`，例如：`value >= 0 && value <= 100`
