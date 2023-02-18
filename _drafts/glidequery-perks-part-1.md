---
layout: post
title: 'GlideQuery Perks, Part 1: JavaScript not Java'
author: Joey
date: 2023-01-01
categories: glidequery
---

One of the main points I made in [Part 0](/2023/01/30/glidequery-perks-part-0.html) was that GlideRecord is a Java object and its various methods return Java types and more Java objects, so it doesn't behave like native JavaScript. In contrast, though it does call GlideRecord to perform database operations, GlideQuery's public interface is written entirely in JavaScript and was intentionally designed to behave like JavaScript.

## JavaScript native types

A big part of how GlideQuery behaves like JavaScript is its consistent use of JavaScript native types. For example, if you ask GlideQuery to return the contents of an Integer or Decimal column in the ServiceNow database, GlideQuery will give you the value as a native JavaScript number. If you ask it to return the contents of a True/False field, you'll get a native JavaScript boolean.


* eliminates pass-by-reference bugs (`array.push(gr.sys_id)` is Bad™)
* `gr.first_name === 'Fred'` won't work because it's not a string, it's a GlideElement object; this led me, and likely many many others, to just always use fuzzy equality instead
* `gr.getValue('active')` is always truthy because it's either string '0' or string '1', both of which are truthy (oddly, this behaves fine if you just use `gr.active`, but what arcane witch magic enables that?)
* GlideAggregate counts are strings, too, but GlideQuery counts are numbers



~~~ javascript
// code goes here
// ...
~~~





{% include endmark.html %}

