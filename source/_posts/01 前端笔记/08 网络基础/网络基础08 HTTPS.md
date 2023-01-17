---
title: 网络基础08 HTTPS
top: false
date: 2017-12-08 11:54:30
updated: 2020-06-09 15:01:16
tags:
- 对称加密
- 非对称加密
- HTTPS
- CSP
- 混合内容
categories: 网络基础
---

HTTPS学习笔记。

<!-- more -->

# SSL/TLS协议

HTTPS（HTTP Secure）是HTTP加上加密、认证和完整性保护。不是一种新的协议，只是在HTTP通信接口部分用了SSL和TLS协议代替，也就是说在HTTP与TCP之间增加了SSL，HTTP与SSL通信，再由SSL与TCP通信。

TLS是SSL协议的升级版， 目前应用最广泛的是TLS1.0和SSL3.0。

不使用SSL/TLS的HTTP通信，就是不加密的通信。所有信息明文传播，带来了三大风险：

 1. 窃听风险（eavesdropping）：第三方可以获知通信内容。
 2. 篡改风险（tampering）：第三方可以修改通信内容。
 3. 冒充风险（pretending）：第三方可以冒充他人身份参与通信。

SSL/TLS协议是为了解决这三大风险而设计的，希望达到：

1. 所有信息都是加密传播，第三方无法窃听。
2. 具有校验机制，一旦被篡改，通信双方会立刻发现。
3. 配备身份证书，防止身份被冒充。

SSL/TLS协议的基本思路是采用了公开密钥加密法，客户端首先向服务器索要公钥，然后用公钥加密信息，服务器收到密文后，用自己的私钥解密。

这里面有两个问题：

**（1）如何保证公钥不被篡改**

将公钥放在CA机构颁发的数字证书中，客户端收到服务器的公钥时会使CA机构内置在浏览器中的公钥进行验证。因此只要数字证书是可信的，公钥就是可信的

**（2）公钥加密计算量太大，如何减少耗用时间**

在建立连接阶段使用公开密钥加密计算，生成一个“对话密钥”，在随后的通话期间信息由“对话密钥”进行共享密钥加密。共享密钥加密是对称加密，运算速度很快，而服务器公钥只用于加密“对话密钥”本身，这样就减少了加密运算的时间。

# HTTPS的工作模式

非对称加密在性能上不如对称加密，所以HTTPS采取了对称加密和非对称加密相结合的工作模式：**公钥私钥主要用于传输对称加密的秘钥，而真正的双方大数据量的通信都是通过对称加密进行的**。

非对称加密需要通过证书和权威机构来验证公钥的合法性。

# HTTPS的握手机制

HTTPS的握手相当于SSL/TLS协议的握手和TCP协议的握手，这里主要介绍的是SSL/TLS的握手。

SSL/TLS的握手阶段设涉及四次通信，这四次通信都是明文的：

（1）客户端发出请求（ClientHello）

客户端先想服务器发出加密通信的请求，提供一些基本信息，比如支持的TLS/SSL版本、用于生成“对话密钥”的**随机数**，支持的加密方法等。

（2）服务器回应（ServerHello）

服务器收到客户端请求后，向客户端发出回应，包含用于协商建立SSL通信的信息，比如确认使用的加密通信加密版本，**服务器证书（包含服务器的公钥）**、确认使用的加密方法、服务器生成的用来生成“对话密钥“的**随机数**

此外，如果服务器需要确认客户端身份，就会再包含一项请求，要求客户端提供“客户端证书”。比如金融机构往往只允许认证客户连入自己的网络，就会想正式客户提供USB密钥，里面包含了一张客户端证书。

（3）客户端回应

客户端收到服务器的回应后，首先验证服务器证书，如果证书不是可信机构颁布，或者证书的域名与实际域名不一致，或者证书已经过期，就会想访问者显示警告，由其选择是否还要继续通信。

如果证书没有问题，客户端就会从证书中取出服务器的公钥，然后向服务器发送下面三项信息：

```BASH
1. 一个随机数（pre-master-key)。这个随机数会用服务器公钥加密
2. 编码改变通知，表示随后的信息都将用双方商定的加密方法和密钥发送
3. 客户端握手结束通知，表示客户端的握手阶段结束。这一项同时也是前面发送的所有内容的hash值，用来供服务器校验
```

到此为止，握手阶段已经有了三个随机数，随后，双方就会用实现商定的加密方法，用着三个随机数各自生成用于本次会话加密的同一把“会话密钥”。为什么一定要用三个随机数呢？

