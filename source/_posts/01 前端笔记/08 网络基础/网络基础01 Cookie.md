---
title: 网络基础01 Cookie
top: false
date: 2018-01-04 09:53:00
updated: 2020-10-26 10:06:36
tags:
- HTTP
- cookie
- set-cookie
categories: 网络基础
---

Cookie学习笔记

<!-- more -->

# 什么是cookie

cookie是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。

通常，它用于告知服务端两个请求是否来自同一浏览器，如保持用户的登录状态。cookie使基于无状态的HTTP协议记录稳定的状态信息成为了可能。

cookie主要用于以下三个方面：

- 会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
- 个性化设置（如用户自定义设置、主题等）
- 浏览器行为跟踪（如跟踪分析用户行为等）

cookie曾一度用于客户端数据的存储，因当时并没有其它合适的存储办法而作为唯一的存储手段，但现在随着现代浏览器开始支持各种各样的存储方式，Cookie渐渐被淘汰。==由于服务器指定cookie后，浏览器的每次请求都会携带Cookie数据，会带来额外的性能开销==（尤其是在移动环境下）。

新的浏览器API已经允许开发者直接将数据存储到本地，如使用Webstorage（本地存储和会话存储）或IndexedDB 。

# cookie的分类

cookie分为会话期cookie和持久性cookie。

- 会话期cookie在浏览器关闭后会自动删除，没有指定`Expires`和`Max-age`的cookie为会话期cookie
- 持久性cookie可以指定一个特定的过期时间（`Expires`）和有效期（`Max-age`），设定的日期和时间只与客户端相关，而不是服务端。

# cookie创建时包含的字段

## `<cookie-name>=<cookie-value>`

设置`name`和`value`, 是必选字段，其他字段皆为可选

## `Expires=<date>`

规定cookie的过期日期，是一个绝对时间的日期值，如果没设置，cookie将在session结束后（即浏览器关闭后）失效（即会话期cookie）。HTTP/1.0的特性。

## `Max-Age=<seconds>`

规定了cookie的有效期（经过`seconds`秒后失效），是一个相对值。它是HTTP/1.1的特性，**假如`Expires`和`Max-Age`均存在，那么`Max-Age`优先级更高**。

## `Path=<path-value>`

指定一个URL路径，控制请求哪些路径下的页面能够读取cookie，`/`表示根路径下的文件有权限读取该cookie，`path`权限有继承性，此目录的下级目录也满足匹配的条件

例如，如果`path=/docs`，那么`/docs`、`/docs/Web/`或者`/docs/Web/HTTP`都满足匹配的条件

Cookie的路径是在服务器创建Cookie时设置的，它的作用是**决定浏览器访问服务器的某个资源时，需要将浏览器端保存的那些Cookie归还给服务器**。

## `Domain=<domain-value>`

指定cookie生效的域名。假如没有指定，那么默认值为当前文档访问地址中的主机部分（但是不包含子域名）。

假如指定了域名，那么相当于各个子域名也包含在内了。例如，`.baidu.com`的设置表明在`baidu.com`的所有域名中生效，如果设置为`www.baidu.com`，则仅仅在`www.baidu.com`域名中生效

## `Secure`

`Secure`设置为`true`时，cookie只应通过被HTTPS协议加密过的请求发送给服务器。默认为`false`

> 但即便设置了`Secure`，敏感信息也不应该通过cookie传输，因为Cookie有其固有的不安全性，`Secure`标记也无法提供确实的安全保障。

## `HttOnly`

设置了`HttpOnly`属性的cookie不能使用JavaScript进行访问，仅仅用于网络请求发送给服务端，目的是为了防范跨站脚本攻击（XSS）。

> XSS（Cross-site scripting）：跨站脚本攻击是一种安全漏洞，攻击者可以利用这种漏洞在网站上注入恶意的客户端代码。当被攻击者登陆网站时就会自动运行这些恶意代码，从而，攻击者可以突破网站的访问权限，冒充受害者。

# cookie的读取、设置和自动携带

## 不能设置或读取子域的cookie

