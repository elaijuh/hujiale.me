+++
categories = ["tech"]
date = "2014-12-04T01:14:12+08:00"
draft = false
slug = ""
tags = ["javascript"]
title = "Function overload in JavaScript"

+++

As we know, Javscript has no function overload in nature. If you try to declare the same function with different signature, the previous one will be overwritten rather than overloaded. As function overload in most OO programming languages are by means of parameters, it operates on different logic according to the passed in parameters. So there is still a way to do function overload in JS.

<span class="more"></span>

``` javascript
var foo = {
    bar: function () {
        switch (arguments.length) {
            case 0:
                console.log(0);
                break;
            case 1:
                console.log(arguments[0]);
                break;
            case 2:
                console.log(arguments[0] + arguments[1]);
                break;
        }
    }
}

foo.bar(); // 0
foo.bar(1); // 1
foo.bar(1, 2); // 3
```
This approach is workable but not tidy and flexiable. With more and more overloads, the switch block becomes longer and longer scaring away developers.

With `function.length` and `closure`, we could implement the same in a neat and ninja way.
``` javascript
var foo = {
    overloadMethod: function (name, fn) {
        var baseFn = this[name];
        this[name] = function () {
            if (fn.length === arguments.length) {
                return fn.apply(this, arguments);
            } else if (typeof baseFn === 'function') {
                return baseFn.apply(this, arguments); // baseFn is in a closure
            }
        };
    }
};

foo.overloadMethod('bar', function() {
    console.log(0);
})
foo.overloadMethod('bar', function(x) {
    console.log(x);
})
foo.overloadMethod('bar', function(x, y) {
    console.log(x + y);
})


foo.bar(); // 0
foo.bar(1); // 1
foo.bar(1, 2); // 3
```
So everytime you want to overload the method, just call `foo.overloadMethod` with the method name and anonymous function passed in. It looks like `foo.bar` is overwritten again and again, but actually every single version of `foo.bar` is completely kept in closure to be referenced. 

---
Reference book: [Secrets of the JavaScript Ninja](http://www.amazon.com/Secrets-JavaScript-Ninja-John-Resig/dp/193398869X/ref=sr_1_1?ie=UTF8&qid=1417626477&sr=8-1&keywords=secrets+of+javascript+ninja), [John Resig](http://ejohn.org/)

