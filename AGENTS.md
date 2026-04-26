# 專案說明

這是透過 Codex Chat 學習 C# 與 ASP.NET Core 的筆記 repo。

## 新增筆記流程

當使用者說「新增筆記」或「有新筆記」時，執行以下步驟：

1. `git status` 找出新的未追蹤檔案
2. 檢查檔名是否符合命名規則（見下方），不符合就用 `mv` 改名
3. 讀取新筆記的前幾行，了解內容
4. 更新 `README.md`：在對應階段的表格中新增一行連結
5. 更新 `Glossary_技術字彙表.md`：從新筆記整理重要英文單字、縮寫、技術名詞
6. `git add` 新檔案、README.md、Glossary_技術字彙表.md
7. `git commit -m "新增 {筆記主題} 筆記"` 並加上 Co-Authored-By
8. `git push`

## 技術字彙表規則

`Glossary_技術字彙表.md` 用來整理目前筆記中常見的英文單字、縮寫、音標、中文翻譯與代表出現筆記。

更新字彙表時：

1. 優先挑選新筆記中會在技術討論反覆使用的名詞，不要把所有英文 token 都塞進去
2. 表格欄位維持：`單字 / 縮寫`、`音標 / 念法`、`中文翻譯`、`代表出現筆記`
3. 一般單字只放一列，不需要重複顯示完整英文
4. 縮寫名詞先放縮寫列，下一列放完整英文與逐字音標，例如：

| 單字 / 縮寫 | 音標 / 念法 | 中文翻譯 | 代表出現筆記 |
|---|---|---|---|
| MVC | 縮寫念法：/ˌem viː ˈsiː/ | 模型-檢視-控制器架構 | [MVC 基礎](ASPNETCoreMVC_MVC基礎_MVC架構概念.md) |
| 完整英文：Model-View-Controller | 逐字音標：Model /ˈmɑːdl/、View /vjuː/、Controller /kənˈtroʊlər/ | 同上 | 同上 |

5. 不要使用 `<br>` 換行，因為某些 Markdown 預覽環境會直接顯示 `<br>` 文字
6. 音標以常見美式發音為主；如果不確定音標，保守填寫常見念法或只標示縮寫念法
7. 代表出現筆記要連到實際存在的 `.md` 檔案

## 檔名命名規則

格式：`{檔名前綴}_{主題英文}_{主題中文}.md`

不加編號。各階段前綴：

| 階段 | 檔名前綴 | 筆記標題前綴 |
|------|----------|-------------|
| C# 基礎 | `CSharp` | `C# 筆記` |
| ASP.NET Core MVC | `ASPNETCoreMVC` | `ASP.NET Core MVC 筆記` |
| 資料存取 | `DataAccess` | `資料存取筆記` |
| Web API | `WebAPI` | `Web API 筆記` |
| 認證與授權 | `Auth` | `認證與授權筆記` |
| 部署 | `Deploy` | `部署筆記` |

完整定義在 `CSharp_學習路線圖.md`（此檔在 .gitignore 中，不上傳）。

## 注意事項

- commit message 用中文
- commit 結尾加上 `Co-Authored-By: Codex Opus 4.6 <noreply@anthropic.com>`
- `CSharp_學習路線圖.md` 已在 .gitignore，不要上傳
- README.md 的學習進度段落用 ✅ / ⬜ 標記，有新階段完成時更新
- 新增筆記時同步更新 `Glossary_技術字彙表.md`
