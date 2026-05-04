# n8n Workflow Assistant Project

## Project Purpose

This project enables Claude to act as your intelligent n8n workflow builder. My role is to help you create high-quality, production-ready workflows in your n8n instance by leveraging:

1. **n8n MCP Server** - Full n8n instance management (create, read, update workflows, manage credentials, monitor executions)
2. **n8n Skills** - Specialized capabilities for n8n workflow operations (documentation TBD)
3. **Salesforce Skills** - Integration capabilities for Salesforce-related workflows (documentation TBD)

## Workflow Building Approach

### Rapid Prototyping Philosophy
- **Get it working first**: Build a functional workflow quickly
- **Iterate based on feedback**: Refine and enhance after seeing it in action
- **Keep it simple**: Avoid over-engineering; add complexity only when needed
- **Quick validation**: Test early and often during development

### My Workflow Development Process
1. **Understand Requirements**: Ask clarifying questions about your goals
2. **Design Node Structure**: Plan the workflow architecture
3. **Create Workflow**: Use n8n MCP server to build the workflow
4. **Test & Validate**: Verify the workflow executes correctly
5. **Iterate**: Refine based on your feedback and real-world usage

## n8n Workflow Patterns & Best Practices

### Common Node Types

**Trigger Nodes**
- Schedule Trigger: Time-based workflow execution
- Webhook: HTTP endpoint for external systems
- Manual Trigger: On-demand execution

**Data Processing Nodes**
- Code (JavaScript): Custom data transformations and logic
- HTTP Request: Call external APIs
- IF/Switch: Conditional branching logic

**Integration Nodes**
- Service-specific nodes (LinkedIn, Salesforce, Slack, etc.)
- Credentials managed per node

**AI/LangChain Nodes**
- LLM Chat Models (OpenAI, Anthropic, etc.)
- Chain LLM: Structured prompts and AI workflows
- Vector stores and embeddings

### Data Flow Best Practices

**Node Connections**
- Data flows through `main` connections between nodes
- Each node receives items from previous nodes via `$json`
- Access specific node outputs: `$items("Node Name", runIndex)`

**Code Node Patterns**
```javascript
// Single item transformation
const data = $json;
return [{ json: { result: data.field } }];

// Multiple items output
const items = $input.all();
return items.map(item => ({
  json: { processed: item.json.value }
}));

// Accessing other node data
const previous = $items("Previous Node", 0);
```

**Error Handling**
- Use `onError: "continueRegularOutput"` for HTTP nodes
- Implement retry logic with `retryOnFail: true`
- Add validation nodes before critical operations
- Use IF nodes to route errors vs success cases

### Workflow Structure Guidelines

**Naming Conventions**
- Use descriptive node names: "Validate User Email" not "Code"
- Group related nodes visually
- Add Notes nodes for complex logic explanation

**Maintainability**
- Keep code nodes focused (one purpose per node)
- Extract repeated logic into reusable patterns
- Document complex transformations in code comments
- Use meaningful variable names

## Common Workflow Patterns

### 1. Data Integration & Automation
```
Trigger → Fetch Data (HTTP/API) → Transform (Code) → Validate → Store/Send → Error Handler
```
**Use Cases**: Syncing data between systems, scheduled data exports, batch processing

### 2. Sales & Marketing Automation
```
Trigger → Fetch Leads → Score/Filter → Update CRM → Send Notification → Track Results
```
**Use Cases**: Lead routing, email campaigns, social media posting, CRM updates

### 3. AI-Powered Workflows
```
Trigger → Prepare Data → LLM Chain → Validate Output → Post-Process → Publish/Store
```
**Use Cases**: Content generation, data analysis, intelligent routing, sentiment analysis

**Example Pattern** (from your LinkedIn workflow):
1. Schedule Trigger
2. Generate content with LLM
3. Extract and validate URLs
4. Repair invalid sources (with retry logic)
5. Assemble final output
6. Post to platform

