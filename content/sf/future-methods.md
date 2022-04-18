+++
title = "Future Methods"
date = 2022-04-18T10:49:09+12:00
description = "What I figured from Future Method Trailhead"
draft = false
toc = false
categories = ["Salesforce"]
tags = ["Apex", "Future Methods"]
images = [
"https://source.unsplash.com/collection/983219/1600x900"
] # overrides site-wide open graph image
[[copyright]]
owner = "prasanthabr"
date = "2022"
license = "cc-by-sa-4.0"
+++

This was a simple test to count the number of contacts associated with a given account.  
The task was merely to create a class and not have a triggering mechanism.  
Helped me update my understanding when the test was merely to call the class and method directly. I think the attempts at building go skills were helpful here.

### Considerations in Future Methods

Future methods can be thought similar to fire and forget actions.  
These methods are self sufficient and contains all functions in the context of the future method.

1. The method can accept only primitive data types  
   A. Does not accept sobjects
1. The method is static --> I think it means that there cannot be instances here, need to read and understand.
1. The method does not return anything at all. In line with the idea of being self contained.
1. The future method has a decorator called callout that can be used. I am yet to explore how that is different
1. Testing future methods are interesting. Wrapping them in Test.Start() and Test.End()
1. Also wrapped my head around calling the method directly to test it.
1. Future methods do not follow the order in which they are triggered
1. Future methods can run in parallel, which may cause row locking issues. While i have not explored this much, but this is where a queue patten makes sense to safely fail in case of row locking

#### Other things to look at

1. Objects that cannot be used together in DML Ops. This came about since this is a common use case for future methodshttps://developer.salesforce.com/docs/atlas.en-us.236.0.apexcode.meta/apexcode/apex_dml_non_mix_sobjects.htm
1. In regards to the code/use case below; One of the approaches I had taken was to have a count() for the contacts. However, you cannot have a count() (is this called aggregate function) in a sub query

### Class Code

```java
public with sharing class AccountProcessor {


    @future
    public static void countContacts(List<Id> accountids){

        List<Account> accounts = new List<Account>([Select id, Number_Of_Contacts__c, (Select id from contacts) from account where id in :accountids]);

        for (Account x : accounts){
            x.Number_Of_Contacts__c = 0;
            for (contact y: x.Contacts){
                x.Number_Of_Contacts__c++;
            }
        }

        update accounts;

    }
}

```

### Test Class Code

```java
/*
Test for the Account Processor class
*/

@isTest
private class AccountProcessorTest {

    @IsTest
    static void contactAccount(){

        Map<id, Account> testAccounts = new Map<id, Account>([Select id, Number_Of_Contacts__c from Account]);
        List<id> accids = new List<id>(testAccounts.keySet());

        Test.startTest();
        AccountProcessor.countContacts(accids);
        Test.stopTest();

        Integer count = 0;

        for (Account acc : [Select id, Number_Of_Contacts__c from Account order by id asc]){
            System.AssertEquals(acc.Number_Of_Contacts__c,count,count+' is failing');
            count++;

        }
    }

    @TestSetup
    static void makeData(){
        List<Account> testAcc = new List<Account>();
        List<Contact> testCon = new List<contact>();
        Integer count = 10;

        for (Integer i=0; i<count; i++){
            testAcc.add(new Account(Name = 'TestAcc'+i));
            }


        insert testAcc;

        for (Integer x =0; x<count; x++){
            for (Integer y = 0; y < x; y++){
            testCon.add(new Contact(LastName='LastName'+x+y,Email='prasanthabr+'+x+y+'@gmail.com',AccountId=testAcc[x].Id));
            }

}

insert testCon;

}
}
```
