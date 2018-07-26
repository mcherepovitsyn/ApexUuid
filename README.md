# Apex UUID
[![Build Status](https://travis-ci.org/jongpie/ApexUuid.svg?branch=master)](https://travis-ci.org/jongpie/ApexUuid)

<a href="https://githubsfdeploy.herokuapp.com" target="_blank">
    <img alt="Deploy to Salesforce" src="https://raw.githubusercontent.com/afawcett/githubsfdeploy/master/deploy.png">
</a>

## Usage
Provides a way to generate a [UUID (Universally Unique Identifier)](https://en.wikipedia.org/wiki/Universally_unique_identifier) in Salesforce's Apex language. This uses Verion 4 of the UUID standard - more details available [here](https://en.wikipedia.org/wiki/Universally_unique_identifier#Version_4_(random))

### Generating & using a UUID
To generate a UUID, simply instantiate a new instance. The string value can be retrieved with `getValue()`
```
Uuid myUuid = new Uuid();
String myUuidValue = myUuid.getValue();
```

You can use the UUID value as a unique ID, generated by Apex. This lets you do some powerful things - for example, you can it as an external ID for inserting parent & child records with only 1 DML statement.

> ##### Example: Using a code-generated external ID (such as a UUID), we can create multiple accounts and related contacts with only 1 DML statement

```
List<Sobject> recordsToCreate = new List<Sobject>();
// Create 10 sample accounts
for(Integer accountCount = 0; accountCount < 10; accountCount++) {
    // Create a new account & set a custom external ID text field called Uuid__c
    Account newAccount = new Account(
        Name    = 'Account ' + accountCount,
        Uuid__c = new Uuid().getValue()
    );
    recordsToCreate.add(newAccount);

    // For each sample account, create 10 sample contacts
    for(Integer contactCount = 0; contactCount < 10; contactCount++) {
        // Instead of setting contact.AccountId with a Salesforce ID...
        // we can use an Account object with a Uuid__c value to set the Contact-Account relationship
        Contact newContact = new Contact(
            Account  = new Account(Uuid__c = newAccount.Uuid__c),
            LastName = 'Contact ' + contactCount
        );
        recordsToCreate.add(newContact);
    }
}
// Sort so that the accounts are created before contacts (accounts are the parent object)
recordsToCreate.sort();
insert recordsToCreate;

// Verify that we only used 1 DML statement
System.assertEquals(1, Limits.getDMLStatements());
```

### Using a UUID's string value
If you already have a UUID as a string (previously generated in Apex, generated by an external system, etc), there are 3 static methods to help work with the string value.
1. **Do I have a valid UUID value?**

    This checks if the string value matches the regex for a UUID v4
    ```
    Boolean isValid = Uuid.isValid(myUuidValue);
    ```
2. **I have a UUID value but need to format it**

    This returns a formatted string that follows the UUID pattern: 8-4-4-4-12 in lowercase
    ```
    String formattedUuidValue = Uuid.formatValue(myUnformattedUuidValue);
    ```
3. **I have a UUID value, how can I use it to construct a UUID?**

    This will automatically format the value for you, but the intial value must be a valid (unformatted) UUID string
    ```
    Uuid myUuid = Uuid.valueOf(myUuidValue);
    ```