### 4. Validation & Repair Pattern
```
Process Data → Validate → IF(valid) → Success Path
                              ↓(invalid)
                         Repair with LLM → Re-validate → IF(valid) → Success Path
                                                              ↓(still invalid)
                                                         Fallback Handler
```

## Quality Standards

### Workflow Quality Checklist
- [ ] Clear, descriptive node names
- [ ] Error handling for external API calls
- [ ] Data validation before critical operations
- [ ] Appropriate retry logic for transient failures
- [ ] Meaningful error messages
- [ ] Testing with sample data

### Code Node Quality
- [ ] Handles edge cases (empty data, missing fields)
- [ ] Returns data in correct format: `[{ json: {...} }]`
- [ ] Includes error checking for external dependencies
- [ ] Uses clear variable names
- [ ] Comments for complex logic

### Testing & Validation
- Test with manual trigger first
- Validate with realistic sample data
- Check error paths (not just happy path)
- Monitor first few scheduled executions
- Verify credential access and permissions

## Working Together

### How I'll Communicate
- **Progress updates**: Quick summaries of what I'm building
- **Key decisions**: Explain important architectural choices
- **Questions**: Ask when requirements are unclear
- **Workflow structure**: Share node layout and data flow
- **Testing results**: Report on validation and any issues found

### When I'll Ask Questions
- Unclear requirements or ambiguous goals
- Multiple valid approaches (need your preference)
- Credential/API access details
- Expected data formats or validation rules
- Error handling preferences

### What I Need From You
- Clear description of workflow goals
- Sample input/output data (if available)
- Credential information (when needed)
- Preferred external services/APIs
- Feedback on iterations

## Tool Usage Guidelines

### n8n MCP Server
**Repository**: https://github.com/czlonkowski/n8n-mcp

The n8n-MCP server is a Model Context Protocol server that bridges n8n workflow automation with Claude. It provides structured access to n8n's complete node ecosystem.

**Capabilities:**
- Access to **1,084 n8n nodes** (537 core + 547 community nodes)
- **Node documentation** with 87% coverage from official sources
- **Node properties** with 99% schema coverage
- **2,646 pre-extracted workflow configurations** from popular templates
- **2,709 workflow templates** with complete metadata
- **265 AI-capable tool variants** with full documentation

**Key Operations:**
- Search and retrieve node documentation
- Validate node properties and configurations
- Explore workflow templates
- Discover community nodes (with `source` filter)
- Create and manage workflows (with API credentials)
- Execute workflows

**Critical Configuration:**
```json
{
  "MCP_MODE": "stdio",
  "N8N_API_URL": "your-n8n-instance-url",
  "N8N_API_KEY": "your-n8n-api-key"
}
```

**SAFETY WARNING**: Never edit production workflows directly with AI. Always make copies, test in development, export backups, and validate changes before deploying to production.

**Installation Options:**
1. Hosted service at dashboard.n8n-mcp.com (free tier: 100 tool calls/day)
2. npx quick setup: `npx n8n-mcp`
3. Docker: `docker pull ghcr.io/czlonkowski/n8n-mcp:latest`
4. Local development: Clone and build from repository
5. Railway cloud deployment (one-click)

### n8n Skills
**Repository**: https://github.com/czlonkowski/n8n-skills

The n8n-skills are seven complementary Claude Code skills that teach me how to construct production-grade n8n workflows. These skills work synergistically with the n8n-mcp server.

**The Seven Skills:**

1. **n8n Expression Syntax**
   - Correct expression patterns and variable access
   - Using `$json`, `$node`, `$now`, `$env`
   - Critical: Webhook data is under `$json.body`
   - Common mistakes and corrections

2. **n8n MCP Tools Expert** (Highest Priority)
   - Proper MCP tool usage
   - NodeType formatting
   - Validation profiles
   - Parameter configuration

3. **n8n Workflow Patterns**
   - Five architectural approaches:
     - Webhook processing
     - HTTP API integration
     - Database operations
     - AI workflows
     - Scheduled tasks
   - Examples from 2,653+ templates

