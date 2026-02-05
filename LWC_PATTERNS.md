# Lightning Web Component Patterns

## Component Structure

### HTML Template
```html
<!-- myComponent.html -->
<template>
    <lightning-card title="Account Details" icon-name="standard:account">
        <div class="slds-p-around_medium">
            <template if:true={account.data}>
                <lightning-record-form
                    record-id={recordId}
                    object-api-name="Account"
                    fields={fields}
                    onsubmit={handleSubmit}
                    onsuccess={handleSuccess}
                    onerror={handleError}>
                </lightning-record-form>
            </template>
            <template if:true={account.error}>
                <div class="slds-text-color_error">
                    Error loading account: {account.error.body.message}
                </div>
            </template>
        </div>
    </lightning-card>
</template>
```

### JavaScript Controller
```javascript
// myComponent.js
import { LightningElement, api, wire, track } from 'lwc';
import { ShowToastEvent } from 'lightning/platformShowToastEvent';
import { getRecord, updateRecord } from 'lightning/uiRecordApi';
import getAccountData from '@salesforce/apex/AccountController.getAccountData';

const FIELDS = [
    'Account.Name',
    'Account.Industry',
    'Account.AnnualRevenue',
    'Account.BillingCity'
];

export default class MyComponent extends LightningElement {
    @api recordId;
    @track fields = FIELDS;

    // Wire to get record data
    @wire(getRecord, { recordId: '$recordId', fields: FIELDS })
    account;

    // Wire to Apex method
    @wire(getAccountData, { accountId: '$recordId' })
    wiredAccountData({ error, data }) {
        if (data) {
            // Handle data
            console.log('Account data:', data);
        } else if (error) {
            // Handle error
            this.showToast('Error', error.body.message, 'error');
        }
    }

    handleSubmit(event) {
        event.preventDefault(); // Prevent default submit
        const fields = event.detail.fields;

        // Custom validation
        if (!fields.Name) {
            this.showToast('Error', 'Name is required', 'error');
            return;
        }

        // Submit the form
        this.template.querySelector('lightning-record-form').submit(fields);
    }

    handleSuccess(event) {
        const updatedRecord = event.detail.id;
        this.showToast('Success', 'Account updated successfully', 'success');
    }

    handleError(event) {
        const error = event.detail;
        this.showToast('Error', 'Error updating account', 'error');
    }

    showToast(title, message, variant) {
        const evt = new ShowToastEvent({
            title: title,
            message: message,
            variant: variant,
        });
        this.dispatchEvent(evt);
    }
}
```

### Meta XML
```xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>64.0</apiVersion>
    <isExposed>true</isExposed>
    <targets>
        <target>lightning__RecordPage</target>
        <target>lightning__AppPage</target>
        <target>lightning__HomePage</target>
    </targets>
    <targetConfigs>
        <targetConfig targets="lightning__RecordPage">
            <objects>
                <object>Account</object>
            </objects>
        </targetConfig>
    </targetConfigs>
</LightningComponentBundle>
```

### CSS Styling
```css
/* myComponent.css */
.container {
    padding: 1rem;
}

.error-message {
    color: var(--lwc-colorTextError);
    font-weight: bold;
}

.success-message {
    color: var(--lwc-colorTextSuccess);
}
```

## Calling Apex from LWC

### Apex Controller (Server-Side)
```apex
public with sharing class AccountController {

    @AuraEnabled(cacheable=true)
    public static List<Account> getAccountData(Id accountId) {
        try {
            return [
                SELECT Id, Name, Industry, AnnualRevenue, BillingCity
                FROM Account
                WHERE Id = :accountId
                WITH SECURITY_ENFORCED
                LIMIT 1
            ];
        } catch (Exception e) {
            throw new AuraHandledException(e.getMessage());
        }
    }

    @AuraEnabled
    public static void updateAccountRevenue(Id accountId, Decimal newRevenue) {
        try {
            Account acc = new Account(
                Id = accountId,
                AnnualRevenue = newRevenue
            );
            update acc;
        } catch (Exception e) {
            throw new AuraHandledException(e.getMessage());
        }
    }
}
```

