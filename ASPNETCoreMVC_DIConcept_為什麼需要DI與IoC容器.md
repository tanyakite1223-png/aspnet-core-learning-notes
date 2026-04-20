# ASP.NET Core MVC 筆記:DI 概念 — 為什麼需要 DI、IoC 容器

## 一、問題的起點:Controller 自己 `new` 依賴會怎樣?

一個看起來很直覺的寫法:

```csharp
public class OrderController : Controller
{
    private EmailService _emailService;

    public OrderController()
    {
        _emailService = new EmailService();  // 自己 new
    }

    public IActionResult PlaceOrder()
    {
        _emailService.SendEmail("通知訂單建立");
        return View();
    }
}
```

這段程式**能跑**,但會帶來三個問題:

### 問題 1:加新功能要改舊程式
當老闆說「要加 SMS 通知、LINE 通知、Telegram 通知……」,
Controller 裡的 `if-else` 會越長越長,每加一種通知方式,就要動 Controller。

### 問題 2:無法做單元測試
`new EmailService()` 一執行,就一定會真的去寄信。測試時沒辦法替換成「假裝有寄、其實什麼都沒做」的替身。

### 問題 3:職責混亂
`OrderController` 的工作應該是「處理訂單」,但它現在還要:
- 知道有哪些通知方式
- 知道怎麼建立這些通知服務
- 判斷要用哪種通知

違反了「一個類別只做一件事」的原則。

---

## 二、解決方向:抽象化 + 從外面交給我

### Step 1:用 Interface 定義契約

```csharp
public interface INotificationService
{
    void Send(string message);
}

public class EmailService : INotificationService
{
    public void Send(string message) { /* 寄 email */ }
}

public class SmsService : INotificationService
{
    public void Send(string message) { /* 寄簡訊 */ }
}
```

這個 interface 是**我們自己定的契約**,不是 ASP.NET Core 內建的。
它描述:「在這個系統裡,一個通知服務應該長什麼樣(必須有 `Send` 方法)」。

### Step 2:Controller 只依賴契約,不自己 `new`

```csharp
public class OrderController : Controller
{
    private INotificationService _notificationService;

    // 不自己 new,等外面把做好的東西塞進來
    public OrderController(INotificationService notificationService)
    {
        _notificationService = notificationService;
    }

    public IActionResult PlaceOrder()
    {
        _notificationService.Send("通知訂單建立");
        return View();
    }
}
```

**關鍵差別:**

| | 原本寫法 | DI 寫法 |
|---|---|---|
| 誰決定用哪個服務 | Controller 自己 | 外面的人 |
| Controller 認識哪些類別 | 全部具體類別 | 只認識 interface |
| 加新通知方式要改 Controller | 要 | **不用** |
| 測試時可以換假的嗎 | 不行 | **可以** |

---

## 三、核心概念:DI(依賴注入)

### 定義
**Dependency Injection(依賴注入)**:不要自己建立你需要的東西,讓別人從外面交給你。

### 拆解
- **Dependency(依賴)**:`OrderController` 需要 `INotificationService` 才能工作 → `INotificationService` 是它的依賴
- **Injection(注入)**:把依賴**從外面送進來**的動作 → 通常透過建構子(建構式注入)

### 一句話記住
> 不要自己 `new`,讓別人把做好的東西交給我。

---

## 四、IoC 容器:那個「外面」是誰?

### IoC = Inversion of Control(控制反轉)

| | 以前(沒有 IoC) | 現在(有 IoC) |
|---|---|---|
| 誰決定用哪個類別 | Controller 主動決定、主動 `new` | 容器決定 |
| 控制權 | 在 Controller 手上 | **反轉**到容器手上 |

### IoC 容器的角色

**可以想像成「購物網站的後台系統」**:

1. 一開始,告訴後台:「當有人要 `INotificationService`,請給他 `EmailService`」(**註冊** — 下一個單元的內容)
2. 當 Controller 被建立時,容器看到它建構子需要 `INotificationService`
3. 容器拿出 `EmailService`,塞進 Controller 的建構子

