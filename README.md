# ASP.NET Core 學習筆記

透過 Claude Chat 學習 C# 與 ASP.NET Core 的過程筆記。

## 學習方式

使用 Claude Chat 的 Project 功能進行學習，先規劃好學習路線圖放在 Project Files 中，每完成一個單元就產生筆記，並更新路線圖供下一個進度的 Chat 參考，確保不同對話之間的學習銜接。

## 字彙表

| 資源 | 內容 |
|------|------|
| [技術字彙表](Glossary_技術字彙表.md) | 整理目前筆記中常見的英文單字、縮寫全名、音標、中文翻譯與代表出現筆記；縮寫的完整英文會接在下一列 |

## 學習筆記

依照學習路線排列，目前已完成 C# 基礎與 OOP 部分。

### C# 基礎

| 單元 | 內容概要 |
|------|----------|
| [Basics（基本語法）](CSharp_Basics_基本語法.md) | 變數、資料型別、Method、void、回傳值、Class、Constructor |
| [Encapsulation（封裝）](CSharp_Encapsulation_封裝.md) | private field、getter/setter、為什麼需要封裝 |
| [Property（屬性）](CSharp_Property_屬性.md) | full property、value 關鍵字、auto-property |
| [Constructor 與 Property（建構子與屬性綜合運用）](CSharp_ConstructorAndProperty_建構子與屬性綜合運用.md) | Field vs Property、this 關鍵字、Auto Property vs 完整 Property、驗證邏輯 |
| [Inheritance（繼承）](CSharp_Inheritance_繼承.md) | `:` 語法、`base`、建構子執行順序 |
| [Polymorphism（多型）](CSharp_Polymorphism_多型.md) | `virtual` + `override`、同一方法不同行為 |
| [Abstract Class（抽象類別）](CSharp_AbstractClass_抽象類別.md) | 抽象類別與抽象方法、強制子類別實作 |
| [Interface（介面）](CSharp_Interface_介面.md) | 介面定義與實作、多重實作 |
| [Static（靜態）](CSharp_Static_靜態.md) | 靜態方法、靜態屬性、屬於類別而非物件 |

### ASP.NET Core MVC

