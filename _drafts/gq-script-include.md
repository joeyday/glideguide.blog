---
layout: post
title: "The GQ Script Include"
subtitle: "The GlideQuery helper functions you never knew you needed"
author: Joey
date: 2024-03-05
categories:
  - glidequery
  - gq
  - gqx
  - script include
excerpt_separator: <!--more-->
---

<abbr>I wonder if you know</abbr> the <abbr>GQ</abbr> script include exists. Maybe you've heard of GlideQuery, but <abbr>GQ</abbr> is a bit of an undocumented feature. <!--more--> I first heard about it from Peter Bell himself (creator of GlideQuery) in an episode of *Creator Toolbox*, where he calls it, "a little undocumented magic, if you're into that kind of stuff."[^1] Why, yes, yes I am.

## What is it?

In a nutshell, <abbr>GQ</abbr> is a library of helper functions designed for use with GlideQuery, particularly with the filter, map, and reduce methods of the Stream object. In the video, Bell demonstrates the use of `GQ.jsonDebug()` with the map method to send whole records to the system log with pretty formatting, a bit like this:

~~~ javascript
new global.GlideQuery('incident')
  .where('active', true)
  .limit(5)
  .select('number', 'short_description', 'caller_id$DISPLAY')
  .map(global.GQ.jsonDebug)
  .forEach((incident) => {
    // do something
    // ...
  });
~~~

Don't miss this: the `jsonDebug()` method isn’t being invoked. Instead it’s being passed to the `map()` method as the callback function parameter. If you're not familiar, JavaScript is a first-class functions language. Functions are also variables and can be passed in and out of other functions as parameters and return values.

## Partially-applicable functions

The <abbr>GQ</abbr> script include has a really awesome feature called partially-applicable functions. The `labelDebug()` method is a great example. It takes two parameters, a string label and some value (usually a record object). If you call it with both, it system logs a prettified JSON string of the value parameter with the label parameter prepended. The system log can be pretty chatty, so a label can make it easier to find the debug statements you're looking for.

But if you know how `map()` works, you know it only passes one argument to the callback function you pass in, so how do you get a version of `labelDebug()` that only takes one argument? This is the beauty of partially-applicable functions. If you call `labelDebug()` with just the first argument, the label, it doesn't immediately send output to the debug log. Instead, it returns a new purpose-built function. The returned function, which has the label permanently baked in, only requires one parameter, the value.

So, unlike `jsonDebug()`, you do invoke this one when you use it with `map()`, but you only pass the first of the two arguments causing a new function to get returned and passed into `map()`. Each time through its own loop, the `map()` method will then supply the second argument to the new function.

~~~ javascript
new global.GlideQuery('incident')
  .where('active', true)
  .limit(5)
  .select('number', 'short_description', 'caller_id$DISPLAY')
  .map(global.GQ.labelDebug('Active Incident: '))
  .forEach((incident) => { // ... });
~~~

My favorite and probably the most useful method in the whole <abbr>GQ</abbr> utility is `get()`. It's partially-applicable like `labelDebug()`, taking a field name as it's first parameter and some value (again, usually a record object) as its second parameter. It takes the field name and returns the value of that property from the record object. Use it like this:

~~~ javascript
let names = new GlideQuery('sys_user')
  .whereNotNull('first_name')
  .select('first_name')
  .map(GQ.get('first_name'))
  .toArray(20);
~~~

If all you need is a list of sys_ids or a list of names, `get()` is a very handy way to convert a Stream of record objects into the simpler list you need.

## GQX

The <abbr>GQ</abbr> utility has other functions like `merge()` and `values()` that I highly recommend you check out, but I also want to share with you another utility I made myself that complements <abbr>GQ</abbr> with a few helpful functions if its own. I call it <abbr>GQX</abbr>.

~~~ javascript

~~~



https://www.youtube.com/watch?v=nuML9SmdbJ4





{% include endmark.html %}

[^1]: See the episode here: [Creator Toolbox - GlideQuery](https://www.youtube.com/watch?v=IobUxnK3LDo) (my direct quote is from around 52 minutes into the video).