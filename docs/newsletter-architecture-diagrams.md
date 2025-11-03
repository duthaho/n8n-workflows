# Daily Newsletter Aggregator - Visual Architecture

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Daily Newsletter Aggregator                   │
│                         n8n Workflow                             │
└─────────────────────────────────────────────────────────────────┘
                                │
                ┌───────────────┴───────────────┐
                │                               │
        ┌───────▼────────┐            ┌────────▼────────┐
        │  Part 1: Daily │            │  Part 2: Save   │
        │  Aggregation   │            │  & Research     │
        └───────┬────────┘            └────────┬────────┘
                │                               │
    ┌───────────┴──────────────┐               │
    │                          │               │
┌───▼────┐              ┌──────▼─────┐   ┌────▼─────┐
│ Fetch  │              │  Process   │   │ Webhook  │
│ & Parse│─────────────▶│  & Enhance │   │ Trigger  │
└────────┘              └──────┬─────┘   └────┬─────┘
                               │               │
                        ┌──────▼─────┐   ┌────▼─────┐
                        │  Telegram  │   │  Notion  │
                        │  Delivery  │   │  Storage │
                        └────────────┘   └──────────┘
```

## 📥 Data Sources

```
External Sources                n8n Workflow
─────────────────              ──────────────

🚀 TechCrunch RSS  ────────┐
                           │
📱 The Verge RSS  ─────────┤
                           │
🔥 Hacker News API ────────┼──▶  Normalize  ──▶  Merge  ──▶  AI
                           │                              Summarize
💻 Dev.to API  ────────────┤
                           │
📚 Stack Overflow RSS  ────┘
```

## 🔄 Daily Aggregation Flow

```
Step 1: Collection (Parallel)
┌─────────────────────────────────────────────────────────┐
│  📡 Fetch from 5 sources simultaneously                 │
│  ├─ TechCrunch → Parse RSS (Last 48 hours)             │
│  ├─ The Verge → Parse Atom Feed (Last 48 hours)        │
│  ├─ Hacker News → Get Top 10 IDs → Fetch Items         │
│  ├─ Dev.to → Direct API (Last 24 hours)                │
│  └─ Stack Overflow → Parse RSS (Last 72 hours)         │
│                                                         │
│  NOTE: Different time windows per source due to        │
│        posting frequency variations                    │
└─────────────────────────────────────────────────────────┘
                        ↓
Step 2: Normalization (Per Source)
┌─────────────────────────────────────────────────────────┐
│  🔧 Transform to standard format                        │
│  ├─ Parse XML/JSON structure                           │
│  ├─ Extract: title, link, description, date            │
│  ├─ Add source name and category                       │
│  ├─ Filter by time window                              │
│  └─ Limit results (5 per source, except HN)            │
│                                                         │
│  Standard Format:                                      │
│  {                                                      │
│    title: string,                                       │
│    link: string,                                        │
│    description: string (max 200 chars),                │
│    source: string,                                      │
│    date: ISO string,                                    │
│    category: string,                                    │
│    score?: number (HN only)                            │
│  }                                                      │
└─────────────────────────────────────────────────────────┘
                        ↓
Step 3: Formatting (Per Article)
┌─────────────────────────────────────────────────────────┐
│  🎨 Format for Telegram with inline button             │
│  ├─ Add source emoji (🚀📱🔥💻📚)                       │
│  ├─ Format as MarkdownV2                               │
│  ├─ Escape special characters                          │
│  ├─ Add score for HN articles                          │
│  ├─ Include "Read more" link                           │
│  └─ Attach "💾 Save to Notion" button                  │
│                                                         │
│  Button Data (truncated to 64 chars):                 │
│  callback_data: JSON.stringify({                       │
│    action: 'save',                                     │
│    link: url,                                          │
│    title: title,                                       │
│    source: source                                      │
│  }).substring(0, 64)                                   │
│                                                         │
│  NOTE: Full data extracted from message entities,      │
│        not from truncated callback_data                │
└─────────────────────────────────────────────────────────┘
                        ↓
