# n8n Cloud Solution - Salesforce AI Agent (No Execute Command)

## ⚠️ Problem: Execute Command Deprecated in n8n Cloud

The Execute Command node has been deprecated/disabled in n8n cloud for security reasons. This affects Workflow 3 which was designed to use Salesforce CLI commands.

## ✅ Solution Options for n8n Cloud

### Option 1: Use Salesforce Tooling API (Recommended for Cloud)

Replace Salesforce CLI commands with direct **Salesforce Tooling API** calls using HTTP Request nodes.

**Advantages:**
- ✅ Works natively in n8n cloud
- ✅ No external dependencies
- ✅ Uses existing Salesforce OAuth credentials
- ✅ Direct API access - more reliable
- ✅ Better error handling

**Disadvantages:**
- ⚠️ More complex to implement
- ⚠️ Requires understanding Salesforce Tooling API
- ⚠️ May not support all metadata types initially

---

### Option 2: External Service with Webhook

Set up an external service (Railway, Heroku, AWS Lambda) that has Salesforce CLI installed.

**Architecture:**
```
n8n Workflow → Webhook → External Service (with SF CLI) → Response
```

**Advantages:**
- ✅ Can use all SF CLI features
- ✅ Existing workflow logic mostly unchanged
- ✅ Full control over deployment process

**Disadvantages:**
- ⚠️ Requires external infrastructure
- ⚠️ Additional cost
- ⚠️ More complex setup
- ⚠️ Requires maintaining external service

---

### Option 3: Simplified Approach - Use n8n Salesforce Node

Use the built-in n8n Salesforce node for simpler operations.

**Advantages:**
- ✅ Simple configuration
- ✅ Works in cloud
- ✅ No external dependencies

**Disadvantages:**
- ❌ Limited metadata type support
- ❌ Can't create validation rules, workflows, page layouts
- ❌ Only supports basic CRUD operations

---

## Recommended Implementation: Salesforce Tooling API

### Architecture Overview

```
Workflow 3 (Updated):
  Parse Generated Metadata
    ↓
  Convert to Tooling API Format (Code Node)
    ↓
  Deploy via Tooling API (HTTP Request)
    ↓
  Check Status (Code Node)
    ↓
  Success Comment → Mark Done
```

### Key Changes from Original Design

| Original (SF CLI) | New (Tooling API) |
|-------------------|-------------------|
| Execute Command - Create Dir | Not needed (API-based) |
| Execute Command - Write Files | Code Node (prepare JSON) |
| Execute Command - Validate | HTTP Request (Tooling API) |
| Execute Command - Deploy | HTTP Request (Tooling API) |

---

## Implementation Details

### Node 1: Convert to Tooling API Format (Code Node)

Replace the "Execute Command - Create Directory" with:

**Node Name**: Prepare Tooling API Requests
**Type**: Code Node (JavaScript)
**Code**:

```javascript
// Convert AI-generated metadata to Salesforce Tooling API format
const data = $input.first().json;
const metadata = data.metadata;

const requests = [];

// Handle Custom Fields
for (const meta of metadata.metadata || []) {
  if (meta.type === 'CustomField') {
    // Parse field definition from AI
    const fieldDef = parseCustomField(meta);

    requests.push({
      type: 'CustomField',
      method: 'POST',
      endpoint: '/services/data/v64.0/tooling/sobjects/CustomField',
      body: fieldDef
    });
  }

  // Handle Validation Rules
  if (meta.type === 'ValidationRule') {
    const validationDef = parseValidationRule(meta);

    requests.push({
      type: 'ValidationRule',
      method: 'POST',
      endpoint: '/services/data/v64.0/tooling/sobjects/ValidationRule',
      body: validationDef
    });
  }
}

function parseCustomField(meta) {
  // Parse XML or JSON metadata to Tooling API format
  return {
    FullName: `${meta.objectName}.${meta.fieldName}`,
    Metadata: {
      type: meta.fieldType || 'Text',
      label: meta.label,
      length: meta.length,
      required: meta.required || false
    }
  };
}

function parseValidationRule(meta) {
  return {
    FullName: `${meta.objectName}.${meta.ruleName}`,
    Metadata: {
      active: true,
      errorConditionFormula: meta.formula,
      errorMessage: meta.errorMessage
    }
  };
}

return [{
  json: {
    ...data,
    toolingRequests: requests,
    totalRequests: requests.length
  }
}];
```

