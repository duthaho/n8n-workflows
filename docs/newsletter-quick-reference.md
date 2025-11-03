# Daily Newsletter Aggregator - Quick Reference

## 🚀 Quick Start

### 1. Prerequisites Checklist
- [ ] n8n running (`docker-compose up -d`)
- [ ] OpenAI API key configured in n8n
- [ ] Telegram bot token configured
- [ ] Telegram chat ID in `.env`
- [ ] Notion integration created
- [ ] Notion database created with correct properties
- [ ] Notion database ID in `.env`

### 2. Import Workflow
```bash
# In n8n UI:
# Click + → Import from File → Select daily-newsletter-aggregator.json
```

### 3. Configure Notion Node
- Open workflow
- Click "Save to Notion" node
- Select Notion credentials
- Select database
- Click Save

### 4. Activate
- Toggle "Active" switch in workflow editor

## 📋 Daily Usage

### Automatic Digest
**When:** Every day at 9:00 AM (Asia/Ho_Chi_Minh)  
**What:** Receive 15-25 separate article messages in Telegram  
**Format:** Each message includes:
- Source emoji + name
- Article title
- Brief description
- Read more link
- "💾 Save to Notion" button

**Action:** 
1. Scroll through articles
2. Click "Read more" for interesting ones
3. Click "💾 Save to Notion" for valuable articles

### Save Article with AI Research (Interactive)
**Method:** Click the "💾 Save to Notion" button on any article

**What Happens:**
1. ⚡ Instant popup: "✅ Saved! Processing with AI..."
2. 🔘 Button disappears from message
3. 🤖 AI researches article in background (30-60 seconds)
4. 💾 Saves to Notion with comprehensive analysis
5. ✅ Confirmation message with Notion link

**No Command-Line Required!** Everything is done through Telegram buttons.

## 🔧 Common Tasks

### Change Digest Time
1. Open workflow
2. Click "Daily Trigger" node
3. Change `hour` value (0-23)
4. Save

### Add New Source
1. Add HTTP Request node
2. Enter RSS/API URL
3. Add Normalize function node
4. Connect to "Merge All Articles"
5. Update emoji mapping in "Format Telegram Message"

### Customize AI Summary
1. Click "AI Summarize Articles" node
2. Edit system prompt or user message
3. Adjust `maxTokens` (50-200)
4. Adjust `temperature` (0.0-1.0)
5. Save

### Customize Deep Research
1. Click "AI Deep Research" node
2. Modify research structure in prompt
3. Adjust `maxTokens` (1000-4000)
4. Save

## 📊 Current Sources

| Source | Type | Articles | Time Window |
|--------|------|----------|-------------|
| 🚀 TechCrunch | RSS | Top 5 | 48 hours |
| 📱 The Verge | Atom Feed | Top 5 | 48 hours |
| 🔥 Hacker News | API | Top 10 | 24 hours |
| 💻 Dev.to | API | Top 5 | 24 hours |
| 📚 Stack Overflow | RSS | Top 3 | 72 hours |

**Note:** Stack Overflow has longer window (72h) due to less frequent posts

## 🎨 Notion Database Structure

```
┌─────────────────────────────────────────────────┐
│ Title (title) - Auto-filled from article       │
├─────────────────────────────────────────────────┤
│ URL (url) - Link to original article           │
├─────────────────────────────────────────────────┤
│ Source (select) - TechCrunch, The Verge, etc.  │
│ Options: TechCrunch, The Verge, Hacker News,   │
│          Dev.to, Stack Overflow                │
├─────────────────────────────────────────────────┤
│ Tags (multi-select) - AI-extracted keywords    │
│ Auto-generated: 5-7 tags per article           │
│ Examples: AI, Tech, Programming, Framework      │
├─────────────────────────────────────────────────┤
│ Status (select) - Automatically set            │
│ Default: "Researched"                          │
│ Options: Researched, Reading, Read, Archived   │
├─────────────────────────────────────────────────┤
│ Saved At (date) - Timestamp when saved         │
└─────────────────────────────────────────────────┘

Content Block: First 2000 chars of AI research
```

**Important:** Property names in n8n use format: `"PropertyName|type"`
- Example: `"URL|url"`, `"Source|select"`, `"Tags|multi_select"`

## 🐛 Quick Troubleshooting

### No Digest Received
```bash
# Check n8n logs
docker-compose logs -f n8n

# Check workflow execution
# In n8n UI: Go to Executions tab
```

