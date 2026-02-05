# Jira Integration Patterns for n8n

## Pattern 1: Create Jira Issue from Webhook

### Use Case
External system (form, app, API) sends data to create Jira issues automatically.

### Workflow Structure
```
Webhook
  → Code (validate & transform)
  → Jira (create issue)
  → IF (success)
    → Respond to Webhook (200 OK + issue key)
    → Slack (notification)
  → IF (error)
    → Respond to Webhook (400 Error)
    → Log error
```

### Code Node Example
```javascript
// Validate and transform webhook data
const input = $json.body;

// Validation
if (!input.summary || !input.project) {
  throw new Error('Missing required fields: summary, project');
}

// Transform to Jira format
return [{
  json: {
    project: {
      key: input.project // e.g., "PROJ"
    },
    issuetype: {
      name: input.type || "Task"
    },
    summary: input.summary,
    description: input.description || "",
    priority: {
      name: input.priority || "Medium"
    },
    labels: input.labels || []
  }
}];
```

---

## Pattern 2: Salesforce Case to Jira Issue Sync

### Use Case
Automatically create Jira issues when high-priority Salesforce Cases are created.

### Workflow Structure
```
Salesforce Trigger (new Case)
  → IF (Priority = High)
    → Code (transform Case to Jira format)
    → Jira (create Issue)
    → Code (extract Jira key)
    → Salesforce (update Case with Jira Issue Key)
    → Email (notify team)
```

### Code Node: Transform Case to Jira
```javascript
// Transform Salesforce Case to Jira Issue format
const sfCase = $json;

return [{
  json: {
    fields: {
      project: {
        key: "SUP" // Support project
      },
      issuetype: {
        name: "Bug"
      },
      summary: `[SF Case ${sfCase.CaseNumber}] ${sfCase.Subject}`,
      description: `
*Salesforce Case*: ${sfCase.CaseNumber}
*Account*: ${sfCase.Account.Name}
*Contact*: ${sfCase.Contact.Name}
*Priority*: ${sfCase.Priority}

*Description*:
${sfCase.Description}

*Case Link*: https://yourinstance.salesforce.com/${sfCase.Id}
      `,
      priority: {
        name: sfCase.Priority
      },
      labels: ["salesforce", "customer-case"]
    }
  }
}];
```

### Code Node: Update Salesforce with Jira Key
```javascript
// Extract Jira key and prepare Salesforce update
const jiraResponse = $items("Jira")[0].json;
const caseId = $items("Salesforce Trigger")[0].json.Id;

return [{
  json: {
    Id: caseId,
    Jira_Issue_Key__c: jiraResponse.key // Custom field in Salesforce
  }
}];
```

---

## Pattern 3: Jira Issue Status Sync

### Use Case
Monitor Jira issues and sync status changes to external systems (Salesforce, Slack, database).

### Workflow Structure
```
Jira Trigger (issue updated)
  → IF (status changed)
    → Code (determine status mapping)
    → Branch (multiple paths):
      → Salesforce (update Case status)
      → Database (log status change)
      → Slack (notify team)
```

### Code Node: Status Change Detection
```javascript
// Check if status actually changed
const issue = $json;
const changelog = issue.changelog || {};

// Find status change in changelog
const statusChange = changelog.items?.find(item => item.field === 'status');

if (!statusChange) {
  // No status change, skip processing
  return [];
}

return [{
  json: {
    issueKey: issue.key,
    oldStatus: statusChange.fromString,
    newStatus: statusChange.toString,
    timestamp: issue.updated,
    summary: issue.fields.summary
  }
}];
```

---

## Pattern 4: AI-Powered Issue Categorization

### Use Case
Use AI to automatically categorize, prioritize, and label new Jira issues.

### Workflow Structure
```
Jira Trigger (new issue)
  → Code (extract issue details)
  → OpenAI Chat Model (analyze & categorize)
  → Code (parse AI response)
  → Jira (update issue with labels & priority)
  → Jira (add comment with AI analysis)
```

### Code Node: Prepare AI Prompt
```javascript
// Prepare issue data for AI analysis
const issue = $json;

const prompt = `Analyze this Jira issue and provide categorization:

**Summary**: ${issue.fields.summary}

**Description**:
${issue.fields.description || "No description provided"}

**Current Priority**: ${issue.fields.priority?.name || "None"}

