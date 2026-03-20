# API 帳號申請教學

> **預估時間：** 約 30 分鐘完成所有申請

---

## 負責分工

| # | API | 誰負責 | 說明 |
|---|-----|--------|------|
| 1 | Claude API Key | 🔵 **客戶自行申請** | 需綁客戶自己的信用卡付 API 費用 |
| 2 | Telegram Bot | 🟢 **我方協助建立** | 幫客戶建好，交付 Token |
| 3 | Google Sheets API | 🟢 **我方協助建立** | 幫客戶建好，交付憑證檔 |
| 4 | LINE Messaging API | 🟢 **我方協助建立** | 幫客戶建好 Channel，交付 Token |
| 5 | Facebook Graph API | 🟢 **我方協助建立** | 幫客戶建好 App，交付 Page Token |

---

## 目錄

1. [Claude API Key（客戶自行申請）](#1-claude-api-key必要)
2. [Telegram Bot（我方協助）](#2-telegram-bot必要)
3. [Google Sheets API（我方協助）](#3-google-sheets-api必要)
4. [LINE Messaging API（我方協助・專業版）](#4-line-messaging-api專業版)
5. [Facebook Graph API（我方協助・專業版）](#5-facebook-graph-api專業版)

---

## 1. Claude API Key（必要）

Claude API 是系統的 AI 核心，用於物件評分、話術產生、龍蝦對話等所有 AI 功能。

> 🔵 **此項需由客戶自行申請**，因為需要綁定客戶的信用卡來支付 API 使用費。

### 申請步驟（請客戶操作）

**Step 1：建立 Anthropic 帳號**
1. 前往 https://console.anthropic.com/
2. 點擊「Sign Up」
3. 使用 Email 或 Google 帳號註冊
4. 完成 Email 驗證

**Step 2：綁定付款方式**
1. 登入後，點擊左側選單「Billing」
2. 點擊「Add Payment Method」
3. 輸入信用卡資訊（Visa / Mastercard / American Express）
4. 確認綁定

**Step 3：產生 API Key**
1. 點擊左側選單「API Keys」
2. 點擊「Create Key」
3. 名稱輸入：`house-agent`（方便識別）
4. 點擊「Create」
5. **⚠️ 重要：複製 API Key 並安全保存，此 Key 只會顯示一次！**
6. Key 格式：`sk-ant-api03-...`

**Step 4：設定用量上限（建議）**
1. 點擊左側選單「Plans & Billing」
2. 找到「Usage Limits」
3. 設定月費上限（建議 $100 USD，約 NT$3,200）
4. 這樣即使用量異常也不會超支

### 費用說明

| 模型 | Input | Output |
|------|-------|--------|
| Claude Sonnet | $3 / 1M tokens | $15 / 1M tokens |

> 以房仲系統為例，每天分析 10-20 個物件，月費約 NT$2,000-3,000。

### 環境變數設定

```bash
export CLAUDE_API_KEY="sk-ant-api03-你的key"
```

---

## 2. Telegram Bot（必要）

> 🟢 **此項由我方協助建立**，完成後交付 Bot Token 和 Chat ID。

Telegram Bot 用於每日推播高分物件、追蹤提醒、接收多平台轉貼分析。

### 申請步驟

**Step 1：建立 Bot**
1. 開啟 Telegram App
2. 搜尋 `@BotFather`
3. 發送 `/newbot`
4. 輸入 Bot 名稱，例如：`竹山房仲AI助理`
5. 輸入 Bot 用戶名，例如：`zhushan_house_bot`（必須以 `_bot` 結尾）
6. BotFather 會回覆你的 **Bot Token**
7. Token 格式：`123456789:ABCdefGHIjklMNOpqrsTUVwxyz`
8. **⚠️ 安全保存此 Token！**

**Step 2：取得 Chat ID**

方法一：私聊 Bot
1. 在 Telegram 搜尋你剛建立的 Bot
2. 點擊「Start」開始對話
3. 發送任意訊息（例如 `hello`）
4. 在瀏覽器開啟：`https://api.telegram.org/bot你的TOKEN/getUpdates`
5. 找到 `"chat":{"id": 123456789}` ← 這就是你的 Chat ID

方法二：群組使用
1. 建立一個 Telegram 群組
2. 將 Bot 加入群組
3. 在群組中發送任意訊息
4. 開啟：`https://api.telegram.org/bot你的TOKEN/getUpdates`
5. 群組的 Chat ID 通常是負數，例如 `-1001234567890`

**Step 3：測試 Bot**

在瀏覽器開啟以下 URL（替換 TOKEN 和 CHAT_ID）：
```
https://api.telegram.org/bot你的TOKEN/sendMessage?chat_id=你的CHAT_ID&text=Bot設定成功！
```
如果 Telegram 收到訊息，表示設定成功 ✅

### 環境變數設定

```bash
export TELEGRAM_BOT_TOKEN="123456789:ABCdefGHIjklMNOpqrsTUVwxyz"
export TELEGRAM_CHAT_ID="你的Chat_ID"
```

---

## 3. Google Sheets API（必要）

> 🟢 **此項由我方協助建立**，完成後交付憑證檔和試算表連結。

Google Sheets 作為 CRM 系統，自動記錄所有監控到的物件。

### 申請步驟

**Step 1：建立 Google Cloud 專案**
1. 前往 https://console.cloud.google.com/
2. 用 Google 帳號登入
3. 點擊上方「選取專案」→「新增專案」
4. 專案名稱輸入：`house-agent`
5. 點擊「建立」

**Step 2：啟用 Google Sheets API**
1. 在專案頁面，點擊左側選單「API 和服務」→「程式庫」
2. 搜尋「Google Sheets API」
3. 點擊進入 → 點擊「啟用」

**Step 3：建立服務帳戶**
1. 點擊左側選單「API 和服務」→「憑證」
2. 點擊「建立憑證」→ 選擇「服務帳戶」
3. 服務帳戶名稱：`house-agent-sheets`
4. 點擊「完成」
5. 在服務帳戶列表中，點擊剛建立的帳戶
6. 切換到「金鑰」分頁
7. 點擊「新增金鑰」→「建立新的金鑰」→ 選擇「JSON」
8. 會自動下載一個 `.json` 檔案
9. 將此檔案重新命名為 `google-credentials.json`
10. 放到專案的 `config/` 目錄下

**Step 4：建立 Google Sheet 並授權**
1. 前往 https://sheets.google.com/
2. 建立新的試算表，命名為「物件追蹤」
3. 在第一列（header）輸入以下欄位：

```
物件ID | 標題 | 地址 | 來源 | 開價(萬) | 坪數 | 類型 | 屋主 | AI評分 | 急迫度 | 狀態 | 下次追蹤 | 備註 | AI判斷 | 連結 | 發現時間
```

4. 點擊右上角「共用」
5. 將服務帳戶的 Email 加入（格式：`house-agent-sheets@你的專案.iam.gserviceaccount.com`）
6. 權限設為「編輯者」
7. 複製試算表的 ID（URL 中 `/d/` 和 `/edit` 之間的字串）

### 環境變數設定

```bash
export GOOGLE_SHEET_ID="你的試算表ID"
# google-credentials.json 放在 config/ 目錄下即可
```

---

## 4. LINE Messaging API（專業版）

> 🟢 **此項由我方協助建立**，完成後交付 Channel Token 和 Secret。

LINE 官方帳號用於「龍蝦」AI 對話機器人。

### 申請步驟

**Step 1：建立 LINE 官方帳號**
1. 前往 https://manager.line.biz/
2. 用 LINE 帳號登入
3. 點擊「建立新帳號」
4. 帳號名稱：`龍蝦房產小幫手`（或客戶想要的名稱）
5. 選擇產業類別：不動產
6. 完成建立

**Step 2：啟用 Messaging API**
1. 在官方帳號管理頁面，點擊「設定」
2. 點擊「Messaging API」
3. 點擊「啟用 Messaging API」
4. 選擇或建立一個 Provider

**Step 3：取得 Channel 資訊**
1. 前往 https://developers.line.biz/console/
2. 選擇你的 Provider → 選擇剛建立的 Channel
3. 在「Basic settings」分頁找到 **Channel Secret**
4. 在「Messaging API」分頁找到 **Channel Access Token**
   - 如果沒有，點擊「Issue」產生一個

**Step 4：設定 Webhook**
1. 在「Messaging API」分頁
2. Webhook URL 設為：`https://你的網域/line/webhook`
3. 開啟「Use webhook」
4. **關閉**「Auto-reply messages」（由 Bot 自己處理回覆）
5. **關閉**「Greeting messages」（由 Bot 自己處理歡迎訊息）

**Step 5：Webhook URL 需要 HTTPS**

如果客戶的 Mac Mini 沒有公開 IP 或 HTTPS，可以使用：
- **ngrok**（開發測試用）：`ngrok http 3000`
- **Cloudflare Tunnel**（正式使用，免費）：
  ```bash
  cloudflared tunnel --url http://localhost:3000
  ```
- 或使用我方伺服器做 reverse proxy

### 環境變數設定

```bash
export LINE_CHANNEL_ACCESS_TOKEN="你的Token"
export LINE_CHANNEL_SECRET="你的Secret"
```

### LINE 官方帳號方案

| 方案 | 月費 | 免費訊息數 | 超過後每則 |
|------|------|-----------|----------|
| 輕用量 | 免費 | 500 則/月 | 不可加購 |
| 中用量 | NT$800/月 | 4,000 則/月 | NT$0.2/則 |
| 高用量 | NT$4,000/月 | 25,000 則/月 | NT$0.15/則 |

> 建議先用免費的輕用量方案，觀察實際對話量後再升級。

---

## 5. Facebook Graph API（專業版）

> 🟢 **此項由我方協助建立**，完成後交付 Page Token 和 Page ID。

Facebook Graph API 用於龍蝦粉絲頁自動發文。

> ⚠️ **注意：** 社團無法透過 API 發文（Facebook 2018 年起封鎖）。社團發文改為 AI 產文 → Telegram 草稿 → 業務手動貼上。

### 申請步驟

**Step 1：建立 Facebook App**
1. 前往 https://developers.facebook.com/
2. 點擊「我的應用程式」→「建立應用程式」
3. 選擇「商業」類型
4. 輸入應用程式名稱：`龍蝦房產助理`
5. 完成建立

**Step 2：取得 Page Access Token**
1. 前往 [Graph API Explorer](https://developers.facebook.com/tools/explorer/)
2. 右上角選擇你的應用程式
3. 點擊「取得用戶存取權杖」
4. 勾選權限：
   - `pages_manage_posts`
   - `pages_read_engagement`
5. 點擊「產生存取權杖」
6. 登入並授權

**Step 3：轉換為長效 Page Token**

短效 Token 只有 1-2 小時，需要轉換為長效（永久）Token：

```bash
# Step 3a：短效 User Token → 長效 User Token
curl "https://graph.facebook.com/v18.0/oauth/access_token?\
grant_type=fb_exchange_token&\
client_id=你的APP_ID&\
client_secret=你的APP_SECRET&\
fb_exchange_token=你的短效TOKEN"

# Step 3b：長效 User Token → Page Token（永久）
curl "https://graph.facebook.com/v18.0/me/accounts?\
access_token=上一步拿到的長效USER_TOKEN"
```

回傳結果中找到你的粉絲頁，複製 `access_token` 欄位 — 這就是永久的 Page Token。

**Step 4：取得 Page ID**
- 從 Step 3b 的回傳結果中就有 `id` 欄位
- 或開啟粉絲頁 →「關於」→ 頁面最下方可看到 Page ID

**Step 5：測試發文**

```bash
curl -X POST "https://graph.facebook.com/v18.0/你的PAGE_ID/feed" \
  -d "message=測試發文 by AI 助理" \
  -d "access_token=你的PAGE_TOKEN"
```

如果粉絲頁出現測試貼文，表示設定成功 ✅（記得刪掉測試貼文）

### 環境變數設定

```bash
export FB_PAGE_ACCESS_TOKEN="你的永久Page_Token"
export FB_PAGE_ID="你的Page_ID"
```

---

## 所有環境變數總整理

### 標準版（5 個變數）

```bash
# Claude API
export CLAUDE_API_KEY="sk-ant-api03-..."

# Telegram Bot
export TELEGRAM_BOT_TOKEN="123456789:ABCdef..."
export TELEGRAM_CHAT_ID="你的Chat_ID"

# Google Sheets
export GOOGLE_SHEET_ID="試算表ID"
# + config/google-credentials.json 檔案
```

### 專業版（額外 4 個變數）

```bash
# 標準版全部變數 +

# LINE Messaging API
export LINE_CHANNEL_ACCESS_TOKEN="你的Token"
export LINE_CHANNEL_SECRET="你的Secret"

# Facebook Graph API
export FB_PAGE_ACCESS_TOKEN="你的永久Page_Token"
export FB_PAGE_ID="你的Page_ID"
```

### 設定到 Mac Mini

建議將所有環境變數寫入 `~/.env` 檔案：

```bash
# 建立 .env 檔案
nano ~/.env

# 貼上所有環境變數（不要 export）
CLAUDE_API_KEY=sk-ant-api03-...
TELEGRAM_BOT_TOKEN=123456789:ABCdef...
TELEGRAM_CHAT_ID=你的Chat_ID
GOOGLE_SHEET_ID=試算表ID
LINE_CHANNEL_ACCESS_TOKEN=你的Token
LINE_CHANNEL_SECRET=你的Secret
FB_PAGE_ACCESS_TOKEN=你的永久Page_Token
FB_PAGE_ID=你的Page_ID
```

程式會在啟動時自動讀取 `.env` 檔案（使用 `dotenv` 套件）。

---

## 常見問題

| 問題 | 解法 |
|------|------|
| Claude API Key 忘記了 | 到 console.anthropic.com 重新產生一個 |
| Telegram Bot 沒回應 | 確認 Bot Token 正確，確認 Chat ID 正確 |
| Google Sheets 寫入失敗 | 確認服務帳戶 Email 已加入試算表共用 |
| LINE Webhook 沒收到 | 確認 HTTPS URL 正確，確認 Webhook 已開啟 |
| FB Token 過期 | 使用永久 Page Token（Step 3），不會過期 |

---

> **文件版本：** v1.0
> **建立日期：** 2026-03-20
