---
layout: post
title: 
author: 
date: 2023-01-01
categories: 
---




~~~ javascript
// code goes here
// ...
~~~






Let's compare how different types of values get returned by GlideRecord. I have some examples in [GlideQuery Perks, Part 0](/2023/01/30/glidequery-perks-part-0.html) showing how I sussed this out with the `instanceof` operator (see also `JSUtil.instance_of()`).

| Column type | Accessed directly | Accessed with `getValue` |
|:------------|:-----------------|:----------------|
| String | GlideElement / Java string | JavaScript string |
| Integer | GlideElementNumeric / Java string | JavaScript string |
| True/False | GlideElementBoolean / Java boolean | JavaScript string (0 or 1) |

It's honestly pretty odd, right? If we don't use `getValue`, sometimes we get the right type, but actually not really since they're Java types instead of native JavaScript types. If we do use `getValue`, we consistently get a native JavaScript string but then we have to cast it to whichever type we need.

{% include endmark.html %}

