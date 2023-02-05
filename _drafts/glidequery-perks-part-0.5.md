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

Peter Bell is the senior engineer within ServiceNow who created GlideQuery as a side project to help him with his normal day job on the IT Asset Management team. Naturally, his own content is very well worth checking out.

Bell's original CreatorCon 2020 breakout session, _[GlideQuery: A modern upgrade to GlideRecord (CCB3052)](https://www.servicenow.com/community/creatorcon-blogs/glidequery-a-modern-upgrade-to-gliderecord/ba-p/2331050)_ (and accompanying [slide deck](/files/2023-02-01-ccb3052-bell-glidequery.pdf)), is, as far as I know, the first time GlideQuery was announced officially to the public. It's a great introduction to the API and all of its benefits over GlideRecord.

From January to April 2021, Bell published a series of excellent blog posts explaining how to use GlideQuery. There's a lot of overlap here with the official documentation, but, where the docs simply list methods alphabetically, these posts order the topics more pedagogically:
- [GlideQuery Part 1](https://developer.servicenow.com/blog.do?p=/post/glidequery-p1/)
- [GlideQuery Part 2](https://developer.servicenow.com/blog.do?p=/post/glidequery-p2/)
- [GlideQuery Part 3](https://developer.servicenow.com/blog.do?p=/post/glidequery-p3/)
- [GlideQuery Part 4 - Aggregates](https://developer.servicenow.com/blog.do?p=/post/glidequery-p4/)
- [GlideQuery Part 5 - Dotwalking and Flags](https://developer.servicenow.com/blog.do?p=/post/glidequery-p5/)
- [GlideQuery - Stream Processing Part 1](https://developer.servicenow.com/blog.do?p=/post/glidequery-p6/)
- [GlideQuery - Stream Processing Part 2](https://developer.servicenow.com/blog.do?p=/post/glidequery-p7/)
- [GlideQuery - Complex Streams](https://developer.servicenow.com/blog.do?p=/post/glidequery-p8/)

In April ’21 Bell did an episode of _Creator Toolbox_ on YouTube with Chuck Tomasi, Brad Tilton, and Andrew Barnes, “[Creator Toolbox - GlideQuery](https://www.youtube.com/live/IobUxnK3LDo)”. They work on refactoring some existing GlideRecord code to use GlideQuery instead, and along the way Bell shares a few interesting undocumented features.

In June ’21 Bell did another interview with Chuck Tomasi, this time on the _Breakpoint_ podcast, “[GlideQuery with Peter Bell](https://developer.servicenow.com/blog.do?p=/post/break-point-025/)”. This one is largely a review of the material in the original CreatorCon session, but he talks about some of the inspirations for GlideQuery and shares some useful info about the performance of GlideQuery compared to GlideRecord.

## Community resources

- [Jace Benson › What is GlideQuery?](https://jace.pro/post/2020-04-28-what-is-glidequery/)
- [Jace Benson › Glide Freaking Query Wow](https://jace.pro/post/2020-05-24-glide-freaking-query-wow/)
- [Alberto Colombo › GlideQuery Cheat Sheet](https://blog.kofko.xyz/glidequery-cheat-sheet)
- [Sam's ServiceNow Adventures › Overview of GlideQuery](https://www.snow-adventures.com/blog/overview-of-glidequery/)
- [Sam's ServiceNow Adventures › GlideQuery Cheat Sheet](https://www.snow-adventures.com/blog/glidequery-cheat-sheet/)

- [ServiceNow 911 › GlideRecord vs. GlideQuery](https://www.youtube.com/watch?v=yY9YNe8nPfo)
- [Dhruv Gupta › GlideQuery not GlideRecord](https://www.youtube.com/watch?v=M5AOlMM35pg)

Other stuff from the community?




~~~ javascript
// code goes here
// ...
~~~





{% include endmark.html %}

