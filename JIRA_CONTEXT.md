# Jira Integration Context

## Jira Instance Details
- **Instance URL**: https://<your-domain>.atlassian.net
- **Type**: Jira Cloud
- **Email**: your-email@example.com
- **API Token**: Configured ✅
- **Authentication**: API Token (email + token)

## n8n Jira Integration

### Available n8n Jira Nodes

1. **Jira Software Node** (`nodes-base.jira`)
   - **Resources**: Issue, Issue Attachment, Issue Comment, User
   - **Operations**:
     - Issue: Create, Update, Delete, Get, Get Many, Changelog, Notify, Status
     - Attachment: Add, Get, Get Many, Remove
     - Comment: Add, Get, Get Many, Remove, Update
     - User: Get, Get Many
   - **Authentication**: API Token (Cloud), Basic Auth (Server)
   - **Use Cases**: Standard Jira operations

2. **Jira Trigger** (`nodes-base.jiraTrigger`)
   - **Type**: Polling trigger
   - **Purpose**: Monitors Jira for events (new issues, updates, comments)
   - **Use Cases**: Trigger workflows on Jira changes

3. **Jira Tool** (`nodes-base.jiraTool`)
   - **Type**: AI Tool variant for AI Agents
   - **Purpose**: Jira operations within LangChain workflows
   - **Use Cases**: AI agents that need to query or update Jira

## Common Jira Operations

### Issue Management
- **Create Issue**: Requires project key, issue type, and summary
- **Update Issue**: Modify fields like status, assignee, description, priority
- **Get Issue**: Retrieve issue details by issue key (e.g., PROJ-123)
- **Search Issues**: JQL (Jira Query Language) for complex queries

### Comments & Attachments
- **Add Comment**: Post comments to issues
- **Add Attachment**: Upload files to issues
- **Get Comments**: Retrieve all comments from an issue

### User Management
- **Get User**: Retrieve user details
- **Assign Issue**: Assign issues to users

## Jira Query Language (JQL)

### Common JQL Queries
```jql
# Find all open issues in project
project = MYPROJECT AND status = Open

# Issues assigned to me
assignee = currentUser() AND status != Done

# Recently created issues
created >= -7d

# High priority bugs
priority = High AND type = Bug

# Issues updated today
updated >= startOfDay()
```

## Integration Patterns

### 1. Salesforce → Jira Integration
**Use Case**: Create Jira issues from Salesforce Cases

**Pattern**:
```
Salesforce Trigger (new Case)
  → Code (transform data)
  → Jira (create Issue)
  → Salesforce (update Case with Jira key)
```

### 2. Automated Issue Creation
**Use Case**: Create Jira issues from external sources

**Pattern**:
```
Webhook (external event)
  → Code (validate & format)
  → Jira (create Issue)
  → Slack (notification)
```

### 3. Issue Status Sync
**Use Case**: Sync Jira issue status to external systems

**Pattern**:
```
Jira Trigger (issue updated)
  → IF (status changed)
  → HTTP Request (update external system)
  → Jira (add comment)
```

### 4. AI-Powered Issue Triage
**Use Case**: Automatically categorize and prioritize issues

**Pattern**:
```
Jira Trigger (new issue)
  → OpenAI Chat Model (analyze description)
  → Code (parse AI response)
  → Jira (update priority & labels)
```

### 5. Cross-System Synchronization
**Use Case**: Keep Jira and Salesforce in sync

**Pattern**:
```
Schedule Trigger (hourly)
  → Salesforce (get updated Cases)
  → Code (map fields)
  → Jira (update Issues)
  → Log errors to monitoring
```

## Authentication Setup

### For n8n Workflows

**Credentials Configuration**:
- **URL**: https://<your-domain>.atlassian.net
- **Email**: your-email@example.com
- **API Token**: Use the provided token
- **Authentication Type**: API Token (for Cloud)

### Creating API Token
1. Go to https://id.atlassian.com/manage-profile/security/api-tokens
2. Click "Create API token"
3. Give it a descriptive name
4. Copy and store securely
5. Use with email for authentication

## Jira Field Mapping

### Required Fields for Issue Creation
- **project**: Project key or ID (e.g., "PROJ")
- **issuetype**: Issue type (e.g., "Bug", "Task", "Story")
- **summary**: Brief title/description

### Common Optional Fields
- **description**: Detailed description (supports Jira markdown)
- **priority**: Priority level (e.g., "High", "Medium", "Low")
- **assignee**: User account ID
- **labels**: Array of labels
- **duedate**: Due date (YYYY-MM-DD format)
- **components**: Array of component IDs
- **customfield_***: Custom fields specific to your instance

## Best Practices

### Issue Creation
1. **Validate input data** before creating issues
2. **Use descriptive summaries** (50-100 characters)
3. **Include context** in description
4. **Set appropriate priority** based on urgency
5. **Add labels** for categorization
6. **Link related issues** when applicable

### Error Handling
1. **Check for duplicate issues** before creation
2. **Validate project and issue type** exist
3. **Handle authentication failures** gracefully
4. **Log failed operations** for debugging
5. **Implement retry logic** for transient failures

### Performance
1. **Bulk operations** when possible
2. **Use pagination** for large result sets
3. **Cache project/issue type data**
4. **Minimize API calls** with efficient queries
5. **Use webhooks** instead of polling when possible

## Jira Automation Triggers

### When to Use Jira Trigger vs Webhook
- **Jira Trigger** (Polling):
  - Simple setup
  - No Jira admin access required
  - 1-5 minute delay
  - Good for: scheduled checks, low-frequency events

- **Jira Webhook** (Real-time):
  - Requires Jira admin access
  - Instant notification
  - More complex setup
  - Good for: real-time updates, high-frequency events

## Security Considerations

1. **API Token Security**:
   - Never hardcode tokens in workflows
   - Use n8n credentials manager
   - Rotate tokens periodically
   - Limit token scope if possible

2. **Data Validation**:
   - Validate all input from external sources
   - Sanitize HTML/markdown in descriptions
   - Check user permissions before assignments
   - Verify project access

3. **Error Messages**:
   - Don't expose sensitive data in errors
   - Log errors securely
   - Provide generic error messages to end users

## Common Jira API Limits

- **Rate Limits**:
  - Cloud: ~10 requests per second per IP
  - Server: Configured by admin
- **Search Results**: Max 1000 issues per query (use pagination)
- **Attachment Size**: Max 10MB per file (Cloud)

## Troubleshooting

### Common Issues
1. **Authentication Failed**: Verify email and API token are correct
2. **Project Not Found**: Check project key spelling and access permissions
3. **Invalid Issue Type**: Ensure issue type exists in the project
4. **Field Required**: Some projects have custom required fields
5. **Rate Limit Exceeded**: Implement exponential backoff and retry

### Debug Tips
1. Test API calls with Postman or curl first
2. Check Jira audit log for API activity
3. Verify project permissions
4. Use Jira REST API documentation for field names
5. Enable debug logging in n8n workflows

## Resources
- **Jira REST API**: https://developer.atlassian.com/cloud/jira/platform/rest/v3/
- **JQL Documentation**: https://support.atlassian.com/jira-service-management-cloud/docs/use-advanced-search-with-jira-query-language-jql/
- **n8n Jira Node**: https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.jira/
- **API Token Management**: https://id.atlassian.com/manage-profile/security/api-tokens
