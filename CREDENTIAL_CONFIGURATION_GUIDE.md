# Credential Configuration Guide - SF Agent

## Overview

This guide walks you through configuring all credentials needed for the autonomous AI Salesforce developer agent to work in your n8n cloud instance.

**Time Required**: 30-45 minutes
**n8n Instance**: https://yourinstance.app.n8n.cloud

---

## Credentials Required

| Credential | Purpose | Used In |
|------------|---------|---------|
| **Salesforce OAuth2 API** | Deploy metadata via Tooling API | Workflow 3 - Deploy via Tooling API node |
| **Jira Software Cloud API** | Read/update Jira issues | All 3 workflows - Jira nodes |
| **Anthropic API** | Claude AI for analysis and metadata generation | Workflows 1, 2, 3 - Claude nodes |
| **n8n Webhook** | Internal webhook triggers | Workflows 2, 3 (auto-configured) |

---

## Part 1: Salesforce OAuth2 API Credential

### Step 1: Create Salesforce Connected App

1. **Log into Salesforce**:
   - Go to: your salesforce instance
   - Login as: your salesforce id
     
2. **Navigate to App Manager**:
   - Click ⚙️ (gear icon) → **Setup**
   - In Quick Find box, type: `App Manager`
   - Click **App Manager**

3. **Create New Connected App**:
   - Click **New Connected App** (top right)
   - Fill in the form:

**Basic Information**:
```
Connected App Name: XXXXXXXXXXXXXX
API Name: XXXXXXXXXXXXXX (auto-populated)
Contact Email: xyz@xyz.com
Description: Allows n8n to deploy Salesforce metadata via Tooling API
```

**API (Enable OAuth Settings)**:
- ✅ Check **Enable OAuth Settings**
- **Callback URL**:
  ```
  https://XXXXXXXXX.app.n8n.cloud/rest/oauth2-credential/callback
  ```
- **Selected OAuth Scopes** (Add these 4 scopes):
  - ✅ **Access and manage your data (api)**
  - ✅ **Perform requests on your behalf at any time (refresh_token, offline_access)**
  - ✅ **Provide access to your data via the Web (web)**
  - ✅ **Full access (full)**

**Additional Settings**:
- **Require Proof Key for Code Exchange (PKCE)**: ⬜ Unchecked (for n8n compatibility)
- **Require Secret for Web Server Flow**: ⬜ Unchecked
- **Require Secret for Refresh Token Flow**: ⬜ Unchecked
- **Enable Client Credentials Flow**: ⬜ Unchecked

4. **Save**:
   - Click **Save**
   - Click **Continue** on the warning message
   - **Important**: You'll see "Consumer Key" and "Consumer Secret" - KEEP THIS PAGE OPEN

