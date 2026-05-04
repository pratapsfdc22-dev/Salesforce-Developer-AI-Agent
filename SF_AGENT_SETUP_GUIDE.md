# Autonomous AI Salesforce Developer Agent - Setup Guide

## 🎉 Workflows Created Successfully!

Three n8n workflows have been created in your instance at `https://<your-n8n-instance>.app.n8n.cloud`

### Workflow IDs
1. **SF Agent 1 - Initial Analysis**: `2A6XqlHc9b48hjd6`
2. **SF Agent 2 - Answer Processing**: `Xa6YL7mUW7KB9efA`
3. **SF Agent 3 - Execute Changes**: `QjoWn0koYUvNNMC1`

## 📋 Setup Checklist

### Step 1: Configure Credentials in n8n

#### 1.1 Jira API Credentials
1. Go to n8n: https://<your-n8n-instance>.app.n8n.cloud
2. Navigate to **Settings** → **Credentials**
3. Click **Add Credential** → Search for **Jira Software**
4. Configure:
   - **Credential Name**: `jira-credential` (or any name)
   - **Email**: `your-email@example.com`
   - **API Token**: `YOUR_JIRA_API_TOKEN_HERE` (Get from https://id.atlassian.com/manage-profile/security/api-tokens)
   - **Domain**: `<your-domain>.atlassian.net`
5. Test and save

#### 1.2 Anthropic API Credentials
1. In n8n, go to **Settings** → **Credentials**
2. Click **Add Credential** → Search for **Anthropic**
3. Configure:
   - **Credential Name**: `anthropic-main` (or any name)
   - **API Key**: Your Anthropic API key
   - If you don't have one, get it from: https://console.anthropic.com/settings/keys
4. Test and save

#### 1.3 Update Workflow Credentials
For each workflow (1, 2, and 3):
1. Open the workflow in n8n
2. Click on each Jira node and select the `jira-credential` credential
3. Click on each Claude/Anthropic HTTP node and select the `anthropic-main` credential
4. Save the workflow

### Step 2: Set Up Jira Webhook

#### 2.1 Get Webhook URL from n8n
1. Open **Workflow 2 - Answer Processing** in n8n
2. Click on the **"Jira Webhook - Comment Added"** node
3. Copy the **Production URL** (should be: `https://<your-n8n-instance>.app.n8n.cloud/webhook/sf-agent-comment`)

#### 2.2 Configure Webhook in Jira
1. Go to Jira: https://<your-domain>.atlassian.net
2. Click ⚙️ **Settings** (top right) → **System**
3. In left sidebar, click **WebHooks**
4. Click **Create a WebHook**
5. Configure:
   - **Name**: `n8n SF Agent Comment Webhook`
   - **Status**: Enabled
   - **URL**: `https://<your-n8n-instance>.app.n8n.cloud/webhook/sf-agent-comment`
   - **Description**: Triggers AI agent when comments are added to Salesforce stories
   - **Events**: Check **Comment** → **created**
   - **JQL Filter** (optional but recommended): `labels = sf-agent-awaiting-response`
6. Click **Create**

#### 2.3 Get Webhook URL for Workflow 3
1. Open **Workflow 3 - Execute Changes** in n8n
2. Click on the **"Webhook - Execute Trigger"** node
3. Copy the **Production URL** (should be: `https://<your-n8n-instance>.app.n8n.cloud/webhook/sf-agent-execute`)
4. Note: This webhook is called programmatically by Workflow 2, no manual setup needed

### Step 3: Activate Workflows

1. Open each workflow in n8n
2. Click the toggle switch in the top right to **Activate**
3. Ensure all 3 workflows show "Active" status:
   - ✅ SF Agent 1 - Initial Analysis
   - ✅ SF Agent 2 - Answer Processing
   - ✅ SF Agent 3 - Execute Changes

### Step 4: Verify Salesforce CLI

Ensure Salesforce CLI is authenticated and ready:

```bash
# Check SF CLI version
sf --version

# List authenticated orgs
sf org list

# Verify target org
sf org display --target-org your-email@example.com
```

Expected output should show:
- **Org ID**: <YOUR_ORG_ID>
- **Instance URL**: https://<your-org>.my.salesforce.com
- **Status**: Connected

## 🧪 Testing the Agent

### Test 1: Simple Custom Field (No Questions)

1. Go to Jira: https://<your-domain>.atlassian.net
2. Create a new **Story**:
   - **Project**: Any project you have access to
   - **Issue Type**: Story
   - **Summary**: Add Annual Revenue field to Account
   - **Description**:
     ```
     Need a currency field on Account object to track annual revenue.
     Field name: Annual_Revenue__c
     Field label: Annual Revenue
     Field type: Currency
     Decimal places: 2
     Default value: 0
     ```
   - **Labels**: Add label `salesforce`
3. Click **Create**

**Expected Behavior:**
1. Within 1-2 minutes, Workflow 1 triggers
2. AI analyzes the requirement
3. Since all details are provided, no questions are needed
4. AI posts comment: "Ready to execute"
5. Workflow 3 automatically triggers
6. Salesforce field is created
7. Success comment posted with verification link
8. Issue automatically marked as **Done**

### Test 2: Validation Rule (With Questions)

1. Create a new **Story** in Jira:
   - **Summary**: Prevent duplicate Contacts based on email
   - **Description**:
     ```
     Create a validation rule that prevents users from saving duplicate Contact records.
     Should check email address for duplicates.
     ```
   - **Labels**: Add label `salesforce`
2. Click **Create**

**Expected Behavior:**
1. Workflow 1 triggers and AI analyzes
2. AI identifies missing information and posts questions:
   - "Should this check across all Contacts or only within the same Account?"
   - "What error message should users see?"
   - "Should this apply to all users or only certain profiles?"
3. You see a comment with label `sf-agent-awaiting-response`
4. **You respond** in a comment with answers:
   ```
   1. Check across all Contacts globally
   2. Error message: "A Contact with this email already exists"
   3. Apply to all users
   ```
5. Within seconds (webhook triggers), Workflow 2 processes answers
6. AI confirms all questions answered
7. Workflow 3 automatically triggers
8. Validation rule is created in Salesforce
9. Issue marked as **Done**

### Test 3: Invalid Request

1. Create a **Story** with:
   - **Summary**: Make the system faster
   - **Description**: I want Salesforce to load faster
   - **Labels**: Add label `salesforce`

**Expected Behavior:**
1. AI analyzes and determines this is NOT a configuration request
2. Posts comment: "This doesn't appear to be a valid Salesforce configuration request"
3. Adds label `sf-agent-invalid`
4. Stops execution

## 📊 Monitoring & Troubleshooting

### View Workflow Executions

1. Go to n8n: https://<your-n8n-instance>.app.n8n.cloud
2. Click **Executions** in left sidebar
3. Filter by workflow name
4. Click on any execution to see:
   - Each node's input/output
   - Success/failure status
   - Error messages (if any)

### Common Issues & Solutions

#### Issue: Workflow doesn't trigger
**Solutions:**
- Verify workflow is **Active** (toggle on)
- Check Jira issue has label `salesforce`
- Check Jira issue type is **Story**
- Check Jira Trigger node JQL filter

#### Issue: "Authentication failed" error
**Solutions:**
- Verify Jira API token is correct
- Verify Anthropic API key is valid
- Check credential names match in workflow nodes

#### Issue: "Salesforce CLI command not found"
**Solutions:**
- Ensure SF CLI is installed on the n8n server
- Run: `sf --version` to verify
- Check that n8n has permission to run bash commands

#### Issue: Webhook not firing (Workflow 2)
**Solutions:**
- Verify webhook is configured in Jira settings
- Check webhook URL matches Production URL from n8n
- Test webhook manually in Jira (Settings → WebHooks → Edit → Test)
- Check JQL filter doesn't exclude your test issues

#### Issue: Deployment fails
**Solutions:**
- Check Salesforce CLI authentication: `sf org list`
- Verify target org in deployment command
- Review deployment logs in Jira comment
- Check Salesforce metadata API is accessible

### Jira Label States

| Label | Meaning | Action |
|-------|---------|--------|
| `salesforce` | SF-related issue | Triggers Workflow 1 |
| `sf-agent-awaiting-response` | Waiting for answers | Add comment to trigger Workflow 2 |
| `sf-agent-ready-to-execute` | Ready to deploy | Automatic - Workflow 3 triggers |
| `sf-agent-in-progress` | Currently deploying | Wait - deployment in progress |
| `sf-agent-completed` | Success! | Issue is Done |
| `sf-agent-error` | Error occurred | Check comments for details |
| `sf-agent-invalid` | Not a valid request | Clarify requirement |

## 🔐 Security Notes

### Credentials Storage
- All credentials are stored securely in n8n's credential vault
- API tokens are never logged or exposed in workflow outputs
- Jira comments do not contain sensitive information

### Salesforce Access
- Uses existing authenticated Salesforce CLI
- Target org: `your-email@example.com`
- Org: <your-org>.my.salesforce.com
- Deployment uses `--dry-run` validation before actual deployment

### Webhook Security
- Webhook URLs are unique and non-guessable
- Consider adding Jira IP whitelist for production use
- Webhooks validate issue labels before processing

## 📚 What Each Workflow Does

### Workflow 1: SF Agent - Initial Analysis
**Trigger**: New Jira Story with label "salesforce"
**Process**:
1. Extract issue details
2. Analyze requirement with Claude AI
3. Determine if valid SF configuration request
4. Identify what needs to be configured
5. Ask clarifying questions if needed OR proceed directly

**Nodes**: 9 nodes
- Jira Trigger
- Extract Issue Details (Code)
- Claude AI Analysis (HTTP → Anthropic API)
- Parse AI Response (Code)
- IF node (has questions?)
- Jira comment posting
- Label management

### Workflow 2: SF Agent - Answer Processing
**Trigger**: Jira comment added (via webhook)
**Process**:
1. Verify issue is awaiting response
2. Extract all comments
3. Separate AI questions from user answers
4. Re-analyze with Claude AI
5. Determine if more questions needed OR ready to execute
6. Trigger Workflow 3 if ready

**Nodes**: 12 nodes
- Webhook trigger
- Jira issue fetching
- Comment extraction (Code)
- Claude AI re-analysis (HTTP)
- IF node (more questions?)
- Label updates
- HTTP request to trigger Workflow 3

### Workflow 3: SF Agent - Execute Changes
**Trigger**: HTTP webhook from Workflow 2
**Process**:
1. Verify issue is ready to execute
2. Generate Salesforce metadata with Claude AI
3. Create temp directory structure
4. Write metadata files
5. Validate with Salesforce CLI (dry-run)
6. Deploy to Salesforce
7. Verify deployment
8. Post success comment
9. Mark issue as Done

**Nodes**: 18 nodes
- Webhook trigger
- Jira issue verification
- Claude AI metadata generation (HTTP)
- Multiple Bash nodes for CLI operations
- Validation checks
- Success/error handling
- Issue transition to Done

## 🎯 Next Steps

1. ✅ Complete credential setup in n8n
2. ✅ Configure Jira webhook
3. ✅ Activate all 3 workflows
4. ✅ Create test Jira Story
5. ✅ Monitor first execution
6. ✅ Verify Salesforce changes
7. ✅ Refine AI prompts based on results
8. ✅ Add more test scenarios

## 🚀 Future Enhancements

### Phase 2 Features (Optional)
- **Multi-org support**: Deploy to multiple Salesforce orgs
- **Rollback capability**: Undo deployments if issues found
- **Approval workflow**: Optional human approval before deployment
- **Slack notifications**: Real-time updates in Slack
- **Impact analysis**: Analyze impact on existing configurations
- **Test generation**: Auto-generate Apex tests
- **Change sets**: Create change sets for production
- **Metrics dashboard**: Track agent performance

### Advanced Configuration Types
- **Flow Builder**: More complex Flow configurations
- **Permission Sets**: Create and manage permission sets
- **Custom Objects**: Full custom object creation
- **Lightning Pages**: Page layout automation
- **Reports & Dashboards**: Automated reporting setup

## 📞 Support

### Resources
- **Plan File**: `/Users/<username>/.claude/plans/lively-watching-tulip.md`
- **CLAUDE.md**: `/Users/<username>/Documents/n8nClaude/CLAUDE.md`
- **Jira Context**: `/Users/<username>/Documents/n8nClaude/JIRA_CONTEXT.md`
- **Salesforce Context**: `/Users/<username>/Documents/n8nClaude/SALESFORCE_CONTEXT.md`

### Getting Help
- Check workflow execution logs in n8n
- Review Jira comments for AI agent messages
- Verify Salesforce deployment logs
- Check this guide's troubleshooting section

---

**🎊 Congratulations! Your Autonomous AI Salesforce Developer Agent is ready to use!**

Just create a Jira Story with the label `salesforce`, and watch the AI agent handle the entire configuration lifecycle automatically.
