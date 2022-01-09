+++
categories = ["tech"]
date = "2015-06-17T01:27:00+08:00"
draft = false
slug = ""
tags = ["angular1"]
title = "Unit test Angular directive"

+++

I seldom test angular directive unless there is some DOM mutation being processed in the directive, like add/remove classes, show/hide elements or compile/destroy elements. If you have a heavy  dependencies on directives, the unit test will be a little bit clunky as for mocking the inline controller of the required directive. Stack overflow gives several alternatives to do so, I am picking two of them which I prefer to go  and demo them here. At the bottom of this post, I will attach the SO link for reference.

<span class="more"></span>

Let say we have two directives `foo` and `bar` here and bar is requiring foo for it's controller. We want to test directive `bar`.
```javascript
// index.html
// <foo><bar>{{bar}}</bar></foo>

// app.js
app
  .directive('foo', function () {
    return {
      restrict: 'EA',
      controller: function () {
        this.getFoo = function () {
          return 'foo';
        };
      },
      link: angular.noop
    }
  })
  .directive('bar', function () {
    return {
      restrict: 'EA',
      require: '^foo',
      link: function (scope, elm, attrs, controller) {
        var fooCtrl = controller;
        scope.bar = fooCtrl.getFoo() + 'bar';
      }
    };
  });
```

###  Bind controller to element.data
Fundamentally Angular will bind directive's inline controller object to it's DOM through data property. When child directive is requiring parent directive, it will finally find the controller object in element.data. Based on this mechanism, we can mock parent directive controller by element.data.
```javascript
// speca.js

var $scope, element;
beforeEach(inject(function($rootScope, $compile) {
  $scope = $rootScope.$new();
  element = angular.element('<bar></bar>');
  // bind the controller to element.data
  element.data('$fooController', {
    getFoo: function () { return 'foo'; }
  });
  $compile(element)($scope);
}));

it('should bind bar to scope', function() {
  expect($scope.bar).toEqual('foobar');
}); 
```

### Inject the directive
Another way is to inject the parent directive and mock it's controller property directly. This is also a straightforward way. But you might need to compile from parent directive even if you only want to test the child directive. See the example below:
```javascript
var $scope, element;
beforeEach(inject(function($rootScope, $compile, _fooDirective_) {
  var fooDirective = _fooDirective_[0];
  fooDirective.controller = function () {
    this.getFoo = function () { return 'foo'; };
  };
    
  $scope = $rootScope.$new();
  // compile from parent directive foo
  element = angular.element('<foo><bar></bar></foo>');
  $compile(element)($scope);
}));

it('should bind bar to scope', function() {
  expect($scope.bar).toEqual('foobar');
});
```

I have  a [plunker](http://plnkr.co/edit/CUvQi8LQQ13fwHdgzgpw?p=preview) here to demo both ways of unit testing. And all that shedding light on me is this [SO](http://stackoverflow.com/questions/19227036/testing-directives-that-require-controllers) post.

