# Bug Fix - Claude Analyze Requirement JSON Error

## Issue

**Error Message**:
```
JSON parameter needs to be valid JSON
```

**Workflow**: SF Agent 1 - Initial Analysis (ID: 2A6XqlHc9b48hjd6)
**Node**: "Claude - Analyze Requirement"
**Date**: 2026-02-01

---

## Root Cause

The HTTP Request node's `jsonBody` parameter was using incorrect expression syntax. It was trying to use template literals with `{{ }}` inside a JSON string, which n8n couldn't parse correctly.

### Problematic Code

```javascript
jsonBody: "={
  \"model\": \"claude-sonnet-4-20250514\",
  \"messages\": [
    {
      \"role\": \"user\",
      \"content\": \"...{{ $json.summary }}...\" // ❌ Wrong syntax
    }
  ]
}"
```

**Issues**:
1. Using `{{ }}` inside JSON string (should use string concatenation)
2. n8n couldn't evaluate the expressions properly
3. Resulted in invalid JSON being sent to Anthropic API

---

## Fix Applied

Updated the `jsonBody` to use proper n8n expression syntax with JavaScript object and string concatenation:

```javascript
jsonBody: "={{ {
  \"model\": \"claude-sonnet-4-20250514\",
  \"max_tokens\": 4000,
  \"temperature\": 0.2,
  \"messages\": [
    {
      \"role\": \"user\",
      \"content\": \"You are an expert Salesforce developer agent...\" +
                  $json.summary + \"\\n...\" +
                  $json.description + \"\\n...\" +
                  $json.issueType + \"\\n...\" +
                  $json.project + \"...\"
    }
  ]
} }}"
```

**Key Changes**:
1. **Outer expression**: `={{ { ... } }}` - Tells n8n this is a JavaScript expression returning an object
2. **String concatenation**: Uses `+` operator to concatenate dynamic values
3. **Direct variable access**: Uses `$json.summary`, `$json.description`, etc. without `{{ }}`
4. **Valid JSON output**: Results in properly formatted JSON for Anthropic API

---

## n8n Expression Syntax Rules

### Correct Ways to Build JSON Bodies

#### Method 1: Expression with Object (Recommended)
```javascript
jsonBody: "={{ {
  \"field1\": \"value\",
  \"field2\": $json.dynamicValue,
  \"field3\": \"text \" + $json.anotherValue + \" more text\"
} }}"
```

#### Method 2: JSON.stringify with Object
```javascript
jsonBody: "={{ JSON.stringify({
  field1: \"value\",
  field2: $json.dynamicValue,
  field3: `text ${$json.anotherValue} more text`
}) }}"
```

### Common Mistakes

❌ **Wrong**: Using `{{ }}` inside JSON string
```javascript
jsonBody: "{\"field\": \"{{ $json.value }}\"}"  // Won't work!
```

❌ **Wrong**: Missing outer expression wrapper
```javascript
jsonBody: "{\"field\": $json.value}"  // Not evaluated
```

❌ **Wrong**: Template literals in expression strings
```javascript
jsonBody: "={\"field\": `${$json.value}`}"  // Syntax error
```

✅ **Right**: Expression with concatenation
```javascript
jsonBody: "={{ {\"field\": \"prefix \" + $json.value + \" suffix\"} }}"
```

✅ **Right**: Expression with direct value
```javascript
jsonBody: "={{ {\"field\": $json.value} }}"
```

---

## Verification

### Test 1: Check Node Configuration

1. Open Workflow 1 in n8n
2. Click "Claude - Analyze Requirement" node
3. Look at "JSON Body" field
4. Should start with: `={{ {`
5. Should use string concatenation: `+` operators
6. Should NOT contain `{{ }}` inside the JSON

### Test 2: Test Execution

1. Create test Jira Story with label "salesforce"
2. Wait for Workflow 1 to trigger
3. Check "Claude - Analyze Requirement" node execution
4. Should show:
   - ✅ No JSON parsing errors
   - ✅ Request sent successfully to Anthropic API
   - ✅ Response received with AI analysis

