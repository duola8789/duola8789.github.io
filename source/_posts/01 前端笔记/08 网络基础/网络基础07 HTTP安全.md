---
title: 网络基础07 HTTP安全
top: false
date: 2017-11-06 19:01:30
updated: 2019-09-20 09:29:07
tags:
- HTTP
- 安全
- XSS
- CSRF
categories: 网络基础
---

HTTP安全学习笔记。

<!-- more -->

## XSS跨站脚本攻击

### 现象

XSS（cross site scripting），即跨站点脚本工具，发生在用户的浏览器端。用户输入非法字符，向页面植入恶意代码，在目标网站上执行非原有的脚本。

如果网页会使用用户的输入渲染为HMTL内容渲染时就会执行被植入的恶意代码，利用这些恶意脚本，攻击者可获取用户的敏感信息如Cookie、SessionID等，进而危害数据安全。

XSS的本质是：恶意代码未经过滤，与网站正常的代码混在一起；浏览器无法分辨哪些脚本是可信的，导致恶意脚本被执行。

### 分类

一般会分为存储型XSS、反射型XSS和DOM型XSS

（1）存储型XSS

攻击者将恶意代码提交到目标网站的数据库中，用户打开网站时网站将恶意代码从数据库中取出，拼接在HTML中返回给浏览器，在浏览器中执行恶意代码

恶意代码窃取用户数据，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作

这种攻击常见于**带有用户保存数据的网站功能**，比如论坛发帖、商品评论、用户私信等。

（2）反射型XSS

攻击者构造出包含恶意代码的特殊的URL，用户打开URL后，网站服务端取出URL中的恶意代码，凭借在HTML中返回给浏览器，在浏览器中执行恶意代码

反射型XSS与存储型XSS的区别是，存储型XSS的恶意代码存在数据库，而反射型XSS的恶意代码存在URL中。

反射型XSS攻击常见于通通过URL传递参数的功能，比如网站搜索、跳转等。由于需要用户主动打开URL才能生效，所以攻击者往往会结合多种手段诱导用户点击。

（3）DOM型XSS

攻击者构造出包含恶意代码的特殊的URL，用户打开URL后，JavaScript取出URL中的恶意代码并执行。恶意代码窃取用户数据，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

DOM型XSS与前两种区别是，DOM型XSS的取出和执行恶意代码都有浏览器端完成，属于前端JavaScript自身的安全漏洞，而其他两种属于服务端的安全漏洞。

### 防护

XSS攻击有两大要素：

1. 攻击者提交恶意代码
2. 浏览器执行恶意代码

针对第一个要素，需要对用户输入进行过滤，但是并非完全可靠，因为在提交较短并不确定内容要输出哪里，对内容的转义有可能导致乱码和不确定性问题。

所以更合理的预防方法是针对第二个要素，通过**防止浏览器执行恶意代码来防范XSS**，分为两类：

1. 防止HTML中出现注入
2. 防止JavaScript执行恶意代码

总结起来，可以采取的预防方法有：

（1）一定要针对用户输入做相关的格式检查、过滤等操作，防止任何可能的前端注入

（2）对于不需要考虑SEO的内部系统、管理系统，使用纯前端渲染代替服务端渲染，把**代码与数据分开**

（3）对于需要服务端拼接HTML的情况，要对HTML中的敏感字符（比如`>`、`>`等）进行HTML转义（`&lt;`、`&gt;`）

（4）由于HTML转义是很复杂的，不同情况下要采取不同的准一规则，应当尽量避免自己写转义库，而是采取成熟的、业界通用的转义库

（5）对于跳转链接例如`<a href="xxx">`或者`location.href="xxx"`要检验内容，禁止以`javascript:`开头的链接

（6）遵循GET请求不改变服务状态的原则，改变状态的请求都是用POST或者PUT方法

（7）前端在使用`innerHTML`、`outerHTML`、`document.write`时要注意，不要把不可信数据作为HTML插入到页面

（8）使用Vue/React技术栈时不要使用`v-html`、`dangerouslySetInnerHTML`功能，在前端render阶段避免XSS隐患

