# Rainmaker System Template
## Reusable Bot Architecture for LibreChat Projects

*Created: December 25, 2025*
*Based on: Rainmaker (Pipeline Pro) System*

---

## 🎯 System Overview

The Rainmaker system is a client onboarding and intelligence generation platform that:
1. Collects client information via chat interface (Open WebUI → LibreChat for new bots)
2. Stores and manages data in Airtable
3. Performs AI-powered analysis (website scraping, market research)
4. Generates reports and marketing campaigns
5. Delivers outputs via email and web reports

---

## 🔧 Technology Stack

| Component | Rainmaker Uses | LibreChat Adaptation |
|-----------|----------------|---------------------|
| Chat Interface | Open WebUI | **LibreChat** |
| Workflow Engine | n8n | n8n (same) |
| Database | Airtable | Airtable (same) |
| AI Models | Claude (Anthropic) | Claude (same) |
| Real-time Research | Perplexity API | Perplexity (same) |
| Email | Gmail (n8n node) | Gmail (same) |
| Web Scraping | HTTP Request + Jina | Same pattern |

---

## 📊 n8n Workflow Inventory

### Core Workflows (8 total)

#### 1. Smart Save (`I4GK3VwiR4sWnGBy`)
**Purpose:** Save/update client data from chat to Airtable
**Webhook:** `/webhook/rainmaker-smart-save`
**Pattern:** Upsert (create or update based on email)

| Node Type | Function |
|-----------|----------|
| `webhook` | Receives data from chat interface |
| `airtable` | Check for existing record by email |
| `if` | Branch: exists vs new |
| `set` | Transform field names |
| `airtable` | Create OR Update record |

**Key Pattern:** Email-based record lookup + upsert logic

---

#### 2. User Lookup (`Tut4i6wbhYgoetyE`)
**Purpose:** Check if user exists, return their data
**Webhook:** `/webhook/rainmaker-user-lookup`

| Node Type | Function |
|-----------|----------|
| `webhook` | Receives email lookup request |
| `airtable` | Search by email |
| `if` | Found vs not found |
| `set` | Format response (found or not found) |
| `respondToWebhook` | Return JSON response |

**Key Pattern:** Simple lookup with formatted response

---

#### 3. Website/LinkedIn Analysis (`ZextpRZU97VcBk6V`)
**Purpose:** Scrape and analyze website + LinkedIn, generate initial intelligence
**Webhook:** `/webhook/rainmaker-website-analysis`
**Most Complex:** 33 nodes

| Node Type | Function |
|-----------|----------|
| `webhook` | Trigger with email |
| `code` | Validate URL input |
| `airtable` | Early lookup to get stored URLs |
| `code` | Merge URL sources (input vs stored) |
| `if` | Check if URLs exist |
| `httpRequest` | Fetch website content (Jina reader) |
| `httpRequest` | Fetch LinkedIn content |
| `code` | Check fetch results |
| `code` | Merge content sources |
| `switch` | Route based on success/failure |
| `chainLlm` | Claude: Analyze website content |
| `lmChatAnthropic` | Claude Sonnet model |
| `code` | Parse Claude response |
| `airtable` | Lookup client for update |
| `httpRequest` | Update Airtable with analysis |
| `code` | Prepare research context |
| `httpRequest` | Perplexity: Research challenges |
| `httpRequest` | Perplexity: Research trends |
| `merge` | Wait for both Perplexity calls |
| `chainLlm` | Claude: Synthesize Top 3s |
| `code` | Parse MI response |
| `httpRequest` | Update MI fields in Airtable |
| `code` | Format success response |
| `respondToWebhook` | Return success/failure |
| `airtable` (x4) | Log API usage costs |

**Key Patterns:**
- Parallel HTTP requests (website + LinkedIn)
- Parallel AI calls (Perplexity challenges + trends)
- Merge nodes to wait for parallel operations
- LangChain integration for Claude
- API cost logging

---

#### 4. Market Intelligence Generator (`fyOE3SNz3fKmEydJ`)
**Purpose:** Generate market intelligence based on collected data
**Webhook:** `/webhook/rainmaker-market-intelligence`