设置或者读取cookie时，不能设置或者读取子域的cookie，只能向当前域或者更高级域名设置或读取cookid。

例如，`client.com`不能向`a.client.com`设置`cookie`，也不能读取`a.client.com`下的cookie，但是相反的，`a.client.com`可以设置或者读取`client.com`下的cookie

## cookie的自动携带

### 非Ajax请求

在进行非Ajax请求时，客户端只会带上与请求同源的cookie。例如请求`www.client.com/index.html`时，会带上`client.com`域名下的cookie，包括`client.com`和`www.client.com`下的cookie

### Ajax请求

原生的`fetch`请求方式，**不管是同域还是跨域请求都不会带上cookie**，只有设置了`credentials`时才会带上该请求所在域（及顶级域）的cookie，如果是跨域请求，服务端需要设置`Access-Control-Allow-Credentials: true`，否则会因为安全限制报错，无法获得响应。

```JS
fetch(url, {
    credentials: "include", // include, same-origin, omit
})

# include: 跨域 Ajax 带上 cookie
# same-origin: 仅同域 Ajax 带上 cookie
# omit: 任何情况都不带 cookie
```

而在使用axios进行Ajax请求时，同域会带上cookie，但是跨域请求不会，需要设置`withCredentials`，同时服务端设置跨域响应头


```JS
axios.get('http://server.com', {withCredentials: true})
```

# 对Cookie跨域的理解

假设服务端的域名是`a.com`，发送跨域请求的前端的域名是`b.com`，那么在`b.com`向`a.com`发送跨域请求时，通过上述设置后，是可以携带cookie的，但是携带的**这个cookie是域名为`a.com`下的cookie**

也就是说，跨域请求携带的cookie，是目标页面所在域的cookie，而非发送者所在域的cookie

我之前的理解有两个误区：

1. `b.com`向`a.com`发送跨域请求，可以把`b.com`域名下的cookie带上。如上面所说的，这是行不通的
2. `b.com`通过JS在本机生成一个域名`a.com`的Cookie，或者`a.com`的服务端在发送响应时`setCookie`的`domain`为`b.com`。这两种做法都是行不通的，因为**设置cookie的`domain`可以设置为父域名和自身，但是不能设置其他域名和子域名**，否则cookie设置不会成功。

`domain`假如没有指定，那么默认值为当前文档访问地址中的主机部分（但是不包含子域名）。与之前的规范不同的是，域名之前的点号会被忽略。假如指定了域名，那么相当于各个子域名也包含在内了。

也就是说：

1. 服务端是无法跨域设置cookie的（set-cookie），只能设置自身域名或者父域名的Cookie
2. 前端是可以带cookie跨域的，前提是cookie是目标服务器所在域的cookie


# cookie在网络请求中的传递

网络请求过程中cookie的传递有两个部分：

## `Set-Cookie`

服务器可以通过==响应头==中的`Set-Cookie`字段，请求浏览器保存==一个==cookie信息

多个字段用`;`分割

```
Set-Cookie: <cookie-name>=<cookie-value>; Domain=<domain-value>; Secure; HttpOnly

Set-Cookie: id=123; Domain=.toutiao.com; expires=Wed, 04-Apr-2018 09:59:25 GMT; Max-Age=7804800; Path=/
```

注意，不能将多个cookie放在一个`Set-Cookie`中，要设置多个cookie，需要添加同样多的set-Cookie字段

