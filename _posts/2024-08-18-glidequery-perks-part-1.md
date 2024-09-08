---
layout: post
title: 'GlideQuery Perks, Part 1: Unambiguous Boolean Logic'
author: Joey
date: 2024-08-18
categories:
 - glidequery
 - glidequery perks series
excerpt_separator: <!--more-->
---

<span class="lead">I published [Part 0](/2023/01/30/glidequery-perks-part-0.html) and [Part 0.5](/2023/02/08/glidequery-perks-part-0.5.html) of this series so long ago</span> you probably thought I forgot about it, but this is a favorite topic of mine so I couldn't leave it unfinished. 😅 If you haven't read the first part, I highly recommend catching up before reading this.

One of the problems I mentioned in Part 0 is that GlideRecord is squirrelly when it comes to precedence of logical <abbr>AND</abbr> and <abbr>OR</abbr>. GlideQuery avoids this issue by disallowing ambiguous boolean logic.

<!--more-->

## Mind your <abbr>AND</abbr>s and <abbr>OR</abbr>s

To avoid uncertainty in queries, GlideQuery's interface requires you to nest <abbr>AND</abbr>'s and <abbr>OR</abbr>'s that could have more than one possible order of operation.

For example, "A <abbr>AND</abbr> B <abbr>AND</abbr> C" raises no cause for alarm, as does "A <abbr>OR</abbr> B <abbr>OR</abbr> C", but "A <abbr>OR</abbr> B <abbr>AND</abbr> C" is problematic. How so? You could simply evaluate the logic in order, taking the <abbr>OR</abbr> first and then the <abbr>AND</abbr>. However, in most programming languages and query languages, including JavaScript, <abbr>AND</abbr> takes precedence over <abbr>OR</abbr> (without any need for parentheses; cf. how multiplication takes natural precedence over addition in arithmetic), but, curiously, in ServiceNow queries—both encoded queries and GlideRecord queries—<abbr>OR</abbr> has always taken precedence over <abbr>AND</abbr>. In fact, with normal calls to `addQuery` and `addOrCondition`, it's not even possible to force <abbr>AND</abbr> to take precedence over <abbr>OR</abbr> (you have to switch to using an encoded query with the `^NQ` operator).

Given the inconsistency, you can see there's a lot of potential for confusion here. Rather than simply evaluating a query from left to right, and rather than letting either logical operator take precedence over the other, GlideQuery takes the approach of simply always requiring parentheses (in a manner of speaking) so that there is never any ambiguity about order of operations in a query.

The following is fine in GlideQuery:

~~~ javascript
new GlideQuery('incident')
  .where('active', true)
  .where('short_description', 'CONTAINS', 'blue smoke')
  .where('assigned_to.name', 'ITIL User');
~~~

And this is fine, too:

~~~ javascript
new GlideQuery('incident')
  .where('active', true)
  .orWhere('short_description', 'CONTAINS', 'blue smoke')
  .orWhere('assigned_to.name', 'ITIL User');
~~~

But the following will throw what GlideQuery calls a "nice error":

~~~ javascript
new GlideQuery('incident')
  .where('active', true)
  .orWhere('short_description', 'CONTAINS', 'blue smoke')
  .where('assigned_to.name', 'ITIL User');
~~~

## Nice errors

You might think this is annoying, but GlideQuery throws a lot more errors than GlideRecord very much on purpose. It's trying to help you discover bugs or potential problems immediately after writing your code, rather than silently failing in such a way that you may not notice problems until weeks down the road (and probably in the production environment by then). We'll talk a lot more about nice errors in Part 5 of this series.

In the case of our third example above, the error is "Ambiguous query: cannot contain multiple where() expressions with an orWhere() expression", and, as with all nice errors, execution of your script will simply stop. GlideQuery refuses to go any further in order to avoid the possibility of any data corruption or worse. I'm almost never jazzed when I see a nice error, but I'm really glad GlideQuery throws them because that little bit of pain up front helps me identify the problem immediately and avoid what would probably have been a lot more pain later.

## How to write "parentheses" in GlideQuery

The way GlideQuery disambiguates whether <abbr>AND</abbr> should come before <abbr>OR</abbr> and vice versa is by having you pass a new table-agnostic GlideQuery object as the only parameter to a `where` or `orWhere` clause, like these examples:

~~~ javascript
// (active OR blue smoke) AND itil user
new GlideQuery('incident')
  .where(new GlideQuery()
    .where('active', true)
    .orWhere('short_description', 'CONTAINS', 'blue smoke'))
  .where('assigned_to.name', 'ITIL User');
  
// active OR (blue smoke AND itil user)
new GlideQuery('incident')
  .where('active', true)
  .orWhere(new GlideQuery()
    .where('short_description', 'CONTAINS', 'blue smoke')
    .where('assigned_to.name', 'ITIL User'));
~~~

Doing it this way feels somewhat cumbersome (count your parentheses carefully!), but if a little awkwardness is the price to pay for clearer code, I'm here for it. Usually, writing them the way I have above is readable enough, but you could make this look less busy and complicated by assigning the sub-queries to variables beforehand.


## Conclusion

ServiceNow's inexplicable "<abbr>OR</abbr>" bias has always bothered me, but the way these GlideQuery clauses can be nested addresses my gripe and makes the boolean logic more transparent and precise. And recall that with GlideRecord you _can't_ make <abbr>AND</abbr> take precedence over <abbr>OR</abbr> without resorting to encoded queries, so being able to do so with normal calls to `where` and `orWhere` with GlideQuery is a massive improvement.

That's it for this installment. I hope this starts to persuade you GlideQuery is superior to GlideRecord and should be the way forward for all pro-code solutions on the ServiceNow platform, and I hope you're looking forward to future articles in the series. Cheers!{% include endmark.html %}

Next in the series: [Part 2: JavaScript Not Java &rarr;](/2024/09/07/glidequery-perks-part-2.html)


