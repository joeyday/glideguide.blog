---
layout: post
title: 'GlideQuery: Selecting All Fields'
author: Joey
date: 2023-01-15
categories: glidequery
---

I love how GlideQuery's select method takes a list of column names and returns a Stream of records with only those columns. Since it uses GlideRecord under the hood, I'm not sure we get much performance gain from it, but it makes working with the resulting objects so much more pleasant, having exactly and only the columns you need in a given context. But what can you do if you really need every column for a given record?

It'd be great if GlideQuery had some equivalent of `SELECT *`, but it doesn't, so I asked the creator of GlideQuery, Peter Bell, and he shared with me a simple way to get all the columns for a given table using the Schema <abbr>API</abbr>.

{% highlight javascript %}
var schema = Schema.of('incident', ['*']);
var columns = Object.keys(schema.incident);
{% endhighlight %}

And now you've got the complete list of column names ready to be passed directly into the select method.

{% highlight javascript %}
var GlideQuery('incident')
  .select(columns);
{% endhighlight %}

A few notes though:

- If you're working in a scoped application, on the first line you'll need to use `global.Schema`.
- If you don't know the table name but are using a variable instead, you'll need to access the property on the second line with bracket notation, like `schema[table]`.

That's all there is to it. I don't recommend doing this all the time, but next time you need it I hope it comes in handy.{% include endmark.html %}