Step 4: Delivery (Separate Messages)
┌─────────────────────────────────────────────────────────┐
│  📨 Send to Telegram (One message per article)          │
│  ├─ Each article = separate message                    │
│  ├─ MarkdownV2 format with escaped chars               │
│  ├─ Inline keyboard with Save button                   │
│  ├─ Disable web page preview                           │
│  └─ Total: ~15-25 messages per day                     │
│                                                         │
│  Message Format:                                       │
│  "🚀 *TechCrunch*                                      │
│                                                         │
│  *[Escaped Title]*                                     │
│  _[Escaped Description]_                               │
│  ⭐ Score: 123 (HN only)                               │
│  🔗 [Read more](link)                                  │
│                                                         │
│  [💾 Save to Notion]"                                  │
└─────────────────────────────────────────────────────────┘
```

## 💾 Save & Research Flow (Interactive Button)

```
Step 1: Trigger
┌─────────────────────────────────────────────────────────┐
│  � Telegram Button Click                               │
│  User clicks "💾 Save to Notion" button                │
│  Callback query received by webhook                    │
└─────────────────────────────────────────────────────────┘
                        ↓
Step 2: Parse Button Callback
┌─────────────────────────────────────────────────────────┐
│  📋 Extract data from message entities                  │
│  ├─ Parse source from first line (emoji + name)        │
│  ├─ Extract title from bold entities [1]               │
│  ├─ Get link from text_link entity (full URL)          │
│  ├─ Capture chatId and messageId                       │
│  └─ Store callbackQueryId                              │
│                                                         │
│  NOTE: Uses message entities instead of callback_data  │
│        to avoid 64-character truncation                │
└─────────────────────────────────────────────────────────┘
                        ↓
Step 3: Answer Callback Query (IMMEDIATE)
┌─────────────────────────────────────────────────────────┐
│  ⚡ Instant Response to Telegram                        │
│  ├─ Show popup: "✅ Saved! Processing with AI..."      │
│  ├─ Prevent timeout (must respond within 5 seconds)    │
│  └─ User sees immediate feedback                       │
│                                                         │
│  CRITICAL: This executes before heavy processing       │
└─────────────────────────────────────────────────────────┘
                        ↓
        ┌───────────────┴───────────────┐
        │                               │