Controller 從頭到尾不知道自己拿到的是什麼實作,它只知道「我有一個能 `Send` 的東西」。

### 🛒 購物網站比喻(以付款為例)

想像妳在購物網站買東西,按下「**結帳**」按鈕。

| 程式裡的角色 | 購物網站比喻 | 在做什麼 |
|---|---|---|
| **Controller** | **買家(妳)** | 按下結帳,發出需求:「我要付款」 |
| **Interface**(`IPaymentService`) | **「付款方式」這個選項** | 選單上的「信用卡」「LINE Pay」「貨到付款」 |
| **具體類別**(`NewebPayService`) | **實際處理付款的公司** | 藍新金流、綠界、台新銀行…… |
| **IoC 容器** | **購物網站後台系統** | 妳選了「信用卡」,系統自動接上對應的金流公司 |

**流程對應:**

```
買家(妳)                    →  OrderController
   ↓ 「我要付款」                  ↓ 建構子需要 IPaymentService
購物網站後台(容器)          →  IoC 容器
   ↓ 根據設定,找到對應的公司       ↓ 根據註冊,找到對應的實作
藍新金流(具體公司)          →  NewebPayService(具體類別)
   ↓ 實際處理付款                  ↓ 實際做事
付款完成                     →  執行完成
```

**買家只要知道「我要付款」,不用管是哪家金流在處理。**
**這就是為什麼 Controller 只認 Interface,不認具體類別。**

---

## 五、背後的設計原則

### 依賴反轉原則(Dependency Inversion Principle)
> **依賴抽象,不依賴具體。**

寫程式時要依賴 **契約(interface / abstract class)**,而不是 **具體類別**。

### 開放封閉原則(Open-Closed Principle)
> **對擴充開放,對修改封閉。**

加新功能時只寫新類別,不改舊程式。

---

## 六、實際驗證:加一個 Telegram 通知,要改什麼?

照 DI 做法:

| 動作 | 要不要改? |
|---|---|
| `INotificationService`(契約) | ❌ 不改 |
| `OrderController` | ❌ 不改 |
| `EmailService`、`SmsService` | ❌ 不改 |
| 新增 `TelegramService` 類別,實作 `INotificationService` | ✅ 新增 |
| 容器註冊(下一個單元) | ✅ 新增一行 |

```csharp
public class TelegramService : INotificationService
{
    public void Send(string message)
    {
        // 發 Telegram 訊息
    }
}
```

**舊程式一行都不動,新功能就加上去了。** 這就是 DI 的威力。

---

## 七、未解決的疑問(下一個單元)

容器怎麼知道「當有人要 `INotificationService`,要給 `EmailService` 而不是 `SmsService`」?

→ 下一個單元:**服務註冊 — Transient、Scoped、Singleton**

而且還要決定:這個服務要**活多久**?
- 每次請求都給一個新的?
- 整個 HTTP Request 共用一個?
- 整個網站只有一個?

這就是三種生命週期的故事。

---

## 📌 小結

1. **為什麼需要 DI**:解決 Controller 自己 `new` 依賴造成的「難擴充、難測試、職責混亂」問題。
2. **DI 的核心做法**:不自己 `new`,透過建構子讓外面把依賴塞進來。
3. **IoC 容器的角色**:負責建立、管理、注入依賴的「管理員」,控制權從類別本身反轉到容器手上。
4. **關鍵設計原則**:依賴抽象(interface),不依賴具體類別。

---

> 📝 本課重點口訣:**「不要自己 new,讓別人把做好的東西交給我。我只認契約,不認實作。」**

---

## 八、自我檢核題

> 💡 **使用方式**:先遮住答案,自己先用「講出來 / 寫下來」的方式回答,再看參考答案。
> 有 ⭐ 標記的題目是**面試常考** / **需要特別鞏固**的重點。

---

### 🔹 基礎觀念題

---

#### **Q1.** 下面這段程式,為什麼說 `OrderController` 有問題?請至少講出 2 個問題點。