| Node Type | Function |
|-----------|----------|
| `webhook` | Trigger with email |
| `if` | Validate email exists |
| `airtable` | Lookup client |
| `if` | Check required data exists |
| `code` | Prepare AI context |
| `perplexity` | Research challenges (native node) |
| `perplexity` | Research trends (native node) |
| `merge` | Wait for both Perplexity |
| `code` | Combine Perplexity with context |
| `chainLlm` | Generate market intelligence |
| `lmChatAnthropic` | Claude Sonnet model |
| `code` | Parse Claude response |
| `if` | Check parse success |
| `code` | Prep Airtable update |
| `httpRequest` | Update Airtable |
| `code` | Format success response |
| `respondToWebhook` | Return result |

**Key Patterns:**
- Perplexity native node (vs HTTP request)
- Citation extraction and storage
- JSON.stringify for array fields

---

#### 5. Campaign Message Generator (`7SqNq9Eqe86MjtJJ`)
**Purpose:** Generate LinkedIn sequences, email campaigns, engagement questions
**Webhook:** `/webhook/rainmaker-campaign-generator`

| Node Type | Function |
|-----------|----------|
| `webhook` | Trigger with email |
| `if` | Validate email |
| `airtable` | Lookup client |
| `if` | Check MI data exists |
| `code` | Prepare campaign context |
| `chainLlm` | Generate campaign content |
| `lmChatAnthropic` | Claude Haiku (faster/cheaper) |
| `code` | Parse campaign response |
| `if` | Check parse success |
| `set` | Capture campaign data |
| `httpRequest` | Update Airtable |
| `code` | Format success |
| `respondToWebhook` | Return result |

**Key Pattern:** Uses Claude Haiku for cost efficiency

---

#### 6. Client Report Generator (`XaRnOiLf8uLrEMA8`)
**Purpose:** Generate HTML report from all collected data
**Webhook:** `/webhook/rainmaker-client-report`

| Node Type | Function |
|-----------|----------|
| `webhook` | Trigger with email |
| `if` | Validate input |
| `airtable` | Fetch client by email |
| `code` | Generate Report HTML (main logic) |
| `respondToWebhook` | Return HTML |
| `gmail` | Send report email |

**Key Patterns:**
- Full HTML generation in Code node
- Citation linking (JSON array → clickable links)
- Responsive CSS styling
- Parallel: respond AND send email

---

#### 7. Client Summary Email (`8Hn0K2zpEbxtUlIj`)
**Purpose:** Send summary email to client
**Webhook:** `/webhook/rainmaker-client-summary`

| Node Type | Function |
|-----------|----------|
| `webhook` | Trigger with email |
| `if` | Validate input |
| `airtable` | Fetch client |
| `code` | Generate summary HTML |
| `gmail` | Send email |
| `respondToWebhook` | Confirm success |

---

#### 8. View Report (`DlachzqqtZQuZWmY`)
**Purpose:** GET endpoint to view report in browser
**Webhook:** `/webhook/rainmaker-view-report` (GET)

| Node Type | Function |
|-----------|----------|
| `webhook` | GET request with record_id |
| `if` | Validate record ID |
| `airtable` | Fetch client |
| `code` | Generate report HTML |
| `respondToWebhook` | Return HTML page |

**Key Pattern:** GET webhook for browser viewing

---

## 📁 Node Types Reference

### Triggers
| Node | Purpose | Config Notes |
|------|---------|--------------|
| `n8n-nodes-base.webhook` | HTTP trigger | POST or GET, path, response mode |

### Data Operations
| Node | Purpose | Config Notes |
|------|---------|--------------|
| `n8n-nodes-base.airtable` | CRUD operations | Base ID, Table ID, operation type |
| `n8n-nodes-base.httpRequest` | API calls | For Airtable PATCH, Perplexity, Jina |
| `n8n-nodes-base.set` | Transform data | Map fields, set values |
| `n8n-nodes-base.code` | JavaScript logic | Parsing, formatting, validation |
| `n8n-nodes-base.merge` | Combine branches | Wait for parallel operations |

