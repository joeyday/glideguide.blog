---
layout: post
title: "GlideQuery Perks, Part 0: What's wrong with GlideRecord?"
author: Joey
date: 2023-01-01
categories: 
---

When I first encountered GlideQuery, for a brief moment I naively assumed it was a complete modern replacement for GlideRecord and I got really excited. But I was quickly disappointed when I learned it's just a wrapper for GlideRecord. Since it's built on top of GlideRecord I thought it must have all the same drawbacks and limitations, and for a while I admit I was an unbeliever and a naysayer. But as I learned more about it I realized GlideQuery is one of the best things that ever happened to ServiceNow. I've been using GlideQuery exclusively for nearly two years and I hope I can convince you to switch if you haven't.

Before I talk about GlideQuery, though, I want to begin the series with a preface of sorts (hence, part zero) to enumerate and explain the downsides and imperfections of GlideRecord. Why have I been hoping for years something new would come along? Isn't GlideRecord fine? If it ain't broke, why fix it?

<img style="width: 90%; max-width: 400px; display: block !important; margin: auto;" src="/assets/images/this-is-fine.png" alt="This is fine" />

## GlideRecord isn't <abbr>SQL</abbr>

Some common complaints I've seen around the interwebtubes are that GlideRecord can't do [stored procedures](https://en.wikipedia.org/wiki/Stored_procedure), [atomic transactions](https://en.wikipedia.org/wiki/Atomicity_(database_systems)), or [set operations](https://en.wikipedia.org/wiki/Set_operations_(SQL)). GlideRecord can't even select for specific columns; instead it always returns all the columns in a given table.

It's always annoyed me that in GlideRecord logical <abbr>OR</abbr> takes precedence over logical <abbr>AND</abbr>, which is backward from every other programming language and query language I know. Incredibly, there's not even a way to force <abbr>AND</abbr> to take precedence over <abbr>OR</abbr> without using encoded queries, and, even then, you can't go more than a couple levels deep with nested <abbr>AND</abbr>'s and <abbr>OR</abbr>'s. It's remarkable this doesn't prevent us from getting things done more often.

If you know me you know I love to gripe about GlideRecord's poor support for joins. First, GlideRecord joins only facilitate adding additional where clauses to filter the returned records; you don't actually get any columns from the joined tables in the result set. Additionally, there's no support for encoded queries in the join clause, so if you're trying to load the join query from a conditions field on a table, no dice. (I stumbled on a few Community posts recently about a `^JOIN` operator in encoded queries, but I'm pretty sure those aren't supported in conditions fields either.) Lastly, I've seen weird behavior depending on the order of the `addCondition` and `addOrCondition` methods used for the join clause. In short, support for joins is a big dumpster fire.

The above are variously considered indispensable features of nearly all <abbr>SQL</abbr>-like query languages. Comparatively, GlideRecord just doesn't cut the mustard.

## GlideRecord isn't JavaScript

A major source of confusion and bugs in ServiceNow development is that GlideRecord just doesn't behave like a JavaScript API. GlideRecord is actually a Java object cleverly disguised as a JavaScript object through the magic of the Mozilla Rhino JavaScript engine. Rhino is the engine that executes all JavaScript scripts on the ServiceNow platform, and it has some pretty neat features that allow sharing of Java objects into the JavaScript environment.

Since GlideRecord is a Java object and not a JavaScript object, it behaves in some unpredictable ways.

~~~ javascript
var gr = new GlideRecord('incident');
gr.get('active', true);

var str = 'Routing to oregon mail server';

gr.short_description;           // Routing to oregon mail server
str;                            // Routing to oregon mail server
gr.short_description == str;    // true
gr.short_description === str;   // false (!)
~~~

Shenanigans like these made me give up on strict equality nearly a decade ago, though I really wish I hadn't. So what's going on here? Why doesn't strict equality work?

Strict equality in JavaScript tests not only for equality of the values, but that the types of the variables are identical. We're pretty sure `gr.short_description` is a string, and fuzzy equality works, so why does the strict comparison fail? Because it's a Java string, not a JavaScript string. Yep, let that one sink in for a minute.

~~~ javascript
typeof str === 'string';                                    // true
str instanceof Packages.java.lang.String;                   // false

typeof gr.short_description === 'string';                   // false
gr.short_description instanceof Packages.java.lang.String;  // true
~~~



## Conclusion

I really tried not to exaggerate anything above, but I know I probably sounded like an infomercial. As you'll see in future articles in this series, GlideQuery fixes several, but not all, of the problems I've described. It also fixes a handful of issues that weren't even on my radar until GlideQuery showed me a better way. Even if none of the above gets you rankled up, I hope you'll stay tuned to learn all the ways GlideQuery might be able to level up your development on the ServiceNow platform.{% include endmark.html %}

