# Apex Code Patterns & Templates

## Trigger Handler Pattern

### Main Trigger (One Per Object)
```apex
trigger AccountTrigger on Account (before insert, before update, before delete,
                                    after insert, after update, after delete, after undelete) {
    new AccountTriggerHandler().run();
}
```

### Handler Class
```apex
public with sharing class AccountTriggerHandler extends TriggerHandler {

    private List<Account> newRecords;
    private List<Account> oldRecords;
    private Map<Id, Account> newRecordsMap;
    private Map<Id, Account> oldRecordsMap;

    public AccountTriggerHandler() {
        this.newRecords = (List<Account>) Trigger.new;
        this.oldRecords = (List<Account>) Trigger.old;
        this.newRecordsMap = (Map<Id, Account>) Trigger.newMap;
        this.oldRecordsMap = (Map<Id, Account>) Trigger.oldMap;
    }

    public override void beforeInsert() {
        AccountService.validateAccountData(newRecords);
        AccountService.setDefaultValues(newRecords);
    }

    public override void afterUpdate() {
        AccountService.handleRelatedContactUpdates(newRecordsMap, oldRecordsMap);
    }
}
```

### Base TriggerHandler Class
```apex
public virtual class TriggerHandler {

    private static Set<String> bypassedHandlers = new Set<String>();

    public void run() {
        if (shouldBypass()) return;

        if (Trigger.isBefore) {
            if (Trigger.isInsert) beforeInsert();
            if (Trigger.isUpdate) beforeUpdate();
            if (Trigger.isDelete) beforeDelete();
        } else {
            if (Trigger.isInsert) afterInsert();
            if (Trigger.isUpdate) afterUpdate();
            if (Trigger.isDelete) afterDelete();
            if (Trigger.isUndelete) afterUndelete();
        }
    }

    protected virtual void beforeInsert() {}
    protected virtual void beforeUpdate() {}
    protected virtual void beforeDelete() {}
    protected virtual void afterInsert() {}
    protected virtual void afterUpdate() {}
    protected virtual void afterDelete() {}
    protected virtual void afterUndelete() {}

    private Boolean shouldBypass() {
        return bypassedHandlers.contains(String.valueOf(this).split(':')[0]);
    }

    public static void bypass(String handlerName) {
        bypassedHandlers.add(handlerName);
    }

    public static void clearBypass(String handlerName) {
        bypassedHandlers.remove(handlerName);
    }
}
```

## Service Class Pattern (Business Logic)

```apex
public with sharing class AccountService {

    /**
     * Validates account data before DML
     * @param accounts List of accounts to validate
     */
    public static void validateAccountData(List<Account> accounts) {
        for (Account acc : accounts) {
            if (String.isBlank(acc.Name)) {
                acc.addError('Account Name is required');
            }
            if (acc.AnnualRevenue != null && acc.AnnualRevenue < 0) {
                acc.AnnualRevenue.addError('Annual Revenue cannot be negative');
            }
        }
    }

    /**
     * Sets default field values
     * @param accounts List of accounts
     */
    public static void setDefaultValues(List<Account> accounts) {
        for (Account acc : accounts) {
            if (String.isBlank(acc.Type)) {
                acc.Type = 'Prospect';
            }
        }
    }

    /**
     * Handles related contact updates when account changes
     * @param newAccountsMap Map of new account records
     * @param oldAccountsMap Map of old account records
     */
    public static void handleRelatedContactUpdates(Map<Id, Account> newAccountsMap,
                                                    Map<Id, Account> oldAccountsMap) {
        // Identify accounts with billing address changes
        Set<Id> accountIds = new Set<Id>();
        for (Id accountId : newAccountsMap.keySet()) {
            Account newAcc = newAccountsMap.get(accountId);
            Account oldAcc = oldAccountsMap.get(accountId);

            if (newAcc.BillingCity != oldAcc.BillingCity) {
                accountIds.add(accountId);
            }
        }

        if (accountIds.isEmpty()) return;

        // Bulkified SOQL
        List<Contact> contactsToUpdate = [
            SELECT Id, MailingCity, AccountId
            FROM Contact
            WHERE AccountId IN :accountIds
        ];

        // Update contacts
        for (Contact con : contactsToUpdate) {
            con.MailingCity = newAccountsMap.get(con.AccountId).BillingCity;
        }

        // Bulkified DML
        if (!contactsToUpdate.isEmpty()) {
            update contactsToUpdate;
        }
    }
}
```

## Test Class Pattern

```apex
@IsTest
private class AccountTriggerHandler_Test {

    @TestSetup
    static void setupTestData() {
        // Create test data once for all test methods
        List<Account> accounts = new List<Account>();
        for (Integer i = 0; i < 200; i++) {
            accounts.add(new Account(
                Name = 'Test Account ' + i,
                Type = 'Customer',
                BillingCity = 'San Francisco'
            ));
        }
        insert accounts;

        // Create related contacts
        List<Contact> contacts = new List<Contact>();
        for (Account acc : accounts) {
            contacts.add(new Contact(
                FirstName = 'Test',
                LastName = 'Contact',
                AccountId = acc.Id,
                MailingCity = acc.BillingCity
            ));
        }
        insert contacts;
    }

    @IsTest
    static void testAccountInsertValidation() {
        Test.startTest();

        // Test successful insert
        Account validAccount = new Account(Name = 'Valid Account');
        insert validAccount;

        // Test validation error
        Account invalidAccount = new Account(AnnualRevenue = -1000);
        try {
            insert invalidAccount;
            System.assert(false, 'Should have thrown validation error');
        } catch (DmlException e) {
            System.assert(e.getMessage().contains('Name is required'));
        }

        Test.stopTest();
    }

    @IsTest
    static void testBulkAccountUpdate() {
        Test.startTest();

        // Query test accounts
        List<Account> accounts = [SELECT Id, BillingCity FROM Account LIMIT 200];

        // Bulk update
        for (Account acc : accounts) {
            acc.BillingCity = 'New York';
        }
        update accounts;

        Test.stopTest();

        // Verify contact updates
        List<Contact> updatedContacts = [
            SELECT MailingCity
            FROM Contact
            WHERE AccountId IN :accounts
        ];

        for (Contact con : updatedContacts) {
            System.assertEquals('New York', con.MailingCity,
                'Contact mailing city should match account billing city');
        }
    }

    @IsTest
    static void testGovernorLimits() {
        // Test against governor limits
        Test.startTest();

        List<Account> accounts = [SELECT Id FROM Account];
        System.assertEquals(200, accounts.size(), 'Should have 200 test accounts');

        // Verify we're not hitting limits
        System.assert(Limits.getQueries() < Limits.getLimitQueries(),
            'Should not exceed SOQL query limit');

        Test.stopTest();
    }
}
```

