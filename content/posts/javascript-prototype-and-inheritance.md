+++
categories = ["tech"]
date = "2014-12-01T05:13:52+08:00"
draft = false
slug = ""
tags = ["javascript"]
title = "JavaScript prototype and inheritance"

+++

Inheritance in JS is quite different from class based inheritance which is popular for being used in C++ and Java. In class based inheritance language, we have to abstract a class which is not a real world subject. While in prototype based inheritance, we can implement inheritance on objects directly.

<span class="more"></span>

The post walks you through how to do inheritance in JS.

### Simple object inheritance
As every object has an internal property `[[prototype]]` which points to it's prototype, we can directly assign a super object to it. [[prototype]] could be implemented in different ways in different browsers. In Chrome and Firefox, it can be refered as `__proto__`.

``` javascript
var sup = {x: 'x', y: 'y'};
var sub = {a: 'a'};
console.log(sub.x); // undefined

sub.__proto__ = sup;
console.log(sub.x); // x
```

### Inheritance by prototype chaining
In most of the time, especially in OO style, we would like to have a constructor and then create objects from it. In JS, constructor is not a class and not different from any regular function except that you can `new` it. Usually a constructor does not explicitly return anything, it's side effect is to assign all the `this` properties you defined in the constructor body to the object and return it. A constructor function has a `prototype` property which points to the prototype object of the constructor. The object created by the constructor has it's [[prototype]] pointing to the constructor's prototype object. Inheritance is implemented by assigning super object to sub constructor's prototype. Take a look at this:
``` javascript
function Sup() {
    this.x = 'x';
    this.y = 'y';
}

function Sub() {
    this.a = 'a';
}

Sub.prototype = new Sup();

var sub = new Sub();
console.log(sub.x); // x

```
The sub object has no x itself, so it will find x in it's prototype which is a super object. This is cool, but the problem comes when there is reference property in super object.
``` javascript
function Sup() {
    this.x = 'x';
    this.z = ['z'];
}

function Sub() {
    this.a = 'a';
}

Sub.prototype = new Sup();

var sub1 = new Sub();
var sub2 = new Sub();
sub1.z.push('zz');
console.log(sub1.z); // ['z', 'zz']
console.log(sub2.z); // ['z', 'zz'], !expected to be ['z']
```
Both sub1.z and sub2.z are pointing to the same z in super object. Any sub object's operation on z will have impact on all sub objects. That's dangerous!

### Classical inheritance by constructor stealing
Now we see the problem in prototype chaining inheritance, is there a way to solve it?
``` javascript
function Sup() {
    this.z = ['z'];
}

function Sub() {
    Sup.call(this);
    this.a = 'a';
}

var sub1 = new Sub();
var sub2 = new Sub();
sub1.z.push('zz');
console.log(sub1.z); // ['z', 'zz']
console.log(sub2.z); // ['z'], !cool
```
In sub constructor we steal super constructor and call it with `this` reference. Now that every sub object has a copy of z and will not interfere with each other. This solves the problem in prototype chaining inheritance, but it's downside is every property including method to be inheritted should be defined inside the super constructor. So there is no method reuse. How about we combine constructor stealing and prototype chaining together?

### Combination inheritance (pseudoclassical inheritance)
Both prototype chaining inheritance and constructor stealing inheritance have downsides, what if we combine them together?
``` javascript
function Sup() {
    this.z = ['z'];
}
Sup.prototype.mo = function () {
    console.log('mo');
};

function Sub() {
    Sup.call(this);
    this.a = 'a';
}
Sub.prototype = new Sup();

var sub1 = new Sub();
var sub2 = new Sub();

sub1.z.push('zz');
console.log(sub1.z); // ['z', 'zz']
sub1.mo(); // mo
console.log(sub2.z); // ['z']
sub2.mo(); // mo
```
That's cool, we don't need to define methods in super constructor's body and they are reusable now. The combination inheritance addresses both downsides of prototype chaining and constructor stealing. It's widely used in real life. It also preserves the behavior of `instanceof` and `isPrototypeOf` which looks like perfect now. But, yeah there is always another but :), if you look deep into the pattern, you might find that the super constructor has been called twice, `Sup.call(this)` and `Sub.prototype = new Sup()` which means the set of properties are duplicated in memory. This is an unavoidable ineffecient part of combination inheritance. And on another hand, combination inheritance still relies on constructor function which is kind of like class inheritance, while JS is really good at inheritance on objects. [Douglas Crockford](http://www.crockford.com) has introduced prototypal inheritance in 2006 based on the premise that inheritance on objects.

### Prototypal inheritance
So how to do inheritance directly on objects without constructors?
``` javascript
function create(supObj) {
    function Empty() {};
    Empty.prototype = supObj;
    return new Empty();
}
```
Crockford introduced a factory like function to create the sub object based on the passed in super object. So super constructor is not a must now. But the type of the sub object is lost. Based on this create function, Crockford introduced parasitic inheritance which is closer to real life.

### Parasitic inheritance
Much like prototypal inheritance, parasitic inheritance just encapsulates all the properties or methods for the sub object in one function.
``` javascript
function createAnother(supObj) {
    var sub = create(supObj);
    
    sub.a = 'a';
    sub.mo = function () {
        console.log('mo');
    };
    return sub;
}
```
Now the sub object has all the properties of super object and it's own property a and method mo. Parasitic inheritance addresses the duplicated properties downside in combination inheritance, but it loses object type which means `instanceof` and `isPrototypeOf` don't work for sub object any more. Let's keep improvement, finally we work out a way combining constructor stealing and parasitic inheritance.

### Parasitic combination inheritance

``` javascript
function create(supObj) {
    function Empty() {};
    Empty.prototype = supObj;
    return new Empty();
}

function inheritPrototype(Sub, Sup) {
    var subPrototype = create(Sup.prototype);
    subPrototype.constructor = Sub; // link constructor back to Sub
    Sub.prototype = subPrototype;
}

function Sup() {
    this.x = 'x';
    this.z = ['z'];
}

Sup.prototype.mo = function () {
    console.log('mo');
};

function Sub() {
    Sup.call(this);
    this.a = 'a';
}

inheritPrototype(Sub, Sup);

var sub1 = new Sub();
var sub2 = new Sub();
sub1.z.push('zz');
sub1.mo(); // mo
cnosole.log(sub1.z); // ['z', 'zz']
sub2.mo(); // mo
console.log(sub2.z); // ['z']

console.log(sub1 instanceof Sub); // true
console.log(Sub.prototype.isPrototypeOf(sub2)); //true
```
Now both the downside in combination inheritance - duplicated properties and the downside in parasitic inheritance - object type lost are addressed. We are kind of having a perfect inheritance machenism now. BUT...personally I will not suggest to use inheritance heavily unless you want to mess yourself up.