### Call Imperatively in LWC
```javascript
import updateAccountRevenue from '@salesforce/apex/AccountController.updateAccountRevenue';

async handleUpdateRevenue() {
    try {
        await updateAccountRevenue({
            accountId: this.recordId,
            newRevenue: 1000000
        });
        this.showToast('Success', 'Revenue updated', 'success');
    } catch (error) {
        this.showToast('Error', error.body.message, 'error');
    }
}
```

## Component Communication

### Parent to Child (Public Property)
```javascript
// child.js
import { LightningElement, api } from 'lwc';

export default class Child extends LightningElement {
    @api message; // Public property
    @api recordId;

    @api
    refreshData() {
        // Public method
        console.log('Refreshing data...');
    }
}
```

```html
<!-- parent.html -->
<template>
    <c-child message={parentMessage} record-id={accountId}></c-child>
</template>
```

### Child to Parent (Custom Event)
```javascript
// child.js
handleClick() {
    const event = new CustomEvent('selected', {
        detail: { recordId: this.recordId }
    });
    this.dispatchEvent(event);
}
```

```html
<!-- parent.html -->
<template>
    <c-child onselected={handleSelection}></c-child>
</template>
```

```javascript
// parent.js
handleSelection(event) {
    const recordId = event.detail.recordId;
    console.log('Selected record:', recordId);
}
```

### Publish-Subscribe Pattern (Lightning Message Service)
```javascript
// publisher.js
import { LightningElement, wire } from 'lwc';
import { publish, MessageContext } from 'lightning/messageService';
import ACCOUNT_SELECTED_CHANNEL from '@salesforce/messageChannel/AccountSelected__c';

export default class Publisher extends LightningElement {
    @wire(MessageContext)
    messageContext;

    handleAccountSelect(accountId) {
        const payload = { recordId: accountId };
        publish(this.messageContext, ACCOUNT_SELECTED_CHANNEL, payload);
    }
}
```

```javascript
// subscriber.js
import { LightningElement, wire } from 'lwc';
import { subscribe, MessageContext } from 'lightning/messageService';
import ACCOUNT_SELECTED_CHANNEL from '@salesforce/messageChannel/AccountSelected__c';

export default class Subscriber extends LightningElement {
    @wire(MessageContext)
    messageContext;

    subscription = null;
    selectedAccountId;

    connectedCallback() {
        this.subscribeToMessageChannel();
    }

    subscribeToMessageChannel() {
        this.subscription = subscribe(
            this.messageContext,
            ACCOUNT_SELECTED_CHANNEL,
            (message) => this.handleMessage(message)
        );
    }

    handleMessage(message) {
        this.selectedAccountId = message.recordId;
    }
}
```

## Data Table Pattern

```html
<!-- dataTableComponent.html -->
<template>
    <lightning-card title="Account List" icon-name="standard:account">
        <div class="slds-m-around_medium">
            <lightning-datatable
                key-field="Id"
                data={accounts}
                columns={columns}
                onrowselection={handleRowSelection}>
            </lightning-datatable>
        </div>
    </lightning-card>
</template>
```

```javascript
// dataTableComponent.js
import { LightningElement, wire } from 'lwc';
import getAccounts from '@salesforce/apex/AccountController.getAccounts';

const COLUMNS = [
    { label: 'Name', fieldName: 'Name', type: 'text' },
    { label: 'Industry', fieldName: 'Industry', type: 'text' },
    { label: 'Revenue', fieldName: 'AnnualRevenue', type: 'currency' },
    {
        label: 'Website',
        fieldName: 'Website',
        type: 'url',
        typeAttributes: { label: { fieldName: 'Name' }, target: '_blank' }
    }
];

export default class DataTableComponent extends LightningElement {
    accounts = [];
    columns = COLUMNS;

    @wire(getAccounts)
    wiredAccounts({ error, data }) {
        if (data) {
            this.accounts = data;
        } else if (error) {
            console.error('Error loading accounts:', error);
        }
    }

    handleRowSelection(event) {
        const selectedRows = event.detail.selectedRows;
        console.log('Selected rows:', selectedRows);
    }
}
```

