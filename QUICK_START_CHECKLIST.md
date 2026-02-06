# Quick Start Checklist - SF Agent Setup

## Overview

Follow these steps in order to configure and test your autonomous AI Salesforce developer agent.

**Estimated Time**: 30-45 minutes

---

## Pre-Setup Verification

Before starting, verify you have:

- ✅ n8n cloud instance running: https://pratapkp.app.n8n.cloud
- ✅ Salesforce org access: https://westsideconsulting-dev-ed.my.salesforce.com
- ✅ Jira instance access: https://pratappattanayak.atlassian.net
- ✅ Anthropic API account: https://console.anthropic.com
- ✅ All 3 workflows created in n8n (IDs: 2A6XqlHc9b48hjd6, Xa6YL7mUW7KB9efA, QjoWn0koYUvNNMC1)

---

## Step-by-Step Checklist

### Part 1: Salesforce OAuth2 Setup (15 minutes)

- [ ] **1.1** Log into Salesforce: https://westsideconsulting-dev-ed.my.salesforce.com
- [ ] **1.2** Go to Setup → App Manager
- [ ] **1.3** Click "New Connected App"
- [ ] **1.4** Fill in:
  - Connected App Name: `n8n SF Agent Integration`
  - Contact Email: `pratap.pattanayak@gmail.com`
  - Enable OAuth Settings: ✅
  - Callback URL: `https://pratapkp.app.n8n.cloud/rest/oauth2-credential/callback`
  - Selected Scopes: `api`, `refresh_token`, `web`, `full`
- [ ] **1.5** Save and copy:
  - Consumer Key (Client ID)
  - Consumer Secret (Client Secret)
- [ ] **1.6** Edit Policies:
  - Permitted Users: "All users may self-authorize"
  - IP Relaxation: "Relax IP restrictions"
- [ ] **1.7** Go to n8n → Settings → Credentials
- [ ] **1.8** Add "Salesforce OAuth2 API" credential:
  - Name: `Salesforce - westsideconsulting`
  - Client ID: [paste Consumer Key]
  - Client Secret: [paste Consumer Secret]
  - Instance URL: `https://westsideconsulting-dev-ed.my.salesforce.com`
- [ ] **1.9** Click "Connect my account" and authorize
- [ ] **1.10** Open Workflow 3, link credential to "Deploy via Tooling API" node
- [ ] **1.11** Save Workflow 3

**Verification**: "Deploy via Tooling API" node shows green checkmark (no red error)

---

### Part 2: Jira API Setup (10 minutes)

- [ ] **2.1** Verify you have Jira API token:
  ```
  Email: pratap.pattanayak@gmail.com
  API Token: YOUR_JIRA_API_TOKEN_HERE (Get from https://id.atlassian.com/manage-profile/security/api-tokens)
  Domain: pratappattanayak.atlassian.net
  ```
- [ ] **2.2** Go to n8n → Settings → Credentials
- [ ] **2.3** Add "Jira Software Cloud" credential:
  - Name: `Jira - pratappattanayak`
  - Email: `pratap.pattanayak@gmail.com`
  - API Token: [paste token above]
  - Domain: `pratappattanayak.atlassian.net`
- [ ] **2.4** Click "Test" (should show "Connection successful")
- [ ] **2.5** Save credential
- [ ] **2.6** Open Workflow 1, link credential to all Jira nodes (6 nodes)
- [ ] **2.7** Open Workflow 2, link credential to all Jira nodes (6 nodes)
- [ ] **2.8** Open Workflow 3, link credential to all Jira nodes (6 nodes)
- [ ] **2.9** Save all 3 workflows

**Verification**: All Jira nodes show green checkmark (no credential warnings)

---

### Part 3: Anthropic API Setup (5 minutes)

- [ ] **3.1** Go to: https://console.anthropic.com/settings/keys
- [ ] **3.2** Create new API key (if needed):
  - Name: `n8n SF Agent`
  - Copy key (starts with `sk-ant-api03-...`)
- [ ] **3.3** Go to n8n → Settings → Credentials
- [ ] **3.4** Add "Anthropic" credential:
  - Name: `Anthropic - Main`
  - API Key: [paste your key]
