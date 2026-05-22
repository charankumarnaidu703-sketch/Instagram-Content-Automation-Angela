<p align="center">
  <img src="https://img.shields.io/badge/Platform-n8n-FF6D5A?style=for-the-badge&logo=n8n&logoColor=white" />
  <img src="https://img.shields.io/badge/AI-OpenAI%20%7C%20Gemini-412991?style=for-the-badge&logo=openai&logoColor=white" />
  <img src="https://img.shields.io/badge/Visuals-Placid.app-5C4EE5?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Messaging-Telegram%20Bot-26A5E4?style=for-the-badge&logo=telegram&logoColor=white" />
  <img src="https://img.shields.io/badge/Publishing-Instagram%20Graph%20API-E4405F?style=for-the-badge&logo=instagram&logoColor=white" />
</p>

# 📸 Instagram Content Automation — Angela

> **A fully autonomous, AI-powered content pipeline that plans, designs, reviews, publishes, and optimizes Instagram content — producing 21 pieces per week with zero manual effort.**

Built for a real-world beauty & wellness brand (**Angela Italy**), this system transforms a single Google Sheet into a complete social media operation. Five interconnected **n8n workflows** orchestrate the entire lifecycle — from AI-driven content ideation to automated publishing via the Instagram Graph API — with a Telegram bot providing human-in-the-loop approval.

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        WEEKLY AUTOMATION CYCLE                         │
│                                                                        │
│   ┌──────────────┐    ┌──────────────┐    ┌──────────────────────┐     │
│   │   CONTENT     │───▶│   VISUAL      │───▶│  TELEGRAM APPROVAL   │     │
│   │   PLANNING    │    │   CREATION    │    │  BOT (Human Review)  │     │
│   │   ENGINE      │    │   ENGINE      │    │                      │     │
│   │               │    │               │    │  ✅ Approve All      │     │
│   │  7 Feed Posts │    │  Placid.app   │    │  🖼️ Change Image     │     │
│   │ 14 Stories    │    │  Templates    │    │  ✏️ Edit Text/Hook   │     │
│   │  AI-Powered   │    │  Batch Render │    │  🔄 Regenerate       │     │
│   └──────────────┘    └──────────────┘    └──────────┬───────────┘     │
│                                                       │                 │
│                                            ┌──────────▼───────────┐     │
│   ┌──────────────┐                         │   PUBLISHING         │     │
│   │   QUALITY     │◀────────────────────── │   ENGINE             │     │
│   │   CONTROL     │                         │                      │     │
│   │   AGENT       │    Feedback Loop        │  Auto-Publish via    │     │
│   │               │───────────────────────▶ │  Instagram Graph API │     │
│   │  Daily Digest │                         │  Every 15 min check  │     │
│   │  Weekly Report│                         └──────────────────────┘     │
│   │  AI Recommend.│                                                     │
│   └──────────────┘                                                      │
│                                                                        │
│   ┌────────────────────────────────────────────────────────────────┐    │
│   │              📊 GOOGLE SHEETS — Central Data Hub               │    │
│   │  Config │ Products │ Calendar │ Analytics │ Archive │ Logs     │    │
│   └────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## ⚙️ The Five Engines

### 1. 📝 Content Planning Engine
**`Content Planning Engine (Weekly - 21 Pieces).json`**

| Detail | Value |
|--------|-------|
| **Trigger** | Every Sunday at 7 PM (+ manual) |
| **Output** | 7 feed posts + 14 stories |
| **AI Models** | OpenAI GPT-4 (primary) → Gemini 1.5 Flash (fallback) |

**What it does:**
- Reads brand config (tone, audience, content pillars, blacklisted words) from Google Sheets
- Pulls last 30 days of analytics to identify top-performing formats, hooks, and topics
- Ingests the product catalog to strategically feature services
- Analyzes the content archive to avoid repetition and double-down on winners
- Generates 7 feed posts (3 single images, 2 carousels, 2 reels) with full captions, hooks, hashtags, CTAs, and visual direction
- Generates 14 stories (2 per day — morning + evening) with interactive elements (polls, quizzes, questions)
- Runs a **Quality Gate** — validates caption length, hook character count, hashtag count, blacklisted words, and missing fields
- Flags non-compliant content for human review
- Writes everything to the Content Calendar sheet

