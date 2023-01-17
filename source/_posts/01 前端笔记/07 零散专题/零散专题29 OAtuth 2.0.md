---
title: 零散专题29 OAuth 2.0
top: false
date: 2019-04-28 15:37:17
updated: 2019-04-28 15:37:17
tags:
- CSRF
- OAuth
- 第三方登录
categories: 其他
---

OAuth是一种授权机制，数据的所有者告诉系统，同意授权第三方应用进入系统，获取这些数据。系统从而产生一个短期的进入令牌（token），用来代替密码，供第三方应用使用。

<!-- more -->

## 定义

OAuth是一种授权机制，数据的所有者告诉系统，同意授权第三方应用进入系统，获取这些数据。系统从而产生一个短期的进入令牌（token），用来代替密码，供第三方应用使用。

## 例子

举个例子，京东App要求访问我的微信头像和昵称，这里面，我是数据的所有者，而微信就是服务提供商，也就是上面的系统，而京东就是第三方的客户端。

当京东要求获取我的微信数据时，我不会将我的微信账号和密码直接告诉京东，然后让京东自己去登陆我的微信账号去获取对应的数据，因为这样做会有以下的缺点：

1. 京东也许会保存我的密码，这样很不安全
2. 京东通过密码能够获取我在微信的全部数据，我没有办法限制京东能获得哪些数据、不能获得哪些数据
3. 我除了修改密码之外，没有能够取消京东获取我的微信数据的能力
4. 只要有一个京东这样的、知晓我的密码的第三方APP被破解，我的微信密码就会泄露

所以京东需要通过OAuth这种授权机制来获得我的微信数据。思路是这样的：

1. 京东在它的APP里选择以微信账号登陆京东APP；
2. 京东客户端会跳转到微信设置的认证服务器，向我询问是否允许获取我的微信相关资料；
3. 当我同意之后，微信的认证服务器就会向京东APP颁发登陆令牌（token），这个token与用户密码不同，并且在登陆的时候可以指定token的权限范围有效期；
4. 京东APP获得令牌后，就可以向微信的资源服务器来申请获取用户的微信资料，微信的资源服务器根据令牌的权限和有效期向京东APP开放我的微信头像和昵称。

## 授权模式

OAuth的核心就是向第三方应用颁发令牌，OAuth 2.0定义了四种授权方式来让客户端得到用户的授权：

1. 授权码（authorization-code）
2. 隐藏式（implicit）
3. 密码式（password）
4. 客户端凭证（client credentials）

以上这些授权方式，第三方应用在申请令牌之前，都必须先到系统备案，说明自己的身份，然后会拿到两个身份识别码：客户端ID（client ID）和客户端密钥（client secret）。

## 授权码模式

授权码（authorization-code）模式，指的是第三方应用先申请**授权码**，然后用该授权码获取令牌。

这种方式是最常用的，也是安全性最高的，适用于后后端的Web应用。授权码通过前端传送，令牌存储在后端，所有与资源服务器的通信都在后端完成。这样的前后端分离可以避免令牌泄露。

仍旧以京东和微信举例

（1）第一步，当用户点击京东APP以微信登陆的链接后，就会跳转到微信的认证服务器，下面就是一个示意的跳转链接：

```TEXT
https://weixin.com/oauth/authorize?
  response_type=code&
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=read
```
`weixin.com/oauth/authorize`就是微信认证服务器的URL，`response_type`参数表示要求返回授权码（`code`），`client_id`参数用来表明客户端的ID，`redirect_uri`是认证服务器接受或拒接请求后的跳转网址，`scope`参数表示请求的授权范围

（2）第二步，用户跳转到认证服务器的页面后，会要求用户登录，然后询问是否同意给京东APP授权。用户表示同意后，认证服务器就会调回`redirect_uri`参数指定的网址，在跳转时会以URL中的`query`参数的形式传回一个授权码（`code`）

```TEXT
https://jd.com/callback?code=AUTHORIZATION_CODE
```
上面的URL中，`code`参数对应的值就是授权码

（3）第三步，京东拿到授权码后，就可以在**后端**向微信认证服务器请求令牌（token）

