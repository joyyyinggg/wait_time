# 等候時間看板 README

一個輕量的 RWD 網頁，讓你隨時用手機更新等候時間，客人掃 LINE 連結即可即時查看。

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

## 設定區（檔案底部 `<script>` 內）

用文字編輯器（記事本、VS Code 等）開啟 `waittime.html`，找到以下幾行修改：

```javascript
const SHEET_URL      = 'https://script.google.com/macros/s/...';  // Apps Script 網址
const ADMIN_PW       = 'a';       // ← 請改成你的密碼
const SHEET_EDIT_URL = '';        // ← 選填：貼入 Google 試算表的網址
const POLL_MS        = 10000;     // ← 自動更新間隔（毫秒），建議 10000 以上
```

> ⚠️ **安全提醒**：密碼目前是 `a`，請務必改成較複雜的密碼再正式上線。

---

## 自動更新間隔建議

設定太短會造成請求堆積並超出 Google Apps Script 免費額度：

| POLL_MS | 更新間隔 | 每日請求次數 | 說明 |
|---------|---------|------------|------|
| 3000    | 3 秒    | 28,800 次  | ⚠️ 超過免費額度（20,000 次/日） |
| 6000    | 6 秒    | 14,400 次  | ✅ 可用，但較緊繃 |
| 10000   | 10 秒   | 8,640 次   | ✅ 建議值 |
| 15000   | 15 秒   | 5,760 次   | ✅ 最穩定 |

程式內建**防重疊保護**：若上一個請求尚未回應，下一次輪詢會自動跳過，不會造成堆積。

---

## 錯誤通知（鈴鐺）說明

管理員登入後，若發生連線異常會出現以下提示：

| 位置 | 說明 |
|------|------|
| 卡片右上角 🔔 | 紅色角標顯示累計錯誤次數，點擊展開詳情 |
| 詳情浮層 | 顯示錯誤訊息與發生時間，可手動清除 |
| 管理面板頂部 | 紅色錯誤列顯示最近一次錯誤原因 |
| 底部狀態點 | 正常為綠色，異常時變紅色 |

> 鈴鐺需手動點「清除通知」才消失，確保管理員不會漏看。
> 觀看者畫面只會看到底部狀態點變紅，不會看到鈴鐺。

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
直接開啟網頁網址，畫面自動更新，不需要任何操作。

### 管理員（你）
1. 開啟同一個網頁網址
2. 點右上角 **⚙** 圖示
3. 輸入密碼
4. 調整等候時間（快速按鈕或手動輸入）
5. 填寫備註（選填）
6. 需要時開啟「暫停服務」開關
7. 按「儲存並更新 Google 試算表」

---

## 功能列表

- 大字體等候時間，手機遠看清晰
- 快速預設按鈕：5～80 分鐘
- 手動輸入任意分鐘數（0～999）
- 備註欄，支援換行
- 暫停服務開關
- 管理面板密碼保護
- **防請求重疊**：上一筆未回應時自動跳過，不堆積
- **鈴鐺錯誤通知**：連線異常時通知管理員，含錯誤訊息與時間
- 底部狀態點：正常綠色 / 異常紅色
- 關閉按鈕在底部，LINE 內建瀏覽器也點得到
- 每 N 秒自動刷新（可設定）
- 支援手機、平板、電腦（RWD）

---

## 部署到 GitHub Pages

1. 前往 [github.com](https://github.com) 註冊或登入
2. 點右上角「+」→「New repository」，名稱填 `waittime`，選 Public
3. 點「uploading an existing file」，上傳 `waittime.html`，按「Commit changes」
4. 進入「Settings」→「Pages」，Branch 選 `main`，資料夾選 `/ (root)`，按 Save
5. 等約 1 分鐘，網址為：

```
https://你的帳號名稱.github.io/waittime/waittime.html
```

---

## 已知限制

- 密碼儲存在 HTML 原始碼中，知道網址的人若查看原始碼可看到密碼。若有高度安全需求，建議改用後端驗證。
- Apps Script 免費版每日讀寫上限 20,000 次，建議 POLL_MS 設 10000 以上。
- 儲存時使用 `no-cors` 模式，無法直接確認寫入成功，但正常網路下均可運作。
