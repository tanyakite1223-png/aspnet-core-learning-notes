# 資料存取筆記:建立 ExpenseSystem 專案與 git 初始化

---

## 報銷系統的設計理由

選擇報銷系統作為實戰專題，原因如下：

- **業務真實**：公司實際使用的系統類型，不是純練習題
- **自然涵蓋 CRUD**：每張報銷單都需要新增、查詢、修改、刪除
- **有層次的資料模型**：一張報銷單（主檔）底下有多筆費用明細（明細檔）
- **自然需要權限控管**：員工、主管、出納、會計各有不同角色與操作範圍
- **可當履歷作品**：做完是一個真實可展示的系統

業務流程：員工上傳費用 → 長官簽核 → 出納 → 會計

---

## CLAUDE.md 的作用

Claude Code 在某個目錄啟動時，會自動讀取該目錄內的 `CLAUDE.md`。這份檔案是**給 Claude Code 的規範書**，告訴它：

- 這個專案是什麼
- 它在這個專案裡的角色是什麼
- 有哪些規定要遵守

不同目錄有各自的 `CLAUDE.md`，互不干擾。

---

## git 最低限度心智模型

git 管理程式碼有四個區域：

```
工作區           →      暫存區          →      本地 repo       →      遠端 repo
(你的資料夾)           (git add)              (git commit)           (git push)
Working Tree         Staging Area            Local Repo             Remote (GitHub)
```

**比喻：**
- 工作區：桌子，東西散在上面
- 暫存區：整理好、準備裝箱的東西
- 本地 repo：已封箱貼標籤、放進倉庫（存在 `.git\` 資料夾）
- 遠端 repo：把箱子寄到 GitHub 備份

**重要觀念：**
- commit 是存在自己電腦裡的，沒有 push，GitHub 上什麼都看不到
- `.git\` 是隱藏資料夾，只要它還在，整個 git 歷史就還在
- 暫存區存在 `.git\index` 檔案裡，不是你能打開看的資料夾

---

## 主要指令

| 指令 | 做什麼 |
|------|--------|
| `git init` | 在目前目錄建立 `.git\` 資料夾，初始化 repo |
| `git add .` | 把所有變更從工作區加到暫存區 |
| `git add 檔名` | 只把特定檔案加到暫存區 |
| `git commit -m "說明"` | 把暫存區的內容封存成一個 commit，存進本地 repo |
| `git push` | 把本地 repo 的 commit 推到遠端 repo（GitHub） |
| `git remote add origin URL` | 設定遠端 repo 的位址 |
| `git branch -M main` | 把目前分支重新命名為 main |
| `git push -u origin main` | 第一次 push，同時設定追蹤關係 |
| `git rm --cached 檔名` | 把已追蹤的檔案從 git 移除（不刪除本機檔案） |
| `git status` | 查看目前工作區、暫存區狀態 |
| `git log` | 查看 commit 歷史 |

---

## .gitignore 的作用

.gitignore 是一份黑名單，告訴 git 哪些檔案不要追蹤、不要 commit。

**為什麼需要它：**
- .NET 專案編譯時會產生 `bin\`、`obj\` 等大量暫存檔
- 這些是機器產生的，每個人自己編譯就有，不需要共享
- 沒有 .gitignore 會讓 repo 膨脹、下載慢、產生無意義的 git 衝突

**核心精神：只追蹤「人寫的東西」，不追蹤「機器產生的東西」。**

**注意：.gitignore 只對「還沒被追蹤」的檔案有效。已經 commit 過的檔案需要用 `git rm --cached` 明確移除。**

---

## HTTPS 認證流程

GitHub 不接受帳號密碼做 git 操作，需要使用 PAT（Personal Access Token，個人存取權杖）：

1. 第一次 push，終端機跳出視窗要求登入
2. 輸入 GitHub 帳號 + PAT（不是 GitHub 登入密碼）
3. Git Credential Manager（認證管理員）把認證資料存起來
4. 之後再 push，自動帶入，不需要重新輸入

---

## SSH key 的本質與好習慣

### SSH key 是什麼

SSH 私鑰是 GitHub 認可的身份證明，等同於密碼。拿到私鑰的人可以假冒你的身份對 GitHub 做任何操作。

- **私鑰**（`id_ed25519_xxx`，無副檔名）：絕對不能外傳
- **公鑰**（`id_ed25519_xxx.pub`）：可以放心貼到 GitHub

### 共用電腦的相處之道

這台電腦的 `~/.ssh/` 目錄裡有 Will 的私鑰。不要好奇打開、複製、外傳這些檔案——它是別人的身份證明。

### 為什麼現在用 HTTPS

HTTPS 設定簡單，第一次接觸 git 不需要同時學認證機制。不是因為 SSH 不安全，而是時機的問題。

### 未來建立 SSH key 的好習慣

- 使用 `ed25519`（現代演算法，比 RSA 更短更安全）
- 設定 passphrase（私鑰被偷時的第二道防線）
- 私鑰絕不離開本機（不放 OneDrive、Dropbox 等雲端同步目錄）
- 不把私鑰貼到任何聊天視窗、文件、email
- 驗證方式：`ssh -T git@github.com` 應回應 "Hi {username}"

---

## ExpenseSystem 目錄結構

```
D:\Workshops\Amber\ExpenseSystem\
├── .git\              ← git 大腦（暫存區 + 所有 commit 歷史）
├── .gitignore         ← .NET 標準範本 + tmpclaude-* 排除規則
├── CLAUDE.md          ← 實戰場 Claude Code 規範書
└── README.md          ← 專案說明（名稱：報銷系統，目的：取代人工單流程）
```

**GitHub repo：** `https://github.com/tanyakite1223-png/ExpenseSystem`（Private）
