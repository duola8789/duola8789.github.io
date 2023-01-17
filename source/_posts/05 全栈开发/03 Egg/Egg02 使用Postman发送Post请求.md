---
title: Egg02 使用Postman发送Post请求
top: false
date: 2019-05-05 19:44:42
updated: 2019-05-05 19:44:45
tags:
- Postman
- Node
categories: Egg
---

使用Postman测试Egg框架实现的API

<!-- more -->

## 关闭安全防范

默认情况下，Egg在框架中内置了安全插件`egg-security`，插件中针对`post`请求做出了一些处理，提供了一些默认的安全实践，并且框架的安全插件是默认开启的，如果需要关闭其中一些安全防范，直接设置该项的`enable`属性为`false`即可。

写例子的话可临时在`config/config.default.js`中设置

```JS
exports.security = {
  csrf: false
};
```

## 在网页发送请求

在AJAX请求中，默认配置下，token会被设置在Cookie中，在AJAX请求时，可以从Cookie中获取到token，放置到query、body或者header中发送给服务端：

使用jQuery：

```JS
var csrftoken = Cookies.get('csrfToken');

function csrfSafeMethod(method) {
  // these HTTP methods do not require CSRF protection
  return (/^(GET|HEAD|OPTIONS|TRACE)$/.test(method));
}
$.ajaxSetup({
  beforeSend: function(xhr, settings) {
    if (!csrfSafeMethod(settings.type) && !this.crossDomain) {
      xhr.setRequestHeader('x-csrf-token', csrftoken);
    }
  },
});
```

通过header传递CSRF token的字段可以在配置中改变：

```JS
// config/config.default.js
module.exports = {
  security: {
    csrf: {
      headerName: 'x-csrf-token', // 通过 header 传递 CSRF token 的默认字段为 x-csrf-token
    },
  },
};
```

## 在Postman中发送请求

使用Postman来测试API时，直接发送POST请求，会返回403，因为没有传递CSRF token，这个时候可以通过环境配置，来让Postman自动生成CSRF token。

这个功能需要使用独立的Postman APP，Chrome插件是不行的。

在Postman右上角，新建一个环境：

![postman1.png](http://image.oldzhou.cn/postman1.png)

点击`Add`添加后，将环境切换为新建的环境：

![postman2.png](http://image.oldzhou.cn/postman2.png)

然后在`Tests`标签下，通过下面的脚本，获取cookie中的`csfrtoken`，并且写入到postman的全局变量中：

```JS
var csrf_token = postman.getResponseCookie("csrftoken").value
postman.clearGlobalVariable("csrftoken");
postman.setGlobalVariable("csrftoken", csrf_token);
```
然后发送一个Get请求，来写入Cookie：

![postman3.png](http://image.oldzhou.cn/postman3.png)

在发送Post请求时，在Header中添加`{{csrftoken}}`字段：

![postman4.png](http://image.oldzhou.cn/postman4.png)

再点击发送Post请求就没问题了。

## 参考
- [安全@Egg](https://eggjs.org/zh-cn/core/security.html#)
- [postman自动设置token(csrf及authorization token)@knktc的杂乱空间](https://knktc.com/2018/06/03/postman-set-token/)
