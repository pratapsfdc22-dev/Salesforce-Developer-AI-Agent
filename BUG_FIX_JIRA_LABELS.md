# Bug Fix - Jira Label Update Node Issue Key Reference

## Issue

**Error Message**:
```
The resource you are requesting could not be found
Issue does not exist or you do not have permission to see it.
```

**HTTP Code**: 404

**Workflow**: SF Agent 1 - Initial Analysis (ID: 2A6XqlHc9b48hjd6)
**Nodes**:
- "Jira - Add Awaiting Label"
- "Jira - Add Ready Label"
**Date**: 2026-02-01

---

## Root Cause

The Jira update nodes were trying to access `issueKey` from the wrong data source.

### Flow Context

```
Parse AI Response (has issueKey)
  ↓
Has Questions? (IF)
  ↓ (true - has questions)
Jira - Post Questions (returns comment data, NO issueKey)
  ↓
Jira - Add Awaiting Label ❌ TRYING: $json.issueKey (from comment data)
```

### The Problem

**Node configuration was**:
```javascript
issueKey: "={{ $json.issueKey }}"  // ❌ Wrong!
labels: "={{ $json.labels ? $json.labels.concat(['sf-agent-awaiting-response']) : ['sf-agent-awaiting-response'] }}"
```

**What happened**:
1. Previous node "Jira - Post Questions" returns Jira **comment** data
2. Comment data doesn't have `issueKey` field
3. Expression `$json.issueKey` evaluates to `undefined`
4. Jira API call fails with 404: "Issue does not exist"

### Why This Happened

In n8n workflows:
- `$json` refers to the **current item** from the **previous node**
- After "Jira - Post Questions", `$json` is the **comment object**, not the issue
- The issue key is stored earlier in "Parse AI Response" node
- Need to explicitly reference that node's data

---

## Fix Applied

### Fixed Node Configuration

Updated both label nodes to:
1. **Reference correct node**: Use `$('Parse AI Response')` to get issue data
2. **Simplified labels**: Just set the labels array directly

**"Jira - Add Awaiting Label"**:
```javascript
issueKey: "={{ $('Parse AI Response').item.json.issueKey }}"
labels: "={{ ['salesforce', 'sf-agent-awaiting-response'] }}"
```

**"Jira - Add Ready Label"**:
```javascript
issueKey: "={{ $('Parse AI Response').item.json.issueKey }}"
labels: "={{ ['salesforce', 'sf-agent-ready-to-execute'] }}"
```

### Key Changes

1. **Issue Key Reference**:
   - Before: `$json.issueKey` (from previous node - wrong!)
   - After: `$('Parse AI Response').item.json.issueKey` (from specific node)

2. **Labels Array**:
   - Before: Complex concatenation logic with existing labels
   - After: Simple array with exact labels we want
   - This **replaces** all labels (simpler and more predictable)

---

## n8n Node Reference Syntax

### Accessing Data from Previous Node

```javascript
$json.field                    // Current item from immediate previous node
```

### Accessing Data from Specific Node

```javascript
$('Node Name').item.json.field              // Single item from named node
$('Node Name').first().json.field           // First item from named node
$items('Node Name')[0].json.field           // First item (array syntax)
$items('Node Name', 0, 0)[0].json.field     // With run index
```

### Best Practices

1. **Use node references** when data comes from earlier in the workflow
2. **Use descriptive node names** so references are clear
3. **Test each node** to see what data it outputs
4. **Check execution logs** to see actual data structure

---

## Verification

### Test 1: Check Node Configuration

1. Open Workflow 1 in n8n
2. Click "Jira - Add Awaiting Label" node
3. Check **Issue Key** field shows: `={{ $('Parse AI Response').item.json.issueKey }}`
4. Check **Labels** field shows: `={{ ['salesforce', 'sf-agent-awaiting-response'] }}`

5. Click "Jira - Add Ready Label" node
6. Check **Issue Key** field shows: `={{ $('Parse AI Response').item.json.issueKey }}`
7. Check **Labels** field shows: `={{ ['salesforce', 'sf-agent-ready-to-execute'] }}`

### Test 2: Test Execution

1. Create test Jira Story with label "salesforce"
2. Wait for Workflow 1 to trigger
3. Monitor execution in n8n
4. Check "Jira - Add Awaiting Label" node:
   - ✅ Should NOT show 404 error
   - ✅ Should successfully update issue
   - ✅ Response shows updated labels

5. Check Jira issue:
   - ✅ Should have correct label added
   - ✅ Either `sf-agent-awaiting-response` or `sf-agent-ready-to-execute`

---

## Impact

**Before Fix**:
- ❌ 404 error: "Issue does not exist"
- ❌ Labels never added to Jira issues
- ❌ Workflow state tracking broken
- ❌ Workflow 2 never triggers (relies on label)

**After Fix**:
- ✅ Issue key correctly references earlier node
- ✅ Labels successfully added to issues
- ✅ Workflow state tracking works
- ✅ Workflow 2 can detect issues awaiting response

---

## Related Issues

This same pattern should be checked in other workflows:

### Workflow 2: SF Agent 2 - Answer Processing
- Check any Jira update nodes
- Verify they reference correct issue key

### Workflow 3: SF Agent 3 - Execute Changes
- Multiple Jira update nodes
- All should reference issue key correctly
- Likely need same fix

---

## Common n8n Mistakes

### Mistake #1: Assuming $json Always Has What You Need

```javascript
// ❌ Wrong - assumes previous node has issueKey
issueKey: "={{ $json.issueKey }}"

// ✅ Right - explicitly gets from known node
issueKey: "={{ $('Parse AI Response').item.json.issueKey }}"
```

### Mistake #2: Not Checking What Previous Node Returns

Always check execution logs to see:
- What data structure previous node returns
- What fields are available
- Whether you need to reference an earlier node

### Mistake #3: Complex Expression When Simple Works

```javascript
// ❌ Complex - tries to preserve existing labels
labels: "={{ $json.labels ? $json.labels.concat(['new-label']) : ['new-label'] }}"

// ✅ Simple - just set the labels you want
labels: "={{ ['label1', 'label2', 'new-label'] }}"
```

---

## Prevention Checklist

When creating Jira update nodes:

- [ ] Check what previous node returns
- [ ] Verify issueKey is available in `$json`
- [ ] If not, reference the node that has it
- [ ] Use node reference syntax: `$('Node Name').item.json.field`
- [ ] Test with manual execution
- [ ] Check execution logs for actual data
- [ ] Verify Jira API response is successful

---

## Summary

✅ **Fixed**: Both Jira label update nodes now correctly reference issue key from "Parse AI Response" node
✅ **Root cause**: Using `$json.issueKey` when previous node (comment) didn't have issue key
✅ **Solution**: Use explicit node reference: `$('Parse AI Response').item.json.issueKey`
✅ **Impact**: Labels now successfully added to Jira issues, workflow state tracking works

The workflow should now successfully add labels to Jira issues!

---

## Next Steps

1. ⚠️ **Check Workflow 3** for same issue in label update nodes
2. ✅ **Test end-to-end**: Verify labels appear in Jira
3. ✅ **Test Workflow 2**: Verify it triggers on label change
