+++
title = "判断控制台是否开启(chrome)"
date = 2019-06-02T20:08:23+08:00
tags = ["javascript"]
description = "判断控制台是否开启(chrome)"
+++

最近在网上瞎逛看到stack overflow上的这个老话题[ Find out whether Chrome console is open ](https://stackoverflow.com/questions/7798748/find-out-whether-chrome-console-is-open/30638226#30638226)很有意思，就决定实践学习一下。

**判断控制台是否开启**这种需求虽然没遇见过，但感觉还是有可能会遇到的，特别是做一些简单的安全限制，稍微的提高一下安全门槛。

至于如何实现这个需求，大佬们也给出了[解法](https://stackoverflow.com/a/7809413)
```js
var devtools = /./;
devtools.toString = function() {
  this.opened = true;
}

console.log('%c', devtools);
// devtools.opened will become true if/when the console is opened
```

经实验并不可行，我稍微改了下
```js
function checkOpen() {}
checkOpen.toString = function() {
  this.opened = true;
};
console.log("%c", checkOpen);
// checkOpen.opened will become true if/when the console is opened
```

当然这样并不能很好的观察结果，所以我又加了些东西搞了个[小demo](https://learn.eon-lee.site/build/detect-console-open/)
![detect-devtool-open.gif](./images/detect-devtool-open.gif)
[codepen](https://codepen.io/3wellh/pen/OYoGNq)

当开启控制台后会显示已开启

## 原理学习

首先，这种方法其实是一个hack来的，涉及的知识点包括[类型转换](https://yanhaijing.com/es5/#102)以及[console.log api](https://developer.mozilla.org/en-US/docs/Web/API/console)。
具体理解上就是首先先定义一个具名空函数`checkOpen`，然后覆写它的`toString`方法。
之后执行`console.log('%c',checkOpen)`。当`console.log()`遇上`%c`这个[Substitution string](https://developer.mozilla.org/en-US/docs/Web/API/console#Outputting_text_to_the_console)时，第二个参数是字符串，因此当传入的是checkOpen这个function是就会进行类型转换，触发checkOpen的`toString`方法。当然这里也可以用`console.log('%s',checkOpen)`。
而因为`console.log()`这个api只有在devtool打开时才会真正的执行，也就是说只有控制台开启后checkOpen的`toString`方法才会执行。
```js
控制台开启 => console.log执行 => 触发类型转换调用toString
```
利用这些特性就能很好的监控控制台是否开启。

前面有说到stack overflow上的答案已经没有作用(chrome 74)，是因为较新版本的chrome在`console.log()`正则表达式对象时，不会调用`toString`或者`valueOf`。

~~只要改成这样就可以生效~~
```js
/**var devtools = /./;
devtools.__proto__.toString = function() {
  this.opened = true;
};
console.log(1 + devtools);*/
```

## 结语
最基础的语言知识，灵活运用组合起来后就能实现这种第一反应可能认为没办法实现的需求，果然基础很重要，思维更重要啊。

## 其他
为了预防以后真的遇到这种需求。我就写了[小模块](https://github.com/eon-lee96/detect-devtool-open)个发到[npm](https://www.npmjs.com/package/@eonlee/detect-devtool-open)了。
然后我就发现一个做的更好更完善的包了[devtools-detector](https://github.com/AEPKILL/devtools-detector)，考虑了浏览器兼容性以及console方法可能被hook的场景。
后续学习大佬的代码后可以继续丰富此文和优化自己的小模块。

## 参考资料
* [类型转换与测试](https://yanhaijing.com/es5/#102)
* [console.log api](https://developer.mozilla.org/en-US/docs/Web/API/console)
* [stack overflow 话题](https://stackoverflow.com/questions/7798748/find-out-whether-chrome-console-is-open/30638226#30638226)
