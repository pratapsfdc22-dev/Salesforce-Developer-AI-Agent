# Claude Code for Salesforce - Quick Start

## Files Created
- `.clauderc` - Claude Code configuration for Salesforce
- `SALESFORCE_CONTEXT.md` - Org details and development standards
- `APEX_PATTERNS.md` - Apex code templates and patterns
- `LWC_PATTERNS.md` - Lightning Web Component patterns
- `SECURITY_CHECKLIST.md` - Pre-deployment security verification
- `CLAUDE_CODE_README.md` - This quick reference guide
- `CLAUDE.md` - Complete n8n + Salesforce integration documentation

## Your Salesforce Org
- **Instance:** <your-org>.my.salesforce.com
- **User:** your-email@example.com
- **Org ID:** <YOUR_ORG_ID>
- **API Version:** 64.0
- **CLI Status:** ✅ Authenticated

## Quick Commands

### Authentication & Org Management
```bash
# Display current org info
sf org display

# List all authenticated orgs
sf org list

# Open org in browser
sf org open

# Logout from org
sf org logout --target-org your-email@example.com
```

### Metadata Operations
```bash
# Retrieve metadata from org
sf project retrieve start --manifest manifest/package.xml

# Retrieve specific metadata
sf project retrieve start --metadata ApexClass:AccountTriggerHandler

# Deploy to org
sf project deploy start --manifest manifest/package.xml

# Deploy with tests
sf project deploy start --manifest manifest/package.xml --test-level RunLocalTests

# Quick deploy (after validation)
sf project deploy quick --job-id <deployment-id>
```

### Running Tests
```bash
# Run all local tests
sf apex run test --test-level RunLocalTests --result-format human --code-coverage

# Run specific test class
sf apex run test --class-names AccountTriggerHandler_Test --result-format human

# Run specific test method
sf apex run test --tests AccountTriggerHandler_Test.testAccountInsertValidation

# Check code coverage
sf apex get test --test-run-id <test-run-id> --code-coverage
```

### Data Operations
```bash
# Query data
sf data query --query "SELECT Id, Name FROM Account LIMIT 10"

# Export data
sf data export tree --query "SELECT Id, Name FROM Account" --output-dir ./data

# Import data
sf data import tree --files data/Account.json
```

### Creating Metadata
```bash
# Create Apex class
sf apex generate class --name MyClass --output-dir force-app/main/default/classes

# Create trigger
sf apex generate trigger --name AccountTrigger --sobject Account

# Create LWC
sf lightning generate component --name myComponent --type lwc
```

## Example Claude Code Prompts

### Create Custom Object
```
Create a custom object called "Project__c" with the following fields:
- Project_Name__c (Text, 80, Required, Unique)
- Project_Status__c (Picklist: Planning, In Progress, Completed, On Hold)
- Start_Date__c (Date, Required)
- End_Date__c (Date)
- Budget__c (Currency)
- Account__c (Lookup to Account, Required)

Include a validation rule that ensures End_Date__c is after Start_Date__c.
```

### Create Apex Trigger with Handler
```
Create a trigger on Opportunity that:
1. When an Opportunity is marked as Closed Won, create a Project__c record
2. Copy relevant fields (Name, Amount becomes Budget, Account)
3. Set Project Status to "Planning"
4. Use trigger handler pattern
5. Include bulkification for 200+ records
6. Include comprehensive test class with 100% coverage
```

### Create Lightning Web Component
```
Create an LWC that displays all Opportunities for an Account:
1. Show data in a lightning-datatable
2. Include columns: Name, Stage, Amount, Close Date
3. Add search filter functionality
4. Show loading spinner while fetching data
5. Handle errors with toast notifications
6. Use @wire to fetch data from Apex
```

### Create n8n Workflow
```
Create an n8n workflow that:
1. Listens for webhook from external form
2. Validates the incoming data (email format, required fields)
3. Creates a Lead in Salesforce
4. If Lead creation successful, send confirmation email
5. If Lead creation fails, log error and send notification to Slack
```

### Create Batch Job
```
Create a batch job that:
1. Finds all Accounts with Type = 'Prospect' and no activity in 90 days
2. Updates their Status to 'Inactive'
3. Creates a Task assigned to Account Owner
4. Runs in batches of 200
5. Includes error handling
6. Sends completion summary email
```

### Security Review
```
Review the AccountService class for security issues:
1. Check for CRUD/FLS violations
2. Verify SOQL injection prevention
3. Confirm proper use of `with sharing`
4. Identify any hardcoded values
5. Suggest improvements
```

## n8n + Salesforce Integration

### Available n8n Nodes
1. **Salesforce Node** (`nodes-base.salesforce`)
   - Operations: Create, Read, Update, Delete, Upsert
   - Resources: Account, Contact, Lead, Opportunity, Case, Task, Custom Objects

2. **Salesforce Trigger** (`nodes-base.salesforceTrigger`)
   - Polling-based trigger
   - Monitors Salesforce for new/updated records

3. **Salesforce Tool** (`nodes-base.salesforceTool`)
   - AI Agent variant for LangChain workflows

### Example Integration Workflow
```
Schedule Trigger (every hour)
  → Salesforce Trigger (get new Leads)
    → Code (validate and transform data)
      → HTTP Request (call external scoring API)
        → Salesforce (update Lead score)
          → IF (score > 80)
            → Salesforce (convert Lead to Opportunity)
            → Email notification to sales team
```

## Common Development Patterns

### Trigger Handler Pattern
```
1. One trigger per object
2. Trigger calls handler class
3. Handler extends base TriggerHandler
4. Business logic in service classes
5. Test coverage for all scenarios
```

### Service Layer Pattern
```
1. Service classes contain business logic
2. Static methods for reusability
3. Proper bulkification
4. Security checks (CRUD/FLS)
5. Error handling with custom exceptions
```

### Test Data Factory
```
1. @TestSetup for common data
2. Bulk insert test records (200+)
3. Reusable factory methods
4. Avoid hard dependencies
5. Test governor limits
```

## Debugging & Troubleshooting

### View Debug Logs
```bash
# Stream debug logs in real-time
sf apex tail log

# Get recent logs
sf apex get log --number 5
```

### Anonymous Apex
```bash
# Execute Anonymous Apex
sf apex run --file scripts/myScript.apex

# Execute inline
echo "System.debug('Hello');" | sf apex run
```

### Common Issues
- **CRUD/FLS Error:** Add `with sharing` and CRUD/FLS checks
- **SOQL 101 Error:** Move SOQL out of loops, use maps
- **DML 150 Error:** Bulkify DML operations
- **Heap Size Error:** Process data in smaller batches
- **CPU Time Error:** Optimize loops and SOQL queries

## Resources
- **Salesforce Developer Docs:** https://developer.salesforce.com/docs
- **Apex Best Practices:** https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/
- **LWC Guide:** https://developer.salesforce.com/docs/component-library/documentation/en/lwc
- **n8n Salesforce Docs:** https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.salesforce/
- **SFDX CLI Reference:** https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/

## Getting Help
Ask Claude Code to:
- Generate code with best practices
- Review code for security issues
- Create test classes
- Build LWC components
- Design n8n workflows
- Debug Apex issues
- Optimize SOQL queries
- Implement design patterns
