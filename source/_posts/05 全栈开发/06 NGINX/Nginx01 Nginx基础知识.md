---
title: Nginx01 Nginx基础知识
top: false
date: 2019-06-23 21:09:33
updated: 2020-03-25 14:38:32
tags:
- Nginx
- 跨域
categories: 全栈开发
---

Nginx是一个强大的轻量级的高性能网页服务器、反向代理服务器和电子邮件代理服务器。作为负载均衡服务器，Nginx可以在内部直接支持Rails和PHP程序对外进行服务，也可以支持作为HTTP代理服务器对外进行服务。

<!-- more -->

# 基本知识

在介绍下面的使用Nginx进行跨域之前，需要对Nginx的一些基本知识有所了解。

# 安装

## Mac系统下

需要借助`homebrew`来安装，可以通过`brew -v`来查看是否安装了homebrew，如果没有安装，则通过终端命令安装

```BASH
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

安装成功就可以直接使用`homebrew`来安装Nginx：

```BASH
brew install nginx
```

安装完成后，主页的`index`文件在`/usr/local/var/www`目录下，对应的配置文件在`/usr/local/etc/nginx/nginx.conf`目录下

然后使用`nginx`命令就可以启动Nginx，默认`8080`端口

## 基本命令

使用Nginx的命令需要首先定位到`nginx.exe`所在的目录（Windows系统）

Nginx的一些基本命令：

```BASH
# 查看版本
nginx -v

# 启动
nginx

# 停止
nginx -s stop  # 快速停止Nginx，可能并不保存相关信息
nginx -s quit  # quit完整有序的停止Nginx，并保存相关信息。

# 重新启动
nginx -s reload  # 当配置信息修改，需要重新载入这些配置时使用此命令

# 重新打开日志文件
nginx -s reope

# 查看配置文件路径及是否调用有效
nginx -t
```

## 配置文件

Nginx的功能都是通过`nginx.conf`配置文件来实现的，一个服务器上可能会有多个配置文件，可以执行`nginx -t`来查看配置文件的路径以及是否调用有效。

Nginx的配置文件主要分为四部分：

1. `main`，全局设置，设置的指令将影响其他所有部分的设置
2. `server`，主机设置，用于指定虚拟主机域名、IP和端口
3. `upstream`，上游服务器设置，主要为反向代理、负载均衡相关配置
4. `location`，URL匹配特定设置

他们之间，`location`继承`server`，`server`继承`main`，`upstream`既不会继承指令也不会被继承。

## 关于`location`

Nginx配置文件中的`loaction`是主机访问的地址，在Nginx服务器做一个代理，转发到`location`中配置的地址。它有如下的匹配指令：

- `=`表示进行普通字符精确匹配，只有请求的URL路径与后面的字符串完全相等时，才会命中
- `~`表示执行一个正则匹配，区分大小写
- `~*`表示执行一个正则匹配，不区分大小写
- `^~`表示普通字符匹配，如果该选项匹配，只匹配该选项，不匹配别的选项，一般用来匹配目录
- `@`定义一个命名的`location`，使用在内部定向时，例如`error_page`, `try_files`

`loaction`匹配的优先级（与`loaction`在配置文件中的顺序无关）

- `=`精确匹配会第一个被处理。如果发现精确匹配，Nginx停止搜索其他匹配。
- 普通字符匹配，正则表达式规则和长的块规则将被优先和查询匹配，也就是说如果该项匹配还需去看有没有正则表达式匹配和更长的匹配。
- `^~ `则只匹配该规则，Nginx停止搜索其他匹配，否则Nginx会继续处理其他`loaction`指令。
- 最后匹配理带有`~`和`~*`的指令，如果找到相应的匹配，则Nginx停止搜索其他匹配；当没有正则表达式或者没有正则表达式被匹配的情况下，那么匹配程度最高的逐字匹配指令会被使用。

```bash
location  = / {
  # 只匹配"/".
  [ configuration A ]
}

location  / {
  # 匹配任何请求，因为所有请求都是以"/"开始
  # 但是更长字符匹配或者正则表达式匹配会优先匹配
  [ configuration B ]
}

location ^~ /images/ {
  # 匹配任何以 /images/ 开始的请求，并停止匹配 其它location
  [ configuration C ]
}

