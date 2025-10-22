---
layout: post
title: "Type Coercion in ServiceNow JavaScript"
author: Dan
date: 2025-10-22
categories: 
---

One of JavaScript's most helpful (and sometimes confusing) features is how flexible it is with data types. The language will often convert values from one type to another automatically to make your code work. This is called [type coercion](https://developer.mozilla.org/en-US/docs/Glossary/Type_coercion), and while it can make scripts shorter and sometimes easier to read, it can also obscure what's actually happening, especially in ServiceNow.

Let me show you what I mean. Have you ever seen code like this?

~~~ javascript
if (!current.name) {
    // do something
}
~~~

Simple, right? But there's more happening here than meets the eye.

## What's Really Happening

This code checks if a GlideRecord's name field is empty. But here's the thing: `current.name` isn't a string, it's a [GlideElement](https://developer.servicenow.com/dev.do#!/reference/api/latest/server/no-namespace/c_GlideElementScopedAPI) object. So how does this condition work?

It works because ServiceNow's GlideElement overrides its `toString()` method to return the field's value. When JavaScript evaluates this condition in a boolean context, it converts the GlideElement to a string (getting `""`), then converts that empty string to a boolean. Since empty strings are falsy, the condition evaluates to true.

This is type coercion in action—JavaScript automatically converting one type to another behind the scenes.

## Why This Matters

This automatic behavior can make code look clean and concise, but it hides what's really going on. That's especially true in ServiceNow, where objects like GlideRecord and GlideElement have custom conversion behavior that doesn't exist in standard JavaScript.

## A Better Approach

To make your intent clear and consistent, be explicit about what you're checking:

~~~ javascript
if (current.name.toString() === '') {
    // explicitly converts the GlideElement to a string value
    // check if the value is equal to an empty string
}
~~~

For this specific type of check ServiceNow has also provided a built-in method that is both readable and safe:

~~~ javascript
if (current.name.nil()) {
    // safe and readable - uses GlideElement's built-in check for an empty value
}
~~~

Being explicit avoids surprises and makes your code easier to reason about for anyone reading it later, including future you.

## For Developers New to Plain JavaScript

If you've mostly worked within ServiceNow, pay special attention to how objects like GlideElement behave. Understanding the difference between ServiceNow's custom objects and standard JavaScript types helps prevent subtle bugs and builds habits that translate well to JavaScript development outside the platform.