Step 4a: Pass Original Data      Step 4b: Fetch Article Content
┌──────────────────────┐          ┌──────────────────────┐
│  📦 Preserve Context │          │  🌐 HTTP Request     │
│  Uses $('Parse       │          │  Get full HTML from  │
│  Button Callback')   │          │  article URL         │
│  to pass:            │          └──────────┬───────────┘
│  - chatId            │                     │
│  - title             │                     ↓
│  - source            │          ┌──────────────────────┐
│  - link              │          │  🧠 AI Deep Research │
│  - savedAt           │          │  GPT-4o-mini         │
└──────────┬───────────┘          │  Max: 2000 tokens    │
           │                      │  Temp: 0.7           │
           │                      │                      │
           │                      │  Report Structure:   │
           │                      │  1. Tóm tắt chi tiết │
           │                      │  2. Key Takeaways    │
           │                      │  3. Context          │
           │                      │  4. Pros & Cons      │
           │                      │  5. Related Tech     │
           │                      │  6. Applications     │
           │                      │  7. Tags (5-7)       │
           │                      └──────────┬───────────┘
           └─────────────┬─────────────────┘
                         ↓
Step 5: Merge Content + Research
┌─────────────────────────────────────────────────────────┐
│  🔀 Combine data from both paths                        │
│  ├─ Input 0: AI research result                        │
│  ├─ Input 1: Original parsed data                      │
│  └─ Output: Complete article with research             │
└─────────────────────────────────────────────────────────┘
                        ↓
Step 6: Prepare for Notion
┌─────────────────────────────────────────────────────────┐
│  📝 Format for Notion API                               │
│  ├─ Extract tags from research using regex             │
│  ├─ Split and sanitize tags (remove #, trim spaces)    │
│  ├─ Ensure tags is array (not undefined)               │
│  ├─ Fallback to ['tech', 'research'] if no tags        │
│  ├─ Set status to "Researched"                         │
│  ├─ Truncate research to 2000 chars (Notion limit)     │
│  └─ Preserve chatId for confirmation                   │
└─────────────────────────────────────────────────────────┘
                        ↓
Step 7: Save to Notion
┌─────────────────────────────────────────────────────────┐
│  💾 Create Notion database page                         │
│  Properties:                                           │
│  ├─ Title: "={{ $json.title }}"                        │
│  ├─ URL|url: "={{ $json.link }}"                       │
│  ├─ Source|select: "={{ $json.source }}"               │
│  ├─ Tags|multi_select: "={{ $json.tags }}"             │
│  ├─ Status|select: "Researched"                        │
│  └─ Saved At|date: "={{ $json.savedAt }}"              │
│                                                         │
│  Content:                                              │
│  └─ Research (first 2000 chars)                        │
│                                                         │
│  NOTE: Uses "PropertyName|type" format                 │
└─────────────────────────────────────────────────────────┘
                        ↓
Step 8: Prepare Confirmation
┌─────────────────────────────────────────────────────────┐
│  🔍 Retrieve preserved data                             │
│  ├─ Get Notion page URL from response                  │
│  ├─ Retrieve chatId from $('Prepare for Notion')       │
│  ├─ Get title and source                               │
│  └─ Prepare confirmation message                       │
│                                                         │
│  CRITICAL: ChatId must be retrieved from earlier node  │
│            as it's lost after Save to Notion           │
└─────────────────────────────────────────────────────────┘
                        ↓
Step 9: Send Confirmation
┌─────────────────────────────────────────────────────────┐
│  ✅ Send success message to Telegram                    │
│  Message format:                                       │
│  "✅ *Article Saved to Notion\!*                       │
│                                                         │
│  *Title:* [escaped title]                              │
│  *Source:* [source name]                               │
│  🔗 [View in Notion]([notion_url])"                    │
│                                                         │
│  ├─ Uses MarkdownV2 format                             │
│  ├─ Escapes special chars: _*[]()~`>#+=|{}.!-          │
│  └─ Includes clickable Notion link                     │
└─────────────────────────────────────────────────────────┘
```

## 🔌 Integration Points

```
┌──────────────┐
│   OpenAI     │◄─────── AI Summaries & Research
│   GPT-4o-mini│
└──────────────┘

┌──────────────┐
│   Telegram   │◄─────── Daily Digest & Confirmations
│   Bot API    │
└──────────────┘

┌──────────────┐
│   Notion     │◄─────── Knowledge Base Storage
│   API        │
└──────────────┘

┌──────────────┐
│   Various    │◄─────── Article Sources
│   RSS/APIs   │
└──────────────┘
```

## 📊 Data Transformation

```
Raw RSS/API Response
        ↓
┌──────────────────────────┐
│  <rss><item>            │
│    <title>Article</title>│
│    <link>URL</link>     │
│    <description>...</>   │
│    <pubDate>...</>       │
│  </item></rss>          │
└──────────────────────────┘
        ↓ [Parse XML]
┌──────────────────────────┐
│  {                       │
│    title: "Article",     │
│    link: "URL",          │
│    description: "...",   │
│    date: "ISO"           │
│  }                       │
└──────────────────────────┘
        ↓ [Normalize]
┌──────────────────────────┐
│  {                       │
│    title: "Article",     │
│    link: "URL",          │
│    description: "...",   │
│    source: "TechCrunch", │
│    date: "ISO",          │
│    category: "Tech"      │
│  }                       │
└──────────────────────────┘
        ↓ [AI Summarize]
┌──────────────────────────┐
│  {                       │
│    ...previous fields... │
│    summary: "Vietnamese  │
│              2-3 sentence│
│              summary"    │
│  }                       │
└──────────────────────────┘
        ↓ [Format]
┌──────────────────────────┐
│  "🚀 *TechCrunch*        │
│                          │
│  *Article Title*         │
│  _Summary in Vietnamese_ │
│  🔗 [Read more](URL)"    │
└──────────────────────────┘
```

## 🎯 User Journey

```
Morning Routine
─────────────

1. User wakes up ☕
   │
   ↓
2. Opens Telegram 📱
   │
   ↓
3. Sees Daily Digest 📰
   (15-25 separate messages)
   │
   ├─► Quick scan article → Read description
   │
   ├─► Interesting article → Click "Read more"
   │
   └─► Very valuable article
       │
       ↓
4. Clicks "💾 Save to Notion" button �
   │
   ↓
5. Sees instant popup ⚡
   "✅ Saved! Processing with AI..."
   │
   ├─ Button disappears (removed from message)
   │
   ↓
6. Continues reading other articles 📖
   (AI processes in background)
   │
   ↓
7. Receives confirmation (~30-60 sec later) ✅
   "✅ Article Saved to Notion!
   Title: [Article Title]
   Source: TechCrunch
   🔗 View in Notion"
   │
   ↓
8. Clicks Notion link 📚
   │
   ↓
9. Reads comprehensive AI research 🧠
   - Detailed summary (Vietnamese)
   - Key takeaways
   - Context & background
   - Pros & cons
   - Related technologies
   - Practical applications
   - Auto-generated tags
   │
   ↓
10. Adds personal notes ✍️
    Tags article for later reference
```

## 🔄 State Management

```
Workflow States
───────────────

┌─────────────┐
│   Waiting   │ ◄──────┐
│  (Cron/     │        │
│  Webhook)   │        │
└──────┬──────┘        │
       │               │
       ↓               │
┌─────────────┐        │
│  Executing  │        │
│  (Running)  │        │
└──────┬──────┘        │
       │               │
       ├─ Success ─────┤
       │               │
       └─ Error ───► [Log & Retry]
```

## 💡 Smart Features

```
Duplicate Detection
───────────────────
[Article A] ─┐
[Article B] ─┤─► Compare URLs ─► Keep unique
[Article C] ─┘

Time-based Filtering
────────────────────
Current Time: 9:00 AM Nov 2
Article Published: 8:00 AM Nov 1
Time Diff: 25 hours
Filter: 24 hours
Result: EXCLUDE ❌

Article Published: 10:00 AM Nov 1
Time Diff: 23 hours
Filter: 24 hours
Result: INCLUDE ✅

AI Context Building
───────────────────
Input: Raw article HTML
       ↓
[Extract] → Clean text
       ↓
[Summarize] → Key points
       ↓
[Enrich] → Add context
       ↓
Output: Structured research
```

## 📈 Performance Flow

```
Execution Timeline
──────────────────

00:00 │ Cron triggers
      │
00:05 │ ├─ Fetch sources (parallel)
      │ │  ├─ TechCrunch ✓
      │ │  ├─ The Verge ✓
      │ │  ├─ Hacker News ✓
      │ │  ├─ Dev.to ✓
      │ │  └─ Stack Overflow ✓
      │
00:30 │ ├─ Normalize all
      │
00:45 │ ├─ Merge articles
      │
01:00 │ ├─ AI summarize (parallel)
      │ │  Processing 25 articles...
      │
02:30 │ ├─ Format message
      │
02:45 │ └─ Send to Telegram ✓
      │
03:00 │ Workflow complete ✅
```

---

**Legend:**
- `─▶` : Data flow
- `┌─┐` : Process/Node
- `◄─►` : Two-way connection
- `├─┤` : Branch/Merge
- `✓` : Success
- `✗` : Failed
- `⏱️` : Time-dependent
- `🤖` : AI-powered

**Note:** All diagrams are conceptual representations of the actual n8n workflow implementation.
