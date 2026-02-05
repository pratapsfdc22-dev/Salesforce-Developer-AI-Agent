# Salesforce Org Context

## Org Details
- **Type:** Developer Edition with DevHub
- **Instance:** <your-org>.my.salesforce.com
- **User:** your-email@example.com
- **Org ID:** <YOUR_ORG_ID>
- **API Version:** 64.0
- **CLI Status:** ✅ Authenticated

## Development Standards

### Apex Best Practices
1. **Bulkification:** All triggers and code must handle bulk operations (200+ records)
2. **Governor Limits:** Stay within SOQL queries (100), DML statements (150), heap size (6MB sync/12MB async)
3. **SOQL Selective Queries:** Use indexed fields in WHERE clauses
4. **Test Coverage:** Minimum 85% coverage, aim for 100%
5. **Security:** Always check CRUD/FLS permissions before DML operations

### Naming Conventions
- **Apex Classes:** PascalCase (e.g., `AccountTriggerHandler`)
- **Apex Methods:** camelCase (e.g., `processAccountUpdates`)
- **Triggers:** ObjectName + Trigger (e.g., `AccountTrigger`)
- **Custom Objects:** PascalCase with `__c` suffix (e.g., `Project__c`)
- **Custom Fields:** Snake_case with `__c` suffix (e.g., `Project_Status__c`)

### Code Structure
```
force-app/
├── main/
│   └── default/
│       ├── classes/          # Apex classes
│       ├── triggers/         # Apex triggers
│       ├── lwc/             # Lightning Web Components
│       ├── aura/            # Aura components
│       ├── objects/         # Custom objects
│       ├── flows/           # Flows
│       ├── permissionsets/  # Permission sets
│       └── ...
```

## Common Development Tasks

### 1. Creating Custom Objects
- Define business requirements first
- Plan relationships (Master-Detail vs Lookup)
- Set up page layouts and record types
- Configure security (profiles/permission sets)

### 2. Writing Apex Triggers
- Use trigger handler pattern (one trigger per object)
- Delegate logic to handler classes
- Implement trigger framework for recursion prevention

### 3. Building Flows
- Use Record-Triggered Flows for automation
- Screen Flows for guided user experiences
- Autolaunched Flows for scheduled processes

### 4. Lightning Web Components
- Use base components when possible
- Follow LDS (Lightning Data Service) for data operations
- Implement proper error handling
- Use @wire for reactive data

## n8n Integration

### Available n8n Salesforce Nodes
1. **Salesforce Node** - CRUD operations on Salesforce objects
2. **Salesforce Trigger** - Polling-based triggers for workflow automation
3. **Salesforce Tool** - AI Agent variant for LangChain workflows

### Common Integration Patterns
- **Lead Routing:** Webhook → Validation → Salesforce Lead Creation
- **Opportunity Sync:** Scheduled polling → Data transformation → External API
- **Data Validation:** Salesforce Trigger → Validation Logic → Error Handling
- **AI Workflows:** LangChain Agent → Salesforce Tool → SOQL Queries

## Security Checklist
- [ ] Check object permissions (CRUD)
- [ ] Check field permissions (FLS)
- [ ] Use `with sharing` in Apex classes
- [ ] Validate user input
- [ ] Prevent SOQL injection
- [ ] Test with different profiles

## Governor Limits Quick Reference
| Limit | Sync | Async |
|-------|------|-------|
| SOQL Queries | 100 | 200 |
| DML Statements | 150 | 150 |
| Heap Size | 6 MB | 12 MB |
| CPU Time | 10,000ms | 60,000ms |
| Records per DML | 10,000 | 10,000 |

## Salesforce CLI Quick Commands

### Authentication & Org Management
```bash
# Display current org
sf org display

# List all authenticated orgs
sf org list

# Open org in browser
sf org open
```

### Deployment
```bash
# Deploy source to org
sf project deploy start --manifest manifest/package.xml

# Deploy with tests
sf project deploy start --manifest manifest/package.xml --test-level RunLocalTests

# Retrieve metadata from org
sf project retrieve start --manifest manifest/package.xml
```

### Testing
```bash
# Run all tests
sf apex run test --test-level RunLocalTests --result-format human --code-coverage

# Run specific test class
sf apex run test --class-names MyTestClass --result-format human
```

## Resources
- **Salesforce Developer Guide:** https://developer.salesforce.com/docs
- **Apex Best Practices:** https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/
- **LWC Guide:** https://developer.salesforce.com/docs/component-library/documentation/en/lwc
- **n8n Salesforce Node Docs:** https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.salesforce/