location ~* \.(gif|jpg|jpeg)$ {
  # 匹配以 gif, jpg, or jpeg结尾的请求.
  # 但是所有 /images/ 目录的请求将由 [Configuration C]处理.
  [ configuration D ]
}
```

## 关于`rewrite`

`rewrite`只能对URL中的路径部分除去传递的参数外的字符串起作用，不关心域名部分和查询参数，例如：

```TEXT
http://www.baidu.com/a/bb/ccc.php?id=1&uu=str
```

`rewrite`只会对`/a/bb/ccc.php`重写。

语法：

```
rewrite regex replacement [flag]
```

执行顺序是首先执行`location`匹配，然后执行`location`中的`rewirte`指令。

## 关于`proxy_pass`

`proxy_pass`地址后面加上了`/`则相当于转发到绝对根路径，Nginx不会对`loaction`中匹配的路径部分进行转发，例如：

```NGINX
location ^~/proxy/html/ {
  # 匹配任何以/proxy/html/开头的地址，匹配符合以后不继续往下搜索
  proxy_pass http://www.b.com/
}
```

如果请求的url为：`http://localhost/proxy/html/test.json`，则转发后的地址是：`http://www.b.com/test.json`。

如果`proxy_pass`地址后面不加`/`，则Nginx会对`loaction`中匹配的路径部分进行转发，例如:

```NGINX
location ^~/proxy/html/{
  # 匹配任何以/proxy/html/开头的地址，匹配符合以后不继续往下搜索
  proxy_pass http://www.b.com
}
```

如果请求的url为：`http://localhost/proxy/html/test.json`，则转发后的地址是：`http://www.b.com/proxy/html/test.json`。

# Nginx部署多个前端代码

前一段项目中要在同一个端口下部署两套前端代码，实现的效果是：

- 根目录匹配A系统打包的前端代码
- `v0`目录下匹配B系统打包的前端代码

配置文件如下：

```NGINX
server {
	listen 9001;

	# static front page for new system
    location / {
        root        /etc/nginx/html;
        index       index.html;
    }

	# static front page for old system
	location /v0 {
	    alias   	/etc/nginx/html/v0;
	    index	    index.html;
	}
}
```

根目录下使用的`root`来匹配目录，`v0`下使用的`alias`来匹配目录，由于使用的是Dockert径向部署，对应的目录都应该是指向的是容器内部的地址，而实际资源在服务器上的目录应该是与Docker设置的镜像相匹配的

注意`root`与`alias`的区别，例如：

```NGINX
# 错误写法，会报404
location /admin {
    root /projects/admin/;
    index index.html;
}

location /admin {
    alias /projects/admin/;
    index index.html;
}
```
当访问`http://***/admin`，`root`实际指向的是`http://***/admin/projects/admin/`, `alias`实际指向的是`http://***/admin`, 可以在`/projects/admin/`找到`index.html`

# Nginx的正向代理

正向代理的工作原理就像一个跳板，简单的说，我是一个用户，我访问不了某网站，但是我能访问一个代理服务器这个代理服务器呢,他能访问那个我不能访问的网站。

于是我先连上代理服务器，告诉他我需要那个无法访问网站的内容，代理服务器去取回来,然后返回给我。从网站的角度,只在代理服务器来取内容的时候有一次记录。有时候并不知道是用户的请求,也隐藏了用户的资料,这取决于代理告不告诉网站。

结论就是正向代理是一个位于客户端和原始服务器(Origin Server)之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。客户端必须要进行一些特别的设置才能使用正向代理。

# Nginx的反向代理

> 正向代理代理的是客户端，反向代理代理的服务器

## 什么是反向代理

所谓反向代理（Reverse Proxy）方式是指以代理服务器来接受Internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给Internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。