**Smart Features:**
- **Performance Memory** — identifies high-engagement and low-engagement topics from the archive and instructs the AI to replicate winners and avoid losers
- **Dynamic Format Allocation** — adjusts the single image / carousel / reel split based on template performance scores
- **Template Preferences** — feeds top-performing and weak-performing template data to guide visual direction

---

### 2. 🎨 Visual Creation Engine
**`Visual Creation Engine (Weekly Batch - Templated.io).json`**

| Detail | Value |
|--------|-------|
| **Trigger** | Every Sunday at 8 PM (after content planning) |
| **Rendering** | Placid.app API (images + videos) |
| **AI** | Gemini (layer mapping) |

**What it does:**
- Reads pending content from the Content Calendar
- Matches each post to the correct Placid template using intelligent pattern matching (supports `(page-N)` for carousels, `(scene-N)` for reels, `(variant-X)` for images)
- Maps content (hooks, captions, CTAs) to template layers via Gemini
- Renders images via `POST /api/rest/images` and videos via `POST /api/rest/videos`
- Uploads rendered assets to Google Drive (organized by content type and week)
- Updates the Content Calendar with `Visual_URL`, `Slide_URLs`, and `Visual_Created` status
- Sends a batch summary via Telegram with inline approval buttons

**Supported Content Types:**

| Type | Template Pattern | Output |
|------|-----------------|--------|
| Single Image | Direct UUID or name match | `.jpg` |
| Carousel | `template-name (page-1)`, `(page-2)`, ... | Multiple `.jpg` slides |
| Reel | `template-name (scene-1)`, `(scene-2)`, ... | `.mp4` video |
| Story | Direct UUID or story type match | `.jpg` |

---

### 3. 🤖 Telegram Content Approval Bot
**`Telegram Content Approval Bot.json`**

| Detail | Value |
|--------|-------|
| **Trigger** | Webhook (`POST /telegram-approval-bot`) |
| **Interface** | Telegram inline keyboard buttons |

**What it does:**
- Receives the weekly content batch preview via Telegram
- Provides inline action buttons for each batch:

| Action | Command | Description |
|--------|---------|-------------|
| ✅ Approve All | Callback button | Marks all `visuals_created` posts as `approved` |
| 🖼️ Change Image | `IMAGE PostID URL` | Swaps the visual and re-renders via Placid |
| ✏️ Change Text | `TEXT PostID new caption` / `HOOK PostID new hook` | AI-refines the text (GPT-4), re-renders the visual |
| 🔄 Regenerate | `REGEN PostID` | Fully regenerates content + visuals for a post |

- After each change, sends the updated visual back with new approval buttons
- Updates the Content Calendar with approval status, timestamps, and modified content

---

### 4. 📤 Publishing Engine
**`Publishing Engine (Auto-Publish Feed Posts).json`**

| Detail | Value |
|--------|-------|
| **Trigger** | Every 15 minutes (cron) |
| **API** | Instagram Graph API v18.0 / v25.0 |

**What it does:**
- Reads the Content Calendar for posts that are:
  - Status = `approved`
  - `Visual_Created` = `true`
  - `Already_Published` ≠ `true`
  - Scheduled time ≤ now + 10 minutes
- Publishes each content type via the Instagram Graph API:

| Type | API Flow |
|------|----------|
| **Single Image** | `POST /media` → `POST /media_publish` |
| **Carousel** | Create slide containers → `POST /media` (CAROUSEL type with children) → `POST /media_publish` |
| **Reel** | `POST /media` (REELS type) → wait 60s → `POST /media_publish` |
| **Story** | `POST /media` (STORIES type) → `POST /media_publish` |

- Updates the Content Calendar with `published` status, `Instagram_Post_ID`, and `Published_At` timestamp
- Sends a publishing report via email (Gmail)
- Logs results to a Publishing Log sheet

