---
layout: post
title: 'GlideQuery Perks, Part 5: Friendly, Helpful Error Messages'
author: Joey
date: 2023-01-01
categories: glidequery
---


Make sure to use `current.getValue('date') || null` when updating date fields. GlideRecord's `getValue()` method passes you an empty string if the date field is empty, but GlideQuery expects to be given null in that case.

Make sure to use `gr.integer.nil() ? null : Number(gr.integer)` when updating number fields if you want to preserve null values. GlideRecord's `getValue()` method passes you null if the number field is empty, but casting that to a number with `Number()` will turn it into a zero.

(Are there any other wonky field types?)





~~~ javascript
// code goes here
// ...
~~~





{% include endmark.html %}