### Control Flow
| Node | Purpose | Config Notes |
|------|---------|--------------|
| `n8n-nodes-base.if` | Conditional branch | Check values, route data |
| `n8n-nodes-base.switch` | Multi-way branch | Multiple output paths |

### AI/LLM
| Node | Purpose | Config Notes |
|------|---------|--------------|
| `@n8n/n8n-nodes-langchain.chainLlm` | LLM chain | Prompt template + model |
| `@n8n/n8n-nodes-langchain.lmChatAnthropic` | Claude models | Sonnet or Haiku |
| `n8n-nodes-base.perplexity` | Perplexity API | Real-time web research |

### Output
| Node | Purpose | Config Notes |
|------|---------|--------------|
| `n8n-nodes-base.respondToWebhook` | Return response | JSON or HTML |
| `n8n-nodes-base.gmail` | Send email | OAuth credentials |

---

## 🗄️ Airtable Structure

### Base: Rainmaker Client Onboarding (`appFO6M2NzdZA9raZ`)

### Table: Client Onboarding Data (`tbl9HhYDp9UTHaQMQ`)

#### Contact Information Fields
| Field Name | Type | Description |
|------------|------|-------------|
| `Full_Name` | singleLineText | Primary field |
| `First_Name` | singleLineText | First name |
| `Last_Name` | singleLineText | Last name |
| `Email` | email | Required - used as lookup key |
| `Phone` | phoneNumber | Contact phone |
| `Organization` | singleLineText | Company name |
| `Website` | url | Company website |
| `LinkedIn_Profile` | url | LinkedIn URL |

#### Address Fields
| Field Name | Type |
|------------|------|
| `Street_Address` | singleLineText |
| `City` | singleLineText |
| `State` | singleLineText |
| `Country` | singleLineText |
| `Postal_Code` | singleLineText |
| `Full_Address` | formula (concatenated) |

#### Business Context Fields
| Field Name | Type | Description |
|------------|------|-------------|
| `Current_Marketing_Strategies` | multipleSelects | Marketing tactics used |
| `Geographical_Locations` | multilineText | Target geography |
| `Target_Company_Sizes` | multipleSelects | Company size ranges |
| `Job_Titles_Targeting` | multilineText | Target job titles |
| `Specific_Target_Niche_Market` | multilineText | Niche focus |
| `Vertical_1` | singleLineText | Primary vertical |
| `Vertical_2` | singleLineText | Secondary vertical |
| `Ideal_Prospect_Description` | multilineText | ICP description |
| `Value_Proposition` | multilineText | Value prop |
| `Pain_Points_Business_Solves` | multilineText | Pain points addressed |
| `Prospects_Current_State` | multilineText | Before state |
| `Why_Solution_Better` | multilineText | Competitive advantage |
| `Business_Impact` | multilineText | Impact description |
| `Average_Sales_Cycle` | singleLineText | Sales cycle length |
| `Average_Deal_Size` | currency | Deal value |
| `Irresistible_Offer` | multilineText | Lead magnet |
| `Preferred_CTA` | multilineText | Call to action |
| `Calendar_Link` | url | Scheduling link |

#### Website Analysis Fields (AI-Generated)
| Field Name | Type | Description |
|------------|------|-------------|
| `Website_Business_Summary` | multilineText | 2-3 bullet summary |
| `Website_USP_Extracted` | multilineText | Unique selling proposition |
| `Website_Problems_Solved` | multilineText | Problems identified |
| `Website_Competitive_Advantages` | multilineText | Competitive advantages |
| `Website_Analysis_Date` | dateTime | When analyzed |

#### Market Intelligence Fields (AI-Generated)
| Field Name | Type | Description |
|------------|------|-------------|
| `MI_Top_Challenges` | multilineText | Top prospect challenges |
| `MI_Industry_Insights` | multilineText | Industry-specific insights |
| `MI_Persona_Language` | multilineText | Persona-tailored language |
| `MI_Positioning_Recs` | multilineText | Positioning recommendations |
| `MI_Generated_Date` | dateTime | When generated |

