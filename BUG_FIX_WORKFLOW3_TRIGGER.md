# Bug Fix - Workflow 3 Not Triggering Autonomously

## Issue

**User Report**: "Workflow3 is not getting executed autonomously"

**Workflow**: SF Agent 1 - Initial Analysis (ID: 2A6XqlHc9b48hjd6)
**Affected Node**: "Trigger Workflow 3" (HTTP Request)
**Date**: 2026-02-01

---

## Root Cause

Data format mismatch between Workflow 1's HTTP Request and Workflow 3's Webhook expectations.

### Data Flow Problem

```
Workflow 1: "Trigger Workflow 3" node
  ↓ Sends: { issueKey: "KAN-9", summary: "...", description: "..." }
  ↓
Workflow 3: Webhook node receives data
  ↓ Expects: { body: { issueKey: "KAN-9", ... } }
  ↓
Workflow 3: "Jira - Get Issue" node
  ↓ Tries to access: $json.body.issueKey
  ↓ Result: undefined (because data is at root, not under 'body')
```

### The Problem

**Workflow 1 sent**:
```javascript
jsonBody: "={{ JSON.stringify({
  issueKey: $json.issueKey,
  summary: $json.summary,
  description: $json.description
}) }}"

// Results in:
{
  "issueKey": "KAN-9",
  "summary": "Add Customer Segment field",
  "description": "..."
}
```

**Workflow 3 expected**:
```javascript
// Webhook receives data at $json
// But code accesses: $json.body.issueKey

// Expected structure:
{
  "body": {
    "issueKey": "KAN-9",
    "summary": "...",
    "description": "..."
  }
}
```

**What happened**:
1. Workflow 1 completes successfully, adds "sf-agent-ready-to-execute" label
2. Workflow 1 sends HTTP POST to Workflow 3's webhook
3. Workflow 3 receives the data, but at wrong nesting level
4. Workflow 3's "Jira - Get Issue" node tries to access `$json.body.issueKey`
5. `$json.body` is `undefined` because data is at `$json.issueKey`
6. Workflow 3 fails or can't find the issue

---

## Fix Applied

### Updated "Trigger Workflow 3" Node

Changed the jsonBody to wrap data under `body` key:

```javascript
// Before:
jsonBody: "={{ JSON.stringify({
  issueKey: $json.issueKey,
  summary: $json.summary,
  description: $json.description
}) }}"

// After:
jsonBody: "={{ JSON.stringify({
  body: {
    issueKey: $json.issueKey,
    summary: $json.summary,
    description: $json.description
  }
}) }}"
```

### Complete Node Configuration

```json
{
  "id": "http-trigger-workflow3-1",
  "name": "Trigger Workflow 3",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4,
  "parameters": {
    "method": "POST",
    "url": "https://<your-n8n-instance>.app.n8n.cloud/webhook/sf-agent-execute",
    "sendBody": true,
    "specifyBody": "json",
    "jsonBody": "={{ JSON.stringify({ body: { issueKey: $json.issueKey, summary: $json.summary, description: $json.description } }) }}"
  }
}
```

---

## Why This Happened

When creating inter-workflow HTTP triggers, different webhook patterns can be used:

### Pattern 1: Root Level Data (What we tried first)
```javascript
// Sender:
{ issueKey: "KAN-9", summary: "..." }

// Receiver accesses:
$json.issueKey  // ✅ Works
```

### Pattern 2: Nested Under Body (n8n webhook convention)
```javascript
// Sender:
{ body: { issueKey: "KAN-9", summary: "..." } }

// Receiver accesses:
$json.body.issueKey  // ✅ Works
```

**Our issue**: We mixed the patterns
- Workflow 1 used Pattern 1 (root level)
- Workflow 3 expected Pattern 2 (nested)

This is a common pattern in n8n webhooks where external systems send data nested under `body`, `query`, or `headers` keys.

---

## Verification

### Test 1: Check Node Configuration

1. Open Workflow 1 in n8n
2. Click "Trigger Workflow 3" node
3. Check **JSON Body** field
4. Should show: `={{ JSON.stringify({ body: { ... } }) }}`
5. Verify data is wrapped under `body` key

### Test 2: Manual Test

1. Open Workflow 1
2. Click "Execute Workflow" (use test data)
3. Check "Trigger Workflow 3" node output
4. Should show successful HTTP request
5. Check Workflow 3 execution list
6. Should see new execution triggered

### Test 3: End-to-End Test

1. Create new Jira Story with label "salesforce"
2. Wait for Workflow 1 to trigger (1 minute polling)
3. Verify Workflow 1 completes all steps
4. **Critical**: Check Workflow 3 executions list
5. Should see new execution appear
6. Verify Workflow 3 receives correct data:
   ```json
   {
     "body": {
       "issueKey": "KAN-9",
       "summary": "Add Customer Segment field",
       "description": "..."
     }
   }
   ```