```csharp
public class OrderController : Controller
{
    private EmailService _emailService;

    public OrderController()
    {
        _emailService = new EmailService();
    }
}
```

<details>
<summary>點我看答案</summary>

至少要講到以下 2 點:
- **加新通知方式要改舊程式**(if-else 越寫越長)
- **無法單元測試**(一執行就真的會寄信,不能替換成假的)
- **職責混亂**(Controller 不該知道通知怎麼建立)
- **綁死在 `EmailService`**(無法換成其他實作)

</details>

---

#### ⭐ **Q2.** 「**DI(依賴注入)**」這個詞拆成兩半,分別是什麼意思?

<details>
<summary>點我看答案</summary>

- **Dependency(依賴)**:一個類別要運作**需要用到的東西**。
例如 `OrderController` 需要 `INotificationService` 才能工作 → `INotificationService` 是 `OrderController` 的「依賴」。

- **Injection(注入)**:把依賴**從外面送進來**的動作。
最常見的方式是**透過建構子參數**接收,叫「建構式注入」。

**白話記法**:
> 依賴 = 我需要的東西
> 注入 = 別人把它塞給我
> DI = 我需要的東西,請別人塞給我,不要自己做

</details>

---

#### ⭐ **Q3.** 用自己的話解釋「**IoC(控制反轉)**」,反轉的是什麼?從誰手上反轉到誰手上?

<details>
<summary>點我看答案</summary>

**反轉的是「控制權」——從「類別自己」反轉到「容器」手上。**

| | 反轉前 | 反轉後 |
|---|---|---|
| 誰決定用哪個類別? | Controller 自己 | 容器(根據註冊) |
| 誰 `new` 物件? | Controller 自己 | 容器 |
| 主動權在誰手上? | Controller | 容器 |

**白話記法**:
> 以前:我自己煎蛋(我主動)
> 現在:飯店上菜給我(飯店主動)
> 反轉的就是「誰在主動」

</details>

---

### 🔹 判斷題(True / False,並說明原因)

---

#### **Q4.** `INotificationService` 這個介面是 ASP.NET Core 框架內建的。

<details>
<summary>點我看答案</summary>

**❌ 錯(False)**

`INotificationService` 是**我們自己定義的介面**,不是 ASP.NET Core 內建的。
框架只提供 `interface` 這個**語法工具**,介面的名字、方法、契約都是根據業務需求自己設計。

**判斷原則**:
- 介面名字是**業務相關**(訂單、通知、使用者) → 幾乎都是自己定的
- 介面名字是**技術相關**(`ILogger`、`IConfiguration`、`IHttpClientFactory`) → 通常是框架內建

</details>

---

#### **Q5.** 使用 DI 之後,如果要新增一種通知方式(例如 LINE 通知),需要修改 `OrderController` 的程式碼。

<details>
<summary>點我看答案</summary>

**❌ 錯(False)**

只需要**新增一個實作 `INotificationService` 的新類別**(例如 `LineService`),
舊程式(Controller、Interface、其他服務類別)**完全不用動**。
這就是**開放封閉原則**(對擴充開放,對修改封閉)的體現。

</details>

---

#### ⭐ **Q6.** 在 ASP.NET Core 裡,DI 是一個「選用功能」,可以用也可以不用。

<details>
<summary>點我看答案</summary>

**❌ 錯(False)**

**DI 不是選用功能,是 ASP.NET Core 的基石。**

要區分兩件事:

| 事情 | 可不可以選? |
|---|---|
| 自己寫的類別要不要用 DI 註冊 | ✅ 可以選 |
| 框架核心功能是不是建立在 DI 上 | ❌ 不能選,一定是 |

**關鍵理由**:
Logging(`ILogger`)、Configuration(`IConfiguration`)、EF Core(`DbContext`)、
HttpClient(`IHttpClientFactory`)……這些**框架核心功能全部都只能透過 DI 拿**,沒有替代路徑。

**正確說法**:
> 「只要在寫 ASP.NET Core,就已經在用 DI 了。」

</details>

---