![](http://image.oldzhou.cn/18-11-15/20784963.jpg)

从上图可以看出，反向代理服务器位于网站机房，代理网站web服务器接受Http请求，对请求进行转发。

例用户访问`http://ooxx.me/readme`，但`ooxx.me`上并不存在`readme`页面，他是偷偷从另外一台服务器上取回来,然后作为自己的内容吐给用户，但用户并不知情。这里所提到的`ooxx.me`这个域名对应的服务器就设置了反向代理功能。

反向代理对于客户端而言它就像是原始服务器，并且客户端不需要进行任何特别的设置。客户端向反向代理发送普通请求，接着反向代理将判断向何处(原始服务器)转交请求，并将获得的内容返回给客户端，就像这些内容原本就是它自己的一样。

## 反向代理的作用

（1）保护网站安全：任何来自Internet的请求都必须先经过代理服务器；

![](http://image.oldzhou.cn/18-11-15/23647615.jpg)

（2）通过配置缓存功能加速Web请求：可以缓存真实Web服务器上的某些静态资源，减轻真实Web服务器的负载压力；

![](http://image.oldzhou.cn/Fj11NRzqCe24KCIMQWmb0U81vRQO)

（3）实现负载均衡：充当负载均衡服务器均衡的分发请求，平衡集群中各个服务器的负载压力；

![](http://image.oldzhou.cn/FpmutJqv9nekezJRPjcWJAV7bA6L)


# 利用Nginx的反向代理实现跨域

利用Nginx反向代理实现跨域，不需要目标服务器配合，但需要搭建中转Nginx服务器，用于转发请求。

## 实现原理

![](http://image.oldzhou.cn/FjmlXAq82rM10u8-TVzBiHxs0--b)

对于请求者而言，发送的不是跨域的请求，而是对本地资源的请求，Nginx识别出特定的本地资源请求，转发到真实的跨域的资源的地址。

而作为服务端而言，是不存在跨域的（需要添加对应的请求头），所以会把资源返回给Nginx服务，再由Nginx服务将资源返回给真正的请求页面。

## 基础配置

首先找到`nginx.conf`中的下面这部分内容：

```NGINX
server {
  listen 80;
  server_name localhost;
  location / {
    root   ../Project;
    index  index.html index.htm;
  }
}
```

其中`server`代表启动的一个服务，`location`是一个定位规则，是Nginx用来跨域的入口。

`location /`的意思是所有以`/`开头的地址，实际上是所有请求，后面的地址可以是绝对地址，也可以是相对地址；`root`的意思是去请求相对上一层中的Project文件夹里的文件，`index`是去指定首页。

在`location /{}`中添加一下代码：

```TEXT
add_header 'Access-Control-Allow-Origin' '*'';
add_header 'Access-Control-Allow-Credentials' 'true';
add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
rewrite regex replacement [flag]
proxy_pass http://ip:port/;
```

其中：

（1）第一条指令：授权从`other.subdomain.com`的请求；是W3C标准里用来检查该跨域请求是否可以被通过；

（2）第二条指令：当该标志为`true`时，响应于该请求可以被暴露；

（3）第三条指令：制定请求的方式，可以使`GET`/`POST`等；

（4）第四条指令：对域名（根路径）后面除去传递的参数外的字符串的`url`进行重写及重定向；

> URL是URI的子集。任何东西，只要能够唯一地标识出来，都可以说这个标识是URI。如果这个标识是一个可获取到上述对象的路径，那么同时它也可以是一个URL；但如果这个标识不提供获取到对象的路径，那么它就必然不是URL。

（4）第五条指令：实际要访问的请求地址；

前三条是用来指定自己的服务器是否可以被跨域访问的指令，是可以没有的。

第四条利用Nginx提供的变量，结合正则表示和标志位来实现比较复杂的重定向，简单的情况也是可以没有的。

## 实现方法1：`location` + `proxy_pass`

适用于将`location`直接转发至`proxy_pass`地址的情况，这种情况一般比较简单。

比如我们要访问的跨域资源位于` http://115.29.203.53:10013/students/`，前端代码如下：

```JS
var button = document.getElementById("button");

$(button).click(function() {
  $.get("students", function(data) {
    alert(data.data[0].id)
  }, "json");

  $.getJSON("students", function(data) {
    alert(data.data[0].id)
  });

  $.ajax({
    url: "students",
    type: "GET",
    dataType: "json",
    success: function(data) {
      alert(data.data[0].id)
    }
  })
})
```

上面的三种方法都可以实现跨域，要注意的是使用`get`方法获取JSON数据时一定要声明第四个参数`dataType`为`JSON`，否则总是无法获得正确的对象。

跨域的部分，`url`地址是一个相对地址`students`，转化为绝对地址就是`http://localhost/Project2/task5/students/`。而本地并没有这样的资源，所以需要利用Nginx进行转发处理。

```NGINX
server {
  listen 80;
  server_name localhost;
  location ^~/Project2/task5/students/ {
    proxy_pass http://115.29.203.53:10013/students/;
  }
}
```

配置完成后重启Nginx服务，首先定位到`nginx.exe`所在的目录（Windows系统），然后：

```BASH
nginx.exe -s reload
```

这个是比较蠢的做法，直接将全部的URL替换为最终的真实URL，这样就将地址转发出去了，实现了跨域。

## 实现方法2：`location` + `proxy_pass` + `rewrite`

适用于直接将`location`转发至`proxy_pass`不能满足需要，需要对虚拟根路径及后面的路径使用`rewrite`进行改写的情况。例如：

我们主机的地址是`www.a.com/html/index.html`，想请求`www.b.com/api/msg?method=1&para=2`，请求如下：

```JS
$.ajax({
  type: "get",
  url: "www.b.com/api/msg?method=1&para=2",
  success: function(res) {
    alert("success")
  }
})
```

这样必然因为跨域问题发生错误，无法获得`b`网站的数据。将请求更改为：

```JS
$.ajax({
  type: "get",
  url: "proxy/api/msg?method=1&para=2",
  // 绝对路径是 www.a.com/proxy/html/api/msg?method=1&para=2
  success: function(res) {
    alert("success")
  }
})
```

这是的URL请求的就是本地的资源了，但是在本地并不存在相应的数据，所以需要将这个地址通过Nginx转发出去。

在刚才的路径中匹配到这个请求，在`location`下面再添加一个`location`：

```NGINX
server {
  listen 80;
  server_name localhost;
  location / {
    root   ../Project;
    index  index.html index.htm;
  }

  # 匹配任何以 /proxy/html/ 开头的地址，匹配符合以后不继续往下搜索
  location ^~/proxy/html/ {
    rewrite ^/proxy/html/(.*)$ /$1 break;
    proxy_pass http://www.b.com/
  }
}
```

`location ^~/proxy/html/`用于拦截请求，是一个匹配规则，匹配任何以`proxy/html/`开头的地址，这里匹配到的就是`proxy/html/api/msg?method=1&para=2`

`rewrite ^proxy/html/(.*)$ /$1 break`用来重写拦截的请求，并且只对**域名后面除去传递参数外的字符**起作用，即重写上面匹配到的地址的这一部分进行重写：`proxy/html/api/msg`

`rewrite`后面是一个正则表达式，表示匹配以`/proxy/html/`开头的任何字符至结尾，并且将`proxy/html/`后面的任意字符存到第一个捕获组`$1`之中，`break`表示匹配一个后停止。

重写的结果是：`/api/msg`

`proxy_pass http://www.b.com/`用来把请求代理到其他主机，即`www.a.com`代理到`www.b.com`，请求路径最终变为`http://www.b.com/api/msg? method=1&para=2`


配置完成后重启Nginx服务即可。

这样就可以更改上面的情况1中的做法，JS文件不变，Nginx的`location`的配置如下：

```NGINX
server {
  listen 80;
  server_name localhost;
  location ^~/Project2/task5/students/ {
    rewrite ^/Project2/task5/(.*)$ /$1 break;
    proxy_pass http://115.29.203.53:10013/;
  }
}
```
同样可以实现跨域

## 又一个例子

前端请求：

```JS
$.ajax({
  type: "get",
  url: "sohu/api/msg?method=1&para=2",
  // 真实地址是 www.c.com/proxy/html/api/msg?method=1&para=2
  success: function(res) {
    alert("success")
  }
})
```

Nginx进行如下配置：

```NGINX
server {
  listen 80;
  server_name localhost;
  location ^~/sohu{
    # 匹配任何以/proxy/html/开头的地址，匹配符合以后不继续往下搜索
    rewrite ^.+sohu/?(.*)$ /$1 break;
    proxy_pass http://www.sohu.com/
  }
}
```

所以：

- `location`定位到：`/sohu/api/msg?method=1&para=2`
- `rewrite`的结果：`api/msg`
- `proxy_pass`后的结果：`http://www.sohu.com/api/msg?method=1&para=2`

## 最后一个例子

![](http://image.oldzhou.cn/FnnSs8kLR2yySIPf8CLJ7cqfsyFQ)

# 参考

- [MAC下安装nginx@segmentfault](https://segmentfault.com/a/1190000016020328)
- [nginx服务器安装及配置文件详解@Sean's Note](http://seanlook.com/2015/05/17/nginx-install-and-config/)
- [nginx反向代理服务器的工作原理@CSDN](https://blog.csdn.net/ywl570717586/article/details/51556912)
- [nginx配置location总结及rewrite规则写法@segmentfault](https://segmentfault.com/a/1190000002797606)
- [Nginx proxy pass简单用法@51CTO](https://blog.51cto.com/stevenlee87/1188295)
- [最简单实现跨域的方法----使用nginx反向代理@CSDN](https://blog.csdn.net/shendl/article/details/48443299)
- [一文弄懂Nginx的location匹配@segmentfault](https://segmentfault.com/a/1190000013267839)
- [Nginx同一个server部署多个静态资源目录@掘金](https://juejin.im/post/5d7b72b7e51d453b386a63bc)
