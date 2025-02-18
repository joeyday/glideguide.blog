---
layout: post
title: 'GlideQuery Perks, Part 2: JavaScript Not Java'
author: Joey
date: 2024-09-07
categories:
 - GlideQuery
 - GlideQuery Perks Series
---

<span class="lead">Another point I made in [Part 0](/2023/01/30/glidequery-perks-part-0.html) of this series</span> was that GlideRecord is a Java object and its various methods return Java types and more Java objects, so it doesn't behave like native JavaScript. In contrast, though GlideQuery uses GlideRecord to perform database operations in its private implementation, its public interface is written entirely in JavaScript and was intentionally designed to behave like JavaScript.

## JavaScript native variable types

A big part of how GlideQuery acts like JavaScript is its consistent use of JavaScript native types. For example, if you ask GlideQuery to return the contents of an Integer or Decimal column in the ServiceNow database, GlideQuery will give you the value as a native JavaScript number. If you ask it to return the contents of a True/False field, you'll get a native JavaScript boolean.

~~~ javascript
var inc = new GlideQuery('incident')
  .get(exampleID, ['description', 'sys_mod_count', 'active'])
  .get();

typeof inc.description === 'string';    // true
typeof inc.sys_mod_count === 'number';  // true
typeof inc.active === 'boolean';        // true
~~~

As you can see, they're not Java types nor are they Java objects. Refreshingly, GlideElement is nowhere to be found here.

### In hindsight, GlideRecord is more sane than I thought

