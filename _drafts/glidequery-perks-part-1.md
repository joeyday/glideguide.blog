---
layout: post
title: 'GlideQuery Perks, Part 1: JavaScript not Java'
author: Joey
date: 2023-01-01
categories: glidequery
---

One of the main points I made in [Part 0](/2023/01/30/glidequery-perks-part-0.html) of this series was that GlideRecord is a Java object and its various methods return Java types and more Java objects, so it doesn't behave like native JavaScript. In contrast, though GlideQuery does call GlideRecord to perform database operations under the hood, its public interface is written entirely in JavaScript and was intentionally designed to behave like JavaScript.

## JavaScript native variable types

A big part of how GlideQuery behaves like JavaScript is its consistent use of JavaScript native types. For example, if you ask GlideQuery to return the contents of an Integer or Decimal column in the ServiceNow database, GlideQuery will give you the value as a native JavaScript number. If you ask it to return the contents of a True/False field, you'll get a native JavaScript boolean.

Let's compare how different types of values get returned by GlideRecord. I have some examples in Part 0 showing how I sussed this out with the `instanceof` operator (see also `JSUtil.instance_of()`).

| Column type | Accessed directly | Accessed with `getValue()` |
|:------------|:-----------------|:----------------|
| String | GlideElement / Java string | JavaScript string |
| Integer | GlideElementNumeric / Java string | JavaScript string |
| True/False | GlideElementBoolean / Java boolean | JavaScript string (0 or 1) |

It's honestly pretty odd, right? If we don't use `getValue()`, sometimes we get the right type, but actually not really since they're Java types instead of native JavaScript types. If we do use `getValue()`, we consistently get a native JavaScript string but then we have to cast it to whichever type we need.

Now let's directly inspect some values from GlideQuery:

~~~ javascript
var inc = new GlideQuery('incident')
  .get(exampleID, ['description', 'sys_mod_count', 'active'])
  .get();

typeof inc.desciption === 'string';     // true
typeof inc.sys_mod_count === 'number';  // true
typeof inc.active === 'boolean';        // true
~~~

As you can see, refreshingly, the various kinds of table fields just come to us as their appropriate native JavaScript types. They're not Java types nor are they Java objects. GlideElement, GlideElementNumeric, and GlideElementBoolean are nowhere to be found here.

### GlideRecord is more sane than I thought

