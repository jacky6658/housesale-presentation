# AI 房仲智能助理 — 專業版建置手冊

> **方案定價：** NT$268,000（一次性建置費）
> **技術支援：** 驗收後 2 個月
> **預估工期：** 4-5 週（Week 1-2 核心 + Week 3 進階 + Week 4-5 LINE/FB）

---

## 目錄

1. [專案總覽](#1-專案總覽)
2. [技術棧與環境需求](#2-技術棧與環境需求)
3. [專案目錄結構](#3-專案目錄結構)
4. [設定檔規格](#4-設定檔規格)
5. [標準版模組（共用）](#5-標準版模組共用)
6. [模組六：多平台 AI 分析](#6-模組六多平台-ai-分析)
7. [模組七：多區域監控](#7-模組七多區域監控)
8. [模組八：AI 話術建議產生器](#8-模組八ai-話術建議產生器)
9. [模組九：每週物件彙整報告](#9-模組九每週物件彙整報告)
10. [模組十：客製化篩選條件](#10-模組十客製化篩選條件)
11. [模組十一：LINE 官方帳號 — 龍蝦 AI 對話機器人](#11-模組十一line-官方帳號--龍蝦-ai-對話機器人)
12. [模組十二：Facebook AI 發文](#12-模組十二facebook-ai-發文)
13. [OpenClaw Agent 整合（專業版）](#13-openclaw-agent-整合專業版)
14. [部署指南](#14-部署指南)
15. [驗收標準](#15-驗收標準)
16. [維護與故障排除](#16-維護與故障排除)

---

## 1. 專案總覽

### 功能範圍

| 功能 | 標準版 | 專業版 | 說明 |
|------|:------:|:------:|------|
| 591 自動監控 | ✓ | ✓ | 定時爬取指定區域 591 屋主自售物件 |
| AI 智能評分 | ✓ | ✓ | Claude AI 分析物件急迫度 0-100 分 |
| Telegram 推播 | ✓ | ✓ | 每日推播高分物件 |
| 追蹤提醒 | ✓ | ✓ | CRM 到期自動提醒 |
| Google Sheets CRM | ✓ | ✓ | 物件自動寫入 |
| 多平台 AI 分析 | ✗ | ✓ | FB/PTT/樂屋網 轉貼即評分 |
| 多區域監控 | ✗ | ✓ | 同時監控最多 5 個區域 |
| AI 話術產生器 | ✗ | ✓ | 電話/簡訊/LINE 話術建議 |
| 每週彙整報告 | ✗ | ✓ | 本週物件總結 + 趨勢分析 |
| 客製化篩選 | ✗ | ✓ | 價格/坪數/類型/關鍵字篩選 |
| LINE 龍蝦 AI 對話 | ✗ | ✓ | LINE 官方帳號 AI 客服 + 需求情蒐 |
| FB AI 發文 | ✗ | ✓ | 粉絲頁自動發佈（Graph API）+ 社團 AI 產文草稿 |

### 系統架構圖

```
┌───────────────────────────────────────────────────────────────┐
│                      OpenClaw Agent                            │
│            （主控排程 + 任務編排 + 多區域管理 + 對話管理）          │
└──┬──────────┬──────────┬──────────┬──────────┬───────────────┘
   │          │          │          │          │
   ▼          ▼          ▼          ▼          ▼
┌───────┐ ┌──────┐ ┌───────┐ ┌──────────┐ ┌──────────────────┐
│ 591   │ │ AI   │ │通知 & │ │ LINE     │ │  其他專業版模組    │
│ 爬蟲  │ │ 評分 │ │ CRM  │ │ 龍蝦 AI  │ │                  │
│       │ │ 引擎 │ │      │ │          │ │ - 多平台 AI 分析  │
│-多區域│ │      │ │- Tg  │ │- 客服接待 │ │ - AI 話術產生器   │
│-定時  │ │-Claude│ │-GSheet│ │- 需求情蒐│ │ - 每週彙整報告   │
│-篩選  │ │-評分 │ │- 提醒│ │- 情報回傳│ │ - 客製化篩選     │
└──┬────┘ └──┬───┘ └──┬───┘ └────┬─────┘ │ - FB 社團發文    │
   │         │        │          │        └────────┬─────────┘
   ▼         ▼        ▼          ▼                 ▼
┌───────────────────────────────────────────────────────────────┐
│                本地 SQLite + JSON 資料儲存                      │
└───────────────────────────────────────────────────────────────┘
   ▲                              ▲                    ▲
   │ 轉貼即分析                     │ 對話               │ 發文
┌──┴─────────────┐  ┌─────────────┴──────┐  ┌─────────┴──────┐
│ Telegram Bot   │  │ LINE 官方帳號       │  │ Facebook       │
│ (業務端)        │  │ (買方/賣方聊天)     │  │ Graph API      │
└────────────────┘  └────────────────────┘  │ (龍蝦社團)      │
                                            └────────────────┘
```

---

## 2. 技術棧與環境需求

### 技術棧（在標準版基礎上新增）

| 技術 | 版本 | 用途 | 新增 |
|------|------|------|:----:|
| Node.js | >= 18.x | 執行環境 | |
| TypeScript | >= 5.x | 開發語言 | |
| OpenClaw | latest | AI Agent 框架 | |
| Claude API | claude-sonnet-4-20250514 | AI 分析與評分 | |
| Telegram Bot API | - | 推播 + **接收轉貼** | ✓ |
| Google Sheets API | v4 | CRM 資料存取 | |
| node-cron | >= 3.x | 排程管理 | |
| Playwright | latest | 591 爬蟲 | |
| **cheerio** | latest | HTML 解析（樂屋網） | ✓ |
| **sharp** | latest | 截圖 OCR 前處理 | ✓ |

### 伺服器需求（專業版）

| 項目 | 規格 |
|------|------|
| CPU | 2-4 Core |
| RAM | 8 GB（多區域爬取同時運行） |
| 硬碟 | 40 GB SSD |
| 其他 | 同標準版 |

---

## 3. 專案目錄結構

```
house-agent-professional/
├── src/
│   ├── index.ts
│   ├── config.ts
│   ├── agent/
│   │   └── house-agent.ts
│   ├── crawler/
│   │   ├── crawler-591.ts          # 591 爬蟲（標準版）
│   │   ├── parser-591.ts
│   │   └── anti-ban.ts
│   ├── ai/
│   │   ├── scoring-engine.ts       # AI 評分（標準版）
│   │   ├── speech-generator.ts     # 【專業版】AI 話術產生器
│   │   └── prompts.ts              # Prompt 模板（含話術 prompt）
│   ├── telegram/
│   │   ├── bot.ts
│   │   ├── daily-push.ts
│   │   ├── tracker-reminder.ts
│   │   ├── multi-platform.ts       # 【專業版】多平台轉貼接收
│   │   └── templates.ts
│   ├── crm/
│   │   ├── google-sheets.ts
│   │   └── schema.ts
│   ├── report/
│   │   └── weekly-report.ts        # 【專業版】每週彙整報告
│   ├── filter/
│   │   └── custom-filter.ts        # 【專業版】客製化篩選
│   ├── multi-area/
│   │   └── area-manager.ts         # 【專業版】多區域管理
│   ├── line/
│   │   ├── lobster-bot.ts          # 【專業版】LINE 龍蝦 AI Bot
│   │   ├── conversation-manager.ts # 【專業版】對話狀態管理
│   │   ├── intent-analyzer.ts      # 【專業版】意圖分析 + 情蒐
│   │   └── lobster-prompts.ts      # 【專業版】龍蝦人設 Prompt
│   ├── facebook/
│   │   ├── fb-poster.ts            # 【專業版】FB 社團發文
│   │   ├── post-generator.ts       # 【專業版】AI 產文引擎
│   │   └── post-templates.ts       # 【專業版】發文模板
│   ├── storage/
│   │   ├── database.ts
│   │   └── models.ts
│   └── utils/
│       ├── logger.ts
│       └── helpers.ts
├── config/
│   ├── config.yaml
│   ├── config.example.yaml
│   ├── filters.yaml                # 【專業版】篩選條件設定
│   └── speech-samples.yaml         # 【專業版】話術範例庫
├── data/
├── logs/
├── package.json
├── tsconfig.json
└── README.md
```

---

## 4. 設定檔規格

### config.yaml（專業版完整版）

```yaml
# ===== 591 爬蟲設定 =====
crawler:
  # 【專業版】多區域監控（最多 5 個）
  areas:
    - county: "南投縣"
      district: "竹山鎮"
    - county: "南投縣"
      district: "鹿谷鄉"
    - county: "南投縣"
      district: "南投市"
    # 最多可設定 5 個

  interval_minutes: 30
  owner_only: true
  house_types: []

  proxy:
    enabled: false
    url: "http://proxy-server:port"
    rotate: true

# ===== Claude AI 設定 =====
ai:
  api_key: "${CLAUDE_API_KEY}"
  model: "claude-sonnet-4-20250514"
  score_threshold: 60
  batch_size: 20

# ===== Telegram 設定 =====
telegram:
  bot_token: "${TELEGRAM_BOT_TOKEN}"
  chat_id: "${TELEGRAM_CHAT_ID}"
  daily_push_time: "08:00"
  max_push_items: 5
  # 【專業版】啟用多平台接收
  multi_platform_enabled: true

# ===== Google Sheets CRM =====
google_sheets:
  credentials_path: "./config/google-credentials.json"
  spreadsheet_id: "${GOOGLE_SHEET_ID}"
  sheet_name: "物件追蹤"

# ===== 【專業版】話術產生器 =====
speech:
  # 話術範例檔案路徑
  samples_path: "./config/speech-samples.yaml"
  # 產生的話術類型
  types: ["phone", "sms", "line"]
  # 業務名稱（用於話術中）
  agent_name: "Jacky"
  # 公司名稱
  company_name: "有巢氏房屋"

# ===== 【專業版】每週報告 =====
weekly_report:
  enabled: true
  # 每週幾推送（1=週一, 7=週日）
  day_of_week: 1
  time: "09:00"

# ===== 【專業版】客製化篩選 =====
filters:
  config_path: "./config/filters.yaml"

# ===== 【專業版】LINE 龍蝦 AI =====
line:
  channel_access_token: "${LINE_CHANNEL_ACCESS_TOKEN}"
  channel_secret: "${LINE_CHANNEL_SECRET}"
  # 龍蝦人設
  persona:
    name: "龍蝦"
    personality: "親切、幽默、像鄰居朋友聊天"
    company: "有巢氏房屋"
  # 對話達到情蒐目標後通知業務
  notify_telegram: true
  # 對話超過幾輪後轉接真人
  handoff_after_rounds: 10
  # Webhook URL（需 HTTPS）
  webhook_url: "https://your-domain.com/line/webhook"

# ===== 【專業版】Facebook AI 發文 =====
facebook:
  page_access_token: "${FB_PAGE_ACCESS_TOKEN}"
  page_id: "${FB_PAGE_ID}"
  # 注意：社團無法透過 API 發文（FB 2018 年封鎖），社團改為 AI 產文草稿 → 業務手動發佈
  # 發文排程
  post_schedule:
    enabled: true
    # 每天幾點發文（可多個時間）
    times: ["10:00", "15:00"]
    # 每天最多發幾篇
    max_posts_per_day: 2
  # 發文類型權重
  post_types:
    property_listing: 40     # 物件推薦
    market_insight: 30       # 市場觀察
    tips: 20                 # 買賣小知識
    engagement: 10           # 互動型貼文

# ===== 系統設定 =====
system:
  timezone: "Asia/Taipei"
  log_level: "info"
  data_dir: "./data"
```

### filters.yaml（篩選條件設定）

```yaml
# 客製化篩選條件
# 符合任一組條件的物件會被保留

filters:
  # 篩選規則 1：高價值透天
  - name: "竹山透天"
    enabled: true
    conditions:
      house_type: ["透天"]
      price_min: 500       # 萬
      price_max: 2000      # 萬
      area_min: 30         # 坪
      keywords: []         # 標題關鍵字（空 = 不限）
      exclude_keywords: ["法拍", "公告"]

  # 篩選規則 2：低總價公寓
  - name: "低總價公寓"
    enabled: true
    conditions:
      house_type: ["公寓", "華廈"]
      price_min: 200
      price_max: 800
      area_min: 15
      keywords: []
      exclude_keywords: ["法拍"]

  # 篩選規則 3：土地
  - name: "土地投資"
    enabled: true
    conditions:
      house_type: ["土地"]
      price_min: 100
      price_max: 5000
      keywords: ["農地", "建地"]
      exclude_keywords: []
```

### speech-samples.yaml（話術範例庫）

```yaml
# 業務提供的成功話術範例
# AI 會參考這些範例的語氣和風格，產生類似的建議話術

samples:
  phone:
    - situation: "591 屋主自售"
      script: |
        您好，我是有巢氏房屋的 Jacky，
        看到您在 591 刊登的物件，
        想請問目前還有在出售嗎？
        我手上剛好有幾位客戶在找這一帶的房子...
    - situation: "急售物件"
      script: |
        您好，我是竹山在地的房仲 Jacky，
        看到您有出售的需求，
        我們在這區很活躍，買方資源也多，
        如果您希望快速成交，我可以幫您評估看看。

  sms:
    - situation: "初次聯繫"
      script: |
        您好，我是竹山在地房仲 Jacky，
        看到您有出售的需求，
        想了解一下狀況，方便聊聊嗎？

  line:
    - situation: "初次聯繫"
      script: |
        您好，我是竹山在地的房仲 Jacky。
        看到您有出售的需求，
        我可以免費幫您評估市場行情，
        有興趣的話隨時可以聊聊。
```

---

## 5. 標準版模組（共用）

專業版包含標準版全部 5 個模組，實作方式完全相同，請參考 **[standard-guide.md](./standard-guide.md)** 中的以下章節：

- **模組一：591 爬蟲監控引擎** → 同標準版（專業版增加多區域管理，見模組七）
- **模組二：AI 評分引擎** → 同標準版（專業版增加多平台分析 prompt）
- **模組三：Telegram Bot 推播** → 同標準版（專業版增加轉貼接收）
- **模組四：Telegram 追蹤提醒** → 完全同標準版
- **模組五：Google Sheets CRM** → 同標準版（專業版增加「來源」欄位值：FB轉貼/PTT）

---

## 6. 模組六：多平台 AI 分析

### 功能說明

業務在 Facebook 社團、PTT 房版、樂屋網等平台看到有興趣的物件，將連結或截圖轉貼到 Telegram Bot，AI 在 10 秒內回覆評分和建議。

### 運作流程

```
業務看到 FB 社團貼文
        │
        ▼
貼連結/截圖到 Telegram Bot
        │
        ▼
Bot 接收 → 判斷輸入類型
        │
  ┌─────┼─────┐
  ▼     ▼     ▼
連結   文字   截圖
  │     │     │
  ▼     ▼     ▼
解析   直接   OCR
網頁   分析   辨識
  │     │     │
  └─────┼─────┘
        ▼
  Claude AI 分析
  (評分 + 話術建議)
        │
        ▼
  回覆到 Telegram
  + 寫入 CRM
```

### Telegram Bot 接收轉貼

```typescript
// src/telegram/multi-platform.ts

import TelegramBot from 'node-telegram-bot-api';
import { ScoringEngine } from '../ai/scoring-engine';
import { SpeechGenerator } from '../ai/speech-generator';
import { GoogleSheetsCRM } from '../crm/google-sheets';
import { logger } from '../utils/logger';

export class MultiPlatformAnalyzer {
  private bot: TelegramBot;
  private scorer: ScoringEngine;
  private speechGen: SpeechGenerator;
  private crm: GoogleSheetsCRM;
  private authorizedChatId: string;

  constructor(
    bot: TelegramBot,
    scorer: ScoringEngine,
    speechGen: SpeechGenerator,
    crm: GoogleSheetsCRM,
    chatId: string,
  ) {
    this.bot = bot;
    this.scorer = scorer;
    this.speechGen = speechGen;
    this.crm = crm;
    this.authorizedChatId = chatId;
  }

  async init(): Promise<void> {
    // 監聽所有訊息
    this.bot.on('message', async (msg) => {
      // 安全檢查：只回應授權的 chat
      if (msg.chat.id.toString() !== this.authorizedChatId) {
        return;
      }

      try {
        await this.handleMessage(msg);
      } catch (error) {
        logger.error(`Multi-platform analysis failed: ${error}`);
        await this.bot.sendMessage(msg.chat.id, '❌ 分析失敗，請稍後再試');
      }
    });

    logger.info('Multi-platform analyzer initialized (polling mode)');
  }

  private async handleMessage(msg: TelegramBot.Message): Promise<void> {
    const chatId = msg.chat.id;

    // 判斷輸入類型
    if (msg.photo) {
      // 截圖 → OCR 辨識後分析
      await this.bot.sendMessage(chatId, '📸 收到截圖，AI 分析中...');
      await this.handleScreenshot(chatId, msg);

    } else if (msg.text) {
      const text = msg.text;

      // 判斷是否為連結
      const urlMatch = text.match(/https?:\/\/[^\s]+/);

      if (urlMatch) {
        const url = urlMatch[0];
        const platform = this.detectPlatform(url);
        await this.bot.sendMessage(chatId, `🔗 偵測到 ${platform} 連結，AI 分析中...`);
        await this.handleUrl(chatId, url, platform);
      } else {
        // 純文字 → 直接分析
        await this.bot.sendMessage(chatId, '📝 收到文字，AI 分析中...');
        await this.handleText(chatId, text);
      }
    }
  }

  private detectPlatform(url: string): string {
    if (url.includes('facebook.com') || url.includes('fb.com')) return 'Facebook';
    if (url.includes('ptt.cc')) return 'PTT';
    if (url.includes('rakuya.com.tw')) return '樂屋網';
    if (url.includes('591.com.tw')) return '591';
    return '其他平台';
  }

  private async handleUrl(chatId: number, url: string, platform: string): Promise<void> {
    // 根據平台解析內容
    let content: string;

    switch (platform) {
      case 'PTT':
        content = await this.parsePttUrl(url);
        break;
      case '樂屋網':
        content = await this.parseRakuyaUrl(url);
        break;
      default:
        // Facebook 等平台無法爬取，請用截圖或複製文字
        content = `來源連結：${url}\n（無法直接讀取此平台內容，請複製貼文文字或截圖傳送）`;
        await this.bot.sendMessage(chatId,
          `⚠️ ${platform} 無法直接讀取，建議：\n` +
          `1. 複製貼文文字直接傳過來\n` +
          `2. 截圖傳過來\n\n` +
          `AI 就能幫你分析！`
        );
        return;
    }

    // AI 分析
    const analysis = await this.analyzeContent(content, platform, url);

    // 回覆結果
    await this.bot.sendMessage(chatId, analysis.message, { parse_mode: 'Markdown' });

    // 寫入 CRM
    if (analysis.score >= 50) {
      await this.crm.appendExternalProperty({
        source: platform,
        content,
        score: analysis.score,
        analysis: analysis,
        url,
      });
    }
  }

  private async handleText(chatId: number, text: string): Promise<void> {
    // 直接分析文字內容
    const analysis = await this.analyzeContent(text, '手動轉貼', '');
    await this.bot.sendMessage(chatId, analysis.message, { parse_mode: 'Markdown' });

    if (analysis.score >= 50) {
      await this.crm.appendExternalProperty({
        source: '手動轉貼',
        content: text,
        score: analysis.score,
        analysis,
        url: '',
      });
    }
  }

  private async handleScreenshot(chatId: number, msg: TelegramBot.Message): Promise<void> {
    // 取得最大尺寸的圖片
    const photos = msg.photo!;
    const largestPhoto = photos[photos.length - 1];
    const fileLink = await this.bot.getFileLink(largestPhoto.file_id);

    // 使用 Claude Vision 直接分析圖片
    const analysis = await this.analyzeImage(fileLink);
    await this.bot.sendMessage(chatId, analysis.message, { parse_mode: 'Markdown' });

    if (analysis.score >= 50) {
      await this.crm.appendExternalProperty({
        source: '截圖分析',
        content: analysis.extractedText,
        score: analysis.score,
        analysis,
        url: '',
      });
    }
  }

  private async analyzeContent(
    content: string,
    platform: string,
    url: string,
  ): Promise<AnalysisResult> {
    const prompt = this.buildMultiPlatformPrompt(content, platform);

    const response = await this.scorer.client.messages.create({
      model: this.scorer.model,
      max_tokens: 1500,
      messages: [{ role: 'user', content: prompt }],
    });

    const text = response.content[0].type === 'text' ? response.content[0].text : '';
    const result = JSON.parse(text.match(/\{[\s\S]*\}/)![0]);

    // 組合回覆訊息
    const scoreEmoji = result.score >= 85 ? '🔴' : result.score >= 70 ? '🟡' : '🔵';
    const message =
      `${scoreEmoji} *AI 分析結果*\n\n` +
      `📊 評分：*${result.score} 分*\n` +
      `📌 急迫度：${result.urgency}\n` +
      `💰 價格分析：${result.price_analysis}\n` +
      `📝 判斷：${result.reason}\n` +
      `👉 建議：*${result.recommendation}*\n\n` +
      (result.suggested_speech ?
        `━━━━━━━━━━━━━━\n` +
        `💬 *建議話術：*\n${result.suggested_speech}\n` : '') +
      `\n_來源：${platform}_`;

    return {
      score: result.score,
      message,
      extractedText: content,
      ...result,
    };
  }

  private async analyzeImage(imageUrl: string): Promise<AnalysisResult> {
    // 使用 Claude Vision API 分析截圖
    const response = await this.scorer.client.messages.create({
      model: this.scorer.model,
      max_tokens: 2000,
      messages: [{
        role: 'user',
        content: [
          {
            type: 'image',
            source: { type: 'url', url: imageUrl },
          },
          {
            type: 'text',
            text: `你是台灣房仲業務專家。這張截圖是從房屋社團或售屋平台截取的。
請分析這個物件資訊，回傳 JSON：
{
  "extracted_text": "<從截圖中辨識出的所有文字>",
  "score": <0-100>,
  "urgency": "<急售|可能急售|一般|觀望>",
  "price_analysis": "<價格分析>",
  "recommendation": "<建議行動>",
  "reason": "<判斷理由>",
  "suggested_speech": "<建議電話開場白>"
}
只回傳 JSON。`,
          },
        ],
      }],
    });

    const text = response.content[0].type === 'text' ? response.content[0].text : '';
    const result = JSON.parse(text.match(/\{[\s\S]*\}/)![0]);

    const scoreEmoji = result.score >= 85 ? '🔴' : result.score >= 70 ? '🟡' : '🔵';
    const message =
      `${scoreEmoji} *截圖 AI 分析結果*\n\n` +
      `📊 評分：*${result.score} 分*\n` +
      `📌 急迫度：${result.urgency}\n` +
      `💰 價格分析：${result.price_analysis}\n` +
      `📝 判斷：${result.reason}\n` +
      `👉 建議：*${result.recommendation}*\n\n` +
      `💬 *建議話術：*\n${result.suggested_speech}\n\n` +
      `_來源：截圖分析_`;

    return {
      score: result.score,
      message,
      extractedText: result.extracted_text,
      ...result,
    };
  }

  private buildMultiPlatformPrompt(content: string, platform: string): string {
    return `你是台灣房仲業務專家。以下是從 ${platform} 取得的物件資訊，請分析。

## 貼文內容
${content}

## 請回傳 JSON：
{
  "score": <0-100>,
  "urgency": "<急售|可能急售|一般|觀望>",
  "owner_type_analysis": "<是否為屋主自售？判斷依據>",
  "price_analysis": "<價格分析>",
  "key_signals": ["<關鍵訊號>"],
  "recommendation": "<建議行動>",
  "reason": "<判斷理由>",
  "suggested_speech": "<針對此物件的建議電話開場白（一段話）>"
}
只回傳 JSON。`;
  }

  private async parsePttUrl(url: string): Promise<string> {
    // PTT 網頁版可直接抓取
    const response = await fetch(url, {
      headers: {
        'Cookie': 'over18=1',  // PTT 18+ 確認
      },
    });
    const html = await response.text();
    const cheerio = await import('cheerio');
    const $ = cheerio.load(html);
    return $('#main-content').text().trim();
  }

  private async parseRakuyaUrl(url: string): Promise<string> {
    const response = await fetch(url);
    const html = await response.text();
    const cheerio = await import('cheerio');
    const $ = cheerio.load(html);
    // 解析樂屋網物件頁面
    const title = $('h1').text().trim();
    const price = $('.price').text().trim();
    const info = $('.info-table').text().trim();
    return `標題：${title}\n價格：${price}\n${info}`;
  }
}

interface AnalysisResult {
  score: number;
  message: string;
  extractedText: string;
  [key: string]: any;
}
```

### 支援的輸入方式

| 輸入方式 | 處理邏輯 | 回應時間 |
|---------|---------|---------|
| 貼 PTT 連結 | 自動抓取 PTT 內容 → AI 分析 | ~5 秒 |
| 貼樂屋網連結 | 自動抓取頁面 → AI 分析 | ~5 秒 |
| 貼 Facebook 連結 | 提示用戶改用文字/截圖 | 即時 |
| 複製貼文文字 | 直接 AI 分析 | ~3 秒 |
| 截圖 | Claude Vision 辨識 + 分析 | ~8 秒 |

---

## 7. 模組七：多區域監控

### 功能說明

同時監控最多 5 個區域的 591 物件，每個區域獨立排程、獨立記錄。

### 實作邏輯

```typescript
// src/multi-area/area-manager.ts

import { Crawler591 } from '../crawler/crawler-591';
import { ScoringEngine } from '../ai/scoring-engine';
import { logger } from '../utils/logger';

interface AreaConfig {
  county: string;
  district: string;
}

export class AreaManager {
  private areas: AreaConfig[];
  private crawler: Crawler591;
  private scorer: ScoringEngine;

  constructor(areas: AreaConfig[], crawler: Crawler591, scorer: ScoringEngine) {
    if (areas.length > 5) {
      throw new Error('最多支援 5 個監控區域');
    }
    this.areas = areas;
    this.crawler = crawler;
    this.scorer = scorer;
  }

  async crawlAllAreas(): Promise<CrawlResult[]> {
    const results: CrawlResult[] = [];

    // 依序爬取每個區域（避免同時發送太多請求）
    for (const area of this.areas) {
      logger.info(`Crawling area: ${area.county} ${area.district}`);

      try {
        const properties = await this.crawler.crawlArea(area);

        // 評分
        const scores = await this.scorer.scoreBatch(properties);

        results.push({
          area,
          properties,
          scores,
          error: null,
        });

      } catch (error) {
        logger.error(`Failed to crawl ${area.county} ${area.district}: ${error}`);
        results.push({
          area,
          properties: [],
          scores: new Map(),
          error: error as Error,
        });
      }

      // 區域間間隔 1-3 分鐘
      const delay = Math.floor(Math.random() * 120000) + 60000;
      await new Promise(resolve => setTimeout(resolve, delay));
    }

    return results;
  }

  // 取得各區域統計
  getAreaStats(): AreaStats[] {
    // 從資料庫查詢各區域的物件數量和平均分數
    return this.areas.map(area => ({
      area,
      totalProperties: 0,  // 從 DB 查詢
      avgScore: 0,
      newThisWeek: 0,
    }));
  }
}

interface CrawlResult {
  area: AreaConfig;
  properties: Property591[];
  scores: Map<string, ScoringResult>;
  error: Error | null;
}
```

### 區域管理設定介面

業務可透過 Telegram Bot 指令管理區域：

```
/areas           → 列出目前監控的區域
/add_area 南投縣 草屯鎮  → 新增區域
/remove_area 3   → 移除第 3 個區域
```

---

## 8. 模組八：AI 話術建議產生器

### 功能說明

根據物件資訊和業務提供的成功話術範例，AI 產生電話/簡訊/LINE 三種話術建議。

### Prompt 設計

```typescript
// src/ai/speech-generator.ts

import Anthropic from '@anthropic-ai/sdk';
import { logger } from '../utils/logger';

export class SpeechGenerator {
  private client: Anthropic;
  private model: string;
  private config: SpeechConfig;
  private samples: SpeechSamples;

  constructor(client: Anthropic, model: string, config: SpeechConfig, samples: SpeechSamples) {
    this.client = client;
    this.model = model;
    this.config = config;
    this.samples = samples;
  }

  async generateSpeech(
    property: Property591,
    scoring: ScoringResult,
  ): Promise<SpeechSuggestion> {
    const prompt = this.buildPrompt(property, scoring);

    const response = await this.client.messages.create({
      model: this.model,
      max_tokens: 2000,
      messages: [{ role: 'user', content: prompt }],
    });

    const text = response.content[0].type === 'text' ? response.content[0].text : '';
    return JSON.parse(text.match(/\{[\s\S]*\}/)![0]);
  }

  private buildPrompt(property: Property591, scoring: ScoringResult): string {
    // 組合範例話術作為 few-shot
    const phoneExamples = this.samples.phone
      .map(s => `情境：${s.situation}\n話術：${s.script}`)
      .join('\n\n');

    const smsExamples = this.samples.sms
      .map(s => `情境：${s.situation}\n話術：${s.script}`)
      .join('\n\n');

    const lineExamples = this.samples.line
      .map(s => `情境：${s.situation}\n話術：${s.script}`)
      .join('\n\n');

    return `你是房仲話術專家。請根據以下物件資訊和 AI 分析結果，產生 3 種聯繫話術。

## 業務資訊
- 業務名稱：${this.config.agent_name}
- 公司名稱：${this.config.company_name}

## 物件資訊
- 標題：${property.title}
- 地址：${property.address}
- 開價：${property.price} 萬
- 類型：${property.houseType}
- 來源：591 屋主自售

## AI 分析結果
- 評分：${scoring.score} 分
- 急迫度：${scoring.urgency}
- 判斷：${scoring.reason}
- 關鍵訊號：${scoring.key_signals.join('、')}

## 過去成功的話術範例（請參考語氣和風格）

### 電話話術範例
${phoneExamples}

### 簡訊話術範例
${smsExamples}

### LINE 話術範例
${lineExamples}

## 請產生以下 JSON：
{
  "phone": "<電話開場白，約 3-4 句話>",
  "sms": "<簡訊內容，簡短 2-3 句>",
  "line": "<LINE 訊息，稍微長一些 3-4 句>",
  "tips": "<AI 提醒：這個物件聯繫時要注意什麼>"
}

注意：
1. 語氣要自然、親切，不要太制式
2. 要參考範例的風格，產生「類似但不完全一樣」的話術
3. 根據物件的急迫度調整話術策略：
   - 急售：強調「快速成交」「買方資源充足」
   - 一般：強調「免費評估」「在地服務」
4. 只回傳 JSON`;
  }
}

interface SpeechSuggestion {
  phone: string;
  sms: string;
  line: string;
  tips: string;
}
```

### 話術回覆格式（Telegram）

```
📞 *針對「延平路透天」的建議話術*

*【電話版】*
「您好，我是有巢氏房屋的 Jacky，看到您在 591 刊登延平路的透天，
想請問目前還有在出售嗎？我手上剛好有幾位客戶在找這一帶的透天...」

*【簡訊/私訊版】*
「您好，我是竹山在地的房仲 Jacky，看到您有出售的需求，
想了解一下狀況，方便聊聊嗎？」

*【LINE 版】*
「您好，我是竹山在地的房仲 Jacky。看到您有出售的需求，
我可以免費幫您評估市場行情，有興趣的話隨時可以聊聊。」

⚠️ *AI 提醒：* 這位屋主標題寫「急售」，建議強調「快速成交」和「買方資源」，
不要一開始就談價格。
```

---

## 9. 模組九：每週物件彙整報告

### 功能說明

每週一早上自動產生本週物件彙整報告，包含新物件數量、價格區間、重點摘要，推播到 Telegram。

### 實作邏輯

```typescript
// src/report/weekly-report.ts

import cron from 'node-cron';
import { HouseBot } from '../telegram/bot';
import { db } from '../storage/database';
import { ScoringEngine } from '../ai/scoring-engine';
import { logger } from '../utils/logger';

export class WeeklyReport {
  private bot: HouseBot;
  private scorer: ScoringEngine;

  constructor(bot: HouseBot, scorer: ScoringEngine) {
    this.bot = bot;
    this.scorer = scorer;
  }

  scheduleWeeklyReport(dayOfWeek: number, time: string): void {
    const [hour, minute] = time.split(':');

    // dayOfWeek: 1=Mon, 7=Sun
    cron.schedule(`${minute} ${hour} * * ${dayOfWeek}`, async () => {
      logger.info('Generating weekly report...');
      try {
        const report = await this.generateReport();
        await this.bot.sendMessage(report);
        logger.info('Weekly report sent');
      } catch (error) {
        logger.error(`Weekly report failed: ${error}`);
      }
    }, {
      timezone: 'Asia/Taipei',
    });
  }

  async generateReport(): Promise<string> {
    const now = new Date();
    const weekAgo = new Date(now.getTime() - 7 * 24 * 60 * 60 * 1000);

    // 查詢本週數據
    const stats = await db.getWeeklyStats(weekAgo, now);

    // 各區域統計
    const areaStats = await db.getWeeklyAreaStats(weekAgo, now);

    // 高分物件 TOP 5
    const topProperties = await db.getTopScoredProperties({
      minScore: 0,
      limit: 5,
      since: weekAgo,
    });

    // 使用 AI 產生趨勢分析
    const trendAnalysis = await this.generateTrendAnalysis(stats, areaStats);

    // 組合報告
    let report = `📊 *【${formatDate(weekAgo)} ~ ${formatDate(now)} 週報】*\n\n`;

    report += `📈 *本週數據總覽*\n`;
    report += `• 新物件數量：*${stats.totalNew}* 個\n`;
    report += `• 高分物件 (≥70分)：*${stats.highScoreCount}* 個\n`;
    report += `• 平均 AI 評分：*${stats.avgScore.toFixed(1)}* 分\n`;
    report += `• 平均開價：*${stats.avgPrice.toFixed(0)}* 萬\n\n`;

    report += `📍 *各區域統計*\n`;
    for (const area of areaStats) {
      report += `• ${area.county}${area.district}：${area.newCount} 個新物件，均價 ${area.avgPrice.toFixed(0)} 萬\n`;
    }
    report += `\n`;

    report += `🏆 *本週 TOP 5 高分物件*\n`;
    topProperties.forEach((prop, i) => {
      report += `${i + 1}. ${prop.title} (${prop.score}分) - ${prop.price}萬\n`;
    });
    report += `\n`;

    report += `🔍 *AI 趨勢分析*\n`;
    report += trendAnalysis;

    report += `\n━━━━━━━━━━━━━━\n`;
    report += `_每週自動產生 by AI 房仲助理_`;

    return report;
  }

  private async generateTrendAnalysis(
    stats: WeeklyStats,
    areaStats: AreaWeeklyStats[],
  ): Promise<string> {
    const prompt = `你是房仲市場分析師。根據以下本週數據，寫一段簡短的趨勢分析（3-4 句話）。

本週新物件：${stats.totalNew} 個
高分物件：${stats.highScoreCount} 個
平均評分：${stats.avgScore} 分
平均開價：${stats.avgPrice} 萬
上週新物件：${stats.lastWeekTotal} 個
上週平均價：${stats.lastWeekAvgPrice} 萬

各區域：
${areaStats.map(a => `${a.county}${a.district}: ${a.newCount}個, 均價${a.avgPrice}萬`).join('\n')}

只回傳分析文字，不要 JSON。用繁體中文。`;

    const response = await this.scorer.client.messages.create({
      model: this.scorer.model,
      max_tokens: 500,
      messages: [{ role: 'user', content: prompt }],
    });

    return response.content[0].type === 'text' ? response.content[0].text : '';
  }
}

function formatDate(date: Date): string {
  return `${date.getMonth() + 1}/${date.getDate()}`;
}
```

---

## 10. 模組十：客製化篩選條件

### 功能說明

根據業務設定的篩選條件（價格、坪數、房屋類型、關鍵字），在爬取後自動過濾，只保留符合條件的物件。

### 實作邏輯

```typescript
// src/filter/custom-filter.ts

import yaml from 'js-yaml';
import fs from 'fs';
import { logger } from '../utils/logger';

interface FilterRule {
  name: string;
  enabled: boolean;
  conditions: {
    house_type: string[];
    price_min: number;
    price_max: number;
    area_min: number;
    keywords: string[];
    exclude_keywords: string[];
  };
}

export class CustomFilter {
  private rules: FilterRule[];

  constructor(configPath: string) {
    const config = yaml.load(fs.readFileSync(configPath, 'utf-8')) as any;
    this.rules = config.filters.filter((f: FilterRule) => f.enabled);
    logger.info(`Loaded ${this.rules.length} active filter rules`);
  }

  filter(properties: Property591[]): Property591[] {
    if (this.rules.length === 0) return properties;

    return properties.filter(prop => {
      // 物件必須符合至少一個篩選規則
      return this.rules.some(rule => this.matchRule(prop, rule));
    });
  }

  private matchRule(property: Property591, rule: FilterRule): boolean {
    const c = rule.conditions;

    // 房屋類型檢查
    if (c.house_type.length > 0 && !c.house_type.includes(property.houseType)) {
      return false;
    }

    // 價格範圍
    if (property.price < c.price_min || property.price > c.price_max) {
      return false;
    }

    // 最小坪數
    if (property.area < c.area_min) {
      return false;
    }

    // 必須包含的關鍵字
    if (c.keywords.length > 0) {
      const hasKeyword = c.keywords.some(kw =>
        property.title.includes(kw) || property.description.includes(kw)
      );
      if (!hasKeyword) return false;
    }

    // 排除關鍵字
    if (c.exclude_keywords.length > 0) {
      const hasExcluded = c.exclude_keywords.some(kw =>
        property.title.includes(kw) || property.description.includes(kw)
      );
      if (hasExcluded) return false;
    }

    return true;
  }

  // Telegram Bot 指令：查看目前篩選規則
  getFilterSummary(): string {
    return this.rules.map((rule, i) =>
      `${i + 1}. *${rule.name}*\n` +
      `   類型：${rule.conditions.house_type.join('、') || '全部'}\n` +
      `   價格：${rule.conditions.price_min}-${rule.conditions.price_max} 萬\n` +
      `   最小坪數：${rule.conditions.area_min} 坪`
    ).join('\n\n');
  }

  // 動態重新載入設定
  reload(configPath: string): void {
    const config = yaml.load(fs.readFileSync(configPath, 'utf-8')) as any;
    this.rules = config.filters.filter((f: FilterRule) => f.enabled);
    logger.info(`Reloaded ${this.rules.length} active filter rules`);
  }
}
```

---

## 11. 模組十一：LINE 官方帳號 — 龍蝦 AI 對話機器人

### 功能說明

買方/賣方加入 LINE 官方帳號後，與「龍蝦」AI 對話。龍蝦扮演親切的在地朋友角色，先回答房產相關問題建立信任，再自然地引導對方透露真實需求（急不急賣、心理底價、賣屋原因等），最後將情報彙整推播給業務。

### 運作流程

```
買方/賣方加入 LINE 官方帳號
        │
        ▼
龍蝦 AI 主動打招呼
「嗨！我是龍蝦 🦞 竹山在地的房產小幫手，有什麼可以幫你的？」
        │
        ▼
┌─────────────────────────────────────┐
│         Phase 1：客服建立信任         │
│                                     │
│  回答問題：行情、流程、稅費、注意事項   │
│  語氣：親切、幽默、像朋友聊天          │
│  目標：讓對方放下戒心                  │
└──────────────┬──────────────────────┘
               │ 自然轉換（2-3 輪後）
               ▼
┌─────────────────────────────────────┐
│         Phase 2：需求情蒐             │
│                                     │
│  引導話題：你的房子在哪？想賣多少？    │
│           為什麼想賣？急不急？         │
│  AI 即時分析對話，提取關鍵情報         │
└──────────────┬──────────────────────┘
               │ 情報達標 or 對方想找真人
               ▼
┌─────────────────────────────────────┐
│         Phase 3：轉接真人業務          │
│                                     │
│  龍蝦：「我幫你轉接我們最專業的顧問！」 │
│  → 推播情報摘要到業務 Telegram         │
│  → 寫入 CRM                          │
└─────────────────────────────────────┘
```

### LINE Messaging API 設定

**前置步驟：**
1. 前往 [LINE Developers Console](https://developers.line.biz/)
2. 建立 Provider → 建立 Messaging API Channel
3. 取得 Channel Access Token 和 Channel Secret
4. 設定 Webhook URL（需要 HTTPS 的公開網址）
5. 關閉「自動回應訊息」，改用 Webhook

### 龍蝦 AI 核心實作

```typescript
// src/line/lobster-bot.ts

import { Client, WebhookEvent, TextMessage, MessageAPIResponseBase } from '@line/bot-sdk';
import { ConversationManager } from './conversation-manager';
import { IntentAnalyzer } from './intent-analyzer';
import { buildLobsterPrompt } from './lobster-prompts';
import { HouseBot } from '../telegram/bot';
import { GoogleSheetsCRM } from '../crm/google-sheets';
import { logger } from '../utils/logger';
import Anthropic from '@anthropic-ai/sdk';

export class LobsterBot {
  private lineClient: Client;
  private claude: Anthropic;
  private model: string;
  private convManager: ConversationManager;
  private intentAnalyzer: IntentAnalyzer;
  private telegramBot: HouseBot;
  private crm: GoogleSheetsCRM;
  private config: LineConfig;

  constructor(
    lineConfig: LineConfig,
    claude: Anthropic,
    model: string,
    telegramBot: HouseBot,
    crm: GoogleSheetsCRM,
  ) {
    this.lineClient = new Client({
      channelAccessToken: lineConfig.channel_access_token,
      channelSecret: lineConfig.channel_secret,
    });
    this.claude = claude;
    this.model = model;
    this.convManager = new ConversationManager();
    this.intentAnalyzer = new IntentAnalyzer(claude, model);
    this.telegramBot = telegramBot;
    this.crm = crm;
    this.config = lineConfig;
  }

  // LINE Webhook 處理
  async handleWebhook(events: WebhookEvent[]): Promise<void> {
    for (const event of events) {
      if (event.type === 'message' && event.message.type === 'text') {
        await this.handleTextMessage(event);
      } else if (event.type === 'follow') {
        // 新用戶加入
        await this.handleFollow(event);
      }
    }
  }

  // 新用戶加入 → 龍蝦打招呼
  private async handleFollow(event: WebhookEvent): Promise<void> {
    const userId = event.source.userId!;
    const greeting = `嗨！我是龍蝦 🦞\n\n` +
      `竹山在地的房產小幫手～\n` +
      `不管你是想買房、賣房、還是想了解行情，都可以問我！\n\n` +
      `有什麼可以幫你的嗎？😊`;

    await this.lineClient.replyMessage(event.replyToken, {
      type: 'text',
      text: greeting,
    });

    // 初始化對話
    this.convManager.initConversation(userId);
    logger.info(`New LINE user followed: ${userId}`);
  }

  // 處理文字訊息
  private async handleTextMessage(event: any): Promise<void> {
    const userId = event.source.userId;
    const userMessage = event.message.text;

    // 取得或建立對話歷史
    const conversation = this.convManager.getConversation(userId);
    conversation.addUserMessage(userMessage);

    // 判斷目前對話階段
    const phase = this.convManager.getPhase(userId);

    // 建構 Claude prompt（含對話歷史 + 龍蝦人設）
    const systemPrompt = buildLobsterPrompt(this.config.persona, phase);
    const messages = conversation.toClaude(); // 轉換為 Claude messages 格式

    try {
      // 呼叫 Claude 產生回覆
      const response = await this.claude.messages.create({
        model: this.model,
        max_tokens: 500,
        system: systemPrompt,
        messages: messages,
      });

      const replyText = response.content[0].type === 'text'
        ? response.content[0].text : '抱歉，我剛剛恍神了 😅 再說一次？';

      // 回覆 LINE 用戶
      await this.lineClient.replyMessage(event.replyToken, {
        type: 'text',
        text: replyText,
      });

      // 記錄助理回覆
      conversation.addAssistantMessage(replyText);

      // 即時分析對話，提取情報
      const intel = await this.intentAnalyzer.analyze(conversation);

      if (intel.hasNewIntel) {
        // 更新 CRM
        await this.crm.updateLineIntel(userId, intel);

        // 如果情報達標（知道地址+價格+急迫度），通知業務
        if (intel.readyForHandoff) {
          await this.notifyAgent(userId, intel, conversation);
        }
      }

      // 檢查是否該轉接真人
      if (conversation.roundCount >= this.config.handoff_after_rounds || intel.requestedHuman) {
        await this.handoffToAgent(userId, event.replyToken, conversation, intel);
      }

    } catch (error) {
      logger.error(`Lobster reply failed for ${userId}: ${error}`);
      await this.lineClient.replyMessage(event.replyToken, {
        type: 'text',
        text: '不好意思，我剛剛卡住了 😅\n等我一下，馬上回你！',
      });
    }
  }

  // 通知業務有高價值線索
  private async notifyAgent(
    userId: string,
    intel: IntelResult,
    conversation: Conversation,
  ): Promise<void> {
    const msg =
      `🦞 *龍蝦情報通知*\n\n` +
      `有一位 LINE 用戶透露了重要資訊：\n\n` +
      `📌 *類型：* ${intel.userType === 'seller' ? '賣方' : '買方'}\n` +
      `🏠 *物件位置：* ${intel.location || '未透露'}\n` +
      `💰 *期望價格：* ${intel.expectedPrice || '未透露'}\n` +
      `⏰ *急迫度：* ${intel.urgency || '未透露'}\n` +
      `📝 *賣屋原因：* ${intel.reason || '未透露'}\n` +
      `💬 *對話摘要：* ${intel.summary}\n\n` +
      `👉 建議：*${intel.recommendation}*\n\n` +
      `_對話已進行 ${conversation.roundCount} 輪_`;

    if (this.config.notify_telegram) {
      await this.telegramBot.sendMessage(msg);
    }
  }

  // 轉接真人業務
  private async handoffToAgent(
    userId: string,
    replyToken: string,
    conversation: Conversation,
    intel: IntelResult,
  ): Promise<void> {
    await this.lineClient.replyMessage(replyToken, {
      type: 'text',
      text: `聊了這麼多，我覺得你可以直接跟我們最專業的顧問聊聊！\n\n` +
        `他是竹山在地深耕多年的 ${this.config.persona.company} 顧問，\n` +
        `人很好很專業，不會給你壓力的 😊\n\n` +
        `我幫你轉接，他很快會聯繫你～`,
    });

    // 推播完整情報給業務
    await this.notifyAgent(userId, intel, conversation);
  }
}
```

### 對話狀態管理

```typescript
// src/line/conversation-manager.ts

export interface ConversationMessage {
  role: 'user' | 'assistant';
  content: string;
  timestamp: Date;
}

export class Conversation {
  userId: string;
  messages: ConversationMessage[] = [];
  phase: 'greeting' | 'service' | 'intel' | 'handoff' = 'greeting';
  roundCount: number = 0;
  intel: Partial<IntelResult> = {};
  createdAt: Date = new Date();

  constructor(userId: string) {
    this.userId = userId;
  }

  addUserMessage(content: string): void {
    this.messages.push({ role: 'user', content, timestamp: new Date() });
    this.roundCount++;
  }

  addAssistantMessage(content: string): void {
    this.messages.push({ role: 'assistant', content, timestamp: new Date() });
  }

  // 轉換為 Claude API messages 格式
  toClaude(): Array<{ role: 'user' | 'assistant'; content: string }> {
    // 只保留最近 20 輪對話（避免 token 過多）
    const recent = this.messages.slice(-40);
    return recent.map(m => ({ role: m.role, content: m.content }));
  }
}

export class ConversationManager {
  private conversations: Map<string, Conversation> = new Map();

  initConversation(userId: string): Conversation {
    const conv = new Conversation(userId);
    this.conversations.set(userId, conv);
    return conv;
  }

  getConversation(userId: string): Conversation {
    if (!this.conversations.has(userId)) {
      return this.initConversation(userId);
    }
    return this.conversations.get(userId)!;
  }

  getPhase(userId: string): string {
    const conv = this.getConversation(userId);

    // 根據對話輪數自動判斷階段
    if (conv.roundCount <= 2) return 'greeting';
    if (conv.roundCount <= 5) return 'service';
    return 'intel';
  }
}
```

### 龍蝦人設 Prompt

```typescript
// src/line/lobster-prompts.ts

export function buildLobsterPrompt(persona: PersonaConfig, phase: string): string {
  const basePrompt = `你是「${persona.name}」，${persona.company}的 AI 房產小幫手。

## 你的人設
- 名字：龍蝦 🦞
- 個性：${persona.personality}
- 說話方式：口語化、會用表情符號、偶爾開玩笑
- 專業知識：台灣房地產買賣流程、稅費、行情、注意事項
- 禁止：不能報具體價格（只能說行情範圍）、不能承諾成交、不能批評其他房仲

## 重要規則
1. 永遠不要說「我是 AI」或「我是機器人」，你就是龍蝦
2. 回覆要簡短（2-4 句話），像 LINE 聊天不要寫長文
3. 適當使用表情符號但不要太多（每則 1-2 個）
4. 如果對方問你不知道的事，誠實說「這個我要幫你確認一下」
5. 永遠不要主動提到競爭對手`;

  const phasePrompts: Record<string, string> = {
    greeting: `
## 目前階段：打招呼
- 親切歡迎，了解對方是要買房還是賣房
- 不要急著問太多問題
- 輕鬆聊天就好`,

    service: `
## 目前階段：客服建立信任
- 認真回答對方的問題
- 展現專業知識
- 適時分享在地資訊（竹山哪裡生活機能好等）
- 開始自然地了解對方的需求
- 可以問：「你是在看哪一帶的房子呀？」`,

    intel: `
## 目前階段：需求情蒐
- 你已經建立了一定信任，開始深入了解
- 自然地引導對方說出：
  1. 物件位置/地址
  2. 期望價格（買方：預算 / 賣方：希望賣多少）
  3. 急迫度（多久想成交）
  4. 原因（為什麼要買/賣）
- 用輕鬆的方式問，例如：
  「大概心裡有個底價嗎？我幫你參考看看行情～」
  「是想趕快處理還是慢慢來都可以？」
- 不要像審問，一次只問一個問題
- 如果對方不想說，不要勉強，換個話題再回來`,
  };

  return basePrompt + (phasePrompts[phase] || phasePrompts.service);
}
```

### 意圖分析與情蒐引擎

```typescript
// src/line/intent-analyzer.ts

import Anthropic from '@anthropic-ai/sdk';
import { Conversation } from './conversation-manager';

export interface IntelResult {
  hasNewIntel: boolean;
  readyForHandoff: boolean;
  requestedHuman: boolean;
  userType: 'seller' | 'buyer' | 'unknown';
  location: string | null;
  expectedPrice: string | null;
  urgency: string | null;
  reason: string | null;
  summary: string;
  recommendation: string;
}

export class IntentAnalyzer {
  private claude: Anthropic;
  private model: string;

  constructor(claude: Anthropic, model: string) {
    this.claude = claude;
    this.model = model;
  }

  async analyze(conversation: Conversation): Promise<IntelResult> {
    // 每 3 輪分析一次（節省 API 費用）
    if (conversation.roundCount % 3 !== 0 && conversation.roundCount > 3) {
      return { hasNewIntel: false } as IntelResult;
    }

    const prompt = `分析以下 LINE 對話，提取房產情報。

## 對話記錄
${conversation.messages.map(m =>
  `${m.role === 'user' ? '用戶' : '龍蝦'}：${m.content}`
).join('\n')}

## 請回傳 JSON：
{
  "hasNewIntel": <boolean, 本次分析是否有新情報>,
  "readyForHandoff": <boolean, 情報是否足以轉接業務（至少知道位置+價格或急迫度）>,
  "requestedHuman": <boolean, 用戶是否要求跟真人說話>,
  "userType": "<seller|buyer|unknown>",
  "location": "<物件位置，null 如果未提及>",
  "expectedPrice": "<期望價格，null 如果未提及>",
  "urgency": "<急迫度描述，null 如果未提及>",
  "reason": "<買賣原因，null 如果未提及>",
  "summary": "<一句話總結目前掌握的情報>",
  "recommendation": "<給業務的建議行動>"
}
只回傳 JSON。`;

    const response = await this.claude.messages.create({
      model: this.model,
      max_tokens: 500,
      messages: [{ role: 'user', content: prompt }],
    });

    const text = response.content[0].type === 'text' ? response.content[0].text : '{}';
    return JSON.parse(text.match(/\{[\s\S]*\}/)![0]);
  }
}
```

### LINE Webhook Express Server

```typescript
// src/line/webhook-server.ts

import express from 'express';
import { middleware, MiddlewareConfig } from '@line/bot-sdk';
import { LobsterBot } from './lobster-bot';

export function createLineWebhookServer(
  lobsterBot: LobsterBot,
  config: { channelSecret: string; port: number },
): express.Express {
  const app = express();

  const middlewareConfig: MiddlewareConfig = {
    channelSecret: config.channelSecret,
  };

  app.post('/line/webhook', middleware(middlewareConfig), async (req, res) => {
    try {
      await lobsterBot.handleWebhook(req.body.events);
      res.status(200).send('OK');
    } catch (error) {
      console.error('Webhook error:', error);
      res.status(500).send('Error');
    }
  });

  app.get('/health', (req, res) => res.send('OK'));

  app.listen(config.port, () => {
    console.log(`LINE webhook server running on port ${config.port}`);
  });

  return app;
}
```

### LINE 設定步驟

1. 前往 [LINE Developers Console](https://developers.line.biz/)
2. 建立 Provider → Messaging API Channel
3. 取得 **Channel Access Token**（長效）和 **Channel Secret**
4. Webhook URL 設為 `https://your-domain.com/line/webhook`
5. 關閉「自動回應訊息」和「加入好友的歡迎訊息」（由 Bot 自行處理）
6. 開啟 Webhook

### API 費用預估

| 項目 | 費用 |
|------|------|
| LINE 官方帳號（輕用量） | 免費（每月 500 則免費推播） |
| LINE 官方帳號（中用量） | NT$800/月 |
| Claude API（對話） | NT$500-1,500/月（看對話量） |
| Claude API（情蒐分析） | NT$200-500/月 |

---

## 12. 模組十二：Facebook AI 發文

### 功能說明

AI 自動產生高品質貼文，協助經營龍蝦粉絲頁與社團。

**重要限制：** Facebook 自 2018 年起封鎖第三方 App 在社團發文（Graph API 不支援）。因此：
- **粉絲頁：** 使用 Graph API 全自動發佈 ✅
- **社團：** AI 產文 → 推播草稿到 Telegram → 業務手動複製貼到社團（10 秒完成）✅

> 不使用 Selenium/Browser 自動化操作社團，避免帳號被 FB 偵測限制。

### 發文類型

| 類型 | 說明 | 範例 |
|------|------|------|
| 🏠 物件推薦 | 推薦近期高分物件 | 「竹山延平路透天，3房2廳，開價 800 萬...」 |
| 📊 市場觀察 | 區域行情趨勢分析 | 「本週竹山新物件較上週增加 20%...」 |
| 💡 買賣小知識 | 房產知識科普 | 「你知道買房要付哪些稅嗎？」 |
| 🤔 互動型貼文 | 引發留言互動 | 「如果有 800 萬預算，你會選竹山透天還是南投市大樓？」 |

### Facebook Graph API 設定（粉絲頁自動發佈）

**前置步驟：**
1. 前往 [Facebook Developers](https://developers.facebook.com/)
2. 建立應用程式 → 取得 App ID 和 App Secret
3. 取得 Page Access Token（長效）
4. 申請 `pages_manage_posts` 權限
5. 取得 Page ID

> ⚠️ 注意：`publish_to_groups` 權限已於 2018 年被 Facebook 封鎖，無法透過 API 在社團發文。社團發文改為 AI 產文 + 人工發佈。

### AI 發文引擎

```typescript
// src/facebook/post-generator.ts

import Anthropic from '@anthropic-ai/sdk';
import { db } from '../storage/database';
import { logger } from '../utils/logger';

export type PostType = 'property_listing' | 'market_insight' | 'tips' | 'engagement';

export interface GeneratedPost {
  type: PostType;
  content: string;
  hashtags: string[];
}

export class PostGenerator {
  private claude: Anthropic;
  private model: string;

  constructor(claude: Anthropic, model: string) {
    this.claude = claude;
    this.model = model;
  }

  // 根據權重隨機選擇發文類型
  selectPostType(weights: Record<PostType, number>): PostType {
    const total = Object.values(weights).reduce((a, b) => a + b, 0);
    let random = Math.random() * total;

    for (const [type, weight] of Object.entries(weights)) {
      random -= weight;
      if (random <= 0) return type as PostType;
    }
    return 'property_listing';
  }

  async generatePost(type: PostType): Promise<GeneratedPost> {
    switch (type) {
      case 'property_listing':
        return this.generatePropertyPost();
      case 'market_insight':
        return this.generateMarketPost();
      case 'tips':
        return this.generateTipsPost();
      case 'engagement':
        return this.generateEngagementPost();
    }
  }

  // 物件推薦貼文
  private async generatePropertyPost(): Promise<GeneratedPost> {
    // 取得本週高分物件
    const topProperties = await db.getTopScoredProperties({
      minScore: 70,
      limit: 3,
      since: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000),
    });

    if (topProperties.length === 0) {
      // 沒有高分物件，改發市場觀察
      return this.generateMarketPost();
    }

    const property = topProperties[0]; // 取最高分的

    const prompt = `你是龍蝦 🦞，在Facebook房產社團發文推薦物件。

## 物件資訊
- 標題：${property.title}
- 地址：${property.address}
- 開價：${property.price} 萬
- 坪數：${property.area} 坪
- 類型：${property.houseType}
- 格局：${property.rooms}

## 發文要求
1. 用輕鬆親切的口吻，像朋友推薦
2. 不要太業務感，要有溫度
3. 開頭要吸引人（用表情符號）
4. 300-500 字
5. 結尾引導互動（「有興趣私訊我」或「留言+1」）
6. 不要提到具體屋主資訊
7. 建議 3-5 個 hashtag

回傳 JSON：
{
  "content": "<貼文內容>",
  "hashtags": ["#tag1", "#tag2"]
}`;

    const response = await this.claude.messages.create({
      model: this.model,
      max_tokens: 1000,
      messages: [{ role: 'user', content: prompt }],
    });

    const text = response.content[0].type === 'text' ? response.content[0].text : '';
    const result = JSON.parse(text.match(/\{[\s\S]*\}/)![0]);

    return {
      type: 'property_listing',
      content: result.content + '\n\n' + result.hashtags.join(' '),
      hashtags: result.hashtags,
    };
  }

  // 市場觀察貼文
  private async generateMarketPost(): Promise<GeneratedPost> {
    const stats = await db.getWeeklyStats(
      new Date(Date.now() - 7 * 24 * 60 * 60 * 1000),
      new Date(),
    );

    const prompt = `你是龍蝦 🦞，在Facebook社團發一篇本週市場觀察。

## 本週數據
- 新物件數：${stats.totalNew}
- 平均開價：${stats.avgPrice} 萬
- 上週新物件：${stats.lastWeekTotal}

## 要求
1. 300-400 字
2. 輕鬆但專業
3. 分享實用觀點
4. 結尾問大家的看法

回傳 JSON：{"content": "...", "hashtags": [...]}`;

    const response = await this.claude.messages.create({
      model: this.model,
      max_tokens: 800,
      messages: [{ role: 'user', content: prompt }],
    });

    const text = response.content[0].type === 'text' ? response.content[0].text : '';
    const result = JSON.parse(text.match(/\{[\s\S]*\}/)![0]);

    return { type: 'market_insight', content: result.content + '\n\n' + result.hashtags.join(' '), hashtags: result.hashtags };
  }

  // 買賣小知識
  private async generateTipsPost(): Promise<GeneratedPost> {
    const topics = [
      '買房要付哪些稅和費用',
      '如何判斷合理房價',
      '看房必看的 10 個重點',
      '屋主自售 vs 找房仲的差別',
      '簽約前一定要注意的事',
      '房貸利率怎麼談比較好',
      '什麼是實價登錄，怎麼查',
      '預售屋 vs 中古屋怎麼選',
      '驗屋要驗什麼',
      '買房殺價的技巧',
    ];
    const topic = topics[Math.floor(Math.random() * topics.length)];

    const prompt = `你是龍蝦 🦞，在Facebook社團分享買賣房知識。

主題：${topic}

要求：
1. 300-500 字
2. 用條列式，方便閱讀
3. 語氣親切，不要太教條
4. 結尾互動：「有其他問題也可以問龍蝦喔！」

回傳 JSON：{"content": "...", "hashtags": [...]}`;

    const response = await this.claude.messages.create({
      model: this.model,
      max_tokens: 800,
      messages: [{ role: 'user', content: prompt }],
    });

    const text = response.content[0].type === 'text' ? response.content[0].text : '';
    const result = JSON.parse(text.match(/\{[\s\S]*\}/)![0]);

    return { type: 'tips', content: result.content + '\n\n' + result.hashtags.join(' '), hashtags: result.hashtags };
  }

  // 互動型貼文
  private async generateEngagementPost(): Promise<GeneratedPost> {
    const prompt = `你是龍蝦 🦞，在Facebook房產社團發一篇引發討論的互動貼文。

要求：
1. 150-250 字（短一點比較多人互動）
2. 提出一個有趣的選擇題或問題
3. 例如：「透天 vs 大樓你選哪個？」「你覺得竹山房價會漲還跌？」
4. 用投票或留言的方式引導互動
5. 輕鬆有趣，不要太嚴肅

回傳 JSON：{"content": "...", "hashtags": [...]}`;

    const response = await this.claude.messages.create({
      model: this.model,
      max_tokens: 500,
      messages: [{ role: 'user', content: prompt }],
    });

    const text = response.content[0].type === 'text' ? response.content[0].text : '';
    const result = JSON.parse(text.match(/\{[\s\S]*\}/)![0]);

    return { type: 'engagement', content: result.content + '\n\n' + result.hashtags.join(' '), hashtags: result.hashtags };
  }
}
```

### Facebook 發文排程

```typescript
// src/facebook/fb-poster.ts

import cron from 'node-cron';
import { PostGenerator, PostType } from './post-generator';
import { logger } from '../utils/logger';

export class FacebookPoster {
  private pageAccessToken: string;
  private pageId: string;
  private telegramBot: HouseBot;
  private postGenerator: PostGenerator;
  private config: FacebookConfig;
  private todayPostCount: number = 0;

  constructor(
    pageAccessToken: string,
    pageId: string,
    telegramBot: HouseBot,
    postGenerator: PostGenerator,
    config: FacebookConfig,
  ) {
    this.pageAccessToken = pageAccessToken;
    this.pageId = pageId;
    this.telegramBot = telegramBot;
    this.postGenerator = postGenerator;
    this.config = config;
  }

  // 排程發文
  schedulePosting(): void {
    for (const time of this.config.post_schedule.times) {
      const [hour, minute] = time.split(':');

      cron.schedule(`${minute} ${hour} * * *`, async () => {
        if (this.todayPostCount >= this.config.post_schedule.max_posts_per_day) {
          logger.info('Max daily posts reached, skipping');
          return;
        }

        try {
          const postType = this.postGenerator.selectPostType(this.config.post_types);
          logger.info(`Generating ${postType} post...`);

          const post = await this.postGenerator.generatePost(postType);

          // 粉絲頁：Graph API 自動發佈
          await this.publishToPage(post.content);

          // 社團：推播草稿到 Telegram，業務手動複製貼到社團
          await this.sendGroupDraftToTelegram(post.content);

          this.todayPostCount++;
          logger.info(`FB post published to page + group draft sent`);

        } catch (error) {
          logger.error(`FB posting failed: ${error}`);
        }
      }, { timezone: 'Asia/Taipei' });
    }

    cron.schedule('0 0 * * *', () => {
      this.todayPostCount = 0;
    }, { timezone: 'Asia/Taipei' });

    logger.info(`FB posting scheduled at ${this.config.post_schedule.times.join(', ')}`);
  }

  // ✅ 粉絲頁：Graph API 自動發佈（合規）
  private async publishToPage(message: string): Promise<void> {
    const url = `https://graph.facebook.com/v18.0/${this.pageId}/feed`;

    const response = await fetch(url, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        message: message,
        access_token: this.pageAccessToken,
      }),
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(`FB Page API Error: ${JSON.stringify(error)}`);
    }

    const result = await response.json();
    logger.info(`FB page post created: ${result.id}`);
  }

  // 📋 社團：AI 產文草稿 → 推播到 Telegram，業務手動複製貼到社團
  // （Facebook 2018 年起封鎖第三方 App 在社團發文，因此不使用 API）
  private async sendGroupDraftToTelegram(content: string): Promise<void> {
    const msg =
      `📋 *龍蝦社團貼文草稿*\n\n` +
      `以下內容已自動發佈到粉絲頁 ✅\n` +
      `社團請手動複製貼上：\n\n` +
      `━━━━━━━━━━━━━━\n` +
      `${content}\n` +
      `━━━━━━━━━━━━━━\n\n` +
      `👆 長按複製 → 貼到龍蝦社團`;

    await this.telegramBot.sendMessage(msg);
  }

  // 手動觸發發文（Telegram 指令）
  async manualPost(type?: PostType): Promise<string> {
    const postType = type || this.postGenerator.selectPostType(this.config.post_types);
    const post = await this.postGenerator.generatePost(postType);
    await this.publishToPage(post.content);
    await this.sendGroupDraftToTelegram(post.content);
    return post.content;
  }
}
```

### Facebook API 設定步驟（粉絲頁）

1. 前往 [Facebook Developers](https://developers.facebook.com/)
2. 建立應用程式 → 選擇「商業」類型
3. 新增產品 → 選擇「Facebook 登入」
4. 取得 **Page Access Token**（永久的）：
   - 使用 [Graph API Explorer](https://developers.facebook.com/tools/explorer/)
   - 選擇你的應用程式和粉絲頁
   - 勾選 `pages_manage_posts` 權限
   - 產生長效 Token
5. 取得 Page ID：
   - 粉絲頁「關於」頁面最下方可看到
   - 或使用 Graph API: `GET /me/accounts`

### 重要限制與注意事項

| 項目 | 說明 |
|------|------|
| ❌ 社團 API 發文 | Facebook 2018 年起封鎖，`publish_to_groups` 已停用 |
| ✅ 粉絲頁 API 發文 | 使用 `pages_manage_posts` 權限，官方支援 |
| 社團發文方式 | AI 產文 → Telegram 草稿 → 業務手動貼到社團（10 秒） |
| 粉絲頁發文頻率 | 建議每天不超過 2-3 篇 |
| Token 過期 | Page Access Token 需定期更新（建議用長效 Token） |

### 運作模式總結

```
AI 產生貼文內容
       │
       ├──→ 粉絲頁：Graph API 自動發佈 ✅（業務可開啟草稿審核模式）
       │
       └──→ 社團：推播草稿到 Telegram → 業務手動複製貼上 📋
```

粉絲頁可選配「草稿審核模式」：

```typescript
// 粉絲頁草稿審核模式（選配，預設關閉）
const PAGE_DRAFT_MODE = false;

if (PAGE_DRAFT_MODE) {
  // 先推播預覽到 Telegram，等業務 /approve 後才發佈到粉絲頁
  await telegramBot.sendMessage(
    `📝 *粉絲頁發文草稿*\n\n` +
    `${post.content}\n\n` +
    `━━━━━━━━━━━━━━\n` +
    `回覆 /approve 發佈到粉絲頁\n` +
    `回覆 /reject 取消`
  );
} else {
  // 直接發佈到粉絲頁（預設）
  await publishToPage(post.content);
}

// 社團草稿一律推播到 Telegram（無法 API 發佈）
await sendGroupDraftToTelegram(post.content);
```

### API 費用預估

| 項目 | 費用 |
|------|------|
| Facebook Graph API | 免費 |
| Claude API（產文） | NT$100-300/月（每天 2 篇） |

---

## 13. OpenClaw Agent 整合（專業版）

### 完整 Agent 定義

```typescript
// src/agent/house-agent.ts（專業版）

import { Agent } from 'openclaw';
import { Crawler591 } from '../crawler/crawler-591';
import { ScoringEngine } from '../ai/scoring-engine';
import { SpeechGenerator } from '../ai/speech-generator';
import { HouseBot } from '../telegram/bot';
import { MultiPlatformAnalyzer } from '../telegram/multi-platform';
import { GoogleSheetsCRM } from '../crm/google-sheets';
import { AreaManager } from '../multi-area/area-manager';
import { CustomFilter } from '../filter/custom-filter';
import { WeeklyReport } from '../report/weekly-report';
import { loadConfig } from '../config';

export function createHouseAgentPro(): Agent {
  const config = loadConfig();

  // 初始化所有模組
  const crawler = new Crawler591(config.crawler);
  const scorer = new ScoringEngine(config.ai.api_key, config.ai.model);
  const speechGen = new SpeechGenerator(scorer.client, config.ai.model, config.speech, loadSamples());
  const bot = new HouseBot(config.telegram.bot_token, config.telegram.chat_id);
  const crm = new GoogleSheetsCRM(/*...*/);
  const areaManager = new AreaManager(config.crawler.areas, crawler, scorer);
  const filter = new CustomFilter(config.filters.config_path);
  const weeklyReport = new WeeklyReport(bot, scorer);

  // 初始化多平台分析（Telegram polling）
  const multiPlatform = new MultiPlatformAnalyzer(
    bot.rawBot, scorer, speechGen, crm, config.telegram.chat_id
  );

  const agent = new Agent({
    name: 'house-agent-pro',
    description: 'AI 房仲智能助理（專業版）',

    tools: [
      // 標準版工具
      {
        name: 'crawl_all_areas',
        description: '爬取所有監控區域的 591 物件',
        execute: async () => {
          const results = await areaManager.crawlAllAreas();
          return results;
        },
      },
      {
        name: 'filter_properties',
        description: '用客製化條件篩選物件',
        execute: async (input: { properties: Property591[] }) => {
          return filter.filter(input.properties);
        },
      },
      {
        name: 'score_and_save',
        description: '評分並儲存到 CRM',
        execute: async (input: { properties: Property591[] }) => {
          for (const prop of input.properties) {
            const score = await scorer.scoreProperty(prop);
            await crm.appendProperty(prop, score);

            // 專業版：同時產生話術
            if (score.score >= config.ai.score_threshold) {
              const speech = await speechGen.generateSpeech(prop, score);
              // 儲存話術到資料庫，供推播時使用
              await db.saveSpeech(prop.id, speech);
            }
          }
        },
      },
      {
        name: 'generate_speech',
        description: '產生話術建議',
        execute: async (input: { property: Property591; score: ScoringResult }) => {
          return speechGen.generateSpeech(input.property, input.score);
        },
      },
    ],

    workflow: async (ctx) => {
      // Step 1: 多區域爬取
      const crawlResults = await ctx.useTool('crawl_all_areas');

      // Step 2: 合併所有新物件
      const allNew = crawlResults.flatMap((r: any) => r.properties);

      if (allNew.length === 0) return 'No new properties';

      // Step 3: 客製化篩選
      const filtered = await ctx.useTool('filter_properties', { properties: allNew });

      // Step 4: AI 評分 + 話術 + 儲存 CRM
      await ctx.useTool('score_and_save', { properties: filtered });

      return `Processed ${filtered.length}/${allNew.length} properties`;
    },
  });

  return agent;
}
```

### 主入口（專業版）

```typescript
// src/index.ts（專業版）

import { createHouseAgentPro } from './agent/house-agent';
import { scheduleDailyPush } from './telegram/daily-push';
import { scheduleTrackerReminder } from './telegram/tracker-reminder';
import { MultiPlatformAnalyzer } from './telegram/multi-platform';
import { WeeklyReport } from './report/weekly-report';
import { loadConfig } from './config';

async function main() {
  const config = loadConfig();
  await db.init();

  const agent = createHouseAgentPro();
  const bot = new HouseBot(config.telegram.bot_token, config.telegram.chat_id);

  // 排程：定時爬取（多區域）
  const intervalMs = config.crawler.interval_minutes * 60 * 1000;
  setInterval(() => agent.run(), intervalMs);

  // 排程：每日推播（含話術）
  scheduleDailyPush(bot, config.telegram.daily_push_time);

  // 排程：追蹤提醒
  scheduleTrackerReminder(bot);

  // 排程：每週報告
  const weeklyReport = new WeeklyReport(bot, scorer);
  weeklyReport.scheduleWeeklyReport(
    config.weekly_report.day_of_week,
    config.weekly_report.time,
  );

  // 啟動多平台 AI 分析（Telegram polling 模式）
  if (config.telegram.multi_platform_enabled) {
    const multiPlatform = new MultiPlatformAnalyzer(
      bot.rawBot, scorer, speechGen, crm, config.telegram.chat_id
    );
    await multiPlatform.init();
  }

  // 首次執行
  await agent.run();

  logger.info('House Agent PRO started');
  await bot.sendMessage(
    '🤖 AI 房仲助理（專業版）已啟動！\n\n' +
    '功能：\n' +
    '• 591 多區域監控中\n' +
    '• 轉貼連結/截圖即可 AI 分析\n' +
    '• 每日推播 + 話術建議\n' +
    '• 每週彙整報告\n\n' +
    '指令：/help 查看所有指令'
  );
}

main().catch(console.error);
```

---

## 14. 部署指南

與標準版相同，請參考 [standard-guide.md #部署指南](./standard-guide.md#11-部署指南)。

### 專業版額外步驟

```bash
# 額外安裝 cheerio（HTML 解析）和 sharp（圖片處理）
npm install cheerio sharp

# 複製篩選條件設定
cp config/filters.example.yaml config/filters.yaml

# 複製話術範例庫
cp config/speech-samples.example.yaml config/speech-samples.yaml

# 編輯話術範例（由業務提供成功案例）
nano config/speech-samples.yaml

# Telegram Bot 需改為 polling 模式（接收轉貼訊息）
# 在 config.yaml 中設定 multi_platform_enabled: true

# 安裝 LINE Bot SDK
npm install @line/bot-sdk

# 設定 LINE 環境變數
export LINE_CHANNEL_ACCESS_TOKEN="..."
export LINE_CHANNEL_SECRET="..."

# 設定 Facebook 環境變數（粉絲頁）
export FB_PAGE_ACCESS_TOKEN="..."
export FB_PAGE_ID="..."
# 注意：社團發文無需 API 設定，AI 產文草稿透過 Telegram 推播，業務手動貼到社團
```

### Telegram Bot 指令列表（專業版）

| 指令 | 說明 |
|------|------|
| `/help` | 查看所有指令 |
| `/status` | 系統狀態 |
| `/areas` | 列出監控區域 |
| `/filters` | 列出篩選規則 |
| `/report` | 手動觸發週報 |
| `/score <text>` | 手動輸入文字請 AI 分析 |
| `/fb_post` | 手動觸發 FB 社團發文 |
| `/fb_preview` | 預覽下一篇 FB 貼文 |
| `/approve` | 核准 FB 草稿發佈 |
| `/reject` | 取消 FB 草稿 |
| `/line_stats` | 查看 LINE 龍蝦對話統計 |
| 貼連結 | 自動 AI 分析 |
| 傳截圖 | 自動 AI Vision 分析 |

---

## 15. 驗收標準

### 標準版驗收（同 standard-guide.md）

| # | 驗收項目 | 通過條件 |
|---|---------|---------|
| 1-8 | 同標準版 | 見 standard-guide.md |

### 專業版額外驗收

| # | 驗收項目 | 通過條件 |
|---|---------|---------|
| 9 | 多區域監控 | 設定 3 個區域，全部能正常爬取，不互相干擾 |
| 10 | PTT 連結分析 | 貼 PTT 房版連結，10 秒內回覆評分 |
| 11 | 樂屋網連結分析 | 貼樂屋網連結，10 秒內回覆評分 |
| 12 | 文字轉貼分析 | 複製 FB 貼文文字，5 秒內回覆評分 |
| 13 | 截圖分析 | 傳截圖，10 秒內 OCR + 評分 |
| 14 | AI 話術產生 | 高分物件推播時附帶 3 種話術建議 |
| 15 | 話術風格 | 話術與業務提供的範例風格相近（業務主觀確認） |
| 16 | 客製化篩選 | 設定篩選條件後，只收到符合條件的推播 |
| 17 | 每週報告 | 週一收到完整週報，含數據 + AI 趨勢分析 |
| 18 | Bot 指令 | 所有 Telegram Bot 指令正常運作 |
| 19 | LINE 龍蝦打招呼 | 新用戶加入 LINE 後，龍蝦自動打招呼，語氣自然 |
| 20 | LINE 龍蝦客服 | 問房產問題（稅費、流程），能正確回答且語氣親切 |
| 21 | LINE 龍蝦情蒐 | 對話 5-8 輪後，能提取出用戶的位置/價格/急迫度 |
| 22 | LINE 情報推播 | 情蒐達標時，業務 Telegram 收到情報摘要 |
| 23 | LINE 轉接真人 | 對話達上限或用戶要求時，龍蝦引導轉接 |
| 24 | FB 社團 AI 發文 | AI 產生的貼文語氣自然、內容正確、不違反社團規範 |
| 25 | FB 發文排程 | 每天指定時間自動發文，不超過設定上限 |
| 26 | FB 草稿審核 | 業務能在 Telegram 預覽並核准/拒絕 FB 草稿 |

---

## 16. 維護與故障排除

### 標準版問題

同 [standard-guide.md #維護與故障排除](./standard-guide.md#13-維護與故障排除)

### 專業版額外問題

| 問題 | 原因 | 解法 |
|------|------|------|
| 多平台分析沒回應 | Bot polling 斷線 | 重啟 PM2，檢查 Bot Token |
| 截圖分析失敗 | 圖片太小或模糊 | 提示用戶傳高解析度截圖 |
| 話術風格不對 | 範例不夠 | 更新 speech-samples.yaml，增加更多範例 |
| 週報沒收到 | cron 排程問題 | 檢查 timezone 設定，手動觸發 `/report` 測試 |
| 篩選太嚴格 | 條件設定過窄 | 調整 filters.yaml 的價格/坪數範圍 |
| 樂屋網解析失敗 | 頁面改版 | 更新 parseRakuyaUrl 中的 selector |
| Claude Vision 費用偏高 | 截圖太多 | 建議用戶優先用文字轉貼 |
| LINE 龍蝦回覆不自然 | Prompt 需調整 | 修改 lobster-prompts.ts 的人設描述 |
| LINE Webhook 沒收到 | SSL 憑證或 URL 問題 | 確認 HTTPS 正確，到 LINE Console 測試 Webhook |
| LINE 龍蝦說太多話 | max_tokens 太高 | 降低 max_tokens 到 300 |
| LINE 情蒐太早問隱私 | Phase 切換太快 | 調整 ConversationManager 的輪數閾值 |
| FB 發文被刪 | 觸發反垃圾機制 | 降低發文頻率、調整內容避免太商業 |
| FB Token 過期 | 短效 Token | 改用長效 Token，設定自動更新機制 |
| FB 發文內容重複 | AI 產文缺乏多樣性 | 在 prompt 中加入「不要重複之前的主題」 |

### API 費用預估（專業版/月）

| 項目 | 標準版 | 專業版 |
|------|--------|--------|
| 591 評分 | NT$150-450 | NT$300-800（多區域） |
| 多平台分析 | - | NT$200-500（看使用頻率） |
| 話術產生 | - | NT$100-300 |
| 週報分析 | - | NT$20-50 |
| Vision (截圖) | - | NT$100-300（看使用頻率） |
| LINE 龍蝦 AI 對話 | - | NT$500-1,500（看對話量） |
| LINE 情蒐分析 | - | NT$200-500 |
| FB 社團 AI 產文 | - | NT$100-300 |
| LINE 官方帳號月費 | - | NT$0-800（視方案） |
| **月合計** | **NT$150-450** | **NT$1,520-4,750** |

> **⚠️ 以上所有 API 費用皆由客戶自行負擔，不包含在 NT$268,000 建置費中。**
> 伺服器使用客戶自有 Mac Mini，無額外伺服器費用。客戶每月營運成本約 NT$1,520-4,750。
> 可另外向客戶提供每月 NT$5,000-8,000 的技術維護方案（含系統監控、bug 修復、591 改版應對等）。

---

> **文件版本：** v1.0
> **建立日期：** 2026-03-20
> **適用方案：** 專業版（NT$268,000）