### Save Button Not Working
1. Check "Telegram Button Trigger" is active
2. Verify Telegram bot has webhook set up
3. Check n8n is accessible from internet (if using ngrok)
4. Review execution logs for errors

### "Query is too old" Error
- **Cause:** Telegram callback timeout (5 seconds)
- **Solution:** Workflow already optimized for sequential execution
- **Check:** Verify "Answer Callback Query" executes immediately after "Parse Button Callback"

### Save Article Fails
1. Check article URL is valid and accessible
2. Verify Notion credentials are active
3. Check OpenAI API quota
4. Verify database properties match workflow configuration

### Notion Save Error
1. Verify database ID is correct
2. Check integration has access to database
3. Verify property names match exactly
4. Check credential is valid

## 📈 Monitoring

### Check Last Execution
```bash
# View in n8n UI
Executions → Click latest run → View details
```

### View Workflow Status
```bash
# In n8n UI
Workflows → Check "Active" column
```

### Check API Usage
- OpenAI: https://platform.openai.com/usage
- Notion: Check integration activity in Notion

## 🎯 Workflow IDs Reference

| Component | ID/Name |
|-----------|---------|
| Workflow Name | Daily Newsletter Aggregator |
| Cron Trigger | Daily at 9:00 AM |
| Webhook Path | `/webhook/newsletter-save` |
| Telegram Credential | `duthaho-bot` |
| OpenAI Credential | `OpenAi account` |
| Notion Credential | `Notion API` |

## 💡 Pro Tips

### Better Summaries
- Use more specific system prompts
- Increase temperature for more creative summaries
- Add examples in the prompt

### Faster Research
- Reduce maxTokens in research node
- Cache common researched topics
- Use GPT-3.5-turbo instead of GPT-4 for speed

### Better Organization
- Create views in Notion by Tags
- Add date filters for recent articles
- Create a "Favorites" status option

### Save from Mobile
- **Easy!** Just click the button in Telegram on your phone
- No scripts or automation needed
- Works on iOS, Android, and all Telegram clients

## 🔗 Important URLs

| Service | URL |
|---------|-----|
| n8n Web UI | http://localhost:5678 |
| Webhook Endpoint | http://localhost:5678/webhook/newsletter-save |
| Notion Integrations | https://www.notion.so/my-integrations |
| OpenAI Platform | https://platform.openai.com |
| Telegram Bot Father | https://t.me/BotFather |

## 📚 Documentation

- Full Setup: [`docs/newsletter-aggregator-setup.md`](./newsletter-aggregator-setup.md)
- Main README: [`../README.md`](../README.md)
- Workflow File: [`workflows/daily-newsletter-aggregator.json`](../workflows/daily-newsletter-aggregator.json)

## 🎓 Learning Resources

### Understanding the Flow

**Daily Aggregation:**
1. **Collection Phase**: Fetch from 5 sources in parallel
2. **Normalization Phase**: Parse and standardize data format
3. **Formatting Phase**: Create Telegram messages with buttons
4. **Delivery Phase**: Send individual messages to Telegram

**On-Demand Research (Button Click):**
1. **Trigger Phase**: Telegram button callback received
2. **Parse Phase**: Extract data from message entities
3. **Response Phase**: Immediate callback answer (prevents timeout)
4. **Fetch Phase**: Get full article content
5. **Research Phase**: AI deep analysis (GPT-4o-mini)
6. **Merge Phase**: Combine research with original data
7. **Prepare Phase**: Format for Notion, extract/sanitize tags
8. **Storage Phase**: Save to Notion database
9. **Confirmation Phase**: Send success message with link

### Key Concepts
- **Message Entities**: Telegram's rich text structure (bold, links)
- **Callback Queries**: Button click responses (5-second timeout)
- **Sequential Flow**: Ensures callback answered before heavy processing
- **Node References**: `$('NodeName').first().json` to access earlier data
- **RSS/Atom Parsing**: Convert XML feeds to JSON
- **API Integration**: Direct REST API calls
- **Merge Nodes**: Combine parallel data streams
- **Function Nodes**: Custom JavaScript transformations
- **MarkdownV2**: Telegram's markdown format with escaping

### Technical Highlights
- **No AI in Daily Flow**: Saves resources, faster delivery
- **Inline Buttons**: Interactive UI in Telegram messages
- **Data Preservation**: ChatId/title passed through workflow
- **Error Handling**: Tags validation, content truncation, fallbacks
- **Property Format**: `"PropertyName|type"` for Notion API

---

**Last Updated:** November 3, 2025  
**Version:** 2.0  
**Status:** Production Ready ✅
