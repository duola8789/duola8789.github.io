---
title: Node12 AMD、CMD、UMD
top: false
date: 2018-07-05 17:49:17
updated: 2019-04-30 11:10:46
tags:
- AMD
- CMD
- UMD
categories: Node
---

重新学习Node，整理以前的日志。AMD/CMD/UMD学习笔记。

<!-- more -->

## AMD

AMD 是"Asynchronous Module Definition"的缩写，意思就是"异步模块定义"，也是由RequireJS定义的模块形式。

它采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。

### 定义模块

```JS
// 文件名: foo.js
define(['jquery'], function ($) {
  // 方法
  function myFunc(){};
 
  // 暴露公共方法
  return myFunc;
});
```
定义的第一个部分是一个依赖数组，第二部分是回调函数，只有当依赖的组件可用时回调函数才会执行。

### 加载模块

```JS
require(['foo'], function(foo) {
  console.log(foo)
})
```

## CommonJS

Node.js的模块和包机制的实现参照了CommonJS的标准，但并未完全遵循。不过两者的区别不大。

CommonJS规范加载模块是同步的，也就是说，只有加载完成，才能执行后面的操作。

### 定义模块

在CMD规范中，一个模块就是一个文件。`define`是一个全局函数，用来定义模块。

`define`接受`factory`参数，`factory`可以是一个函数，也可以是一个对象或字符串。

比如可以定义一个JSON数据模块：

```
define({"foo": "bar"});
```
`factory`是一个函数，有三个参数，`function(require, exports, module)`

1. `require`是一个方法，接受模块标识作为唯一参数，用来获取其他模块提供的接口：`require(id)`
2. `exports`是一个对象，用来向外提供模块接口
3. `module`是一个对象，上面存储了与当前模块相关联的一些属性和方法

```JS
define(function(require, exports, module) {
  var a = require('./a');
  a.doSomething();
  // 依赖就近书写，什么时候用到什么时候引入
  var b = require('./b');
  b.doSomething();
});
```
### NodeJS中

```JS
// 文件名: foo.js
// 依赖
var $ = require('jquery');
// 方法
function myFunc(){};
 
// 暴露公共方法（一个）
module.exports = myFunc;
```
## UMD通用模块规范

UMD是AMD和CommonJS的糅合。

```JS
((root, factory) => {
  if (typeof define === 'function' && define.amd) {
    //AMD
    define(['jquery'], factory);
  } else if (typeof exports === 'object') {
    //CommonJS
    var $ = requie('jquery');
    module.exports = factory($);
  } else {
    //都不是，浏览器全局定义
    root.testModule = factory(root.jQuery);
  }
})(this, ($) => {
  //do something...  这里是真正的函数体
});
```

更详细的内容看[这个教程](http://javascript.ruanyifeng.com/tool/requirejs.html#toc3)。


## 参考
- https://github.com/hstarorg/HstarDoc/blob/master/%E5%89%8D%E7%AB%AF%E7%9B%B8%E5%85%B3/%E8%AE%A4%E8%AF%86AMD%E3%80%81CMD%E3%80%81UMD%E3%80%81CommonJS.md
- http://web.jobbole.com/82238/
- https://mp.weixin.qq.com/s/WG_n9t4E4q0kBWczkSEdEA
- https://neveryu.github.io/2017/03/20/amd-cmd/
