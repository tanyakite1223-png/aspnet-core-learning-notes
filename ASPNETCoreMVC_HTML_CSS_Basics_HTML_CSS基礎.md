# ASP.NET Core MVC 筆記：HTML / CSS 基礎（隨表單實作穿插學習）

---

## HTML 元素的兩大分類

HTML 元素分成兩大類：

### Block（區塊元素）
- **特性**：自己佔一整行，前後元素會被推到上下
- **常見標籤**：`<div>`、`<p>`、`<h1>`~`<h6>`、`<form>`
- 範例：
```html
<div>AAA</div>
<div>BBB</div>
<!-- AAA 和 BBB 會在不同行 -->
```

### Inline（行內元素）
- **特性**：不會換行，跟前後內容排在同一行
- **常見標籤**：`<span>`、`<a>`、`<strong>`、`<input>`
- 範例：
```html
<span>AAA</span>
<span>BBB</span>
<!-- AAA 和 BBB 會在同一行 -->
```

### 常用標籤整理
| 標籤 | 類型 | 用途 |
|------|------|------|
| `<div>` | Block | 區塊容器，用來包一組內容 |
| `<p>` | Block | 段落（paragraph） |
| `<h1>`~`<h6>` | Block | 標題，h1 最大、h6 最小 |
| `<span>` | Inline | 行內容器，用來標記一小段文字 |
| `<a>` | Inline | 超連結 |
| `<strong>` | Inline | 粗體強調 |

---

## CSS 是什麼？

CSS（Cascading Style Sheets，階層式樣式表）負責控制網頁的**外觀樣式**。

### HTML / CSS / JavaScript 的分工
| 技術 | 負責 | 比喻 |
|------|------|------|
| HTML | 結構 | 房子的牆壁、門、窗戶 |
| CSS | 樣式 | 房子的裝潢、顏色、材質 |
| JavaScript | 行為 | 房子的電器設備（開關、門鈴） |

---

## CSS 的三種寫法

### 1. Inline Style（行內樣式）
直接寫在 HTML 標籤的 `style` 屬性上：
```html
<h1 style="color: red; font-size: 50px;">標題</h1>
```
- 缺點：樣式散落在各處，難以維護

### 2. Internal Style（內部樣式）
用 `<style>` 標籤寫在同一個 HTML 檔案裡：
```html
<style>
    h1 {
        color: red;
        font-size: 50px;
    }
</style>
```
- 缺點：只對單一頁面有效

### 3. External Style（外部樣式）⭐ 最常用
寫在獨立的 `.css` 檔案，透過 `<link>` 標籤引入：
```html
<link rel="stylesheet" href="~/css/site.css" />
```
- 優點：**集中管理，改一次全站生效**
- 實務上最常使用的方式

### MVC 專案中的 CSS
- `wwwroot/css/site.css`：專案自訂的樣式檔
- 透過 `_Layout.cshtml` 裡的 `<link>` 標籤引入，所有共用 Layout 的頁面都會套用
- Bootstrap 的 CSS 也是以同樣方式引入（`bootstrap.min.css`）

---

## CSS 語法結構

```css
selector {
    property: value;
}
```

- **Selector（選取器）**：決定「對誰套用」
- **Property（屬性）**：決定「改什麼」
- **Value（值）**：決定「改成什麼」

### 常用 CSS 屬性
| 屬性 | 功能 | 範例 |
|------|------|------|
| `color` | 文字顏色 | `color: red;` |
| `background-color` | 背景顏色 | `background-color: yellow;` |
| `font-size` | 文字大小 | `font-size: 50px;` |

---

## CSS Selector（選取器）

### 1. 標籤選取器（Element Selector）
選取所有符合的標籤：
```css
h1 {
    color: red;
}
/* 頁面上所有 <h1> 都會變紅色 */
```

### 2. Class 選取器 ⭐ 最常用
用 `.` 開頭，選取有指定 class 的元素：
```css
.highlight {
    color: red;
}
```
```html
<h1 class="highlight">會變紅</h1>
<h1>不會變</h1>
```
- **可以重複使用**，多個元素可以有同一個 class
- 一個元素可以有多個 class，用空格隔開：`class="btn my-submit-btn"`

#### 多個 Class 的運作方式
當一個元素有多個 class 時，瀏覽器會**用空格拆開，逐一去 CSS 裡找對應的樣式**，找到的全部疊加套用：
```html
<button class="btn my-submit-btn">送出</button>
```
```css
/* Bootstrap 定義的 .btn → 提供基本按鈕樣式（圓角、padding、游標變手指） */
/* 你自訂的 .my-submit-btn → 額外加上綠色背景、白色文字 */
.my-submit-btn {
    background-color: green;
    color: white;
}
```
- `btn` 的樣式 ✅ 套用（Bootstrap 提供的基本外觀）
- `my-submit-btn` 的樣式 ✅ 也套用（你自訂的顏色）
- 兩者**同時生效**，不會衝突

> 💡 實務上通常會保留 Bootstrap 的 `btn` 來享受基本按鈕樣式，再加自己的 class 做額外客製化。如果拿掉 `btn`，按鈕會失去 Bootstrap 的圓角、padding 等細節，變成瀏覽器最原始的樣子。

### 3. ID 選取器
用 `#` 開頭，選取特定 ID 的元素：
```css
#main-title {
    color: blue;
}
```
```html
<h1 id="main-title">標題</h1>
```
- **不能重複**，整個頁面只能有一個元素用這個 ID

### Class vs ID 的差異
| 比較 | Class | ID |
|------|-------|-----|
| CSS 符號 | `.`（點） | `#`（井號） |
| 可重複使用 | ✅ 可以，多個元素共用 | ❌ 整頁只能一個 |
| 用途 | 大部分情況都適用 | 需要指定唯一元素時才用 |

---

## 容易搞錯的地方

### ❌ CSS class 選取器忘記加點
```css
my-submit-btn {
    color: red;
}
```

### ✅ CSS class 選取器要加 `.`
```css
.my-submit-btn {
    color: red;
}
```

### ❌ 搞混 `<p>` 的用途
`<p>` 不是「一行」，是「一個段落（paragraph）」，裡面可以有很多行文字。

### ❌ 搞混 CSS 和 JavaScript 的職責
- CSS → 控制外觀樣式（顏色、大小、排版）
- JavaScript → 控制互動行為（點擊事件、動態顯示隱藏）

### ❌ 直接修改 Bootstrap 的 class 樣式
如果要自訂按鈕外觀，不要直接改 `.btn` 的樣式（會影響全站所有按鈕），應該加自訂 class：
```html
<button class="btn my-submit-btn">送出</button>
```
```css
.my-submit-btn {
    background-color: green;
    color: white;
}
```