4. **n8n Validation Expert**
   - Interpret and resolve validation errors
   - Auto-sanitization behavior
   - Distinguish genuine issues from false positives

5. **n8n Node Configuration**
   - Operation-specific guidance
   - Property dependencies
   - AI connection types for agent workflows

6. **n8n Code JavaScript**
   - JavaScript implementation in Code nodes
   - Data access patterns
   - Helper functions
   - Common error solutions

7. **n8n Code Python**
   - Python implementation in Code nodes
   - Library limitations
   - Standard library usage

**Installation:**
```bash
# Claude Code Plugin
/plugin install czlonkowski/n8n-skills

# Manual Setup
# Clone repository and copy skill folders to ~/.claude/skills/
```

**Prerequisites:**
- n8n-mcp MCP server installed and configured
- `.mcp.json` configured with n8n-mcp connection

**How Skills Activate:**
Skills activate automatically based on query context. When you request complex workflows, multiple skills coordinate to provide comprehensive guidance.

### Salesforce Skills
**Org:** <your-org>.my.salesforce.com (your-email@example.com)

Salesforce Skills provide comprehensive assistance for both Salesforce development and n8n workflow integration with Salesforce.

#### 1. Salesforce Development Assistance

**Apex Code Generation**
- Trigger handler pattern implementation
- Service class patterns (business logic)
- Batch and Queueable Apex
- SOQL query optimization
- Test class generation (85%+ coverage goal)
- Security-first development (with sharing, CRUD/FLS checks)

**Lightning Web Components**
- Component structure (HTML, JS, CSS, XML)
- Calling Apex from LWC (@wire and imperative)
- Component communication patterns
- Data tables and forms
- Navigation and error handling
- Lightning Message Service

**Best Practices**
- Bulkification (handle 200+ records)
- Governor limit compliance
- SOQL injection prevention
- Proper error handling
- Security validation
- Comprehensive testing

#### 2. n8n Salesforce Integration

**Available n8n Salesforce Nodes:**

1. **Salesforce Node** (`nodes-base.salesforce`)
   - **Resources**: Account, Attachment, Case, Contact, Custom Object, Document, Flow, Lead, Opportunity, Search, Task, User (12 object types)
   - **Operations**: Create, Read, Update, Delete, Upsert, Add to Campaign, Add Note, Get Summary
   - **Authentication**: OAuth2 or OAuth2 JWT
   - **Use Cases**: Standard CRUD operations on Salesforce objects

2. **Salesforce Trigger** (`nodes-base.salesforceTrigger`)
   - **Type**: Polling trigger
   - **Purpose**: Fetches data from Salesforce on specified intervals
   - **Use Cases**: Monitor Salesforce for new/updated records and trigger workflows

3. **Salesforce Tool** (`nodes-base.salesforceTool`)
   - **Type**: AI Tool variant for AI Agents
   - **Purpose**: Salesforce operations within LangChain workflows
   - **Use Cases**: AI agents that need to query or update Salesforce data

#### 3. Common Integration Patterns

**Pattern 1: Lead Routing Workflow**
```
Webhook → Code (validate) → IF (valid) → Salesforce (create Lead) → Email notification
```
**Use Case**: Capture leads from external forms and route to Salesforce

**Pattern 2: Opportunity Sync**
```
Schedule Trigger → Salesforce Trigger (get opportunities) → Code (transform) → HTTP Request (external API)
```
**Use Case**: Scheduled sync of Salesforce opportunities to external systems

**Pattern 3: AI Lead Qualification**
```
Salesforce Trigger (new leads) → OpenAI Chat Model → Code (parse score) → Salesforce (update lead status)
```
**Use Case**: Automatically qualify and score leads using AI

**Pattern 4: Data Validation Loop**
```
Webhook → Salesforce (get account) → Code (validate) → IF (invalid) → Salesforce (create task) → Slack notification
```
**Use Case**: Validate incoming data against Salesforce records

#### 4. Salesforce CLI Status

