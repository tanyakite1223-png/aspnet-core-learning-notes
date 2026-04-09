# C# 筆記：Encapsulation（封裝）

---

## 為什麼需要封裝？

如果 field 是 `public`，外面任何人都可以直接設定任意值，完全沒有保護：

```csharp
BankAccount account = new BankAccount();
account.balance = -99999;  // 沒有任何攔截，直接接受
```

這會造成資料不合理的問題（例如銀行餘額變成負數）。

---

## 封裝的做法

封裝就是兩個動作：

1. 把 field 改成 `private`（外面看不到、碰不到）
2. 在 class 裡面提供專門的 Method 讓外面存取

---

## private 的效果

```csharp
class BankAccount
{
    private int balance;  // 外面碰不到
}
```

如果外面直接存取，會出現錯誤：

```
CS0122: 'BankAccount.balance' 由於其保護層級之故，所以無法存取
```

---

## SetXxx — 設定值的 Method

- 回傳型別用 `void`（不需要回傳任何東西）
- 在裡面加 `if` 判斷來攔截不合理的值

```csharp
public void SetBalance(int number)
{
    if (number > 0)
    {
        balance = number;
    }
}
```

---

## GetXxx — 讀取值的 Method

- 回傳型別要跟 field 一樣（例如 `int`）
- 用 `return` 把值回傳給外面

```csharp
public int GetBalance()
{
    return balance;
}
```

---

## 完整範例

```csharp
BankAccount account = new BankAccount();
account.SetBalance(500);
Console.WriteLine(account.GetBalance());  // 印出 500

class BankAccount
{
    private int balance;

    public void SetBalance(int number)
    {
        if (number > 0)
        {
            balance = number;
        }
    }

    public int GetBalance()
    {
        return balance;
    }
}
```

---

## 練習題：Student 分數

```csharp
Student s = new Student();
s.SetScore(85);   // ✅ 成功存入
s.SetScore(-10);  // ❌ 被攔截
s.SetScore(150);  // ❌ 被攔截
Console.WriteLine(s.GetScore());

class Student
{
    private int score;

    public void SetScore(int number)
    {
        if (0 <= number && number <= 100)
        {
            score = number;
        }
    }

    public int GetScore()
    {
        return score;
    }
}
```

---

## 重點整理

| 關鍵字 | 用途 |
|--------|------|
| `private` | field 不讓外面直接存取 |
| `public void SetXxx` | 外面透過這個 Method 設定值，內部可以加判斷 |
| `public int GetXxx` | 外面透過這個 Method 讀取值 |
| `&&` | 「而且」，兩個條件都要成立 |

---

## 注意事項

- C# 關鍵字全部小寫：`private`、`public`、`void`、`return`
- `SetXxx` 不需要回傳值，用 `void`
- `GetXxx` 需要回傳值，回傳型別要跟 field 一樣
- 判斷「在範圍內」用 `&&`，例如：`0 <= number && number <= 100`