5. **Copy Credentials**:
   - **Consumer Key** (Client ID): Copy this (starts with `3MVG...`)
   - **Consumer Secret** (Client Secret): Click to reveal, then copy
   - Save both in a secure location (you'll need them in Step 2)

6. **Edit Policies** (Important for immediate access):
   - Click **Manage** → **Edit Policies**
   - **Permitted Users**: Select **All users may self-authorize**
   - **IP Relaxation**: Select **Relax IP restrictions**
   - Click **Save**

### Step 2: Configure Credential in n8n

1. **Open n8n**:
   - Go to: https://pratapkp.app.n8n.cloud
   - Click **Settings** (left sidebar) → **Credentials**

2. **Add New Credential**:
   - Click **Add Credential** (top right)
   - Search for: `Salesforce OAuth2 API`
   - Click on it to open configuration

3. **Configure OAuth2 Credential**:

**Credential Name**:
```
Salesforce - westsideconsulting
```

**Environment**:
- Select: **Production** (or "Sandbox" if you want to test in sandbox first)

**Access Type**:
- Select: **Access Token**

**OAuth2 Configuration**:
- **Client ID**: Paste the Consumer Key from Step 1.5
- **Client Secret**: Paste the Consumer Secret from Step 1.5

**Salesforce Instance**:
```
https://westsideconsulting-dev-ed.my.salesforce.com
```

4. **Authorize**:
   - Click **Connect my account** (or **Sign in with Salesforce**)
   - You'll be redirected to Salesforce login
   - Login as: pratap.pattanayak@gmail.com
   - Click **Allow** on the OAuth consent screen
   - You'll be redirected back to n8n

5. **Verify & Save**:
   - You should see a green checkmark: "Connected"
   - Click **Save**

### Step 3: Link Credential to Workflow

1. **Open Workflow 3**:
   - In n8n, go to **Workflows** (left sidebar)
   - Click on **SF Agent 3 - Execute Changes** (ID: QjoWn0koYUvNNMC1)

2. **Configure "Deploy via Tooling API" Node**:
   - Click on the **"Deploy via Tooling API"** node
   - In the node settings panel:
     - **Authentication**: Should show **Predefined Credential Type**
     - **Credential Type**: Should show **Salesforce OAuth2 API**
     - **Credential for Salesforce OAuth2 API**: Select **Salesforce - westsideconsulting**
   - The node should now show a green checkmark (no errors)

3. **Save Workflow**:
   - Click **Save** (top right of workflow editor)

---

## Part 2: Jira Software Cloud API Credential

### Step 1: Verify API Token

You've already provided the Jira API token:
```
Email: pratap.pattanayak@gmail.com
Domain: pratappattanayak.atlassian.net
API Token: YOUR_JIRA_API_TOKEN_HERE (Get from https://id.atlassian.com/manage-profile/security/api-tokens)
```

**If you need to generate a new token**:
1. Go to: https://id.atlassian.com/manage-profile/security/api-tokens
2. Click **Create API token**
3. Label: `n8n SF Agent`
4. Click **Create**
5. Copy the token immediately (you can't view it again)

### Step 2: Configure Credential in n8n

1. **Open n8n**:
   - Go to: https://pratapkp.app.n8n.cloud
   - Click **Settings** → **Credentials**

2. **Add New Credential**:
   - Click **Add Credential**
   - Search for: `Jira Software Cloud`
   - Click on it to open configuration

3. **Configure Jira Credential**:

**Credential Name**:
```
Jira - pratappattanayak
```

**Authentication Method**:
- Select: **API Token**

**Configuration**:
```
Email: pratap.pattanayak@gmail.com
API Token: YOUR_JIRA_API_TOKEN_HERE (Get from https://id.atlassian.com/manage-profile/security/api-tokens)
Jira Domain: pratappattanayak.atlassian.net
```

4. **Test Credential** (Optional but recommended):
   - Click **Test** button
   - Should show: "Connection successful"

5. **Save**:
   - Click **Save**

### Step 3: Link Credential to All Workflows

#### Workflow 1: SF Agent 1 - Initial Analysis

1. Open workflow (ID: 2A6XqlHc9b48hjd6)
2. Update these Jira nodes:
   - **Jira Trigger - New SF Story**: Select credential `Jira - pratappattanayak`
   - **Jira - Get Issue Details**: Select credential `Jira - pratappattanayak`
   - **Jira - Post Questions Comment**: Select credential `Jira - pratappattanayak`
   - **Jira - Add Awaiting Response Label**: Select credential `Jira - pratappattanayak`
   - **Jira - Ready Comment**: Select credential `Jira - pratappattanayak`
   - **Jira - Add Ready Label**: Select credential `Jira - pratappattanayak`
3. Save workflow

#### Workflow 2: SF Agent 2 - Answer Processing

1. Open workflow (ID: Xa6YL7mUW7KB9efA)
2. Update these Jira nodes:
   - **Jira - Get Issue**: Select credential `Jira - pratappattanayak`
   - **Jira - Get Comments**: Select credential `Jira - pratappattanayak`
   - **Jira - More Questions Comment**: Select credential `Jira - pratappattanayak`
   - **Jira - Execute Comment**: Select credential `Jira - pratappattanayak`
   - **Jira - Remove Awaiting Label**: Select credential `Jira - pratappattanayak`
   - **Jira - Add Ready Label**: Select credential `Jira - pratappattanayak`
3. Save workflow

#### Workflow 3: SF Agent 3 - Execute Changes

1. Open workflow (ID: QjoWn0koYUvNNMC1)
2. Update these Jira nodes:
   - **Jira - Get Issue**: Select credential `Jira - pratappattanayak`
   - **Jira - Starting Comment**: Select credential `Jira - pratappattanayak`
   - **Jira - Update to In Progress**: Select credential `Jira - pratappattanayak`
   - **Jira - Success Comment**: Select credential `Jira - pratappattanayak`
   - **Jira - Mark Done**: Select credential `Jira - pratappattanayak`
   - **Jira - Transition to Done**: Select credential `Jira - pratappattanayak`
   - **Jira - Validation Error Comment**: Select credential `Jira - pratappattanayak`
3. Save workflow

---

## Part 3: Anthropic API Credential

### Step 1: Get Anthropic API Key

1. **Go to Anthropic Console**:
   - Visit: https://console.anthropic.com/settings/keys
   - Login with your Anthropic account

2. **Create API Key** (if you don't have one):
   - Click **Create Key**
   - Name: `n8n SF Agent`
   - Click **Create Key**
   - **Copy the key immediately** (you can't view it again)

3. **Verify API Key Format**:
   - Should start with: `sk-ant-api03-...`
   - Length: ~100+ characters

### Step 2: Configure Credential in n8n

1. **Open n8n**:
   - Go to: https://pratapkp.app.n8n.cloud
   - Click **Settings** → **Credentials**

2. **Add New Credential**:
   - Click **Add Credential**
   - Search for: `Anthropic`
   - Click on it to open configuration

3. **Configure Anthropic Credential**:

**Credential Name**:
```
Anthropic - Main
```

**API Key**:
```
[Paste your Anthropic API key here]
```

4. **Save**:
   - Click **Save**

### Step 3: Link Credential to All Workflows

The Claude AI nodes in all 3 workflows use HTTP Request nodes that call the Anthropic API directly. We need to add the API key to the headers.

#### Workflow 1: SF Agent 1 - Initial Analysis

1. Open workflow (ID: 2A6XqlHc9b48hjd6)
2. Click on **"Claude - Analyze Requirement"** node
3. In **Headers** section:
   - Find the header: `x-api-key`
   - Value should be: `={{ $credentials.anthropic.apiKey }}`
   - **Or use direct expression**: Update the header value to reference your credential
4. **Alternative Method** (if credential reference doesn't work):
   - Change header value to: `sk-ant-api03-YOUR_KEY_HERE`
   - (Not recommended for security, but works for testing)
5. Save workflow

#### Workflow 2: SF Agent 2 - Answer Processing

1. Open workflow (ID: Xa6YL7mUW7KB9efA)
2. Click on **"Claude - Re-analyze with Answers"** node
3. Update the `x-api-key` header (same as Workflow 1)
4. Save workflow

#### Workflow 3: SF Agent 3 - Execute Changes

1. Open workflow (ID: QjoWn0koYUvNNMC1)
2. Click on **"Claude - Generate Metadata"** node
3. Update the `x-api-key` header (same as Workflow 1)
4. Save workflow

**Better Approach** (Recommended):

Instead of referencing credentials in HTTP nodes, let's update the nodes to use proper authentication:

1. For each Claude HTTP Request node:
   - **Authentication**: Select **Generic Credential Type**
   - **Generic Auth Type**: Select **Header Auth**
   - **Credential for Header Auth**: Create new credential:
     - **Name**: `Anthropic API Header Auth`
     - **Header Name**: `x-api-key`
     - **Header Value**: `sk-ant-api03-YOUR_KEY_HERE`
   - Save and use this credential in all Claude nodes

---

## Part 4: Jira Webhook Configuration

### Step 1: Get Webhook URL from n8n

1. **Open Workflow 2** in n8n:
   - Go to **Workflows** → **SF Agent 2 - Answer Processing**
   - Click on **"Jira Webhook - Comment Added"** node

2. **Copy Production URL**:
   - You should see: **Production URL**
   - Copy the URL (should be):
     ```
     https://pratapkp.app.n8n.cloud/webhook/sf-agent-comment
     ```

### Step 2: Configure Webhook in Jira

1. **Go to Jira Settings**:
   - Visit: https://pratappattanayak.atlassian.net
   - Click ⚙️ **Settings** (top right) → **System**

2. **Navigate to WebHooks**:
   - In left sidebar, scroll down to **Advanced**
   - Click **WebHooks**

3. **Create New WebHook**:
   - Click **+ Create a WebHook** (top right)

4. **Configure WebHook**:

**Name**:
```
n8n SF Agent Comment Webhook
```

**Status**:
- ✅ **Enabled**

**URL**:
```
https://pratapkp.app.n8n.cloud/webhook/sf-agent-comment
```

**Description**:
```
Triggers n8n SF Agent when comments are added to Salesforce stories awaiting response
```

**Events** (Check these):
- Under **Issue**:
  - ⬜ Uncheck all
- Under **Comment**:
  - ✅ **created** (Check this ONLY)
- Under **Issue link**, **Worklog**, **Attachment**:
  - ⬜ Uncheck all

**JQL Filter** (Optional but recommended):
```
labels = sf-agent-awaiting-response
```
This ensures webhook only fires for issues waiting for answers.

**Exclude body** (Optional):
- ⬜ Leave unchecked (we need the full payload)

5. **Create**:
   - Click **Create**

6. **Test Webhook** (Optional):
   - Click **Test** next to your new webhook
   - Should show: "Webhook sent successfully"
   - Check n8n Workflow 2 executions to see if it received the test payload

### Step 3: Verify Webhook

1. **Create Test Jira Issue**:
   - Go to your Jira project
   - Create a Story with label: `salesforce`
   - Add label: `sf-agent-awaiting-response`
   - Add a comment: "Testing webhook"

2. **Check n8n Execution**:
   - Go to n8n → **Executions**
   - Filter by: **SF Agent 2 - Answer Processing**
   - You should see a new execution with the comment data

3. **If webhook doesn't fire**:
   - Check webhook is Enabled in Jira
   - Verify URL is correct
   - Check JQL filter doesn't exclude your test issue
   - Try removing JQL filter temporarily
   - Check Jira webhook logs (Settings → System → WebHooks → View details)

---

## Part 5: Activate All Workflows

### Step 1: Activate Workflow 1

1. Go to **Workflows** → **SF Agent 1 - Initial Analysis**
2. Click the **toggle switch** in top right (should turn blue/green)
3. Status should show: **Active**

### Step 2: Activate Workflow 2

1. Go to **Workflows** → **SF Agent 2 - Answer Processing**
2. Click the **toggle switch** in top right
3. Status should show: **Active**

### Step 3: Activate Workflow 3

1. Go to **Workflows** → **SF Agent 3 - Execute Changes**
2. Click the **toggle switch** in top right
3. Status should show: **Active**

### Step 4: Verify All Active

1. Go to **Workflows** (main list)
2. All 3 workflows should show green "Active" badge:
   - ✅ SF Agent 1 - Initial Analysis
   - ✅ SF Agent 2 - Answer Processing
   - ✅ SF Agent 3 - Execute Changes

---

## Part 6: Testing Credential Configuration

### Test 1: Test Jira Connection

1. **Create Test Issue**:
   - Go to: https://pratappattanayak.atlassian.net
   - Create new **Story**:
     - **Summary**: Test - Simple field
     - **Description**: Add text field named Test_Field__c to Account
     - **Labels**: Add `salesforce`
   - Click **Create**

2. **Check Workflow 1 Execution**:
   - Go to n8n → **Executions**
   - Filter by: **SF Agent 1 - Initial Analysis**
   - Should see new execution within 1-2 minutes (polling interval)
   - Click on execution to see details

3. **Expected Result**:
   - "Jira Trigger" node: Shows issue data
   - "Extract Issue Details" node: Shows parsed data
   - **If this fails**: Jira credential is not configured correctly

### Test 2: Test Anthropic API

1. **Check Claude Node**:
   - In the same Workflow 1 execution
   - Click on "Claude - Analyze Requirement" node
   - Should show Claude's JSON response

2. **Expected Result**:
   - Response should be JSON with fields: `isValid`, `configType`, `questions`, etc.
   - **If this fails**: Anthropic API key is invalid or not configured

### Test 3: Test Salesforce OAuth2

1. **Manual Test** (before full workflow):
   - Go to Workflow 3
   - Click **Execute Workflow** (top right)
   - Manually trigger with test data
   - Check "Deploy via Tooling API" node

2. **Expected Result**:
   - Should NOT show authentication error
   - **If fails with 401 Unauthorized**: OAuth2 credential needs re-authorization

---

## Troubleshooting

### Issue: Salesforce OAuth2 "Authentication failed"

**Solutions**:
1. Re-authorize the credential:
   - Go to Settings → Credentials
   - Click on "Salesforce - westsideconsulting"
   - Click **Reconnect**
   - Complete OAuth flow again

2. Check Connected App in Salesforce:
   - Go to Setup → App Manager
   - Find your Connected App
   - Click **Manage** → **Edit Policies**
   - Ensure **IP Relaxation** is "Relax IP restrictions"

3. Verify scopes:
   - Connected App must have: `api`, `refresh_token`, `web`, `full`

### Issue: Jira "401 Unauthorized"

**Solutions**:
1. Verify API token is correct:
   - API tokens expire if not used for 90 days
   - Generate new token: https://id.atlassian.com/manage-profile/security/api-tokens
   - Update credential in n8n

2. Check email matches:
   - Email in credential must match the account that created the API token

3. Verify domain format:
   - Should be: `pratappattanayak.atlassian.net` (no `https://`)

### Issue: Anthropic API "Invalid API Key"

**Solutions**:
1. Verify API key format:
   - Should start with: `sk-ant-api03-`
   - No extra spaces or line breaks

2. Check API key is active:
   - Go to: https://console.anthropic.com/settings/keys
   - Verify key is listed and not deleted

3. Check billing:
   - Ensure Anthropic account has credits/billing configured

### Issue: Webhook not triggering Workflow 2

**Solutions**:
1. Verify webhook URL:
   - Must exactly match Production URL from n8n
   - Should end with `/webhook/sf-agent-comment`

2. Check webhook is Enabled in Jira

3. Test webhook manually:
   - Jira → Settings → System → WebHooks
   - Click your webhook → **Test**
   - Check n8n Executions for test payload

4. Remove JQL filter temporarily:
   - JQL filter might be excluding test issues

---

## Verification Checklist

Before proceeding to full testing, verify:

- [ ] Salesforce OAuth2 credential created and authorized
- [ ] "Deploy via Tooling API" node shows green checkmark (no auth error)
- [ ] Jira credential created with correct API token
- [ ] All Jira nodes in all 3 workflows linked to Jira credential
- [ ] Anthropic API key configured in all Claude HTTP Request nodes
- [ ] Jira webhook created and enabled
- [ ] Jira webhook URL matches n8n Production URL
- [ ] All 3 workflows are Active (green badge)
- [ ] Test Jira issue created triggers Workflow 1
- [ ] Test comment on issue with `sf-agent-awaiting-response` label triggers Workflow 2

---

## Next Steps

Once all credentials are configured and verified:

1. **Review**: [TOOLING_API_IMPLEMENTATION.md](TOOLING_API_IMPLEMENTATION.md:1) - Complete implementation details
2. **Test**: [SF_AGENT_SETUP_GUIDE.md](SF_AGENT_SETUP_GUIDE.md:99) - Follow Test 1, 2, 3 scenarios
3. **Monitor**: Check n8n Executions after each test
4. **Iterate**: Refine AI prompts based on results

---

## Quick Reference

### Credentials Summary

| Credential | Name in n8n | Purpose |
|------------|-------------|---------|
| Salesforce OAuth2 | `Salesforce - westsideconsulting` | Tooling API deployment |
| Jira Software Cloud | `Jira - pratappattanayak` | Issue management |
| Anthropic API | `Anthropic - Main` | Claude AI analysis |

### URLs

- **n8n Instance**: https://pratapkp.app.n8n.cloud
- **Salesforce Org**: https://westsideconsulting-dev-ed.my.salesforce.com
- **Jira Instance**: https://pratappattanayak.atlassian.net
- **Anthropic Console**: https://console.anthropic.com
- **Workflow 2 Webhook**: https://pratapkp.app.n8n.cloud/webhook/sf-agent-comment

### Workflow IDs

- **Workflow 1**: `2A6XqlHc9b48hjd6` (Initial Analysis)
- **Workflow 2**: `Xa6YL7mUW7KB9efA` (Answer Processing)
- **Workflow 3**: `QjoWn0koYUvNNMC1` (Execute Changes)

---

**Ready to test!** Once all credentials are configured, create a test Jira Story and watch the autonomous AI Salesforce developer agent in action.
