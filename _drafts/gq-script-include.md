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

In a nutshell, <abbr>GQ</abbr> is a small library of helper functions designed for use with GlideQuery, particularly with the filter, map, and reduce methods of the Stream object. In the video, Bell demonstrates the use of `GQ.jsonDebug()` with the map method to send whole records to the system log with pretty formatting, a bit like this:

~~~ javascript
new global.GlideQuery('incident')
  .where('active', true)
  .limit(5)
  .select('number', 'short_description', 'caller_id$DISPLAY')
  .map(global.GQ.jsonDebug)
  .forEach((incident) => {
    // do something
    // ...
  })
~~~





{% include endmark.html %}

[^1]: See the episode here: [Creator Toolbox - GlideQuery](https://www.youtube.com/watch?v=IobUxnK3LDo) (my direct quote is from around 52 minutes into the video).