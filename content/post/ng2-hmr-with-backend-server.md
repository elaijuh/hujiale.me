+++
categories = ["tech"]
date = "2016-09-03T20:53:14+08:00"
draft = false
slug = ""
tags = ["angular2"]
title = "Angular2 HMR with backend server supported"

+++


Currenly I am developing a client + server side boilerplate with [Angular 2](https://angular.io) and [Feathers](http://feathersjs.com).
For server side, I am using `ts-node` with nodemon, so far so good. But I find it cumbesome that every time I need to bundle client side code.  
After some exploring, I find a way to solve the problem. These are the dependencies:

- [angular2-hmr](https://github.com/AngularClass/angular2-hmr) a bootload wrapper on `bootstrapModule`, it's cleared classified how to use in github
- [angular2-hmr-loader](https://github.com/AngularClass/angular2-hmr-loader), a webpack loader to work with the previous one
- [webpack-dev-middleware](https://github.com/webpack/webpack-dev-middleware), just like `webpack-dev-server`
- [webpack-hot-middleware](https://github.com/glenjamin/webpack-hot-middleware), an `express`/`feather` middleware just like `webpack-dev-server` w/ hot


All of their READMEs clearly walk you through, it just suprises me that few information could be connected on this while it's a quite general practice in `React`