（9）DOM中的内联事件监听器，比如`onclick`、`onload`、`onmouseover`、JavaScript中的`eval`、`setTimeout`、`setInterval`、`eval`都可以将字符串作为代码运行，如果不可信数据传递给上述API，很容易产生安全隐患，所以要尽量避免，

```HTML
<a href="javascript:alert(1)"></a>
<iframe src="javascript:alert(1)" />
<img src='x' onerror="alert(1)" />
<video src='x' onerror="alert(1)"></video>
<div onclick="alert(1)" onmouseover="alert(2)"><div>
```

（10）对于不受信任的输入，可以限制合理的长度，可以增加XSS攻击的难度

（11）某些敏感cookie应该设置`HTTP-only`，避免通过JavaScript读取cookie

（12）添加验证码，防止脚本冒充用户提交危险操作

（13）主动监测发现XSS漏洞，使用XSS攻击字符串和自动扫描工具寻找潜在的XSS漏洞

（14）JSONP的callback参数非常危险，风险性主要存在于会意外截断JS代码或者被添加恶意标签


## CSRF

Cookie往往用来存储用户的身份信息，恶意网站可以设法伪造带有正确Cookie的HTTP请求，这就是CSRF攻击。

### 现象

CSRF（Corss Site Request Forgery），有时也被缩写为XSRF，跨站伪造请求，伪造不是来自于用户的意愿，诱导用户发起请求。

通过CSRF，攻击者可以盗用用户的身份，以用户的名义发送恶意请求，比如以你的名义发送邮件、发送消息、盗取账号等。

CSRF攻击的思想：

