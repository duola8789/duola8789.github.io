---
title: Egg06 参数校验和异常处理
top: false
date: 2019-06-12 16:25:09
updated: 2019-06-12 16:25:11
tags:
- validate
- error
categories: Egg
---

Egg中参数校验和异常处理的实践

<!-- more -->

## 参数校验

### 手动校验

之前的参数都是在Controller的入口处，手动的进行校验：

```JS
async index() {
  const {ctx } = this
  const { query } = ctx.request
  try {
    const { type } = query
    // 缺少参数，没法查
    if (!type) {
      const errMsg = '缺少参数'
      ctx.response.status = this.config.httpCodeHash.badRequest
      ctx.response.body = ctx.helper.makeErrorResponse(errMsg)
      this.logger.error(new Error(errMsg))
      return
    }
    // 响应内容
    const data = await ctx.service.settings.findSettings(type)
    ctx.response.status = this.config.httpCodeHash.ok
    ctx.response.body = data
  } catch (err) {
    ctx.response.body = err.message || '查询规则错误'
    ctx.response.status = this.config.httpCodeHash.serverError
    this.logger.error(err)
  }
}
```

这样很很导致大量的代码冗余，每个Controll都要写这样进行校验，如果失败手动返回错误结果（实际上参数校验失败也应该统一处理，后面的异常处理部分会提到）

### `egg-validate`

