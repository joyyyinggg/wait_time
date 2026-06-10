# 等候時間看板 README

一個輕量的 RWD 網頁，隨時用手機更新等候時間，客人掃 LINE 連結即可即時查看。

---

## 架構說明

```
waittime.html  ←→  Google Apps Script  ←→  Google 試算表
（前端網頁）         （API 中介層）            （資料儲存）
```

- 前端每 **10 秒**自動向 Google Apps Script 發出請求，取得最新資料並更新畫面。
- 管理員儲存時，同樣透過 Apps Script 將資料寫回試算表。
- 所有裝置讀取的都是同一份試算表，因此資料即時同步。

---

## 檔案說明

| 檔案 | 說明 |
|------|------|
| `waittime.html` | 主網頁，含觀看者畫面與管理員面板，只需這一個檔案 |

---

## 設定區（檔案頂部 `<script>` 內）

用文字編輯器（記事本、VS Code 等）開啟 `waittime.html`，找到以下三行修改：

```javascript
const SHEET_URL      = 'https://script.google.com/macros/s/...';  // Apps Script 網址
const ADMIN_PW       = '*';       // ← 請改成你的密碼(此處隱藏密碼)
const SHEET_EDIT_URL = '';        // ← 選填：貼入 Google 試算表的網址，管理面板會顯示快捷連結
```

> ⚠️ **安全提醒**：務必改成較複雜的密碼再正式上線。

---

## Google Apps Script 設定

Apps Script 需貼入以下程式碼，並以「所有人可存取」部署為網頁應用程式：

```javascript
function doGet(e) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const params = e.parameter;

  // 有帶參數時寫入
  if (params.minutes !== undefined) {
    sheet.getRange('A2').setValue(Number(params.minutes));
    sheet.getRange('B2').setValue(params.note || '');
    sheet.getRange('C2').setValue(params.closed === 'true');
  }

  // 永遠回傳目前值
  const data = {
    minutes: sheet.getRange('A2').getValue(),
    note:    sheet.getRange('B2').getValue(),
    closed:  sheet.getRange('C2').getValue()
  };

  return ContentService
    .createTextOutput(JSON.stringify(data))
    .setMimeType(ContentService.MimeType.JSON);
}
```

**部署步驟：**

1. 在試算表點「擴充功能」→「Apps Script」
2. 貼上上方程式碼並儲存
3. 點「部署」→「管理部署作業」→ 鉛筆圖示（編輯）
4. 版本選「新版本」
5. 執行身份：**我**；存取權：**所有人**
6. 按部署（網址不會改變）

> 每次修改 Apps Script 程式碼後，都要重新部署（選新版本）才會生效。

---

## Google 試算表格式

試算表第一行為欄位標題，第二行為實際資料：

| A | B | C |
|---|---|---|
| minutes | note | closed |
| 30 | 請稍候 | FALSE |

---

## 使用方式

### 觀看者（客人）
直接開啟網頁網址，畫面會自動每 10 秒刷新一次，不需要任何操作。

### 管理員（你）
1. 開啟同一個網頁網址
2. 點右上角 **⚙** 圖示
3. 輸入密碼
4. 調整等候時間（可點快速按鈕或手動輸入）
5. 填寫備註（選填）
6. 需要時開啟「暫停服務」開關
7. 按「儲存並更新 Google 試算表」

儲存後，所有觀看者在 10 秒內會自動看到新資訊。

---

## 功能列表

- 大字體等候時間顯示，手機遠看清晰
- 快速預設按鈕：5、10、15、20、25、30、35、40、45、50、55、60、70、80 分鐘
- 手動輸入任意分鐘數（0～999）
- 備註欄：可填寫任何說明文字，支援換行
- 暫停服務開關：開啟後畫面顯示「暫停服務」，隱藏分鐘數
- 管理面板密碼保護
- 點遮罩或 × 可關閉管理面板
- 每 10 秒自動刷新，顯示最後更新時間
- 支援手機、平板、電腦（RWD）

---

## 部署到 GitHub Pages（取得網址）

1. 前往 [github.com](https://github.com) 註冊或登入
2. 點右上角「+」→「New repository」，名稱填 `waittime`，選 Public
3. 點「uploading an existing file」，上傳 `waittime.html`，按「Commit changes」
4. 進入「Settings」→「Pages」，Branch 選 `main`，資料夾選 `/ (root)`，按 Save
5. 等約 1 分鐘，網址為：

```
https://你的帳號名稱.github.io/waittime/waittime.html
```

將此網址貼到 LINE 即可分享。

---

## 已知限制

- 密碼儲存在 HTML 原始碼中，知道網址的人若查看原始碼可看到密碼。若有高度安全需求，建議改用後端驗證機制。
- Apps Script 免費版每日讀寫上限為 20,000 次，一般使用場景完全足夠。
- 儲存時使用 `no-cors` 模式，無法直接判斷寫入是否成功，但正常網路環境下均可正常運作。
