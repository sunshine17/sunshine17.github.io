---
layout: post
category: "ionic"
title:  "Ionic 1 如何使用Native Storage"
tags: [ionic nodejs]
summary: "Ionic 2 自带了native storage的库，但Ionic 1并没有，所以本文主要讨论如何在Ionic 1 的项目中添加native storage的支持，以实现native的持久化存储方案。"
---

Ionic 2 自带了native storage的库，但ionic 1并没有，所以本文主要讨论如何在ionic 1 的项目中添加native storage的支持，以实现native的持久化存储方案。

## 用cordova命令添加native storage plugin

```shell
cordova plugin add https://github.com/TheCocoaProject/cordova-plugin-nativestorage
```
若看到以下输出，表示安装成功。其实原本cordova的命令是可以直接通过插件名安装的，但我的尝试显示cordova的repo应该是被和谐了，所以只能通过上面这种直接指定git repo的方式安装。

```shell
Fetching plugin "https://github.com/TheCocoaProject/cordova-plugin-nativestorage" via git clone
Installing "cordova-plugin-nativestorage" for android
Installing "cordova-plugin-nativestorage" for ios
```

## 下载native storage的前端js库

### 通过bower安装

```shell
bower install git://github.com/TheCocoaProject/ngcordova-wrapper-nativestorage --save-dev
```

### 在index.html中引入库

```html
<script src="lib/ngcordova-wrapper-nativestorage/dist/ngcordova-wrapper-nativestorage.min.js"></script>
```

## 在app.js中添加module依赖

``` nodejs
angular.module('app', [
	'ionic',
	'ngCordova',
	'ngCordova.plugins.nativeStorage',	// THIS IS WHAT WE NEED REALLY !
	'app.controllers',
	'app.routes',
	'app.directives',
	'app.services',
	'ionic.cloud',
	'ionic.cloud.init',
])
```

## **最后最重要的：必须手动初始化angular**

- 在index.html中，把`body`标签中的`ng-app`指令去除: 
    `<body ng-app="app">` 变成 `<body>`
- 在html中加入以下js来手动初始化angular

```html
<script>
  angular.element(document).ready(function () {
    if (window.cordova) {
        console.log("Running in Cordova, will bootstrap AngularJS once 'deviceready' event fires.");
        document.addEventListener('deviceready', function () {
            console.log("Deviceready event has fired, bootstrapping AngularJS.");
            angular.bootstrap(document.body, ['app']);
        }, false);
    } else {
        console.log("Running in browser, bootstrapping AngularJS now.");
        angular.bootstrap(document.body, ['app']);
    }
  });
</script>
```

## 调试模式启动APP
在`serve`或`run android`命令后加上`--livereload --consolelogs`选项就可以看到APP启动时的日志。

```shell
ionic serve --livereload --consolelogs
```

