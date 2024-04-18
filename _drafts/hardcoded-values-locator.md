---
layout: post
title: Hardcoded Values Locator
author: Joey
date: 2024-04-18
categories:
 - script include
 - workflow
 - flow designer
---

<span class="lead">I don't know about your ServiceNow environment</span>, but in our environment we have a lot of technical debt built up in the form of hardcoded values (users, groups, etc.) in legacy Workflows and Flow Designer Flows.

For example, we might have hardcoded a specific user to be an approver for some request process, but of course it's only a matter of time before that user wins the lottery and leaves the company, and then that process is broken. In an effort to mitigate the issue, we may end up using a group instead, but of course if nobody's paying attention it doesn't take long for all the users in the group to rotate out to new positions and then we're right back in the same boat.

Though it probably does deserve a blog post all its own, preventing/solving this problem by establishing better practices up front isn't my topic today. A project I worked on recently has enabled us to be more proactive in mitigating incidents related to the problem.

## The Script Include

Here's a Script Include that can help you locate hardcoded values in legacy Workflows and Flow Designer Flows:

~~~ javascript
// code goes here
// ...
~~~

### How to use it

So far I've been using this by just calling it manually in a background script and having it log out the results, but you could conceivably use this in a scheduled job to either notify someone or raise tasks to clean up newly discovered examples of the problem.

When you instantiate the HardcodedValuesLocator

{% include endmark.html %}

