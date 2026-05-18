# 工具筆記：Git 版本控制

---

## 1. 兩個重要標籤

執行 `git log --oneline` 時，你會看到這兩個標籤：

```
2d7e378 (HEAD -> main, origin/main) Role Claim 授權...
```

| 標籤 | 意思 |
|------|------|
| `HEAD -> main` | 你**本機**目前所在的位置（目前在哪個分支、哪個 commit） |
| `origin/main` | **GitHub 上**的 main 分支目前停在哪個 commit |

兩個標籤在同一個 commit 上 → 本機與 GitHub **已同步**。

---

## 2. 什麼是 repo（repository）

- repo 是「儲存庫」，一個會記住所有檔案歷史變更的資料夾
- 你有兩個 repo：
  - **本機 repo**：`D:\Workshops\Amber\ExpenseSystem\`
  - **遠端 repo（origin）**：GitHub 上的 `tanyakite1223-png/ExpenseSystem`

---

## 3. 為什麼需要分支（branch）

分支讓你在**不影響主線**的情況下，安全地開發新功能或修 bug。

**沒有分支的問題**：新功能寫到一半，`main` 就有半成品，無法上線。

**有分支的做法**：
- `main` 分支 = 正式版、穩定的程式碼，全程保持乾淨
- 新功能在獨立分支上開發，完成後再合併回 `main`

> 不是「複製整個資料夾」——分支是在同一個 repo 裡的平行線，git 自動管理版本切換與合併。

---

## 4. 分支常用指令

| 指令 | 說明 |
|------|------|
| `git branch` | 查詢所有分支（`*` 表示目前所在分支） |
| `git branch 名稱` | 建立新分支 |
| `git checkout 名稱` | 切換到指定分支 |
| `git merge 名稱` | 將指定分支合併到**目前所在分支** |
| `git branch -d 名稱` | 刪除分支（合併後才能刪） |

---

## 5. 完整工作流程

```
# 1. 建立新分支
git branch feature/export-excel

# 2. 切換過去
git checkout feature/export-excel

# 3. 開發、commit（可多次）
git add .
git commit -m "feat: 新增匯出 Excel 功能"

# 4. 切回 main
git checkout main

# 5. 合併
git merge feature/export-excel

# 6. 刪除已合併的分支
git branch -d feature/export-excel

# 7. 推到 GitHub
git push origin main
```

---

## 6. Fast-forward 合併

合併時若出現 `Fast-forward`，代表：
- `main` 在分支建立後**沒有新的 commit**
- git 直接把 `main` 往前推到分支的最新 commit
- Git Graph 上**不會出現分叉線**（因為兩條線從未真正分開）

若 `main` 和分支**各自都有新的 commit**，合併時才會出現真正的分叉線。

---

## 7. 本次實作摘要

在 ExpenseSystem 專案中：
1. 建立 `feature/export-excel` 分支
2. 在分支上修改檔案並 commit（`6aaaed7`）
3. 切回 `main` 後合併（Fast-forward）
4. 刪除 `feature/export-excel`
5. Push 到 GitHub（`2d7e378..6aaaed7`）

驗證結果：切換分支時，VS Code 檔案總管的內容會自動跟著變換。

---

## 8. PowerShell 常用 git 指令總整理

| 指令 | 說明 |
|------|------|
| `git status` | 查看目前工作區狀態 |
| `git log --oneline` | 查看 commit 記錄（簡短版） |
| `git add .` | 將所有修改加入暫存區 |
| `git commit -m "訊息"` | 建立一個 commit |
| `git push origin main` | 推送到 GitHub |
| `git branch` | 查詢所有分支 |
| `git branch 名稱` | 建立新分支 |
| `git checkout 名稱` | 切換分支 |
| `git merge 名稱` | 將指定分支合併到目前分支 |
| `git branch -d 名稱` | 刪除分支 |