Please provide:
1. Suggested priority (High/Medium/Low)
2. Appropriate labels (max 3)
3. Issue type classification (Bug/Feature/Improvement/Task)
4. Brief reasoning

Format as JSON:
{
  "priority": "High|Medium|Low",
  "labels": ["label1", "label2"],
  "type": "Bug|Feature|Improvement|Task",
  "reasoning": "explanation"
}`;

return [{
  json: {
    issueKey: issue.key,
    prompt: prompt
  }
}];
```

### Code Node: Parse AI Response
```javascript
// Parse AI response and format for Jira update
const aiResponse = $json.message?.content || $json.text;
const issueKey = $items("Jira Trigger")[0].json.key;

let analysis;
try {
  // Extract JSON from AI response
  const jsonMatch = aiResponse.match(/\{[\s\S]*\}/);
  analysis = JSON.parse(jsonMatch[0]);
} catch (e) {
  throw new Error('Failed to parse AI response: ' + e.message);
}

return [{
  json: {
    issueKey: issueKey,
    update: {
      priority: { name: analysis.priority },
      labels: analysis.labels
    },
    comment: `🤖 AI Analysis:\n\n${analysis.reasoning}\n\n*Suggested Priority*: ${analysis.priority}\n*Labels*: ${analysis.labels.join(', ')}`
  }
}];
```

---

## Pattern 5: Bulk Issue Creation from CSV/Database

### Use Case
Create multiple Jira issues from a CSV file or database export.

### Workflow Structure
```
Schedule Trigger (daily)
  → Spreadsheet File / Database (read data)
  → Code (split into items)
  → Loop Over Items:
    → Code (validate & format)
    → Jira (create issue)
    → Code (track results)
  → Send Summary Email
```

### Code Node: Process CSV Data
```javascript
// Split CSV/database data into individual items
const data = $json.data; // Array of records

return data.map(row => ({
  json: {
    fields: {
      project: { key: row.project },
      issuetype: { name: row.type || "Task" },
      summary: row.summary,
      description: row.description,
      assignee: row.assignee ? { accountId: row.assignee } : null,
      priority: { name: row.priority || "Medium" },
      labels: row.labels ? row.labels.split(',') : [],
      duedate: row.duedate || null
    }
  }
}));
```

---

## Pattern 6: Epic and Story Management

### Use Case
Create epics with multiple child stories automatically.

### Workflow Structure
```
Webhook (epic data)
  → Jira (create Epic)
  → Code (extract epic key)
  → Loop Over Stories:
    → Jira (create Story)
    → Jira (link Story to Epic)
  → Jira (update Epic with summary)
```

### Code Node: Create Story with Epic Link
```javascript
// Create story linked to epic
const story = $json;
const epicKey = $items("Create Epic")[0].json.key;

return [{
  json: {
    fields: {
      project: { key: "PROJ" },
      issuetype: { name: "Story" },
      summary: story.summary,
      description: story.description,
      parent: {
        key: epicKey // Link to epic
      },
      labels: ["epic-" + epicKey.toLowerCase()]
    }
  }
}];
```

---

## Pattern 7: Issue Comment Notifications

### Use Case
Send notifications when specific keywords appear in Jira comments.

### Workflow Structure
```
Jira Trigger (comment added)
  → Code (check for keywords)
  → IF (keyword found)
    → Extract mentioned users
    → Slack (notify users)
    → Email (send notification)
```

### Code Node: Keyword Detection
```javascript
// Check comment for keywords and extract context
const comment = $json;
const keywords = ["urgent", "blocked", "help needed", "@team"];

const commentText = comment.body?.toLowerCase() || "";
const foundKeywords = keywords.filter(kw => commentText.includes(kw));

if (foundKeywords.length === 0) {
  // No keywords found, skip
  return [];
}

return [{
  json: {
    issueKey: comment.issueKey,
    commentId: comment.id,
    author: comment.author.displayName,
    keywords: foundKeywords,
    commentText: comment.body,
    timestamp: comment.created,
    url: `https://<your-domain>.atlassian.net/browse/${comment.issueKey}`
  }
}];
```

---

## Pattern 8: SLA Monitoring and Escalation

### Use Case
Monitor issue age and escalate overdue items.

### Workflow Structure
```
Schedule Trigger (every hour)
  → Jira (search: JQL query for old issues)
  → Code (calculate age & urgency)
  → IF (overdue)
    → Jira (update priority)
    → Jira (add comment)
    → Slack (escalation notification)
    → Email (manager notification)