![](http://image.oldzhou.cn/18-12-18/10017224.jpg)

## `Cookie`

浏览器会在发送HTPP请求时，通过==请求头==中的`Cookie`字段，携带上当前页面保存的所有cookie

可以在`Cookie`字段中设置多个cookie:

![](http://image.oldzhou.cn/18-12-18/68010914.jpg)


# JavaScript操作本地cookie

JavaScript可以通过`document.cookie`创建cookie，也可以访问非`HttpOnly`标记的cookie

## 创建cookie

`document.cookie`每次只能写入一条cookie，需要写入多条cookie时，需要重复调用

后写入的重名的cookie会覆盖旧的cookie

```JS
document.cookie = "userId=828";
document.cookie = "username=ok";
document.cookie = "userId=919";
```

在Cookie的名或值中不能使用分号（`;`）、逗号（`,`）、等号（`=`）以及空格，所以在创建Cookie时需要用`encodeURIComponent`函数对要存入的值进行安全编码，读取时进行相应的解码

```JS
document.cookie = "userId=" + encodeURIComponent("string");
```

## 读取cookie

获取cookie值可以直接取值

```JS
var strCookie = document.cookie;
// "name=123; ok=999; ok2=0; ok3=999; ok4=123"
```

只能够一次获取所有的Cookie值，而不能指定cookie名称来获得指定的值

要获取`userId`的值，可以这样实现（==要注意的是分号`;`后面是有一个空格的，需要在分割为数组的时候处理==）：

```JS
var strCookie = document.cookie;

//将多cookie切割为多个名/值对
var arrCookie = strCookie.split(";");

var userId;
//遍历cookie数组，处理每个cookie对
for (var i = 0; i < arrCookie.length; i++) {
  var arr = arrCookie[i].split("=");
  //找到名称为userId的cookie，并返回它的值
  if ("userId" == arr[0]) {
    userId = arr[1];
    break;
  }
}
```

## 删除cookie

因为cookie是存储在客户端，所以服务器端是没法删除的，必须通知客户端去删除。==客户端通过设置cookie的`max-age`或者`expires`为`-1`将其删除==

为了考虑兼容性，二者应该同时使用。


## 封装

对cookie的操作比较复杂，所以可以进行封装：

```JS
const cookieJar = {
  set(name, value, days = 1) {
    const seconds = days * 24 * 60 * 60;
    document.cookie = `${name}=${value}; expires=${new Date(Date.now() + seconds * 1000)}; max-age=${seconds}`
  },
  get(name) {
    const {cookie} = document;
    if (!cookie) {
      return
    }
    const cookieObj = cookie.split('; ').reduce((total, current) => {
      const [ key, value ] = current.split('=');
      return Object.assign(total, { [key]: value} );
    }, {});
    return cookieObj[name]
  },
  remove(name) {
    document.cookie = `${name}=''; expires=-1; max-age=-1`
  }
}
```

或者直接使用线程的NPM包[js-cookie](https://www.npmjs.com/package/js-cookie)

# cookie的安全性问题

cookie的安全性存在着很大的不确定性，所以一个重要的原则是，**不能通过cookie存储、传输敏感信息**

所以通过cookie维护用户的登陆状态时，应该是通过cookie保存、传输经过加密后的一份登陆状态，而不是直接保存用户的密码信息。

常见的窃取cookie的攻击有：跨站脚本攻击（XSS）和跨站伪造请求。

一些可以用来提高安全性的手段：

1. 防止XSS攻击，对用于输入进行过滤
2. 使用HTTPS传输cookie
3. 用于明敏感信息的cookie只能拥有较短的生命周期
4. cookie加密
5. cookie里面加入IP信息，判断cookie中的IP信息和发送请求的IP是否相等，不相等采取措施，比如重新登录。
6. 物理机窃取是防止不了的，就是到你电脑上，拷走cookie，是防止不了的。

# 追踪和隐私

欧盟已经在2009/136/EC指令中提了相关要求，该指令已于2011年5月25日生效。虽然指令并不属于法律，但它要求欧盟各成员国通过制定相关的法律来满足该指令所提的要求。当然，各国实际制定法律会有所差别。

该欧盟指令的大意：在征得用户的同意之前，网站不允许通过计算机、手机或其他设备存储、检索任何信息。自从那以后，很多网站都在网站声明中添加了相关说明，告诉用户他们的cookie将用于何处。


# 参考

- [HTTP cookies@MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies)
- [Set-Cookie@MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Set-Cookie)
- [JavaScript Cookies@W3school](http://www.w3school.com.cn/js/js_cookies.asp)
- [如何防止cookie被盗用？？@segmentfault](http://blog.sina.com.cn/s/blog_70c4d9410100z3il.html)
- [Cookie的设置、读取以及是否自动携带问题@掘金](https://juejin.im/post/6844903648384778247)
