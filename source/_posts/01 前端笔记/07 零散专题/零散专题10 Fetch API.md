---
title: 零散专题10 Fetch API
top: false
date: 2017-05-08 16:45:30
updated: 2019-05-09 11:46:47
tags:
- Fetch
- Axios
- Ajax
categories: 其他
---

Fetch是基于Promise设计的，是属于`window`对象的方法，可以用来替代Ajax。旧浏览器不支持Promise，需要使用Polyfill进行处理 。

<!-- more -->

## 兼容性

![ajax08.png](http://image.oldzhou.cn/ajax08.png)

目前浏览器对Fetch的原生的支持率并不高，幸运的是，引入下面这些polyfill后可以完美支持IE8+ ：

- 由于IE8是ES3，需要引入ES5的polyfill: `es5-shim`, `es5-sham`
- 引入`Promise`的`polyfill: es6-promise`
- 引入Fetch探测库：`fetch-detector`
- 引入Fetch的polyfill: `fetch-ie8`
- 可选：如果你还使用了Jsonp，引入`fetch-jsonp`

如果在Node使用，还需要安装`node-fetch`模块。

## 使用

一个基本的Fetch请求如下：

```JS
fetch('http://example.com/movies.json')
  .then(function(response) {
    return response.json();
  })
  .then(function(myJson) {
    console.log(myJson);
  });
```

也可以通过`Request`构造器函数创建一个新的请求对象，这也是建议标准的一部分。第一个参数是请求的URL，第二个参数是一个选项对象，用于配置请求。请求对象一旦创建了，便可以将所创建的对象传递给`fetch()`方法，用于替代默认的URL字符串。示例代码如下：

```JS
const req = new Request(URL, {method: 'GET', cache: 'reload'});

fetch(req).then(function(response) {
  return response.json();
}).then(function(json) {
  insertPhotos(json);
});
```

上面的代码中指明了请求使用的方法为GET，并且指定不缓存响应的结果。

## 参数

fetch接受两个参数：

- 第一个参数是请求的目标URL
- 第二个参数是一个配置项对象，包括所有对请求的设置，可选参数有`method`、`headers`、`mode`、`credentials`、`cache`等

关于参数的详细说明可以参考[MDN的文档](https://developer.mozilla.org/zh-CN/docs/Web/API/WindowOrWorkerGlobalScope/fetch)。

## 返回值

Fetch的返回值是一个Promise对象，当这个Promise对象resolve时，在`then`方法中获得的是一个Resopnse对象。

Response对象中包含了当次请求的相应数据，需要调用对应的方法将Response对象转换为相应格式的数据，最常用的就是`json()`方法，它将返回一个被解析为JSON格式的Promise对象，并将Response对象设置为已读。

Response对象常用属性包括`headers`、`ok`、`redirected`、`status`、`statusText`、`type`、`url`等，常用的方法包括`clone()`、`error()`、`formData()`、`text()`等，具体的说明参考[MDN的文档](https://developer.mozilla.org/zh-CN/docs/Web/API/Response)。


## 与 Ajax 的区别

Ajax是基于XMLHttpRequest对象来发送网络请求、获取数据的。
 
Fetch是基于Promise设计的，是属于`window`对象的方法，可以用来替代Ajax。

使用原生的Ajax发送一个JSON请求一般是这样：

```JS
var xhr = new XMLHttpRequest();
xhr.open('GET', url);
xhr.responseType = 'json';
xhr.send(null);

xhr.onredaystatechange = function() {
  if (xhr.readystate === 4) {
     if (xhr.status === 200) {
        console.log(xhr.response) 
     } 
  }
};

xhr.onerror = function() {
  console.log("Oops, error");
};
```

使用Fetch来进行同样的操作：

```JS
fetch(url).then(response => response.json())
  .then(data => console.log(data))
  .catch(e => console.log("Oops, error", e))
```

可以使用`Async/Await`来实现“

```JS
try {
  let response = await fetch(url);
  let data = response.json();
  console.log(data);
} catch(e) {
  console.log("Oops, error", e);
}
// 注：这段代码如果想运行，外面需要包一个 async function
```

总结起来，Fetch与Ajax的主要的**不同点**在于：

（1）Fetch基于Promise对象，Ajax基于XMLHttpRequest对象

（2）当收到代表错误的HTTP状态码时(4xx/5xx)，从`fetch()`返回的Promise**不会被标记为reject**，而是标记为resolve（但是会将resolve的返回值的`ok`属性标记为`false`），仅当网络故障时或请求被阻止，才会标记为reject；

来看下面的Fetch请求：

```JS
fetch('http://127.0.0.1:7001/getTitle')
  .then(v => console.log('resolve, v.ok = ', v.ok))
  .catch(e => console.log('reject', e));
  
// 当服务器返回结果正常（200）时，返回结果：
// resolve, v.ok =  true

// 当服务器返回结果异常（500）时，返回结果：
// resolve, v.ok =  false

// 当网络故障或请求被阻止时（设置跨域失败），返回结果：
// reject TypeError: Failed to fetch
```

当使用原生的Ajax时：

```JS
const xhr = new XMLHttpRequest();
xhr.open('GET', 'http://127.0.0.1:7001/getTitle');
xhr.send();

xhr.onreadystatechange = () => {
  if (xhr.readyState === 4) {
    if (xhr.status === 200) {
      console.log('success', xhr.status);
    } else {
      console.log('error', xhr.status);
    }
  }
};

xhr.onerror = (e) => {
  console.log('onerror', e.type);
}

// 当服务器返回结果正常（200）时，返回结果：
// success 200

// 当服务器返回结果异常（500）时，返回结果：
// error 500

// 当网络故障或请求被阻止时（设置跨域失败），返回结果：
// error 0
// nerror error
```

现在使用比较多的axios也是基于原生的XMLHttpRequest对象进行的封装，它可以在Node中使用（Fetch不行），它的行为比起Fetch更加合理，当收到代表错误的HTTP状态码时(4xx/5xx)，从`axios()`返回的Promise会被标记为reject，

```JS
axios.get('http://127.0.0.1:7001/getTitle')
  .then(v => console.log('resolve', v.data))
  .catch(e => console.log('reject', e));

// 当服务器返回结果正常（200）时，返回结果：
// resolve {title: "OK"}

// 当服务器返回结果异常（500）时，返回结果：
// reject Error: Request failed with status code 500
//    at createError (createError.js:17)
//    at settle (settle.js:19)
//    at XMLHttpRequest.handleLoad (xhr.js:78)

// 当网络故障或请求被阻止时（设置跨域失败），返回结果：
// reject Error: Network Error
//    at createError (createError.js:17)
//    at XMLHttpRequest.handleError (xhr.js:87)
```

（3）默认情况下，Fetch不会从服务端发送或接受任何cookie，要发送cookie，必须设置credentials选项。而AJAX请求默认自动带上同源的cookie，不会带上不同源的cookie。可以通过前端设置`withCredentials`为`true`、后端设置`Header`的方式来让Ajax带上不同源的Cookie

## fetch发送Cookie

为了让浏览器发送包含凭据的请求（即使是跨域源），要将`credentials: 'include'`添加到传递给`fetch()`方法的`init`对象。

```JS
fetch('https://example.com', {
  credentials: 'include'  
})
```

果你只想在请求URL与调用脚本位于同一起源处时发送凭据，请添加`credentials: 'same-origin'`。要改为确保浏览器不在请求中包含凭据，请使用`credentials: 'omit'`。

## 例子

### 上传JSON数据

```JS
const url = 'https://example.com/profile';
const data = { username: 'example' };

fetch(url, {
  method: 'POST', // or 'PUT'
  body: JSON.stringify(data), // data can be `string` or {object}!
  headers: new Headers({
    'Content-Type': 'application/json'
  })
}).then(res => res.json())
  .catch(error => console.error('Error:', error))
  .then(response => console.log('Success:', response));
```

### 上传文件

可以通过HTML`<input type="file" />`元素，`FormData()`和`fetch()`上传文件。

```JS
var formData = new FormData();
var fileField = document.querySelector("input[type='file']");

formData.append('username', 'abc123');
formData.append('avatar', fileField.files[0]);

fetch('https://example.com/profile/avatar', {
  method: 'PUT',
  body: formData
}).then(response => response.json())
  .catch(error => console.error('Error:', error))
  .then(response => console.log('Success:', response));
```

### 获取图片

下面这个例子，是在React中，使用`fetch`获取图片，并将图片转换为Blob对象，赋给`<img>`的`src`，在页面上展示图片：

```
import React, { useState, useEffect, useRef } from 'react';
import fetch2 from './fetch2.png';

export default function Statistics() {
  const [src, setSrc] = useState('');
  useEffect(() => {
    fetch('fetch2.png').then(v => v.blob()).then(v => {
      setSrc(URL.createObjectURL(v));
    });
  }, [param]);

  return (
    <div className={styles.container}>
      <img src={} />
    </div>
  );
}
```

## 使用Fetch需要注意的问题

1. 考虑Fetch兼容性（IE浏览器不支持Fetch）
2. 考虑环境对Promise的支持情况
3. 如果在Node中使用需要安装`node-fetch`
4. 服务端返回错误时（4xx/5xx），Fetch不会reject，可以通过返回的Response对象的OK属性判断
5. 默认情况下Fetch不会接受或者发送Cookie，需要使用`credentials`选项开启

## 参考

- [使用Fetch@MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch)
- [fetch API@DWB](https://davidwalsh.name/fetch)
- [传统 Ajax 已死，Fetch 永生@segmentfault](https://segmentfault.com/a/1190000003810652)
- [WorkerOrGlobalScope.fetch()@MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/WindowOrWorkerGlobalScope/fetch)
- [Response@MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Response)
- [【误】Ajax不会自动带上cookie/利用withCreadentials带上cookie@知乎](https://zhuanlan.zhihu.com/p/28818954)
- [XMLHttpRequest.withCredentials@MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/withCredentials)
- [fetch和ajax的区别@segmentfault](https://segmentfault.com/a/1190000011019618)
