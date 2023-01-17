---
title: Node11 process.env对象
top: false
date: 2018-07-05 09:58:17
updated: 2019-04-30 11:09:04
tags:
- process
categories: Node
---

重新学习Node，整理以前的日志。process.env对象的学习笔记。

<!-- more -->

## `process`对象

`process`对象是Node的一个全局独享，提供当前Node进程的信息。他可以在脚本的任意位置使用，不必通过`require`命令加载

## 属性

`process`对象提供了一系列的属性，用于返回系统信息

- `process.argv`：返回一个数组，成员是当前进程的所有命令行参数
- `process.env`：返回一个对象，成员为当前Shell的环境变量
- `process.pid`：返回一个数字，表示当前进程的进程号
- `process.platform`：返回一个字符串，表示当前的操作系统，比如Linux
- `process.version`：返回一个字符串，表示当前使用的 Node 版本，比如v7.10.0

## `process.env`

`process.env`返回一个对象，包含了当前Shell的所有环境变量

通常的做法是，新建一个环境变量`NODE_ENV`，用它确定当前所处的开发阶段，生产阶段设定为`production`，开发阶段设定为`development`，然后在脚本中读取`process.env.NODE_ENV`

运行脚本时改变环境变量可以采用下面的写法：

```BASH
NODE_ENV=production node app.js
```
## `ross-env`

如果按照上面的写法，在windows系统下是会报错的：

```BASH
'NODE_ENV' 不是内部或外部命令，也不是可运行的程序或批处理文件。
```

因为windows下不支持这种设置环境变量的方式，正确的方法是：

```BASH
set NODE_ENV=production && node app.js
```
但是这样需要维护两个脚本命令，使用[cross-env](https://www.npmjs.com/package/cross-env)就可以解决这个问题

`cross-env`提供了一个设置环境变量的脚本，让我们能够以Linux的方式设置环境变量，在Windows下可以兼容运行

安装：

```BASH
npm install cross-env --save-dev
```

使用时只需要在原来的脚本前面加上`cross-env`就可以了

```BASH
cross-env NODE_ENV=development nodemon ./index.js 
```

## 使用`.env`文件

在Vue-cli3.0和Create-react-app两款脚手架中，都支持使用`.env`文件来引入环境变量

### Vue中

#### Vue-cli3.0

在Vue-cli3.0的项目中，可以直接传递`--mode`参数复写环境变量，例如想要使用`.env.staging`，可以这样：

```BASH
vue-cli-service build --mode staging
```

这个时候`NODE_ENV`被改写为`staging`，同时会加载`.env.staging`文件

在`.env`中定义的变量除了`NODE_ENV`之外，都需要以`VUE_APP_`开头，同样可以在`.env`文件制定`NODE_ENV`，同样可以生效，例如上面的`.env.staging`文件中可以：

```
NODE_ENV=production
```
这样`NODE_ENV`又被重新改写为了`production`

#### Vue-cli2.0

在以前的项目中，是不能直接使用`.env`文件的，一般都是直接使用`process.env.NODE_ENV`来区分环境。除了内置的`development`/`production`/`test`环境变量外，想要增加新的环境有两个做法

一个是在`build`的文件夹中，复制为`build-staging.js`，同时中为`process.env.NODE_ENV`重新赋值，在运行`npm run build:staging`命令时，运行`build-staging`文件

另外一个就是使用上面提到的`corss-env`，在运行npm命令时赋值

### React中

如果是使用Create-react-app脚手架的项目，在未eject之前，是不能覆写`NODE_ENV`的， 也没有提供`--mode`的选项来指定新的`.env`文件，可以使用`cmd-env`来指定`.env`文件，但是这个时候的`NODE_ENV`是不变的。

具体的可以参考《React提高08 Create React App》这篇笔记。


## 参考

- [process对象@JavaScript标准参考教程](http://javascript.ruanyifeng.com/nodejs/process.html#)
- [使用cross-env解决跨平台设置NODE_ENV的问题@segmentfault](https://segmentfault.com/a/1190000005811347)
- [环境变量和模式@Vue CLI](https://cli.vuejs.org/zh/guide/mode-and-env.html#%E6%A8%A1%E5%BC%8F)