- [ ] **3.5** Save credential
- [ ] **3.6** Open Workflow 1, configure "Claude - Analyze Requirement" node:
  - Authentication: Generic Credential Type → Header Auth
  - Create credential: "Anthropic API Header Auth"
  - Header Name: `x-api-key`
  - Header Value: [your Anthropic API key]
- [ ] **3.7** Open Workflow 2, link "Claude - Re-analyze with Answers" to same credential
- [ ] **3.8** Open Workflow 3, link "Claude - Generate Metadata" to same credential
- [ ] **3.9** Save all 3 workflows

**Verification**: No authentication errors when executing Claude nodes

---

### Part 4: Jira Webhook Setup (5 minutes)

- [ ] **4.1** Open Workflow 2 in n8n
- [ ] **4.2** Click "Jira Webhook - Comment Added" node
- [ ] **4.3** Copy Production URL:
  ```
  https://pratapkp.app.n8n.cloud/webhook/sf-agent-comment
  ```
- [ ] **4.4** Go to Jira: https://pratappattanayak.atlassian.net
- [ ] **4.5** Click Settings (⚙️) → System → WebHooks
- [ ] **4.6** Click "+ Create a WebHook"
- [ ] **4.7** Configure:
  - Name: `n8n SF Agent Comment Webhook`
  - Status: ✅ Enabled
  - URL: `https://pratapkp.app.n8n.cloud/webhook/sf-agent-comment`
  - Events: Comment → ✅ created (ONLY)
  - JQL Filter: `labels = sf-agent-awaiting-response`
- [ ] **4.8** Click "Create"
- [ ] **4.9** Click "Test" (optional)

**Verification**: Webhook shows "Enabled" status in Jira, test payload received in n8n

---

### Part 5: Activate Workflows (2 minutes)

- [ ] **5.1** Go to n8n → Workflows
- [ ] **5.2** Open "SF Agent 1 - Initial Analysis"
- [ ] **5.3** Click toggle switch (top right) → Active
- [ ] **5.4** Open "SF Agent 2 - Answer Processing"
- [ ] **5.5** Click toggle switch → Active
- [ ] **5.6** Open "SF Agent 3 - Execute Changes"
- [ ] **5.7** Click toggle switch → Active

**Verification**: All 3 workflows show green "Active" badge in workflow list

---

### Part 6: Test the Agent (10 minutes)

#### Test 1: Simple Custom Field (No Questions)

- [ ] **6.1** Go to Jira: https://pratappattanayak.atlassian.net
- [ ] **6.2** Create new Story:
  ```
  Summary: Add Customer Email field to Account
  Description:
  Need a text field on Account object to store customer email.
  Field name: Customer_Email__c
  Field label: Customer Email
  Field length: 255
  Required: No

  Labels: salesforce
  ```
- [ ] **6.3** Click "Create"
- [ ] **6.4** Wait 1-2 minutes for Workflow 1 to trigger
- [ ] **6.5** Go to n8n → Executions
- [ ] **6.6** Filter by "SF Agent 1 - Initial Analysis"
- [ ] **6.7** Check execution:
  - Jira Trigger: ✅ Shows issue data
  - Extract Issue Details: ✅ Shows parsed fields
  - Claude - Analyze: ✅ Shows JSON response
  - Parse AI Response: ✅ Shows `readyToExecute: true`
- [ ] **6.8** Check Jira issue:
  - Should have comment: "Ready to execute Salesforce configuration"
  - Should have label: `sf-agent-ready-to-execute`
- [ ] **6.9** Wait 1-2 minutes for Workflow 3 to trigger
- [ ] **6.10** Check Workflow 3 execution:
  - Prepare Tooling API Request: ✅ Shows API request format
  - Deploy via Tooling API: ✅ Shows Salesforce response
  - Check Deployment Result: ✅ Shows `validationPassed: true`
- [ ] **6.11** Check Jira issue:
  - Should have success comment with deployment details
  - Should have label: `sf-agent-completed`
  - Should be marked "Done"
- [ ] **6.12** Verify in Salesforce:
  - Go to Setup → Object Manager → Account → Fields & Relationships
  - Search for: `Customer_Email__c`
  - Field should exist ✅

**Expected Result**: Entire flow completes automatically, field created in Salesforce, issue marked Done

---

#### Test 2: Validation Rule (With Questions)

- [ ] **6.13** Create new Story in Jira:
  ```
  Summary: Validate Account name length
  Description:
  Create validation rule to ensure Account name is at least 3 characters.
  Object: Account

  Labels: salesforce
  ```
