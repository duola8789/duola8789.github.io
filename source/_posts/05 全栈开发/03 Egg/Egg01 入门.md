---
title: Egg01 入门
top: false
date: 2019-04-15 16:50:20
updated: 2019-04-15 16:50:20
tags:
- Egg
- Node
categories: Egg
---

在调研BFF的过程中，看到蚂蚁金服自研的BFF的开发框架BFF Chair是基于Egg.js开发的。BFF Chair没有开源，但是Egg.js已经开源了，如果需要引入BFF，需要基于Egg.js的基础上开发自己的BFF开发框架。

<!-- more -->

> 在调研BFF的过程中，看到蚂蚁金服自研的BFF的开发框架BFF Chair是基于Egg.js开发的。BFF Chair没有开源，但是Egg.js已经开源了，如果需要引入BFF，需要基于Egg.js的基础上开发自己的BFF开发框架。

## 1 Egg.js是什么

Egg是一个为了开发企业级应用的框架，没有集成太多功能，值专注于提供Web开发的核心功能，并且提供了一套灵活可扩展的插件机制，来满足定制需求。

Egg奉行『约定优于配置』，在团队内应当按照**一套统一的约定**进行应用开发，降低沟通成本和学习成本。但约定不等于扩展性差，可以按照团队的约定制定框架。

## 2 Egg与其他Web框架的比较

### 2.1 与Express的差异

Express应用很广泛，简单且扩展性强，适合做个人项目，但框架本身缺少约定，标准的MVC模型会有各种千奇百怪的写法。而Egg按照约定进行开发，团队协作成本低。

### 2.2 与Koa的关系

Koa和Express的设计风格类似，底层都是公用的同一套HTTP基础库，但是二者有一些区别：

（1）异步解决方案

Express的异步编程模型是callback形式的，而Koa使用了`async/await`的形式

（2）Middleware中间件

Koa的中间件模型是洋葱圈模型