### 🔹 看程式判斷

---

#### **Q7.** 下面兩段程式,哪一段是 DI 寫法?為什麼?

```csharp
// 程式 A
public OrderController(EmailService emailService)
{
    _emailService = emailService;
}
```

```csharp
// 程式 B
private EmailService _emailService = new EmailService();
```

<details>
<summary>點我看答案</summary>

**程式 A 是 DI 寫法**。

**原因**:
- 程式 A **沒有自己 `new`**,透過**建構子參數**接收依賴 → 代表它「願意被外面注入」
- 程式 B 在宣告欄位時就直接 `new` 了,和在建構子裡 `new` 是**同樣的問題**,只是換個地方寫

**⚠️ 注意**:雖然程式 A 是 DI 寫法,但還可以改得**更好** — 應該依賴 `INotificationService` 而不是具體的 `EmailService`:

```csharp
// 更好的寫法
public OrderController(INotificationService notificationService)
```

這樣才符合「**依賴抽象,不依賴具體**」的原則。

</details>

---

### 🔹 觀念應用題

---

#### **Q8.** 用「購物網站付款」的比喻回答:
- **Controller** 在比喻裡是誰?
- **IoC 容器** 在比喻裡是誰?
- **Interface** 在比喻裡是什麼?

<details>
<summary>點我看答案</summary>

| 程式裡的角色 | 購物網站比喻 | 說明 |
|---|---|---|
| **Controller** | **買家(妳)** | 按下結帳,發出需求:「我要付款」 |
| **IoC 容器** | **購物網站後台系統** | 根據設定,自動接上對應的金流公司 |
| **Interface**(`IPaymentService`) | **「付款方式」這個選項** | 選單上的「信用卡」「LINE Pay」「貨到付款」 |
| **具體類別**(`NewebPayService`) | **實際處理付款的公司** | 藍新金流、綠界、台新銀行…… |

**一句話記住**:
> 買家說要付款,後台找金流公司。買家只認選項,不認公司。

</details>

---

#### **Q9.** 有人問:「我不做單元測試,那還需要學 DI 嗎?」妳會怎麼回答?

<details>
<summary>點我看答案</summary>

**要!** 測試只是 DI 的好處**之一**,不是全部。其他理由:

1. **擴充容易**:新增功能不用改舊程式(加通知方式、換通知平台)
2. **替換實作**:想換 DB、換簡訊供應商時,改註冊就好
3. **共用服務**:同一個 Service 被多個 Controller 使用
4. **團隊協作**:一致的寫法,大家看得懂
5. **職責清楚**:改一處不影響其他地方
6. **ASP.NET Core 框架本身就建立在 DI 上**:不學 DI = 不會用框架核心功能

</details>

---

#### **Q10.** 以前寫 WebForm 習慣「自己 new」,現在到 ASP.NET Core 為什麼要改?(職涯角度)

<details>
<summary>點我看答案</summary>

1. **技術面**:ASP.NET Core **強制依賴 DI**(Logging、Configuration、EF Core 全都是),不改習慣沒辦法寫
2. **工程面**:舊寫法難維護、風格不一;DI 有一致規則,團隊好協作
3. **擴充面**:新功能只需加新類別,舊程式不動,程式碼乾淨好維護
4. **職涯面**:**台灣徵 ASP.NET Core 工程師,面試必考 DI**。不會 DI = 無法證明自己是現代 .NET 開發者
5. **時代面**:WebForm 時代沒有內建 IoC 容器,所以習慣自己 new 很正常;現在框架內建了,不用這個習慣就是自找麻煩

</details>

---

## 九、複習重點(三大必記)

遮起來自問自答,能完整講出來才算過關:

1. ⭐ **DI 是什麼?** → 把「我要的東西」從「外面」塞進來,不要自己 `new`
2. ⭐ **IoC 反轉什麼?** → 反轉「控制權」,從類別自己反轉到容器手上
3. ⭐ **為什麼要用 DI?** → 好擴充、好替換、好測試、ASP.NET Core 框架本身就要
