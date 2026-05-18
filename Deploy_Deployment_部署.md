# 部署筆記:部署（Deployment）

---

## 一、為什麼需要「發行」？

開發時按 Run 跑的是**原始碼 + Debug 模式**，不適合直接放到伺服器上：

- 原始碼（`.cs`）需要編譯才能執行，伺服器不一定有編譯工具
- Debug 模式效能差、會輸出除錯資訊
- 開發資料夾包含很多伺服器不需要的檔案

`dotnet publish` 解決這些問題。

---

## 二、dotnet publish

### 指令

```powershell
dotnet publish -c Release -o ./publish
```

- `-c Release`：以 Release 模式編譯（效能最佳化）
- `-o ./publish`：輸出到 `./publish` 資料夾

### publish 資料夾內容

| 檔案 / 資料夾 | 說明 |
|---|---|
| `ExpenseSystem.dll` | 編譯後的程式本體（含 `.cshtml`） |
| `ExpenseSystem.exe` | 可直接執行的檔案 |
| 各種 `.dll` | NuGet 套件（EF Core、Scalar 等） |
| `appsettings.json` | 設定檔 |
| `web.config` | IIS 部署用的設定檔 |

> 注意：原始碼（`.cs`）、`Controllers/`、`Views/` 資料夾**不會出現在 publish 資料夾**。  
> `.cshtml` 在編譯時會被打包進 `ExpenseSystem.dll`。

---

## 三、IIS 部署

### Kestrel vs IIS

ASP.NET Core 有內建的輕量網頁伺服器 **Kestrel**，開發時按 Run 就是由 Kestrel 在監聽請求。

正式部署時，通常在 Kestrel 前面加一層 IIS：

```
瀏覽器 → IIS → Kestrel（ASP.NET Core 程式）
```

IIS 負責對外，Kestrel 在背後跑程式。IIS 把請求轉給 Kestrel 的動作叫做 **Reverse Proxy（反向代理）**。

### 為什麼需要 IIS？

Kestrel 是輕量伺服器，生產環境需要 IIS 提供的額外功能：

- 處理程序管理（程式當掉時自動重啟）
- SSL 憑證管理（HTTPS）
- 多個網站共用同一台伺服器
- 靜態檔案快取、壓縮

### 前置條件

伺服器上只需要安裝一個東西：

**ASP.NET Core Hosting Bundle**

它包含：
- .NET Runtime
- ASP.NET Core Runtime
- ASP.NET Core Module（讓 IIS 認識 ASP.NET Core）

### web.config 的作用

`dotnet publish` 自動產生，告訴 IIS 如何處理請求：

```xml
<handlers>
  <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" ... />
</handlers>
<aspNetCore processPath="dotnet" arguments=".\ExpenseSystem.dll" hostingModel="inprocess" />
```

- `AspNetCoreModuleV2`：IIS 用來轉發請求的模組
- `arguments=".\ExpenseSystem.dll"`：要執行的程式
- `hostingModel="inprocess"`：程式在 IIS 處理程序內執行（效能較好）

### 部署步驟（流程）

1. 伺服器安裝 **ASP.NET Core Hosting Bundle**
2. 開啟 IIS，新增網站（Site）
3. 指定實體路徑到 publish 資料夾
4. 設定 Port 或網域
5. 應用程式集區（Application Pool）選「**無受控程式碼**」

> **為什麼選「無受控程式碼」？**  
> 預設的應用程式集區是為舊的 .NET Framework（WebForm 時代）設計的。ASP.NET Core 有自己的執行環境，不需要 IIS 管理，所以設定成「無受控程式碼」讓 ASP.NET Core 自己處理。

---

## 四、CI/CD 觀念

### 手動部署的問題

沒有 CI/CD 時，工程師每次改完程式需要手動：

1. `dotnet publish`
2. 複製檔案到伺服器
3. 重啟 IIS

多人團隊下容易發生：版本不一致、操作錯誤、不知道誰部署了什麼。

### CI/CD 是什麼

- **CI（Continuous Integration，持續整合）**：每次 push 程式碼，自動編譯、跑測試
- **CD（Continuous Deployment，持續部署）**：CI 通過後，自動 publish、部署到伺服器

工程師只需要 `git push`，後面的步驟全部自動化。

### 團隊協作流程（搭配 Git branch）

```
每個工程師在自己的 branch 開發
        ↓
push 到 GitHub 自己的 branch
        ↓
發 Pull Request（PR）請其他人審查程式碼
        ↓
審查通過 → 合併到 main
        ↓
CI/CD 偵測到 main 有變動 → 自動部署
```

> 部署只在 **main branch 有變動**時觸發，不是每次 push 都部署。  
> Git 分支與 PR 實作會在「七、工具」主題學習。

### 常見工具

- GitHub Actions
- Azure DevOps

---

## 五、常見誤解

| 誤解 | 正確觀念 |
|---|---|
| Hosting Bundle 包含 IIS | IIS 是 Windows 內建功能，需另外開啟；Hosting Bundle 包含的是 .NET Runtime、ASP.NET Core Runtime、ASP.NET Core Module |
| 「無受控程式碼」是因為會轉給 Kestrel | 原因是 ASP.NET Core 有自己的執行環境，不需要 IIS 管理；選此選項是告訴 IIS「不用管這個程式」 |
| 導入 CI/CD 後工程師還需要手動執行 `dotnet publish` | CI/CD 後工程師只需要 `git push`，publish 和部署全部自動化 |

---

## 六、本次未實作項目

以下內容因需要完整伺服器環境，本次只學觀念，未實際操作：

- IIS 安裝與網站設定
- Hosting Bundle 安裝
- CI/CD 流程設定（GitHub Actions 等）

實際在工作環境中做過一次部署後，這些觀念會更清楚。
