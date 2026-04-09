# ASP.NET Core 筆記：專案結構詳解 — 各資料夾與檔案的用途

---

## 專案結構總覽

以 ASP.NET Core MVC 專案為例，方案總管中的資料夾與檔案：

| 資料夾 / 檔案 | 用途 |
|---|---|
| **Controllers** | 放 Controller（控制器） |
| **Models** | 放 Model（資料模型） |
| **Views** | 放 View（畫面，`.cshtml` 檔案） |
| **wwwroot** | 放靜態檔案（Static Files） |
| **Properties/launchSettings.json** | 開發時的啟動設定 |
| **appsettings.json** | 應用程式設定檔 |
| **Program.cs** | 程式進入點，負責啟動和設定整個應用程式 |
| **Connected Services** | 外部服務連接（目前不用） |
| **相依性** | 套件和函式庫清單（類似「材料清單」） |

---

## 各項說明

### Controllers / Models / Views

MVC 三大核心資料夾，分別對應 MVC 架構的三個角色：

- **Controller**：接收請求、協調、回傳結果（服務生）
- **Model**：處理資料、商業邏輯（廚房）
- **View**：呈現畫面給使用者（菜單/擺盤）

### wwwroot — 靜態檔案（Static Files）

放不需要經過程式處理、直接丟給瀏覽器用的檔案：

- **css/**：樣式表（控制顏色、字體大小、按鈕樣式）
- **js/**：JavaScript（控制互動效果，例如按鈕彈出視窗）
- **lib/**：第三方函式庫
- **favicon.ico**：瀏覽器分頁上的小圖示

### Properties/launchSettings.json — 開發時的啟動設定

按下 Visual Studio 的「執行」按鈕時的設定：

- 使用哪個埠號（port number）
- 自動開哪個瀏覽器
- 要不要啟動 HTTPS

**重點：這是 Visual Studio 專用的，離開 Visual Studio 就用不到了。上線後由伺服器（如 IIS）來管這些事。**

### appsettings.json — 應用程式設定檔

應用程式本身的設定，開發和上線都會用到：

- 目前預設只有 `Logging`（日誌設定）和 `AllowedHosts`
- 未來學資料庫時，**資料庫連線字串**會寫在這裡

### launchSettings.json vs appsettings.json

| 比較 | launchSettings.json | appsettings.json |
|------|-------------------|-----------------|
| 誰用 | Visual Studio 專用 | 應用程式本身 |
| 什麼時候用 | 開發時按「執行」 | 開發 + 上線都用 |
| 比喻 | 在家煮飯的廚房設備 | 食譜本身 |
| 上線後 | 不需要，由伺服器管理 | 繼續使用 |

### Program.cs — 程式進入點

負責「畫面開啟前的準備跟開啟畫面」：

- 建立和設定整個應用程式（WebApplicationBuilder）
- 啟動 Routing 等功能
- 是整個程式的起點

---

## 下一步

學習 Action 方法與回傳型別（IActionResult、ViewResult）