---

## Impact

**Before Fix**:
- ❌ Workflow 1 completed successfully
- ❌ Workflow 3 never triggered or failed silently
- ❌ Issue stuck with "sf-agent-ready-to-execute" label
- ❌ No deployment attempted
- ❌ No errors visible to user

**After Fix**:
- ✅ Workflow 1 completes and triggers Workflow 3
- ✅ Workflow 3 receives data in correct format
- ✅ Workflow 3 can access `$json.body.issueKey`
- ✅ Deployment proceeds automatically
- ✅ Complete autonomous workflow execution

---

## Related Webhook Patterns in n8n

### HTTP Request to Webhook Patterns

**Pattern A: Simple Root Level** (for internal n8n webhooks)
```javascript
// Sender:
{ data: "value" }

// Receiver:
$json.data
```

**Pattern B: Nested Body** (standard HTTP convention)
```javascript
// Sender:
{ body: { data: "value" } }

// Receiver:
$json.body.data
```

**Pattern C: Multiple Nested** (complex webhooks)
```javascript
// Sender:
{
  body: { data: "value" },
  headers: { "content-type": "application/json" },
  query: { param: "value" }
}

// Receiver:
$json.body.data
$json.headers["content-type"]
$json.query.param
```

### Best Practice for Inter-Workflow Communication

When triggering one n8n workflow from another via webhook:

1. **Check the receiving workflow's code first**
   - Look at how it accesses `$json`
   - Match your sender to that pattern

2. **Use consistent nesting**
   - If receiver expects `$json.body`, send `{ body: {...} }`
   - If receiver expects `$json`, send data at root level

3. **Test both sides**
   - Test sending workflow's HTTP Request output
   - Test receiving workflow's webhook input
   - Verify data structure matches

4. **Document the contract**
   - Document expected webhook payload format
   - Add code comments in both workflows
   - Use descriptive variable names

---

## Prevention Checklist

When creating HTTP Request → Webhook connections:

- [ ] Check receiving webhook's data access pattern
- [ ] Match sender's payload structure to receiver's expectations
- [ ] Test HTTP Request node output format
- [ ] Test Webhook node input structure
- [ ] Verify data nesting level (root vs. nested)
- [ ] Test end-to-end with manual execution
- [ ] Check execution logs on both sides
- [ ] Document payload contract in code comments

---

## Common Mistakes

### Mistake #1: Assuming Root Level Access
```javascript
// ❌ Wrong - assumes webhook receives at root
jsonBody: "={{ { issueKey: $json.issueKey } }}"

// When webhook code expects:
$json.body.issueKey  // undefined!
```

### Mistake #2: Not Checking Receiver Code
```javascript
// Created sender without looking at receiver
// Result: data structure mismatch
```

### Mistake #3: Inconsistent Nesting
```javascript
// ❌ Wrong - mixed patterns
Workflow 1: sends { data: "value" }
Workflow 2: accesses $json.body.data
Workflow 3: accesses $json.data
```

✅ **Right**: Consistent pattern across all workflows

---

## Code Pattern Template

### For HTTP Request (Sender)
```javascript
// Check receiver's webhook code first!
// If receiver accesses $json.body.*, wrap data under body:

jsonBody: "={{ JSON.stringify({
  body: {
    issueKey: $json.issueKey,
    summary: $json.summary,
    // ... other fields
  }
}) }}"
```

### For Webhook (Receiver)
```javascript
// Access data based on how sender structured it
const webhookData = $input.first().json;

// If sender wrapped under 'body':
const data = webhookData.body;

// If sender sent at root:
const data = webhookData;

// Defensive: handle both
const data = webhookData.body || webhookData;
```

---

## Summary

✅ **Fixed**: "Trigger Workflow 3" node now wraps data under `body` key
✅ **Root cause**: Data format mismatch between sender and receiver
✅ **Solution**: Match webhook payload structure to receiver's expectations
✅ **Impact**: Workflow 3 now triggers autonomously when Workflow 1 completes

The complete autonomous workflow now executes from Jira issue creation to Salesforce deployment!

---

## Next Steps

1. ✅ **Test end-to-end**: Create Jira Story and verify all 3 workflows execute
2. ⏳ **Check Workflow 2**: Verify similar webhook trigger has correct format
3. ⏳ **Monitor executions**: Check execution logs for successful triggers
4. ⏳ **Document payload contracts**: Add comments in webhook code

---

**Workflow 3 should now execute autonomously!** 🎉