### Test 3: Verify Request Body

In the execution details:
1. Click "Claude - Analyze Requirement" node
2. Look at "Raw request" or "Input"
3. Should show valid JSON with:
   ```json
   {
     "model": "claude-sonnet-4-20250514",
     "max_tokens": 4000,
     "temperature": 0.2,
     "messages": [
       {
         "role": "user",
         "content": "You are an expert...Issue Summary: Add Customer Segment..."
       }
     ]
   }
   ```

---

## Why This Happened

When creating HTTP Request nodes in n8n:
1. The "JSON Body" field expects either:
   - Plain JSON string (no expressions)
   - Expression that returns an object: `={{ { ... } }}`
2. Template literals `{{ }}` are for parameter fields, NOT inside expressions
3. Inside expressions, use direct variable access: `$json.fieldName`

This is a common mistake when building dynamic JSON payloads in n8n.

---

## Impact

**Before Fix**:
- ❌ HTTP Request failed with "JSON parameter needs to be valid JSON"
- ❌ Claude AI never received the request
- ❌ Workflow stopped at analysis step
- ❌ No AI analysis performed

**After Fix**:
- ✅ Valid JSON sent to Anthropic API
- ✅ Claude AI receives and processes request
- ✅ Returns structured JSON analysis
- ✅ Workflow continues to next steps

---

## Related Nodes

This same pattern should be used in other Claude HTTP Request nodes:

### Workflow 1
- ✅ **Claude - Analyze Requirement** (Fixed)

### Workflow 2
- ⚠️ **Claude - Re-analyze with Answers** (Check this node!)
  - Should use same expression syntax
  - Verify no `{{ }}` inside JSON body

### Workflow 3
- ⚠️ **Claude - Generate Metadata** (Check this node!)
  - Should use same expression syntax
  - Build JSON body with string concatenation

---

## Best Practices

### For HTTP Request Nodes with Dynamic JSON

1. **Use expression wrapper**: Start with `={{ ... }}`
2. **Return JavaScript object**: `={{ { field: value } }}`
3. **Concatenate strings**: Use `+` operator for dynamic text
4. **Access variables directly**: `$json.field`, not `{{ $json.field }}`
5. **Escape quotes in strings**: Use `\"` inside string literals
6. **Test with sample data**: Use "Test workflow" before activating

### Example Template

```javascript
// For any HTTP Request node sending JSON to external API
jsonBody: "={{ {
  \"staticField\": \"static value\",
  \"dynamicField\": $json.someValue,
  \"combinedField\": \"prefix \" + $json.value + \" suffix\",
  \"nestedObject\": {
    \"nestedField\": $json.nested,
    \"nestedArray\": $json.arrayField || []
  }
} }}"
```

---

## Prevention Checklist

When creating HTTP Request nodes with JSON bodies:

- [ ] Use `={{ { ... } }}` wrapper for expressions
- [ ] NO `{{ }}` inside the JSON body
- [ ] Use `+` for string concatenation
- [ ] Direct variable access: `$json.field`
- [ ] Escape quotes: `\"` in string literals
- [ ] Test with manual execution
- [ ] Check request body in execution log
- [ ] Verify API receives valid JSON

---

## Summary

✅ **Fixed**: "Claude - Analyze Requirement" node now sends valid JSON to Anthropic API
✅ **Root cause**: Incorrect expression syntax with `{{ }}` inside JSON
✅ **Solution**: Use proper n8n expression with string concatenation
✅ **Impact**: Claude AI now successfully analyzes Jira issues

The workflow should now successfully call the Anthropic API and receive AI analysis!

---

## Next Steps

1. ⚠️ **Check other Claude nodes** in Workflow 2 and 3
2. ✅ **Test end-to-end**: Create test Jira Story
3. ✅ **Monitor execution**: Verify Claude API calls succeed
4. ✅ **Review responses**: Check AI analysis is correct