| 單元 | 內容概要 |
|------|----------|
| [MVC 基礎概念與實作](ASPNETCoreMVC_MVC基礎_MVC架構概念.md) | 專案建立、MVC 架構概念（Controller / Model / View）、Routing |
| [Program.cs 啟動流程](ASPNETCoreMVC_ProgramCs_啟動流程.md) | WebApplicationBuilder、WebApplication、啟動設定 |
| [專案結構詳解](ASPNETCoreMVC_ProjectStructure_專案結構詳解.md) | 各資料夾與檔案的用途 |
| [Action 方法與回傳型別](ASPNETCoreMVC_ActionResult_Action方法與回傳型別.md) | IActionResult、ViewResult |
| [參數傳遞](ASPNETCoreMVC_ParameterPassing_參數傳遞.md) | Route 參數、Query String |
| [Routing 設定](ASPNETCoreMVC_Routing_路由設定.md) | 慣例路由、屬性路由（Attribute Routing） |
| [Razor 語法基礎](ASPNETCoreMVC_RazorBasics_Razor語法基礎.md) | @ 符號、@{ }、@if、@foreach、模式切換 |
| [Layout 與版面配置](ASPNETCoreMVC_Layout_版面配置.md) | Layout 概念、_Layout.cshtml、_ViewStart.cshtml |
| [Partial View（部分檢視）](ASPNETCoreMVC_PartialView_部分檢視.md) | 可重複使用的 HTML 區塊、Html.Partial、載入 Partial View |
| [Tag Helpers（Tag 輔助器）](ASPNETCoreMVC_TagHelpers_Tag輔助器.md) | asp-controller、asp-action、Form Tag Helper、Input Tag Helper |
| [View Components（元件化概念）](ASPNETCoreMVC_ViewComponents_元件化概念.md) | View Component 結構、與 Partial View 的差異、使用時機 |
| [Model 定義與 Data Annotation](ASPNETCoreMVC_ModelDefinitionAndDataAnnotation_Model定義與DataAnnotation.md) | Model 定義、驗證屬性（Required、StringLength 等） |
| [Model Binding（模型繫結）](ASPNETCoreMVC_ModelBinding_模型繫結.md) | 表單資料如何對應到 Model、Binding 來源 |
| [表單驗證 — Server-side Validation](ASPNETCoreMVC_ServerSideValidation_表單驗證.md) | ModelState.IsValid、驗證錯誤回傳 View |
| [HTML / CSS 基礎](ASPNETCoreMVC_HTML_CSS_Basics_HTML_CSS基礎.md) | 隨表單實作穿插學習的 HTML / CSS 基礎概念 |
| [Middleware（中介軟體）](ASPNETCoreMVC_Middleware_中介軟體.md) | Middleware 概念、Request/Response Pipeline、內建 Middleware 與執行順序 |
| [DI 概念](ASPNETCoreMVC_DIConcept_為什麼需要DI與IoC容器.md) | 為什麼需要 DI、IoC 容器 |
| [服務註冊](ASPNETCoreMVC_ServiceRegistration_服務註冊.md) | Transient、Scoped、Singleton 差異與使用時機 |
| [Constructor Injection（建構式注入）](ASPNETCoreMVC_ConstructorInjection_建構式注入.md) | 建構式注入實作、在 Controller 中使用 DI |
| [DI（依賴注入）綜合運用](ASPNETCoreMVC_DI_依賴注入.md) | DI 完整應用、在 Controller 與 View 中使用 |
| [appsettings.json 與環境設定檔](ASPNETCoreMVC_appsettings_組態設定.md) | appsettings.json、appsettings.Development.json、ASPNETCORE_ENVIRONMENT、環境設定檔 |
| [環境變數與 ASPNETCORE_ENVIRONMENT](ASPNETCoreMVC_Environment_環境變數與ASPNETCORE_ENVIRONMENT.md) | ASPNETCORE_ENVIRONMENT、環境檔覆寫機制、優先順序 |
| [Logging（日誌）](ASPNETCoreMVC_Logging_日誌.md) | Log Level、ILogger<T>、日誌過濾門檻 |
| [設定 Logging 輸出](ASPNETCoreMVC_LoggingOutput_設定Logging輸出.md) | Logging Provider 概念、內建 Provider、第三方套件（Serilog/NLog）介紹 |

### 資料存取