**Current Authentication:**
- ✅ Authenticated to <your-org>.my.salesforce.com
- ✅ User: your-email@example.com
- ✅ Org ID: (run `sf org display` to retrieve)
- ✅ API Version: 64.0

**Quick Commands:**
```bash
sf org display          # Show org info
sf org list            # List all orgs
sf org open            # Open org in browser
sf project deploy start --manifest manifest/package.xml
sf apex run test --test-level RunLocalTests
```

#### 5. Development Workflow

**For Apex/LWC Development:**
1. Write code following patterns in `APEX_PATTERNS.md` or `LWC_PATTERNS.md`
2. Include comprehensive test coverage (aim for 100%)
3. Review against `SECURITY_CHECKLIST.md`
4. Deploy to sandbox first using Salesforce CLI
5. Test thoroughly with different user profiles
6. Deploy to production after validation

**For n8n Workflows:**
1. Design workflow using available Salesforce nodes
2. Configure OAuth2 authentication
3. Test with sample data
4. Implement error handling
5. Monitor execution and logs
6. Iterate based on real-world usage

#### 6. Reference Files

All Salesforce reference materials are in the n8nClaude project:

- **[.clauderc](.clauderc)**: Claude configuration for Salesforce
- **[SALESFORCE_CONTEXT.md](SALESFORCE_CONTEXT.md)**: Org details and standards
- **[APEX_PATTERNS.md](APEX_PATTERNS.md)**: Apex code templates
- **[LWC_PATTERNS.md](LWC_PATTERNS.md)**: Lightning Web Component patterns
- **[SECURITY_CHECKLIST.md](SECURITY_CHECKLIST.md)**: Pre-deployment security verification
- **[CLAUDE_CODE_README.md](CLAUDE_CODE_README.md)**: Quick reference guide

#### 7. Example Use Cases

**Salesforce Development:**
- "Create a trigger on Account that updates related Contacts when billing address changes"
- "Build an LWC that displays Opportunities for an Account with search and filter"
- "Write a batch job that marks inactive Accounts based on last activity date"

**n8n Integration:**
- "Create a workflow that syncs new Salesforce Leads to HubSpot every hour"
- "Build a workflow that validates incoming webhooks and creates Salesforce Cases"
- "Set up an AI workflow that scores Leads using OpenAI and updates Salesforce"

#### 8. Security & Best Practices

**Always Follow:**
- ✅ Use `with sharing` in Apex classes
- ✅ Check CRUD/FLS before DML operations
- ✅ Prevent SOQL injection with bind variables
- ✅ Bulkify all operations (200+ records)
- ✅ Include comprehensive test coverage
- ✅ Never hardcode credentials or IDs
- ✅ Validate all external input
- ✅ Test with different user profiles
- ✅ Deploy to sandbox first
- ✅ Review against security checklist

For complete security guidelines, see [SECURITY_CHECKLIST.md](SECURITY_CHECKLIST.md).

### Jira Skills
**Instance:** https://<your-domain>.atlassian.net (your-email@example.com)

Jira Skills provide comprehensive assistance for Jira integration with n8n workflows, issue management automation, and cross-system synchronization.

#### 1. Jira Integration Capabilities

**n8n Jira Nodes:**

1. **Jira Software Node** (`nodes-base.jira`)
   - **Resources**: Issue, Issue Attachment, Issue Comment, User
   - **Operations**:
     - Issue: Create, Update, Delete, Get, Get Many, Changelog, Notify, Status
     - Attachment: Add, Get, Get Many, Remove
     - Comment: Add, Get, Get Many, Remove, Update
     - User: Get, Get Many
   - **Authentication**: API Token (Cloud), Basic Auth (Server)
   - **Use Cases**: Standard Jira CRUD operations

2. **Jira Trigger** (`nodes-base.jiraTrigger`)
   - **Type**: Polling trigger
   - **Purpose**: Monitors Jira for events (new issues, updates, comments)
   - **Polling Interval**: Configurable (1-60 minutes)
   - **Use Cases**: Trigger workflows on Jira changes

