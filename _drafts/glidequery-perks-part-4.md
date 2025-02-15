---
layout: post
title: 'GlideQuery Perks, Part 4: Many Ways to Process Results'
author: Joey
date: 2023-01-01
categories:
 - glidequery
 - glidequery perks series
---

<span class="lead">One of the problems I never knew GlideRecord has</span> is there's only a couple ways to get records out of it—either `get()` for a single record or `query()` and `next()` for multiple records. GlideQuery breaks the mold by offering a gaggle of expressive new ways to loop through the resulting records.

This article offers a high level overview and a few samples of the kinds of things you can do with GlideQuery, but alongside this article I'm also publishing a new resource I call GlideQuery Design Patterns. More on that at the end of the article.

## GlideQuery, Stream, and Optional


## Working with Streams


## Working with Optionals







## Outline for this article

Stream processing (after select)
 - forEach
 - map, filter, reduce (GQ script include deserves a mention)
 - every, some
 - flatMap, chunk
 - find
 - toArray (mention this only briefly)

Optional processing (after selectOne, get, or getBy)
 - ifPresent
 - isEmpty
 - isPresent
 - orElse
 - get
 - filter, map, flatMap (mention these only briefly)




## Complete overview of GlideQuery, Stream, and Optional methods

Querying
 - where
 - orWhere
 - whereNull
 - orWhereNull
 - whereNotNull
 - orWhereNotNull
 - limit
 - orderBy
 - orderByDesc

Querying concepts
 - dot-walking
 - flags
 - operators

Misc
 - disableWorkflow
 - disableAutoSysFields
 - forceUpdate
 - withAcls
 - parse
 - toGlideRecord

Read
 - select
 - selectOne
 - get
 - getBy

Write
 - insert
 - update
 - updateMultiple
 - insertOrUpdate
 
Delete
 - deleteMultiple
 
Aggregation
 - count
 - sum, avg, min, max
 - aggregate
 - groupBy
 - having

Stream processing (after select)
 - forEach
 - map, filter, reduce (GQ script include deserves a mention)
 - every, some
 - find
 - flatMap, chunk
 - toArray

Optional processing (after selectOne)
 - ifPresent
 - isEmpty
 - isPresent
 - orElse
 - get
 - filter, map, flatMap


(I have my suspicions about why Optional doesn't have an `ifEmpty` method. I think it could lead to anti-patterns and/or its just plan not useful to be down inside a function with no data to work with. Why would you want to be stuck down inside a function for no reason??)

The one thing GlideQuery is just straight-up missing is the ability to interact with journal fields.