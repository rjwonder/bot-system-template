# Bot System Template

Reusable template for AI chatbot systems using n8n, Airtable, Claude, and Perplexity.

**Based on:** Rainmaker (Pipeline Pro) System
**Built for:** LibreChat

---

## 🎯 What This Template Provides

A complete architecture for building AI-powered bots that:

1. **Collect information** via chat interface
2. **Store and manage data** in Airtable
3. **Perform AI analysis** (website scraping, market research)
4. **Generate reports and content** using Claude
5. **Deliver outputs** via email and web reports

---

## 🔧 Technology Stack

| Component | Purpose |
|-----------|--------|
| **Chat Interface** | LibreChat |
| **Workflow Engine** | n8n (self-hosted or cloud) |
| **Database** | Airtable |
| **AI Models** | Claude (Anthropic) |
| **Real-time Research** | Perplexity API |
| **Email** | Gmail via n8n |

---

## 📁 Repository Structure

```
bot-system-template/
├── README.md                 # This file
├── SYSTEM_TEMPLATE.md        # Complete system documentation
├── workflows/                # n8n workflow JSON exports
│   └── README.md            # How to import workflows
├── airtable/                 # Airtable schema documentation
│   └── schema.md            # Field definitions
├── patterns/                 # Reusable code patterns
│   ├── citations.js         # Citation linking
│   ├── html-report.js       # HTML generation
│   └── validation.js        # Input validation
└── docs/                     # Additional documentation
    ├── librechat-setup.md   # LibreChat architecture & migration
    ├── css-theming.md       # CSS theming (the v1-v10 journey)
    └── testing-protocols.md # Data verification protocols
```

---

## 🚀 Quick Start

### 1. Clone This Template
```bash
git clone https://github.com/rjwonder/bot-system-template.git my-new-bot
cd my-new-bot
```

### 2. Set Up Airtable
- Create a new base
- Follow the schema in `airtable/schema.md`
- Get your API key

### 3. Import n8n Workflows
- Import JSON files from `workflows/`
- Update Airtable base/table IDs
- Configure credentials

### 4. Configure LibreChat
- Set up your LibreChat instance
- Create Actions (OpenAPI specs) pointing to n8n webhooks
- Configure your Agent with system prompt
- Test the flow!

---

## 📊 Core Workflow Patterns

| Pattern | Purpose |
|---------|--------|
| **Smart Save** | Upsert data from chat to Airtable |
| **User Lookup** | Check if user exists, return data |
| **AI Analysis** | Scrape + analyze with Claude |
| **Research** | Real-time research with Perplexity |
| **Report Generation** | HTML reports with citations |
| **Email Delivery** | Send formatted emails |

---

## 📋 New Bot Checklist

- [ ] Clone this template
- [ ] Create Airtable base with required fields
- [ ] Import and configure n8n workflows
- [ ] Set up LibreChat Agent with Actions
- [ ] Configure credentials (Airtable, Claude, Perplexity, Gmail)
- [ ] Test end-to-end flow
- [ ] Customize for your use case

---

## 📖 Full Documentation

See [SYSTEM_TEMPLATE.md](./SYSTEM_TEMPLATE.md) for:
- Complete workflow inventory
- All Airtable field definitions
- Integration patterns with code examples
- LibreChat setup guide
- Reusable code snippets

### Lessons Learned (The Jefferies Tube Playbook)

| Doc | What You'll Learn |
|-----|-------------------|
| [LibreChat Setup](./docs/librechat-setup.md) | Architecture, Actions, Agent configuration |
| [CSS Theming](./docs/css-theming.md) | The v1-v10 journey, critical rules, copy-paste CSS template |
| [Testing Protocols](./docs/testing-protocols.md) | Data verification protocol - *conversation pass ≠ integration pass*

---

## 📝 Knowledge Workflow

This repository works alongside a **Notion playbook** for capturing lessons in real-time.

| Location | Purpose |
|----------|---------|
| **Notion** | Working notes - capture lessons AS they happen during debugging |
| **This Repo** | Published playbook - polished, organized, version-controlled reference |

**The workflow:**
1. Hit a problem? Jot it in Notion immediately (low friction)
2. Solve it? Add the solution to Notion
3. Periodically transfer polished lessons here to `docs/`
4. This repo becomes the "official" reference; Notion stays your scratch pad

---

## 🤝 Contributing

This template evolves with each new bot built. Contributions welcome!

---

*Created by RJ @ Black Belt Bots - December 2025*