> SSL协议中的证书是静态的，因此十分有必要引入一种随机因素来保证每次会话协商出来的密钥的随机性。所以不管是客户端还是服务器，都需要随机数，这样每次会话生成的密钥才不会每次都一样。
>
> 对于RSA密钥交换算法来说，pre-master-key本身是一个随机数，再加上Hello消息中的随机数，三个随机数通过密钥到初七最终导出一个对称密钥。
>
> SSL协议不信任每个主机都能产生完全随机的随机数，如果随机数不随机，那么对话密钥就有可能被猜出来。而三个随机数一同生成的密钥就不容易被猜出来了，一个伪随机可能完全不随机，可是三个伪随机就十分接近随机了，每增加一个自由度，随机性增加的可不是一。

此外，如果在第二步服务器要求客户端提供证书，那么客户端在这一步还会发送证书及相关信息。

（4）服务器的最后回应

服务器收到客户端的第三个随机数pre-master-key后，计算生成本次会话用的“会话密钥”，然后向客户端发最后发送下面信息

```BASH
1. 编码改变通知，表示表示随后的信息都将用双方商定的加密方法和密钥发送
2. 服务器握手结束通知，表示服务器的握手阶段已经结束。这一项同时也是前面发送的所有内容的hash值，用来供客户端校验。
```

至此，整个握手阶段全部结束，接下来客户端与服务器进入加密通信，就是完全使用HTTP协议，只不过使用“对话密钥”加密内容。

