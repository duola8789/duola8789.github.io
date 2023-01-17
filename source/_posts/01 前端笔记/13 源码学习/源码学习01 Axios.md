---
title: 源码学习01 Axios
top: false
date: 2019-07-31 17:42:00
updated: 2019-07-31 17:42:00
tags: 
- 源码
- Axios
categories: 源码学习
---

[Axios](https://github.com/axios/axios)是一个基于Promise的HTTP请求库，可以用在浏览器和Node.js中。平时在Vue项目中，经常使用它来实现HTTP请求。

它的使用简便、灵活，并且有interceptors、数据转换器等强大的功能，以前用的时候并没有仔细研究过这些功能是如何实现的，正好在知乎的大前端专栏看到[一篇文章](https://zhuanlan.zhihu.com/p/58349237)对Axios的源码进行了解读。借着这篇文章的帮助，我开始了自己阅读源码的道路。

以后要多多的读源码，更多的独立完成，向大神们学习。

<!-- more -->

## Axios的目录结构

Axios的目录结构相对还是比较简单的

![](http://image.oldzhou.cn/FpZywJvuNKZw9iz1vGYhUWtrhQ1V)

目录里面`adapters/`目录下定义的是如何发出一个HTTP请求，这也就是为什么Axios技能应用在浏览器中（XHR）又能用在Node.js中（`http.request`)，`core/Axios.js`是Axios的核心主类，`axios.js`是整个Axios的入口。

## Axios的实现流程

```
graph TB
引入axios-->Axios构造函数实例化
Axios构造函数实例化-->Interceptors请求拦截器
Interceptors请求拦截器-->dispatchRequest方法
dispatchRequest方法-->请求转换器transformRequest
请求转换器transformRequest-->http请求适配器adapter
http请求适配器adapter-->响应转换器transformResponse
响应转换器transformResponse-->Interceptors响应拦截器
```

## 工具函数的学习

### `forEach`

这个`forEach`与原生的数组的`forEach`并不相同，它可以遍历对象，也可以遍历数组，还可以遍历基本值：

```JS
function forEach(obj, fn) {
  // 如果是空值就返回
  if (obj === null || typeof obj === 'undefined') {
    return;
  }

  // 如果是基本类型，则放到数组里面进行遍历
  if (typeof obj !== 'object') {
    /*eslint no-param-reassign:0*/
    obj = [obj];
  }

  if (isArray(obj)) {
    // 遍历数组
    for (var i = 0, l = obj.length; i < l; i++) {
      fn.call(null, obj[i], i, obj);
    }
  } else {
    // 遍历对象
    for (var key in obj) {
      if (Object.prototype.hasOwnProperty.call(obj, key)) {
        fn.call(null, obj[key], key, obj);
      }
    }
  }
}
```

### `merge`和`deepMerge`

用来合并对象，二者的区别只是对于嵌套的深层的对象，`deepMerge`也会进行深层的拷贝，而不是指针的改变

```
function merge(/* obj1, obj2, obj3, ... */) {
  var result = {};
  function assignValue(val, key) {
    if (typeof result[key] === 'object' && typeof val === 'object') {
      result[key] = merge(result[key], val);
    } else {
      result[key] = val;
    }
  }

  for (var i = 0, l = arguments.length; i < l; i++) {
    forEach(arguments[i], assignValue);
  }
  return result;
}

function deepMerge(/* obj1, obj2, obj3, ... */) {
  var result = {};
  function assignValue(val, key) {
    if (typeof result[key] === 'object' && typeof val === 'object') {
      result[key] = deepMerge(result[key], val);
    } else if (typeof val === 'object') {
      result[key] = deepMerge({}, val);
    } else {
      result[key] = val;
    }
  }

  for (var i = 0, l = arguments.length; i < l; i++) {
    forEach(arguments[i], assignValue);
  }
  return result;
}
```

### `isStandardBrowserEnv`

用来判断是否是标准的浏览器环境，对于Web Workers，

```
typeof window -> undefined
typeof document -> undefined
```

对于RN和NativeScript

```
react-native:
navigator.product -> 'ReactNative'

nativescript
navigator.product -> 'NativeScript' or 'NS'
```
所以有：


```JS
/**
 * Determine if we're running in a standard browser environment
 *
 * This allows axios to run in a web worker, and react-native.
 * Both environments support XMLHttpRequest, but not fully standard globals.
 *
 * web workers:
 *  typeof window -> undefined
 *  typeof document -> undefined
 *
 * react-native:
 *  navigator.product -> 'ReactNative'
 * nativescript
 *  navigator.product -> 'NativeScript' or 'NS'
 */
function isStandardBrowserEnv() {
  if (typeof navigator !== 'undefined' && (navigator.product === 'ReactNative' ||
                                           navigator.product === 'NativeScript' ||
                                           navigator.product === 'NS')) {
    return false;
  }
  return (
    typeof window !== 'undefined' &&
    typeof document !== 'undefined'
  );
}
```


## Axios的多种使用方式

Axios有多种使用方式：

```JS
import axios from 'axios';

// 第一种 axios(option)
axios({ url, method, headers });

// 第二种 axios(url[, option]);
axios(url, { method, headers })

// 第三种（对于 get/delete 等方法） axios.[method](url[, option])
axios.get(url, { headers })

// 第四种（对于 post/put等方法）axios[.method](url[, data[, option]])
axios.post(url, data, { headers })

// 第五种 axios.request(option)
axios.request({ url, method, headers })
```

下面从入口文件`axios.js`来分析这些使用方式都是如何实现的

```JS
// axios.js

// 用来创建一个 axios 的实例
function createInstance(defaultConfig) {
  // 通过默认配置新建一个 axios 实例
  var context = new Axios(defaultConfig);
  
  // 通过 bind 方法获取到 instance，并且绑定 this 上下文
  // instance 是一个函数，实际上就是 Axios.prototype.request.bind(context)，其上下文指向 context
  // 所以可以通过 instance(options)的方法调用
  var instance = bind(Axios.prototype.request, context);

  // 将 Axios 原型上的属性和方法复制到 instance 上，作为静态属性和静态方法
  // Axios.prototype 上定义了 get/delete/post 等方法，所以可以直接使用 instance.get 这种形式调用
  utils.extend(instance, Axios.prototype, context);

  // 将 context 实例的属性和方法复制到 instance 上，作为静态属性和静态方法
  // context.request 指向 Axios.prototype.request，所以可以通过instance.request 这种形式调用
  utils.extend(instance, context);

  // 返回 request 方法，它与 context 的差别仅仅在于它本身是一个函数，可以直接调用
  return instance;
}

// 接受默认配置项作为参数，创建一个 request 方法，具有 Axios 的各种实例属性、方法以及原型属性和方法
// 可以认为导出的就是 Axios 的实例
var axios = createInstance(defaults);

// 暴露 Axios 类，用于继承
axios.Axios = Axios;

// 实例上定义的 create 方法，可以创建新的 axios 实例
axios.create = function create(instanceConfig) {
  return createInstance(mergeConfig(axios.defaults, instanceConfig));
};

// 暴露出 Cancel 相关的类
axios.Cancel = require('./cancel/Cancel');
axios.CancelToken = require('./cancel/CancelToken');
axios.isCancel = require('./cancel/isCancel');

// 暴露出 all 方法，实际上利用的就是 Promise.all方法
axios.all = function all(promises) {
  return Promise.all(promises);
};

// 导出一个 apply 的语法糖，用来将多参数作为输入传入函数，功能和 rest 参数相同
axios.spread = require('./helpers/spread');

// 导出实例
module.exports = axios;

// Allow use of default import syntax in TypeScript
module.exports.default = axios;
```

有个疑问，为什么`createInstance`函数需要绕那么大一个弯，而不是直接导出`new Axios`的实例呢？我的理解是如果直接导出`new Axios`是没有办法使用`axios(option)`和`axios(url, option)`这两种方式来实现调用。

`createInstance`最终返回的是一个函数，它指向`Axios.prototype.request`，并且绑定了`new Axios`实例作为上下文对象，同时这个导出的函数还有`Axios.prototype`以及`new Axios`实例的各个方法和属性作为其静态属性和方法，这些方法的上下文都指向`new Axios`这同一个对象。


上面的代码解释了除了第二种Axios的调用方法之外的调用方法，第二种调用方法是在`Axios.prototype.request`中对第一个参数的数据类型进行判断来实现的，后面在学习`Axios.prototype.request`代码时会提到。

## `Axios`类

`Axios.js`是Axios的核心，它声明了`Axios`这个类，并在原型添加了一些方法，其中最核心的就是`request`方法，上面提到的各种调用方法都是通过调用`request`方法实现的。

首先来看`Axios`类的声明

```JS
function Axios(instanceConfig) {
  // 将传入的配置保存到实例属性上
  this.defaults = instanceConfig;
  
  // 声明 interceptors 属性，用来保存请求拦截器和响应拦截器
  this.interceptors = {
    request: new InterceptorManager(),
    response: new InterceptorManager()
  };
}
```

`Axios`类只声明了两个实例属性，拦截器属性都是`InterceptorManager`的实例。`InterceptorManager`这个类位于`core/InterceptorManager.js`，并不复杂，定义了一个实例属性来存放拦截器，定义了一些原型方法来对队列中拦截器进行添加、移除和遍历的操作

```JS
// 定义实例属性 handlers 来存放拦截器
function InterceptorManager() {
  this.handlers = [];
}

// 添加拦截器到队列中，接受两个参数，分别对应 Promise的 resolve 和 reject
// 返回拦截器的 ID， 用来移除它
InterceptorManager.prototype.use = function use(fulfilled, rejected) {
  this.handlers.push({
    fulfilled: fulfilled,
    rejected: rejected
  });
  return this.handlers.length - 1;
};

// 在队列中移除拦截器
InterceptorManager.prototype.eject = function eject(id) {
  if (this.handlers[id]) {
    this.handlers[id] = null;
  }
};

// 用来遍历执行队列中的拦截器
InterceptorManager.prototype.forEach = function forEach(fn) {
  utils.forEach(this.handlers, function forEachHandler(h) {
    if (h !== null) {
      fn(h);
    }
  });
};
```

实际上Axios的实例属性`interceptors.request`用来存放请求拦截器，`interceptors.response`用来存放响应拦截器，

然后来看核心的`request`方法的代码：

```JS
Axios.prototype.request = function request(config) {
  // 通过参数的类型判断实现 axios(url[, option]) 的调用方式
  if (typeof config === 'string') {
    config = arguments[1] || {};
    config.url = arguments[0];
  } else {
    config = config || {};
  }

  // 合并配置项
  config = mergeConfig(this.defaults, config);
  config.method = config.method ? config.method.toLowerCase() : 'get';

  // 添加拦截器
  var chain = [dispatchRequest, undefined];
  var promise = Promise.resolve(config);

  // 将请求拦截器添加到 chain 前面
  this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
    chain.unshift(interceptor.fulfilled, interceptor.rejected);
  });

  // 将响应拦截器添加到 chain 后面
  this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
    chain.push(interceptor.fulfilled, interceptor.rejected);
  });

  // 对 chain 进行循环
  while (chain.length) {
    promise = promise.then(chain.shift(), chain.shift());
  }

  return promise;
};
```

个人感觉上面的代码十分巧妙，首先生命了`chain`这个数组，填了两个成员`dispatchRequest`和`undefined`，然后定义了一个立刻`resolve`的Promise对象`promise`，它返回的是`config`对象。

我们在添加请求拦截器时：

```JS
// 添加请求拦截器
const myRequestInterceptor = axios.interceptors.request.use(config => {
  // 在发送http请求之前做些什么
  return config; // 有且必须有一个config对象被返回
}, error => {
    // 对请求错误做些什么
    return Promise.reject(error);
});
```
使用了`use`方法，将请求拦截器添加到了`this.interceptors.request.handlers`对列中，然后通过`forEach`方法，对这个队列进行遍历，要注意请求拦截器使用的是`unshift`方法，添加到`dispatchRequest`前面，而响应拦截器使用`push`方法添加到`undefined`后面，所以对于请求拦截器来说，先添加的拦截器会后执行，对于响应拦截器来说，先添加的拦截器会先执行：

```JS
axios.interceptors.request.use(fn1, fn1_1);
axios.interceptors.request.use(fn2, fn2_1);
axios.interceptors.response.use(fn3, fn3_1);
axios.interceptors.response.use(fn4, fn4_1);
```

按照上面的顺序添加的拦截器，存储到`chain`数组中是这样的：

```JS
[fn2, fn2_1, fn1, fn1_1, dispatchRequest, undefined, fn3, fn3_1, fn4, fn4_1]
```

`InterceptorManager.prototype.use`方法接受两个参数分别是Promise成功和失败的回调函数，如果之传入一个函数，那么默认失败的情况对应的就是`undefined`。所以`chain`数组中是**两个成员为一组的**，分别对应一次Promise状态改变的两个回调函数。

然后对`chain`进行循环：

```JS
while (chain.length) {
  promise = promise.then(chain.shift(), chain.shift());
 }
```

在循环过程中对`promise`重新复制，`then`方法中的两个参数分别是`chain.shift()`，`chain.shift()`的作用有二：

1. 减小`chain`的长度
2. 将剪切得到的两个`chain`成员作为`then`方法的两个参数执行。

Axios规定，在使用拦截器时，请求拦截器必须返回`config`对象，响应拦截器必须返回`response`对象，这样才能实现`promise`的链式调用

在请求拦截器中进行链式调用时，将`config`对象作为Promise的结果进行传递，使得所有请求拦截器共享`config`对象，直到真正发出请求的`dispatchRequest`接收到`config`对象并发出请求后将接收到的`response`作为结果返回给后续的响应拦截器，并继续传递。

`chain`数组中的`undefined`是作为`dispatchRequest`一组的`then`方法的第二个回调函数，它的作用是兜住最后一个响应拦截器的错误对象，不会破坏`chain`两个回调函数一组的匹配顺序。

`Axios.js`中还有一些其他的代码，主要的作用是为`Axios.prototype`添加了`get`、`post`等方法，实际上都是调用`Axios.prototype.request`方法

```JS
utils.forEach(['delete', 'get', 'head', 'options'], function forEachMethodNoData(method) {
  // 添加到原型对象
  Axios.prototype[method] = function(url, config) {
    // 调用 Axios.prototype.request 方法
    return this.request(utils.merge(config || {}, {
      method: method,
      url: url
    }));
  };
});

utils.forEach(['post', 'put', 'patch'], function forEachMethodWithData(method) {
  // 添加到原型对象
  Axios.prototype[method] = function(url, data, config) {
   // 调用 Axios.prototype.request 方法
    return this.request(utils.merge(config || {}, {
      method: method,
      url: url,
      data: data
    }));
  };
});

```


## `dispathReqeust`

上面提到的，真正发出HTTP请求的是`dispathReqeust`方法，`dispathReqeust`主要完成了三件事：

1. 拿到`config`对象，进行处理、合并，传递给HTTP请求适配器
2. HTTP请求适配器根据`config`对象发起HTTP请求
3. 请求完成后，根据数据转换器对得到的数据进行二次处理，返回`response`

```JS
// 如果取消了请求，则抛出原因
function throwIfCancellationRequested(config) {
  if (config.cancelToken) {
    config.cancelToken.throwIfRequested();
  }
}

module.exports = function dispatchRequest(config) {
  // 如果执行了 cancelToken 的回调函数则终端流程， throw 出回调函数传入的 reason
  // 是否执行 cancelToken 的回调函数就是通过是否有 reason 来判断的
  throwIfCancellationRequested(config);

  // 如果传入的不是绝对地址，并且配置了 baseUrl，则将 baseUrl 与 config.url 组合
  if (config.baseURL && !isAbsoluteURL(config.url)) {
    config.url = combineURLs(config.baseURL, config.url);
  }

  // 确保 config.headers 是一个对象
  config.headers = config.headers || {};

  // 如果定义了 config.transformRequest，则据此对 data 和 headers 进行处理
  config.data = transformData(
    config.data,
    config.headers,
    config.transformRequest
  );

  // 处理 headers
  config.headers = utils.merge(
    config.headers.common || {},
    config.headers[config.method] || {},
    config.headers || {}
  );

  // 删除各种请求方法下的特定的header
  // 因为上面都已经合并到 config.headers 中，所以没用了
  utils.forEach(
    ['delete', 'get', 'head', 'post', 'put', 'patch', 'common'],
    function cleanHeaderConfig(method) {
      delete config.headers[method];
    }
  );

  // 指定 HTTP 请求适配器，一般来说默认的适配器就可以满足需要
  // 默认的适配器会根据环境自动选择 XHR 或者 Node 的 http.request 方法发送网络请求
  // 手动指定适配器需要返回一个 Promise 对象，一般可以用来拦截请求，返回 mock 数据
  var adapter = config.adapter || defaults.adapter;

  return adapter(config).then(function onAdapterResolution(response) {
    // 请求成功
    // 再次判断是否取消
    throwIfCancellationRequested(config);

    // 根据 config.transformResponse 对响应数据和响应头进行处理
    response.data = transformData(
      response.data,
      response.headers,
      config.transformResponse
    );

    return response;
  }, function onAdapterRejection(reason) {
    // 请求失败
    // 判断是否是手动取消导致的失败
    if (!isCancel(reason)) {
      // 如果早晚要取消，不妨再次抛出取消的原因
      throwIfCancellationRequested(config);

      // 根据 config.transformResponse 对响应数据和响应头进行处理
      if (reason && reason.response) {
        reason.response.data = transformData(
          reason.response.data,
          reason.response.headers,
          config.transformResponse
        );
      }
    }
    
    // 返回一个 reject 的 Promise
    // 没有直接 return reason，是因为只有返回 reject 的 Promise，才能走到响应拦截器的第二个参数（处理异常的函数）
    // 否则就会走到第一个参数（处理正常的函数）
    return Promise.reject(reason);
  });
};
```

`dispathReqeust`方法返回一个Promise，携带着成功求情后的响应数据，或者是失败后的错误对象。用户就可以在调用`axios()`方法后的`then`或者`catch`中进行业务处理了。

## Adapter

上面代码的注释里面提到了，在`dispatchRequest`中通过`config.adapter`或者`defaults.adapter`指定HTTP请求适配器，一般来说默认的适配器就可以满足需要，默认的适配器会根据环境自动选择XHR或者Node的`http.request`方法发送网络请求

在`defaults.js`中的`adatper`方法完成的就是根据环境选择HTTP适配器

```JS
function getDefaultAdapter() {
  var adapter;
  // 通过判断 process 是否存在以及类型（toString）来判断是否是 Node 环境
  if (typeof process !== 'undefined' && Object.prototype.toString.call(process) === '[object process]') {
    // Node 环境使用 http 适配器
    adapter = require('./adapters/http');
  } else if (typeof XMLHttpRequest !== 'undefined') {
    // 浏览器环境使用 XHR 
    adapter = require('./adapters/xhr');
  }
  return adapter;
}
```

Axios是基于Promise的，而HTTP请求的实现又是通过传统的Ajax实现的，所以`adapter/xhr.js`的主要功能就是面试时经常遇到的一道题，将Ajax改为Promise的形式。来学习一下Axios是如何实现的。

```JS
// xhr.js

module.exports = function xhrAdapter(config) {
  // 返回一个 Promise 对象
  return new Promise(function dispatchXhrRequest(resolve, reject) {
    var requestData = config.data;
    var requestHeaders = config.headers;
    
    // 如果 data 是 FromData ，则删除 Content-Type 的 Header
    if (utils.isFormData(requestData)) {
      delete requestHeaders['Content-Type']; // Let the browser set it
    }
    
    // 新建 xhr 对象
    var request = new XMLHttpRequest();

    // HTTP 认证信息
    if (config.auth) {
      var username = config.auth.username || '';
      var password = config.auth.password || '';
      requestHeaders.Authorization = 'Basic ' + btoa(username + ':' + password);
    }
    
    // 新建一个 AJAX 连接，注意 HTTP 方法都应该是大写的
    // 请求的 url 是通过 buildURL 方法创建的，它会将查询参数进行序列化并安全编码，附在 url 后
    request.open(config.method.toUpperCase(), buildURL(config.url, config.params, config.paramsSerializer), true);

    // 设定超时时间，单位是毫秒
    request.timeout = config.timeout;

    // 监听 ready state 的改变
    request.onreadystatechange = function handleLoad() {
      if (!request || request.readyState !== 4) {
        return;
      }

      // 如果请求出错，并且没有收到响应，这种情况会被 onerror 时间捕获到
      // 只有一种例外情况：请求使用的是 file 协议，大多数浏览器会返回的 status 为 0
      if (request.status === 0 && !(request.responseURL && request.responseURL.indexOf('file:') === 0)) {
        return;
      }

      // getAllResponseHeaders 方法可以返回所有的响应头
      var responseHeaders = 'getAllResponseHeaders' in request ? parseHeaders(request.getAllResponseHeaders()) : null;
      //  提取响应结果
      var responseData = !config.responseType || config.responseType === 'text' ? request.responseText : request.response;
      var response = {
        data: responseData,
        status: request.status,
        statusText: request.statusText,
        headers: responseHeaders,
        config: config, // 将 config 对象传递给 response 对象
        request: request
      };
      
      // 根据 response.status 来 resolve 或者 reject 当前的 Promise
      settle(resolve, reject, response);

      // 重置 request
      request = null;
    };

    // 处理浏览器的终止事件，返回 reject 的 Promise
    // 当一个请求被终止时，request 的 readyState 属性被置为 0
    request.onabort = function handleAbort() {
      if (!request) {
        return;
      }
      
      // 返回 reject 的 Promise，内容是定制的 Error 对象
      reject(createError('Request aborted', config, 'ECONNABORTED', request));

      request = null;
    };

    // 处理网络错误导致的资源加载失败
    request.onerror = function handleError() {
      // 真正的错误被浏览器拦截了，只有当网络错误时才会触发 onerror 事件
      reject(createError('Network Error', config, null, request));

      // Clean up request
      request = null;
    };

    // 处理超时的情况
    request.ontimeout = function handleTimeout() {
      reject(createError('timeout of ' + config.timeout + 'ms exceeded', config, 'ECONNABORTED',
        request));

      // Clean up request
      request = null;
    };

    // 添加 xsrf 的请求头，这只适用于在浏览器环境，不适用于 Web Worker 和 RN 无效
    if (utils.isStandardBrowserEnv()) {
      var cookies = require('./../helpers/cookies');

      var xsrfValue = (config.withCredentials || isURLSameOrigin(config.url)) && config.xsrfCookieName ?
        cookies.read(config.xsrfCookieName) :
        undefined;

      if (xsrfValue) {
        requestHeaders[config.xsrfHeaderName] = xsrfValue;
      }
    }

    // 使用 setRequestHeader 方法来添加请求头
    // 此方法必须在 open() 和 send() 之间调用。
    // 如果多次对同一个请求头赋值，只会生成一个合并了多个值的请求头。
    if ('setRequestHeader' in request) {
      utils.forEach(requestHeaders, function setRequestHeader(val, key) {
        if (typeof requestData === 'undefined' && key.toLowerCase() === 'content-type') {
          // 如果没有 data  属性，移除 Content-Type 属性
          delete requestHeaders[key];
        } else {
          request.setRequestHeader(key, val);
        }
      });
    }

    // 添加 withCredentials 属性
    if (config.withCredentials) {
      request.withCredentials = true;
    }

    // 添加 request.responseType 
    // 需要确保服务器所返回的类型和所设置的返回值类型是兼容的。
    // 如果两者类型不兼容，服务器返回的数据变成了null，即使服务器返回了数据。
    if (config.responseType) {
      try {
        request.responseType = config.responseType;
      } catch (e) {
        if (config.responseType !== 'json') {
          throw e;
        }
      }
    }

    // Handle progress if needed
    if (typeof config.onDownloadProgress === 'function') {
      request.addEventListener('progress', config.onDownloadProgress);
    }

    // Not all browsers support upload events
    if (typeof config.onUploadProgress === 'function' && request.upload) {
      request.upload.addEventListener('progress', config.onUploadProgress);
    }
    
    // 处理 cancelToken，后面会单独介绍
    if (config.cancelToken) {
      // 也是一个 Promise 对象
      config.cancelToken.promise.then(function onCanceled(cancel) {
        // 如果 cancel 时 request 已经完成，那就返回
        if (!request) {
          return;
        }
        
        // 使用了 abort 事件
        request.abort();
        reject(cancel);
        
        request = null;
      });
    }
    
    // 根据标准，如果是 GET 或者 HEAD 请求，应将请求主体设置为 null
    if (requestData === undefined) {
      requestData = null;
    }

    // 发送请求
    request.send(requestData);
  });
};
```

`xhrAdapter`的XHR发送请求成功后会执行Promise对象的`resolve`方法，将请求的数据传递出去，如果请求失败（超时、网络出错、终止）则执行`reject`方法，并将自定义的错误信息传递出去。

## Settle

`xhrAdapter`中将改变Promise状态的功能抽离成为单独的`settle`方法

```JS
// settle.js

module.exports = function settle(resolve, reject, response) {
  // 获得配置的 validateStatus 方法
  var validateStatus = response.config.validateStatus;
  
  // 如果不存在 validateStatus 或者验证通过，resolve
  if (!validateStatus || validateStatus(response.status)) {
    resolve(response);
  } else {
    reject(createError(
      'Request failed with status code ' + response.status,
      response.config,
      null,
      response.request,
      response
    ));
  }
};
```

`validateStatus`接受`response.stastus`作为参数，对于给定的HTTP状态码确定其成功失败状态，比如：

```JS
validateStatus: function (status) {
  return status >= 200 && status < 300; // 默认的
}
```

## 数据转换器

前面也提到了数据转换器，可以对请求对象和响应和数据进行转换，可以全局使用：


```JS
// 往现有的请求转换器里增加转换方法
axios.defaults.transformRequest.push((data, headers) => {
  // ...处理data
  return data;
});

// 往现有的响应转换器里增加转换方法
axios.defaults.transformResponse.push((data, headers) => {
  // ...处理data
  return data;
});
```

也可以在单次请求中使用：


```JS
// 往已经存在的转换器里增加转换方法
axios.get(url, {
  // ...
  transformRequest: [
    ...axios.defaults.transformRequest, // 去掉这行代码就等于重写请求转换器了
    (data, headers) => {
      // ...处理data
      return data;
    }
  ],
  transformResponse: [
    ...axios.defaults.transformResponse, // 去掉这行代码就等于重写响应转换器了
    (data, headers) => {
      // ...处理data
      return data;
    }
  ],
})
```

在`defaults`配置项中已经默认自定义了一个请求转换器和响应转换器


```JS
// 请求转换器
transformRequest: [
  function transformRequest(data, headers) {
    normalizeHeaderName(headers, 'Accept');
    normalizeHeaderName(headers, 'Content-Type');
    if (utils.isFormData(data) || utils.isArrayBuffer(data) || utils.isBuffer(data) || utils.isStream(data) || utils.isFile(data) || utils.isBlob(data)) {
      return data;
    }
    if (utils.isArrayBufferView(data)) {
      return data.buffer;
    }
    if (utils.isURLSearchParams(data)) {
      setContentTypeIfUnset(headers, 'application/x-www-form-urlencoded;charset=utf-8');
      return data.toString();
    }
    if (utils.isObject(data)) {
      setContentTypeIfUnset(headers, 'application/json;charset=utf-8');
      return JSON.stringify(data);
    }
    return data;
  }
],

// 响应转换器
transformResponse: [
  function transformResponse(data) {
    /*eslint no-param-reassign:0*/
    if (typeof data === 'string') {
      try {
        data = JSON.parse(data);
      } catch (e) { /* Ignore */ }
    }
    return data;
  }
],
```

默认的请求转换器对请求数据和请求头进行标准化处理，默认的响应转换器用来自动将字符串解析为JSON对象。

使用的时候是通过`transformData`这个方法，对数组中的转换器进行遍历：

```JS
// transformData.js
module.exports = function transformData(data, headers, fns) {
  /*eslint no-param-reassign:0*/
  utils.forEach(fns, function transform(fn) {
    data = fn(data, headers);
  });

  return data;
};
```

转换器和拦截器都可以对请求和响应的数据进行拦截处理，但是一般情况下，拦截器主要负责拦截修改`config`配置项，数据转换器主要用来负责拦截转换请求主体和响应数据。


## Cancel

Axios提供了取消请求的功能，有两种使用方法：


```JS
// 第一种取消方法
axios.get(url, {
  cancelToken: new axios.CancelToken(cancel => {
    if (/* 取消条件 */) {
      cancel('取消日志');
    }
  })
});

// 第二种取消方法
const CancelToken = axios.CancelToken;
const source = CancelToken.source();
axios.get(url, {
  cancelToken: source.token
});
source.cancel('取消日志');
```

Axios用了三个模块来实现这个功能，首先是`Cancel`这个类：


```JS
// ./cancel/Cancel.js
function Cancel(message) {
  this.message = message;
}

Cancel.prototype.toString = function toString() {
  return 'Cancel' + (this.message ? ': ' + this.message : '');
};

Cancel.prototype.__CANCEL__ = true;

module.exports = Cancel;
```
主要是定义了`Cancel`的`message`实例属性，和原型上的内部用的`__CANCEL__`属性，还定义了一个`toString`方法

`isCancel`返回布尔值，根据是否传入了`value`以及是否有`__CANCEL__`属性，判断是否是Cancel实例

核心的代码在`CancelToken.js`中：


```JS
// ./cancel/CancelToken.js

function CancelToken(executor) {
  // executor 必须是函数
  if (typeof executor !== 'function') {
    throw new TypeError('executor must be a function.');
  }

  // 通过闭包，将 this.promise 的控制权导出到 resolvePromise 变量上，此时 promise 状态为 pending
  var resolvePromise;
  this.promise = new Promise(function promiseExecutor(resolve) {
    resolvePromise = resolve;
  });

  // 传入一个函数作为 executor 的参数，将 promise 状态的控制权导出到 executor 函数中
  // 当 executor 的参数执行时，为 this.reason 赋值，并且改变 this.promise 状态为 resolve
  var token = this;
  executor(function cancel(message) {
    if (token.reason) {
      // Cancellation has already been requested
      return;
    }

    token.reason = new Cancel(message);
    resolvePromise(token.reason);
  });
}

// 由于 this.reason 是在 exxcutor 的方法执行后才赋值的，所以据此判断是否已经执行了取消操作
// 如果已经执行取消操作，则抛出 this.reason
CancelToken.prototype.throwIfRequested = function throwIfRequested() {
  if (this.reason) {
    throw this.reason;
  }
};

// 对应第二种使用方法，相当于默认构建了一个 executor 函数
// 导出 token 传入 config，导出 cancel 函数来执行取消操作
CancelToken.source = function source() {
  var cancel;
  var token = new CancelToken(function executor(c) {
    cancel = c;
  });
  return {
    token: token,
    cancel: cancel
  };
};
```

## 总结

阅读Axios的过程，还是学到了很多东西：

1. Promise的串联操作
2. 拦截器的添加和执行原理
3. 将Promise的控制权导出，让外界决定Promise的状态

还有很繁琐也很重要的一部分没有涉及，就是针对HTTP请求的标准化处理，比如Heder的处理等，这也是大大方便开发者的功能之一，我们不用再担心这些细节的处理，只需要关注核心逻辑的实现。这也是优秀的组件和库的标准之一，暴露出简单、直接的接口让使用者调用，复杂、琐碎的逻辑隐藏在内部。

## 参考

- [Axios源码深度剖析 - AJAX新王者@知乎](https://zhuanlan.zhihu.com/p/58349237)
- [Axios中文说明@看云](https://www.kancloud.cn/yunye/axios/234845)
- [XMLHttpRequest@MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)
- [XMLHttpRequest@whatwg](https://xhr.spec.whatwg.org/#interface-formdata)
- [XMLHttpRequest.timeout@MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/timeout)
- [XMLHttpRequest.setRequestHeader()@MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/setRequestHeader)
- [XMLHttpRequest: error 事件@MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/error_event)
- [XMLHttpRequest.abort()@MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/abort)
- [XMLHttpRequest.getAllResponseHeaders()@MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/getAllResponseHeaders)
- [XMLHttpRequest.send()@MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/send)