## Form Validation Pattern

```html
<!-- formComponent.html -->
<template>
    <lightning-card title="Create Account">
        <div class="slds-m-around_medium">
            <lightning-input
                label="Account Name"
                value={accountName}
                onchange={handleNameChange}
                required>
            </lightning-input>

            <lightning-combobox
                label="Industry"
                value={industry}
                placeholder="Select Industry"
                options={industryOptions}
                onchange={handleIndustryChange}>
            </lightning-combobox>

            <lightning-input
                type="number"
                label="Annual Revenue"
                value={revenue}
                onchange={handleRevenueChange}
                formatter="currency">
            </lightning-input>

            <div class="slds-m-top_medium">
                <lightning-button
                    variant="brand"
                    label="Save"
                    onclick={handleSave}
                    disabled={isSaveDisabled}>
                </lightning-button>
            </div>
        </div>
    </lightning-card>
</template>
```

```javascript
// formComponent.js
import { LightningElement, track } from 'lwc';
import { ShowToastEvent } from 'lightning/platformShowToastEvent';
import createAccount from '@salesforce/apex/AccountController.createAccount';

export default class FormComponent extends LightningElement {
    @track accountName = '';
    @track industry = '';
    @track revenue = 0;

    industryOptions = [
        { label: 'Technology', value: 'Technology' },
        { label: 'Finance', value: 'Finance' },
        { label: 'Healthcare', value: 'Healthcare' },
        { label: 'Manufacturing', value: 'Manufacturing' }
    ];

    handleNameChange(event) {
        this.accountName = event.target.value;
    }

    handleIndustryChange(event) {
        this.industry = event.detail.value;
    }

    handleRevenueChange(event) {
        this.revenue = event.target.value;
    }

    get isSaveDisabled() {
        return !this.accountName || !this.industry;
    }

    async handleSave() {
        try {
            await createAccount({
                name: this.accountName,
                industry: this.industry,
                revenue: this.revenue
            });

            this.showToast('Success', 'Account created successfully', 'success');
            this.resetForm();
        } catch (error) {
            this.showToast('Error', error.body.message, 'error');
        }
    }

    resetForm() {
        this.accountName = '';
        this.industry = '';
        this.revenue = 0;
    }

    showToast(title, message, variant) {
        const event = new ShowToastEvent({
            title: title,
            message: message,
            variant: variant
        });
        this.dispatchEvent(event);
    }
}
```

## Navigation Pattern

```javascript
import { LightningElement, api } from 'lwc';
import { NavigationMixin } from 'lightning/navigation';

export default class NavigationComponent extends NavigationMixin(LightningElement) {
    @api recordId;

    navigateToRecord() {
        this[NavigationMixin.Navigate]({
            type: 'standard__recordPage',
            attributes: {
                recordId: this.recordId,
                objectApiName: 'Account',
                actionName: 'view'
            }
        });
    }

    navigateToList() {
        this[NavigationMixin.Navigate]({
            type: 'standard__objectPage',
            attributes: {
                objectApiName: 'Account',
                actionName: 'list'
            },
            state: {
                filterName: 'Recent'
            }
        });
    }

    navigateToNewRecord() {
        this[NavigationMixin.Navigate]({
            type: 'standard__objectPage',
            attributes: {
                objectApiName: 'Account',
                actionName: 'new'
            }
        });
    }
}
```

## Best Practices

1. **Use Lightning Data Service (LDS)** when possible for automatic caching and synchronization
2. **Minimize Apex calls** - use @wire for cacheable data
3. **Handle errors gracefully** - always include error handling
4. **Use @api decorators** for public properties and methods
5. **Follow naming conventions** - camelCase for properties, PascalCase for components
6. **Keep components small and focused** - single responsibility principle
7. **Test with Lightning Testing Service (LTS)** or Jest
8. **Use SLDS** (Salesforce Lightning Design System) for consistent styling