| 單元 | 內容概要 |
|------|----------|
| [RDBMS 基礎觀念](DataAccess_RDBMS_RDBMS基礎觀念.md) | Table、Column、Row、Primary Key、Foreign Key、Normalization、NULL 觀念 |
| [資料關聯 — 一對多關聯](DataAccess_OneToMany_資料關聯.md) | 一對多設計原則、FK 放置位置、命名慣例、SQL 實作 |
| [INSERT / UPDATE / DELETE](DataAccess_InsertUpdateDelete_新增修改刪除.md) | 新增修改刪除語法、CRUD 概念、IDENTITY 特性、軟刪除 vs 硬刪除 |
| [JOIN (INNER JOIN)](DataAccess_InnerJoin_JOIN查詢.md) | 隱式 JOIN vs 顯式 JOIN、INNER JOIN 語法、ON vs WHERE、多表查詢練習 |
| [終端機與 CLI 入門](DataAccess_Terminal_終端機與CLI入門.md) | Windows Terminal、PowerShell 指令、Claude Code CLI 簡介 |
| [建立專案與 git 初始化](DataAccess_ProjectInit_建立專案與git初始化.md) | 報銷系統設計、git 心智模型、.gitignore、HTTPS 與 SSH 認證 |
| [EF Core 概念 — ORM 與 DbContext](DataAccess_EFCore概念_ORM與DbContext.md) | ORM 概念、EF Core 安裝、DbContext 與 DbSet、Model 定義 |
| [連線設定 — Connection String](DataAccess_ConnectionString_連線設定.md) | appsettings.json 設定、Program.cs 註冊 DbContext |
| [Migration — 資料庫遷移](DataAccess_Migration_資料庫遷移.md) | Code First 流程、add-migration、database-update、__EFMigrationsHistory |
| [Scaffold-DbContext — 反向工程](DataAccess_ScaffoldDbContext_反向工程.md) | Database First 流程、Scaffold 指令參數、導覽屬性概念 |
| [LINQ 基礎語法 — Where、Select、OrderBy](DataAccess_LINQBasics_LINQ基礎語法.md) | Lambda 運算式、Where/Select/OrderBy 語法、常用輔助方法 |
| [LINQ Join 與 GroupBy](DataAccess_LINQJoinGroupBy_LINQ_Join與GroupBy.md) | LINQ GroupBy 分組統計、Join 關聯查詢、匿名型別 |
| [LINQ 與 EF Core 的結合使用](DataAccess_LINQWithEFCore_LINQ與EFCore的結合使用.md) | IQueryable vs IEnumerable、延遲執行、SQL 翻譯對照 |
| [變更追蹤 (Change Tracking)](DataAccess_ChangeTracking_變更追蹤.md) | EntityState 狀態、Snapshot 比對、AsNoTracking 效能優化 |
| [GroupBy 聚合函數](DataAccess_GroupBy_GroupBy聚合函數.md) | COUNT、SUM、GROUP BY 語法、HAVING 過濾 |
| [子查詢 (Subquery)](DataAccess_Subquery_子查詢.md) | 子查詢概念、= vs IN、NOT IN、SQL 執行順序 |
| [CRUD 實作](DataAccess_CRUD_CRUD實作.md) | 整合 MVC + EF Core、PRG 模式、View 對應規則、Tag Helpers 應用 |

### Web API

| 單元 | 內容概要 |
|------|----------|
| [Web API 與 MVC 的差異](WebAPI_WebAPIvsMVC_WebAPI與MVC的差異.md) | JSON 格式介紹、回傳內容差異、適用情境比較 |
| [RESTful 設計原則](WebAPI_RESTful_RESTful設計原則.md) | HTTP Method 對應 CRUD、HTTP 狀態碼、async/await 規則 |
| [ApiController 與路由屬性](WebAPI_ApiController_ApiController與路由屬性.md) | ControllerBase、[ApiController] 行為、屬性路由（Attribute Routing） |
| [回傳格式 — JSON、IActionResult](WebAPI_ReturnFormats_回傳格式.md) | ActionResult<T>、序列化與反序列化、ProducesResponseType |
| [CORS（跨來源資源共用）](WebAPI_CORS_跨來源資源共用.md) | 同源政策（Same-Origin Policy）、CORS 設定、安全考量 |
| [Swagger / OpenAPI (API 文件產生)](WebAPI_Swagger_OpenAPI_API文件產生.md) | OpenAPI 標準、Scalar UI、自動推斷與手動標註狀態碼 |

### 認證與授權

| 單元 | 內容概要 |
|------|----------|
| [Authentication vs Authorization 概念](Auth_AuthenticationVsAuthorization_認證與授權概念.md) | 認證（是誰）與授權（能做什麼）的差異、HTTP 無狀態與 Cookie |
| [Cookie-based 驗證](Auth_CookieAuthentication_Cookie驗證.md) | Claim/Identity/Principal 三層結構、SignInAsync、[Authorize] 屬性 |
| [JWT Token 驗證 (概念了解)](Auth_JWTToken_JWT驗證概念.md) | JWT 三段結構 (Header/Payload/Signature)、密鑰安全性、Bearer Token |

> 這份 MVC 筆記是學習旅程的起點。上完這堂課後，判斷需要先回頭打好 C# 基礎，因此才產生了上方的 C# 系列筆記。

## 學習進度

- ✅ C# 基本語法
- ✅ OOP（物件導向）— 封裝、屬性、繼承、多型、抽象類別、介面、靜態
- ✅ ASP.NET Core MVC（基礎完成）
- ✅ 資料存取（SQL、EF Core、LINQ）
- ✅ Web API
- ✅ 認證與授權
- ⬜ 部署
