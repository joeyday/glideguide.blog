---
layout: post
title: 
author: 
date: 2023-01-01
categories: 
---



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

Douglas Crockford's example code [^1]:

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

[^1]: _JavaScript: The Good Parts_ (pp. 63-64). O'Reilly Media. Kindle Edition. 
