# Testing Protocols

*Because "it works" isn't the same as "it works correctly"*

---

## The Data Verification Gap

### The Lesson (December 15, 2025)

> **Pipeline Pro bot passed 14/14 conversation checkpoints. But Airtable contained garbage data.**

The `Full_Name` field contained "Start my Pipeline Pro session" instead of the actual name. Root cause: `extract_collected_data()` had a TODO comment and only extracted email.

### The Key Insight

> **n8n success ≠ correct data. Conversation pass ≠ integration pass. Always verify the destination.**

---

## The Three-Layer Testing Protocol

### Layer 1: Conversation Testing
- Does the bot respond correctly?
- Does it follow the conversation flow?
- Does it collect all required information?

### Layer 2: Integration Testing
- Does the webhook fire?
- Does n8n receive the payload?
- Does the workflow complete without errors?

### Layer 3: Data Verification ⚠️ MOST COMMONLY SKIPPED
- Is the data in Airtable CORRECT?
- Are all fields populated with ACTUAL values?
- Do the values match what the user provided?

---

## Data Verification Checklist

After every bot test, verify:

- [ ] Open Airtable/destination database
- [ ] Find the test record by session ID or timestamp
- [ ] Check EVERY field that should be populated:
  - [ ] `Full_Name` contains actual name (not prompt text)
  - [ ] `Email` contains valid email format
  - [ ] `Phone` contains actual phone number
  - [ ] `Company` contains company name
  - [ ] Custom fields contain expected values
- [ ] Verify timestamps are correct timezone
- [ ] Check for duplicate records

---

## Automated Verification with MCP

When Claude has MCP connections to your database:

```
After running conversation test:
1. Use Airtable MCP to query the latest record
2. Verify each field contains expected data
3. Report any mismatches immediately
```

### Example Verification Query

```javascript
// Query latest record from Airtable
const records = await airtable.list_records({
  baseId: "your-base-id",
  tableId: "Client Onboarding Data",
  maxRecords: 1,
  sort: [{ field: "Created", direction: "desc" }]
});

// Verify fields
const record = records[0];
const issues = [];

if (!record.Full_Name || record.Full_Name.includes("session")) {
  issues.push("Full_Name contains prompt text, not actual name");
}

if (!record.Email || !record.Email.includes("@")) {
  issues.push("Email is missing or invalid");
}

return issues.length > 0 ? issues : "All fields verified!";
```

---

## Pre-Deployment Checklist

### Knowledge Base Verification

Before deploying any bot:

1. **Identify Source KB** - Where does the bot's knowledge come from?
2. **Compare System Prompt** - Does it reference the correct KB?
3. **Gap Analysis** - What's missing between KB and prompt?
4. **Embed Critical Sections** - Include must-have info directly in prompt

### Why This Matters

> **"Claude was scoring vibes, not criteria"**

Without explicit criteria in the system prompt, the AI will improvise. Good for conversation, bad for data collection.

---

## Test Record Cleanup

After testing, clean up test data:

- [ ] Mark test records as "TEST" in status field
- [ ] Or delete test records entirely
- [ ] Clear any test sessions from logging tables
- [ ] Reset any counters or sequences if applicable

---

## Red Flags During Testing

Stop and investigate if you see:

| Red Flag | Likely Cause |
|----------|--------------|
| Correct conversation, wrong data | Extraction function broken |
| n8n shows success, empty Airtable | Field mapping wrong |
| Partial data only | Some fields not in payload |
| Duplicate records | Webhook firing multiple times |
| Old data appearing | Caching or session issue |

---

## The Golden Rule

> **Never ship a bot until you've verified the DESTINATION data, not just the CONVERSATION flow.**

Conversation testing tells you the bot talks correctly.
Data verification tells you the bot WORKS correctly.