实际上使用[egg-validate](https://github.com/eggjs/egg-validate)插件可以大大简化和标准化参数校验的流程。

安装：

```BASH
npm i egg-validate --save
```

需要在`plugin.js`中开启插件：

```JS
// config/plugin.js
exports.validate = {
  enable: true,
  package: 'egg-validate',
};
```

`egg-validate`实际上是由[parameter](https://github.com/node-modules/parameter)这个库封装而来，它可以针对很多类型的参数进行校验，比如`string`、`dateTime`、`number`、`enum`等，具体的使用方法可以参考它的文档。

使用`egg-validate`进行参数校验的正确姿势：

```JS
'use strict'
const Controller = require('egg').Controller

// 创建规则的校验规则
const createRule = {
  type: { type: 'enum', values: [ 'pre', 'single', 'other' ] },
  name: { type: 'string', trim: true },
  packageName: { type: 'string', trim: true },
  content: { type: 'object' },
}

class PrivacyController extends Controller {
  // 新建预设规则
  async create() {
    const { ctx } = this
    const { name, packageName, type, content } = ctx.request.body

    // 参数校验
    ctx.validate(createRule, ctx.request.body)

    // 创建新规则
    const data = await ctx.service.settings.createSetting(name.trim(), packageName.trim(), type.trim(), content)
    
    // 创建成功
    ctx.response.status = this.config.httpCodeHash.created.code
    ctx.response.body = insertSetting
  }
}

module.exports = PrivacyController
```

`ctx.validate`的第一个参数就是校验的规则，第二个参数是被校验的参数，我们的请求方法是POST，所有的参数都在`body`中，所以传入的是`ctx.request.body`

如果参数校验没有通过，将会抛出一个`status`为`422`的异常：

![](http://image.oldzhou.cn/FkBFmprqHzBa_uCoTmIjt38GLCbW)

这个错误我们没有在Controller中捕获，后面会提到是如何处理的。

要注意的是，在校验规则中，某些类型是可以传入自定义的错误提示信息的，比如对`string`的校验，如果使用了`formate`选项，那么传入的`message`就会有效，其他时刻传入`message`无效，无法自定义错误提示信息：

```JS
const indexRule = {
  id: { type: 'string', trim: true, format: /^.{24}$/, message: '非法ID' }, // Mongo生成的ID长度为24位
  packageName: { type: 'string', trim: true },
}
```
查看它的源码，发现它只有显示或者隐式（`type`为`email`等）这种情况下才会提示自定义的提示信息：

```JS
function checkString(rule, value) {
  if (typeof value !== 'string') {
    return this.t('should be a string');
  }

  // if required === false, set allowEmpty to true by default
  if (!rule.hasOwnProperty('allowEmpty') && rule.required === false) {
    rule.allowEmpty = true;
  }

  var allowEmpty = rule.hasOwnProperty('allowEmpty')
    ? rule.allowEmpty
    : rule.empty;

  if (!value) {
    if (allowEmpty) return;
    return this.t('should not be empty');
  }

  if (rule.hasOwnProperty('max') && value.length > rule.max) {
    return this.t('length should smaller than %s', rule.max);
  }
  if (rule.hasOwnProperty('min') && value.length < rule.min) {
    return this.t('length should bigger than %s', rule.min);
  }

  if (rule.format && !rule.format.test(value)) {
    return rule.message || this.t('should match %s', rule.format);
  }
}

function checkEnum(rule, value) {
  if (!Array.isArray(rule.values)) {
    throw new TypeError('check enum need array type values');
  }
  if (rule.values.indexOf(value) === -1) {
    return this.t('should be one of %s', rule.values.join(', '));
  }
}
```

有时间想提一个PR，支持所有的类型校验都支持自定义提示信息，但是现在由于无法完全自定义，所以索性在异常处理的时候不对外暴漏具体的`message`了，只给出统一的参数校验失败的提示：

```JS
{
  "code": -1,
  "message": "Validation Failed"
}
```

## 统一异常处理

一开始我都是在Controller中使用`try...catch`来捕获错误，每个Controller都这样做很烦，虽然编写了一个helper中的生成错误响应的方法，但是到处都要调用也很麻烦。

在Controller和Service中都有可能抛出异常，这也是Egg推荐的编码方式。当发现客户端参数传递错误或者调用后端服务异常时，通过抛出异常的方式来进行中断

常见的终端的情形有：

1. Controller中`this.ctx.validate`进行参数校验，失败抛出异常
2. Service中调用`this.ctx.curl()`进行HTTP请求，可能由于网络问题等原因抛出服务端异常
3. Service中获取到`this.ctx.curl()`的调用失败的结果，也会抛出异常
4. 其他意料之外的错误，也会抛出异常

Egg提供了默认的异常处理，但是可能与系统中统一的接口约定不一致，因此需要自己实现一个统一错误处理的中间件来对错误处理。

在`app/middleware`目录下新建`errorHanlder.js`文件，新建一个中间件：

```JS
// app/middleware/error_handler.js
module.exports = () => {
  return async function errorHandler(ctx, next) {
    try {
      await next()
    } catch (err) {
      // 所有的异常都在 app 上触发一个 error 事件，框架会记录一条错误日志
      ctx.app.emit('error', err, ctx)
      const status = err.status || 500
      const message = err.message || 'Internal Server Error'

      // HTTP Code
      ctx.status = status

      // 生产环境
      const isProd = ctx.app.config.env === 'prod'

      // 错误响应对象
      ctx.body = {
        code: -1,
        message: (status === 500 && isProd) ? 'Internal Server Error' : message,
        // detail: status === 422 ? err.errors : undefined, // 参数校验未通过
      }
    }
  }
}
```

生产环境时500错误的消息错误内容不应该返回给客户端，因为可能包含敏感信息，所以只返回固定的错误信息。

通过这个中间件，可以捕获所有异常，并且按照想要的格式封装了响应，将这个中间件通过配置文件加载进来：

```JS
// config/config.default.js
module.exports = {
  // 加载 errorHandler 中间件
  middleware: [ 'errorHandler' ],
  // 只对 /api 前缀的 url 路径生效
  errorHandler: {
    match: '/api',
  },
};
```

## 中间件的加载

单独拿出来这一节，是因为当时踩了一个坑，按照上面的配置之后，发现所有的请求根本没有经过我们的`errorHandler`中间件。

这是因为Egg支持定义多个环境的配置文件：

```BASH
config
|- config.default.js
|- config.prod.js
|- config.unittest.js
`- config.local.js
```

`config.default.js`是默认的配置文件，所有所有环境都会加载这个配置文件，一般也会作为开发环境的默认配置文件。

当指定`env`时也会同时加载对应的额配置文件，并且覆盖默认配置文件的同名配置，比如`prod`环境会加载`config.prod.js`和`config.default.js`文件，**前者会覆盖后者的同名配置**

配置合并使用了[extend2](https://github.com/eggjs/extend2)模块进行深度拷贝，对数组进行合并时会直接覆盖数组，而不是进行合并

```JS
const a = {
  arr: [ 1, 2 ],
};
const b = {
  arr: [ 3 ],
};

extend(true, a, b);
// => { arr: [ 3 ] }
```

这就是我们的中间件没有生效的原因，我们的目录里面同时配置了`config.local.js`和`config.default.js`，在`config.default.js`虽然配置了中间件，但是在`config.local.js`中的`middleware`对应的属性值是一个空数组

根据上面的合并规则，导致最终的`middleware`是一个空数组，没有加载任何的中间件，所以或者在所有的配置文件的`middleware`的数组中都加上`errorHandler`中间件，或者直接在除了`config.default.js`之外的配置文件中删除`middleware`属性。

## 统一的错误对象

我们现在有了统一的异常处理机制，在Controller或者Service中有时候我们要主动抛出异常，抛出的异常应该是一个Error对象，这样才会带上堆栈信息。

但是有一些与HTTP状态有关的异常，应该统一进行管理，保持整个系统的统一。所以使用了`egg-errors`插件，它内置了统一的异常和错误对象。

安装：

```BASH
npm i egg-errors --save
```

这里主要使用的是`egg-errors`内置的HTTP错误对象，它内置了`400`到`500`的错误对象，它提供了对应的`status`和`headers`属性：

```JS
const { ForbiddenError } = require('egg-errors');
const err = new ForbiddenError('your request is forbidden');
console.log(err.status); // 403
```

也可以使用简写来调用对应的错误：

```JS
const { E403 } = require('egg-errors');
const err = new E403('your request is forbidden');
console.log(err.status); // 403
```

我们在`config`中新建了一个`httpCodeHash.js`配置文件，在这个配置文件中引入了`egg-errors`，根据语义化的HTTP返回值进行了配置：

```JS
// HTTP 响应代码: https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status
const errors = require('egg-errors')

// TODO: httpCodeHash.code
module.exports = {
  continue: { code: 100, message: 'Continue' },
  ok: { code: 200, message: 'OK' },
  created: { code: 201, message: 'Created' },
  noContent: { code: 204, message: 'No Content' },
  movedPermanently: { code: 301, message: 'Moved Permanently' },
  found: { code: 302, message: 'Found' },
  notModified: { code: 304, message: 'Not Modified' },
  badRequest: { code: 400, message: 'Bad Request', error: errors.E400 },
  unauthorized: { code: 401, message: 'Unauthorized', error: errors.E401 },
  forbidden: { code: 403, message: 'Forbidden', error: errors.E403 },
  notFound: { code: 404, message: 'Not Found', error: errors.E404 },
  conflict: { code: 409, message: 'Conflict', error: errors.E409 },
  unprocessable: { code: 422, message: 'Unprocessable Entity', error: errors.E422 },
  serverError: { code: 500, message: 'serverError', error: errors.E500 },
  otherServerError: { code: 502, message: 'Bad Gateway', error: errors.E502 },
  errors,
}
```

使用的时候如果只需要加载对应的信息而不需要抛出错误，那么对应的信息都是统一的：

```JS
// 响应内容
const data = await ctx.service.log.findPrivacyLog({ id, packageName })
ctx.response.status = this.config.httpCodeHash.ok.code
ctx.response.body = data
```

如果需要抛出错误的时候，那么就是用对应的`error`属性，新建一个错误对象，并传入对应的自定义错误提示：

```JS
throw new this.config.httpCodeHash.notFound.error('检测记录不存在')
```

这样保证了抛出的错误对象的语义化且统一。

## 参考

- [开启 validate 插件@Egg](https://eggjs.org/zh-cn/tutorials/restful.html#%E5%BC%80%E5%90%AF-validate-%E6%8F%92%E4%BB%B6)
- [eggjs/egg-validate@github](https://github.com/eggjs/egg-validate)
- [node-modules/parameter@github](https://github.com/node-modules/parameter)
- [Config 配置@Egg](https://eggjs.org/zh-cn/basics/config.html)
- [eggjs/egg-errors@github](https://github.com/eggjs/egg-errors)
