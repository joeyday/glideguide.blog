---
layout: post
title: 'GlideQuery Perks, Part 3: Pure Functions, Fluent Method Chaining, and Object Re-usability'
author: Joey
date: 2025-02-15
categories:
 - glidequery
 - glidequery perks series
---

<span class="lead">Compare these bog standard GlideRecord and GlideQuery examples</span> and you'll see something is quite different between them:

~~~ javascript
let gr = new GlideRecord(someTable);
gr.addQuery(someField, someValue);
gr.addNotNullQuery(someOtherField);
gr.query();
while (gr.next()) {
  // do something...
}
~~~

~~~ javascript
new GlideQuery(someTable)
  .where(someField, someValue)
  .whereNotNullQuery(someOtherField)
  .select()
  .forEach((record) => {
  	// do something...
  });
~~~

The difference might be subtle or it might jump out at you right away. They're the same number of lines and they seem to be doing all the same steps in the same order (indeed they are). What is it about GlideRecord and GlideQuery that makes the syntax appear so different, and, if this is important, why is it important?

## GlideRecord is founded on object mutation

In the GlideRecord example, we first instantiate an object and assign it to a variable. From that point on we call methods on the object that cause the object to change. We don't have to assign the output of those method calls to any new variable, because each time we add a new query the object itself changes (it would also change if we used other methods like `setLimit` or `orderBy`). When we execute the query the object changes again, and each time we iterate to the next record the object changes yet again. And the whole time it remains assigned to the same variable name.

These changes in the object are sometimes referred to as mutations or side effects, and it's considered a best practice by some to avoid them. But aren't variables meant to vary? After all, it's right there in the name "variable". What's really wrong with allowing the GlideRecord object to mutate like this? We saw one extreme (and thankfully uncommon) example in [Part 0](/2023/01/30/glidequery-perks-part-0.html) and [Part 2](/2024/09/07/glidequery-perks-part-2.html) where mutation due to the `next` method caused undesirable behavior, but are there other drawbacks?

## GlideQuery objects are immutable

In functional programming there is a design pattern called a pure function. Pure functions always produce the same output given the same input and have no observable side effects. With very few exceptions, the methods involved in using GlideQuery are all pure functions. Each time you call one of the methods, instead of mutating the object, the method leaves the original object unchanged and returns a new object that's like the old object but with the new step applied.

This is cumbersome, but it helps to see what's really going on if we rewrite the GlideQuery example above like this:

~~~ javascript
let query1 = new GlideQuery(someTable)
let query2 = query1.where(someField, someValue)
let query3 = query2.whereNotNullQuery(someOtherField)
let stream = query3.select()
stream.forEach((record) => {
  // do something...
});
~~~

Here you can see clearly that with each new step we get back a new GlideQuery or Stream object (we'll talk more about Stream and Optional objects in Part 4). In this example we're assigning each object to a new variable, then using that variable to call the method for the next step.

## Fluent method chaining

One immediate benefit of this architecture is it enables the fluent method chaining we usually use with GlideQuery. Each chained method call acts on the new object returned from the method call immediately preceding it. We usually don't have any need to keep all those intermediate objects so we don't bother assigning them to variables. This generally leads to shorter and cleaner-looking lines of code and fewer overall keystrokes.

When you chain methods like this, even though the method calls span multiple lines, you're actually creating one long single expression. In some cases, like our first GlideQuery example above, this means you don't need to assign anything to a variable. If you do start with declaring a variable, keep in mind the value assigned to the variable will be the result of the entire expression.

~~~ javascript
let n = new GlideQuery(table)
  .where(someField, someValue)
  .count()
~~~

The GlideQuery object instantiated on the first line never gets assigned to `n`. Instead, the entire method chain is evaluated and, ultimately, the variable will be assigned the return value from the `count` method.

And notice the indentation convention used here. Indenting each of the chained method call lines with a single tab is deliberate and recommended since it reminds us the whole thing is one long expression, not a series of individual expressions or statements.

## Object re-usability

A less obvious but very powerful feature of this pure functional architecture is object re-use. Since the intermediate objects are never mutated, if we do decide to save one by assigning it to a variable, we can go back and re-use it later.






{% include endmark.html %}
