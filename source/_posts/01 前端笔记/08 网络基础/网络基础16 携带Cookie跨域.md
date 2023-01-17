---
title: 网络基础16 携带Cookie跨域
top: false
date: 2018-01-06 10:50:00
updated: 2019-06-11 16:49:47
tags:
- HTTP
- 跨域
- cookie
- set-cookie
categories: 网络基础
---

**普通的AJAX请求（非跨域的情况）是默认携带cookie的**，但是在跨域时则是不携带cookie的。跨域时携带cookie的方法有三种：

1. Nginx反向代理
2. JSONP
3. CORS

<!-- more -->

## 对Cookie跨域的理解

假设服务端的域名是`a.com`，发送跨域请求的前端的域名是`b.com`，那么在`b.com`想`a.com`发送跨域请求时，是可以携带cookie的，但是**这个cookie必须是域名为`a.com`下的cookie**

也就是说，`b.com`的前端发送的跨域请求携带的cookie，是目标页面所在域的cookie。

所以带cookie跨域的前提是目标页面的cookie在本机存在，**跨域要携带的cookie必须是目标页面所在域的cookie**

之前的理解有两个误区：

1. `b.com`向`a.com`发送跨域请求，可以把`b.com`域名下的cookie带上。如上面所说的，这是行不通的
2. `b.com`通过JS在本机生成一个域名`a.com`的Cookie，或者`a.com`的服务端在发送响应时`setCookie`的`domain`为`b.com`。这两种做法都是行不通的，因为**设置cookie的`domain`可以设置为父域名和自身，但是不能设置其他域名和子域名**，否则cookie设置不会成功。

`domain`假如没有指定，那么默认值为当前文档访问地址中的主机部分（但是不包含子域名）。与之前的规范不同的是，域名之前的点号会被忽略。假如指定了域名，那么相当于各个子域名也包含在内了。

1. 服务端是无法跨域设置cookie的（set-cookie），只能设置自身域名或者父域名的Cookie
2. 前端是可以带cookie跨域的，前提是cookie是目标服务器所在域的cookie


![](http://image.oldzhou.cn/FmuC1l7sIZo5etrwrkgh01no0_ya)


## Nginx反向代理

通过Nginx反向代理来解决cookie跨域问题可以携带cookie。

## JOSNP

使用JSONP跨域可以携带cookie，但是只能是GET请求，需要在请求的选项中添加`xhrFiles`对象：

```JS
$.ajax({
  url: 'http://b.fdipzone.com/server.php', // 跨域
  xhrFields: {
    withCredentials: true // 发送凭据
  }, 
  dataType: 'json',
  type: 'post',
  data: {
    'name': 'fdipzone'
  },
  success: function(ret) {
    if (ret['success'] == true) {
      alert('cookie:' + ret['cookie']);
    }
  }
});
```

## CORS

CORS的全称是“跨域资源共享“（Cross-Origin Resource Sharing）

它允许浏览器向跨源服务器发出XMLHttpRequest请求，解决跨域问题。

CORS请求默认不发送Cookie和HTTP认证信息。如果要把Cookie发到服务器，一方面要服务器同意，指定`Access-Control-Allow-Credentials`字段：

```
header("Access-Control-Allow-Origin: http://www.xxx.com");  
header("Access-Control-Allow-Credentials: true"); 
```

另一方面，开发者必须在AJAX请求中打开`withCredentials`属性：

```
// 原生
var xhr = new XMLHttpRequest();
xhr.open('GET', 'xx.com');
xhr.withCredentials = truel
xhr.send()

// jQuery
$.ajax({
  type: 'GET',
  url: 'xx.com',
  xhrFields: {
    withCredentials: true
  },
  crossDomain: true
})
```
需要注意的是，如果要发送Cookie，`Access-Control-Allow-Origin`就不能设为星号，**必须指定明确的、与请求网页一致的域名**。

同时，Cookie依然遵循**同源政策**，只有用服务器域名设置的Cookie才会上传，其他域名的Cookie并不会上传，且（跨源）原网页代码中的`document.cookie`也无法读取服务器域名下的Cookie。

CORS与JSONP的使用目的相同，但是比JSONP更强大。

JSONP只支持GET请求，CORS支持所有类型的HTTP请求。JSONP的优势在于支持老式浏览器，以及可以向不支持CORS的网站请求数据。

## 参考

- [HTTP cookies@MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies)
- [Set-Cookie@MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Set-Cookie)
- [解决cookie跨域访问@博客园](https://www.cnblogs.com/hujunzheng/p/5744755.html)
- [跨域资源共享 CORS 详解@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2016/04/cors.html)
- [跨域请求带cookie的解决方案@CSDN](https://blog.csdn.net/sysuzjz/article/details/51566713)
