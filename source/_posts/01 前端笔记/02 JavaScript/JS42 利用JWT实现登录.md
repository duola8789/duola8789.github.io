---
title: JS42 利用JWT实现登录
top: false
date: 2018-01-07 13:05:00
updated: 2020-01-15 08:45:43
tags:
- JWT
- Json Web Token
- 登陆
categories: JavaScript
---

JWT学习笔记。

<!-- more -->

[TOC]

# 利用JWT实现登录

## 1 什么是JWT

JWT是JSON WEB TOKEN的缩写，是为了在网络应用环境建传递声明而执行的一种基于JSON的开放标准。该Toekn被设计为紧凑并且安全的，特别适用于分布式站点的单点登录（SSO）

## 2 传统的session认证

传统的session认证是为了让我们的应用能识别是哪个用户发送的请求，需要在服务器存储一份用户的登陆信息，这份登陆信息会在返给用户的响应时传递给浏览器，浏览器保存为cookie，下次请求时发送给服务器，用来验证用户身份。

基于session认证存在一些问题：

1. 用户增多，服务器保存的用户信息越来越多，服务端开销明显增大
2. 在分布式的应用上，会限制负载均衡器的能力
3. 由于是基于cookie来进行用户识别的，如果cookie被截取，会受到跨站请求伪造（CSRF）的攻击

## 3 基于token的鉴权机制

基于token的鉴权机制类似于http协议，也是无状态的，不需要在服务端保留用户的认证信息或者会话信息。这就意味着基于token认证的应用不需要考虑用户在哪一台服务器登陆了，为应用的扩展提供了遍历。

主要的流程如下：

1. 用户使用用户名和密码来请求服务器登陆
2. 服务器验证用户信息
3. 服务器对通过验证的用户下发一个token
4. 客户端收到token并存储，并且在以后的每次请求时再请求头中携带这个token
5. 服务端在收到后续请求后对token验证，通过时返回对应数据

## 4 JWT的构成

JWT由三段信息构成，第一部分为头部（header），第二部分为载荷（payload），第三部分是签名（signature）