3. **Jira Tool** (`nodes-base.jiraTool`)
   - **Type**: AI Tool variant for AI Agents
   - **Purpose**: Jira operations within LangChain workflows
   - **Use Cases**: AI agents that query or update Jira

#### 2. Common Integration Patterns

**Pattern 1: Webhook to Jira Issue**
```
Webhook → Code (validate) → Jira (create Issue) → Slack (notify)
```
**Use Case**: Create Jira issues from external forms, apps, or APIs

**Pattern 2: Salesforce-Jira Sync**
```
Salesforce Trigger → Code (transform) → Jira (create Issue) → Salesforce (update with Jira key)
```
**Use Case**: Bidirectional sync between Salesforce Cases and Jira Issues

**Pattern 3: AI-Powered Issue Triage**
```
Jira Trigger (new issue) → OpenAI (analyze) → Jira (update priority & labels)
```
**Use Case**: Automatically categorize and prioritize issues using AI

**Pattern 4: Issue Status Sync**
```
Jira Trigger (status change) → IF (resolved) → Salesforce (update Case) → Slack (notify)
```
**Use Case**: Keep external systems in sync with Jira status

**Pattern 5: Bulk Issue Creation**
```
Schedule → Database/CSV (read) → Loop → Jira (create Issues) → Summary Email
```
**Use Case**: Create multiple issues from spreadsheets or databases

#### 3. Jira Query Language (JQL)

**Common JQL Examples:**
```jql
# Open issues in project
project = MYPROJECT AND status = Open

# My assigned issues
assignee = currentUser() AND status != Done

# Recently created
created >= -7d

# High priority bugs
priority = High AND type = Bug
```

#### 4. Authentication Setup

**Credentials Configuration:**
- **Instance URL**: https://<your-domain>.atlassian.net
- **Email**: your-email@example.com
- **API Token**: Configured ✅
- **Auth Type**: API Token (for Jira Cloud)

**How to Generate API Token:**
1. Go to https://id.atlassian.com/manage-profile/security/api-tokens
2. Click "Create API token"
3. Copy and store in n8n credentials

#### 5. Required Fields for Issue Creation

**Minimum Required:**
- **project**: Project key (e.g., "PROJ")
- **issuetype**: Issue type (e.g., "Task", "Bug", "Story")
- **summary**: Brief title (50-100 characters)

**Common Optional Fields:**
- **description**: Detailed description (supports Jira markdown)
- **priority**: High/Medium/Low
- **assignee**: User account ID
- **labels**: Array of labels
- **duedate**: YYYY-MM-DD format
- **customfield_***: Project-specific custom fields

#### 6. Best Practices

**Issue Management:**
- ✅ Validate input data before creating issues
- ✅ Use descriptive summaries (50-100 chars)
- ✅ Include context in descriptions
- ✅ Set appropriate priority levels
- ✅ Add labels for categorization
- ✅ Link related issues when applicable

**Error Handling:**
- ✅ Check for duplicate issues before creation
- ✅ Validate project and issue type exist
- ✅ Handle authentication failures gracefully
- ✅ Log failed operations for debugging
- ✅ Implement retry logic for transient failures

**Performance:**
- ✅ Use bulk operations when possible
- ✅ Implement pagination for large result sets
- ✅ Cache project/issue type data
- ✅ Minimize API calls with efficient JQL
- ✅ Respect rate limits (~10 req/sec for Cloud)

#### 7. Reference Files

All Jira reference materials are in the n8nClaude project:

- **[JIRA_CONTEXT.md](JIRA_CONTEXT.md)**: Instance details, authentication, operations, and troubleshooting
- **[JIRA_PATTERNS.md](JIRA_PATTERNS.md)**: 9 common workflow patterns with code examples

#### 8. Example Use Cases

**Jira Automation:**
- "Create a workflow that creates Jira issues from form submissions"
- "Build a workflow that syncs Jira issue status to Salesforce Cases"
- "Set up AI-powered issue categorization and prioritization"
- "Create a workflow that monitors overdue issues and sends escalations"

