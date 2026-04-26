# 技術字彙表

這份字彙表整理目前 C# 與 ASP.NET Core MVC 筆記中常見的英文單字與技術名詞。

使用方式建議：

- 縮寫名詞先背「縮寫怎麼唸」與「完整英文」，例如 `MVC` 通常逐字母唸 M-V-C。
- 優先背會在討論中反覆出現的名詞，例如 `Class`、`Method`、`Controller`、`Model`、`View`、`Routing`、`DI`。
- 音標以常見美式發音為主；縮寫會先標示縮寫念法，再列出完整英文中每個字的音標。

| 單字 / 縮寫 | 音標 / 念法 | 中文翻譯 | 代表出現筆記 |
|---|---|---|---|
| C#<br>C Sharp | 縮寫念法：/siː ʃɑːrp/<br>完整英文：C /siː/、Sharp /ʃɑːrp/ | C# 程式語言 | [Basics](CSharp_Basics_基本語法.md) |
| ASP.NET Core<br>Active Server Pages .NET Core | 縮寫念法：/ˌeɪ es piː dɑːt net kɔːr/<br>完整英文：Active /ˈæktɪv/、Server /ˈsɝːvər/、Pages /ˈpeɪdʒɪz/、Core /kɔːr/ | ASP.NET Core 框架 | [MVC 基礎](ASPNETCoreMVC_MVC基礎_MVC架構概念.md) |
| MVC<br>Model-View-Controller | 縮寫念法：/ˌem viː ˈsiː/<br>完整英文：Model /ˈmɑːdl/、View /vjuː/、Controller /kənˈtroʊlər/ | 模型-檢視-控制器架構 | [MVC 基礎](ASPNETCoreMVC_MVC基礎_MVC架構概念.md) |
| OOP<br>Object-Oriented Programming | 縮寫念法：/ˌoʊ oʊ ˈpiː/<br>完整英文：Object /ˈɑːbdʒekt/、Oriented /ˈɔːrientɪd/、Programming /ˈproʊɡræmɪŋ/ | 物件導向程式設計 | [Encapsulation](CSharp_Encapsulation_封裝.md) |
| Class | /klæs/ | 類別 | [Basics](CSharp_Basics_基本語法.md) |
| Object | /ˈɑːbdʒekt/ | 物件 | [Basics](CSharp_Basics_基本語法.md) |
| Method | /ˈmeθəd/ | 方法 | [Basics](CSharp_Basics_基本語法.md) |
| Variable | /ˈveriəbl/ | 變數 | [Basics](CSharp_Basics_基本語法.md) |
| Data Type | Data /ˈdeɪtə/、Type /taɪp/ | 資料型別 | [Basics](CSharp_Basics_基本語法.md) |
| Return Value | Return /rɪˈtɝːn/、Value /ˈvæljuː/ | 回傳值 | [Basics](CSharp_Basics_基本語法.md) |
| Constructor | /kənˈstrʌktər/ | 建構子 | [Constructor 與 Property](CSharp_ConstructorAndProperty_建構子與屬性綜合運用.md) |
| Property | /ˈprɑːpərti/ | 屬性 | [Property](CSharp_Property_屬性.md) |
| Field | /fiːld/ | 欄位 | [Property](CSharp_Property_屬性.md) |
| Encapsulation | /ɪnˌkæpsəˈleɪʃən/ | 封裝 | [Encapsulation](CSharp_Encapsulation_封裝.md) |
| Private | /ˈpraɪvət/ | 私有的 | [Encapsulation](CSharp_Encapsulation_封裝.md) |
| Getter | /ˈɡetər/ | 取值器 | [Encapsulation](CSharp_Encapsulation_封裝.md) |
| Setter | /ˈsetər/ | 設值器 | [Encapsulation](CSharp_Encapsulation_封裝.md) |
| Inheritance | /ɪnˈherɪtəns/ | 繼承 | [Inheritance](CSharp_Inheritance_繼承.md) |
| Base | /beɪs/ | 基底、父類別相關關鍵字 | [Inheritance](CSharp_Inheritance_繼承.md) |
| Polymorphism | /ˌpɑːliˈmɔːrfɪzəm/ | 多型 | [Polymorphism](CSharp_Polymorphism_多型.md) |
| Virtual | /ˈvɝːtʃuəl/ | 虛擬的、可覆寫的 | [Polymorphism](CSharp_Polymorphism_多型.md) |
| Override | /ˌoʊvərˈraɪd/ | 覆寫 | [Polymorphism](CSharp_Polymorphism_多型.md) |
| Abstract Class | Abstract /ˈæbstrækt/、Class /klæs/ | 抽象類別 | [Abstract Class](CSharp_AbstractClass_抽象類別.md) |
| Abstract Method | Abstract /ˈæbstrækt/、Method /ˈmeθəd/ | 抽象方法 | [Abstract Class](CSharp_AbstractClass_抽象類別.md) |
| Interface | /ˈɪntərfeɪs/ | 介面 | [Interface](CSharp_Interface_介面.md) |
| Static | /ˈstætɪk/ | 靜態 | [Static](CSharp_Static_靜態.md) |
| Controller | /kənˈtroʊlər/ | 控制器 | [MVC 基礎](ASPNETCoreMVC_MVC基礎_MVC架構概念.md) |
| Model | /ˈmɑːdl/ | 模型 | [MVC 基礎](ASPNETCoreMVC_MVC基礎_MVC架構概念.md) |
| View | /vjuː/ | 檢視、畫面 | [MVC 基礎](ASPNETCoreMVC_MVC基礎_MVC架構概念.md) |
| Routing | /ˈruːtɪŋ/ | 路由 | [Routing](ASPNETCoreMVC_Routing_路由設定.md) |
| Route | /ruːt/ | 路由、路徑規則 | [參數傳遞](ASPNETCoreMVC_ParameterPassing_參數傳遞.md) |
| Action | /ˈækʃən/ | Controller 中的動作方法 | [Action 方法與回傳型別](ASPNETCoreMVC_ActionResult_Action方法與回傳型別.md) |
| ActionResult | Action /ˈækʃən/、Result /rɪˈzʌlt/ | Action 的回傳結果 | [Action 方法與回傳型別](ASPNETCoreMVC_ActionResult_Action方法與回傳型別.md) |
| IActionResult<br>Interface ActionResult | 縮寫念法：/aɪ ˈækʃən rɪˈzʌlt/<br>完整英文：Interface /ˈɪntərfeɪs/、Action /ˈækʃən/、Result /rɪˈzʌlt/ | Action 回傳型別介面 | [Action 方法與回傳型別](ASPNETCoreMVC_ActionResult_Action方法與回傳型別.md) |
| ViewResult | View /vjuː/、Result /rɪˈzʌlt/ | 回傳 View 的結果型別 | [Action 方法與回傳型別](ASPNETCoreMVC_ActionResult_Action方法與回傳型別.md) |
| Parameter | /pəˈræmɪtər/ | 參數 | [參數傳遞](ASPNETCoreMVC_ParameterPassing_參數傳遞.md) |
| Query String | Query /ˈkwɪri/、String /strɪŋ/ | 查詢字串 | [參數傳遞](ASPNETCoreMVC_ParameterPassing_參數傳遞.md) |
| Attribute Routing | Attribute /ˈætrɪbjuːt/、Routing /ˈruːtɪŋ/ | 屬性路由 | [Routing](ASPNETCoreMVC_Routing_路由設定.md) |
| Razor | /ˈreɪzər/ | Razor 語法 / 檢視引擎 | [Razor 語法基礎](ASPNETCoreMVC_RazorBasics_Razor語法基礎.md) |
| Layout | /ˈleɪaʊt/ | 版面配置 | [Layout](ASPNETCoreMVC_Layout_版面配置.md) |
| Partial View | Partial /ˈpɑːrʃəl/、View /vjuː/ | 部分檢視 | [Partial View](ASPNETCoreMVC_PartialView_部分檢視.md) |
| Tag Helper | Tag /tæɡ/、Helper /ˈhelpər/ | Tag 輔助器 | [Tag Helpers](ASPNETCoreMVC_TagHelpers_Tag輔助器.md) |
| View Component | View /vjuː/、Component /kəmˈpoʊnənt/ | 檢視元件 | [View Components](ASPNETCoreMVC_ViewComponents_元件化概念.md) |
| Component | /kəmˈpoʊnənt/ | 元件 | [View Components](ASPNETCoreMVC_ViewComponents_元件化概念.md) |
| Data Annotation | Data /ˈdeɪtə/、Annotation /ˌænoʊˈteɪʃən/ | 資料註解、驗證屬性 | [Model 定義與 Data Annotation](ASPNETCoreMVC_ModelDefinitionAndDataAnnotation_Model定義與DataAnnotation.md) |
| Required | /rɪˈkwaɪərd/ | 必填 | [Model 定義與 Data Annotation](ASPNETCoreMVC_ModelDefinitionAndDataAnnotation_Model定義與DataAnnotation.md) |
| StringLength<br>String Length | 念法：String /strɪŋ/、Length /leŋθ/ | 字串長度 | [Model 定義與 Data Annotation](ASPNETCoreMVC_ModelDefinitionAndDataAnnotation_Model定義與DataAnnotation.md) |
| Model Binding | Model /ˈmɑːdl/、Binding /ˈbaɪndɪŋ/ | 模型繫結 | [Model Binding](ASPNETCoreMVC_ModelBinding_模型繫結.md) |
| Binding | /ˈbaɪndɪŋ/ | 繫結 | [Model Binding](ASPNETCoreMVC_ModelBinding_模型繫結.md) |
| Validation | /ˌvælɪˈdeɪʃən/ | 驗證 | [表單驗證](ASPNETCoreMVC_ServerSideValidation_表單驗證.md) |
| Server-side | Server /ˈsɝːvər/、Side /saɪd/ | 伺服器端 | [表單驗證](ASPNETCoreMVC_ServerSideValidation_表單驗證.md) |
| ModelState | Model /ˈmɑːdl/、State /steɪt/ | Model 狀態 | [表單驗證](ASPNETCoreMVC_ServerSideValidation_表單驗證.md) |
| HTML<br>HyperText Markup Language | 縮寫念法：/ˌeɪtʃ tiː em ˈel/<br>完整英文：HyperText /ˈhaɪpərtekst/、Markup /ˈmɑːrkʌp/、Language /ˈlæŋɡwɪdʒ/ | 超文字標記語言 | [HTML / CSS 基礎](ASPNETCoreMVC_HTML_CSS_Basics_HTML_CSS基礎.md) |
| CSS<br>Cascading Style Sheets | 縮寫念法：/ˌsiː es ˈes/<br>完整英文：Cascading /kæˈskeɪdɪŋ/、Style /staɪl/、Sheets /ʃiːts/ | 階層式樣式表 | [HTML / CSS 基礎](ASPNETCoreMVC_HTML_CSS_Basics_HTML_CSS基礎.md) |
| Middleware | /ˈmɪdlwer/ | 中介軟體 | [Middleware](ASPNETCoreMVC_Middleware_中介軟體.md) |
| Request | /rɪˈkwest/ | 請求 | [Middleware](ASPNETCoreMVC_Middleware_中介軟體.md) |
| Response | /rɪˈspɑːns/ | 回應 | [Middleware](ASPNETCoreMVC_Middleware_中介軟體.md) |
| Pipeline | /ˈpaɪplaɪn/ | 管線、處理流程 | [Middleware](ASPNETCoreMVC_Middleware_中介軟體.md) |
| DI<br>Dependency Injection | 縮寫念法：/ˌdiː ˈaɪ/<br>完整英文：Dependency /dɪˈpendənsi/、Injection /ɪnˈdʒekʃən/ | 依賴注入 | [DI 概念](ASPNETCoreMVC_DIConcept_為什麼需要DI與IoC容器.md) |
| Dependency | /dɪˈpendənsi/ | 依賴 | [DI 概念](ASPNETCoreMVC_DIConcept_為什麼需要DI與IoC容器.md) |
| Injection | /ɪnˈdʒekʃən/ | 注入 | [Constructor Injection](ASPNETCoreMVC_ConstructorInjection_建構式注入.md) |
| IoC<br>Inversion of Control | 縮寫念法：/ˌaɪ oʊ ˈsiː/<br>完整英文：Inversion /ɪnˈvɝːʒən/、Control /kənˈtroʊl/ | 控制反轉 | [DI 概念](ASPNETCoreMVC_DIConcept_為什麼需要DI與IoC容器.md) |
| Container | /kənˈteɪnər/ | 容器 | [DI 概念](ASPNETCoreMVC_DIConcept_為什麼需要DI與IoC容器.md) |
| Service | /ˈsɝːvɪs/ | 服務 | [服務註冊](ASPNETCoreMVC_ServiceRegistration_服務註冊.md) |
| Registration | /ˌredʒɪˈstreɪʃən/ | 註冊 | [服務註冊](ASPNETCoreMVC_ServiceRegistration_服務註冊.md) |
| Transient | /ˈtrænziənt/ | 暫時性生命週期 | [服務註冊](ASPNETCoreMVC_ServiceRegistration_服務註冊.md) |
| Scoped | /skoʊpt/ | 範圍內生命週期 | [服務註冊](ASPNETCoreMVC_ServiceRegistration_服務註冊.md) |
| Singleton | /ˈsɪŋɡltən/ | 單例生命週期 | [服務註冊](ASPNETCoreMVC_ServiceRegistration_服務註冊.md) |
| Constructor Injection | Constructor /kənˈstrʌktər/、Injection /ɪnˈdʒekʃən/ | 建構式注入 | [Constructor Injection](ASPNETCoreMVC_ConstructorInjection_建構式注入.md) |
| Tight Coupling | Tight /taɪt/、Coupling /ˈkʌplɪŋ/ | 緊耦合 | [DI 綜合運用](ASPNETCoreMVC_DI_依賴注入.md) |
| Unit Test | Unit /ˈjuːnɪt/、Test /test/ | 單元測試 | [DI 綜合運用](ASPNETCoreMVC_DI_依賴注入.md) |
| Configuration | /kənˌfɪɡjəˈreɪʃən/ | 組態設定 | [appsettings](ASPNETCoreMVC_appsettings_組態設定.md) |
| Configuration File | Configuration /kənˌfɪɡjəˈreɪʃən/、File /faɪl/ | 組態設定檔 | [appsettings](ASPNETCoreMVC_appsettings_組態設定.md) |
| appsettings.json | app /æp/、settings /ˈsetɪŋz/、JSON /ˈdʒeɪsən/ | ASP.NET Core 組態設定檔 | [appsettings](ASPNETCoreMVC_appsettings_組態設定.md) |
| JSON<br>JavaScript Object Notation | 縮寫念法：/ˈdʒeɪsən/<br>完整英文：JavaScript /ˈdʒɑːvəskrɪpt/、Object /ˈɑːbdʒekt/、Notation /noʊˈteɪʃən/ | JavaScript 物件表示法 | [appsettings](ASPNETCoreMVC_appsettings_組態設定.md) |
| API<br>Application Programming Interface | 縮寫念法：/ˌeɪ piː ˈaɪ/<br>完整英文：Application /ˌæplɪˈkeɪʃən/、Programming /ˈproʊɡræmɪŋ/、Interface /ˈɪntərfeɪs/ | 應用程式介面 | [appsettings](ASPNETCoreMVC_appsettings_組態設定.md) |
| Environment | /ɪnˈvaɪrənmənt/ | 環境 | [appsettings](ASPNETCoreMVC_appsettings_組態設定.md) |
| Environment Variable | Environment /ɪnˈvaɪrənmənt/、Variable /ˈveriəbl/ | 環境變數 | [appsettings](ASPNETCoreMVC_appsettings_組態設定.md) |
| Development | /dɪˈveləpmənt/ | 開發環境 | [appsettings](ASPNETCoreMVC_appsettings_組態設定.md) |
| Production | /prəˈdʌkʃən/ | 正式環境 | [appsettings](ASPNETCoreMVC_appsettings_組態設定.md) |
| IConfiguration<br>Interface Configuration | 縮寫念法：/aɪ kənˌfɪɡjəˈreɪʃən/<br>完整英文：Interface /ˈɪntərfeɪs/、Configuration /kənˌfɪɡjəˈreɪʃən/ | 讀取設定值的介面 | [appsettings](ASPNETCoreMVC_appsettings_組態設定.md) |
| Options Pattern | Options /ˈɑːpʃənz/、Pattern /ˈpætərn/ | 強型別設定模式 | [appsettings](ASPNETCoreMVC_appsettings_組態設定.md) |
| Strongly Typed | Strongly /ˈstrɔːŋli/、Typed /taɪpt/ | 強型別的 | [appsettings](ASPNETCoreMVC_appsettings_組態設定.md) |
| Connection String | Connection /kəˈnekʃən/、String /strɪŋ/ | 資料庫連線字串 | [appsettings](ASPNETCoreMVC_appsettings_組態設定.md) |
| Log Level | Log /lɔːɡ/、Level /ˈlevl/ | 日誌等級 | [appsettings](ASPNETCoreMVC_appsettings_組態設定.md) |
