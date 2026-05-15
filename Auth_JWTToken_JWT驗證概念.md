# 認證與授權筆記:JWT Token 驗證（概念了解）

---

## 為什麼需要 JWT？

Cookie 驗證是瀏覽器專屬的功能，瀏覽器會自動帶上 Cookie。但手機 App 或後端 Server 呼叫 API 時，沒有瀏覽器，也沒有 Cookie 機制。

JWT（JSON Web Token）就是為了這個場景設計的——一種不依賴瀏覽器、任何 Client 都能使用的驗證方式。

---

## JWT 的三段結構

JWT 是一串文字，用兩個點（`.`）分成三段：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiQW1iZXIiLCJyb2xlIjoiVXNlciJ9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

| 段落 | 名稱 | 內容 |
|------|------|------|
| 第一段 | Header（標頭） | 使用哪種加密演算法，例如 `{"alg":"HS256"}` |
| 第二段 | Payload（內容） | 使用者資訊，JSON 格式 |
| 第三段 | Signature（簽名） | 用密鑰計算出來的驗證碼，防止內容被竄改 |

### Payload 範例

```json
{
  "name": "Amber",
  "role": "User",
  "exp": 1748000000
}
```

- `name`、`role`：使用者資訊
- `exp`：到期時間（expiration）

---

## Signature 的用途

Payload 的內容**沒有加密**，任何人拿到 JWT 都可以解碼看到裡面的資訊。

但 Signature 可以防止內容被竄改：

- Server 用自己保管的密鑰，對整個 JWT 內容計算出簽名
- 如果有人偷偷修改 Payload（例如把 `"role":"User"` 改成 `"role":"Admin"`）
- Server 收到後用密鑰重新計算簽名，和第三段比對
- 內容被改過，算出來的簽名一定對不上 → 拒絕請求

### 密鑰是什麼？

密鑰是 Server 自己私下保管的一串秘密字串，**從來不會放進 JWT 裡**，也不會傳給 Client。

驗證時，Server 拿 JWT 的第一段 + 第二段，加上自己的密鑰，丟進演算法重新計算：

```
第一段（Header）+ 第二段（Payload）+ 密鑰  →  算出結果  →  跟第三段比對
```

- 第三段 Signature 不參與計算，只用來比對答案
- 如果 Payload 被人改過，重新計算出來的結果就會跟第三段對不上

密鑰一直放在 Server，從來不給任何人。不管有一個使用者還是一百萬個使用者，都是同一把密鑰在驗。**使用者拿到的只是蓋了印的 JWT，不是密鑰本身。**

> 注意：「Server 不儲存登入狀態」和「Server 儲存密鑰」是兩件不同的事。密鑰是系統設定，不是使用者資料，不會隨使用者增加而變多。

---

## JWT 與 Cookie 驗證的差異

| | Cookie-based 驗證 | JWT 驗證 |
|---|---|---|
| 適用情境 | 網頁（瀏覽器） | API、手機 App |
| 資料存在哪 | Server 記住登入狀態 | Token 本身（Client 端） |
| 帶法 | 瀏覽器自動帶 Cookie | 工程師手動放進 Header |
| Server 需要記狀態嗎 | 需要 | 不需要，只驗簽名 |

---

## JWT 完整驗證流程

**登入階段（每次登入執行一次）：**

1. Client 送出帳號密碼
2. Server 驗證正確
3. Server 用密鑰產生 JWT，回傳給 Client
4. Client 自己存起來

**之後每次請求：**

5. Client 發 API 請求，手動在 Header 帶上 JWT
6. Server 用密鑰重算簽名，比對第三段
7. 符合 → 正常回應；不符合 → 拒絕請求

### HTTP Header 的帶法

```
Authorization: Bearer eyJhbGci...
```

`Bearer` 是固定寫法，後面接 JWT 內容。

---

## JWT 為什麼要設定期限？

JWT 一旦發出，Server 就無法主動讓它失效（因為 Server 不記狀態）。

如果 JWT 永遠有效，萬一手機被偷，拿到手機的人就可以一直用那組 JWT 呼叫 API，Server 無法分辨。

設定期限（`exp`）的用意：**就算 JWT 被偷，過了期限就自動失效，損害時間有限。**

JWT 過期後需要重新登入，取得新的 JWT。

---

## 本次未實作

本主題為概念了解，不進行程式碼實作。  
實作將在後續主題（`[Authorize]`、角色授權）中結合 ExpenseSystem 進行。