![](http://image.oldzhou.cn/68747470733a2f2f7261772e6769746875622e636f6d2f66656e676d6b322f6b6f612d67756964652f6d61737465722f6f6e696f6e2e706e67.png)

当中间件在执行时：

![middleware.gif](http://image.oldzhou.cn/middleware.gif)

所有的请求经过一个中间件的时候都会执行两次，可以很方便的话实现后置处理逻辑。

（3）Context

Express只有Request和Response两个对象，Koa增加了Context对象，作为当次请求的上下文对象（Koa1中为中间件的`this`，Koa2中作为中间件的第一个参数传入），可以将以此请求相关的上下文都挂载到这个对象，同时Context也挂在了Request和Response两个对象，这两个对象提供了大量的便捷方法辅助开发，例如：

```
request.query
request.hostname
response.body
response.status
```
（4）异常处理

通过同步方式编写异步代码带来的另一个非常大的好处就是异常处理非常自然，使用`try/catch`就可以将按照规范编写的代码中的所有错误捕获到。可以很方便的编写一个自定义的错误处理中间件：

```JS
async function onerror(ctx, next) {
  try {
    await next();
  } catch (err) {
    ctx.app.emit('error', err);
    ctx.body = 'server error';
    ctx.status = err.status || 500;
  }
}
```
只需要将这个中间件放在其他中间件之前，就可以捕获它们所有的同步或者异步代码中抛出的异常了。

### 2.3 Egg继承于Koa

Egg选择了Koa作为基础框架，在它的模型基础上，做了一些增强。

#### 2.3.1 扩展

在基于Egg的框架或者应用中，可以通过定义`app/extend/{application, context, request, response}.js`中来扩展Koa中对应的四个对象的原型。通过这个功能，可以快速的增加更多的辅助方法。

例如，在`app/extend/context.js`中写入以下代码：

```JS
// app/extend/context.js
module.exports = {
  get isIOS() {
    const iosReg = /iphone|ipad|ipod/i;
    return iosReg.test(this.get('user-agent'));
  },
};
```
在Controller中，就可以使用刚才定义的便捷属性了：

```JS
// app/controller/home.js
exports.handler = ctx => {
  ctx.body = ctx.isIOS
    ? 'Your operating system is iOS.'
    : 'Your operating system is not iOS.';
};
```

#### 2.3.2 插件

Egg和Koa一样，可以通过中间件来提供各种功能，Egg的插件机制更加强大，让独立领域的功能模块可以更加容易编写。

一个插件可以包含：

- `extend`：扩展基础对象的上下文，提供各种工具类、属性
- `middleware`：增加一个或多个中间件，提供请求的前置、后置处理逻辑
- `config`：配置各个环境下插件自身的默认配置项

## 3 快速入门

可以直接使用脚手架快速生成项目

```BASH
npm init egg --type=simple
# 等同于 npx egg --type=simple

npm i

# 启动项目
npm run dev

# open localhost:7001
```
生成项目后，可以编写Controller和对应的Router，可以在`config`文件夹下添加配置文件，此时目录结构如下：

```BASH
egg-example
├── app
│   ├── controller
│   │   └── home.js
│   └── router.js
├── config
│   └── config.default.js
└── package.json
```
Egg内置了static插件，默认映射`/public/* -> app/public/*`

Egg不强制使用某种模板引擎，只是约定了View插件开发规范，使用某种模板引擎需要在`config`目录下的`plugin.js`中开启：

```JS
// config/plugin.js
exports.nunjucks = {
  enable: true,
  package: 'egg-view-nunjucks'
};
```

在`config/config.default.js`中设置文件关联：

```JS
// config/config.default.js
exports.keys = <此处改为你自己的 Cookie 安全字符串>;
// 添加 view 配置
exports.view = {
  defaultViewEngine: 'nunjucks',
  mapping: {
    '.tpl': 'nunjucks',
  },
};
```
实际应用中，Controller一般不会自己产出数据，也不会包含复杂的逻辑，复杂的过程应抽象为业务逻辑层Service

可以使用中间件，完成一些独立的功能。

Egg提供了强大的配置合并管理功能：

- 支持按环境变量加载不同的配置文件，如`config.local.js`，`config.prod.js`等
- 应用/插件/框架都可以配置自己的配置文件，框架将按顺序合并加载

在项目根目录下，以`test.js`为后缀名，即`{app_root}/test/**/*.`(`test/app/middleware/robot.test.js`)

## 4 渐进式开发

在Egg里面的渐进式开发路径是：

插件（path） →  插件（package） → 框架

### 4.1 Step1 原始代码

当我们编写了一段具有通用型逻辑的代码时，可以抽离成为插件，比如`context.js`中的内容：

```BASH
example-app
├── app
│   ├── extend
│   │   └── context.js
│   └── router.js
├── test
│   └── index.test.js
└── package.json
```

### 4.2 Step2 插件（path）

但是在一开始的时候，功能还没完善，直接独立成为插件，维护比较麻烦，这是可以把代码写成**插件的形式，但不独立出去**（即使用path来引用的插件形式的代码）

这个时候新的目录结构为：

```BASH
example-app
├── app
│   └── router.js
├── config
│   └── plugin.js
├── lib
│   └── plugin
│       └── egg-ua
│           ├── app
│           │   └── extend
│           │       └── context.js
│           └── package.json
├── test
│   └── index.test.js
└── package.json
```

核心代码：

（1）`app/extend/context.js`移动到`lib/plugin/egg-us/app/extend/context.js`

（2）`lib/plugin/egg-ua/package.json`声明插件

```JS
{
  "eggPlugin": {
    "name": "ua"
  }
}
```

（3）在`config/plugin.js`中通过`path`来挂载组件

```JS
// config/plugin.js
const path = require('path');
exports.ua = {
  enable: true,
  path: path.join(__dirname, '../lib/plugin/egg-ua'),
};
```

### 4.3 Step3 独立插件（package）

经过一段时间开发后，该模块的功能成熟，此时可以抽离出来成为独立的插件

首先抽离出一个`egg-ua`插件，具体的方法的需要看[插件文档](https://eggjs.org/zh-cn/advanced/plugin.html)学习。

代码在[这里](https://github.com/eggjs/examples/tree/master/progressive/step3/egg-ua)，目录结构：

```BASH
egg-ua
├── app
│   └── extend
│       └── context.js
├── test
│   ├── fixtures
│   │   └── test-app
│   │       ├── app
│   │       │   └── router.js
│   │       └── package.json
│   └── ua.test.js
└── package.json
```

然后对原有应用改造，代码参见[这里](https://github.com/eggjs/examples/tree/master/progressive/step3/egg-ua)。

- 移除`lib/plugin/egg-ua`目录
- `package.json`中声明对`egg-ua`的依赖
- `config/plugin.js`中修改依赖声明为`package`方式

```JS
// config/plugin.js
exports.ua = {
  enable: true,
  package: 'egg-ua',
};
```
在插件没发布前，可以通过`npm link`的方式进行本地测试，具体参见[npm-link](https://docs.npmjs.com/cli/link)

### 4.4 Step4 框架

当积累了插件和配置后我们会发现，团队的大部分项目都会用到这些插件。此时就可以考虑抽象出一个适合团队业务场景的框架。

首先抽象出`example-framework`框架，具体的方法还是得看[文档](https://eggjs.org/zh-cn/advanced/framework.html)学习

[代码在这里](https://github.com/eggjs/examples/tree/master/progressive/step4/example-framework)，目录结构：

```BASH
example-framework
├── config
│   ├── config.default.js
│   └── plugin.js
├── lib
│   ├── agent.js
│   └── application.js
├── test
│   ├── fixtures
│   │   └── test-app
│   └── framework.test.js
├── README.md
├── index.js
└── package.json
```

把原来的`egg-ua`等插件的依赖从原来的应用中移除，配置到该框架的`package.json`和`config/plugin.js`中，然后改造原有的应用，对应的代码参考[这里](https://github.com/eggjs/examples/tree/master/progressive/step4/example-app)

- 移除`config/plugin.js`中对`egg-ua`的依赖
- `package.json`中移除对`egg-ua`的依赖
- `package.json`中声明对`example-framework`的以阿里，并配置`egg.framework`

```JS
{
  "name": "progressive",
  "version": "1.0.0",
  "private": true,
  "egg": {
    "framework": "example-framework"
  },
  "dependencies": {
    "example-framework": "*"
  }
}
```
同样，在插件没发布前，可以通过`npm link`的方式进行本地测试

### 4.5 渐进式开发总结

总的来说，Egg.js还是和适合一步步的渐进地去进行框架演进，具体流程如下：

1. 当应用中有可能会复用到的通用逻辑，抽离出来放到`lib/plugin`中
2. 当插件功能稳定后，独立出来作为一个`node moudle`
3. 如此以往，应用中相对复用性较强的代码都会逐渐独立为单独的插件
4. 当应用逐渐进化到针对某类业务场景的解决方案时，将其抽象为独立的framework进行发布
5. 在新项目中抽象出的插件，下沉集成到框架后，其他项目只需要简单的重新`npm install`后就可以使用，可以提高团队效率

## 5 完整目录结构

<pre>egg-project
├── package.json
├── app.js (可选) ------------------------- # 用于自定义自动时的初始化工作
├── agent.js (可选) ----------------------- # 用于 Agent 机制的配置
├── app
|   ├── router.js  ------------------------ # 用于配置 URL 路由规则
│   ├── controller ------------------------ # 用于解析用户的输入
│   |   └── home.js
│   ├── service (可选) --------------------- # 用于编写业务逻辑层，建议使用
│   |   └── user.js
│   ├── middleware (可选) ------------------ # 用于编写中间件
│   |   └── response_time.js
│   ├── schedule (可选) -------------------- # 用于定时任务
│   |   └── my_task.js
│   ├── public (可选) ---------------------- # 用于放置静态资源
│   |   └── reset.css
│   ├── view (可选) ------------------------ # 用于放模板文件
│   |   └── home.tpl
│   ├── Modal (可选) ----------------------- # 用于放置领域模型，可选，由领域类相关插件约定
│   |   └── mySQL.db
│   └── extend (可选) ---------------------- # 用于框架的扩展
│       ├── helper.js (可选)
│       ├── request.js (可选)
│       ├── response.js (可选)
│       ├── context.js (可选)
│       ├── application.js (可选)
│       └── agent.js (可选)
├── config
|   ├── plugin.js -------------------------- # 用于配置需要加载的插件
|   ├── config.default.js
│   ├── config.prod.js --------------------- # 配置文件，根据不同环境加载
|   ├── config.test.js (可选)
|   ├── config.local.js (可选)
|   └── config.unittest.js (可选)
└── test ----------------------------------- # 用于单元测屙屎
    ├── middleware
    |   └── response_time.test.js
    └── controller
        └── home.test.js</pre>

## 6 静态资源

`egg-view-assets`提供了通用的静态资源管理和本地开发，[文档在这里](https://github.com/eggjs/egg-view-assets)。

可以配合基于Webpack封装的[roadhog](https://github.com/sorrycc/roadhog)、[umi](https://umijs.org/zh/guide/)，通过自动的方式添加静态资源。

重要的是和构建工具整合，保证本地开发体验及自动部署，所以构建工具和框架有一层约定。

### 6.1 映射关系

构建工具的Entry配置决定了映射关系，基于Webpack封装的roadhog、umi内置了映射关系，如果单独使用Webpack需要根据这层映射来选择使用哪种方式：

- 文件源码`app/assets/index.js`，对应的Entry为`index.js`
- 本地静态服务接受以此为Entry，如请求`http://127.0.0.1:8000/index.js`
- 构建生成的文件需要有这层映射关系，如生成`index.{hash}.js`并生成Mainfest文件描述关系如：

```
{
  "index.js": "index.{hash}.js"
}
```
roadhog完全满足这个映射关系，所以可以直接使用assets模板引擎，Umi不满足文件映射，所以选用其他模板引擎的方案。

### 6.2 开发、部署、CDN

[文档](https://eggjs.org/zh-cn/tutorials/assets.html)的介绍多是基于roadhog的基础上进行的配置，如果使用Webpack需要自己配置，还是比较繁琐的。

所以考虑使用Easy-Team提供的EGG + Vue工程化解决方案。

## 参考

- [Egg文档](https://eggjs.org/zh-cn/intro/)

