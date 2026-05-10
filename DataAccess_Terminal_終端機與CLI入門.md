# 資料存取筆記:終端機與 Claude Code CLI 入門

---

## CLI 在職場的價值

CLI（Command Line Interface，命令列介面）不是因為「很酷」才用，而是有些事情只有它能做：

| 情境 | 說明 |
|------|------|
| **遠端操作伺服器** | 透過 SSH 連到伺服器，只有文字介面，沒有桌面可以點 |
| **自動化腳本** | 重複的步驟（備份、部署）可以寫成腳本，一行指令全部跑完 |
| **操作紀錄清楚** | 打了什麼指令、產生什麼結果，都是文字，可以貼給同事或記錄在文件裡 |

---

## Windows Terminal 與 PowerShell

- **Windows Terminal**：終端機容器，可以開多個分頁，每個分頁跑不同的 Shell
- **PowerShell**：真正接收並執行指令的程式（Shell，殼層）
- 本課程使用 **PowerShell 7**（PowerShell Core，跨平台版本），比 Windows 內建的 PowerShell 5 更現代

### 提示字元說明

```
Pinecone@DESKTOP-I0RM57E ~
```

| 部分 | 意思 |
|------|------|
| `Pinecone` | 目前登入的使用者名稱 |
| `DESKTOP-I0RM57E` | 電腦名稱 |
| `~` | 目前所在目錄（家目錄，即 `C:\Users\Pinecone`） |

---

## PowerShell 常用指令速查

| 指令 | 用途 | 範例 |
|------|------|------|
| `pwd` | 顯示目前所在目錄 | `pwd` |
| `ls` | 列出目錄內容 | `ls` |
| `cd 目錄` | 切換到指定目錄 | `cd D:\Workshops\Amber` |
| `mkdir 名稱` | 建立新目錄 | `mkdir Sandbox` |
| `code .` | 用 VS Code 開啟目前目錄 | `code .` |
| `Get-Content 檔名` | 讀取檔案內容（相當於 Linux 的 `cat`） | `Get-Content hello.txt` |

### 特殊符號

| 符號 | 意思 |
|------|------|
| `~` | 家目錄（`C:\Users\使用者名稱`） |
| `.` | 目前所在目錄 |

---

## Claude Code CLI

### 什麼是 Claude Code？

Claude Code 是一個在終端機裡執行的 AI 工具，可以直接讀取、建立、修改你電腦上的檔案，並執行指令。

### 啟動與結束

```powershell
# 啟動（在目標目錄下執行）
claude

# 結束
/exit
```

啟動時會詢問是否信任目前目錄，選「Yes, I trust this folder」才能繼續。

### Session 概念

- 每個 session 是**獨立的對話**，結束就結束
- 重新啟動後 Claude Code **不記得**上次 session 的內容
- 因此每次派任務時，必須把背景資訊一起帶進去

---

## 兩個 Claude 的角色分工

| | Project Chat（這裡） | Claude Code CLI |
|---|---|---|
| **在哪裡** | 瀏覽器 / Claude 桌面版 | PowerShell 終端機 |
| **能做什麼** | 對話、教學、產筆記、維護路線圖 | 讀寫電腦上的檔案、執行指令 |
| **看得到你的檔案嗎** | ❌ 看不到 | ✅ 看得到（在啟動的目錄內）|
| **角色** | 教室 | 實作場 |

**關鍵原則**：Project Chat 派任務 → Amber 貼到 Claude Code → Claude Code 引導實作 → 結束時回報給 Project Chat

---

## 安全提醒

- `~/.ssh/` 資料夾裡放的是 SSH 金鑰（私鑰），**不要隨便進去、不要刪除、不要複製給別人**