---

### 5. 📊 Quality Control Agent
**`Quality Control Agent (Daily + Weekly Analytics).json`**

| Detail | Value |
|--------|-------|
| **Trigger** | Daily at 9 PM + Weekly on Sunday at 6 PM |
| **API** | Instagram Graph API (Post Insights) |
| **AI** | OpenAI GPT-4 (recommendations) |

**What it does:**
- **Daily**: Fetches insights for today's published posts → generates a quick performance digest
- **Weekly**: Fetches insights for all posts published in the past 7 days → runs deep analysis

**Analytics Collected:**

| Content Type | Metrics |
|-------------|---------|
| Images / Carousels | `reach`, `likes`, `comments`, `shares`, `saved`, `total_interactions` |
| Reels | Above + `plays` |
| Stories | `reach` |

**Deep Analysis includes:**
- Average engagement by content type (images vs. carousels vs. reels)
- Best-performing format, hook style, and posting hour
- Optimal caption length and hashtag count
- CTA impact analysis (with CTA vs. without CTA engagement)
- Top 5 posts ranked by engagement

**AI Recommendations:**
- GPT-4 analyzes the performance data and returns exactly 5 actionable recommendations labeled as `HIGH IMPACT`, `MEDIUM IMPACT`, or `LOW EFFORT`
- Recommendations cover: content mix, hook/caption style, topics, story improvements, and timing
- All metrics and recommendations are written back to the Analytics Dashboard sheet

---

## 📊 Data Layer — Google Sheets Structure

The entire system is driven by a single Google Spreadsheet with 8 interconnected sheets:

| Sheet | Purpose | Key Fields |
|-------|---------|------------|
| **Config** | Brand settings, tone, audience, posting times, API tokens | `brandName`, `tone`, `audience`, `contentPillars`, `igAccessToken` |
| **Product List** | Catalog of services/products to feature | `ProductID`, `Name`, `Category`, `KeyBenefits`, `Price`, `Status` |
| **Content Calendar** | Master schedule — the single source of truth | `PostID`, `ContentType`, `ScheduledDate`, `Caption`, `Status`, `Visual_URL` |
| **Story Reminders** | Story schedule with interactive elements | `StoryType`, `MainText`, `InteractiveElement` (JSON), `Status` |
| **Flagged Content** | Review queue for QA failures | `PostID`, `ReviewFlags`, `AISource` |
| **Analytics Dashboard** | Performance data + AI recommendations | `EngagementRate`, `Likes`, `Reach`, `Ai recommendation` |
| **Content Archive** | Historical topics/hooks for de-duplication | `Topic`, `Hook`, `ContentType`, `Pillar` |
| **Logs** | Automation run logs | `Date`, `Status`, `Feed Posts`, `Stories`, `Flagged` |

---

## 🚀 Getting Started

### Prerequisites

