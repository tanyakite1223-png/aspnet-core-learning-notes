# 專案說明

這是透過 Claude Chat 學習 C# 與 ASP.NET Core 的筆記 repo。

## 新增筆記流程

當使用者說「新增筆記」或「有新筆記」時，執行以下步驟：

1. `git status` 找出新的未追蹤檔案
2. 檢查檔名是否符合命名規則（見下方），不符合就用 `mv` 改名
3. 讀取新筆記的前幾行，了解內容
4. 更新 `README.md`：在對應階段的表格中新增一行連結
5. `git add` 新檔案和 README.md
6. `git commit -m "新增 {筆記主題} 筆記"` 並加上 Co-Authored-By
7. `git push`

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
- commit 結尾加上 `Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>`
- `CSharp_學習路線圖.md` 已在 .gitignore，不要上傳
- README.md 的學習進度段落用 ✅ / ⬜ 標記，有新階段完成時更新
