# ASP.NET Core 學習筆記

透過 Claude Chat 學習 C# 與 ASP.NET Core 的過程筆記。

## 學習方式

使用 Claude Chat 的 Project 功能進行學習，先規劃好學習路線圖放在 Project Files 中，每完成一個單元就產生筆記，並更新路線圖供下一個進度的 Chat 參考，確保不同對話之間的學習銜接。

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

> 這份 MVC 筆記是學習旅程的起點。上完這堂課後，判斷需要先回頭打好 C# 基礎，因此才產生了上方的 C# 系列筆記。

## 學習進度

- ✅ C# 基本語法
- ✅ OOP（物件導向）— 封裝、屬性、繼承、多型、抽象類別、介面、靜態
- ⬜ ASP.NET Core MVC（進行中）
- ⬜ 資料存取（SQL、EF Core、LINQ）
- ⬜ Web API
- ⬜ 認證與授權
- ⬜ 部署
