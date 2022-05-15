+++
title = "Using a Custom iterator on a Batch"
date = 2022-05-14T10:49:09+12:00
description = "What I figured from writing a Custom Iterator for a Batch Job" 
draft = false
toc = false
categories = ["Salesforce"]
tags = ["Apex", "Batch Class", "Custom Iterator"]
images = [
"https://source.unsplash.com/collection/983219/1600x900"
] # overrides site-wide open graph image
[[copyright]]
owner = "prasanthabr"
date = "2022"
license = "cc-by-sa-4.0"
+++
tl; dr: There are always creative ways to leverage tools; this mostly comes from experience.
Custom Iterators take a bit to wrap your head around, but at its simplest, there is an Iterator and there is an Iterable.
The Iterable feeds in the list that allows the iterator to pick out the next grouping.

__Learnings__
1. The trick is to start with the iterating element; Eg: in a batch on a list of Accounts, the element is Account
2. Creatively use constructors to help define behaviors in various situations
3. Think about what the unit of processing is for you; if we are looking at working on a list as a unit, then the element should somehow contain that list
4. Biggest learning, in a batch if you find it difficult to come from the detail and group, see if you cna come from the top and query in execute to group.


### Background References
The requirement was to identify failures in a row for each test type for collections across a date range, for a given entity.

This can be imagined as a three level problem. This means that the grouping has to occur at three levels. When building out the data from the lowest level, it means we would have to have a custom iterator and custom iteration to support the grouping.

```java
public class ConsecutiveItemFailureBatch{

    public class BusinessFinder implements Iterator<BusinessTestQuality>{//this is our iterator
        Map<String, BusinessTestQuality> completeSet;
        String key;

       
        public boolean hasNext(){
            return true; //<-- this is being mocked
        }

        public BusinessTestQuality next(){
            return new BusinessTestQuality(); //<-- this is being mocked
        }

    }

    public class BusinessSet implements Iterable<BusinessTestQuality> {

       public Iterator<BusinessTestQuality> iterator(){
        Map<String, BusinessTestQuality> completeSet;
        String key;

        for (Item_Quality__c mq :
            [Select id, Name, Collection_Pickup_Date__c, Test_Result_Numeric__c, Business_No__r.Id, DA_Consecutive_Test_Failure__c  
            from Item_Quality__c
            where
            (Collection_Pickup_Date__c = LAST_N_DAYS:7 and (Name = :Test1 OR
            Name = :Test3))
            OR
            (Name = :Test2 and Collection_Pickup_Date__c = LAST_N_DAYS:10)
            order by Collection_Pickup_Date__c asc]){

                key = mq.Name + '-' + mq.Business_No__r.Id;

                if (completeSet.get(key) == null){
                    BusinessTestQuality ftq = new BusinessTestQuality(mq);
                    completeSet.put(key,ftq);
                } else {
                    completeSet.get(key).collect(mq);
                }

            }

        return new BusinessFinder(); //this needs to be replaced.

       }
    }

    public class BusinessTestQuality{ // This is the stuff that is holding the grouping
        List<Item_Quality__c> ItemQualities;
        String Business;
        String test;
        Integer count;

        //How do we create this bugger?

        public BusinessTestQuality(){}

        public BusinessTestQuality(Item_Quality__c ItemQuality){
            this.Business = ItemQuality.Business_No__c;
            this.test = ItemQuality.Name; //this may have to change to be the test itself. but this should suffice for now.
            this.count = 1;
            this.ItemQualities.add(ItemQuality);
        } //we expect a Item quality every single time
       
        public void collect(Item_Quality__c ItemQuality){
            this.ItemQualities.add(ItemQuality);
            this.count++;
            if(this.Business == null){
                this.Business = ItemQuality.Business_No__c;
                this.test = ItemQuality.Name;
            }
        }
    }
}
```