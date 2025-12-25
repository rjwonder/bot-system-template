# LibreChat Setup & Migration Guide

*Lessons learned from migrating bots from Open WebUI to LibreChat*

---

## Architecture Overview

```
LibreChat Agent + Action → Claude API → n8n Webhook (Async) → Airtable (Logging)
```

### Key Philosophy
- **User gets instant streaming responses** - Claude responds directly
- **Logging happens asynchronously** - n8n handles data persistence in the background
- **Actions use OpenAPI specs** - No embedded Python code like Open WebUI pipes

### Component Breakdown

| Component | Purpose | Key Files |
|-----------|---------|-----------|
| LibreChat Agent | Orchestrates conversation | Agent config in UI |
| Actions | External API calls | OpenAPI spec (YAML/JSON) |
| n8n Webhooks | Async processing | Workflow JSON |
| Airtable | Data persistence | Base schema |

---

## Bot Migration Checklist (Open WebUI → LibreChat)

### From Old System (Open WebUI)

- [ ] Export system prompt from pipe function's `get_system_prompt()`
- [ ] Document webhook URL and payload structure
- [ ] Note any special state management logic
- [ ] Export any knowledge base content
- [ ] Document conversation flow stages

### For LibreChat

- [ ] Create OpenAPI spec for any webhook Actions
- [ ] Adapt system prompt (add Action calling instructions)
- [ ] Create Agent in LibreChat UI
- [ ] Configure model settings (Claude recommended)
- [ ] Test end-to-end flow
- [ ] Verify n8n receives correct payload

### Key Differences

| Open WebUI | LibreChat |
|------------|-----------|
| Pipe Functions (Python) | Actions (OpenAPI specs) |
| `get_system_prompt()` | Agent system prompt field |
| Model dropdown | Agent model configuration |
| Valves for config | Environment variables |

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
                session_id:
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
- session_id: The current conversation ID
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

## Common Migration Pitfalls

### 1. State Management
- **Open WebUI**: Session state in pipe valves
- **LibreChat**: Stateless - use conversation context or external storage

### 2. Webhook Payloads
- Ensure n8n workflows expect the new payload format from Actions
- Test with actual LibreChat requests, not assumptions

### 3. System Prompt Length
- LibreChat may have different token limits
- Test with full production prompts before deployment

---

## Deployment Checklist

- [ ] LibreChat instance running (Railway, Docker, etc.)
- [ ] Agent created and configured
- [ ] Actions tested with n8n webhooks
- [ ] Custom CSS applied (if branding required)
- [ ] SSL/HTTPS configured
- [ ] Environment variables set
- [ ] Backup of configuration exported
