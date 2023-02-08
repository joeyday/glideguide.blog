---
layout: post
title: 
author: 
date: 2023-01-01
categories: 
---

Have you ever wondered what happens if you edit or remove the starting boilerplate code in a Script Include? Surely that code's needed for the Script Include to work properly, right? As it turns out, not exactly. There are a number of alternative design patterns that can be used in Script Includes. In this article I'll tell you about one, the Module pattern, which I really like and think you should consider adopting.

I remember my mind was completely blown when I read Travis Toulson's _CodeCreative_ article, ["Interface Design Patterns for Script Includes"](https://codecreative.io/blog/interface-design-patterns-for-script-includes/). It completely demystified for me what's really happening with Script Includes. A Script Include is exactly what it says on the tin: a way to include some script from one place in other places, and the content and design pattern of that script can be pretty much anything you want. ServiceNow provides a suggested format, but you're under no obligation to follow it. As long as you start your script by declaring a variable with the same name as your Script Include record, you can call it from anywhere just like any other Script Include.[^1]

What I can't understand is why it took me until just a couple years ago to actually try using some of these alternative patterns, but, let me tell you, since I did I've been hooked.

## What's so bad about the default class pattern?

Nothing is particularly wrong with the default pattern for Script Includes, but there is one thing that's always bugged me. Ever seen a Script Include like this?

~~~ javascript
var MyFirstScriptInclude = Class.create();
MyFirstScriptInclude.prototype = {
    initialize: function () {
    },
    
    publicGreeting: function () {
    	return _privateGreeting();
    },
    
    _privateGreeting: function () {
    	return 'Hello, world!';
    },

    type: 'MyFirstScriptInclude'
};
~~~

If you've seen this before, you probably know the underscore at the front of the `_privateGreeting` method makes it a private method, inaccessible from outside the Script Include. Except, that's totally wrong. A lot of languages do have ways of declaring truly private functions, but JavaScript doesn't. The underscore doesn't do anything to make a method private or secret, it's just a naming convention. The method remains perfectly accessible from outside.

I appreciate Douglas Crockford's scathing critique in his _[Code Conventions for the JavaScript Programming Language](https://www.crockford.com/code.html)_ (emphasis mine): "Do not use underbar as the first or last character of a name. It is sometimes intended to indicate privacy, but it does not actually provide privacy. If privacy is important, use closure. _Avoid conventions that demonstrate a lack of competence_." Ouch!

Sick burn aside, he mentioned something called "closure". What's that and how can we use it? Well, a closure is...

~~~ javascript
String.prototype.deentityify = (function () {
    // The entity table. It maps entity names to
    // characters.
    var entity = {
        quot: '"',
        lt:   '<',
        gt:   '>'
    };

    // Return the deentityify method.
    return function (  ) {
        // This is the deentityify method. It calls the string
        // replace method, looking for substrings that start
        // with '&' and end with ';'. If the characters in
        // between are in the entity table, then replace the
        // entity with the character from the table. It uses
        // a regular expression (Chapter 7).
        return this.replace(/&([^&;]+);/g,
            function (a, b) {
                var r = entity[b];
                return typeof r === 'string' ? r : a;
            }
        );
    };
}());
~~~

Douglas Crockford's example code [^2]:

~~~ javascript
var serial_maker = function (  ) {
    // Produce an object that produces unique strings. A
    // unique string is made up of two parts: a prefix
    // and a sequence number. The object comes with
    // methods for setting the prefix and sequence
    // number, and a gensym method that produces unique
    // strings.
    var prefix = '';
    var seq = 0;

    return {
        set_prefix: function (p) {
            prefix = String(p);
        },
        set_seq: function (s) {
            seq = s;
        },
        gensym: function ( ) {
            var result = prefix + seq;
            seq += 1;
            return result;
        }
    };
};

var seqer = serial_maker( );
seqer.set_prefix('Q');
seqer.set_seq(1000);
var unique = seqer.gensym( ); // unique is "Q1000"
~~~

Travis Toulson's example code:

~~~ javascript
var ModulePattern = (function() {
    var privateVariable;

    function myFunction() {
        // Put function code here
    }

    function privateFunction() {
        // Put private function code here
    }

    return {
        'myFunction': myFunction
    }
})();
~~~

ChatGPT example code:

~~~ javascript
// Define the module
const Module = (function () {
    // Private variables and functions
    let privateCounter = 0;
    function privateFunction() {
        privateCounter++;
    }

    // Public variables and functions
    return {
        publicMethod: function() {
            privateFunction();
        },
        publicCounter: function() {
            return privateCounter;
        }
    };
})();

// Use the module
Module.publicMethod();
console.log(Module.publicCounter()); // 1
~~~












{% include endmark.html %}



I must've been aware of the article shortly after it was first published in 2017, and I'd also read _[JavaScript: The Good Parts](https://www.oreilly.com/library/view/javascript-the-good/9780596517748/)_ by Douglas Crockford, who is staunchly against the `new` operator (pp. 46–47) and advocates instead for the Module pattern (pp. 61–64, 77–82). But it wasn't until a couple years ago that I tried writing my first Script Include using the Module pattern, and since then I've been completely hooked.


[^1]: And you might think this is the only strictly necessary part of a Script Include, but not even this is mandatory. If there isn't a variable in the Script Include with the same name, you can still include it manually using [gs.include()](https://developer.servicenow.com/dev.do#!/reference/api/tokyo/server/no-namespace/c_GlideSystemScopedAPI#r_ScopedGlideSystemInclude_String).