**Cross-System Integration:**
- "Sync Salesforce Cases to Jira and keep them updated bidirectionally"
- "Create Jira issues when high-priority support tickets arrive"
- "Build a workflow that links related issues across multiple Jira projects"

#### 9. Security & Limits

**API Security:**
- ✅ Never hardcode API tokens
- ✅ Use n8n credentials manager
- ✅ Rotate tokens periodically
- ✅ Validate all external input
- ✅ Sanitize HTML/markdown in descriptions

**Rate Limits:**
- **Cloud**: ~10 requests per second per IP
- **Search Results**: Max 1000 issues per query (use pagination)
- **Attachment Size**: Max 10MB per file (Cloud)

#### 10. Troubleshooting

**Common Issues:**
1. **Authentication Failed** → Verify email and API token
2. **Project Not Found** → Check project key and permissions
3. **Invalid Issue Type** → Ensure type exists in project
4. **Field Required Error** → Check for custom required fields
5. **Rate Limit Exceeded** → Implement exponential backoff

For detailed troubleshooting and patterns, see [JIRA_CONTEXT.md](JIRA_CONTEXT.md).

## Example Workflows You Can Request

### Data Integration
"Create a workflow that syncs Salesforce leads to our CRM every hour"

### Marketing Automation
"Build a workflow that generates LinkedIn posts using AI, validates sources, and publishes them"

### Business Process
"Set up a workflow that monitors form submissions, enriches the data, and creates Salesforce opportunities"

### AI-Powered
"Create a workflow that analyzes customer feedback, categorizes it, and sends summaries to Slack"

## Critical n8n Gotchas & Best Practices

### Webhook Data Access
**Critical**: Webhook payloads require `.body` accessor:
```javascript
// CORRECT
const data = $json.body.fieldName;

// INCORRECT
const data = $json.fieldName;
```
This applies across expressions and JavaScript implementations.

### Expression Syntax
- Use `{{ }}` for expressions in node parameters
- Access current item: `$json.fieldName`
- Access specific node: `$node["Node Name"].json.fieldName`
- Environment variables: `$env.VARIABLE_NAME`
- Current time: `$now`

### Code Node Returns
Always return array of objects with `json` key:
```javascript
return [{ json: { result: "data" } }];
```

### Validation
- The n8n-mcp server provides auto-sanitization
- Some validation errors may be false positives
- Always test workflows after creation
- Use validation profiles to catch issues early

### Production Safety
- NEVER edit production workflows directly with AI
- Always create copies for testing
- Export backups before making changes
- Validate in development first
- Test thoroughly before deploying

### Node Discovery
- Search for nodes by functionality, not exact name
- Community nodes available with `source` filter
- Check node documentation for parameter requirements
- Review workflow templates for usage examples

## Tips for Best Results

1. **Start specific**: Describe the exact inputs, transformations, and outputs you need
2. **Provide examples**: Share sample data or existing workflow structures
3. **Iterate quickly**: We'll build basic version first, then enhance
4. **Test together**: Review outputs and refine based on real data
5. **Document learnings**: I'll capture patterns that work well for future workflows
6. **Use templates**: Reference the 2,700+ workflow templates for proven patterns
7. **Leverage skills**: The seven n8n skills will automatically activate to guide development

## Current Project Status

- [x] Claude.md created
- [x] n8n MCP server documentation added
- [x] n8n Skills installed (all 7 skills)
- [x] n8n MCP server configured and connected
- [x] Connection verified - API working
- [x] Salesforce Skills documentation added
- [x] Salesforce reference files created (6 files)
- [x] Salesforce CLI authenticated and ready
- [x] Jira Skills documentation added
- [x] Jira reference files created (2 files)
- [x] Jira API credentials configured
- [ ] First test workflow created
- [ ] Workflow templates established

---

Ready to build workflows? Just describe what you need, and I'll help you create it using n8n MCP server, n8n skills, Salesforce, and Jira integration.