## SOQL Best Practices

### Use Selective Queries
```apex
// ✅ GOOD - Uses indexed field
List<Account> accounts = [
    SELECT Id, Name
    FROM Account
    WHERE Id = :accountId
];

// ✅ GOOD - Uses indexed field with IN clause
List<Contact> contacts = [
    SELECT Id, Name
    FROM Contact
    WHERE AccountId IN :accountIds
];

// ❌ BAD - Non-selective query
List<Account> accounts = [
    SELECT Id, Name
    FROM Account
    WHERE Custom_Field__c = 'Value'
];
```

### Bulkify Queries
```apex
// ❌ BAD - Query in loop
for (Account acc : accounts) {
    List<Contact> contacts = [
        SELECT Id FROM Contact WHERE AccountId = :acc.Id
    ];
}

// ✅ GOOD - Single query
Map<Id, Account> accountsWithContacts = new Map<Id, Account>([
    SELECT Id, (SELECT Id FROM Contacts)
    FROM Account
    WHERE Id IN :accountIds
]);
```

### Prevent SOQL Injection
```apex
// ❌ BAD - Vulnerable to injection
String query = 'SELECT Id FROM Account WHERE Name = \'' + userInput + '\'';
List<Account> accounts = Database.query(query);

// ✅ GOOD - Use bind variables
String searchName = userInput;
List<Account> accounts = [
    SELECT Id FROM Account WHERE Name = :searchName
];

// ✅ GOOD - Use String.escapeSingleQuotes() if dynamic SOQL is necessary
String safeName = String.escapeSingleQuotes(userInput);
String query = 'SELECT Id FROM Account WHERE Name = \'' + safeName + '\'';
```

## Batch Apex Pattern

```apex
public class AccountBatchProcessor implements Database.Batchable<sObject> {

    private String query;

    public AccountBatchProcessor(String soqlQuery) {
        this.query = soqlQuery;
    }

    public Database.QueryLocator start(Database.BatchableContext bc) {
        return Database.getQueryLocator(query);
    }

    public void execute(Database.BatchableContext bc, List<Account> scope) {
        // Process batch of records
        for (Account acc : scope) {
            // Your business logic here
            acc.Description = 'Processed by batch';
        }

        // Bulkified DML
        update scope;
    }

    public void finish(Database.BatchableContext bc) {
        // Send notification email or perform cleanup
        System.debug('Batch job completed');
    }
}

// Execute batch
String query = 'SELECT Id, Name, Description FROM Account WHERE Type = \'Customer\'';
AccountBatchProcessor batch = new AccountBatchProcessor(query);
Database.executeBatch(batch, 200);
```

## Queueable Apex Pattern

```apex
public class AccountQueueableProcessor implements Queueable {

    private List<Account> accounts;

    public AccountQueueableProcessor(List<Account> accs) {
        this.accounts = accs;
    }

    public void execute(QueueableContext context) {
        // Perform async processing
        for (Account acc : accounts) {
            // Your logic here
        }

        // Chain another queueable if needed
        // System.enqueueJob(new AnotherQueueable());
    }
}

// Enqueue
System.enqueueJob(new AccountQueueableProcessor(accountList));
```

## Security & Sharing Rules

### Using with sharing
```apex
public with sharing class SecureAccountService {

    // Respects user's sharing rules
    public static List<Account> getAccounts() {
        return [
            SELECT Id, Name
            FROM Account
            WITH SECURITY_ENFORCED
        ];
    }
}
```

### Checking CRUD/FLS
```apex
public class SecureDataService {

    public static void createAccount(Account acc) {
        // Check CRUD
        if (!Schema.sObjectType.Account.isCreateable()) {
            throw new SecurityException('User cannot create Accounts');
        }

        // Check FLS
        if (!Schema.sObjectType.Account.fields.Name.isCreateable()) {
            throw new SecurityException('User cannot set Account Name');
        }

        insert acc;
    }
}
```

## Error Handling Pattern

```apex
public class RobustService {

    public static void processRecords(List<Account> accounts) {
        try {
            // Business logic
            update accounts;
        } catch (DmlException e) {
            // Handle DML errors
            for (Integer i = 0; i < e.getNumDml(); i++) {
                System.debug('Error on record ' + e.getDmlIndex(i) + ': ' + e.getDmlMessage(i));
            }
            throw new CustomException('Failed to update accounts: ' + e.getMessage());
        } catch (Exception e) {
            // Handle generic errors
            System.debug('Unexpected error: ' + e.getMessage());
            throw e;
        }
    }
}

public class CustomException extends Exception {}
```