---

### Node 2: Deploy via Tooling API (HTTP Request)

**Node Name**: Deploy to Salesforce (Tooling API)
**Type**: HTTP Request
**Configuration**:

```
Method: POST
URL: =https://westsideconsulting-dev-ed.my.salesforce.com/services/data/v64.0/tooling/sobjects/{{ $json.toolingRequests[0].type }}

Authentication: OAuth2 (Salesforce)
- Use your existing Salesforce credential

Headers:
  Content-Type: application/json

Body:
  ={{ JSON.stringify($json.toolingRequests[0].body) }}
```

**For Multiple Requests**, use a **Loop** node:
```
Loop over items → HTTP Request → Collect Results
```

---

### Node 3: Check Deployment Status (Code Node)

**Node Name**: Verify Deployment
**Type**: Code Node
**Code**:

```javascript
// Check if Tooling API deployment was successful
const response = $input.first().json;
const issueData = $items('Parse Generated Metadata')[0].json;

// Tooling API returns { id: "...", success: true } on success
const isSuccess = response.success === true && response.id;

// Get any errors
const errors = response.errors || [];

return [{
  json: {
    ...issueData,
    validationPassed: isSuccess,
    deploymentResult: response,
    deploymentId: response.id,
    validationMessage: isSuccess
      ? 'Deployment successful'
      : `Deployment failed: ${errors.join(', ')}`
  }
}];
```

---

## Salesforce Tooling API Reference

### Endpoints

#### 1. Create Custom Field
```
POST /services/data/v64.0/tooling/sobjects/CustomField
```

**Body Example**:
```json
{
  "FullName": "Account.Industry_Type__c",
  "Metadata": {
    "type": "Picklist",
    "label": "Industry Type",
    "required": false,
    "valueSet": {
      "valueSetDefinition": {
        "value": [
          {"fullName": "Technology", "label": "Technology"},
          {"fullName": "Healthcare", "label": "Healthcare"},
          {"fullName": "Finance", "label": "Finance"}
        ]
      }
    }
  }
}
```

#### 2. Create Validation Rule
```
POST /services/data/v64.0/tooling/sobjects/ValidationRule
```

**Body Example**:
```json
{
  "FullName": "Account.Prevent_Duplicate_Name",
  "Metadata": {
    "active": true,
    "errorConditionFormula": "LEN(Name) < 3",
    "errorMessage": "Account name must be at least 3 characters",
    "errorDisplayField": "Name"
  }
}
```

#### 3. Check Deployment Status
```
GET /services/data/v64.0/tooling/sobjects/CustomField/{Id}
```

---

## Updated AI Prompts

### Prompt 3: Generate Metadata (Updated for Tooling API)

```
You are an expert Salesforce developer agent. Generate Salesforce Tooling API requests to implement this configuration.

**Requirement**: {{ $json.summary }}
{{ $json.description }}

**Target Org**: westsideconsulting-dev-ed.my.salesforce.com

Generate Tooling API request bodies (NOT CLI commands, NOT XML). Use Tooling API JSON format.

Respond in JSON format:
{
  "metadata": [
    {
      "type": "CustomField",
      "objectName": "Account",
      "fieldName": "Industry_Type__c",
      "toolingApiBody": {
        "FullName": "Account.Industry_Type__c",
        "Metadata": {
          "type": "Picklist",
          "label": "Industry Type",
          "valueSet": {...}
        }
      }
    }
  ],
  "verificationSteps": [
    "Navigate to Setup → Object Manager → Account → Fields",
    "Verify Industry_Type__c field exists",
    "Check picklist values"
  ]
}
```

---

## Complete Node Flow (Updated for n8n Cloud)

