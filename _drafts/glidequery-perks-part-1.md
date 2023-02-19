---
layout: post
title: 'GlideQuery Perks, Part 1: JavaScript not Java'
author: Joey
date: 2023-01-01
categories: glidequery
---

One of the main points I made in [Part 0](/2023/01/30/glidequery-perks-part-0.html) of this series was that GlideRecord is a Java object and its various methods return Java types and more Java objects, so it doesn't behave like native JavaScript. In contrast, though GlideQuery does call GlideRecord to perform database operations under the hood, its public interface is written entirely in JavaScript and was intentionally designed to behave like JavaScript.

## JavaScript native types

A big part of how GlideQuery behaves like JavaScript is its consistent use of JavaScript native types. For example, if you ask GlideQuery to return the contents of an Integer or Decimal column in the ServiceNow database, GlideQuery will give you the value as a native JavaScript number. If you ask it to return the contents of a True/False field, you'll get a native JavaScript boolean.

Let's compare how different types of values get returned by GlideRecord:

| Column type | Accessed directly | Accessed with `getValue` |
|:------------|:-----------------|:----------------|
| String | GlideElement / Java string | JavaScript string |
| Integer | GlideElementNumeric / Java string | JavaScript string |
| True/False | GlideElementBoolean / Java boolean | JavaScript string (0 or 1) |

It's honestly pretty odd, right? If we don't use `getValue`, sometimes we get the right type, but actually not really since they're Java types instead of native JavaScript types. If we do use `getValue`, we always get a native JavaScript string but then we have to cast it to whichever type we need.

You can use `typeof` and `instanceof` (see also `JSUtil.instance_of`) to inspect this behavior for yourself. Here's an example:

~~~ javascript
var inc = new GlideRecord('incident');
inc.get('sys_id', exampleID);
inc.sys_mod_count instanceof GlideElementNumeric;        // true
inc.sys_mod_count instanceof Packages.java.lang.String;  // true
typeof inc.getValue('sys_mod_count') === 'string';       // true
typeof inc.sys_mod_count === 'number';                   // false ✗

var inc = new GlideQuery('incident')
  .get(exampleID, ['sys_mod_count'])
  .get()
inc.sys_mod_count instanceof GlideElement;               // false
inc.sys_mod_count instanceof Packages.java.lang.String;  // false
typeof inc.getValue('sys_mod_count') === 'string';       // false
typeof inc.sys_mod_count === 'number';                   // true ✓
~~~

## GlideRecord is more sane than I thought

Years ago I adopted the best practice of just always using `getValue` whenever interacting with columns on a GlideRecord object. But in researching this series I've come to realize GlideElement objects actually behave better than I remembered.

Because Integer and Decimal columns are evaluated to GlideElementNumeric objects, you can do math with them directly. I thought surely, since `instanceof` reports them as Java strings, addition would be evaluated as string concatenation, but no, it works fine.

~~~ javascript
var gr = new GlideRecord('alm_asset');
gr.get('model.display_name', 'Acer Notebook Battery');
gr.quantity;       // 3
gr.quantity + 10;  // 310? Nope, it's 13 ✓
~~~

Similarly, because Boolean columns come out as GlideElementBoolean objects (and Java booleans), they work fine in boolean logic operations and conditionals.

~~~ javascript
var gr = new GlideRecord('incident');
gr.get('active', true);
if (gr.active) {
  // Stuff in here will execute
  // ...
}

var gr = new GlideRecord('incident');
gr.get('active', false);
if (gr.active) {
  // Stuff in here will _not_ execute
  // ...
}
~~~

So it turns out I've given myself extra work by always using `getValue`. I've needlessly casted a lot of strings to numbers and tested a lot of strings for equality with `'0'` and `'1'`, when I could've and arguably should've just been directly accessing the values I needed. But there are two real issues with GlideRecord and GlideElement objects that drove me to adopt `getValue` in the first place, and they're both handily avoided by GlideQuery's use of native types.

## No pass-by-reference shenanigans

Since column values returned by GlideQuery are just JavaScript native types and not GlideElement objects or Java types, they always get passed-by-value which prevents unexpected pass-by-reference behavior.

~~~ javascript
var gr_array = [];
var inc = new GlideRecord('incident')
inc.query();
while (inc.next()) {
  gr_array.push(inc.number);
}
gr_array;  // ['INC0010019', 'INC0010019', 'INC0010019'] ✗

var gq_array = [];
new GlideQuery('incident')
  .select('number')
  .forEach(function (inc) {
    gq_array.push(inc.number);
  });
gq_array;  // ['INC0010004', 'INC0010005', 'INC0010019'] ✓
~~~

As I said, using `getValue` avoids this bug, but GlideQuery shines here for not having the problem in the first place.

(Note there are better ways to load values into an array with GlideQuery, so don't follow this example—I'll give you a better one in Part 3 of the series. Also note the real issue here is pass-by-reference combined with the way GlideRecord objects mutate when the `next` method is called, as I explained in Part 0. GlideQuery not only avoids pass-by-reference but also any kind of object mutation, but that's another thing I'll cover in Part 3.)

## Strict comparisons are possible

Because GlideQuery's returned values are native JavaScript types, they can safely be strictly compared with other values of the same type.

~~~ javascript
var inc = new GlideRecord('incident');
inc.get('number', 'INC0010004');
inc.number == 'INC0010004';   // true
inc.number === 'INC0010004';  // false ✗

var inc = new GlideQuery('incident')
  .getBy({ number: 'INC0010004' }, [ 'number' ])
  .get();
inc.number == 'INC0010004';   // true
inc.number === 'INC0010004';  // true ✓
~~~

As I mentioned in Part 0, GlideRecord's inconsistent behavior here made me distrust strict equality in JavaScript in my early ServiceNow days. And, continuing the pattern, using `getValue` and/or casting to the appropriate native type avoids the problem, but GlideQuery just hands over the value in the correct native type. Easy peasy.

## Conclusion

Because GlideQuery behaves so much more predictably in always returning JavaScript native types, a lot of confusion and potential bugs are just removed from the picture. I know I would've understood and trusted the platform a lot more in my early years if I'd cut my teeth scripting with GlideQuery instead of GlideRecord.{% include endmark.html %}

