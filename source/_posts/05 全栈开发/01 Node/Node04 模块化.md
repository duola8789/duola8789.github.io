---
title: Node04 模块化
top: false
date: 2017-04-03 17:54:17
updated: 2019-04-30 11:01:43
tags:
- Exports
- Module
- Require
- Import
- Export
categories: Node
---

重新学习Node，整理以前的日志。Node中模块化的学习笔记。

<!-- more -->

## 几种形式

模块导入和导出有几种形式：

1. `require`：Node和ES6都支持的引入
2. `export`/`import`：只有ES6支持的导出/引入格式
3. `module.exports/exports`：只有Node支持的导出格式

## Node

Node的模块系统遵循的是CommonJS规范，CommonJS定义的模块分为模块标识（`module`）、模块定义（`exports`）、模块引用（`require`）

### `exports`和`module.exports`

Node在执行一个文件时，会给这个文件内生成一个`exports`和`module`对象，`module`又有一个`exports`属性，他们的关系如下，都指向一个内存区域：

```JS
exports = module.exports = {};
```
![内存结构示意图](http://image.oldzhou.cn/18-7-9/54544823.jpg)

如果人为将二者指向不同的内存区域，最终导出的内容是`module.exports`的内容，而不是`exports`的内容，所以**尽量都使用`module.exports`导出，用`require`导入**

### `require`

Node.js提供了`exports`和`require`两个对象，其中`exports`是模块公开的接口，`require`用来从外部获取一个模块接口，即获取模块的`exports`对象。

Node.js可以加载的文件模块分为三种：

- `.js`，通过`fs`模块同步读取JS文件并编译执行
- `.node`，通过C/C++进行编写的Addon。通过`dlopen`方法进行加载
- `.json`，读取文件，调用`JSON.parse`解析加载

`require`可以接受以下几种参数的传递：

- `lodash`、`jQuery`等原生模块
- 相对路径的文件模块
- 绝对路径的文件模块
- 非原生模块的文件模块

在引用文件模块的时候后要加上文件的路径：

- `/.../.../xxx.js`表示绝对路径、
- `./xxx.js`表示相对路径(同一文件夹下的`xxx.js`)
- `../`表示上一级目录
- 如果既不加`/.../、../`又不加`./`的话，则该模块要么是核心模块，要么是从一个`node_modules`文件夹加载

### 小例子

`test1.js`文件：

```JS
const tt1 = function (a, b) {
  return a + b;
};

function tt2(a, b) {
  return a * b;
}

module.exports.tt1 = tt1;
module.exports.tt2 = tt2;
```

`test2.js`文件

```JS
const test1 = require("./test1.js");

console.log(test1.tt1(1, 2));//3
console.log(test1.tt2(1, 2));//2
```

在命令行执行

```BASH
node test2.js
```

上面代码要注意：

1. 被引用的`test1.js`需要将要导出的对象存入`module.exports`（或者是`exports`）中
2. 在`test2.js`文件中引入`test1.js`的结果，引入的就是`module.exports`对象，该对象对应的方法就是在`test1.js`中定义的函数
3. 在使用`require`时要表明路径，否则会找不到对应的文件。

## ES6的模块化

ES6中的规范是使用`export`和`import`来导出和导入模块

导出的时候有的时候会增加一个`default`关键字，它其实就是一个语法糖：

```JS
// test.js
const a = 123;

export default a

// 等同于 export { a as default }
```
引入的时候就可以直接使用任意变量名引入：

```JS
import x from './test.js'

// 等同于 impor { default as x } from './test.js'
```
要注意的是一个模块里面只能有一个`export default`。


还要注意，ES6导出的模块是静态的，我是这样理解的：导出的只是一段代码，并没有实际运行，导出的如果是函数也没有执行

## 参考
- [exports、module.exports 和 export、export default 到底是咋回事@掘金](https://juejin.im/post/597ec55a51882556a234fcef) 
