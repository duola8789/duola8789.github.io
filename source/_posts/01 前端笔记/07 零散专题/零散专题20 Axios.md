---
title: 零散专题20 Axios
top: false
date: 2017-12-22 12:03:35
updated: 2019-11-27 12:30:30
tags:
- axios
categories: 其他
---

Axios学习笔记以及进行超时重试。

<!-- more -->

在学习Vue的时候，接触到了Axios这个库，查了一些资料，发现这个库还很吊，Vue更新到2.0之后，作者就宣告不再对`vue-resource`更新，而是推荐使用Axios

## 简介

[Axios](https://www.npmjs.com/package/axios) 是一个基于Promise的HTTP库，可以用在浏览器和Node中。

使用Npm安装

```BASH
$ npm install axios -S
```

直接引入：

```HTML
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```

## API

### `GET`请求

```JS
// 为给定 ID 的 user 创建请求
axios.get('/user?ID=12345')
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });


// 可选地，上面的请求可以这样做
axios.get('/user', {
    params: {
      ID: 12345
    }
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
```

### `POST`请求

```JS
axios.post('/user', {
    firstName: 'Fred',
    lastName: 'Flintstone'
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
```

### 执行多个并发请求

```JS
function getUserAccount() {
  return axios.get('/user/12345');
}

function getUserPermissions() {
  return axios.get('/user/12345/permissions');
}

axios.all([getUserAccount(), getUserPermissions()])
  .then(axios.spread(function (acct, perms) {
    // 两个请求现在都执行完成
  }));
```

## `config`配置

config是对一些基本信息的配置，比如请求头，`baseURL`

只有`url`选项是必须的，如果`method`选项未定义，那么它默认是以`GET`的方式发出请求

```JS
{
  // url 是请求的服务器地址
  url:'/user',
  
  // method 是请求资源的方式
  method:'get'// default
  
  // 如果 url 不是绝对地址，那么 baseURL 将会加到 url 的前面
  // 当 url 是相对地址的时候，设置 baseURL 会非常的方便
  baseURL:'https://some-domain.com/api/',
  
  // transformRequest 选项允许我们在请求发送到服务器之前对请求的数据做出一些改动
  // 数组中的函数必须返回一个字符串、-一个 ArrayBuffer 或者 Stream
  transformRequest:[function(data){
    //在这里根据自己的需求改变数据
    return data;
  }],
  
  // transformResponse 选项允许我们在数据传送到 then/catch 方法之前对数据进行改动
  transformResponse:[function(data){
    //在这里根据自己的需求改变数据
    return data;
  }],
  
  // headers 选项是需要被发送的自定义请求头信息
  headers: {'X-Requested-With':'XMLHttpRequest'},
  
  // params 选项是要随请求一起发送的请求参数----一般链接在URL后面
  // 他的类型必须是一个纯对象或者是URLSearchParams对象
  params: {
    ID: 12345
  },
  
  // paramsSerializer 是一个可选的函数，起作用是让参数（params）序列化
  paramsSerializer: function(params){
    return Qs.stringify(params,{arrayFormat:'brackets'})
  },
  
  // data 选项是作为一个请求体而需要被发送的数据
  // 该选项只适用于方法：put/post/patch
  // 当没有设置 transformRequest 选项时 dada 必须是以下几种类型之一
  // string/plain/object/ArrayBuffer/ArrayBufferView/URLSearchParams
  // 仅仅浏览器：FormData/File/Bold
  // 仅 node:Stream
  data {
    firstName:"Fred"
  },
  
  // timeout 选项定义了请求发出的延迟毫秒数
  // 如果请求花费的时间超过延迟的时间，那么请求会被终止
  timeout: 1000,
  
  // withCredentails 选项表明了跨域请求是否携带 cookie
  withCredentials:false, // default
  
  // adapter 适配器选项允许自定义处理请求，这会使得测试变得方便
  // 返回一个promise,并提供验证返回
  adapter: function(config){
    /*..........*/
  },
  
  // auth 表明 HTTP 基础的认证应该被使用，并提供证书
  // 这会设置一个 authorization 头,并覆盖你在 header 设置的 Authorization 头信息
  auth: {
    username: "zhangsan",
    password: "s00sdkf"
  },
  
  // 返回数据的格式
  // 其可选项是 arraybuffer, blob, document, json, text, tream
  responseType: 'json', // default

  // xsrfCookieName 是用作 xsrf token 的值的cookie的名称
  xsrfCookieName: 'XSRF-TOKEN', // default
  
  // xsrfHeaderName 是承载 xsrf token 的值的 HTTP 头的名称
  xsrfHeaderName: 'X-XSRF-TOKEN', // default
  
  // onUploadProgress 上传进度事件
  onUploadProgress: function (progressEvent) {
    // 对原生进度事件的处理
  },
  
  // 下载进度的事件
  onDownloadProgress: function (progressEvent){
    // 对原生进度事件的处理
  },
  
  //相应内容的最大值
  maxContentLength: 2000,
  
  // validateStatus 定义了是否根据 http 相应状态码，来 resolve 或者 reject promise
  // 如果 validateStatus 返回 true (或者设置为 null 或者 undefined),那么 promise 的状态将会是 resolved，否则其状态就是 rejected
  validateStatus: function (status) {
    return status >= 200 && status < 300; // default
  },
  
  // maxRedirects 定义了在 nodejs 中重定向的最大数量
  maxRedirects: 5, // default
  
  // httpAgent/httpsAgent 定义了当发送 http/https 请求要用到的自定义代理
  // keeyAlive 在选项中没有被默认激活
  httpAgent: new http.Agent({ keeyAlive:true }),
  httpsAgent: new https.Agent({ keeyAlive:true }),
  
  // proxy 定义了主机名字和端口号，
  // auth 表明 http 基本认证应该与 proxy 代理链接，并提供证书
  // 这将会设置一个 Proxy-Authorization header,并且会覆盖掉已经存在的 Proxy-Authorization header
  proxy: {
    host: '127.0.0.1',
    port: 9000,
    auth: {
      username:'skda',
      password:'radsd'
    }
  },
  
  // cancelToken 定义了一个用于取消请求的 cancel token
  // 详见cancelation部分
  cancelToken: new cancelToken(function (cancel) {
  })
}
```

## 请求返回的内容


```JS
{
  data: {},         // 由服务器提供的响应
  status: 200，     // 从服务器返回的 http 状态码
  statusText: 'OK', // 从服务器返回的 http 状态文本
  headers: {},      // 响应头信息
  config: {}        // config 是在请求的时候的一些配置信息
}
```

可以这样来获取响应信息：

```JS
axios.get('/user/12345')
  .then(function (res) {
    console.log(res.data);
    console.log(res.status);
    console.log(res.statusText);
    console.log(res.headers);
    console.log(res.config);
  })
```

其他API的介绍可以看[这里](https://www.kancloud.cn/yunye/axios/234845)。

## 利用拦截器进行超时重试

响应拦截器可以接受两个参数，第一个参数时响应成功的拦截函数，第二个参数时响应失败的拦截函数。当我们在创建`axios`实例的时候，如果配置了`timeout`选项，当超过这个时间接口未响应，axios就会终止请求，并且返回一个`err`对象。

这个`err`对象中不仅包括了错误信息，还包括了请求的`config`对象，可以从axios源码的`xhr.js`中发现：

```JS
// Handle timeout
request.ontimeout = function handleTimeout() {
  reject(createError('timeout of ' + config.timeout + 'ms exceeded', config, 'ECONNABORTED', request));
  // Clean up request
  request = null;
};
```

可以通过`err.message`来判断是否是超时错误，而且通过`config`对象，我们可以再次发起请求。

首先在创建`axios`实例的时候添加了除了`timeout`之外的三个自定义属性，用来限制是否重试、重试的最大次数和重试的间隔：

```JS
const axiosInstance = axios.create({
  timeout: 1000,
  retry: true,
  retryInterval: 1000,
  retryMaximum: 5,
});
```

这些都是可配置的，因为它们都会被axios合并入`config`对象中

然后在响应拦截器中进行重试处理：

```JS
axiosInstance.interceptors.response.use(function (response) {
  return response;
}, function axiosRetryInterceptor(err) {
  const { message, config } = err;

  // 只针对超时的错误进行重试
  if (!message.includes('timeout')) {
    return Promise.reject(err);
  }

  // 不需要重试
  if (!config || !config.retry)  {
    return Promise.reject(err);
  }

  const retryInterval = config.retryInterval || 1000,
    retryMaximum = config.retryMaximum || 5;

  config._retryCount = config._retryCount || 0;

  // 已经超过最大重试次数
  if (config._retryCount >= retryMaximum) {
    return Promise.reject(err);
  }

  // 重试次数 + 1
  config._retryCount += 1;


  const retryPromise = new Promise((resolve) => {
    setTimeout(resolve, retryInterval)
  });

  return retryPromise.then((() => axiosInstance.request(config)));
});
```


## 参考

- [axios@简书](http://www.jianshu.com/p/df464b26ae58)
- [Axios中文说明@看云](https://www.kancloud.cn/yunye/axios/234845)
- [axios基本用法@博客园](https://www.cnblogs.com/Upton/p/6180512.html)
- [vue axios全攻略@博客园](https://www.cnblogs.com/libin-1/p/6607945.html)
- [axios请求超时,设置重新请求的完美解决方法@掘金](https://juejin.im/post/5abe0f94518825558a06bcd9)
