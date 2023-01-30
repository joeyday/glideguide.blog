---
layout: post
title: "GlideQuery Perks, Part 0: What's wrong with GlideRecord?"
author: Joey
date: 2023-01-01
categories: 
---

When I first encountered GlideQuery, for a brief moment I naively assumed it was a complete modern replacement for GlideRecord and I got really excited. But I was quickly disappointed when I learned it's just a wrapper for GlideRecord. Since it's built on top of GlideRecord I thought it must have all the same drawbacks and limitations, and for a while I admit I was an unbeliever and a naysayer. But as I learned more about it I realized GlideQuery is one of the best things to ever happen to ServiceNow. I've been using GlideQuery exclusively for the last two years and in this series I hope to convince you to switch, too.

Before I talk about GlideQuery, though, I want to start the series out with a preface of sorts (hence, part zero) to enumerate and explain the downsides and imperfections of GlideRecord. Why have I been hoping for years something new would come along? Isn't GlideRecord fine? If it ain't broke, why fix it?

<img style="width: 90%; max-width: 400px; display: block !important; margin: auto;" src="/assets/images/this-is-fine.png" alt="This is fine" />

## GlideRecord isn't <abbr>SQL</abbr>

Some common complaints I've seen around the interwebtubes are that GlideRecord can't do [stored procedures](https://en.wikipedia.org/wiki/Stored_procedure), [atomic transactions](https://en.wikipedia.org/wiki/Atomicity_(database_systems)), or [set operations](https://en.wikipedia.org/wiki/Set_operations_(SQL)). Amazingly, GlideRecord can't even select specific columns, instead always selecting all the columns in a given table. These are core features of nearly all <abbr>SQL</abbr>-like languages, and, comparatively, GlideRecord just doesn't cut the mustard.

It's always annoyed me that in GlideRecord queries logical <abbr>OR</abbr> takes precedence over logical <abbr>and</abbr>, which is backward from every other programming language and query language I know. And not only this, but there's no way to force <abbr>AND</abbr> to take precedence over <abbr>OR</abbr> using the native methods of GlideRecord! Your only resort is to do an encoded query, and, even then, good luck going more than a couple levels deep with nested <abbr>AND</abbr>'s and <abbr>OR</abbr>'s.

If you know me you know I love to gripe about GlideRecord's poor support for join queries. Five years ago some colleagues and I built what we thought would be a fairly simple Service Portal page that showed parent and child records in a nested view. Users were presented with a paginated and filtered list of parent records, each of which could be clicked to expand a Bootstrap accordion containing a filtered list of child records. Since the parent records were filtered in or out based on the presence or absence of specific child records, the only way to perform the query efficiently was with a database join. This turned out to be a total nightmare.

First off, GlideRecord joins only facilitate adding additional where clauses to filter the returned records; you don't actually get any columns from the joined tables in the result set. So after we got the parent records we still had to turn around and ask for the child records with a separate GlideRecord query. Additionally, there's no support for encoded queries in the join part of the query, so if you're trying to load the join query from a conditions field on a table, no dice. (I stumbled on a few Community posts recently about a `^JOIN` operator in encoded queries, but I'm pretty sure those aren't supported in conditions fields either.) Lastly, I've seen weird behavior depending on the order of the `addCondition` and `addOrCondition` methods used for the join portion of the query. Long story short, support for joins is a big dumpster fire.

## GlideRecord isn't JavaScript






~~~ javascript
// code goes here
// ...
~~~





{% include endmark.html %}