#### Perplexity Research Fields
| Field Name | Type | Description |
|------------|------|-------------|
| `Perplexity_Top_Challenges` | multilineText | Real-time challenges |
| `Perplexity_Top_Trends` | multilineText | Real-time trends |
| `Perplexity_Challenges_Citations` | multilineText | JSON array of URLs |
| `Perplexity_Trends_Citations` | multilineText | JSON array of URLs |

#### Campaign Content Fields (AI-Generated)
| Field Name | Type | Description |
|------------|------|-------------|
| `Campaign_LinkedIn_Sequence` | multilineText | 5 LinkedIn messages |
| `Campaign_Email_Sequence` | multilineText | 5 Email sequences |
| `Campaign_Engagement_Questions` | multilineText | Engagement questions |
| `Campaign_Zingers_Used` | multilineText | Conversion phrases |
| `Campaign_Generated_Date` | dateTime | When generated |
| `ROI_Messaging_Framework` | multilineText | ROI messaging |

#### Status & Admin Fields
| Field Name | Type | Description |
|------------|------|-------------|
| `Status` | singleSelect | Overall status |
| `Client_Status` | singleSelect | Detailed phase |
| `Submission_Date` | dateTime | When submitted |
| `Source` | singleLineText | Submission source |
| `Priority` | singleSelect | High/Medium/Low |
| `Assigned_To` | singleLineText | Team member |
| `Follow_Up_Date` | date | Next follow-up |
| `Internal_Notes` | multilineText | Team notes |

#### Report Fields
| Field Name | Type | Description |
|------------|------|-------------|
| `Report_URL` | url | Link to report |
| `Report_Generated_Date` | dateTime | When generated |
| `Report_Status` | singleSelect | Generation status |
| `Report_Version` | number | Version number |

#### System Fields
| Field Name | Type |
|------------|------|
| `Record_ID` | autoNumber |
| `Created_Time` | createdTime |
| `Last_Modified` | lastModifiedTime |

---

### Table: API_Usage_Log (`tblp3rSUnlKXGPQIR`)

| Field Name | Type | Description |
|------------|------|-------------|
| `Name` | singleLineText | Log entry name |
| `Timestamp` | dateTime | When API called |
| `API_Provider` | singleSelect | Claude/Perplexity type |
| `Workflow_Name` | singleLineText | Which workflow |
| `Client_Email` | singleLineText | Which client |
| `Tokens_Input` | number | Input tokens |
| `Tokens_Output` | number | Output tokens |
| `Tokens_Total` | formula | Calculated total |
| `Estimated_Cost` | currency | Cost estimate |
| `Execution_ID` | singleLineText | n8n execution ID |
| `Success` | checkbox | Whether successful |
| `Error_Message` | multilineText | Error details |

---

## 🔌 Integration Patterns

### Pattern 1: Chat Interface → n8n Webhook
```
Chat sends HTTP POST to n8n webhook with:
{
  "email": "client@example.com",
  "field_name": "value",
  ...
}
```
**LibreChat adaptation:** Configure LibreChat actions/tools to call n8n webhooks

### Pattern 2: Airtable Lookup by Email
```javascript
// n8n Airtable node config
Operation: Search
Base: appFO6M2NzdZA9raZ
Table: tbl9HhYDp9UTHaQMQ
Filter: {Email} = "{{ $json.email }}"
```

### Pattern 3: Airtable Update via HTTP Request
```javascript
// When native Airtable node doesn't work well
// Use HTTP Request node with:
Method: PATCH
URL: https://api.airtable.com/v0/{{baseId}}/{{tableId}}/{{recordId}}
Headers: 
  Authorization: Bearer {{airtableApiKey}}
  Content-Type: application/json
Body: {
  "fields": {
    "Field_Name": "value"
  }
}
```

### Pattern 4: Perplexity API Call
```javascript
// Native Perplexity node OR HTTP Request
// Returns: { content: "...", citations: ["url1", "url2"] }
// Store citations: JSON.stringify(citations)
```

