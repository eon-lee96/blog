+++
title = "前端复习——立即调用函数表达式(IIFE)"
date = 2019-01-20T22:53:31+08:00
tags = ["前端复习"]
draft = false
description = "复习立即执行函数IIFE"
+++

## 概念理解

所谓立即执行函数表达式就是用来表达一个可以立即执行的函数的式子。

一个函数只要声明后显式写上调用语句即可立即执行。

如:
``` javascript
function fool(){}
fool();
```
便是一个立即执行的函数了。

但它并不是一个`IIFE`，因为它只是声明语句+表达式(函数低调用)而不是单纯一个表达式。

表达式的概念[戳这里](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Expressions_and_Operators#%E8%A1%A8%E8%BE%BE%E5%BC%8F)。

温习了什么是表达式之后可能会写出这样的代码:

``` javascript
function foo(){}() //Uncaught SyntaxError: Unexpected token )
```
或
``` javascript
function(){}() // Uncaught SyntaxError: Unexpected token (
```

这两种都会报错的，第一种可以看作是函数声明后执行`()`，固然是报错的，因为圆括号里面需要有表达式。
而第二种错的原因是直接使用书写一个匿名函数的话，js引擎会当作声明语句，而函数声明语句要求一定要有函数名。

也许会想到可以这样写
``` javascript
var foo = function(){}()
```
确实不会报错，但依旧这不是一个表达式。
实际上这里是吧`function(){}`当作左值表达式了，赋值后再调用。

也就是说只要保证`fucntion(){}`能够被js引擎当作表达式来处理而不是声明语句就没问题。
正如前面说到`()`里面必须是表达式，那如下代码是不是就不会报错？

``` javascript
(function(){})
```
的确是不报错的，这里的意思其实就是表达一个匿名函数，当然，具名也是可以的，但没必要占用全局命名空间。

上面也说了，函数调用也是个表达式，所以我们组合一下

``` javascript
( function(){}() )
```
以及
``` javascript
( function(){} )()
```
两种都是`IIFE`。

其实我们的目的就是让函数声明变成一个表达式，然后调用这个表达式所表达的值，让函数变成表达式的方式其实不止`()`，组合上逻辑运算符、赋值等：
``` javascript
  var i = function(){ return 10; }();
  true && function(){ /* code */ }();
  0, function(){ /* code */ }();
   -function(){ /* code */ }();
  +function(){ /* code */ }();
```
都能让函数声明变成表达式，但都有一些副作用，相比下`()`是相对较好的了。

## 用途

扯了那么久概念是时候聊聊`IIFE`存在的意义，也就是它能用来干嘛。


### 隔离作用域

我们知道浏览器加载一段脚本后就会立刻的解释执行，这个过程中会产生许多的全局变量污染全局作用域，解决这个问题的最好办法就是利用函数能够创建自己作用域的特性，然而如果单纯把代码放进函数而不调用的话，浏览器加载解析完什么都不会发生，此时就需要`IIFE`

也就是说`IIFE`的一大作用就是能够让脚本被加载解析执行完就能马上创建一个独立的作用域，就很好的隔离作用域了。


### 惰性函数

先po[`惰性函数的参考文章`](http://peter.michaux.ca/articles/lazy-function-definition-pattern)。

在我的理解里所谓惰性函数就是一个函数内部的分支只会在第一次执行的时候走到，之后这个函数便会被覆盖，之后再次调用此函数时将不会走到内部的分支，而是直达目标。

结合`IIFE`就能做到脚本一旦解释运行了，便对函数进行第一次调用，然后后续就可以惰性的使用此函数。

最常见于浏览器兼容的一些判断，如`跨浏览器的事件监听`.

原始版本：
``` javascript
function addEvent(type, el, fn) {
  if (window.addEventListener) {
    el.addEventListener(type, fn, false);
  } else if (window.attachEvent) {
    el.attachEvent('on' + type, fn);
  }
}
```

这样写没啥不对，但是每次执行的时候都要进行一次分支判断，但实际上一旦开始执行了，浏览器的环境基本上是不变了吧。。每次都判断就冗余了。这时候就需要惰性函数：

``` javascript
function addEvents(el, type, fn) {
  if (window.addEventListener) {
    addEvents = function (type, el, fn) {
      el.addEventListener(type, fn, false);
    }
  } else if (window.attachEvent) {
    addEvents = function (type, el, fn) {
      el.attachEvent('on' + type, fn);
    }
  }
}
```

这样就只在第一次执行的时候进行判断，然后函数被重写为浏览器版本对应的事件监听方式。

再进一步能否在脚本加载完立马就能让addEvents是正确的？有了`IIFE`当然可以:

``` javascript
var addEvent = (function () {
  if (window.addEventListener) {
    return function (type, el, fn) {
      el.addEventListener(type, fn, false);
    }
  } else if (window.attachEvent) {
    return function (type, el, fn) {
      el.attachEvent('on' + type, fn);
    }
  }
}());
```

### 外部脚本(external javascript file)

这个就比较好理解了，正如前文所说，`IIFE`能够保证脚本可以立即执行的同时并且形成隔离作用域，那对于一些需要引入即可执行而不需要事件触发或显示调用的外部脚本来说就很有用了，比如一些第三方的广告、统计库等，只需script引入即可生效，还不会污染全局空间，本质就是`IIFE`。

## 参考资料

[Immediately-Invoked Function Expression (IIFE)](http://benalman.com/news/2010/11/immediately-invoked-function-expression/#iife)

[IIFE](https://developer.mozilla.org/zh-CN/docs/Glossary/%E7%AB%8B%E5%8D%B3%E6%89%A7%E8%A1%8C%E5%87%BD%E6%95%B0%E8%A1%A8%E8%BE%BE%E5%BC%8F)

[JavaScript专题之惰性函数](https://github.com/mqyqingfeng/Blog/issues/44)