Years ago I adopted the best practice of just always using `getValue()` whenever interacting with columns on a GlideRecord object (and [many](https://snprotips.com/blog/2017/4/9/always-use-getters-and-setters) [other](https://www.servicenow.com/community/developer-articles/gliderecord-hints-tips-common-issues-and-good-practices/ta-p/2323766) [people](https://www.servicenow.com/community/developer-forum/get-field-value-of-gliderecord-best-practice-method/m-p/1619218) [have](https://www.youtube.com/live/j57cXWGQD98?feature=share&t=1025) [also](https://www.servicenow.com/community/developer-blog/tnt-the-importance-of-using-quot-getvalue-quot-when-getting-data/ba-p/2273338), or have advocated for some [alternative](https://codecreative.io/blog/is-gliderecord-getvalue-the-king-of-the-string/)). But in researching this series I've come to realize GlideElement objects actually behave more intuitively than I remembered.

Because Integer and Decimal columns are evaluated to GlideElementNumeric objects, you can do math with them directly. Similarly, because Boolean columns come out as GlideElementBoolean objects (and Java booleans), they work fine in logic operations and conditionals.

But there are two significant issues with GlideRecord and GlideElement objects that drove us all to adopt `getValue()` in the first place, and they're both handily avoided by GlideQuery's use of native types.

### No mutable object shenanigans

As I explained in Part 0, GlideElement objects mutate when the `next()` method is called, producing some unexpected behavior. Since column values returned by GlideQuery are just JavaScript native types (strings, numbers, and booleans) and not GlideElement objects or any other Java types, they're naturally immutable:

~~~ javascript
var result = [];

new GlideQuery('incident')
  .select('number')
  .forEach(function (inc) {
    result.push(inc.number);
  });

result;  // ['INC0010001', 'INC0010002', 'INC0010003', ...]
~~~

(Note there are better ways to load values into an array with GlideQuery, so don't follow this example—I'll give you a better one in Part 4 of the series.)

We already know using `getValue()` turns GlideElement objects into JavaScript primitive types, preventing unexpected behavior, but GlideQuery shines here for not having the problem in the first place. And stay tuned as I'll have more to say about object mutation in Part 3.

### Strict comparisons are possible

Another benefit of GlideQuery's use of native types is they can safely be strictly compared with other values.

~~~ javascript
var inc = new GlideQuery('incident')
  .getBy({ number: 'INC0010005' }, [ 'number' ])
  .get();

inc.number == 'INC0010005';   // true
inc.number === 'INC0010005';  // true
~~~

As I mentioned in Part 0, GlideRecord's inconsistent behavior here made me distrust strict equality in JavaScript in my early ServiceNow days. Continuing the pattern, using `getValue()` and/or casting to the appropriate native type works the same way, but GlideQuery gives us the value in the correct native type without any casting or coercion. Easy peasy.

## JavaScript native objects

In addition to column values being native types, even whole objects returned by GlideQuery are native JavaScript objects. I can think of three reasons why this is useful (and you can probably think of more).

### Easier debugging

Maybe the simplest implication of GlideQuery returning native objects is the whole object can be logged out for quick and easy debugging.

~~~ javascript
var inc = new GlideQuery('incident')
  .get(exampleID, ['number', 'short_description'])
  .get();

gs.debug(inc);

// {
//   "sys_id": "...",
//   "number": "INC0010005",
//   "short_description": "Email server is down"
// }
~~~

### Easier <abbr>JSON</abbr> serialization

Another nice use case for this is that objects returned by GlideQuery can be directly serialized into <abbr>JSON</abbr> for sending across a <abbr>REST</abbr> connection. How many Scripted <abbr>REST</abbr> <abbr>API</abbr>s have you written where you had to copy data out of a GlideRecord object into a native object before it could be serialized? Consider this example I found in ServiceNow's documentation (["Scripted <abbr>REST</abbr> <abbr>API</abbr> example - streaming vs object serialization"](https://docs.servicenow.com/bundle/tokyo-application-development/page/integrate/custom-web-services/reference/r_ScriptedRESTExampleStreamVsLO.html)):

~~~ javascript
(function runOperation(request, response) {
  result_arr = [],
  
  gr = new GlideRecord('incident');
  gr.setLimit(100);
  gr.query();

  // iterate over incident records and build JSON
  // representations to be streamed out
  while (gr.next()) {
    var incidentObj = {};
    incidentObj.number = gr.number + '';
    incidentObj.short_description = gr.short_description + '';
    result_arr.push(incidentObj);
  }
  
  return result_arr;
})(request, response);
~~~

This looks simple enough, but imagine we had a lot more fields to load into the objects to be serialized—the more fields we had, the more lines of code we'd need.

This is much nicer to implement with GlideQuery. Since the objects are already native objects, we can push each one onto the array as is and return the array with full confidence it can be safely stringified into <abbr>JSON</abbr>.

~~~ javascript
(function runOperation(request, response) {
  result = [],

  new GlideQuery('incident')
    .limit(100)
    .select('number', 'short_description')
    .forEach(function (inc) {
      result.push(inc);
    });
  
  return result;
})(request, response);
~~~

(As before, note this is a poor way to create arrays with GlideQuery. I kept this similar to the GlideRecord example for a more apples-to-apples comparison. Again, keep an eye out for better array examples in Part 4.)

### Native object methods

Yet another benefit to GlideQuery's native objects is all of JavaScript's object methods can be used such as `hasOwnProperty()`. We can also use helper functions like `Object.keys()` and features like `for...in` loops.

~~~ javascript
var inc = new GlideQuery('incident')
  .get(exampleID, ['number', 'short_description'])
  .get()

for (var field in inc) {
  gs.debug(field + ': ' + inc[field]);
}

// sys_id: ...
// number: INC0010005
// short_description: Email server is down
~~~

(And if you're lucky enough to be on the Tokyo release or later and in a context where you can use modern JavaScript language features, you might find a use for `Object.entries()` or `Object.values()`.)

## Conclusion

Because GlideQuery behaves so much more predictably in always returning JavaScript native types and native objects, a lot of confusion and potential bugs are avoided. We get the simple convenience of rarely if ever having to convert values and objects into other kinds of values and objects, and we get the benefits of JavaScript native methods for the various types. I'm certain I would've more intuitively understood and had a lot more trust in the platform in my early years if I'd cut my teeth scripting with GlideQuery instead of GlideRecord.{% include endmark.html %}

_Next in the series: [Part 3: Pure Functions, Fluent Method Chaining, and Object Re-usability &rarr;](/2025/02/17/glidequery-perks-part-3.html)_
