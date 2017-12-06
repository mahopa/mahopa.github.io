---
title:  "Testing non-writeable fields and objects in Apex"
date:   2017-12-05 20:53:21 -0600
categories: Salesforce Apex testing
---
Salesforce makes it difficult to test objects with non-writeable fields. Object history records fall into this category.

Take the example code below.

{% highlight java %}
List<ContactHistory> contactHistories = [SELECT Id, Field, NewValue, OldValue];

{% endhighlight %}