![](http://image.oldzhou.cn/Fl_aibyiQ7_rLSLp70FbCMGIp6Ay)

### 4.1 header

header用来描述描述关于该JWT的最基本的信息，承载两部分信息：

1. 声明类型，这里是JWT
2. 声明签名算法

这也可以被表示成一个JSON对象。

```JS
{
  'typ': 'JWT',
  'alg': 'HS256'
}
```
然后将头部进行base64加密，构成了第一部分

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```
### 4.2 payload

载荷是存放有效信息的地方，包含三个部分：

1. 标准中注册的声明（建议但不强制使用）
    - iss: jwt签发者
    - sub: jwt所面向的用户
    - aud: 接收jwt的一方
    - exp: jwt的过期时间，这个过期时间必须要大于签发时间
    - nbf: 定义在什么时间之前，该jwt都是不可用的
    - iat: jwt的签发时间
    - jti: jwt的唯一身份标识，主要用来作为一次性token，从而回避重放攻击。
2. 公共的声明：可以添加任何信息，一般添加用户的相关信息或其他业务需要的必要信息，但**不建议添加敏感信息，因为该部分在客户端可解密**
3. 私有的声明：提供者和消费者所共同定义的声明，不建议存放敏感信息，因为base64编码并不是加密过程，可以归类为明文信息

一个payload如下：

```JS
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```
然后对其进行base64编码，得到JWT的第二部分

```TEXT
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
```

### 4.3 signature

签名信息由三个部分组成：

1. header（base64编码后的）
2. payload（base64编码后的）
3. secret

secret是利用HS256T算法进行加密时的密钥，将header和payload使用secret进行加密后，构成了jwt的第三部分

```JS
// javascript
var encodedString = base64UrlEncode(header) + '.' + base64UrlEncode(payload);

var signature = HMACSHA256(encodedString, 'secret'); // TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```
将这三部分用`.`连接成一个完整的字符串，构成了最终的jwt

**注意，jwt的签发生成是在服务器端的，secret是保存在服务端的，用来进行jwt的签发和验证。它是服务端的密钥，在任何场景都不应该流出。一旦客户端得到这个sercret，那就意味着客户端可以自我签发jwt了。**

## 5 如何存储发送JWT

一般来说，有两种方案来存储和发送JWT：

### 5.1 利用Cookie

第一种是将由服务端下发一个cookie，并且设置`HttpOnly`（还可以设置`secure`，确保cookie只通过Https来传输），将JWT放在这个cookie中，当发送请求时自动带着这个cookie进行请求。

它的优点是，黑客无法直接读取设置了`HttpOnly`的cookie，从某种程度上来说更安全，但是仍要小心CSRF和XSS的攻击。（尤其是CSRF，因为cookie会由浏览器自动发送，当被攻击者登陆了危险网站时，cookie可能会被无意间盗用）


### 5.2 利用 Local Storgae

另外一种（更常见）的方案是在Local Storage中存储JWT，然后由前端提取出来添加到请求的Authorization Header中发送。

由于请求头中的信息不像cookie一样是由浏览器自动发送的，所以对[CSRF](https://duola8789.github.io/2017/11/06/01%20%E5%89%8D%E7%AB%AF%E7%AC%94%E8%AE%B0/08%20%E7%BD%91%E7%BB%9C%E5%9F%BA%E7%A1%80/%E7%BD%91%E7%BB%9C%E5%9F%BA%E7%A1%8007%20HTTP%E5%AE%89%E5%85%A8/#CSRF)具有天然的防御，并且Local Storage的容量比Cookie大的多。

它的缺点是，如果网站出现了XSS漏洞，那么攻击者可以极其轻松的获取到JWT，然后就可以伪造并且用户身份去实现攻击行为。

### 5.3 具体操作

比较常见和稳妥的做法是：确保代码以及第三方库的代码有足够的XSS检查，在此之上将jwt存放在Local Storage中。前端具体的实现根据采用的Ajax工具不同有所不同：

#### （1）使用`fetch`发送

发送jwt时一般是在请求头里加入`Authorization`，并加上`Bearer`标注：

```JS
fetch('api/user/1', {
  headers: {
    'Authorization': 'Bearer ' + token
  }
})
```

服务端会验证token，如果验证通过就会返回相应的资源。

#### （2）使用`axios`发送

在Vue中经常使用[axios](https://www.axios.com/)来帮助我们处理网络请求，使用axios设置请求头如下

```JS
import axios from 'axios';
import JsonWebToken from 'jsonwebtoken';

// 在登录成功后应该已经将token存到本机了
const token = sessionStorage.getItem('userToken');

// iss是我们预先定义信息，用来识别token是我们所需要的token
const isTokenValid = !!(token && JsonWebToken.decode(token) && (JsonWebToken.decode(token).iss === config.userToken.iss));

// 如果token有效
if (isTokenValid) {
  // get请求
  axios.get('/api/user/deatil', {
    headers: {
      'Authorization': 'Bearer ' + token,
      'Cookie' : 'sessionId=' + sessionId + '; recId=' + recId,
    }
    params: {
      param1: 'string',
      param2: 'string'
    },
  })

  // post请求
  axios.post('/api/user/deatil',
    {
      data: {value1: 'value}'
    },
    {
      headers: {
        'Authorization': 'Bearer ' + token,
        'Cookie' : 'sessionId=' + sessionId + '; recId=' + recId,
      }
  })

  // 全局设定
  axios.headers.common['Authorization'] = 'Bearer ' + token
}
```


## 6 在Koa2中的应用

[jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken)是Node环境下JWT的实现，提供了一些API帮助我们快速的实现JWT的设计。

安装：

```
$ npm install jsonwebtoken --save
```
### 6.1 签发token（加密）

```JS
jwt.sign(payload, secretOrPrivateKey, [options, callback])
```

- `payload`：可以是`object`或者`string`或者`buffer`（`object`会被使用`JSON.stringify`方法转换为字符串）
- `secretOrPrivateKey`：用来加密的密钥
- `options`：
    - `algorithm`：加密算法（默认值SH256）--（`alg`）
    - `expriesIn`：失效时间，数字默认单位为秒，字符串需指明时间单位，如`60`、`2 days`、`10h`、`7d`（如果是字符串并且没有单位，单位默认为毫秒）--（`exp`）
    - `notBefore`：生效时间，单位规则同`expriesInd`--（`nbf`）
    - `audience`：token的接受者--（`aud`）
    - `issuer`：签名的发行者--（`iss`）
    - `jwtid`：JWT的ID
    - `subject`：JWT的主题--（`sub`）
    - `noTimestamp`：不需要时间戳
    - `header`：头部信息
- `callback`：加密失败的回调函数

生成一个有效期为1小时的token:

```JS
jwt.sign({
  data: 'foobar'
}, 'secret', { expiresIn: 60 * 60 });

jwt.sign({
  data: 'foobar'
}, 'secret', { expiresIn: '1h' });
```

也可以用下面这种形式：

```JS
jwt.sign({
  exp: Math.floor(Date.now() / 1000) + (60 * 60),
  data: 'foobar'
}, 'secret');
```
### 6.2 验证token

验证token的合法性：

```JS
jwt.verify（token，secretOrPublicKey，[options，callback]）
```
来看验证时的一些简单的用法，更详细的用法参考[官网](https://www.npmjs.com/package/jsonwebtoken)。

```JS
// 同步验证成功
var decoded = jwt.verify(token, 'shhhhh');
console.log(decoded.foo) // bar

// 异步验证成功
jwt.verify(token, 'shhhhh', function(err, decoded) {
  console.log(decoded.foo) // bar
});

// 同步验证失败--token无效
try {
  var decoded = jwt.verify(token, 'wrong-secret');
} catch(err) {
  // err
}

// 异步验证失败--token无效
jwt.verify(token, 'wrong-secret', function(err, decoded) {
  // err
  // decoded undefined
})
```

### 6.3 解码token的payload

将payload解码，**注意：这个过程无论验证是否通过都会执行**


```JS
jwt.decode(token [, options])

// get the decoded payload ignoring signature, no secretOrPrivateKey needed
var decoded = jwt.decode(token);

// get the decoded payload and header
var decoded = jwt.decode(token, {complete: true});
console.log(decoded.header);
console.log(decoded.payload)
```
- `token`：JWT的token
- `options`：
    - `json`：强制使用`JSON.parse()`机械payload，即使头信息中不包括`"typ": "JWT"`
    - `complete`：返回一个包含解码后的payload和对象

看一下例子：

```JS
// 解码payload不关心签名，不需要提供私钥
var decoded = jwt.decode(token);

// 获得对象
var decoded = jwt.decode(token, {complete: true});
console.log(decoded.header);
console.log(decoded.payload)
```
### 6.4 实际例子

来看一个实际的例子：

```JS
const jwt = require('jsonwebtoken');

//密钥，不能暴露
const secret = 'aaa';

// payload中不应该包含敏感信息
const payload = {
  username: 'jay',
  userId: 123
}

// 选项（非必须）
const options = {
  expiresIn: 60 * 60 * 24 // 或者为"1d"
}

// 生成token
const token = jwt.sign(payload, secret, options)

// 验证token
jwt.verify(token, secret, function (err, decoded) {
  // 如果验证通过
  if (!err){
    console.log(decoded.username);  // 'jay'
  }
})
```
## 7 koa-jwt

在Koa2中实现JWT的认证有中间件[koa-jwt](https://www.npmjs.com/package/koa-jwt)帮助我们完成，但是签发token的过程还是要安装上述过程，使用jsonwebtoken实现

直接看一个简单的例子：


```JS
import Koa from 'koa';
import jwt from 'koa-jwt';

const app = new Koa();
const router = new KoaRouter();
const authRouter = auth.router;

// 除了/public/之外的请求都需要经过JWT验证
app.use(jwt({ secret: 'shared-secret' }).unless({ path: [/^\/public/] }));

// 无论请求中是否存在Authorization header都保证执行
app.use(jwt({ secret: 'shared-secret', passthrough: true }));

// 所有走/api/打头的请求都需要经过JWT验证
router.use('/api', jwt({ secret: 'shared-secret' }), apiRouter.routes());
```
## 8 JWT的缺点和适用场景

[这篇文章](https://www.guonanjun.com/220.html)提出了一些自己观点，可以参考一下，它认为jwt的token方案最适合的场景是无状态的场景，若需要考虑token注销和某些token续签的场景，实际上就是给token加上了状态，这就类似于传统的方案了，但是相比传统方案的优势是，服务端可以不用长期保存用户状态，仅仅在一定时间段内保留一小部分状态即可。


## 9 总结

JWT的优点如下：

1. 因为JSON的通用性，所以JWT是可以跨语言支持的
2. 因为有了payload部分，所以JWT可以在自身存储一些其他业务逻辑所必要的非敏感信息
3. JWT体积很小，便于传输
4. 不需要在服务端保存回话信息，所以易于应用的扩展

在安全方面需要注意：

1. 不应该在payload部分存放敏感信息，因为该部分是非加密的
2. 保护好secret私钥，不应该暴露给客户端
3. 应该使用https协议

Koa2中的具体实现：

1. 签发token使用jsonwebtoken
2. 验证token使用koa-jwt中间件

## 参考

- [什么是 JWT -- JSON WEB TOKEN@简书](https://www.jianshu.com/p/576dbf44b2ae)
- J[SON Web Token - 在Web应用间安全地传递信息@回田园](http://blog.leapoahead.com/2015/09/06/understanding-jwt/)
- [八幅漫画理解使用JSON Web Token设计单点登录系统@回田园](http://blog.leapoahead.com/2015/09/07/user-authentication-with-jwt/?utm_source=tuicool&utm_medium=referral)
- [Node JWT/jsonwebtoken 使用与原理分析@简书](https://www.jianshu.com/p/a7882080c541)
- [讲真，别再使用JWT了！@掘金](https://juejin.im/entry/5993a030f265da24941202c2)
- [如何安全储存JWT之Cookie与Web Storage@掘金](https://juejin.im/post/5dbef6dc6fb9a0206e1fee35)
