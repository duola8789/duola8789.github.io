---
title: Nginx02 Nginx与前端开发
top: false
date: 2020-07-27 11:46:26
updated: 2020-07-27 11:46:28
tags:
- Nginx
categories: 全栈开发
---

Nginx在前端开发中的应用。

<!-- more -->

# Nginx与Node.js

> “Nginx是一款轻量级的HTTP服务器，采用事件驱动的异步非阻塞处理方式框架，这让其具有极好的IO性能，时常用于服务端的反向代理和负载均衡。”

作为前端开发，即使没用过Nginx，但一定听说过上面这句话。这句经典的话，基本构成了所有人对Nginx的第一印象。

Nginx发布于2004年，经过初期几年的沉淀之后，迅速蹿升为“网红”，成为了当年互联网技术圈最火的词汇和技术。然而经过多年的发展，到现在，当年的网红早已“过气”。因为如今基本上所有的大型网站都搭建在Nginx之上，Nginx不再是一个什么新词，而是互联网网站搭建的必选技术之一。看到这里，“HTTP服务器”、“事件驱动”、“异步非阻塞”以及Nginx的网红经历，是不是让前端童鞋们想到了Nodejs?

在工作上，由于工作平台和语言的原因，对于大部分前端童鞋，更倾向于用Nodejs来搭建服务器，进而实现一些需求，对Nginx有天然的抗拒感。的确，Nginx中的绝大部分功能，如果单纯的使用Node.js也可以满足和实现。但实际上，Nginx和Node.js并不冲突，都有自己擅长的领域：Nginx更擅长于底层服务器端资源的处理（静态资源处理转发、反向代理，负载均衡等），Node.js更擅长于上层具体业务逻辑的处理。两者可以实现完美组合，助力前端开发。

首章最后要说几句。本文的目的是通过对Nginx的简单介绍，来让前端童靴了解其实通过Nginx可以强有力地助力前端开发：完全可以把之前Node.js的一些工作放到Nginx上，而不是痛苦地在npm中找包或者造轮子。但实际上，Nginx种看似简单的配置，实则学问深深。在Nginx实现一个同样的功能，不同的配置编写写法，效率上可能差上好几倍。而这些完全是在建立在对Nginx原理的深入理解和常年的配置运维经验上，哪怕是你们公司的后端都可能对Nginx的了解并不深入。如果真的想深入学习Nginx，还是找专业的SA或者PE请教吧。

# 代理

互联网应用基本都基于CS基本结构，即client端和server端。代理其实就是在client端和真正的server端之前增加一层提供特定服务的服务器，即代理服务器。

## 1 正向代理

反向代理不好理解，正向代理大家总有用过，翻墙工具其实就是一个正向代理工具。它会把访问墙外服务器server的网页请求，代理到一个可以访问该网站的代理服务器proxy，这个代理服务器proxy把墙外服务器server上的网页内容获取，再转发给客户。具体的流程如下图。

