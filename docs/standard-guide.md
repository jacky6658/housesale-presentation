# AI 房仲智能助理 — 標準版建置手冊

> **方案定價：** NT$168,000（一次性建置費）
> **技術支援：** 驗收後 14 天
> **預估工期：** 2 週（Week 1-2 交付）

---

## 目錄

1. [專案總覽](#1-專案總覽)
2. [技術棧與環境需求](#2-技術棧與環境需求)
3. [專案目錄結構](#3-專案目錄結構)
4. [設定檔規格](#4-設定檔規格)
5. [模組一：591 爬蟲監控引擎](#5-模組一591-爬蟲監控引擎)
6. [模組二：AI 評分引擎](#6-模組二ai-評分引擎)
7. [模組三：Telegram Bot 推播](#7-模組三telegram-bot-推播)
8. [模組四：Telegram 追蹤提醒](#8-模組四telegram-追蹤提醒)
9. [模組五：Google Sheets CRM](#9-模組五google-sheets-crm)
10. [OpenClaw Agent 整合](#10-openclaw-agent-整合)
11. [部署指南](#11-部署指南)
12. [驗收標準](#12-驗收標準)
13. [維護與故障排除](#13-維護與故障排除)

---

## 1. 專案總覽

### 功能範圍

| 功能 | 說明 |
|------|------|
| 591 自動監控 | 定時爬取指定區域 591 屋主自售物件 |
| AI 智能評分 | Claude AI 分析物件急迫度，0-100 分 |
| Telegram 推播 | 每日推播高分物件到業務的 Telegram |
| 追蹤提醒 | 根據 CRM 追蹤日期自動推播提醒 |
| Google Sheets CRM | 物件自動寫入、狀態追蹤、零手動輸入 |

### 系統架構圖

```
┌─────────────────────────────────────────────────┐
│                 OpenClaw Agent                   │
│            （主控排程 + 任務編排）                  │
└──────────┬──────────┬──────────┬────────────────┘
           │          │          │
           ▼          ▼          ▼
┌──────────────┐ ┌──────────┐ ┌─────────────────┐
│  591 爬蟲引擎 │ │ AI 評分  │ │  通知 & CRM      │
│              │ │  引擎    │ │                  │
│ - 定時爬取    │ │          │ │ - Telegram Bot   │
│ - 資料解析    │ │ - Claude │ │ - Google Sheets  │
│ - 去重過濾    │ │   API    │ │ - 追蹤提醒排程    │
└──────┬───────┘ │ - 評分   │ └────────┬────────┘
       │         │   0-100  │          │
       │         └────┬─────┘          │
       │              │                │
       ▼              ▼                ▼
┌──────────────────────────────────────────────────┐
│              本地 JSON/SQLite 資料儲存              │
└──────────────────────────────────────────────────┘
```

---

## 2. 技術棧與環境需求

### 技術棧

| 技術 | 版本 | 用途 |
|------|------|------|
| Node.js | >= 18.x | 執行環境 |
| TypeScript | >= 5.x | 開發語言 |
| OpenClaw | latest | AI Agent 框架 |
| Claude API | claude-sonnet-4-20250514 | AI 分析與評分 |
| Telegram Bot API | - | 推播通知 |
| Google Sheets API | v4 | CRM 資料存取 |
| node-cron | >= 3.x | 排程管理 |
| Playwright / Puppeteer | latest | 591 爬蟲（備用） |

### 伺服器需求

| 項目 | 最低規格 |
|------|---------|
| 作業系統 | Ubuntu 22.04 LTS / macOS |
| CPU | 2 Core |
| RAM | 4 GB |
| 硬碟 | 20 GB SSD |
| 網路 | 固定 IP（建議）或動態 IP + Proxy |
| 運行方式 | 24 小時持續運行（PM2 管理） |
| 部署方式 | 客戶自有 Mac Mini（免伺服器費用） |

### API 費用預估（月）— 客戶自行負擔

> **⚠️ 以下費用皆由客戶自行負擔，不包含在 NT$168,000 建置費中。**

| 項目 | 費用 |
|------|------|
| Claude API | NT$2,000-3,000 |
| 伺服器 | 免費（使用客戶自有 Mac Mini） |
| Proxy（選配，防 591 封鎖） | NT$0-500 |
| Telegram + Google Sheets | 免費 |
| **客戶月營運成本** | **NT$2,000-3,500** |

---

## 3. 專案目錄結構

```
house-agent-standard/
├── src/
│   ├── index.ts                 # 主入口，OpenClaw Agent 初始化
│   ├── config.ts                # 設定檔載入
│   ├── agent/
│   │   └── house-agent.ts       # OpenClaw Agent 定義
│   ├── crawler/
│   │   ├── crawler-591.ts       # 591 爬蟲核心邏輯
│   │   ├── parser-591.ts        # 591 頁面解析器
│   │   └── anti-ban.ts          # 反偵測策略
│   ├── ai/
│   │   ├── scoring-engine.ts    # AI 評分引擎
│   │   └── prompts.ts           # Prompt 模板
│   ├── telegram/
│   │   ├── bot.ts               # Telegram Bot 初始化
│   │   ├── daily-push.ts        # 每日推播邏輯
│   │   ├── tracker-reminder.ts  # 追蹤提醒
│   │   └── templates.ts         # 訊息模板
│   ├── crm/
│   │   ├── google-sheets.ts     # Google Sheets API 串接
│   │   └── schema.ts            # CRM 欄位定義
│   ├── storage/
│   │   ├── database.ts          # 本地資料庫（SQLite）
│   │   └── models.ts            # 資料模型
│   └── utils/
│       ├── logger.ts            # 日誌工具
│       └── helpers.ts           # 通用工具函式
├── config/
│   ├── config.yaml              # 主設定檔
│   └── config.example.yaml      # 設定範例
├── data/                        # 本地資料儲存
├── logs/                        # 日誌檔案
├── package.json
├── tsconfig.json
└── README.md
```

---

## 4. 設定檔規格

### config.yaml

```yaml
# ===== 591 爬蟲設定 =====
crawler:
  # 監控區域（縣市 + 鄉鎮市區）
  areas:
    - county: "南投縣"
      district: "竹山鎮"

  # 爬取間隔（分鐘）
  interval_minutes: 30

  # 只抓屋主自售
  owner_only: true

  # 房屋類型篩選（空 = 全部）
  house_types: []  # "透天", "公寓", "華廈", "大樓", "土地"

  # Proxy 設定（選填）
  proxy:
    enabled: false
    url: "http://proxy-server:port"
    rotate: true

# ===== Claude AI 設定 =====
ai:
  api_key: "${CLAUDE_API_KEY}"
  model: "claude-sonnet-4-20250514"
  # 評分門檻（低於此分數不推播）
  score_threshold: 60
  # 每次最多分析幾個物件
  batch_size: 20

# ===== Telegram 設定 =====
telegram:
  bot_token: "${TELEGRAM_BOT_TOKEN}"
  chat_id: "${TELEGRAM_CHAT_ID}"
  # 每日推播時間（24 小時制）
  daily_push_time: "08:00"
  # 最多推播幾個物件
  max_push_items: 5

# ===== Google Sheets CRM 設定 =====
google_sheets:
  credentials_path: "./config/google-credentials.json"
  spreadsheet_id: "${GOOGLE_SHEET_ID}"
  sheet_name: "物件追蹤"

# ===== 系統設定 =====
system:
  timezone: "Asia/Taipei"
  log_level: "info"  # debug, info, warn, error
  data_dir: "./data"
```

---

## 5. 模組一：591 爬蟲監控引擎

### 功能說明

定時爬取 591 房屋交易網的「屋主自售」專區，解析物件資料，與本地資料庫比對，識別新上架物件。

### 爬取目標 URL

```
https://sale.591.com.tw/?shType=list&regionid={county_id}&sectionid={district_id}&showMore=1&other=owner
```

### 關鍵參數

| 參數 | 說明 | 範例值 |
|------|------|--------|
| regionid | 縣市代碼 | 10 (南投縣) |
| sectionid | 鄉鎮區代碼 | 對應各鄉鎮 |
| other | 篩選條件 | owner (屋主自售) |

### 資料模型

```typescript
interface Property591 {
  id: string;              // 591 物件 ID
  title: string;           // 標題
  address: string;         // 地址
  price: number;           // 開價（萬）
  pricePerPing: number;    // 每坪單價（萬）
  area: number;            // 坪數
  houseType: string;       // 房屋類型（透天/公寓/...）
  rooms: string;           // 格局（3房2廳2衛）
  floor: string;           // 樓層
  age: number;             // 屋齡
  ownerType: string;       // "屋主" | "代理人"
  description: string;     // 物件描述全文
  publishDate: string;     // 刊登日期
  url: string;             // 物件連結
  images: string[];        // 圖片 URL
  crawledAt: string;       // 爬取時間 (ISO 8601)
  isNew: boolean;          // 是否為新發現物件
}
```

### 實作邏輯

```typescript
// src/crawler/crawler-591.ts

import { chromium } from 'playwright';
import { AntibanStrategy } from './anti-ban';
import { parse591Page } from './parser-591';
import { db } from '../storage/database';
import { logger } from '../utils/logger';

export class Crawler591 {
  private config: CrawlerConfig;

  constructor(config: CrawlerConfig) {
    this.config = config;
  }

  async crawl(): Promise<Property591[]> {
    const browser = await chromium.launch({
      headless: true,
      args: ['--no-sandbox'],
    });

    const newProperties: Property591[] = [];

    try {
      for (const area of this.config.areas) {
        const context = await browser.newContext({
          userAgent: AntibanStrategy.getRandomUA(),
          viewport: { width: 1920, height: 1080 },
        });

        const page = await context.newPage();

        // 隨機延遲，模擬人類行為
        await AntibanStrategy.randomDelay(2000, 5000);

        const url = this.buildUrl(area);
        logger.info(`Crawling: ${area.county} ${area.district}`);

        await page.goto(url, { waitUntil: 'networkidle' });

        // 解析物件列表
        const properties = await parse591Page(page);

        // 與資料庫比對，找出新物件
        for (const prop of properties) {
          const exists = await db.propertyExists(prop.id);
          if (!exists) {
            prop.isNew = true;
            newProperties.push(prop);
            await db.insertProperty(prop);
            logger.info(`New property found: ${prop.title} (${prop.price}萬)`);
          }
        }

        await context.close();
      }
    } finally {
      await browser.close();
    }

    logger.info(`Crawl complete. ${newProperties.length} new properties found.`);
    return newProperties;
  }

  private buildUrl(area: AreaConfig): string {
    const countyId = COUNTY_MAP[area.county];
    const districtId = DISTRICT_MAP[area.county]?.[area.district];
    return `https://sale.591.com.tw/?shType=list&regionid=${countyId}&sectionid=${districtId}&showMore=1&other=owner`;
  }
}
```

### 591 頁面解析器

```typescript
// src/crawler/parser-591.ts

import { Page } from 'playwright';

export async function parse591Page(page: Page): Promise<Property591[]> {
  const properties: Property591[] = [];

  // 等待物件列表載入
  await page.waitForSelector('.vue-list-rent-item', { timeout: 10000 });

  const items = await page.$$('.vue-list-rent-item');

  for (const item of items) {
    const property: Property591 = {
      id: await item.getAttribute('data-bind') || '',
      title: await item.$eval('.item-title', el => el.textContent?.trim() || ''),
      address: await item.$eval('.item-area', el => el.textContent?.trim() || ''),
      price: parsePrice(await item.$eval('.item-price', el => el.textContent?.trim() || '0')),
      pricePerPing: 0,
      area: parseFloat(await item.$eval('.item-area-num', el => el.textContent?.trim() || '0')),
      houseType: await item.$eval('.item-house-type', el => el.textContent?.trim() || ''),
      rooms: await item.$eval('.item-layout', el => el.textContent?.trim() || ''),
      floor: await item.$eval('.item-floor', el => el.textContent?.trim() || ''),
      age: 0,
      ownerType: '屋主',
      description: '',
      publishDate: '',
      url: `https://sale.591.com.tw/home/${await item.getAttribute('data-bind')}`,
      images: [],
      crawledAt: new Date().toISOString(),
      isNew: false,
    };

    // 計算每坪單價
    if (property.area > 0) {
      property.pricePerPing = Math.round((property.price / property.area) * 100) / 100;
    }

    properties.push(property);
  }

  return properties;
}

function parsePrice(priceStr: string): number {
  // "800 萬" => 800
  return parseFloat(priceStr.replace(/[^\d.]/g, '')) || 0;
}
```

### Anti-Ban 策略

```typescript
// src/crawler/anti-ban.ts

export class AntibanStrategy {
  private static userAgents = [
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/120.0.0.0 Safari/537.36',
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 Chrome/120.0.0.0 Safari/537.36',
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:121.0) Gecko/20100101 Firefox/121.0',
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 Safari/605.1.15',
  ];

  static getRandomUA(): string {
    return this.userAgents[Math.floor(Math.random() * this.userAgents.length)];
  }

  static async randomDelay(min: number, max: number): Promise<void> {
    const delay = Math.floor(Math.random() * (max - min + 1)) + min;
    await new Promise(resolve => setTimeout(resolve, delay));
  }

  // 每次爬取間隔隨機化（base ± 30%）
  static getRandomInterval(baseMinutes: number): number {
    const variation = baseMinutes * 0.3;
    const min = baseMinutes - variation;
    const max = baseMinutes + variation;
    return Math.floor(Math.random() * (max - min + 1) + min) * 60 * 1000;
  }
}
```

### 注意事項

- 591 前端使用 Vue.js SPA，需要等待 JS 渲染完成
- 建議使用 Playwright 而非純 HTTP 請求，因為 591 有 JS-based 的反爬機制
- 爬取頻率建議 30 分鐘一次，避免 IP 被封鎖
- 若被封鎖，自動切換 Proxy 或等待 1 小時後重試
- 591 selector 可能因改版而變化，需定期檢查維護

---

## 6. 模組二：AI 評分引擎

### 功能說明

使用 Claude API 分析每個物件的文字資訊，判斷屋主急迫度、成交機率，產出 0-100 的評分和判斷理由。

### 評分維度

| 維度 | 權重 | 判斷依據 |
|------|------|---------|
| 急迫度 | 40% | 標題含「急售」「急」「降價」、描述提到搬家/資金需求 |
| 價格合理度 | 25% | 與同區域行情比較，低於行情 = 高分 |
| 屋主類型 | 20% | 屋主自售 > 代理人，自售更有議價空間 |
| 物件條件 | 15% | 屋齡、坪數、格局、樓層等條件 |

### Claude API Prompt 設計

```typescript
// src/ai/prompts.ts

export function buildScoringPrompt(property: Property591, areaContext: string): string {
  return `你是一位有 10 年經驗的台灣房仲業務專家。請分析以下物件並給出評分。

## 物件資訊
- 標題：${property.title}
- 地址：${property.address}
- 開價：${property.price} 萬
- 坪數：${property.area} 坪
- 每坪單價：${property.pricePerPing} 萬/坪
- 房屋類型：${property.houseType}
- 格局：${property.rooms}
- 樓層：${property.floor}
- 屋主類型：${property.ownerType}
- 刊登日期：${property.publishDate}
- 物件描述：${property.description}

## 區域行情參考
${areaContext}

## 請回傳 JSON 格式：
{
  "score": <0-100 整數>,
  "urgency": "<急售|可能急售|一般|觀望>",
  "owner_type_analysis": "<屋主類型判斷說明>",
  "price_analysis": "<價格分析，與行情比較>",
  "key_signals": ["<關鍵訊號1>", "<關鍵訊號2>"],
  "recommendation": "<建議行動：今天就打電話 / 可以聯繫 / 先觀察>",
  "reason": "<一句話總結判斷理由>"
}

注意：
1. 標題或描述出現「急」「急售」「降價」「出國」「搬家」「資金」等字眼，急迫度要明顯提高
2. 刊登超過 30 天的物件，可能更願意降價，但不一定急迫
3. 每坪單價明顯低於行情 10% 以上，是重要訊號
4. 只回傳 JSON，不要有其他文字`;
}
```

### AI 評分引擎實作

```typescript
// src/ai/scoring-engine.ts

import Anthropic from '@anthropic-ai/sdk';
import { buildScoringPrompt } from './prompts';
import { logger } from '../utils/logger';

export interface ScoringResult {
  score: number;
  urgency: string;
  owner_type_analysis: string;
  price_analysis: string;
  key_signals: string[];
  recommendation: string;
  reason: string;
}

export class ScoringEngine {
  private client: Anthropic;
  private model: string;

  constructor(apiKey: string, model: string) {
    this.client = new Anthropic({ apiKey });
    this.model = model;
  }

  async scoreProperty(property: Property591): Promise<ScoringResult> {
    const areaContext = await this.getAreaContext(property.address);
    const prompt = buildScoringPrompt(property, areaContext);

    try {
      const response = await this.client.messages.create({
        model: this.model,
        max_tokens: 1024,
        messages: [{ role: 'user', content: prompt }],
      });

      const text = response.content[0].type === 'text'
        ? response.content[0].text : '';

      // 解析 JSON 回應
      const jsonMatch = text.match(/\{[\s\S]*\}/);
      if (!jsonMatch) {
        throw new Error('AI response is not valid JSON');
      }

      const result: ScoringResult = JSON.parse(jsonMatch[0]);

      // 確保 score 在 0-100 範圍
      result.score = Math.max(0, Math.min(100, Math.round(result.score)));

      logger.info(`Scored "${property.title}": ${result.score} (${result.urgency})`);
      return result;

    } catch (error) {
      logger.error(`Scoring failed for "${property.title}": ${error}`);
      // 回傳預設評分
      return {
        score: 50,
        urgency: '一般',
        owner_type_analysis: '無法分析',
        price_analysis: '無法分析',
        key_signals: [],
        recommendation: '需要人工判斷',
        reason: 'AI 分析失敗，建議人工查看',
      };
    }
  }

  async scoreBatch(properties: Property591[]): Promise<Map<string, ScoringResult>> {
    const results = new Map<string, ScoringResult>();

    // 依序評分，避免 API rate limit
    for (const property of properties) {
      const result = await this.scoreProperty(property);
      results.set(property.id, result);

      // API 呼叫間隔 1 秒
      await new Promise(resolve => setTimeout(resolve, 1000));
    }

    return results;
  }

  private async getAreaContext(address: string): Promise<string> {
    // 從本地資料庫取得該區域的歷史行情作為參考
    // 初期可以硬編碼，後續再改為動態查詢
    return `竹山鎮透天行情：約 600-1200 萬，每坪 8-15 萬
竹山鎮公寓行情：約 300-600 萬，每坪 6-10 萬
竹山鎮土地行情：每坪 2-8 萬（視地目）`;
  }
}
```

### API 費用估算

| 項目 | 數值 |
|------|------|
| 每日新物件數（預估） | 5-15 個 |
| 每個物件 prompt token | ~500 tokens |
| 每個物件 response token | ~300 tokens |
| 每日 API 成本 | ~NT$5-15 |
| 每月 API 成本 | ~NT$150-450 |

> 使用 claude-sonnet 而非 claude-opus 可以有效控制成本，且對此場景的準確度已足夠。

---

## 7. 模組三：Telegram Bot 推播

### 功能說明

每天早上在指定時間推播當日高分物件（超過門檻分數）到業務的 Telegram。

### Bot 設定流程

1. 在 Telegram 搜尋 `@BotFather`
2. 發送 `/newbot`，依指示設定名稱
3. 取得 Bot Token（格式：`123456:ABC-DEF...`）
4. 將 Bot 加入目標群組或私聊
5. 取得 Chat ID（透過 `getUpdates` API）

### 訊息格式模板

```typescript
// src/telegram/templates.ts

export function buildDailyPushMessage(
  date: string,
  totalNew: number,
  topProperties: ScoredProperty[]
): string {
  let msg = `📊 *【${date} 今日高價值物件】*\n\n`;
  msg += `今天監控到 *${totalNew} 個新物件*，\n`;
  msg += `AI 篩選出 *${topProperties.length} 個值得聯繫*：\n\n`;

  topProperties.forEach((prop, index) => {
    const scoreEmoji = prop.score >= 85 ? '🔴' : prop.score >= 70 ? '🟡' : '🔵';

    msg += `━━━━━━━━━━━━━━\n`;
    msg += `*${index + 1}. ${prop.title}*\n`;
    msg += `📍 來源：${prop.source}\n`;
    msg += `💰 開價：${prop.price} 萬\n`;
    msg += `${scoreEmoji} AI 評分：*${prop.score} 分*\n`;
    msg += `📝 判斷：${prop.reason}\n`;
    msg += `👉 建議：*${prop.recommendation}*\n`;
    msg += `🔗 [查看物件](${prop.url})\n\n`;
  });

  msg += `━━━━━━━━━━━━━━\n`;
  msg += `_以上由 AI 房仲助理自動產生_`;

  return msg;
}

export function buildTrackerReminderMessage(
  property: TrackedProperty
): string {
  return `⏰ *追蹤提醒*\n\n` +
    `物件：*${property.title}*\n` +
    `屋主：${property.ownerName}\n` +
    `狀態：${property.status}\n` +
    `備註：${property.notes}\n\n` +
    `_今天該跟進了！_`;
}
```

### Bot 核心實作

```typescript
// src/telegram/bot.ts

import TelegramBot from 'node-telegram-bot-api';
import { logger } from '../utils/logger';

export class HouseBot {
  private bot: TelegramBot;
  private chatId: string;

  constructor(token: string, chatId: string) {
    this.bot = new TelegramBot(token, { polling: false });
    this.chatId = chatId;
  }

  async sendMessage(text: string): Promise<void> {
    try {
      await this.bot.sendMessage(this.chatId, text, {
        parse_mode: 'Markdown',
        disable_web_page_preview: true,
      });
      logger.info('Telegram message sent successfully');
    } catch (error) {
      logger.error(`Failed to send Telegram message: ${error}`);
      throw error;
    }
  }

  async sendDailyPush(properties: ScoredProperty[]): Promise<void> {
    const today = new Date().toISOString().split('T')[0];
    const totalNew = await db.countTodayNewProperties();
    const message = buildDailyPushMessage(today, totalNew, properties);
    await this.sendMessage(message);
  }
}
```

### 每日推播排程

```typescript
// src/telegram/daily-push.ts

import cron from 'node-cron';
import { HouseBot } from './bot';
import { db } from '../storage/database';
import { logger } from '../utils/logger';

export function scheduleDailyPush(bot: HouseBot, pushTime: string): void {
  const [hour, minute] = pushTime.split(':');

  // 每天固定時間推播
  cron.schedule(`${minute} ${hour} * * *`, async () => {
    logger.info('Starting daily push...');

    try {
      // 取得今日高分物件（已排序）
      const topProperties = await db.getTopScoredProperties({
        minScore: 60,
        limit: 5,
        since: getYesterday(),
      });

      if (topProperties.length === 0) {
        await bot.sendMessage('📊 今日無新的高價值物件，AI 持續監控中...');
        return;
      }

      await bot.sendDailyPush(topProperties);
      logger.info(`Daily push sent: ${topProperties.length} properties`);

    } catch (error) {
      logger.error(`Daily push failed: ${error}`);
    }
  }, {
    timezone: 'Asia/Taipei',
  });

  logger.info(`Daily push scheduled at ${pushTime}`);
}
```

---

## 8. 模組四：Telegram 追蹤提醒

### 功能說明

每天早上檢查 CRM 中「下次追蹤」日期為今天的物件，自動推播提醒。

### 實作邏輯

```typescript
// src/telegram/tracker-reminder.ts

import cron from 'node-cron';
import { HouseBot } from './bot';
import { db } from '../storage/database';

export function scheduleTrackerReminder(bot: HouseBot): void {
  // 每天早上 08:30 檢查追蹤提醒
  cron.schedule('30 8 * * *', async () => {
    const today = new Date().toISOString().split('T')[0];

    const dueProperties = await db.getPropertiesDueForFollowUp(today);

    for (const prop of dueProperties) {
      const message = buildTrackerReminderMessage(prop);
      await bot.sendMessage(message);

      // 間隔 2 秒，避免 Telegram rate limit
      await new Promise(resolve => setTimeout(resolve, 2000));
    }
  }, {
    timezone: 'Asia/Taipei',
  });
}
```

### CRM 追蹤狀態流程

```
待聯繫 → 已聯繫 → 已約看屋 → 已簽委託 / 放棄
                ↗ 需再跟進（設定下次追蹤日期）
```

---

## 9. 模組五：Google Sheets CRM

### 功能說明

將所有監控到的物件自動寫入 Google Sheets，作為輕量化 CRM 系統。

### Google Sheets 欄位設計

| 欄位 | 類型 | 說明 |
|------|------|------|
| A: 物件ID | string | 591 物件 ID |
| B: 標題 | string | 物件標題 |
| C: 地址 | string | 物件地址 |
| D: 來源 | string | 591 |
| E: 開價(萬) | number | 開價 |
| F: 坪數 | number | 坪數 |
| G: 類型 | string | 房屋類型 |
| H: 屋主 | string | 屋主姓名（業務手動填入） |
| I: AI評分 | number | 0-100 |
| J: 急迫度 | string | 急售/可能急售/一般/觀望 |
| K: 狀態 | string | 待聯繫/已聯繫/已約看屋/... |
| L: 下次追蹤 | date | 下次跟進日期 |
| M: 備註 | string | 業務手動備註 |
| N: AI判斷 | string | AI 判斷理由摘要 |
| O: 連結 | string | 591 物件連結 |
| P: 發現時間 | datetime | 系統第一次爬到的時間 |

### Google Sheets API 串接

```typescript
// src/crm/google-sheets.ts

import { google } from 'googleapis';
import { logger } from '../utils/logger';

export class GoogleSheetsCRM {
  private sheets: any;
  private spreadsheetId: string;
  private sheetName: string;

  constructor(credentialsPath: string, spreadsheetId: string, sheetName: string) {
    this.spreadsheetId = spreadsheetId;
    this.sheetName = sheetName;
  }

  async init(): Promise<void> {
    const auth = new google.auth.GoogleAuth({
      keyFile: this.credentialsPath,
      scopes: ['https://www.googleapis.com/auth/spreadsheets'],
    });

    this.sheets = google.sheets({ version: 'v4', auth });
    logger.info('Google Sheets CRM initialized');
  }

  async appendProperty(property: Property591, scoring: ScoringResult): Promise<void> {
    const row = [
      property.id,
      property.title,
      property.address,
      '591',
      property.price,
      property.area,
      property.houseType,
      '',  // 屋主姓名（業務手動填入）
      scoring.score,
      scoring.urgency,
      '待聯繫',
      '',  // 下次追蹤（業務手動填入）
      '',  // 備註
      scoring.reason,
      property.url,
      new Date().toISOString(),
    ];

    await this.sheets.spreadsheets.values.append({
      spreadsheetId: this.spreadsheetId,
      range: `${this.sheetName}!A:P`,
      valueInputOption: 'USER_ENTERED',
      requestBody: { values: [row] },
    });

    logger.info(`CRM: Added "${property.title}"`);
  }

  async getPropertiesDueForFollowUp(date: string): Promise<TrackedProperty[]> {
    const response = await this.sheets.spreadsheets.values.get({
      spreadsheetId: this.spreadsheetId,
      range: `${this.sheetName}!A:P`,
    });

    const rows = response.data.values || [];
    const dueProperties: TrackedProperty[] = [];

    // 從第 2 行開始（跳過 header）
    for (let i = 1; i < rows.length; i++) {
      const row = rows[i];
      const followUpDate = row[11]; // L 欄：下次追蹤

      if (followUpDate === date) {
        dueProperties.push({
          id: row[0],
          title: row[1],
          ownerName: row[7] || '未知',
          status: row[10],
          notes: row[12] || '',
          followUpDate: row[11],
        });
      }
    }

    return dueProperties;
  }
}
```

### Google API 設定步驟

1. 前往 [Google Cloud Console](https://console.cloud.google.com/)
2. 建立專案 → 啟用 Google Sheets API
3. 建立服務帳戶（Service Account）
4. 下載 JSON 憑證檔，存為 `config/google-credentials.json`
5. 在 Google Sheets 中將試算表共用給服務帳戶的 email

---

## 10. OpenClaw Agent 整合

### Agent 定義

```typescript
// src/agent/house-agent.ts

import { Agent } from 'openclaw';
import { Crawler591 } from '../crawler/crawler-591';
import { ScoringEngine } from '../ai/scoring-engine';
import { HouseBot } from '../telegram/bot';
import { GoogleSheetsCRM } from '../crm/google-sheets';
import { loadConfig } from '../config';

export function createHouseAgent(): Agent {
  const config = loadConfig();

  const crawler = new Crawler591(config.crawler);
  const scorer = new ScoringEngine(config.ai.api_key, config.ai.model);
  const bot = new HouseBot(config.telegram.bot_token, config.telegram.chat_id);
  const crm = new GoogleSheetsCRM(
    config.google_sheets.credentials_path,
    config.google_sheets.spreadsheet_id,
    config.google_sheets.sheet_name,
  );

  const agent = new Agent({
    name: 'house-agent',
    description: 'AI 房仲智能助理 - 591 監控 + AI 評分 + 推播',

    tools: [
      {
        name: 'crawl_591',
        description: '爬取 591 新物件',
        execute: async () => {
          const newProperties = await crawler.crawl();
          return { newCount: newProperties.length, properties: newProperties };
        },
      },
      {
        name: 'score_properties',
        description: '使用 AI 評分新物件',
        execute: async (input: { properties: Property591[] }) => {
          const scores = await scorer.scoreBatch(input.properties);
          return Object.fromEntries(scores);
        },
      },
      {
        name: 'save_to_crm',
        description: '將評分結果寫入 Google Sheets CRM',
        execute: async (input: { property: Property591; score: ScoringResult }) => {
          await crm.appendProperty(input.property, input.score);
          return { success: true };
        },
      },
      {
        name: 'send_notification',
        description: '發送 Telegram 推播通知',
        execute: async (input: { message: string }) => {
          await bot.sendMessage(input.message);
          return { success: true };
        },
      },
    ],

    // Agent 的主要工作流程
    workflow: async (ctx) => {
      // Step 1: 爬取新物件
      const { properties } = await ctx.useTool('crawl_591');

      if (properties.length === 0) {
        return 'No new properties found';
      }

      // Step 2: AI 評分
      const scores = await ctx.useTool('score_properties', { properties });

      // Step 3: 寫入 CRM
      for (const prop of properties) {
        const score = scores[prop.id];
        await ctx.useTool('save_to_crm', { property: prop, score });
      }

      // Step 4: 推播高分物件（由排程觸發，此處僅儲存）
      return `Processed ${properties.length} properties`;
    },
  });

  return agent;
}
```

### 主入口

```typescript
// src/index.ts

import { createHouseAgent } from './agent/house-agent';
import { scheduleDailyPush } from './telegram/daily-push';
import { scheduleTrackerReminder } from './telegram/tracker-reminder';
import { HouseBot } from './telegram/bot';
import { loadConfig } from './config';
import { db } from './storage/database';
import cron from 'node-cron';
import { logger } from './utils/logger';

async function main() {
  const config = loadConfig();

  // 初始化資料庫
  await db.init();

  // 建立 Agent
  const agent = createHouseAgent();

  // 初始化 Telegram Bot
  const bot = new HouseBot(config.telegram.bot_token, config.telegram.chat_id);

  // 排程：定時爬取
  const intervalMs = config.crawler.interval_minutes * 60 * 1000;
  setInterval(async () => {
    try {
      await agent.run();
    } catch (error) {
      logger.error(`Agent run failed: ${error}`);
    }
  }, intervalMs);

  // 排程：每日推播
  scheduleDailyPush(bot, config.telegram.daily_push_time);

  // 排程：追蹤提醒
  scheduleTrackerReminder(bot);

  // 首次執行
  await agent.run();

  logger.info('House Agent started successfully');
  await bot.sendMessage('🤖 AI 房仲助理已啟動！開始監控 591...');
}

main().catch(console.error);
```

---

## 11. 部署指南

### Step 1：環境安裝

```bash
# 安裝 Node.js (使用 nvm)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 18
nvm use 18

# 安裝 PM2（程序管理）
npm install -g pm2

# 安裝 Playwright 瀏覽器
npx playwright install chromium
```

### Step 2：專案設定

```bash
# Clone 專案
git clone <repo-url> house-agent
cd house-agent

# 安裝依賴
npm install

# 複製設定檔
cp config/config.example.yaml config/config.yaml

# 編輯設定檔
nano config/config.yaml
```

### Step 3：API Key 設定

```bash
# 設定環境變數
export CLAUDE_API_KEY="sk-ant-..."
export TELEGRAM_BOT_TOKEN="123456:ABC..."
export TELEGRAM_CHAT_ID="-100123456789"
export GOOGLE_SHEET_ID="1abc..."
```

### Step 4：啟動

```bash
# 編譯 TypeScript
npm run build

# 使用 PM2 啟動（背景執行 + 自動重啟）
pm2 start dist/index.js --name house-agent

# 查看日誌
pm2 logs house-agent

# 設定開機自動啟動
pm2 save
pm2 startup
```

### Step 5：驗證

```bash
# 確認程序在運行
pm2 status

# 手動觸發一次爬取測試
curl http://localhost:3001/api/trigger-crawl

# 確認 Telegram 有收到訊息
# 確認 Google Sheets 有新資料
```

---

## 12. 驗收標準

### 模組驗收 Checklist

| # | 驗收項目 | 通過條件 |
|---|---------|---------|
| 1 | 591 爬蟲正常運作 | 能成功爬取指定區域物件，至少連續 24 小時不中斷 |
| 2 | 新物件偵測 | 手動在 591 刊登測試物件，系統能在下次爬取時發現 |
| 3 | AI 評分正確 | 評分結果合理（急售物件分數 > 一般物件），10 個測試物件人工確認 |
| 4 | Telegram 推播 | 每天指定時間收到推播，格式正確，連結可開啟 |
| 5 | 追蹤提醒 | 在 CRM 設定追蹤日期，當天收到提醒 |
| 6 | Google Sheets CRM | 新物件自動寫入，欄位完整，不重複 |
| 7 | 系統穩定性 | PM2 管理，異常自動重啟，日誌記錄完整 |
| 8 | 操作文件 | 提供操作手冊，客戶能自行修改監控區域 |

---

## 13. 維護與故障排除

### 常見問題

| 問題 | 原因 | 解法 |
|------|------|------|
| 591 爬不到資料 | IP 被封或頁面改版 | 切換 Proxy / 更新 selector |
| AI 評分異常 | Prompt 需調整 | 修改 prompts.ts 中的 prompt |
| Telegram 發不出 | Bot Token 過期 | 重新跟 BotFather 申請 |
| Google Sheets 寫入失敗 | 憑證過期或權限不足 | 重新授權服務帳戶 |
| 記憶體不足 | Playwright 佔用 | 確保爬完後 close browser |

### 591 改版應對策略

1. 建立 `selector-monitor` 腳本，每天檢測關鍵 selector 是否存在
2. selector 失效時自動通知開發者
3. 在 `parser-591.ts` 中使用 fallback selector 設計
4. 維護一份 selector 版本歷史，方便快速回滾

### 日誌查看

```bash
# 查看即時日誌
pm2 logs house-agent

# 查看錯誤日誌
pm2 logs house-agent --err

# 查看歷史日誌
cat logs/house-agent-$(date +%Y-%m-%d).log
```

---

> **文件版本：** v1.0
> **建立日期：** 2026-03-20
> **適用方案：** 標準版（NT$168,000）
