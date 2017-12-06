---
title:  "Testing non-writeable fields and objects in Apex"
date:   2017-12-05 20:53:21 -0600
categories: Salesforce Apex testing
---

## Problem

Salesforce makes it difficult to test objects with non-writeable fields. Object history records fall into this category.

Take the example code below.

<pre>
<code class="language-java">
public class ContactHistoryMagic {

  public List<Change> convertContactHistoriesToChanges(List<String> contactIds) {
    
    List<ContactHistory> contactHistories = [
      SELECT Id, 
        Field, 
        NewValue, 
        OldValue
      FROM ContactHistory
      IN :contactIds
    ];

    List<Change> changes = new List<Change>();

    for(ContactHistory c: contactHistories){
      //Do stuff
    }

    return changes;
  }
}
</code>
</pre>

Lets take a couple looks at options to test this code.

### Update a contact and then retrieve the history

<pre>
<code class="language-java">

ContactHistoryMagicTest{
  private static void testConvertContactHistoriesToChanges(){
    ContactHistory contactHistory = new ContactHistory();
    contactHistory.
  }
}
</code>
</pre>

Unfortunately this doesn't work. The contactHistory entries aren't created until after the test completes. It's hard to test what isn't there. :-(

### Create the object, insert, test

<pre>
<code class="language-java">
ContactHistoryMagicTest{
  private static void testConvertContactHistoriesToChanges(){
    ContactHistory contactHistory = new ContactHistory();
    contactHistory.
  }
}
</code>
</pre>

This is no good either. The objects fields aren't writable, so we can't create it, let alone insert it.

## Solution

To solve the issue, we need to do two things. Refactor the code to separate the business logic from SOQL retrieval and figure out a way to create a mock of the object.

### Let's start with the mock

Sometimes the best mock of an object is the object itself.

I know, I know. The fields aren't writeable, it won't let us build up the object. ContactHistory fields aren't writable, but sObject fields are ;-).

<pre>
<code class="language-java">
ContactHistoryMagicTest{
  private static void testConvertContactHistoriesToChanges(){
    
    //Polymorphism is awesome
    sObject contactHistory = new ContactHistory();
    
    //Update the needed fields
    contactHistory.put('NewValue', 'this value is new');
    contactHistory.put('OldValue', 'this value is old');
    
    //Cast the the contact history back to the correct type
    ContactHistory contactHistory2 = (ContactHistory) contactHistory;

  }
}
</code>
</pre>

Surprisingly this works. There is a caveat. Inserting the records will cause a runtime error. That means we'll need to separate the SOQL that retrieves these records from the logic that manipulates them.

<pre>
<code class="language-java">
public class ContactHistoryMagic {

  public List<Change> convertContactHistoriesToChanges(List<String> contactIds) {
    
    List<ContactHistory> contactHistories = [
      SELECT Id, 
        Field, 
        NewValue, 
        OldValue
      FROM ContactHistory
      IN :contactIds
    ];

    List<Change> changes = convertContactHistoriesToChange(contactHistories);

    return changes;
  }

  List<Change> convertContactHistoriesToChange(List<ContactHistory> contactHistories){
      List<Change> changes = new List<Change>();
      
      for(ContactHistory c: contactHistories){
        //Do stuff
      }

      return changes;
  }
}
</code>
</pre>

By pulling the logic into a separate method, we can test the logic on it's own. Better separation of concerns, and more importantly we can use tests to make sure our code is behaving as expected.

Do we get 100% test coverage, no. We do get a chance to cover any complex logic on objects with writeable fields.