![](http://image.oldzhou.cn/FjSUtFehKucfBIg_sI3TOGl4Zl4J)

# 数字证书

公开密钥加密还有一个问题：每个服务器都可以生成自己的私钥和公钥，客户端并没有办法确定自己接收到到的公钥是否真的是目标网站的公钥，还是被替换过。

即：**如何证明服务器下发的公钥没有被替换？**

现在通用的解决方法就是通过数字证书认证机构（Certificate Authority，CA）颁发的公开密钥证书来解决这个问题。

CA会为提交申请的服务器的公钥做认证，CA利用自己的私钥，对服务器的公钥和一些相关信息一起加密，生成数字证书（Digital Certificate）。这样服务器在建立HTTPS连接时不再直接下发公钥，而是下发包含着服务器公钥（经过CA私钥加密过的）的数字证书。

浏览器会将CA的公开密钥事先内置到浏览器当中（避免了这个证书在传递中出现的问题）。所以客户端收到信息后，会利用CA的公钥解密数字证书，拿到服务器的真实的公钥。

拿到真实的公钥后，客户端（浏览器）会根据内置的“证书管理器”来验证公钥是否真的属于目标网站

![](http://image.oldzhou.cn/FuCbT4h2bs8S_pH6cVtuyiCQFCTz)

客户端的“证书管理器”有`受信任的根证书办法机构”列表，客户端会根据这个列表，查看解开数字证书的公钥是否在列表内。

如果证书不是可信机构颁布，或者证书的域名与实际域名不一致，或者证书已经过期，就会想访问者显示警告，由其选择是否还要继续通信。

# SSL延迟

```TEXT
HTTP耗时 = TCP握手(3次)

HTTPS耗时 = TCP握手(3次) + SSL握手
```

HTTPS肯定比HTTP耗时，这就叫SSL延迟。

从运行结果可以看到，SSL握手的耗时大概是TCP握手的三倍。也就是说，在建立连接的阶段，HTTPS连接比HTTP连接要长3倍的时间，具体数字取决于CPU的快慢和网络状况。

所以，如果是对安全性要求不高的场合，为了提高网页性能，建议不要采用保密强度很高的数字证书。一般场合下，1024位的证书已经足够了，2048位和4096位的证书将进一步延长SSL握手的耗时。

# 升级HTTPS

## （1）获取证书

升级到HTTPS协议的第一步，就是要获得一张证书。

证书是一个二进制文件，里面包含经过认证的网站公钥和一些元数据，要从经销商购买。

- GoGetSSL.com
- SSLs.com
- SSLmate.com

证书有很多类型，首先分为三种认证级别。

- 域名认证（Domain Validation）：最低级别认证，可以确认申请人拥有这个域名。对于这种证书，浏览器会在地址栏显示一把锁。
- 公司认证（Company Validation）：确认域名所有人是哪一家公司，证书里面会包含公司信息。
- 扩展认证（Extended Validation）：最高级别的认证，浏览器地址栏会显示公司名。

还分为三种覆盖范围。

- 单域名证书：只能用于单一域名，`foo.com`的证书不能用于`www.foo.com`
- 通配符证书：可以用于某个域名及其所有一级子域名，比如`*.foo.com`的证书可以用于`foo.com`，也可以用于`www.foo.com`
- 多域名证书：可以用于多个域名，比如`foo.com`和`bar.com`

认证级别越高、覆盖范围越广的证书，价格越贵。还有一个免费证书的选择。为了推广HTTPS协议，电子前哨基金会EFF成立了`Let's Encrypt`，提供免费证书（教程和工具）。

拿到证书以后，可以用SSL Certificate Check检查一下，信息是否正确。

## （2）安装证书

证书可以放在`/etc/ssl`目录（Linux 系统），然后根据你使用的Web服务器进行配置。

## （3）修改链接（前端主要的工作都在此）

网页加载的HTTP资源，要全部改成HTTPS链接。**因为加密网页内如果有非加密的资源，浏览器是不会加载那些资源的。**

```HTML
<script src="http://foo.com/jquery.js"></script>
```
上面这行加载命令，有两种改法：

```HTML
<!-- 改法一 -->
<script src="https://foo.com/jquery.js"></script>

<!-- 改法二 -->
<script src="//foo.com/jquery.js"></script>
```
其中，改法二会根据当前网页的协议，加载相同协议的外部资源，更灵活一些。

另外，如果页面头部用到了`rel="canonical"`，也要改成HTTPS网址。

```HTML
<link rel="canonical" href="https://foo.com/bar.html" />
```
## （4）`301`重定向

下一步，修改Web服务器的配置文件，使用`301`重定向，将HTTP协议的访问导向HTTPS协议。

```NGINX
server {
  listen 80;
  server_name domain.com www.domain.com;
  return 301 https://domain.com$request_uri;
}
```

# HTTPS的七个误解

（1）HTTPS无法缓存

许多人以为，出于安全考虑，浏览器不会在本地保存HTTPS缓存。实际上，只要在HTTP头中使用特定命令，HTTPS是可以缓存的。

（2）SSL证书很贵

如果你在网上搜一下，就会发现很多便宜的SSL证书，大概10美元一年，这和一个`.com`域名的年费差不多。而且事实上，还能找到免费的SSL证书。

（3）HTTPS站点必须有独享的IP地址

每个IP地址只能安装一张SSL证书，这是毫无疑问的。但是，如果你使用子域名通配符SSL证书（wildcard SSL certificate，价格大约是每年125美元），就能在一个IP地址上部署多个HTTPS子域名。

比如，`https://www.httpwatch.com`和`https://store.httpwatch.com`，就共享同一个IP地址。

（4）转移服务器时要购买新证书

IS的做法是生成一个可以转移的`.pfx`文件，并加以密码保护。将这个文件传入其他服务器，将可以继续使用原来的SSL证书了。

（5）HTTPS太慢

第一次打开网页的时候，HTTPS协议会比HTTP协议慢一点，这是因为读取和验证SSL证书的时间。

但是，一旦有效的HTTPS连接建立起来，再刷新网页，两种协议几乎没有区别。

（6）有了HTTPS，Cookie和查询字符串就安全了

虽然无法直接从HTTPS数据中读取Cookie和查询字符串，但是你仍然需要使它们的值变得难以预测。

（7）只有注册登录页，才需要HTTPS

这种想法很普遍。人们觉得，HTTPS可以保护用户的密码，此外就不需要了。在Twitter和Facebook上，劫持其他人的Session是非常容易的。

# 混合内容

初始前端页面通过HTTPS加载，但是其他资源（例如图像、视频、样式表、脚本）通过HTTP连接，这种情况称为混合内容。

混合内容会降低HTTPS网站的安全性和用户体验，混入内容中HTTP请求容易遭受到中间人攻击

混合内容分为两种：主动混合内容和被动混合内容。

## 被动混合内容

被动混合内容指的是不与页面其余部分进行交互的内容，从而使中间人攻击在拦截或更改改内容时能够执行的操作首先。

被动混合内容包括图像、视频和音频内容，以及无法与页面其余部分进行交互的内容

被动混合内容会为网站带来安全危险，例如攻击者可以拦截网站上的图像的HTTP请求，调换或者更换这些图像，另外攻击者可以使用混合内容请求跟踪用户，窥探用户隐私。

对这些混合内容，大多数浏览器允许渲染此类型的内容，但是会显示警告：

![](http://image.oldzhou.cn/FkA6VwAB-5grcMaSOPJ6ulbViqvH)

要注意！！！**从2020年2月开始，Chrome80会将所有图像、视频、音频等混合内容自动升级为HTTPS**。即使在HTML中的`src`为HTTP，但是Chrome实际发出的请求会自动升级为HTTPS，如果对应的HTTPS资源不存在就会导致加载失败。


## 主动混合内容

主动混合内容作为整体与页面进行交互，并且几乎允许攻击者对页面进行任何操作。主动混合内容包括浏览器可下载和执行的脚本、样式表、iframe、flash资源及其他代码

与被动混合内容相比，主动混合内容的威胁更大。攻击者可以拦截和重写主动内容，从而完全控制页面。攻击者可以更改页面任何内容，显示完全不同的内容、窃取密码、窃取cookie、将用户进行重定向等。

鉴于这种威胁的严重性，大部分浏览器都默认阻止此类型的内容以保护用户

![](http://image.oldzhou.cn/FvFKoWNld9AKUg2JaSOj5v2X3cMt)

## 使用CSP应对混合内容

内容安全政策（CSP）是一个多用途浏览器功能，可以用来管理大批量的混合内容。CSP报告机制可以用于跟踪网站上的混合内容，强制政策可以通过升级或组织混合内容。

可以在服务器的响应中添加`Content-Security-Policy`或`Content-Security-Policy-Report-Only`响应头为页面启用这些功能，或者在页面的`<head>`部分使用`<meat>`标签设置`Content-Security-Policy`来启用CSP

### 查找混合内容

服务器的响应头中添加`Content-Security-Policy-Report-Only`指令，可以启用此功能，另CSP手机页面的混合内容

```
Content-Security-Policy-Report-Only: default-src https: 'unsafe-inline' 'unsafe-eval'; report-uri https://example.com/reportingEndpoint
```

### 升级不安全请求

使用`upgrade-insecure-request`这个CSP指令来自动修正混合内容，它会让浏览器在进行网络请求之前升级不安全的网址，将HTTP请求自动升级为HTTPS请求

可以通过服务器添加`Content-Security-Policy`响应头来开启：

```
Content-Security-Policy: upgrade-insecure-requests
```

也可以使用`<meta>`元素开启：

```HTML
<meta http-equiv="Content-Security-Policy" content="upgrade-insecure-requests">
```

`upgrade-insecure-request`会级联到`<iframe>`文档中，从而保护整个页面。

### 组织所有混合内容

在浏览器不支持`upgrade-insecure-request`指令的话，可以使用`block-all-mixed-content`这个CSP指令，命令浏览器不加载混合内容，所有混合内容资源请求均被阻止。

这个选项也会级联到`<iframe>`文档中。

开启方式可以通过服务器欠佳响应头开启：

```
Content-Security-Policy: block-all-mixed-content
```

也可以在页面中开启：

```HTML
<meta http-equiv="Content-Security-Policy" content="block-all-mixed-content">
```

# 参考

- [SSL/TLS协议运行机制的概述@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html)
- [HTTPS 协议：点外卖的过程原来这么复杂！@infoQ](https://www.infoq.cn/article/bZTED-LPHd1JVQqFk2ql)
- [HTTPS的七个误解（译文）@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2011/02/seven_myths_about_https.html)
- [SSL延迟有多大？@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2014/09/ssl-latency.html)
- [HTTPS 升级指南@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2016/08/migrate-from-http-to-https.html)
- [数字签名是什么？@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)
- 第七章 确保Web安全的HTTPS@《图解HTTP》
- [什么是混合内容？@Web.dev](https://developers.google.com/web/fundamentals/security/prevent-mixed-content/what-is-mixed-content?hl=zh-cn)
- [防止混合内容@Web.dev](https://developers.google.com/web/fundamentals/security/prevent-mixed-content/fixing-mixed-content?hl=zh-cn)Cookie和查询字符串，但是你仍然需要使它们的值变得难以预测。

（7）只有注册登录页，才需要HTTPS

这种想法很普遍。人们觉得，HTTPS可以保护用户的密码，此外就不需要了。在Twitter和Facebook上，劫持其他人的Session是非常容易的。

## 参考

- [SSL/TLS协议运行机制的概述@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html)
- [HTTPS 协议：点外卖的过程原来这么复杂！@infoQ](https://www.infoq.cn/article/bZTED-LPHd1JVQqFk2ql)
- [HTTPS的七个误解（译文）@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2011/02/seven_myths_about_https.html)
- [SSL延迟有多大？@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2014/09/ssl-latency.html)
- [HTTPS 升级指南@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2016/08/migrate-from-http-to-https.html)
- [数字签名是什么？@阮一峰的网络日志](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)
- 第七章 确保Web安全的HTTPS@《图解HTTP》
