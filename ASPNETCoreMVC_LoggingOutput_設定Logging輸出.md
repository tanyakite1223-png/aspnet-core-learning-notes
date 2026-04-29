# ASP.NET Core MVC 筆記:設定 Logging 輸出(概念了解)

## 一、核心概念:Log 寫法 vs Log 去向

ASP.NET Core 把 Logging 拆成兩部分,各司其職:

| 角色 | 對應 | 負責什麼 |
|---|---|---|
| **`ILogger`(介面)** | 寫法 | 提供 `LogInformation`、`LogWarning`、`LogError`...等方法 |
| **Logging Provider(實作)** | 去向 | 決定 log 要送到哪裡(主控台、檔案、資料庫...) |

> 💡 這就是 **Interface(介面)的應用** —— 把「做什麼」跟「怎麼做」分開。
> 換 Provider 不需要改 Controller 程式碼。

```csharp
// Controller 裡的寫法永遠不變
_logger.LogInformation("使用者登入:{User}", userName);
// ↑ 不管 log 最後送去哪,這行都不用改
```

---

## 二、ASP.NET Core 內建的 Logging Provider

當你寫:
```csharp
var builder = WebApplication.CreateBuilder(args);
```

這行會**自動註冊**以下 Provider:

| Provider | log 送到哪裡 | 用途 |
|---|---|---|
| **Console** | 黑色主控台視窗 | 開發階段最常看到的(F5 啟動時的黑窗) |
| **Debug** | Visual Studio「輸出」視窗 | 偵錯模式用 |
| **EventSource** | 系統事件追蹤(ETW) | 進階效能分析 |
| **EventLog** | Windows 事件檢視器 | 僅 Windows 系統有效 |

⚠️ **內建 Provider 沒有「寫入檔案」這個選項!**

---

## 三、為什麼內建沒有 File Provider?

微軟的設計哲學:**「核心要小,擴充靠社群。」**

### 1. 寫檔案需求太多樣
- 怎麼分檔?按日期?按大小?
- 什麼格式?純文字?JSON?XML?
- 保留多久?7 天?30 天?
- 舊檔處理?刪除?壓縮?

微軟很難做一個「滿足所有人」的 File Provider。

### 2. 第三方做得比微軟好
業界已有成熟方案,微軟不重新造輪子:

| 第三方套件 | 特色 |
|---|---|
| **Serilog** | 業界主流,台灣職場面試常見 |
| **NLog** | 老牌套件,功能完整 |

兩者都透過 NuGet 安裝,在 `Program.cs` 註冊後即可使用。

### 3. 介面不變,實作隨便換
要寫檔案?裝 Serilog 或 NLog,Controller 程式碼**一行都不用改**。

---

## 四、Logging 設定:`appsettings.json`

### 預設範本

```json
"Logging": {
  "LogLevel": {
    "Default": "Information",
    "Microsoft.AspNetCore": "Warning"
  }
}
```

### 拆解說明

#### `"Default": "Information"`
- 預設情況下,**Information 等級(含)以上**才會輸出
- `Trace` 跟 `Debug` 會被過濾掉

#### `"Microsoft.AspNetCore": "Warning"`
- 針對「特定來源」的覆蓋設定
- 所有以 `Microsoft.AspNetCore` 開頭的 log,**只有 Warning 以上**才輸出
- 為什麼要特別調高這個?**避免框架自己的 log 淹沒你寫的 log**

### Log Level 順序(由低到高)
```
Trace < Debug < Information < Warning < Error < Critical
```

📌 **重點**:設定的等級代表「**這個等級以上都會被記錄**」。

---

## 五、不同環境用不同設定檔

### 檔案結構
```
專案根目錄/
├── appsettings.json                  ← 基底設定(所有環境共用)
├── appsettings.Development.json      ← 開發環境覆蓋
├── appsettings.Staging.json          ← 預備環境覆蓋(可選)
└── appsettings.Production.json       ← 正式環境覆蓋(可選)
```

### 載入順序與覆蓋規則

```
1. 先讀 appsettings.json              ← 基底
2. 再讀 appsettings.{環境}.json       ← 同名設定會覆蓋基底
```

