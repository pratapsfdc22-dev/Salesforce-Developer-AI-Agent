# Salesforce Security Checklist

Before deploying ANY code, verify:

## Apex Security
- [ ] Classes use `with sharing` or `inherited sharing`
- [ ] SOQL queries include `WITH SECURITY_ENFORCED` where appropriate
- [ ] DML operations check `Schema.sObjectType.ObjectName.isCreateable()` etc.
- [ ] User input is validated and sanitized
- [ ] No SOQL injection vulnerabilities
- [ ] No hardcoded credentials
- [ ] Sensitive data is encrypted
- [ ] Governor limits are not exceeded
- [ ] All operations are bulkified (handle 200+ records)

## Field-Level Security (FLS)
- [ ] Code respects FLS using `Schema.DescribeFieldResult.isAccessible()`
- [ ] Code respects FLS using `Schema.DescribeFieldResult.isUpdateable()`
- [ ] Permission sets configured correctly
- [ ] Field permissions match business requirements

## Object-Level Security (CRUD)
- [ ] Check object create permission before insert
- [ ] Check object read permission before query
- [ ] Check object update permission before update
- [ ] Check object delete permission before delete
- [ ] Use `stripInaccessible()` to automatically remove inaccessible fields

## Data Access & Sharing
- [ ] Sharing rules considered
- [ ] Record access validated
- [ ] Proper use of `with sharing` / `without sharing` / `inherited sharing`
- [ ] Manual sharing implemented where necessary
- [ ] Test with different user profiles

## Lightning Components (LWC/Aura)
- [ ] User input is escaped
- [ ] Apex controllers use `@AuraEnabled`
- [ ] Sensitive operations require additional authentication
- [ ] CSP (Content Security Policy) compliance
- [ ] No hardcoded data or credentials in JavaScript
- [ ] Proper error handling without exposing sensitive information

## n8n Integration Security
- [ ] OAuth2 authentication configured correctly
- [ ] API credentials stored securely
- [ ] Webhook endpoints use authentication
- [ ] Data validation before Salesforce operations
- [ ] Error handling doesn't expose sensitive data
- [ ] Rate limiting considered for API calls

## Testing
- [ ] Test with different user profiles (System Admin, Standard User, Custom)
- [ ] Test with restricted permissions
- [ ] Test bulk operations (200+ records)
- [ ] Test governor limits
- [ ] Test error scenarios
- [ ] Achieve minimum 85% code coverage (aim for 100%)

## Code Review Checklist
- [ ] All public methods documented with comments
- [ ] No debug statements or commented code in production
- [ ] Proper exception handling throughout
- [ ] No hardcoded Ids or environment-specific values
- [ ] Following naming conventions
- [ ] Code is bulkified
- [ ] No SOQL/DML in loops

## Deployment Checklist
- [ ] Code deployed to sandbox first
- [ ] All tests passing in sandbox
- [ ] Manual testing completed
- [ ] Security review completed
- [ ] Change set or metadata package prepared
- [ ] Deployment window scheduled
- [ ] Rollback plan documented

## Common Security Vulnerabilities to Avoid

### SOQL Injection
```apex
// ❌ BAD
String query = 'SELECT Id FROM Account WHERE Name = \'' + userInput + '\'';
List<Account> accounts = Database.query(query);

// ✅ GOOD
String safeName = String.escapeSingleQuotes(userInput);
String query = 'SELECT Id FROM Account WHERE Name = \'' + safeName + '\'';

// ✅ BETTER
List<Account> accounts = [SELECT Id FROM Account WHERE Name = :userInput];
```

### Missing CRUD/FLS Checks
```apex
// ❌ BAD
public static void createAccount(Account acc) {
    insert acc;
}

// ✅ GOOD
public static void createAccount(Account acc) {
    if (!Schema.sObjectType.Account.isCreateable()) {
        throw new SecurityException('No permission to create Account');
    }
    insert acc;
}
```

### Sharing Violations
```apex
// ❌ BAD - No sharing declaration
public class AccountService {
    // May bypass sharing rules
}

// ✅ GOOD - Explicit sharing
public with sharing class AccountService {
    // Respects user sharing rules
}
```

### Exposed Sensitive Data
```apex
// ❌ BAD
@AuraEnabled
public static String getApiKey() {
    return 'sk-12345...'; // Never expose secrets
}

// ✅ GOOD
@AuraEnabled
public static void callExternalService() {
    String apiKey = System.Label.API_Key; // Use custom metadata/labels
    // Make API call with key
}
```

## Resources
- **Salesforce Security Guide:** https://developer.salesforce.com/docs/atlas.en-us.securityImplGuide.meta/securityImplGuide/
- **OWASP Top 10:** https://owasp.org/www-project-top-ten/
- **Secure Coding Guidelines:** https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_security_sharing.htm