```
Workflow 3: SF Agent - Execute Changes (Cloud Version)

1. Webhook - Execute Trigger
2. Jira - Get Issue
3. Verify Ready to Execute (Code)
4. Jira - Starting Comment
5. Jira - Update to In Progress
6. Claude - Generate Metadata (with updated prompt)
7. Parse Generated Metadata (Code)
8. Prepare Tooling API Requests (Code) ← NEW
9. Loop Node (if multiple requests) ← NEW
   └─ Deploy via Tooling API (HTTP Request) ← NEW
10. Check Deployment Status (Code) ← NEW
11. IF Deployment Passed?
12a. Jira - Success Comment
12b. Jira - Mark Done
12c. Jira - Transition to Done
OR
13. Jira - Validation Error Comment
```

---

## Quick Start: Rebuild Workflow 3 for Cloud

### Step 1: Delete Old Nodes

In Workflow 3, delete these nodes (they won't work in cloud):
- ❌ Execute Command - Create Directory Structure
- ❌ Execute Command - Execute Write Script
- ❌ Execute Command - Validate Deployment
- ❌ Bash - Deploy to Salesforce

### Step 2: Add New Nodes

1. **After "Parse Generated Metadata"**, add:
   - Code Node: "Prepare Tooling API Requests"
   - Copy the JavaScript code from section above

2. **After that**, add:
   - HTTP Request Node: "Deploy via Tooling API"
   - Configure OAuth2 to Salesforce
   - URL: `https://westsideconsulting-dev-ed.my.salesforce.com/services/data/v64.0/tooling/sobjects/CustomField`
   - Method: POST
   - Body: `={{ JSON.stringify($json.toolingRequests[0].toolingApiBody) }}`

3. **After HTTP Request**, add:
   - Code Node: "Check Deployment Status"
   - Copy the verification code from above

4. **Connect to existing "Validation Passed?" IF node**

### Step 3: Update Claude AI Prompt

Update the "Claude - Generate Metadata" node to use the new Tooling API prompt format (see above).

### Step 4: Test

1. Create test Jira Story with simple custom field
2. Trigger workflow manually
3. Verify Tooling API deployment works
4. Check Salesforce for new field

---

## Alternative: External Service Solution

If Tooling API is too complex, set up an external service:

### Option A: Railway Deployment

1. **Create Railway service**:
   ```bash
   # Dockerfile
   FROM node:18
   RUN npm install -g @salesforce/cli
   COPY server.js .
   CMD ["node", "server.js"]
   ```

2. **Simple Express server** (server.js):
   ```javascript
   const express = require('express');
   const { execSync } = require('child_process');

   const app = express();
   app.use(express.json());

   app.post('/deploy', (req, res) => {
     const { issueKey, metadata } = req.body;

     try {
       // Write files
       // Run SF CLI deploy
       const result = execSync('sf project deploy start --source-dir force-app --target-org user@org.com --json');
       res.json({ success: true, result: JSON.parse(result) });
     } catch (error) {
       res.json({ success: false, error: error.message });
     }
   });

   app.listen(3000);
   ```

3. **Update n8n Workflow 3**:
   - Replace Execute Command nodes with HTTP Request to Railway service
   - URL: `https://your-railway-service.up.railway.app/deploy`
   - Body: Send metadata and issue key

---

## Recommendation

For **n8n cloud**, I recommend:

1. **Start with Tooling API approach** (Option 1)
   - Best for simple metadata (custom fields)
   - No external dependencies
   - More maintainable long-term

2. **Use external service** (Option 2) if you need:
   - Complex metadata types
   - Full SF CLI feature set
   - Page layouts, Flows, complex workflows

3. **Start simple**:
   - Phase 1: Support only Custom Fields (Tooling API)
   - Phase 2: Add Validation Rules (Tooling API)
   - Phase 3: Add complex metadata (external service if needed)

---

## Next Steps

1. ✅ Decide on approach (Tooling API vs External Service)
2. ✅ Update Workflow 3 nodes
3. ✅ Update Claude AI prompts
4. ✅ Test with simple custom field
5. ✅ Expand to other metadata types

Would you like me to help you implement the Tooling API approach or set up an external service?