![nginx-proxy](http://image.oldzhou.cn/18-10-26/9489928.jpg)

## 2 反向代理

反向代理则正好相反，先看流程图图。

![nginx-proxy-reverse](http://image.oldzhou.cn/18-10-26/53488822.jpg)

在反向代理中（事实上，这种情况基本发生在所有的大型网站的页面请求中），客户端发送的请求，想要访问server服务器上的内容。但将被发送到一个代理服务器proxy，这个代理服务器将把请求代理到和自己属于同一个LAN下的内部服务器上，而用户真正想获得的内容就储存在这些内部服务器上。看到区别了吗，这里proxy服务器代理的并不是客户，而是服务器，即向外部客户端提供了一个统一的代理入口，客户端的请求，都先经过这个proxy服务器，至于在内网真正访问哪台服务器内容，由这个proxy去控制。一般代理是指代理客户端，而这里代理的对象是服务器，这就是“反向”这个词的意思。Nginx就是来充当这个proxy的作用。

概括说：就是代理服务器和真正server服务器可以直接互相访问，属于一个LAN（服务器内网）；代理对用户是透明的，即无感知。不论加不加这个反向代理，用户都是通过相同的请求进行的，且不需要任何额外的操作；代理服务器通过**代理内部服务器**接受域外客户端的请求，并将请求发送到对应的内部服务器上。

## 3 为什么要Nginx反向代理

使用反向代理最主要的两个原因：

### 1）安全及权限

可以看出，使用反向代理后，用户端将无法直接通过请求访问真正的内容服务器，而必须首先通过Nginx。可以通过在Nginx层上将危险或者没有权限的请求内容过滤掉，从而保证了服务器的安全。

### 2）负载均衡

例如一个网站的内容被部署在若干台服务器上，可以把这些机子看成一个集群，那么Nginx可以将接收到的客户端请求“均匀地”分配到这个集群中所有的服务器上（内部模块提供了多种负载均衡算法），从而实现服务器压力的负载均衡。此外，nginx还带有健康检查功能（服务器心跳检查），会定期轮询向集群里的所有服务器发送健康检查请求，来检查集群中是否有服务器处于异常状态，一旦发现某台服务器异常，那么在以后代理进来的客户端请求都不会被发送到该服务器上（直到后面的健康检查发现该服务器恢复正常），从而保证客户端访问的稳定性。

# 前端可以用Nginx做些什么

下面的内容建立在对Nginx配置有基本认知的情况下。如果没有的话，请先从网上查阅资料（例如[基本配置](http://www.nginx.cn/76.html)）做简单了解。如果你想本地安装Nginx，强烈建议采用[源码编译安装](http://www.nginx.cn/install)，这样后续添加模块更为方便。

## 1 快速实现简单的访问限制

经常会遇到希望网站让某些特定用户的群体（比如只让公司内网）访问，或者控制某个uri不让人访问。Nginx配置如下：

```NGINX
location / {
  deny      192.168.1.100;
  allow     192.168.1.10/200;
  allow     10.110.50.16;
  deny      all;
}
```

其实`deny`和`allow`是`ngx_http_access_module`模块（已内置）中的语法。采用的是从上到下匹配方式，匹配到就跳出不再继续匹配。上述配置的意思就是，首先禁止`192.168.1.100`访问，然后允许`192.168.1.10-200`ip段内的访问（排除`192.168.1.100`），同时允许`10.110.50.16`这个单独ip的访问，剩下未匹配到的全部禁止访问。实际生产中，经常和`ngx_http_geo_module`模块（可以更好地管理ip地址表，已内置）配合使用。

## 2 解决跨域

在众多的解决跨域方式中，都不可避免的都需要服务端进行支持，使用Nginx可以纯前端解决请求跨域问题。特别是在前后端分离调试时， 经常需要在本地起前端工程，接口希望拉取服务端的实际数据而不是本地的mock。而如果本地程序直接访问远程接口，肯定会遇到跨域问题。

现在前端成熟的做法，一般是把node proxy server集成进来。事实上，用Nginx同样可以解决问题，甚至可以应用于线上。

本地起一个`nginx server`。`server_name`是`mysite-base.com`，比如现在需要请求线上`www.kaola.com`域下的线上接口`www.kaola.com/getPCBanner…`的数据，当在页面里直接请求，浏览器会报错：

![浏览器报错](http://image.oldzhou.cn/18-10-26/49321568.jpg)

为了绕开浏览器的跨域安全限制，现在需要将请求的域名改成`mysite-base.com`。同时约定一个url规则来表明代理请求的身份，然后Nginx通过匹配该规则，将请求代理回原来的域。Nginx配置如下：

```NGINX
# 请求跨域，这里约定代理请求url path是以/apis/开头
location ^~/apis/ {
  # 这里重写了请求，将正则匹配中的第一个()中$1的path，拼接到真正的请求后面，并用break停止后续匹配
  rewrite       ^/apis/(.*)$ /$1 break;
  proxy_pass    https://www.kaola.com/;
}
```

在页面代码里，把请求url换成`http://mysite-base.com/apis/getPCBannerList.html`。这样就可以正常请求到数据。

这样其实是通过Nginx，用类似于hack的方式规避掉了浏览器跨域限制，实现了跨域访问。

## 3 适配PC与移动环境

现在很多网站都存在PC站和H5站两个站点，因此根据用户的浏览环境自动切换站点是很常见的需求。Nginx可以通过内置变量`$http_user_agent`，获取到请求客户端的userAgent，从而知道用户处于移动端还是PC，进而控制重定向到H5站还是PC站。

例如PC端站点是`mysite-base.com`，H5端是`mysite-base-H5.com`。pc端Nginx配置如下：

```NGINX
location / {
  # 移动、pc设备适配
  if ($http_user_agent ~* '(Android|webOS|iPhone|iPod|BlackBerry)') {
    set $mobile_request '1';
  }
  if ($mobile_request = '1') {
    rewrite ^.+ http://mysite-base-H5.com;
  }
}
```

这样当浏览设备切换成移动模式，再次刷新页面后，站点被自动切换到H5站。如下：

![站点切换](http://image.oldzhou.cn/18-10-26/76764633.jpg)

## 4 合并请求

前端性能优化中重要一点就是尽量减少http资源请求的数量。通过`nginx-http-concat`模块（淘宝开发的第三方模块，需要单独安装）用一种特殊的请求url规则（例子：`example.com/??1.js,2.js,3.js`），前端可以将多个资源的请求合并成一个请求，后台Nginx会获取各个资源并拼接成一个结果进行返回。例如上面的例子通过一个请求将`1.js`,`2.js`,`3.js`三个js资源合并成一个请求，减少了浏览器开销。

本地`mysite-base.com`为例，`static/js`文件夹下有三个文件，文件内容很简单，分别为：

![](http://image.oldzhou.cn/18-10-26/41887509.jpg)

Nginx配置如下：

```NGINX
# js资源http-concat
# nginx-http-concat模块的参数远不止下面三个，剩下的请查阅文档
location /static/js/ {
  concat            on; # 是否打开资源合并开关
  concat_types      application/javascript; # 允许合并的资源类型
  concat_unique     off; # 是否允许合并不同类型的资源
  concat_max_files  5; # 允许合并的最大资源数目
}
```

当在浏览器请求`http://mysite-base.com/static/js/??a.js,b.js,c.js`时，发现三个js被合并成一个返回了，如下图：

![](http://image.oldzhou.cn/18-10-26/69630633.jpg)

## 5 图片处理

在前端开发中，经常需要不同尺寸的图片。现在的云储存基本对图片都提供有处理服务（一般是通过在图片链接上加参数）。其实用Nginx，可以通过几十行配置，搭建出一个属于自己的本地图片处理服务，完全能够满足日常对图片的裁剪/缩放/旋转/图片品质等处理需求。

要用到`ngx_http_image_filter_module`模块。这个模块是非基本模块，需要安装。下面是图片缩放功能部分的Nginx配置：

```NGINX
# 图片缩放处理
# 这里约定的图片处理url格式：以 mysite-base.com/img/路径访问
location ~* /img/(.+)$ {
	alias           /Users/cc/Desktop/server/static/image/$1; #图片服务端储存地址
	set $width      -; #图片宽度默认值
	set $height     -; #图片高度默认值
	if ($arg_width != "") {
		set $width $arg_width;
	}
	if ($arg_height != "") {
		set $height $arg_height;
	}
	image_filter_resize         $width $height; #设置图片宽高
	image_filter_buffer         10M; #设置Nginx读取图片的最大buffer。
	image_filter_interlace      on; #是否开启图片图像隔行扫描
	error_page                  415 = 415.png; #图片处理错误提示图，例如缩放参数不是数字
}
```

![](http://image.oldzhou.cn/18-10-26/19593548.jpg)

这里只是最基本的配置。此外，可以通过`proxy_cache`配置Nginx缓存，避免每次请求都重新处理图片，减少Nginx服务器处理压力；还以可以通过和`nginx-upload-module`一起使用加入图片上传的功能等。

# 参考

- [掘金 - Nginx与前端开发](https://juejin.im/post/5bacbd395188255c8d0fd4b2?utm_source=gold_browser_extension)
