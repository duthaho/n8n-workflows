# Daily Newsletter Aggregator - Setup Guide

## 📋 Overview

This workflow provides a comprehensive newsletter aggregation system with:
- Daily automated digest from 5+ tech sources
- AI-powered summaries in Vietnamese
- On-demand deep research capability
- Notion knowledge base integration

## 🎯 Features

### 1. Daily Newsletter Digest (Automated)
- Runs every day at 9:00 AM
- Fetches latest articles (last 24 hours) from:
  - 🚀 TechCrunch
  - 📱 The Verge
  - 🔥 Hacker News
  - 💻 Dev.to
  - 📚 Stack Overflow Blog
- Generates AI summaries for each article
- Sends organized digest to Telegram

### 2. Deep Research & Save (On-Demand)
- Trigger via webhook for any article URL
- AI performs comprehensive research:
  - Detailed summary (Vietnamese)
  - Key takeaways
  - Context & background analysis
  - Pros & cons evaluation
  - Related technologies
  - Practical applications
  - Auto-generated tags
- Saves to Notion database
- Confirms via Telegram

## 🔧 Setup Instructions

### Step 1: Notion Setup

1. **Create Notion Database**

   Go to Notion and create a new database (table view) with these properties:

   | Property Name | Type | Options |
   |--------------|------|---------|
   | Title | Title | - |
   | URL | URL | - |
   | Tags | Multi-select | Create tags like: AI, Tech, Programming, etc. |
   | Status | Select | Options: "Saved", "Researched", "Reading", "Archived" |
   | Saved At | Date | Include time |

2. **Create Notion Integration**

   - Visit: https://www.notion.so/my-integrations
   - Click "New integration"
   - Name it: "n8n Newsletter Aggregator"
   - Select your workspace
   - Click "Submit"
   - **Copy the "Internal Integration Token"** (starts with `secret_`)

3. **Share Database with Integration**

   - Open your Notion database
   - Click "..." (three dots) in top right
   - Click "Add connections"
   - Select your integration "n8n Newsletter Aggregator"

4. **Get Database ID**

   - Open your database in Notion
   - Copy the URL, it looks like:
     ```
     https://www.notion.so/workspace/abc123def456?v=...
     ```
   - The database ID is the part after your workspace name and before the `?`:
     ```
     abc123def456
     ```

### Step 2: Environment Variables

Add to your `.env` file:

```env
# Notion Configuration
NOTION_DATABASE_ID=your_database_id_here

# Webhook URL (if using ngrok or custom domain)
N8N_WEBHOOK_URL=https://your-domain.com

# Existing variables
TELEGRAM_CHAT_ID=your_chat_id
```

### Step 3: Import Workflow

1. Open n8n web interface: `http://localhost:5678`
2. Click "+" → "Import from File"
3. Select `workflows/daily-newsletter-aggregator.json`
4. Click "Import"

### Step 4: Configure Credentials

#### OpenAI Credentials
1. In n8n, go to **Settings** → **Credentials**
2. Click "Add Credential" → Search "OpenAI"
3. Enter your OpenAI API key
4. Click "Save"

#### Telegram Credentials
1. Already configured from previous workflows
2. Make sure `TELEGRAM_CHAT_ID` is set in `.env`

#### Notion Credentials
1. In n8n, go to **Settings** → **Credentials**
2. Click "Add Credential" → Search "Notion API"
3. Enter your Notion Integration Token (from Step 1.2)
4. Click "Save"

### Step 5: Update Workflow Nodes

1. Open the imported workflow
2. Click on **"Save to Notion"** node
3. Select your Notion credentials
4. Select your database from the dropdown
5. Map the properties (should auto-detect):
   - Title → Title
   - URL → URL
   - Tags → Tags (multi-select)
   - Status → Status (select)
   - Saved At → Saved At (date)
6. Click "Execute Node" to test
7. Click "Save" workflow

### Step 6: Test the Workflow

#### Test Daily Digest (Manual Trigger)
1. Open the workflow
2. Click on **"Daily Trigger"** node
3. Click "Execute Node"
4. Wait for execution to complete
5. Check your Telegram for the digest message

#### Test Save & Research Feature
Using PowerShell (Windows):
```powershell
.\scripts\save-article.ps1 "https://techcrunch.com/some-article"
```

Using Bash (Linux/Mac):
```bash
chmod +x scripts/save-article.sh
./scripts/save-article.sh "https://techcrunch.com/some-article"
```

Using cURL directly:
```bash
curl -X POST http://localhost:5678/webhook/newsletter-save \
  -H "Content-Type: application/json" \
  -d '{"link":"https://techcrunch.com/some-article"}'
```

### Step 7: Activate Workflow

1. In n8n workflow editor
2. Toggle the switch at top to **"Active"**
3. The workflow will now run daily at 9:00 AM

## 📱 Usage Examples

### Daily Digest

You'll receive a Telegram message like:

```
📰 Daily Tech Newsletter Digest
Thứ Bảy, 2 tháng 11, 2025

🚀 TechCrunch

AI Startup Raises $100M Series B
Một startup AI mới vừa gọi vốn thành công 100 triệu USD để phát triển nền tảng AI cho doanh nghiệp. 
Đây là vòng gọi vốn lớn nhất trong lĩnh vực AI năm nay.
🔗 Đọc thêm

📱 The Verge

New iPhone Features Leaked
...

🔥 Hacker News

...

Powered by AI • 15 articles
```

### Save Article to Notion