![](http://image.oldzhou.cn/FvWo3AQwS_R7ZqKhSG3OxvFxPjG8)

要完成一次CSRF攻击，受害者必须依次完成两个步骤：

1. 登陆受信任网站A，并在本地生成cookie
2. 在不登出信任网站A的情况下（session有效），访问危险网站B

CSRF攻击是源于**HTTP使用cookie实现的隐式身份验证机制**，这种机制可以保证一个请求是来自于某个用户的浏览器，但是无法保证该请求是用户批准发送的。

### 攻击形式

同源策略并不能访问CSRF，因为CSRF会通过`<img>`、`<srcipt>`或者通过`action`提交表单的形式来让用户在不知情的情况下发送请求，这些形式是不受浏览器的跨域限制的，所以请求正常发送，也就会默认将目标网站的cookie带上，伪造了用户的身份。

GET类型的CSRF一般会利用`<img>`标签：

```HTML
<img src="http://bank.example/withdraw?amount=10000&for=hacker" > 
```

POST类型的CSRF通常会利用自动提交的表单：

```HTML
<form action="http://bank.example/withdraw" method=POST>
  <input type="hidden" name="account" value="xiaoming" />
  <input type="hidden" name="amount" value="10000" />
  <input type="hidden" name="for" value="hacker" />
</form>
<script> document.forms[0].submit(); </script> 
```

有时候也会有通链接形式`<a>`的攻击，这种形式需要诱导用户主动点击：

```HTML
<a href="http://test.com/csrf/withdraw.php?amount=1000&for=hacker" taget="_blank">
重磅消息！！
<a/>
```

### 防护

由于CSRF通常发生在第三方域名，并且CSRF不能获取到目标网站的Cookie，仅仅是利用，所以可以从两方面定制防护策略：

- 同源检测
- 附加信息认证

（1）同源检测

可以通过请求头Header中的`Origin`自断判断域名是否同源，但是它对IE11和302重定向都不支持

也可以使用Header中的`Referer`来确定，它记录了HTTP请求的来源地址，对于Ajax请求、图片、脚本等资源请求，Referer是发起请求的页面地址，对于页面跳转，它是打开页面历史记录的前一个页面地址。这种方式并不安全，因为攻击者可以修改或者隐藏的Referer：

```HTML
<img src="http://bank.example/withdraw?amount=10000&for=hacker" referrerpolicy="no-referrer"> 
```

所以同源检测的可靠性并不是很高，尤其是在Origin和Referer同时不存在的情况下，最好阻止这次网络请求，但是对外域请求如果一概阻止又会将来自搜索引擎链接的请求误伤。并且CSRF也可能来自同源页面（例如用户可以发布带链接和图片的评论等）

综上所述，同源请求是一个相对接单的防范方法，能够防范一定的CSRF攻击，但是对于安全性较高或者有较多用户输入内容的网站，就需要对关键接口进行进一步的防护。


（2）附加信息认证

**附加信息认证中心思想就是在客户端页面增加伪随机数**。

CSRF的另一个特征是，攻击者无法直接窃取到用户的信息（Cookie，Header，网站内容等），仅仅是冒用Cookie中的信息。

而CSRF攻击之所以能够成功，是因为服务器误把攻击者发送的请求当成了用户自己的请求。那么我们可以要求所有的用户请求都携带一个CSRF攻击者无法获取到的Token。服务器通过校验请求是否携带正确的Token，来把正常的请求和攻击的请求区分开，也可以防范CSRF的攻击。

（1） Cookie Hashing

这是比较简单的解决方案，所有的表单都包含同一个伪随机Hash值。在表单中添加一个隐藏的字段，值就是服务单构造的伪随机cookie，随表单一起发送。服务端对表单中的Hash值进行验证。

理论上攻击者无法获取第三方的cookie，那么也就无法伪造表单了，也就杜绝CSRF的攻击了。

Egg默认的预防CSRF的方式就是通过这种方式实现的，框架会在Cookie存放一个CSRF Token，放到query、body或者header中发送给服务端进行验证

**注意不能放在cookie中直接验证，那样就没有意义了，因为即便是CSRF伪造的请求，也会携带cookie。**

（2）验证码

这种方法也是最常见的方法之一，每次用户提交都需要在表单中填写一个图片上的伪随机字符串（或者实现某些特殊的交互），来让攻击者无法获取对应的字符串或者操作，从而伪造身份失败。

（3）JWT

可以通过JSON-WEB-Token来实现，在响应页面时将Token渲染到页面上，在提交表单的时候通过隐藏域提交上来

（4）`SameSite`属性

Google起草了一分草案，为`Set-Cookie`响应头新增`SameSite`属性，用来标明这个Cookie是个同站Coookie。同站Cookie只能作为第一方Cookie，不能作为第三方Cookie。

```BASH
Set-Cookie: foo=1; Samesite=Strict
```

设置`Samesite`为`strict`为严格模式，表明这个cookie任何情况下都不能作为第三方cookie，没有例外。假设我们为`b.com`设置了上面的cookie，那么在`a.com`发起对`b.com`的跨域请求，`foo`这个cookie都不会包含在cookie请求头中。

举个实际的例子就是，假如淘宝网站用来识别用户登录与否的Cookie被设置成了`Samesite=Strict`，那么用户从百度搜索页面甚至天猫页面的链接点击进入淘宝后，淘宝都不会是登录状态，因为淘宝的服务器不会接受到那个Cookie，其它网站发起的对淘宝的任意请求都不会带上那个Cookie。

```BASH
Set-Cookie: foo=1; Samesite=Strict
Set-Cookie: bar=2; Samesite=Lax
Set-Cookie: baz=3
```

设置`Samesite`为`lax`为宽松模式，假设这个请求改变了当前页面（或者打开了新页面）且同时是一个GET请求，那么这个cookie可以作为第三方cookie

加入`b.com`设置了上面的两个cookie，当从`a.com`点击链接进入`b.com`时，`foo`这个cookie不会包含在请求头中，但是`bar`和`baz`会。

也就是说用户在不同网站之间通过链接跳转不会受影响，到那时如果在`a.com`发起对`b.com`发起网络请求，或者页面跳转是通过表单的POST请求触发的，那么`bar`也不会包含在请求头中。

如果将`Samesite`被设置为`Strict`，浏览器在任何网络请求中都不会携带Cookie，可以完全杜绝CSRF攻击，但是新打开标签或者跳转到子域名的网站都需要重新登录，用户体验很差。

如果将`Samesite`被设置为`Lax`可以保障链接跳转是的用户体验，但是安全性也比较低。

此外`Samesite`的兼容性也很差，并且不支持子域，也就是说`user.a.com`不能使用`a.com`下的Samesite Cookie，如果网站有多个子域名，不能使用Samesite Cookie在主域名存储用户登录信息。

所以Samesite Cookie是一个可能替代同源验证的方案，但还不成熟，应用场景有待观望。

关于CSRF攻击美团技术团队的[这篇文章](https://juejin.im/post/5bc009996fb9a05d0a055192)写的非常好，可以仔细阅读学习。

## HTTP劫持（中间人攻击）

当我们使用HTTP请求请求一个网站页面的时候，营商可在用户发起请求时直接跳转到某个广告，或者直接改变搜索结果插入自家的广告，让客户端（通常是浏览器）展示“错误”的数据，通常是一些弹窗，宣传性广告或者直接显示某网站的内容。如果劫持代码出现了BUG ，则直接让用户无法使用，出现白屏。

数据泄露、请求劫持、内容篡改等等问题，核心原因就在于HTTP是全裸式的明文请求，域名、路径和参数都被中间人们看得一清二楚。

最直接的解决方法就是**使用HTTPS协议代替HTTP协议**。HTTPS做的就是给请求加密，让其对用户更加安全。对于自身而言除了保障用户利益外，还可避免本属于自己的流量被挟持，以保护自身利益。

## DNS劫持

DNS劫持就是通过劫持了DNS服务器，通过某些手段取得某域名的解析记录控制权，进而修改此域名的解析结果，导致对该域名的访问由原IP地址转入到修改后的指定IP，其结果就是对特定的网址不能访问或访问的是假网址，从而实现窃取资料或者破坏原有正常服务的目的

这种攻击作为网站开发者并没有什么好的解决方式，可能的解决方有两个：

1. 使用大公司提供的CDN服务，减少被坚持的概率
2. 被劫持后向工信部投诉解决，或者包茎

## 点击劫持

点击劫持是一种视觉欺骗，将我们的网页放置到和原页面相同大小的`iframe`中，用透明的`iframe`或者图片覆盖页面，诱导用户点击，访问其他页面。

可以在前端预防，判断当前页面是否被嵌套了在`iframe`中，如果被嵌套了的话则重定向外层页面到我们的正常页面。

```JS
if (top.location !== location){
  top.location = self.location
}
```
但是前端防护很容易被绕过，更可靠的方法是通过设置HTT响应头信息的`X-FRAME-OPTION`属性来进行防护，用来给浏览器指示允许一个页面可否在`<frame>`，`<iframe>`，`<embed>`或者`<object>`中展现。站点可以通过确保网站没有被嵌入到别人的站点里面，从而避免点击劫持攻击。

可取的属性值有`DENY`/`SAMEORIGIN`/`ALLOWFROM`

- `DENY`：表示该页面不允许 frame中展示，即便是在相同域名的页面中嵌套也不允许。
- `SAMEORIGIN`：表示该页面可以在相同域名页面的frame中展示。
- `ALLOW-FROM`： 表示该页面可以在指定来源的frame中展示。


## 参考

- [前端安全系列（一）：如何防止XSS攻击？@美团技术团队](https://tech.meituan.com/2018/09/27/fe-security.html)
- [安全威胁 CSRF 的防范@Egg](https://eggjs.org/zh-cn/core/security.html#安全威胁-csrf-的防范)
- [浅谈CSRF攻击方式@hyddd](https://www.cnblogs.com/hyddd/archive/2009/04/09/1432744.html)
- [JavaScript防http劫持与XSS@CSDN](https://blog.csdn.net/z69183787/article/details/52496188)
- [浅析点击劫持攻击@FREEBUF](https://www.freebuf.com/articles/web/67843.html)
- [安全@Egg](https://eggjs.org/zh-cn/core/security.html)
- [X-Frame-Options 响应头@MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/X-Frame-Options)
- [前端安全系列之二：如何防止CSRF攻击？@掘金](https://juejin.im/post/5bc009996fb9a05d0a055192)
