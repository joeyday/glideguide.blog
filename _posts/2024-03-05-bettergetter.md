---
layout: post
title: BetterGetter
author: Joey
date: 2024-03-05
categories:
  - bettergetter
  - gliderecord
  - script include
---

In keeping with my series about GlideQuery, I'd like to share a tool Dan and I made that can slightly improve the experience when you'd prefer to use GlideQuery but have no choice but to use GlideRecord.

## The problem

Imagine you're working in a before business rule and you need to interact with the current record. It's possible earlier business rules have already changed some values, so using GlideQuery to retrieve the current record from the database will give you, at best, stale data or, at worst, no data at all (i.e. if the record is new and won't even be inserted until after the before business rules finish). In this and other scenarios, your only option is to interact with the GlideRecord object you've been given.

The annoying thing about GlideRecord is what I've already been discussing over in my GlideQuery series: the property `gr.short_description` isn't a native JavaScript string. It's a weird Frankenstein of a Java GlideElement object and a Java string, and it behaves counterintuitively. Strict equality for comparisons fails in mysterious ways, and if you store the value for later use (say in an array) you'll find it changes out from under you the next time you call `gr.next()`.

We all have our favorite tricks like using `gr.getValue()` to cast everything to strings, but that's less than ideal for numeric and boolean fields.

## There is a better way

Dan and I workshopped a tool and put it up on Share last year. We call it [BetterGetter](https://developer.servicenow.com/connect.do#!/share/contents/1148200_bettergetter). It's a wrapper for GlideRecord with its own magic `getValue()` method that returns native JavaScript types rather than weird GlideElement objects or Java types.

Use it like this:

~~~ javascript
var bg = new BetterGetter(current);
bg.getValue('short_description');  // string
bg.getValue('priority');           // number
bg.getValue('active');             // boolean
bg.getValue('watch_list');         // array of strings
~~~

Note that BetterGetter is only a getter, not a setter, so if you need to set values you would just use `current.setValue()` as usual. The neat thing is your BetterGetter object becomes sort of quantum entangled with the GlideRecord object passed to it, so you can get some values with BetterGetter, set a value or two the normal way, then get them later with BetterGetter and you'll get the updated values.

If you do happen to be iterating over a GlideRecord, you can make a single BetterGetter object outside your while loop, then refer to it inside the loop. Since it maintains a reference to the original GlideRecord object you'll get appropriate values for each new record after calling `gr.next()`, but they'll be native JavaScript values so you can safely store them for later use.

If this sounds awesome and you want to try it out, grab the Update Set from Share or copy/paste the Script Include below. Let us know if you find a cool use for this. Cheers!{% include endmark.html %}

<p style="text-align: center;">∗ ∗ ∗</p>

#### Settings:

<img style="width: 100%; display: block !important; margin: auto;" src="/assets/images/2024-03-05-bettergetter.png" alt="BetterGetter Script Include settings" />

#### Script:

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