+++
categories = ["tech"]
date = "2014-12-11T22:56:24+08:00"
draft = false
slug = ""
tags = ["angular1"]
title = " $provide.decorator - tweak Angular servicer"

+++

If you are using Angular, you can't avoid depending on third-party modules. Sometimes you find it not that perfect API the module provides, so that you hack into the source code and about to pull request. But the author's comments on your pull request, usually for popular repo, 'sorry-that's not our priority' turns you down totally. Either you work around it or turns to another third-party module. Here is the third option, if you want to share the private data in service closure and provide some API for your specific need, you might find `$provide.decorator` helpful. Let's look at this:

<span class="more"></span>

```javascript
angular
    .module('sortModule', [])
    .factory('sortService', function () {
        var list = [1, 3, 10, 2];
        var service = {
            doSort: doSort
        }
        return service;
        ////////////////
        function doSort() {
            return list.sort().slice(); // [1, 10, 2, 3] default sort method compared by string
        }
    });

var myApp = angular.module('myApp', ['sortModule']);
myApp.config(function ($provide) {
    $provide.decorator('sortService', function ($delegate) {
        $delegate.doCustSort = function (fn) {
            return list.sort(fn).slice(); // [1, 2, 3, 10] compared by number
        };
        
        return $delegate;
    });
});

myApp.controller('MyCtrl', function($scope, sortService) {
    console.log(sortService.doSort()); // [1, 10, 2, 3]
    var compare = function (a, b) {
        if (a < b) return -1;
        if (a > b) return 1;
        return 0;
    };
    // we tweaked sortService with doCustSort method
    console.log(sortService.doCustSort(compare)); // [1, 2, 3, 10]
});
```
Here we add a customized sort method `doCustSort` to `$delegate` which references to the service itself. So that we can share the private `list` and do some specific logic on our need. And basically you can tweak most components in Angular as long as it's a `$provide`.

[Plnkr demo](http://plnkr.co/edit/etgYtx?p=preview)