| 設定 | appsettings.json | appsettings.Development.json | 最終生效 |
|---|---|---|---|
| Default | Information | (沒寫) | Information |
| Default | Information | **Debug** | **Debug**(被覆蓋) |
| Microsoft.AspNetCore | Warning | (沒寫) | Warning |

---

## 六、環境是怎麼決定的?

由環境變數 **`ASPNETCORE_ENVIRONMENT`** 決定:

| 值 | 用途 | 對應檔案 |
|---|---|---|
| `Development` | 開發環境 | `appsettings.Development.json` |
| `Staging` | 預備環境(上線前測試) | `appsettings.Staging.json` |
| `Production` | 正式環境 | `appsettings.Production.json` |

> 📌 **沒設定的話預設是 `Production`**(正式環境)。
> Visual Studio 啟動時會自動設成 `Development`。

---

## 七、不同環境的 Log Level 策略

| 環境 | Default 建議值 | 原因 |
|---|---|---|
| **開發** | `Information` | 資訊量適中,方便偵錯 |
| **正式** | `Warning` | 避免 log 爆量、效能下降、成本問題 |

### 為什麼正式環境要用 Warning?

1. **檔案會爆炸大** 💾 — 一天可能 10GB,硬碟爆滿
2. **找問題反而更難** 🔍 — 真正的錯誤被淹沒在海量訊息裡
3. **效能會變差** ⚡ — 寫 log 是 I/O 操作,有成本
4. **雲端費用高** 💰 — 通常按 log 量收費

---

## 八、整個流程串起來

```
Amber 在 VS 按 F5
    ↓
VS 自動設定 ASPNETCORE_ENVIRONMENT=Development
    ↓
ASP.NET Core 讀取 appsettings.json
    ↓
再讀取 appsettings.Development.json(覆蓋同名設定)
    ↓
LogLevel 確定 → 哪些 log 會輸出
    ↓
透過已註冊的 Provider(Console, Debug...) 
    ↓
log 跑去黑色視窗
```

---

## 九、常見錯誤

### ❌ 錯誤 1:正式環境 Default 設太高,漏掉重要錯誤
```json
// ❌ 正式環境設成 Critical
"LogLevel": {
  "Default": "Critical"
}
```
> ⚠️ 後果:`Error` 等級的 log 會被過濾掉!出錯時找不到原因。
> 因為 `Critical` 是**最高等級**,設成 `Critical` 表示**只記錄 Critical**。

```json
// ✅ 正式環境合理設定
"LogLevel": {
  "Default": "Warning"
}
```

---

### ❌ 錯誤 2:開發環境 Microsoft.AspNetCore 設成 Information
```json
// ❌ 框架的 log 會淹沒自己的 log
"LogLevel": {
  "Default": "Information",
  "Microsoft.AspNetCore": "Information"
}
```
> ⚠️ 後果:每個請求印出幾十行框架 log,自己寫的訊息根本找不到。

```json
// ✅ 把舞台留給開發者自己的 log
"LogLevel": {
  "Default": "Information",
  "Microsoft.AspNetCore": "Warning"
}
```

---

### ❌ 錯誤 3:以為改 `appsettings.json` 就能寫到檔案
> 內建 Provider **沒有 File**,光改 LogLevel 不會讓 log 寫到檔案!
> ✅ 要寫檔案必須安裝 Serilog 或 NLog 之類的第三方套件。

---

## 十、五個核心重點(背起來)

1. **`ILogger` 是介面,Provider 是實作** — 換 Provider 不用改 Controller 程式碼
2. **內建 Provider 沒有 File** — 要寫檔案要用 Serilog / NLog
3. **Logging 設定寫在 `appsettings.json`** — 包含 LogLevel 跟 Provider
4. **不同環境用不同設定檔** — 透過 `ASPNETCORE_ENVIRONMENT` 切換
5. **開發用 Information、正式用 Warning** — 平衡資訊量跟效能

---

## 十一、補充:本主題未實作部分(將來工作會用到)

本主題標註為「**概念了解**」,所以沒做以下實作。將來工作上需要時再深入:

- 安裝並設定 Serilog 把 log 寫到檔案
- 設定 log 按日期分檔、檔案大小限制、保留天數
- 把 log 送到雲端服務(如 Azure Application Insights、Seq)
- 自訂 Log 格式(JSON 結構化日誌)
