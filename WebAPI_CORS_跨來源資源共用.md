# Web API 筆記:CORS（跨來源資源共用）

---

## 1. 為什麼需要 CORS？

### Same-Origin Policy（同源政策）

瀏覽器內建的安全機制：**JavaScript 只能呼叫「同源」的資源**，不同源的請求會被瀏覽器擋下。

「同源」的定義 — 以下三項必須**全部相同**：

| 項目 | 說明 | 範例 |
|------|------|------|
| Protocol（協定） | http 或 https | `https://` |
| Host（主機） | 網域名稱 | `mycompany.com` |
| Port（埠號） | 連接埠 | `5242` |

**範例：不同源**
- 前端：`http://frontend.mycompany.com`
- API：`http://api.mycompany.com`
- → Host 不同 → 瀏覽器擋下請求

### 為什麼要擋？

防止惡意網站用 JavaScript 偷偷呼叫其他網站的 API，竊取使用者資料。

---

## 2. CORS 是什麼？

**Cross-Origin Resource Sharing（跨來源資源共用）**

由**後端**決定「允許哪些來源呼叫我」，並透過 HTTP Response Header 告訴瀏覽器。瀏覽器收到允許的回應後才放行。

```
前端 JS 發請求
    → 瀏覽器問後端：「你允許我嗎？」
        → 後端回應 Header：「我允許 frontend.mycompany.com」
            → 瀏覽器：OK，放行！
```

---

## 3. ASP.NET Core 設定 CORS

設定在 `Program.cs`，分兩步：

### 步驟一：註冊 CORS Policy（builder 階段）

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAll", policy =>
    {
        policy.AllowAnyOrigin()
              .AllowAnyHeader()
              .AllowAnyMethod();
    });
});
```

### 步驟二：啟用 CORS Middleware（app 階段）

```csharp
app.UseHttpsRedirection();
app.UseCors("AllowAll");       // ← UseRouting 之後、UseAuthorization 之前
app.UseAuthorization();
app.MapControllerRoute(...);
```

### Policy 名稱的作用

`"AllowAll"` 出現兩次，是用來**串聯兩個步驟**：
- 步驟一：「定義一個叫 AllowAll 的規則」
- 步驟二：「套用那個叫 AllowAll 的規則」

名稱可以自訂，兩邊一致即可。

### Middleware 順序

`UseCors` 必須放在 `UseRouting` 之後、`UseAuthorization` 之前。

> 注意：.NET 6+ 的 `UseRouting` 會隱式帶入，不一定會明確看到這行，但順序邏輯不變。

---

## 4. AllowAnyOrigin vs WithOrigins

| 設定 | 適用情境 | 說明 |
|------|----------|------|
| `AllowAnyOrigin()` | **開發測試** | 任何來源都允許，方便但不安全 |
| `WithOrigins("https://...")` | **正式環境** | 只允許指定來源，安全 |

**正式環境範例：**

```csharp
policy.WithOrigins("https://frontend.mycompany.com")
      .AllowAnyHeader()
      .AllowAnyMethod();
```

### 為什麼正式環境不能用 AllowAnyOrigin？

任何人只要架一個網站，就能在裡面寫 JavaScript 呼叫你的 API，把資料庫裡的資料全部撈走。

---

## 5. 本次實作

**ExpenseSystem 加入 CORS 設定（開發用）：**

```csharp
// Program.cs — builder 階段
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAll", policy =>
    {
        policy.AllowAnyOrigin()
              .AllowAnyHeader()
              .AllowAnyMethod();
    });
});

// Program.cs — app 階段
app.UseCors("AllowAll");
```

- git commit：`989363d` — Add CORS policy (AllowAll for development)

---

> 📌 CORS 是後端設定，不需要背程式碼，工作上查文件即可。記住「為什麼需要」和「開發 vs 正式環境的差異」最重要。