```TEXT
https://weixin.com/oauth/token?
 client_id=CLIENT_ID&
 client_secret=CLIENT_SECRET&
 grant_type=authorization_code&
 code=AUTHORIZATION_CODE&
 redirect_uri=CALLBACK_URL
```
上面URL中，`client_id`和`client_secert`参数用来让认证服务器确认京东APP的身份（`client_secret`是保密的，所以只能在后端发送请求），`grant_type`参数的值是`AUTHORIZATION_CODE`，表示采用的授权方式是授权码，`code`参数是上一步拿到的授权码，`redirect_uir`参数是令牌颁发后的回调地址。

（4）第四步，微信认证服务器收到请求后，核对信息后就会办法令牌，具体的做法是向`redirect_uri`指定的网址发送一段JSON数据：

```TEXT
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{    
  "access_token":"ACCESS_TOKEN",
  "token_type":"bearer",
  "expires_in":2592000,
  "refresh_token":"REFRESH_TOKEN",
  "scope":"read",
  "uid":100101,
  "info":{...}
}
```
上面JSON对象中，`access_token`就是令牌，京东在后端服务器拿到了。整个过程如下图所示：

![](http://image.oldzhou.cn/bg2019040905.jpg)

## 第二种方式：隐藏式

有一些Web应用是纯前端应用，没有后端，这时就不能使用第一种方式了，需要将令牌存储在前端。这种方式成为隐藏式（implicit），**允许直接向前端颁发令牌**，而没有授权码这个中间步骤。

（1）第一步，A网站提供一个连接，要求用户跳转到B网站，授权用户数据给A网站使用

```TEXT
https://b.com/oauth/authorize?
  response_type=token&
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=read
```
上面的URL中，`response_type`参数为`token`，表示要求直接返回令牌

（2）第二步，用户跳转到上面的URL后，登陆，然后同意向A网站授权。这时B网站就会跳回`redirect_uri`参数指定的跳转网址，并且把令牌作为URL的Hash参数，传递给A网站。

```TEXT
https://a.com/callback#token=ACCESS_TOKEN
```

上面的`token`参数就是令牌，A网站直接在前端拿到令牌。

注意，令牌在URL中是使用了Hash参数，也就是锚点位置储存，而不是使用查询字符串。这是因为当跳转网址时HTTP协议时，存在“中间人攻击”的风险，而浏览器跳转时Hash参数不会发送到服务器，减少了令牌泄露的风险。

下面于OAuth的安全设置的内容笔记，专门讨论这一部分内容。

这个过程如下所示：

![](http://image.oldzhou.cn/bg2019040906.jpg)

这种方式把令牌直接传给前端，是很不安全的。因此只适用于安全性不搞的场景，且令牌的有效期必须非常短，通常是会话期间有效，浏览器关了令牌就失效。

## 第三种方式：密码式

如果高度信任某个应用，允许用户把用户名和密码直接告诉该应用，该应用使用密码来申请令牌。这种方式称为密码是（password）

（1）第一步，A网站要求用户提供B网站的用户名和密码，拿到以后，A就直接向B请求令牌

```TEXT
https://oauth.b.com/token?
  grant_type=password&
  username=USERNAME&
  password=PASSWORD&
  client_id=CLIENT_ID
```

上面的URL中，`grant_type`参数是`password`，代表授权方式是密码式，`username`和`password`就是B的用户名和密码。

（2）第二步，B网站验证身份通过后，直接给出令牌，这里给出令牌的方式不再是通过URL的跳转，而是将令牌放在JSON数据中，作为HTTP响应，返回给A网站。

这种方式需要用户给出用户名和密码，风险很大，因此只适用于其他授权方式都无法采用、并且对应用高度信任的情况下。

## 第四种方式：凭证式

凭证式（client credentials）适用于没有前端的命令行应用，即在命令行下请求令牌。

（1）第一步，A应用在命令行向B发出请求

```TEXT
https://oauth.b.com/token?
  grant_type=client_credentials&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET
```
上面的URL中，`grant_type`参数`client_credentials`表示采用凭证式的授权方式，`client_id`和`client_sercret`用来表示A应用的身份。

（2）第二步，B网站验证通过后，直接返回令牌。

这种方式给出的令牌，是针对第三方应用的，而不是针对用户的，有可能多个用户共享一个令牌。

## 令牌的使用

拿到令牌后，A网站就可以向B网站提供数据资料的API请求数据了。

此时，每个发到API的请求，都必须带有令牌，具体做法是在请求的头信息加上`Authoization`字段，令牌就在这个字段里。

```BASH
curl -H "Authorization: Bearer ACCESS_TOKEN" \
"https://api.b.com"
```
上面的命令中，`ACCESS_TOKEN`就是拿到的令牌。

> `curl`命令是一个命令行工具，支持多种协议，可以再命令行发出网络请求，然后得到和提取数据，显示在标准输出（stdout）上面。`-H`参数用来自定义头信息发送给服务器。

## 更新令牌

令牌的有效期到了，如果让用户重新走一遍上面的获取流程，申请新的令牌，体验会很差。OAuth 2.0允许用户自动更新令牌。

具体做法是，B网站颁发令牌的时候，一次性颁发两个令牌，一个用来获取数据，一个用于获取新的令牌。令牌到期前，用户使用`refresh token`发送请求，更新令牌

```TEXT
https://b.com/oauth/token?
  grant_type=refresh_token&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET&
  refresh_token=REFRESH_TOKEN
```

上面的URL中，`grant_type`参数为`refresh_token`表示要求更新令牌，`refresh_token`参数与就是用于更新令牌的令牌。

B网站验证通过后，就会颁发新的令牌。

## 例子

按照阮一峰的例子，做一个Demo，通过OAuth，以Github进行第三方登录，获取API数据。代码[在这里](https://github.com/duola8789/OAuth-learning)。

登陆的过程就是OAuth授权，实际上和前面以微信账号登陆京东APP的例子是一样的。这里面我们的网站是A，允许使用Github账号登陆。

流程如下：

1. 用户点击A网站“使用Gihutb登陆”的链接，页面跳转到Github的授权页面；
2. Github要求用户登陆，然后循环是否授权使用当前Github账号登陆A网站
3. 用户同意登陆，Github会重定向回A网站在最开始的链接中指明的地址，同时发回一个授权码。
4. A网站使用授权码，向Github请求令牌。这个过程必须在后端完成。
5. Github返回令牌。
6. A网站使用令牌，向Github请求用户数据。

具体过程：

（1）应用要求OAuth授权，必须在对方网站登记，换取clientID和clientSecret，证明自己的身份。Github的登记地址是这个[网址](https://github.com/settings/applications/new)。

![bg2019042102.jpg](http://image.oldzhou.cn/bg2019042102.jpg)

注意，应用的callback URL必须是Homepage下的子域名。

（2）新建了一个`index.html`作为项目的首页

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>OAuth 2.0 Demo</title>
</head>
<body>
<a id="login">Login with Github</a>
</body>

<script>
  const client_id = 'f38d5d8c4b1bb8259159';
  const authorize_uri = 'https://github.com/login/oauth/authorize';
  const redirect_uri = 'http://localhost:8080/oauth/redirect';
  
  const link = document.querySelector('#login');
  link.href = `${authorize_uri}?client_id=${client_id}&redirect_uri=${redirect_uri}`
</script>
</html>
```
在HTML代码中，跳转链接设置为`https://github.com/login/oauth/authorize?client_id=f38d5d8c4b1bb8259159&redirect_uri=http://localhost:8080/oauth/redirect`

其中`https://github.com/login/oauth/authorize`就是Github用来获取授权码的地址，具体参考它的[文档](https://developer.github.com/apps/building-oauth-apps/authorizing-oauth-apps/)。

在`index.js`中使用koa-static来托管访问静态资源，监听8080端口：

```JS
const Koa = require('koa');
const app = new Koa();

// Koa静态文件服务的目录
app.use(staticServer(path.resolve('public')));

app.listen(8080, () => {
  console.log('Koa is listening in 8080');
});
```
（3）当点击链接，页面会跳转到Github授权的页面

![WX20190428-135646.png](http://image.oldzhou.cn/WX20190428-135646.png)

（4）点击授权后，页面跳转到之前指定的Callback URI，也就是`http://localhost:8080/oauth/redirect`，所以需要在服务端代码对这个路由进行处理：

```JS
const KoaRouter = require('koa-router');
const router = new KoaRouter();

router.get('/oauth/redirect', oauthController);

// 加载路由中间件
app.use(router.routes());
```
当访问`/oauth/redirect`会执行`oauthController`的逻辑：

```JS
const oauthController = async ctx => {
  // 获取授权码
  const { code } = ctx.request.query;
  console.log('authorization code: ', code);

  try {
    // 使用授权码 + clientID + clientSecret 获取 token
    const tokenRes = await axios({
      method: 'post',
      url: `https://github.com/login/oauth/access_token?client_id=${config.clientID}&client_secret=${config.clientSecret}&code=${code}`,
      headers: {
        accept: 'application/json'
      }
    });

    const token = tokenRes.data.access_token;
    console.log('access token: ', token);

    // 使用令牌获取数据
    const res = await axios.get('https://api.github.com/user', {
      headers: {
        accept: 'application/json',
        'Authorization': `token ${token}`
      }
    });
    console.log('result: ', res);

    // 拿到用户名，渲染到页面
    const { name } = res.data;
    ctx.response.redirect(`/welcome.html?name=${encodeURIComponent(name)}`);
  } catch (e) {
    console.log('There is something wrong when getting token: ');
  }
};
```
在`oauthController`中，首先我们从`request`的查询参数中获取授权码，然后使用授权码 + clientID + clientSecret从Github获取 token，获取token之后再使用相应的API获取用户名，这时需要将上面获得的token作为`Authoriztion`字段，加到HTTP请求的头信息里面。

拿到用户名，将页面重定向到`welcome.html`，在这个页面会将URL中的用户名取出来并显示在页面。注意由于用户名可能是中文，需要在`redirect`中进行安全编码，否则会发生问题。

## 关于OAuth的安全性问题

### 回调域名

在做微信登陆时，到了获取授权码code这一步，如果给微信服务器传的`redirect_uri`不是申请appid时输入的域名，微信会立刻返回`redirect_uri错误`的提示。

为什么会这样？因为如果微信不校验`redirect_uri`，会导致中间人攻击，攻击者伪造受害者的身份，使用受害者的微信账号，登陆目标网站。

假设在申请appid时的域名是`a.com`，申请的appid是`123`，假设微信获取code地址是`https://wx.om/code`，这样获取code的完整URL是：

```TEXT
https://wx.com/code?
  redirect_uri=http://a.com&
  appid=11111&
  response_type=code&
  scope=userinfo
```
把这个地址发送给任何一个微信联系人，他们都可以通过自己的微信账号获取code，微信会带着code参数回调到`http://a.com?code=455`，然后`a.com`再通过code换取access token，用户就可以使用微信账户登录`a.com`。

但是如果微信不验证`redirect_uri`是否是`a.com`，攻击者将`redirect_uri`换为他的网站（假设为`hack.com`），受害人访问此链接，确认登录，微信生成code之后通过回调的方式将code传给了攻击者的网站`http://hack.com?code=151`，拿到code之后，攻击者再将域名切换为`http://a.com?code=151`，而这时`a.com`是无法分辨这是微信直接回调还是有人从中动了手脚的地址，无差别的获取code对应的access token，攻击者以受害者身份登陆成功。

对于Implicit方式，更需要校验`redirect_uri`，因为是一步到位的获取access token。

所以作为OAuth2.0 Server，`redirect_uri`的域名限制是一定要做的。而作为调用者，到不必这么担心，目前大部分遵循OAuth2.0的服务都不会犯这个错误。

对于调用者开发过程中，微信后台对于`redirect_uri`域名设置的限制，对于本地地址是无法正常登陆了。解决方法就是更改Host文件，将`redirect_uri`的域名直接指向内网就行了：

```BASH
# /etc/hosts
127.0.0.1 a.com
```
## code与secret

前面提到了，在使用OAuth之前，需要到提供的服务的服务商进行注册，获取appid（clientID）和secert（client Secret），appid的目的是为了告诉身份认证服务器我是`a.com`，而secret是为了告诉任务服务器，我真的是`a.com`。这个secert相当于客户端与认证服务器之间的信物，这个信物是不能暴露给用户的，所以它的传递只能通过服务器进行传递。能够被暴露的只有appid。

如果没有secert，当出现DNS污染，本该发往`a.com`的code被发往了`hack.com`，这时候攻击者就会直接使用code换取token了。

OAuth 2.0设计时的一个目标是，让不支持HTTPS的网站也能够安全使用。所以code才是必须的。如果没有code，直接获取access token，流程如下：

1. 用户浏览器访问`a.com`，跳转到微信OAuth服务器获取access token
2. 用户在微信的网页上登陆成功，并确认授权`a.com`使用微信账号登陆，微信服务器跳转到`redirect_uri`并且带上access_token参数
3. 用户浏览器访问带access token的连接，完成登陆。

如果`a.com`不支持HTTPS，那么在最后一步，access token就完全暴露在浏览器和`a.com`服务器之间的线路中。

如果`a.com`支持HTTPS，那么理论上来说，可以省略获取code这一步的。

但是如果`a.com`不支持HTTPS，那么使用了code，code被暴露的后果是好于access token被暴露的。这是因为OAuth 2.0协议对此规定：

1. code只能使用一次
2. 如果攻击者比正常用户先用了code，当用户第二次使用code时，之前通过此code获取的access token将被撤回。

所以当code被泄露时，攻击最多让正常用户有点困扰，可能登陆意外失败，或者明明看起来登陆成功但还是获取不到用户信息的情况（access token已失效），攻击者拿不到数据。

## state参数

在获取code时，一般服务器会返回一个state参数，一般来说没什么用，也可以为空，但是OAuth2.0文档标注的是Recommended，在什么时候使用呢？

它在防御CSRF时是非常有用的。进行CSRF攻击，攻击者：

1. 申请一个的专门用于攻击的账号
2. 走正常流程，跳转到微信上登陆此账号
3. 登陆成功后，微信带着code跳转回`a.com`，这个时候，攻击者拦截自己的请求不再继续进行，而是将code的链接发送给受害者，棋牌受害者点击
4. 受害人点击后，继续攻击者登陆流程，不知不觉登陆了攻击者的账号

而state参数如果利用起来，作为**CSRF Token**，就能避免此时的发生：

1. 攻击者依旧获取code并打算骗受害者点击
2. 受害者点击链接，但是服务器（`a.com`）分配给受害者的设备的state值和链接里面的state值不一样，服务器（`a.com`）直接返回验证state失败

state或者CSRF Token这种与设备绑定的随机字符串，复杂一点，攻击者就无计可施。


**设置一个让攻击者猜不到的、跟设备（或者浏览器）绑定的state或者CSRF Token值，就是解决CSRF的关键。**

### Implicit授权模式

Implicit授权模式一步到位，直接返回了access token，它的最重要也是最巧妙的设计是，登陆成功后身份认证服务器跳转回来带的参数都是放在`#`后面的，而不是查询参数。

这是因为，如果在没有使用HTTPS的线路上通信时，access token很容易被偷走，但是如果access token放在`#`后面，浏览器发起请求时，`#`后面的内容不会碎请求发送到服务器。这一样就可以防止中间人共计而只让设备用用access token。

同样，access token是一定不能存放在cookie这种可能被中间人发现的地方（除非使用HTTPS）。为了做到粳稻的安全性，access token最好连local/session storage都别放。这样理解，Implicit也就只适合SPA了，SPA不刷新页面可以让access token一直在内存里，直到关掉页面。


## 参考

- [理解OAuth 2.0@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)
- [OAuth 2.0 的一个简单解释@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2019/04/oauth_design.html)
- [OAuth 2.0 的四种方式@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html)
- [GitHub OAuth 示例教程@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2019/04/github-oauth.html)
- [关于 OAuth2.0 安全性你应该要知道的一些事@Chrisyue's Blog](https://www.chrisyue.com/security-issue-about-oauth-2-0-you-should-know.html)
