# LibreChat Setup Guide

*How to configure LibreChat bots with n8n backends*

---

## Architecture Overview

```
LibreChat Agent + Action → Claude API → n8n Webhook (Async) → Airtable (Logging)
```

### Key Philosophy
- **User gets instant streaming responses** - Claude responds directly
- **Logging happens asynchronously** - n8n handles data persistence in the background
- **Actions use OpenAPI specs** - Clean API definitions for external calls

### Component Breakdown

| Component | Purpose | Key Files |
|-----------|---------|-----------|
| LibreChat Agent | Orchestrates conversation | Agent config in UI |
| Actions | External API calls | OpenAPI spec (YAML/JSON) |
| n8n Webhooks | Async processing | Workflow JSON |
| Airtable | Data persistence | Base schema |

---

## New Bot Setup Checklist

### 1. LibreChat Instance
- [ ] LibreChat running (Railway, Docker, etc.)
- [ ] SSL/HTTPS configured
- [ ] Anthropic API key set

### 2. Create Your Agent
- [ ] Create new Agent in LibreChat UI
- [ ] Set system prompt with data collection flow
- [ ] Add Action calling instructions to prompt
- [ ] Configure Claude model

### 3. Create Actions (OpenAPI Specs)
- [ ] Write OpenAPI spec for each n8n webhook
- [ ] Test each Action individually
- [ ] Attach Actions to your Agent

### 4. Backend Setup
- [ ] n8n workflows imported and configured
- [ ] Airtable base created with required fields
- [ ] Credentials configured (Airtable, Gmail, etc.)

### 5. Testing
- [ ] End-to-end flow works
- [ ] Data appears correctly in Airtable
- [ ] Emails send properly

---

## Creating LibreChat Actions

Actions in LibreChat use OpenAPI specifications. Here's the pattern:

```yaml
openapi: 3.0.0
info:
  title: Bot Action API
  version: 1.0.0
servers:
  - url: https://your-n8n-instance.com
paths:
  /webhook/your-endpoint:
    post:
      operationId: submitData
      summary: Submit collected data to backend
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                email:
                  type: string
                collected_data:
                  type: object
      responses:
        '200':
          description: Success
```

### Action Calling in System Prompts

Add explicit instructions for when to call actions:

```
When you have collected all required information, call the submitData action with:
- email: The user's email address
- collected_data: Object containing all collected fields
```

---

## Environment Configuration

LibreChat uses environment variables for configuration:

```env
# API Keys
ANTHROPIC_API_KEY=your-key

# Feature Flags
ENDPOINTS=anthropic
CHECK_BALANCE=false

# Custom Branding (optional)
APP_TITLE=Your Bot Name
```

---

## Common Setup Issues

### 1. State Management
LibreChat is stateless - use conversation context or external storage (Airtable) for persisting data between sessions.

### 2. Webhook Payloads
- Ensure n8n workflows expect the payload format from your Actions
- Test with actual LibreChat requests, not assumptions
- Use the testing-protocols.md to verify data actually saves correctly

### 3. System Prompt Length
- Test with full production prompts before deployment
- Claude can handle long prompts, but keep them focused

---

## Deployment Checklist

- [ ] LibreChat instance running (Railway, Docker, etc.)
- [ ] Agent created and configured
- [ ] Actions tested with n8n webhooks
- [ ] Custom CSS applied (if branding required - see css-theming.md)
- [ ] SSL/HTTPS configured
- [ ] Environment variables set
- [ ] Backup of configuration exported
