---
layout: post
title: 'GlideQuery Perks, Part 1: Unambiguous Boolean Logic'
author: Joey
date: 2023-01-01
categories: glidequery
---

One of the problems I mentioned in [Part 0](/2023/01/30/glidequery-perks-part-0.html) of this series is that GlideRecord is squirrelly when it comes to precedence of logical <abbr>AND</abbr> and <abbr>OR</abbr>. GlideQuery avoids this issue by disallowing ambiguous boolean logic.

## Mind your <abbr>AND</abbr>s and <abbr>OR</abbr>s

To avoid uncertainty in queries, GlideQuery's interface requires you to nest <abbr>AND</abbr>'s and <abbr>OR</abbr>'s that could have more than one possible order of operation.

For example, "A <abbr>AND</abbr> B <abbr>AND</abbr> C" raises no cause for alarm, as does "A <abbr>OR</abbr> B <abbr>OR</abbr> C", but "A <abbr>OR</abbr> B <abbr>AND</abbr> C" is problematic. How so? You could simply evaluate the logic in order, taking the <abbr>OR</abbr> first and then the <abbr>AND</abbr>. However, in most programming languages and query languages, including JavaScript, <abbr>AND</abbr> takes precedence over <abbr>OR</abbr> (without any need for parentheses; cf. how multiplication takes natural precedence over addition in arithmetic), but, curiously, in ServiceNow queries, both encoded queries and GlideRecord queries, <abbr>OR</abbr> has always taken precedence over <abbr>AND</abbr>.

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




{% include endmark.html %}