Years ago I adopted the best practice of just always using `getValue()` whenever interacting with columns on a GlideRecord object (and [many](https://snprotips.com/blog/2017/4/9/always-use-getters-and-setters) [other](https://www.servicenow.com/community/developer-articles/gliderecord-hints-tips-common-issues-and-good-practices/ta-p/2323766) [people](https://www.servicenow.com/community/developer-forum/get-field-value-of-gliderecord-best-practice-method/m-p/1619218) [have](https://www.youtube.com/live/j57cXWGQD98?feature=share&t=1025) [also](https://www.servicenow.com/community/developer-blog/tnt-the-importance-of-using-quot-getvalue-quot-when-getting-data/ba-p/2273338), or have advocated for some [alternative](https://codecreative.io/blog/is-gliderecord-getvalue-the-king-of-the-string/)). But in researching this series I've come to realize GlideElement objects actually behave more intuitively than I remembered.

Because Integer and Decimal columns are evaluated to GlideElementNumeric objects, you can do math with them directly. (I had thought, surely, since `instanceof` reports them as Java strings, addition would be evaluated as string concatenation, but no, it works fine.)

~~~ javascript
var gr = new GlideRecord('alm_asset');
gr.get('model.display_name', 'Acer Notebook Battery');

gr.quantity;       // 3
gr.quantity + 10;  // '310'? Nope, 13
~~~

Similarly, because Boolean columns come out as GlideElementBoolean objects (and Java booleans), they work fine in logic operations and conditionals.

~~~ javascript
var gr = new GlideRecord('incident');
gr.addInactiveQuery();

if (gr.active) {
  // Stuff in here won't execute
  // ...
}
~~~

So it turns out I've made extra work for myself by always using `getValue()`. I've needlessly casted a lot of strings to numbers or tested them for equality with `'0'` and `'1'`, when I could've (and arguably should've) just been directly accessing the values I needed.

But there are two significant issues with GlideRecord and GlideElement objects that drove us all to adopt `getValue()` in the first place, and they're both handily avoided by GlideQuery's use of native types.

### No pass-by-reference shenanigans

Since column values returned by GlideQuery are just JavaScript native types and not GlideElement objects or Java types, they always get passed-by-value, preventing unexpected pass-by-reference behavior.

~~~ javascript
var result = [];

new GlideQuery('incident')
  .select('number')
  .forEach(function (inc) {
    result.push(inc.number);
  });

result;  // ['INC0010001', 'INC0010002', 'INC0010003', ...]
~~~

We already know using `getValue()` with GlideRecord avoids this bug, but GlideQuery shines here for not having the problem in the first place.

(Note there are better ways to load values into an array with GlideQuery, so don't follow this example—I'll give you a better one in Part 3 of the series. Also note the real issue is pass-by-reference combined with the way GlideRecord objects mutate when the `next()` method is called, as I explained in Part 0. GlideQuery not only avoids pass-by-reference but also any kind of object mutation, and that's another thing I'll cover in Part 3.)

### Strict comparisons are possible

Another benefit of GlideQuery's native type return values is they can safely be strictly compared with other values.

~~~ javascript
var inc = new GlideQuery('incident')
  .getBy({ number: 'INC0010005' }, [ 'number' ])
  .get();

inc.number == 'INC0010005';   // true
inc.number === 'INC0010005';  // true
~~~

As I mentioned in Part 0, GlideRecord's inconsistent behavior here made me distrust strict equality in JavaScript in my early ServiceNow days. And, continuing the pattern, using `getValue()` and/or casting to the appropriate native type avoids the problem, but GlideQuery just hands over the value in the correct native type. Easy peasy.

## JavaScript native objects

In addition to column values being native types, even whole objects returned by GlideQuery are native JavaScript objects. I can think of three reasons why this is useful (and you can probably think of more).

### Easier debugging

Maybe the simplest implication of GlideQuery returning native objects is they can be logged out for quick and easy debugging.

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

### Easier JSON serialization

Another nice use case for this is that objects returned by GlideQuery can be directly serialized into JSON for sending across a REST connection. How many Scripted REST APIs have you written where you had to copy data out of a GlideRecord object into a native object before it could be serialized? Consider this example I found in ServiceNow's documentation (["Scripted REST API example - streaming vs object serialization"](https://docs.servicenow.com/bundle/tokyo-application-development/page/integrate/custom-web-services/reference/r_ScriptedRESTExampleStreamVsLO.html)):

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

This is much nicer to implement with GlideQuery. Since the objects are already native objects, we can push each one onto the array as is and return the array with full confidence it can be safely stringified into JSON.

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

(As before, note this is a poor way to create arrays with GlideQuery. I kept this similar to the GlideRecord example for a more apples-to-apples comparison. Keep an eye out for Part 3 for some vastly improved GlideQuery array building examples.)

### Native object methods

Yet another benefit to GlideQuery's native objects is all of JavaScript's object methods can be used such as `hasOwnProperty()`. We can also use `Object.keys()` to get just the field names from the object, or we can use a `for...in` loop to iterate over all the properties.

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

## Conclusion

Because GlideQuery behaves so much more predictably in always returning JavaScript native types and native objects, a lot of confusion and potential bugs are avoided. We get the simple convenience of rarely if ever having to convert values and objects into other kinds of values and objects, and we get the benefits of JavaScript native methods for the various types. I know I would've more intuitively understood and had a lot more trust in the platform in my early years if I'd cut my teeth scripting with GlideQuery instead of GlideRecord.{% include endmark.html %}

