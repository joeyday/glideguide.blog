---
layout: post
title: 'GlideQuery Perks, Part 0.5: Resources'
author: Joey
date: 2023-02-01
categories: 
---

Before I start my GlideQuery series proper, I'd be remiss if I didn't take two steps back and recognize all the great content already out there on the topic. I certainly didn't discover all the benefits of GlideQuery myself, though I do have direct experience with most of them. This article will be a complete rundown on every publicly available resource I can find.

Mind you, I'm not presenting any of the following as pre-requisites to my series. I won't assume you've engaged with any of this material beforehand. But likewise, I'll do my best not to just regurgitate what others have said. Though of necessity I'll be covering a lot of the same ground, wherever possible I'll try to give it my own unique spin and share my own original usage examples.

So feel free to engage with as much or as little of what follows, or ignore it for now and explore it later if the rest of my series piques your interest.

## Official documentation

Of course, the best place to start is arguably the official documentation. There are Docs pages for [GlideQuery](https://docs.servicenow.com/bundle/tokyo-application-development/page/app-store/dev_portal/API_reference/GlideQuery/concept/GlideQueryGlobalAPI.html), [Stream](https://docs.servicenow.com/bundle/tokyo-application-development/page/app-store/dev_portal/API_reference/Stream/concept/StreamGlobalAPI.html), and [Optional](https://docs.servicenow.com/bundle/tokyo-application-development/page/app-store/dev_portal/API_reference/Optional/concept/OptionalGlobalAPI.html). There are also mirrors of the same documentation on the Developer site ([GlideQuery](https://developer.servicenow.com/dev.do#!/reference/api/tokyo/server/no-namespace/GlideQueryAPI), [Stream](https://developer.servicenow.com/dev.do#!/reference/api/tokyo/server/no-namespace/StreamGlobalAPI), and [Optional](https://developer.servicenow.com/dev.do#!/reference/api/tokyo/server/no-namespace/OptionalGlobalAPI)).

## Content featuring Peter Bell

Peter Bell is the senior engineer within ServiceNow who created GlideQuery as a side project to help him with his normal day job on the <abbr>IT</abbr> Asset Management team. Naturally, his own content is very well worth checking out.

Bell's original CreatorCon 2020 breakout session, _[GlideQuery: A modern upgrade to GlideRecord (CCB3052)](https://www.servicenow.com/community/creatorcon-blogs/glidequery-a-modern-upgrade-to-gliderecord/ba-p/2331050)_ (and accompanying [slide deck](/files/2023-02-01-ccb3052-bell-glidequery.pdf)), is, as far as I know, the first time GlideQuery was announced officially to the public. It's a great introduction to the <abbr>API</abbr> and all of its benefits over GlideRecord.

From January to April 2021, Bell published a series of excellent blog posts explaining how to use GlideQuery. There's a lot of overlap here with the official documentation, but, where the docs simply list methods alphabetically, these posts order the topics more pedagogically:

<div class="column-left" markdown="1">
- ["GlideQuery Part 1"](https://developer.servicenow.com/blog.do?p=/post/glidequery-p1/)
- ["GlideQuery Part 2"](https://developer.servicenow.com/blog.do?p=/post/glidequery-p2/)
- ["GlideQuery Part 3"](https://developer.servicenow.com/blog.do?p=/post/glidequery-p3/)
- ["GlideQuery Part 4 - Aggregates"](https://developer.servicenow.com/blog.do?p=/post/glidequery-p4/)
</div>
<div class="column-right" markdown="1">
- ["GlideQuery Part 5 - Dotwalking and Flags"](https://developer.servicenow.com/blog.do?p=/post/glidequery-p5/)
- ["GlideQuery - Stream Processing Part 1"](https://developer.servicenow.com/blog.do?p=/post/glidequery-p6/)
- ["GlideQuery - Stream Processing Part 2"](https://developer.servicenow.com/blog.do?p=/post/glidequery-p7/)
- ["GlideQuery - Complex Streams"](https://developer.servicenow.com/blog.do?p=/post/glidequery-p8/)
</div>

In April ’21 Bell did an episode of _Creator Toolbox_ on YouTube with Chuck Tomasi, Brad Tilton, and Andrew Barnes, ["Creator Toolbox - GlideQuery"](https://www.youtube.com/live/IobUxnK3LDo). They work on refactoring some existing GlideRecord code to use GlideQuery instead, and along the way Bell shares a few interesting undocumented features.

In June ’21 Bell did another interview with Chuck Tomasi, this time on the _Breakpoint_ podcast, ["GlideQuery with Peter Bell"](https://developer.servicenow.com/blog.do?p=/post/break-point-025/). This one is largely a review of the material in the original CreatorCon session, but he talks about some of the inspirations for GlideQuery and shares some useful info about the performance of GlideQuery compared to GlideRecord.

## Community resources

GlideQuery shipped in an early form as part of the <abbr>ITAM</abbr> suite in the Orlando release. A few people discovered it, but this post by Jace Benson is the only thing I can find that predates the official announcement: ["What is GlideQuery"](https://jace.pro/post/2020-04-28-what-is-glidequery/). About the same time, Dhruv Gupta (with his colleague Debasis Majhi) published a YouTube video on the _ServiceNow Mavericks_ channel, ["GlideQuery not GlideRecord"](https://www.youtube.com/watch?v=M5AOlMM35pg), showing off several GlideQuery examples.

After the official CreatorCon session came out (GlideQuery shipped officially in the Paris release), Benson followed his first post up with a full rundown of GlideQuery's feature set: ["Glide Freaking Query Wow"](https://jace.pro/post/2020-05-24-glide-freaking-query-wow/). Others have since posted cheat sheets for GlideQuery, such as [Samuel Meylan's "GlideQuery Cheat Sheet"](https://www.snow-adventures.com/blog/glidequery-cheat-sheet/) and [Alberto Colombo’s "GlideQuery Cheat Sheet"](https://blog.kofko.xyz/glidequery-cheat-sheet).

Lubos Strejcek wrote about GlideQuery on the <abbr>STREYDA</abbr> blog in May 2021, ["GlideRecord or GlideQuery? That's the question"](https://www.streyda.eu/post/gliderecordorglidequery), and the anonymous content creator at the _ServiceNow 911_ channel referenced Strejcek's article in his July 2022 video, ["GlideRecord vs. GlideQuery"](https://www.youtube.com/watch?v=yY9YNe8nPfo).

- https://www.johnskender.com/articles/using-glidequery-to-check-if-a-single-record-exists
- https://ismr.dev/posts/glidequery-main
- https://ismr.dev/posts/glidequery-debug-post
- https://ismr.dev/posts/glidequery-parse-post
- https://ahmeddrar.me/2022/11/16/querying-with-glidequery/
- https://dhruvsn.wordpress.com/2020/08/24/glidequery-not-gliderecord/
- https://www.servicenow.com/community/developer-blog/glidequery-vs-gliderecord-a-comparison-part-2/ba-p/2268423

## Conclusion

Well, there you have it. That's every single resource I can find online about GlideQuery. It's honestly not much, and a lot of the community content simply summarizes either the official documentation or the original CreatorCon session. 


~~~ javascript
// code goes here
// ...
~~~





{% include endmark.html %}