```

### JQL Query for Old Issues
```jql
project = PROJ
AND status != Done
AND created <= -7d
AND priority != High
ORDER BY created ASC
```

### Code Node: Calculate Issue Age
```javascript
// Calculate issue age and determine escalation
const issue = $json;
const createdDate = new Date(issue.fields.created);
const now = new Date();
const ageInDays = Math.floor((now - createdDate) / (1000 * 60 * 60 * 24));

// Escalation rules
const needsEscalation =
  (ageInDays > 7 && issue.fields.priority.name !== "High") ||
  (ageInDays > 14 && issue.fields.status.name === "To Do");

if (!needsEscalation) {
  return [];
}

return [{
  json: {
    issueKey: issue.key,
    summary: issue.fields.summary,
    ageInDays: ageInDays,
    currentPriority: issue.fields.priority.name,
    currentStatus: issue.fields.status.name,
    assignee: issue.fields.assignee?.displayName || "Unassigned",
    escalationReason: ageInDays > 14 ? "Stale issue (14+ days)" : "Aging issue (7+ days)"
  }
}];
```

---

## Pattern 9: Cross-Project Issue Linking

### Use Case
Automatically link related issues across different Jira projects.

### Workflow Structure
```
Jira Trigger (issue created)
  → Code (extract keywords/references)
  → Jira (search related issues via JQL)
  → IF (related issues found)
    → Loop Over Related Issues:
      → Jira (create issue link)
    → Jira (add comment with links)
```

### Code Node: Find Related Issues
```javascript
// Build JQL to find related issues
const issue = $json;
const summary = issue.fields.summary.toLowerCase();
const description = (issue.fields.description || "").toLowerCase();

// Extract potential project keys (e.g., PROJ-123)
const projectKeyPattern = /([A-Z]{2,}-\d+)/g;
const referencedIssues = [...summary.matchAll(projectKeyPattern),
                           ...description.matchAll(projectKeyPattern)]
  .map(match => match[1]);

// Build JQL to search for related issues
const keywords = summary.split(' ').filter(w => w.length > 4).slice(0, 3);
const jql = `summary ~ "${keywords.join(' ')}" AND key != ${issue.key}`;

return [{
  json: {
    sourceIssueKey: issue.key,
    referencedIssues: [...new Set(referencedIssues)],
    searchJQL: jql
  }
}];
```

---

## Best Practices for Jira Workflows

### Error Handling
```javascript
try {
  // Jira operation
  const result = await jiraOperation();
  return [{ json: result }];
} catch (error) {
  // Log error with context
  console.error('Jira operation failed:', {
    issueKey: $json.key,
    operation: 'create',
    error: error.message
  });

  // Return empty to skip or throw for retry
  throw new Error(`Jira error: ${error.message}`);
}
```

### Rate Limiting
```javascript
// Add delay between bulk operations
const items = $input.all();
const delay = ms => new Promise(resolve => setTimeout(resolve, ms));

for (let i = 0; i < items.length; i++) {
  // Process item

  // Wait 100ms between requests
  if (i < items.length - 1) {
    await delay(100);
  }
}
```

### Field Validation
```javascript
// Validate required fields before Jira API call
const required = ['project', 'issuetype', 'summary'];
const missing = required.filter(field => !$json.fields[field]);

if (missing.length > 0) {
  throw new Error(`Missing required fields: ${missing.join(', ')}`);
}
```

## Testing Tips

1. **Start with Manual Trigger**: Test workflows manually before enabling automated triggers
2. **Use Test Projects**: Create a test Jira project for development
3. **Validate Data**: Add validation nodes to catch errors early
4. **Log Operations**: Use Set node to log intermediate data
5. **Test Error Paths**: Intentionally trigger errors to test error handling
6. **Monitor API Usage**: Keep track of API call volume and rate limits

## Resources
- [Jira REST API Documentation](https://developer.atlassian.com/cloud/jira/platform/rest/v3/)
- [JQL Reference](https://support.atlassian.com/jira-service-management-cloud/docs/use-advanced-search-with-jira-query-language-jql/)
- [n8n Jira Integration](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.jira/)
