---
layout: post
title: BetterGetter
author: Joey
date: 2024-03-05
categories:
  - gliderecord
  - script include
---

In keeping with my series about GlideQuery, I'd like to share a tool Dan and I made that can slightly improve the experience when you'd prefer to use GlideQuery but have no choice but to use GlideRecord.

For example, imagine you're working in a before business rule and you need to interact with the current record. It's possible earlier business rules have already changed some values, so using GlideQuery to retrieve the current record from the database will give you, at best, stale data or, at worst, no data at all (if the record is new and won't even be inserted until after the before business rules finish).

The annoying thing is what I've already been discussing over in my GlideQuery series: when you refer to `gr.short_description` you're not really referring to a native JavaScript string. It's a weird both/and of a Java GlideElement object and a Java string, and it behaves very strangely as a consequence. When you try to use strict equality for comparisons it won't work as expected, and if you try to store it for later retrieval (say in an array) you might find it changes out from under you the next time you call `gr.next()`.

We all have our favorite tricks like using `gr.getValue()` to turn everything into a string, but that's less than ideal, especially for numeric and boolean fields.

## A better way

Dan and I workshopped a tool and put it up on Share last year. We call it [BetterGetter](https://developer.servicenow.com/connect.do#!/share/contents/1148200_bettergetter). It's a wrapper for GlideRecord with its own `getValue()` method that returns values in native JavaScript types rather than weird GlideElement objects or Java types.

Use it like this:

~~~ javascript
var bg = new BetterGetter(current);
bg.getValue('short_description');  // native JavaScript string
bg.getValue('priority');  // native JavaScript number
bg.getValue('active');  // native JavaScript boolean
~~~

If this looks like something you'd find useful, grab the code from Share, or copy/paste it from below. Hope you find it useful!{% include endmark.html %}

<img style="width: 90%; display: block !important; margin: auto;" src="/assets/images/2024-03-05-bettergetter.png" alt="BetterGetter Script Include settings" />

~~~ javascript
// Directly accessing GlideRecord elements gives you Java native types,
// and GlideRecord's getValue() method always returns JavaScript strings.
// Neither is very helpful! Using GlideQuery is the recommended best
// practice, but sometimes you are forced to work with GlideRecord
// objects, e.g. the `current` object in Business Rules.
// 
// BetterGetter's getValue method returns the correct JavaScript native
// type based on the data type of an element's dictionary entry. BetterGetter
// also provides convenient methods for converting an element to any other
// preferred JavaScript native type, if possible.
// 
// Use it like this:
//    var active = new BetterGetter(current).getValue('active');
// 
// Or like this:
//    var getter = new BetterGetter(current);
//    var active = getter.getValue('active');

function BetterGetter(gr) {
	// In case caller omits the new keyword
    if (!(this instanceof BetterGetter)) return new BetterGetter(gr);
	this._gr = gr;
}

BetterGetter.prototype.getValue = function(element) {
    switch (this._gr[element].getED().getInternalType() + '') {
        case 'boolean':
            return this.getBoolean(element);
        case 'glide_date':
            return this.getDate(element);
        case 'glide_date_time':
            return this.getDateTime(element);
        case 'glide_list':
            return this.getList(element);
        case 'decimal':
        case 'float':
        case 'integer':
        case 'longint':
        case 'long':
        case 'numeric':
            return this.getNumber(element);
        default:
            return this.getString(element);
    }
};

BetterGetter.prototype.getBoolean = function(element) {
    return Boolean(this._gr[element]);
};

BetterGetter.prototype.getDate = function(element) {
    return (this._gr[element] + '').slice(0, 10) || null;
};

BetterGetter.prototype.getDateTime = function(element) {
    return this._gr[element] + '' || null;
};

BetterGetter.prototype.getList = function(element) {
    if (this._gr[element].nil()) return null;
    return (this._gr[element] + '').split(',');
};

BetterGetter.prototype.getListDisplay = function(element) {
    if (this._gr[element].nil()) return null;
    return (this._gr.getDisplayValue(element)).split(',');
};

BetterGetter.prototype.getNumber = function(element) {
    if (this._gr[element].nil()) return null;
    return Number(this._gr[element]);
};

BetterGetter.prototype.getString = function(element) {
    return this._gr[element] + '' || null;
};
~~~