- [ ] **6.14** Wait for Workflow 1 to trigger
- [ ] **6.15** Check Jira issue:
  - Should have comment with questions (e.g., "What error message should be displayed?")
  - Should have label: `sf-agent-awaiting-response`
- [ ] **6.16** Add comment with answers:
  ```
  Rule name: Name_Min_Length
  Formula: LEN(Name) < 3
  Error message: Account name must be at least 3 characters
  Display on field: Name
  ```
- [ ] **6.17** Wait for Workflow 2 to trigger (webhook, should be instant)
- [ ] **6.18** Check Workflow 2 execution:
  - Extract Comments: ✅ Shows user answers
  - Claude - Re-analyze: ✅ Shows `allQuestionsAnswered: true`
- [ ] **6.19** Check Jira issue:
  - Should have comment: "All questions answered, proceeding to execute"
  - Label changed to: `sf-agent-ready-to-execute`
- [ ] **6.20** Wait for Workflow 3 to trigger
- [ ] **6.21** Check Workflow 3 execution and Jira issue (same as Test 1)
- [ ] **6.22** Verify in Salesforce:
  - Go to Setup → Object Manager → Account → Validation Rules
  - Find: `Name_Min_Length`
  - Rule should exist ✅

**Expected Result**: Agent asks questions, user answers, agent completes configuration, issue marked Done

---

## Troubleshooting Reference

If you encounter issues, refer to:

- **Detailed credential setup**: [CREDENTIAL_CONFIGURATION_GUIDE.md](CREDENTIAL_CONFIGURATION_GUIDE.md:1)
- **Troubleshooting section**: [CREDENTIAL_CONFIGURATION_GUIDE.md](CREDENTIAL_CONFIGURATION_GUIDE.md:439)
- **Implementation details**: [TOOLING_API_IMPLEMENTATION.md](TOOLING_API_IMPLEMENTATION.md:1)
- **Complete setup guide**: [SF_AGENT_SETUP_GUIDE.md](SF_AGENT_SETUP_GUIDE.md:1)

### Common Quick Fixes

**Workflow 1 doesn't trigger**:
- Verify workflow is Active
- Check Jira Trigger JQL filter: `type = Story AND labels = salesforce`
- Check polling interval (default: 1 minute)

**Claude API error**:
- Verify Anthropic API key is correct
- Check billing is set up in Anthropic console
- Ensure API key is linked to HTTP Request node authentication

**Salesforce deployment fails**:
- Re-authorize Salesforce OAuth2 credential
- Check Connected App scopes include `api` and `full`
- Verify IP Relaxation is enabled

**Webhook doesn't fire**:
- Verify webhook URL exactly matches n8n Production URL
- Check webhook is Enabled in Jira
- Try removing JQL filter temporarily

---

## Success Criteria

You've successfully configured the agent when:

- ✅ Test Story created with label `salesforce` triggers Workflow 1
- ✅ Workflow 1 analyzes requirement and posts comment in Jira
- ✅ Workflow 3 deploys to Salesforce via Tooling API
- ✅ Field/validation rule appears in Salesforce
- ✅ Success comment posted in Jira
- ✅ Issue automatically marked "Done"

---

## Next Steps After Setup

Once all tests pass:

1. **Add more test scenarios**:
   - Different field types (Picklist, Lookup, etc.)
   - Complex validation rules
   - Multiple fields at once

2. **Refine AI prompts**:
   - Based on real-world usage
   - Improve question quality
   - Add more metadata types

3. **Monitor performance**:
   - Track execution times
   - Monitor API usage
   - Review error rates

4. **Scale up**:
   - Add more configuration types (Flows, Workflows)
   - Support multiple Salesforce orgs
   - Integrate with Slack for notifications

---

## Contact & Support

- **Documentation**: See all `.md` files in `/Users/pratappattanayak/Documents/n8nClaude/`
- **Workflow IDs**:
  - Workflow 1: `2A6XqlHc9b48hjd6`
  - Workflow 2: `Xa6YL7mUW7KB9efA`
  - Workflow 3: `QjoWn0koYUvNNMC1`
- **n8n Instance**: https://pratapkp.app.n8n.cloud

---

**Ready to start!** Begin with Part 1: Salesforce OAuth2 Setup above.