| Requirement | Purpose |
|-------------|---------|
| [n8n](https://n8n.io/) (self-hosted or cloud) | Workflow automation engine |
| [OpenAI API Key](https://platform.openai.com/) | GPT-4 for content generation and text refinement |
| [Google Cloud OAuth 2.0](https://console.cloud.google.com/) | Google Sheets read/write access |
| [Instagram Graph API](https://developers.facebook.com/) | Publishing + insights (`igAccessToken` + `igAccountId`) |
| [Placid.app API Key](https://placid.app/) | Template-based visual rendering |
| [Telegram Bot Token](https://core.telegram.org/bots) | Content approval bot |

### Setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/charankumarnaidu703-sketch/Instagram-Content-Automation-Angela.git
   cd Instagram-Content-Automation-Angela
   ```

2. **Create the Google Spreadsheet**
   - Copy the sheet structure from [`Sheet Structure For automation`](./Sheet%20Structure%20For%20automation)
   - Fill in the **Config** sheet with your brand details, API tokens, and posting preferences
   - Add your products/services to the **Product List** sheet

3. **Import workflows into n8n**
   - Open your n8n instance
   - Import each `.json` workflow file:
     - `Content Planning Engine (Weekly - 21 Pieces).json`
     - `Visual Creation Engine (Weekly Batch - Templated.io).json`
     - `Telegram Content Approval Bot.json`
     - `Publishing Engine (Auto-Publish Feed Posts).json`
     - `Quality Control Agent (Daily + Weekly Analytics).json`

4. **Configure credentials in n8n**
   - **Google Sheets OAuth2** — link to your Google account
   - **OpenAI API** — add your API key
   - **Placid API** — add your Placid token
   - **Instagram Graph API** — set `igAccessToken` and `igAccountId` in the Config sheet
   - **Telegram Bot** — set `telegramBotToken` in the Config sheet
   - **Gmail OAuth2** — for publishing report emails

5. **Set up Placid templates**
   - Create templates in Placid.app matching your brand
   - Use naming conventions: `template-name (page-1)` for carousel slides
   - Add template UUIDs to the Content Calendar's `Preferred Template` column

6. **Register the Telegram webhook**
   ```
   https://api.telegram.org/bot<YOUR_BOT_TOKEN>/setWebhook?url=<YOUR_N8N_WEBHOOK_URL>/telegram-approval-bot
   ```

7. **Activate all workflows** in n8n and the system runs autonomously.

---

## 📅 Weekly Automation Timeline

| Day | Time | Engine | Action |
|-----|------|--------|--------|
| Sunday | 6:00 PM | Quality Control | Weekly analytics report + AI recommendations |
| Sunday | 7:00 PM | Content Planning | Generate 21 pieces for next week |
| Sunday | 8:00 PM | Visual Creation | Render all visuals via Placid |
| Sunday | ~8:30 PM | Telegram Bot | Send batch for approval |
| Mon–Sun | Every 15 min | Publishing Engine | Auto-publish approved content at scheduled times |
| Daily | 9:00 PM | Quality Control | Daily performance digest |

---

## 🛡️ Quality & Safety

- **Quality Gate** — Every AI-generated post is validated against configurable rules (caption length, hook length, hashtag count, blacklisted words)
- **Human-in-the-loop** — Nothing publishes without Telegram approval
- **AI Fallback Chain** — OpenAI GPT-4 → Gemini 1.5 Flash → static templates (ensures content is always generated)
- **Retry Logic** — All API calls have retry-on-fail with configurable max tries and wait intervals
- **Content De-duplication** — Archive-based topic avoidance prevents repetitive content
- **Blacklist Enforcement** — Banned words and hashtags are checked at generation and flagged

---

## 🔧 Tech Stack

| Layer | Technology |
|-------|-----------|
| **Orchestration** | n8n (workflow automation) |
| **AI — Content** | OpenAI GPT-4, Google Gemini 1.5 Flash |
| **AI — Text Refinement** | OpenAI GPT-4 (via Telegram bot) |
| **Visual Rendering** | Placid.app (images + videos) |
| **Data Store** | Google Sheets |
| **File Storage** | Google Drive |
| **Publishing** | Instagram Graph API (v18.0 / v25.0) |
| **Approval Interface** | Telegram Bot API |
| **Notifications** | Gmail, Telegram |

---

## 📁 Repository Structure

```
Instagram-Content-Automation-Angela/
│
├── Content Planning Engine (Weekly - 21 Pieces).json    # AI content generation workflow
├── Visual Creation Engine (Weekly Batch - Templated.io).json  # Visual rendering workflow
├── Telegram Content Approval Bot.json                    # Human review interface
├── Publishing Engine (Auto-Publish Feed Posts).json       # Instagram publishing workflow
├── Quality Control Agent (Daily + Weekly Analytics).json  # Performance tracking workflow
├── Sheet Structure For automation                        # Google Sheet schema documentation
├── .gitignore
└── README.md
```

---

## 📄 License

This project is proprietary and built for a specific client use case. Please reach out before reusing or adapting.

---

<p align="center">
  <b>Built with ❤️ by <a href="https://github.com/charankumarnaidu703-sketch">Charan Kumar</a></b>
  <br/>
  <sub>Automating creativity, one post at a time.</sub>
</p>
