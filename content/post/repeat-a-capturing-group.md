+++
categories = ["tech"]
date = "2014-12-23T01:41:37+08:00"
draft = false
slug = ""
tags = ["regex"]
title = "Repeat a capturing group, pitfall in RegExp"

+++

RegExp is esoteric, known to all, but you will find it incredibly effecient when you start to get used to it. I like to use RegExp, but sometimes I tend to make mistakes, the most common of which is to repeat a capturing group. Let's think about this. Given a string `'a=1;b=2;...;k=n'`, I would like to capture `k, n` of each pair to form an array like `['a', 1, 'b', 2, ... 'k', n]`.

<span class="more"></span>

Below is our first try:

```javascript
var s = 'a=1;b=2;c=3';
var r = /(?:(\w)=(\d))?(?:;(\w)=(\d))*/;

// expected to be ['a', 1, 'b', 2, 'c', 3]
s.match(r).slice(1, 7); // ['a', 1, 'c', 3] !ops
```

As you can see, `b, 2` is lost. What happens behind is, when RegExp engine finishes matching for group 3 (which is 'b') and group 4 (which is 2) it finds the `*` which makes it redo the match for group 3 and 4 from last index. So `b, 2` is overridden by `c, 3`. Here we are trying to repeat a capturing group `(?:;(\w)=(\d))*` which leads to the unexpected result.

So can we achieve the goal by global matching? Let's do the second try:
```javascript
var s = 'a=1;b=2;c=3';
var r = /(?:(\w)=(\d))/g;

// expected to be ['a', 1, 'b', 2, 'c', 3]
s.match(r); // ['a=1', 'b=2', 'c=3'] !ops
```
Looks like it's even further from what we expected. That's true, in global matching, capturing group lose it's magic and it always return the matching result as a whole RegExp.

Finally we yield to use some JavaScript snippet, various ways to go, not to address here.

---
http://www.regular-expressions.info/captureall.html