### Pattern 5: Citation Linking in HTML
```javascript
const formatTextWithCitations = (text, citationsJson) => {
  let formatted = text.replace(/\n/g, '<br>');
  let citations = JSON.parse(citationsJson || '[]');
  
  // Replace [1], [2], etc. with clickable links
  for (let i = citations.length; i >= 1; i--) {
    const url = citations[i - 1];
    if (url) {
      const regex = new RegExp('\\[' + i + '\\]', 'g');
      formatted = formatted.replace(regex, 
        '<a href="' + url + '" target="_blank">[' + i + ']</a>');
    }
  }
  return formatted;
};
```

### Pattern 6: Parallel API Calls with Merge
```
┌─→ Perplexity Challenges ─┐
│                          ├─→ Merge → Continue
└─→ Perplexity Trends ─────┘
```
Use Merge node to wait for both parallel operations

### Pattern 7: HTML Report Generation
```javascript
// Code node generates full HTML document
const html = `
<!DOCTYPE html>
<html>
<head>
  <style>
    /* Responsive CSS */
  </style>
</head>
<body>
  <div class="container">
    ${generateSections(data)}
  </div>
</body>
</html>
`;
return { html };
```

---

## 🔄 LibreChat Adaptation Guide

### Key Differences from Open WebUI

| Aspect | Open WebUI | LibreChat |
|--------|------------|-----------|
| Tool calling | Functions | Actions/Plugins |
| Webhook config | Model settings | Agent config |
| Memory | Built-in | Separate config |
| UI customization | Limited | Theme-based |

### What Stays the Same
- All n8n workflows (just update webhook source expectations if needed)
- Airtable structure (100% reusable)
- AI prompts and logic
- Report generation code
- Email templates

### What Needs Adaptation
1. **Webhook triggers** - May need to adjust expected payload format
2. **User identification** - How email/user ID is passed from chat
3. **Response formatting** - How results display in chat

---

## 📋 New Bot Checklist

### 1. Airtable Setup
- [ ] Create new base OR duplicate Rainmaker structure
- [ ] Adjust fields for new use case
- [ ] Create API key with correct permissions

### 2. n8n Workflow Setup
- [ ] Duplicate core workflows OR create new ones
- [ ] Update Airtable base/table IDs
- [ ] Update webhook paths (unique per bot)
- [ ] Configure credentials (Airtable, Claude, Perplexity, Gmail)

### 3. LibreChat Configuration
- [ ] Create new agent/assistant
- [ ] Configure tools/actions to call n8n webhooks
- [ ] Set up system prompts for data collection
- [ ] Test end-to-end flow

### 4. Testing Flow
- [ ] Chat collects data → n8n receives
- [ ] n8n stores in Airtable → verify data
- [ ] AI analysis runs → check outputs
- [ ] Reports generate → verify formatting
- [ ] Emails send → check delivery

---

## 🎨 Reusable Code Snippets

### Email Validation
```javascript
const email = $json.email?.trim().toLowerCase();
if (!email || !email.includes('@')) {
  return { error: 'Valid email required' };
}
```

### Safe JSON Parse
```javascript
const safeJsonParse = (str, fallback = []) => {
  try {
    return JSON.parse(str);
  } catch (e) {
    return fallback;
  }
};
```

### Format Text for HTML
```javascript
const formatText = (text) => {
  if (!text) return '<span class="empty">Not yet generated</span>';
  return text
    .replace(/\n/g, '<br>')
    .replace(/##\s*/g, '<strong>')
    .replace(/\*\*/g, '');
};
```

### Strip Leading Numbers
```javascript
const formatTextStripNumbers = (text) => {
  if (!text) return '<span class="empty">Not yet generated</span>';
  // Remove "1. ", "2. " etc. when text already has "Recommendation 1:" format
  return text.replace(/^\d+\.\s*/gm, '').replace(/\n/g, '<br>');
};
```

---

## 📝 Notes

- **Email as primary key**: All lookups use email address
- **JSON for arrays**: Store citation URLs as JSON.stringify'd arrays
- **Parallel when possible**: Use Merge nodes to wait for parallel API calls
- **Cost logging**: Track API usage in separate table
- **Graceful failures**: Always have error response paths

---

*Template created from Rainmaker (Pipeline Pro) system - December 2024*