1. Find an interesting article from the daily digest
2. Copy the article URL
3. Run the save script:
   ```powershell
   .\scripts\save-article.ps1 "https://article-url.com"
   ```
4. Wait 10-30 seconds for AI research
5. Check Notion for the comprehensive analysis
6. Receive Telegram confirmation

### Notion Entry Example

**Title:** AI Startup Raises $100M Series B  
**URL:** https://techcrunch.com/...  
**Tags:** AI, Funding, Startup, Enterprise  
**Status:** Researched  
**Saved At:** Nov 2, 2025 10:30 AM  

**Content:**
```markdown
## Tóm tắt chi tiết

Startup AI này đã phát triển một nền tảng cho phép doanh nghiệp tích hợp 
AI vào quy trình làm việc hiện tại một cách dễ dàng. Với 100 triệu USD 
vừa gọi được, họ có kế hoạch mở rộng đội ngũ và phát triển thêm nhiều 
tính năng mới...

## Key Takeaways

• Vòng gọi vốn Series B trị giá $100M
• Định hướng vào thị trường enterprise AI
• Đã có 500+ khách hàng doanh nghiệp
• Kế hoạch mở rộng sang châu Á trong Q2 2025

## Context & Background

Thị trường AI enterprise đang bùng nổ với tốc độ tăng trưởng 40% mỗi năm...

## Pros & Cons

**Pros:**
• Easy integration with existing systems
• Strong customer base
• Proven product-market fit

**Cons:**
• High competition in AI space
• Dependency on cloud infrastructure

## Related Technologies/Concepts

• Large Language Models (LLMs)
• API-first architecture
• Cloud computing
• MLOps

## Practical Applications

1. Customer service automation
2. Document processing
3. Data analysis and insights
4. Workflow optimization
```

## 🔄 Customization

### Add More Sources

To add additional newsletter sources:

1. Open the workflow in n8n
2. Add a new **HTTP Request** node
3. Configure the RSS/API endpoint
4. Add a corresponding **Normalize** function node
5. Connect to the **"Merge All Articles"** node
6. Update the `sourceEmoji` object in **"Format Telegram Message"** node

### Adjust Timing

To change the daily trigger time:

1. Click on **"Daily Trigger"** node
2. Modify the `hour` parameter (24-hour format)
3. Save the workflow

### Customize AI Prompts

**For Summaries:**
- Edit the **"AI Summarize Articles"** node
- Modify the system message or user prompt
- Adjust `maxTokens` (50-200) for length
- Adjust `temperature` (0.0-1.0) for creativity

**For Deep Research:**
- Edit the **"AI Deep Research"** node
- Customize the research structure
- Adjust `maxTokens` (1000-4000) for depth

### Change Article Time Window

By default, only articles from the last 24 hours are included:

1. Edit each **Normalize** function node
2. Find the line: `if (hoursDiff <= 24)`
3. Change `24` to desired hours (e.g., `48` for 2 days)

## 🐛 Troubleshooting

### No Articles Appear

**Problem:** Empty digest or no articles  
**Solutions:**
- Check if sources are accessible (test with browser)
- Increase time window from 24 to 48 hours
- Check n8n execution logs for errors

### Notion Save Fails

**Problem:** Error when saving to Notion  
**Solutions:**
- Verify Notion credentials are correct
- Check database ID is correct
- Ensure database is shared with integration
- Verify property names match exactly (case-sensitive)

### AI Research Takes Too Long

**Problem:** Webhook times out  
**Solutions:**
- Increase n8n timeout settings
- Use a smaller `maxTokens` value
- Split research into multiple smaller AI calls

### Telegram Not Receiving Messages

**Problem:** No Telegram notifications  
**Solutions:**
- Verify `TELEGRAM_CHAT_ID` is correct
- Check bot token is valid
- Test with a simple message first
- Ensure bot is not blocked

## 📊 Monitoring

### Check Execution History

1. In n8n, go to **Executions**
2. View all workflow runs
3. Click on any execution to see details
4. Check for errors in failed executions

### View Logs

```bash
# View n8n logs
docker-compose logs -f n8n

# View last 100 lines
docker-compose logs --tail=100 n8n
```

## 🎯 Next Steps

### Phase 1: Basic Usage (Current)
- ✅ Daily digest delivery
- ✅ Manual article saving
- ✅ Notion knowledge base

### Phase 2: Enhancements (Future)
- 🔲 Add AI sources (The Batch, Import AI, TLDR AI)
- 🔲 Telegram inline buttons for one-click save
- 🔲 Smart categorization and tagging
- 🔲 Weekly summary reports
- 🔲 Search functionality in Notion
- 🔲 Export to PDF/Markdown

### Phase 3: Advanced (Future)
- 🔲 Multi-language support
- 🔲 Custom filtering by keywords
- 🔲 AI-powered article recommendations
- 🔲 Integration with other tools (Slack, Discord)
- 🔲 Analytics dashboard

## 🤝 Contributing

Ideas for improvement:
- Add more newsletter sources
- Improve AI prompts
- Create browser extension for quick saves
- Build Telegram bot commands
- Add voice summary feature

## 📚 Resources

- [n8n Documentation](https://docs.n8n.io/)
- [Notion API Documentation](https://developers.notion.com/)
- [OpenAI API Documentation](https://platform.openai.com/docs)
- [Telegram Bot API](https://core.telegram.org/bots/api)

---

**Need Help?** Check the main README or open an issue!
