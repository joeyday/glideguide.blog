---
layout: post
title: Retrieving records with all fields using GlideQuery
author: Joey
date: 2023-01-15
categories: glidequery
---

I love how the GlideQuery `select` method takes a list of fields and returns a Stream of record objects with only those fields. Since it uses GlideRecord under the hood, I'm not sure we get much performance gain from it, but it just makes working with the resulting objects so much more pleasant, having exactly and only the fields you need in any given context. But what can you do if you really need every field for a given record?

It'd be great if GlideQuery had some equivalent of `SELECT *`, but it doesn't, so asked the creator of GlideQuery, Peter Bell, and he shared with me a way to get all the fields for a given table.

