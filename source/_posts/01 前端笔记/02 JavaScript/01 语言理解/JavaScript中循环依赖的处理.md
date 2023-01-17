---
title: JavaScript中循环依赖的处理
date: 2021-11-26 20:21:51
updated: 2021-11-26 20:21:54
tags:
- Webpack
- CommonJS
- ESM
- Node

categories: JavaScript
---

JavaScript中循环依赖的处理。

<!-- more -->
# 什么是循环依赖

循环依赖一般会伴随模块化一起出现，就是在`a`模块中依赖`b`模块，而`b`模块又依赖`a`模块。

在以前开发时就遇到过这种情况，在`store`初始化时使用了`utils`模块中的方法，而`utils`中有利用到了`store`中的数据

在开发时没有问题，但是在打包后运行时就报错了，实际上就是遇到了循环依赖的问题。

对于循环依赖，Node默认的CommonJS模块和ES6的模块以及Webpack打包时的处理各不相同

# CommonJS中对循环依赖的处理

可以看Node官方对[循环依赖的介绍](https://nodejs.org/api/modules.html#modules_cycles)：

`a.js`，通过`require`，引用了`b`模块：

```js
console.log('a start');
exports.done = false;

const b = require('./b.js');
console.log('in a, b.done = %j', b.done);

exports.done = true;
console.log('a done');
```

在`b.js`中，通过`require`引用了`a`模块：

```js
console.log('b start');
exports.done = false;

const a = require('./a.js');
console.log('in b, a.done = %j', a.done);

exports.done = true;
console.log('b done');
```

在`main.js`中，先后引用`a`和`b`模块：

```js
console.log('main start');

const a = require('./a.js');
const b = require('./b.js');

console.log('in main, a.done = %j, b.done = %j', a.done, b.done);
```

然后执行`node main.js`，输出结果会是什么呢：

```bash
main starting

a starting

b starting

in b, a.done = false

b done

in a, b.done = true

a done

in main, a.done = true, b.done = true
```

在运行`a`模块是，遇到了`require('b')`，就去运行`b.js`，在里面又遇到了`require('a')`，为了避免循环引用，一个未执行完成的`a.js`的`exports`的副本（`unfinished copy`）会作为`require('a')`的结果返回给`b`

所以这时候，在`b`中`a.done`是`false`，执行完成`b`后，继续执行`a`模块，`a`模块的`b.done`也就变成了`true`

从上面的例子可以看出来，CommonJS模块对循环依赖进行了很好的处理，主要依赖于它的两个特点：

1. 模块在运行时加载
2. 会缓存已加载（包括未完成的）模块

# ESM的处理

在`a.mjs`中通过`import`获取`b.mjs`中导出的`bar`，`b.mjs`通过`import`获取`a.mjs`中的`foo`

> `.mjs`用来标识使用了ES6 Modoule

`a.mjs`中：

```js
import {bar} from './b.mjs';

console.log('a.mjs');
console.log(bar);

export const foo = 'foo';
```

`b.mjs`中：

```js
import {foo} from './a.mjs';

console.log('b.mjs');
console.log(foo);

export const bar = 'bar';
```

然后在Node环境下执行：

```bash
node --experimental-modules a.mjs
```

执行结果：

```bash
b.mjs
console.log(foo);
            ^
ReferenceError: Cannot access 'foo' before initialization
```

根据阮一峰老师的讲解，在执行`a.mjs`后，引擎发现加载了`b.mjs`，然后优先执行`b.mjs`。

在执行`b.mjs`时，发现从`a`中导入了`foo`，这时不会去执行`a.mjs`，会认为`foo`已经存在，继续完成执行，直到运行到`console.log(foo)`时，才发现`foo`根本没定义，所以报错了

如果将`a.mjs`中的最后一个变量的声明有`const`改为`var`，由于`foo`拥有了变量提升，输出结果就发生了变化，不在报错：

```js
b.mjs
undefined
a.mjs
bar
```

上面的结果也是符合ESM的特性：

1. ESM模块输出的是值的引用
2. 输出接口动态执行
3. 静态接口

# Webpack对循环依赖的处理

首先安装了`Webpack`和`Webpack-CLI`：

```bash
npm install webpack webpack-cli -D
```

然后在项目中新建了`webpack.config.js`配置文件：

```js
const path = require('path');

module.exports = {
  entry: path.resolve(__dirname, 'demo12/commonjs/index.js'),
  output: {
    path: path.resolve(__dirname, 'demo12/dist'),
    filename: 'my-first-webpack.bundle.js'
  },
};
```

配置了一个最简单的打包配置，然后运行`package.json`中配置好的`webpack`命令后：

```bash
> webpack

asset my-first-webpack.bundle.js 533 bytes [compared for emit] [minimized] (name: main)
./demo12/commonjs/index.js 193 bytes [built] [code generated]
./demo12/commonjs/a.js 203 bytes [built] [code generated]
./demo12/commonjs/b.js 203 bytes [built] [code generated]
```

Webpack没有对循环依赖做出任何检测，打包过程也没有任何报错，在浏览器中执行打包后的结果，与CommonJS的结果完全相同

如果需要让Webpack对循环依赖做出检测，需要使用[circular-dependency-plugin](hhttps://github.com/aackerman/circular-dependency-plugin)这个插件：

```bash
npm i circular-dependency-plugin -D
```

然后在`webpack.config.js`中添加如下配置：

```js
const path = require('path');
const CircularDependencyPlugin = require('circular-dependency-plugin');

module.exports = {
  entry: path.resolve(__dirname, 'demo12/commonjs/index.js'),
  output: {
    path: path.resolve(__dirname, 'demo12/dist'),
    filename: 'my-first-webpack.bundle.js'
  },
  plugins: [
    new CircularDependencyPlugin({
      exclude: /node_modules/,
      include: /demo12/,
      failOnError: true,
      allowAsyncCycles: false,
      cwd: process.cwd()
    })
  ]
};
```

再执行打包，结果插件对循环依赖做出了检测，并根据我们的配置让打包失败了：

![](https://image.oldzhou.cn/FlMfpNJAjyy8qpyPHxW-Jru-ODBd)

# 参考

- [ webpack模块化原理解析（五）—— webpack对循环依赖的处理@Zachary's blog](https://sirius-desktop-web.cowork.netease.com/#?writeMailToContact=maobuyi@lx.elysys.net)
- [Cycles@Node.js](https://nodejs.org/api/modules.html#modules_cycles)
